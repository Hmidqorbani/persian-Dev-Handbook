# رایانش ابری (Cloud Computing)

**مدت یادگیری**: ۴ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: فایل ۱۵ (وب) و ۰۲ (شبکه)

---

## فهرست محتوا

- [فصل ۱: مقدمه رایانش ابری](#فصل-۱-مقدمه-رایانش-ابری)
- [فصل ۲: مدل مسئولیت مشترک](#فصل-۲-مدل-مسئولیت-مشترک)
- [فصل ۳: حملات ابری](#فصل-۳-حملات-ابری)
- [فصل ۴: امنیت کانتینر و Kubernetes](#فصل-۴-امنیت-کانتینر-و-kubernetes)
- [فصل ۵: آزمایشگاه و اقدامات متقابل](#فصل-۵-آزمایشگاه-و-اقدامات-متقابل)

---

## فصل ۱: مقدمه رایانش ابری

### مقدمه

**رایانش ابری** یعنی استفاده از منابع محاسباتی (سرور، ذخیره‌سازی، دیتابیس) از طریق اینترنت به‌صورت اجاره‌ای. ابرامنیت یکی از پرتقاضاترین حوزه‌های شغلی است.

> **تشبیه:** به‌جای خرید و نگهداری خودرو، از تاکسی/اجاره استفاده می‌کنید. کنترل کمتر، اما انعطاف بیشتر.

### مفاهیم اساسی

**مدل‌های سرویس:**

| مدل | ارائه‌دهنده مدیریت | مشتری مدیریت |
|-----|-------------------|--------------|
| **IaaS** | زیرساخت | OS، اپ، داده |
| **PaaS** | زیرساخت + OS + Runtime | اپ، داده |
| **SaaS** | همه‌چیز | فقط داده |

**مدل‌های استقرار:**
- **Public:** AWS، Azure، GCP (اشتراکی).
- **Private:** اختصاصی سازمان.
- **Hybrid:** ترکیب عمومی و خصوصی.
- **Community:** اشتراکی بین چند سازمان.

**ارائه‌دهندگان اصلی:** AWS، Microsoft Azure، Google Cloud، Alibaba Cloud.

### نکات مهم (Tips & Tricks)

- **Misconfiguration** بزرگ‌ترین خطر ابر است (نه خود پلتفرم).
- **S3 Bucket باز** یکی از رایج‌ترین نشت‌های داده است.
- ابر **امن** است اما **پیکربندی نادرست** ناامن می‌شود.

### خلاصه فصل

- مدل‌ها: IaaS، PaaS، SaaS.
- استقرار: Public، Private، Hybrid، Community.
- Misconfiguration خطر اصلی است.

---

## فصل ۲: مدل مسئولیت مشترک

### مقدمه

در ابر، امنیت بین ارائه‌دهنده و مشتری **تقسیم** می‌شود. فهم این مرز حیاتی است.

### مفاهیم اساسی

**Shared Responsibility Model:**

```
┌─────────────────────────────────────────┐
│  مشتری مسئول:                            │
│  - داده و رمزنگاری                        │
│  - مدیریت دسترسی (IAM)                    │
│  - پیکربندی اپ و شبکه                     │
│  - سیستم‌عامل (در IaaS)                   │
├─────────────────────────────────────────┤
│  ارائه‌دهنده مسئول:                        │
│  - زیرساخت فیزیکی                         │
│  - شبکه، مجازی‌سازی                       │
│  - امنیت پلتفرم ابری                      │
└─────────────────────────────────────────┘
```

**نکته کلیدی:** ارائه‌دهنده ابر را امن می‌کند، اما **محتوای شما** مسئولیت شماست.

**ابزارهای امنیتی ابری:**
- **IAM:** مدیریت هویت و دسترسی.
- **Security Groups / NACL:** فایروال.
- **KMS:** مدیریت کلید.
- **CloudTrail/Activity Log:** لاگ.
- **WAF / Shield:** حفاظت.

### مثال عملی

```bash
# === AWS CLI ===
# پیکربندی
aws configure

# بررسی کاربر و مجوزها
aws sts get-caller-identity
aws iam list-users
aws iam list-attached-user-policies --user-name <user>

# بررسی S3 Bucketها (عمومی؟)
aws s3 ls
aws s3api get-bucket-acl --bucket <bucket>
aws s3api get-bucket-policy --bucket <bucket>

# بررسی Security Groups
aws ec2 describe-security-groups

# بررسی CloudTrail
aws cloudtrail lookup-events
```

### اشکال شایع

- **مشکل:** فکر می‌کنم ابر مسئول همه‌چیز است → نه؛ پیکربندی مسئولیت شماست.
- **مشکل:** IAM بیش از حد باز است → اصل Least Privilege را رعایت کنید.

### نکات مهم (Tips & Tricks)

- **IAM Misconfiguration** ریشه بسیاری از نشت‌هاست.
- **Wildcard (*)** در IAM policy = خطر.
- **MFA** برای همه حساب‌های ابری الزامی است.
- **CloudTrail** را فعال نگه دارید.

### خلاصه فصل

- امنیت ابر = مسئولیت مشترک.
- مشتری مسئول داده، IAM و پیکربندی است.
- IAM، KMS، CloudTrail ابزارهای کلیدی.

---

## فصل ۳: حملات ابری

### مقدمه

حملات ابری اغلب **پیکربندی نادرست** و **مدیریت هویت** را هدف می‌گیرند.

### مفاهیم اساسی

**حملات رایج:**
- **Misconfiguration:** S3 باز، Security Group باز، IAM باز.
- **SSRF به Metadata:** دسترسی به `169.254.169.254`.
- **IAM Privilege Escalation:** ارتقا از کاربر به ادمین.
- **Credential Leakage:** افشای کلیدها (در کد، Git).
- **Account Hijacking:** سرقت حساب.
- **Data Breach:** نشت داده از Bucket/DB باز.
- **Serverless Attacks:** تزریق در Lambda/Functions.
- **Container Escape:** فرار از کانتینر.

### مثال عملی

```bash
# === SSRF به Metadata ابری ===
# از یک آسیب‌پذیری SSRF:
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>
# → کلیدهای موقت IAM!

# === کشف Bucket عمومی ===
# ابزارها: S3Scanner, cloud_enum
pip install cloud_enum
cloud_enum -k companyname

# === جستجوی کلیدهای لو رفته ===
# ابزار: trufflehog, gitleaks
trufflehog github --org=companyname

# === Pacu (چارچوب تست AWS) ===
pip install pacu
pacu
# > import_keys
# > run iam__enum_permissions
# > run iam__privesc_scan

# === ScoutSuite (ارزیابی ابری) ===
scout aws
# → گزارش جامع پیکربندی
```

### اشکال شایع

- **مشکل:** کلیدها را در Git لو می‌دهید → از Secret Manager استفاده کنید.
- **مشکل:** SSRF فیلتر شده → از تکنیک‌های Bypass یا IMDSv2 استفاده کنید.
- **مشکل:** Account Lockout → از API keyهای اضافی استفاده کنید.

### نکات مهم (Tips & Tricks)

- **IMDSv2** (Token-based) جلوی SSRF به Metadata را می‌گیرد.
- **Least Privilege** در IAM حیاتی است.
- **cloud_enum** و **S3Scanner** برای کشف دارایی‌های عمومی.
- **Pacu** و **ScoutSuite** ابزارهای تست ابر هستند.
- کلیدها را **هرگز** در کد قرار ندهید.

### خلاصه فصل

- حملات: Misconfig، SSRF، IAM PrivEsc، Credential Leak.
- Metadata endpoint (169.254.169.254) هدف کلیدی است.
- ابزارها: Pacu، ScoutSuite، cloud_enum، trufflehog.

---

## فصل ۴: امنیت کانتینر و Kubernetes

### مقدمه

**Docker** و **Kubernetes** استاندارد استقرار مدرن‌اند و آسیب‌پذیری‌های خاص خود را دارند.

### مفاهیم اساسی

**مفاهیم کانتینر:**
- **Image:** قالب کانتینر.
- **Container:** نمونه در حال اجرا.
- **Registry:** مخزن Image (Docker Hub).
- **Dockerfile:** دستور ساخت Image.

**مفاهیم Kubernetes:**
- **Pod:** کوچک‌ترین واحد.
- **Node:** ماشین میزبان.
- **Cluster:** مجموعه Nodeها.
- **kubelet / API Server:** اجزای اصلی.
- **RBAC:** کنترل دسترسی.

**خطرات کانتینر:**
- Image آلوده (Base Image قدیمی).
- کانتینر با دسترسی روت.
- **Container Escape** به میزبان.
- Secret لو رفته.
- Kubernetes API باز.

### مثال عملی

```bash
# === تحلیل Image ===
docker images
docker history <image>
trivy image <image>          # اسکن آسیب‌پذیری
# یا: docker scan <image>

# === بررسی کانتینر در حال اجرا ===
docker ps
docker exec -it <container> sh
# داخل کانتینر:
id                          # کاربر
cat /etc/shadow             # اگر روت باشد
ls /var/run/docker.sock     # اگر mount باشد → خطر!

# === Container Escape (مثال آموزشی) ===
# اگر docker.sock mount باشد:
docker -H unix:///var/run/docker.sock ps
# → کنترل کامل Docker daemon!

# === Kubernetes ===
kubectl get pods
kubectl get secrets
kubectl auth can-i --list   # بررسی مجوزها
kubectl get clusterrolebindings

# === ابزارها ===
kube-hunter --remote <cluster>   # تست امنیت K8s
kubeaudit
# Trivy برای اسکن
trivy k8s --report summary cluster
```

### اشکال شایع

- **مشکل:** Container Escape موفق نشد → نیاز به misconfig خاص (privileged، socket mount).
- **مشکل:** Kubernetes API محافظت‌شده → نیاز به Credential.
- **مشکل:** Image آلوده شناسایی نشد → از Trivy/Clair استفاده کنید.

### نکات مهم (Tips & Tricks)

- **هرگز docker.sock را در کانتینر mount نکنید.**
- کانتینر را با کاربر **غیرروت** اجرا کنید.
- **Base Image** را به‌روز و Minimal نگه دارید.
- **RBAC** سخت‌گیرانه در Kubernetes.
- **Network Policy** برای محدودسازی ترافیک Pod.
- **Trivy** و **kube-hunter** ابزارهای ضروری.

### خلاصه فصل

- کانتینر و Kubernetes آسیب‌پذیری خاص دارند.
- Container Escape و docker.sock خطر بزرگ‌اند.
- RBAC و Image Scanning دفاع‌های اصلی.

---

## فصل ۵: آزمایشگاه و اقدامات متقابل

### مقدمه

یک آزمایشگاه ابری/کانتینر بسازیم.

### مثال عملی — آزمایشگاه

```bash
# === کانتینر ===
# ۱. نصب Docker
sudo apt install -y docker.io

# ۲. اجرای یک اپ آسیب‌پذیر (OWASP Juice Shop)
sudo docker run -d -p 3000:3000 bkimminich/juice-shop

# ۳. اسکن Image با Trivy
trivy image bkimminich/juice-shop

# === Kubernetes (با Minikube) ===
# ۱. نصب Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# ۲. اجرای cluster
minikube start

# ۳. بررسی
kubectl get nodes
kubectl get pods --all-namespaces

# ۴. تست امنیت
kube-hunter --remote <cluster-ip>

# === AWS (با حساب آزمایشی) ===
# از AWS Free Tier برای تمرین استفاده کنید
# ابزارها: ScoutSuite, Pacu, aws-cli
```

### اشکال شایع

- **مشکل:** Minikube منابع زیادی می‌خواهد → RAM کافی اختصاص دهید.
- **مشکل:** AWS هزینه دارد → از Free Tier و منابع محدود استفاده کنید.

### نکات مهم (Tips & Tricks)

- **Juice Shop** روی Docker برای تمرین وب.
- **Minikube** برای یادگیری Kubernetes.
- **LocalStack** شبیه‌ساز ابر محلی (AWS).
- همیشه محیط تمرین را از محیط واقعی جدا نگه دارید.

### اقدامات متقابل (Countermeasures)

| تهدید | راهکار |
|-------|--------|
| Misconfiguration | CSPM، ارزیابی منظم، IaC Scanning |
| S3 باز | Block Public Access، Encryption |
| SSRF Metadata | IMDSv2، محدودسازی خروجی |
| IAM PrivEsc | Least Privilege، MFA، بازبینی منظم |
| Credential Leak | Secret Manager، Pre-commit Hook |
| Container Escape | غیرروت، حذف Capability، Seccomp |
| K8s Misconfig | RBAC، Network Policy، Pod Security |

### خلاصه فصل

- آزمایشگاه: Docker، Minikube، LocalStack.
- Juice Shop برای تمرین.
- دفاع: CSPM، Least Privilege، IMDSv2، RBAC.

---

## منابع و مراجع

- **AWS Security:** https://aws.amazon.com/security/
- **Azure Security:** https://azure.microsoft.com/en-us/security/
- **GCP Security:** https://cloud.google.com/security
- **OWASP Cloud Security:** https://owasp.org/www-project-cloud-security/
- **Pacu:** https://github.com/RhinoSecurityLabs/pacu
- **ScoutSuite:** https://github.com/nccgroup/ScoutSuite
- **cloud_enum:** https://github.com/initstring/cloud_enum
- **Trivy:** https://github.com/aquasecurity/trivy
- **kube-hunter:** https://github.com/aquasecurity/kube-hunter
- **LocalStack:** https://localstack.cloud/
