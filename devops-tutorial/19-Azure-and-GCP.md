# فایل ۱۹ — آژور و گوگل کلاد (Azure و GCP)

**مدت یادگیری**: ۱۰ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۱۸ (مبانی ابر و AWS)، فایل ۱۵ (Terraform برای IaC چندابری)

---

## فهرست محتوا

- [فصل ۹۱: مقدمه‌ای بر Azure و GCP (ساختار و نگاشت مفاهیم)](#فصل-۹۱-مقدمهای-بر-azure-و-gcp-ساختار-و-نگاشت-مفاهیم)
- [فصل ۹۲: محاسبات و ذخیره‌سازی در Azure](#فصل-۹۲-محاسبات-و-ذخیرهسازی-در-azure)
- [فصل ۹۳: محاسبات و ذخیره‌سازی در GCP](#فصل-۹۳-محاسبات-و-ذخیرهسازی-در-gcp)
- [فصل ۹۴: شبکه و مدیریت دسترسی در Azure و GCP](#فصل-۹۴-شبکه-و-مدیریت-دسترسی-در-azure-و-gcp)
- [فصل ۹۵: استراتژی Multi-Cloud و مهاجرت (Migration)](#فصل-۹۵-استراتژی-multi-cloud-و-مهاجرت-migration)

---

## فصل ۹۱: مقدمه‌ای بر Azure و GCP (ساختار و نگاشت مفاهیم)

### مقدمه

در فصل قبل AWS را کامل دیدیم. اما دنیای ابر فقط AWS نیست. **Microsoft Azure** (دومین ابر بزرگ، قدرت در ادغام با اکوسیستم مایکروسافت/سازمانی) و **Google Cloud Platform / GCP** (قوی در داده، تحلیل، Kubernetes — چون گوگل خالق K8s است) دو رقیب اصلی‌اند. درک ساختار سازمانی و نگاشت (mapping) مفاهیم میان این سه، کلید تبدیل شدن به مهندس چندابری (multi-cloud) است.

### مفاهیم اساسی

**ساختار سازمانی Azure:**
- **Tenant (Entra ID / Azure AD):** نمونهٔ دایرکتوری سازمان شما.
- **Subscription (اشتراک):** واحد صورت‌حساب و دسترسی؛ منابع زیر آن ساخته می‌شوند.
- **Resource Group (RG):** ظرف منطقی برای مدیریت چرخهٔ حیات منابع مرتبط.
- **Management Group:** سلسله‌مراتب بالای Subscription برای حکمرانی (governance) سازمانی.
- **Region / Availability Zone:** مشابه AWS.

**ساختار سازمانی GCP:**
- **Organization:** ریشهٔ سلسله‌مراتب (معمولاً مرتبط با دامنه شرکت).
- **Folder:** گروه‌بندی میانی (مثل بخش‌های شرکت).
- **Project:** واحد صورت‌حساب و ایزولاسیون — همهٔ منابع در یک Project ساخته می‌شوند (مهم‌ترین تفاوت با AWS که حساب/Subscription دارد).
- **Region / Zone:** Zone معادل AZ، Region معادل Region.

**نگاشت مفاهیم کلیدی (چیت‌شیت):**

| مفهوم | AWS | Azure | GCP |
|---|---|---|---|
| ذخیره‌سازی اشیاء | S3 | Blob Storage | Cloud Storage |
| ماشین مجازی | EC2 | Virtual Machine | Compute Engine |
| کانتینر مدیریت‌شده | ECS / EKS | AKS / Container Instances | GKE / Cloud Run |
| بدون‌سرور (تابع) | Lambda | Azure Functions | Cloud Functions |
| PaaS وب | Elastic Beanstalk | App Service | App Engine |
| پایگاه‌داده رابطه‌ای | RDS | Azure SQL | Cloud SQL |
| NoSQL | DynamoDB | Cosmos DB | Firestore / Bigtable |
| شبکه خصوصی | VPC | VNet | VPC (global) |
| فایروال نمونه | Security Group | NSG | Firewall Rule |
| DNS | Route53 | Azure DNS | Cloud DNS |
| احراز هویت | IAM | Entra ID + RBAC | Cloud IAM |
| IaC | CloudFormation | ARM / Bicep | Deployment Manager / Terraform |

### مثال‌های عملی

**الف) ایجاد Resource Group در Azure CLI:**
```bash
az group create --name my-rg --location eastus
az vm create \
  --resource-group my-rg \
  --name myvm \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys
```

**ب) ایجاد Project و VM در GCP (gcloud):**
```bash
gcloud projects create my-proj-2026 --name="My Project"
gcloud config set project my-proj-2026
gcloud compute instances create myvm \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --image-family=ubuntu-2204-lts \
  --image-project=ubuntu-os-cloud
```

**ج) تفاوت ساختار — نمای درختی:**
```
AWS:   Account → Region → VPC → Resources
Azure: Tenant → Subscription → RG → Resources
GCP:   Organization → Folder → Project → Resources
```

### مقایسه و دیدگاه‌های مختلف

**فلسفه طراحی:**
- **AWS:** «هر سرویس جداگانه، شما ترکیب کنید» (primitive-rich) — انعطاف بالا، منحنی یادگیری تند.
- **Azure:** «یکپارچه با ابزار سازمانی» (Active Directory، Office، .NET) — راحت برای Enterprise.
- **GCP:** «اول داده و Kubernetes» — بهترین ابزارهای تحلیل (BigQuery) و K8s اصیل (GKE).

**مدل Project/Account:**
- AWS/GCP: چندین حساب/Project برای ایزولاسیون هزینه و دسترسی.
- Azure: Subscription به‌علاوهٔ Management Group برای حکمرانی مرکزی.

**دیدگاه بازار:** تیم‌های .NET/سازمانی معمولاً Azure را ترجیح می‌دهند؛ تیم‌های data/ML و مبتنی بر K8s معمولاً GCP؛ استارتاپ‌های عمومی AWS. اما همهٔ سه ابر امروز همه‌چیز دارند — تفاوت در اکوسیستم و تعرفه است نه قابلیت.

### اشتباهات رایج

1. **فرض یکسان بودن نام‌ها:** مثلاً گمان کردن «VPC در GCP مثل AWS است» در حالی که VPC در GCP **سراسری (global)** است و Subnetها در Zoneها تقسیم می‌شوند.
2. **عدم ایزولاسیون با Project/Subscription:** ریختن همه چیز در یک Project → مشکل در صورت‌حساب و دسترسی.
3. **نادیده گرفتن تفاوت هزینه‌دهی:** ساختار صورت‌حساب در هر ابر متفاوت است؛ تخمین اشتباه.
4. **یادگیری صفر تا صد هر سه به‌طور همزمان:** بهتر است یکی (معمولاً AWS) را عمیق یاد بگیرید، سپس دو تای دیگر را با نگاشت مفاهیم سریع بگیرید.
5. **فراموشی منطق Resource Group در Azure:** حذف RG همهٔ منابع درونش را حذف می‌کند (عملیات تخریبی).

### نکات مهم و ترفندها

- **چیت‌شیت نگاشت:** همیشه یک جدول مقایسهٔ سرویس‌ها دم دست داشته باشید (بالا آورده شد).
- **Cloud Shell:** هر سه ابر یک ترمینال مبتنی بر مرورگر با CLI پیش‌نصب (azure CLI / gcloud / aws cli) می‌دهند — برای یادگیری بی‌نیاز از نصب.
- **Free Tier:** Azure و GCP هم سهمیهٔ رایگان دارند (GCP ۹۰ روز + اعتبار؛ Azure ۱۲ ماه + اعتبار).
- **Terraform Provider چندگانه:** با Terraform (فایل ۱۵) می‌توانید هر سه ابر را با یک زبان مدیریت کنید — کلید multi-cloud.
- **Cloud Pricing آزاد:** استفاده از ابزارهای third-party مثل Infracost برای تخمین هزینه پیش از اجرا.

### خلاصه فصل

Azure و GCP دو ابر بزرگ دیگرند با فلسفه‌های متفاوت (سازمانی/داده‌محور). ساختار سازمانی آن‌ها متفاوت است: Azure بر Tenant→Subscription→RG، GCP بر Organization→Folder→Project استوار است. **نگاشت مفاهیم** (جدول چیت‌شیت) بهترین ابزار برای انتقال دانش از AWS است. نکتهٔ بحرانی: VPC در GCP **سراسری** است. در فصل بعد محاسبات و ذخیره‌سازی تک‌تک را با مثال بررسی می‌کنیم.

---

## فصل ۹۲: محاسبات و ذخیره‌سازی در Azure

### مقدمه

با ساختار Azure آشنا شدیم. حالا به هستهٔ عملیاتی می‌رویم: چگونه یک برنامه را در Azure اجرا و داده‌هایش را ذخیره کنیم. این فصل Virtual Machines، App Service، Functions، AKS و گزینه‌های ذخیره‌سازی/پایگاه‌داده Azure را پوشش می‌دهد.

### مفاهیم اساسی

**محاسبات (Compute):**
- **Azure Virtual Machine (VM):** معادل EC2. خانواده‌هایی مثل `B-series` (اقتصادی/ناپایدار)، `D-series` (عمومی)، `F-series` (محاسباتی).
- **Azure App Service:** PaaS برای میزبانی وب/API بدون مدیریت OS (پشتیبانی از .NET، Node، Python، Java). مقیاس‌بندی خودکار و deployment slots (برای Blue-Green).
- **Azure Functions:** سرورلس تابعی (مثل Lambda)، مدل‌های میزبانی Consumption (پرداخت به‌ازای اجرا) و Premium.
- **Azure Kubernetes Service (AKS):** نسخهٔ مدیریت‌شدهٔ Kubernetes — control plane رایگان؛ شما فقط نودها را مدیریت می‌کنید.
- **Azure Container Instances (ACI):** اجرای سریع یک کانتینر بدون ارکستراسیون (برای jobهای کوتاه).
- **VM Scale Sets:** ASG معادل AWS — خودکار مقیاس‌بندی نمونه‌ها.

**ذخیره‌سازی و پایگاه‌داده:**
- **Blob Storage:** معادل S3 — اشیاء در Container (نه Bucket). لایه‌ها: Hot / Cool / Archive.
- **Azure Disk:** معادل EBS برای VM.
- **Azure Files:** فایل‌سیستم SMB/NFS اشتراکی.
- **Azure SQL Database:** PaaS معادل RDS (SQL Server مدیریت‌شده).
- **Cosmos DB:** پایگاه‌دادهٔ NoSQL جهانی توزیع‌شده (معادل قدرت DynamoDB اما multi-model: document/key-value/graph).
- **Azure Cache for Redis:** معادل ElastiCache.

### مثال‌های عملی

**الف) استقرار وب‌اپلیکیشن با App Service + GitHub Actions (نمای کلی):**
```bash
az webapp up --name my-python-app --runtime "PYTHON:3.11"
# پس از آن در GitHub Actions:
# azure/webapps-deploy@v2  → انتشار خودکار
```

**ب) تابع Azure Functions (Python — پاسخ به HTTP):**
```python
import logging
import azure.functions as func

def main(req: func.HttpRequest) -> func.HttpResponse:
    name = req.params.get('name', 'جهان')
    return func.HttpResponse(f"سلام، {name}!")
```
فایل `function.json` نوع trigger را تعریف می‌کند (http / timer / queue).

