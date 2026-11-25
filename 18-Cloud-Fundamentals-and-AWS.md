# فایل ۱۸ — مبانی ابر و AWS (Cloud Fundamentals و AWS)

**مدت یادگیری**: ۱۲ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۲ (لینوکس/شبکه)، فایل ۱۵ (Terraform — اختیاری اما کمک‌کننده)

---

## فهرست محتوا

- [فصل ۸۶: مفاهیم پایهٔ محاسبات ابری (Cloud Fundamentals)](#فصل-۸۶-مفاهیم-پایهٔ-محاسبات-ابری-cloud-fundamentals)
- [فصل ۸۷: معرفی AWS و سرویس‌های محاسباتی (Compute)](#فصل-۸۷-معرفی-aws-و-سرویسهای-محاسباتی-compute)
- [فصل ۸۸: ذخیره‌سازی و پایگاه‌داده در AWS](#فصل-۸۸-ذخیرهسازی-و-پایگاهداده-در-aws)
- [فصل ۸۹: شبکه در AWS (VPC و همکاران)](#فصل-۸۹-شبکه-در-aws-vpc-و-همکاران)
- [فصل ۹۰: مدیریت دسترسی و IaC اولیه در AWS (IAM و CloudFormation)](#فصل-۹۰-مدیریت-دسترسی-و-iac-اولیه-در-aws-iam-و-cloudformation)

---

## فصل ۸۶: مفاهیم پایهٔ محاسبات ابری (Cloud Fundamentals)

### مقدمه

واژهٔ «ابر» (Cloud) امروز آن‌قدر پرکاربرد شده که گاهی معنای دقیقش فراموش می‌شود. در ساده‌ترین تعریف، **محاسبات ابری (Cloud Computing)** یعنی اجارهٔ منابع محاسباتی — سرور، ذخیره‌سازی، شبکه، پایگاه‌داده — از یک ارائه‌دهندهٔ خارجی که از طریق اینترنت (یا لینک اختصاصی) در دسترس‌اند. به‌جای خرید فیزیکی سرور و نصب در رک، شما منابع را «تقاضا کنید و مصرف کنید» (on-demand). این فصل پایه‌هایی را می‌سازد که تمام بحث‌های AWS و سایر ابرها روی آن استوار است.

### مفاهیم اساسی

**مدل‌های سرویس‌دهی (Service Models)** — سه لایهٔ اصلی:

1. **IaaS (Infrastructure as a Service):** زیربنای سخت‌افزاری مجازی‌سازی‌شده. شما OS را مدیریت می‌کنید، ارائه‌دهنده سخت‌افزار/شبکه/hypervisor را. مثال: EC2، Virtual Machines آژور، Compute Engine گوگل.
2. **PaaS (Platform as a Service):** پلتفرم آماده برای اجرای کد/برنامه بدون مدیریت OS. مثال: AWS Elastic Beanstalk، Azure App Service، Google App Engine.
3. **SaaS (Software as a Service):** نرم‌افزار آمادهٔ مصرف‌کننده. مثال: Gmail، Salesforce، Slack.

لایه هرچه بالاتر، مسئولیت مدیریتی شما کمتر است. در DevOps معمولاً بین IaaS و PaaS در نوسانید.

**مدل‌های استقرار (Deployment Models):**

- **Public Cloud:** اشتراک منابع میان مشتریان (multi-tenant). ارزان و مقیاس‌پذیر. (AWS، Azure، GCP)
- **Private Cloud:** ابر اختصاصی یک سازمان (on-prem یا میزبانی‌شده). کنترل و انطباق (compliance) بیشتر.
- **Hybrid Cloud:** ترکیبی از هر دو؛ بارهای حساس در private، بارهای متغیر در public.
- **Multi-Cloud:** استفاده همزمان از چند ارائه‌دهنده (مثلاً AWS + GCP) برای جلوگیری از وابستگی به یک فروشنده (vendor lock-in).

**مدل مسئولیت مشترک (Shared Responsibility Model):** ارائه‌دهنده امنیت «ابر» (زیرساخت فیزیکی، hypervisor، شبکهٔ فیزیکی) را بر عهده دارد؛ شما امنیت «درون ابر» (OS، پیکربندی، داده‌ها، IAM) را. این تقسیم در هر مدل سرویس‌دهی متفاوت است (در SaaS مسئولیت شما بسیار کم است).

**مفاهیم کلیدی دیگر:**

- **Region (منطقه):** مجموعه‌ای از مراکز داده در یک جغرافیا (مثلاً `us-east-1` در ورginia). انتخاب Region روی latency، قیمت و قوانین داده (data residency) اثر دارد.
- **Availability Zone / AZ (منطقهٔ دسترس‌پذیری):** مرکز دادهٔ مستقل با برق/شبکه/خنک‌کنندهٔ مجزا درون یک Region. طراحی برای fault tolerance روی پخش بار میان چند AZ است.
- **Edge Locations / CDN:** نقاط حضور نزدیک کاربر برای تحویل سریع محتوا (مثل CloudFront).
- **CAPEX vs OPEX:** ابر هزینهٔ سرمایه‌ای (خرید سخت‌افزار) را به هزینهٔ عملیاتی (اجارهٔ ماهانه) تبدیل می‌کند.
- **Elasticity (کشش):** افزایش/کاهش خودکار منابع بر اساس بار.
- **Metered Service / Pay-as-you-go:** پرداخت فقط بابت مصرف.

### مثال‌های عملی

سناریوی سنتی در مقابل ابری را ببینید:

**قبل (on-prem):**
```
خرید ۱۰ سرور فیزیکی → نصب در رک → نصب OS → پیکربندی →
راه‌اندازی → اما ۹۰٪ زمان بیکار (over-provisioning برای spikeهای نادر)
هزینه: ۵۰٬۰۰۰ دلار سرمایه + نگهداری + فضای فیزیکی
```

**بعد (cloud):**
```bash
# راه‌اندازی یک نمونه (instance) در AWS با CLI
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t3.medium \
  --key-name my-key \
  --subnet-id subnet-0abcd1234 \
  --security-group-ids sg-01234abcd

# پاسخ: نمونه ظرف ۶۰ ثانیه آماده است؛ فقط بابت ثانیه‌های روشن‌بودن پرداخت می‌کنید
```

**مثال Elasticity با Auto Scaling Group (نمای کلی):**
```
بار کم  → ۲ نمونه فعال   → هزینه کم
بار زیاد → خودکار ۸ نمونه → مقیاس‌پذیری
بار برگشت → بازگشت به ۲ نمونه → صرفه‌جویی
```

**مثال انتخاب Region بر اساس قانون:**
- داده‌های شهروندان اتحادیه اروپا → ترجیحاً `eu-central-1` (فرانکفورت) برای انطباق با GDPR.
- کاربران ایران → نزدیک‌ترین Region از نظر latency معمولاً `me-central-1` (امارات) یا `eu-south-1` (میلان).

### مقایسه و دیدگاه‌های مختلف

**ابر در مقابل on-premise:**

| معیار | On-Premise | Cloud |
|---|---|---|
| سرمایه اولیه | بالا (CAPEX) | صفر (OPEX) |
| زمان راه‌اندازی | هفته‌ها/ماه‌ها | دقایق/ساعت |
| مقیاس‌پذیری | سخت و کُند | آنی (Elasticity) |
| نگهداری | تیم داخلی | ارائه‌دهنده |
| کنترل | کامل | مشترک |
| قفل شدن (lock-in) | — | ریسک وجود دارد |

**دیدگاه‌های متفاوت:**
- **مبلغان ابر:** انعطاف، سرعت نوآوری، نیاز به تیم کوچک‌تر زیرساخت.
- **منتقدان:** هزینهٔ بلندمدت در مصرف بالا گاهی از on-prem گران‌تر می‌شود؛ نگرانی حاکمیت داده و وابستگی به فروشنده.
- **رویکرد عملی (FinOps):** ابر را با نظارت مالی (cost governance) اداره کنید — نه «اجارهٔ بی‌مرز»، بلکه «مصرف آگاهانه».

**Public در برابر Hybrid:** بانک‌ها و نهادهای دولتی معمولاً Hybrid را ترجیح می‌دهند (دادهٔ حساس در private)؛ استارتاپ‌ها عمدتاً Public.

### اشتباهات رایج

1. **نادیده گرفتن Shared Responsibility:** «ابر امن است پس من کاری ندارم» — اشتباه؛ نشت داده معمولاً از پیکربندی غلطِ خودِ مشتری است (مثل S3 عمومیِ اشتباه).
2. **انتخاب Region فقط بر اساس عادت:** Region ارزان‌تر یا نزدیک‌تر لزوماً بهتر نیست؛ قابلیت سرویس‌ها در همه Regionها یکسان نیست.
3. **عدم خاموش کردن منابع آزمایشی:** نمونه‌های تست که روشن می‌مانند هزینهٔ سنگینی ایجاد می‌کنند.
4. **تخمین هزینه بدون ابزار:** استفاده نکردن از Pricing Calculator قبل از مهاجرت.
5. **قفل شدن بی‌برنامه:** نوشتن همه چیز با سرویس‌های انحصاریِ یک ابر بدون لایهٔ انتزاعی (abstraction).

### نکات مهم و ترفندها

- **قانون ۳-AZ:** برای تولید، معماری را طوری ببینید که حداقل در ۲–۳ AZ پخش شود تا خرابی یک AZ کل سرویس را از کار نیندازد.
- **ابزار هزینه:** AWS Cost Explorer، Budgets و علامت‌گذاری منابع (tagging) با کلید `Environment=prod` را از روز اول فعال کنید.
- **Well-Architected Framework:** پنج ستون — Operational Excellence، Security، Reliability، Performance Efficiency، Cost Optimization. چک‌لیست رایگان AWS است.
- **Sandbox/Account Strategy:** حساب‌های جداگانه برای تولید/تست/توسعه (AWS Organizations) استفاده کنید تا هزینه و دسترسی ایزوله شود.
- **Infrastructure as Code:** حتی یک نمونهٔ تست را هم با Terraform/CloudFormation بسازید؛ «کلیک کردن در کنسول» قابل بازتولید (reproducible) نیست.
- **Free Tier:** AWS سال اول منابع رایگان می‌دهد — برای یادگیری حیاتی است.

### خلاصه فصل

ابر مدل مصرف منابع را از «مالکیت» به «اجارهٔ تقاضامحور» تغییر داد. سه مدل سرویس‌دهی (IaaS/PaaS/SaaS) و چهار مدل استقرار (Public/Private/Hybrid/Multi-Cloud) چارچوب گفت‌وگو را می‌سازند. مدل مسئولیت مشترک یادآوری می‌کند که امنیت همچنان نیمی از آن بر عهدهٔ شماست. Region/AZ پایهٔ دسترس‌پذیری، و مفاهیم Elasticity/Pay-as-you-go پایهٔ اقتصاد ابرند. در فصل بعد به‌طور مشخص وارد AWS می‌شویم.

---

## فصل ۸۷: معرفی AWS و سرویس‌های محاسباتی (Compute)

### مقدمه

**AWS (Amazon Web Services)** پیشگام و بزرگ‌ترین ارائه‌دهندهٔ ابری است (بیش از ۲۰۰ سرویس). درک معماری جهانی آن (Global Infrastructure) و سرویس‌های محاسباتی (Compute) کلید ورود به DevOps روی ابر است. این فصل EC2، Lambda، ECS و EKS — قلب محاسباتی AWS — را بررسی می‌کند.

### مفاهیم اساسی

**زیرساخت جهانی AWS:**
- **Region:** مثلاً `us-east-1` (N. Virginia)، `eu-west-1` (Irland)، `me-central-1` (UAE).
- **AZ:** هر Region ۲ تا ۶ AZ دارد (مثلاً `us-east-1a`، `us-east-1b`). نام‌گذاری AZ بین حساب‌ها یکسان نیست (برای توازن بار).
- **Edge/Pop:** برای CloudFront و Route53.

**۱. Amazon EC2 (Elastic Compute Cloud):** سرویس IaaS — نمونه‌های (مجازی) سرور.
- **AMI (Amazon Machine Image):** قالب دیسک شامل OS و نرم‌افزار برای راه‌اندازی نمونه.
- **Instance Types:** خانواده‌هایی بر اساس نیاز — `t3` (عمومی/ارزان)، `m5` (متوازن)، `c5` (محاسباتی)، `r5` (حافظه‌محور)، `g4` (GPU).
- **EBS (Elastic Block Store):** دیسک بلوکی پایدار متصل به نمونه (مثل هارد داخلی ماشین).
- **Security Group (SG):** فایروال مجازی stateful در سطح نمونه.
- **Key Pair:** برای ورود SSH (کلید عمومی در AWS، خصوصی نزد شما).
- **Auto Scaling Group (ASG):** حفظ تعداد نمونه بر اساس سیاست (مقیاس‌بندی خودکار).
- **Load Balancer (ELB):** توزیع ترافیک میان نمونه‌ها (ALB برای HTTP، NLB برای TCP).

**۲. AWS Lambda:** محاسبات بدون سرور (Serverless) — شما کد می‌دهید، AWS زمان‌بندی/مقیاس را مدیریت می‌کند. پرداخت فقط بابت میلی‌ثانیهٔ اجرا.
- مناسب: پردازش رویداد، API سبک، اتوماسیون.
- محدودیت: ۱۵ دقیقه سقف اجرا، حافظه تا ۱۰ گیگ.

**۳. Amazon ECS (Elastic Container Service):** ارکستراسیون کانتینرِ مدیریت‌شده توسط AWS.
- **Fargate:** اجرای کانتینر بدون مدیریت سرور (Serverless containers).
- **EC2 launch type:** کانتینر روی نمونه‌های خودتان.

**۴. Amazon EKS (Elastic Kubernetes Service):** نسخهٔ مدیریت‌شدهٔ Kubernetes در AWS — در فصل‌های ۱۱–۱۳ با Kuberentes آشنا شدید؛ EKS همان را با مدیریت control plane توسط AWS می‌دهد.

### مثال‌های عملی

**الف) راه‌اندازی EC2 با User Data (Bootstrap):**
```bash
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.micro \
  --key-name devops-key \
  --security-group-ids sg-0a1b2c3d \
  --user-data '#!/bin/bash
    sudo yum update -y
    sudo yum install -y docker
    sudo systemctl enable --now docker'
```

**ب) یک تابع Lambda ساده (Python) برای پاسخ به آپلود در S3:**
```python
import json

def lambda_handler(event, context):
    # هر بار فایلی در bucket می‌افتد، این تابع اجرا می‌شود
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        print(f"فایل جدید آپلود شد: s3://{bucket}/{key}")
    return {'statusCode': 200}
```

**ج) تعریف یک Task در ECS (نمای کلی JSON):**
```json
{
  "family": "web-service",
  "containerDefinitions": [
    {
      "name": "nginx",
      "image": "nginx:1.25",
      "portMappings": [{ "containerPort": 80 }],
      "essential": true
    }
  ],
  "cpu": "256",
  "memory": "512"
}
```

**د) اتصال kubectl به EKS:**
```bash
aws eks update-kubeconfig --name my-cluster --region us-east-1
kubectl get nodes   # همان دستورات Kubernetes فصل ۱۱
```

### مقایسه و دیدگاه‌های مختلف

**انتخاب لایهٔ محاسبات:**

| نیاز | گزینه پیشنهادی |
|---|---|
| کنترل کامل OS / نرم‌افزار قدیمی | EC2 |
| سرویس وب کوچک / پاسخ به رویداد | Lambda |
| کانتینرهای بدون مدیریت K8s | ECS Fargate |
| اکوسیستم Kubernetes / ابزارهای K8s | EKS |
| می‌خواهید کد را بدون زیرساخت اجرا کنید | Lambda / Fargate |

**EC2 در برابر Lambda:**
- EC2: همیشه روشن (هزینه ثابت)، کنترل کامل، مناسب بار پیوسته.
- Lambda: فقط هنگام رویداد (هزینه متغیر)، نگهداری صفر، مناسب بار متناوب.

**دیدگاه Serverless-first:** برخی تیم‌ها تا جای ممکن Lambda/Fargate استفاده می‌کنند تا نگهداری زیرساخت را به AWS بسپارند. منتقدان می‌گویند debugging و قفل شدن روی Lambda سخت‌تر است. رویکرد عملی: «اگر بار پیوسته است → EC2/ECS؛ اگر رویدادمحور است → Lambda».

### اشتباهات رایج

1. **انتخاب نوع نمونهٔ اشتباه:** استفاده از `t3` برای بار سنگین پایگاه‌داده → کُندی؛ یا `r5` گران برای وب‌سرور سبک → هزینهٔ اضافه.
2. **باز گذاشتن Security Group:** اجازهٔ SSH (پورت ۲۲) از `0.0.0.0/0` برای همه → ریسک امنیتی بزرگ.
3. **فراموشی خاموش کردن نمونه:** نمونه‌های تست روشن‌مانده هزینه تولید می‌کنند.
4. **استفاده از نمونهٔ On-Demand برای بار پایدار:** بار همیشگی می‌تواند با **Savings Plans / Reserved Instances** تا ۷۰٪ ارزان‌تر شود.
5. **نادیده گرفتن محدودیت Lambda:** فراخوانی سرویس با زمان اجرای بیش از ۱۵ دقیقه در Lambda → شکست؛ باید EC2/ECS.

### نکات مهم و ترفندها

- **Instance Purchasing Options:** On-Demand (انعطاف)، Reserved (تعهد ۱–۳ سال، ارزان‌تر)، Spot (ظرفیت مازاد، تا ۹۰٪ ارزان‌تر اما ممکن است قطع شود — عالی برای batch jobها).
- **IMDSv2:** همیشه از نسخهٔ دوم متادیتا (با توکن) استفاده کنید تا از حملات SSRF محافظت شود.
- **Systems Manager (SSM):** به‌جای باز کردن پورت SSH، از `aws ssm start-session` برای ورود امن بدون کلید استفاده کنید.
- **Launch Templates:** به‌جای Launch Configurations قدیمی، از Launch Template برای ASG استفاده کنید (نسخه‌بندی دارد).
- **Container Insights:** برای ECS/EKS مانیتورینگ آماده فعال کنید.
- **Local Zones / Outposts:** برای latency بسیار پایین یا نیاز on-prem همکار AWS فراهم است.

### خلاصه فصل

AWS با Region/AZ ساختار جهانی خود را می‌سازد. در لایهٔ Compute، **EC2** کنترل کامل (IaaS)، **Lambda** اجرای بدون‌سرور (رویدادمحور)، **ECS** ارکستراسیون کانتینر مدیریت‌شده، و **EKS** نسخهٔ مدیریت‌شدهٔ Kubernetes را ارائه می‌دهد. انتخاب میان آن‌ها بر اساس نیاز به کنترل در برابر نگهداریِ کم است. ابزارهای هزینه (Spot/Reserved) و امنیت (SSM/IMDSv2/SG) در عمل حیاتی‌اند. فصل بعد به ذخیره‌سازی و پایگاه‌داده می‌پردازیم.

---

## فصل ۸۸: ذخیره‌سازی و پایگاه‌داده در AWS

### مقدمه

داده قلب هر سیستم است. AWS طیف وسیعی از گزینه‌های ذخیره‌سازی — از اشیاء (objects) گرفته تا بلوک (block) و پایگاه‌دادهٔ رابطه‌ای و NoSQL — ارائه می‌دهد. انتخاب اشتباه ذخیره‌سازی می‌تواند روی performance، هزینه و دوام (durability) اثر بگذارد. این فصل S3، EBS، RDS، DynamoDB و همکاران را بررسی می‌کند.

### مفاهیم اساسی

**۱. Amazon S3 (Simple Storage Service):** ذخیره‌سازی اشیاء (object storage) با دوام ۹۹.۹۹۹۹۹۹۹۹۹٪ (۱۱ نه صفر).
- **Bucket:** ظرف منطقی نام‌گذاری‌شده سراسری (نام باید یکتا در کل AWS).
- **Object:** داده + متادیتا، با کلید (key) یکتا.
- **Storage Classes:** Standard (دسترسی سریع)، Intelligent-Tiering (خودکار ارزان‌سازی)، Standard-IA (نایاب)، Glacier (آرشیو ارزان، بازیابی کُند).
- **Versioning:** نگهداری نسخه‌های قبلی (برای بازیابی از حذف اشتباه).
- **Lifecycle Policy:** انتقال خودکار به کلاس ارزان‌تر یا حذف پس از N روز.
- **Static Website Hosting:** میزبانی سایت ایستا مستقیماً از S3.
- **Encryption:** SSE-S3 / SSE-KMS / SSE-C.

**۲. Amazon EBS:** دیسک بلوکی برای EC2 (فصل قبل). نوع‌ها: `gp3` (عمومی SSD، ارزان و سریع)، `io2` (بسیار بالا IOPS)، `st1` (HDD برای جریان داده).

**۳. Amazon RDS (Relational Database Service):** پایگاه‌دادهٔ رابطه‌ای مدیریت‌شده (PostgreSQL، MySQL، MariaDB، SQL Server، Oracle). AWS پچ، بکاپ، replication را مدیریت می‌کند.
- **Multi-AZ:** رونوشت همزمان در AZ دوم برای high availability.
- **Read Replicas:** رونوشت فقط‌خواندنی برای مقیاس خواندن.

**۴. Amazon Aurora:** نسخهٔ AWS از MySQL/PostgreSQL با معماری توزیع‌شده (تا ۵ برابر عملکرد MySQL) و دوام ۶-way replication در ۳ AZ.

**۵. Amazon DynamoDB:** پایگاه‌دادهٔ NoSQL کلید-مقدار (key-value) و سند (document)، سرورلس، با تأخیر میلی‌ثانیه‌ای حتی در مقیاس پتابایت.
- **Partition Key / Sort Key:** طراحی جدول حیاتی است.
- **On-Demand vs Provisioned:** پرداخت بر اساس مصرف یا ظرفیت رزرو شده.

**۶. Amazon ElastiCache:** کش در‌حافظه (Redis / Memcached) برای کاهش بار پایگاه‌داده.

### مثال‌های عملی

**الف) ساخت bucket و آپلود با CLI:**
```bash
aws s3 mb s3://my-app-assets-2026 --region us-east-1
aws s3 cp ./build/ s3://my-app-assets-2026/ --recursive
# فعال‌سازی versioning
aws s3api put-bucket-versioning \
  --bucket my-app-assets-2026 \
  --versioning-configuration Status=Enabled
```

