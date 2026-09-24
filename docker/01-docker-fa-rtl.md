<div dir="rtl" align="right">

# راهنمای عملی Docker برای شروع کار حرفه‌ای

این سند برای کسی نوشته شده که می‌خواهد Docker را از پایه بفهمد و بعد بتواند بدون حفظ‌کردن کورکورانه‌ی دستورها، سناریوهای واقعی را تحلیل و اجرا کند.

هدف این فایل آموزش `Dockerfile` یا `Docker Compose` نیست؛ آن دو در سند دوم بررسی می‌شوند. اینجا ابتدا باید خود Docker، `Image`، `Container`، `Port`، `Storage`، `Network` و چرخه‌ی اجرای Container را درست بفهمیم.

---

<a id="toc"></a>
<details open>
<summary>

## فهرست مطالب

</summary>

1. [Docker چه مسئله‌ای را حل می‌کند؟](#docker-problem)
2. [Container با Virtual Machine چه فرقی دارد؟](#container-vm)
3. [معماری Docker](#architecture)
4. [Image و Container](#image-container)
5. [اولین اجرای Docker](#first-run)
6. [دستور docker run را درست بخوانیم](#docker-run)
7. [چرخه‌ی زندگی Container](#lifecycle)
8. [ورود به Container و اجرای دستور با exec](#exec)
9. [مشاهده وضعیت، Log و Inspect](#observe)
10. [Port و Port Mapping](#ports)
11. [Environment Variable](#environment)
12. [Storage: writable layer، docker cp، Bind Mount و Volume](#storage)
13. [سناریوی عملی docker cp در برابر Bind Mount](#cp-vs-bind)
14. [Docker Network](#network)
15. [مدیریت Image، Tag و Registry](#images)
16. [docker commit چه زمانی مفید است؟](#commit)
17. [save/load در برابر export/import](#archive)
18. [پاک‌سازی Container و Image](#cleanup)
19. [خطاهای رایج و روش فکر کردن برای Debug](#errors)
20. [تمرین نهایی](#lab)
21. [Cheat Sheet](#cheatsheet)
22. [بعد از این سند چه بخوانیم؟](#next)

</details>

---

<a id="docker-problem"></a>
## 1. Docker چه مسئله‌ای را حل می‌کند؟

یک برنامه معمولاً فقط «کد» نیست. ممکن است برای اجرا به نسخه‌ی مشخصی از `Python` یا `Node.js`، Libraryها، Environment Variableها، فایل Configuration و ابزارهای سیستم‌عاملی نیاز داشته باشد.

اگر این وابستگی‌ها را مستقیماً روی هر Server یا Laptop نصب کنیم، خیلی زود با تفاوت نسخه‌ها و تداخل Packageها روبه‌رو می‌شویم. Docker کمک می‌کند **محیط اجرای برنامه را به شکل استاندارد بسته‌بندی و اجرا کنیم**.

```text
Application
+ Runtime
+ Dependencies
+ Configuration
        ↓
      Image
        ↓
   Container
```

Docker قرار نیست Host OS را حذف کند. Container روی Host اجرا می‌شود و از Kernel آن استفاده می‌کند، اما Processها، File System و Network خودش را به شکل ایزوله‌تری می‌بیند.

---

<a id="container-vm"></a>
## 2. Container با Virtual Machine چه فرقی دارد؟

Virtual Machine معمولاً یک Guest OS کامل دارد:

```text
Hardware
└── Host OS
    └── Hypervisor
        ├── Guest OS A
        │   └── Application
        └── Guest OS B
            └── Application
```

Containerها معمولاً Kernel Host را به اشتراک می‌گذارند:

```text
Hardware
└── Host OS
    └── Docker Engine
        ├── Container A
        │   └── Application
        └── Container B
            └── Application
```

Containerها معمولاً سریع‌تر Start می‌شوند و سبک‌ترند، اما این نتیجه را نگیریم که `Container = VM کوچک`. این دو ابزار دقیقاً یک مسئله را حل نمی‌کنند.

---

<a id="architecture"></a>
## 3. معماری Docker

وقتی می‌نویسیم:

```bash
docker run nginx
```

چند جزء درگیر هستند:

```text
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
```

`Docker CLI` همان دستورهایی است که می‌نویسیم. `Docker Daemon` عملیات واقعی را انجام می‌دهد. `Registry` هم محل نگهداری Imageهاست و Docker Hub یکی از Registryهای شناخته‌شده است.

برای ادامه فرض می‌کنیم Docker نصب شده است. بررسی سریع:

```bash
docker --version
docker info
```

---

<a id="image-container"></a>
## 4. Image و Container

### Image

Image یک الگوی آماده برای ساخت Container است:

```text
ubuntu:24.04
nginx:alpine
python:3.13-slim
node:22
```

Image خودش Process در حال اجرا نیست.

### Container

Container نمونه‌ای است که از روی Image ساخته و اجرا می‌شود:

```text
nginx:alpine
     │
     ├── web-1
     ├── web-2
     └── web-3
```

مدل ذهنی:

```text
Image
= الگو / بسته‌ی ساخت Container

Container
= Instance قابل اجرا از Image
```

### Layer

Imageها از Layerها ساخته می‌شوند. Layerهای مشترک می‌توانند بین Imageها reuse شوند. برای مشاهده‌ی مصرف فضا:

```bash
docker system df
docker system df -v
```

---

<a id="first-run"></a>
## 5. اولین اجرای Docker

```bash
docker run hello-world
```

مدل آموزشی:

```text
docker run hello-world
        │
        ├── Image محلی وجود دارد؟
        ├── اگر نه: Pull
        ├── Create Container
        └── Start Container
```

پس `docker run` را فقط «روشن کردن» در نظر نگیر؛ معمولاً Container جدید می‌سازد و آن را اجرا می‌کند.

---

<a id="docker-run"></a>
## 6. دستور `docker run` را درست بخوانیم

فرم کلی:

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

اجرای ساده:

```bash
docker run nginx
```

Background:

```bash
docker run -d nginx
```

نام‌گذاری:

```bash
docker run -d --name web nginx
```

Interactive shell:

```bash
docker run -it ubuntu bash
```

- `-i` ورودی تعاملی را باز نگه می‌دارد.
- `-t` Terminal مجازی ایجاد می‌کند.
- `bash` Command داخل Container است.

Container موقت:

```bash
docker run --rm ubuntu echo "hello"
```

اگر Image یک Command پیش‌فرض داشته باشد، Command انتهای `docker run` می‌تواند آن را Override کند. مثلاً:

```bash
docker run -it myapp bash
```

ممکن است به‌جای Application، Bash را اجرا کند.

---

<a id="lifecycle"></a>
## 7. چرخه‌ی زندگی Container

```text
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
```

دستورهای اصلی:

```bash
docker ps
docker ps -a
docker stop web
docker start web
docker restart web
docker rm web
docker rm -f web
```

تفاوت مهم:

```text
docker run
→ معمولاً Container جدید می‌سازد و اجرا می‌کند

docker start
→ همان Container موجود را دوباره اجرا می‌کند
```

`-f` یعنی Force. در Lab مفید است، اما در محیط واقعی بدون دلیل از آن استفاده نکن.

---

<a id="exec"></a>
## 8. ورود به Container و اجرای دستور با `exec`

فرض کن `web` Running است.

اجرای یک Command:

```bash
docker exec web ls -lah /
```

ورود به Bash:

```bash
docker exec -it web bash
```

اگر Bash موجود نبود:

```bash
docker exec -it web sh
```

تفاوت:

```text
docker start web
→ خود Container را Running می‌کند

docker exec -it web bash
→ داخل Container Running یک Process جدید باز می‌کند
```

روی Container متوقف‌شده ابتدا باید `start` انجام شود.

---

<a id="observe"></a>
## 9. مشاهده وضعیت، Log و Inspect

قبل از حذف و ساخت مجدد همه‌چیز، اطلاعات جمع کن.

```bash
docker logs web
docker logs -f web
docker top web
docker stats
docker inspect web
docker diff web
```

`docker inspect` برای دیدن مواردی مثل IP، Network، Mount، Environment Variable، Port Binding، State و Image بسیار مهم است.

---

<a id="ports"></a>
## 10. Port و Port Mapping

فرض کن Application داخل Container روی Port `3000` گوش می‌دهد.

فرم Publish:

```bash
-p HOST_PORT:CONTAINER_PORT
```

مثال:

```bash
docker run -p 2000:3000 myapp
```

```text
Browser / curl
      ↓
Host :2000
      ↓
Docker
      ↓
Container :3000
      ↓
Application
```

Host Port لازم نیست با Container Port یکی باشد:

```bash
docker run -p 5000:3000 myapp
```

### `0.0.0.0` داخل Application

اگر Application فقط روی `127.0.0.1` داخل Container گوش دهد، معمولاً از بیرون Container قابل دسترسی نیست. برای مثال Flask:

```python
app.run(host="0.0.0.0", port=4000)
```

### `port is already allocated`

اگر Host Port اشغال باشد:

```bash
docker ps
ss -tulpn
```

یا Process قبلی را Stop کن یا Host Port دیگری انتخاب کن.

### `EXPOSE` با `-p` یکی نیست

`EXPOSE` بیشتر Port مورد انتظار Image را بیان می‌کند. Publish واقعی با `-p` یا تنظیم معادل در Compose انجام می‌شود.

---

<a id="environment"></a>
## 11. Environment Variable

```bash
docker run \
  -e APP_ENV=development \
  -e PORT=3000 \
  myapp
```

فایل env:

```bash
docker run --env-file .env myapp
```

نمونه `.env`:

```ini
APP_ENV=development
PORT=3000
DB_HOST=database
```

برای Repository عمومی، Secret واقعی را Commit نکن. `.env.example` برای نمایش نام Variableها مناسب است.

---

<a id="storage"></a>
## 12. Storage: writable layer، `docker cp`، Bind Mount و Volume

### Writable Layer

فایلی که داخل Container می‌سازی در File System همان Container قرار می‌گیرد. Stop/Start معمولاً آن را حفظ می‌کند، اما با حذف Container روی آن برای Data مهم حساب نکن.

### `docker cp`

```bash
docker cp file.txt web:/tmp/file.txt
docker cp web:/tmp/file.txt ./file.txt
```

قاعده:

```text
docker cp
= Copy
≠ Sync
```

### Bind Mount

```bash
docker run -v "$PWD":/app myapp
```

```text
Host directory
     ⇅
Container directory
```

برای Development مفید است؛ تغییر Host داخل Container دیده می‌شود. اما حذف یا تغییر فایل Mount‌شده داخل Container می‌تواند روی Host هم اثر بگذارد.

### Named Volume

```bash
docker volume create db-data

docker run \
  -v db-data:/var/lib/postgresql/data \
  postgres
```

بررسی:

```bash
docker volume ls
docker volume inspect db-data
```

### tmpfs

برای Data موقتی در Memory:

```bash
docker run --tmpfs /cache nginx
```

مدل نهایی:

```text
Writable layer → وابسته به عمر Container
Bind Mount     → مسیر واقعی Host
Named Volume   → Storage مدیریت‌شده Docker
tmpfs          → Memory و موقت
```

---

<a id="cp-vs-bind"></a>
## 13. سناریوی عملی `docker cp` در برابر Bind Mount

فرض کنیم روی Host فایل `wilson.c` داریم.

Container اول:

```bash
docker run -it -d --name copied ubuntu bash
docker cp wilson.c copied:/home/wilson.c
```

از این لحظه دو نسخه مستقل داریم.

Container دوم:

```bash
docker run -it -d \
  --name mounted \
  -v "$PWD":/code \
  gcc bash
```

داخل Container:

```bash
docker exec -it mounted bash
cd /code
gcc -o main wilson.c
./main
```

چون `/code` Bind Mount است، فایل `main` روی Host هم دیده می‌شود.

حالا فایل Host را تغییر بده. Container اول نسخه قبلی را می‌بیند:

```bash
docker exec copied cat /home/wilson.c
```

Container دوم نسخه جدید را می‌بیند:

```bash
docker exec mounted cat /code/wilson.c
```

| ویژگی | `docker cp` | Bind Mount |
|---|---|---|
| انتقال فایل | بله | بله |
| ارتباط زنده | خیر | بله |
| تغییر Host دیده می‌شود | خیر | بله |
| مناسب Development | محدود | بله |

---

<a id="network"></a>
## 14. Docker Network

مشاهده:

```bash
docker network ls
```

Network اختصاصی:

```bash
docker network create app-net
```

دو Container روی یک Network:

```bash
docker run -d --name database --network app-net postgres

docker run -d --name backend --network app-net my-backend
```

در User-defined Network، بهتر است از نام Container/Service استفاده کنیم:

```text
database:5432
```

نه IP موقت:

```text
172.18.0.5:5432
```

بررسی:

```bash
docker network inspect app-net
```

اتصال و جداسازی:

```bash
docker network connect app-net web
docker network disconnect app-net web
```

Driverهایی که فعلاً باید اسمشان را بشناسی:

```text
bridge   → رایج روی یک Host
host     → استفاده نزدیک‌تر از Network Host
none     → بدون Network معمول Docker
overlay  → سناریوهای چند Host
macvlan  → سناریوهای خاص شبکه
```

برای Junior مهم‌ترین بخش فعلاً `bridge` و User-defined Network است.

---

<a id="images"></a>
## 15. مدیریت Image، Tag و Registry

```bash
docker pull nginx:alpine
docker image ls
docker rmi nginx:alpine
```

Tag:

```bash
docker tag myapp:1.0.0 myapp:stable
```

Tag الزاماً Image را کپی نمی‌کند؛ یک Reference جدید می‌سازد.

الگوی رایج نام Image:

```text
REGISTRY/OWNER/IMAGE:TAG
```

---

<a id="commit"></a>
## 16. `docker commit` چه زمانی مفید است؟

```bash
docker run -it --name lab ubuntu bash
```

داخل Container:

```bash
apt update
apt install -y curl
touch /home/example.txt
exit
```

ساخت Image از وضعیت فعلی:

```bash
docker commit lab mylab:1.0.0
```

```text
ubuntu
  ↓
lab container
  ↓ manual changes
docker commit
  ↓
mylab:1.0.0
```

برای Lab و Snapshot سریع مفید است، اما برای پروژه واقعی روش اصلی نیست؛ چون مراحل ساخت داخل Code ثبت نشده‌اند. در پروژه واقعی `Dockerfile` روش قابل‌تکرارتر است.

---

<a id="archive"></a>
## 17. `save/load` در برابر `export/import`

دو خانواده جدا داریم.

### Image

```text
Image
  ↓ docker save
tar
  ↓ docker load
Image
```

```bash
docker save myapp:1.0.0 -o myapp.tar
docker load -i myapp.tar
```

### Container File System

```text
Container
  ↓ docker export
tar
  ↓ docker import
New Image
```

```bash
docker export mycontainer -o container.tar
docker import container.tar imported-app:1.0.0
```

| موضوع | `save/load` | `export/import` |
|---|---|---|
| مبدأ | Image | Container |
| هدف | انتقال/Restore Image | File System Container |
| Layerهای Image | حفظ می‌شوند | ساختار اصلی Layerها حفظ نمی‌شود |
| Runtime metadata | مناسب‌تر برای Restore | ممکن است از دست برود |

Image ساخته‌شده با `docker import` ممکن است Command پیش‌فرض مناسب نداشته باشد. در آن حالت باید Command را صریح بدهی:

```bash
docker run -it imported-app:1.0.0 bash
```

---

<a id="cleanup"></a>
## 18. پاک‌سازی Container و Image

Container:

```bash
docker rm container-name
docker rm $(docker ps -a -q)
docker rm -f $(docker ps -a -q)
```

Image:

```bash
docker rmi image-name
docker rmi $(docker images -q)
```

این دستور معمولاً اشتباه مفهومی است:

```bash
docker rmi $(docker ps -a -q)
```

چون `docker ps` شناسه Container می‌دهد ولی `docker rmi` برای Image است.

Prune:

```bash
docker container prune
docker image prune
docker volume prune
docker network prune
docker system prune
```

قبل از `volume prune` مطمئن شو Data مهمی حذف نمی‌شود.

---

<a id="errors"></a>
## 19. خطاهای رایج و روش فکر کردن برای Debug

### `command not found`

Image `ubuntu` الزاماً Python، Node یا GCC ندارد. Image مناسب را انتخاب کن یا Package لازم را نصب کن.

### `docker` داخل Container وجود ندارد

Docker Engine روی Host اجرا می‌شود؛ Container معمولی الزاماً Docker CLI ندارد.

### `exec` روی Container متوقف‌شده

```bash
docker start web
docker exec -it web bash
```

### Port درست Publish شده ولی برنامه جواب نمی‌دهد

این سه سؤال را بررسی کن:

```text
Application Running است؟
داخل Container روی چه Portی Listen می‌کند؟
روی 0.0.0.0 گوش می‌دهد یا فقط 127.0.0.1؟
```

ابزارها:

```bash
docker logs web
docker inspect web
```

### تغییر Host داخل Container دیده نمی‌شود

بپرس فایل با `docker cp`/`COPY` آمده یا Bind Mount است.

### مدل Debug پنج مرحله‌ای

```text
1. State   → docker ps -a
2. Logs    → docker logs
3. Config  → docker inspect
4. Process / Port
5. Storage / Network
```

قبل از `rm -f` همه‌چیز، این مراحل اطلاعات بسیار بیشتری می‌دهند.

---

<a id="lab"></a>
## 20. تمرین نهایی

Nginx:

```bash
docker run -d \
  --name web-lab \
  -p 8080:80 \
  nginx:alpine
```

تست:

```bash
docker ps
curl http://localhost:8080
```

Log و Inspect:

```bash
docker logs web-lab
docker inspect web-lab
```

ورود:

```bash
docker exec -it web-lab sh
```

حالا Bind Mount:

```bash
docker rm -f web-lab
mkdir -p nginx-lab
echo '<h1>Hello from host</h1>' > nginx-lab/index.html

docker run -d \
  --name web-lab \
  -p 8080:80 \
  -v "$PWD/nginx-lab":/usr/share/nginx/html:ro \
  nginx:alpine
```

تست:

```bash
curl http://localhost:8080
```

Network:

```bash
docker network create lab-net
docker run -d --name helper --network lab-net alpine sleep 3600
docker network connect lab-net web-lab
docker network inspect lab-net
```

اگر بتوانی بعد از این تمرین `Image`، `Container`، `run`، `start`، `exec`، Port Mapping، Bind Mount، Network، `inspect` و `logs` را توضیح دهی، آماده‌ی سند دوم هستی.

---

<a id="cheatsheet"></a>
## 21. Cheat Sheet

### Container

```bash
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
```

### Image

```bash
docker pull IMAGE
docker image ls
docker rmi IMAGE
docker tag SOURCE TARGET
docker save IMAGE -o image.tar
docker load -i image.tar
```

### Storage

```bash
docker cp file.txt container:/tmp/file.txt
docker volume create data
docker volume ls
docker volume inspect data
docker run -v data:/data IMAGE
docker run -v "$PWD":/app IMAGE
```

### Network

```bash
docker network ls
docker network create app-net
docker network inspect app-net
docker network connect app-net CONTAINER
docker network disconnect app-net CONTAINER
```

### Port و Environment

```bash
docker run -p 8080:80 IMAGE
docker run -e APP_ENV=dev IMAGE
docker run --env-file .env IMAGE
```

---

<a id="next"></a>
## 22. بعد از این سند چه بخوانیم؟

```text
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
```

سند دوم از همین نقطه شروع می‌شود: ساخت Image قابل‌تکرار با `Dockerfile` و مدیریت چند Service با `Docker Compose`.

</div>