# ۱۶ — عیب‌یابی خطاهای رایج

**مدت یادگیری**: ۳ تا ۴ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: مطالعه فایل‌های ۰۲ تا ۰۷

---

## فهرست محتوا

- [فصل ۱: روش‌شناسی دیباگ در Vercel](#فصل-۱)
- [فصل ۲: Build Fail و خطاهای ساخت](#فصل-۲)
- [فصل ۳: خطاهای DNS و دامنه](#فصل-۳)
- [فصل ۴: CORS و مشکلات درخواست](#فصل-۴)
- [فصل ۵: Timeout و Cold Start](#فصل-۵)
- [فصل ۶: خطاهای رایج دیگر](#فصل-۶)

---

<a id="فصل-۱"></a>
## فصل ۱: روش‌شناسی دیباگ در Vercel

### مقدمه

دیباگ مؤثر یک **فرایند** است، نه جست‌وجوی تصادفی. در این فصل، یک روش نظام‌مند یاد می‌گیریم که در هر خطایی به کار می‌آید.

### مفاهیم اساسی

**۱. چهار سؤال کلیدی**

| سؤال | ابزار |
|---|---|
| **کجا** خطا رخ داد؟ (Build یا Runtime) | Build/Runtime Logs |
| **چه زمانی** شروع شد؟ | Deployments |
| **چه تغییری** قبلش دادم؟ | Git History |
| **چه کسی** تحت تأثیر است؟ | Analytics، Speed Insights |

**۲. محل‌های بررسی**

```
۱. Build Logs       → خطاهای ساخت
۲. Runtime Logs     → خطاهای اجرا
۳. Deployments      → تاریخچه و وضعیت
۴. Analytics        → الگوی ترافیک
۵. Speed Insights   → عملکرد
۶. Firewall Logs    → مسدودسازی‌ها
```

**۳. انواع خطا بر اساس کد وضعیت**

| کد | دسته | معنی |
|---|---|---|
| **4xx** | خطای کلاینت | درخواست اشتباه |
| **5xx** | خطای سرور | مشکل در کد/زیرساخت |

**۴. تفکیک Build و Runtime**

| نوع | نشانه | محل بررسی |
|---|---|---|
| **Build Fail** | استقرار Error، سایت به‌روز نمی‌شود | Build Logs |
| **Runtime Error** | سایت بالا اما خطا در عملکرد | Runtime Logs |

**۵. اصل تغییر تک‌متغیره**
در هر آزمایش فقط **یک چیز** را تغییر دهید تا بدانید چه چیزی مؤثر بوده.

### مثال‌های عملی

**مثال ۱ — فرایند دیباگ گام‌به‌گام**

```
【 مرحله ۱: تعیین نوع 】
استقرار Error خورد؟ → Build Logs
سایت بالا اما خطا می‌دهد؟ → Runtime Logs

【 مرحله ۲: خواندن لاگ 】
- آخرین خط خطا را بخوانید (نه اولی)
- به دنبال "Error"، "Failed"، "Cannot find" بگردید

【 مرحله ۳: بازتولید محلی 】
- همان نسخه را محلی Build کنید
- اگر محلی موفق است، تفاوت محیط را بررسی کنید

【 مرحله ۴: آزمایش فرضیه 】
- یک تغییر، یک آزمایش

【 مرحله ۵: رفع و تأیید 】
- Deploy و بررسی
```

**مثال ۲ — دستورات تشخیص سریع**

```bash
# وضعیت استقرارها
vercel ls

# لاگ استقرار خاص
vercel logs <deployment-url>

# بازتولید Build محلی
npm run build

# بررسی وابستگی‌ها
npm ls --depth=0
npm audit

# بررسی نسخه‌ها
node -v && npm -v
```

**مثال ۳ — جدول تشخیص سریع**

| نشانه | احتمال اول | بررسی |
|---|---|---|
| Build Fail | وابستگی/تایپ | Build Logs |
| سایت به‌روز نمی‌شود | Deploy نشده | Deployments |
| ۵۰۰ | باگ کد | Runtime Logs |
| ۴۰۴ | مسیر/rewrite | Build Logs |
| ۴۰۱ | auth | Runtime Logs |
| CORS | هدر | Network tab |
| کندی | Cold Start/دیتابیس | Speed Insights |

### مقایسه و الگوهای مختلف

**دو رویکرد دیباگ**

| رویکرد | مزیت | عیب |
|---|---|---|
| **از خطا به عقب** | سریع | ممکن است ریشه را از دست بدهد |
| **از تغییرات به جلو** | دقیق | کندتر |

**محلی در برابر ابری**

| معیار | محلی | ابری |
|---|---|---|
| سرعت تکرار | ✅ سریع | کندتر |
| تطابق محیط | ⚠️ متفاوت | ✅ دقیق |
| مناسب برای | خطاهای منطقی | خطاهای محیطی |

### اشکال شایع

**۱. تغییر چند چیز با هم.**
نمی‌دانید کدام مؤثر بود.

**۲. خواندن اولین خط خطا.**
معمولاً خطای اصلی در **آخرین** خطوط است.

**۳. نادیده گرفتن تفاوت محیط محلی/ابری.**
نسخهٔ Node، متغیرها، پلتفرم.

**۴. جست‌وجوی تصادفی در لاگ.**
با فیلتر و الگو جست‌وجو کنید.

**۵. نبود بازتولید محلی.**
بدون بازتولید، دیباگ کند است.

### نکات مهم (Tips & Tricks)

- **اول نوع خطا را تعیین کنید** (Build یا Runtime).
- **آخرین خط** لاگ را بخوانید.
- محلی **بازتولید** کنید.
- **یک تغییر** در هر آزمایش.
- تغییرات اخیر Git را مرور کنید.

### خلاصه فصل

- چهار سؤال: **کجا، چه زمانی، چه تغییری، چه کسی**.
- ابزارها: Build/Runtime Logs، Deployments، Analytics.
- **Build Fail** در Build Logs، **Runtime Error** در Runtime Logs.
- اصل **تغییر تک‌متغیره**.
- بازتولید محلی سرعت دیباگ را بالا می‌برد.

---

<a id="فصل-۲"></a>
## فصل ۲: Build Fail و خطاهای ساخت

### مقدمه

**Build Fail** رایج‌ترین خطای Vercel است و خوشبختانه بیشترین راه‌حل مشخص را دارد. در این فصل، همهٔ انواع را پوشش می‌دهیم.

### مفاهیم اساسی

**۱. چرخهٔ Build و محل خطا**

```
Clone → Install → Build → Collect Output → Deploy
          ↑         ↑            ↑
       خطای npm   خطای کامپایل  خطای خروجی
```

**۲. انواع خطای Build**

| نوع | پیام نمونه |
|---|---|
| **Module not found** | `Cannot find module 'x'` |
| **Type error** | `Type 'X' is not assignable to 'Y'` |
| **Lint error** | `ESLint found errors` |
| **Command failed** | `Command "npm run build" exited with 1` |
| **No Output** | `No Output Directory named 'dist' found` |
| **Memory** | `JavaScript heap out of memory` |

**۳. دلایل رایج**

| دلیل | راه‌حل |
|---|---|
| نسخهٔ Node متفاوت | پین کردن `engines` |
| فایل Commit نشده | `git status` |
| متغیر محیطی غایب | تعریف در Vercel |
| وابستگی ناسازگار | پاک‌سازی lock file |
| حساس به حروف | بررسی مسیر ایمپورت |

### مثال‌های عملی

**مثال ۱ — خطای «Module not found»**

```bash
# پیام:
# Module not found: Can't resolve './components/Header'

# دلایل:
۱. حساسیت به حروف (Linux vs Windows)
   ❌ import Header from './components/header'  (فایل: Header.tsx)
   ✅ import Header from './components/Header'

۲. فایل Commit نشده
   git status   # فایل در لیست untracked؟
   git add . && git commit -m "add missing file"

۳. مسیر اشتباه
   ✅ از alias استفاده کنید: '@/components/Header'
```

**مثال ۲ — خطای Type در TypeScript**

```tsx
// ❌ خطا
function greet(name: string) {
  return 'سلام ' + name;
}
greet(123);   // Type 'number' is not assignable to 'string'

// ✅ رفع
greet('علی');

// یا اگر واقعاً عدد ممکن است:
function greet(name: string | number) {
  return 'سلام ' + String(name);
}
```

**مثال ۳ — خطای نسخهٔ Node**

```json
// package.json — پین کردن نسخه
{
  "engines": {
    "node": "20.x"
  }
}
```

```
# یا از Dashboard:
Project Settings → General → Node.js Version → 20.x
```

**مثال ۴ — خطای Output Directory**

```json
// vercel.json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist"
}
```

```
# بررسی کنید خروجی واقعاً کجاست:
ls dist/        # یا
ls .next/       # یا
ls build/
```

**مثال ۵ — پاک‌سازی وابستگی‌ها**

```bash
# حذف و بازنصب تمیز
rm -rf node_modules package-lock.json
npm install
npm run build
```

**مثال ۶ — خطای حافظه در Build**

```json
// package.json
{
  "scripts": {
    "build": "NODE_OPTIONS=--max-old-space-size=4096 next build"
  }
}
```

### مقایسه و الگوهای مختلف

**سه رویکرد رفع خطای Build**

| رویکرد | مزیت | مناسب برای |
|---|---|---|
| **بازتولید محلی** | ✅ سریع | اکثر خطاها |
| **بررسی Build Logs** | دقیق | خطاهای محیطی |
| **بازگشت به نسخهٔ سالم** | فوری | بحران |

**محلی موفق، ابری ناموفق**

| تفاوت احتمالی | بررسی |
|---|---|
| نسخهٔ Node | `engines` |
| متغیر محیطی | Vercel Settings |
| فایل Commit نشده | `git status` |
| حساسیت به حروف | مسیر ایمپورت |
| Cache | Rebuild بدون Cache |

### اشکال شایع

**۱. Commit نکردن فایل جدید.**
محلی کار می‌کند، ابری نه. `git status` را چک کنید.

**۲. حساسیت به حروف.**
Windows حساس نیست، Linux هست. مسیرها را دقیق بنویسید.

**۳. نبود متغیر محیطی.**
خطا در زمان Build (نه اجرا). متغیر را تعریف کنید.

**۴. نسخهٔ Node متفاوت.**
`engines` را در `package.json` پین کنید.

**۵. Cache معیوب.**
با گزینهٔ «Redeploy without cache» دوباره بسازید.

### نکات مهم (Tips & Tricks)

- **همیشه محلی Build بگیرید** قبل از Push.
- نسخهٔ Node را **پین** کنید.
- مسیر ایمپورت را **دقیق** بنویسید.
- برای خطای عجیب، **بدون Cache** دوباره Deploy کنید.
- `git status` را قبل از Commit چک کنید.

### خلاصه فصل

- چرخهٔ Build: **Clone → Install → Build → Output → Deploy**.
- خطاهای رایج: Module not found، Type، Lint، Command failed، Output.
- **محلی موفق، ابری ناموفق** = تفاوت محیط یا فایل Commit‌نشده.
- نسخهٔ Node را پین کنید.
- برای خطای عجیب، Cache را پاک کنید.

---

<a id="فصل-۳"></a>
## فصل ۳: خطاهای DNS و دامنه

### مقدمه

دامنه وصل نمی‌شود؟ «Invalid Configuration» می‌بینید؟ در این فصل، همهٔ خطاهای DNS و دامنه را حل می‌کنیم.

### مفاهیم اساسی

**۱. خطاهای رایج دامنه**

| پیام | معنی |
|---|---|
| **Invalid Configuration** | رکورد DNS اشتباه یا غایب |
| **Domain not found** | دامنه اضافه نشده |
| **Certificate error** | گواهی صادر نشده |
| **Too many redirects** | حلقهٔ ریدایرکت |
| **DNS_PROBE_FINISHED_NXDOMAIN** | رکورد پیدا نشد |

**۲. رکوردهای درست**

| دامنه | رکورد | مقدار |
|---|---|---|
| apex (`myapp.com`) | A | `76.76.21.21` |
| www | CNAME | `cname.vercel-dns.com` |
| Wildcard | CNAME | `cname.vercel-dns.com` |

**۳. مراحل انتشار DNS**

```
تغییر رکورد → انتشار در سرورهای نام → کش جهانی → در دسترس
(دقیقه تا ۴۸ ساعت، بر اساس TTL)
```

**۴. ابزارهای بررسی**

| ابزار | کاربرد |
|---|---|
| `dig` / `nslookup` | بررسی رکورد |
| `dnschecker.org` | انتشار جهانی |
| `whatsmydns.net` | انتشار جهانی |

**۵. SSL خودکار**
پس از تأیید DNS، گواهی خودکار صادر می‌شود (چند دقیقه).

### مثال‌های عملی

**مثال ۱ — بررسی رکوردها**

```bash
# رکورد A
dig myapp.com A +short
# انتظار: 76.76.21.21

# رکورد CNAME
dig www.myapp.com CNAME +short
# انتظار: cname.vercel-dns.com.

# سرورهای نام
dig myapp.com NS +short

# در Windows
nslookup myapp.com
nslookup -type=CNAME www.myapp.com
```

**مثال ۲ — رفع «Invalid Configuration»**

```
【 بررسی ۱: رکورد درست؟ 】
- apex → A record: 76.76.21.21
- www  → CNAME: cname.vercel-dns.com

【 بررسی ۲: رکورد قدیمی؟ 】
- رکورد A یا CNAME قبلی را حذف کنید
- رکوردهای متناقض باعث خطا می‌شوند

【 بررسی ۳: پراکسی؟ 】
- اگر Cloudflare دارید، Proxy را خاموش کنید (DNS only)
- ابر نارنجی → خاکستری

【 بررسی ۴: منتظر بمانید 】
- انتشار DNS ممکن است زمان ببرد
- TTL را کم بگذارید (۳۰۰)
```

**مثال ۳ — رفع حلقهٔ ریدایرکت**

```json
// ❌ اشتباه: حلقه
// apex → www  و  www → apex

// ✅ درست: یک‌طرفه
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

**مثال ۴ — بررسی گواهی SSL**

```bash
# بررسی تاریخ انقضا و دامنه
echo | openssl s_client -connect myapp.com:443 -servername myapp.com 2>/dev/null \
  | openssl x509 -noout -dates -subject

# بررسی هدرها
curl -I https://myapp.com
```

**مثال ۵ — پاک کردن Cache DNS محلی**

```bash
# Windows
ipconfig /flushdns

# macOS
sudo dscacheutil -flushcache

# Linux
sudo systemd-resolve --flush-caches
```

### مقایسه و الگوهای مختلف

**دو روش اتصال DNS**

| روش | راه‌اندازی | کنترل |
|---|---|---|
| **Vercel Nameservers** | ساده | کمتر |
| **External DNS** | متوسط | ✅ بیشتر |

**خطاهای دامنه و راه‌حل**

| خطا | راه‌حل |
|---|---|
| Invalid Config | رکورد درست + حذف قدیمی |
| Certificate | تأیید DNS + صبر |
| Too many redirects | جهت یک‌طرفه |
| NXDOMAIN | رکورد اضافه کنید |

### اشکال شایع

**۱. CNAME روی apex.**
استاندارد DNS اجازه نمی‌دهد. از A استفاده کنید.

**۲. رکورد قدیمی باقی‌مانده.**
باعث تناقض می‌شود. حذف کنید.

**۳. Cloudflare Proxy روشن.**
با Vercel تناقض دارد. «DNS only» کنید.

**۴. انتظار فوری انتشار.**
TTL و انتشار زمان می‌برد.

**۵. فراموش کردن `www`.**
هم `apex` و هم `www` را تنظیم کنید.

### نکات مهم (Tips & Tricks)

- **TTL کم** (۳۰۰) بگذارید.
- رکوردهای **قدیمی را حذف** کنید.
- Cloudflare Proxy را **خاموش** کنید.
- با `dig` تأیید کنید.
- **هر دو** apex و www را تنظیم کنید.

### خلاصه فصل

- خطاهای رایج: **Invalid Config، Certificate، Redirect Loop، NXDOMAIN**.
- apex با **A**، www با **CNAME**.
- رکوردهای قدیمی را حذف کنید.
- Cloudflare Proxy را خاموش کنید.
- انتشار DNS زمان می‌برد.

---

<a id="فصل-۴"></a>
## فصل ۴: CORS و مشکلات درخواست

### مقدمه

خطای CORS یکی از گیج‌کننده‌ترین خطاهای توسعهٔ وب است. در این فصل، آن را کامل می‌فهمیم و حل می‌کنیم.

### مفاهیم اساسی

**۱. CORS چیست؟**
**Cross-Origin Resource Sharing** مکانیزمی امنیتی مرورگر که تعیین می‌کند یک دامنه چطور به منابع دامنهٔ دیگر دسترسی داشته باشد.

> **تشبیه**: CORS مثل نگهبان یک ساختمان است. مرورگر می‌پرسد «آیا این مهمان اجازهٔ ورود دارد؟» و سرور باید با هدر تأیید کند.

**۲. Origin چیست؟**
ترکیب **پروتکل + دامنه + پورت**:
```
https://myapp.com:443  → Origin
https://myapp.com      → همان Origin
http://myapp.com       → Origin متفاوت!
https://api.myapp.com  → Origin متفاوت!
```

**۳. Preflight Request**
برای درخواست‌های پیچیده، مرورگر ابتدا یک درخواست `OPTIONS` می‌فرستد.

**۴. هدرهای مهم**

| هدر | کار |
|---|---|
| `Access-Control-Allow-Origin` | دامنه‌های مجاز |
| `Access-Control-Allow-Methods` | متدهای مجاز |
| `Access-Control-Allow-Headers` | هدرهای مجاز |
| `Access-Control-Allow-Credentials` | ارسال کوکی |
| `Access-Control-Max-Age` | مدت کش Preflight |

**۵. پیام‌های خطای CORS**

```
Access to fetch at 'https://api.example.com' from origin 'https://myapp.com'
has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header
is present on the requested resource.
```

### مثال‌های عملی

**مثال ۱ — تنظیم CORS ساده**

```js
// api/data.js
export default function handler(req, res) {
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');

  // مهم: درخواست Preflight را پاسخ بده
  if (req.method === 'OPTIONS') {
    return res.status(204).end();
  }

  res.json({ data: 'ok' });
}
```

**مثال ۲ — CORS امن (فقط دامنه‌های مجاز)**

```js
// lib/cors.js
const ALLOWED_ORIGINS = [
  'https://myapp.com',
  'https://www.myapp.com',
  process.env.NODE_ENV === 'development' ? 'http://localhost:3000' : null,
].filter(Boolean);

export function setCors(req, res) {
  const origin = req.headers.origin;

  if (ALLOWED_ORIGINS.includes(origin)) {
    res.setHeader('Access-Control-Allow-Origin', origin);
    res.setHeader('Vary', 'Origin');   // مهم برای کش
  }

  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  res.setHeader('Access-Control-Max-Age', '86400');
}
```

```js
// api/data.js
import { setCors } from '../lib/cors';

export default function handler(req, res) {
  setCors(req, res);

  if (req.method === 'OPTIONS') return res.status(204).end();

  res.json({ data: 'ok' });
}
```

**مثال ۳ — CORS با کوکی (Credentials)**

```js
// api/auth/me.js
export default function handler(req, res) {
  const origin = req.headers.origin;

  // با Credentials، نمی‌توان از '*' استفاده کرد
  res.setHeader('Access-Control-Allow-Origin', origin);
  res.setHeader('Access-Control-Allow-Credentials', 'true');
  res.setHeader('Vary', 'Origin');

  if (req.method === 'OPTIONS') return res.status(204).end();

  res.json({ user: req.user });
}
```

**مثال ۴ — CORS با Next.js Route Handler**

```ts
// app/api/data/route.ts
import { NextResponse } from 'next/server';

const CORS_HEADERS = {
  'Access-Control-Allow-Origin': 'https://myapp.com',
  'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
  'Access-Control-Allow-Headers': 'Content-Type',
};

export async function OPTIONS() {
  return new NextResponse(null, { status: 204, headers: CORS_HEADERS });
}

export async function GET() {
  return NextResponse.json({ data: 'ok' }, { headers: CORS_HEADERS });
}
```

**مثال ۵ — CORS سراسری در `vercel.json`**

```json
{
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        { "key": "Access-Control-Allow-Origin", "value": "https://myapp.com" },
        { "key": "Access-Control-Allow-Methods", "value": "GET,POST,PUT,DELETE,OPTIONS" },
        { "key": "Access-Control-Allow-Headers", "value": "Content-Type,Authorization" },
        { "key": "Access-Control-Max-Age", "value": "86400" }
      ]
    }
  ]
}
```

### مقایسه و الگوهای مختلف

**`*` در برابر دامنهٔ مشخص**

| معیار | `*` | دامنهٔ مشخص |
|---|---|---|
| امنیت | ❌ | ✅ |
| با Credentials | ❌ کار نمی‌کند | ✅ |
| مناسب برای | API عمومی | اپ با auth |

**دو راه‌حل CORS**

| راه‌حل | توضیح | مناسب برای |
|---|---|---|
| **هدر در سرور** | تنظیم CORS | API خودتان |
| **پروکسی** | عبور از API خودتان | API شخص ثالث |

```js
// پروکسی برای API شخص ثالث (دور زدن CORS)
// api/proxy/route.ts
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const target = searchParams.get('url');

  // فقط دامنه‌های مجاز
  if (!target?.startsWith('https://api.trusted.com')) {
    return Response.json({ error: 'غیرمجاز' }, { status: 400 });
  }

  const res = await fetch(target);
  const data = await res.json();
  return Response.json(data);
}
```

### اشکال شایع

**۱. `*` با `credentials: true`.**
مرورگر رد می‌کند. باید دامنهٔ مشخص بدهید.

**۲. فراموش کردن پاسخ `OPTIONS`.**
Preflight شکست می‌خورد. همیشه `OPTIONS` را پاسخ دهید.

**۳. نبود `Vary: Origin`.**
کش CDN پاسخ اشتباه می‌دهد.

**۴. فقط در Production ست کردن CORS.**
در توسعه کار نمی‌کند. `localhost` را هم اضافه کنید.

**۵. نبود هدرهای لازم.**
اگر `Authorization` می‌فرستید، باید در `Allow-Headers` باشد.

### نکات مهم (Tips & Tricks)

- **همیشه `OPTIONS` را پاسخ دهید**.
- برای امنیت، **دامنهٔ مشخص** بدهید (نه `*`).
- `Vary: Origin` را فراموش نکنید.
- `localhost` را برای توسعه اضافه کنید.
- برای API شخص ثالث، **پروکسی** بسازید.

### خلاصه فصل

- **CORS** دسترسی بین دامنه‌ها را کنترل می‌کند.
- **Origin** = پروتکل + دامنه + پورت.
- **Preflight (`OPTIONS`)** را حتماً پاسخ دهید.
- `*` با Credentials کار نمی‌کند.
- `Vary: Origin` برای کش ضروری است.

---

<a id="فصل-۵"></a>
## فصل ۵: Timeout و Cold Start

### مقدمه

تابع شما کند است؟ یا خطای Timeout می‌دهد؟ این دو مشکل رایج Serverless را در این فصل حل می‌کنیم.

### مفاهیم اساسی

**۱. Timeout چیست؟**
تابع بیش از حد مجاز اجرا می‌شود و متوقف می‌شود.

| پلن | حداکثر مدت (تقریبی) |
|---|---|
| Hobby | کمتر |
| Pro | بیشتر |
| Enterprise | بیشترین |

> مقدار دقیق را در مستندات Limits بررسی کنید.

**۲. Cold Start**
تأخیر ساخت نمونهٔ جدید (فایل ۰۵).

**۳. علل کندی**

| علت | نشانه |
|---|---|
| Cold Start | اولین درخواست کند |
| کوئری کند | همه درخواست‌ها کند |
| وابستگی سنگین | شروع کند |
| اتصال دیتابیس | کندی دوره‌ای |
| API بیرونی کند | متغیر |

**۴. خطاهای رایج**

| پیام | معنی |
|---|---|
| `FUNCTION_INVOCATION_TIMEOUT` | تابع از حد گذشت |
| `FUNCTION_INVOCATION_FAILED` | خطای اجرا |
| `504 Gateway Timeout` | پاسخ نیامد |

**۵. راه‌های کاهش**

```
- کاهش اندازهٔ بسته
- کش کردن
- Edge برای منطق سبک
- Fluid Compute
- Connection Pooling
- بهینه‌سازی کوئری
```

### مثال‌های عملی

**مثال ۱ — افزایش مدت اجرا**

```json
// vercel.json
{
  "functions": {
    "api/heavy.js": {
      "maxDuration": 60,
      "memory": 1024
    }
  }
}
```

**مثال ۲ — بهینه‌سازی کوئری کند**

```js
// ❌ کند: چند کوئری جدا
const user = await sql`SELECT * FROM users WHERE id = ${id}`;
const orders = await sql`SELECT * FROM orders WHERE user_id = ${id}`;
const items = await sql`SELECT * FROM items WHERE user_id = ${id}`;

// ✅ سریع: یک کوئری با JOIN یا موازی
const [user, orders, items] = await Promise.all([
  sql`SELECT * FROM users WHERE id = ${id}`,
  sql`SELECT * FROM orders WHERE user_id = ${id}`,
  sql`SELECT * FROM items WHERE user_id = ${id}`,
]);
```

**مثال ۳ — Timeout در فراخوانی بیرونی**

```js
// lib/fetchWithTimeout.js
export async function fetchWithTimeout(url, options = {}, timeoutMs = 8000) {
  const controller = new AbortController();
  const timer = setTimeout(() => controller.abort(), timeoutMs);

  try {
    return await fetch(url, { ...options, signal: controller.signal });
  } catch (err) {
    if (err.name === 'AbortError') {
      throw new Error(`درخواست به ${url} از حد زمانی گذشت`);
    }
    throw err;
  } finally {
    clearTimeout(timer);
  }
}
```

**مثال ۴ — کاهش Cold Start با ایمپورت سبک**

```js
// ❌ سنگین
import { S3Client } from '@aws-sdk/client-s3';   // کل SDK

// ✅ سبک
import { S3Client } from '@aws-sdk/client-s3/dist-es/S3Client';   // فقط لازم
// یا از fetch مستقیم استفاده کنید
```

**مثال ۵ — کش برای کاهش بار**

```js
// api/products.js
import { kv } from '@vercel/kv';

export default async function handler(req, res) {
  const cached = await kv.get('products');
  if (cached) return res.json(cached);   // ✅ سریع، بدون کوئری

  const { rows } = await sql`SELECT * FROM products LIMIT 50`;
  await kv.set('products', rows, { ex: 300 });

  res.json(rows);
}
```

**مثال ۶ — استفاده از Edge برای پاسخ سریع**

```js
// api/status.js — منطق سبک → Edge
export const config = { runtime: 'edge' };

export default function handler() {
  return new Response(JSON.stringify({ status: 'ok' }), {
    headers: { 'content-type': 'application/json' },
  });
}
```

### مقایسه و الگوهای مختلف

**سه راه کاهش Cold Start**

| راه | اثر | تلاش |
|---|---|---|
| سبک کردن بسته | ✅ زیاد | کم |
| Edge | ✅✅ زیاد | متوسط |
| Fluid Compute | ✅ زیاد | صفر |

**سه راه رفع Timeout**

| راه | چه زمانی |
|---|---|
| افزایش `maxDuration` | کار واقعاً طولانی |
| بهینه‌سازی | کار قابل سریع‌تر شدن |
| پس‌زمینه (Queue) | کار خیلی طولانی |

```js
// الگوی پس‌زمینه برای کار طولانی
export default async function handler(req, res) {
  // سریع پاسخ بده
  res.status(202).json({ queued: true });

  // کار را در صف بگذار
  await kv.lpush('jobs', JSON.stringify(req.body));
}
// و با Cron پردازش کن (فایل ۰۹)
```

### اشکال شایع

**۱. تنظیم `maxDuration` بالاتر از پلن.**
Build خطا می‌دهد. حد پلن را بررسی کنید.

**۲. نبود Timeout در fetch بیرونی.**
تابع معلق می‌ماند و هزینه می‌سوزاند.

**۳. کوئری‌های سری (Sequential).**
با `Promise.all` موازی کنید.

**۴. ایمپورت کل SDK.**
Cold Start را کند می‌کند.

**۵. کار طولانی در درخواست.**
به صف منتقل کنید.

### نکات مهم (Tips & Tricks)

- **بسته را سبک کنید** — مؤثرترین راه.
- **کش** بار دیتابیس را کم می‌کند.
- از **`Promise.all`** برای موازی‌سازی.
- **Timeout** روی همهٔ fetch های بیرونی.
- کار طولانی → **صف + Cron**.

### خلاصه فصل

- **Timeout** و **Cold Start** دو مشکل رایج Serverless‌اند.
- کاهش Cold Start: بستهٔ سبک، Edge، Fluid Compute.
- رفع Timeout: بهینه‌سازی، افزایش حد، صف.
- `Promise.all` کوئری‌های سری را موازی می‌کند.
- همیشه Timeout برای fetch بیرونی بگذارید.

---

<a id="فصل-۶"></a>
## فصل ۶: خطاهای رایج دیگر

### مقدمه

در این فصل، خطاهای متنوعی را که ممکن است ببینید، جمع می‌کنیم.

### خطاهای رایج و راه‌حل

**۱. خطای ۵۰۰ داخلی**

```js
// ❌ خطای بدون مدیریت
export default async function handler(req, res) {
  const data = await riskyOperation();   // اگر خطا بدهد، ۵۰۰
  res.json(data);
}

// ✅ مدیریت خطا
export default async function handler(req, res) {
  try {
    const data = await riskyOperation();
    res.json(data);
  } catch (err) {
    console.error(JSON.stringify({ event: 'error', message: err.message, stack: err.stack }));
    res.status(500).json({ error: 'خطای داخلی' });
  }
}
```

**۲. متغیر محیطی غایب**

```js
// ❌ خطا در زمان اجرا
const key = process.env.API_KEY;   // undefined

// ✅ بررسی
if (!process.env.API_KEY) {
  throw new Error('API_KEY تنظیم نشده');
}
```

**۳. خطای ۴۰۴ روی مسیرهای SPA**

```json
// vercel.json — ریدایرکت همه به index.html
{
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ]
}
```

**۴. «Too many redirects»**

```
- جهت ریدایرکت را یک‌طرفه کنید
- مسیرهای استثنا را در نظر بگیرید
- در Middleware، مسیر هدف را مستثنا کنید
```

**۵. تصاویر بارگذاری نمی‌شوند**

```js
// next.config.js — دامنه‌های بیرونی
module.exports = {
  images: {
    remotePatterns: [
      { protocol: 'https', hostname: 'images.example.com' },
    ],
  },
};
```

**۶. خطای Mixed Content**

```
صفحه HTTPS اما منبع HTTP
→ همهٔ منابع را HTTPS کنید
→ در DevTools → Console خطا را ببینید
```

**۷. کوکی تنظیم نمی‌شود**

```js
// ✅ ویژگی‌های درست
res.setHeader('Set-Cookie',
  `session=${id}; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=604800`
);

// ❌ اشتباه رایج
// - Secure در HTTP (کار نمی‌کند)
// - SameSite=None بدون Secure
// - Path اشتباه
```

**۸. فرم به‌روز نمی‌شود**

```tsx
// ✅ router.refresh() + revalidateTag
'use client';
import { useRouter } from 'next/navigation';

const router = useRouter();
await fetch('/api/revalidate?tag=data', { method: 'POST' });
router.refresh();
```

**۹. خطای Hydration**

```
Error: Hydration failed because the initial UI does not match

دلایل:
- تاریخ/زمان در رندر
- Math.random() در رندر
- localStorage در رندر

راه‌حل: در useEffect قرار دهید
```

```tsx
// ❌ خطا
export default function Component() {
  return <div>{new Date().toLocaleString()}</div>;
}

// ✅ درست
'use client';
export default function Component() {
  const [time, setTime] = useState('');
  useEffect(() => setTime(new Date().toLocaleString()), []);
  return <div>{time}</div>;
}
```

**۱۰. خطای «Module not found: Can't resolve 'fs'»**

```js
// ❌ ایمپورت ماژول Node در Edge یا کلاینت
import fs from 'fs';

// ✅ فقط در Node Runtime
export const config = { runtime: 'nodejs' };
```

**۱۱. حجم بستهٔ تابع زیاد**

```bash
# بررسی حجم
npm run build

# راه‌حل‌ها:
- حذف وابستگی‌های بی‌استفاده
- ایمپورت جزئی
- خارج کردن فایل‌های بزرگ از بسته
```

**۱۲. خطای «No Output Directory»**

```json
// vercel.json — مطمئن شوید خروجی درست است
{
  "outputDirectory": "dist"
}
```

### جدول جامع خطاها

| خطا | علت | راه‌حل |
|---|---|---|
| ۵۰۰ | باگ کد | try/catch + لاگ |
| ۴۰۴ | مسیر/SPA | rewrites |
| ۴۰۱ | auth | بررسی توکن |
| ۴۰۳ | دسترسی | بررسی نقش |
| ۴۲۹ | Rate Limit | محدودیت کمتر |
| Build Fail | وابستگی/تایپ | Build Logs |
| Invalid DNS | رکورد | تنظیم درست |
| CORS | هدر | تنظیم CORS |
| Timeout | مدت | بهینه/افزایش |
| Hydration | عدم تطابق | useEffect |
| Mixed Content | HTTP/HTTPS | همه HTTPS |

### چک‌لیست دیباگ نهایی

- [ ] نوع خطا مشخص شد (Build/Runtime)
- [ ] لاگ مربوطه بررسی شد
- [ ] محلی بازتولید شد
- [ ] تفاوت محیط بررسی شد (Node، متغیرها)
- [ ] تغییرات اخیر Git مرور شد
- [ ] یک فرضیه آزمایش شد
- [ ] رفع و تأیید شد
- [ ] مستندسازی شد

### نکات مهم (Tips & Tricks)

- همیشه **try/catch** داشته باشید.
- **متغیرهای محیطی** را در زمان شروع بررسی کنید.
- برای SPA، **rewrites** بگذارید.
- **Hydration** را با `useEffect` حل کنید.
- خطاها را **ساختاریافته لاگ** کنید.

### خلاصه فصل

- ۱۲ خطای رایج با راه‌حل پوشش داده شد.
- جدول جامع خطاها را به خاطر بسپارید.
- مدیریت خطا و لاگ ساختاریافته ضروری است.
- چک‌لیست دیباگ را در جریان کار بگذارید.
- بیشتر خطاها ریشهٔ مشخص و راه‌حل ساده دارند.

---

## منابع و مراجع

### مستندات رسمی
- Vercel Troubleshooting: <https://vercel.com/docs/errors>
- Build Errors: <https://vercel.com/docs/deployments/troubleshoot-a-build>
- Domain Troubleshooting: <https://vercel.com/docs/domains/troubleshooting>
- Function Limits: <https://vercel.com/docs/functions/limitations>

### مفاهیم پیشنهادی برای مطالعه بیشتر
- MDN HTTP Status Codes: <https://developer.mozilla.org/docs/Web/HTTP/Status>
- CORS (MDN): <https://developer.mozilla.org/docs/Web/HTTP/CORS>
- DNS Checker: <https://dnschecker.org>

### گام بعدی
- [فایل ۱۷ — پروژه نهایی واقعی](./17-Final-Project.md)
- [فایل ۱۸ — چک‌لیست‌های جامع](./18-Checklist.md)
