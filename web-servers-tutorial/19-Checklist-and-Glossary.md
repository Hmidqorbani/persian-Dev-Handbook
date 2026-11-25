# بخش ۱۹: چک‌لیست، واژه‌نامه و نقشه راه

**مدت یادگیری**: ۲ تا ۳ ساعت
**سطح سختی**: مرجع
**پیش‌نیاز**: بخش ۰۱ تا ۱۸ (جمع‌بندی)

---

## فهرست محتوا

- [فصل ۸۱: چک‌لیست آمادگی تولید](#فصل-۸۱)
- [فصل ۸۲: واژه‌نامه‌ی وب‌سرور](#فصل-۸۲)
- [فصل ۸۳: نقشه راه یادگیری و منابع](#فصل-۸۳)

---

## فصل ۸۱: چک‌لیست آمادگی تولید

### مقدمه

این فصل، فهرستی است که قبل از هر `go-live` باید از آن رد شوی. هر مورد، از تجربه‌ی واقعی آمده: چیزی که فراموش شد و بعداً درد داد. چاپش کن.

### مفاهیم اساسی

**۱) هفت حوزه**

| حوزه | تعداد |
|---|---|
| نصب و پایه | ۸ |
| TLS و هدرها | ۹ |
| امنیت | ۱۰ |
| محتوا و مسیر | ۸ |
| کارایی و کش | ۸ |
| لاگ و مانیتورینگ | ۷ |
| عملیات و بازیابی | ۷ |

**۲) سه سطح اولویت**

| سطح | معنا |
|---|---|
| 🔴 بحرانی | بدون آن، `go-live` نکن |
| 🟡 مهم | قبل از ترافیک واقعی |
| 🟢 مطلوب | در ۳۰ روز اول |

**۳) قاعده‌ی «تست‌نشده = انجام‌نشده»**

هر تیک باید مدرک داشته باشد: خروجی دستور، لینک داشبورد، لاگ تست.

### مثال‌های عملی

**الف) نصب و پایه**

- 🔴 نسخه‌ی دقیق (`nginx/1.26.x`، `httpd 2.4.x`، IIS 10)
- 🔴 `nginx -t` / `apachectl configtest` در CI
- 🔴 `worker_processes auto` و `worker_connections` متناسب
- 🔴 `worker_rlimit_nofile` و `limits.conf`
- 🟡 `sendfile`، `tcp_nopush`، `tcp_nodelay`
- 🟡 `keepalive_timeout` تنظیم‌شده
- 🟡 `client_max_body_size` هماهنگ با اپ
- 🟢 `open_file_cache` برای استاتیک

```bash
# تأیید
nginx -v
nginx -t
ps -eo cmd | grep 'nginx: worker' | wc -l
cat /proc/$(pgrep -f 'nginx: worker' | head -1)/limits | grep 'open files'
```

**ب) TLS و هدرها**

- 🔴 `fullchain.pem` (نه `cert.pem`)
- 🔴 `TLSv1.2 TLSv1.3` فقط
- 🔴 HSTS (`max-age`، `includeSubDomains`)
- 🔴 redirect `http → https`
- 🔴 `X-Content-Type-Options: nosniff`
- 🔴 `X-Frame-Options` یا `frame-ancestors`
- 🟡 `Referrer-Policy`
- 🟡 `Permissions-Policy`
- 🟡 CSP (یا report-only)

```bash
curl -sSI https://lab.example.com/ | grep -Ei 'strict-transport|x-content|x-frame|referrer|permissions|content-security'
echo | openssl s_client -connect lab.example.com:443 -servername lab.example.com 2>/dev/null | openssl x509 -noout -dates
docker run --rm -it drwetter/testssl.sh --quiet --severity HIGH https://lab.example.com
```

**ج) امنیت**

