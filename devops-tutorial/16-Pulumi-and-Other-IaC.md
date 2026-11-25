# فایل ۱۶ — Pulumi و دیگر ابزارهای IaC

**مدت یادگیری**: ۹ ساعت
**سطح سختی**: متوسط–پیشرفته
**پیش‌نیاز**: فایل ۱۵ (Terraform و مفاهیم IaC)

---

## فهرست محتوا

- [فصل ۷۶: محدودیت‌های HCL و ظهور Pulumi](#فصل-۷۶-محدودیتهای-hcl-و-ظهور-pulumi)
- [فصل ۷۷: مبانی Pulumi (SDK، Stack، State، زبان‌ها)](#فصل-۷۷-مبانی-pulumi-sdk-stack-state-زبانها)
- [فصل ۷۸: Pulumi در مقایسه با Terraform](#فصل-۷۸-pulumi-در-مقایسه-با-terraform)
- [فصل ۷۹: ابزارهای دیگر (CloudFormation، Crossplane، CDK)](#فصل-۷۹-ابزارهای-دیگر-cloudformation-crossplane-cdk)
- [فصل ۸۰: انتخاب ابزار IaC و ترکیب آن‌ها](#فصل-۸۰-انتخاب-ابزار-iac-و-ترکیب-آنها)

---

## فصل ۷۶: محدودیت‌های HCL و ظهور Pulumi

### مدمه

Terraform با HCL موفق بود، اما برای منطق پیچیده (حلقه‌ها، شرط‌های پیچیده، بازاستفاده از تابع/کلاس) محدود می‌شود. **Pulumi** ایدهٔ جسورانه‌ای داشت: چرا زبان جدید اختراع کنیم وقتی زبان‌های مدرن (Python، TypeScript، Go) داریم؟ در این فصل انگیزهٔ Pulumi را می‌فهمیم.

### مفاهیم اساسی

**محدودیت‌های HCL/Terraform:**
- زبان declarative ساده است اما برای منطق پیچیده (loops، abstraction) خسته‌کننده.
- توابع محدود؛ برای الگوریتم پیچیده باید خارج از Terraform کار کرد.
- تیم‌هایی که با Go/Python/TS مسلط‌اند، باید HCL یاد بگیرند (context switch).
- اشکال‌زدایی (debugging) در HCL ضعیف‌تر از IDEهای زبان واقعی.

**ایدهٔ Pulumi:**
استفاده از **زبان‌های برنامه‌نویسی عمومی** (General-Purpose Languages - GPL) برای تعریف زیرساخت. کدی مثل هر برنامهٔ دیگر می‌نویسید، اما خروجی آن منابع ابری است.

**مزایا:**
- **تمام قدرت زبان:** حلقه، تابع، کلاس، ماژول‌های npm/pip.
- **IDE/Debugging:** IntelliSense، type-check، breakpoints.
- **بازاستفاده:** کتابخانه‌های موجود قابل استفاده.
- **تست واحد:** با pytest/Jest زیرساخت را تست کنید.

**مفهوم Underlying:**
Pulumi در زیرِ خود از همان Providerها (بسیاری مستقیماً از Terraform providers) استفاده می‌کند — فقط لایهٔ نوشتن متفاوت است. State model مشابه Terraform دارد.

**معماری:**
شما کد را اجرا می‌کنید (`pulumi up`)، Pulumi engine آن را به یک گراف منابع تبدیل می‌کند و از طریق provider به ابر اعمال می‌کند.

### مثال‌های عملی

**۱. مقایسهٔ مستقیم (S3 bucket):**

```hcl
# Terraform (HCL)
resource "aws_s3_bucket" "b" {
  bucket = "my-bucket"
}
```

```typescript
// Pulumi (TypeScript)
import * as aws from "@pulumi/aws";
const bucket = new aws.s3.Bucket("b", {
  bucket: "my-bucket",
});
```

**۲. قدرت زبان (حلقه در Pulumi):**

```python
# Pulumi (Python) - ساخت ۳ bucket با حلقه
import pulumi
import pulumi_aws as aws

for i in range(3):
    aws.s3.Bucket(f"bucket-{i}", bucket=f"my-bucket-{i}")
```

```hcl
# در Terraform معادل:
resource "aws_s3_bucket" "b" {
  count  = 3
  bucket = "my-bucket-${count.index}"
}
# (محدودتر برای منطق پیچیده‌تر)
```

**۳. استفاده از کتابخانهٔ استاندارد:**

```typescript
// Pulumi (TS) - استفاده از crypto
import * as crypto from "crypto";
const password = crypto.randomBytes(16).toString("hex");
```

### مقایسه و دیدگاه‌های مختلف

| جنبه | Terraform (HCL) | Pulumi (GPL) |
|---|---|---|
| منحنی یادگیری | کم (DSL ساده) | بستگی به زبان شما |
| قدرت منطق | متوسط | کامل (زبان واقعی) |
| IDE support | ضعیف | عالی |
| Debugging | سخت | آسان (breakpoint) |
| اکوسیستم | بزرگ (Registry) | رو به رشد |

**دیدگاه‌ها:**
- تیم‌هایی با پیش‌زمینهٔ قوی برنامه‌نویسی Pulumi را ترجیح می‌دهند.
- تیم‌های عملیاتی/ساده‌تر Terraform را راحت‌تر یاد می‌گیرند.
- Pulumi برای زیرساخت + اپلیکیشن در یک کد (مثل serverless) عالی است.

**تجارت:** قدرت بیشتر = مسئولیت بیشتر (می‌توانید کد نوشت که اشتباه اجرا شود؛ نیاز به discipline).

### اشتباهات رایج

- **استفاده از عبارت‌های غیرقطعی:** در Pulumi، کد باید **deterministic** باشد (مثل `Math.random()` نباید استفاده شود — نام‌ها هر بار عوض می‌شود).
- **ترکیب state Pulumi و Terraform بدون plan:** هر دو state جدا دارند.
- **فراموش `pulumi up` در CI:** مثل Terraform نیاز به اجرا دارد.
- **وابستگی ضمنی نادیده گرفته:** در زبان ممکن است ترتیب اجرا نامشخص شود (از صریح استفاده کنید).

### نکات مهم و ترفندها

- **`pulumi preview`:** مثل `terraform plan`.
- **`pulumi up --stack prod`:** اجرا روی محیط خاص.
- **Testing:** با `@pulumi/pulumi`unit test در TS/Python.
- **Cross-language:** Pulumi اجازه می‌دهد منابع را در یک زبان بسازید و در زبان دیگر consume کنید (via packages).
- **Pulumi Service / Self-hosted:** backend برای state (مثل Terraform Cloud).

### خلاصه فصل

Pulumi محدودیت‌های HCL را با استفاده از زبان‌های برنامه‌نویسی واقعی (Python/TS/Go) حل می‌کند — قدرت کامل زبان، IDE، تست واحد. زیرِ خود از همان providerها استفاده می‌کند. کلید: کد باید deterministic باشد (هیچ random در نام‌ها)، و `pulumi preview`/`up` جایگزین plan/apply. تیم‌های برنامه‌نویس‌محور Pulumi را ترجیح می‌دهند؛ تیم‌های ops-simple ترجیح Terraform. قدرت بیشتر = نیاز به discipline بیشتر.

---

## فصل ۷۷: مبانی Pulumi (SDK، Stack، State، زبان‌ها)

### مدمه

برای شروع واقعی با Pulumi، باید با مفاهیم **Project**، **Stack**، **State** و زبان‌های پشتیبانی‌شده آشنا شوید. این فصل ساختار یک پروژهٔ Pulumi را می‌سازد.

### مفاهیم اساسی

**Project:**
پوشه‌ای با فایل `Pulumi.yaml` که نام پروژه و زبان را مشخص می‌کند.

**Stack:**
مثل **Workspace** در Terraform — نمونهٔ مستقل از پیکربندی (dev/staging/prod). هر stack state جدا دارد و متغیرهای خودش را دارد.

**State:**
مثل Terraform، Pulumi state را در backend نگه می‌دارد (Pulumi Service رایگان، یا self-hosted مثل S3/Azure Blob/GCS).

**SDK:**
کتابخانه‌های provider (مثل `@pulumi/aws`، `pulumi_aws` در پایتون).

**زبان‌های پشتیبانی‌شده:**
- **TypeScript / JavaScript** (محبوب‌ترین)
- **Python**
- **Go**
- **C# / .NET**
- **Java / YAML** (محدودتر)

**Output vs Input:**
در Pulumi، خروجی منابع (مثل `bucket.id`) یک **Output<T>** است (Promise-like) — نمی‌توانید مستقیم چاپش کنید؛ باید درون `apply()` استفاده کنید یا به عنوان ورودی منبع دیگر بدهید.

**Config:**
متغیرهای stack با `pulumi config set key value --stack dev`.

### مثال‌های عملی

**۱. ساختار پروژه (TypeScript):**

```
my-infra/
├── Pulumi.yaml
├── Pulumi.dev.yaml      # تنظیمات stack dev
├── Pulumi.prod.yaml
├── index.ts
├── package.json
└── tsconfig.json
```

**۲. Pulumi.yaml:**

```yaml
name: my-infra
runtime: nodejs
description: My infrastructure project
```

**۳. index.ts (مثال کامل):**

```typescript
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

// خواندن config
const config = new pulumi.Config();
const env = config.require("env"); // مثلاً "dev"

// ساخت VPC (ساده‌سازی شده)
const vpc = new aws.ec2.Vpc("main-vpc", {
  cidrBlock: "10.0.0.0/16",
  tags: { Name: `vpc-${env}` },
});

// خروجی
export const vpcId = vpc.id;
export const vpcCidr = vpc.cidrBlock;
```

**۴. اجرا:**

```bash
pulumi login                      # احراز (service یا self-hosted)
pulumi stack init dev             # ساخت stack
pulumi config set env dev --stack dev
pulumi preview                    # پیش‌نمایش
pulumi up                         # اعمال
pulumi stack output vpcId         # دیدن خروجی
```

**۵. Python معادل (بخش):**

```python
import pulumi
import pulumi_aws as aws

vpc = aws.ec2.Vpc("main-vpc", cidr_block="10.0.0.0/16")
pulumi.export("vpc_id", vpc.id)
```

### مقایسه و دیدگاه‌های مختلف

| مفهوم | Terraform | Pulumi |
|---|---|---|
| Project | دایرکتوری | Project + Pulumi.yaml |
| Environment | workspace/state key | Stack |
| Variables | `variable` + tfvars | `Config` + stack yaml |
| State | backend | Pulumi Service/backend |
| Output | `output` | `export` |

**دیدگاه:** مدل Stack در Pulumi تمیزتر از Workspace Terraform است (فایل پیکربندی جداگانه). تیم‌های چندزبانه می‌توانند هر کسی با زبان خودش زیرساخت بنویسد (اما در یک تیم، یک زبان بهتر است).

**Output<T>:** بزرگ‌ترین تفاوت ذهنی — درک async nature منابع ضروری است.

### اشتباهات رایج

- **چاپ کردن Output مستقیم:** `console.log(vpc.id)` مقدار را نمی‌دهد (یک promise است)؛ از `vpcId.apply(v => console.log(v))` یا `export` استفاده کنید.
- **deterministic نبودن:** استفاده از `Date.now()` یا `Math.random()` در نام‌ها → هر اجرا تغییر.
- **فراموش `pulumi login`:** state ذخیره نمی‌شود.
- **stack اشتباه:** `pulumi up` روی stack اشتباه → تغییر ناخواسته.
- **اختلاط زبان در یک تیم:** نگهداری سخت می‌شود.

### نکات مهم و ترفندها

- **`pulumi stack select`:** سوییچ بین stackها.
- **`pulumi history`:** تاریخچهٔ اجراها.
- **`pulumi destroy --stack dev`:** حذف یک stack.
- **Secrets در config:** `pulumi config set db_pass --secret` (رمزنگاری‌شده در state).
- **Component Resources:** برای ساخت abstractionهای سفارشی (مثل Terraform module).
- **`pulumi convert`:** تبدیل Terraform به Pulumi (تجربی).

### خلاصه فصل

پروژهٔ Pulumi شامل `Pulumi.yaml` (نام/زبان) و **Stack** (مثل workspace) با state جدا است. زبان‌های پشتیبانی‌شده: TS/JS، Python، Go، C#. نکتهٔ کلیدی: منابع **Output<T>** برمی‌گردانند (async) — نباید مستقیم چاپ شوند. کلید: `pulumi login`، استفاده از `Config` برای متغیرها، deterministic بودن کد (هیچ random)، و `--secret` برای رمز در config. Component Resources برای abstraction مثل ماژول‌های Terraform.

---

## فصل ۷۸: Pulumi در مقایسه با Terraform

### مدمه

تصمیم بین Pulumi و Terraform یکی از مباحث داغ DevOps است. این فصل مقایسهٔ جامع و راهنمای تصمیم‌گیری ارائه می‌دهد تا بر اساس بافتار تیم تصمیم بگیرید.

### مفاهیم اساسی

**ماتریس مقایسه:**

| معیار | Terraform | Pulumi |
|---|---|---|
| زبان | HCL (DSL) | GPL (Python/TS/Go) |
| یادگیری | کم (تیم ops) | بستگی به تسلط زبان |
| منطق پیچیده | محدود | کامل |
| اکوسیستم ماژول | بزرگ (Registry) | کوچک‌تر اما +npm/pip |
| State management | بالغ (S3/DynamoDB) | بالغ (Service/backends) |
| Policy as Code | Sentinel/OPA | Pulumi Policy (اختیاری) |
| هزینه | رایگان (OSS) / Cloud | رایگان (OSS) / Service (limit) |
| ترکیب با GitOps | خوب | خوب |

**شباهت‌های عمیق:**
- هر دو **declarative در نتیجه** (شما وضعیت مطلوب را می‌دهید، engine تفاوت را محاسبه می‌کند).
- هر دو state-based (نقشهٔ واقعیت).
- هر دو provider-agnostic (multi-cloud).
- هر دو `plan`/`preview` و `apply`/`up` دارند.

**تفاوت فلسفی:**
- Terraform: «زبان ساده برای همه».
- Pulumi: «از بهترین ابزار (زبان) استفاده کن».

**عملکرد:**
هر دو engine مشابه هستند؛ سرعت بیشتر به provider/API ابر بستگی دارد نه خود ابزار.

### مثال‌های عملی

**۱. منطق شرطی پیچیده:**

```hcl
# Terraform: باید با count/for_each و lookup کار کنید
resource "aws_instance" "web" {
  count = var.env == "prod" ? 5 : 1
  # ...
}
```

```python
# Pulumi: هر منطق پایتون
count = 5 if env == "prod" else 1
for i in range(count):
    aws.ec2.Instance(f"web-{i}", ...)
```

**۲. استفاده از کتابخانهٔ خارجی:**

```typescript
// Pulumi: استفاده از lodash، axios و...
import * as _ from "lodash";
const names = _.chunk(servers, 2);
```

**۳. معماری مشابه:**

```
Terraform:  main.tf → terraform plan → terraform apply → state
Pulumi:     index.ts → pulumi preview → pulumi up → state
```

### مقایسه و دیدگاه‌های مختلف

| سناریو | توصیه |
|---|---|
| تیم ops، زیرساخت ساده | Terraform |
| تیم dev قوی، منطق پیچیده | Pulumi |
| نیاز به Registry آماده زیاد | Terraform |
| Serverless + IaC یکجا | Pulumi |
| سازمان محافظه‌کار | Terraform (پخته‌تر) |
| تست واحد زیرساخت | Pulumi |

**دیدگاه‌ها:**
- بسیاری معتقدند Terraform برای «زیرساخت خالص» و Pulumi برای «زیرساخت + اپلیکیشن (مثل serverless apps)» بهتر است.
- تیم‌های mixed (dev+ops) اغلب Pulumi را ترجیح می‌دهند چون یک زبان مشترک دارند.
- OpenTofu (فورک Terraform) برای کسانی که نگران لایسنس هستند گزینهٔ سوم است.

**تجارت نهایی:** هیچ‌کدام ذاتاً «بهتر» نیست؛ انتخاب بر اساس **تسلط تیم** و **نوع کار** است.

### اشتباهات رایج

- **انتخاب Pulumi فقط چون "مدرن‌تر":** شاید تیم Terraform را سریع‌تر یاد بگیرد.
- **انتخاب Terraform چون "محبوب‌تر" برای کار پیچیده:** Pulumi در منطق پیچیده صرفه‌جویی واقعی می‌کند.
- **ترکیب بی‌برنامه:** هر دو روی همان منابع → conflict state.
- **نادیده گرفتن هزینهٔ Pulumi Service:** نسخهٔ رایگان محدود (seats)؛ برای تیم بزرگ ممکن است هزینه داشته باشد (یا self-host کنید).

### نکات مهم و ترفندها

- **شروع با همان چیزی که تیم بلد است:** اگر تیم Python می‌داند → Pulumi؛ اگر ops ساده → Terraform.
- **Terraform Provider در Pulumi:** بسیاری providerهای Terraform مستقیم در Pulumi کار می‌کنند (پوشش وسیع).
- **مهاجرت تدریجی:** می‌توان بخشی را Terraform و بخشی را Pulumi نگه داشت (state جدا، با data source ارجاع).
- **Policy:** Pulumi CrossGuard برای Policy as Code.
- **Testing:** Pulumi برای unit test زیرساخت بهتر است.

### خلاصه فصل

Terraform (HCL ساده) و Pulumi (زبان واقعی) هر دو ابزارهای بالغ state-based هستند. انتخاب بر اساس تسلط تیم و نوع کار: Terraform برای زیرساخت ساده/اپراتوری، Pulumi برای منطق پیچیده و تیم‌های برنامه‌نویس. هر دو multi-cloud و provider-agnostic. کلید: انتخاب بر اساس نیاز واقعی نه مد (Pulumi برای serverless+IaC یکجا عالی؛ Terraform برای Registry غنی و پختگی). ترکیب بی‌برنامه باعث conflict state می‌شود — مرزبندی کنید.

---

## فصل ۷۹: ابزارهای دیگر (CloudFormation، Crossplane، CDK)

### مدمه

Terraform و Pulumi تنها بازیگران نیستند. ابزارهای دیگری مثل **CloudFormation** (بومی AWS)، **Crossplane** (K8s-native IaC) و **CDK** (IaC با زبان روی CloudFormation) هر کدام جایگاهی دارند. این فصل نقشهٔ کامل اکوسیستم IaC را می‌کشد.

### مفاهیم اساسی

**AWS CloudFormation (CFN):**
سرویس بومی AWS برای تعریف منابع با JSON/YAML. رایگان (فقط هزینهٔ منابع). محدود به AWS.
- مزیت: یکپارچگی کامل با AWS (IAM، rollback خودکار).
- عیب: فقط AWS، سینتکس verbose، بدون abstraction قدرتمند.

**AWS CDK (Cloud Development Kit):**
لایه‌ای روی CloudFormation که اجازه می‌دهد با TypeScript/Python/Java زیرساخت بنویسید — اما خروجی‌اش CloudFormation است.
- مشابه Pulumi اما **فقط AWS** و خروجی CFN.
- خوب برای تیم‌های AWS-محور.

**Crossplane:**
ابزار **Kubernetes-native** — زیرساخت را به عنوان **Custom Resources** در K8s تعریف می‌کند. شما `yaml` می‌نویسید که منابع ابری (مثل RDS، S3) را می‌سازد، و Crossplane از طریق provider آن‌ها را در ابر ایجاد می‌کند.
- مزیت: یکپارچگی با GitOps (ArgoCD/Flux می‌توانند زیرساخت را هم مدیریت کنند!).
- عیب: یادگیری کانسپت K8s، پیچیدگی.

**AWS CDK for Terraform (CDKTF):**
ترکیب CDK با Terraform — بنویس با زبان، خروجی Terraform.

**Pulumi vs CDK:** هر دو زبان‌محور، اما CDK خروجی‌اش CFN (فقط AWS)، Pulumi مستقیم به ابر (multi-cloud).

### مثال‌های عملی

**۱. CloudFormation (YAML):**

```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-cfn-bucket
```

**۲. AWS CDK (TypeScript):**

```typescript
import * as cdk from "aws-cdk-lib";
import * as s3 from "aws-cdk-lib/aws-s3";

class MyStack extends cdk.Stack {
  constructor(scope, id) {
    super(scope, id);
    new s3.Bucket(this, "MyBucket", { bucketName: "my-cdk-bucket" });
  }
}
```

**۳. Crossplane (Kubernetes YAML):**

```yaml
apiVersion: s3.aws.crossplane.io/v1beta1
kind: Bucket
metadata:
  name: my-crossplane-bucket
spec:
  forProvider:
    bucketName: my-crossplane-bucket
    region: us-east-1
  providerConfigRef:
    name: aws-provider
# کنترلر Crossplane این را در AWS می‌سازد
```

**۴. CDK for Terraform:**

```typescript
import { TerraformStack } from "cdktf";
import { AwsProvider } from "@cdktf/provider-aws";
import { S3Bucket } from "@cdktf/provider-aws/lib/s3";

class MyStack extends TerraformStack {
  constructor(scope, id) {
    super(scope, id);
    new AwsProvider(this, "aws", { region: "us-east-1" });
    new S3Bucket(this, "bucket", { bucket: "my-cdktf-bucket" });
  }
}
```

### مقایسه و دیدگاه‌های مختلف

| ابزار | زبان | محدوده | خروجی |
|---|---|---|---|
| CloudFormation | JSON/YAML | AWS | مستقیم AWS |
| AWS CDK | TS/Py/Java | AWS | CloudFormation |
| CDKTF | TS/Py/Java | Multi-cloud | Terraform |
| Crossplane | YAML (K8s) | Multi-cloud | K8s CRD→ابر |
| Terraform | HCL | Multi-cloud | مستقیم |
| Pulumi | GPL | Multi-cloud | مستقیم |

**دیدگاه‌ها:**
- **Crossplane** برای تیم‌های K8s-native که می‌خواهند «زیرساخت هم GitOps باشد» انقلابی است.
- **CDK** برای تیم‌های AWS که زبان می‌خواهند اما به اکوسیستم AWS وفادارند.
- **CloudFormation** برای موارد ساده AWS یا وقتی ابزار دیگر در دسترس نیست.

**GitOps + Crossplane:** آیندهٔ احتمالی — همه چیز (اپ + زیرساخت) از یک کلاستر K8s با ArgoCD مدیریت می‌شود.

### اشتباهات رایج

- **قفل شدن روی ابر (Vendor lock-in):** CloudFormation/CDK فقط AWS — مهاجرت سخت.
- **Crossplane بدون فهم K8s:** پیچیدگی CRDها گیج‌کننده است.
- **ترکیب بی‌رویه:** ۳ ابزار IaC هم‌زمان → chaos.
- **CDKtf بدون درک Terraform:** خروجی را نمی‌فهمید اگر Terraform بلد نباشید.

### نکات مهم و ترفندها

- **انتخاب بر اساس ابر:** فقط AWS → CDK یا CFN؛ Multi-cloud → Terraform/Pulumi/Crossplane.
- **Crossplane + ArgoCD:** برای GitOps کامل (زیرساخت + اپلیکیشن).
- **CDKTF:** اگر Terraform دوست دارید اما زبان می‌خواهید.
- **Terraform provider برای CFN:** می‌توانید CFN stack را از Terraform مدیریت کنید.
- **Cost:** CloudFormation رایگان؛ Crossplane نیاز به کلاستر K8s دارد (هزینهٔ خودش).

### خلاصه فصل

فراتر از Terraform/Pulumi، اکوسیستم IaC غنی است: CloudFormation (بومی AWS)، AWS CDK (زبان روی CFN)، CDKTF (زبان روی Terraform)، و Crossplane (K8s-native، زیرساخت به عنوان CRD — قابل مدیریت با GitOps). کلید انتخاب: ابر تک‌گانه (AWS) → CDK/CFN؛ multi-cloud → Terraform/Pulumi/Crossplane. Crossplane + ArgoCD آیندهٔ GitOps کامل (اپ + زیرساخت یکجا) است. از ترکیب بی‌رویه ابزارها اجتناب کنید (vendor lock-in و chaos).

---

## فصل ۸۰: انتخاب ابزار IaC و ترکیب آن‌ها

### مدمه

با این همه گزینه، سؤال نهایی: **چه چیزی برای مناسب است؟** این فصل چک‌لیست تصمیم‌گیری و الگوهای ترکیبی (Composition) را ارائه می‌دهد تا استراتژی IaC منسجمی داشته باشید.

### مفاهیم اساسی

**چک‌لیست انتخاب:**

```
[ ] آیا فقط یک ابر دارید؟
    → بله AWS: CDK یا Terraform
    → خیر (multi-cloud): Terraform / Pulumi / Crossplane
[ ] تیم چقدر با برنامه‌نویسی آشناست؟
    → کم: Terraform (HCL ساده)
    → زیاد: Pulumi / CDK
[ ] آیا K8s نویس هستید؟
    → بله: Crossplane + GitOps
[ ] به اکوسیستم آماده (Registry) نیاز دارید؟
    → Terraform (بزرگ‌ترین)
[ ] نگران لایسنس HashiCorp هستید؟
    → OpenTofu / Pulumi
```

**الگوهای ترکیبی (Composition):**
1. **Terraform + GitOps:** Terraform زیرساخت (VPC/EKS)، ArgoCD اپلیکیشن درون کلاستر (فصل ۱۴/۱۵).
2. **Crossplane + ArgoCD:** همه چیز از K8s (زیرساخت + اپ) با GitOps.
3. **Pulumi + CI:** Pulumi برای provisioning، معمولاً در CI اجرا می‌شود.
4. **Terraform + Ansible:** Terraform زیرساخت، Ansible پیکربندی درون ماشین (فصل ۱۷).

**اصل جدایی (Separation of Concerns):**
- **Provisioning** (ساخت): Terraform/Pulumi/Crossplane.
- **Configuration** (پیکربندی): Ansible/Chef/Puppet.
- **Deployment** (استقرار اپ): GitOps/K8s manifests.
- **Orchestration** (ارکستراسیون): Kubernetes.

**استراتژی State:**
هر ابزار state خودش را دارد → مرزبندی کنید (مثلاً Terraform فقط شبکه، Pulumi فقط اپ‌سرورها — اما این خطرناک است اگر تداخل کنند؛ بهتر: یک ابزار غالب داشته باشید).

### مثال‌های عملی

**۱. استراتژی ترکیبی پیشنهادی (معمول):**

```
Terraform  → VPC، Subnets، EKS Cluster، RDS، IAM
ArgoCD     → همهٔ اپلیکیشن‌های درون EKS (Deployment/Service/Ingress)
Ansible    → نصب agentهای درون VMهای غیرکانتینری (اگر دارید)
```

**۲. Crossplane + ArgoCD (GitOps کامل):**

```yaml
# ArgoCD هر دو را مدیریت می‌کند:
applications:
  - infrastructure (Crossplane YAMLs)
  - applications (K8s manifests)
```

**۳. تصمیم‌گیری ساده:**

```
تیم کوچک، AWS:      → Terraform (شروع سریع)
تیم dev سنگین:      → Pulumi (Python/TS)
سازمان K8s-native:  → Crossplane + ArgoCD
سازمان محافظه‌کار:   → Terraform / OpenTofu
```

### مقایسه و دیدگاه‌های مختلف

| استراتژی | بهترین برای |
|---|---|
| Terraform-only | اکثر تیم‌ها (شروع ساده) |
| Pulumi-only | تیم‌های برنامه‌نویس |
| Crossplane+GitOps | سازمان‌های K8s-native |
| Terraform+Ansible | ترکیب provision+config سنتی |
| Multi-tool حداقل | اجتناب از complexity |

**دیدگاه نهایی:** «بهترین ابزار» وجود ندارد، «بهترین ترکیب برای شما» وجود دارد. بیشتر تیم‌ها با **Terraform + GitOps** شروع می‌کنند (بالغ، مستند) و بعد بر اساس نیاز تکامل می‌دهند. Crossplane برای آیندهٔ GitOps-native جذاب است اما منحنی یادگیری دارد.

**کلید موفقیت:** یک ابزار غالب برای provisioning + GitOps برای deployment + جدایی clear بین لایه‌ها.

### اشتباهات رایج

- **استفاده از ۴ ابزار هم‌زمان:** نگهداری غیرممکن.
- **بدون مرزبندی state:** دو ابزار روی یک منبع → conflict.
- **انتخاب بر اساس ترند:** نه نیاز واقعی.
- **نادیده گرفتن یادگیری تیم:** ابزار قدرتمند که کسی بلد نیست = بی‌فایده.
- **فراموش GitOps برای اپ:** IaC فقط برای زیرساخت کافی نیست؛ اپلیکیشن هم نیاز به هم‌راستایی دارد.

### نکات مهم و ترفندها

- **یک ابزار غالب:** بیش از یک ابزار provisioning نداشته باشید مگر ضرورت.
- **Git برای همه:** IaC + اپلیکیشن هر دو در Git (GitOps).
- **Policy guardrails:** OPA/Gatekeeper روی همهٔ منابع.
- **Drift detection:** برای هر ابزار.
- **Documentation:** تصمیم انتخاب ابزار را مستند کنید (چرا این؟).
- **Cost awareness:** هر ابزار هزینهٔ خودش (مثل Crossplane نیاز به K8s دارد).

### خلاصه فصل

انتخاب IaC بر اساس ابر (تک‌/multi)، تسلط تیم، و معماری (K8s-native یا نه) است. استراتژی پیشنهادی: یک ابزار غالب برای provisioning (معمولاً Terraform/OpenTofu) + GitOps (ArgoCD/Flux) برای deployment + Ansible برای config درون ماشین. Crossplane+GitOps آیندهٔ K8s-native است. کلید: مرزبندی clear بین لایه‌ها (provision/config/deploy)، اجتناب از ابزار زیاد، و مستندسازی تصمیم. بیشتر تیم‌ها با Terraform+GitOps شروع و تکامل می‌یابند.

---

## منابع و مراجع

- Pulumi Documentation: https://www.pulumi.com/docs/
- Pulumi vs Terraform: https://www.pulumi.com/docs/intro/vs/
- OpenTofu: https://opentofu.org
- AWS CloudFormation: https://docs.aws.amazon.com/cloudformation/
- AWS CDK: https://docs.aws.amazon.com/cdk/
- CDK for Terraform: https://developer.hashicorp.com/terraform/cdktf
- Crossplane: https://www.crossplane.io
- Terraform Best Practices (Anton Babenko): https://www.terraform-best-practices.com
- Gruntwork (Terragrunt): https://terragrunt.gruntwork.io

**بعدی:** [فایل ۱۷ — Ansible: پیکربندی و مدیریت پیکربندی](17-Ansible.md)
