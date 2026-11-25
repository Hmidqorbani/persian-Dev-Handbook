# ۱۹ — واژه‌نامه

**کاربرد**: مرجع سریع اصطلاحات Vercel و وب مدرن

> این واژه‌نامه شامل اصطلاحات کلیدی کتابچه است. برای هر اصطلاح، معنی فارسی، توضیح کوتاه و محل یادگیری آورده شده است.

---

## فهرست الفبایی

[A](#a) · [B](#b) · [C](#c) · [D](#d) · [E](#e) · [F](#f) · [G](#g) · [H](#h) · [I](#i) · [J](#j) · [K](#k) · [L](#l) · [M](#m) · [N](#n) · [O](#o) · [P](#p) · [Q](#q) · [R](#r) · [S](#s) · [T](#t) · [U](#u) · [V](#v) · [W](#w) · [X](#x) · [Z](#z)

---

## A

**Alias (نام مستعار)**
آدرس جایگزین برای یک استقرار. مثلاً `staging.myapp.com` که همیشه به آخرین استقرار Preview اشاره می‌کند.
📁 فایل ۰۳

**Apex Domain (دامنهٔ ریشه)**
دامنهٔ اصلی بدون زیردامنه، مثل `myapp.com`. با رکورد **A** تنظیم می‌شود.
📁 فایل ۰۷

**App Router**
سیستم مسیریابی مدرن Next.js که از پوشهٔ `app/` استفاده می‌کند و Server Components را پشتیبانی می‌کند.
📁 فایل ۰۴

**Attack Challenge Mode**
حالت Vercel Firewall که هنگام حمله فعال می‌شود و همهٔ بازدیدکنندگان چالش اثبات انسان بودن می‌بینند.
📁 فایل ۱۲

**AVIF / WebP**
فرمت‌های تصویر مدرن با فشرده‌سازی بهتر. Next.js خودکار به این‌ها تبدیل می‌کند.
📁 فایل ۱۰

---

## B

**Blob**
سرویس ذخیره‌سازی فایل Vercel (Object Storage). برای تصویر، ویدیو و فایل آپلودی.
📁 فایل ۰۸

**BotID / Bot Management**
سیستم تشخیص ربات از انسان. برای مقابله با اسکرپرها و حملات.
📁 فایل ۱۲

**Build**
فرایند تبدیل کد منبع به خروجی قابل اجرا (HTML/JS/CSS/توابع).
📁 فایل ۰۳

**Build Cache**
کش نتایج Build قبلی برای سرعت‌بخشی به Buildهای بعدی.
📁 فایل ۰۳

---

## C

**Cache-Control**
هدر HTTP که قوانین کش را تعیین می‌کند (`s-maxage`، `max-age`، `no-store` و ...).
📁 فایل ۱۱

**Cache Tag**
برچسبی که به یک `fetch` می‌چسبانید تا بعداً با `revalidateTag` کش را دقیق پاک کنید.
📁 فایل ۱۱

**CDN (Content Delivery Network)**
شبکهٔ جهانی سرورها که محتوا را از نزدیک‌ترین نقطه به کاربر می‌رساند.
📁 فایل ۱۰

**Client Component**
کامپوننت React که در مرورگر اجرا می‌شود و با `"use client"` مشخص می‌شود.
📁 فایل ۰۴

**CLS (Cumulative Layout Shift)**
معیار پرش ناخواستهٔ چیدمان. هدف: زیر ۰.۱.
📁 فایل ۱۰

**Cold Start**
تأخیر ساخت نمونهٔ جدید تابع Serverless.
📁 فایل ۰۵

**Connection Pooling**
مدیریت اتصال‌های دیتابیس برای جلوگیری از اشباع در محیط Serverless.
📁 فایل ۰۸

**Core Web Vitals**
سه معیار اصلی عملکرد گوگل: **LCP، INP، CLS**.
📁 فایل ۱۰

**CORS (Cross-Origin Resource Sharing)**
مکانیزم امنیتی مرورگر برای کنترل دسترسی بین دامنه‌ها.
📁 فایل ۱۶

**Cron Jobs**
زمان‌بندی اجرای خودکار توابع. در `vercel.json` تعریف می‌شود.
📁 فایل ۰۹

**CSR (Client-Side Rendering)**
رندر صفحه در مرورگر با JavaScript.
📁 فایل ۰۱

---

## D

**Data Cache**
لایهٔ کش Next.js برای نتایج `fetch` بین رندرها.
📁 فایل ۱۱

**Deploy Hook**
URL یکتا که با فراخوانی آن، یک استقرار جدید ساخته می‌شود.
📁 فایل ۰۹

**Deployment**
یک استقرار. هر Push یک Deployment جدید می‌سازد.
📁 فایل ۰۳

**Deployment Protection**
مجموعه ابزارهای کنترل دسترسی به استقرارها (Auth، Password، IP).
📁 فایل ۰۳

**DNS (Domain Name System)**
سیستم تبدیل نام دامنه به آدرس IP.
📁 فایل ۰۷

**Drain**
ارسال داده‌های Vercel (لاگ، متریک) به سرویس بیرونی.
📁 فایل ۱۳

**DDoS (Distributed Denial of Service)**
حملهٔ حجمی برای از کار انداختن سرویس.
📁 فایل ۱۲

---

## E

**Edge Config**
ذخیره‌گاه کلید-مقدار فوق‌سریع برای تنظیمات و Feature Flag.
📁 فایل ۰۸

**Edge Function**
تابعی که روی نقاط لبه (نزدیک کاربر) اجرا می‌شود.
📁 فایل ۰۵

**Edge Network**
شبکهٔ جهانی Vercel برای سرو محتوا و کد از نزدیک‌ترین نقطه.
📁 فایل ۱۰

**Environment Variable**
جفت نام-مقدار بیرون از کد برای پیکربندی و Secrets.
📁 فایل ۰۷

---

## F

**Fast Data Transfer**
معیار مصرف پهنای باند در Vercel.
📁 فایل ۱۴

**Firewall**
لایهٔ امنیتی در لبه برای بررسی و مسدودسازی درخواست‌ها.
📁 فایل ۱۲

**Fluid Compute**
مدل اجرایی مدرن Vercel با همزمانی درون‌تابعی و Cold Start کمتر.
📁 فایل ۰۵

**Full Route Cache**
لایهٔ کش Next.js برای HTML کامل مسیرهای استاتیک.
📁 فایل ۱۱

---

## G

**generateStaticParams**
تابعی در Next.js که مسیرهای پویا را از قبل می‌سازد.
📁 فایل ۰۴

**Geo-targeting**
شخصی‌سازی محتوا یا ریدایرکت بر اساس موقعیت جغرافیایی کاربر.
📁 فایل ۰۶

---

## H

**Hobby Plan**
پلن رایگان Vercel، فقط برای استفادهٔ غیرتجاری.
📁 فایل ۱۴

**HMAC**
الگوریتم امضای رمزنگاری برای تأیید Webhook.
📁 فایل ۰۹

**HSTS (HTTP Strict Transport Security)**
هدری که مرورگر را مجبور به استفاده از HTTPS می‌کند.
📁 فایل ۰۷

---

## I

**Idempotency**
خاصیتی که اجرای چندبارهٔ یک عملیات، نتیجه را تغییر نمی‌دهد.
📁 فایل ۰۹

**IDOR (Insecure Direct Object Reference)**
آسیب‌پذیری ناشی از نبود بررسی مالکیت در دسترسی به داده.
📁 فایل ۱۲

**Image Optimization**
بهینه‌سازی خودکار تصاویر (فرمت، اندازه، کش) روی لبه.
📁 فایل ۱۰

**INP (Interaction to Next Paint)**
معیار پاسخ‌دهی به تعامل کاربر. هدف: زیر ۲۰۰ms.
📁 فایل ۱۰

**ISR (Incremental Static Regeneration)**
ترکیب استاتیک و بازسازی دوره‌ای.
📁 فایل ۱۱

---

## J

**JAMstack**
معماری مبتنی بر JavaScript، API و Markup.
📁 فایل ۰۱

---

## K

**KV (Key-Value)**
دیتابیس کلید-مقدار فوق‌سریع (Redis). برای نشست، کش، شمارنده.
📁 فایل ۰۸

---

## L

**LCP (Largest Contentful Paint)**
زمان نمایش بزرگ‌ترین عنصر. هدف: زیر ۲.۵ ثانیه.
📁 فایل ۱۰

**Let's Encrypt**
مرجع صدور گواهی SSL رایگان که Vercel استفاده می‌کند.
📁 فایل ۰۷

**Log Drain**
ارسال لاگ‌ها به سرویس بیرونی برای نگهداری بلندمدت.
📁 فایل ۱۳

---

## M

**Middleware**
کدی که قبل از رسیدن درخواست به صفحه اجرا می‌شود (روی Edge).
📁 فایل ۰۶

**Monorepo**
مخزنی که چند پروژه/پکیج را نگه می‌دارد.
📁 فایل ۰۲

---

## N

**Next.js**
فریم‌ورک React ساختهٔ Vercel برای اپ‌های فول‌استک.
📁 فایل ۰۴

**Node.js Runtime**
محیط اجرای کامل Node.js برای توابع Serverless.
📁 فایل ۰۵

---

## O

**Observability**
توانایی فهمیدن وضعیت سیستم از طریق لاگ، متریک و تراس.
📁 فایل ۱۳

**On-Demand Revalidation**
بازسازی کش به‌صورت دستی با `revalidatePath` یا `revalidateTag`.
📁 فایل ۱۱

---

## P

**Pages Router**
سیستم مسیریابی قدیمی‌تر Next.js با پوشهٔ `pages/`.
📁 فایل ۰۴

**Postgres**
دیتابیس رابطه‌ای (SQL) برای دادهٔ ساختاریافته.
📁 فایل ۰۸

**PPR (Partial Prerendering)**
پوستهٔ استاتیک فوری + بخش‌های پویای استریمی.
📁 فایل ۱۱

**Preview Deployment**
استقرار خودکار برای هر شاخه/PR با آدرس یکتا.
📁 فایل ۰۳

**Promotion**
ارتقای یک استقرار Preview به Production بدون Build مجدد.
📁 فایل ۰۳

**Protection Bypass**
توکن برای عبور ابزارهای خودکار از Deployment Protection.
📁 فایل ۰۳

---

## Q

**Query Parameter**
پارامتر در URL (`?key=value`).
📁 فایل ۰۴

---

## R

**Rate Limiting**
محدود کردن تعداد درخواست در بازهٔ زمانی.
📁 فایل ۱۲

**Region**
منطقهٔ جغرافیایی اجرای توابع.
📁 فایل ۰۵

**Request Memoization**
کش خودکار `fetch` تکراری در یک رندر.
📁 فایل ۱۱

**REST API**
رابط برنامه‌نویسی مبتنی بر HTTP Vercel.
📁 فایل ۰۹

**Rollback (Instant Rollback)**
بازگشت فوری به استقرار قبلی بدون Build مجدد.
📁 فایل ۰۳

**Route Handler**
تابع API در App Router (`route.ts`).
📁 فایل ۰۴

**Router Cache**
کش مرورگر برای صفحات بازدیدشده.
📁 فایل ۱۱

---

## S

**Secret**
دادهٔ حساس (کلید، رمز) که باید محافظت شود.
📁 فایل ۱۲

**Server Component**
کامپوننت React که روی سرور رندر می‌شود (پیش‌فرض App Router).
📁 فایل ۰۴

**Serverless**
اجرای کد بدون مدیریت سرور.
📁 فایل ۰۵

**Skew Protection**
تضمین سازگاری نسخه در حین Deploy.
📁 فایل ۰۳

**Speed Insights**
ابزار دادهٔ میدانی Core Web Vitals.
📁 فایل ۱۳

**Spend Management**
تعیین سقف هزینه برای جلوگیری از صورتحساب غافلگیرکننده.
📁 فایل ۱۴

**SSG (Static Site Generation)**
تولید صفحه در زمان Build.
📁 فایل ۰۱

**SSL/TLS**
پروتکل رمزنگاری ارتباط.
📁 فایل ۰۷

**SSR (Server-Side Rendering)**
رندر صفحه در هر درخواست روی سرور.
📁 فایل ۰۱

**Streaming**
ارسال تدریجی HTML به مرورگر.
📁 فایل ۰۶

**Suspense**
کامپوننت React برای نمایش fallback در حین بارگذاری.
📁 فایل ۰۶

---

## T

**Timeout**
توقف تابع پس از مدت مشخص.
📁 فایل ۱۶

**TTFB (Time to First Byte)**
زمان دریافت اولین بایت پاسخ.
📁 فایل ۱۰

---

## U

**Upstash**
ارائه‌دهندهٔ Redis پشت Vercel KV.
📁 فایل ۰۸

---

## V

**Vercel CLI**
ابزار خط فرمان Vercel.
📁 فایل ۰۲

**Vercel KV / Postgres / Blob**
سرویس‌های ذخیره‌سازی Vercel.
📁 فایل ۰۸

---

## W

**WAF (Web Application Firewall)**
فایروال لایهٔ اپلیکیشن.
📁 فایل ۱۲

**Web Analytics**
ابزار تحلیل رفتار کاربران.
📁 فایل ۱۳

**Webhook**
تماس معکوس HTTP هنگام رخ دادن رویداد.
📁 فایل ۰۹

**Wildcard Domain**
دامنه‌ای که همهٔ زیردامنه‌ها را می‌گیرد (`*.myapp.com`).
📁 فایل ۰۷

---

## X

**XSS (Cross-Site Scripting)**
تزریق اسکریپت مخرب در صفحه.
📁 فایل ۱۲

---

## Z

**Zod**
کتابخانهٔ اعتبارسنجی تایپ‌سیف برای ورودی‌ها.
📁 فایل ۱۲

---

## اصطلاحات فارسی–انگلیسی سریع

| فارسی | انگلیسی |
|---|---|
| استقرار | Deployment |
| ساخت | Build |
| توابع بدون سرور | Serverless Functions |
| توابع لبه | Edge Functions |
| متغیر محیطی | Environment Variable |
| دامنهٔ اختصاصی | Custom Domain |
| گواهی امنیتی | SSL Certificate |
| کش لبه | Edge Cache |
| بازسازی افزایشی | Incremental Static Regeneration |
| رندر جزئی اولیه | Partial Prerendering |
| محدودسازی نرخ | Rate Limiting |
| پایش | Observability |
| بازگشت | Rollback |
| محافظت از استقرار | Deployment Protection |
| واژه‌نامه | Glossary |

---

## گام بعدی

- [فایل ۲۰ — منابع و مراجع](./20-Resources.md)
- [فایل ۰۱ — آشنایی با Vercel](./01-Introduction.md)
