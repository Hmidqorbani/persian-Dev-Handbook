# فایل ۲۴ — DevSecOps و امنیت در چرخه (DevSecOps)

**مدت یادگیری**: ۱۲ ساعت
**سطح سختی**: سخت
**پیش‌نیاز**: فایل ۰۴ (CI/CD)، فایل ۰۸/۰۹ (کانتینر/ثبت‌گاه)، فایل ۱۲ (امنیت K8s)، فایل ۱۵ (IaC)

---

## فهرست محتوا

- [فصل ۱۱۶: مبانی DevSecOps و امنیت در چرخه](#فصل-۱۱۶-مبانی-devsecops-و-امنیت-در-چرخه)
- [فصل ۱۱۷: امنیت در CI/CD (SAST، DAST، SCA، Secret Scanning)](#فصل-۱۱۷-امنیت-در-cicd-sast-dast-sca-secret-scanning)
- [فصل ۱۱۸: امنیت کانتینر و زنجیرهٔ تأمین (Supply Chain)](#فصل-۱۱۸-امنیت-کانتینر-و-زنجیرهٔ-تأمین-supply-chain)
- [فصل ۱۱۹: مدیریت رمز و دسترسی امن + جمع‌بندی DevSecOps](#فصل-۱۱۹-مدیریت-رمز-و-دسترسی-امن--جمعبندی-devsecops)

---

## فصل ۱۱۶: مبانی DevSecOps و امنیت در چرخه

### مقدمه

«DevOps» سرعت را آورد؛ «DevSecOps» امنیت را در همان سرعت ادغام می‌کند. به‌جای تست امنیت در انتهای چرخه (که باعث تأخیر و کشف دیرهنگام می‌شود)، DevSecOps امنیت را **به چپ می‌راند (Shift Left)** — یعنی از همان خط اول کد. این فصل فلسفه و چارچوب را می‌سازد.

### مفاهیم اساسی

**چرا DevSecOps؟**
- کشف آسیب‌پذیری در تولید = هزینهٔ ۱۰۰ برابر نسبت به زمان توسعه.
- حملات زنجیرهٔ تأمین (SolarWinds، Log4Shell، xz backdoor) نشان دادند نرم‌افزار از کجا می‌آید مهم است.
- DevOps بدون امنیت = «سریع اما ناامن».

**اصول کلیدی:**
- **Shift Left:** امنیت در مرحلهٔ نوشتن کد و CI، نه تست نهایی.
- **Security as Code:** سیاست‌های امنیتی به‌صورت کد (قابل نسخه‌کنترل، قابل آزمون).
- **Automation:** اسکن خودکار در هر commit/PR.
- **Shared Responsibility:** توسعه‌دهنده هم مسئول امنیت کدش است (نه فقط تیم امنیت).
- **Least Privilege:** کمترین دسترسی لازم (تکرار از فصل ۹۰/۱۱۲).
- **Defense in Depth:** لایه‌های متعدد دفاع.

**مدل تهدید (Threat Modeling):**
- شناسایی دارایی‌ها، تهدیدها، و نقاط ضعف پیش از کدنویسی.
- چارچوب **STRIDE** (Spoofing, Tampering, Repudiation, Information disclosure, Denial, Elevation).

**زنجیرهٔ تأمین نرم‌افزار (Software Supply Chain):**
هر چیزی که روی کد اثر می‌گذارد: کتابخانه‌های third-party، base image، CI/CD pipeline، registry، و dependencies. بیشتر حملات مدرن از اینجا وارد می‌شوند.

**مقررات و استانداردها:** SOC 2، ISO 27001، GDPR، PCI-DSS، و چارچوب **SLSA** (Supply-chain Levels for Software Artifacts) از گوگل.

### مثال‌های عملی

**الف) نمودار Shift Left:**
```
سنتی:  Code → Build → Test → Deploy → 🔴 Security Test (دیر!)
DevSecOps: 🔒Security → Code(🔒) → Build(🔒) → Test(🔒) → Deploy(🔒)
          (اسکن مخفی در هر مرحله)
```

**ب) STRIDE روی یک صفحهٔ ورود:**
```
Spoofing: جعل هویت کاربر
Tampering: دستکاری توکن در کلاینت
Repudiation: انکار انجام تراکنش
Info Disclosure: نشت خطا در پاسخ API
Denial: حملهٔ brute force
Elevation: IDOR (دسترسی به سفارش دیگران)
```

**ج) سیاست به‌عنوان کد (مفهومی):**
```yaml
# policy-as-code: هیچ تصویری با USER root اجازهٔ استقرار ندارد
constraint: no-root-container
enforced_in: [staging, production]
```

### مقایسه و دیدگاه‌های مختلف

**DevSecOps در برابر Security قدیم:**
- قدیم: تیم امنیت جداگانه، تست در انتها، «مانع سرعت».
- DevSecOps: امنیت در جریان توسط توسعه‌دهنده، خودکار، «تسهیل‌کننده».

**Shift Left در برابر Shift Right:**
- Shift Left: پیشگیری (ارزان‌تر).
- Shift Right: تست در تولید (مکمل، مثلاً runtime protection).

**دیدگاه‌ها:**
- **طرفداران:** کاهش هزینهٔ آسیب‌پذیری، انطباق (compliance) خودکار.
- **منتقدان:** افزایش باری روی توسعه‌دهنده؛ ابزارهای اسکن گاهی نویز (false positive) دارند.
- **رویکرد عملی:** خودکارسازی با «دروازهٔ نرم» (soft gate) — هشدار اول، مسدودسازی بعد از maduruh.

### اشتباهات رایج

1. **امنیت فقط در انتهای چرخه:** کشف دیرهنگام → بازنویسی گران.
2. **فکر کردن «امنیت کار تیم دیگر است»:** مسئولیت اشتراکی است.
3. **اعتماد کورکورانه به dependencies:** بسیاری حملات از کتابخانهٔ فرعی وارد می‌شوند.
4. **غفلت از Pipeline خودش:** CI/CD معمولاً ضعیف‌ترین حلقه است (دسترسی بالا به همه چیز).
5. **بدون Threat Modeling:** کد را بدون درک دشمن می‌نویسید.

### نکات مهم و ترفندها

- **شروع با یک ابزار:** مثلاً SCA (اسکن وابستگی) — بیشترین بازدهی سریع.
- **SBOM (فایل ۰۹):** فهرست مواد سازندهٔ نرم‌افزار را تولید کنید (الزام قانونی در برخی کشورها).
- **Dependency Pinning + Review:** نسخه را قفل کنید اما به‌روزرسانی را منظم کنید.
- **Secrets هرگز در repo:** از ابتدای پروژه Vault/Secret Manager (فصل ۱۱۹).
- **آموزش توسعه‌دهنده:** OWASP Top 10 را بیاموزید.
- **اجبار Signing (فصل ۱۱۸):** artefacts امضا شوند تا تغییر غیرمجاز مشخص شود.

### خلاصه فصل

**DevSecOps** امنیت را به چپ (Shift Left) می‌راند تا ارزان‌تر و زودتر اعمال شود. اصول: امنیت به‌عنوان کد، خودکارسازی، مسئولیت اشتراکی، و توجه به **زنجیرهٔ تأمین**. **Threat Modeling (STRIDE)** پیش از کد کمک می‌کند. اشتباه کلیدی: امنیت صرفاً در انتهای چرخه و غفلت از خودِ pipeline. در فصل بعد اسکن‌های عملی CI/CD را بررسی می‌کنیم.

---

## فصل ۱۱۷: امنیت در CI/CD (SAST، DAST، SCA، Secret Scanning)

### مقدمه

حالا ابزارها را می‌بینیم. چهار ستون اسکن در pipeline: **SAST** (کد ایستا)، **DAST** (زمان اجرا)، **SCA** (وابستگی)، و **Secret Scanning**. علاوه بر این، اسکن **IaC** (زیربنا به‌عنوان کد) و دروازه‌های (gates) کیفیت. این فصل هر کدام را با مثال عملی در pipeline بررسی می‌کند.

### مفاهیم اساسی

**انواع اسکن:**
1. **SAST (Static Application Security Testing):** تحلیل کد منبع بدون اجرا — یافتن الگوهای ناامن (SQL injection، XSS). ابزار: **Semgrep**، **SonarQube**، CodeQL.
2. **DAST (Dynamic / فرانت‌اند):** حمله به برنامهٔ در حال اجرا — یافتن آسیب‌پذیری در زمان اجرا. ابزار: **OWASP ZAP**، Burp Suite.
3. **SCA (Software Composition Analysis):** بررسی کتابخانه‌های third-party برای CVE شناخته‌شده. ابزار: **Trivy**، **Dependabot**، Snyk.
4. **Secret Scanning:** جست‌وجوی کلید/رمز سخت‌کد شده در repo. ابزار: **gitleaks**، **TruffleHog**.
5. **IaC Scanning:** بررسی Terraform/K8s manifest برای پیکربندی ناامن (مثلاً container با privilege). ابزار: **Checkov**، **tfsec**، **kube-linter**.

**دروازهٔ کیفیت (Quality Gate):**
پipeline در صورت یافتن آسیب‌پذیری حیاتی (critical) متوقف می‌شود (fail the build).

**مفهوم False Positive / Negative:**
- Positive کاذب: هشدار اشتباه → نویز.
- Negative کاذب: عدم شناسایی واقعی → خطرناک‌تر.

### مثال‌های عملی

**الف) GitHub Actions با چند اسکن (نمای کلی):**
```yaml
name: sec-scan
on: [pull_request]
jobs:
  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: returntocorp/semgrep-action@v1
        with: { config: "p/security-audit" }
  sca:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aquasecurity/trivy-action@master
        with: { scan-type: "fs", severity: "CRITICAL,HIGH" }
  secrets:
    runs-on: ubuntu-latest
    steps:
      - uses: gitleaks/gitleaks-action@v2
```

