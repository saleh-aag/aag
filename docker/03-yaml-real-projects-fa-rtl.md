<div dir="rtl" align="right">

# قواعد YAML برای Docker Compose و پروژه‌های واقعی DevOps

این سند YAML را به‌عنوان یک زبان مستقل آموزش می‌دهد، اما تمرکز اصلی روی جایی است که یک Junior بیشترین برخورد را با آن دارد: `Docker Compose` و Configuration پروژه‌های واقعی.

هدف این نیست که ده‌ها قابلیت کم‌استفاده YAML را حفظ کنیم. هدف این است که بتوانیم یک فایل واقعی را **بخوانیم، بنویسیم، Validate کنیم و خطایش را پیدا کنیم**.

---

<a id="toc"></a>
## فهرست مطالب

1. [YAML چیست و چه چیزی نیست؟](#what)
2. [Syntax با Schema فرق دارد](#syntax-schema)
3. [سه ساختار اصلی YAML](#structures)
4. [Indentation](#indentation)
5. [Colon و Dash](#colon-dash)
6. [نوع مقدارها و Quote](#values)
7. [Comment](#comment)
8. [متن چندخطی با | و >](#multiline)
9. [Flow Style با {} و []](#flow)
10. [Anchor، Alias و Merge](#anchors)
11. [روش خواندن یک YAML پیچیده](#reading)
12. [ساختار پایه compose.yaml](#compose-base)
13. [services](#services)
14. [build و image](#build-image)
15. [ports](#ports)
16. [environment، .env و env_file](#env)
17. [volumes و انواع Mount](#volumes)
18. [networks](#networks)
19. [depends_on و healthcheck](#health)
20. [restart، command و entrypoint](#runtime)
21. [Secretها را کجا نگذاریم؟](#secrets)
22. [سناریو 1: Nginx تک‌سرویسی](#scenario1)
23. [سناریو 2: Backend + Database](#scenario2)
24. [سناریو 3: Development با Bind Mount](#scenario3)
25. [سناریو 4: Network اختصاصی](#scenario4)
26. [سناریو 5: .env برای Environmentهای مختلف](#scenario5)
27. [سناریو 6: Healthcheck واقعی](#scenario6)
28. [Compose چندفایلی و Override](#override)
29. [Reuse با x- و Anchor](#reuse)
30. [سه نوع خطا: YAML، Compose Schema و Runtime](#error-types)
31. [Validation و Debug](#validation)
32. [Style Guide پیشنهادی](#style)
33. [تمرین مرحله‌ای](#practice)
34. [Cheat Sheet](#cheatsheet)

---

<a id="what"></a>
## 1. YAML چیست و چه چیزی نیست؟

`YAML` روشی برای نوشتن داده ساختاریافته است.

```yaml
name: Amir
age: 22
enabled: true
```

برای انسان:

```text
name    → Amir
age     → 22
enabled → true
```

برای برنامه چیزی شبیه Object / Dictionary است.

YAML خودش Docker را اجرا نمی‌کند، Network نمی‌سازد و Container ایجاد نمی‌کند. فقط Data را با Syntax مشخص بیان می‌کند.

این Data می‌تواند توسط ابزارهای مختلف خوانده شود:

```text
Docker Compose
GitHub Actions
Kubernetes
Ansible
CI/CD systems
```

مهارت اصلی:

```text
YAML syntax
+
schema ابزار
```

---

<a id="syntax-schema"></a>
## 2. Syntax با Schema فرق دارد

این یکی از مهم‌ترین مفاهیم کل سند است.

```yaml
services:
  web:
    image: nginx:alpine
```

YAML فقط ساختار را می‌فهمد:

```text
services
└── web
    └── image: nginx:alpine
```

اما معنی `services` و `image` را Docker Compose تعیین می‌کند.

این YAML هم از نظر Syntax معتبر است:

```yaml
pizza:
  cheese: lots
  olives:
    - black
    - green
```

ولی Compose چنین Schemaای ندارد.

پس یک فایل می‌تواند:

```text
YAML-valid
ولی
Compose-invalid
```

باشد.

مدل خطا:

```text
Layer 1: YAML Syntax
  ↓
indentation, :, -, list, mapping

Layer 2: Tool Schema
  ↓
services, image, ports, volumes ...

Layer 3: Runtime
  ↓
image pull, port conflict, app crash ...
```

---

<a id="structures"></a>
## 3. سه ساختار اصلی YAML

تقریباً بیشتر YAMLهایی که در DevOps می‌بینی از سه نوع اصلی ساخته شده‌اند.

### 3.1. Scalar

```yaml
name: backend
replicas: 2
enabled: true
timeout: 2.5
optional: null
```

### 3.2. Mapping

```yaml
database:
  host: database
  port: 5432
```

```text
database
├── host → database
└── port → 5432
```

### 3.3. Sequence

```yaml
ports:
  - "8080:80"
  - "8443:443"
```

### ترکیب

```yaml
services:
  backend:
    environment:
      APP_ENV: production
    ports:
      - "8080:4000"
```

```text
services                 Mapping
└── backend              Mapping
    ├── environment      Mapping
    │   └── APP_ENV      Scalar
    └── ports            Sequence
        └── "8080:4000"  Scalar
```

اگر بتوانی YAML را به چنین درختی تبدیل کنی، فایل‌های بزرگ خیلی قابل‌فهم‌تر می‌شوند.

---

<a id="indentation"></a>
## 4. Indentation

فاصله ابتدای خط در YAML معنی ساختاری دارد.

در این سند از دو Space برای هر Level استفاده می‌کنیم:

```text
Level 0 → 0 spaces
Level 1 → 2 spaces
Level 2 → 4 spaces
Level 3 → 6 spaces
```

درست:

```yaml
services:
  backend:
    image: my-backend
    environment:
      APP_ENV: production
```

ساختار:

```text
services
└── backend
    ├── image
    └── environment
        └── APP_ENV
```

نامنظم:

```yaml
services:
   backend:
    image: my-backend
```

ممکن است Error یا ساختار اشتباه ایجاد کند.

قاعده عملی:

```text
برای Indentation از Space استفاده کن.
در Editor، Tab را برای YAML به Space تبدیل کن.
```

YAML الزام نمی‌کند همیشه دقیقاً دو Space استفاده شود، اما Consistency مهم است.

---

<a id="colon-dash"></a>
## 5. Colon و Dash

### `:`

Key را از Value جدا می‌کند:

```yaml
name: backend
```

اگر بعد از Colon مقدار مستقیم نباشد:

```yaml
database:
```

یعنی زیر آن Structure دیگری می‌آید:

```yaml
database:
  host: db
  port: 5432
```

### `-`

Dash + Space معمولاً یک عضو List می‌سازد:

```yaml
networks:
  - frontend
  - backend
```

List می‌تواند از Mapping تشکیل شود:

```yaml
servers:
  - name: api-1
    port: 3000

  - name: api-2
    port: 3001
```

قاعده خواندن:

```text
Dash دیدی؟
→ یک item جدید در List شروع شده.
```

---

<a id="values"></a>
## 6. نوع مقدارها و Quote

```yaml
name: backend
replicas: 2
timeout: 1.5
debug: true
cache: false
optional: null
```

| مقدار | نوع تقریبی |
|---|---|
| `backend` | String |
| `2` | Integer |
| `1.5` | Float |
| `true` | Boolean |
| `null` | Null |

برای Valueهایی که باید قطعاً String بمانند، Quote مفید است:

```yaml
version: "1.10"
phone_like: "01234"
answer: "yes"
mode: "on"
```

Port Mapping:

```yaml
ports:
  - "8080:80"
```

Image Tag:

```yaml
image: "myapp:1.4.2"
```

URL و Cron:

```yaml
url: "https://example.com/api"
schedule: "0 3 * * *"
```

Password با `#`:

```yaml
password: "abc#123"
```

قاعده امن:

```text
اگر مقدار شبیه Number، Boolean، Date یا Syntax خاص است
ولی باید String بماند
→ Quote بگذار.
```

---

<a id="comment"></a>
## 7. Comment

```yaml
# Application configuration
name: backend
```

Inline:

```yaml
port: 3000 # internal application port
```

اگر `#` بخشی از Value است Quote بگذار:

```yaml
password: "abc#123"
```

Comment باید Context یا دلیل بدهد، نه این‌که فقط همان خط را تکرار کند.

ضعیف:

```yaml
ports:
  - "8080:80" # port
```

بهتر:

```yaml
ports:
  - "8080:80" # local public entrypoint
```

---

<a id="multiline"></a>
## 8. متن چندخطی با `|` و `>`

### `|`

Line breakها را حفظ می‌کند:

```yaml
message: |
  line one
  line two
  line three
```

### `>`

خط‌ها را بیشتر به شکل متن پیوسته Fold می‌کند:

```yaml
description: >
  This is a long description
  written across multiple lines.
```

در Compose روزمره کمتر لازم می‌شود، اما در CI/CD و Configها ممکن است ببینی.

---

<a id="flow"></a>
## 9. Flow Style با `{}` و `[]`

فرم فشرده:

```yaml
user: {name: Ali, role: backend}
ports: ["80", "443"]
```

فرم Block معمولاً خواناتر است:

```yaml
user:
  name: Ali
  role: backend

ports:
  - "80"
  - "443"
```

Flow Style را وقتی استفاده کن که واقعاً خوانایی بهتر شود.

---

<a id="anchors"></a>
## 10. Anchor، Alias و Merge

برای Reuse:

```yaml
x-common-env: &common-env
  LOG_LEVEL: info
  TZ: UTC
```

استفاده:

```yaml
services:
  api:
    environment:
      <<: *common-env
      APP_NAME: api

  worker:
    environment:
      <<: *common-env
      APP_NAME: worker
```

```text
&common-env → Anchor
*common-env → Alias
<<:         → Merge
```

این قابلیت مفید است، اما اگر خوانایی را برای تیم کم کند، ساده‌تر نوشتن بهتر است.

---

<a id="reading"></a>
## 11. روش خواندن یک YAML پیچیده

مثال:

```yaml
services:
  backend:
    build:
      context: ./backend
    ports:
      - "8080:4000"
    environment:
      DB_HOST: database
    networks:
      - app-net
```

به درخت تبدیلش کن:

```text
services
└── backend
    ├── build
    │   └── context: ./backend
    ├── ports [list]
    │   └── "8080:4000"
    ├── environment
    │   └── DB_HOST: database
    └── networks [list]
        └── app-net
```

چهار سؤال:

```text
1. Parent این خط چیست؟
2. Mapping است یا List؟
3. Value Scalar است یا Structure؟
4. معنی Key را YAML تعیین می‌کند یا Tool؟
```

---

<a id="compose-base"></a>
## 12. ساختار پایه `compose.yaml`

کوچک‌ترین نمونه کاربردی:

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

کامل‌تر:

```yaml
services:
  backend:
    build: ./backend

  database:
    image: postgres:17

volumes:
  db-data:

networks:
  app-net:
```

Root Keyهای پرتکرار:

```text
services
volumes
networks
```

در Compose جدید معمولاً نیازی به `version:` قدیمی نیست.

---

<a id="services"></a>
## 13. `services`

`services` یک Mapping است:

```yaml
services:
  backend:
    image: my-backend

  database:
    image: postgres:17
```

```text
services
├── backend
└── database
```

نام Service در DNS داخلی Compose مهم است. Backend می‌تواند Database را با نام `database` پیدا کند.

---

<a id="build-image"></a>
## 14. `build` و `image`

Image آماده:

```yaml
services:
  nginx:
    image: nginx:alpine
```

Build از Dockerfile:

```yaml
services:
  backend:
    build: ./backend
```

فرم واضح‌تر:

```yaml
services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
```

Build و Tag:

```yaml
services:
  backend:
    build:
      context: ./backend
    image: my-backend:1.0.0
```

```text
image → از Image مشخص استفاده کن
build → Image را از Source بساز
```

---

<a id="ports"></a>
## 15. `ports`

```yaml
ports:
  - "8080:80"
```

```text
Host :8080
   ↓
Container :80
```

اگر Application داخل Container روی `4000` است:

```yaml
ports:
  - "9000:4000"
```

از بیرون `localhost:9000` و داخل Container Port `4000` است.

لزومی ندارد همه Serviceها Port عمومی داشته باشند. Database داخلی می‌تواند فقط روی Network Compose در دسترس Backend باشد.

---

<a id="env"></a>
## 16. `environment`، `.env` و `env_file`

### `environment`

Variable داخل Container:

```yaml
services:
  backend:
    environment:
      APP_ENV: production
      DB_HOST: database
```

فرم List هم ممکن است دیده شود:

```yaml
environment:
  - APP_ENV=production
  - DB_HOST=database
```

برای خوانایی، Mapping معمولاً واضح‌تر است.

### `.env` و Interpolation

`.env`:

```ini
APP_PORT=8080
IMAGE_TAG=1.0.0
```

Compose:

```yaml
services:
  backend:
    image: "my-backend:${IMAGE_TAG}"
    ports:
      - "${APP_PORT}:4000"
```

برای دیدن نتیجه Resolve شده:

```bash
docker compose config
```

### `env_file`

```yaml
services:
  backend:
    env_file:
      - .env
```

Variableهای فایل وارد Environment Container می‌شوند.

مدل:

```text
${VAR} در compose.yaml
→ Interpolation در Configuration

environment:
→ Environment Container

env_file:
→ Load Environment Container از فایل
```

`.env.example` را Commit کن و `.env` واقعی را در `.gitignore` قرار بده.

---

<a id="volumes"></a>
## 17. `volumes` و انواع Mount

### Named Volume

```yaml
services:
  database:
    image: postgres:17
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

سمت چپ `db-data` Volume Docker و سمت راست مسیر داخل Container است.

### Bind Mount

```yaml
services:
  backend:
    volumes:
      - ./backend:/app
```

سمت چپ مسیر Host و سمت راست مسیر Container است.

### Read-only

```yaml
volumes:
  - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
```

### Anonymous Volume

ممکن است ببینی:

```yaml
volumes:
  - /app/node_modules
```

برای شروع، Named Volume و Bind Mount را کامل بفهم و Anonymous Volume را فقط وقتی استفاده کن که دلیلش روشن باشد.

مدل تصمیم:

```text
Database data → Named Volume
Source code in development → Bind Mount
Config file from host → Bind Mount، اغلب :ro
```

---

<a id="networks"></a>
## 18. `networks`

```yaml
services:
  backend:
    networks:
      - backend-net

  database:
    networks:
      - backend-net

networks:
  backend-net:
```

چند Network:

```yaml
services:
  nginx:
    networks:
      - frontend-net

  backend:
    networks:
      - frontend-net
      - backend-net

  database:
    networks:
      - backend-net

networks:
  frontend-net:
  backend-net:
```

```text
Internet
   ↓
Nginx
   │ frontend-net
Backend
   │ backend-net
Database
```

این الگو کمک می‌کند Serviceهایی که لازم نیست مستقیم با هم ارتباط داشته باشند روی یک Network مشترک قرار نگیرند.

---

<a id="health"></a>
## 19. `depends_on` و `healthcheck`

Dependency ساده:

```yaml
services:
  backend:
    depends_on:
      - database
```

اما Start شدن Container الزاماً یعنی Service Ready شده نیست.

Healthcheck:

```yaml
services:
  database:
    image: postgres:17
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 5s
      timeout: 3s
      retries: 10
```

وابستگی به Health:

```yaml
services:
  backend:
    depends_on:
      database:
        condition: service_healthy
```

---

<a id="runtime"></a>
## 20. `restart`، `command` و `entrypoint`

Restart Policy:

```yaml
restart: unless-stopped
```

یا:

```yaml
restart: on-failure
```

Override CMD:

```yaml
command: ["python", "app.py"]
```

Override ENTRYPOINT:

```yaml
entrypoint: ["/app/start.sh"]
```

قاعده:

```text
اگر Dockerfile رفتار درست دارد،
بی‌دلیل command یا entrypoint را در Compose عوض نکن.
```

---

<a id="secrets"></a>
## 21. Secretها را کجا نگذاریم؟

بد:

```yaml
environment:
  DB_PASSWORD: my-production-password
```

اگر فایل Commit شود، Secret وارد History می‌شود.

بهتر:

```yaml
environment:
  DB_PASSWORD: ${DB_PASSWORD}
```

و مقدار واقعی بیرون Repository.

برای Production جدی، Secret Management باید متناسب با Platform انتخاب شود. `.env` برای Development و محیط‌های محدود مفید است، اما جای همه راهکارهای Secret Management را نمی‌گیرد.

---

<a id="scenario1"></a>
## 22. سناریو 1: Nginx تک‌سرویسی

```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
```

Validate:

```bash
docker compose config
```

Run:

```bash
docker compose up -d
```

Test:

```bash
curl http://localhost:8080
```

این ساده‌ترین ساختار برای فهم `services → service → image → ports` است.

---

<a id="scenario2"></a>
## 23. سناریو 2: Backend + Database

```yaml
services:
  backend:
    image: my-backend:1.0.0
    ports:
      - "8080:4000"
    environment:
      DB_HOST: database
      DB_PORT: "5432"
    depends_on:
      - database

  database:
    image: postgres:17
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: dev-password
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

ساختار:

```text
services
├── backend
└── database

volumes
└── db-data
```

در پروژه واقعی Password را از `.env` یا Secret مناسب بگیر.

---

<a id="scenario3"></a>
## 24. سناریو 3: Development با Bind Mount

```yaml
services:
  backend:
    build:
      context: ./backend
    ports:
      - "4000:4000"
    volumes:
      - ./backend:/app
    environment:
      APP_ENV: development
```

```text
Host ./backend
      ⇅
Container /app
```

اگر Runtime Auto Reload داشته باشد، Development سریع‌تر می‌شود.

اما Mount کردن `/app` می‌تواند فایل‌های قبلی همان مسیر در Image را پنهان کند؛ مسیر را آگاهانه انتخاب کن.

---

<a id="scenario4"></a>
## 25. سناریو 4: Network اختصاصی

```yaml
services:
  nginx:
    image: nginx:alpine
    networks:
      - frontend-net

  backend:
    image: my-backend:1.0.0
    networks:
      - frontend-net
      - backend-net

  database:
    image: postgres:17
    networks:
      - backend-net

networks:
  frontend-net:
  backend-net:
```

```text
nginx
  │
frontend-net
  │
backend
  │
backend-net
  │
database
```

از نظر YAML این فقط Mapping و Sequence است؛ معنی Network را Compose و Docker تعیین می‌کنند.

---

<a id="scenario5"></a>
## 26. سناریو 5: `.env` برای Environmentهای مختلف

`.env`:

```ini
APP_PORT=8080
APP_ENV=development
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=dev-secret
```

`compose.yaml`:

```yaml
services:
  backend:
    image: my-backend:1.0.0
    ports:
      - "${APP_PORT}:4000"
    environment:
      APP_ENV: ${APP_ENV}
      DB_HOST: database
      DB_NAME: ${POSTGRES_DB}
      DB_USER: ${POSTGRES_USER}
      DB_PASSWORD: ${POSTGRES_PASSWORD}

  database:
    image: postgres:17
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
```

نمایش نتیجه:

```bash
docker compose config
```

---

<a id="scenario6"></a>
## 27. سناریو 6: Healthcheck واقعی

```yaml
services:
  database:
    image: postgres:17
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: dev-secret
    healthcheck:
      test:
        - CMD-SHELL
        - pg_isready -U appuser -d appdb
      interval: 5s
      timeout: 3s
      retries: 10

  backend:
    image: my-backend:1.0.0
    depends_on:
      database:
        condition: service_healthy
```

همان `test` را می‌توان Flow Style هم نوشت:

```yaml
test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
```

هر دو را باید بتوانی بخوانی.

---

<a id="override"></a>
## 28. Compose چندفایلی و Override

Base:

`compose.yaml`:

```yaml
services:
  backend:
    build: ./backend
    environment:
      APP_ENV: production
```

Development:

`compose.dev.yaml`:

```yaml
services:
  backend:
    environment:
      APP_ENV: development
    volumes:
      - ./backend:/app
```

اجرا:

```bash
docker compose \
  -f compose.yaml \
  -f compose.dev.yaml \
  up -d
```

نتیجه Merge شده:

```bash
docker compose \
  -f compose.yaml \
  -f compose.dev.yaml \
  config
```

برای پروژه کوچک، چند فایل بیش از حد می‌تواند پیچیدگی غیرضروری بسازد.

---

<a id="reuse"></a>
## 29. Reuse با `x-` و Anchor

```yaml
x-common-env: &common-env
  LOG_LEVEL: info
  TZ: UTC

services:
  api:
    image: my-api
    environment:
      <<: *common-env
      APP_ROLE: api

  worker:
    image: my-worker
    environment:
      <<: *common-env
      APP_ROLE: worker
```

هدف:

```text
یک Configuration مشترک
→ چند Service
```

اما معیار اصلی خوانایی است. اگر Anchor باعث شود فهم فایل سخت‌تر شود، Reuse ارزشش را از دست می‌دهد.

---

<a id="error-types"></a>
## 30. سه نوع خطا: YAML، Compose Schema و Runtime

### 1. YAML Syntax Error

```yaml
services:
   backend:
    image: my-backend
```

Indentation خراب است.

### 2. Compose Schema Error

YAML درست است:

```yaml
services:
  backend:
    pizza: large
```

ولی `pizza` Key معتبر Compose نیست.

### 3. Runtime Error

YAML و Schema درست‌اند:

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

اما Host Port `8080` قبلاً اشغال است.

مدل Debug:

```text
YAML parse
   ↓
Compose validation
   ↓
Docker runtime
   ↓
Application runtime
```

---

<a id="validation"></a>
## 31. Validation و Debug

مهم‌ترین Command:

```bash
docker compose config
```

کاربرد:

- Parse فایل
- Resolve Variableها
- Merge فایل‌ها
- نمایش Configuration نهایی
- پیدا کردن بسیاری از خطاهای Structure

Runtime:

```bash
docker compose ps
docker compose logs -f
docker compose logs -f backend
docker compose exec backend sh
```

چند فایل:

```bash
docker compose \
  -f compose.yaml \
  -f compose.dev.yaml \
  config
```

Checklist:

```text
[ ] Indentation درست است؟
[ ] Parent هر Key درست است؟
[ ] List و Mapping را قاطی نکرده‌ام؟
[ ] Key در Schema Compose معتبر است؟
[ ] Variableها Resolve شده‌اند؟
[ ] Port اشغال نیست؟
[ ] Service Name برای DNS درست است؟
[ ] Volume path درست است؟
[ ] Healthcheck کار می‌کند؟
[ ] Application داخل Container Running است؟
```

---

<a id="style"></a>
## 32. Style Guide پیشنهادی

### دو Space برای Indentation

```yaml
services:
  backend:
    image: my-backend
```

### Tab استفاده نکن

Editor را طوری تنظیم کن که YAML با Space Indent شود.

### Port Mapping را Quote کن

```yaml
ports:
  - "8080:80"
```

### Value مبهم را Quote کن

```yaml
version: "1.10"
code: "0123"
```

### Secret واقعی Commit نکن

```text
.env → gitignored
.env.example → committed
```

### Service Name واضح

خوب:

```yaml
services:
  backend:
  database:
  nginx:
```

ضعیف:

```yaml
services:
  a1:
  x:
  srv2:
```

### Comment برای «چرا»

ضعیف:

```yaml
ports:
  - "8080:80" # port
```

بهتر:

```yaml
ports:
  - "8080:80" # local public entrypoint
```

### IP ثابت Hard-code نکن

بد:

```yaml
DB_HOST: "172.20.0.4"
```

خوب:

```yaml
DB_HOST: database
```

### Configuration را بیش از حد Clever نکن

Anchor، Override و چند Network ابزارند، نه هدف. اگر نسخه ساده‌تر همان کار را واضح‌تر انجام می‌دهد، نسخه ساده‌تر برای تیم بهتر است.

### قبل از Commit یا Deploy Validate کن

```bash
docker compose config
```

---

<a id="practice"></a>
## 33. تمرین مرحله‌ای

### مرحله 1: Nginx

```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
```

```bash
docker compose config
docker compose up -d
```

### مرحله 2: Named Volume

```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - web-data:/usr/share/nginx/html

volumes:
  web-data:
```

### مرحله 3: Network و Service دوم

```yaml
services:
  nginx:
    image: nginx:alpine
    networks:
      - app-net

  helper:
    image: alpine
    command: ["sleep", "3600"]
    networks:
      - app-net

networks:
  app-net:
```

### مرحله 4: `.env`

```ini
NGINX_PORT=8080
```

Compose:

```yaml
ports:
  - "${NGINX_PORT}:80"
```

نتیجه:

```bash
docker compose config
```

### مرحله 5: خطای Indentation عمدی

یک خط را بد Indent کن و با `docker compose config` خطا را پیدا کن.

### مرحله 6: Schema Error عمدی

```yaml
pizza: large
```

تفاوت YAML-valid و Compose-invalid را ببین.

اگر این مراحل را بفهمی، YAML برای Compose دیگر مجموعه‌ای از خط‌های حفظی نیست؛ تبدیل به Structure قابل‌تحلیل می‌شود.

---

<a id="cheatsheet"></a>
## 34. Cheat Sheet

### ساختار YAML

```yaml
key: value

parent:
  child: value

list:
  - item1
  - item2

list_of_objects:
  - name: one
    port: 3000
  - name: two
    port: 3001
```

### Compose پایه

```yaml
services:
  backend:
    image: my-backend
    ports:
      - "8080:4000"
    environment:
      APP_ENV: production
    volumes:
      - ./backend:/app
    networks:
      - app-net

volumes:
  data:

networks:
  app-net:
```

### Variable

```yaml
image: "myapp:${IMAGE_TAG}"
```

`.env`:

```ini
IMAGE_TAG=1.0.0
```

### Volume

```yaml
volumes:
  - db-data:/var/lib/postgresql/data
```

### Bind Mount

```yaml
volumes:
  - ./src:/app/src
```

### Network

```yaml
networks:
  - app-net
```

### Healthcheck

```yaml
healthcheck:
  test: ["CMD-SHELL", "curl -f http://localhost:4000/health || exit 1"]
  interval: 10s
  timeout: 3s
  retries: 5
```

### Validate

```bash
docker compose config
```

اصل نهایی:

```text
اول Structure YAML را بخوان.
بعد Schema ابزار را بررسی کن.
بعد Runtime را Debug کن.
```

</div>
