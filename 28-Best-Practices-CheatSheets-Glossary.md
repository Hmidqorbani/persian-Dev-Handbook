# فایل ۲۸ — بهترین روش‌ها، چیت‌شیت‌ها و واژه‌نامه (جمع‌بندی)

**مدت یادگیری**: ۸ ساعت (مرور و تثبیت)
**سطح سختی**: عمومی (جمع‌بندی)
**پیش‌نیاز**: تمام فایل‌های ۰۱ تا ۲۷

---

## فهرست محتوا

- [فصل ۱۳۵: بهترین روش‌های DevOps (فرهنگی و فنی)](#فصل-۱۳۵-بهترین-روشهای-devops-فرهنگی-و-فنی)
- [فصل ۱۳۶: چیت‌شیت‌های ابزار (CLI Cheat Sheets)](#فصل-۱۳۶-چیتشیتهای-ابزار-cli-cheat-sheets)
- [فصل ۱۳۷: معماری و الگوهای طراحی](#فصل-۱۳۷-معماری-و-الگوهای-طراحی)
- [فصل ۱۳۸: نقشهٔ راه یادگیری و شغلی (Roadmap)](#فصل-۱۳۸-نقشهٔ-راه-یادگیری-و-شغلی-roadmap)
- [فصل ۱۳۹: واژه‌نامه، منابع نهایی و پایان کتاب](#فصل-۱۳۹-واژهنامه-منابع-نهایی-و-پایان-کتاب)

---

## فصل ۱۳۵: بهترین روش‌های DevOps (فرهنگی و فنی)

### مدمه

این فصل خلاصهٔ «چه باید کرد» در DevOps را می‌دهد — بازگشت به **CALMS** (فایل ۰۱) و فهرستی از بهترین روش‌ها و ضدالگوها (anti-patterns) که در ۲۷ فایل پراکنده بودند.

### مفاهیم اساسی

**بازخوانی CALMS:**
- **C**ulture: همکاری توسعه و عملیات.
- **A**utomation: خودکارسازی تکرارپذیر.
- **L**ean: کاهش ضایعات (WIP کم، بازخورد سریع).
- **M**easurement: اندازه‌گیری با متریک/DORA.
- **S**haring: اشتراک دانش و مسئولیت.

**۱۵ بهترین روشٔ فنی:**
1. **همه‌چیز کد (IaC/Config/Dashboard as Code).**
2. **Git به‌عنوان منبع حقیقت (source of truth).**
3. **Pull-based GitOps** برای استقرار.
4. **Pipeline کوتاه** (<۱۰ دقیقه) با شکست زودهنگام.
5. **تصاویر کانتینر کوچک** (multi-stage/distroless).
6. **سیستم مانیتورینگ مبتنی بر SLO** (نه آستانهٔ دلخواه).
7. **تریس/متریک/لاگ یکپارچه** (Observability trilogy).
8. **شیفت-به‌چپ امنیت** (SAST/DAST/SCA در CI).
9. **رمزها در Vault** (هرگز در کد).
10. **Progressive Delivery** (Canary/Blue-Green).
11. **خودکارسازی toil** (کاهش کار دستی).
12. **Postmortem بی‌سرزنش** پس از هر SEV1/2.
13. **Least Privilege** در IAM/RBAC.
14. **تست خرابی (Chaos/Game Day)** منظم.
15. **مستندسازی مداوم** (README، RUNBOOK).

**ضدالگوها (Anti-patterns) — فرار کنید:**
- ❌ «اینجا کار می‌کند، مشکل محیط شماست» (محیط ناهمگون).
- ❌ کلید دستی `kubectl apply` در تولید (نقض GitOps).
- ❌ هشدارهای بی‌ربط (alert fatigue).
- ❌ رمز سخت‌کد در کد/متغیر محیطی.
- ❌ انتشار بزرگ نادر (big-bang release).
- ❌ ابزار جدید بدون یادگیری تیم.

**شاخص‌های موفقیت (DORA Metrics — فایل ۰۱):**
- Deployment Frequency (بیشتر بهتر).
- Lead Time for Changes (کمتر بهتر).
- Change Failure Rate (کمتر بهتر).
- Time to Restore (MTTR کمتر بهتر).

### مثال‌های عملی

**الف) چک‌لیست بازبینی (Pre-prod checklist):**
```
[ ] تست‌ها در CI سبز است؟
[ ] تصویر اسکن‌شده و امضا شده؟
[ ] SLO هنوز بودجه دارد؟
[ ] Runbook به‌روز است؟
[ ] Rollback آماده است (GitOps revert)؟
```

**ب) تعریف موفقیت با DORA:**
```
تیم الف: ۱۰ استقرار/روز، MTTR ۱ ساعت   → Elite
تیم ب: ۱ استقرار/ماه، MTTR ۱ هفته     → Low
```

### مقایسه و دیدگاه‌های مختلف

**بهترین روش در برابر «بستگی دارد»:**
- بسیاری قواعد «همیشه درست» نیستند — بستگی به اندازه/بلوغ تیم دارند.
- مثلاً GitOps برای تیم ۲ نفره ممکن است overkill باشد.

**فرهنگ در برابر ابزار:**
- ابزار بدون فرهنگ = اتلاف.
- فرهنگ بدون ابزار = وعدهٔ محقق‌نشده.
- **هر دو لازم‌اند.**

**دیدگاه:** «DevOps سفر است نه مقصد — هر فصل این کتاب یک گام است».

### اشتباهات رایج

1. **خرید ابزار به امید حل مشکل فرهنگی:** ابزار جادو نمی‌کند.
2. **کپی کورکورانه از شرکت بزرگ:** آنچه برای گوگل خوب است لزوماً برای شما نیست.
3. **توقف یادگیری پس از اولین pipeline:** DevOps دائماً تکامل می‌یابد.
4. **اندازه‌گیری نادرست:** تمرکز روی تعداد کامیت به‌جای DORA.

### نکات مهم و ترفندها

- **شروع کوچک، تکاملی:** یک بهبود در هر چرخه.
- **بازخورد سریع** را مقدس بدارید (هر چه سریع‌تر بشنوید، ارزان‌تر اصلاح کنید).
- **جشن گرفتن شکست‌های یادگیری‌شده** (فرهنگ بی‌سرزنش).
- **استفاده از Platform Engineering** برای کاهش بار تیم محصول.
- **بخوانید:** «Accelerate» و «The DevOps Handbook».

### خلاصه فصل

بهترین روش‌های DevOps دور **CALMS** و **۱۵ قاعدهٔ فنی** (IaC، GitOps، SLO، Shift-Left، Vault، Progressive Delivery، Postmortem) می‌چرخند و از **ضدالگوها** (محیط ناهمگون، کلید دستی، alert fatigue) پرهیز می‌کنند. **DORA Metrics** موفقیت را اندازه می‌گیرند. یادآوری: ابزار بدون فرهنگ بی‌فایده است. در فصل بعد چیت‌شیت‌های سریع ابزارها را داریم.

---

## فصل ۱۳۶: چیت‌شیت‌های ابزار (CLI Cheat Sheets)

### مدمه

مرجع سریع دستورات پرکاربرد برای ابزارهای اصلی که در کتاب دیدید. برای یادآوری سریع هنگام کار — نه جایگزین مستندات.

### مفاهیم اساسی

**چیت‌شیت‌های زیر کوتاه و عملیاتی‌اند.** (مسیرها/نام‌ها را با محیط خود تطبیق دهید.)

### مثال‌های عملی

**الف) Git (فصل ۱۱–۱۵):**
```bash
git clone <url>                  # کپی مخزن
git checkout -b feature/x        # شاخه جدید
git add -p                       # اضافه کردن تکه‌ای
git commit -m "feat: ..."        # conventional commit
git rebase main                 # بازپیوندی تمیز
git cherry-pick <hash>          # انتقال یک کامیت
git bisect start                # یافتن کامیت خراب
git reflog                      # تاریخچهٔ تغییرات HEAD
```

**ب) Docker (فصل ۳۶–۴۰):**
```bash
docker build -t app:v1 .         # ساخت تصویر
docker run -p 8080:80 app:v1     # اجرا با پورت
docker ps -a                     # لیست همهٔ کانتینرها
docker logs -f <id>              # لاگ زنده
docker exec -it <id> sh          # ورود به کانتینر
docker system prune -a           # پاکسازی (احتیاط)
docker compose up -d             # اجرای Compose
```

**ج) Kubernetes / kubectl (فصل ۵۱–۶۰):**
```bash
kubectl get pods -n app          # لیست پادها
kubectl describe pod <p>         # جزئیات/رویدادها
kubectl logs <p> -f --tail=50    # لاگ
kubectl exec -it <p> -- sh       # ورود
kubectl apply -f deploy.yaml     # اعمال
kubectl rollout status deploy/x  # وضعیت رول‌اوت
kubectl rollout undo deploy/x    # بازگشت
kubectl port-forward svc/x 8080  # تونل محلی
kubectl get events --sort-by=.lastTimestamp  # رویدادها
```

