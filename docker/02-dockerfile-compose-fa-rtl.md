<div dir="rtl" align="right">

<span dir="ltr">Dockerfile</span> و <span dir="ltr">Docker Compose</span> در پروژه‌های واقعی

این سند ادامه‌ی راهنمای <span dir="ltr">Docker</span> است. فرض می‌کنیم مفاهیم <code dir="ltr">Image</code>، <code dir="ltr">Container</code>، <code dir="ltr">Port Mapping</code>، <code dir="ltr">Volume</code> و <code dir="ltr">Network</code> را می‌دانیم.

هدف این فایل پاسخ به دو سؤال است:

چگونه محیط اجرای یک <span dir="ltr">Application</span> را به شکل قابل‌تکرار با <code dir="ltr">Dockerfile</code> بسازیم؟

چگونه چند <span dir="ltr">Service</span> مرتبط را با <code dir="ltr">Docker Compose</code> به عنوان یک پروژه واحد اجرا و مدیریت کنیم؟

<a id="toc" name="toc"></a>

فهرست مطالب

<ol dir="rtl" align="right">
  <li><a href="#purpose"><span dir="ltr">Dockerfile</span> و <span dir="ltr">Compose</span> چه مسئله‌ای را حل می‌کنند؟</a></li>
  <li><a href="#dockerfile"><span dir="ltr">Dockerfile</span> چیست؟</a></li>
  <li><a href="#dockerfile-instructions">دستورهای اصلی <span dir="ltr">Dockerfile</span></a></li>
  <li><a href="#build-context"><span dir="ltr">Build Context</span> و .<span dir="ltr">dockerignore</span></a></li>
  <li><a href="#cache"><span dir="ltr">Layer</span> و <span dir="ltr">Build Cache</span></a></li>
  <li><a href="#cmd-entrypoint"><span dir="ltr">CMD</span> و <span dir="ltr">ENTRYPOINT</span></a></li>
  <li><a href="#copy-bind"><span dir="ltr">COPY</span> و <span dir="ltr">Bind Mount</span></a></li>
  <li><a href="#node">سناریوی <span dir="ltr">Node.js:</span> از <span dir="ltr">Dockerfile</span> تا <span dir="ltr">Image</span></a></li>
  <li><a href="#flask">سناریوی <span dir="ltr">Flask: Application</span> داخل <span dir="ltr">Container</span></a></li>
  <li><a href="#compose"><span dir="ltr">Docker Compose</span> چیست؟</a></li>
  <li><a href="#compose-structure">ساختار <span dir="ltr">compose.yaml</span></a></li>
  <li><a href="#service"><span dir="ltr">Service</span> و تنظیمات اصلی آن</a></li>
  <li><a href="#compose-network"><span dir="ltr">Network</span> در <span dir="ltr">Compose</span></a></li>
  <li><a href="#compose-storage"><span dir="ltr">Volume</span> و <span dir="ltr">Bind Mount</span> در <span dir="ltr">Compose</span></a></li>
  <li><a href="#compose-env">.<span dir="ltr">env</span>، <span dir="ltr">environment</span> و <span dir="ltr">env_file</span></a></li>
  <li><a href="#health"><span dir="ltr">depends_on</span> و <span dir="ltr">healthcheck</span></a></li>
  <li><a href="#runtime-options"><span dir="ltr">restart</span>، <span dir="ltr">command</span> و <span dir="ltr">entrypoint</span></a></li>
  <li><a href="#real-project">سناریوی پروژه واقعی <span dir="ltr">Backend</span> + <span dir="ltr">PostgreSQL</span> + <span dir="ltr">Nginx</span></a></li>
  <li><a href="#workflow">چرخه کار روزانه با <span dir="ltr">Docker Compose</span></a></li>
  <li><a href="#dev-prod"><span dir="ltr">Development</span> در برابر <span dir="ltr">Production</span></a></li>
  <li><a href="#debug"><span dir="ltr">Debug</span> و <span dir="ltr">Troubleshooting</span></a></li>
  <li><a href="#mistakes">اشتباهات رایج</a></li>
  <li><a href="#cheatsheet"><span dir="ltr">Cheat Sheet</span></a></li>
  <li><a href="#next">آمادگی برای سند <span dir="ltr">YAML</span></a></li>
</ol>

<a id="purpose" name="purpose"></a>

1. <span dir="ltr">Dockerfile</span> و <span dir="ltr">Compose</span> چه مسئله‌ای را حل می‌کنند؟

با <span dir="ltr">Docker</span> خام می‌توانیم <span dir="ltr">Container</span> بسازیم، اما اگر ساخت محیط <span dir="ltr">Application</span> را دستی انجام دهیم، مراحل قابل‌تکرار نیستند.

مثلاً این روند:

<pre dir="ltr"><code>Ubuntu Container
   ↓
apt install ...
   ↓
pip install ...
   ↓
