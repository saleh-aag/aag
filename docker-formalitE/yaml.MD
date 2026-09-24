<div dir="rtl">

# 🧩 راهنمای جامع <span dir="ltr">YAML</span> برای <span dir="ltr">Docker</span> و <span dir="ltr">DevOps</span>

از «این <span dir="ltr">:</span> و <span dir="ltr">-</span> چی‌ان؟» تا نوشتن فایل‌های واقعی <span dir="ltr">Docker Compose</span>، <span dir="ltr">GitHub Actions</span>، <span dir="ltr">Kubernetes</span> و <span dir="ltr">Ansible</span>

این راهنما برای کسی نوشته شده که ممکن است صفرِ صفر باشد؛ یعنی حتی اگر تا امروز یک فایل <span dir="ltr">.yaml</span> هم ننوشته‌ای، قرار نیست وسط صفحهٔ سوم احساس کنی وارد زبان باستانی موجودات فضایی شده‌ای 😄

> 💡 **نکتهٔ نمایش:** این فایل با `<div dir="rtl">` باز شده تا جهت پایهٔ همهٔ بلوک‌ها <span dir="ltr">RTL</span> باشد. <span dir="ltr">Code Block</span>ها، مسیرها، نام فایل‌ها و دستورها داخل بلوک‌های <span dir="ltr">```</span> قرار گرفته‌اند و به‌طور طبیعی <span dir="ltr">LTR</span> باقی می‌مانند تا ترکیب فارسی و انگلیسی در <span dir="ltr">GitHub</span> و <span dir="ltr">VS Code</span> به‌هم نریزد.

🎯 هدف این راهنما: فقط <span dir="ltr">Syntax</span> حفظ نکنیم؛ بفهمیم <span dir="ltr">YAML</span> چطور فکر می‌کند، چرا <span dir="ltr">Docker</span> و ابزارهای <span dir="ltr">DevOps</span> از آن استفاده می‌کنند، و در سناریوهای واقعی چطور فایل درست بنویسیم و خطاهایش را پیدا کنیم.

<a id="toc"></a>
## 📚 فهرست مطالب

اگر دنبال موضوع خاصی هستی مستقیم بپر همان بخش؛ این فایل قرار نیست مثل جزوهٔ ساعت ۷ صبح دانشگاه خوانده شود 😴

- [🧩 راهنمای جامع YAML برای Docker و DevOps](#-راهنمای-جامع-yaml-برای-docker-و-devops)
  - [📚 فهرست مطالب](#-فهرست-مطالب)
  - [1. 🧩 YAML اصلاً چیست؟](#1--yaml-اصلاً-چیست)
  - [2. 🚨 مهم‌ترین قانون: YAML با Docker فرق دارد](#2--مهمترین-قانون-yaml-با-docker-فرق-دارد)
  - [3. 📄 فایل .yaml یا .yml؟](#3--فایل-yaml-یا-yml)
  - [4. 🧱 سه ساختمان اصلی YAML](#4--سه-ساختمان-اصلی-yaml)
    - [4.1. Scalar — یک مقدار](#41-scalar--یک-مقدار)
    - [4.2. Mapping — کلید و مقدار](#42-mapping--کلید-و-مقدار)
    - [4.3. Sequence — لیست](#43-sequence--لیست)
  - [5. 📐 قانون طلایی Indentation](#5--قانون-طلایی-indentation)
  - [6. 🔑 : دقیقاً چه کاری می‌کند؟](#6---دقیقاً-چه-کاری-میکند)
  - [7. ➖ - دقیقاً چه کاری می‌کند؟](#7----دقیقاً-چه-کاری-میکند)
  - [8. 🪆 ترکیب Mapping و List](#8--ترکیب-mapping-و-list)
  - [9. 💬 Comment نویسی با #](#9--comment-نویسی-با-)
  - [10. 🎭 String، Number، Boolean و Null](#10--string-number-boolean-و-null)
  - [11. 🗣️ چه وقت Quote بگذاریم؟](#11-️-چه-وقت-quote-بگذاریم)
  - [12. 🥊 Single Quote و Double Quote](#12--single-quote-و-double-quote)
  - [13. 📜 متن چندخطی با | و \>](#13--متن-چندخطی-با--و-)
  - [14. 🧺 {} و \[\]؛ Flow Style](#14---و--flow-style)
  - [15. 📚 --- و ... یعنی چه؟](#15------و--یعنی-چه)
  - [16. ⚓ Anchor و Alias با \& و \*](#16--anchor-و-alias-با--و-)
  - [17. 🧬 Merge با \<\<](#17--merge-با-)
  - [18. 💣 خطاهای معروف YAML](#18--خطاهای-معروف-yaml)
  - [19. 🧠 روش ذهنی خواندن هر فایل YAML](#19--روش-ذهنی-خواندن-هر-فایل-yaml)
  - [20. 🐳 YAML در Docker Compose](#20--yaml-در-docker-compose)
  - [21. 🧩 کالبدشکافی services](#21--کالبدشکافی-services)
  - [22. 🏗️ build و context](#22-️-build-و-context)
  - [23. 🖼️ image](#23-️-image)
  - [24. 🚪 ports](#24--ports)
  - [25. 🌱 environment](#25--environment)
  - [26. 🌿 .env و Variable Interpolation](#26--env-و-variable-interpolation)
  - [27. 💽 volumes](#27--volumes)
  - [28. 🌐 networks](#28--networks)
  - [29. ❤️ depends\_on و healthcheck](#29-️-depends_on-و-healthcheck)
  - [30. 🔁 restart](#30--restart)
  - [31. 🧾 command و entrypoint](#31--command-و-entrypoint)
  - [32. 🔐 Secrets؛ Password را کجا نگذاریم؟](#32--secrets-password-را-کجا-نگذاریم)
  - [33. 🧪 سناریو 1 — Nginx تک‌سرویسی](#33--سناریو-1--nginx-تکسرویسی)
  - [34. 🧪 سناریو 2 — Frontend + Backend](#34--سناریو-2--frontend--backend)
  - [35. 🧪 سناریو 3 — Frontend + API + MongoDB](#35--سناریو-3--frontend--api--mongodb)
  - [36. 🧪 سناریو 4 — Database با Volume](#36--سناریو-4--database-با-volume)
  - [37. 🧪 سناریو 5 — شبکهٔ اختصاصی](#37--سناریو-5--شبکهٔ-اختصاصی)
  - [38. 🧪 سناریو 6 — .env برای Development و Production](#38--سناریو-6--env-برای-development-و-production)
  - [39. 🧪 سناریو 7 — Healthcheck واقعی](#39--سناریو-7--healthcheck-واقعی)
  - [40. 🧪 سناریو 8 — Compose چندفایلی و Override](#40--سناریو-8--compose-چندفایلی-و-override)
  - [41. ♻️ Reuse در Compose با x- و Anchor](#41-️-reuse-در-compose-با-x--و-anchor)
  - [42. ✅ تست و Validate کردن Compose](#42--تست-و-validate-کردن-compose)
  - [43. 🤖 YAML در GitHub Actions](#43--yaml-در-github-actions)
  - [44. 🧪 Workflow واقعی Build و Test](#44--workflow-واقعی-build-و-test)
  - [45. 🐳 Workflow ساخت و Push کردن Docker Image](#45--workflow-ساخت-و-push-کردن-docker-image)
  - [46. 🔐 Secrets در GitHub Actions](#46--secrets-در-github-actions)
  - [47. ☸️ YAML در Kubernetes](#47-️-yaml-در-kubernetes)
  - [48. 🚀 Deployment را خط‌به‌خط بخوانیم](#48--deployment-را-خطبهخط-بخوانیم)
  - [49. 🌐 Service را خط‌به‌خط بخوانیم](#49--service-را-خطبهخط-بخوانیم)
  - [50. 📚 چند Resource در یک فایل](#50--چند-resource-در-یک-فایل)
  - [51. 🔧 YAML در Ansible](#51--yaml-در-ansible)
  - [52. 🧪 Playbook سادهٔ واقعی](#52--playbook-سادهٔ-واقعی)
  - [53. 🔁 Variables، List و Loop در Ansible](#53--variables-list-و-loop-در-ansible)
  - [54. 🧠 تفاوت YAML در ابزارهای مختلف DevOps](#54--تفاوت-yaml-در-ابزارهای-مختلف-devops)
  - [55. 🎨 Style Guide پیشنهادی](#55--style-guide-پیشنهادی)
  - [56. 🕵️ Debug کردن خطاهای YAML](#56-️-debug-کردن-خطاهای-yaml)
  - [57. 🛡️ دستورهای Validation که باید حفظ باشی](#57-️-دستورهای-validation-که-باید-حفظ-باشی)
  - [58. 🧪 تمرین‌های مرحله‌ای](#58--تمرینهای-مرحلهای)
  - [59. 🎯 آزمون سریع](#59--آزمون-سریع)
  - [60. 🧾 Cheat Sheet نهایی](#60--cheat-sheet-نهایی)
  - [61. 🗺️ نقشهٔ راه ادامه](#61-️-نقشهٔ-راه-ادامه)
  - [62. 📖 منابع رسمی](#62--منابع-رسمی)
  - [❤️ حرف آخر](#️-حرف-آخر)

---

<a id="yaml-what"></a>
## 1. 🧩 <span dir="ltr">YAML</span> اصلاً چیست؟

<span dir="ltr">YAML</span> یک <span dir="ltr">Data Serialization Language</span> است؛ یعنی روشی برای نوشتن دادهٔ ساختاریافته به شکلی که هم انسان راحت بخواند و هم برنامه بتواند آن را <span dir="ltr">Parse</span> کند.

مثلاً این اطلاعات:

```yaml
name: Amir
age: 22
city: Mashhad
```

برای مغز ما تقریباً این معنی را دارد:

```
name  → Amir
age   → 22
city  → Mashhad
```

و برای یک برنامه می‌تواند شبیه یک <span dir="ltr">Object / Dictionary</span> باشد.

اسم <span dir="ltr">YAML</span> از عبارت زیر می‌آید:

```
YAML Ain't Markup Language
```

<span dir="ltr">YAML</span> در کارهای <span dir="ltr">DevOps</span> خیلی زیاد دیده می‌شود، مثلاً:

```
Docker Compose      → compose.yaml
GitHub Actions      → .github/workflows/deploy.yml
Kubernetes          → deployment.yaml
Ansible             → playbook.yml
CI/CD tools         → pipeline configuration
Cloud tools         → infrastructure/config files
```

🧠 مدل ذهنی: <span dir="ltr">YAML</span> خودش برنامه را اجرا نمی‌کند؛ <span dir="ltr">YAML</span> فقط اطلاعات را با یک ساختار مشخص توصیف می‌کند.

<a id="yaml-vs-schema"></a>
## 2. 🚨 مهم‌ترین قانون: <span dir="ltr">YAML</span> با <span dir="ltr">Docker</span> فرق دارد

این احتمالاً مهم‌ترین نکتهٔ کل این داکیومنت است.

فرض کن این را می‌بینی:

```yaml
services:
  web:
    image: nginx
