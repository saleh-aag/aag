<div dir="rtl">

# 🐳 راهنمای جامع <span dir="ltr">Docker</span> به زبان فارسی

از «<span dir="ltr">Docker</span> چیه؟» تا <span dir="ltr">Image</span>، <span dir="ltr">Container</span>، <span dir="ltr">Dockerfile</span>، <span dir="ltr">Volume</span>، <span dir="ltr">Network</span> و <span dir="ltr">Docker Hub</span>

این راهنما برای کسی نوشته شده که ممکن است هیچ پیش‌زمینه‌ای از <span dir="ltr">Docker</span> نداشته باشد.

> 💡 **نکتهٔ نمایش:** این فایل با `<div dir="rtl">` باز شده تا جهت پایهٔ همهٔ بلوک‌ها RTL باشد؛ اصطلاحات انگلیسی همچنان با <span dir="ltr">span dir="ltr"</span> ایزوله شده‌اند و <span dir="ltr">Command</span>ها / <span dir="ltr">Code Block</span>ها نیز <span dir="ltr">LTR</span> باقی می‌مانند تا ترکیب فارسی و انگلیسی در <span dir="ltr">GitHub</span> به‌هم نریزد.

هدف این نیست که فقط چند دستور حفظ کنیم؛ هدف این است که بفهمیم <span dir="ltr">Docker</span> دقیقاً چه کاری انجام می‌دهد و چرا هر دستور را می‌زنیم.

## 📚 فهرست مطالب

🎯 اگر دنبال موضوع خاصی هستی، مستقیم از این فهرست بپر همان بخش؛ لازم نیست فایل را مثل کتاب درسی از اول تا آخر بخوانی 😄