**ج) ایجاد AKS و اتصال kubectl:**
```bash
az aks create --resource-group my-rg --name my-aks --node-count 2 --enable-addons monitoring
az aks get-credentials --resource-group my-rg --name my-aks
kubectl get nodes   # همان Kubernetes فصل ۱۱
```

**د) آپلود به Blob Storage (Python SDK):**
```python
from azure.storage.blob import BlobServiceClient
svc = BlobServiceClient.from_connection_string(CONN_STR)
container = svc.get_container_client("assets")
container.upload_blob("logo.png", open("logo.png", "rb"))
```

### مقایسه و دیدگاه‌های مختلف

**App Service در برابر VM:**
- App Service: نگهداری صفر، deployment slots عالی، اما کنترل OS محدود.
- VM: کنترل کامل، اما نگهداری/پچ دستی.

**AKS در برابر Container Instances:**
- ACI: یک کانتینر سریع، بدون K8s — برای job.
- AKS: ارکستراسیون کامل K8s — برای تولید مقیاس‌پذیر.

**Cosmos DB در برابر Azure SQL:**
- Cosmos: NoSQL جهانی، تأخیر میلی‌ثانیه‌ای در هر نقطه، هزینه بالاتر.
- SQL: رابطه‌ای، تراکنش ACID، ارزان‌تر برای دادهٔ ساختاریافته.