copy source code
   ↓
start app</code></pre>

اگر فقط در <span dir="ltr">Terminal</span> انجام شود، عضو بعدی تیم دقیقاً نمی‌داند چه کارهایی انجام شده است.

<code dir="ltr">Dockerfile</code> مراحل ساخت <span dir="ltr">Image</span> را تبدیل به <span dir="ltr">Code</span> می‌کند:

<pre dir="ltr"><code>Dockerfile
   ↓
docker build
   ↓
Image</code></pre>

اگر پروژه چند بخش داشته باشد:

<pre dir="ltr"><code>Frontend
Backend
Database
Redis
Nginx</code></pre>

مدیریت همه‌ی آن‌ها با چندین <code dir="ltr">docker run</code> سخت می‌شود. <code dir="ltr">Docker Compose</code> تنظیم اجرای کل <span dir="ltr">Stack</span> را در یک فایل تعریف می‌کند.

پس:

<table dir="rtl">
  <thead><tr><th>ابزار</th><th>سؤال اصلی که پاسخ می‌دهد</th></tr></thead>
  <tbody>
    <tr><td><code dir="ltr">Dockerfile</code></td><td><span dir="ltr">Image</span> پروژه چگونه ساخته شود؟</td></tr>
    <tr><td><code dir="ltr">Docker Compose</code></td><td><span dir="ltr">Service</span>های پروژه چگونه کنار هم تعریف و اجرا شوند؟</td></tr>
  </tbody>
</table>

<a id="dockerfile" name="dockerfile"></a>

2. <span dir="ltr">Dockerfile</span> چیست؟

<code dir="ltr">Dockerfile</code> فایل متنی‌ای است که مراحل ساخت <span dir="ltr">Image</span> را تعریف می‌کند.

نمونه ساده:

FROM python:3.13-slim

WORKDIR /app

COPY app.py .

RUN pip install flask

EXPOSE 4000

CMD ["python", "app.py"]

مدل ذهنی:

<pre dir="ltr"><code>Base Image
   ↓
Working Directory
   ↓
Dependencies
   ↓
Application Files
   ↓
Default Runtime Command
   ↓
Final Image</code></pre>

<span dir="ltr">Build:</span>

docker build -t myapp:1.0.0 .

<span dir="ltr">Run:</span>

docker run -p 4000:4000 myapp:1.0.0

<a id="dockerfile-instructions" name="dockerfile-instructions"></a>

3. دستورهای اصلی <span dir="ltr">Dockerfile</span>

<code dir="ltr">FROM</code>

FROM node:22

<span dir="ltr">Base Image</span> را تعیین می‌کند.

<code dir="ltr">WORKDIR</code>

WORKDIR /app

مسیر کاری داخل <span dir="ltr">Image</span> را مشخص می‌کند. مدل ذهنی تقریبی آن شبیه <code dir="ltr">cd /app</code> است.

<code dir="ltr">COPY</code>

COPY app.js .

اگر <code dir="ltr">WORKDIR /app</code> باشد:

<pre dir="ltr"><code>Host ./app.js
     ↓ COPY
Image /app/app.js</code></pre>

<code dir="ltr">COPY</code> اتصال زنده نیست؛ نسخه فایل در زمان <span dir="ltr">Build</span> وارد <span dir="ltr">Image</span> می‌شود.

<code dir="ltr">RUN</code>

RUN npm install

یا:

RUN pip install -r requirements.txt

در زمان <span dir="ltr">Build</span> اجرا می‌شود.

<pre dir="ltr"><code>RUN
→ Build time

CMD / ENTRYPOINT
→ Container runtime</code></pre>

<code dir="ltr">ENV</code>

ENV APP_ENV=production

<span dir="ltr">Environment Variable</span> پیش‌فرض داخل <span dir="ltr">Image.</span>

<code dir="ltr">ARG</code>

ARG APP_VERSION=dev

برای <span dir="ltr">Build-time variable.</span>

docker build \
  --build-arg APP_VERSION=1.2.0 \
  -t myapp:1.2.0 .

<pre dir="ltr"><code>ARG → Build time
ENV → Image / Runtime environment</code></pre>

<code dir="ltr">EXPOSE</code>

EXPOSE 3000

<span dir="ltr">Port</span> مورد انتظار <span dir="ltr">Application</span> را بیان می‌کند، اما <span dir="ltr">Port</span> را روی <span dir="ltr">Host Publish</span> نمی‌کند.

<span dir="ltr">Publish</span> واقعی:

docker run -p 8080:3000 myapp

<code dir="ltr">CMD</code>

CMD ["node", "app.js"]

<span dir="ltr">Command</span> پیش‌فرض <span dir="ltr">Container.</span>

<code dir="ltr">ENTRYPOINT</code>

ENTRYPOINT ["python", "app.py"]

<span dir="ltr">Executable</span> اصلی <span dir="ltr">Container</span> را تعریف می‌کند.

<code dir="ltr">USER</code>

USER appuser

در پروژه واقعی اجرای <span dir="ltr">Application</span> با <span dir="ltr">User</span> غیر <span dir="ltr">root</span> معمولاً الگوی امن‌تری است، به شرط این‌که <span dir="ltr">Permission</span>ها درست تنظیم شده باشند.

<code dir="ltr">HEALTHCHECK</code>

HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1

در بسیاری از پروژه‌ها <span dir="ltr">Healthcheck</span> را در <span dir="ltr">Compose</span> تعریف می‌کنند تا <span dir="ltr">Runtime Configuration</span> کنار بقیه <span dir="ltr">Service</span>ها باشد.

<a id="build-context" name="build-context"></a>

4. <span dir="ltr">Build Context</span> و <code dir="ltr">.dockerignore</code>

docker build -t myapp .

نقطه آخر یعنی <span dir="ltr">Build Context</span> پوشه فعلی است.

<pre dir="ltr"><code>project/
├── app.js
├── package.json
└── Dockerfile</code></pre>

وقتی <span dir="ltr">Dockerfile</span> می‌گوید:

COPY app.js .

<span dir="ltr">Docker</span> فایل را از <span dir="ltr">Context</span> پیدا می‌کند.

اگر <span dir="ltr">Context</span> را <code dir="ltr">./backend</code> بدهیم:

docker build -t backend ./backend

<span dir="ltr">Dockerfile</span> به فایل‌های خارج از <span dir="ltr">Context</span> دسترسی مستقیم ندارد.

<code dir="ltr">.dockerignore</code>

<pre dir="ltr"><code>node_modules
.git
.env
*.log
__pycache__
venv
dist</code></pre>

مزایا:

<span dir="ltr">Build Context</span> کوچک‌تر

<span dir="ltr">Build</span> سریع‌تر

کاهش ورود فایل غیرضروری یا <span dir="ltr">Secret</span> به <span dir="ltr">Image</span>

<a id="cache" name="cache"></a>

5. <span dir="ltr">Layer</span> و <span dir="ltr">Build Cache</span>

ترتیب <span dir="ltr">Dockerfile</span> روی <span dir="ltr">Cache</span> اثر دارد.

برای <span dir="ltr">Node:</span>

FROM node:22
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .

اگر فقط <span dir="ltr">Source Code</span> تغییر کند ولی فایل‌های <span dir="ltr">Package</span> تغییر نکنند، <span dir="ltr">Docker</span> ممکن است <span dir="ltr">Layer</span> نصب <span dir="ltr">Dependency</span>ها را <span dir="ltr">reuse</span> کند.

برای <span dir="ltr">Python:</span>

FROM python:3.13-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .

اصل ذهنی:

<pre dir="ltr"><code>Dependency manifest
        │
        ▼
Install dependencies
        │
        ▼
Source code</code></pre>

تا تغییر کوچک در <span dir="ltr">Source</span> باعث نصب دوباره همه <span dir="ltr">Dependency</span>ها نشود.

<a id="cmd-entrypoint" name="cmd-entrypoint"></a>

6. <code dir="ltr">CMD</code> و <code dir="ltr">ENTRYPOINT</code>

<span dir="ltr">CMD</span>

CMD ["node", "app.js"]

<span dir="ltr">Command</span> پیش‌فرض است و با <span dir="ltr">Command</span> انتهای <code dir="ltr">docker run</code> راحت <span dir="ltr">Override</span> می‌شود:

docker run myapp bash

<span dir="ltr">ENTRYPOINT</span>

ENTRYPOINT ["python", "app.py"]

<span dir="ltr">Executable</span> اصلی را ثابت‌تر می‌کند.

ترکیب:

ENTRYPOINT ["python", "app.py"]
CMD ["--port", "3000"]

مدل <span dir="ltr">Runtime:</span>

<pre dir="ltr"><code>python app.py --port 3000</code></pre>

برای شروع:

<table dir="rtl">
  <thead><tr><th>دستور</th><th>مدل ذهنی</th></tr></thead>
  <tbody>
    <tr><td><code dir="ltr">CMD</code></td><td>دستور یا آرگومان پیش‌فرض که معمولاً در زمان اجرا قابل <span dir="ltr">Override</span> است.</td></tr>
    <tr><td><code dir="ltr">ENTRYPOINT</code></td><td>هستهٔ فرمان اجرایی <span dir="ltr">Container</span>.</td></tr>
    <tr><td><code dir="ltr">ENTRYPOINT + CMD</code></td><td><span dir="ltr">Executable</span> ثابت به‌همراه آرگومان‌های پیش‌فرض.</td></tr>
  </tbody>
</table>

<a id="copy-bind" name="copy-bind"></a>

7. <code dir="ltr">COPY</code> و <span dir="ltr">Bind Mount</span>

