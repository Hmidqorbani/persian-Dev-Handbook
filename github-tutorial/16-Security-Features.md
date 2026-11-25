# بخش ۱۶: قابلیت‌های امنیتی GitHub

**مدت یادگیری**: ۵ تا ۶ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: بخش ۰۹ (Rulesets)، بخش ۱۲ (Actions Security)

---

## فهرست محتوا

- [فصل ۷۱: Dependabot — به‌روزرسانی و هشدار وابستگی](#فصل-۷۱)
- [فصل ۷۲: Code Scanning با CodeQL](#فصل-۷۲)
- [فصل ۷۳: Secret Scanning و Push Protection](#فصل-۷۳)
- [فصل ۷۴: Security Advisories و هماهنگی افشا](#فصل-۷۴)
- [فصل ۷۵: SBOM، Artifact Attestations و زنجیره‌ی تأمین](#فصل-۷۵)

---

## فصل ۷۱: Dependabot — به‌روزرسانی و هشدار وابستگی

### مقدمه

۸۰٪ کد یک اپلیکیشن مدرن از وابستگی‌ها می‌آید. و وابستگی‌ها آسیب‌پذیری پیدا می‌کنند — مرتب. Dependabot سه کار می‌کند: هشدار می‌دهد، PR برای رفع می‌زند، و نسخه‌ها را به‌روز نگه می‌دارد.

**چرا مهم است؟** یک وابستگی آسیب‌پذیر می‌تواند کل اپ را به خطر بیندازد. و اگر به‌روزرسانی دستی باشد، هرگز انجام نمی‌شود.

**کاربرد عملی:** Dependabot ارزان‌ترین و مؤثرترین قدم امنیتی است. ۱۰ دقیقه تنظیم، سال‌ها سود.

### مفاهیم اساسی

#### سه قابلیت

| قابلیت | چه می‌کند | نیاز به فایل |
|---|---|---|
| **Dependabot alerts** | هشدار آسیب‌پذیری | ❌ (خودکار) |
| **Dependabot security updates** | PR برای رفع آسیب‌پذیری | ❌ (تنظیم در UI) |
| **Dependabot version updates** | PR برای به‌روزرسانی معمولی | ✅ `dependabot.yml` |

#### اکوسیستم‌های پشتیبانی‌شده

```
npm، pip، go modules، cargo، maven، gradle، nuget، composer،
bundler، docker، github-actions، terraform، devcontainers،
helm، pub، mix، swift، bun، uv
```

#### alerts از کجا می‌آید

GitHub Advisory Database — یک پایگاه داده از CVEها و advisoryها که GitHub و جامعه به‌روز می‌کنند. همچنین از طریق dependency graph، وابستگی‌های غیرمستقیم را هم می‌بیند.

### مثال‌های عملی

#### مثال ۱: `dependabot.yml` کامل

`.github/dependabot.yml`:

```yaml
version: 2

updates:
  # npm
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "06:00"
      timezone: "Asia/Tehran"
    open-pull-requests-limit: 10
    versioning-strategy: increase
    commit-message:
      prefix: "chore(deps)"
      prefix-development: "chore(deps-dev)"
      include: "scope"
    labels:
      - "dependencies"
      - "automerge"
    reviewers:
      - "myorg/core-team"
    groups:
      # گروه‌بندی: یک PR برای همه‌ی minor/patch
      minor-and-patch:
        patterns: ["*"]
        update-types: ["minor", "patch"]
      # گروه جدا برای React
      react:
        patterns: ["react*", "@types/react*"]
    ignore:
      - dependency-name: "aws-sdk"
        update-types: ["version-update:semver-major"]
      - dependency-name: "eslint"
        versions: ["9.x"]

  # GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
    commit-message:
      prefix: "ci"
    groups:
      actions:
        patterns: ["*"]

  # Docker
  - package-ecosystem: "docker"
    directory: "/docker"
    schedule:
      interval: "weekly"
    commit-message:
      prefix: "build(docker)"

  # Terraform
  - package-ecosystem: "terraform"
    directory: "/infra"
    schedule:
      interval: "monthly"

  # pip
  - package-ecosystem: "pip"
    directory: "/backend"
    schedule:
      interval: "weekly"
```

#### مثال ۲: گروه‌بندی — کاهش نویز

**مشکل:** بدون گروه‌بندی، ۲۰ PR در هفته می‌آید و تیم نادیده می‌گیرد.

**راه‌حل:** `groups`:

```yaml
groups:
  minor-and-patch:
    patterns: ["*"]
    update-types: ["minor", "patch"]
```

حالا همه‌ی minor/patch در **یک** PR می‌آیند. majorها جدا می‌مانند (چون ریسک دارند).

#### مثال ۳: automerge برای patch

```yaml
# .github/workflows/dependabot-automerge.yml
name: Dependabot auto-merge

on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  automerge:
    if: github.actor == 'dependabot[bot]'
    runs-on: ubuntu-24.04
    steps:
      - name: دریافت متادیتا
        id: meta
        uses: dependabot/fetch-metadata@<sha>
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}

      - name: automerge برای patch
        if: steps.meta.outputs.update-type == 'version-update:semver-patch'
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**قاعده:** automerge فقط برای patch (و شاید minor)، هرگز برای major.

#### مثال ۴: فعال کردن alerts

```bash
# alerts
gh api -X PUT repos/OWNER/REPO/vulnerability-alerts

# security updates
gh api -X PUT repos/OWNER/REPO/automated-security-fixes

# بررسی
gh api repos/OWNER/REPO/vulnerability-alerts -i | head -1
```

یا UI: **Settings → Code security → Dependabot**.

#### مثال ۵: دیدن alerts

```bash
gh api repos/OWNER/REPO/dependabot/alerts \
  --jq '.[] | "\(.security_advisory.severity) \(.dependency.package.name) \(.security_advisory.summary)"'

# فقط critical
gh api repos/OWNER/REPO/dependabot/alerts?state=open \
  --jq '.[] | select(.security_advisory.severity=="critical") | .dependency.package.name'
```

#### مثال ۶: رفع دستی alert

```bash
# ۱) ببین کدام وابستگی
gh api repos/OWNER/REPO/dependabot/alerts --jq '.[0].dependency.package.name'

# ۲) به‌روزرسانی
npm update <package>

# ۳) یا اگر نسخه‌ی امن موجود نیست
npm audit fix --force    # احتیاط: ممکن است breaking باشد
```

#### مثال ۷: ignore کردن alert (با دلیل)

```bash
gh api -X PATCH repos/OWNER/REPO/dependabot/alerts/<number> \
  -f state=dismissed \
  -f dismissed_reason=not_used \
  -f dismissed_comment="این وابستگی فقط در dev استفاده می‌شود"
```

دلایل: `fix_started`، `inaccurate`، `no_bandwidth`، `not_used`، `tolerable_risk`.

**هشدار:** dismiss بی‌دلیل، بدهی امنیتی می‌سازد. هر dismiss را در Issue ثبت کن.

#### مثال ۸: محدودیت‌های Dependabot

| محدودیت | توضیح |
|---|---|
| فقط manifest‌های شناخته‌شده | lock file‌های عجیب پشتیبانی نمی‌شوند |
| فقط dependency graph | کد vendored را نمی‌بیند |
| بدون تست خودکار | PR باید تست شود |
| نسخه‌ی major | ممکن است خودکار نشود |

#### مثال ۹: ترکیب با Ruleset

Dependabot PR هم باید review و CI داشته باشد — ولی می‌توانی برایش مسیر سریع‌تر بگذاری:

```
Ruleset bypass: 
  - App: dependabot (فقط برای patch)
```

**هشدار:** bypass به Dependabot، ریسک دارد. اگر Dependabot هک شود (که نادر است ولی ممکن)، می‌تواند کد merge کند. بهتر: automerge با شرط `update-type == patch`.

#### مثال ۱۰: بررسی هفتگی

```bash
# گزارش هفتگی
gh api repos/OWNER/REPO/dependabot/alerts?state=open \
  --jq 'group_by(.security_advisory.severity) | map({severity: .[0].security_advisory.severity, count: length})'
```

### مقایسه و الگوهای مختلف

#### alerts در برابر security updates در برابر version updates

| | alerts | security updates | version updates |
|---|---|---|---|
| هشدار | ✅ | ✅ | ❌ |
| PR می‌زند | ❌ | ✅ (فقط آسیب‌پذیر) | ✅ (همه) |
| نیاز به فایل | ❌ | ❌ | ✅ |
| مناسب | همه | همه | تیم بالغ |

#### automerge: patch در برابر minor در برابر major

| نوع | automerge | چرا |
|---|---|---|
| patch | ✅ | ریسک کم |
| minor | ⚠️ | معمولاً سازگار، ولی نه همیشه |
| major | ❌ | breaking |

#### گروه‌بندی در برابر PR جدا

| | گروه | جدا |
|---|---|---|
| نویز | ✅ کم | زیاد |
| ریسک | متوسط (چند تغییر با هم) | ✅ کم |
| rollback | سخت | ✅ آسان |

**توصیه:** گروه برای minor/patch، جدا برای major.

### اشکال شایع

#### ۱. alerts خاموش

```
Settings → Code security → Dependabot alerts: Disabled
```

اولین کاری که در هر مخزن جدید باید بکنی: روشن کن.

#### ۲. ۵۰ PR باز Dependabot

`open-pull-requests-limit` را تنظیم کن (۱۰-۱۵). و گروه‌بندی.

#### ۳. automerge برای major

```yaml
✗ if: github.actor == 'dependabot[bot]'
```

همه‌چیز automerge می‌شود، حتی major. شرط `update-type` بگذار.

#### ۴. ignore بدون مستندسازی

```
✗ ignore: aws-sdk
```

شش ماه بعد، هیچ‌کس نمی‌داند چرا. در `dependabot.yml` کامنت بگذار یا Issue بساز.

#### ۵. Dependabot PR بدون CI

اگر CI ندارد، ممکن است چیزی بشکند. `dependabot.yml` را طوری تنظیم کن که PRها CI را trigger کنند (پیش‌فرض: trigger می‌شوند).

#### ۶. انتظار پوشش کامل

Dependabot فقط وابستگی‌های شناخته‌شده را می‌بیند. کد vendored، binary، یا dependency داخلی را نمی‌بیند.

#### ۷. نادیده گرفتن transitive dependencies

Dependabot وابستگی‌های غیرمستقیم را هم هشدار می‌دهد، ولی رفعشان ممکن است نیاز به به‌روزرسانی والد داشته باشد.

#### ۸. `directory` اشتباه

```yaml
✗ directory: "."
✓ directory: "/"        # برای ریشه
✓ directory: "/backend"  # برای زیرپوشه
```

### نکات مهم (Tips & Tricks)

- **alerts را در هر مخزن روشن کن** — ۱۰ دقیقه، سال‌ها سود.
- **گروه‌بندی** برای کاهش نویز.
- **automerge فقط patch.**
- **`open-pull-requests-limit: 10`.**
- **Dependabot برای `github-actions`** — SHAها به‌روز می‌مانند (فصل ۵۵).
- **هر dismiss را مستند کن.**

### خلاصه فصل

- Dependabot سه قابلیت: alerts، security updates، version updates.
- `dependabot.yml` برای version updates و تنظیمات.
- **گروه‌بندی** نویز را کم می‌کند.
- **automerge فقط patch.**
- Dependabot برای `github-actions` هم کار می‌کند.
- هر dismiss را در Issue ثبت کن.

**سؤالات مروری:**
1. تفاوت سه قابلیت Dependabot چیست؟
2. چرا automerge برای major خطرناک است؟
3. چرا Dependabot برای `github-actions` مهم است؟

---

## فصل ۷۲: Code Scanning با CodeQL

### مقدمه

Code Scanning یک تحلیل ایستا (static analysis) است که کد را **بدون اجرا** بررسی می‌کند و آسیب‌پذیری‌ها را پیدا می‌کند: SQL injection، XSS، path traversal، deserialization ناامن، و صدها الگوی دیگر.

**چرا مهم است؟** بعضی آسیب‌پذیری‌ها در review انسانی دیده نمی‌شوند (data flow پیچیده). CodeQL مسیر داده را از منبع تا sink دنبال می‌کند.

**کاربرد عملی:** در مخزن عمومی رایگان است. در مخزن خصوصی نیاز به GitHub Advanced Security (یا Code Security) دارد.

### مفاهیم اساسی

#### دو حالت راه‌اندازی

| | Default setup | Advanced setup |
|---|---|---|
| تنظیم | چند کلیک | فایل workflow |
| زبان | خودکار تشخیص | صریح |
| سفارشی‌سازی | محدود | ✅ کامل |
| build سفارشی | ❌ | ✅ |
| مناسب | ۹۰٪ پروژه‌ها | پروژه‌های پیچیده |

#### زبان‌های پشتیبانی‌شده

```
C/C++، C#، Go، Java/Kotlin، JavaScript/TypeScript، Python، Ruby، Swift
```

#### Query suiteها

| Suite | تعداد query | زمان |
|---|---|---|
| `default` | متوسط | متوسط |
| `security-extended` | بیشتر | بیشتر |
| `security-and-quality` | بیشترین | بیشترین |

#### نتیجه

هر alert شامل:

- **Rule** (مثل `js/sql-injection`)
- **Severity** (critical، high، medium، low)
- **Location** (فایل + خط)
- **Data flow** (از کجا تا کجا)
- **Recommendation**

### مثال‌های عملی

#### مثال ۱: Default setup

```
Settings → Code security → Code scanning → Set up
  ○ Default
  ● Advanced
```

Default:

1. **Set up** → **Default**
2. زبان‌ها خودکار تشخیص داده می‌شوند
3. اولین اجرا شروع می‌شود (چند دقیقه تا چند ساعت)

#### مثال ۲: Advanced setup — JavaScript

`.github/workflows/codeql.yml`:

```yaml
name: CodeQL

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: "0 4 * * 1"    # هفتگی
  merge_group:

permissions:
  contents: read
  security-events: write
  actions: read

jobs:
  analyze:
    name: Analyze (${{ matrix.language }})
    runs-on: ubuntu-24.04
    timeout-minutes: 30
    strategy:
      fail-fast: false
      matrix:
        language: ["javascript-typescript"]
    steps:
      - uses: actions/checkout@<sha>

      - uses: github/codeql-action/init@<sha>
        with:
          languages: ${{ matrix.language }}
          queries: security-extended
          config-file: ./.github/codeql/codeql-config.yml

      - uses: github/codeql-action/autobuild@<sha>
        if: matrix.language != 'javascript-typescript'

      - uses: github/codeql-action/analyze@<sha>
        with:
          category: "/language:${{ matrix.language }}"
```

**نکته:** JavaScript/TypeScript نیاز به build ندارد (extracted مستقیم). زبان‌های کامپایل‌شده `autobuild` یا build دستی می‌خواهند.

#### مثال ۳: Advanced — Java با build سفارشی

```yaml
      - uses: github/codeql-action/init@<sha>
        with:
          languages: java-kotlin
          queries: security-extended

      - uses: actions/setup-java@<sha>
        with:
          java-version: "21"
          distribution: "temurin"
          cache: maven

      # build سفارشی (به‌جای autobuild)
      - run: mvn -B clean compile -DskipTests

      - uses: github/codeql-action/analyze@<sha>
```

**چرا build سفارشی؟** `autobuild` ممکن است build واقعی پروژه را درست تشخیص ندهد (مثلاً Gradle با profile خاص).

#### مثال ۴: config برای ignore کردن مسیرها

`.github/codeql/codeql-config.yml`:

```yaml
name: "CodeQL config"

queries:
  - uses: security-extended

paths-ignore:
  - "**/node_modules/**"
  - "**/dist/**"
  - "**/*.test.ts"
  - "**/__tests__/**"
  - "**/migrations/**"

paths:
  - "src"

# queryهای اضافی
packs:
  - codeql/javascript-queries:AlertSuppression.ql
```

#### مثال ۵: بررسی نتایج

```bash
# لیست alertها
gh api repos/OWNER/REPO/code-scanning/alerts \
  --jq '.[] | "\(.rule.severity) \(.rule.id) \(.most_recent_instance.location.path):\(.most_recent_instance.location.start_line)"'

# فقط critical/high
gh api repos/OWNER/REPO/code-scanning/alerts?state=open \
  --jq '.[] | select(.rule.severity=="error") | .rule.id'

# جزئیات یک alert
gh api repos/OWNER/REPO/code-scanning/alerts/<number> | jq
```

#### مثال ۶: رفع alert

```javascript
// alert: js/sql-injection
✗ db.query(`SELECT * FROM users WHERE id = ${req.params.id}`);

// رفع:
✓ db.query("SELECT * FROM users WHERE id = $1", [req.params.id]);
```

بعد از push، alert خودکار بسته می‌شود (اگر در همان خط باشد).

#### مثال ۷: dismiss با دلیل

```bash
gh api -X PATCH repos/OWNER/REPO/code-scanning/alerts/<number> \
  -f state=dismissed \
  -f dismissed_reason="false positive" \
  -f dismissed_comment="این ورودی از enum می‌آید، نه کاربر"
```

دلایل: `false positive`، `won't fix`، `used in tests`.

#### مثال ۸: required check

```
Ruleset → Require code scanning results
  Security alerts: Errors
  Security alerts: High or higher
```

حالا PR با alert جدید، merge نمی‌شود.

**نکته:** این در Rulesets موجود است ولی در Branch protection قدیمی نه.

#### مثال ۹: SARIF از ابزار دیگر

CodeQL تنها نیست. هر ابزاری که SARIF تولید می‌کند:

```yaml
      - name: ESLint با SARIF
        run: npx eslint . --format @microsoft/eslint-formatter-sarif --output-file eslint.sarif
        continue-on-error: true

      - uses: github/codeql-action/upload-sarif@<sha>
        with:
          sarif_file: eslint.sarif
          category: eslint
```

ابزارهای دیگر: Semgrep، Snyk، Trivy، Checkov، tfsec، Bandit، gosec.

#### مثال ۱۰: Semgrep به‌عنوان مکمل

```yaml
      - uses: semgrep/semgrep-action@<sha>
        with:
          config: >-
            p/security-audit
            p/owasp-top-ten
            p/typescript
```

Semgrep سریع‌تر است و قواعد سفارشی دارد. CodeQL عمیق‌تر است. **هر دو** ارزش دارند.

### مقایسه و الگوهای مختلف

#### CodeQL در برابر Semgrep در برابر ESLint

| | CodeQL | Semgrep | ESLint |
|---|---|---|---|
| عمق | ✅✅ | ✅ | ❌ |
| سرعت | کند | ✅ سریع | ✅ |
| قواعد سفارشی | ✅ (QL) | ✅ (YAML) | ✅ |
| data flow | ✅ | ✅ | ❌ |
| رایگان در خصوصی | ❌ | ✅ | ✅ |

#### Default در برابر Advanced

| | Default | Advanced |
|---|---|---|
| تنظیم | ۲ کلیک | فایل |
| زبان | خودکار | صریح |
| suite | default | قابل انتخاب |
| build سفارشی | ❌ | ✅ |
| مناسب | اکثر | پیچیده |

#### `security-extended` در برابر `security-and-quality`

| | security-extended | security-and-quality |
|---|---|---|
| امنیت | ✅ | ✅ |
| کیفیت کد | ❌ | ✅ |
| نویز | متوسط | زیاد |
| زمان | متوسط | بیشتر |

**توصیه:** `security-extended` برای PR، `security-and-quality` هفتگی.

### اشکال شایع

#### ۱. CodeQL روی مخزن خصوصی بدون لایسنس

```
Code scanning is not enabled for this repository
```

نیاز به GitHub Code Security (یا Advanced Security). مخزن عمومی رایگان است.

#### ۲. `autobuild` که build نمی‌کند

```
We were unable to automatically build your code
```

build سفارشی بنویس (مثال ۳).

#### ۳. alertهای زیاد در روز اول

اگر پروژه‌ی قدیمی است، ۲۰۰ alert می‌آید. راه‌حل:

1. baseline بگیر (alertهای موجود را dismiss با `won't fix` یا در baseline)
2. فقط alertهای **جدید** را required کن

```yaml
# Ruleset
Require code scanning: Errors (فقط جدیدها)
```

#### ۴. alert تکراری

هر push، alert جدید برای همان خط. GitHub خودکار dedupe می‌کند (بر اساس rule + location).

#### ۵. نادیده گرفتن data flow

alert می‌گوید SQL injection در خط ۴۲. ولی منبع داده در خط ۱۰ است. **مسیر را کامل بخوان.**

#### ۶. dismiss بدون بررسی

```
✗ gh api ... -f dismissed_reason="false positive"
```

اگر واقعاً false positive نیست، آسیب‌پذیری باقی می‌ماند. هر dismiss را با دلیل دقیق ثبت کن.

#### ۷. CodeQL کند است

برای پروژه‌ی بزرگ، ۳۰-۶۰ دقیقه. راه‌حل: فقط روی `main` و هفتگی، نه هر PR. یا `paths-ignore`.

#### ۸. زبان اشتباه

```yaml
✗ language: "javascript"
✓ language: "javascript-typescript"
```

نام زبان‌ها تغییر کرده. `javascript-typescript` جدید است.

### نکات مهم (Tips & Tricks)

- **Default setup** برای شروع — ساده و کافی.
- **`security-extended`** تعادل خوبی است.
- **baseline** برای پروژه‌ی قدیمی — فقط alertهای جدید required.
- **`paths-ignore`** برای تست‌ها و node_modules.
- **SARIF از ابزار دیگر** هم آپلود کن.
- **CodeQL + Semgrep** مکمل‌اند.
- **هر dismiss را مستند کن.**

### خلاصه فصل

- Code Scanning کد را بدون اجرا تحلیل می‌کند.
- Default setup ساده؛ Advanced برای سفارشی‌سازی.
- `security-extended` تعادل خوب.
- **`Require code scanning results`** در Ruleset، merge را مسدود می‌کند.
- SARIF از هر ابزاری قابل آپلود است.
- برای پروژه‌ی قدیمی، baseline بگیر.

**سؤالات مروری:**
1. تفاوت Default و Advanced setup چیست؟
2. چرا CodeQL برای JavaScript به build نیاز ندارد ولی برای Java دارد؟
3. چطور در پروژه‌ی قدیمی با ۲۰۰ alert شروع کنی؟

---

## فصل ۷۳: Secret Scanning و Push Protection

### مقدمه

Secret Scanning مخزن را برای کلیدها و توکن‌ها اسکن می‌کند. **Push Protection** حتی جلوتر می‌رود: جلوی push کلید را **قبل از** رسیدن به مخزن می‌گیرد.

**چرا مهم است؟** یک کلید لو رفته در چند دقیقه می‌تواند کل حساب را به خطر بیندازد. مهاجم‌ها مخزن‌های عمومی را خودکار اسکن می‌کنند — معمولاً در کمتر از ۵ دقیقه.

**کاربرد عملی:** Push Protection تنها قابلیتی است که **جلوگیری** می‌کند، نه فقط **تشخیص**. در مخزن عمومی و خصوصی رایگان است.

### مفاهیم اساسی

#### سه لایه

| لایه | چه می‌کند | کِی |
|---|---|---|
| **Secret scanning** | مخزن را اسکن می‌کند | بعد از push |
| **Push protection** | جلوی push را می‌گیرد | قبل از push |
| **Validity check** | بررسی می‌کند کلید هنوز معتبر است | بعد از شناسایی |

#### چه چیزی شناسایی می‌شود

| نوع | مثال |
|---|---|
| **Partner patterns** | ۱۰۰+ سرویس (AWS، Stripe، Slack، npm، ...) |
| **Custom patterns** | الگوهای خودت (توکن داخلی) |
| **Generic patterns** | الگوهای عمومی (private key، JWT) |

#### انواع alert

| نوع | معنی |
|---|---|
| `user` | در مخزن کاربر |
| `commit` | در تاریخ commit |
| `issue` | در Issue/PR |
| `discussion` | در Discussion |
| `wiki` | در Wiki |

#### Validity

GitHub با سرویس مربوطه چک می‌کند که کلید **هنوز معتبر است**. `active` = فاجعه، `inactive` = کم‌خطر.

### مثال‌های عملی

#### مثال ۱: فعال‌سازی

```bash
# Secret scanning (خودکار در مخزن عمومی، دستی در خصوصی)
gh api -X PATCH repos/OWNER/REPO \
  -f 'security_and_analysis[secret_scanning][status]=enabled'

# Push protection
gh api -X PATCH repos/OWNER/REPO \
  -f 'security_and_analysis[secret_scanning_push_protection][status]=enabled'
```

یا UI: **Settings → Code security**.

#### مثال ۲: دیدن alertها

```bash
gh api repos/OWNER/REPO/secret-scanning/alerts \
  --jq '.[] | "\(.secret_type_display_name) — \(.validity) — \(.secret)"'

# فقط معتبرها (خطرناک)
gh api repos/OWNER/REPO/secret-scanning/alerts?validity=active \
  --jq '.[] | .secret_type_display_name'
```

#### مثال ۳: Push Protection در عمل

```bash
git push
# remote: error: GH013: Repository rule violations found for refs/heads/main.
# remote:
# remote: —— Secret detected ——
# remote:   Location: config/aws.env:3
# remote:   Secret: AWS Access Key
# remote:
# remote: Push cannot contain secrets.
# remote:
# remote: —— How to fix ——
# remote:   - Remove the secret and commit again
# remote:   - Or visit the URL to allow the secret
```

#### مثال ۴: bypass کردن (وقتی false positive است)

```
remote:   - https://github.com/OWNER/REPO/security/secret-scanning/unblock-secret/...
```

با کلیک، می‌توانی دلیل بدهی (`used in tests`، `false positive`، `will fix later`) و push کنی. **این bypass در audit log ثبت می‌شود.**

```bash
# با gh (نیاز به permission)
gh api -X POST repos/OWNER/REPO/secret-scanning/push-protection-bypasses \
  -f reason="false positive" \
  -f placeholder_id="..." 
```

#### مثال ۵: Custom pattern

اگر توکن داخلی داری با فرمت `myco_` + ۳۲ کاراکتر:

```
Settings → Code security → Secret scanning → Custom patterns → New pattern

Name: Internal API Token
Pattern: myco_[a-zA-Z0-9]{32}
```

یا با API:

```bash
gh api -X POST repos/OWNER/REPO/secret-scanning/patterns \
  -f name="Internal API Token" \
  -f pattern="myco_[a-zA-Z0-9]{32}"
```

#### مثال ۶: اگر کلید لو رفت — پروتکل

```
۱. فوراً کلید را در سرویس اصلی revoke کن
   (AWS: IAM → Access keys → Deactivate/Delete)
   (npm: Settings → Tokens → Revoke)
   (Stripe: Dashboard → API keys → Roll)

۲. کلید جدید بساز

۳. کلید جدید را در secret manager بگذار (نه در کد)

۴. alert را در GitHub ببند

۵. تاریخ Git را پاک کن (اگر لازم است — بخش بعدی)

۶. لاگ سرویس را بررسی کن: آیا کسی استفاده کرده؟
```

**ترتیب حیاتی است:** اول revoke، بعد پاکسازی. اگر اول پاک کنی، کلید هنوز معتبر است.

#### مثال ۷: پاک کردن از تاریخ Git

```bash
# ۱) نصب git-filter-repo (بهتر از filter-branch)
pip install git-filter-repo

# ۲) پشتیبان
git clone --mirror git@github.com:OWNER/REPO.git repo-backup.git

# ۳) حذف فایل
cd REPO
git filter-repo --path config/aws.env --invert-paths

# ۴) force push
git push --force --all
git push --force --tags

# ۵) به تیم اطلاع بده: همه باید دوباره clone کنند
```

**هشدارها:**

- این تاریخ را کاملاً بازنویسی می‌کند
- همه‌ی forkها و cloneها نسخه‌ی قدیمی دارند
- GitHub ممکن است نسخه‌ی قدیمی را در کش داشته باشد (تا وقتی support پاک کند)
- **فقط revoke کردن کافی و ضروری است** — پاکسازی تاریخ، دفاع لایه‌ی دوم است

#### مثال ۸: جلوگیری از تکرار

```
۱. .gitignore کامل (فصل ۸)
۲. Push Protection روشن
۳. pre-commit hook با gitleaks
۴. secret manager برای همه‌ی کلیدها
۵. آموزش تیم
```

`gitleaks` در pre-commit:

```yaml
# .pre-commit-config.yaml
- repo: https://github.com/gitleaks/gitleaks
  rev: v8.21.2
  hooks:
    - id: gitleaks
```

#### مثال ۹: gitleaks در CI

```yaml
- name: gitleaks
  uses: gitleaks/gitleaks-action@<sha>
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

#### مثال ۱۰: بررسی دوره‌ای

```bash
# اسکن کل تاریخ با gitleaks
gitleaks detect --source . --verbose --redact

# trufflehog
trufflehog git file://. --only-verified
```

### مقایسه و الگوهای مختلف

#### Secret scanning در برابر Push protection

| | scanning | push protection |
|---|---|---|
| کِی | بعد از push | قبل از push |
| جلوگیری | ❌ | ✅ |
| پوشش | همه‌ی تاریخ | فقط push جدید |
| مناسب | کشف | پیشگیری |

**هر دو لازم است.** push protection جلوی جدیدها را می‌گیرد؛ scanning قدیمی‌ها را پیدا می‌کند.

#### Partner patterns در برابر custom

| | Partner | Custom |
|---|---|---|
| تعداد | ۱۰۰+ سرویس | خودت |
| Validity check | ✅ | ❌ |
| تنظیم | صفر | دستی |
| مناسب | سرویس‌های عمومی | توکن داخلی |

#### revoke در برابر پاکسازی تاریخ

| | revoke | پاکسازی |
|---|---|---|
| سرعت | ✅ فوری | کند |
| اثربخشی | ✅ کامل | متوسط |
| ضروری | ✅ | ⚠️ (بسته به شرایط) |

**همیشه revoke. پاکسازی فقط اگر لازم است.**

### اشکال شایع

#### ۱. revoke نکردن

```
✗ alert را بستم، تمام
```

کلید هنوز معتبر است. **اول revoke.**

#### ۲. فکر کردن پاک کردن فایل کافی است

```
✗ git rm .env && git commit
```

فایل در تاریخ هست. `git log -p` نشانش می‌دهد.

#### ۳. force push بدون اطلاع تیم

بعد از rewrite، همه باید دوباره clone کنند. اگر کسی push کند، تاریخ برمی‌گردد.

#### ۴. bypass بی‌دلیل

هر bypass در audit log ثبت می‌شود. اگر مدیر ببیند ۲۰ bypass در ماه، اعتماد از بین می‌رود.

#### ۵. custom pattern خیلی وسیع

```
✗ pattern: [a-zA-Z0-9]{20,}
```

هر رشته‌ی طولانی alert می‌شود. دقیق باش.

#### ۶. secret در Actions log

Push protection فقط push را می‌گیرد، نه چاپ در لاگ. برای آن، `::add-mask::` و مراقبت.

#### ۷. secret در artifact

Push protection artifact را نمی‌بیند. مراقب باش (فصل ۵۶).

#### ۸. فراموش کردن forkها

اگر مخزن fork شده، forkها نسخه‌ی قدیمی را دارند. باید آن‌ها را هم پاک کنی (یا از GitHub بخواهی).

### نکات مهم (Tips & Tricks)

- **Push Protection را در هر مخزن روشن کن** — رایگان و مؤثر.
- **اول revoke، بعد پاکسازی.**
- **`gitleaks` در pre-commit** — لایه‌ی سوم.
- **custom pattern برای توکن داخلی.**
- **validaity check** — بدان کدام کلید هنوز معتبر است.
- **audit log bypassها** را ماهانه چک کن.

### خلاصه فصل

- Secret scanning مخزن را اسکن می‌کند؛ Push protection جلوی push را می‌گیرد.
- هر دو رایگان و ضروری.
- **اگر کلید لو رفت: اول revoke، بعد پاکسازی.**
- `git filter-repo` برای بازنویسی تاریخ.
- Custom pattern برای توکن داخلی.
- `gitleaks` لایه‌ی سوم.

**سؤالات مروری:**
1. تفاوت secret scanning و push protection چیست؟
2. اگر کلید لو رفت، اولین کار چیست؟
3. چرا پاک کردن فایل با `git rm` کافی نیست؟

---

## فصل ۷۴: Security Advisories و هماهنگی افشا

### مقدمه

وقتی آسیب‌پذیری در پروژه‌ی خودت پیدا می‌کنی (یا کسی گزارش می‌دهد)، باید **قبل از** افشای عمومی، fix آماده کنی. GitHub Security Advisories این فرآیند را مدیریت می‌کند.

**چرا مهم است؟** افشای بدون fix = کاربران در معرض خطر. افشای بدون هماهنگی = از دست دادن اعتماد. Advisory یک فضای خصوصی برای هماهنگی می‌دهد.

**کاربرد عملی:** اگر پروژه‌ی متن‌باز داری، این فصل ضروری است.

### مفاهیم اساسی

#### دو نوع advisory

| نوع | چه کسی | کجا |
|---|---|---|
| **Repository advisory** | نگه‌دارنده‌ی مخزن | مخزن خودت |
| **Global advisory** | GitHub / CVE Numbering Authority | GitHub Advisory Database |

#### چرخه‌ی عمر

```
۱. گزارش خصوصی (private vulnerability reporting)
۲. تأیید و ارزیابی
۳. draft advisory (خصوصی)
۴. توسعه‌ی fix در fork خصوصی
۵. درخواست CVE
۶. انتشار advisory + release fix
۷. اطلاع‌رسانی
```

#### اجزای advisory

| فیلد | مثال |
|---|---|
| **GHSA ID** | `GHSA-xxxx-yyyy-zzzz` |
| **CVE ID** | `CVE-2026-12345` |
| **Severity** | Critical / High / Medium / Low |
| **CVSS** | `CVSS:3.1/AV:N/AC:L/...` |
| **Affected versions** | `< 1.4.2` |
| **Patched versions** | `1.4.2` |
| **CWE** | `CWE-89` (SQL Injection) |
| **Description** | توضیح فنی |
| **Credits** | گزارش‌دهنده |

### مثال‌های عملی

#### مثال ۱: فعال کردن گزارش خصوصی

```
Settings → Code security → Private vulnerability reporting → Enable
```

حالا هر کسی می‌تواند از `Security → Advisories → Report a vulnerability` گزارش بدهد — بدون Issue عمومی.

**بدون این، گزارش‌دهنده مجبور است Issue عمومی بزند** — که آسیب‌پذیری را افشا می‌کند.

#### مثال ۲: ساخت draft advisory

```
Security → Advisories → New draft security advisory

Ecosystem: npm
Package name: @myorg/ui
Affected versions: < 1.4.2
Patched versions: 1.4.2
Severity: High
CWE: CWE-89

Title: SQL Injection in search endpoint
Description: ...
```

draft خصوصی است — فقط تیم می‌بیند.

#### مثال ۳: fork خصوصی برای fix

```
Advisory → "Start a temporary private fork"
```

یک fork خصوصی ساخته می‌شود که فقط تیم دسترسی دارد. fix را آنجا توسعه می‌دهی، بدون اینکه کسی ببیند.

بعد از آماده شدن:

```
Advisory → "Create pull request" → merge به main
```

#### مثال ۴: درخواست CVE

```
Advisory → "Request CVE"
```

GitHub یک CVE ID می‌گیرد (چون GitHub یک CNA است). این فرآیند معمولاً چند روز طول می‌کشد.

#### مثال ۵: انتشار

```
Advisory → "Publish advisory"
```

بعد از انتشار:

- advisory عمومی می‌شود
- در GitHub Advisory Database ثبت می‌شود
- Dependabot به همه‌ی مخزن‌های تحت تأثیر هشدار می‌دهد
- در `github.com/advisories` قابل جست‌وجو است

#### مثال ۶: متن advisory خوب

```markdown
## خلاصه
در نسخه‌های قبل از ۱.۴.۲، endpoint `POST /api/search` پارامتر `q` را
بدون sanitize در کوئری SQL استفاده می‌کند. مهاجم می‌تواند با یک
درخواست ساده، داده‌ی دلخواه را بخواند.

## جزئیات فنی
تابع `searchProducts(q)` در `src/repo/products.ts`:
```ts
db.query(`SELECT * FROM products WHERE name LIKE '%${q}%'`)
```
ورودی کاربر مستقیم در رشته‌ی SQL قرار می‌گیرد.

## اثبات مفهوم
```bash
curl -X POST https://api.example.com/search \
  -d '{"q": "%27 OR 1=1--"}'
# تمام محصولات، شامل غیرفعال‌ها، برگردانده می‌شود
```

## اثر
- خواندن داده‌ی غیرمجاز (فقط SELECT)
- بدون امکان نوشتن یا اجرای کد

## نسخه‌های تحت تأثیر
- `>= 1.0.0، < 1.4.2`

## نسخه‌ی رفع‌شده
- `1.4.2`

## راه‌حل موقت
اگر ارتقا ممکن نیست، endpoint را غیرفعال کنید یا ورودی را در
پروکسی با regex `^[\w\s-]+$` فیلتر کنید.

## اعتبار
ممنون از @researcher-x برای گزارش مسئولانه.
```

#### مثال ۷: هماهنگی با گزارش‌دهنده

```
۱. تأیید دریافت (۴۸ ساعت)
۲. ارزیابی اولیه (۵ روز)
۳. به‌روزرسانی منظم (هفتگی)
۴. هماهنگی تاریخ انتشار
۵. اعتبار در advisory
```

**مهم:** گزارش‌دهنده ممکن است بخواهد قبل از انتشار، مقاله بنویسد. تاریخ را هماهنگ کن.

#### مثال ۸: سیاست افشا در `SECURITY.md`

```markdown
## زمان‌بندی

| مرحله | زمان |
|---|---|
| تأیید دریافت | ۴۸ ساعت |
| ارزیابی اولیه | ۵ روز کاری |
| fix (Critical) | ۷ روز |
| fix (High) | ۳۰ روز |
| fix (Medium/Low) | ۹۰ روز |
| افشای عمومی | بعد از انتشار fix |

## محدوده

### در محدوده
- آسیب‌پذیری در کد این مخزن
- وابستگی‌های مستقیم با نسخه‌ی آسیب‌پذیر

### خارج از محدوده
- حمله‌ی DoS با حجم زیاد
- مسائل نیازمند دسترسی فیزیکی
- نسخه‌های EOL
- حمله به سرویس‌های شخص ثالث

## قدردانی
از گزارش‌دهنده در advisory و release notes تشکر می‌کنیم
(مگر اینکه ناشناس بخواهد).
```

#### مثال ۹: بررسی advisoryهای وابستگی

```bash
# advisoryهای مربوط به وابستگی‌های تو
gh api /advisories --paginate \
  -f ecosystem=npm \
  --jq '.[] | "\(.severity) \(.summary)"'
```

یا `npm audit` (فصل ۷۱).

#### مثال ۱۰: CVE در commit message

```
fix(security): رفع SQL injection در search (CVE-2026-12345)

Closes GHSA-xxxx-yyyy-zzzz
```

### مقایسه و الگوهای مختلف

#### Repository advisory در برابر Global advisory

| | Repository | Global |
|---|---|---|
| چه کسی می‌سازد | نگه‌دارنده | GitHub/مخزن |
| CVE | می‌تواند بگیرد | ✅ |
| در database | بعد از انتشار | ✅ |
| Dependabot alert | ✅ | ✅ |

#### افشای کامل در برابر افشای مسئولانه

| | کامل (full disclosure) | مسئولانه |
|---|---|---|
| سرعت | فوری | هماهنگ |
| خطر برای کاربران | ✅ زیاد | کم |
| اعتماد | ❌ | ✅ |
| توصیه | ❌ | ✅ |

#### CVE در برابر GHSA

| | CVE | GHSA |
|---|---|---|
| استاندارد | بین‌المللی | GitHub |
| فرمت | `CVE-YYYY-NNNNN` | `GHSA-xxxx-...` |
| پوشش | همه‌ی نرم‌افزار | GitHub ecosystem |
| رابطه | یک advisory می‌تواند هر دو داشته باشد | |

### اشکال شایع

#### ۱. گزارش خصوصی غیرفعال

گزارش‌دهنده مجبور است Issue عمومی بزند → افشا. **فعال کن.**

#### ۲. افشا قبل از fix

```
✗ Issue عمومی: "ما SQL injection داریم"
```

کاربران در معرض خطر و تو هنوز fix نداری.

#### ۳. advisory بدون نسخه‌ی رفع‌شده

اگر `patched versions` را نگذاری، Dependabot نمی‌تواند پیشنهاد بدهد.

#### ۴. فراموش کردن release

advisory منتشر شد ولی نسخه‌ی fix در registry نیست. کاربر نمی‌تواند ارتقا دهد.

#### ۵. CVE دیر درخواست شده

CVE گرفتن چند روز طول می‌کشد. **قبل از انتشار** درخواست بده.

#### ۶. عدم اطلاع‌رسانی

advisory منتشر شد ولی هیچ‌کس نمی‌داند. در Discussions، توییتر/شبکه‌های اجتماعی، و release notes اطلاع بده.

#### ۷. نادیده گرفتن گزارش‌دهنده

اگر در advisory از گزارش‌دهنده تشکر نکنی، دفعه‌ی بعد گزارش نمی‌دهد. اعتبار بده.

#### ۸. بدون محدوده در SECURITY.md

هر روز گزارش‌های خارج از محدوده می‌آید. محدوده را صریح بنویس.

### نکات مهم (Tips & Tricks)

- **Private vulnerability reporting** را روشن کن.
- **draft advisory** قبل از fix.
- **fork خصوصی** برای توسعه‌ی fix.
- **CVE قبل از انتشار** درخواست بده.
- **تاریخ انتشار را با گزارش‌دهنده هماهنگ کن.**
- **اعتبار بده** — فرهنگ گزارش‌دهی می‌سازد.

### خلاصه فصل

- Advisory فضای خصوصی برای هماهنگی افشا است.
- Private vulnerability reporting را روشن کن.
- draft → fork خصوصی → fix → CVE → انتشار.
- `patched versions` را دقیق پر کن.
- افشای مسئولانه، اعتماد می‌سازد.
- به گزارش‌دهنده اعتبار بده.

**سؤالات مروری:**
1. چرا گزارش خصوصی مهم است؟
2. چرخه‌ی advisory چیست؟
3. چرا CVE باید قبل از انتشار درخواست شود؟

---

## فصل ۷۵: SBOM، Artifact Attestations و زنجیره‌ی تأمین

### مقدمه

SBOM (Software Bill of Materials) فهرست همه‌ی اجزای نرم‌افزار توست. Attestation یک گواهی امضاشده است که می‌گوید «این artifact از این commit و این workflow ساخته شده».

**چرا مهم است؟** وقتی یک آسیب‌پذیری در یک کتابخانه پیدا می‌شود (مثل Log4Shell)، باید بدانی آیا از آن استفاده می‌کنی. بدون SBOM، باید همه‌چیز را دستی بگردی. و attestation ثابت می‌کند artifact دستکاری نشده.

**کاربرد عملی:** برای محصولات تجاری و دولتی، SBOM اغلب الزام قانونی است. برای متن‌باز، اعتماد می‌سازد.

### مفاهیم اساسی

#### SBOM — چه چیزی

| استاندارد | فرمت |
|---|---|
| **SPDX** | ISO استاندارد |
| **CycloneDX** | OWASP، محبوب در امنیت |

شامل:

- نام و نسخه‌ی هر وابستگی
- license
- hash
- رابطه‌ی وابستگی
- supplier

#### Attestation — چه چیزی

یک «statement» امضاشده با فرمت in-toto:

```json
{
  "_type": "https://in-toto.io/Statement/v1",
  "subject": [{ "name": "app.tar.gz", "digest": { "sha256": "..." } }],
  "predicateType": "https://slsa.dev/provenance/v1",
  "predicate": {
    "buildDefinition": {
      "buildType": "https://actions.github.io/buildtypes/workflow/v1",
      "externalParameters": {
        "workflow": { "ref": "refs/heads/main", "repository": "..." }
      }
    },
    "runDetails": { "builder": { "id": "..." } }
  }
}
```

#### SLSA levelها

| Level | معنی |
|---|---|
| 1 | provenance موجود |
| 2 | build سرویس میزبانی‌شده، امضاشده |
| 3 | جداسازی build، غیرقابل دستکاری |
| 4 | hermetic، reproducible |

GitHub Actions با attestation به **SLSA 3** می‌رسد (در بسیاری موارد).

### مثال‌های عملی

#### مثال ۱: تولید SBOM با Syft

```yaml
      - name: نصب Syft
        uses: anchore/sbom-action/download-syft@<sha>

      - name: تولید SBOM
        run: |
          syft dir:. -o spdx-json=sbom.spdx.json
          syft dir:. -o cyclonedx-json=sbom.cdx.json

      - uses: actions/upload-artifact@<sha>
        with:
          name: sbom
          path: sbom.*
```

#### مثال ۲: SBOM با `anchore/sbom-action`

```yaml
      - uses: anchore/sbom-action@<sha>
        with:
          path: .
          format: spdx-json
          artifact-name: sbom.spdx.json
```

#### مثال ۳: SBOM برای Docker image

```yaml
      - uses: anchore/sbom-action@<sha>
        with:
          image: ghcr.io/myorg/app:${{ github.sha }}
          format: cyclonedx-json
          artifact-name: sbom.cdx.json
```

#### مثال ۴: Attestation برای binary

```yaml
permissions:
  contents: read
  id-token: write
  attestations: write

jobs:
  build:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
      - run: ./build.sh

      - uses: actions/attest-build-provenance@<sha>
        with:
          subject-path: dist/app
```

#### مثال ۵: Attestation برای SBOM

```yaml
      - uses: actions/attest-sbom@<sha>
        with:
          subject-path: dist/app
          sbom-path: sbom.spdx.json
```

#### مثال ۶: بررسی attestation

```bash
# نصب gh (attestation در gh 2.49+)
gh attestation verify dist/app --repo OWNER/REPO

# خروجی:
# ✓ Verification succeeded!
# - The artifact was built by ...
# - The workflow was ...
```

#### مثال ۷: بررسی با cosign

```bash
# برای container
cosign verify-attestation \
  --type slsaprovenance \
  --certificate-identity-regexp "https://github.com/OWNER/REPO/.*" \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  ghcr.io/myorg/app:sha-abc123
```

#### مثال ۸: SBOM در release

```yaml
      - uses: softprops/action-gh-release@<sha>
        with:
          files: |
            dist/app.tar.gz
            sbom.spdx.json
            sbom.cdx.json
```

#### مثال ۹: بررسی SBOM برای آسیب‌پذیری

```bash
# Grype روی SBOM
grype sbom:sbom.spdx.json

# خروجی:
# NAME          INSTALLED  FIXED-IN  VULNERABILITY  SEVERITY
# lodash        4.17.20    4.17.21   CVE-2021-23337 High
```

#### مثال ۱۰: سیاست زنجیره‌ی تأمین

```yaml
# .github/workflows/supply-chain.yml
name: Supply Chain

on:
  push:
    branches: [main]
  release:
    types: [published]

permissions:
  contents: read
  id-token: write
  attestations: write

jobs:
  build:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
      - run: npm ci && npm run build

      - name: SBOM
        uses: anchore/sbom-action@<sha>
        with:
          path: dist
          format: spdx-json
          artifact-name: sbom.spdx.json

      - name: اسکن آسیب‌پذیری
        uses: anchore/scan-action@<sha>
        with:
          path: dist
          fail-build: true
          severity-cutoff: high

      - name: Attestation
        uses: actions/attest-build-provenance@<sha>
        with:
          subject-path: dist/**

      - name: SBOM attestation
        uses: actions/attest-sbom@<sha>
        with:
          subject-path: dist/**
          sbom-path: sbom.spdx.json
```

### مقایسه و الگوهای مختلف

#### SPDX در برابر CycloneDX

| | SPDX | CycloneDX |
|---|---|---|
| استاندارد | ISO | OWASP |
| تمرکز | license و composition | امنیت |
| ابزار | گسترده | ✅ امنیتی |
| مناسب | compliance | امنیت |

**توصیه:** هر دو تولید کن — ارزان است.

#### Attestation در برابر امضای ساده

| | Attestation | امضا (cosign) |
|---|---|---|
| چه می‌گوید | از کجا و چطور ساخته شد | چه کسی امضا کرد |
| SLSA | ✅ | ❌ |
| پیچیدگی | کم (action) | متوسط |

#### SBOM در برابر dependency graph

| | SBOM | dependency graph |
|---|---|---|
| فرمت | استاندارد | GitHub-specific |
| انتقال | ✅ | ❌ |
| license | ✅ | محدود |
| برای مشتری | ✅ | ❌ |

### اشکال شایع

#### ۱. SBOM بدون attestation

SBOM می‌تواند جعلی باشد. attestation ثابت می‌کند از کجا آمده.

#### ۲. attestation بدون `id-token: write`

```
Error: Failed to get ID token
```

#### ۳. SBOM فقط برای source، نه container

اگر container منتشر می‌کنی، SBOM باید شامل base image و layerها باشد.

#### ۴. فراموش کردن SBOM در release

SBOM تولید شد ولی به release الصاق نشد. مشتری نمی‌بیند.

#### ۵. SBOM بدون scan

SBOM فهرست است، نه هشدار. باید با Grype/Trivy اسکن شود.

#### ۶. `subject-path` اشتباه

```yaml
✗ subject-path: dist        # پوشه
✓ subject-path: dist/**     # فایل‌ها
```

#### ۷. attestation برای artifact تغییرکرده

اگر بعد از attestation، artifact را عوض کنی، attestation نامعتبر می‌شود (hash نمی‌خواند).

#### ۸. انتظار SLSA 4

GitHub Actions به SLSA 3 می‌رسد. SLSA 4 نیاز به build hermetic دارد که در Actions ممکن نیست (بدون تلاش زیاد).

### نکات مهم (Tips & Tricks)

- **SBOM را در هر release** الصاق کن.
- **attestation برای هر artifact** — یک action.
- **`gh attestation verify`** را به مشتری یاد بده.
- **SBOM + Grype** برای اسکن.
- **SPDX + CycloneDX** هر دو.
- **SLSA 3 کافی است** برای اکثر نیازها.

### خلاصه فصل

- SBOM = فهرست اجزا (SPDX/CycloneDX).
- Attestation = گواهی امضاشده‌ی منشأ (in-toto/SLSA).
- `actions/attest-build-provenance` و `attest-sbom`.
- `gh attestation verify` برای بررسی.
- SBOM را در release الصاق کن.
- GitHub Actions به SLSA 3 می‌رسد.

**سؤالات مروری:**
1. تفاوت SBOM و attestation چیست؟
2. چرا SBOM بدون attestation کافی نیست؟
3. چطور مشتری می‌تواند artifact را بررسی کند؟

---

**بعدی:** [`17-GitHub-CLI.md`](17-GitHub-CLI.md)

---

## منابع و مراجع

- GitHub Docs: Dependabot — https://docs.github.com/en/code-security/dependabot
- GitHub Docs: Code scanning — https://docs.github.com/en/code-security/code-scanning
- GitHub Docs: Secret scanning — https://docs.github.com/en/code-security/secret-scanning
- GitHub Docs: Repository security advisories — https://docs.github.com/en/code-security/security-advisories
- GitHub Docs: Artifact attestations — https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations
- GitHub Advisory Database — https://github.com/advisories
- SLSA — https://slsa.dev/
- CycloneDX — https://cyclonedx.org/
- SPDX — https://spdx.dev/
- `gitleaks` — https://github.com/gitleaks/gitleaks
- `anchore/sbom-action` — https://github.com/anchore/sbom-action
