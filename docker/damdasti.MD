<div dir="rtl" align="right">

<h1>سناریوهای دم‌دستی Docker — تمرین‌های عملی از صفر</h1>

<p>این فایل برای کسی نوشته شده که یک ترمینال <span dir="ltr">Linux</span> جلویش باز کرده، ممکن است هنوز فرق <span dir="ltr">Image</span> و <span dir="ltr">Container</span> را کامل نداند، و می‌خواهد با چند سناریوی کوتاه و واقعی دستش راه بیفتد.</p>

<p>هدف این فایل حفظ کردن دستور نیست. هر سناریو قرار است یک سؤال واقعی را جواب بدهد: «الان دقیقاً چه اتفاقی افتاد؟»، «این فایل کجاست؟»، «چرا تغییرم دیده نشد؟»، «این پورت مال کدام طرف است؟» و «اگر Container را پاک کنم چه چیزی می‌ماند؟».</p>

<p><strong>قاعدهٔ مطالعه:</strong> سناریوها را به‌ترتیب انجام بده. لازم نیست همه را یک روز تمام کنی. اگر یک مرحله برایت مبهم شد، همان‌جا متوقف شو و خروجی دستورها را با متن مقایسه کن.</p>

<p><strong>نکتهٔ امنیتی:</strong> در این درسنامه هیچ <span dir="ltr">IP</span>، رمز عبور، نام دامنه یا اطلاعات واقعی سرور استفاده نشده است. هر جا لازم باشد از مقدارهایی مثل <code dir="ltr">SERVER_IP</code> یا <code dir="ltr">CHANGE_ME</code> استفاده شده است.</p>

<hr>

<h2>فهرست مطالب</h2>

<ul>
  <li><a href="#before-start">قبل از شروع — سرور را آوردی بالا، حالا کجایی؟</a></li>
  <li><a href="#docker-login">ورود به Docker Hub — Login از طریق Terminal و Docker Desktop</a></li>
  <li><a href="#scenario-1">سناریو ۱ — اولین Container و چرخهٔ run / stop / start / exec</a></li>
  <li><a href="#scenario-2">سناریو ۲ — تغییر داخل Container و فهم ماندگاری آن</a></li>
  <li><a href="#scenario-3">سناریو ۳ — commit و tag؛ ساخت Image از یک Container تغییرکرده</a></li>
  <li><a href="#scenario-4">سناریو ۴ — Python روی Host؛ venv، Flask و نصب Package بدون به‌هم‌ریختن سیستم</a></li>
  <li><a href="#scenario-5">سناریو ۵ — Dockerize کردن Flask با Dockerfile</a></li>
  <li><a href="#scenario-6">سناریو ۶ — Node.js + Express؛ Dockerfile، Build و Port Mapping</a></li>
  <li><a href="#scenario-7">سناریو ۷ — COPY در برابر Bind Mount؛ آزمایش Hello / Bye Bye</a></li>
  <li><a href="#scenario-8">سناریو ۸ — docker cp در برابر Bind Mount با یک فایل C</a></li>
  <li><a href="#scenario-9">سناریو ۹ — save/load در برابر export/import</a></li>
  <li><a href="#scenario-10">سناریو ۱۰ — اولین Compose کوچک و قابل فهم</a></li>
  <li><a href="#mistakes">اشتباهات رایج که همین سناریوها جلویشان را می‌گیرند</a></li>
  <li><a href="#final-map">نقشهٔ ذهنی نهایی</a></li>
</ul>

<hr>

<a id="before-start"></a>

<h2>قبل از شروع — سرور را آوردی بالا، حالا کجایی؟</h2>

<p>فرض می‌کنیم یک سرور <span dir="ltr">Ubuntu</span> داری و <span dir="ltr">Docker</span> از قبل روی آن نصب شده است. اولین کار این نیست که فوراً ده دستور <span dir="ltr">Docker</span> بزنی. اول مطمئن شو روی کدام سیستم و در کدام مسیر هستی.</p>

<p>اگر از یک سیستم دیگر با <span dir="ltr">SSH</span> وصل می‌شوی، مقدار <code dir="ltr">SERVER_IP</code> را با آدرس واقعی سرور خودت عوض کن. آدرس واقعی را داخل مستندات عمومی یا Repository قرار نده.</p>

</div>

```bash
ssh USER@SERVER_IP
```

<div dir="rtl" align="right">

<p>بعد از ورود، سه سؤال ساده را جواب بده: «من چه Userی هستم؟»، «کجا هستم؟»، «اینجا چه فایل‌هایی دارم؟».</p>

</div>

```bash
whoami
pwd
ls -la
```

<div dir="rtl" align="right">

<p>حالا وضعیت <span dir="ltr">Docker</span> را بررسی کن.</p>

</div>

```bash
docker --version
docker version
docker ps
docker images
```

<div dir="rtl" align="right">

<p>اگر سرویس <span dir="ltr">Docker</span> نصب است ولی اجرا نمی‌شود، وضعیتش را ببین.</p>

</div>

```bash
sudo systemctl status docker
```

<div dir="rtl" align="right">

<p>اگر لازم بود آن را اجرا کن.</p>

</div>

```bash
sudo systemctl start docker
```

<div dir="rtl" align="right">

