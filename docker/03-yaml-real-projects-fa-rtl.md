<div dir="rtl" align="right">

قواعد <span dir="ltr">YAML</span> برای <span dir="ltr">Docker Compose</span> و پروژه‌های واقعی <span dir="ltr">DevOps</span>

این سند <span dir="ltr">YAML</span> را به‌عنوان یک زبان مستقل آموزش می‌دهد، اما تمرکز اصلی روی جایی است که یک <span dir="ltr">Junior</span> بیشترین برخورد را با آن دارد: <code dir="ltr">Docker Compose</code> و <span dir="ltr">Configuration</span> پروژه‌های واقعی.

هدف این نیست که ده‌ها قابلیت کم‌استفاده <span dir="ltr">YAML</span> را حفظ کنیم. هدف این است که بتوانیم یک فایل واقعی را بخوانیم، بنویسیم، <span dir="ltr">Validate</span> کنیم و خطایش را پیدا کنیم.

<a id="toc" name="toc"></a>

فهرست مطالب

<ol dir="rtl" align="right">
  <li><a href="#what"><span dir="ltr">YAML</span> چیست و چه چیزی نیست؟</a></li>
  <li><a href="#syntax-schema"><span dir="ltr">Syntax</span> با <span dir="ltr">Schema</span> فرق دارد</a></li>
  <li><a href="#structures">سه ساختار اصلی <span dir="ltr">YAML</span></a></li>
  <li><a href="#indentation"><span dir="ltr">Indentation</span></a></li>
  <li><a href="#colon-dash"><span dir="ltr">Colon</span> و <span dir="ltr">Dash</span></a></li>
  <li><a href="#values">نوع مقدارها و <span dir="ltr">Quote</span></a></li>
  <li><a href="#comment"><span dir="ltr">Comment</span></a></li>
  <li><a href="#multiline">متن چندخطی با | و &gt;</a></li>
  <li><a href="#flow"><span dir="ltr">Flow Style</span> با <code dir="ltr">{}</code> و <code dir="ltr">[]</code></a></li>
  <li><a href="#anchors"><span dir="ltr">Anchor</span>، <span dir="ltr">Alias</span> و <span dir="ltr">Merge</span></a></li>
  <li><a href="#reading">روش خواندن یک <span dir="ltr">YAML</span> پیچیده</a></li>
  <li><a href="#compose-base">ساختار پایه <span dir="ltr">compose.yaml</span></a></li>
  <li><a href="#services"><span dir="ltr">services</span></a></li>
  <li><a href="#build-image"><span dir="ltr">build</span> و <span dir="ltr">image</span></a></li>
  <li><a href="#ports"><span dir="ltr">ports</span></a></li>
  <li><a href="#env"><span dir="ltr">environment</span>، .<span dir="ltr">env</span> و <span dir="ltr">env_file</span></a></li>
  <li><a href="#volumes"><span dir="ltr">volumes</span> و انواع <span dir="ltr">Mount</span></a></li>
  <li><a href="#networks"><span dir="ltr">networks</span></a></li>
  <li><a href="#health"><span dir="ltr">depends_on</span> و <span dir="ltr">healthcheck</span></a></li>
  <li><a href="#runtime"><span dir="ltr">restart</span>، <span dir="ltr">command</span> و <span dir="ltr">entrypoint</span></a></li>
  <li><a href="#secrets"><span dir="ltr">Secret</span>ها را کجا نگذاریم؟</a></li>
  <li><a href="#scenario1">سناریو 1: <span dir="ltr">Nginx</span> تک‌سرویسی</a></li>
  <li><a href="#scenario2">سناریو 2: <span dir="ltr">Backend</span> + <span dir="ltr">Database</span></a></li>
  <li><a href="#scenario3">سناریو 3: <span dir="ltr">Development</span> با <span dir="ltr">Bind Mount</span></a></li>
  <li><a href="#scenario4">سناریو 4: <span dir="ltr">Network</span> اختصاصی</a></li>
  <li><a href="#scenario5">سناریو 5: .<span dir="ltr">env</span> برای <span dir="ltr">Environment</span>های مختلف</a></li>
  <li><a href="#scenario6">سناریو 6: <span dir="ltr">Healthcheck</span> واقعی</a></li>
  <li><a href="#override"><span dir="ltr">Compose</span> چندفایلی و <span dir="ltr">Override</span></a></li>
  <li><a href="#reuse"><span dir="ltr">Reuse</span> با <span dir="ltr">x-</span> و <span dir="ltr">Anchor</span></a></li>
  <li><a href="#error-types">سه نوع خطا: <span dir="ltr">YAML</span>، <span dir="ltr">Compose Schema</span> و <span dir="ltr">Runtime</span></a></li>
  <li><a href="#validation"><span dir="ltr">Validation</span> و <span dir="ltr">Debug</span></a></li>
  <li><a href="#style"><span dir="ltr">Style Guide</span> پیشنهادی</a></li>
  <li><a href="#practice">تمرین مرحله‌ای</a></li>
  <li><a href="#cheatsheet"><span dir="ltr">Cheat Sheet</span></a></li>
