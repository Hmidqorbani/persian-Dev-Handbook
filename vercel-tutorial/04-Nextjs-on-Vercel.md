# ۰۴ — Next.js روی Vercel

**مدت یادگیری**: ۴ تا ۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: مطالعه فایل‌های ۰۱ تا ۰۳، آشنایی مقدماتی با React و JSX

---

## فهرست محتوا

- [فصل ۱: چرا Next.js و چرا Vercel؟](#فصل-۱)
- [فصل ۲: ساختار پروژه — App Router و Pages Router](#فصل-۲)
- [فصل ۳: مسیریابی، Layout و کامپوننت‌های Server/Client](#فصل-۳)
- [فصل ۴: Data Fetching — SSR، SSG، ISR](#فصل-۴)
- [فصل ۵: API Routes و Route Handlers](#فصل-۵)
- [فصل ۶: بهینه‌سازی تصویر و فونت در Next.js](#فصل-۶)

---

<a id="فصل-۱"></a>
## فصل ۱: چرا Next.js و چرا Vercel؟

### مقدمه

اگر با React کار کرده‌اید، می‌دانید که React فقط «رابط کاربری» می‌سازد و دربارهٔ مسیریابی، SEO، تصاویر و بارگذاری داده چیزی نمی‌گوید. **Next.js** این شکاف‌ها را پر می‌کند. و چون سازندهٔ Next.js همان Vercel است، این دو مثل یک جفت دستکش با هم کار می‌کنند. در این فصل، دلیل این هم‌افزایی را می‌فهمیم.

### مفاهیم اساسی

**۱. Next.js چیست؟**
یک **فریم‌ورک React** برای ساخت اپلیکیشن‌های وب کامل (Full-Stack). امکاناتی که به React اضافه می‌کند:

| قابلیت | فایده |
|---|---|
| **مسیریابی فایل‌محور** | هر فایل = یک مسیر (بدون Router دستی) |
| **رندر سمت سرور (SSR)** | SEO بهتر، بار اول سریع‌تر |
| **تولید سایت استاتیک (SSG)** | سرعت بالا، هزینه کم |
| **ISR** | ترکیب استاتیک + به‌روزرسانی |
| **API Routes** | بک‌اند در همان پروژه |
| **بهینه‌سازی تصویر** | تصاویر خودکار فشرده و ریسپانسیو |
| **بهینه‌سازی فونت** | حذف درخواست‌های اضافه |
| **Server Components** | رندر روی سرور، ارسال JavaScript کمتر |

**۲. چرا Next.js + Vercel؟**
چون Vercel سازندهٔ Next.js است، **هر قابلیت جدید Next.js از روز اول روی Vercel بهینه کار می‌کند**. مثال‌ها:
- **ISR** و **PPR** روی Vercel بومی هستند.
- **Image Optimization** روی Edge اجرا می‌شود.
- **Middleware** و **Edge Functions** یکپارچه‌اند.
- **Fluid Compute** برای توابع Next.js بهینه شده است.

**۳. دو مدل رندر در Next.js**
- **Server Components**: روی سرور رندر می‌شوند؛ JavaScript کمتری به مرورگر می‌رود.
- **Client Components**: در مرورگر اجرا می‌شوند؛ برای تعامل (state، event).

**۴. تفاوت با Create React App (CRA)**
CRA فقط یک اپ تک‌صفحه‌ای می‌سازد که در مرورگر رندر می‌شود (CSR). Next.js می‌تواند SSR، SSG و ISR هم انجام دهد.

### مثال‌های عملی

**مثال ۱ — ساخت یک پروژه Next.js**

```bash
# ساخت پروژه با create-next-app (پیشنهادی)
npx create-next-app@latest my-next-app

# سؤال‌هایی می‌پرسد:
# TypeScript? → Yes
# ESLint? → Yes
# Tailwind CSS? → Yes
# src/ directory? → No (یا Yes)
# App Router? → Yes  (توصیه می‌شود)
# Turbopack? → Yes (اختیاری)

cd my-next-app
npm run dev
# → http://localhost:3000
```

**مثال ۲ — اولین صفحه (App Router)**

```tsx
// app/page.tsx
// این کامپوننت به‌طور پیش‌فرض یک Server Component است
export default function HomePage() {
  return (
    <main style={{ padding: '2rem' }}>
      <h1>سلام از Next.js!</h1>
      <p>این صفحه روی سرور رندر می‌شود.</p>
    </main>
  );
}
```

**مثال ۳ — استقرار روی Vercel**

```bash
# Build محلی برای اطمینان
npm run build

# استقرار
vercel          # Preview
vercel --prod   # Production

# یا فقط Push کنید به GitHub → Vercel خودکار Deploy می‌کند
```

### مقایسه و الگوهای مختلف

**Next.js در برابر React خالص (Vite/CRA)**

| معیار | React خالص (Vite) | Next.js |
|---|---|---|
| مسیریابی | نیاز به React Router | ✅ فایل‌محور |
| SEO | ضعیف (CSR) | ✅ عالی (SSR/SSG) |
| API بک‌اند | جداگانه | ✅ داخلی |
| بهینه‌سازی تصویر | دستی | ✅ خودکار |
| پیچیدگی | کمتر | بیشتر |
| مناسب برای | داشبورد، SPA | سایت، فروشگاه، محتوا |

**App Router در برابر Pages Router**

| معیار | App Router | Pages Router |
|---|---|---|
| پوشه | `app/` | `pages/` |
| Server Components | ✅ پیش‌فرض | ❌ |
| Layout تودرتو | ✅ | محدود |
| Streaming | ✅ | محدود |
| وضعیت | ✅ توصیه‌شده | پشتیبانی (قدیمی) |

### اشکال شایع

**۱. استفاده از `useState`/`useEffect` در Server Component.**
خطا می‌دهد. برای این کارها باید `"use client"` بگذارید.

**۲. ساخت پروژه بدون App Router.**
پروژه‌های قدیمی روی Pages Router هستند. برای پروژه جدید، App Router را انتخاب کنید.

**۳. انتظار «کد یکسان روی هر پلتفرم».**
بیشتر قابلیت‌های Next.js همه‌جا کار می‌کنند، اما بعضی (مثل PPR) روی Vercel بومی و بهینه‌ترند.

**۴. نادیده گرفتن تفاوت نسخه‌ها.**
Next.js سریع تغییر می‌کند. همیشه مستندات نسخهٔ خودتان را بخوانید.

**۵. تصور اینکه Next.js فقط برای سایت است.**
با Route Handlers می‌توانید API کامل هم بسازید.

### نکات مهم (Tips & Tricks)

- برای پروژه‌های جدید، **App Router** را انتخاب کنید.
- **TypeScript** را فعال کنید؛ در پروژه‌های واقعی کمک بزرگی است.
- از همان ابتدا **Server/Client** را درست تفکیک کنید (فصل ۳).
- برای SEO، از **Metadata API** استفاده کنید (نه دستی در `<head>`).
- قبل از هر Push، `npm run build` بگیرید تا خطاهای سرور-کلاینت را ببینید.

### خلاصه فصل

- **Next.js** فریم‌ورکی React برای اپ‌های فول‌استک است.
- سازندهٔ آن Vercel است؛ پس بهترین یکپارچگی را دارد.
- امکانات کلیدی: مسیریابی، SSR/SSG/ISR، API، بهینه‌سازی تصویر/فونت.
- **App Router** روش مدرن و توصیه‌شده است.
- Server Components پیش‌فرض‌اند؛ Client Components با `"use client"`.

---

<a id="فصل-۲"></a>
## فصل ۲: ساختار پروژه — App Router و Pages Router

### مقدمه

ساختار پوشه‌ها در Next.js معنا دارد: هر پوشه و فایل نقشی مشخص دارد. در این فصل، ساختار پروژه را دقیق می‌شناسیم تا بدانید کد را کجا بگذارید.

### مفاهیم اساسی

**۱. ساختار App Router**
```
my-next-app/
├── app/                          ← ریشهٔ مسیرها
│   ├── layout.tsx                ← چیدمان ریشه (اجباری)
│   ├── page.tsx                  ← صفحهٔ خانه (/)
│   ├── globals.css
│   ├── about/
│   │   └── page.tsx              ← /about
│   ├── blog/
│   │   ├── page.tsx              ← /blog
│   │   └── [slug]/
│   │       └── page.tsx          ← /blog/anything
│   └── api/
│       └── hello/
│           └── route.ts          ← /api/hello
├── components/                   ← کامپوننت‌های مشترک
├── lib/                          ← توابع کمکی
├── public/                       ← فایل‌های استاتیک
├── next.config.js
├── package.json
└── tsconfig.json
```

**۲. فایل‌های ویژه در App Router**

| فایل | نقش |
|---|---|
| `page.tsx` | محتوای یک مسیر (قابل دسترس برای کاربر) |
| `layout.tsx` | چیدمان مشترک (حالت، هدر، فوتر) |
| `template.tsx` | مثل layout اما در هر ناوبری بازساخته می‌شود |
| `loading.tsx` | حالت بارگذاری (Suspense) |
| `error.tsx` | مدیریت خطا (Client Component) |
| `not-found.tsx` | صفحهٔ ۴۰۴ |
| `route.ts` | Route Handler (API) |
| `default.tsx` | برای مسیرهای موازی (Parallel Routes) |

**۳. مسیرهای پویا (Dynamic Routes)**

| الگو | تطبیق | مثال |
|---|---|---|
| `[slug]` | یک بخش | `/blog/hello` |
| `[...slug]` | یک یا چند بخش | `/shop/a/b/c` |
| `[[...slug]]` | صفر یا چند بخش | `/shop` و `/shop/a` |

**۴. Route Groups**
پوشه‌های داخل پرانتز `(name)` در URL ظاهر نمی‌شوند؛ فقط برای سازمان‌دهی هستند:
```
app/
├── (marketing)/
│   ├── about/page.tsx      → /about
│   └── contact/page.tsx    → /contact
└── (shop)/
    └── products/page.tsx   → /products
```

**۵. ساختار Pages Router (قدیمی‌تر)**
```
pages/
├── index.tsx          → /
├── about.tsx          → /about
├── blog/
│   ├── index.tsx      → /blog
│   └── [slug].tsx     → /blog/anything
└── api/
    └── hello.ts       → /api/hello
```

### مثال‌های عملی

**مثال ۱ — ساخت چند مسیر**

```tsx
// app/layout.tsx — چیدمان ریشه (اجباری)
import type { Metadata } from 'next';
import './globals.css';

export const metadata: Metadata = {
  title: 'سایت من',
  description: 'ساخته‌شده با Next.js',
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="fa" dir="rtl">
      <body>{children}</body>
    </html>
  );
}
```

```tsx
// app/about/page.tsx → /about
export default function AboutPage() {
  return <h1>درباره ما</h1>;
}
```

**مثال ۲ — مسیر پویا با پارامتر**

```tsx
// app/blog/[slug]/page.tsx → /blog/هر-چیزی
export default async function BlogPost({
  params,
}: {
  params: Promise<{ slug: string }>;
}) {
  const { slug } = await params;
  return (
    <article>
      <h1>مقاله: {slug}</h1>
    </article>
  );
}
```

**مثال ۳ — حالت بارگذاری و خطا**

```tsx
// app/blog/loading.tsx — هنگام بارگذاری نمایش داده می‌شود
export default function Loading() {
  return <p>در حال بارگذاری...</p>;
}
```

```tsx
// app/blog/error.tsx — مدیریت خطا (باید Client Component باشد)
'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <h2>خطایی رخ داد!</h2>
      <p>{error.message}</p>
      <button onClick={reset}>تلاش مجدد</button>
    </div>
  );
}
```

### مقایسه و الگوهای مختلف

**چیدمان دو Router در کنار هم**

| مورد | App Router | Pages Router |
|---|---|---|
| مسیر صفحه | `app/x/page.tsx` | `pages/x.tsx` |
| چیدمان | `layout.tsx` | `_app.tsx` + کامپوننت‌ها |
| API | `app/api/x/route.ts` | `pages/api/x.ts` |
| متادیتا | `export const metadata` | `next/head` |
| Data Fetching | `fetch` در کامپوننت | `getStaticProps`/`getServerSideProps` |

**سازمان‌دهی پروژه‌های بزرگ**

| الگو | توضیح |
|---|---|
| **Colocation** | کامپوننت‌های مربوط به یک مسیر، کنار همان مسیر |
| **Route Groups** | جدا کردن بخش‌ها (marketing, shop, admin) |
| **Shared Components** | پوشهٔ `components/` برای مشترک‌ها |
| **Lib/Utils** | پوشهٔ `lib/` برای توابع کمکی |

### اشکال شایع

**۱. فراموش کردن `page.tsx`.**
پوشه بدون `page.tsx` مسیر نمی‌سازد. فقط `layout.tsx` کافی نیست.

**۲. تداخل App Router و Pages Router.**
اگر هر دو `app/` و `pages/` را داشته باشید، App Router اولویت می‌گیرد و ممکن است مسیرها قاطی شوند.

**۳. اشتباه در نام‌گذاری فایل‌های ویژه.**
`Page.tsx` با `P` بزرگ کار نمی‌کند. نام‌ها **حساس به حروف کوچک/بزرگ** هستند.

**۴. استفاده از `params` به‌صورت همگام.**
در نسخه‌های جدید Next.js، `params` یک Promise است و باید `await` شود.

**۵. گذاشتن فایل‌های غیرمسیر در `app/`.**
هر پوشه‌ای که `page.tsx` ندارد مسیر نمی‌سازد، اما فایل‌های اشتباه می‌توانند گیج‌کننده باشند. از `components/` و `lib/` استفاده کنید.

### نکات مهم (Tips & Tricks)

- **`layout.tsx` ریشه اجباری است** — همیشه `<html>` و `<body>` را بگذارید.
- برای بارگذاری بهتر، از **`loading.tsx`** استفاده کنید (تجربهٔ کاربری روان‌تر).
- مسیرهای API را در `app/api/` بگذارید تا با کد فرانت یکپارچه باشند.
- از **Route Groups** برای پروژه‌های بزرگ استفاده کنید.
- برای فایل‌های کمکی، **هرگز** داخل `app/` نگذارید.

### خلاصه فصل

- ساختار App Router: `app/` ریشه، `page.tsx` صفحه، `layout.tsx` چیدمان.
- فایل‌های ویژه: `loading`، `error`، `not-found`، `route`.
- مسیرهای پویا با `[slug]`، `[...slug]`، `[[...slug]]`.
- **Route Groups** برای سازمان‌دهی (بدون تأثیر روی URL).
- نام فایل‌ها حساس به حروف است.

---

<a id="فصل-۳"></a>
## فصل ۳: مسیریابی، Layout و کامپوننت‌های Server/Client

### مقدمه

قلب App Router، تفکیک **Server Components** از **Client Components** است. اگر این را درست بفهمید، هم عملکرد سایت‌تان بالا می‌رود و هم خطاهای عجیب را حذف می‌کنید. در این فصل، این تفکیک را با مثال‌های واقعی یاد می‌گیریم.

### مفاهیم اساسی

**۱. Server Component (پیش‌فرض)**
- روی سرور رندر می‌شود.
- می‌تواند مستقیم به دیتابیس دسترسی داشته باشد.
- JavaScript آن به مرورگر **فرستاده نمی‌شود**.
- **نمی‌تواند** از `useState`، `useEffect` یا رویدادها استفاده کند.

**۲. Client Component**
- با `"use client"` در ابتدای فایل مشخص می‌شود.
- در مرورگر اجرا می‌شود (و همچنین روی سرور برای HTML اولیه).
- می‌تواند از state، effect و رویدادها استفاده کند.
- JavaScript آن به مرورگر فرستاده می‌شود.

**۳. قاعدهٔ ساده**
> **پیش‌فرض: Server. فقط وقتی به تعامل نیاز داری: Client.**

**۴. Layout تودرتو (Nested Layout)**
`layout.tsx` ها تودرتو می‌شوند. مثلاً یک layout برای کل سایت، و یک layout مخصوص `/dashboard`:

```
app/
├── layout.tsx              ← چیدمان کل سایت (هدر، فوتر)
├── page.tsx
└── dashboard/
    ├── layout.tsx          ← چیدمان داشبورد (سایدبار)
    └── page.tsx
```

**۵. Navigation**
با کامپوننت `<Link>` از `next/link` برای ناوبری سمت کلاینت (بدون رفرش کامل).

**۶. الگوی Composition**
برای اینکه یک Server Component داخل Client Component استفاده شود، آن را به‌عنوان `children` پاس بدهید (الگوی Children Pattern).

### مثال‌های عملی

**مثال ۱ — Server Component با دسترسی مستقیم به داده**

```tsx
// app/products/page.tsx — Server Component (پیش‌فرض)
async function getProducts() {
  // مستقیم از دیتابیس یا API
  const res = await fetch('https://api.example.com/products', {
    next: { revalidate: 60 },
  });
  return res.json();
}

export default async function ProductsPage() {
  const products = await getProducts();

  return (
    <ul>
      {products.map((p: any) => (
        <li key={p.id}>{p.name}</li>
      ))}
    </ul>
  );
}
```

**مثال ۲ — Client Component با تعامل**

```tsx
// components/Counter.tsx
'use client';                       // ← لازم برای state

import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      شمارنده: {count}
    </button>
  );
}
```

**مثال ۳ — الگوی Children برای ترکیب Server داخل Client**

```tsx
// components/ClientWrapper.tsx
'use client';

export default function ClientWrapper({
  children,
}: {
  children: React.ReactNode;
}) {
  const [open, setOpen] = useState(true);

  return (
    <div>
      <button onClick={() => setOpen(!open)}>
        {open ? 'بستن' : 'باز کردن'}
      </button>
      {open && children}          {/* ← Server Component داخل Client */}
    </div>
  );
}
```

```tsx
// app/page.tsx — Server Component
import ClientWrapper from '@/components/ClientWrapper';
import ServerData from '@/components/ServerData';

export default function Page() {
  return (
    <ClientWrapper>
      <ServerData />              {/* Server Component به‌عنوان children */}
    </ClientWrapper>
  );
}
```

### مقایسه و الگوهای مختلف

**Server در برابر Client Component**

| معیار | Server Component | Client Component |
|---|---|---|
| اجرا | روی سرور | مرورگر (+ سرور) |
| JavaScript به مرورگر | ❌ | ✅ |
| دسترسی به دیتابیس | ✅ مستقیم | ❌ (نیاز به API) |
| state / effect | ❌ | ✅ |
| رویداد (onClick) | ❌ | ✅ |
| امنیت (کلیدها) | ✅ پنهان می‌مانند | ⚠️ افشا می‌شوند |
| مناسب برای | داده، محتوا | تعامل، فرم |

**سه روش ناوبری**

| روش | کامپوننت | مناسب برای |
|---|---|---|
| `<Link>` | `next/link` | ناوبری داخلی (پیشنهادی) |
| `useRouter()` | `next/navigation` | ناوبری برنامه‌ای |
| `<a href>` | HTML خالص | لینک بیرونی |

### اشکال شایع

**۱. استفاده از `useState` در Server Component.**
خطای زمان Build. باید `"use client"` اضافه کنید یا state را به یک کامپوننت فرزند منتقل کنید.

**۲. قرار دادن `"use client"` در همه‌جا.**
همهٔ مزایای Server Components را از دست می‌دهید و حجم JavaScript زیاد می‌شود.

**۳. پاس دادن تابع از Server به Client.**
توابع قابل سریال‌سازی نیستند. فقط داده‌های ساده (JSON) پاس بدهید.

**۴. افشای کلید API در Client Component.**
هر متغیری که با `NEXT_PUBLIC_` شروع نشود، در کلاینت در دسترس نیست — اما از کلید حساس در Client استفاده نکنید.

**۵. استفاده از `useRouter` از `next/router` در App Router.**
باید از `next/navigation` استفاده کنید. `next/router` مخصوص Pages Router است.

### نکات مهم (Tips & Tricks)

- **داده را در Server Component بگیرید** و فقط بخش تعاملی را Client کنید.
- `"use client"` را **پایین‌ترین سطح درخت** بگذارید، نه در ریشه.
- از الگوی **Children** برای ترکیب استفاده کنید.
- برای فرم‌ها، **Server Actions** را بررسی کنید (جایگزین مدرن API دستی).
- متادیتا را با **Metadata API** در Server Component تعریف کنید.

### خلاصه فصل

- **Server Component** پیش‌فرض است؛ روی سرور رندر می‌شود و JS کمتری می‌فرستد.
- **Client Component** با `"use client"` برای تعامل.
- `layout.tsx` ها تودرتو می‌شوند.
- ناوبری داخلی با `<Link>` از `next/link`.
- برای ترکیب، از الگوی **children** استفاده کنید.

---

<a id="فصل-۴"></a>
## فصل ۴: Data Fetching — SSR، SSG، ISR

### مقدمه

«چطور داده را بیاورم؟» مهم‌ترین سؤال در هر اپلیکیشن است. Next.js چهار استراتژی اصلی دارد و انتخاب درست، تفاوت بین یک سایت کند و یک سایت پرسرعت است. در این فصل، همه را با مثال یاد می‌گیریم.

### مفاهیم اساسی

**۱. در App Router، همه چیز با `fetch` انجام می‌شود**
Next.js متد `fetch` را گسترش داده است. با گزینه‌های `cache` و `next.revalidate` کنترل می‌کنید:

| تنظیم | رفتار | معادل قدیمی |
|---|---|---|
| `{ cache: 'force-cache' }` | کش دائمی (پیش‌فرض) | SSG |
| `{ cache: 'no-store' }` | بدون کش، هر درخواست تازه | SSR |
| `{ next: { revalidate: 60 } }` | کش با بازسازی هر ۶۰ ثانیه | ISR |

**۲. چهار استراتژی**

| استراتژی | زمان ساخت | سرعت | داده پویا |
|---|---|---|---|
| **SSG** | زمان Build | ⚡ بسیار سریع | ❌ |
| **SSR** | هر درخواست | متوسط | ✅ |
| **ISR** | Build + بازسازی دوره‌ای | ⚡ سریع | ✅ (با تأخیر) |
| **CSR** | مرورگر | کند در بار اول | ✅ |

**۳. `generateStaticParams` (برای مسیرهای پویا)**
به Next.js می‌گوید کدام مسیرهای پویا را از قبل بسازد (مثلاً همهٔ مقالات یک وبلاگ).

**۴. Streaming و Suspense**
می‌توانید بخش‌های کند صفحه را با `<Suspense>` جدا کنید تا بقیهٔ صفحه سریع نمایش داده شود.

**۵. Request Memoization**
اگر یک `fetch` را چند بار در یک رندر صدا بزنید، Next.js آن را **یک بار** اجرا می‌کند (کش خودکار در همان رندر).

### مثال‌های عملی

**مثال ۱ — SSG (تولید استاتیک)**

```tsx
// app/blog/page.tsx
async function getPosts() {
  const res = await fetch('https://api.example.com/posts', {
    cache: 'force-cache',        // ← SSG: یک بار در Build
  });
  return res.json();
}

export default async function BlogPage() {
  const posts = await getPosts();
  return <ul>{posts.map((p: any) => <li key={p.id}>{p.title}</li>)}</ul>;
}
```

**مثال ۲ — SSR (رندر در هر درخواست)**

```tsx
// app/dashboard/page.tsx
async function getUserData() {
  const res = await fetch('https://api.example.com/me', {
    cache: 'no-store',           // ← SSR: همیشه تازه
  });
  return res.json();
}

export default async function DashboardPage() {
  const user = await getUserData();
  return <h1>سلام {user.name}</h1>;
}
```

**مثال ۳ — ISR (بازسازی دوره‌ای)**

```tsx
// app/news/page.tsx
async function getNews() {
  const res = await fetch('https://api.example.com/news', {
    next: { revalidate: 60 },    // ← ISR: هر ۶۰ ثانیه بازسازی
  });
  return res.json();
}

export default async function NewsPage() {
  const news = await getNews();
  return <ul>{news.map((n: any) => <li key={n.id}>{n.title}</li>)}</ul>;
}
```

**مثال ۴ — مسیر پویا با `generateStaticParams`**

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts').then(r => r.json());
  // از قبل همهٔ مسیرها را می‌سازد
  return posts.map((p: any) => ({ slug: p.slug }));
}

export default async function Post({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;
  const post = await fetch(`https://api.example.com/posts/${slug}`).then(r => r.json());
  return <article><h1>{post.title}</h1><p>{post.body}</p></article>;
}
```

**مثال ۵ — Streaming با Suspense**

```tsx
// app/page.tsx
import { Suspense } from 'react';

async function SlowComponent() {
  const data = await fetch('https://api.example.com/slow', { cache: 'no-store' })
    .then(r => r.json());
  return <div>{data.value}</div>;
}

export default function Page() {
  return (
    <main>
      <h1>صفحه اصلی</h1>
      {/* بخش سریع فوراً نمایش داده می‌شود */}
      <Suspense fallback={<p>در حال بارگذاری بخش کند...</p>}>
        <SlowComponent />
      </Suspense>
    </main>
  );
}
```

### مقایسه و الگوهای مختلف

**App Router در برابر Pages Router (Data Fetching)**

| App Router | Pages Router | توضیح |
|---|---|---|
| `fetch` با `cache` | `getStaticProps` | SSG |
| `fetch` با `no-store` | `getServerSideProps` | SSR |
| `fetch` با `revalidate` | `getStaticProps` + `revalidate` | ISR |
| `generateStaticParams` | `getStaticPaths` | مسیرهای پویا |
| `Suspense` + `loading.tsx` | CSR + `useEffect` | Streaming |

**انتخاب استراتژی بر اساس نوع صفحه**

| نوع صفحه | استراتژی پیشنهادی |
|---|---|
| صفحهٔ اصلی سایت | SSG یا ISR |
| وبلاگ / مستندات | SSG + ISR |
| محصولات فروشگاه | ISR (revalidate کوتاه) |
| داشبورد کاربری | SSR یا CSR |
| پروفایل شخصی | SSR |
| دادهٔ لحظه‌ای (قیمت سهام) | CSR یا SSR با no-store |

### اشکال شایع

**۱. استفاده از `no-store` برای همه چیز.**
سرور را زیر بار می‌برد و هزینه را بالا می‌برد. فقط برای دادهٔ واقعاً پویا.

**۲. `force-cache` برای داده‌ای که باید تازه باشد.**
کاربر دادهٔ قدیمی می‌بیند. برای این‌ها ISR یا `no-store` استفاده کنید.

**۳. فراموش کردن `generateStaticParams`.**
مسیرهای پویا در اولین درخواست (on-demand) ساخته می‌شوند که کندتر است.

**۴. صدا زدن `fetch` در Client Component بدون نیاز.**
اگر می‌توانید روی سرور بگیرید، همان‌جا بگیرید.

**۵. نادیده گرفتن `revalidate` در CMS.**
اگر محتوا از CMS می‌آید، مقدار مناسب `revalidate` یا بازسازی On-Demand را فراموش نکنید.

### نکات مهم (Tips & Tricks)

- **پیش‌فرض را استاتیک بگیرید**؛ فقط در صورت نیاز پویا کنید.
- برای محتوای دوره‌ای، **ISR با `revalidate`** بهترین انتخاب است.
- از **`generateStaticParams`** برای مسیرهای پویا استفاده کنید.
- برای بخش‌های کند، **Streaming با Suspense** تجربهٔ کاربری را بهتر می‌کند.
- **Request Memoization** را به خاطر بسپارید — نیازی به کش دستی در همان رندر نیست.

### خلاصه فصل

- در App Router همه چیز با `fetch` و گزینه‌های `cache`/`revalidate` کنترل می‌شود.
- **SSG** سریع‌ترین، **SSR** پویاترین، **ISR** ترکیبی است.
- **`generateStaticParams`** مسیرهای پویا را از قبل می‌سازد.
- **Streaming/Suspense** بخش‌های کند را جدا می‌کند.
- انتخاب درست استراتژی، کلید عملکرد و هزینه است.

---

<a id="فصل-۵"></a>
## فصل ۵: API Routes و Route Handlers

### مقدمه

گاهی نیاز دارید منطقی روی سرور اجرا شود: خواندن از دیتابیس، فراخوانی API با کلید مخفی، پردازش فرم. در App Router این کار با **Route Handlers** انجام می‌شود (و در Pages Router با API Routes). در این فصل، هر دو را یاد می‌گیریم.

### مفاهیم اساسی

**۱. Route Handler چیست؟**
فایلی به نام `route.ts` در `app/api/...` که متدهای HTTP را مدیریت می‌کند:

```ts
export async function GET(request: Request) { ... }
export async function POST(request: Request) { ... }
export async function PUT(request: Request) { ... }
export async function DELETE(request: Request) { ... }
```

**۲. تفاوت با API Routes (Pages Router)**

| معیار | Route Handler (App) | API Route (Pages) |
|---|---|---|
| مسیر | `app/api/x/route.ts` | `pages/api/x.ts` |
| امضای تابع | `(request: Request)` | `(req, res)` |
| پاسخ | `Response` استاندارد | `res.json()` |
| Runtime | Node.js / Edge | Node.js |
| وضعیت | ✅ توصیه‌شده | پشتیبانی |

**۳. Runtime**
هر Route Handler می‌تواند روی **Node.js** یا **Edge** اجرا شود:

```ts
export const runtime = 'edge';   // یا 'nodejs' (پیش‌فرض)
```

**۴. کنترل کش**
```ts
export const dynamic = 'force-dynamic';   // همیشه پویا (SSR)
export const revalidate = 60;             // کش با بازسازی
```

**۵. خواندن ورودی‌ها**
- Query: `request.nextUrl.searchParams`
- Body: `await request.json()`
- Headers: `request.headers.get('...')`
- مسیر پویا: پارامتر دوم `{ params }`

### مثال‌های عملی

**مثال ۱ — Route Handler ساده**

```ts
// app/api/hello/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  return NextResponse.json({
    message: 'سلام!',
    time: new Date().toISOString(),
  });
}
```

**مثال ۲ — مدیریت POST با اعتبارسنجی**

```ts
// app/api/contact/route.ts
import { NextResponse } from 'next/server';

export async function POST(request: Request) {
  try {
    const body = await request.json();

    // اعتبارسنجی ساده
    if (!body.name || !body.email) {
      return NextResponse.json(
        { error: 'نام و ایمیل الزامی است' },
        { status: 400 }
      );
    }

    // اینجا می‌توانید در دیتابیس ذخیره کنید یا ایمیل بفرستید
    console.log('پیام جدید:', body);

    return NextResponse.json(
      { success: true, message: 'پیام دریافت شد' },
      { status: 201 }
    );
  } catch {
    return NextResponse.json(
      { error: 'بدنه درخواست نامعتبر است' },
      { status: 400 }
    );
  }
}
```

**مثال ۳ — مسیر پویا در Route Handler**

```ts
// app/api/users/[id]/route.ts
import { NextResponse } from 'next/server';

export async function GET(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  // در عمل: خواندن از دیتابیس
  return NextResponse.json({ id, name: `کاربر ${id}` });
}
```

**مثال ۴ — استفاده از متغیر محیطی و پروکسی امن**

```ts
// app/api/weather/route.ts
import { NextResponse } from 'next/server';

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const city = searchParams.get('city') ?? 'tehran';

  // کلید API فقط روی سرور است و در مرورگر افشا نمی‌شود
  const res = await fetch(
    `https://api.weatherapi.com/v1/current.json?key=${process.env.WEATHER_API_KEY}&q=${city}`
  );

  if (!res.ok) {
    return NextResponse.json({ error: 'خطا در دریافت داده' }, { status: 502 });
  }

  const data = await res.json();
  return NextResponse.json(data);
}
```

**مثال ۵ — Edge Runtime برای پاسخ سریع**

```ts
// app/api/ping/route.ts
export const runtime = 'edge';

export async function GET() {
  return new Response(
    JSON.stringify({ pong: true, region: process.env.VERCEL_REGION }),
    { headers: { 'content-type': 'application/json' } }
  );
}
```

### مقایسه و الگوهای مختلف

**Node.js در برابر Edge Runtime**

| معیار | Node.js | Edge |
|---|---|---|
| کتابخانه‌های npm | ✅ همه | محدود (Web APIs) |
| دسترسی به فایل سیستم | ✅ | ❌ |
| Latency | بیشتر | کمتر |
| مناسب برای | دیتابیس، پردازش سنگین | پاسخ سریع، پروکسی |
| محدودیت زمان | بیشتر | کوتاه |

**سه روش مدیریت داده در فرم**

| روش | توضیح | مناسب برای |
|---|---|---|
| **API Route + fetch** | سنتی و انعطاف‌پذیر | اکثر موارد |
| **Server Actions** | تابع سرور مستقیم از فرم | فرم‌های ساده |
| **Third-party (tRPC)** | تایپ‌سیف کامل | پروژه‌های بزرگ |

### اشکال شایع

**۱. قرار دادن کلید API در کد سمت کلاینت.**
همیشه کلیدها را در Route Handler (سمت سرور) استفاده کنید.

**۲. استفاده از Edge برای کارهای سنگین.**
دسترسی به بعضی پکیج‌ها و فایل سیستم وجود ندارد و ممکن است خطا بدهد.

**۳. فراموش کردن اعتبارسنجی ورودی.**
هر ورودی از بیرون را اعتبارسنجی کنید؛ در غیر این صورت در معرض حمله هستید.

**۴. مدیریت نکردن خطاها.**
همیشه `try/catch` و کد وضعیت مناسب برگردانید.

**۵. کش ناخواسته روی پاسخ‌های پویا.**
اگر پاسخ باید همیشه تازه باشد، `dynamic = 'force-dynamic'` را تنظیم کنید.

### نکات مهم (Tips & Tricks)

- از **`NextResponse.json()`** برای پاسخ استاندارد استفاده کنید.
- برای مسیرهای محافظت‌شده، توکن را در **Middleware** بررسی کنید (فایل ۰۶).
- برای کار با دیتابیس، **Node.js Runtime** انتخاب کنید.
- ورودی‌ها را با کتابخانه‌ای مثل **Zod** اعتبارسنجی کنید.
- کدهای وضعیت درست برگردانید (۲۰۰، ۲۰۱، ۴۰۰، ۴۰۱، ۵۰۰).

### خلاصه فصل

- **Route Handlers** با `route.ts` در `app/api/` ساخته می‌شوند.
- متدهای HTTP: `GET`, `POST`, `PUT`, `DELETE`.
- می‌توانید روی **Node.js** یا **Edge** اجرا کنید.
- کلیدهای حساس همیشه سمت سرور بمانند.
- همیشه ورودی را اعتبارسنجی و خطا را مدیریت کنید.

---

<a id="فصل-۶"></a>
## فصل ۶: بهینه‌سازی تصویر و فونت در Next.js

### مقدمه

تصاویر و فونت‌ها معمولاً بزرگ‌ترین بخش حجم یک صفحه‌اند. Next.js دو ابزار قدرتمند برای این‌ها دارد: `next/image` و `next/font`. در این فصل یاد می‌گیریم چطور سایت‌مان را سبک و سریع کنیم.

### مفاهیم اساسی

**۱. کامپوننت `<Image>`**
جایگزین `<img>` است و این کارها را خودکار انجام می‌دهد:
- تبدیل به فرمت‌های مدرن (**WebP/AVIF**).
- تغییر اندازه بر اساس دستگاه.
- **Lazy loading** (بارگذاری فقط در صورت نیاز).
- جلوگیری از **Layout Shift** (با مشخص کردن ابعاد).

**۲. بارگذارنده (Loader) تصویر**
Vercel یک **Image Optimization API** دارد که تصاویر را روی لبه بهینه می‌کند. با `next.config.js` قابل تنظیم است.

**۳. `next/font`**
فونت‌ها را در زمان Build دانلود و میزبانی می‌کند؛ یعنی:
- درخواست اضافه به سرورهای گوگل حذف می‌شود.
- **Layout Shift** صفر می‌شود.
- حریم خصوصی بهتر (بدون درخواست به دامنهٔ سوم).

**۴. `remotePatterns`**
برای استفاده از تصاویر دامنه‌های بیرونی، باید دامنه را در `next.config.js` مجاز کنید.

### مثال‌های عملی

**مثال ۱ — استفاده از `<Image>`**

```tsx
// app/page.tsx
import Image from 'next/image';

export default function Page() {
  return (
    <div>
      {/* تصویر محلی از پوشه public */}
      <Image
        src="/hero.jpg"
        alt="تصویر اصلی"
        width={800}
        height={400}
        priority              // برای تصویر بالای صفحه (LCP)
      />

      {/* تصویر با پرکردن والد */}
      <div style={{ position: 'relative', width: '100%', height: 300 }}>
        <Image
          src="/banner.jpg"
          alt="بنر"
          fill
          style={{ objectFit: 'cover' }}
        />
      </div>
    </div>
  );
}
```

**مثال ۲ — تنظیم دامنه‌های تصویر بیرونی**

```js
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'images.example.com',
        pathname: '/uploads/**',
      },
      {
        protocol: 'https',
        hostname: 'cdn.shopify.com',
      },
    ],
  },
};

module.exports = nextConfig;
```

**مثال ۳ — استفاده از `next/font`**

```tsx
// app/layout.tsx
import { Inter, Vazirmatn } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
});

const vazir = Vazirmatn({
  subsets: ['arabic'],
  display: 'swap',
});

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="fa" dir="rtl" className={`${inter.className} ${vazir.className}`}>
      <body>{children}</body>
    </html>
  );
}
```

**مثال ۴ — فونت محلی**

```tsx
// app/layout.tsx
import localFont from 'next/font/local';

const myFont = localFont({
  src: [
    { path: './fonts/Vazir-Regular.woff2', weight: '400', style: 'normal' },
    { path: './fonts/Vazir-Bold.woff2', weight: '700', style: 'normal' },
  ],
  display: 'swap',
});

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="fa" dir="rtl" className={myFont.className}>
      <body>{children}</body>
    </html>
  );
}
```

### مقایسه و الگوهای مختلف

**`<img>` در برابر `<Image>`**

| معیار | `<img>` | `<Image>` |
|---|---|---|
| فرمت مدرن (WebP/AVIF) | ❌ دستی | ✅ خودکار |
| تغییر اندازه | ❌ دستی | ✅ خودکار |
| Lazy loading | ❌ | ✅ پیش‌فرض |
| Layout Shift | ⚠️ | ✅ جلوگیری |
| مناسب برای | SVG، تصویر ثابت خاص | اکثر تصاویر |

**سه راه بارگذاری فونت**

| روش | مزیت | عیب |
|---|---|---|
| **`next/font/google`** | ساده، بهینه | نیاز به اینترنت در Build |
| **`next/font/local`** | کنترل کامل، آفلاین | مدیریت دستی فایل‌ها |
| **`<link>` دستی** | انعطاف | درخواست اضافه، Layout Shift |

### اشکال شایع

**۱. فراموش کردن `width`/`height` (یا `fill`).**
Next.js خطا می‌دهد و از Layout Shift جلوگیری نمی‌شود.

**۲. استفاده از `<img>` برای همه تصاویر.**
مزایای بهینه‌سازی را از دست می‌دهید.

**۳. تصویر بیرونی بدون `remotePatterns`.**
خطای «hostname is not configured» می‌گیرید.

**۴. `priority` روی همهٔ تصاویر.**
باید فقط برای تصویر اصلی (بالای صفحه / LCP) استفاده شود.

**۵. فونت‌های زیاد.**
هر فونت اضافه، حجم را بالا می‌برد. حداکثر ۲–۳ وزن انتخاب کنید.

**۶. محدودیت مصرف Image Optimization.**
در پلن‌های بالاتر، مصرف تصویر هزینه دارد. تصاویر را بهینه و کش کنید.

### نکات مهم (Tips & Tricks)

- تصویر **LCP** (بزرگ‌ترین تصویر بالای صفحه) را با `priority` مشخص کنید.
- از فرمت **AVIF/WebP** استفاده کنید (پیش‌فرض Next.js).
- برای تصاویر پس‌زمینه، از `fill` + `objectFit` استفاده کنید.
- فونت را با `display: 'swap'` بارگذاری کنید تا متن فوراً دیده شود.
- تصاویر را قبل از آپلود با ابزاری مثل Squoosh فشرده کنید.

### خلاصه فصل

- **`<Image>`** تصاویر را خودکار بهینه، ریسپانسیو و Lazy می‌کند.
- دامنه‌های بیرونی را در `remotePatterns` مجاز کنید.
- **`next/font`** فونت‌ها را بهینه و بدون Layout Shift بارگذاری می‌کند.
- `priority` را فقط برای تصویر اصلی استفاده کنید.
- تصویر و فونت بهینه = سرعت بالاتر و Core Web Vitals بهتر.

---

## منابع و مراجع

### مستندات رسمی
- Next.js Documentation: <https://nextjs.org/docs>
- Next.js on Vercel: <https://vercel.com/docs/frameworks/nextjs>
- App Router (Routing): <https://nextjs.org/docs/app/building-your-application/routing>
- Data Fetching: <https://nextjs.org/docs/app/building-your-application/data-fetching>
- Route Handlers: <https://nextjs.org/docs/app/building-your-application/routing/route-handlers>
- next/image: <https://nextjs.org/docs/app/api-reference/components/image>
- next/font: <https://nextjs.org/docs/app/api-reference/components/font>

### مفاهیم پیشنهادی برای مطالعه بیشتر
- React Server Components: <https://react.dev/reference/rsc/server-components>
- Web Vitals: <https://web.dev/vitals/>
- Learn Next.js (دوره رسمی رایگان): <https://nextjs.org/learn>

### گام بعدی
- [فایل ۰۵ — توابع Serverless و Edge](./05-Serverless-and-Edge-Functions.md)
- [فایل ۱۱ — کش، ISR و PPR](./11-Caching-ISR-and-PPR.md)