**ب) Lifecycle برای انتقال به Glacier پس از ۹۰ روز:**
```json
{
  "Rules": [{
    "ID": "archive-old",
    "Status": "Enabled",
    "Prefix": "logs/",
    "Transitions": [{ "Days": 90, "StorageClass": "GLACIER" }],
    "Expiration": { "Days": 365 }
  }]
}
```

**ج) ایجاد RDS PostgreSQL (نمای کلی):**
```bash
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t3.medium \
  --engine postgres \
  --master-username admin \
  --master-user-password 'StrongPass!123' \
  --allocated-storage 20 \
  --multi-az
```

**د) جدول DynamoDB با Python (boto3):**
```python
import boto3
dynamodb = boto3.resource('dynamodb')
table = dynamodb.create_table(
    TableName='Orders',
    KeySchema=[
        {'AttributeName': 'order_id', 'KeyType': 'HASH'},   # Partition key
        {'AttributeName': 'created_at', 'KeyType': 'RANGE'} # Sort key
    ],
    AttributeDefinitions=[
        {'AttributeName': 'order_id', 'AttributeType': 'S'},
        {'AttributeName': 'created_at', 'AttributeType': 'S'}
    ],
    BillingMode='PAY_PER_REQUEST'
)
print("جدول ساخته شد:", table.table_status)
```

