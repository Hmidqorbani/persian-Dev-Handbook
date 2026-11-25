# ۰۶ — Middleware و Streaming

**مدت یادگیری**: ۳ تا ۴ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: مطالعه فایل‌های ۰۱ تا ۰۵، آشنایی با Next.js App Router

---

## فهرست محتوا

- [فصل ۱: Middleware چیست و کجا اجرا می‌شود؟](#فصل-۱)
- [فصل ۲: کاربردها — احراز هویت، A/B Testing، Redirect، Geo](#فصل-۲)
- [فصل ۳: Matcher و کنترل مسیرها](#فصل-۳)
- [فصل ۴: Streaming و React Suspense](#فصل-۴)
- [فصل ۵: مثال‌های عملی و اشکالات رایج](#فصل-۵)

---

<a id="فصل-۱"></a>
## فصل ۱: Middleware چیست و کجا اجرا می‌شود؟

### مقدمه

تصور کنید می‌خواهید قبل از اینکه کاربر به هر صفحه‌ای برسد، یک بررسی انجام دهید: آیا لاگین کرده؟ آیا باید به زبان دیگری هدایت شود؟ آیا نسخهٔ A یا B را ببیند؟ **Middleware** دقیقاً همین‌جاست — یک لایهٔ کد که **قبل از** رسیدن درخواست به صفحه اجرا می‌شود.

### مفاهیم اساسی

**۱. تعریف**
**Middleware** یک تابع است که قبل از کامل شدن درخواست اجرا می‌شود و می‌تواند:
- درخواست را **تغییر** دهد (اضافه کردن هدر، کوکی).
- **ریدایرکت** کند.
- **بازنویسی** کند (نمایش محتوای دیگر بدون تغییر URL).
- پاسخ را **مستقیماً** برگرداند (بدون رسیدن به صفحه).

**۲. محل قرارگیری فایل**
```
my-app/
├── middleware.ts        ← در ریشه پروژه (یا داخل src/)
├── app/
└── next.config.js
```

**۳. کجا اجرا می‌شود؟**
Middleware روی **Edge Runtime** اجرا می‌شود — یعنی نزدیک کاربر و بسیار سریع. این یعنی:
- Latency بسیار کم.
- محدودیت‌های Edge را دارد (بدون `fs`، بدون API های Node).

**۴. چه زمانی اجرا می‌شود؟**
- قبل از هر درخواست به مسیرهای مطابق `matcher`.
- برای **هر** درخواست: ناوبری، رفرش، Prefetch، فایل‌های استاتیک (اگر در matcher باشند).

**۵. ترتیب اجرا**
```
درخواست کاربر
   ↓
Middleware (اگر مطابق matcher باشد)
   ↓
Route Handler / صفحه (SSR)
   ↓
پاسخ
```

**۶. سه نوع پاسخ در Middleware**

| متد | کار |
|---|---|
| `NextResponse.next()` | ادامه دادن به مسیر عادی |
| `NextResponse.redirect(url)` | هدایت به آدرس جدید |
| `NextResponse.rewrite(url)` | نمایش محتوای دیگر با همان URL |
| `NextResponse.json(data)` | پاسخ JSON مستقیم |

### مثال‌های عملی

**مثال ۱ — ساده‌ترین Middleware**

```ts
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // فقط یک لاگ برای شروع
  console.log('درخواست به:', request.nextUrl.pathname);
  return NextResponse.next();
}

// مشخص کردن مسیرهایی که Middleware روی آن‌ها اجرا شود
export const config = {
  matcher: '/dashboard/:path*',
};
```

**مثال ۲ — اضافه کردن هدر سفارشی**

```ts
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const response = NextResponse.next();

  // اضافه کردن هدر امنیتی به همهٔ پاسخ‌ها
  response.headers.set('X-Frame-Options', 'DENY');
  response.headers.set('X-Content-Type-Options', 'nosniff');

  // پاس دادن اطلاعات به صفحه (از طریق هدر سفارشی)
  response.headers.set('x-user-path', request.nextUrl.pathname);

  return response;
}

export const config = { matcher: '/(.*)' };
```

**مثال ۳ — خواندن و تنظیم کوکی**

```ts
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const response = NextResponse.next();

  // خواندن کوکی
  const theme = request.cookies.get('theme')?.value ?? 'light';

  // تنظیم کوکی (اگر وجود نداشت)
  if (!request.cookies.has('theme')) {
    response.cookies.set('theme', 'light', {
      path: '/',
      maxAge: 60 * 60 * 24 * 365,   // یک سال
      sameSite: 'lax',
      httpOnly: false,               // اگر کلاینت نیاز دارد false
    });
  }

  return response;
}
```

### مقایسه و الگوهای مختلف

**Middleware در برابر سایر لایه‌ها**

| معیار | Middleware | Route Handler | Server Component | Client |
|---|---|---|---|---|
| زمان اجرا | قبل از درخواست | در درخواست | در رندر | در مرورگر |
| Runtime | Edge | Node/Edge | Node/Edge | مرورگر |
| دسترسی به دیتابیس | ❌ | ✅ | ✅ | ❌ |
| سرعت | بسیار بالا | بالا | بالا | متوسط |
| مناسب برای | auth، redirect، هدر | API، منطق | داده، محتوا | تعامل |

**`redirect` در برابر `rewrite`**

| معیار | redirect | rewrite |
|---|---|---|
| URL در مرورگر | تغییر می‌کند | تغییر نمی‌کند |
| مناسب برای | انتقال دائمی، login | A/B، multi-tenant |
| SEO | ممکن است تأثیر بگذارد | بی‌اثر |
| سرعت | یک درخواست اضافه | بدون درخواست اضافه |

### اشکال شایع

**۱. اجرای Middleware روی همهٔ مسیرها بدون matcher.**
روی هر درخواست (حتی فایل‌های استاتیک) اجرا می‌شود و سرعت را کم می‌کند. همیشه `matcher` بگذارید.

**۲. انتظار دسترسی به دیتابیس در Middleware.**
Middleware روی Edge است؛ اتصال TCP معمولاً ممکن نیست.

**۳. منطق سنگین در Middleware.**
Middleware باید **سبک و سریع** باشد. منطق سنگین را به Route Handler بسپارید.

**۴. فراموش کردن `return NextResponse.next()`.**
اگر چیزی برنگردانید، درخواست ممکن است معلق بماند.

**۵. خواندن بدنهٔ درخواست (body).**
در Middleware به‌طور پیش‌فرض دسترسی به بدنهٔ کامل توصیه نمی‌شود و می‌تواند مشکل‌ساز باشد.

### نکات مهم (Tips & Tricks)

- **همیشه `matcher` بگذارید** — این مهم‌ترین بهینه‌سازی است.
- Middleware را **سبک** نگه دارید؛ فقط تصمیم‌های سریع.
- برای احراز هویت، فقط **وجود و اعتبار توکن** را بررسی کنید، نه کوئری دیتابیس.
- از Middleware برای **هدرهای امنیتی** استفاده کنید.
- Middleware را برای **A/B Testing** و **جغرافیا** بکار ببرید.

### خلاصه فصل

- **Middleware** قبل از رسیدن درخواست به صفحه اجرا می‌شود.
- روی **Edge Runtime** و بسیار سریع است.
- سه پاسخ اصلی: `next()`، `redirect()`، `rewrite()`.
- همیشه با `matcher` محدودش کنید.
- سبک نگه دارید؛ منطق سنگین جای دیگری دارد.

---

<a id="فصل-۲"></a>
## فصل ۲: کاربردها — احراز هویت، A/B Testing، Redirect، Geo

### مقدمه

حالا که با مفهوم آشنا شدیم، ببینیم Middleware در دنیای واقعی چه کارهایی انجام می‌دهد. این فصل، پرکاربردترین الگوها را با کد کامل نشان می‌دهد.

### مفاهیم اساسی

**۱. احراز هویت (Authentication)**
رایج‌ترین کاربرد Middleware: بررسی اینکه کاربر لاگین کرده یا نه، قبل از دسترسی به صفحات محافظت‌شده.

**۲. A/B Testing**
نمایش نسخه‌های مختلف یک صفحه به کاربران مختلف و اندازه‌گیری نتیجه.

**۳. Redirect هوشمند**
هدایت کاربر بر اساس شرایط (کشور، دستگاه، زبان، وضعیت لاگین).

**۴. Geo-targeting**
استفاده از اطلاعات جغرافیایی کاربر برای شخصی‌سازی محتوا یا ریدایرکت.

**۵. مدیریت زبان (i18n)**
هدایت کاربر به نسخهٔ زبان مناسب بر اساس هدر `Accept-Language` یا کوکی.

**۶. Rate Limiting سبک**
محدود کردن تعداد درخواست‌ها از یک IP (نسخهٔ ساده در Middleware؛ نسخهٔ کامل در Firewall).

### مثال‌های عملی

**مثال ۱ — محافظت از مسیر با بررسی توکن**

```ts
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

const PROTECTED_PATHS = ['/dashboard', '/settings', '/admin'];

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;

  // اگر مسیر محافظت‌شده است، توکن را بررسی کن
  const isProtected = PROTECTED_PATHS.some((p) => pathname.startsWith(p));

  if (isProtected) {
    const token = request.cookies.get('auth_token')?.value;

    if (!token) {
      // ذخیره مسیر فعلی برای بازگشت بعد از لاگین
      const loginUrl = new URL('/login', request.url);
      loginUrl.searchParams.set('next', pathname);
      return NextResponse.redirect(loginUrl);
    }

    // (اختیاری) بررسی اعتبار توکن — فقط بررسی سطحی، نه کوئری دیتابیس
    try {
      // در عمل: تأیید JWT (بدون verify کامل به دلیل محدودیت Edge)
      const payload = JSON.parse(
        atob(token.split('.')[1] ?? '')
      );
      if (payload.exp && payload.exp * 1000 < Date.now()) {
        return NextResponse.redirect(new URL('/login', request.url));
      }
    } catch {
      return NextResponse.redirect(new URL('/login', request.url));
    }
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/settings/:path*', '/admin/:path*'],
};
```

**مثال ۲ — A/B Testing با کوکی**

```ts
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // اگر کاربر قبلاً نسخه گرفته، همان را نگه دار
  let variant = request.cookies.get('ab_variant')?.value;

  const response = NextResponse.next();

  if (!variant) {
    // انتخاب تصادفی نسخه
    variant = Math.random() < 0.5 ? 'A' : 'B';
    response.cookies.set('ab_variant', variant, {
      path: '/',
      maxAge: 60 * 60 * 24 * 30,   // ۳۰ روز
    });
  }

  // بازنویسی مسیر به نسخهٔ انتخاب‌شده (URL عوض نمی‌شود)
  if (request.nextUrl.pathname === '/') {
    return NextResponse.rewrite(
      new URL(`/home-${variant.toLowerCase()}`, request.url),
      { headers: { 'x-ab-variant': variant } }
    );
  }

  return response;
}

export const config = { matcher: '/' };
```

**مثال ۳ — ریدایرکت بر اساس جغرافیا**

```ts
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const country = request.headers.get('x-vercel-ip-country') ?? 'IR';
  const { pathname } = request.nextUrl;

  // اگر کاربر ایرانی است و در مسیر بین‌المللی است → هدایت به نسخه فارسی
  if (country === 'IR' && !pathname.startsWith('/fa') && !pathname.startsWith('/api')) {
    return NextResponse.redirect(new URL(`/fa${pathname}`, request.url));
  }

  // اضافه کردن کشور به هدر برای استفاده در صفحه
  const response = NextResponse.next();
  response.headers.set('x-user-country', country);
  return response;
}

export const config = { matcher: '/((?!_next|api).*)' };
```

**مثال ۴ — مدیریت زبان (i18n) ساده**

```ts
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

const SUPPORTED = ['fa', 'en', 'ar'];
const DEFAULT = 'fa';

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;

  // اگر مسیر از قبل زبان دارد، رد شو
  const hasLocale = SUPPORTED.some(
    (l) => pathname === `/${l}` || pathname.startsWith(`/${l}/`)
  );
  if (hasLocale) return NextResponse.next();

  // تعیین زبان از کوکی یا هدر
  const cookieLang = request.cookies.get('lang')?.value;
  const acceptLang = request.headers.get('accept-language') ?? '';
  const detected =
    cookieLang ??
    SUPPORTED.find((l) => acceptLang.toLowerCase().includes(l)) ??
    DEFAULT;

  return NextResponse.redirect(
    new URL(`/${detected}${pathname}`, request.url)
  );
}

export const config = {
  matcher: ['/((?!_next|api|favicon.ico|.*\\..*).*)'],
};
```

**مثال ۵ — Rate Limiting سبک**

```ts
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

// ⚠️ در حافظه است و برای Production مناسب نیست؛ برای نمونه است
const hits = new Map<string, { count: number; reset: number }>();
const LIMIT = 20;
const WINDOW = 60_000;   // یک دقیقه

export function middleware(request: NextRequest) {
  const ip = request.headers.get('x-forwarded-for')?.split(',')[0] ?? 'unknown';
  const now = Date.now();

  const entry = hits.get(ip);
  if (!entry || now > entry.reset) {
    hits.set(ip, { count: 1, reset: now + WINDOW });
    return NextResponse.next();
  }

  entry.count++;
  if (entry.count > LIMIT) {
    return NextResponse.json(
      { error: 'تعداد درخواست‌ها زیاد است' },
      { status: 429, headers: { 'Retry-After': '60' } }
    );
  }

  return NextResponse.next();
}

export const config = { matcher: '/api/:path*' };
```

> برای Rate Limiting واقعی، از **Vercel Firewall** استفاده کنید (فایل ۱۲).

### مقایسه و الگوهای مختلف

**Middleware در برابر سایر روش‌ها برای احراز هویت**

| روش | سرعت | امنیت | مناسب برای |
|---|---|---|---|
| **Middleware** | بسیار بالا | سطحی (بررسی وجود) | محافظت اولیه، UX |
| **Server Component** | بالا | بالا | بررسی دقیق با دیتابیس |
| **Route Handler** | بالا | بالا | API محافظت‌شده |
| **Firewall** | بسیار بالا | بالا | Rate Limit، Bot |

**سه الگوی Geo**

| الگو | توضیح |
|---|---|
| **Redirect** | هدایت به نسخهٔ منطقه‌ای |
| **Rewrite** | نمایش محتوای منطقه‌ای با همان URL |
| **Header** | پاس دادن کشور به صفحه برای تصمیم‌گیری |

### اشکال شایع

**۱. احراز هویت کامل در Middleware.**
Middleware نمی‌تواند کوئری دیتابیس بزند. فقط **بررسی وجود/اعتبار سطحی** توکن را انجام دهید و بررسی دقیق را در صفحه بگذارید.

**۲. اعتماد به هدر `x-forwarded-for` به‌تنهایی.**
می‌تواند جعل شود. برای Rate Limiting جدی از Firewall استفاده کنید.

**۳. حلقهٔ ریدایرکت (Redirect Loop).**
اگر شرط ریدایرکت اشتباه باشد، کاربر در حلقه می‌افتد. همیشه مسیرهای استثنا را در نظر بگیرید.

**۴. ذخیرهٔ حالت در حافظهٔ Middleware.**
مثل مثال Rate Limiting بالا — در Production قابل اعتماد نیست.

**۵. نادیده گرفتن `_next` و `api` در matcher.**
باعث اجرای بی‌مورد روی فایل‌های داخلی و API می‌شود.

### نکات مهم (Tips & Tricks)

- برای احراز هویت، **دو لایه** داشته باشید: Middleware (سریع) + صفحه (دقیق).
- همیشه مسیرهای استثنا (`_next`, `api`, فایل‌های استاتیک) را در matcher حذف کنید.
- برای A/B Testing، نسخهٔ کاربر را در **کوکی** نگه دارید تا تجربه ثابت بماند.
- از هدر `x-vercel-ip-country` برای Geo استفاده کنید.
- برای Rate Limiting واقعی، به سراغ **Vercel Firewall** بروید.

### خلاصه فصل

- کاربردهای اصلی: **احراز هویت، A/B Testing، Redirect، Geo، i18n**.
- Middleware برای بررسی‌های **سریع و سبک** است.
- بررسی دقیق (دیتابیس) را در Server Component/Route Handler انجام دهید.
- همیشه مسیرهای استثنا را در matcher بگذارید.
- Rate Limiting جدی = Firewall.

---

<a id="فصل-۳"></a>
## فصل ۳: Matcher و کنترل مسیرها

### مقدمه

`matcher` مشخص می‌کند Middleware روی کدام مسیرها اجرا شود. اگر اشتباه تنظیم شود، یا Middleware بی‌مورد اجرا می‌شود (کند) یا روی مسیرهای مهم اجرا نمی‌شود (نقص امنیتی). در این فصل، matcher را کامل یاد می‌گیریم.

### مفاهیم اساسی

**۱. `config.matcher`**
آرایه‌ای از الگوهای مسیر. Middleware فقط روی مسیرهای مطابق اجرا می‌شود.

```ts
export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*'],
};
```

**۲. الگوهای مسیر**

| الگو | معنی | مثال تطبیق |
|---|---|---|
| `/about` | فقط این مسیر | `/about` |
| `/about/:path` | یک بخش | `/about/team` |
| `/about/:path*` | صفر یا چند بخش | `/about`, `/about/a/b` |
| `/(.*)` | همه چیز | همه مسیرها |
| `/:path*` | همه چیز | همه مسیرها |

**۳. الگوی حذف (Negative Lookahead)**
برای اینکه Middleware روی مسیرهای خاصی اجرا **نشود**:
```ts
matcher: ['/((?!_next|api|static|.*\\..*).*)']
```
معنی: همهٔ مسیرها **به‌جز** `_next`، `api`، `static` و هر فایلی که نقطه دارد (مثل `logo.png`).

**۴. Regex در matcher**
می‌توانید از Regex کامل استفاده کنید:
```ts
matcher: ['/blog/(\\d+)']   // فقط مسیرهایی که عدد دارند
```

**۵. Middleware شرطی درون کد**
علاوه بر matcher، می‌توانید درون کد هم شرط بگذارید:
```ts
if (request.nextUrl.pathname.startsWith('/admin')) { ... }
```

### مثال‌های عملی

**مثال ۱ — matcher استاندارد و بهینه**

```ts
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  return NextResponse.next();
}

export const config = {
  // همه مسیرها به‌جز فایل‌های داخلی و استاتیک
  matcher: [
    /*
     * تطبیق با همهٔ مسیرها به‌جز:
     * - _next/static (فایل‌های استاتیک Next.js)
     * - _next/image (بهینه‌سازی تصویر)
     * - favicon.ico
     * - فایل‌های عمومی با پسوند
     */
    '/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)',
  ],
};
```

**مثال ۲ — چند matcher برای مسیرهای مختلف**

```ts
export const config = {
  matcher: [
    '/dashboard/:path*',
    '/admin/:path*',
    '/api/private/:path*',
    '/blog/(\\d+)',            // فقط مقالات با شناسه عددی
  ],
};
```

**مثال ۳ — شرط‌گذاری درون کد (علاوه بر matcher)**

```ts
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;

  // نادیده گرفتن درخواست‌های Prefetch
  if (request.headers.get('x-middleware-prefetch')) {
    return NextResponse.next();
  }

  // فقط API ها را بررسی کن
  if (pathname.startsWith('/api/')) {
    const apiKey = request.headers.get('x-api-key');
    if (!apiKey || apiKey !== process.env.API_KEY) {
      return NextResponse.json({ error: 'کلید نامعتبر' }, { status: 401 });
    }
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/api/:path*', '/dashboard/:path*'],
};
```

### مقایسه و الگوهای مختلف

**سه استراتژی matcher**

| استراتژی | الگو | مزیت | عیب |
|---|---|---|---|
| **همه چیز** | `'/(.*)'` | ساده | کند (اجرا روی استاتیک) |
| **حذف داخلی‌ها** | `'/((?!_next|api|.*\\..*).*)'` | ✅ بهینه | کمی پیچیده |
| **مسیرهای مشخص** | آرایه مسیرها | ✅ سریع‌ترین | نیاز به نگهداری |

**کجا matcher بگذاریم و کجا شرط درون کد؟**

| سناریو | راه پیشنهادی |
|---|---|
| محافظت از چند مسیر مشخص | matcher |
| نادیده گرفتن فایل‌های استاتیک | matcher (Regex) |
| شرط‌های پیچیده (هدر، کوکی) | کد |
| نادیده گرفتن Prefetch | کد |

### اشکال شایع

**۱. `matcher: '/(.*)'` بدون حذف استاتیک.**
Middleware روی هر تصویر و فایل CSS هم اجرا می‌شود → کندی و هزینهٔ اضافه.

**۲. اشتباه در Regex حذف.**
اگر الگو اشتباه باشد، ممکن است مسیرهای مهم هم حذف شوند.

**۳. فراموش کردن `:path*`.**
`/dashboard` فقط همان مسیر را می‌گیرد، نه `/dashboard/settings`.

**۴. اجرای Middleware روی درخواست‌های Prefetch.**
می‌تواند هزینه و بار اضافه ایجاد کند. با هدر `x-middleware-prefetch` نادیده بگیرید.

**۵. matcher روی مسیرهای API حساس فراموش شود.**
باعث دسترسی بدون احراز هویت می‌شود.

### نکات مهم (Tips & Tricks)

- الگوی **حذف داخلی‌ها** را به‌عنوان پیش‌فرض به خاطر بسپارید.
- مسیرهای API محافظت‌شده را **حتماً** در matcher بیاورید.
- از Regex فقط در صورت نیاز استفاده کنید — خوانایی مهم است.
- برای Prefetch، `x-middleware-prefetch` را بررسی کنید.
- matcher را با کامنت توضیح دهید تا تیم بفهمد.

### خلاصه فصل

- `config.matcher` تعیین می‌کند Middleware کجا اجرا شود.
- الگوهای مهم: `/:path*`، `/(.*)`، Negative Lookahead.
- همیشه مسیرهای داخلی و استاتیک را حذف کنید.
- می‌توانید شرط‌های پیچیده را درون کد بگذارید.
- matcher بهینه = سرعت و امنیت بهتر.

---

<a id="فصل-۴"></a>
## فصل ۴: Streaming و React Suspense

### مقدمه

تصور کنید صفحه‌ای دارید که بخشی از آن سریع آماده می‌شود و بخشی کند است (مثلاً کوئری سنگین دیتابیس). بدون Streaming، کاربر باید منتظر **کل** صفحه بماند. با **Streaming**، بخش‌های آماده فوراً نمایش داده می‌شوند و بخش‌های کند بعداً می‌آیند.

### مفاهیم اساسی

**۱. Streaming چیست؟**
ارسال تدریجی HTML به مرورگر. به‌جای فرستادن یک‌بارهٔ کل صفحه، بخش‌ها به‌محض آماده شدن فرستاده می‌شوند.

> **تشبیه**: به‌جای اینکه منتظر بمانید کل غذا آماده شود و بعد سرو شود، پیش‌غذا را فوراً می‌آورند و غذای اصلی را بعداً.

**۲. چرا مهم است؟**
- **Time to First Byte (TTFB)** بهتر.
- کاربر سریع‌تر چیزی می‌بیند (احساس سرعت).
- بخش‌های کند صفحه را مسدود نمی‌کنند.

**۳. ابزارهای Streaming در Next.js**

| ابزار | کاربرد |
|---|---|
| `<Suspense>` | جدا کردن بخش‌های کند |
| `loading.tsx` | حالت بارگذاری خودکار برای یک مسیر |
| `ReadableStream` | استریم در Route Handler |
| `use` hook | خواندن Promise در Client Component |

**۴. `loading.tsx`**
یک فایل ویژه که به‌طور خودکار بخش بارگذاری یک مسیر را می‌سازد. Next.js به‌طور خودکار محتوای صفحه را در `<Suspense>` می‌پیچد.

**۵. Progressive Rendering**
HTML ابتدا با بخش‌های آماده فرستاده می‌شود، سپس بخش‌های کند "جای‌گذاری" و بعداً پر می‌شوند.

### مثال‌های عملی

**مثال ۱ — Streaming با Suspense**

```tsx
// app/page.tsx
import { Suspense } from 'react';

// بخش کند
async function SlowSection() {
  const data = await fetch('https://api.example.com/slow', {
    cache: 'no-store',
  }).then((r) => r.json());

  return (
    <section>
      <h2>دادهٔ کند</h2>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </section>
  );
}

// بخش سریع
function FastSection() {
  return (
    <section>
      <h1>خوش آمدید!</h1>
      <p>این بخش فوراً نمایش داده می‌شود.</p>
    </section>
  );
}

export default function Page() {
  return (
    <main>
      <FastSection />

      {/* بخش کند بعداً می‌آید؛ کاربر منتظر نمی‌ماند */}
      <Suspense fallback={<p>در حال بارگذاری بخش کند...</p>}>
        <SlowSection />
      </Suspense>
    </main>
  );
}
```

**مثال ۲ — استفاده از `loading.tsx`**

```tsx
// app/dashboard/loading.tsx
// Next.js به‌طور خودکار این را به‌عنوان fallback نمایش می‌دهد
export default function Loading() {
  return (
    <div className="skeleton">
      <div className="skeleton-header" />
      <div className="skeleton-line" />
      <div className="skeleton-line" />
    </div>
  );
}
```

```tsx
// app/dashboard/page.tsx — بدون نیاز به Suspense دستی
async function getDashboardData() {
  const res = await fetch('https://api.example.com/dashboard', {
    cache: 'no-store',
  });
  return res.json();
}

export default async function DashboardPage() {
  const data = await getDashboardData();
  return <div>{/* محتوای داشبورد */}</div>;
}
```

**مثال ۳ — Streaming در Route Handler**

```ts
// app/api/stream/route.ts
export async function GET() {
  const encoder = new TextEncoder();

  const stream = new ReadableStream({
    async start(controller) {
      const steps = ['دریافت درخواست', 'پردازش', 'آماده‌سازی', 'پایان'];
      for (const step of steps) {
        controller.enqueue(encoder.encode(`${step}\n`));
        await new Promise((r) => setTimeout(r, 300));
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

**مثال ۴ — استریم پاسخ AI (Server-Sent Events)**

```ts
// app/api/chat/route.ts
export const runtime = 'edge';

export async function POST(request: Request) {
  const { prompt } = await request.json();

  const encoder = new TextEncoder();
  const stream = new ReadableStream({
    async start(controller) {
      // در عمل: اتصال به مدل AI و استریم پاسخ
      const words = (prompt ?? '').split(' ');
      for (const word of words) {
        controller.enqueue(
          encoder.encode(`data: ${JSON.stringify({ token: word })}\n\n`)
        );
        await new Promise((r) => setTimeout(r, 150));
      }
      controller.enqueue(encoder.encode('data: [DONE]\n\n'));
      controller.close();
    },
  });

  return new Response(stream, {
    headers: {
      'content-type': 'text/event-stream',
      'cache-control': 'no-cache',
      connection: 'keep-alive',
    },
  });
}
```

### مقایسه و الگوهای مختلف

**بدون Streaming در برابر با Streaming**

| معیار | بدون Streaming | با Streaming |
|---|---|---|
| زمان تا اولین محتوا | کند (انتظار کل صفحه) | سریع |
| TTFB | بالا | پایین |
| تجربهٔ کاربر | صفحهٔ سفید طولانی | محتوای تدریجی |
| پیچیدگی | ساده | کمی بیشتر |
| مناسب برای | صفحات سبک | صفحات با بخش کند |

**سه سطح Streaming**

| سطح | ابزار | کنترل |
|---|---|---|
| خودکار | `loading.tsx` | کم |
| دستی | `<Suspense>` | زیاد |
| خام | `ReadableStream` | کامل |

### اشکال شایع

**۱. قرار دادن کل صفحه در یک Suspense.**
مزیت را از دست می‌دهید؛ باید بخش‌های کند را **جدا** کنید.

**۲. استفاده از `loading.tsx` برای صفحات بسیار سبک.**
می‌تواند باعث پرش (flicker) شود. برای صفحات سبک لازم نیست.

**۳. فراموش کردن `cache-control` در استریم.**
پاسخ ممکن است به اشتباه کش شود. برای استریم `no-store` یا `no-cache` بگذارید.

**۴. عدم مدیریت خطا در استریم.**
اگر وسط استریم خطا رخ دهد، باید `controller.error()` را صدا بزنید.

**۵. استفاده از Streaming برای SEO-محتوای اصلی.**
موتورهای جست‌وجو معمولاً HTML نهایی را می‌خوانند؛ محتوای مهم را در بخش‌های سریع بگذارید.

### نکات مهم (Tips & Tricks)

- بخش‌های **کند** را در `<Suspense>` بگذارید، نه کل صفحه.
- از `loading.tsx` برای اسکلتون (Skeleton) زیبا استفاده کنید.
- برای پاسخ‌های AI، از **SSE** (`text/event-stream`) استفاده کنید.
- استریم را با **Edge Runtime** ترکیب کنید تا TTFB کم شود.
- برای محتوای مهم SEO، از استریم پرهیز کنید یا آن را در بخش سریع بگذارید.

### خلاصه فصل

- **Streaming** HTML را تدریجی می‌فرستد و تجربهٔ کاربری را بهتر می‌کند.
- ابزارها: **`<Suspense>`**، **`loading.tsx`**، **`ReadableStream`**.
- بخش‌های کند را جدا کنید، نه کل صفحه.
- برای AI و پاسخ‌های طولانی، **SSE** بهترین انتخاب است.
- Streaming را با Edge ترکیب کنید تا TTFB کم شود.

---

<a id="فصل-۵"></a>
## فصل ۵: مثال‌های عملی و اشکالات رایج

### مقدمه

در این فصل، Middleware و Streaming را در یک سناریوی واقعی ترکیب می‌کنیم: یک اپلیکیشن با احراز هویت، Geo-targeting و داشبورد استریمی.

### مثال ۱ — Middleware کامل برای یک اپ SaaS

```ts
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

const PUBLIC_PATHS = ['/', '/login', '/signup', '/pricing', '/blog'];
const AUTH_PATHS = ['/login', '/signup'];

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  const token = request.cookies.get('session')?.value;
  const country = request.headers.get('x-vercel-ip-country') ?? 'IR';

  // ۱. اگر کاربر لاگین کرده و در صفحهٔ auth است → داشبورد
  if (token && AUTH_PATHS.some((p) => pathname.startsWith(p))) {
    return NextResponse.redirect(new URL('/dashboard', request.url));
  }

  // ۲. اگر مسیر عمومی است → اجازه بده
  const isPublic = PUBLIC_PATHS.some(
    (p) => pathname === p || pathname.startsWith(`${p}/`)
  );
  if (isPublic) {
    const response = NextResponse.next();
    response.headers.set('x-user-country', country);
    return response;
  }

  // ۳. مسیر محافظت‌شده و بدون توکن → لاگین
  if (!token) {
    const loginUrl = new URL('/login', request.url);
    loginUrl.searchParams.set('next', pathname);
    return NextResponse.redirect(loginUrl);
  }

  // ۴. اضافه کردن هدرهای امنیتی
  const response = NextResponse.next();
  response.headers.set('X-Frame-Options', 'SAMEORIGIN');
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
  response.headers.set('x-user-country', country);

  return response;
}

export const config = {
  matcher: [
    '/((?!_next/static|_next/image|favicon.ico|api|.*\\.(?:svg|png|jpg|jpeg|gif|webp|css|js)$).*)',
  ],
};
```

### مثال ۲ — داشبورد استریمی

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react';

// بخش ۱: آمار سریع
function StatsCard() {
  return (
    <div className="card">
      <h3>بازدید امروز</h3>
      <p>۱٬۲۳۴</p>
    </div>
  );
}

// بخش ۲: نمودار کند (از دیتابیس)
async function SlowChart() {
  const data = await fetch('https://api.example.com/chart', {
    cache: 'no-store',
  }).then((r) => r.json());
  return <div className="chart">{/* نمودار */}</div>;
}

// بخش ۳: جدول فعالیت (متوسط)
async function ActivityTable() {
  const rows = await fetch('https://api.example.com/activity', {
    next: { revalidate: 30 },
  }).then((r) => r.json());
  return <table>{/* ردیف‌ها */}</table>;
}

function CardSkeleton() {
  return <div className="card skeleton">در حال بارگذاری...</div>;
}

export default function DashboardPage() {
  return (
    <div className="dashboard">
      {/* فوری */}
      <StatsCard />

      {/* کند — با اسکلتون */}
      <Suspense fallback={<CardSkeleton />}>
        <SlowChart />
      </Suspense>

      {/* متوسط — با اسکلتون */}
      <Suspense fallback={<CardSkeleton />}>
        <ActivityTable />
      </Suspense>
    </div>
  );
}
```

### مثال ۳ — API استریمی برای گزارش

```ts
// app/api/report/route.ts
export const runtime = 'edge';

export async function GET() {
  const encoder = new TextEncoder();

  const stream = new ReadableStream({
    async start(controller) {
      try {
        controller.enqueue(encoder.encode('شروع گزارش...\n'));

        for (let i = 1; i <= 5; i++) {
          // شبیه‌سازی پردازش هر بخش
          await new Promise((r) => setTimeout(r, 400));
          controller.enqueue(encoder.encode(`بخش ${i} پردازش شد\n`));
        }

        controller.enqueue(encoder.encode('گزارش کامل شد ✅\n'));
        controller.close();
      } catch (err) {
        controller.error(err);   // مدیریت خطا در استریم
      }
    },
  });

  return new Response(stream, {
    headers: {
      'content-type': 'text/plain; charset=utf-8',
      'cache-control': 'no-store',
      'x-accel-buffering': 'no',
    },
  });
}
```

### اشکالات رایج و راه‌حل

**۱. حلقهٔ ریدایرکت در Middleware**

```ts
// ❌ اشتباه: /login هم ریدایرکت می‌شود
if (!token) return NextResponse.redirect(new URL('/login', request.url));

// ✅ درست: مسیر /login را مستثنا کن
if (!token && !pathname.startsWith('/login')) {
  return NextResponse.redirect(new URL('/login', request.url));
}
```

**۲. Middleware روی فایل‌های استاتیک**

```ts
// ❌ کند: روی همه چیز اجرا می‌شود
export const config = { matcher: '/(.*)' };

// ✅ سریع: داخلی‌ها حذف شده‌اند
export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico|.*\\..*).*)'],
};
```

**۳. Suspense در سطح کل صفحه**

```tsx
// ❌ اشتباه: کل صفحه در یک Suspense
<Suspense fallback={<FullPageLoader />}>
  <FastPart />
  <SlowPart />
</Suspense>

// ✅ درست: فقط بخش کند
<FastPart />
<Suspense fallback={<Skeleton />}>
  <SlowPart />
</Suspense>
```

**۴. کش شدن پاسخ استریمی**

```ts
// ❌ اشتباه: پاسخ استریمی کش می‌شود
return new Response(stream);

// ✅ درست
return new Response(stream, {
  headers: { 'cache-control': 'no-store' },
});
```

**۵. نبود مدیریت خطا در استریم**

```ts
// ✅ همیشه خطا را مدیریت کنید
const stream = new ReadableStream({
  async start(controller) {
    try {
      // ...
    } catch (err) {
      controller.error(err);
    }
  },
});
```

### نکات مهم (Tips & Tricks)

- **Middleware را سبک نگه دارید** و منطق سنگین را به صفحه منتقل کنید.
- در Middleware همیشه **مسیرهای عمومی و استثنا** را مشخص کنید.
- برای احراز هویت **دو لایه‌ای** (Middleware + صفحه) عمل کنید.
- در Streaming، **بخش‌های کند** را جدا کنید.
- برای استریم، همیشه `cache-control: no-store` بگذارید.
- خطاها را در استریم با `controller.error()` مدیریت کنید.

### خلاصه فصل

- Middleware و Streaming می‌توانند با هم یک تجربهٔ سریع بسازند.
- الگوی حرفه‌ای: **Middleware سبک** + **Streaming برای بخش‌های کند**.
- حلقهٔ ریدایرکت، matcher سنگین و Suspense اشتباه، رایج‌ترین اشکالات‌اند.
- همیشه مسیرهای استثنا و کش را درست تنظیم کنید.
- خطا در استریم را حتماً مدیریت کنید.

---

## منابع و مراجع

### مستندات رسمی
- Next.js Middleware: <https://nextjs.org/docs/app/building-your-application/routing/middleware>
- Next.js Streaming: <https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming>
- Vercel Middleware: <https://vercel.com/docs/functions/edge-middleware>
- React Suspense: <https://react.dev/reference/react/Suspense>
- Server-Sent Events (MDN): <https://developer.mozilla.org/docs/Web/API/Server-sent_events>

### مفاهیم پیشنهادی برای مطالعه بیشتر
- Vercel Edge Middleware Limits: <https://vercel.com/docs/functions/edge-middleware/limitations>
- ReadableStream (MDN): <https://developer.mozilla.org/docs/Web/API/ReadableStream>
- Progressive Rendering: <https://web.dev/articles/progressive-rendering>

### گام بعدی
- [فایل ۰۷ — متغیرهای محیطی، دامنه، DNS و SSL](./07-Environment-Variables-and-Domains.md)
- [فایل ۱۲ — امنیت](./12-Security.md)