**ب) اسکن IaC با Checkov:**
```bash
checkov -d ./terraform/ --framework terraform
# خروجی: "FAILED: CKV_AWS_.... Ensure S3 bucket has versioning" → بلوک می‌شود
```

**ج) مثال SAST (Semgrep rule برای SQL injection):**
```yaml
rules:
  - id: sql-injection
    pattern: "SELECT * FROM $T WHERE id = " + $INPUT
    message: "احتمال SQL injection — از پارامترهای آماده استفاده کنید"
    severity: ERROR
```

**د) DAST با OWASP ZAP (در محیط تست):**
```bash
# اجرای اسپایدر روی URL تست
zap-cli quick-scan --self-contained --start-options "-config api.key=xxx" http://staging.app.local
```

**ه) GitLab CI (اسکن داخلی):**
```yaml
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
  - template: Security/Secret-Detection.gitlab-ci.yml
```

### مقایسه و دیدگاه‌های مختلف

**SAST در برابر DAST:**
- SAST: زودتر (در CI)، کم‌هزینه، اما فقط کد را می‌بیند.
- DAST: واقعی‌تر (برنامهٔ اجراشده)، اما نیاز به محیط اجرا.
- **هر دو لازم‌اند** — SAST برای پیشگیری، DAST برای تأیید.

