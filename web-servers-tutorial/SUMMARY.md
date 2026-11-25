# فهرست کامل — ۸۳ فصل

**وب‌سرورها از صفر تا حرفه‌ای** · Nginx 1.26 · Apache 2.4 · IIS 10

---

## بخش ۰۱ — مبانی و HTTP

| فصل | عنوان | فایل |
|---|---|---|
| ۱ | وب‌سرور چیست و در مسیر یک درخواست کجا می‌نشیند | [`01`](01-Fundamentals.md#فصل-۱) |
| ۲ | HTTP/1.1 — پیام، متد، کد وضعیت و هدر | [`01`](01-Fundamentals.md#فصل-۲) |
| ۳ | HTTP/2 و HTTP/3 — چه چیزی واقعاً عوض شد | [`01`](01-Fundamentals.md#فصل-۳) |
| ۴ | HTTPS، TLS و گواهی — از handshake تا مرورگر | [`01`](01-Fundamentals.md#فصل-۴) |

## بخش ۰۲ — شبکه

| فصل | عنوان | فایل |
|---|---|---|
| ۵ | TCP، سوکت، پورت و صف اتصال | [`02`](02-Networking-Basics.md#فصل-۵) |
| ۶ | DNS — از دامنه تا IP | [`02`](02-Networking-Basics.md#فصل-۶) |
| ۷ | پروکسی، NAT، فایروال و `X-Forwarded-*` | [`02`](02-Networking-Basics.md#فصل-۷) |
| ۸ | WebSocket، SSE و اتصال‌های طولانی | [`02`](02-Networking-Basics.md#فصل-۸) |

## بخش ۰۳ — معماری و انتخاب

| فصل | عنوان | فایل |
|---|---|---|
| ۹ | Apache — MPM، ماژول و `.htaccess` | [`03`](03-Architectures-and-Comparison.md#فصل-۹) |
| ۱۰ | Nginx — master/worker و event loop | [`03`](03-Architectures-and-Comparison.md#فصل-۱۰) |
| ۱۱ | IIS — `HTTP.sys` و Application Pool | [`03`](03-Architectures-and-Comparison.md#فصل-۱۱) |
| ۱۲ | مقایسه، انتخاب و گزینه‌های دیگر | [`03`](03-Architectures-and-Comparison.md#فصل-۱۲) |

## بخش ۰۴ — Apache

| فصل | عنوان | فایل |
|---|---|---|
| ۱۳ | نصب روی Ubuntu/Debian و RHEL | [`04`](04-Apache-Install-and-Config.md#فصل-۱۳) |
| ۱۴ | نصب روی ویندوز و تفاوت‌ها | [`04`](04-Apache-Install-and-Config.md#فصل-۱۴) |
| ۱۵ | ساختار پیکربندی و سینتکس | [`04`](04-Apache-Install-and-Config.md#فصل-۱۵) |
| ۱۶ | `VirtualHost` — میزبانی چند سایت | [`04`](04-Apache-Install-and-Config.md#فصل-۱۶) |
| ۱۷ | `Directory`، `.htaccess` و `AllowOverride` | [`04`](04-Apache-Install-and-Config.md#فصل-۱۷) |

## بخش ۰۵ — Nginx

| فصل | عنوان | فایل |
|---|---|---|
| ۱۸ | نصب Nginx روی لینوکس و ویندوز | [`05`](05-Nginx-Install-and-Config.md#فصل-۱۸) |
| ۱۹ | ساختار `nginx.conf` و contextها | [`05`](05-Nginx-Install-and-Config.md#فصل-۱۹) |
| ۲۰ | `server` block و میزبانی چند سایت | [`05`](05-Nginx-Install-and-Config.md#فصل-۲۰) |
| ۲۱ | `location` — تطبیق، اولویت و تله‌ها | [`05`](05-Nginx-Install-and-Config.md#فصل-۲۱) |
| ۲۲ | `root`/`alias`، `try_files`، `internal` و `error_page` | [`05`](05-Nginx-Install-and-Config.md#فصل-۲۲) |

## بخش ۰۶ — IIS

| فصل | عنوان | فایل |
|---|---|---|
| ۲۳ | نصب IIS و سه راه مدیریت | [`06`](06-IIS-Install-and-Config.md#فصل-۲۳) |
| ۲۴ | Site، Application Pool و Binding | [`06`](06-IIS-Install-and-Config.md#فصل-۲۴) |
| ۲۵ | `web.config`، ماژول‌ها و handlerها | [`06`](06-IIS-Install-and-Config.md#فصل-۲۵) |
| ۲۶ | IIS و ASP.NET Core، URL Rewrite و ARR | [`06`](06-IIS-Install-and-Config.md#فصل-۲۶) |

## بخش ۰۷ — محتوای استاتیک و داینامیک

| فصل | عنوان | فایل |
|---|---|---|
| ۲۷ | سرو فایل استاتیک — مسیر، MIME، `index` و `autoindex` | [`07`](07-Static-and-Dynamic-Content.md#فصل-۲۷) |
| ۲۸ | PHP-FPM با Nginx و Apache | [`07`](07-Static-and-Dynamic-Content.md#فصل-۲۸) |
| ۲۹ | Python و Node.js — WSGI/ASGI و reverse proxy | [`07`](07-Static-and-Dynamic-Content.md#فصل-۲۹) |
| ۳۰ | خطاهای ۴۰۳، ۴۰۴ و ۵۰۰ — دیباگ مرحله‌به‌مرحله | [`07`](07-Static-and-Dynamic-Content.md#فصل-۳۰) |

## بخش ۰۸ — Reverse Proxy

| فصل | عنوان | فایل |
|---|---|---|
| ۳۱ | Reverse Proxy چیست و چرا لازم است | [`08`](08-Reverse-Proxy.md#فصل-۳۱) |
| ۳۲ | `proxy_pass` و `upstream` در Nginx | [`08`](08-Reverse-Proxy.md#فصل-۳۲) |
| ۳۳ | `mod_proxy` در Apache | [`08`](08-Reverse-Proxy.md#فصل-۳۳) |
| ۳۴ | هدرها، `X-Forwarded-*`، `real_ip` و timeout | [`08`](08-Reverse-Proxy.md#فصل-۳۴) |
| ۳۵ | WebSocket و SSE از پشت پروکسی | [`08`](08-Reverse-Proxy.md#فصل-۳۵) |

## بخش ۰۹ — Load Balancing

| فصل | عنوان | فایل |
|---|---|---|
| ۳۶ | مفاهیم — L4 در برابر L7 و الگوریتم‌ها | [`09`](09-Load-Balancing.md#فصل-۳۶) |
| ۳۷ | Nginx — `upstream`، `keepalive` و health check | [`09`](09-Load-Balancing.md#فصل-۳۷) |
| ۳۸ | HAProxy و `mod_proxy_balancer` | [`09`](09-Load-Balancing.md#فصل-۳۸) |
| ۳۹ | Sticky Session و اشتراک Session | [`09`](09-Load-Balancing.md#فصل-۳۹) |
| ۴۰ | ARR در IIS و Load Balancer ابری | [`09`](09-Load-Balancing.md#فصل-۴۰) |

## بخش ۱۰ — TLS

| فصل | عنوان | فایل |
|---|---|---|
| ۴۱ | TLS 1.2 و 1.3 — handshake و cipher suite | [`10`](10-TLS-and-Certificates.md#فصل-۴۱) |
| ۴۲ | Let's Encrypt و ACME با Certbot | [`10`](10-TLS-and-Certificates.md#فصل-۴۲) |
| ۴۳ | پیکربندی TLS در Nginx، Apache و IIS | [`10`](10-TLS-and-Certificates.md#فصل-۴۳) |
| ۴۴ | HSTS، OCSP Stapling، Redirect و mTLS | [`10`](10-TLS-and-Certificates.md#فصل-۴۴) |
| ۴۵ | مدیریت گواهی، چرخش و تست | [`10`](10-TLS-and-Certificates.md#فصل-۴۵) |

## بخش ۱۱ — لاگ و مانیتورینگ

| فصل | عنوان | فایل |
|---|---|---|
| ۴۶ | لاگ در Nginx — فرمت، متغیر و چرخش | [`11`](11-Logs-and-Monitoring.md#فصل-۴۶) |
| ۴۷ | لاگ در Apache و IIS | [`11`](11-Logs-and-Monitoring.md#فصل-۴۷) |
| ۴۸ | تحلیل لاگ — `GoAccess`، `awk` و ELK | [`11`](11-Logs-and-Monitoring.md#فصل-۴۸) |
| ۴۹ | مانیتورینگ — `stub_status` و متریک | [`11`](11-Logs-and-Monitoring.md#فصل-۴۹) |

## بخش ۱۲ — کارایی و کش

| فصل | عنوان | فایل |
|---|---|---|
| ۵۰ | کش — مرورگر، پروکسی و micro-cache | [`12`](12-Performance-and-Caching.md#فصل-۵۰) |
| ۵۱ | `proxy_cache` و `fastcgi_cache` در Nginx | [`12`](12-Performance-and-Caching.md#فصل-۵۱) |
| ۵۲ | فشرده‌سازی — `gzip`، Brotli و `zstd` | [`12`](12-Performance-and-Caching.md#فصل-۵۲) |
| ۵۳ | HTTP/2 و HTTP/3 در عمل | [`12`](12-Performance-and-Caching.md#فصل-۵۳) |
| ۵۴ | تنظیم — `keepalive`، بافر، worker و `fd` | [`12`](12-Performance-and-Caching.md#فصل-۵۴) |

## بخش ۱۳ — امنیت

| فصل | عنوان | فایل |
|---|---|---|
| ۵۵ | کاهش سطح حمله — نسخه، ماژول، کاربر | [`13`](13-Security-and-Hardening.md#فصل-۵۵) |
| ۵۶ | هدرهای امنیتی — CSP و بقیه | [`13`](13-Security-and-Hardening.md#فصل-۵۶) |
| ۵۷ | محدودسازی — نرخ، اتصال و IP | [`13`](13-Security-and-Hardening.md#فصل-۵۷) |
| ۵۸ | Directory Traversal، LFI/RFI، آپلود و Misconfiguration | [`13`](13-Security-and-Hardening.md#فصل-۵۸) |
| ۵۹ | WAF، `fail2ban`، DDoS و چک‌لیست | [`13`](13-Security-and-Hardening.md#فصل-۵۹) |

## بخش ۱۴ — بازنویسی URL

| فصل | عنوان | فایل |
|---|---|---|
| ۶۰ | `mod_rewrite` در Apache و `rewrite`/`return` در Nginx | [`14`](14-URL-Rewriting-and-Redirects.md#فصل-۶۰) |
| ۶۱ | Redirect، اسلش پایانی و canonical | [`14`](14-URL-Rewriting-and-Redirects.md#فصل-۶۱) |
| ۶۲ | SPA و الگوی front controller | [`14`](14-URL-Rewriting-and-Redirects.md#فصل-۶۲) |
| ۶۳ | URL Rewrite در IIS | [`14`](14-URL-Rewriting-and-Redirects.md#فصل-۶۳) |

## بخش ۱۵ — دسترس‌پذیری بالا

| فصل | عنوان | فایل |
|---|---|---|
| ۶۴ | HA برای وب‌سرور — active-passive و active-active | [`15`](15-High-Availability.md#فصل-۶۴) |
| ۶۵ | Keepalived، VIP و health check | [`15`](15-High-Availability.md#فصل-۶۵) |
| ۶۶ | Kubernetes Ingress و Ingress Controller | [`15`](15-High-Availability.md#فصل-۶۶) |
| ۶۷ | CDN و لبه — کش، WAF و DDoS | [`15`](15-High-Availability.md#فصل-۶۷) |

## بخش ۱۶ — کانتینر و استقرار

| فصل | عنوان | فایل |
|---|---|---|
| ۶۸ | Docker برای Nginx و Apache | [`16`](16-Containers-and-Deployment.md#فصل-۶۸) |
| ۶۹ | Compose — Nginx + app + Certbot | [`16`](16-Containers-and-Deployment.md#فصل-۶۹) |
| ۷۰ | Kubernetes برای وب‌سرور | [`16`](16-Containers-and-Deployment.md#فصل-۷۰) |
| ۷۱ | CI/CD، config as code و blue/green | [`16`](16-Containers-and-Deployment.md#فصل-۷۱) |

## بخش ۱۷ — عیب‌یابی

| فصل | عنوان | فایل |
|---|---|---|
| ۷۲ | `502`، `504`، `refused` و `timeout` | [`17`](17-Troubleshooting.md#فصل-۷۲) |
| ۷۳ | `403`، `404`، `500` و مجوز/SELinux | [`17`](17-Troubleshooting.md#فصل-۷۳) |
| ۷۴ | TLS، گواهی و DNS | [`17`](17-Troubleshooting.md#فصل-۷۴) |
| ۷۵ | جعبه‌ابزار — `curl`، `openssl`، `tcpdump`، `strace` | [`17`](17-Troubleshooting.md#فصل-۷۵) |

## بخش ۱۸ — پروژه

| فصل | عنوان | فایل |
|---|---|---|
| ۷۶ | سناریو، نیازمندی و معماری | [`18`](18-Complete-Lab-Project.md#فصل-۷۶) |
| ۷۷ | پیاده‌سازی گام‌به‌گام | [`18`](18-Complete-Lab-Project.md#فصل-۷۷) |
| ۷۸ | امنیت و بهینه‌سازی | [`18`](18-Complete-Lab-Project.md#فصل-۷۸) |
| ۷۹ | مانیتورینگ، لاگ و بکاپ | [`18`](18-Complete-Lab-Project.md#فصل-۷۹) |
| ۸۰ | تست بار، تحویل و چک‌لیست | [`18`](18-Complete-Lab-Project.md#فصل-۸۰) |

## بخش ۱۹ — مرجع

| فصل | عنوان | فایل |
|---|---|---|
| ۸۱ | چک‌لیست آمادگی تولید | [`19`](19-Checklist-and-Glossary.md#فصل-۸۱) |
| ۸۲ | واژه‌نامه‌ی وب‌سرور | [`19`](19-Checklist-and-Glossary.md#فصل-۸۲) |
| ۸۳ | نقشه راه یادگیری و منابع | [`19`](19-Checklist-and-Glossary.md#فصل-۸۳) |

---

## مسیرهای پیشنهادی

**مسیر کامل (۴۵–۶۰ ساعت):**
`01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09 → 10 → 11 → 12 → 13 → 14 → 15 → 16 → 17 → 18 → 19`

**مسیر «فقط Nginx» (~۲۲ ساعت):**
`01 → 02 → 03 (فصل ۱۰) → 05 → 07 → 08 → 09 → 10 → 12 → 13 → 18`

**مسیر «فقط IIS / ویندوز» (~۱۵ ساعت):**
`01 → 02 → 03 (فصل ۱۱) → 06 → 07 (فصل ۲۹) → 10 → 13 → 17`

**مسیر «DevOps / SRE» (~۳۰ ساعت):**
`01 → 02 → 03 → 05 → 08 → 09 → 10 → 11 → 12 → 13 → 15 → 16 → 17`

**مسیر «سریع به پروژه» (~۱۲ ساعت):**
`01 → 05 → 08 → 10 → 18`