</ol>

<a id="what" name="what"></a>

1. <span dir="ltr">YAML</span> چیست و چه چیزی نیست؟

<code dir="ltr">YAML</code> روشی برای نوشتن داده ساختاریافته است.

name: Amir
age: 22
enabled: true

برای انسان:

<pre dir="ltr"><code>name    → Amir
age     → 22
enabled → true</code></pre>

برای برنامه چیزی شبیه <span dir="ltr">Object</span> / <span dir="ltr">Dictionary</span> است.

<span dir="ltr">YAML</span> خودش <span dir="ltr">Docker</span> را اجرا نمی‌کند، <span dir="ltr">Network</span> نمی‌سازد و <span dir="ltr">Container</span> ایجاد نمی‌کند. فقط <span dir="ltr">Data</span> را با <span dir="ltr">Syntax</span> مشخص بیان می‌کند.

این <span dir="ltr">Data</span> می‌تواند توسط ابزارهای مختلف خوانده شود:

<pre dir="ltr"><code>Docker Compose
GitHub Actions
Kubernetes
Ansible
CI/CD systems</code></pre>

مهارت اصلی:

<pre dir="ltr"><code>YAML Syntax
     +
Tool Schema</code></pre>

<a id="syntax-schema" name="syntax-schema"></a>

2. <span dir="ltr">Syntax</span> با <span dir="ltr">Schema</span> فرق دارد

این یکی از مهم‌ترین مفاهیم کل سند است.

services:
  web:
    image: nginx:alpine

<span dir="ltr">YAML</span> فقط ساختار را می‌فهمد:

<pre dir="ltr"><code>services
└── web
    └── image: nginx:alpine</code></pre>

اما معنی <code dir="ltr">services</code> و <code dir="ltr">image</code> را <span dir="ltr">Docker Compose</span> تعیین می‌کند.

این <span dir="ltr">YAML</span> هم از نظر <span dir="ltr">Syntax</span> معتبر است:

pizza:
  cheese: lots
  olives:
    - black
    - green

ولی <span dir="ltr">Compose</span> چنین <span dir="ltr">Schema</span>ای ندارد.

پس یک فایل می‌تواند:

<pre dir="ltr"><code>YAML-valid
    but
Compose-invalid</code></pre>

باشد.

مدل خطا:

<pre dir="ltr"><code>Layer 1: YAML Syntax
  ↓
indentation, :, -, list, mapping

Layer 2: Tool Schema
  ↓
services, image, ports, volumes ...

Layer 3: Runtime
  ↓
image pull, port conflict, app crash ...</code></pre>

<a id="structures" name="structures"></a>

3. سه ساختار اصلی <span dir="ltr">YAML</span>

تقریباً بیشتر <span dir="ltr">YAML</span>هایی که در <span dir="ltr">DevOps</span> می‌بینی از سه نوع اصلی ساخته شده‌اند.

3.1. <span dir="ltr">Scalar</span>

name: backend
replicas: 2
enabled: true
timeout: 2.5
optional: null

3.2. <span dir="ltr">Mapping</span>

database:
  host: database
  port: 5432

<pre dir="ltr"><code>database
├── host → database
└── port → 5432</code></pre>

3.3. <span dir="ltr">Sequence</span>

ports:
  - "8080:80"
  - "8443:443"

ترکیب

services:
  backend:
    environment:
      APP_ENV: production
    ports:
      - "8080:4000"

<pre dir="ltr"><code>services                 Mapping
└── backend              Mapping
    ├── environment      Mapping
    │   └── APP_ENV      Scalar
    └── ports            Sequence
        └── "8080:4000"  Scalar</code></pre>

اگر بتوانی <span dir="ltr">YAML</span> را به چنین درختی تبدیل کنی، فایل‌های بزرگ خیلی قابل‌فهم‌تر می‌شوند.

<a id="indentation" name="indentation"></a>

4. <span dir="ltr">Indentation</span>

فاصله ابتدای خط در <span dir="ltr">YAML</span> معنی ساختاری دارد.

در این سند از دو <span dir="ltr">Space</span> برای هر <span dir="ltr">Level</span> استفاده می‌کنیم:

<pre dir="ltr"><code>Level 0 → 0 spaces
Level 1 → 2 spaces
Level 2 → 4 spaces
Level 3 → 6 spaces</code></pre>

درست:

services:
  backend:
    image: my-backend
    environment:
      APP_ENV: production

ساختار:

<pre dir="ltr"><code>services
└── backend
    ├── image
    └── environment
        └── APP_ENV</code></pre>

نامنظم:

services:
   backend:
    image: my-backend

ممکن است <span dir="ltr">Error</span> یا ساختار اشتباه ایجاد کند.

