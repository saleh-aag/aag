<div dir="rtl" align="right">

راهنمای عملی <span dir="ltr">YAML</span> و <span dir="ltr">Docker Compose</span> — الگوهای آماده برای کپی

این فایل برای خواندن خطی از اول تا آخر طراحی نشده است. هدف این است که وقتی در یک پروژه واقعی به چیزی مثل <code dir="ltr">ports</code>، <code dir="ltr">volumes</code>، <code dir="ltr">.env</code>، <code dir="ltr">networks</code> یا <code dir="ltr">healthcheck</code> نیاز داشتی، از فهرست مستقیم به همان بخش بروی، نمونه را Copy کنی، مقادیر مشخص‌شده را تغییر بدهی و فایل را <span dir="ltr">Validate</span> کنی.

روش استفاده از این سند: <span dir="ltr">Find → Copy → Change → Validate → Run</span>

Find the pattern
      |
      v
Copy the example
      |
      v
Change project values
      |
      v
docker compose config
      |
      v
docker compose up -d

هر <span dir="ltr">Code Block</span> مستقل نوشته شده تا در <span dir="ltr">GitHub</span> بتوانی با دکمه‌ی <span dir="ltr">Copy</span> همان بلوک را برداری و در پروژه خودت استفاده کنی.

<a id="toc" name="toc"></a>

فهرست مطالب

<ol dir="rtl" align="right">
  <li><a href="#workflow">روش استفاده سریع از این فایل</a></li>
  <li><a href="#yaml-core">قواعد پایه <span dir="ltr">YAML</span> در یک نگاه</a></li>
  <li><a href="#indentation"><span dir="ltr">Indentation</span> و ساختار تو‌در‌تو</a></li>
  <li><a href="#mapping-list"><span dir="ltr">Mapping</span> و <span dir="ltr">List</span></a></li>
  <li><a href="#quotes"><span dir="ltr">Quote</span> و مقدارهای حساس</a></li>
  <li><a href="#multiline">متن چندخطی با <code dir="ltr">|</code> و <code dir="ltr">&gt;</code></a></li>
  <li><a href="#compose-minimal">الگوی حداقلی <code dir="ltr">compose.yaml</code></a></li>
  <li><a href="#image">استفاده از <code dir="ltr">image</code></a></li>
  <li><a href="#build">استفاده از <code dir="ltr">build</code></a></li>
  <li><a href="#ports"><code dir="ltr">ports</code> — انتشار پورت</a></li>
  <li><a href="#environment"><code dir="ltr">environment</code></a></li>
  <li><a href="#dot-env"><code dir="ltr">.env</code> و <span dir="ltr">Variable Interpolation</span></a></li>
  <li><a href="#env-file"><code dir="ltr">env_file</code></a></li>
  <li><a href="#named-volume"><span dir="ltr">Named Volume</span></a></li>
  <li><a href="#bind-mount"><span dir="ltr">Bind Mount</span></a></li>
  <li><a href="#readonly-mount"><span dir="ltr">Read-only Mount</span></a></li>
  <li><a href="#anonymous-volume"><span dir="ltr">Anonymous Volume</span></a></li>
  <li><a href="#network"><span dir="ltr">Network</span> اختصاصی</a></li>
  <li><a href="#two-networks">تفکیک <span dir="ltr">Frontend</span> و <span dir="ltr">Backend Network</span></a></li>
  <li><a href="#depends-on"><code dir="ltr">depends_on</code></a></li>
  <li><a href="#healthcheck"><code dir="ltr">healthcheck</code> و <code dir="ltr">service_healthy</code></a></li>
  <li><a href="#restart-command"><code dir="ltr">restart</code>، <code dir="ltr">command</code> و <code dir="ltr">entrypoint</code></a></li>
  <li><a href="#anchors"><span dir="ltr">Anchor</span> و <span dir="ltr">Reuse</span></a></li>
  <li><a href="#template-nginx">قالب کامل 1 — <span dir="ltr">Nginx</span></a></li>
  <li><a href="#template-backend-db">قالب کامل 2 — <span dir="ltr">Backend + PostgreSQL</span></a></li>
  <li><a href="#template-env-db">قالب کامل 3 — <span dir="ltr">Backend + PostgreSQL + .env</span></a></li>
  <li><a href="#template-dev">قالب کامل 4 — محیط <span dir="ltr">Development</span> با <span dir="ltr">Bind Mount</span></a></li>
  <li><a href="#template-networks">قالب کامل 5 — سه سرویس با دو <span dir="ltr">Network</span></a></li>
  <li><a href="#override"><span dir="ltr">Compose Override</span> برای <span dir="ltr">Development</span></a></li>
  <li><a href="#validation"><span dir="ltr">Validate</span> و <span dir="ltr">Debug</span></a></li>
  <li><a href="#errors">سه نوع خطای مهم</a></li>
  <li><a href="#copy-checklist"><span dir="ltr">Copy Checklist</span> قبل از استفاده</a></li>
  <li><a href="#cheatsheet"><span dir="ltr">Cheat Sheet</span> نهایی</a></li>
