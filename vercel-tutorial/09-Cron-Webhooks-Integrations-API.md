# ۰۹ — Cron، Webhooks، Integrations و REST API

**مدت یادگیری**: ۳ تا ۴ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: مطالعه فایل‌های ۰۵ (توابع) و ۰۷ (متغیرهای محیطی)

---

## فهرست محتوا

- [فصل ۱: Cron Jobs و زمان‌بندی کارها](#فصل-۱)
- [فصل ۲: Webhooks و ارتباط بین سرویس‌ها](#فصل-۲)
- [فصل ۳: Integrations و مارکت‌پلیس](#فصل-۳)
- [فصل ۴: Vercel REST API](#فصل-۴)
- [فصل ۵: مثال‌های عملی و امنیت](#فصل-۵)

---

<a id="فصل-۱"></a>
## فصل ۱: Cron Jobs و زمان‌بندی کارها

### مقدمه

بعضی کارها باید در زمان مشخصی اجرا شوند: ارسال ایمیل هفتگی، پاک‌سازی داده‌های قدیمی، تولید گزارش شبانه. **Cron Jobs** در Vercel این زمان‌بندی را ممکن می‌کند — بدون نیاز به سرور جداگانه.

### مفاهیم اساسی

**۱. Cron چیست؟**
یک زمان‌بند که در زمان‌های مشخص یک کار را اجرا می‌کند. در Vercel، هر Cron یک درخواست به یک مسیر API می‌فرستد.

> **تشبیه**: مثل زنگ ساعت برای یک کار مشخص. «هر شب ساعت ۳ بیدارم کن و این کار را انجام بده.»

**۲. نحوهٔ تعریف**
در `vercel.json`:

```json
{
  "crons": [
    { "path": "/api/daily-report", "schedule": "0 3 * * *" }
  ]
}
```

**۳. عبارت Cron (Cron Expression)**
پنج بخش دارد:

```
┌──── دقیقه (0-59)
│ ┌─── ساعت (0-23)
│ │ ┌── روز ماه (1-31)
│ │ │ ┌─ ماه (1-12)
│ │ │ │ ┌ روز هفته (0-6، یکشنبه=0)
│ │ │ │ │
* * * * *
```

| عبارت | معنی |
|---|---|
| `* * * * *` | هر دقیقه |
| `0 * * * *` | هر ساعت (دقیقه صفر) |
| `0 3 * * *` | هر روز ساعت ۳ بامداد |
| `0 0 * * 0` | هر یکشنبه نیمه‌شب |
| `0 9 * * 1-5` | هر روز کاری ساعت ۹ صبح |
| `*/15 * * * *` | هر ۱۵ دقیقه |
| `0 0 1 * *` | اول هر ماه |

> **نکته**: منطقهٔ زمانی Cron در Vercel **UTC** است. برای ایران (UTC+3:30)، ساعت ۳ بامداد UTC می‌شود ۶:۳۰ صبح تهران.

**۴. محدودیت‌ها**

| پلن | تعداد Cron | حداقل فاصله |
|---|---|---|
| **Hobby** | محدود (چند مورد) | یک بار در روز |
| **Pro** | بیشتر | هر دقیقه |
| **Enterprise** | زیاد | هر دقیقه |

> مقادیر دقیق با پلن و زمان تغییر می‌کنند؛ مستندات را بررسی کنید.

**۵. نکات کلیدی**
- Cron فقط روی **Production Deployment** اجرا می‌شود.
- Cron یک درخواست **GET** به مسیر می‌فرستد.
- اگر متغیر `CRON_SECRET` تعریف شده باشد، Vercel هدر `Authorization: Bearer <CRON_SECRET>` می‌فرستد.

### مثال‌های عملی

**مثال ۱ — تعریف چند Cron**

```json
{
  "crons": [
    { "path": "/api/cron/daily-report", "schedule": "0 3 * * *" },
    { "path": "/api/cron/cleanup", "schedule": "0 0 * * 0" },
    { "path": "/api/cron/sync", "schedule": "*/15 * * * *" }
  ]
}
```

**مثال ۲ — تابع Cron با تأیید امنیتی**

```js
// api/cron/daily-report.js
export default async function handler(req, res) {
  // ۱. تأیید اینکه درخواست از Vercel Cron آمده
  const authHeader = req.headers.authorization;

  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return res.status(401).json({ error: 'دسترسی غیرمجاز' });
  }

  // ۲. اجرای کار
  try {
    const stats = await generateDailyReport();
    await sendEmail('admin@example.com', 'گزارش روزانه', stats);

    console.log('گزارش روزانه ارسال شد:', new Date().toISOString());
    return res.status(200).json({ ok: true, stats });
  } catch (err) {
    console.error('خطا در گزارش روزانه:', err);
    return res.status(500).json({ error: 'اجرا ناموفق' });
  }
}
```

**مثال ۳ — پاک‌سازی دادهٔ قدیمی**

```js
// api/cron/cleanup.js
import { sql } from '@vercel/postgres';
import { del, list } from '@vercel/blob';

export default async function handler(req, res) {
  if (req.headers.authorization !== `Bearer ${process.env.CRON_SECRET}`) {
    return res.status(401).end();
  }

  // حذف رکوردهای قدیمی‌تر از ۳۰ روز
  const { rowCount } = await sql`
    DELETE FROM logs WHERE created_at < NOW() - INTERVAL '30 days'
  `;

  // حذف فایل‌های موقت قدیمی از Blob
  const { blobs } = await list({ prefix: 'temp/' });
  const old = blobs.filter(
    (b) => Date.now() - new Date(b.uploadedAt).getTime() > 7 * 24 * 3600 * 1000
  );
  await Promise.all(old.map((b) => del(b.url)));

  res.status(200).json({ deletedRows: rowCount, deletedBlobs: old.length });
}
```

**مثال ۴ — همگام‌سازی دوره‌ای**

```js
// api/cron/sync.js
import { kv } from '@vercel/kv';
import { sql } from '@vercel/postgres';

export default async function handler(req, res) {
  if (req.headers.authorization !== `Bearer ${process.env.CRON_SECRET}`) {
    return res.status(401).end();
  }

  // همگام‌سازی شمارنده‌های KV با دیتابیس
  const keys = await kv.keys('post:*:likes');

  let synced = 0;
  for (const key of keys) {
    const postId = key.split(':')[1];
    const likes = await kv.get<number>(key);
    if (likes != null) {
      await sql`UPDATE posts SET likes = ${likes} WHERE id = ${postId}`;
      synced++;
    }
  }

  res.status(200).json({ synced });
}
```

**مثال ۵ — اجرای دستی Cron برای تست**

```bash
# با CRON_SECRET
curl -H "Authorization: Bearer $CRON_SECRET" \
  https://myapp.com/api/cron/daily-report

# بدون CRON_SECRET (فقط در توسعه)
curl http://localhost:3000/api/cron/daily-report
```

### مقایسه و الگوهای مختلف

**Cron Vercel در برابر سرور Cron سنتی**

| معیار | Vercel Cron | Cron سرور سنتی |
|---|---|---|
| راه‌اندازی | فقط `vercel.json` | نصب crontab |
| مقیاس | خودکار | دستی |
| نظارت | Logs در Dashboard | لاگ سیستم |
| هزینه | بخشی از پلتفرم | هزینهٔ سرور |
| منطقهٔ زمانی | UTC | قابل تنظیم |

**سه روش زمان‌بندی**

| روش | مناسب برای |
|---|---|
| **Vercel Cron** | کارهای دوره‌ای ساده |
| **سرویس بیرونی** (مثل Upstash QStash) | زمان‌بندی پیچیده، دقیق |
| **Queue + Worker** | کارهای سنگین و طولانی |

### اشکال شایع

**۱. فراموش کردن تأیید `CRON_SECRET`.**
هر کسی می‌تواند مسیر Cron را صدا بزند و کار حساس را اجرا کند.

**۲. انتظار اجرای Cron روی Preview.**
Cron فقط روی **Production** اجرا می‌شود.

**۳. اشتباه در منطقهٔ زمانی.**
Cron در UTC است. برای تهران ۳:۳۰ ساعت اضافه کنید.

**۴. کار طولانی در Cron.**
محدودیت مدت اجرای تابع را رد می‌کند. کار سنگین را به چند بخش بشکنید.

**۵. نبود لاگ و نظارت.**
اگر Cron شکست بخورد، باید بتوانید بفهمید. همیشه لاگ کنید.

**۶. زمان‌بندی خیلی مکرر در پلن Hobby.**
محدودیت پلن را رد می‌کند و Cron ساخته نمی‌شود.

### نکات مهم (Tips & Tricks)

- همیشه **`CRON_SECRET`** تعریف و در تابع بررسی کنید.
- Cron را **Idempotent** طراحی کنید (اجرای دوباره مشکلی ایجاد نکند).
- برای کارهای حساس، **لاگ ساختاریافته** بنویسید.
- زمان را بر اساس **UTC** محاسبه کنید.
- برای زمان‌بندی دقیق‌تر از **سرویس بیرونی** استفاده کنید.

### خلاصه فصل

- **Cron Jobs** در `vercel.json` تعریف می‌شوند.
- عبارت Cron پنج بخش دارد و در **UTC** است.
- Cron فقط روی **Production** اجرا می‌شود.
- همیشه با **`CRON_SECRET`** محافظت کنید.
- کارهای سنگین را به بخش‌های کوچک بشکنید.

---

<a id="فصل-۲"></a>
## فصل ۲: Webhooks و ارتباط بین سرویس‌ها

### مقدمه

**Webhook** یک «تماس معکوس» است: به‌جای اینکه شما مدام بپرسید «خبری هست؟»، سرویس بیرونی هنگام رخ دادن رویداد به شما خبر می‌دهد. این فصل، هر دو طرف ماجرا را پوشش می‌دهد: دریافت و ارسال Webhook.

### مفاهیم اساسی

**۱. Webhook چیست؟**
یک درخواست HTTP که هنگام رخ دادن یک رویداد به یک URL مشخص فرستاده می‌شود.

> **تشبیه**: Polling مثل این است که هر دقیقه زنگ بزنید و بپرسید «نامه‌ام رسیده؟». Webhook مثل این است که پستچی هنگام رسیدن نامه زنگ در خانه را بزند.

**۲. Polling در برابر Webhook**

| معیار | Polling | Webhook |
|---|---|---|
| مصرف منابع | زیاد (درخواست مداوم) | کم (فقط هنگام رویداد) |
| تأخیر | بستگی به فاصله | ✅ لحظه‌ای |
| پیچیدگی | ساده | متوسط |
| مناسب برای | تغییرات نامنظم | رویدادهای مشخص |

**۳. انواع Webhook در Vercel**

| نوع | توضیح |
|---|---|
| **Incoming** | سرویس بیرونی به اپ شما می‌فرستد |
| **Outgoing (Deployment Webhooks)** | Vercel هنگام استقرار به شما اطلاع می‌دهد |
| **Deploy Hooks** | URL برای فعال‌سازی Deploy از بیرون |

**۴. امنیت Webhook**
چون URL عمومی است، هر کسی می‌تواند درخواست جعلی بفرستد. راه‌های تأیید:
- **امضای HMAC** (رایج‌ترین).
- **توکن مخفی در هدر**.
- **Basic Auth**.
- **IP Whitelist**.

**۵. Deploy Hook**
یک URL یکتا که با فراخوانی آن، Vercel یک استقرار جدید می‌سازد. برای CMS و اتوماسیون مفید است.

### مثال‌های عملی

**مثال ۱ — دریافت Webhook با تأیید امضا**

```js
// api/webhook/stripe.js
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
  if (req.method !== 'POST') return res.status(405).end();

  const rawBody = await readRawBody(req);
  const signature = req.headers['stripe-signature'];
  const secret = process.env.STRIPE_WEBHOOK_SECRET;

  // تأیید امضا
  const expected = crypto
    .createHmac('sha256', secret)
    .update(rawBody)
    .digest('hex');

  if (signature !== expected) {
    console.error('امضای Webhook نامعتبر');
    return res.status(401).json({ error: 'امضا نامعتبر' });
  }

  const event = JSON.parse(rawBody.toString('utf-8'));

  // پردازش رویداد
  switch (event.type) {
    case 'payment.succeeded':
      await handlePayment(event.data);
      break;
    case 'subscription.cancelled':
      await handleCancellation(event.data);
      break;
    default:
      console.log('رویداد نادیده گرفته شد:', event.type);
  }

  // همیشه سریع پاسخ بده (۲۰۰) تا سرویس دوباره تلاش نکند
  res.status(200).json({ received: true });
}
```

**مثال ۲ — ارسال Webhook به سرویس دیگر**

```js
// lib/notify.js
export async function notifySlack(message) {
  const response = await fetch(process.env.SLACK_WEBHOOK_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ text: message }),
  });

  if (!response.ok) {
    console.error('ارسال به Slack ناموفق:', response.status);
  }
}
```

```js
// api/orders/create.js
import { notifySlack } from '../../lib/notify';

export default async function handler(req, res) {
  const order = await createOrder(req.body);

  // اطلاع‌رسانی به تیم (بدون مسدود کردن پاسخ)
  notifySlack(`سفارش جدید: ${order.id} به مبلغ ${order.total}`).catch(console.error);

  res.status(201).json(order);
}
```

**مثال ۳ — Deploy Hook برای CMS**

```bash
# ساخت Deploy Hook از Dashboard:
# Settings → Git → Deploy Hooks → Create Hook

# فراخوانی آن برای فعال‌سازی Deploy
curl -X POST "https://api.vercel.com/v1/integrations/deploy/prj_xxx/yyy"

# در CMS (مثلاً Sanity/Contentful):
# یک Webhook بسازید که هنگام انتشار محتوا این URL را صدا بزند
```

**مثال ۴ — Webhook با صف برای پردازش سنگین**

```js
// api/webhook/github.js
import { kv } from '@vercel/kv';

export default async function handler(req, res) {
  if (req.method !== 'POST') return res.status(405).end();

  // تأیید توکن ساده
  if (req.headers['x-webhook-token'] !== process.env.WEBHOOK_TOKEN) {
    return res.status(401).end();
  }

  const event = req.body;

  // ۱. سریع پاسخ بده (GitHub انتظار پاسخ سریع دارد)
  res.status(202).json({ queued: true });

  // ۲. کار سنگین را در صف بگذار
  await kv.lpush('webhook:queue', JSON.stringify({
    type: event.type ?? event.action,
    payload: event,
    receivedAt: Date.now(),
  }));
}
```

```js
// api/cron/process-queue.js — پردازش صف
import { kv } from '@vercel/kv';

export default async function handler(req, res) {
  if (req.headers.authorization !== `Bearer ${process.env.CRON_SECRET}`) {
    return res.status(401).end();
  }

  let processed = 0;
  while (processed < 10) {
    const item = await kv.rpop('webhook:queue');
    if (!item) break;

    const job = typeof item === 'string' ? JSON.parse(item) : item;
    await processWebhookJob(job);
    processed++;
  }

  res.status(200).json({ processed });
}
```

### مقایسه و الگوهای مختلف

**سه روش تأیید Webhook**

| روش | امنیت | پیچیدگی | مناسب برای |
|---|---|---|---|
| **HMAC Signature** | ✅ بالا | متوسط | Stripe، GitHub |
| **توکن در هدر** | متوسط | کم | سرویس‌های ساده |
| **Basic Auth** | متوسط | کم | ابزارهای داخلی |
| **IP Whitelist** | متوسط | کم | سرویس‌های IP-ثابت |

**پاسخ سریع در برابر پردازش همگام**

| رویکرد | مزیت | عیب |
|---|---|---|
| **پاسخ سریع + صف** | ✅ سرویس فرستنده راضی | نیاز به صف |
| **پردازش همگام** | ساده | ممکن است Timeout شود |

### اشکال شایع

**۱. نبود تأیید امضا.**
هر کسی می‌تواند رویداد جعلی بفرستد. همیشه امضا/توکن را بررسی کنید.

**۲. پردازش طولانی قبل از پاسخ.**
سرویس فرستنده Timeout می‌شود و دوباره تلاش می‌کند → پردازش تکراری.

**۳. فراموش کردن `bodyParser: false`.**
امضای HMAC روی بدنهٔ خام است؛ اگر parse شود، نامعتبر می‌شود.

**۴. نبود Idempotency.**
اگر Webhook دو بار بیاید، عملیات دو بار انجام می‌شود. از شناسهٔ رویداد برای جلوگیری استفاده کنید.

**۵. اعتماد به ترتیب رسیدن Webhookها.**
Webhookها ممکن است بی‌ترتیب برسند. برای رویدادهای وابسته، زمان رویداد را بررسی کنید.

### نکات مهم (Tips & Tricks)

- همیشه **امضا** را تأیید کنید.
- **سریع پاسخ بدهید** (۲۰۰/۲۰۲) و کار سنگین را صف کنید.
- برای جلوگیری از تکرار، **Idempotency Key** ذخیره کنید.
- رویدادهای دریافتی را **لاگ** کنید (برای دیباگ).
- از **Deploy Hook** برای CMS و اتوماسیون استفاده کنید.

### خلاصه فصل

- **Webhook** به‌جای Polling، رویدادها را لحظه‌ای می‌رساند.
- امنیت با **امضای HMAC** یا **توکن** تأمین می‌شود.
- همیشه **سریع پاسخ دهید** و کار سنگین را صف کنید.
- `bodyParser: false` برای امضای HMAC لازم است.
- **Deploy Hook** برای فعال‌سازی استقرار از بیرون.

---

<a id="فصل-۳"></a>
## فصل ۳: Integrations و مارکت‌پلیس

### مقدمه

Vercel فقط یک پلتفرم مستقل نیست؛ با **مارکت‌پلیس Integrations** به ده‌ها سرویس بیرونی وصل می‌شود: دیتابیس، CMS، آنالیتیکس، مانیتورینگ و بیشتر. در این فصل، این اکوسیستم را می‌شناسیم.

### مفاهیم اساسی

**۱. Integration چیست؟**
یک یکپارچگی آماده که سرویس بیرونی را به پروژهٔ شما وصل می‌کند. مزیت: **بدون پیکربندی دستی**، متغیرهای محیطی خودکار ست می‌شوند.

**۲. دسته‌بندی Integrations**

| دسته | نمونه‌ها | کاربرد |
|---|---|---|
| **دیتابیس** | Neon، Upstash، Supabase | ذخیره‌سازی داده |
| **CMS** | Contentful، Sanity، Storyblok | مدیریت محتوا |
| **آنالیتیکس** | Google Analytics، PostHog | تحلیل رفتار |
| **مانیتورینگ** | Sentry، Datadog، Checkly | خطا و عملکرد |
| **احراز هویت** | Auth0، Clerk، Descope | مدیریت کاربران |
| **ایمیل** | Resend، SendGrid | ارسال ایمیل |
| **پرداخت** | Stripe، Lemon Squeezy | پرداخت |
| **جست‌وجو** | Algolia، Meilisearch | جست‌وجوی سریع |

**۳. انواع Integration**

| نوع | توضیح |
|---|---|
| **Native Integration** | ساختهٔ خود Vercel یا شریک رسمی |
| **Marketplace Integration** | از فروشندگان شخص ثالث |
| **Custom Integration** | ساختهٔ خودتان با API Vercel |

**۴. مزیت اصلی**
هنگام نصب Integration، Vercel:
- متغیرهای محیطی لازم را **خودکار** ست می‌کند.
- منابع لازم را می‌سازد.
- اتصال را مدیریت می‌کند.

### مثال‌های عملی

**مثال ۱ — نصب یک Integration**

```
۱. Dashboard → Integrations → Marketplace
۲. سرویس موردنظر را پیدا کنید (مثلاً Neon)
۳. روی «Install» کلیک کنید
۴. حساب سرویس بیرونی را وصل کنید (OAuth)
۵. پروژه‌های موردنظر را انتخاب کنید
۶. Vercel خودکار متغیرهای محیطی را ست می‌کند ✅
۷. در کد، فقط از متغیرها استفاده کنید
```

**مثال ۲ — استفاده از سرویس نصب‌شده در کد**

```js
// api/analytics.js
// فرض: Integration مربوطه متغیرهای محیطی را ست کرده است
export default async function handler(req, res) {
  // نام متغیرها را از مستندات Integration بگیرید
  const apiKey = process.env.ANALYTICS_API_KEY;
  const projectId = process.env.ANALYTICS_PROJECT_ID;

  if (!apiKey || !projectId) {
    return res.status(500).json({ error: 'Integration پیکربندی نشده' });
  }

  const data = await fetchAnalytics({ apiKey, projectId });
  res.json(data);
}
```

**مثال ۳ — ساخت Integration سفارشی (نمای کلی)**

یک Integration سفارشی شامل:

| بخش | توضیح |
|---|---|
| **Manifest** | تعریف Integration (نام، آیکون، دسترسی‌ها) |
| **OAuth** | احراز هویت با Vercel |
| **Webhooks** | دریافت رویدادهای Vercel |
| **Configuration UI** | رابط پیکربندی |

```js
// نمونه: دریافت Webhook از Vercel در Integration شما
// api/vercel-integration/webhook.js
export default async function handler(req, res) {
  const { type, payload } = req.body;

  switch (type) {
    case 'deployment.created':
      console.log('استقرار جدید:', payload.deployment.url);
      break;
    case 'deployment.succeeded':
      await onDeploySuccess(payload);
      break;
    case 'deployment.error':
      await notifyTeam(payload);
      break;
  }

  res.status(200).json({ ok: true });
}
```

### مقایسه و الگوهای مختلف

**Integration در برابر پیکربندی دستی**

| معیار | Integration | دستی |
|---|---|---|
| راه‌اندازی | ✅ چند کلیک | زمان‌بر |
| ست کردن متغیرها | ✅ خودکار | دستی |
| به‌روزرسانی | ✅ مدیریت‌شده | دستی |
| کنترل | کمتر | ✅ کامل |
| مناسب برای | سرویس‌های رایج | سرویس‌های خاص |

**سه سطح یکپارچگی**

| سطح | توضیح | مناسب برای |
|---|---|---|
| **Marketplace** | نصب آماده | اکثر نیازها |
| **Deploy Hook** | اتصال سبک | CMS، اتوماسیون |
| **Custom Integration** | ساخت کامل | محصول SaaS |

### اشکال شایع

**۱. نصب Integration بدون بررسی دسترسی‌ها.**
ممکن است دسترسی بیش از حد بدهید. فقط پروژه‌های لازم را انتخاب کنید.

**۲. اشتباه در نام متغیرهای محیطی.**
هر Integration نام‌های خودش را دارد. از مستندات آن استفاده کنید.

**۳. وابستگی کامل به یک سرویس.**
اگر Integration حذف شود، اپ از کار می‌افتد. لایهٔ انتزاعی داشته باشید.

**۴. نادیده گرفتن هزینهٔ سرویس بیرونی.**
هر Integration ممکن است هزینهٔ جداگانه داشته باشد.

**۵. حذف Integration بدون پاک‌سازی متغیرها.**
متغیرهای یتیم باقی می‌مانند. بعد از حذف، پاک‌سازی کنید.

### نکات مهم (Tips & Tricks)

- از **Marketplace** برای سرویس‌های رایج استفاده کنید (سریع‌تر).
- دسترسی را به **پروژه‌های لازم** محدود کنید.
- نام متغیرها را از **مستندات** بگیرید.
- برای سرویس‌های خاص، **Integration سفارشی** بسازید.
- همیشه یک **لایهٔ انتزاعی** (Wrapper) برای سرویس‌های بیرونی داشته باشید.

### خلاصه فصل

- **Integrations** اتصال به سرویس‌های بیرونی را ساده می‌کنند.
- دسته‌ها: دیتابیس، CMS، آنالیتیکس، مانیتورینگ، احراز هویت و ...
- متغیرهای محیطی **خودکار** ست می‌شوند.
- برای نیازهای خاص، **Integration سفارشی** بسازید.
- همیشه دسترسی را محدود و لایهٔ انتزاعی داشته باشید.

---

<a id="فصل-۴"></a>
## فصل ۴: Vercel REST API

### مقدمه

گاهی می‌خواهید Vercel را از بیرون کنترل کنید: ساخت پروژه، فعال‌سازی Deploy، مدیریت متغیرها — همه از کد. **Vercel REST API** این امکان را می‌دهد.

### مفاهیم اساسی

**۱. آدرس پایه**
```
https://api.vercel.com
```

**۲. احراز هویت**
با **Bearer Token**:

```bash
curl -H "Authorization: Bearer $VERCEL_TOKEN" https://api.vercel.com/v2/user
```

**۳. ساخت توکن**
```
Account Settings → Tokens → Create Token
- نام توکن
- Scope (حساب/تیم)
- Expiration
```

**۴. Team ID**
برای عملیات روی تیم، پارامتر `teamId` لازم است:
```
GET /v9/projects?teamId=team_xxx
```

**۵. نسخه‌بندی**
API نسخه‌بندی‌شده است: `/v6/`, `/v9/`, `/v10/`, `/v13/`. همیشه آخرین نسخهٔ پایدار را استفاده کنید.

**۶. Rate Limiting**
API محدودیت تعداد درخواست دارد. برای عملیات انبوه، بین درخواست‌ها فاصله بگذارید.

### مثال‌های عملی

**مثال ۱ — لیست پروژه‌ها و استقرارها**

```bash
# لیست پروژه‌ها
curl -s -H "Authorization: Bearer $VERCEL_TOKEN" \
  "https://api.vercel.com/v9/projects?teamId=$TEAM_ID" \
  | jq '.projects[] | {name, id}'

# لیست آخرین استقرارها
curl -s -H "Authorization: Bearer $VERCEL_TOKEN" \
  "https://api.vercel.com/v6/deployments?limit=10&teamId=$TEAM_ID" \
  | jq '.deployments[] | {url, state, created}'
```

**مثال ۲ — ساخت پروژه و استقرار**

```bash
# ساخت پروژه
curl -X POST \
  -H "Authorization: Bearer $VERCEL_TOKEN" \
  -H "Content-Type: application/json" \
  "https://api.vercel.com/v11/projects?teamId=$TEAM_ID" \
  -d '{
    "name": "my-new-app",
    "framework": "nextjs",
    "gitRepository": {
      "type": "github",
      "repo": "username/my-new-app"
    }
  }'

# فعال‌سازی استقرار (Redeploy)
curl -X POST \
  -H "Authorization: Bearer $VERCEL_TOKEN" \
  -H "Content-Type: application/json" \
  "https://api.vercel.com/v13/deployments?teamId=$TEAM_ID" \
  -d '{
    "name": "my-new-app",
    "gitSource": {
      "type": "github",
      "repoId": 123456,
      "ref": "main"
    }
  }'
```

**مثال ۳ — مدیریت متغیرهای محیطی با API**

```bash
# لیست متغیرها
curl -s -H "Authorization: Bearer $VERCEL_TOKEN" \
  "https://api.vercel.com/v9/projects/my-app/env?teamId=$TEAM_ID"

# ساخت متغیر
curl -X POST \
  -H "Authorization: Bearer $VERCEL_TOKEN" \
  -H "Content-Type: application/json" \
  "https://api.vercel.com/v10/projects/my-app/env?teamId=$TEAM_ID" \
  -d '{
    "key": "API_KEY",
    "value": "secret_value",
    "type": "encrypted",
    "target": ["production", "preview"]
  }'

# حذف متغیر
curl -X DELETE \
  -H "Authorization: Bearer $VERCEL_TOKEN" \
  "https://api.vercel.com/v9/projects/my-app/env/ENV_ID?teamId=$TEAM_ID"
```

**مثال ۴ — استفاده در Node.js (کلاینت رسمی)**

```bash
npm i @vercel/sdk
```

```ts
// scripts/deploy-status.ts
import { Vercel } from '@vercel/sdk';

const vercel = new Vercel({
  bearerToken: process.env.VERCEL_TOKEN!,
});

async function main() {
  // لیست پروژه‌ها
  const { projects } = await vercel.projects.getProjects({
    teamId: process.env.TEAM_ID,
  });

  console.log('پروژه‌ها:', projects.map((p) => p.name));

  // آخرین استقرارهای یک پروژه
  const { deployments } = await vercel.deployments.getDeployments({
    projectId: projects[0].id,
    limit: 5,
  });

  for (const d of deployments) {
    console.log(`${d.url} → ${d.readyState}`);
  }
}

main().catch(console.error);
```

**مثال ۵ — اسکریپت اتوماسیون (Deploy همهٔ پروژه‌ها)**

```bash
#!/usr/bin/env bash
# deploy-all.sh — استقرار همه پروژه‌ها

set -euo pipefail

PROJECTS=$(curl -s -H "Authorization: Bearer $VERCEL_TOKEN" \
  "https://api.vercel.com/v9/projects?teamId=$TEAM_ID" \
  | jq -r '.projects[].id')

for id in $PROJECTS; do
  echo "در حال استقرار پروژه: $id"
  curl -s -X POST \
    -H "Authorization: Bearer $VERCEL_TOKEN" \
    -H "Content-Type: application/json" \
    "https://api.vercel.com/v13/deployments?teamId=$TEAM_ID&forceNew=1" \
    -d "{\"name\":\"$id\"}" > /dev/null
done

echo "همه پروژه‌ها فعال شدند ✅"
```

### مقایسه و الگوهای مختلف

**سه روش تعامل با Vercel**

| روش | مناسب برای | سرعت |
|---|---|---|
| **Dashboard** | کارهای بصری، بررسی | متوسط |
| **CLI** | توسعه روزمره | ✅ سریع |
| **REST API** | اتوماسیون، CI/CD | سریع |

**توکن شخصی در برابر توکن تیم**

| معیار | Personal Token | Team Token |
|---|---|---|
| دامنهٔ دسترسی | حساب شخصی | تیم |
| مناسب CI/CD | ⚠️ | ✅ |
| مدیریت | خودتان | تیم |

### اشکال شایع

**۱. لو دادن `VERCEL_TOKEN`.**
دسترسی کامل می‌دهد. هرگز در کد یا مخزن نگذارید؛ از Secret استفاده کنید.

**۲. استفاده از توکن بدون تاریخ انقضا.**
اگر لو برود، تا ابد معتبر است. همیشه Expiration بگذارید.

**۳. نادیده گرفتن Rate Limit.**
درخواست‌های سریع پشت‌سرهم مسدود می‌شوند. فاصله بگذارید یا Retry منطقی داشته باشید.

**۴. اشتباه در نسخهٔ API.**
استفاده از نسخهٔ قدیمی می‌تواند خطا بدهد. از آخرین نسخهٔ پایدار استفاده کنید.

**۵. فراموش کردن `teamId`.**
عملیات روی حساب شخصی به‌جای تیم انجام می‌شود.

### نکات مهم (Tips & Tricks)

- توکن را در **Secret های CI** ذخیره کنید.
- برای پروژه‌های بزرگ، از **کلاینت رسمی (`@vercel/sdk`)** استفاده کنید.
- همیشه **Expiration** برای توکن بگذارید.
- درخواست‌ها را با **`jq`** پردازش کنید (خواناتر).
- برای عملیات انبوه، **Rate Limit** را در نظر بگیرید.

### خلاصه فصل

- **REST API** آدرس پایهٔ `https://api.vercel.com` دارد.
- احراز هویت با **Bearer Token**.
- توکن را از **Account Settings → Tokens** بسازید.
- کلاینت رسمی `@vercel/sdk` کار را ساده می‌کند.
- توکن را همیشه در **Secret** نگه دارید.

---

<a id="فصل-۵"></a>
## فصل ۵: مثال‌های عملی و امنیت

### مقدمه

در این فصل، همهٔ مفاهیم قبلی را در یک سناریوی واقعی ترکیب می‌کنیم: یک سیستم گزارش‌گیری خودکار با Cron، Webhook و API.

### مثال ۱ — سیستم گزارش خودکار کامل

```json
// vercel.json
{
  "crons": [
    { "path": "/api/cron/collect", "schedule": "0 * * * *" },
    { "path": "/api/cron/report", "schedule": "0 6 * * *" }
  ]
}
```

```js
// api/cron/collect.js — هر ساعت داده جمع می‌کند
import { kv } from '@vercel/kv';
import { sql } from '@vercel/postgres';

export default async function handler(req, res) {
  if (req.headers.authorization !== `Bearer ${process.env.CRON_SECRET}`) {
    return res.status(401).end();
  }

  const { rows } = await sql`
    SELECT COUNT(*)::int AS count, AVG(duration)::numeric(10,2) AS avg
    FROM events WHERE created_at > NOW() - INTERVAL '1 hour'
  `;

  const snapshot = {
    hour: new Date().toISOString(),
    ...rows[0],
  };

  await kv.lpush('metrics:hourly', JSON.stringify(snapshot));
  await kv.ltrim('metrics:hourly', 0, 167);   // ۷ روز × ۲۴ ساعت

  res.status(200).json(snapshot);
}
```

```js
// api/cron/report.js — هر روز گزارش می‌سازد
import { kv } from '@vercel/kv';

export default async function handler(req, res) {
  if (req.headers.authorization !== `Bearer ${process.env.CRON_SECRET}`) {
    return res.status(401).end();
  }

  const raw = await kv.lrange('metrics:hourly', 0, 23);
  const items = raw.map((r) => (typeof r === 'string' ? JSON.parse(r) : r));

  const total = items.reduce((s, i) => s + i.count, 0);
  const avg = items.reduce((s, i) => s + Number(i.avg ?? 0), 0) / (items.length || 1);

  // ارسال گزارش (مثلاً با Resend)
  await fetch('https://api.resend.com/emails', {
    method: 'POST',
    headers: {
      Authorization: `Bearer ${process.env.RESEND_API_KEY}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      from: 'reports@myapp.com',
      to: 'team@myapp.com',
      subject: 'گزارش روزانه',
      html: `<h2>گزارش</h2><p>کل رویدادها: ${total}</p><p>میانگین: ${avg.toFixed(2)}</p>`,
    }),
  });

  res.status(200).json({ total, avg });
}
```

### مثال ۲ — Webhook پرداخت با Idempotency

```js
// api/webhook/payment.js
import crypto from 'crypto';
import { sql } from '@vercel/postgres';
import { kv } from '@vercel/kv';

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
  const rawBody = await readRawBody(req);
  const signature = req.headers['x-signature'];

  const expected = crypto
    .createHmac('sha256', process.env.WEBHOOK_SECRET)
    .update(rawBody)
    .digest('hex');

  if (signature !== expected) {
    return res.status(401).json({ error: 'امضا نامعتبر' });
  }

  const event = JSON.parse(rawBody.toString('utf-8'));
  const eventId = event.id;

  // Idempotency: اگر قبلاً پردازش شده، رد کن
  const alreadyProcessed = await kv.get(`webhook:processed:${eventId}`);
  if (alreadyProcessed) {
    return res.status(200).json({ duplicate: true });
  }

  // علامت‌گذاری به‌عنوان پردازش‌شده (با انقضای ۷ روز)
  await kv.set(`webhook:processed:${eventId}`, true, { ex: 60 * 60 * 24 * 7 });

  // پردازش
  if (event.type === 'payment.succeeded') {
    await sql`
      UPDATE orders SET status = 'paid', paid_at = NOW()
      WHERE id = ${event.data.orderId}
    `;
  }

  res.status(200).json({ ok: true });
}
```

### مثال ۳ — اسکریپت CI/CD با REST API

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Vercel
        env:
          VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
          VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
          VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
        run: |
          npx vercel@latest pull --yes --environment=production --token=$VERCEL_TOKEN
          npx vercel@latest build --prod --token=$VERCEL_TOKEN
          npx vercel@latest deploy --prebuilt --prod --token=$VERCEL_TOKEN
```

### مثال ۴ — چک‌لیست امنیتی فراخوانی API

```js
// lib/secureFetch.js — فراخوانی امن API های بیرونی
export async function secureFetch(url, options = {}) {
  const controller = new AbortController();
  const timeout = setTimeout(() => controller.abort(), 10_000);   // ۱۰ ثانیه

  try {
    const response = await fetch(url, {
      ...options,
      signal: controller.signal,
      headers: {
        'User-Agent': 'myapp/1.0',
        ...options.headers,
      },
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    return await response.json();
  } catch (err) {
    if (err.name === 'AbortError') {
      throw new Error('درخواست به دلیل Timeout لغو شد');
    }
    throw err;
  } finally {
    clearTimeout(timeout);
  }
}
```

### نکات امنیتی کلیدی

| نکته | دلیل |
|---|---|
| **تأیید امضای Webhook** | جلوگیری از درخواست جعلی |
| **`CRON_SECRET`** | جلوگیری از اجرای دستی Cron |
| **Idempotency Key** | جلوگیری از پردازش تکراری |
| **Timeout در fetch** | جلوگیری از معلق ماندن تابع |
| **ذخیرهٔ توکن در Secret** | جلوگیری از نشت |
| **محدود کردن دسترسی Integration** | اصل کمترین دسترسی |
| **لاگ بدون دادهٔ حساس** | حفظ حریم خصوصی |

