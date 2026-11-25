# ۰۳ — گیت، استقرار و گردش کار حرفه‌ای

**مدت یادگیری**: ۳ تا ۴ ساعت
**سطح سختی**: مبتدی
**پیش‌نیاز**: مطالعه فایل‌های ۰۱ و ۰۲، داشتن حساب Vercel و GitHub

---

## فهرست محتوا

- [فصل ۱: Git و GitHub به زبان ساده](#فصل-۱)
- [فصل ۲: چرخه استقرار — Push → Build → Deploy](#فصل-۲)
- [فصل ۳: Preview Deployment و بررسی هر تغییر](#فصل-۳)
- [فصل ۴: Production Deployment و تفاوت محیط‌ها](#فصل-۴)
- [فصل ۵: Rollback و Instant Rollback](#فصل-۵)
- [فصل ۶: Deployment Protection](#فصل-۶)

---

<a id="فصل-۱"></a>
## فصل ۱: Git و GitHub به زبان ساده

### مقدمه

اگر می‌خواهید با Vercel حرفه‌ای کار کنید، باید با Git راحت باشید. نگران نباشید — برای ۹۰٪ کارهای روزمره فقط به چند دستور نیاز دارید. در این فصل، Git و GitHub را از صفر و با تشبیه‌های ساده یاد می‌گیریم.

### مفاهیم اساسی

**۱. Git چیست؟**
Git یک **سیستم کنترل نسخه (Version Control)** است که روی کامپیوتر شما اجرا می‌شود. کارش این است که تاریخچهٔ تغییرات کد شما را نگه دارد، تا بتوانید:
- ببینید چه چیزی، چه زمانی و توسط چه کسی تغییر کرده.
- به هر نسخهٔ قبلی برگردید.
- با دیگران روی یک پروژه کار کنید.

> **تشبیه**: Git مثل «ذخیره بازی» (Save Game) است. هر بار که یک مرحله را تمام می‌کنید، یک Save می‌گیرید و اگر خراب شد، به Save قبلی برمی‌گردید.

**۲. GitHub چیست؟**
GitHub یک سرویس ابری است که مخازن Git شما را میزبانی می‌کند. تفاوت:
- **Git** = ابزار روی کامپیوتر شما.
- **GitHub** = محل ذخیرهٔ آنلاین کد (مثل Google Drive برای کد).

**۳. واژه‌نامهٔ ضروری**

| واژه | معنی |
|---|---|
| **Repository (Repo)** | مخزن؛ پوشهٔ پروژه که Git آن را رصد می‌کند |
| **Commit** | یک «عکس فوری» از تغییرات با پیام توضیحی |
| **Branch** | یک شاخهٔ جدا برای کار موازی |
| **Main / Master** | شاخهٔ اصلی پروژه |
| **Remote** | نسخهٔ آنلاین مخزن (روی GitHub) |
| **Push** | فرستادن Commitها به Remote |
| **Pull** | گرفتن تغییرات از Remote |
| **Merge** | ادغام یک شاخه در شاخهٔ دیگر |
| **Pull Request (PR)** | درخواست ادغام یک شاخه، با امکان بررسی |

**۴. سه حالت فایل در Git**

```
Working Directory  →  Staging Area  →  Repository
   (فایل‌های شما)      (git add)       (git commit)
```

- **Working Directory**: فایل‌هایی که در حال ویرایش‌شان هستید.
- **Staging Area**: تغییراتی که آمادهٔ Commit کردن هستند.
- **Repository**: تغییراتی که ثبت شده‌اند.

### مثال‌های عملی

**مثال ۱ — دستورات ضروری Git**

```bash
# --- راه‌اندازی اولیه ---
git init                      # ساخت مخزن در پوشه فعلی
git clone <url>               # کپی یک مخزن از GitHub

# --- بررسی وضعیت ---
git status                    # چه فایل‌هایی تغییر کرده‌اند؟
git log --oneline             # تاریخچه فشرده
git diff                      # تغییرات ثبت‌نشده

# --- ثبت تغییرات ---
git add .                     # همه فایل‌ها به Staging
git add src/app.js            # فقط یک فایل
git commit -m "افزودن صفحه لاگین"

# --- کار با Remote ---
git remote add origin <url>   # اتصال به مخزن GitHub
git push -u origin main       # فرستادن شاخه اصلی
git pull origin main          # گرفتن تغییرات
```

**مثال ۲ — گردش کار شاخه (Branch)**

```bash
# ساخت شاخه جدید برای یک قابلیت
git checkout -b feature/login

# ... کدنویسی و Commit ...
git add .
git commit -m "افزودن فرم لاگین"

# فرستادن شاخه به GitHub
git push -u origin feature/login

# سپس در GitHub یک Pull Request بسازید
# بعد از تأیید و Merge، شاخه را پاک کنید:
git checkout main
git pull
git branch -d feature/login
```

**مثال ۳ — برگرداندن تغییرات (Undo)**

```bash
# لغو تغییرات یک فایل (برگشت به آخرین Commit)
git restore src/app.js

# خارج کردن فایل از Staging
git restore --staged src/app.js

# اصلاح پیام آخرین Commit
git commit --amend -m "پیام درست"

# برگشت به یک Commit خاص (نسخه جدید commit)
git revert <commit-hash>
```

### مقایسه و الگوهای مختلف

**سه سبک گردش کار Git**

| سبک | توضیح | مناسب برای |
|---|---|---|
| **GitHub Flow** | یک شاخهٔ `main` + شاخه‌های کوتاه‌عمر برای هر قابلیت | ✅ اکثر پروژه‌ها |
| **Git Flow** | شاخه‌های `develop`, `release`, `hotfix` | پروژه‌های بزرگ با انتشار دوره‌ای |
| **Trunk-Based** | همه روی `main`، Commitهای کوچک و مکرر | تیم‌های باتجربه، CI/CD قوی |

**`revert` در برابر `reset`**

| دستور | کار | تاریخچه |
|---|---|---|
| `git revert` | یک Commit جدید می‌سازد که تغییرات قبلی را خنثی می‌کند | ✅ تاریخچه حفظ می‌شود |
| `git reset` | Commitها را حذف/جابه‌جا می‌کند | ⚠️ تاریخچه تغییر می‌کند |

> **قاعده مهم**: روی شاخه‌های مشترک (مثل `main`) همیشه از `revert` استفاده کنید، نه `reset`.

### اشکال شایع

**۱. Commit کردن `node_modules`.**
حجم مخزن را چند صد مگابایت می‌کند. همیشه در `.gitignore` بگذارید.

**۲. Commit کردن فایل `.env`.**
کلیدها و رمزها لو می‌روند — حتی اگر بعداً حذف کنید، در تاریخچه می‌مانند.

**۳. کار مستقیم روی `main` بدون شاخه.**
اگر خراب شود، سایت اصلی خراب می‌شود. همیشه از شاخهٔ جداگانه برای قابلیت‌های جدید استفاده کنید.

**۴. پیام‌های Commit بی‌معنا.**
`"update"`، `"fix"`، `"asdf"` — بعداً نمی‌فهمید چه تغییر کرده. پیام واضح و کوتاه بنویسید.

**۵. فراموش کردن `git pull` قبل از شروع کار.**
باعث تداخل (Conflict) می‌شود. همیشه قبل از کار، شاخه را به‌روز کنید.

### نکات مهم (Tips & Tricks)

- **قرارداد پیام Commit** (Conventional Commits):
  ```
  feat: افزودن صفحه پروفایل
  fix: رفع خطای لاگین در موبایل
  docs: به‌روزرسانی README
  refactor: بازنویسی تابع اعتبارسنجی
  chore: به‌روزرسانی وابستگی‌ها
  ```
- **Commitهای کوچک و مکرر** بهتر از یک Commit بزرگ هستند.
- `.gitignore` استاندارد را از ابتدا بسازید.
- از `git log --oneline --graph` برای دیدن تاریخچهٔ بصری استفاده کنید.
- اگر Commit اشتباهی زدید، `git revert` امن‌ترین راه است.

### خلاصه فصل

- **Git** ابزار کنترل نسخه روی کامپیوتر شما، **GitHub** محل آنلاین آن است.
- سه حالت فایل: **Working → Staging → Repository**.
- دستورات ضروری: `add`، `commit`، `push`، `pull`، `branch`، `status`.
- روی `main` مستقیم کار نکنید؛ از شاخه و Pull Request استفاده کنید.
- هرگز `.env` و `node_modules` را Commit نکنید.

---

<a id="فصل-۲"></a>
## فصل ۲: چرخه استقرار — Push → Build → Deploy

### مقدمه

اکنون ببینیم وقتی `git push` می‌زنید، در پشت صحنه چه اتفاقی می‌افتد. این فصل، «جعبهٔ سیاه» استقرار Vercel را باز می‌کند و به شما نشان می‌دهد دقیقاً در چه مرحله‌ای چه چیزی اجرا می‌شود.

### مفاهیم اساسی

**۱. از Push تا سایت زنده**
```
git push
   ↓
GitHub Webhook → Vercel
   ↓
۱. Clone مخزن (در محیط Build ایزوله)
۲. Install وابستگی‌ها (npm install)
۳. Build پروژه (npm run build)
۴. آپلود خروجی به Edge Network
۵. تخصیص URL و فعال‌سازی
   ↓
سایت زنده ✅
```

**۲. محیط Build**
Vercel هر Build را در یک **کانتینر ایزوله و موقت** اجرا می‌کند:
- سیستم‌عامل Linux (Amazon Linux 2023).
- نسخهٔ Node قابل انتخاب (۱۸، ۲۰، ۲۲...).
- دسترسی به اینترنت برای نصب پکیج.
- بعد از Build، محیط پاک می‌شود.

**۳. Ignored Build Step**
قابلیتی که اجازه می‌دهد Build را «شرطی» کنید. مثلاً اگر فقط فایل README تغییر کرده، Build انجام نشود. باعث صرفه‌جویی در زمان و هزینه می‌شود.

**۴. Build Cache**
Vercel نتایج Build قبلی (مثل `node_modules` و `.next/cache`) را کش می‌کند تا Buildهای بعدی سریع‌تر شوند.

**۵. Concurrency (همزمانی)**
اگر چند Push سریع بزنید، Vercel فقط آخرین را Deploy می‌کند و بقیه را لغو می‌کند (تا منابع هدر نرود).

### مثال‌های عملی

**مثال ۱ — یک چرخهٔ کامل با Commit و Push**

```bash
# ۱. تغییر کد
echo "<h1>نسخه جدید</h1>" > index.html

# ۲. بررسی وضعیت
git status
# modified: index.html

# ۳. ثبت تغییر
git add index.html
git commit -m "feat: به‌روزرسانی صفحه اصلی"

# ۴. فرستادن به GitHub
git push origin main

# ۵. Vercel خودکار شروع می‌کند:
#    Cloning github.com/user/repo (Branch: main)
#    Installing dependencies...
#    Running "npm run build"
#    Uploading build outputs...
#    Deployment completed
#    https://my-app.vercel.app
```

**مثال ۲ — Ignored Build Step (فقط Build در صورت تغییر پوشهٔ مشخص)**

در `vercel.json`:

```json
{
  "git": {
    "deploymentEnabled": {
      "main": true,
      "dev": false
    }
  },
  "ignoreCommand": "git diff --quiet HEAD^ HEAD -- ./apps/web"
}
```

معنی: فقط اگر چیزی در `apps/web` تغییر کرده باشد، Build انجام می‌شود.

**مثال ۳ — تنظیم نسخه Node.js**

روش ۱ — در `package.json`:

```json
{
  "engines": {
    "node": "20.x"
  }
}
```

روش ۲ — در Dashboard: **Project Settings → General → Node.js Version**.

### مقایسه و الگوهای مختلف

**Build خودکار در برابر Build دستی (Prebuilt)**

| معیار | Build در Vercel | Build محلی + `--prebuilt` |
|---|---|---|
| سرعت | خوب (با کش) | سریع‌تر (کنترل کامل) |
| سادگی | ✅ | نیاز به تنظیم |
| مناسب برای | اکثر پروژه‌ها | CI/CD پیشرفته، Monorepo بزرگ |

**سه حالت همزمانی استقرار**

| حالت | رفتار |
|---|---|
| **همه را Deploy کن** | هر Push یک استقرار می‌سازد |
| **فقط آخرین** (پیش‌فرض) | Pushهای قبلی لغو می‌شوند |
| **لغو نکن** | همهٔ استقرارها کامل می‌شوند |

### اشکال شایع

**۱. خطای Build محلی نداشتن اما خطا در Vercel.**
دلیل رایج: تفاوت نسخهٔ Node، نبود متغیر محیطی، یا فایل Commit‌نشده.

**۲. `npm install` خطا می‌دهد.**
ممکن است `package-lock.json` ناسازگار باشد. گاهی با حذف و بازسازی قفل حل می‌شود.

**۳. Build بیش از حد طول می‌کشد.**
وابستگی‌های سنگین، تصاویر بزرگ یا Build نامناسب. از کش و Ignored Build Step استفاده کنید.

**۴. Deploy شدن شاخهٔ اشتباه روی Production.**
بررسی کنید کدام شاخه Production است (Settings → Git → Production Branch).

**۵. تغییرات اعمال نمی‌شود.**
ممکن است Commit نکرده باشید، یا Push به شاخهٔ دیگری رفته باشد. `git log` و وضعیت Deploy را چک کنید.

### نکات مهم (Tips & Tricks)

- **همیشه محلی Build بگیرید** قبل از Push: `npm run build`.
- نسخهٔ Node را **پین** کنید (`engines`) تا تفاوت محیطی نداشته باشید.
- در Monorepoها از **Ignored Build Step** استفاده کنید.
- از **Build Cache** حداکثر استفاده را ببرید (وابستگی‌های تغییرنکرده را دوباره نصب نکنید).
- اگر Build کند است، لاگ آن را دقیق بررسی کنید تا گلوگاه را پیدا کنید.

### خلاصه فصل

- چرخه: **Push → Clone → Install → Build → Deploy → Live**.
- Build در یک محیط ایزولهٔ موقت اجرا می‌شود.
- **Ignored Build Step** جلوی Buildهای بی‌مورد را می‌گیرد.
- **Build Cache** سرعت را بالا می‌برد.
- نسخهٔ Node را پین کنید تا محیط محلی و ابری یکسان باشد.

---

<a id="فصل-۳"></a>
## فصل ۳: Preview Deployment و بررسی هر تغییر

### مقدمه

یکی از قوی‌ترین قابلیت‌های Vercel، **Preview Deployment** است: برای هر شاخه و هر Pull Request، یک نسخهٔ زندهٔ جداگانه ساخته می‌شود. این یعنی می‌توانید قبل از انتشار روی سایت اصلی، تغییرات را در یک آدرس واقعی ببینید و به دیگران نشان دهید.

### مفاهیم اساسی

**۱. Preview Deployment چیست؟**
هر Push به شاخه‌ای غیر از شاخهٔ Production، یک استقرار Preview می‌سازد با آدرس یکتا:

```
https://my-app-git-<branch>-<user>.vercel.app
https://my-app-<hash>-<user>.vercel.app
```

**۲. تفاوت Preview و Production**

| معیار | Preview | Production |
|---|---|---|
| شاخه | غیر از `main` (معمولاً) | `main` |
| دامنه | آدرس موقت یکتا | دامنهٔ اصلی |
| متغیرهای محیطی | Environment = Preview | Environment = Production |
| ایندکس موتور جست‌وجو | ❌ (به‌طور پیش‌فرض) | ✅ |
| مناسب برای | تست، بررسی کد | کاربران واقعی |

**۳. کامنت خودکار روی Pull Request**
Vercel به‌طور خودکار روی هر PR یک کامنت می‌گذارد با لینک Preview، وضعیت Build و اطلاعات استقرار. این کار بررسی (Review) را بسیار ساده می‌کند.

**۴. Aliases (نام‌های مستعار)**
می‌توانید برای یک استقرار، یک نام ساده‌تر بسازید (مثلاً `staging.myapp.com`) تا همیشه به آخرین استقرار آن محیط اشاره کند.

**۵. متغیرهای محیطی مجزا**
می‌توانید برای Preview متغیرهای متفاوتی (مثلاً دیتابیس تست) تعریف کنید تا به داده‌های واقعی دست نزنید.

### مثال‌های عملی

**مثال ۱ — گردش کار کامل با Pull Request**

```bash
# ۱. ساخت شاخه برای قابلیت جدید
git checkout -b feature/new-header

# ۲. کدنویسی و Commit
git add .
git commit -m "feat: طراحی هدر جدید"

# ۳. Push شاخه
git push -u origin feature/new-header

# ۴. Vercel خودکار:
#    - یک Preview Deployment می‌سازد
#    - روی GitHub کامنت می‌گذارد با لینک
#    https://my-app-git-feature-new-header-user.vercel.app

# ۵. در GitHub یک Pull Request بسازید
# ۶. تیم Preview را بررسی می‌کند
# ۷. بعد از تأیید، PR را Merge کنید → Production Deploy
```

**مثال ۲ — متغیر محیطی مجزا برای Preview**

```bash
# متغیر مخصوص محیط Preview
vercel env add DATABASE_URL preview
# مقدار: postgres://test-db...

# بررسی
vercel env ls
# name           value      environments
# DATABASE_URL   Encrypted  Production, Preview
```

**مثال ۳ — ساخت Alias برای محیط Staging**

```bash
# استقرار در Preview
vercel

# خروجی: https://my-app-abc123-user.vercel.app

# ساخت یک Alias ثابت
vercel alias set my-app-abc123-user.vercel.app staging.myapp.com
# اکنون staging.myapp.com همیشه به این نسخه اشاره می‌کند
```

### مقایسه و الگوهای مختلف

**سه استراتژی Branch**

| استراتژی | Production Branch | Preview برای | مناسب برای |
|---|---|---|---|
| **Main-only** | `main` | سایر شاخه‌ها | اکثر پروژه‌ها |
| **Staging + Main** | `main` | `staging` + شاخه‌ها | تیم‌های بزرگ‌تر |
| **Disabled** | — | هیچ (خودکار خاموش) | پروژه‌های حساس |

**نحوهٔ دسترسی به Preview**

| روش | توضیح |
|---|---|
| لینک کامنت PR | سریع‌ترین راه |
| Dashboard → Deployments | مشاهده همه |
| CLI: `vercel ls` | از خط فرمان |
| Alias ثابت | برای Staging |

### اشکال شایع

**۱. Preview با داده‌های واقعی Production کار می‌کند.**
خطرناک است! همیشه برای Preview دیتابیس و کلیدهای جدا تعریف کنید.

**۲. ایندکس شدن Preview توسط گوگل.**
به‌طور پیش‌فرض Vercel با هدر `X-Robots-Tag: noindex` جلوگیری می‌کند، اما اگر دامنهٔ اختصاصی به Preview وصل کنید، مراقب باشید.

**۳. اشتباه گرفتن Preview و Production در تیم.**
ممکن است کسی لینک اشتباه بفرستد. از نام‌گذاری واضح و Alias استفاده کنید.

**۴. Preview قدیمی و گیج‌کننده.**
اگر PR را Merge نکنید، Preview می‌ماند. Deployments را دوره‌ای پاک‌سازی کنید.

**۵. فراموش کردن متغیرهای Preview.**
Build در Preview ممکن است خطا بدهد چون متغیرهای محیطی Preview تعریف نشده‌اند.

### نکات مهم (Tips & Tricks)

- **Preview = محیط تست حرفه‌ای.** از آن برای نمایش به مشتری و بررسی تیم استفاده کنید.
- برای هر محیط (Preview/Production) **متغیرهای جدا** تعریف کنید.
- از **Alias** برای ساخت محیط Staging پایدار استفاده کنید.
- در تنظیمات Git، می‌توانید برای شاخه‌های خاص Preview را **غیرفعال** کنید.
- Preview ها بعد از Merge خودکار به Production می‌روند — بدون کار اضافه.

### خلاصه فصل

- **Preview Deployment** برای هر شاخه/PR یک نسخهٔ زنده می‌سازد.
- آدرس‌ها یکتا و موقت‌اند و روی PR کامنت می‌شوند.
- متغیرهای محیطی Preview جدا از Production هستند.
- **Alias** امکان ساخت آدرس ثابت (Staging) را می‌دهد.
- هرگز Preview را به داده‌های واقعی Production وصل نکنید.

---

<a id="فصل-۴"></a>
## فصل ۴: Production Deployment و تفاوت محیط‌ها

### مقدمه

Production یعنی نسخه‌ای که کاربران واقعی می‌بینند. در این فصل، یاد می‌گیریم Production Deployment چطور فعال می‌شود، چطور محیط‌ها را جدا نگه داریم و چطور بدون ریسک منتشر کنیم.

### مفاهیم اساسی

**۱. Production Deployment چگونه فعال می‌شود؟**
سه راه:
- **Merge یک PR** در شاخهٔ Production (`main`).
- **Push مستقیم** به شاخهٔ Production.
- **`vercel --prod`** از CLI.

**۲. Production Branch**
شاخه‌ای که Vercel آن را Production می‌داند (پیش‌فرض: `main`). در Settings → Git قابل تغییر است.

**۳. سه محیط و متغیرهایشان**

| محیط | زمان فعال شدن | متغیرها |
|---|---|---|
| **Development** | `vercel dev` روی سیستم شما | از `.env.local` |
| **Preview** | هر Push غیر Production | Environment = Preview |
| **Production** | Push/Merge به شاخهٔ اصلی | Environment = Production |

**۴. ترتیب اولویت متغیرها**
```
Production  >  Preview  >  Development
```
اگر متغیری در محیط Production تعریف شده باشد، همان استفاده می‌شود؛ در غیر این صورت به Preview و بعد Development سقوط می‌کند.

**۵. Promotion (ارتقا)**
می‌توانید یک استقرار Preview را **بدون Build مجدد** به Production ارتقا دهید. سریع و مطمئن است، چون همان Build تست‌شده منتشر می‌شود.

**۶. Skew Protection**
اگر کاربری در حین Deploy در سایت باشد، Vercel تضمین می‌کند درخواست‌هایش به همان نسخهٔ سازگار برود (تا خطای ناهماهنگی نسخه پیش نیاید).

### مثال‌های عملی

**مثال ۱ — انتشار از طریق Merge یک Pull Request**

```bash
# فرض: PR تأیید شده و آماده است
# در GitHub، دکمه «Merge pull request» را بزنید

# نتیجه در Vercel:
# - یک Production Deployment جدید ساخته می‌شود
# - دامنه اصلی به نسخه جدید اشاره می‌کند
# - استقرار قبلی حفظ می‌شود (برای Rollback)
```

**مثال ۲ — ارتقای یک Preview به Production (Promotion)**

```
۱. Dashboard → Deployments
۲. استقرار Preview موردنظر را پیدا کنید
۳. منوی «...» → «Promote to Production»
۴. بدون Build مجدد، همان نسخه منتشر می‌شود
```

**مثال ۳ — تنظیم شاخهٔ Production**

```
۱. Project Settings → Git
۲. «Production Branch» را به شاخه دلخواه تغییر دهید (مثلاً release)
۳. ذخیره کنید
```

### مقایسه و الگوهای مختلف

**سه مدل انتشار**

| مدل | توضیح | ریسک | مناسب برای |
|---|---|---|---|
| **Continuous Deployment** | هر Merge مستقیم به Production | متوسط | تیم‌های با تست خوب |
| **Preview → Promote** | ابتدا Preview، بعد Promotion دستی | کم | پروژه‌های حساس |
| **Release Branch** | انتشار از شاخهٔ `release` | کم | انتشار دوره‌ای |

**Promotion در برابر Redeploy**

| معیار | Promotion | Redeploy |
|---|---|---|
| Build مجدد | ❌ | ✅ |
| سرعت | بسیار سریع | معمولی |
| تضمین یکسان بودن | ✅ | ✅ (اگر کد یکسان) |
| مناسب برای | انتشار سریع تست‌شده | بازسازی با تنظیمات جدید |

### اشکال شایع

**۱. Push مستقیم به `main` بدون تست.**
بزرگ‌ترین منبع خرابی سایت. از شاخه و PR استفاده کنید.

**۲. متغیر محیطی Production را اشتباه ست کردن.**
باعث اتصال به دیتابیس اشتباه می‌شود. همیشه بعد از تغییر، یک بار تست کنید.

**۳. فراموش کردن Cache پس از تغییر دامنه.**
ممکن است نسخهٔ قدیمی دیده شود. Cache را Invalidate کنید.

**۴. انتشار در ساعت پرترافیک.**
بهتر است در زمان کم‌ترافیک منتشر کنید تا اگر مشکلی پیش آمد، تأثیر کم باشد.

**۵. نبود Rollback Plan.**
همیشه بدانید چطور سریع برگردید (فصل ۵).

### نکات مهم (Tips & Tricks)

- **همیشه از Preview به Production بروید** — هرگز مستقیم روی `main` کار نکنید.
- از **Promotion** برای انتشار سریع نسخهٔ تست‌شده استفاده کنید.
- متغیرهای محیطی را **از هم جدا** نگه دارید (خصوصاً دیتابیس).
- بعد از هر انتشار، سایت را یک بار به‌صورت دستی بررسی کنید (Smoke Test).
- **Skew Protection** را برای اپ‌های پویا فعال نگه دارید.

### خلاصه فصل

- Production = نسخهٔ کاربران واقعی؛ با Merge/Push به شاخهٔ اصلی فعال می‌شود.
- سه محیط: **Development، Preview، Production** با متغیرهای مستقل.
- **Promotion** اجازه می‌دهد Preview تست‌شده را بدون Build مجدد منتشر کنید.
- از مدل **Preview → Promote** برای کاهش ریسک استفاده کنید.
- همیشه برنامهٔ بازگشت (Rollback) داشته باشید.

---

<a id="فصل-۵"></a>
## فصل ۵: Rollback و Instant Rollback

### مقدمه

حتی بهترین تیم‌ها هم گاهی نسخهٔ خراب منتشر می‌کنند. تفاوت تیم حرفه‌ای در «نساختن خطا» نیست، در «سرعت بازگشت» است. Vercel با **Instant Rollback** این کار را به یک کلیک کاهش داده است.

### مفاهیم اساسی

**۱. Rollback چیست؟**
برگرداندن سایت به یک استقرار قبلی و سالم. در Vercel، همهٔ استقرارها **تغییرناپذیر (Immutable)** هستند؛ یعنی هر نسخهٔ قبلی دست‌نخورده باقی می‌ماند و می‌توان به آن برگشت.

**۲. Instant Rollback**
یک قابلیت که در چند ثانیه دامنهٔ Production را به یک استقرار قبلی برمی‌گرداند — **بدون Build مجدد**. چون نسخهٔ قبلی از قبل ساخته شده و آماده است.

**۳. تفاوت Rollback و Redeploy**

| معیار | Rollback | Redeploy |
|---|---|---|
| Build مجدد | ❌ | ✅ |
| سرعت | چند ثانیه | چند دقیقه |
| هدف | بازگشت فوری به نسخه سالم | انتشار نسخهٔ جدید |
| ریسک | بسیار کم | عادی |

**۴. آیا داده‌ها از بین می‌روند؟**
Rollback فقط **کد** را برمی‌گرداند. داده‌های شما در دیتابیس/KV/Blob **دست‌نخورده** می‌مانند. پس اگر یک مهاجرت دیتابیس (Migration) انجام داده باشید، Rollback کد ممکن است با اسکیمای جدید ناسازگار شود — این را در نظر بگیرید.

### مثال‌های عملی

**مثال ۱ — Instant Rollback از Dashboard**

```
۱. Project → Deployments
۲. یک استقرار قبلی و سالم را پیدا کنید (وضعیت Ready)
۳. منوی «...» → «Instant Rollback»
۴. تأیید کنید
۵. دامنهٔ Production بلافاصله به آن نسخه برمی‌گردد
```

**مثال ۲ — Rollback از CLI**

```bash
# لیست استقرارها
vercel ls my-app

# خروجی:
# url                                status
# my-app-abc123.vercel.app           ● Ready   (فعلی)
# my-app-def456.vercel.app           ● Ready   (قبلی، سالم)

# ارتقای یک استقرار قبلی به Production
vercel promote my-app-def456.vercel.app

# یا با Alias، دامنه را به نسخه قبلی برگردانید
vercel alias set my-app-def456.vercel.app myapp.com
```

**مثال ۳ — استراتژی Rollback سریع با Alias**

```bash
# دامنه اصلی همیشه به یک Alias اشاره کند
vercel alias set my-app-abc123.vercel.app production.myapp.com

# در صورت مشکل، فقط Alias را عوض کنید (چند ثانیه)
vercel alias set my-app-def456.vercel.app production.myapp.com
```

### مقایسه و الگوهای مختلف

**سه روش بازگشت**

| روش | سرعت | مزیت | عیب |
|---|---|---|---|
| **Instant Rollback** | چند ثانیه | ساده، مطمئن | نیاز به نسخهٔ سالم قبلی |
| **Promote نسخه قدیمی** | چند ثانیه | انعطاف بیشتر | کمی دستی‌تر |
| **Git Revert + Push** | چند دقیقه | تاریخچه شفاف | نیاز به Build مجدد |

**Rollback در برابر Forward Fix**

| رویکرد | چه زمانی |
|---|---|
| **Rollback** | خطا بحرانی است و باید فوراً سایت سالم شود |
| **Forward Fix** | خطا کوچک است و اصلاح سریع‌تر از بازگشت است |

### اشکال شایع

**۱. Rollback پس از مهاجرت دیتابیس.**
اگر اسکیمای دیتابیس تغییر کرده، کد قدیمی ممکن است خطا بدهد. مهاجرت‌ها را **سازگار با عقب (Backward Compatible)** طراحی کنید.

**۲. نبود نسخهٔ سالم قبلی.**
اگر چند انتشار پشت‌سرهم خراب بوده، ممکن است نسخهٔ سالم دور باشد. به تاریخچهٔ Deployments نگاه کنید.

**۳. انتظار پاک شدن داده‌ها از Rollback.**
Rollback داده‌ها را برنمی‌گرداند؛ فقط کد.

**۴. گیج شدن با Cache.**
ممکن است Rollback انجام شده باشد اما مرورگر نسخهٔ کش‌شده را نشان دهد. Hard Refresh کنید.

**۵. فراموش کردن اعلان به تیم.**
بعد از Rollback، تیم باید بداند سایت به نسخهٔ قبلی برگشته است.

### نکات مهم (Tips & Tricks)

- **Instant Rollback را قبل از نیاز یاد بگیرید** — در لحظهٔ بحران وقت یادگیری نیست.
- از **Alias** برای Production استفاده کنید تا Rollback فقط یک دستور باشد.
- مهاجرت‌های دیتابیس را **دو مرحله‌ای** انجام دهید: اول اسکیما سازگار، بعد کد جدید.
- بعد از Rollback، یک **Post-mortem** کوتاه برگزار کنید.
- نسخه‌های سالم را برچسب بزنید (مثلاً با Git Tag) تا سریع پیدایشان کنید.

### خلاصه فصل

- استقرارهای Vercel **تغییرناپذیر** هستند؛ بازگشت همیشه ممکن است.
- **Instant Rollback** در چند ثانیه سایت را برمی‌گرداند، بدون Build مجدد.
- Rollback فقط کد را برمی‌گرداند، نه داده‌ها.
- از **Alias** برای Rollback سریع استفاده کنید.
- مهاجرت‌های دیتابیس را سازگار با عقب طراحی کنید.

---

<a id="فصل-۶"></a>
## فصل ۶: Deployment Protection

### مقدمه

اگر سایت شما در حال توسعه است یا محتوای خصوصی دارد، نمی‌خواهید هر کسی به Previewها یا حتی Production دسترسی داشته باشد. **Deployment Protection** مجموعه‌ای از ابزارهاست که کنترل می‌کند چه کسی به کدام استقرار دسترسی دارد.

### مفاهیم اساسی

**۱. چرا محافظت لازم است؟**
- Previewها ممکن است حاوی قابلیت‌های منتشرنشده باشند.
- ممکن است دادهٔ تستی یا محرمانه نمایش دهند.
- ممکن است قبل از انتشار، توسط ربات‌ها ایندکس شوند.

**۲. روش‌های محافظت**

| روش | توضیح | سطح پلن |
|---|---|---|
| **Vercel Authentication** | فقط اعضای تیم با حساب Vercel دسترسی دارند | همه |
| **Password Protection** | با رمز عبور | Pro+ |
| **Trusted IPs** | فقط از IP های مجاز | Enterprise |
| **Shareable Link** | لینک با توکن موقت | Pro+ |
| **Protection Bypass** | توکن برای عبور ابزارهای خودکار | Pro+ |

**۳. Standard Protection در برابر All Deployments**

| حالت | Preview | Production |
|---|---|---|
| **Standard Protection** | 🔒 محافظت‌شده | 🌐 عمومی |
| **All Deployments** | 🔒 محافظت‌شده | 🔒 محافظت‌شده |

**۴. Protection Bypass for Automation**
برای اینکه ابزارهای خودکار (مثل تست‌های E2E یا اسکرین‌شات‌گیرها) بتوانند به Previewهای محافظت‌شده دسترسی داشته باشند، یک توکن ویژه ساخته می‌شود.

### مثال‌های عملی

**مثال ۱ — فعال‌سازی Vercel Authentication**

```
۱. Project Settings → Deployment Protection
۲. «Vercel Authentication» را فعال کنید
۳. انتخاب کنید: Standard Protection یا All Deployments
۴. ذخیره کنید
# اکنون Previewها فقط برای اعضای وارد‌شده باز می‌شوند
```

**مثال ۲ — محافظت با رمز عبور**

```
۱. Project Settings → Deployment Protection
۲. «Password Protection» را فعال کنید
۳. رمز عبور را وارد کنید
۴. ذخیره کنید
# هر بازدیدکننده باید رمز را وارد کند
```

**مثال ۳ — استفاده از Protection Bypass در تست خودکار**

```bash
# دریافت توکن Bypass از Dashboard
# Settings → Deployment Protection → Protection Bypass

# استفاده در درخواست خودکار
curl -H "x-vercel-protection-bypass: YOUR_BYPASS_SECRET" \
     https://my-app-git-branch-user.vercel.app/api/health

# در Playwright (تست E2E)
# extraHTTPHeaders:
#   x-vercel-protection-bypass: YOUR_BYPASS_SECRET
```

### مقایسه و الگوهای مختلف

**چهار سطح محافظت**

| سطح | دسترسی | مناسب برای |
|---|---|---|
| **عمومی** | همه | سایت منتشرشده |
| **Vercel Auth** | اعضای تیم | Preview داخلی |
| **Password** | دارندگان رمز | مشتری/کارفرما |
| **Trusted IPs** | شبکه‌های خاص | سازمانی، امنیتی |

**محافظت Preview در برابر محافظت Production**

| سناریو | تنظیم پیشنهادی |
|---|---|
| پروژه در حال توسعه | همهٔ Deployments محافظت‌شده |
| پروژه منتشرشده | فقط Preview محافظت‌شده |
| سایت داخلی سازمانی | همه محافظت‌شده + Trusted IPs |
| دمو برای مشتری | Password Protection |

### اشکال شایع

**۱. محافظت Production و قطع دسترسی کاربران واقعی.**
مراقب باشید All Deployments را روی سایت زنده فعال نکنید.

**۲. فراموش کردن Bypass برای CI/CD.**
تست‌های خودکار با خطای ۴۰۱ شکست می‌خورند. توکن Bypass را در CI ست کنید.

**۳. اشتباه بین Vercel Auth و Password.**
Vercel Auth نیازمند حساب Vercel است؛ Password برای افراد بیرونی مناسب‌تر است.

**۴. لو رفتن لینک Preview.**
اگرچه محافظت‌شده است، لینک را عمومی نکنید.

**۵. فراموش کردن Shareable Link برای مشتری.**
برای دیدن یک Preview خاص توسط فرد بیرونی، Shareable Link بسازید.

### نکات مهم (Tips & Tricks)

- برای **همهٔ پروژه‌های در حال توسعه**، Standard Protection را روشن کنید.
- برای دموی مشتری از **Password Protection** یا **Shareable Link** استفاده کنید.
- توکن **Protection Bypass** را در متغیرهای محیطی CI ذخیره کنید (نه در کد).
- **Trusted IPs** را فقط برای نیازهای سازمانی استفاده کنید.
- به‌طور دوره‌ای Previewهای قدیمی را پاک کنید تا سطح حمله کاهش یابد.

### خلاصه فصل

- **Deployment Protection** کنترل دسترسی به استقرارها را ممکن می‌کند.
- روش‌ها: **Vercel Auth، Password، Trusted IPs، Shareable Link، Bypass**.
- **Standard Protection** فقط Previewها را محافظت می‌کند؛ **All Deployments** همه را.
- برای CI/CD از **Protection Bypass** استفاده کنید.
- مراقب باشید Production را اشتباهاً قفل نکنید.

---

## منابع و مراجع

### مستندات رسمی
- Git Integration: <https://vercel.com/docs/deployments/git>
- Preview Deployments: <https://vercel.com/docs/deployments/preview-deployments>
- Production Deployments: <https://vercel.com/docs/deployments/environments>
- Instant Rollback: <https://vercel.com/docs/deployments/instant-rollback>
- Deployment Protection: <https://vercel.com/docs/security/deployment-protection>
- Environments & Env Vars: <https://vercel.com/docs/environment-variables>

### مفاهیم پیشنهادی برای مطالعه بیشتر
- Pro Git Book (رایگان): <https://git-scm.com/book>
- Conventional Commits: <https://www.conventionalcommits.org>
- GitHub Flow Guide: <https://docs.github.com/get-started/quickstart/github-flow>

### گام بعدی
- [فایل ۰۴ — Next.js روی Vercel](./04-Nextjs-on-Vercel.md)
- [فایل ۱۲ — امنیت](./12-Security.md)