<p>در پایان یک تست ساده انجام بده.</p>

</div>

```bash
docker run --rm hello-world
```

<div dir="rtl" align="right">

<p>اگر این مرحله جواب داد، محیط برای سناریوهای بعدی آماده است. اگر دستور <code dir="ltr">docker</code> اصلاً پیدا نشد، این فایل وارد نصب Docker نمی‌شود؛ اول نصب Docker را کامل کن و بعد برگرد.</p>

</div>

```text
Linux Server
    |
    v
Docker CLI
    |
    v
Docker Engine
    |
    v
Ready for Labs
```

<div dir="rtl" align="right">

<hr>

<a id="scenario-1"></a>

<h2>سناریو ۱ — اولین Container و چرخهٔ <span dir="ltr">run / stop / start / exec</span></h2>

<p><strong>هدف:</strong> بفهمی <code dir="ltr">docker run</code>، <code dir="ltr">docker start</code> و <code dir="ltr">docker exec</code> سه کار متفاوت انجام می‌دهند.</p>

<p>یک <span dir="ltr">Ubuntu Container</span> بساز و مستقیم وارد <span dir="ltr">Bash</span> آن شو.</p>

</div>

```bash
docker run -it --name lab-ubuntu ubuntu bash
```

<div dir="rtl" align="right">

<p>اگر Image روی سیستم نباشد، Docker ابتدا آن را Pull می‌کند و سپس Container را می‌سازد. Prompt باید شبیه یک محیط جدا از Host شود.</p>

</div>

```text
root@CONTAINER_ID:/#
```

<div dir="rtl" align="right">

<p>داخل Container چند دستور عادی Linux بزن.</p>

</div>

```bash
pwd
ls -la
cat /etc/os-release
```

<div dir="rtl" align="right">

<p>از Shell خارج شو.</p>

</div>

```bash
exit
```

<div dir="rtl" align="right">

<p>چون Process اصلی این Container همان <span dir="ltr">Bash</span> بود، با خروج از آن Container معمولاً متوقف می‌شود. حالا روی Host وضعیت را ببین.</p>

</div>

```bash
docker ps
docker ps -a
```

<div dir="rtl" align="right">

<p><code dir="ltr">docker ps</code> فقط Containerهای Running را نشان می‌دهد؛ <code dir="ltr">docker ps -a</code> همه را.</p>

<p>همان Container قبلی را دوباره روشن کن. این دستور Container جدید نمی‌سازد.</p>

</div>

```bash
docker start lab-ubuntu
```

<div dir="rtl" align="right">

<p>حالا بدون ورود به Shell، یک دستور داخل Container اجرا کن.</p>

</div>

```bash
docker exec lab-ubuntu ls -la /
```

<div dir="rtl" align="right">

<p>حالا یک Shell تعاملی جدید داخل همان Container باز کن.</p>

</div>

```bash
docker exec -it lab-ubuntu bash
```

<div dir="rtl" align="right">

<p>خارج شو و سپس Container را از روی Host متوقف کن.</p>

</div>

```bash
exit
docker stop lab-ubuntu
```

<div dir="rtl" align="right">

<p>فرق اصلی این سه دستور را همین‌جا تثبیت کن:</p>

<table>
<thead><tr><th>دستور</th><th>کاری که انجام می‌دهد</th></tr></thead>
<tbody>
<tr><td><code dir="ltr">docker run IMAGE</code></td><td>از Image یک Container جدید می‌سازد و آن را اجرا می‌کند.</td></tr>
<tr><td><code dir="ltr">docker start CONTAINER</code></td><td>همان Container موجود را دوباره اجرا می‌کند.</td></tr>
<tr><td><code dir="ltr">docker exec CONTAINER COMMAND</code></td><td>داخل یک Container روشن، Process یا Command جدید اجرا می‌کند.</td></tr>
</tbody>
</table>

</div>

```text
Image
  |
  v
Container Created
  |
  v
Container Running
  |
  v
Container Stopped
  |
  v
Container Started Again
```

<div dir="rtl" align="right">

<hr>

<a id="scenario-2"></a>

<h2>سناریو ۲ — تغییر داخل Container و فهم ماندگاری آن</h2>

<p><strong>هدف:</strong> بفهمی تغییرهایی که داخل Container می‌دهی متعلق به همان Container هستند، نه Image اصلی.</p>

<p>Container قبلی را روشن کن و واردش شو.</p>

</div>

```bash
docker start lab-ubuntu
docker exec -it lab-ubuntu bash
```

<div dir="rtl" align="right">

<p>داخل Container یک فایل بساز.</p>

</div>

```bash
cd /home
printf 'hello from container\n' > note.txt
cat note.txt
```

<div dir="rtl" align="right">

<p>یک ابزار هم نصب کن تا تغییر بزرگ‌تری نسبت به Image اصلی داشته باشیم.</p>

</div>

```bash
apt update
apt install -y curl
which curl
```

<div dir="rtl" align="right">

<p>حالا خارج شو، Container را Stop و دوباره Start کن.</p>

</div>

```bash
exit
docker stop lab-ubuntu
docker start lab-ubuntu
docker exec -it lab-ubuntu bash
```

<div dir="rtl" align="right">

<p>فایل و Package هنوز وجود دارند، چون همان Container قبلی را حذف نکرده‌ای.</p>

