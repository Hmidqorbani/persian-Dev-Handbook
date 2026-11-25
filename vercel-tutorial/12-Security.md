# ۱۲ — امنیت

**مدت یادگیری**: ۳ تا ۴ ساعت
**سطح سختی**: متوسط تا پیشرفته
**پیش‌نیاز**: مطالعه فایل‌های ۰۵ (توابع)، ۰۷ (متغیرها) و ۰۹ (Webhook)

---

## فهرست محتوا

- [فصل ۱: اصول امنیت در وب مدرن](#فصل-۱)
- [فصل ۲: Secrets و مدیریت اطلاعات حساس](#فصل-۲)
- [فصل ۳: Vercel Firewall](#فصل-۳)
- [فصل ۴: Bot Management و Rate Limiting](#فصل-۴)
- [فصل ۵: DDoS و محافظت در لبه](#فصل-۵)
- [فصل ۶: چک‌لیست امنیتی پروژه](#فصل-۶)

---

<a id="فصل-۱"></a>
## فصل ۱: اصول امنیت در وب مدرن

### مقدمه

امنیت یک قابلیت نیست؛ یک **عادت** است. در این فصل، ذهنیت امنیتی و تهدیدهای اصلی را می‌شناسیم تا بدانیم در ادامه چه چیزی را باید محافظت کنیم.

### مفاهیم اساسی

**۱. سه اصل امنیت (CIA)**

| اصل | معنی |
|---|---|
| **Confidentiality** | فقط افراد مجاز داده را ببینند |
| **Integrity** | داده دست‌نخورده و معتبر بماند |
| **Availability** | سرویس همیشه در دسترس باشد |

**۲. تهدیدهای اصلی (OWASP Top 10 خلاصه)**

| تهدید | توضیح | مثال |
|---|---|---|
| **Injection** | تزریق کد | SQL Injection |
| **Broken Auth** | احراز هویت ضعیف | رمز ساده |
| **Sensitive Data Exposure** | افشای داده | کلید در کد |
| **Broken Access Control** | کنترل دسترسی ضعیف | دسترسی به داده دیگران |
| **Security Misconfiguration** | پیکربندی اشتباه | CORS باز |
| **XSS** | تزریق اسکریپت | `<script>` در کامنت |
| **CSRF** | جعل درخواست | فرم جعلی |
| **SSRF** | جعل درخواست سرور | فراخوانی URL داخلی |

**۳. اصل کمترین دسترسی (Least Privilege)**
هر جزء فقط به اندازهٔ نیازش دسترسی داشته باشد.

**۴. دفاع لایه‌ای (Defense in Depth)**
یک لایه کافی نیست. چند لایه داشته باشید:
```
مرورگر → Edge/Firewall → Middleware → Route Handler → دیتابیس
```

**۵. اعتماد نکردن به ورودی کاربر**
هر ورودی از بیرون **غیرقابل اعتماد** است: فرم، URL، هدر، کوکی، Webhook.

**۶. Fail Securely**
اگر خطایی رخ داد، به‌طور پیش‌فرض **دسترسی را ببند**، نه باز.

### مثال‌های عملی

**مثال ۱ — جلوگیری از SQL Injection**

```js
// ❌ خطرناک: رشته‌سازی مستقیم
const query = `SELECT * FROM users WHERE email = '${email}'`;
// کاربر می‌تواند بفرستد: ' OR '1'='1

// ✅ امن: پارامتری
const { rows } = await sql`SELECT * FROM users WHERE email = ${email}`;
```

**مثال ۲ — جلوگیری از XSS**

```tsx
// ❌ خطرناک
<div dangerouslySetInnerHTML={{ __html: userComment }} />

// ✅ امن: React خودش escape می‌کند
<div>{userComment}</div>

// ✅ اگر HTML لازم است، پاک‌سازی کنید
import DOMPurify from 'isomorphic-dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userComment) }} />
```

**مثال ۳ — کنترل دسترسی (IDOR)**

```js
// ❌ خطرناک: هر کاربری می‌تواند سفارش دیگران را ببیند
const order = await sql`SELECT * FROM orders WHERE id = ${orderId}`;

// ✅ امن: بررسی مالکیت
const order = await sql`
  SELECT * FROM orders WHERE id = ${orderId} AND user_id = ${currentUserId}
`;
if (!order.rows[0]) return res.status(404).json({ error: 'پیدا نشد' });
```

**مثال ۴ — اعتبارسنجی ورودی**

```js
// lib/validate.js
export function validateUser(input) {
  const errors = [];

  if (typeof input.email !== 'string' || !/^[^@]+@[^@]+\.[^@]+$/.test(input.email)) {
    errors.push('ایمیل نامعتبر');
  }
  if (typeof input.name !== 'string' || input.name.trim().length < 2) {
    errors.push('نام باید حداقل ۲ حرف باشد');
  }
  if (typeof input.age !== 'number' || input.age < 13 || input.age > 120) {
    errors.push('سن نامعتبر');
  }

  return { valid: errors.length === 0, errors };
}
```

### مقایسه و الگوهای مختلف

**روش‌های احراز هویت**

| روش | امنیت | پیچیدگی | مناسب برای |
|---|---|---|---|
| **Session + Cookie** | ✅ بالا | متوسط | اپ‌های وب |
| **JWT** | متوسط | متوسط | API، موبایل |
| **OAuth** | ✅ بالا | زیاد | ورود با گوگل/گیت‌هاب |
| **Passkey** | ✅✅ بالا | متوسط | آینده |

**کوکی امن در برابر ناامن**

| ویژگی | امن | ناامن |
|---|---|---|
| `HttpOnly` | ✅ | ❌ (قابل خواندن با JS) |
| `Secure` | ✅ | ❌ |
| `SameSite` | `Lax`/`Strict` | `None` بدون Secure |

### اشکال شایع

**۱. اعتماد به اعتبارسنجی سمت کلاینت.**
هر کسی می‌تواند درخواست دستی بفرستد. همیشه **سمت سرور** هم اعتبارسنجی کنید.

**۲. نمایش پیام خطای دقیق به کاربر.**
به مهاجم اطلاعات می‌دهد. پیام عمومی بدهید، جزئیات را لاگ کنید.

**۳. CORS با `*`.**
هر دامنه‌ای می‌تواند درخواست بزند. فقط دامنه‌های مجاز را لیست کنید.

**۴. نبود بررسی مالکیت (IDOR).**
کاربر می‌تواند دادهٔ دیگران را با تغییر ID ببیند.

**۵. ذخیرهٔ رمز بدون هش.**
رمزها باید با **bcrypt/argon2** هش شوند.

### نکات مهم (Tips & Tricks)

- **هر ورودی را غیرقابل اعتماد بدانید**.
- اعتبارسنجی **دو طرفه** (کلاینت + سرور).
- کوکی‌ها با `HttpOnly; Secure; SameSite=Lax`.
- رمزها با **bcrypt/argon2** هش شوند.
- پیام‌های خطای عمومی بدهید.

### خلاصه فصل

- سه اصل: **محرمانگی، یکپارچگی، دسترس‌پذیری**.
- تهدیدهای اصلی: **Injection، XSS، CSRF، IDOR، SSRF**.
- **اصل کمترین دسترسی** و **دفاع لایه‌ای** را رعایت کنید.
- هر ورودی غیرقابل اعتماد است.
- اعتبارسنجی همیشه سمت سرور.

---

<a id="فصل-۲"></a>
## فصل ۲: Secrets و مدیریت اطلاعات حساس

### مقدمه

بیشتر نشت‌های امنیتی از یک چیز ساده شروع می‌شوند: **کلید لو رفته**. در این فصل، مدیریت درست Secrets را یاد می‌گیریم.

### مفاهیم اساسی

**۱. Secret چیست؟**
هر داده‌ای که اگر لو برود خطرناک است: کلید API، رمز دیتابیس، توکن، کلید رمزنگاری.

**۲. قوانین طلایی**

| قانون | دلیل |
|---|---|
| هرگز در کد ننویس | در Git می‌ماند |
| هرگز Commit نکن | تاریخچه پاک نمی‌شود |
| فقط در Env Vars | جدا از کد |
| دوره‌ای چرخش بده | محدودسازی نشت |
| متفاوت در هر محیط | تست ≠ تولید |

**۳. متغیر حساس در Vercel**
متغیری که پس از ساخت **قابل خواندن نیست** — فقط قابل جایگزینی.

**۴. `.env.example`**
فایلی که **نام** متغیرها را دارد (بدون مقدار) و در Git می‌ماند — راهنمای تیم.

**۵. چرخش (Rotation)**
جایگزینی دوره‌ای کلیدها تا اگر لو رفت، مدت اعتبارش کوتاه باشد.

**۶. اسکنرهای Secret**
ابزارهایی که مخزن را برای کلید لو رفته بررسی می‌کنند (GitHub Secret Scanning، TruffleHog).

### مثال‌های عملی

**مثال ۱ — ساختار درست فایل‌ها**

```bash
# .env.local (در Git نیست)
DATABASE_URL="postgres://..."
STRIPE_SECRET_KEY="sk_live_..."
JWT_SECRET="random-long-string"

# .env.example (در Git هست — فقط نام‌ها)
DATABASE_URL=""
STRIPE_SECRET_KEY=""
JWT_SECRET=""
```

```gitignore
# .gitignore
.env
.env.local
.env.*.local
.vercel
```

**مثال ۲ — بررسی نشت در کد**

```bash
# جست‌وجوی الگوهای مشکوک
grep -rn "sk_live" --include="*.js" --include="*.ts" .
grep -rn "password.*=.*['\"]" --include="*.js" .
grep -rn "api[_-]?key" --include="*.ts" -i .
```

**مثال ۳ — تأیید وجود Secrets در زمان اجرا**

```js
// lib/env.js — بررسی در شروع برنامه
const REQUIRED = [
  'DATABASE_URL',
  'JWT_SECRET',
  'STRIPE_SECRET_KEY',
];

export function checkEnv() {
  const missing = REQUIRED.filter((k) => !process.env[k]);

  if (missing.length > 0) {
    throw new Error(`متغیرهای محیطی غایب: ${missing.join(', ')}`);
  }
}
```

```js
// api/startup.js
import { checkEnv } from '../lib/env';

export default function handler(req, res) {
  try {
    checkEnv();
    res.json({ ok: true });
  } catch (err) {
    console.error(err.message);
    res.status(500).json({ error: 'پیکربندی ناقص' });
  }
}
```

**مثال ۴ — چرخش کلید بدون قطع سرویس**

```js
// lib/keys.js — پشتیبانی از دو کلید در زمان چرخش
export function getActiveKey() {
  return process.env.API_KEY_CURRENT;
}

export function getLegacyKey() {
  return process.env.API_KEY_PREVIOUS;
}

export function verifyApiKey(provided) {
  // در زمان چرخش، هر دو کلید معتبرند
  return provided === getActiveKey() || provided === getLegacyKey();
}
```

**مثال ۵ — اسکن خودکار در CI**

```yaml
# .github/workflows/secret-scan.yml
name: Secret Scan

on: [push, pull_request]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          head: HEAD
```

### مقایسه و الگوهای مختلف

**سه روش نگهداری Secrets**

| روش | امنیت | مناسب برای |
|---|---|---|
| **Env Vars پلتفرم** | ✅ بالا | اکثر پروژه‌ها |
| **Secret Manager ابری** | ✅✅ بالا | سازمانی |
| **فایل رمزنگاری‌شده** | متوسط | پروژه‌های خاص |

**چه زمانی چرخش؟**

| رویداد | اقدام |
|---|---|
| لو رفتن مشکوک | فوری |
| ترک کارمند | فوری |
| دوره‌ای | هر ۹۰ روز |
| پس از انتشار نسخهٔ بزرگ | پیشنهادی |

### اشکال شایع

**۱. Commit کردن `.env`.**
رایج‌ترین اشتباه. اگر رخ داد، **فوراً کلیدها را عوض کنید** (حذف فایل کافی نیست).

**۲. کلید در کد سمت کلاینت.**
هر چیزی در باندل کلاینت قابل خواندن است.

**۳. `NEXT_PUBLIC_` برای کلید مخفی.**
در مرورگر افشا می‌شود.

**۴. یک کلید برای همهٔ محیط‌ها.**
اگر Preview لو برود، Production در خطر است.

**۵. لاگ کردن Secret.**
```js
// ❌ هرگز
console.log('کلید:', process.env.API_KEY);
// ✅ فقط وضعیت
console.log('کلید تنظیم شده:', Boolean(process.env.API_KEY));
```

### نکات مهم (Tips & Tricks)

- `.env.example` بسازید و در Git بگذارید.
- Secrets را **Sensitive** علامت بزنید.
- **Secret Scanning** را در GitHub فعال کنید.
- دورهٔ **چرخش** تعیین کنید.
- کلید تست و تولید را **کاملاً جدا** نگه دارید.

### خلاصه فصل

- Secrets هرگز در کد یا Git نباشند.
- فقط در **Environment Variables**، ترجیحاً Sensitive.
- `.env.example` راهنمای تیم است.
- **چرخش دوره‌ای** کلیدها ضروری است.
- Secret Scanning را خودکار کنید.

---

<a id="فصل-۳"></a>
## فصل ۳: Vercel Firewall

### مقدمه

قبل از اینکه درخواست به کد شما برسد، می‌توانید آن را در **لبه** بررسی و مسدود کنید. **Vercel Firewall** یک WAF (Web Application Firewall) است که این کار را انجام می‌دهد.

### مفاهیم اساسی

**۱. Firewall چیست؟**
لایه‌ای که درخواست‌های ورودی را قبل از رسیدن به اپ بررسی می‌کند و می‌تواند آن‌ها را **مسدود**، **چالش** یا **مجاز** کند.

**۲. انواع قوانین**

| نوع | توضیح |
|---|---|
| **Managed Ruleset** | قوانین آماده (مقابله با حملات رایج) |
| **Custom Rules** | قوانین شما بر اساس شرط |
| **IP Blocking** | مسدودسازی IP |
| **System Bypass** | مسیرهای معاف (مثل health check) |

**۳. اقدامات (Actions)**

| اقدام | کار |
|---|---|
| **Log** | فقط ثبت (بدون مسدودسازی) |
| **Challenge** | چالش مرورگر (اثبات انسان بودن) |
| **Deny** | مسدود کردن (۴۰۳) |
| **Bypass** | اجازهٔ کامل |

**۴. معیارهای شرط**

| معیار | مثال |
|---|---|
| IP / CIDR | `203.0.113.0/24` |
| Country | `IR`, `US` |
| Path | `/api/admin/*` |
| Method | `POST` |
| Header | `User-Agent` |
| Rate | تعداد درخواست |

**۵. حالت نظارتی (Monitor Mode)**
قبل از مسدودسازی، یک قانون را در حالت **Log** اجرا کنید تا ببینید چه چیزی مسدود می‌شود (جلوگیری از مسدودسازی اشتباه کاربران واقعی).

### مثال‌های عملی

**مثال ۱ — مسدودسازی مسیرهای حساس**

```
۱. Project → Firewall → Rules → Create Rule
۲. Name: Block admin from outside
۳. Condition:
   - Path matches /admin/*
   - Country is not IR
۴. Action: Deny
۵. Save
```

**مثال ۲ — مسدودسازی IP مخرب**

```
۱. Firewall → Rules → Create Rule
۲. Name: Block abusive IP
۳. Condition: IP equals 203.0.113.45
۴. Action: Deny
۵. (اختیاری) Expiration: ۷ روز
```

**مثال ۳ — چالش برای User-Agent مشکوک**

```
Condition:
  - User-Agent contains "curl" OR "python-requests"
  - Path starts with /api/
Action: Challenge
```

**مثال ۴ — محافظت از مسیر API حساس با هدر**

```
Condition:
  - Path matches /api/internal/*
  - Header "x-internal-token" does not exist
Action: Deny
```

**مثال ۵ — حالت نظارتی قبل از اجرا**

```
۱. یک قانون بسازید با Action = Log
۲. چند روز صبر کنید
۳. در Firewall → Observability، درخواست‌های ثبت‌شده را بررسی کنید
۴. اگر مطمئن شدید کاربران واقعی آسیب نمی‌بینند، Action را به Deny/Challenge تغییر دهید
```

### مقایسه و الگوهای مختلف

**Firewall در برابر Middleware**

| معیار | Firewall | Middleware |
|---|---|---|
| محل | لبه (قبل از اپ) | لبه (در اپ) |
| سرعت | ✅✅ بسیار بالا | بالا |
| منطق پیچیده | محدود | ✅ کامل |
| مناسب برای | IP، Country، Rate | auth، i18n، A/B |
| هزینهٔ منابع | صفر برای اپ | مصرف تابع |

**سه سطح محافظت**

| سطح | ابزار | هدف |
|---|---|---|
| **شبکه** | DDoS Protection | حملات حجمی |
| **لبه** | Firewall | حملات لایه ۷ |
| **اپ** | Middleware/کد | منطق کسب‌وکار |

### اشکال شایع

**۱. مسدودسازی بدون حالت نظارتی.**
ممکن است کاربران واقعی را مسدود کنید. اول Log کنید.

**۲. مسدودسازی کل یک کشور.**
می‌تواند ترافیک واقعی را قطع کند. دقیق‌تر باشید.

**۳. نبود System Bypass برای health check.**
ابزارهای مانیتورینگ مسدود می‌شوند.

**۴. قانون بیش از حد گسترده.**
`Path starts with /` همه چیز را مسدود می‌کند.

**۵. فراموش کردن بازبینی قوانین.**
قوانین قدیمی ممکن است مشکل‌ساز شوند. دوره‌ای بازبینی کنید.

### نکات مهم (Tips & Tricks)

- **اول Log، بعد Deny** — همیشه.
- برای مسیرهای حساس، **چند لایه** بگذارید.
- **System Bypass** برای health check و مانیتورینگ.
- قوانین را **مستند و نام‌گذاری** کنید.
- قوانین را **دوره‌ای بازبینی** کنید.

### خلاصه فصل

- **Vercel Firewall** درخواست‌ها را در لبه بررسی می‌کند.
- اقدامات: **Log، Challenge، Deny، Bypass**.
- شرط‌ها: IP، Country، Path، Method، Header، Rate.
- همیشه با **حالت نظارتی** شروع کنید.
- System Bypass برای ابزارهای داخلی.

---

<a id="فصل-۴"></a>
## فصل ۴: Bot Management و Rate Limiting

### مقدمه

بخش بزرگی از ترافیک اینترنت **ربات** است: بعضی مفید (گوگل)، بعضی مضر (اسکرپر، حمله). در این فصل یاد می‌گیریم چطور انسان را از ربات تشخیص دهیم و درخواست‌ها را محدود کنیم.

### مفاهیم اساسی

**۱. انواع ربات**

| نوع | مثال | برخورد |
|---|---|---|
| **خوب** | Googlebot، Bingbot | ✅ اجازه |
| **خنثی** | مانیتورینگ، Uptime | ✅ اجازه |
| **بد** | اسکرپر، حمله‌گر | ❌ مسدود |

**۲. Bot Management چیست؟**
سیستمی که ربات‌ها را از انسان‌ها تشخیص می‌دهد و بر اساس آن تصمیم می‌گیرد.

**۳. رویکردهای تشخیص**

| رویکرد | توضیح |
|---|---|
| **امضای User-Agent** | ساده اما قابل جعل |
| **رفتار** | الگوی درخواست |
| **چالش (Challenge)** | اثبات انسان بودن |
| **BotID** | تشخیص نامرئی مدرن |

**۴. Rate Limiting**
محدود کردن تعداد درخواست در بازهٔ زمانی:

| الگو | توضیح |
|---|---|
| **Fixed Window** | پنجرهٔ ثابت (ساده) |
| **Sliding Window** | پنجرهٔ لغزان (دقیق‌تر) |
| **Token Bucket** | سطل توکن (انعطاف‌پذیر) |

**۵. پاسخ‌های استاندارد**

| کد | معنی |
|---|---|
| **429** | تعداد درخواست زیاد |
| **403** | ممنوع |
| هدر `Retry-After` | زمان انتظار |

### مثال‌های عملی

**مثال ۱ — اجازه به ربات‌های خوب**

```
Firewall Rule:
  Condition:
    - User-Agent contains "Googlebot"
    - (تأیید IP گوگل — پیشنهادی)
  Action: Bypass
```

**مثال ۲ — چالش برای ترافیک مشکوک**

```
Firewall Rule:
  Condition:
    - Path matches /api/*
    - User-Agent does not contain "Mozilla"
  Action: Challenge
```

**مثال ۳ — Rate Limiting با KV**

```js
// lib/rateLimit.js
import { kv } from '@vercel/kv';

export async function rateLimit(identifier, { limit, windowSec }) {
  const key = `rl:${identifier}`;
  const now = Date.now();
  const windowStart = now - windowSec * 1000;

  // استفاده از Sorted Set برای پنجرهٔ لغزان
  await kv.zadd(key, { score: now, member: `${now}:${Math.random()}` });
  await kv.zremrangebyscore(key, 0, windowStart);

  const count = await kv.zcard(key);
  await kv.expire(key, windowSec);

  return {
    allowed: count <= limit,
    remaining: Math.max(0, limit - count),
    resetIn: windowSec,
  };
}
```

```js
// api/data.js
import { rateLimit } from '../lib/rateLimit';

export default async function handler(req, res) {
  const ip = req.headers['x-forwarded-for']?.split(',')[0] ?? 'unknown';
  const { allowed, remaining, resetIn } = await rateLimit(ip, {
    limit: 60,
    windowSec: 60,
  });

  res.setHeader('X-RateLimit-Remaining', remaining);

  if (!allowed) {
    res.setHeader('Retry-After', resetIn);
    return res.status(429).json({ error: 'تعداد درخواست‌ها زیاد است' });
  }

  return res.json({ ok: true });
}
```

**مثال ۴ — Rate Limiting مخصوص لاگین**

```js
// api/auth/login.js
import { rateLimit } from '../../lib/rateLimit';
import bcrypt from 'bcryptjs';

export default async function handler(req, res) {
  const ip = req.headers['x-forwarded-for']?.split(',')[0] ?? 'unknown';
  const { email, password } = req.body;

  // محدودیت سخت‌گیرانه برای لاگین
  const byIp = await rateLimit(`login:ip:${ip}`, { limit: 5, windowSec: 300 });
  const byEmail = await rateLimit(`login:email:${email}`, { limit: 5, windowSec: 300 });

  if (!byIp.allowed || !byEmail.allowed) {
    return res.status(429).json({ error: 'تلاش‌های زیاد. بعداً امتحان کنید.' });
  }

  const user = await findUser(email);

  // پیام یکسان برای جلوگیری از افشای وجود کاربر
  if (!user || !(await bcrypt.compare(password, user.passwordHash))) {
    return res.status(401).json({ error: 'ایمیل یا رمز نادرست' });
  }

  return res.json({ ok: true });
}
```

**مثال ۵ — هدرهای استاندارد Rate Limit**

```js
res.setHeader('X-RateLimit-Limit', 60);
res.setHeader('X-RateLimit-Remaining', remaining);
res.setHeader('X-RateLimit-Reset', Math.floor(Date.now() / 1000) + resetIn);
res.setHeader('Retry-After', resetIn);   // فقط هنگام 429
```

### مقایسه و الگوهای مختلف

**سه سطح Rate Limiting**

| سطح | ابزار | دقت | هزینه |
|---|---|---|---|
| **Firewall** | Vercel Firewall | متوسط | بخشی از پلتفرم |
| **KV** | کد اپلیکیشن | ✅ زیاد | مصرف KV |
| **حافظه** | Map در تابع | کم (ناپایدار) | صفر |

**روش‌های تشخیص ربات**

| روش | دقت | مزاحمت |
|---|---|---|
| User-Agent | کم | صفر |
| رفتار | متوسط | صفر |
| Challenge | ✅ زیاد | کم |
| BotID | ✅ زیاد | صفر (نامرئی) |

### اشکال شایع

**۱. مسدودسازی Googlebot.**
رتبهٔ SEO خراب می‌شود. ربات‌های خوب را مستثنا کنید.

**۲. Rate Limit فقط بر اساس IP.**
IP می‌تواند مشترک باشد (NAT) یا جعل شود. برای عملیات حساس، بر اساس کاربر/ایمیل هم محدود کنید.

**۳. Rate Limit در حافظهٔ تابع.**
ناپایدار است و در چند نمونه کار نمی‌کند. از KV یا Firewall استفاده کنید.

**۴. پیام خطای متفاوت برای کاربر موجود/ناموجود.**
افشای اطلاعات. پیام یکسان بدهید.

**۵. نبود `Retry-After`.**
کلاینت‌های خوب نمی‌دانند چه زمانی دوباره تلاش کنند.

### نکات مهم (Tips & Tricks)

- **ربات‌های خوب را مستثنا کنید**.
- برای لاگین و ثبت‌نام، **Rate Limit سخت‌گیرانه**.
- پیام خطای احراز هویت را **یکسان** نگه دارید.
- هدرهای استاندارد Rate Limit را بفرستید.
- از **KV یا Firewall** برای Rate Limit استفاده کنید (نه حافظه).

### خلاصه فصل

- ربات‌ها سه دسته‌اند: **خوب، خنثی، بد**.
- **Bot Management** انسان را از ربات جدا می‌کند.
- **Rate Limiting** سه الگو دارد: Fixed، Sliding، Token Bucket.
- پاسخ استاندارد: **429 + Retry-After**.
- برای لاگین، محدودیت سخت‌گیرانه و پیام یکسان.

---

<a id="فصل-۵"></a>
## فصل ۵: DDoS و محافظت در لبه

### مقدمه

**DDoS** (حملهٔ انکار سرویس توزیع‌شده) یعنی فرستادن حجم عظیمی از ترافیک برای از کار انداختن سایت. خبر خوب: Vercel بخش زیادی از این را خودکار مدیریت می‌کند.

### مفاهیم اساسی

**۱. لایه‌های حمله**

| لایه | نوع | نمونه |
|---|---|---|
| **L3/L4** | شبکه/حمل‌ونقل | SYN Flood، UDP Flood |
| **L7** | اپلیکیشن | HTTP Flood، Slowloris |

**۲. محافظت Vercel**

| لایه | محافظت |
|---|---|
| L3/L4 | خودکار (Anycast، توزیع‌شده) |
| L7 | Firewall + Attack Challenge Mode |

**۳. Attack Challenge Mode**
حالتی که هنگام حمله فعال می‌کنید: همهٔ بازدیدکنندگان چالش اثبات انسان بودن می‌بینند.

**۴. چرا CDN کمک می‌کند؟**
ترافیک بین صدها نقطه پخش می‌شود و هیچ نقطه‌ای اشباع نمی‌شود.

**۵. Anycast**
یک IP در چند مکان اعلام می‌شود و ترافیک به نزدیک‌ترین نقطه می‌رود — توزیع طبیعی بار.

### مثال‌های عملی

**مثال ۱ — فعال‌سازی Attack Challenge Mode**

```
۱. Project → Firewall
۲. «Attack Challenge Mode» را فعال کنید
۳. همهٔ بازدیدکنندگان چالش می‌بینند
۴. پس از پایان حمله، غیرفعال کنید
```

**مثال ۲ — آماده‌سازی برای حمله (Runbook)**

```
【 شناسایی 】
- افزایش ناگهانی ترافیک
- نرخ خطای بالا
- کندی پاسخ

【 اقدام فوری 】
۱. Firewall → Observability را بررسی کنید
۲. Attack Challenge Mode را فعال کنید
۳. قوانین Deny برای الگوهای حمله بسازید
۴. تیم را مطلع کنید

【 پس از حمله 】
۱. Challenge Mode را خاموش کنید
۲. قوانین موقت را حذف کنید
۳. گزارش Post-mortem بنویسید
```

**مثال ۳ — محدودسازی مسیرهای گران**

```js
// api/expensive.js — مسیر پرهزینه با محدودیت سخت
import { rateLimit } from '../../lib/rateLimit';

export default async function handler(req, res) {
  const ip = req.headers['x-forwarded-for']?.split(',')[0] ?? 'unknown';
  const { allowed } = await rateLimit(`expensive:${ip}`, {
    limit: 5,
    windowSec: 60,
  });

  if (!allowed) {
    return res.status(429).json({ error: 'محدودیت درخواست' });
  }

  // عملیات پرهزینه
  const result = await heavyComputation();
  res.json(result);
}
```

**مثال ۴ — کش لبه به‌عنوان سپر**

```js
// api/public-data.js — کش طولانی، بار را از سرور برمی‌دارد
export default async function handler(req, res) {
  const data = await getPublicData();

  // حمله به این مسیر، به دیتابیس نمی‌رسد
  res.setHeader('Cache-Control', 'public, s-maxage=300, stale-while-revalidate=3600');
  res.json(data);
}
```

### مقایسه و الگوهای مختلف

**سه سطح دفاع در برابر DDoS**

| سطح | ابزار | مسئول |
|---|---|---|
| L3/L4 | Vercel (خودکار) | پلتفرم |
| L7 عمومی | Firewall Managed Rules | پلتفرم |
| L7 هدفمند | Custom Rules + Challenge | شما |

**استراتژی‌های کاهش اثر**

| استراتژی | اثر |
|---|---|
| کش لبه | ✅ زیاد |
| Rate Limiting | ✅ زیاد |
| Challenge Mode | ✅ زیاد |
| صفحات سبک | متوسط |
| Graceful Degradation | متوسط |

### اشکال شایع

**۱. نبود برنامهٔ واکنش.**
هنگام حمله، سردرگمی. Runbook از قبل آماده کنید.

**۲. فراموش کردن خاموش کردن Challenge Mode.**
کاربران واقعی آزار می‌بینند. پس از حمله خاموش کنید.

**۳. مسیرهای بدون کش و پرهزینه.**
هدف آسان برای مهاجم. کش و محدودیت بگذارید.

**۴. نبود مانیتورینگ.**
بدون هشدار، حمله دیر کشف می‌شود. Alerting راه‌اندازی کنید.

**۵. تمرکز فقط روی کد.**
محافظت شبکه‌ای هم لازم است — اما در Vercel بخش زیادی خودکار است.

### نکات مهم (Tips & Tricks)

- **Runbook** آماده داشته باشید.
- **کش لبه** بهترین سپر در برابر حمله است.
- **Alerting** روی نرخ خطا و ترافیک راه‌اندازی کنید.
- مسیرهای پرهزینه را **محدود** کنید.
- پس از حمله، **Post-mortem** بنویسید.

### خلاصه فصل

- DDoS در لایه‌های **L3/L4** و **L7** رخ می‌دهد.
- Vercel لایهٔ شبکه را **خودکار** محافظت می‌کند.
- **Attack Challenge Mode** ابزار فوری هنگام حمله است.
- **کش لبه** بار را از سرور برمی‌دارد.
- Runbook و Alerting از قبل آماده کنید.

---

<a id="فصل-۶"></a>
## فصل ۶: چک‌لیست امنیتی پروژه

### مقدمه

در این فصل، همهٔ موارد را در قالب یک چک‌لیست عملی جمع می‌کنیم تا قبل از هر انتشار بررسی کنید.

### چک‌لیست کامل

**۱. Secrets و پیکربندی**

- [ ] هیچ کلیدی در کد یا مخزن نیست
- [ ] `.env` و `.env.local` در `.gitignore` هستند
- [ ] `.env.example` وجود دارد (فقط نام‌ها)
- [ ] کلیدها در Vercel **Sensitive** علامت خورده‌اند
- [ ] کلیدهای تست و تولید جدا هستند
- [ ] Secret Scanning در GitHub فعال است
- [ ] برنامهٔ چرخش کلید وجود دارد

**۲. احراز هویت و دسترسی**

- [ ] رمزها با **bcrypt/argon2** هش می‌شوند
- [ ] کوکی‌ها `HttpOnly; Secure; SameSite` دارند
- [ ] JWT ها تاریخ انقضا دارند
- [ ] بررسی **مالکیت** در همهٔ مسیرهای داده (جلوگیری از IDOR)
- [ ] اصل **کمترین دسترسی** رعایت شده
- [ ] Rate Limiting روی لاگین/ثبت‌نام فعال است
- [ ] پیام‌های خطای احراز هویت **یکسان** هستند

**۳. ورودی و خروجی**

- [ ] همهٔ ورودی‌ها سمت **سرور** اعتبارسنجی می‌شوند
- [ ] کوئری‌های دیتابیس **پارامتری** هستند
- [ ] HTML کاربر با **DOMPurify** پاک‌سازی می‌شود
- [ ] CORS فقط دامنه‌های مجاز را می‌پذیرد
- [ ] محدودیت حجم و نوع فایل آپلودی اعمال می‌شود
- [ ] پیام‌های خطا اطلاعات حساس نمی‌دهند

**۴. هدرهای امنیتی**

```js
// next.config.js
module.exports = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          { key: 'Strict-Transport-Security', value: 'max-age=63072000; includeSubDomains; preload' },
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'X-Frame-Options', value: 'SAMEORIGIN' },
          { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
          { key: 'Permissions-Policy', value: 'camera=(), microphone=(), geolocation=()' },
          { key: 'Content-Security-Policy', value: "default-src 'self'; img-src 'self' data: https:; script-src 'self'" },
        ],
      },
    ];
  },
};
```

- [ ] **HSTS** فعال است
- [ ] **CSP** تنظیم شده
- [ ] `X-Content-Type-Options: nosniff`
- [ ] `X-Frame-Options` یا `frame-ancestors`
- [ ] `Referrer-Policy` تنظیم شده

**۵. Webhook و Cron**

- [ ] امضای **HMAC** تأیید می‌شود
- [ ] `bodyParser: false` برای امضا
- [ ] **Idempotency** برای جلوگیری از تکرار
- [ ] `CRON_SECRET` تعریف و بررسی می‌شود
- [ ] پاسخ سریع + صف برای پردازش سنگین
- [ ] Timeout در فراخوانی‌های بیرونی

**۶. Firewall و DDoS**

- [ ] قوانین Firewall برای مسیرهای حساس
- [ ] ربات‌های خوب مستثنا شده‌اند
- [ ] **System Bypass** برای health check
- [ ] Runbook حمله آماده است
- [ ] Alerting روی نرخ خطا فعال است

**۷. داده و ذخیره‌سازی**

- [ ] دادهٔ شخصی **کش نمی‌شود**
- [ ] دیتابیس فقط از شبکهٔ خصوصی/امن
- [ ] Connection Pooling فعال
- [ ] Backup دیتابیس تنظیم شده
- [ ] دادهٔ حساس رمزنگاری شده

**۸. لاگ و نظارت**

- [ ] لاگ‌ها **دادهٔ حساس** ندارند
- [ ] لاگ ساختاریافته (JSON)
- [ ] خطاها با جزئیات لاگ می‌شوند (نه به کاربر)
- [ ] هشدار برای رویدادهای امنیتی

**۹. وابستگی‌ها**

- [ ] `npm audit` اجرا و مشکلات رفع شده
- [ ] وابستگی‌ها به‌روز هستند
- [ ] Dependabot فعال است

```bash
npm audit
npm audit fix
```

**۱۰. استقرار**

- [ ] Deployment Protection برای Preview فعال
- [ ] شاخهٔ Production محافظت‌شده (Branch Protection)
- [ ] بازبینی کد اجباری برای PR
- [ ] محیط‌های Preview با دادهٔ جدا

### مثال عملی — بررسی سریع امنیت

```bash
#!/usr/bin/env bash
# security-check.sh

echo "=== بررسی Secret در کد ==="
grep -rn "sk_live\|sk_test\|password\s*=\s*['\"]" \
  --include="*.js" --include="*.ts" --include="*.jsx" --include="*.tsx" \
  . 2>/dev/null || echo "✓ چیزی پیدا نشد"

echo ""
echo "=== بررسی .gitignore ==="
grep -q "^\.env" .gitignore && echo "✓ .env در gitignore" || echo "✗ .env در gitignore نیست!"

echo ""
echo "=== بررسی وابستگی‌های آسیب‌پذیر ==="
npm audit --audit-level=high

echo ""
echo "=== بررسی هدرهای امنیتی ==="
curl -sI https://myapp.com | grep -iE "strict-transport|content-type-options|x-frame|referrer"
```

### نکات مهم (Tips & Tricks)

- این چک‌لیست را در **PR template** بگذارید.
- **قبل از هر انتشار Production** مرور کنید.
- چک‌لیست را **دوره‌ای به‌روز** کنید.
- از ابزارهای خودکار (Snyk، Dependabot) استفاده کنید.
- تیم را **آموزش** دهید — امنیت کار یک نفر نیست.

### خلاصه فصل

- چک‌لیست ۱۰ بخشی: **Secrets، Auth، ورودی، هدرها، Webhook، Firewall، داده، لاگ، وابستگی، استقرار**.
- هدرهای امنیتی را در `next.config.js` تنظیم کنید.
- `npm audit` و Dependabot را فعال کنید.
- چک‌لیست را در PR template بگذارید.
- امنیت یک **فرایند مستمر** است.

---

## منابع و مراجع

### مستندات رسمی
- Vercel Security: <https://vercel.com/docs/security>
- Vercel Firewall: <https://vercel.com/docs/security/vercel-firewall>
- Bot Management: <https://vercel.com/docs/security/bot-management>
- Rate Limiting: <https://vercel.com/docs/security/rate-limiting>
- Deployment Protection: <https://vercel.com/docs/security/deployment-protection>

### مفاهیم پیشنهادی برای مطالعه بیشتر
- OWASP Top 10: <https://owasp.org/www-project-top-ten/>
- Security Headers: <https://securityheaders.com>
- MDN Web Security: <https://developer.mozilla.org/docs/Web/Security>
- Mozilla Observatory: <https://observatory.mozilla.org>

### گام بعدی
- [فایل ۱۳ — پایش و مشاهده‌پذیری](./13-Observability.md)
- [فایل ۱۶ — عیب‌یابی خطاهای رایج](./16-Troubleshooting.md)