### مقایسه و دیدگاه‌های مختلف

**ذخیره‌سازی اشیاء در برابر بلوک:**
- **S3 (object):** برای فایل‌های بزرگ، آرشیو، محتوای استاتیک، بکاپ. دسترسی از طریق API/HTTP.
- **EBS (block):** برای فایل‌سیستم که نمونهٔ EC2 نیاز دارد (مثل دیسک ریشه). مستقیماً به یک نمونه متصل است.

**رابطه‌ای در برابر NoSQL:**
- **RDS/Aurora:** داده با روابط پیچیده، تراکنش ACID، SQL. (مثلاً سیستم حسابداری)
- **DynamoDB:** داده ساختاریافتهٔ ساده با مقیاس انفجاری و نیاز به تأخیر پایین (مثلاً سبد خرید، جلسات کاربر).

**مدیریت‌شده در برابر خودمدیریت (Self-managed):**
- RDS: پچ/بکاپ خودکار → نگهداری کم.
- نصب PostgreSQL روی EC2: کنترل کامل اما مسئولیت بکاپ/پچ با شماست.
- دیدگاه: «اگر تخصص پایگاه‌داده ندارید یا می‌خواهید زمان صرف محصول کنید → حتماً Managed».

### اشتباهات رایج

1. **S3 عمومیِ اشتباه:** باز گذاشتن bucket روی public با دادهٔ حساس → نشت اطلاعات (یکی از شایع‌ترین حوادث امنیتی).
2. **لایهٔ ذخیره‌سازی نادرست:** نگه داشتن لاگ‌های قدیمی در Standard (گران) به‌جای Glacier.
3. **عدم فعال‌سازی Multi-AZ برای تولید:** خرابی AZ → قطع پایگاه‌داده.
4. **اندازهٔ EBS ناکافی:** پر شدن دیسک → خرابی برنامه (همیشه alarm روی ۸۰٪ ظرفیت بگذارید).
5. **طراحی بد Partition Key در DynamoDB:** توزیع ناعادلانه (hot partition) → محدودیت throughput.
6. **ذخیرهٔ رمز عبور در RDS به‌صورت plaintext:** باید هش‌شده + KMS برای کلیدها.

