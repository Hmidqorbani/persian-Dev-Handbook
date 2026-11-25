# فایل ۱۵ — Terraform: زیرساخت به عنوان کد (IaC)

**مدت یادگیری**: ۱۲ ساعت
**سطح سختی**: متوسط–پیشرفته
**پیش‌نیاز**: فایل ۰۲ (لینوکس/شبکه)، فایل ۱۸/۱۹ (ابر — پیشنهاد مطالعه بعد)

---

## فهرست محتوا

- [فصل ۷۱: مفهوم IaC و معرفی Terraform](#فصل-۷۱-مفهوم-iac-و-معرفی-terraform)
- [فصل ۷۲: ساختار Terraform (HCL، Providers، Resources، State)](#فصل-۷۲-ساختار-terraform-hcl-providers-resources-state)
- [فصل ۷۳: مدیریت State و ماژول‌ها](#فصل-۷۳-مدیریت-state-و-ماژولها)
- [فصل ۷۴: Terraform در تیم (Remote State، Locking، CI)](#فصل-۷۴-terraform-در-تیم-remote-state-locking-ci)
- [فصل ۷۵: بهترین روش‌ها و ترکیب با Kubernetes](#فصل-۷۵-بهترین-روشها-و-ترکیب-با-kubernetes)

---

## فصل ۷۱: مفهوم IaC و معرفی Terraform

### مدمه

روزگاری زیرساخت با کلیک در کنسول ابری ساخته می‌شد — رویکردی که به سرعت به فاجعهٔ «برفی بودن» (Snowflake) و عدم تکرارپذیری منجر می‌شد. **Infrastructure as Code (IaC)** زیرساخت را مثل کد مدیریت می‌کند. **Terraform** محبوب‌ترین ابزار IaC است. این فصل چرا و چگونه را توضیح می‌دهد.

### مفاهیم اساسی

**Infrastructure as Code چیست؟**
تعریف زیرساخت (VM، شبکه، دیتابیس، دسترس) در فایل‌های متنی که در Git نگهداری، نسخه‌گذاری و از طریق ابزار اعمال می‌شوند.

**مزایا:**
- **تکرارپذیری (Reproducibility):** محیط dev دقیقاً مثل prod.
- **Version Control:** تغییرات با PR قابل بررسی.
- **Automation:** ایجاد ۱۰۰ سرور مثل ۱ سرور.
- **Documentation:** کد خودش سند است.
- **Rollback:** به نسخهٔ قبلی برگردید.

**انواع ابزار IaC:**
- **Imperative (دستوری):** چطور (مثل Ansible scripts، bash) — فصل ۱۷.
- **Declarative (اعلانی):** چه می‌خواهید (مثل Terraform، CloudFormation) — شما وضعیت مطلوب را می‌گویید، ابزار تفاوت را محاسبه می‌کند.

**Terraform چیست؟**
ابزار متن‌باز HashiCorp (اکنون BusL) برای Provisioning زیرساخت به صورت declarative با زبان **HCL** (HashiCorp Configuration Language). Provider-agnostic: AWS، Azure، GCP، Kubernetes، و صدها ارائه‌دهنده دیگر.

**جریان کاری Terraform:**
```
write .tf files → terraform init → plan → apply → state ثبت می‌شود
```

**جایگزین‌ها:** AWS CloudFormation (فقط AWS)، Pulumi (زبان برنامه‌نویسی واقعی — فصل ۱۶)، OpenTofu (فورک متن‌باز Terraform پس از تغییر لایسنس).

### مثال‌های عملی

**۱. نصب Terraform:**

```bash
# لینوکس
wget https://releases.hashicorp.com/terraform/1.9.0/terraform_1.9.0_linux_amd64.zip
unzip terraform_1.9.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/
terraform version
```

**۲. اولین فایل (`main.tf`):**

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "my_bucket" {
  bucket = "my-unique-bucket-2026"
}
```

**۳. اجرای اولیه:**

```bash
terraform init      # دانلود provider
terraform plan      # پیش‌نمایش تغییرات
terraform apply     # اعمال
terraform destroy   # حذف
```

**۴. مقایسه Imperative در برابر Declarative:**

```bash
# Imperative (bash): aws s3 mb s3://x  (دستور مستقیم)
# Declarative (tf):   resource "aws_s3_bucket" "x" {}  (وضعیت مطلوب)
```

### مقایسه و دیدگاه‌های مختلف

| ابزار | نوع | زبان | محدوده |
|---|---|---|---|
| Terraform | Declarative | HCL | Multi-cloud |
| CloudFormation | Declarative | JSON/YAML | فقط AWS |
| Ansible | Imperative/Declarative | YAML | Config+Provision |
| Pulumi | Declarative | Python/TS/Go | Multi-cloud |
| OpenTofu | Declarative | HCL | Multi-cloud (OS) |

**دیدگاه‌ها:**
- Terraform استاندارد صنعت برای provisioning زیرساخت ابری است.
- تغییر لایسنس Terraform (به BSL) باعث ظهور OpenTofu شد — برای پروژه‌های ضدانحصار توصیه می‌شود.
- Pulumi برای تیم‌هایی که ترجیح می‌دهند از زبان برنامه‌نویسی استفاده کنند جذاب است (فصل ۱۶).

**Terraform vs Ansible:** Terraform برای «ساخت زیرساخت» (lifecycle کوتاه‌تر، create/destroy)؛ Ansible برای «پیکربندی داخل ماشین» (نصب نرم‌افزار، فصل ۱۷).

### اشتباهات رایج

- **دستکاری دستی در کنسول پس از terraform apply:** انحراف (drift) — Terraform نمی‌داند.
- **فراموش `terraform init`:** provider دانلود نشده → خطا.
- **استفاده از `latest` در provider:** تغییر نسخه باعث خرابی غیرمنتظره.
- **نام‌های غیریکتا (مثل bucket):** S3 bucket names globally unique هستند → خطا.
- **`apply` بدون `plan` اول:** تغییر خطرناک بدون بازبینی.

### نکات مهم و ترفندها

- **همیشه `plan` قبل از `apply`:** یا `apply` با `-out` برای بررسی.
- **`terraform fmt`:** فرمت‌بندی خودکار (مثل gofmt).
- **`terraform validate`:** اعتبارسنجی نحو قبل از plan.
- **Version pinning:** همیشه نسخهٔ provider را قفل کنید.
- **`.gitignore`:** فایل‌های `terraform.tfstate` و `.terraform/` را از Git exclude کنید (state در remote باشد — فصل ۷۴).
- **Workspaces:** برای محیط‌های مختلف (dev/prod) در همان کد.

### خلاصه فصل

IaC زیرساخت را مثل کد مدیریت می‌کند (تکرارپذیر، versioned، خودکار). Terraform ابزار declarative با زبان HCL است که وضعیت مطلوب را می‌گیرد و تفاوت را محاسبه می‌کند — multi-cloud و provider-agnostic. جریان: init → plan → apply. کلید: همیشه plan قبل از apply، نسخهٔ provider را قفل کنید، از دستکاری دستی در کنسول اجتناب کنید (drift)، و state را در Git نگذارید. OpenTofu جایگزین متن‌باز پس از تغییر لایسنس است.

---

## فصل ۷۲: ساختار Terraform (HCL، Providers، Resources، State)

### مدمه

فهم ساختار فایل‌های Terraform و نحوهٔ تعامل با **Provider**، **Resource** و **State** برای نوشتن کد ایمن و قابل نگهداری ضروری است. در این فصل سینتکس HCL و مفاهیم هسته را کالبدشکافی می‌کنیم.

### مفاهیم اساسی

**ساختار پروژه:**

```
project/
├── main.tf           # منابع اصلی
├── variables.tf      # متغیرهای ورودی
├── outputs.tf        # خروجی‌ها
├── providers.tf      # تنظیم provider
├── terraform.tfvars # مقادیر متغیرها
└── versions.tf       # نسخه‌بندی
```

**عناصر HCL:**

| عنصر | کاربرد |
|---|---|
| `provider` | اتصال به پلتفرم (aws، azurerm، google) |
| `resource` | یک منبع زیرساخت (مثل `aws_instance`) |
| `variable` | ورودی پارامتریک |
| `output` | خروجی قابل نمایش/ارجاع |
| `module` | بسته‌بندی منابع (فصل ۷۳) |
| `data` | خواندن منبع موجود (غیرمدیریتی) |
| `locals` | متغیر محاسبه‌شده محلی |

**Provider:**
افزونه‌ای که API پلتفرم را می‌شناسد. هر provider منابع خاص خودش را دارد (مثل `aws_instance`).

**Resource addressing:**
`resource "type" "name" { ... }` → آدرس در state: `type.name` (مثل `aws_instance.web`).

**State:**
فایل (معمولاً JSON) که نگاشت بین کد شما و منابع واقعی در ابر را نگه می‌دارد. Terraform برای محاسبهٔ تفاوت، state را با واقعیت مقایسه می‌کند.

**Expressions & Functions:**
HCL توابع دارد: `length()`, `merge()`, `lookup()`, `${...}` برای interpolation.

### مثال‌های عملی

**۱. فایل کامل (`main.tf`):**

```hcl
terraform {
  required_version = ">= 1.5"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.region
}

variable "region" {
  type    = string
  default = "us-east-1"
}

variable "instance_type" {
  type    = string
  default = "t3.micro"
}

resource "aws_instance" "web" {
  ami           = "ami-0abcdef1234567890"
  instance_type = var.instance_type

  tags = {
    Name = "web-server"
    Env  = "dev"
  }
}

output "public_ip" {
  value = aws_instance.web.public_ip
}
```

**۲. استفاده از `data` (منبع موجود):**

```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

resource "aws_instance" "web" {
  ami = data.aws_ami.ubuntu.id
  # ...
}
```

**۳. متغیرها از tfvars:**

```hcl
# terraform.tfvars
region        = "eu-west-1"
instance_type = "t3.small"
```

```bash
terraform apply -var="region=us-west-2"
# یا: terraform apply -var-file="prod.tfvars"
```

**۴. locals و توابع:**

```hcl
locals {
  full_name = "${var.env}-${var.app}"
}

resource "aws_s3_bucket" "b" {
  bucket = lower(local.full_name)
}
```

### مقایسه و دیدگاه‌های مختلف

| مفهوم | مشابه در برنامه‌نویسی |
|---|---|
| resource | تعریف شیء/کلاس |
| variable | پارامتر تابع |
| output | مقدار بازگشتی |
| module | کتابخانه/تابع |
| state | پایگاه دادهٔ نقشه |

**دیدگاه:** جداسازی `variables.tf`، `outputs.tf`، `main.tf` قرارداد رایجی است که خوانایی را بالا می‌برد. استفاده از `data` به جای hard-code کردن IDها (مثل AMI) کد را portable می‌کند.

**State importance:** بدون state، Terraform نمی‌داند چه چیزی را مدیریت می‌کند → هر اجرا recreate می‌کند (فاجعه).

### اشتباهات رایج

- **hard-code کردن AMI/ID:** بین منطقه‌ها کار نمی‌کند؛ از `data` استفاده کنید.
- **فراموش `default` در variable حساس:** خطا هنگام apply.
- **state در Git:** فایل tfstate ممکن است رمز (در user_data) داشته باشد → نشت.
- **تغییر نام resource بدون lifecycle:** Terraform آن را destroy+create می‌کند (از `lifecycle { prevent_destroy = true }` یا `moved` استفاده کنید).
- **متغیر بدون type:** خطای نوع در زمان اجرا.

### نکات مهم و ترفندها

- **`terraform show`:** نمایش state فعلی.
- **`terraform state list/mv/rm`:** مدیریت state.
- **`precondition`/`postcondition`:** اعتبارسنجی در resource (Terraform ۱.۲+).
- **Sensitive variables:** `variable "db_pass" { sensitive = true }` تا در خروجی ماسک شود.
- **`count` و `for_each`:** ایجاد چند منبع مشابه:
  ```hcl
  resource "aws_instance" "web" {
    count = 3
    # ...
  }
  ```
- **`depends_on`:** اجبار ترتیب صریح (به ندرت نیاز است؛ Terraform معمولاً خودش تشخیص می‌دهد).

### خلاصه فصل

ساختار Terraform بر پایه HCL است: provider (اتصال)، resource (منبع)، variable (ورودی)، output (خروجی)، data (خواندن موجود)، locals (محاسبه). State فایل نقشهٔ واقعیت است که Terraform برای محاسبهٔ تفاوت استفاده می‌کند — حیاتی و نباید در Git باشد. کلید: استفاده از `data` به جای hard-code، جداسازی فایل‌ها، `sensitive = true` برای رمز، و `count`/`for_each` برای تکرار. plan همیشه قبل از apply.

---

## فصل ۷۳: مدیریت State و ماژول‌ها

### مدمه

وقتی پروژه بزرگ می‌شود، دو چالش پدیدار می‌شود: **مدیریت state** (چطور آن را ایمن و مشترک نگه داریم) و **بازاستفاده از کد** (ماژول‌ها). این فصل دو مفهوم کلیدی توسعهٔ Terraform را پوشش می‌دهد.

### مفاهیم اساسی

**چرا State مهم و حساس است؟**
- نقشهٔ منابع واقعی است.
- می‌تواند اطلاعات حساس (مثل رمز در `user_data`) داشته باشد.
- اگر دو نفر هم‌زمان اجرا کنند، state خراب می‌شود (نیاز به **Locking**).

**State Backend (محل نگهداری):**
- **Local:** فایل `terraform.tfstate` روی دیسک (تنها برای تست).
- **Remote:** S3 + DynamoDB (AWS)، Azure Blob، GCS، یا Terraform Cloud — برای تیم ضروری.

**State Locking:**
جلوگیری از اجرای هم‌زمان که state را خراب کند. در S3+DynamoDB، جدول DynamoDB لاک را نگه می‌دارد.

**Modules (ماژول‌ها):**
بسته‌بندی منابع قابل بازاستفاده. مثل تابع/کتابخانه در برنامه‌نویسی. ساختار:
```
module "vpc" {
  source = "./modules/vpc"
  cidr   = "10.0.0.0/16"
}
```

**ماژول استاندارد:**
- `main.tf` (منابع)
- `variables.tf` (ورودی)
- `outputs.tf` (خروجی)
- `versions.tf`

**فواید:** DRY، استانداردسازی (مثل ماژول VPC تأییدشده تیم)، و انتزاع.

### مثال‌های عملی

**۱. Remote State (S3 + DynamoDB):**

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/network/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

**۲. ساخت ماژول (`modules/vpc/main.tf`):**

```hcl
# modules/vpc/variables.tf
variable "cidr" {
  type    = string
  default = "10.0.0.0/16"
}

# modules/vpc/main.tf
resource "aws_vpc" "this" {
  cidr_block = var.cidr
  tags = { Name = "main-vpc" }
}

# modules/vpc/outputs.tf
output "vpc_id" {
  value = aws_vpc.this.id
}
```

**۳. استفاده از ماژول:**

```hcl
module "vpc" {
  source = "./modules/vpc"
  cidr   = "10.1.0.0/16"
}

resource "aws_subnet" "sub" {
  vpc_id     = module.vpc.vpc_id
  cidr_block = "10.1.1.0/24"
}
```

**۴. ماژول از Registry:**

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  name    = "my-vpc"
  cidr    = "10.0.0.0/16"
  # ...
}
```

### مقایسه و دیدگاه‌های مختلف

| رویکرد State | کاربرد |
|---|---|
| Local | یادگیری/تست تک‌نفره |
| S3+DynamoDB | تیم AWS |
| Terraform Cloud | SaaS مدیریت‌شده |
| GCS/Azure Blob | تیم غیرAWS |

| رویکرد کد | کاربرد |
|---|---|
| Copy-paste | ❌ ضدالگو |
| Local module | تیم واحد |
| Registry module | سازمان/عمومی |

**دیدگاه:** Remote state با locking برای هر تیم ضروری است. ماژول‌های Registry (مثل terraform-aws-modules) صدها ساعت کار را ذخیره می‌کنند — اما بررسی امنیتی‌شان فراموش نشود (Supply chain).

**State separation:** محیط‌های مختلف (dev/prod) باید state جدا داشته باشند (جداگانه bucket یا key متفاوت).

### اشتباهات رایج

- **Local state در تیم:** تداخل و از دست رفتن state.
- **بدون Locking:** دو `apply` هم‌زمان → خرابی state.
- **ماژول بدون `versions.tf`:** ناسازگاری provider.
- **state حاوی رمز در Git:** نشت اطلاعات حساس.
- **ماژول بیش از حد انتزاعی:** ماژول ۲۰۰ خطی با ۵۰ ورودی = غیرقابل فهم.

### نکات مهم و ترفندها

- **`terraform state mv`:** جابه‌جایی منبع بدون recreate.
- **`terraform import`:** وارد کردن منبع موجود به state.
- **State encryption:** همیشه `encrypt = true` در backend.
- **Versioned bucket:** نسخه‌گذاری روی S3 برای بازیابی state خراب.
- **Module version pinning:** همیشه `version` در ماژول Registry.
- **`terraform graph`:** نمایش وابستگی منابع (دیباگ).

### خلاصه فصل

State نقشهٔ واقعیت است و باید در **remote backend** (S3+DynamoDB) با **locking** و **encryption** نگهداری شود — هرگز local یا در Git. ماژول‌ها کد را DRY و استاندارد می‌کنند (مثل توابع). کلید: state جدا برای هر محیط، استفاده از ماژول‌های Registry تأییدشده (با بررسی امنیتی)، `terraform import`/`state mv` برای مدیریت موجود، و ماژول‌های ساده و خوانا. Remote state با locking برای هر تیم الزامی است.

---

## فصل ۷۴: Terraform در تیم (Remote State، Locking، CI)

### مدمه

در محیط تک‌نفره، Terraform ساده است. اما در تیم، چالش‌های همکاری، ایمنی و خودکارسازی ظاهر می‌شوند. این فصل ادغام Terraform در خط لولهٔ CI/CD و کار تیمی ایمن را بررسی می‌کند.

### مفاهیم اساسی

**همکاری ایمن:**
- **Remote state** (فصل ۷۳) با locking اجباری.
- **PR-based workflow:** تغییرات از طریق PR، با `terraform plan` خودکار در CI به عنوان comment.
- **Terraform Cloud / Enterprise:** سرویس مدیریت‌شده با UI، policy (Sentinel/OPA)، و private registry.

**CI Integration:**
در یک PR:
1. `terraform init` (با backend از پیش تنظیم).
2. `terraform fmt -check` (فرمت).
3. `terraform validate` (اعتبارسنجی).
4. `terraform plan -out plan.tfplan` → نمایش diff در PR comment.
5. پس از approve و merge: `terraform apply plan.tfplan` (در محیط تولید).

**Terraform Cloud:**
- **Runs API:** مدیریت اجرا در ابر.
- **Policy as Code:** Sentinel یا OPA برای بررسی تغییرات (مثل "هیچ S3 بدون encryption نباشد").
- **Private Module Registry.**

**State Access Control:**
فقط CI/ Terraform Cloud به state دسترس داشته باشد، نه همهٔ توسعه‌دهندگان (کاهش ریسک).

**Terraform Workspaces:**
جدا کردن state برای محیط‌های مختلف در یک پروژه (اما معمولاً جدا کردن state backend تمیزتر است).

### مثال‌های عملی

**۱. GitHub Actions برای Terraform:**

```yaml
name: Terraform
on: [pull_request]
jobs:
  plan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
      - run: terraform init
      - run: terraform fmt -check
      - run: terraform validate
      - run: terraform plan -out=tfplan
      - name: Comment plan
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs')
            const plan = fs.readFileSync('tfplan', 'utf8')
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              body: '```\n' + plan + '\n```'
            })
```

**۲. Apply پس از merge (محیط prod):**

```yaml
on:
  push:
    branches: [main]
jobs:
  apply:
    environment: production
    steps:
      - terraform init
      - terraform apply -auto-approve tfplan
```

**۳. Terraform Cloud backend:**

```hcl
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "prod-network"
    }
  }
}
```

**۴. Policy (Sentinel مثال - مفهومی):**

```
# مثال: اجبار encryption روی S3
import "tfplan"
main = rule {
  all tfplan.resources.aws_s3_bucket as _, true  # (واقعی پیچیده‌تر)
}
```

### مقایسه و دیدگاه‌های مختلف

| رویکرد | مزیت | مناسب برای |
|---|---|---|
| Terraform OSS + CI | کنترل کامل | تیم با CI موجود |
| Terraform Cloud | مدیریت آسان، Policy | تیم بدون زیرسازی |
| GitLab Terraform | یکپارچه با GitLab | کاربران GitLab |
| Atlantis | PR-based در هر Git | تیم چابک |

**دیدگاه:** برای تیم‌های کوچک، Terraform OSS + GitHub Actions کافی است. برای سازمان‌های بزرگ، Terraform Cloud یا Atlantis (PR-based دیباگ‌پذیر) مقیاس‌پذیرتر. همیشه `plan` در PR و `apply` پس از approve.

**Environment separation:** هرگز apply مستقیم در prod از لپ‌تاپ نکنید؛ از CI/Cloud با محیط محافظت‌شده.

### اشتباهات رایج

- **`apply` دستی در prod:** بدون بررسی PR، ریسک بالا.
- **state در دسترس همه:** نشت اطلاعات حساس state.
- **فراموش `fmt -check` در CI:** کد نامرتب.
- **plan بدون `-out`:** کسی بین plan و apply تغییر می‌دهد → apply چیز دیگر.
- **Terraform Cloud بدون policy:** تغییرات ناایمن اعمال می‌شوند.

### نکات مهم و ترفندها

- **`-out planfile` سپس `apply planfile`:** اطمینان از اعمال همان plan.
- **Terraform Cloud run tasks:** یکپارچه با ابزار امنیتی (مثل Snyk).
- **Atlantis:** `atlantis plan`/`apply` از طریق PR comment.
- **OIDC به ابر:** CI بدون کلید ثابت (فصل ۲۸/۳۴).
- **Drift detection:** Terraform Cloud یا اسکریپت دوره‌ای `plan` برای تشخیص انحراف.
- **Cost estimation:** Terraform Cloud هزینهٔ plan را تخمین می‌زند.

### خلاصه فصل

در تیم، Terraform نیازمند remote state با locking، PR-based workflow (plan در PR comment)، و apply پس از approve در CI است. Terraform Cloud/Atlantis مدیریت را ساده می‌کنند و Policy as Code (Sentinel/OPA) تغییرات ناایمن را مسدود می‌کنند. کلید: هرگز apply دستی در prod، استفاده از `-out` planfile برای اعمال دقیق همان plan، محدود کردن دسترس به state، و OIDC به جای کلید ثابت در CI. Drift detection دوره‌ای فراموش نشود.

---

## فصل ۷۵: بهترین روش‌ها و ترکیب با Kubernetes

### مدمه

Terraform قدرتمند است اما با قدرت، مسئولیت می‌آید. در این فصل چک‌لیست بهترین روش‌ها و نحوهٔ تعامل Terraform با اکوسیستم Kubernetes (که خودش IaC دارد) را بررسی می‌کنیم تا از تداخل ابزارها جلوگیریم.

### مفاهیم اساسی

**چک‌لیست بهترین روش‌ها:**
1. **Remote state** با locking + encryption (هرگز local/Git).
2. **Version pinning:** Terraform و provider versions قفل شوند.
3. **Modules** برای DRY و استانداردسازی.
4. **`terraform plan` همیشه در CI** (PR comment).
5. **Sensitive = true** برای متغیرهای حساس.
6. **Naming conventions:** مداوم و معنادار.
7. **Small, focused modules:** نه غول‌آسا.
8. **`prevent_destroy`** برای منابع حیاتی (مثل DB).
9. **Tagging:** همهٔ منابع با `owner`، `env`، `project`.
10. **Separation of concerns:** شبکه، محاسبات، داده در ماژول/state جدا.

**Terraform vs Kubernetes — تقسیم کار:**
- **Terraform:** زیرساخت سطح پایین — VPC، شبکه، cluster K8s، RDS، IAM، دیسک.
- **Kubernetes manifests / GitOps:** چیزهای درون کلاستر — Deployment، Service، Ingress، CRD.

**Provider Kubernetes در Terraform:**
می‌توانید منابع K8s را هم با Terraform بسازید (مثل `kubernetes_deployment`)، اما ترکیب با **GitOps** (ArgoCD/Flux — فصل ۱۴) تمیزتر است: Terraform کلاستر را می‌سازد، ArgoCD محتوای درونی را.

**Terraform Import:**
وارد کردن منابع موجود (ساخته‌شده دستی) به state برای مدیریت بعدی.

**Terragrunt:**
ابزار لایه‌ای روی Terraform برای مدیریت چند محیط/ماژول (DRY در سطح بالاتر).

### مثال‌های عملی

**۱. جداسازی شبکه و اپلیکیشن:**

```hcl
# network.tf (state جدا)
module "vpc" { source = "./modules/vpc" }

# k8s-cluster.tf
module "eks" {
  source = "./modules/eks"
  vpc_id = module.vpc.vpc_id
}

# اپلیکیشن داخل کلاستر → با ArgoCD (نه Terraform)
```

**۲. `prevent_destroy`:**

```hcl
resource "aws_db_instance" "prod" {
  # ...
  lifecycle {
    prevent_destroy = true
  }
}
```

**۳. Import منبع موجود:**

```bash
terraform import aws_s3_bucket.my_bucket existing-bucket-name
```

**۴. Terraform + Kubernetes provider (محدود):**

```hcl
provider "kubernetes" {
  config_path = "~/.kubeconfig"
}
resource "kubernetes_namespace" "app" {
  metadata { name = "production" }
}
# (ترجیحاً این را به GitOps بسپارید)
```

### مقایسه و دیدگاه‌های مختلف

| وظیفه | ابزار |
|---|---|
| VPC/شبکه/IAM | Terraform |
| K8s cluster (EKS/GKE) | Terraform |
| Deploy اپلیکیشن در K8s | GitOps (ArgoCD/Flux) |
| Config درون ماشین | Ansible (فصل ۱۷) |

**دیدگاه:** قانون طلایی: **Terraform برای زیرساخت، GitOps برای اپلیکیشن**. استفاده از Terraform برای manage کردن هر Pod باعث کندی و پیچیدگی می‌شود. اگر کلاستر را با Terraform ساختید، اجازه دهید ArgoCD بقیه را کند.

**Terragrunt:** برای سازمان‌هایی با ده‌ها محیط که DRY در سطح state می‌خواهند.

### اشتباهات رایج

- **مدیریت Podها با Terraform:** خیلی کند و شکننده؛ از K8s/GitOps استفاده کنید.
- **یک state غول‌آسا برای همه:** blast radius بزرگ؛ در صورت خرابی همه می‌افتد.
- **بدون tagging:** پاکسازی هزینه‌ها غیرممکن می‌شود.
- **`prevent_destroy` فراموش شده روی ماژول:** نمی‌توانید destroy کنید.
- **ترکیب ابزار بدون مرز:** Terraform و Ansible هر دو تغییر ایجاد می‌کنند → conflict.

### نکات مهم و ترفندها

- **Small states:** state را بر اساس دامنه (شبکه، داده، محاسبات) تقسیم کنید.
- **`terraform taint`/`untaint`:** اجبار recreate یک منبع.
- **Drift detection منظم:** cron job برای `terraform plan` خاموش (مقایسه).
- **Cost tags + Budgets:** در ابر هزینه را مانیتور کنید.
- **Modules versioned in Registry:** برای reuse سازمانی.
- **Terraform Cloud cost estimation:** قبل از apply هزینه را ببینید.

### خلاصه فصل

بهترین روش‌های Terraform: remote state (locking+encryption)، version pinning، modules کوچک، plan در CI، sensitive variables، و tagging همه‌چیز. تقسیم کار با K8s: **Terraform زیرساخت (VPC/IAM/Cluster)، GitOps اپلیکیشن (Deployment/Service)** — مدیریت Pod با Terraform ضدالگو است. کلید: stateهای کوچک و جدا، `prevent_destroy` برای منابع حیاتی، `import` برای موجود، و ترجیح GitOps بر Terraform Kubernetes provider. Terragrunt برای DRY سطح بالا در سازمان‌های بزرگ.

---

## منابع و مراجع

- Terraform Documentation: https://developer.hashicorp.com/terraform
- Terraform Best Practices: https://developer.hashicorp.com/terraform/language
- Terraform Modules: https://developer.hashicorp.com/terraform/language/modules
- Terraform Backend (S3): https://developer.hashicorp.com/terraform/language/settings/backends/s3
- OpenTofu: https://opentofu.org
- Terraform Cloud: https://developer.hashicorp.com/terraform/cloud
- Atlantis: https://www.runatlantis.io
- Terragrunt: https://terragrunt.gruntwork.io
- AWS Provider: https://registry.terraform.io/providers/hashicorp/aws/latest

**بعدی:** [فایل ۱۶ — Pulumi و دیگر ابزارهای IaC](16-Pulumi-and-Other-IaC.md)
