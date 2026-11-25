# بخش ۰۳: شاخه‌بندی و ادغام

**مدت یادگیری**: ۳ ساعت
**سطح سختی**: مبتدی
**پیش‌نیاز**: بخش ۰۲ (مبانی Git)

---

## فهرست محتوا

- [فصل ۱: مفهوم Branch](#فصل-۱-مفهوم-branch)
- [فصل ۲: کار با شاخه‌ها](#فصل-۲-کار-با-شاخهها)
- [فصل ۳: ادغام (Merge)](#فصل-۳-ادغام-merge)
- [فصل ۴: حل تعارض (Conflict)](#فصل-۴-حل-تعارض-conflict)
- [فصل ۵: ابزارهای بصری Merge](#فصل-۵-ابزارهای-بصری-merge)
- [فصل ۶: استراتژی‌های Merge](#فصل-۶-استراتژیهای-merge)

---

## فصل ۱: مفهوم Branch

### مقدمه

**شاخه (Branch)** یکی از قدرتمندترین و در عین حال ساده‌ترین مفاهیم Git است. شاخه به شما اجازه می‌دهد بدون تأثیر بر کد اصلی، ویژگی جدیدی را توسعه دهید یا باگی را رفع کنید.

### اهمیت این موضوع

در سیستم‌های قدیمی مثل SVN، ساخت شاخه کند و پرهزینه بود. در Git، شاخه فقط یک **اشاره‌گر سبک** به یک commit است. ساخت شاخه در Git تقریباً بی‌هزینه و لحظه‌ای است؛ همین ویژگی باعث می‌شود تیم‌ها آزادانه شاخه بسازند.

### مفاهیم اساسی

تصور کنید commitها گره‌های یک زنجیر هستند:

```
A ← B ← C ← D  (main)
```

شاخه `main` فقط به commit `D` اشاره می‌کند. اگر شاخه جدیدی بسازید:

```
A ← B ← C ← D  (main)
             ↑
          (feature)
```

هر دو شاخه به `D` اشاره می‌کنند. حالا اگر روی `feature` کار کنید:

```
A ← B ← C ← D  (main)
             ↑
              E ← F  (feature)
```

شاخه `main` دست‌نخورده مانده. این «سبکی» شاخه‌ها، انقلاب Git است.

**`HEAD`** اشاره‌گر به شاخه فعلی است. وقتی `HEAD` به `feature` اشاره می‌کند، commitهای جدید روی `feature` می‌نشینند.

### اشکال شایع

- **اشتباه رایج**: ترس از ساخت شاخه. شاخه‌ها ارزان و بی‌خطرند؛ فقط نگران `push` ناخواسته باشید.
- **اشتباه رایج**: کار مستقیم روی `main` بدون شاخه. همیشه برای تغییرات جدید شاخه بسازید.

### نکات مهم (Tips & Tricks)

- یک قاعده طلایی: **`main` همیشه قابل انتشار باشد.** روی آن مستقیم کار نکنید.
- نام شاخه‌ها را معنادار بگذارید: `feature/user-login` نه `test1`.

### خلاصه فصل

شاخه یک اشاره‌گر سبک به یک commit است. ساخت شاخه در Git سریع و ارزان است و به شما امکان می‌دهد جدا از کد اصلی کار کنید. `HEAD` شاخه فعلی را نشان می‌دهد.

---

## فصل ۲: کار با شاخه‌ها

### مقدمه

حالا که مفهوم شاخه را فهمیدیم، دستورات عملی مدیریت آن را یاد می‌گیریم.

### مفاهیم اساسی

```bash
# ساخت شاخه
git branch feature/login

# جابه‌جایی به شاخه
git checkout feature/login
# یا (روش جدید)
git switch feature/login

# ساخت و جابه‌جایی همزمان
git switch -c feature/login
git checkout -b feature/login   # روش قدیمی

# مشاهده شاخه‌ها
git branch          # شاخه‌های محلی
git branch -a       # شامل راه دور
git branch -v       # با آخرین commit

# تغییر نام شاخه
git branch -m old-name new-name

# حذف شاخه
git branch -d feature/login      # فقط اگر merge شده
git branch -D feature/login      # اجبار حذف
```

### مثال‌های عملی

یک جریان کاری کامل:

```bash
# شروع از main
git switch main
git pull origin main

# ساخت شاخه جدید
git switch -c feature/add-login

# کار روی فایل
echo "def login(): pass" > auth.py
git add auth.py
git commit -m "feat: add login function"

# مشاهده وضعیت
git branch -v
git log --oneline --graph --all
```

### مقایسه و الگوهای مختلف

| دستور قدیمی | دستور جدید | کاربرد |
|-------------|------------|--------|
| `git checkout <branch>` | `git switch <branch>` | جابه‌جایی شاخه |
| `git checkout -b <branch>` | `git switch -c <branch>` | ساخت و جابه‌جایی |
| `git checkout -- <file>` | `git restore <file>` | بازگرداندن فایل |

دستورات جدید (`switch` و `restore`) از Git 2.23 معرفی شدند تا ابهام `checkout` را برطرف کنند.

### اشکال شایع

- **اشتباه رایج**: جابه‌جایی شاخه با تغییرات uncommitted. اگر تغییرات با شاخه مقصد تعارض داشته باشند، Git اجازه نمی‌دهد. اول commit یا stash کنید.
- **اشتباه رایج**: حذف شاخه‌ای که هنوز merge نشده با `-d`. Git هشدار می‌دهد؛ اگر مطمئنید از `-D` استفاده کنید.

### نکات مهم (Tips & Tricks)

- `git switch -` به شاخه قبلی برمی‌گردد (مثل `cd -`).
- قبل از حذف شاخه، با `git branch --merged` چک کنید کدام شاخه‌ها merge شده‌اند.
- شاخه‌ها را بعد از merge حذف کنید تا مخزن تمیز بماند.

### خلاصه فصل

با `git branch` شاخه‌ها را مدیریت و با `git switch` جابه‌جا می‌شوید. دستورات جدید `switch`/`restore` جایگزین شفاف‌تر `checkout` هستند. شاخه‌ها را پس از merge حذف کنید.

---

## فصل ۳: ادغام (Merge)

### مقدمه

پس از اتمام کار روی یک شاخه، باید آن را در شاخه اصلی ادغام (merge) کنید. Git دو نوع اصلی ادغام دارد: Fast-forward و Three-way.

### مفاهیم اساسی

**Fast-forward Merge:**

وقتی شاخه مقصد هیچ commit جدیدی ندارد، Git فقط اشاره‌گر را جلو می‌برد:

```
قبل:  A ← B ← C  (main)
                ↑
                 D ← E  (feature)

بعد:  A ← B ← C ← D ← E  (main, feature)
```

```bash
git switch main
git merge feature   # Fast-forward
```

**Three-way Merge:**

وقتی هر دو شاخه commit جدید دارند، Git یک commit ادغام جدید می‌سازد:

```
      D ← E  (feature)
     /      \
A ← B ← C ← M  (main)   ← M یک merge commit است
```

```bash
git merge feature
# خروجی: Merge made by the 'ort' strategy.
```

**Merge Commit**: commitی با دو والد (parent). پیام پیش‌فرض: `"Merge branch 'feature'"`.

### مثال‌های عملی

سناریوی Three-way:

```bash
# روی main یک commit
git switch main
echo "main change" >> file.txt
git commit -am "update on main"

# ادغام feature
git merge feature/add-login
# حالا یک merge commit ساخته می‌شود

git log --oneline --graph
```

### مقایسه و الگوهای مختلف

| نوع | شرط | نتیجه |
|-----|-----|-------|
| Fast-forward | مقصد بدون تغییر | جابه‌جایی اشاره‌گر |
| Three-way | هر دو تغییر کرده | merge commit جدید |

### اشکال شایع

- **اشتباه رایج**: ادغام شاخه اشتباه. همیشه قبل از merge با `git branch` مطمئن شوید روی شاخه درست هستید.
- **اشتباه رایج**: merge کردن `main` به feature به‌جای برعکس (به فصل استراتژی‌ها مراجعه کنید).

### نکات مهم (Tips & Tricks)

- `git merge --abort` یک merge نیمه‌کاره را لغو می‌کند.
- برای دیدن اینکه چه چیزی merge می‌شود: `git log main..feature --oneline`.
- قبل از merge، شاخه‌های راه دور را با `git fetch` به‌روز کنید.

### خلاصه فصل

Merge شاخه‌ها را ترکیب می‌کند. Fast-forward وقتی ممکن است ساده‌ترین حالت است؛ Three-way یک merge commit می‌سازد. همیشه قبل از merge مطمئن شوید روی شاخه درست هستید.

---

## فصل ۴: حل تعارض (Conflict)

### مقدمه

تعارض (Conflict) زمانی رخ می‌دهد که دو شاخه، یک خط یکسان را به شکل متفاوت تغییر داده باشند. Git نمی‌تواند تصمیم بگیرد کدام درست است و از شما کمک می‌خواهد. تعارض ترسناک نیست؛ فقط باید بدانید چطور آن را حل کنید.

### مفاهیم اساسی

وقتی تعارض رخ می‌دهد، Git در فایل نشانگرهایی می‌گذارد:

```
<<<<<<< HEAD
print("سلام از main")
=======
print("سلام از feature")
>>>>>>> feature
```

- بین `<<<<<<< HEAD` و `=======` : نسخه شاخه فعلی (main).
- بین `=======` و `>>>>>>> feature` : نسخه شاخه در حال ادغام.

**مراحل حل تعارض:**
1. فایل‌های دارای تعارض را ببینید: `git status`.
2. فایل را باز کنید و نسخه درست را انتخاب/ترکیب کنید.
3. نشانگرها را حذف کنید.
4. `git add <file>`.
5. `git commit` (یا `git merge --continue`).

### مثال‌های عملی

```bash
git switch main
git merge feature
# خروجی: CONFLICT (content): Merge conflict in file.txt
#         Automatic merge failed; fix conflicts and then commit the result.

# ۱. دیدن فایل‌های متعارض
git status

# ۲. ویرایش فایل (مثلاً با VS Code)
code file.txt
# نسخه نهایی را بنویسید و نشانگرها را حذف کنید

# ۳. افزودن و تکمیل
git add file.txt
git commit -m "merge: resolve conflict in file.txt"
```

**لغو merge:**
```bash
git merge --abort
```

**انتخاب کامل یک طرف:**
```bash
git checkout --ours file.txt    # نسخه شاخه فعلی
git checkout --theirs file.txt  # نسخه شاخه دیگر
```

### مقایسه و الگوهای مختلف

| وضعیت | راه‌حل |
|-------|--------|
| تعارض ساده | ویرایش دستی |
| انتخاب یک طرف کامل | `--ours` یا `--theirs` |
| انصراف از merge | `git merge --abort` |
| تعارض زیاد | استفاده از mergetool |

### اشکال شایع

- **اشتباه رایج**: فراموش کردن حذف نشانگرها (`<<<<<<<`). کد شکسته باقی می‌ماند.
- **اشتباه رایج**: panic کردن و حذف کل پروژه. `git merge --abort` همه‌چیز را برمی‌گرداند.
- **اشتباه رایج**: commit کردن بدون تست پس از حل تعارض.

### نکات مهم (Tips & Tricks)

- بعد از حل تعارض، همیشه تست‌ها را اجرا کنید.
- `git diff --check` وجود نشانگرهای باقی‌مانده را تشخیص می‌دهد.
- برای تعارض‌های پیچیده، از VS Code یا `git mergetool` استفاده کنید.

### خلاصه فصل

تعارض وقتی رخ می‌دهد که دو شاخه یک خط را متفاوت تغییر دهند. با ویرایش دستی، حذف نشانگرها، `add` و `commit` حل می‌شود. `git merge --abort` راه فرار است.

---

## فصل ۵: ابزارهای بصری Merge

### مقدمه

حل تعارض در فایل‌های بزرگ با متن دشوار است. ابزارهای بصری (Merge Tools) این کار را ساده می‌کنند.

### مفاهیم اساسی

ابزارهای محبوب:
- **VS Code**: داخلی و رایگان، با دکمه‌های «Accept Current/Incoming/Both».
- **Meld**: رایگان و چندسکویی.
- **Beyond Compare**: تجاری و قدرتمند.
- **KDiff3**: رایگان.

**پیکربندی mergetool:**
```bash
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait --merge $REMOTE $LOCAL $BASE $MERGED'
git config --global mergetool.keepBackup false
```

### مثال‌های عملی

```bash
# اجرای mergetool برای تعارض‌ها
git mergetool

# VS Code سه‌پنلی باز می‌کند:
# چپ: Incoming (feature)
# راست: Current (main)
# پایین: نتیجه نهایی
```

### اشکال شایع

- **اشتباه رایج**: ذخیره نکردن فایل پس از ویرایش در mergetool.
- **اشتباه رایج**: گیج شدن بین LOCAL و REMOTE. در merge، LOCAL همان شاخه فعلی است.

### نکات مهم (Tips & Tricks)

- VS Code با افزونه GitLens تجربه عالی‌ای می‌دهد.
- `git mergetool` فایل‌های `.orig` می‌سازد؛ با `mergetool.keepBackup false` غیرفعالش کنید.

### خلاصه فصل

ابزارهای بصری حل تعارض را ساده می‌کنند. VS Code گزینه پیشنهادی است. با `git mergetool` می‌توانید آن‌ها را به Git وصل کنید.

---

## فصل ۶: استراتژی‌های Merge

### مقدمه

Merge فقط یک دستور نیست؛ چند استراتژی دارد که هرکدام تاریخچه متفاوتی می‌سازند. انتخاب درست، تاریخچه را خوانا می‌کند.

### مفاهیم اساسی

**Merge معمولی (Three-way):** حفظ ساختار شاخه‌ها با merge commit.

**Fast-forward only:** فقط اگر امکان fast-forward باشد merge می‌کند:
```bash
git merge --ff-only feature
```

**No-fast-forward:** حتی اگر ممکن باشد، merge commit می‌سازد (تاریخچه شاخه‌ای واضح):
```bash
git merge --no-ff feature
```

**Squash merge:** همه commitهای شاخه را در یک commit فشرده می‌کند:
```bash
git merge --squash feature
git commit -m "feat: add login feature"
```

**Rebase:** تاریخچه را خطی می‌کند (در بخش ۰۵ کامل توضیح می‌دهیم).

### مقایسه و الگوهای مختلف

| استراتژی | تاریخچه | کاربرد |
|----------|---------|--------|
| Fast-forward | خطی | تغییرات کوچک |
| `--no-ff` | شاخه‌ای واضح | حفظ context |
| Squash | یک commit | PRهای پرcommit |
| Rebase | خطی تمیز | تاریخچه خوانا |

### مثال‌های عملی

```bash
# Merge با حفظ merge commit
git switch main
git merge --no-ff feature/add-login -m "merge: add login feature"

# Squash merge
git merge --squash feature/add-login
git commit -m "feat: add login feature (squashed)"
```

### اشکال شایع

- **اشتباه رایج**: squash کردن شاخه‌ای که در حال کار مشترک است. این کار تاریخچه را از دست می‌دهد.
- **اشتباه رایج**: rebase کردن شاخه‌های عمومی (در بخش ۰۵ هشدار می‌دهیم).

### نکات مهم (Tips & Tricks)

- برای PRها، `--no-ff` یا squash بسته به سیاست تیم مناسب است.
- `--ff-only` از ساخت ناخواسته merge commit جلوگیری می‌کند.
- استراتژی را در تیم استاندارد کنید تا همه یکسان عمل کنند.

### خلاصه فصل

چهار استراتژی اصلی merge داریم: fast-forward، `--no-ff`، squash و rebase. هرکدام تاریخچه متفاوتی می‌سازند. انتخاب درست به سیاست تیم و ماهیت تغییر بستگی دارد.

---

## منابع و مراجع

- مستندات Branching: [https://git-scm.com/book/fa/v2/Git-Branching-Branches-in-a-Nutshell](https://git-scm.com/book/fa/v2/Git-Branching-Branches-in-a-Nutshell)
- حل تعارض: [https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)
- راهنمای Atlassian درباره Merge: [https://www.atlassian.com/git/tutorials/using-branches/git-merge](https://www.atlassian.com/git/tutorials/using-branches/git-merge)
- VS Code Merge Tool: [https://code.visualstudio.com/docs/editor/versioncontrol](https://code.visualstudio.com/docs/editor/versioncontrol)