</div>

```bash
cat /home/note.txt
which curl
```

<div dir="rtl" align="right">

<p>این نکته را از همین حالا از هم جدا نگه دار:</p>

<ul>
<li><strong>Stop کردن</strong> Container مساوی حذف کردن آن نیست.</li>
<li><strong>Remove کردن</strong> Container یعنی Writable Layer همان Container را از دست می‌دهی، مگر داده را بیرون از آن نگه داشته باشی.</li>
</ul>

<p>فعلاً Container را پاک نکن؛ در سناریوی بعدی از همین وضعیت Snapshot می‌گیریم.</p>

<hr>

<a id="scenario-3"></a>

<h2>سناریو ۳ — <span dir="ltr">commit</span> و <span dir="ltr">tag</span>؛ ساخت Image از یک Container تغییرکرده</h2>

<p><strong>هدف:</strong> با چشم ببینی چطور تغییرهای یک Container می‌توانند وارد Image جدید شوند، و بعد بفهمی چرا برای کار واقعی Dockerfile انتخاب بهتری است.</p>

<p>از Container تغییرکردهٔ قبلی یک Image بساز.</p>

</div>

```bash
docker commit lab-ubuntu lab-ubuntu-tools:1.0.0
```

<div dir="rtl" align="right">

<p>Image جدید را ببین.</p>

</div>

```bash
docker images lab-ubuntu-tools
```

<div dir="rtl" align="right">

<p>برای همان Image یک Tag دیگر بساز.</p>

</div>

```bash
docker tag lab-ubuntu-tools:1.0.0 lab-ubuntu-tools:stable
```

<div dir="rtl" align="right">

<p><code dir="ltr">docker tag</code> Image را دوباره کپی نمی‌کند؛ یک Reference جدید به همان Image می‌سازد.</p>

<p>حالا از Image جدید یک Container تازه بساز.</p>

</div>

```bash
docker run -it --name lab-ubuntu-copy lab-ubuntu-tools:1.0.0 bash
```

<div dir="rtl" align="right">

<p>بررسی کن فایل و <span dir="ltr">curl</span> منتقل شده‌اند.</p>

</div>

```bash
cat /home/note.txt
which curl
```

<div dir="rtl" align="right">

<p>این تمرین برای فهم Snapshot عالی است، ولی مشکل <code dir="ltr">docker commit</code> این است که مراحل ساخت Image در یک فایل خوانا و قابل تکرار ثبت نمی‌شوند.</p>

<p>برای همین در پروژه واقعی معمولاً می‌خواهیم همین منطق در <span dir="ltr">Dockerfile</span> نوشته شود.</p>

</div>

```text
Container
   |
   v
Manual Changes
   |
   v
Docker Commit
   |
   v
New Image
```

<div dir="rtl" align="right">

<hr>

<a id="scenario-4"></a>

<h2>سناریو ۴ — Python روی Host؛ <span dir="ltr">venv</span>، Flask و نصب Package بدون به‌هم‌ریختن سیستم</h2>

<p><strong>هدف:</strong> قبل از Dockerize کردن Python، یک بار محیط Python روی Host را درست و تمیز بسازی تا فرق «محیط پروژه روی Host» و «محیط داخل Image» برایت روشن شود.</p>

<p>یک پوشهٔ پروژه بساز.</p>

</div>

```bash
mkdir -p ~/labs/flask-demo
cd ~/labs/flask-demo
```

<div dir="rtl" align="right">

<p>محیط مجازی بساز.</p>

</div>

```bash
python3 -m venv venv
```

<div dir="rtl" align="right">

<p>اگر این مرحله گفت ماژول <span dir="ltr">venv</span> در دسترس نیست، روی Ubuntu ممکن است لازم باشد Package مربوط را نصب کنی.</p>

</div>

```bash
sudo apt update
sudo apt install -y python3-venv
```

<div dir="rtl" align="right">

<p>محیط را فعال کن.</p>

</div>

```bash
source venv/bin/activate
```

<div dir="rtl" align="right">

<p>اگر فعال شده باشد معمولاً نام <code dir="ltr">venv</code> را کنار Prompt می‌بینی. حالا Flask را فقط داخل همین محیط نصب کن.</p>

</div>

```bash
pip install flask
```

<div dir="rtl" align="right">

<p>نسخه‌ها را بررسی کن.</p>

</div>

```bash
python --version
pip --version
pip show flask
```

<div dir="rtl" align="right">

<p>حالا وابستگی‌های همین محیط را داخل فایل پروژه ثبت کن. این کار پلی است بین <span dir="ltr">venv</span> روی Host و Dockerfile سناریوی بعدی.</p>

</div>

```bash
pip freeze > requirements.txt
cat requirements.txt
```

<div dir="rtl" align="right">

<p>یک فایل <code dir="ltr">app.py</code> بساز.</p>

</div>

```bash
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=4000)
```

<div dir="rtl" align="right">

<p>برنامه را روی Host اجرا کن.</p>

</div>

```bash
python app.py
```

<div dir="rtl" align="right">

<p>در ترمینال دوم همان سرور تست کن.</p>

</div>

```bash
curl http://127.0.0.1:4000
```

<div dir="rtl" align="right">

<p>باید پاسخ زیر را بگیری.</p>