قاعده عملی:

<ul dir="rtl">
  <li>برای <span dir="ltr">Indentation</span> از <span dir="ltr">Space</span> استفاده کن.</li>
  <li><span dir="ltr">Editor</span> را طوری تنظیم کن که <span dir="ltr">Tab</span> در فایل‌های <span dir="ltr">YAML</span> به <span dir="ltr">Space</span> تبدیل شود.</li>
</ul>

<span dir="ltr">YAML</span> الزام نمی‌کند همیشه دقیقاً دو <span dir="ltr">Space</span> استفاده شود، اما <span dir="ltr">Consistency</span> مهم است.

<a id="colon-dash" name="colon-dash"></a>

5. <span dir="ltr">Colon</span> و <span dir="ltr">Dash</span>

<code dir="ltr">:</code>

<span dir="ltr">Key</span> را از <span dir="ltr">Value</span> جدا می‌کند:

name: backend

اگر بعد از <span dir="ltr">Colon</span> مقدار مستقیم نباشد:

database:

یعنی زیر آن <span dir="ltr">Structure</span> دیگری می‌آید:

database:
  host: db
  port: 5432

<code dir="ltr">-</code>

<span dir="ltr">Dash</span> + <span dir="ltr">Space</span> معمولاً یک عضو <span dir="ltr">List</span> می‌سازد:

networks:
  - frontend
  - backend

<span dir="ltr">List</span> می‌تواند از <span dir="ltr">Mapping</span> تشکیل شود:

servers:
  - name: api-1
    port: 3000

  - name: api-2
    port: 3001

قاعده خواندن:

<p dir="rtl">هر وقت در ساختار بلوکی <span dir="ltr">YAML</span> یک <code dir="ltr">-</code> در محل درست دیدی، معمولاً یک عضو جدید از <span dir="ltr">List / Sequence</span> شروع شده است.</p>

<a id="values" name="values"></a>

6. نوع مقدارها و <span dir="ltr">Quote</span>

name: backend
replicas: 2
timeout: 1.5
debug: true
cache: false
optional: null

مقدار

نوع تقریبی

<code dir="ltr">backend</code>

<span dir="ltr">String</span>

<code dir="ltr">2</code>

<span dir="ltr">Integer</span>

<code dir="ltr">1.5</code>

<span dir="ltr">Float</span>

<code dir="ltr">true</code>

<span dir="ltr">Boolean</span>

<code dir="ltr">null</code>

<span dir="ltr">Null</span>

برای <span dir="ltr">Value</span>هایی که باید قطعاً <span dir="ltr">String</span> بمانند، <span dir="ltr">Quote</span> مفید است:

version: "1.10"
phone_like: "01234"
answer: "yes"
mode: "on"

<span dir="ltr">Port Mapping:</span>

ports:
  - "8080:80"

<span dir="ltr">Image Tag:</span>

image: "myapp:1.4.2"

<span dir="ltr">URL</span> و <span dir="ltr">Cron:</span>

url: "https://example.com/api"
schedule: "0 3 * * *"

<span dir="ltr">Password</span> با <code dir="ltr">#</code>:

password: "abc#123"

قاعده امن:

<p dir="rtl">اگر یک مقدار از نظر ظاهری ممکن است <span dir="ltr">Number</span>، <span dir="ltr">Boolean</span>، <span dir="ltr">Date</span> یا بخشی از <span dir="ltr">Syntax</span> تعبیر شود، اما برای پروژه باید <span dir="ltr">String</span> باقی بماند، آن را داخل <span dir="ltr">Quote</span> قرار بده.</p>

<a id="comment" name="comment"></a>

7. <span dir="ltr">Comment</span>

# Application configuration
name: backend

<span dir="ltr">Inline:</span>

port: 3000 # internal application port

اگر <code dir="ltr">#</code> بخشی از <span dir="ltr">Value</span> است <span dir="ltr">Quote</span> بگذار:

password: "abc#123"

<span dir="ltr">Comment</span> باید <span dir="ltr">Context</span> یا دلیل بدهد، نه این‌که فقط همان خط را تکرار کند.

ضعیف:

ports:
  - "8080:80" # port

بهتر:

ports:
  - "8080:80" # local public entrypoint

<a id="multiline" name="multiline"></a>

8. متن چندخطی با <code dir="ltr">|</code> و <code dir="ltr">></code>

<code dir="ltr">|</code>

<span dir="ltr">Line break</span>ها را حفظ می‌کند:

message: |
  line one
  line two
  line three

<code dir="ltr">></code>

خط‌ها را بیشتر به شکل متن پیوسته <span dir="ltr">Fold</span> می‌کند:

description: >
  This is a long description
  written across multiple lines.

در <span dir="ltr">Compose</span> روزمره کمتر لازم می‌شود، اما در <span dir="ltr">CI/CD</span> و <span dir="ltr">Config</span>ها ممکن است ببینی.

