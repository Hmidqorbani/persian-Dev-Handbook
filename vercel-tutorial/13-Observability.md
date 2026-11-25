# ۱۳ — پایش و مشاهده‌پذیری

**مدت یادگیری**: ۲ تا ۳ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: مطالعه فایل‌های ۰۵ (توابع) و ۱۰ (عملکرد)

---

## فهرست محتوا

- [فصل ۱: Observability چیست؟](#فصل-۱)
- [فصل ۲: Logs — لاگ‌های Runtime و Build](#فصل-۲)
- [فصل ۳: Web Analytics](#فصل-۳)
- [فصل ۴: Speed Insights](#فصل-۴)
- [فصل ۵: Drains — ارسال داده‌ها](#فصل-۵)
- [فصل ۶: دیباگ با داده‌های واقعی](#فصل-۶)

---

<a id="فصل-۱"></a>
## فصل ۱: Observability چیست؟

### مقدمه

«سایت کار نمی‌کند» جملهٔ بی‌فایده‌ای است. **Observability** یعنی توانایی پاسخ دادن به این سؤال‌ها: چه چیزی، کجا، چه زمانی و چرا خراب شد؟ در این فصل، این مفهوم را می‌شناسیم.

### مفاهیم اساسی

**۱. سه ستون Observability**

| ستون | پاسخ به |
|---|---|
| **Logs** | چه اتفاقی افتاد؟ |
| **Metrics** | چقدر/چند وقت؟ |
| **Traces** | مسیر درخواست کجا بود؟ |

**۲. Monitoring در برابر Observability**

| معیار | Monitoring | Observability |
|---|---|---|
| سؤال | «آیا خراب است؟» | «چرا خراب است؟» |
| رویکرد | هشدار روی معیار | کاوش در داده |
| داده | محدود و از پیش تعریف‌شده | جامع |

**۳. ابزارهای Vercel**

| ابزار | نوع | کاربرد |
|---|---|---|
| **Runtime Logs** | Logs | خطاهای اجرا |
| **Build Logs** | Logs | خطاهای ساخت |
| **Web Analytics** | Metrics | بازدیدکنندگان |
| **Speed Insights** | Metrics | Core Web Vitals |
| **Observability Plus** | همه | جست‌وجو و تحلیل پیشرفته |
| **Drains** | Export | ارسال به سرویس بیرونی |

**۴. آنچه باید پایش کنید**

```
- نرخ خطا (Error Rate)
- زمان پاسخ (Latency)
- مصرف منابع (CPU، Memory)
- Core Web Vitals
- ترافیک و بازدید
- نرخ موفقیت استقرار
```

**۵. سطح بلوغ Observability**

| سطح | وضعیت |
|---|---|
| ۱ | فقط لاگ‌های پیش‌فرض |
| ۲ | + Analytics و Speed Insights |
| ۳ | + لاگ ساختاریافته و Alerting |
| ۴ | + Drains و سرویس تخصصی |

### مثال‌های عملی

**مثال ۱ — دسترسی به لاگ‌ها**

```
Dashboard → Project → Logs
- Runtime Logs: لاگ توابع و صفحات
- Build Logs: لاگ فرایند ساخت
- فیلتر بر اساس: استقرار، مسیر، سطح (error/warn/info)
```

**مثال ۲ — بررسی سریع سلامت**

```bash
# با CLI
vercel logs myapp.com

# با curl (health check)
curl -s https://myapp.com/api/health | jq
```

```js
// api/health.js — نقطهٔ بررسی سلامت
export default function handler(req, res) {
  res.status(200).json({
    status: 'ok',
    timestamp: new Date().toISOString(),
    env: process.env.VERCEL_ENV,
    region: process.env.VERCEL_REGION,
    commit: process.env.VERCEL_GIT_COMMIT_SHA?.slice(0, 7),
  });
}
```

**مثال ۳ — لاگ ساختاریافته**

```js
// lib/logger.js
export function log(level, message, meta = {}) {
  console.log(JSON.stringify({
    level,
    message,
    timestamp: new Date().toISOString(),
    ...meta,
  }));
}

export const logger = {
  info: (msg, meta) => log('info', msg, meta),
  warn: (msg, meta) => log('warn', msg, meta),
  error: (msg, meta) => log('error', msg, meta),
};
```

```js
// api/order.js
import { logger } from '../lib/logger';

export default async function handler(req, res) {
  const start = Date.now();
  const requestId = crypto.randomUUID();

  try {
    logger.info('سفارش شروع شد', { requestId, userId: req.body.userId });

    const order = await createOrder(req.body);

    logger.info('سفارش ساخته شد', {
      requestId,
      orderId: order.id,
      duration: Date.now() - start,
    });

    res.status(201).json(order);
  } catch (err) {
    logger.error('سفارش ناموفق', {
      requestId,
      error: err.message,
      duration: Date.now() - start,
    });
    res.status(500).json({ error: 'خطای داخلی', requestId });
  }
}
```

### مقایسه و الگوهای مختلف

**سه سطح لاگ‌گذاری**

| سطح | چه زمانی |
|---|---|
| `info` | رویدادهای عادی |
| `warn` | وضعیت مشکوک |
| `error` | خطا |

**لاگ متنی در برابر ساختاریافته**

| معیار | متنی | JSON |
|---|---|---|
| خوانایی انسان | ✅ | متوسط |
| جست‌وجو | سخت | ✅ آسان |
| تحلیل خودکار | ❌ | ✅ |
| مناسب برای | دیباگ سریع | Production |

### اشکال شایع

**۱. لاگ کردن دادهٔ حساس.**
رمز، توکن و اطلاعات کارت هرگز لاگ نشوند.

**۲. لاگ بیش از حد.**
هزینه و نویز بالا. فقط رویدادهای مهم.

**۳. نبود شناسهٔ درخواست.**
ردیابی یک درخواست در چند لاگ سخت می‌شود. `requestId` بگذارید.

**۴. پیام‌های لاگ مبهم.**
`"error"` به‌تنهایی بی‌فایده است. زمینه بدهید.

**۵. نادیده گرفتن Build Logs.**
بیشتر خطاهای استقرار همان‌جا مشخص می‌شوند.

### نکات مهم (Tips & Tricks)

- **لاگ ساختاریافته (JSON)** بنویسید.
- `requestId` برای ردیابی درخواست.
- دادهٔ حساس را **هرگز** لاگ نکنید.
- **Alerting** روی نرخ خطا راه‌اندازی کنید.
- Build Logs را در هر خطای استقرار ببینید.

### خلاصه فصل

- سه ستون: **Logs، Metrics، Traces**.
- ابزارها: **Runtime/Build Logs، Analytics، Speed Insights، Drains**.
- لاگ **ساختاریافته** برای Production.
- `requestId` ردیابی را ساده می‌کند.
- دادهٔ حساس هرگز لاگ نشود.

---

<a id="فصل-۲"></a>
## فصل ۲: Logs — لاگ‌های Runtime و Build

### مقدمه

لاگ‌ها اولین جای سرکشی هنگام مشکل هستند. در این فصل، دو نوع لاگ در Vercel را کامل می‌شناسیم.

### مفاهیم اساسی

**۱. Runtime Logs**
لاگ‌های توابع Serverless و صفحات SSR. در زمان اجرای درخواست تولید می‌شوند.

**۲. Build Logs**
لاگ‌های فرایند ساخت. هنگام Deploy تولید می‌شوند.

**۳. تفاوت‌ها**

| معیار | Runtime Logs | Build Logs |
|---|---|---|
| زمان | هنگام اجرا | هنگام ساخت |
| محتوا | `console.log`، خطاها | نصب، کامپایل، خطاهای ساخت |
| نگهداری | کوتاه‌مدت | با استقرار |
| کاربرد | دیباگ رفتار | دیباگ ساخت |

**۴. سطوح لاگ**

| سطح | متد |
|---|---|
| `info` | `console.log` |
| `warn` | `console.warn` |
| `error` | `console.error` |
| `debug` | `console.debug` |

**۵. دسترسی**
- **Dashboard**: Logs tab (زنده و قابل فیلتر).
- **CLI**: `vercel logs`.
- **Drains**: ارسال به سرویس بیرونی.

### مثال‌های عملی

**مثال ۱ — مشاهدهٔ لاگ با CLI**

```bash
# لاگ زندهٔ یک استقرار
vercel logs https://myapp-abc123.vercel.app

# لاگ Production
vercel logs myapp.com

# فقط خطاها
vercel logs myapp.com | grep -i error

# ذخیره در فایل
vercel logs myapp.com > logs.txt
```

**مثال ۲ — لاگ با زمینهٔ کامل**

```js
// api/users/[id].js
export default async function handler(req, res) {
  const { id } = req.query;
  const start = Date.now();

  console.log(JSON.stringify({
    level: 'info',
    event: 'user.fetch.start',
    userId: id,
    method: req.method,
    path: req.url,
  }));

  try {
    const user = await getUser(id);

    if (!user) {
      console.warn(JSON.stringify({
        level: 'warn',
        event: 'user.not_found',
        userId: id,
      }));
      return res.status(404).json({ error: 'کاربر پیدا نشد' });
    }

    console.log(JSON.stringify({
      level: 'info',
      event: 'user.fetch.success',
      userId: id,
      duration: Date.now() - start,
    }));

    res.json(user);
  } catch (err) {
    console.error(JSON.stringify({
      level: 'error',
      event: 'user.fetch.failed',
      userId: id,
      error: err.message,
      stack: err.stack,
      duration: Date.now() - start,
    }));
    res.status(500).json({ error: 'خطای داخلی' });
  }
}
```

**مثال ۳ — اندازه‌گیری زمان‌های کلیدی**

```js
// lib/timing.js
export function createTimer(label) {
  const marks = {};
  const start = Date.now();

  return {
    mark(name) {
      marks[name] = Date.now() - start;
    },
    done(meta = {}) {
      console.log(JSON.stringify({
        level: 'info',
        event: 'timing',
        label,
        total: Date.now() - start,
        marks,
        ...meta,
      }));
    },
  };
}
```

```js
// api/search.js
import { createTimer } from '../lib/timing';

export default async function handler(req, res) {
  const timer = createTimer('search');

  const results = await db.search(req.query.q);
  timer.mark('db');

  const enriched = await enrich(results);
  timer.mark('enrich');

  timer.done({ query: req.query.q, count: enriched.length });

  res.json(enriched);
}
```

**مثال ۴ — لاگ خطای ساخت**

```bash
# در Build Logs به دنبال این الگوها بگردید:
# "Module not found"
# "Type error"
# "Failed to compile"
# "Command failed with exit code 1"
# "npm ERR!"
```

### مقایسه و الگوهای مختلف

**سه روش دسترسی به لاگ**

| روش | مزیت | عیب |
|---|---|---|
| **Dashboard** | بصری، زنده | نیاز به مرورگر |
| **CLI** | سریع، اسکریپت‌پذیر | خط فرمان |
| **Drains** | بلندمدت، جست‌وجو | نیاز به سرویس بیرونی |

**نگهداری لاگ**

| پلن | مدت نگهداری |
|---|---|
| Hobby | کوتاه‌مدت |
| Pro | بلندتر |
| Enterprise | قابل تنظیم |
| Drains | نامحدود (سرویس شما) |

### اشکال شایع

**۱. جست‌وجو در لاگ متنی.**
اگر JSON نباشد، فیلتر سخت است.

**۲. نبود شناسه در لاگ.**
ردیابی درخواست ممکن نیست.

**۳. لاگ در حلقه.**
میلیون‌ها لاگ و هزینهٔ بالا. فقط رویدادهای مهم.

**۴. اعتماد به `console.log` به‌تنهایی.**
بدون ساختار، تحلیل سخت است.

**۵. نادیده گرفتن لاگ هشدارها.**
`warn` ها نشانهٔ مشکلات آینده‌اند.

### نکات مهم (Tips & Tricks)

- **JSON ساختاریافته** بنویسید.
- `event` نام‌گذاری منظم (`user.fetch.start`).
- **زمان اجرا** را اندازه بگیرید.
- دادهٔ حساس را **ماسک** کنید.
- برای نگهداری بلندمدت، **Drains** راه‌اندازی کنید.

### خلاصه فصل

- دو نوع لاگ: **Runtime** و **Build**.
- لاگ **ساختاریافته** قابل جست‌وجو و تحلیل است.
- `event` و `requestId` ردیابی را ساده می‌کنند.
- زمان اجرای بخش‌های کلیدی را اندازه بگیرید.
- Drains برای نگهداری بلندمدت.

---

<a id="فصل-۳"></a>
## فصل ۳: Web Analytics

### مقدمه

**Web Analytics** به شما می‌گوید کاربران چه می‌کنند: از کجا می‌آیند، چه صفحاتی را می‌بینند و کجا می‌روند. تمرکز آن روی **حریم خصوصی** است.

### مفاهیم اساسی

**۱. Web Analytics چیست؟**
یک ابزار تحلیل سبک و حریم‌خصوصی‌محور که بدون کوکی‌های ردیابی کار می‌کند.

**۲. معیارهای اصلی**

| معیار | معنی |
|---|---|
| **Page Views** | تعداد بازدید صفحات |
| **Visitors** | بازدیدکنندگان یکتا |
| **Top Pages** | پرطرفدارترین صفحات |
| **Referrers** | منابع ورودی |
| **Countries** | کشورها |
| **Devices** | دستگاه‌ها |
| **Bounce Rate** | نرخ پرش |

**۳. رویدادهای سفارشی**
می‌توانید رویدادهای دلخواه (کلیک، خرید، ثبت‌نام) را ردیابی کنید.

**۴. حریم خصوصی**
- بدون کوکی ردیابی.
- بدون جمع‌آوری دادهٔ شخصی.
- سازگار با GDPR.

**۵. تفاوت با Google Analytics**

| معیار | Vercel Analytics | Google Analytics |
|---|---|---|
| حریم خصوصی | ✅ بالا | متوسط |
| پیچیدگی | ساده | زیاد |
| دادهٔ تاریخی | محدود | ✅ زیاد |
| رایگان | ✅ (Hobby) | ✅ |

### مثال‌های عملی

**مثال ۱ — نصب**

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

**مثال ۲ — ردیابی رویداد سفارشی**

```tsx
// components/SignupButton.tsx
'use client';

import { track } from '@vercel/analytics';

export default function SignupButton() {
  return (
    <button
      onClick={() => {
        track('signup_click', {
          location: 'hero',
          plan: 'pro',
        });
      }}
    >
      ثبت‌نام
    </button>
  );
}
```

**مثال ۳ — ردیابی در Server Component**

```tsx
// app/products/[id]/page.tsx — با Web Analytics Plus
import { track } from '@vercel/analytics/server';

export default async function Product({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;

  // ردیابی سمت سرور
  await track('product_view', { productId: id });

  const product = await getProduct(id);
  return <div>{product.name}</div>;
}
```

**مثال ۴ — فیلتر کردن ربات‌ها**

```tsx
// app/layout.tsx
import { Analytics } from '@vercel/analytics/next';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="fa" dir="rtl">
      <body>
        {children}
        <Analytics mode={process.env.NODE_ENV === 'production' ? 'production' : 'development'} />
      </body>
    </html>
  );
}
```

### مقایسه و الگوهای مختلف

**سه سطح تحلیل**

| سطح | ابزار | سؤال |
|---|---|---|
| **ترافیک** | Web Analytics | چه کسی آمد؟ |
| **عملکرد** | Speed Insights | چقدر سریع بود؟ |
| **رفتار** | رویدادهای سفارشی | چه کرد؟ |

**رویدادهای پیشنهادی**

| رویداد | کاربرد |
|---|---|
| `signup_click` | علاقه به ثبت‌نام |
| `purchase` | تبدیل |
| `cta_click` | عملکرد دکمه |
| `search` | آنچه کاربران می‌خواهند |
| `error_shown` | تجربهٔ خطا |

### اشکال شایع

**۱. ردیابی همه چیز.**
دادهٔ زیاد و بی‌فایده. فقط رویدادهای مهم.

**۲. نبود نام‌گذاری منظم رویداد.**
`click1`، `click2` بی‌فایده است. از الگوی `object_action` استفاده کنید.

**۳. انتظار دادهٔ فوری.**
تحلیل معمولاً با تأخیر به‌روز می‌شود.

**۴. نادیده گرفتن حریم خصوصی.**
حتی با ابزار حریم‌خصوصی‌محور، قوانین را رعایت کنید.

**۵. مقایسهٔ کورکورانه با GA.**
معیارها متفاوت‌اند؛ مستقیم مقایسه نکنید.

### نکات مهم (Tips & Tricks)

- **نام‌گذاری منظم** رویدادها.
- فقط **رویدادهای مهم** را ردیابی کنید.
- رویدادها را با **خصیصه** همراه کنید (location، plan).
- برای رفتار عمیق، ابزارهای تخصصی را هم در نظر بگیرید.
- داده را **دوره‌ای بررسی** کنید، نه فقط هنگام مشکل.

### خلاصه فصل

- **Web Analytics** تحلیل سبک و حریم‌خصوصی‌محور است.
- معیارها: بازدید، بازدیدکننده، صفحات، منابع، کشورها.
- **رویدادهای سفارشی** رفتار کاربر را نشان می‌دهند.
- نام‌گذاری منظم رویدادها ضروری است.
- فقط رویدادهای مهم را ردیابی کنید.

---

<a id="فصل-۴"></a>
## فصل ۴: Speed Insights

### مقدمه

**Speed Insights** دادهٔ **واقعی** عملکرد را از کاربران شما جمع می‌کند: Core Web Vitals در دنیای واقعی، نه آزمایشگاه.

### مفاهیم اساسی

**۱. Speed Insights چیست؟**
ابزاری که Core Web Vitals و دیگر معیارها را از **کاربران واقعی** (RUM) جمع می‌کند.

**۲. معیارهای گزارش‌شده**

| معیار | معنی |
|---|---|
| **Real Experience Score** | امتیاز کلی تجربه |
| **LCP** | بزرگ‌ترین عنصر |
| **INP** | پاسخ‌دهی |
| **CLS** | پرش چیدمان |
| **FCP** | اولین محتوا |
| **TTFB** | اولین بایت |

**۳. تفکیک‌ها**
- بر اساس **مسیر**.
- بر اساس **کشور**.
- بر اساس **دستگاه**.
- بر اساس **نسخهٔ استقرار**.

**۴. مقایسه با Lighthouse**

| معیار | Speed Insights | Lighthouse |
|---|---|---|
| داده | ✅ واقعی | آزمایشگاهی |
| نمونه | هزاران کاربر | یک بار |
| واقع‌گرایی | ✅ بالا | متوسط |
| تکرارپذیری | متوسط | ✅ بالا |

**۵. نسخه‌های قدیمی در برابر جدید**
Speed Insights نسخه‌های قدیمی و جدید را جدا نشان می‌دهد تا اثر تغییرات را ببینید.

### مثال‌های عملی

**مثال ۱ — نصب**

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

**مثال ۲ — افزودن زمینهٔ سفارشی**

```tsx
// app/layout.tsx
import { SpeedInsights } from '@vercel/speed-insights/next';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="fa" dir="rtl">
      <body>
        {children}
        <SpeedInsights
          route={undefined}
          // می‌توانید داده‌های اضافی ضمیمه کنید
        />
      </body>
    </html>
  );
}
```

**مثال ۳ — تحلیل نتایج**

```
Dashboard → Speed Insights

【 مشاهده 】
- Real Experience Score: ۸۵/۱۰۰ (خوب)
- LCP: ۲.۱s (نیاز به بهبود — هدف < ۲.۵s ✅)
- INP: ۲۴۰ms (نیاز به بهبود — هدف < ۲۰۰ms ❌)
- CLS: ۰.۰۴ (خوب ✅)

【 اقدام 】
- INP بالا → کاهش JavaScript، Server Components بیشتر
- بررسی مسیرهای با امتیاز پایین
- بررسی تفاوت موبایل/دسکتاپ
```

**مثال ۴ — بودجهٔ عملکرد بر اساس داده**

```js
// budget.js — اهداف بر اساس Speed Insights
export const BUDGETS = {
  LCP: 2500,    // ms
  INP: 200,     // ms
  CLS: 0.1,
  TTFB: 800,    // ms
};

export function checkBudget(metrics) {
  const failures = Object.entries(BUDGETS)
    .filter(([key, max]) => metrics[key] > max)
    .map(([key, max]) => `${key}: ${metrics[key]} > ${max}`);

  return { pass: failures.length === 0, failures };
}
```

### مقایسه و الگوهای مختلف

**سه منبع دادهٔ عملکرد**

| منبع | نوع | مزیت |
|---|---|---|
| **Speed Insights** | میدانی | ✅ واقعی |
| **Lighthouse** | آزمایشگاهی | تکرارپذیر |
| **DevTools** | آزمایشگاهی | عمیق |

**اولویت‌بندی بهبود**

| معیار | اولویت | راه‌حل |
|---|---|---|
| LCP بالا | ۱ | تصویر، TTFB |
| INP بالا | ۱ | JS کمتر |
| CLS بالا | ۲ | ابعاد، فونت |
| TTFB بالا | ۲ | کش، Edge |

### اشکال شایع

**۱. انتظار دادهٔ فوری.**
داده پس از جمع شدن نمونه‌ها ظاهر می‌شود.

**۲. قضاوت بر اساس نمونهٔ کم.**
با ترافیک کم، دادهٔ معتبر نیست.

**۳. نادیده گرفتن تفکیک موبایل.**
موبایل معمولاً بدتر است. جدا بررسی کنید.

**۴. تمرکز روی میانگین.**
توزیع (صدک ۷۵) مهم‌تر از میانگین است.

**۵. نبود مقایسهٔ نسخه‌ها.**
بدون مقایسه، اثر تغییرات مشخص نیست.

### نکات مهم (Tips & Tricks)

- **صدک ۷۵** را معیار بگیرید (نه میانگین).
- **موبایل** را جدا بررسی کنید.
- تغییرات را **قبل/بعد** مقایسه کنید.
- روی **مسیرهای پرترافیک** تمرکز کنید.
- با **Lighthouse** برای دیباگ عمیق ترکیب کنید.

### خلاصه فصل

- **Speed Insights** دادهٔ **میدانی** عملکرد را می‌دهد.
- معیارها: **LCP، INP، CLS، FCP، TTFB**.
- تفکیک بر اساس مسیر، کشور، دستگاه، نسخه.
- **صدک ۷۵** معیار درست‌تری از میانگین است.
- با Lighthouse ترکیب کنید.

---

<a id="فصل-۵"></a>
## فصل ۵: Drains — ارسال داده‌ها

### مقدمه

لاگ‌های Vercel نگهداری محدودی دارند. اگر می‌خواهید داده‌ها را بلندمدت نگه دارید یا در ابزار تخصصی تحلیل کنید، **Drains** راه‌حل است.

### مفاهیم اساسی

**۱. Drain چیست؟**
قابلیتی که داده‌های Vercel (لاگ‌ها، متریک‌ها، تراس‌ها) را به یک مقصد بیرونی می‌فرستد.

**۲. انواع Drain**

| نوع | دادهٔ ارسالی |
|---|---|
| **Log Drain** | لاگ‌های Runtime و Build |
| **Trace Drain** | تراس‌های درخواست |
| **Speed Insights Drain** | دادهٔ عملکرد |
| **Analytics Drain** | دادهٔ تحلیل |

**۳. مقصدهای پشتیبانی‌شده**

| مقصد | مناسب برای |
|---|---|
| **Datadog** | مانیتورینگ جامع |
| **Splunk** | تحلیل سازمانی |
| **New Relic** | APM |
| **Elasticsearch** | جست‌وجوی لاگ |
| **Endpoint سفارشی** | هر سرویس HTTP |

**۴. قالب داده**
لاگ‌ها معمولاً به‌صورت **NDJSON** (هر خط یک JSON) ارسال می‌شوند.

**۵. چرا Drains؟**

```
- نگهداری بلندمدت (بیش از مدت پیش‌فرض Vercel)
- جست‌وجوی پیشرفته
- همبستگی با داده‌های دیگر
- هشدار سفارشی
- انطباق قانونی
```

### مثال‌های عملی

**مثال ۱ — ساخت Log Drain**

```
۱. Project → Settings → Drains
۲. «Create Drain» → نوع: Log Drain
۳. مقصد را انتخاب کنید (مثلاً Datadog)
۴. کلید API مقصد را وارد کنید
۵. منابع لاگ را انتخاب کنید (Build، Runtime، Static)
۶. ذخیره
```

**مثال ۲ — Drain به Endpoint سفارشی**

```js
// api/drain-endpoint.js — دریافت لاگ‌های Vercel
import crypto from 'crypto';

export const config = { api: { bodyParser: false } };

function readRawBody(req) {
  return new Promise((resolve, reject) => {
    const chunks = [];
    req.on('data', (c) => chunks.push(c));
    req.on('end', () => resolve(Buffer.concat(chunks)));
    req.on('error', reject);
  });
}

export default async function handler(req, res) {
  const raw = await readRawBody(req);

  // تأیید امضا (اختیاری اما توصیه‌شده)
  const signature = req.headers['x-vercel-signature'];
  const expected = crypto
    .createHmac('sha1', process.env.DRAIN_SECRET)
    .update(raw)
    .digest('hex');

  if (signature && signature !== expected) {
    return res.status(401).end();
  }

  // پردازش لاگ‌ها (NDJSON)
  const lines = raw.toString('utf-8').trim().split('\n');
  const logs = lines.map((l) => {
    try {
      return JSON.parse(l);
    } catch {
      return null;
    }
  }).filter(Boolean);

  for (const log of logs) {
    // فیلتر خطاها و ارسال هشدار
    if (log.level === 'error') {
      await sendAlert(log);
    }
    // ذخیره در دیتابیس
    await saveLog(log);
  }

  res.status(200).json({ received: logs.length });
}
```

**مثال ۳ — فیلتر کردن دادهٔ حساس قبل از ارسال**

```js
// lib/sanitizeLog.js
const SENSITIVE_KEYS = ['password', 'token', 'secret', 'apiKey', 'authorization'];

export function sanitizeLog(log) {
  const clone = { ...log };

  for (const key of Object.keys(clone)) {
    if (SENSITIVE_KEYS.some((s) => key.toLowerCase().includes(s.toLowerCase()))) {
      clone[key] = '[REDACTED]';
    }
  }

  // ماسک کردن ایمیل
  if (clone.email) {
    clone.email = clone.email.replace(/(.{2}).*(@.*)/, '$1***$2');
  }

  return clone;
}
```

### مقایسه و الگوهای مختلف

**سه گزینهٔ نگهداری لاگ**

| گزینه | مدت | هزینه | جست‌وجو |
|---|---|---|---|
| **پیش‌فرض Vercel** | کوتاه | رایگان | محدود |
| **Observability Plus** | بیشتر | اشتراک | ✅ خوب |
| **Drains** | نامحدود | سرویس شما | ✅✅ عالی |

**انتخاب مقصد**

| نیاز | مقصد پیشنهادی |
|---|---|
| مانیتورینگ جامع | Datadog |
| جست‌وجوی لاگ | Elasticsearch |
| هشدار ساده | Endpoint سفارشی |
| سازمانی | Splunk |

### اشکال شایع

**۱. ارسال دادهٔ حساس.**
لاگ‌ها ممکن است حاوی توکن باشند. قبل از ارسال **پاک‌سازی** کنید.

**۲. نبود تأیید امضا در endpoint.**
هر کسی می‌تواند لاگ جعلی بفرستد.

**۳. حجم زیاد و هزینه.**
Drains ممکن است هزینهٔ سرویس مقصد را بالا ببرد. فیلتر کنید.

**۴. ذخیرهٔ همه چیز.**
لازم نیست همه لاگ‌ها ذخیره شوند. فقط مهم‌ها.

**۵. نبود برنامهٔ نگهداری.**
داده‌ها بی‌نهایت رشد می‌کنند. سیاست نگهداری تعیین کنید.

### نکات مهم (Tips & Tricks)

- **پاک‌سازی** دادهٔ حساس قبل از ارسال.
- **امضای** درخواست Drain را تأیید کنید.
- فقط **لاگ‌های مهم** را ذخیره کنید.
- **سیاست نگهداری** تعیین کنید.
- هشدار روی **خطاهای بحرانی** بگذارید.

### خلاصه فصل

- **Drains** داده‌های Vercel را به مقصد بیرونی می‌فرستند.
- انواع: **Log، Trace، Speed Insights، Analytics**.
- مقصدها: Datadog، Splunk، Elasticsearch، Endpoint سفارشی.
- دادهٔ حساس را قبل از ارسال **پاک‌سازی** کنید.
- سیاست نگهداری و هزینه را در نظر بگیرید.

---

<a id="فصل-۶"></a>
## فصل ۶: دیباگ با داده‌های واقعی

### مقدمه

در این فصل، یک سناریوی واقعی را قدم‌به‌قدم دیباگ می‌کنیم تا ببینید ابزارها چطور کنار هم کار می‌کنند.

### سناریو: «سایت کند شده است»

**مرحله ۱ — تأیید مشکل**

```
Dashboard → Speed Insights
- Real Experience Score: ۶۲ (افت از ۸۵)
- LCP: ۴.۲s (بود ۲.۱s)
- تفکیک: فقط موبایل و فقط مسیر /products
```

**مرحله ۲ — بررسی لاگ‌ها**

```bash
# لاگ مسیر مشکوک
vercel logs myapp.com | grep "/products"

# جست‌وجوی کندی
vercel logs myapp.com | grep -i "duration" | jq 'select(.duration > 1000)'
```

**مرحله ۳ — بررسی Analytics**

```
Dashboard → Analytics
- نرخ بازدید /products افزایش یافته (کمپین تبلیغاتی)
- ترافیک از کشورهای دور (بدون کش)
```

**مرحله ۴ — فرضیه و آزمایش**

| فرضیه | آزمایش | نتیجه |
|---|---|---|
| کوئری دیتابیس کند | زمان اجرا در لاگ | ✅ ۲.۸s |
| تصویر بزرگ | بررسی تصویر LCP | ✅ ۱.۲MB |
| کش نداشتن | بررسی هدر | `x-vercel-cache: MISS` |

**مرحله ۵ — رفع**

```tsx
// ۱. کش کردن کوئری
async function getProducts() {
  return fetch('https://api.example.com/products', {
    next: { revalidate: 300, tags: ['products'] },   // ← اضافه شد
  }).then((r) => r.json());
}
```

```tsx
// ۲. بهینه‌سازی تصویر LCP
<Image
  src="/hero.jpg"
  alt=""
  width={1200}
  height={600}
  priority                    // ← اضافه شد
  sizes="100vw"
  quality={75}                // ← از ۱۰۰ به ۷۵
/>
```

**مرحله ۶ — تأیید بهبود**

```
۲۴ ساعت بعد → Speed Insights
- Real Experience Score: ۸۸ ✅
- LCP: ۱.۹s ✅
- x-vercel-cache: HIT ✅
```

### جدول تشخیص سریع

| نشانه | احتمال | بررسی |
|---|---|---|
| LCP بالا | تصویر بزرگ، TTFB | Speed Insights + Network |
| INP بالا | JS زیاد | Bundle Analyzer |
| CLS بالا | فونت/تصویر بدون ابعاد | DevTools |
| TTFB بالا | Cold Start، دیتابیس | Runtime Logs |
| خطای ۵۰۰ | باگ کد | Runtime Logs |
| خطای ۴۰۴ | مسیر اشتباه | Build Logs |
| Build Fail | وابستگی/تایپ | Build Logs |
| کندی ناگهانی | ترافیک، حمله | Firewall Observability |

### چک‌لیست دیباگ

- [ ] مشکل را با **داده** تأیید کردم (نه حدس)
- [ ] **دامنهٔ مشکل** را محدود کردم (کدام مسیر/دستگاه/کشور)
- [ ] **لاگ‌ها** را بررسی کردم
- [ ] **فرضیه** ساختم و آزمایش کردم
- [ ] **رفع** کردم و اندازه گرفتم
- [ ] بهبود را **مستند** کردم

### نکات مهم (Tips & Tricks)

- **همیشه با داده شروع کنید**، نه با حدس.
- مشکل را **محدود** کنید (کدام مسیر/کاربر).
- یک فرضیه در هر زمان آزمایش کنید.
- پس از رفع، **دوباره اندازه** بگیرید.
- **Learning را مستند** کنید.

### خلاصه فصل

- دیباگ حرفه‌ای: **تأیید → محدودسازی → فرضیه → آزمایش → رفع → تأیید**.
- ابزارها کنار هم تصویر کامل می‌دهند.
- جدول تشخیص سریع را به خاطر بسپارید.
- همیشه با داده شروع کنید.
- نتایج را مستند کنید.

---

## منابع و مراجع

### مستندات رسمی
- Vercel Observability: <https://vercel.com/docs/observability>
- Runtime Logs: <https://vercel.com/docs/observability/runtime-logs>
- Build Logs: <https://vercel.com/docs/deployments/logs>
- Web Analytics: <https://vercel.com/docs/analytics>
- Speed Insights: <https://vercel.com/docs/speed-insights>
- Drains: <https://vercel.com/docs/drains>

### مفاهیم پیشنهادی برای مطالعه بیشتر
- Core Web Vitals (web.dev): <https://web.dev/vitals/>
- Datadog: <https://docs.datadoghq.com>
- OpenTelemetry: <https://opentelemetry.io/docs/>

### گام بعدی
- [فایل ۱۴ — پلن‌ها و قیمت‌گذاری](./14-Plans-and-Pricing.md)
- [فایل ۱۶ — عیب‌یابی خطاهای رایج](./16-Troubleshooting.md)