**Trivy در برابر Snyk:**
- Trivy: متن‌باز، سریع، هم کانتینر هم SCA.
- Snyk: تجاری، دیتابیس بهتر، fix پیشنهادی.

**Block در برابر Warn:**
- Block (fail build): امن‌تر اما کُند برای تیم.
- Warn: سریع‌تر، اما آسیب‌پذیری انباشته می‌شود.
- **ترکیبی:** Block برای Critical/High جدید، Warn برای Low.

**دیدگاه:** «ابزارها شروع خوبی‌اند، اما فرهنگ مهم‌تر است — اگر توسعه‌دهنده هشدار را نادیده بگیرد، هیچ اسکنی کمک نمی‌کند».

### اشتباهات رایج

1. **فقط SAST، بدون SCA:** ۸۰٪ آسیب‌پذیری‌ها از dependencies می‌آیند نه کد خودتان.
2. **اسکن فقط روی main، نه PR:** مشکل بعد از ادغام کشف می‌شود.
3. **نادیده گرفتن هشدارها:** انباشت بدهی امنیتی.
4. **Secret در repo حتی پس از حذف:** تاریخچه git هنوز دارد — باید چرخش (rotation) شود.
5. **اعتماد کور به ابزار:** false negative باعث احساس امنیت کاذب می‌شود.
6. **DAST بدون محیط ایزوله:** حمله به تولید → فاجعه.

### نکات مهم و ترفندها

