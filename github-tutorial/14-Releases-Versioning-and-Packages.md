# بخش ۱۴: Release، نسخه‌بندی و Packages

**مدت یادگیری**: ۳ تا ۴ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۱۳ (CI/CD)

---

## فهرست محتوا

- [فصل ۶۲: Release و tag — از commit تا نسخه](#فصل-۶۲)
- [فصل ۶۳: Semantic Versioning و Conventional Commits](#فصل-۶۳)
- [فصل ۶۴: GitHub Packages — npm، container، Maven](#فصل-۶۴)
- [فصل ۶۵: انتشار پکیج با Actions](#فصل-۶۵)

---

## فصل ۶۲: Release و tag — از commit تا نسخه

### مقدمه

یک tag فقط یک برچسب روی commit است. یک **release** یک مفهوم بالاتر است: نسخه‌ای که برای انسان‌ها معنا دارد، با توضیح تغییرات، فایل‌های قابل دانلود، و وضعیت (pre-release، latest).

**چرا مهم است؟** کاربران و ابزارها به release نگاه می‌کنند، نه به tag. `npm install`, `docker pull`, `gh release download` — همه از release/tag می‌خوانند. release بد = کاربر گیج.

**کاربرد عملی:** بعد از این فصل می‌دانی چطور یک نسخه را درست منتشر کنی و چطور notes خودکار بسازی.

### مفاهیم اساسی

#### tag در برابر release

| | Tag | Release |
|---|---|---|
| ماهیت | اشاره‌گر Git | شیء GitHub |
| در Git | ✅ | ✅ (به tag وصل) |
| فایل قابل دانلود | ❌ | ✅ |
| Release notes | ❌ | ✅ |
| Pre-release flag | ❌ | ✅ |
| Latest flag | ❌ | ✅ |
| در UI | کوچک | برجسته |

#### دو نوع tag

```bash
# lightweight — فقط اشاره‌گر
git tag v1.0.0

# annotated — با متادیتا (توصیه‌شده)
git tag -a v1.0.0 -m "نسخه ۱.۰.۰ — انتشار اولیه"
```

| | Lightweight | Annotated |
|---|---|---|
| متادیتا (نویسنده، تاریخ، پیام) | ❌ | ✅ |
| `git describe` | ❌ | ✅ |
| امضا (GPG/SSH) | ❌ | ✅ |
| push | `--tags` | `--tags` یا `--follow-tags` |
| توصیه | تگ موقت | ✅ release |

#### اجزای یک release

| جزء | توضیح |
|---|---|
| **Tag** | `v1.4.0` |
| **Title** | `v1.4.0 — جست‌وجوی سریع` |
| **Notes** | changelog |
| **Assets** | فایل‌های قابل دانلود (binary، zip) |
| **Pre-release** | `true` برای beta/rc |
| **Latest** | کدام release «آخرین» است |
| **Discussion** | ایجاد گفت‌وگو |

### مثال‌های عملی

#### مثال ۱: release کامل با gh

```bash
# ۱) تگ annotated
git switch main && git pull
git tag -a v1.4.0 -m "نسخه ۱.۴.۰"

# ۲) push تگ
git push origin v1.4.0

# ۳) release با notes خودکار
gh release create v1.4.0 \
  --title "v1.4.0 — جست‌وجوی سریع" \
  --generate-notes \
  --verify-tag

# ۴) یا با فایل
gh release create v1.4.0 dist/*.zip --generate-notes
```

#### مثال ۲: pre-release

```bash
gh release create v1.5.0-rc.1 \
  --title "v1.5.0-rc.1" \
  --prerelease \
  --generate-notes
```

`--prerelease` باعث می‌شود GitHub آن را «Latest» نگیرد.

#### مثال ۳: draft release

```bash
gh release create v1.4.0 --draft --generate-notes
# بعداً در UI بازبینی و publish کن
```

**چرا draft؟** می‌توانی notes را ویرایش کنی، assets اضافه کنی، و بعد منتشر کنی.

#### مثال ۴: notes دستی

````bash
gh release create v1.4.0 --notes "$(cat <<'EOF'
## 🚀 ویژگی‌های جدید
- جست‌وجوی فوری با `Cmd+K` (#142)
- پشتیبانی از تم تاریک (#156)

## 🐛 رفع باگ
- رفع خطای ورود با ایمیل بزرگ‌حرف (#151)
- رفع نمایش تاریخ در Safari (#160)

## ⚠️ تغییرات ناسازگار
- `searchV1()` حذف شد. از `search()` استفاده کنید.

## 🙏 مشارکت‌کنندگان
@ali-rezaei، @maryam-ahmadi
EOF
)"
````

#### مثال ۵: notes خودکار با دسته‌بندی

`.github/release.yml`:

```yaml
changelog:
  exclude:
    labels:
      - ignore-for-release
      - dependencies
    authors:
      - dependabot
      - github-actions
  categories:
    - title: "⚠️ تغییرات ناسازگار"
      labels: [breaking-change]
    - title: "🚀 ویژگی‌های جدید"
      labels: [enhancement, feature]
    - title: "🐛 رفع باگ"
      labels: [bug, fix]
    - title: "📚 مستندات"
      labels: [documentation]
    - title: "🔧 نگهداری"
      labels: [chore, ci, refactor]
    - title: "سایر تغییرات"
      labels: ["*"]
```

حالا `--generate-notes` یا دکمه‌ی UI، این ساختار را می‌سازد.

#### مثال ۶: دانلود asset

```bash
# لیست releaseها
gh release list

# دانلود همه‌ی assetهای یک release
gh release download v1.4.0

# فقط یک الگو
gh release download v1.4.0 --pattern "*.zip"

# آخرین release
gh release download --pattern "app-linux"
```

#### مثال ۷: release از workflow

```yaml
name: Release

on:
  push:
    tags: ["v*.*.*"]

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
        with:
          fetch-depth: 0

      - run: npm ci && npm run build

      - name: ساخت archive
        run: |
          tar -czf "app-${{ github.ref_name }}.tar.gz" -C dist .

      - uses: softprops/action-gh-release@<sha>
        with:
          generate_release_notes: true
          files: |
            app-*.tar.gz
          fail_on_unmatched_files: true
```

#### مثال ۸: بررسی یکپارچگی asset

```yaml
      - name: checksum
        run: |
          cd dist
          sha256sum * > checksums.txt
          cat checksums.txt

      - uses: softprops/action-gh-release@<sha>
        with:
          files: |
            dist/*
            dist/checksums.txt
```

کاربر می‌تواند با `sha256sum -c checksums.txt` بررسی کند.

#### مثال ۹: امضای asset

```yaml
      - name: امضا با cosign
        run: |
          cosign sign-blob --yes \
            --output-signature app.sig \
            --output-certificate app.pem \
            dist/app.tar.gz

      - uses: softprops/action-gh-release@<sha>
        with:
          files: |
            dist/app.tar.gz
            app.sig
            app.pem
```

#### مثال ۱۰: حذف release

```bash
gh release delete v1.4.0 --yes
# تگ هم پاک شود؟
gh release delete v1.4.0 --yes --cleanup-tag
```

### مقایسه و الگوهای مختلف

#### `--generate-notes` در برابر دستی

| | خودکار | دستی |
|---|---|---|
| سرعت | ✅ | کند |
| کیفیت | متوسط | ✅ |
| نیاز به label | ✅ | ❌ |
| مناسب | release مکرر | release مهم |

**راه میانه:** خودکار بساز، بعد در draft ویرایش کن.

#### Release در برابر Discussion announcement

| | Release | Discussion |
|---|---|---|
| فایل | ✅ | ❌ |
| نسخه | ✅ | ❌ |
| گفت‌وگو | محدود | ✅ |
| مناسب | انتشار | اعلام |

**بهترین:** release + Discussion که به آن لینک می‌دهد.

#### Latest flag

| | latest | pre-release |
|---|---|---|
| `/releases/latest` | ✅ | ❌ |
| `gh release download` | ✅ | ❌ |
| npm `@latest` | — | — |

اگر pre-release را latest کنی، کاربران نسخه‌ی ناپایدار می‌گیرند. **هرگز.**

### اشکال شایع

#### ۱. lightweight tag برای release

بدون متادیتا، `git describe` کار نمی‌کند و نمی‌دانی چه کسی و کِی tag زده. **annotated استفاده کن.**

#### ۲. tag جابه‌جا شده

```
✗ git tag -f v1.0.0 && git push --force origin v1.0.0
```

کسانی که نسخه‌ی قبلی را نصب کرده‌اند، چیز متفاوتی دارند. **نسخه‌ی جدید بزن.**

#### ۳. release بدون tag

`gh release create` خودش tag می‌سازد اگر نباشد. ولی بهتر است صریح باشی.

#### ۴. `--generate-notes` بدون label

اگر PRها label ندارند، همه در «سایر تغییرات» می‌روند. label را جدی بگیر.

#### ۵. release از branch اشتباه

```bash
✗ git tag v1.0.0   # روی feat/experimental
```

قبل از tag: `git log --oneline -1` و `git status` را چک کن.

#### ۶. asset با نام تکراری

هر asset باید نام یکتا داشته باشد. `app.tar.gz` و `app.tar.gz` در یک release ممکن نیست.

#### ۷. فراموش کردن push tag

```bash
git tag -a v1.0.0 -m "..."
# فراموش کردی push کنی
```

`git push --follow-tags` یا `git push origin v1.0.0`.

#### ۸. pre-release که latest می‌شود

اگر `--prerelease` را نگذاری، GitHub آن را latest می‌گیرد. برای beta حتماً بگذار.

### نکات مهم (Tips & Tricks)

- **annotated tag** — همیشه.
- **draft release** برای بازبینی notes.
- **`.github/release.yml`** برای changelog دسته‌بندی‌شده.
- **checksum + امضا** برای asset.
- **`--prerelease`** برای beta/rc.
- **tag را هرگز جابه‌جا نکن.**

### خلاصه فصل

- tag = اشاره‌گر Git؛ release = شیء GitHub با notes و asset.
- **annotated tag** برای release.
- `gh release create --generate-notes` سریع‌ترین راه.
- `.github/release.yml` changelog را دسته‌بندی می‌کند.
- `--prerelease` برای نسخه‌های ناپایدار.
- tag منتشرشده تغییرناپذیر است.

**سؤالات مروری:**
1. تفاوت tag و release چیست؟
2. چرا annotated tag بهتر است؟
3. چرا pre-release نباید latest باشد؟

---

## فصل ۶۳: Semantic Versioning و Conventional Commits

### مقدمه

نسخه‌بندی یک قرارداد است، نه یک عدد تصادفی. اگر درست باشد، کاربر می‌داند ارتقا امن است یا نه. اگر غلط باشد، هر ارتقا یک ریسک است.

**چرا مهم است؟** `npm install`، `go get`، `pip install` — همه از semver برای تصمیم‌گیری استفاده می‌کنند. اگر `1.4.0` یک تغییر ناسازگار داشته باشد و آن را `1.5.0` بزنی، همه‌ی کاربران می‌شکنند.

**کاربرد عملی:** semver + conventional commits = نسخه‌بندی خودکار و changelog خودکار.

### مفاهیم اساسی

#### SemVer: سه عدد

```
MAJOR.MINOR.PATCH
  1  .  4  .  2
```

| تغییر | اثر | مثال |
|---|---|---|
| رفع باگ سازگار | PATCH +1 | `1.4.2` → `1.4.3` |
| ویژگی سازگار | MINOR +1، PATCH=0 | `1.4.2` → `1.5.0` |
| تغییر ناسازگار | MAJOR +1، بقیه=0 | `1.4.2` → `2.0.0` |

#### قواعد مهم

1. **`0.x.y`** — API ناپایدار. هر تغییری می‌تواند ناسازگار باشد.
2. **`1.0.0`** — اولین API پایدار. تعهد می‌دهی.
3. **pre-release:** `1.5.0-alpha.1`، `1.5.0-beta.2`، `1.5.0-rc.1`
4. **build metadata:** `1.5.0+build.123` (در مقایسه نادیده گرفته می‌شود)

#### ترتیب pre-release

```
1.0.0-alpha < 1.0.0-alpha.1 < 1.0.0-alpha.beta < 1.0.0-beta < 1.0.0-beta.2 < 1.0.0-rc.1 < 1.0.0
```

#### Conventional Commits

```
<type>[optional scope]: <description>

[optional body]

[optional footer]
```

| type | اثر semver |
|---|---|
| `fix:` | PATCH |
| `feat:` | MINOR |
| `feat!:` یا `BREAKING CHANGE:` | MAJOR |
| `docs:`، `style:`، `refactor:`، `perf:`، `test:`، `build:`، `ci:`، `chore:` | هیچ |

#### محدوده‌ی اثر در monorepo

```
feat(ui): افزودن دکمه
feat(api): افزودن endpoint
```

scope به نسخه‌بندی per-package کمک می‌کند (با changesets).

### مثال‌های عملی

#### مثال ۱: تصمیم نسخه

| تغییر | نوع commit | نسخه‌ی جدید |
|---|---|---|
| رفع typo در پیام خطا | `fix: ...` | patch |
| افزودن پارامتر اختیاری | `feat: ...` | minor |
| حذف یک تابع export‌شده | `feat!: ...` | major |
| تغییر پیش‌فرض یک تنظیم | `feat!: ...` | major |
| ارتقای وابستگی (سازگار) | `chore(deps): ...` | هیچ |
| بازآرایی داخلی | `refactor: ...` | هیچ |

#### مثال ۲: نصب commitlint

```bash
npm install -D @commitlint/cli @commitlint/config-conventional
```

`commitlint.config.js`:

```javascript
export default {
  extends: ["@commitlint/config-conventional"],
  rules: {
    "type-enum": [
      2,
      "always",
      ["feat", "fix", "docs", "style", "refactor", "perf", "test", "build", "ci", "chore", "revert"],
    ],
    "subject-case": [0],           // فارسی/انگلیسی آزاد
    "header-max-length": [2, "always", 72],
    "body-max-line-length": [0],
  },
};
```

#### مثال ۳: hook با husky

```bash
npm install -D husky
npx husky init
echo 'npx --no -- commitlint --edit "$1"' > .husky/commit-msg
```

#### مثال ۴: hook با lefthook (سبک‌تر)

`lefthook.yml`:

```yaml
commit-msg:
  commands:
    commitlint:
      run: npx commitlint --edit {1}

pre-commit:
  parallel: true
  commands:
    lint:
      glob: "*.{js,ts,jsx,tsx}"
      run: npx eslint {staged_files}
    format:
      glob: "*.{js,ts,jsx,tsx,json,md}"
      run: npx prettier --check {staged_files}
```

```bash
npm install -D lefthook
npx lefthook install
```

#### مثال ۵: بررسی در CI (برای کسانی که hook را دور می‌زنند)

```yaml
  commitlint:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
        with:
          fetch-depth: 0
      - uses: actions/setup-node@<sha>
        with: { node-version: "22", cache: npm }
      - run: npm ci
      - run: |
          npx commitlint \
            --from ${{ github.event.pull_request.base.sha }} \
            --to ${{ github.event.pull_request.head.sha }} \
            --verbose
```

#### مثال ۶: بررسی دستی پیام

```bash
echo "feat(api): افزودن endpoint" | npx commitlint
echo "update stuff" | npx commitlint
# ✖ subject may not be empty
# ✖ type may not be empty
```

#### مثال ۷: تغییر پیش‌فرض = breaking

این یکی از ظریف‌ترین تصمیم‌هاست:

```javascript
// قبل
function connect(url = "localhost") { ... }

// بعد
function connect(url = "127.0.0.1") { ... }
```

تغییر «کوچک» است ولی رفتار پیش‌فرض عوض شده. کسی که `connect()` صدا می‌زند، نتیجه‌ی متفاوتی می‌گیرد.

```
✓ feat!: تغییر پیش‌فرض connect به 127.0.0.1

BREAKING CHANGE: مقدار پیش‌فرض از localhost به 127.0.0.1 تغییر کرد.
اگر به localhost وابسته بودید، صریح پاس بدهید.
```

#### مثال ۸: نسخه‌بندی با `npm version`

```bash
npm version patch    # 1.4.2 → 1.4.3
npm version minor    # 1.4.2 → 1.5.0
npm version major    # 1.4.2 → 2.0.0
npm version prerelease --preid=rc   # 1.4.2 → 1.4.3-rc.0

# با پیام custom
npm version minor -m "chore(release): %s"
```

`npm version` خودش commit و tag می‌زند (اگر در مخزن Git باشی).

#### مثال ۹: بررسی سازگاری

```bash
# آیا نسخه‌ی جدید با کد فعلی کار می‌کند؟
npm install mypackage@2.0.0
npm test

# ابزار: npm-check-updates
npx ncu
npx ncu -u && npm install
```

#### مثال ۱۰: `peerDependencies` و semver

```json
{
  "peerDependencies": {
    "react": "^18.0.0 || ^19.0.0"
  }
}
```

`^18.0.0` = `>=18.0.0 <19.0.0`. اگر React 19 را پشتیبانی می‌کنی، باید صریح اضافه کنی.

### مقایسه و الگوهای مختلف

#### `^` در برابر `~` در برابر exact

| نوشتار | معنی | مثال |
|---|---|---|
| `^1.4.2` | `>=1.4.2 <2.0.0` | minor و patch |
| `~1.4.2` | `>=1.4.2 <1.5.0` | فقط patch |
| `1.4.2` | دقیقاً | هیچ |
| `*` | هر چیزی | ❌ خطرناک |

**توصیه:** `^` برای کتابخانه، exact (یا lock) برای اپ.

#### SemVer در برابر CalVer

| | SemVer | CalVer |
|---|---|---|
| نمونه | `2.4.1` | `2026.10` |
| معنا | سازگاری | تاریخ |
| مناسب | کتابخانه | محصول/سیستم‌عامل |

#### Conventional Commits در برابر Gitmoji

| | Conventional | Gitmoji |
|---|---|---|
| ابزارپذیر | ✅ | متوسط |
| خوانایی | ✅ | ✅ |
| در ترمینال | ✅ | ایموجی مشکل‌دار |

### اشکال شایع

#### ۱. breaking change در minor

```
✗ feat: حذف پارامتر قدیمی   (در 1.4.0)
```

کاربران با `^1.4.0` ارتقا می‌دهند و می‌شکنند. باید `2.0.0` باشد.

#### ۲. تغییر پیش‌فرض بدون breaking

مثال ۷ بالا. ظریف ولی مهم.

#### ۳. `0.x` که هرگز `1.0.0` نمی‌شود

بعضی پروژه‌ها سال‌ها در `0.x` می‌مانند. یعنی هیچ تعهدی به سازگاری نیست. برای کتابخانه‌ی جدی، `1.0.0` بزن.

#### ۴. نسخه‌ی نادرست در `package.json`

```json
✗ "version": "1.0.0"   (ولی tag v2.1.0 است)
```

باید همگام باشند. خودکارسازی (semantic-release/changesets) این را حل می‌کند.

#### ۵. `feat!` که breaking نیست

اگر `!` بگذاری، major می‌شود. برای تغییر ناسازگار استفاده کن، نه برای «مهم».

#### ۶. Conventional Commits برای commitهای محلی

اگر `wip` می‌زنی، commitlint شکست می‌خورد. راه‌حل: `--no-verify` (موقت) یا squash قبل از merge.

#### ۷. پیام commit با فارسی و `!`

```
✗ feat!:افزودن (بدون فاصله)
✓ feat!: افزودن
```

commitlint به فاصله حساس است.

### نکات مهم (Tips & Tricks)

- **Conventional Commits از روز اول** — بعداً مهاجرت دردناک است.
- **commitlint در hook + CI** — hook قابل دور زدن است.
- **`feat!` برای breaking** — نه برای «مهم».
- **تغییر پیش‌فرض = breaking.**
- **`^` برای کتابخانه، lock برای اپ.**
- **`0.x` را جدی بگیر** — یا `1.0.0` بزن یا صریح بگو ناپایدار است.

### خلاصه فصل

- SemVer: MAJOR.MINOR.PATCH بر اساس سازگاری.
- `fix:` → patch، `feat:` → minor، `feat!:`/`BREAKING CHANGE:` → major.
- تغییر پیش‌فرض = breaking.
- commitlint در hook + CI.
- `^1.4.2` = `>=1.4.2 <2.0.0`.
- `0.x` = API ناپایدار.

**سؤالات مروری:**
1. چه زمانی نسخه باید major بزند؟
2. چرا تغییر مقدار پیش‌فرض breaking است؟
3. تفاوت `^` و `~` چیست؟

---

## فصل ۶۴: GitHub Packages — npm، container، Maven

### مقدمه

GitHub Packages یک registry است که در همان‌جا که کدت هست، پکیج‌هایت را نگه می‌دارد. سه مزیت: احراز هویت با `GITHUB_TOKEN`، دسترسی هم‌سطح با مخزن، و بدون سرویس جدا.

**چرا مهم است؟** برای پکیج‌های داخلی شرکت (که در npm عمومی نمی‌روند)، GitHub Packages یک registry خصوصی رایگان است.

**کاربرد عملی:** npm، container، Maven، NuGet، RubyGems — همه پشتیبانی می‌شوند.

### مفاهیم اساسی

#### registryهای پشتیبانی‌شده

| نوع | آدرس |
|---|---|
| npm | `npm.pkg.github.com` |
| Container | `ghcr.io` |
| Maven | `maven.pkg.github.com` |
| Gradle | `maven.pkg.github.com` |
| NuGet | `nuget.pkg.github.com` |
| RubyGems | `rubygems.pkg.github.com` |

#### دسترسی

| سطح | تنظیم |
|---|---|
| مخزن | خودکار (هم‌سطح مخزن) |
| کاربر | با `GITHUB_TOKEN` یا PAT |
| سازمان | با permission |

#### محدودیت‌ها (Free)

| | مقدار |
|---|---|
| فضای ذخیره‌سازی | ۵۰۰MB |
| انتقال داده | ۱GB/ماه |
| پکیج خصوصی | ✅ |

#### `GITHUB_TOKEN` در برابر PAT

| | `GITHUB_TOKEN` | PAT |
|---|---|---|
| در Actions | ✅ | ✅ |
| در local | ❌ | ✅ |
| عمر | job | ماه‌ها |
| scope | `packages: write` | `write:packages` |

**نکته:** برای publish از Actions، `GITHUB_TOKEN` کافی است. برای install در local، PAT لازم داری.

### مثال‌های عملی

#### مثال ۱: publish به npm registry گیت‌هاب

`.npmrc` در پروژه:

```
@myorg:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${NODE_AUTH_TOKEN}
```

`package.json`:

```json
{
  "name": "@myorg/ui",
  "version": "1.4.0",
  "publishConfig": {
    "registry": "https://npm.pkg.github.com"
  }
}
```

**نکته‌ی حیاتی:** نام پکیج باید با scope سازمان شروع شود (`@myorg/...`).

workflow:

```yaml
name: Publish

on:
  release:
    types: [published]

permissions:
  contents: read
  packages: write

jobs:
  publish:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
      - uses: actions/setup-node@<sha>
        with:
          node-version: "22"
          cache: npm
          registry-url: "https://npm.pkg.github.com"
          scope: "@myorg"
      - run: npm ci
      - run: npm run build
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

#### مثال ۲: install از GitHub Packages

`.npmrc`:

```
@myorg:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${NODE_AUTH_TOKEN}
```

```bash
export NODE_AUTH_TOKEN=ghp_xxx
npm install @myorg/ui
```

یا در `package.json`:

```json
{
  "dependencies": {
    "@myorg/ui": "^1.4.0"
  }
}
```

**در CI:**

```yaml
- uses: actions/setup-node@<sha>
  with:
    node-version: "22"
    registry-url: "https://npm.pkg.github.com"
    scope: "@myorg"
- run: npm ci
  env:
    NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**توجه:** اگر پکیج در مخزن دیگری است، `GITHUB_TOKEN` دسترسی ندارد (فقط همان مخزن). باید PAT یا GitHub App بدهی.

#### مثال ۳: container در GHCR

```yaml
      - uses: docker/login-action@<sha>
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - uses: docker/build-push-action@<sha>
        with:
          push: true
          tags: ghcr.io/myorg/app:${{ github.sha }}
```

**توجه:** `ghcr.io/myorg/app` — نام با lowercase.

#### مثال ۴: Maven

`pom.xml`:

```xml
<distributionManagement>
  <repository>
    <id>github</id>
    <url>https://maven.pkg.github.com/myorg/myrepo</url>
  </repository>
</distributionManagement>
```

`~/.m2/settings.xml`:

```xml
<servers>
  <server>
    <id>github</id>
    <username>${env.GITHUB_ACTOR}</username>
    <password>${env.GITHUB_TOKEN}</password>
  </server>
</servers>
```

workflow:

```yaml
- uses: actions/setup-java@<sha>
  with:
    java-version: "21"
    distribution: "temurin"
    server-id: github
    settings-path: ${{ github.workspace }}

- run: mvn deploy -s $GITHUB_WORKSPACE/settings.xml
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

#### مثال ۵: Gradle

`build.gradle.kts`:

```kotlin
publishing {
    repositories {
        maven {
            name = "GitHubPackages"
            url = uri("https://maven.pkg.github.com/myorg/myrepo")
            credentials {
                username = System.getenv("GITHUB_ACTOR")
                password = System.getenv("GITHUB_TOKEN")
            }
        }
    }
}
```

#### مثال ۶: NuGet

```yaml
- uses: actions/setup-dotnet@<sha>
  with:
    source-url: https://nuget.pkg.github.com/myorg/index.json
  env:
    NUGET_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

- run: dotnet pack -c Release
- run: dotnet nuget push "bin/Release/*.nupkg" --skip-duplicate
```

#### مثال ۷: حذف نسخه‌ی پکیج

```bash
# لیست نسخه‌ها
gh api /orgs/myorg/packages/npm/ui/versions --jq '.[] | "\(.id) \(.name)"'

# حذف یک نسخه
gh api -X DELETE /orgs/myorg/packages/npm/ui/versions/<version-id>

# حذف پکیج کامل
gh api -X DELETE /orgs/myorg/packages/npm/ui
```

#### مثال ۸: visibility پکیج

پکیج به‌طور پیش‌فرض **خصوصی** است، حتی اگر مخزن عمومی باشد. برای عمومی کردن:

```
Package settings → Danger Zone → Change visibility → Public
```

یا با API:

```bash
gh api -X PATCH /orgs/myorg/packages/npm/ui \
  -f visibility=public
```

#### مثال ۹: لینک پکیج به مخزن

```
Package → Connect repository → myorg/ui
```

این کار:

- پکیج را در صفحه‌ی مخزن نشان می‌دهد
- README مخزن را در صفحه‌ی پکیج نمایش می‌دهد
- دسترسی را با مخزن هم‌سطح می‌کند

#### مثال ۱۰: provenance برای npm

```yaml
- run: npm publish --provenance --access public
  env:
    NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

`--provenance` یک attestation می‌سازد که نشان می‌دهد پکیج از کدام مخزن و commit ساخته شده. کاربر می‌تواند با `npm audit signatures` بررسی کند.

### مقایسه و الگوهای مختلف

#### GitHub Packages در برابر npmjs.com

| | GitHub Packages | npmjs.com |
|---|---|---|
| پکیج خصوصی رایگان | ✅ (۵۰۰MB) | پولی |
| احراز هویت در Actions | ✅ `GITHUB_TOKEN` | token جدا |
| سرعت نصب | متوسط | ✅ CDN |
| کش عمومی | ❌ | ✅ |
| مناسب | پکیج داخلی | پکیج عمومی |

#### GHCR در برابر Docker Hub

| | GHCR | Docker Hub |
|---|---|---|
| خصوصی رایگان | ✅ | ۱ عدد |
| rate limit | بالا | ۱۰۰/۶ساعت (ناشناس) |
| یکپارچگی | ✅ | ❌ |

#### `.npmrc` در مخزن در برابر CI

```
✗ .npmrc با token در مخزن
✓ .npmrc با ${NODE_AUTH_TOKEN} + token در secret
```

### اشکال شایع

#### ۱. نام پکیج بدون scope

```
✗ "name": "ui"
✓ "name": "@myorg/ui"
```

GitHub Packages نیاز به scope دارد.

#### ۲. `publishConfig` فراموش شد

اگر `publishConfig.registry` نباشد، npm به npmjs.com می‌فرستد.

#### ۳. token در `.npmrc` commit شده

فاجعه. از `${NODE_AUTH_TOKEN}` استفاده کن.

#### ۴. دسترسی به پکیج مخزن دیگر

`GITHUB_TOKEN` فقط به مخزن خودش دسترسی دارد. برای پکیج در مخزن دیگر، PAT یا App.

#### ۵. پکیج خصوصی که فکر می‌کردی عمومی است

پیش‌فرض خصوصی است. برای عمومی کردن، visibility را عوض کن.

#### ۶. نسخه‌ی تکراری

```
npm ERR! 403 Forbidden - PUT ... - Cannot publish over existing version
```

نسخه را افزایش بده. **هرگز نسخه‌ی منتشرشده را overwrite نکن.**

#### ۷. `ghcr.io` با uppercase

```
✗ ghcr.io/MyOrg/MyApp
✓ ghcr.io/myorg/myapp
```

registry نام lowercase می‌خواهد.

#### ۸. `packages: write` فراموش شد

```
denied: permission_denied
```

#### ۹. محدودیت حجم

۵۰۰MB در Free. اگر بیشتر شد، نسخه‌های قدیمی را پاک کن.

### نکات مهم (Tips & Tricks)

- **scope اجباری** برای npm.
- **`GITHUB_TOKEN` در Actions کافی است** — PAT فقط برای local.
- **`--provenance`** برای زنجیره‌ی تأمین.
- **پکیج را به مخزن connect کن** — دسترسی و README.
- **visibility را چک کن** — پیش‌فرض خصوصی.
- **نسخه‌های قدیمی را پاک کن** — محدودیت ۵۰۰MB.

### خلاصه فصل

- GitHub Packages: npm، container، Maven، NuGet، RubyGems.
- `GITHUB_TOKEN` + `packages: write` برای publish از Actions.
- نام npm باید `@scope/name` باشد.
- `publishConfig.registry` را تنظیم کن.
- پکیج پیش‌فرض خصوصی است.
- نسخه‌ی منتشرشده را overwrite نکن.

**سؤالات مروری:**
1. چرا نام پکیج npm باید scope داشته باشد؟
2. چرا `GITHUB_TOKEN` به پکیج مخزن دیگر دسترسی ندارد؟
3. چرا پکیج پیش‌فرض خصوصی است؟

---

## فصل ۶۵: انتشار پکیج با Actions

### مقدمه

این فصل یک pipeline کامل انتشار می‌سازد: از merge به `main` تا نسخه‌ی جدید در registry، با changelog، tag، release و attestation.

**چرا مهم است؟** انتشار دستی خطا دارد: نسخه‌ی اشتباه، فراموش کردن tag، changelog ناقص. خودکارسازی اینها را حذف می‌کند.

**کاربرد عملی:** الگوی این فصل را می‌توانی در هر پکیجی استفاده کنی.

### مفاهیم اساسی

#### دو الگوی انتشار

| الگو | trigger | مناسب |
|---|---|---|
| **خودکار از commit** | push به main | semantic-release |
| **PR نسخه** | changesets | کنترل انسانی |

#### پیش‌نیازها

| نیاز | چرا |
|---|---|
| Conventional Commits | تعیین نسخه |
| `fetch-depth: 0` | تاریخ کامل |
| `NPM_TOKEN` یا `GITHUB_TOKEN` | publish |
| `id-token: write` | provenance |
| Ruleset روی tag | جلوگیری از تغییر |

#### چک‌لیست انتشار

- [ ] CI سبز روی commit
- [ ] نسخه افزایش یافت
- [ ] changelog به‌روز شد
- [ ] tag annotated زده شد
- [ ] release با asset ساخته شد
- [ ] پکیج منتشر شد
- [ ] attestation/provenance
- [ ] smoke test نصب

### مثال‌های عملی

#### مثال ۱: انتشار کامل با semantic-release

```yaml
name: Release

on:
  push:
    branches: [main]

permissions:
  contents: write
  issues: write
  pull-requests: write
  id-token: write

concurrency:
  group: release
  cancel-in-progress: false

jobs:
  release:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
        with:
          fetch-depth: 0
          persist-credentials: false

      - uses: actions/setup-node@<sha>
        with:
          node-version: "22"
          cache: npm
          registry-url: "https://registry.npmjs.org"

      - run: npm ci
      - run: npm test
      - run: npm run build

      - run: npx semantic-release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

`.releaserc.json`:

```json
{
  "branches": ["main", { "name": "beta", "prerelease": true }],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    ["@semantic-release/changelog", { "changelogFile": "CHANGELOG.md" }],
    ["@semantic-release/npm", { "provenance": true }],
    ["@semantic-release/git", {
      "assets": ["CHANGELOG.md", "package.json"],
      "message": "chore(release): ${nextRelease.version} [skip ci]"
    }],
    "@semantic-release/github"
  ]
}
```

#### مثال ۲: انتشار با changesets

```yaml
name: Release

on:
  push:
    branches: [main]

permissions:
  contents: write
  pull-requests: write
  id-token: write

jobs:
  release:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
        with:
          fetch-depth: 0

      - uses: pnpm/action-setup@<sha>
        with: { version: 9 }

      - uses: actions/setup-node@<sha>
        with:
          node-version: "22"
          cache: pnpm
          registry-url: "https://registry.npmjs.org"

      - run: pnpm install --frozen-lockfile
      - run: pnpm build

      - uses: changesets/action@<sha>
        with:
          publish: pnpm release
          version: pnpm version-packages
          commit: "chore: انتشار نسخه"
          title: "chore: انتشار نسخه"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

`package.json`:

```json
{
  "scripts": {
    "version-packages": "changeset version && pnpm install --lockfile-only",
    "release": "changeset publish"
  }
}
```

#### مثال ۳: انتشار به GitHub Packages

```yaml
name: Publish to GHCR-npm

on:
  release:
    types: [published]

permissions:
  contents: read
  packages: write
  id-token: write

jobs:
  publish:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>

      - uses: actions/setup-node@<sha>
        with:
          node-version: "22"
          cache: npm
          registry-url: "https://npm.pkg.github.com"
          scope: "@myorg"

      - run: npm ci
      - run: npm run build

      - name: بررسی تطابق نسخه با tag
        run: |
          PKG=$(node -p "require('./package.json').version")
          TAG="${{ github.event.release.tag_name }}"
          TAG="${TAG#v}"
          if [ "$PKG" != "$TAG" ]; then
            echo "::error::نسخه package.json ($PKG) با tag ($TAG) نمی‌خواند"
            exit 1
          fi

      - run: npm publish --provenance
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**بررسی تطابق نسخه با tag** یکی از مهم‌ترین محافظت‌هاست — جلوی انتشار نسخه‌ی اشتباه را می‌گیرد.

#### مثال ۴: انتشار چند registry

```yaml
      - name: publish به npmjs
        run: npm publish --provenance --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}

      - name: publish به GH Packages
        run: |
          npm config set @myorg:registry https://npm.pkg.github.com
          npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

#### مثال ۵: smoke test بعد از انتشار

```yaml
      - name: smoke test
        run: |
          VERSION=$(node -p "require('./package.json').version")
          mkdir -p /tmp/smoke && cd /tmp/smoke
          npm init -y > /dev/null
          npm install "my-package@$VERSION"
          node -e "const p = require('my-package'); console.log('نسخه:', p.version)"
```

**چرا مهم است؟** گاهی publish موفق است ولی پکیج ناقص است (فایل‌های build در `files` نیستند).

#### مثال ۶: `files` در package.json

```json
{
  "files": ["dist", "README.md", "LICENSE"]
}
```

**تله‌ی رایج:** بدون `files`، همه‌چیز publish می‌شود (حتی تست‌ها). با `files`، فقط آنچه لازم است.

#### مثال ۷: بررسی محتوای پکیج قبل از publish

```bash
npm pack --dry-run
# npm notice 📦  @myorg/ui@1.4.0
# npm notice Tarball Contents
# npm notice 12.3kB dist/index.js
# npm notice 1.2kB  README.md
# npm notice Tarball Details
# npm notice total files: 2
```

**همیشه قبل از publish این را ببین.** اگر `node_modules` یا `.env` در لیست است، متوقف شو.

#### مثال ۸: tag خودکار

```yaml
      - name: تگ
        run: |
          VERSION=$(node -p "require('./package.json').version")
          git tag -a "v$VERSION" -m "نسخه $VERSION"
          git push origin "v$VERSION"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**توجه:** با `GITHUB_TOKEN`، push تگ workflow جدید trigger نمی‌کند (محافظت داخلی). اگر می‌خواهی release workflow را trigger کنی، باید GitHub App یا PAT بدهی.

#### مثال ۹: انتشار دستی با تأیید

```yaml
on:
  workflow_dispatch:
    inputs:
      bump:
        type: choice
        options: [patch, minor, major]
        required: true

jobs:
  publish:
    runs-on: ubuntu-24.04
    environment: npm-release     # approval
    steps:
      - uses: actions/checkout@<sha>
        with: { fetch-depth: 0 }
      - uses: actions/setup-node@<sha>
        with: { node-version: "22", cache: npm, registry-url: "https://registry.npmjs.org" }
      - run: npm ci && npm test && npm run build
      - run: npm version ${{ inputs.bump }} -m "chore(release): %s"
      - run: npm publish --provenance
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
      - run: git push --follow-tags
```

#### مثال ۱۰: بررسی provenance

```bash
# کاربر می‌تواند بررسی کند
npm audit signatures

# یا
npm view my-package@1.4.0 dist.attestations
```

### مقایسه و الگوهای مختلف

#### npmjs در برابر GH Packages

| | npmjs | GH Packages |
|---|---|---|
| مخاطب | عمومی | داخلی |
| CDN | ✅ | ❌ |
| خصوصی رایگان | ❌ | ✅ |
| provenance | ✅ | محدود |
| توصیه | پکیج عمومی | پکیج داخلی |

#### خودکار در برابر دستی با approval

| | خودکار | دستی |
|---|---|---|
| سرعت | ✅ | کند |
| کنترل | کم | ✅ |
| ریسک | انتشار اشتباه | فراموشی |
| مناسب | کتابخانه‌ی بالغ | پکیج حساس |

#### `npm version` در برابر semantic-release

| | `npm version` | semantic-release |
|---|---|---|
| تعیین نسخه | دستی/input | خودکار |
| changelog | ❌ | ✅ |
| tag | ✅ | ✅ |
| مناسب | ساده | بالغ |

### اشکال شایع

#### ۱. publish بدون build

```yaml
✗ - run: npm publish
✓ - run: npm run build && npm publish
```

پکیج خالی منتشر می‌شود.

#### ۲. `files` تنظیم نشده

`node_modules`، تست‌ها، `.env` در پکیج. حجم زیاد و نشت اطلاعات.

#### ۳. نسخه‌ی تکراری

```
npm ERR! 403 You cannot publish over the previously published versions
```

نسخه را افزایش بده.

#### ۴. نسخه‌ی package.json با tag نمی‌خواند

```
✗ package.json: 1.4.0، tag: v1.5.0
```

بررسی تطابق بگذار (مثال ۳).

#### ۵. `NPM_TOKEN` بدون 2FA bypass

```
npm ERR! 403 Two-factor authentication required
```

token باید «Automation» type باشد.

#### ۶. `id-token: write` فراموش شد

```
Error: provenance generation failed
```

#### ۷. `fetch-depth: 1`

semantic-release/changesets به تاریخ کامل نیاز دارند.

#### ۸. حلقه‌ی انتشار

```
✗ commit release → trigger release → commit release → ...
```

`[skip ci]` در پیام commit.

#### ۹. publish از PR

```yaml
✗ on: pull_request
```

هر PR نسخه منتشر می‌کند. فقط `push` به main یا `release`.

### نکات مهم (Tips & Tricks)

- **`npm pack --dry-run`** قبل از publish.
- **`files` در package.json** — محتوای پکیج را کنترل کن.
- **بررسی تطابق نسخه با tag.**
- **smoke test بعد از publish.**
- **`--provenance`** برای زنجیره‌ی تأمین.
- **`[skip ci]`** در commit release.
- **environment approval** برای پکیج حساس.

### خلاصه فصل

- دو الگو: خودکار (semantic-release) و PR نسخه (changesets).
- `fetch-depth: 0` و `id-token: write` لازم است.
- **`files` در package.json** محتوای پکیج را کنترل می‌کند.
- بررسی تطابق نسخه با tag، جلوی انتشار اشتباه را می‌گیرد.
- smoke test بعد از publish.
- `--provenance` برای attestation.

**سؤالات مروری:**
1. چرا `npm pack --dry-run` قبل از publish مهم است؟
2. چرا بررسی تطابق نسخه با tag لازم است؟
3. چرا `GITHUB_TOKEN` نمی‌تواند workflow جدید trigger کند؟

---

**بعدی:** [`15-Copilot.md`](15-Copilot.md)

---

## منابع و مراجع

- GitHub Docs: About releases — https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases
- GitHub Docs: Automatically generated release notes — https://docs.github.com/en/repositories/releasing-projects-on-github/automatically-generated-release-notes
- GitHub Docs: Working with a GitHub Packages registry — https://docs.github.com/en/packages/working-with-a-github-packages-registry
- GitHub Docs: Configuring npm for GitHub Packages — https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-npm-registry
- Semantic Versioning — https://semver.org/
- Conventional Commits — https://www.conventionalcommits.org
- semantic-release — https://semantic-release.gitbook.io/
- changesets — https://github.com/changesets/changesets
- npm: provenance — https://docs.npmjs.com/generating-provenance-statements