**د) Terraform (فصل ۷۱–۷۵):**
```bash
terraform init                  # مقداردهی
terraform plan -out=tfplan      # پیش‌نمایش
terraform apply tfplan          # اعمال
terraform fmt                   # فرمت‌دهی
terraform state list            # لیست منابع
terraform destroy               # تخریب (خطر!)
```

**ه) Helm (فصل ۶۱–۶۵):**
```bash
helm repo add stable <url>
helm install myrel stable/nginx -n app
helm upgrade myrel stable/nginx --set replicaCount=3
helm rollback myrel 1
helm template myrel ./chart     # خروجی YAML (بدون اعمال)
```

**و) ابر CLI:**
```bash
aws configure                    # تنظیم اعتبار
aws ec2 describe-instances
az login && az group list        # Azure
gcloud auth login && gcloud config set project X  # GCP
```

**ز) observability:**
```bash
kubectl port-forward -n monitoring svc/prometheus 9090  # Prometheus UI
kubectl port-forward -n monitoring svc/grafana 3000
promtool check rules alerts.yml                        # اعتبارسنجی
```

### مقایسه و دیدگاه‌های مختلف

**چیت‌شیت در برابر حفظ کردن:**
- چیت‌شیت: مرجع سریع، کاهش بار حافظه.
- حفظ کردن: سرعت در مواقع بحران (اما چیت‌شیت همیشه دقیق‌تر).