- **همهٔ اسکن‌ها در PR** اجرا شوند تا پیش از ادغام متوقف شود.
- **پین کردن نسخهٔ ابزار اسکن** (مثلاً `semgrep-action@v1`) برای تکرارپذیری.
- **Base Branch Diff Scan:** فقط مشکلات جدید را بلوک کنید تا تیم غرق نشود.
- **SBOM را با Trivy/Syft تولید کنید** (`syft dir:. -o spdx-json`).
- **Dependabot/Renovate** را برای به‌روزرسانی خودکار وابستگی فعال کنید.
- **Pre-commit hook:** gitleaks را روی کامیت محلی اجرا کنید تا قبل از push گیر بیفتد.
- **نتایج را متمرکز کنید:** ابزارهایی مثل DefectDojo برای مدیریت یافته‌ها.

### خلاصه فصل

امنیت CI/CD بر چهار رکن استوار است: **SAST** (کد ایستا)، **DAST** (زمان اجرا)، **SCA** (وابستگی/CVE) و **Secret Scanning**، به‌علاوهٔ **IaC Scanning** (Checkov/Trivy). اجرای آن‌ها در **PR** با **دروازهٔ کیفیت** (fail برای Critical) کلید جلوگیری از انتشار ناامن است. اشتباه بحرانی: فقط SAST (در حالی که ۸۰٪ حملات از dependency) و نادیده گرفتن هشدارها. ابزار + فرهنگ باهم لازم‌اند. در فصل بعد امنیت کانتینر و زنجیرهٔ تأمین را عمیق‌تر می‌بینیم.

---

## فصل ۱۱۸: امنیت کانتینر و زنجیرهٔ تأمین (Supply Chain)

### مقدمه

کانتینرها و تصاویرشان بخش بزرگی از زنجیرهٔ تأمین مدرن‌اند. یک base image آلوده یا تصویر امضا‌نشده می‌تواند هزاران سرور را به خطر بیندازد (مثل حوادث اخیر). این فصل اسکن تصویر، امضا (signing)، SBOM، تصویرهای حداقلی، و استانداردهایی مثل **SLSA/Sigstore** را بررسی می‌کند. (بخشی از این مفاهیم در فصل ۴۱–۴۵ هم دیده شد — اینجا تمرکز امنیتی است.)

### مفاهیم اساسی

**اسکن تصویر (Image Scanning):**
- بررسی لایه‌های image برای بسته‌های آسیب‌پذیر (CVE). ابزار: **Trivy**، **Grype**، Clair.
- باید در CI (قبل از push) و در registry (پیوسته) انجام شود.

**امضای تصویر (Image Signing):**
- اطمینان از اینکه تصویر تغییر نیافته و از منبع معتبر است.
- **cosign** (پروژهٔ Sigstore): امضای image با کلید، تأیید (verify) قبل از اجرا.
- **Notary / Docker Content Trust:** سیستم قدیمی‌تر.

**SBOM (Software Bill of Materials):**
- فهرست همهٔ بسته‌ها/لایه‌ها در image. ابزار: **Syft**، **SPDX**، **CycloneDX**.
- برای پاسخ سریع به آسیب‌پذیری تازه‌کشف‌شده (مثل Log4Shell) حیاتی است.

**تصویرهای حداقلی (Minimal/Distroless):**
- استفاده از **distroless** یا **scratch** به‌جای Ubuntu کامل → سطح حمله (attack surface) کمتر.
- چندمرحله‌ای (multi-stage build، فصل ۳۸) حجم و ریسک را کاهش می‌دهد.

**زنجیرهٔ تأمین و SLSA:**
- **SLSA:** چارچوبی برای اثبات اصالت artefact (سطوح ۱–۴).
- **Sigstore:** زیرساخت امضای کلیدعمومی (keyless) — cosign از آن استفاده می‌کند.
- **Provenance (منشأ):** متادیتا که می‌گوید artefact کجا و چگونه ساخته شد.

**امنیت Registry:**
- دسترسی محدود، اسکن خودکار، امضا اجباری، tagging ایمن (نه `latest` بی‌قید).

### مثال‌های عملی

**الف) اسکن image با Trivy در CI:**
```bash
trivy image --exit-code 1 --severity CRITICAL myapp:build
# اگر CVE حیاتی باشد → build شکست می‌خورد
```