### نکات مهم و ترفندها

- **S3 Intelligent-Tiering:** اگر الگوی دسترسی نامشخص است، این کلاس خودکار بین لایه‌ها جابه‌جا می‌کند بدون هزینهٔ بازیابی.
- **Cross-Region Replication (CRR):** رونوشت S3 در Region دیگر برای disaster recovery.
- **RDS Proxy:** برای جلوگیری از exhaustion اتصال (connection storm) در serverless/Lambda.
- **DynamoDB Accelerator (DAX):** کش در‌حافظه برای DynamoDB جهت کاهش تأخیر خواندن.
- **Backup:** RDS Automated Backup + PITR (بازیابی به نقطهٔ زمانی)؛ EBS Snapshots برای دیسک.
- **Cost:** S3 به‌ازای هر درخواست هم هزینه دارد — حلقه‌های پرتکرار خواندن کوچک را با batchینگ کاهش دهید.

### خلاصه فصل

ذخیره‌سازی در AWS لایه‌بندی شده: **S3** برای اشیاء/آرشیو/محتوای استاتیک با دوام تقریباً کامل؛ **EBS** دیسک بلوکی EC2؛ **RDS/Aurora** پایگاه‌دادهٔ رابطه‌ای مدیریت‌شده با Multi-AZ و Read Replicas؛ **DynamoDB** NoSQL سرورلس برای مقیاس انفجاری؛ **ElastiCache** برای کش. کلید موفقیت: انتخاب بر اساس الگوی دسترسی (نه فقط راحتی)، فعال‌سازی versioning/Multi-AZ/backup، و مدیریت هزینه با lifecycle و لایه‌های مناسب. فصل بعد شبکه (VPC) را می‌سازیم که همهٔ این‌ها را به هم متصل می‌کند.

