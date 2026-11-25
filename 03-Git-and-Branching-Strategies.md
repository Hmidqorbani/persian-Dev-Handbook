# فایل ۰۳ — Git و استراتژی‌های Branching

**مدت یادگیری**: ۹ ساعت
**سطح سختی**: مبتدی–متوسط
**پیش‌نیاز**: فایل ۰۱، فایل ۰۲، آشنایی با خط فرمان لینوکس

---

## فهرست محتوا

- [فصل ۱۱: Git از پایه تا پیشرفته](#فصل-۱۱-git-از-پایه-تا-پیشرفته)
- [فصل ۱۲: استراتژی‌های Branching (GitFlow، GitHub Flow، Trunk-based)](#فصل-۱۲-استراتژیهای-branching-gitflow-github-flow-trunk-based)
- [فصل ۱۳: Pull Request و Code Review](#فصل-۱۳-pull-request-و-code-review)
- [فصل ۱۴: Git Hooks و خودکارسازی](#فصل-۱۴-git-hooks-و-خودکارسازی)
- [فصل ۱۵: Monorepo، Submodule و مدیریت کد در مقیاس](#فصل-۱۵-monorepo-submodule-و-مدیریت-کد-در-مقیاس)

---

## فصل ۱۱: Git از پایه تا پیشرفته

### مقدمه

Git قلب تپندهٔ هر جریان DevOps است. پیش از اینکه خطی از کد را در یک pipeline اجرا کنید، پیش از اینکه کانتینری بسازید، و پیش از اینکه به ابر مستقر شوید — کد شما در Git ذخیره شده است. **Version Control** (کنترل نسخه) بدون Git در دنیای مدرن تقریباً غیرقابل تصور است.

در این فصل از مفهوم پایه‌ای کنترل نسخه شروع می‌کنیم، سپس مدل داده‌ای Git (object model) را کالبدشکافی می‌کنیم تا بفهمید چرا Git این‌قدر سریع و انعطاف‌پذیر است. سپس به عملیات روزمره و دستورات پیشرفته (rebase، cherry-pick، reflog، Bisect) می‌رسیم که هر مهندس DevOps باید با آن‌ها راحت باشد.

### مفاهیم اساسی

**چرا کنترل نسخه (Version Control)؟**

بدون کنترل نسخه، مدیریت تغییرات روی کد یک کابوس است:
- شماره‌گذاری دستی فایل‌ها (`final_v2_real.py`) یک فاجعه است.
- همکاری هم‌زمان چند نفر روی یک فایل غیرممکن است.
- بازگشت به یک نسخهٔ سالم پس از خرابی بسیار سخت است.
- ردیابی اینکه «چه کسی، چه چیزی را، چرا و کی» تغییر داده، ممکن نیست.

**دو نوع کنترل نسخه:**
- **Centralized (متمرکز)** — مانند SVN، Perforce: یک سرور مرکزی دارید؛ کاربران فقط آخرین نسخه را دارند.
- **Distributed (توزیع‌شده)** — مانند **Git**، Mercurial: هر کاربر یک **clone** کامل از تاریخچه (history) را دارد. این یعنی کار آفلاین، سرعت بالا و مقاومت در برابر خرابی سرور.

**مدل داده‌ای Git (Git Object Model):**

Git یک سیستم فایل محتوا-محور (content-addressable) است. چهار نوع شیء (object) دارد:

| شیء | کاربرد |
|---|---|
| **Blob** | محتوای خام فایل (بدون نام) |
| **Tree** | لیست فایل‌ها و پوشه‌ها (مثل پوشه در سیستم‌عامل) |
| **Commit** | اشاره به یک Tree + والد (parent) + متادیتا (نویسنده، پیام، زمان) |
| **Tag** | اشاره به یک Commit (برچسب نسخه) |

هر شیء با **SHA-1 hash** شناسایی می‌شود (مثلاً `a1b2c3...`). این هش بر اساس محتواست، نه مکان — پس اگر محتوا یکسان باشد، هش یکسان است. این ویژگی باعث یکپارچگی (integrity) و سرعت Git می‌شود.

**سه ناحیه (Three Areas) در Git:**

```
Working Directory  →  Staging Area (Index)  →  Repository (.git)
   (فایل‌های خام)        (چیزی که می‌خواهید کامیت کنید)     (تاریخچه دائمی)
```

- `git add` از Working Directory به Staging می‌برد.
- `git commit` از Staging به Repository می‌برد.
- `git checkout` / `git restore` از Repository به Working Directory برمی‌گرداند.

**HEAD** اشاره‌گر (pointer) به branch فعلی است؛ branch هم اشاره‌گر به آخرین commit آن است.

### مثال‌های عملی

**۱. مخزن اولیه و اولین کامیت:**

```bash
# نصب (در اوبونتو)
sudo apt update && sudo apt install -y git

# پیکربندی هویت (ضروری)
git config --global user.name "Hamideh Qorbani"
git config --global user.email "hamideh@example.com"
git config --global init.defaultBranch main

# ساخت مخزن
mkdir myapp && cd myapp
git init
git remote add origin https://github.com/user/myapp.git

# اولین کامیت
echo "# My App" > README.md
git add README.md
git commit -m "Initial commit: add README"
git branch -M main
git push -u origin main
```

**۲. بررسی وضعیت و تاریخچه:**

```bash
git status            # وضعیت ناحیه‌ها
git log --oneline -10 # ۱۰ کامیت آخر به صورت فشرده
git log --graph --all --decorate  # نمایش گراف شاخه‌ها
git diff              # تفاوت Working Directory و Staging
git diff --staged     # تفاوت Staging و آخرین commit
```

**۳. بازنویسی تاریخچه (Rebase تعاملی):**

```bash
# ادغام ۳ کامیت آخر به یک کامیت تمیز
git rebase -i HEAD~3
# در ویرایشگر: اولی را pick، بقیه را squash می‌کنیم
# نتیجه: یک کامیت با پیام ترکیبی
```

**۴. استخراج یک کامیت خاص (Cherry-pick):**

```bash
# انتقال یک کامیت از branch دیگر بدون merge کل
git cherry-pick a1b2c3d4
```

**۵. پیدا کردن کامیت خراب (Bisect):**

```bash
git bisect start
git bisect bad HEAD          # نسخه فعلی خراب است
git bisect good v1.0.0       # v1.0.0 سالم بود
# Git به صورت باینری جستجو می‌کند؛ شما فقط می‌گویید good/bad
git bisect run ./test.sh     # خودکار با یک اسکریپت تست
git bisect reset             # پایان
```

**۶. نجات از اشتباه (Reflog):**

```bash
git reflog            # لیست همهٔ تغییرات HEAD
git reset --hard HEAD@{2}   # بازگشت به ۲ حرکت قبل
# حتی پس از ریست اشتباه، کامیت‌ها در reflog باقی می‌مانند (تا ۳۰ روز)
```

### مقایسه و دیدگاه‌های مختلف

**GitFlow vs GitHub Flow vs Trunk-based:** (جزئیات کامل در فصل ۱۲)

اما در سطح عملیات Git، مقایسه‌های کلیدی:

| عملیات | Merge | Rebase |
|---|---|---|
| تاریخچه | غیرخطی (درختی) | خطی و تمیز |
| ایمنی | همیشه بی‌خطر | می‌تواند تاریخچه را بازنویسی کند |
| کاربرد | ادغام branchهای بلند | تمیز کردن قبل از merge |

قاعده طلایی: **روی branchهای خصوصی rebase کنید، روی branchهای اشتراکی merge کنید.** هرگز روی `main` یا `develop` rebase نکنید چون تاریخچهٔ دیگران را می‌شکنید.

**Git vs Mercurial:** Git محبوب‌تر و ابزارهای اطرافش بیشتر است (GitHub/GitLab/Bitbucket). Mercurial ساده‌تر اما کمتر کاربرد دارد.

**GUI vs CLI:** ابزارهای گرافیکی (GitKraken، SourceTree، VS Code GitLens) برای درک بصری عالی‌اند، اما CLI برای خودکارسازی در pipeline ضروری است.

### اشتباهات رایج

- **کامیت کردن روی `main` به صورت مستقیم:** همیشه روی branch جدا کار کنید.
- **پوش کردن (push) با `--force` روی branch اشتراکی:** از `--force-with-lease` استفاده کنید تا اگر دیگری تغییر کرده بود، خراب نکنید.
- **کامیت کردن رمز عبور و فایل‌های حساس:** همیشه `.gitignore` داشته باشید. اگر رمزی پوش شد، آن را **revoke** کنید (حذف از Git کافی نیست چون در تاریخچه مانده).
- **پیام کامیت ضعیف:** `git commit -m "fix"` فاجعه است. از Conventional Commits استفاده کنید.
- **فراموش کردن `git pull --rebase` قبل از push:** باعث کامیت‌های merge بی‌مورد می‌شود.
- **حذف branch محلی با تغییرات کارنشده (uncommitted):** `git stash` اول، بعد حذف.

### نکات مهم و ترفندها

- **Aliases برای سرعت:**
  ```bash
  git config --global alias.co checkout
  git config --global alias.br branch
  git config --global alias.st status
  git config --global alias.lg "log --graph --oneline --all"
  ```
- **`git stash` برای تغییرات موقت:**
  ```bash
  git stash push -m "wip: feature X"
  git stash list
  git stash pop   # بازگرداندن آخرین stash
  ```
- **Partial staging با `git add -p`:** فقط بخشی از یک فایل را stage کنید.
- **`git blame`** برای دیدن اینکه هر خط را چه کسی نوشته (عالی برای پیگیری باگ).
- **`git worktree`** برای چک‌اوت هم‌زمان چند branch روی پوشه‌های جدا (بدون نیاز به clone دوم).
- **Conventional Commits:** `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `test:` — این‌ها توسط ابزارهایی مثل `semantic-release` خوانده می‌شوند تا نسخه‌گذاری خودکار شود.
- **Sign commits** با GPG برای اثبات اصالت (در محیط‌های سازمانی مهم است).

### خلاصه فصل

Git یک سیستم توزیع‌شدهٔ کنترل نسخه با مدل object-based (Blob/Tree/Commit/Tag) است. سه ناحیه (Working/Staging/Repository) جریان کار را می‌سازند. دستورات پیشرفته مثل `rebase -i`، `cherry-pick`، `bisect` و `reflog` قدرت واقعی Git را نشان می‌دهند. قانون: branch خصوصی را rebase کن، branch اشتراکی را merge کن، و هرگز روی `main` force-push نکن.

---

## فصل ۱۲: استراتژی‌های Branching (GitFlow، GitHub Flow، Trunk-based)

### مقدمه

وجود Git به تنهایی کافی نیست؛ شما باید **قرارداد** (convention) داشته باشید که تیم چطور روی آن کار کند. این قرارداد «استراتژی branching» نامیده می‌شود. انتخاب اشتباه استراتژی باعث تداخل کد، انتشارهای دردناک و سردرگمی می‌شود.

در این فصل سه استراتژی اصلی را با جزئیات، مزایا، معایب و سناریوهای مناسب بررسی می‌کنیم: **GitFlow** (سنتی و سنگین)، **GitHub Flow** (سبک و مدرن)، و **Trunk-based Development** (سریع‌ترین، مناسب DevOps و CI/CD).

### مفاهیم اساسی

**چرا استراتژی branching مهم است؟**
- تعیین می‌کند چطور ویژگی‌ها (features)، اصلاحات (fixes) و انتشارها (releases) جریان می‌یابند.
- روی **DORA metrics** (Lead Time، Deployment Frequency) تأثیر مستقیم دارد.
- هماهنگ‌کنندهٔ همکاری تیم است.

**۱. GitFlow (توسط Vincent Driessen معرفی شد):**

شاخه‌های طولانی‌مدت:
- `main` / `master`: کد تولید (production-ready)، هر کامیت = یک نسخه.
- `develop`: کد تجمیع‌شده برای توسعه.
- `feature/*`: برای ویژگی‌های جدید (از `develop` منشعب، به `develop` merge می‌شود).
- `release/*`: آماده‌سازی انتشار (از `develop`، به `main` و `develop` merge).
- `hotfix/*`: اصلاح اضطراری تولید (از `main`، به `main` و `develop`).

**۲. GitHub Flow (سبک):**

فقط دو نوع شاخه:
- `main`: همیشه قابل استقرار (deployable).
- `feature branches`: هر کار جدید روی branch جدا، سپس Pull Request → review → merge به `main` → deploy خودکار.

**۳. Trunk-based Development (TBD):**

- تقریباً همه روی یک شاخهٔ `main` (trunk) کار می‌کنند.
- Branchهای کوتاه‌عمر (کمتر از ۱-۲ روز) قبل از merge.
- **Feature flags** (پرچم ویژگی) برای جدا کردن استقرار از انتشار.
- نیازمند تست خودکار قوی و CI سریع.

### مثال‌های عملی

**GitFlow — چرخهٔ انتشار:**

```bash
# شروع ویژگی
git checkout develop
git checkout -b feature/login

# پس از تکمیل
git checkout develop
git merge --no-ff feature/login
git branch -d feature/login

# آماده‌سازی انتشار
git checkout -b release/1.2.0 develop
# تست نهایی، رفع باگ‌های جزئی
git checkout main
git merge --no-ff release/1.2.0
git tag -a v1.2.0
git checkout develop
git merge --no-ff release/1.2.0

# hotfix اضطراری
git checkout -b hotfix/1.2.1 main
# رفع
git checkout main
git merge --no-ff hotfix/1.2.1
git tag -a v1.2.1
```

**GitHub Flow — جریان ساده:**

```bash
git checkout -b fix-typo main
# تغییرات
git commit -am "fix: correct typo in README"
git push -u origin fix-typo
# → باز کردن Pull Request در GitHub → Review → Merge
```

**Trunk-based — کار روی trunk با short-lived branch:**

```bash
# صبح: branch کوتاه
git checkout -b feat-search main
# کار در چند ساعت
git commit -am "feat: add search endpoint"
# بعدازظهر: merge سریع
git checkout main
git merge --no-ff feat-search
git push origin main   # CI اجرا می‌شود، deploy خودکار
```

**فایل `.gitignore` نمونه برای پروژه:**

```gitignore
node_modules/
__pycache__/
.env
*.log
dist/
.idea/
```

### مقایسه و دیدگاه‌های مختلف

| معیار | GitFlow | GitHub Flow | Trunk-based |
|---|---|---|---|
| پیچیدگی | بالا | متوسط | کم (اما نیازمند انضباط) |
| سرعت انتشار | کم | متوسط | بالا |
| طول عمر branch | طولانی | متوسط | کوتاه (< ۲ روز) |
| مناسب برای | نرم‌افزار با نسخه‌بندی سخت (مثل دسکتاپ) | وب‌اپلیکیشن‌ها | SaaS، تیم‌های DevOps بالغ |
| نیاز به Feature Flags | کم | متوسط | زیاد |
| یادگیری | سخت | آسان | متوسط |

**کدام را انتخاب کنیم؟**
- محصولی با نسخه‌های صریح و چرخهٔ کند (مثل نرم‌افزار دسکتاپ/تجاری): **GitFlow**.
- وب‌اپلیکیشن با انتشارهای منظم و تیم متوسط: **GitHub Flow**.
- SaaS مدرن، تیم DevOps بالغ، CI/CD کامل: **Trunk-based** (توصیهٔ اکید DevOps).

**دیدگاه DevOps:** Trunk-based Development به عنوان بهترین روش (best practice) برای دستیابی به Lead Time کم و Deployment Frequency بالا شناخته می‌شود، به شرطی که تست خودکار و feature flags داشته باشید.

### اشتباهات رایج

- **Branchهای بلندمدت طولانی:** هرچه branch طولانی‌تر، merge سخت‌تر و احتمال conflict بیشتر. در TBD branch را بیش از ۲ روز زنده نگه ندارید.
- **فراموش کردن `--no-ff` در GitFlow:** باعث می‌شود تاریخچهٔ featureها از بین برود.
- **نداشتن Deploy Branch تمیز:** در GitHub Flow، اگر `main` همیشه deployable نباشد، کل مدل می‌شکند.
- **انتشار بدون Tag:** بدون tag، بازگشت به نسخهٔ پایدار (rollback) سخت است.
- **ترکیب استراتژی‌ها به صورت نیمه‌کاره:** مثلاً Trunk-based بدون feature flags = فاجعه (کد نیمه‌کاره به تولید می‌رود).

### نکات مهم و ترفندها

- **Branch naming convention:** `type/scope-description` مانند `feat/auth-jwt`، `fix/cart-total`، `chore/deps-bump`.
- **Protected branches:** در GitHub/GitLab، `main` را محافظت کنید تا مستقیم push نشود و اجبار به PR + review + status check داشته باشد.
- **Branch cleanup خودکار:** در GitLab گزینهٔ `delete source branch after merge` را فعال کنید.
- **Feature Flags با ابزارهایی مثل LaunchDarkly یا Unleash:** کد را merge کنید اما ویژگی را غیرفعال نگه دارید تا زمان انتشار واقعی.
- **Squash merge** برای تمیز نگه داشتن تاریخچهٔ `main` (چند کامیت کوچک feature → یک کامیت در main).
- **Release branches کوتاه** حتی در GitFlow: release را بیش از چند روز زنده نگه ندارید.

### خلاصه فصل

سه استراتژی اصلی branching عبارتند از GitFlow (سنگین، نسخه‌محور)، GitHub Flow (میانه‌رو) و Trunk-based (سریع، DevOps-friendly). برای DevOps مدرن، Trunk-based همراه با feature flags و CI قوی توصیه می‌شود. کلید موفقیت: branchهای کوتاه‌عمر، protected branches، tag برای هر انتشار، و نام‌گذاری قراردادی.

---

## فصل ۱۳: Pull Request و Code Review

### مقدمه

Pull Request (PR) یا Merge Request (MR در GitLab) نقطهٔ تلاقی کد، همکاری و کیفیت است. PR فقط یک «درخواست ادغام» نیست؛ بلست فرآیندی برای **بررسی کد (Code Review)**، **اجرای خودکار تست‌ها (CI)**، و **انتقال دانش** بین اعضای تیم.

در این فصل یاد می‌گیرید چطور یک PR عالی بنویسید، چطور کد دیگران را مؤثر بازبینی کنید، و چطور PRها را با pipelineهای CI ترکیب کنید.

### مفاهیم اساسی

**Pull Request چیست؟**
یک PR اعلام می‌کند: «من تغییراتی روی branch `feature/X` دارم که می‌خواهم به `main` اضافه شود.» سایر اعضا می‌توانند کامنت بگذارند، approve کنند یا تغییر درخواست کنند.

**چرا Code Review مهم است؟**
- **کاهش باگ:** اشکالات قبل از رسیدن به تولید پیدا می‌شوند.
- **انتقال دانش:** نویسنده و بازبین هر دو یاد می‌گیرند.
- **یکپارچگی معماری:** جلوی انحراف از استانداردهای پروژه گرفته می‌شود.
- **مالکیت مشترک (Collective Ownership):** کد متعلق به تیم است نه فرد.

**اجزای یک PR:**
- عنوان و توضیح (Description) — شامل «چه چیزی، چرا، چطور تست شد».
- لیست تغییرات (Diff).
- بررسی‌کنندگان (Reviewers).
- چک‌لیست‌های (Checklists) خودکار یا دستی.
- CI status checks (pass/fail).

**انواع بازبینی:**
- **Formal review:** تأیید اجباری از چند نفر (محیط‌های regulated).
- **Lightweight review:** یک approve کافی است (رایج در تیم‌های چابک).
- **Pair/Mob review:** بازبینی هم‌زمان چند نفر (در TBD).

### مثال‌های عملی

**۱. ساختار یک توضیح PR خوب (Template):**

```markdown
## توضیح
افزودن احراز هویت JWT به endpointهای محافظت‌شده.

## چرا؟
برای امنیت API بر اساس درخواست فایل ۰۲ (امنیت پایه).

## تغییرات
- افزودن میدل‌ور auth
- به‌روزرسانی مستندات API

## چک‌لیست
- [ ] تست‌ها اضافه شد
- [ ] lint رد نشد
- [ ] مستندات به‌روز شد

## نحوهٔ تست
`pytest tests/test_auth.py -v`
```

**۲. فایل `.github/PULL_REQUEST_TEMPLATE.md`:**

```markdown
### نوع تغییر
- [ ] feat / [ ] fix / [ ] docs / [ ] refactor

### تست‌ها
چه تست‌هایی اضافه یا اجرا شد؟

### اسکرین‌شات (در صورت UI)
```

**۳. اعمال تغییر پس از بازبینی:**

```bash
# اصلاح بر اساس کامنت‌ها
git commit -am "fix: address review comments"
git push   # PR به‌روزرسانی می‌شود، نیازی به PR جدید نیست
```

**۴. یک پیکربندی سادهٔ بررسی‌کنندهٔ اجباری (GitHub branch protection):**

```
Settings → Branches → Add rule (main)
☑ Require a pull request before merging
☑ Required approvals: 1
☑ Require status checks to pass (CI)
☑ Dismiss stale approvals when new commits are pushed
```

### مقایسه و دیدگاه‌های مختلف

| رویکرد | مزیت | معایب | مناسب برای |
|---|---|---|---|
| **Approval اجباری ۱ نفره** | سریع، تعادل خوب | یک نقطهٔ شکست | اکثر تیم‌ها |
| **Approval ۲ نفره** | کیفیت بالاتر | کندتر | تیم مالی/پزشکی |
| **مالکیت کد (CODEOWNERS)** | مسئولیت مشخص | نیازمند نگهداری | تیم‌های بزرگ |
| **Auto-merge** | سرعت بالا | ریسک اگر تست ضعیف | TBD با CI قوی |

**فلسفه بازبینی:** بازبین باید به «کد» نقد کند نه به «نویسنده». جملاتی مثل «این بخش گیج‌کننده است» بهتر از «تو گیج نوشتی» است.

**دیدگاه‌های مختلف:**
- گروهی معتقدند review باید **سنگین و دقیق** باشد (کاهش defect).
- گروهی (طرفدار TBD) معتقدند review باید **سبک و سریع** باشد چون تغییرات کوچک است و CI بار سنگین را برمی‌دارد.

### اشتباهات رایج

- **PRهای غول‌پیکر (huge PR):** PR با ۵۰۰ خط تغییر عملاً بررسی‌ناپذیر است. آن را کوچک (زیر ۲۰۰-۴۰۰ خط) نگه دارید.
- **توضیح خالی:** «fix stuff» باعث می‌شود بازبین وقت بگذارد تا بفهمد چه شد.
- **خودبینی (ego) در نقد:** دفاع از کد به جای پذیرش بهبود.
- **LGTM بدون خواندن:** approve کردن بی‌دلیل کیفیت را می‌کشد.
- **بررسی‌کنندهٔ نامناسب:** کسی را انتخاب کنید که با آن بخش آشنا باشد (از CODEOWNERS استفاده کنید).
- **ادغام بدون انتظار برای CI:** همیشه منتظر pass شدن تست‌ها بمانید.

### نکات مهم و ترفندها

- **کوچک نگه داشتن PR:** اصل طلایی. یک PR = یک تغییر منطقی.
- **Draft PR:** وقتی هنوز کار تمام نیست، PR را `Draft` بگذارید تا بازخورد زودهنگام بگیرید بدون فشار merge.
- **فایل `CODEOWNERS`:**
  ```
  /src/auth/    @team-security
  /infra/       @team-devops
  ```
- **برچسب‌های (labels) PR:** `bug`، `enhancement`، `needs-review` برای سازماندهی.
- **ربات‌های بازبینی:** Dependabot (بروزرسانی وابستگی)، CodeQL (امنیت)، SonarQube (کیفیت).
- **زمان بازبینی:** طبق پژوهش Google، بازبینی را زیر ۱ ساعت انجام دهید تا context-switching کم شود.
- **اتیکت (netiquette):** از پیشنهاد («چطور نظرت دربارهٔ X؟») به جای دستور استفاده کنید.

### خلاصه فصل

PR/MR مرکز همکاری و کیفیت است. یک PR خوب توضیح روشن، تغییر کوچک و چک‌لیست دارد. Code review با احترام، روی کد نه نویسنده، انجام می‌شود. استفاده از CODEOWNERS، branch protection و ربات‌های بازبینی فرآیند را خودکار و ایمن می‌کند. کلید: PR کوچک، توضیح واضح، منتظر CI بمانید.

---

## فصل ۱۴: Git Hooks و خودکارسازی

### مقدمه

Git Hooks اسکریپت‌هایی هستند که در نقاط خاص چرخهٔ حیات Git به صورت خودکار اجرا می‌شوند — مثلاً قبل از کامیت، بعد از کامیت، قبل از push. آن‌ها دریچه‌ای برای **اعمال استانداردها** (lint، فرمت، تست) در سطح محلی (client-side) و **کنترل‌های امنیتی** در سطح سرور (server-side) هستند.

در این فصل انواع hooks، نحوهٔ نوشتن آن‌ها، و ابزارهایی مثل Husky و pre-commit را بررسی می‌کنیم تا کیفیت کد را پیش از رسیدن به CI تضمین کنید.

### مفاهیم اساسی

**Hook چیست؟**
فایل‌های اجرایی در پوشهٔ `.git/hooks/` که نام خاصی دارند. Git آن‌ها را در رویدادهای مشخص صدا می‌زند.

**انواع کلاینت‌ساید (Client-side) Hooks:**

| Hook | زمان اجرا | کاربرد |
|---|---|---|
| `pre-commit` | قبل از ثبت کامیت | lint، فرمت، تست سریع |
| `prepare-commit-msg` | قبل از باز شدن ویرایشگر پیام | قالب‌بندی پیام |
| `commit-msg` | بعد از نوشتن پیام | اعتبارسنجی پیام (Conventional Commits) |
| `post-commit` | بعد از کامیت | نوتیفیکیشن |
| `pre-push` | قبل از push | اجرای تست کامل |
| `pre-rebase` | قبل از rebase | جلوگیری از rebase روی کامیت‌های public |

**انواع سرور-ساید (Server-side) Hooks:**
- `pre-receive` / `update`: قبل از پذیرش push (مثلاً اجبار به GPG signed).
- `post-receive`: بعد از push (مثلاً trigger کردن CI یا deploy).

**چرا خودکارسازی محلی؟**
اجرای lint/tst قبل از کامیت ارزان‌تر از شکست در CI است. «شکست سریع، شکست ارزان» (Fail Fast).

### مثال‌های عملی

**۱. یک `pre-commit` ساده برای چک کردن فایل‌های حساس:**

```bash
#!/usr/bin/env bash
# .git/hooks/pre-commit
set -euo pipefail

# جلوگیری از کامیت فایل‌های .env یا کلید
forbidden=$(git diff --cached --name-only | grep -E '\.env$|secrets/' || true)
if [ -n "$forbidden" ]; then
  echo "❌ متوقف شد: فایل حساس در حال کامیت: $forbidden"
  exit 1
fi

# اجرای linter روی فایل‌های پایتون تغییریافته
changed=$(git diff --cached --name-only --diff-filter=ACM | grep '\.py$' || true)
if [ -n "$changed" ]; then
  echo "$changed" | xargs -P 4 flake8
fi
```

**۲. اعتبارسنجی پیام کامیت (`commit-msg`):**

```bash
#!/usr/bin/env bash
# .git/hooks/commit-msg
msg=$(cat "$1")
if ! echo "$msg" | grep -qE '^(feat|fix|docs|chore|refactor|test|build|ci):'; then
  echo "❌ پیام باید با Conventional Commits شروع شود: feat/fix/docs/..."
  exit 1
fi
```

**۳. استفاده از پکیج `pre-commit` (زبان‌مستقل):**

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-merge-conflict
  - repo: https://github.com/psf/black
    rev: 24.4.2
    hooks:
      - id: black
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
```

نصب:
```bash
pip install pre-commit
pre-commit install
pre-commit run --all-files
```

**۴. Husky برای پروژه‌های Node.js:**

```bash
# package.json
{
  "scripts": {
    "prepare": "husky install"
  },
  "husky": {
    "hooks": {
      "pre-commit": "npm run lint && npm test"
    }
  }
}
```

```bash
# .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"
npm run lint
npm run test:unit
```

### مقایسه و دیدگاه‌های مختلف

| ابزار | زبان | مزیت | معایب |
|---|---|---|---|
| **Git native hooks** | هر زبان | بدون وابستگی | دستی، share نمی‌شود (در `.git` است) |
| **pre-commit (Python)** | چندزبانه | کتابخانهٔ غنی، portable | نیاز به pip |
| **Husky** | Node | محبوب، یکپارچه با npm | فقط جاوااسکریپت |
| **Lefthook** | Go | سریع، cross-platform | اکوسیستم کوچک‌تر |

**نکته مهم:** Hooks محلی را نمی‌توان اجباری کرد (کاربر می‌تواند با `--no-verify` دور بزند). بنابراین hooks **مکمل** CI هستند نه جایگزین آن. کنترل‌های امنیتی حیاتی باید در **server-side hooks** یا **CI** باشند.

**دیدگاه Shift-Left:** هرچه چک‌ها را به چپ (به عقب، به مرحلهٔ محلی) ببرید، ارزان‌تر و سریع‌تر خطا پیدا می‌شود.

### اشتباهات رایج

- **اتکای صرف به hooks محلی:** کاربر می‌تواند `git commit --no-verify` بزند. حتماً در CI هم چک کنید.
- **hooks سنگین که کامیت را کند می‌کنند:** اگر pre-commit ۳۰ ثانیه طول بکشد، تیم آن را دور می‌زند. فقط چک‌های سریع (lint/format) بگذارید؛ تست سنگین در pre-push یا CI.
- **فراموش کردن `chmod +x` روی hook:** Git آن را اجرا نمی‌کند.
- **سخت‌گیری بیش از حد:** hooks نباید جلوی کار را بگیرند؛ راهنما باشند.
- **عدم share کردن hooks:** hooks را در `.git/hooks` نگذارید (version نمی‌شود)؛ از `pre-commit` یا Husky با فایل config استفاده کنید تا در repo باشد.

### نکات مهم و ترفندها

- **`core.hooksPath`:** برای قرار دادن hooks در پوشه‌ای version-controlled (مثل `.githooks/`):
  ```bash
  git config core.hooksPath .githooks
  ```
- **Skip با دلیل:** اگر واقعاً لازم است دور بزنید، کامنت بگذارید چرا: `git commit -m "..." --no-verify # دور زدن موقت`.
- **Detect Secrets:** ابزاری برای جلوگیری از پوش شدن رمز (بسیار مهم برای DevSecOps).
- **Lint-staged:** فقط فایل‌های staged را lint کند (سرعت بالا):
  ```json
  { "lint-staged": { "*.py": ["black", "flake8"] } }
  ```
- **Server-side `pre-receive`** برای اجبار GPG-sign یا جلوگیری از force-push روی `main`.
- **CI as ultimate gate:** hooks فقط لایهٔ اول دفاع هستند.

### خلاصه فصل

Git Hooks اسکریپت‌هایی در نقاط چرخهٔ Git (pre-commit، commit-msg، pre-push و غیره) هستند که کیفیت را خودکار می‌کنند. ابزارهایی مثل pre-commit (Python) و Husky (Node) آن‌ها را portable و shareable می‌کنند. قانون: hooks محلی مکمل CI هستند نه جایگزین (کاربر می‌تواند `--no-verify` بزند)؛ کنترل‌های امنیتی حیاتی باید در server-side یا CI باشند. اصل: شکست سریع و ارزان.

---

## فصل ۱۵: Monorepo، Submodule و مدیریت کد در مقیاس

### مقدمه

وقتی یک سازمان رشد می‌کند، سؤال مهم می‌شود: **یک مخزن بزرگ (Monorepo) یا چندین مخزن کوچک (Polyrepo/Multirepo)؟** انتخاب معماری مخزن روی همکاری، CI، انتشار و مدیریت دسترسی تأثیر می‌گذارد.

در این فصل Monorepo و Polyrepo را مقایسه می‌کنیم، ابزارهای Monorepo (Bazel، Nx، Turborepo، Lerna) را معرفی می‌کنیم، و سپس Git Submodule و Subtree را برای ترکیب مخازن بررسی می‌کنیم.

### مفاهیم اساسی

**Monorepo (Single Repository):**
همهٔ کد (سرویس‌های مختلف، فرانت‌اند، بک‌اند، زیرساخت) در **یک** مخزن Git. مثال‌ها: Google، Meta، Microsoft، Uber.

**Polyrepo / Multirepo (Multiple Repositories):**
هر سرویس یا کامپوننت در مخزن جدا. رایج‌ترین مدل در تیم‌های کوچک تا متوسط.

**Git Submodule:**
مخزن Git داخل مخزن Git دیگر (به عنوان زیرپوشه). اشاره به یک commit خاص از مخزن فرعی دارد.

**Git Subtree:**
ادغام یک مخزن به عنوان پوشه در مخزن اصلی، بدون نیاز به submodule (تاریخچه در هم ادغام می‌شود).

**چرا Monorepo؟**
- **Refactoring اتمیک:** تغییر یک API و همهٔ مصرف‌کنندگانش در یک کامیت.
- **اشتراک کد:** کتابخانه‌های مشترک به راحتی استفاده می‌شوند.
- **دید یکپارچه:** تمام کد سازمان در یک جا.
- **CI هوشمند:** فقط پکیج‌های تغییریافته rebuild شوند (affected builds).

**چرا Polyrepo؟**
- **دسترسی و مالکیت جدا:** تیم A به repo B دسترسی ندارد.
- **انتشار مستقل:** هر سرویس نسخه و چرخهٔ خودش را دارد.
- **مقیاس‌پذیری Git:** مخازن کوچک سریع‌تر کلون می‌شوند.

### مثال‌های عملی

**۱. ساختار یک Monorepo با Nx (Node/TS):**

```
my-org/
├── apps/
│   ├── web/          # فرانت‌اند
│   └── api/          # بک‌اند
├── libs/
│   ├── shared-ui/    # کامپوننت مشترک
│   └── utils/        # توابع مشترک
├── tools/
├── nx.json
└── package.json
```

```bash
npx nx run-many -t build -p affected   # فقط پکیج‌های تغییریافته
npx nx graph                            # نمایش dependency graph
```

**۲. افزودن Submodule:**

```bash
git submodule add https://github.com/org/shared-lib.git libs/shared-lib
git commit -m "chore: add shared-lib submodule"
# کلون بعدی نیاز به:
git submodule update --init --recursive
```

فایل `.gitmodules`:
```ini
[submodule "libs/shared-lib"]
    path = libs/shared-lib
    url = https://github.com/org/shared-lib.git
```

**۳. Subtree (جایگزین سبک‌تر):**

```bash
git subtree add --prefix=libs/shared https://github.com/org/shared.git main --squash
# به‌روزرسانی
git subtree pull --prefix=libs/shared https://github.com/org/shared.git main --squash
```

**۴. پیکربندی CI برای Monorepo (تشخیص تغییرات):**

```yaml
# GitLab CI snippet
changes:
  - apps/api/**/*
only:
  changes:
    - apps/api/**/*
```

### مقایسه و دیدگاه‌های مختلف

| معیار | Monorepo | Polyrepo | Submodule |
|---|---|---|---|
| دسترسی دانه‌ای | سخت | آسان | متوسط |
| Refactor اتمیک | عالی | سخت | سخت |
| انتشار مستقل | نیاز به ابزار | طبیعی | طبیعی |
| پیچیدگی Git | کم (یک repo) | کم | بالا (مدیریت commit اشاره‌گر) |
| مقیاس‌پذیری | نیاز به ابزار (Bazel) | طبیعی | متوسط |
| یادگیری | متوسط | کم | سخت |

**دیدگاه‌ها:**
- **Google/Meta:** Monorepo متمرکز با ابزارهای اختصاصی (Bazel، Buck) برای مقیاس‌پذیری.
- **Netflix/Amazon:** Polyrepo با تیم‌های مستقل (معماری microservice-oriented).
- **Submodule:** اغلب «anti-pattern» شناخته می‌شود چون پیچیدگی Git را بالا می‌برد؛ Subtree یا Package Manager ترجیح داده می‌شود.

**توصیه DevOps:** برای تیم‌های کوچک تا متوسط، Monorepo با ابزار مناسب (Nx/Turbo) معمولاً بهترین تعادل را می‌دهد. برای سازمان‌های بزرگ با جدا سازی صریح، Polyrepo منطقی‌تر است.

### اشتباهات رایج

- **Monorepo بدون ابزار:** بدون Bazel/Nx، Monorepo بزرگ کلون کند و CI کند دارد.
- **Submodule درهم‌رفته:** فراموش کردن `git submodule update --init` باعث پوشهٔ خالی می‌شود.
- **Commit اشاره‌گر قدیمی در Submodule:** اگر submodule را آپدیت نکنید، تیم نسخهٔ قدیمی را می‌بیند.
- **مخلوط کردن مستقل/وابسته:** سرویس کاملاً مستقل را در Monorepo بزرگ قرار دادن = بار اضافی.
- **فقدان CODEOWNERS در Monorepo:** بدون مالکیت مشخص، هر کسی همه جا را تغییر می‌دهد.
- **انتشار یکپارچه در Monorepo:** نباید همه چیز با هر کامیت deploy شود؛ از «affected builds» استفاده کنید.

### نکات مهم و ترفندها

- **Affected builds:** در Monorepo فقط پکیج‌هایی که تغییر کرده‌اند (یا وابستگانشان) را بسازید — سرعت CI ۱۰ برابر.
- **CODEOWNERS در Monorepo:** مسئولیت هر پوشه را مشخص کنید.
- **Trunk-based در Monorepo:** طبق فصل ۱۲، branch کوتاه + feature flags.
- **Package managers به جای Submodule:** برای کتابخانه‌های مشترک، از npm/pip/Go modules استفاده کنید (نسخه‌گذاری تمیزتر).
- **Git Large File Storage (LFS):** برای فایل‌های بزرگ (مدل‌های ML، تصاویر) در Monorepo.
- **Sparse checkout:** در Monorepo بزرگ فقط پوشهٔ مورد نیاز را کلون کنید:
  ```bash
  git sparse-checkout init --cone
  git sparse-checkout set apps/api
  ```

### خلاصه فصل

انتخاب بین Monorepo (یک مخزن بزرگ) و Polyrepo (چند مخزن) روی همکاری و انتشار اثر می‌گذارد. Monorepo با ابزارهایی مثل Nx/Turborepo/Bazel Refactor اتمیک و اشتراک کد آسان می‌دهد اما نیازمند CI هوشمند (affected builds) است. Git Submodule/Subtree برای ترکیب مخازن هستند اما پیچیدگی دارند؛ اغلب Package Manager ترجیح داده می‌شود. کلید: ابزار مناسب برای مقیاس، و CODEOWNERS برای مالکیت.

---

## منابع و مراجع

- Pro Git (Scott Chacon, Ben Straub): https://git-scm.com/book
- Git Documentation: https://git-scm.com/doc
- Conventional Commits: https://www.conventionalcommits.org
- Atlassian Git Tutorials: https://www.atlassian.com/git/tutorials
- GitFlow: https://nvie.com/posts/a-successful-git-branching-model/
- Trunk Based Development: https://trunkbaseddevelopment.com
- pre-commit framework: https://pre-commit.com
- Husky: https://typicode.github.io/husky/
- Nx Monorepo: https://nx.dev
- Turborepo: https://turbo.build/repo
- Google Engineering Practices (Code Review): https://google.github.io/eng-practices/
- Git Submodule Tutorial: https://git-scm.com/book/en/v2/Git-Tools-Submodules

**بعدی:** [فایل ۰۴ — مبانی CI/CD و خطوط لوله](04-CI-CD-Fundamentals.md)
