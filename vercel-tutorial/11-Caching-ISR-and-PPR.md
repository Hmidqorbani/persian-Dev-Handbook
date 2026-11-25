# ۱۱ — کش، ISR و PPR

**مدت یادگیری**: ۴ تا ۵ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: مطالعه فایل‌های ۰۴ (Next.js) و ۱۰ (عملکرد)

---

## فهرست محتوا

- [فصل ۱: کش در Next.js — نگاه کلی](#فصل-۱)
- [فصل ۲: ISR — بازسازی افزایشی استاتیک](#فصل-۲)
- [فصل ۳: PPR — رندر جزئی اولیه](#فصل-۳)
- [فصل ۴: revalidate، Cache Tags و On-Demand](#فصل-۴)
- [فصل ۵: الگوهای ترکیبی و اشکالات رایج](#فصل-۵)

---

<a id="فصل-۱"></a>
## فصل ۱: کش در Next.js — نگاه کلی

### مقدمه

Next.js چند لایهٔ کش دارد که اگر درست استفاده شوند، سایت شما را فوق‌سریع می‌کنند. اما اگر نفهمید کدام لایه کجاست، رفتارهای عجیب می‌بینید («چرا داده به‌روز نشد؟»). در این فصل، نقشهٔ کامل کش را می‌کشیم.

### مفاهیم اساسی

**۱. چهار لایهٔ کش در Next.js**

| لایه | محل | چه چیزی کش می‌شود |
|---|---|---|
| **Request Memoization** | یک رندر | تکرار یک `fetch` در همان رندر |
| **Data Cache** | سرور/لبه | نتیجهٔ `fetch` بین رندرها |
| **Full Route Cache** | سرور/لبه | HTML کامل یک مسیر استاتیک |
| **Router Cache** | مرورگر | صفحات بازدیدشده (ناوبری سریع) |

**۲. ترتیب کار**
```
درخواست کاربر
   ↓
Router Cache (مرورگر) — اگر هست، فوری
   ↓
Full Route Cache (لبه) — اگر هست، سریع
   ↓
Data Cache — نتیجهٔ fetch
   ↓
منبع (API/دیتابیس)
```

**۳. کنترل با `fetch`**

```js
fetch(url, { cache: 'force-cache' })          // کش دائمی (پیش‌فرض)
fetch(url, { cache: 'no-store' })             // بدون کش
fetch(url, { next: { revalidate: 60 } })      // کش با بازسازی ۶۰ ثانیه
fetch(url, { next: { tags: ['products'] } })  // کش برچسبی
```

**۴. تفاوت Static و Dynamic Route**

| نوع | چه زمانی |
|---|---|
| **Static** | بدون دادهٔ پویا در زمان درخواست |
| **Dynamic** | استفاده از `cookies()`، `headers()`، `searchParams`، `no-store` |

**۵. `dynamic` و `revalidate` در سطح مسیر**

```ts
export const dynamic = 'force-static';   // همیشه استاتیک
export const dynamic = 'force-dynamic';  // همیشه پویا
export const revalidate = 60;            // بازسازی هر ۶۰ ثانیه
```

### مثال‌های عملی

**مثال ۱ — مشاهدهٔ رفتار کش**

```tsx
// app/products/page.tsx
async function getProducts() {
  const res = await fetch('https://api.example.com/products', {
    next: { revalidate: 60, tags: ['products'] },
  });
  return res.json();
}

export default async function ProductsPage() {
  const products = await getProducts();
  console.log('این لاگ فقط هنگام بازسازی چاپ می‌شود');  // ← نکته مهم
  return <ul>{products.map((p: any) => <li key={p.id}>{p.name}</li>)}</ul>;
}
```

**مثال ۲ — Request Memoization**

```tsx
// در یک رندر، این fetch فقط یک بار اجرا می‌شود
async function getUser(id: string) {
  return fetch(`https://api.example.com/users/${id}`).then((r) => r.json());
}

export default async function Page() {
  const user1 = await getUser('1');
  const user2 = await getUser('1');   // ← از حافظه می‌آید، نه درخواست جدید
  return <div>{user1.name} / {user2.name}</div>;
}
```

**مثال ۳ — مسیر پویا**

```tsx
// app/dashboard/page.tsx
import { cookies } from 'next/headers';

export default async function Dashboard() {
  const cookieStore = await cookies();
  const token = cookieStore.get('token')?.value;

  // استفاده از cookies → مسیر به‌طور خودکار Dynamic می‌شود
  const data = await fetch('https://api.example.com/me', {
    headers: { Authorization: `Bearer ${token}` },
    cache: 'no-store',
  }).then((r) => r.json());

  return <h1>{data.name}</h1>;
}
```

### مقایسه و الگوهای مختلف

**Static در برابر Dynamic Route**

| معیار | Static | Dynamic |
|---|---|---|
| سرعت | ✅✅ | ✅ |
| هزینه | کم | بیشتر |
| دادهٔ پویا | ❌ | ✅ |
| مناسب برای | وبلاگ، محصولات | داشبورد، پروفایل |

**چهار ترکیب `cache` و `revalidate`**

| تنظیم | رفتار | معادل |
|---|---|---|
| `force-cache` | کش دائمی | SSG |
| `no-store` | بدون کش | SSR |
| `revalidate: n` | کش + بازسازی | ISR |
| `revalidate: 0` | همیشه تازه | SSR |

### اشکال شایع

**۱. انتظار به‌روز شدن فوری دادهٔ کش‌شده.**
`force-cache` تا بازسازی، قدیمی می‌ماند. برای دادهٔ لحظه‌ای `no-store`.

**۲. استفاده از `cookies()` بدون آگاهی.**
مسیر به‌طور خودکار Dynamic می‌شود و مزایای استاتیک از دست می‌رود.

**۳. اشتباه بین Data Cache و Full Route Cache.**
هر کدام جدا کنترل می‌شوند.

**۴. فراموش کردن Router Cache.**
ناوبری سریع است اما ممکن است نسخهٔ قدیمی نشان دهد.

**۵. کش کردن دادهٔ کاربر-محور در Data Cache.**
کاربر دیگران دادهٔ شما را می‌بیند. برای این‌ها `no-store`.

### نکات مهم (Tips & Tricks)

- **چهار لایهٔ کش** را به خاطر بسپارید.
- برای محتوای مشترک: `force-cache` یا `revalidate`.
- برای دادهٔ شخصی: `no-store`.
- برچسب (`tags`) بگذارید تا Invalidation آسان شود.
- `console.log` در Server Component نشان می‌دهد کِی بازسازی می‌شود.

### خلاصه فصل

- Next.js چهار لایهٔ کش دارد: **Memoization، Data، Full Route، Router**.
- `fetch` با `cache` و `revalidate` کنترل می‌شود.
- Static سریع‌تر و ارزان‌تر است؛ Dynamic برای دادهٔ شخصی.
- `cookies()`/`headers()` مسیر را Dynamic می‌کند.
- دادهٔ شخصی هرگز کش نشود.

---

<a id="فصل-۲"></a>
## فصل ۲: ISR — بازسازی افزایشی استاتیک

### مقدمه

**ISR** ترکیب جادویی SSG و SSR است: صفحه استاتیک و سریع است، اما در بازه‌های زمانی مشخص بازسازی می‌شود. برای وبلاگ، فروشگاه و اخبار ایده‌آل است.

### مفاهیم اساسی

**۱. ISR چیست؟**
صفحه در زمان Build ساخته می‌شود (مثل SSG)، اما با یک `revalidate` مشخص، پس از انقضا در پس‌زمینه بازسازی می‌شود.

**۲. چرخهٔ ISR**
```
درخواست ۱ → صفحهٔ استاتیک (سریع) ✅
   ↓ (گذشت ۶۰ ثانیه)
درخواست ۲ → صفحهٔ قدیمی سرو می‌شود + بازسازی در پس‌زمینه
   ↓
درخواست ۳ → صفحهٔ تازه ✅
```

**۳. انواع ISR**

| نوع | توضیح |
|---|---|
| **Time-based** | بازسازی پس از مدت مشخص (`revalidate: 60`) |
| **On-Demand** | بازسازی با فراخوانی API |
| **Tag-based** | بازسازی همهٔ صفحات با یک برچسب |

**۴. `generateStaticParams`**
مسیرهای پویا را از قبل می‌سازد:

```tsx
export async function generateStaticParams() {
  const posts = await getPosts();
  return posts.map((p) => ({ slug: p.slug }));
}
```

**۵. `dynamicParams`**
کنترل رفتار مسیرهایی که در Build ساخته نشده‌اند:

```tsx
export const dynamicParams = true;   // در اولین درخواست ساخته شود (پیش‌فرض)
export const dynamicParams = false;  // فقط مسیرهای از پیش ساخته → ۴۰۴
```

### مثال‌های عملی

**مثال ۱ — ISR پایه**

```tsx
// app/blog/page.tsx
export const revalidate = 60;   // کل صفحه هر ۶۰ ثانیه بازسازی

async function getPosts() {
  const res = await fetch('https://api.example.com/posts');
  return res.json();
}

export default async function BlogPage() {
  const posts = await getPosts();
  return <ul>{posts.map((p: any) => <li key={p.id}>{p.title}</li>)}</ul>;
}
```

**مثال ۲ — ISR برای مسیر پویا**

```tsx
// app/blog/[slug]/page.tsx
export const revalidate = 3600;   // هر ساعت

export async function generateStaticParams() {
  const posts = await fetch('https://api.example.com/posts').then((r) => r.json());
  return posts.map((p: any) => ({ slug: p.slug }));
}

export default async function Post({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;
  const post = await fetch(`https://api.example.com/posts/${slug}`, {
    next: { revalidate: 3600, tags: [`post:${slug}`] },
  }).then((r) => r.json());

  return <article><h1>{post.title}</h1><p>{post.body}</p></article>;
}
```

**مثال ۳ — ISR با صفحهٔ محصول (قیمت)**

```tsx
// app/products/[id]/page.tsx
export const revalidate = 300;   // قیمت هر ۵ دقیقه تازه می‌شود

export default async function Product({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const product = await fetch(`https://api.example.com/products/${id}`, {
    next: { revalidate: 300, tags: [`product:${id}`] },
  }).then((r) => r.json());

  return (
    <div>
      <h1>{product.name}</h1>
      <p>قیمت: {product.price}</p>
      <p>آخرین به‌روزرسانی: {new Date().toLocaleString('fa-IR')}</p>
    </div>
  );
}
```

**مثال ۴ — بازسازی دستی پس از تغییر**

```ts
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache';
import { NextResponse } from 'next/server';

export async function POST(request: Request) {
  const { searchParams } = new URL(request.url);
  const secret = searchParams.get('secret');

  if (secret !== process.env.REVALIDATE_SECRET) {
    return NextResponse.json({ error: 'دسترسی غیرمجاز' }, { status: 401 });
  }

  const path = searchParams.get('path');
  const tag = searchParams.get('tag');

  if (tag) {
    revalidateTag(tag);            // همهٔ صفحات با این برچسب
    return NextResponse.json({ revalidated: true, tag });
  }

  if (path) {
    revalidatePath(path);          // یک مسیر خاص
    return NextResponse.json({ revalidated: true, path });
  }

  return NextResponse.json({ error: 'path یا tag لازم است' }, { status: 400 });
}
```

### مقایسه و الگوهای مختلف

**ISR در برابر SSG و SSR**

| معیار | SSG | ISR | SSR |
|---|---|---|---|
| سرعت | ✅✅ | ✅✅ | ✅ |
| تازگی داده | ❌ | ✅ (با تأخیر) | ✅✅ |
| هزینه | کم | کم | بیشتر |
| مناسب برای | محتوای ثابت | محتوای دوره‌ای | دادهٔ شخصی |

**انتخاب `revalidate`**

| نوع محتوا | مقدار پیشنهادی |
|---|---|
| اخبار فوری | ۳۰–۶۰ ثانیه |
| محصولات فروشگاه | ۵–۱۵ دقیقه |
| وبلاگ | ۱–۲۴ ساعت |
| مستندات | ۱ روز یا بیشتر |

### اشکال شایع

**۱. `revalidate` خیلی کوتاه برای همه چیز.**
هزینه و بار سرور بالا می‌رود. مقدار مناسب انتخاب کنید.

**۲. انتظار ISR فوری.**
بازسازی در پس‌زمینه است؛ کاربر اول دادهٔ قدیمی می‌بیند.

**۳. فراموش کردن `generateStaticParams`.**
مسیرها در اولین درخواست ساخته می‌شوند (کندتر).

**۴. نبود برچسب برای Invalidation.**
بازسازی انتخابی سخت می‌شود.

**۵. ISR روی دادهٔ شخصی.**
خطر نشت داده. برای دادهٔ شخصی از Dynamic استفاده کنید.

### نکات مهم (Tips & Tricks)

- **`revalidate` مناسب** برای هر نوع محتوا انتخاب کنید.
- **`tags`** بگذارید تا Invalidation دقیق ممکن شود.
- از **On-Demand Revalidation** بعد از تغییر CMS استفاده کنید.
- `dynamicParams = false` برای مسیرهای محدود (امنیت بیشتر).
- لاگ بگذارید تا بدانید بازسازی کِی رخ می‌دهد.

### خلاصه فصل

- **ISR** = استاتیک + بازسازی دوره‌ای.
- `revalidate` مدت تازگی را تعیین می‌کند.
- `generateStaticParams` مسیرهای پویا را از پیش می‌سازد.
- **On-Demand** و **Tag-based** بازسازی دقیق می‌دهند.
- `revalidate` را متناسب با نوع محتوا انتخاب کنید.

---

<a id="فصل-۳"></a>
## فصل ۳: PPR — رندر جزئی اولیه

### مقدمه

**PPR (Partial Prerendering)** جدیدترین نوآوری Next.js است: ترکیب یک پوستهٔ استاتیک فوری با بخش‌های پویا که استریم می‌شوند. نتیجه: سریع‌ترین تجربهٔ ممکن.

### مفاهیم اساسی

**۱. PPR چیست؟**
یک صفحه می‌تواند هم **استاتیک** و هم **پویا** باشد: پوسته (Shell) از قبل ساخته و فوری سرو می‌شود، و بخش‌های پویا در **همان درخواست** استریم می‌شوند.

**۲. تفاوت با ISR**

| معیار | ISR | PPR |
|---|---|---|
| کل صفحه | استاتیک | ترکیبی |
| بخش پویا | ندارد | ✅ استریم |
| سرعت اولیه | سریع | ✅✅ فوری |
| مناسب برای | محتوای دوره‌ای | صفحات با بخش شخصی |

**۳. چطور کار می‌کند؟**
```
۱. پوستهٔ استاتیک از کش لبه → فوری ✅
۲. مرزهای Suspense به‌صورت placeholder ارسال می‌شوند
۳. بخش‌های پویا در همان درخواست استریم می‌شوند
```

**۴. پیش‌نیازها**
PPR در حال بلوغ است و ممکن است نیاز به فعال‌سازی در `next.config.js` داشته باشد:

```js
// next.config.js
module.exports = {
  experimental: {
    ppr: 'incremental',   // فعال‌سازی افزایشی
  },
};
```

سپس در سطح مسیر:

```tsx
export const experimental_ppr = true;
```

> **نکته**: وضعیت PPR با نسخه‌های Next.js تغییر می‌کند. مستندات نسخهٔ خود را بررسی کنید.

**۵. چه زمانی PPR بهترین است؟**
صفحه‌ای که بیشترش استاتیک است اما بخشی (سبد خرید، نام کاربر) پویاست.

### مثال‌های عملی

**مثال ۱ — فعال‌سازی PPR برای یک مسیر**

```tsx
// app/page.tsx
import { Suspense } from 'react';
import { cookies } from 'next/headers';

export const experimental_ppr = true;

// بخش استاتیک (از پیش ساخته)
function StaticHeader() {
  return <header><h1>فروشگاه من</h1></header>;
}

// بخش پویا (شخصی) — استریم می‌شود
async function UserCart() {
  const cookieStore = await cookies();
  const cart = await getCart(cookieStore.get('cartId')?.value);
  return <div>سبد خرید: {cart.items.length} آیتم</div>;
}

export default function Page() {
  return (
    <main>
      <StaticHeader />
      <Suspense fallback={<div>در حال بارگذاری سبد...</div>}>
        <UserCart />
      </Suspense>
    </main>
  );
}
```

**مثال ۲ — مقایسه با ISR (چرا PPR بهتر است)**

```tsx
// با ISR: کل صفحه استاتیک می‌ماند یا کل صفحه پویا می‌شود
// با PPR: بهترین هر دو

export const experimental_ppr = true;

export default function ProductPage() {
  return (
    <div>
      {/* استاتیک: فوری از کش */}
      <ProductDetails />

      {/* پویا: استریم در همان درخواست */}
      <Suspense fallback={<RecommendationsSkeleton />}>
        <PersonalizedRecommendations />
      </Suspense>
    </div>
  );
}
```

**مثال ۳ — مرزهای Suspense دقیق**

```tsx
export const experimental_ppr = true;

export default function Dashboard() {
  return (
    <div>
      {/* پوستهٔ استاتیک */}
      <Sidebar />
      <Header />

      {/* بخش‌های پویا با اسکلتون */}
      <Suspense fallback={<StatsSkeleton />}>
        <LiveStats />
      </Suspense>

      <Suspense fallback={<TableSkeleton />}>
        <RecentActivity />
      </Suspense>
    </div>
  );
}
```

### مقایسه و الگوهای مختلف

**PPR در برابر روش‌های قبلی**

| معیار | SSR | ISR | PPR |
|---|---|---|---|
| سرعت اولیه | متوسط | سریع | ✅✅ فوری |
| دادهٔ شخصی | ✅ | ❌ | ✅ |
| هزینه | بالا | کم | ✅ کم |
| پیچیدگی | کم | متوسط | متوسط |

**چه زمانی کدام؟**

| سناریو | انتخاب |
|---|---|
| صفحهٔ کاملاً ثابت | SSG |
| محتوای دوره‌ای | ISR |
| صفحهٔ کاملاً شخصی | SSR |
| بیشتر استاتیک + بخش شخصی | ✅ PPR |

### اشکال شایع

**۱. فعال کردن PPR برای همهٔ مسیرها.**
ممکن است برای صفحات کاملاً پویا مزیتی نداشته باشد.

**۲. Suspense خیلی بزرگ.**
مرزها را دقیق کنید تا بخش‌های بیشتری فوری نمایش داده شوند.

**۳. فرض پایداری کامل PPR.**
PPR در حال تکامل است؛ رفتار ممکن است بین نسخه‌ها تغییر کند.

**۴. نادیده گرفتن سازگاری.**
برخی کتابخانه‌ها ممکن است با PPR مشکل داشته باشند.

**۵. تست نکردن روی Preview.**
قبل از Production، PPR را روی Preview بسنجید.

### نکات مهم (Tips & Tricks)

- PPR برای صفحات **اکثراً استاتیک با بخش پویا** عالی است.
- مرزهای **Suspense را دقیق** بگذارید.
- برای هر بخش پویا **اسکلتون** مناسب طراحی کنید.
- وضعیت PPR را در مستندات نسخهٔ خود بررسی کنید.
- قبل از Production روی Preview تست کنید.

### خلاصه فصل

- **PPR** پوستهٔ استاتیک فوری + بخش‌های پویای استریمی است.
- بهترین ترکیب سرعت و پویایی.
- مرزهای **Suspense** را دقیق کنید.
- مناسب صفحات «بیشتر استاتیک + بخش شخصی».
- وضعیت آن با نسخه‌ها تغییر می‌کند؛ مستندات را ببینید.

---

<a id="فصل-۴"></a>
## فصل ۴: revalidate، Cache Tags و On-Demand

### مقدمه

کنترل دقیق کش، تفاوت بین «دادهٔ قدیمی» و «دادهٔ تازه» است. در این فصل، ابزارهای Invalidation را کامل یاد می‌گیریم.

### مفاهیم اساسی

**۱. سه روش Invalidation**

| روش | ابزار | دقت |
|---|---|---|
| **زمانی** | `revalidate` | کم |
| **مسیری** | `revalidatePath('/blog')` | متوسط |
| **برچسبی** | `revalidateTag('products')` | ✅ زیاد |

**۲. Cache Tag چیست؟**
یک برچسب که به یک `fetch` می‌چسبانید. بعداً با آن برچسب، همهٔ کش‌های مرتبط را پاک می‌کنید.

**۳. `revalidatePath`**
همهٔ کش‌های یک مسیر را پاک می‌کند.

**۴. `revalidateTag`**
همهٔ کش‌هایی که آن برچسب را دارند پاک می‌کند — دقیق‌ترین روش.

**۵. تفاوت با `router.refresh()`**
`router.refresh()` فقط Router Cache مرورگر را تازه می‌کند، نه Data Cache سرور.

### مثال‌های عملی

**مثال ۱ — برچسب‌گذاری درست**

```tsx
// lib/data.ts
export async function getProducts() {
  const res = await fetch('https://api.example.com/products', {
    next: { revalidate: 3600, tags: ['products'] },
  });
  return res.json();
}

export async function getProduct(id: string) {
  const res = await fetch(`https://api.example.com/products/${id}`, {
    next: { revalidate: 3600, tags: ['products', `product:${id}`] },
  });
  return res.json();
}
```

**مثال ۲ — Invalidation برچسبی پس از تغییر CMS**

```ts
// app/api/webhook/cms/route.ts
import { revalidateTag } from 'next/cache';
import { NextResponse } from 'next/server';

export async function POST(request: Request) {
  const { type, id } = await request.json();

  // تأیید (ساده)
  const auth = request.headers.get('x-cms-secret');
  if (auth !== process.env.CMS_WEBHOOK_SECRET) {
    return NextResponse.json({ error: 'غیرمجاز' }, { status: 401 });
  }

  switch (type) {
    case 'product.updated':
      revalidateTag('products');           // لیست محصولات
      revalidateTag(`product:${id}`);      // صفحهٔ همان محصول
      break;

    case 'post.published':
      revalidateTag('posts');
      break;

    default:
      return NextResponse.json({ ignored: true });
  }

  return NextResponse.json({ revalidated: true, type, id });
}
```

**مثال ۳ — Invalidation مسیری**

```ts
// app/api/revalidate-path/route.ts
import { revalidatePath } from 'next/cache';
import { NextResponse } from 'next/server';

export async function POST(request: Request) {
  const { path } = await request.json();

  // پاک کردن یک مسیر خاص
  revalidatePath(path);

  // یا با نوع صفحه
  // revalidatePath('/blog/[slug]', 'page');

  return NextResponse.json({ revalidated: true, path });
}
```

**مثال ۴ — Invalidation کامل (همه چیز)**

```ts
// app/api/revalidate-all/route.ts
import { revalidatePath } from 'next/cache';

export async function POST() {
  // پاک کردن همهٔ مسیرها
  revalidatePath('/', 'layout');
  return Response.json({ revalidated: 'all' });
}
```

**مثال ۵ — تازه‌سازی از سمت کلاینت**

```tsx
// components/RefreshButton.tsx
'use client';

import { useRouter } from 'next/navigation';
import { useState } from 'react';

export default function RefreshButton() {
  const router = useRouter();
  const [loading, setLoading] = useState(false);

  async function handleRefresh() {
    setLoading(true);
    // ابتدا کش سرور را پاک کن
    await fetch('/api/revalidate?tag=products&secret=' + process.env.NEXT_PUBLIC_SECRET, {
      method: 'POST',
    });
    // سپس Router Cache مرورگر را تازه کن
    router.refresh();
    setLoading(false);
  }

  return <button onClick={handleRefresh} disabled={loading}>{loading ? '...' : 'تازه‌سازی'}</button>;
}
```

### مقایسه و الگوهای مختلف

**سه روش Invalidation در عمل**

| روش | دقت | سرعت | مناسب برای |
|---|---|---|---|
| **`revalidate`** | کم | خودکار | محتوای دوره‌ای |
| **`revalidatePath`** | متوسط | سریع | تغییر یک صفحه |
| **`revalidateTag`** | ✅ زیاد | سریع | CMS، دادهٔ مرتبط |

**استراتژی برچسب‌گذاری**

| الگو | مثال | کاربرد |
|---|---|---|
| **موجودیت** | `products`, `posts` | لیست‌ها |
| **تک‌آیتم** | `product:123` | صفحهٔ یک آیتم |
| **دسته** | `category:tech` | دسته‌بندی |
| **کاربر** | `user:456` | دادهٔ کاربری |

### اشکال شایع

**۱. نبود برچسب.**
بازسازی انتخابی ممکن نیست و باید همه چیز را پاک کنید.

**۲. برچسب‌های بیش از حد ریز.**
مدیریت سخت می‌شود. تعادل داشته باشید.

**۳. اشتباه بین `router.refresh()` و `revalidateTag`.**
اولی فقط مرورگر، دومی کش سرور را پاک می‌کند. معمولاً به **هر دو** نیاز دارید.

**۴. فراموش کردن تأیید امنیتی در endpoint بازسازی.**
هر کسی می‌تواند کش را پاک کند → حملهٔ DoS.

**۵. Invalidation زیاد.**
اگر هر ثانیه پاک کنید، مزیت کش از بین می‌رود.

### نکات مهم (Tips & Tricks)

- **الگوی برچسب** منظم بسازید (`entity:id`).
- endpoint بازسازی را با **secret** محافظت کنید.
- برای CMS، از **Webhook + revalidateTag** استفاده کنید.
- `revalidatePath('/', 'layout')` همه چیز را پاک می‌کند.
- معمولاً `revalidateTag` + `router.refresh()` با هم لازم است.

### خلاصه فصل

- سه روش: **زمانی، مسیری، برچسبی**.
- **Cache Tag** دقیق‌ترین روش Invalidation است.
- `revalidateTag` کش سرور، `router.refresh()` کش مرورگر را تازه می‌کند.
- endpoint بازسازی را با secret محافظت کنید.
- برچسب‌گذاری منظم، کلید مدیریت کش است.

---

<a id="فصل-۵"></a>
## فصل ۵: الگوهای ترکیبی و اشکالات رایج

### مقدمه

در این فصل، همهٔ ابزارها را در الگوهای واقعی ترکیب می‌کنیم و اشکالات رایج را با راه‌حل نشان می‌دهیم.

### الگو ۱ — وبلاگ با ISR + On-Demand

```tsx
// app/blog/[slug]/page.tsx
export const revalidate = 86400;   // پشتیبان: هر روز

export async function generateStaticParams() {
  const posts = await getPosts();
  return posts.map((p) => ({ slug: p.slug }));
}

export default async function Post({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;
  const post = await fetch(`https://cms.example.com/posts/${slug}`, {
    next: { tags: [`post:${slug}`, 'posts'] },
  }).then((r) => r.json());

  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.html }} />
    </article>
  );
}
```

```ts
// app/api/cms-webhook/route.ts — انتشار در CMS → بازسازی
import { revalidateTag } from 'next/cache';

export async function POST(request: Request) {
  const body = await request.json();
  if (request.headers.get('x-secret') !== process.env.CMS_SECRET) {
    return new Response('unauthorized', { status: 401 });
  }

  revalidateTag('posts');
  revalidateTag(`post:${body.slug}`);
  return Response.json({ ok: true });
}
```

### الگو ۲ — فروشگاه با PPR

```tsx
// app/products/[id]/page.tsx
import { Suspense } from 'react';
import { cookies } from 'next/headers';

export const experimental_ppr = true;
export const revalidate = 300;

// استاتیک: جزئیات محصول
async function ProductInfo({ id }: { id: string }) {
  const product = await fetch(`https://api.example.com/products/${id}`, {
    next: { revalidate: 300, tags: [`product:${id}`] },
  }).then((r) => r.json());

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.price}</p>
    </div>
  );
}

// پویا: سبد خرید کاربر
async function UserCart() {
  const cookieStore = await cookies();
  const cartId = cookieStore.get('cartId')?.value;
  const cart = await getCart(cartId);
  return <div>سبد: {cart.count} آیتم</div>;
}

export default async function ProductPage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  return (
    <main>
      <ProductInfo id={id} />
      <Suspense fallback={<div>...</div>}>
        <UserCart />
      </Suspense>
    </main>
  );
}
```

### الگو ۳ — داشبورد با کش لایه‌ای

```ts
// api/dashboard/route.ts
import { kv } from '@vercel/kv';
import { sql } from '@vercel/postgres';

export async function GET(request: Request) {
  const userId = request.headers.get('x-user-id');
  if (!userId) return Response.json({ error: 'unauthorized' }, { status: 401 });

  const cacheKey = `dashboard:${userId}`;

  // لایه ۱: KV
  const cached = await kv.get(cacheKey);
  if (cached) {
    return Response.json(cached, {
      headers: { 'x-cache': 'hit' },
    });
  }

  // لایه ۲: دیتابیس
  const { rows } = await sql`
    SELECT * FROM orders WHERE user_id = ${userId} ORDER BY created_at DESC LIMIT 10
  `;

  const result = { orders: rows, generatedAt: Date.now() };

  // ذخیره با TTL کوتاه
  await kv.set(cacheKey, result, { ex: 30 });

  return Response.json(result, { headers: { 'x-cache': 'miss' } });
}
```

### الگو ۴ — کش سلسله‌مراتبی

```
Router Cache (مرورگر)
   ↓ miss
Full Route Cache (لبه)     ← استاتیک/ISR/PPR
   ↓ miss
Data Cache (لبه/سرور)      ← fetch با revalidate/tags
   ↓ miss
KV Cache (اپلیکیشن)        ← کش دستی
   ↓ miss
دیتابیس
```

### اشکالات رایج و راه‌حل

**۱. «چرا داده به‌روز نشد؟»**

```tsx
// ❌ مشکل: force-cache بدون revalidate
fetch(url, { cache: 'force-cache' });

// ✅ راه‌حل: revalidate + tag
fetch(url, { next: { revalidate: 60, tags: ['data'] } });
// و پس از تغییر:
revalidateTag('data');
```

**۲. «چرا صفحه پویا شد؟»**

```tsx
// ❌ استفادهٔ ناخواسته از cookies/searchParams
const cookieStore = await cookies();   // ← مسیر را Dynamic می‌کند

// ✅ اگر لازم نیست، استفاده نکنید؛ یا به کامپوننت جدا منتقل کنید
// و آن کامپوننت را در Suspense بگذارید (با PPR)
```

**۳. «چرا کش در Preview متفاوت است؟»**

```ts
// هر محیط کش مستقل دارد
// Preview و Production کش جداگانه دارند — طبیعی است
```

**۴. «چرا دادهٔ کاربر دیگران را می‌بینم؟»**

```tsx
// ❌ دادهٔ شخصی در Data Cache
fetch(`/api/me`, { cache: 'force-cache' });

// ✅ همیشه no-store برای دادهٔ شخصی
fetch(`/api/me`, { cache: 'no-store' });
// یا کلید کش را شامل شناسهٔ کاربر کنید
```

**۵. «چرا Invalidation کار نمی‌کند؟»**

```tsx
// ❌ برچسب ناهماهنگ
fetch(url, { next: { tags: ['products'] } });
revalidateTag('product');   // ← اشتباه تایپی!

// ✅ برچسب دقیقاً یکسان
revalidateTag('products');
```

### جدول تصمیم نهایی

| نوع داده | استراتژی | Invalidation |
|---|---|---|
| کاملاً ثابت | `force-cache` | هرگز |
| دوره‌ای (ساعت) | `revalidate: 3600` | زمانی + برچسبی |
| نیمه‌پویا (دقیقه) | `revalidate: 300` | برچسبی |
| فوری (ثانیه) | `revalidate: 30` | برچسبی |
| شخصی | `no-store` | — |
| لحظه‌ای (قیمت سهام) | `no-store` + CSR | — |

### نکات مهم (Tips & Tricks)

- **برچسب‌گذاری منظم** را از روز اول شروع کنید.
- برای CMS، همیشه **Webhook + revalidateTag**.
- PPR را برای صفحات «استاتیک + بخش شخصی» استفاده کنید.
- کش را **لایه‌ای** طراحی کنید (Router → Route → Data → KV).
- هنگام دیباگ کش، `console.log` و هدر `x-cache` بگذارید.

### خلاصه فصل

- الگوهای واقعی: **وبلاگ (ISR+On-Demand)، فروشگاه (PPR)، داشبورد (لایه‌ای)**.
- کش را **سلسله‌مراتبی** طراحی کنید.
- برچسب ناهماهنگ، رایج‌ترین دلیل «Invalidation کار نمی‌کند».
- دادهٔ شخصی هرگز کش نشود.
- جدول تصمیم را به خاطر بسپارید.

---

## منابع و مراجع

### مستندات رسمی
- Next.js Caching: <https://nextjs.org/docs/app/building-your-application/caching>
- ISR: <https://nextjs.org/docs/app/building-your-application/data-fetching/incremental-static-regeneration>
- Partial Prerendering: <https://nextjs.org/docs/app/building-your-application/rendering/partial-prerendering>
- revalidateTag / revalidatePath: <https://nextjs.org/docs/app/api-reference/functions/revalidateTag>
- Vercel CDN Caching: <https://vercel.com/docs/edge-network/caching>

### مفاهیم پیشنهادی برای مطالعه بیشتر
- HTTP Caching (MDN): <https://developer.mozilla.org/docs/Web/HTTP/Caching>
- stale-while-revalidate (web.dev): <https://web.dev/articles/stale-while-revalidate>

### گام بعدی
- [فایل ۱۲ — امنیت](./12-Security.md)
- [فایل ۱۷ — پروژه نهایی واقعی](./17-Final-Project.md)