**k9s در برابر kubectl:**
- k9s: رابط تعاملی (TUI) سریع برای مدیریت خوشه.
- kubectl: خط فرمان برای اسکریپت/اتوماسیون.

**دیدگاه:** «چیت‌شیت خوب، دوست خوب — آن را در README پروژه یا Obsidian نگه دارید».

### اشتباهات رایج

1. **`terraform destroy` بدون تأیید:** فاجعهٔ تخریب کل زیربنا.
2. **`docker system prune -a` در تولید:** حذف تصاویر در حال اجرا.
3. **`kubectl delete` روی اشتباهی:** همیشه `-n` و نام را چک کنید.
4. **استفاده از `--force` بدون درک:** رد کردن هشدارهای ایمنی.

### نکات مهم و ترفندها

- **k9s / lazygit / tfswitch** ابزارهای TUI برای سرعت.
- **aliases:** `alias k=kubectl`، `alias tf=terraform`.
- **فایل‌های `.envrc`** برای تغییر خودکار محیط (direnv).
- **چیت‌شیت را در مخزن Portfolio قرار دهید** (فصل ۱۳۴).

### خلاصه فصل

این چیت‌شیت مرجع سریع Git/Docker/K8s/Terraform/Helm/Cloud/Observability است — برای یادآوری هنگام کار. نکتهٔ ایمنی: دستورات مخرب (`destroy`، `prune -a`، `delete`) را با تأیید دوباره اجرا کنید. در فصل بعد الگوهای طراحی را مرور می‌کنیم.

---

## فصل ۱۳۷: معماری و الگوهای طراحی

### مدمه

DevOps فراتر از ابزار، به نحوهٔ طراحی سیستم ربط دارد. این فصل الگوهای بالغ — **۱۲-Factor App**، میکروسرویس، Progressive Delivery، IaC patterns و **Platform Engineering** — را جمع‌بندی می‌کند.

### مفاهیم اساسی

**۱۲-Factor App (عوامل ۱۲گانه):**
1. Codebase (یک repo در کنترل نسخه).
2. Dependencies (صریح و ایزوله).
3. Config (در محیط، نه کد).
4. Backing Services (مثل منابع خارجی).
5. Build/Release/Run (جدا).
6. Processes (stateless).
7. Port Binding (خودمیزبان).
8. Concurrency (مقیاس با فرآیند).
9. Disposability (راه‌اندازی/توقف سریع).
10. Dev/Prod Parity (همگونی محیط‌ها).
11. Logs (جریان رویداد، نه فایل).
12. Admin Processes (مدیریت یکباره).

**الگوهای میکروسرویس:**
- **API Gateway** (ورودی یکتا).
- **Service Mesh** (ارتباط درونی — فصل ۱۲۰).
- **Database per Service** (هر سرویس DB خودش).
- **Saga** (تراکنش توزیع‌شده).
- **Strangler Fig** (مهاجرت تدریجی).

**Progressive Delivery (فصل ۴/۱۴):**
- Canary، Blue-Green، Shadow، A/B — کاهش شعاع انفجار.

**الگوهای IaC (فصل ۷۱–۸۰):**
- **Modules** برای reuse.
- **Remote State + Locking** (S3+DynamoDB / Terraform Cloud).
- **Environments as code** (dev/staging/prod یکسان).
- **Policy as Code** (Sentinel/OPA).

**Platform Engineering:**
- تیم پلتفرم ابزار self-service (Internal Developer Platform) می‌سازد تا تیم محصول روی محصول تمرکز کند (مثل Backstage).

### مثال‌های عملی

**الف) نمونهٔ ۱۲-Factor (Config):**
```bash
# غلط:
DB_HOST=10.0.0.5 در کد پایتون
# درست:
export DB_HOST=$DB_HOST   # از محیط (Vault/Env) می‌آید
```

**ب) Module Terraform (reuse):**
```hcl
module "orders_db" {
  source   = "./modules/rds"
  name     = "orders"
  env      = var.env
}
module "payments_db" {
  source   = "./modules/rds"   # همان ماژول
  name     = "payments"
}
```

**ج) Internal Developer Platform (مفهومی):**
```
تیم محصول:  `idp create service orders --template python-api`
  → Repo + CI + Helm + ArgoCD + مانیتورینگ آماده در ۱ دقیقه
```

### مقایسه و دیدگاه‌های مختلف

**مونولیت در برابر میکروسرویس:**
- مونولیت: ساده‌تر در ابتدا، مناسب تیم کوچک/MVP.
- میکروسرویس: مقیاس‌پذیر، اما پیچیدگی عملیاتی بالا.
- **رویکرد:** با مونولیت ماژولار شروع کنید، در صورت نیاز جدا کنید (Strangler).

**Platform Engineering در برابر DevOps:**
- DevOps: مسئولیت اشتراکی.
- Platform Eng: تیمی که ابزار را برای دیگران می‌سازد (تکامل DevOps در مقیاس).

**دیدگاه:** «الگوها راهنمایند نه قانون — استفادهٔ بیش‌ازحد از میکروسرویس (microservice envy) اشتباه رایج است».

### اشتباهات رایج

1. **میکروسرویس زودهنگام:** پیچیدگی بی‌دلیل برای استارتاپ کوچک.
2. **Shared Database در میکروسرویس:** کوپلینگ محکم (در تضاد با هدف).
3. **Monolith در تابع Serverless:** نقض روح Serverless (فصل ۱۲۷).
4. **IaC بدون remote state locking:** دو نفر همزمان → خرابی state.
5. **عدم همگونی Dev/Prod:** «اینجا کار می‌کند».

### نکات مهم و ترفندها

- **شروع با مونولیت ماژولار** سپس جدا کردن (Strangler).
- **هر سرویس مالک DB خودش** باشد.
- **IaC modules** را در registry داخلی به اشتراک بگذارید.
- **Backstage** (Spotify) برای Internal Developer Portal بررسی کنید.
- **Dev/Prod parity** با کانتینر و IaC یکسان تضمین شود.

### خلاصه فصل

طراحی خوب شامل **۱۲-Factor** (stateless، config در محیط، logs به‌عنوان جریان)، **میکروسرویس با مرزهای درست** (DB per service)، **Progressive Delivery**، و **IaC modular** است. **Platform Engineering** ابزار self-service می‌سازد. اشتباهات: میکروسرویس زودهنگام و shared DB. توصیه: با مونولیت ماژولار شروع کنید. در فصل بعد نقشهٔ راه یادگیری و شغلی را می‌بینیم.

---

## فصل ۱۳۸: نقشهٔ راه یادگیری و شغلی (Roadmap)

### مدمه

این کتاب پایان نیست، آغاز است. این فصل مسیر ادامهٔ یادگیری، گواهینامه‌ها و جوامع را برای تبدیل شدن به مهندس DevOps/SRE ارشد ترسیم می‌کند.

### مفاهیم اساسی

**مسیر یادگیری پیشنهادی (بر اساس این کتاب):**
```
۱. Linux/Shell/Networking  → (فایل ۰۲)
۲. Git/Branching          → (فایل ۱۱)
۳. CI/CD (یکی از Jenkins/GHA/GitLab) → (فایل ۰۴–۰۷)
۴. Docker/Podman          → (فایل ۰۸–۱۰)
۵. Kubernetes             → (فایل ۱۱–۱۳)
۶. GitOps                → (فایل ۱۴)
۷. IaC (Terraform)       → (فایل ۱۵)
۸. Cloud (AWS اول)        → (فایل ۱۸–۱۹)
۹. Observability         → (فایل ۲۰–۲۲)
۱۰. SRE/Incident         → (فایل ۲۳)
۱۱. DevSecOps            → (فایل ۲۴)
۱۲. Mesh/Serverless      → (فایل ۲۵–۲۶)
۱۳. Capstone             → (فایل ۲۷)
```

**گواهینامه‌های مرتبط:**
- **Kubernetes:** CKA (ادمین)، CKAD (توسعه‌دهنده)، CKS (امنیت).
- **Cloud:** AWS Solutions Architect Associate/Professional، Azure AZ-104، GCP PCA.
- **Terraform:** HashiCorp Terraform Associate.
- **SRE:** Google SRE (دوره‌های رایگان).
- **Linux:** LPIC-1/2 (پایه).

**جوامع و منابع:**
- **CNCF** (بنیاد ابر بومی) — رویدادها، پروژه‌ها.
- **Kubernetes Slack**، **DevOps Iran** (جوامع فارسی).
- **GitHub** (مشارکت متن‌باز)، **Stack Overflow**.
- **KubeCon / DevOpsDays** (کنفرانس).

**مهارت‌های نرم (Soft Skills):**
- ارتباط (برای Postmortem/On-call).
- یادگیری مداوم (این حوزه سریع تغییر می‌کند).
- تفکر سیستمی.

### مثال‌های عملی

**الف) برنامهٔ ۶ ماهه (نمونه):**
```
ماه ۱–۲: Linux + Git + یک CI/CD
ماه ۳–۴: Docker + K8s + پروژه شخصی
ماه ۵:   Terraform + AWS + GitOps
ماه ۶:   Observability + SRE + Capstone + CKA
```

