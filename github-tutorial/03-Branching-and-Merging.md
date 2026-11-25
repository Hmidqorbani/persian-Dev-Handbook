# بخش ۰۳: شاخه‌زنی، ادغام و استراتژی‌ها

**مدت یادگیری**: ۵ تا ۶ ساعت
**سطح سختی**: مبتدی → متوسط
**پیش‌نیاز**: بخش ۰۲ (commit، سه ناحیه)

---

## فهرست محتوا

- [فصل ۱۰: branch — ارزان‌ترین ابزار دنیا](#فصل-۱۰)
- [فصل ۱۱: merge — fast-forward، three-way و حل تعارض](#فصل-۱۱)
- [فصل ۱۲: rebase — تاریخ خطی و قاعده‌ی طلایی](#فصل-۱۲)
- [فصل ۱۳: استراتژی‌های شاخه‌زنی — GitHub Flow، Git Flow، Trunk-based](#فصل-۱۳)
- [فصل ۱۴: انتخاب روش merge در Pull Request](#فصل-۱۴)

---

## فصل ۱۰: branch — ارزان‌ترین ابزار دنیا

### مقدمه

در فصل ۲ دیدی که branch فقط یک اشاره‌گر ۴۱ بایتی است. این فصل می‌گوید با همین اشاره‌گر چه کارهایی می‌شود کرد — و چرا کسانی که از branch نمی‌ترسند، سریع‌تر و با اطمینان بیشتری کار می‌کنند.

**چرا مهم است؟** بدون branch، هر تغییری مستقیم روی کد اصلی می‌رود. یعنی: نمی‌توانی چیزی را امتحان کنی و رد کنی؛ نمی‌توانی هم‌زمان روی دو ویژگی کار کنی؛ نمی‌توانی PR بزنی. branch پایه‌ی تمام همکاری است.

**کاربرد عملی:** تیم پنج‌نفره‌ای که به‌جای «همه روی main»، هر کس روی branch خودش کار می‌کند، به‌طور طبیعی conflict کمتری دارد و review معنادارتری می‌گیرد.

### مفاهیم اساسی

#### branch از کجا می‌آید

```bash
git branch feature          # از HEAD فعلی
git branch hotfix v1.0.0    # از یک تگ
git branch fix a1b2c3d      # از یک commit مشخص
git switch -c new-branch    # بساز و برو
git switch -c new origin/main  # از remote branch
```

#### branchهای محلی در برابر remote

| | محلی | remote-tracking | remote |
|---|---|---|---|
| نمونه | `main` | `origin/main` | `origin` (نام) |
| کجاست | `.git/refs/heads/` | `.git/refs/remotes/origin/` | سرور |
| قابل commit | ✅ | ❌ (فقط خواندن) | ❌ |
| چطور به‌روز می‌شود | با commit محلی | با `git fetch` | با `git push` |

```
محلی:            main → 3f2a1b9
remote-tracking: origin/main → 3f2a1b9
remote (سرور):   main → 3f2a1b9
```

`origin/main` یک **آینه‌ی محلی** از آخرین وضعیتی است که از سرور می‌دانی. تا وقتی `fetch` نزنی، ممکن است قدیمی باشد. این یکی از بزرگ‌ترین منابع سردرگمی است: «ولی من pull زدم!» — pull = fetch + merge، ولی اگر fetch نکردی، `origin/main` قدیمی است.

#### upstream — ردیابی branch

```bash
git switch -c feature
git push -u origin feature
# branch 'feature' set up to track 'origin/feature'
```

از این به بعد:

```bash
git push      # می‌داند کجا برود
git pull      # می‌داند از کجا بیاید
git status    # می‌گوید "ahead 2, behind 1"
```

بدون upstream، هر بار باید بنویسی `git push origin feature`.

#### عملیات روی branch

```bash
git branch                    # لیست محلی
git branch -r                 # لیست remote
git branch -a                 # همه
git branch -v                 # با آخرین commit
git branch -vv                # با upstream و وضعیت
git branch --merged main      # merge شده‌ها
git branch --no-merged main   # merge نشده‌ها

git branch -d old-feature     # حذف (اگر merge شده)
git branch -D old-feature     # حذف اجباری (اگر merge نشده)
git branch -m new-name        # تغییر نام branch جاری
git branch -m old new         # تغییر نام یک branch دیگر
```

#### `git switch` در برابر `git checkout`

| | `git switch` | `git checkout` |
|---|---|---|
| branch | `git switch main` | `git checkout main` |
| branch جدید | `git switch -c x` | `git checkout -b x` |
| detached | `git switch --detach <ref>` | `git checkout <ref>` |
| بازگرداندن فایل | ❌ (از `git restore` استفاده کن) | `git checkout -- file` |
| خطر | کم | زیاد (دو کار در یک دستور) |

از Git 2.23 به بعد، `switch` و `restore` جای `checkout` را گرفتند تا معنادارتر باشند. از `checkout` فقط برای سازگاری استفاده کن.

### مثال‌های عملی

#### مثال ۱: چرخه‌ی کامل یک ویژگی

```bash
cd ~/github-lab/shop
git switch main
git pull

# branch جدید
git switch -c feature/cart

# کار کن
echo "def add_to_cart(): pass" >> src/app.py
git add src/app.py
git commit -m "feat(cart): افزودن تابع سبد خرید"

# چند commit دیگر
echo "def remove_from_cart(): pass" >> src/app.py
git commit -am "feat(cart): افزودن حذف از سبد"

# برگرد به main
git switch main

# ببین feature جلو رفته
git log --oneline --graph --all -10
```

#### مثال ۲: کار موازی روی دو ویژگی

```bash
git switch main
git switch -c feature/search

# نیمه‌کاره رها کن، برو سراغ فوری
git switch main
git switch -c hotfix/login-500

echo "FIXED = True" >> src/app.py
git commit -am "fix(auth): رفع خطای 500 در ورود"

git switch main
git merge hotfix/login-500
git push

# برگرد سراغ ویژگی
git switch feature/search
```

این قدرت اصلی branch است: می‌توانی وسط کار، فوری به چیز دیگری بروی، بدون اینکه چیزی را ذخیره/بازیابی کنی (Git تغییرات commit‌نشده را نگه می‌دارد اگر با فایل‌های branch دیگر تعارض نداشته باشد).

**تله:** اگر در branch `feature/search` تغییرات commit‌نشده داشته باشی و به `hotfix` بروی، Git تغییرات را با خودش می‌برد. اگر روی همان فایل کار کنی، ممکن است قاطی شوند. راه امن: `git stash` (فصل ۲۳) یا commit موقت.

#### مثال ۳: branch از remote

```bash
git fetch origin
git switch -c feature/login origin/feature/login
# یا کوتاه‌تر (Git 2.23+):
git switch feature/login   # اگر فقط یک remote این نام را دارد
```

#### مثال ۴: پاک کردن branchهای مرده

```bash
# branchهای merge‌شده در main را ببین
git branch --merged main

# پاک کن (به‌جز main و develop)
git branch --merged main | grep -v -E '^\*|main|develop' | xargs -r git branch -d

# branchهای remote که روی سرور پاک شده‌اند
git fetch --prune
# یا یک‌بار برای همیشه:
git config --global fetch.prune true
```

### مقایسه و الگوهای مختلف

#### نام‌گذاری branch

| الگو | نمونه | مزیت |
|---|---|---|
| `type/description` | `feat/user-profile` | خوانا، ابزارپذیر |
| `type/ticket-description` | `feat/PROJ-123-user-profile` | اتصال به Issue tracker |
| `username/description` | `ali/user-profile` | در تیم‌های بزرگ |
| `description` | `user-profile` | ساده ولی مبهم |

**توصیه:** `type/short-description` با type از همان مجموعه‌ی Conventional Commits. GitHub خودش از Issue، branch با نام `123-issue-title` می‌سازد.

#### branchهای بلندعمر در برابر کوتاه‌عمر

| | کوتاه‌عمر (روز) | بلندعمر (هفته/ماه) |
|---|---|---|
| conflict | کم | زیاد و پیچیده |
| review | آسان | سخت |
| ادغام | ساده | دردناک |
| مناسب | ویژگی‌های کوچک | بازنویسی بزرگ (با feature flag) |

**قاعده:** هر branch بیشتر از ۲-۳ روز زنده بماند، احتمال conflict به‌طور تصاعدی بالا می‌رود. اگر کاری بزرگ است، آن را به قطعات کوچک بشکن که هر کدام مستقل merge شوند (feature flag).

### اشکال شایع

#### ۱. `error: Your local changes would be overwritten by checkout`

یعنی تغییرات commit‌نشده‌ای داری که با branch مقصد تعارض دارند. سه راه:

```bash
git stash              # ذخیره، بعد switch، بعد stash pop
git commit -m "wip"    # commit موقت
git restore .          # دور ریختن (خطرناک!)
```

#### ۲. branch ساختم ولی push نمی‌شود

```bash
git push -u origin feature/my-branch
```

بدون `-u`، Git نمی‌داند branch محلی به کدام remote branch وصل شود.

#### ۳. `git branch -d` خطا می‌دهد

```
error: The branch 'feature' is not fully merged.
```

یعنی commit‌هایی دارد که در branch جاری نیست. اگر مطمئنی لازم نیست:

```bash
git branch -D feature
# یا اول چک کن چه چیزی از دست می‌رود:
git log main..feature --oneline
```

#### ۴. روی branch اشتباه commit زدم

```bash
# فرض: روی main بودی، فکر می‌کردی روی feature هستی
git branch feature          # branch از موقعیت فعلی
git reset --hard HEAD~1     # main را عقب ببر
git switch feature          # commit حالا روی feature است
```

اگر چند commit زده‌ای، `HEAD~3` یا هش commit قبل از شروع کارت.

#### ۵. branch را حذف کردم و commit‌هایش رفت

`git reflog` (فصل ۲۲). تا وقتی garbage collection اجرا نشده، commit‌ها هستند.

#### ۶. `origin/main` به‌روز نیست

`git fetch` نزده‌ای. `pull` هم fetch می‌کند، ولی اگر فقط `status` می‌زنی، Git از سرور خبر ندارد.

### نکات مهم (Tips & Tricks)

- **`git switch -`** به branch قبلی برمی‌گردد (مثل `cd -`). برای جابه‌جایی سریع عالی است.
- **`git branch --sort=-committerdate`** جدیدترین branchها را اول نشان می‌دهد.
- **alias مفید:**
  ```bash
  git config --global alias.recent "branch --sort=-committerdate --format='%(committerdate:short) %(refname:short)'"
  ```
- **قبل از `git switch`، `git status` بزن.** عادت کوچکی که جلوی خیلی از «چرا فایلم عوض شد» را می‌گیرد.
- **branch را با نام Issue بساز** تا GitHub خودکار لینک کند.

### خلاصه فصل

- branch یک اشاره‌گر ارزان است؛ ساختن و حذفش تقریباً صفر هزینه دارد.
- **remote-tracking branch** (`origin/main`) آینه‌ی محلی از سرور است و فقط با `fetch` به‌روز می‌شود.
- **upstream** با `push -u` تنظیم می‌شود و `push`/`pull` را ساده می‌کند.
- `switch`/`restore` جانشین مدرن `checkout` هستند.
- branchهای کوتاه‌عمر conflict کمتری دارند.

**سؤالات مروری:**
1. تفاوت `main` و `origin/main` چیست و چه زمانی همگام می‌شوند؟
2. چطور یک commit که اشتباهاً روی `main` زده‌ای را به یک branch جدید منتقل کنی؟
3. چرا `git branch -d` گاهی خطا می‌دهد؟

---

## فصل ۱۱: merge — fast-forward، three-way و حل تعارض

### مقدمه

merge یعنی «تاریخ دو branch را به هم وصل کن». ساده به نظر می‌رسد، ولی ۹۰٪ ترس مردم از Git از همین‌جا می‌آید: conflict.

**چرا مهم است؟** conflict اجتناب‌ناپذیر است. کسی که بلد باشد conflict را درست حل کند، با اطمینان بیشتری branch می‌سازد و کد بیشتری merge می‌کند. کسی که نتواند، همه‌چیز را روی `main` می‌زند.

**کاربرد عملی:** در یک تیم، هر PR حداقل یک merge است. اگر روزی یک conflict حل کنی، در سال ۲۰۰ conflict حل کرده‌ای.

### مفاهیم اساسی

#### سه نوع merge

**۱. Fast-forward** — وقتی branch مقصد هیچ commit جدیدی ندارد:

```
قبل:  A ← B ← C        (main)
              ↖
               D ← E    (feature)

بعد:  A ← B ← C ← D ← E   (main و feature هر دو اینجا)
```

هیچ commit جدیدی ساخته نمی‌شود؛ فقط اشاره‌گر `main` جلو می‌رود. تاریخ خطی می‌ماند.

**۲. Three-way merge** — وقتی هر دو branch جلو رفته‌اند:

```
قبل:  A ← B ← C ← F      (main)
            ↖
             D ← E       (feature)

بعد:  A ← B ← C ← F ← M  (main)
            ↖       ↗
             D ← E

M = merge commit با دو والد (F و E)
```

Git از **سه نقطه** استفاده می‌کند: `B` (merge base)، `F` (نوک main)، `E` (نوک feature). برای هر فایل، تغییرات هر طرف نسبت به `B` را اعمال می‌کند.

**۳. Octopus merge** — ادغام بیش از دو branch با هم:

```bash
git merge feature-a feature-b feature-c
```

commit با سه والد. فقط وقتی کار می‌کند که conflict نباشد. در عمل کم استفاده می‌شود.

#### merge base چیست

```bash
git merge-base main feature
# هش commit مشترک نزدیک‌ترین
```

این همان نقطه‌ای است که دو branch از هم جدا شدند. تمام محاسبات merge بر مبنای آن است.

#### conflict چرا پیش می‌آید

وقتی **هر دو طرف یک خط را تغییر داده باشند**، Git نمی‌تواند تصمیم بگیرد:

```
base:     price = 100
main:     price = 120      ← تغییر کرده
feature:  price = 150      ← تغییر کرده
```

نتیجه:

```
<<<<<<< HEAD
price = 120
=======
price = 150
>>>>>>> feature
```

اگر فقط یک طرف تغییر کرده باشد، Git خودش آن را می‌گیرد. اگر هر دو یک **تغییر یکسان** داده باشند، Git هم قبول می‌کند. conflict فقط در تغییر متناقض است.

#### نشانگرهای conflict

```
<<<<<<< HEAD                    ← نسخه‌ی branch جاری (مثلاً main)
تغییرات من
||||||| merged common ancestors  ← فقط با conflictstyle=diff3
نسخه‌ی base
=======
تغییرات طرف دیگر
>>>>>>> feature                  ← نام branch دیگر
```

با `merge.conflictstyle=diff3` (فصل ۵)، نسخه‌ی base هم نمایش داده می‌شود. این خیلی کمک می‌کند، چون می‌فهمی **هر طرف چه چیزی را عوض کرد**، نه فقط نتیجه‌ی نهایی.

### مثال‌های عملی

#### مثال ۱: fast-forward

```bash
git switch main
git switch -c feature/a
echo "a" > a.txt && git add a.txt && git commit -m "feat: افزودن a"

git switch main
git merge feature/a
# Fast-forward
#  a.txt | 1 +
```

اگر نمی‌خواهی fast-forward (می‌خواهی merge commit داشته باشی تا رد branch بماند):

```bash
git merge --no-ff feature/a -m "merge: ادغام feature/a"
```

#### مثال ۲: three-way merge

```bash
git switch main
echo "main change" >> README.md
git commit -am "docs: تغییر در main"

git switch feature/a
echo "feature change" >> a.txt
git commit -am "feat: تغییر در feature"

git switch main
git merge feature/a
# Merge made by the 'ort' strategy.
```

#### مثال ۳: conflict کامل — از شروع تا پایان

```bash
# آماده‌سازی
git switch main
cat > config.txt <<'EOF'
port = 3000
host = localhost
EOF
git add config.txt && git commit -m "chore: config اولیه"

git switch -c feature/port
sed -i 's/port = 3000/port = 8080/' config.txt
git commit -am "chore: تغییر پورت به 8080"

git switch main
sed -i 's/port = 3000/port = 9090/' config.txt
git commit -am "chore: تغییر پورت به 9090"

# حالا merge
git merge feature/port
# Auto-merging config.txt
# CONFLICT (content): Merge conflict in config.txt
# Automatic merge failed; fix conflicts and then commit the result.

git status
# both modified: config.txt

cat config.txt
# <<<<<<< HEAD
# port = 9090
# =======
# port = 8080
# >>>>>>> feature/port
# host = localhost
```

**حل:**

```bash
# ۱) تصمیم بگیر: 8080، 9090، یا هر دو؟
cat > config.txt <<'EOF'
port = 8080
host = localhost
EOF

# ۲) فایل را stage کن (این یعنی "حل شد")
git add config.txt

# ۳) merge را کامل کن
git commit
# پیام پیش‌فرض merge باز می‌شود، ذخیره کن

git log --oneline --graph -5
```

#### مثال ۴: لغو یک merge نیمه‌کاره

```bash
git merge --abort
# همه‌چیز به حالت قبل از merge برمی‌گردد
```

این جان نجات است. هر وقت در conflict گیر کردی و خواستی از نو شروع کنی: `git merge --abort`.

#### مثال ۵: ابزارهای حل conflict

```bash
# استفاده از نسخه‌ی یک طرف به‌طور کامل
git checkout --ours config.txt     # نسخه‌ی branch جاری
git checkout --theirs config.txt   # نسخه‌ی branch دیگر
git add config.txt

# با merge tool گرافیکی
git mergetool
# یا مستقیم:
code config.txt    # VS Code سه‌پنجره‌ای نشان می‌دهد
```

VS Code بهترین تجربه را دارد: «Accept Current» / «Accept Incoming» / «Accept Both» / «Compare Changes».

#### مثال ۶: دیدن فایل در سه نسخه

```bash
# نسخه‌ی base (نقطه‌ی مشترک)
git show :1:config.txt

# نسخه‌ی ours
git show :2:config.txt

# نسخه‌ی theirs
git show :3:config.txt
```

این شماره‌ها (stages) در ایندکس هستند و ابزارهای merge از همین استفاده می‌کنند.

#### مثال ۷: merge با استراتژی مشخص

```bash
# همیشه یک merge commit بساز، حتی اگر fast-forward ممکن باشد
git merge --no-ff feature/x

# فقط fast-forward (اگر ممکن نبود خطا بده)
git merge --ff-only feature/x

# squash همه‌ی commitها به یکی
git merge --squash feature/x
git commit -m "feat: کل ویژگی x"
```

### مقایسه و الگوهای مختلف

#### fast-forward در برابر no-ff

| | `--ff` (پیش‌فرض) | `--no-ff` |
|---|---|---|
| commit اضافه | ندارد | merge commit |
| تاریخ | خطی | شاخه‌دار |
| رد branch | از بین می‌رود | می‌ماند |
| revert کل ویژگی | سخت (باید تک‌تک) | آسان (یک commit) |
| مناسب | branchهای کوتاه | ویژگی‌های بزرگ |

**توصیه:** برای PRها از `--no-ff` (یا squash) استفاده کن تا رد ویژگی بماند. برای branchهای موقت محلی، fast-forward خوب است.

#### squash در برابر merge

| | merge | squash |
|---|---|---|
| تاریخ branch | حفظ می‌شود | یکی می‌شود |
| commitهای «wip» | در تاریخ می‌مانند | حذف می‌شوند |
| attribution | هر commit | یک commit (با Co-authored-by) |
| bisect | دقیق‌تر | درشت‌تر |

#### ابزار حل conflict

| ابزار | مزیت | عیب |
|---|---|---|
| دستی (ادیتور) | کنترل کامل | کند |
| VS Code | سه‌پنجره‌ای، بصری | نیاز به تنظیم merge tool |
| `git mergetool` + meld/kdiff3 | بصری، قدرتمند | نصب جدا |
| `git checkout --ours/--theirs` | سریع برای «یکی را کامل بگیر» | خطرناک اگر اشتباه |

### اشکال شایع

#### ۱. نشانگرهای conflict را commit کردم

```
✗ <<<<<<< HEAD در کد production
```

پیشگیری: بعد از حل، `git grep -n "<<<<<<<"` بزن. یا در CI یک مرحله بگذار که این را چک کند (فصل ۵۷).

#### ۲. `git add` را زدم ولی merge تمام نشد

بعد از حل همه‌ی فایل‌ها، باید `git commit` بزنی. `add` فقط می‌گوید «این فایل حل شد».

#### ۳. `--ours` و `--theirs` را برعکس فهمیدم

- `--ours` = branchی که **رویش هستی** (مثلاً `main` وقتی `git merge feature` می‌زنی)
- `--theirs` = branchی که **merge می‌کنی** (`feature`)

این از دید توست، نه از دید Git. در rebase **برعکس** می‌شود (چون rebase نقش‌ها را عوض می‌کند) — یک تله‌ی معروف.

#### ۴. conflict در فایل باینری

```
warning: Cannot merge binary files: logo.png (HEAD vs. feature)
```

Git نمی‌تواند باینری را ادغام کند. باید یکی را انتخاب کنی:

```bash
git checkout --ours logo.png
git add logo.png
```

پیشگیری: در `.gitattributes` بنویس `*.png binary` و از LFS استفاده کن (فصل ۸۶).

#### ۵. conflict در `package-lock.json`

فاجعه‌ی رایج. راه‌حل:

```bash
git checkout --ours package-lock.json
npm install
git add package-lock.json
```

یا بهتر: در `.gitattributes` بنویس `package-lock.json -diff` تا حداقل diff نشان ندهد.

#### ۶. merge را اشتباه زدم و push کردم

```bash
git revert -m 1 <merge-commit-hash>
# -m 1 یعنی والد اول (branch اصلی) را نگه دار
```

**تله‌ی معروف:** اگر بعداً بخواهی همان branch را دوباره merge کنی، Git تغییرات را «قبلاً merge شده» می‌بیند و اعمال نمی‌کند. باید revert را revert کنی.

### نکات مهم (Tips & Tricks)

- **`git merge --abort`** را حفظ کن. اولین کاری که در conflict گیج‌شده باید بزنی.
- **`merge.conflictstyle=diff3`** را تنظیم کن (فصل ۵). تفاوتش شب و روز است.
- **قبل از merge، `git log --oneline main..feature`** بزن تا بدانی چه چیزی merge می‌شود.
- **`git merge --no-commit --no-ff feature`** merge را آماده می‌کند ولی commit نمی‌زند — برای بازبینی نهایی.
- **در conflict، به «کدام نسخه» فکر نکن؛ به «نتیجه‌ی درست» فکر کن.** گاهی جواب ترکیبی از هر دو است.
- **بعد از حل، تست‌ها را اجرا کن.** merge می‌تواند کد را کامپایل‌پذیر ولی غلط بگذارد.

### خلاصه فصل

- سه نوع merge: **fast-forward** (بدون commit جدید)، **three-way** (با merge commit)، **octopus**.
- Git از **merge base** برای محاسبه‌ی تغییرات هر طرف استفاده می‌کند.
- conflict فقط وقتی است که **هر دو طرف یک خط را متفاوت** تغییر داده باشند.
- `git merge --abort` هر merge نیمه‌کاره را لغو می‌کند.
- `--ours`/`--theirs` از دید branch جاری است (در rebase برعکس).

**سؤالات مروری:**
1. چه زمانی fast-forward ممکن است و چه زمانی نه؟
2. merge base چیست و چرا مهم است؟
3. چرا `git add` بعد از حل conflict کافی نیست؟

---

## فصل ۱۲: rebase — تاریخ خطی و قاعده‌ی طلایی

### مقدمه

rebase یکی از دو ابزار ادغام است (دیگری merge). تفاوت بنیادی: merge تاریخ را **حفظ** می‌کند، rebase تاریخ را **بازنویسی** می‌کند تا خطی شود.

**چرا مهم است؟** یک تاریخ خطی خواندنش آسان‌تر است: `git log --oneline` یک لیست تمیز است، نه یک جنگل. خیلی از تیم‌ها rebase را قبل از PR الزامی می‌کنند. ولی rebase ابزار خطرناکی است اگر قاعده‌اش را ندانی.

**کاربرد عملی:** سناریوی روزمره: روی `feature` کار می‌کنی، `main` جلو رفته. می‌خواهی feature را روی آخرین main ببری تا conflict را همین‌جا حل کنی، نه در PR.

### مفاهیم اساسی

#### rebase چه می‌کند

```
قبل:
        A ← B ← C ← D     (main)
             ↖
              E ← F        (feature)

git switch feature
git rebase main

بعد:
        A ← B ← C ← D          (main)
                     ↖
                      E' ← F'      (feature)
```

Git:
1. commitهای `E` و `F` را پیدا می‌کند (آن‌هایی که در `main` نیستند).
2. `feature` را به `D` منتقل می‌کند.
3. هر commit را **دوباره اعمال** می‌کند و commit **جدید** می‌سازد (`E'` و `F'`).

**نکته‌ی حیاتی:** `E'` یک commit **جدید** است، نه `E`. هش عوض می‌شود، تاریخ نویسنده ممکن است حفظ شود ولی committer date عوض می‌شود.

#### merge در برابر rebase — تصویری

```
merge:                          rebase:
  A ← B ← C ← D ← M               A ← B ← C ← D
       ↖       ↗                              ↖
        E ← F                                  E' ← F'
```

- merge: تاریخ واقعی (چه زمانی چه اتفاقی افتاد) حفظ می‌شود، ولی گراف شاخه‌دار می‌شود.
- rebase: تاریخ «داستان تمیز» می‌شود، ولی دیگر واقعی نیست (commitها در زمان دیگری بازنویسی شده‌اند).

#### قاعده‌ی طلایی rebase

> **هرگز branchی را rebase نکن که دیگران رویش کار می‌کنند.**

چرا؟ چون هش‌ها عوض می‌شوند. اگر تو `feature` را rebase کنی و push کنی، مریم که همان branch را دارد، commitهای قدیمی را دارد. حالا Git دو نسخه‌ی متفاوت از یک تاریخ دارد و merge فاجعه می‌شود (همان commitها دو بار ظاهر می‌شوند).

**استثنا:** اگر مطمئنی فقط تو روی branch کار می‌کنی، rebase آزاد است. و اگر push کرده‌ای، `--force-with-lease`.

#### rebase تعاملی (`-i`)

قدرتمندترین ابزار برای تمیز کردن تاریخ قبل از PR:

```bash
git rebase -i HEAD~4
```

ادیتور باز می‌شود:

```
pick a1b2c3d feat: افزودن فرم
pick e4f5g6h wip
pick i7j8k9l fix typo
pick m0n1o2p feat: اعتبارسنجی

# Commands:
# p, pick   = استفاده کن
# r, reword = استفاده کن، ولی پیام را عوض کن
# e, edit   = استفاده کن، ولی متوقف شو تا اصلاح کنم
# s, squash = با commit قبلی ادغام کن (پیام‌ها ترکیب می‌شوند)
# f, fixup  = با commit قبلی ادغام کن (پیام قبلی را نگه دار)
# x, exec   = اجرای دستور shell
# d, drop   = حذف کن
```

مثال: `wip` و `fix typo` را با commit قبلی squash کن:

```
pick a1b2c3d feat: افزودن فرم
fixup e4f5g6h wip
fixup i7j8k9l fix typo
pick m0n1o2p feat: اعتبارسنجی
```

نتیجه: دو commit تمیز.

#### `--autosquash` و `--fixup`

```bash
# وقتی commit می‌زنی و می‌دانی بعداً باید اصلاح شود
git commit --fixup=a1b2c3d

# بعداً
git rebase -i --autosquash main
# خودکار fixup را کنار هدفش می‌گذارد
```

### مثال‌های عملی

#### مثال ۱: به‌روز کردن feature با main

```bash
git switch feature/login
git fetch origin
git rebase origin/main

# اگر conflict بود:
# CONFLICT (content): Merge conflict in src/auth.js
# حل کن، بعد:
git add src/auth.js
git rebase --continue

# یا لغو:
git rebase --abort

# یا رد کردن این commit:
git rebase --skip
```

**نکته:** در rebase، بعد از حل هر conflict باید `git add` + `git rebase --continue`. Git یکی‌یکی commitها را اعمال می‌کند، پس ممکن است چند بار conflict بدهد.

#### مثال ۲: تمیز کردن تاریخ قبل از PR

```bash
git log --oneline main..HEAD
# m0n1o2p feat: افزودن جست‌وجو
# i7j8k9l wip
# e4f5g6h fix: غلط املایی
# a1b2c3d feat: ساختار جست‌وجو

git rebase -i main
```

در ادیتور:

```
pick a1b2c3d feat: ساختار جست‌وجو
fixup e4f5g6h fix: غلط املایی
fixup i7j8k9l wip
reword m0n1o2p feat: افزودن جست‌وجو
```

نتیجه: دو commit تمیز با پیام‌های خوب.

#### مثال ۳: تقسیم یک commit بزرگ

```bash
git rebase -i HEAD~1
# commit را از pick به edit تغییر بده

# حالا در حالت "stopped":
git reset HEAD~1              # commit را باز کن (تغییرات در working dir)
git add -p                    # بخش اول
git commit -m "feat: بخش اول"
git add -p                    # بخش دوم
git commit -m "feat: بخش دوم"
git rebase --continue
```

#### مثال ۴: حذف یک commit از وسط تاریخ

```bash
git rebase -i HEAD~5
# خط commit مورد نظر را از pick به drop تغییر بده
```

**هشدار:** اگر commitهای بعدی به آن وابسته باشند، conflict می‌دهد.

#### مثال ۵: rebase با `--onto` — جابه‌جایی branch

```bash
# فرض: feature از develop منشعب شده، ولی باید از main باشد
git rebase --onto main develop feature
# یعنی: commitهای feature که در develop نیستند را روی main بگذار
```

این یکی از پیشرفته‌ترین کاربردهاست و در فصل ۱۹ (Advanced Git) بیشتر می‌بینیم.

#### مثال ۶: push بعد از rebase

```bash
git push --force-with-lease
```

**چرا `--force-with-lease` و نه `--force`؟**

| | `--force` | `--force-with-lease` |
|---|---|---|
| رفتار | هر چیزی روی سرور را می‌نویسد | فقط اگر سرور همان چیزی باشد که تو آخرین بار دیدی |
| خطر | اگر کسی commit جدید push کرده باشد، پاک می‌شود | خطا می‌دهد، امن |
| توصیه | ❌ هرگز | ✅ همیشه |

### مقایسه و الگوهای مختلف

#### merge در برابر rebase — جدول تصمیم

| موقعیت | merge | rebase |
|---|---|---|
| branch مشترک تیمی | ✅ | ❌ |
| branch شخصی، قبل از PR | ❌ | ✅ |
| می‌خواهی تاریخ واقعی بماند | ✅ | ❌ |
| می‌خواهی تاریخ خوانا باشد | ❌ | ✅ |
| merge commit می‌خواهی | ✅ | ❌ |
| conflict را یک‌بار حل کنی | ❌ (هر بار) | ✅ (یک‌بار) |

#### سه رویکرد به «به‌روز کردن branch»

| رویکرد | دستور | نتیجه |
|---|---|---|
| **merge main into feature** | `git merge main` | merge commit، تاریخ شاخه‌دار |
| **rebase feature onto main** | `git rebase main` | تاریخ خطی، هش عوض |
| **merge با `--no-ff`** | `git merge --no-ff main` | همیشه merge commit صریح |

**توصیه‌ی رایج:** rebase محلی، merge در PR. یعنی: روی سیستم خودت feature را روی main rebase کن تا conflict را حل کنی، ولی وقتی PR merge می‌شود، از merge commit یا squash استفاده کن.

### اشکال شایع

#### ۱. rebase روی branch مشترک

```
✗ git rebase main روی branchی که ۳ نفر رویش کار می‌کنند
```

نتیجه: تاریخ دوگانه، commitهای تکراری، سردرگمی کامل. اگر اشتباه کردی، `git reflog` و برگرد.

#### ۲. `git push` بعد از rebase رد می‌شود

```
! [rejected] feature -> feature (non-fast-forward)
```

طبیعی است — تاریخ عوض شده. `git push --force-with-lease`.

#### ۳. conflict در rebase و `--ours`/`--theirs` برعکس است

در rebase، Git نقش‌ها را عوض می‌کند:
- `--ours` = branch مقصد (`main`)
- `--theirs` = commitی که در حال اعمالش است (از `feature`)

این تله خیلی‌ها را گول می‌زند.

#### ۴. rebase را نیمه‌کاره رها کردم

```bash
git status
# You are currently rebasing branch 'feature' on 'a1b2c3d'.
#   (fix conflicts and then run "git rebase --continue")
#   (use "git rebase --skip" to skip this patch)
#   (use "git rebase --abort" to check out the original branch)

git rebase --abort    # برگرد
# یا حل کن و continue
```

#### ۵. `git rebase -i` و ادیتور باز نمی‌شود

`core.editor` تنظیم نیست. `git config --global core.editor "code --wait"` یا `nano`.

#### ۶. بعد از rebase، commit گم شد

`git reflog` → هش قبل از rebase → `git reset --hard <hash>`.

### نکات مهم (Tips & Tricks)

- **`git rebase -i` قبل از هر PR** — تاریخ تمیز، review راحت‌تر.
- **`--force-with-lease` همیشه، `--force` هرگز.**
- **`git config --global rebase.autoStash true`** — تغییرات نیمه‌کاره خودکار stash می‌شوند.
- **`git rebase --exec "npm test" main`** — بعد از هر commit تست اجرا می‌کند. برای اطمینان از اینکه هر commit مستقل سالم است.
- **`git pull --rebase`** — pull بدون merge commit. برای تیم‌هایی که تاریخ خطی می‌خواهند:
  ```bash
  git config --global pull.rebase true
  ```
- **قبل از rebase یک branch پشتیبان بساز:** `git branch feature-backup`. ارزان است و آرامش می‌دهد.

### خلاصه فصل

- rebase commitها را روی یک base جدید **بازسازی** می‌کند؛ هش‌ها عوض می‌شوند.
- **قاعده‌ی طلایی:** branch مشترک را rebase نکن.
- `rebase -i` برای squash، reword، drop، edit و split.
- `--autosquash` + `--fixup` تاریخ را خودکار مرتب می‌کند.
- بعد از rebase فقط `push --force-with-lease`.
- در rebase، `--ours`/`--theirs` برعکس merge است.

**سؤالات مروری:**
1. چرا rebase هش commitها را عوض می‌کند؟
2. چرا `--force-with-lease` از `--force` امن‌تر است؟
3. چطور سه commit نامرتب را به یک commit تمیز تبدیل کنی؟

---

## فصل ۱۳: استراتژی‌های شاخه‌زنی — GitHub Flow، Git Flow، Trunk-based

### مقدمه

«چطور branch بزنیم» یک تصمیم فنی نیست، یک تصمیم **تیمی** است. سه مدل اصلی وجود دارد و هر کدام برای نوعی از تیم و محصول ساخته شده. انتخاب اشتباه، تیم را فلج می‌کند: یا سرعت را می‌کُشد یا کیفیت را.

**چرا مهم است؟** در مصاحبه‌ها می‌پرسند. در عمل، تیم‌هایی که مدل مشخصی ندارند، هرج‌ومرج دارند: یکی روی `develop` کار می‌کند، یکی روی `main`، یکی روی branch شخصی — و هیچ‌کس نمی‌داند کدام نسخه در production است.

**کاربرد عملی:** بعد از این فصل می‌توانی برای تیمت یک مدل انتخاب کنی و در `CONTRIBUTING.md` مکتوبش کنی.

### مفاهیم اساسی

#### مدل ۱: GitHub Flow (ساده‌ترین)

```
main ────●────●────●────●────●────●────→
          \  /      \  /      \  /
           ●●        ●●        ●●
        feature   hotfix    feature
```

قواعد:
1. `main` همیشه قابل انتشار است (deployable).
2. برای هر کار، یک branch از `main`.
3. commit، push، PR.
4. review + CI.
5. merge به `main`.
6. deploy فوری (یا خودکار).

**مزیت:** ساده، سریع، مناسب CI/CD.
**عیب:** برای محصولی که چند نسخه‌ی موازی دارد (مثل اپ موبایل با نسخه‌های ۲.۱ و ۲.۲ در دست کاربران) مناسب نیست.

#### مدل ۲: Git Flow (کلاسیک و سنگین)

```
main     ─────●─────────────────────●────────→  (releaseها)
              \                   /
release        ●─────●─────●──────●
              /       \
develop  ────●────●────●────●────●────●──────→
              \  /      \  /
feature        ●●        ●●
hotfix   ─────────────────────────●────→
```

branchها:
- `main` — فقط نسخه‌های منتشرشده (تگ می‌خورند)
- `develop` — یکپارچه‌سازی توسعه
- `feature/*` — از `develop`
- `release/*` — آماده‌سازی انتشار، از `develop` به `main`
- `hotfix/*` — از `main`، به `main` و `develop`

**مزیت:** کنترل کامل روی نسخه‌ها، مناسب محصولات با چرخه‌ی انتشار طولانی.
**عیب:** پیچیده، branchهای بلندعمر، conflict زیاد، با CI/CD مداوم سازگار نیست.

#### مدل ۳: Trunk-based Development (مدرن)

```
main ──●──●──●──●──●──●──●──●──●──●──→  (همه مستقیم روی trunk)
        \    /      \   /
         ●──●        ●──●
      (branch کوتاه: کمتر از یک روز)
```

قواعد:
1. یک branch اصلی (`main` یا `trunk`).
2. branchهای کوتاه‌عمر (ساعت‌ها، نه روزها).
3. commitهای کوچک و مکرر.
4. **Feature flag** برای ویژگی‌های ناتمام.
5. CI قوی و تست خودکار.

**مزیت:** سرعت بالا، conflict کم، مناسب تیم‌های بالغ.
**عیب:** نیاز به انضباط و ابزار (feature flag، CI سریع).

#### مدل ۴: Release branch (برای محصولات versioned)

```
main ────●────●────●────●────●──→
          \         \
release/1.0 ●───●───●  (پچ‌ها)
                    \
release/1.1          ●───●
```

مناسب: کتابخانه‌ها، اپ‌های موبایل، محصولات on-premise که مشتری چند نسخه دارد.

### مثال‌های عملی

#### مثال ۱: GitHub Flow در عمل

```bash
# ۱) از main، branch بساز
git switch main && git pull
git switch -c feat/user-profile

# ۲) کار کن، commit کن (چند بار)
git commit -am "feat(profile): افزودن فرم"
git commit -am "feat(profile): اتصال به API"

# ۳) push
git push -u origin feat/user-profile

# ۴) PR بساز
gh pr create --title "افزودن پروفایل کاربر" --body "Closes #42"

# ۵) review + CI
# ۶) merge (از UI یا gh)
gh pr merge --squash --delete-branch

# ۷) برگرد و به‌روز کن
git switch main && git pull
```

#### مثال ۲: Git Flow با gh

```bash
# شروع ویژگی
git switch develop && git pull
git switch -c feature/payment

# ... کار ...

# ادغام به develop (با PR)
gh pr create --base develop --title "feat: درگاه پرداخت"

# آماده‌سازی انتشار
git switch develop && git pull
git switch -c release/1.4.0
# فقط باگ‌فیکس، بدون ویژگی جدید
git commit -am "chore: به‌روزرسانی نسخه به 1.4.0"
gh pr create --base main --title "release: 1.4.0"

# بعد از merge به main: تگ بزن
git switch main && git pull
git tag -a v1.4.0 -m "نسخه 1.4.0"
git push origin v1.4.0

# برگرداندن به develop
git switch develop
git merge main
```

#### مثال ۳: Trunk-based با feature flag

```javascript
// flags.js
export const flags = {
  newCheckout: process.env.FF_NEW_CHECKOUT === "true",
};

// CheckoutPage.js
import { flags } from "./flags";

export default function CheckoutPage() {
  return flags.newCheckout ? <NewCheckout /> : <OldCheckout />;
}
```

حالا می‌توانی کد ناتمام را merge کنی، چون کاربر نمی‌بیندش. بعد از آماده شدن، flag را روشن می‌کنی.

#### مثال ۴: نوشتن استراتژی در `CONTRIBUTING.md`

```markdown
## استراتژی شاخه‌زنی

ما از **GitHub Flow** استفاده می‌کنیم:

1. `main` همیشه قابل انتشار است.
2. برای هر کار یک branch از `main` بساز:
   - `feat/<توضیح>` برای ویژگی
   - `fix/<توضیح>` برای باگ
   - `docs/<توضیح>` برای مستندات
   - `chore/<توضیح>` برای کارهای نگهداری
3. نام branch حداکثر ۵۰ کاراکتر، با خط تیره.
4. PR را در ۴۸ ساعت merge کن. اگر طولانی‌تر شد، آن را بشکن.
5. merge با **Squash** — یک commit تمیز به ازای هر PR.
6. branch بعد از merge خودکار حذف می‌شود.

### قواعد commit
- Conventional Commits الزامی است.
- هر commit باید تست‌ها را پاس کند.
```

### مقایسه و الگوهای مختلف

#### جدول تصمیم

| معیار | GitHub Flow | Git Flow | Trunk-based |
|---|---|---|---|
| اندازه تیم | ۲–۲۰ | ۱۰+ | ۵–۵۰۰ (Google، Meta) |
| تعداد نسخه‌ی موازی | ۱ | چند | ۱ (+flag) |
| سرعت انتشار | بالا | پایین | خیلی بالا |
| پیچیدگی | کم | زیاد | متوسط |
| نیاز به CI | متوسط | متوسط | **بالا** |
| نیاز به feature flag | کم | کم | **زیاد** |
| مناسب متن‌باز | ✅ | متوسط | ✅ |

#### کدام برای تو؟

```
آیا محصولت چند نسخه‌ی موازی دارد (اپ موبایل، کتابخانه، on-prem)؟
├── بله → Release branch (روی GitHub Flow)
└── خیر
    ├── تیم < ۱۰ نفر و CI متوسط؟ → GitHub Flow
    ├── تیم بزرگ و نیاز به کنترل شدید انتشار؟ → Git Flow
    └── تیم بالغ، CI قوی، می‌خواهی روزی چند بار deploy کنی؟ → Trunk-based
```

### اشکال شایع

#### ۱. branchهای بلندعمر `develop` که همیشه conflict دارند

نشانه‌ی Git Flow در تیمی که به آن نیاز ندارد. راه‌حل: به GitHub Flow مهاجرت کن.

#### ۲. «همه مستقیم روی main» بدون CI

این Trunk-based نیست، هرج‌ومرج است. Trunk-based یعنی commit مکرر **با** CI قوی و feature flag. بدون این‌ها، فقط سرعت خرابی بالا می‌رود.

#### ۳. feature flag که هرگز پاک نمی‌شود

```
✗ if (flags.newCheckout) ...  // ۲ سال بعد، flag همیشه true
```

flagها بدهی فنی‌اند. برای هر flag یک تاریخ انقضا بگذار و در Issue ثبت کن.

#### ۴. release branch بدون قاعده

اگر روی `release/1.4` هم feature جدید merge کنی، دیگر release نیست. فقط باگ‌فیکس.

#### ۵. نام branchهای بی‌معنی

```
✗ test، temp، ali، new، fix2، fix3-final
✓ feat/payment-retry، fix/login-500، docs/api-examples
```

### نکات مهم (Tips & Tricks)

- **مدل را مکتوب کن.** در `CONTRIBUTING.md`. تیم بدون سند، مدل ندارد.
- **مدل را با ابزار اجبار کن:** Rulesets (فصل ۳۹) می‌تواند merge به `main` را بدون review ممنوع کند.
- **branchهای کوتاه = زندگی راحت‌تر.** قاعده‌ی سرانگشتی: اگر بیش از ۴۰۰ خط تغییر داری، آن را بشکن.
- **Squash merge + GitHub Flow** ترکیب طلایی برای اکثر تیم‌هاست: تاریخ `main` تمیز، هر PR یک commit.
- **در Git Flow، `develop` را با `main` همگام نگه دار** بعد از هر release، وگرنه forever-diverged می‌شود.

### خلاصه فصل

- **GitHub Flow:** یک `main`، branch کوتاه، PR، merge، deploy. ساده و پیش‌فرض اکثر تیم‌ها.
- **Git Flow:** `main` + `develop` + feature/release/hotfix. قدرتمند ولی سنگین.
- **Trunk-based:** همه روی trunk، branchهای ساعتی، feature flag. سریع‌ترین، نیازمند انضباط.
- **Release branch** برای محصولات چندنسخه‌ای.
- انتخاب مدل به اندازه‌ی تیم، تعداد نسخه‌ها و بلوغ CI بستگی دارد.

**سؤالات مروری:**
1. چرا Git Flow برای تیم سه‌نفره با deploy روزانه مناسب نیست؟
2. Trunk-based بدون feature flag چه مشکلی دارد؟
3. کدام مدل برای یک کتابخانه‌ی npm با نسخه‌های ۱.x و ۲.x بهتر است؟

---

## فصل ۱۴: انتخاب روش merge در Pull Request

### مقدمه

GitHub سه دکمه‌ی merge دارد: **Merge commit**، **Squash and merge**، **Rebase and merge**. انتخاب اشتباه، تاریخ `main` را برای همیشه تحت تأثیر می‌گذارد — و برگشتش سخت است.

**چرا مهم است؟** این تصمیم یک‌بار برای هر مخزن گرفته می‌شود و روی همه‌ی PRها اثر می‌گذارد. تیمی که همه‌ی سه گزینه را باز بگذارد، تاریخ ناهمگون دارد: بعضی PRها یک commit، بعضی پنج commit، بعضی merge commit.

**کاربرد عملی:** بعد از این فصل می‌دانی برای پروژه‌ات کدام را انتخاب کنی و چطور در تنظیمات مخزن اجبارش کنی.

### مفاهیم اساسی

#### سه روش

**۱. Merge commit (`--no-ff`)**

```
main:  A ← B ← C ←─────── M
                ↖       ↗
                 D ← E
```

- همه‌ی commitهای branch حفظ می‌شوند.
- یک merge commit اضافه می‌شود.
- تاریخ دقیقاً واقعی است.

**۲. Squash and merge**

```
main:  A ← B ← C ← S
```

- همه‌ی commitهای branch به **یک** commit تبدیل می‌شوند.
- پیام commit قابل ویرایش است.
- commitهای «wip» حذف می‌شوند.

**۳. Rebase and merge**

```
main:  A ← B ← C ← D' ← E'
```

- commitها بازنویسی و یکی‌یکی روی `main` گذاشته می‌شوند.
- بدون merge commit.
- هش‌ها عوض می‌شوند.

#### کجا تنظیم می‌شود

**Settings → General → Pull Requests:**

- ☑ Allow merge commits
- ☑ Allow squash merging
- ☑ Allow rebase merging
- ☑ Automatically delete head branches

**توصیه:** فقط یکی را باز بگذار. تیم را مجبور کن یکسان رفتار کند.

### مثال‌های عملی

#### مثال ۱: دیدن تفاوت در عمل

```bash
# ساخت سناریو
git switch main
git switch -c feat/demo
echo "1" >> demo.txt && git add . && git commit -m "feat: بخش ۱"
echo "2" >> demo.txt && git commit -am "wip"
echo "3" >> demo.txt && git commit -am "fix typo"
git switch main
echo "main" >> other.txt && git add . && git commit -m "docs: main"

# روش ۱: merge commit
git merge --no-ff feat/demo -m "merge: feat/demo"
git log --oneline --graph -6

# (بازگردانی برای آزمایش روش بعدی)
git reset --hard HEAD~2

# روش ۲: squash
git merge --squash feat/demo
git commit -m "feat: افزودن demo (۳ تغییر)"
git log --oneline -3

# روش ۳: rebase
git reset --hard HEAD~1
git switch feat/demo
git rebase main
git switch main
git merge --ff-only feat/demo
git log --oneline --graph -6
```

#### مثال ۲: تنظیم از CLI

```bash
# فقط squash را مجاز کن
gh api -X PATCH repos/OWNER/REPO \
  -f allow_merge_commit=false \
  -f allow_squash_merge=true \
  -f allow_rebase_merge=false \
  -f delete_branch_on_merge=true

# بررسی
gh api repos/OWNER/REPO --jq '{merge:.allow_merge_commit,squash:.allow_squash_merge,rebase:.allow_rebase_merge}'
```

#### مثال ۳: merge از CLI با انتخاب روش

```bash
gh pr merge 42 --squash --delete-branch
gh pr merge 42 --merge
gh pr merge 42 --rebase
```

#### مثال ۴: پیام squash را خودکار تمیز کن

GitHub پیام پیش‌فرض squash را از عنوان PR + لیست commitها می‌سازد. برای تمیز کردن، در **Settings → General → Pull Requests → Squash merging**:

```
Default commit message: Pull request title and description
```

یا در PR، هنگام merge، پیام را دستی ویرایش کن.

### مقایسه و الگوهای مختلف

#### جدول کامل مقایسه

| معیار | Merge commit | Squash | Rebase |
|---|---|---|---|
| commitهای main | همه + merge | یکی | همه (بازنویسی‌شده) |
| تاریخ خطی | ❌ | ✅ | ✅ |
| رد ویژگی در تاریخ | ✅ (merge commit) | ✅ (یک commit) | ❌ (پراکنده) |
| `git bisect` | دقیق | درشت | دقیق |
| revert کل PR | آسان (revert merge) | آسان | سخت (چند commit) |
| commitهای «wip» | می‌مانند | حذف | می‌مانند |
| attribution چند نفر | ✅ | با Co-authored-by | ✅ |
| نیاز به force push | ❌ | ❌ | ❌ (GitHub خودش) |

#### کدام برای کدام پروژه

| پروژه | توصیه | چرا |
|---|---|---|
| اپ SaaS با deploy روزانه | **Squash** | تاریخ main تمیز، هر PR یک تغییر |
| کتابخانه/فریم‌ورک متن‌باز | **Merge commit** یا **Rebase** | حفظ attribution و تاریخ دقیق مشارکت‌کننده |
| مونوریپو با CI سنگین | **Squash** | یک commit = یک build، bisect معنادار |
| تیم با commitهای تمیز | **Rebase** | تاریخ خطی بدون از دست دادن granularity |
| پروژه با نیاز به revert مکرر | **Merge commit** | revert یک merge ساده است |

#### مشکل معروف revert merge

اگر با merge commit کار می‌کنی و بعد از merge یک PR را revert می‌کنی، **نمی‌توانی همان branch را دوباره merge کنی** — Git تغییرات را «قبلاً merge شده» می‌بیند.

```bash
# revert merge
git revert -m 1 <merge-hash>

# بعداً برای merge مجدد، باید revert را revert کنی
git revert <revert-hash>
git merge feature/again
```

با squash این مشکل وجود ندارد (چون یک commit معمولی است).

### اشکال شایع

#### ۱. هر سه گزینه باز گذاشته شده

نتیجه: تاریخ ناهمگون. یکی merge commit می‌زند، یکی squash، یکی rebase. `git log` غیرقابل خواندن.

#### ۲. Squash و از دست رفتن attribution

اگر PR را چند نفر نوشته‌اند و squash می‌کنی، فقط نویسنده‌ی PR ثبت می‌شود. راه‌حل: در پیام squash، `Co-authored-by:` اضافه کن.

#### ۳. Rebase merge و commitهای زیاد

اگر PR ۴۰ commit داشته باشد، rebase merge ۴۰ commit را روی `main` می‌گذارد. تاریخ خطی است ولی پر از commitهای ریز.

#### ۴. تغییر روش merge بعد از ۵۰۰ PR

ممکن است ولی تاریخ ناهمگون می‌شود. اگر مجبوری، از یک تاریخ مشخص تغییر بده و در `CONTRIBUTING.md` بنویس.

#### ۵. squash و از دست رفتن تاریخ branch

بعد از squash، commitهای اصلی branch در `main` نیستند. اگر بعداً بخواهی ببینی «چطور این ویژگی ساخته شد»، باید PR را ببینی، نه `git log`. این محدودیت واقعی squash است.

### نکات مهم (Tips & Tricks)

- **فقط یک روش را باز بگذار.** اجبار بهترین سیاست است.
- **`delete_branch_on_merge=true`** را روشن کن. branchهای مرده انبار می‌شوند.
- **عنوان PR را مثل پیام commit بنویس** اگر squash استفاده می‌کنی — چون همان می‌شود پیام commit.
- **برای PRهای بزرگ، merge commit؛ برای کوچک، squash.**
- **در GitHub، "Rebase and merge" هش‌ها را عوض می‌کند** — اگر بعداً کسی همان branch را داشته باشد، دوباره push نمی‌تواند بکند. برای branchهایی که فقط تو داری، مشکلی نیست.

### خلاصه فصل

- سه روش: **Merge commit** (تاریخ واقعی)، **Squash** (یک commit تمیز)، **Rebase** (خطی، بدون merge).
- **Squash** پیش‌فرض اکثر تیم‌های محصول است.
- **Merge commit** برای پروژه‌های متن‌باز و نیاز به revert آسان.
- **Rebase** برای تیم‌هایی که commitهای تمیز می‌نویسند.
- فقط یک روش را در تنظیمات مخزن باز بگذار.

**سؤالات مروری:**
1. چرا revert یک merge commit مشکل merge مجدد ایجاد می‌کند؟
2. کدام روش برای مونوریپو با CI سنگین بهتر است و چرا؟
3. چطور از دست رفتن attribution در squash را جبران کنی؟

---

**بعدی:** [`04-Remote-and-Collaboration.md`](04-Remote-and-Collaboration.md)

---

## منابع و مراجع

- Pro Git، فصل ۳ (شاخه‌زنی) — https://git-scm.com/book/fa/v2
- مستندات `git merge` — https://git-scm.com/docs/git-merge
- مستندات `git rebase` — https://git-scm.com/docs/git-rebase
- GitHub Flow — https://docs.github.com/en/get-started/using-github/github-flow
- A successful Git branching model (Vincent Driessen) — https://nvie.com/posts/a-successful-git-branching-model/
- Trunk Based Development — https://trunkbaseddevelopment.com
- مستندات merge methods در GitHub — https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges
