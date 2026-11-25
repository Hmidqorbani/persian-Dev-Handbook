# ۱۰ — بهینه‌سازی عملکرد

**مدت یادگیری**: ۴ تا ۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: مطالعه فایل‌های ۰۴ (Next.js)، ۰۵ (توابع) و ۰۷ (دامنه)

---

## فهرست محتوا

- [فصل ۱: مبانی عملکرد وب و Core Web Vitals](#فصل-۱)
- [فصل ۲: CDN و Edge Network](#فصل-۲)
- [فصل ۳: Edge Cache و استراتژی‌های کش](#فصل-۳)
- [فصل ۴: Image Optimization](#فصل-۴)
- [فصل ۵: Fluid Compute](#فصل-۵)
- [فصل ۶: اندازه‌گیری و بهبود مستمر](#فصل-۶)

---

<a id="فصل-۱"></a>
## فصل ۱: مبانی عملکرد وب و Core Web Vitals

### مقدمه

«سایت سریع» یعنی چه؟ سرعت فقط یک حس نیست؛ **قابل اندازه‌گیری** است. گوگل سه معیار اصلی تعریف کرده که هم روی تجربهٔ کاربر و هم روی رتبهٔ SEO اثر دارند: **Core Web Vitals**.

### مفاهیم اساسی

**۱. سه معیار Core Web Vitals**

| معیار | نام کامل | معنی | مقدار خوب |
|---|---|---|---|
| **LCP** | Largest Contentful Paint | زمان نمایش بزرگ‌ترین عنصر | زیر ۲.۵ ثانیه |
| **INP** | Interaction to Next Paint | پاسخ‌دهی به تعامل کاربر | زیر ۲۰۰ms |
| **CLS** | Cumulative Layout Shift | پرش ناخواستهٔ چیدمان | زیر ۰.۱ |

**۲. معیارهای کمکی**

| معیار | معنی |
|---|---|
| **TTFB** | Time to First Byte (زمان اولین بایت) |
| **FCP** | First Contentful Paint (اولین محتوا) |
| **TTI** | Time to Interactive (قابل تعامل) |

**۳. عوامل مؤثر بر عملکرد**

```
سرعت = f(شبکه، حجم، محاسبات، کش)
```

| عامل | تأثیر | راه بهبود |
|---|---|---|
| فاصلهٔ کاربر-سرور | زیاد | CDN/Edge |
| حجم JavaScript | زیاد | Code Splitting، Server Components |
| حجم تصویر | زیاد | Image Optimization |
| کوئری دیتابیس | زیاد | کش، ایندکس |
| Cold Start | متوسط | Fluid Compute، Edge |

**۴. بودجهٔ عملکرد (Performance Budget)**
هدف مشخص تعیین کنید:
```
- LCP < 2.5s
- JavaScript < 200KB (gzipped)
- تصویر اصلی < 200KB
- TTFB < 500ms
```

### مثال‌های عملی

**مثال ۱ — اندازه‌گیری با Lighthouse**

```bash
# با Chrome DevTools: تب Lighthouse → Generate report

# یا با CLI
npm i -g lighthouse
lighthouse https://myapp.com --output html --output-path ./report.html

# خروجی: امتیاز ۰ تا ۱۰۰ برای Performance، Accessibility، SEO
```

**مثال ۲ — اندازه‌گیری Core Web Vitals در کد**

```tsx
// app/layout.tsx — با Vercel Speed Insights
import { SpeedInsights } from '@vercel/speed-insights/next';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="fa" dir="rtl">
      <body>
        {children}
        <SpeedInsights />
      </body>
    </html>
  );
}
```

**مثال ۳ — اندازه‌گیری دستی با Performance API**

```js
// lib/vitals.js
export function measureVitals() {
  if (typeof window === 'undefined') return;

  // LCP
  new PerformanceObserver((list) => {
    const entries = list.getEntries();
    const last = entries[entries.length - 1];
    console.log('LCP:', last.startTime.toFixed(0), 'ms');
  }).observe({ type: 'largest-contentful-paint', buffered: true });

  // CLS
  let cls = 0;
  new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      if (!entry.hadRecentInput) cls += entry.value;
    }
    console.log('CLS:', cls.toFixed(4));
  }).observe({ type: 'layout-shift', buffered: true });
}
```

### مقایسه و الگوهای مختلف

**سه سطح بهینه‌سازی**

| سطح | تمرکز | تلاش | اثر |
|---|---|---|---|
| **سریع** | تصویر، فونت، JS | کم | زیاد |
| **متوسط** | کش، CDN | متوسط | زیاد |
| **پیشرفته** | Streaming، PPR، Edge | زیاد | متوسط تا زیاد |

**معیارهای آزمایشگاهی در برابر میدانی**

| معیار | منبع | مزیت |
|---|---|---|
| **Lab (Lighthouse)** | محیط کنترل‌شده | تکرارپذیر |
| **Field (RUM)** | کاربران واقعی | واقعی‌تر |

> همیشه به **داده‌های میدانی** (Speed Insights) اعتماد کنید، نه فقط Lighthouse.

### اشکال شایع

**۱. تمرکز فقط روی Lighthouse.**
امتیاز ۱۰۰ در Lighthouse تضمین تجربهٔ واقعی نیست. دادهٔ میدانی مهم‌تر است.

**۲. نادیده گرفتن CLS.**
پرش چیدمان حس بدی می‌دهد. ابعاد تصویر/فونت را مشخص کنید.

**۳. بهینه‌سازی موبایل را فراموش کردن.**
بیشتر کاربران موبایل‌اند و شبکهٔ کندتری دارند.

**۴. اندازه‌گیری فقط در شبکهٔ سریع.**
با Throttling در DevTools تست کنید.

**۵. بهینه‌سازی یک‌باره.**
عملکرد مستمر است. پس از هر تغییر مهم، دوباره اندازه بگیرید.

### نکات مهم (Tips & Tricks)

- **LCP** را با تصویر `priority` و TTFB پایین بهبود دهید.
- **CLS** را با ابعاد مشخص و `font-display: swap` حل کنید.
- **INP** را با JS کمتر و Server Components بهبود دهید.
- از **Speed Insights** برای دادهٔ میدانی استفاده کنید.
- **بودجهٔ عملکرد** تعیین و در CI بررسی کنید.

### خلاصه فصل

- سه معیار اصلی: **LCP، INP، CLS**.
- عوامل مؤثر: شبکه، حجم، محاسبات، کش.
- دادهٔ **میدانی** مهم‌تر از آزمایشگاهی است.
- بودجهٔ عملکرد تعیین کنید.
- بهینه‌سازی یک فرایند مستمر است.

---

<a id="فصل-۲"></a>
## فصل ۲: CDN و Edge Network

### مقدمه

چرا کاربری در تهران باید منتظر سروری در آمریکا بماند؟ **CDN** این فاصله را حذف می‌کند. در این فصل، شبکهٔ لبهٔ Vercel را می‌شناسیم.

### مفاهیم اساسی

**۱. CDN چیست؟**
شبکه‌ای جهانی از سرورها که محتوا را از **نزدیک‌ترین نقطه** به کاربر می‌رساند.

**۲. Edge Network Vercel**
- صدها نقطهٔ حضور (PoP) در سراسر جهان.
- محتوا و کد را از نزدیک‌ترین نقطه سرو می‌کند.
- درخواست‌های استاتیک از **کش** و پویا از **توابع** پاسخ می‌گیرند.

**۳. مسیر یک درخواست**

```
کاربر (تهران)
   ↓
نزدیک‌ترین PoP (مثلاً دبی/فرانکفورت)
   ├── محتوای کش‌شده؟ → پاسخ فوری ✅
   └── نه؟ → رفتن به منبع (دیتاسنتر) → کش → پاسخ
```

**۴. انواع محتوا و محل سرو**

| محتوا | محل |
|---|---|
| فایل استاتیک (JS/CSS/تصویر) | CDN |
| HTML استاتیک | CDN |
| HTML پویا | Edge (ISR/PPR) |
| API | تابع Serverless/Edge |

**۵. Cache-Control و CDN**
هدرهای کش تعیین می‌کنند CDN چطور رفتار کند (فصل ۳).

### مثال‌های عملی

**مثال ۱ — بررسی هدرهای CDN**

```bash
curl -I https://myapp.com/

# هدرهای مهم:
# cache-control: public, max-age=0, must-revalidate
# x-vercel-cache: HIT   ← از کش آمد
# x-vercel-id: fra1::xxx  ← منطقهٔ پردازش
# age: 123              ← چند ثانیه در کش بوده
```

`x-vercel-cache` مقادیر: `HIT`, `MISS`, `STALE`, `BYPASS`, `REVALIDATED`.

**مثال ۲ — تنظیم Region تابع برای کاربران ایرانی**

```json
// vercel.json
{
  "functions": {
    "api/*.js": {
      "regions": ["fra1", "dxb1"]
    }
  }
}
```

`fra1` (فرانکفورت) و `dxb1` (دبی) نزدیک‌ترین مناطق به ایران هستند.

**مثال ۳ — بررسی توزیع جهانی**

```bash
# ابزارهای آنلاین: check-host.net، pingdom.com
# یا از چند نقطه تست کنید
for host in fra1 iad1 hnd1; do
  echo "=== $host ==="
  curl -o /dev/null -s -w "زمان اتصال: %{time_connect}s | TTFB: %{time_starttransfer}s\n" \
    https://myapp.com
done
```

### مقایسه و الگوهای مختلف

**CDN در برابر سرور مرکزی**

| معیار | سرور مرکزی | CDN |
|---|---|---|
| Latency برای کاربر دور | بالا | ✅ کم |
| مقیاس | دستی | ✅ خودکار |
| هزینهٔ پهنای باند | بالا | ✅ بهینه |
| DDoS | آسیب‌پذیر | ✅ توزیع‌شده |

**استاتیک در برابر پویا در Edge**

| نوع | محل پاسخ | سرعت |
|---|---|---|
| فایل استاتیک | کش CDN | ✅✅ |
| HTML استاتیک | کش CDN | ✅✅ |
| API (Serverless) | دیتاسنتر منطقه‌ای | ✅ |
| API (Edge) | نزدیک کاربر | ✅✅ |

### اشکال شایع

**۱. تنظیم Region نامناسب.**
کاربران ایرانی با `iad1` (آمریکا) تأخیر بیشتری دارند. از `fra1`/`dxb1` استفاده کنید.

**۲. کش نکردن فایل‌های استاتیک.**
هر بار از منبع می‌آید. هدر `immutable` بگذارید.

**۳. اشتباه در `x-vercel-cache`.**
اگر `BYPASS` می‌بینید، یعنی کش کار نمی‌کند — هدرها را بررسی کنید.

**۴. فراموش کردن Invalidation.**
بعد از تغییر، کش قدیمی سرو می‌شود.

**۵. تست از یک نقطه.**
عملکرد CDN را از چند منطقه تست کنید.

### نکات مهم (Tips & Tricks)

- Region را بر اساس **مخاطبان اصلی** انتخاب کنید.
- فایل‌های هش‌دار (`app-a1b2c3.js`) را با `immutable` کش کنید.
- از `x-vercel-cache` برای دیباگ استفاده کنید.
- برای کاربران ایرانی، `fra1`/`dxb1` را در نظر بگیرید.
- توزیع جهانی را از چند نقطه تست کنید.

### خلاصه فصل

- **CDN/Edge Network** محتوا را به کاربر نزدیک می‌کند.
- مسیر درخواست: کاربر → نزدیک‌ترین PoP → کش/منبع.
- انتخاب **Region** روی تأخیر اثر مستقیم دارد.
- `x-vercel-cache` وضعیت کش را نشان می‌دهد.
- برای ایران، `fra1`/`dxb1` مناسب‌ترند.

---

<a id="فصل-۳"></a>
## فصل ۳: Edge Cache و استراتژی‌های کش

### مقدمه

کش، مهم‌ترین ابزار عملکرد است: پاسخ آماده را فوراً می‌دهد. اما کش اشتباه، فاجعه می‌سازد (کاربر دادهٔ قدیمی یا شخصی دیگران را می‌بیند). در این فصل، کش را درست یاد می‌گیریم.

### مفاهیم اساسی

**۱. هدرهای کش**

| هدر | معنی |
|---|---|
| `Cache-Control` | قوانین کش |
| `s-maxage` | مدت کش در CDN/لبه |
| `max-age` | مدت کش در مرورگر |
| `stale-while-revalidate` | سرو نسخهٔ قدیمی در حین بازسازی |
| `no-store` | هرگز کش نکن |
| `private` | فقط مرورگر (نه CDN) |

**۲. سطوح کش**

```
مرورگر  →  CDN/Edge  →  دیتاسنتر  →  دیتابیس
(کوتاه)    (متوسط)      (کش تابع)    (کش کوئری)
```

**۳. `stale-while-revalidate` (SWR)**
به‌جای اینکه کاربر منتظر تازه‌سازی بماند، نسخهٔ قدیمی فوراً سرو می‌شود و در پس‌زمینه تازه می‌شود.

**۴. Invalidation**
پاک کردن کش پس از تغییر داده:
- **زمانی** (TTL).
- **دستی** (On-Demand).
- **برچسبی** (Cache Tags).

**۵. سه استراتژی اصلی**

| استراتژی | هدر | مناسب برای |
|---|---|---|
| **کش طولانی + Immutable** | `s-maxage=31536000, immutable` | فایل هش‌دار |
| **SWR** | `s-maxage=60, stale-while-revalidate=600` | داده نیمه‌پویا |
| **بدون کش** | `no-store` | داده شخصی |

### مثال‌های عملی

**مثال ۱ — کش کردن پاسخ API**

```js
// api/products.js
export default async function handler(req, res) {
  const products = await getProducts();

  res.setHeader(
    'Cache-Control',
    'public, s-maxage=60, stale-while-revalidate=600'
  );
  // ↑ ۶۰ ثانیه تازه، تا ۱۰ دقیقهٔ بعد نسخهٔ قدیمی سرو می‌شود

  res.json(products);
}
```

**مثال ۲ — فایل استاتیک با کش طولانی**

```js
// next.config.js
module.exports = {
  async headers() {
    return [
      {
        source: '/static/:path*',
        headers: [
          {
            key: 'Cache-Control',
            value: 'public, max-age=31536000, immutable',
          },
        ],
      },
    ];
  },
};
```

**مثال ۳ — دادهٔ شخصی بدون کش**

```js
// api/me.js
export default async function handler(req, res) {
  const user = await getUser(req);

  // دادهٔ شخصی → هرگز کش نشود
  res.setHeader('Cache-Control', 'private, no-store');

  res.json(user);
}
```

**مثال ۴ — کش تابعی (Function Cache) در حافظه**

```js
// lib/cache.js — کش ساده در حافظه (فقط برای نمونه)
const cache = new Map();

export async function cachedFetch(key, ttlMs, fetcher) {
  const hit = cache.get(key);
  if (hit && Date.now() - hit.time < ttlMs) {
    return hit.value;
  }

  const value = await fetcher();
  cache.set(key, { value, time: Date.now() });
  return value;
}
```

> ⚠️ در Serverless، حافظه **پایدار نیست**. برای کش پایدار از **KV** استفاده کنید.

**مثال ۵ — Invalidation دستی کش CDN**

```bash
# با Vercel CLI
vercel cache invalidate --tag products

# یا از Dashboard: Deployments → Cache
```

### مقایسه و الگوهای مختلف

**سه سطح TTL**

| سطح | مقدار | مناسب برای |
|---|---|---|
| کوتاه | ۱۰–۶۰ ثانیه | قیمت، موجودی |
| متوسط | ۵–۶۰ دقیقه | لیست محصولات، اخبار |
| بلند | ساعت/روز | صفحات ثابت، فایل |

**کش در مرورگر در برابر کش CDN**

| معیار | `max-age` (مرورگر) | `s-maxage` (CDN) |
|---|---|---|
| محل | مرورگر کاربر | لبه |
| تأثیر | کاربر فعلی | همهٔ کاربران |
| Invalidation | سخت | ✅ آسان |
| مناسب برای | فایل ثابت | محتوای مشترک |

### اشکال شایع

**۱. کش کردن دادهٔ شخصی.**
کاربر دیگران دادهٔ شما را می‌بینند! `private, no-store` بگذارید.

**۲. TTL خیلی بلند برای دادهٔ پویا.**
قیمت اشتباه نمایش داده می‌شود. TTL مناسب انتخاب کنید.

**۳. فراموش کردن Invalidation.**
بعد از تغییر داده، کش قدیمی می‌ماند.

**۴. `max-age` بلند بدون `s-maxage`.**
مرورگر نسخهٔ قدیمی را برای مدت طولانی نگه می‌دارد.

**۵. کش کردن پاسخ خطا.**
خطاهای موقت کش می‌شوند و کاربر همچنان خطا می‌بیند. برای خطاها کش نگذارید.

### نکات مهم (Tips & Tricks)

- **پیش‌فرض: کش کنید**، مگر دادهٔ شخصی.
- از **`stale-while-revalidate`** برای تجربهٔ روان استفاده کنید.
- فایل‌های هش‌دار را با `immutable` کش کنید.
- دادهٔ شخصی → `private, no-store`.
- پس از تغییر داده، **Invalidate** کنید.

### خلاصه فصل

- هدرهای کلیدی: **`s-maxage`، `max-age`، `stale-while-revalidate`، `no-store`**.
- سه استراتژی: **طولانی+Immutable، SWR، بدون کش**.
- `s-maxage` برای CDN و `max-age` برای مرورگر است.
- دادهٔ شخصی هرگز کش نشود.
- Invalidation بخشی از طراحی کش است.

---

<a id="فصل-۴"></a>
## فصل ۴: Image Optimization

### مقدمه

تصاویر معمولاً سنگین‌ترین بخش یک صفحه‌اند. بهینه‌سازی درست آن‌ها می‌تواند LCP را چند ثانیه بهبود دهد. Vercel یک **Image Optimization API** در لبه دارد.

### مفاهیم اساسی

**۱. بهینه‌سازی خودکار Vercel**
هنگام استفاده از `next/image`، Vercel:
- فرمت را به **WebP/AVIF** تبدیل می‌کند.
- اندازهٔ مناسب دستگاه را می‌سازد.
- در **کش لبه** نگه می‌دارد.

**۲. پارامترهای مهم `<Image>`**

| پارامتر | کار |
|---|---|
| `width`/`height` | ابعاد (جلوگیری از CLS) |
| `sizes` | اندازهٔ نمایش در نقاط شکست |
| `priority` | بارگذاری فوری (برای LCP) |
| `quality` | کیفیت (۱–۱۰۰) |
| `placeholder` | تصویر موقت (`blur`) |
| `fill` | پرکردن والد |

**۳. `sizes` چیست؟**
به مرورگر می‌گوید تصویر در هر اندازهٔ صفحه چقدر جا می‌گیرد، تا فایل مناسب را دانلود کند.

**۴. محدودیت‌ها و هزینه**
هر تصویر بهینه‌شده مصرف دارد. تصاویر را کش کنید و از کیفیت مناسب استفاده کنید.

**۵. تصاویر بیرونی**
دامنه‌های بیرونی را در `remotePatterns` مجاز کنید (فایل ۰۴).

### مثال‌های عملی

**مثال ۱ — تصویر LCP با اولویت**

```tsx
import Image from 'next/image';

export default function Hero() {
  return (
    <Image
      src="/hero.jpg"
      alt="تصویر اصلی"
      width={1200}
      height={600}
      priority                    // ← برای LCP
      sizes="100vw"
      quality={85}
      placeholder="blur"
      blurDataURL="data:image/jpeg;base64,..."
    />
  );
}
```

**مثال ۲ — تصویر ریسپانسیو در گرید**

```tsx
<Image
  src="/product.jpg"
  alt="محصول"
  width={400}
  height={400}
  sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 33vw"
  // ↑ موبایل: تمام عرض | تبلت: نصف | دسکتاپ: یک‌سوم
  quality={80}
/>
```

**مثال ۳ — تصویر پس‌زمینه با `fill`**

```tsx
<div style={{ position: 'relative', width: '100%', height: 400 }}>
  <Image
    src="/banner.jpg"
    alt="بنر"
    fill
    sizes="100vw"
    style={{ objectFit: 'cover' }}
    priority
  />
</div>
```

**مثال ۴ — بهینه‌سازی SVG و تصاویر کوچک**

```tsx
// SVG نیازی به Image ندارد
<img src="/logo.svg" alt="لوگو" width={120} height={40} />

// تصویر کوچک (کمتر از ۱۰KB) → شاید base64 بهتر باشد
// یا از unoptimized استفاده کنید
<Image src="/tiny-icon.png" alt="" width={24} height={24} unoptimized />
```

### مقایسه و الگوهای مختلف

**فرمت‌های تصویر**

| فرمت | مزیت | مناسب برای |
|---|---|---|
| **AVIF** | بهترین فشرده‌سازی | ✅ عکس |
| **WebP** | پشتیبانی گسترده | ✅ عکس |
| **JPEG** | سازگاری | عکس قدیمی |
| **PNG** | شفافیت | لوگو، آیکون |
| **SVG** | برداری، سبک | آیکون، لوگو |

**`priority` در برابر Lazy**

| حالت | چه زمانی |
|---|---|
| `priority` | تصویر بالای صفحه (LCP) |
| پیش‌فرض (Lazy) | تصاویر پایین صفحه |

### اشکال شایع

**۱. `priority` روی همهٔ تصاویر.**
بارگذاری همه فوری می‌شود و LCP خراب می‌شود. فقط تصویر اصلی.

**۲. نبود `sizes`.**
مرورگر تصویر بزرگ‌تر از نیاز دانلود می‌کند.

**۳. کیفیت ۱۰۰.**
حجم زیاد بدون بهبود محسوس. ۷۵–۸۵ کافی است.

**۴. تصویر بیرونی بدون `remotePatterns`.**
خطای Build می‌گیرید.

**۵. نادیده گرفتن مصرف Image Optimization.**
در پلن‌های بالاتر هزینه دارد. تصاویر را کش کنید.

### نکات مهم (Tips & Tricks)

- **LCP** را با `priority` + کیفیت مناسب بهینه کنید.
- همیشه **`sizes`** بدهید.
- کیفیت **۷۵–۸۵** برای اکثر تصاویر کافی است.
- برای SVG از `<img>` ساده استفاده کنید.
- تصاویر را قبل از آپلود **فشرده** کنید.

### خلاصه فصل

- Vercel تصاویر را خودکار به **WebP/AVIF** تبدیل و کش می‌کند.
- `priority` فقط برای تصویر **LCP**.
- `sizes` جلوی دانلود فایل بزرگ را می‌گیرد.
- کیفیت ۷۵–۸۵ نقطهٔ تعادل است.
- دامنه‌های بیرونی در `remotePatterns` مجاز شوند.

---

<a id="فصل-۵"></a>
## فصل ۵: Fluid Compute

### مقدمه

توابع Serverless سنتی هر درخواست را جدا اجرا می‌کنند و Cold Start دارند. **Fluid Compute** مدل مدرن Vercel است که این مشکلات را حل می‌کند و هزینه را بهینه می‌سازد.

### مفاهیم اساسی

**۱. Fluid Compute چیست؟**
مدل اجرایی که نمونه‌های تابع را **بازاستفاده** می‌کند و اجازه می‌دهد یک نمونه **چند درخواست را همزمان** سرو کند.

**۲. مزایا**

| مزیت | توضیح |
|---|---|
| **Cold Start کمتر** | نمونه‌ها بیشتر زنده می‌مانند |
| **همزمانی درون‌تابعی** | یک نمونه، چند درخواست |
| **هزینهٔ بهینه** | پرداخت بر پایهٔ CPU فعال |
| **مناسب AI** | استریم و بارهای طولانی |

**۳. مدل هزینه**
در مدل سنتی، هزینهٔ **کل زمان** اجرا محاسبه می‌شود (شامل انتظار I/O). در Fluid Compute، فقط **زمان CPU فعال** محاسبه می‌شود — یعنی انتظار برای دیتابیس هزینه ندارد.

**۴. تنظیم**
Fluid Compute در تنظیمات پروژه فعال می‌شود و برای پروژه‌های جدید پیش‌فرض فعال است.

**۵. تأثیر روی همزمانی**
یک نمونه می‌تواند چند درخواست را همزمان پردازش کند → تعداد نمونه‌های کمتر → Cold Start کمتر.

### مثال‌های عملی

**مثال ۱ — بررسی فعال بودن Fluid Compute**

```bash
# در Dashboard:
# Project Settings → Functions → Fluid Compute

# بررسی در کد (نام منطقه/محیط)
curl https://myapp.com/api/whoami
# خروجی: { "region": "fra1", "runtime": "nodejs" }
```

**مثال ۲ — تابع با انتظار I/O (کمترین هزینه با Fluid)**

```js
// api/slow-io.js
export default async function handler(req, res) {
  const start = Date.now();

  // انتظار برای دیتابیس (زمان CPU مصرف نمی‌کند)
  const data = await db.query('SELECT ...');   // ۲ ثانیه انتظار

  // پردازش (زمان CPU مصرف می‌کند)
  const processed = data.map((row) => transform(row));   // ۱۰ms

  res.json({
    result: processed,
    totalTime: Date.now() - start,
    note: 'زمان انتظار در هزینه محاسبه نمی‌شود',
  });
}
```

**مثال ۳ — تنظیم حافظه برای بارهای سنگین**

```json
{
  "functions": {
    "api/ai-generate.js": {
      "memory": 3008,
      "maxDuration": 300
    }
  }
}
```

**مثال ۴ — مقایسه با Edge برای مسیرهای سریع**

```js
// مسیر سریع و بدون I/O → Edge
// api/ping.js
export const config = { runtime: 'edge' };
export default function handler() {
  return new Response(JSON.stringify({ pong: true }));
}
```

### مقایسه و الگوهای مختلف

**سه مدل اجرا**

| مدل | Cold Start | هزینه | مناسب برای |
|---|---|---|---|
| **Serverless سنتی** | بیشتر | کل زمان | API سبک |
| **Fluid Compute** | کمتر | CPU فعال | API + AI |
| **Edge** | تقریباً صفر | درخواست | منطق سبک |

**Fluid در برابر Edge**

| معیار | Fluid Compute | Edge |
|---|---|---|
| Runtime | Node.js کامل | Web APIs |
| دیتابیس | ✅ | محدود |
| Cold Start | کم | تقریباً صفر |
| مناسب برای | API + AI | auth، redirect |

### اشکال شایع

**۱. انتظار صفر بودن Cold Start.**
Fluid آن را **کم** می‌کند، نه حذف کامل.

**۲. استفاده از Fluid برای منطق سبک.**
Edge سریع‌تر و ارزان‌تر است.

**۳. نادیده گرفتن حافظه.**
بار سنگین با حافظهٔ کم کند می‌شود.

**۴. فرض یکسان بودن هزینه در همهٔ بارها.**
بار CPU-محور هزینهٔ بیشتری دارد.

**۵. تست نکردن عملکرد پس از فعال‌سازی.**
همیشه قبل و بعد را اندازه بگیرید.

### نکات مهم (Tips & Tricks)

- **Fluid Compute** را فعال نگه دارید (پیش‌فرض جدید).
- برای بارهای I/O-محور، صرفه‌جویی هزینه چشمگیر است.
- برای منطق **سبک و سریع**، Edge بهتر است.
- حافظه را متناسب با بار تنظیم کنید.
- عملکرد را با **Speed Insights** بسنجید.

### خلاصه فصل

- **Fluid Compute** نمونه‌ها را بازاستفاده و همزمانی را زیاد می‌کند.
- Cold Start را کم و هزینه را بهینه می‌کند.
- پرداخت بر پایهٔ **CPU فعال**، نه کل زمان.
- برای API + AI عالی است.
- برای منطق سبک، Edge انتخاب بهتری است.

---

<a id="فصل-۶"></a>
## فصل ۶: اندازه‌گیری و بهبود مستمر

### مقدمه

بهینه‌سازی بدون اندازه‌گیری، حدس است. در این فصل، یک چرخهٔ حرفه‌ای اندازه‌گیری و بهبود را می‌سازیم.

### مفاهیم اساسی

**۱. چرخهٔ بهبود**
```
اندازه‌گیری → شناسایی گلوگاه → بهینه‌سازی → اندازه‌گیری مجدد
     ↑                                              │
     └──────────────────────────────────────────────┘
```

**۲. ابزارهای اندازه‌گیری**

| ابزار | نوع | کاربرد |
|---|---|---|
| **Speed Insights** | میدانی | Core Web Vitals واقعی |
| **Lighthouse** | آزمایشگاهی | بررسی جامع |
| **Web Analytics** | میدانی | رفتار کاربران |
| **DevTools** | آزمایشگاهی | دیباگ عمیق |

**۳. گلوگاه‌های رایج و راه‌حل**

| گلوگاه | نشانه | راه‌حل |
|---|---|---|
| تصویر بزرگ | LCP بالا | Image Optimization |
| JS زیاد | INP بالا | Code Splitting |
| Cold Start | TTFB بالا | Fluid/Edge |
| کوئری کند | TTFB بالا | ایندکس، کش |
| فونت | CLS بالا | `next/font` |

**۴. بودجهٔ عملکرد در CI**
می‌توانید در CI بررسی کنید که عملکرد از حد مشخصی بدتر نشود.

**۵. A/B Testing عملکرد**
دو نسخه را مقایسه کنید و نسخهٔ بهتر را نگه دارید.

### مثال‌های عملی

**مثال ۱ — نصب Speed Insights**

```bash
npm i @vercel/speed-insights
```

```tsx
// app/layout.tsx
import { SpeedInsights } from '@vercel/speed-insights/next';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="fa" dir="rtl">
      <body>
        {children}
        <SpeedInsights />
      </body>
    </html>
  );
}
```

**مثال ۲ — نصب Web Analytics**

```bash
npm i @vercel/analytics
```

```tsx
// app/layout.tsx
import { Analytics } from '@vercel/analytics/next';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="fa" dir="rtl">
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  );
}
```

**مثال ۳ — ردیابی رویداد سفارشی**

```tsx
// components/CTAButton.tsx
'use client';

import { track } from '@vercel/analytics';

export default function CTAButton() {
  return (
    <button
      onClick={() => {
        track('cta_click', { location: 'hero', variant: 'A' });
      }}
    >
      شروع کنید
    </button>
  );
}
```

**مثال ۴ — بررسی بودجهٔ عملکرد در CI**

```yaml
# .github/workflows/lighthouse.yml
name: Lighthouse

on: [pull_request]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: treosh/lighthouse-ci-action@v11
        with:
          urls: |
            https://myapp-git-preview.vercel.app
          budgetPath: ./budget.json
          uploadArtifacts: true
```

```json
// budget.json
[
  {
    "path": "/*",
    "timings": [
      { "metric": "largest-contentful-paint", "budget": 2500 },
      { "metric": "interactive", "budget": 3500 }
    ],
    "resourceSizes": [
      { "resourceType": "script", "budget": 200 },
      { "resourceType": "image", "budget": 500 }
    ]
  }
]
```

**مثال ۵ — جدول پیگیری بهبود**

| تاریخ | LCP | INP | CLS | تغییر |
|---|---|---|---|---|
| ۱۴۰۳/۰۱/۰۱ | ۳.۸s | ۲۵۰ms | ۰.۱۸ | شروع |
| ۱۴۰۳/۰۱/۱۵ | ۲.۱s | ۱۸۰ms | ۰.۰۵ | بهینه‌سازی تصویر |
| ۱۴۰۳/۰۲/۰۱ | ۱.۶s | ۱۲۰ms | ۰.۰۳ | کش + Edge |

### مقایسه و الگوهای مختلف

**دادهٔ آزمایشگاهی در برابر میدانی**

| معیار | آزمایشگاهی | میدانی |
|---|---|---|
| کنترل | ✅ کامل | ❌ |
| واقع‌گرایی | متوسط | ✅ بالا |
| تعداد نمونه | یک | هزاران |
| ابزار | Lighthouse | Speed Insights |

**سه سطح اولویت‌دهی**

| اولویت | تمرکز | اثر |
|---|---|---|
| ۱ | تصویر + JS | زیاد |
| ۲ | کش + CDN | زیاد |
| ۳ | Streaming + PPR | متوسط |

### اشکال شایع

**۱. بهینه‌سازی بدون اندازه‌گیری.**
نمی‌دانید چه چیزی مؤثر بوده.

**۲. اندازه‌گیری فقط روی دسکتاپ.**
بیشتر کاربران موبایل‌اند.

**۳. نادیده گرفتن دادهٔ میدانی.**
Lighthouse خوب اما کاربران ناراضی.

**۴. رها کردن پس از یک بهینه‌سازی.**
عملکرد با هر تغییر جدید افت می‌کند.

**۵. نبود بودجهٔ عملکرد.**
بدون هدف، افت تدریجی اجتناب‌ناپذیر است.

### نکات مهم (Tips & Tricks)

- **Speed Insights** را نصب کنید و دادهٔ میدانی بگیرید.
- بودجهٔ عملکرد تعیین و در **CI** بررسی کنید.
- ابتدا **تصویر و JS** را بهینه کنید (بیشترین اثر).
- موبایل و شبکهٔ کند را در تست لحاظ کنید.
- بهبود را **مستمر** نگه دارید.

### خلاصه فصل

- چرخهٔ بهبود: **اندازه‌گیری → شناسایی → بهینه‌سازی → اندازه‌گیری**.
- ابزارها: **Speed Insights، Lighthouse، Analytics، DevTools**.
- دادهٔ **میدانی** معیار نهایی است.
- بودجهٔ عملکرد را در CI بررسی کنید.
- بهینه‌سازی مستمر است، نه یک‌باره.

---

## منابع و مراجع

### مستندات رسمی
- Vercel Performance: <https://vercel.com/docs/performance>
- Speed Insights: <https://vercel.com/docs/speed-insights>
- Web Analytics: <https://vercel.com/docs/analytics>
- Edge Network: <https://vercel.com/docs/edge-network/overview>
- Fluid Compute: <https://vercel.com/docs/fluid-compute>
- Image Optimization: <https://vercel.com/docs/image-optimization>

### مفاهیم پیشنهادی برای مطالعه بیشتر
- Core Web Vitals (web.dev): <https://web.dev/vitals/>
- Lighthouse CI: <https://github.com/GoogleChrome/lighthouse-ci>
- PageSpeed Insights: <https://pagespeed.web.dev>
- WebPageTest: <https://www.webpagetest.org>

### گام بعدی
- [فایل ۱۱ — کش، ISR و PPR](./11-Caching-ISR-and-PPR.md)
- [فایل ۱۳ — پایش و مشاهده‌پذیری](./13-Observability.md)
