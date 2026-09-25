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

</div>

```yaml
name: Amir
age: 22
enabled: true
```

<div dir="rtl" align="right">


برای انسان:

</div>

```text
name    → Amir
age     → 22
enabled → true
```

<div dir="rtl" align="right">


برای برنامه چیزی شبیه Object / Dictionary است.

YAML خودش Docker را اجرا نمی‌کند، Network نمی‌سازد و Container ایجاد نمی‌کند. فقط Data را با Syntax مشخص بیان می‌کند.

این Data می‌تواند توسط ابزارهای مختلف خوانده شود:

</div>

```text
Docker Compose
GitHub Actions
Kubernetes
Ansible
CI/CD systems
```

<div dir="rtl" align="right">


مهارت اصلی:

</div>

```text
YAML syntax
+
schema ابزار
```

<div dir="rtl" align="right">


---

<a id="syntax-schema"></a>
## 2. Syntax با Schema فرق دارد

این یکی از مهم‌ترین مفاهیم کل سند است.

</div>

```yaml
services:
  web:
    image: nginx:alpine
```

<div dir="rtl" align="right">


YAML فقط ساختار را می‌فهمد:

</div>

```text
services
└── web
    └── image: nginx:alpine
```

<div dir="rtl" align="right">


اما معنی `services` و `image` را Docker Compose تعیین می‌کند.

این YAML هم از نظر Syntax معتبر است:

</div>

```yaml
pizza:
  cheese: lots
  olives:
    - black
    - green
```

<div dir="rtl" align="right">


ولی Compose چنین Schemaای ندارد.

پس یک فایل می‌تواند:

</div>

```text
YAML-valid
ولی
Compose-invalid
```

<div dir="rtl" align="right">


باشد.

مدل خطا:

</div>

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

<div dir="rtl" align="right">


---

<a id="structures"></a>
## 3. سه ساختار اصلی YAML

تقریباً بیشتر YAMLهایی که در DevOps می‌بینی از سه نوع اصلی ساخته شده‌اند.

### 3.1. Scalar

</div>

```yaml
name: backend
replicas: 2
enabled: true
timeout: 2.5
optional: null
```

<div dir="rtl" align="right">


### 3.2. Mapping

</div>

```yaml
database:
  host: database
  port: 5432
```

<div dir="rtl" align="right">

</div>

```text
database
├── host → database
└── port → 5432
```

<div dir="rtl" align="right">


### 3.3. Sequence

</div>

```yaml
ports:
  - "8080:80"
  - "8443:443"
```

<div dir="rtl" align="right">


### ترکیب

</div>

```yaml
services:
  backend:
    environment:
      APP_ENV: production
    ports:
      - "8080:4000"
```

<div dir="rtl" align="right">

</div>

```text
services                 Mapping
└── backend              Mapping
    ├── environment      Mapping
    │   └── APP_ENV      Scalar
    └── ports            Sequence
        └── "8080:4000"  Scalar
```

<div dir="rtl" align="right">


اگر بتوانی YAML را به چنین درختی تبدیل کنی، فایل‌های بزرگ خیلی قابل‌فهم‌تر می‌شوند.

---

<a id="indentation"></a>
## 4. Indentation

فاصله ابتدای خط در YAML معنی ساختاری دارد.

در این سند از دو Space برای هر Level استفاده می‌کنیم:

</div>

```text
Level 0 → 0 spaces
Level 1 → 2 spaces
Level 2 → 4 spaces
Level 3 → 6 spaces
```

<div dir="rtl" align="right">


درست:

</div>

```yaml
services:
  backend:
    image: my-backend
    environment:
      APP_ENV: production
```

<div dir="rtl" align="right">


ساختار:

</div>

```text
services
└── backend
    ├── image
    └── environment
        └── APP_ENV
```

<div dir="rtl" align="right">


نامنظم:

</div>

```yaml
services:
   backend:
    image: my-backend
```

<div dir="rtl" align="right">


ممکن است Error یا ساختار اشتباه ایجاد کند.

قاعده عملی:

</div>

```text
برای Indentation از Space استفاده کن.
در Editor، Tab را برای YAML به Space تبدیل کن.
```

<div dir="rtl" align="right">


YAML الزام نمی‌کند همیشه دقیقاً دو Space استفاده شود، اما Consistency مهم است.

---

