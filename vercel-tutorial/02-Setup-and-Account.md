# ۰۲ — راه‌اندازی حساب و اولین استقرار

**مدت یادگیری**: ۲ تا ۳ ساعت
**سطح سختی**: مبتدی
**پیش‌نیاز**: مطالعه فایل ۰۱ (آشنایی با Vercel و وب مدرن)، نصب بودن Node.js و Git

---

## فهرست محتوا

- [فصل ۱: ساخت حساب و آشنایی با داشبورد](#فصل-۱)
- [فصل ۲: اتصال GitHub و مدیریت تیم و پروژه](#فصل-۲)
- [فصل ۳: اولین استقرار — از یک فایل HTML تا سایت زنده](#فصل-۳)
- [فصل ۴: نصب و کار با Vercel CLI](#فصل-۴)
- [فصل ۵: ساختار پروژه و فایل تنظیمات vercel.json](#فصل-۵)

---

<a id="فصل-۱"></a>
## فصل ۱: ساخت حساب و آشنایی با داشبورد

### مقدمه

اولین قدم، ساختن حساب کاربری در Vercel است. خبر خوب اینکه این کار کمتر از دو دقیقه طول می‌کشد و برای شروع هیچ کارت بانکی‌ای لازم نیست. در این فصل، هم حساب می‌سازیم و هم نقشهٔ کامل داشبورد را می‌بینیم تا بدانید هر بخش چه کاری انجام می‌دهد.

### مفاهیم اساسی

**۱. راه‌های ثبت‌نام**
Vercel از چند روش ورود پشتیبانی می‌کند:

| روش | مزیت | توصیه |
|---|---|---|
| **GitHub** | یکپارچگی خودکار با مخازن کد | ✅ پیشنهادی |
| **GitLab** | برای تیم‌هایی که GitLab دارند | خوب |
| **Bitbucket** | برای کاربران Atlassian | خوب |
| **ایمیل** | ساده، بدون اتصال Git | برای شروع سریع |

> **توصیه مهم**: اگر قصد دارید پروژه واقعی بسازید، حتماً با **GitHub** وارد شوید. چون در ادامه برای استقرار خودکار به آن نیاز دارید.

**۲. حساب شخصی در برابر تیم (Team)**
- **Personal Account**: حساب شخصی شما؛ همهٔ پروژه‌ها و تنظیمات شخصی.
- **Team**: یک فضای کاری مشترک که چند عضو می‌توانند در آن همکاری کنند. برای پروژه‌های تجاری و تیمی ضروری است.

هر تیم می‌تواند پلن مستقل (Hobby/Pro/Enterprise) داشته باشد.

**۳. پلن پیش‌فرض**
به‌طور خودکار روی پلن **Hobby** (رایگان) قرار می‌گیرید. این پلن برای یادگیری و پروژه‌های شخصی کافی است، اما **برای پروژه تجاری مجاز نیست**.

**۴. ساختار داشبورد**
پس از ورود، این بخش‌های اصلی را می‌بینید:

| بخش | کاربرد |
|---|---|
| **Overview** | نمای کلی پروژه‌ها و آخرین استقرارها |
| **Projects** | لیست همه پروژه‌ها |
| **Deployments** | تاریخچهٔ همه استقرارها (موفق/ناموفق) |
| **Analytics** | آمار بازدیدکنندگان |
| **Observability** | لاگ‌ها، Speed Insights، مصرف منابع |
| **Storage** | ساخت و مدیریت KV / Postgres / Blob / Edge Config |
| **Integrations** | اتصال سرویس‌های بیرونی |
| **Settings** | تنظیمات حساب/تیم، دامنه‌ها، متغیرها، اعضا |

### مثال‌های عملی

**مثال ۱ — ساخت حساب گام‌به‌گام**

```
۱. به https://vercel.com/signup بروید
۲. روی «Continue with GitHub» کلیک کنید
۳. به GitHub هدایت می‌شوید؛ اجازه دسترسی بدهید (Authorize)
۴. نام کاربری (Username) را انتخاب کنید — این در آدرس پیش‌فرض پروژه‌ها ظاهر می‌شود
۵. یک «Scope» (حساب شخصی یا تیم) انتخاب کنید
۶. تمام! وارد داشبورد می‌شوید
```

**مثال ۲ — تنظیمات اولیهٔ حساب**

پس از ورود، در **Settings → Account** این موارد را بررسی کنید:

```
- Name: نام نمایشی شما
- Email: ایمیل اصلی (برای اعلان‌ها)
- Username: شناسهٔ یکتا (در URL پروژه‌ها استفاده می‌شود)
- Avatar: تصویر پروفایل
- Security: فعال‌سازی ورود دو مرحله‌ای (Two-Factor Authentication)
```

**مثال ۳ — ساخت یک تیم (اختیاری اما توصیه‌شده)**

```
۱. از منوی بالا سمت چپ، نام حساب را باز کنید
۲. روی «Create Team» کلیک کنید
۳. نام تیم را وارد کنید (مثلاً my-company)
۴. یک Slug انتخاب کنید (در URL ظاهر می‌شود)
۵. تیم ساخته می‌شود و می‌توانید اعضا را دعوت کنید
```

### مقایسه و الگوهای مختلف

**حساب شخصی در برابر تیم**

| معیار | حساب شخصی | تیم |
|---|---|---|
| هزینه | رایگان (Hobby) | از Pro شروع می‌شود |
| تعداد اعضا | فقط خودتان | چند عضو |
| پروژه تجاری | ❌ مجاز نیست | ✅ مجاز است |
| کنترل دسترسی نقش‌محور | محدود | کامل |
| مناسب برای | یادگیری، پروژه شخصی | کسب‌وکار، تیم |

**سه سطح دسترسی در تیم (Role)**

| نقش | دسترسی |
|---|---|
| **Owner** | کنترل کامل، صورت‌حساب، حذف تیم |
| **Member** | کار روی پروژه‌ها، استقرار |
| **Viewer** | فقط مشاهده (بدون تغییر) |

### اشکال شایع

**۱. استفاده از حساب Hobby برای پروژه تجاری.**
این نقض قوانین Vercel است و ممکن است پروژه‌تان محدود شود. برای کار تجاری پلن Pro بگیرید.

**۲. انتخاب نام کاربری بی‌دقت.**
نام کاربری در آدرس پروژه‌ها (`project-username.vercel.app`) می‌آید؛ قابل تغییر است اما بعداً دردسرساز می‌شود.

**۳. فراموش کردن فعال‌سازی ورود دو مرحله‌ای.**
حساب شما به کد و دامنه وصل است؛ امنیتش جدی است.

**۴. اشتباه بین Scope شخصی و تیمی.**
ممکن است پروژه‌ای بسازید و بعد ببینید در تیم موردنظر نیست. هنگام ساخت، همیشه Scope را چک کنید.

### نکات مهم (Tips & Tricks)

- **همیشه با GitHub وارد شوید** تا بعداً استقرار خودکار فعال شود.
- **Two-Factor Authentication** را از روز اول روشن کنید.
- اگر تیم دارید، از ابتدا **نام‌گذاری منظم پروژه‌ها** را رعایت کنید (مثلاً `web-app`، `docs`، `api`).
- در داشبورد، کلید میانبر `Ctrl/Cmd + K` را امتحان کنید — جست‌وجوی سریع.

### خلاصه فصل

- ثبت‌نام Vercel کمتر از دو دقیقه است و کارت بانکی لازم ندارد.
- با **GitHub** وارد شوید تا یکپارچگی کامل داشته باشید.
- پلن پیش‌فرض **Hobby** است؛ برای کار تجاری باید **Pro** بگیرید.
- تفاوت **حساب شخصی** و **تیم** را بشناسید.
- بخش‌های کلیدی داشبورد: Projects، Deployments، Storage، Observability، Settings.

---

<a id="فصل-۲"></a>
## فصل ۲: اتصال GitHub و مدیریت تیم و پروژه

### مقدمه

حالا که حساب داریم، باید کد را به Vercel وصل کنیم. این اتصال قلب گردش کار Vercel است: هر Push روی GitHub به‌طور خودکار یک استقرار جدید می‌سازد. در این فصل، اتصال GitHub، ساخت پروژه و مدیریت آن را یاد می‌گیریم.

### مفاهیم اساسی

**۱. GitHub App در برابر OAuth**
Vercel از طریق یک **GitHub App** به مخازن شما دسترسی می‌گیرد. تفاوت مهم:

- می‌توانید دسترسی را به **همهٔ مخازن** یا فقط **مخازن منتخب** بدهید.
- دسترسی‌ها از تنظیمات GitHub قابل لغو است.

**۲. مخزن (Repository)**
محل نگهداری کد شما در GitHub. Vercel هر مخزن را می‌تواند به یک **پروژه** تبدیل کند.

**۳. پروژه (Project) در Vercel**
یک پروژه = یک اپلیکیشن که به یک مخزن Git وصل است و تنظیمات Build/Deploy خودش را دارد.

**۴. Framework Preset**
Vercel هنگام Import، فریم‌ورک شما را تشخیص می‌دهد و تنظیمات پیش‌فرض (Build Command، Output Directory) را خودش پر می‌کند.

| فریم‌ورک | Build Command پیش‌فرض | Output Directory |
|---|---|---|
| Next.js | `next build` | `.next` |
| Vite | `vite build` | `dist` |
| Create React App | `react-scripts build` | `build` |
| Astro | `astro build` | `dist` |
| Nuxt | `nuxt build` | `.output` |
| SvelteKit | `vite build` | `.svelte-kit` |
| Other | خالی (دستی) | خالی (دستی) |

**۵. Root Directory (پوشه ریشه)**
اگر کد پروژه در زیرپوشه‌ای از مخزن است (مثل `apps/web`)، باید Root Directory را تنظیم کنید. برای Monorepoها حیاتی است.

### مثال‌های عملی

**مثال ۱ — ساخت یک مخزن و اتصال به Vercel**

```bash
# ۱. ساخت پوشه پروژه محلی
mkdir my-first-vercel-app
cd my-first-vercel-app

# ۲. ساخت فایل index.html
cat > index.html << 'EOF'
<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <title>اولین پروژه Vercel</title>
</head>
<body>
  <h1>سلام دنیا! 👋</h1>
  <p>این اولین سایت من روی Vercel است.</p>
</body>
</html>
EOF

# ۳. تبدیل به مخزن Git
git init
git add .
git commit -m "first commit"
git branch -M main

# ۴. ساخت مخزن در GitHub (با CLI اگر gh نصب است)
gh repo create my-first-vercel-app --public --source=. --push

# یا به‌صورت دستی: مخزن را در github.com بسازید و بعد:
# git remote add origin https://github.com/USERNAME/my-first-vercel-app.git
# git push -u origin main
```

**مثال ۲ — Import پروژه در Vercel**

```
۱. وارد داشبورد Vercel شوید
۲. روی «Add New...» → «Project» کلیک کنید
۳. مخزن my-first-vercel-app را پیدا کنید و «Import» بزنید
۴. صفحه Configure Project باز می‌شود:
   - Framework Preset: Other
   - Root Directory: ./
   - Build Command: (خالی)
   - Output Directory: (خالی)
۵. روی «Deploy» کلیک کنید
۶. چند ثانیه بعد، آدرس زنده شما آماده است 🎉
```

**مثال ۳ — مدیریت اعضای تیم**

```
۱. به Settings تیم بروید
۲. تب «Members» را باز کنید
۳. روی «Invite» کلیک کنید
۴. ایمیل عضو را وارد و نقش (Owner/Member/Viewer) را انتخاب کنید
۵. دعوت‌نامه ارسال می‌شود
```

### مقایسه و الگوهای مختلف

**سه روش اتصال کد به Vercel**

| روش | مزیت | عیب | مناسب برای |
|---|---|---|---|
| **GitHub App** | استقرار خودکار، Preview | نیاز به GitHub | ✅ اکثر پروژه‌ها |
| **Vercel CLI** | سریع، بدون نیاز به Git | تاریخچه Git نمی‌سازد | تست سریع، CI/CD |
| **Deploy Hook / API** | اتوماسیون کامل | پیچیده‌تر | خطوط تولید پیشرفته |

**دسترسی همهٔ مخازن در برابر مخازن منتخب**

| حالت | مزیت | ریسک |
|---|---|---|
| **All repositories** | ساده، همه پروژه‌ها آماده | دسترسی گسترده |
| **Only select repositories** | امن‌تر، محدود | باید هر مخزن را دستی اضافه کنید |

### اشکال شایع

**۱. پیدا نشدن مخزن در لیست Import.**
دلیل: دسترسی GitHub App محدود است. در GitHub → Settings → Applications → Vercel → Configure، مخزن را اضافه کنید.

**۲. اشتباه در Root Directory.**
باعث خطای «No Output Directory» می‌شود. اگر Monorepo دارید، پوشه درست را تنظیم کنید.

**۳. انتخاب Framework Preset اشتباه.**
مثلاً پروژه Vite را با preset Next.js Import کنید → Build Fail.

**۴. Import کردن مخزن اشتباه.**
اگر چند پروژه مشابه دارید، قبل از Deploy نام و Branch را چک کنید.

**۵. Commit نکردن فایل‌های لازم.**
اگر `.gitignore` بیش از حد سختگیرانه باشد، فایل‌های ضروری Push نمی‌شوند و Build شکست می‌خورد.

### نکات مهم (Tips & Tricks)

- از ابتدا **دسترسی را به مخازن منتخب محدود کنید** — اصل کمترین دسترسی.
- برای هر پروژه، یک **Branch اصلی مشخص** (مثل `main`) داشته باشید.
- از **Deploy Preview** برای هر Pull Request استفاده کنید (فایل ۰۳).
- در Monorepoها، از **Root Directory** و **Ignored Build Step** درست استفاده کنید تا فقط بخش تغییریافته Build شود.
- نام پروژه در Vercel را می‌توانید جدا از نام مخزن تنظیم کنید (Settings → General).

### خلاصه فصل

- اتصال GitHub از طریق **GitHub App** انجام می‌شود و قابل محدودسازی است.
- هر مخزن می‌تواند به یک **پروژه Vercel** تبدیل شود.
- **Framework Preset** تنظیمات Build را خودکار پر می‌کند.
- **Root Directory** برای Monorepoها حیاتی است.
- مدیریت تیم با نقش‌های Owner / Member / Viewer انجام می‌شود.

---

<a id="فصل-۳"></a>
## فصل ۳: اولین استقرار — از یک فایل HTML تا سایت زنده

### مقدمه

اکنون لحظهٔ هیجان‌انگیز: اولین سایت واقعی خود را زنده می‌کنیم. در این فصل، سه سناریو را از ساده به پیچیده طی می‌کنیم: یک فایل HTML، یک پروژه استاتیک، و یک اپلیکیشن با تابع سرور. هدف این است که بفهمید «استقرار» در عمل چه شکلی است.

### مفاهیم اساسی

**۱. چرخهٔ استقرار (Deployment Lifecycle)**
هر استقرار این مراحل را طی می‌کند:

```
Queued → Building → Uploading → Deploying → Ready
                                              ↓
                                          (یا Error)
```

| وضعیت | معنی |
|---|---|
| **Queued** | در صف انتظار برای شروع |
| **Building** | در حال نصب وابستگی‌ها و ساخت |
| **Ready** | با موفقیت منتشر شد ✅ |
| **Error** | در ساخت یا انتشار خطا خورد ❌ |
| **Canceled** | دستی لغو شد |

**۲. آدرس‌های پروژه**
- **Production URL**: آدرس اصلی (مثل `my-app.vercel.app`).
- **Preview URL**: برای هر استقرار غیر Production (مثل `my-app-git-branch-user.vercel.app`).
- **Deployment URL**: آدرس یکتای هر استقرار.

**۳. محیط‌ها (Environments)**
- **Production**: Branch اصلی (معمولاً `main`).
- **Preview**: سایر Branchها و Pull Requestها.
- **Development**: با `vercel dev` روی کامپیوتر خودتان.

**۴. Build Settings**
تنظیمات مهمی که هنگام Import می‌بینید:

| تنظیم | توضیح |
|---|---|
| **Build Command** | دستور ساخت (مثل `npm run build`) |
| **Output Directory** | پوشه خروجی (مثل `dist`) |
| **Install Command** | دستور نصب وابستگی (مثل `npm install`) |
| **Node.js Version** | نسخه Node (مثلاً 20.x) |

### مثال‌های عملی

**مثال ۱ — استقرار یک فایل HTML ساده (سریع‌ترین راه)**

```bash
# ساخت پروژه
mkdir html-demo && cd html-demo

# فایل اصلی
cat > index.html << 'EOF'
<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>سایت استاتیک من</title>
  <style>
    body { font-family: system-ui; display: grid; place-items: center;
           min-height: 100vh; margin: 0; background: #f5f5f5; }
    .card { background: #fff; padding: 2rem 3rem; border-radius: 12px;
            box-shadow: 0 4px 20px rgba(0,0,0,.08); }
  </style>
</head>
<body>
  <div class="card">
    <h1>🎉 سایت من زنده شد!</h1>
    <p>منتشر شده با Vercel</p>
  </div>
</body>
</html>
EOF

# استقرار با CLI
vercel          # → محیط Preview
vercel --prod   # → محیط Production
```

**مثال ۲ — پروژه با تابع Serverless**

ساختار پروژه:

```
my-api-app/
├── index.html
├── api/
│   └── time.js
└── package.json
```

```js
// api/time.js — یک endpoint سرور بدون سرور
export default function handler(req, res) {
  const now = new Date();
  res.status(200).json({
    iso: now.toISOString(),
    persian: now.toLocaleString('fa-IR', { timeZone: 'Asia/Tehran' }),
    method: req.method,
  });
}
```

```json
// package.json — برای اینکه Vercel پروژه را Node بشناسد
{
  "name": "my-api-app",
  "version": "1.0.0",
  "private": true
}
```

پس از Deploy، دو آدرس فعال دارید:

```
https://my-api-app.vercel.app/           → index.html
https://my-api-app.vercel.app/api/time   → تابع Serverless
```

**مثال ۳ — مشاهده وضعیت استقرار از CLI**

```bash
# لیست استقرارها
vercel list

# خروجی نمونه:
# age  project      url                                          status
# 1m   my-api-app   my-api-app-abc123.vercel.app                 ● Ready
# 5m   my-api-app   my-api-app-def456.vercel.app                 ● Error

# دیدن لاگ‌های یک استقرار
vercel logs my-api-app-abc123.vercel.app
```

### مقایسه و الگوهای مختلف

**استقرار با Git در برابر استقرار با CLI**

| معیار | Git (خودکار) | CLI (دستی) |
|---|---|---|
| استقرار خودکار | ✅ | ❌ |
| Preview برای PR | ✅ | ❌ |
| تاریخچه Git | ✅ | ❌ |
| سرعت شروع | متوسط | بسیار سریع |
| مناسب برای | پروژه واقعی | تست، اسکریپت، CI |

**سه سناریوی استقرار**

| سناریو | Framework Preset | Build Command | Output Dir |
|---|---|---|---|
| HTML خالص | Other | خالی | خالی |
| Vite/React | Vite | `vite build` | `dist` |
| Next.js | Next.js | `next build` | `.next` |

### اشکال شایع

**۱. خطای «No Output Directory named 'dist' found».**
یعنی Build انجام شده اما خروجی در پوشهٔ دیگری است. Output Directory را با خروجی واقعی هم‌تراز کنید.

**۲. فراموش کردن `package.json`.**
در پروژه‌های Node، نبود این فایل باعث می‌شود Vercel نتواند وابستگی‌ها را نصب کند.

**۳. استقرار با CLI و تعجب از نبود Preview.**
CLI استقرار دستی می‌سازد؛ برای Preview خودکار باید از Git استفاده کنید.

**۴. اشتباه گرفتن `vercel` و `vercel --prod`.**
اولی Preview می‌سازد، دومی Production. اشتباه در این دستور می‌تواند تغییر ناخواسته روی سایت اصلی بگذارد.

**۵. نادیده گرفتن وضعیت Error.**
اگر استقرار Error خورد، Deployment همچنان فعال است اما سایت به‌روز نشده. همیشه وضعیت را چک کنید.

### نکات مهم (Tips & Tricks)

- **قبل از هر Push، محلی Build بگیرید**: `npm run build`. این کار بیشترین خطاها را جلوگیری می‌کند.
- از **`vercel --prod`** فقط وقتی مطمئنید استفاده کنید.
- برای تست سریع بدون Git، CLI فوق‌العاده است؛ برای پروژه واقعی، Git.
- در Dashboard → Deployments می‌توانید **Build Logs** را کامل ببینید — بهترین دوست شما در دیباگ.
- هر استقرار یک **URL دائمی** دارد؛ می‌توانید برای تست مستقیم استفاده کنید.

### خلاصه فصل

- چرخهٔ استقرار: **Queued → Building → Ready/Error**.
- سه محیط: **Development، Preview، Production**.
- ساده‌ترین راه: یک `index.html` + `vercel`.
- با پوشهٔ `api/` می‌توانید توابع سرور بسازید.
- همیشه قبل از Push، محلی Build بگیرید.

---

<a id="فصل-۴"></a>
## فصل ۴: نصب و کار با Vercel CLI

### مقدمه

Vercel CLI یک ابزار خط فرمان قدرتمند است که به شما اجازه می‌دهد بدون باز کردن مرورگر، پروژه بسازید، استقرار کنید، لاگ ببینید و متغیرها را مدیریت کنید. برای توسعه‌دهندگان حرفه‌ای، CLI سریع‌ترین راه کار با Vercel است.

### مفاهیم اساسی

**۱. Vercel CLI چیست؟**
یک برنامهٔ Node.js که روی کامپیوتر شما نصب می‌شود و با API های Vercel ارتباط می‌گیرد.

**۲. دستورات پرکاربرد**

| دستور | کار |
|---|---|
| `vercel login` | ورود به حساب |
| `vercel` | استقرار در Preview |
| `vercel --prod` | استقرار در Production |
| `vercel dev` | اجرای پروژه به‌صورت محلی (شبیه Vercel) |
| `vercel link` | اتصال پوشه محلی به پروژه Vercel |
| `vercel list` | لیست پروژه‌ها/استقرارها |
| `vercel logs <url>` | دیدن لاگ‌ها |
| `vercel env` | مدیریت متغیرهای محیطی |
| `vercel domains` | مدیریت دامنه‌ها |
| `vercel pull` | دریافت تنظیمات پروژه از Vercel |
| `vercel build` | ساخت محلی با تنظیمات Vercel |
| `vercel deploy --prebuilt` | استقرار خروجی ساخته‌شده |

**۳. پوشهٔ `.vercel`**
وقتی با CLI کار می‌کنید، پوشهٔ `.vercel` در پروژه ساخته می‌شود که اطلاعات اتصال به پروژه (Project ID، Org ID) را نگه می‌دارد. این پوشه **نباید** در Git Commit شود.

### مثال‌های عملی

**مثال ۱ — نصب، ورود و اولین استقرار**

```bash
# ۱. نصب سراسری
npm i -g vercel

# ۲. بررسی نسخه
vercel --version

# ۳. ورود (مرورگر باز می‌شود، تأیید کنید)
vercel login

# ۴. بررسی اینکه با کدام حساب وارد شده‌اید
vercel whoami

# ۵. در پوشهٔ پروژه، اولین استقرار
cd my-project
vercel
# سؤال‌هایی می‌پرسد: Set up and deploy? → Y
# Which scope? → حساب یا تیم خود را انتخاب کنید
# Link to existing project? → N
# What's your project's name? → my-project
# In which directory is your code located? → ./
# Want to modify these settings? → N

# ۶. استقرار در Production
vercel --prod
```

**مثال ۲ — مدیریت متغیرهای محیطی با CLI**

```bash
# افزودن یک متغیر برای محیط Production
vercel env add DATABASE_URL production
# سپس مقدار را وارد کنید (مخفی تایپ می‌شود)

# افزودن متغیر برای همهٔ محیط‌ها
vercel env add API_KEY

# لیست متغیرها
vercel env ls

# حذف یک متغیر
vercel env rm DATABASE_URL production

# دریافت متغیرها به فایل محلی (برای توسعه)
vercel env pull .env.local
```

**مثال ۳ — اجرای پروژه به‌صورت محلی شبیه Vercel**

```bash
# اجرای سرور توسعه با شبیه‌سازی محیط Vercel
vercel dev

# خروجی:
# > Ready! Available at http://localhost:3000
# توابع api/ هم به‌صورت محلی در دسترس‌اند:
#   http://localhost:3000/api/time
```

### مقایسه و الگوهای مختلف

**CLI در برابر Dashboard**

| کار | CLI | Dashboard |
|---|---|---|
| استقرار سریع | ✅ سریع‌تر | کندتر |
| دیدن لاگ زنده | ✅ (`vercel logs`) | ✅ |
| مدیریت دامنه | ✅ | ✅ بصری‌تر |
| مدیریت اعضا | محدود | ✅ کامل |
| اتوماسیون/اسکریپت | ✅ عالی | ❌ |
| برای مبتدی | کمی دشوار | ✅ آسان‌تر |

**`vercel dev` در برابر `npm run dev`**

| معیار | `vercel dev` | `npm run dev` |
|---|---|---|
| شبیه‌سازی محیط Vercel | ✅ | ❌ |
| اجرای توابع `api/` | ✅ | فقط با تنظیم دستی |
| متغیرهای محیطی | از Vercel می‌خواند | از `.env.local` |
| سرعت | کمی کندتر | سریع‌تر |
| مناسب برای | تست نزدیک به تولید | توسعه روزمره |

### اشکال شایع

**۱. خطای `vercel: command not found`.**
یعنی CLI نصب نشده یا در PATH نیست. دوباره `npm i -g vercel` بزنید.

**۲. ورود به حساب اشتباه.**
با `vercel whoami` بررسی کنید. برای تغییر: `vercel logout` و بعد `vercel login`.

**۳. Commit کردن پوشهٔ `.vercel`.**
این پوشه شامل شناسه‌های اتصال است و نباید در Git باشد. در `.gitignore` بگذارید.

**۴. انتظار Preview از CLI.**
CLI به‌طور پیش‌فرض استقرار می‌سازد اما Preview مبتنی بر PR ندارد. آن قابلیت مخصوص Git است.

**۵. استفاده از `vercel --prod` روی پروژه نادرست.**
همیشه مطمئن شوید در پوشهٔ درست هستید و پروژهٔ متصل، همانی است که می‌خواهید.

### نکات مهم (Tips & Tricks)

- برای **توسعه روزمره** از `npm run dev` استفاده کنید و برای **تست نهایی** `vercel dev`.
- از `vercel env pull` برای همگام‌سازی متغیرها با فایل `.env.local` استفاده کنید.
- در CI/CD، از `vercel build` + `vercel deploy --prebuilt` برای سرعت بیشتر استفاده کنید.
- `vercel link` را یک‌بار اجرا کنید تا پروژه محلی به پروژهٔ Vercel وصل شود؛ بعد از آن دستورات سریع‌تر اجرا می‌شوند.
- همیشه `.vercel` را در `.gitignore` داشته باشید.

### خلاصه فصل

- **Vercel CLI** ابزار خط فرمان برای استقرار، لاگ، متغیر و مدیریت پروژه است.
- مهم‌ترین دستورات: `vercel`، `vercel --prod`، `vercel dev`، `vercel env`، `vercel logs`.
- پوشهٔ `.vercel` را هرگز Commit نکنید.
- `vercel dev` محیط Vercel را محلی شبیه‌سازی می‌کند.
- برای پروژه‌های واقعی، Git را ترجیح دهید؛ CLI را برای سرعت و اتوماسیون نگه دارید.

---

<a id="فصل-۵"></a>
## فصل ۵: ساختار پروژه و فایل تنظیمات vercel.json

### مقدمه

Vercel در بیشتر موارد بدون هیچ فایل تنظیماتی کار می‌کند، اما برای کنترل دقیق مسیرها، ریدایرکت‌ها، هدرها، توابع و Cronها به فایل `vercel.json` نیاز دارید. در این فصل، ساختار پروژه و این فایل قدرتمند را کامل یاد می‌گیریم.

### مفاهیم اساسی

**۱. ساختار استاندارد یک پروژه**
```
my-project/
├── api/                  ← توابع Serverless (هر فایل = یک endpoint)
│   ├── hello.js
│   └── users/
│       └── [id].js       ← مسیر پویا
├── public/               ← فایل‌های استاتیک (تصویر، فونت)
├── src/                  ← کد منبع
├── package.json          ← وابستگی‌ها و اسکریپت‌ها
├── vercel.json           ← تنظیمات (اختیاری)
├── .vercelignore         ← فایل‌هایی که نباید آپلود شوند
└── .gitignore
```

**۲. فایل `vercel.json` چیست؟**
یک فایل JSON که رفتار پروژه را در Vercel تعیین می‌کند. کلیدهای مهم:

| کلید | کاربرد |
|---|---|
| `buildCommand` | دستور Build سفارشی |
| `outputDirectory` | پوشهٔ خروجی |
| `installCommand` | دستور نصب |
| `framework` | فریم‌ورک (مثلاً `nextjs`) |
| `rewrites` | بازنویسی مسیر (بدون تغییر URL) |
| `redirects` | ریدایرکت (با تغییر URL) |
| `headers` | هدرهای سفارشی HTTP |
| `cleanUrls` | حذف `.html` از URL |
| `trailingSlash` | اسلش انتهایی |
| `regions` | منطقه‌های اجرای توابع |
| `functions` | تنظیمات توابع (حافظه، مدت) |
| `crons` | زمان‌بندی Cron Jobs |
| `images` | تنظیمات بهینه‌سازی تصویر |

**۳. تفاوت `rewrites` و `redirects`**
- **rewrite**: URL در مرورگر **تغییر نمی‌کند**، اما محتوای دیگری نمایش داده می‌شود.
- **redirect**: مرورگر به URL **جدید** هدایت می‌شود (آدرس عوض می‌شود).

**۴. فایل `.vercelignore`**
مانند `.gitignore`، اما مخصوص Vercel. فایل‌هایی که نمی‌خواهید آپلود شوند را اینجا لیست می‌کنید.

### مثال‌های عملی

**مثال ۱ — یک `vercel.json` کامل و کاربردی**

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "framework": "nextjs",
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "installCommand": "npm install",
  "regions": ["iad1", "fra1"],
  "cleanUrls": true,
  "trailingSlash": false,

  "redirects": [
    { "source": "/old-blog", "destination": "/blog", "permanent": true },
    { "source": "/twitter", "destination": "https://twitter.com/example", "permanent": false }
  ],

  "rewrites": [
    { "source": "/api/proxy/:path*", "destination": "https://api.example.com/:path*" },
    { "source": "/dashboard/:path*", "destination": "/app?view=:path*" }
  ],

  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        { "key": "Access-Control-Allow-Origin", "value": "*" },
        { "key": "Cache-Control", "value": "no-store" }
      ]
    },
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "X-Frame-Options", "value": "DENY" }
      ]
    }
  ]
}
```

**مثال ۲ — تنظیمات توابع و Cron**

```json
{
  "functions": {
    "api/heavy-task.js": {
      "memory": 1024,
      "maxDuration": 60
    }
  },
  "crons": [
    { "path": "/api/daily-report", "schedule": "0 3 * * *" },
    { "path": "/api/cleanup", "schedule": "0 0 * * 0" }
  ]
}
```

> `maxDuration` بالاتر از حد پلن، نیاز به پلن Pro/Enterprise دارد.

**مثال ۳ — مسیرهای پویا در توابع**

```
api/
├── users/
│   ├── index.js      → /api/users
│   └── [id].js       → /api/users/123
└── posts/
    └── [...slug].js  → /api/posts/a/b/c (catch-all)
```

```js
// api/users/[id].js — مسیر پویا
export default function handler(req, res) {
  const { id } = req.query;   // مقدار از URL
  res.json({ userId: id, name: `کاربر ${id}` });
}
```

### مقایسه و الگوهای مختلف

**`rewrites` در برابر `redirects` در برابر Middleware**

| معیار | rewrites | redirects | Middleware |
|---|---|---|---|
| تغییر URL مرورگر | ❌ | ✅ | بستگی دارد |
| منطق پویا | محدود | محدود | ✅ کامل |
| محل تنظیم | `vercel.json` | `vercel.json` | کد (`middleware.ts`) |
| سرعت | بسیار بالا (لبه) | بسیار بالا | بالا |
| مناسب برای | پروکسی، SPA routing | تغییر آدرس دائمی | احراز هویت، A/B |

**SPA routing: دو راه**

```json
// راه ۱: با rewrites در vercel.json
{ "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }] }
```
```js
// راه ۲: تنظیم در خود فریم‌ورک (Vite/React Router)
// معمولاً با یک فایل _redirects در public/
```

### اشکال شایع

**۱. JSON نامعتبر.**
یک کاما یا براکت اشتباه، کل پروژه را از کار می‌اندازد. از `$schema` و اعتبارسنجی ویرایشگر استفاده کنید.

**۲. تداخل rewrite و فایل واقعی.**
اگر rewrite ای همهٔ مسیرها را به `index.html` بفرستد، ممکن است فایل‌های واقعی هم مسدود شوند. از الگوهای دقیق‌تر استفاده کنید.

**۳. اشتباه بین `source` و `destination`.**
همیشه `source` مسیر ورودی و `destination` مقصد است.

**۴. تنظیم دستی `buildCommand` روی پروژه‌ای که تشخیص خودکار دارد.**
می‌تواند تنظیمات درست Vercel را خراب کند. فقط در صورت نیاز تغییر دهید.

**۵. گذاشتن رمز و کلید در `vercel.json`.**
هرگز! این فایل در مخزن Commit می‌شود. اطلاعات حساس فقط در **Environment Variables**.

### نکات مهم (Tips & Tricks)

- اگر نیازی به تنظیم خاصی ندارید، **`vercel.json` نسازید** — Vercel هوشمند است.
- همیشه `"$schema"` را در ابتدای فایل بگذارید تا ویرایشگر خطاها را نشان دهد.
- برای تست تنظیمات، `vercel dev` را با فایل `vercel.json` اجرا کنید.
- از **rewrites** برای پروکسی امن به API های بیرونی استفاده کنید (بدون افشای کلید در کلاینت).
- `cleanUrls` را برای سایت‌های استاتیک فعال کنید تا URL ها تمیزتر شوند.

### خلاصه فصل

- ساختار پروژه: `api/` برای توابع، `public/` برای استاتیک، `src/` برای کد.
- `vercel.json` کنترل کامل مسیرها، هدرها، توابع و Cronها را می‌دهد.
- تفاوت کلیدی: **rewrite** (URL عوض نمی‌شود) و **redirect** (URL عوض می‌شود).
- مسیرهای پویا با `[id]` و `[...slug]` در پوشهٔ `api/` ساخته می‌شوند.
- هیچ‌وقت اطلاعات حساس را در `vercel.json` نگذارید.

---

## منابع و مراجع

### مستندات رسمی
- Project Configuration (vercel.json): <https://vercel.com/docs/projects/project-configuration>
- Vercel CLI Reference: <https://vercel.com/docs/cli>
- Deployments Overview: <https://vercel.com/docs/deployments/overview>
- Git Integration: <https://vercel.com/docs/deployments/git>
- Functions Overview: <https://vercel.com/docs/functions>

### مفاهیم پیشنهادی برای مطالعه بیشتر
- GitHub Docs — Repositories: <https://docs.github.com/repositories>
- HTTP Headers (MDN): <https://developer.mozilla.org/docs/Web/HTTP/Headers>
- Cron Expression Guide: <https://crontab.guru>

### گام بعدی
- [فایل ۰۳ — گیت، استقرار و گردش کار حرفه‌ای](./03-Git-and-Deployments.md)
- [فایل ۰۷ — متغیرهای محیطی، دامنه، DNS و SSL](./07-Environment-Variables-and-Domains.md)