</div>

```bash
Hello World
```

<div dir="rtl" align="right">

<p>برنامه را با <code dir="ltr">Ctrl+C</code> متوقف کن و از محیط مجازی خارج شو.</p>

</div>

```bash
deactivate
```

<div dir="rtl" align="right">

<p><strong>چرا venv مهم بود؟</strong> چون Packageهای پروژه را از Python سیستم جدا می‌کند. این همان عادت خوبی است که بعداً فهم Dockerfile را هم راحت‌تر می‌کند.</p>

<p>یک آزمایش کوتاه: Image خام Ubuntu الزاماً Python ندارد.</p>

</div>

```bash
docker run --rm -it ubuntu bash
```

<div dir="rtl" align="right">

<p>داخل آن:</p>

</div>

```bash
python
```

<div dir="rtl" align="right">

<p>ممکن است ببینی:</p>

</div>

```bash
python: command not found
```

<div dir="rtl" align="right">

<p>اما Image رسمی Python از قبل Python و pip دارد.</p>

</div>

```bash
docker run --rm -it python:3.11 python --version
```

<div dir="rtl" align="right">

<hr>

<a id="scenario-5"></a>

<h2>سناریو ۵ — Dockerize کردن Flask با Dockerfile</h2>

<p><strong>هدف:</strong> همان برنامه Flask را از محیط Host برداری و به یک Image قابل تکرار تبدیل کنی.</p>

<p>در سناریوی قبل با <code dir="ltr">pip freeze &gt; requirements.txt</code> وابستگی‌ها را ثبت کردیم. این یک بهبود عملی روی سناریوی ساده است: Dependencyها دیگر فقط در محیط مجازی یا History ترمینال نیستند و Dockerfile می‌تواند از همان فایل استفاده کند.</p>

<p>ساختار پروژه:</p>

</div>

```bash
flask-demo/
├── app.py
├── requirements.txt
└── Dockerfile
```

<div dir="rtl" align="right">

<p>یک Dockerfile تمیز و قابل تکرار:</p>

</div>

```bash
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 4000

CMD ["python", "app.py"]
```

<div dir="rtl" align="right">

<p>خط به خط:</p>

<ul>
<li><code dir="ltr">FROM</code>: محیط پایه Python را مشخص می‌کند.</li>
<li><code dir="ltr">WORKDIR /app</code>: مسیر کاری داخل Image را مشخص می‌کند.</li>
<li>اول <code dir="ltr">requirements.txt</code> کپی می‌شود تا Layer نصب Dependencyها بتواند بهتر از Build Cache استفاده کند.</li>
<li><code dir="ltr">RUN pip install ...</code>: Packageها هنگام Build داخل Image نصب می‌شوند؛ این با venv روی Host فرق دارد.</li>
<li>بعد فایل برنامه کپی می‌شود.</li>
<li><code dir="ltr">EXPOSE 4000</code> پورت مورد انتظار برنامه را مستند می‌کند؛ خودش Port را روی Host Publish نمی‌کند.</li>
<li><code dir="ltr">CMD</code> Command پیش‌فرض زمان اجرای Container است.</li>
</ul>

<p>Image را Build کن.</p>

</div>

```bash
docker build -t flask-demo:1.0 .
```

<div dir="rtl" align="right">

<p>نقطهٔ آخر یعنی Build Context پوشهٔ فعلی است.</p>

<p>Image را ببین.</p>

</div>

```bash
docker images flask-demo
```

<div dir="rtl" align="right">

<p>Container را اجرا کن. سمت چپ پورت Host و سمت راست پورت Container است.</p>

</div>

```bash
docker run --rm -d --name flask-web -p 4000:4000 flask-demo:1.0
```

<div dir="rtl" align="right">

<p>تست:</p>

</div>

```bash
curl http://127.0.0.1:4000
```

<div dir="rtl" align="right">

<p>Logها:</p>

</div>

```bash
docker logs flask-web
```

<div dir="rtl" align="right">

<p>توقف:</p>

</div>

```bash
docker stop flask-web
```

<div dir="rtl" align="right">

<p><strong>یک نکتهٔ مهم:</strong> تغییر دادن <code dir="ltr">app.py</code> روی Host، Image قبلی را تغییر نمی‌دهد. <code dir="ltr">COPY</code> هنگام Build اتفاق افتاده است. یا باید دوباره Build کنی، یا در Development از Bind Mount استفاده کنی.</p>

</div>

```text
app.py
  |
  v
Dockerfile
  |
  v
Docker Build
  |
  v
Image
  |
  v
Container
  |
  v
Flask :4000
```

<div dir="rtl" align="right">

<hr>

<a id="scenario-6"></a>

<h2>سناریو ۶ — Node.js + Express؛ Dockerfile، Build و Port Mapping</h2>

<p><strong>هدف:</strong> همان مسیر را این بار با Node ببینی و فرق Image خام Node با Image پروژهٔ خودت را لمس کنی.</p>

<p>یک پوشهٔ جدید بساز.</p>

</div>

```bash
mkdir -p ~/labs/node-demo
cd ~/labs/node-demo
```

<div dir="rtl" align="right">

<p>اول Image رسمی Node را بگیر و مطمئن شو Node داخل Docker در دسترس است. در این سناریو عمداً لازم نیست Node را روی Ubuntu Host نصب کنیم.</p>

