# فایل ۰۶ — GitHub Actions: CI/CD در اکوسیستم GitHub

**مدت یادگیری**: ۸ ساعت
**سطح سختی**: مبتدی–متوسط
**پیش‌نیاز**: فایل ۰۱ تا ۰۴ (مفاهیم CI/CD)، حساب GitHub

---

## فهرست محتوا

- [فصل ۲۶: معماری GitHub Actions (رویداد، Workflow، Job، Runner)](#فصل-۲۶-معماری-github-actions-رویداد-workflow-job-runner)
- [فصل ۲۷: نوشتن Workflow (YAML، Matrix، Artifacts، Caching)](#فصل-۲۷-نوشتن-workflow-yaml-matrix-artifacts-caching)
- [فصل ۲۸: Secrets، Environments و Approvals](#فصل-۲۸-secrets-environments-و-approvals)
- [فصل ۲۹: Reusable Workflows و Composite Actions](#فصل-۲۹-reusable-workflows-و-composite-actions)
- [فصل ۳۰: Self-hosted Runners و بهترین روش‌ها](#فصل-۳۰-self-hosted-runners-و-بهترین-روشها)

---

## فصل ۲۶: معماری GitHub Actions (رویداد، Workflow، Job، Runner)

### مقدمه

**GitHub Actions** سرویس CI/CD داخلی GitHub است که مستقیماً با مخازن شما یکپارچه شده. برخلاف Jenkins که نیاز به سرور جداگانه دارد، Actions «در همان‌جا که کد است» اجرا می‌شود — همین باعث محبوبیت انفجاری‌اش شده است. در این فصل اجزای معماری آن را کالبدشکافی می‌کنیم.

### مفاهیم اساسی

**اجزای اصلی:**

| جزء | تعریف |
|---|---|
| **Event** | محرک (trigger): `push`، `pull_request`، `schedule`، `release` |
| **Workflow** | یک فایل YAML در `.github/workflows/` که خط لوله را تعریف می‌کند |
| **Job** | مجموعه‌ای از steps روی یک Runner (به صورت پیش‌فرض موازی با jobهای دیگر) |
| **Step** | یک دستور (run) یا فراخوانی Action |
| **Action** | واحد قابل بازاستفاده (مثل `actions/checkout`) — در بازارچه (Marketplace) |
| **Runner** | ماشینی که Job را اجرا می‌کند (GitHub-hosted یا self-hosted) |

**جریان اجرا:**

```
Event (push به main)
   ↓
Workflow فعال می‌شود
   ↓
Job ها (موازی، مگر با needs)
   ↓
Steps (run / uses)
   ↓
Runner (ubuntu-latest و...)
```

**GitHub-hosted Runners:** ماشین‌های ابری آماده (Ubuntu، Windows، macOS). رایگان برای public repo (دقایق محدود برای private).

**Marketplace:** هزاران Action آماده مثل `actions/checkout@v4`، `actions/setup-node@v4`، `docker/build-push-action`.

### مثال‌های عملی

**۱. ساختار پوشه:**

```
myapp/
└── .github/
    └── workflows/
        └── ci.yml
```

**۲. یک Workflow حداقلی:**

```yaml
name: CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Hello from GitHub Actions"
```

**۳. بررسی Runner در حال اجرا:**

```
Repo → Actions → یک اجرا → "Set up job" log:
  Runner image: ubuntu-22.04
  Memory: 7GB
  Disk: 90GB
```

**۴. Eventهای رایج:**

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 2 * * *'   # هر روز ساعت ۲ بامداد
  release:
    types: [published]
```

### مقایسه و دیدگاه‌های مختلف

| ویژگی | GitHub Actions | Jenkins |
|---|---|---|
| نصب | بدون نصب (SaaS) | نیاز به سرور |
| یکپارچه‌سازی | کامل با GitHub | نیاز به پلاگین |
| انعطاف Groovy | نه (YAML ساده) | بله (بی‌نهایت) |
| Self-hosted | ساده | ساده |
| هزینه | دقیقه‌ای (private) | سرور شما |

**دیدگاه‌ها:**
- تیم‌های روی GitHub: Actions انتخاب اول (کمترین اصطکاک).
- نیازهای بسیار پیچیده/خاص: Jenkins یا GitLab CI ممکن است منعطف‌تر باشد.
- Actions برای اکثر پروژه‌ها «کافی و عالی» است.

### اشتباهات رایج

- **فراموش کردن `actions/checkout`:** بدون آن، کد مخزن چک‌اوت نمی‌شود (پوشه خالی).
- **نسخهٔ ثابت ندادن به Action:** `uses: actions/checkout@v4` (نه `@main` که ناپایدار است).
- **`runs-on` اشتباه:** مثلاً ابزار ویندوزی روی `ubuntu` اجرا شود.
- **Secret در لاگ:** رمز را `echo` نکنید؛ GitHub آن را ماسک می‌کند اما احتیاط کنید.
- **Workflow بیش از حد طولانی:** خوانایی کم می‌شود؛ Reusable استفاده کنید (فصل ۲۹).

### نکات مهم و ترفندها

- **Versions pinning:** همیشه `@v4` یا commit SHA دقیق بدهید (امنیت).
- **Concurrency:** جلوگیری از اجرای هم‌زمان روی یک محیط:
  ```yaml
  concurrency:
    group: deploy-${{ github.ref }}
    cancel-in-progress: true
  ```
- **`github` context:** دسترسی به اطلاعات اجرا (`github.sha`، `github.ref`).
- **Marketplace:** قبل از نوشتن Action خودتان، بگردید شاید آماده باشد.
- **OIDC:** برای دسترسی به ابر (AWS/Azure) بدون ذخیرهٔ رمز بلندمدت از توکن موقت استفاده کنید.

### خلاصه فصل

GitHub Actions در سه لایه است: Event (محرک) → Workflow (فایل YAML) → Job/Step روی Runner. Actionها واحدهای بازاستفاده‌اند (مثل checkout). مزیت اصلی: یکپارچگی کامل با GitHub و بدون نصب سرور. کلید: همیشه `actions/checkout@v4`، نسخه ثابت بدهید، و از concurrency برای جلوگیری از تداخل استفاده کنید.

---

## فصل ۲۷: نوشتن Workflow (YAML، Matrix، Artifacts، Caching)

### مقدمه

زبان GitHub Actions همان YAML است که در فصل ۱۷ دیدیم، اما با ویژگی‌های خاص خودش: **Matrix** برای تست روی چند محیط، **Artifacts** برای انتقال خروجی، و **Cache** برای سرعت. در این فصل یک Workflow واقعی و کامل می‌سازیم.

### مفاهیم اساسی

**ساختار پایه Workflow:**

```yaml
name: <نام>
on: <رویداد>
jobs:
  <job-id>:
    runs-on: <runner>
    steps:
      - uses: <action>
      - run: <دستور>
```

**Matrix Strategy:** اجرای job روی ترکیب‌های مختلف (OS، نسخه زبان):

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    node: [18, 20, 22]
```

**Artifacts:** فایل‌های خروجی را بین jobها یا برای دانلود آپلود می‌کنید:

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: dist
    path: dist/
```

**Cache:** دانلودهای تکراری را کش می‌کنید:

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: npm-${{ hashFiles('package-lock.json') }}
```

**Dependencies (`needs`):** jobها را به هم وابسته می‌کنید.

### مثال‌های عملی

**۱. Workflow کامل با Matrix، Test، Build، Artifact:**

```yaml
name: CI Pipeline
on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest]
        node: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
          cache: npm
      - run: npm ci
      - run: npm test

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/
      - run: ./deploy.sh
```

**۲. کش هوشمند با کلید محتوا:**

```yaml
- uses: actions/cache@v4
  with:
    path: |
      ~/.cache/pip
      venv/
    key: py-${{ hashFiles('requirements.txt') }}
    restore-keys: py-
```

**۳. استفاده از Action آماده برای داکر:**

```yaml
- name: Build and push
  uses: docker/build-push-action@v6
  with:
    context: .
    push: true
    tags: user/app:${{ github.sha }}
```

**۴. اجرای دستور روی چند خط:**

```yaml
- name: Run migrations
  run: |
    python manage.py migrate
    python manage.py collectstatic --noinput
```

### مقایسه و دیدگاه‌های مختلف

| ویژگی | GitHub Actions | GitLab CI |
|---|---|---|
| سینتکس | YAML | YAML |
| Matrix | `strategy.matrix` | `parallel: matrix` |
| Artifacts | upload/download-action | `artifacts:` |
| Cache | actions/cache | `cache:` داخلی |
| Secrets | `secrets.*` | `variables:` |

**دیدگاه:** سینتکس مشابه است؛ مهاجرت بین آن‌ها آسان است. تفاوت در اکوسیستم و یکپارچه‌سازی است.

**Fail-fast:** در Matrix، با `fail-fast: false` اجازه دهید بقیه ادامه دهند حتی اگر یکی شکست.

### اشتباهات رایج

- **Matrix بی‌حد:** ۳ OS × ۵ نسخه × ۳ ابزار = ۴۵ اجرا (هزینه/زمان). محدود کنید.
- **فراموش upload قبل از download:** job بعدی artifact را نمی‌بیند.
- **کش بدون کلید:** کش فاسد باعث باگ مرموز می‌شود.
- **`if` اشتباه:** اجرای deploy روی PR به جای main.
- **نادیده گرفتن `timeout-minutes`:** job گیر کرده منابع را هدر می‌دهد.

### نکات مهم و ترفندها

- **`timeout-minutes`:** همیشه بگذارید (مثلاً `timeout-minutes: 15`).
- **Reusable setup:** مراحل تکراری را به composite action ببرید (فصل ۲۹).
- **Paths filtering:** فقط وقتی فایل خاصی تغییر کرد اجرا شود:
  ```yaml
  on:
    push:
      paths: ['src/**', 'package.json']
  ```
- **Annotations:** خروجی تست را به صورت annotation در PR نشان دهید (با `::error::` syntax).
- **Large runners:** برای buildهای سنگین، GitHub Large Runners (CPU بالا) استفاده کنید.

### خلاصه فصل

Workflow در `.github/workflows/` با YAML نوشته می‌شود. Matrix برای تست روی چند محیط، Artifacts برای انتقال خروجی بین jobها، و Cache برای سرعت (کلید = hash). کلید: `actions/checkout` اول، `needs` برای وابستگی، `timeout-minutes`، و محدود کردن Matrix. فایل‌های حساس هرگز در لاگ.

---

## فصل ۲۸: Secrets، Environments و Approvals

### مقدمه

استقرار به تولید نیاز به **رمزها (Secrets)** و **تأیید انسانی (Approval)** دارد. GitHub Actions سیستم Secrets و Environments را دارد که امنیت و کنترل ریسک را فراهم می‌کند. در این فصل مدیریت رمز، محیط‌ها (Environments)، و دروازه‌های تأیید (Approval Gates) را یاد می‌گیرید.

### مفاهیم اساسی

**Secrets:**
متغیرهای رمزگذاری‌شده در سطح repo یا environment. در workflow با `${{ secrets.NAME }}` خوانده می‌شوند. GitHub آن‌ها را در لاگ ماسک می‌کند.

انواع:
- **Repository secrets:** برای کل repo.
- **Environment secrets:** مخصوص یک محیط (مثل production).
- **Organization secrets:** برای چند repo در سازمان.

**Environments:**
موجودیتی برای نمایش محیط‌های هدف (staging، production). ویژگی‌ها:
- **Protection rules:** نیاز به Approver مشخص قبل از اجرا.
- **Environment secrets:** رمزهای خاص آن محیط.
- **Deployment history:** تاریخچهٔ استقرارها.

**Approval Gate (دروازهٔ تأیید):**
وقتی Job روی environment با `required_reviewers` تنظیم شود، اجرا متوقف می‌شود تا فرد تأییدکننده approve کند.

**OIDC (OpenID Connect):**
به جای ذخیرهٔ کلید دائمی ابر، GitHub توکن موقت صادر می‌کند — بسیار امن‌تر.

### مثال‌های عملی

**۱. استفاده از Secret:**

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying with token"
        env:
          TOKEN: ${{ secrets.CLOUD_TOKEN }}
```

**۲. تعریف Environment با تأیید:**

```yaml
jobs:
  deploy-prod:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://myapp.example.com
    steps:
      - run: ./deploy.sh prod
```

در UI: `Settings → Environments → production → Required reviewers: @team-lead`

**۳. OIDC برای AWS (بدون کلید ثابت):**

```yaml
permissions:
  id-token: write
  contents: read
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123:role/github-oidc
          aws-region: us-east-1
      - run: aws s3 sync dist/ s3://my-bucket
```

**۴. استفاده از Dependabot secrets:**

```
Settings → Secrets → Dependabot → اضافه کردن توکن رجیستری خصوصی
```

### مقایسه و دیدگاه‌های مختلف

| مکانیسم | امنیت | کاربرد |
|---|---|---|
| Secret ثابت | متوسط (خطر نشت) | ساده |
| OIDC | بالا (موقت) | ابر (AWS/GCP/Azure) |
| Environment + Approver | بالا (کنترل انسانی) | تولید |
| Organization secret | متوسط | به‌اشتراک‌گذاری بین repo |

**دیدگاه DevOps:** OIDC + Environment Approvals استاندارد طلایی برای استقرار امن به ابر است. Secrets ثابت فقط برای موارد ساده.

**Principle of Least Privilege:** IAM Role برای OIDC فقط دسترسی لازم (مثلاً فقط S3) داشته باشد.

### اشتباهات رایج

- **پرینت کردن Secret:** `echo ${{ secrets.TOKEN }}` — حتی با ماسک، خطرناک است.
- **Secret در forked PR:** PR از fork نباید به secrets دسترسی داشته باشد (GitHub پیش‌فرض مسدود می‌کند، اما مراقب باشید).
- **بدون Approver روی production:** استقرار خودکار بدون تأیید = ریسک.
- **Environment secret تکراری:** به جای repo secret، از environment secret استفاده کنید تا ایزوله باشد.
- **کلید دائمی ابر:** به جای OIDC، کلید طولانی می‌دهید (نشت = فاجعه).

### نکات مهم و ترفندها

- **`permissions:` صریح:** فقط دسترسی لازم را بدهید (مثلاً `contents: read`).
- **Secret scanning:** GitHub Secrets را در کد اسکن می‌کند (Push Protection).
- **Environment URL:** پس از deploy، لینک محیط در تاریخچه ظاهر می‌شود.
- **Encrypted secrets limit:** حجم هر secret محدود است (۶۴KB)؛ فایل‌های بزرگ را Split کنید.
- **Rotation:** رمزها را دوره‌ای عوض کنید؛ با OIDC نیاز کمتر.
- **`GITHUB_TOKEN`:** توکن خودکار هر اجرا (موقت) — برای commit/push درون workflow استفاده شود.

### خلاصه فصل

Secrets متغیرهای رمزگذاری‌شده (`secrets.NAME`) هستند. Environments محیط‌های هدف با Protection Rules و Approver اند — برای تولید ضروری. OIDC توکن موقت برای ابر می‌دهد (به جای کلید ثابت، بسیار امن‌تر). کلید: هرگز Secret را پرینت نکنید، Approver روی production داشته باشید، و `permissions` را حداقلی بدهید.

---

## فصل ۲۹: Reusable Workflows و Composite Actions

### مقدمه

وقتی چندین مخزن یا چند workflow دارید، کپی‌کردن YAML تکراری اشتباه است. GitHub Actions دو ابزار برای **بازاستفاده** دارد: **Reusable Workflows** (کل خط لوله) و **Composite Actions** (مراحل درون یک step). این فصل نشان می‌دهد چطور DRY (Don't Repeat Yourself) را در CI رعایت کنید.

### مفاهیم اساسی

**Reusable Workflow:**
یک workflow کامل که توسط workflow دیگر «فراخوانی» می‌شود. مثل یک تابع در برنامه‌نویسی.

```yaml
# در فراخوان‌کننده
jobs:
  call-build:
    uses: org/repo/.github/workflows/build.yml@main
```

**Composite Action:**
چندین step را در یک Action ترکیب می‌کند تا در چند workflow reused شود.

```yaml
# action.yml
runs:
  using: composite
  steps:
    - run: npm ci
      shell: bash
    - run: npm test
      shell: bash
```

**تفاوت:**
- **Reusable Workflow:** کل job/workflow را بازاستفاده می‌کند؛ ورودی/خروجی (inputs/outputs) دارد.
- **Composite Action:** فقط steps را در یک step واحد جمع می‌کند؛ برای کپسوله‌سازی منطق کوچک.

**مزایا:** کاهش تکرار، یکپارچگی استاندارد، نگهداری متمرکز.

### مثال‌های عملی

**۱. Reusable Workflow (تعریف):**

```yaml
# .github/workflows/reusable-build.yml
name: Reusable Build
on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string
    secrets:
      TOKEN:
        required: true
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci && npm run build
```

**۲. فراخوانی آن:**

```yaml
# .github/workflows/ci.yml
jobs:
  call-build:
    uses: ./.github/workflows/reusable-build.yml
    with:
      node-version: '20'
    secrets:
      TOKEN: ${{ secrets.CLOUD_TOKEN }}
```

**۳. Composite Action:**

```yaml
# .github/actions/setup-node-app/action.yml
name: Setup Node App
runs:
  using: composite
  steps:
    - uses: actions/setup-node@v4
      with:
        node-version: 20
        cache: npm
    - run: npm ci
      shell: bash
```

استفاده:
```yaml
steps:
  - uses: actions/checkout@v4
  - uses: ./.github/actions/setup-node-app
```

**۴. Repository Dispatch (فراخوانی cross-repo):**

```yaml
- name: Trigger deploy repo
  uses: peter-evans/repository-dispatch@v3
  with:
    token: ${{ secrets.PAT }}
    repository: org/deploy-repo
    event-type: deploy
```

### مقایسه و دیدگاه‌های مختلف

| ابزار | کاربرد | ورودی/خروجی |
|---|---|---|
| Reusable Workflow | کل pipeline چندباره | `workflow_call` inputs/outputs |
| Composite Action | مراحل تکراری در step | `inputs` ساده |
| 3rd-party Action | قابلیت آماده | متنوع |

**دیدگاه:** اگر کل job تکرار می‌شود → Reusable Workflow. اگر فقط ۳-۴ step → Composite Action. اگر منطق در سازمان گسترده است → Reusable در repo مرکزی.

**Monorepo:** در Monorepo (فصل ۱۵)، Reusable Workflow با `if: contains()` روی مسیر تغییر یافته اجرا شود.

### اشتباهات رایج

- **Reusable بدون ورودی:** سخت‌کد کردن نسخه در هر فراخوانی = تکرار.
- **Circular reuse:** workflow A فراخوانی B، B فراخوانی A → خطا.
- **Composite بدون `shell`:** در composite action هر `run` نیاز به `shell` دارد.
- **نسخه ثابت ندادن به reusable:** `uses: org/repo/.github/workflows/x.yml@main` (به جای SHA ثابت ریسک دارد؛ ترجیحاً تگ).
- **Secret در ورودی:** secrets را در `inputs` ندهید؛ در بخش `secrets` بدهید.

### نکات مهم و ترفندها

- **SHA pinning:** برای امنیت، reusable را با commit SHA ثابت فراخوانی کنید.
- **Outputs:** Reusable workflow می‌تواند خروجی بدهد:
  ```yaml
  outputs:
    artifact-url: ${{ steps.build.outputs.url }}
  ```
- **Local actions:** با `uses: ./.github/actions/my-action` اکشن محلی فراخوانی کنید.
- **`workflow_dispatch`:** اجازه اجرای دستی reusable از UI.
- **Matrix در reusable:** ورودی را در Matrix پاس بدهید.

### خلاصه فصل

برای جلوگیری از تکرار، از Reusable Workflow (کل pipeline، با inputs/outputs) و Composite Action (مراحل در یک step) استفاده کنید. Reusable برای jobهای کامل، Composite برای منطق کوچک. کلید: ورودی‌ها را صریح بدهید، secrets را در بخش secrets (نه inputs)، و نسخه را با SHA/tag ثابت کنید. اصل DRY در CI.

---

## فصل ۳۰: Self-hosted Runners و بهترین روش‌ها

### مقدمه

گاهی Runnerهای ابری GitHub کافی نیستند — شاید نیاز به سخت‌افزار خاص، شبکهٔ داخلی، یا کنترل کامل دارید. **Self-hosted Runners** اجازه می‌دهد ماشین خودتان Jobها را اجرا کند. اما با قدرت، مسئولیت امنیتی می‌آید. در این فصل نصب، مدیریت و بهترین روش‌های Runner خودمیزبان را بررسی می‌کنیم.

### مفاهیم اساسی

**Self-hosted Runner چیست؟**
یک فرآیند (`runner`) روی ماشین شما که به GitHub متصل می‌شود و منتظر Job می‌ماند. وقتی Job فرستاده شود، آن را اجرا می‌کند.

**چرا self-hosted؟**
- دسترسی به منابع داخلی (پایگاه دادهٔ company).
- سخت‌افزار خاص (GPU، ARM).
- کنترل کامل محیط (ابزار از پیش نصب).
- صرفه‌جویی مالی در اجراهای سنگین مداوم.

**انواع:**
- **Repository runner:** فقط برای یک repo.
- **Organization runner:** برای چند repo.
- **Ephemeral runner:** پس از هر Job خود را نابود می‌کند (ایزولاسیون بالا).

**ریسک امنیتی:** کد کاربر روی ماشین شما اجرا می‌شود. هرگز runner را با دسترسی ادمین یا روی ماشین حساس اجرا نکنید.

### مثال‌های عملی

**۱. نصب Runner روی لینوکس:**

```bash
# دانلود
mkdir actions-runner && cd actions-runner
curl -o runner.tar.gz -L https://github.com/actions/runner/releases/download/v2.3xx/runner-linux-x64.tar.gz
tar xzf runner.tar.gz

# پیکربندی (توکن از Settings → Actions → Runners)
./config.sh --url https://github.com/org/repo --token <TOKEN>

# اجرا (foreground)
./run.sh

# یا به عنوان سرویس
sudo ./svc.sh install
sudo ./svc.sh start
```

**۲. استفاده در Workflow:**

```yaml
jobs:
  build:
    runs-on: self-hosted
    # یا با label
    # runs-on: [self-hosted, linux, gpu]
```

**۳. Runner Ephemeral (Docker):**

```bash
./config.sh --url ... --token ... --ephemeral
# پس از هر job، runner خارج می‌شود → بازسازی توسط orchestrator
```

**۴. Runner در Kubernetes (Actions Runner Controller):**

```bash
helm install actions-runner-controller \
  --namespace actions-runner-system \
  actions-runner-controller/actions-runner-controller
```

### مقایسه و دیدگاه‌های مختلف

| معیار | GitHub-hosted | Self-hosted |
|---|---|---|
| راه‌اندازی | صفر | نیاز به نصب/نگهداری |
| ایزولاسیون | بالا (موقت) | بستگی به تنظیم |
| دسترسی داخلی | نه | بله |
| هزینه | دقیقه‌ای | سخت‌افزار شما |
| امنیت | مدیریت‌شده | مسئولیت شما |

**دیدگاه:** ترکیب کنید — Jobهای عمومی روی GitHub-hosted، Jobهای نیازمند داخلی روی self-hosted. Ephemeral runner با ARC در K8s بهترین ایزولاسیون را می‌دهد.

**Hybrid approach:** اکثر تیم‌ها از GitHub-hosted استفاده می‌کنند مگر موارد خاص.

### اشتباهات رایج

- **اجرای runner با sudo/root:** کد مخرب کنترل کل ماشین را می‌گیرد.
- **Runner روی ماشین مشترک حساس:** نشت داده بین Jobها.
- **فراموش کردن به‌روزرسانی runner:** نسخه قدیمی آسیب‌پذیر است.
- **بدون ephemeral در محیط ناامن:** state بین Jobها باقی می‌ماند.
- **Labels اشتباه:** Job به runner غلط هدایت می‌شود.

### نکات مهم و ترفندها

- **Ephemeral همیشه:** `--ephemeral` برای ایزولاسیون کامل (ARC توصیه می‌کند).
- **Least privilege user:** runner را با کاربر محدود اجرا کنید.
- **Auto-update:** runner خودش را به‌روزرسانی می‌کند؛ اجازه دهید.
- **Scaling با ARC:** در K8s، تعداد runner را بر اساس صف Job تنظیم کنید (Horizontal Runner Autoscaling).
- **Runner groups:** دسترسی repoها به گروه‌های runner را محدود کنید.
- **Monitoring:** لاگ runner را مانیتور کنید تا خرابی زود دیده شود.

### خلاصه فصل

Self-hosted Runner ماشین شما را اجراکنندهٔ Job می‌کند — برای دسترسی داخلی، سخت‌افزار خاص، یا صرفه‌جویی. ریسک امنیتی بزرگ: کد کاربر روی ماشین شما اجرا می‌شود. کلید: همیشه Ephemeral (ایزولاسیون)، اجرا با کاربر کم‌دسترسی (نه root)، به‌روزرسانی خودکار، و استفاده از ARC در K8s برای مقیاس‌پذیری. ترکیب با GitHub-hosted بهترین رویکرد.

---

## منابع و مراجع

- GitHub Actions Documentation: https://docs.github.com/actions
- Workflow Syntax: https://docs.github.com/actions/using-workflows/workflow-syntax-for-github-actions
- Reusable Workflows: https://docs.github.com/actions/using-workflows/reusing-workflows
- Encrypted Secrets: https://docs.github.com/actions/security-guides/encrypted-secrets
- OIDC with Cloud: https://docs.github.com/actions/deployment/security-hardening-your-deployments/configuring-openid-connect
- Self-hosted Runners: https://docs.github.com/actions/hosting-your-own-runners
- Actions Runner Controller (ARC): https://github.com/actions/actions-runner-controller
- GitHub Marketplace (Actions): https://github.com/marketplace?type=actions

**بعدی:** [فایل ۰۷ — GitLab CI: یکپارچه و کامل](07-GitLab-CI.md)
