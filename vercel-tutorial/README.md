# کتابچه آموزشی جامع Vercel — از صفر تا تولید

> راهنمای کامل، ساده و عملی برای توسعه‌دهندگان فرانت‌اند و فول‌استک که می‌خواهند از صفر با Vercel آشنا شوند، پروژه بسازند و آن را حرفه‌ای منتشر کنند.

---

## این کتابچه برای چه کسی است؟

- **مبتدی‌ها**: کسانی که با HTML/CSS/JS آشنایی دارند و می‌خواهند اولین پروژه‌شان را روی اینترنت منتشر کنند.
- **توسعه‌دهندگان فرانت‌اند**: کسانی که با React/Next.js کار می‌کنند و می‌خواهند استقرار و مقیاس‌پذیری را حرفه‌ای یاد بگیرند.
- **توسعه‌دهندگان فول‌استک**: کسانی که می‌خواهند توابع سرور، دیتابیس، ذخیره‌سازی و API را روی Vercel پیاده کنند.
- **مدرسان**: کسانی که می‌خواهند این محتوا را تدریس کنند.

## پیش‌نیازها

| پیش‌نیاز | سطح لازم |
|---|---|
| HTML / CSS / JavaScript | متوسط (بتوانید یک صفحه ساده بسازید) |
| Git و GitHub | مقدماتی (commit و push) |
| Node.js و npm | مقدماتی (نصب و اجرای `npm install`) |
| خط فرمان (Terminal) | مقدماتی (جابه‌جایی بین پوشه‌ها) |

> اگر هر کدام از این‌ها را نمی‌دانید نگران نباشید — در همان فایل‌هایی که لازم می‌شود، مفاهیم پایه توضیح داده شده‌اند.

## آنچه یاد خواهید گرفت

- استقرار خودکار با Git و مدیریت محیط‌های Preview و Production
- ساخت API با **Serverless Functions** و **Edge Functions**
- **Middleware**، احراز هویت، A/B Testing و **Streaming**
- متغیرهای محیطی، **دامنه اختصاصی**، HTTPS/SSL و DNS
- سرویس‌های داده: **KV، Postgres، Blob، Edge Config**
- **Cron Jobs**، **Webhooks**، **Integrations** و **REST API**
- بهینه‌سازی عملکرد: **CDN**، **Edge Cache**، **Image Optimization**، **Fluid Compute**، **ISR** و **PPR**
- امنیت: **Secrets**، **Firewall**، **Bot Management**، **Rate Limiting**، **DDoS**
- پایش سیستم: **Logs**، **Analytics**، **Speed Insights** و **Drains**
- مقایسه با Netlify / Cloudflare Pages / AWS Amplify و **مهاجرت** به Vercel
- عیب‌یابی خطاهای رایج: Build Fail، DNS، CORS، Timeout، Cold Start
- انتخاب پلن درست: **Hobby / Pro / Enterprise**

---

## نقشه راه یادگیری

مسیر پیشنهادی در **۴ مرحله**:

```
مرحله ۱ — پایه (بفهم چه چیزی و چرا)
   01-Introduction → 02-Setup-and-Account → 03-Git-and-Deployments

مرحله ۲ — استقرار حرفه‌ای (پروژه‌ات را زنده کن)
   04-Nextjs-on-Vercel → 07-Environment-Variables-and-Domains

مرحله ۳ — قابلیت‌های پیشرفته (اپلیکیشن واقعی بساز)
   05-Serverless-and-Edge-Functions → 06-Middleware-and-Streaming
   → 08-Storage-Services → 09-Cron-Webhooks-Integrations-API

مرحله ۴ — تولید و کیفیت (حرفه‌ای تحویل بده)
   10-Performance → 11-Caching-ISR-and-PPR → 12-Security
   → 13-Observability → 14-Plans-and-Pricing
   → 15-Comparison-and-Migration → 16-Troubleshooting → 17-Final-Project
```

---

## فهرست فایل‌ها