</ol>

<a id="workflow" name="workflow"></a>

1. روش استفاده سریع از این فایل

برای هر الگو چهار مرحله داریم:

<table dir="rtl">
  <thead><tr><th>مرحله</th><th>کار</th></tr></thead>
  <tbody>
    <tr><td>1</td><td>نمونه مناسب را از فهرست پیدا کن.</td></tr>
    <tr><td>2</td><td><span dir="ltr">Code Block</span> را کامل <span dir="ltr">Copy</span> کن.</td></tr>
    <tr><td>3</td><td>فقط مقادیر پروژه خودت مثل نام سرویس، مسیر، پورت و نام <span dir="ltr">Volume</span> را تغییر بده.</td></tr>
    <tr><td>4</td><td>قبل از اجرا، <code dir="ltr">docker compose config</code> بزن.</td></tr>
  </tbody>
</table>

دستور پایه برای بررسی فایل:

docker compose config

اگر درست بود:

docker compose up -d

برای دیدن وضعیت:

docker compose ps

برای دیدن لاگ‌ها:

docker compose logs -f

<a id="yaml-core" name="yaml-core"></a>

2. قواعد پایه <span dir="ltr">YAML</span> در یک نگاه

سه ساختمان اصلی که تقریباً همه‌جا می‌بینی:

مقدار ساده — <span dir="ltr">Scalar</span>

name: backend
port: 4000
enabled: true

کلید و مقدار — <span dir="ltr">Mapping</span>

database:
  host: database
  port: 5432

لیست — <span dir="ltr">Sequence</span>

ports:
  - "8080:80"
  - "8443:443"

مدل ذهنی: علامت <code dir="ltr">:</code> معمولاً <span dir="ltr">Key</span> را از مقدارش جدا می‌کند و <code dir="ltr">-</code> معمولاً یک عضو جدید از <span dir="ltr">List</span> را شروع می‌کند.

<a id="indentation" name="indentation"></a>

3. <span dir="ltr">Indentation</span> و ساختار تو‌در‌تو

الگوی پیشنهادی این سند: دو Space برای هر Level.

درست — قابل الگوبرداری

services:
  backend:
    image: my-backend:1.0.0
    environment:
      APP_ENV: production

اشتباه

services:
   backend:
    image: my-backend:1.0.0

قاعده عملی:

برای <span dir="ltr">Indentation</span> از <span dir="ltr">Space</span> استفاده کن.

<span dir="ltr">Tab</span> را برای فایل‌های <span dir="ltr">YAML</span> به <span dir="ltr">Space</span> تبدیل کن.

همه‌ی فرزندان یک <span dir="ltr">Parent</span> باید Level یکسان داشته باشند.

<a id="mapping-list" name="mapping-list"></a>

4. <span dir="ltr">Mapping</span> و <span dir="ltr">List</span> را با هم اشتباه نکن

<span dir="ltr">Mapping</span>

environment:
  APP_ENV: production
  DB_HOST: database

<span dir="ltr">List</span>

networks:
  - frontend-net
  - backend-net

لیست از چند <span dir="ltr">Object</span>

servers:
  - name: api-1
    port: 3000
  - name: api-2
    port: 3001

وقتی یک فایل پیچیده شد، اول تشخیص بده بخش فعلی Mapping است یا List؛ بعد سراغ معنی کلیدها برو.

<a id="quotes" name="quotes"></a>

5. <span dir="ltr">Quote</span> و مقدارهای حساس

برای مقدارهایی که باید حتماً <span dir="ltr">String</span> باقی بمانند، <span dir="ltr">Quote</span> انتخاب امنی است.

الگوی آماده

