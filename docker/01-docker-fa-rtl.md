<div dir="rtl" align="right">

راهنمای عملی <span dir="ltr">Docker</span> برای شروع کار حرفه‌ای

این سند برای کسی نوشته شده که می‌خواهد <span dir="ltr">Docker</span> را از پایه بفهمد و بعد بتواند بدون حفظ‌کردن کورکورانه‌ی دستورها، سناریوهای واقعی را تحلیل و اجرا کند.

هدف این فایل آموزش <code dir="ltr">Dockerfile</code> یا <code dir="ltr">Docker Compose</code> نیست؛ آن دو در سند دوم بررسی می‌شوند. اینجا ابتدا باید خود <span dir="ltr">Docker</span>، <code dir="ltr">Image</code>، <code dir="ltr">Container</code>، <code dir="ltr">Port</code>، <code dir="ltr">Storage</code>، <code dir="ltr">Network</code> و چرخه‌ی اجرای <span dir="ltr">Container</span> را درست بفهمیم.

<a id="toc" name="toc"></a>

فهرست مطالب

<ol dir="rtl" align="right">
  <li><a href="#docker-problem"><span dir="ltr">Docker</span> چه مسئله‌ای را حل می‌کند؟</a></li>
  <li><a href="#container-vm"><span dir="ltr">Container</span> با <span dir="ltr">Virtual Machine</span> چه فرقی دارد؟</a></li>
  <li><a href="#architecture">معماری <span dir="ltr">Docker</span></a></li>
  <li><a href="#image-container"><span dir="ltr">Image</span> و <span dir="ltr">Container</span></a></li>
  <li><a href="#first-run">اولین اجرای <span dir="ltr">Docker</span></a></li>
  <li><a href="#docker-run">دستور <span dir="ltr">docker run</span> را درست بخوانیم</a></li>
  <li><a href="#lifecycle">چرخه‌ی زندگی <span dir="ltr">Container</span></a></li>
  <li><a href="#exec">ورود به <span dir="ltr">Container</span> و اجرای دستور با <span dir="ltr">exec</span></a></li>
  <li><a href="#observe">مشاهده وضعیت، <span dir="ltr">Log</span> و <span dir="ltr">Inspect</span></a></li>
  <li><a href="#ports"><span dir="ltr">Port</span> و <span dir="ltr">Port Mapping</span></a></li>
  <li><a href="#environment"><span dir="ltr">Environment Variable</span></a></li>
  <li><a href="#storage"><span dir="ltr">Storage: writable layer</span>، <span dir="ltr">docker cp</span>، <span dir="ltr">Bind Mount</span> و <span dir="ltr">Volume</span></a></li>
  <li><a href="#cp-vs-bind">سناریوی عملی <span dir="ltr">docker cp</span> در برابر <span dir="ltr">Bind Mount</span></a></li>
  <li><a href="#network"><span dir="ltr">Docker Network</span></a></li>
  <li><a href="#images">مدیریت <span dir="ltr">Image</span>، <span dir="ltr">Tag</span> و <span dir="ltr">Registry</span></a></li>
  <li><a href="#commit"><span dir="ltr">docker commit</span> چه زمانی مفید است؟</a></li>
  <li><a href="#archive"><span dir="ltr">save/load</span> در برابر <span dir="ltr">export/import</span></a></li>
  <li><a href="#cleanup">پاک‌سازی <span dir="ltr">Container</span> و <span dir="ltr">Image</span></a></li>
  <li><a href="#errors">خطاهای رایج و روش فکر کردن برای <span dir="ltr">Debug</span></a></li>
  <li><a href="#lab">تمرین نهایی</a></li>
  <li><a href="#cheatsheet"><span dir="ltr">Cheat Sheet</span></a></li>
  <li><a href="#next">بعد از این سند چه بخوانیم؟</a></li>
</ol>

<a id="docker-problem" name="docker-problem"></a>

1. <span dir="ltr">Docker</span> چه مسئله‌ای را حل می‌کند؟

یک برنامه معمولاً فقط «کد» نیست. ممکن است برای اجرا به نسخه‌ی مشخصی از <code dir="ltr">Python</code> یا <code dir="ltr">Node.js</code>، <span dir="ltr">Library</span>ها، <span dir="ltr">Environment Variable</span>ها، فایل <span dir="ltr">Configuration</span> و ابزارهای سیستم‌عاملی نیاز داشته باشد.

