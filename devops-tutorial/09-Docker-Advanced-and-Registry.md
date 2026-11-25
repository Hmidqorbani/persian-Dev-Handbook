# فایل ۰۹ — داکر پیشرفته و رجیستری

**مدت یادگیری**: ۹ ساعت
**سطح سختی**: متوسط–پیشرفته
**پیش‌نیاز**: فایل ۰۸ (مبانی داکر)، فایل ۲۴ (DevSecOps - مفاهیم اولیه)

---

## فهرست محتوا

- [فصل ۴۱: Image Registry و Repository (Docker Hub و رجیستری خصوصی)](#فصل-۴۱-image-registry-و-repository-docker-hub-و-رجیستری-خصوصی)
- [فصل ۴۲: استراتژی‌های نسخه‌گذاری و Tagging](#فصل-۴۲-استراتژیهای-نسخهگذاری-و-tagging)
- [فصل ۴۳: امضای Image و زنجیرهٔ تأمین (Signing و SBOM)](#فصل-۴۳-امضای-image-و-زنجیرهٔ-تأمین-signing-و-sbom)
- [فصل ۴۴: امنیت Runtime کانتینر (Capabilities، seccomp، Rootless)](#فصل-۴۴-امنیت-runtime-کانتینر-capabilities-seccomp-rootless)
- [فصل ۴۵: شبکه و ذخیره‌سازی پیشرفته داکر](#فصل-۴۵-شبکه-و-ذخیرهسازی-پیشرفته-داکر)

---

## فصل ۴۱: Image Registry و Repository (Docker Hub و رجیستری خصوصی)

### مقدمه

Imageهای داکر باید جایی ذخیره و توزیع شوند — آن «جا» **Registry** نامیده می‌شود. درک رجیستری‌ها، احراز هویت، و گزینه‌های خصوصی/عمومی برای هر خط لولهٔ CI/CD ضروری است. در این فصل Docker Hub، رجیستری‌های ابری (ECR/GCR/ACR) و رجیستری خصوصی را بررسی می‌کنیم.

### مفاهیم اساسی

**Registry چیست؟**
سرویسی برای ذخیره، نسخه‌گذاری و توزیع Imageها. فرمت استاندارد OCI را دنبال می‌کند.

**معماری نام‌گذاری:**
```
<registry>/<namespace>/<repository>:<tag>
مثال: registry.example.com/team/app:v1.2.3
```
اگر registry حذف شود، پیش‌فرض `docker.io` (Docker Hub) است.

**انواع Registry:**

| نوع | مثال | کاربرد |
|---|---|---|
| Public | Docker Hub | اشتراک متن‌باز |
| Cloud | AWS ECR، GCP Artifact Registry، Azure ACR | تولید ابری |
| Self-hosted | Harbor، GitLab Registry، Nexus | داخلی/سازمانی |
| Local | `registry:2` | تست/آفلاین |

**احراز هویت (Auth):**
`docker login <registry>` → توکن در `~/.docker/config.json` (رمزگذاری‌شده با base64 — نه رمزنگاری!).

**Pull/Push:**
```bash
docker pull nginx:alpine
docker tag myapp myapp:v1
docker push registry.example.com/team/myapp:v1
```

**Garbage Collection:** رجیستری‌های خصوصی باید Imageهای یتیم را پاک کنند (مثل Harbor که سیاست نگهداری دارد).

### مثال‌های عملی

**۱. استفاده از Docker Hub:**

```bash
docker login
docker pull ubuntu:22.04
docker tag ubuntu:22.04 myuser/ubuntu-custom:v1
docker push myuser/ubuntu-custom:v1
```

**۲. AWS ECR (Elastic Container Registry):**

```bash
# احراز با CLI
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin <account>.dkr.ecr.us-east-1.amazonaws.com

# تگ و پوش
docker tag myapp:latest <account>.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
docker push <account>.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
```

**۳. رجیستری خصوصی محلی:**

```bash
docker run -d -p 5000:5000 --name registry registry:2
docker tag myapp localhost:5000/myapp:v1
docker push localhost:5000/myapp:v1
```

**۴. Harbor (رجیستری سازمانی با امنیت):**

```
# ویژگی‌ها: RBAC، Vulnerability Scanning، Notary ( signing)، Replication
# نصب با Helm:
helm install harbor harbor/harbor
```

### مقایسه و دیدگاه‌های مختلف

| Registry | هزینه | ویژگی‌های امنیتی | مناسب برای |
|---|---|---|---|
| Docker Hub | Freemium | محدود | متن‌باز/کوچک |
| ECR | پرداخت به ازای استفاده | IAM، Scan | AWS |
| GCR/Artifact | پرداخت | IAM، Binary Auth | GCP |
| ACR | پرداخت | Entra ID، Policies | Azure |
| Harbor | رایگان (self-host) | RBAC، Notary، Scan | دیتاسنتر |

**دیدگاه:** برای تولید ابری، رجیستری بومی ابر (ECR/GCR/ACR) بهترین یکپارچه‌سازی با IAM را دارد. Harbor برای محیط‌های on-prem یا multi-cloud عالی است.

**امنیت:** هرگز رمز را در `config.json` نگه ندارید؛ از توکن موقت (OIDC/IAM) استفاده کنید.

### اشتباهات رایج

- **پوش داده‌های حساس در image:** رجیستری عمومی = لو رفتن رمز.
- **`config.json` با رمز در repo:** این فایل را در Git قرار ندهید.
- **استفاده از `latest` در تولید:** عدم تکرارپذیری.
- **فراموش سیاست نگهداری:** رجیستری پر از imageهای قدیمی.
- **دسترس پابلیک روی رجیستری خصوصی:** اشتباه در تنظیمات → image نشت.

### نکات مهم و ترفندها

- **CI از توکن موقت:** در GitHub Actions/GitLab، از OIDC برای ورود به ECR استفاده کنید (فصل ۲۸/۳۴).
- **Immutable tags:** در ECR/Harbor تگ را غیرقابل تغییر کنید تا بازنویسی جلوگیری شود.
- **Regional replication:** Harbor/GCR تصویر را در منطقهٔ نزدیک replicate می‌کنند (سرعت pull).
- **`.docker/config.json` با credential helper:** از `docker-credential-ecr-login` استفاده کنید.
- **Scan on push:** رجیستری بسیاری اسکن آسیب‌پذیری هنگام پوش انجام می‌دهند.
- **Pull-through cache:** رجیستری خصوصی می‌تواند Docker Hub را کش کند (جلوگیری از rate limit).

### خلاصه فصل

Registry محل ذخیره و توزیع Imageهاست. Docker Hub (عمومی)، ECR/GCR/ACR (ابری با IAM) و Harbor (سازمانی با RBAC/Scan/Sign) گزینه‌ها هستند. احراز با `docker login` و توکن در `config.json` (نه رمزنگاری‌شده — از IAM/OIDC استفاده کنید). کلید: Immutable tags، سیاست نگهداری، Scan on push، و هرگز رمز در image/Config.

---

## فصل ۴۲: استراتژی‌های نسخه‌گذاری و Tagging

### مقدمه

یک Image بدون استراتژی نسخه‌گذاری دردسرساز است: «کدام نسخه در تولید است؟» «چطور برگردیم؟». در این فصل الگوهای Tagging (Semantic Versioning، Git SHA،日期) و بهترین روش‌ها را بررسی می‌کنیم.

### مفاهیم اساسی

**چرا Tagging مهم است؟**
- **Traceability:** ردیابی کد پشت هر Image.
- **Rollback:** بازگشت به نسخهٔ شناخته‌شده.
- **Reproducibility:** ساخت دقیقاً یکسان.

**الگوهای رایج:**

| الگو | مثال | مزیت | معایب |
|---|---|---|---|
| Semantic Version | `v1.2.3` | معنادار (major/minor/patch) | نیاز به هماهنگی |
| Git SHA | `a1b2c3d` | دقیق، غیرقابل تغییر | غیرانسانی |
| Branch | `main`, `develop` | ساده | متحرک (mutable) |
| Date | `2026.10.17` | زمان‌محور | تداخل |
| `latest` | `latest` | ❌ پرهیز شود | غیرقابل تکرار |

**ترکیبی (توصیه‌شده):**
هر build همزمان چند تگ می‌گیرد:
```
myapp:1.2.3          # semver
myapp:1.2            # minor
myapp:1              # major
myapp:a1b2c3d        # git sha
```

**Immutable vs Mutable:**
تگ‌های `latest` و `main` **mutable** (تغییر می‌کنند) — خطرناک در تولید. تگ‌های SHA/semver باید **immutable** باشند.

**Conventional Commits → SemVer:** ابزارهایی مثل `semantic-release` از پیام کامیت (فصل ۱۱) نسخه را استخراج می‌کنند.

### مثال‌های عملی

**۱. تگ‌گذاری چندگانه در CI:**

```bash
VERSION=$(git describe --tags --always)
SHA=$(git rev-parse --short HEAD)
docker build -t myapp:$VERSION -t myapp:$SHA -t myapp:latest .
docker push myapp:$VERSION
docker push myapp:$SHA
# latest فقط برای dev
```

**۲. semantic-release در GitHub Actions:**

```yaml
- uses: cycjimmy/semantic-release-action@v4
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
# خروجی: نسخهٔ بعدی بر اساس commit ها
```

**۳. Immutable tag در ECR:**

```bash
aws ecr put-image-tag-immutability --repository-name myapp --image-tag-mutability IMMUTABLE
```

**۴. Rollback با تگ:**

```bash
kubectl set image deploy/myapp myapp=registry/myapp:v1.2.2
# یا
docker service update --image myapp:v1.2.2 myapp
```

### مقایسه و دیدگاه‌های مختلف

| استراتژی | مناسب برای |
|---|---|
| SemVer + SHA | تولید (توصیه) |
| Branch tags | تست/staging |
| `latest` | ❌ فقط محلی/موقت |

**دیدگاه DevOps:** همیشه Image را با **Git SHA** تگ کنید (قطعی‌ترین پیوند به کد). SemVer برای مصرف‌کنندهٔ انسانی. `latest` ممنوع در تولید.

**Clone tracking:** تگ SHA اجازه می‌دهد بدانید دقیقاً کدام کامیت در کدام پاد اجرا می‌شود (مهم برای دیباگ).

### اشتباهات رایج

- **فقط `latest`:** نمی‌دانید چه نسخه‌ای در تولید است؛ بازگشت غیرممکن.
- **بازنویسی تگ semver:** تگ `v1.2.3` را دوباره بسازید → تناقض.
- **عدم تگ SHA:** ردیابی باگ به کد غیرممکن.
- **Mutable tags در rollback:** `kubectl rollout undo` به تگ اشتباه اشاره می‌کند.
- **پاکسازی ناگهانی:** حذف تگی که در تولید است → کرش هنگام restart.

### نکات مهم و ترفندها

- **Digest هم ذخیره کنید:** `myapp@sha256:abc...` قطعی‌تر از تگ.
- **Automate با CI:** هرگز دستی تگ نزنید؛ CI انجام دهد.
- **Retention policy:** مثلاً ۱۰ نسخهٔ آخر + همهٔ release tags.
- **Promotion flow:** `dev → staging → prod` با تگ‌های متفاوت روی همان digest.
- **`IMAGE_DIGEST` در K8s:** از digest برای جلوگیری از pull غیرمنتظره استفاده کنید.

### خلاصه فصل

استراتژی نسخه‌گذاری = ردیابی + بازگشت + تکرارپذیری. توصیه: هر Image همزمان SemVer (انسانی) و Git SHA (دقیق) تگ بگیرد. `latest` در تولید ممنوع؛ تگ‌های semver/SHA باید immutable باشند. Rollback با تگ/digest انجام شود. کلید: SHA برای قطعیت، SemVer برای خوانایی، و Retention policy برای مدیریت فضا.

---

## فصل ۴۳: امضای Image و زنجیرهٔ تأمین (Signing و SBOM)

### مقدمه

امنیت فقط در لایهٔ شبکه نیست — باید مطمئن شویم Imageای که اجرا می‌کنید همانی است که ساخته‌اید و تغییر نکرده. **Image Signing** و **SBOM** بخشی از **Supply Chain Security** (امنیت زنجیرهٔ تأمین) هستند که در DevSecOps (فصل ۲۴) عمیق‌تر می‌شوند. این فصل پایه‌های آن را می‌گذارد.

### مفاهیم اساسی

**زنجیرهٔ تأمین نرم‌افزار (Software Supply Chain):**
هر چیزی که وارد build شما می‌شود: کد، کتابخانه‌ها، base image، ابزارها. حمله به زنجیره (مثل SolarWinds، Codecov) فاجعه‌بار است.

**SBOM (Software Bill of Materials):**
فهرست تمام اجزای نرم‌افزار (کتابخانه‌ها، ورژن‌ها، لایسنس). فرمت‌های استاندارد: **SPDX**، **CycloneDX**، **Syft**. لازم برای شناسایی آسیب‌پذیری (مثلاً Log4Shell).

**Image Signing:**
امضای رمزنگاری‌شده روی Image برای اثبات اصالت و عدم تغییر (integrity). ابزارها:
- **Notary / Docker Content Trust (DCT):** امضای سنتی داکر.
- **cosign (Sigstore):** مدرن، با KMS و Fulcio (گواهی کوتاه‌مدت)، پشتیبانی K8s policy (الگوی بسیار رایج امروز).
- **in-toto:** فرادادهٔ کل خط لوله.

**Verification (تأیید):**
در زمان اجرا (یا admission در K8s)، سیستم بررسی می‌کند Image امضا شده و توسط کلید معتبر است (مثلاً با **Connaisseur** یا **Kyverno**).

### مثال‌های عملی

**۱. تولید SBOM با Syft:**

```bash
syft myapp:latest -o spdx-json > sbom.spdx.json
syft myapp:latest -o cyclonedx-xml > sbom.cdx.xml
```

**۲. امضای Image با cosign:**

```bash
# کلید محلی (یا با KMS)
cosign generate-key-pair
cosign sign --key cosign.key registry/myapp:v1.2.3

# تأیید
cosign verify --key cosign.pub registry/myapp:v1.2.3
```

**۳. cosign با KMS (AWS KMS):**

```bash
cosign sign --key awskms:///alias/mykey registry/myapp:v1.2.3
```

**۴. فعال‌سازی Docker Content Trust:**

```bash
export DOCKER_CONTENT_TRUST=1
docker push myapp:v1   # نیاز به امضا
```

**۵. اسکن و SBOM در CI:**

```yaml
- name: Generate SBOM
  uses: anchore/sbom-action@v0
  with:
    image: myapp:${{ github.sha }}
- name: Scan
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: myapp:${{ github.sha }}
```

### مقایسه و دیدگاه‌های مختلف

| ابزار | کاربرد | مزیت |
|---|---|---|
| cosign (Sigstore) | امضا | مدرن، KMS، K8s policy |
| Notary/DCT | امضا (داکر) | یکپارچه با داکر |
| Syft/Grype | SBOM/اسکن | استاندارد صنعت |
| Trivy | اسکن | سریع، جامع |
| Kyverno/Connaisseur | اجرای policy | رد Image بی‌امضا در K8s |

**دیدگاه:** cosign + Sigstore استاندارد جدید است (حتی دولت آمریکا در فرمان اجرایی ۱۴۰۲۸ آن را تشویق می‌کند). SBOM دیگر اختیاری نیست — بسیاری از مشتریان آن را اجباری کرده‌اند.

**SLSA Framework:** چارچوبی برای تأمین «سطح اطمینان» زنجیره (Provenance). هدف: اثبات اینکه Image از کجا و چطور ساخته شده.

### اشتباهات رایج

- **عدم تولید SBOM:** هنگام افشای آسیب‌پذیری، نمی‌دانید کجا استفاده شده.
- **امضا بدون verification:** امضا بی‌فایده است اگر کسی چک نکند.
- **کلیدهای امضا در repo:** کلید خصوصی را در Git قرار ندهید؛ از KMS استفاده کنید.
- **فقط scan، بدون policy:** اسکن کافی نیست؛ image بی‌امضا باید رد شود.
- **SBOM فقط برای image نهایی:** SBOM باید تمام لایه‌ها (از base) را شامل شود.

### نکات مهم و ترفندها

- **Attestation:** cosign می‌تواند SBOM را به عنوان attestation ضمیمه کند (غیرقابل جدایی).
- **Keyless signing:** با Fulcio (Sigstore) نیاز به مدیریت کلید ندارید (گواهی کوتاه‌مدت).
- **Admission control:** در K8s، Kyverno/Connaisseur Imageهای بی‌امضا را رد کنند.
- **SBOM در Registry:** Harbor/GCR SBOM را نگه می‌دارند و جستجوی آسیب‌پذیری می‌دهند.
- **Provenance:** `cosign attest --type=slsaprovenance` برای ردیابی خط لوله.
- **Automate همه در CI:** امضا و SBOM باید گام ثابت خط لوله باشند.

### خلاصه فصل

امنیت زنجیرهٔ تأمین شامل SBOM (فهرست اجزا) و Image Signing (اثبات اصالت) است. cosign/Sigstore استاندارد مدرن امضا (با KMS یا keyless) و Syft/Trivy برای SBOM/اسکن هستند. امضا بدون verification (در K8s با Kyverno/Connaisseur) بی‌فایده است. کلید: SBOM خودکار در CI، امضا با KMS، رد Image بی‌امضا در admission، و پیروی از SLSA برای provenance.

---

## فصل ۴۴: امنیت Runtime کانتینر (Capabilities، seccomp، Rootless)

### مقدمه

حتی Image امن، در زمان اجرا می‌تواند خطرناک باشد. کانتینرها به طور پیش‌فرض ایزوله‌اند اما پیش‌فرض‌های داکر همیشه کافی نیست. در این فصل مکانیزم‌های امنیت runtime — capabilities، seccomp، AppArmor، و اجرای rootless — را بررسی می‌کنیم.

### مفاهیم اساسی

**ایزولاسیون کانتینر:**
کانتینرها از namespaces (PID، Network، Mount...) و cgroups استفاده می‌کنند اما **kernel را مشترک** دارند. پس escape از کانتینر = دستریر به kernel هاست.

**Linux Capabilities:**
به جای دادن دستریر root کامل، Linux قابلیت‌ها را تقسیم کرده (مثل `NET_BIND_SERVICE`، `SYS_ADMIN`). کانتینر پیش‌فرض مجموعه‌ای کوچک از capabilities دارد.

```bash
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE myapp
```

**seccomp (Secure Computing Mode):**
فیلتر فراخوان‌های سیستم (syscalls). پروفایل پیش‌فرض داکر بسیاری از syscalls خطرناک را مسدود می‌کند.

**AppArmor / SELinux:**
ماژول‌های امنیتی هسته که رفتار برنامه را محدود می‌کنند (مسیرهای فایل، شبکه).

**Rootless Docker:**
داکر دایمون را به عنوان کاربر غیر-root اجرا می‌کند. حتی اگر کانتینر escape کند، به سطح کاربر محدود است.

**Principle of Least Privilege:** کانتینر باید کمترین دسترس ممکن را داشته باشد.

### مثال‌های عملی

**۱. اجرا با کمترین دسترس:**

```bash
docker run -d \
  --cap-drop ALL \
  --security-opt no-new-privileges \
  --read-only \
  --user 1000:1000 \
  myapp:secure
```

**۲. فعال‌سازی seccomp سفارشی:**

```bash
docker run --security-opt seccomp=/path/to/profile.json myapp
# پروفایل پیش‌فرض: /usr/share/docker/seccomp/default.json
```

**۳. AppArmor:**

```bash
docker run --security-opt apparmor=docker-default myapp
# یا پروفایل سفارشی
docker run --security-opt apparmor=custom-profile myapp
```

**۴. Rootless Docker (نصب):**

```bash
# به عنوان کاربر عادی
curl -fsSL https://get.docker.com/rootless | sh
export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock
dockerd-rootless-setuptool.sh install
```

**۵. بدون privilege در Compose:**

```yaml
services:
  web:
    image: myapp
    read_only: true
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    user: "1000:1000"
```

### مقایسه و دیدگاه‌های مختلف

| مکانیزم | سطح | مزیت |
|---|---|---|
| `--user` | کانتینر | اجرای غیر-root |
| `cap-drop ALL` | هسته | حذف قابلیت‌های غیرضروری |
| `no-new-privileges` | هسته | جلوگیری از escalate |
| seccomp | سیستم‌عامل | فیلتر syscall |
| AppArmor/SELinux | هسته | محدودیت رفتار |
| Rootless | دایمون | daemon غیر-root |

**دیدگاه:** ترکیب `--user` + `cap-drop ALL` + `no-new-privileges` + `read-only` + seccomp پیکربندی «secure-by-default» خوبی است. Rootless برای محیط‌های چندمستأجری (multi-tenant) ضروری.

**Kubernetes:** این مفاهیم در K8s از طریق Pod Security Standards (فصل ۱۲) اعمال می‌شوند.

### اشتباهات رایج

- **اجرا به عنوان root:** حمله به daemon = root روی هاست.
- **`--privileged`:** دستریر کامل هسته → escape آسان (فقط برای dind مدیریت‌شده).
- **فراموش `no-new-privileges`:** برنامه می‌تواند setuid کند.
- **`cap-add SYS_ADMIN`:** تقریباً معادل root است.
- **Mounting دیسک هاست بدون محدودیت:** `/var/run/docker.sock` را به کانتینر ندهید (کنترل کل daemon).

### نکات مهم و ترفندها

- **`--security-opt no-new-privileges`:** همیشه فعال کنید.
- **داکر سوکت:** هرگز `/var/run/docker.sock` را mount نکنید مگر ابزار خاص (مثل CI runner مدیریت‌شده).
- **Read-only FS:** `--read-only` + volume برای نوشتن ضروری.
- **User namespaces remap:** نگاشت کاربر root کانتینر به کاربر غیرمجاز هاست (`--userns-remap`).
- **CIS Benchmark:** چک‌لیست امنیت داکر (داکر بنچ `docker-bench-security` اجرا کنید).
- **gVisor / Kata:** runtimeهای ایزوله‌تر (sandbox) برای چندمستأجری.

### خلاصه فصل

امنیت runtime شامل کمینه کردن دسترس است: `--user` غیر-root، `cap-drop ALL` برای حذف capabilities، `no-new-privileges` جلوی escalate، seccomp/AppArmor فیلتر رفتار، و Rootless برای daemon غیر-root. اصل Least Privilege. کلید: هرگز `--privileged` یا سوکت داکر را mount نکنید، `read-only` + volume استفاده کنید، و بنچ امنیتی (CIS) را بزنید. این‌ها در K8s با Pod Security Standards ادامه می‌یابند.

---

## فصل ۴۵: شبکه و ذخیره‌سازی پیشرفته داکر

### مدمه

برای سناریوهای پیچیده، شبکهٔ پیش‌فرض bridge کافی نیست. **شبکه‌های پیشرفته** (overlay، macvlan) و **درایورهای ذخیره‌سازی** (overlay2، btrfs) عملکرد و ایزولاسیون سطح بالا می‌دهند. این فصل پایه‌ای برای درک K8s networking و storage است.

### مفاهیم اساسی

**درایورهای شبکه داکر:**

| درایور | کاربرد |
|---|---|
| `bridge` | پیش‌فرض، تک‌هاست |
| `host` | اشتراک شبکهٔ هاست (بدون NAT) |
| `overlay` | چند هاست (Swarm)، VXLAN |
| `macvlan` | کانتینر MAC مستقل (like فیزیکی) |
| `none` | ایزوله کامل |

**شبکه overlay:**
در Swarm/K8s، کانتینرها روی هاست‌های مختلف با شبکهٔ مجازی (VXLAN) صحبت می‌کنند. رمزنگاری ترافیک اختیاری (IPSEC).

**macvlan:**
کانتینر مستقیماً با MAC آدرس خودش روی شبکهٔ فیزیکی ظاهر می‌شود — برای میراث‌خواهی (legacy) یا نیاز به IP واقعی.

**درایورهای ذخیره‌سازی (Storage Drivers):**
داکر چطور لایه‌ها را مدیریت می‌کند:
- **overlay2** (پیش‌فرض، بهترین): بر پایه OverlayFS.
- **btrfs / zfs:** ویژگی‌های پیشرفته اما پیچیده.
- **aufs / devicemapper:** قدیمی، اجتناب شود.

**Union Mount:** همهٔ لایه‌های فقط‌خواندنی + یک لایهٔ نوشتنی (copy-on-write) ترکیب می‌شوند تا FS یکپارچه دیده شود.

### مثال‌های عملی

**۱. شبکه overlay (Swarm):**

```bash
docker swarm init
docker network create -d overlay --attachable my-overlay
docker service create --network my-overlay --name web nginx
docker service create --network my-overlay --name api myapi
# web و api حتی روی هاست متفاوت همدیگر را می‌بینند
```

**۲. macvlan:**

```bash
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 mymacvlan
docker run -d --network mymacvlan --ip=192.168.1.100 myapp
```

**۳. بررسی درایور ذخیره‌سازی:**

```bash
docker info | grep -i storage
# Storage Driver: overlay2
```

**۴. تغییر درایور (در daemon.json):**

```json
{
  "storage-driver": "overlay2"
}
```

### مقایسه و دیدگاه‌های مختلف

| درایور | ایزولاسیون | پیچیدگی | کاربرد |
|---|---|---|---|
| bridge | متوسط | کم | تک‌هاست |
| overlay | بالا | متوسط | multi-host |
| macvlan | بالا | بالا | legacy/IP واقعی |
| host | کم | کم | عملکرد بالا |

**دیدگاه:** در تولید واقعی، شبکهٔ کانتینر توسط **Kubernetes** (CNI پلاگین‌ها مثل Calico/Flannel) مدیریت می‌شود، نه داکر overlay. اما درک آن‌ها پایهٔ لازم است.

**Storage driver:** overlay2 استاندارد طلایی است؛ هرگز به عقب (aufs) برنگردید.

### اشتباهات رایج

- **استفاده از `host` network در تولید:** همهٔ پورت‌ها روی هاست در دسترس → تداخل.
- **overlay بدون swarm:** overlay درایور نیاز به Swarm/K8s دارد (تک‌هاست کار نمی‌کند مگر with --attachable در swarm).
- **macvlan اشتباه:** کانتینر نمی‌تواند هاست را ببیند (hairpinning) — رایج و گیج‌کننده.
- **تغییر storage driver روی دادهٔ موجود:** imageها از بین می‌روند (بکاپ بگیرید).
- **عدم تطابق MTU:** overlay با MTU نامناسب → قطع ترافیک بزرگ.

### نکات مهم و ترفندها

- **MTU هم‌راستا:** اگر VXLAN استفاده می‌کنید، MTU کمتر (مثلاً ۱۴۵۰) تنظیم کنید.
- **Network aliases:** در compose، `aliases` برای نام‌های چندگانه سرویس.
- **Inspect network:** `docker network inspect mynet` برای دیباگ.
- **Storage در K8s:** از داکر volume ساده عبور کنید → CSI drivers / PV/PVC.
- **صرف‌نظر از داکر شبکه در K8s:** K8s خودش networking را مدیریت می‌کند (داکر فقط runtime).

### خلاصه فصل

شبکهٔ پیشرفته شامل overlay (multi-host با VXLAN) و macvlan (IP واقعی) است؛ در تولید واقعی K8s/CNI این را مدیریت می‌کند. درایور ذخیره‌سازی overlay2 استاندارد است (Union Mount با copy-on-write). کلید: MTU را برای overlay تنظیم کنید، از `host` network در تولید اجتناب کنید، و تغییر storage driver با بکاپ انجام شود. این پایه برای درک networking در Kubernetes (فصل ۱۱+) است.

---

## منابع و مراجع

- Docker Registry: https://docs.docker.com/registry/
- Harbor: https://goharbor.io
- AWS ECR: https://docs.aws.amazon.com/ecr/
- cosign / Sigstore: https://docs.sigstore.dev
- Syft & Grype: https://github.com/anchore/syft
- Trivy: https://trivy.dev
- SBOM / SPDX: https://spdx.dev
- SLSA Framework: https://slsa.dev
- Docker Security (CIS): https://www.cisecurity.org/benchmark/docker
- Rootless Docker: https://docs.docker.com/engine/security/rootless/
- Docker Storage Drivers: https://docs.docker.com/storage/storagedriver/

**بعدی:** [فایل ۱۰ — Podman و Buildah: جایگزین‌های بدون دایمون](10-Podman-and-Buildah.md)
