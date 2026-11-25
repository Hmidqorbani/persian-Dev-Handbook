# بخش ۱۹: Git پیشرفته

**مدت یادگیری**: ۴ تا ۵ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: بخش ۰۵ (بازنویسی تاریخ)، بخش ۰۳ (branch)

---

## فهرست محتوا

- [فصل ۸۴: `git worktree` — کار موازی روی چند شاخه](#فصل-۸۴)
- [فصل ۸۵: Submodule و Subtree](#فصل-۸۵)
- [فصل ۸۶: Git LFS — فایل‌های بزرگ](#فصل-۸۶)
- [فصل ۸۷: Hooks — pre-commit، commit-msg و CI محلی](#فصل-۸۷)
- [فصل ۸۸: امضای commit و tag](#فصل-۸۸)

---

## فصل ۸۴: `git worktree` — کار موازی روی چند شاخه

### مقدمه

می‌خواهی هم‌زمان روی `feat/login` کار کنی و یک باگ فوری در `main` را رفع کنی. راه معمول: `git stash`، `switch`، کار، برگشت، `stash pop`. راه بهتر: `git worktree`.

`worktree` یک پوشه‌ی **دیگر** می‌سازد که همان مخزن را با branch دیگری نشان می‌دهد. بدون clone دوم، بدون stash.

**چرا مهم است؟** برای کارهای موازی (feature + hotfix + review) و برای اجرای هم‌زمان تست روی دو نسخه، `worktree` بهترین ابزار است.

**کاربرد عملی:** وقتی reviewer می‌گوید «PR را چک کن»، `worktree` باز می‌کنی، تست می‌کنی، پاک می‌کنی — بدون اینکه کار خودت را رها کنی.

### مفاهیم اساسی

#### چطور کار می‌کند

```
repo/
├── .git/                    ← مخزن مشترک
└── src/                     ← worktree اصلی (main)

repo-hotfix/                 ← worktree دوم (hotfix/x)
repo-review/                 ← worktree سوم (feat/y)
```

هر worktree یک branch **متفاوت** checkout می‌کند. نمی‌توانی دو worktree روی یک branch داشته باشی.

#### چرا بهتر از clone دوم

| | worktree | clone دوم |
|---|---|---|
| فضا | کم (فقط working dir) | کامل (کل تاریخ) |
| همگام‌سازی | ✅ خودکار (همان .git) | دستی |
| branch مشترک | ✅ | ❌ (جدا) |
| سرعت ساخت | ثانیه | دقیقه |

#### دستورها

```bash
git worktree add <path> <branch>
git worktree list
git worktree remove <path>
git worktree prune
git worktree move <path> <newpath>
```

### مثال‌های عملی

#### مثال ۱: hotfix در حین کار روی ویژگی

```bash
cd ~/projects/app
git switch feat/login
# وسط کار، باگ فوری آمد

git worktree add ../app-hotfix main
cd ../app-hotfix
# رفع باگ
git commit -am "fix(auth): رفع خطای ورود"
git push

cd ~/projects/app
git worktree remove ../app-hotfix
# کار خودت دست‌نخورده
```

#### مثال ۲: review یک PR

```bash
git fetch origin pull/42/head:pr-42
git worktree add ../review-42 pr-42
cd ../review-42
npm install && npm test
# بررسی
cd ..
git worktree remove ../review-42
git branch -D pr-42
```

**مزیت:** کار خودت رها نمی‌شود، و `node_modules` جداگانه است (که گاهی هم عیب است).

#### مثال ۳: branch جدید از یک worktree

```bash
git worktree add ../app-feature2 -b feat/search
cd ../app-feature2
# branch جدید ساخته و checkout شد
```

#### مثال ۴: اجرای هم‌زمان تست روی دو نسخه

```bash
git worktree add ../app-old v1.4.0
cd ../app-old && npm ci && npm test > /tmp/old.log 2>&1 &
cd ~/projects/app && npm ci && npm test > /tmp/new.log 2>&1 &
wait
diff /tmp/old.log /tmp/new.log
```

#### مثال ۵: ساختار پیشنهادی

```
~/projects/
├── app/              ← main
├── app-hotfix/       ← hotfix
├── app-review-42/    ← review
└── app-experiment/   ← آزمایش
```

**نکته:** `worktree`ها را در پوشه‌ی جدا بگذار، نه داخل `app/` (وگرنه Git آن‌ها را untracked می‌بیند).

#### مثال ۶: بررسی وضعیت

```bash
git worktree list
# /home/user/projects/app          a1b2c3d [main]
# /home/user/projects/app-hotfix   e4f5g6h [hotfix/x]
# /home/user/projects/app-review   i7j8k9l [pr-42]
```

#### مثال ۷: پاک کردن

```bash
# اگر تغییری نداری
git worktree remove ../app-hotfix

# اگر تغییر داری (اجباری)
git worktree remove --force ../app-hotfix

# اگر پوشه را دستی پاک کردی
git worktree prune
```

#### مثال ۸: قفل کردن

```bash
# جلوگیری از prune
git worktree lock ../app-hotfix --reason "در حال دیباگ"
git worktree unlock ../app-hotfix
```

برای worktree روی درایو قابل حمل مفید است.

#### مثال ۹: محدودیت‌ها

| محدودیت | توضیح |
|---|---|
| یک branch، یک worktree | نمی‌شود دو بار checkout کرد |
| submodule | هر worktree باید `submodule update` جدا بزند |
| `node_modules` | جداگانه (نه مشترک) |
| hookها | مشترک (در `.git/hooks`) |

#### مثال ۱۰: alias مفید

```bash
git config --global alias.wt "worktree"
# استفاده: git wt add ../hotfix main
```

### مقایسه و الگوهای مختلف

#### worktree در برابر stash

| | worktree | stash |
|---|---|---|
| کار موازی | ✅ واقعی | ❌ سریال |
| context | ✅ کامل | ❌ باید برگردی |
| فضا | کمی بیشتر | صفر |
| مناسب | کار طولانی | وقفه‌ی کوتاه |

#### worktree در برابر clone

| | worktree | clone |
|---|---|---|
| فضا | کم | زیاد |
| همگام‌سازی | خودکار | دستی |
| fork/remote جدا | ❌ | ✅ |
| مناسب | همان مخزن | مخزن دیگر |

#### worktree در برابر branch ساده

| | worktree | switch |
|---|---|---|
| هم‌زمانی | ✅ | ❌ |
| build جدا | ✅ | ❌ |
| مناسب | تست موازی | کار متوالی |

### اشکال شایع

#### ۱. worktree داخل مخزن

```bash
✗ git worktree add ./hotfix main
```

Git آن را untracked می‌بیند. بیرون بگذار.

#### ۲. همان branch در دو worktree

```
fatal: 'main' is already checked out at '/home/user/app'
```

branch را عوض کن یا worktree جدید بساز.

#### ۳. پاک کردن پوشه بدون `remove`

اگر `rm -rf` بزنی، Git می‌فهمد (بعد از `prune`) ولی بهتر است `remove` بزنی.

#### ۴. فراموش کردن worktreeهای قدیمی

```bash
git worktree list
git worktree prune
```

#### ۵. submodule در worktree جدید

```bash
cd ../app-hotfix
git submodule update --init --recursive
```

#### ۶. `.env` جدا

هر worktree `.env` جدا می‌خواهد (چون gitignore شده). کپی کن.

### نکات مهم (Tips & Tricks)

- **worktree را بیرون مخزن بگذار.**
- **`git worktree list`** برای دیدن وضعیت.
- **`prune`** برای پاکسازی.
- **برای review، `pull/42/head`** را fetch کن.
- **`node_modules` جدا** — می‌تواند مزیت (جداسازی) یا عیب (فضا) باشد.

### خلاصه فصل

- `worktree` چند پوشه از یک مخزن با branchهای مختلف.
- سبک‌تر از clone، همگام‌سازی خودکار.
- هر branch فقط در یک worktree.
- برای hotfix، review، تست موازی.
- بیرون مخزن بگذار.

**سؤالات مروری:**
1. چرا worktree بهتر از clone دوم است؟
2. چرا نمی‌شود دو worktree روی یک branch داشت؟
3. چطور یک PR را در worktree تست کنی؟

---

## فصل ۸۵: Submodule و Subtree

### مقدمه

گاهی یک پروژه به مخزن دیگری وابسته است (کتابخانه‌ی مشترک، تم، مستندات). دو راه: **submodule** (اشاره‌گر به یک commit) و **subtree** (کپی محتوا در مخزن).

**چرا مهم است؟** انتخاب اشتباه، درد بزرگی می‌سازد. submodule برای «مخزن مستقل با نسخه‌بندی» است؛ subtree برای «کد داخل پروژه».

**کاربرد عملی:** اگر کتابخانه‌ی داخلی داری که در چند پروژه استفاده می‌شود، این فصل تصمیم درست را می‌دهد.

### مفاهیم اساسی

#### Submodule

```
main-repo/
├── .gitmodules
├── src/
└── libs/shared/          ← submodule (اشاره به commit مشخص)
    └── .git               ← فایل، نه پوشه (اشاره به ../.git/modules/...)
```

`.gitmodules`:

```ini
[submodule "libs/shared"]
    path = libs/shared
    url = https://github.com/myorg/shared.git
    branch = main
```

main repo فقط **هش commit** را ذخیره می‌کند، نه محتوا.

#### Subtree

```
main-repo/
├── libs/shared/          ← محتوای کامل، در تاریخ main repo
└── (بدون .gitmodules)
```

محتوا کپی می‌شود، با تاریخ جدا (که در `git log` دیده می‌شود).

#### تفاوت بنیادی

| | Submodule | Subtree |
|---|---|---|
| محتوا | اشاره‌گر | کپی |
| `.gitmodules` | ✅ | ❌ |
| clone | نیاز به `--recursive` | ✅ معمولی |
| به‌روزرسانی | `submodule update` | `subtree pull` |
| تاریخ | جدا | در تاریخ اصلی |
| fork آسان | ❌ | ✅ |
| مناسب | کتابخانه‌ی مستقل | کد vendored |

### مثال‌های عملی

#### مثال ۱: افزودن submodule

```bash
git submodule add https://github.com/myorg/shared.git libs/shared
git commit -m "chore: افزودن shared به‌عنوان submodule"

# .gitmodules ساخته شد
cat .gitmodules
```

#### مثال ۲: clone با submodule

```bash
git clone --recurse-submodules https://github.com/myorg/app.git

# یا اگر قبلاً clone کردی:
git submodule update --init --recursive
```

**تله‌ی کلاسیک:** بدون `--recursive`، پوشه‌ی submodule **خالی** است. خیلی‌ها فکر می‌کنند کد گم شده.

#### مثال ۳: به‌روزرسانی submodule

```bash
cd libs/shared
git fetch
git checkout v2.1.0
cd ../..
git add libs/shared
git commit -m "chore(deps): ارتقای shared به v2.1.0"
```

**نکته:** `git add libs/shared` هش جدید را ثبت می‌کند. بدون آن، commit نمی‌شود.

#### مثال ۴: به‌روزرسانی همه

```bash
git submodule update --remote --merge
# یا
git submodule foreach 'git pull origin main'
```

`--remote` از branch تعریف‌شده در `.gitmodules` می‌گیرد.

#### مثال ۵: حذف submodule

```bash
git submodule deinit -f libs/shared
git rm -f libs/shared
rm -rf .git/modules/libs/shared
git commit -m "chore: حذف submodule"
```

**چهار مرحله لازم است** — یکی را جا بیندازی، مشکل می‌شود.

#### مثال ۶: تغییر URL

```bash
git config -f .gitmodules submodule.libs/shared.url git@github.com:myorg/shared.git
git submodule sync
git commit -am "chore: تغییر URL submodule"
```

#### مثال ۷: افزودن subtree

```bash
git subtree add --prefix=libs/shared https://github.com/myorg/shared.git main --squash
```

`--squash` تاریخ subtree را به یک commit خلاصه می‌کند (ساده‌تر).

#### مثال ۸: به‌روزرسانی subtree

```bash
git subtree pull --prefix=libs/shared https://github.com/myorg/shared.git main --squash
```

#### مثال ۹: push تغییرات به subtree

اگر در subtree تغییر دادی و می‌خواهی به مخزن اصلی بفرستی:

```bash
git subtree push --prefix=libs/shared https://github.com/myorg/shared.git feature/x
```

**هشدار:** این کار تاریخ را بازسازی می‌کند و ممکن است conflict بدهد. بهتر: تغییر را در مخزن اصلی بده و pull کن.

#### مثال ۱۰: submodule در CI

```yaml
- uses: actions/checkout@<sha>
  with:
    submodules: recursive
    token: ${{ secrets.SUBMODULE_TOKEN }}    # اگر خصوصی است
```

**بدون `submodules: recursive`**، build شکست می‌خورد (کد نیست).

#### مثال ۱۱: submodule خصوصی

اگر submodule خصوصی است، `GITHUB_TOKEN` به آن دسترسی ندارد (فقط به مخزن خودش). باید PAT یا GitHub App بدهی:

```yaml
- uses: actions/checkout@<sha>
  with:
    submodules: recursive
    token: ${{ secrets.PAT_FOR_SUBMODULES }}
```

#### مثال ۱۲: بررسی وضعیت

```bash
git submodule status
#  3f2a1b9c4d5e... libs/shared (v2.1.0)
# +e4f5g6h7i8j9... libs/other (v1.0.0-3-ge4f5g6h)
#  ↑ + یعنی commit محلی با آنچه ثبت شده فرق دارد
```

### مقایسه و الگوهای مختلف

#### Submodule در برابر Subtree در برابر پکیج

| | Submodule | Subtree | پکیج (npm/pip) |
|---|---|---|---|
| نسخه‌بندی | commit | commit | ✅ semver |
| به‌روزرسانی | دستی | دستی | ✅ خودکار |
| fork آسان | ❌ | ✅ | ✅ |
| تغییر هم‌زمان | ✅ | ❌ | ❌ |
| مناسب | مخزن مشترک | کد vendored | کتابخانه |

**توصیه‌ی کلی:** اگر می‌شود، **پکیج** کن (npm/pip registry خصوصی). submodule و subtree برای وقتی است که پکیج ممکن نیست.

#### Submodule در برابر monorepo

| | Submodule | Monorepo |
|---|---|---|
| استقلال | ✅ | ❌ |
| هم‌زمانی تغییر | ✅ | ✅ |
| CI | پیچیده | ✅ یکپارچه |
| مناسب | تیم‌های جدا | یک تیم |

#### `--squash` در subtree

| | با squash | بدون |
|---|---|---|
| تاریخ | یک commit | کامل |
| حجم | کم | زیاد |
| `git log` | ساده | پیچیده |
| توصیه | ✅ | فقط اگر تاریخ مهم است |

### اشکال شایع

#### ۱. clone بدون `--recursive`

پوشه‌ی submodule خالی است. `git submodule update --init --recursive`.

#### ۲. commit نکردن تغییر submodule

```bash
cd libs/shared && git checkout v2.0.0
cd ../.. && git commit -am "..."   # ❌ submodule ثبت نمی‌شود با -a
```

باید `git add libs/shared` صریح.

#### ۳. detached HEAD در submodule

submodule به‌طور پیش‌فرض در detached HEAD است. اگر می‌خواهی روی branch کار کنی:

```bash
cd libs/shared
git switch main
```

#### ۴. حذف ناقص submodule

اگر فقط `git rm` بزنی، `.git/modules` می‌ماند و بعداً مشکل می‌سازد. چهار مرحله (مثال ۵).

#### ۵. submodule خصوصی در CI

`GITHUB_TOKEN` دسترسی ندارد. PAT بده.

#### ۶. subtree push که conflict می‌دهد

تاریخ بازسازی می‌شود. راه‌حل: تغییر را در مخزن اصلی بده.

#### ۷. `.gitmodules` با URL نسبی

```ini
url = ../shared.git
```

نسبی است به remote، نه local. گاهی گیج‌کننده.

#### ۸. submodule که به‌روز نمی‌شود

```bash
git submodule update --remote
```

بدون `--remote`، فقط commit ثبت‌شده را checkout می‌کند (که ممکن است قدیمی باشد).

### نکات مهم (Tips & Tricks)

- **اول پکیج، بعد submodule/subtree.**
- **`--recurse-submodules`** در clone.
- **`git submodule status`** برای دیدن وضعیت.
- **`git add` صریح** برای submodule.
- **`submodules: recursive`** در Actions.
- **subtree با `--squash`** ساده‌تر.

### خلاصه فصل

- submodule = اشاره‌گر به commit؛ subtree = کپی محتوا.
- submodule نیاز به `--recursive` در clone.
- subtree در fork آسان‌تر، در به‌روزرسانی سخت‌تر.
- **اگر می‌شود، پکیج کن.**
- submodule خصوصی در CI نیاز به PAT دارد.

**سؤالات مروری:**
1. تفاوت بنیادی submodule و subtree چیست؟
2. چرا clone بدون `--recursive` مشکل می‌سازد؟
3. چرا پکیج بهتر از submodule است؟

---

## فصل ۸۶: Git LFS — فایل‌های بزرگ

### مقدمه

Git برای فایل‌های متنی طراحی شده. یک فایل ۱۰۰MB (PSD، ویدیو، dataset) که ۱۰ بار تغییر کند، ۱GB به تاریخ اضافه می‌کند — و clone برای همه کند می‌شود.

**Git LFS** (Large File Storage) فایل‌های بزرگ را بیرون از مخزن نگه می‌دارد و فقط یک اشاره‌گر کوچک در Git می‌گذارد.

**چرا مهم است؟** GitHub برای فایل بیش از ۱۰۰MB خطا می‌دهد و مخزن بیش از ۱GB را توصیه نمی‌کند. LFS این محدودیت را دور می‌زند (با محدودیت خودش).

**کاربرد عملی:** برای assetهای طراحی، مدل‌های ML، ویدیو، dataset.

### مفاهیم اساسی

#### چطور کار می‌کند

```
Git repo:     assets/logo.psd → یک فایل متنی کوچک:
              version https://git-lfs.github.com/spec/v1
              oid sha256:abc123...
              size 104857600

LFS storage:  ذخیره‌ی واقعی فایل (روی سرور LFS)
```

#### محدودیت‌های GitHub LFS

| | Free | Pro | Team | Enterprise |
|---|---|---|---|---|
| Storage | ۱GB | ۲GB | ۲GB/کاربر | ۲۵۰GB |
| Bandwidth | ۱GB/ماه | ۲GB/ماه | ۲GB/کاربر | ۲۵۰GB |
| فایل اضافه | پولی | پولی | پولی | پولی |

**نکته:** LFS رایگان نیست بعد از سهمیه. برای پروژه‌ی asset-heavy، هزینه دارد.

#### `.gitattributes`

```gitattributes
*.psd filter=lfs diff=lfs merge=lfs -text
*.mp4 filter=lfs diff=lfs merge=lfs -text
*.zip filter=lfs diff=lfs merge=lfs -text
```

### مثال‌های عملی

#### مثال ۱: نصب و راه‌اندازی

```bash
# نصب
brew install git-lfs          # macOS
sudo apt install git-lfs      # Debian/Ubuntu
winget install GitHub.GitLFS  # Windows

# فعال‌سازی (یک بار)
git lfs install
# Updated Git hooks.
# Git LFS initialized.

# در مخزن
cd my-repo
git lfs track "*.psd"
git lfs track "*.mp4"
git add .gitattributes
git commit -m "chore: فعال‌سازی LFS برای assetها"
```

#### مثال ۲: افزودن فایل

```bash
git add design/hero.psd
git commit -m "feat: افزودن تصویر hero"
git push
# Uploading LFS objects: 100% (1/1), 45 MB
```

Git خودش تشخیص می‌دهد (چون `.gitattributes` دارد).

#### مثال ۳: بررسی

```bash
git lfs ls-files
# 3f2a1b9c4d - design/hero.psd

git lfs status
git lfs env
```

#### مثال ۴: clone

```bash
# LFS خودکار دانلود می‌شود (اگر نصب باشد)
git clone https://github.com/myorg/design.git

# اگر LFS نصب نیست، فایل‌ها اشاره‌گر می‌مانند
# راه‌حل:
git lfs install
git lfs pull
```

#### مثال ۵: clone بدون LFS (سریع)

```bash
GIT_LFS_SKIP_SMUDGE=1 git clone https://github.com/myorg/big-repo.git
# فایل‌ها اشاره‌گر می‌مانند — برای CI که asset لازم ندارد
```

#### مثال ۶: fetch فقط فایل‌های لازم

```bash
git lfs fetch --include "design/hero.psd"
git lfs pull --include "design/*.psd"
```

#### مثال ۷: مهاجرت مخزن موجود به LFS

```bash
# نصب git-lfs-migrate
# یا استفاده از git filter-repo

git lfs migrate import --include="*.psd,*.mp4" --everything

# این تاریخ را بازنویسی می‌کند!
git push --force --all
```

**هشدار:** تاریخ بازنویسی می‌شود. همه باید دوباره clone کنند.

#### مثال ۸: LFS در Actions

```yaml
- uses: actions/checkout@<sha>
  with:
    lfs: true
```

**نکته:** `lfs: true` فایل‌ها را دانلود می‌کند (کند برای مخزن بزرگ). اگر لازم نیست، `false` (پیش‌فرض).

#### مثال ۹: پاک کردن LFS cache محلی

```bash
git lfs prune
# حذف objectهای قدیمی که در commit فعلی نیستند
```

#### مثال ۱۰: اگر سهمیه تمام شد

```
batch response: This repository is over its data quota.
```

راه‌حل‌ها:

1. خرید data pack
2. حذف فایل‌های بزرگ از تاریخ (`git lfs migrate`)
3. استفاده از storage دیگر (S3 + `git-annex`، DVC)

#### مثال ۱۱: DVC به‌جای LFS برای dataset

اگر dataset بزرگ داری (چند GB):

```bash
pip install dvc
dvc init
dvc add data/dataset.csv
git add data/dataset.csv.dvc data/.gitignore
git commit -m "data: افزودن dataset"
```

DVC فایل را در S3/GCS/Drive نگه می‌دارد و فقط `.dvc` (کوچک) در Git است.

### مقایسه و الگوهای مختلف

#### LFS در برابر commit معمولی

| | LFS | معمولی |
|---|---|---|
| حجم مخزن | کم | زیاد |
| clone | سریع (بدون asset) | کند |
| سهمیه | محدود | ۱GB توصیه |
| diff | ❌ | ✅ (متن) |

#### LFS در برابر DVC

| | LFS | DVC |
|---|---|---|
| هدف | فایل بزرگ عمومی | dataset/مدل ML |
| storage | GitHub | S3/GCS/Drive |
| نسخه‌بندی | ✅ | ✅ |
| pipeline | ❌ | ✅ |
| هزینه | GitHub | storage خودت |

#### LFS در برابر release asset

| | LFS | release asset |
|---|---|---|
| در clone | ✅ | ❌ |
| نسخه‌بندی | ✅ | ✅ (per release) |
| حجم | سهمیه | ۲GB per file |
| مناسب | asset در کد | خروجی build |

### اشکال شایع

#### ۱. `.gitattributes` بعد از commit

اگر فایل را قبل از `git lfs track` commit کنی، در تاریخ معمولی می‌ماند. باید migrate کنی.

#### ۲. LFS نصب نیست

```
This repository is configured for Git LFS but 'git-lfs' was not found
```

`git lfs install`.

#### ۳. فایل‌ها اشاره‌گر ماندند

```bash
git lfs pull
```

#### ۴. سهمیه تمام شد

هزینه دارد. برنامه‌ریزی کن یا به storage دیگر برو.

#### ۵. `git lfs track` قبل از `git add .gitattributes`

بدون commit کردن `.gitattributes`، تنظیم اعمال نمی‌شود.

#### ۶. LFS در CI کند است

`lfs: false` اگر asset لازم نیست.

#### ۷. تاریخ بازنویسی بعد از migrate

همه باید دوباره clone کنند. اطلاع‌رسانی.

#### ۸. LFS و fork

forkها سهمیه‌ی جدا دارند. assetها کپی نمی‌شوند (اشاره‌گر می‌مانند).

### نکات مهم (Tips & Tricks)

- **`.gitattributes` را اول commit کن.**
- **`git lfs ls-files`** برای بررسی.
- **`GIT_LFS_SKIP_SMUDGE=1`** برای clone سریع.
- **`git lfs prune`** برای آزاد کردن فضا.
- **DVC برای dataset بزرگ.**
- **`lfs: false` در CI** اگر لازم نیست.

### خلاصه فصل

- LFS فایل بزرگ را بیرون مخزن نگه می‌دارد.
- `.gitattributes` تنظیم می‌کند.
- سهمیه‌ی رایگان محدود (۱GB) — بعدش پولی.
- `git lfs migrate` برای مهاجرت (با بازنویسی تاریخ).
- DVC برای dataset/مدل ML.
- در CI `lfs: true/false` را آگاهانه بگذار.

**سؤالات مروری:**
1. چرا فایل ۱۰۰MB در Git مشکل است؟
2. `.gitattributes` چه نقشی دارد؟
3. چه زمانی DVC بهتر از LFS است؟

---

## فصل ۸۷: Hooks — pre-commit، commit-msg و CI محلی

### مقدمه

Hook یک اسکریپت است که Git در نقاط مشخص اجرا می‌کند: قبل از commit، قبل از push، بعد از merge. برای lint، format، تست، و بررسی پیام commit.

**چرا مهم است؟** hook جلوی commit بد را **قبل از** رسیدن به CI می‌گیرد. بازخورد در ۲ ثانیه، نه ۵ دقیقه.

**کاربرد عملی:** یک hook خوب، ۸۰٪ خطاهای CI را قبل از push می‌گیرد.

### مفاهیم اساسی

#### hookهای مهم

| Hook | کِی | برای |
|---|---|---|
| `pre-commit` | قبل از commit | lint، format، تست سریع |
| `commit-msg` | بعد از نوشتن پیام | commitlint |
| `pre-push` | قبل از push | تست کامل، typecheck |
| `post-merge` | بعد از merge | `npm install` اگر lock عوض شد |
| `post-checkout` | بعد از checkout | همان |
| `pre-rebase` | قبل از rebase | محافظت |

#### مشکل: hook در `.git/hooks` commit نمی‌شود

`.git/hooks/*` در مخزن نیست. راه‌حل: ابزار مدیریت hook.

| ابزار | زبان | مزیت |
|---|---|---|
| **Husky** | Node | محبوب، ساده |
| **Lefthook** | Go | ✅ سریع، موازی |
| **pre-commit** | Python | اکوسیستم بزرگ |
| **simple-git-hooks** | Node | سبک |
| **overcommit** | Ruby | |

### مثال‌های عملی

#### مثال ۱: hook خام (بدون ابزار)

`.git/hooks/pre-commit`:

```bash
#!/usr/bin/env bash
set -euo pipefail

# فقط فایل‌های staged
FILES=$(git diff --cached --name-only --diff-filter=ACM | grep -E '\.(js|ts|jsx|tsx)$' || true)
[ -z "$FILES" ] && exit 0

echo "$FILES" | xargs npx eslint --fix
echo "$FILES" | xargs npx prettier --write
git add $FILES
```

```bash
chmod +x .git/hooks/pre-commit
```

**عیب:** commit نمی‌شود، هر نفر باید دستی بسازد.

#### مثال ۲: Husky

```bash
npm install -D husky
npx husky init
```

`.husky/pre-commit`:

```bash
npx lint-staged
```

`.husky/commit-msg`:

```bash
npx --no -- commitlint --edit "$1"
```

`package.json`:

```json
{
  "lint-staged": {
    "*.{js,ts,jsx,tsx}": ["eslint --fix", "prettier --write"],
    "*.{json,md,yml,yaml}": ["prettier --write"]
  }
}
```

#### مثال ۳: Lefthook

```bash
npm install -D lefthook
npx lefthook install
```

`lefthook.yml`:

```yaml
pre-commit:
  parallel: true
  commands:
    lint:
      glob: "*.{js,ts,jsx,tsx}"
      run: npx eslint --fix {staged_files}
      stage_fixed: true
    format:
      glob: "*.{js,ts,jsx,tsx,json,md,yml}"
      run: npx prettier --write {staged_files}
      stage_fixed: true
    secrets:
      run: npx gitleaks protect --staged --redact

commit-msg:
  commands:
    commitlint:
      run: npx commitlint --edit {1}

pre-push:
  parallel: true
  commands:
    typecheck:
      run: npm run typecheck
    test:
      run: npm test -- --onlyChanged
```

**مزیت Lefthook:** سریع (Go)، موازی، `stage_fixed` خودکار.

#### مثال ۴: pre-commit (Python)

`.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-json
      - id: check-added-large-files
        args: ["--maxkb=1000"]
      - id: detect-private-key

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.7.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/rhysd/actionlint
    rev: v1.7.7
    hooks:
      - id: actionlint

  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.21.2
    hooks:
      - id: gitleaks
```

```bash
pip install pre-commit
pre-commit install
pre-commit install --hook-type commit-msg
pre-commit run --all-files
```

#### مثال ۵: pre-push با تست

```yaml
pre-push:
  commands:
    test:
      run: npm test
```

**هشدار:** اگر تست ۵ دقیقه است، هر push کند می‌شود. برای تست سریع (`--onlyChanged`) یا فقط `typecheck`.

#### مثال ۶: post-merge برای نصب خودکار

`.husky/post-merge`:

```bash
#!/usr/bin/env bash
# اگر lock عوض شد، نصب کن
if git diff HEAD@{1} HEAD --name-only | grep -qE 'package-lock.json|pnpm-lock.yaml'; then
  echo "lock عوض شد، نصب وابستگی‌ها..."
  npm ci
fi
```

#### مثال ۷: bypass کردن (اضطراری)

```bash
git commit --no-verify -m "..."
git push --no-verify
```

**قاعده:** فقط در اضطراری، و بعداً fix کن. اگر زیاد استفاده کنی، hook بی‌فایده است.

#### مثال ۸: hook در CI (برای کسانی که bypass می‌کنند)

```yaml
  hooks:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
      - uses: actions/setup-node@<sha>
        with: { node-version: 22, cache: npm }
      - run: npm ci
      - run: npx lefthook run pre-commit --all-files
```

#### مثال ۹: hook برای بررسی branch

`.husky/pre-commit`:

```bash
#!/usr/bin/env bash
BRANCH=$(git branch --show-current)
if [ "$BRANCH" = "main" ]; then
  echo "❌ commit مستقیم روی main ممنوع است. branch بساز."
  exit 1
fi
```

**مزیت:** جلوگیری از اشتباه رایج.

#### مثال ۱۰: hook برای secret

```bash
# gitleaks در pre-commit
npx gitleaks protect --staged --redact --verbose
```

اگر secret پیدا شد، commit متوقف می‌شود — **قبل از** اینکه لو برود.

### مقایسه و الگوهای مختلف

#### Husky در برابر Lefthook در برابر pre-commit

| | Husky | Lefthook | pre-commit |
|---|---|---|---|
| زبان | Node | Go | Python |
| سرعت | متوسط | ✅ سریع | متوسط |
| موازی | ❌ | ✅ | ❌ |
| اکوسیستم | npm | متنوع | ✅ بزرگ |
| مناسب | پروژه‌ی Node | هر پروژه | پروژه‌ی Python |

#### `pre-commit` در برابر `pre-push`

| | pre-commit | pre-push |
|---|---|---|
| سرعت | باید < ۵ ثانیه | می‌تواند کندتر |
| برای | lint، format، secret | تست، typecheck |
| اگر کند | توسعه‌دهنده bypass می‌کند | قابل تحمل |

#### `lint-staged` در برابر همه‌ی فایل‌ها

| | staged | همه |
|---|---|---|
| سرعت | ✅ | کند |
| خطاهای قبلی | نادیده | ✅ می‌گیرد |
| مناسب | pre-commit | CI |

### اشکال شایع

#### ۱. hook کند است

اگر ۳۰ ثانیه طول بکشد، همه `--no-verify` می‌زنند. **pre-commit باید < ۵ ثانیه باشد.**

#### ۲. hook در `.git/hooks` که commit نمی‌شود

هر نفر باید دستی بسازد. از ابزار استفاده کن.

#### ۳. `npm install` که hook را نمی‌سازد

اگر `prepare` script نداشته باشی:

```json
{
  "scripts": {
    "prepare": "husky"
  }
}
```

#### ۴. hook در CI اجرا نمی‌شود

CI از `.git/hooks` استفاده نمی‌کند. همان بررسی را در CI هم بگذار.

#### ۵. `stage_fixed` در Lefthook

بدون `stage_fixed: true`، فایل fix شده ولی stage نمی‌شود و در commit نمی‌رود.

#### ۶. hook که همه‌ی فایل‌ها را بررسی می‌کند

اگر ۵۰۰۰ فایل داری، pre-commit ۲ دقیقه می‌شود. `lint-staged` (فقط staged).

#### ۷. bypass عادت

اگر هر روز `--no-verify` می‌زنی، hook را حذف کن یا سریعش کن.

#### ۸. hook روی ویندوز

اسکریپت bash در ویندوز نیاز به Git Bash دارد. Husky این را مدیریت می‌کند.

### نکات مهم (Tips & Tricks)

- **pre-commit < ۵ ثانیه** — وگرنه bypass می‌شود.
- **`lint-staged`** فقط فایل‌های staged.
- **`stage_fixed: true`** در Lefthook.
- **gitleaks در pre-commit** — جلوگیری از نشت.
- **همان بررسی در CI** — برای bypassکننده‌ها.
- **`--no-verify` فقط اضطراری.**

### خلاصه فصل

- Hook در نقاط مشخص Git اجرا می‌شود.
- `.git/hooks` commit نمی‌شود → از Husky/Lefthook/pre-commit.
- pre-commit سریع (< ۵s)، pre-push کندتر.
- `lint-staged` فقط staged.
- gitleaks در pre-commit.
- همان بررسی در CI هم لازم است.

**سؤالات مروری:**
1. چرا hook در `.git/hooks` کافی نیست؟
2. چرا pre-commit باید سریع باشد؟
3. چرا همان بررسی در CI لازم است؟

---

## فصل ۸۸: امضای commit و tag

### مقدمه

یک commit می‌گوید «علی این تغییر را داد» — ولی هیچ‌کس این را تأیید نکرده. هر کسی می‌تواند `user.name` را عوض کند و به نام دیگری commit بزند. **امضا** این را حل می‌کند.

**چرا مهم است؟** برای پروژه‌های متن‌باز حساس، releaseها، و سازمان‌هایی که compliance دارند. GitHub نشان «Verified» می‌دهد و می‌توانی ruleset بسازی که فقط commitهای امضاشده merge شوند.

**کاربرد عملی:** بعد از این فصل، commitهایت با نشان سبز Verified ظاهر می‌شوند.

### مفاهیم اساسی

#### سه روش امضا

| روش | کلید | مزیت | عیب |
|---|---|---|---|
| **GPG** | GPG key | استاندارد قدیمی | پیچیده |
| **SSH** | SSH key | ✅ ساده (همان کلید) | نیاز به Git 2.34+ |
| **S/MIME** | X.509 | سازمانی | پیچیده |

**توصیه:** SSH signing — همان کلید SSH که برای GitHub داری، بدون تنظیم اضافه.

#### چه چیزی امضا می‌شود

- **commit:** محتوا + متادیتا + والدها
- **tag (annotated):** نام + پیام + commit

**lightweight tag امضا نمی‌شود.**

#### نشان Verified

GitHub سه حالت نشان می‌دهد:

| نشان | معنی |
|---|---|
| **Verified** | امضا معتبر و کلید متعلق به حساب است |
| **Partially verified** | امضا معتبر ولی کلید به حساب وصل نیست |
| **Unverified** | امضا نامعتبر یا وجود ندارد |

### مثال‌های عملی

#### مثال ۱: امضا با SSH (توصیه‌شده)

```bash
# ۱) کلید SSH موجود (از فصل ۴)
ls ~/.ssh/id_ed25519.pub

# ۲) تنظیم Git
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub

# ۳) امضا به‌طور پیش‌فرض
git config --global commit.gpgsign true
git config --global tag.gpgsign true

# ۴) کلید را در GitHub به‌عنوان "Signing Key" اضافه کن
# Settings → SSH and GPG keys → New SSH key → Key type: Signing Key

# ۵) تست
git commit --allow-empty -m "test: بررسی امضا"
git log --show-signature -1
```

**توجه:** کلید باید **دو بار** در GitHub ثبت شود: یک‌بار به‌عنوان Authentication Key (برای push)، یک‌بار به‌عنوان Signing Key (برای امضا). همان کلید، دو نقش.

#### مثال ۲: امضا با GPG

```bash
# ۱) کلید بساز
gpg --full-generate-key
# RSA 4096، 2y، نام و ایمیل (باید با GitHub یکی باشد)

# ۲) key ID را بگیر
gpg --list-secret-keys --keyid-format=long
# sec   rsa4096/3F2A1B9C4D5E6F7A 2026-10-23

# ۳) export public key
gpg --armor --export 3F2A1B9C4D5E6F7A
# -----BEGIN PGP PUBLIC KEY BLOCK-----
# ...

# ۴) در GitHub ثبت کن: Settings → SSH and GPG keys → New GPG key

# ۵) تنظیم Git
git config --global user.signingkey 3F2A1B9C4D5E6F7A
git config --global commit.gpgsign true
```

#### مثال ۳: امضای tag

```bash
# annotated tag (امضا در tag است)
git tag -s v1.4.0 -m "نسخه ۱.۴.۰"

# بررسی
git tag -v v1.4.0
```

**نکته:** `-s` (sign) نه `-a` (annotate). `-a` فقط متادیتا دارد، بدون امضا.

#### مثال ۴: بررسی امضاها

```bash
# یک commit
git log --show-signature -1

# همه‌ی commitها
git log --show-signature

# با فرمت
git log --pretty="%h %G? %s" -20
# %G? : G=good, B=bad, U=unknown, N=no signature, E=error
```

#### مثال ۵: `git verify-commit`

```bash
git verify-commit HEAD
# gpg: Signature made ...
# gpg: Good signature from "علی رضایی <ali@example.com>"
```

#### مثال ۶: امضای commit موجود (rebase)

اگر commitهای قبلی امضا ندارند:

```bash
# بازنویسی با امضا
git rebase --exec 'git commit --amend --no-edit -S' HEAD~5
```

**هشدار:** تاریخ را بازنویسی می‌کند. فقط برای branch شخصی.

#### مثال ۷: Ruleset برای commit امضاشده

```
Ruleset → Require signed commits
```

حالا هر PR بدون امضا، merge نمی‌شود.

**تله:** Copilot Cloud Agent و Dependabot ممکن است امضا نداشته باشند. باید به bypass اضافه شوند (یا GitHub خودش امضا می‌زند — بسته به نسخه).

#### مثال ۸: امضای SSH با کلید جداگانه

اگر نمی‌خواهی کلید اصلی را برای امضا استفاده کنی:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_signing -C "signing@example.com"
git config --global user.signingkey ~/.ssh/id_ed25519_signing.pub
```

و کلید **public** را در GitHub به‌عنوان Signing Key ثبت کن.

#### مثال ۹: `allowed_signers` (برای verify محلی)

برای اینکه `git log --show-signature` بتواند امضای SSH را verify کند:

`~/.config/git/allowed_signers`:

```
ali@example.com ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI...
```

```bash
git config --global gpg.ssh.allowedSignersFile ~/.config/git/allowed_signers
```

#### مثال ۱۰: امضا در CI

```yaml
- name: وارد کردن کلید
  env:
    SIGNING_KEY: ${{ secrets.SSH_SIGNING_KEY }}
  run: |
    mkdir -p ~/.ssh
    echo "$SIGNING_KEY" > ~/.ssh/id_ed25519_signing
    chmod 600 ~/.ssh/id_ed25519_signing
    git config --global gpg.format ssh
    git config --global user.signingkey ~/.ssh/id_ed25519_signing
    git config --global commit.gpgsign true
```

**نکته:** commitهایی که Actions می‌زند (مثل release) با `GITHUB_TOKEN`، معمولاً «Unverified» هستند. برای Verified، باید با GitHub App یا کلید امضا کنی.

#### مثال ۱۱: بررسی همه‌ی commitهای یک PR

```bash
gh pr view 42 --json commits --jq '.commits[] | {oid: .oid[0:7], verified: .authors[0].login}'

# یا
git log --pretty="%h %G? %s" origin/main..HEAD
```

### مقایسه و الگوهای مختلف

#### SSH در برابر GPG

| | SSH | GPG |
|---|---|---|
| تنظیم | ✅ ساده | پیچیده |
| کلید موجود | ✅ استفاده کن | باید بسازی |
| پشتیبانی GitHub | ✅ | ✅ |
| پشتیبانی ابزار | خوب | ✅ گسترده |
| مناسب | اکثر | محیط GPG-محور |

#### `commit.gpgsign` در برابر `-S`

| | config | `-S` |
|---|---|---|
| همه‌ی commitها | ✅ | ❌ |
| کنترل | ❌ | ✅ per-commit |

#### Verified در برابر Unverified

| | Verified | Unverified |
|---|---|---|
| امضا معتبر | ✅ | ❌ |
| کلید در حساب | ✅ | ❌ |
| Ruleset | ✅ پاس | ❌ رد |

### اشکال شایع

#### ۱. کلید فقط به‌عنوان Authentication ثبت شده

```
Unverified — the email in this signature doesn't match the committer email
```

یا نشان Verified نمی‌آید. کلید باید **هم** Authentication **هم** Signing باشد (دو ثبت جدا).

#### ۲. ایمیل نمی‌خواند

```bash
git config user.email
# باید با ایمیل حساب GitHub (یا noreply) یکی باشد
```

#### ۳. `gpg: signing failed: Inappropriate ioctl for device`

```bash
export GPG_TTY=$(tty)
# در .bashrc بگذار
```

#### ۴. passphrase هر بار

```bash
# gpg-agent با cache
echo "default-cache-ttl 3600" >> ~/.gnupg/gpg-agent.conf
gpg-connect-agent reloadagent /bye
```

یا کلید SSH بدون passphrase (نه توصیه‌شده) یا با `ssh-agent`.

#### ۵. کلید منقضی شده

```
gpg: Note: This key has expired!
```

تمدید کن یا کلید جدید بساز.

#### ۶. Ruleset اجباری بدون اطلاع تیم

همه‌ی PRها block می‌شوند. **اول اطلاع‌رسانی، بعد اجبار.**

#### ۷. Copilot/Dependabot بدون امضا

اگر Ruleset «Require signed commits» دارد، این botها رد می‌شوند. به bypass اضافه کن یا از GitHub App استفاده کن.

#### ۸. `-s` در برابر `-a` برای tag

```bash
✗ git tag -a v1.0.0    # annotated، بدون امضا
✓ git tag -s v1.0.0    # signed
```

#### ۹. rebase امضاها را می‌شکند

هر rebase، commit جدید می‌سازد. اگر `commit.gpgsign` روشن باشد، دوباره امضا می‌شود. اگر نه، امضا گم می‌شود.

### نکات مهم (Tips & Tricks)

- **SSH signing** — ساده‌ترین راه.
- **کلید را دو بار ثبت کن** (Authentication + Signing).
- **`git log --pretty="%h %G?"`** برای بررسی سریع.
- **`gpg.format ssh`** در Git 2.34+.
- **`GPG_TTY`** برای GPG.
- **Ruleset را بعد از آموزش فعال کن.**

### خلاصه فصل

- امضا ثابت می‌کند commit از توست.
- سه روش: GPG، SSH (توصیه)، S/MIME.
- SSH: همان کلید، دو نقش در GitHub.
- `-s` برای tag امضاشده.
- `git log --show-signature` برای بررسی.
- Ruleset می‌تواند امضا را اجباری کند.

**سؤالات مروری:**
1. چرا SSH signing ساده‌تر از GPG است؟
2. چرا کلید باید دو بار در GitHub ثبت شود؟
3. تفاوت `git tag -a` و `git tag -s` چیست؟

---

**بعدی:** [`20-Troubleshooting.md`](20-Troubleshooting.md)

---

## منابع و مراجع

- `git worktree` — https://git-scm.com/docs/git-worktree
- Pro Git، فصل ۷.۱۱ (Submodules) — https://git-scm.com/book/fa/v2
- `git subtree` — https://github.com/git/git/blob/master/contrib/subtree/git-subtree.txt
- Git LFS — https://git-lfs.com/
- Git hooks — https://git-scm.com/docs/githooks
- Husky — https://typicode.github.io/husky/
- Lefthook — https://github.com/evilmartians/lefthook
- pre-commit — https://pre-commit.com/
- GitHub Docs: Signing commits — https://docs.github.com/en/authentication/managing-commit-signature-verification
- GitHub Docs: SSH commit signature verification — https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification
