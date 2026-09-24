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

اگر فقط در Terminal انجام شود، عضو بعدی تیم دقیقاً نمی‌داند چه کارهایی انجام شده است.

`Dockerfile` مراحل ساخت Image را تبدیل به Code می‌کند:

```text
Dockerfile
   ↓
docker build
   ↓
Image
```

اگر پروژه چند بخش داشته باشد:

```text
Frontend
Backend
Database
Redis
Nginx
```

مدیریت همه‌ی آن‌ها با چندین `docker run` سخت می‌شود. `Docker Compose` تنظیم اجرای کل Stack را در یک فایل تعریف می‌کند.

پس:

```text
Dockerfile
→ چگونه Image ساخته شود؟

Docker Compose
→ Serviceهای پروژه چگونه کنار هم اجرا شوند؟
```

---

<a id="dockerfile"></a>
## 2. Dockerfile چیست؟

`Dockerfile` فایل متنی‌ای است که مراحل ساخت Image را تعریف می‌کند.

نمونه ساده:

```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY app.py .

RUN pip install flask

EXPOSE 4000

CMD ["python", "app.py"]
```

مدل ذهنی:

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

Build:

```bash
docker build -t myapp:1.0.0 .
```

Run:

```bash
docker run -p 4000:4000 myapp:1.0.0
```

---

<a id="dockerfile-instructions"></a>
## 3. دستورهای اصلی Dockerfile

### `FROM`

```dockerfile
FROM node:22
```

Base Image را تعیین می‌کند.

### `WORKDIR`

```dockerfile
WORKDIR /app
```

مسیر کاری داخل Image را مشخص می‌کند. مدل ذهنی تقریبی آن شبیه `cd /app` است.

### `COPY`

```dockerfile
COPY app.js .
```

اگر `WORKDIR /app` باشد:

```text
Host ./app.js
     ↓ COPY
Image /app/app.js
```

`COPY` اتصال زنده نیست؛ نسخه فایل در زمان Build وارد Image می‌شود.

### `RUN`

```dockerfile
RUN npm install
```

یا:

```dockerfile
RUN pip install -r requirements.txt
```

در زمان Build اجرا می‌شود.

```text
RUN
→ Build time

CMD / ENTRYPOINT
→ Container runtime
```

### `ENV`

```dockerfile
ENV APP_ENV=production
```

Environment Variable پیش‌فرض داخل Image.

### `ARG`

```dockerfile
ARG APP_VERSION=dev
```

برای Build-time variable.

```bash
docker build \
  --build-arg APP_VERSION=1.2.0 \
  -t myapp:1.2.0 .
```

```text
ARG → Build time
ENV → Image / Runtime environment
```

### `EXPOSE`

```dockerfile
EXPOSE 3000
```

Port مورد انتظار Application را بیان می‌کند، اما Port را روی Host Publish نمی‌کند.

Publish واقعی:

```bash
docker run -p 8080:3000 myapp
```

### `CMD`

```dockerfile
CMD ["node", "app.js"]
```

Command پیش‌فرض Container.

### `ENTRYPOINT`

```dockerfile
ENTRYPOINT ["python", "app.py"]
```

Executable اصلی Container را تعریف می‌کند.

### `USER`

```dockerfile
USER appuser
```

در پروژه واقعی اجرای Application با User غیر root معمولاً الگوی امن‌تری است، به شرط این‌که Permissionها درست تنظیم شده باشند.