---

## فصل ۸۹: شبکه در AWS (VPC و همکاران)

### مقدمه

تا اینجا منابع محاسباتی و ذخیره‌سازی را دیدیم، اما آن‌ها باید در یک شبکهٔ ایزوله و قابل کنترل با هم صحبت کنند و با اینترنت ارتباط داشته باشند. **VPC (Virtual Private Cloud)** شبکهٔ مجازی اختصاصی شماست درون AWS. این فصل اجزای VPC — Subnet، Route Table، Internet Gateway، NAT، Security Group، NACL، Route53، CloudFront، ALB/NLB — را می‌سازد.

### مفاهیم اساسی

**VPC و CIDR:** VPC فضای آدرس‌دهی خصوصی دارد (مثل `10.0.0.0/16`). زیربخش‌های آن Subnetها هستند.

**انواع Subnet:**
- **Public Subnet:** مسیر خروج به اینترنت از طریق **Internet Gateway (IGW)** دارد. معمولاً میزبان Load Balancer و Bastion.
- **Private Subnet:** دسترسی خروجی از طریق **NAT Gateway** (برای آپدیت سیستم) اما ورودی مستقیم از اینترنت ندارد. میزبان برنامه/پایگاه‌داده.
- **Isolated/Private-only:** بدون دسترسی اینترنت (مثل دادهٔ حساس).

**مولفه‌های کلیدی:**
- **Route Table:** تعیین مسیر ترافیک هر Subnet (مثلاً `0.0.0.0/0 → IGW`).
- **Internet Gateway (IGW):** پل VPC به اینترنت (دوطرفه).
- **NAT Gateway:** اجازه خروج private subnet به اینترنت بدون پذیرش ورودی.
- **Security Group (SG):** فایروال stateful در سطح نمونه (فقط «اجازه»، پیش‌فرض رد).
- **Network ACL (NACL):** فایروال stateless در سطح Subnet (باز/بسته صریح هر دو جهت).
- **VPC Peering / Transit Gateway:** اتصال VPCها به هم.
- **VPN / Direct Connect:** لینک اختصاصی از دیتاسنتر شما به AWS.

**سرویس‌های لبه (Edge):**
- **Route53:** سرویس DNS مدیریت‌شده (رزولوشن نام، health check، routing policies مثل latency-based).
- **CloudFront:** CDN برای تحویل سریع محتوای استاتیک/پویا از نقاط Edge.
- **ALB (Application Load Balancer):** لایهٔ ۷ (HTTP/HTTPS) — routing هوشمند بر اساس مسیر/هدر.
- **NLB (Network Load Balancer):** لایهٔ ۴ (TCP) — کارایی بسیار بالا، آدرس IP ثابت.

### مثال‌های عملی

**الف) ایجاد VPC + Subnet + IGW (CLI ترکیبی):**
```bash
# ساخت VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --region us-east-1
# ساخت Subnet عمومی
aws ec2 create-subnet --vpc-id vpc-0abc123 --cidr-block 10.0.1.0/24
# ساخت Internet Gateway و اتصال
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --internet-gateway-id igw-0abc --vpc-id vpc-0abc123
```

**ب) تنظیم Security Group (فقط اجازه SSH از IP شرکت و HTTP از همه):**
```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-0web123 \
  --protocol tcp --port 22 \
  --cidr 203.0.113.0/24
aws ec2 authorize-security-group-ingress \
  --group-id sg-0web123 \
  --protocol tcp --port 80 \
  --cidr 0.0.0.0/0
```

**ج) رکورد Route53 ساده (ALIAS به CloudFront):**
```json
{
  "Changes": [{
    "Action": "UPSERT",
    "ResourceRecordSet": {
      "Name": "www.example.com",
      "Type": "A",
      "AliasTarget": {
        "DNSName": "d123.cloudfront.net",
        "HostedZoneId": "Z2FDTNDATAQYW2"
      }
    }
  }]
}
```

**د) توزیع CloudFront روی bucket S3:**
```
Origin: my-app-assets-2026.s3.amazonaws.com
Behavior: /*  →  Cache Policy: CachingOptimized
          →  Viewer Protocol: Redirect to HTTPS
نتیجه: کاربران از نزدیک‌ترین Edge محتوا را با HTTPS دریافت می‌کنند
```

### مقایسه و دیدگاه‌های مختلف

**SG در برابر NACL:**
- SG: stateful — اگر ترافیک خروجی مجاز باشد، پاسخ آن خودکار وارد می‌شود. در سطح نمونه.
- NACL: stateless — هر دو جهت را جداگانه باید تعریف کرد. در سطح Subnet. معمولاً لایهٔ دفاع دوم.

**استراتژی Subnet:**
- **یک‌لایه (Flat):** همه در public — ساده اما ناامن.
- **دو‌لایه (Public/Private):** استاندارد تولید — LB در public، برنامه/DB در private.
- **سه‌لایه:** + لایهٔ data ایزوله‌تر.

**ALB در برابر NLB:**
- ALB: نیازمندی‌های وب مدرن (path-based routing، auth، WebSockets)؛ ارزان‌تر در مقیاس کوچک.
- NLB: throughput بسیار بالا، latency بسیار پایین، UDP support.

**دیدگاه:** بسیاری معماران «هر چیزی که نیاز به اینترنت ندارد را در private subnet قرار بده» را قانون طلایی می‌دانند.

### اشتباهات رایج

1. **قرار دادن پایگاه‌داده در Public Subnet:** دسترسی مستقیم از اینترنت → فاجعهٔ امنیتی.
2. **فراموشی Route Table:** ساخت Subnet اما تنظیم نکردن مسیر به IGW/NAT → «نمونه اینترنت ندارد».
3. **استفاده از CIDR اشتباه/تداخلی:** همپوشانی VPCها مانع Peering می‌شود.
4. **تکیه فقط بر SG یا فقط بر NACL:** هر دو لایه را برای defense-in-depth داشته باشید.
5. **NAT Gateway گران در ترافیک بالا:** NAT Gateway به‌ازای داده هزینه دارد؛ برای ترافیک زیاد گاهی NAT Instance یا PrivateLink بهتر است.
6. **باز گذاشتن SG روی ۰.۰.۰.۰/۰ برای همه پورت‌ها:** فقط پورت‌های ضروری را باز کنید.