<a id="flow" name="flow"></a>

9. <span dir="ltr">Flow Style</span> با <code dir="ltr">{}</code> و <code dir="ltr">[]</code>

فرم فشرده:

user: {name: Ali, role: backend}
ports: ["80", "443"]

فرم <span dir="ltr">Block</span> معمولاً خواناتر است:

user:
  name: Ali
  role: backend

ports:
  - "80"
  - "443"

<span dir="ltr">Flow Style</span> را وقتی استفاده کن که واقعاً خوانایی بهتر شود.

<a id="anchors" name="anchors"></a>

10. <span dir="ltr">Anchor</span>، <span dir="ltr">Alias</span> و <span dir="ltr">Merge</span>

برای <span dir="ltr">Reuse:</span>

x-common-env: &common-env
  LOG_LEVEL: info
  TZ: UTC

استفاده:

services:
  api:
    environment:
      <<: *common-env
      APP_NAME: api

  worker:
    environment:
      <<: *common-env
      APP_NAME: worker

<pre dir="ltr"><code>&amp;common-env → Anchor
*common-env → Alias
&lt;&lt;:         → Merge</code></pre>

این قابلیت مفید است، اما اگر خوانایی را برای تیم کم کند، ساده‌تر نوشتن بهتر است.

<a id="reading" name="reading"></a>

11. روش خواندن یک <span dir="ltr">YAML</span> پیچیده

مثال:

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

به درخت تبدیلش کن:

<pre dir="ltr"><code>services
└── backend
    ├── build
    │   └── context: ./backend
    ├── ports [list]
    │   └── "8080:4000"
    ├── environment
    │   └── DB_HOST: database
    └── networks [list]
        └── app-net</code></pre>

چهار سؤال:

<ol dir="rtl">
  <li><span dir="ltr">Parent</span> این خط چیست؟</li>
  <li>ساختار فعلی <span dir="ltr">Mapping</span> است یا <span dir="ltr">List</span>؟</li>
  <li><span dir="ltr">Value</span> یک <span dir="ltr">Scalar</span> است یا یک ساختار تو‌در‌تو؟</li>
  <li>معنی این <span dir="ltr">Key</span> را خود <span dir="ltr">YAML</span> تعیین می‌کند یا ابزار مصرف‌کننده مثل <span dir="ltr">Docker Compose</span>؟</li>
</ol>

<a id="compose-base" name="compose-base"></a>

12. ساختار پایه <code dir="ltr">compose.yaml</code>

کوچک‌ترین نمونه کاربردی:

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"

کامل‌تر:

services:
  backend:
    build: ./backend

  database:
    image: postgres:17

volumes:
  db-data:

networks:
  app-net:

<span dir="ltr">Root Key</span>های پرتکرار:

<pre dir="ltr"><code>services
volumes
networks</code></pre>

در <span dir="ltr">Compose</span> جدید معمولاً نیازی به <code dir="ltr">version:</code> قدیمی نیست.

<a id="services" name="services"></a>

13. <code dir="ltr">services</code>

<code dir="ltr">services</code> یک <span dir="ltr">Mapping</span> است:

services:
  backend:
    image: my-backend

  database:
    image: postgres:17

<pre dir="ltr"><code>services
├── backend
└── database</code></pre>

نام <span dir="ltr">Service</span> در <span dir="ltr">DNS</span> داخلی <span dir="ltr">Compose</span> مهم است. <span dir="ltr">Backend</span> می‌تواند <span dir="ltr">Database</span> را با نام <code dir="ltr">database</code> پیدا کند.

<a id="build-image" name="build-image"></a>

14. <code dir="ltr">build</code> و <code dir="ltr">image</code>

<span dir="ltr">Image</span> آماده:

services:
  nginx:
    image: nginx:alpine

<span dir="ltr">Build</span> از <span dir="ltr">Dockerfile:</span>

services:
  backend:
    build: ./backend

فرم واضح‌تر:

services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile

<span dir="ltr">Build</span> و <span dir="ltr">Tag:</span>

services:
  backend:
    build:
      context: ./backend
    image: my-backend:1.0.0

<table dir="rtl">
  <thead><tr><th><span dir="ltr">Key</span></th><th>معنی در <span dir="ltr">Compose</span></th></tr></thead>
  <tbody>
    <tr><td><code dir="ltr">image</code></td><td>از یک <span dir="ltr">Image</span> مشخص استفاده کن.</td></tr>
    <tr><td><code dir="ltr">build</code></td><td><span dir="ltr">Image</span> را از سورس و <code dir="ltr">Dockerfile</code> بساز.</td></tr>
  </tbody>
</table>

<a id="ports" name="ports"></a>

15. <code dir="ltr">ports</code>

ports:
  - "8080:80"

<pre dir="ltr"><code>Host :8080
   ↓
Container :80</code></pre>

اگر <span dir="ltr">Application</span> داخل <span dir="ltr">Container</span> روی <code dir="ltr">4000</code> است:

ports:
  - "9000:4000"

از بیرون <code dir="ltr">localhost:9000</code> و داخل <span dir="ltr">Container Port</span> <code dir="ltr">4000</code> است.

لزومی ندارد همه <span dir="ltr">Service</span>ها <span dir="ltr">Port</span> عمومی داشته باشند. <span dir="ltr">Database</span> داخلی می‌تواند فقط روی <span dir="ltr">Network Compose</span> در دسترس <span dir="ltr">Backend</span> باشد.

<a id="env" name="env"></a>

16. <code dir="ltr">environment</code>، <code dir="ltr">.env</code> و <code dir="ltr">env_file</code>

<code dir="ltr">environment</code>

<span dir="ltr">Variable</span> داخل <span dir="ltr">Container:</span>

services:
  backend:
    environment:
      APP_ENV: production
      DB_HOST: database

فرم <span dir="ltr">List</span> هم ممکن است دیده شود:

environment:
  - APP_ENV=production
  - DB_HOST=database

برای خوانایی، <span dir="ltr">Mapping</span> معمولاً واضح‌تر است.

<code dir="ltr">.env</code> و <span dir="ltr">Interpolation</span>

<code dir="ltr">.env</code>:

APP_PORT=8080
IMAGE_TAG=1.0.0

<span dir="ltr">Compose:</span>

services:
  backend:
    image: "my-backend:${IMAGE_TAG}"
    ports:
      - "${APP_PORT}:4000"

برای دیدن نتیجه <span dir="ltr">Resolve</span> شده:

docker compose config

<code dir="ltr">env_file</code>

services:
  backend:
    env_file:
      - .env

<span dir="ltr">Variable</span>های فایل وارد <span dir="ltr">Environment Container</span> می‌شوند.

مدل:

<table dir="rtl">
  <thead><tr><th>ساختار</th><th>نقش</th></tr></thead>
  <tbody>
    <tr><td><code dir="ltr">${VAR}</code> در <code dir="ltr">compose.yaml</code></td><td><span dir="ltr">Interpolation</span> هنگام پردازش تنظیمات <span dir="ltr">Compose</span>.</td></tr>
    <tr><td><code dir="ltr">environment:</code></td><td>تعریف <span dir="ltr">Environment Variable</span>هایی که به <span dir="ltr">Container</span> داده می‌شوند.</td></tr>
    <tr><td><code dir="ltr">env_file:</code></td><td>خواندن متغیرهای محیطی <span dir="ltr">Container</span> از یک فایل.</td></tr>
  </tbody>
</table>

<code dir="ltr">.env.example</code> را <span dir="ltr">Commit</span> کن و <code dir="ltr">.env</code> واقعی را در <code dir="ltr">.gitignore</code> قرار بده.

<a id="volumes" name="volumes"></a>

17. <code dir="ltr">volumes</code> و انواع <span dir="ltr">Mount</span>

<span dir="ltr">Named Volume</span>

services:
  database:
    image: postgres:17
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:

سمت چپ <code dir="ltr">db-data</code> <span dir="ltr">Volume Docker</span> و سمت راست مسیر داخل <span dir="ltr">Container</span> است.

<span dir="ltr">Bind Mount</span>

services:
  backend:
    volumes:
      - ./backend:/app

سمت چپ مسیر <span dir="ltr">Host</span> و سمت راست مسیر <span dir="ltr">Container</span> است.

<span dir="ltr">Read-only</span>

volumes:
  - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro

<span dir="ltr">Anonymous Volume</span>

ممکن است ببینی:

volumes:
  - /app/node_modules

برای شروع، <span dir="ltr">Named Volume</span> و <span dir="ltr">Bind Mount</span> را کامل بفهم و <span dir="ltr">Anonymous Volume</span> را فقط وقتی استفاده کن که دلیلش روشن باشد.

مدل تصمیم:

<table dir="rtl">
  <thead><tr><th>نیاز</th><th>انتخاب معمول</th></tr></thead>
  <tbody>
    <tr><td>دادهٔ پایگاه‌داده</td><td><code dir="ltr">Named Volume</code></td></tr>
    <tr><td><span dir="ltr">Source Code</span> در محیط توسعه</td><td><code dir="ltr">Bind Mount</code></td></tr>
    <tr><td>فایل تنظیمات از <span dir="ltr">Host</span></td><td><code dir="ltr">Bind Mount</code>، اغلب به‌صورت <code dir="ltr">:ro</code> اگر فقط خواندن لازم است.</td></tr>
  </tbody>
</table>

<a id="networks" name="networks"></a>

18. <code dir="ltr">networks</code>

services:
  backend:
    networks:
      - backend-net

  database:
    networks:
      - backend-net

networks:
  backend-net:

چند <span dir="ltr">Network:</span>

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

<pre dir="ltr"><code>Internet
   ↓