**ب) نمونهٔ رزومه (بولت):**
```
- طراحی خط لولهٔ GitOps با ArgoCD (زمان استقرار ۳۰ دقیقه → ۳ دقیقه)
- کاهش MTTR با SLO/Burn-rate alerting (۴ ساعت → ۳۰ دقیقه)
- مهاجرت ۲۰ سرویس به K8s با صفر خرابی (Canary)
```

**ج) سهم در متن‌باز:**
```bash
# شروع با مستندسازی/مثال در پروژه‌های CNCF (مثل kubernetes/examples)
```

### مقایسه و دیدگاه‌های مختلف

**گواهینامه در برابد تجربه:**
- گواهینامه: دروازهٔ HR و ساختار یادگیری.
- تجربه (Portfolio): اثبات واقعی — مهم‌تر.
- **هر دو:** گواهینامه برای مصاحبه، Portfolio برای قبولی.

**تخصص در برابر T-shaped:**
- عمیق در یک حوزه (مثل K8s) + وسیع در بقیه (T-shaped) ایده‌آل است.
- Generalist صرف: در مدیریت خوب؛ Specialist صرف: در مشکلات محدود.

**دیدگاه:** «بهترین مهندس DevOps کسی است که هم کد می‌نویسد هم با آدم‌ها حرف می‌زند».

### اشتباهات رایج

1. **جمع‌آوری گواهینامه بدون تجربه:** رزومه پر اما مهارت خالی.
2. **پرش از لینوکس به K8s:** پایه ضعیف → درک سطحی.
3. **یادگیری ابزار بدون حل مسئله:** دانش تئوری بی‌استفاده.
4. **نادیده گرفتن مهارت نرم:** SRE بدون ارتباط در حادثه شکست می‌خورد.

### نکات مهم و ترفندها

- **یک ابر را عمیق یاد بگیرید** (معمولاً AWS) سپس دیگران را سریع.
- **بسازید و منتشر کنید** — Portfolio از ۱۰ گواهینامه ارزشمندتر است.
- **مطالعهٔ SRE Book گوگل** (رایگان) را جدی بگیرید.
- **پیگیری CNCF Landscape** برای دیدن ابزارهای جدید.
- **تدریس/وبلاگ‌نویسی:** بهترین راه تثبیت یادگیری.

### خلاصه فصل

نقشهٔ راه از پایه (Linux/Git) تا تخصص (K8s/IaC/Cloud/SRE) می‌رود. گواهینامه‌ها (CKA/AWS/Terraform) دروازه‌اند، اما **Portfolio و تجربه** تعیین‌کننده‌اند. جوامع (CNCF/KubeCon) و مهارت نرم حیاتی‌اند. اشتباه: پرش از پایه و جمع‌آوری گواهینامه بدون تجربه. در فصل آخر واژه‌نامه و منابع نهایی را داریم.

---

## فصل ۱۳۹: واژه‌نامه، منابع نهایی و پایان کتاب

### مدمه

فصل پایانی: واژه‌نامهٔ فارسی↔انگلیسی اصطلاحات کلیدی، فهرست منابع برتر، و سخن پایانی این سفر ۲۸ فایلی.

### مفاهیم اساسی

**واژه‌نامهٔ (Glossary) کلیدی:**