اگر این وابستگی‌ها را مستقیماً روی هر <span dir="ltr">Server</span> یا <span dir="ltr">Laptop</span> نصب کنیم، خیلی زود با تفاوت نسخه‌ها و تداخل <span dir="ltr">Package</span>ها روبه‌رو می‌شویم. <span dir="ltr">Docker</span> کمک می‌کند محیط اجرای برنامه را به شکل استاندارد بسته‌بندی و اجرا کنیم.

Application
+ Runtime
+ Dependencies
+ Configuration
        ↓
      Image
        ↓
   Container

<span dir="ltr">Docker</span> قرار نیست <span dir="ltr">Host OS</span> را حذف کند. <span dir="ltr">Container</span> روی <span dir="ltr">Host</span> اجرا می‌شود و از <span dir="ltr">Kernel</span> آن استفاده می‌کند، اما <span dir="ltr">Process</span>ها، <span dir="ltr">File System</span> و <span dir="ltr">Network</span> خودش را به شکل ایزوله‌تری می‌بیند.

<a id="container-vm" name="container-vm"></a>

2. <span dir="ltr">Container</span> با <span dir="ltr">Virtual Machine</span> چه فرقی دارد؟

<span dir="ltr">Virtual Machine</span> معمولاً یک <span dir="ltr">Guest OS</span> کامل دارد:

Hardware
└── Host OS
    └── Hypervisor
        ├── Guest OS A
        │   └── Application
        └── Guest OS B
            └── Application

<span dir="ltr">Container</span>ها معمولاً <span dir="ltr">Kernel Host</span> را به اشتراک می‌گذارند:

Hardware
└── Host OS
    └── Docker Engine
        ├── Container A
        │   └── Application
        └── Container B
            └── Application

<span dir="ltr">Container</span>ها معمولاً سریع‌تر <span dir="ltr">Start</span> می‌شوند و سبک‌ترند، اما این نتیجه را نگیریم که <code dir="ltr">Container = VM کوچک</code>. این دو ابزار دقیقاً یک مسئله را حل نمی‌کنند.

<a id="architecture" name="architecture"></a>

3. معماری <span dir="ltr">Docker</span>

وقتی می‌نویسیم:

docker run nginx

چند جزء درگیر هستند:

You
 │
 ▼
Docker CLI
 │ Docker API
 ▼
Docker Daemon / Engine
 │
 ├── Images
 ├── Containers
 ├── Networks
 └── Volumes

<code dir="ltr">Docker CLI</code> همان دستورهایی است که می‌نویسیم. <code dir="ltr">Docker Daemon</code> عملیات واقعی را انجام می‌دهد. <code dir="ltr">Registry</code> هم محل نگهداری <span dir="ltr">Image</span>هاست و <span dir="ltr">Docker Hub</span> یکی از <span dir="ltr">Registry</span>های شناخته‌شده است.

برای ادامه فرض می‌کنیم <span dir="ltr">Docker</span> نصب شده است. بررسی سریع:

docker --version
docker info

<a id="image-container" name="image-container"></a>

4. <span dir="ltr">Image</span> و <span dir="ltr">Container</span>

<span dir="ltr">Image</span>

<span dir="ltr">Image</span> یک الگوی آماده برای ساخت <span dir="ltr">Container</span> است:

ubuntu:24.04
nginx:alpine
python:3.13-slim
node:22

<span dir="ltr">Image</span> خودش <span dir="ltr">Process</span> در حال اجرا نیست.

<span dir="ltr">Container</span>

<span dir="ltr">Container</span> نمونه‌ای است که از روی <span dir="ltr">Image</span> ساخته و اجرا می‌شود:

nginx:alpine
     │
     ├── web-1
     ├── web-2
     └── web-3

مدل ذهنی:

Image
= الگو / بسته‌ی ساخت Container

Container
= Instance قابل اجرا از Image

<span dir="ltr">Layer</span>