<span dir="ltr">COPY</span>

COPY app.js .

<pre dir="ltr"><code>Host app.js
   ↓ docker build
Image /app/app.js</code></pre>

بعد از <span dir="ltr">Build</span>، تغییر <span dir="ltr">Host</span> وارد <span dir="ltr">Image</span> قبلی نمی‌شود.

<span dir="ltr">Bind Mount</span>

docker run \
  -v "$PWD/app.js":/app/app.js \
  myapp

<pre dir="ltr"><code>Host app.js
     ⇅
Container /app/app.js</code></pre>

پس:

<table dir="rtl">
  <thead><tr><th>روش</th><th>زمان و رفتار</th></tr></thead>
  <tbody>
    <tr><td><code dir="ltr">COPY</code></td><td>از فایل در زمان <span dir="ltr">Build</span> یک نسخه داخل <span dir="ltr">Image</span> قرار می‌دهد.</td></tr>
    <tr><td><code dir="ltr">Bind Mount</code></td><td>در زمان <span dir="ltr">Runtime</span> فایل یا پوشهٔ واقعی <span dir="ltr">Host</span> را داخل <span dir="ltr">Container</span> متصل می‌کند.</td></tr>
  </tbody>
</table>

اگر کل <code dir="ltr">/app</code> را <span dir="ltr">Mount</span> کنی:

-v "$PWD":/app

محتویات قبلی <code dir="ltr">/app</code> داخل <span dir="ltr">Image</span> ممکن است پشت <span dir="ltr">Mount</span> پنهان شوند. برای همین مسیر <span dir="ltr">Mount</span> باید آگاهانه انتخاب شود.

<a id="node" name="node"></a>

8. سناریوی <span dir="ltr">Node.js:</span> از <span dir="ltr">Dockerfile</span> تا <span dir="ltr">Image</span>

ساختار:

<pre dir="ltr"><code>node-app/
├── app.js
├── package.json
├── package-lock.json
├── Dockerfile
└── .dockerignore</code></pre>

<code dir="ltr">app.js</code>:

const express = require("express");
const app = express();

app.get("/", (req, res) => {
  res.send("Hello from Docker");
});

app.listen(3000, "0.0.0.0", () => {
  console.log("Listening on port 3000");
});

<code dir="ltr">Dockerfile</code>:

FROM node:22

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

EXPOSE 3000

CMD ["node", "app.js"]

<code dir="ltr">.dockerignore</code>:

<pre dir="ltr"><code>node_modules
npm-debug.log
.git
.env</code></pre>

<span dir="ltr">Build:</span>

docker build -t node-app:1.0.0 .

<span dir="ltr">Run:</span>

docker run \
  --name node-app \
  -p 2000:3000 \
  node-app:1.0.0

<span dir="ltr">Test:</span>

curl http://localhost:2000

<pre dir="ltr"><code>curl
 ↓
Host :2000
 ↓
Container :3000
 ↓
node app.js</code></pre>

برای دیدن تفاوت <span dir="ltr">COPY</span> و <span dir="ltr">Bind Mount</span>، بعد از <span dir="ltr">Build</span> متن <span dir="ltr">Response</span> را روی <span dir="ltr">Host</span> تغییر بده. بدون <span dir="ltr">Build</span> مجدد <span dir="ltr">Image</span> همان نسخه قدیمی را دارد؛ با <span dir="ltr">Mount</span> کردن <code dir="ltr">app.js</code> نسخه فعلی <span dir="ltr">Host</span> دیده می‌شود.

<a id="flask" name="flask"></a>

9. سناریوی <span dir="ltr">Flask: Application</span> داخل <span dir="ltr">Container</span>

ساختار:

<pre dir="ltr"><code>flask-app/
├── app.py
├── requirements.txt
├── Dockerfile
└── .dockerignore</code></pre>

<code dir="ltr">app.py</code>:

from flask import Flask

app = Flask(__name__)

@app.get("/")
def hello():
    return "Hello World"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=4000)

<code dir="ltr">requirements.txt</code>:

<pre dir="ltr"><code>Flask</code></pre>

<code dir="ltr">Dockerfile</code>:

FROM python:3.13-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 4000

CMD ["python", "app.py"]

<span dir="ltr">Build:</span>

docker build -t flask-app:1.0.0 .

<span dir="ltr">Run:</span>

docker run \
  --name flask-app \
  -p 4000:4000 \
  flask-app:1.0.0

دو نکته:

<pre dir="ltr"><code>Application
    │
    ▼
0.0.0.0:4000 inside container
    │
    ▼
Docker port publishing
    │
    ▼
Host:4000</code></pre>

<a id="compose" name="compose"></a>

10. <span dir="ltr">Docker Compose</span> چیست؟

در پروژه واقعی معمولاً فقط یک <span dir="ltr">Container</span> نداریم:

<pre dir="ltr"><code>Application
├── Nginx
├── Backend
├── PostgreSQL
└── Redis</code></pre>

بدون <span dir="ltr">Compose</span> باید <span dir="ltr">Network</span>، <span dir="ltr">Volume</span> و <code dir="ltr">docker run</code>های مختلف را جدا مدیریت کنیم.

<span dir="ltr">Compose</span> معماری <span dir="ltr">Runtime</span> پروژه را در <span dir="ltr">YAML</span> تعریف می‌کند:

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"

اجرا:

docker compose up -d

پس:

<pre dir="ltr"><code>Dockerfile → Image definition
compose.yaml → Runtime topology</code></pre>

<span dir="ltr">Compose</span> جایگزین <span dir="ltr">Docker</span> نیست؛ روی همان <span dir="ltr">Docker Engine</span> کار می‌کند.

<a id="compose-structure" name="compose-structure"></a>

11. ساختار <code dir="ltr">compose.yaml</code>

سه بخش پرتکرار:

services:

volumes:

networks:

مثال:

services:
  backend:
    image: my-backend:1.0.0

  database:
    image: postgres:17

volumes:
  db-data:

networks:
  app-net:

<code dir="ltr">Service</code> یعنی یک بخش از <span dir="ltr">Application</span> که <span dir="ltr">Compose</span> آن را مدیریت می‌کند.

<a id="service" name="service"></a>

12. <span dir="ltr">Service</span> و تنظیمات اصلی آن

<code dir="ltr">image</code>

services:
  nginx:
    image: nginx:alpine

<code dir="ltr">build</code>

services:
  backend:
    build: ./backend

فرم کامل‌تر:

services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile

<code dir="ltr">ports</code>

ports:
  - "8080:80"

یعنی <span dir="ltr">Host</span> <code dir="ltr">8080</code> به <span dir="ltr">Container</span> <code dir="ltr">80</code>.

<code dir="ltr">environment</code>

environment:
  APP_ENV: development
  DB_HOST: database

<code dir="ltr">container_name</code>

<span dir="ltr">Compose</span> امکان نام ثابت دارد:

container_name: backend

اما معمولاً بهتر است بی‌دلیل به نام ثابت وابسته نشویم؛ خود <span dir="ltr">Compose Naming</span> و <span dir="ltr">Service Discovery</span> را مدیریت می‌کند.

<a id="compose-network" name="compose-network"></a>

13. <span dir="ltr">Network</span> در <span dir="ltr">Compose</span>

<span dir="ltr">Compose</span> معمولاً برای <span dir="ltr">Project</span> یک <span dir="ltr">Network</span> پیش‌فرض می‌سازد.

<pre dir="ltr"><code>backend
 database
 nginx
   │
   └── project_default</code></pre>

<span dir="ltr">Service</span>ها با نام <span dir="ltr">Service</span> همدیگر را پیدا می‌کنند.

اگر <span dir="ltr">Service</span> دیتابیس <code dir="ltr">database</code> باشد:

<pre dir="ltr"><code>database:5432</code></pre>

اشتباه رایج داخل <span dir="ltr">Backend:</span>

<pre dir="ltr"><code>DB_HOST=localhost</code></pre>

<code dir="ltr">localhost</code> داخل <span dir="ltr">Backend</span> یعنی خود <span dir="ltr">Backend Container</span>، نه <span dir="ltr">Database.</span>

درست:

<pre dir="ltr"><code>DB_HOST=database</code></pre>

<span dir="ltr">Network</span> اختصاصی:

services:
  backend:
    networks:
      - app-net

  database:
    networks:
      - app-net

networks:
  app-net:

<a id="compose-storage" name="compose-storage"></a>

14. <span dir="ltr">Volume</span> و <span dir="ltr">Bind Mount</span> در <span dir="ltr">Compose</span>

<span dir="ltr">Named Volume:</span>

services:
  database:
    image: postgres:17
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:

برای <span dir="ltr">Database Data</span> مناسب است.

<span dir="ltr">Bind Mount</span> در <span dir="ltr">Development:</span>

services:
  backend:
    build: ./backend
    volumes:
      - ./backend:/app

مدل پیشنهادی برای شروع:

<table dir="rtl">
  <thead><tr><th>نوع داده</th><th>انتخاب معمول</th></tr></thead>
  <tbody>
    <tr><td><span dir="ltr">Source Code</span> در محیط توسعه</td><td><code dir="ltr">Bind Mount</code></td></tr>
    <tr><td>دادهٔ پایگاه‌داده</td><td><code dir="ltr">Named Volume</code></td></tr>
  </tbody>
</table>

<a id="compose-env" name="compose-env"></a>

15. <code dir="ltr">.env</code>، <code dir="ltr">environment</code> و <code dir="ltr">env_file</code>

این سه مفهوم را قاطی نکن.

<code dir="ltr">environment</code>