| انگلیسی | فارسی |
|---|---|
| DevOps | دوآپس (توسعه + عملیات) |
| CALMS | فرهنگ/خودکارسازی/تراز/اندازه‌گیری/اشتراک |
| CI/CD | ادغام/تحویل پیوسته |
| Pipeline | خط لوله |
| Container | کانتینر |
| Image | تصویر (کانتینر) |
| Registry | ثبت‌گاه (ذخیرهٔ تصاویر) |
| Orchestration | ارکستراسیون (مثل K8s) |
| Kubernetes (K8s) | کوبرنتیز |
| Pod | پاد (واحد اجرا در K8s) |
| Deployment | استقرار |
| Service | سرویس (شبکه داخلی) |
| Ingress | دروازهٔ ورودی HTTP |
| Helm | هلم (مدیریت چارت K8s) |
| GitOps | گیت‌آپس (استقرار مبتنی بر git) |
| IaC (Infrastructure as Code) | زیربنا به‌عنوان کد |
| Terraform | ترافورم |
| Ansible | انسیبل |
| Cloud | ابر |
| Region / AZ | منطقه / منطقهٔ دسترس‌پذیری |
| VPC / VNet | شبکهٔ خصوصی مجازی |
| IAM / RBAC | مدیریت دسترسی / کنترل دسترسی نقش‌محور |
| Observability | مشاهده‌پذیری |
| Metrics / Logs / Traces | متریک / لاگ / ردیابی |
| Prometheus / Grafana | پرومتئوس / گرافانا |
| Loki / ELK | لوکی / ای‌ال‌کی |
| OpenTelemetry (OTel) | تله‌متری باز |
| SRE (Site Reliability Engineering) | مهندسی قابلیت اطمینان |
| SLO / SLI / SLA | هدف/شاخص/توافق‌نامهٔ سطح سرویس |
| Error Budget | بودجهٔ خطا |
| Incident / Postmortem | حادثه / گزارش پس از حادثه |
| DevSecOps | دوآپس‌امنیت (امنیت در چرخه) |
| SAST / DAST / SCA | تحلیل ایستا/پویا/ترکیب نرم‌افزار |
| SBOM | فهرست مواد سازندهٔ نرم‌افزار |
| mTLS | TLS متقابل |
| Service Mesh | سرویس مش |
| Serverless / FaaS | بدون‌سرور / تابع به‌عنوان سرویس |
| Canary / Blue-Green | کاناری / آبی-سبز (روش استقرار) |
| Progressive Delivery | تحویل تدریجی |
| DORA Metrics | شاخص‌های دورا |
| Drift | انحراف (تفاوت وضعیت واقعی با کد) |
| Idempotent | توان‌اتم (نتیجهٔ یکسال با اجرای چندباره) |

**منابع برتر (نهایی):**
- **کتاب‌ها:** «The DevOps Handbook»، «Accelerate» (Forsgren et al.)، «Site Reliability Workbook» (گوگل، رایگان)، «Kubernetes: Up and Running»، «Prometheus: Up & Running».
- **مستندات رسمی:** kubernetes.io، prometheus.io، terraform.io، docs.aws، cloud.google.com، learn.microsoft.com.
- **پروژه‌های CNCF:** cncf.io/projects (۲۰۰+ پروژه).
- **تمرین رایگان:** Killercoda، Katacoda (آموزش مرحله‌ای)، Play with Docker/Kubernetes.
- **وبلاگ‌ها:** Google SRE، AWS Architecture، CNCF Blog.

**سخن پایانی:**
این کتاب از مفهوم DevOps (فایل ۰۱) تا پیاده‌سازی کامل یک پلتفرم تولیدی (فایل ۲۷) را پیمود. شما حالا با: فرهنگ و روش، ابزارهای CI/CD، کانتینر و ارکستراسیون، IaC، ابر، مشاهده‌پذیری، SRE، امنیت، سرویس مش و سرورلس آشنایید. اما دانش با **انجام دادن** تثبیت می‌شود — پروژه‌های فایل ۲۷ را بسازید، در جوامع مشارکت کنید، و هر روز یک قدم جلوتر بروید.

> «DevOps سفر است، نه مقصد.» — با معماری مرجع فایل ۱۳۰ شروع کنید و تکاملی بسازید.

### مثال‌های عملی

**الف) خلاصهٔ دستور شروع:**
```bash
# محیط آزمایشی رایگان
kind create cluster --name learn
helm install prometheus prometheus-community/kube-prometheus-stack -n mon --create-namespace
# حالا اپلیکیشن خود را مستقر و مانیتور کنید
```