</div>

```bash
docker pull node:22-slim
docker run --rm node:22-slim node --version
docker run --rm node:22-slim npm --version
```

<div dir="rtl" align="right">

<p>اگر یک Container از Image خام Node باز کنی، Node و npm را داری، ولی فایل‌های پروژهٔ خودت داخلش نیستند.</p>

</div>

```bash
docker run --rm -it node:22-slim bash
```

<div dir="rtl" align="right">

<p>داخل آن اگر دنبال <code dir="ltr">app.js</code> بگردی چیزی پیدا نمی‌کنی؛ چون Image خام Node پروژهٔ شخصی تو را نمی‌شناسد. از Shell خارج شو و فایل‌های پروژه را روی Host بساز.</p>

</div>

```bash
exit
```

<div dir="rtl" align="right">

<p>فایل <code dir="ltr">package.json</code> را به‌صورت ساده بساز. در پروژهٔ واقعی این فایل معمولاً توسط خود پروژه و Package Manager نگهداری می‌شود.</p>

</div>

```bash
{
  "name": "node-demo",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "express": "^5.1.0"
  }
}
```

<div dir="rtl" align="right">

<p>فایل <code dir="ltr">app.js</code>:</p>

</div>

```bash
const express = require("express");

const app = express();

app.get("/", (req, res) => {
  res.send("Hello");
});

app.listen(3000, "0.0.0.0", () => {
  console.log("Listening on port 3000");
});
```

<div dir="rtl" align="right">

<p>Dockerfile:</p>

</div>

```bash
FROM node:22-slim

WORKDIR /app

COPY package*.json ./
RUN npm install --omit=dev

COPY app.js .

EXPOSE 3000

CMD ["node", "app.js"]
```

<div dir="rtl" align="right">

<p>اگر پروژه <code dir="ltr">package-lock.json</code> معتبر دارد، در پروژه‌های واقعی معمولاً <code dir="ltr">npm ci</code> انتخاب تکرارپذیرتری است. برای این آزمایش ساده <code dir="ltr">npm install</code> کافی است.</p>

<p>Build:</p>

</div>

```bash
docker build -t node-demo:1.0 .
```

<div dir="rtl" align="right">

<p>اجرا با Host Port متفاوت:</p>

</div>

```bash
docker run --rm -d --name node-web -p 2000:3000 node-demo:1.0
```

<div dir="rtl" align="right">

<p>تست:</p>

</div>

```bash
curl http://127.0.0.1:2000
```

<div dir="rtl" align="right">

<p>در این دستور:</p>

<table>
<thead><tr><th>سمت</th><th>مقدار</th></tr></thead>
<tbody>
<tr><td>Host</td><td><code dir="ltr">2000</code></td></tr>
<tr><td>Container</td><td><code dir="ltr">3000</code></td></tr>
</tbody>
</table>

<p>Node داخل Container هنوز روی 3000 گوش می‌دهد. Docker فقط مسیر ورودی Host را به آن وصل کرده است.</p>

<p>یک اشتباه آموزشی مهم: اگر آخر <code dir="ltr">docker run</code> بنویسی <code dir="ltr">bash</code>، Command پیش‌فرض Dockerfile را Override می‌کنی.</p>

</div>

```bash
docker run --rm -it node-demo:1.0 bash
```

<div dir="rtl" align="right">

<p>در این حالت برنامه خودکار Start نمی‌شود و اگر می‌خواهی اجرا شود باید خودت داخل Container بگویی:</p>

</div>

```bash
node app.js
```

<div dir="rtl" align="right">

<hr>

<a id="scenario-7"></a>

<h2>سناریو ۷ — <span dir="ltr">COPY</span> در برابر <span dir="ltr">Bind Mount</span>؛ آزمایش Hello / Bye Bye</h2>

<p><strong>هدف:</strong> یکی از مهم‌ترین سوءتفاهم‌های Docker را با چشم ببینی: فایلی که هنگام Build با <code dir="ltr">COPY</code> وارد Image شده، همان فایل زندهٔ Host نیست.</p>

<p>فرض کن Image قبلی را زمانی Build کرده‌ای که پاسخ برنامه <code dir="ltr">Hello</code> بوده است.</p>

<p>حالا روی Host، <code dir="ltr">app.js</code> را تغییر بده و پاسخ را به <code dir="ltr">Bye Bye</code> تبدیل کن. فعلاً دوباره Build نزن.</p>

</div>

```bash
app.get("/", (req, res) => {
  res.send("Bye Bye");
});
```

<div dir="rtl" align="right">

<p>الان دو واقعیت جدا داری:</p>

<table>
<thead><tr><th>محل</th><th>محتوا</th></tr></thead>
<tbody>
<tr><td>Image قبلی</td><td><code dir="ltr">Hello</code></td></tr>
<tr><td>فایل فعلی Host</td><td><code dir="ltr">Bye Bye</code></td></tr>
</tbody>
</table>

<p>حالا کل پوشهٔ Host را در مسیر دیگری از Container Mount کن تا هر دو نسخه را هم‌زمان ببینی.</p>

</div>

```bash
docker run --rm -it -p 2000:3000 \
  -v "$PWD":/home/new_app \
  node-demo:1.0 bash
```

