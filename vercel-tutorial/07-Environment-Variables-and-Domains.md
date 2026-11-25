# ۰۷ — متغیرهای محیطی، دامنه، DNS و SSL

**مدت یادگیری**: ۳ تا ۴ ساعت
**سطح سختی**: مبتدی تا متوسط
**پیش‌نیاز**: مطالعه فایل‌های ۰۱ تا ۰۳، داشتن یک پروژهٔ مستقرشده

---

## فهرست محتوا

- [فصل ۱: متغیرهای محیطی — مفهوم و اهمیت](#فصل-۱)
- [فصل ۲: مدیریت متغیرها در Vercel](#فصل-۲)
- [فصل ۳: دامنه اختصاصی، DNS و اتصال](#فصل-۳)
- [فصل ۴: HTTPS و SSL به زبان ساده](#فصل-۴)
- [فصل ۵: دامنه‌های چندگانه، Redirect و Wildcard](#فصل-۵)

---

<a id="فصل-۱"></a>
## فصل ۱: متغیرهای محیطی — مفهوم و اهمیت

### مقدمه

تصور کنید رمز دیتابیس را مستقیم در کد نوشته‌اید. حالا کد را روی GitHub می‌گذارید و... رمز شما برای همه قابل دیدن است! **متغیرهای محیطی (Environment Variables)** دقیقاً برای جلوگیری از همین فاجعه هستند. در این فصل، این مفهوم حیاتی را از ریشه می‌فهمیم.

### مفاهیم اساسی

**۱. تعریف**
**Environment Variable** یک جفت «نام = مقدار» است که بیرون از کد شما ذخیره می‌شود و برنامه در زمان اجرا آن را می‌خواند.

```js
// به‌جای این (❌ خطرناک):
const apiKey = 'sk_live_abc123xyz';

// این را بنویسید (✅ امن):
const apiKey = process.env.STRIPE_SECRET_KEY;
```

**۲. چرا مهم است؟**

| دلیل | توضیح |
|---|---|
| **امنیت** | کلیدها در مخزن کد نیستند |
| **انعطاف** | یک کد، چند محیط (تست/تولید) |
| **همکاری** | هر توسعه‌دهنده کلید خودش را دارد |
| **تغییر بدون Deploy** | مقدار را عوض می‌کنید، بدون تغییر کد |

**۳. سه محیط و متغیرهایشان**

| محیط | کاربرد | فایل محلی |
|---|---|---|
| **Development** | روی کامپیوتر شما | `.env.local` |
| **Preview** | تست هر تغییر | از Dashboard/CLI |
| **Production** | کاربران واقعی | از Dashboard/CLI |

**۴. متغیرهای سیستمی Vercel (خودکار)**
Vercel خودش چند متغیر در اختیار شما می‌گذارد:

| متغیر | مقدار |
|---|---|
| `VERCEL` | `"1"` (نشان می‌دهد روی Vercel اجرا می‌شوید) |
| `VERCEL_ENV` | `production` / `preview` / `development` |
| `VERCEL_URL` | آدرس استقرار فعلی |
| `VERCEL_REGION` | منطقهٔ اجرای تابع |
| `VERCEL_GIT_COMMIT_SHA` | هش Commit |
| `VERCEL_GIT_COMMIT_REF` | نام شاخه |
| `VERCEL_GIT_REPO_SLUG` | نام مخزن |

**۵. متغیرهای عمومی در برابر حساس**
- متغیرهایی که با `NEXT_PUBLIC_` شروع شوند، در **مرورگر** هم در دسترس‌اند.
- بقیه فقط سمت سرور در دسترس‌اند.

```js
process.env.NEXT_PUBLIC_API_URL      // ✅ در مرورگر هم هست
process.env.DATABASE_URL             // 🔒 فقط سرور
```

### مثال‌های عملی

**مثال ۱ — فایل `.env.local` برای توسعه**

```bash
# .env.local  (هرگز در Git Commit نشود!)
DATABASE_URL="postgres://user:pass@localhost:5432/mydb"
STRIPE_SECRET_KEY="sk_test_..."
NEXT_PUBLIC_SITE_URL="http://localhost:3000"
NEXT_PUBLIC_ANALYTICS_ID="dev-analytics"
```

```gitignore
# .gitignore
.env
.env.local
.env.*.local
```

**مثال ۲ — استفاده در کد**

```js
// api/db.js — فقط سمت سرور
export default async function handler(req, res) {
  // DATABASE_URL هرگز به مرورگر نمی‌رود
  const url = process.env.DATABASE_URL;
  if (!url) {
    return res.status(500).json({ error: 'پیکربندی ناقص' });
  }
  // ... اتصال به دیتابیس
  res.json({ ok: true });
}
```

```tsx
// components/Analytics.tsx — در مرورگر
'use client';

export default function Analytics() {
  // NEXT_PUBLIC_ در مرورگر در دسترس است
  const id = process.env.NEXT_PUBLIC_ANALYTICS_ID;
  return <div>شناسه آنالیتیکس: {id}</div>;
}
```

**مثال ۳ — بررسی وجود متغیر در زمان اجرا**

```js
// api/health.js
export default function handler(req, res) {
  const required = ['DATABASE_URL', 'STRIPE_SECRET_KEY', 'AUTH_SECRET'];
  const missing = required.filter((key) => !process.env[key]);

  if (missing.length > 0) {
    return res.status(500).json({
      status: 'unhealthy',
      missing,   // فقط نام‌ها، نه مقادیر!
    });
  }

  res.json({
    status: 'healthy',
    env: process.env.VERCEL_ENV,
    region: process.env.VERCEL_REGION,
  });
}
```

### مقایسه و الگوهای مختلف

**متغیر محیطی در برابر هاردکد**

| معیار | هاردکد در کد | متغیر محیطی |
|---|---|---|
| امنیت | ❌ فاجعه | ✅ |
| تغییر بین محیط‌ها | ❌ نیاز به تغییر کد | ✅ |
| نشت در Git | ❌ بله | ✅ خیر |
| چرخش کلید (Rotation) | ❌ سخت | ✅ آسان |

**سه روش نگهداری متغیرها**

| روش | مناسب برای | امنیت |
|---|---|---|
| `.env.local` | توسعه محلی | ✅ (Git-ignore) |
| Dashboard Vercel | Preview/Production | ✅ (رمزنگاری‌شده) |
| Secret Manager بیرونی | سازمانی | ✅✅ (بالاترین) |

### اشکال شایع

**۱. Commit کردن `.env` در Git.**
بزرگ‌ترین اشتباه. اگر این کار را کردید، **کلیدها را فوراً عوض کنید** (حتی اگر فایل را حذف کنید، در تاریخچه می‌ماند).

**۲. استفاده از `NEXT_PUBLIC_` برای دادهٔ حساس.**
هر چیزی با این پیشوند، در مرورگر قابل دیدن است. هرگز کلید مخفی را با `NEXT_PUBLIC_` نگذارید.

**۳. فراموش کردن تعریف متغیر در Vercel.**
کد محلی کار می‌کند اما روی Vercel خطا می‌دهد.

**۴. استفاده از نام‌های غیراستاندارد.**
نام‌ها را با حروف بزرگ و زیرخط بنویسید: `DATABASE_URL` (نه `databaseUrl`).

**۵. چاپ کردن مقدار متغیر در لاگ.**
هرگز مقدار کامل را لاگ نکنید؛ فقط نام یا وضعیت وجود/عدم وجود.

### نکات مهم (Tips & Tricks)

- **نام‌گذاری استاندارد**: `SCREAMING_SNAKE_CASE`.
- فایل `.env.example` بسازید (با مقادیر نمونه) و در Git بگذارید — راهنمای تیم.
- کلیدها را دوره‌ای **چرخش (Rotate)** دهید.
- از همان ابتدا کلیدهای **تست و تولید** را جدا کنید.
- برای پروژه‌های بزرگ، از **Secret Manager** بیرونی استفاده کنید.

### خلاصه فصل

- متغیرهای محیطی، پیکربندی را از کد جدا می‌کنند.
- سه محیط: **Development، Preview، Production**.
- `NEXT_PUBLIC_` = در مرورگر قابل دیدن (هرگز برای حساس‌ها).
- Vercel متغیرهای سیستمی (مثل `VERCEL_ENV`) را خودکار فراهم می‌کند.
- هرگز `.env` را Commit نکنید.

---

<a id="فصل-۲"></a>
## فصل ۲: مدیریت متغیرها در Vercel

### مقدمه

حالا که مفهوم را می‌دانیم، ببینیم چطور در Vercel متغیرها را بسازیم، ویرایش کنیم و بین محیط‌ها مدیریت کنیم. این کار از سه راه ممکن است: Dashboard، CLI و REST API.

### مفاهیم اساسی

**۱. سه محیط در Vercel**

| محیط | چه زمانی استفاده می‌شود |
|---|---|
| **Production** | استقرارهای Production |
| **Preview** | استقرارهای Preview |
| **Development** | با `vercel dev` روی سیستم شما |

می‌توانید یک متغیر را برای **چند محیط** همزمان تعریف کنید.

**۲. متغیر حساس (Sensitive)**
Vercel اجازه می‌دهد متغیری را «حساس» علامت بزنید. این متغیرها **پس از ساخت قابل خواندن نیستند** — فقط قابل جایگزینی.

**۳. ترتیب اولویت (Precedence)**
اگر متغیری در چند سطح تعریف شده باشد:
```
Project Env Vars  >  Shared (Team) Env Vars  >  Development (محلی)
```

**۴. رمزنگاری**
متغیرها **در حالت سکون رمزنگاری‌شده** ذخیره می‌شوند و هنگام Build/اجرا رمزگشایی می‌شوند.

**۵. متغیرهای Shared (تیمی)**
در پلن‌های بالاتر، می‌توانید متغیرها را در سطح تیم تعریف کنید تا بین چند پروژه مشترک باشند.

### مثال‌های عملی

**مثال ۱ — ساخت متغیر از Dashboard**

```
۱. Project → Settings → Environment Variables
۲. روی «Add New» کلیک کنید
۳. Key را وارد کنید: DATABASE_URL
۴. Value را وارد کنید: postgres://...
۵. Environments را انتخاب کنید: Production, Preview, Development
۶. (اختیاری) «Sensitive» را تیک بزنید
۷. Save
```

**مثال ۲ — مدیریت با CLI**

```bash
# افزودن متغیر برای همه محیط‌ها
vercel env add API_KEY
# → مقدار را وارد کنید (مخفی تایپ می‌شود)
# → محیط‌ها را انتخاب کنید

# افزودن برای محیط مشخص
vercel env add DATABASE_URL production

# افزودن از فایل (مقدار از stdin)
echo "sk_live_abc123" | vercel env add STRIPE_KEY production

# لیست همه متغیرها
vercel env ls

# خروجی نمونه:
# name             value        environments
# DATABASE_URL     Encrypted    Production, Preview
# API_KEY          Encrypted    Production

# حذف یک متغیر
vercel env rm DATABASE_URL production

# دریافت متغیرها به فایل محلی برای توسعه
vercel env pull .env.local
```

**مثال ۳ — متغیر مخصوص هر محیط (مقدار متفاوت)**

```bash
# Production: دیتابیس واقعی
vercel env add DATABASE_URL production
# → postgres://prod-db...

# Preview: دیتابیس تست
vercel env add DATABASE_URL preview
# → postgres://staging-db...

# Development: دیتابیس محلی
vercel env add DATABASE_URL development
# → postgres://localhost:5432/dev
```

**مثال ۴ — استفاده از REST API برای مدیریت متغیرها**

```bash
# لیست متغیرهای پروژه
curl -H "Authorization: Bearer $VERCEL_TOKEN" \
  "https://api.vercel.com/v9/projects/my-app/env?teamId=$TEAM_ID"

# ساخت متغیر جدید
curl -X POST \
  -H "Authorization: Bearer $VERCEL_TOKEN" \
  -H "Content-Type: application/json" \
  "https://api.vercel.com/v10/projects/my-app/env?teamId=$TEAM_ID" \
  -d '{
    "key": "NEW_KEY",
    "value": "new_value",
    "type": "encrypted",
    "target": ["production", "preview"]
  }'
```

### مقایسه و الگوهای مختلف

**سه روش مدیریت متغیرها**

| روش | سرعت | مناسب برای |
|---|---|---|
| **Dashboard** | متوسط | کارهای گاه‌به‌گاه، بررسی بصری |
| **CLI** | ✅ سریع | توسعه‌دهندگان، اسکریپت |
| **REST API** | سریع | اتوماسیون، CI/CD |

**متغیر عادی در برابر حساس**

| معیار | عادی | حساس (Sensitive) |
|---|---|---|
| پس از ساخت قابل خواندن | ✅ | ❌ |
| قابل ویرایش مقدار | ✅ | فقط جایگزینی |
| مناسب برای | کلیدهای عمومی | رمزها، کلیدهای مخفی |

### اشکال شایع

**۱. فراموش کردن محیط Preview.**
Build در Preview خطا می‌دهد چون متغیر تعریف نشده. همهٔ محیط‌های لازم را تیک بزنید.

**۲. تغییر متغیر بدون Redeploy.**
متغیرهای محیطی در **زمان Build** خوانده می‌شوند؛ بعد از تغییر، باید دوباره Deploy کنید.

**۳. استفاده از یک دیتابیس برای Preview و Production.**
خطرناک! ممکن است داده‌های واقعی خراب شوند.

**۴. خواندن مقدار متغیر حساس از Dashboard.**
امکان‌پذیر نیست. اگر مقدار را گم کردید، باید جایگزین کنید.

**۵. لو دادن `VERCEL_TOKEN` در مخزن.**
این توکن دسترسی کامل به پروژه می‌دهد. هرگز آن را Commit نکنید.

### نکات مهم (Tips & Tricks)

- بعد از تغییر هر متغیر، **حتماً Redeploy** کنید.
- برای Preview، از **دیتابیس و کلیدهای جدا** استفاده کنید.
- کلیدهای حساس را **Sensitive** علامت بزنید.
- از `vercel env pull .env.local` برای همگام‌سازی سریع استفاده کنید.
- در CI/CD، توکن را از **Secret** های پلتفرم CI بخوانید (نه در کد).

### خلاصه فصل

- متغیرها در سه محیط **Production / Preview / Development** مدیریت می‌شوند.
- سه راه: **Dashboard، CLI، REST API**.
- متغیر **حساس** پس از ساخت قابل خواندن نیست.
- تغییر متغیر نیازمند **Redeploy** است.
- Preview باید دیتابیس و کلید جدا داشته باشد.

---

<a id="فصل-۳"></a>
## فصل ۳: دامنه اختصاصی، DNS و اتصال

### مقدمه

آدرس `my-app.vercel.app` برای شروع خوب است، اما برای یک پروژهٔ واقعی به دامنهٔ اختصاصی (مثل `myapp.com`) نیاز دارید. در این فصل، دامنه را از صفر وصل می‌کنیم و DNS را به زبان ساده می‌فهمیم.

### مفاهیم اساسی

**۱. DNS چیست؟**
**DNS** (Domain Name System) مثل «دفترچه تلفن اینترنت» است: نام دامنه را به آدرس IP تبدیل می‌کند.

> **تشبیه**: شما نام «علی» را می‌دانید اما شماره‌اش را نه. DNS دفترچه‌ای است که نام را به شماره (IP) تبدیل می‌کند.

**۲. انواع رکوردهای DNS**

| رکورد | کار | مثال |
|---|---|---|
| **A** | نام → آدرس IPv4 | `example.com → 76.76.21.21` |
| **AAAA** | نام → آدرس IPv6 | `example.com → 2606:4700::...` |
| **CNAME** | نام → نام دیگر | `www → cname.vercel-dns.com` |
| **TXT** | متن (برای تأیید مالکیت) | `_vercel → ...` |
| **MX** | ایمیل | `@ → mail.example.com` |
| **NS** | سرورهای نام | `ns1.vercel-dns.com` |

**۳. دامنهٔ ریشه در برابر زیردامنه**

| نوع | مثال | رکورد مناسب |
|---|---|---|
| **Apex (ریشه)** | `example.com` | A یا ALIAS/ANAME |
| **Subdomain** | `www.example.com` | CNAME |
| **Wildcard** | `*.example.com` | CNAME (نیاز به پلن Pro) |

**۴. دو روش اتصال DNS**

| روش | توضیح | مزیت |
|---|---|---|
| **Vercel Nameservers** | DNS را کامل به Vercel بسپارید | ساده‌ترین، مدیریت کامل |
| **External DNS** | رکوردها را در ثبت‌کنندهٔ فعلی تنظیم کنید | کنترل بیشتر، ایمیل فعلی حفظ می‌شود |

**۵. مقادیر DNS استاندارد Vercel**

| رکورد | نام | مقدار |
|---|---|---|
| **A** | `@` | `76.76.21.21` |
| **CNAME** | `www` | `cname.vercel-dns.com` |
| **NS** | `@` | `ns1.vercel-dns.com` و `ns2.vercel-dns.com` |

> مقادیر ممکن است تغییر کنند؛ همیشه مقادیر دقیق را از Dashboard پروژه کپی کنید.

### مثال‌های عملی

**مثال ۱ — افزودن دامنه از Dashboard**

```
۱. Project → Settings → Domains
۲. دامنه را وارد کنید: myapp.com
۳. روی «Add» کلیک کنید
۴. Vercel می‌گوید چه رکوردی تنظیم کنید:
   - A: 76.76.21.21  (برای apex)
   - CNAME: cname.vercel-dns.com  (برای www)
۵. رکوردها را در ثبت‌کنندهٔ دامنه تنظیم کنید
۶. منتظر بمانید تا DNS منتشر شود (چند دقیقه تا چند ساعت)
۷. وضعیت «Valid Configuration» شود ✅
```

**مثال ۲ — تنظیم رکوردها در پنل ثبت‌کننده**

```
【 برای دامنهٔ ریشه (myapp.com) 】
Type: A
Name: @
Value: 76.76.21.21
TTL: 3600 (یا Automatic)

【 برای www 】
Type: CNAME
Name: www
Value: cname.vercel-dns.com
TTL: 3600
```

**مثال ۳ — بررسی انتشار DNS**

```bash
# بررسی رکورد A
nslookup myapp.com
# یا
dig myapp.com +short

# بررسی CNAME
dig www.myapp.com CNAME +short

# بررسی NS
dig myapp.com NS +short

# در Windows
nslookup -type=A myapp.com
nslookup -type=CNAME www.myapp.com
```

**مثال ۴ — افزودن دامنه با CLI**

```bash
# افزودن دامنه
vercel domains add myapp.com my-app

# لیست دامنه‌ها
vercel domains ls

# بررسی وضعیت
vercel domains inspect myapp.com

# حذف دامنه
vercel domains rm myapp.com
```

### مقایسه و الگوهای مختلف

**Vercel Nameservers در برابر External DNS**

| معیار | Vercel NS | External DNS |
|---|---|---|
| راه‌اندازی | بسیار ساده | متوسط |
| مدیریت رکورد | در Vercel | در ثبت‌کننده |
| حفظ ایمیل فعلی | نیاز به تنظیم MX در Vercel | ✅ ساده |
| سرعت انتشار | سریع | بستگی به TTL |
| کنترل | کمتر | بیشتر |

**apex در برابر www**

| معیار | apex (`myapp.com`) | www (`www.myapp.com`) |
|---|---|---|
| رکورد | A / ALIAS | CNAME |
| توصیه | هدایت به www | ✅ میزبانی اصلی |
| مشکل رایج | CNAME در apex مجاز نیست | — |

> **بهترین شیوه**: `www` را میزبان اصلی کنید و `myapp.com` را به آن ریدایرکت کنید (یا برعکس، مهم یکسان بودن است).

### اشکال شایع

**۱. تنظیم CNAME روی apex.**
استاندارد DNS اجازه نمی‌دهد. برای apex از رکورد A استفاده کنید.

**۲. تداخل رکوردهای قبلی.**
اگر رکورد A یا CNAME قدیمی وجود دارد، باید حذف شود.

**۳. انتظار فوری انتشار DNS.**
DNS ممکن است تا چند ساعت طول بکشد (بر اساس TTL).

**۴. فراموش کردن `www`.**
کاربران هم `myapp.com` و هم `www.myapp.com` را امتحان می‌کنند. هر دو را تنظیم کنید.

**۵. اشتباه در نام رکورد.**
بعضی پنل‌ها `@` و بعضی نام دامنه را می‌خواهند. راهنمای پنل خود را بخوانید.

### نکات مهم (Tips & Tricks)

- **TTL را کم بگذارید** (۳۰۰ ثانیه) تا تغییرات سریع‌تر منتشر شوند.
- هم `apex` و هم `www` را تنظیم و یکی را به دیگری ریدایرکت کنید.
- قبل از تغییر NS، **رکوردهای ایمیل (MX) را یادداشت کنید**.
- از ابزارهای آنلاین (مثل `dnschecker.org`) برای بررسی انتشار جهانی استفاده کنید.
- اگر از ایمیل روی دامنه استفاده می‌کنید، External DNS انتخاب بهتری است.

### خلاصه فصل

- **DNS** نام دامنه را به IP تبدیل می‌کند.
- رکوردهای مهم: **A، AAAA، CNAME، TXT، MX، NS**.
- apex با **A** و www با **CNAME** تنظیم می‌شود.
- دو روش: **Vercel NS** (ساده) یا **External DNS** (کنترل بیشتر).
- انتشار DNS ممکن است زمان ببرد.

---

<a id="فصل-۴"></a>
## فصل ۴: HTTPS و SSL به زبان ساده

### مقدمه

قفل سبز کنار آدرس سایت و `https://` نشانهٔ امنیت است. در Vercel این کار **خودکار** انجام می‌شود — نیازی به خرید یا نصب گواهی ندارید. در این فصل می‌فهمیم چطور کار می‌کند.

### مفاهیم اساسی

**۱. HTTP در برابر HTTPS**

| معیار | HTTP | HTTPS |
|---|---|---|
| رمزنگاری | ❌ | ✅ |
| قفل مرورگر | ❌ | ✅ |
| SEO | ضعیف‌تر | ✅ بهتر |
| اعتماد کاربر | کم | زیاد |
| پورت | 80 | 443 |

**۲. SSL/TLS چیست؟**
پروتکل رمزنگاری که ارتباط بین مرورگر و سرور را امن می‌کند. **SSL** نام قدیمی و **TLS** نام جدیدتر است (اما هنوز «SSL» گفته می‌شود).

**۳. گواهی (Certificate)**
یک فایل دیجیتال که هویت سرور را تأیید می‌کند. در Vercel:
- گواهی‌ها از **Let's Encrypt** صادر می‌شوند.
- **خودکار** صادر و **خودکار** تمدید می‌شوند.
- **رایگان** هستند.

**۴. چطور کار می‌کند؟**
```
مرورگر → درخواست HTTPS
   ↓
سرور → ارسال گواهی
   ↓
مرورگر → تأیید گواهی (معتبر؟ منقضی نشده؟ دامنه درست؟)
   ↓
تبادل کلید رمزنگاری
   ↓
ارتباط امن ✅
```

**۵. HSTS**
هدری که به مرورگر می‌گوید همیشه از HTTPS استفاده کند (حتی اگر کاربر `http://` بزند).

**۶. Certificate Order**
برای دامنه‌های خاص یا Wildcard، ممکن است به گواهی سفارشی نیاز داشته باشید (پلن‌های بالاتر).

### مثال‌های عملی

**مثال ۱ — HTTPS خودکار (هیچ کاری لازم نیست)**

```
۱. دامنه را اضافه کنید
۲. Vercel خودکار گواهی SSL صادر می‌کند
۳. چند دقیقه بعد: https://myapp.com ✅
۴. تمدید خودکار هر ۹۰ روز
```

**مثال ۲ — بررسی گواهی سایت**

```bash
# بررسی گواهی SSL
openssl s_client -connect myapp.com:443 -servername myapp.com </dev/null 2>/dev/null \
  | openssl x509 -noout -dates -subject

# خروجی نمونه:
# notBefore=...
# notAfter=... (تاریخ انقضا)
# subject=CN = myapp.com

# بررسی هدرهای امنیتی
curl -I https://myapp.com
```

**مثال ۳ — فعال‌سازی HSTS در `next.config.js`**

```js
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'Strict-Transport-Security',
            value: 'max-age=63072000; includeSubDomains; preload',
          },
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'X-Frame-Options', value: 'SAMEORIGIN' },
          { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
        ],
      },
    ];
  },
};

module.exports = nextConfig;
```

**مثال ۴ — ریدایرکت HTTP به HTTPS در `vercel.json`**

```json
{
  "redirects": [
    {
      "source": "/(.*)",
      "has": [{ "type": "header", "key": "x-forwarded-proto", "value": "http" }],
      "destination": "https://myapp.com/$1",
      "permanent": true
    }
  ]
}
```

> در Vercel، ریدایرکت HTTP→HTTPS **به‌طور پیش‌فرض** فعال است؛ این مثال برای موارد خاص است.

### مقایسه و الگوهای مختلف

**انواع گواهی SSL**

| نوع | اعتبارسنجی | مناسب برای |
|---|---|---|
| **DV** (Domain Validated) | فقط مالکیت دامنه | اکثر سایت‌ها (Vercel پیش‌فرض) |
| **OV** (Organization Validated) | مالکیت + سازمان | شرکت‌ها |
| **EV** (Extended Validated) | بررسی کامل | بانک‌ها، سازمان‌های بزرگ |
| **Wildcard** | `*.example.com` | زیردامنه‌های زیاد |

**مدیریت SSL در Vercel در برابر سنتی**

| معیار | Vercel | سنتی |
|---|---|---|
| صدور | خودکار | دستی (Let's Encrypt/cPanel) |
| تمدید | خودکار | دستی/کرون |
| هزینه | رایگان | متغیر |
| پشتیبانی Wildcard | ✅ (پلن Pro) | بستگی به ارائه‌دهنده |
| Certificate Order | ✅ | ❌ |

### اشکال شایع

**۱. خطای «Mixed Content».**
صفحه HTTPS است اما منابعی (تصویر/اسکریپت) از HTTP لود می‌شوند. همهٔ منابع را HTTPS کنید.

**۲. عدم تطابق دامنه در گواهی.**
اگر `www` را تنظیم نکرده باشید، گواهی برای آن صادر نمی‌شود. همیشه همهٔ زیردامنه‌ها را اضافه کنید.

**۳. انتظار صدور فوری گواهی.**
پس از تأیید DNS، صدور چند دقیقه طول می‌کشد.

**۴. تنظیم HSTS با `max-age` خیلی بالا در ابتدا.**
اگر اشتباهی HTTPS را قطع کنید، کاربران تا مدت زیادی نمی‌توانند وصل شوند. ابتدا با `max-age` کم شروع کنید.

**۵. اشتباه در ریدایرکت.**
حلقهٔ ریدایرکت HTTP↔HTTPS. مسیرهای استثنا را در نظر بگیرید.

### نکات مهم (Tips & Tricks)

- در Vercel **هیچ کاری برای SSL لازم نیست** — خودکار است.
- هدر **HSTS** را بعد از اطمینان از پایداری HTTPS فعال کنید.
- همیشه برای `www` و `apex` گواهی داشته باشید.
- خطای **Mixed Content** را با کنسول مرورگر پیدا کنید.
- از ابزارهایی مثل `ssllabs.com` برای بررسی کیفیت گواهی استفاده کنید.

### خلاصه فصل

- Vercel گواهی **SSL/TLS** را خودکار و رایگان از **Let's Encrypt** صادر می‌کند.
- تمدید خودکار است؛ نیازی به مدیریت دستی نیست.
- **HSTS** ارتباط را همیشه امن نگه می‌دارد.
- خطای **Mixed Content** را جدی بگیرید.
- ریدایرکت HTTP→HTTPS به‌طور پیش‌فرض فعال است.

---

<a id="فصل-۵"></a>
## فصل ۵: دامنه‌های چندگانه، Redirect و Wildcard

### مقدمه

یک پروژه می‌تواند چند دامنه داشته باشد: دامنهٔ اصلی، زیردامنه‌ها، دامنه‌های قدیمی و دامنه‌های منطقه‌ای. در این فصل، مدیریت این‌ها و تنظیم ریدایرکت‌های حرفه‌ای را یاد می‌گیریم.

### مفاهیم اساسی

**۱. انواع دامنه در یک پروژه**

| نوع | مثال | کاربرد |
|---|---|---|
| **Production Domain** | `myapp.com` | دامنهٔ اصلی |
| **Redirect Domain** | `old.com` → `myapp.com` | انتقال |
| **Subdomain** | `docs.myapp.com` | بخش‌های جدا |
| **Wildcard** | `*.myapp.com` | زیردامنه‌های پویا |
| **Regional** | `ir.myapp.com` | نسخهٔ منطقه‌ای |

**۲. Redirect (ریدایرکت) در سطح دامنه**
وقتی دامنه‌ای را به‌عنوان Redirect تنظیم کنید، همهٔ درخواست‌ها به دامنهٔ اصلی می‌روند.

**۳. Wildcard Domain**
`*.myapp.com` همهٔ زیردامنه‌ها را می‌گیرد. نیاز به:
- پلن **Pro** یا بالاتر.
- رکورد DNS از نوع Wildcard CNAME.

**۴. ریدایرکت در `vercel.json`**

| نوع | کد | توضیح |
|---|---|---|
| **permanent** | ۳۰۸ | دائمی (SEO-friendly) |
| **موقت** | ۳۰۷ | موقت |

**۵. Rewrite در برابر Redirect**

| معیار | Redirect | Rewrite |
|---|---|---|
| URL مرورگر | تغییر می‌کند | تغییر نمی‌کند |
| مناسب برای | انتقال دائمی | نمایش محتوای دیگر |

### مثال‌های عملی

**مثال ۱ — ریدایرکت دامنهٔ قدیمی به جدید**

```
۱. Settings → Domains
۲. دامنهٔ قدیمی (old.com) را اضافه کنید
۳. در تنظیمات دامنه، گزینه «Redirect to» را انتخاب کنید
۴. دامنهٔ مقصد: myapp.com
۵. ذخیره — همهٔ درخواست‌ها به دامنهٔ جدید می‌روند
```

**مثال ۲ — ریدایرکت `apex` به `www` در `vercel.json`**

```json
{
  "redirects": [
    {
      "source": "/:path*",
      "has": [{ "type": "host", "value": "myapp.com" }],
      "destination": "https://www.myapp.com/:path*",
      "permanent": true
    }
  ]
}
```

**مثال ۳ — ریدایرکت مسیرهای قدیمی**

```json
{
  "redirects": [
    { "source": "/blog/old-post", "destination": "/blog/new-post", "permanent": true },
    { "source": "/products/:id", "destination": "/shop/:id", "permanent": true },
    { "source": "/promo", "destination": "/pricing", "permanent": false }
  ]
}
```

**مثال ۴ — ریدایرکت شرطی بر اساس کشور**

```json
{
  "redirects": [
    {
      "source": "/pricing",
      "has": [{ "type": "header", "key": "x-vercel-ip-country", "value": "IR" }],
      "destination": "/fa/pricing",
      "permanent": false
    }
  ]
}
```

**مثال ۵ — استفاده از زیردامنه برای سرویس‌های مختلف**

```
myapp.com              → اپ اصلی (Next.js)
www.myapp.com          → ریدایرکت به myapp.com
docs.myapp.com         → مستندات (پروژه جدا)
api.myapp.com          → API (پروژه جدا)
*.myapp.com            → Wildcard (برای زیردامنه‌های کاربران)
```

### مقایسه و الگوهای مختلف

**سه روش مدیریت چند دامنه**

| روش | مناسب برای | پیچیدگی |
|---|---|---|
| **Redirect Domain** | انتقال دامنهٔ قدیمی | کم |
| **Subdomain (پروژه جدا)** | سرویس‌های مستقل | متوسط |
| **Subdomain (rewrite)** | یک اپ، چند دامنه | متوسط |
| **Wildcard** | زیردامنه‌های پویا | بالا (پلن Pro) |

**نوع ریدایرکت**

| کد | نام | SEO | کش |
|---|---|---|---|
| 301/308 | دائمی | ✅ منتقل می‌شود | ✅ |
| 302/307 | موقت | ❌ | ❌ |

### اشکال شایع

**۱. حلقهٔ ریدایرکت.**
اگر `apex` را به `www` و `www` را به `apex` ریدایرکت کنید، حلقه می‌سازید. جهت را یک‌طرفه کنید.

**۲. استفاده از ریدایرکت موقت برای انتقال دائمی.**
SEO منتقل نمی‌شود. برای انتقال دائمی از **308/301** استفاده کنید.

**۳. Wildcard بدون پلن مناسب.**
در پلن Hobby کار نمی‌کند و خطا می‌دهد.

**۴. فراموش کردن زیردامنه‌های مهم.**
اگر `api` یا `docs` را تنظیم نکنید، ممکن است به دامنهٔ اشتباه بروند.

**۵. تنظیم نشدن DNS برای زیردامنه.**
دامنه در Vercel اضافه شده اما رکورد DNS ست نشده → خطای «Invalid Configuration».

### نکات مهم (Tips & Tricks)

- همیشه **جهت ریدایرکت را یک‌طرفه** نگه دارید.
- برای انتقال دائمی از **308** استفاده کنید.
- زیردامنه‌های مستقل را **پروژهٔ جدا** بسازید (جدایی بهتر).
- Wildcard نیازمند **پلن Pro** است.
- قبل از حذف دامنهٔ قدیمی، مطمئن شوید ریدایرکت کار می‌کند.

### خلاصه فصل

- یک پروژه می‌تواند چند دامنه و زیردامنه داشته باشد.
- **Redirect Domain** برای انتقال دامنه‌های قدیمی.
- **Wildcard** نیازمند پلن Pro است.
- ریدایرکت دائمی = **308/301**، موقت = **307/302**.
- جهت ریدایرکت را یک‌طرفه نگه دارید تا حلقه نسازید.

---

## منابع و مراجع

### مستندات رسمی
- Environment Variables: <https://vercel.com/docs/environment-variables>
- Managing Env Vars: <https://vercel.com/docs/environment-variables/managing-environment-variables>
- Domains Overview: <https://vercel.com/docs/domains/working-with-domains>
- DNS Records: <https://vercel.com/docs/domains/working-with-domains/add-a-domain>
- SSL / Certificates: <https://vercel.com/docs/domains/working-with-domains/manage-domains>
- Redirects & Rewrites: <https://vercel.com/docs/edge-network/redirects>

### مفاهیم پیشنهادی برای مطالعه بیشتر
- DNS (Cloudflare Learning): <https://www.cloudflare.com/learning/dns/>
- Let's Encrypt: <https://letsencrypt.org>
- HSTS (MDN): <https://developer.mozilla.org/docs/Web/HTTP/Headers/Strict-Transport-Security>
- SSL Labs Test: <https://www.ssllabs.com/ssltest/>

### گام بعدی
- [فایل ۰۸ — سرویس‌های داده و ذخیره‌سازی](./08-Storage-Services.md)
- [فایل ۱۲ — امنیت](./12-Security.md)