**ب) تولید SBOM با Syft:**
```bash
syft myapp:build -o spdx-json > sbom.spdx.json
# هنگام حملهٔ جدید، جست‌وجو کنید کدام image آسیب‌پذیر است
```

**ج) امضای image با cosign:**
```bash
# امضا
cosign sign --key cosign.key myregistry/myapp@sha256:abc123
# تأیید قبل از اجرا (مثلاً در admission)
cosign verify --key cosign.pub myregistry/myapp@sha256:abc123
```

**د) Dockerfile حداقلی (distroless):**
```dockerfile
FROM golang:1.22 AS build
WORKDIR /src
COPY . .
RUN CGO_ENABLED=0 go build -o app .

FROM gcr.io/distroless/static-debian12
COPY --from=build /src/app /app
USER nonroot:nonroot     # کاربر غیرریشته
ENTRYPOINT ["/app"]
```

**ه) ممنوعیت اجرای image امضا‌نشده (سیاست K8s با Kyverno — فصل ۶۰):**
```yaml
# خط‌مشی: فقط imageهای امضا‌شده با کلید ما اجازهٔ اجرا دارند
validation:
  - pattern: "cosign verify ... {{ image }} ..."
```

### مقایسه و دیدگاه‌های مختلف

**امضا (Signing) در برابر فقط اسکن:**
- اسکن: «اکنون چه آسیب‌پذیری دارد؟»
- امضا: «آیا همان چیزی است که ساخته‌ام؟» (تمامیت/اصالت)
- **هر دو لازم‌اند** — اسکن وضعیت، امضا هویت.

**Distroless در برابر Ubuntu کامل:**
- Distroless: کوچک، ایمن، اما دیباگ سخت‌تر (بدون shell).
- Ubuntu: راحت برای دیباگ، اما سطح حمله بزرگ.

**Trivy در برابر Grype:**
- هر دو عالی و متن‌باز؛ Trivy چندکاره‌تر (هم image هم SCA هم IaC).

**دیدگاه:** «SBOM دیگر اختیاری نیست — در بسیاری حوزه‌ها قانونی است (مثل فرمان اجرایی آمریکا دربارهٔ امنیت زنجیرهٔ تأمین)».

### اشتباهات رایج

1. **Push image بدون اسکن:** آسیب‌پذیری شناخته‌شده به registry می‌رود.
2. **استفاده از `latest` tag در تولید:** نسخه نامشخص → عدم بازتولید و ریسک.
3. **image بر پایهٔ Ubuntu حجیم:** سطح حمله بزرگ، بوت کُند.
4. **عدم نگهداری SBOM:** هنگام حمله نمی‌دانید کجا آسیب‌پذیرید.
5. **امضای image اما عدم verify در اجرا:** امضا بی‌فایده است اگر K8s بررسی نکند.
6. **کلید cosign را در repo گذاشتن:** کلید خصوصی فاش → هر کسی می‌تواند امضا کند.

### نکات مهم و ترفندها

- **اسکن در دو نقطه:** CI (قبل از push) + Registry (پیوسته/زمان‌دار).
- **Sigstore keyless:** cosign با هویت OIDC امضا کند (نیاز به مدیریت کلید کمتر).
- **Distroless + multi-stage:** حجم را ۱۰ برابر کاهش دهید.
- **Admission Controller** (Kyverno/OPA، فصل ۶۰) تصاویر امضا‌نشده را رد کند.
- **SBOM را در artefact pipeline ذخیره کنید** (مثلاً در Nexus/Habor).
- **Base image را پین کنید به digest** (`@sha256:...`) نه tag.
- **Continuous Patching:** imageها را هفتگی بازسازی کنید.

### خلاصه فصل

امنیت کانتینر زنجیرهٔ تأمین را می‌سازد: **اسکن** (Trivy/Grype) آسیب‌پذیری، **امضا** (cosign/Sigstore) اصالت، و **SBOM** (Syft) شفافیت می‌دهد. تصاویر **distroless/multi-stage** سطح حمله را کاهش می‌دهند و **Admission Controller** اجرای تصاویر امضا‌نشده را مسدود می‌کند. اشتباهات: push بدون اسکن، `latest` در تولید، و عدم verify امضا در اجرا. با SBOM و امضا، پاسخ به حملهٔ جدید از «هفته‌ها» به «دقایق» می‌رسد. در فصل بعد مدیریت رمز و دسترسی را می‌بینیم.