<div dir="rtl" align="right">

<p>نسخه‌ای که داخل Image ساخته شده:</p>

</div>

```bash
cat /app/app.js
```

<div dir="rtl" align="right">

<p>نسخهٔ زندهٔ Host:</p>

</div>

```bash
cat /home/new_app/app.js
```

<div dir="rtl" align="right">

<p>اگر درست پیش رفته باشی، اولی هنوز <code dir="ltr">Hello</code> و دومی <code dir="ltr">Bye Bye</code> است.</p>

<p>نسخهٔ Image را اجرا کن.</p>

</div>

```bash
node /app/app.js
```

<div dir="rtl" align="right">

<p>از ترمینال دیگر:</p>

</div>

```bash
curl http://127.0.0.1:2000
```

<div dir="rtl" align="right">

<p>برنامه را با <code dir="ltr">Ctrl+C</code> متوقف کن. حالا نسخهٔ Mount شده را اجرا کن. چون Dependencyهای Node در <code dir="ltr">/app/node_modules</code> هستند، در همین آزمایش آموزشی مسیرشان را به Node معرفی می‌کنیم.</p>

</div>

```text
NODE_PATH=/app/node_modules node /home/new_app/app.js
```

<div dir="rtl" align="right">

<p>دوباره تست کن.</p>

</div>

```bash
curl http://127.0.0.1:2000
```

<div dir="rtl" align="right">

<p>این بار باید <code dir="ltr">Bye Bye</code> را ببینی.</p>

<p>قانون این سناریو:</p>

<table>
<thead><tr><th>روش</th><th>مدل ذهنی</th></tr></thead>
<tbody>
<tr><td><code dir="ltr">COPY</code></td><td>Snapshot فایل در زمان Build</td></tr>
<tr><td><span dir="ltr">Bind Mount</span></td><td>نمای زنده از فایل یا پوشهٔ Host</td></tr>
</tbody>
</table>

<p>یک دام رایج: اگر کل <code dir="ltr">/app</code> را بی‌فکر Mount کنی، ممکن است فایل‌هایی که قبلاً داخل Image آن مسیر وجود داشتند پشت Mount پنهان شوند؛ مثلاً <code dir="ltr">node_modules</code>.</p>

<hr>

<a id="scenario-8"></a>

<h2>سناریو ۸ — <span dir="ltr">docker cp</span> در برابر <span dir="ltr">Bind Mount</span> با یک فایل C</h2>

<p><strong>هدف:</strong> فرق «یک بار Copy کردن» و «اتصال زنده» را بدون وابستگی به Framework ببینی.</p>

<p>یک پوشهٔ Lab بساز و فایل C را داخلش قرار بده.</p>

</div>

```bash
mkdir -p ~/labs/copy-vs-mount
cd ~/labs/copy-vs-mount
```

<div dir="rtl" align="right">

<p>فایل <code dir="ltr">wilson.c</code>:</p>

</div>

```bash
#include <stdio.h>

int main(void) {
    printf("hello-world\n");
    return 0;
}
```

<div dir="rtl" align="right">

<p>یک Ubuntu Container در Background بساز.</p>

</div>

```bash
docker run -it -d --name copy-lab ubuntu bash
```

<div dir="rtl" align="right">

<p>فایل را فقط یک بار داخل آن Copy کن.</p>

</div>

```bash
docker cp wilson.c copy-lab:/home/wilson.c
```

<div dir="rtl" align="right">

<p>حالا Container دوم را از Image دارای GCC بساز و پوشهٔ فعلی Host را روی <code dir="ltr">/tmp</code> Mount کن.</p>

</div>

```bash
docker run -it -d --name mount-lab -v "$PWD":/tmp gcc bash
```

<div dir="rtl" align="right">

<p>فایل‌ها را در Container دوم ببین.</p>

</div>

```bash
docker exec mount-lab ls -la /tmp
```

<div dir="rtl" align="right">

<p>داخل Container دوم برنامه را Compile کن.</p>

</div>

```bash
docker exec -w /tmp mount-lab gcc -o main wilson.c
```

<div dir="rtl" align="right">

<p>حالا روی Host فایل <code dir="ltr">main</code> را ببین. چون خروجی داخل مسیر Mount شده ساخته شد، روی Host هم ظاهر می‌شود.</p>

</div>

```bash
ls -la
```

<div dir="rtl" align="right">

<p>فایل <code dir="ltr">wilson.c</code> روی Host را تغییر بده؛ مثلاً متن خروجی را عوض کن. بعد دو Container را مقایسه کن.</p>

<p>نسخه‌ای که قبلاً با <code dir="ltr">docker cp</code> کپی شده:</p>

</div>

```bash
docker exec copy-lab cat /home/wilson.c
```

<div dir="rtl" align="right">

<p>نسخه‌ای که از Bind Mount خوانده می‌شود:</p>

</div>

```bash
docker exec mount-lab cat /tmp/wilson.c
```

<div dir="rtl" align="right">

<p>اولی تغییر جدید Host را نمی‌بیند؛ دومی بلافاصله می‌بیند.</p>

