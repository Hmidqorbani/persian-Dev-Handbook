# ۰۵ — توابع Serverless و Edge

**مدت یادگیری**: ۴ تا ۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: مطالعه فایل‌های ۰۱ تا ۰۴، آشنایی با JavaScript و مفاهیم HTTP

---

## فهرست محتوا

- [فصل ۱: Serverless چیست؟](#فصل-۱)
- [فصل ۲: توابع Serverless در Vercel](#فصل-۲)
- [فصل ۳: Edge Functions و تفاوت آن با Serverless](#فصل-۳)
- [فصل ۴: Runtimeها و انتخاب درست — Node.js یا Edge](#فصل-۴)
- [فصل ۵: محدودیت‌ها، Cold Start و Fluid Compute](#فصل-۵)
- [فصل ۶: مثال‌های عملی کامل](#فصل-۶)

---

<a id="فصل-۱"></a>
## فصل ۱: Serverless چیست؟

### مقدمه

«Serverless» یکی از گیج‌کننده‌ترین واژه‌های دنیای وب است، چون در واقع **سرور وجود دارد** — فقط شما آن را مدیریت نمی‌کنید. در این فصل، این مفهوم را از ریشه و با تشبیه‌های ساده می‌فهمیم.

### مفاهیم اساسی

**۱. تعریف ساده**
**Serverless** یعنی: شما فقط **کد یک تابع** را می‌نویسید و پلتفرم مسئول اجرا، مقیاس‌دهی، نگهداری و سرور است.

> **تشبیه**: تاکسی در برابر ماشین شخصی. با ماشین شخصی (سرور سنتی) باید بنزین، بیمه، تعمیر و پارکینگ را خودتان مدیریت کنید. با تاکسی (Serverless) فقط می‌گویید کجا می‌روید و کرایه را می‌پردازید — راننده، ماشین و تعمیرات به شما ربطی ندارد.

**۲. سه ویژگی اصلی**

| ویژگی | معنی |
|---|---|
| **بدون مدیریت سرور** | نصب، به‌روزرسانی و مقیاس با پلتفرم است |
| **مقیاس خودکار** | از صفر تا هزاران درخواست، خودکار |
| **پرداخت به‌ازای مصرف** | فقط وقتی تابع اجرا می‌شود، هزینه می‌دهید |

**۳. چرخهٔ اجرای یک تابع (Invocation Lifecycle)**
```
درخواست می‌آید
   ↓
آیا نمونهٔ آماده هست؟ (Warm)
   ├── بله → اجرا (سریع)
   └── خیر → ساخت نمونه جدید (Cold Start) → اجرا
   ↓
اجرا و برگرداندن پاسخ
   ↓
نمونه برای مدتی آماده می‌ماند (ممکن است بعداً دوباره استفاده شود)
```

**۴. Stateless بودن (بی‌حالت)**
توابع Serverless **حالت‌دار نیستند**: هر اجرا مستقل است. نمی‌توانید داده را در حافظهٔ تابع نگه دارید. برای ذخیرهٔ داده از **دیتابیس، KV یا Blob** استفاده کنید.

**۵. مفاهیم مرتبط**

| اصطلاح | معنی |
|---|---|
| **Cold Start** | تأخیر اولیهٔ ساخت نمونهٔ جدید |
| **Warm** | نمونهٔ آماده و سریع |
| **Invocation** | یک بار اجرای تابع |
| **Runtime** | محیط اجرای کد (Node.js، Edge، Python...) |
| **Concurrency** | تعداد اجراهای همزمان |

### مثال‌های عملی

**مثال ۱ — تفاوت ذهنی: سرور سنتی در برابر Serverless**

```
【 سرور سنتی 】
- یک سرور همیشه روشن
- برای ترافیک کم: هدررفت منابع
- برای ترافیک زیاد: باید خودتان مقیاس دهید
- هزینه: ثابت (ماهانه)

【 Serverless 】
- تابع فقط در صورت درخواست اجرا می‌شود
- ترافیک کم: هزینهٔ نزدیک صفر
- ترافیک زیاد: خودکار مقیاس می‌شود
- هزینه: به‌ازای مصرف
```

**مثال ۲ — یک تابع Serverless حداقلی**

```js
// api/hello.js
export default function handler(req, res) {
  // این کد فقط زمانی اجرا می‌شود که کسی /api/hello را صدا بزند
  res.status(200).json({ message: 'سلام از Serverless!' });
}
```

**مثال ۳ — نشان دادن Stateless بودن**

```js
// api/counter-broken.js — ❌ اشتباه: حالت در حافظه
let counter = 0;

export default function handler(req, res) {
  counter++;   // ← قابل اعتماد نیست!
  // چون نمونه‌ها ممکن است عوض شوند یا موازی اجرا شوند
  res.json({ counter });
}
```

```js
// api/counter-correct.js — ✅ درست: حالت در KV
import { kv } from '@vercel/kv';

export default async function handler(req, res) {
  const counter = await kv.incr('visits');   // ذخیره‌سازی بیرونی
  res.json({ counter });
}
```

### مقایسه و الگوهای مختلف

**سرور سنتی در برابر Serverless**

| معیار | سرور سنتی | Serverless |
|---|---|---|
| مدیریت | کامل بر عهده شما | بر عهده پلتفرم |
| مقیاس | دستی | خودکار |
| هزینه در ترافیک کم | بالا (سرور همیشه روشن) | نزدیک صفر |
| هزینه در ترافیک زیاد | باید ارتقا دهید | خودکار (اما ممکن است گران شود) |
| Cold Start | ندارد | دارد |
| حالت (State) | قابل نگهداری | ❌ (Stateless) |
| اجرای طولانی | ✅ | محدود |
| مناسب برای | پردازش مداوم، WebSocket | API، پردازش رویداد |

**سه مدل محاسباتی در Vercel**

| مدل | توضیح | مثال |
|---|---|---|
| **Static** | فایل‌های از پیش ساخته | صفحهٔ HTML، تصویر |
| **Serverless Functions** | کد در دیتاسنتر منطقه‌ای | کوئری دیتابیس، پردازش |
| **Edge Functions** | کد روی لبه (نزدیک کاربر) | ریدایرکت، احراز هویت |

### اشکال شایع

**۱. تصور اینکه «Serverless یعنی بدون هزینه».**
هزینه دارد، اما فقط به‌ازای مصرف. در ترافیک بسیار بالا می‌تواند گران شود.

**۲. نگه‌داشتن حالت در حافظه.**
توابع Stateless هستند. برای دادهٔ پایدار از دیتابیس/KV استفاده کنید.

**۳. اجرای کارهای طولانی در Serverless.**
محدودیت زمان وجود دارد. کارهای طولانی را به سرویس‌های تخصصی بسپارید.

**۴. اشتباه گرفتن Serverless با «بدون سرور».**
سرور هست؛ فقط مدیریتش با شما نیست.

**۵. نادیده گرفتن Cold Start.**
برای مسیرهای حساس، از Edge یا Fluid Compute استفاده کنید.

### نکات مهم (Tips & Tricks)

- **Stateless را بپذیرید**: هر داده‌ای که باید بماند، در سرویس ذخیره‌سازی.
- برای اتصال دیتابیس، از **Connection Pooling** استفاده کنید (توابع زیاد = اتصال زیاد).
- برای پاسخ‌های پرتکرار، از **کش لبه** استفاده کنید.
- Cold Start را با **Fluid Compute** و نگه‌داشتن توابع سبک کاهش دهید.
- منطق را کوچک و متمرکز نگه دارید؛ توابع بزرگ کندتر هستند.

### خلاصه فصل

- **Serverless** = اجرای کد بدون مدیریت سرور، با مقیاس خودکار و پرداخت به‌ازای مصرف.
- توابع **Stateless** هستند؛ داده را در سرویس‌های بیرونی نگه دارید.
- **Cold Start** تأخیر اولیهٔ ساخت نمونهٔ جدید است.
- سه مدل در Vercel: **Static، Serverless، Edge**.
- مناسب برای API و پردازش رویداد؛ نه کارهای طولانی و حالت‌دار.

---

<a id="فصل-۲"></a>
## فصل ۲: توابع Serverless در Vercel

### مقدمه

حالا که مفهوم را می‌دانیم، ببینیم در Vercel چطور یک تابع Serverless می‌سازیم. ساده‌ترین راه: یک فایل داخل پوشهٔ `api/` بگذارید — تمام!

### مفاهیم اساسی

**۱. ساختار پوشهٔ `api/`**
هر فایل داخل `api/` به یک endpoint تبدیل می‌شود:

```
api/
├── hello.js            → /api/hello
├── users.js            → /api/users
├── users/
│   ├── index.js        → /api/users
│   └── [id].js         → /api/users/123
└── posts/
    └── [...slug].js    → /api/posts/a/b/c
```

**۲. امضای تابع**
```js
export default function handler(request, response) { }
```
- `request`: اطلاعات درخواست (متد، هدرها، بدنه، کوئری).
- `response`: ابزار پاسخ (`res.json()`, `res.status()`, `res.send()`).

**۳. تنظیمات تابع**
با `export const config`:

```js
export const config = {
  runtime: 'nodejs',        // یا 'edge'
  maxDuration: 30,          // حداکثر ثانیه
  memory: 1024,             // مگابایت
  regions: ['iad1', 'fra1'], // منطقه‌ها
};
```

**۴. Region (منطقه)**
به‌طور پیش‌فرض، توابع در یک منطقهٔ پیش‌فرض اجرا می‌شوند. برای کاربران ایرانی/اروپایی، منطقهٔ نزدیک‌تر (مثل `fra1` فرانکفورت) تأخیر را کم می‌کند.

| کد منطقه | محل |
|---|---|
| `iad1` | واشینگتن (پیش‌فرض) |
| `fra1` | فرانکفورت |
| `cdg1` | پاریس |
| `dxb1` | دبی |
| `hnd1` | توکیو |
| `sfo1` | سان‌فرانسیسکو |

**۵. مسیرهای پویا**
نام فایل با `[param]` پارامتر می‌گیرد:
```js
// api/users/[id].js
export default function handler(req, res) {
  const { id } = req.query;
  res.json({ userId: id });
}
```

### مثال‌های عملی

**مثال ۱ — تابع با متدهای مختلف**

```js
// api/todos.js
let todos = [
  { id: 1, title: 'یادگیری Vercel', done: false },
];

export default function handler(req, res) {
  // CORS (برای فراخوانی از دامنه‌های دیگر)
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET,POST,PUT,DELETE,OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type');

  if (req.method === 'OPTIONS') {
    return res.status(200).end();
  }

  switch (req.method) {
    case 'GET':
      return res.status(200).json(todos);

    case 'POST':
      // بدنه به‌طور خودکار parse می‌شود (اگر JSON باشد)
      const newTodo = { id: Date.now(), ...req.body, done: false };
      todos.push(newTodo);
      return res.status(201).json(newTodo);

    case 'DELETE':
      todos = [];
      return res.status(200).json({ message: 'همه پاک شد' });

    default:
      res.setHeader('Allow', ['GET', 'POST', 'DELETE']);
      return res.status(405).json({ error: 'متد مجاز نیست' });
  }
}
```

**مثال ۲ — مسیر پویا با اعتبارسنجی**

```js
// api/users/[id].js
export default async function handler(req, res) {
  const { id } = req.query;

  // اعتبارسنجی ساده
  if (!/^\d+$/.test(id)) {
    return res.status(400).json({ error: 'شناسه باید عدد باشد' });
  }

  // در عمل: خواندن از دیتابیس
  const user = { id: Number(id), name: `کاربر ${id}`, email: `user${id}@example.com` };

  if (!user) {
    return res.status(404).json({ error: 'کاربر پیدا نشد' });
  }

  return res.status(200).json(user);
}
```

**مثال ۳ — تنظیم حافظه و مدت اجرا در `vercel.json`**

```json
{
  "functions": {
    "api/report.js": {
      "memory": 1024,
      "maxDuration": 60,
      "regions": ["fra1"]
    }
  }
}
```

**مثال ۴ — تابع با کلید مخفی (امن)**

```js
// api/translate.js
export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'فقط POST' });
  }

  const { text, target } = req.body;

  if (!text) {
    return res.status(400).json({ error: 'متن الزامی است' });
  }

  try {
    // کلید فقط روی سرور است؛ در مرورگر افشا نمی‌شود
    const response = await fetch('https://api.example-translator.com/translate', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${process.env.TRANSLATE_API_KEY}`,
      },
      body: JSON.stringify({ text, target: target || 'en' }),
    });

    if (!response.ok) {
      return res.status(502).json({ error: 'خطا در سرویس ترجمه' });
    }

    const data = await response.json();
    return res.status(200).json({ translated: data.result });
  } catch (err) {
    console.error(err);
    return res.status(500).json({ error: 'خطای داخلی سرور' });
  }
}
```

### مقایسه و الگوهای مختلف

**دو سبک نوشتن تابع در Vercel**

| سبک | مسیر | امضا | مناسب برای |
|---|---|---|---|
| **Standalone Functions** | `api/*.js` | `(req, res)` | پروژه‌های غیر Next.js |
| **Route Handlers** | `app/api/*/route.ts` | `(request: Request)` | پروژه‌های Next.js App Router |

**سه راه تنظیم توابع**

| راه | محل | مناسب برای |
|---|---|---|
| `export const config` | داخل فایل تابع | تنظیم مخصوص یک تابع |
| `vercel.json` → `functions` | فایل تنظیمات | تنظیم گروهی |
| Dashboard → Settings → Functions | رابط وب | تنظیم سریع پیش‌فرض پروژه |

### اشکال شایع

**۱. فراموش کردن `await` در تابع `async`.**
پاسخ قبل از اتمام کار برگردانده می‌شود و ممکن است خطا بدهد.

**۲. استفادهٔ زیاد از اتصال دیتابیس بدون Pooling.**
با هر اجرا یک اتصال جدید باز می‌شود و دیتابیس اشباع می‌شود. از **Connection Pooler** استفاده کنید.

**۳. برگرداندن اشیای غیرقابل سریال‌سازی.**
`res.json()` باید دادهٔ قابل تبدیل به JSON بگیرد.

**۴. نادیده گرفتن `OPTIONS` برای CORS.**
درخواست‌های Preflight شکست می‌خورند.

**۵. لو دادن جزئیات خطا به کاربر.**
در Production، پیام خطای دقیق را در لاگ بگذارید و پیام عمومی برگردانید.

**۶. انتخاب منطقهٔ نامناسب.**
کاربران ایرانی با منطقهٔ `fra1` (فرانکفورت) یا `dxb1` (دبی) تجربهٔ بهتری دارند.

### نکات مهم (Tips & Tricks)

- همیشه **متد HTTP** را بررسی کنید و برای متد نامعتبر `405` برگردانید.
- از **`process.env`** برای کلیدها استفاده کنید؛ هرگز آن‌ها را در کد ننویسید.
- ورودی‌ها را **اعتبارسنجی** کنید (ترجیحاً با Zod یا Joi).
- برای کارهای سنگین، `memory` و `maxDuration` را افزایش دهید.
- لاگ‌های ساختاریافته (JSON) بنویسید تا در Observability قابل جست‌وجو باشند.

### خلاصه فصل

- پوشهٔ `api/` هر فایل را به یک endpoint تبدیل می‌کند.
- امضای تابع: `(req, res)`؛ با `export const config` تنظیم می‌شود.
- **Region** روی تأخیر تأثیر مستقیم دارد.
- کلیدهای حساس فقط سمت سرور.
- همیشه متد و ورودی را بررسی و خطا را مدیریت کنید.

---

<a id="فصل-۳"></a>
## فصل ۳: Edge Functions و تفاوت آن با Serverless

### مقدمه

Serverless Functions در یک منطقهٔ مشخص اجرا می‌شوند. اما اگر کاربر شما در ایران باشد و تابع در آمریکا، تأخیر محسوس می‌شود. **Edge Functions** این مشکل را حل می‌کنند: کد روی **صدها نقطهٔ نزدیک به کاربر** اجرا می‌شود.

### مفاهیم اساسی

**۱. Edge Function چیست؟**
تابعی که روی **Edge Runtime** (مبتنی بر V8 Isolate) اجرا می‌شود و در سراسر شبکهٔ جهانی Vercel توزیع شده است.

> **تشبیه**: Serverless مثل یک شعبهٔ مرکزی در یک شهر است. Edge مثل هزار شعبهٔ کوچک در سراسر جهان که همه دقیقاً همان کار را انجام می‌دهند.

**۲. V8 Isolate در برابر Container**

| معیار | Serverless (Node) | Edge (Isolate) |
|---|---|---|
| واحد اجرا | کانتینر (سنگین) | Isolate (سبک) |
| زمان شروع | بیشتر | چند میلی‌ثانیه |
| مصرف حافظه | زیاد | کم |
| مقیاس | خوب | عالی |

**۳. API های موجود در Edge**
Edge بر پایهٔ **استانداردهای وب** است:

| موجود | توضیح |
|---|---|
| `Request` / `Response` | استاندارد Fetch API |
| `fetch()` | درخواست شبکه |
| `Headers` | مدیریت هدرها |
| `URL` | پردازش آدرس |
| `crypto` | رمزنگاری |
| `TextEncoder/Decoder` | پردازش متن |

**۴. محدودیت‌های Edge**

| محدودیت | توضیح |
|---|---|
| بدون `fs` | دسترسی به فایل سیستم وجود ندارد |
| بدون API های اختصاصی Node | مثل `Buffer`، `process` کامل |
| کتابخانه‌های محدود | بعضی npm packages کار نمی‌کنند |
| حجم کد | محدود (حدود ۱ مگابایت) |
| مدت اجرا | کوتاه‌تر از Serverless |
| بدون اتصال مستقیم TCP | بعضی دیتابیس‌ها کار نمی‌کنند |

### مثال‌های عملی

**مثال ۱ — یک Edge Function با اطلاعات جغرافیایی**

```js
// api/geo.js
export const config = { runtime: 'edge' };

export default function handler(request) {
  // در Edge به اطلاعات جغرافیایی کاربر دسترسی داریم
  const { country, city, region } = request.geo ?? {};

  return new Response(
    JSON.stringify({
      country: country ?? 'نامشخص',
      city: city ?? 'نامشخص',
      region: region ?? 'نامشخص',
      region_edge: process.env.VERCEL_REGION,
    }),
    {
      status: 200,
      headers: {
        'content-type': 'application/json; charset=utf-8',
        'cache-control': 'no-store',
      },
    }
  );
}
```

**مثال ۲ — پاسخ سریع با کش لبه**

```js
// api/quote.js — نقل‌قول تصادفی با کش ۶۰ ثانیه‌ای
export const config = { runtime: 'edge' };

const quotes = [
  'سادگی، نهایت پیچیدگی است.',
  'هر روز یک قدم کوچک بهتر از هیچ است.',
  'کد خوب، کدی است که دیگران بفهمند.',
];

export default function handler() {
  const quote = quotes[Math.floor(Math.random() * quotes.length)];

  return new Response(JSON.stringify({ quote }), {
    headers: {
      'content-type': 'application/json',
      'cache-control': 'public, s-maxage=60, stale-while-revalidate=300',
    },
  });
}
```

**مثال ۳ — Edge Function در Next.js (Route Handler)**

```ts
// app/api/edge-info/route.ts
import { NextResponse } from 'next/server';

export const runtime = 'edge';       // ← اجرا روی Edge

export async function GET(request: Request) {
  const country = request.headers.get('x-vercel-ip-country') ?? 'نامشخص';

  return NextResponse.json({
    country,
    message: 'این پاسخ از نزدیک‌ترین نقطه به شما آمد!',
  });
}
```

**مثال ۴ — A/B Testing سریع روی لبه**

```js
// api/ab.js
export const config = { runtime: 'edge' };

export default function handler(request) {
  const cookie = request.headers.get('cookie') ?? '';
  const existing = cookie.match(/variant=(\w+)/)?.[1];

  if (existing) {
    return new Response(JSON.stringify({ variant: existing }), {
      headers: { 'content-type': 'application/json' },
    });
  }

  // انتخاب تصادفی نسخه
  const variant = Math.random() < 0.5 ? 'A' : 'B';

  return new Response(JSON.stringify({ variant }), {
    headers: {
      'content-type': 'application/json',
      'set-cookie': `variant=${variant}; Path=/; Max-Age=86400`,
    },
  });
}
```

### مقایسه و الگوهای مختلف

**Edge در برابر Serverless — جدول کامل**

| معیار | Serverless Functions | Edge Functions |
|---|---|---|
| محل اجرا | منطقه‌ای (چند دیتاسنتر) | جهانی (صدها نقطه) |
| Latency | متوسط | بسیار کم |
| Cold Start | دارد | تقریباً ندارد |
| Runtime | Node.js کامل | V8 Isolate (Web APIs) |
| کتابخانه‌های npm | ✅ تقریباً همه | ⚠️ محدود |
| دسترسی فایل سیستم | ✅ | ❌ |
| اتصال دیتابیس سنتی | ✅ | ⚠️ محدود |
| مدت اجرا | طولانی‌تر | کوتاه |
| هزینه | به‌ازای زمان CPU | به‌ازای درخواست |
| مناسب برای | دیتابیس، پردازش | احراز هویت، ریدایرکت، Geo، A/B |

**قاعدهٔ انتخاب**

```
کار سنگین + دیتابیس + کتابخانه Node  →  Serverless
پاسخ سریع + منطق سبک + Latency مهم   →  Edge
```

### اشکال شایع

**۱. استفاده از Edge برای کوئری دیتابیس.**
بسیاری از دیتابیس‌ها در Edge کار نمی‌کنند (بدون TCP). از HTTP-based یا Serverless استفاده کنید.

**۲. ایمپورت کتابخانه‌های Node در Edge.**
خطای «module not found» می‌گیرید. فقط کتابخانه‌های سازگار با Web APIs.

**۳. انتظار `process.env` کامل در Edge.**
فقط متغیرهای محیطی مشخص در دسترس‌اند.

**۴. نوشتن منطق سنگین در Edge.**
Edge برای پاسخ‌های سریع و سبک است، نه پردازش سنگین.

**۵. نادیده گرفتن محدودیت حجم کد.**
بستهٔ بزرگ Edge باعث خطای Build می‌شود.

### نکات مهم (Tips & Tricks)

- برای **احراز هویت** و **ریدایرکت**، Edge بهترین انتخاب است.
- در Edge از **`cache-control`** استفاده کنید تا پاسخ‌ها روی لبه کش شوند.
- اطلاعات جغرافیایی با هدر `x-vercel-ip-country` در دسترس است.
- برای کار با دیتابیس، **Serverless** را ترجیح دهید.
- Edge برای **Middleware** هم استفاده می‌شود (فایل ۰۶).

### خلاصه فصل

- **Edge Functions** روی صدها نقطهٔ جهانی اجرا می‌شوند و Latency را کم می‌کنند.
- بر پایهٔ **V8 Isolate** و **Web APIs** هستند.
- محدودیت‌هایی دارند: بدون `fs`، کتابخانه‌های محدود، مدت کوتاه.
- برای منطق سبک و سریع عالی‌اند؛ برای کار سنگین و دیتابیس، Serverless بهتر است.
- در Next.js با `export const runtime = 'edge'` فعال می‌شوند.

---

<a id="فصل-۴"></a>
## فصل ۴: Runtimeها و انتخاب درست — Node.js یا Edge

### مقدمه

«کدام Runtime را انتخاب کنم؟» یکی از تصمیم‌های کلیدی است که روی سرعت، هزینه و حتی امکان‌پذیری پروژه اثر می‌گذارد. در این فصل، معیارهای انتخاب را دقیق بررسی می‌کنیم.

### مفاهیم اساسی

**۱. Runtime چیست؟**
محیطی که کد شما در آن اجرا می‌شود. در Vercel دو Runtime اصلی داریم:
- **Node.js Runtime**: محیط کامل Node.js.
- **Edge Runtime**: محیط سبک بر پایهٔ Web APIs.

**۲. Runtimeهای دیگر**
Vercel از Runtimeهای دیگری هم پشتیبانی می‌کند:
- **Python** (برای توابع Python)
- **Ruby**
- **Go** (با تنظیم خاص)

اما در این کتابچه روی Node.js و Edge تمرکز می‌کنیم.

**۳. معیارهای انتخاب**

| سؤال | اگر بله → |
|---|---|
| به دیتابیس سنتی (TCP) وصل می‌شوم؟ | Node.js |
| از کتابخانه‌ای با API های Node استفاده می‌کنم؟ | Node.js |
| پردازش سنگین یا طولانی دارم؟ | Node.js |
| فقط می‌خواهم سریع پاسخ دهم؟ | Edge |
| احراز هویت، ریدایرکت، Geo می‌خواهم؟ | Edge |
| Latency برایم حیاتی است؟ | Edge |

**۴. Runtime پیش‌فرض**
در Vercel، پیش‌فرض **Node.js** است. برای Edge باید صریحاً مشخص کنید:
```js
export const config = { runtime: 'edge' };    // در توابع api/
```
```ts
export const runtime = 'edge';                // در Next.js Route Handlers
```

**۵. آیا می‌توان ترکیب کرد؟**
بله! در یک پروژه می‌توانید بعضی مسیرها را Edge و بعضی را Node.js کنید. این یک الگوی رایج و حرفه‌ای است.

### مثال‌های عملی

**مثال ۱ — ترکیب Edge و Node در یک پروژه**

```
app/api/
├── auth/route.ts        → Edge (سریع، بررسی توکن)
├── products/route.ts    → Node.js (کوئری دیتابیس)
├── geo/route.ts         → Edge (اطلاعات جغرافیایی)
└── report/route.ts      → Node.js (پردازش سنگین)
```

```ts
// app/api/auth/route.ts — Edge برای سرعت
import { NextResponse } from 'next/server';
export const runtime = 'edge';

export async function GET(request: Request) {
  const token = request.headers.get('authorization');
  if (!token) {
    return NextResponse.json({ error: 'توکن ندارد' }, { status: 401 });
  }
  // بررسی سریع توکن (بدون دیتابیس)
  const valid = token === `Bearer ${process.env.STATIC_TOKEN}`;
  return NextResponse.json({ valid });
}
```

```ts
// app/api/products/route.ts — Node.js برای دیتابیس
import { NextResponse } from 'next/server';
import { db } from '@/lib/db';
export const runtime = 'nodejs';      // پیش‌فرض

export async function GET() {
  const products = await db.query('SELECT * FROM products LIMIT 20');
  return NextResponse.json(products);
}
```

**مثال ۲ — تشخیص Runtime در زمان اجرا**

```js
// api/whoami.js
export default function handler(req, res) {
  res.json({
    runtime: process.env.NEXT_RUNTIME ?? 'nodejs',
    node_version: process.version,
    region: process.env.VERCEL_REGION,
    platform: process.platform,
  });
}
```

**مثال ۳ — استفاده از کتابخانهٔ Node (فقط در Node Runtime)**

```js
// api/hash.js — استفاده از crypto ماژول Node
import crypto from 'crypto';

export default function handler(req, res) {
  const { text } = req.query;
  if (!text) return res.status(400).json({ error: 'متن لازم است' });

  const hash = crypto.createHash('sha256').update(text).digest('hex');
  res.json({ hash });
}
```

> ⚠️ همین کد در Edge کار نمی‌کند، چون ماژول `crypto` Node در دسترس نیست. در Edge باید از `crypto.subtle` استفاده کنید.

### مقایسه و الگوهای مختلف

**جدول تصمیم نهایی**

| سناریو | Runtime پیشنهادی | دلیل |
|---|---|---|
| کوئری Postgres/MySQL | **Node.js** | اتصال TCP |
| احراز هویت با JWT ساده | **Edge** | سرعت، بدون دیتابیس |
| آپلود فایل به Blob | **Node.js** | پردازش فایل |
| ریدایرکت بر اساس کشور | **Edge** | Geo + سرعت |
| تولید PDF | **Node.js** | کتابخانه‌های سنگین |
| پاسخ از کش | **Edge** | Latency کم |
| پردازش تصویر | **Node.js** | کتابخانه‌های native |
| Webhook ساده | **Node.js** | پردازش کامل |

**هزینه: Edge در برابر Node**

| معیار | Edge | Node.js |
|---|---|---|
| مدل هزینه | به‌ازای درخواست + زمان | به‌ازای زمان CPU/GB-s |
| در ترافیک کم | ارزان | ارزان |
| در ترافیک بسیار زیاد | ممکن است گران شود | بسته به مصرف CPU |
| بهینه‌سازی | کش لبه | Fluid Compute |

### اشکال شایع

**۱. انتخاب Edge بدون بررسی سازگاری کتابخانه‌ها.**
بعد از Build متوجه خطا می‌شوید. قبل از انتخاب، سازگاری کتابخانه را بررسی کنید.

**۲. استفاده از Node برای همه چیز.**
Latency بالاتر و Cold Start بیشتر. برای منطق سبک، Edge بهتر است.

**۳. اشتباه در امضای تابع.**
در Node از `(req, res)` و در Edge از `(request)` با `Response` استفاده کنید — قابل تعویض نیستند.

**۴. فرض اینکه `process.env` در Edge کامل است.**
در Edge فقط متغیرهای مشخص در دسترس‌اند.

**۵. نادیده گرفتن تفاوت در تست محلی.**
`vercel dev` هر دو Runtime را شبیه‌سازی می‌کند؛ از آن استفاده کنید.

### نکات مهم (Tips & Tricks)

- **پیش‌فرض Node.js**؛ فقط در صورت نیاز به Edge بروید.
- برای مسیرهای **حساس به Latency** (auth، redirect)، Edge انتخاب کنید.
- Runtime را **مستند کنید** تا تیم بداند چرا هر مسیر کدام Runtime را دارد.
- برای دیتابیس، همیشه **Connection Pooling** را در نظر بگیرید.
- عملکرد هر Runtime را با **Speed Insights** و لاگ‌ها اندازه بگیرید.

### خلاصه فصل

- دو Runtime اصلی: **Node.js** (کامل) و **Edge** (سبک، جهانی).
- Node برای دیتابیس، کتابخانه‌های سنگین و پردازش طولانی.
- Edge برای Latency کم، احراز هویت، ریدایرکت و Geo.
- می‌توانید در یک پروژه **ترکیب** کنید.
- انتخاب Runtime باید آگاهانه و مستند باشد.

---

<a id="فصل-۵"></a>
## فصل ۵: محدودیت‌ها، Cold Start و Fluid Compute

### مقدمه

توابع Serverless عالی هستند، اما محدودیت‌هایی دارند. بزرگ‌ترینشان **Cold Start** است. در این فصل، محدودیت‌ها را می‌شناسیم و با **Fluid Compute** — راهکار مدرن Vercel — آشنا می‌شویم.

### مفاهیم اساسی

**۱. Cold Start چیست؟**
وقتی تابعی مدتی اجرا نشده باشد، نمونه‌اش از حافظه پاک می‌شود. درخواست بعدی باید نمونهٔ جدید بسازد → **تأخیر چند صد میلی‌ثانیه‌ای تا چند ثانیه**.

```
درخواست ۱ (بعد از مدت بی‌کاری):
   ساخت نمونه (۵۰۰ms) + اجرا (۱۰۰ms) = ۶۰۰ms  ← Cold Start

درخواست ۲ (بلافاصله بعد):
   اجرا (۱۰۰ms) = ۱۰۰ms                       ← Warm
```

**۲. Fluid Compute**
یک مدل اجرایی مدرن که Vercel معرفی کرده و برای پروژه‌های جدید پیش‌فرض است. مزایا:

| مزیت | توضیح |
|---|---|
| **کاهش Cold Start** | نمونه‌ها بیشتر و بهتر بازاستفاده می‌شوند |
| **همزمانی درون‌تابعی (In-function Concurrency)** | یک نمونه چند درخواست را همزمان سرو می‌کند |
| **صرفه‌جویی در هزینه** | پرداخت بر پایهٔ زمان CPU فعال، نه کل زمان انتظار |
| **بهینه برای AI** | مناسب بارهای استریم و طولانی |

**۳. محدودیت‌های اصلی توابع**

| محدودیت | مقدار تقریبی | یادداشت |
|---|---|---|
| **حداکثر مدت اجرا** | طبق پلن (Hobby کمتر، Pro/Enterprise بیشتر) | قابل تنظیم |
| **حافظه** | از ۱۲۸MB تا چند GB | قابل تنظیم |
| **حجم بستهٔ تابع** | حدود ۲۵۰MB (فشرده‌نشده) | بستهٔ بزرگ = Cold Start بیشتر |
| **پهنای باند پاسخ** | محدود | برای فایل‌های بزرگ از Blob استفاده کنید |
| **تعداد فایل‌های تابع** | محدود در هر استقرار | برای پروژه‌های بزرگ مهم است |

> **نکته**: مقادیر دقیق با پلن و زمان تغییر می‌کنند؛ همیشه صفحهٔ Limits مستندات Vercel را بررسی کنید.

**۴. عوامل مؤثر بر Cold Start**
- **حجم بسته** (وابستگی‌های زیاد = کندتر).
- **زبان و Runtime** (Node کندتر از Edge).
- **تعداد نمونه‌های همزمان**.
- **فاصلهٔ زمانی بین درخواست‌ها**.

### مثال‌های عملی

**مثال ۱ — کاهش Cold Start با سبک کردن وابستگی‌ها**

```js
// ❌ بد: ایمپورت کل کتابخانه
import _ from 'lodash';                 // حجم زیاد
const result = _.chunk([1,2,3,4], 2);

// ✅ خوب: ایمپورت فقط بخش لازم
import chunk from 'lodash/chunk';       // حجم کم
const result = chunk([1,2,3,4], 2);
```

**مثال ۲ — تنظیم Fluid Compute و حافظه**

```json
// vercel.json
{
  "functions": {
    "api/ai-stream.js": {
      "memory": 1769,
      "maxDuration": 300
    }
  }
}
```

> Fluid Compute در تنظیمات پروژه فعال/غیرفعال می‌شود؛ برای پروژه‌های جدید به‌طور پیش‌فرض فعال است.

**مثال ۳ — Warm-up خودکار با Cron**

```json
// vercel.json — هر ۵ دقیقه تابع را بیدار نگه دار
{
  "crons": [
    { "path": "/api/warmup", "schedule": "*/5 * * * *" }
  ]
}
```

```js
// api/warmup.js
export default function handler(req, res) {
  // یک تابع سبک که نمونه را Warm نگه می‌دارد
  res.status(200).json({ ok: true, ts: Date.now() });
}
```

**مثال ۴ — استفاده از Edge برای حذف Cold Start**

```js
// api/ping.js — به‌جای Node، از Edge استفاده کنید
export const config = { runtime: 'edge' };

export default function handler() {
  return new Response(JSON.stringify({ pong: true }), {
    headers: { 'content-type': 'application/json' },
  });
}
// Edge عملاً Cold Start محسوسی ندارد
```

### مقایسه و الگوهای مختلف

**Serverless سنتی در برابر Fluid Compute**

| معیار | Serverless سنتی | Fluid Compute |
|---|---|---|
| مدل اجرا | یک درخواست = یک نمونه | یک نمونه = چند درخواست |
| Cold Start | بیشتر | کمتر |
| همزمانی | بین نمونه‌ها | درون نمونه + بین نمونه‌ها |
| هزینه | بر پایهٔ زمان تخصیص | بر پایهٔ CPU فعال |
| مناسب برای | API های سبک | API + AI + استریم |

**راه‌های کاهش Cold Start**

| راه | اثر | هزینه |
|---|---|---|
| سبک کردن بسته | متوسط | صفر |
| استفاده از Edge | زیاد | تغییر Runtime |
| Fluid Compute | زیاد | — |
| Warm-up با Cron | متوسط | مصرف منابع |
| افزایش حافظه | کم تا متوسط | هزینه بیشتر |

### اشکال شایع

**۱. ایمپورت کتابخانه‌های سنگین بدون نیاز.**
هر مگابایت اضافه، Cold Start را کندتر می‌کند.

**۲. تنظیم `maxDuration` بیش از حد بالا.**
ممکن است در پلن شما مجاز نباشد و Build خطا بدهد.

**۳. انتظار نبود Cold Start در Node.**
Cold Start بخشی از مدل Serverless است؛ فقط می‌توان آن را کم کرد.

**۴. نادیده گرفتن هزینهٔ Warm-up.**
اگر با Cron مداوم تابع را بیدار نگه دارید، مصرف بالا می‌رود.

**۵. استفاده از حافظهٔ بسیار زیاد برای تابع سبک.**
هزینهٔ بی‌مورد. حافظه را متناسب با نیاز تنظیم کنید.

### نکات مهم (Tips & Tricks)

- **بسته را سبک نگه دارید** — مؤثرترین راه کاهش Cold Start.
- برای مسیرهای حساس، **Edge** انتخاب کنید (تقریباً بدون Cold Start).
- **Fluid Compute** را برای پروژه‌های جدید فعال نگه دارید.
- از **Cron** برای Warm-up فقط در ساعات پرترافیک استفاده کنید.
- برای بارهای سنگین (مثل AI)، `maxDuration` و `memory` را بالا ببرید.

### خلاصه فصل

- **Cold Start** تأخیر ساخت نمونهٔ جدید است و بخشی از مدل Serverless.
- **Fluid Compute** Cold Start را کم و هزینه را بهینه می‌کند.
- محدودیت‌ها: مدت اجرا، حافظه، حجم بسته، پهنای باند.
- سبک کردن بسته و استفاده از Edge، مؤثرترین راه‌های بهبودند.
- مقادیر دقیق را در مستندات Limits بررسی کنید.

---

<a id="فصل-۶"></a>
## فصل ۶: مثال‌های عملی کامل

### مقدمه

در این فصل، چند مثال کامل و قابل اجرا را می‌سازیم که می‌توانید مستقیماً در پروژهٔ خود استفاده کنید: یک API کامل CRUD، یک پروکسی امن، یک تابع با کش و یک استریم پاسخ.

### مثال ۱ — API کامل CRUD با اعتبارسنجی

ساختار:
```
crud-api/
├── api/
│   └── items/
│       ├── index.js       → GET (لیست) + POST (ساخت)
│       └── [id].js        → GET + PUT + DELETE
└── package.json
```

```js
// api/items/index.js
// ذخیره‌سازی موقت در حافظه (فقط برای نمونه؛ در عمل از دیتابیس استفاده کنید)
let items = [
  { id: '1', name: 'آیتم اول', createdAt: new Date().toISOString() },
];

export default function handler(req, res) {
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET,POST,OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type');

  if (req.method === 'OPTIONS') return res.status(200).end();

  if (req.method === 'GET') {
    return res.status(200).json({ count: items.length, items });
  }

  if (req.method === 'POST') {
    const { name } = req.body ?? {};
    if (!name || typeof name !== 'string' || name.trim().length === 0) {
      return res.status(400).json({ error: 'نام الزامی و باید متن باشد' });
    }
    const item = {
      id: String(Date.now()),
      name: name.trim(),
      createdAt: new Date().toISOString(),
    };
    items.push(item);
    return res.status(201).json(item);
  }

  res.setHeader('Allow', ['GET', 'POST', 'OPTIONS']);
  return res.status(405).json({ error: 'متد مجاز نیست' });
}
```

```js
// api/items/[id].js
let items = [
  { id: '1', name: 'آیتم اول', createdAt: new Date().toISOString() },
];

export default function handler(req, res) {
  const { id } = req.query;
  const index = items.findIndex((i) => i.id === id);

  if (index === -1) {
    return res.status(404).json({ error: 'آیتم پیدا نشد' });
  }

  switch (req.method) {
    case 'GET':
      return res.status(200).json(items[index]);

    case 'PUT': {
      const { name } = req.body ?? {};
      if (!name) return res.status(400).json({ error: 'نام الزامی است' });
      items[index] = { ...items[index], name, updatedAt: new Date().toISOString() };
      return res.status(200).json(items[index]);
    }

    case 'DELETE':
      items.splice(index, 1);
      return res.status(204).end();

    default:
      res.setHeader('Allow', ['GET', 'PUT', 'DELETE']);
      return res.status(405).json({ error: 'متد مجاز نیست' });
  }
}
```

### مثال ۲ — پروکسی امن (پنهان‌کردن کلید API)

```js
// api/proxy/github.js
// کلاینت به این endpoint می‌زند؛ کلید GitHub هرگز افشا نمی‌شود
export default async function handler(req, res) {
  const { username } = req.query;

  if (!username) {
    return res.status(400).json({ error: 'نام کاربری لازم است' });
  }

  try {
    const response = await fetch(`https://api.github.com/users/${username}`, {
      headers: {
        Accept: 'application/vnd.github+json',
        Authorization: `Bearer ${process.env.GITHUB_TOKEN}`,  // ← فقط سرور
        'User-Agent': 'vercel-tutorial',
      },
    });

    if (response.status === 404) {
      return res.status(404).json({ error: 'کاربر پیدا نشد' });
    }
    if (!response.ok) {
      return res.status(502).json({ error: 'خطا در سرویس GitHub' });
    }

    const data = await response.json();

    // فقط فیلدهای لازم را برگردان (کاهش حجم و حفظ حریم خصوصی)
    return res.status(200).json({
      login: data.login,
      name: data.name,
      public_repos: data.public_repos,
      followers: data.followers,
    });
  } catch (err) {
    console.error('proxy error:', err);
    return res.status(500).json({ error: 'خطای داخلی' });
  }
}
```

### مثال ۳ — تابع با کش هوشمند لبه

```js
// api/rates.js
// نرخ ارز با کش ۵ دقیقه‌ای در لبه
export default async function handler(req, res) {
  try {
    const response = await fetch('https://api.exchangerate.host/latest?base=USD');

    if (!response.ok) {
      return res.status(502).json({ error: 'خطا در سرویس نرخ ارز' });
    }

    const data = await response.json();

    // کش در CDN لبه: ۵ دقیقه تازه، تا ۱۰ دقیقه پس از آن stale سرو می‌شود
    res.setHeader(
      'Cache-Control',
      'public, s-maxage=300, stale-while-revalidate=600'
    );

    return res.status(200).json({ base: 'USD', rates: data.rates });
  } catch (err) {
    console.error(err);
    return res.status(500).json({ error: 'خطای داخلی' });
  }
}
```

### مثال ۴ — استریم پاسخ (مناسب AI و لاگ‌های طولانی)

```js
// api/stream.js
export const config = { runtime: 'edge' };

export default async function handler(request) {
  const encoder = new TextEncoder();

  const stream = new ReadableStream({
    async start(controller) {
      const words = ['سلام', ' ', 'این', ' ', 'یک', ' ', 'پاسخ', ' ', 'استریمی', ' ', 'است.'];
      for (const word of words) {
        controller.enqueue(encoder.encode(word));
        // تأخیر مصنوعی برای نمایش استریم
        await new Promise((r) => setTimeout(r, 200));
      }
      controller.close();
    },
  });

  return new Response(stream, {
    headers: {
      'content-type': 'text/plain; charset=utf-8',
      'cache-control': 'no-store',
    },
  });
}
```

### مثال ۵ — Webhook با تأیید امضا

```js
// api/webhook.js
import crypto from 'crypto';

export const config = {
  api: { bodyParser: false },   // برای خواندن بدنهٔ خام لازم است
};

function readRawBody(req) {
  return new Promise((resolve, reject) => {
    const chunks = [];
    req.on('data', (c) => chunks.push(c));
    req.on('end', () => resolve(Buffer.concat(chunks)));
    req.on('error', reject);
  });
}

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'فقط POST' });
  }

  const rawBody = await readRawBody(req);
  const signature = req.headers['x-signature'];
  const secret = process.env.WEBHOOK_SECRET;

  // تأیید امضا برای اطمینان از اینکه درخواست از سرویس درست آمده
  const expected = crypto
    .createHmac('sha256', secret)
    .update(rawBody)
    .digest('hex');

  if (signature !== expected) {
    return res.status(401).json({ error: 'امضا نامعتبر' });
  }

  const payload = JSON.parse(rawBody.toString('utf-8'));
  console.log('Webhook دریافت شد:', payload.event);

  return res.status(200).json({ received: true });
}
```

### مقایسه و الگوهای مختلف

**سه الگوی کش در توابع**

| الگو | هدر | مناسب برای |
|---|---|---|
| بدون کش | `no-store` | دادهٔ شخصی، لحظه‌ای |
| کش لبه با بازسازی | `s-maxage=300, stale-while-revalidate=600` | نرخ ارز، لیست محصولات |
| کش دائمی | `s-maxage=31536000, immutable` | فایل‌های تغییرناپذیر |

**سه الگوی احراز هویت در توابع**

| الگو | توضیح |
|---|---|
| **API Key در هدر** | ساده، مناسب سرویس‌به‌سرویس |
| **JWT** | مناسب کاربران، بدون حالت |
| **امضای HMAC** | مناسب Webhookها |

### اشکال شایع

**۱. استفاده از حافظه برای ذخیرهٔ داده.**
در نمونه‌ها فقط برای سادگی است؛ در Production از **KV/Postgres** استفاده کنید.

**۲. فراموش کردن `bodyParser: false` در Webhook.**
امضای HMAC روی بدنهٔ خام محاسبه می‌شود؛ اگر بدنه parse شود، امضا نامعتبر می‌شود.

**۳. نبود مدیریت خطا در فراخوانی API بیرونی.**
همیشه `response.ok` را بررسی کنید.

**۴. کش کردن دادهٔ شخصی.**
می‌تواند دادهٔ کاربران را به دیگران نشان دهد. برای دادهٔ شخصی `no-store` بگذارید.

**۵. برگرداندن کل پاسخ API بیرونی.**
فقط فیلدهای لازم را برگردانید تا حجم و ریسک کاهش یابد.

### نکات مهم (Tips & Tricks)

- همیشه **متد** را بررسی و برای نامعتبر `405` برگردانید.
- ورودی‌ها را **اعتبارسنجی** کنید (نوع، طول، الگو).
- کلیدهای حساس را در **Environment Variables** نگه دارید.
- برای پاسخ‌های قابل کش، هدر **`Cache-Control`** مناسب بگذارید.
- خطاها را **لاگ** کنید اما جزئیات را به کاربر نشان ندهید.
- از **استریم** برای پاسخ‌های طولانی و AI استفاده کنید.

### خلاصه فصل

- الگوهای رایج: **CRUD API، پروکسی امن، کش لبه، استریم، Webhook**.
- همیشه متد و ورودی را بررسی کنید.
- کلیدها فقط سمت سرور.
- کش مناسب، هزینه و سرعت را بهینه می‌کند.
- برای دادهٔ پایدار، از سرویس‌های ذخیره‌سازی استفاده کنید (فایل ۰۸).

---

## منابع و مراجع

### مستندات رسمی
- Vercel Functions: <https://vercel.com/docs/functions>
- Fluid Compute: <https://vercel.com/docs/fluid-compute>
- Edge Runtime: <https://vercel.com/docs/functions/runtimes/edge>
- Node.js Runtime: <https://vercel.com/docs/functions/runtimes/node-js>
- Function Limits: <https://vercel.com/docs/functions/limitations>
- Configuring Functions: <https://vercel.com/docs/functions/configuring-functions>

### مفاهیم پیشنهادی برای مطالعه بیشتر
- Web APIs (MDN): <https://developer.mozilla.org/docs/Web/API>
- V8 Isolates: <https://v8.dev/blog/design-of-v8-isolates>
- HTTP Caching (MDN): <https://developer.mozilla.org/docs/Web/HTTP/Caching>

### گام بعدی
- [فایل ۰۶ — Middleware و Streaming](./06-Middleware-and-Streaming.md)
- [فایل ۱۰ — بهینه‌سازی عملکرد](./10-Performance-Optimization.md)