**دیدگاه Enterprise:** بسیاری از شرکت‌های مایکروسافتی از App Service + Azure SQL برای سرعت، و AKS برای بارهای مدرن K8s استفاده می‌کنند.

### اشتباهات رایج

1. **استفاده از VM برای همه چیز:** در حالی که App Service/Functions نگهداری کمتری دارند.
2. **انتخاب لایهٔ Blob اشتباه:** نگه داشتن لاگ در Hot (گران) به‌جای Cool/Archive.
3. **فراموشی حذف Resource Group:** هزینهٔ منابع یتیم (orphaned).
4. **عدم استفاده از Managed Identity:** قرار دادن کلیدهای Storage در کد به‌جای هویت مدیریت‌شده.
5. **نادیده گرفتن Reserved Instances:** بار پایدار می‌تواند با تعهد ۱–۳ سال تا ۷۲٪ ارزان‌تر شود.

### نکات مهم و ترفندها

- **Managed Identity:** به‌جای کلید، به منبع هویت بدهید تا خودکار توکن بگیرد (معادل IAM Role در AWS).
- **Deployment Slots:** در App Service یک slot «staging» داشته باشید و با «swap» بدون خواباندن سرویس منتشر کنید (Blue-Green ساده).
- **Azure Policy:** برای اجبار کردن برچسب‌گذاری / منطقهٔ مجاز در سطح سازمان.
- **Bicep:** زبان DSL خوانا برای IaC در Azure (جایگزین JSON خسته‌کنندهٔ ARM Templates) — مشابه Terraform اما بومی آژور.
- **Cost Management + Budgets:** داشبورد رایگان آژور برای کنترل هزینه.