<a id="colon-dash"></a>
## 5. Colon و Dash

### `:`

Key را از Value جدا می‌کند:

</div>

```yaml
name: backend
```

<div dir="rtl" align="right">


اگر بعد از Colon مقدار مستقیم نباشد:

</div>

```yaml
database:
```

<div dir="rtl" align="right">


یعنی زیر آن Structure دیگری می‌آید:

</div>

```yaml
database:
  host: db
  port: 5432
```

<div dir="rtl" align="right">


### `-`

Dash + Space معمولاً یک عضو List می‌سازد:

</div>

```yaml
networks:
  - frontend
  - backend
```

<div dir="rtl" align="right">


List می‌تواند از Mapping تشکیل شود:

</div>

```yaml
servers:
  - name: api-1
    port: 3000

  - name: api-2
    port: 3001
```

<div dir="rtl" align="right">


قاعده خواندن:

</div>

```text
Dash دیدی؟
→ یک item جدید در List شروع شده.
```

<div dir="rtl" align="right">


---

<a id="values"></a>
## 6. نوع مقدارها و Quote

</div>

```yaml
name: backend
replicas: 2
timeout: 1.5
debug: true
cache: false
optional: null
```

<div dir="rtl" align="right">


| مقدار | نوع تقریبی |
|---|---|
| `backend` | String |
| `2` | Integer |
| `1.5` | Float |
| `true` | Boolean |
| `null` | Null |

برای Valueهایی که باید قطعاً String بمانند، Quote مفید است:

</div>

```yaml
version: "1.10"
phone_like: "01234"
answer: "yes"
mode: "on"
```

<div dir="rtl" align="right">


Port Mapping:

</div>

```yaml
ports:
  - "8080:80"
```

<div dir="rtl" align="right">


Image Tag:

</div>

```yaml
image: "myapp:1.4.2"
```

<div dir="rtl" align="right">


URL و Cron:

</div>

```yaml
url: "https://example.com/api"
schedule: "0 3 * * *"
```

<div dir="rtl" align="right">


Password با `#`:

</div>

```yaml
password: "abc#123"
```

<div dir="rtl" align="right">


قاعده امن:

</div>

```text
اگر مقدار شبیه Number، Boolean، Date یا Syntax خاص است
ولی باید String بماند
→ Quote بگذار.
```

<div dir="rtl" align="right">


---

<a id="comment"></a>
## 7. Comment

</div>

```yaml
# Application configuration
name: backend
```

<div dir="rtl" align="right">


Inline:

</div>

```yaml
port: 3000 # internal application port
```

<div dir="rtl" align="right">


اگر `#` بخشی از Value است Quote بگذار:

</div>

```yaml
password: "abc#123"
```

<div dir="rtl" align="right">


Comment باید Context یا دلیل بدهد، نه این‌که فقط همان خط را تکرار کند.

ضعیف:

</div>

```yaml
ports:
  - "8080:80" # port
```

<div dir="rtl" align="right">


بهتر:

</div>

```yaml
ports:
  - "8080:80" # local public entrypoint
```

<div dir="rtl" align="right">


---

<a id="multiline"></a>
## 8. متن چندخطی با `|` و `>`

### `|`

Line breakها را حفظ می‌کند:

</div>

```yaml
message: |
  line one
  line two
  line three
```

<div dir="rtl" align="right">


### `>`

خط‌ها را بیشتر به شکل متن پیوسته Fold می‌کند:

</div>

```yaml
description: >
  This is a long description
  written across multiple lines.
```

<div dir="rtl" align="right">


در Compose روزمره کمتر لازم می‌شود، اما در CI/CD و Configها ممکن است ببینی.

---

<a id="flow"></a>
## 9. Flow Style با `{}` و `[]`

فرم فشرده:

</div>

```yaml
user: {name: Ali, role: backend}
ports: ["80", "443"]
```

<div dir="rtl" align="right">


فرم Block معمولاً خواناتر است:

</div>

```yaml
user:
  name: Ali
  role: backend

ports:
  - "80"
  - "443"
```

<div dir="rtl" align="right">


Flow Style را وقتی استفاده کن که واقعاً خوانایی بهتر شود.

---

<a id="anchors"></a>
## 10. Anchor، Alias و Merge

برای Reuse:

</div>

```yaml
x-common-env: &common-env
  LOG_LEVEL: info
  TZ: UTC
```

