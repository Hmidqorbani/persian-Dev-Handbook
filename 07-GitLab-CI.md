# فایل ۰۷ — GitLab CI: یکپارچه و کامل

**مدت یادگیری**: ۸ ساعت
**سطح سختی**: مبتدی–متوسط
**پیش‌نیاز**: فایل ۰۱ تا ۰۴ (مفاهیم CI/CD)، حساب GitLab

---

## فهرست محتوا

- [فصل ۳۱: معماری GitLab CI (Runner، Executors، فایل پیکربندی)](#فصل-۳۱-معماری-gitlab-ci-runner-executors-فایل-پیکربندی)
- [فصل ۳۲: نوشتن `.gitlab-ci.yml` (مراحل، Jobها، Artifacts، Cache)](#فصل-۳۲-نوشتن-gitlab-ciyml-مراحل-jobها-artifacts-cache)
- [فصل ۳۳: انواع Pipeline (Parent/Child، Merge Request، DAG)](#فصل-۳۳-انواع-pipeline-parentchild-merge-request-dag)
- [فصل ۳۴: Variables، Environments و Secrets](#فصل-۳۴-variables-environments-و-secrets)
- [فصل ۳۵: مدیریت Runner و بهترین روش‌ها](#فصل-۳۵-مدیریت-runner-و-بهترین-روشها)

---

## فصل ۳۱: معماری GitLab CI (Runner، Executors، فایل پیکربندی)

### مقدمه

**GitLab CI/CD** بخشی از پلتفرم یکپارچهٔ GitLab است که SCM، CI/CD، Container Registry، و حتی Kubernetes را در یک جا جمع می‌کند. برخلاف GitHub Actions که جدا از SCM رشد کرد، GitLab از همان ابتدا CI را درون خود داشت. در این فصل اجزای معماری آن را می‌شناسیم.

### مفاهیم اساسی

**اجزای اصلی:**

| جزء | نقش |
|---|---|
| **GitLab Server** | میزبان مخزن، UI، و هماهنگ‌کنندهٔ Pipeline |
| **GitLab Runner** | فرآیندی که Jobها را اجرا می‌کند (مثل Agent در Jenkins) |
| **Executor** | محیط اجرای Job (Shell، Docker، Kubernetes، VirtualBox...) |
| **`.gitlab-ci.yml`** | فایل پیکربندی در ریشهٔ مخزن |
| **Pipeline** | مجموعه‌ای از Stageها و Jobها |

**GitLab Runner چیست؟**
برنامه‌ای (نوشته‌شده در Go) که روی ماشین شما نصب می‌شود، به GitLab متصل می‌شود و منتظر Job می‌ماند. GitLab Server وظیفه را می‌فرستد، Runner اجرا می‌کند و نتیجه را برمی‌گرداند.

**انواع Executor:**

| Executor | کاربرد |
|---|---|
| **Shell** | اجرا روی ماشین میزبان (ساده اما بی‌ایزولاسیون) |
| **Docker** | هر Job در کانتینر جدا (ایزولاسیون خوب) |
| **Docker-in-Docker (dind)** | ساخت image داکر درون Job |
| **Kubernetes** | هر Job در پاد K8s (مقیاس‌پذیر) |
| **SSH/VirtualBox/Parallels** | محیط‌های خاص |

**Shared vs Specific Runners:**
- **Shared:** توسط GitLab.com ارائه می‌شود (برای همه).
- **Specific:** فقط برای یک پروژه.
- **Group:** برای گروهی از پروژه‌ها.

### مثال‌های عملی

**۱. نصب GitLab Runner (لینوکس):**

```bash
# دانلود باینری
curl -L --output gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
chmod +x gitlab-runner
sudo mv gitlab-runner /usr/local/bin/

# ثبت (registration) با توکن پروژه
sudo gitlab-runner register \
  --url https://gitlab.com \
  --token <PROJECT_TOKEN> \
  --executor docker \
  --docker-image alpine:latest \
  --description "docker-runner-1"

# اجرا
sudo gitlab-runner start
```

**۲. فایل `.gitlab-ci.yml` حداقلی:**

```yaml
stages:
  - build
  - test

build_job:
  stage: build
  script:
    - echo "Building..."
    - make build

test_job:
  stage: test
  script:
    - make test
```

**۳. بررسی Runner متصل:**

```
پروژه → Settings → CI/CD → Runners
Status: 🟢 online
Executor: docker
Tags: docker, linux
```

### مقایسه و دیدگاه‌های مختلف

| ویژگی | GitLab CI | GitHub Actions |
|---|---|---|
| یکپارچگی | کامل (SCM+CI+Registry) | CI جدا (اما یکپارچه با GitHub) |
| پیکربندی | `.gitlab-ci.yml` | `workflows/*.yml` |
| Runner | GitLab Runner (شما نصب) | GitHub-hosted یا self |
| DAG | `needs` داخلی | `needs` داخلی |
| قیمت‌گذاری | دقیقه‌ای (SaaS) یا self-hosted رایگان | مشابه |

**دیدگاه‌ها:**
- GitLab برای سازمان‌هایی که می‌خواهند «یک پلتفرم برای همه چیز» (single pane of glass) عالی است.
- GitHub Actions برای اکوسیستم GitHub ساده‌تر است.
- هر دو YAML-based و یادگیری یکی، دیگری را آسان می‌کند.

### اشتباهات رایج

- **فراموش کردن ثبت Runner:** Pipeline در وضعیت «pending» می‌ماند (هیچ Runner آزاد نیست).
- **Executor اشتباه:** مثلاً دستور داکر در Shell executor (داکر نصب نیست).
- **فایل در ریشه نیست:** `.gitlab-ci.yml` باید در ریشه (یا مسیر تعریف‌شده) باشد.
- **YAML نادرست:** تورفتگی (indentation) اشتباه → خطای parse.
- **Runner بدون tag:** اگر Job `tags` دارد، Runner باید همان tag را داشته باشد.

### نکات مهم و ترفندها

- **Tag-based routing:** Job را با `tags: [docker]` به Runner خاص هدایت کنید.
- **Concurrent jobs:** در `config.toml` تعداد `concurrent` را تنظیم کنید.
- **`gitlab-runner verify`:** بررسی سلامت Runner.
- **Helm chart:** برای اجرای Runner در K8s از chart رسمی استفاده کنید.
- **Auto DevOps:** GitLab قابلیت «Auto DevOps» دارد که بدون نوشتن YAML، pipeline پیش‌فرض می‌سازد (برای شروع سریع).

### خلاصه فصل

GitLab CI بخشی از پلتفرم یکپارچهٔ GitLab است. GitLab Runner (برنامه Go) Jobها را با Executorهای مختلف (Shell/Docker/K8s) اجرا می‌کند. پیکربندی در `.gitlab-ci.yml` در ریشه است. مزیت: یکپارچگی کامل SCM+CI+Registry. کلید: Runner را ثبت کنید، Executor مناسب انتخاب کنید، و Jobها را با tags به Runner هدایت کنید.

---

## فصل ۳۲: نوشتن `.gitlab-ci.yml` (مراحل، Jobها، Artifacts، Cache)

### مقدمه

فایل `.gitlab-ci.yml` قلب GitLab CI است. ساختار آن شبیه GitHub Actions اما با تفاوت‌های ظریف: مفهوم صریح `stages`، قابلیت `artifacts`/`cache` داخلی، و `rules` برای کنترل اجرا. در این فصل یک فایل کامل می‌نویسیم.

### مفاهیم اساسی

**ساختار پایه:**

```yaml
stages:
  - build
  - test
  - deploy

job_name:
  stage: build
  script:
    - echo "command"
  artifacts: {...}
  cache: {...}
  rules: [...]
```

**عناصر کلیدی:**
- `stages`: لیست مراحل به ترتیب اجرا.
- `job`: یک کار؛ `stage` مشخص می‌کند کجای نوار قرار گیرد.
- `script`: دستورات (اجباری).
- `before_script` / `after_script`: قبل/بعد از هر job.
- `artifacts`: خروجی برای دانلود یا job بعدی.
- `cache`: داده برای سرعت (بین اجراها).
- `rules`: چه زمانی job اجرا شود (جایگزین `only/except` قدیمی).
- `needs`: وابستگی صریح (DAG — اجرای موازی هوشمند).
- `tags`: هدایت به Runner خاص.
- `image`: ایمیج داکر برای Job (در Docker executor).

**تفاوت artifacts و cache:**
- `artifacts`: برای انتقال خروجی (مثل باینری) — منقضی‌شوند (`expire_in`).
- `cache`: برای فایل‌های تکراری (مثل `node_modules`) — برای سرعت.

### مثال‌های عملی

**۱. Pipeline کامل:**

```yaml
stages:
  - build
  - test
  - deploy

variables:
  NODE_VERSION: "20"

build:
  stage: build
  image: node:$NODE_VERSION
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

unit_test:
  stage: test
  image: node:$NODE_VERSION
  script:
    - npm ci
    - npm test
  coverage: '/Coverage: \d+\.\d+%/'

integration_test:
  stage: test
  needs: [build]
  image: node:$NODE_VERSION
  script:
    - npm ci
    - npm run test:integration

deploy_staging:
  stage: deploy
  environment:
    name: staging
  rules:
    - if: '$CI_COMMIT_BRANCH == "develop"'
  script:
    - ./deploy.sh staging

deploy_prod:
  stage: deploy
  environment:
    name: production
  when: manual
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
  script:
    - ./deploy.sh production
```

**۲. Cache موثر:**

```yaml
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
    - .npm/
  policy: pull-push   # یا pull برای jobهای فقط-خواندنی
```

**۳. Rule-based اجرا:**

```yaml
job:
  script: echo "only on tags"
  rules:
    - if: $CI_COMMIT_TAG
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: never
```

**۴. Matrix build:**

```yaml
test:
  stage: test
  parallel:
    matrix:
      - PYTHON: ["3.10", "3.11", "3.12"]
  script:
    - python --version
```

### مقایسه و دیدگاه‌های مختلف

| ویژگی | GitLab CI | GitHub Actions |
|---|---|---|
| Stage صریح | بله (`stages`) | نه (فقط `needs`) |
| Rules | قدرتمند (`rules`) | `if`/`when` |
| Artifacts | داخلی | action نیاز |
| Cache | داخلی | action نیاز |
| DAG | `needs` | `needs` |

**دیدگاه:** `rules` در GitLab از `only/except` قدیمی قدرتمندتر است؛ همیشه از `rules` استفاده کنید. Stage صریح برای درک بصری بهتر است.

**Fail-fast:** `interruptible: true` روی jobهای طولانی تا وقتی push جدید می‌آید، job قبلی لغو شود.

### اشتباهات رایج

- **فراموش `expire_in` در artifacts:** فضا پر می‌شود.
- **استفاده از `only/except` (منسوخ):** از `rules` استفاده کنید.
- **cache بدون key:** همهٔ branchها کش یکسان → تداخل.
- **Job بدون stage:** به طور پیش‌فرض `test` می‌رود (گیج‌کننده).
- **فراموش `tags`:** اگر Runner شما tagged است، Job اجرا نمی‌شود.

### نکات مهم و ترفندها

- **`default:` بخش:** تنظیمات مشترک را یک‌بار بنویسید:
  ```yaml
  default:
    image: node:20
    tags: [docker]
  ```
- **`interruptible`:** برای لغو jobهای قدیمی هنگام push جدید.
- **`.gitlab-ci.yml` شامل (include):** فایل‌های مشترک را include کنید:
  ```yaml
  include:
    - '/.gitlab/ci/build.yml'
  ```
- **`extends`:** وراثت job:
  ```yaml
  .base_test: &base
    image: node:20
  test_unit:
    <<: *base
    script: npm test
  ```
- **Pipeline Editor:** UI آنلاین GitLab برای اعتبارسنجی YAML.

### خلاصه فصل

`.gitlab-ci.yml` شامل `stages` (مراحل صریح) و `jobs` با `script` است. `artifacts` برای انتقال خروجی، `cache` برای سرعت، `rules` برای کنترل اجرا (جایگزین only/except)، و `needs` برای DAG. کلید: `expire_in` برای artifacts، `key` برای cache، `tags` برای هدایت Runner، و استفاده از `default`/`include`/`extends` برای DRY.

---

## فصل ۳۳: انواع Pipeline (Parent/Child، Merge Request، DAG)

### مقدمه

GitLab چندین الگوی Pipeline را پشتیبانی می‌کند که برای مقیاس و همکاری طراحی شده‌اند. **Parent-Child Pipelines** پیچیدگی را تقسیم می‌کنند، **Merge Request Pipelines** کیفیت PR را تضمین می‌کنند، و **DAG** (با `needs`) اجرای موازی بهینه می‌دهد. این فصل این الگوها را بررسی می‌کند.

### مفاهیم اساسی

**۱. DAG Pipelines (با `needs`):**
به جای اجرای ترتیبی stageها，jobها مستقیماً به هم وابسته می‌شوند. GitLab گراف اجرا می‌سازد و هرچه آماده شد را زودتر اجرا می‌کند.

**۲. Parent-Child Pipeline:**
Pipeline اصلی (parent) یک یا چند pipeline فرعی (child) را trigger می‌کند. برای جدا کردن منطق بزرگ به فایل‌های کوچک (مثل Monorepo).

**۳. Merge Request (MR) Pipeline:**
وقتی روی MR (شاخهٔ feature) اجرا می‌شود，قبل از ادغام. اطمینان از سبز بودن قبل از merge.

**۴. Merged Result Pipeline:** نسخهٔ ادغام‌شدهٔ (merged) branch با `main` را تست می‌کند (جلوگیری از شکست بعد از merge).

**۵. Pipelines for Fork:** اجرای تست روی MR از fork (با تنظیم دسترسی).

**Trigger keyword:** برای راه‌اندازی child pipeline یا pipeline در پروژهٔ دیگر.

### مثال‌های عملی

**۱. DAG با `needs`:**

```yaml
stages: [build, test, deploy]
build:
  stage: build
  script: make build
test_unit:
  stage: test
  needs: [build]    # فقط منتظر build، نه کل stage
  script: make unit
test_int:
  stage: test
  needs: [build]
  script: make int
deploy:
  stage: deploy
  needs: [test_unit, test_int]
  script: make deploy
```

**۲. Parent-Child Pipeline:**

```yaml
# در .gitlab-ci.yml (parent)
microservices:
  trigger:
    include:
      - local: /ci/microservices.gitlab-ci.yml
    strategy: depend   # منتظر نتیجهٔ child بمان

# در ci/microservices.gitlab-ci.yml (child)
build_service_a:
  script: make service-a
```

**۳. MR Pipeline:**

```yaml
job:
  script: npm test
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
```

**۴. Merged Result:**

```
Settings → Merge requests → Enable "Merged results pipelines"
# حالا تست روی ترکیب branch + target اجرا می‌شود
```

### مقایسه و دیدگاه‌های مختلف

| الگو | مزیت | معایب |
|---|---|---|
| Stage خطی | ساده | کند (سریال) |
| DAG (needs) | موازی‌سازی بهینه | پیچیدگی گراف |
| Parent-Child | تقسیم پیچیدگی | دیباگ سخت‌تر |
| MR Pipeline | کیفیت قبل از merge | زمان اضافه |

**دیدگاه DevOps:** MR Pipelines + Merged Result بهترین تضمین کیفیت قبل از ادغام را می‌دهند. DAG برای سرعت pipeline ضروری است. Parent-Child برای Monorepo (فصل ۱۵) عالی است.

**GitLab vs GitHub:** مفهوم `needs` در هر دو هست؛ MR Pipeline مشابه PR Checks در GitHub.

### اشتباهات رایج

- **بدون `needs` در pipeline بزرگ:** همه چیز serial → کند.
- **Parent-Child بدون `strategy: depend`:** parent منتظر child نمی‌ماند → نتیجه نادیده گرفته می‌شود.
- **MR pipeline بدون rule:** روی همهٔ push اجرا می‌شود (هدر رفت منابع).
- **Dependency cycle:** A نیاز به B، B نیاز به A → خطا.
- **فراموش کردن merged result:** شکست بعد از merge (تست فقط روی branch جدا بود).

### نکات مهم و ترفندها

- **`needs: [job, optional:other]`:** job اختیاری (اگر وجود داشت صبر کن).
- **Child with variables:** متغیر به child پاس بدهید:
  ```yaml
  trigger:
    include: child.yml
    forward:
      pipeline_variables: true
  ```
- **`workflow: rules`:** کنترل کلی چه زمانی pipeline ساخته شود.
- **Pipeline Graph:** از نمایش گراف برای دیدن وابستگی‌ها استفاده کنید.
- **Cancellation:** `interruptible: true` روی childها.

### خلاصه فصل

GitLab الگوهای پیشرفته دارد: DAG (با `needs` برای موازی‌سازی بهینه)، Parent-Child (تقسیم پیچیدگی، مناسب Monorepo)، و MR Pipelines (تست قبل از ادغام). Merged Result حتی نسخهٔ ادغام‌شده را تست می‌کند. کلید: `needs` برای سرعت، `strategy: depend` برای منتظر ماندن روی child، و MR rules برای اجرای فقط روی درخواست ادغام.

---

## فصل ۳۴: Variables، Environments و Secrets

### مقدمه

مدیریت پیکربندی و رمز در GitLab CI از طریق **CI/CD Variables**، **Environments** و **Protected Branches** انجام می‌شود. این فصل نشان می‌دهد چطور رمزها را ایمن نگه دارید و استقرار را با محیط‌های تست/تولید کنترل کنید — با دروازهٔ تأیید انسانی.

### مفاهیم اساسی

**انواع Variables:**
- **Project variables:** در `Settings → CI/CD → Variables` — برای پروژه.
- **Group variables:** برای گروه پروژه‌ها.
- **Instance variables:** سراسری (admin).
- **`.gitlab-ci.yml` variables:** محلی در فایل (نه حساس).
- **Predefined variables:** مثل `CI_COMMIT_SHA`، `CI_PIPELINE_ID`، `CI_ENVIRONMENT_NAME`.

**پرچم‌های Variable:**
- **Protected:** فقط روی protected branches/tags در دسترس.
- **Masked:** در لاگ ماسک می‌شود (باید فرمت خاص داشته باشد، مثلاً base64 یا ۸+ کاراکتر).
- **Hidden (Expandable):** کنترل تعویض متغیر.

**Environments:**
مثل GitHub، محیط‌های هدف (staging/production) با تاریخچهٔ Deployment. قابلیت **Manual Job** (دروازهٔ تأیید) دارد.

**Protected Branches:**
شاخه‌هایی که فقط با دستراص کافی قابل push هستند؛ متغیرهای Protected فقط آنجا فعال‌اند.

### مثال‌های عملی

**۱. تعریف Variable در YAML:**

```yaml
variables:
  BUILD_DIR: "build"
  # حساس نیست → در YAML اوکی

deploy:
  script:
    - echo "Deploying $CI_COMMIT_SHA to $ENV_NAME"
```

**۲. استفاده از Variable تعریف‌شده در UI (Protected + Masked):**

```yaml
deploy_prod:
  stage: deploy
  environment:
    name: production
  when: manual
  script:
    - ./deploy.sh --token $PROD_TOKEN
```

**۳. Environment با URL و Knee:**

```yaml
deploy_staging:
  environment:
    name: staging
    url: https://staging.example.com
  script:
    - ./deploy.sh staging
  rules:
    - if: $CI_COMMIT_BRANCH == "develop"

deploy_prod:
  environment:
    name: production
    url: https://example.com
  when: manual    # دروازهٔ تأیید
  allow_failure: false
  script:
    - ./deploy.sh prod
```

**۴. Dynamic Environment (Review Apps):**

```yaml
review_app:
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    on_stop: stop_review
  script:
    - ./deploy_review.sh
stop_review:
  when: manual
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    action: stop
  script:
    - ./stop_review.sh
```

### مقایسه و دیدگاه‌های مختلف

| مکانیسم | GitLab | GitHub |
|---|---|---|
| Variables | CI/CD Variables (Protected/Masked) | Secrets |
| Environments | Environments + Manual | Environments + Approver |
| Masking | خودکار (با شرط) | خودکار |
| Review Apps | بله (built-in) | نیاز به اسکریپت |

**دیدگاه:** GitLab Environments با Review Apps (محیط موقت برای هر MR) برتری دارد — برای تست تغییرات قبل از ادغام عالی است. Protected + Masked برای production توصیه می‌شود.

**Least Privilege:** متغیرهای production را Protected کنید تا فقط روی `main` فعال شوند.

### اشتباهات رایج

- **رمز در YAML:** هرگز رمز واقعی را در فایل ننویسید (حتی اگر masked باشد، در repo است).
- **فراموش Masked بودن:** متغیر کوتاه‌تر از ۸ کاراکتر یا با کاراکتر خاص ماسک نمی‌شود.
- **بدون Protected روی production token:** روی هر branch فعال می‌شود (نشت).
- **`when: manual` بدون `allow_failure: false`:** کسی approve نکند، pipeline «موفق» فرض می‌شود.
- **اکسپوز کردن Variable در لاگ:** `echo $TOKEN` حتی با mask خطرناک است.

### نکات مهم و ترفندها

- **File-type variable:** برای فایل‌های حساس (مثل کلید JSON) از نوع `file` استفاده کنید (محتوا در فایل موقت می‌رود، نه متغیر).
- **Scope variables:** متغیر را فقط برای محیط خاص (Environment scope) محدود کنید.
- **Predefined vars:** از `CI_COMMIT_TAG`، `CI_MERGE_REQUEST_ID` برای منطق استفاده کنید.
- **Secret detection:** GitLab پلاگین Escaped Secrets دارد که رمز را در لاگ پیدا می‌کند.
- **Vault integration:** GitLab می‌تواند از HashiCorp Vault توکن بگیرد (فصل بعدی‌ها).
- **`.gitlab-ci.yml` lint:** API `/ci/lint` برای اعتبارسنجی.

### خلاصه فصل

GitLab Variables (Project/Group/Instance) با پرچم Protected (فقط روی branch محافظت‌شده) و Masked (مخفی در لاگ) رمزها را ایمن می‌کند. Environments محیط‌های هدف با تاریخچه و Manual Job (دروازهٔ تأیید) هستند. Review Apps محیط موقت برای هر MR می‌سازد. کلید: رمز هرگز در YAML، Protected+Masked برای production، و `allow_failure: false` برای manual jobهای حیاتی.

---

## فصل ۳۵: مدیریت Runner و بهترین روش‌ها

### مقدمه

Runnerها کار واقعی را انجام می‌دهند، پس مدیریت درست آن‌ها مستقیماً روی سرعت، هزینه و امنیت اثر می‌گذارد. در این فصل نصب مقیاس‌پذیر Runner (مخصوصاً در Kubernetes)، ایزولاسیون، و چک‌لیست بهترین روش‌های GitLab CI را می‌آوریم.

### مفاهیم اساسی

**مقیاس‌پذیری Runner:**
- **Static:** تعداد ثابت Runner روی ماشین‌ها.
- **Autoscaling:** GitLab Runner می‌تواند در ابر (AWS/Azure) ماشین جدید ایجاد کند وقتی صف پر شود (با Docker Machine — اکنون deprecated، توصیه K8s).
- **Kubernetes Executor:** هر Job در پاد جدا → ایزولاسیون کامل و autoscaling طبیعی.

**ایزولاسیون:**
- هر Job در کانتینر/پاد جدا اجرا شود (Shell executor ایزولاسیون ندارد).
- استفاده از `cleanup` خودکار برای حذف state.

**امنیت Runner:**
- Runner نباید دسترسی ادمین داشته باشد.
- Protected Runners فقط برای branchهای محافظت‌شده.
- تگ‌گذاری دقیق برای جلوگیری از اجرای Job حساس روی Runner اشتباه.

**بهترین روش‌ها (Checklist):**
1. Docker/K8s executor برای ایزولاسیون.
2. `cache` و `artifacts` با کلید مناسب.
3. `rules` به جای `only/except`.
4. DRY با `include`/`extends`.
5. Manual job برای production.
6. Pipeline کوتاه (fail fast).
7. Protected variables برای رمز.

### مثال‌های عملی

**۱. نصب Runner در Kubernetes (Helm):**

```bash
helm repo add gitlab https://charts.gitlab.io
helm install gitlab-runner gitlab/gitlab-runner \
  --namespace gitlab-runner \
  --set gitlabUrl=https://gitlab.com \
  --set runnerRegistrationToken=<TOKEN> \
  --set rbac.create=true
```

**۲. config.toml برای Docker executor:**

```toml
[[runners]]
  name = "docker-runner"
  url = "https://gitlab.com"
  token = "<TOKEN>"
  executor = "docker"
  [runners.docker]
    image = "alpine:latest"
    privileged = false      # امنیت: false نگه دارید
    volumes = ["/cache"]
  [runners.cache]
    Type = "s3"
    Shared = true
```

**۳. Autoscaling (مفهوم K8s):**

```yaml
# values.yaml برای Helm
runner:
  concurrent: 10
  tagList: "k8s"
  runUntagged: false
```

**۴. بررسی سلامت Runner:**

```bash
gitlab-runner status
gitlab-runner verify --delete   # حذف Runnerهای غیرفعال
```

### مقایسه و دیدگاه‌های مختلف

| Executor | ایزولاسیون | مقیاس‌پذیری | پیچیدگی |
|---|---|---|---|
| Shell | کم | دستی | کم |
| Docker | متوسط | متوسط | کم |
| Kubernetes | بالا | بالا (اتوماتیک) | متوسط |
| Docker Machine (deprecated) | بالا | بالا | بالا |

**دیدگاه:** برای تولید، Kubernetes executor بهترین تعادل ایزولاسیون/مقیاس است. Shell executor فقط برای تست محلی. `privileged: false` همیشه (مگر dind که مدیریت شده باشد).

**GitLab.com shared runners:** برای شروع، از shared runners استفاده کنید؛ بعد به specific/K8s مهاجرت کنید.

### اشتباهات رایج

- **Shell executor در تولید:** بدون ایزولاسیون، تداخل Jobها.
- **`privileged: true` بی‌دلیل:** ریسک امنیتی بالا (دستریر به داکر هاست).
- **Runner بدون tag روی پروژه حساس:** Jobهای هرزه روی آن اجرا می‌شوند.
- **عدم autoscaling:** صف طولانی در زمان شلوغی.
- **فراموش بکاپ config.toml:** پیکربندی Runner از دست می‌رود.

### نکات مهم و ترفندها

- **`runUntagged: false`:** فقط Jobهای tagged اجرا شوند (کنترل دقیق).
- **Node affinity:** در K8s، Runner را روی نود خاص با resource guarantee بگذارید.
- **Metrics:** GitLab Runner متریک Prometheus می‌دهد — مانیتور کنید.
- **`concurrent`:** تعداد Job هم‌زمان را بر اساس منابع تنظیم کنید.
- **Cleanup policy:** imageهای داکر قدیمی را پاک کنید (`docker image prune`).
- **Tag strategy:** `tags: [docker, linux]` دقیق‌تر از `tags: [docker]`.

### خلاصه فصل

مدیریت Runner مستقیماً روی سرعت/امنیت اثر دارد. Kubernetes executor بهترین ایزولاسیون و autoscaling را می‌دهد؛ Shell executor فقط برای تست محلی. کلید: `privileged: false`، `runUntagged: false`، tag دقیق، مانیتورینگ متریک، و بکاپ `config.toml`. بهترین روش‌ها: Docker/K8s executor، DRY با include/extends، Manual job برای production، و Protected variables.

---

## منابع و مراجع

- GitLab CI/CD Documentation: https://docs.gitlab.com/ee/ci/
- GitLab Runner: https://docs.gitlab.com/runner/
- .gitlab-ci.yml Reference: https://docs.gitlab.com/ee/ci/yaml/
- CI/CD Variables: https://docs.gitlab.com/ee/ci/variables/
- Environments & Deployments: https://docs.gitlab.com/ee/ci/environments/
- Parent-Child Pipelines: https://docs.gitlab.com/ee/ci/pipelines/parent_child_pipelines.html
- GitLab Runner Helm Chart: https://docs.gitlab.com/runner/install/kubernetes.html
- Review Apps: https://docs.gitlab.com/ee/ci/review_apps/

**بعدی:** [فایل ۰۸ — مبانی داکر و کانتینریزاسیون](08-Docker-Fundamentals.md)