### خلاصه فصل

Azure طیفی از محاسبات از VM (کنترل کامل) تا App Service/Functions (PaaS/سرورلس) و AKS (K8s مدیریت‌شده) ارائه می‌دهد. در ذخیره‌سازی، Blob (مثل S3)، Disk، Files و در پایگاه‌داده Azure SQL و Cosmos DB (NoSQL جهانی) حضور دارند. **Managed Identity** و **Deployment Slots** دو قابلیت طلایی آژور برای امنیت و انتشار بدون خواب هستند. **Bicep** راه ابری برای IaC است. در فصل بعد همین موضوع را در GCP بررسی می‌کنیم.

---

## فصل ۹۳: محاسبات و ذخیره‌سازی در GCP

### مقدمه

گوگل کلاد (GCP) خالق Kubernetes و BigQuery است؛ بنابراین در دو حوزهٔ کانتینر و تحلیل داده بسیار قدرتمند است. این فصل محاسبات (Compute Engine، Cloud Run، GKE، Functions، App Engine) و ذخیره‌سازی/پایگاه‌داده (Cloud Storage، Cloud SQL، Firestore، BigQuery) GCP را با مثال بررسی می‌کند.

### مفاهیم اساسی

**محاسبات (Compute):**
- **Compute Engine:** معادل EC2 — VM با کنترل کامل. نمونه‌های پیشemptible (مثل Spot) بسیار ارزان.
- **Google Kubernetes Engine (GKE):** بهترین نسخهٔ مدیریت‌شدهٔ K8s (چون گوگل سازندهٔ K8s است) — ویژگی‌های پیشرفته مثل Autopilot (مدیریت کامل نودها).
- **Cloud Run:** اجرای کانتینرهای بدون‌سرور (مثل Fargate/Lambda) — مقیاس به صفر (scale-to-zero) که هزینه را به حداقل می‌رساند. بسیار محبوب.
- **Cloud Functions:** توابع سرورلس (مثل Lambda) پاسخ به رویداد.
- **App Engine:** PaaS قدیمی‌تر گوگل برای وب‌اپلیکیشن.
- **Instance Groups:** ASG معادل — مقیاس‌بندی خودکار VM.

**ذخیره‌سازی و پایگاه‌داده:**
- **Cloud Storage:** معادل S3 — Buckets با کلاس‌های Standard / Nearline / Coldline / Archive.
- **Persistent Disk:** معادل EBS.
- **Cloud SQL:** PaaS برای PostgreSQL/MySQL/SQL Server (معادل RDS).
- **Firestore:** پایگاه‌دادهٔ NoSQL سندمحور (معادل DynamoDB) — real-time و offline-first.
- **Bigtable:** NoSQL عظیم‌مقیاس (columnar) برای IoT/تحلیل.
- **BigQuery:** انبار دادهٔ (Data Warehouse) سرورلس برای تحلیل SQL روی پتابایت داده — نقطهٔ قوت بی‌رقیب GCP.
- **Memorystore:** Redis مدیریت‌شده.

### مثال‌های عملی

**الف) استقرار در Cloud Run (از image):**
```bash
gcloud run deploy my-api \
  --image gcr.io/my-proj-2026/my-api:latest \
  --region us-central1 \
  --allow-unauthenticated
# نتیجه: URL عمومی با مقیاس‌بندی خودکار و scale-to-zero
```

**ب) ایجاد GKE Autopilot:**
```bash
gcloud container clusters create-auto my-cluster \
  --region us-central1
gcloud container clusters get-credentials my-cluster --region us-central1
kubectl get nodes
```

**ج) آپلود به Cloud Storage (Python — google-cloud-storage):**
```python
from google.cloud import storage
client = storage.Client()
bucket = client.bucket("my-assets-2026")
blob = bucket.blob("reports/q1.csv")
blob.upload_from_filename("q1.csv")
print("آپلود شد:", blob.public_url)
```

**د) پرس‌وجوی BigQuery (تحلیل روی دادهٔ عمومی):**
```sql
SELECT country, COUNT(*) AS cnt
FROM `bigquery-public-data.weather_geo.gsod`
GROUP BY country
ORDER BY cnt DESC
LIMIT 10;
```
اجرا با `bq query` یا کنسول — بدون نیاز به زیرساخت.

### مقایسه و دیدگاه‌های مختلف

**Cloud Run در برابر GKE:**
- Cloud Run: کانتینر بدون‌سرور، scale-to-zero، ساده، برای microservices سبک ایده‌آل.
- GKE: کنترل کامل K8s، برای سیستم پیچیده با ده‌ها سرویس.

**Firestore در برابر Cloud SQL:**
- Firestore: NoSQL real-time، مناسب اپلیکیشن موبایل/وب پویا.
- Cloud SQL: رابطه‌ای، برای سیستم‌های سنتی/تراکنشی.