<div dir="rtl" align="right">


استفاده:

</div>

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

<div dir="rtl" align="right">

</div>

```text
&common-env → Anchor
*common-env → Alias
<<:         → Merge
```

<div dir="rtl" align="right">


این قابلیت مفید است، اما اگر خوانایی را برای تیم کم کند، ساده‌تر نوشتن بهتر است.

---

<a id="reading"></a>
## 11. روش خواندن یک YAML پیچیده

مثال:

</div>

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

<div dir="rtl" align="right">


به درخت تبدیلش کن:

</div>

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

<div dir="rtl" align="right">


چهار سؤال:

</div>

```text
1. Parent این خط چیست؟
2. Mapping است یا List؟
3. Value Scalar است یا Structure؟
4. معنی Key را YAML تعیین می‌کند یا Tool؟
```

<div dir="rtl" align="right">


---

<a id="compose-base"></a>
## 12. ساختار پایه `compose.yaml`

کوچک‌ترین نمونه کاربردی:

</div>

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

<div dir="rtl" align="right">


کامل‌تر:

</div>

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

<div dir="rtl" align="right">


Root Keyهای پرتکرار:

</div>

```text
services
volumes
networks
```

<div dir="rtl" align="right">


در Compose جدید معمولاً نیازی به `version:` قدیمی نیست.

---

<a id="services"></a>
## 13. `services`

`services` یک Mapping است:

</div>

```yaml
services:
  backend:
    image: my-backend

  database:
    image: postgres:17
```

<div dir="rtl" align="right">

</div>

```text
services
├── backend
└── database
```

<div dir="rtl" align="right">


نام Service در DNS داخلی Compose مهم است. Backend می‌تواند Database را با نام `database` پیدا کند.

---

<a id="build-image"></a>
## 14. `build` و `image`

Image آماده:

</div>

```yaml
services:
  nginx:
    image: nginx:alpine
```

<div dir="rtl" align="right">


Build از Dockerfile:

</div>

```yaml
services:
  backend:
    build: ./backend
```

<div dir="rtl" align="right">


فرم واضح‌تر:

</div>

```yaml
services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
```

<div dir="rtl" align="right">


Build و Tag:

</div>

```yaml
services:
  backend:
    build:
      context: ./backend
    image: my-backend:1.0.0
```

<div dir="rtl" align="right">

</div>

```text
image → از Image مشخص استفاده کن
build → Image را از Source بساز
```

<div dir="rtl" align="right">


---

<a id="ports"></a>
## 15. `ports`

</div>

```yaml
ports:
  - "8080:80"
```

<div dir="rtl" align="right">

</div>

```text
Host :8080
   ↓
Container :80
```

<div dir="rtl" align="right">


اگر Application داخل Container روی `4000` است:

</div>

```yaml
ports:
  - "9000:4000"
```

<div dir="rtl" align="right">


از بیرون `localhost:9000` و داخل Container Port `4000` است.

لزومی ندارد همه Serviceها Port عمومی داشته باشند. Database داخلی می‌تواند فقط روی Network Compose در دسترس Backend باشد.

---

<a id="env"></a>
## 16. `environment`، `.env` و `env_file`

### `environment`

Variable داخل Container:

</div>

```yaml
services:
  backend:
    environment:
      APP_ENV: production
      DB_HOST: database
```

<div dir="rtl" align="right">


فرم List هم ممکن است دیده شود:

</div>

```yaml
environment:
  - APP_ENV=production
  - DB_HOST=database
```

<div dir="rtl" align="right">


برای خوانایی، Mapping معمولاً واضح‌تر است.

### `.env` و Interpolation

`.env`:

</div>

```ini
APP_PORT=8080
IMAGE_TAG=1.0.0
```

<div dir="rtl" align="right">


Compose:

</div>

```yaml
services:
  backend:
    image: "my-backend:${IMAGE_TAG}"
    ports:
      - "${APP_PORT}:4000"
```

<div dir="rtl" align="right">


برای دیدن نتیجه Resolve شده:

</div>

```bash
docker compose config
```

<div dir="rtl" align="right">


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

مدل:

</div>

```text
${VAR} در compose.yaml
→ Interpolation در Configuration

environment:
→ Environment Container

env_file:
→ Load Environment Container از فایل
```

<div dir="rtl" align="right">


`.env.example` را Commit کن و `.env` واقعی را در `.gitignore` قرار بده.

---

<a id="volumes"></a>
## 17. `volumes` و انواع Mount

### Named Volume

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


سمت چپ `db-data` Volume Docker و سمت راست مسیر داخل Container است.

### Bind Mount

</div>

```yaml
services:
  backend:
    volumes:
      - ./backend:/app
```

<div dir="rtl" align="right">


سمت چپ مسیر Host و سمت راست مسیر Container است.

### Read-only

</div>

```yaml
volumes:
  - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
```

<div dir="rtl" align="right">


### Anonymous Volume

ممکن است ببینی:

</div>

```yaml
volumes:
  - /app/node_modules
```

<div dir="rtl" align="right">


برای شروع، Named Volume و Bind Mount را کامل بفهم و Anonymous Volume را فقط وقتی استفاده کن که دلیلش روشن باشد.

مدل تصمیم:

</div>

```text
Database data → Named Volume
Source code in development → Bind Mount
Config file from host → Bind Mount، اغلب :ro
```

<div dir="rtl" align="right">


---

<a id="networks"></a>
## 18. `networks`

</div>

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

<div dir="rtl" align="right">


چند Network:

</div>

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

<div dir="rtl" align="right">

</div>

```text
Internet
   ↓
Nginx
   │ frontend-net
Backend
   │ backend-net
Database
```

<div dir="rtl" align="right">


این الگو کمک می‌کند Serviceهایی که لازم نیست مستقیم با هم ارتباط داشته باشند روی یک Network مشترک قرار نگیرند.

---

<a id="health"></a>
## 19. `depends_on` و `healthcheck`

Dependency ساده:

</div>

```yaml
services:
  backend:
    depends_on:
      - database
```

<div dir="rtl" align="right">


اما Start شدن Container الزاماً یعنی Service Ready شده نیست.

Healthcheck:

</div>

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

<div dir="rtl" align="right">


وابستگی به Health:

</div>

```yaml
services:
  backend:
    depends_on:
      database:
        condition: service_healthy
```

<div dir="rtl" align="right">


---

<a id="runtime"></a>
## 20. `restart`، `command` و `entrypoint`

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


Override CMD:

</div>

```yaml
command: ["python", "app.py"]
```

<div dir="rtl" align="right">


Override ENTRYPOINT:

</div>

```yaml
entrypoint: ["/app/start.sh"]
```

<div dir="rtl" align="right">


قاعده:

</div>

```text
اگر Dockerfile رفتار درست دارد،
بی‌دلیل command یا entrypoint را در Compose عوض نکن.
```

<div dir="rtl" align="right">


---

<a id="secrets"></a>
## 21. Secretها را کجا نگذاریم؟

بد:

</div>

```yaml
environment:
  DB_PASSWORD: my-production-password
```

<div dir="rtl" align="right">


اگر فایل Commit شود، Secret وارد History می‌شود.

بهتر:

</div>

```yaml
environment:
  DB_PASSWORD: ${DB_PASSWORD}
```

<div dir="rtl" align="right">


و مقدار واقعی بیرون Repository.

برای Production جدی، Secret Management باید متناسب با Platform انتخاب شود. `.env` برای Development و محیط‌های محدود مفید است، اما جای همه راهکارهای Secret Management را نمی‌گیرد.

---

<a id="scenario1"></a>
## 22. سناریو 1: Nginx تک‌سرویسی

</div>

```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
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
docker compose up -d
```

<div dir="rtl" align="right">


Test:

</div>

```bash
curl http://localhost:8080
```

<div dir="rtl" align="right">


این ساده‌ترین ساختار برای فهم `services → service → image → ports` است.

---

<a id="scenario2"></a>
## 23. سناریو 2: Backend + Database

</div>

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

<div dir="rtl" align="right">


ساختار:

</div>

```text
services
├── backend
└── database

volumes
└── db-data
```

<div dir="rtl" align="right">


در پروژه واقعی Password را از `.env` یا Secret مناسب بگیر.

---

<a id="scenario3"></a>
## 24. سناریو 3: Development با Bind Mount

</div>

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

<div dir="rtl" align="right">

</div>

```text
Host ./backend
      ⇅
Container /app
```

<div dir="rtl" align="right">


اگر Runtime Auto Reload داشته باشد، Development سریع‌تر می‌شود.

اما Mount کردن `/app` می‌تواند فایل‌های قبلی همان مسیر در Image را پنهان کند؛ مسیر را آگاهانه انتخاب کن.

---

<a id="scenario4"></a>
## 25. سناریو 4: Network اختصاصی

</div>

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

<div dir="rtl" align="right">

</div>

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

<div dir="rtl" align="right">


از نظر YAML این فقط Mapping و Sequence است؛ معنی Network را Compose و Docker تعیین می‌کنند.

---

<a id="scenario5"></a>
## 26. سناریو 5: `.env` برای Environmentهای مختلف

`.env`:

</div>

```ini
APP_PORT=8080
APP_ENV=development
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=dev-secret
```

<div dir="rtl" align="right">


`compose.yaml`:

</div>

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

<div dir="rtl" align="right">


نمایش نتیجه:

</div>

```bash
docker compose config
```

<div dir="rtl" align="right">


---

<a id="scenario6"></a>
## 27. سناریو 6: Healthcheck واقعی

</div>

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

<div dir="rtl" align="right">


همان `test` را می‌توان Flow Style هم نوشت:

</div>

```yaml
test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
```

<div dir="rtl" align="right">


هر دو را باید بتوانی بخوانی.

---

<a id="override"></a>
## 28. Compose چندفایلی و Override

Base:

`compose.yaml`:

</div>

```yaml
services:
  backend:
    build: ./backend
    environment:
      APP_ENV: production
```

<div dir="rtl" align="right">


Development:

`compose.dev.yaml`:

</div>

```yaml
services:
  backend:
    environment:
      APP_ENV: development
    volumes:
      - ./backend:/app
```

<div dir="rtl" align="right">


اجرا:

</div>

```bash
docker compose \
  -f compose.yaml \
  -f compose.dev.yaml \
  up -d
```

<div dir="rtl" align="right">


نتیجه Merge شده:

</div>

```bash
docker compose \
  -f compose.yaml \
  -f compose.dev.yaml \
  config
```

<div dir="rtl" align="right">


برای پروژه کوچک، چند فایل بیش از حد می‌تواند پیچیدگی غیرضروری بسازد.

---

<a id="reuse"></a>
## 29. Reuse با `x-` و Anchor

</div>

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

<div dir="rtl" align="right">


هدف:

</div>

```text
یک Configuration مشترک
→ چند Service
```

<div dir="rtl" align="right">


اما معیار اصلی خوانایی است. اگر Anchor باعث شود فهم فایل سخت‌تر شود، Reuse ارزشش را از دست می‌دهد.

---

<a id="error-types"></a>
## 30. سه نوع خطا: YAML، Compose Schema و Runtime

### 1. YAML Syntax Error

</div>

```yaml
services:
   backend:
    image: my-backend
```

<div dir="rtl" align="right">


Indentation خراب است.

### 2. Compose Schema Error

YAML درست است:

</div>

```yaml
services:
  backend:
    pizza: large
```

<div dir="rtl" align="right">


ولی `pizza` Key معتبر Compose نیست.

### 3. Runtime Error

YAML و Schema درست‌اند:

</div>

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

<div dir="rtl" align="right">


اما Host Port `8080` قبلاً اشغال است.

مدل Debug:

</div>

```text
YAML parse
   ↓
Compose validation
   ↓
Docker runtime
   ↓
Application runtime
```

<div dir="rtl" align="right">


---

<a id="validation"></a>
## 31. Validation و Debug

مهم‌ترین Command:

</div>

```bash
docker compose config
```

<div dir="rtl" align="right">


کاربرد:

- Parse فایل
- Resolve Variableها
- Merge فایل‌ها
- نمایش Configuration نهایی
- پیدا کردن بسیاری از خطاهای Structure

Runtime:

</div>

```bash
docker compose ps
docker compose logs -f
docker compose logs -f backend
docker compose exec backend sh
```

<div dir="rtl" align="right">


چند فایل:

</div>

```bash
docker compose \
  -f compose.yaml \
  -f compose.dev.yaml \
  config
```

<div dir="rtl" align="right">


Checklist:

</div>

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

<div dir="rtl" align="right">


---

<a id="style"></a>
## 32. Style Guide پیشنهادی

### دو Space برای Indentation

</div>

```yaml
services:
  backend:
    image: my-backend
```

<div dir="rtl" align="right">


### Tab استفاده نکن