- [🐳 راهنمای جامع Docker به زبان فارسی](#-راهنمای-جامع-docker-به-زبان-فارسی)
  - [📚 فهرست مطالب](#-فهرست-مطالب)
  - [1. 🐳 Docker چیست؟](#1--docker-چیست)
  - [2. 😵 قبل از Docker چه مشکلی داشتیم؟](#2--قبل-از-docker-چه-مشکلی-داشتیم)
  - [3. 🆚 Container با Virtual Machine چه فرقی دارد؟](#3--container-با-virtual-machine-چه-فرقی-دارد)
    - [نتیجه](#نتیجه)
  - [4. 🧠 معماری Docker](#4--معماری-docker)
  - [5. ⚙️ Docker Engine، Daemon، CLI و REST API](#5-️-docker-engine-daemon-cli-و-rest-api)
    - [Docker Engine](#docker-engine)
    - [Docker Daemon](#docker-daemon)
    - [Docker CLI](#docker-cli)
    - [REST API](#rest-api)
  - [6. 📦 نصب Docker Engine روی Ubuntu](#6--نصب-docker-engine-روی-ubuntu)
    - [سیستم‌های Ubuntu پشتیبانی‌شده](#سیستمهای-ubuntu-پشتیبانیشده)
    - [مرحله 1 — حذف Packageهای متداخل](#مرحله-1--حذف-packageهای-متداخل)
    - [مرحله 2 — نصب پیش‌نیازها](#مرحله-2--نصب-پیشنیازها)
    - [مرحله 3 — اضافه کردن GPG Key رسمی Docker](#مرحله-3--اضافه-کردن-gpg-key-رسمی-docker)
    - [مرحله 4 — اضافه کردن Repository رسمی](#مرحله-4--اضافه-کردن-repository-رسمی)
    - [مرحله 5 — نصب Docker](#مرحله-5--نصب-docker)
    - [مرحله 6 — بررسی وضعیت Docker](#مرحله-6--بررسی-وضعیت-docker)
    - [مرحله 7 — تست](#مرحله-7--تست)
    - [بررسی Version](#بررسی-version)
    - [اجرای Docker بدون sudo](#اجرای-docker-بدون-sudo)
  - [7. 👶 اولین اجرای Docker](#7--اولین-اجرای-docker)
  - [8. 🖼️ Image چیست؟](#8-️-image-چیست)
  - [9. 📦 Container چیست؟](#9--container-چیست)
  - [10. 🔄 چرخهٔ زندگی Container](#10--چرخهٔ-زندگی-container)
  - [11. 🚀 دستور `docker run` به زبان آدمیزاد](#11--دستور-docker-run-به-زبان-آدمیزاد)
  - [12. 🎛️ مدیریت Containerها](#12-️-مدیریت-containerها)
  - [13. 🚪 اجرای دستور داخل Container با `docker exec`](#13--اجرای-دستور-داخل-container-با-docker-exec)
  - [14. 📁 کپی فایل با `docker cp`](#14--کپی-فایل-با-docker-cp)
  - [15. 📊 مانیتورینگ با `docker stats` و `docker events`](#15--مانیتورینگ-با-docker-stats-و-docker-events)
  - [16. 🌐 Port، EXPOSE و Publish](#16--port-expose-و-publish)
  - [17. 🌱 Environment Variable](#17--environment-variable)
  - [18. 🧅 Docker Image و Layerها](#18--docker-image-و-layerها)
  - [19. 📸 ساخت Image با `docker commit`](#19--ساخت-image-با-docker-commit)
  - [20. 📝 Dockerfile چیست؟](#20--dockerfile-چیست)
  - [21. 🧰 دستورهای مهم Dockerfile](#21--دستورهای-مهم-dockerfile)
  - [22. 🥊 CMD در برابر ENTRYPOINT](#22--cmd-در-برابر-entrypoint)
  - [23. 📂 COPY در برابر ADD](#23--copy-در-برابر-add)
  - [24. 🏗️ Build کردن Image](#24-️-build-کردن-image)
  - [25. ⚡ Docker Build Cache](#25--docker-build-cache)
  - [26. 🏷️ Tag و Versioning](#26-️-tag-و-versioning)
  - [27. 🌍 Docker Registry و Docker Hub](#27--docker-registry-و-docker-hub)
  - [28. ☁️ Push کردن Image به Docker Hub](#28-️-push-کردن-image-به-docker-hub)
  - [29. 💾 `docker save/load` در برابر `export/import`](#29--docker-saveload-در-برابر-exportimport)
  - [30. 💽 Storage در Docker](#30--storage-در-docker)
  - [31. 📦 Volume چیست؟](#31--volume-چیست)
  - [32. 🔗 Bind Mount چیست؟](#32--bind-mount-چیست)
  - [33. 🧠 tmpfs چیست؟](#33--tmpfs-چیست)
  - [34. ⚖️ Volume یا Bind Mount؟](#34-️-volume-یا-bind-mount)
  - [35. 🌐 Docker Network چیست؟](#35--docker-network-چیست)
  - [36. 🌉 Bridge Network](#36--bridge-network)
  - [37. 🔎 User-defined Network و DNS داخلی](#37--user-defined-network-و-dns-داخلی)
  - [38. 🗺️ Host، None، Overlay و Macvlan](#38-️-host-none-overlay-و-macvlan)
  - [39. 🚪 Port Publishing در شبکه](#39--port-publishing-در-شبکه)
  - [40. 🧩 Docker Compose چیست؟](#40--docker-compose-چیست)
  - [41. 🧹 پاک‌سازی Docker و `prune`](#41--پاکسازی-docker-و-prune)
  - [42. 🔬 `docker inspect`](#42--docker-inspect)
  - [43. 🧾 دستورهای طلایی Docker](#43--دستورهای-طلایی-docker)
  - [44. 🤦 اشتباهات رایج مبتدی‌ها](#44--اشتباهات-رایج-مبتدیها)
  - [45. 🧪 تمرین عملی از صفر](#45--تمرین-عملی-از-صفر)
  - [46. 🗺️ نقشهٔ راه بعد از این راهنما](#46-️-نقشهٔ-راه-بعد-از-این-راهنما)
    - [🧠 خلاصهٔ نهایی در 60 ثانیه](#-خلاصهٔ-نهایی-در-60-ثانیه)
  - [47. 📖 منابع](#47--منابع)
  - [❤️ مشارکت](#️-مشارکت)

---

<a id="what-is-docker"></a>
## 1. 🐳 <span dir="ltr">Docker</span> چیست؟

فرض کن یک برنامه نوشته‌ای که برای اجرا شدن به این چیزها نیاز دارد:

- <span dir="ltr">Python 3.12</span>
- چند <span dir="ltr">Library</span> مشخص
- <span dir="ltr">Nginx</span>
- یک نسخهٔ خاص از <span dir="ltr">Node.js</span>
- چند <span dir="ltr">Environment Variable</span>
- چند فایل <span dir="ltr">Configuration</span>

روی لپ‌تاپ خودت همه‌چیز عالی کار می‌کند.

بعد پروژه را برای دوستت می‌فرستی و جملهٔ معروف برنامه‌نویس‌ها متولد می‌شود:

> «ولی روی سیستم من کار می‌کرد!» 😐

<span dir="ltr">Docker</span> آمده تا این مشکل را تا حد زیادی حل کند.

<span dir="ltr">Docker</span> برنامه و چیزهایی را که برای اجرا نیاز دارد در یک محیط نسبتاً ایزوله به نام <span dir="ltr">Container</span> اجرا می‌کند.

به زبان خیلی ساده:

**<span dir="ltr">Docker</span> = روشی استاندارد برای بسته‌بندی و اجرای نرم‌افزار**

<a id="before-docker"></a>
## 2. 😵 قبل از <span dir="ltr">Docker</span> چه مشکلی داشتیم؟

فرض کن سه پروژه داری:

| پروژه | نیازمندی |
|---|---|
| <span dir="ltr">Project A</span> | <span dir="ltr">Python 3.10</span> |
| <span dir="ltr">Project B</span> | <span dir="ltr">Python 3.12</span> |
| <span dir="ltr">Project C</span> | <span dir="ltr">Node.js 22</span> |

ممکن است <span dir="ltr">Dependency</span>های این پروژه‌ها با هم تداخل داشته باشند.

<span dir="ltr">Docker</span> می‌گوید:

- <span dir="ltr">Project A</span> را در <span dir="ltr">Container</span> خودش اجرا کن.
- <span dir="ltr">Project B</span> را در <span dir="ltr">Container</span> خودش اجرا کن.
- <span dir="ltr">Project C</span> را هم جدا اجرا کن.

در نتیجه هر برنامه محیط خودش را دارد.

<a id="container-vs-vm"></a>
## 3. 🆚 <span dir="ltr">Container</span> با <span dir="ltr">Virtual Machine</span> چه فرقی دارد؟

<span dir="ltr">Virtual Machine</span> یا <span dir="ltr">VM</span> معمولاً یک سیستم‌عامل کامل دارد.

اما <span dir="ltr">Container</span>ها معمولاً <span dir="ltr">Kernel</span> سیستم <span dir="ltr">Host</span> را به اشتراک می‌گذارند.

**<span dir="ltr">Virtual Machine</span>**

```
Hardware
└── Host OS
    └── Hypervisor
        ├── Guest OS 1
        │   └── Application
        └── Guest OS 2
            └── Application
```

**<span dir="ltr">Container</span>**

```
Hardware
└── Host OS
    └── Docker Engine
        ├── Container 1
        │   └── Application
        └── Container 2
            └── Application
```

### نتیجه

<span dir="ltr">Container</span>ها معمولاً:

- سریع‌تر <span dir="ltr">Start</span> می‌شوند.
- فضای کمتری می‌گیرند.
- سبک‌تر از <span dir="ltr">VM</span> هستند.
- برای <span dir="ltr">Deployment</span> و <span dir="ltr">CI/CD</span> بسیار مناسب‌اند.

<span dir="ltr">Docker</span> جایگزین کامل <span dir="ltr">VM</span> نیست. هرکدام مسئلهٔ متفاوتی را حل می‌کنند.

<a id="docker-architecture"></a>
## 4. 🧠 معماری <span dir="ltr">Docker</span>

مهم‌ترین اجزای <span dir="ltr">Docker</span>:

```
You
 │
 │ docker run nginx
 ▼
Docker CLI
 │
 │ Docker API
 ▼
Docker Daemon (dockerd)
 │
 ├── Images
 ├── Containers
 ├── Networks
 └── Volumes
```

وقتی می‌نویسی:

```bash
docker run nginx
```

خود دستور `docker` قرار نیست <span dir="ltr">Container</span> را جادو کند!

<span dir="ltr">Docker CLI</span> درخواست را برای <span dir="ltr">Docker Daemon</span> می‌فرستد و <span dir="ltr">Daemon</span> کار واقعی را انجام می‌دهد.

<a id="docker-engine"></a>
## 5. ⚙️ <span dir="ltr">Docker Engine</span>، <span dir="ltr">Daemon</span>، <span dir="ltr">CLI</span> و <span dir="ltr">REST API</span>

### <span dir="ltr">Docker Engine</span>

هستهٔ اصلی <span dir="ltr">Docker</span> است.

### <span dir="ltr">Docker Daemon</span>

پردازشی با نام:

```
dockerd
```

که مسئول مدیریت این موارد است:

- <span dir="ltr">Container</span>
- <span dir="ltr">Image</span>
- <span dir="ltr">Network</span>
- <span dir="ltr">Volume</span>

### <span dir="ltr">Docker CLI</span>

همان دستور `docker` است:

```bash
docker ps
docker run nginx
docker image ls
```

### <span dir="ltr">REST API</span>

<span dir="ltr">CLI</span> از طریق <span dir="ltr">API</span> با <span dir="ltr">Docker Daemon</span> ارتباط می‌گیرد.

یعنی ابزارهای دیگر هم می‌توانند برنامه‌نویسی‌شده با <span dir="ltr">Docker</span> ارتباط برقرار کنند.

<a id="install-docker"></a>
## 6. 📦 نصب <span dir="ltr">Docker Engine</span> روی <span dir="ltr">Ubuntu</span>

این بخش بر اساس روش فعلی مستندات رسمی <span dir="ltr">Docker</span> نوشته شده است.

### سیستم‌های <span dir="ltr">Ubuntu</span> پشتیبانی‌شده

در زمان نگارش این راهنما، <span dir="ltr">Docker</span> برای نسخه‌های 64-<span dir="ltr">bit</span> زیر دستور رسمی دارد:

- <span dir="ltr">Ubuntu 22.04 LTS</span>
- <span dir="ltr">Ubuntu 24.04 LTS</span>
- <span dir="ltr">Ubuntu 26.04 LTS</span>

### مرحله 1 — حذف <span dir="ltr">Package</span>های متداخل

اگر قبلاً نسخه‌های دیگری نصب کرده‌ای، ممکن است لازم باشد <span dir="ltr">Package</span>های متداخل را حذف کنی.

```bash
sudo apt remove $(dpkg --get-selections \
  docker.io docker-compose docker-compose-v2 docker-doc \
  docker-buildx podman-docker containerd runc | cut -f1)
```

اگر چیزی نصب نباشد، خطای خاصی نیست.

### مرحله 2 — نصب پیش‌نیازها

```bash
sudo apt update
sudo apt install ca-certificates curl
```

### مرحله 3 — اضافه کردن <span dir="ltr">GPG Key</span> رسمی <span dir="ltr">Docker</span>

```bash
sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### مرحله 4 — اضافه کردن <span dir="ltr">Repository</span> رسمی

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

سپس:

```bash
sudo apt update
```

### مرحله 5 — نصب <span dir="ltr">Docker</span>

```bash
sudo apt install \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin
```

### مرحله 6 — بررسی وضعیت <span dir="ltr">Docker</span>

```bash
sudo systemctl status docker
```

اگر اجرا نشده بود:

```bash
sudo systemctl start docker
```

### مرحله 7 — تست

```bash
sudo docker run hello-world
```

### بررسی <span dir="ltr">Version</span>

```bash
docker --version
docker version
docker info
```

### اجرای <span dir="ltr">Docker</span> بدون sudo

می‌توانی <span dir="ltr">User</span> را عضو گروه `docker` کنی:

```bash
sudo usermod -aG docker $USER
```

بعد باید <span dir="ltr">Session</span> جدید بگیری؛ ساده‌ترین روش <span dir="ltr">Logout/Login</span> است.

> ⚠️ **نکتهٔ امنیتی:** عضویت در گروه `docker` عملاً دسترسی بسیار قدرتمندی روی <span dir="ltr">Host</span> می‌دهد. روی <span dir="ltr">Server</span>های حساس این موضوع را دست‌کم نگیر.

<a id="hello-docker"></a>
## 7. 👶 اولین اجرای <span dir="ltr">Docker</span>

```bash
docker run hello-world
```

چه اتفاقی می‌افتد؟

```
docker run hello-world
        │
        ├── آیا image محلی وجود دارد؟
        │
        ├── اگر نه → Pull از Registry
        │
        ├── Create Container
        │
        └── Start Container
```

یک فرمول ذهنی عالی:

```
docker run
≈
docker pull
+
docker create
+
docker start
```

<a id="docker-image"></a>
## 8. 🖼️ <span dir="ltr">Image</span> چیست؟

<span dir="ltr">Image</span> را می‌توان قالب آمادهٔ ساخت <span dir="ltr">Container</span> در نظر گرفت.

مثلاً:

```
nginx:latest
ubuntu:24.04
redis:8
python:3.13-slim
```

<span dir="ltr">Image</span> خودش <span dir="ltr">Process</span> در حال اجرا نیست.

برای اجرا باید از آن <span dir="ltr">Container</span> بسازیم.

**تشبیه**

```
Image     = قالب کیک
Container = کیکی که با آن قالب ساخته‌ای
```

از یک <span dir="ltr">Image</span> می‌توانی چند <span dir="ltr">Container</span> بسازی.

```
nginx image
 ├── nginx-container-1
 ├── nginx-container-2
 └── nginx-container-3
```

<a id="docker-container"></a>
## 9. 📦 <span dir="ltr">Container</span> چیست؟

<span dir="ltr">Container</span> یک <span dir="ltr">Instance</span> قابل اجرای <span dir="ltr">Image</span> است.

مثلاً:

```bash
docker run nginx
```

<span dir="ltr">Docker</span> از <span dir="ltr">Image</span> مربوط به <span dir="ltr">Nginx</span> یک <span dir="ltr">Container</span> می‌سازد.

<span dir="ltr">Container</span> می‌تواند:

- <span dir="ltr">Start</span> شود.
- <span dir="ltr">Stop</span> شود.
- <span dir="ltr">Restart</span> شود.
- حذف شود.
- <span dir="ltr">Network</span> داشته باشد.
- <span dir="ltr">Volume</span> داشته باشد.
- <span dir="ltr">Port</span> داشته باشد.
- <span dir="ltr">Resource Limit</span> داشته باشد.

<a id="container-lifecycle"></a>
## 10. 🔄 چرخهٔ زندگی <span dir="ltr">Container</span>

```
Created
   │
   ▼
Running
   │
   ├── stop
   ▼
Stopped / Exited
   │
   ├── start
   │      └── Running
   │
   └── rm
          └── Deleted
```

برای دیدن <span dir="ltr">Container</span>های درحال اجرا:

```bash
docker ps
```

همه:

```bash
docker ps -a
```

<a id="docker-run"></a>
## 11. 🚀 دستور `docker run` به زبان آدمیزاد

**اجرای ساده**

```bash
docker run nginx
```

**اجرای <span dir="ltr">Background</span>**

```bash
docker run -d nginx
```

`-d` یعنی: `detached`

**دادن اسم**

```bash
docker run -d --name web nginx
```

**<span dir="ltr">Interactive Terminal</span>**

```bash
docker run -it ubuntu bash
```

- `-i` → ورودی <span dir="ltr">Terminal</span> باز بماند.
- `-t` → <span dir="ltr">pseudo-TTY</span> بسازد.

**حذف خودکار بعد از پایان**

```bash
docker run --rm ubuntu echo "Hello Docker"
```

**<span dir="ltr">Resource Limit</span>**

```bash
docker run \
  --cpus="1.5" \
  --memory="512m" \
  nginx
```

<a id="container-management"></a>
## 12. 🎛️ مدیریت <span dir="ltr">Container</span>ها

مشاهده:

```bash
docker ps
docker ps -a
```

<span dir="ltr">Stop</span>:

```bash
docker stop web
```

<span dir="ltr">Start</span>:

```bash
docker start web
```

<span dir="ltr">Restart</span>:

```bash
docker restart web
```

<span dir="ltr">Remove</span>:

```bash
docker rm web
```

<span dir="ltr">Container</span> درحال اجرا را <span dir="ltr">Force Remove</span>:

```bash
docker rm -f web
```

> ⚠️ داده‌ای که فقط داخل <span dir="ltr">Writable Layer</span> خود <span dir="ltr">Container</span> بوده باشد با حذف <span dir="ltr">Container</span> از بین می‌رود. برای دادهٔ مهم از <span dir="ltr">Volume</span> استفاده کن.

<a id="docker-exec"></a>
## 13. 🚪 اجرای دستور داخل <span dir="ltr">Container</span> با `docker exec`

فرض کن <span dir="ltr">Container</span> به نام `web` درحال اجراست.

ورود به <span dir="ltr">Shell</span>:

```bash
docker exec -it web bash
```

اگر <span dir="ltr">Bash</span> نداشت:

```bash
docker exec -it web sh
```

اجرای یک دستور بدون ورود:

```bash
docker exec web ls /usr/share/nginx/html
```

**نکتهٔ مهم**

هر تغییری که با `docker exec` داخل <span dir="ltr">Container</span> بدهی، اگر <span dir="ltr">Container</span> حذف شود ممکن است از بین برود.

برای <span dir="ltr">Configuration</span> دائمی بهتر است از این‌ها استفاده کنی:

- <span dir="ltr">Dockerfile</span>
- <span dir="ltr">Volume</span>
- <span dir="ltr">Bind Mount</span>
- <span dir="ltr">Environment Variable</span>

<a id="docker-cp"></a>
## 14. 📁 کپی فایل با `docker cp`

<span dir="ltr">Host</span> → <span dir="ltr">Container</span>:

```bash
docker cp ./index.html web:/usr/share/nginx/html/index.html
```

<span dir="ltr">Container</span> → <span dir="ltr">Host</span>:

```bash
docker cp web:/etc/nginx/nginx.conf ./nginx.conf
```

حتی <span dir="ltr">Container</span> لازم نیست حتماً <span dir="ltr">Running</span> باشد.

<a id="monitoring"></a>
## 15. 📊 مانیتورینگ با `docker stats` و `docker events`

**<span dir="ltr">Resource Usage</span>**

```bash
docker stats
```

مواردی مثل:

- <span dir="ltr">CPU</span>
- <span dir="ltr">Memory</span>
- <span dir="ltr">Network I/O</span>
- <span dir="ltr">Block I/O</span>

را نمایش می‌دهد.

یک <span dir="ltr">Snapshot</span> بدون <span dir="ltr">Stream</span> دائمی:

```bash
docker stats --no-stream
```

**<span dir="ltr">Docker Events</span>**

```bash
docker events
```

اتفاقات لحظه‌ای <span dir="ltr">Docker Daemon</span> را نشان می‌دهد:

- <span dir="ltr">start</span>
- <span dir="ltr">stop</span>
- <span dir="ltr">die</span>
- <span dir="ltr">create</span>
- <span dir="ltr">destroy</span>
- <span dir="ltr">pull</span>
- <span dir="ltr">push</span>

مثلاً:

```bash
docker events --filter type=container
```

<a id="docker-ports"></a>
## 16. 🌐 <span dir="ltr">Port</span>، <span dir="ltr">EXPOSE</span> و <span dir="ltr">Publish</span>

فرض کن <span dir="ltr">Nginx</span> داخل <span dir="ltr">Container</span> روی <span dir="ltr">Port</span> 80 گوش می‌دهد.

اگر فقط <span dir="ltr">Container</span> را اجرا کنی:

```bash
docker run -d nginx
```

این به معنی قابل دسترس بودن <span dir="ltr">Port 80</span> از <span dir="ltr">Host</span> نیست.

برای <span dir="ltr">Publish</span> کردن:

```bash
docker run -d -p 8080:80 nginx
```

قانون:

```
HOST_PORT:CONTAINER_PORT
```

پس:

```
localhost:8080
      │
      ▼
container:80
```

چند <span dir="ltr">Port</span>:

```bash
docker run -d \
  -p 8080:80 \
  -p 8443:443 \
  nginx
```

<span dir="ltr">Bind</span> به <span dir="ltr">Interface</span> مشخص:

```bash
docker run -d \
  -p 127.0.0.1:8080:80 \
  nginx
```

این روش برای جلوگیری از <span dir="ltr">Public</span> شدن ناخواستهٔ <span dir="ltr">Service</span> بسیار مهم است.

**<span dir="ltr">EXPOSE</span> چیست؟**

در <span dir="ltr">Dockerfile</span>:

```
EXPOSE 80
```

این بیشتر <span dir="ltr">Documentation/Metadata</span> است.

خودش <span dir="ltr">Port</span> را روی <span dir="ltr">Host Publish</span> نمی‌کند.

برای <span dir="ltr">Publish</span> معمولاً باید از `-p` استفاده کنی.

<a id="environment-variable"></a>
## 17. 🌱 <span dir="ltr">Environment Variable</span>

مثال:

```bash
docker run \
  -e APP_ENV=production \
  -e PORT=3000 \
  myapp
```

داخل <span dir="ltr">Container</span>:

```bash
echo $APP_ENV
```

برای <span dir="ltr">Database Image</span>ها بسیار رایج است.

مثال <span dir="ltr">MySQL</span>:

```bash
docker run -d \
  --name mysql \
  -e MYSQL_ROOT_PASSWORD='strong-password' \
  mysql
```

> ⚠️ <span dir="ltr">Secret</span>های واقعی را مستقیم داخل <span dir="ltr">Git Repository</span> یا <span dir="ltr">Dockerfile Hard-code</span> نکن.

<a id="image-layers"></a>
## 18. 🧅 <span dir="ltr">Docker Image</span> و <span dir="ltr">Layer</span>ها

<span dir="ltr">Docker Image</span> از <span dir="ltr">Layer</span>ها ساخته می‌شود.

مثلاً:

```dockerfile
FROM ubuntu:24.04
RUN apt update
RUN apt install -y nginx
COPY . /app
```

به شکل ذهنی:

```
Layer 4 → COPY . /app
Layer 3 → install nginx
Layer 2 → apt update
Layer 1 → ubuntu:24.04
```

<span dir="ltr">Layer</span>های <span dir="ltr">Image</span> معمولاً <span dir="ltr">Immutable</span> هستند.

وقتی <span dir="ltr">Container</span> ساخته می‌شود، یک <span dir="ltr">Writable Layer</span> روی آن‌ها قرار می‌گیرد:

```
Container Writable Layer
────────────────────────
Image Layer 4
Image Layer 3
Image Layer 2
Image Layer 1
```

دیدن <span dir="ltr">History</span>:

```bash
docker history nginx
```

اطلاعات کامل:

```bash
docker inspect nginx
```

<a id="docker-commit"></a>
## 19. 📸 ساخت <span dir="ltr">Image</span> با `docker commit`

فرض کن داخل <span dir="ltr">Container</span> چیزی نصب کرده‌ای:

```bash
docker run -it --name test ubuntu bash
```

داخلش:

```bash
apt update
apt install -y curl
```

حالا:

```bash
docker commit test my-ubuntu-with-curl:v1
```

یک <span dir="ltr">Image</span> جدید ساخته می‌شود.

ولی... برای <span dir="ltr">Production</span> این روش معمولاً توصیه نمی‌شود.

چرا؟ چون دقیق نمی‌دانی <span dir="ltr">Image</span> چطور ساخته شده است.

بهتر:

```
Dockerfile
+
Git
=
Build قابل تکرار
```

<a id="dockerfile"></a>
## 20. 📝 <span dir="ltr">Dockerfile</span> چیست؟

<span dir="ltr">Dockerfile</span> یک فایل متنی است که <span dir="ltr">Recipe</span> ساخت <span dir="ltr">Image</span> را مشخص می‌کند.

مثال:

```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY . .

CMD ["python", "app.py"]
```

<span dir="ltr">Build</span>:

```bash
docker build -t my-python-app:v1 .
```

<span dir="ltr">Run</span>:

```bash
docker run --rm my-python-app:v1
```

<a id="dockerfile-commands"></a>
## 21. 🧰 دستورهای مهم <span dir="ltr">Dockerfile</span>

**FROM** — <span dir="ltr">Base Image</span>:

```dockerfile
FROM ubuntu:24.04
```

**RUN** — دستوری که هنگام <span dir="ltr">Build</span> اجرا می‌شود:

```dockerfile
RUN apt-get update && apt-get install -y curl
```

**WORKDIR** — مسیر کاری:

```dockerfile
WORKDIR /app
```

بعد از آن دستورهای بعدی نسبت به `/app` اجرا می‌شوند.

**COPY** — کپی فایل:

```dockerfile
COPY . .
```

**ENV** — <span dir="ltr">Environment Variable</span>:

```dockerfile
ENV APP_ENV=production
```

**EXPOSE** — اعلام <span dir="ltr">Port</span> مورد انتظار:

```dockerfile
EXPOSE 3000
```

**CMD** — <span dir="ltr">Default Command</span>:

```dockerfile
CMD ["python", "app.py"]
```

**ENTRYPOINT** — برنامهٔ اصلی <span dir="ltr">Container</span>:

```dockerfile
ENTRYPOINT ["python"]
```

**USER** — برای جلوگیری از اجرای برنامه با <span dir="ltr">Root</span>:

```dockerfile
USER appuser
```

**HEALTHCHECK** — بررسی سلامت:

```dockerfile
HEALTHCHECK \
  --interval=30s \
  --timeout=3s \
  CMD curl -f http://localhost:8080/ || exit 1
```

**LABEL** — <span dir="ltr">Metadata</span>:

```dockerfile
LABEL org.opencontainers.image.title="my-app"
```

<a id="cmd-vs-entrypoint"></a>
## 22. 🥊 <span dir="ltr">CMD</span> در برابر <span dir="ltr">ENTRYPOINT</span>

مثال:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

<span dir="ltr">Default</span>:

```bash
docker run myapp
```

تقریباً اجرا می‌شود:

```
python app.py
```

ولی اگر بنویسی:

```bash
docker run myapp test.py
```

می‌شود:

```
python test.py
```

مدل ذهنی:

```
ENTRYPOINT = برنامهٔ اصلی
CMD        = آرگومان یا Default Command
```

<a id="copy-vs-add"></a>
## 23. 📂 <span dir="ltr">COPY</span> در برابر <span dir="ltr">ADD</span>

**<span dir="ltr">COPY</span>**

```dockerfile
COPY ./src /app
```

واضح، ساده و قابل پیش‌بینی. برای اکثر کارها همین را استفاده کن.

**<span dir="ltr">ADD</span>**

قابلیت‌های اضافه دارد؛ مثلاً بعضی <span dir="ltr">Archive</span>های <span dir="ltr">Local</span> را <span dir="ltr">Extract</span> می‌کند.

```dockerfile
ADD archive.tar.gz /app
```

قاعدهٔ ساده: اگر فقط می‌خواهی فایل کپی کنی → `COPY`

<a id="docker-build"></a>
## 24. 🏗️ <span dir="ltr">Build</span> کردن <span dir="ltr">Image</span>

<span dir="ltr">Dockerfile</span>:

```dockerfile
FROM nginx:alpine

COPY ./html /usr/share/nginx/html
```

<span dir="ltr">Build</span>:

```bash
docker build -t mysite:v1 .
```

معنی دستور:

```
docker build
-t mysite:v1     → نام و Tag
.                → Build Context
```

مشاهده <span dir="ltr">Image</span>ها:

```bash
docker image ls
```

یا:

```bash
docker images
```

<a id="build-cache"></a>
## 25. ⚡ <span dir="ltr">Docker Build Cache</span>

<span dir="ltr">Docker</span> تلاش می‌کند <span dir="ltr">Layer</span>های قبلی را <span dir="ltr">Reuse</span> کند.

مثلاً در <span dir="ltr">Node.js</span>:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

CMD ["npm", "start"]
```

چرا اول `package.json` را <span dir="ltr">Copy</span> کردیم؟

چون اگر فقط <span dir="ltr">Source Code</span> عوض شود ولی <span dir="ltr">Dependency</span>ها تغییر نکنند، <span dir="ltr">Layer</span> مربوط به `npm ci` دوباره استفاده می‌شود.

<span dir="ltr">Build</span> سریع‌تر می‌شود.

<a id="docker-tag"></a>
## 26. 🏷️ <span dir="ltr">Tag</span> و <span dir="ltr">Versioning</span>

مثال:

```
myapp:v1
myapp:v1.1
myapp:v2
myapp:latest
```

<span dir="ltr">Tag</span> جدید:

```bash
docker tag myapp:v1 myapp:v1.1
```

برای <span dir="ltr">Docker Hub</span>:

```bash
docker tag myapp:v1 username/myapp:v1
```

چرا فقط `latest` بد است؟

چون نمی‌دانی دقیقاً کدام نسخه <span dir="ltr">Deploy</span> شده.

در <span dir="ltr">Production</span> ترجیحاً <span dir="ltr">Version</span> مشخص داشته باش:

```
myapp:1.4.2
```

<a id="docker-hub"></a>
## 27. 🌍 <span dir="ltr">Docker Registry</span> و <span dir="ltr">Docker Hub</span>

<span dir="ltr">Registry</span> جایی برای نگهداری <span dir="ltr">Image</span>هاست.

<span dir="ltr">Docker Hub</span> یکی از معروف‌ترین <span dir="ltr">Registry</span>هاست.

<span dir="ltr">Flow</span>:

```
Developer
   │
docker push
   ▼
Registry
   │
docker pull
   ▼
Server
```

<span dir="ltr">Pull</span>:

```bash
docker pull nginx
```

<span dir="ltr">Version</span> مشخص:

```bash
docker pull nginx:1.27
```

<a id="docker-push"></a>
## 28. ☁️ <span dir="ltr">Push</span> کردن <span dir="ltr">Image</span> به <span dir="ltr">Docker Hub</span>

<span dir="ltr">Login</span>:

```bash
docker login
```

از قرار دادن <span dir="ltr">Password</span> در <span dir="ltr">Command History</span> با `-p PASSWORD` خودداری کن.

<span dir="ltr">Tag</span>:

```bash
docker tag myapp:v1 username/myapp:v1
```

<span dir="ltr">Push</span>:

```bash
docker push username/myapp:v1
```

<span dir="ltr">Pull</span> روی <span dir="ltr">Server</span> دیگر:

```bash
docker pull username/myapp:v1
```

<a id="save-export"></a>
## 29. 💾 `docker save/load` در برابر `export/import`

این قسمت خیلی‌ها را گیج می‌کند.

**<span dir="ltr">Image</span>**

برای <span dir="ltr">Image</span>:

```bash
docker save
docker load
```

مثال:

```bash
docker save -o myapp.tar myapp:v1
```

<span dir="ltr">Load</span>:

```bash
docker load -i myapp.tar
```

این روش <span dir="ltr">Metadata</span> و <span dir="ltr">Layer</span>های <span dir="ltr">Image</span> را نگه می‌دارد.

**<span dir="ltr">Container Filesystem</span>**

برای <span dir="ltr">Container</span>:

```bash
docker export
docker import
```

<span dir="ltr">Export</span>:

```bash
docker export mycontainer > container.tar
```

<span dir="ltr">Import</span>:

```bash
docker import container.tar myimage:v1
```

اما اطلاعاتی مثل <span dir="ltr">Image History</span> و بعضی <span dir="ltr">Metadata</span>های <span dir="ltr">Container</span> حفظ نمی‌شوند.

**جدول طلایی**

| کار | دستور |
|---|---|
| <span dir="ltr">Backup/Transfer Image</span> | `docker save` |
| <span dir="ltr">Restore Image</span> | `docker load` |
| <span dir="ltr">Export filesystem</span> یک <span dir="ltr">Container</span> | `docker export` |
| تبدیل <span dir="ltr">filesystem</span> به <span dir="ltr">Image</span> | `docker import` |

<a id="docker-storage"></a>
## 30. 💽 <span dir="ltr">Storage</span> در <span dir="ltr">Docker</span>

سه روش اصلی برای <span dir="ltr">Mount</span> داده:

```
Docker Storage
├── Volume
├── Bind Mount
└── tmpfs
```

<a id="docker-volume"></a>
## 31. 📦 <span dir="ltr">Volume</span> چیست؟

<span dir="ltr">Volume</span> توسط <span dir="ltr">Docker</span> مدیریت می‌شود.

ساخت:

```bash
docker volume create app-data
```

مشاهده:

```bash
docker volume ls
```

<span dir="ltr">Inspect</span>:

```bash
docker volume inspect app-data
```

استفاده:

```bash
docker run -d \
  --name web \
  -v app-data:/data \
  nginx
```

روی <span dir="ltr">Linux</span> معمولاً <span dir="ltr">Volume</span>های <span dir="ltr">Local</span> در فضایی زیر این مسیر مدیریت می‌شوند:

```
/var/lib/docker/volumes/
```

بهتر است برنامه‌های عادی مستقیم داخل <span dir="ltr">Directory</span>های داخلی <span dir="ltr">Docker</span> دستکاری نکنند.

**چرا <span dir="ltr">Volume</span> مهم است؟**

<span dir="ltr">Container</span> ممکن است حذف شود:

```bash
docker rm -f web
```

اما <span dir="ltr">Volume</span> مستقل می‌تواند باقی بماند.

پس برای موارد زیر بسیار مهم است:

- <span dir="ltr">Database</span>
- <span dir="ltr">Upload</span>
- <span dir="ltr">Persistent Data</span>

<a id="bind-mount"></a>
## 32. 🔗 <span dir="ltr">Bind Mount</span> چیست؟

در <span dir="ltr">Bind Mount</span> یک <span dir="ltr">Path</span> واقعی <span dir="ltr">Host</span> را وارد <span dir="ltr">Container</span> می‌کنی.

مثال:

```bash
docker run -d \
  --name web \
  -v /home/user/project:/app \
  myapp
```

یعنی:

```
Host
/home/user/project
        │
        ▼
Container
/app
```

برای <span dir="ltr">Development</span> فوق‌العاده است؛ <span dir="ltr">Source Code</span> را تغییر می‌دهی و <span dir="ltr">Container</span> همان فایل‌ها را می‌بیند.

<span dir="ltr">Read-only</span>:

```bash
docker run -d \
  -v /etc/myapp:/config:ro \
  myapp
```

`ro` یعنی: `read-only`

<a id="tmpfs"></a>
## 33. 🧠 <span dir="ltr">tmpfs</span> چیست؟

داده در <span dir="ltr">RAM Host</span> قرار می‌گیرد و <span dir="ltr">Persistent</span> نیست.

```bash
docker run \
  --tmpfs /tmp/cache \
  myapp
```

با <span dir="ltr">Limit</span>:

```bash
docker run \
  --tmpfs /tmp/cache:size=100M \
  myapp
```

مناسب برای:

- <span dir="ltr">Cache</span> موقت
- فایل‌های <span dir="ltr">Temporary</span>
- بعضی داده‌های حساس کوتاه‌عمر

با <span dir="ltr">Stop</span> شدن <span dir="ltr">Container</span> داده از بین می‌رود.

<a id="volume-vs-bind"></a>
## 34. ⚖️ <span dir="ltr">Volume</span> یا <span dir="ltr">Bind Mount</span>؟

| ویژگی | <span dir="ltr">Volume</span> | <span dir="ltr">Bind Mount</span> |
|---|---|---|
| مدیریت توسط <span dir="ltr">Docker</span> | ✅ | ❌ |
| وابستگی به <span dir="ltr">Path Host</span> | کم | زیاد |
| مناسب <span dir="ltr">Database</span> | ✅ | گاهی |
| مناسب <span dir="ltr">Source Code</span> در <span dir="ltr">Development</span> | خوب | ✅ عالی |
| <span dir="ltr">Portability</span> | بهتر | کمتر |
| کنترل مستقیم روی <span dir="ltr">Host Path</span> | کمتر | ✅ |

قاعدهٔ ساده:

```
Database / Persistent Application Data → Volume
Source Code / Config Local Development → Bind Mount
Temporary RAM Data → tmpfs
```

<a id="docker-network"></a>
## 35. 🌐 <span dir="ltr">Docker Network</span> چیست؟

<span dir="ltr">Container</span>ها برای ارتباط با یکدیگر و بیرون نیاز به <span dir="ltr">Networking</span> دارند.

مشاهده <span dir="ltr">Network</span>ها:

```bash
docker network ls
```

ساخت:

```bash
docker network create app-network
```

<span dir="ltr">Inspect</span>:

```bash
docker network inspect app-network
```

حذف:

```bash
docker network rm app-network
```

<a id="bridge-network"></a>
## 36. 🌉 <span dir="ltr">Bridge Network</span>

رایج‌ترین <span dir="ltr">Network</span> برای <span dir="ltr">Container</span>هایی که روی یک <span dir="ltr">Host</span> هستند.

<span dir="ltr">Docker</span> به صورت پیش‌فرض <span dir="ltr">Network</span>ی به نام `bridge` دارد.

اما معمولاً بهتر است برای <span dir="ltr">Application</span> خودت <span dir="ltr">User-defined Bridge</span> بسازی:

```bash
docker network create app-network
```

<span dir="ltr">Container</span> اول:

```bash
docker run -d \
  --name backend \
  --network app-network \
  my-backend
```

<span dir="ltr">Container</span> دوم:

```bash
docker run -d \
  --name frontend \
  --network app-network \
  my-frontend
```

<a id="custom-network"></a>
## 37. 🔎 <span dir="ltr">User-defined Network</span> و <span dir="ltr">DNS</span> داخلی

یکی از مهم‌ترین قابلیت‌ها: روی <span dir="ltr">User-defined Network</span>، <span dir="ltr">Container</span>ها معمولاً می‌توانند با نام <span dir="ltr">Container</span> یکدیگر را پیدا کنند.

مثلاً:

```
frontend
   │
   │ http://backend:3000
   ▼
backend
```

به جای اینکه <span dir="ltr">IP Container</span> را <span dir="ltr">Hard-code</span> کنی.

این مهم است چون <span dir="ltr">IP Container</span> می‌تواند تغییر کند.

اتصال <span dir="ltr">Container</span> موجود به <span dir="ltr">Network</span>:

```bash
docker network connect app-network web
```

قطع:

```bash
docker network disconnect app-network web
```

<a id="network-types"></a>
## 38. 🗺️ <span dir="ltr">Host</span>، <span dir="ltr">None</span>، <span dir="ltr">Overlay</span> و <span dir="ltr">Macvlan</span>

**<span dir="ltr">Bridge</span>**

مناسب اکثر <span dir="ltr">Application</span>های <span dir="ltr">Single Host</span>.

**<span dir="ltr">Host</span>**

```bash
docker run --network host nginx
```

<span dir="ltr">Container Network Namespace</span> مستقل معمول را کنار می‌گذارد و مستقیماً از <span dir="ltr">Network Stack Host</span> استفاده می‌کند.

<span dir="ltr">Isolation</span> کمتر می‌شود.

**<span dir="ltr">None</span>**

```bash
docker run --network none ubuntu
```

برای <span dir="ltr">Container</span> بدون <span dir="ltr">Network</span>.

**<span dir="ltr">Overlay</span>**

برای ارتباط <span dir="ltr">Network</span> بین چند <span dir="ltr">Docker Host</span>؛ بیشتر در <span dir="ltr">Orchestration</span>هایی مثل <span dir="ltr">Docker Swarm</span> مطرح است.

**<span dir="ltr">Macvlan</span>**

<span dir="ltr">Container</span> می‌تواند مانند یک <span dir="ltr">Device</span> مجزا در <span dir="ltr">Network</span> فیزیکی ظاهر شود.

<span dir="ltr">Use Case</span> تخصصی‌تر دارد.

<a id="network-publish"></a>
## 39. 🚪 <span dir="ltr">Port Publishing</span> در شبکه

<span dir="ltr">Container</span>ها در <span dir="ltr">Network</span> داخلی می‌توانند بدون <span dir="ltr">Publish</span> کردن <span dir="ltr">Port</span> با هم ارتباط داشته باشند.

<span dir="ltr">Publish</span> زمانی لازم است که بخواهی <span dir="ltr">Service</span> از بیرون <span dir="ltr">Docker Host</span> قابل دسترس باشد.

مثال:

```bash
docker run -d \
  --name nginx \
  -p 8080:80 \
  nginx
```

```
Browser
  │
  │ localhost:8080
  ▼
Host:8080
  │
  ▼
Container:80
```

> ⚠️ <span dir="ltr">Publish</span> کردن <span dir="ltr">Port</span> روی همه <span dir="ltr">Interface</span>ها ممکن است <span dir="ltr">Service</span> را روی <span dir="ltr">Network</span> یا <span dir="ltr">Internet</span> قابل دسترس کند. <span dir="ltr">Binding</span> و <span dir="ltr">Firewall</span> را آگاهانه تنظیم کن.

<a id="docker-compose"></a>
## 40. 🧩 <span dir="ltr">Docker Compose</span> چیست؟

وقتی <span dir="ltr">Application</span> فقط یک <span dir="ltr">Container</span> نیست چه؟

مثلاً:

```
Application
├── Frontend
├── Backend
├── PostgreSQL
└── Redis
```

نوشتن چهار دستور `docker run` طولانی و مدیریت <span dir="ltr">Network/Volume</span> سخت می‌شود.

<span dir="ltr">Docker Compose Configuration</span> را در <span dir="ltr">YAML</span> تعریف می‌کند.

مثال:

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"

  redis:
    image: redis:alpine
```

اجرا:

```bash
docker compose up -d
```

مشاهده:

```bash
docker compose ps
```

<span dir="ltr">Log</span>:

```bash
docker compose logs -f
```

<span dir="ltr">Stop</span> و حذف <span dir="ltr">Container</span>های <span dir="ltr">Compose</span>:

```bash
docker compose down
```

**نکته**

در <span dir="ltr">Docker</span>های جدید، شکل اصلی دستور `docker compose` است.

فرم قدیمی `docker-compose` <span dir="ltr">Standalone/Legacy</span> محسوب می‌شود.

<a id="docker-prune"></a>
## 41. 🧹 پاک‌سازی <span dir="ltr">Docker</span> و `prune`

<span dir="ltr">Docker</span> با گذشت زمان ممکن است این موارد را جمع کند:

- <span dir="ltr">Image</span>
- <span dir="ltr">Container</span> متوقف‌شده
- <span dir="ltr">Network</span>
- <span dir="ltr">Build Cache</span>
- <span dir="ltr">Volume</span>

بررسی مصرف:

```bash
docker system df
```

<span dir="ltr">Container</span>های متوقف‌شده:

```bash
docker container prune
```

<span dir="ltr">Image</span>های بلااستفاده:

```bash
docker image prune
```

<span dir="ltr">Aggressive</span>تر:

```bash
docker image prune -a
```

<span dir="ltr">Network</span>:

```bash
docker network prune
```

<span dir="ltr">Volume</span>:

```bash
docker volume prune
```

<span dir="ltr">System</span>:

```bash
docker system prune
```

<span dir="ltr">Aggressive</span>:

```bash
docker system prune -a
```

خطرناک‌تر:

```bash
docker system prune -a --volumes
```

> 🚨 **هشدار جدی:** دستورهای `prune`، مخصوصاً با `-a` و `--volumes`، می‌توانند <span dir="ltr">Resource</span>هایی را پاک کنند که بعداً لازم داری. قبل از اجرای آن‌ها بدان دقیقاً چه چیزی حذف می‌شود.

هیچ‌وقت فقط چون یک <span dir="ltr">Tutorial</span> نوشته:

```bash
docker system prune -af --volumes
```

کورکورانه اجرا نکن!

<a id="docker-inspect"></a>
## 42. 🔬 `docker inspect`

یکی از بهترین ابزارهای <span dir="ltr">Debug</span>.

<span dir="ltr">Container</span>:

```bash
docker inspect web
```

<span dir="ltr">Image</span>:

```bash
docker inspect nginx
```

<span dir="ltr">Network</span>:

```bash
docker network inspect app-network
```

<span dir="ltr">Volume</span>:

```bash
docker volume inspect app-data
```

فقط <span dir="ltr">Mount</span>ها:

```bash
docker inspect \
  --format '{{ json .Mounts }}' \
  web
```

<span dir="ltr">Environment Variable</span>ها:

```bash
docker inspect \
  --format '{{ range .Config.Env }}{{ println . }}{{ end }}' \
  web
```

<span dir="ltr">Inspect</span> معمولاً خروجی <span dir="ltr">JSON</span> می‌دهد و برای <span dir="ltr">Troubleshooting</span> بسیار ارزشمند است.

<a id="docker-cheatsheet"></a>
## 43. 🧾 دستورهای طلایی <span dir="ltr">Docker</span>

**اطلاعات**

```bash
docker version
docker info
```

**<span dir="ltr">Images</span>**

```bash
docker image ls
docker pull nginx
docker rmi nginx
docker history nginx
```

**<span dir="ltr">Containers</span>**

```bash
docker ps
docker ps -a

docker run nginx
docker start NAME
docker stop NAME
docker restart NAME
docker rm NAME
```

**<span dir="ltr">Logs</span>**

```bash
docker logs NAME
docker logs -f NAME
```

**<span dir="ltr">Exec</span>**

```bash
docker exec -it NAME bash
```

**<span dir="ltr">Copy</span>**

```bash
docker cp SOURCE DESTINATION
```

**<span dir="ltr">Network</span>**

```bash
docker network ls
docker network create NAME
docker network inspect NAME
docker network rm NAME
```

**<span dir="ltr">Volume</span>**

```bash
docker volume ls
docker volume create NAME
docker volume inspect NAME
docker volume rm NAME
```

**<span dir="ltr">Build</span>**

```bash
docker build -t app:v1 .
```

**<span dir="ltr">Compose</span>**

```bash
docker compose up -d
docker compose ps
docker compose logs -f
docker compose down
```

**<span dir="ltr">Storage Usage</span>**

```bash
docker system df
```

<a id="common-mistakes"></a>
## 44. 🤦 اشتباهات رایج مبتدی‌ها

**اشتباه 1 — <span dir="ltr">Image</span> و <span dir="ltr">Container</span> را یکی بدانیم**

غلط:

```
Image = Container
```

درست:

```
Image → Template
Container → Running/Created Instance
```

**اشتباه 2 — فکر کنیم `EXPOSE 80` یعنی <span dir="ltr">Port Public</span> شده**

نه. برای <span dir="ltr">Publish</span> کردن `-p 8080:80` لازم است.

**اشتباه 3 — <span dir="ltr">Database</span> را بدون <span dir="ltr">Volume</span> اجرا کنیم**

مثلاً:

```bash
docker run postgres
```

بعد <span dir="ltr">Container</span> را حذف کنیم و انتظار داشته باشیم <span dir="ltr">Data</span> مهم باقی بماند.

برای <span dir="ltr">Database</span>، <span dir="ltr">Persistent Storage</span> را جدی بگیر.

**اشتباه 4 — <span dir="ltr">IP Container</span> را <span dir="ltr">Hard-code</span> کنیم**

بهتر است روی <span dir="ltr">User-defined Network</span> از <span dir="ltr">Service/Container Name</span> استفاده کنی.

**اشتباه 5 — همیشه `latest`**

برای <span dir="ltr">Production</span>، <span dir="ltr">Version</span> مشخص بهتر است.

**اشتباه 6 — <span dir="ltr">Secret</span> داخل <span dir="ltr">Dockerfile</span>**

بد:

```dockerfile
ENV DB_PASSWORD=my-super-secret-password
```

این <span dir="ltr">Secret</span> ممکن است داخل <span dir="ltr">Image Metadata/History</span> یا <span dir="ltr">Repository</span> لو برود.

**اشتباه 7 — اجرای کورکورانهٔ `prune`**

خصوصاً:

```bash
docker system prune -a --volumes
```

**اشتباه 8 — همه‌چیز را با <span dir="ltr">Root</span> اجرا کنیم**

تا جای ممکن <span dir="ltr">Application</span> داخل <span dir="ltr">Container</span> را با <span dir="ltr">User</span> محدود اجرا کن.

**اشتباه 9 — <span dir="ltr">Container</span> را <span dir="ltr">VM</span> کوچک تصور کنیم**

<span dir="ltr">Container</span> برای اجرای <span dir="ltr">Process/Application</span> طراحی شده، نه اینکه الزاماً مثل یک <span dir="ltr">Server</span> سنتی پر از <span dir="ltr">Service</span>های مختلف باشد.

<a id="practice"></a>
## 45. 🧪 تمرین عملی از صفر

حالا یک <span dir="ltr">Mini Project</span> کامل.

**مرحله 1 — ساخت <span dir="ltr">Directory</span>**

```bash
mkdir docker-beginner-demo
cd docker-beginner-demo
```

**مرحله 2 — ساخت <span dir="ltr">HTML</span>**

```bash
cat > index.html <<'EOF'
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>Docker Demo</title>
</head>
<body>
  <h1>Hello Docker 🐳</h1>
</body>
</html>
EOF
```

**مرحله 3 — <span dir="ltr">Dockerfile</span>**

```bash
cat > Dockerfile <<'EOF'
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
EOF
```

**مرحله 4 — <span dir="ltr">Build</span>**

```bash
docker build -t beginner-site:v1 .
```

**مرحله 5 — مشاهده <span dir="ltr">Image</span>**

```bash
docker image ls
```

**مرحله 6 — <span dir="ltr">Run</span>**

```bash
docker run -d \
  --name beginner-web \
  -p 8080:80 \
  beginner-site:v1
```

**مرحله 7 — بررسی**

<span dir="ltr">Browser</span>:

```
http://localhost:8080
```

**مرحله 8 — <span dir="ltr">Logs</span>**

```bash
docker logs beginner-web
```

**مرحله 9 — <span dir="ltr">Inspect</span>**

```bash
docker inspect beginner-web
```

**مرحله 10 — <span dir="ltr">Stop</span>**

```bash
docker stop beginner-web
```

**مرحله 11 — <span dir="ltr">Start</span>**

```bash
docker start beginner-web
```

**مرحله 12 — <span dir="ltr">Remove</span>**

```bash
docker rm -f beginner-web
```

<span dir="ltr">Image</span> همچنان وجود دارد:

```bash
docker image ls
```

این تمرین تفاوت <span dir="ltr">Image</span> و <span dir="ltr">Container</span> را عملاً نشان می‌دهد.

<a id="roadmap"></a>
## 46. 🗺️ نقشهٔ راه بعد از این راهنما

اگر تا اینجا را فهمیدی، پیشنهاد ترتیب ادامه:

```
1. Docker CLI
        ↓
2. Image / Container
        ↓
3. Dockerfile
        ↓
4. Volume
        ↓
5. Network
        ↓
6. Docker Compose
        ↓
7. Multi-stage Build
        ↓
8. Security
        ↓
9. Registry / CI-CD
        ↓
10. Docker در Production
        ↓
11. Kubernetes
```

**موضوعات مرحلهٔ بعد**

- `.dockerignore`
- <span dir="ltr">Multi-stage Builds</span>
- <span dir="ltr">BuildKit</span>
- <span dir="ltr">Healthcheck</span> حرفه‌ای
- <span dir="ltr">Restart Policy</span>
- <span dir="ltr">Docker Compose Networks</span>
- <span dir="ltr">Docker Compose Volumes</span>
- <span dir="ltr">Secrets</span>
- <span dir="ltr">Rootless Docker</span>
- <span dir="ltr">Image Scanning</span>
- <span dir="ltr">Logging Driver</span>
- <span dir="ltr">Resource Limit</span>
- <span dir="ltr">Reverse Proxy</span>
- <span dir="ltr">Private Registry</span>
- <span dir="ltr">CI/CD</span>
- <span dir="ltr">Kubernetes</span>

### 🧠 خلاصهٔ نهایی در 60 ثانیه

اگر فقط چند مفهوم را یادت بماند:

```
Docker Engine
    │
    ├── Image       → قالب برنامه
    │
    ├── Container   → Instance ساخته‌شده از Image
    │
    ├── Volume      → دادهٔ Persistent
    │
    └── Network     → ارتباط Containerها
```

و:

```
Dockerfile
   │
   ▼
docker build
   │
   ▼
Image
   │
   ▼
docker run
   │
   ▼
Container
```

و برای چند <span dir="ltr">Service</span>:

```
compose.yaml
    │
    ▼
docker compose up
    │
    ├── frontend
    ├── backend
    ├── database
    └── redis
```

اگر این سه تصویر ذهنی را خوب بفهمی، بخش بزرگی از سردرگمی اولیهٔ <span dir="ltr">Docker</span> حل می‌شود.

<a id="sources"></a>
## 47. 📖 منابع

این راهنما با بازنویسی، یکپارچه‌سازی و ساده‌سازی مطالب فایل‌های آموزشی زیر تهیه شده است:

```
01-docker_engine.md
02-Install_docker.md
03-docker_container.md
04-docker_images.md
05-docker_file.md
06-push_to_dockerhub.md
07-docker_volume.md
08-docker_network.md
```

برای بخش‌هایی که اطلاعات منبع قدیمی شده بود، ساختار با مستندات رسمی جدید <span dir="ltr">Docker</span> تطبیق داده شد.

**مستندات رسمی**

- <span dir="ltr">Docker Engine</span>: https://docs.docker.com/engine/
- <span dir="ltr">Install Docker Engine</span>: https://docs.docker.com/engine/install/
- <span dir="ltr">Docker Compose</span>: https://docs.docker.com/compose/
- <span dir="ltr">Dockerfile Reference</span>: https://docs.docker.com/reference/dockerfile/
- <span dir="ltr">Docker Storage</span>: https://docs.docker.com/engine/storage/
- <span dir="ltr">Docker Networking</span>: https://docs.docker.com/engine/network/

## ❤️ مشارکت

اگر این فایل را در <span dir="ltr">GitHub</span> قرار دادی، می‌توانی <span dir="ltr">Pull Request</span> و <span dir="ltr">Issue</span> را برای موارد زیر باز بگذاری:

- اصلاح غلط تایپی
- مثال بهتر
- توضیح ساده‌تر
- اضافه کردن تمرین
- به‌روزرسانی <span dir="ltr">Version</span>های <span dir="ltr">Docker</span>

هدف این <span dir="ltr">README</span> این است که کسی که امروز هیچ چیزی از <span dir="ltr">Docker</span> نمی‌داند، بعد از خواندنش حداقل بتواند با اعتمادبه‌نفس <span dir="ltr">Image</span>، <span dir="ltr">Container</span>، <span dir="ltr">Dockerfile</span>، <span dir="ltr">Volume</span> و <span dir="ltr">Network</span> را از هم تشخیص دهد.

🐳 <span dir="ltr">Happy Dockerizing</span>!

</div>