### `HEALTHCHECK`

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1
```

در بسیاری از پروژه‌ها Healthcheck را در Compose تعریف می‌کنند تا Runtime Configuration کنار بقیه Serviceها باشد.

---

<a id="build-context"></a>
## 4. Build Context و `.dockerignore`

```bash
docker build -t myapp .
```

نقطه آخر یعنی Build Context پوشه فعلی است.

```text
project/
├── app.js
├── package.json
└── Dockerfile
```

وقتی Dockerfile می‌گوید:

```dockerfile
COPY app.js .
```

Docker فایل را از Context پیدا می‌کند.

اگر Context را `./backend` بدهیم:

```bash
docker build -t backend ./backend
```

Dockerfile به فایل‌های خارج از Context دسترسی مستقیم ندارد.

### `.dockerignore`

```text
node_modules
.git
.env
*.log
__pycache__
venv
dist
```

مزایا:

- Build Context کوچک‌تر
- Build سریع‌تر
- کاهش ورود فایل غیرضروری یا Secret به Image

---

<a id="cache"></a>
## 5. Layer و Build Cache

ترتیب Dockerfile روی Cache اثر دارد.

برای Node:

```dockerfile
FROM node:22
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
```

اگر فقط Source Code تغییر کند ولی فایل‌های Package تغییر نکنند، Docker ممکن است Layer نصب Dependencyها را reuse کند.

برای Python:

```dockerfile
FROM python:3.13-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
```

اصل ذهنی:

```text
Dependency manifest
→ قبل از Source Code
```

تا تغییر کوچک در Source باعث نصب دوباره همه Dependencyها نشود.

---

<a id="cmd-entrypoint"></a>
## 6. `CMD` و `ENTRYPOINT`

### CMD

```dockerfile
CMD ["node", "app.js"]
```

Command پیش‌فرض است و با Command انتهای `docker run` راحت Override می‌شود:

```bash
docker run myapp bash
```

### ENTRYPOINT

```dockerfile
ENTRYPOINT ["python", "app.py"]
```

Executable اصلی را ثابت‌تر می‌کند.

ترکیب:

```dockerfile
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "3000"]
```

مدل Runtime:

```text
python app.py --port 3000
```

برای شروع:

```text
CMD
→ پیش‌فرض قابل Override

ENTRYPOINT
→ هسته Command

ENTRYPOINT + CMD
→ Executable ثابت + default arguments
```

---

<a id="copy-bind"></a>
## 7. `COPY` و Bind Mount

### COPY

```dockerfile
COPY app.js .
```

```text
Host app.js
   ↓ docker build
Image /app/app.js
```

بعد از Build، تغییر Host وارد Image قبلی نمی‌شود.

### Bind Mount

```bash
docker run \
  -v "$PWD/app.js":/app/app.js \
  myapp
```

```text
Host app.js
     ⇅
Container /app/app.js
```

پس:

```text
COPY = Snapshot در Build time
Bind Mount = اتصال Runtime به فایل Host
```

اگر کل `/app` را Mount کنی:

```bash
-v "$PWD":/app
```

محتویات قبلی `/app` داخل Image ممکن است پشت Mount پنهان شوند. برای همین مسیر Mount باید آگاهانه انتخاب شود.

---

<a id="node"></a>
## 8. سناریوی Node.js: از Dockerfile تا Image

ساختار:

```text
node-app/
├── app.js
├── package.json
├── package-lock.json
├── Dockerfile
└── .dockerignore
```

`app.js`:

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

`Dockerfile`:

```dockerfile
FROM node:22

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

EXPOSE 3000

CMD ["node", "app.js"]
```

`.dockerignore`:

```text
node_modules
npm-debug.log
.git
.env
```

Build:

```bash
docker build -t node-app:1.0.0 .
```

Run:

```bash
docker run \
  --name node-app \
  -p 2000:3000 \
  node-app:1.0.0
```

Test:

```bash
curl http://localhost:2000
```

```text
curl
 ↓
Host :2000
 ↓
Container :3000
 ↓
node app.js
```

برای دیدن تفاوت COPY و Bind Mount، بعد از Build متن Response را روی Host تغییر بده. بدون Build مجدد Image همان نسخه قدیمی را دارد؛ با Mount کردن `app.js` نسخه فعلی Host دیده می‌شود.

---

<a id="flask"></a>
## 9. سناریوی Flask: Application داخل Container

ساختار:

```text
flask-app/
├── app.py
├── requirements.txt
├── Dockerfile
└── .dockerignore
```

`app.py`:

```python
from flask import Flask

app = Flask(__name__)

@app.get("/")
def hello():
    return "Hello World"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=4000)
```

`requirements.txt`:

```text
Flask
```

`Dockerfile`:

```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 4000

CMD ["python", "app.py"]
```

Build:

```bash
docker build -t flask-app:1.0.0 .
```

Run:

```bash
docker run \
  --name flask-app \
  -p 4000:4000 \
  flask-app:1.0.0
```

دو نکته:

```text
Application داخل Container
→ روی 0.0.0.0 گوش می‌دهد

Port Mapping
→ Host 4000 را به Container 4000 وصل می‌کند
```

---

<a id="compose"></a>
## 10. Docker Compose چیست؟

در پروژه واقعی معمولاً فقط یک Container نداریم:

```text
Application
├── Nginx
├── Backend
├── PostgreSQL
└── Redis
```

بدون Compose باید Network، Volume و `docker run`های مختلف را جدا مدیریت کنیم.

Compose معماری Runtime پروژه را در YAML تعریف می‌کند:

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

اجرا:

```bash
docker compose up -d
```

پس:

```text
Dockerfile → Image definition
compose.yaml → Runtime topology
```

Compose جایگزین Docker نیست؛ روی همان Docker Engine کار می‌کند.

---

<a id="compose-structure"></a>
## 11. ساختار `compose.yaml`

سه بخش پرتکرار:

```yaml
services:

volumes:

networks:
```

مثال:

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

`Service` یعنی یک بخش از Application که Compose آن را مدیریت می‌کند.

---

<a id="service"></a>
## 12. Service و تنظیمات اصلی آن

### `image`

```yaml
services:
  nginx:
    image: nginx:alpine
```

### `build`

```yaml
services:
  backend:
    build: ./backend
```

فرم کامل‌تر:

```yaml
services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
```

### `ports`

```yaml
ports:
  - "8080:80"
```

یعنی Host `8080` به Container `80`.

### `environment`

```yaml
environment:
  APP_ENV: development
  DB_HOST: database
```

### `container_name`

Compose امکان نام ثابت دارد:

```yaml
container_name: backend
```

اما معمولاً بهتر است بی‌دلیل به نام ثابت وابسته نشویم؛ خود Compose Naming و Service Discovery را مدیریت می‌کند.

---

<a id="compose-network"></a>
## 13. Network در Compose

Compose معمولاً برای Project یک Network پیش‌فرض می‌سازد.

```text
backend
 database
 nginx
   │
   └── project_default
```

Serviceها با نام Service همدیگر را پیدا می‌کنند.

اگر Service دیتابیس `database` باشد:

```text
database:5432
```

اشتباه رایج داخل Backend:

```text
DB_HOST=localhost
```

`localhost` داخل Backend یعنی خود Backend Container، نه Database.

درست:

```text
DB_HOST=database
```

Network اختصاصی:

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

---

<a id="compose-storage"></a>
## 14. Volume و Bind Mount در Compose

Named Volume:

```yaml
services:
  database:
    image: postgres:17
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

برای Database Data مناسب است.

Bind Mount در Development:

```yaml
services:
  backend:
    build: ./backend
    volumes:
      - ./backend:/app
```

مدل پیشنهادی برای شروع:

```text
Source Code در Development → Bind Mount
Database Data → Named Volume
```

---

<a id="compose-env"></a>
## 15. `.env`، `environment` و `env_file`

این سه مفهوم را قاطی نکن.

### `environment`

```yaml
services:
  backend:
    environment:
      APP_ENV: development
      DB_HOST: database
```

### Interpolation با `.env`

`.env`:

```ini
APP_PORT=3000
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=change-me
```

Compose:

```yaml
ports:
  - "${APP_PORT}:3000"
```

Compose مقدار `${APP_PORT}` را هنگام پردازش فایل جایگزین می‌کند.

### `env_file`

```yaml
services:
  backend:
    env_file:
      - .env
```

Variableهای فایل وارد Environment Container می‌شوند.

برای Repository:

```text
.env → مقدار واقعی → در .gitignore
.env.example → نام Variableها و مقدار نمونه → قابل Commit
```

---

<a id="health"></a>
## 16. `depends_on` و `healthcheck`

صرف Running شدن Container همیشه به معنی Ready بودن Service نیست.

Database:

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

Backend:

```yaml
services:
  backend:
    depends_on:
      database:
        condition: service_healthy
```

```text
database started
      ↓
healthcheck passes
      ↓
database healthy
      ↓
backend starts
```

Application همچنان باید در برابر قطع موقت Dependencyها رفتار مناسبی داشته باشد؛ Healthcheck جای Retry Logic برنامه را کامل نمی‌گیرد.

---

<a id="runtime-options"></a>
## 17. `restart`، `command` و `entrypoint`

Restart Policy:

```yaml
restart: unless-stopped
```

یا:

```yaml
restart: on-failure
```

Override کردن CMD:

```yaml
command: ["python", "app.py"]
```

Override کردن ENTRYPOINT:

```yaml
entrypoint: ["sh", "/app/start.sh"]
```

اگر Dockerfile رفتار درست دارد، بی‌دلیل `command` و `entrypoint` را در Compose تغییر نده.

---

<a id="real-project"></a>
## 18. سناریوی پروژه واقعی: Backend + PostgreSQL + Nginx

ساختار:

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

### Backend

`backend/app.py`:

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

`backend/requirements.txt`:

```text
Flask
gunicorn
```

`backend/Dockerfile`:

```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 4000

CMD ["gunicorn", "--bind", "0.0.0.0:4000", "app:app"]
```

### Nginx

`nginx/default.conf`:

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

`backend` نام Service است؛ نیازی به IP ثابت نداریم.

### `.env`

```ini
APP_PORT=8080
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=change-me
```

### `.env.example`

```ini
APP_PORT=8080
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=replace-me
```

### `.gitignore`

```text
.env
__pycache__/
*.pyc
```

### `compose.yaml`

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

معماری:

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

Validate:

```bash
docker compose config
```

Run:

```bash
docker compose up -d --build
```

بررسی:

```bash
docker compose ps
docker compose logs -f
curl http://localhost:8080
```

---

<a id="workflow"></a>
## 19. چرخه کار روزانه با Docker Compose

Validate:

```bash
docker compose config
```

Build:

```bash
docker compose build
```

Build بدون Cache، فقط در صورت نیاز:

```bash
docker compose build --no-cache
```

Up:

```bash
docker compose up
docker compose up -d
docker compose up -d --build
```

Status:

```bash
docker compose ps
```

Logs:

```bash
docker compose logs -f
docker compose logs -f backend
```

Exec:

```bash
docker compose exec backend sh
```

Restart:

```bash
docker compose restart backend
```

Stop/Start:

```bash
docker compose stop
docker compose start
```

Down:

```bash
docker compose down
```

Down همراه Volumeها:

```bash
docker compose down -v
```

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

Production معمولاً می‌خواهد:

- Source داخل Image
- Build قابل‌تکرار
- Runtime کوچک‌تر
- Restart Policy مشخص
- Secret Management مناسب
- Healthcheck
- Portهای داخلی غیرضروری Publish نشوند

اصل:

```text
Development convenience
≠
Production configuration
```

---

<a id="debug"></a>
## 21. Debug و Troubleshooting

ترتیب پیشنهادی:

### 1. Configuration

```bash
docker compose config
```

### 2. وضعیت Serviceها

```bash
docker compose ps
```

### 3. Log

```bash
docker compose logs backend
docker compose logs database
```

### 4. داخل Container

```bash
docker compose exec backend sh
env
```

### 5. DNS داخلی

اگر ابزار مربوطه در Image وجود داشته باشد:

```bash
getent hosts database
```

### 6. Volume

```bash
docker volume ls
docker volume inspect PROJECT_db-data
```

### 7. Build

```bash
docker compose build --progress=plain backend
```

سؤال‌های Debug:

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

---

<a id="mistakes"></a>
## 22. اشتباهات رایج

### `localhost` برای Service دیگر

داخل Backend، `localhost` یعنی همان Backend Container. برای Database از نام Service مثل `database` استفاده کن.

### Database بدون Volume

اگر Data مهم است Named Volume تعریف کن.

### تغییر Source و انتظار تغییر Image

`COPY` فقط در Build اجرا می‌شود. برای Image جدید:

```bash
docker compose up -d --build
```

یا در Development از Bind Mount استفاده کن.

### `EXPOSE` را Publish فرض کردن

`EXPOSE 4000` به معنی دسترسی Host نیست. در Compose:

```yaml
ports:
  - "8080:4000"
```

### Secret داخل YAML

بد:

```yaml
POSTGRES_PASSWORD: my-real-password
```

بهتر:

```yaml
POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
```

### IP ثابت Container

بد:

```text
172.20.0.4
```

خوب:

```text
database
```

### `docker compose down -v` بدون توجه

ممکن است Volume و Data را حذف کند.

### یک Container برای همه‌چیز

در پروژه‌های چندبخشی، مسئولیت‌ها را معمولاً در Serviceهای جدا نگه دار:

```text
backend
database
reverse proxy
cache
```

---

<a id="cheatsheet"></a>
## 23. Cheat Sheet

### Dockerfile

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

### Build

```bash
docker build -t app:1.0.0 .
docker build -f Dockerfile.dev -t app:dev .
docker build --no-cache -t app:test .
```

### Compose

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

با احتیاط:

```bash
docker compose down -v
```

---

<a id="next"></a>
## 24. آمادگی برای سند YAML

بعد از این سند باید بتوانی یک فایل Compose را از نظر معماری Docker بفهمی. سند سوم لایه‌ی Syntax و قواعد نوشتن YAML را جدا و منظم بررسی می‌کند:

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

</div>
