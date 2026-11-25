# بخش ۱۱: GitHub Actions — پیشرفته

**مدت یادگیری**: ۵ تا ۶ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: بخش ۱۰ (workflow، matrix، context)

---

## فهرست محتوا

- [فصل ۴۷: Caching — کش وابستگی‌ها و build](#فصل-۴۷)
- [فصل ۴۸: Artifacts — انتقال داده و نگهداری خروجی](#فصل-۴۸)
- [فصل ۴۹: Reusable workflows و composite actions](#فصل-۴۹)
- [فصل ۵۰: Environment، approval، concurrency و timeout](#فصل-۵۰)
- [فصل ۵۱: Self-hosted runner — نصب، امنیت، مقیاس](#فصل-۵۱)

---

## فصل ۴۷: Caching — کش وابستگی‌ها و build

### مقدمه

بدون کش، هر اجرای CI از صفر `npm ci` می‌زند: ۴۵ ثانیه تا ۲ دقیقه تلف. با کش، ۵ تا ۱۵ ثانیه.

**چرا مهم است؟** کش بزرگ‌ترین برد سریع در CI است. یک تیم ۱۰ نفره با ۲۰ اجرا در روز، سالانه صدها ساعت صرفه‌جویی می‌کند. و CI سریع‌تر = بازخورد سریع‌تر = merge سریع‌تر.

**کاربرد عملی:** هر workflow واقعی باید کش داشته باشد. این فصل الگوهای درست و تله‌ها را می‌دهد.

### مفاهیم اساسی

#### دو نوع کش

| نوع | ابزار | برای |
|---|---|---|
| **کش وابستگی** | `setup-node cache`، `actions/cache` | node_modules، pip، go mod |
| **کش build** | `actions/cache`، `docker/build-push-action cache` | .next/cache، target/، Docker layer |

#### چطور کار می‌کند

```
۱. workflow شروع می‌شود
۲. Git به‌دنبال cache با key مشخص می‌گردد
۳. اگر پیدا شد → restore (چند ثانیه)
۴. اگر نه → miss
۵. در پایان job → save با همان key (اگر miss بود)
```

**نکته‌ی مهم:** cache فقط **یک بار** با یک key ذخیره می‌شود. اگر key موجود باشد، ذخیره نمی‌شود (immutable).

#### `key` — مهم‌ترین تصمیم

```
key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
```

- `runner.os` → کش بین OSها قاطی نمی‌شود
- `hashFiles('**/package-lock.json')` → با تغییر lock، key عوض می‌شود
- `restore-keys` → اگر key دقیق نبود، نزدیک‌ترین را بگیر

```yaml
key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
restore-keys: |
  ${{ runner.os }}-node-
```

#### محدودیت‌ها

| محدودیت | مقدار |
|---|---|
| حجم کل cache در مخزن | ۱۰GB |
| عمر | ۷ روز بدون دسترسی |
| کش بین branchها | branch پیش‌فرض قابل دسترس است |
| کش بین forkها | ❌ |
| نوشتن از fork | ❌ |

### مثال‌های عملی

#### مثال ۱: کش ساده با `setup-node`

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: "22"
    cache: "npm"
```

تمام. `setup-node` خودش `package-lock.json` را هش می‌کند.

**پیش‌نیاز:** `package-lock.json` باید وجود داشته باشد و در مسیر ریشه (یا `cache-dependency-path` بده).

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: "22"
    cache: "npm"
    cache-dependency-path: |
      package-lock.json
      frontend/package-lock.json
```

#### مثال ۲: کش Python

```yaml
- uses: actions/setup-python@v5
  with:
    python-version: "3.12"
    cache: "pip"
    cache-dependency-path: requirements*.txt
```

#### مثال ۳: کش Go

```yaml
- uses: actions/setup-go@v5
  with:
    go-version: "1.23"
    cache: true       # خودکار go.sum را هش می‌کند
```

#### مثال ۴: کش دستی با `actions/cache`

```yaml
- name: کش node_modules
  uses: actions/cache@v4
  id: npm-cache
  with:
    path: |
      ~/.npm
      node_modules
    key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-npm-

- name: نصب
  if: steps.npm-cache.outputs.cache-hit != 'true'
  run: npm ci

- name: نصب (cache hit)
  if: steps.npm-cache.outputs.cache-hit == 'true'
  run: npm ci --prefer-offline --no-audit
```

**نکته:** حتی با cache hit، `npm ci` را اجرا کن (سریع است و node_modules را درست می‌سازد). فقط `--prefer-offline` بگذار.

#### مثال ۵: کش build (Next.js)

```yaml
- name: کش Next.js
  uses: actions/cache@v4
  with:
    path: ${{ github.workspace }}/.next/cache
    key: ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-${{ hashFiles('**/*.js', '**/*.jsx', '**/*.ts', '**/*.tsx') }}
    restore-keys: |
      ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-
```

**چرا دو هش؟** lock (وابستگی) + کد. اگر فقط کد عوض شد، cache نزدیک‌ترین را می‌گیرد.

#### مثال ۶: کش Docker layer

```yaml
- uses: docker/setup-buildx-action@v3

- uses: docker/build-push-action@v6
  with:
    context: .
    push: true
    tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

`type=gha` از cache داخلی Actions استفاده می‌کند — بدون نیاز به registry جدا.

#### مثال ۷: کش بین jobها

Cache بین jobها هم کار می‌کند:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: npm }
      - run: npm ci
      - run: npm run build
      - uses: actions/cache/save@v4
        with:
          path: dist
          key: dist-${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/cache/restore@v4
        with:
          path: dist
          key: dist-${{ github.sha }}
      - run: ./deploy.sh dist/
```

**نکته:** برای انتقال داده، **artifact** معمولاً بهتر است (فصل ۴۸). Cache برای "داده‌ی قابل استفاده‌ی مجدد" است، artifact برای "خروجی این اجرا".

#### مثال ۸: پاک کردن کش

```bash
# لیست کش‌ها
gh api repos/OWNER/REPO/actions/caches --jq '.actions_caches[] | "\(.id) \(.key) \(.size_in_bytes)"'

# حذف یک کش
gh api -X DELETE repos/OWNER/REPO/actions/caches/<cache-id>

# حذف همه
gh api -X DELETE repos/OWNER/REPO/actions/caches
```

یا در UI: **Actions → Caches**.

### مقایسه و الگوهای مختلف

#### `setup-*` cache در برابر `actions/cache`

| | `setup-node cache` | `actions/cache` |
|---|---|---|
| سادگی | ✅ | متوسط |
| کنترل | محدود | کامل |
| برای | وابستگی استاندارد | کش سفارشی |
| توصیه | ✅ اول | وقتی نیاز داری |

#### Cache در برابر Artifact

| | Cache | Artifact |
|---|---|---|
| هدف | سرعت (داده‌ی تکراری) | خروجی (نتیجه‌ی build) |
| عمر | ۷ روز بی‌استفاده | ۹۰ روز (قابل تنظیم) |
| دسترسی | فقط workflow | دانلود دستی |
| immutable | ✅ | ❌ (می‌شود overwrite) |
| مناسب | node_modules، .next/cache | dist/، گزارش تست |

#### کش در برابر `npm ci --prefer-offline`

| | کش کامل | `--prefer-offline` |
|---|---|---|
| سرعت | ~۱۰s | ~۳۰s |
| پیچیدگی | متوسط | صفر |
| مناسب | CI فعال | پروژه‌ی کوچک |

### اشکال شایع

#### ۱. کش هرگز hit نمی‌شود

- `key` هر بار عوض می‌شود (مثلاً `github.sha` در key)
- `path` اشتباه است
- cache از branch دیگر و branch پیش‌فرض نیست

**چک کن:** در لاگ، `Cache restored from key: ...` یا `Cache not found for input keys: ...`.

#### ۲. کش کهنه و نتایج غلط

```
✗ key: ${{ runner.os }}-npm
```

بدون هش lock، کش هرگز عوض نمی‌شود و وابستگی‌های قدیمی می‌مانند. **همیشه `hashFiles` بگذار.**

#### ۳. `node_modules` را کش کردم و عجیب رفتار می‌کند

`node_modules` شامل فایل‌های باینری وابسته به OS است. اگر بین OSها کش کنی، خراب می‌شود. `runner.os` را در key بگذار.

**بهتر:** فقط `~/.npm` را کش کن (cache پکیج‌ها)، نه `node_modules`.

#### ۴. کش بزرگ‌تر از ۱۰GB

مخزن به سقف می‌رسد و کش جدید ذخیره نمی‌شود. کش‌های قدیمی را پاک کن، یا فقط cache پکیج را کش کن (نه build).

#### ۵. cache-hit همیشه false

```yaml
✗ if: steps.cache.outputs.cache-hit
```

`cache-hit` رشته است (`'true'`/`'false'`). `'false'` truthy است! استفاده کن:

```yaml
✓ if: steps.cache.outputs.cache-hit == 'true'
```

#### ۶. کش در fork کار نمی‌کند

forkها به cache مخزن base دسترسی ندارند. طراحی امنیتی.

#### ۷. `actions/cache` در `post` ذخیره می‌کند

cache در پایان job ذخیره می‌شود. اگر job شکست بخورد، ذخیره نمی‌شود (مگر `save-always`).

### نکات مهم (Tips & Tricks)

- **`hashFiles` در key** — بدون آن، کش کهنه می‌شود.
- **`runner.os` در key** — جلوگیری از قاطی شدن OS.
- **`restore-keys`** — کش نزدیک را بگیر اگر دقیق نبود.
- **فقط `~/.npm` را کش کن، نه `node_modules`** — سبک‌تر و مطمئن‌تر.
- **`type=gha` برای Docker** — ساده‌ترین کش layer.
- **کش را ماهانه پاک کن** — از سقف ۱۰GB جلوگیری کن.

### خلاصه فصل

- کش = بزرگ‌ترین برد سرعت در CI.
- `setup-node cache: npm` ساده‌ترین راه.
- `key` باید شامل `runner.os` و `hashFiles(lock)` باشد.
- **`cache-hit` رشته است** — با `== 'true'` مقایسه کن.
- Cache برای داده‌ی تکراری، Artifact برای خروجی.
- سقف ۱۰GB در هر مخزن.

**سؤالات مروری:**
1. چرا `key` بدون `hashFiles` خطرناک است؟
2. تفاوت cache و artifact چیست؟
3. چرا کش `node_modules` بین OSها مشکل‌ساز است؟

---

## فصل ۴۸: Artifacts — انتقال داده و نگهداری خروجی

### مقدمه

Artifact فایلی است که یک job تولید می‌کند و بعداً (در job دیگر یا بعد از اتمام workflow) قابل دسترسی است. بر خلاف cache، artifact برای **نگهداری** است نه سرعت.

**چرا مهم است؟** بدون artifact، خروجی build گم می‌شود، گزارش تست قابل مشاهده نیست، و deploy نمی‌تواند از build استفاده کند.

**کاربرد عملی:** سه الگوی اصلی: انتقال بین jobها، نگهداری گزارش، و انتشار در release.

### مفاهیم اساسی

#### Artifact در برابر Cache

| | Artifact | Cache |
|---|---|---|
| هدف | خروجی، نگهداری | سرعت |
| عمر | ۹۰ روز (پیش‌فرض) | ۷ روز بی‌استفاده |
| دسترسی | دانلود دستی + API | فقط workflow |
| حجم | ۱۰GB (Free) | ۱۰GB کل مخزن |
| overwrite | ✅ (با نام یکسان) | ❌ |

#### نسخه‌ی Actions

| Action | نسخه | نکته |
|---|---|---|
| `actions/upload-artifact` | **v4** | v3 deprecated (ژانویه ۲۰۲۵) |
| `actions/download-artifact` | **v4** | ناسازگار با v3 |

**مهم:** v3 دیگر کار نمی‌کند. اگر workflow قدیمی داری، ارتقا بده.

#### محدودیت‌ها (v4)

| محدودیت | مقدار |
|---|---|
| حجم کل | ۱۰GB (Free)، ۵۰۰GB (Enterprise) |
| تعداد فایل | ۱۰٬۰۰۰ در هر artifact |
| نام | یکتا در هر workflow run (v4) |
| فشرده‌سازی | خودکار (zip) |

### مثال‌های عملی

#### مثال ۱: آپلود و دانلود پایه

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: npm }
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/
          retention-days: 7

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/
      - run: ls -la dist/
      - run: ./deploy.sh dist/
```

#### مثال ۲: آپلود چند مسیر

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: |
      coverage/
      reports/junit.xml
      reports/playwright/
    retention-days: 14
    if-no-files-found: error     # یا warn / ignore
```

#### مثال ۳: آپلود حتی در شکست

```yaml
- name: تست
  run: npm test

- name: آپلود نتایج (همیشه)
  if: always()
  uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: |
      coverage/
      reports/
```

**چرا `if: always()`؟** وقتی تست شکست می‌خورد، بیشترین نیاز به گزارش را داری.

#### مثال ۴: artifact در matrix — نام یکتا

```yaml
jobs:
  test:
    strategy:
      matrix:
        node: [20, 22]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: "${{ matrix.node }}", cache: npm }
      - run: npm ci
      - run: npm test -- --coverage
      - uses: actions/upload-artifact@v4
        with:
          name: coverage-node-${{ matrix.node }}    # ← نام یکتا
          path: coverage/
```

**تله‌ی v4:** نام artifact باید در یک run یکتا باشد. اگر دو job با نام یکسان آپلود کنند، خطا می‌دهد.

#### مثال ۵: artifact برای گزارش HTML

```yaml
- name: تست
  run: npx playwright test

- uses: actions/upload-artifact@v4
  if: always()
  with:
    name: playwright-report
    path: playwright-report/
    retention-days: 30
```

بعد از اجرا، از UI دانلود کن و `index.html` را باز کن. (GitHub Pages هم می‌تواند میزبانی کند — فصل ۸۰.)

#### مثال ۶: دانلود همه‌ی artifactها

```yaml
- uses: actions/download-artifact@v4
  with:
    path: all-artifacts
    # بدون name → همه را دانلود می‌کند، هر کدام در زیرپوشه‌ای با نامش
```

#### مثال ۷: artifact به release

```yaml
jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
      - run: ./build.sh
      - uses: softprops/action-gh-release@v2
        with:
          files: |
            dist/app-linux
            dist/app-macos
            dist/app.exe
```

این فایل‌ها به release الصاق می‌شوند (نه artifact) — قابل دانلود دائمی.

#### مثال ۸: دانلود با gh

```bash
# لیست artifactهای یک run
gh run view <run-id> --json artifacts --jq '.artifacts[] | {name, size_in_bytes}'

# دانلود همه
gh run download <run-id>

# دانلود یکی
gh run download <run-id> -n coverage

# به مسیر مشخص
gh run download <run-id> -n dist -D ./output
```

#### مثال ۹: artifact برای attestation

```yaml
- uses: actions/attest-build-provenance@v2
  with:
    subject-path: dist/app
```

این یک attestation (گواهی منشأ) برای artifact می‌سازد — فصل ۷۵.

### مقایسه و الگوهای مختلف

#### Artifact در برابر Release asset

| | Artifact | Release asset |
|---|---|---|
| عمر | ۹۰ روز | دائمی |
| دسترسی | نیاز به لاگین (خصوصی) | عمومی |
| مناسب | خروجی CI | نسخه‌ی منتشرشده |
| حجم | ۱۰GB | ۲GB per file |

#### Artifact در برابر Cache برای انتقال

| | Artifact | Cache |
|---|---|---|
| بین job | ✅ | ✅ |
| بین workflow | ❌ (نیاز به API) | ✅ |
| overwrite | ✅ | ❌ |
| مناسب | خروجی build | وابستگی |

**قاعده:** داده‌ای که job بعدی لازم دارد و **مخصوص این اجراست** → artifact. داده‌ای که **تکرار می‌شود** → cache.

#### `if-no-files-found`

| مقدار | رفتار |
|---|---|
| `warn` (پیش‌فرض) | هشدار، ادامه |
| `error` | شکست job |
| `ignore` | بی‌صدا |

**توصیه:** `error` برای build artifact (اگر نبود، مشکلی هست).

### اشکال شایع

#### ۱. `upload-artifact@v3` کار نمی‌کند

از ژانویه ۲۰۲۵، v3 حذف شد:

```
This request has been automatically failed because it uses a deprecated version
of actions/upload-artifact: v3
```

به `@v4` ارتقا بده. توجه: `download-artifact` هم باید v4 باشد.

#### ۲. نام تکراری در matrix

```
Error: Conflict: an artifact with this name already exists on the workflow run
```

v4 اجازه نمی‌دهد دو artifact با یک نام. از `${{ matrix.x }}` در نام استفاده کن.

#### ۳. مسیر اشتباه

```yaml
✗ path: ./dist        # اگر dist وجود نداشته باشد، warn می‌دهد
✓ path: dist/         # با اسلش
✓ if-no-files-found: error
```

#### ۴. فایل‌های مخفی آپلود نمی‌شوند

به‌طور پیش‌فرض، فایل‌های شروع‌شده با `.` (مثل `.env`) آپلود نمی‌شوند:

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: config
    path: .config/
    include-hidden-files: true    # ← لازم
```

#### ۵. artifact خیلی بزرگ

```
Artifact size exceeded the limit
```

- فشرده کن (`tar.gz`)
- فایل‌های غیرضروری را حذف کن (`node_modules` را آپلود نکن)
- از `retention-days` کوتاه استفاده کن

#### ۶. download-artifact همه را می‌گیرد ولی ساختار عجیب است

```yaml
- uses: actions/download-artifact@v4
  with:
    path: artifacts
```

ساختار: `artifacts/<artifact-name>/<files>`.

اگر یک artifact خاص می‌خواهی، `name:` بده.

### نکات مهم (Tips & Tricks)

- **`if: always()`** برای گزارش تست — بیشترین نیاز در شکست.
- **نام یکتا در matrix** — `${{ matrix.x }}` در نام.
- **`retention-days`** کوتاه برای artifactهای بزرگ.
- **`include-hidden-files: true`** برای فایل‌های مخفی.
- **`if-no-files-found: error`** برای build artifact.
- **Release asset برای نسخه‌ی نهایی، artifact برای CI.**

### خلاصه فصل

- Artifact برای نگهداری خروجی؛ cache برای سرعت.
- **v4 اجباری است** (v3 حذف شده).
- نام artifact در یک run باید یکتا باشد.
- `if: always()` برای گزارش تست.
- `include-hidden-files` برای فایل‌های `.`دار.
- Release asset برای دانلود دائمی.

**سؤالات مروری:**
1. تفاوت artifact و cache در هدف چیست؟
2. چرا در matrix باید نام artifact یکتا باشد؟
3. چرا `if: always()` برای آپلود گزارش تست مهم است؟

---

## فصل ۴۹: Reusable workflows و composite actions

### مقدمه

وقتی ۵ مخزن داری و همه به یک CI یکسان نیاز دارند، کپی کردن ۵ فایل YAML یعنی ۵ بار نگهداری. GitHub دو راه برای حل این می‌دهد: **reusable workflow** (کل workflow) و **composite action** (مجموعه‌ی stepها).

**چرا مهم است؟** DRY در CI یعنی تغییر یک جا، اعمال در همه. و برای سازمان، استانداردسازی امنیتی.

**کاربرد عملی:** اگر بیش از دو مخزن با CI مشابه داری، این فصل صرفه‌جویی بزرگی است.

### مفاهیم اساسی

#### سه نوع اشتراک‌گذاری

| نوع | چیست | سطح |
|---|---|---|
| **Reusable workflow** | کل workflow با jobها | `on: workflow_call` |
| **Composite action** | مجموعه‌ی stepها | `action.yml` با `runs.using: composite` |
| **JavaScript action** | کد JS | `action.yml` با `runs.using: node20` |

#### تفاوت کلیدی

| | Reusable workflow | Composite action |
|---|---|---|
| چه چیزی را می‌گیرد | کل workflow (jobها) | stepها |
| runner | خودش تعیین می‌کند | در workflow میزبان اجرا می‌شود |
| secrets | باید صریح پاس بدهی | در context میزبان است |
| مناسب | CI استاندارد | منطق تکراری داخل job |

#### محدودیت‌های reusable workflow

| محدودیت | مقدار |
|---|---|
| عمق تودرتویی | ۴ سطح |
| تعداد فراخوانی | ۲۰ در هر workflow |
| دسترسی | همان org یا public |
| `env` از caller | ❌ (باید `with` بدهی) |

### مثال‌های عملی

#### مثال ۱: reusable workflow برای CI

`.github/workflows/reusable-ci.yml` (در مخزن `.github` سازمان):

```yaml
name: Reusable CI

on:
  workflow_call:
    inputs:
      node-version:
        type: string
        default: "22"
      working-directory:
        type: string
        default: "."
      run-e2e:
        type: boolean
        default: false
    secrets:
      NPM_TOKEN:
        required: false
    outputs:
      coverage:
        description: "درصد پوشش"
        value: ${{ jobs.test.outputs.coverage }}

jobs:
  test:
    runs-on: ubuntu-24.04
    timeout-minutes: 15
    outputs:
      coverage: ${{ steps.cov.outputs.value }}
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: npm
          cache-dependency-path: ${{ inputs.working-directory }}/package-lock.json

      - run: npm ci
        env:
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}

      - run: npm run lint
      - run: npm run typecheck

      - id: cov
        run: |
          npm test -- --coverage
          PCT=$(node -p "require('./coverage/coverage-summary.json').total.lines.pct")
          echo "value=$PCT" >> "$GITHUB_OUTPUT"

      - if: inputs.run-e2e
        run: npx playwright test

      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: coverage
          path: ${{ inputs.working-directory }}/coverage/
```

استفاده در مخزن دیگر:

```yaml
name: CI

on: [push, pull_request]

jobs:
  ci:
    uses: myorg/.github/.github/workflows/reusable-ci.yml@main
    with:
      node-version: "22"
      run-e2e: true
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

**نکته:** مسیر `myorg/.github/.github/workflows/...@main` — مخزن `.github` سازمان، پوشه‌ی `.github/workflows`، و `@ref`.

#### مثال ۲: reusable workflow برای deploy

```yaml
name: Deploy

on:
  workflow_call:
    inputs:
      environment:
        type: string
        required: true
      image-tag:
        type: string
        required: true
    secrets:
      DEPLOY_KEY:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-24.04
    environment: ${{ inputs.environment }}
    steps:
      - uses: actions/checkout@v4
      - name: Deploy
        run: ./deploy.sh --env "${{ inputs.environment }}" --tag "${{ inputs.image-tag }}"
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
```

#### مثال ۳: composite action

`.github/actions/setup-project/action.yml`:

```yaml
name: "Setup Project"
description: "نصب Node، وابستگی‌ها و ابزارهای پروژه"

inputs:
  node-version:
    description: "نسخه Node"
    required: false
    default: "22"
  install-args:
    description: "آرگومان‌های نصب"
    required: false
    default: ""

outputs:
  cache-hit:
    description: "آیا کش hit شد"
    value: ${{ steps.cache.outputs.cache-hit }}

runs:
  using: "composite"
  steps:
    - name: Setup Node
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
        cache: npm

    - name: نصب وابستگی‌ها
      shell: bash
      run: npm ci ${{ inputs.install-args }}

    - name: نصب ابزارهای CLI
      shell: bash
      run: |
        npm install -g @commitlint/cli @commitlint/config-conventional
```

استفاده:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: ./.github/actions/setup-project
        with:
          node-version: "22"
      - run: npm test
```

**نکته‌ی حیاتی:** در composite action، هر step **باید `shell:` داشته باشد** (برخلاف workflow معمولی که `bash` پیش‌فرض است).

#### مثال ۴: composite action برای انتشار

`.github/actions/publish/action.yml`:

```yaml
name: "Publish to npm"
description: "انتشار پکیج با بررسی نسخه"

inputs:
  tag:
    description: "dist-tag"
    default: "latest"
  token:
    required: true

runs:
  using: "composite"
  steps:
    - name: بررسی نسخه
      shell: bash
      run: |
        NAME=$(node -p "require('./package.json').name")
        VERSION=$(node -p "require('./package.json').version")
        if npm view "$NAME@$VERSION" version 2>/dev/null; then
          echo "::error::نسخه $VERSION قبلاً منتشر شده"
          exit 1
        fi
        echo "منتشر می‌شود: $NAME@$VERSION"

    - name: publish
      shell: bash
      run: npm publish --tag "${{ inputs.tag }}" --provenance
      env:
        NODE_AUTH_TOKEN: ${{ inputs.token }}
```

#### مثال ۵: فراخوانی reusable workflow در matrix

```yaml
jobs:
  ci:
    strategy:
      matrix:
        node: [20, 22]
    uses: myorg/.github/.github/workflows/reusable-ci.yml@main
    with:
      node-version: ${{ matrix.node }}
```

#### مثال ۶: `secrets: inherit`

```yaml
jobs:
  ci:
    uses: myorg/.github/.github/workflows/reusable-ci.yml@main
    secrets: inherit    # همه‌ی secretهای caller پاس داده می‌شوند
```

**هشدار:** `inherit` راحت است ولی اصل کمترین دسترسی را نقض می‌کند. برای workflow در org دیگر، استفاده نکن.

#### مثال ۷: تست reusable workflow

```yaml
name: Test reusable

on:
  push:
    paths:
      - '.github/workflows/reusable-ci.yml'

jobs:
  test:
    uses: ./.github/workflows/reusable-ci.yml
    with:
      node-version: "22"
```

#### مثال ۸: نسخه‌بندی

```
✗ uses: myorg/.github/.github/workflows/ci.yml@main
```

با `@main`، هر تغییری فوراً روی همه‌ی مخزن‌ها اثر می‌گذارد — که می‌تواند همه را بشکند.

```
✓ uses: myorg/.github/.github/workflows/ci.yml@v1
```

با tag، تغییر کنترل‌شده است. و `@<sha>` برای حداکثر امنیت (فصل ۵۵).

### مقایسه و الگوهای مختلف

#### Reusable workflow در برابر Composite action

| | Reusable workflow | Composite |
|---|---|---|
| سطح | کل workflow | stepها |
| runner | خودش | میزبان |
| چند job | ✅ | ❌ |
| `if` روی job | ✅ | ❌ |
| مناسب | CI استاندارد | setup تکراری |
| پیچیدگی | بیشتر | کمتر |

**قاعده:** اگر منطق شامل jobهای موازی و runner است → reusable workflow. اگر فقط چند step تکراری است → composite.

#### `@main` در برابر `@v1` در برابر `@sha`

| | `@main` | `@v1` | `@sha` |
|---|---|---|---|
| به‌روزرسانی | خودکار | دستی | دستی |
| امنیت | ❌ | متوسط | ✅ |
| مناسب | توسعه | تولید | تولید حساس |

#### `secrets: inherit` در برابر صریح

| | inherit | صریح |
|---|---|---|
| سادگی | ✅ | ❌ |
| امنیت | ❌ | ✅ |
| مناسب | مخزن خودت | org دیگر |

### اشکال شایع

#### ۱. `shell:` را در composite فراموش کردم

```
Error: Required property is missing: shell
```

هر `run` در composite باید `shell:` داشته باشد.

#### ۲. مسیر اشتباه reusable workflow

```
✗ uses: myorg/.github/workflows/ci.yml@main
✓ uses: myorg/.github/.github/workflows/ci.yml@main
```

مخزن `.github`، پوشه‌ی `.github/workflows`.

#### ۳. `env` از caller نمی‌رسد

`env` در سطح workflow caller به reusable workflow منتقل **نمی‌شود**. باید `with` بدهی و در reusable، `env` بسازی.

#### ۴. `secrets` پاس ندادی

```
Error: Secret NPM_TOKEN is required but not provided
```

`secrets:` را در caller اضافه کن.

#### ۵. عمق تودرتویی

reusable workflow که reusable workflow صدا بزند، حداکثر ۴ سطح. بیشتر از آن خطا می‌دهد.

#### ۶. `@main` و شکستن همه

یک تغییر در reusable workflow، همه‌ی مخزن‌ها را می‌شکند. از tag استفاده کن و با PR تغییر بده.

#### ۷. composite action و `working-directory`

`working-directory` در composite به‌طور خودکار اعمال نمی‌شود. باید در هر step صریح بدهی:

```yaml
- shell: bash
  working-directory: ${{ inputs.dir }}
  run: npm ci
```

### نکات مهم (Tips & Tricks)

- **`@v1` به‌جای `@main`** برای reusable workflow در تولید.
- **`shell: bash` در هر step composite** — فراموش نکن.
- **`secrets:` صریح** به‌جای `inherit` برای org دیگر.
- **reusable workflow را تست کن** با یک workflow که خودش را صدا می‌زند.
- **`defaults.run.working-directory`** در reusable workflow برای مسیر.
- **مخزن `.github` سازمان** محل طبیعی reusable workflowهاست.

### خلاصه فصل

- **Reusable workflow** کل workflow را به اشتراک می‌گذارد (`workflow_call`).
- **Composite action** مجموعه‌ی stepها (`runs.using: composite`).
- Composite نیاز به `shell:` در هر step دارد.
- `env` منتقل نمی‌شود؛ `with` بده.
- `@v1` برای تولید، `@main` برای توسعه.
- `secrets: inherit` راحت ولی ناامن برای org دیگر.

**سؤالات مروری:**
1. تفاوت reusable workflow و composite action چیست؟
2. چرا `@main` در reusable workflow خطرناک است؟
3. چرا `shell:` در composite اجباری است؟

---

## فصل ۵۰: Environment، approval، concurrency و timeout

### مقدمه

سه کنترل که workflow را از «اسکریپت» به «سیستم تولیدی» تبدیل می‌کند: **environment** (تأیید انسانی و secret جدا)، **concurrency** (جلوگیری از اجرای هم‌زمان)، و **timeout** (جلوگیری از گیر کردن).

**چرا مهم است؟** بدون environment approval، هر merge به `main` مستقیم به production می‌رود. بدون concurrency، دو deploy هم‌زمان می‌توانند همدیگر را خراب کنند. بدون timeout، یک job گیرکرده ۶ ساعت دقیقه مصرف می‌کند.

**کاربرد عملی:** هر workflow تولیدی باید این سه را داشته باشد.

### مفاهیم اساسی

#### Environment

یک environment یک «هدف deploy» است با تنظیمات خودش:

| تنظیم | کار |
|---|---|
| **Required reviewers** | تا کسی تأیید نکند، job اجرا نمی‌شود |
| **Wait timer** | تأخیر اجباری قبل از اجرا |
| **Deployment branches** | فقط از branch مشخص |
| **Environment secrets** | secret مخصوص این محیط |
| **Environment variables** | متغیر مخصوص |
| **Deployment protection rules** | قواعد سفارشی (App) |

#### Concurrency

```yaml
concurrency:
  group: <نام گروه>
  cancel-in-progress: true|false
```

- یک گروه = یک «قفل». فقط یک اجرا هم‌زمان.
- `cancel-in-progress: true` → اجرای قبلی لغو می‌شود.
- `cancel-in-progress: false` → اجرای جدید در صف می‌ماند.

#### Timeout

| سطح | پیش‌فرض | حداکثر |
|---|---|---|
| job | ۳۶۰ دقیقه | ۳۶۰ |
| step | — | — |
| workflow | ۳۵ روز | — |

### مثال‌های عملی

#### مثال ۱: environment با approval

```yaml
jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - run: ./deploy.sh staging

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://example.com
    steps:
      - run: ./deploy.sh production
        env:
          API_KEY: ${{ secrets.PROD_API_KEY }}    # secret مخصوص production
```

تنظیم environment در **Settings → Environments → New environment**:

```
Name: production
☑ Required reviewers: @myorg/release-managers (حداقل ۱)
☑ Wait timer: 5 minutes
Deployment branches: Selected → main
Environment secrets:
  PROD_API_KEY = ...
  PROD_DB_URL = ...
```

حالا وقتی workflow به `deploy-production` می‌رسد، متوقف می‌شود و منتظر تأیید می‌ماند.

#### مثال ۲: environment در PR (بدون deploy)

```yaml
jobs:
  preview:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    environment:
      name: preview-pr-${{ github.event.pull_request.number }}
      url: https://pr-${{ github.event.pull_request.number }}.preview.example.com
    steps:
      - run: ./deploy-preview.sh
```

GitHub یک environment موقت برای هر PR می‌سازد و لینک را در PR نشان می‌دهد.

#### مثال ۳: concurrency برای CI

```yaml
concurrency:
  group: ci-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}
```

- روی یک PR، هر push جدید اجرای قبلی را لغو می‌کند.
- روی `main`، اجراها صف می‌شوند (نباید لغو شوند — ممکن است deploy باشند).

#### مثال ۴: concurrency برای deploy

```yaml
jobs:
  deploy:
    concurrency:
      group: deploy-production
      cancel-in-progress: false     # صف، نه لغو
    runs-on: ubuntu-latest
    environment: production
    steps:
      - run: ./deploy.sh
```

`cancel-in-progress: false` حیاتی است: اگر deploy در حال اجراست، deploy جدید باید **بعد** از آن اجرا شود، نه اینکه وسط کار قطعش کند.

#### مثال ۵: concurrency با `github.ref` — تله

```yaml
✗ concurrency:
    group: ci
    cancel-in-progress: true
```

همه‌ی branchها یک گروه → push روی `main` اجرای PR را لغو می‌کند. استفاده کن:

```yaml
✓ group: ci-${{ github.ref }}
```

و برای `merge_group`:

```yaml
✓ group: ci-${{ github.event.pull_request.number || github.ref }}
```

#### مثال ۶: timeout در همه سطوح

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 15          # کل job
    steps:
      - run: npm ci
        timeout-minutes: 5       # این step
      - run: npm test
        timeout-minutes: 10
```

#### مثال ۷: environment با branch restriction

```
Deployment branches: Selected branches
  - main
  - release/*
```

اگر workflow از branch دیگری تلاش کند، خطا می‌دهد. محافظت خوبی است.

#### مثال ۸: environment secret در برابر repo secret

| | repo secret | environment secret |
|---|---|---|
| دسترسی | همه‌ی jobها | فقط job با آن environment |
| تأیید | ❌ | ✅ (اگر required reviewers) |
| مناسب | token عمومی | کلید production |

**قاعده:** کلید production را در environment secret بگذار، نه repo secret. اگر کسی workflow را تغییر دهد، باز هم بدون approval به آن دسترسی ندارد.

#### مثال ۹: deployment protection rule سفارشی

می‌توانی یک GitHub App بسازی که قبل از deploy چک کند:

```yaml
# در App
- بررسی اینکه همه‌ی Issueهای milestone بسته شده‌اند
- بررسی اینکه ساعت کاری است
- بررسی اینکه incident فعال نیست
```

این قابلیت نیاز به ساخت App دارد — برای سازمان‌های بالغ.

### مقایسه و الگوهای مختلف

#### `cancel-in-progress` — جدول تصمیم

| سناریو | مقدار |
|---|---|
| CI روی PR | `true` |
| CI روی main | `false` |
| Deploy | `false` |
| Lint | `true` |
| Nightly | `false` |

#### Environment در برابر Ruleset

| | Environment | Ruleset |
|---|---|---|
| کنترل deploy | ✅ | ❌ |
| کنترل merge | ❌ | ✅ |
| approval انسانی | ✅ | (review) |
| secret جدا | ✅ | ❌ |

**هر دو لازم است:** Ruleset برای merge، Environment برای deploy.

#### timeout: کوتاه در برابر بلند

| | کوتاه (۵ دقیقه) | بلند (۶۰ دقیقه) |
|---|---|---|
| مصرف دقیقه در خطا | کم | زیاد |
| خطر قطع job سالم | ✅ | ❌ |
| مناسب | تست | build سنگین |

**توصیه:** timeout را کمی بیشتر از زمان معمول بگذار (مثلاً ۲× زمان نرمال).

### اشکال شایع

#### ۱. environment بدون required reviewers

اگر required reviewer نداشته باشد، environment فقط یک برچسب است. باید reviewer تعریف کنی.

#### ۲. `concurrency` که deploy را وسط کار قطع می‌کند

```yaml
✗ concurrency:
    group: deploy
    cancel-in-progress: true
```

deploy نیمه‌کاره می‌ماند. `false` بگذار.

#### ۳. environment secret در job بدون environment

```yaml
✗ steps:
    - run: echo ${{ secrets.PROD_KEY }}    # خالی است
```

secret محیطی فقط در job با `environment:` قابل دسترسی است.

#### ۴. `Wait timer` و اجرای دستی

`Wait timer` فقط برای jobهای غیردستی است. اگر از UI approve کنی، timer نادیده گرفته می‌شود (در بعضی نسخه‌ها).

#### ۵. timeout خیلی کوتاه

job سالم قطع می‌شود و تیم فکر می‌کند CI خراب است. زمان نرمال را اندازه بگیر و ۲× بگذار.

#### ۶. environment برای هر PR

اگر environment با نام `preview-${{ github.event.pull_request.number }}` بسازی، GitHub خودکار environment می‌سازد. ولی برای ۵۰۰ PR، ۵۰۰ environment داری. برای پروژه‌های فعال، یک environment `preview` با `url` داینامیک بهتر است.

#### ۷. deployment branches و tag

اگر فقط `main` را مجاز کنی، deploy از tag کار نمی‌کند. `v*` را هم اضافه کن اگر از tag deploy می‌کنی.

### نکات مهم (Tips & Tricks)

- **کلید production در environment secret** — نه repo secret.
- **`cancel-in-progress: false` برای deploy** — هرگز deploy را قطع نکن.
- **`concurrency` با `github.ref`** — نه گروه ثابت.
- **`timeout-minutes` در هر job** — عادت کن.
- **`environment.url`** لینک deploy را در PR نشان می‌دهد.
- **Deployment branches** را محدود کن — جلوگیری از deploy از branch شخصی.

### خلاصه فصل

- **Environment** = تأیید انسانی + secret جدا + محدودیت branch.
- **Concurrency** = جلوگیری از اجرای هم‌زمان؛ `false` برای deploy.
- **Timeout** = جلوگیری از job گیرکرده.
- کلید production را در environment secret بگذار.
- `cancel-in-progress: true` فقط برای CI/PR.

**سؤالات مروری:**
1. چرا deploy نباید `cancel-in-progress: true` داشته باشد؟
2. تفاوت repo secret و environment secret چیست؟
3. چرا `concurrency: ci` (بدون ref) خطرناک است؟

---

## فصل ۵۱: Self-hosted runner — نصب، امنیت، مقیاس

### مقدمه

Runnerهای GitHub رایگان و ساده‌اند، ولی محدودیت دارند: ۴ vCPU، ۱۶GB RAM، ۶ ساعت، و دسترسی به شبکه‌ی داخلی شرکت ندارند. اگر به GPU، رم زیاد، یا دسترسی به دیتابیس داخلی نیاز داری، باید runner خودت را داشته باشی.

**چرا مهم است؟** برای بعضی سازمان‌ها، self-hosted runner تنها راه است (compliance، شبکه‌ی بسته، سخت‌افزار خاص). ولی امنیت آن کاملاً به عهده‌ی توست — و اشتباه در آن می‌تواند فاجعه باشد.

**کاربرد عملی:** این فصل نصب، امنیت، و مقیاس را پوشش می‌دهد.

### مفاهیم اساسی

#### چه زمانی self-hosted

| نیاز | GitHub-hosted | Self-hosted |
|---|---|---|
| دسترسی به DB داخلی | ❌ | ✅ |
| GPU | ❌ (larger runners) | ✅ |
| رم > ۱۶GB | larger runners | ✅ |
| هزینه‌ی کم برای اجرای زیاد | ❌ | ✅ |
| compliance / data residency | ❌ | ✅ |
| سادگی | ✅ | ❌ |
| امنیت | ✅ (GitHub مدیریت می‌کند) | ❌ (خودت) |

#### انواع

| نوع | توضیح |
|---|---|
| **Repository-level** | فقط یک مخزن |
| **Organization-level** | همه‌ی مخزن‌های org |
| **Enterprise-level** | همه‌ی orgها |
| **Runner group** | گروه‌بندی با دسترسی مشخص |

#### خطر امنیتی اصلی

> **هر کسی که بتواند workflow را تغییر دهد، می‌تواند کد دلخواه روی runner تو اجرا کند.**

اگر runner به شبکه‌ی داخلی دسترسی دارد، مهاجم می‌تواند به دیتابیس، سرورها، و secretهای دیگر برسد.

**قاعده‌ی طلایی:** هرگز runner self-hosted را برای مخزن **عمومی** استفاده نکن. PR از fork می‌تواند کد دلخواه اجرا کند.

### مثال‌های عملی

#### مثال ۱: نصب runner (لینوکس)

```bash
# ۱) از GitHub یک token بگیر
# Settings → Actions → Runners → New self-hosted runner
# یا با gh:
TOKEN=$(gh api -X POST repos/OWNER/REPO/actions/runners/registration-token --jq .token)

# ۲) دانلود و نصب
mkdir -p ~/actions-runner && cd ~/actions-runner
curl -o actions-runner.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.320.0/actions-runner-linux-x64-2.320.0.tar.gz
tar xzf actions-runner.tar.gz

# ۳) پیکربندی
./config.sh \
  --url https://github.com/OWNER/REPO \
  --token "$TOKEN" \
  --name "runner-01" \
  --labels "self-hosted,linux,x64,gpu" \
  --work "_work" \
  --unattended

# ۴) نصب به‌عنوان سرویس
sudo ./svc.sh install
sudo ./svc.sh start
sudo ./svc.sh status
```

#### مثال ۲: استفاده در workflow

```yaml
jobs:
  test:
    runs-on: [self-hosted, linux, x64, gpu]
    steps:
      - uses: actions/checkout@v4
      - run: nvidia-smi
      - run: python train.py
```

`runs-on` آرایه = همه‌ی labelها باید منطبق باشند.

#### مثال ۳: runner group با دسترسی محدود

```bash
# ساخت گروه
gh api -X POST orgs/MYORG/actions/runner-groups \
  -f name="production-runners" \
  -f visibility="selected" \
  -f 'selected_repository_ids[]=123'
```

و در workflow:

```yaml
jobs:
  deploy:
    runs-on:
      group: production-runners
      labels: [self-hosted, linux]
```

#### مثال ۴: runner موقت (ephemeral) — امن‌ترین

```bash
./config.sh \
  --url https://github.com/OWNER/REPO \
  --token "$TOKEN" \
  --name "ephemeral-$(date +%s)" \
  --labels "self-hosted,linux" \
  --ephemeral \
  --unattended
```

با `--ephemeral`، runner بعد از **یک** job خودش را حذف می‌کند. هر job روی ماشین تازه = بدون آلودگی بین jobها.

**بهترین روش:** با autoscaling (مثال بعدی) runner موقت بساز.

#### مثال ۵: autoscaling با `actions-runner-controller` (Kubernetes)

```yaml
apiVersion: actions.summerwind.dev/v1alpha1
kind: RunnerDeployment
metadata:
  name: runners
  namespace: actions
spec:
  replicas: 3
  template:
    spec:
      repository: myorg/backend
      labels:
        - self-hosted
        - linux
---
apiVersion: actions.summerwind.dev/v1alpha1
kind: HorizontalRunnerAutoscaler
metadata:
  name: runners-autoscaler
spec:
  scaleTargetRef:
    name: runners
  minReplicas: 1
  maxReplicas: 20
  metrics:
    - type: TotalNumberOfQueuedAndInProgressWorkflowRuns
      repositoryNames:
        - myorg/backend
```

**نکته:** `actions-runner-controller` (ARC) پروژه‌ای است که سریع تغییر می‌کند. نسخه‌ی جدیدتر `gha-runner-scale-set` است. مستندات را چک کن.

#### مثال ۶: سخت‌سازی runner

```bash
# ۱) کاربر جداگانه، بدون sudo
sudo useradd -m -s /bin/bash runner
sudo -u runner ./config.sh ...

# ۲) runner را در container جدا اجرا کن
docker run -d \
  --name runner \
  --restart always \
  -e RUNNER_NAME=runner-docker \
  -e RUNNER_TOKEN="$TOKEN" \
  -e RUNNER_REPOSITORY_URL=https://github.com/OWNER/REPO \
  -v /var/run/docker.sock:/var/run/docker.sock \
  myoung34/github-runner:latest

# ۳) firewall: فقط به GitHub و مقصدهای لازم دسترسی
# ۴) لاگ audit
# ۵) به‌روزرسانی خودکار
```

#### مثال ۷: runner در شبکه‌ی بسته (بدون اینترنت)

اگر runner به اینترنت دسترسی ندارد، باید از GitHub Enterprise Server استفاده کنی (self-hosted کامل). برای GitHub.com، runner حداقل به `github.com` و `*.actions.githubusercontent.com` نیاز دارد.

#### مثال ۸: بررسی وضعیت runnerها

```bash
gh api repos/OWNER/REPO/actions/runners \
  --jq '.runners[] | "\(.name): \(.status) — \(.busy)"'

# حذف runner آفلاین
gh api -X DELETE repos/OWNER/REPO/actions/runners/<id>
```

### مقایسه و الگوهای مختلف

#### GitHub-hosted در برابر self-hosted

| | GitHub-hosted | Self-hosted |
|---|---|---|
| نگهداری | صفر | کامل |
| امنیت | ✅ | ❌ (خودت) |
| سرعت شروع | فوری | از قبل آماده |
| هزینه | دقیقه‌ای | سرور |
| دسترسی شبکه | عمومی | ✅ داخلی |
| مناسب | ۹۰٪ پروژه‌ها | نیاز خاص |

#### Persistent در برابر Ephemeral

| | Persistent | Ephemeral |
|---|---|---|
| سرعت | ✅ (کش، ابزار نصب‌شده) | کندتر |
| امنیت | ❌ (آلودگی بین jobها) | ✅ |
| دیباگ | آسان | سخت |
| توصیه | توسعه | تولید |

#### Runner-level در برابر Organization-level

| | Repo | Org |
|---|---|---|
| دامنه | یک مخزن | همه |
| مدیریت | ساده | متمرکز |
| امنیت | بهتر (جداسازی) | ❌ (همه دسترسی دارند) |
| مناسب | پروژه‌ی حساس | تیم بزرگ |

### اشکال شایع

#### ۱. self-hosted runner برای مخزن عمومی

**فاجعه.** هر کسی می‌تواند PR بدهد، workflow را تغییر دهد، و روی سرور تو کد اجرا کند. GitHub خودش هشدار قرمز می‌دهد.

#### ۲. runner با دسترسی root

اگر runner با root اجرا شود، هر کد مخربی می‌تواند کل سیستم را کنترل کند. **همیشه کاربر غیر root.**

#### ۳. runner مشترک بین مخزن‌های حساس و غیرحساس

اگر یک مخزن کمتر امن روی همان runner اجرا شود، می‌تواند داده‌ی مخزن حساس را بخواند (فایل‌های `_work` قبلی).

#### ۴. دیسک پر می‌شود

`_work` پر از cloneهای قدیمی. cleanup بگذار:

```bash
# cron روزانه
find ~/actions-runner/_work -maxdepth 1 -mtime +3 -exec rm -rf {} \;
```

#### ۵. runner آفلاین و job گیرکرده

runner که پاسخ نمی‌دهد، job را برای همیشه در صف نگه می‌دارد. مانیتور کن و حذف کن.

#### ۶. به‌روزرسانی نکردن runner

نسخه‌ی قدیمی ممکن است با GitHub ناسازگار شود. به‌روزرسانی خودکار بگذار:

```bash
# cron
0 3 * * * cd ~/actions-runner && ./run.sh --update 2>/dev/null || true
```

#### ۷. runner در Docker بدون `/var/run/docker.sock`

اگر job به Docker نیاز دارد، باید socket را mount کنی — که خودش یک ریسک امنیتی است (معادل root).

#### ۸. label اشتباه

```yaml
✗ runs-on: self-hosted
```

اگر چند runner با labelهای مختلف داری، بدون label دقیق، job روی runner اشتباه می‌رود. همیشه label دقیق بده.

### نکات مهم (Tips & Tricks)

- **هرگز runner self-hosted برای مخزن عمومی.** GitHub هشدار می‌دهد — جدی بگیر.
- **`--ephemeral`** برای تولید — هر job روی ماشین تازه.
- **کاربر غیر root** — همیشه.
- **runner group** برای جداسازی دسترسی.
- **cleanup دیسک** با cron.
- **`runs-on: [self-hosted, linux, x64]`** با label دقیق.
- **مانیتورینگ** — runner آفلاین، jobها را گیر می‌اندازد.

### خلاصه فصل

- Self-hosted برای GPU، شبکه‌ی داخلی، رم زیاد، compliance.
- **خطر اصلی:** هر کسی که workflow را تغییر دهد، کد روی سرور تو اجرا می‌کند.
- **هرگز برای مخزن عمومی.**
- `--ephemeral` امن‌ترین حالت.
- کاربر غیر root، runner group، cleanup دیسک.
- `actions-runner-controller` برای autoscaling در K8s.

**سؤالات مروری:**
1. چرا runner self-hosted برای مخزن عمومی خطرناک است؟
2. تفاوت persistent و ephemeral چیست؟
3. چطور از پر شدن دیسک runner جلوگیری کنی؟

---

**بعدی:** [`12-Actions-Security.md`](12-Actions-Security.md)

---

## منابع و مراجع

- GitHub Docs: Caching dependencies — https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows
- GitHub Docs: Storing workflow data as artifacts — https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts
- GitHub Docs: Reusing workflows — https://docs.github.com/en/actions/using-workflows/reusing-workflows
- GitHub Docs: Creating a composite action — https://docs.github.com/en/actions/creating-actions/creating-a-composite-action
- GitHub Docs: Using environments for deployment — https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment
- GitHub Docs: Controlling concurrency — https://docs.github.com/en/actions/using-jobs/using-concurrency
- GitHub Docs: Self-hosted runners — https://docs.github.com/en/actions/hosting-your-own-runners
- `actions-runner-controller` — https://github.com/actions/actions-runner-controller
