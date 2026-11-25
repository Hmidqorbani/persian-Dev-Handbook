# وب‌سرورها — از صفر تا حرفه‌ای

**راهنمای جامع و عملی Apache، Nginx و IIS — از HTTP تا Hardening**

---

## این کتابچه برای کیست؟

برای کسی که **کمی** با لینوکس و شبکه آشناست و می‌خواهد از «یک سایت بالا آوردم» برسد به «می‌توانم یک وب‌سرور را برای ترافیک واقعی طراحی، امن، بهینه و عیب‌یابی کنم».

پیش‌نیاز رسمی:
- کار با خط فرمان لینوکس در سطح `cd`، `systemctl`، `tail`، `grep`
- آشنایی مقدماتی با شبکه (IP، پورت، DNS)
- آشنایی با یک زبان برنامه‌نویسی برای فصل‌های ۲۸ و ۲۹ (اختیاری)

**لازم نیست** HTTP را از قبل بدانی، TLS بشناسی یا Apache دیده باشی. هر مفهومی که لازم شود، از صفر توضیح داده می‌شود.

---

## چه چیزی یاد می‌گیری

| حوزه | خروجی عملی |
|---|---|
| پروتکل | HTTP/1.1، HTTP/2، HTTP/3، TLS — از پیام خام تا handshake |
| Apache | نصب، `VirtualHost`، `mod_rewrite`، MPM، `.htaccess`، PHP-FPM |
| Nginx | `server`/`location`، `proxy_pass`، `try_files`، cache، stream |
| IIS | `Application Pool`، binding، `web.config`، ARR، ASP.NET Core |
| Reverse Proxy | هدرهای `X-Forwarded-*`، `real_ip`، WebSocket، timeout |
| Load Balancing | L4/L7، الگوریتم‌ها، health check، sticky session، HAProxy |
| TLS | Let's Encrypt، cipher suite، HSTS، mTLS، تست با `testssl.sh` |
| کارایی | gzip/brotli/zstd، micro-cache، HTTP/2 push و HTTP/3، tuning |
| امنیت | هدرهای امنیتی، rate limit، جلوگیری از traversal، WAF، `fail2ban` |
| عملیات | لاگ و تحلیل، مانیتورینگ، HA، Docker، Kubernetes Ingress |
| عیب‌یابی | `502`/`504`، `403`، TLS، DNS — با ابزار و روش |

---

## ساختار کتابچه

**۱۹ فایل محتوایی + این README + `SUMMARY.md` — مجموعاً ۸۳ فصل**

| # | فایل | موضوع | فصل‌ها | سطح |
|---|---|---|---|---|
| ۰۱ | `01-Fundamentals.md` | وب‌سرور چیست، HTTP/1.1، HTTP/2 و ۳، TLS | ۱–۴ | مبتدی |
| ۰۲ | `02-Networking-Basics.md` | TCP، پورت، DNS، پروکسی، WebSocket | ۵–۸ | مبتدی |
| ۰۳ | `03-Architectures-and-Comparison.md` | Apache/Nginx/IIS، مقایسه و انتخاب | ۹–۱۲ | مبتدی→متوسط |
| ۰۴ | `04-Apache-Install-and-Config.md` | نصب، `VirtualHost`، `.htaccess` | ۱۳–۱۷ | مبتدی |
| ۰۵ | `05-Nginx-Install-and-Config.md` | نصب، `server`، `location`، `try_files` | ۱۸–۲۲ | مبتدی |
| ۰۶ | `06-IIS-Install-and-Config.md` | نصب، `App Pool`، `web.config`، ASP.NET | ۲۳–۲۶ | متوسط |
| ۰۷ | `07-Static-and-Dynamic-Content.md` | MIME، PHP-FPM، Python/Node، خطاهای ۴۰۳/۵۰۰ | ۲۷–۳۰ | متوسط |
| ۰۸ | `08-Reverse-Proxy.md` | `proxy_pass`، `mod_proxy`، هدرها، WebSocket | ۳۱–۳۵ | متوسط |
| ۰۹ | `09-Load-Balancing.md` | L4/L7، upstream، HAProxy، ARR، sticky | ۳۶–۴۰ | متوسط→پیشرفته |
| ۱۰ | `10-TLS-and-Certificates.md` | TLS 1.3، ACME، HSTS، mTLS، تست | ۴۱–۴۵ | متوسط |
| ۱۱ | `11-Logs-and-Monitoring.md` | فرمت لاگ، `GoAccess`، `stub_status`، متریک | ۴۶–۴۹ | متوسط |
| ۱۲ | `12-Performance-and-Caching.md` | cache، compression، HTTP/2 و ۳، tuning | ۵۰–۵۴ | پیشرفته |
| ۱۳ | `13-Security-and-Hardening.md` | هدرها، rate limit، traversal، WAF، چک‌لیست | ۵۵–۵۹ | پیشرفته |
| ۱۴ | `14-URL-Rewriting-and-Redirects.md` | `mod_rewrite`، `return`، SPA، canonical | ۶۰–۶۳ | متوسط |
| ۱۵ | `15-High-Availability.md` | Keepalived، VIP، Ingress، CDN | ۶۴–۶۷ | پیشرفته |
| ۱۶ | `16-Containers-and-Deployment.md` | Docker، Compose، K8s، CI/CD | ۶۸–۷۱ | متوسط |
| ۱۷ | `17-Troubleshooting.md` | `502`/`504`، permission، TLS، ابزارها | ۷۲–۷۵ | متوسط |
| ۱۸ | `18-Complete-Lab-Project.md` | پروژه‌ی کامل: Nginx + app + TLS + LB | ۷۶–۸۰ | پروژه |
| ۱۹ | `19-Checklist-and-Glossary.md` | چک‌لیست تولید، واژه‌نامه، نقشه راه | ۸۱–۸۳ | مرجع |

### نقشه‌ی مسیر پیشنهادی

```
مسیر کامل (۴۵ تا ۶۰ ساعت):
۰۱ → ۰۲ → ۰۳ → ۰۴ → ۰۵ → ۰۶ → ۰۷ → ۰۸ → ۰۹
   → ۱۰ → ۱۱ → ۱۲ → ۱۳ → ۱۴ → ۱۵ → ۱۶ → ۱۷ → ۱۸ → ۱۹

مسیر «فقط Nginx» (~۲۲ ساعت):
۰۱ → ۰۲ → ۰۳ (فصل ۱۰) → ۰۵ → ۰۷ → ۰۸ → ۰۹ → ۱۰ → ۱۲ → ۱۳ → ۱۸

مسیر «فقط IIS / ویندوز» (~۱۵ ساعت):
۰۱ → ۰۲ → ۰۳ (فصل ۱۱) → ۰۶ → ۰۷ (فصل ۲۹) → ۱۰ → ۱۳ (بخش IIS) → ۱۷

مسیر «DevOps / SRE» (~۳۰ ساعت):
۰۱ → ۰۲ → ۰۳ → ۰۵ → ۰۸ → ۰۹ → ۱۰ → ۱۱ → ۱۲ → ۱۳ → ۱۵ → ۱۶ → ۱۷
```

---

## چطور از این کتابچه استفاده کنیم

1. **از فایل ۰۱ شروع کن.** شماره‌گذاری فصل‌ها سراسری است (۱ تا ۸۳)، پس ارجاع‌ها همه‌جا معتبرند.
2. **هر فصل یک `Lab Scenario` دارد.** آن را روی یک VM یا Docker اجرا کن؛ خواندن config بدون اجرا، یادگیری نیست.
3. **سه سرور را با هم مقایسه کن.** در هر مبحث، جدول مقایسه Apache/Nginx/IIS هست. این جدول‌ها، مهارت واقعی بازار کار هستند.
4. **فایل ۱۸ (پروژه) را جدی بگیر.** تنها جایی است که همه‌ی تکه‌ها کنار هم می‌نشیند.
5. **فایل ۱۹ را چاپ کن.** چک‌لیست تولید، چیزی است که سرِ پروژه‌ی واقعی به آن نگاه می‌کنی.

---

## محیط آزمایشگاه کتابچه

برای اینکه مثال‌ها قابل اجرا باشند، از این محیط استفاده می‌کنیم:

```
┌──────────────────────────────────────────────────────┐
│  کلاینت (curl / مرورگر)                              │
└───────────────────────┬──────────────────────────────┘
                        │ :80 / :443
                        ▼
┌──────────────────────────────────────────────────────┐
│  web01 (Ubuntu 24.04)                                │
│  ├── Nginx 1.26   ← reverse proxy + TLS + static     │
│  ├── Apache 2.4   ← در فصل‌های مقایسه                │
│  └── PHP-FPM 8.3  ← اپلیکیشن نمونه                   │
└───────────────────────┬──────────────────────────────┘
                        │ :8080
                        ▼
┌──────────────────────────────────────────────────────┐
│  app01 (Node.js / Python) — سرویس نمونه              │
└──────────────────────────────────────────────────────┘

دامنه‌ی آزمایشی: lab.example.com  →  192.168.56.10
سیستم‌عامل دوم: Windows Server 2022 (برای IIS)
```

اگر Docker داری، سریع‌ترین راه:

```bash
docker run -d --name web01 -p 8080:80 -p 8443:443 \
  -v "$PWD/conf:/etc/nginx/conf.d" \
  -v "$PWD/html:/usr/share/nginx/html" \
  nginx:1.26
```

---

## کنوانسیون‌های این کتابچه

| مورد | قاعده |
|---|---|
| شماره‌ی فصل | سراسری و پیوسته (۱ تا ۸۳) |
| ساختار هر فصل | ۷ زیربخش ثابت: مقدمه، مفاهیم اساسی، مثال‌های عملی، مقایسه، اشکال شایع، نکات مهم، خلاصه |
| نسخه‌ی مبنا | Nginx 1.26، Apache 2.4، IIS 10 (Windows Server 2022) |
| مسیرها | لینوکس: `/etc/nginx/`، `/etc/apache2/` — ویندوز در جدول جدا |
| کد | کامل و قابل کپی؛ هر config با `nginx -t` یا `apachectl configtest` تست‌شده |
| ارجاع متقابل | «→ فصل ۳۲» یعنی به آن فصل برو |
| منابع | انتهای هر فایل، بخش `## منابع و مراجع` |

> **نکته‌ی نسخه:** نام فایل‌ها و ماژول‌ها بین توزیع‌ها متفاوت است (Debian: `apache2.conf` + `sites-available/`؛ RHEL: `httpd.conf` + `conf.d/`). هر جا تفاوت مهم بود، هر دو آمده.

---

## مجوز و استفاده

MIT — آزاد برای استفاده‌ی شخصی، تدریس و انتشار، با ذکر منبع.

---

**شروع از:** [`01-Fundamentals.md`](01-Fundamentals.md) — یا فهرست کامل ۸۳ فصل در [`SUMMARY.md`](SUMMARY.md).