version: "1.10"
code: "01234"
answer: "yes"
mode: "on"
url: "https://example.com/api"
schedule: "0 3 * * *"
password: "abc#123"

برای <span dir="ltr">Port Mapping</span> هم همین الگو را استفاده کن:

ports:
  - "8080:80"

<a id="multiline" name="multiline"></a>

6. متن چندخطی با <code dir="ltr">|</code> و <code dir="ltr">></code>

حفظ <span dir="ltr">Line Break</span>ها

message: |
  line one
  line two
  line three

تبدیل چند خط به متن پیوسته‌تر

description: >
  This is a long description
  written across multiple lines.

اگر در پروژه روزمره به این قابلیت نیاز نداری، لازم نیست برای شروع بیشتر از همین دو الگو حفظ کنی.

<a id="compose-minimal" name="compose-minimal"></a>

7. الگوی حداقلی <code dir="ltr">compose.yaml</code>

آماده برای کپی

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"

چه چیزهایی را تغییر بدهم؟

<table dir="rtl">
  <thead><tr><th>مقدار</th><th>برای پروژه خودت</th></tr></thead>
  <tbody>
    <tr><td><code dir="ltr">web</code></td><td>نام سرویس</td></tr>
    <tr><td><code dir="ltr">nginx:alpine</code></td><td>نام و Tag ایمیج</td></tr>
    <tr><td><code dir="ltr">8080:80</code></td><td><span dir="ltr">HOST_PORT:CONTAINER_PORT</span></td></tr>
  </tbody>
</table>

تست

docker compose config
docker compose up -d
docker compose ps

<a id="image" name="image"></a>

8. استفاده از <code dir="ltr">image</code>

وقتی می‌خواهی از یک <span dir="ltr">Image</span> آماده استفاده کنی:

services:
  web:
    image: nginx:alpine

نمونه دیگر:

services:
  database:
    image: postgres:17

فقط این بخش را تغییر بده: مقدار مقابل <code dir="ltr">image:</code>.

<a id="build" name="build"></a>

9. استفاده از <code dir="ltr">build</code>

وقتی پروژه خودت <code dir="ltr">Dockerfile</code> دارد:

حالت کوتاه

services:
  backend:
    build: ./backend

حالت واضح‌تر و قابل توسعه

services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile

<span dir="ltr">Build</span> + نام‌گذاری <span dir="ltr">Image</span>

services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    image: my-backend:1.0.0

<table dir="rtl">
  <thead><tr><th>قسمت</th><th>تغییر بده به</th></tr></thead>
  <tbody>
    <tr><td><code dir="ltr">./backend</code></td><td>مسیر پوشه پروژه</td></tr>
    <tr><td><code dir="ltr">Dockerfile</code></td><td>نام Dockerfile در صورت متفاوت بودن</td></tr>
    <tr><td><code dir="ltr">my-backend:1.0.0</code></td><td>نام و Tag دلخواه Image</td></tr>
  </tbody>
</table>

<a id="ports" name="ports"></a>

10. <code dir="ltr">ports</code> — انتشار پورت

فرم اصلی:

ports:
  - "8080:80"

معنی:

Host 8080
   |
   v
Container 80

اگر برنامه داخل <span dir="ltr">Container</span> روی پورت <code dir="ltr">4000</code> اجرا می‌شود و می‌خواهی روی Host با <code dir="ltr">9000</code> باز شود:

ports:
  - "9000:4000"

قانون حفظی: سمت چپ <span dir="ltr">Host</span>، سمت راست <span dir="ltr">Container</span>.

<a id="environment" name="environment"></a>

11. <code dir="ltr">environment</code>

الگوی پیشنهادی

services:
  backend:
    environment:
      APP_ENV: production
      DB_HOST: database
      DB_PORT: "5432"

فرم <span dir="ltr">List</span> هم وجود دارد:

environment:
  - APP_ENV=production
  - DB_HOST=database

برای خوانایی، در این سند از فرم <span dir="ltr">Mapping</span> استفاده می‌کنیم.

<a id="dot-env" name="dot-env"></a>

12. <code dir="ltr">.env</code> و <span dir="ltr">Variable Interpolation</span>

فایل <code dir="ltr">.env</code>

APP_PORT=8080
IMAGE_TAG=1.0.0
APP_ENV=development

استفاده در <code dir="ltr">compose.yaml</code>