Nginx
   │ frontend-net
Backend
   │ backend-net
Database</code></pre>

این الگو کمک می‌کند <span dir="ltr">Service</span>هایی که لازم نیست مستقیم با هم ارتباط داشته باشند روی یک <span dir="ltr">Network</span> مشترک قرار نگیرند.

<a id="health" name="health"></a>

19. <code dir="ltr">depends_on</code> و <code dir="ltr">healthcheck</code>

<span dir="ltr">Dependency</span> ساده:

services:
  backend:
    depends_on:
      - database

اما <span dir="ltr">Start</span> شدن <span dir="ltr">Container</span> الزاماً یعنی <span dir="ltr">Service Ready</span> شده نیست.

<span dir="ltr">Healthcheck:</span>

services:
  database:
    image: postgres:17
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 5s
      timeout: 3s
      retries: 10

وابستگی به <span dir="ltr">Health:</span>

services:
  backend:
    depends_on:
      database:
        condition: service_healthy

<a id="runtime" name="runtime"></a>

20. <code dir="ltr">restart</code>، <code dir="ltr">command</code> و <code dir="ltr">entrypoint</code>

<span dir="ltr">Restart Policy:</span>

restart: unless-stopped

یا:

restart: on-failure

<span dir="ltr">Override CMD:</span>

command: ["python", "app.py"]

<span dir="ltr">Override ENTRYPOINT:</span>

entrypoint: ["/app/start.sh"]

قاعده:

<p dir="rtl">اگر <code dir="ltr">Dockerfile</code> رفتار اجرایی درست را تعریف کرده است، بدون نیاز مشخص <code dir="ltr">command</code> یا <code dir="ltr">entrypoint</code> را در <span dir="ltr">Compose</span> بازنویسی نکن.</p>

<a id="secrets" name="secrets"></a>

21. <span dir="ltr">Secret</span>ها را کجا نگذاریم؟

بد:

environment:
  DB_PASSWORD: my-production-password

اگر فایل <span dir="ltr">Commit</span> شود، <span dir="ltr">Secret</span> وارد <span dir="ltr">History</span> می‌شود.

بهتر:

environment:
  DB_PASSWORD: ${DB_PASSWORD}

و مقدار واقعی بیرون <span dir="ltr">Repository.</span>

برای <span dir="ltr">Production</span> جدی، <span dir="ltr">Secret Management</span> باید متناسب با <span dir="ltr">Platform</span> انتخاب شود. <code dir="ltr">.env</code> برای <span dir="ltr">Development</span> و محیط‌های محدود مفید است، اما جای همه راهکارهای <span dir="ltr">Secret Management</span> را نمی‌گیرد.

<a id="scenario1" name="scenario1"></a>

22. سناریو 1: <span dir="ltr">Nginx</span> تک‌سرویسی

services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"

<span dir="ltr">Validate:</span>

docker compose config

<span dir="ltr">Run:</span>

docker compose up -d

<span dir="ltr">Test:</span>

curl http://localhost:8080

این ساده‌ترین ساختار برای فهم رابطهٔ بین <code dir="ltr">services</code>، نام هر <span dir="ltr">service</span>، <code dir="ltr">image</code> و <code dir="ltr">ports</code> است.

<a id="scenario2" name="scenario2"></a>

23. سناریو 2: <span dir="ltr">Backend</span> + <span dir="ltr">Database</span>

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

ساختار:

<pre dir="ltr"><code>services
├── backend
└── database

volumes
└── db-data</code></pre>

در پروژه واقعی <span dir="ltr">Password</span> را از <code dir="ltr">.env</code> یا <span dir="ltr">Secret</span> مناسب بگیر.

<a id="scenario3" name="scenario3"></a>

24. سناریو 3: <span dir="ltr">Development</span> با <span dir="ltr">Bind Mount</span>

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

<pre dir="ltr"><code>Host ./backend
      ⇅
Container /app</code></pre>

اگر <span dir="ltr">Runtime Auto Reload</span> داشته باشد، <span dir="ltr">Development</span> سریع‌تر می‌شود.

اما <span dir="ltr">Mount</span> کردن <code dir="ltr">/app</code> می‌تواند فایل‌های قبلی همان مسیر در <span dir="ltr">Image</span> را پنهان کند؛ مسیر را آگاهانه انتخاب کن.

<a id="scenario4" name="scenario4"></a>

25. سناریو 4: <span dir="ltr">Network</span> اختصاصی

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

<pre dir="ltr"><code>nginx
  │
frontend-net
  │
backend
  │
backend-net
  │
database</code></pre>

از نظر <span dir="ltr">YAML</span> این فقط <span dir="ltr">Mapping</span> و <span dir="ltr">Sequence</span> است؛ معنی <span dir="ltr">Network</span> را <span dir="ltr">Compose</span> و <span dir="ltr">Docker</span> تعیین می‌کنند.