**BigQuery در برابر سایر:** تفاوت بنیادی — BigQuery «انبار دادهٔ سرورلس» است؛ شما برای پرس‌وجو پرداخت می‌کنید نه برای سرور. رقبا (Redshift/Azure Synapse) مشابه‌اند اما GCP پیشرو در سادگی است.

**دیدگاه مدرن:** بسیاری تیم‌ها ترکیب «Cloud Run + Firestore + BigQuery» را برای سرعت نوآوری ترجیح می‌دهند.

### اشتباهات رایج

1. **فراموشی scale-to-zero در Cloud Run:** در حالی که هزینه را حذف می‌کند، cold start اولین درخواست را کُند می‌کند — برای بار حساس به latency تنظیم min-instances کنید.
2. **عدم حذف Project در GCP:** برخلاف AWS که Resource را جدا حذف می‌کنید، در GCP بسیاری هزینه را با حذف Project کنترل می‌کنند.
3. **انتخاب غلط کلاس Storage:** نگه داشتن آرشیو در Standard.
4. **استفاده از کلید حساب سرویس (Service Account Key) در کد:** به‌جای Workload Identity (معادل Role).
5. **نادیده گرفتن Quotas:** GCP Quotaهای پیش‌فرض محافظه‌کارانه دارد — برای تولید درخواست افزایش دهید.

### نکات مهم و ترفندها

- **Workload Identity:** روش امن اتصال K8s به سرویس‌های GCP بدون کلید (معادل IRSA در AWS و Pod Identity در Azure).
- **Cloud Run + Eventarc:** ترکیب Cloud Run با رویدادهای Pub/Sub، Storage و غیره برای معماری رویدادمحور.
- **Preemptible/Spot VMs:** تا ۸۰٪ ارزان‌تر برای batch jobهای fault-tolerant.
- **gcloud config configurations:** مدیریت چند پروفایل پروژه/حساب به‌راحتی.
- **Deployment Manager / Terraform:** برای IaC در GCP، Terraform (فایل ۱۵) محبوب‌تر از Deployment Manager بومی است.

### خلاصه فصل

GCP در کانتینر (GKE اصیل، Cloud Run بی‌رقیب با scale-to-zero) و تحلیل داده (BigQuery) می‌درخشد. Compute Engine، Cloud Functions، App Engine گزینه‌های دیگرند. ذخیره‌سازی شامل Cloud Storage (مثل S3)، Persistent Disk، Cloud SQL، Firestore (NoSQL) و Bigtable می‌شود. **Workload Identity** و **Cloud Run** قابلیت‌های کلیدی‌اند. با این فصل، هر سه ابر در لایهٔ محاسبات/ذخیره‌سازی پوشش داده شد. فصل بعد شبکه و دسترسی را در هر دو بررسی می‌کنیم.

---

## فصل ۹۴: شبکه و مدیریت دسترسی در Azure و GCP

### مقدمه

امنیت و ارتباط هر ابر به لایه‌های شبکه و احراز هویت بستگی دارد. این فصل تفاوت‌های ظریف VNet (Azure) و VPC (GCP — که سراسری است)، فایروال‌ها، بارمتعادل‌کننده‌ها و سیستم‌های IAM/هویت را در دو ابر بررسی می‌کند.

### مفاهیم اساسی

**Azure Networking:**
- **VNet (Virtual Network):** معادل VPC اما درون یک Region. Subnetها درون آن.
- **NSG (Network Security Group):** فایروال stateful در سطح Subnet یا NIC (ترکیب SG+NACL آمازون).
- **Azure Firewall / Application Gateway:** فایروال مدیریت‌شده و L7 LB با WAF.
- **Azure Load Balancer (L4) / Application Gateway (L7).**
- **Azure Front Door:** CDN + WAF سراسری (معادل CloudFront+ALB).
- **Azure DNS:** سرویس DNS مدیریت‌شده.
- **ExpressRoute:** لینک اختصاصی (مثل Direct Connect).
- **Peering:** اتصال VNetها (مثل VPC Peering).

**Azure Identity:**
- **Microsoft Entra ID** (سابق Azure Active Directory): دایرکتوری هویت سازمان.
- **RBAC (Role-Based Access Control):** نقش‌ها (Built-in مثل Contributor/Owner/Reader) روی Scope (RG/Subscription).
- **Managed Identity:** هویت برای منابع جهت دسترسی بدون کلید.
- **Service Principal:** هویت برای اپلیکیشن/CI (معادل IAM User).