| # | فایل | موضوع | سطح | مدت تقریبی |
|---|---|---|---|---|
| — | [README.md](./README.md) | معرفی و نقشه راه | — | ۱۰ دقیقه |
| — | [SUMMARY.md](./SUMMARY.md) | فهرست کامل فصل‌ها | — | — |
| ۰۱ | [01-Introduction.md](./01-Introduction.md) | آشنایی با Vercel و وب مدرن | مبتدی | ۲–۳ ساعت |
| ۰۲ | [02-Setup-and-Account.md](./02-Setup-and-Account.md) | راه‌اندازی حساب و اولین استقرار | مبتدی | ۲–۳ ساعت |
| ۰۳ | [03-Git-and-Deployments.md](./03-Git-and-Deployments.md) | گیت، استقرار و گردش کار | مبتدی | ۳–۴ ساعت |
| ۰۴ | [04-Nextjs-on-Vercel.md](./04-Nextjs-on-Vercel.md) | Next.js روی Vercel | متوسط | ۴–۵ ساعت |
| ۰۵ | [05-Serverless-and-Edge-Functions.md](./05-Serverless-and-Edge-Functions.md) | توابع Serverless و Edge | متوسط | ۴–۵ ساعت |
| ۰۶ | [06-Middleware-and-Streaming.md](./06-Middleware-and-Streaming.md) | Middleware و Streaming | متوسط | ۳–۴ ساعت |
| ۰۷ | [07-Environment-Variables-and-Domains.md](./07-Environment-Variables-and-Domains.md) | متغیرهای محیطی، دامنه، DNS و SSL | مبتدی–متوسط | ۳–۴ ساعت |
| ۰۸ | [08-Storage-Services.md](./08-Storage-Services.md) | KV / Postgres / Blob / Edge Config | متوسط | ۴–۵ ساعت |
| ۰۹ | [09-Cron-Webhooks-Integrations-API.md](./09-Cron-Webhooks-Integrations-API.md) | Cron، Webhooks، API | متوسط | ۳–۴ ساعت |
| ۱۰ | [10-Performance-Optimization.md](./10-Performance-Optimization.md) | بهینه‌سازی عملکرد | متوسط | ۴–۵ ساعت |
| ۱۱ | [11-Caching-ISR-and-PPR.md](./11-Caching-ISR-and-PPR.md) | کش، ISR و PPR | پیشرفته | ۴–۵ ساعت |
| ۱۲ | [12-Security.md](./12-Security.md) | امنیت | متوسط–پیشرفته | ۳–۴ ساعت |
| ۱۳ | [13-Observability.md](./13-Observability.md) | Logs، Analytics، Drains | متوسط | ۲–۳ ساعت |
| ۱۴ | [14-Plans-and-Pricing.md](./14-Plans-and-Pricing.md) | پلن‌ها و قیمت‌گذاری | مبتدی | ۲ ساعت |
| ۱۵ | [15-Comparison-and-Migration.md](./15-Comparison-and-Migration.md) | مقایسه و مهاجرت | متوسط | ۳–۴ ساعت |
| ۱۶ | [16-Troubleshooting.md](./16-Troubleshooting.md) | عیب‌یابی خطاها | متوسط | ۳–۴ ساعت |
| ۱۷ | [17-Final-Project.md](./17-Final-Project.md) | پروژه نهایی واقعی | متوسط | ۶–۸ ساعت |
| ۱۸ | [18-Checklist.md](./18-Checklist.md) | چک‌لیست‌های جامع | — | ۳۰ دقیقه |
| ۱۹ | [19-Glossary.md](./19-Glossary.md) | واژه‌نامه | — | — |
| ۲۰ | [20-Resources.md](./20-Resources.md) | منابع و مراجع | — | — |

**مجموع زمان تخمینی: ۲۵ تا ۴۰ ساعت**

---

## نحوه استفاده

1. **ترتیبی (پیشنهادی)**: از فایل ۰۱ شروع کنید و به ترتیب جلو بروید. هر فایل بر پایه فایل قبلی ساخته شده است.
2. **موضوعی**: اگر با Vercel آشنایی دارید، مستقیم سراغ موضوع مورد نیازتان بروید (مثلاً فقط `12-Security.md`).
3. **مرجع**: هنگام کار واقعی، از فایل‌های `18-Checklist.md`، `19-Glossary.md` و `16-Troubleshooting.md` به‌عنوان مرجع سریع استفاده کنید.

> **نکته**: هر فایل مستقل قابل خواندن است، اما برای درک عمیق‌تر پیشنهاد می‌کنیم ترتیب را رعایت کنید.

---

## ساختار هر فایل

هر فایل محتوایی شامل این بخش‌هاست:

- **سرصفحه**: مدت یادگیری، سطح سختی، پیش‌نیاز و فهرست محتوا
- **چند فصل** که هر فصل دارای این ساختار است:
  - مقدمه
  - مفاهیم اساسی
  - مثال‌های عملی
  - مقایسه و الگوهای مختلف
  - اشکال شایع
  - نکات مهم (Tips & Tricks)
  - خلاصه فصل
- **منابع و مراجع** (فقط در انتهای فایل)

---

## مشارکت و بازخورد

- اگر خطا یا نکته‌ای پیدا کردید، Issue باز کنید یا Pull Request بفرستید.
- پیشنهاد می‌شود مخزن را **Star** کنید تا از به‌روزرسانی‌ها باخبر شوید.

## مجوز

این محتوا برای استفاده آموزشی آزاد است. استفاده تجاری از نام‌ها و علائم تجاری متعلق به صاحبان آن‌ها (Vercel Inc. و دیگران) بدون مجوز مجاز نیست.

---

**شروع مطالعه**: [فایل ۰۱ — آشنایی با Vercel و وب مدرن](./01-Introduction.md)