### نکات مهم و ترفندها

- **VPC Flow Logs:** فعال کنید تا ترافیک شبکه را برای عیب‌یابی و امنیت ثبت کنید (به CloudWatch/S3).
- **PrivateLink (VPC Endpoint):** دسترسی به سرویس‌های AWS (مثل S3، KMS) بدون خروج به اینترنت از طریق endpoint خصوصی — امن‌تر و ارزان‌تر از NAT.
- **AWS Transit Gateway:** برای اتصال چند VPC / چند حساب / on-prem در مقیاس بزرگ (ستاره‌ای).
- **Security Group as Source:** می‌توانید SG دیگر را به‌عنوان منبع (source) یک SG قرار دهید — مثلاً «فقط نمونه‌های لایه وب اجازه دسترس به لایه DB دارند».
- **Egress-only Internet Gateway:** برای IPv6 خروجی بدون ورودی.
- **VPC Wizard:** برای شروع سریع، کنسول VPC «VPC with Public and Private Subnets» را پیشنهاد می‌دهد.

### خلاصه فصل

شبکه در AWS حول **VPC** می‌چرخد: فضای خصوصی با Subnetهای public/private، مسیریابی via Route Table، دسترسی اینترنت via IGW و NAT. **Security Group** (stateful، در سطح نمونه) و **NACL** (stateless، در سطح Subnet) لایه‌های دفاع شبکه‌اند. در لبه، **Route53** (DNS)، **CloudFront** (CDN) و **ALB/NLB** (ترافیک) قرار دارند. اصل طلایی: منابع حساس در private subnet، فقط پورت‌های ضروری باز، و استفاده از Flow Logs و PrivateLink برای دید و امنیت. فصل بعد به مدیریت دسترسی (IAM) و آغاز Infrastructure as Code در AWS می‌پردازیم.

---

## فصل ۹۰: مدیریت دسترسی و IaC اولیه در AWS (IAM و CloudFormation)

### مقدمه

دو موضوع پایانی این فایل روی «چه کسی اجازهٔ چه کاری را دارد» و «چگونه زیرساخت را قابل بازتولید بسازیم» تمرکز دارند. **IAM (Identity and Access Management)** کنترل دسترسی است — حیاتی‌ترین لایهٔ امنیت ابر. **CloudFormation** (و در ادامه Terraform که در فایل ۱۵ دیدید) روش رسمی AWS برای IaC است. این فصل هر دو را پیوند می‌زند.

### مفاهیم اساسی

**IAM — هویت و دسترسی:**
- **User:** هویت برای یک شخص/سرویس (مثلاً `alice@company`).
- **Group:** مجموعهٔ کاربران با سیاست یکسان.
- **Role:** هویتی که به آن «نقش» داده می‌شود تا منابع AWS دسترسی موقت بگیرند (مثلاً EC2 اجازه خواندن S3). **Role نه کلید ثابت دارد نه رمز** — امن‌ترین روش.
- **Policy:** سند JSON که «چه کسی، چه عملی، روی چه منبعی» را تعریف می‌کند. دو نوع: Managed (قابل استفاده مجدد) و Inline (مختص یک هویت).
- **Permissions Boundary / MFA / Password Policy:** مکانیزم‌های سخت‌کردن امنیت.
- **Least Privilege:** اصل طلایی — فقط حداقل دسترسی لازم.

**AWS CLI:** ابزار خط فرمان برای تعامل با همهٔ سرویس‌ها (نیازمند تنظیم credentials via `aws configure`).

**CloudFormation (CFN) — IaC بومی AWS:**
- **Template:** فایل YAML/JSON که منابع (Resources)، پارامترها (Parameters)، خروجی‌ها (Outputs) و شرط‌ها (Conditions) را توصیف می‌کند.
- **Stack:** نمونهٔ اجراشده از یک Template — واحد ایجاد/به‌روزرسانی/حذف.
- **Change Set:** پیش‌نمایش تغییرات قبل از اعمال (برای بررسی ریسک).
- **StackSets:** اجرای یک Stack در چندین حساب/Region.
- **Drift Detection:** تشخیص تفاوت بین Template و واقعیت (اگر کسی در کنسول دستی تغییر داد).

**AWS CDK (Cloud Development Kit):** نوشتن زیرساخت با زبان‌های برنامه‌نویسی (TypeScript/Python/Java) که به CFN کامپایل می‌شود — پل بین فایل ۱۶ (Pulumi) و CFN.

### مثال‌های عملی

**الف) سیاست IAM (فقط اجازهٔ خواندن از یک bucket خاص):**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject"],
    "Resource": "arn:aws:s3:::my-app-assets-2026/*"
  }]
}
```
پیوستن به یک Role و انتساب به نمونه (به‌جای کلید ثابت در کد).

**ب) تنظیم اولیه CLI:**
```bash
aws configure
# AWS Access Key ID: ********
# AWS Secret Access Key: ********
# Default region: us-east-1
# Output format: json
```

**ج) Template ساده CloudFormation (YAML) — یک نمونه EC2 + Security Group:**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: نمونهٔ مقدماتی EC2
Parameters:
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: نام Key Pair برای SSH
Resources:
  WebServerSG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: اجازه HTTP/SSH
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 203.0.113.0/24
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t3.micro
      ImageId: ami-0c55b159cbfafe1f0
      KeyName: !Ref KeyName
      SecurityGroups:
        - !Ref WebServerSG
Outputs:
  InstanceId:
    Description: شناسه نمونه
    Value: !Ref WebServer
```
اجرا:
```bash
aws cloudformation create-stack \
  --stack-name my-web-stack \
  --template-body file://ec2.yaml \
  --parameters ParameterKey=KeyName,ParameterValue=devops-key
```