```

<span dir="ltr">YAML</span> فقط می‌فهمد:

```
services
└── web
    └── image: nginx
```

ولی اینکه `services` چیست، `image` چیست و آیا اصلاً چنین <span dir="ltr">Key</span>هایی مجازند یا نه را <span dir="ltr">Docker Compose</span> تعیین می‌کند.

یعنی دو لایه داریم:

```
لایه 1: YAML Syntax
        │
        │  فاصله، :, -, List, Mapping...
        ▼
لایه 2: Tool Schema
        │
        │  services, image, ports, jobs, steps...
        ▼
Docker / GitHub / Kubernetes / Ansible
```

مثلاً این از نظر <span dir="ltr">YAML</span> کاملاً معتبر است:

```yaml
pizza:
  cheese: lots
  olives:
    - black
    - green
```

ولی اگر آن را به عنوان `compose.yaml` بدهی به <span dir="ltr">Docker</span>:

```
Docker: داداش من pizza نمی‌شناسم 😐
```

✅ نتیجه: یک فایل می‌تواند از نظر <span dir="ltr">YAML</span> درست باشد ولی از نظر <span dir="ltr">Docker Compose</span> یا <span dir="ltr">Kubernetes</span> کاملاً غلط باشد.

<a id="yaml-extension"></a>
## 3. 📄 فایل <span dir="ltr">.yaml</span> یا <span dir="ltr">.yml</span>؟

هر دو معمولاً برای <span dir="ltr">YAML</span> استفاده می‌شوند:

```
config.yaml
config.yml
```

از نظر مفهوم <span dir="ltr">YAML</span> تفاوتی ندارند.

در پروژه‌های جدید <span dir="ltr">Docker</span> معمولاً اسم رایج و پیشنهادی:

```
compose.yaml
```

است.

در <span dir="ltr">GitHub Actions</span> هم هر دو <span dir="ltr">Extension</span> پذیرفته می‌شوند:

```
.github/workflows/ci.yml
.github/workflows/deploy.yaml
```

🎯 داخل یک پروژه یکی را انتخاب کن و بی‌دلیل بین <span dir="ltr">.yaml</span> و <span dir="ltr">.yml</span> پینگ‌پنگ نکن.

<a id="yaml-three-types"></a>
## 4. 🧱 سه ساختمان اصلی <span dir="ltr">YAML</span>

تقریباً هر چیز مهمی که در <span dir="ltr">YAML</span> می‌بینی از سه نوع اصلی ساخته شده:

```
1. Scalar    → یک مقدار
2. Mapping   → Key / Value
3. Sequence  → List
```

اگر این سه‌تا را بفهمی، ۷۰٪ ترس <span dir="ltr">YAML</span> همان‌جا می‌ریزد 😄

<a id="yaml-scalar"></a>
### 4.1. <span dir="ltr">Scalar</span> — یک مقدار

<span dir="ltr">Scalar</span> یعنی یک مقدار ساده:

```yaml
name: Amir
age: 22
enabled: true
price: 12.5
value: null
```

اینجا این‌ها <span dir="ltr">Scalar</span> هستند:

```
Amir
22
true
12.5
null
```

<a id="yaml-mapping"></a>
### 4.2. <span dir="ltr">Mapping</span> — کلید و مقدار

<span dir="ltr">Mapping</span> تقریباً شبیه <span dir="ltr">Object</span> در <span dir="ltr">JavaScript</span> یا <span dir="ltr">Dictionary</span> در <span dir="ltr">Python</span> است.

```yaml
user:
  name: Amir
  age: 22
```

مدل ذهنی:

```
user
├── name → Amir
└── age  → 22
```

معادل تقریبی <span dir="ltr">JSON</span>:

```json
{
  "user": {
    "name": "Amir",
    "age": 22
  }
}
```

<a id="yaml-sequence"></a>
### 4.3. <span dir="ltr">Sequence</span> — لیست

<span dir="ltr">Sequence</span> یعنی <span dir="ltr">List / Array</span>:

```yaml
colors:
  - black
  - white
  - gray
```

مدل ذهنی:

```
colors
├── black
├── white
└── gray
```

معادل تقریبی <span dir="ltr">JSON</span>:

```json
{
  "colors": ["black", "white", "gray"]
}
```

<a id="yaml-indentation"></a>
## 5. 📐 قانون طلایی <span dir="ltr">Indentation</span>

در <span dir="ltr">YAML</span> فاصلهٔ ابتدای خط معنی ساختاری دارد.

مثلاً:

```yaml
person:
  name: Amir
  address:
    city: Mashhad
    country: Iran
```

ساختار:

```
person
├── name: Amir
└── address
    ├── city: Mashhad
    └── country: Iran
```

ما در این راهنما برای هر <span dir="ltr">Level</span> از دو <span dir="ltr">Space</span> استفاده می‌کنیم:

```
Level 0 → 0 space
Level 1 → 2 spaces
Level 2 → 4 spaces
Level 3 → 6 spaces
```

مثال:

```yaml
level0:
  level1:
    level2:
      value: hello
```

❌ اشتباه

```yaml
services:
   web:
    image: nginx
```

اینجا فاصله‌ها نامنظم‌اند و احتمال خطا یا ساختار اشتباه زیاد است.

✅ بهتر

```yaml
services:
  web:
    image: nginx
```

🚨 برای <span dir="ltr">Indentation</span> از <span dir="ltr">Tab</span> استفاده نکن. <span dir="ltr">YAML</span> در <span dir="ltr">Block Structure</span> برای <span dir="ltr">Indent</span> به <span dir="ltr">Space</span> متکی است. <span dir="ltr">Editor</span> را طوری تنظیم کن که <span dir="ltr">Tab</span> برای <span dir="ltr">YAML</span> به <span dir="ltr">Space</span> تبدیل شود.

<a id="yaml-colon"></a>
## 6. 🔑 <span dir="ltr">:</span> دقیقاً چه کاری می‌کند؟

<span dir="ltr">Colon</span> معمولاً <span dir="ltr">Key</span> را از <span dir="ltr">Value</span> جدا می‌کند:

```yaml
name: Amir
```

```
Key   = name
Value = Amir
```

اگر بعد از `:` چیزی نیاید:

```yaml
database:
```

معمولاً یعنی قرار است زیر آن یک ساختار <span dir="ltr">Nested</span> قرار بگیرد:

```yaml
database:
  host: localhost
  port: 5432
```

**قانون خواندن**

وقتی می‌بینی:

```yaml
something:
```

از خودت بپرس:

«چه چیزهایی متعلق به `something` هستند؟»

بعد خطوط <span dir="ltr">Indent</span> شدهٔ زیرش را نگاه کن.

<a id="yaml-dash"></a>
## 7. ➖ <span dir="ltr">-</span> دقیقاً چه کاری می‌کند؟

<span dir="ltr">Dash + Space</span> یعنی یک عضو از <span dir="ltr">List</span>:

```yaml
developers:
  - Ali
  - Sara
  - Reza
```

<span dir="ltr">List</span> می‌تواند از <span dir="ltr">Object</span> هم تشکیل شود:

```yaml
developers:
  - name: Ali
    role: backend

  - name: Sara
    role: frontend
```

ساختار:

```
developers
├── item 1
│   ├── name: Ali
│   └── role: backend
│
└── item 2
    ├── name: Sara
    └── role: frontend
