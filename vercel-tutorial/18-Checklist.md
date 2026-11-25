# ۱۸ — چک‌لیست‌های جامع

**مدت مطالعه**: ۳۰ دقیقه
**کاربرد**: مرجع سریع قبل از هر انتشار و در طول توسعه

> این فایل، چک‌لیست‌های عملی برای موقعیت‌های مختلف است. می‌توانید آن را به‌عنوان مرجع سریع در پروژه‌های واقعی استفاده کنید.

---

## فهرست

- [۱. چک‌لیست شروع پروژه](#۱-چکلیست-شروع-پروژه)
- [۲. چک‌لیست پیش از استقرار](#۲-چکلیست-پیش-از-استقرار)
- [۳. چک‌لیست امنیت](#۳-چکلیست-امنیت)
- [۴. چک‌لیست عملکرد](#۴-چکلیست-عملکرد)
- [۵. چک‌لیست SEO](#۵-چکلیست-seo)
- [۶. چک‌لیست دامنه و DNS](#۶-چکلیست-دامنه-و-dns)
- [۷. چک‌لیست API و توابع](#۷-چکلیست-api-و-توابع)
- [۸. چک‌لیست کش](#۸-چکلیست-کش)
- [۹. چک‌لیست هزینه](#۹-چکلیست-هزینه)
- [۱۰. چک‌لیست دیباگ](#۱۰-چکلیست-دیباگ)
- [۱۱. چک‌لیست انتشار Production](#۱۱-چکلیست-انتشار-production)
- [۱۲. چک‌لیست تیم و همکاری](#۱۲-چکلیست-تیم-و-همکاری)

---

## ۱. چک‌لیست شروع پروژه

- [ ] Node.js LTS نصب است
- [ ] Git نصب و پیکربندی شده
- [ ] حساب GitHub ساخته شده
- [ ] حساب Vercel ساخته شده (با GitHub)
- [ ] Vercel CLI نصب شده (`npm i -g vercel`)
- [ ] ویرایشگر کد با افزونه‌های لازم
- [ ] `.gitignore` استاندارد ساخته شده
- [ ] `.env.example` ساخته شده (بدون مقادیر)
- [ ] ساختار پوشه‌ها طراحی شده
- [ ] فریم‌ورک انتخاب شده (Next.js پیشنهادی)
- [ ] TypeScript فعال شده

**دستورات سریع:**

```bash
node -v && npm -v && git --version
npm i -g vercel
vercel login
```

---

## ۲. چک‌لیست پیش از استقرار

**کد و Build:**

- [ ] `npm run build` محلی موفق است
- [ ] `npm run lint` بدون خطا
- [ ] `npm audit` بررسی شده
- [ ] فایل‌های لازم Commit شده‌اند
- [ ] `git status` تمیز است
- [ ] مسیر ایمپورت‌ها دقیق (حساس به حروف)
- [ ] نسخهٔ Node در `engines` پین شده

**پیکربندی:**

- [ ] `vercel.json` معتبر است
- [ ] Framework Preset درست انتخاب شده
- [ ] Build Command و Output Directory درست
- [ ] Root Directory (برای Monorepo) تنظیم شده

**متغیرهای محیطی:**

- [ ] همهٔ متغیرهای لازم تعریف شده
- [ ] برای همهٔ محیط‌ها (Production/Preview/Development)
- [ ] کلیدهای حساس Sensitive علامت خورده‌اند
- [ ] کلید تست و تولید جدا هستند

```bash
npm run build
vercel env ls
```

---

## ۳. چک‌لیست امنیت

**Secrets:**

- [ ] هیچ کلیدی در کد نیست
- [ ] `.env` و `.env.local` در `.gitignore`
- [ ] `.env.example` وجود دارد
- [ ] Secret Scanning فعال است
- [ ] برنامهٔ چرخش کلید وجود دارد

**احراز هویت:**

- [ ] رمزها با **bcrypt/argon2** هش می‌شوند
- [ ] کوکی‌ها `HttpOnly; Secure; SameSite=Lax`
- [ ] JWT ها تاریخ انقضا دارند
- [ ] Rate Limiting روی auth
- [ ] پیام خطای auth یکسان است

**کنترل دسترسی:**

- [ ] بررسی مالکیت در همهٔ عملیات (IDOR)
- [ ] اصل کمترین دسترسی
- [ ] نقش‌ها تعریف شده
- [ ] مسیرهای admin محافظت‌شده

**ورودی/خروجی:**

- [ ] اعتبارسنجی سمت سرور (Zod)
- [ ] کوئری‌های پارامتری
- [ ] HTML پاک‌سازی می‌شود (DOMPurify)
- [ ] CORS محدود به دامنه‌های مجاز
- [ ] محدودیت حجم/نوع آپلود

**هدرها:**

- [ ] `Strict-Transport-Security`
- [ ] `X-Content-Type-Options: nosniff`
- [ ] `X-Frame-Options` یا `frame-ancestors`
- [ ] `Referrer-Policy`
- [ ] `Content-Security-Policy`

**Webhook/Cron:**

- [ ] امضای HMAC تأیید می‌شود
- [ ] `bodyParser: false` برای امضا
- [ ] Idempotency برای Webhook
- [ ] `CRON_SECRET` بررسی می‌شود

**Firewall:**

- [ ] قوانین برای مسیرهای حساس
- [ ] ربات‌های خوب مستثنا
- [ ] System Bypass برای health check
- [ ] Runbook حمله آماده

---

## ۴. چک‌لیست عملکرد

**تصویر:**

- [ ] از `next/image` استفاده می‌شود
- [ ] `width`/`height` یا `fill` مشخص
- [ ] `sizes` تعیین شده
- [ ] `priority` فقط برای تصویر LCP
- [ ] کیفیت ۷۵–۸۵
- [ ] دامنه‌های بیرونی در `remotePatterns`

**فونت:**

- [ ] از `next/font` استفاده می‌شود
- [ ] `display: 'swap'`
- [ ] حداکثر ۲–۳ وزن
- [ ] Subset مناسب

**JavaScript:**

- [ ] Server Components تا جای ممکن
- [ ] `"use client"` در پایین‌ترین سطح
- [ ] Code Splitting برای بخش‌های سنگین
- [ ] ایمپورت جزئی کتابخانه‌ها

**کش و شبکه:**

- [ ] `Cache-Control` مناسب روی API
- [ ] فایل‌های هش‌دار `immutable`
- [ ] ISR برای صفحات عمومی
- [ ] Region مناسب انتخاب شده
- [ ] دادهٔ شخصی `no-store`

**توابع:**

- [ ] بسته سبک (ایمپورت جزئی)
- [ ] Runtime مناسب (Node/Edge)
- [ ] Timeout روی fetch بیرونی
- [ ] Fluid Compute فعال

**اندازه‌گیری:**

- [ ] Speed Insights نصب
- [ ] Lighthouse اجرا شده
- [ ] بودجهٔ عملکرد تعیین شده
- [ ] موبایل تست شده

```bash
lighthouse https://myapp.com --output html
```

---

## ۵. چک‌لیست SEO

- [ ] `metadata` یا `generateMetadata` تعریف شده
- [ ] `title` و `description` یکتا برای هر صفحه
- [ ] Open Graph tags
- [ ] Twitter Card
- [ ] `sitemap.xml` تولید می‌شود
- [ ] `robots.txt` درست
- [ ] Canonical URL
- [ ] ساختار HTML معنایی (h1، h2)
- [ ] محتوای مهم در HTML اولیه (SSR/SSG)
- [ ] تصاویر با `alt`
- [ ] سرعت مطلوب (Core Web Vitals)
- [ ] HTTPS فعال

```ts
// app/sitemap.ts
import type { MetadataRoute } from 'next';

export default function sitemap(): MetadataRoute.Sitemap {
  return [
    { url: 'https://myapp.com', lastModified: new Date(), priority: 1 },
    { url: 'https://myapp.com/about', lastModified: new Date(), priority: 0.8 },
  ];
}
```

```ts
// app/robots.ts
import type { MetadataRoute } from 'next';

export default function robots(): MetadataRoute.Robots {
  return {
    rules: { userAgent: '*', allow: '/', disallow: '/dashboard/' },
    sitemap: 'https://myapp.com/sitemap.xml',
  };
}
```

---

## ۶. چک‌لیست دامنه و DNS

- [ ] دامنه اضافه شده در Vercel
- [ ] رکورد A برای apex
- [ ] رکورد CNAME برای www
- [ ] رکوردهای قدیمی حذف شده
- [ ] Cloudflare Proxy خاموش (اگر استفاده می‌کنید)
- [ ] TTL مناسب (۳۰۰)
- [ ] DNS منتشر شده (`dig`)
- [ ] SSL صادر شده
- [ ] ریدایرکت یک‌طرفه (apex ↔ www)
- [ ] هر دو دامنه تست شده
- [ ] HSTS فعال

```bash
dig myapp.com A +short
dig www.myapp.com CNAME +short
curl -I https://myapp.com
```

---

## ۷. چک‌لیست API و توابع

- [ ] متد HTTP بررسی می‌شود
- [ ] پاسخ `405` برای متد نامعتبر
- [ ] ورودی اعتبارسنجی می‌شود
- [ ] مدیریت خطا (`try/catch`)
- [ ] کد وضعیت درست (۲۰۰/۲۰۱/۴۰۰/۴۰۱/۴۰۴/۵۰۰)
- [ ] کلیدهای حساس سمت سرور
- [ ] Timeout روی فراخوانی بیرونی
- [ ] Runtime مناسب
- [ ] Region مناسب
- [ ] `maxDuration` و `memory` تنظیم شده (در صورت نیاز)
- [ ] CORS درست (اگر لازم)
- [ ] لاگ ساختاریافته
- [ ] دادهٔ حساس در لاگ نیست

---

## ۸. چک‌لیست کش

- [ ] استراتژی کش برای هر نوع داده مشخص
- [ ] دادهٔ شخصی `no-store`
- [ ] فایل‌های استاتیک `immutable`
- [ ] API های عمومی با `s-maxage`
- [ ] `stale-while-revalidate` برای تجربهٔ روان
- [ ] Cache Tag برای Invalidation
- [ ] Invalidation پس از تغییر داده
- [ ] `Vary: Origin` (اگر CORS دارد)
- [ ] Router Cache با `router.refresh()` تازه می‌شود
- [ ] کش با `x-vercel-cache` بررسی شده

**جدول مرجع:**

| نوع داده | استراتژی |
|---|---|
| کاملاً ثابت | `force-cache` |
| دوره‌ای (ساعت) | `revalidate: 3600` |
| نیمه‌پویا (دقیقه) | `revalidate: 300` |
| فوری (ثانیه) | `revalidate: 30` |
| شخصی | `no-store` |

---

## ۹. چک‌لیست هزینه

- [ ] Spend Cap تنظیم شده
- [ ] هشدار مصرف (۵۰٪، ۸۰٪، ۱۰۰٪)
- [ ] مصرف ماهانه بررسی می‌شود
- [ ] کش روی API های عمومی
- [ ] کیفیت تصویر بهینه
- [ ] وابستگی‌های سنگین حذف شده
- [ ] ربات‌های مضر فیلتر شده
- [ ] Cron محدود به ساعات لازم
- [ ] Runtime مناسب (Edge برای سبک)
- [ ] پلن مناسب انتخاب شده
- [ ] Image Optimization پایش می‌شود

```
Dashboard → Usage → بررسی نمودارها
Billing → Spend Management → تنظیم سقف
```

---

## ۱۰. چک‌لیست دیباگ

- [ ] نوع خطا مشخص شد (Build/Runtime)
- [ ] Build Logs بررسی شد (اگر Build Fail)
- [ ] Runtime Logs بررسی شد (اگر Runtime Error)
- [ ] آخرین خط خطا خوانده شد
- [ ] محلی بازتولید شد
- [ ] تفاوت محیط بررسی شد (Node، متغیرها)
- [ ] تغییرات اخیر Git مرور شد
- [ ] یک فرضیه آزمایش شد
- [ ] رفع و تأیید شد
- [ ] مستندسازی شد

**جدول تشخیص سریع:**

| نشانه | بررسی اول |
|---|---|
| Build Fail | Build Logs |
| ۵۰۰ | Runtime Logs |
| ۴۰۴ | مسیر/rewrite |
| CORS | Network tab |
| کندی | Speed Insights |
| دامنه | `dig` |
| کش | `x-vercel-cache` |

---

## ۱۱. چک‌لیست انتشار Production

**قبل از انتشار:**

- [ ] همهٔ تست‌ها موفق
- [ ] Preview بررسی و تأیید شده
- [ ] متغیرهای Production تنظیم
- [ ] مهاجرت دیتابیس (در صورت نیاز)
- [ ] برنامهٔ Rollback آماده
- [ ] زمان انتشار (کم‌ترافیک)

**هنگام انتشار:**

- [ ] Merge PR یا `vercel --prod`
- [ ] وضعیت استقرار بررسی
- [ ] Smoke Test روی سایت
- [ ] بررسی Core Web Vitals

**بعد از انتشار:**

- [ ] سایت کامل کار می‌کند
- [ ] لاگ‌ها بررسی شده
- [ ] Analytics سالم
- [ ] تیم مطلع شده
- [ ] برچسب نسخه (Git Tag)

**در صورت مشکل:**

```bash
vercel ls                          # یافتن نسخهٔ سالم
vercel promote <deployment-url>    # بازگشت سریع
```

---

## ۱۲. چک‌لیست تیم و همکاری

- [ ] Branch Protection روی `main`
- [ ] بازبینی کد اجباری برای PR
- [ ] PR Template با چک‌لیست
- [ ] Preview Deployment برای هر PR
- [ ] Deployment Protection برای Preview
- [ ] نام‌گذاری منظم پروژه‌ها
- [ ] مستندسازی معماری
- [ ] نقش‌های تیم تعریف شده
- [ ] متغیرهای محیطی مستند
- [ ] Runbook برای حوادث

**PR Template نمونه:**

```markdown
## تغییرات
- ...

## چک‌لیست
- [ ] Build محلی موفق
- [ ] تست‌ها موفق
- [ ] متغیرهای محیطی جدید مستند شده
- [ ] Preview بررسی شده
- [ ] چک‌لیست امنیت مرور شده
```

---

## چک‌لیست نهایی (یک صفحه‌ای)

**قبل از هر انتشار، این ۱۰ مورد را بررسی کنید:**

1. [ ] Build محلی موفق
2. [ ] متغیرهای محیطی کامل
3. [ ] چک‌لیست امنیت مرور شده
4. [ ] تصاویر بهینه
5. [ ] کش مناسب
6. [ ] لاگ ساختاریافته
7. [ ] Preview تأیید شده
8. [ ] برنامهٔ Rollback
9. [ ] Spend Cap تنظیم
10. [ ] تیم مطلع

---

## گام بعدی

- [فایل ۱۹ — واژه‌نامه](./19-Glossary.md)
- [فایل ۲۰ — منابع و مراجع](./20-Resources.md)
- [فایل ۱۷ — پروژه نهایی واقعی](./17-Final-Project.md)