**د) بررسی Drift:**
```bash
aws cloudformation detect-stack-drift --stack-name my-web-stack
```

### مقایسه و دیدگاه‌های مختلف

**User/Key در برابر Role (در EC2):**
- کلید ثابت در کد/نمونه: ریسک نشت، نیاز به چرخش (rotation) دستی.
- Role انتسابی به نمونه: توکن موقت خودکار توسط AWS مدیریت می‌شود → بسیار امن‌تر.
- **قانون:** هرگز کلید دسترسی طولانی‌مدت را در نمونه یا کد نگذارید؛ از Role استفاده کنید.

**CloudFormation در برابر Terraform:**
- CFN: بومی AWS، یکپارچه، رایگان، پشتیبانی کامل از سرویس‌های جدید AWS. قفل شدن روی AWS.
- Terraform (فایل ۱۵): ابزار third-party، multi-cloud، state خارجی، HCL خوانا. برای تیم multi-cloud بهتر.
- **رویکرد عملی:** تیم صرفاً AWS → CFN یا CDK؛ تیم multi-cloud یا با Terraform موجود → Terraform.

**CDK در برابر YAML مستقیم:**
- YAML مستقیم: شفاف، نسخه‌پذیر، بدون نیاز به زبان برنامه‌نویسی.
- CDK: تکرار کمتر، منطق برنامه‌نویسی، اما نیاز به کامپایل و فهم خروجی CFN.

### اشتباهات رایج

1. **استفاده از کاربر Root برای کار روزمره:** حساب Root همه‌چیز را می‌تواند； باید MFA داشته باشد و هرگز برای اتوماسیون استفاده نشود.
2. **دادن `*` در همه جا:** سیاست `Effect: Allow, Action: *, Resource: *` → نقض Least Privilege.
3. **کلیدهای ثابت در کد (Hardcoded):** چک کردن کلید در GitHub → ربات‌های حمله ظرف دقایقی حساب را خالی می‌کنند.
4. **عدم فعال‌سازی MFA:** به‌ویژه روی حساب‌های privileged.
5. **فراموشی حذف Stack:** منابع باقی‌مانده هزینه تولید می‌کنند (مخصوصاً NAT Gateway، EIP).
6. **ویرایش دستی در کنسول پس از CFN:** باعث Drift و سردرگمی در به‌روزرسانی بعدی می‌شود.

### نکات مهم و ترفندها

- **IAM Access Analyzer:** تشخیص دسترسی‌های برون‌سازمانی (مثلاً S3 عمومی) را خودکار کنید.
- **AWS Organizations + SCP:** برای شرکت‌ها، Service Control Policies سقف دسترسی کل حساب‌ها را تعیین می‌کند.
- **Credential Rotation:** کلیدها را هر ۹۰ روز بچرخانید؛ ترجیحاً از Role استفاده کنید که نیازی به چرخش دستی ندارد.
- **CloudFormation Stack Policies:** جلوگیری از حذف/تغییر ناخواسته منابع حیاتی (مثل پایگاه‌داده) هنگام update.
- **Parameter Store / Secrets Manager:** رمزها و تنظیمات را در AWS Systems Manager نگه دارید، نه در Template.
- **Tags Everywhere:** `aws:ResourceTag` در سیاست‌ها برای محدود کردن دسترسی بر اساس برچسب (مثلاً فقط منابع `Env=dev`).

### خلاصه فصل

**IAM** ستون امنیت ابر است: User/Group/Role و Policyهای مبتنی بر JSON با اصل Least Privilege. **Role** (نه کلید ثابت) امن‌ترین راه برای دسترسی منابع به هم است. **CloudFormation** روش بومی AWS برای IaC است — Template → Stack → Change Set → Drift Detection. برای تیم multi-cloud، Terraform (فایل ۱۵) گزینه است؛ برای کد-محور بودن، CDK. اشتباهات کلیدی شامل استفاده از Root، دسترسی `*` و کلید سخت‌کد شده‌اند. با این فایل، پایهٔ ابر و AWS کامل شد؛ در فایل بعدی Azure و GCP را بررسی می‌کنیم تا دید multi-cloud بیابید.

---

## منابع و مراجع

- مستندات رسمی AWS — https://docs.aws.amazon.com
  - What is Cloud Computing: https://aws.amazon.com/what-is-cloud-computing/
  - EC2 User Guide: https://docs.aws.amazon.com/ec2/
  - VPC Developer Guide: https://docs.aws.amazon.com/vpc/
  - S3 Developer Guide: https://docs.aws.amazon.com/s3/
  - IAM User Guide: https://docs.aws.amazon.com/iam/
  - CloudFormation User Guide: https://docs.aws.amazon.com/cloudformation/
- AWS Well-Architected Framework — https://aws.amazon.com/architecture/well-architected/
- AWS Pricing Calculator — https://calculator.aws/
- «Cloud Computing Patterns» (کتاب مرجع معماری ابر)
- آشنایی با مفاهیم Shared Responsibility — https://aws.amazon.com/compliance/shared-responsibility-model/
- مقایسه سرویس‌های محاسباتی — https://aws.amazon.com/products/compute/
- boto3 (SDK پایتون AWS) — https://boto3.amazonaws.com/v1/documentation/api/latest/

**بعدی:** [فایل ۱۹ — آژور و گوگل کلاد (Azure و GCP)](19-Azure-and-GCP.md)