<span dir="ltr">Image</span>ها از <span dir="ltr">Layer</span>ها ساخته می‌شوند. <span dir="ltr">Layer</span>های مشترک می‌توانند بین <span dir="ltr">Image</span>ها <span dir="ltr">reuse</span> شوند. برای مشاهده‌ی مصرف فضا:

docker system df
docker system df -v

<a id="first-run" name="first-run"></a>

5. اولین اجرای <span dir="ltr">Docker</span>

docker run hello-world

مدل آموزشی:

docker run hello-world
        │
        ├── Image محلی وجود دارد؟
        ├── اگر نه: Pull
        ├── Create Container
        └── Start Container

پس <code dir="ltr">docker run</code> را فقط «روشن کردن» در نظر نگیر؛ معمولاً <span dir="ltr">Container</span> جدید می‌سازد و آن را اجرا می‌کند.

<a id="docker-run" name="docker-run"></a>

6. دستور <code dir="ltr">docker run</code> را درست بخوانیم

فرم کلی:

docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

اجرای ساده:

docker run nginx

<span dir="ltr">Background:</span>

docker run -d nginx

نام‌گذاری:

docker run -d --name web nginx

<span dir="ltr">Interactive shell:</span>

docker run -it ubuntu bash

<code dir="ltr">-i</code> ورودی تعاملی را باز نگه می‌دارد.

<code dir="ltr">-t</code> <span dir="ltr">Terminal</span> مجازی ایجاد می‌کند.

<code dir="ltr">bash</code> <span dir="ltr">Command</span> داخل <span dir="ltr">Container</span> است.

<span dir="ltr">Container</span> موقت:

docker run --rm ubuntu echo "hello"

اگر <span dir="ltr">Image</span> یک <span dir="ltr">Command</span> پیش‌فرض داشته باشد، <span dir="ltr">Command</span> انتهای <code dir="ltr">docker run</code> می‌تواند آن را <span dir="ltr">Override</span> کند. مثلاً:

docker run -it myapp bash

ممکن است به‌جای <span dir="ltr">Application</span>، <span dir="ltr">Bash</span> را اجرا کند.

<a id="lifecycle" name="lifecycle"></a>

7. چرخه‌ی زندگی <span dir="ltr">Container</span>

Created
   │
   ▼
Running
   │
   ├── stop
   ▼
Stopped / Exited
   │
   ├── start ─────→ Running
   └── rm ────────→ Deleted

دستورهای اصلی:

docker ps
docker ps -a
docker stop web
docker start web
docker restart web
docker rm web
docker rm -f web

تفاوت مهم:

docker run
→ معمولاً Container جدید می‌سازد و اجرا می‌کند

docker start
→ همان Container موجود را دوباره اجرا می‌کند

<code dir="ltr">-f</code> یعنی <span dir="ltr">Force.</span> در <span dir="ltr">Lab</span> مفید است، اما در محیط واقعی بدون دلیل از آن استفاده نکن.

<a id="exec" name="exec"></a>

8. ورود به <span dir="ltr">Container</span> و اجرای دستور با <code dir="ltr">exec</code>

فرض کن <code dir="ltr">web</code> <span dir="ltr">Running</span> است.

اجرای یک <span dir="ltr">Command:</span>

docker exec web ls -lah /

ورود به <span dir="ltr">Bash:</span>

docker exec -it web bash

اگر <span dir="ltr">Bash</span> موجود نبود:

docker exec -it web sh

تفاوت:

docker start web
→ خود Container را Running می‌کند

docker exec -it web bash
→ داخل Container Running یک Process جدید باز می‌کند

روی <span dir="ltr">Container</span> متوقف‌شده ابتدا باید <code dir="ltr">start</code> انجام شود.

<a id="observe" name="observe"></a>

9. مشاهده وضعیت، <span dir="ltr">Log</span> و <span dir="ltr">Inspect</span>

قبل از حذف و ساخت مجدد همه‌چیز، اطلاعات جمع کن.

docker logs web
docker logs -f web
docker top web
docker stats
docker inspect web
docker diff web

<code dir="ltr">docker inspect</code> برای دیدن مواردی مثل <span dir="ltr">IP</span>، <span dir="ltr">Network</span>، <span dir="ltr">Mount</span>، <span dir="ltr">Environment Variable</span>، <span dir="ltr">Port Binding</span>، <span dir="ltr">State</span> و <span dir="ltr">Image</span> بسیار مهم است.