```

این الگو را در <span dir="ltr">Kubernetes</span>، <span dir="ltr">Ansible</span> و <span dir="ltr">GitHub Actions</span> خیلی زیاد می‌بینی.

<a id="yaml-nesting"></a>
## 8. 🪆 ترکیب <span dir="ltr">Mapping</span> و <span dir="ltr">List</span>

اینجاست که <span dir="ltr">YAML</span> اولش ترسناک به نظر می‌رسد.

مثلاً:

```yaml
team:
  name: DevOps
  members:
    - name: Ali
      skills:
        - Docker
        - Linux

    - name: Sara
      skills:
        - Kubernetes
        - Terraform
```

آرام بخوان:

```
team
├── name: DevOps
└── members [LIST]
    │
    ├── member 1
    │   ├── name: Ali
    │   └── skills [LIST]
    │       ├── Docker
    │       └── Linux
    │
    └── member 2
        ├── name: Sara
        └── skills [LIST]
            ├── Kubernetes
            └── Terraform
```

🧠 اگر <span dir="ltr">YAML</span> پیچیده شد، آن را تبدیل به درخت کن. ناگهان همه‌چیز خیلی قابل‌فهم‌تر می‌شود.

<a id="yaml-comments"></a>
## 9. 💬 <span dir="ltr">Comment</span> نویسی با <span dir="ltr">#</span>

<span dir="ltr">Comment</span>:

```yaml
# This is a comment
name: Amir
```

<span dir="ltr">Inline Comment</span>:

```yaml
port: 3000 # application port
```

اما حواست باشد `#` داخل <span dir="ltr">String</span> ممکن است بخشی از مقدار باشد:

```yaml
password: "abc#123"
```

برای <span dir="ltr">Value</span>هایی که `#` دارند <span dir="ltr">Quote</span> گذاشتن انتخاب خوبی است.

<a id="yaml-values"></a>
## 10. 🎭 <span dir="ltr">String</span>، <span dir="ltr">Number</span>، <span dir="ltr">Boolean</span> و <span dir="ltr">Null</span>

مثال:

```yaml
name: Amir
age: 22
price: 12.5
debug: true
cache_enabled: false
optional_value: null
```

نوع ذهنی:

| مقدار | نوع تقریبی |
|---|---|
| <span dir="ltr">Amir</span> | <span dir="ltr">String</span> |
| <span dir="ltr">22</span> | <span dir="ltr">Integer</span> |
| <span dir="ltr">12.5</span> | <span dir="ltr">Float</span> |
| <span dir="ltr">true</span> | <span dir="ltr">Boolean</span> |
| <span dir="ltr">null</span> | <span dir="ltr">Null</span> |

<span dir="ltr">YAML Parser</span>ها و ابزارهای مختلف ممکن است در <span dir="ltr">Resolution</span> بعضی <span dir="ltr">Value</span>های مبهم رفتار متفاوتی داشته باشند.

برای داده‌هایی که حتماً باید <span dir="ltr">String</span> بمانند، <span dir="ltr">Quote</span> کمک می‌کند:

```yaml
app_version: "1.10"
phone_like_value: "01234"
answer: "yes"
mode: "on"
```

🎯 قاعدهٔ امن <span dir="ltr">DevOps</span>: اگر مقدار از نظر ظاهر می‌تواند اشتباه به <span dir="ltr">Number / Boolean / Date</span> تعبیر شود ولی برای تو <span dir="ltr">String</span> است، <span dir="ltr">Quote</span> بگذار.

<a id="yaml-quotes"></a>
## 11. 🗣️ چه وقت <span dir="ltr">Quote</span> بگذاریم؟

این‌ها مثال‌های خوبی برای <span dir="ltr">Quote</span> هستند:

```yaml
port_mapping: "8080:80"
image: "myapp:1.4.2"
url: "https://example.com/api"
cron: "0 3 * * *"
version: "1.0"
password: "abc#123"
```

این <span dir="ltr">Value</span> معمولاً بدون <span dir="ltr">Quote</span> هم مشکلی ندارد:

```yaml
name: backend
replicas: 3
enabled: true
```

**پیشنهاد عملی من**

برای <span dir="ltr">Docker Compose</span>:

```yaml
ports:
  - "3000:3000"
  - "8080:80"
```

را به همین شکل <span dir="ltr">Quote</span>‌شده ترجیح بده؛ <span dir="ltr">Quote</span> خوانایی و نیت تو را واضح‌تر می‌کند.

<a id="yaml-quote-types"></a>
## 12. 🥊 <span dir="ltr">Single Quote</span> و <span dir="ltr">Double Quote</span>

**<span dir="ltr">Double Quote</span>**

```yaml
message: "Hello World"
```

<span dir="ltr">Escape Sequence</span>ها داخل <span dir="ltr">Double Quote</span> معنا پیدا می‌کنند:

```yaml
message: "line1\nline2"
```

**<span dir="ltr">Single Quote</span>**

```yaml
message: 'Hello World'
```

معمولاً <span dir="ltr">Literal</span>تر رفتار می‌کند.

مثلاً در سناریوهای <span dir="ltr">Variable</span>:

```yaml
value1: "${APP_PORT}"
value2: '${APP_PORT}'
```

اما اینکه <span dir="ltr">Tool</span> موردنظر <span dir="ltr">Interpolation</span> را قبل یا بعد از <span dir="ltr">Parse</span> چگونه انجام دهد، به خود <span dir="ltr">Tool</span> بستگی دارد.

🚨 <span dir="ltr">YAML Syntax</span> را با <span dir="ltr">Variable System</span> ابزار قاطی نکن. <span dir="ltr">${...}</span> ویژگی خود <span dir="ltr">YAML</span> نیست؛ مثلاً <span dir="ltr">Docker Compose</span> آن را تفسیر می‌کند.

<a id="yaml-multiline"></a>
## 13. 📜 متن چندخطی با <span dir="ltr">|</span> و <span dir="ltr">></span>

**<span dir="ltr">|</span> — <span dir="ltr">Line Break</span>ها را نگه دار**

```yaml
script: |
  echo "step 1"
  echo "step 2"
  echo "step 3"
```

تقریباً خروجی:

```
echo "step 1"
echo "step 2"
echo "step 3"
```

این در <span dir="ltr">CI/CD</span> خیلی کاربرد دارد.

**<span dir="ltr">></span> — خط‌ها را <span dir="ltr">Fold</span> کن**

```yaml
description: >
  This is a long
  description that can
  be written on multiple lines.
```

خط‌های معمولی به <span dir="ltr">Space</span> تبدیل می‌شوند.

**مثال <span dir="ltr">GitHub Actions</span>**

```yaml
- name: Build application
  run: |
    npm ci
    npm test
    npm run build
```

🧠 برای <span dir="ltr">Shell Script</span> چندخطی معمولاً `|` همان چیزی است که می‌خواهی.

<a id="yaml-flow"></a>
## 14. 🧺 <span dir="ltr">{}</span> و <span dir="ltr">[]</span>؛ <span dir="ltr">Flow Style</span>

<span dir="ltr">YAML</span> می‌تواند شبیه <span dir="ltr">JSON</span> هم نوشته شود.

**<span dir="ltr">List</span>**

```yaml
ports: [80, 443, 8080]
```

معادل:

```yaml
ports:
  - 80
  - 443
  - 8080
```

**<span dir="ltr">Mapping</span>**

```yaml
user: {name: Amir, age: 22}
```

معادل:

```yaml
user:
  name: Amir
  age: 22
```

برای <span dir="ltr">Configuration</span>های <span dir="ltr">DevOps</span> معمولاً <span dir="ltr">Block Style</span> خواناتر است.

<a id="yaml-documents"></a>
## 15. 📚 <span dir="ltr">---</span> و <span dir="ltr">...</span> یعنی چه؟

`---` می‌تواند شروع یک <span dir="ltr">YAML Document</span> را مشخص کند:

```yaml
---
name: app1
```

و می‌توان چند <span dir="ltr">Document</span> را در یک فایل قرار داد:

```yaml
---
name: app1

---
name: app2
```

این در <span dir="ltr">Kubernetes</span> کاربردی است:

```yaml
---
apiVersion: apps/v1
kind: Deployment
# ...

---
apiVersion: v1
kind: Service
# ...
```

`...` می‌تواند پایان <span dir="ltr">Document</span> را مشخص کند ولی در بسیاری از فایل‌های <span dir="ltr">DevOps</span> لازم نیست.

<a id="yaml-anchor"></a>
## 16. ⚓ <span dir="ltr">Anchor</span> و <span dir="ltr">Alias</span> با <span dir="ltr">&</span> و <span dir="ltr">*</span>

فرض کن یک <span dir="ltr">Configuration</span> تکراری داری.

<span dir="ltr">Anchor</span> تعریف می‌کنی:

```yaml
default_environment: &default_env
  NODE_ENV: production
  LOG_LEVEL: info
```

و <span dir="ltr">Alias</span> استفاده می‌کنی:

```yaml
app:
  environment: *default_env
```

مدل ذهنی:

```
&default_env → یک تکه را اسم‌گذاری کن
*default_env → همان تکه را دوباره استفاده کن
```

مثال کامل:

```yaml
common: &common
  retries: 3
  timeout: 10

service_a:
  config: *common

service_b:
  config: *common
```