---

## فصل ۱۱۹: مدیریت رمز و دسترسی امن + جمع‌بندی DevSecOps

### مدمه

آخرین ستون DevSecOps: **مدیریت رمز (Secrets)** و **دسترسی ایمن**. حتی با کد امن، رمز سخت‌کد شده یا IAM ضعیف همه چیز را نابود می‌کند. این فصل Vault، External Secrets، سیاست به‌عنوان کد (OPA/Kyverno)، Zero Trust و جمع‌بندی کل DevSecOps را پوشش می‌دهد.

### مفاهیم اساسی

**مدیریت رمز (Secrets Management):**
- **هیچ رمزی در کد/متغیر محیطی ایستا نباشد.** استفاده از **Vault** (HashiCorp) یا **AWS Secrets Manager / GCP Secret Manager / Azure Key Vault**.
- **Dynamic Secrets:** Vault رمز را به‌صورت موقت (short-lived) برای هر سرویس صادر می‌کند → چرخش خودکار.
- **External Secrets Operator** (فصل ۶۰): همگام‌سازی رمز از Vault/Cloud به K8s Secret.

**کمترین دسترسی (Least Privilege):**
- IAM Role/Policy با حداقل دسترسی (فصل ۹۰/۹۴).
- در K8s: **RBAC** (فصل ۵۹) + ServiceAccount محدود.
- **Workload Identity** (فصل ۹۴): کانتینر بدون کلید ثابت به سرویس ابری دسترسی یابد.

**سیاست به‌عنوان کد (Policy as Code):**
- **OPA (Open Policy Agent):** موتور سیاست عمومی با زبان Rego.
- **Kyverno:** سیاست K8s-native (YAML). هر دو در فصل ۶۰ دیده شدند — اینجا نقش امنیتی‌شان در DevSecOps.
- مثال: «هر Pod باید resource limits داشته باشد و با non-root اجرا شود».

**Zero Trust:**
- «هیچ چیز را به‌طور پیش‌فرض اعتماد نکن» — حتی درون شبکه داخلی.
- احراز هویت/مجوز در هر درخواست، رمزنگاری همه‌جا (mTLS).

**امنیت Pipeline:**
- دسترسی CI/CD محدود؛ توکن‌ها short-lived (OIDC به جای کلید ثابت).
- کانتینرهای CI ایزوله.

### مثال‌های عملی

**الف) Vault — دریافت رمز موقت (CLI):**
```bash
# احراز با نقش (نه کلید ثابت)
vault login -method=oidc
# دریافت رمز پایگاه‌داده (عمر ۱ ساعت)
vault read database/creds/app-role
# خروجی: username/password موقت → در برنامه استفاده شود
```

**ب) External Secrets (K8s ← Vault):**
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata: { name: db-secret }
spec:
  data:
    - secretKey: password
      remoteRef: { key: secret/db, property: password }
# اپلیکیشن فقط به kubectl get secret db-secret دسترسی دارد
```

**ج) سیاست Kyverno (اجبار non-root):**
```yaml
rules:
  - name: require-non-root
    match: { resources: { kinds: [Pod] } }
    validate:
      message: "Pod باید با کاربر غیرریشته اجرا شود"
      pattern:
        spec:
          securityContext:
            runAsNonRoot: true
```

**د) OIDC در CI (بدون کلید ثابت):**
```yaml
# GitHub Actions → AWS بدون Access Key
permissions: { id-token: write, contents: read }
jobs:
  deploy:
    steps:
      - uses: aws-actions/configure-aws-credentials@v4
        with: { role-to-assume: arn:aws:iam::123:role/gha-deploy, aws-region: us-east-1 }