<table>
<thead><tr><th>روش</th><th>رفتار</th></tr></thead>
<tbody>
<tr><td><code dir="ltr">docker cp</code></td><td>یک بار فایل را کپی می‌کند؛ بعد از آن دو نسخه مستقل‌اند.</td></tr>
<tr><td><span dir="ltr">Bind Mount</span></td><td>Host و Container همان دادهٔ Mount شده را می‌بینند.</td></tr>
</tbody>
</table>

</div>

```text
Host File
   |
   v
Docker Copy
   |
   v
Independent Container Copy

Host Directory
   |
   v
Bind Mount
   |
   v
Shared View Inside Container
```

<div dir="rtl" align="right">

<hr>

<a id="scenario-9"></a>

<h2>سناریو ۹ — <span dir="ltr">save/load</span> در برابر <span dir="ltr">export/import</span></h2>

<p><strong>هدف:</strong> یک بار برای همیشه بدانی چه زمانی Image را ذخیره می‌کنی و چه زمانی File System یک Container را.</p>

<p>یک Image آماده داشته باش. برای مثال از Image سناریوی commit استفاده می‌کنیم.</p>

</div>

```bash
docker images lab-ubuntu-tools
```

<div dir="rtl" align="right">

<p>Image را به فایل TAR تبدیل کن.</p>

</div>

```bash
docker save lab-ubuntu-tools:1.0.0 -o lab-image.tar
```

<div dir="rtl" align="right">

<p>فایل را ببین.</p>

</div>

```bash
ls -lh lab-image.tar
```

<div dir="rtl" align="right">

<p>همان Archive را دوباره به Docker Load کن.</p>

</div>

```bash
docker load -i lab-image.tar
```

<div dir="rtl" align="right">

<p>این زوج با Image کار می‌کند:</p>

</div>

```text
Image
  |
  v
Save
  |
  v
TAR Archive
  |
  v
Load
  |
  v
Image
```

<div dir="rtl" align="right">

<p>حالا یک Container موجود را Export کن.</p>

</div>

```bash
docker export lab-ubuntu -o lab-container.tar
```

<div dir="rtl" align="right">

<p>از File System Export شده یک Image جدید بساز.</p>

</div>

```bash
docker import lab-container.tar lab-imported:1.0
```

<div dir="rtl" align="right">

<p>این زوج با File System فعلی Container سروکار دارد:</p>

</div>

```text
Container Filesystem
       |
       v
Export
       |
       v
TAR Archive
       |
       v
Import
       |
       v
New Image
```

<div dir="rtl" align="right">

<p>اگر Image Import شده Command پیش‌فرض نداشته باشد، ممکن است اجرای ساده جواب ندهد.</p>

</div>

```bash
docker run -it lab-imported:1.0
```

<div dir="rtl" align="right">

<p>در آن صورت Command را صریح بده.</p>

</div>

```bash
docker run -it lab-imported:1.0 bash
```

<div dir="rtl" align="right">

<table>
<thead><tr><th>زوج دستور</th><th>موضوع اصلی</th></tr></thead>
<tbody>
<tr><td><code dir="ltr">save / load</code></td><td>Image</td></tr>
<tr><td><code dir="ltr">export / import</code></td><td>File System یک Container و ساخت Image جدید از آن</td></tr>
</tbody>
</table>

<hr>

<a id="scenario-10"></a>

<h2>سناریو ۱۰ — اولین Compose کوچک و قابل فهم</h2>

<p><strong>هدف:</strong> بعد از اینکه با <code dir="ltr">docker run</code> راحت شدی، ببینی Compose همان تنظیمات را در یک فایل قابل تکرار جمع می‌کند.</p>

<p>یک پوشه بساز.</p>

</div>

```bash
mkdir -p ~/labs/compose-demo
cd ~/labs/compose-demo
```

<div dir="rtl" align="right">

<p>فایل <code dir="ltr">compose.yaml</code>:</p>

</div>

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

<div dir="rtl" align="right">

<p>قبل از اجرا، فایل Compose را Validate کن.</p>

</div>

```bash
docker compose config
```

<div dir="rtl" align="right">

<p>Stack را اجرا کن.</p>

</div>

```bash
docker compose up -d
```

<div dir="rtl" align="right">

<p>وضعیت:</p>

</div>

```bash
docker compose ps
```

<div dir="rtl" align="right">

<p>تست روی خود سرور:</p>

</div>

```bash
curl http://127.0.0.1:8080
```

<div dir="rtl" align="right">

<p>Logها:</p>

</div>

```bash
docker compose logs -f
```

<div dir="rtl" align="right">

<p>پایان سناریو:</p>

</div>

```bash
docker compose down
```

<div dir="rtl" align="right">

<p>حالا یک قدم واقعی‌تر: Backend و Database معمولاً باید با نام Service همدیگر را پیدا کنند، نه با IP ثابت Container. نمونهٔ زیر فقط برای دیدن شکل معماری است؛ رمز واقعی را داخل Repository نگذار.</p>

<p>فایل <code dir="ltr">.env</code>:</p>

</div>

```text
POSTGRES_PASSWORD=CHANGE_ME
```

<div dir="rtl" align="right">

<p>نمونهٔ Compose:</p>

</div>