<a id="ports" name="ports"></a>

10. <span dir="ltr">Port</span> و <span dir="ltr">Port Mapping</span>

فرض کن <span dir="ltr">Application</span> داخل <span dir="ltr">Container</span> روی <span dir="ltr">Port</span> <code dir="ltr">3000</code> گوش می‌دهد.

فرم <span dir="ltr">Publish:</span>

-p HOST_PORT:CONTAINER_PORT

مثال:

docker run -p 2000:3000 myapp

Browser / curl
      ↓
Host :2000
      ↓
Docker
      ↓
Container :3000
      ↓
Application

<span dir="ltr">Host Port</span> لازم نیست با <span dir="ltr">Container Port</span> یکی باشد:

docker run -p 5000:3000 myapp

<code dir="ltr">0.0.0.0</code> داخل <span dir="ltr">Application</span>

اگر <span dir="ltr">Application</span> فقط روی <code dir="ltr">127.0.0.1</code> داخل <span dir="ltr">Container</span> گوش دهد، معمولاً از بیرون <span dir="ltr">Container</span> قابل دسترسی نیست. برای مثال <span dir="ltr">Flask:</span>

app.run(host="0.0.0.0", port=4000)

<code dir="ltr">port is already allocated</code>

اگر <span dir="ltr">Host Port</span> اشغال باشد:

docker ps
ss -tulpn

یا <span dir="ltr">Process</span> قبلی را <span dir="ltr">Stop</span> کن یا <span dir="ltr">Host Port</span> دیگری انتخاب کن.

<code dir="ltr">EXPOSE</code> با <code dir="ltr">-p</code> یکی نیست

<code dir="ltr">EXPOSE</code> بیشتر <span dir="ltr">Port</span> مورد انتظار <span dir="ltr">Image</span> را بیان می‌کند. <span dir="ltr">Publish</span> واقعی با <code dir="ltr">-p</code> یا تنظیم معادل در <span dir="ltr">Compose</span> انجام می‌شود.

<a id="environment" name="environment"></a>

11. <span dir="ltr">Environment Variable</span>

docker run \
  -e APP_ENV=development \
  -e PORT=3000 \
  myapp

فایل <span dir="ltr">env:</span>

docker run --env-file .env myapp

نمونه <code dir="ltr">.env</code>:

APP_ENV=development
PORT=3000
DB_HOST=database

برای <span dir="ltr">Repository</span> عمومی، <span dir="ltr">Secret</span> واقعی را <span dir="ltr">Commit</span> نکن. <code dir="ltr">.env.example</code> برای نمایش نام <span dir="ltr">Variable</span>ها مناسب است.

<a id="storage" name="storage"></a>

12. <span dir="ltr">Storage: writable layer</span>، <code dir="ltr">docker cp</code>، <span dir="ltr">Bind Mount</span> و <span dir="ltr">Volume</span>

<span dir="ltr">Writable Layer</span>

فایلی که داخل <span dir="ltr">Container</span> می‌سازی در <span dir="ltr">File System</span> همان <span dir="ltr">Container</span> قرار می‌گیرد. <span dir="ltr">Stop/Start</span> معمولاً آن را حفظ می‌کند، اما با حذف <span dir="ltr">Container</span> روی آن برای <span dir="ltr">Data</span> مهم حساب نکن.

<code dir="ltr">docker cp</code>

docker cp file.txt web:/tmp/file.txt
docker cp web:/tmp/file.txt ./file.txt

قاعده:

docker cp
= Copy
≠ Sync

<span dir="ltr">Bind Mount</span>

docker run -v "$PWD":/app myapp

Host directory
     ⇅
Container directory

برای <span dir="ltr">Development</span> مفید است؛ تغییر <span dir="ltr">Host</span> داخل <span dir="ltr">Container</span> دیده می‌شود. اما حذف یا تغییر فایل <span dir="ltr">Mount</span>‌شده داخل <span dir="ltr">Container</span> می‌تواند روی <span dir="ltr">Host</span> هم اثر بگذارد.

<span dir="ltr">Named Volume</span>

docker volume create db-data

docker run \
  -v db-data:/var/lib/postgresql/data \
  postgres