services:
  backend:
    environment:
      APP_ENV: development
      DB_HOST: database

<span dir="ltr">Interpolation</span> با <code dir="ltr">.env</code>

<code dir="ltr">.env</code>:

APP_PORT=3000
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=change-me

<span dir="ltr">Compose:</span>

ports:
  - "${APP_PORT}:3000"

<span dir="ltr">Compose</span> مقدار <code dir="ltr">${APP_PORT}</code> را هنگام پردازش فایل جایگزین می‌کند.

<code dir="ltr">env_file</code>

services:
  backend:
    env_file:
      - .env

<span dir="ltr">Variable</span>های فایل وارد <span dir="ltr">Environment Container</span> می‌شوند.

برای <span dir="ltr">Repository:</span>

<table dir="rtl">
  <thead><tr><th>فایل</th><th>نقش</th><th>وضعیت در Git</th></tr></thead>
  <tbody>
    <tr><td><code dir="ltr">.env</code></td><td>مقادیر واقعی محیط، از جمله مقادیری که ممکن است حساس باشند.</td><td>معمولاً در <code dir="ltr">.gitignore</code></td></tr>
    <tr><td><code dir="ltr">.env.example</code></td><td>نام متغیرها و مقدارهای نمونهٔ غیرحساس برای راه‌اندازی پروژه.</td><td>قابل <span dir="ltr">Commit</span></td></tr>
  </tbody>
</table>

<a id="health" name="health"></a>

16. <code dir="ltr">depends_on</code> و <code dir="ltr">healthcheck</code>

صرف <span dir="ltr">Running</span> شدن <span dir="ltr">Container</span> همیشه به معنی <span dir="ltr">Ready</span> بودن <span dir="ltr">Service</span> نیست.

<span dir="ltr">Database:</span>

services:
  database:
    image: postgres:17
    environment:
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: apppass
      POSTGRES_DB: appdb
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 5s
      timeout: 3s
      retries: 10

<span dir="ltr">Backend:</span>

services:
  backend:
    depends_on:
      database:
        condition: service_healthy

<pre dir="ltr"><code>database started
      ↓
healthcheck passes
      ↓
database healthy
      ↓
backend starts</code></pre>

<span dir="ltr">Application</span> همچنان باید در برابر قطع موقت <span dir="ltr">Dependency</span>ها رفتار مناسبی داشته باشد؛ <span dir="ltr">Healthcheck</span> جای <span dir="ltr">Retry Logic</span> برنامه را کامل نمی‌گیرد.

<a id="runtime-options" name="runtime-options"></a>

17. <code dir="ltr">restart</code>، <code dir="ltr">command</code> و <code dir="ltr">entrypoint</code>

<span dir="ltr">Restart Policy:</span>

restart: unless-stopped

یا:

restart: on-failure

<span dir="ltr">Override</span> کردن <span dir="ltr">CMD:</span>

command: ["python", "app.py"]

<span dir="ltr">Override</span> کردن <span dir="ltr">ENTRYPOINT:</span>

entrypoint: ["sh", "/app/start.sh"]

اگر <span dir="ltr">Dockerfile</span> رفتار درست دارد، بی‌دلیل <code dir="ltr">command</code> و <code dir="ltr">entrypoint</code> را در <span dir="ltr">Compose</span> تغییر نده.

<a id="real-project" name="real-project"></a>

18. سناریوی پروژه واقعی: <span dir="ltr">Backend</span> + <span dir="ltr">PostgreSQL</span> + <span dir="ltr">Nginx</span>

ساختار:

<pre dir="ltr"><code>project/
├── backend/
│   ├── app.py
│   ├── requirements.txt
│   └── Dockerfile
├── nginx/
│   └── default.conf
├── compose.yaml
├── .env
├── .env.example
└── .gitignore</code></pre>

<span dir="ltr">Backend</span>

<code dir="ltr">backend/app.py</code>:

from flask import Flask
import os

app = Flask(__name__)

@app.get("/")
def index():
    db_host = os.getenv("DB_HOST", "not-set")
    return f"Backend is running. DB_HOST={db_host}"

@app.get("/health")
def health():
    return {"status": "ok"}, 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=4000)

<code dir="ltr">backend/requirements.txt</code>:

<pre dir="ltr"><code>Flask
gunicorn</code></pre>

<code dir="ltr">backend/Dockerfile</code>:

FROM python:3.13-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 4000

CMD ["gunicorn", "--bind", "0.0.0.0:4000", "app:app"]

<span dir="ltr">Nginx</span>

<code dir="ltr">nginx/default.conf</code>:

server {
    listen 80;

    location / {
        proxy_pass http://backend:4000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

<code dir="ltr">backend</code> نام <span dir="ltr">Service</span> است؛ نیازی به <span dir="ltr">IP</span> ثابت نداریم.

<code dir="ltr">.env</code>

APP_PORT=8080
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=change-me

<code dir="ltr">.env.example</code>

APP_PORT=8080
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=replace-me

<code dir="ltr">.gitignore</code>

<pre dir="ltr"><code>.env
__pycache__/
*.pyc</code></pre>

<code dir="ltr">compose.yaml</code>

services:
  backend:
    build:
      context: ./backend
    environment:
      DB_HOST: database
      DB_PORT: "5432"
      DB_NAME: ${POSTGRES_DB}
      DB_USER: ${POSTGRES_USER}
      DB_PASSWORD: ${POSTGRES_PASSWORD}
    depends_on:
      database:
        condition: service_healthy
    networks:
      - app-net

  database:
    image: postgres:17
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - db-data:/var/lib/postgresql/data
    healthcheck:
      test:
        - CMD-SHELL
        - pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}
      interval: 5s
      timeout: 3s
      retries: 10
    networks:
      - app-net

  nginx:
    image: nginx:alpine
    ports:
      - "${APP_PORT}:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - backend
    networks:
      - app-net

volumes:
  db-data:

networks:
  app-net:

معماری:

<pre dir="ltr"><code>Browser
   ↓
Host :8080
   ↓
Nginx :80
   ↓
backend:4000
   ↓
database:5432
   ↓
db-data volume</code></pre>

<span dir="ltr">Validate:</span>

docker compose config

<span dir="ltr">Run:</span>

docker compose up -d --build

بررسی:

docker compose ps
docker compose logs -f
curl http://localhost:8080

<a id="workflow" name="workflow"></a>

19. چرخه کار روزانه با <span dir="ltr">Docker Compose</span>

<span dir="ltr">Validate:</span>

docker compose config

<span dir="ltr">Build:</span>

docker compose build

<span dir="ltr">Build</span> بدون <span dir="ltr">Cache</span>، فقط در صورت نیاز:

docker compose build --no-cache

<span dir="ltr">Up:</span>

docker compose up
docker compose up -d
docker compose up -d --build

<span dir="ltr">Status:</span>

docker compose ps

<span dir="ltr">Logs:</span>

docker compose logs -f
docker compose logs -f backend

<span dir="ltr">Exec:</span>

docker compose exec backend sh

<span dir="ltr">Restart:</span>

docker compose restart backend

<span dir="ltr">Stop/Start:</span>

docker compose stop
docker compose start

<span dir="ltr">Down:</span>

docker compose down

<span dir="ltr">Down</span> همراه <span dir="ltr">Volume</span>ها:

docker compose down -v

<code dir="ltr">-v</code> می‌تواند <span dir="ltr">Data</span> مربوط به <span dir="ltr">Named Volume</span>های پروژه را حذف کند؛ برای <span dir="ltr">Database</span> بدون آگاهی استفاده نکن.

<a id="dev-prod" name="dev-prod"></a>

20. <span dir="ltr">Development</span> در برابر <span dir="ltr">Production</span>

<span dir="ltr">Development</span> معمولاً به این‌ها نیاز دارد:

<span dir="ltr">Source Code</span> با <span dir="ltr">Bind Mount</span>

<span dir="ltr">Auto Reload</span>

<span dir="ltr">Port</span>های <span dir="ltr">Debug</span>

<span dir="ltr">Log</span> بیشتر

نمونه:

services:
  backend:
    build: ./backend
    volumes:
      - ./backend:/app
    command:
      - flask
      - --app
      - app
      - run
      - --host=0.0.0.0
      - --port=4000
      - --debug

<span dir="ltr">Production</span> معمولاً می‌خواهد:

<span dir="ltr">Source</span> داخل <span dir="ltr">Image</span>

<span dir="ltr">Build</span> قابل‌تکرار

<span dir="ltr">Runtime</span> کوچک‌تر

<span dir="ltr">Restart Policy</span> مشخص

<span dir="ltr">Secret Management</span> مناسب

<span dir="ltr">Healthcheck</span>

<span dir="ltr">Port</span>های داخلی غیرضروری <span dir="ltr">Publish</span> نشوند

اصل:

<pre dir="ltr"><code>Development convenience
≠
Production configuration</code></pre>

<a id="debug" name="debug"></a>

21. <span dir="ltr">Debug</span> و <span dir="ltr">Troubleshooting</span>

ترتیب پیشنهادی:

1. <span dir="ltr">Configuration</span>

docker compose config

2. وضعیت <span dir="ltr">Service</span>ها

docker compose ps

3. <span dir="ltr">Log</span>

docker compose logs backend
docker compose logs database

4. داخل <span dir="ltr">Container</span>

docker compose exec backend sh
env

5. <span dir="ltr">DNS</span> داخلی

اگر ابزار مربوطه در <span dir="ltr">Image</span> وجود داشته باشد:

getent hosts database

6. <span dir="ltr">Volume</span>

docker volume ls
docker volume inspect PROJECT_db-data

7. <span dir="ltr">Build</span>

docker compose build --progress=plain backend

سؤال‌های <span dir="ltr">Debug:</span>

<ol dir="rtl">
  <li>آیا <span dir="ltr">Service</span> ساخته و اجرا شده است؟</li>
  <li>آیا <span dir="ltr">Process</span> اصلی هنوز در حال اجراست؟</li>
  <li>آیا <span dir="ltr">Application</span> روی <span dir="ltr">Port</span> درست گوش می‌دهد؟</li>
  <li>آیا همان <span dir="ltr">Port</span> درست روی <span dir="ltr">Host</span> منتشر شده است؟</li>
  <li>آیا نام <span dir="ltr">Service</span> برای ارتباط داخلی درست استفاده شده است؟</li>
  <li>آیا <span dir="ltr">Environment Variable</span>ها مقدار درست دارند؟</li>
  <li>آیا مسیر <span dir="ltr">Volume</span> یا <span dir="ltr">Bind Mount</span> درست است؟</li>
  <li>وضعیت <span dir="ltr">Healthcheck</span> چیست؟</li>
</ol>

<a id="mistakes" name="mistakes"></a>

22. اشتباهات رایج

<code dir="ltr">localhost</code> برای <span dir="ltr">Service</span> دیگر

داخل <span dir="ltr">Backend</span>، <code dir="ltr">localhost</code> یعنی همان <span dir="ltr">Backend Container.</span> برای <span dir="ltr">Database</span> از نام <span dir="ltr">Service</span> مثل <code dir="ltr">database</code> استفاده کن.

<span dir="ltr">Database</span> بدون <span dir="ltr">Volume</span>

اگر <span dir="ltr">Data</span> مهم است <span dir="ltr">Named Volume</span> تعریف کن.

تغییر <span dir="ltr">Source</span> و انتظار تغییر <span dir="ltr">Image</span>

<code dir="ltr">COPY</code> فقط در <span dir="ltr">Build</span> اجرا می‌شود. برای <span dir="ltr">Image</span> جدید:

docker compose up -d --build

یا در <span dir="ltr">Development</span> از <span dir="ltr">Bind Mount</span> استفاده کن.

<code dir="ltr">EXPOSE</code> را <span dir="ltr">Publish</span> فرض کردن

<code dir="ltr">EXPOSE 4000</code> به معنی دسترسی <span dir="ltr">Host</span> نیست. در <span dir="ltr">Compose:</span>

ports:
  - "8080:4000"

<span dir="ltr">Secret</span> داخل <span dir="ltr">YAML</span>

بد:

POSTGRES_PASSWORD: my-real-password

بهتر:

POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

<span dir="ltr">IP</span> ثابت <span dir="ltr">Container</span>

بد:

<pre dir="ltr"><code>172.20.0.4</code></pre>

خوب:

<pre dir="ltr"><code>database</code></pre>

<code dir="ltr">docker compose down -v</code> بدون توجه

ممکن است <span dir="ltr">Volume</span> و <span dir="ltr">Data</span> را حذف کند.

یک <span dir="ltr">Container</span> برای همه‌چیز

در پروژه‌های چندبخشی، مسئولیت‌ها را معمولاً در <span dir="ltr">Service</span>های جدا نگه دار:

<pre dir="ltr"><code>backend
database
reverse proxy
cache</code></pre>

<a id="cheatsheet" name="cheatsheet"></a>

23. <span dir="ltr">Cheat Sheet</span>

<span dir="ltr">Dockerfile</span>

FROM ...
WORKDIR ...
COPY ...
RUN ...
ARG ...
ENV ...
EXPOSE ...
USER ...
CMD [...]
ENTRYPOINT [...]

<span dir="ltr">Build</span>

docker build -t app:1.0.0 .
docker build -f Dockerfile.dev -t app:dev .
docker build --no-cache -t app:test .

<span dir="ltr">Compose</span>

docker compose config
docker compose build
docker compose up
docker compose up -d
docker compose up -d --build
docker compose ps
docker compose logs -f
docker compose logs -f backend
docker compose exec backend sh
docker compose restart backend
docker compose stop
docker compose start
docker compose down

با احتیاط:

docker compose down -v

<a id="next" name="next"></a>

24. آمادگی برای سند <span dir="ltr">YAML</span>

بعد از این سند باید بتوانی یک فایل <span dir="ltr">Compose</span> را از نظر معماری <span dir="ltr">Docker</span> بفهمی. سند سوم لایه‌ی <span dir="ltr">Syntax</span> و قواعد نوشتن <span dir="ltr">YAML</span> را جدا و منظم بررسی می‌کند:

<pre dir="ltr"><code>Mapping
Sequence
Indentation
Quote
.env
services
ports
volumes
networks
depends_on
healthcheck
Override
Validation</code></pre>

</div>