<a id="scenario5" name="scenario5"></a>

26. سناریو 5: <code dir="ltr">.env</code> برای <span dir="ltr">Environment</span>های مختلف

<code dir="ltr">.env</code>:

APP_PORT=8080
APP_ENV=development
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=dev-secret

<code dir="ltr">compose.yaml</code>:

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

نمایش نتیجه:

docker compose config

<a id="scenario6" name="scenario6"></a>

27. سناریو 6: <span dir="ltr">Healthcheck</span> واقعی

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

همان <code dir="ltr">test</code> را می‌توان <span dir="ltr">Flow Style</span> هم نوشت:

test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]

هر دو را باید بتوانی بخوانی.

<a id="override" name="override"></a>

28. <span dir="ltr">Compose</span> چندفایلی و <span dir="ltr">Override</span>

<span dir="ltr">Base:</span>

<code dir="ltr">compose.yaml</code>:

services:
  backend:
    build: ./backend
    environment:
      APP_ENV: production

<span dir="ltr">Development:</span>

<code dir="ltr">compose.dev.yaml</code>:

services:
  backend:
    environment:
      APP_ENV: development
    volumes:
      - ./backend:/app

اجرا:

docker compose \
  -f compose.yaml \
  -f compose.dev.yaml \
  up -d

نتیجه <span dir="ltr">Merge</span> شده:

docker compose \
  -f compose.yaml \
  -f compose.dev.yaml \
  config

برای پروژه کوچک، چند فایل بیش از حد می‌تواند پیچیدگی غیرضروری بسازد.

<a id="reuse" name="reuse"></a>

29. <span dir="ltr">Reuse</span> با <code dir="ltr">x-</code> و <span dir="ltr">Anchor</span>

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

هدف:

<pre dir="ltr"><code>Shared configuration
        │
        ▼
Multiple services</code></pre>

اما معیار اصلی خوانایی است. اگر <span dir="ltr">Anchor</span> باعث شود فهم فایل سخت‌تر شود، <span dir="ltr">Reuse</span> ارزشش را از دست می‌دهد.

<a id="error-types" name="error-types"></a>

30. سه نوع خطا: <span dir="ltr">YAML</span>، <span dir="ltr">Compose Schema</span> و <span dir="ltr">Runtime</span>

1. <span dir="ltr">YAML Syntax Error</span>

services:
   backend:
    image: my-backend

<span dir="ltr">Indentation</span> خراب است.

2. <span dir="ltr">Compose Schema Error</span>

<span dir="ltr">YAML</span> درست است:

services:
  backend:
    pizza: large

ولی <code dir="ltr">pizza</code> <span dir="ltr">Key</span> معتبر <span dir="ltr">Compose</span> نیست.

3. <span dir="ltr">Runtime Error</span>

<span dir="ltr">YAML</span> و <span dir="ltr">Schema</span> درست‌اند:

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"

اما <span dir="ltr">Host Port</span> <code dir="ltr">8080</code> قبلاً اشغال است.

مدل <span dir="ltr">Debug:</span>

<pre dir="ltr"><code>YAML parse
   ↓
Compose validation
   ↓
Docker runtime
   ↓
Application runtime</code></pre>

<a id="validation" name="validation"></a>

31. <span dir="ltr">Validation</span> و <span dir="ltr">Debug</span>

مهم‌ترین <span dir="ltr">Command:</span>

docker compose config

کاربرد:

<span dir="ltr">Parse</span> فایل

<span dir="ltr">Resolve Variable</span>ها

<span dir="ltr">Merge</span> فایل‌ها

نمایش <span dir="ltr">Configuration</span> نهایی

پیدا کردن بسیاری از خطاهای <span dir="ltr">Structure</span>

<span dir="ltr">Runtime:</span>

docker compose ps
docker compose logs -f
docker compose logs -f backend
docker compose exec backend sh

چند فایل:

docker compose \
  -f compose.yaml \
  -f compose.dev.yaml \
  config

<span dir="ltr">Checklist:</span>

<ul dir="rtl">
  <li>☐ <span dir="ltr">Indentation</span> درست است؟</li>
  <li>☐ <span dir="ltr">Parent</span> هر <span dir="ltr">Key</span> درست است؟</li>
  <li>☐ <span dir="ltr">List</span> و <span dir="ltr">Mapping</span> را با هم اشتباه نکرده‌ام؟</li>
  <li>☐ این <span dir="ltr">Key</span> در <span dir="ltr">Compose Schema</span> معتبر است؟</li>
  <li>☐ متغیرها درست <span dir="ltr">Resolve</span> شده‌اند؟</li>
  <li>☐ <span dir="ltr">Port</span> موردنظر آزاد است؟</li>
  <li>☐ نام <span dir="ltr">Service</span> برای <span dir="ltr">DNS</span> داخلی درست است؟</li>
  <li>☐ مسیر <span dir="ltr">Volume</span> یا <span dir="ltr">Bind Mount</span> درست است؟</li>
  <li>☐ <span dir="ltr">Healthcheck</span> واقعاً کار می‌کند؟</li>
  <li>☐ <span dir="ltr">Application</span> داخل <span dir="ltr">Container</span> در حال اجراست؟</li>
