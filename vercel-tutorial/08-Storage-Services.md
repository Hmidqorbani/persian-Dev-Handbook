# ۰۸ — سرویس‌های داده و ذخیره‌سازی

**مدت یادگیری**: ۴ تا ۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: مطالعه فایل‌های ۰۵ (توابع Serverless) و ۰۷ (متغیرهای محیطی)

---

## فهرست محتوا

- [فصل ۱: معرفی اکوسیستم ذخیره‌سازی Vercel](#فصل-۱)
- [فصل ۲: Vercel KV — کش کلید-مقدار](#فصل-۲)
- [فصل ۳: Vercel Postgres — دیتابیس رابطه‌ای](#فصل-۳)
- [فصل ۴: Vercel Blob — ذخیره‌سازی فایل](#فصل-۴)
- [فصل ۵: Edge Config](#فصل-۵)
- [فصل ۶: انتخاب درست و مثال پروژه واقعی](#فصل-۶)

---

<a id="فصل-۱"></a>
## فصل ۱: معرفی اکوسیستم ذخیره‌سازی Vercel

### مقدمه

توابع Serverless **Stateless** هستند؛ یعنی نمی‌توانند داده را در حافظه نگه دارند. پس داده‌ها کجا می‌روند؟ پاسخ: در **سرویس‌های ذخیره‌سازی**. در این فصل، نقشهٔ کامل این سرویس‌ها را می‌بینیم و می‌فهمیم کدام برای چه کاری مناسب است.

### مفاهیم اساسی

**۱. چرا به ذخیره‌سازی جدا نیاز داریم؟**

```
تابع Serverless (موقت)  →  ✗ نمی‌تواند داده نگه دارد
                              ↓
                        سرویس ذخیره‌سازی (پایدار)
```

چون نمونه‌های تابع ممکن است هر لحظه پاک یا جابه‌جا شوند، داده باید در یک سرویس **بیرونی و پایدار** باشد.

**۲. چهار سرویس اصلی**

| سرویس | نوع | کاربرد اصلی |
|---|---|---|
| **KV** | کلید-مقدار (Redis) | نشست، کش، شمارنده، Rate Limit |
| **Postgres** | رابطه‌ای (SQL) | داده ساختاریافته، کاربران، سفارش‌ها |
| **Blob** | شیء (Object Storage) | تصویر، ویدیو، فایل آپلودی |
| **Edge Config** | پیکربندی سریع | Feature Flag، تنظیمات لحظه‌ای |

**۳. نکتهٔ مهم دربارهٔ اکوسیستم امروز**
Vercel در حال انتقال تدریجی سرویس‌های داده به **Marketplace** است؛ یعنی برخی سرویس‌ها به‌صورت یکپارچه با ارائه‌دهندگان تخصصی ارائه می‌شوند:

| سرویس Vercel | ارائه‌دهندهٔ پشتیبان |
|---|---|
| Vercel KV | Redis (Upstash) |
| Vercel Postgres | Neon |
| Vercel Blob | سرویس بومی Vercel |
| Edge Config | سرویس بومی Vercel |

> **نتیجهٔ عملی**: API ها و مفاهیم یکسان‌اند؛ فقط نحوهٔ ساخت ممکن است از طریق Marketplace یا Dashboard باشد. در این کتابچه روی **مفاهیم و API** تمرکز می‌کنیم که پایدارترند.

**۴. تفاوت داده‌های «داغ» و «سرد»**

| نوع داده | دسترسی | سرویس مناسب |
|---|---|---|
| **داغ (Hot)** | بسیار پرتکرار، لحظه‌ای | KV، Edge Config |
| **گرم (Warm)** | پرتکرار، ساختاریافته | Postgres |
| **سرد (Cold)** | کم‌تکرار، حجم زیاد | Blob |

**۵. تفاوت SQL و NoSQL در یک نگاه**

| معیار | Postgres (SQL) | KV (NoSQL) |
|---|---|---|
| ساختار | جدول، سطر، ستون | کلید → مقدار |
| کوئری پیچیده | ✅ | ❌ |
| سرعت خواندن ساده | بالا | ✅ بسیار بالا |
| مناسب برای | داده رابطه‌ای | نشست، کش |

### مثال‌های عملی

**مثال ۱ — کدام سرویس برای کدام سناریو؟**

```
【 ثبت‌نام و لاگین کاربران 】
→ Postgres (داده ساختاریافته، کوئری)
+ KV (ذخیرهٔ نشست)

【 نمایش پست‌های یک وبلاگ 】
→ Postgres (محتوا)
+ Blob (تصاویر)
+ KV (کش پست‌های پرطرفدار)

【 آپلود تصویر پروفایل 】
→ Blob (فایل)
+ Postgres (لینک فایل)

【 Feature Flag برای تست A/B 】
→ Edge Config (سریع، در لبه)

【 شمارش بازدید 】
→ KV (شمارنده، بسیار سریع)
```

**مثال ۲ — یک اتصال ترکیبی**

```js
// api/profile.js — ترکیب Postgres + Blob
import { sql } from '@vercel/postgres';

export default async function handler(req, res) {
  // داده ساختاریافته از Postgres
  const { rows } = await sql`
    SELECT id, name, avatar_url FROM users WHERE id = ${req.query.id}
  `;

  if (rows.length === 0) {
    return res.status(404).json({ error: 'کاربر پیدا نشد' });
  }

  // avatar_url به فایل ذخیره‌شده در Blob اشاره می‌کند
  return res.status(200).json(rows[0]);
}
```

### مقایسه و الگوهای مختلف

**جدول جامع انتخاب**

| نیاز | KV | Postgres | Blob | Edge Config |
|---|---|---|---|---|
| کوئری پیچیده (JOIN) | ❌ | ✅ | ❌ | ❌ |
| سرعت خواندن بالا | ✅ | متوسط | متوسط | ✅✅ |
| ذخیره فایل | ❌ | ⚠️ (نامناسب) | ✅ | ❌ |
| داده بزرگ (GB) | ⚠️ | ✅ | ✅ | ❌ |
| دسترسی از Edge | ✅ | ⚠️ محدود | ✅ | ✅✅ |
| هزینه | کم | متوسط | بر اساس حجم | بسیار کم |
| تغییرات مکرر | ✅ | ✅ | ⚠️ | ⚠️ (کم) |

**الگوی معمول یک اپ واقعی**

```
Edge Config  →  Feature Flags و تنظیمات
KV           →  نشست، کش، Rate Limit
Postgres     →  کاربران، سفارش‌ها، محتوا
Blob         →  تصاویر، فایل‌ها، ویدیو
```

### اشکال شایع

**۱. استفاده از Postgres برای کش.**
Postgres برای کش سریع ساخته نشده. برای کش از **KV** استفاده کنید.

**۲. ذخیرهٔ فایل در Postgres.**
دیتابیس را سنگین می‌کند. فایل‌ها را در **Blob** بگذارید و فقط لینک را ذخیره کنید.

**۳. استفاده از KV برای دادهٔ رابطه‌ای.**
بدون JOIN و کوئری پیچیده، نگهداری داده پیچیده در KV فاجعه است.

**۴. فراموش کردن اتصال Pooling.**
با توابع Serverless، اتصال‌های Postgres زیاد می‌شود. از Pooler استفاده کنید.

**۵. استفاده از Edge Config برای دادهٔ پویا.**
Edge Config برای داده‌ای است که **کم** تغییر می‌کند. برای داده پویا مناسب نیست.

### نکات مهم (Tips & Tricks)

- ابتدا **نیازتان را مشخص کنید**، بعد سرویس را انتخاب کنید.
- برای دادهٔ پرتکرار و کم‌تغییر، **کش** بگذارید (KV).
- فایل‌ها همیشه در **Blob**، لینک در دیتابیس.
- برای تنظیمات لحظه‌ای بدون Deploy، از **Edge Config** استفاده کنید.
- همهٔ کلیدهای اتصال را در **Environment Variables** بگذارید.

### خلاصه فصل

- توابع Stateless نیازمند سرویس‌های ذخیره‌سازی بیرونی هستند.
- چهار سرویس: **KV، Postgres، Blob، Edge Config**.
- Vercel در حال انتقال به **Marketplace** است؛ مفاهیم و API پایدارند.
- انتخاب درست بر اساس نوع داده (داغ/گرم/سرد) انجام می‌شود.
- الگوی رایج: ترکیب چند سرویس در یک اپ.

---

<a id="فصل-۲"></a>
## فصل ۲: Vercel KV — کش کلید-مقدار

### مقدمه

**KV** مخفف Key-Value است: ساده‌ترین نوع دیتابیس. یک کلید دارید و یک مقدار. سرعتش فوق‌العاده است چون بر پایهٔ **Redis** ساخته شده. در این فصل، کاربردها و کد کامل آن را یاد می‌گیریم.

### مفاهیم اساسی

**۱. KV چیست؟**
یک ذخیره‌گاه کلید-مقدار در حافظه (in-memory) که برای دسترسی **بسیار سریع** طراحی شده است.

> **تشبیه**: KV مثل یک قفسهٔ کلید در هتل است. به‌جای گشتن در همهٔ اتاق‌ها، فقط شمارهٔ اتاق را می‌گویید و کلید را فوراً می‌گیرید.

**۲. کاربردهای اصلی**

| کاربرد | توضیح |
|---|---|
| **نشست (Session)** | ذخیرهٔ توکن/داده کاربر |
| **کش (Cache)** | نتیجهٔ کوئری سنگین |
| **شمارنده** | بازدید، لایک |
| **Rate Limiting** | محدودسازی درخواست |
| **صف (Queue)** | کارهای در انتظار |
| **Feature Flag** | فعال/غیرفعال کردن قابلیت |

**۳. عملیات اصلی**

| عملیات | توضیح |
|---|---|
| `set(key, value)` | ذخیره |
| `get(key)` | خواندن |
| `del(key)` | حذف |
| `incr(key)` | افزایش عددی |
| `expire(key, sec)` | تعیین انقضا |
| `hset/hget` | ذخیرهٔ هش (شیء) |

**۴. انواع داده در KV**
علاوه بر رشته، می‌توانید JSON، لیست، مجموعه (Set) و هش ذخیره کنید.

### مثال‌های عملی

**مثال ۱ — ذخیره و خواندن ساده**

```js
// api/kv-demo.js
import { kv } from '@vercel/kv';

export default async function handler(req, res) {
  // نوشتن
  await kv.set('greeting', 'سلام دنیا');

  // خواندن
  const value = await kv.get('greeting');

  res.json({ value });
}
```

**مثال ۲ — کش کردن نتیجهٔ کوئری سنگین**

```js
// api/products.js
import { kv } from '@vercel/kv';
import { sql } from '@vercel/postgres';

export default async function handler(req, res) {
  const CACHE_KEY = 'products:all';
  const TTL = 300;   // ۵ دقیقه

  // ۱. تلاش برای خواندن از کش
  const cached = await kv.get(CACHE_KEY);
  if (cached) {
    return res.status(200).json({ source: 'cache', data: cached });
  }

  // ۲. اگر نبود، از دیتابیس بخوان
  const { rows } = await sql`SELECT * FROM products ORDER BY created_at DESC LIMIT 50`;

  // ۳. ذخیره در کش با انقضا
  await kv.set(CACHE_KEY, rows, { ex: TTL });

  return res.status(200).json({ source: 'db', data: rows });
}
```

**مثال ۳ — شمارندهٔ بازدید**

```js
// api/views.js
import { kv } from '@vercel/kv';

export default async function handler(req, res) {
  const { slug } = req.query;
  if (!slug) return res.status(400).json({ error: 'slug لازم است' });

  // افزایش اتمیک شمارنده
  const views = await kv.incr(`views:${slug}`);

  res.status(200).json({ slug, views });
}
```

**مثال ۴ — نشست کاربر (Session)**

```js
// api/login.js
import { kv } from '@vercel/kv';
import crypto from 'crypto';

export default async function handler(req, res) {
  if (req.method !== 'POST') return res.status(405).end();

  const { email, password } = req.body;

  // در عمل: بررسی اعتبار کاربر در Postgres
  const validUser = await checkUser(email, password);
  if (!validUser) {
    return res.status(401).json({ error: 'ایمیل یا رمز نادرست' });
  }

  // ساخت توکن نشست
  const sessionId = crypto.randomUUID();

  // ذخیرهٔ نشست با انقضای ۷ روز
  await kv.set(
    `session:${sessionId}`,
    { userId: validUser.id, email },
    { ex: 60 * 60 * 24 * 7 }
  );

  // تنظیم کوکی
  res.setHeader(
    'Set-Cookie',
    `session=${sessionId}; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=604800`
  );

  return res.status(200).json({ ok: true });
}
```

**مثال ۵ — Rate Limiting دقیق با KV**

```js
// lib/rateLimit.js
import { kv } from '@vercel/kv';

export async function rateLimit(identifier, limit = 10, windowSec = 60) {
  const key = `ratelimit:${identifier}`;

  const count = await kv.incr(key);

  // اولین درخواست در این پنجره → تنظیم انقضا
  if (count === 1) {
    await kv.expire(key, windowSec);
  }

  if (count > limit) {
    const ttl = await kv.ttl(key);
    return { allowed: false, remaining: 0, resetIn: ttl };
  }

  return { allowed: true, remaining: limit - count, resetIn: windowSec };
}
```

```js
// api/limited.js
import { rateLimit } from '../lib/rateLimit';

export default async function handler(req, res) {
  const ip = req.headers['x-forwarded-for']?.split(',')[0] ?? 'unknown';
  const result = await rateLimit(ip, 10, 60);

  if (!result.allowed) {
    return res.status(429).json({
      error: 'تعداد درخواست‌ها زیاد است',
      resetIn: result.resetIn,
    });
  }

  return res.status(200).json({ ok: true, remaining: result.remaining });
}
```

### مقایسه و الگوهای مختلف

**KV در برابر کش CDN**

| معیار | KV | کش CDN |
|---|---|---|
| محل | دیتاسنتر | لبه (Edge) |
| مناسب برای | داده پویا، نشست | پاسخ HTTP |
| کنترل | کد | هدر `Cache-Control` |
| دسترسی از Edge | ✅ | — |
| مناسب نشست | ✅ | ❌ |

**سه استراتژی کش**

| استراتژی | توضیح | مناسب برای |
|---|---|---|
| **Cache-Aside** | اول کش، بعد دیتابیس | ✅ عمومی‌ترین |
| **Write-Through** | نوشتن همزمان در کش و دیتابیس | داده‌ای که سریع خوانده می‌شود |
| **TTL کوتاه** | انقضای کوتاه برای تازگی | داده‌های نیمه‌پویا |

### اشکال شایع

**۱. کش کردن بدون انقضا (TTL).**
داده تا ابد قدیمی می‌ماند. همیشه `ex` تعیین کنید.

**۲. کش کردن دادهٔ شخصی با کلید ثابت.**
کاربر الف دادهٔ کاربر ب را می‌بیند! کلید باید شامل شناسهٔ کاربر باشد: `user:${id}:data`.

**۳. فراموش کردن Invalidating کش.**
بعد از تغییر داده، کش را پاک کنید: `kv.del(key)`.

**۴. استفاده از KV برای دادهٔ رابطه‌ای.**
بدون JOIN، نگهداری روابط پیچیده دردناک است.

**۵. مصرف زیاد KV به‌خاطر کلیدهای زیاد.**
ساختار کلید را منظم و قابل مدیریت نگه دارید.

### نکات مهم (Tips & Tricks)

- **نام‌گذاری کلید** با پیشوند: `session:`, `cache:`, `ratelimit:`.
- همیشه **TTL** تعیین کنید (مگر دادهٔ دائمی).
- برای شمارنده‌ها از `incr` (اتمیک) استفاده کنید.
- برای ذخیرهٔ شیء، JSON را مستقیم پاس دهید (`kv.set` خودش سریالایز می‌کند).
- Rate Limiting جدی را با **Vercel Firewall** انجام دهید؛ KV برای موارد سبک.

### خلاصه فصل

- **KV** یک ذخیره‌گاه کلید-مقدار فوق‌سریع (Redis) است.
- کاربردها: **نشست، کش، شمارنده، Rate Limit، Feature Flag**.
- همیشه **TTL** بگذارید.
- کلید دادهٔ شخصی باید شامل شناسهٔ کاربر باشد.
- برای دادهٔ رابطه‌ای از Postgres استفاده کنید.

---

<a id="فصل-۳"></a>
## فصل ۳: Vercel Postgres — دیتابیس رابطه‌ای

### مقدمه

وقتی دادهٔ شما ساختار دارد (کاربران، سفارش‌ها، محصولات) و به کوئری پیچیده نیاز دارید، **Postgres** انتخاب درست است. در این فصل، اتصال، کوئری و الگوهای حرفه‌ای آن را یاد می‌گیریم.

### مفاهیم اساسی

**۱. Postgres چیست؟**
یک دیتابیس **رابطه‌ای (Relational)** که داده را در جدول‌ها با روابط نگه می‌دارد. از زبان **SQL** استفاده می‌کند.

**۲. ساختار پایه**

```
users                 orders
┌────┬───────┐       ┌────┬─────────┬─────────┐
│ id │ name  │       │ id │ user_id │ total   │
├────┼───────┤       ├────┼─────────┼─────────┤
│ 1  │ علی   │◄──────│ 1  │ 1       │ 250000  │
│ 2  │ سارا  │       │ 2  │ 1       │ 90000   │
└────┴───────┘       └────┴─────────┴─────────┘
                     (user_id → users.id)
```

**۳. Connection Pooling**
توابع Serverless اتصال‌های زیادی باز می‌کنند. **Pooler** اتصال‌ها را مدیریت می‌کند و از اشباع دیتابیس جلوگیری می‌کند.

| نوع اتصال | مناسب برای |
|---|---|
| **Direct** | Migration، ابزارهای مدیریت |
| **Pooled** | توابع Serverless (✅) |

**۴. کتابخانه‌ها**

| کتابخانه | مناسب برای |
|---|---|
| `@vercel/postgres` | API ساده با تگ‌تمپلیت |
| `@neondatabase/serverless` | دسترسی مستقیم به Neon |
| `pg` / `postgres.js` | کنترل کامل (نیاز به Pooling) |
| **Drizzle / Prisma** | ORM با تایپ‌سیفتی |

**۵. کوئری پارامتری (امنیت)**
همیشه از پارامتر استفاده کنید تا از **SQL Injection** جلوگیری شود:

```js
// ❌ خطرناک
sql`SELECT * FROM users WHERE id = ${userId}`   // ✅ در واقع امن است (تگ‌تمپلیت)

// ❌ واقعاً خطرناک (string concat)
`SELECT * FROM users WHERE id = '${userId}'`
```

### مثال‌های عملی

**مثال ۱ — ساخت جدول و درج داده**

```js
// api/setup.js — فقط یک بار اجرا کنید
import { sql } from '@vercel/postgres';

export default async function handler(req, res) {
  await sql`
    CREATE TABLE IF NOT EXISTS users (
      id SERIAL PRIMARY KEY,
      name VARCHAR(100) NOT NULL,
      email VARCHAR(255) UNIQUE NOT NULL,
      created_at TIMESTAMP DEFAULT NOW()
    );
  `;

  await sql`
    INSERT INTO users (name, email)
    VALUES ('علی', 'ali@example.com')
    ON CONFLICT (email) DO NOTHING;
  `;

  res.json({ ok: true });
}
```

**مثال ۲ — خواندن با فیلتر**

```js
// api/users.js
import { sql } from '@vercel/postgres';

export default async function handler(req, res) {
  const { search } = req.query;

  const { rows } = await sql`
    SELECT id, name, email, created_at
    FROM users
    WHERE name ILIKE ${'%' + (search ?? '') + '%'}
    ORDER BY created_at DESC
    LIMIT 20
  `;

  res.status(200).json({ count: rows.length, users: rows });
}
```

**مثال ۳ — عملیات تراکنشی (Transaction)**

```js
// api/transfer.js
import { sql } from '@vercel/postgres';

export default async function handler(req, res) {
  const { fromId, toId, amount } = req.body;

  const client = await sql.connect();
  try {
    await client.sql`BEGIN`;

    // کسر از مبدأ
    await client.sql`
      UPDATE accounts SET balance = balance - ${amount} WHERE id = ${fromId}
    `;

    // افزودن به مقصد
    await client.sql`
      UPDATE accounts SET balance = balance + ${amount} WHERE id = ${toId}
    `;

    await client.sql`COMMIT`;
    res.json({ ok: true });
  } catch (err) {
    await client.sql`ROLLBACK`;
    console.error(err);
    res.status(500).json({ error: 'انتقال ناموفق' });
  } finally {
    client.release();
  }
}
```

**مثال ۴ — استفاده با ORM (Drizzle)**

```ts
// lib/db.ts
import { drizzle } from 'drizzle-orm/vercel-postgres';
import { sql } from '@vercel/postgres';
import { pgTable, serial, varchar, timestamp } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: varchar('name', { length: 100 }).notNull(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  createdAt: timestamp('created_at').defaultNow(),
});

export const db = drizzle(sql);
```

```ts
// api/users-orm.ts
import { db } from '@/lib/db';
import { users } from '@/lib/db';
import { eq } from 'drizzle-orm';

export async function GET() {
  // تایپ‌سیف و خوانا
  const all = await db.select().from(users).limit(20);
  return Response.json(all);
}
```

### مقایسه و الگوهای مختلف

**سه راه کار با Postgres**

| روش | مزیت | عیب |
|---|---|---|
| **`@vercel/postgres`** | ساده، تگ‌تمپلیت امن | کوئری دستی |
| **ORM (Drizzle/Prisma)** | تایپ‌سیف، خوانا | حجم بیشتر |
| **`pg` خام** | کنترل کامل | مدیریت دستی Pooling |

**Direct در برابر Pooled Connection**

| معیار | Direct | Pooled |
|---|---|---|
| مناسب Serverless | ❌ | ✅ |
| مناسب Migration | ✅ | ❌ |
| مدیریت اتصال | دستی | خودکار |
| کارایی در بار بالا | ضعیف | ✅ خوب |

### اشکال شایع

**۱. استفاده از اتصال Direct در Serverless.**
دیتابیس اشباع می‌شود. از **Pooled** استفاده کنید.

**۲. ساخت جدول در زمان اجرا (Runtime).**
هر بار اجرا می‌شود و کند است. Migration را جداگانه اجرا کنید.

**۳. نبود ایندکس (Index) روی ستون‌های پرجست‌وجو.**
کوئری‌ها کند می‌شوند. روی `email`، `user_id` و ... ایندکس بگذارید.

**۴. استفاده از `SELECT *` در جدول بزرگ.**
دادهٔ اضافه می‌خواند. فقط ستون‌های لازم را انتخاب کنید.

**۵. فراموش کردن `LIMIT`.**
کوئری می‌تواند هزاران سطر برگرداند و تابع را کند کند.

### نکات مهم (Tips & Tricks)

- همیشه از **Pooled Connection** در توابع استفاده کنید.
- روی ستون‌های فیلترشده **Index** بگذارید.
- از **ORM** برای پروژه‌های بزرگ استفاده کنید (تایپ‌سیفتی).
- کوئری‌های پرتکرار را در **KV** کش کنید.
- Migration ها را **نسخه‌بندی** و در CI/CD اجرا کنید.

### خلاصه فصل

- **Postgres** دیتابیس رابطه‌ای برای دادهٔ ساختاریافته است.
- در Serverless همیشه از **Pooled Connection** استفاده کنید.
- کتابخانه‌ها: `@vercel/postgres`، Neon، ORM ها.
- از کوئری پارامتری برای امنیت استفاده کنید.
- ایندکس و LIMIT، کلید کارایی هستند.

---

<a id="فصل-۴"></a>
## فصل ۴: Vercel Blob — ذخیره‌سازی فایل

### مقدمه

تصاویر، ویدیوها، PDF و فایل‌های آپلودی جایشان در دیتابیس نیست. **Blob** یک سرویس ذخیره‌سازی شیء (Object Storage) است که برای این فایل‌ها طراحی شده و با CDN یکپارچه است.

### مفاهیم اساسی

**۱. Blob چیست؟**
یک سرویس ذخیره‌سازی فایل روی فضای ابری که فایل‌ها را با یک **URL عمومی یا خصوصی** در دسترس قرار می‌دهد.

**۲. تفاوت با ذخیرهٔ فایل در پروژه**
- فایل‌های `public/` با هر Deploy بازنشانی می‌شوند.
- فایل‌های Blob **پایدار** و مستقل از Deploy هستند.

**۳. عملیات اصلی**

| عملیات | کار |
|---|---|
| `put()` | آپلود فایل |
| `del()` | حذف فایل |
| `list()` | لیست فایل‌ها |
| `head()` | اطلاعات یک فایل |
| `copy()` | کپی فایل |

**۴. دسترسی عمومی در برابر خصوصی**

| نوع | دسترسی | مناسب برای |
|---|---|---|
| **Public** | همه با URL | تصاویر عمومی |
| **Private** | با توکن | فایل‌های کاربر، اسناد محرمانه |

**۵. محدودیت حجم**
هر فایل محدودیت حجمی دارد (طبق پلن). برای فایل‌های بسیار بزرگ، از آپلود چندبخشی (Multipart) استفاده کنید.

### مثال‌های عملی

**مثال ۱ — آپلود فایل از یک فرم**

```js
// api/upload.js
import { put } from '@vercel/blob';

export const config = {
  api: { bodyParser: false },   // برای دریافت فایل خام لازم است
};

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'فقط POST' });
  }

  const filename = req.headers['x-filename'] ?? `file-${Date.now()}`;

  try {
    // آپلود به Blob
    const blob = await put(filename, req, {
      access: 'public',
      addRandomSuffix: true,     // جلوگیری از تداخل نام
    });

    // در عمل: این URL را در Postgres ذخیره کنید
    return res.status(200).json({
      url: blob.url,
      size: blob.size,
      contentType: blob.contentType,
    });
  } catch (err) {
    console.error(err);
    return res.status(500).json({ error: 'آپلود ناموفق' });
  }
}
```

**مثال ۲ — آپلود از سمت کلاینت (روش پیشنهادی)**

```tsx
// components/FileUpload.tsx
'use client';

import { useState } from 'react';
import { upload } from '@vercel/blob/client';

export default function FileUpload() {
  const [url, setUrl] = useState<string | null>(null);
  const [progress, setProgress] = useState(0);

  async function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
    const file = e.target.files?.[0];
    if (!file) return;

    // آپلود مستقیم از کلاینت به Blob (بدون عبور از تابع)
    const blob = await upload(file.name, file, {
      access: 'public',
      handleUploadUrl: '/api/upload-token',
      onUploadProgress: ({ percentage }) => setProgress(percentage),
    });

    setUrl(blob.url);
  }

  return (
    <div>
      <input type="file" onChange={handleChange} />
      {progress > 0 && <p>پیشرفت: {progress.toFixed(0)}%</p>}
      {url && <a href={url}>مشاهده فایل آپلودشده</a>}
    </div>
  );
}
```

```js
// api/upload-token.js — صدور توکن آپلود
import { handleUpload } from '@vercel/blob/client';

export default async function handler(req, res) {
  const body = req.body;

  const jsonResponse = await handleUpload({
    body,
    request: req,
    onBeforeGenerateToken: async (pathname) => {
      // اعتبارسنجی: فقط کاربران مجاز
      const user = await getUser(req);
      if (!user) throw new Error('دسترسی ندارید');

      return {
        allowedContentTypes: ['image/jpeg', 'image/png', 'application/pdf'],
        maximumSizeInBytes: 5 * 1024 * 1024,   // ۵ مگابایت
        tokenPayload: JSON.stringify({ userId: user.id }),
      };
    },
    onUploadCompleted: async ({ blob, tokenPayload }) => {
      const { userId } = JSON.parse(tokenPayload ?? '{}');
      // ذخیره در دیتابیس
      await saveFileRecord(userId, blob.url);
    },
  });

  res.status(200).json(jsonResponse);
}
```

**مثال ۳ — لیست و حذف فایل**

```js
// api/files.js
import { list, del } from '@vercel/blob';

export default async function handler(req, res) {
  if (req.method === 'GET') {
    const { blobs } = await list({ prefix: 'uploads/', limit: 50 });
    return res.status(200).json(blobs);
  }

  if (req.method === 'DELETE') {
    const { url } = req.body;
    if (!url) return res.status(400).json({ error: 'url لازم است' });

    await del(url);
    return res.status(200).json({ deleted: true });
  }

  res.status(405).end();
}
```

**مثال ۴ — فایل خصوصی با توکن**

```js
// api/private-file.js
import { put } from '@vercel/blob';

export default async function handler(req, res) {
  const blob = await put('private/report.pdf', req, {
    access: 'private',      // ← فقط با توکن قابل دسترسی
  });

  // برای دسترسی، یک URL امضاشده بسازید
  res.json({ url: blob.url });
}
```

### مقایسه و الگوهای مختلف

**Blob در برابر ذخیره در پروژه**

| معیار | `public/` | Blob |
|---|---|---|
| پایداری | ❌ با هر Deploy | ✅ |
| مناسب آپلود کاربر | ❌ | ✅ |
| CDN | ✅ | ✅ |
| حجم | محدود (مخزن) | زیاد |
| دسترسی خصوصی | ❌ | ✅ |

**آپلود از سرور در برابر از کلاینت**

| معیار | از سرور | از کلاینت (مستقیم) |
|---|---|---|
| سرعت | کندتر (عبور از تابع) | ✅ سریع‌تر |
| محدودیت حجم | محدودیت تابع | بالاتر |
| امنیت | کنترل کامل | نیاز به توکن |
| مناسب برای | فایل کوچک | فایل بزرگ، ویدیو |

### اشکال شایع

**۱. فراموش کردن `bodyParser: false`.**
دریافت فایل خام با بدنهٔ parse‌شده ممکن نیست.

**۲. آپلود بدون اعتبارسنجی نوع/حجم.**
کاربر می‌تواند فایل مخرب آپلود کند. همیشه `allowedContentTypes` و `maximumSizeInBytes` بگذارید.

**۳. استفاده از نام فایل کاربر بدون تغییر.**
تداخل و مشکلات امنیتی. از `addRandomSuffix: true` استفاده کنید.

**۴. آپلود فایل بزرگ از سرور.**
محدودیت تابع را رد می‌کند. از آپلود مستقیم کلاینت استفاده کنید.

**۵. فایل خصوصی با دسترسی عمومی.**
اطلاعات محرمانه لو می‌رود. `access: 'private'` بگذارید.

### نکات مهم (Tips & Tricks)

- برای فایل‌های بزرگ، **آپلود مستقیم از کلاینت** را انتخاب کنید.
- همیشه **نوع و حجم** را اعتبارسنجی کنید.
- `addRandomSuffix: true` برای جلوگیری از تداخل.
- لینک فایل را در **Postgres** ذخیره کنید، نه خود فایل.
- فایل‌های قدیمی و بی‌استفاده را با `del` پاک کنید (مدیریت هزینه).

### خلاصه فصل

- **Blob** برای ذخیرهٔ تصویر، ویدیو و فایل است.
- عملیات: **`put`، `del`، `list`، `head`**.
- دسترسی می‌تواند **عمومی یا خصوصی** باشد.
- آپلود مستقیم کلاینت برای فایل‌های بزرگ بهتر است.
- همیشه نوع و حجم را اعتبارسنجی کنید.

---

<a id="فصل-۵"></a>
## فصل ۵: Edge Config

### مقدمه

گاهی می‌خواهید یک تنظیم را **بدون Deploy** تغییر دهید: مثلاً یک Feature Flag را روشن/خاموش کنید یا لیست کشورهای مسدود را عوض کنید. **Edge Config** برای همین ساخته شده: خواندن فوق‌سریع در لبه.

### مفاهیم اساسی

**۱. Edge Config چیست؟**
یک ذخیره‌گاه کلید-مقدار **فوق‌سریع** که در **همهٔ نقاط لبه** کپی می‌شود. خواندنش چند میلی‌ثانیه است.

**۲. تفاوت با KV**

| معیار | Edge Config | KV |
|---|---|---|
| سرعت خواندن | ✅✅ فوق‌سریع | ✅ سریع |
| محل | همهٔ نقاط لبه | دیتاسنتر |
| نوشتن | کم (از Dashboard/API) | زیاد |
| مناسب برای | تنظیمات، Feature Flag | نشست، کش |
| تغییرات | کم‌تکرار | پرتکرار |

**۳. کاربردها**
- **Feature Flags** (فعال/غیرفعال کردن قابلیت).
- **لیست‌های مسدود** (IP، کشور، دامنه).
- **تنظیمات A/B Testing**.
- **Redirect Rules** پویا.
- **کلیدهای عمومی** (مثل شناسهٔ آنالیتیکس).

**۴. نکتهٔ مهم دربارهٔ حجم**
Edge Config برای دادهٔ **کوچک** (چند کیلوبایت) طراحی شده. دادهٔ بزرگ را اینجا نگذارید.

### مثال‌های عملی

**مثال ۱ — Feature Flag ساده**

```js
// api/features.js
import { get } from '@vercel/edge-config';

export default async function handler(req, res) {
  const newCheckout = await get('newCheckoutEnabled');

  res.status(200).json({
    newCheckout: Boolean(newCheckout),
  });
}
```

**مثال ۲ — استفاده در Middleware**

```ts
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { get } from '@vercel/edge-config';

export async function middleware(request: NextRequest) {
  // بررسی Feature Flag در لبه
  const maintenance = await get<boolean>('maintenanceMode');

  if (maintenance && request.nextUrl.pathname !== '/maintenance') {
    return NextResponse.rewrite(new URL('/maintenance', request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/((?!_next|api|maintenance|.*\\..*).*)'],
};
```

**مثال ۳ — لیست مسدود (Blocklist)**

```js
// api/blocked.js
import { get } from '@vercel/edge-config';

export default async function handler(req, res) {
  const country = req.headers['x-vercel-ip-country'];
  const blocked = (await get<string[]>('blockedCountries')) ?? [];

  if (blocked.includes(country)) {
    return res.status(403).json({ error: 'دسترسی از کشور شما مجاز نیست' });
  }

  res.json({ ok: true });
}
```

**مثال ۴ — خواندن چند مقدار با هم**

```js
// api/config.js
import { getAll } from '@vercel/edge-config';

export default async function handler(req, res) {
  // همهٔ تنظیمات را یک‌جا بخوان (بهینه‌تر)
  const config = await getAll();

  res.json({
    features: {
      newCheckout: config.newCheckoutEnabled,
      darkMode: config.darkModeDefault,
    },
    analyticsId: config.analyticsId,
  });
}
```

### مقایسه و الگوهای مختلف

**Edge Config در برابر متغیر محیطی**

| معیار | Env Var | Edge Config |
|---|---|---|
| تغییر بدون Deploy | ❌ | ✅ |
| سرعت خواندن | فوری (در حافظه) | بسیار سریع |
| مناسب Feature Flag | ⚠️ | ✅ |
| مناسب کلید مخفی | ✅ | ❌ (مقدار قابل خواندن) |
| قابل ویرایش در زمان اجرا | ❌ | ✅ |

**سه روش Feature Flag**

| روش | مزیت | عیب |
|---|---|---|
| **Edge Config** | سریع، بدون Deploy | حجم محدود |
| **KV** | انعطاف، داده بیشتر | کمی کندتر |
| **سرویس بیرونی** | امکانات کامل | وابستگی بیرونی |

### اشکال شایع

**۱. گذاشتن دادهٔ حساس در Edge Config.**
مقادیر قابل خواندن‌اند. کلید مخفی را اینجا نگذارید.

**۲. استفاده برای دادهٔ حجیم.**
حجم محدود است. برای دادهٔ زیاد از KV یا Postgres استفاده کنید.

**۳. انتظار تغییرات لحظه‌ای.**
Edge Config برای داده‌ای است که **کم** تغییر می‌کند؛ برای دادهٔ پویا مناسب نیست.

**۴. نوشتن مکرر در Edge Config.**
از طریق API/Dashboard انجام می‌شود و برای نوشتن پرتکرار مناسب نیست.

**۵. فراموش کردن مقادیر پیش‌فرض.**
اگر کلید وجود نداشته باشد، `get` مقدار `undefined` می‌دهد. همیشه مقدار پیش‌فرض بگذارید.

### نکات مهم (Tips & Tricks)

- برای **Feature Flag** و **تنظیمات** استفاده کنید.
- با `getAll()` همه را یک‌جا بخوانید (کمتر رفت‌وبرگشت).
- همیشه مقدار **پیش‌فرض** در نظر بگیرید.
- برای دادهٔ پویا و پرتغییر از **KV** استفاده کنید.
- کلید مخفی را اینجا نگذارید.

### خلاصه فصل

- **Edge Config** ذخیره‌گاه فوق‌سریع تنظیمات در لبه است.
- کاربرد اصلی: **Feature Flag، Blocklist، تنظیمات A/B**.
- تغییر **بدون Deploy** ممکن است.
- برای دادهٔ حساس یا حجیم مناسب نیست.
- `get` و `getAll` عملیات اصلی هستند.

---

<a id="فصل-۶"></a>
## فصل ۶: انتخاب درست و مثال پروژه واقعی

### مقدمه

حالا که هر چهار سرویس را می‌شناسیم، در این فصل یک پروژهٔ واقعی می‌سازیم که هر چهار را کنار هم استفاده می‌کند: یک اپ اشتراک عکس.

### مفاهیم اساسی

**۱. معماری پروژه: اپ اشتراک عکس**

```
┌──────────────────────────────────────────────┐
│              Vercel Edge Network              │
├──────────────────────────────────────────────┤
│  Edge Config  →  Feature Flags، لیست مسدود    │
│  Middleware   →  احراز هویت، Geo              │
├──────────────────────────────────────────────┤
│  KV           →  نشست، کش فید، شمارنده لایک   │
│  Postgres     →  کاربران، پست‌ها، لایک‌ها      │
│  Blob         →  تصاویر آپلودشده              │
├──────────────────────────────────────────────┤
│  Serverless   →  API ها                        │
└──────────────────────────────────────────────┘
```

**۲. تصمیم‌گیری برای هر بخش**

| بخش | سرویس | دلیل |
|---|---|---|
| ثبت‌نام/لاگین | Postgres + KV | داده پایدار + نشست سریع |
| فید پست‌ها | Postgres + KV (کش) | کوئری + کش |
| آپلود تصویر | Blob | فایل، نه دیتابیس |
| شمارش لایک | KV | شمارندهٔ سریع |
| Feature Flag | Edge Config | بدون Deploy |
| کش تنظیمات | Edge Config | فوق‌سریع |

### مثال‌های عملی

**مثال ۱ — اسکیمای دیتابیس**

```sql
-- migration.sql
CREATE TABLE IF NOT EXISTS users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  avatar_url TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS posts (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  image_url TEXT NOT NULL,
  caption TEXT,
  likes INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX IF NOT EXISTS idx_posts_user ON posts(user_id);
CREATE INDEX IF NOT EXISTS idx_posts_created ON posts(created_at DESC);
```

**مثال ۲ — API فید با کش**

```js
// api/feed.js
import { sql } from '@vercel/postgres';
import { kv } from '@vercel/kv';

export default async function handler(req, res) {
  const page = Number(req.query.page ?? 1);
  const CACHE_KEY = `feed:page:${page}`;

  // ۱. کش
  const cached = await kv.get(CACHE_KEY);
  if (cached) {
    return res.status(200).json({ source: 'cache', posts: cached });
  }

  // ۲. دیتابیس
  const { rows } = await sql`
    SELECT p.id, p.image_url, p.caption, p.likes, p.created_at,
           u.name AS author_name, u.avatar_url AS author_avatar
    FROM posts p
    JOIN users u ON u.id = p.user_id
    ORDER BY p.created_at DESC
    LIMIT 20 OFFSET ${(page - 1) * 20}
  `;

  // ۳. ذخیره در کش (۱ دقیقه)
  await kv.set(CACHE_KEY, rows, { ex: 60 });

  res.status(200).json({ source: 'db', posts: rows });
}
```

**مثال ۳ — API لایک با شمارندهٔ KV**

```js
// api/posts/[id]/like.js
import { kv } from '@vercel/kv';
import { sql } from '@vercel/postgres';

export default async function handler(req, res) {
  const { id } = req.query;
  const userId = req.headers['x-user-id'];   // از Middleware

  if (req.method !== 'POST') return res.status(405).end();

  // جلوگیری از لایک تکراری با Set
  const liked = await kv.sadd(`post:${id}:likers`, userId);
  if (!liked) {
    return res.status(200).json({ message: 'قبلاً لایک کرده‌اید' });
  }

  // شمارنده
  const likes = await kv.incr(`post:${id}:likes`);

  // همگام‌سازی دوره‌ای با دیتابیس (async، بدون انتظار)
  sql`UPDATE posts SET likes = ${likes} WHERE id = ${id}`.catch(console.error);

  // پاک کردن کش فید
  await kv.del('feed:page:1');

  res.status(200).json({ likes });
}
```

**مثال ۴ — آپلود تصویر با اعتبارسنجی**

```js
// api/posts/create.js
import { put } from '@vercel/blob';
import { sql } from '@vercel/postgres';

export const config = { api: { bodyParser: false } };

export default async function handler(req, res) {
  const userId = req.headers['x-user-id'];
  if (!userId) return res.status(401).json({ error: 'وارد نشده‌اید' });

  const filename = req.headers['x-filename'] ?? `photo-${Date.now()}.jpg`;

  try {
    const blob = await put(`posts/${userId}/${filename}`, req, {
      access: 'public',
      addRandomSuffix: true,
      contentType: 'image/jpeg',
    });

    const caption = req.headers['x-caption'] ?? '';

    const { rows } = await sql`
      INSERT INTO posts (user_id, image_url, caption)
      VALUES (${userId}, ${blob.url}, ${caption})
      RETURNING id, image_url, caption, created_at
    `;

    // پاک کردن کش فید
    await kv.del('feed:page:1');

    res.status(201).json(rows[0]);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'ایجاد پست ناموفق' });
  }
}
```

**مثال ۵ — Middleware ترکیبی (Edge Config + KV)**

```ts
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { get } from '@vercel/edge-config';
import { kv } from '@vercel/kv';

export async function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;

  // ۱. حالت تعمیر (Edge Config)
  const maintenance = await get<boolean>('maintenanceMode');
  if (maintenance && pathname !== '/maintenance') {
    return NextResponse.rewrite(new URL('/maintenance', request.url));
  }

  // ۲. بررسی نشست (KV)
  const sessionId = request.cookies.get('session')?.value;
  const protectedPaths = ['/upload', '/profile', '/settings'];
  const needsAuth = protectedPaths.some((p) => pathname.startsWith(p));

  if (needsAuth) {
    if (!sessionId) {
      return NextResponse.redirect(new URL('/login', request.url));
    }

    const session = await kv.get<{ userId: number }>(`session:${sessionId}`);
    if (!session) {
      return NextResponse.redirect(new URL('/login', request.url));
    }

    // پاس دادن شناسهٔ کاربر به API
    const response = NextResponse.next();
    response.headers.set('x-user-id', String(session.userId));
    return response;
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/((?!_next|api|login|maintenance|.*\\..*).*)'],
};
```

### مقایسه و الگوهای مختلف

**سه الگوی معماری داده**

| الگو | توضیح | مناسب برای |
|---|---|---|
| **ساده** | فقط Postgres | پروژه‌های کوچک |
| **کش‌شده** | Postgres + KV | اکثر اپ‌های واقعی |
| **چندلایه** | Edge Config + KV + Postgres + Blob | اپ‌های بزرگ |

**هزینه در برابر پیچیدگی**

| سطح | سرویس‌ها | هزینه | پیچیدگی |
|---|---|---|---|
| شروع | Postgres | کم | کم |
| رشد | + KV + Blob | متوسط | متوسط |
| مقیاس | + Edge Config + کش | بهینه | زیاد |

### اشکال شایع

**۱. استفاده از همهٔ سرویس‌ها از روز اول.**
پیچیدگی بی‌مورد. با Postgres شروع کنید و در صورت نیاز اضافه کنید.

**۲. نبود Invalidating کش.**
بعد از تغییر داده، کش قدیمی می‌ماند. همیشه کش را پاک کنید.

**۳. نبود ایندکس در Postgres.**
کوئری فید کند می‌شود. روی ستون‌های مرتب‌سازی و فیلتر ایندکس بگذارید.

**۴. ذخیرهٔ تصویر در دیتابیس.**
دیتابیس را سنگین می‌کند. تصویر در Blob، لینک در دیتابیس.

**۵. عدم همگام‌سازی شمارنده با دیتابیس.**
اگر فقط در KV بشمارید، داده در دیتابیس قدیمی می‌ماند. دوره‌ای همگام کنید.

### نکات مهم (Tips & Tricks)

- **از ساده شروع کنید**: فقط Postgres.
- کش را **دوره‌ای** و با **Invalidation** مدیریت کنید.
- شمارنده‌های سریع (لایک، بازدید) در **KV**.
- فایل‌ها در **Blob**، لینک در **Postgres**.
- Feature Flag ها در **Edge Config**.
- همهٔ اتصال‌ها در **Environment Variables**.

### خلاصه فصل

- هر سرویس نقش مشخصی دارد؛ آن‌ها را ترکیب کنید.
- الگوی حرفه‌ای: **Edge Config → KV → Postgres → Blob**.
- از ساده شروع کنید و بر اساس نیاز اضافه کنید.
- کش، ایندکس و Invalidation کلید کارایی هستند.
- فایل همیشه در Blob، لینک در دیتابیس.

---

## منابع و مراجع

### مستندات رسمی
- Vercel Storage Overview: <https://vercel.com/docs/storage>
- Vercel KV: <https://vercel.com/docs/storage/vercel-kv>
- Vercel Postgres: <https://vercel.com/docs/storage/vercel-postgres>
- Vercel Blob: <https://vercel.com/docs/storage/vercel-blob>
- Edge Config: <https://vercel.com/docs/storage/edge-config>
- Marketplace Storage: <https://vercel.com/marketplace>

### مفاهیم پیشنهادی برای مطالعه بیشتر
- Redis Documentation: <https://redis.io/docs/>
- PostgreSQL Tutorial: <https://www.postgresql.org/docs/>
- Drizzle ORM: <https://orm.drizzle.team>
- Prisma: <https://www.prisma.io/docs>

### گام بعدی
- [فایل ۰۹ — Cron، Webhooks، Integrations و REST API](./09-Cron-Webhooks-Integrations-API.md)
- [فایل ۱۰ — بهینه‌سازی عملکرد](./10-Performance-Optimization.md)