🎯 <span dir="ltr">Anchor</span> و <span dir="ltr">Alias</span> بخشی از <span dir="ltr">YAML</span> هستند؛ ولی اینکه <span dir="ltr">Tool</span> با <span dir="ltr">Merge</span> یا ساختار نهایی چه رفتاری دارد باید با <span dir="ltr">Documentation</span> همان <span dir="ltr">Tool</span> چک شود.

<a id="yaml-merge"></a>
## 17. 🧬 <span dir="ltr">Merge</span> با <span dir="ltr"><<</span>

در بعضی <span dir="ltr">Tool</span>ها و <span dir="ltr">Parser</span>ها، خصوصاً سناریوهای <span dir="ltr">Compose</span>، <span dir="ltr">Merge</span> برای <span dir="ltr">Mapping</span> بسیار کاربردی است:

```yaml
defaults: &defaults
  restart: unless-stopped
  environment:
    LOG_LEVEL: info

services:
  api:
    <<: *defaults
    image: my-api:latest
```

یعنی تقریباً:

```yaml
services:
  api:
    restart: unless-stopped
    environment:
      LOG_LEVEL: info
    image: my-api:latest
```

⚠️ <span dir="ltr">Merge</span> فقط برای <span dir="ltr">Mapping</span> منطقی است؛ برای <span dir="ltr">List</span>ها رفتار یکسانی انتظار نداشته باش.

<a id="yaml-errors"></a>
## 18. 💣 خطاهای معروف <span dir="ltr">YAML</span>

**خطا 1 — <span dir="ltr">Tab</span>**

```
services:
<TAB>web:
```

نکن 😐

**خطا 2 — <span dir="ltr">Indentation</span> ناهماهنگ**

```yaml
services:
  web:
     image: nginx
    ports:
      - "80:80"
```

**خطا 3 — فراموش کردن <span dir="ltr">-</span>**

غلط:

```yaml
ports:
  "80:80"
  "443:443"
```

درست:

```yaml
ports:
  - "80:80"
  - "443:443"
```

**خطا 4 — <span dir="ltr">List</span> و <span dir="ltr">Mapping</span> را قاطی کنیم**

غلط:

```yaml
environment:
  - NODE_ENV: production
    PORT: 3000
```

ممکن است <span dir="ltr">Parser</span> ساختاری غیر از چیزی که فکر می‌کنی بسازد.

برای <span dir="ltr">Compose</span> یکی از دو <span dir="ltr">Style</span> شفاف را انتخاب کن:

```yaml
environment:
  NODE_ENV: production
  PORT: "3000"
```

یا:

```yaml
environment:
  - NODE_ENV=production
  - PORT=3000
```

**خطا 5 — <span dir="ltr">Duplicate Key</span>**

```yaml
app:
  port: 3000
  port: 4000
```

بسته به <span dir="ltr">Parser</span> ممکن است <span dir="ltr">Error</span> بگیری یا یکی <span dir="ltr">Override</span> شود. روی چنین چیزی حساب نکن.

**خطا 6 — <span dir="ltr">Value</span> حساس بدون <span dir="ltr">Quote</span>**

```yaml
token: abc#123
```

اینجا احتمال دارد بخشی بعد از `#` <span dir="ltr">Comment</span> دیده شود.

بهتر:

```yaml
token: "abc#123"
```

<a id="yaml-reading-method"></a>
## 19. 🧠 روش ذهنی خواندن هر فایل <span dir="ltr">YAML</span>

هر بار یک <span dir="ltr">YAML</span> دیدی این ۵ سؤال را بپرس:

```
1. Top-level Keyها چی هستند؟
2. کدام بخش Mapping است؟
3. کدام بخش List است؟
4. هر Indentation یعنی «متعلق به چه چیزی»؟
5. کدام Key مربوط به YAML نیست و مربوط به Tool است؟
```

مثلاً:

```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"
```

تحلیل:

```
services        → Mapping
web             → یک Key داخل services
image           → Key داخل web
nginx           → Scalar
ports           → Key داخل web
- "8080:80"     → List Item
```

حالا بعد می‌رویم سراغ <span dir="ltr">Docker</span> و می‌پرسیم `image` و `ports` چه معنایی دارند.

<a id="compose-intro"></a>
## 20. 🐳 <span dir="ltr">YAML</span> در <span dir="ltr">Docker Compose</span>

<span dir="ltr">Compose</span> از <span dir="ltr">YAML</span> برای تعریف <span dir="ltr">Application</span> چندسرویسی استفاده می‌کند.

ساختار رایج:

```yaml
services:
  web:
    image: nginx

volumes:
  app_data:

networks:
  app_network:
```

<span dir="ltr">Top-level Element</span>های مهم <span dir="ltr">Compose</span> شامل این‌ها هستند:

```
services
networks
volumes
configs
secrets
```

در نسخه‌های جدید <span dir="ltr">Compose Specification</span> معمولاً لازم نیست `version: "3"` یا `version: "3.8"` را مثل <span dir="ltr">Tutorial</span>های قدیمی بنویسی.

<a id="compose-services"></a>
## 21. 🧩 کالبدشکافی <span dir="ltr">services</span>

هر <span dir="ltr">Container/Application Component</span> معمولاً زیر `services` تعریف می‌شود:

```yaml
services:
  frontend:
    image: nginx

  backend:
    image: my-api

  db:
    image: postgres
```

درخت:

```
services
├── frontend
├── backend
└── db
```

اسم‌های `frontend`، `backend` و `db` را تو انتخاب می‌کنی.

این اسم‌ها در <span dir="ltr">Network</span> داخلی <span dir="ltr">Compose</span> هم مهم می‌شوند؛ <span dir="ltr">Service</span>ها معمولاً می‌توانند همدیگر را با نام <span dir="ltr">Service</span> پیدا کنند.

<a id="compose-build"></a>
## 22. 🏗️ <span dir="ltr">build</span> و <span dir="ltr">context</span>

وقتی <span dir="ltr">Image</span> را خودت از <span dir="ltr">Source Code</span> می‌سازی:

```yaml
services:
  api:
    build:
      context: ./backend
```

یعنی:

```
compose.yaml
   │
   └── api
       └── build
           └── context → ./backend
```

معمولاً <span dir="ltr">Docker</span> در آن <span dir="ltr">Build Context</span> دنبال <span dir="ltr">Dockerfile</span> می‌گردد.

اگر <span dir="ltr">Dockerfile</span> اسم یا مسیر دیگری دارد:

```yaml
services:
  api:
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
```

<a id="compose-image"></a>
## 23. 🖼️ <span dir="ltr">image</span>

اگر <span dir="ltr">Image</span> آماده داری:

```yaml
services:
  db:
    image: postgres:17
```

یا:

```yaml
services:
  redis:
    image: redis:alpine
```

فرق ذهنی:

```
build  → خودم Image را می‌سازم
image  → Image مشخصی را استفاده کن
```

گاهی هر دو با هم استفاده می‌شوند تا <span dir="ltr">Image</span> ساخته‌شده <span dir="ltr">Tag</span> هم بگیرد:

```yaml
services:
  api:
    build: ./backend
    image: mycompany/api:dev
```

<a id="compose-ports"></a>
## 24. 🚪 <span dir="ltr">ports</span>

مثال:

```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"
```

فرمول:

```
HOST_PORT : CONTAINER_PORT
```

پس:

```
localhost:8080
      │
      ▼
container:80
```

چند <span dir="ltr">Port</span>:

```yaml
ports:
  - "8080:80"
  - "8443:443"
```

<span dir="ltr">Bind</span> فقط به <span dir="ltr">localhost</span>:

```yaml
ports:
  - "127.0.0.1:8080:80"
```

🔐 این آخری برای <span dir="ltr">Service</span>هایی که نباید از <span dir="ltr">Network</span> بیرونی قابل دسترس باشند خیلی مهم است.

<a id="compose-environment"></a>
## 25. 🌱 <span dir="ltr">environment</span>

دو <span dir="ltr">Style</span> رایج:

**<span dir="ltr">Mapping Style</span>**

```yaml
environment:
  NODE_ENV: production
  PORT: "3000"
```

**<span dir="ltr">List Style</span>**

```yaml
environment:
  - NODE_ENV=production
  - PORT=3000
```

برای خوانایی و <span dir="ltr">Anchor/Merge</span>، <span dir="ltr">Mapping Style</span> خیلی وقت‌ها بهتر است.

<a id="compose-env-file"></a>
## 26. 🌿 <span dir="ltr">.env</span> و <span dir="ltr">Variable Interpolation</span>

فایل `.env`:

```ini
APP_PORT=3000
POSTGRES_VERSION=17
```

<span dir="ltr">Compose</span>:

```yaml
services:
  api:
    image: "my-api:latest"
    ports:
      - "${APP_PORT}:3000"

  db:
    image: "postgres:${POSTGRES_VERSION}"
```

<span dir="ltr">Docker Compose</span> قبل از اجرا <span dir="ltr">Variable</span>ها را <span dir="ltr">Resolve</span> می‌کند.

**مقدار <span dir="ltr">Default</span>**

```yaml
ports:
  - "${APP_PORT:-3000}:3000"
```