بررسی:

docker volume ls
docker volume inspect db-data

<span dir="ltr">tmpfs</span>

برای <span dir="ltr">Data</span> موقتی در <span dir="ltr">Memory:</span>

docker run --tmpfs /cache nginx

مدل نهایی:

Writable layer → وابسته به عمر Container
Bind Mount     → مسیر واقعی Host
Named Volume   → Storage مدیریت‌شده Docker
tmpfs          → Memory و موقت

<a id="cp-vs-bind" name="cp-vs-bind"></a>

13. سناریوی عملی <code dir="ltr">docker cp</code> در برابر <span dir="ltr">Bind Mount</span>

فرض کنیم روی <span dir="ltr">Host</span> فایل <code dir="ltr">wilson.c</code> داریم.

<span dir="ltr">Container</span> اول:

docker run -it -d --name copied ubuntu bash
docker cp wilson.c copied:/home/wilson.c

از این لحظه دو نسخه مستقل داریم.

<span dir="ltr">Container</span> دوم:

docker run -it -d \
  --name mounted \
  -v "$PWD":/code \
  gcc bash

داخل <span dir="ltr">Container:</span>

docker exec -it mounted bash
cd /code
gcc -o main wilson.c
./main

چون <code dir="ltr">/code</code> <span dir="ltr">Bind Mount</span> است، فایل <code dir="ltr">main</code> روی <span dir="ltr">Host</span> هم دیده می‌شود.

حالا فایل <span dir="ltr">Host</span> را تغییر بده. <span dir="ltr">Container</span> اول نسخه قبلی را می‌بیند:

docker exec copied cat /home/wilson.c

<span dir="ltr">Container</span> دوم نسخه جدید را می‌بیند:

docker exec mounted cat /code/wilson.c

ویژگی

<code dir="ltr">docker cp</code>

<span dir="ltr">Bind Mount</span>

انتقال فایل

بله

بله

ارتباط زنده

خیر

بله

تغییر <span dir="ltr">Host</span> دیده می‌شود

خیر

بله

مناسب <span dir="ltr">Development</span>

محدود

بله

<a id="network" name="network"></a>

14. <span dir="ltr">Docker Network</span>

مشاهده:

docker network ls

<span dir="ltr">Network</span> اختصاصی:

docker network create app-net

دو <span dir="ltr">Container</span> روی یک <span dir="ltr">Network:</span>

docker run -d --name database --network app-net postgres

docker run -d --name backend --network app-net my-backend

در <span dir="ltr">User-defined Network</span>، بهتر است از نام <span dir="ltr">Container/Service</span> استفاده کنیم:

database:5432

نه <span dir="ltr">IP</span> موقت:

172.18.0.5:5432

بررسی:

docker network inspect app-net

اتصال و جداسازی:

docker network connect app-net web
docker network disconnect app-net web

<span dir="ltr">Driver</span>هایی که فعلاً باید اسمشان را بشناسی:

bridge   → رایج روی یک Host
host     → استفاده نزدیک‌تر از Network Host
none     → بدون Network معمول Docker
overlay  → سناریوهای چند Host
macvlan  → سناریوهای خاص شبکه

برای <span dir="ltr">Junior</span> مهم‌ترین بخش فعلاً <code dir="ltr">bridge</code> و <span dir="ltr">User-defined Network</span> است.

<a id="images" name="images"></a>

15. مدیریت <span dir="ltr">Image</span>، <span dir="ltr">Tag</span> و <span dir="ltr">Registry</span>

docker pull nginx:alpine
docker image ls
docker rmi nginx:alpine

<span dir="ltr">Tag:</span>

docker tag myapp:1.0.0 myapp:stable

<span dir="ltr">Tag</span> الزاماً <span dir="ltr">Image</span> را کپی نمی‌کند؛ یک <span dir="ltr">Reference</span> جدید می‌سازد.

الگوی رایج نام <span dir="ltr">Image:</span>

REGISTRY/OWNER/IMAGE:TAG

<a id="commit" name="commit"></a>

16. <code dir="ltr">docker commit</code> چه زمانی مفید است؟

docker run -it --name lab ubuntu bash

داخل <span dir="ltr">Container:</span>

apt update
apt install -y curl
touch /home/example.txt
exit

