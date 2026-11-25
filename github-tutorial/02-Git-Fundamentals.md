# بخش ۰۲: مبانی Git — نصب، commit، ignore و تاریخ

**مدت یادگیری**: ۵ تا ۶ ساعت
**سطح سختی**: مبتدی
**پیش‌نیاز**: بخش ۰۱ (مدل ذهنی snapshot و اشاره‌گر)

---

## فهرست محتوا

- [فصل ۵: نصب و پیکربندی Git در ویندوز، مک و لینوکس](#فصل-۵)
- [فصل ۶: اولین مخزن — سه ناحیه و چرخه‌ی روزمره](#فصل-۶)
- [فصل ۷: commit حرفه‌ای — پیام، اتمیک بودن و diff](#فصل-۷)
- [فصل ۸: `.gitignore` و مدیریت فایل‌های ناخواسته](#فصل-۸)
- [فصل ۹: کاوش در تاریخ — log، show، blame و جست‌وجو](#فصل-۹)

---

## فصل ۵: نصب و پیکربندی Git در ویندوز، مک و لینوکس

### مقدمه

نصب Git ساده است، ولی پیکربندی درست آن چیزی است که تفاوت بین «کار می‌کند» و «درست کار می‌کند» را می‌سازد. اکثر مشکلات عجیبی که بعداً می‌بینی — خط‌های CRLF در ویندوز، نام فایل فارسی که به `\303\251` تبدیل شده، commitهایی که به پروفایلت وصل نمی‌شوند — ریشه در تنظیماتی دارد که در پنج دقیقه‌ی اول انجام ندادی.

**چرا مهم است؟** Git روی سه سیستم‌عامل رفتار کمی متفاوت دارد. اگر تیم مختلط داری (ویندوز + مک + لینوکس)، بدون تنظیم درست `core.autocrlf` و `core.quotepath`، هر PR پر از diffهای الکی می‌شود.

**کاربرد عملی:** این فصل را یک بار درست انجام بده؛ بعد از آن هیچ‌وقت لازم نیست برگردی.

### مفاهیم اساسی

#### Git کجا نصب می‌شود

| سیستم | مسیر معمول | روش نصب |
|---|---|---|
| لینوکس (Debian/Ubuntu) | `/usr/bin/git` | `apt install git` |
| لینوکس (Fedora/RHEL) | `/usr/bin/git` | `dnf install git` |
| مک | `/usr/bin/git` (Xcode CLT) یا `/opt/homebrew/bin/git` | `brew install git` |
| ویندوز | `C:\Program Files\Git\cmd\git.exe` | Git for Windows |

**نکته‌ی مک:** `/usr/bin/git` نسخه‌ی اپل است که معمولاً قدیمی است. اگر با Homebrew نصب کنی، نسخه‌ی جدیدتر می‌گیری. چک کن:

```bash
which -a git
# /opt/homebrew/bin/git
# /usr/bin/git
git --version
```

#### سه سطح پیکربندی

Git تنظیمات را در سه سطح می‌خواند، به ترتیب اولویت:

```
1. --system   → /etc/gitconfig           (کل سیستم)
2. --global   → ~/.gitconfig             (کاربر جاری)  ← بیشتر کارها اینجا
3. --local    → .git/config              (همین مخزن)   ← اولویت بالاتر
```

هر سطح بالاتر، سطح پایین‌تر را override می‌کند. برای دیدن همه‌چیز با مبدأ:

```bash
git config --list --show-origin
```

#### تنظیمات حیاتی

| کلید | مقدار پیشنهادی | چرا |
|---|---|---|
| `user.name` | نام واقعی | در commit ثبت می‌شود |
| `user.email` | ایمیل تأییدشده‌ی GitHub | commit به پروفایل وصل می‌شود |
| `init.defaultBranch` | `main` | جلوگیری از هشدار «master» |
| `core.autocrlf` | ویندوز: `true`، مک/لینوکس: `input` | مدیریت پایان خط |
| `core.quotepath` | `false` | نمایش درست نام فایل فارسی |
| `pull.rebase` | `false` (یا `true` اگر rebase را دوست داری) | جلوگیری از merge commitهای ناخواسته |
| `push.default` | `simple` | رفتار قابل پیش‌بینی |
| `fetch.prune` | `true` | پاک کردن branchهای حذف‌شده‌ی remote |
| `rebase.autoStash` | `true` | ذخیره‌ی خودکار تغییرات نیمه‌کاره |

### مثال‌های عملی

#### مثال ۱: نصب و تنظیم اولیه (لینوکس/WSL)

```bash
# نصب
sudo apt update && sudo apt install -y git

# تنظیمات پایه
git config --global user.name "علی رضایی"
git config --global user.email "ali@example.com"
git config --global init.defaultBranch main

# پایان خط: در لینوکس فایل‌ها LF هستند
git config --global core.autocrlf input

# نام فایل فارسی
git config --global core.quotepath false

# رفتار pull و push
git config --global pull.rebase false
git config --global push.default simple
git config --global fetch.prune true

# ادیتور پیش‌فرض
git config --global core.editor "code --wait"   # یا nano، vim

# بررسی
git config --list --show-origin | head -20
```

#### مثال ۲: ویندوز

```bash
# بعد از نصب Git for Windows (که Git Bash را هم می‌آورد)
git config --global user.name "Ali Rezaei"
git config --global user.email "ali@example.com"
git config --global init.defaultBranch main

# در ویندوز: تبدیل LF → CRLF در checkout و برعکس در commit
git config --global core.autocrlf true

git config --global core.quotepath false
git config --global core.editor "code --wait"
```

**تله‌ی مهم ویندوز:** اگر `autocrlf` را روی `false` بگذاری و تیم لینوکسی داشته باشی، هر فایل متنی در diff «کاملاً تغییر کرده» نشان داده می‌شود. اگر `true` بگذاری و مخزن از قبل LF دارد، همه‌چیز درست است. راه حرفه‌ای‌تر: `.gitattributes` (فصل ۲۵).

#### مثال ۳: aliasهای ضروری

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.lg "log --graph --oneline --decorate --all"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.unstage "reset HEAD --"
git config --global alias.undo "reset --soft HEAD~1"
```

حالا `git lg` یک نمای درختی رنگی می‌دهد و `git undo` آخرین commit را برمی‌گرداند ولی تغییرات را نگه می‌دارد.

#### مثال ۴: فایل `~/.gitconfig` کامل

```ini
[user]
    name = علی رضایی
    email = ali@example.com
    signingkey = ~/.ssh/id_ed25519.pub

[init]
    defaultBranch = main

[core]
    autocrlf = input
    quotepath = false
    editor = code --wait
    pager = less -FRX

[pull]
    rebase = false

[push]
    default = simple
    autoSetupRemote = true

[fetch]
    prune = true

[rebase]
    autoStash = true

[merge]
    conflictstyle = diff3

[color]
    ui = auto

[alias]
    st = status
    lg = log --graph --oneline --decorate --all
    undo = reset --soft HEAD~1
    unstage = reset HEAD --
    last = log -1 HEAD --stat

[commit]
    gpgsign = false

[gpg]
    format = ssh
```

**`merge.conflictstyle = diff3`** یکی از بهترین تنظیمات است: موقع conflict، نسخه‌ی مشترک (base) را هم نشان می‌دهد، نه فقط «مال من» و «مال او». حل conflict ده برابر آسان‌تر می‌شود.

**`push.autoSetupRemote = true`** (Git 2.37+): بار اول `git push` بدون `-u` کار می‌کند.

### مقایسه و الگوهای مختلف

#### `autocrlf`: سه گزینه

| مقدار | چه می‌کند | مناسب برای |
|---|---|---|
| `true` | در checkout LF→CRLF، در commit CRLF→LF | ویندوز با تیم مختلط |
| `input` | در checkout دست نمی‌زند، در commit CRLF→LF | مک/لینوکس با تیم مختلط |
| `false` | هیچ تبدیلی نمی‌کند | وقتی همه LF هستند یا `.gitattributes` داری |

**بهترین راه در ۲۰۲۶:** `core.autocrlf=false` + `.gitattributes` با `* text=auto eol=lf`. این صریح، قابل حمل و مستقل از تنظیمات هر دستگاه است.

#### ادیتور پیش‌فرض

| ادیتور | دستور | نکته |
|---|---|---|
| VS Code | `code --wait` | `--wait` حیاتی است، وگرنه Git منتظر نمی‌ماند |
| nano | `nano` | ساده‌ترین برای تازه‌کار |
| vim | `vim` | همه‌جا هست |
| Notepad++ (ویندوز) | `notepad++ -multiInst -nosession` | |

### اشکال شایع

#### ۱. `Please tell me who you are`

```
*** Please tell me who you are.
Run
  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"
```

یعنی `user.name`/`user.email` تنظیم نشده. دقیقاً همان دو دستور را بزن.

#### ۲. commitها به پروفایلم وصل نمی‌شوند

اگر ایمیل در `user.email` با ایمیل تأییدشده‌ی حساب GitHub یکی نباشد، commitها «unverified» و بدون آواتار نشان داده می‌شوند. چک کن:

```bash
git config user.email
# و در GitHub: Settings → Emails
```

اگر ایمیلت خصوصی است، GitHub یک ایمیل `ID+username@users.noreply.github.com` می‌دهد؛ همان را بگذار.

#### ۳. نام فایل فارسی به `\303\251` تبدیل شده

```bash
git config --global core.quotepath false
```

#### ۴. هر فایل «کاملاً تغییر کرده» نشان داده می‌شود

مشکل پایان خط. راه‌حل:

```bash
# ببین چه خبر است
git diff --stat
file somefile.txt   # باید بگوید "ASCII text" نه "with CRLF line terminators"

# راه‌حل: .gitattributes
echo "* text=auto eol=lf" > .gitattributes
git add --renormalize .
git commit -m "chore: normalize line endings"
```

#### ۵. `code: command not found`

در VS Code: `Ctrl+Shift+P` → «Shell Command: Install 'code' command in PATH».

#### ۶. `git` در PowerShell کار می‌کند ولی در CMD نه

مسیر Git در `PATH` نیست. Git for Windows معمولاً اضافه می‌کند؛ اگر نه، دستی `C:\Program Files\Git\cmd` را به PATH اضافه کن.

### نکات مهم (Tips & Tricks)

- **`git config --global --edit`** فایل config را در ادیتور باز می‌کند — برای ویرایش دستی راحت‌تر از ده دستور.
- **`git config --get-regexp alias`** همه‌ی aliasها را نشان می‌دهد.
- **alias برای چیزهای طولانی:** `git config --global alias.cleanup "!git branch --merged | grep -v '\\*\\|main\\|develop' | xargs -r git branch -d"` — branchهای merge‌شده را پاک می‌کند.
- **`git help <command>`** مستندات کامل را در مرورگر باز می‌کند. `git <command> -h` خلاصه را در ترمینال.
- **در شرکت، `--local` را برای ایمیل کاری استفاده کن:**
  ```bash
  cd ~/work/project
  git config --local user.email "ali@company.com"
  ```
  این‌طوری commitهای کاری و شخصی قاطی نمی‌شوند.

### خلاصه فصل

- Git سه سطح config دارد: **system → global → local** (اولویت برعکس).
- حداقل تنظیمات: `user.name`، `user.email`، `init.defaultBranch`.
- **`core.autocrlf`** را درست تنظیم کن یا از `.gitattributes` استفاده کن.
- **`core.quotepath=false`** برای نام فایل فارسی.
- **`merge.conflictstyle=diff3`** حل conflict را بسیار آسان‌تر می‌کند.
- aliasها سرعت کار روزمره را چند برابر می‌کنند.

**سؤالات مروری:**
1. تفاوت `--global` و `--local` چیست و کدام اولویت دارد؟
2. چرا در ویندوز `core.autocrlf=true` و در لینوکس `input`؟
3. اگر commitهایت به پروفایل GitHub وصل نمی‌شوند، اول چه چیزی را چک می‌کنی؟

---

## فصل ۶: اولین مخزن — سه ناحیه و چرخه‌ی روزمره

### مقدمه

در فصل ۱ یک بار چرخه‌ی پایه را دیدی. این فصل همان را عمیق‌تر می‌کند و به سؤال‌هایی جواب می‌دهد که معمولاً بی‌جواب می‌مانند: `git init` دقیقاً چه می‌سازد؟ `git add .` چه فرقی با `git add -A` دارد؟ چرا `git status` گاهی می‌گوید «Changes not staged for commit» و گاهی «Changes to be committed»؟

**چرا مهم است؟** تمام دستورهای بعدی (reset، restore، stash، commit --amend) روی همین سه ناحیه کار می‌کنند. اگر مرزهایشان را نشناسی، هر کدام یک جعبه‌ی سیاه می‌شود.

**کاربرد عملی:** بعد از این فصل می‌توانی هر تغییری را دقیقاً به جایی که می‌خواهی ببری — بدون حدس.

### مفاهیم اساسی

#### `git init` چه می‌سازد

```bash
git init
```

پوشه‌ی `.git/` با این ساختار:

```
.git/
├── HEAD              → اشاره‌گر به branch جاری
├── config            → تنظیمات local
├── description       → (برای gitweb، بی‌اهمیت)
├── hooks/            → نمونه‌ی hookها (فصل ۸۷)
├── info/
│   └── exclude       → مثل .gitignore ولی فقط محلی
├── objects/          → همه‌ی blob/tree/commit/tag
│   ├── 3f/
│   │   └── 2a1b9c...  (فشرده‌شده با zlib)
│   ├── pack/         → اشیاء بسته‌بندی‌شده
│   └── info/
├── refs/
│   ├── heads/        → branchها (main، feature، ...)
│   └── tags/         → تگ‌ها
└── index             → Staging Area
```

**`index` همان Staging Area است** — یک فایل باینری که می‌گوید «در commit بعدی چه چیزی باشد».

#### `git clone` چه می‌کند

```bash
git clone git@github.com:user/repo.git
```

1. پوشه‌ای به نام `repo` می‌سازد (یا نامی که بدهی).
2. `git init` می‌زند.
3. remote به نام `origin` اضافه می‌کند.
4. همه‌ی commitها را از remote می‌گیرد (`fetch`).
5. branch پیش‌فرض را checkout می‌کند (`main` معمولاً).
6. `HEAD` را روی آن تنظیم می‌کند.

#### تفاوت‌های `git add`

| دستور | چه چیزی را stage می‌کند |
|---|---|
| `git add file.txt` | فقط همان فایل |
| `git add .` | همه‌ی تغییرات در پوشه‌ی جاری و زیرپوشه‌ها (فایل‌های حذف‌شده را در Git 2.x **نه**) |
| `git add -A` | همه‌ی تغییرات در کل مخزن، شامل حذف‌ها |
| `git add -u` | فقط فایل‌های ردیابی‌شده (modify + delete)، بدون فایل جدید |
| `git add -p` | تکه‌تکه، با تأیید تو |
| `git add -i` | حالت تعاملی کامل |
| `git add -f` | حتی اگر در `.gitignore` باشد |

**تله:** از Git 2.0، `git add .` هم حذف‌ها را شامل می‌شود (قبلاً نه). ولی برای اطمینان، در مخزن‌های بزرگ `git add -A` صریح‌تر است.

#### وضعیت‌های فایل

```
Untracked  →  git add  →  Staged (new file)
Modified   →  git add  →  Staged (modified)
Staged     →  git commit →  Unmodified (clean)
Unmodified →  ویرایش   →  Modified
Staged     →  git restore --staged → Modified
```

#### `git status` را خواندن

```bash
$ git status
On branch feature/login

Changes to be committed:              ← در staging، آماده‌ی commit
  (use "git restore --staged <file>..." to unstage)
        new file:   auth.js
        modified:   package.json

Changes not staged for commit:        ← تغییر کرده ولی stage نشده
  (use "git add <file>..." to update what will be committed)
        modified:   README.md

Untracked files:                      ← Git نمی‌شناسدش
  (use "git add <file>..." to include in what will be committed)
        notes.txt
```

### مثال‌های عملی

#### مثال ۱: ساخت مخزن و چرخه‌ی کامل

```bash
mkdir ~/github-lab/shop && cd ~/github-lab/shop
git init

# ساختار اولیه
mkdir -p src tests
cat > README.md <<'EOF'
# فروشگاه

یک API ساده برای مدیریت سفارش.
EOF

cat > src/app.py <<'EOF'
def health():
    return {"status": "ok"}
EOF

git status --short
# ?? README.md
# ?? src/

git add README.md src/app.py
git status --short
# A  README.md
# A  src/app.py

git commit -m "feat: ساختار اولیه‌ی پروژه"
git status
# nothing to commit, working tree clean
```

**`--short`** یک نمای فشرده می‌دهد که برای کار روزمره عالی است:

| کد | معنی |
|---|---|
| `??` | untracked |
| `A ` | added (در staging) |
| `M ` | modified در staging |
| ` M` | modified در working dir |
| `MM` | هم staging هم working تغییر کرده |
| `D ` | deleted در staging |
| `R ` | renamed |

#### مثال ۲: `add -p` — کامیت تمیز از تغییرات قاطی

فرض کن در `app.py` سه تغییر دادی: یک باگ fix، یک تابع جدید، و یک `print` دیباگ که یادت رفته پاک کنی.

```bash
git add -p src/app.py
# Git تکه‌تکه نشان می‌دهد و می‌پرسد:
# Stage this hunk [y,n,q,a,d,s,e,?]?

# y = بله، این تکه را stage کن
# n = نه
# s = این hunk را کوچک‌تر کن
# e = دستی ویرایش کن
# q = خروج
```

با `y`/`n` می‌توانی فقط باگ‌فیکس و تابع جدید را stage کنی و `print` را نه. سپس:

```bash
git commit -m "fix(api): رفع خطای اعتبارسنجی و افزودن تابع سلامت"
# print هنوز در working directory است، stage نشده
```

#### مثال ۳: حذف فایل از staging بدون از دست دادن تغییرات

```bash
git add .
# اوه، فایل اشتباهی هم اضافه شد

git restore --staged secrets.txt     # Git 2.23+
# یا: git reset HEAD secrets.txt     (قدیمی)

git status --short
# ?? secrets.txt    ← برگشت به untracked، محتوا دست‌نخورده
```

#### مثال ۴: حذف فایل از مخزن ولی نگه داشتن روی دیسک

```bash
# فایلی را قبلاً commit کرده‌ای، حالا می‌خواهی Git دیگر ردیابی‌اش نکند
git rm --cached .env
echo ".env" >> .gitignore
git commit -m "chore: توقف ردیابی فایل env"

ls -la .env    # فایل هنوز هست
```

**توجه:** این فایل از commit بعدی حذف می‌شود، ولی **در تاریخچه می‌ماند**. اگر محتوا حساس بود، فصل ۹۳ را بخوان.

#### مثال ۵: تغییر نام فایل

```bash
git mv old-name.py new-name.py
# معادل: mv old new && git add new && git rm old
```

Git خودش تشخیص می‌دهد rename است (اگر محتوا شبیه باشد) و در `git status` با `R` نشان می‌دهد.

#### مثال ۶: commit فقط بخشی از فایل با `git commit -p`

```bash
git commit -p -m "refactor: جدا کردن تابع اعتبارسنجی"
# مثل add -p ولی مستقیم commit می‌کند
```

### مقایسه و الگوهای مختلف

#### `init` + `remote add` در برابر `clone`

| | `git init` | `git clone` |
|---|---|---|
| برای | پروژه‌ی جدید از صفر | پروژه‌ی موجود |
| تاریخ | خالی | کامل |
| remote | باید دستی اضافه کنی | خودکار `origin` |
| branch | `main` محلی | `main` ردیاب `origin/main` |

روش رایج برای «پروژه‌ی محلی دارم، می‌خواهم به GitHub ببرم»:

```bash
cd my-project
git init
git add .
git commit -m "chore: نسخه‌ی اولیه"
gh repo create my-project --public --source=. --push
```

#### `git add .` در برابر `git add -A`

| | `add .` | `add -A` |
|---|---|---|
| دامنه | پوشه‌ی جاری | کل مخزن |
| حذف‌ها | بله (Git 2.x) | بله |
| فایل جدید | بله | بله |
| کِی خطرناک | وقتی در زیرپوشه هستی و فکر می‌کنی همه‌چیز را add کردی | وقتی فایل بزرگی هست که نباید |

### اشکال شایع

#### ۱. `fatal: not a git repository`

یعنی در پوشه‌ای هستی که `.git` ندارد (یا در زیرپوشه‌ی اشتباه). `pwd` بزن و چک کن.

#### ۲. فایل جدید در commit نیست

`git commit -am` فایل‌های untracked را شامل نمی‌شود. اول `git add` لازم است.

#### ۳. مخزن تودرتو (nested repo)

اگر داخل پروژه‌ات یک پوشه باشد که خودش `.git` دارد (مثلاً clone کرده‌ای)، Git آن را نادیده می‌گیرد یا به‌عنوان submodule می‌بیند. راه‌حل: پوشه‌ی `.git` داخلی را پاک کن (اگر تاریخش را نمی‌خواهی) یا submodule درست بساز (فصل ۸۵).

#### ۴. `git add .` و سپس «فایل‌های عجیب» در staging

معمولاً `node_modules/`، `__pycache__/`، `.venv/`، فایل‌های `.DS_Store`. راه‌حل: `.gitignore` (فصل ۸) و بعد:

```bash
git rm -r --cached node_modules
git commit -m "chore: حذف node_modules از مخزن"
```

#### ۵. commit خالی (empty commit)

```bash
git commit --allow-empty -m "ci: اجرای مجدد workflow"
```

گاهی لازم است (trigger کردن CI، علامت‌گذاری نقطه‌ی شروع). ولی اگر بدون `--allow-empty` خطای «nothing to commit» گرفتی، یعنی واقعاً چیزی stage نکرده‌ای.

### نکات مهم (Tips & Tricks)

- **`git status -sb`** نسخه‌ی کوتاه با نام branch و وضعیت upstream. سریع‌ترین نگاه روزمره.
- **`git add -N <file>`** (intent-to-add) فایل را بدون محتوا ثبت می‌کند تا در `git diff` دیده شود. برای دیدن diff فایل جدید قبل از add.
- **`git commit --dry-run`** نشان می‌دهد چه چیزی commit می‌شود، بدون انجام.
- **`git diff --cached`** = `git diff --staged`. هر دو یک چیز.
- **پوشه‌ی `.git` را در بکاپ‌های ابزاری کپی نکن.** اگر ابزاری مثل OneDrive یا Dropbox پوشه‌ی `.git` را sync کند، احتمال خرابی index زیاد است.

### خلاصه فصل

- `git init` پوشه‌ی `.git` را با objects، refs و **index** می‌سازد.
- سه ناحیه: **Working Directory → Staging (index) → Repository**.
- `git add -p` برای commitهای تمیز ضروری است.
- `git restore --staged` فایل را از staging بیرون می‌آورد بدون از دست دادن تغییرات.
- `git rm --cached` فایل را از ردیابی خارج می‌کند ولی روی دیسک نگه می‌دارد.

**سؤالات مروری:**
1. تفاوت `git add .` و `git add -u` چیست؟
2. چطور یک فایل را از staging خارج کنی بدون اینکه تغییراتش برود؟
3. چرا `git commit -am` فایل جدید را شامل نمی‌شود؟

---

## فصل ۷: commit حرفه‌ای — پیام، اتمیک بودن و diff

### مقدمه

commit واحد بنیادی تاریخ توست. کیفیت تاریخ = کیفیت commitها. یک مخزن با ۵۰۰۰ commit بی‌معنی، عملاً تاریخ ندارد؛ یک مخزن با ۵۰۰ commit خوب، یک مستند زنده است که می‌گوید چرا هر تصمیمی گرفته شد.

**چرا مهم است؟** سه سال بعد، وقتی باگی پیدا می‌شود، کسی باید بفهمد کدام تغییر آن را آورد و چرا. اگر تاریخ بی‌کیفیت باشد، باید کل کد را بخواند. اگر خوب باشد، در دو دقیقه جواب می‌گیرد. همچنین: `git bisect`، `git blame`، changelog خودکار، و حتی Copilot — همه به کیفیت commit وابسته‌اند.

**کاربرد عملی:** بعد از این فصل، `git log` تو شبیه یک گزارش قابل خواندن می‌شود، نه یک لیست تصادفی.

### مفاهیم اساسی

#### یک commit از چه ساخته شده

```
commit 3f2a1b9c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a
Author:     علی رضایی <ali@example.com>
AuthorDate: Mon Oct 20 14:32:11 2026 +0330
Commit:     علی رضایی <ali@example.com>
CommitDate: Mon Oct 20 14:35:02 2026 +0330

    fix(auth): رفع خطای ورود با ایمیل بزرگ‌حرف

    ایمیل‌ها در ثبت‌نام lowercase می‌شدند ولی در ورود نه.
    حالا هر دو مسیر normalize می‌شوند.

diff --git a/src/auth.js b/src/auth.js
...
```

اجزا:

- **hash:** شناسه‌ی یکتا (۴۰ کاراکتر)
- **Author / AuthorDate:** چه کسی نوشت، کِی
- **Commit / CommitDate:** چه کسی commit زد (مثلاً بعد از rebase عوض می‌شود)
- **message:** عنوان + بدنه
- **parent:** هش commit قبلی
- **tree:** snapshot فایل‌ها

#### پیام commit — ساختار استاندارد

```
<type>(<scope>): <subject>

<body>

<footer>
```

- **type:** `feat`، `fix`، `docs`، `style`، `refactor`، `perf`، `test`، `build`، `ci`، `chore`، `revert`
- **scope:** بخشی از کد (`auth`، `api`، `ui`، `db`)
- **subject:** خلاصه در ~۵۰ کاراکتر، فعل امری، بدون نقطه‌ی آخر
- **body:** چرا این تغییر لازم بود، چه چیزی عوض شد، چه چیزی عوض نشد
- **footer:** `BREAKING CHANGE:`، `Closes #42`، `Co-authored-by:`

این **Conventional Commits** است. چرا مهم است؟ چون ابزارها می‌توانند از آن changelog و شماره‌ی نسخه بسازند (فصل ۶۳).

#### اتمیک بودن

یک commit = یک تغییر منطقی. سه تست:

1. **قابل بازگردانی؟** اگر فقط این commit را revert کنم، آیا کد کامپایل می‌شود؟
2. **قابل توصیف با یک جمله؟** اگر برای توصیفش «و» لازم داری، احتمالاً دو commit است.
3. **قابل بازبینی؟** آیا یک نفر می‌تواند در ۵ دقیقه بفهمد چه شد؟

| ✗ بد | ✓ خوب |
|---|---|
| `update` با ۳۰ فایل | `fix(auth): رفع نرمال‌سازی ایمیل` |
| `fix + refactor + bump deps` | سه commit جدا |
| `wip` | `wip` فقط روی branch شخصی، قبل از merge squash می‌شود |

#### diff چیست و چطور بخوانیمش

```
diff --git a/src/auth.js b/src/auth.js
index 1a2b3c4..5d6e7f8 100644
--- a/src/auth.js
+++ b/src/auth.js
@@ -10,7 +10,8 @@ function login(email, password) {
   const user = findUser(email);
-  if (!user) return null;
+  const normalized = email.trim().toLowerCase();
+  const user = findUser(normalized);
+  if (!user) return null;
   return verify(user, password);
 }
```

- `---` فایل قدیمی، `+++` فایل جدید
- `@@ -10,7 +10,8 @@` یعنی از خط ۱۰ فایل قدیمی ۷ خط، از خط ۱۰ فایل جدید ۸ خط
- خطوط `-` حذف‌شده، `+` اضافه‌شده، بدون علامت = context

### مثال‌های عملی

#### مثال ۱: commit چندخطی با ادیتور

```bash
git commit
# ادیتور باز می‌شود:
```

```
fix(auth): رفع خطای ورود با ایمیل بزرگ‌حرف

کاربری که با Ali@Example.com ثبت‌نام می‌کرد، بعداً
نمی‌توانست با همان ایمیل وارد شود، چون در ثبت‌نام
lowercase می‌شد ولی در ورود نه.

- تابع normalizeEmail اضافه شد
- در هر دو مسیر login و register استفاده می‌شود
- تست regression اضافه شد

Closes #128
```

**قاعده:** خط اول ≤ ۵۰ کاراکتر، خط خالی بعدش، بدنه با ۷۲ کاراکتر شکسته.

#### مثال ۲: commit با چند نویسنده

```
feat(search): افزودن جست‌وجوی فازی

Co-authored-by: مریم احمدی <maryam@example.com>
Co-authored-by: Copilot <copilot@github.com>
```

GitHub هر `Co-authored-by` را به‌عنوان contributor نشان می‌دهد.

#### مثال ۳: `git commit --fixup` برای اصلاح بعدی

```bash
# فرض کن commit های a1b2c3d و e4f5g6h داری
# می‌خواهی تغییر جدیدت به a1b2c3d بچسبد

git add .
git commit --fixup=a1b2c3d

# بعداً با rebase خودکار مرتبش کن
git rebase -i --autosquash main
```

#### مثال ۴: دیدن diff به شکل‌های مختلف

```bash
git diff                          # working vs staging
git diff --staged                 # staging vs آخرین commit
git diff HEAD                     # working vs آخرین commit
git diff main..feature            # بین دو branch
git diff --stat                   # خلاصه‌ی آماری
git diff --name-only              # فقط نام فایل‌ها
git diff -w                       # نادیده گرفتن تغییرات whitespace
git diff --word-diff              # تغییرات کلمه‌به‌کلمه (برای متن)
```

#### مثال ۵: `git commit --amend` — اصلاح آخرین commit

```bash
# پیام را اشتباه نوشتی
git commit --amend -m "fix(auth): رفع نرمال‌سازی ایمیل"

# فایلی را جا انداختی
git add forgotten.js
git commit --amend --no-edit      # پیام را نگه دار

# نویسنده را عوض کن
git commit --amend --author="علی رضایی <ali@work.com>" --no-edit
```

**هشدار:** amend هش را عوض می‌کند. اگر commit را push کرده‌ای، باید `push --force-with-lease` بزنی — و اگر دیگران رویش کار کرده‌اند، **این کار را نکن** (فصل ۱۷).

### مقایسه و الگوهای مختلف

#### سبک‌های پیام commit

| سبک | نمونه | مزیت | عیب |
|---|---|---|---|
| **Conventional Commits** | `feat(api): افزودن endpoint` | ابزارپذیر، changelog خودکار | کمی رسمی |
| **Gitmoji** | `✨ feat: افزودن endpoint` | بصری | ایموجی در ترمینال/ایمیل بد |
| **فعل امری ساده** | `Add endpoint` | ساده | بدون ساختار، ابزارپذیر نیست |
| **Ticket-first** | `PROJ-123 افزودن endpoint` | اتصال به Jira | وابسته به ابزار |

**توصیه:** Conventional Commits، چون هم ساده است هم ابزارها (semantic-release، changesets) با آن کار می‌کنند.

#### `-m` در برابر ادیتور

| | `-m "..."` | `git commit` (ادیتور) |
|---|---|---|
| سرعت | سریع | کندتر |
| پیام چندخطی | سخت (`-m` چندبار) | راحت |
| مناسب | تغییر کوچک | تغییر مهم |

```bash
# چند -m = چند پاراگراف
git commit -m "fix(auth): رفع نرمال‌سازی" -m "ایمیل‌ها حالا در هر دو مسیر lowercase می‌شوند." -m "Closes #128"
```

### اشکال شایع

#### ۱. پیام‌های «wip» و «asdf»

روی branch شخصی مشکلی ندارد، **اگر** قبل از merge squash کنی. ولی اگر مستقیم روی `main` می‌زنی، تاریخ را نابود می‌کنی.

#### ۲. commit غول‌پیکر

```
✗ 47 files changed, 3120 insertions(+), 890 deletions(-)
```

غیرقابل بازبینی، غیرقابل revert، غیرقابل bisect. اگر مجبوری، حداقل در PR توضیح بده چرا.

#### ۳. مخلوط کردن refactor و feature

```
✗ refactor: تغییر نام متغیرها + feat: افزودن جست‌وجو
```

اگر جست‌وجو باگ داشته باشد، revert کردنش refactor را هم برمی‌گرداند. جدا کن.

#### ۴. commit کردن فایل تولیدشده

```
✗ dist/bundle.js
✗ *.pyc
✗ .env
```

اینها باید در `.gitignore` باشند.

#### ۵. `git commit -a` روی مخزن بزرگ

ممکن است فایل‌هایی را commit کنی که نمی‌خواستی. `git add -p` امن‌تر است.

#### ۶. پیام commit با کاراکترهای مشکل‌دار در ویندوز

```bash
# در CMD/PowerShell، کوتیشن‌ها ممکن است بلاک شوند
git commit -m "fix: رفع `temp` variable"   # بک‌تیک مشکل می‌سازد
```

راه‌حل: از Git Bash استفاده کن یا کوتیشن تک‌نما بزن.

### نکات مهم (Tips & Tricks)

- **`git commit -v`** diff را در پایین پیام نشان می‌دهد — یادت می‌اندازد چه چیزی commit می‌کنی.
- **`git commit --fixup` + `--autosquash`** بهترین ترکیب برای تاریخ تمیز.
- **`git log --oneline -20`** بعد از هر کار، برای دیدن اینکه تاریخ چه شکلی است.
- **قالب commit برای تیم بساز:** `.gitmessage` و `git config commit.template ~/.gitmessage`.
- **`git commit --allow-empty -m "..."`** برای trigger کردن CI یا علامت‌گذاری.
- **در PR، از squash استفاده کن** اگر commitهای وسطی «wip» هستند — ولی commitهای محلی را تمیز بنویس، چون در rebase/merge ممکن است لازم شوند.

### خلاصه فصل

- پیام commit ساختار دارد: **type(scope): subject** + body + footer.
- **اتمیک بودن** سه تست دارد: قابل revert، قابل توصیف با یک جمله، قابل بازبینی.
- `git diff` سه شکل اصلی دارد: working↔staging، staging↔HEAD، branch↔branch.
- `--amend` آخرین commit را اصلاح می‌کند ولی هش را عوض می‌کند.
- **Conventional Commits** پایه‌ی changelog و نسخه‌بندی خودکار است.

**سؤالات مروری:**
1. چرا «و» در توصیف commit نشانه‌ی بدی است؟
2. تفاوت `git diff` و `git diff --staged` چیست؟
3. چرا `--amend` روی commitهای push‌شده خطرناک است؟

---

## فصل ۸: `.gitignore` و مدیریت فایل‌های ناخواسته

### مقدمه

هر پروژه‌ای فایل‌هایی دارد که نباید در مخزن باشند: `node_modules/`، `.venv/`، `dist/`، `.env`، فایل‌های IDE، لاگ‌ها. `.gitignore` اینها را نادیده می‌گیرد.

**چرا مهم است؟** دو دلیل جدی: (۱) حجم — `node_modules` می‌تواند ۵۰۰ مگابایت باشد و clone را برای همه کُند کند؛ (۲) امنیت — `.env` با پسورد دیتابیس، اگر یک بار push شود، برای همیشه در تاریخ می‌ماند.

**کاربرد عملی:** یک `.gitignore` خوب، ۹۰٪ مشکلات «فایل اشتباهی commit شد» را حذف می‌کند.

### مفاهیم اساسی

#### قواعد نگارش

```gitignore
# کامنت

# یک فایل مشخص
secret.txt

# همه‌ی فایل‌های یک پسوند
*.log
*.tmp

# یک پوشه (و همه‌ی محتوایش)
node_modules/
dist/
__pycache__/

# هر فایلی با این نام، در هر عمقی
.DS_Store
Thumbs.db

# فقط در ریشه
/config.yml

# الگوی glob
build-*/
*.min.js

# استثنا (با !)
*.log
!important.log        # important.log را نادیده نگیر

# استثنای پوشه
dist/*
!dist/keep.txt
```

**نکات ظریف:**

- `/` در ابتدا = نسبت به ریشه‌ی `.gitignore`
- `/` در انتها = فقط پوشه
- `**` = هر تعداد سطح پوشه (`**/temp/*.txt`)
- `!` = استثنا — **ولی** اگر پوشه‌ای ignore شده باشد، نمی‌توانی فایلی داخلش را با `!` برگردانی (مگر اینکه خود پوشه را هم استثنا کنی)
- ترتیب مهم است: آخرین قاعده برنده است

#### سه فایل، سه دامنه

| فایل | دامنه | در مخزن؟ | برای چه |
|---|---|---|---|
| `.gitignore` | همه‌ی کسانی که clone می‌کنند | ✅ بله | فایل‌های پروژه (build، deps) |
| `.git/info/exclude` | فقط تو، فقط این مخزن | ❌ نه | فایل‌های شخصی (تنظیمات IDE من) |
| `~/.config/git/ignore` (یا `core.excludesFile`) | فقط تو، همه‌ی مخزن‌ها | ❌ نه | عادت‌های شخصی (`.DS_Store`، `.idea/`) |

**قاعده:** چیزی که برای همه مهم است → `.gitignore`. چیزی که فقط مال توست → `exclude`.

#### `.gitkeep` — ترفند پوشه‌ی خالی

Git پوشه‌ی خالی را ردیابی نمی‌کند. اگر لازم است ساختار پوشه حفظ شود:

```bash
mkdir -p uploads
touch uploads/.gitkeep
echo "uploads/*" >> .gitignore
echo "!uploads/.gitkeep" >> .gitignore
```

### مثال‌های عملی

#### مثال ۱: `.gitignore` کامل برای Node.js

```gitignore
# وابستگی‌ها
node_modules/
.pnp.*
.yarn/*

# build
dist/
build/
.next/
out/
*.tsbuildinfo

# محیط
.env
.env.local
.env.*.local
!.env.example

# لاگ
logs/
*.log
npm-debug.log*
yarn-error.log*

# تست
coverage/
.nyc_output/
playwright-report/

# سیستم‌عامل
.DS_Store
Thumbs.db
Desktop.ini

# IDE
.vscode/*
!.vscode/extensions.json
!.vscode/settings.json
.idea/

# موقت
*.swp
*.swo
.cache/
```

**نکته:** `!.vscode/settings.json` را نگه می‌داریم چون تنظیمات مشترک تیم مفید است (مثلاً فرمت خودکار). ولی `.vscode/launch.json` شخصی است.

#### مثال ۲: `.gitignore` برای Python

```gitignore
__pycache__/
*.py[cod]
*.so
.Python
build/
dist/
*.egg-info/
.eggs/

.venv/
venv/
env/

.pytest_cache/
.mypy_cache/
.ruff_cache/
htmlcov/
.coverage
.coverage.*

.env
.env.*
!.env.example

.ipynb_checkpoints/
```

#### مثال ۳: پاک کردن فایلی که قبلاً commit شده

`.gitignore` فقط روی فایل‌های **ردیابی‌نشده** اثر دارد. اگر فایل قبلاً commit شده، اضافه کردنش به `.gitignore` کافی نیست:

```bash
# ۱) از ایندکس حذف کن (فایل روی دیسک می‌ماند)
git rm -r --cached node_modules

# ۲) مطمئن شو در .gitignore هست
echo "node_modules/" >> .gitignore

# ۳) commit
git add .gitignore
git commit -m "chore: توقف ردیابی node_modules"

# ۴) اگر حساس بود، تاریخ را هم پاک کن (فصل ۹۳)
```

#### مثال ۴: بررسی اینکه چرا یک فایل ignore شده

```bash
git check-ignore -v node_modules/express/index.js
# .gitignore:2:node_modules/    node_modules/express/index.js
```

این دستور می‌گوید کدام خط از کدام فایل، آن مسیر را ignore کرده. برای دیباگ `.gitignore` عالی است.

#### مثال ۵: دیدن فایل‌های ignore‌شده

```bash
git status --ignored --short
# !! .env
# !! node_modules/
# !! dist/
```

#### مثال ۶: ignore کردن فایل محلی بدون دست زدن به `.gitignore` مشترک

```bash
# .git/info/exclude
echo "config.local.yml" >> .git/info/exclude
```

این تغییر commit نمی‌شود و بقیه نمی‌بینندش.

#### مثال ۷: `.gitattributes` — تنظیمات فایل به‌جای ignore

`.gitignore` می‌گوید «ردیابی نکن». `.gitattributes` می‌گوید «چطور رفتار کن»:

```gitattributes
# همه‌ی فایل‌های متنی با LF
* text=auto eol=lf

# فایل‌های باینری
*.png binary
*.jpg binary
*.pdf binary
*.zip binary

# در diff به‌عنوان باینری دیده شوند
*.lock -diff

# زبان GitHub برای آمار
*.ipynb linguist-language=Jupyter
docs/* linguist-documentation

# LFS (فصل ۸۶)
*.psd filter=lfs diff=lfs merge=lfs -text
```

**`*.lock -diff`** عالی است: `package-lock.json` تغییرات غول‌پیکر در PR نشان نمی‌دهد.

### مقایسه و الگوهای مختلف

#### `.gitignore` در برابر `.gitattributes`

| | `.gitignore` | `.gitattributes` |
|---|---|---|
| هدف | نادیده گرفتن فایل | تنظیم رفتار Git روی فایل |
| کاربرد | build، deps، secret | پایان خط، باینری، LFS، diff |
| روی فایل ردیابی‌شده | اثری ندارد | اثر دارد |

#### منابع آماده

| منبع | چرا خوب است |
|---|---|
| github.com/github/gitignore | قالب رسمی برای ۵۰۰+ زبان/ابزار |
| gitignore.io | تولید تعاملی بر اساس tech stack |
| `gh repo create` | خودش `.gitignore` مناسب می‌گذارد |

### اشکال شایع

#### ۱. `.gitignore` نوشتم ولی فایل هنوز در status است

فایل قبلاً ردیابی می‌شود. `git rm --cached` لازم است.

#### ۲. `!` کار نمی‌کند

اگر پوشه‌ی والد ignore شده باشد، استثنا در فرزند کار نمی‌کند:

```gitignore
✗ dist/
  !dist/keep.txt      ← کار نمی‌کند

✓ dist/*
  !dist/keep.txt      ← کار می‌کند
```

#### ۳. `.env.example` را هم ignore کردم

اشتباه رایج. `.env` را ignore کن، ولی `.env.example` را **commit کن** تا بقیه بدانند چه متغیرهایی لازم است:

```gitignore
.env
!.env.example
```

#### ۴. فایل `.gitignore` را در زیرپوشه گذاشتم

کار می‌کند (دامنه‌اش همان زیرپوشه است) ولی گیج‌کننده است. یک `.gitignore` در ریشه + `exclude` برای شخصی، ساده‌تر است.

#### ۵. `node_modules` را commit کردم و حالا مخزن ۲GB است

```bash
git rm -r --cached node_modules
echo "node_modules/" >> .gitignore
git commit -m "chore: حذف node_modules"
# تاریخچه هنوز بزرگ است — برای کوچک کردن باید rewrite کنی (git filter-repo)
```

### نکات مهم (Tips & Tricks)

- **همیشه `.gitignore` را در اولین commit اضافه کن.** بعداً اضافه کردنش دردسر دارد.
- **`git check-ignore -v`** بهترین ابزار دیباگ.
- **الگوهای خیلی وسیع ننویس:** `*.json` ممکن است `package.json` را هم بگیرد.
- **فایل‌های تولیدشده را commit نکن:** `dist/`، `coverage/`، `*.pyc` — مگر اینکه پروژه‌ات واقعاً به آن نیاز داشته باشد (مثل سایت Pages که باید build را داشته باشد، ولی آنجا هم Actions بهتر است).
- **`core.excludesFile`** را برای عادت‌های شخصی تنظیم کن:
  ```bash
  git config --global core.excludesFile ~/.config/git/ignore
  echo ".DS_Store" >> ~/.config/git/ignore
  echo ".idea/" >> ~/.config/git/ignore
  ```

### خلاصه فصل

- `.gitignore` فقط روی فایل‌های **ردیابی‌نشده** اثر دارد.
- سه دامنه: `.gitignore` (مشترک)، `.git/info/exclude` (شخصی/مخزن)، `core.excludesFile` (شخصی/همه‌جا).
- `git rm --cached` فایل ردیابی‌شده را از ایندکس خارج می‌کند.
- `!` برای استثنا، ولی پوشه‌ی والد باید با `/*` ignore شده باشد.
- `.gitattributes` تنظیمات رفتاری است، نه نادیده گرفتن.

**سؤالات مروری:**
1. چرا اضافه کردن `.env` به `.gitignore` بعد از commit کافی نیست؟
2. تفاوت `.gitignore` و `.git/info/exclude` چیست؟
3. چطور بفهمی کدام قاعده یک فایل را ignore کرده؟

---

## فصل ۹: کاوش در تاریخ — log، show، blame و جست‌وجو

### مقدمه

تاریخ Git یک پایگاه داده است، نه فقط یک لیست. می‌توانی بپرسی «چه کسی این خط را نوشت؟»، «کِی این تابع اضافه شد؟»، «کدام commit کلمه‌ی `stripe` را آورد؟»، «بین دو نسخه چه چیزی عوض شد؟». کسی که این ابزارها را بلد است، در دیباگ ده برابر سریع‌تر است.

**چرا مهم است؟** ۸۰٪ دیباگ، پیدا کردن «چه زمانی و چرا این کد این‌طوری شد» است. `git log` و `git blame` این را از ساعت‌ها به دقیقه می‌رسانند.

**کاربرد عملی:** وقتی باگ در production پیدا می‌شود، اول `git log` فایل مربوطه، بعد `git blame` خط مشکوک، بعد `git show` آن commit — و معمولاً همان‌جا جواب پیدا می‌شود.

### مفاهیم اساسی

#### `git log` — پایه

```bash
git log                          # تاریخ کامل با جزئیات
git log --oneline                # یک خط به ازای هر commit
git log --oneline -10            # ده commit آخر
git log --graph --oneline --all  # نمای درختی
git log --stat                   # فایل‌های تغییر‌یافته + آمار
git log -p                       # با diff کامل
git log --author="مریم"          # فیلتر نویسنده
git log --since="2 weeks ago"    # فیلتر زمان
git log --until="2026-09-01"
git log --grep="رفع"             # جست‌وجو در پیام
git log -S"stripe"               # جست‌وجو در محتوا (pickaxe)
git log --follow src/name.js     # دنبال کردن rename
```

#### `--graph` چطور بخوانیم

```
*   a1b2c3d (HEAD -> main) Merge branch 'feature/login'
|\
| * e4f5g6h (feature/login) feat(auth): افزودن OTP
| * i7j8k9l fix(auth): رفع انقضای توکن
|/
* m0n1o2p docs: به‌روزرسانی README
```

- `*` یک commit
- `|\` شاخه‌ی دوم (merge)
- `|/` ادغام شاخه
- `(HEAD -> main)` یعنی الان اینجا هستی

#### `git show`

```bash
git show                  # آخرین commit
git show a1b2c3d          # یک commit مشخص
git show HEAD~1           # یکی قبل از آخر
git show v1.0.0           # یک تگ
git show main:src/app.js  # محتوای یک فایل در یک ref (بدون checkout!)
```

آخری خیلی کاربردی است: می‌خواهی نسخه‌ی قبلی یک فایل را ببینی بدون اینکه چیزی را عوض کنی.

#### `git blame` — چه کسی این خط را نوشت

```bash
git blame src/auth.js
# 3f2a1b9c (علی رضایی 2026-09-14 10:22:03 +0330 12) function login(email) {
# e4f5g6h7 (مریم احمدی 2026-10-02 16:45:11 +0330 13)   const u = findUser(email)
```

ستون‌ها: هش کوتاه، نویسنده، تاریخ، شماره‌ی خط، محتوا.

**تله‌ی مهم:** `git blame` آخرین کسی را نشان می‌دهد که خط را **لمس** کرده — که ممکن است فقط یک تغییر whitespace باشد. برای دیدن تاریخ واقعی منطقی:

```bash
git blame -w src/auth.js          # نادیده گرفتن whitespace
git blame -C -C src/auth.js       # تشخیص کد کپی‌شده از فایل‌های دیگر
git blame -L 10,20 src/auth.js    # فقط خطوط ۱۰ تا ۲۰
```

### مثال‌های عملی

#### مثال ۱: پیدا کردن اینکه کِی یک تابع اضافه شد

```bash
# روش ۱: جست‌وجو در پیام‌ها
git log --oneline --all --grep="jostojoo" -i

# روش ۲: pickaxe — کِی رشته ظاهر/ناپدید شد
git log -S"function validateEmail" --oneline --all
# 9a8b7c6 feat(auth): افزودن اعتبارسنجی ایمیل
# 5d4e3f2 refactor(auth): انتقال اعتبارسنجی به utils

# روش ۳: تغییرات یک فایل خاص
git log --oneline -- src/auth.js
```

#### مثال ۲: `git log -L` — تاریخ یک بازه‌ی خطی

```bash
git log -L 10,20:src/auth.js
# تاریخچه‌ی خطوط ۱۰ تا ۲۰ را با diff نشان می‌دهد
```

این یکی از قدرتمندترین و کم‌شناخته‌شده‌ترین دستورهاست. برای «این ۱۰ خط از کجا آمدند؟» جواب کامل می‌دهد.

#### مثال ۳: مقایسه‌ی دو نسخه

```bash
# بین دو branch
git log main..feature --oneline         # commitهایی که در feature هست و در main نه
git log feature..main --oneline         # برعکس
git log main...feature --oneline        # تفاوت متقارن (هر دو طرف)

# diff کامل
git diff main...feature                 # از نقطه‌ی انشعاب
git diff main..feature                  # بین دو نوک

# خلاصه
git diff --stat main..feature
```

**تفاوت `..` و `...` — یک بار برای همیشه:**

```
        A ← B ← C ← D   (main)
             ↖
              E ← F     (feature)

git log main..feature   → E, F        (در feature، نه در main)
git log feature..main   → C, D        (در main، نه در feature)
git log main...feature  → C, D, E, F  (متقارن)
git diff main..feature  → مقایسه‌ی D با F
git diff main...feature → مقایسه‌ی B (merge-base) با F
```

#### مثال ۴: پیدا کردن commit که فایل را حذف کرد

```bash
git log --oneline --diff-filter=D -- "*.js"
# --diff-filter: A=added, D=deleted, M=modified, R=renamed

# محتوای فایل حذف‌شده را ببین
git show <hash>^:path/to/file.js
```

#### مثال ۵: جست‌وجو در همه‌ی branchها

```bash
git log --all --oneline -S"API_KEY"
# در همه‌ی شاخه‌ها می‌گردد
```

اگر `API_KEY` پیدا شد، یعنی یک کلید در تاریخ هست — حتی اگر الان در کد نباشد. این اولین قدم بررسی نشت secret است.

#### مثال ۶: `git shortlog` — خلاصه‌ی مشارکت

```bash
git shortlog -sn --all
#   142  علی رضایی
#    87  مریم احمدی
#    12  سارا کریمی
```

برای گزارش تیمی یا دیدن اینکه چه کسی بیشتر روی چه بخشی کار کرده.

#### مثال ۷: `git log` با فرمت سفارشی

```bash
git log --pretty=format:"%h %ad %an %s" --date=short
# a1b2c3d 2026-10-20 علی رضایی fix(auth): رفع نرمال‌سازی

# فرمت‌های مفید:
# %h هش کوتاه، %H کامل
# %an نویسنده، %ae ایمیل
# %ad تاریخ نویسنده، %cd تاریخ commit
# %s عنوان، %b بدنه
# %d refها (branch/tag)
```

### مقایسه و الگوهای مختلف

#### ابزارهای کاوش

| ابزار | برای چه | سرعت |
|---|---|---|
| `git log` | تاریخ عمومی | سریع |
| `git blame` | مالکیت خط | متوسط |
| `git log -S` (pickaxe) | جست‌وجوی محتوا در تاریخ | کند در مخزن بزرگ |
| `git log -G<regex>` | جست‌وجوی regex در diff | کند |
| `git bisect` | پیدا کردن commit باگ‌دار | O(log n) — خیلی سریع |
| `git grep <pat> <ref>` | جست‌وجو در یک نسخه | سریع |

#### `git grep` در برابر `grep -r`

| | `git grep` | `grep -r` |
|---|---|---|
| فقط فایل‌های ردیابی‌شده | ✅ | ❌ (شامل node_modules و ...) |
| سرعت | بالا (از ایندکس) | متوسط |
| جست‌وجو در ref | ✅ `git grep pat v1.0` | ❌ |

```bash
git grep "TODO"                     # در working tree
git grep "TODO" v1.0.0              # در تگ v1.0.0
git grep -n "console.log" -- "*.js" # با شماره خط، فقط js
git grep -l "deprecated"            # فقط نام فایل‌ها
```

### اشکال شایع

#### ۱. `git log` فقط یک خط نشان می‌دهد

چون ترمینال در حالت pager است و `q` نزده‌ای. یا `core.pager` تنظیم نیست. `q` بزن.

#### ۲. `git blame` می‌گوید همه‌چیز مال من است

چون یک commit بزرگ فرمت‌دهی یا rename زده‌ای. راه‌حل: `git blame -w` (whitespace) و `git blame -C` (کپی). یا از `git log -L` استفاده کن.

#### ۳. commit گم شد

`git reflog` (فصل ۲۲). تقریباً همیشه پیدا می‌شود.

#### ۴. `git log --follow` rename را دنبال نمی‌کند

`--follow` فقط برای **یک فایل** کار می‌کند، نه با `--all` یا چند مسیر. محدودیت شناخته‌شده‌ی Git است.

#### ۵. تاریخ به‌هم‌ریخته است

ممکن است چند نفر با ساعت اشتباه commit زده باشند، یا rebase تاریخ‌ها را عوض کرده باشد. `git log --date-order` یا `--topo-order` مرتب‌سازی منطقی‌تری می‌دهد.

### نکات مهم (Tips & Tricks)

- **`git lg`** (alias از فصل ۵) بهترین نمای روزمره است.
- **`git log --oneline --graph --all --decorate -20`** را در یک alias بگذار.
- **`git log -S`** برای «کِی این رشته آمد/رفت» بی‌رقیب است.
- **`git log --author="علی" --since="last month"`** برای گزارش کار ماهانه.
- **`git show main:file`** برای دیدن فایل در branch دیگر بدون checkout — فوق‌العاده برای مقایسه.
- **`git blame` را به‌عنوان ابزار سرزنش استفاده نکن.** برای فهمیدن است، نه برای پیدا کردن مقصر.

### خلاصه فصل

- `git log` با `--oneline`، `--graph`، `--stat`، `--author`، `--grep`، `-S` قابل فیلتر است.
- `git show <ref>:<path>` محتوای فایل در هر نسخه را می‌دهد.
- `git blame` آخرین لمس‌کننده‌ی خط را می‌دهد؛ با `-w` و `-C` دقیق‌تر می‌شود.
- `git log -L start,end:file` تاریخ یک بازه‌ی خطی را می‌دهد.
- `..` و `...` تفاوت مهمی دارند: «بین دو نقطه» در برابر «از نقطه‌ی انشعاب».
- `git grep` سریع‌تر و دقیق‌تر از `grep -r` است.

**سؤالات مروری:**
1. چطور بفهمی یک رشته‌ی خاص کِی به کد اضافه شد؟
2. تفاوت `git diff main..feature` و `git diff main...feature` چیست؟
3. چرا `git blame` ممکن است گمراه‌کننده باشد و چطور رفعش کنی؟

---

**بعدی:** [`03-Branching-and-Merging.md`](03-Branching-and-Merging.md)

---

## منابع و مراجع

- Pro Git، فصل ۲ (مبانی Git) — https://git-scm.com/book/fa/v2
- مستندات `gitignore` — https://git-scm.com/docs/gitignore
- مجموعه‌ی قالب‌های `.gitignore` — https://github.com/github/gitignore
- مستندات `gitattributes` — https://git-scm.com/docs/gitattributes
- مستندات `git log` — https://git-scm.com/docs/git-log
- راهنمای Conventional Commits — https://www.conventionalcommits.org
- gitignore.io (تولید تعاملی) — https://www.toptal.com/developers/gitignore