یعنی اگر `APP_PORT` نبود، `3000` را استفاده کن.

**<span dir="ltr">Variable</span> اجباری**

```yaml
image: "my-api:${APP_TAG:?APP_TAG is required}"
```

اگر `APP_TAG` تعریف نشده باشد <span dir="ltr">Compose</span> <span dir="ltr">Error</span> می‌دهد.

**فرق مهم <span dir="ltr">.env</span> و <span dir="ltr">environment</span>**

```
.env
   │
   ├── می‌تواند Compose file را interpolate کند
   │
   └── لزوماً Variable را داخل Container نمی‌فرستد
```

برای داخل <span dir="ltr">Container</span> معمولاً:

```yaml
environment:
  APP_ENV: "${APP_ENV}"
```

یا:

```yaml
env_file:
  - .env.app
```

استفاده می‌شود.

<a id="compose-volumes"></a>
## 27. 💽 <span dir="ltr">volumes</span>

**<span dir="ltr">Named Volume</span>**

```yaml
services:
  db:
    image: postgres:17
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

مدل ذهنی:

```
Named Volume
postgres_data
     │
     ▼
/var/lib/postgresql/data
داخل Container
```

حذف <span dir="ltr">Container</span> به‌تنهایی <span dir="ltr">Volume</span> را لزوماً حذف نمی‌کند.

**<span dir="ltr">Bind Mount</span>**

```yaml
services:
  api:
    volumes:
      - ./backend:/app
```

یعنی <span dir="ltr">Folder</span> واقعی <span dir="ltr">Host</span> یعنی `./backend` به `/app` داخل <span dir="ltr">Container</span> وصل شود.

<a id="compose-networks"></a>
## 28. 🌐 <span dir="ltr">networks</span>

```yaml
services:
  frontend:
    image: my-frontend
    networks:
      - app_net

  backend:
    image: my-backend
    networks:
      - app_net

networks:
  app_net:
```

<span dir="ltr">Frontend</span> می‌تواند معمولاً <span dir="ltr">Backend</span> را با <span dir="ltr">Service Name</span> پیدا کند:

```
http://backend:3000
```

نه با <span dir="ltr">IP</span> ثابت.

🧠 در <span dir="ltr">Docker Compose</span>، <span dir="ltr">localhost</span> داخل <span dir="ltr">Frontend Container</span> یعنی خود <span dir="ltr">Frontend Container</span>، نه <span dir="ltr">Backend</span> و نه کامپیوتر تو.

<a id="compose-health"></a>
## 29. ❤️ <span dir="ltr">depends_on</span> و <span dir="ltr">healthcheck</span>

این:

```yaml
depends_on:
  - db
```

به‌تنهایی الزاماً یعنی «صبر کن <span dir="ltr">DB</span> کاملاً آمادهٔ <span dir="ltr">Query</span> شود» نیست.

برای آمادگی واقعی می‌توانی <span dir="ltr">Healthcheck</span> تعریف کنی.

مثال:

```yaml
services:
  db:
    image: postgres:17
    environment:
      POSTGRES_PASSWORD: example
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 10

  api:
    build: ./backend
    depends_on:
      db:
        condition: service_healthy
```

مدل ذهنی:

```
db container started
       │
       ▼
healthcheck succeeds
       │
       ▼
db = healthy
       │
       ▼
api starts
```

<a id="compose-restart"></a>
## 30. 🔁 <span dir="ltr">restart</span>

مثلاً:

```yaml
services:
  api:
    image: my-api
    restart: unless-stopped
```

<span dir="ltr">Policy</span>های رایج را بسته به نیاز انتخاب می‌کنی:

```
no
always
on-failure
unless-stopped
```

کورکورانه `always` نزن؛ بفهم <span dir="ltr">Service</span> چه رفتاری باید داشته باشد.

<a id="compose-command"></a>
## 31. 🧾 <span dir="ltr">command</span> و <span dir="ltr">entrypoint</span>

می‌توان <span dir="ltr">Command</span> پیش‌فرض <span dir="ltr">Image</span> را <span dir="ltr">Override</span> کرد:

```yaml
services:
  api:
    image: node:22-alpine
    command: ["npm", "run", "dev"]
```

یا:

```yaml
command: >
  sh -c "npm ci &&
         npm run dev"
```

برای <span dir="ltr">Shell</span> چندمرحله‌ای معمولاً `|` هم خواناست:

```yaml
command:
  - sh
  - -c
  - |
    npm ci
    npm run dev
```

<a id="compose-secrets"></a>
## 32. 🔐 <span dir="ltr">Secrets</span>؛ <span dir="ltr">Password</span> را کجا نگذاریم؟

❌ بد

```yaml
environment:
  DB_PASSWORD: super-secret-password
```

اگر فایل داخل <span dir="ltr">Git Commit</span> شود، <span dir="ltr">Secret</span> رفت توی تاریخچه 😐

**بهتر**

<span dir="ltr">Secret</span> را از <span dir="ltr">Repository</span> جدا نگه دار.

<span dir="ltr">Compose</span> قابلیت `secrets` هم دارد:

```yaml
services:
  db:
    image: postgres:17
    secrets:
      - db_password

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

🚨 فایل واقعی <span dir="ltr">Secret</span> را <span dir="ltr">Commit</span> نکن. <span dir="ltr">.gitignore</span> و <span dir="ltr">Secret Management</span> را جدی بگیر.

<a id="scenario-nginx"></a>
## 33. 🧪 سناریو 1 — <span dir="ltr">Nginx</span> تک‌سرویسی

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

اجرا:

```bash
docker compose up -d
```

بعد:

```
http://localhost:8080
```

تحلیل <span dir="ltr">YAML</span>:

```
services      → Mapping
web           → Mapping
image         → Scalar
ports         → Sequence
"8080:80"     → Scalar داخل Sequence
```

تحلیل <span dir="ltr">Docker</span>:

```
web service
   │
   ├── image = nginx:alpine
   └── host 8080 → container 80
```

<a id="scenario-fullstack"></a>
## 34. 🧪 سناریو 2 — <span dir="ltr">Frontend + Backend</span>

ساختار پروژه:

```
project/
├── frontend/
│   └── Dockerfile
├── backend/
│   └── Dockerfile
└── compose.yaml
```

`compose.yaml`:

```yaml
services:
  frontend:
    build:
      context: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend

  backend:
    build:
      context: ./backend
    ports:
      - "3001:3001"
```

نکته:

اگر <span dir="ltr">Frontend</span> داخل <span dir="ltr">Container</span> بخواهد به <span dir="ltr">Backend</span> وصل شود، بسته به <span dir="ltr">Architecture</span> برنامه، آدرس `http://backend:3001` ممکن است <span dir="ltr">Address</span> داخلی مناسب باشد؛ اما کدی که داخل <span dir="ltr">Browser</span> کاربر اجرا می‌شود نمی‌تواند لزوماً <span dir="ltr">DNS</span> داخلی <span dir="ltr">Docker</span> را <span dir="ltr">Resolve</span> کند. پس فرق <span dir="ltr">Server-side</span> و <span dir="ltr">Browser-side</span> را بفهم.

<a id="scenario-mongo"></a>
## 35. 🧪 سناریو 3 — <span dir="ltr">Frontend + API + MongoDB</span>

این دقیقاً نزدیک همان مدلی است که خیلی از <span dir="ltr">Tutorial</span>ها نشان می‌دهند:

```yaml
services:
  web:
    build:
      context: ./frontend
    ports:
      - "3000:3000"

  api:
    build:
      context: ./backend
    ports:
      - "3001:3001"
    environment:
      MONGO_URL: "mongodb://admin:pass123@db:27017/app?authSource=admin"
    depends_on:
      - db

  db:
    image: mongo:8
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: pass123
```

درخت:

```
services
├── web
│   ├── build
│   │   └── context: ./frontend
│   └── ports
│       └── 3000:3000
│
├── api
│   ├── build
│   │   └── context: ./backend
│   ├── ports
│   │   └── 3001:3001
│   ├── environment
│   └── depends_on
│
└── db
    ├── image: mongo:8
    ├── ports
    └── environment
```

**ایراد مهم این نسخه**

<span dir="ltr">Password</span> داخل فایل <span dir="ltr">Hard-code</span> شده.

در بخش‌های بعدی آن را بهتر می‌کنیم.

<a id="scenario-volume"></a>
## 36. 🧪 سناریو 4 — <span dir="ltr">Database</span> با <span dir="ltr">Volume</span>

```yaml
services:
  db:
    image: mongo:8
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: "${MONGO_PASSWORD}"
    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data:
```

`.env`:

```ini
MONGO_PASSWORD=change-me-locally
```

حالا داده:

```
Container db
     │
     ▼
/data/db
     │
     ▼
mongo_data volume
```

با <span dir="ltr">Recreate</span> شدن <span dir="ltr">Container</span>، داده روی <span dir="ltr">Named Volume</span> می‌تواند باقی بماند.

<a id="scenario-network"></a>
## 37. 🧪 سناریو 5 — شبکهٔ اختصاصی

```yaml
services:
  api:
    build: ./backend
    networks:
      - internal

  db:
    image: postgres:17
    networks:
      - internal

networks:
  internal:
```

