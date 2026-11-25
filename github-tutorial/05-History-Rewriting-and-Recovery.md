# بخش ۰۵: بازنویسی تاریخ و بازیابی

**مدت یادگیری**: ۵ تا ۶ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۰۳ و ۰۴ (branch، merge، rebase، push)

---

## فهرست محتوا

- [فصل ۲۰: اصلاح و بازگردانی — amend، restore، reset](#فصل-۲۰)
- [فصل ۲۱: revert — بازگردانی امن در تاریخ مشترک](#فصل-۲۱)
- [فصل ۲۲: reflog — شبکه‌ی نجات](#فصل-۲۲)
- [فصل ۲۳: cherry-pick و stash — انتقال کار](#فصل-۲۳)
- [فصل ۲۴: bisect — پیدا کردن commit مقصر](#فصل-۲۴)

---

## فصل ۲۰: اصلاح و بازگردانی — amend، restore، reset

### مقدمه

این فصل سه دستور را پوشش می‌دهد که بیشترین استفاده و بیشترین سوءاستفاده را دارند. تفاوتشان در یک چیز است: **چه چیزی را جابه‌جا می‌کنند**.

- `restore` → فایل‌ها را
- `reset` → اشاره‌گر branch را (و اختیاراً فایل‌ها را)
- `amend` → آخرین commit را

**چرا مهم است؟** `git reset --hard` یکی از معدود دستورهایی است که می‌تواند کار واقعی را نابود کند. اگر بدانی دقیقاً چه می‌کند، می‌توانی بدون ترس از `reset --soft` و `--mixed` استفاده کنی — که بسیار کاربردی‌اند.

**کاربرد عملی:** «commit زدم ولی یک فایل جا افتاد»، «می‌خواهم سه commit آخر را یکی کنم»، «فایل را خراب کردم و می‌خواهم به نسخه‌ی قبلی برگردم» — همه همین‌جا.

### مفاهیم اساسی

#### سه حالت `reset`

`git reset` دو کار می‌کند: (۱) اشاره‌گر branch را جابه‌جا می‌کند، (۲) اختیاراً staging و working directory را.

```
           --soft        --mixed       --hard
HEAD ────────●─────────────●─────────────●
              ↓             ↓             ↓
           (فقط HEAD)   (+ index)    (+ working dir)
```

| حالت | HEAD | Staging | Working Directory | تغییرات از دست می‌رود؟ |
|---|---|---|---|---|
| `--soft` | جابه‌جا | دست‌نخورده | دست‌نخورده | ❌ |
| `--mixed` (پیش‌فرض) | جابه‌جا | ریست | دست‌نخورده | ❌ |
| `--hard` | جابه‌جا | ریست | **ریست** | ✅ |

**تصویر ذهنی:**

```
--soft   → "commit را باز کن، تغییرات را در staging نگه دار"
--mixed  → "commit را باز کن، تغییرات را در working dir نگه دار"
--hard   → "همه‌چیز را پاک کن، مثل اینکه commit هرگز نبوده"
```

#### `restore` — جانشین مدرن `checkout --`

```bash
git restore file.txt                # تغییرات working dir را دور بریز
git restore --staged file.txt       # از staging خارج کن (تغییرات می‌ماند)
git restore --source=HEAD~2 file.txt  # نسخه‌ی دو commit قبل
git restore --staged --worktree file.txt  # هر دو
git restore .                       # همه‌ی فایل‌ها (خطرناک!)
```

#### `reset` در برابر `restore` در برابر `revert`

| دستور | چه چیزی را عوض می‌کند | تاریخ را می‌شکند؟ | برای تاریخ مشترک؟ |
|---|---|---|---|
| `restore` | فایل‌ها | ❌ | ✅ |
| `reset` | HEAD + اختیاراً فایل‌ها | ✅ | ❌ |
| `revert` | یک commit جدید می‌سازد | ❌ | ✅ |

#### `amend` چه می‌کند

`git commit --amend` در واقع یک commit **جدید** می‌سازد که جای قبلی را می‌گیرد:

```
قبل:  A ← B ← C
بعد:  A ← B ← C'     (C' هش جدید دارد، C در reflog می‌ماند)
```

پس amend هم تاریخ را بازنویسی می‌کند.

### مثال‌های عملی

#### مثال ۱: فایل جا افتاده

```bash
git commit -m "feat: افزودن فرم"
# اوه، forgot.js را add نکردم

git add forgot.js
git commit --amend --no-edit
# همان پیام، فایل اضافه شد
```

#### مثال ۲: پیام اشتباه

```bash
git commit --amend -m "feat(auth): افزودن فرم ورود"
```

#### مثال ۳: سه commit آخر را یکی کن (با reset --soft)

```bash
git log --oneline -4
# d4e5f6g feat: بخش سوم
# c3d4e5f wip
# b2c3d4e feat: بخش اول
# a1b2c3d docs: قبلی

git reset --soft HEAD~3
# حالا HEAD روی a1b2c3d است، همه‌ی تغییرات در staging

git commit -m "feat: افزودن کامل فرم ورود"
```

این یکی از پرکاربردترین الگوهاست: `reset --soft` + یک commit جدید = squash سریع.

#### مثال ۴: تقسیم یک commit به دو

```bash
git reset --mixed HEAD~1     # commit باز شد، تغییرات در working dir
git add -p                   # بخش اول
git commit -m "refactor: جدا کردن اعتبارسنجی"
git add -p                   # بخش دوم
git commit -m "feat: افزودن فرم"
```

#### مثال ۵: بازگرداندن یک فایل به نسخه‌ی قبل

```bash
git restore src/app.js
# تغییرات uncommitted از بین رفت

# نسخه‌ی commit قبلی
git restore --source=HEAD~1 src/app.js

# نسخه‌ی یک branch دیگر
git restore --source=main src/app.js

# نسخه‌ی یک commit مشخص
git restore --source=a1b2c3d src/app.js
```

#### مثال ۶: خارج کردن فایل از staging

```bash
git add .
git restore --staged .env
git status --short
# ?? .env
```

#### مثال ۷: `reset --hard` — و چطور از آن فرار کنی

```bash
# فرض: این کار را کردی و پشیمان شدی
git reset --hard HEAD~2

# راه نجات:
git reflog
# a1b2c3d HEAD@{0}: reset: moving to HEAD~2
# e4f5g6h HEAD@{1}: commit: feat: بخش دوم
# c3d4e5f HEAD@{2}: commit: feat: بخش اول

git reset --hard e4f5g6h    # برگشتی
```

**درس:** `--hard` تغییرات **uncommitted** را برای همیشه نابود می‌کند. commitها را می‌شود برگرداند، فایل‌های ذخیره‌نشده را نه.

#### مثال ۸: برگرداندن فایل حذف‌شده

```bash
git rm src/old.js
git commit -m "chore: حذف فایل قدیمی"

# پشیمان شدی
git restore --source=HEAD~1 src/old.js
# یا
git checkout HEAD~1 -- src/old.js
```

#### مثال ۹: `git reset` روی یک فایل (بدون دست زدن به HEAD)

```bash
git reset HEAD src/app.js     # فایل را از staging خارج کن
# معادل git restore --staged src/app.js
```

**توجه:** وقتی مسیر می‌دهی، `reset` HEAD را جابه‌جا نمی‌کند — فقط ایندکس را.

### مقایسه و الگوهای مختلف

#### `restore` در برابر `checkout`

| کار | مدرن | قدیمی |
|---|---|---|
| دور ریختن تغییرات فایل | `git restore f` | `git checkout -- f` |
| خارج کردن از staging | `git restore --staged f` | `git reset HEAD f` |
| گرفتن نسخه از commit دیگر | `git restore --source=X f` | `git checkout X -- f` |
| عوض کردن branch | `git switch b` | `git checkout b` |

#### `reset --soft` در برابر `rebase -i` برای squash

| | `reset --soft` | `rebase -i` |
|---|---|---|
| سرعت | سریع | کندتر |
| کنترل پیام | یک پیام جدید | می‌توانی هر commit را reword کنی |
| حفظ ترتیب | همه با هم | انتخابی |
| مناسب | «همه را یکی کن» | «چند تا را با هم، بقیه جدا» |

#### `--hard` در برابر `--keep` در برابر `--merge`

| گزینه | رفتار |
|---|---|
| `--hard` | همه‌چیز را ریست کن |
| `--keep` | اگر فایل‌های محلی با مقصد تعارض دارند، متوقف شو |
| `--merge` | سعی کن تغییرات محلی را نگه داری، conflict بده |

`--keep` امن‌تر از `--hard` است وقتی می‌خواهی branch را جابه‌جا کنی ولی مطمئن نیستی تغییرات محلی‌ات ارزش دارند.

### اشکال شایع

#### ۱. `reset --hard` روی تغییرات uncommitted

کار رفته. `reflog` کمکی نمی‌کند چون چیزی commit نشده بود. تنها امید: ابزارهای بازیابی فایل سیستم، یا اگر ادیتور history دارد (VS Code Timeline).

**پیشگیری:** قبل از هر عملیات خطرناک، `git stash` یا `git commit -m wip`.

#### ۲. `reset` روی commit push‌شده

```bash
✗ git reset --hard HEAD~1 && git push --force
```

اگر دیگران آن commit را دارند، تاریخشان می‌شکند. برای تاریخ مشترک از `revert` استفاده کن (فصل ۲۱).

#### ۳. `restore` همه‌چیز را پاک کرد

`git restore .` تمام تغییرات uncommitted را دور می‌ریزد. اگر commit نشده بودند، رفته.

#### ۴. `amend` روی commit push‌شده

هش عوض می‌شود، push رد می‌شود، `--force-with-lease` لازم است. اگر روی branch مشترک است، **نکن**.

#### ۵. `reset` و detached HEAD

اگر در detached HEAD باشی، `reset` فقط HEAD را جابه‌جا می‌کند، نه هیچ branchی. نتیجه: commitها «گم» می‌شوند (در reflog هستند).

#### ۶. `reset --soft` روی merge commit

```bash
git reset --soft HEAD~1    # merge commit باز می‌شود
```

اگر merge commit دو والد داشته باشد، `HEAD~1` به والد اول می‌رود. برای باز کردن merge، باید `-m` بدهی یا از `reset --soft <parent>` استفاده کنی.

### نکات مهم (Tips & Tricks)

- **قبل از هر عملیات خطرناک:** `git branch backup-$(date +%s)`. رایگان است و آرامش می‌دهد.
- **`git reset --soft HEAD~1`** = undo آخرین commit با نگه داشتن تغییرات. alias کن: `git undo`.
- **`git restore -p`** تکه‌تکه دور می‌ریزد — دقیق‌تر از `restore .`.
- **`git status` بعد از `reset --mixed`** نشان می‌دهد چه چیزی unstage شد.
- **`git diff HEAD`** قبل از `restore .` بزن — ببین چه چیزی را دور می‌ریزی.
- **`--keep` را به‌جای `--hard` امتحان کن** وقتی branch را جابه‌جا می‌کنی.

### خلاصه فصل

- `--soft` فقط HEAD، `--mixed` + index، `--hard` + working dir.
- **`--hard` تغییرات uncommitted را نابود می‌کند** — commitها را می‌شود برگرداند، فایل‌ها را نه.
- `restore` جانشین مدرن `checkout --` است.
- `amend` یک commit جدید می‌سازد، هش را عوض می‌کند.
- `reset --soft` + commit = squash سریع.
- برای تاریخ مشترک از `revert` استفاده کن، نه `reset`.

**سؤالات مروری:**
1. تفاوت `--soft`، `--mixed` و `--hard` در چیست؟
2. چطور سه commit آخر را به یکی تبدیل کنی؟
3. چرا `reset --hard` روی تغییرات uncommitted قابل بازیابی نیست؟

---

## فصل ۲۱: revert — بازگردانی امن در تاریخ مشترک

### مقدمه

`revert` یک commit **جدید** می‌سازد که تغییرات commit قبلی را خنثی می‌کند. تاریخ دست‌نخورده می‌ماند — فقط یک commit اضافه می‌شود که می‌گوید «این تغییر را برمی‌گردانم».

**چرا مهم است؟** وقتی commit روی `main` push شده و دیگران آن را دارند، `reset` فاجعه است. `revert` تنها راه امن است. در production، revert همان دکمه‌ی «undo» است که با آن می‌شود سریع یک تغییر بد را برگرداند.

**کاربرد عملی:** deploy کردی، همه‌چیز خراب شد. `git revert <hash>` + push + deploy مجدد. پنج دقیقه، بدون panic.

### مفاهیم اساسی

#### revert چه می‌کند

```
قبل:  A ← B ← C ← D          (main)
              ↑ باگ اینجاست

git revert C

بعد:  A ← B ← C ← D ← C'     (main)
                        ↑ معکوس C
```

`C'` یک commit جدید است که diff آن، **معکوس** diff `C` است. اگر `C` یک خط اضافه کرده، `C'` همان خط را حذف می‌کند.

#### revert در برابر reset

| | revert | reset |
|---|---|---|
| تاریخ | دست‌نخورده + commit جدید | بازنویسی |
| برای تاریخ مشترک | ✅ | ❌ |
| اثری روی دیگران | ندارد | مخرب |
| قابل revert کردن | ✅ (revert را revert کن) | ❌ |
| audit trail | ✅ (می‌بینی چه چیزی برگشت) | ❌ |

#### revert merge commit

merge commit دو والد دارد. Git نمی‌داند کدام طرف را «خنثی» کند:

```bash
git revert -m 1 <merge-hash>
```

`-m 1` یعنی «والد اول را به‌عنوان خط اصلی در نظر بگیر» (معمولاً `main`). `-m 2` والد دوم.

### مثال‌های عملی

#### مثال ۱: revert یک commit

```bash
git log --oneline -5
# e4f5g6h feat: افزودن کش
# d3e4f5g fix: رفع typo
# c2d3e4f feat: صفحه‌ی پرداخت
# ...

# کش مشکل داشت
git revert e4f5g6h
# ادیتور باز می‌شود با پیام:
# Revert "feat: افزودن کش"
# This reverts commit e4f5g6h...

git push
```

#### مثال ۲: revert بدون باز شدن ادیتور

```bash
git revert --no-edit e4f5g6h
```

#### مثال ۳: revert چند commit

```bash
# revert کردن یک بازه (از جدید به قدیم!)
git revert --no-commit e4f5g6h..c2d3e4f
git commit -m "revert: بازگردانی تغییرات پرداخت و کش"

# یا تک‌تک
git revert --no-commit e4f5g6h d3e4f5g
git commit -m "revert: بازگردانی دو commit"
```

**نکته:** `git revert A..B` commitهای **بعد از A تا B** را برمی‌گرداند، به ترتیب معکوس. اگر ترتیب اشتباه باشد، conflict می‌دهد.

#### مثال ۴: revert یک merge commit (سناریوی واقعی)

```bash
# PR #42 merge شد، ولی مشکل دارد
git log --oneline --merges -3
# m1n2o3p Merge pull request #42 from feat/payment

git revert -m 1 m1n2o3p
git push
```

**تله‌ی معروف:** حالا اگر بخواهی `feat/payment` را دوباره merge کنی (بعد از fix)، Git می‌گوید «همه‌ی این commitها قبلاً merge شده‌اند» و هیچ چیزی اعمال نمی‌شود.

راه‌حل: اول revert را revert کن:

```bash
git revert <revert-hash>     # "revert of revert"
git merge feat/payment       # حالا کار می‌کند
```

این یکی از گیج‌کننده‌ترین رفتارهای Git است و در مصاحبه‌ها هم می‌پرسند.

#### مثال ۵: revert در production (runbook)

```bash
# ۱) کدام نسخه deploy شده؟
git log --oneline -1 origin/main

# ۲) revert کن
git switch main && git pull
git revert --no-edit HEAD
git push

# ۳) CI خودکار deploy می‌کند
# ۴) بعداً با آرامش fix کن
```

**توصیه:** این را در `RUNBOOK.md` مکتوب کن. در لحظه‌ی بحران، کسی نباید فکر کند.

#### مثال ۶: revert یک فایل خاص از یک commit

`revert` کل commit را برمی‌گرداند. اگر فقط یک فایل را می‌خواهی:

```bash
git restore --source=<hash>^ -- path/to/file
git commit -m "revert: بازگردانی فایل X به نسخه‌ی قبل"
```

### مقایسه و الگوهای مختلف

#### revert در برابر reset در برابر restore

| سناریو | دستور درست |
|---|---|
| commit push شده، می‌خواهم خنثی کنم | `revert` |
| commit push نشده، می‌خواهم حذف کنم | `reset` |
| فایل را خراب کردم، commit نشده | `restore` |
| می‌خواهم فایل را به نسخه‌ی commit قدیمی برگردانم | `restore --source` |
| merge را می‌خواهم برگردانم | `revert -m 1` |

#### revert در برابر hotfix

| | revert | hotfix |
|---|---|---|
| سرعت | فوری | نیاز به تحلیل و تست |
| ریسک | کم (تغییر را کامل حذف می‌کند) | متوسط |
| مناسب | production خراب است | باگ مشخص است |
| اثر | ویژگی از دست می‌رود | ویژگی می‌ماند |

**روش استاندارد:** revert فوری برای توقف خون‌ریزی، بعد hotfix با آرامش.

### اشکال شایع

#### ۱. `git revert` conflict می‌دهد

طبیعی است: commitهای بعدی همان خطوط را عوض کرده‌اند. حل کن، `git add`، `git revert --continue`.

```bash
git revert --abort    # اگر می‌خواهی لغو کنی
```

#### ۲. revert merge و merge مجدد کار نمی‌کند

همان تله‌ی `-m 1`. راه‌حل: revert را revert کن.

#### ۳. revert یک commit که خیلی قدیمی است

اگر ۵۰ commit بعدش به همان کد وابسته شده‌اند، conflict بزرگ می‌شود. بهتر: `cherry-pick` معکوس روی فایل، یا دستی fix کن.

#### ۴. revert و از دست رفتن تغییرات دیگران

اگر commitی که revert می‌کنی شامل کار دیگران هم بوده (مثلاً در یک squash)، کار آن‌ها هم برمی‌گردد. **دلیل دیگری برای commitهای اتمیک.**

#### ۵. `revert` روی branch اشتباه

revert فقط روی branch فعلی اثر می‌گذارد. اگر روی `feature` بزنی، `main` دست‌نخورده می‌ماند.

### نکات مهم (Tips & Tricks)

- **`git revert --no-commit <hash>`** برای revert چندتایی و ساخت یک commit.
- **در production، revert اولین کار است، نه تحلیل.** اول خون‌ریزی را بند بیاور.
- **پیام revert را عوض نکن** مگر برای اضافه کردن دلیل. خودکار `This reverts commit ...` را نگه دار (ابزارها تشخیص می‌دهند).
- **`git revert HEAD`** = خنثی کردن آخرین commit. سریع‌ترین undo برای تاریخ مشترک.
- **در `RUNBOOK` بنویس** که revert merge نیاز به `-m 1` دارد.

### خلاصه فصل

- `revert` یک commit جدید می‌سازد که تغییرات قبلی را خنثی می‌کند.
- برای تاریخ مشترک، **همیشه revert، نه reset**.
- revert merge commit نیاز به `-m 1` دارد.
- revert یک merge، merge مجدد را می‌شکند تا وقتی revert را revert کنی.
- revert سریع‌ترین راه توقف خرابی در production است.

**سؤالات مروری:**
1. چرا revert برای تاریخ مشترک امن است ولی reset نه؟
2. `-m 1` در revert merge چه معنایی دارد؟
3. چرا بعد از revert یک merge نمی‌توانی همان branch را دوباره merge کنی؟

---

## فصل ۲۲: reflog — شبکه‌ی نجات

### مقدمه

`git reflog` تاریخچه‌ی **حرکت HEAD** است. هر بار که commit می‌زنی، branch عوض می‌کنی، reset می‌کنی، rebase می‌کنی، merge می‌کنی — Git ثبت می‌کند. این ثبت محلی است و به‌طور پیش‌فرض ۹۰ روز نگه داشته می‌شود.

**چرا مهم است؟** تقریباً هر «کارم گم شد» با reflog حل می‌شود. این تنها ابزاری است که Git را از «ترسناک» به «قابل اعتماد» تبدیل می‌کند.

**کاربرد عملی:** `reset --hard` زدی، branch را حذف کردی، rebase خراب شد، commit گم شد — همه با reflog برمی‌گردند.

### مفاهیم اساسی

#### reflog چه چیزی ثبت می‌کند

```
a1b2c3d HEAD@{0}: reset: moving to HEAD~2
e4f5g6h HEAD@{1}: commit: feat: بخش دوم
c3d4e5f HEAD@{2}: commit: feat: بخش اول
b2c3d4e HEAD@{3}: checkout: moving from main to feature
a1b2c3d HEAD@{4}: commit (initial): init
```

هر خط: هش، موقعیت قبلی HEAD، عملیات، توضیح.

#### انواع reflog

| reflog | چه چیزی را ثبت می‌کند |
|---|---|
| `HEAD` | حرکت HEAD (شامل checkout) |
| `refs/heads/main` | حرکت branch `main` |
| `refs/remotes/origin/main` | حرکت remote-tracking |

```bash
git reflog                  # HEAD
git reflog show main        # branch main
git reflog show origin/main
```

#### چرا reflog محلی است

reflog در `.git/logs/` است و **push نمی‌شود**. یعنی:
- روی سرور، reflog متفاوتی هست (GitHub نگه می‌دارد ولی در دسترس عمومی نیست).
- اگر مخزن را دوباره clone کنی، reflog خالی است.

#### مدت نگهداری

پیش‌فرض:
- ورودی‌های قابل دسترس: ۹۰ روز
- ورودی‌های غیرقابل دسترس (commitهای orphan): ۳۰ روز

```bash
git config gc.reflogExpire "180 days"
git config gc.reflogExpireUnreachable "60 days"
```

بعد از این مدت، `git gc` آن‌ها را پاک می‌کند و commit واقعاً می‌رود.

### مثال‌های عملی

#### مثال ۱: بازیابی بعد از `reset --hard`

```bash
git log --oneline -3
# a1b2c3d feat: کار مهم
# b2c3d4e docs: قبلی

git reset --hard HEAD~1     # اوه!

git reflog
# b2c3d4e HEAD@{0}: reset: moving to HEAD~1
# a1b2c3d HEAD@{1}: commit: feat: کار مهم

git reset --hard a1b2c3d    # برگشتی
```

#### مثال ۲: بازیابی branch حذف‌شده

```bash
git branch -D feature/important
# Deleted branch feature/important (was e4f5g6h)

git reflog
# e4f5g6h HEAD@{5}: commit: feat: آخرین کار روی branch

git switch -c feature/important e4f5g6h
# branch برگشت با همه‌ی commitها
```

**توجه:** `git branch -D` هش را چاپ می‌کند — همان را یادداشت کن، ساده‌ترین راه.

#### مثال ۳: بازیابی بعد از rebase خراب

```bash
git rebase main
# فاجعه شد، همه‌چیز قاطی

git reflog
# 9i8h7g6 HEAD@{0}: rebase (finish): returning to refs/heads/feature
# 5f4e3d2 HEAD@{1}: rebase (start): checkout main
# e4f5g6h HEAD@{2}: commit: feat: آخرین commit قبل از rebase

git reset --hard e4f5g6h
# برگشتی به قبل از rebase
```

**نکته:** rebase معمولاً `ORIG_HEAD` را هم تنظیم می‌کند:

```bash
git reset --hard ORIG_HEAD
```

سریع‌تر و به‌یادماندنی‌تر.

#### مثال ۴: بازیابی commit در detached HEAD

```bash
git switch --detach HEAD~3
# کار کن
git commit -m "آزمایش"
# اوه، switch کردم به main بدون ساختن branch

git switch main
git reflog
# a1b2c3d HEAD@{1}: commit: آزمایش

git switch -c rescue a1b2c3d
```

#### مثال ۵: پیدا کردن commit که «گم شد»

```bash
# همه‌ی commitهای ۲۴ ساعت گذشته، حتی orphan
git reflog --all --since="24 hours ago"

# یا فقط پیام‌ها
git log -g --oneline
```

`git log -g` = `git log --walk-reflogs` — نمای کامل‌تر با گراف.

#### مثال ۶: بازیابی بعد از merge اشتباه

```bash
git merge wrong-branch
# merge کردم، push هم نکردم

git reset --hard ORIG_HEAD
# برگشت
```

#### مثال ۷: چه کسی branch را جابه‌جا کرد (reflog یک branch)

```bash
git reflog show main
# a1b2c3d main@{0}: commit: feat: X
# b2c3d4e main@{1}: pull: Fast-forward
# c3d4e5f main@{2}: reset: moving to origin/main
```

### مقایسه و الگوهای مختلف

#### reflog در برابر log

| | `git log` | `git reflog` |
|---|---|---|
| چه چیزی | تاریخ commitهای قابل دسترس | تاریخ حرکت HEAD |
| commitهای orphan | ❌ | ✅ |
| branchهای حذف‌شده | ❌ | ✅ |
| روی سرور | ✅ | ❌ (محلی) |
| ترتیب | گراف | زمانی |

#### ابزارهای بازیابی

| ابزار | برای |
|---|---|
| `git reflog` | commitهای گم‌شده، reset، rebase |
| `git fsck --lost-found` | اشیاء orphan (وقتی reflog هم نیست) |
| `git stash list` | تغییرات stash‌شده |
| `git branch -a --contains <hash>` | کدام branchها این commit را دارند |

```bash
# آخرین راه: پیدا کردن همه‌ی commitهای orphan
git fsck --lost-found
# dangling commit a1b2c3d...
git show a1b2c3d
```

### اشکال شایع

#### ۱. reflog خالی است

- مخزن تازه clone شده (reflog سرور منتقل نمی‌شود).
- `git gc` اجرا شده و ورودی‌ها پاک شده‌اند.
- در پوشه‌ی اشتباهی هستی.

#### ۲. `git reset --hard` روی تغییرات uncommitted

reflog کمکی نمی‌کند. چیزی commit نشده بود.

#### ۳. reflog commit را نشان می‌دهد ولی `git show` خطا می‌دهد

`git gc` آن را پاک کرده. گاهی `git fsck` پیدا می‌کند، اغلب نه.

#### ۴. فکر کردن reflog روی سرور هم هست

نیست. اگر لپ‌تاپت بسوزد و مخزن را از GitHub clone کنی، reflog خالی است. (commitهای push‌شده سالم‌اند، ولی commitهای محلیِ push‌نشده رفته‌اند.)

#### ۵. `ORIG_HEAD` تنظیم نشده

`ORIG_HEAD` فقط بعد از merge، rebase، reset تنظیم می‌شود. بعد از commit معمولی، نه.

### نکات مهم (Tips & Tricks)

- **`git reflog` اولین دستوری است که در «کارم گم شد» می‌زنی.** به‌خاطر بسپار.
- **`git reset --hard ORIG_HEAD`** برای لغو merge/rebase.
- **`git branch -D` هش را چاپ می‌کند** — همان را نگه دار.
- **`git reflog --date=iso`** زمان دقیق‌تری نشان می‌دهد.
- **`git config --global alias.rescue "reflog --date=iso"`** برای دسترسی سریع.
- **reflog را در مخزن‌های حساس طولانی‌تر کن** (`gc.reflogExpire`).
- **reflog جای بکاپ نیست.** برای کارهای بزرگ، branch پشتیبان بساز.

### خلاصه فصل

- `git reflog` تاریخ حرکت HEAD را ثبت می‌کند و **محلی** است.
- تقریباً هر commit گم‌شده با reflog برمی‌گردد.
- پیش‌فرض ۹۰ روز نگه‌داری می‌شود (۳۰ روز برای orphanها).
- `ORIG_HEAD` میان‌بر لغو merge/rebase است.
- `git fsck --lost-found` آخرین راه برای اشیاء orphan.

**سؤالات مروری:**
1. چرا reflog روی سرور در دسترس نیست؟
2. چطور بعد از `reset --hard` اشتباه برگردی؟
3. تفاوت `git log` و `git reflog` چیست؟

---

## فصل ۲۳: cherry-pick و stash — انتقال کار

### مقدمه

دو دستور که هر کدام یک مسئله‌ی روزمره را حل می‌کنند:

- **cherry-pick:** «این commit را از آن branch بردار و روی این branch بگذار.»
- **stash:** «کارم نیمه‌تمام است، فعلاً کنارش بگذار تا کار فوری‌تری بکنم.»

**چرا مهم است؟** cherry-pick برای hotfix در production حیاتی است: باگ در `main` رفع شد، باید همان fix در `release/1.4` هم باشد. stash برای جابه‌جایی سریع بین کارها بدون commit کردن نیمه‌کاره.

**کاربرد عملی:** هر روز از stash استفاده می‌کنی؛ از cherry-pick چند بار در ماه.

### مفاهیم اساسی

#### cherry-pick چه می‌کند

```
main:      A ← B ← C ← D
                      ↑ fix مهم

release:   A ← B ← X

git switch release
git cherry-pick D

release:   A ← B ← X ← D'
```

`D'` یک commit **جدید** است با همان تغییرات `D` ولی هش و والد متفاوت. یعنی **کپی**، نه انتقال.

#### cherry-pick در برابر merge

| | cherry-pick | merge |
|---|---|---|
| دامنه | یک یا چند commit | کل branch |
| تاریخ | کپی با هش جدید | حفظ کامل |
| مناسب | hotfix، انتقال انتخابی | ادغام کامل |
| خطر | commit تکراری در دو branch | conflict بزرگ |

**تله‌ی کلاسیک:** اگر cherry-pick کنی و بعداً branchها را merge کنی، Git می‌تواند همان تغییر را دو بار اعمال کند (چون هش‌ها متفاوتند و Git نمی‌فهمد یکی هستند). معمولاً Git با تشخیص محتوای یکسان جلوی این را می‌گیرد، ولی گاهی conflict مصنوعی می‌دهد.

#### stash چه می‌کند

`git stash` تغییرات uncommitted (working dir + staging) را در یک «قفسه» ذخیره می‌کند و working directory را تمیز می‌کند.

```
قبل:  working dir (تغییرات) + staging (تغییرات)
stash → working dir تمیز + یک ورودی در stash list
pop  → تغییرات برمی‌گردند
```

stash یک **commit** است (با دو یا سه والد) که در `refs/stash` ذخیره می‌شود.

### مثال‌های عملی

#### مثال ۱: cherry-pick یک fix به release

```bash
# باگ در main رفع شد
git log --oneline main -3
# d4e5f6g fix(auth): رفع خطای توکن منقضی
# c3d4e5f feat: ...

# باید در release/1.4 هم باشد
git switch release/1.4
git cherry-pick d4e5f6g

# اگر conflict:
# حل کن → git add → git cherry-pick --continue
# یا: git cherry-pick --abort
# یا: git cherry-pick --skip
```

#### مثال ۲: cherry-pick چند commit

```bash
# یک بازه
git cherry-pick c3d4e5f..d4e5f6g

# چندتای مشخص
git cherry-pick c3d4e5f d4e5f6g e5f6g7h

# بدون commit (برای ادغام دستی)
git cherry-pick -n c3d4e5f d4e5f6g
git commit -m "fix: انتقال fixهای auth به release"
```

#### مثال ۳: cherry-pick با حفظ نویسنده

به‌طور پیش‌فرض، cherry-pick نویسنده‌ی اصلی را حفظ می‌کند و committer تو می‌شوی. اگر می‌خواهی خودت نویسنده باشی:

```bash
git cherry-pick --no-commit d4e5f6g
git commit --author="علی رضایی <ali@example.com>" -m "..."
```

#### مثال ۴: stash پایه

```bash
# وسط کار، یک باگ فوری آمد
git stash
# Saved working directory and index state WIP on feature: a1b2c3d ...

git switch main
git switch -c hotfix/urgent
# fix کن، commit، push

git switch feature
git stash pop
# تغییرات برگشتند
```

#### مثال ۵: stash با پیام

```bash
git stash push -m "کار نیمه‌تمام روی فرم پرداخت"
git stash list
# stash@{0}: On feature: کار نیمه‌تمام روی فرم پرداخت
```

#### مثال ۶: stash شامل فایل‌های جدید

```bash
git stash -u        # شامل untracked
git stash -a        # شامل untracked + ignored (احتیاط!)
```

بدون `-u`، فایل‌های جدید stash نمی‌شوند و در working dir می‌مانند — که ممکن است باعث conflict شود.

#### مثال ۷: مدیریت stash list

```bash
git stash list
# stash@{0}: On feature: کار دوم
# stash@{1}: On feature: کار اول

git stash show -p stash@{1}    # محتوای کامل
git stash apply stash@{1}      # اعمال بدون حذف
git stash pop stash@{1}        # اعمال + حذف
git stash drop stash@{1}       # حذف
git stash clear                # همه را حذف کن (خطرناک)
```

**تفاوت `apply` و `pop`:** `pop` = `apply` + `drop`. `apply` امن‌تر است چون اگر conflict بدهد، stash می‌ماند.

#### مثال ۸: stash بخشی از تغییرات

```bash
git stash push -p -m "فقط بخش auth"
# تکه‌تکه می‌پرسد
```

#### مثال ۹: تبدیل stash به branch

```bash
git stash branch feature/from-stash stash@{0}
# branch جدید از commitی که stash رویش بود می‌سازد،
# stash را apply و drop می‌کند
```

این بهترین راه حل conflict در stash است.

### مقایسه و الگوهای مختلف

#### stash در برابر commit موقت

| | stash | `commit -m wip` |
|---|---|---|
| در تاریخ | ❌ (جدا) | ✅ |
| فراموش‌شدنی | ✅ (خیلی!) | ❌ |
| اشتراک با دیگران | ❌ | ✅ (با push) |
| مناسب | چند دقیقه | چند ساعت/روز |

**توصیه:** برای وقفه‌های کوتاه stash، برای کارهای بلندتر commit روی branch موقت.

#### cherry-pick در برابر merge در برابر rebase

| سناریو | دستور |
|---|---|
| یک fix به release | `cherry-pick` |
| کل branch به main | `merge` یا `rebase` |
| چند commit انتخابی | `cherry-pick` |
| انتقال branch به base جدید | `rebase --onto` |

#### گزینه‌های cherry-pick

| گزینه | کار |
|---|---|
| `-n` / `--no-commit` | اعمال بدون commit |
| `-x` | اضافه کردن `(cherry picked from commit ...)` به پیام |
| `-e` | ویرایش پیام |
| `--signoff` | افزودن `Signed-off-by` |
| `-m 1` | برای merge commit |

`-x` را در پروژه‌های متن‌باز استفاده کن — ردیابی منشأ مهم است.

### اشکال شایع

#### ۱. commit تکراری بعد از cherry-pick و merge

cherry-pick یک کپی می‌سازد. اگر بعداً branch را merge کنی، ممکن است دو commit با محتوای یکسان داشته باشی. Git معمولاً تشخیص می‌دهد، ولی گاهی نه.

**راه‌حل:** اگر قرار است branch را merge کنی، cherry-pick نکن.

#### ۲. stash را گم کردم

```bash
git fsck --unreachable | grep commit
# یا
git log -g refs/stash
```

stashها commit هستند، پس با reflog/fsck پیدا می‌شوند.

#### ۳. `git stash pop` conflict داد

stash **حذف نمی‌شود** اگر conflict بدهد (رفتار درست). حل کن، `git add`، بعد `git stash drop`.

#### ۴. stash در branch اشتباه

stash به branch وابسته نیست — می‌توانی روی هر branch اعمالش کنی. این هم مزیت است هم خطر (ممکن است در جای اشتباه اعمال شود).

#### ۵. `git stash` چیزی ذخیره نکرد

اگر working dir تمیز باشد، stash چیزی ندارد. یا همه‌چیز commit شده.

#### ۶. cherry-pick از branch که merge شده

اگر commit قبلاً در branch مقصد هست (از طریق merge)، cherry-pick آن «empty» می‌شود:

```
The previous cherry-pick is now empty, possibly due to conflict resolution.
```

یعنی تغییرات از قبل وجود دارند. `git cherry-pick --skip`.

### نکات مهم (Tips & Tricks)

- **`git stash -u`** را عادت کن — فایل‌های جدید را هم می‌گیرد.
- **`git stash push -m "..."`** همیشه پیام بده. `stash@{3}` بی‌معنی است.
- **stash را فراموش نکن.** ماهی یک بار `git stash list` بزن و پاک کن.
- **`git cherry-pick -x`** در متن‌باز، برای ردیابی.
- **`git stash branch`** بهترین راه حل conflict.
- **برای hotfix، `cherry-pick` از `main` به release** الگوی استاندارد است.
- **بعد از cherry-pick، تست را اجرا کن** — ممکن است کد به چیزهایی وابسته باشد که در branch مقصد نیستند.

### خلاصه فصل

- `cherry-pick` یک commit را **کپی** می‌کند (هش جدید).
- برای hotfix به release branch عالی است.
- `stash` تغییرات uncommitted را کنار می‌گذارد؛ `pop` برمی‌گرداند.
- `stash -u` فایل‌های جدید را هم شامل می‌شود.
- `apply` امن‌تر از `pop` است.
- stash به branch وابسته نیست — هم مزیت، هم خطر.

**سؤالات مروری:**
1. چرا cherry-pick commit تکراری می‌سازد و چه زمانی مشکل می‌شود؟
2. تفاوت `stash apply` و `stash pop` چیست؟
3. چطور یک hotfix را از `main` به `release/1.4` منتقل کنی؟

---

## فصل ۲۴: bisect — پیدا کردن commit مقصر

### مقدمه

یک باگ هست، نمی‌دانی کِی آمد. ۵۰۰ commit از آخرین نسخه‌ی سالم گذشته. خواندن تکی همه غیرممکن است.

`git bisect` یک **جست‌وجوی دودویی** است: با ~۹ مرحله (log₂ 500 ≈ ۹) commit مقصر را پیدا می‌کند. این یکی از زیباترین ابزارهای Git است.

**چرا مهم است؟** دیباگ معمولاً «کدام کد باگ دارد» است. bisect به «کدام commit باگ را آورد» جواب می‌دهد — که سؤال متفاوت و اغلب مفیدتری است.

**کاربرد عملی:** کاربر می‌گوید «از نسخه‌ی ۲.۳ این‌طور شد». تو بین v2.2 و v2.3 ده مرحله bisect می‌زنی و در ۵ دقیقه commit مقصر را پیدا می‌کنی.

### مفاهیم اساسی

#### جست‌وجوی دودویی

```
۵۱۲ commit → ۹ مرحله
۱۰۲۴ commit → ۱۰ مرحله
۱ میلیون commit → ۲۰ مرحله
```

هر مرحله: نصف باقی‌مانده حذف می‌شود.

#### چطور کار می‌کند

```bash
git bisect start
git bisect bad                  # نسخه‌ی فعلی خراب است
git bisect good v2.2.0          # این نسخه سالم بود

# Git یک commit وسط را checkout می‌کند
# تست کن: سالم است یا خراب؟
git bisect good                 # یا git bisect bad

# تکرار تا Git بگوید:
# <hash> is the first bad commit
```

#### حالت خودکار

اگر تست خودکار داری، می‌توانی کاملاً خودکار کنی:

```bash
git bisect start HEAD v2.2.0
git bisect run npm test
```

Git خودش تست را در هر مرحله اجرا می‌کند و بر اساس exit code تصمیم می‌گیرد (۰ = good، غیر ۰ = bad).

### مثال‌های عملی

#### مثال ۱: bisect دستی

```bash
cd ~/github-lab/shop
git bisect start
git bisect bad                    # HEAD خراب
git bisect good a1b2c3d           # این commit سالم بود

# Bisecting: 7 revisions left to test after this (roughly 3 steps)
# [e4f5g6h] feat: افزودن کش

# تست کن
npm test
# شکست خورد
git bisect bad

# Git بعدی را checkout می‌کند
# [b2c3d4e] refactor: تغییر ساختار

npm test
# پاس شد
git bisect good

# ... تکرار ...

# در پایان:
# b2c3d4e is the first bad commit
# commit b2c3d4e
# Author: ...
# Date: ...
#     refactor: تغییر ساختار

git bisect reset                  # برگرد به branch اصلی
```

#### مثال ۲: bisect خودکار (بهترین روش)

```bash
git bisect start HEAD v2.2.0
git bisect run npm test
# Git همه‌چیز را خودکار انجام می‌دهد
```

اسکریپت تست باید:
- موفق: exit code ۰
- شکست: exit code غیر ۰
- **skip:** exit code ۱۲۵ (Git این commit را نادیده می‌گیرد)

#### مثال ۳: اسکریپت تست سفارشی

`test-bug.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

# نصب وابستگی‌ها اگر لازم است
npm ci --silent

# تست مشخصی که باگ را می‌گیرد
if npm run test:e2e -- --grep "checkout flow"; then
  exit 0        # سالم
else
  exit 1        # خراب
fi
```

```bash
chmod +x test-bug.sh
git bisect start HEAD v2.2.0
git bisect run ./test-bug.sh
```

#### مثال ۴: skip کردن commitهای غیرقابل تست

گاهی یک commit کامپایل نمی‌شود (مثلاً وابستگی اضافه شده ولی نصب نشده):

```bash
git bisect skip
# یا خودکار: exit 125
```

Git با skip هوشمندانه‌تر جست‌وجو می‌کند.

#### مثال ۵: bisect روی یک فایل

```bash
git bisect start
git bisect bad
git bisect good v2.2.0
git bisect run sh -c 'grep -q "TODO" src/app.js && exit 1 || exit 0'
```

#### مثال ۶: دیدن تاریخچه‌ی bisect

```bash
git bisect log > bisect-log.txt
# بعداً می‌توانی بازتولید کنی:
git bisect replay bisect-log.txt
```

### مقایسه و الگوهای مختلف

#### bisect در برابر جست‌وجوی خطی

| | bisect | خطی |
|---|---|---|
| ۵۰۰ commit | ~۹ مرحله | تا ۵۰۰ |
| نیاز به تست خودکار | مفید | نه |
| دقت | commit اول خراب | همان |
| زمان (با تست ۱ دقیقه‌ای) | ۹ دقیقه | ۸ ساعت |

#### ابزارهای جایگزین

| ابزار | برای |
|---|---|
| `git bisect` | پیدا کردن commit |
| `git blame` | پیدا کردن نویسنده‌ی خط |
| `git log -S` | پیدا کردن commit که رشته را آورد |
| `git log --diff-filter` | پیدا کردن حذف/افزودن فایل |

#### حالت‌های bisect

| دستور | کار |
|---|---|
| `git bisect start [bad] [good]` | شروع با مشخص کردن دو سر |
| `git bisect good/bad/skip` | علامت‌گذاری |
| `git bisect run <cmd>` | خودکار |
| `git bisect reset` | پایان و بازگشت |
| `git bisect log` | ثبت تاریخچه |
| `git bisect replay <file>` | بازتولید |

### اشکال شایع

#### ۱. `git bisect reset` نزدم و در detached HEAD گیر کردم

```bash
git bisect reset
# یا اگر فراموش کردی که در bisect هستی:
git switch main
```

#### ۲. تست خودکار قابل اعتماد نیست (flaky)

اگر تست گاهی پاس و گاهی شکست می‌خورد، bisect نتیجه‌ی غلط می‌دهد. راه‌حل: تست را ۳ بار اجرا کن و اگر هر سه پاس شد، good.

```bash
#!/usr/bin/env bash
for i in 1 2 3; do
  npm test || exit 1
done
exit 0
```

#### ۳. commitها کامپایل نمی‌شوند

`git bisect skip` یا exit 125.

#### ۴. bisect روی مخزن بزرگ کند است

هر مرحله checkout + build + test. اگر build سنگین است، ۲۰ دقیقه‌ای می‌شود. راه‌حل: کش build، یا تست سبک‌تر مخصوص bisect.

#### ۵. `good` و `bad` را برعکس دادم

Git معمولاً خطا می‌دهد: «good revision is not an ancestor of bad revision». اگر نه، ممکن است نتیجه‌ی بی‌معنی بدهد. `git bisect reset` و از نو.

#### ۶. بعد از bisect، تغییراتم رفت

bisect checkout می‌کند، پس تغییرات uncommitted ممکن است مانع شوند. قبل از شروع: `git stash`.

### نکات مهم (Tips & Tricks)

- **`git bisect run` را یاد بگیر** — تفاوت بین ۱۰ دقیقه و ۱۰ ثانیه.
- **exit 125 برای skip** — کلید bisect خودکار روی commitهای شکسته.
- **`git bisect log` را ذخیره کن** — برای مستندسازی و بازتولید.
- **تست را مخصوص باگ بنویس** — هرچه دقیق‌تر، bisect دقیق‌تر.
- **قبل از bisect، `git stash` بزن** — working dir باید تمیز باشد.
- **در CI هم می‌شود bisect کرد** — برای پیدا کردن commit که flaky test را آورد.

### خلاصه فصل

- `git bisect` جست‌وجوی دودویی روی تاریخ است: log₂(n) مرحله.
- `start` → `bad` → `good` → تکرار → `reset`.
- `git bisect run <cmd>` کاملاً خودکار است.
- exit code ۱۲۵ = skip.
- قبل از شروع، working directory را تمیز کن.

**سؤالات مروری:**
1. چرا bisect با ۱۰۰۰ commit فقط ~۱۰ مرحله لازم دارد؟
2. exit code ۱۲۵ در bisect چه معنایی دارد؟
3. چطور یک تست flaky می‌تواند bisect را خراب کند و چه کنی؟

---

**بعدی:** [`06-Repository-Setup-and-Files.md`](06-Repository-Setup-and-Files.md)

---

## منابع و مراجع

- Pro Git، فصل ۷ (ابزارها: reset، reflog، bisect) — https://git-scm.com/book/fa/v2
- مستندات `git reset` — https://git-scm.com/docs/git-reset
- مستندات `git revert` — https://git-scm.com/docs/git-revert
- مستندات `git reflog` — https://git-scm.com/docs/git-reflog
- مستندات `git bisect` — https://git-scm.com/docs/git-bisect
- مستندات `git stash` — https://git-scm.com/docs/git-stash
- "Git reset demystified" — https://git-scm.com/blog/2011/07/11/reset.html