**ب) فهرست بازبینی نهایی (آیا آماده‌اید؟):**
```
[ ] یک pipeline کامل ساخته‌اید؟
[ ] یک اپلیکیشن را در K8s با GitOps مستقر کرده‌اید؟
[ ] SLO تعریف و هشدار داده‌اید؟
[ ] یک حادثه را در Game Day تمرین کرده‌اید؟
[ ] امنیت (اسکن/رمز/مTLS) را اعمال کرده‌اید؟
[ ] یک Portfolio در GitHub دارید؟
```

### مقایسه و دیدگاه‌های مختلف

**یادگیری کتابی در برابر عملی:**
- کتاب/این مجموعه: چارچوب و پوشش.
- عملی (پروژه/شغل): عمق واقعی.
- **هر دو ضروری.**

**وسعت در برابر عمق:**
- ابزارهای زیاد سطحی vs یک ابزار عمیق → T-shaped بهترین.

**دیدگاه:** «این کتاب نقشه است، نه سفر — پاهایتان (عمل) مسیر را طی می‌کند».

### اشتباهات رایج

1. **خواندن بدون عمل:** دانش تئوری بدون تمرین زود فراموش می‌شود.
2. **کامل‌گرایی:** منتظر «یادگیری همه چیز» قبل از شروع پروژه.
3. **منزوی ماندن:** نیاموختن از جامعه.

### نکات مهم و ترفندها

- **بخش‌بندی یادگیری:** هر هفته یک فایل + یک آزمایش.
- **یادداشت‌برداری فعال:** هنگام خواندن، دستورات را اجرا کنید.
- **بازگشت دوره‌ای:** هر ۶ ماه مفاهیم را بازبینی کنید (چون ابزارها تغییر می‌کنند).
- **اشتراک‌گذاری:** چیزی که آموختید را آموزش دهید (بهترین یادگیری).

### خلاصه فصل

فصل پایانی واژه‌نامهٔ جامع (فارسی↔انگلیسی)، منابع برتر (کتاب‌ها/مستندات/CNCF) و سخن پایانی را ارائه داد. شما اکنون نقشهٔ کامل DevOps را دارید — از فرهنگ تا ابزار تا معماری تولید. کلید نهایی: **یادگیری با عمل** (پروژه‌های فایل ۲۷) و **اشتراک دانش**. این پایان کتاب ۲۸ فایلی است؛ آغاز سفر حرفه‌ای شماست.

---

## منابع و مراجع نهایی

- **کتاب‌های کلیدی:**
  - «The DevOps Handbook» — Gene Kim et al.
  - «Accelerate» — Nicole Forsgren, Jez Humble, Gene Kim
  - «Site Reliability Engineering» & «The SRE Workbook» — Google (رایگان در sre.google/books)
  - «Kubernetes: Up and Running» — Hightower, Burns, Beda
  - «Prometheus: Up & Running» — O'Reilly
- **مستندات رسمی:** kubernetes.io · prometheus.io · grafana.com · terraform.io · ansible.com · docs.aws · cloud.google.com · learn.microsoft.com
- **پروژه‌های CNCF:** https://www.cncf.io/projects/ (شامل Kubernetes، Prometheus، Envoy، Istio، Argo، Flux، OpenTelemetry و صدها پروژه)
- **تمرین تعاملی:** Killercoda (killercoda.com) · Play with Docker (labs.play-with-docker.com) · Play with Kubernetes
- **جوامع:** CNCF Slack · Kubernetes Slack · رویدادهای KubeCon + CloudNativeCon · DevOpsDays
- **گواهینامه:** CKA/CKAD/CKS (cncf.io) · AWS SA / Azure AZ-104 / GCP PCA · HashiCorp Terraform Associate

**پایان کتاب — DevOps از صفر تا تولید 🎉**

با معماری مرجع (فایل ۱۳۰) شروع کنید، پروژه‌های Capstone (فایل ۲۷) را بسازید، و سفر خود را در GitHub به اشتراک بگذارید. موفق باشید!