<span dir="ltr">API</span> آدرس `db:5432` را به عنوان <span dir="ltr">Address</span> داخلی <span dir="ltr">Database</span> استفاده می‌کند.

لازم نیست <span dir="ltr">Database Port</span> را حتماً روی <span dir="ltr">Host Publish</span> کنی:

```yaml
db:
  image: postgres:17
```

اگر فقط <span dir="ltr">API</span> داخل <span dir="ltr">Compose</span> باید به <span dir="ltr">DB</span> دسترسی داشته باشد، حذف `ports` از <span dir="ltr">DB</span> می‌تواند <span dir="ltr">Exposure</span> را کمتر کند.

<a id="scenario-envs"></a>
## 38. 🧪 سناریو 6 — <span dir="ltr">.env</span> برای <span dir="ltr">Development</span> و <span dir="ltr">Production</span>

`compose.yaml`:

```yaml
services:
  api:
    image: "mycompany/api:${APP_TAG}"
    ports:
      - "${APP_PORT}:3000"
    environment:
      APP_ENV: "${APP_ENV}"
```

`.env.dev`:

```ini
APP_TAG=dev
APP_PORT=3000
APP_ENV=development
```

`.env.prod`:

```ini
APP_TAG=1.4.2
APP_PORT=8080
APP_ENV=production
```

<span dir="ltr">Development</span>:

```bash
docker compose --env-file .env.dev up -d
```

<span dir="ltr">Production</span>:

```bash
docker compose --env-file .env.prod up -d
```

قبل از <span dir="ltr">Run</span> نتیجه را ببین:

```bash
docker compose --env-file .env.prod config
```

<a id="scenario-health"></a>
## 39. 🧪 سناریو 7 — <span dir="ltr">Healthcheck</span> واقعی

```yaml
services:
  redis:
    image: redis:alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
      start_period: 10s

  api:
    build: ./backend
    depends_on:
      redis:
        condition: service_healthy
```

این سناریو برای جلوگیری از <span dir="ltr">Startup Race</span> خیلی مفید است.

<a id="scenario-override"></a>
## 40. 🧪 سناریو 8 — <span dir="ltr">Compose</span> چندفایلی و <span dir="ltr">Override</span>

`compose.yaml`:

```yaml
services:
  api:
    image: mycompany/api:1.0
    environment:
      APP_ENV: production
```

`compose.override.yaml`:

```yaml
services:
  api:
    environment:
      APP_ENV: development
    volumes:
      - ./backend:/app
```

<span dir="ltr">Compose</span> می‌تواند <span dir="ltr">Configuration</span>ها را <span dir="ltr">Merge</span> کند.

یا دستی:

```bash
docker compose \
  -f compose.yaml \
  -f compose.dev.yaml \
  up -d
```

🧠 فایل‌های بعدی می‌توانند بخش‌هایی از فایل قبلی را <span dir="ltr">Override / Merge</span> کنند؛ رفتار دقیق <span dir="ltr">Mapping</span> و <span dir="ltr">Sequence</span> را باید با <span dir="ltr">Compose Rules</span> بشناسی.

<a id="compose-reuse"></a>
## 41. ♻️ <span dir="ltr">Reuse</span> در <span dir="ltr">Compose</span> با <span dir="ltr">x-</span> و <span dir="ltr">Anchor</span>

<span dir="ltr">Compose Extension Field</span>هایی که با `x-` شروع می‌شوند برای <span dir="ltr">Reuse</span> خیلی مفیدند.

```yaml
x-common: &common
  restart: unless-stopped
  environment:
    LOG_LEVEL: info

services:
  api:
    <<: *common
    image: my-api:latest

  worker:
    <<: *common
    image: my-worker:latest
```

اینجا:

```
x-common
   │
   └── &common
         │
         ├── api    ← *common
         └── worker ← *common
```

برای <span dir="ltr">Environment</span> مشترک:

```yaml
x-env: &default_env
  LOG_LEVEL: info
  TZ: UTC

services:
  api:
    image: my-api
    environment:
      <<: *default_env
      APP_MODE: api

  worker:
    image: my-worker
    environment:
      <<: *default_env
      APP_MODE: worker
```

<a id="compose-validation"></a>
## 42. ✅ تست و <span dir="ltr">Validate</span> کردن <span dir="ltr">Compose</span>

قبل از اینکه با `up` کل پروژه را منفجر کنی 😂:

```bash
docker compose config
```

این دستور فایل‌ها را <span dir="ltr">Parse</span> می‌کند، <span dir="ltr">Variable</span>ها را <span dir="ltr">Resolve</span> می‌کند، <span dir="ltr">Merge</span>ها را اعمال می‌کند و <span dir="ltr">Configuration</span> نهایی را نشان می‌دهد.

فقط <span dir="ltr">Validate</span>:

```bash
docker compose config -q
```

دیدن <span dir="ltr">Environment</span> مورد استفاده:

```bash
docker compose config --environment
```

دیدن <span dir="ltr">Service</span>ها:

```bash
docker compose config --services
```

🎯 عادت حرفه‌ای: بعد از تغییر مهم در `compose.yaml` اول `docker compose config`، بعد `docker compose up`.

<a id="github-actions"></a>
## 43. 🤖 <span dir="ltr">YAML</span> در <span dir="ltr">GitHub Actions</span>

<span dir="ltr">Workflow</span>ها در این مسیر قرار می‌گیرند:

```
.github/workflows/
```

مثلاً:

```
.github/workflows/ci.yml
```

ساختار پایه:

```yaml
name: CI

on:
  push:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Run tests
        run: npm test
```

حالا <span dir="ltr">YAML</span> را جدا بخوان:

```
name     → Scalar
on       → Mapping
jobs     → Mapping
test     → Mapping
steps    → Sequence
هر step  → Mapping
```

و <span dir="ltr">GitHub Schema</span> می‌گوید هر <span dir="ltr">Key</span> چه معنایی دارد.

<a id="github-actions-build"></a>
## 44. 🧪 <span dir="ltr">Workflow</span> واقعی <span dir="ltr">Build</span> و <span dir="ltr">Test</span>

```yaml
name: CI

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "22"

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build
```

`steps` یک <span dir="ltr">List</span> است:

```
steps
├── checkout
├── setup node
├── npm ci
├── npm test
└── npm run build
```

هر عضو <span dir="ltr">List</span> یک <span dir="ltr">Mapping</span> دارد.

<a id="github-actions-docker"></a>
## 45. 🐳 <span dir="ltr">Workflow</span> ساخت و <span dir="ltr">Push</span> کردن <span dir="ltr">Docker Image</span>

یک نمونهٔ آموزشی:

```yaml
name: Docker Build

on:
  push:
    branches:
      - main

jobs:
  docker:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Login to registry
        run: |
          echo "${{ secrets.REGISTRY_PASSWORD }}" |
          docker login \
            --username "${{ secrets.REGISTRY_USERNAME }}" \
            --password-stdin

      - name: Build image
        run: |
          docker build \
            -t "${{ secrets.REGISTRY_USERNAME }}/myapp:${{ github.sha }}" \
            .

      - name: Push image
        run: |
          docker push \
            "${{ secrets.REGISTRY_USERNAME }}/myapp:${{ github.sha }}"
```

🔐 برای <span dir="ltr">Production</span> معمولاً <span dir="ltr">Action</span>های رسمی / شناخته‌شدهٔ <span dir="ltr">Docker</span> برای <span dir="ltr">Login</span> و <span dir="ltr">Buildx</span> امکانات بیشتری دارند؛ این مثال عمداً ساده است تا <span dir="ltr">YAML</span> و <span dir="ltr">Flow</span> را بفهمی.

<a id="github-actions-secrets"></a>
## 46. 🔐 <span dir="ltr">Secrets</span> در <span dir="ltr">GitHub Actions</span>

این:

```yaml
password: my-real-password
```

فاجعه است.

از <span dir="ltr">GitHub Secrets</span> استفاده می‌کنی:

```
${{ secrets.REGISTRY_PASSWORD }}
```

مثلاً:

```yaml
env:
  API_TOKEN: "${{ secrets.API_TOKEN }}"
```

نکته: <span dir="ltr">`${{ ... }}`</span> <span dir="ltr">Syntax</span> خود <span dir="ltr">GitHub Expressions</span> است؛ <span dir="ltr">YAML</span> نیست. <span dir="ltr">YAML</span> فقط آن را به عنوان <span dir="ltr">Value</span> حمل می‌کند.

<a id="kubernetes"></a>
## 47. ☸️ <span dir="ltr">YAML</span> در <span dir="ltr">Kubernetes</span>

<span dir="ltr">Manifest</span>های <span dir="ltr">Kubernetes</span> معمولاً با <span dir="ltr">YAML</span> نوشته می‌شوند.

ساختار پایهٔ خیلی از <span dir="ltr">Resource</span>ها:

```yaml
apiVersion: ...
kind: ...
metadata:
  name: ...
spec:
  ...
```

مثلاً:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
```

<span dir="ltr">YAML</span> فقط <span dir="ltr">Structure</span> است.

<span dir="ltr">Kubernetes Schema</span> تعیین می‌کند `apiVersion`، `kind`، `metadata` و `spec` چه معنی و چه محدودیتی دارند.

<a id="kubernetes-deployment"></a>
## 48. 🚀 <span dir="ltr">Deployment</span> را خط‌به‌خط بخوانیم

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx
          image: nginx:stable
          ports:
            - containerPort: 80
```