**GCP Networking:**
- **VPC (Global):** یک VPC در سراسر Regionها پخش می‌شود؛ Subnetها در Zoneها تعریف می‌شوند (تفاوت کلیدی با AWS/Azure).
- **Firewall Rules:** stateful، در سطح VPC (مثل SG).
- **Cloud Load Balancing:** سراسری (بدون نمونهٔ LB مجزا) — HTTP(S)/TCP/UDP.
- **Cloud CDN:** با backendهای Cloud Storage/VM.
- **Cloud DNS:** DNS مدیریت‌شده.
- **Cloud NAT:** خروج private بدون IP عمومی (مثل NAT Gateway).
- **Cloud Interconnect / VPN:** لینک اختصاصی/رمزنگاری‌شده.

**GCP IAM:**
- **Roles (Primitive: Owner/Editor/Viewer; Predefined; Custom).**
- **Service Accounts:** هویت برای سرویس/VM (معادل Role در AWS).
- **IAM Bindings:** چه کسی + چه نقشی + روی چه منبعی (شبیه Policy آمازون).
- **Workload Identity Federation:** اتصال بدون کلید از GitHub Actions/other cloud.

### مثال‌های عملی

**الف) NSG در Azure (اجازه فقط HTTP):**
```bash
az network nsg rule create \
  --resource-group my-rg --nsg-name my-nsg \
  --name AllowHTTP --protocol Tcp --port 80 \
  --source-address-prefixes '*' --access Allow --priority 100
```

**ب) Firewall Rule در GCP (اجازه SSH فقط از IP شرکت):**
```bash
gcloud compute firewall-rules create allow-ssh \
  --network my-vpc \
  --allow tcp:22 \
  --source-ranges 203.0.113.0/24
```

**ج) نقش RBAC در Azure (اختصاص Contributor به یک کاربر روی RG):**
```bash
az role assignment create \
  --assignee alice@company.com \
  --role Contributor \
  --resource-group my-rg
```

**د) Service Account در GCP برای VM:**
```bash
gcloud iam service-accounts create my-sa --display-name "My SA"
gcloud projects add-iam-policy-binding my-proj-2026 \
  --member "serviceAccount:my-sa@my-proj-2026.iam.gserviceaccount.com" \
  --role "roles/storage.objectViewer"
```

### مقایسه و دیدگاه‌های مختلف

**مدل شبکه:**
- AWS/Azure: VPC/VNet درون Region؛ Subnet درون Zone/AZ.
- GCP: VPC **سراسری**؛ Subnet در Zone — طراحی ساده‌تر برای multi-region.

**مدل فایروال:**
- AWS: تفکیک SG (stateful، نمونه) و NACL (stateless، subnet).
- Azure: NSG هر دو سطح را پوشش می‌دهد.
- GCP: Firewall Rules در سطح VPC، stateful.

**هویت:**
- AWS: IAM User/Role/Policy جداگانه.
- Azure: Entra ID + RBAC با Scope.
- GCP: IAM با bindings + Service Accounts.
- هر سه ایدهٔ «هویت برای منابع (Role/MI/SA) بدون کلید ثابت» را توصیه می‌کنند.

### اشتباهات رایج

1. **فرض VPC محلی در GCP:** گمان کردن VPC در GCP مثل AWS منحصر به Region است — در حالی که سراسری است.
2. **باز گذاشتن NSG/Firewall روی ۰.۰.۰.۰/۰:** مشابه AWS، بزرگ‌ترین ریسک.
3. **استفاده از کلید Service Account در GCP:** به‌جای Workload Identity Federation.
4. **دادن نقش Owner وسیع در Azure:** به‌جای Least Privilege با Scope محدود.
5. **فراموشی Private Google Access:** VM در subnet private برای دسترسی به API گوگل نیاز به تنظیم دارد.

### نکات مهم و ترفندها

- **Azure Private Link / GCP Private Service Connect:** دسترسی به سرویس‌های مدیریت‌شده بدون خروج به اینترنت (مثل PrivateLink آمازون).
- **Defender for Cloud / Security Command Center:** داشبورد امنیت بومی هر ابر برای شناسایی پیکربندی ناایمن.
- **Azure Policy / GCP Organization Policy:** اجبار قوانین سازمانی (مثل «فقط رمزنگاری شده»).
- **Service Account Impersonation در GCP:** CI را با توکن موقت اجرا کنید بدون کلید طولانی‌مدت.
- **Hub-and-Spoke:** مدل شبکهٔ مرکزی برای چند VNet/Project در سازمان (Transit Gateway معادل).

### خلاصه فصل

در Azure، **VNet/NSG/Entra ID+RBAC** و در GCP، **VPC سراسری/Firewall Rules/IAM+Service Accounts** ساختار شبکه و دسترسی را می‌سازند. تفاوت بنیادی GCP در VPC سراسری و تفکیک به‌ظاهر ساده‌تر فایروال است. در هر سه ابر، «هویت بدون کلید» (Role/Managed Identity/Service Account) اصل امنیت است. ابزارهای بومی مثل Defender/Security Command Center و Organization Policy به حکمرانی کمک می‌کنند. اکنون آماده می‌شویم برای استراتژی چندابری در فصل بعد.

---