- 🔴 `server_tokens off` / `ServerTokens Prod` / `removeServerHeader`
- 🔴 `.git`، `.env`، `.bak`، `.sql` بسته
- 🔴 `autoindex off` / `-Indexes`
- 🔴 `TRACE` غیرفعال
- 🔴 rate limit روی login
- 🔴 مجوز `644`/`755` (بدون `777`)
- 🟡 `fail2ban`
- 🟡 WAF (ModSecurity + CRS)
- 🟡 `unattended-upgrades`
- 🟢 بازبینی ماهانه

```bash
for p in .git/config .env config.php.bak; do
  printf '%-18s → ' "$p"
  curl -sS -o /dev/null -w '%{http_code}\n' "https://lab.example.com/$p"
done

sudo find /var/www /srv/lab -type f -perm -o+w -ls | head
sudo fail2ban-client status
```

**د) محتوا و مسیر**

- 🔴 `root`/`alias` درست (`/` پایانی)
- 🔴 `try_files`/`FallbackResource` برای SPA
- 🔴 MIME برای `.woff2`، `.webp`، `.wasm`
- 🔴 `charset utf-8`
- 🟡 `404`/`500` سفارشی
- 🟡 API جدا از SPA
- 🟡 `X-Accel-Redirect` برای فایل خصوصی
- 🟢 `canonical` و redirect یکدست

```bash
curl -sSI https://lab.example.com/assets/app.css | grep -i content-type
curl -sS -o /dev/null -w '%{http_code}\n' https://lab.example.com/dashboard
curl -sS -o /dev/null -w '%{http_code}\n' https://lab.example.com/api/missing
```

**ه) کارایی و کش**

- 🔴 `gzip` + `gzip_vary`
- 🔴 `Cache-Control` برای استاتیک (با hash → `immutable`)
- 🔴 HTML بدون کش (`no-cache`)
- 🟡 `gzip_static` / Brotli
- 🟡 micro-cache API
- 🟡 HTTP/2 (و HTTP/3 اگر ممکن)
- 🟡 `proxy_cache_lock`
- 🟢 `stale-while-revalidate`

```bash
curl -sSI -H 'Accept-Encoding: gzip' https://lab.example.com/assets/app.css | grep -Ei 'content-encoding|vary'
curl -sSI https://lab.example.com/index.html | grep -i cache-control
curl -sSI --http2 https://lab.example.com/ | head -1
```

**و) لاگ و مانیتورینگ**

- 🔴 `access_log` با `$request_time` و `$upstream_addr`
- 🔴 `error_log` با سطح `warn`
- 🔴 `logrotate` + `USR1`
- 🔴 `stub_status` محدود
- 🟡 متریک 5xx، p95، cache hit
- 🟡 هشدار دیسک و گواهی
- 🟢 لاگ متمرکز

```bash
curl -s http://127.0.0.1:8081/status
sudo logrotate -d /etc/logrotate.d/nginx
sudo grep -c ' 5[0-9][0-9] ' /var/log/nginx/access.log
```

**ز) عملیات و بازیابی**

- 🔴 بکاپ روزانه + تست `restore`
- 🔴 `runbook` حادثه
- 🔴 `deploy.sh` با rollback
- 🔴 `health` واقعی (DB + cache)
- 🟡 `smoke test` بعد از deploy
- 🟡 `PDB`/`max_fails` برای HA
- 🟢 تست بار دوره‌ای

```bash
sudo -u lab /srv/lab/scripts/healthcheck.sh
ls -la /backup/lab/ | tail -5
sudo -u postgres createdb lab_restore && sudo -u postgres pg_restore -d lab_restore /backup/lab/db_*.dump && sudo -u postgres dropdb lab_restore
```

### مقایسه و الگوهای مختلف

**چک‌لیست کامل در برابر حداقلی**

| | کامل (۵۷) | حداقلی (۱۲) |
|---|---|---|
| زمان | ⚠️ ۲ روز | ✅ ۲ ساعت |
| پوشش | ✅ | ⚠️ |
| مناسب | تولید جدی | MVP |