درخت:

```
Deployment
├── metadata
│   └── name
│
└── spec
    ├── replicas: 3
    ├── selector
    │   └── matchLabels
    │       └── app: nginx
    │
    └── template
        ├── metadata
        │   └── labels
        │       └── app: nginx
        │
        └── spec
            └── containers [LIST]
                └── container 1
                    ├── name: nginx
                    ├── image: nginx:stable
                    └── ports [LIST]
                        └── containerPort: 80
```

اینجا اگر `containers` <span dir="ltr">List</span> نبود، نمی‌توانستی چند <span dir="ltr">Container</span> در یک <span dir="ltr">Pod Template</span> تعریف کنی.

<a id="kubernetes-service"></a>
## 49. 🌐 <span dir="ltr">Service</span> را خط‌به‌خط بخوانیم

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  selector:
    app: nginx

  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

بخش `ports` <span dir="ltr">List</span> است چون یک <span dir="ltr">Service</span> می‌تواند چند <span dir="ltr">Port</span> داشته باشد.

`selector` مشخص می‌کند <span dir="ltr">Service</span> دنبال چه <span dir="ltr">Pod</span>هایی بگردد.

<a id="kubernetes-multi"></a>
## 50. 📚 چند <span dir="ltr">Resource</span> در یک فایل

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo
spec:
  replicas: 2
  # ...

---
apiVersion: v1
kind: Service
metadata:
  name: demo
spec:
  # ...
```

`---` دو <span dir="ltr">YAML Document</span> را جدا می‌کند.

اجرا:

```bash
kubectl apply -f app.yaml
```

<a id="ansible"></a>
## 51. 🔧 <span dir="ltr">YAML</span> در <span dir="ltr">Ansible</span>

<span dir="ltr">Ansible Playbook</span>ها هم <span dir="ltr">YAML</span> هستند.

یک <span dir="ltr">Playbook</span> معمولاً یک <span dir="ltr">List</span> از <span dir="ltr">Play</span>هاست:

```yaml
- name: Configure web servers
  hosts: webservers

  tasks:
    - name: Install Nginx
      ansible.builtin.package:
        name: nginx
        state: present
```

از دید <span dir="ltr">YAML</span>:

```
ROOT = List
 │
 └── Play = Mapping
      ├── name
      ├── hosts
      └── tasks = List
           └── Task = Mapping
```

<a id="ansible-playbook"></a>
## 52. 🧪 <span dir="ltr">Playbook</span> سادهٔ واقعی

```yaml
- name: Prepare Docker host
  hosts: docker_servers
  become: true

  tasks:
    - name: Update package cache
      ansible.builtin.apt:
        update_cache: true

    - name: Ensure curl is installed
      ansible.builtin.apt:
        name: curl
        state: present

    - name: Create application directory
      ansible.builtin.file:
        path: /opt/myapp
        state: directory
        mode: "0755"
```

اینجا `tasks` یک <span dir="ltr">List</span> است چون <span dir="ltr">Task</span>ها به ترتیب اجرا می‌شوند.

<a id="ansible-vars"></a>
## 53. 🔁 <span dir="ltr">Variables</span>، <span dir="ltr">List</span> و <span dir="ltr">Loop</span> در <span dir="ltr">Ansible</span>

```yaml
- name: Install tools
  hosts: servers
  become: true

  vars:
    packages:
      - curl
      - git
      - htop

  tasks:
    - name: Install packages
      ansible.builtin.apt:
        name: "{{ item }}"
        state: present
      loop: "{{ packages }}"
```

دوباره:

```
vars
└── packages [LIST]

tasks [LIST]
└── task
    ├── module config
    └── loop
```

`{{ ... }}` مربوط به <span dir="ltr">Templating</span> سیستم <span dir="ltr">Ansible / Jinja</span> است؛ خود <span dir="ltr">YAML</span> نیست.

<a id="yaml-tool-differences"></a>
## 54. 🧠 تفاوت <span dir="ltr">YAML</span> در ابزارهای مختلف <span dir="ltr">DevOps</span>

این چهار فایل همه <span dir="ltr">YAML</span> هستند:

```
compose.yaml
deployment.yaml
playbook.yml
ci.yml
```

ولی نمی‌توانی <span dir="ltr">Key</span>هایشان را جابه‌جا کنی.

**<span dir="ltr">Docker Compose</span>**

```yaml
services:
  api:
    image: my-api
```

**<span dir="ltr">GitHub Actions</span>**

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
```

**<span dir="ltr">Kubernetes</span>**

```yaml
apiVersion: apps/v1
kind: Deployment
```

**<span dir="ltr">Ansible</span>**

```yaml
- hosts: servers
  tasks:
    - name: Test
      ansible.builtin.ping:
```

پس:

```
YAML Syntax      = زبان مشترک
Tool Schema      = دستور زبان تخصصی هر ابزار
```

مثل این: الفبا یکی است ولی فرم بانکی با نسخه پزشک فرق دارد 😄

<a id="yaml-style-guide"></a>
## 55. 🎨 <span dir="ltr">Style Guide</span> پیشنهادی

این‌ها قانون رسمی جهانی نیستند؛ <span dir="ltr">Style</span> پیشنهادی برای فایل‌های خوانای <span dir="ltr">DevOps</span> هستند.

**۱. دو <span dir="ltr">Space</span>**

```yaml
services:
  api:
    image: my-api
```

**۲. <span dir="ltr">Tab</span> نزن**

**۳. <span dir="ltr">Port Mapping</span> را <span dir="ltr">Quote</span> کن**

```yaml
ports:
  - "8080:80"
```

**۴. <span dir="ltr">Version</span> را <span dir="ltr">String</span> کن**

```yaml
app_version: "1.10"
```

**۵. <span dir="ltr">Mapping Style</span> برای <span dir="ltr">Environment</span>های پیچیده**

```yaml
environment:
  NODE_ENV: production
  PORT: "3000"
```

**۶. <span dir="ltr">Comment</span> توضیحی، نه رمان**

خوب:

```yaml
# Internal API; do not publish publicly
api:
```

بد:

```yaml
# در این قسمت ما یک سرویس داریم که داستانش از سال‌ها پیش...
```

**۷. <span dir="ltr">Secret</span> را <span dir="ltr">Commit</span> نکن**

**۸. اسم‌ها <span dir="ltr">Consistent</span>**

```
frontend
backend
database
```

نه:

```
front
backend-service
DB_Thing
```

**۹. فایل بزرگ را <span dir="ltr">Modular</span> کن**

در <span dir="ltr">Compose</span>:

```
compose.yaml
compose.dev.yaml
compose.prod.yaml
```

در <span dir="ltr">Ansible</span>:

```
roles/
group_vars/
playbooks/
```

در <span dir="ltr">Kubernetes</span>:

```
deployment.yaml
service.yaml
configmap.yaml
```

<a id="yaml-debug"></a>
## 56. 🕵️ <span dir="ltr">Debug</span> کردن خطاهای <span dir="ltr">YAML</span>

وقتی خطایی مثل این دیدی:

```
mapping values are not allowed here
```

یا:

```
did not find expected key
```

این مراحل را برو:

```
1. همان خط را نگاه کن.
2. دو سه خط بالاتر را هم نگاه کن.
3. Indentation را چک کن.
4. Tab را چک کن.
5. : و - را چک کن.
6. Quote بازمانده را چک کن.
7. List / Mapping را تشخیص بده.
8. بعد Schema ابزار را بررسی کن.
```

خیلی وقت‌ها <span dir="ltr">Error</span> روی <span dir="ltr">Line 20</span> گزارش می‌شود ولی اشتباه واقعی در <span dir="ltr">Line 18</span> است.

**تکنیک درخت**

این:

```yaml
services:
  api:
    ports:
      - "3000:3000"
```

را روی کاغذ تبدیل کن:

```
services
└── api
    └── ports
        └── item
```

اگر درختی که در ذهن داری با <span dir="ltr">Indentation</span> فایل یکی نیست، مشکل پیدا شد.

<a id="yaml-validation-tools"></a>
## 57. 🛡️ دستورهای <span dir="ltr">Validation</span> که باید حفظ باشی

**<span dir="ltr">Docker Compose</span>**

```bash
docker compose config
```

فقط <span dir="ltr">Validate</span>:

```bash
docker compose config -q
```

**<span dir="ltr">Kubernetes</span>**

<span dir="ltr">Client-side Dry Run</span>:

```bash
kubectl apply \
  --dry-run=client \
  -f deployment.yaml
```

**<span dir="ltr">Ansible</span>**

<span dir="ltr">Syntax Check</span>:

```bash
ansible-playbook \
  --syntax-check \
  playbook.yml
```

<span dir="ltr">Check Mode</span>:

```bash
ansible-playbook \
  --check \
  playbook.yml
```

**<span dir="ltr">GitHub Actions</span>**

<span dir="ltr">VS Code Extension</span>ها و <span dir="ltr">Linter</span>ها کمک زیادی می‌کنند؛ اما <span dir="ltr">Schema</span> نهایی <span dir="ltr">Workflow</span> را <span dir="ltr">GitHub</span> تفسیر می‌کند.