### اشکال شایع

**۱. Cron بدون محافظت.**
هر کسی می‌تواند کار حساس را اجرا کند. همیشه `CRON_SECRET`.

**۲. Webhook بدون Idempotency.**
پرداخت دو بار ثبت می‌شود. همیشه شناسهٔ رویداد را ذخیره کنید.

**۳. Timeout نبودن در فراخوانی API.**
تابع معلق می‌ماند و هزینه می‌سوزاند. همیشه `AbortController`.

**۴. توکن در کد.**
نشت قطعی. همیشه Secret.

**۵. لاگ کردن کل payload حساس.**
اطلاعات کارت/رمز در لاگ می‌ماند. فقط فیلدهای لازم را لاگ کنید.

### خلاصه فصل

- Cron + Webhook + API = اتوماسیون کامل.
- **امنیت** در هر سه لایه ضروری است.
- `CRON_SECRET`، امضای HMAC و Idempotency، سه ستون امنیت‌اند.
- همیشه **Timeout** برای فراخوانی‌های بیرونی بگذارید.
- توکن‌ها فقط در **Secret**.

---

## منابع و مراجع

### مستندات رسمی
- Vercel Cron Jobs: <https://vercel.com/docs/cron-jobs>
- Webhooks (Vercel): <https://vercel.com/docs/webhooks>
- Deploy Hooks: <https://vercel.com/docs/deploy-hooks>
- Vercel REST API: <https://vercel.com/docs/rest-api>
- Vercel SDK (TypeScript): <https://vercel.com/docs/rest-api/sdk>
- Integrations: <https://vercel.com/docs/integrations>
- Vercel Marketplace: <https://vercel.com/marketplace>

### مفاهیم پیشنهادی برای مطالعه بیشتر
- Cron Expression Generator: <https://crontab.guru>
- Webhooks Explained: <https://webhook.site>
- HMAC (Wikipedia): <https://en.wikipedia.org/wiki/HMAC>

### گام بعدی
- [فایل ۱۰ — بهینه‌سازی عملکرد](./10-Performance-Optimization.md)
- [فایل ۱۲ — امنیت](./12-Security.md)