## فصل ۹۵: استراتژی Multi-Cloud و مهاجرت (Migration)

### مقدمه

بسیاری سازمان‌ها امروز در بیش از یک ابر حضور دارند — چه از روی تصادف (خرید شرکت دیگر) چه استراتژی. اما multi-cloud هم فرصت است هم تله. این فصل دلایل، چالش‌ها، ابزارهای یکپارچه (مثل Terraform) و رویکردهای مهاجرت (6Rs) را بررسی می‌کند تا تصمیم‌گیری آگاهانه داشته باشید.

### مفاهیم اساسی

**چرا Multi-Cloud؟**
- **جلوگیری از Lock-in:** وابستگی به یک فروشنده کاهش می‌یابد.
- **بهترینِ هر ابر:** مثلاً BigQuery در GCP برای تحلیل + AWS برای اکوسیستم سرویس.
- **تداوم کسب‌وکار (BC/DR):** افتادگی یک ابر کل سیستم را از کار نمی‌اندازد.
- **مقررات/حاکمیت:** داده در ابری که منطقهٔ قانونی را رعایت کند.
- **مذاکره قدرت:** اهرم چانه‌زنی با فروشندگان.

**چالش‌ها:**
- **پیچیدگی عملیاتی:** یادگیری سه ابر، سه کنسول، سه مدل هزینه.
- **انتقال داده (Egress) هزینه:** جابه‌جایی داده میان ابرها گران است.
- **مهارت:** کمبود مهندس چندابری.
- **ابزار یکپارچه:** نیاز به لایهٔ انتزاعی (Terraform، Kubernetes، Service Mesh).

**ابزارهای یکپارچه (Abstraction):**
- **Terraform (فایل ۱۵):** یک زبان برای هر سه ابر (provider aws/azurerm/google).
- **Kubernetes:** اجرای بار کانتینری یکسان در EKS/AKS/GKE.
- **Crossplane (فایل ۱۶):** مدیریت منابع ابری با API کوبرنتیز.
- **GitOps (فایل ۱۴):** ArgoCD/Flux روی هر ابر یکسان.
- **OpenTelemetry (فایل ۲۲):** جمع‌آوری تله‌متری یکسان.

**استراتژی‌های مهاجرت (The 6 Rs):**
1. **Rehost (Lift & Shift):** کپی ساده به ابر (سریع، کم‌بهینه).
2. **Replatform:** تغییر جزئی (مثلاً پایگاه‌داده به Managed).
3. **Refactor / Rearchitect:** بازنویسی برای بهره‌وری ابری (مثل microservices).
4. **Repurchase:** تعویض با SaaS (مثل جایگزینی با Salesforce).
5. **Retain:** نگه داشتن on-prem (فعلی).
6. **Retire:** حذف برنامهٔ غیرضروری.

**Landing Zone:** چارچوب پیش‌پیکربندی‌شده (حساب/شبکه/IAM/هزینه) قبل از مهاجرت واقعی.

**FinOps:** رویکرد مالی برای بهینه‌سازی هزینهٔ ابر در تیم‌های چندابری.

### مثال‌های عملی

**الف) Terraform چندابری (پروژهٔ یکسان، دو ارائه‌دهنده):**
```hcl
# main.tf
terraform {
  required_providers {
    aws     = { source = "hashicorp/aws" }
    google  = { source = "hashicorp/google" }
  }
}
provider "aws"   { region = "us-east-1" }
provider "google" { project = "my-proj-2026" region = "us-central1" }

resource "aws_s3_bucket" "assets" { bucket = "shared-assets-2026" }
resource "google_storage_bucket" "backup" {
  name     = "shared-backup-2026"
  location = "US"
}
```

**ب) بار یکسان در هر سه ابر با Kubernetes:**
```
Docker image یکسان →
  AWS: EKS  /  Azure: AKS  /  GCP: GKE
همه با همان Manifestهای kubectl → انتقال آسان
```

**ج) انتخاب استراتژی (ماتریس تصمیم):**
```
برنامه legacy ساده     → Rehost (سریع)
برنامه با پایگاه‌داده → Replatform (RDS/Cloud SQL)
برنامه مقیاس‌پذیر جدید → Refactor (microservices + K8s)
نرم‌افزار تکراری       → Repurchase (SaaS)
داده حساس/قدیمی       → Retain (on-prem)
ابزار منسوخ          → Retire
```

### مقایسه و دیدگاه‌های مختلف

**Single-Cloud در برابر Multi-Cloud:**

| | Single-Cloud | Multi-Cloud |
|---|---|---|
| پیچیدگی | کم | زیاد |
| هزینه عملیاتی | کمتر | بیشتر |
| قفل شدن | بله | کمتر |
| انعطاف | کم | زیاد |
| سرعت نوآوری | سریع‌تر (تمرکز) | کُندتر (پراکندگی) |