**حداقلِ حداقل‌ها (اگر وقت نداری)**

```
۱. fullchain + TLS 1.2+
۲. redirect HTTPS + HSTS
۳. .git/.env بسته
۴. server_tokens off
۵. gzip + Cache-Control
۶. logrotate
۷. stub_status
۸. health واقعی
۹. بکاپ + تست restore
۱۰. rate limit login
۱۱. runbook
۱۲. deploy با rollback
```

**سه سطح بلوغ**

| سطح | ویژگی |
|---|---|
| ۱ | کار می‌کند، امن پایه |
| ۲ | مانیتور، کش، تست بار |
| ۳ | HA، WAF، CI/CD، SLO |

**دیدگاه «چک‌لیست بوروکراسی است»**: چک‌لیست، حافظه‌ی بیرونی است. تیم باتجربه هم موردی را فراموش می‌کند.

### اشکال شایع

**۱) تیک بدون مدرک**
«بکاپ داریم» بدون تست.

**۲) `cert.pem` به‌جای `fullchain`**
خطا در بعضی کلاینت‌ها.

**۳) `autoindex on` فراموش‌شده**
افشا.

**۴) `Cache-Control` برای HTML**
کاربر نسخه‌ی قدیمی.

**۵) `logrotate` بدون `USR1`**
فایل قدیمی.

**۶) `health` فقط `SELECT 1`**
DB `read-only` را نمی‌بیند.

**۷) `deploy` بدون rollback**
بحران.

**۸) `runbook` تست‌نشده**
اولین بار در حادثه.

**۹) `777` برای رفع سریع**
امنیت.

**۱۰) نبود `smoke`**
نسخه‌ی خراب.

### نکات مهم (Tips & Tricks)

- **چاپ کن**، در ویکی تیم بگذار.
- **مدرک** برای هر تیک.
- **بازبینی فصلی**.
- **حداقلِ حداقل‌ها** جدا.
- **`restore` ماهانه** با زمان‌سنجی.
- **`RCA`** بعد از حادثه.
- **`ADR`** برای تصمیم‌های بزرگ.

### خلاصه فصل

- هفت حوزه، ۵۷ مورد، سه اولویت.
- بحرانی: TLS، هدرها، فایل حساس، بکاپ تست‌شده.
- هر تیک، مدرک لازم دارد.
- حداقلِ حداقل‌ها را حفظ کن.
- بازبینی فصلی.

**سؤال مروری:** سه موردی که اگر انجام نشوند، `go-live` نکن، کدام‌اند و چرا؟

---

## فصل ۸۲: واژه‌نامه‌ی وب‌سرور

### مقدمه

این واژه‌نامه، اصطلاحاتی است که در کتابچه استفاده شد. برای مرجع سریع و تدریس.

### مفاهیم اساسی

**الف) پروتکل و HTTP**

| اصطلاح | معنا |
|---|---|
| **HTTP** | پروتکل درخواست-پاسخ، متنی، بی‌حالت |
| **HTTPS** | HTTP روی TLS |
| **`Host`** | هدر دامنه (اجباری در HTTP/1.1) |
| **`keep-alive`** | استفاده‌ی مجدد از اتصال TCP |
| **`chunked`** | انتقال بدنه با طول نامعلوم |
| **`Range`** | درخواست بخشی از فایل (`206`) |
| **`ETag`** | شناسه‌ی نسخه برای کش شرطی |
| **`Vary`** | کش بر اساس کدام هدر |
| **`301`/`302`** | redirect دائمی/موقت |
| **`307`/`308`** | redirect با حفظ متد |
| **`502`** | پاسخ بد از upstream |
| **`504`** | timeout upstream |
| **`499`** | کلاینت بست (Nginx) |
| **`429`** | درخواست زیاد |
| **`103 Early Hints`** | پیش‌بارگذاری |
| **`HOL blocking`** | مسدود شدن صف |
| **`multiplexing`** | چند stream روی یک اتصال |
| **`HPACK`/`QPACK`** | فشرده‌سازی هدر |
| **`ALPN`** | انتخاب پروتکل در TLS |
| **`SNI`** | نام دامنه در handshake |