```yaml
services:
  backend:
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      DB_HOST: database
      DB_PORT: "5432"

  database:
    image: postgres:17
    environment:
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

<div dir="rtl" align="right">

<p>اینجا Backend به جای یک IP موقت، نام Service یعنی <code dir="ltr">database</code> را استفاده می‌کند. Compose برای پروژه Network داخلی می‌سازد و Serviceها می‌توانند با نام یکدیگر ارتباط بگیرند.</p>

<p><strong>هشدار:</strong> فایل <code dir="ltr">.env</code> واقعی را اگر Secret دارد Commit نکن. این مثال فقط از Placeholder استفاده می‌کند.</p>

<hr>

<a id="mistakes"></a>

<h2>اشتباهات رایج که همین سناریوها جلویشان را می‌گیرند</h2>

<table>
<thead><tr><th>اشتباه</th><th>تصحیح ذهنی</th></tr></thead>
<tbody>
<tr><td>فکر کنم <code dir="ltr">docker pull node</code> یعنی Node روی Host نصب شده</td><td>Image داخل Docker دانلود شده؛ Host لزوماً Node ندارد.</td></tr>
<tr><td>روی Container متوقف‌شده <code dir="ltr">docker exec</code> بزنم</td><td>اول باید Container Running باشد.</td></tr>
<tr><td><code dir="ltr">docker start</code> را با <code dir="ltr">docker run</code> یکی بدانم</td><td>run معمولاً Container جدید می‌سازد؛ start همان Container قبلی را روشن می‌کند.</td></tr>
<tr><td>سمت پورت‌ها را برعکس بخوانم</td><td><code dir="ltr">-p HOST:CONTAINER</code></td></tr>
<tr><td>فکر کنم <code dir="ltr">EXPOSE</code> پورت Host را باز می‌کند</td><td>برای Publish کردن Port هنوز به <code dir="ltr">-p</code> یا تنظیم Compose نیاز داری.</td></tr>
<tr><td>بعد از تغییر فایل Host انتظار داشته باشم Image قبلی تغییر کند</td><td><code dir="ltr">COPY</code> در زمان Build انجام شده است.</td></tr>
<tr><td><code dir="ltr">docker cp</code> را Sync زنده بدانم</td><td>فقط یک Copy است.</td></tr>
<tr><td>کل <code dir="ltr">/app</code> را بی‌فکر Bind Mount کنم</td><td>ممکن است فایل‌های موجود در Image را پشت Mount پنهان کنی.</td></tr>
<tr><td><code dir="ltr">save</code> و <code dir="ltr">export</code> را یکی بدانم</td><td>save برای Image؛ export برای File System Container.</td></tr>
<tr><td>IP داخلی Container را Hard-code کنم</td><td>در Networkهای Docker و Compose تا جای ممکن از نام Service/Container استفاده کن.</td></tr>
</tbody>
</table>

<hr>

<a id="final-map"></a>

<h2>نقشهٔ ذهنی نهایی</h2>

<p>بعد از انجام این سناریوها باید بتوانی این مسیر را بدون حفظ کردن کورکورانه توضیح بدهی:</p>

</div>

```text
Source Code
    |
    v
Dependency Files
    |
    v
Dockerfile
    |
    v
Docker Build
    |
    v
Image
    |
    v
Docker Run
    |
    v
Container
    |
    v
Port Mapping
    |
    v
Application Access

Host File
    |
    +--> COPY during Build --> Image Snapshot
    |
    +--> Bind Mount --------> Live Container View
```

<div dir="rtl" align="right">

<p>و برای مدیریت پروژه:</p>

</div>

```text
Dockerfile
    |
    v
Image
    |
    v
Compose Service
    |
    v
Container
    |
    v
Network + Volume + Environment
```

<div dir="rtl" align="right">

<h3>بعد از این فایل چه چیزی را باید بلد باشی؟</h3>

<ul>
<li>از روی Image یک Container بسازی و چرخهٔ زندگی‌اش را مدیریت کنی.</li>
<li>فرق Host و Container را گم نکنی.</li>
<li>فرق <span dir="ltr">Image</span>، <span dir="ltr">Container</span> و <span dir="ltr">Dockerfile</span> را با مثال توضیح بدهی.</li>
<li>Port Mapping را از روی <code dir="ltr">HOST:CONTAINER</code> بخوانی.</li>
<li>در Python از <span dir="ltr">venv</span> استفاده کنی و بفهمی چرا داخل Dockerfile همان کار به شکل دیگری انجام می‌شود.</li>
<li>فرق <code dir="ltr">COPY</code>، <code dir="ltr">docker cp</code> و <span dir="ltr">Bind Mount</span> را تشخیص بدهی.</li>
<li>یک Dockerfile ساده برای Flask یا Node بسازی.</li>
<li>بدانی <code dir="ltr">docker commit</code> برای یادگیری مفید است ولی Dockerfile مسیر قابل تکرارتری می‌دهد.</li>
<li>فرق <code dir="ltr">save/load</code> و <code dir="ltr">export/import</code> را بدانی.</li>
<li>یک Compose ساده را Validate، اجرا، بررسی و متوقف کنی.</li>
</ul>

<p>اگر این‌ها را واقعاً با ترمینال انجام داده باشی، دیگر Docker برایت مجموعه‌ای از دستورهای تصادفی نیست؛ هر دستور جای مشخصی در یک چرخهٔ قابل فهم دارد.</p>

</div>