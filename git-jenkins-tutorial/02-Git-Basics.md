# بخش ۰۲: مبانی Git

**مدت یادگیری**: ۳ ساعت
**سطح سختی**: مبتدی
**پیش‌نیاز**: بخش ۰۱ (مقدمه و نقشه راه)

---

## فهرست محتوا

- [فصل ۱: نصب و پیکربندی Git](#فصل-۱-نصب-و-پیکربندی-git)
- [فصل ۲: ساختار داخلی Git](#فصل-۲-ساختار-داخلی-git)
- [فصل ۳: اولین مخزن (init و clone)](#فصل-۳-اولین-مخزن-init-و-clone)
- [فصل ۴: ثبت تغییرات](#فصل-۴-ثبت-تغییرات)
- [فصل ۵: مدیریت فایل‌ها و .gitignore](#فصل-۵-مدیریت-فایلها-و-gitignore)
- [فصل ۶: بررسی تاریخچه](#فصل-۶-بررسی-تاریخچه)

---

## فصل ۱: نصب و پیکربندی Git

### مقدمه

قبل از هر کاری باید Git را نصب و پیکربندی کنید. پیکربندی درست، از همان ابتدا از مشکلاتی مثل ثبت commit با نام اشتباه جلوگیری می‌کند.

### مفاهیم اساسی

**نصب بر اساس سیستم‌عامل:**

**Linux (Debian/Ubuntu):**
```bash
sudo apt update
sudo apt install git
```

**Linux (Fedora/RHEL):**
```bash
sudo dnf install git
```

**macOS:**
```bash
# با Homebrew
brew install git

# یا نصب Xcode Command Line Tools
xcode-select --install
```

**Windows:** از سایت [git-scm.com](https://git-scm.com) نصب‌کننده Git for Windows را دانلود کنید. این نسخه شامل **Git Bash** است که یک ترمینال شبیه Linux ارائه می‌دهد.

### مثال‌های عملی

پیکربندی اولیه (الزامی):

```bash
# نام و ایمیل (در همه commitها ثبت می‌شود)
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# ادیتور پیش‌فرض
git config --global core.editor "code --wait"

# نام شاخه پیش‌فرض
git config --global init.defaultBranch main

# رنگی کردن خروجی
git config --global color.ui auto

# نمایش تنظیمات
git config --list
```

سه سطح پیکربندی:

| سطح | محل | دستور |
|-----|-----|-------|
| System | کل سیستم | `git config --system` |
| Global | کاربر جاری | `git config --global` |
| Local | مخزن جاری | `git config --local` |

اولویت: Local > Global > System.

### اشکال شایع

- **فراموش کردن پیکربندی ایمیل**: commitها با ایمیل اشتباه ثبت می‌شوند و به حساب GitHub شما وصل نمی‌شوند.
- **استفاده از `--global` روی سیستم مشترک**: تنظیمات روی همه مخازن اعمال می‌شود.

### نکات مهم (Tips & Tricks)

- `git config --global alias.st status` میانبر می‌سازد؛ بعداً می‌توانید `git st` بزنید.
- با `git config --global pull.rebase false` رفتار pull را مشخص کنید.
- فایل تنظیمات در `~/.gitconfig` قرار دارد و قابل ویرایش دستی است.

### خلاصه فصل

Git را با توجه به سیستم‌عامل نصب و با نام، ایمیل و ادیتور پیکربندی کنید. سه سطح تنظیمات (System/Global/Local) داریم که Local بالاترین اولویت را دارد.

---

## فصل ۲: ساختار داخلی Git

### مقدمه

برای استفاده حرفه‌ای از Git، باید بدانید Git کد شما را کجا و چگونه نگه می‌دارد. این درک، رفتار دستورات را برایتان منطقی می‌کند.

### مفاهیم اساسی

Git سه ناحیه اصلی دارد:

```
Working Directory  →  Staging Area  →  Local Repository
    (فایل‌های من)      (آماده commit)      (تاریخچه ثبت‌شده)
         |                  |                    |
         |    git add       |    git commit      |
         |----------------->|------------------->|
```

1. **Working Directory**: فایل‌هایی که می‌بینید و ویرایش می‌کنید.
2. **Staging Area (Index)**: ناحیه میانی که تغییرات آماده commit در آن قرار می‌گیرند.
3. **Local Repository**: تاریخچه دائمی commitها (پوشه `.git`).

**پوشه `.git`** قلب مخزن است. اگر آن را حذف کنید، تاریخچه از بین می‌رود. شامل:
- `objects/`: محتوای واقعی فایل‌ها (به‌صورت فشرده)
- `refs/`: اشاره‌گرها به شاخه‌ها و tagها
- `HEAD`: اشاره‌گر به شاخه فعلی
- `config`: تنظیمات محلی

**چرخه حیات فایل در Git:**

```
Untracked → (add) → Staged → (commit) → Unmodified → (edit) → Modified → (add) → Staged ...
```

### مثال‌های عملی

دیدن وضعیت فایل‌ها:

```bash
git status
```

خروجی نمونه:
```
On branch main
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        hello.py

nothing added to commit but untracked files present
```

### مقایسه و الگوهای مختلف

| ناحیه | دستور انتقال به آن | محتوا |
|-------|---------------------|-------|
| Working Dir | ویرایش فایل | فایل‌های واقعی |
| Staging | `git add` | تغییرات آماده commit |
| Repository | `git commit` | commitهای ثبت‌شده |

### اشکال شایع

- **اشتباه رایج**: تصور اینکه `git add` تغییرات را ذخیره می‌کند. `add` فقط آن‌ها را به Staging می‌برد؛ ذخیره واقعی با `commit` است.
- **اشتباه رایج**: حذف پوشه `.git` به‌خاطر حجم آن. با این کار کل تاریخچه نابود می‌شود.

### نکات مهم (Tips & Tricks)

- Staging Area یکی از قدرتمندترین ویژگی‌های Git است؛ اجازه می‌دهد commitهای تمیز و منطقی بسازید.
- با `git add -p` می‌توانید تغییرات را به‌صورت تکه‌تکه (hunk) اضافه کنید.

### خلاصه فصل

Git سه ناحیه دارد: Working Directory، Staging Area و Local Repository. چرخه حیات فایل از Untracked شروع و با commit ثبت می‌شود. پوشه `.git` تمام تاریخچه را نگه می‌دارد.

---

## فصل ۳: اولین مخزن (init و clone)

### مقدمه

برای شروع کار با Git، دو راه دارید: ساخت مخزن جدید (`init`) یا کپی مخزن موجود (`clone`).

### مفاهیم اساسی

- **`git init`**: یک مخزن خالی در پوشه جاری می‌سازد (پوشه `.git`).
- **`git clone`**: یک مخزن کامل را از راه دور کپی می‌کند (با تمام تاریخچه).

### مثال‌های عملی

**ساخت مخزن جدید:**

```bash
mkdir my-project
cd my-project
git init
# خروجی: Initialized empty Git repository in /path/my-project/.git/
```

**کلون کردن مخزن موجود:**

```bash
git clone https://github.com/username/repo.git
# یا با نام پوشه دلخواه
git clone https://github.com/username/repo.git my-folder
```

**اولین commit:**

```bash
echo "# My Project" > README.md
git add README.md
git commit -m "docs: add initial README"
```

### مقایسه و الگوهای مختلف

| دستور | کاربرد | نتیجه |
|-------|--------|-------|
| `git init` | شروع پروژه جدید | مخزن خالی محلی |
| `git clone` | کپی پروژه موجود | مخزن کامل با تاریخچه |

### اشکال شایع

- **اشتباه رایج**: اجرای `git init` در پوشه اشتباه (مثلاً پوشه Home). همیشه اول با `pwd` مسیر را چک کنید.
- **اشتباه رایج**: `git init` در پوشه‌ای که از قبل مخزن است. Git پیام می‌دهد که مخزن مجدداً مقداردهی شد.

### نکات مهم (Tips & Tricks)

- پس از `git init`، بلافاصله یک فایل `.gitignore` بسازید.
- برای clone، اگر SSH راه‌اندازی کرده‌اید، از آدرس SSH استفاده کنید تا نیازی به وارد کردن رمز نباشد.
- `git clone --depth 1` فقط آخرین commit را می‌گیرد (برای مخازن بزرگ مفید است).

### خلاصه فصل

با `git init` مخزن جدید می‌سازید و با `git clone` مخزن موجود را کپی می‌کنید. اولین commit با `add` و `commit` انجام می‌شود.

---

## فصل ۴: ثبت تغییرات

### مقدمه

قلب کار با Git، ثبت تغییرات است. در این فصل دستورات اصلی `add`، `commit`، `status`، `diff` و `log` را عمیق یاد می‌گیریم.

### مفاهیم اساسی

**`git add`**: افزودن تغییرات به Staging.
```bash
git add file.txt          # یک فایل
git add .                 # همه تغییرات
git add *.py              # همه فایل‌های پایتون
git add -A                # همه تغییرات شامل حذف‌ها
```

**`git commit`**: ثبت تغییرات Staging در تاریخچه.
```bash
git commit -m "پیام"
git commit                # باز کردن ادیتور برای پیام طولانی
git commit -am "پیام"     # add + commit برای فایل‌های tracked
```

**`git status`**: نمایش وضعیت فایل‌ها.

**`git diff`**: نمایش تفاوت‌ها.
```bash
git diff                  # تغییرات Working Dir (unstaged)
git diff --staged         # تغییرات Staged
git diff HEAD             # همه تغییرات نسبت به آخرین commit
```

**`git log`**: نمایش تاریخچه.

### مثال‌های عملی

یک جریان کاری کامل:

```bash
# ۱. بررسی وضعیت
git status

# ۲. دیدن تغییرات
git diff

# ۳. افزودن به Staging
git add script.py

# ۴. بررسی مجدد
git status
git diff --staged

# ۵. ثبت
git commit -m "feat: add data processing script"

# ۶. مشاهده تاریخچه
git log --oneline
```

### مقایسه و الگوهای مختلف

| دستور | محدوده |
|-------|--------|
| `git diff` | Working Dir vs Staging |
| `git diff --staged` | Staging vs آخرین commit |
| `git diff HEAD` | Working Dir vs آخرین commit |

### اشکال شایع

- **اشتباه رایج**: `git commit -am` فایل‌های جدید (untracked) را شامل نمی‌شود؛ فقط فایل‌های tracked را commit می‌کند.
- **اشتباه رایج**: پیام‌های بی‌معنی مثل `"fix"` یا `"update"`. پیام باید بگوید «چه» و «چرا».
- **اشتباه رایج**: commit کردن فایل‌های حساس (`.env`, رمزها) بدون `.gitignore`.

### نکات مهم (Tips & Tricks)

- قبل از هر commit، `git diff --staged` بزنید تا مطمئن شوید چه چیزی ثبت می‌شود.
- commitها را کوچک و منطقی نگه دارید؛ هر commit یک تغییر قابل توضیح.
- از زمان حال و امری در پیام استفاده کنید: `"add feature"` نه `"added feature"`.

### خلاصه فصل

`add` تغییرات را به Staging می‌برد، `commit` آن‌ها را ثبت می‌کند. `status` و `diff` ابزارهای بررسی‌اند. commitهای کوچک و پیام‌های معنادار، تاریخچه‌ای خوانا می‌سازند.

---

## فصل ۵: مدیریت فایل‌ها و .gitignore

### مقدمه

همه فایل‌ها نباید در Git باشند. فایل‌های موقت، وابستگی‌ها، رمزها و خروجی‌های Build باید نادیده گرفته شوند. ابزار این کار `.gitignore` است.

### مفاهیم اساسی

**حذف و جابه‌جایی فایل:**
```bash
git rm file.txt           # حذف فایل و افزودن به Staging
git rm --cached file.txt  # فقط از Git حذف کن، فایل بماند
git mv old.txt new.txt    # جابه‌جایی/تغییر نام
```

**بازیابی فایل:**
```bash
git restore file.txt              # بازگرداندن تغییرات Working Dir
git restore --staged file.txt     # خروج از Staging
```

**الگوهای `.gitignore`:**
```gitignore
# وابستگی‌ها
node_modules/
venv/
__pycache__/

# فایل‌های Build
dist/
build/
*.class
*.jar

# فایل‌های حساس
.env
*.key
secrets.yaml

# فایل‌های سیستم و ادیتور
.DS_Store
Thumbs.db
.idea/
.vscode/

# لاگ‌ها
*.log
```

### مثال‌های عملی

ساخت `.gitignore` برای پروژه پایتون:

```bash
cat > .gitignore << 'EOF'
__pycache__/
*.py[cod]
venv/
.env
.pytest_cache/
*.egg-info/
dist/
build/
EOF

git add .gitignore
git commit -m "chore: add gitignore"
```

اگر فایلی را قبلاً commit کرده‌اید و حالا می‌خواهید نادیده بگیرید:

```bash
git rm --cached .env
echo ".env" >> .gitignore
git commit -m "chore: stop tracking .env"
```

### مقایسه و الگوهای مختلف

| الگو | معنی |
|------|------|
| `*.log` | همه فایل‌های با پسوند log |
| `build/` | پوشه build |
| `/config` | فقط config در ریشه |
| `!important.log` | استثنا (نادیده نگیر) |
| `**/temp` | temp در هر سطح |

### اشکال شایع

- **اشتباه رایج**: commit کردن `.env` و سپس حذف آن. حذف از commit جدید کافی نیست؛ تاریخچه هنوز آن را دارد. باید رمزها را تغییر دهید.
- **اشتباه رایج**: انتظار اعمال `.gitignore` روی فایل‌های از قبل tracked. `.gitignore` فقط روی untrackedها اثر دارد.
- **اشتباه رایج**: استفاده از `.gitignore` سراسری اشتباه.

### نکات مهم (Tips & Tricks)

- برای الگوهای پرتکرار، یک `.gitignore` سراسری بسازید:
  ```bash
  git config --global core.excludesfile ~/.gitignore_global
  ```
- از سایت [gitignore.io](https://www.toptal.com/developers/gitignore) برای ساخت `.gitignore` بر اساس زبان و ادیتور استفاده کنید.
- اگر فایلی اشتباهی tracked شده، `git rm --cached` راه‌حل است.

### خلاصه فصل

`.gitignore` تعیین می‌کند چه فایل‌هایی نادیده گرفته شوند. فایل‌های وابستگی، Build، حساس و سیستمی باید نادیده گرفته شوند. برای فایل‌های از قبل tracked از `git rm --cached` استفاده کنید.

---

## فصل ۶: بررسی تاریخچه

### مقدمه

تاریخچه Git یک گنجینه است: می‌توانید ببینید چه کسی، چه زمانی و چرا تغییری داده. در این فصل ابزارهای کاوش تاریخچه را می‌آموزیم.

### مفاهیم اساسی

**`git log` پیشرفته:**
```bash
git log                      # تاریخچه کامل
git log --oneline            # خلاصه یک‌خطی
git log --graph --oneline    # نمای گرافیکی شاخه‌ها
git log -5                   # ۵ commit آخر
git log --author="Ali"       # فیلتر بر اساس نویسنده
git log --since="2 weeks ago" # فیلتر زمانی
git log -p                   # نمایش تغییرات هر commit
git log --stat               # خلاصه آماری تغییرات
```

**`git show`**: نمایش جزئیات یک commit.
```bash
git show HEAD              # آخرین commit
git show abc1234           # یک commit خاص
git show HEAD~2            # دو commit قبل‌تر
```

**`git blame`**: نمایش نویسنده هر خط.
```bash
git blame file.py
```

### مثال‌های عملی

یافتن تغییرات یک فایل خاص:

```bash
git log --oneline -- path/to/file.py
git log -p -- path/to/file.py
```

جست‌وجو در پیام‌های commit:

```bash
git log --grep="fix bug"
git log -S "function_name"   # commitهایی که این رشته را اضافه/حذف کرده‌اند
```

### مقایسه و الگوهای مختلف

| دستور | کاربرد |
|-------|--------|
| `git log` | تاریخچه کامل |
| `git log --oneline` | خلاصه |
| `git log -p` | با تغییرات |
| `git show <hash>` | یک commit خاص |
| `git blame` | نویسنده هر خط |

### اشکال شایع

- **اشتباه رایج**: نادیده گرفتن `git log` هنگام عیب‌یابی. تاریخچه معمولاً پاسخ را می‌دهد.
- **اشتباه رایج**: گیج شدن با هش‌های طولانی. از `--oneline` استفاده کنید.

### نکات مهم (Tips & Tricks)

- `git log --oneline --graph --all` نمای کاملی از همه شاخه‌ها می‌دهد.
- `git shortlog -sn` تعداد commitهای هر نویسنده را نشان می‌دهد.
- با `alias` میان‌بر بسازید: `git config --global alias.lg "log --oneline --graph --all"`.

### خلاصه فصل

`git log`، `git show` و `git blame` ابزارهای کاوش تاریخچه‌اند. با فیلترها (نویسنده، زمان، محتوا) می‌توانید دقیقاً چیزی که می‌خواهید پیدا کنید.

---

## منابع و مراجع

- مستندات `git log`: [https://git-scm.com/docs/git-log](https://git-scm.com/docs/git-log)
- راهنمای `.gitignore`: [https://git-scm.com/docs/gitignore](https://git-scm.com/docs/gitignore)
- تولید `.gitignore`: [https://www.toptal.com/developers/gitignore](https://www.toptal.com/developers/gitignore)
- Git Reference: [https://git-scm.com/docs](https://git-scm.com/docs)
- GitHub Git Handbook: [https://guides.github.com/introduction/git-handbook/](https://guides.github.com/introduction/git-handbook/)
