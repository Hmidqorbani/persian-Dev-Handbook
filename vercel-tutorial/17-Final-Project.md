# ۱۷ — پروژه نهایی واقعی

**مدت یادگیری**: ۶ تا ۸ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: مطالعه فایل‌های ۰۱ تا ۱۶

> در این فایل، همهٔ آموخته‌ها را در یک پروژهٔ واقعی و کامل به کار می‌بریم: **«بوکمارک‌منیجر با اشتراک‌گذاری عمومی»**.

---

## فهرست محتوا

- [فصل ۱: معرفی پروژه و طراحی معماری](#فصل-۱)
- [فصل ۲: پیاده‌سازی فرانت‌اند](#فصل-۲)
- [فصل ۳: بک‌اند — API، دیتابیس و احراز هویت](#فصل-۳)
- [فصل ۴: استقرار، دامنه و متغیرهای محیطی](#فصل-۴)
- [فصل ۵: بهینه‌سازی، امنیت و مانیتورینگ](#فصل-۵)
- [فصل ۶: جمع‌بندی و توسعه‌های آینده](#فصل-۶)

---

<a id="فصل-۱"></a>
## فصل ۱: معرفی پروژه و طراحی معماری

### مقدمه

**Linkly** یک بوکمارک‌منیجر است: کاربران لینک ذخیره می‌کنند، در مجموعه (Collection) سازمان می‌دهند و می‌توانند مجموعه‌ها را عمومی کنند. این پروژه **همهٔ** مفاهیم کتابچه را پوشش می‌دهد.

### قابلیت‌های پروژه

| قابلیت | فناوری |
|---|---|
| ثبت‌نام/ورود | Postgres + bcrypt + KV (session) |
| ذخیره لینک | Postgres |
| مجموعه‌ها | Postgres (رابطه) |
| آپلود تصویر | Blob |
| کش فید عمومی | KV |
| محافظت از مسیر | Middleware |
| بررسی لینک شکسته | Cron |
| صفحات عمومی ISR | Next.js ISR |
| تحلیل و پایش | Analytics + Speed Insights |
| امنیت | Firewall + Rate Limiting |

### معماری

```
┌─────────────────────────────────────────────────┐
│                 Vercel Edge Network              │
├─────────────────────────────────────────────────┤
│  Middleware      → احراز هویت، هدرهای امنیتی     │
│  Edge Config     → Feature Flags                │
├─────────────────────────────────────────────────┤
│  Next.js (App Router)                           │
│   ├── Server Components  → رندر صفحات           │
│   ├── Route Handlers     → API                  │
│   └── ISR                → مجموعه‌های عمومی     │
├─────────────────────────────────────────────────┤
│  Postgres  → کاربران، لینک‌ها، مجموعه‌ها         │
│  KV        → نشست، کش، Rate Limit               │
│  Blob      → تصاویر آپلودی                      │
├─────────────────────────────────────────────────┤
│  Cron      → بررسی لینک‌های شکسته               │
│  Analytics → رفتار کاربران                      │
│  Speed Insights → عملکرد                        │
└─────────────────────────────────────────────────┘
```

### ساختار پوشه‌ها

```
linkly/
├── app/
│   ├── layout.tsx
│   ├── page.tsx                    # صفحه اصلی
│   ├── login/page.tsx
│   ├── signup/page.tsx
│   ├── dashboard/
│   │   ├── layout.tsx              # چیدمان محافظت‌شده
│   │   ├── page.tsx
│   │   ├── links/page.tsx
│   │   └── collections/page.tsx
│   ├── u/[username]/
│   │   └── [slug]/page.tsx         # مجموعهٔ عمومی (ISR)
│   └── api/
│       ├── auth/
│       │   ├── signup/route.ts
│       │   ├── login/route.ts
│       │   └── logout/route.ts
│       ├── links/route.ts
│       ├── collections/route.ts
│       ├── upload/route.ts
│       ├── revalidate/route.ts
│       └── cron/check-links/route.ts
├── components/
│   ├── LinkCard.tsx
│   ├── CollectionForm.tsx
│   └── Header.tsx
├── lib/
│   ├── db.ts                       # اتصال Postgres
│   ├── auth.ts                     # توابع احراز هویت
│   ├── session.ts                  # مدیریت نشست با KV
│   ├── rateLimit.ts
│   └── logger.ts
├── middleware.ts
├── vercel.json
└── package.json
```

### مدل داده

```sql
-- users
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  username VARCHAR(50) UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  avatar_url TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- collections
CREATE TABLE collections (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  title VARCHAR(200) NOT NULL,
  slug VARCHAR(200) NOT NULL,
  description TEXT,
  is_public BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, slug)
);

-- links
CREATE TABLE links (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  collection_id INTEGER REFERENCES collections(id) ON DELETE SET NULL,
  url TEXT NOT NULL,
  title VARCHAR(300),
  description TEXT,
  image_url TEXT,
  is_alive BOOLEAN DEFAULT true,
  last_checked TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_links_user ON links(user_id);
CREATE INDEX idx_links_collection ON links(collection_id);
CREATE INDEX idx_collections_public ON collections(is_public, user_id);
```

### نکات مهم فصل

- ابتدا **معماری و مدل داده** را طراحی کنید.
- هر سرویس نقش مشخصی دارد (فایل ۰۸).
- **امنیت** از ابتدا در طراحی لحاظ شود.
- ساختار پوشه را **منظم** نگه دارید.

### خلاصه فصل

- پروژه: **بوکمارک‌منیجر با اشتراک‌گذاری عمومی**.
- معماری چندلایه با Postgres، KV، Blob و Edge.
- مدل داده: **users، collections، links**.
- ایندکس روی ستون‌های پرجست‌وجو.
- امنیت از ابتدا در طراحی.

---

<a id="فصل-۲"></a>
## فصل ۲: پیاده‌سازی فرانت‌اند

### مقدمه

در این فصل، رابط کاربری را با Next.js App Router می‌سازیم.

### راه‌اندازی پروژه

```bash
npx create-next-app@latest linkly --typescript --tailwind --app --eslint
cd linkly

# نصب وابستگی‌ها
npm i @vercel/postgres @vercel/kv @vercel/blob bcryptjs zod
npm i -D @types/bcryptjs
```

### چیدمان ریشه

```tsx
// app/layout.tsx
import type { Metadata } from 'next';
import { Vazirmatn } from 'next/font/google';
import { Analytics } from '@vercel/analytics/next';
import { SpeedInsights } from '@vercel/speed-insights/next';
import './globals.css';

const vazir = Vazirmatn({ subsets: ['arabic'], display: 'swap' });

export const metadata: Metadata = {
  title: 'Linkly — بوکمارک‌منیجر',
  description: 'لینک‌های خود را ذخیره، سازمان‌دهی و به اشتراک بگذارید',
};

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="fa" dir="rtl" className={vazir.className}>
      <body className="min-h-screen bg-gray-50">
        {children}
        <Analytics />
        <SpeedInsights />
      </body>
    </html>
  );
}
```

### صفحهٔ اصلی

```tsx
// app/page.tsx
import Link from 'next/link';

export default function HomePage() {
  return (
    <main className="container mx-auto px-4 py-16">
      <section className="text-center max-w-2xl mx-auto">
        <h1 className="text-5xl font-bold mb-6">
          لینک‌هایتان را هوشمند مدیریت کنید
        </h1>
        <p className="text-xl text-gray-600 mb-8">
          ذخیره، سازمان‌دهی و اشتراک‌گذاری لینک‌ها — ساده و سریع
        </p>
        <div className="flex gap-4 justify-center">
          <Link href="/signup" className="btn-primary">شروع رایگان</Link>
          <Link href="/login" className="btn-secondary">ورود</Link>
        </div>
      </section>
    </main>
  );
}
```

### چیدمان محافظت‌شدهٔ داشبورد

```tsx
// app/dashboard/layout.tsx
import { redirect } from 'next/navigation';
import { getSession } from '@/lib/session';
import Header from '@/components/Header';

export default async function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const session = await getSession();

  // محافظت لایه دوم (علاوه بر Middleware)
  if (!session) redirect('/login');

  return (
    <div className="min-h-screen">
      <Header user={session} />
      <main className="container mx-auto px-4 py-8">{children}</main>
    </div>
  );
}
```

### کامپوننت کارت لینک

```tsx
// components/LinkCard.tsx
'use client';

import { useState } from 'react';

type LinkItem = {
  id: number;
  url: string;
  title: string;
  description?: string;
  image_url?: string;
  is_alive: boolean;
};

export default function LinkCard({ link }: { link: LinkItem }) {
  const [deleting, setDeleting] = useState(false);

  async function handleDelete() {
    if (!confirm('این لینک حذف شود؟')) return;
    setDeleting(true);

    await fetch(`/api/links?id=${link.id}`, { method: 'DELETE' });

    // تازه‌سازی کش و UI
    window.location.reload();
  }

  return (
    <article className="card flex gap-4">
      {link.image_url && (
        // eslint-disable-next-line @next/next/no-img-element
        <img
          src={link.image_url}
          alt=""
          width={80}
          height={80}
          className="rounded-lg object-cover"
        />
      )}

      <div className="flex-1">
        <h3 className="font-semibold">
          {link.title || link.url}
          {!link.is_alive && (
            <span className="text-red-500 text-sm mr-2">(لینک شکسته)</span>
          )}
        </h3>

        {link.description && (
          <p className="text-gray-600 text-sm mt-1">{link.description}</p>
        )}

        <a
          href={link.url}
          target="_blank"
          rel="noopener noreferrer"
          className="text-blue-600 text-sm mt-2 inline-block"
        >
          {new URL(link.url).hostname} ↗
        </a>
      </div>

      <button
        onClick={handleDelete}
        disabled={deleting}
        className="text-red-500 text-sm self-start"
        aria-label="حذف لینک"
      >
        {deleting ? '...' : 'حذف'}
      </button>
    </article>
  );
}
```

### فرم افزودن لینک

```tsx
// components/AddLinkForm.tsx
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';

export default function AddLinkForm({ collections }: { collections: { id: number; title: string }[] }) {
  const router = useRouter();
  const [url, setUrl] = useState('');
  const [title, setTitle] = useState('');
  const [collectionId, setCollectionId] = useState<string>('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setError('');
    setLoading(true);

    const res = await fetch('/api/links', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        url,
        title,
        collectionId: collectionId ? Number(collectionId) : null,
      }),
    });

    const data = await res.json();
    setLoading(false);

    if (!res.ok) {
      setError(data.error ?? 'خطا در افزودن لینک');
      return;
    }

    setUrl('');
    setTitle('');
    router.refresh();   // ← تازه‌سازی Server Component
  }

  return (
    <form onSubmit={handleSubmit} className="card space-y-4">
      <input
        type="url"
        value={url}
        onChange={(e) => setUrl(e.target.value)}
        placeholder="https://example.com"
        required
        className="input"
        dir="ltr"
      />

      <input
        type="text"
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="عنوان (اختیاری)"
        className="input"
      />

      <select
        value={collectionId}
        onChange={(e) => setCollectionId(e.target.value)}
        className="input"
      >
        <option value="">بدون مجموعه</option>
        {collections.map((c) => (
          <option key={c.id} value={c.id}>{c.title}</option>
        ))}
      </select>

      {error && <p className="text-red-500 text-sm">{error}</p>}

      <button type="submit" disabled={loading} className="btn-primary">
        {loading ? 'در حال افزودن...' : 'افزودن لینک'}
      </button>
    </form>
  );
}
```

### صفحهٔ مجموعهٔ عمومی (ISR)

```tsx
// app/u/[username]/[slug]/page.tsx
import { notFound } from 'next/navigation';
import { sql } from '@vercel/postgres';
import { Metadata } from 'next';

export const revalidate = 300;   // ISR: هر ۵ دقیقه

async function getCollection(username: string, slug: string) {
  const { rows } = await sql`
    SELECT c.id, c.title, c.description, u.username, u.avatar_url
    FROM collections c
    JOIN users u ON u.id = c.user_id
    WHERE u.username = ${username} AND c.slug = ${slug} AND c.is_public = true
  `;
  return rows[0] ?? null;
}

export async function generateMetadata({
  params,
}: {
  params: Promise<{ username: string; slug: string }>;
}): Promise<Metadata> {
  const { username, slug } = await params;
  const collection = await getCollection(username, slug);

  if (!collection) return { title: 'پیدا نشد' };

  return {
    title: `${collection.title} — ${username}`,
    description: collection.description ?? 'مجموعه لینک‌های عمومی',
    openGraph: { title: collection.title, description: collection.description },
  };
}

export default async function PublicCollection({
  params,
}: {
  params: Promise<{ username: string; slug: string }>;
}) {
  const { username, slug } = await params;
  const collection = await getCollection(username, slug);

  if (!collection) notFound();

  const { rows: links } = await sql`
    SELECT id, url, title, description, image_url
    FROM links
    WHERE collection_id = ${collection.id}
    ORDER BY created_at DESC
    LIMIT 100
  `;

  return (
    <main className="container mx-auto px-4 py-12">
      <header className="mb-8">
        <h1 className="text-3xl font-bold">{collection.title}</h1>
        {collection.description && (
          <p className="text-gray-600 mt-2">{collection.description}</p>
        )}
        <p className="text-sm text-gray-500 mt-2">توسط @{collection.username}</p>
      </header>

      <div className="grid gap-4 md:grid-cols-2">
        {links.map((link: any) => (
          <a
            key={link.id}
            href={link.url}
            target="_blank"
            rel="noopener noreferrer"
            className="card hover:shadow-lg transition"
          >
            <h3 className="font-semibold">{link.title || link.url}</h3>
            {link.description && (
              <p className="text-gray-600 text-sm mt-1">{link.description}</p>
            )}
          </a>
        ))}
      </div>
    </main>
  );
}
```

### نکات مهم فصل

- از **Server Components** برای داده استفاده کنید.
- **Client Components** فقط برای تعامل.
- `router.refresh()` برای تازه‌سازی پس از تغییر.
- **ISR** برای صفحات عمومی.
- `generateMetadata` برای SEO.

### خلاصه فصل

- ساختار: `app/`، `components/`، `lib/`.
- Server Components برای داده، Client برای تعامل.
- صفحات عمومی با **ISR**.
- **Metadata API** برای SEO.
- `router.refresh()` برای تازه‌سازی UI.

---

<a id="فصل-۳"></a>
## فصل ۳: بک‌اند — API، دیتابیس و احراز هویت

### مقدمه

در این فصل، بک‌اند کامل را می‌سازیم: اتصال دیتابیس، احراز هویت امن، API های CRUD و آپلود فایل.

### اتصال دیتابیس

```ts
// lib/db.ts
import { sql } from '@vercel/postgres';

export { sql };

// راه‌اندازی اولیه (یک بار)
export async function setupDatabase() {
  await sql`
    CREATE TABLE IF NOT EXISTS users (
      id SERIAL PRIMARY KEY,
      email VARCHAR(255) UNIQUE NOT NULL,
      username VARCHAR(50) UNIQUE NOT NULL,
      password_hash TEXT NOT NULL,
      avatar_url TEXT,
      created_at TIMESTAMP DEFAULT NOW()
    )
  `;

  await sql`
    CREATE TABLE IF NOT EXISTS collections (
      id SERIAL PRIMARY KEY,
      user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
      title VARCHAR(200) NOT NULL,
      slug VARCHAR(200) NOT NULL,
      description TEXT,
      is_public BOOLEAN DEFAULT false,
      created_at TIMESTAMP DEFAULT NOW(),
      UNIQUE(user_id, slug)
    )
  `;

  await sql`
    CREATE TABLE IF NOT EXISTS links (
      id SERIAL PRIMARY KEY,
      user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
      collection_id INTEGER REFERENCES collections(id) ON DELETE SET NULL,
      url TEXT NOT NULL,
      title VARCHAR(300),
      description TEXT,
      image_url TEXT,
      is_alive BOOLEAN DEFAULT true,
      last_checked TIMESTAMP,
      created_at TIMESTAMP DEFAULT NOW()
    )
  `;

  await sql`CREATE INDEX IF NOT EXISTS idx_links_user ON links(user_id)`;
  await sql`CREATE INDEX IF NOT EXISTS idx_links_collection ON links(collection_id)`;
}
```

### مدیریت نشست با KV

```ts
// lib/session.ts
import { kv } from '@vercel/kv';
import { cookies } from 'next/headers';
import crypto from 'crypto';

const SESSION_TTL = 60 * 60 * 24 * 7;   // ۷ روز

export type Session = {
  userId: number;
  email: string;
  username: string;
};

export async function createSession(user: Session): Promise<string> {
  const sessionId = crypto.randomUUID();
  await kv.set(`session:${sessionId}`, user, { ex: SESSION_TTL });
  return sessionId;
}

export async function getSession(): Promise<Session | null> {
  const cookieStore = await cookies();
  const sessionId = cookieStore.get('session')?.value;
  if (!sessionId) return null;

  const session = await kv.get<Session>(`session:${sessionId}`);

  // تمدید خودکار (Sliding expiration)
  if (session) {
    await kv.expire(`session:${sessionId}`, SESSION_TTL);
  }

  return session ?? null;
}

export async function destroySession(): Promise<void> {
  const cookieStore = await cookies();
  const sessionId = cookieStore.get('session')?.value;
  if (sessionId) {
    await kv.del(`session:${sessionId}`);
  }
}
```

### احراز هویت

```ts
// lib/auth.ts
import bcrypt from 'bcryptjs';
import { sql } from './db';

export async function hashPassword(password: string): Promise<string> {
  return bcrypt.hash(password, 12);
}

export async function verifyPassword(password: string, hash: string): Promise<boolean> {
  return bcrypt.compare(password, hash);
}

export async function findUserByEmail(email: string) {
  const { rows } = await sql`SELECT * FROM users WHERE email = ${email.toLowerCase()}`;
  return rows[0] ?? null;
}

export async function findUserByUsername(username: string) {
  const { rows } = await sql`SELECT * FROM users WHERE username = ${username.toLowerCase()}`;
  return rows[0] ?? null;
}

export async function createUser(email: string, username: string, password: string) {
  const hash = await hashPassword(password);
  const { rows } = await sql`
    INSERT INTO users (email, username, password_hash)
    VALUES (${email.toLowerCase()}, ${username.toLowerCase()}, ${hash})
    RETURNING id, email, username
  `;
  return rows[0];
}
```

### API ثبت‌نام

```ts
// app/api/auth/signup/route.ts
import { NextResponse } from 'next/server';
import { z } from 'zod';
import { createUser, findUserByEmail, findUserByUsername } from '@/lib/auth';
import { createSession } from '@/lib/session';
import { rateLimit } from '@/lib/rateLimit';

const schema = z.object({
  email: z.string().email('ایمیل نامعتبر'),
  username: z.string().min(3).max(50).regex(/^[a-z0-9_]+$/, 'فقط حروف کوچک، عدد و _'),
  password: z.string().min(8, 'رمز باید حداقل ۸ کاراکتر باشد'),
});

export async function POST(request: Request) {
  const ip = request.headers.get('x-forwarded-for')?.split(',')[0] ?? 'unknown';

  const { allowed } = await rateLimit(`signup:${ip}`, { limit: 5, windowSec: 3600 });
  if (!allowed) {
    return NextResponse.json({ error: 'تلاش‌های زیاد. بعداً امتحان کنید.' }, { status: 429 });
  }

  const body = await request.json();
  const parsed = schema.safeParse(body);

  if (!parsed.success) {
    return NextResponse.json({ error: parsed.error.issues[0].message }, { status: 400 });
  }

  const { email, username, password } = parsed.data;

  if (await findUserByEmail(email)) {
    return NextResponse.json({ error: 'این ایمیل قبلاً ثبت شده' }, { status: 409 });
  }
  if (await findUserByUsername(username)) {
    return NextResponse.json({ error: 'این نام کاربری گرفته شده' }, { status: 409 });
  }

  const user = await createUser(email, username, password);
  const sessionId = await createSession({
    userId: user.id,
    email: user.email,
    username: user.username,
  });

  const response = NextResponse.json({ ok: true, user });
  response.cookies.set('session', sessionId, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    path: '/',
    maxAge: 60 * 60 * 24 * 7,
  });

  return response;
}
```

### API ورود

```ts
// app/api/auth/login/route.ts
import { NextResponse } from 'next/server';
import { z } from 'zod';
import { findUserByEmail, verifyPassword } from '@/lib/auth';
import { createSession } from '@/lib/session';
import { rateLimit } from '@/lib/rateLimit';

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(1),
});

export async function POST(request: Request) {
  const ip = request.headers.get('x-forwarded-for')?.split(',')[0] ?? 'unknown';
  const body = await request.json();
  const parsed = schema.safeParse(body);

  if (!parsed.success) {
    return NextResponse.json({ error: 'ورودی نامعتبر' }, { status: 400 });
  }

  const { email, password } = parsed.data;

  // محدودیت سخت‌گیرانه
  const byIp = await rateLimit(`login:ip:${ip}`, { limit: 10, windowSec: 300 });
  const byEmail = await rateLimit(`login:email:${email}`, { limit: 5, windowSec: 300 });

  if (!byIp.allowed || !byEmail.allowed) {
    return NextResponse.json({ error: 'تلاش‌های زیاد. بعداً امتحان کنید.' }, { status: 429 });
  }

  const user = await findUserByEmail(email);

  // پیام یکسان برای جلوگیری از افشای وجود کاربر
  if (!user || !(await verifyPassword(password, user.password_hash))) {
    return NextResponse.json({ error: 'ایمیل یا رمز نادرست' }, { status: 401 });
  }

  const sessionId = await createSession({
    userId: user.id,
    email: user.email,
    username: user.username,
  });

  const response = NextResponse.json({ ok: true });
  response.cookies.set('session', sessionId, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    path: '/',
    maxAge: 60 * 60 * 24 * 7,
  });

  return response;
}
```

### API لینک‌ها

```ts
// app/api/links/route.ts
import { NextResponse } from 'next/server';
import { z } from 'zod';
import { sql } from '@/lib/db';
import { getSession } from '@/lib/session';
import { kv } from '@vercel/kv';

const createSchema = z.object({
  url: z.string().url('آدرس نامعتبر'),
  title: z.string().max(300).optional(),
  collectionId: z.number().int().positive().nullable().optional(),
});

export async function GET() {
  const session = await getSession();
  if (!session) return NextResponse.json({ error: 'وارد نشده‌اید' }, { status: 401 });

  const { rows } = await sql`
    SELECT id, url, title, description, image_url, is_alive, collection_id, created_at
    FROM links
    WHERE user_id = ${session.userId}
    ORDER BY created_at DESC
    LIMIT 200
  `;

  return NextResponse.json({ links: rows });
}

export async function POST(request: Request) {
  const session = await getSession();
  if (!session) return NextResponse.json({ error: 'وارد نشده‌اید' }, { status: 401 });

  const body = await request.json();
  const parsed = createSchema.safeParse(body);

  if (!parsed.success) {
    return NextResponse.json({ error: parsed.error.issues[0].message }, { status: 400 });
  }

  const { url, title, collectionId } = parsed.data;

  // بررسی مالکیت مجموعه (جلوگیری از IDOR)
  if (collectionId) {
    const { rows } = await sql`
      SELECT id FROM collections WHERE id = ${collectionId} AND user_id = ${session.userId}
    `;
    if (!rows[0]) {
      return NextResponse.json({ error: 'مجموعه پیدا نشد' }, { status: 404 });
    }
  }

  const { rows } = await sql`
    INSERT INTO links (user_id, collection_id, url, title)
    VALUES (${session.userId}, ${collectionId ?? null}, ${url}, ${title ?? null})
    RETURNING id, url, title, created_at
  `;

  // پاک کردن کش فید
  await kv.del(`feed:${session.userId}`);

  return NextResponse.json({ link: rows[0] }, { status: 201 });
}

export async function DELETE(request: Request) {
  const session = await getSession();
  if (!session) return NextResponse.json({ error: 'وارد نشده‌اید' }, { status: 401 });

  const { searchParams } = new URL(request.url);
  const id = Number(searchParams.get('id'));

  if (!Number.isInteger(id)) {
    return NextResponse.json({ error: 'شناسه نامعتبر' }, { status: 400 });
  }

  // بررسی مالکیت (جلوگیری از IDOR)
  const { rowCount } = await sql`
    DELETE FROM links WHERE id = ${id} AND user_id = ${session.userId}
  `;

  if (rowCount === 0) {
    return NextResponse.json({ error: 'لینک پیدا نشد' }, { status: 404 });
  }

  await kv.del(`feed:${session.userId}`);
  return NextResponse.json({ ok: true });
}
```

### آپلود تصویر

```ts
// app/api/upload/route.ts
import { put } from '@vercel/blob';
import { getSession } from '@/lib/session';
import { NextResponse } from 'next/server';

export async function POST(request: Request) {
  const session = await getSession();
  if (!session) return NextResponse.json({ error: 'وارد نشده‌اید' }, { status: 401 });

  const formData = await request.formData();
  const file = formData.get('file') as File | null;

  if (!file) return NextResponse.json({ error: 'فایلی ارسال نشد' }, { status: 400 });

  // اعتبارسنجی نوع و حجم
  const allowed = ['image/jpeg', 'image/png', 'image/webp'];
  if (!allowed.includes(file.type)) {
    return NextResponse.json({ error: 'فقط JPG، PNG و WebP مجاز است' }, { status: 400 });
  }
  if (file.size > 5 * 1024 * 1024) {
    return NextResponse.json({ error: 'حجم فایل بیش از ۵ مگابایت است' }, { status: 400 });
  }

  const blob = await put(`avatars/${session.userId}-${Date.now()}`, file, {
    access: 'public',
    addRandomSuffix: true,
  });

  return NextResponse.json({ url: blob.url });
}
```

### Rate Limiting

```ts
// lib/rateLimit.ts
import { kv } from '@vercel/kv';

export async function rateLimit(
  identifier: string,
  { limit, windowSec }: { limit: number; windowSec: number }
) {
  const key = `rl:${identifier}`;
  const now = Date.now();
  const windowStart = now - windowSec * 1000;

  await kv.zadd(key, { score: now, member: `${now}:${Math.random()}` });
  await kv.zremrangebyscore(key, 0, windowStart);

  const count = await kv.zcard(key);
  await kv.expire(key, windowSec);

  return {
    allowed: count <= limit,
    remaining: Math.max(0, limit - count),
  };
}
```

### Middleware

```ts
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  const sessionId = request.cookies.get('session')?.value;

  const isProtected = pathname.startsWith('/dashboard');

  if (isProtected && !sessionId) {
    const loginUrl = new URL('/login', request.url);
    loginUrl.searchParams.set('next', pathname);
    return NextResponse.redirect(loginUrl);
  }

  // اگر لاگین است و در صفحه auth → داشبورد
  if (sessionId && (pathname === '/login' || pathname === '/signup')) {
    return NextResponse.redirect(new URL('/dashboard', request.url));
  }

  const response = NextResponse.next();
  response.headers.set('X-Content-Type-Options', 'nosniff');
  response.headers.set('X-Frame-Options', 'SAMEORIGIN');
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');

  return response;
}

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico|api|.*\\..*).*)'],
};
```

### نکات مهم فصل

- **Nشست در KV**، دادهٔ کاربر در Postgres.
- رمزها با **bcrypt (cost 12)**.
- **Rate Limiting** روی auth.
- **بررسی مالکیت** در همهٔ عملیات (IDOR).
- پیام خطای auth **یکسان**.

### خلاصه فصل

- نشست در **KV**، داده در **Postgres**.
- رمز با **bcrypt**، اعتبارسنجی با **Zod**.
- Rate Limiting روی auth.
- بررسی **مالکیت** در همهٔ عملیات.
- آپلود با **اعتبارسنجی نوع/حجم**.

---

<a id="فصل-۴"></a>
## فصل ۴: استقرار، دامنه و متغیرهای محیطی

### مقدمه

اکنون پروژه را روی Vercel مستقر می‌کنیم.

### `vercel.json`

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "framework": "nextjs",
  "regions": ["fra1"],
  "crons": [
    { "path": "/api/cron/check-links", "schedule": "0 3 * * *" }
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "X-Frame-Options", "value": "SAMEORIGIN" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
        { "key": "Strict-Transport-Security", "value": "max-age=63072000; includeSubDomains" }
      ]
    }
  ]
}
```

### متغیرهای محیطی

```bash
# در Vercel Dashboard → Settings → Environment Variables
POSTGRES_URL=postgres://...
KV_REST_API_URL=https://...
KV_REST_API_TOKEN=...
BLOB_READ_WRITE_TOKEN=vercel_blob_rw_...
CRON_SECRET=random-secret-string
REVALIDATE_SECRET=another-random-secret
```

> با نصب Integration های Marketplace (Postgres/KV/Blob)، این متغیرها **خودکار** ست می‌شوند.

### استقرار گام‌به‌گام

```bash
# ۱. ساخت مخزن و Push
git init
git add .
git commit -m "feat: initial commit"
git branch -M main
gh repo create linkly --public --source=. --push

# ۲. Import در Vercel
# Dashboard → Add New → Project → انتخاب مخزن → Deploy

# ۳. راه‌اندازی دیتابیس (یک بار)
curl -X POST https://linkly.vercel.app/api/setup

# ۴. افزودن دامنه
# Settings → Domains → اضافه کردن linkly.app
```

### Cron بررسی لینک‌های شکسته

```ts
// app/api/cron/check-links/route.ts
import { NextResponse } from 'next/server';
import { sql } from '@/lib/db';
import { logger } from '@/lib/logger';

export const maxDuration = 60;

export async function GET(request: Request) {
  // تأیید Cron
  if (request.headers.get('authorization') !== `Bearer ${process.env.CRON_SECRET}`) {
    return NextResponse.json({ error: 'غیرمجاز' }, { status: 401 });
  }

  // لینک‌هایی که ۷ روز بررسی نشده‌اند
  const { rows: links } = await sql`
    SELECT id, url FROM links
    WHERE last_checked IS NULL OR last_checked < NOW() - INTERVAL '7 days'
    LIMIT 50
  `;

  let checked = 0;
  let broken = 0;

  for (const link of links) {
    try {
      const controller = new AbortController();
      const timer = setTimeout(() => controller.abort(), 5000);

      const res = await fetch(link.url, {
        method: 'HEAD',
        signal: controller.signal,
        redirect: 'follow',
      });
      clearTimeout(timer);

      const alive = res.ok;

      await sql`
        UPDATE links SET is_alive = ${alive}, last_checked = NOW() WHERE id = ${link.id}
      `;

      checked++;
      if (!alive) broken++;
    } catch {
      await sql`
        UPDATE links SET is_alive = false, last_checked = NOW() WHERE id = ${link.id}
      `;
      checked++;
      broken++;
    }
  }

  logger.info('بررسی لینک‌ها کامل شد', { checked, broken });

  return NextResponse.json({ checked, broken });
}
```

### نکات مهم فصل

- Region را بر اساس **مخاطبان** انتخاب کنید.
- متغیرها را در **همهٔ محیط‌های لازم** تعریف کنید.
- Cron را با **`CRON_SECRET`** محافظت کنید.
- دیتابیس را **یک بار** راه‌اندازی کنید.
- دامنه را با **A/CNAME** وصل کنید.

### خلاصه فصل

- `vercel.json` شامل **regions، crons، headers**.
- متغیرها در سه محیط تعریف شوند.
- Integration ها متغیرها را **خودکار** ست می‌کنند.
- Cron بررسی لینک با `maxDuration` و `CRON_SECRET`.
- دامنه و SSL خودکار.

---

<a id="فصل-۵"></a>
## فصل ۵: بهینه‌سازی، امنیت و مانیتورینگ

### مقدمه

پروژه کار می‌کند — حالا آن را حرفه‌ای می‌کنیم.

### کش فید با KV

```ts
// app/api/feed/route.ts
import { NextResponse } from 'next/server';
import { sql } from '@/lib/db';
import { kv } from '@vercel/kv';
import { getSession } from '@/lib/session';

export async function GET() {
  const session = await getSession();
  if (!session) return NextResponse.json({ error: 'وارد نشده‌اید' }, { status: 401 });

  const cacheKey = `feed:${session.userId}`;

  const cached = await kv.get(cacheKey);
  if (cached) {
    return NextResponse.json({ source: 'cache', links: cached });
  }

  const { rows } = await sql`
    SELECT l.id, l.url, l.title, l.is_alive, c.title AS collection
    FROM links l
    LEFT JOIN collections c ON c.id = l.collection_id
    WHERE l.user_id = ${session.userId}
    ORDER BY l.created_at DESC
    LIMIT 50
  `;

  await kv.set(cacheKey, rows, { ex: 60 });

  return NextResponse.json({ source: 'db', links: rows });
}
```

### بازسازی On-Demand

```ts
// app/api/revalidate/route.ts
import { revalidatePath } from 'next/cache';
import { NextResponse } from 'next/server';
import { getSession } from '@/lib/session';

export async function POST(request: Request) {
  const session = await getSession();
  if (!session) return NextResponse.json({ error: 'غیرمجاز' }, { status: 401 });

  const { path } = await request.json();
  revalidatePath(path);

  return NextResponse.json({ revalidated: true });
}
```

### بهینه‌سازی تصاویر

```tsx
// components/Avatar.tsx
import Image from 'next/image';

export default function Avatar({ src, name }: { src?: string; name: string }) {
  if (!src) {
    return (
      <div className="w-10 h-10 rounded-full bg-blue-500 flex items-center justify-center text-white">
        {name[0]?.toUpperCase()}
      </div>
    );
  }

  return (
    <Image
      src={src}
      alt={name}
      width={40}
      height={40}
      className="rounded-full"
      sizes="40px"
      quality={80}
    />
  );
}
```

### Rate Limiting در Middleware

```ts
// middleware.ts — بخشی از فایل
// (در Production از Firewall استفاده کنید)
```

```
Firewall Rules:
۱. Rate limit روی /api/auth/* → ۱۰ درخواست در دقیقه
۲. Challenge روی درخواست‌های مشکوک
۳. Block روی مسیرهای حساس از کشورهای غیرمجاز (اختیاری)
```

### Logging ساختاریافته

```ts
// lib/logger.ts
type Level = 'info' | 'warn' | 'error';

export function log(level: Level, message: string, meta: Record<string, unknown> = {}) {
  console.log(JSON.stringify({
    level,
    message,
    timestamp: new Date().toISOString(),
    env: process.env.VERCEL_ENV,
    ...meta,
  }));
}

export const logger = {
  info: (m: string, meta?: Record<string, unknown>) => log('info', m, meta),
  warn: (m: string, meta?: Record<string, unknown>) => log('warn', m, meta),
  error: (m: string, meta?: Record<string, unknown>) => log('error', m, meta),
};
```

### تحلیل و پایش

```tsx
// app/layout.tsx — نصب ابزارها
import { Analytics } from '@vercel/analytics/next';
import { SpeedInsights } from '@vercel/speed-insights/next';

// ...
<Analytics />
<SpeedInsights />
```

### ردیابی رویداد

```tsx
// components/AddLinkForm.tsx — افزودن ردیابی
'use client';
import { track } from '@vercel/analytics';

// پس از موفقیت:
track('link_added', { hasCollection: Boolean(collectionId) });
```

### چک‌لیست امنیتی پروژه

- [ ] رمزها با **bcrypt (cost 12)**
- [ ] کوکی `HttpOnly; Secure; SameSite=Lax`
- [ ] **بررسی مالکیت** در همهٔ عملیات
- [ ] **Rate Limiting** روی auth
- [ ] اعتبارسنجی با **Zod**
- [ ] **هدرهای امنیتی** در `vercel.json`
- [ ] **Cron Secret** بررسی می‌شود
- [ ] آپلود با **اعتبارسنجی نوع/حجم**
- [ ] دادهٔ حساس در لاگ نیست
- [ ] `.env` در `.gitignore`
- [ ] Deployment Protection برای Preview

### چک‌لیست عملکرد

- [ ] تصویر با `next/image`
- [ ] فونت با `next/font`
- [ ] کش فید با **KV**
- [ ] **ISR** برای صفحات عمومی
- [ ] Region مناسب (`fra1`)
- [ ] **Analytics** و **Speed Insights** نصب
- [ ] لاگ **ساختاریافته**
- [ ] **Drains** برای نگهداری بلندمدت (اختیاری)

### نکات مهم فصل

- **کش لایه‌ای**: KV + CDN + ISR.
- **Invalidation** پس از هر تغییر.
- **امنیت** در همهٔ API ها.
- **پایش** از روز اول.
- **لاگ ساختاریافته** برای دیباگ.

### خلاصه فصل

- کش با **KV** + ISR.
- بازسازی **On-Demand** پس از تغییر.
- امنیت: bcrypt، مالکیت، Rate Limit، هدرها.
- عملکرد: تصویر، فونت، کش، Region.
- پایش: Analytics، Speed Insights، لاگ.

---

<a id="فصل-۶"></a>
## فصل ۶: جمع‌بندی و توسعه‌های آینده

### مقدمه

پروژه کامل شد! در این فصل، جمع‌بندی می‌کنیم و مسیرهای توسعه را می‌بینیم.

### چه چیزی ساختیم؟

| بخش | فناوری |
|---|---|
| فرانت‌اند | Next.js App Router + Server/Client Components |
| احراز هویت | Postgres + bcrypt + KV Session |
| API | Route Handlers + Zod |
| ذخیره‌سازی | Postgres + KV + Blob |
| محافظت | Middleware + Rate Limiting |
| اتوماسیون | Cron + On-Demand Revalidation |
| عملکرد | ISR + کش + Image/Font Optimization |
| پایش | Analytics + Speed Insights + Logs |
| امنیت | bcrypt + هدرها + بررسی مالکیت |
| استقرار | Git + Vercel + دامنه اختصاصی |

### مفاهیمی که به کار بردیم

```
✅ Git Push = Deploy
✅ Preview/Production محیط‌ها
✅ Serverless & Edge Functions
✅ Route Handlers
✅ Middleware
✅ Environment Variables
✅ دامنه + DNS + SSL
✅ Postgres + KV + Blob
✅ Cron Jobs
✅ ISR + کش لایه‌ای
✅ Image/Font Optimization
✅ Rate Limiting + Security Headers
✅ Analytics + Speed Insights
✅ Logging ساختاریافته
✅ Rolling/Region انتخاب
```

### توسعه‌های پیشنهادی

**۱. اشتراک‌گذاری اجتماعی**

```tsx
// افزودن OG Image تولیدی
// app/u/[username]/[slug]/opengraph-image.tsx
import { ImageResponse } from 'next/og';

export const size = { width: 1200, height: 630 };

export default async function OG({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;
  return new ImageResponse(
    (
      <div style={{ display: 'flex', fontSize: 60, background: '#fff', width: '100%', height: '100%', alignItems: 'center', justifyContent: 'center' }}>
        {slug}
      </div>
    ),
    size
  );
}
```

**۲. جست‌وجو با Algolia/Meilisearch**

```ts
// Integration از Marketplace نصب کنید
// سپس API جست‌وجو بسازید
```

**۳. ایمیل با Resend**

```ts
// app/api/cron/weekly-digest/route.ts
import { Resend } from 'resend';

const resend = new Resend(process.env.RESEND_API_KEY);

await resend.emails.send({
  from: 'Linkly <digest@linkly.app>',
  to: user.email,
  subject: 'خلاصه هفتگی لینک‌ها',
  html: generateDigestHtml(links),
});
```

**۴. اشتراک پرداختی با Stripe**

```ts
// Integration Stripe را نصب کنید
// پلن رایگان: ۱۰۰ لینک
// پلن Pro: نامحدود
```

**۵. PWA**

```ts
// app/manifest.ts
import type { MetadataRoute } from 'next';

export default function manifest(): MetadataRoute.Manifest {
  return {
    name: 'Linkly',
    short_name: 'Linkly',
    start_url: '/',
    display: 'standalone',
    background_color: '#ffffff',
    theme_color: '#3b82f6',
    icons: [{ src: '/icon-192.png', sizes: '192x192', type: 'image/png' }],
  };
}
```

**۶. Import از مرورگر**

```ts
// app/api/import/route.ts
// دریافت فایل HTML بوکمارک مرورگر و تبدیل به لینک‌ها
```

**۷. تیم و اشتراک‌گذاری گروهی**

```sql
CREATE TABLE team_members (
  team_id INTEGER REFERENCES teams(id),
  user_id INTEGER REFERENCES users(id),
  role VARCHAR(20) DEFAULT 'member',
  PRIMARY KEY (team_id, user_id)
);
```

### نقشه راه حرفه‌ای شدن

```
【 سطح ۱: پایه 】
✅ استقرار، دامنه، SSL
✅ احراز هویت
✅ CRUD

【 سطح ۲: حرفه‌ای 】
✅ کش لایه‌ای
✅ Cron و اتوماسیون
✅ امنیت کامل
✅ پایش

【 سطح ۳: مقیاس 】
- تست خودکار (E2E)
- CI/CD کامل
- Observability Plus
- Firewall پیشرفته
- تیم و نقش‌ها

【 سطح ۴: محصول 】
- پرداخت
- تحلیل کسب‌وکار
- A/B Testing
- پشتیبانی مشتری
```

### درس‌های کلیدی کتابچه

| درس | توضیح |
|---|---|
| **Git منبع حقیقت است** | همه چیز از Git می‌آید |
| **استاتیک تا جای ممکن** | سریع‌تر و ارزان‌تر |
| **کش را جدی بگیرید** | بزرگ‌ترین اهرم عملکرد |
| **امنیت از ابتدا** | بعداً گران است |
| **اندازه‌گیری کنید** | بدون داده، حدس است |
| **State را بیرون نگه دارید** | Serverless بی‌حالت است |
| **لایهٔ انتزاعی** | مهاجرت را ساده می‌کند |

### منابع ادامهٔ مسیر

- [فایل ۱۸ — چک‌لیست‌های جامع](./18-Checklist.md)
- [فایل ۱۹ — واژه‌نامه](./19-Glossary.md)
- [فایل ۲۰ — منابع و مراجع](./20-Resources.md)

### خلاصه فصل

- پروژهٔ کامل با **همهٔ قابلیت‌های کلیدی** Vercel ساخته شد.
- مسیرهای توسعه: OG Image، جست‌وجو، ایمیل، پرداخت، PWA، تیم.
- نقشهٔ راه چهار سطحی برای حرفه‌ای شدن.
- درس‌های کلیدی را به خاطر بسپارید.
- ادامهٔ مسیر: تست، CI/CD، مقیاس، محصول.

---

## منابع و مراجع

### مستندات رسمی
- Next.js Full-Stack Guide: <https://nextjs.org/docs/app/building-your-application>
- Vercel Templates: <https://vercel.com/templates>
- Vercel Examples: <https://github.com/vercel/examples>

### مفاهیم پیشنهادی برای مطالعه بیشتر
- Resend (ایمیل): <https://resend.com/docs>
- Stripe (پرداخت): <https://stripe.com/docs>
- Playwright (تست E2E): <https://playwright.dev>
- Next.js OG Image: <https://nextjs.org/docs/app/api-reference/file-conventions/opengraph-image>

### گام بعدی
- [فایل ۱۸ — چک‌لیست‌های جامع](./18-Checklist.md)
- [فایل ۲۰ — منابع و مراجع](./20-Resources.md)