services:
  backend:
    image: "my-backend:${IMAGE_TAG}"
    ports:
      - "${APP_PORT}:4000"
    environment:
      APP_ENV: ${APP_ENV}

بررسی مقدار نهایی

docker compose config

الگوی Repository

.env          -> keep local / gitignored
.env.example  -> commit as template

نمونه <code dir="ltr">.env.example</code>:

APP_PORT=8080
IMAGE_TAG=CHANGE_ME
APP_ENV=development

<a id="env-file" name="env-file"></a>

13. <code dir="ltr">env_file</code>

وقتی می‌خواهی متغیرهای یک فایل وارد <span dir="ltr">Environment</span> خود <span dir="ltr">Container</span> شوند:

services:
  backend:
    env_file:
      - .env

برای چند فایل:

services:
  backend:
    env_file:
      - .env
      - .env.local

تفاوتی که باید یادت بماند:

<table dir="rtl">
  <thead><tr><th>ساختار</th><th>نقش</th></tr></thead>
  <tbody>
    <tr><td><code dir="ltr">${VAR}</code></td><td>جایگزینی مقدار هنگام پردازش فایل Compose</td></tr>
    <tr><td><code dir="ltr">environment:</code></td><td>تعریف Environment Variable برای Container</td></tr>
    <tr><td><code dir="ltr">env_file:</code></td><td>خواندن Environment Variableهای Container از فایل</td></tr>
  </tbody>
</table>

<a id="named-volume" name="named-volume"></a>

14. <span dir="ltr">Named Volume</span>

برای داده‌ای که باید مستقل از عمر <span dir="ltr">Container</span> بماند، مثل داده پایگاه‌داده:

آماده برای کپی

services:
  database:
    image: postgres:17
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:

چه چیزهایی را تغییر بدهم؟

<table dir="rtl">
  <thead><tr><th>مقدار</th><th>معنی</th></tr></thead>
  <tbody>
    <tr><td><code dir="ltr">db-data</code></td><td>نام Volume</td></tr>
    <tr><td><code dir="ltr">/var/lib/postgresql/data</code></td><td>مسیر داده داخل Container</td></tr>
  </tbody>
</table>

نکته: اگر نام <span dir="ltr">Volume</span> را سمت چپ استفاده کردی، همان نام را پایین فایل زیر <code dir="ltr">volumes:</code> تعریف کن.

<a id="bind-mount" name="bind-mount"></a>

15. <span dir="ltr">Bind Mount</span>

برای اتصال فایل یا پوشه واقعی <span dir="ltr">Host</span> به داخل <span dir="ltr">Container</span>؛ معمولاً در <span dir="ltr">Development</span>:

پوشه

services:
  backend:
    volumes:
      - ./backend:/app

یک فایل

services:
  nginx:
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf

<table dir="rtl">
  <thead><tr><th>سمت</th><th>معنی</th></tr></thead>
  <tbody>
    <tr><td>چپ</td><td>مسیر روی Host</td></tr>
    <tr><td>راست</td><td>مسیر داخل Container</td></tr>
  </tbody>
</table>

<a id="readonly-mount" name="readonly-mount"></a>

16. <span dir="ltr">Read-only Mount</span>

اگر <span dir="ltr">Container</span> فقط باید فایل را بخواند:

services:
  nginx:
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro

<code dir="ltr"></code> یعنی <span dir="ltr">Read-only</span>.

این الگو برای فایل‌های تنظیماتی که <span dir="ltr">Container</span> نباید تغییرشان دهد، خواناتر و امن‌تر است.

<a id="anonymous-volume" name="anonymous-volume"></a>

17. <span dir="ltr">Anonymous Volume</span>

ممکن است چنین چیزی ببینی:

services:
  backend:
    volumes:
      - /app/node_modules

اینجا سمت چپ نام مشخصی برای Volume وجود ندارد.

برای شروع، Named Volume و Bind Mount را اول کامل بفهم. <span dir="ltr">Anonymous Volume</span> را وقتی استفاده کن که دلیل استفاده‌اش در معماری پروژه روشن است.

<a id="network" name="network"></a>

18. <span dir="ltr">Network</span> اختصاصی

آماده برای کپی

services:
  backend:
    networks:
      - app-net

  database:
    networks:
      - app-net

networks:
  app-net:

اگر هر دو سرویس عضو یک <span dir="ltr">Network</span> باشند، Backend می‌تواند Database را با نام Service پیدا کند.