ساخت <span dir="ltr">Image</span> از وضعیت فعلی:

docker commit lab mylab:1.0.0

ubuntu
  ↓
lab container
  ↓ manual changes
docker commit
  ↓
mylab:1.0.0

برای <span dir="ltr">Lab</span> و <span dir="ltr">Snapshot</span> سریع مفید است، اما برای پروژه واقعی روش اصلی نیست؛ چون مراحل ساخت داخل <span dir="ltr">Code</span> ثبت نشده‌اند. در پروژه واقعی <code dir="ltr">Dockerfile</code> روش قابل‌تکرارتر است.

<a id="archive" name="archive"></a>

17. <code dir="ltr">save/load</code> در برابر <code dir="ltr">export/import</code>

دو خانواده جدا داریم.

<span dir="ltr">Image</span>

Image
  ↓ docker save
tar
  ↓ docker load
Image

docker save myapp:1.0.0 -o myapp.tar
docker load -i myapp.tar

<span dir="ltr">Container File System</span>

Container
  ↓ docker export
tar
  ↓ docker import
New Image

docker export mycontainer -o container.tar
docker import container.tar imported-app:1.0.0

موضوع

<code dir="ltr">save/load</code>

<code dir="ltr">export/import</code>

مبدأ

<span dir="ltr">Image</span>

<span dir="ltr">Container</span>

هدف

انتقال/<span dir="ltr">Restore Image</span>

<span dir="ltr">File System Container</span>

<span dir="ltr">Layer</span>های <span dir="ltr">Image</span>

حفظ می‌شوند

ساختار اصلی <span dir="ltr">Layer</span>ها حفظ نمی‌شود

<span dir="ltr">Runtime metadata</span>

مناسب‌تر برای <span dir="ltr">Restore</span>

ممکن است از دست برود

<span dir="ltr">Image</span> ساخته‌شده با <code dir="ltr">docker import</code> ممکن است <span dir="ltr">Command</span> پیش‌فرض مناسب نداشته باشد. در آن حالت باید <span dir="ltr">Command</span> را صریح بدهی:

docker run -it imported-app:1.0.0 bash

<a id="cleanup" name="cleanup"></a>

18. پاک‌سازی <span dir="ltr">Container</span> و <span dir="ltr">Image</span>

<span dir="ltr">Container:</span>

docker rm container-name
docker rm $(docker ps -a -q)
docker rm -f $(docker ps -a -q)

<span dir="ltr">Image:</span>

docker rmi image-name
docker rmi $(docker images -q)

این دستور معمولاً اشتباه مفهومی است:

docker rmi $(docker ps -a -q)

چون <code dir="ltr">docker ps</code> شناسه <span dir="ltr">Container</span> می‌دهد ولی <code dir="ltr">docker rmi</code> برای <span dir="ltr">Image</span> است.

<span dir="ltr">Prune:</span>

docker container prune
docker image prune
docker volume prune
docker network prune
docker system prune

قبل از <code dir="ltr">volume prune</code> مطمئن شو <span dir="ltr">Data</span> مهمی حذف نمی‌شود.

<a id="errors" name="errors"></a>

19. خطاهای رایج و روش فکر کردن برای <span dir="ltr">Debug</span>

<code dir="ltr">command not found</code>

<span dir="ltr">Image</span> <code dir="ltr">ubuntu</code> الزاماً <span dir="ltr">Python</span>، <span dir="ltr">Node</span> یا <span dir="ltr">GCC</span> ندارد. <span dir="ltr">Image</span> مناسب را انتخاب کن یا <span dir="ltr">Package</span> لازم را نصب کن.

<code dir="ltr">docker</code> داخل <span dir="ltr">Container</span> وجود ندارد

<span dir="ltr">Docker Engine</span> روی <span dir="ltr">Host</span> اجرا می‌شود؛ <span dir="ltr">Container</span> معمولی الزاماً <span dir="ltr">Docker CLI</span> ندارد.

<code dir="ltr">exec</code> روی <span dir="ltr">Container</span> متوقف‌شده

docker start web
docker exec -it web bash

<span dir="ltr">Port</span> درست <span dir="ltr">Publish</span> شده ولی برنامه جواب نمی‌دهد