</ul>

<a id="style" name="style"></a>

32. <span dir="ltr">Style Guide</span> پیشنهادی

دو <span dir="ltr">Space</span> برای <span dir="ltr">Indentation</span>

services:
  backend:
    image: my-backend

<span dir="ltr">Tab</span> استفاده نکن

<span dir="ltr">Editor</span> را طوری تنظیم کن که <span dir="ltr">YAML</span> با <span dir="ltr">Space Indent</span> شود.

<span dir="ltr">Port Mapping</span> را <span dir="ltr">Quote</span> کن

ports:
  - "8080:80"

<span dir="ltr">Value</span> مبهم را <span dir="ltr">Quote</span> کن

version: "1.10"
code: "0123"

<span dir="ltr">Secret</span> واقعی <span dir="ltr">Commit</span> نکن

<pre dir="ltr"><code>.env → gitignored
.env.example → committed</code></pre>

<span dir="ltr">Service Name</span> واضح

خوب:

services:
  backend:
  database:
  nginx:

ضعیف:

services:
  a1:
  x:
  srv2:

<span dir="ltr">Comment</span> برای «چرا»

ضعیف:

ports:
  - "8080:80" # port

بهتر:

ports:
  - "8080:80" # local public entrypoint

<span dir="ltr">IP</span> ثابت <span dir="ltr">Hard-code</span> نکن

بد:

DB_HOST: "172.20.0.4"

خوب:

DB_HOST: database

<span dir="ltr">Configuration</span> را بیش از حد <span dir="ltr">Clever</span> نکن

<span dir="ltr">Anchor</span>، <span dir="ltr">Override</span> و چند <span dir="ltr">Network</span> ابزارند، نه هدف. اگر نسخه ساده‌تر همان کار را واضح‌تر انجام می‌دهد، نسخه ساده‌تر برای تیم بهتر است.

قبل از <span dir="ltr">Commit</span> یا <span dir="ltr">Deploy Validate</span> کن

docker compose config

<a id="practice" name="practice"></a>

33. تمرین مرحله‌ای

مرحله 1: <span dir="ltr">Nginx</span>

services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"

docker compose config
docker compose up -d

مرحله 2: <span dir="ltr">Named Volume</span>

services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - web-data:/usr/share/nginx/html

volumes:
  web-data:

مرحله 3: <span dir="ltr">Network</span> و <span dir="ltr">Service</span> دوم

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

مرحله 4: <code dir="ltr">.env</code>

NGINX_PORT=8080

<span dir="ltr">Compose:</span>

ports:
  - "${NGINX_PORT}:80"

نتیجه:

docker compose config

مرحله 5: خطای <span dir="ltr">Indentation</span> عمدی

یک خط را بد <span dir="ltr">Indent</span> کن و با <code dir="ltr">docker compose config</code> خطا را پیدا کن.

مرحله 6: <span dir="ltr">Schema Error</span> عمدی

pizza: large

تفاوت <span dir="ltr">YAML-valid</span> و <span dir="ltr">Compose-invalid</span> را ببین.

اگر این مراحل را بفهمی، <span dir="ltr">YAML</span> برای <span dir="ltr">Compose</span> دیگر مجموعه‌ای از خط‌های حفظی نیست؛ تبدیل به <span dir="ltr">Structure</span> قابل‌تحلیل می‌شود.

<a id="cheatsheet" name="cheatsheet"></a>

34. <span dir="ltr">Cheat Sheet</span>

ساختار <span dir="ltr">YAML</span>

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

<span dir="ltr">Compose</span> پایه

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

<span dir="ltr">Variable</span>

image: "myapp:${IMAGE_TAG}"

<code dir="ltr">.env</code>:

IMAGE_TAG=1.0.0

<span dir="ltr">Volume</span>

volumes:
  - db-data:/var/lib/postgresql/data

<span dir="ltr">Bind Mount</span>

volumes:
  - ./src:/app/src

<span dir="ltr">Network</span>

networks:
  - app-net

<span dir="ltr">Healthcheck</span>

healthcheck:
  test: ["CMD-SHELL", "curl -f http://localhost:4000/health || exit 1"]
  interval: 10s
  timeout: 3s
  retries: 5

<span dir="ltr">Validate</span>

docker compose config

اصل نهایی:

<ol dir="rtl">
  <li>اول ساختار <span dir="ltr">YAML</span> را بخوان.</li>
  <li>بعد <span dir="ltr">Schema</span> ابزار را بررسی کن.</li>
  <li>در آخر سراغ <span dir="ltr">Runtime Debugging</span> برو.</li>
</ol>

</div>