environment:
  DB_HOST: database

از <span dir="ltr">IP</span> ثابت داخلی استفاده نکن:

DB_HOST: "172.20.0.4"

<a id="two-networks" name="two-networks"></a>

19. تفکیک <span dir="ltr">Frontend</span> و <span dir="ltr">Backend Network</span>

الگوی قابل کپی برای سه سرویس:

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

مدل ارتباط:

Nginx
  |
  v
frontend-net
  |
  v
Backend
  |
  v
backend-net
  |
  v
Database

<a id="depends-on" name="depends-on"></a>

20. <code dir="ltr">depends_on</code>

وابستگی ساده:

services:
  backend:
    depends_on:
      - database

  database:
    image: postgres:17

این الگو ترتیب وابستگی را بیان می‌کند، اما Start شدن Container الزاماً به معنی Ready بودن سرویس داخل آن نیست. اگر Readiness مهم است، الگوی بخش بعد را ببین.

<a id="healthcheck" name="healthcheck"></a>

21. <code dir="ltr">healthcheck</code> و <code dir="ltr">service_healthy</code>

PostgreSQL — آماده برای کپی

services:
  database:
    image: postgres:17
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: dev-secret
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 5s
      timeout: 3s
      retries: 10

  backend:
    image: my-backend:1.0.0
    depends_on:
      database:
        condition: service_healthy

چیزهایی که باید هماهنگ تغییر بدهی

<code dir="ltr">POSTGRES_DB</code>

<code dir="ltr">POSTGRES_USER</code>

مقدارهای همان User/Database در <code dir="ltr">pg_isready</code>

اگر یکی را تغییر دادی و دیگری را جا انداختی، Healthcheck می‌تواند Fail شود.

<a id="restart-command" name="restart-command"></a>

22. <code dir="ltr">restart</code>، <code dir="ltr">command</code> و <code dir="ltr">entrypoint</code>

Restart Policy

restart: unless-stopped

یا:

restart: on-failure

Override کردن <span dir="ltr">CMD</span>

command: ["python", "app.py"]

Override کردن <span dir="ltr">ENTRYPOINT</span>

entrypoint: ["/app/start.sh"]

اگر <code dir="ltr">Dockerfile</code> رفتار اجرایی درست را تعریف کرده است، بدون دلیل مشخص <code dir="ltr">command</code> یا <code dir="ltr">entrypoint</code> را Override نکن.

<a id="anchors" name="anchors"></a>

23. <span dir="ltr">Anchor</span> و <span dir="ltr">Reuse</span>

وقتی چند سرویس تنظیمات مشترک دارند:

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

معنی علامت‌ها:

&common-env  = anchor
*common-env  = alias
<<:          = merge

این الگو را فقط وقتی استفاده کن که واقعاً تکرار را کم و خوانایی را بیشتر می‌کند.

<a id="template-nginx" name="template-nginx"></a>

24. قالب کامل 1 — <span dir="ltr">Nginx</span>

<code dir="ltr">compose.yaml</code>

services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"

اجرا

docker compose config
docker compose up -d

تست

curl http://localhost:8080

توقف

docker compose down

<a id="template-backend-db" name="template-backend-db"></a>

25. قالب کامل 2 — <span dir="ltr">Backend + PostgreSQL</span>

آماده برای کپی

services:
  backend:
    image: my-backend:1.0.0
    ports:
      - "8080:4000"
    environment:
      DB_HOST: database
      DB_PORT: "5432"
      DB_NAME: appdb
      DB_USER: appuser
      DB_PASSWORD: dev-password
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

قبل از استفاده این‌ها را تغییر بده

<table dir="rtl">
  <thead><tr><th>مقدار نمونه</th><th>جایگزین پروژه</th></tr></thead>
  <tbody>
    <tr><td><code dir="ltr">my-backend:1.0.0</code></td><td>Image واقعی Backend</td></tr>
    <tr><td><code dir="ltr">8080:4000</code></td><td>Port Mapping برنامه</td></tr>
    <tr><td><code dir="ltr">appdb</code></td><td>نام Database</td></tr>
    <tr><td><code dir="ltr">appuser</code></td><td>User پایگاه‌داده</td></tr>
    <tr><td><code dir="ltr">dev-password</code></td><td>برای پروژه واقعی از Secret/.env بگیر</td></tr>
  </tbody>
