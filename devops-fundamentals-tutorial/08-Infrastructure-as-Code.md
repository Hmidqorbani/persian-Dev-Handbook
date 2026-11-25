# بخش ۰۸: زیرساخت به‌عنوان کد

**مدت یادگیری**: ۳ تا ۴ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**:
- بخش ۰۶ و ۰۷
- آشنایی مقدماتی با Linux و شبکه

---
## فهرست محتوا
- [فصل ۴۱: محیط به‌عنوان کد](#فصل-۴۱-محیط-بهعنوان-کد)
- [فصل ۴۲: اصول Infrastructure as Code](#فصل-۴۲-اصول-infrastructure-as-code)
- [فصل ۴۳: زیرساخت تغییرناپذیر](#فصل-۴۳-زیرساخت-تغییرناپذیر)
- [فصل ۴۴: Provisioning در برابر Configuration Management](#فصل-۴۴-provisioning-در-برابر-configuration-management)
- [فصل ۴۵: برابری محیط‌ها](#فصل-۴۵-برابری-محیطها)
- [فصل ۴۶: مدیریت State و محیط‌های چندگانه](#فصل-۴۶-مدیریت-state-و-محیطهای-چندگانه)

---

## فصل ۴۱: محیط به‌عنوان کد

### مقدمه

قبل از IaC، سرورها **حیوانات خانگی** بودند: هر کدام اسم داشت، دستی ساخته می‌شد، دستی نگهداری می‌شد، و اگر می‌مرد، کسی نمی‌دانست چطور بازسازی کند.

IaC آن‌ها را به ** скот** تبدیل می‌کند: یکسان، قابل جایگزینی، تولیدشده از کد.

این تغییر، یکی از بنیادی‌ترین تحولات DevOps است.

### مفاهیم اساسی

**حیوانات خانگی در برابر скот:**

| معیار | Pet | Cattle |
|---|---|---|
| نام | `web-01` | `web-a7f3` |
| ساخت | دستی | خودکار از کد |
| تعمیر | دستی، با احتیاط | حذف و جایگزینی |
| یکتا | ✅ | ❌ (یکسان) |
| مقیاس | ❌ | ✅ |

> **تشبیه روزمره**: حیوان خانگی را نمی‌کشی و دوباره می‌سازی؛ دام را می‌فروشی و دام جدید می‌خری. سرور در IaC، دام است.

**چهار مزیت IaC:**

| مزیت | توضیح |
|---|---|
| **تکرارپذیری** | همان کد، همان محیط |
| **سرعت** | دقیقه به‌جای روز |
| **مستندسازی زنده** | کد = مستند |
| **کنترل نسخه** | تاریخ تغییرات |

**«مستندسازی زنده» — مهم‌ترین مزیت پنهان:**

```
مستند سنتی: "سرور با ۴GB RAM، Ubuntu 22.04، nginx 1.24، ..."
  → بعد از ۶ ماه: کهنه، اشتباه

IaC: فایل terraform
  → همیشه دقیق، چون واقعاً اجرا می‌شود
```

### مثال‌های عملی

**قبل و بعد — ساخت یک محیط:**

**قبل (دستی):**
```
۱. در کنسول ابر، ۳ ماشین بساز (۱۵ دقیقه)
۲. SSH به هر کدام
۳. نصب nginx، python، ...
۴. کپی فایل config
۵. تنظیم firewall
۶. ساخت DB
۷. تنظیم شبکه
⏱️ ۴ ساعت، با ۵ اشتباه کوچک
```

**بعد (IaC):**
```
۱. terraform apply
⏱️ ۸ دقیقه، بدون اشتباه
```

**و مهم‌تر**: می‌توانی همان محیط را **دوباره** بسازی. یا ۱۰ نسخه.

**ساختار یک پروژه IaC:**

```
infra/
├── modules/
│   ├── network/
│   ├── compute/
│   └── database/
├── environments/
│   ├── staging/
│   │   ├── main.tf
│   │   └── terraform.tfvars
│   └── production/
│       ├── main.tf
│       └── terraform.tfvars
└── README.md
```

**نمونه مفهوم (شبه‌کد):**

```hcl
# environments/staging/main.tf
module "network" {
  source      = "../../modules/network"
  cidr_block  = "10.10.0.0/16"
  environment = "staging"
}

module "compute" {
  source        = "../../modules/compute"
  instance_type = "t3.medium"
  count         = 2
  subnet_ids    = module.network.private_subnet_ids
}

module "database" {
  source        = "../../modules/database"
  engine        = "postgres"
  version       = "16.4"
  instance_class = "db.t3.medium"
  multi_az      = false     # staging نیازی ندارد
}
```

**نکته**: همان ماژول، پارامتر متفاوت. staging کوچک، production بزرگ.

### مقایسه و الگوهای مختلف

**IaC در برابر ClickOps:**

| معیار | IaC | ClickOps |
|---|---|---|
| تکرارپذیری | ✅✅ | ❌ |
| سرعت | ✅ | ❌ |
| audit | ✅ | ❌ |
| دانش | ✅ در کد | ❌ در سر کسی |
| disaster recovery | ✅ دقیقه | ❌ روز |

**سه سطح IaC:**

| سطح | چه چیزی |
|---|---|
| ۱ | فقط ماشین‌ها |
| ۲ | + شبکه، DB، storage |
| ۳ | ✅ همه‌چیز (شامل monitoring، IAM) |

**Declarative در برابر Imperative:**

| معیار | Declarative | Imperative |
|---|---|---|
| چه می‌گویی | «چه می‌خواهم» | «چطور بساز» |
| idempotent | ✅ | ⚠️ |
| مثال | Terraform | اسکریپت bash |
| مناسب | ✅ زیرساخت | کار خاص |

### اشکال شایع

**اشکال ۱: IaC برای بخشی از زیرساخت**

ماشین‌ها با Terraform، ولی DB دستی. نیمه‌کاره.

**اشکال ۲: drift**

کسی در کنسول چیزی عوض کرد. کد و واقعیت متفاوت. باید دوره‌ای `plan` بزنی.

**اشکال ۳: IaC بدون code review**

تغییر زیرساخت بدون review = ریسک. مثل کد رفتار کن.

**اشکال ۴: state در مخزن Git**

state شامل رمز است. هرگز commit نکن.

**اشکال ۵: ماژول غول‌پیکر**

یک ماژول ۵۰۰۰ خطی برای همه‌چیز. به ماژول‌های کوچک بشکن.

**اشکال ۶: فراموش کردن حذف**

`terraform destroy` برای staging، ولی production فراموش می‌شود. منابع یتیم.

### نکات مهم (Tips & Tricks)

- **همه‌چیز را کد کن** — حتی IAM و monitoring.
- **drift را دوره‌ای چک کن** (`plan` در CI).
- **state را در backend امن** (نه Git).
- **ماژول کوچک** بساز.
- **تغییر زیرساخت، با MR و review.**
- **`destroy` را برای محیط موقت** خودکار کن.

### خلاصه فصل

- Pet به Cattle: یکسان، قابل جایگزینی.
- چهار مزیت: تکرارپذیری، سرعت، مستند زنده، کنترل نسخه.
- Declarative بهتر از Imperative برای زیرساخت.
- drift دشمن IaC است.
- state را امن نگه دار.
- تغییر زیرساخت = تغییر کد (با review).

---

## فصل ۴۲: اصول Infrastructure as Code

### مقدمه

IaC فقط نوشتن فایل config نیست. پنج اصل دارد که اگر رعایت نشوند، IaC به یک **اسکریپت bash بزرگ با ظاهر مدرن** تبدیل می‌شود.

### مفاهیم اساسی

**پنج اصل:**

| اصل | معنا |
|---|---|
| **Declarative** | چه، نه چطور |
| **Idempotent** | اجرای دوباره، همان نتیجه |
| **Versioned** | در Git |
| **Reviewed** | مثل کد |
| **Tested** | اعتبارسنجی قبل از اعمال |

**Idempotency — چرا حیاتی است:**

```
غیر idempotent:
  اجرا ۱: ۳ سرور ساخته شد
  اجرا ۲: ۳ سرور دیگر ساخته شد (مجموع ۶)
  اجرا ۳: ۹ سرور

idempotent:
  اجرا ۱: ۳ سرور
  اجرا ۲: ۳ سرور (هیچ تغییری)
  اجرا ۳: ۳ سرور
```

بدون idempotency، نمی‌توانی دوباره اجرا کنی — پس در عمل استفاده نمی‌شود.

**Declarative در برابر Imperative — مثال:**

```bash
# ❌ Imperative: چطور
apt-get update
apt-get install -y nginx
systemctl enable nginx
systemctl start nginx
# اجرای دوباره: خطا یا رفتار غیرمنتظره

# ✅ Declarative: چه
package:
  nginx:
    ensure: present
service:
  nginx:
    ensure: running
    enable: true
# اجرای دوباره: همان وضعیت
```

**سه سطح اعتبارسنجی:**

| سطح | ابزار | چه می‌گیرد |
|---|---|---|
| Syntax | `fmt`، `validate` | خطای نگارشی |
| Plan | `plan` / `--check` | تغییرات مورد انتظار |
| Policy | OPA، Sentinel | نقض سیاست |

### مثال‌های عملی

**جریان کار IaC — پنج گام:**

```
۱. تغییر در branch
   ↓
۲. CI: fmt + validate
   ↓
۳. CI: plan (نمایش تغییرات)
   ↓
۴. review: کسی plan را می‌خواند
   ↓
۵. apply (بعد از merge)
```

**نکته**: **plan را در MR بگذار.** reviewer باید ببیند چه چیزی ساخته/حذف می‌شود.

**نمونه plan — چرا مهم است:**

```
Plan: 3 to add, 1 to change, 1 to destroy

+ aws_instance.web[2]
    ami: "ami-abc123"
    instance_type: "t3.medium"

~ aws_security_group.web
    ingress.0.cidr_blocks: "0.0.0.0/0" → "10.0.0.0/16"

- aws_instance.legacy
    (حذف می‌شود!)
```

**اگر plan را نخوانی**: ممکن است یک DB حذف شود.

**قاعده «هیچ حذف غیرمنتظره»:**

```
اگر plan چیزی را حذف می‌کند و انتظارش را نداشتی:
  ⛔ متوقف شو، بررسی کن
```

**Policy as Code — جلوگیری از خطا:**

```rego
# policy/deny_public_buckets.rego
package main

deny[msg] {
    resource := input.resource_changes[_]
    resource.type == "aws_s3_bucket_public_access_block"
    not resource.change.after.block_public_acls
    msg := sprintf("S3 bucket %s باید public access را بلاک کند", [resource.address])
}
```

```bash
# در CI
conftest test plan.json -p policy/
```

**مزیت**: حتی اگر کسی اشتباه بنویسد، policy جلویش را می‌گیرد.

### مقایسه و الگوهای مختلف

**سه سطح تست IaC:**

| سطح | هزینه | چه می‌گیرد |
|---|---|---|
| Static | ✅ کم | خطای نگارشی، سیاست |
| Plan | متوسط | تغییرات |
| Integration | ❌ زیاد | رفتار واقعی |

**توصیه**: static + plan در هر MR، integration هفتگی.

**Monolithic در برابر Modular:**

| معیار | یک فایل | ماژول |
|---|---|---|
| شروع | ✅ ساده | متوسط |
| مقیاس | ❌ | ✅ |
| آستانه | ~۵۰۰ خط | بعد از آن |

**سه مدل سازمان‌دهی:**

| مدل | توضیح |
|---|---|
| بر اساس محیط | `env/staging`، `env/prod` |
| بر اساس سرویس | `svc/auth`، `svc/billing` |
| بر اساس لایه | `network`، `compute`، `data` |

**توصیه**: ترکیب — ماژول بر اساس لایه، سازمان‌دهی بر اساس محیط.

### اشکال شایع

**اشکال ۱: IaC غیر idempotent**

اسکریپت bash که دوبار اجرا نمی‌شود. از ابزار declarative استفاده کن.

**اشکال ۲: plan را کسی نمی‌خواند**

تصادفاً DB حذف شد. plan در MR، با تأکید روی حذف.

**اشکال ۳: بدون policy**

کسی bucket عمومی ساخت. policy as code.

**اشکال ۴: apply دستی از لپ‌تاپ**

state قفل می‌شود، نسخه‌ها متفاوت. apply در CI.

**اشکال ۵: ماژول بدون نسخه**

`source = "git::...//module"` بدون ref. تغییر در ماژول، همه‌جا را می‌شکند.

**اشکال ۶: تست نکردن در محیط موقت**

مستقیم روی production. همیشه staging اول.

### نکات مهم (Tips & Tricks)

- **plan در MR** — الزامی.
- **حذف را برجسته کن** — رنگ قرمز در plan.
- **policy as code** برای جلوگیری از خطای رایج.
- **apply در CI**، نه لپ‌تاپ.
- **ماژول با `ref` pin.**
- **staging قبل از production.**

### خلاصه فصل

- پنج اصل: declarative، idempotent، versioned، reviewed، tested.
- plan را در MR بگذار و بخوان.
- policy as code از خطا جلوگیری می‌کند.
- apply در CI، نه لپ‌تاپ.
- ماژول را pin کن.
- staging اول.

---

## فصل ۴۳: زیرساخت تغییرناپذیر

### mمقدمه

**Immutable Infrastructure** یعنی: سرور را **تغییر نمی‌دهی**؛ سرور جدید می‌سازی و قدیمی را حذف می‌کنی.

این تغییر، کلاس کاملی از باگ‌ها را حذف می‌کند: drift، «فقط روی این سرور کار می‌کند»، و پیکربندی ناهمگون.

### مفاهیم اساسی

**Mutable در برابر Immutable:**

| معیار | Mutable | Immutable |
|---|---|---|
| تغییر | روی سرور موجود | سرور جدید |
| drift | ✅ زیاد | ❌ صفر |
| rollback | ❌ سخت | ✅ آسان |
| زمان راه‌اندازی | ✅ سریع | متوسط |
| قابلیت پیش‌بینی | ❌ | ✅✅ |

**مثال mutable:**

```
سرور web-01 (۳ سال عمر):
  - ۴۷ بار وصله امنیتی دستی
  - ۱۲ فایل config که کسی نمی‌داند چرا
  - nginx 1.18 (چون upgrade شکسته بود)
  - یک اسکریپت در /root که کسی نمی‌شناسد
```

**مثال immutable:**

```
web-a7f3 (ساخته‌شده از image v1.4.2):
  - دقیقاً همان چیزی که در image تعریف شده
  - ۲ ساعت عمر
  - فردا جایگزین می‌شود
```

**سه سطح immutability:**

| سطح | مثال |
|---|---|
| **Image** | ایمیج Docker immutable |
| **VM** | image ماشین + جایگزینی |
| **Container** | Pod جدید به‌جای تغییر |

**«گاوهای مقدس» در برابر «گاوهای کشتارگاهی»:**

```
Mutable:  سرور مقدس است، دست نمی‌زنیم، می‌ترسیم
Immutable: سرور موقت است، جایگزین می‌شود، نمی‌ترسیم
```

### مثال‌های عملی

**چرخه immutability در Kubernetes:**

```
Deployment با image: app:v1.4.2
    ↓ (تغییر نسخه)
Deployment با image: app:v1.4.3
    ↓
Podهای جدید ساخته می‌شوند (v1.4.3)
Podهای قدیم حذف می‌شوند
    ↓
هیچ Podی «تغییر» نکرد — همه جایگزین شدند
```

**مزیت**: اگر v1.4.3 مشکل داشت، `kubectl rollout undo` → Podهای v1.4.2 برمی‌گردند. **۳۰ ثانیه.**

**چرخه immutability برای VM:**

```
۱. Packer: ساخت image از template
۲. Registry: ذخیره image با نسخه
۳. Terraform: ساخت VM از image جدید
۴. Load balancer: اضافه کردن VM جدید
۵. حذف VM قدیمی
```

**نکته**: هیچ SSH به سرور برای تغییر. تغییر = image جدید.

**«Phoenix Server» — تست نهایی:**

```
سؤال: اگر این سرور همین حالا بمیرد، چقدر طول می‌کشد تا جایگزین شود؟

Mutable:  ۳ روز (کسی نمی‌داند چطور ساخته شده)
Immutable: ۵ دقیقه (از image ساخته می‌شود)
```

**این تست، بهترین سنجش بلوغ IaC است.**

**ترکیب با immutable data:**

| نوع داده | Immutable؟ |
|---|---|
| کد | ✅ (image) |
| Config | ✅ (image یا ConfigMap) |
| داده کاربر | ❌ (باید بماند) |
| لاگ | ❌ (append-only) |

**نکته**: داده را نمی‌توان immutable کرد. برای آن، volume پایدار و backup.

### مقایسه و الگوهای مختلف

**سه مدل مدیریت سرور:**

| مدل | توضیح | مناسب |
|---|---|---|
| Mutable (pets) | تغییر روی سرور | ❌ |
| Immutable (cattle) | جایگزینی | ✅ |
| Hybrid | کد immutable، داده mutable | ✅✅ |

**Immutable در برابر Configuration Management:**

| معیار | Immutable | Config Mgmt |
|---|---|---|
| drift | ❌ صفر | ⚠️ ممکن |
| سرعت تغییر | متوسط | ✅ سریع |
| rollback | ✅ آسان | ❌ سخت |
| مناسب | ✅ مدرن | legacy |

**سه سطح بلوغ:**

| سطح | ویژگی |
|---|---|
| ۱ | SSH به سرور برای تغییر |
| ۲ | IaC برای ساخت، SSH برای تغییر |
| ۳ | ✅ هیچ SSH، فقط جایگزینی |

### اشکال شایع

**اشکال ۱: «فقط این یک تغییر کوچک»**

SSH زدی و یک فایل عوض کردی. drift شروع شد. تغییر = image جدید.

**اشکال ۲: image بدون نسخه**

`app:latest` → نمی‌دانی چه چیزی روی سرور است.

**اشکال ۳: داده روی container**

container حذف شد، داده رفت. volume پایدار.

**اشکال ۴: زمان راه‌اندازی طولانی**

image ۲GB، ۵ دقیقه boot. image را کوچک کن.

**اشکال ۵: secret در image**

image در registry، secret لو رفت. secret در runtime تزریق کن.

**اشکال ۶: فراموش کردن state**

immutable بودن سرور، state (مثلاً session) را نمی‌پوشاند. session مشترک لازم.

### نکات مهم (Tips & Tricks)

- **تغییر = image جدید**، نه SSH.
- **`Phoenix test`** را ماهانه اجرا کن.
- **داده را از compute جدا کن.**
- **image را کوچک نگه دار** — boot سریع.
- **secret در runtime**، نه image.
- **state را در سرویس مشترک** بگذار.

### خلاصه فصل

- Immutable: سرور جدید، نه تغییر سرور.
- drift صفر، rollback آسان.
- Phoenix test بهترین سنجش است.
- داده و compute را جدا کن.
- هیچ SSH برای تغییر.
- secret در runtime.

---

## فصل ۴۴: Provisioning در برابر Configuration Management

### مقدمه

دو فعالیت متفاوت که اغلب قاطی می‌شوند:

| فعالیت | سؤال |
|---|---|
| **Provisioning** | «چه منابعی بسازم؟» |
| **Configuration Management** | «داخل منابع چه بگذارم؟» |

فهم تفاوت، انتخاب ابزار و معماری را روشن می‌کند.

### مفاهیم اساسی

**تفکیک:**

| Provisioning | Configuration Management |
|---|---|
| ساخت VM، شبکه، DB | نصب نرم‌افزار، فایل config |
| لایه زیرساخت | لایه سیستم |
| Terraform، CloudFormation | Ansible، Puppet، Chef |
| یک‌بار (ساخت) | مکرر (نگهداری) |

**استعاره:**

```
Provisioning  = ساخت خانه (دیوار، سقف، لوله)
Configuration = چیدن مبلمان و وسایل
```

**سه لایه معماری:**

```
لایه ۳: Application    (کد، image)
لایه ۲: Configuration  (Ansible، cloud-init)
لایه ۱: Infrastructure (Terraform)
```

**نکته**: با کانتینر، لایه ۲ کوچک‌تر می‌شود — بیشتر config در image است.

### مثال‌های عملی

**جریان کامل — از صفر تا سرویس:**

```
۱. Terraform: ساخت VPC، subnet، security group
      ↓
۲. Terraform: ساخت ۳ VM از image
      ↓
۳. cloud-init: تنظیم اولیه (کاربر، SSH key)
      ↓
۴. Ansible: نصب docker، تنظیم daemon
      ↓
۵. Docker/K8s: اجرای image اپلیکیشن
      ↓
۶. Ansible: تنظیم nginx، certificate
```

**نکته**: هر لایه ابزار خودش، مسئولیت خودش.

**cloud-init — نمونه:**

```yaml
#cloud-config
users:
  - name: deploy
    ssh_authorized_keys:
      - ssh-ed25519 AAAA... deploy@ci

packages:
  - docker.io
  - nginx

runcmd:
  - systemctl enable --now docker
  - systemctl enable --now nginx
```

**مزیت**: VM با تنظیمات اولیه بالا می‌آید، بدون SSH.

**Ansible — نمونه (idempotent):**

```yaml
---
- name: تنظیم سرور اپلیکیشن
  hosts: app_servers
  become: true
  tasks:
    - name: نصب docker
      apt:
        name: docker.io
        state: present
        update_cache: true

    - name: فعال‌سازی سرویس docker
      service:
        name: docker
        state: started
        enabled: true

    - name: کپی فایل تنظیمات
      template:
        src: app.env.j2
        dest: /etc/app/env
        mode: "0600"
      notify: restart app
```

**چرا idempotent است**: `state: present` یعنی «اگر نبود، نصب کن؛ اگر بود، کاری نکن».

### مقایسه و الگوهای مختلف

**Terraform در برابر Ansible:**

| معیار | Terraform | Ansible |
|---|---|---|
| نوع | Declarative | Declarative (اغلب) |
| scope | منابع ابر | سیستم‌عامل |
| state | ✅ دارد | ❌ ندارد |
| مناسب | provisioning | configuration |

**نکته**: Terraform state دارد (می‌داند چه ساخته)، Ansible ندارد (هر بار چک می‌کند).

**سه رویکرد به configuration:**

| رویکرد | توضیح | مناسب |
|---|---|---|
| **Agent-based** | Puppet، Chef (agent روی سرور) | مقیاس بزرگ |
| **Agentless** | Ansible (SSH) | ✅ اکثر |
| **Image-based** | Packer، Docker | ✅✅ مدرن |

**توصیه مدرن**: image-based برای اپ، agentless برای زیرساخت.

**سه سطح بلوغ:**

| سطح | ویژگی |
|---|---|
| ۱ | همه‌چیز دستی |
| ۲ | Terraform برای infra، Ansible برای config |
| ۳ | ✅ image-based، config در build |

### اشکال شایع

**اشکال ۱: Terraform برای configuration**

«با Terraform nginx نصب کنیم» — با `remote-exec`. شکننده و غیر idempotent.

**اشکال ۲: Ansible برای provisioning**

«با Ansible VM بسازیم» — ممکن ولی state ندارد. Terraform بهتر.

**اشکال ۳: cloud-init غول‌پیکر**

۵۰۰ خط bash در cloud-init. به image یا Ansible منتقل کن.

**اشکال ۴: configuration drift**

Ansible هر شب اجرا نمی‌شود، سرور drift می‌کند. با immutable حل می‌شود.

**اشکال ۵: secret در Ansible playbook**

رمز در Git. از vault یا متغیر محیطی.

**اشکال ۶: لایه‌ها قاطی**

Terraform شبکه می‌سازد و nginx نصب می‌کند. تفکیک مسئولیت.

### نکات مهم (Tips & Tricks)

- **Terraform برای ساخت، Ansible برای تنظیم.**
- **image-based را ترجیح بده** — کمتر config، کمتر drift.
- **cloud-init را کوچک نگه دار.**
- **secret در vault**، نه playbook.
- **هر لایه، ابزار خودش.**
- **idempotency را تست کن** — دوبار اجرا.

### خلاصه فصل

- Provisioning = ساخت منابع؛ Configuration = تنظیم داخل.
- Terraform برای اولی، Ansible برای دومی.
- سه لایه: infrastructure، configuration، application.
- image-based مدرن‌ترین رویکرد است.
- idempotency در هر دو لازم است.
- لایه‌ها را قاطی نکن.

---

## فصل ۴۵: برابری محیط‌ها

### مقدمه

**«روی سیستم من کار می‌کرد»** — معروف‌ترین جمله در مهندسی نرم‌افزار. علتش یک چیز است: **محیط‌ها یکسان نیستند.**

این فصل اصل برابری محیط‌ها (Parity) را می‌آورد.

### مفاهیم اساسی

**چهار بُعد برابری (از Twelve-Factor App):**

| بُعد | معنا |
|---|---|
| **زمان** | فاصله بین deploy و توسعه کم |
| **پرسنل** | همان کسی که نوشت، deploy می‌کند |
| **ابزار** | همان ابزار در همه محیط‌ها |
| **کتابخانه** | همان نسخه dependency |

**سه سطح اختلاف محیط:**

| سطح | مثال | اثر |
|---|---|---|
| **OS** | Ubuntu 20 vs 22 | کتابخانه‌های متفاوت |
| **Runtime** | Python 3.10 vs 3.12 | رفتار متفاوت |
| **Config** | متغیر متفاوت | باگ پنهان |
| **Data** | داده متفاوت | مقیاس متفاوت |

**نکته**: اختلاف در **کد** کم است؛ اختلاف در **محیط** زیاد.

**چرا parity مهم است:**

```
اگر staging شبیه production نباشد:
  staging سبز → production خراب
  → staging بی‌فایده است
```

**قاعده**: محیطی که شبیه production نیست، **تست** نیست.

### مثال‌های عملی

**اختلاف رایج و راه‌حل:**

| اختلاف | راه‌حل |
|---|---|
| نسخه Python | Docker image |
| نسخه dependency | lockfile |
| متغیر محیطی | config مدیریت‌شده |
| OS package | همان base image |
| DB نسخه | همان image در همه محیط‌ها |
| داده | داده ساختگی شبیه production |

**الگوی «همان image در همه محیط‌ها»:**

```
build (یک‌بار)
    ↓
image: app:a1b2c3d
    ↓
├── staging  → همان image، config staging
└── production → همان image، config production
```

**نکته**: فقط **config** متفاوت است، نه **artifact**.

**مدیریت config — سه سطح:**

| سطح | مثال |
|---|---|
| **Build-time** | نسخه زبان (در image) |
| **Deploy-time** | URL دیتابیس (متغیر محیطی) |
| **Runtime** | feature flag |

**قاعده**: config را در **deploy-time** بگذار، نه build-time.

**داده ساختگی شبیه production:**

```
❌ staging با ۱۰ رکورد
✅ staging با ۱۰۰٬۰۰۰ رکورد (anonymized)

چرا: مشکل N+1، index، صفحه‌بندی — فقط در مقیاس ظاهر می‌شوند
```

**نکته امنیتی**: داده production را **anonymize** کن، هرگز مستقیم کپی نکن.

### مقایسه و الگوهای مختلف

**سه سطح parity:**

| سطح | ویژگی | هزینه |
|---|---|---|
| ۱ | dev روی لپ‌تاپ، prod در ابر | ❌ |
| ۲ | Docker در dev و prod | ✅ |
| ۳ | ✅ همان image، همان نسخه‌ها | ✅✅ |

**سه رویکرد به محیط dev:**

| رویکرد | توضیح |
|---|---|
| Local | روی لپ‌تاپ (Docker Compose) |
| Remote dev | محیط مشترک در ابر |
| Ephemeral | محیط موقت برای هر branch |

**توصیه**: Local برای سرعت، ephemeral برای تست integration.

**سه مدل staging:**

| مدل | توضیح |
|---|---|
| Shared staging | یک محیط برای همه |
| Per-team | هر تیم یک محیط |
| Ephemeral | برای هر MR |

**توصیه**: ephemeral (review app) بهترین است.

### اشکال شایع

**اشکال ۱: staging کوچک‌تر**

۲ instance در staging، ۲۰ در production. مشکلات مقیاس دیده نمی‌شوند.

**اشکال ۲: نسخه DB متفاوت**

PostgreSQL 14 در staging، 16 در production. رفتار متفاوت.

**اشکال ۳: config متفاوت در کد**

`if env == "production": ...` در کد. config را بیرون بکش.

**اشکال ۴: داده تست غیرواقعی**

۱۰ رکورد، همه کوتاه. مشکل طول رشته، مشکل encoding دیده نمی‌شود.

**اشکال ۵: dev روی Windows، prod Linux**

line ending، case sensitivity، مسیرها. Docker حل می‌کند.

**اشکال ۶: staging برای همیشه کهنه**

کسی نگه‌داری نمی‌کند. با IaC بازسازی خودکار.

### نکات مهم (Tips & Tricks)

- **همان image در همه محیط‌ها.**
- **config در deploy-time.**
- **lockfile برای dependency.**
- **داده anonymized شبیه production.**
- **Docker برای dev** — همان محیط.
- **staging با IaC بازسازی کن** — کهنه نمی‌شود.

### خلاصه فصل

- چهار بُعد parity: زمان، پرسنل، ابزار، کتابخانه.
- محیط غیرمشابه، تست نیست.
- همان image، config متفاوت.
- config در deploy-time.
- داده anonymized شبیه production.
- staging با IaC بازسازی خودکار.

---

## فصل ۴۶: مدیریت State و محیط‌های چندگانه

### مقدمه

در IaC، **state** حافظه‌ی سیستم است: می‌داند چه ساخته‌ای. اگر state خراب شود، IaC کور می‌شود و ممکن است منابع را دوباره بسازد یا حذف کند.

این فصل مدیریت state و چند محیط را می‌آورد.

### مفاهیم اساسی

**State چیست:**

```
state = نقشه‌ای بین کد و واقعیت

کد:    resource "aws_instance" "web" { ... }
state: aws_instance.web → i-0abc123def456
واقعیت: یک EC2 با ID i-0abc123def456
```

بدون state، IaC نمی‌داند کد به کدام منبع اشاره می‌کند.

**چهار خطر state:**

| خطر | نتیجه |
|---|---|
| **state در Git** | رمز لو می‌رود |
| **state محلی** | تیم نمی‌تواند share کند |
| **قفل نشده** | دو نفر همزمان apply → خرابی |
| **دستکاری دستی** | state با واقعیت ناهماهنگ |

**راه‌حل: Remote backend با قفل**

| ویژگی | چرا |
|---|---|
| Remote | تیم share می‌کند |
| Encrypted | رمز محافظت |
| Locking | apply همزمان ممکن نیست |
| Versioned | بازگشت ممکن |

### مثال‌های عملی

**ساختار چند محیطی:**

```
infra/
├── modules/
│   ├── network/
│   ├── app/
│   └── database/
├── envs/
│   ├── staging/
│   │   ├── backend.tf      # state در S3 key: staging/terraform.tfstate
│   │   ├── main.tf
│   │   └── terraform.tfvars
│   └── production/
│       ├── backend.tf      # state جدا
│       ├── main.tf
│       └── terraform.tfvars
```

**نکته حیاتی**: **state هر محیط جدا**. اگر یکی باشد، apply در staging می‌تواند production را تحت تأثیر قرار دهد.

**backend مشترک، state جدا:**

```hcl
# envs/staging/backend.tf
terraform {
  backend "s3" {
    bucket         = "acme-terraform-state"
    key            = "staging/terraform.tfstate"
    region         = "eu-central-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

```hcl
# envs/production/backend.tf
terraform {
  backend "s3" {
    bucket         = "acme-terraform-state"
    key            = "production/terraform.tfstate"    # ← متفاوت
    region         = "eu-central-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

**نکته**: همان bucket، key متفاوت. قفل مشترک (dynamodb) از تعارض جلوگیری می‌کند.

**Workspaces در برابر پوشه‌های جدا:**

| معیار | Workspace | پوشه جدا |
|---|---|---|
| جداسازی | ⚠️ ضعیف | ✅ قوی |
| خطر | اشتباه در workspace | ✅ کم |
| مناسب | dev/test | ✅ staging/prod |

**توصیه**: برای production، **پوشه جدا** — جداسازی فیزیکی، خطای انسانی کمتر.

**جریان CI/CD برای IaC:**

```yaml
# مفهوم، نه ابزار خاص
plan:
  stage: validate
  script:
    - terraform init
    - terraform fmt -check
    - terraform validate
    - terraform plan -out=tfplan
  artifacts:
    paths: [tfplan]

apply:
  stage: deploy
  script:
    - terraform apply tfplan
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual
  environment:
    name: production
```

**نکته**: `plan` و `apply` با **همان فایل plan** — تضمین می‌کند همان چیزی اجرا شود که دیدی.

### مقایسه و الگوهای مختلف

**سه مدل جداسازی محیط:**

| مدل | جداسازی | خطر |
|---|---|---|
| Workspace | ⚠️ | متوسط |
| پوشه جدا | ✅ | کم |
| مخزن جدا | ✅✅ | ✅✅ (ولی هماهنگی سخت) |

**توصیه**: پوشه جدا برای اکثر تیم‌ها.

**Monorepo در برابر Polyrepo برای IaC:**

| معیار | Monorepo | Polyrepo |
|---|---|---|
| هماهنگی | ✅ | ❌ |
| جداسازی | متوسط | ✅ |
| ماژول مشترک | ✅ آسان | سخت |

**سه سطح بلوغ:**

| سطح | ویژگی |
|---|---|
| ۱ | state محلی، یک محیط |
| ۲ | remote state، چند محیط |
| ۳ | ✅ + policy + drift detection |

### اشکال شایع

**اشکال ۱: state در Git**

رمزهای DB در state. `git rm --cached` و backend.

**اشکال ۲: یک state برای همه محیط‌ها**

apply در staging، production را حذف کرد. state جدا.

**اشکال ۳: state قفل نمی‌شود**

دو نفر همزمان، state خراب. dynamodb یا معادل.

**اشکال ۴: دستکاری state**

`terraform state rm` بدون درک. منابع یتیم.

**اشکال ۵: plan و apply جدا**

`apply` بدون فایل plan، تغییرات جدید اعمال می‌شود. همیشه `-out` و `apply tfplan`.

**اشکال ۶: drift نادیده**

کسی در کنسول تغییر داد. `plan` هفتگی در CI.

**اشکال ۷: ماژول بدون نسخه**

`source = "../../modules/network"` — تغییر در ماژول، همه محیط‌ها را می‌شکند.

### نکات مهم (Tips & Tricks)

- **state هر محیط جدا.**
- **Remote backend با قفل و رمزنگاری.**
- **`plan` در MR، `apply` در main.**
- **`-out=tfplan` و `apply tfplan`.**
- **drift detection هفتگی.**
- **پوشه جدا برای production.**
- **state را دستکاری نکن** — مگر با درک کامل.

### خلاصه فصل

- state = نقشه بین کد و واقعیت.
- چهار خطر: Git، محلی، بدون قفل، دستکاری.
- state هر محیط جدا.
- remote backend با قفل و رمزنگاری.
- plan و apply با همان فایل.
- drift detection دوره‌ای.

---

## منابع و مراجع

**کتاب‌ها:**
- [Infrastructure as Code](https://www.oreilly.com/library/view/infrastructure-as-code/9781098114664/) — Kief Morris
- [Terraform Up & Running](https://www.terraformupandrunning.com/) — Yevgeniy Brikman
- [The Phoenix Project](https://itrevolution.com/product/the-phoenix-project/)
- [Site Reliability Engineering](https://sre.google/sre-book/table-of-contents/)

**مقالات:**
- [The Twelve-Factor App](https://12factor.net/) — به‌ویژه Factor V (Build/Release/Run) و X (Dev/Prod Parity)
- [Martin Fowler: ImmutableServer](https://martinfowler.com/bliki/ImmutableServer.html)
- [Martin Fowler: PhoenixServer](https://martinfowler.com/bliki/PhoenixServer.html)
- [Martin Fowler: SnowflakeServer](https://martinfowler.com/bliki/SnowflakeServer.html)

**ابزارها (مفهومی):**
- [Terraform](https://developer.hashicorp.com/terraform/docs)
- [OpenTofu](https://opentofu.org/)
- [Ansible](https://docs.ansible.com/)
- [Packer](https://developer.hashicorp.com/packer/docs)
- [Open Policy Agent](https://www.openpolicyagent.org/)

---

**بعدی:** [بخش ۰۹: سنجش و متریک‌های DORA](09-Metrics-and-DORA.md)
