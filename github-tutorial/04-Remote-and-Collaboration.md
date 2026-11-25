# بخش ۰۴: remote، همکاری و اولین Pull Request

**مدت یادگیری**: ۴ تا ۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۰۳ (branch، merge)

---

## فهرست محتوا

- [فصل ۱۵: remote — origin، upstream و fetch در برابر pull](#فصل-۱۵)
- [فصل ۱۶: fork و مدل مشارکت متن‌باز](#فصل-۱۶)
- [فصل ۱۷: push، force-with-lease و محافظت از تاریخ مشترک](#فصل-۱۷)
- [فصل ۱۸: اولین Pull Request — از branch تا merge](#فصل-۱۸)
- [فصل ۱۹: همگام‌سازی fork و کار تیمی روی یک مخزن](#فصل-۱۹)

---

## فصل ۱۵: remote — origin، upstream و fetch در برابر pull

### مقدمه

تا اینجا همه‌چیز محلی بود. حالا می‌خواهیم کد را با دیگران به اشتراک بگذاریم. کل مفهوم «remote» همین است: یک نام مستعار برای آدرسی که می‌توانی از آن بخوانی و به آن بنویسی.

**چرا مهم است؟** ۹۰٪ سردرگمی‌های تیمی از سوءتفاهم روی `fetch` و `pull` می‌آید. «pull زدم ولی به‌روز نشد»، «origin/main چیست؟»، «چرا push رد می‌شود؟» — همه ریشه در همین فصل دارند.

**کاربرد عملی:** بعد از این فصل می‌دانی دقیقاً چه زمانی Git با سرور حرف می‌زند و چه زمانی نه.

### مفاهیم اساسی

#### remote چیست

```bash
git remote -v
# origin  git@github.com:ali/shop.git (fetch)
# origin  git@github.com:ali/shop.git (push)
```

یک remote سه بخش دارد:
1. **نام** (`origin`)
2. **URL برای fetch**
3. **URL برای push** (معمولاً یکی)

`origin` یک قرارداد است، نه یک ویژگی. اولین remote که clone می‌سازد `origin` نام می‌گیرد.

#### چهار مکان، یک تصویر

```
┌──────────────┐   fetch    ┌──────────────────┐   push   ┌──────────┐
│    سرور      │ ────────→  │  origin/main     │ ←──────  │          │
│  (GitHub)    │            │  (remote-tracking)│          │          │
└──────────────┘            └────────┬─────────┘          │          │
                                     │ merge/rebase        │          │
                                     ↓                     │          │
                            ┌──────────────────┐           │          │
                            │   main (محلی)     │ ──────────┘          │
                            └────────┬─────────┘                      │
                                     ↓ checkout                      │
                            ┌──────────────────┐                      │
                            │ Working Directory │                     │
                            └──────────────────┘                     │
```

**نکته‌ی کلیدی:** `origin/main` یک branch محلی است که فقط با `fetch` به‌روز می‌شود. تا وقتی fetch نزنی، Git هیچ خبری از سرور ندارد.

#### `fetch` در برابر `pull`

| | `git fetch` | `git pull` |
|---|---|---|
| چه می‌کند | فقط `origin/main` را به‌روز می‌کند | `fetch` + `merge` (یا rebase) |
| فایل‌های کاری | دست نمی‌زند | عوض می‌شوند |
| conflict | ندارد | ممکن است داشته باشد |
| امنیت | کاملاً امن | ممکن است وسط کار چیزی را قاطی کند |
| توصیه | قبل از هر کاری | وقتی مطمئنی |

**`git pull` = `git fetch` + `git merge origin/main`** (یا rebase اگر تنظیم کرده باشی).

#### `push`

```bash
git push                        # با upstream
git push -u origin feature      # بار اول، تنظیم upstream
git push origin main            # صریح
git push --tags                 # تگ‌ها (به‌طور پیش‌فرض push نمی‌شوند!)
git push --all                  # همه‌ی branchها
git push origin --delete old    # حذف branch روی سرور
```

**تله‌ی مهم:** `git push` تگ‌ها را نمی‌فرستد. باید `--tags` یا `--follow-tags` بزنی.

#### upstream — ردیابی

```bash
git branch -vv
# * main    a1b2c3d [origin/main] docs: ...
#   feature e4f5g6h [origin/feature: ahead 2, behind 1]
```

`[origin/main]` یعنی upstream تنظیم شده. `ahead 2` یعنی ۲ commit محلی داری که push نشده. `behind 1` یعنی ۱ commit روی سرور هست که نداری.

### مثال‌های عملی

#### مثال ۱: افزودن remote به مخزن موجود

```bash
cd ~/github-lab/shop
gh repo create shop --public --source=. --remote=origin --push

# یا دستی:
git remote add origin git@github.com:ali/shop.git
git push -u origin main
```

#### مثال ۲: remote دوم (upstream برای fork)

```bash
# origin = fork خودت
# upstream = مخزن اصلی
git remote add upstream https://github.com/original/project.git
git remote -v
# origin    git@github.com:ali/project.git
# upstream  https://github.com/original/project.git
```

#### مثال ۳: تغییر URL remote (HTTPS → SSH)

```bash
git remote set-url origin git@github.com:ali/shop.git
git remote -v
```

#### مثال ۴: `fetch` بدون merge — روش حرفه‌ای

```bash
git fetch origin

# ببین چه چیزی جدید است
git log HEAD..origin/main --oneline

# ببین diff
git diff HEAD...origin/main

# حالا تصمیم بگیر
git merge origin/main          # یا
git rebase origin/main         # یا
git reset --hard origin/main   # (خطرناک — همه‌چیز محلی می‌رود)
```

این روش بسیار امن‌تر از `pull` است چون قبل از هر تغییری می‌بینی چه می‌آید.

#### مثال ۵: `pull` با استراتژی مشخص

```bash
git pull --ff-only              # فقط اگر fast-forward ممکن باشد، وگرنه خطا
git pull --rebase               # rebase به‌جای merge
git pull --no-rebase            # merge (پیش‌فرض معمول)
git pull origin main --no-commit  # merge کن ولی commit نزن (برای بازبینی)
```

`--ff-only` یکی از بهترین عادت‌هاست: اگر تاریخ واگرا شده، Git خطا می‌دهد و تو مجبور می‌شوی آگاهانه تصمیم بگیری.

#### مثال ۶: حذف remote

```bash
git remote remove old-origin
# branchهای remote-tracking هم پاک می‌شوند
```

#### مثال ۷: `git remote show` — اطلاعات کامل

```bash
git remote show origin
# * remote origin
#   Fetch URL: git@github.com:ali/shop.git
#   Push  URL: git@github.com:ali/shop.git
#   HEAD branch: main
#   Remote branches:
#     main    tracked
#     feature tracked
#   Local branch configured for 'git pull':
#     main merges with remote main
#   Local ref configured for 'git push':
#     main pushes to main (up to date)
```

### مقایسه و الگوهای مختلف

#### تنظیمات `pull.rebase`

| مقدار | رفتار `git pull` | مناسب برای |
|---|---|---|
| `false` (پیش‌فرض) | merge | تیم‌هایی که merge commit می‌خواهند |
| `true` | rebase | تاریخ خطی |
| `merges` | rebase ولی merge commitهای محلی حفظ می‌شوند | حالت میانه |

```bash
git config --global pull.rebase true
```

#### `fetch.prune`

| | بدون prune | با prune |
|---|---|---|
| branch حذف‌شده روی سرور | `origin/old` هنوز هست | پاک می‌شود |
| `git branch -r` | پر از branch مرده | تمیز |

```bash
git config --global fetch.prune true
# یا یک‌باره:
git fetch --prune
```

### اشکال شایع

#### ۱. `pull` زدم و merge commit ناخواسته ساختم

چون `pull.rebase` روی `false` است و تاریخ واگرا بود. راه‌حل: `git config --global pull.rebase true`، یا از `git pull --rebase` استفاده کن.

#### ۲. `fatal: refusing to merge unrelated histories`

وقتی دو مخزن کاملاً مستقل را می‌خواهی merge کنی (مثلاً GitHub یک README ساخت و تو هم محلی init کردی):

```bash
git pull origin main --allow-unrelated-histories
```

بهتر: از اول مخزن GitHub را بدون README/`.gitignore`/LICENSE بساز و بعد push کن.

#### ۳. `origin/main` قدیمی است

`fetch` نزده‌ای. `git fetch` بزن.

#### ۴. `push` رد می‌شود: `non-fast-forward`

```
! [rejected] main -> main (non-fast-forward)
error: failed to push some refs
hint: Updates were rejected because the tip of your current branch is behind
```

یعنی روی سرور commitهایی هست که تو نداری. راه‌حل:

```bash
git pull --rebase     # یا git pull
git push
```

**هرگز** `git push --force` نزن روی branch مشترک.

#### ۵. `remote: Permission denied`

احراز هویت یا دسترسی. فصل ۴ و بخش «اشکال شایع» را ببین.

#### ۶. remote با نام اشتباه

```bash
git remote rename orgin origin    # تایپو
```

#### ۷. `git push` می‌گوید `Everything up-to-date` ولی من commit دارم

معمولاً روی branch اشتباهی هستی یا upstream اشتباه است:

```bash
git status          # ببین روی چه branchی هستی
git log --oneline -3
git branch -vv
```

### نکات مهم (Tips & Tricks)

- **قبل از هر کار، `git fetch` بزن.** عادت خوبی است و هیچ خطری ندارد.
- **`git log HEAD..origin/main --oneline`** = «چه چیزی روی سرور هست که من ندارم». فوق‌العاده برای صبح‌ها.
- **`git push --follow-tags`** تگ‌های annotated را با push می‌فرستد (lightweight را نه).
- **`git config --global push.autoSetupRemote true`** (Git 2.37+) — دیگر لازم نیست `-u` بزنی.
- **`git remote set-head origin -a`** شاخه‌ی پیش‌فرض remote را خودکار تشخیص می‌دهد.
- **در مخزن با چند remote، `git push -u origin HEAD`** یک الگوی مفید است (HEAD به branch فعلی اشاره می‌کند).

### خلاصه فصل

- remote یک نام مستعار برای آدرس است؛ `origin` فقط قرارداد است.
- **`fetch`** فقط remote-tracking branch را به‌روز می‌کند؛ **`pull`** = fetch + merge/rebase.
- **upstream** با `push -u` تنظیم می‌شود و `push`/`pull` را ساده می‌کند.
- `push` تگ‌ها را نمی‌فرستد، مگر با `--tags` یا `--follow-tags`.
- `--ff-only` و `pull.rebase` رفتار قابل پیش‌بینی می‌دهند.

**سؤالات مروری:**
1. تفاوت `fetch` و `pull` چیست و چرا `fetch` امن‌تر است؟
2. `origin/main` چه زمانی به‌روز می‌شود؟
3. چرا `git push` بعد از `pull` گاهی هنوز رد می‌شود؟

---

## فصل ۱۶: fork و مدل مشارکت متن‌باز

### مقدمه

در یک تیم، همه به یک مخزن دسترسی write دارند. در متن‌باز، این‌طور نیست: هزاران نفر می‌خواهند مشارکت کنند ولی نمی‌توانند مستقیم push کنند. راه‌حل: **fork**.

**چرا مهم است؟** اگر می‌خواهی به پروژه‌های متن‌باز مشارکت کنی (که یکی از بهترین راه‌های رشد است)، fork اولین قدم است. همچنین در سازمان‌هایی که دسترسی write محدود است، fork الگوی رایج است.

**کاربرد عملی:** یک باگ در یک کتابخانه‌ی متن‌باز پیدا کردی. با fork می‌توانی fix کنی، PR بزنی، و اگر پذیرفته شد، اسمت در contributors می‌آید.

### مفاهیم اساسی

#### fork چیست

fork یک **کپی کامل از مخزن** در حساب توست، با تاریخ کامل. GitHub این کپی را در سمت سرور می‌سازد (نه با `git clone`).

```
github.com/original/project     ← مخزن اصلی (upstream)
        ↓ fork (روی سرور GitHub)
github.com/ali/project          ← fork تو (origin)
        ↓ clone
~/project                       ← نسخه‌ی محلی
```

#### سه remote در یک نگاه

| نام | آدرس | دسترسی | نقش |
|---|---|---|---|
| `origin` | `github.com/ali/project` | write | fork تو — اینجا push می‌کنی |
| `upstream` | `github.com/original/project` | read | مخزن اصلی — از اینجا به‌روز می‌کنی |

#### چرخه‌ی مشارکت متن‌باز

```
۱. fork
۲. clone fork
۳. افزودن upstream
۴. branch جدید
۵. تغییر + commit
۶. push به origin
۷. PR به upstream
۸. review و اصلاح
۹. merge (توسط maintainer)
۱۰. حذف branch و sync
```

#### fork در برابر branch

| | fork | branch |
|---|---|---|
| کجا | سرور (حساب تو) | همان مخزن |
| دسترسی write لازم | ❌ | ✅ |
| برای | غریبه‌ها، متن‌باز | اعضای تیم |
| Actions | سهمیه‌ی حساب تو | سهمیه‌ی مخزن |
| Secrets | مال خودت | مال مخزن |

### مثال‌های عملی

#### مثال ۱: چرخه‌ی کامل fork

```bash
# ۱) fork (با gh — خودکار fork و clone می‌کند)
gh repo fork original/project --clone
cd project

# ۲) upstream را چک کن
git remote -v
# origin    git@github.com:ali/project.git
# upstream  https://github.com/original/project.git

# اگر upstream نبود:
git remote add upstream https://github.com/original/project.git

# ۳) branch بساز
git switch -c fix/typo-in-docs

# ۴) تغییر بده
vim docs/install.md
git commit -am "docs: رفع غلط املایی در بخش نصب"

# ۵) push به fork خودت
git push -u origin fix/typo-in-docs

# ۶) PR به upstream
gh pr create --repo original/project \
  --title "docs: رفع غلط املایی در بخش نصب" \
  --body "در بخش نصب، «نصب» به «نصبب» تبدیل شده بود."
```

#### مثال ۲: fork بدون gh

1. در GitHub.com روی **Fork** بزن.
2. `git clone git@github.com:ali/project.git`
3. `git remote add upstream https://github.com/original/project.git`
4. بقیه‌ی مراحل مثل بالا.

#### مثال ۳: sync کردن fork با upstream

```bash
git fetch upstream
git switch main
git merge upstream/main       # یا git rebase upstream/main
git push origin main
```

با gh:

```bash
gh repo sync ali/project --source original/project
```

#### مثال ۴: کار با چند fork

```bash
# اگر روی چند پروژه کار می‌کنی، نام remoteها را معنادار بگذار
git remote rename upstream base
git remote add other-contributor https://github.com/maryam/project.git

# تست یک PR دیگران قبل از merge
git fetch origin pull/42/head:pr-42
git switch pr-42
npm test
```

`refs/pull/<number>/head` یک namespace مخصوص GitHub است — برای تست PRهای دیگران بدون merge عالی است.

#### مثال ۵: نوشتن PR که پذیرفته شود

```markdown
## چه چیزی عوض شد
رفع غلط املایی «نصبب» → «نصب» در `docs/install.md` خط ۲۴.

## چرا
متن documentation در بخش نصب نامفهوم بود.

## چطور تست شد
- [x] لینک‌ها را چک کردم
- [x] `npm run lint` پاس شد

## Issue مرتبط
Closes #128
```

**قواعد طلایی PR متن‌باز:**

1. **اول Issue باز کن** (اگر وجود ندارد) و بپرس «آیا این تغییر مطلوب است؟» — قبل از اینکه ساعت‌ها کار کنی.
2. **`CONTRIBUTING.md` را بخوان.** هر پروژه قواعد خودش را دارد (سبک کد، پیام commit، DCO).
3. **کوچک بفرست.** PR ۲۰ خطی در دو روز merge می‌شود؛ PR ۲۰۰۰ خطی در دو ماه.
4. **تست اضافه کن.** PR بدون تست در اکثر پروژه‌ها رد می‌شود.
5. **صبور باش.** maintainerها داوطلب‌اند. یک هفته بی‌جواب ماندن طبیعی است.

### مقایسه و الگوهای مختلف

#### fork در برابر branch در سازمان

| | fork (خارجی) | branch (داخلی) |
|---|---|---|
| نیاز به invite | ❌ | ✅ |
| دسترسی به secrets | ❌ | ✅ |
| اجرای Actions | سهمیه‌ی fork | سهمیه‌ی org |
| `pull_request` از fork | نیاز به تأیید maintainer برای اجرای workflow | خودکار |
| مناسب | متن‌باز، خارجی | تیم داخلی |

**نکته‌ی امنیتی مهم:** وقتی PR از fork می‌آید، GitHub به‌طور پیش‌فرض workflow را بدون secrets اجرا می‌کند (workflow از fork نمی‌تواند به secrets دسترسی داشته باشد). این طراحی امنیتی است، نه باگ — فصل ۵۴.

#### `pull_request` در برابر `pull_request_target`

| | `pull_request` | `pull_request_target` |
|---|---|---|
| کد اجراشده | کد PR (از fork) | کد base (از مخزن اصلی) |
| دسترسی به secrets | ❌ | ✅ |
| خطر | کم | **بسیار زیاد** اگر کد PR را checkout کنی |

`pull_request_target` یکی از خطرناک‌ترین triggerهاست. اگر روزی دیدی کسی از آن استفاده می‌کند، با احتیاط. فصل ۵۴ کامل توضیح می‌دهد.

### اشکال شایع

#### ۱. PR از `main` fork زده شده

اگر branch کاری‌ات `main` باشد و PR بزنی، هر بار که fork را sync می‌کنی، PR هم عوض می‌شود. **همیشه branch جدا بساز.**

#### ۲. fork قدیمی شده و PR conflict دارد

```bash
git fetch upstream
git switch fix/my-branch
git rebase upstream/main
git push --force-with-lease
```

#### ۳. `upstream` تنظیم نشده

```bash
git remote add upstream https://github.com/original/project.git
git fetch upstream
```

#### ۴. نمی‌توانم Actions را روی fork اجرا کنم

برای forkها، workflow نیاز به تأیید maintainer دارد (بار اول). همچنین secrets وجود ندارد. راه‌حل برای تست: workflow را طوری بنویس که بدون secret هم کار کند.

#### ۵. PR بستم چون maintainer جواب نداد

قبل از بستن، یک comment مؤدبانه بزن: «سلام، آیا این تغییر مطلوب است؟ اگر نه، می‌بندم.» گاهی فقط فراموش شده.

#### ۶. license را نخواندم

اگر پروژه GPL است و شرکتت می‌خواهد استفاده‌ی تجاری کند، ممکن است مشکل حقوقی باشد. همیشه `LICENSE` را ببین.

### نکات مهم (Tips & Tricks)

- **`gh repo fork --clone --remote`** هر سه کار (fork، clone، remote) را انجام می‌دهد.
- **`gh pr checkout 42`** PR شماره ۴۲ را از هر مخزنی checkout می‌کند — بدون نیاز به fetch دستی.
- **`gh pr diff 42`** diff را در ترمینال نشان می‌دهد.
- **قبل از PR، `git log upstream/main..HEAD --oneline`** بزن — ببین دقیقاً چه چیزی می‌فرستی.
- **در متن PR، اسکرین‌شات بگذار** اگر تغییر ظاهری است. نرخ پذیرش چند برابر می‌شود.
- **اگر PR رد شد، آن را شخصی نگیر.** گاهی دلیلش سیاست پروژه است، نه کیفیت کار تو.

### خلاصه فصل

- **fork** یک کپی سروری از مخزن در حساب توست؛ برای مشارکت بدون دسترسی write.
- سه remote: `origin` (fork تو، write)، `upstream` (مخزن اصلی، read).
- چرخه: fork → clone → branch → commit → push → PR → review → merge.
- همیشه از `main` fork، branch جدا بساز.
- PR کوچک، با تست و توضیح، شانس پذیرش چند برابر دارد.

**سؤالات مروری:**
1. چرا در fork باید `upstream` جدا داشته باشی؟
2. چرا PR از branch `main` در fork مشکل‌ساز است؟
3. چطور یک PR دیگران را قبل از merge تست کنی؟

---

## فصل ۱۷: push، force-with-lease و محافظت از تاریخ مشترک

### مقدمه

push عملیاتی است که کد محلی را به سرور می‌فرستد. ساده به نظر می‌رسد، ولی چند تله‌ی جدی دارد که می‌تواند کار هم‌تیمی‌هایت را نابود کند.

**چرا مهم است؟** `git push --force` روی branch مشترک، یکی از مخرب‌ترین دستورهای Git است: commitهای دیگران را بدون هیچ هشداری حذف می‌کند. کسی که این فصل را بخواند، هرگز این کار را نمی‌کند.

**کاربرد عملی:** بعد از rebase، بعد از amend، و وقتی می‌خواهی branch را روی سرور پاک کنی — این فصل جواب می‌دهد.

### مفاهیم اساسی

#### push چه چیزی را می‌فرستد

```
محلی:    A ← B ← C ← D      (main)
سرور:    A ← B              (main)

git push
→ Git می‌گوید: "سرور روی B است، من C و D را دارم که ادامه‌ی B هستند"
→ سرور قبول می‌کند: fast-forward
→ سرور: A ← B ← C ← D
```

push فقط وقتی قبول می‌شود که **fast-forward** باشد: commitهای جدید باید ادامه‌ی commitهای سرور باشند.

#### چه زمانی push رد می‌شود

```
محلی:    A ← B ← X          (main)
سرور:    A ← B ← Y          (main)
```

`X` و `Y` هر دو از `B` آمده‌اند. سرور نمی‌تواند `X` را اضافه کند بدون از دست دادن `Y`. نتیجه:

```
! [rejected] main -> main (non-fast-forward)
```

#### سه راه حل

| راه | دستور | نتیجه |
|---|---|---|
| **merge** | `git pull` سپس `git push` | merge commit، تاریخ شاخه‌دار |
| **rebase** | `git pull --rebase` سپس `git push` | تاریخ خطی |
| **force** | `git push --force` | **کار سرور از بین می‌رود** |

#### `--force` در برابر `--force-with-lease`

| | `--force` | `--force-with-lease` |
|---|---|---|
| چه می‌کند | هر چیزی روی سرور را با محلی جایگزین می‌کند | فقط اگر سرور همان چیزی باشد که آخرین بار دیدی |
| اگر کسی commit جدید زده باشد | پاک می‌شود | خطا می‌دهد |
| سرعت | سریع | سریع |
| توصیه | ❌ هرگز | ✅ همیشه |

`--force-with-lease` یک «قفل خوش‌بینانه» است: Git چک می‌کند که `origin/main` محلی تو با سرور یکی باشد. اگر نباشد، یعنی کسی چیزی push کرده و تو ندیده‌ای — پس متوقف می‌شود.

```bash
# امن‌ترین شکل: صریح بگو انتظار داری سرور کجا باشد
git push --force-with-lease=main:a1b2c3d
```

#### `--force-if-includes` (Git 2.30+)

حتی از `--force-with-lease` هم امن‌تر:

```bash
git push --force-with-lease --force-if-includes
```

این چک می‌کند که commitهای سرور که overwrite می‌شوند، حداقل یک‌بار در reflog محلی تو بوده باشند. یعنی: «فقط چیزی را پاک کن که واقعاً دیده‌ام».

### مثال‌های عملی

#### مثال ۱: push امن بعد از rebase

```bash
git switch feature/login
git fetch origin
git rebase origin/main

# تاریخ عوض شد، push معمولی رد می‌شود
git push
# ! [rejected] (non-fast-forward)

# راه درست:
git push --force-with-lease
```

#### مثال ۲: push بعد از amend

```bash
git commit --amend -m "fix(auth): پیام درست"
git push --force-with-lease
```

#### مثال ۳: push تگ

```bash
git tag -a v1.0.0 -m "نسخه ۱.۰.۰"
git push origin v1.0.0

# همه‌ی تگ‌ها
git push --tags

# فقط تگ‌های annotated که به commitهای push‌شده اشاره می‌کنند
git push --follow-tags
```

**تله:** اگر تگ را جابه‌جا کنی (حذف و دوباره بسازی)، push معمولی رد می‌شود:

```bash
git push --force origin v1.0.0    # برای تگ، force لازم است
```

#### مثال ۴: حذف branch روی سرور

```bash
git push origin --delete feature/old
# یا
git push origin :feature/old
```

GitHub خودش هم می‌تواند بعد از merge پاک کند (`delete_branch_on_merge`).

#### مثال ۵: push به branch دیگر بدون checkout

```bash
# روی main هستی، می‌خواهی یک commit را به feature بفرستی
git push origin HEAD:feature/login

# یا یک commit مشخص
git push origin a1b2c3d:feature/login
```

#### مثال ۶: push خشک (`--dry-run`)

```bash
git push --dry-run
# To github.com:ali/shop.git
#    a1b2c3d..e4f5g6h  main -> main
# (هیچ چیزی فرستاده نشد)
```

قبل از push به branch مهم، این را بزن.

#### مثال ۷: بازیابی بعد از `--force` اشتباه

```bash
# فرض: روی سرور commit مریم را پاک کردی
# ۱) اگر commit را در reflog خودت داری:
git reflog
# e4f5g6h HEAD@{3}: commit: ...

# ۲) branch را به آن برگردان و push کن
git reset --hard e4f5g6h
git push --force-with-lease

# ۳) اگر نداری، از GitHub API استفاده کن:
gh api repos/OWNER/REPO/events --jq '.[] | select(.type=="PushEvent") | .payload.before'
# یا از reflog سرور (GitHub ۹۰ روز نگه می‌دارد، فقط با پشتیبانی)
```

**درس:** قبل از هر force push، یک branch پشتیبان بساز:

```bash
git branch backup-before-force
```

### مقایسه و الگوهای مختلف

#### گزینه‌های push

| گزینه | کار |
|---|---|
| `--force` | overwrite کامل |
| `--force-with-lease` | overwrite با قفل |
| `--force-with-lease=<ref>:<expect>` | با انتظار صریح |
| `--force-if-includes` | حتی امن‌تر |
| `--dry-run` | نمایش بدون ارسال |
| `--tags` | همه‌ی تگ‌ها |
| `--follow-tags` | تگ‌های annotated مربوط |
| `--all` | همه‌ی branchها |
| `--delete` | حذف branch remote |
| `--set-upstream` / `-u` | تنظیم upstream |
| `--atomic` | همه یا هیچ (اگر یکی رد شد، هیچ‌کدام اعمال نشود) |

`--atomic` خیلی مفید است وقتی چند branch/تگ را با هم push می‌کنی:

```bash
git push --atomic origin main v1.0.0
```

#### سیاست force push در تیم

| سیاست | مناسب برای |
|---|---|
| ممنوع مطلق | تیم بزرگ، branchهای مشترک |
| فقط branch شخصی | رایج‌ترین |
| فقط با `--force-with-lease` | حداقل استاندارد |
| آزاد | پروژه‌ی تک‌نفره |

در GitHub می‌توانی با Rulesets، force push را روی `main` ممنوع کنی (فصل ۳۹) — این بهترین دفاع است.

### اشکال شایع

#### ۱. `--force` روی `main`

فاجعه. اگر کسی در آن لحظه push کرده باشد، کارش رفته. **هرگز.**

#### ۲. `--force-with-lease` هم رد می‌شود

```
! [rejected] feature -> feature (stale info)
```

یعنی `origin/feature` محلی تو قدیمی است. `git fetch` بزن و دوباره.

#### ۳. `Everything up-to-date` ولی commit دارم

- روی branch اشتباهی هستی
- commit را نزده‌ای (`git status`)
- upstream اشتباه تنظیم شده

#### ۴. push تگ رد می‌شود

```
! [rejected] v1.0.0 -> v1.0.0 (already exists)
```

تگ با همان نام وجود دارد. اگر واقعاً می‌خواهی جابه‌جا کنی:

```bash
git push --force origin v1.0.0
```

ولی **بهتر:** تگ منتشرشده را هرگز جابه‌جا نکن. یک نسخه‌ی جدید بزن.

#### ۵. `remote: error: GH006: Protected branch update failed`

Ruleset یا branch protection جلوی push مستقیم را گرفته. باید PR بزنی.

#### ۶. حجم زیاد: `remote: error: File too large`

GitHub محدودیت ۱۰۰MB برای هر فایل دارد (و ۲GB برای مخزن توصیه‌شده). برای فایل بزرگ از LFS استفاده کن (فصل ۸۶).

### نکات مهم (Tips & Tricks)

- **`--force-with-lease` را در alias بگذار:**
  ```bash
  git config --global alias.pushf "push --force-with-lease --force-if-includes"
  ```
- **قبل از force push، `git log origin/branch..branch`** بزن — ببین چه چیزی overwrite می‌شود.
- **`git push --atomic`** برای push چندتایی — اگر یکی شکست، همه لغو.
- **`push.default=current`** (اگر می‌خواهی branch فعلی به همین نام push شود):
  ```bash
  git config --global push.default current
  ```
- **قبل از push به `main`، همیشه `git log origin/main..main`** — ببین چه می‌فرستی.
- **در GitHub، Rulesets می‌تواند force push را کاملاً ممنوع کند** — بهترین محافظ.

### خلاصه فصل

- push فقط fast-forward را قبول می‌کند؛ وگرنه باید merge/rebase/force کنی.
- **`--force` خطرناک، `--force-with-lease` امن.**
- `--force-if-includes` حتی امن‌تر است.
- push تگ‌ها را نمی‌فرستد مگر با `--tags`/`--follow-tags`.
- **هرگز روی branch مشترک force نزن** — با Rulesets اجبارش کن.

**سؤالات مروری:**
1. چرا `--force-with-lease` از `--force` امن‌تر است؟
2. چرا push بعد از rebase رد می‌شود؟
3. چطور یک branch را روی سرور پاک کنی؟

---

## فصل ۱۸: اولین Pull Request — از branch تا merge

### مقدمه

Pull Request قلب همکاری در GitHub است: یک درخواست برای merge کردن تغییرات، با فضایی برای بحث، بازبینی و اجرای خودکار تست‌ها. اگر Git را بلد باشی ولی PR را نه، هنوز نمی‌توانی در تیم کار کنی.

**چرا مهم است؟** PR تنها جایی است که کد قبل از رسیدن به `main` دیده می‌شود. تمام کیفیت، امنیت و دانش‌پراکنی تیمی از همین‌جا می‌آید.

**کاربرد عملی:** بعد از این فصل می‌توانی یک PR کامل و حرفه‌ای بزنی که در ۲۴ ساعت review و merge شود.

### مفاهیم اساسی

#### PR چیست (و چه نیست)

PR **یک branch را به branch دیگر پیشنهاد می‌دهد**. خودش چیزی را merge نمی‌کند تا کسی دکمه بزند.

PR نیست:
- یک commit
- یک push
- یک ابزار Git (این مفهوم مال GitHub/GitLab است؛ Git خودش PR ندارد)

#### اجزای یک PR

| بخش | چیست |
|---|---|
| **Base** | branch مقصد (معمولاً `main`) |
| **Compare** | branch منبع (مثلاً `feat/login`) |
| **Title** | عنوان (اگر squash merge کنی، پیام commit می‌شود) |
| **Description** | توضیح، انگیزه، تست، Issue مرتبط |
| **Commits** | لیست commitها |
| **Files changed** | diff |
| **Checks** | نتیجه‌ی Actions |
| **Reviews** | approve / request changes / comment |
| **Mergeable** | آیا conflict دارد؟ |

#### چرخه‌ی حیات

```
Draft ──→ Ready for review ──→ Review ──→ Approved ──→ Merge
  ↑                              ↓
  └────── Changes requested ─────┘
```

- **Draft:** هنوز کار دارد، review نکنید.
- **Ready:** آماده‌ی review.
- **Approved:** تأیید شده.
- **Merged / Closed:** تمام.

#### کلمات کلیدی در توضیح PR

```
Closes #42
Fixes #42
Resolves #42
```

وقتی PR merge شود، Issue خودکار بسته می‌شود. همچنین:

```
Closes owner/other-repo#42
```

برای Issue در مخزن دیگر.

### مثال‌های عملی

#### مثال ۱: PR کامل با gh

```bash
git switch main && git pull
git switch -c feat/user-profile

# ... کار ...

git push -u origin feat/user-profile

gh pr create \
  --title "feat(profile): افزودن صفحه‌ی پروفایل کاربر" \
  --body "$(cat <<'EOF'
## چه چیزی عوض شد
- صفحه‌ی `/profile` با نمایش نام، ایمیل و آواتار
- امکان ویرایش نام
- اعتبارسنجی سمت سرور با zod

## چرا
کاربران نمی‌توانستند اطلاعات خود را ببینند یا تغییر دهند (#42).

## چطور تست شد
- [x] تست واحد برای `validateProfile`
- [x] تست e2e با Playwright
- [x] تست دستی روی موبایل

## اسکرین‌شات
![پروفایل](/assets/profile.png)

## نکات برای reviewer
بخش `getAvatarUrl` را از `utils/avatar.ts` کپی کردم چون هنوز export نشده.
بعداً می‌توانیم refactor کنیم.

Closes #42
EOF
)" \
  --reviewer maryam-ahmadi \
  --label "feature" \
  --base main
```

#### مثال ۲: PR به‌صورت Draft

```bash
gh pr create --draft --title "WIP: بازطراحی صفحه‌ی پرداخت"
```

بعداً آماده شد:

```bash
gh pr ready 42
```

#### مثال ۳: از GitHub.com

1. بعد از push، GitHub یک بنر زرد نشان می‌دهد: **Compare & pull request**.
2. **base:** `main`، **compare:** `feat/user-profile`.
3. عنوان و توضیح بنویس.
4. **Create pull request**.

اگر بنر را ندیدی: تب **Pull requests** → **New pull request**.

#### مثال ۴: PR بین دو branch غیر از main

```
base: develop    compare: release/1.4.0
```

در صفحه‌ی New PR، dropdown «base» را عوض کن.

#### مثال ۵: تبدیل Issue به PR

از صفحه‌ی Issue، سمت راست: **Create a branch** → GitHub branch می‌سازد. بعد از commit، در همان صفحه دکمه‌ی **Compare & pull request** ظاهر می‌شود و Issue خودکار لینک می‌شود.

#### مثال ۶: PR با template

`.github/pull_request_template.md`:

```markdown
## توضیح
<!-- این PR چه کاری می‌کند؟ -->

## نوع تغییر
- [ ] رفع باگ
- [ ] ویژگی جدید
- [ ] تغییر ناسازگار (breaking change)
- [ ] مستندات
- [ ] بازآرایی

## چک‌لیست
- [ ] تست‌ها اضافه/به‌روز شدند
- [ ] مستندات به‌روز شد
- [ ] `npm run lint` پاس شد
- [ ] خودم diff را کامل خواندم

## Issue مرتبط
Closes #
```

هر PR جدید این متن را خودکار می‌گیرد.

#### مثال ۷: خواندن یک PR به‌عنوان reviewer

```
۱. توضیح را بخوان — آیا می‌فهمم چرا؟
۲. تب Conversation — بحث و checks را ببین
۳. تب Files changed — diff را خط‌به‌خط
۴. کامنت بگذار (single comment یا start a review)
۵. Submit review: Comment / Approve / Request changes
```

### مقایسه و الگوهای مختلف

#### Draft در برابر Ready

| | Draft | Ready |
|---|---|---|
| اجرای CI | ✅ (معمولاً) | ✅ |
| درخواست review | ممکن ولی بی‌ادبانه | ✅ |
| نمایش | خاکستری | عادی |
| مناسب | کار نیمه‌کاره، CI زودهنگام | آماده برای بازبینی |

#### PR کوچک در برابر بزرگ

| | کوچک (< ۴۰۰ خط) | بزرگ (> ۱۰۰۰ خط) |
|---|---|---|
| زمان review | ۳۰ دقیقه | چند روز |
| کیفیت review | بالا | سطحی |
| احتمال باگ | کم | زیاد |
| conflict | کم | زیاد |
| توصیه | ✅ | بشکن |

**تحقیق معروف:** SmartBear و Cisco نشان دادند که reviewer بعد از ۴۰۰ خط، توان تشخیص باگ به‌شدت افت می‌کند. PR بزرگ = باگ‌های پیدا نشده.

#### عنوان PR

| ✗ | ✓ |
|---|---|
| `update` | `fix(auth): رفع خطای ورود با ایمیل بزرگ‌حرف` |
| `PR for issue 42` | `feat(profile): افزودن صفحه‌ی پروفایل` |
| `Ali's changes` | `refactor(api): جدا کردن لایه‌ی اعتبارسنجی` |

### اشکال شایع

#### ۱. PR با ۵۰ فایل و ۳۰۰۰ خط

تقریباً review نمی‌شود. اگر مجبوری، در توضیح بنویس «از کجا شروع کنید» و diff را کامنت‌گذاری کن.

#### ۲. PR بدون توضیح

reviewer باید حدس بزند چرا. نتیجه: review کندتر، سؤال‌های بیشتر، احتمال رد.

#### ۳. PR از `main` fork

اگر branch منبع `main` باشد، هر sync آن را عوض می‌کند. همیشه branch جدا.

#### ۴. PR که conflict دارد و نادیده گرفته می‌شود

```bash
git fetch origin
git rebase origin/main
# حل conflict
git push --force-with-lease
```

#### ۵. force push روی PR در حال review

reviewer کامنت‌هایش را روی commitهای قدیمی گذاشته. اگر force push کنی، کامنت‌ها «outdated» می‌شوند و review سخت می‌شود. **قاعده:** در طول review، commit جدید اضافه کن، force نکن (مگر reviewer بخواهد).

#### ۶. PR بسته شد چون branch حذف شد

اگر branch منبع را حذف کنی، PR بسته می‌شود (ولی قابل بازگشایی است تا وقتی commitها در GitHub باشند).

#### ۷. `There isn't anything to compare`

- branch اشتباه انتخاب شده
- commit نزده‌ای
- push نکرده‌ای

### نکات مهم (Tips & Tricks)

- **عنوان PR = پیام commit آینده.** اگر squash می‌کنی، همین در تاریخ می‌ماند.
- **در توضیح، «چرا» را بنویس، نه «چه».** «چه» را diff نشان می‌دهد.
- **`gh pr create --fill`** عنوان و توضیح را از commitها پر می‌کند.
- **`gh pr view --web`** PR فعلی را در مرورگر باز می‌کند.
- **برای PRهای زنجیره‌ای، base را branch قبلی بگذار**، نه `main`.
- **قبل از درخواست review، خودت diff را بخوان.** اکثر باگ‌ها را خودت پیدا می‌کنی.
- **از `@mention` برای جلب توجه استفاده کن، ولی نه زیاد.** هر mention یک وقفه است.

### خلاصه فصل

- PR یک پیشنهاد merge با فضای بحث و بازبینی است.
- اجزا: base، compare، title، description، checks، reviews.
- `Closes #N` در توضیح، Issue را خودکار می‌بندد.
- **PR کوچک review می‌شود؛ PR بزرگ نه.**
- Draft برای کار نیمه‌کاره؛ Ready برای درخواست review.
- در طول review، commit اضافه کن نه force push.

**سؤالات مروری:**
1. چرا PR بزرگ‌تر از ۴۰۰ خط مشکل‌دار است؟
2. تفاوت PR از `main` و از branch جدا چیست؟
3. چطور Issue را به PR وصل کنی تا خودکار بسته شود؟

---

## فصل ۱۹: همگام‌سازی fork و کار تیمی روی یک مخزن

### مقدمه

این فصل دو سناریوی روزمره را حل می‌کند: (۱) fork من قدیمی شده و PR conflict دارد؛ (۲) در تیم، branch من عقب افتاده و باید همگام شود. هر دو یک مسئله‌ی واحدند: **چطور تاریخ محلی را با تاریخ مشترک همگام کنم بدون اینکه چیزی بشکند.**

**چرا مهم است؟** branch عقب‌افتاده = conflict بزرگ‌تر در آینده. همگام‌سازی منظم، conflict را کوچک و قابل مدیریت نگه می‌دارد.

**کاربرد عملی:** یک عادت روزانه: صبح `fetch`، rebase، ادامه‌ی کار.

### مفاهیم اساسی

#### سه حالت branch نسبت به upstream

```bash
git status -sb
## main...origin/main              ← همگام
## main...origin/main [ahead 2]   ← ۲ commit جلو
## main...origin/main [behind 3]  ← ۳ commit عقب
## main...origin/main [ahead 2, behind 3]  ← واگرا (بدترین حالت)
```

**واگرا** یعنی هر دو طرف commit جدید دارند. اینجاست که باید تصمیم بگیری: merge یا rebase.

#### چه زمانی sync کن

| موقعیت | دستور |
|---|---|
| شروع روز کاری | `git fetch` + `git status` |
| قبل از branch جدید | `git switch main && git pull` |
| قبل از PR | `git rebase origin/main` |
| در طول review | `git merge origin/main` (نه rebase) |
| بعد از merge PR | `git switch main && git pull` |

#### قاعده‌ی طلایی sync در طول review

```
قبل از PR:        rebase آزاد است
در طول review:    merge (commit اضافه کن)
```

چرا؟ چون reviewer کامنت‌ها را روی commitهای مشخصی گذاشته. rebase آن‌ها را ناپدید می‌کند.

### مثال‌های عملی

#### مثال ۱: sync fork با upstream (روش امن)

```bash
git fetch upstream
git switch main
git merge upstream/main --ff-only
git push origin main
```

`--ff-only` تضمین می‌کند که اگر main محلی تو commit اضافه دارد، خطا بدهد — که خوب است، چون یعنی اشتباهاً روی main کار کرده‌ای.

#### مثال ۲: rebase branch روی upstream جدید

```bash
git fetch upstream
git switch feat/my-feature
git rebase upstream/main

# اگر conflict:
# حل کن → git add → git rebase --continue

git push --force-with-lease origin feat/my-feature
```

#### مثال ۳: sync با gh (یک دستور)

```bash
gh repo sync ali/project --source original/project --branch main
```

#### مثال ۴: حل conflict در PR بدون rebase

اگر در طول review هستی و `main` جلو رفته:

```bash
git fetch origin
git merge origin/main
# حل conflict
git add .
git commit    # merge commit
git push
```

reviewer تاریخ را می‌بیند و کامنت‌هایش معتبر می‌مانند.

#### مثال ۵: کار تیمی روی یک branch مشترک

گاهی چند نفر روی یک branch کار می‌کنند (مثلاً یک feature بزرگ).

```bash
# علی
git switch -c feat/big-migration
git push -u origin feat/big-migration

# مریم
git fetch origin
git switch feat/big-migration

# هر دو کار می‌کنند
# علی push می‌کند
# مریم باید sync کند:
git pull --rebase
```

**قاعده:** روی branch مشترک، **rebase نکن** (چون هش‌ها عوض می‌شوند و علی را می‌شکند). از `pull --rebase` استفاده کن که فقط commitهای **خودت** را روی remote می‌گذارد — این امن است چون commitهای علی را بازنویسی نمی‌کند.

#### مثال ۶: همگام‌سازی چند branch با هم

```bash
# همه‌ی branchهای محلی که upstream دارند را به‌روز کن
git fetch --all --prune

# ببین کدام عقب‌اند
git for-each-ref --format='%(refname:short) %(upstream:track)' refs/heads | grep behind

# sync همه (فقط اگر fast-forward ممکن باشد)
git for-each-ref --format='%(refname:short)' refs/heads | while read b; do
  git switch "$b" && git merge --ff-only "@{u}" 2>/dev/null || echo "skip $b"
done
```

#### مثال ۷: پاک کردن branchهای مرده

```bash
# محلی، merge‌شده
git branch --merged main | grep -v -E '^\*|main|develop' | xargs -r git branch -d

# remote-tracking
git fetch --prune

# روی سرور (GitHub)
gh api -X GET repos/OWNER/REPO/branches --paginate \
  | jq -r '.[].name' \
  | grep -v -E '^(main|develop)$' \
  | while read b; do gh api -X DELETE "repos/OWNER/REPO/git/refs/heads/$b"; done
```

**احتیاط:** قبل از حذف branchهای remote، مطمئن شو merge شده‌اند.

### مقایسه و الگوهای مختلف

#### merge در برابر rebase برای sync

| | `git merge origin/main` | `git rebase origin/main` |
|---|---|---|
| تاریخ | merge commit | خطی |
| conflict | یک‌بار | ممکن است چند بار (per commit) |
| در طول review | ✅ امن | ❌ کامنت‌ها outdated |
| قبل از PR | مجاز | ✅ ترجیح |
| روی branch مشترک | ✅ | ❌ |

#### `pull --rebase` چرا روی branch مشترک امن است

`git pull --rebase` فقط **commitهای محلی تو** را rebase می‌کند (آن‌هایی که هنوز push نکرده‌ای). commitهای push‌شده دست‌نخورده می‌مانند. پس اگر مریم commit خودش را push کرده باشد، تو آن را بازنویسی نمی‌کنی.

**ولی:** اگر تو هم روی branch مشترک commit push کرده باشی و بعد `pull --rebase` بزنی، commitهای **خودت** بازنویسی می‌شوند — که اگر کسی رویشان کار نکرده باشد، مشکلی نیست.

### اشکال شایع

#### ۱. fork را با «Sync fork» در UI sync کردم ولی PR هنوز conflict دارد

Sync فقط `main` را به‌روز می‌کند، نه branch کاری. باید branch را rebase کنی.

#### ۲. `git pull` merge commitهای زشت می‌سازد

`pull.rebase` را `true` کن (فصل ۵).

#### ۳. conflict در فایل قفل (`package-lock.json`)

```bash
git checkout --ours package-lock.json
npm install
git add package-lock.json
```

#### ۴. branch خیلی عقب افتاده و rebase صد conflict می‌دهد

به‌جای rebase، branch جدید از main بساز و تغییرات را دستی منتقل کن:

```bash
git switch main && git pull
git switch -c feat/restart
git checkout feat/old -- path/to/file    # فایل‌های لازم
```

یا `git cherry-pick` commitهای مهم (فصل ۲۳).

#### ۵. بعد از sync، تست‌ها می‌شکنند

طبیعی است: کد تو با کد جدید main ناسازگار است. این همان چیزی است که sync زودهنگام جلوی فاجعه‌ی بزرگ‌تر را می‌گیرد.

#### ۶. `git pull` روی branch بدون upstream

```
There is no tracking information for the current branch.
```

```bash
git branch --set-upstream-to=origin/feature feature
# یا
git push -u origin feature
```

### نکات مهم (Tips & Tricks)

- **`git status -sb`** سریع‌ترین راه دیدن وضعیت sync.
- **`git fetch --all --prune`** یک عادت صبحگاهی خوب.
- **`git log HEAD..@{u} --oneline`** = «چه چیزی روی سرور هست که من ندارم».
- **`git log @{u}..HEAD --oneline`** = «چه چیزی دارم که push نکرده‌ام».
- **قبل از rebase یک branch پشتیبان بگیر:** `git branch bak/$(date +%s)`.
- **در تیم، قاعده بگذار:** «branch بیش از ۲ روز عقب نماند».

### خلاصه فصل

- سه حالت: ahead، behind، diverged.
- `fetch` + `merge --ff-only` امن‌ترین sync است.
- **قبل از PR rebase، در طول review merge.**
- روی branch مشترک rebase نکن؛ `pull --rebase` امن است.
- `git fetch --prune` branchهای مرده را پاک می‌کند.

**سؤالات مروری:**
1. چرا در طول review نباید rebase کرد؟
2. تفاوت `git pull` و `git pull --rebase` روی branch مشترک چیست؟
3. چطور بفهمی branchات چند commit عقب است؟

---

**بعدی:** [`05-History-Rewriting-and-Recovery.md`](05-History-Rewriting-and-Recovery.md)

---

## منابع و مراجع

- Pro Git، فصل ۲.۵ (کار با remote) — https://git-scm.com/book/fa/v2
- مستندات `git remote` — https://git-scm.com/docs/git-remote
- مستندات `git push` — https://git-scm.com/docs/git-push
- GitHub Docs: Creating a pull request — https://docs.github.com/en/pull-requests
- GitHub Docs: Fork a repo — https://docs.github.com/en/get-started/quickstart/fork-a-repo
- GitHub Docs: Syncing a fork — https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/syncing-a-fork