**ب) سرور و معماری**

| اصطلاح | معنا |
|---|---|
| **Web Server** | پذیرش HTTP، تولید پاسخ |
| **App Server** | منطق کسب‌وکار |
| **`MPM`** | مدل پردازش Apache (`prefork`/`worker`/`event`) |
| **`event loop`** | حلقه‌ی رویداد Nginx |
| **`worker`** | پردازنده‌ی درخواست |
| **`HTTP.sys`** | لایه‌ی HTTP کرنل ویندوز (IIS) |
| **`App Pool`** | جداسازی پردازه در IIS |
| **`Binding`** | پروتکل + IP + پورت + hostname |
| **`VirtualHost`** | میزبانی چند سایت (Apache) |
| **`server block`** | میزبانی چند سایت (Nginx) |
| **`DocumentRoot`/`root`** | ریشه‌ی فایل |
| **`alias`** | جایگزینی مسیر |
| **`location`** | تطبیق مسیر Nginx |
| **`upstream`** | گروه سرورهای مقصد |
| **`reverse proxy`** | پروکسی به نمایندگی سرور |
| **`forward proxy`** | پروکسی به نمایندگی کلاینت |
| **`load balancer`** | توزیع بار |
| **`L4`/`L7`** | لایه‌ی انتقال/برنامه |
| **`health check`** | بررسی سلامت |
| **`sticky session`** | چسبیدن کاربر به یک سرور |
| **`connection draining`** | تخلیه‌ی اتصال قبل از حذف |

**ج) امنیت**

| اصطلاح | معنا |
|---|---|
| **`TLS`** | لایه‌ی امن انتقال |
| **`handshake`** | توافق کلید و پروتکل |
| **`cipher suite`** | مجموعه‌ی الگوریتم‌ها |
| **`forward secrecy`** | امنیت گذشته در برابر لو رفتن کلید |
| **`HSTS`** | اجبار HTTPS در مرورگر |
| **`OCSP stapling`** | پاسخ وضعیت گواهی از سرور |
| **`mTLS`** | احراز هویت کلاینت با گواهی |
| **`CSP`** | سیاست محتوای مرورگر |
| **`XSS`** | تزریق اسکریپت |
| **`CSRF`** | جعل درخواست |
| **`clickjacking`** | iframe پنهان |
| **`directory traversal`** | دسترسی به مسیر خارج |
| **`LFI`/`RFI`** | شامل‌کردن فایل محلی/دور |
| **`SSRF`** | جعل درخواست سمت سرور |
| **`WAF`** | فایروال لایه‌ی برنامه |
| **`rate limit`** | محدودیت نرخ |
| **`ACME`** | پروتکل صدور خودکار گواهی |
| **`CAA`** | محدود کردن CA مجاز |
| **`misconfiguration`** | پیکربندی اشتباه |
| **`hardening`** | سخت‌سازی |

**د) کارایی**

| اصطلاح | معنا |
|---|---|
| **`TTFB`** | زمان تا اولین بایت |
| **`p95`/`p99`** | صدک زمان پاسخ |
| **`cache hit ratio`** | نسبت پاسخ کش‌شده |
| **`micro-cache`** | کش کوتاه (۱–۵s) |
| **`stale-while-revalidate`** | سرو قدیمی + بازسازی |
| **`gzip`/Brotli/`zstd`** | الگوریتم فشرده‌سازی |
| **`gzip_static`** | فایل `.gz` از پیش |
| **`immutable`** | کش دائمی فایل hash‌دار |
| **`sendfile`** | کپی کرنل |
| **`open_file_cache`** | کش متادیتا |
| **`buffer`** | بافر خواندن/نوشتن |
| **`thundering herd`** | هجوم هم‌زمان |
| **`backlog`** | صف اتصال |
| **`TIME_WAIT`** | حالت بستن TCP |
| **`reuseport`** | پخش بار کرنل |

**ه) عملیات**

| اصطلاح | معنا |
|---|---|
| **`IaC`** | زیرساخت به‌صورت کد |
| **`blue/green`** | دو محیط، سوییچ |
| **`canary`** | درصدی از ترافیک |
| **`rolling update`** | تدریجی |
| **`rollback`** | بازگشت |
| **`runbook`** | دستورکار حادثه |
| **`RCA`** | تحلیل ریشه‌ای |
| **`SLO`** | هدف سطح سرویس |
| **`RTO`/`RPO`** | زمان بازیابی / داده‌ی از دست‌رفتنی |
| **`VIP`** | IP مجازی |
| **`VRRP`** | پروتکل افزونگی |
| **`quorum`** | حد نصاب |
| **`split-brain`** | دو رهبر |
| **`CDN`** | شبکه‌ی توزیع محتوا |
| **`edge`** | لبه‌ی شبکه |
| **`origin`** | سرور اصلی |

**و) اصطلاحات فایل و مسیر**

| اصطلاح | معنا |
|---|---|
| **`MIME type`** | نوع محتوا |
| **`charset`** | کدگذاری متن |
| **`trailing slash`** | اسلش پایانی |
| **`canonical`** | URL رسمی |
| **`fallback`** | مسیر جایگزین |
| **`front controller`** | یک نقطه‌ی ورود |
| **`SPA`** | اپ تک‌صفحه‌ای |
| **`X-Accel-Redirect`** | ارسال فایل توسط Nginx |
| **`internal`** | فقط دسترسی داخلی |
| **`symlink`** | پیوند نمادین |
| **`ACL`** | لیست کنترل دسترسی (ویندوز) |
| **`SELinux`** | کنترل دسترسی اجباری |

### مثال‌های عملی

**مثال ۱: جستجوی اصطلاح در مستندات محلی**

```bash
# Nginx
nginx -T | grep -n 'proxy_pass'

# Apache
apachectl -S
apachectl -M
```

**مثال ۲: جستجوی directive**

```bash
man nginx 2>/dev/null | head
# یا
curl -s https://nginx.org/en/docs/dirindex.html | grep -o 'ngx_http_[a-z_]*' | sort -u | head
```

**مثال ۳: بررسی نسخه و ماژول**

```bash
nginx -V 2>&1 | tr ' ' '\n' | grep module
apachectl -M
```

### مقایسه و الگوهای مختلف

**اصطلاح Nginx در برابر Apache در برابر IIS**

| مفهوم | Nginx | Apache | IIS |
|---|---|---|---|
| سایت | `server` | `VirtualHost` | Site |
| مسیر | `location` | `<Directory>`/`<Location>` | `web.config` |
| پروکسی | `proxy_pass` | `ProxyPass` | ARR |
| گروه | `upstream` | `balancer://` | Farm |
| config | `nginx.conf` | `httpd.conf` | `applicationHost.config` |
| reload | `-s reload` | `graceful` | `iisreset` |

**سه اصطلاح که بیشتر اشتباه می‌شوند**

1. **`root`** ≠ `DocumentRoot` در همه‌ی موارد (`alias` تفاوت دارد).
2. **`reverse proxy`** ≠ `load balancer` (هم‌پوشانی دارند).
3. **`502`** ≠ `504` (پاسخ بد در برابر timeout).

### اشکال شایع

**۱) `VirtualHost` و `server block` یکی نیستند**
مفهوم یکی، سینتکس متفاوت.

**۲) `proxy_pass` و `ProxyPass`**
Nginx و Apache.

**۳) `root` و `alias`**
اضافه در برابر جایگزین.

**۴) `502` و `503`**
upstream در برابر سرویس ناموجود.

**۵) `301` و `308`**
متد.

**۶) `HSTS` و redirect**
مکمل، نه یکی.

**۷) `cache` و `buffer`**
ذخیره در برابر انتقال.

**۸) `L4` و `L7`**
اتصال در برابر درخواست.

### نکات مهم (Tips & Tricks)

- **`nginx -T`** مرجع config مؤثر.
- **`apachectl -S`/`-M`**.
- **`nginx -V`** ماژول‌ها.
- **مستندات رسمی** مقدم بر وبلاگ.
- **`dirindex`** Nginx برای فهرست directive.
- **واژه‌نامه را در تدریس استفاده کن**.

### خلاصه فصل

- واژه‌نامه، مرجع سریع.
- سه اشتباه رایج: `root`/`alias`، proxy/LB، 502/504.
- نام‌ها بین سه سرور متفاوت‌اند.
- مستندات رسمی، منبع نهایی.

**سؤال مروری:** تفاوت `root` و `alias` را با یک مثال مسیر توضیح بده.

---

## فصل ۸۳: نقشه راه یادگیری و منابع

### مقدمه

این کتابچه، «از صفر تا حرفه‌ای» است، ولی مسیر بعد از آن تازه شروع می‌شود. این فصل، نقشه‌ی ۱۲ ماه آینده و منابع معتبر است.

### مفاهیم اساسی

**۱) چهار مسیر تخصصی**

| مسیر | تمرکز | فصل‌های مرتبط |
|---|---|---|
| **وب‌سرور / DevOps** | Nginx، TLS، استقرار | ۰۵، ۰۸–۱۰، ۱۶ |
| **SRE / عملیات** | HA، مانیتورینگ، حادثه | ۰۹، ۱۱، ۱۵، ۱۷ |
| **امنیت وب** | Hardening، WAF، TLS | ۱۰، ۱۳، ۱۷ |
| **معماری** | LB، CDN، مقیاس | ۰۸، ۰۹، ۱۵، ۱۶ |

**۲) نقشه‌ی ۱۲ ماه**

| ماه | هدف | خروجی |
|---|---|---|
| ۱ | HTTP و TCP | یک درخواست خام با `nc`، تحلیل `curl -w` |
| ۲ | Nginx پایه | سه سایت روی یک سرور |
| ۳ | TLS | `A` در SSL Labs |
| ۴ | پروکسی | اپ Python/Node پشت Nginx |
| ۵ | کش و فشرده‌سازی | cache hit >۵۰٪ |
| ۶ | امنیت | چک‌لیست فصل ۸۱ کامل |
| ۷ | لاگ و مانیتورینگ | داشبورد + هشدار |
| ۸ | LB | HAProxy با health check |
| ۹ | HA | Keepalived + failover |
| ۱۰ | کانتینر | Compose + CI/CD |
| ۱۱ | K8s | Ingress + cert-manager |
| ۱۲ | پروژه | فصل ۱۸ در تولید |

**۳) کتاب‌ها**

| کتاب | سطح | چرا |
|---|---|---|
| *High Performance Browser Networking* — Grigorik | متوسط | مرجع HTTP/2، TLS (رایگان) |
| *Bulletproof SSL and TLS* — Ristic | پیشرفته | TLS عمیق |
| *NGINX Cookbook* — DeJonghe | متوسط | دستورهای عملی |
| *The Apache Cookbook* | متوسط | Apache |
| *Web Operations* — Allspaw/Robbins | متوسط | عملیات |
| *Site Reliability Engineering* — Google | پیشرفته | SRE |

**۴) وبلاگ و مستندات**

| منبع | تمرکز |
|---|---|
| [nginx.org/docs](https://nginx.org/en/docs/) | مرجع |
| [httpd.apache.org/docs](https://httpd.apache.org/docs/2.4/) | مرجع |
| [learn.microsoft.com/iis](https://learn.microsoft.com/en-us/iis/) | IIS |
| [Nginx Blog](https://www.nginx.com/blog/) | کارایی |
| [Cloudflare Learning](https://www.cloudflare.com/learning/) | HTTP، TLS، CDN |
| [MDN HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP) | مرجع وب |
| [SSL Labs](https://www.ssllabs.com/) | تست TLS |
| [Mozilla Observatory](https://observatory.mozilla.org/) | هدرها |

**۵) تمرین**

| منبع | چه می‌دهد |
|---|---|
| [HTTP Bin](https://httpbin.org/) | تست HTTP |
| [Bad SSL](https://badssl.com/) | تست TLS |
| [SSL Labs](https://www.ssllabs.com/ssltest/) | نمره |
| [WebPageTest](https://www.webpagetest.org/) | کارایی |
| [Security Headers](https://securityheaders.com/) | هدرها |

**۶) گواهی‌نامه**

| گواهی | سطح |
|---|---|
| LFCS / LFCE | لینوکس |
| CKA / CKAD | Kubernetes |
| AWS/Azure/GCP associate | ابر |
| (اختصاصی وب‌سرور) | ندارد؛ تجربه مهم‌تر است |

**۷) جامعه**

| کانال | کاربرد |
|---|---|
| `#nginx` در Libera IRC | پرسش |
| [Nginx Forum](https://forum.nginx.org/) | پرسش |
| [Server Fault](https://serverfault.com/) | پرسش/پاسخ |
| [r/nginx](https://reddit.com/r/nginx) | اخبار |
| کنفرانس `nginx.conf` | شبکه‌سازی |

### مثال‌های عملی

**مثال ۱: آزمایشگاه شخصی**

```bash
# سه VM یا سه کانتینر
docker network create lab

docker run -d --name web1 --network lab -p 8081:80 nginx:1.26-alpine
docker run -d --name web2 --network lab -p 8082:80 nginx:1.26-alpine
docker run -d --name lb   --network lab -p 8080:80 nginx:1.26-alpine

# تمرین: LB بین web1 و web2
```

**مثال ۲: تمرین TLS**

```bash
# گواهی self-signed
openssl req -x509 -newkey rsa:2048 -nodes -days 1 \
  -keyout /tmp/t.key -out /tmp/t.crt -subj "/CN=lab.local"

# تست
curl --cacert /tmp/t.crt --resolve lab.local:443:127.0.0.1 https://lab.local/
```

**مثال ۳: تمرین کارایی**

```bash
hey -z 30s -c 50 http://localhost:8081/
ab -k -n 20000 -c 100 http://localhost:8081/
```

**مثال ۴: تمرین امنیت**

```bash
# بررسی هدرها
curl -sSI https://lab.example.com/ | grep -Ei 'strict|x-frame|x-content|content-security'

# تست rate limit
for i in $(seq 1 50); do curl -s -o /dev/null -w '%{http_code}\n' http://localhost:8081/login; done | sort | uniq -c

# تست traversal
curl -sS 'http://localhost:8081/?file=../../etc/passwd' | head -3
```

**مثال ۵: پروژه‌ی شخصی**

```
۱. یک سایت استاتیک با Nginx + TLS
۲. یک API کوچک (FastAPI) پشت Nginx
۳. کش ۵ ثانیه + rate limit
۴. مانیتورینگ با stub_status
۵. بکاپ روزانه
۶. deploy.sh با rollback
```

### مقایسه و الگوهای مختلف

**یادگیری عمیق در برابر پهن**

| | عمیق | پهن |
|---|---|---|
| تمرکز | Nginx/TLS | چند ابزار |
| سرعت | ⚠️ | ✅ |
| ارزش بلندمدت | ✅ | ⚠️ |
| مناسب | تخصص | مدیریت |

**مستندات در برابر وبلاگ در برابر کتاب**

| | مستندات | وبلاگ | کتاب |
|---|---|---|---|
| دقت | ✅✅ | ⚠️ | ✅ |
| عمق | ⚠️ | ⚠️ | ✅ |
| به‌روز | ✅✅ | ✅ | ❌ |

**گواهی‌نامه در برابر پروژه**

| | گواهی | پروژه |
|---|---|---|
| دانش | ✅ | ✅ |
| اثبات مهارت | ⚠️ | ✅✅ |
| هزینه | 💰 | ۰ |

**دیدگاه «وب‌سرور را با خواندن یاد می‌گیری»**: با اجرا. هر فصل، یک config واقعی تولید کند.

### اشکال شایع

**۱) خواندن بدون اجرا**
config با اجرا یاد گرفته می‌شود.

**۲) پریدن به K8s قبل از Nginx**
مفاهیم گم می‌شوند.

**۳) کپی config اینترنتی**
بدون درک. تست کن.

**۴) نبود پروژه‌ی واقعی**
دانش فراموش می‌شود.

**۵) ترس از تولید**
با staging و بکاپ، یاد می‌گیری.

**۶) تک‌منبعی بودن**
مستندات + وبلاگ + کتاب.

**۷) نادیده گرفتن لاگ**
لاگ، بهترین معلم.

**۸) نداشتن آزمایشگاه**
بدون VM/کانتینر، تمرین سخت است.

### نکات مهم (Tips & Tricks)

- **آزمایشگاه شخصی** با Docker.
- **هر هفته یک config جدید**.
- **`curl -w`** عادت.
- **`testssl.sh`** قبل از تحویل.
- **پروژه‌ی واقعی**.
- **RCA** بعد از حادثه.
- **تدریس** — بهترین تثبیت.

### خلاصه فصل

- چهار مسیر: DevOps، SRE، امنیت، معماری.
- نقشه‌ی ۱۲ ماه با خروجی مشخص.
- `High Performance Browser Networking` کتاب کلیدی.
- آزمایشگاه Docker برای تمرین.
- پروژه‌ی واقعی، ضروری است.
- تدریس، بهترین تثبیت.

**سؤال مروری:** اگر فقط ۱۰ ساعت در ماه برای یادگیری وب‌سرور داری، سه کار با بیشترین بازده چیست؟

---

## منابع و مراجع

**مستندات و مرجع**
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Apache HTTP Server 2.4 Docs](https://httpd.apache.org/docs/2.4/)
- [IIS Documentation](https://learn.microsoft.com/en-us/iis/)
- [MDN — HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)

**کتاب‌ها**
- *High Performance Browser Networking* — Ilya Grigorik — [hpbn.co](https://hpbn.co/) (رایگان)
- *Bulletproof SSL and TLS* — Ivan Ristić
- *NGINX Cookbook* — Tim DeJonghe (O'Reilly، رایگان)
- *Web Operations* — Allspaw & Robbins
- *Site Reliability Engineering* — Google

**وبلاگ و آموزش**
- [Nginx Blog](https://www.nginx.com/blog/)
- [Cloudflare Learning Center](https://www.cloudflare.com/learning/)
- [Mozilla Observatory](https://observatory.mozilla.org/)
- [Security Headers](https://securityheaders.com/)
- [SSL Labs](https://www.ssllabs.com/ssltest/)
- [WebPageTest](https://www.webpagetest.org/)

**ابزار**
- [testssl.sh](https://testssl.sh/)
- [hey](https://github.com/rakyll/hey)
- [k6](https://k6.io/)
- [GoAccess](https://goaccess.io/)
- [Mozilla SSL Config Generator](https://ssl-config.mozilla.org/)

**جامعه**
- [Nginx Forum](https://forum.nginx.org/)
- [Server Fault](https://serverfault.com/)
- [Stack Overflow — nginx](https://stackoverflow.com/questions/tagged/nginx)
- [Stack Overflow — apache](https://stackoverflow.com/questions/tagged/apache)

---

**پایان کتابچه.**

از [`README.md`](README.md) شروع کن، یا به فهرست کامل ۸۳ فصل در [`SUMMARY.md`](SUMMARY.md) برگرد.