Editor را طوری تنظیم کن که YAML با Space Indent شود.

### Port Mapping را Quote کن

</div>

```yaml
ports:
  - "8080:80"
```

<div dir="rtl" align="right">


### Value مبهم را Quote کن

</div>

```yaml
version: "1.10"
code: "0123"
```

<div dir="rtl" align="right">


### Secret واقعی Commit نکن

</div>

```text
.env → gitignored
.env.example → committed
```

<div dir="rtl" align="right">


### Service Name واضح

خوب:

</div>

```yaml
services:
  backend:
  database:
  nginx:
```

<div dir="rtl" align="right">


ضعیف:

</div>

```yaml
services:
  a1:
  x:
  srv2:
```

<div dir="rtl" align="right">


### Comment برای «چرا»

ضعیف:

</div>

```yaml
ports:
  - "8080:80" # port
```

<div dir="rtl" align="right">


بهتر:

</div>

```yaml
ports:
  - "8080:80" # local public entrypoint
```

<div dir="rtl" align="right">


### IP ثابت Hard-code نکن

بد:

</div>

```yaml
DB_HOST: "172.20.0.4"
```

<div dir="rtl" align="right">


خوب:

</div>

```yaml
DB_HOST: database
```

<div dir="rtl" align="right">


### Configuration را بیش از حد Clever نکن

Anchor، Override و چند Network ابزارند، نه هدف. اگر نسخه ساده‌تر همان کار را واضح‌تر انجام می‌دهد، نسخه ساده‌تر برای تیم بهتر است.

### قبل از Commit یا Deploy Validate کن

</div>

```bash
docker compose config
```

<div dir="rtl" align="right">


---

<a id="practice"></a>
## 33. تمرین مرحله‌ای

### مرحله 1: Nginx

</div>

```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
```

<div dir="rtl" align="right">

</div>

```bash
docker compose config
docker compose up -d
```

<div dir="rtl" align="right">


### مرحله 2: Named Volume

</div>

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

<div dir="rtl" align="right">


### مرحله 3: Network و Service دوم

</div>

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

<div dir="rtl" align="right">


### مرحله 4: `.env`

</div>

```ini
NGINX_PORT=8080
```

<div dir="rtl" align="right">


Compose:

</div>

```yaml
ports:
  - "${NGINX_PORT}:80"
```

<div dir="rtl" align="right">


نتیجه:

</div>

```bash
docker compose config
```

<div dir="rtl" align="right">


### مرحله 5: خطای Indentation عمدی

یک خط را بد Indent کن و با `docker compose config` خطا را پیدا کن.

### مرحله 6: Schema Error عمدی

</div>

```yaml
pizza: large
```

<div dir="rtl" align="right">


تفاوت YAML-valid و Compose-invalid را ببین.

اگر این مراحل را بفهمی، YAML برای Compose دیگر مجموعه‌ای از خط‌های حفظی نیست؛ تبدیل به Structure قابل‌تحلیل می‌شود.

---

<a id="cheatsheet"></a>
## 34. Cheat Sheet

### ساختار YAML

</div>

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

<div dir="rtl" align="right">


### Compose پایه

</div>

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

<div dir="rtl" align="right">


### Variable

</div>

```yaml
image: "myapp:${IMAGE_TAG}"
```

<div dir="rtl" align="right">


`.env`:

</div>

```ini
IMAGE_TAG=1.0.0
```

<div dir="rtl" align="right">


### Volume

</div>

```yaml
volumes:
  - db-data:/var/lib/postgresql/data
```

<div dir="rtl" align="right">


### Bind Mount

</div>

```yaml
volumes:
  - ./src:/app/src
```

<div dir="rtl" align="right">


### Network

</div>

```yaml
networks:
  - app-net
```

<div dir="rtl" align="right">


### Healthcheck

</div>

```yaml
healthcheck:
  test: ["CMD-SHELL", "curl -f http://localhost:4000/health || exit 1"]
  interval: 10s
  timeout: 3s
  retries: 5
```

<div dir="rtl" align="right">


### Validate

</div>

```bash
docker compose config
```

<div dir="rtl" align="right">


اصل نهایی:

</div>

```text
اول Structure YAML را بخوان.
بعد Schema ابزار را بررسی کن.
بعد Runtime را Debug کن.
```

<div dir="rtl" align="right">

</div>