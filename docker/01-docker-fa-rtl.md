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

1\. [Docker چه مسئله‌ای را حل می‌کند؟](#docker-problem)  
2\. [Container با Virtual Machine چه فرقی دارد؟](#container-vm)  
3\. [معماری Docker](#architecture)  
4\. [Image و Container](#image-container)  
5\. [اولین اجرای Docker](#first-run)  
6\. [دستور docker run را درست بخوانیم](#docker-run)  
7\. [چرخه‌ی زندگی Container](#lifecycle)  
8\. [ورود به Container و اجرای دستور با exec](#exec)  
9\. [مشاهده وضعیت، Log و Inspect](#observe)  
10\. [Port و Port Mapping](#ports)  
11\. [Environment Variable](#environment)  
12\. [Storage: writable layer، docker cp، Bind Mount و Volume](#storage)  
13\. [سناریوی عملی docker cp در برابر Bind Mount](#cp-vs-bind)  
14\. [Docker Network](#network)  
15\. [مدیریت Image، Tag و Registry](#images)  
16\. [docker commit چه زمانی مفید است؟](#commit)  
17\. [save/load در برابر export/import](#archive)  
18\. [پاک‌سازی Container و Image](#cleanup)  
19\. [خطاهای رایج و روش فکر کردن برای Debug](#errors)  
20\. [تمرین نهایی](#lab)  
21\. [Cheat Sheet](#cheatsheet)  
22\. [بعد از این سند چه بخوانیم؟](#next)  

</details>

---

<a id="docker-problem"></a>

| سرویس                   | پورت پیش‌فرض | کاربرد               |
| ----------------------- | -----------: | -------------------- |
| **Nginx**               |         `80` | HTTP                 |
| **Nginx**               |        `443` | HTTPS                |
| **Apache**              |         `80` | HTTP                 |
| **SSH**                 |         `22` | اتصال SSH            |
| **FTP**                 |         `21` | FTP                  |
| **DNS**                 |         `53` | DNS                  |
| **MySQL / MariaDB**     |       `3306` | دیتابیس              |
| **PostgreSQL**          |       `5432` | دیتابیس              |
| **MongoDB**             |      `27017` | NoSQL                |
| **Redis**               |       `6379` | Cache / Database     |
| **Memcached**           |      `11211` | Cache                |
| **RabbitMQ**            |       `5672` | AMQP                 |
| **RabbitMQ Management** |      `15672` | پنل وب RabbitMQ      |
| **Kafka**               |       `9092` | Kafka                |
| **Elasticsearch**       |       `9200` | REST API             |
| **Elasticsearch**       |       `9300` | ارتباط داخلی Nodeها  |
| **Kibana**              |       `5601` | Web UI               |
| **Prometheus**          |       `9090` | Monitoring           |
| **Grafana**             |       `3000` | Monitoring Dashboard |
| **Jenkins**             |       `8080` | Web UI               |
| **Jenkins Agent**       |      `50000` | ارتباط Agent         |
| **Docker Registry**     |       `5000` | Private Registry     |
| **cAdvisor**            |       `8080` | Container monitoring |
| **Portainer**           |       `9000` | Docker Management    |
| **Portainer HTTPS**     |       `9443` | پنل HTTPS            |
| **Node.js**             |       `3000` | Web Application      |
| **React Dev Server**    |       `3000` | Development          |
| **Next.js**             |       `3000` | Web Application      |
| **Flask**               |       `5000` | Web Application      |
| **Django**              |       `8000` | Web Application      |
| **Spring Boot**         |       `8080` | Web Application      |
| **PHP-FPM**             |       `9000` | PHP FastCGI          |

## 1. Docker چه مسئله‌ای را حل می‌کند؟

یک برنامه معمولاً فقط «کد» نیست. ممکن است برای اجرا به نسخه‌ی مشخصی از `Python` یا `Node.js`، Libraryها، Environment Variableها، فایل Configuration و ابزارهای سیستم‌عاملی نیاز داشته باشد.

اگر این وابستگی‌ها را مستقیماً روی هر Server یا Laptop نصب کنیم، خیلی زود با تفاوت نسخه‌ها و تداخل Packageها روبه‌رو می‌شویم. Docker کمک می‌کند **محیط اجرای برنامه را به شکل استاندارد بسته‌بندی و اجرا کنیم**.

</div>

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

<div dir="rtl" align="right">


Docker قرار نیست Host OS را حذف کند. Container روی Host اجرا می‌شود و از Kernel آن استفاده می‌کند، اما Processها، File System و Network خودش را به شکل ایزوله‌تری می‌بیند.

---

<a id="container-vm"></a>
## 2. Container با Virtual Machine چه فرقی دارد؟

Virtual Machine معمولاً یک Guest OS کامل دارد:

</div>

```text
Hardware
└── Host OS
    └── Hypervisor
        ├── Guest OS A
        │   └── Application
        └── Guest OS B
            └── Application
```

<div dir="rtl" align="right">


Containerها معمولاً Kernel Host را به اشتراک می‌گذارند:

</div>

```text
Hardware
└── Host OS
    └── Docker Engine
        ├── Container A
        │   └── Application
        └── Container B
            └── Application
```

<div dir="rtl" align="right">


Containerها معمولاً سریع‌تر Start می‌شوند و سبک‌ترند، اما این نتیجه را نگیریم که `Container = VM کوچک`. این دو ابزار دقیقاً یک مسئله را حل نمی‌کنند.

---

<a id="architecture"></a>
## 3. معماری Docker

وقتی می‌نویسیم:

</div>

```bash
docker run nginx
```

<div dir="rtl" align="right">


چند جزء درگیر هستند:

</div>

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

<div dir="rtl" align="right">


`Docker CLI` همان دستورهایی است که می‌نویسیم. `Docker Daemon` عملیات واقعی را انجام می‌دهد. `Registry` هم محل نگهداری Imageهاست و Docker Hub یکی از Registryهای شناخته‌شده است.

برای ادامه فرض می‌کنیم Docker نصب شده است. بررسی سریع:

</div>

```bash
docker --version
docker info
```

<div dir="rtl" align="right">


---

<a id="image-container"></a>
## 4. Image و Container

### Image

Image یک الگوی آماده برای ساخت Container است:

</div>

```text
ubuntu:24.04
nginx:alpine
python:3.13-slim
node:22
```

<div dir="rtl" align="right">


Image خودش Process در حال اجرا نیست.

### Container

Container نمونه‌ای است که از روی Image ساخته و اجرا می‌شود:

</div>

```text
nginx:alpine
     │
     ├── web-1
     ├── web-2
     └── web-3
```

<div dir="rtl" align="right">


مدل ذهنی:

</div>

```text
Image
= الگو / بسته‌ی ساخت Container

Container
= Instance قابل اجرا از Image
```

<div dir="rtl" align="right">


### Layer

Imageها از Layerها ساخته می‌شوند. Layerهای مشترک می‌توانند بین Imageها reuse شوند. برای مشاهده‌ی مصرف فضا:

</div>

```bash
docker system df
docker system df -v
```

<div dir="rtl" align="right">


---

<a id="first-run"></a>
## 5. اولین اجرای Docker

</div>

```bash
docker run hello-world
```

<div dir="rtl" align="right">


مدل آموزشی:

</div>

```text
docker run hello-world
        │
        ├── Image محلی وجود دارد؟
        ├── اگر نه: Pull
        ├── Create Container
        └── Start Container
```

<div dir="rtl" align="right">


پس `docker run` را فقط «روشن کردن» در نظر نگیر؛ معمولاً Container جدید می‌سازد و آن را اجرا می‌کند.

---

<a id="docker-run"></a>
## 6. دستور `docker run` را درست بخوانیم

فرم کلی:

</div>

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

<div dir="rtl" align="right">


اجرای ساده:

</div>

```bash
docker run nginx
```

<div dir="rtl" align="right">


Background:

</div>

```bash
docker run -d nginx
```

<div dir="rtl" align="right">


نام‌گذاری:

</div>

```bash
docker run -d --name web nginx
```

<div dir="rtl" align="right">


Interactive shell:

</div>

```bash
docker run -it ubuntu bash
```

<div dir="rtl" align="right">


- `-i` ورودی تعاملی را باز نگه می‌دارد.
- `-t` Terminal مجازی ایجاد می‌کند.
- `bash` Command داخل Container است.

Container موقت:

</div>

```bash
docker run --rm ubuntu echo "hello"
```

<div dir="rtl" align="right">


اگر Image یک Command پیش‌فرض داشته باشد، Command انتهای `docker run` می‌تواند آن را Override کند. مثلاً:

</div>

```bash
docker run -it myapp bash
```

<div dir="rtl" align="right">


ممکن است به‌جای Application، Bash را اجرا کند.

---

<a id="lifecycle"></a>
## 7. چرخه‌ی زندگی Container

</div>

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

<div dir="rtl" align="right">


دستورهای اصلی:

</div>

```bash
docker ps
docker ps -a
docker stop web
docker start web
docker restart web
docker rm web
docker rm -f web
```

<div dir="rtl" align="right">


تفاوت مهم:

</div>

```text
docker run
→ معمولاً Container جدید می‌سازد و اجرا می‌کند

docker start
→ همان Container موجود را دوباره اجرا می‌کند
```

<div dir="rtl" align="right">


`-f` یعنی Force. در Lab مفید است، اما در محیط واقعی بدون دلیل از آن استفاده نکن.

---

<a id="exec"></a>
## 8. ورود به Container و اجرای دستور با `exec`

فرض کن `web` Running است.

اجرای یک Command:

</div>

```bash
docker exec web ls -lah /
```

<div dir="rtl" align="right">


ورود به Bash:

</div>

```bash
docker exec -it web bash
```

<div dir="rtl" align="right">


اگر Bash موجود نبود:

</div>

```bash
docker exec -it web sh
```

<div dir="rtl" align="right">


تفاوت:

</div>

```text
docker start web
→ خود Container را Running می‌کند

docker exec -it web bash
→ داخل Container Running یک Process جدید باز می‌کند
```

<div dir="rtl" align="right">


روی Container متوقف‌شده ابتدا باید `start` انجام شود.

---

<a id="observe"></a>
## 9. مشاهده وضعیت، Log و Inspect

قبل از حذف و ساخت مجدد همه‌چیز، اطلاعات جمع کن.

</div>

```bash
docker logs web
docker logs -f web
docker top web
docker stats
docker inspect web
docker diff web
```

<div dir="rtl" align="right">


`docker inspect` برای دیدن مواردی مثل IP، Network، Mount، Environment Variable، Port Binding، State و Image بسیار مهم است.

---

<a id="ports"></a>
## 10. Port و Port Mapping

فرض کن Application داخل Container روی Port `3000` گوش می‌دهد.

فرم Publish:

</div>

```bash
-p HOST_PORT:CONTAINER_PORT
```

<div dir="rtl" align="right">


مثال:

</div>

```bash
docker run -p 2000:3000 myapp
```

<div dir="rtl" align="right">

</div>

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

<div dir="rtl" align="right">


Host Port لازم نیست با Container Port یکی باشد:

</div>

```bash
docker run -p 5000:3000 myapp
```

<div dir="rtl" align="right">


### `0.0.0.0` داخل Application

اگر Application فقط روی `127.0.0.1` داخل Container گوش دهد، معمولاً از بیرون Container قابل دسترسی نیست. برای مثال Flask:

</div>

```python
app.run(host="0.0.0.0", port=4000)
```

<div dir="rtl" align="right">


### `port is already allocated`

اگر Host Port اشغال باشد:

</div>

```bash
docker ps
ss -tulpn
```

<div dir="rtl" align="right">


یا Process قبلی را Stop کن یا Host Port دیگری انتخاب کن.

### `EXPOSE` با `-p` یکی نیست

`EXPOSE` بیشتر Port مورد انتظار Image را بیان می‌کند. Publish واقعی با `-p` یا تنظیم معادل در Compose انجام می‌شود.

---

<a id="environment"></a>
## 11. Environment Variable

</div>

```bash
docker run \
  -e APP_ENV=development \
  -e PORT=3000 \
  myapp
```

<div dir="rtl" align="right">


فایل env:

</div>

```bash
docker run --env-file .env myapp
```

<div dir="rtl" align="right">


نمونه `.env`:

</div>

```ini
APP_ENV=development
PORT=3000
DB_HOST=database
```

<div dir="rtl" align="right">


برای Repository عمومی، Secret واقعی را Commit نکن. `.env.example` برای نمایش نام Variableها مناسب است.

---

<a id="storage"></a>
## 12. Storage: writable layer، `docker cp`، Bind Mount و Volume

### Writable Layer

فایلی که داخل Container می‌سازی در File System همان Container قرار می‌گیرد. Stop/Start معمولاً آن را حفظ می‌کند، اما با حذف Container روی آن برای Data مهم حساب نکن.

### `docker cp`

</div>

```bash
docker cp file.txt web:/tmp/file.txt
docker cp web:/tmp/file.txt ./file.txt
```

<div dir="rtl" align="right">


قاعده:

</div>

```text
docker cp
= Copy
≠ Sync
```

<div dir="rtl" align="right">


### Bind Mount

</div>

```bash
docker run -v "$PWD":/app myapp
```

<div dir="rtl" align="right">

</div>

```text
Host directory
     ⇅
Container directory
```

<div dir="rtl" align="right">


برای Development مفید است؛ تغییر Host داخل Container دیده می‌شود. اما حذف یا تغییر فایل Mount‌شده داخل Container می‌تواند روی Host هم اثر بگذارد.

### Named Volume

</div>

```bash
docker volume create db-data

docker run \
  -v db-data:/var/lib/postgresql/data \
  postgres
```

<div dir="rtl" align="right">


بررسی:

</div>

```bash
docker volume ls
docker volume inspect db-data
```

<div dir="rtl" align="right">


### tmpfs

برای Data موقتی در Memory:

</div>

```bash
docker run --tmpfs /cache nginx
```

<div dir="rtl" align="right">


مدل نهایی:

</div>

```text
Writable layer → وابسته به عمر Container
Bind Mount     → مسیر واقعی Host
Named Volume   → Storage مدیریت‌شده Docker
tmpfs          → Memory و موقت
```

<div dir="rtl" align="right">


---

<a id="cp-vs-bind"></a>
## 13. سناریوی عملی `docker cp` در برابر Bind Mount

فرض کنیم روی Host فایل `wilson.c` داریم.

Container اول:

</div>

```bash
docker run -it -d --name copied ubuntu bash
docker cp wilson.c copied:/home/wilson.c
```

<div dir="rtl" align="right">


از این لحظه دو نسخه مستقل داریم.

Container دوم:

</div>

```bash
docker run -it -d \
  --name mounted \
  -v "$PWD":/code \
  gcc bash
```

<div dir="rtl" align="right">


داخل Container:

</div>

```bash
docker exec -it mounted bash
cd /code
gcc -o main wilson.c
./main
```

<div dir="rtl" align="right">


چون `/code` Bind Mount است، فایل `main` روی Host هم دیده می‌شود.

حالا فایل Host را تغییر بده. Container اول نسخه قبلی را می‌بیند:

</div>

```bash
docker exec copied cat /home/wilson.c
```

<div dir="rtl" align="right">


Container دوم نسخه جدید را می‌بیند:

</div>

```bash
docker exec mounted cat /code/wilson.c
```

<div dir="rtl" align="right">


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

</div>

```bash
docker network ls
```

<div dir="rtl" align="right">


Network اختصاصی:

</div>

```bash
docker network create app-net
```

<div dir="rtl" align="right">


دو Container روی یک Network:

</div>

```bash
docker run -d --name database --network app-net postgres

docker run -d --name backend --network app-net my-backend
```

<div dir="rtl" align="right">


در User-defined Network، بهتر است از نام Container/Service استفاده کنیم:

</div>

```text
database:5432
```

<div dir="rtl" align="right">


نه IP موقت:

</div>

```text
172.18.0.5:5432
```

<div dir="rtl" align="right">


بررسی:

</div>

```bash
docker network inspect app-net
```

<div dir="rtl" align="right">


اتصال و جداسازی:

</div>

```bash
docker network connect app-net web
docker network disconnect app-net web
```

<div dir="rtl" align="right">


Driverهایی که فعلاً باید اسمشان را بشناسی:

</div>

```text
bridge   → رایج روی یک Host
host     → استفاده نزدیک‌تر از Network Host
none     → بدون Network معمول Docker
overlay  → سناریوهای چند Host
macvlan  → سناریوهای خاص شبکه
```

<div dir="rtl" align="right">


برای Junior مهم‌ترین بخش فعلاً `bridge` و User-defined Network است.

---

<a id="images"></a>
## 15. مدیریت Image، Tag و Registry

</div>

```bash
docker pull nginx:alpine
docker image ls
docker rmi nginx:alpine
```

<div dir="rtl" align="right">


Tag:

</div>

```bash
docker tag myapp:1.0.0 myapp:stable
```

<div dir="rtl" align="right">


Tag الزاماً Image را کپی نمی‌کند؛ یک Reference جدید می‌سازد.

الگوی رایج نام Image:

</div>

```text
REGISTRY/OWNER/IMAGE:TAG
```

<div dir="rtl" align="right">


---

<a id="commit"></a>
## 16. `docker commit` چه زمانی مفید است؟

</div>

```bash
docker run -it --name lab ubuntu bash
```

<div dir="rtl" align="right">


داخل Container:

</div>

```bash
apt update
apt install -y curl
touch /home/example.txt
exit
```

<div dir="rtl" align="right">


ساخت Image از وضعیت فعلی:

</div>

```bash
docker commit lab mylab:1.0.0
```

<div dir="rtl" align="right">

</div>

```text
ubuntu
  ↓
lab container
  ↓ manual changes
docker commit
  ↓
mylab:1.0.0
```

<div dir="rtl" align="right">


برای Lab و Snapshot سریع مفید است، اما برای پروژه واقعی روش اصلی نیست؛ چون مراحل ساخت داخل Code ثبت نشده‌اند. در پروژه واقعی `Dockerfile` روش قابل‌تکرارتر است.

---

<a id="archive"></a>
## 17. `save/load` در برابر `export/import`

دو خانواده جدا داریم.

### Image

</div>

```text
Image
  ↓ docker save
tar
  ↓ docker load
Image
```

<div dir="rtl" align="right">

</div>

```bash
docker save myapp:1.0.0 -o myapp.tar
docker load -i myapp.tar
```

<div dir="rtl" align="right">


### Container File System

</div>

```text
Container
  ↓ docker export
tar
  ↓ docker import
New Image
```

<div dir="rtl" align="right">

</div>

```bash
docker export mycontainer -o container.tar
docker import container.tar imported-app:1.0.0
```

<div dir="rtl" align="right">


| موضوع | `save/load` | `export/import` |
|---|---|---|
| مبدأ | Image | Container |
| هدف | انتقال/Restore Image | File System Container |
| Layerهای Image | حفظ می‌شوند | ساختار اصلی Layerها حفظ نمی‌شود |
| Runtime metadata | مناسب‌تر برای Restore | ممکن است از دست برود |

Image ساخته‌شده با `docker import` ممکن است Command پیش‌فرض مناسب نداشته باشد. در آن حالت باید Command را صریح بدهی:

</div>

```bash
docker run -it imported-app:1.0.0 bash
```

<div dir="rtl" align="right">


---

<a id="cleanup"></a>
## 18. پاک‌سازی Container و Image

Container:

</div>

```bash
docker rm container-name
docker rm $(docker ps -a -q)
docker rm -f $(docker ps -a -q)
```

<div dir="rtl" align="right">


Image:

</div>

```bash
docker rmi image-name
docker rmi $(docker images -q)
```

<div dir="rtl" align="right">


این دستور معمولاً اشتباه مفهومی است:

</div>

```bash
docker rmi $(docker ps -a -q)
```

<div dir="rtl" align="right">


چون `docker ps` شناسه Container می‌دهد ولی `docker rmi` برای Image است.

Prune:

</div>

```bash
docker container prune
docker image prune
docker volume prune
docker network prune
docker system prune
```

<div dir="rtl" align="right">


قبل از `volume prune` مطمئن شو Data مهمی حذف نمی‌شود.

---

<a id="errors"></a>
## 19. خطاهای رایج و روش فکر کردن برای Debug

### `command not found`

Image `ubuntu` الزاماً Python، Node یا GCC ندارد. Image مناسب را انتخاب کن یا Package لازم را نصب کن.

### `docker` داخل Container وجود ندارد

Docker Engine روی Host اجرا می‌شود؛ Container معمولی الزاماً Docker CLI ندارد.

### `exec` روی Container متوقف‌شده

</div>

```bash
docker start web
docker exec -it web bash
```

<div dir="rtl" align="right">


### Port درست Publish شده ولی برنامه جواب نمی‌دهد

این سه سؤال را بررسی کن:

</div>

```text
Application Running است؟
داخل Container روی چه Portی Listen می‌کند؟
روی 0.0.0.0 گوش می‌دهد یا فقط 127.0.0.1؟
```

<div dir="rtl" align="right">


ابزارها:

</div>

```bash
docker logs web
docker inspect web
```

<div dir="rtl" align="right">


### تغییر Host داخل Container دیده نمی‌شود

بپرس فایل با `docker cp`/`COPY` آمده یا Bind Mount است.

### مدل Debug پنج مرحله‌ای

</div>

```text
1. State   → docker ps -a
2. Logs    → docker logs
3. Config  → docker inspect
4. Process / Port
5. Storage / Network
```

<div dir="rtl" align="right">


قبل از `rm -f` همه‌چیز، این مراحل اطلاعات بسیار بیشتری می‌دهند.

---

<a id="lab"></a>
## 20. تمرین نهایی

Nginx:

</div>

```bash
docker run -d \
  --name web-lab \
  -p 8080:80 \
  nginx:alpine
```

<div dir="rtl" align="right">


تست:

</div>

```bash
docker ps
curl http://localhost:8080
```

<div dir="rtl" align="right">


Log و Inspect:

</div>

```bash
docker logs web-lab
docker inspect web-lab
```

<div dir="rtl" align="right">


ورود:

</div>

```bash
docker exec -it web-lab sh
```

<div dir="rtl" align="right">


حالا Bind Mount:

</div>

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

<div dir="rtl" align="right">


تست:

</div>

```bash
curl http://localhost:8080
```

<div dir="rtl" align="right">


Network:

</div>

```bash
docker network create lab-net
docker run -d --name helper --network lab-net alpine sleep 3600
docker network connect lab-net web-lab
docker network inspect lab-net
```

<div dir="rtl" align="right">


اگر بتوانی بعد از این تمرین `Image`، `Container`، `run`، `start`، `exec`، Port Mapping، Bind Mount، Network، `inspect` و `logs` را توضیح دهی، آماده‌ی سند دوم هستی.

---

<a id="cheatsheet"></a>
## 21. Cheat Sheet

### Container

</div>

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

<div dir="rtl" align="right">


### Image

</div>

```bash
docker pull IMAGE
docker image ls
docker rmi IMAGE
docker tag SOURCE TARGET
docker save IMAGE -o image.tar
docker load -i image.tar
```

<div dir="rtl" align="right">


### Storage

</div>

```bash
docker cp file.txt container:/tmp/file.txt
docker volume create data
docker volume ls
docker volume inspect data
docker run -v data:/data IMAGE
docker run -v "$PWD":/app IMAGE
```

<div dir="rtl" align="right">


### Network

</div>

```bash
docker network ls
docker network create app-net
docker network inspect app-net
docker network connect app-net CONTAINER
docker network disconnect app-net CONTAINER
```

<div dir="rtl" align="right">


### Port و Environment

</div>

```bash
docker run -p 8080:80 IMAGE
docker run -e APP_ENV=dev IMAGE
docker run --env-file .env IMAGE
```

<div dir="rtl" align="right">


---

<a id="next"></a>
## 22. بعد از این سند چه بخوانیم؟

</div>

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

<div dir="rtl" align="right">


سند دوم از همین نقطه شروع می‌شود: ساخت Image قابل‌تکرار با `Dockerfile` و مدیریت چند Service با `Docker Compose`.

</div>