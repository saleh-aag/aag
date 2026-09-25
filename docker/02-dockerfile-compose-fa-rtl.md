<div dir="rtl" align="right">


# Dockerfile و Docker Compose در پروژه‌های واقعی

این سند ادامه‌ی راهنمای Docker است. فرض می‌کنیم مفاهیم `Image`، `Container`، `Port Mapping`، `Volume` و `Network` را می‌دانیم.

هدف این فایل پاسخ به دو سؤال است:

1. چگونه محیط اجرای یک Application را به شکل قابل‌تکرار با `Dockerfile` بسازیم؟
2. چگونه چند Service مرتبط را با `Docker Compose` به عنوان یک پروژه واحد اجرا و مدیریت کنیم؟

---

<a id="toc"></a>
## فهرست مطالب

1. [Dockerfile و Compose چه مسئله‌ای را حل می‌کنند؟](#purpose)
2. [Dockerfile چیست؟](#dockerfile)
3. [دستورهای اصلی Dockerfile](#dockerfile-instructions)
4. [Build Context و .dockerignore](#build-context)
5. [Layer و Build Cache](#cache)
6. [CMD و ENTRYPOINT](#cmd-entrypoint)
7. [COPY و Bind Mount](#copy-bind)
8. [سناریوی Node.js: از Dockerfile تا Image](#node)
9. [سناریوی Flask: Application داخل Container](#flask)
10. [Docker Compose چیست؟](#compose)
11. [ساختار compose.yaml](#compose-structure)
12. [Service و تنظیمات اصلی آن](#service)
13. [Network در Compose](#compose-network)
14. [Volume و Bind Mount در Compose](#compose-storage)
15. [.env، environment و env_file](#compose-env)
16. [depends_on و healthcheck](#health)
17. [restart، command و entrypoint](#runtime-options)
18. [سناریوی پروژه واقعی Backend + PostgreSQL + Nginx](#real-project)
19. [چرخه کار روزانه با Docker Compose](#workflow)
20. [Development در برابر Production](#dev-prod)
21. [Debug و Troubleshooting](#debug)
22. [اشتباهات رایج](#mistakes)
23. [Cheat Sheet](#cheatsheet)
24. [آمادگی برای سند YAML](#next)

---

<a id="purpose"></a>
## 1. Dockerfile و Compose چه مسئله‌ای را حل می‌کنند؟

با Docker خام می‌توانیم Container بسازیم، اما اگر ساخت محیط Application را دستی انجام دهیم، مراحل قابل‌تکرار نیستند.

مثلاً این روند:

</div>

```text
Ubuntu Container
   ↓
apt install ...
   ↓
pip install ...
   ↓
copy source code
   ↓
start app
```

<div dir="rtl" align="right">


اگر فقط در Terminal انجام شود، عضو بعدی تیم دقیقاً نمی‌داند چه کارهایی انجام شده است.

`Dockerfile` مراحل ساخت Image را تبدیل به Code می‌کند:

</div>

```text
Dockerfile
   ↓
docker build
   ↓
Image
```

<div dir="rtl" align="right">


اگر پروژه چند بخش داشته باشد:

</div>

```text
Frontend
Backend
Database
Redis
Nginx
```

<div dir="rtl" align="right">


مدیریت همه‌ی آن‌ها با چندین `docker run` سخت می‌شود. `Docker Compose` تنظیم اجرای کل Stack را در یک فایل تعریف می‌کند.

پس:

</div>

```text
Dockerfile
→ چگونه Image ساخته شود؟

Docker Compose
→ Serviceهای پروژه چگونه کنار هم اجرا شوند؟
```

<div dir="rtl" align="right">


---

<a id="dockerfile"></a>
## 2. Dockerfile چیست؟

`Dockerfile` فایل متنی‌ای است که مراحل ساخت Image را تعریف می‌کند.

نمونه ساده:

</div>

```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY app.py .

RUN pip install flask

EXPOSE 4000

CMD ["python", "app.py"]
```

<div dir="rtl" align="right">


مدل ذهنی:

</div>

```text
Base Image
   ↓
Working Directory
   ↓
Dependencies
   ↓
Application Files
   ↓
Default Runtime Command
   ↓
Final Image
```

<div dir="rtl" align="right">


Build:

</div>

```bash
docker build -t myapp:1.0.0 .
```

<div dir="rtl" align="right">


Run:

</div>

```bash
docker run -p 4000:4000 myapp:1.0.0
```

<div dir="rtl" align="right">


---

<a id="dockerfile-instructions"></a>
## 3. دستورهای اصلی Dockerfile

### `FROM`

</div>

```dockerfile
FROM node:22
```

<div dir="rtl" align="right">


Base Image را تعیین می‌کند.

### `WORKDIR`

</div>

```dockerfile
WORKDIR /app
```

<div dir="rtl" align="right">


مسیر کاری داخل Image را مشخص می‌کند. مدل ذهنی تقریبی آن شبیه `cd /app` است.

### `COPY`

</div>

```dockerfile
COPY app.js .
```

<div dir="rtl" align="right">


اگر `WORKDIR /app` باشد:

</div>

```text
Host ./app.js
     ↓ COPY
Image /app/app.js
```

<div dir="rtl" align="right">


`COPY` اتصال زنده نیست؛ نسخه فایل در زمان Build وارد Image می‌شود.

### `RUN`

</div>

```dockerfile
RUN npm install
```

<div dir="rtl" align="right">


یا:

</div>

```dockerfile
RUN pip install -r requirements.txt
```

<div dir="rtl" align="right">


در زمان Build اجرا می‌شود.

</div>

```text
RUN
→ Build time

CMD / ENTRYPOINT
→ Container runtime
```

<div dir="rtl" align="right">


### `ENV`

</div>

```dockerfile
ENV APP_ENV=production
```

<div dir="rtl" align="right">


Environment Variable پیش‌فرض داخل Image.

### `ARG`

</div>

```dockerfile
ARG APP_VERSION=dev
```

<div dir="rtl" align="right">


برای Build-time variable.

</div>

```bash
docker build \
  --build-arg APP_VERSION=1.2.0 \
  -t myapp:1.2.0 .
```

<div dir="rtl" align="right">

</div>

```text
ARG → Build time
ENV → Image / Runtime environment
```

<div dir="rtl" align="right">


### `EXPOSE`

</div>

```dockerfile
EXPOSE 3000
```

<div dir="rtl" align="right">


Port مورد انتظار Application را بیان می‌کند، اما Port را روی Host Publish نمی‌کند.

Publish واقعی:

</div>

```bash
docker run -p 8080:3000 myapp
```

<div dir="rtl" align="right">


### `CMD`

</div>

```dockerfile
CMD ["node", "app.js"]
```

<div dir="rtl" align="right">


Command پیش‌فرض Container.

### `ENTRYPOINT`

</div>

```dockerfile
ENTRYPOINT ["python", "app.py"]
```

<div dir="rtl" align="right">


Executable اصلی Container را تعریف می‌کند.

### `USER`

</div>

```dockerfile
USER appuser
```

<div dir="rtl" align="right">


در پروژه واقعی اجرای Application با User غیر root معمولاً الگوی امن‌تری است، به شرط این‌که Permissionها درست تنظیم شده باشند.

### `HEALTHCHECK`

</div>

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1
```

<div dir="rtl" align="right">


در بسیاری از پروژه‌ها Healthcheck را در Compose تعریف می‌کنند تا Runtime Configuration کنار بقیه Serviceها باشد.

---

<a id="build-context"></a>
## 4. Build Context و `.dockerignore`

</div>

```bash
docker build -t myapp .
```

<div dir="rtl" align="right">


نقطه آخر یعنی Build Context پوشه فعلی است.

</div>

```text
project/
├── app.js
├── package.json
└── Dockerfile
```

<div dir="rtl" align="right">


وقتی Dockerfile می‌گوید:

</div>

```dockerfile
COPY app.js .
```

<div dir="rtl" align="right">


Docker فایل را از Context پیدا می‌کند.

اگر Context را `./backend` بدهیم:

</div>

```bash
docker build -t backend ./backend
```

<div dir="rtl" align="right">


Dockerfile به فایل‌های خارج از Context دسترسی مستقیم ندارد.

### `.dockerignore`

</div>

```text
node_modules
.git
.env
*.log
__pycache__
venv
dist
```

<div dir="rtl" align="right">


مزایا:

- Build Context کوچک‌تر
- Build سریع‌تر
- کاهش ورود فایل غیرضروری یا Secret به Image

---

<a id="cache"></a>
## 5. Layer و Build Cache

ترتیب Dockerfile روی Cache اثر دارد.

برای Node:

</div>

```dockerfile
FROM node:22
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
```

<div dir="rtl" align="right">


اگر فقط Source Code تغییر کند ولی فایل‌های Package تغییر نکنند، Docker ممکن است Layer نصب Dependencyها را reuse کند.

برای Python:

</div>

```dockerfile
FROM python:3.13-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
```

<div dir="rtl" align="right">


اصل ذهنی:

</div>

```text
Dependency manifest
→ قبل از Source Code
```

<div dir="rtl" align="right">


تا تغییر کوچک در Source باعث نصب دوباره همه Dependencyها نشود.

---

<a id="cmd-entrypoint"></a>
## 6. `CMD` و `ENTRYPOINT`

### CMD

</div>

```dockerfile
CMD ["node", "app.js"]
```

<div dir="rtl" align="right">


Command پیش‌فرض است و با Command انتهای `docker run` راحت Override می‌شود:

</div>

```bash
docker run myapp bash
```

<div dir="rtl" align="right">


### ENTRYPOINT

</div>

```dockerfile
ENTRYPOINT ["python", "app.py"]
```

<div dir="rtl" align="right">


Executable اصلی را ثابت‌تر می‌کند.

ترکیب:

</div>

```dockerfile
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "3000"]
```

<div dir="rtl" align="right">


مدل Runtime:

</div>

```text
python app.py --port 3000
```

<div dir="rtl" align="right">


برای شروع:

</div>

```text
CMD
→ پیش‌فرض قابل Override

ENTRYPOINT
→ هسته Command

ENTRYPOINT + CMD
→ Executable ثابت + default arguments
```

<div dir="rtl" align="right">


---

<a id="copy-bind"></a>
## 7. `COPY` و Bind Mount

### COPY

</div>

```dockerfile
COPY app.js .
```

<div dir="rtl" align="right">

</div>

```text
Host app.js
   ↓ docker build
Image /app/app.js
```

<div dir="rtl" align="right">


بعد از Build، تغییر Host وارد Image قبلی نمی‌شود.

### Bind Mount

</div>

```bash
docker run \
  -v "$PWD/app.js":/app/app.js \
  myapp
```

<div dir="rtl" align="right">

</div>

```text
Host app.js
     ⇅
Container /app/app.js
```

<div dir="rtl" align="right">


پس:

</div>

```text
COPY = Snapshot در Build time
Bind Mount = اتصال Runtime به فایل Host
```

<div dir="rtl" align="right">


اگر کل `/app` را Mount کنی:

</div>

```bash
-v "$PWD":/app
```

<div dir="rtl" align="right">


محتویات قبلی `/app` داخل Image ممکن است پشت Mount پنهان شوند. برای همین مسیر Mount باید آگاهانه انتخاب شود.

---

<a id="node"></a>
## 8. سناریوی Node.js: از Dockerfile تا Image

ساختار:

</div>

```text
node-app/
├── app.js
├── package.json
├── package-lock.json
├── Dockerfile
└── .dockerignore
```

<div dir="rtl" align="right">


`app.js`:

</div>

```javascript
const express = require("express");
const app = express();

app.get("/", (req, res) => {
  res.send("Hello from Docker");
});

app.listen(3000, "0.0.0.0", () => {
  console.log("Listening on port 3000");
});
```

<div dir="rtl" align="right">


`Dockerfile`:

</div>

```dockerfile
FROM node:22

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

EXPOSE 3000

CMD ["node", "app.js"]
```

<div dir="rtl" align="right">


`.dockerignore`:

</div>

```text
node_modules
npm-debug.log
.git
.env
```

<div dir="rtl" align="right">


Build:

</div>

```bash
docker build -t node-app:1.0.0 .
```

<div dir="rtl" align="right">


Run:

</div>

```bash
docker run \
  --name node-app \
  -p 2000:3000 \
  node-app:1.0.0
```

<div dir="rtl" align="right">


Test:

</div>

```bash
curl http://localhost:2000
```

<div dir="rtl" align="right">

</div>

```text
curl
 ↓
Host :2000
 ↓
Container :3000
 ↓
node app.js
```

<div dir="rtl" align="right">


برای دیدن تفاوت COPY و Bind Mount، بعد از Build متن Response را روی Host تغییر بده. بدون Build مجدد Image همان نسخه قدیمی را دارد؛ با Mount کردن `app.js` نسخه فعلی Host دیده می‌شود.

---

<a id="flask"></a>
## 9. سناریوی Flask: Application داخل Container

ساختار:

</div>

```text
flask-app/
├── app.py
├── requirements.txt
├── Dockerfile
└── .dockerignore
```

<div dir="rtl" align="right">


`app.py`:

</div>

```python
from flask import Flask

app = Flask(__name__)

@app.get("/")
def hello():
    return "Hello World"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=4000)
```

<div dir="rtl" align="right">


`requirements.txt`:

</div>

```text
Flask
```

<div dir="rtl" align="right">


`Dockerfile`:

</div>

```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 4000

CMD ["python", "app.py"]
```

<div dir="rtl" align="right">


Build:

</div>

```bash
docker build -t flask-app:1.0.0 .
```

<div dir="rtl" align="right">


Run:

</div>

```bash
docker run \
  --name flask-app \
  -p 4000:4000 \
  flask-app:1.0.0
```

<div dir="rtl" align="right">


دو نکته:

</div>

```text
Application داخل Container
→ روی 0.0.0.0 گوش می‌دهد

Port Mapping
→ Host 4000 را به Container 4000 وصل می‌کند
```

<div dir="rtl" align="right">


---

<a id="compose"></a>
## 10. Docker Compose چیست؟

در پروژه واقعی معمولاً فقط یک Container نداریم:

</div>

```text
Application
├── Nginx
├── Backend
├── PostgreSQL
└── Redis
```

<div dir="rtl" align="right">


بدون Compose باید Network، Volume و `docker run`های مختلف را جدا مدیریت کنیم.

Compose معماری Runtime پروژه را در YAML تعریف می‌کند:

</div>

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

<div dir="rtl" align="right">


اجرا:

</div>

```bash
docker compose up -d
```

<div dir="rtl" align="right">


پس:

</div>

```text
Dockerfile → Image definition
compose.yaml → Runtime topology
```

<div dir="rtl" align="right">


Compose جایگزین Docker نیست؛ روی همان Docker Engine کار می‌کند.

---

<a id="compose-structure"></a>
## 11. ساختار `compose.yaml`

سه بخش پرتکرار:

</div>

```yaml
services:

volumes:

networks:
```

<div dir="rtl" align="right">


مثال:

</div>

```yaml
services:
  backend:
    image: my-backend:1.0.0

  database:
    image: postgres:17

volumes:
  db-data:

networks:
  app-net:
```

<div dir="rtl" align="right">


`Service` یعنی یک بخش از Application که Compose آن را مدیریت می‌کند.

---

<a id="service"></a>
## 12. Service و تنظیمات اصلی آن

### `image`

</div>

```yaml
services:
  nginx:
    image: nginx:alpine
```

<div dir="rtl" align="right">


### `build`

</div>

```yaml
services:
  backend:
    build: ./backend
```

<div dir="rtl" align="right">


فرم کامل‌تر:

</div>

```yaml
services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
```

<div dir="rtl" align="right">


### `ports`

</div>

```yaml
ports:
  - "8080:80"
```

<div dir="rtl" align="right">


یعنی Host `8080` به Container `80`.

### `environment`

</div>

```yaml
environment:
  APP_ENV: development
  DB_HOST: database
```

<div dir="rtl" align="right">


### `container_name`

Compose امکان نام ثابت دارد:

</div>

```yaml
container_name: backend
```

<div dir="rtl" align="right">


اما معمولاً بهتر است بی‌دلیل به نام ثابت وابسته نشویم؛ خود Compose Naming و Service Discovery را مدیریت می‌کند.

---

<a id="compose-network"></a>
## 13. Network در Compose

Compose معمولاً برای Project یک Network پیش‌فرض می‌سازد.

</div>

```text
backend
 database
 nginx
   │
   └── project_default
```

<div dir="rtl" align="right">


Serviceها با نام Service همدیگر را پیدا می‌کنند.

اگر Service دیتابیس `database` باشد:

</div>

```text
database:5432
```

<div dir="rtl" align="right">


اشتباه رایج داخل Backend:

</div>

```text
DB_HOST=localhost
```

<div dir="rtl" align="right">


`localhost` داخل Backend یعنی خود Backend Container، نه Database.

درست:

</div>

```text
DB_HOST=database
```

<div dir="rtl" align="right">


Network اختصاصی:

</div>

```yaml
services:
  backend:
    networks:
      - app-net

  database:
    networks:
      - app-net

networks:
  app-net:
```

<div dir="rtl" align="right">


---

<a id="compose-storage"></a>
## 14. Volume و Bind Mount در Compose

Named Volume:

</div>

```yaml
services:
  database:
    image: postgres:17
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

<div dir="rtl" align="right">


برای Database Data مناسب است.

Bind Mount در Development:

</div>

```yaml
services:
  backend:
    build: ./backend
    volumes:
      - ./backend:/app
```

<div dir="rtl" align="right">


مدل پیشنهادی برای شروع:

</div>

```text
Source Code در Development → Bind Mount
Database Data → Named Volume
```

<div dir="rtl" align="right">


---

<a id="compose-env"></a>
## 15. `.env`، `environment` و `env_file`

این سه مفهوم را قاطی نکن.

### `environment`

</div>

```yaml
services:
  backend:
    environment:
      APP_ENV: development
      DB_HOST: database
```

<div dir="rtl" align="right">


### Interpolation با `.env`

`.env`:

</div>

```ini
APP_PORT=3000
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=change-me
```

<div dir="rtl" align="right">


Compose:

</div>

```yaml
ports:
  - "${APP_PORT}:3000"
```

<div dir="rtl" align="right">


Compose مقدار `${APP_PORT}` را هنگام پردازش فایل جایگزین می‌کند.

### `env_file`

</div>

```yaml
services:
  backend:
    env_file:
      - .env
```

<div dir="rtl" align="right">


Variableهای فایل وارد Environment Container می‌شوند.

برای Repository:

</div>

```text
.env → مقدار واقعی → در .gitignore
.env.example → نام Variableها و مقدار نمونه → قابل Commit
```

<div dir="rtl" align="right">


---

<a id="health"></a>
## 16. `depends_on` و `healthcheck`

صرف Running شدن Container همیشه به معنی Ready بودن Service نیست.

Database:

</div>

```yaml
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
```

<div dir="rtl" align="right">


Backend:

</div>

```yaml
services:
  backend:
    depends_on:
      database:
        condition: service_healthy
```

<div dir="rtl" align="right">

</div>

```text
database started
      ↓
healthcheck passes
      ↓
database healthy
      ↓
backend starts
```

<div dir="rtl" align="right">


Application همچنان باید در برابر قطع موقت Dependencyها رفتار مناسبی داشته باشد؛ Healthcheck جای Retry Logic برنامه را کامل نمی‌گیرد.

---

<a id="runtime-options"></a>
## 17. `restart`، `command` و `entrypoint`

Restart Policy:

</div>

```yaml
restart: unless-stopped
```

<div dir="rtl" align="right">


یا:

</div>

```yaml
restart: on-failure
```

<div dir="rtl" align="right">


Override کردن CMD:

</div>

```yaml
command: ["python", "app.py"]
```

<div dir="rtl" align="right">


Override کردن ENTRYPOINT:

</div>

```yaml
entrypoint: ["sh", "/app/start.sh"]
```

<div dir="rtl" align="right">


اگر Dockerfile رفتار درست دارد، بی‌دلیل `command` و `entrypoint` را در Compose تغییر نده.

---

<a id="real-project"></a>
## 18. سناریوی پروژه واقعی: Backend + PostgreSQL + Nginx

ساختار:

</div>

```text
project/
├── backend/
│   ├── app.py
│   ├── requirements.txt
│   └── Dockerfile
├── nginx/
│   └── default.conf
├── compose.yaml
├── .env
├── .env.example
└── .gitignore
```

<div dir="rtl" align="right">


### Backend

`backend/app.py`:

</div>

```python
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
```

<div dir="rtl" align="right">


`backend/requirements.txt`:

</div>

```text
Flask
gunicorn
```

<div dir="rtl" align="right">


`backend/Dockerfile`:

</div>

```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 4000

CMD ["gunicorn", "--bind", "0.0.0.0:4000", "app:app"]
```

<div dir="rtl" align="right">


### Nginx

`nginx/default.conf`:

</div>

```nginx
server {
    listen 80;

    location / {
        proxy_pass http://backend:4000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

<div dir="rtl" align="right">


`backend` نام Service است؛ نیازی به IP ثابت نداریم.

### `.env`

</div>

```ini
APP_PORT=8080
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=change-me
```

<div dir="rtl" align="right">


### `.env.example`

</div>

```ini
APP_PORT=8080
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=replace-me
```

<div dir="rtl" align="right">


### `.gitignore`

</div>

```text
.env
__pycache__/
*.pyc
```

<div dir="rtl" align="right">


### `compose.yaml`

</div>

```yaml
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
```

<div dir="rtl" align="right">


معماری:

</div>

```text
Browser
   ↓
Host :8080
   ↓
Nginx :80
   ↓
backend:4000
   ↓
database:5432
   ↓
db-data volume
```

<div dir="rtl" align="right">


Validate:

</div>

```bash
docker compose config
```

<div dir="rtl" align="right">


Run:

</div>

```bash
docker compose up -d --build
```

<div dir="rtl" align="right">


بررسی:

</div>

```bash
docker compose ps
docker compose logs -f
curl http://localhost:8080
```

<div dir="rtl" align="right">


---

<a id="workflow"></a>
## 19. چرخه کار روزانه با Docker Compose

Validate:

</div>

```bash
docker compose config
```

<div dir="rtl" align="right">


Build:

</div>

```bash
docker compose build
```

<div dir="rtl" align="right">


Build بدون Cache، فقط در صورت نیاز:

</div>

```bash
docker compose build --no-cache
```

<div dir="rtl" align="right">


Up:

</div>

```bash
docker compose up
docker compose up -d
docker compose up -d --build
```

<div dir="rtl" align="right">


Status:

</div>

```bash
docker compose ps
```

<div dir="rtl" align="right">


Logs:

</div>

```bash
docker compose logs -f
docker compose logs -f backend
```

<div dir="rtl" align="right">


Exec:

</div>

```bash
docker compose exec backend sh
```

<div dir="rtl" align="right">


Restart:

</div>

```bash
docker compose restart backend
```

<div dir="rtl" align="right">


Stop/Start:

</div>

```bash
docker compose stop
docker compose start
```

<div dir="rtl" align="right">


Down:

</div>

```bash
docker compose down
```

<div dir="rtl" align="right">


Down همراه Volumeها:

</div>

```bash
docker compose down -v
```

<div dir="rtl" align="right">


`-v` می‌تواند Data مربوط به Named Volumeهای پروژه را حذف کند؛ برای Database بدون آگاهی استفاده نکن.

---

<a id="dev-prod"></a>
## 20. Development در برابر Production

Development معمولاً به این‌ها نیاز دارد:

- Source Code با Bind Mount
- Auto Reload
- Portهای Debug
- Log بیشتر

نمونه:

</div>

```yaml
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
```

<div dir="rtl" align="right">


Production معمولاً می‌خواهد:

- Source داخل Image
- Build قابل‌تکرار
- Runtime کوچک‌تر
- Restart Policy مشخص
- Secret Management مناسب
- Healthcheck
- Portهای داخلی غیرضروری Publish نشوند

اصل:

</div>

```text
Development convenience
≠
Production configuration
```

<div dir="rtl" align="right">


---

<a id="debug"></a>
## 21. Debug و Troubleshooting

ترتیب پیشنهادی:

### 1. Configuration

</div>

```bash
docker compose config
```

<div dir="rtl" align="right">


### 2. وضعیت Serviceها

</div>

```bash
docker compose ps
```

<div dir="rtl" align="right">


### 3. Log

</div>

```bash
docker compose logs backend
docker compose logs database
```

<div dir="rtl" align="right">


### 4. داخل Container

</div>

```bash
docker compose exec backend sh
env
```

<div dir="rtl" align="right">


### 5. DNS داخلی

اگر ابزار مربوطه در Image وجود داشته باشد:

</div>

```bash
getent hosts database
```

<div dir="rtl" align="right">


### 6. Volume

</div>

```bash
docker volume ls
docker volume inspect PROJECT_db-data
```

<div dir="rtl" align="right">


### 7. Build

</div>

```bash
docker compose build --progress=plain backend
```

<div dir="rtl" align="right">


سؤال‌های Debug:

</div>

```text
Service ساخته شده؟
Process اصلی Running است؟
Application روی Port درست Listen می‌کند؟
Port درست Publish شده؟
Service Name درست است؟
Environment Variable مقدار درست دارد؟
Volume path درست است؟
Healthcheck چه وضعیتی دارد؟
```

<div dir="rtl" align="right">


---

<a id="mistakes"></a>
## 22. اشتباهات رایج

### `localhost` برای Service دیگر

داخل Backend، `localhost` یعنی همان Backend Container. برای Database از نام Service مثل `database` استفاده کن.

### Database بدون Volume

اگر Data مهم است Named Volume تعریف کن.

### تغییر Source و انتظار تغییر Image

`COPY` فقط در Build اجرا می‌شود. برای Image جدید:

</div>

```bash
docker compose up -d --build
```

<div dir="rtl" align="right">


یا در Development از Bind Mount استفاده کن.

### `EXPOSE` را Publish فرض کردن

`EXPOSE 4000` به معنی دسترسی Host نیست. در Compose:

</div>

```yaml
ports:
  - "8080:4000"
```

<div dir="rtl" align="right">


### Secret داخل YAML

بد:

</div>

```yaml
POSTGRES_PASSWORD: my-real-password
```

<div dir="rtl" align="right">


بهتر:

</div>

```yaml
POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
```

<div dir="rtl" align="right">


### IP ثابت Container

بد:

</div>

```text
172.20.0.4
```

<div dir="rtl" align="right">


خوب:

</div>

```text
database
```

<div dir="rtl" align="right">


### `docker compose down -v` بدون توجه

ممکن است Volume و Data را حذف کند.

### یک Container برای همه‌چیز

در پروژه‌های چندبخشی، مسئولیت‌ها را معمولاً در Serviceهای جدا نگه دار:

</div>

```text
backend
database
reverse proxy
cache
```

<div dir="rtl" align="right">


---

<a id="cheatsheet"></a>
## 23. Cheat Sheet

### Dockerfile

</div>

```dockerfile
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
```

<div dir="rtl" align="right">


### Build

</div>

```bash
docker build -t app:1.0.0 .
docker build -f Dockerfile.dev -t app:dev .
docker build --no-cache -t app:test .
```

<div dir="rtl" align="right">


### Compose

</div>

```bash
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
```

<div dir="rtl" align="right">


با احتیاط:

</div>

```bash
docker compose down -v
```

<div dir="rtl" align="right">


---

<a id="next"></a>
## 24. آمادگی برای سند YAML

بعد از این سند باید بتوانی یک فایل Compose را از نظر معماری Docker بفهمی. سند سوم لایه‌ی Syntax و قواعد نوشتن YAML را جدا و منظم بررسی می‌کند:

</div>

```text
Mapping
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
Validation
```

<div dir="rtl" align="right">

</div>