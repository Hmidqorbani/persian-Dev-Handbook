# بخش ۱۰: GitHub Actions — مبانی

**مدت یادگیری**: ۵ تا ۶ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۰۲ و ۰۳ (Git، branch)

---

## فهرست محتوا

- [فصل ۴۲: Actions چیست — workflow، event، job، step، runner](#فصل-۴۲)
- [فصل ۴۳: اولین workflow — CI برای تست و لینت](#فصل-۴۳)
- [فصل ۴۴: Matrix build — چند نسخه، چند سیستم‌عامل](#فصل-۴۴)
- [فصل ۴۵: عبارات، context و شرط‌ها](#فصل-۴۵)
- [فصل ۴۶: دیباگ workflow — لاگ، debug، اجرای محلی](#فصل-۴۶)

---

## فصل ۴۲: Actions چیست — workflow، event، job، step، runner

### مقدمه

GitHub Actions یک پلتفرم CI/CD و اتوماسیون است که داخل خود GitHub زندگی می‌کند. هر رویدادی در مخزن (push، PR، issue، schedule) می‌تواند یک workflow را اجرا کند، و آن workflow روی ماشین‌هایی که GitHub می‌دهد (یا خودت) اجرا می‌شود.

**چرا مهم است؟** قبل از Actions، برای CI نیاز به Jenkins/CircleCI/Travis و یک سرور جدا داشتی. حالا یک فایل YAML در مخزن، و CI داری. برای پروژه‌های متن‌باز، رایگان و نامحدود.

**کاربرد عملی:** تست خودکار، build، deploy، انتشار نسخه، برچسب زدن Issue، ساخت مستندات — همه با یک ابزار.

### مفاهیم اساسی

#### هفت مفهوم پایه

| مفهوم | چیست | کجا |
|---|---|---|
| **Workflow** | یک فرآیند خودکار کامل | `.github/workflows/*.yml` |
| **Event** | چیزی که workflow را trigger می‌کند | `on:` |
| **Job** | مجموعه‌ای از stepها روی یک runner | `jobs:` |
| **Step** | یک کار (دستور یا action) | `steps:` |
| **Action** | واحد قابل استفاده‌ی مجدد | `uses:` |
| **Runner** | ماشینی که job را اجرا می‌کند | `runs-on:` |
| **Artifact** | فایلی که بین jobها یا بعد از اجرا می‌ماند | `actions/upload-artifact` |

#### ساختار یک workflow

```yaml
name: CI                          # ← نام workflow

on:                               # ← eventها
  push:
    branches: [main]
  pull_request:

env:                              # ← متغیرهای سراسری
  NODE_VERSION: "22"

jobs:                             # ← jobها
  test:
    runs-on: ubuntu-latest        # ← runner
    steps:                        # ← stepها
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test
```

#### رویدادهای مهم

| Event | کِی |
|---|---|
| `push` | push به branch/tag |
| `pull_request` | PR باز/به‌روز/بسته شد |
| `pull_request_target` | مثل بالا ولی در context مخزن base (خطرناک) |
| `workflow_dispatch` | دستی (دکمه در UI) |
| `schedule` | cron |
| `release` | release منتشر شد |
| `issues` | Issue باز/بسته/برچسب |
| `issue_comment` | کامنت روی Issue/PR |
| `workflow_call` | از workflow دیگر صدا زده شد |
| `merge_group` | آیتم در merge queue |

#### انواع runner

| Runner | مشخصات | هزینه |
|---|---|---|
| `ubuntu-latest` | ۴ vCPU، ۱۶GB RAM، ۱۴GB SSD | ۱× دقیقه |
| `ubuntu-24.04` | نسخه‌ی مشخص | ۱× |
| `windows-latest` | ۴ vCPU، ۱۶GB | ۲× |
| `macos-latest` | ۳-۴ vCPU، ۱۴GB (Apple Silicon یا Intel) | ۱۰× |
| `macos-14` (M1) | Apple Silicon | ۱۰× |
| **Larger runners** | ۸-۹۶ vCPU | پلن پرداختی |
| **Self-hosted** | هر چیزی | خودت |

**نکته:** `-latest` ممکن است ناگهان عوض شود (مثلاً `ubuntu-latest` از ۲۲.۰۴ به ۲۴.۰۴ رفت). برای production، نسخه‌ی صریح بگذار.

#### محدودیت‌ها

| محدودیت | مقدار |
|---|---|
| زمان هر job | ۶ ساعت |
| زمان کل workflow | ۳۵ روز (در انتظار approval) |
| هم‌زمانی jobها | ۲۰ (Free)، ۵۰۰ (Enterprise) |
| ماتریس | ۲۵۶ job |
| فضای دیسک | ~۱۴GB (ubuntu) |
| اندازه‌ی artifact | ۱۰GB (پلن رایگان)، ۹۰ روز نگهداری |

### مثال‌های عملی

#### مثال ۱: workflow حداقلی

`.github/workflows/hello.yml`:

```yaml
name: سلام

on:
  workflow_dispatch:

jobs:
  greet:
    runs-on: ubuntu-latest
    steps:
      - name: چاپ پیام
        run: echo "سلام از GitHub Actions"

      - name: اطلاعات محیط
        run: |
          echo "OS: $RUNNER_OS"
          echo "Arch: $RUNNER_ARCH"
          echo "Repo: $GITHUB_REPOSITORY"
          echo "SHA: $GITHUB_SHA"
          echo "Ref: $GITHUB_REF"
          echo "Actor: $GITHUB_ACTOR"
```

برای اجرا: **Actions** tab → **سلام** → **Run workflow**.

#### مثال ۲: CI کامل برای Node.js

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - name: دریافت کد
        uses: actions/checkout@v4

      - name: نصب Node
        uses: actions/setup-node@v4
        with:
          node-version: "22"
          cache: "npm"

      - name: نصب وابستگی‌ها
        run: npm ci

      - name: لینت
        run: npm run lint

      - name: بررسی تایپ
        run: npm run typecheck

      - name: تست
        run: npm test -- --coverage

      - name: build
        run: npm run build
```

#### مثال ۳: CI برای Python

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.11", "3.12", "3.13"]
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
          cache: "pip"

      - run: pip install -r requirements-dev.txt
      - run: ruff check .
      - run: mypy src/
      - run: pytest --cov=src --cov-report=term-missing
```

#### مثال ۴: چند job با وابستگی

```yaml
name: Pipeline

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: npm }
      - run: npm ci
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: npm }
      - run: npm ci
      - run: npm test

  build:
    needs: [lint, test]        # ← بعد از هر دو
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: npm }
      - run: npm ci
      - run: npm run build
```

`lint` و `test` موازی اجرا می‌شوند؛ `build` بعد از موفقیت هر دو.

#### مثال ۵: workflow با Docker

```yaml
name: Docker

on:
  push:
    branches: [main]
    tags: ["v*.*.*"]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login به GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build و push
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

#### مثال ۶: بررسی syntax قبل از push

```bash
# نصب actionlint
brew install actionlint     # macOS
# یا
go install github.com/rhysd/actionlint/cmd/actionlint@latest

# بررسی
actionlint .github/workflows/*.yml
```

`actionlint` خطاهای YAML، contextهای نامعتبر، و حتی مشکلات shell را می‌گیرد. **قبل از هر push اجرا کن.**

### مقایسه و الگوهای مختلف

#### Actions در برابر Jenkins/CircleCI

| | Actions | Jenkins | CircleCI |
|---|---|---|---|
| نصب | صفر | سرور + plugin | اکانت |
| یکپارچگی با GitHub | ✅ بومی | plugin | خوب |
| متن‌باز رایگان | ✅ نامحدود | خودت میزبانی | محدود |
| اکوسیستم | بزرگ‌ترین Marketplace | plugin | orb |
| self-host | runner | ✅ | ✅ |
| مناسب | اکثر پروژه‌ها | سازمان با نیاز خاص | تیم با CI پیچیده |

#### `run` در برابر `uses`

| | `run:` | `uses:` |
|---|---|---|
| چه می‌کند | دستور shell | اجرای action |
| مثال | `run: npm test` | `uses: actions/checkout@v4` |
| مناسب | منطق سفارشی | کارهای استاندارد |
| کش/تکرار | ❌ | ✅ |

**قاعده:** اگر کاری را بیش از دو بار می‌نویسی، به action تبدیل کن (فصل ۴۹).

#### `pull_request` در برابر `pull_request_target`

| | `pull_request` | `pull_request_target` |
|---|---|---|
| کد اجراشده | کد PR (از fork) | کد base |
| دسترسی به secrets | ❌ | ✅ |
| امنیت | ✅ | **خطرناک** |

فصل ۵۴ کامل.

### اشکال شایع

#### ۱. YAML اشتباه — بی‌صدا

```yaml
✗ runs-on: ubuntu-latest    (در سطح jobs، نه job)
✗ steps:                    (indentation اشتباه)
```

همیشه `actionlint` بزن.

#### ۲. `actions/checkout` را فراموش کردم

```
fatal: not a git repository
```

هر job باید با `checkout` شروع شود (مگر اینکه به کد نیاز نداشته باشد).

#### ۳. `npm ci` بدون `package-lock.json`

```
npm error: The `npm ci` command can only install with an existing package-lock.json
```

`npm ci` نیاز به lock دارد. یا lock را commit کن، یا از `npm install` استفاده کن.

#### ۴. کش کار نمی‌کند

```yaml
✗ with:
    cache: npm
```

باید `setup-node` قبل از `npm ci` باشد و `package-lock.json` وجود داشته باشد.

#### ۵. `ubuntu-latest` ناگهان عوض شد

نسخه‌ی OS تغییر کرد و چیزی شکست. برای production، `ubuntu-24.04` صریح بگذار.

#### ۶. workflow روی fork اجرا نمی‌شود

PR از fork، نیاز به تأیید maintainer. `Settings → Actions → Fork pull request workflows`.

#### ۷. secrets در PR از fork خالی است

طراحی امنیتی. برای تست، workflow را طوری بنویس که بدون secret هم کار کند.

#### ۸. job طولانی‌تر از ۶ ساعت

```
The job running on runner ... has exceeded the maximum execution time of 360 minutes.
```

بشکن به jobهای کوچک‌تر، یا از self-hosted runner استفاده کن.

### نکات مهم (Tips & Tricks)

- **`actionlint`** را در pre-commit بگذار.
- **`permissions: contents: read`** را در همه‌ی workflowها صریح بنویس.
- **`concurrency`** برای لغو اجراهای قدیمی:
  ```yaml
  concurrency:
    group: ${{ github.workflow }}-${{ github.ref }}
    cancel-in-progress: true
  ```
- **`workflow_dispatch`** به همه اضافه کن — برای دیباگ حیاتی است.
- **نسخه‌ی runner را صریح بگذار** (`ubuntu-24.04` نه `ubuntu-latest`).
- **`timeout-minutes`** برای jobها بگذار — جلوگیری از مصرف بی‌رویه:
  ```yaml
  jobs:
    test:
      timeout-minutes: 15
  ```

### خلاصه فصل

- هفت مفهوم: workflow، event، job، step، action، runner، artifact.
- `on:` trigger، `jobs:` کارها، `steps:` دستورها.
- runnerها: ubuntu (۱×)، windows (۲×)، macos (۱۰×).
- محدودیت‌ها: ۶ ساعت per job، ۲۰ job هم‌زمان.
- `actions/checkout` اولین step اکثر jobهاست.
- `actionlint` قبل از push.

**سؤالات مروری:**
1. تفاوت job و step چیست؟
2. چرا `ubuntu-latest` برای production خطرناک است؟
3. چرا workflow روی PR از fork به secrets دسترسی ندارد؟

---

## فصل ۴۳: اولین workflow — CI برای تست و لینت

### مقدمه

این فصل یک CI واقعی و کامل می‌سازد: از صفر تا workflowی که روی هر PR اجرا می‌شود، تست می‌کند، و در Ruleset به‌عنوان required check ثبت می‌شود.

**چرا مهم است؟** CI اولین قدم بلوغ یک مخزن است. بدون آن، هر merge یک ریسک است.

**کاربرد عملی:** الگوی این فصل را می‌توانی در هر پروژه‌ای کپی کنی.

### مفاهیم اساسی

#### CI خوب چه ویژگی دارد

| ویژگی | چرا |
|---|---|
| **سریع** | < ۵ دقیقه ایده‌آل |
| **قابل اعتماد** | بدون flaky test |
| **شفاف** | لاگ واضح، پیام خطای مفید |
| **موازی** | jobها هم‌زمان |
| **کش‌شده** | وابستگی‌ها کش شوند |
| **قابل تکرار** | همان نتیجه در local |

#### مراحل استاندارد CI

```
۱. checkout
۲. setup (زبان + کش)
۳. install dependencies
۴. lint
۵. typecheck
۶. test
۷. build
```

### مثال‌های عملی

#### مثال ۱: CI کامل Node.js + TypeScript

`.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
  merge_group:

permissions:
  contents: read

concurrency:
  group: ci-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}

jobs:
  lint:
    name: lint
    runs-on: ubuntu-24.04
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "22"
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm run format:check

  typecheck:
    name: typecheck
    runs-on: ubuntu-24.04
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "22"
          cache: npm
      - run: npm ci
      - run: npm run typecheck

  test:
    name: test
    runs-on: ubuntu-24.04
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "22"
          cache: npm
      - run: npm ci
      - run: npm test -- --coverage
      - name: آپلود گزارش پوشش
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/
          retention-days: 7

  build:
    name: build
    needs: [lint, typecheck, test]
    runs-on: ubuntu-24.04
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "22"
          cache: npm
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/
          retention-days: 7

  ci-success:
    name: ci-success
    needs: [lint, typecheck, test, build]
    if: always()
    runs-on: ubuntu-24.04
    steps:
      - name: بررسی نتیجه
        run: |
          echo "نتایج: ${{ join(needs.*.result, ', ') }}"
          if echo "${{ join(needs.*.result, ' ') }}" | grep -qE 'failure|cancelled'; then
            echo "❌ یکی از jobها شکست خورد"
            exit 1
          fi
          echo "✅ همه سبز"
```

**چرا `ci-success`؟** فقط این یک job را required می‌کنی. هر تغییری در jobهای دیگر، ruleset را نمی‌شکند.

#### مثال ۲: CI برای Python با poetry

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: نصب poetry
        run: pipx install poetry

      - name: کش
        uses: actions/cache@v4
        with:
          path: ~/.cache/pypoetry
          key: poetry-${{ hashFiles('poetry.lock') }}
          restore-keys: poetry-

      - run: poetry install --no-interaction

      - run: poetry run ruff check .
      - run: poetry run ruff format --check .
      - run: poetry run mypy src/
      - run: poetry run pytest --cov=src --cov-fail-under=80
```

#### مثال ۳: CI برای Go

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-go@v5
        with:
          go-version: "1.23"
          cache: true

      - run: go vet ./...
      - run: go test -race -coverprofile=coverage.out ./...
      - run: go build ./...

      - name: golangci-lint
        uses: golangci/golangci-lint-action@v6
        with:
          version: latest
```

#### مثال ۴: CI با دیتابیس (services)

```yaml
jobs:
  test:
    runs-on: ubuntu-24.04

    services:
      postgres:
        image: postgres:17
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s

    env:
      DATABASE_URL: postgres://postgres:postgres@localhost:5432/test
      REDIS_URL: redis://localhost:6379

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: npm }
      - run: npm ci
      - run: npx prisma migrate deploy
      - run: npm test
```

`services` یک container در کنار job اجرا می‌کند — بدون نیاز به docker-compose.

#### مثال ۵: ثبت به‌عنوان required check

```bash
# بعد از اولین اجرای موفق، اسم check را ببین
gh pr checks 42
# ✓  CI / ci-success   3m12s

# در Ruleset اضافه کن
gh api -X POST repos/OWNER/REPO/rulesets --input - <<'JSON'
{
  "name": "require-ci",
  "target": "branch",
  "enforcement": "active",
  "conditions": { "ref_name": { "include": ["~DEFAULT_BRANCH"], "exclude": [] } },
  "rules": [{
    "type": "required_status_checks",
    "parameters": {
      "strict_required_status_checks_policy": true,
      "required_status_checks": [{ "context": "CI / ci-success" }]
    }
  }]
}
JSON
```

#### مثال ۶: badge در README

```markdown
[![CI](https://github.com/USER/REPO/actions/workflows/ci.yml/badge.svg)](https://github.com/USER/REPO/actions/workflows/ci.yml)
```

برای branch خاص:

```markdown
![CI](https://github.com/USER/REPO/actions/workflows/ci.yml/badge.svg?branch=main)
```

### مقایسه و الگوهای مختلف

#### یک job در برابر چند job

| | یک job | چند job |
|---|---|---|
| سرعت | کند (سری) | ✅ سریع (موازی) |
| setup تکراری | ❌ | ✅ (هر job جدا) |
| خوانایی | ساده | واضح‌تر (هر شکست مشخص) |
| مصرف دقیقه | کمتر | بیشتر |

**توصیه:** ۳-۵ job. اگر بیشتر شد، در مصرف دقیقه صرفه‌جویی کن.

#### `npm ci` در برابر `npm install`

| | `npm ci` | `npm install` |
|---|---|---|
| نیاز به lock | ✅ | ❌ |
| سرعت | سریع‌تر | کندتر |
| تغییر lock | ❌ | ✅ |
| مناسب CI | ✅ | ❌ |

#### کش: `setup-node` در برابر `actions/cache`

| | `setup-node` cache | `actions/cache` |
|---|---|---|
| سادگی | ✅ یک خط | چند خط |
| انعطاف | محدود | کامل |
| مناسب | ۹۰٪ موارد | monorepo، کش سفارشی |

### اشکال شایع

#### ۱. CI کند است (۱۰+ دقیقه)

- وابستگی‌ها کش نشده
- jobها سری اجرا می‌شوند
- تست‌های integration سنگین در هر push

راه‌حل: کش + موازی‌سازی + جدا کردن تست سنگین (فصل ۶۱).

#### ۲. flaky test

تست که گاهی پاس و گاهی شکست. CI را بی‌اعتبار می‌کند. راه‌حل: `retry`، رفع منبع flakiness، یا quarantining.

#### ۳. `npm ci` با lock قدیمی

```
npm error: `npm ci` can only install packages when your package.json
and package-lock.json are in sync
```

باید `npm install` محلی بزنی و lock را commit کنی.

#### ۴. تست‌ها در CI پاس می‌شوند ولی محلی نه (یا برعکس)

- تفاوت نسخه‌ی Node
- تفاوت timezone (CI معمولاً UTC)
- تفاوت مسیر فایل (حساسیت به حروف)
- متغیرهای محیطی

**راه‌حل:** `.nvmrc`، `TZ=UTC` در تست، `path` نسبی.

#### ۵. `permissions` نداشتن

```
Resource not accessible by integration
```

`permissions: contents: read` یا بیشتر اضافه کن.

#### ۶. `concurrency` که jobهای دیگر را cancel می‌کند

```yaml
✗ concurrency:
    group: ci
    cancel-in-progress: true
```

همه‌ی اجراها یک گروه → هر push، قبلی را cancel می‌کند (حتی روی branch دیگر). از `${{ github.ref }}` استفاده کن.

### نکات مهم (Tips & Tricks)

- **`ci-success` job** برای required check پایدار.
- **`timeout-minutes`** در هر job — جلوگیری از jobهای گیرکرده.
- **`concurrency` با `github.ref`** — لغو اجراهای قدیمی همان branch.
- **`if: always()`** برای آپلود artifact حتی در شکست.
- **`TZ=UTC`** در env تست‌ها — نتایج یکسان.
- **badge** در README — اعتماد می‌سازد.
- **`needs` را برای build بگذار** — build فقط بعد از تست.

### خلاصه فصل

- CI استاندارد: checkout → setup → install → lint → typecheck → test → build.
- **`ci-success` job** ruleset را پایدار می‌کند.
- `services` برای دیتابیس/Redis در job.
- `concurrency` اجراهای قدیمی را لغو می‌کند.
- `npm ci` + lock در CI الزامی است.

**سؤالات مروری:**
1. چرا `ci-success` job لازم است؟
2. چطور یک دیتابیس Postgres در job راه بیندازی؟
3. سه دلیل رایج تفاوت نتیجه‌ی CI و local چیست؟

---

## فصل ۴۴: Matrix build — چند نسخه، چند سیستم‌عامل

### مقدمه

Matrix یک قابلیت ساده با تأثیر بزرگ است: یک job را با چند مقدار مختلف اجرا می‌کند. یک تعریف، ۹ اجرا.

**چرا مهم است؟** اگر کتابخانه‌ای می‌نویسی، باید روی Node 20 و 22، Ubuntu و Windows و macOS تست شود. بدون matrix، ۹ job دستی می‌نویسی.

**کاربرد عملی:** matrix برای تست سازگاری، build چند پلتفرم، و اجرای موازی تست‌ها.

### مفاهیم اساسی

#### سینتکس

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: [20, 22]
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
```

نتیجه: ۶ job (`os` × `node`).

#### گزینه‌های `strategy`

| گزینه | کار |
|---|---|
| `fail-fast: true` (پیش‌فرض) | اگر یکی شکست، بقیه cancel شوند |
| `fail-fast: false` | همه اجرا شوند (برای دیدن همه‌ی شکست‌ها) |
| `max-parallel: N` | حداکثر N هم‌زمان |
| `matrix.include` | افزودن ترکیب خاص |
| `matrix.exclude` | حذف ترکیب |

#### `include` و `exclude`

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, macos-latest]
    node: [20, 22]
    exclude:
      - os: macos-latest
        node: 20          # macOS با Node 20 را اجرا نکن
    include:
      - os: ubuntu-latest
        node: 23
        experimental: true   # متغیر اضافی
```

**نکته‌ی ظریف `include`:** اگر `include` با یک ترکیب موجود منطبق شود، متغیر اضافه می‌کند. اگر منطبق نشود، یک job **جدید** می‌سازد.

### مثال‌های عملی

#### مثال ۱: matrix دو بعدی

```yaml
name: Test Matrix

on: [push, pull_request]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-24.04, windows-2022, macos-14]
        node: [20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
          cache: npm
      - run: npm ci
      - run: npm test
```

۶ job موازی. اسم هر check: `test (ubuntu-24.04, 20)`، `test (windows-2022, 22)` و ...

#### مثال ۲: matrix با include برای build

```yaml
jobs:
  build:
    strategy:
      matrix:
        include:
          - os: ubuntu-24.04
            target: linux-x64
            artifact: app-linux
          - os: ubuntu-24.04
            target: linux-arm64
            artifact: app-linux-arm
            cross: true
          - os: macos-14
            target: darwin-arm64
            artifact: app-macos
          - os: windows-2022
            target: win-x64
            artifact: app.exe
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - run: ./build.sh --target ${{ matrix.target }}
      - uses: actions/upload-artifact@v4
        with:
          name: ${{ matrix.artifact }}
          path: dist/
```

#### مثال ۳: matrix دینامیک با `fromJSON`

```yaml
jobs:
  setup:
    runs-on: ubuntu-latest
    outputs:
      versions: ${{ steps.set.outputs.versions }}
    steps:
      - id: set
        run: |
          # از یک فایل یا API می‌خواند
          echo 'versions=["20","22","23"]' >> "$GITHUB_OUTPUT"

  test:
    needs: setup
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node: ${{ fromJSON(needs.setup.outputs.versions) }}
    steps:
      - uses: actions/setup-node@v4
        with: { node-version: "${{ matrix.node }}" }
      - run: node --version
```

#### مثال ۴: تقسیم تست‌ها با matrix

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        shard: [1, 2, 3, 4]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: npm }
      - run: npm ci
      - run: npm test -- --shard=${{ matrix.shard }}/4
```

۴ برابر سریع‌تر برای تست‌های زیاد. Jest و Vitest از `--shard` پشتیبانی می‌کنند.

#### مثال ۵: matrix با service

```yaml
jobs:
  test:
    strategy:
      matrix:
        db: [postgres:16, postgres:17, postgres:18]
    runs-on: ubuntu-latest
    services:
      db:
        image: ${{ matrix.db }}
        env:
          POSTGRES_PASSWORD: test
        ports: ["5432:5432"]
        options: >-
          --health-cmd pg_isready --health-interval 10s --health-retries 5
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test
        env:
          DATABASE_URL: postgres://postgres:test@localhost:5432/postgres
```

#### مثال ۶: job جمع‌کننده برای matrix

اگر matrix را required کنی، هر تغییر نسخه ruleset را می‌شکند. راه‌حل:

```yaml
jobs:
  test:
    strategy:
      fail-fast: false
      matrix:
        node: [20, 22]
        os: [ubuntu-24.04, windows-2022]
    runs-on: ${{ matrix.os }}
    steps:
      - run: echo "test"

  all-tests:
    name: all-tests
    needs: [test]
    if: always()
    runs-on: ubuntu-24.04
    steps:
      - run: |
          if echo "${{ join(needs.*.result, ' ') }}" | grep -qE 'failure|cancelled'; then
            exit 1
          fi
```

#### مثال ۷: matrix در gh

```bash
gh run list --workflow=ci.yml --limit 5
gh run view <run-id>
# ✓ test (ubuntu-24.04, 20)
# ✓ test (ubuntu-24.04, 22)
# ✗ test (windows-2022, 20)
# ✓ test (windows-2022, 22)
```

### مقایسه و الگوهای مختلف

#### matrix در برابر jobهای دستی

| | matrix | دستی |
|---|---|---|
| تعریف | یک بلوک | N بلوک |
| نگهداری | ✅ آسان | ❌ |
| مصرف دقیقه | یکسان | یکسان |
| خوانایی | ✅ | ❌ |

#### `fail-fast: true` در برابر `false`

| | true | false |
|---|---|---|
| سرعت بازخورد | ✅ سریع | کندتر |
| دیدن همه‌ی شکست‌ها | ❌ | ✅ |
| مصرف دقیقه | کمتر | بیشتر |
| مناسب | PR (فقط می‌خواهی بدانی سبز است) | branch اصلی / شبانه |

**توصیه:** `false` برای matrix بزرگ، `true` برای کوچک.

#### matrix در برابر shard

| | matrix (نسخه‌ها) | shard (تقسیم تست) |
|---|---|---|
| هدف | سازگاری | سرعت |
| تعداد | ۲-۹ | ۲-۱۰ |
| تست کامل | ✅ در هر job | ❌ هر job بخشی |

### اشکال شایع

#### ۱. انفجار تعداد job

```yaml
✗ ۵ OS × ۵ نسخه × ۴ دیتابیس = ۱۰۰ job
```

مصرف دقیقه‌ی نجومی. کاهش بده: ۲ OS × ۳ نسخه کافی است.

#### ۲. اسم check غیرقابل پیش‌بینی

`test (ubuntu-latest, 20)` — اگر `ubuntu-latest` عوض شود، اسم عوض می‌شود. از `ubuntu-24.04` استفاده کن.

#### ۳. `include` که job اضافه می‌کند نه متغیر

```yaml
matrix:
  os: [ubuntu-latest]
  include:
    - os: windows-latest    # ← این یک job جدید می‌سازد
```

اگر می‌خواهی فقط متغیر اضافه کنی، باید `os` موجود باشد.

#### ۴. `exclude` بعد از `include`

ترتیب مهم است: `include` بعد از `exclude` اعمال می‌شود. اگر ترکیبی را exclude کنی و بعد include، ممکن است برگردد.

#### ۵. matrix با `needs` ناسازگار

```yaml
✗ build:
    needs: test
    strategy:
      matrix: ...
```

`needs` به کل job اشاره می‌کند، نه به یک matrix item. build بعد از **همه‌ی** testها اجرا می‌شود (یا هیچ‌کدام اگر fail-fast).

#### ۶. matrix + secret در fork

secretها در fork خالی‌اند. matrix اجرا می‌شود ولی تست‌های وابسته به secret شکست می‌خورند. با `if` شرطی کن.

### نکات مهم (Tips & Tricks)

- **`fail-fast: false`** برای دیدن همه‌ی شکست‌ها.
- **`max-parallel`** برای کنترل هزینه و rate limit.
- **`fromJSON`** برای matrix دینامیک از فایل/API.
- **`--shard`** در Jest/Vitest برای تقسیم تست.
- **`ubuntu-24.04` صریح** به‌جای `-latest`.
- **`all-tests` job** برای required check پایدار.

### خلاصه فصل

- matrix یک job را با چند مقدار اجرا می‌کند: `matrix.os`، `matrix.node`.
- `include`/`exclude` ترکیب‌ها را تنظیم می‌کنند.
- `fail-fast: false` برای دیدن همه‌ی شکست‌ها.
- `fromJSON` برای matrix دینامیک.
- `--shard` تست‌ها را تقسیم می‌کند.
- job جمع‌کننده برای required check.

**سؤالات مروری:**
1. چرا `fail-fast: false` در matrix بزرگ بهتر است؟
2. تفاوت `include` که متغیر اضافه می‌کند و `include` که job می‌سازد چیست؟
3. چطور تست‌ها را ۴ برابر سریع‌تر کنی؟

---

## فصل ۴۵: عبارات، context و شرط‌ها

### مقدمه

workflow بدون شرط، یک اسکریپت خطی است. با `if`، `${{ }}`، و contextها، یک برنامه می‌شود.

**چرا مهم است؟** ۹۰٪ workflowهای پیشرفته از شرط استفاده می‌کنند: «فقط روی main deploy کن»، «فقط اگر فایل‌های src عوض شدند تست کن»، «فقط اگر PR از fork نیست».

**کاربرد عملی:** بعد از این فصل می‌توانی workflowهای شرطی و هوشمند بنویسی.

### مفاهیم اساسی

#### سینتکس `${{ }}`

هر چیزی داخل `${{ }}` یک عبارت است:

```yaml
run: echo "Branch: ${{ github.ref_name }}"
if: ${{ github.event_name == 'push' }}
env:
  VERSION: ${{ github.sha }}
```

**نکته:** در `if:` نیازی به `${{ }}` نیست (خودش expression است)، ولی نوشتنش خطا نمی‌دهد.

#### contextهای مهم

| Context | چه چیزی |
|---|---|
| `github` | اطلاعات رویداد و مخزن |
| `env` | متغیرهای محیطی |
| `vars` | متغیرهای تنظیمات (Settings → Variables) |
| `secrets` | secretها |
| `job` | اطلاعات job جاری |
| `steps` | خروجی stepهای قبلی |
| `needs` | خروجی jobهای وابسته |
| `matrix` | مقدار matrix |
| `runner` | اطلاعات runner |
| `inputs` | ورودی‌های workflow_dispatch/workflow_call |

#### `github` — پرکاربردترین

| فیلد | مثال |
|---|---|
| `github.repository` | `myorg/backend` |
| `github.repository_owner` | `myorg` |
| `github.ref` | `refs/heads/main` |
| `github.ref_name` | `main` |
| `github.sha` | `a1b2c3d...` |
| `github.actor` | `ali` |
| `github.event_name` | `push`، `pull_request` |
| `github.event.pull_request.number` | `42` |
| `github.event.pull_request.head.ref` | `feat/login` |
| `github.event.pull_request.base.ref` | `main` |
| `github.event.pull_request.head.repo.fork` | `true` / `false` |
| `github.event.label.name` | `bug` |
| `github.token` | توکن (مثل secrets.GITHUB_TOKEN) |
| `github.workspace` | مسیر checkout |

#### عملگرها

| عملگر | معنی |
|---|---|
| `==`، `!=` | برابر/نابرابر (case-insensitive برای رشته) |
| `&&`، `\|\|` | و / یا |
| `!` | نقیض |
| `>`، `<`، `>=`، `<=` | مقایسه |
| `contains(a, b)` | آیا a شامل b است |
| `startsWith(a, b)` | شروع |
| `endsWith(a, b)` | پایان |
| `format(fmt, ...)` | قالب‌بندی |
| `join(arr, sep)` | اتصال |
| `toJSON(x)` | تبدیل به JSON |
| `fromJSON(s)` | تبدیل از JSON |
| `success()`، `failure()`، `always()`، `cancelled()` | وضعیت |

**تله‌ی مهم:** در expressionهای GitHub، رشته‌ی خالی و `0` و `null` و `false` همه falsy هستند. ولی `'false'` (رشته) **truthy** است! این منبع باگ‌های عجیب است.

### مثال‌های عملی

#### مثال ۱: deploy فقط روی main

```yaml
jobs:
  deploy:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh
```

#### مثال ۲: skip بر اساس پیام commit

```yaml
jobs:
  test:
    if: "!contains(github.event.head_commit.message, '[skip ci]')"
    runs-on: ubuntu-latest
```

**هشدار:** اگر این job required باشد و skip شود، check با `skipped` ثبت می‌شود که **required را پاس می‌کند**. یعنی CI دور زده می‌شود. فصل ۴۰.

#### مثال ۳: تشخیص fork

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test
      - name: تست‌های نیازمند secret
        if: github.event.pull_request.head.repo.fork == false
        run: npm run test:integration
        env:
          API_KEY: ${{ secrets.API_KEY }}
```

#### مثال ۴: شرط بر اساس فایل‌های تغییر‌یافته

```yaml
jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      backend: ${{ steps.filter.outputs.backend }}
      frontend: ${{ steps.filter.outputs.frontend }}
      docs: ${{ steps.filter.outputs.docs }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            backend:
              - 'backend/**'
              - 'shared/**'
            frontend:
              - 'frontend/**'
              - 'shared/**'
            docs:
              - '**/*.md'

  test-backend:
    needs: changes
    if: needs.changes.outputs.backend == 'true'
    runs-on: ubuntu-latest
    steps:
      - run: echo "تست بک‌اند"

  test-frontend:
    needs: changes
    if: needs.changes.outputs.frontend == 'true'
    runs-on: ubuntu-latest
    steps:
      - run: echo "تست فرانت"

  docs-check:
    needs: changes
    if: needs.changes.outputs.docs == 'true'
    runs-on: ubuntu-latest
    steps:
      - run: echo "بررسی لینک‌های مستندات"
```

**نکته‌ی حیاتی:** اگر job با `if` skip شود، check `skipped` می‌شود. برای required check، از job جمع‌کننده استفاده کن که همیشه اجرا شود.

#### مثال ۵: `if` با `always()` برای پاکسازی

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: npm test

  cleanup:
    needs: test
    if: always()          # حتی اگر test شکست خورد
    runs-on: ubuntu-latest
    steps:
      - run: ./cleanup.sh
```

#### مثال ۶: خروجی step به step

```yaml
steps:
  - id: version
    run: echo "value=$(node -p "require('./package.json').version")" >> "$GITHUB_OUTPUT"

  - run: echo "نسخه: ${{ steps.version.outputs.value }}"

  - if: steps.version.outputs.value != ''
    run: echo "نسخه وجود دارد"
```

**سینتکس `>> "$GITHUB_OUTPUT"`** جای `::set-output` قدیمی را گرفته (که deprecated شد).

#### مثال ۷: متغیر محیطی برای step بعدی

```yaml
steps:
  - run: echo "BUILD_ID=123" >> "$GITHUB_ENV"
  - run: echo "BUILD_ID=$BUILD_ID"
```

#### مثال ۸: `vars` در برابر `secrets`

```yaml
steps:
  - run: echo "Region: ${{ vars.AWS_REGION }}"       # غیرحساس
  - run: echo "Key: ${{ secrets.AWS_KEY }}"          # حساس
```

`vars` در **Settings → Secrets and variables → Actions → Variables** تعریف می‌شود و در لاگ نمایش داده می‌شود. `secrets` ماسک می‌شود.

#### مثال ۹: `format` و `join`

```yaml
- run: echo "${{ format('{0}/{1}', github.repository_owner, github.event.repository.name) }}"
- run: echo "${{ join(needs.*.result, ', ') }}"
```

#### مثال ۱۰: `workflow_dispatch` با input

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: "محیط"
        required: true
        type: choice
        options: [staging, production]
        default: staging
      dry_run:
        description: "فقط شبیه‌سازی"
        type: boolean
        default: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - run: |
          echo "محیط: ${{ inputs.environment }}"
          if [ "${{ inputs.dry_run }}" = "true" ]; then
            echo "حالت شبیه‌سازی"
          else
            ./deploy.sh
          fi
```

### مقایسه و الگوهای مختلف

#### `if` در سطح job در برابر سطح step

| | job | step |
|---|---|---|
| runner | اختصاص نمی‌یابد | اختصاص می‌یابد |
| مصرف دقیقه | صفر | مصرف می‌شود |
| مناسب | شرط بزرگ | شرط کوچک |

**توصیه:** اگر کل job لازم نیست، `if` در سطح job بگذار (صرفه‌جویی دقیقه).

#### `paths-filter` در برابر `paths` در `on`

| | `on.paths` | `paths-filter` |
|---|---|---|
| سطح | کل workflow | هر job |
| انعطاف | محدود | ✅ |
| نیاز به job اضافه | ❌ | ✅ |
| مناسب | workflow تخصصی | مونوریپو |

#### `GITHUB_OUTPUT` در برابر `GITHUB_ENV`

| | `GITHUB_OUTPUT` | `GITHUB_ENV` |
|---|---|---|
| دامنه | همین job (بین stepها) | همین job |
| دسترسی | `steps.<id>.outputs.<name>` | `$NAME` |
| برای | خروجی محاسبه‌شده | متغیر محیطی |

### اشکال شایع

#### ۱. `'false'` (رشته) در `if`

```yaml
✗ if: needs.check.outputs.skip == false
✓ if: needs.check.outputs.skip == 'false'
```

outputها همیشه رشته‌اند. `'false'` truthy است، پس `if: needs.check.outputs.skip` همیشه true می‌شود.

#### ۲. `if` روی job که required check را می‌شکند

job skip می‌شود → check `skipped` → required پاس می‌شود → CI دور زده شد.

#### ۳. `secrets` در `if`

```yaml
✗ if: secrets.MY_KEY != ''
```

secretها در `if` قابل استفاده نیستند (به‌خاطر امنیت). راه‌حل: یک `vars` بگذار که بگوید secret وجود دارد.

#### ۴. `github.event.pull_request` در `push`

اگر workflow روی `push` هم trigger شود، `github.event.pull_request` خالی است و `if` خطا می‌دهد:

```yaml
✗ if: github.event.pull_request.draft == false
✓ if: github.event_name == 'pull_request' && github.event.pull_request.draft == false
```

#### ۵. `needs` بدون `if: always()`

اگر job وابسته شکست بخورد، job بعدی اجرا نمی‌شود — حتی اگر `if: always()` در step باشد. `if: always()` باید در سطح job باشد.

#### ۶. `set-output` قدیمی

```
::set-output name=x::y
```

deprecated شده. از `>> "$GITHUB_OUTPUT"` استفاده کن.

#### ۷. `env` در سطح اشتباه

```yaml
✗ jobs:
    test:
      env:            # درست است
        X: 1
      steps:
        - env:        # درست است
            Y: 2
          run: echo $X $Y
```

هر دو سطح کار می‌کنند، ولی `env` در سطح workflow همه‌ی jobها را می‌گیرد.

### نکات مهم (Tips & Tricks)

- **`dorny/paths-filter`** برای شرط بر اساس فایل‌های تغییر‌یافته.
- **`if: always()` در سطح job** برای پاکسازی.
- **`vars` برای غیرحساس، `secrets` برای حساس.**
- **`fromJSON`** برای خواندن آرایه/آبجکت.
- **`toJSON`** برای دیباگ context:
  ```yaml
  - run: echo '${{ toJSON(github) }}'
  ```
- **`github.event_name`** را همیشه چک کن وقتی روی چند event trigger می‌کنی.

### خلاصه فصل

- `${{ }}` برای expression؛ contextها: `github`، `needs`، `steps`، `matrix`.
- عملگرها: `==`، `&&`، `contains`، `startsWith`، `format`، `fromJSON`.
- **`'false'` رشته، truthy است** — تله‌ی رایج.
- `if` در سطح job، دقیقه صرفه‌جویی می‌کند.
- `paths-filter` برای شرط بر اساس فایل.
- `GITHUB_OUTPUT` جای `set-output` قدیمی.

**سؤالات مروری:**
1. چرا `if: needs.x.outputs.y` ممکن است همیشه true باشد؟
2. چطور فقط وقتی فایل‌های backend عوض شدند تست بک‌اند اجرا شود؟
3. چرا secretها در `if` قابل استفاده نیستند؟

---

## فصل ۴۶: دیباگ workflow — لاگ، debug، اجرای محلی

### مقدمه

workflow که کار نمی‌کند، یکی از ناراحت‌کننده‌ترین تجربه‌های توسعه است: هر تست ۲ دقیقه طول می‌کشد، لاگ طولانی است، و خطا واضح نیست.

**چرا مهم است؟** سرعت دیباگ CI مستقیماً روی سرعت تیم اثر دارد. کسی که ابزارها را بلد است، در ۵ دقیقه حل می‌کند؛ کسی که نه، در ۲ ساعت.

**کاربرد عملی:** این فصل یک راهنمای گام‌به‌گام دیباگ است.

### مفاهیم اساسی

#### سطوح دیباگ

| سطح | چطور | چه چیزی نشان می‌دهد |
|---|---|---|
| **لاگ عادی** | UI | خروجی stepها |
| **`--log-failed`** | `gh run view --log-failed` | فقط شکست |
| **Debug logging** | secret `ACTIONS_STEP_DEBUG=true` | جزئیات step |
| **Runner debug** | secret `ACTIONS_RUNNER_DEBUG=true` | جزئیات runner |
| **SSH به runner** | `tmate` | دسترسی تعاملی |
| **اجرای محلی** | `act` | تکرار سریع |

#### `ACTIONS_STEP_DEBUG`

1. **Settings → Secrets and variables → Actions → New repository secret**
2. Name: `ACTIONS_STEP_DEBUG`، Value: `true`
3. workflow دوباره اجرا کن

حالا لاگ شامل:
- contextها
- expressionهای ارزیابی‌شده
- جزئیات هر step

**هشدار:** لاگ خیلی طولانی می‌شود. بعد از دیباگ، secret را پاک کن.

### مثال‌های عملی

#### مثال ۱: خواندن لاگ شکست

```bash
# لیست اجراها
gh run list --limit 10

# جزئیات
gh run view <run-id>

# فقط بخش شکست‌خورده
gh run view <run-id> --log-failed

# لاگ کامل
gh run view <run-id> --log > full.log

# جست‌وجو در لاگ
gh run view <run-id> --log | grep -i "error"
```

#### مثال ۲: دیباگ context

```yaml
- name: نمایش context
  run: |
    echo "event: ${{ github.event_name }}"
    echo "ref: ${{ github.ref }}"
    echo "sha: ${{ github.sha }}"
    echo "actor: ${{ github.actor }}"
    echo "workspace: ${{ github.workspace }}"

- name: context کامل (JSON)
  run: echo '${{ toJSON(github) }}' | head -100

- name: همه‌ی env
  run: env | sort
```

**`toJSON(github)`** بهترین ابزار برای فهمیدن اینکه چه داده‌ای در دسترس است.

#### مثال ۳: tmate — SSH به runner

```yaml
- name: Setup tmate
  uses: mxschmitt/action-tmate@v3
  if: ${{ failure() }}
  timeout-minutes: 15
```

وقتی job شکست می‌خورد، یک آدرس SSH در لاگ چاپ می‌شود. می‌توانی وصل شوی و محیط را ببینی.

**هشدار امنیتی:** هر کسی که لاگ را ببیند می‌تواند وصل شود. `limit-access-to-actor: true` بگذار:

```yaml
- uses: mxschmitt/action-tmate@v3
  with:
    limit-access-to-actor: true
```

#### مثال ۴: اجرای محلی با `act`

```bash
# نصب
brew install act            # macOS
# یا
curl https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash

# لیست workflowها
act -l

# اجرای pull_request
act pull_request

# با secrets
act -s GITHUB_TOKEN=ghp_xxx

# یک job خاص
act -j test

# با فایل secrets
act --secret-file .secrets
```

**محدودیت‌های `act`:**

| محدودیت | توضیح |
|---|---|
| macOS runner | شبیه‌سازی نمی‌شود (Docker روی Linux) |
| `services` | محدود |
| Cache | متفاوت |
| Secrets | باید دستی بدهی |
| Actions خاص | بعضی کار نمی‌کنند |

**توصیه:** `act` برای دیباگ سریع منطق، نه برای تأیید نهایی.

#### مثال ۵: بررسی syntax با actionlint

```bash
# نصب
brew install actionlint
# یا
go install github.com/rhysd/actionlint/cmd/actionlint@latest

# اجرا
actionlint

# فقط یک فایل
actionlint .github/workflows/ci.yml
```

`actionlint` می‌گیرد:
- خطای YAML
- context نامعتبر
- `if` با سینتکس غلط
- `needs` به job ناموجود
- مشکلات shell (با shellcheck)

**در pre-commit بگذار:**

```yaml
# .pre-commit-config.yaml
- repo: https://github.com/rhysd/actionlint
  rev: v1.7.7
  hooks:
    - id: actionlint
```

#### مثال ۶: پیام خطای مفید

```yaml
- name: تست
  run: |
    set -euo pipefail
    if ! npm test; then
      echo "::error::تست‌ها شکست خوردند. برای دیباگ: npm test -- --verbose"
      exit 1
    fi
```

**دستورهای annotation:**

```yaml
- run: echo "::error file=src/app.js,line=42::متغیر تعریف نشده"
- run: echo "::warning::این step deprecated است"
- run: echo "::notice::build در ۲ دقیقه تمام شد"
- run: echo "::group::بخش اول"
- run: echo "بخش اول"
- run: echo "::endgroup::"
```

`::group::` لاگ را جمع‌وجور می‌کند — برای لاگ طولانی عالی است.

#### مثال ۷: بررسی یک job خاص با gh

```bash
# jobهای یک run
gh run view <run-id> --json jobs --jq '.jobs[] | {name, conclusion}'

# لاگ یک job
gh run view <run-id> --job <job-id> --log

# دانلود artifact
gh run download <run-id> -n coverage
```

#### مثال ۸: مشکل رایج — `command not found`

```yaml
- run: npm ci
# bash: npm: command not found
```

`setup-node` را فراموش کردی. یا در job اشتباه هستی (هر job محیط جدا دارد).

#### مثال ۹: مشکل رایج — permission denied

```yaml
- run: ./scripts/deploy.sh
# bash: ./scripts/deploy.sh: Permission denied
```

```yaml
- run: chmod +x scripts/deploy.sh && ./scripts/deploy.sh
```

**نکته:** مجوز فایل در Git ذخیره می‌شود (`git update-index --chmod=+x`). اگر در ویندوز ساختی، ممکن است مجوز نداشته باشد:

```bash
git update-index --chmod=+x scripts/deploy.sh
```

#### مثال ۱۰: مشکل رایج — exit code

```yaml
- run: |
    npm test
    echo "تست تمام شد"    # ← اگر npm test شکست بخورد، این اجرا نمی‌شود
```

GitHub Actions هر step را با `set -e` اجرا می‌کند (bash). اگر می‌خواهی ادامه دهد:

```yaml
- run: |
    npm test || true
    echo "ادامه"
```

### مقایسه و الگوهای مختلف

#### `act` در برابر push واقعی

| | `act` | push |
|---|---|---|
| سرعت | ثانیه | دقیقه |
| دقت | ۷۰٪ | ۱۰۰٪ |
| هزینه | صفر | دقیقه |
| مناسب | دیباگ سریع | تأیید نهایی |

#### tmate در برابر لاگ

| | tmate | لاگ |
|---|---|---|
| تعاملی | ✅ | ❌ |
| امنیت | ⚠️ | ✅ |
| سرعت | متوسط | سریع |
| مناسب | مشکلات محیطی | خطاهای منطقی |

#### سطوح debug

| Secret | چه می‌دهد | حجم لاگ |
|---|---|---|
| `ACTIONS_STEP_DEBUG` | expressionها، context | ۲-۳× |
| `ACTIONS_RUNNER_DEBUG` | جزئیات runner | ۵× |
| هر دو | همه‌چیز | ۱۰× |

### اشکال شایع

#### ۱. لاگ طولانی و غیرقابل خواندن

از `::group::` استفاده کن:

```yaml
- run: |
    echo "::group::نصب وابستگی‌ها"
    npm ci
    echo "::endgroup::"
    echo "::group::تست"
    npm test
    echo "::endgroup::"
```

#### ۲. خطا در step اول ولی لاگ نشان نمی‌دهد

`ACTIONS_STEP_DEBUG=true` را روشن کن.

#### ۳. `act` کار می‌کند ولی GitHub نه

معمولاً: تفاوت OS، secret، یا action خاصی که `act` پشتیبانی نمی‌کند.

#### ۴. workflow اجرا نمی‌شود

- `on:` شرط را برآورده نمی‌کند (branch، path، event)
- فایل در `.github/workflows/` نیست
- YAML نامعتبر است (GitHub خطا نشان می‌دهد)
- workflow در branch دیگری است (workflow از branch پیش‌فرض trigger می‌شود برای بعضی eventها)

#### ۵. `secrets` خالی است

- در fork (طراحی)
- secret تعریف نشده
- در `if` استفاده شده (ممنوع)

#### ۶. `git` در runner کار نمی‌کند

`actions/checkout` را فراموش کردی. بدون آن، هیچ کدی نیست.

#### ۷. job گیر کرده

`timeout-minutes` بگذار. job بدون timeout تا ۶ ساعت می‌ماند.

### نکات مهم (Tips & Tricks)

- **`actionlint` در pre-commit** — ۸۰٪ خطاها را قبل از push می‌گیرد.
- **`gh run view --log-failed`** — سریع‌ترین راه دیدن خطا.
- **`toJSON(github)`** — بهترین ابزار کشف context.
- **`::group::`** — لاگ خواندنی.
- **`timeout-minutes`** — جلوگیری از job گیرکرده.
- **`act`** برای دیباگ سریع منطق.
- **`tmate`** با `limit-access-to-actor: true` برای مشکلات محیطی.

### خلاصه فصل

- سطوح دیباگ: لاگ → `--log-failed` → `ACTIONS_STEP_DEBUG` → tmate → `act`.
- `actionlint` قبل از push.
- `toJSON(github)` برای کشف داده.
- `::group::` برای لاگ خواندنی.
- `act` ۷۰٪ دقیق، برای دیباگ سریع.
- `timeout-minutes` همیشه بگذار.

**سؤالات مروری:**
1. چطور فقط بخش شکست‌خورده‌ی لاگ را ببینی؟
2. `ACTIONS_STEP_DEBUG` چه چیزی اضافه می‌کند؟
3. سه محدودیت `act` چیست؟

---

**بعدی:** [`11-Actions-Advanced.md`](11-Actions-Advanced.md)

---

## منابع و مراجع

- GitHub Docs: Understanding GitHub Actions — https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions
- GitHub Docs: Workflow syntax — https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions
- GitHub Docs: Expressions — https://docs.github.com/en/actions/learn-github-actions/expressions
- GitHub Docs: Contexts — https://docs.github.com/en/actions/learn-github-actions/contexts
- GitHub Docs: Matrix strategy — https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs
- `actionlint` — https://github.com/rhysd/actionlint
- `act` (اجرای محلی) — https://github.com/nektos/act
- `dorny/paths-filter` — https://github.com/dorny/paths-filter
