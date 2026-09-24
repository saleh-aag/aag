<div dir="rtl">

# 🧩 راهنمای جامع <span dir="ltr">Docker Compose</span> به زبان فارسی

از «<span dir="ltr">Docker Compose</span> چیه؟» تا <span dir="ltr">Service</span>، فایل <span dir="ltr">compose.yaml</span>، مدیریت چند <span dir="ltr">Container</span>، <span dir="ltr">Volume</span>، <span dir="ltr">Network</span> و اجرای پروژه‌های واقعی

این راهنما ادامه‌ی مسیر یادگیری <span dir="ltr">Docker</span> است. فرض شده مفاهیم پایه مثل <span dir="ltr">Image</span>، <span dir="ltr">Container</span>، <span dir="ltr">Dockerfile</span> و <span dir="ltr">YAML</span> را در داکیومنت‌های قبلی مطالعه کرده‌ای.

> 💡 **هدف این فایل:** یادگیری اینکه چگونه چند بخش یک پروژه واقعی را با یک فایل پیکربندی مدیریت کنیم؛ نه تکرار مفاهیم پایهٔ <span dir="ltr">Docker</span>.

## 📚 فهرست مطالب

- [🧩 راهنمای جامع Docker Compose به زبان فارسی](#-راهنمای-جامع-docker-compose-به-زبان-فارسی)
  - [📚 فهرست مطالب](#-فهرست-مطالب)
  - [1. 🧩 Docker Compose چیست؟](#1--docker-compose-چیست)
  - [2. 😵 چرا Docker Compose به وجود آمد؟](#2--چرا-docker-compose-به-وجود-آمد)
  - [3. 🧠 جایگاه Docker Compose در معماری Docker](#3--جایگاه-docker-compose-در-معماری-docker)
  - [4. 📄 فایل compose.yaml چیست؟](#4--فایل-composeyaml-چیست)
  - [5. 🏗️ ساختار اصلی Compose](#5-️-ساختار-اصلی-compose)
  - [6. ⚙️ مفهوم Service در Docker Compose](#6-️-مفهوم-service-در-docker-compose)
  - [7. 📝 نوشتن اولین فایل Compose](#7--نوشتن-اولین-فایل-compose)
  - [8. 🚀 دستورهای مهم Docker Compose](#8--دستورهای-مهم-docker-compose)
  - [9. 🌐 Network در Compose](#9--network-در-compose)
  - [10. 💾 Volume در Compose](#10--volume-در-compose)
  - [11. 🌱 Environment Variable و فایل .env](#11--environment-variable-و-فایل-env)
  - [12. 🔥 پروژه واقعی Backend + Database](#12--پروژه-واقعی-backend--database)
  - [13. 🔍 Debug و بررسی خطاها](#13--debug-و-بررسی-خطاها)
  - [14. 🤦 اشتباهات رایج Juniorها](#14--اشتباهات-رایج-juniorها)
  - [15. 🗺️ نقشه راه بعد از Docker Compose](#15-️-نقشه-راه-بعد-از-docker-compose)
    - [🧠 خلاصه نهایی در ۶۰ ثانیه](#-خلاصه-نهایی-در-۶۰-ثانیه)

---

<a id="what-is-compose"></a>
## 1. 🧩 <span dir="ltr">Docker Compose</span> چیست؟

در پروژه‌های واقعی معمولاً یک برنامه فقط یک <span dir="ltr">Container</span> نیست.

مثلاً:

```
Application
├── Frontend
├── Backend API
├── PostgreSQL
├── Redis
└── Nginx
```

هر کدام از این بخش‌ها می‌تواند یک <span dir="ltr">Container</span> جدا باشد.

بدون <span dir="ltr">Docker Compose</span> باید چندین دستور مختلف اجرا کنیم.

اما <span dir="ltr">Compose</span> اجازه می‌دهد کل معماری پروژه را در یک فایل تعریف کنیم.

به زبان ساده:

**<span dir="ltr">Docker Compose</span> = مدیریت چند <span dir="ltr">Container</span> با یک فایل <span dir="ltr">YAML</span>**

<a id="why-compose"></a>
## 2. 😵 چرا <span dir="ltr">Docker Compose</span> به وجود آمد؟

فرض کن بدون <span dir="ltr">Compose</span> بخواهی یک پروژه را اجرا کنی:

```bash
docker network create app-network

docker run database

docker run backend

docker run frontend
```

مشکل‌ها:

- دستورها زیاد می‌شوند.
- احتمال خطا بالا می‌رود.
- انتقال پروژه به تیم دیگر سخت می‌شود.
- معماری پروژه مشخص نیست.

<span dir="ltr">Compose</span> این اطلاعات را تبدیل به یک فایل قابل خواندن می‌کند.

<a id="compose-place"></a>
## 3. 🧠 جایگاه <span dir="ltr">Docker Compose</span> در معماری <span dir="ltr">Docker</span>

مسیر کلی:

```
Dockerfile
      ↓
Image
      ↓
Container
```

اما پروژه‌های واقعی:

```
Frontend Container
Backend Container
Database Container
Redis Container
        ↓
Docker Compose
```

<span dir="ltr">Compose</span> جایگزین <span dir="ltr">Docker</span> نیست.

فقط مدیریت <span dir="ltr">Container</span>های مرتبط را ساده می‌کند.

<a id="compose-file"></a>
## 4. 📄 فایل <span dir="ltr">compose.yaml</span> چیست؟

فایل اصلی <span dir="ltr">Compose</span> معمولاً:

```
compose.yaml
```

است.

فرم قدیمی:

```
docker-compose.yml
```

نیز وجود دارد.

نمونه:

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

<a id="compose-structure"></a>
## 5. 🏗️ ساختار اصلی <span dir="ltr">Compose</span>

سه بخش اصلی:

```
services:
volumes:
networks:
```

**<span dir="ltr">services</span>**

تعریف برنامه‌هایی که باید اجرا شوند.

مثال:

```yaml
services:
  backend:
    image: my-backend

  database:
    image: postgres
```

**<span dir="ltr">volumes</span>**

برای نگهداری داده:

```yaml
volumes:
  postgres-data:
```

**<span dir="ltr">networks</span>**

برای ارتباط بین <span dir="ltr">Service</span>ها:

```
Backend
   |
Database
```

<a id="service"></a>
## 6. ⚙️ مفهوم <span dir="ltr">Service</span> در <span dir="ltr">Docker Compose</span>

<span dir="ltr">Service</span> یعنی یک بخش از <span dir="ltr">Application</span>.

مثلاً:

```yaml
services:
  api:
    image: backend:v1

  db:
    image: postgres
```

اینجا دو <span dir="ltr">Service</span> داریم:

```
api
db
```

هر <span dir="ltr">Service</span> تنظیمات مربوط به اجرای خودش را دارد.

<a id="first-compose"></a>
## 7. 📝 نوشتن اولین فایل <span dir="ltr">Compose</span>

ساخت فایل:

```
compose.yaml
```

محتوا:

```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
```

اجرا:

```bash
docker compose up -d
```

مشاهده:

```bash
docker compose ps
```

خاموش کردن:

```bash
docker compose down
```

<a id="commands"></a>
## 8. 🚀 دستورهای مهم <span dir="ltr">Docker Compose</span>

اجرای پروژه:

```bash
docker compose up
```

اجرا در پس‌زمینه:

```bash
docker compose up -d
```

مشاهده وضعیت:

```bash
docker compose ps
```

مشاهده <span dir="ltr">Log</span>:

```bash
docker compose logs -f
```

ساخت مجدد <span dir="ltr">Image</span>:

```bash
docker compose up --build
```

توقف کامل:

```bash
docker compose down
```

<a id="network"></a>
## 9. 🌐 <span dir="ltr">Network</span> در <span dir="ltr">Compose</span>

<span dir="ltr">Compose</span> معمولاً یک <span dir="ltr">Network</span> داخلی برای پروژه می‌سازد.

مثلاً:

```
app_default

frontend
    |
backend
    |
database
```

<span dir="ltr">Service</span>ها با نام همدیگر را پیدا می‌کنند.

مثلاً:

خوب:

```
database:5432
```

بد:

```
172.18.0.5:5432
```

چون <span dir="ltr">IP</span> ممکن است تغییر کند.

<a id="volume"></a>
## 10. 💾 <span dir="ltr">Volume</span> در <span dir="ltr">Compose</span>

برای داده‌های مهم:

```yaml
services:
  database:
    image: postgres
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

نتیجه:

```
Container
      ↓
Volume
      ↓
Persistent Data
```

برای:

- <span dir="ltr">Database</span>
- <span dir="ltr">Upload</span>
- فایل‌های مهم

ضروری است.

<a id="environment"></a>
## 11. 🌱 <span dir="ltr">Environment Variable</span> و فایل <span dir="ltr">.env</span>

مثال:

```yaml
services:
  database:
    image: postgres
    environment:
      POSTGRES_PASSWORD: password
```

برای پروژه واقعی بهتر است `.env` داشته باشیم.

مثلاً:

```ini
POSTGRES_PASSWORD=my-password
```

و:

```yaml
environment:
  POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
```

<a id="real-project"></a>
## 12. 🔥 پروژه واقعی <span dir="ltr">Backend + Database</span>

مثال:

```yaml
services:
  backend:
    build: .
    ports:
      - "3000:3000"
    environment:
      DB_HOST: database

  database:
    image: postgres
    volumes:
      - data:/var/lib/postgresql/data

volumes:
  data:
```

معماری:

```
Backend
   |
Database
   |
Volume
```

<a id="debug"></a>
## 13. 🔍 <span dir="ltr">Debug</span> و بررسی خطاها

بررسی فایل نهایی:

```bash
docker compose config
```

دیدن <span dir="ltr">Log</span> یک <span dir="ltr">Service</span>:

```bash
docker compose logs backend
```

ورود به <span dir="ltr">Service</span>:

```bash
docker compose exec backend bash
```

<a id="mistakes"></a>
## 14. 🤦 اشتباهات رایج <span dir="ltr">Junior</span>ها

**اشتباه 1 — فکر کنیم <span dir="ltr">Compose</span> جای <span dir="ltr">Docker</span> است**

غلط:

```
Docker Compose = Docker
```

درست:

```
Docker Compose
       |
       ▼
مدیریت بهتر Docker Containerها
```

**اشتباه 2 — <span dir="ltr">Database</span> بدون <span dir="ltr">Volume</span>**

بد:

```yaml
database:
  image: postgres
```

ممکن است داده از بین برود.

**اشتباه 3 — استفاده از <span dir="ltr">IP</span> ثابت**

بد:

```
172.18.0.5
```

خوب:

```
database
```

**اشتباه 4 — بی‌توجهی به فاصله‌های <span dir="ltr">YAML</span>**

<span dir="ltr">YAML</span> به <span dir="ltr">Space</span> حساس است.

<a id="roadmap"></a>
## 15. 🗺️ نقشه راه بعد از <span dir="ltr">Docker Compose</span>

```
Docker
   ↓
Dockerfile
   ↓
Docker Compose
   ↓
CI/CD
   ↓
Docker Security
   ↓
Kubernetes
```

### 🧠 خلاصه نهایی در ۶۰ ثانیه

اگر فقط چند مفهوم را حفظ کنی:

```
Service
    ↓
Container

Volume
    ↓
Data

Network
    ↓
Communication
```

و:

```
compose.yaml
      ↓
docker compose up
      ↓
Running Application Stack
```

<span dir="ltr">Docker Compose</span> یعنی:

**تعریف معماری یک پروژه چند <span dir="ltr">Container</span>‌ای به شکل <span dir="ltr">Code</span>**

🐳 <span dir="ltr">Happy Composing!</span>

</div>