این سه سؤال را بررسی کن:

Application Running است؟
داخل Container روی چه Portی Listen می‌کند؟
روی 0.0.0.0 گوش می‌دهد یا فقط 127.0.0.1؟

ابزارها:

docker logs web
docker inspect web

تغییر <span dir="ltr">Host</span> داخل <span dir="ltr">Container</span> دیده نمی‌شود

بپرس فایل با <code dir="ltr">docker cp</code>/<code dir="ltr">COPY</code> آمده یا <span dir="ltr">Bind Mount</span> است.

مدل <span dir="ltr">Debug</span> پنج مرحله‌ای

1. State   → docker ps -a
2. Logs    → docker logs
3. Config  → docker inspect
4. Process / Port
5. Storage / Network

قبل از <code dir="ltr">rm -f</code> همه‌چیز، این مراحل اطلاعات بسیار بیشتری می‌دهند.

<a id="lab" name="lab"></a>

20. تمرین نهایی

<span dir="ltr">Nginx:</span>

docker run -d \
  --name web-lab \
  -p 8080:80 \
  nginx:alpine

تست:

docker ps
curl http://localhost:8080

<span dir="ltr">Log</span> و <span dir="ltr">Inspect:</span>

docker logs web-lab
docker inspect web-lab

ورود:

docker exec -it web-lab sh

حالا <span dir="ltr">Bind Mount:</span>

docker rm -f web-lab
mkdir -p nginx-lab
echo '<h1>Hello from host</h1>' > nginx-lab/index.html

docker run -d \
  --name web-lab \
  -p 8080:80 \
  -v "$PWD/nginx-lab":/usr/share/nginx/html:ro \
  nginx:alpine

تست:

curl http://localhost:8080

<span dir="ltr">Network:</span>

docker network create lab-net
docker run -d --name helper --network lab-net alpine sleep 3600
docker network connect lab-net web-lab
docker network inspect lab-net

اگر بتوانی بعد از این تمرین <code dir="ltr">Image</code>، <code dir="ltr">Container</code>، <code dir="ltr">run</code>، <code dir="ltr">start</code>، <code dir="ltr">exec</code>، <span dir="ltr">Port Mapping</span>، <span dir="ltr">Bind Mount</span>، <span dir="ltr">Network</span>، <code dir="ltr">inspect</code> و <code dir="ltr">logs</code> را توضیح دهی، آماده‌ی سند دوم هستی.

<a id="cheatsheet" name="cheatsheet"></a>

21. <span dir="ltr">Cheat Sheet</span>

<span dir="ltr">Container</span>

docker ps
docker ps -a
docker run IMAGE
docker run -d --name NAME IMAGE
docker start NAME
docker stop NAME
docker restart NAME
docker rm NAME
docker rm -f NAME
docker exec -it NAME bash
docker logs -f NAME
docker inspect NAME
docker stats

<span dir="ltr">Image</span>

docker pull IMAGE
docker image ls
docker rmi IMAGE
docker tag SOURCE TARGET
docker save IMAGE -o image.tar
docker load -i image.tar

<span dir="ltr">Storage</span>

docker cp file.txt container:/tmp/file.txt
docker volume create data
docker volume ls
docker volume inspect data
docker run -v data:/data IMAGE
docker run -v "$PWD":/app IMAGE

<span dir="ltr">Network</span>

docker network ls
docker network create app-net
docker network inspect app-net
docker network connect app-net CONTAINER
docker network disconnect app-net CONTAINER

<span dir="ltr">Port</span> و <span dir="ltr">Environment</span>

docker run -p 8080:80 IMAGE
docker run -e APP_ENV=dev IMAGE
docker run --env-file .env IMAGE

<a id="next" name="next"></a>

22. بعد از این سند چه بخوانیم؟

Docker پایه
   ↓
Dockerfile
   ↓
Build Image
   ↓
Docker Compose
   ↓
Multi-container Application
   ↓
CI/CD و Deployment

سند دوم از همین نقطه شروع می‌شود: ساخت <span dir="ltr">Image</span> قابل‌تکرار با <code dir="ltr">Dockerfile</code> و مدیریت چند <span dir="ltr">Service</span> با <code dir="ltr">Docker Compose</code>.

</div>