</table>

<a id="template-env-db" name="template-env-db"></a>

26. قالب کامل 3 — <span dir="ltr">Backend + PostgreSQL + .env</span>

این نسخه برای الگوبرداری بهتر از قرار دادن مستقیم Password داخل Compose است.

<code dir="ltr">.env</code>

APP_PORT=8080
APP_ENV=development
BACKEND_IMAGE=my-backend:1.0.0
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=CHANGE_ME

<code dir="ltr">compose.yaml</code>

services:
  backend:
    image: "${BACKEND_IMAGE}"
    ports:
      - "${APP_PORT}:4000"
    environment:
      APP_ENV: ${APP_ENV}
      DB_HOST: database
      DB_PORT: "5432"
      DB_NAME: ${POSTGRES_DB}
      DB_USER: ${POSTGRES_USER}
      DB_PASSWORD: ${POSTGRES_PASSWORD}
    depends_on:
      - database

  database:
    image: postgres:17
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:

<code dir="ltr">.env.example</code>

APP_PORT=8080
APP_ENV=development
BACKEND_IMAGE=my-backend:1.0.0
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=CHANGE_ME

بررسی

docker compose config

<code dir="ltr">.env.example</code> را می‌توانی به‌عنوان الگو در Repository نگه داری؛ مقدار واقعی Secret را داخل فایل نمونه قرار نده.

<a id="template-dev" name="template-dev"></a>

27. قالب کامل 4 — محیط <span dir="ltr">Development</span> با <span dir="ltr">Bind Mount</span>

وقتی سورس روی Host است و Container باید تغییرهای آن را ببیند:

services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "4000:4000"
    volumes:
      - ./backend:/app
    environment:
      APP_ENV: development

تغییر بده

<code dir="ltr">./backend</code> → مسیر سورس روی Host

<code dir="ltr">/app</code> → مسیر کاری برنامه داخل Container

<code dir="ltr">4000:4000</code> → پورت‌های پروژه

نکته مهم

Mount کردن یک پوشه روی <code dir="ltr">/app</code> می‌تواند فایل‌هایی را که قبلاً در همان مسیر داخل Image بوده‌اند، پشت Mount پنهان کند. مسیر را آگاهانه انتخاب کن.

<a id="template-networks" name="template-networks"></a>

28. قالب کامل 5 — سه سرویس با دو <span dir="ltr">Network</span>

services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
    networks:
      - frontend-net

  backend:
    image: my-backend:1.0.0
    environment:
      DB_HOST: database
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

در این الگو:

<span dir="ltr">Nginx</span> و <span dir="ltr">Backend</span> یک Network مشترک دارند.

<span dir="ltr">Backend</span> و <span dir="ltr">Database</span> Network دیگری دارند.

<span dir="ltr">Nginx</span> مستقیماً عضو <code dir="ltr">backend-net</code> نیست.

<a id="override" name="override"></a>

29. <span dir="ltr">Compose Override</span> برای <span dir="ltr">Development</span>

فایل پایه — <code dir="ltr">compose.yaml</code>

services:
  backend:
    build: ./backend
    environment:
      APP_ENV: production

فایل توسعه — <code dir="ltr">compose.dev.yaml</code>

services:
  backend:
    environment:
      APP_ENV: development
    volumes:
      - ./backend:/app

اجرای ترکیبی

docker compose \
  -f compose.yaml \
  -f compose.dev.yaml \
  up -d

دیدن نتیجه Merge شده

docker compose \
  -f compose.yaml \
  -f compose.dev.yaml \
  config

برای پروژه کوچک، تعداد زیاد فایل Override می‌تواند از سادگی پروژه کم کند؛ فقط وقتی واقعاً لازم است از این الگو استفاده کن.

<a id="validation" name="validation"></a>

30. <span dir="ltr">Validate</span> و <span dir="ltr">Debug</span>

مهم‌ترین دستور قبل از اجرا

docker compose config

این دستور برای این کارها بسیار مفید است:

Parse کردن فایل

Resolve کردن Variableها

Merge کردن فایل‌های Compose

نمایش Configuration نهایی

پیدا کردن بسیاری از خطاهای ساختاری

وضعیت سرویس‌ها

docker compose ps

همه لاگ‌ها

docker compose logs -f

لاگ یک سرویس

docker compose logs -f backend

ورود به Container یک Service

docker compose exec backend sh

اگر Image دارای Bash باشد:

docker compose exec backend bash

<a id="errors" name="errors"></a>

31. سه نوع خطای مهم

1. خطای <span dir="ltr">YAML Syntax</span>

services:
   backend:
    image: my-backend

مشکل: <span dir="ltr">Indentation</span>.

2. فایل YAML درست است ولی <span dir="ltr">Compose Schema</span> غلط است

services:
  backend:
    pizza: large

این فایل از نظر ساختار YAML قابل Parse است، اما <code dir="ltr">pizza</code> کلید معتبر Compose برای این Service نیست.

3. <span dir="ltr">Runtime Error</span>

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"

YAML و Compose می‌توانند درست باشند، اما مثلاً Port <code dir="ltr">8080</code> روی Host قبلاً اشغال باشد.

ترتیب بررسی:

YAML syntax
    |
    v
Compose schema/config
    |
    v
Docker runtime
    |
    v
Application runtime

<a id="copy-checklist" name="copy-checklist"></a>

32. <span dir="ltr">Copy Checklist</span> قبل از استفاده

وقتی یک نمونه را از این فایل Copy کردی، قبل از اجرا این موارد را چک کن:

<ul dir="rtl">
  <li>☐ نام <span dir="ltr">Service</span>ها را با پروژه خودم هماهنگ کرده‌ام.</li>
  <li>☐ <span dir="ltr">Image</span> یا مسیر <code dir="ltr">build</code> درست است.</li>
  <li>☐ <span dir="ltr">Host Port</span> و <span dir="ltr">Container Port</span> را برعکس ننوشته‌ام.</li>
  <li>☐ مسیرهای <span dir="ltr">Bind Mount</span> واقعاً روی Host وجود دارند.</li>
  <li>☐ مسیر سمت راست Volume مربوط به همان برنامه/Image است.</li>
  <li>☐ نام Volumeهای استفاده‌شده پایین فایل تعریف شده‌اند.</li>
  <li>☐ نام Networkهای استفاده‌شده پایین فایل تعریف شده‌اند.</li>
  <li>☐ برای ارتباط داخلی از نام Service استفاده کرده‌ام، نه IP ثابت.</li>
  <li>☐ Secret واقعی داخل فایل Commit‌شونده نگذاشته‌ام.</li>
  <li>☐ متغیرهای <code dir="ltr">${VAR}</code> مقدار دارند.</li>
  <li>☐ در Healthcheck نام User و Database با Environment هماهنگ است.</li>
  <li>☐ قبل از اجرا <code dir="ltr">docker compose config</code> را اجرا کرده‌ام.</li>
</ul>

<a id="cheatsheet" name="cheatsheet"></a>

33. <span dir="ltr">Cheat Sheet</span> نهایی

ساختار YAML

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

Service با Image

services:
  web:
    image: nginx:alpine

Service با Build

services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile

Port

ports:
  - "8080:80"

Environment

environment:
  APP_ENV: production
  DB_HOST: database

Variable از <code dir="ltr">.env</code>

image: "myapp:${IMAGE_TAG}"

IMAGE_TAG=1.0.0

Named Volume

services:
  database:
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:

Bind Mount

volumes:
  - ./backend:/app

Read-only

volumes:
  - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro

Network

services:
  backend:
    networks:
      - app-net

networks:
  app-net:

depends_on

depends_on:
  - database

Healthcheck

healthcheck:
  test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
  interval: 5s
  timeout: 3s
  retries: 10

Restart

restart: unless-stopped

دستورات پایه

docker compose config
docker compose up -d
docker compose ps
docker compose logs -f
docker compose down

جمع‌بندی روش کار

برای کار روزمره لازم نیست تمام <span dir="ltr">YAML</span> را از حفظ بنویسی. مهم‌تر این است که:

ساختار <span dir="ltr">Mapping / List / Indentation</span> را بفهمی.

یک الگوی سالم و نزدیک به نیازت پیدا کنی.

همان الگو را Copy کنی و فقط بخش‌های لازم را تغییر بدهی.

با <code dir="ltr">docker compose config</code> نتیجه را بررسی کنی.

بعد سراغ اجرای پروژه بروی.

این همان روشی است که این سند بر اساس آن چیده شده: کمتر حفظ کن، بیشتر الگوی درست را بخوان، Copy کن، تغییر بده و Validate کن.

</div>