🎯 اصل <span dir="ltr">DevOps</span>: <span dir="ltr">Configuration</span> را قبل از <span dir="ltr">Production</span> تا جایی که <span dir="ltr">Tool</span> اجازه می‌دهد <span dir="ltr">Parse / Validate / Dry-run</span> کن.

<a id="yaml-practice"></a>
## 58. 🧪 تمرین‌های مرحله‌ای

**تمرین 1 — <span dir="ltr">Mapping</span>**

این داده را <span dir="ltr">YAML</span> کن:

```
name = api
port = 3000
debug = true
```

<details>
<summary>✅ جواب</summary>

```yaml
name: api
port: 3000
debug: true
```

</details>

**تمرین 2 — <span dir="ltr">List</span>**

سه <span dir="ltr">Environment</span> بنویس: `development`، `staging`، `production`

<details>
<summary>✅ جواب</summary>

```yaml
environments:
  - development
  - staging
  - production
```

</details>

**تمرین 3 — <span dir="ltr">Nested Mapping</span>**

ساختار:

```
database
  host = db
  port = 5432
```

<details>
<summary>✅ جواب</summary>

```yaml
database:
  host: db
  port: 5432
```

</details>

**تمرین 4 — <span dir="ltr">Compose</span>**

یک <span dir="ltr">Nginx</span> بنویس که: `Service name = web`، `Image = nginx:alpine`، `Host port = 8080`، `Container port = 80`

<details>
<summary>✅ جواب</summary>

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

</details>

**تمرین 5 — <span dir="ltr">Volume</span>**

به <span dir="ltr">PostgreSQL</span> یک <span dir="ltr">Named Volume</span> بده.

<details>
<summary>✅ جواب</summary>

```yaml
services:
  db:
    image: postgres:17
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

</details>

**تمرین 6 — <span dir="ltr">GitHub Actions</span>**

<span dir="ltr">Workflow</span> بساز که روی <span dir="ltr">Push</span> به `main` یک دستور `npm test` بزند.

<details>
<summary>✅ جواب</summary>

```yaml
name: Test

on:
  push:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Run tests
        run: npm test
```

</details>

<a id="yaml-quiz"></a>
## 59. 🎯 آزمون سریع

**سؤال 1**

این چیست؟

```yaml
ports:
  - "80:80"
  - "443:443"
```

جواب:

```
ports = Mapping Key
مقدار ports = Sequence/List
هر Port Mapping = Scalar داخل List
```

**سؤال 2**

این چیست؟

```yaml
database:
  host: db
  port: 5432
```

جواب:

```
database = Mapping
host و port = Keyهای داخل database
```

**سؤال 3**

`-` یعنی؟ → <span dir="ltr">List Item</span>

**سؤال 4**

<span dir="ltr">Indentation</span> یعنی؟ → ساختار و <span dir="ltr">Parent/Child</span> بودن

**سؤال 5**

`services` جزو <span dir="ltr">YAML</span> است؟ ❌ نه — `services` مربوط به <span dir="ltr">Docker Compose Schema</span> است.

**سؤال 6**

`${APP_PORT}` جزو <span dir="ltr">YAML</span> است؟ ❌ نه — در <span dir="ltr">Compose</span> این <span dir="ltr">Syntax</span> مربوط به <span dir="ltr">Variable Interpolation</span> خود <span dir="ltr">Compose</span> است.

**سؤال 7**

`${{ secrets.TOKEN }}` جزو <span dir="ltr">YAML</span> است؟ ❌ نه — این <span dir="ltr">GitHub Actions Expression</span> است.

اگر این سه سؤال آخر را فهمیدی، از خیلی از مبتدی‌هایی که فقط <span dir="ltr">Syntax</span> حفظ می‌کنند جلو افتادی 😄

<a id="yaml-cheatsheet"></a>
## 60. 🧾 <span dir="ltr">Cheat Sheet</span> نهایی

<span dir="ltr">Key / Value</span>

```yaml
name: api
```

<span dir="ltr">Nested Mapping</span>

```yaml
api:
  port: 3000
```

<span dir="ltr">List</span>

```yaml
ports:
  - 80
  - 443
```

<span dir="ltr">List of Mappings</span>

```yaml
users:
  - name: Ali
    role: admin

  - name: Sara
    role: developer
```

<span dir="ltr">Comment</span>

```yaml
# comment
name: api
```

<span dir="ltr">String</span>

```yaml
name: "api"
```

<span dir="ltr">Boolean</span>

```yaml
enabled: true
```

<span dir="ltr">Null</span>

```yaml
value: null
```

<span dir="ltr">Multiline Literal</span>

```yaml
script: |
  echo one
  echo two
```

<span dir="ltr">Folded Text</span>

```yaml
message: >
  this becomes
  mostly one line
```

<span dir="ltr">Anchor</span>

```yaml
defaults: &defaults
  retries: 3
```

<span dir="ltr">Alias</span>

```yaml
config: *defaults
```

<span dir="ltr">Merge</span>

```yaml
service:
  <<: *defaults
  name: api
```

<span dir="ltr">Multiple Documents</span>

```yaml
---
name: one

---
name: two
```

<span dir="ltr">Docker Compose</span>

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

<span dir="ltr">GitHub Actions</span>

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
```

<span dir="ltr">Kubernetes</span>

```yaml
apiVersion: v1
kind: Service
```

<span dir="ltr">Ansible</span>

```yaml
- hosts: servers
  tasks:
    - name: Ping
      ansible.builtin.ping:
```

<a id="yaml-roadmap"></a>
## 61. 🗺️ نقشهٔ راه ادامه

بعد از این داکیومنت پیشنهاد می‌کنم این ترتیب را بروی:

```
YAML Basics
    │
    ▼
Docker Compose
    │
    ├── services
    ├── ports
    ├── volumes
    ├── networks
    ├── environment
    └── healthcheck
    │
    ▼
.env / Secrets
    │
    ▼
Compose multi-file
    │
    ▼
GitHub Actions
    │
    ▼
CI/CD
    │
    ├── build
    ├── test
    ├── image push
    └── deploy
    │
    ▼
Ansible
    │
    ▼
Kubernetes
```

مهم‌تر از همه:

```
Syntax حفظ نکن
Structure را بفهم
        +
Schema ابزار را بخوان
        +
قبل از Deploy Validate کن
```

<a id="yaml-sources"></a>
## 62. 📖 منابع رسمی

برای بخش <span dir="ltr">Syntax</span> عمومی <span dir="ltr">YAML</span>:

- <span dir="ltr">YAML 1.2.2 Specification</span>: https://yaml.org/spec/1.2.2/

برای <span dir="ltr">Docker Compose</span>:

- <span dir="ltr">Compose File Reference</span>: https://docs.docker.com/reference/compose-file/
- <span dir="ltr">Compose Services</span>: https://docs.docker.com/reference/compose-file/services/
- <span dir="ltr">Environment Variables / Interpolation</span>: https://docs.docker.com/compose/how-tos/environment-variables/
- <span dir="ltr">Compose Fragments / Anchors</span>: https://docs.docker.com/reference/compose-file/fragments/
- <span dir="ltr">Compose Extensions</span>: https://docs.docker.com/reference/compose-file/extension/
- <span dir="ltr">Merge Compose Files</span>: https://docs.docker.com/reference/compose-file/merge/
- <span dir="ltr">docker compose config</span>: https://docs.docker.com/reference/cli/docker/compose/config/

برای <span dir="ltr">GitHub Actions</span>:

- <span dir="ltr">Workflow Syntax</span>: https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax

برای <span dir="ltr">Kubernetes</span>:

- <span dir="ltr">Deployments</span>: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
- <span dir="ltr">Services</span>: https://kubernetes.io/docs/concepts/services-networking/service/

برای <span dir="ltr">Ansible</span>:

- <span dir="ltr">Using Ansible Playbooks</span>: https://docs.ansible.com/projects/ansible/latest/playbook_guide/
- <span dir="ltr">YAML Syntax</span>: https://docs.ansible.com/projects/ansible/latest/reference_appendices/YAMLSyntax.html

## ❤️ حرف آخر

قرار نیست وقتی یک فایل ۱۵۰ خطی <span dir="ltr">YAML</span> دیدی همهٔ <span dir="ltr">Key</span>ها را از حفظ باشی.

مهارتی که واقعاً می‌خواهیم این است:

```
یک YAML پیچیده می‌بینم
       │
       ▼
List و Mapping را تشخیص می‌دهم
       │
       ▼
ساختارش را به درخت تبدیل می‌کنم
       │
       ▼
می‌فهمم کدام بخش YAML است
و کدام بخش Schema ابزار
       │
       ▼
Documentation همان ابزار را چک می‌کنم
       │
       ▼
Validate / Dry-run
       │
       ▼
Deploy
```

اگر به این نقطه برسی، دیگر `compose.yaml` یا `deployment.yaml` برایت «دیوار حروف عجیب» نیست؛ فقط یک <span dir="ltr">Configuration Tree</span> است که باید آرام‌آرام بخوانیش 😎

🧩 <span dir="ltr">Happy YAMLing</span> — بدون دعوا با <span dir="ltr">Space</span>ها!

</div>