**دیدگاه‌ها:**
- **موافقان:** انعطاف، قدرت چانه‌زنی، جلوگیری از فاجعهٔ تک‌نقطه‌ای.
- **منتقدان:** بیشتر تیم‌ها صرفاً به دلیل «مد شدن» multi-cloud می‌روند در حالی که هزینه/پیچیدگی آن را مدیریت نمی‌کنند.
- **رویکرد عملی:** «ابتدا در یک ابر عمیق شوید؛ multi-cloud را فقط برای نیاز واقعی (نه ترس از lock-in صرف) اتخاذ کنید، و از لایهٔ انتزاعی مثل Terraform/K8s استفاده کنید».

### اشتباهات رایج

1. **Multi-cloud بدون دلیل استراتژیک:** فقط به خاطر «ترند» → پیچیدگی بی‌دلیل.
2. **نادیده گرفتن Egress cost:** جابه‌جایی داده میان ابرها هزینهٔ سنگینی دارد.
3. **تکرار دانش ابری در هر تیم:** بدون مرکزیت ابزار (Terraform/K8s) → هرج‌ومرج.
4. **Lift & Shift ضعیف:** کپی برنامهٔ on-prem بدون بهینه‌سازی → هزینهٔ ابری بیشتر از on-prem.
5. **عدم وجود Landing Zone:** شروع مهاجرت بدون چارچوب حکمرانی → بدهی فنی امنیتی.

### نکات مهم و ترفندها

- **قانون ۸۰/۲۰:** ۸۰٪ بار را در یک ابرِ اصلی نگه دارید، ۲۰٪ ویژه را (اگر لزوم دارد) در ابر دوم.
- **Terraform Modules چندابری:** ماژول‌هایی بنویسید که با متغیر `cloud_provider` بین aws/azure/gcp سوییچ کنند.
- **Kubernetes به عنوان لایهٔ انتزاع:** بار کانتینری را «ابر-ناآگاه» نگه دارید.
- **FinOps Culture:** مالکیت هزینه را به تیم‌های محصول بدهید، نه فقط زیرساخت.
- **Cloud Asset Inventory:** ابزارهایی مثل Azure Migrate / GCP Migration Center / AWS Migration Hub برای نقشه‌برداری قبل از مهاجرت.
- **Pilot قبل از انتقال کامل:** یک برنامهٔ غیرحیاتی را اول مهاجرت کنید (Proof of Concept).

### خلاصه فصل

Multi-cloud فرصتی برای بهره‌برداری از بهترین هر ابر و کاهش lock-in است، اما پیچیدگی و هزینهٔ عملیاتی دارد. کلید موفقیت: **لایهٔ انتزاعی** (Terraform برای IaC، Kubernetes برای بار، GitOps برای انتشار، OpenTelemetry برای تله‌متری) و **تصمیم استراتژیک** نه احساسی. مهاجرت با **6 Rs** (Rehost تا Retire) و **Landing Zone** آغاز می‌شود؛ **FinOps** هزینه را کنترل می‌کند. توصیهٔ عملی: ابتدا یک ابر را عمیق بیاموزید، سپس چندابری را هدفمند اتخاذ کنید. با این فایل، بخش ابر (فایل‌های ۱۸–۱۹) کامل شد و در فایل بعدی به Observability (Prometheus/Grafana) می‌رویم.

---

## منابع و مراجع

- Microsoft Azure Documentation — https://learn.microsoft.com/azure/
  - Virtual Machines: https://learn.microsoft.com/azure/virtual-machines/
  - App Service: https://learn.microsoft.com/azure/app-service/
  - AKS: https://learn.microsoft.com/azure/aks/
  - Azure Storage: https://learn.microsoft.com/azure/storage/
  - Entra ID / RBAC: https://learn.microsoft.com/entra/
  - Bicep: https://learn.microsoft.com/azure/azure-resource-manager/bicep/
- Google Cloud Documentation — https://cloud.google.com/docs
  - Compute Engine: https://cloud.google.com/compute/docs
  - GKE: https://cloud.google.com/kubernetes-engine/docs
  - Cloud Run: https://cloud.google.com/run/docs
  - Cloud Storage: https://cloud.google.com/storage/docs
  - BigQuery: https://cloud.google.com/bigquery/docs
  - IAM: https://cloud.google.com/iam/docs
- مقایسهٔ سرویس‌ها — https://cloud.google.com/docs/get-started/aws-azure-gcp-service-comparison
- AWS-to-Azure/GCP mapping guides (مستندات رسمی هر ابر)
- «Cloud FinOps» اثر J.R. Mesili و همکاران (مرجع مدیریت هزینهٔ ابری)
- Terraform Multi-Cloud (فایل ۱۵ این مجموعه)

**بعدی:** [فایل ۲۰ — Prometheus و Grafana (Observability بخش ۱)](20-Prometheus-and-Grafana.md)