```

### مقایسه و دیدگاه‌های مختلف

**Vault در برابر Cloud Secret Manager:**
- Vault: قدرتمند، dynamic secrets، multi-cloud، اما نگهداری خودش سخت.
- Cloud Manager (AWS/GCP/Azure): ساده‌تر، یکپارچه با ابر، اما قفل‌شده.

**OPA در برابر Kyverno:**
- OPA (Rego): انعطاف بیشتر، عمومی (فراتر از K8s).
- Kyverno: ساده‌تر (YAML)، مخصوص K8s.

**Zero Trust در برابر Perimeter Security:**
- Perimeter: «دیوار آتش بیرون، اعتماد درون» → خطر پس از نفوذ.
- Zero Trust: «هیچ اعتمادی» → ایمن‌تر در محیط ابری پویا.

**دیدگاه:** «رمز در کد مثل کلید زیر پادری است — راحت اما فاجعه‌بار». مدیریت متمرکز رمز غیرقابل مذاکره است.

### اشتباهات رایج

1. **رمز در متغیر محیطی ایستا / فایل `.env` در repo:** نشت آسان.
2. **کلید ثابت در CI (نه OIDC):** نشت → دسترسی کامل به ابر.
3. **دسترسی broad IAM ("*"):** نقض Least Privilege.
4. **عدم استفاده از External Secrets:** رمز دستی در K8s → چرخش سخت.
5. **سیاست فقط هشدار (نه اجبار):** در تولید اعمال نمی‌شود.
6. **Vault بدون تیم عملیاتی:** نصب اما بدون پشتیبانی → خودش نقطهٔ شکست می‌شود.

### نکات مهم و ترفندها

- **OIDC را جایگزین کلید ثابت در CI کنید** (GitHub/GitLab → AWS/GCP/Azure).
- **Vault Agent Sidecar** رمز را به‌روز در کانتینر تزریق کند.
- **Rotation خودکار:** رمزها هر ۳۰–۹۰ روز (یا موقت) چرخانده شوند.
- **Kyverno/Old policies** را در staging تست کنید تا تیم را مسدود نکند.
- **Audit Log** روی دسترسی به رمز فعال کنید (کی، کجا، چه زمانی).
- **Secrets Scanning** (فصل ۱۱۷) را همیشه روشن نگه دارید — خط دفاع آخر.

### خلاصه فصل (و جمع‌بندی DevSecOps)

مدیریت رمز و دسترسی سنگر پایانی DevSecOps است: **Vault/Cloud Manager** رمز را متمرکز و موقت نگه می‌دارند، **External Secrets** آن را به K8s می‌رساند، **Least Privilege/IAM/RBAC** دسترسی را محدود می‌کند، و **Policy as Code (OPA/Kyverno)** اجرای ایمن را اجباری می‌سازد. **Zero Trust** و **OIDC** جایگزین اعتماد کورکورانه و کلید ثابت می‌شوند. اشتباهات: رمز در کد و کلید ثابت در CI. 

**جمع‌بندی کل DevSecOps:** شیفت-به‌چپ (فصل ۱۱۶) + اسکن در pipeline (فصل ۱۱۷) + امنیت کانتینر/زنجیرهٔ تأمین (فصل ۱۱۸) + مدیریت رمز/دسترسی (فصل ۱۱۹) یک لایهٔ دفاع پیوسته می‌سازند. کلید موفقیت: خودکارسازی + فرهنگ مسئولیت اشتراکی + پیوستگی (همه چیز به‌عنوان کد). با این فصل، آخرین بخش فنی سری کامل شد.

---

## منابع و مراجع

- OWASP — https://owasp.org
  - Top 10: https://owasp.org/www-project-top-ten/
  - DevSecOps: https://owasp.org/www-project-devsecops-guideline/
  - ZAP: https://www.zaproxy.org/
- Semgrep — https://semgrep.dev/docs
- Trivy — https://trivy.dev/  ·  Syft/Sigstore/cosign — https://github.com/sigstore/cosign
- HashiCorp Vault — https://developer.hashicorp.com/vault
- Checkov — https://www.checkov.io/  ·  tfsec — https://github.com/aquasecurity/tfsec
- OPA / Rego — https://www.openpolicyagent.org/  ·  Kyverno — https://kyverno.io/
- SLSA Framework — https://slsa.dev/
- gitleaks — https://github.com/gitleaks/gitleaks
- «DevSecOps» (کتاب/منابع CNCF Security TAG)
- External Secrets Operator — https://external-secrets.io/

**بعدی:** [فایل ۲۵ — سرویس مش (Service Mesh: Istio و Linkerd)](25-Service-Mesh.md)
