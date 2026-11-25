# بخش ۰۱: مبانی وب‌سرور و پروتکل HTTP

**مدت یادگیری**: ۴ تا ۵ ساعت
**سطح سختی**: مبتدی
**پیش‌نیاز**: آشنایی مقدماتی با خط فرمان لینوکس؛ مفهوم IP و پورت در سطح کلی

---

## فهرست محتوا

- [فصل ۱: وب‌سرور چیست و در مسیر یک درخواست کجا می‌نشیند](#فصل-۱)
- [فصل ۲: HTTP/1.1 — پیام، متد، کد وضعیت و هدر](#فصل-۲)
- [فصل ۳: HTTP/2 و HTTP/3 — چه چیزی واقعاً عوض شد](#فصل-۳)
- [فصل ۴: HTTPS، TLS و گواهی — از handshake تا مرورگر](#فصل-۴)

---

## فصل ۱: وب‌سرور چیست و در مسیر یک درخواست کجا می‌نشیند

### مقدمه

وقتی در مرورگر `https://example.com/` را می‌زنی، بین «فشردن Enter» و «دیدن صفحه»، حدود ۲۰۰ تا ۸۰۰ میلی‌ثانیه اتفاق می‌افتد: حل نام دامنه، برقراری اتصال TCP، دست‌دادن TLS، ارسال درخواست HTTP، پردازش در سرور، تولید پاسخ، انتقال، رندر. **وب‌سرور** همان قطعه‌ای است که درخواست HTTP را می‌گیرد و پاسخ را می‌سازد یا از جایی دیگر می‌آورد.

این فصل، نقشه‌ی کلی است. اگر این نقشه را در ذهن داشته باشی، هر خطای `502`، هر کندی، هر مشکل گواهی، جای مشخصی در نقشه پیدا می‌کند.

### مفاهیم اساسی

**۱) وب‌سرور دقیقاً چه می‌کند**

چهار کار اصلی:

| کار | مثال |
|---|---|
| پذیرش اتصال TCP روی پورت | `:80`، `:443` |
| تجزیه‌ی درخواست HTTP | متد، مسیر، هدرها، بدنه |
| تصمیم‌گیری | فایل استاتیک؟ پروکسی به اپ؟ ریدایرکت؟ خطا؟ |
| تولید پاسخ | وضعیت، هدر، بدنه (فایل، خروجی اپ، یا خطا) |

**۲) نقشه‌ی مسیر یک درخواست**

```
┌──────────┐
│ مرورگر   │
└────┬─────┘
     │ ۱. DNS: example.com → 203.0.113.10
     ▼
┌──────────────────┐
│ DNS Resolver     │
└────┬─────────────┘
     │ ۲. TCP handshake (SYN, SYN-ACK, ACK) → :443
     ▼
┌──────────────────────────────────────────────┐
│  لبه (CDN / فایروال / Load Balancer)          │
│  - TLS termination                            │
│  - WAF، rate limit                            │
└────┬─────────────────────────────────────────┘
     │ ۳. HTTP request (متن یا باینری)
     ▼
┌──────────────────────────────────────────────┐
│  وب‌سرور (Nginx / Apache / IIS)               │
│  ┌────────────────────────────────────────┐  │
│  │ server block / VirtualHost / Site      │  │
│  │  ├── فایل استاتیک؟ → از دیسک بخوان      │  │
│  │  ├── PHP؟ → FastCGI به PHP-FPM          │  │
│  │  └── اپ؟ → reverse proxy به :3000       │  │
│  └────────────────────────────────────────┘  │
└────┬─────────────────────────────────────────┘
     │ ۴. HTTP response
     ▼
   مرورگر → رندر
```

**۳) سه خانواده‌ی وب‌سرور**

| خانواده | مدل | نمونه |
|---|---|---|
| **Process/Thread-based** | هر اتصال یک پردازنده | Apache (prefork/worker) |
| **Event-driven** | یک حلقه، هزاران اتصال | Nginx، Caddy، LiteSpeed |
| **ماژول در سیستم‌عامل** | هسته‌ی HTTP + worker | IIS (`HTTP.sys`) |

**۴) وب‌سرور در برابر سرور اپلیکیشن**

- **وب‌سرور**: HTTP، فایل استاتیک، TLS، پروکسی، فشرده‌سازی، کش.
- **سرور اپلیکیشن**: منطق کسب‌وکار (PHP-FPM، Gunicorn، Node، Kestrel، Tomcat).

در معماری مدرن، وب‌سرور **جلوی** سرور اپلیکیشن می‌نشیند (reverse proxy). چرا؟ چون وب‌سرور در کارهایی که «داده‌ی ثابت و اتصال زیاد» است، بهتر است؛ و اپلیکیشن در منطق.

**۵) چه چیزی «سرور» نیست**

- `localhost:3000` که `npm run dev` بالا می‌آورد: یک سرور توسعه است، نه وب‌سرور تولیدی.
- `python -m http.server`: برای اشتراک سریع فایل، نه تولید.

**۶) چرا سه سرور مختلف وجود دارد**

| نیاز | انتخاب طبیعی |
|---|---|
| فایل استاتیک با ترافیک بالا | Nginx |
| `.htaccess` و کانفیگ بدون restart | Apache |
| اکوسیستم ویندوز/ASP.NET | IIS |
| سادگی و TLS خودکار | Caddy |

**۷) تعریف‌های پایه که باید بلد باشی**

| اصطلاح | معنا |
|---|---|
| **Origin** | `scheme + host + port` (مثلاً `https://example.com:443`) |
| **Virtual Host** | چند دامنه روی یک IP/سرور |
| **Document Root** | پوشه‌ای که فایل‌ها از آن سرو می‌شوند |
| **Upstream** | سروری که وب‌سرور به آن پروکسی می‌کند |
| **Worker** | پردازنده‌ای که درخواست را سرو می‌کند |
| **Keep-alive** | استفاده‌ی مجدد از اتصال TCP |

### مثال‌های عملی

**Lab Scenario: دیدن یک درخواست واقعی از نزدیک**

هدف: یک درخواست HTTP خام ببینی و پاسخ خام بگیری.

```bash
# ۱) درخواست خام با netcat (بدون هیچ ابزار اضافه)
printf 'GET / HTTP/1.1\r\nHost: example.com\r\nConnection: close\r\n\r\n' \
  | nc example.com 80 | head -30
```

خروجی نمونه:

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 1256
Connection: close
...
<!doctype html>
<html>
...
```

**تحلیل:** خط اول درخواست (`GET / HTTP/1.1`)، سپس هدرها، سپس یک خط خالی، سپس (اگر بدنه دارد) بدنه. پاسخ هم همان ساختار است.

```bash
# ۲) همان درخواست با curl و دیدن هدرها
curl -sS -D - -o /dev/null https://example.com
```

```bash
# ۳) دیدن زمان‌بندی هر مرحله
curl -sS -o /dev/null -w "
dns:        %{time_namelookup}s
connect:    %{time_connect}s
tls:        %{time_appconnect}s
ttfb:       %{time_starttransfer}s
total:      %{time_total}s
http:       %{http_code}
remote_ip:  %{remote_ip}
" https://example.com
```

این خروجی، نقشه‌ی فصل را عددی می‌کند: `dns` (فصل ۶)، `connect` (فصل ۵)، `tls` (فصل ۴)، `ttfb` (فصل ۱۲).

```bash
# ۴) یک وب‌سرور محلی در ۱۰ ثانیه
mkdir -p /tmp/lab && echo '<h1>سلام</h1>' > /tmp/lab/index.html
docker run --rm -d --name lab-web -p 8080:80 \
  -v /tmp/lab:/usr/share/nginx/html:ro nginx:1.26

curl -sS http://localhost:8080/
```

### مقایسه و الگوهای مختلف

**سه معماری، سه رفتار**

| معیار | Apache (prefork) | Nginx | IIS |
|---|---|---|---|
| مدل | پروسه به ازای اتصال | event loop | `HTTP.sys` + worker |
| حافظه در ۱۰٬۰۰۰ اتصال | ❌ زیاد | ✅ کم | ⚠️ متوسط |
| فایل استاتیک | ⚠️ متوسط | ✅ عالی | ✅ خوب |
| `.htaccess` | ✅ | ❌ | ⚠️ (`web.config`) |
| ماژول پویا | ✅ | ⚠️ (کامپایل) | ✅ |
| ویندوز | ✅ | ⚠️ (محدود) | ✅ بومی |

**سه الگوی استقرار**

1. **تک‌سرور همه‌کاره**: Nginx هم TLS، هم استاتیک، هم پروکسی. ساده، مناسب اکثر پروژه‌ها.
2. **دو لایه**: Nginx (لبه) + Apache/PHP-FPM (اپ). رایج در وردپرس.
3. **لبه‌ی ابری**: CDN + ALB + app. برای مقیاس بزرگ.

**دیدگاه «کدام وب‌سرور بهترین است؟»**: سؤال اشتباه. سؤال درست: «برای این بار و این تیم، کدام کم‌دردسرتر است؟» Nginx برای استاتیک و پروکسی، Apache برای انعطاف پیکربندی، IIS برای اکوسیستم مایکروسافت.

### اشکال شایع

**۱) اشتباه گرفتن «سرور اپ» با «وب‌سرور»**
`npm start` روی پورت ۳۰۰۰ را مستقیم به اینترنت باز می‌کنند. نتیجه: بدون TLS، بدون rate limit، بدون فشرده‌سازی، و یک باگ در اپ = کل سطح حمله. همیشه یک وب‌سرور جلوی آن بگذار.

**۲) تصور «Nginx فایل استاتیک را از رم می‌خواند»**
Nginx از `sendfile` و کش سیستم‌عامل استفاده می‌کند، ولی همه‌چیز در رم نیست. `open_file_cache` برای متادیتا هست، نه محتوا.

**۳) `DocumentRoot` اشتباه**
`root /var/www/html;` در حالی که فایل‌ها در `/var/www/html/public` هستند. نتیجه: `403` یا `404`. (→ فصل ۲۲)

**۴) باز کردن پورت اشتباه در فایروال**
`ufw allow 80` ولی سرور روی `8080` گوش می‌دهد. `ss -ltnp` بزن.

**۵) فراموش کردن `Host` header**
HTTP/1.1 بدون `Host`، پاسخ `400`. در `nc` دستی خیلی‌ها فراموش می‌کنند.

**۶) انتظار «سرور سریع، سایت سریع»**
TTFB بالا می‌تواند از اپ باشد، نه وب‌سرور. `curl -w` تفکیک می‌کند.

### نکات مهم (Tips & Tricks)

- **`curl -w`** بهترین ابزار اولیه برای فهم «کجا کند است».
- **`ss -ltnp`** ببین چه چیزی روی کدام پورت گوش می‌دهد.
- **`curl -v`** هدرهای درخواست و پاسخ را کامل نشان می‌دهد.
- **`curl --resolve`** برای تست دامنه بدون تغییر DNS:

```bash
curl -sS --resolve lab.example.com:443:192.168.56.10 https://lab.example.com/
```

- **`http://` و `https://` دو origin متفاوت‌اند** — کوکی، CORS و redirect بر همین اساس کار می‌کنند.
- **پورت ۸۰ و ۴۴۳ فقط قرارداد است**، نه قانون؛ ولی مرورگر برای `https://` همیشه ۴۴۳ را امتحان می‌کند.

### خلاصه فصل

- وب‌سرور: پذیرش TCP، تجزیه‌ی HTTP، تصمیم، تولید پاسخ.
- سه خانواده: process-based (Apache)، event-driven (Nginx)، ماژول هسته (IIS).
- سرور اپلیکیشن را مستقیم به اینترنت باز نکن؛ وب‌سرور جلوی آن بگذار.
- `curl -w` مرحله‌ی کند را نشان می‌دهد.
- `Origin` = scheme + host + port.

**سؤال مروری:** در مسیر یک درخواست `https://shop.example.com/product/12`، سه نقطه‌ای که می‌تواند باعث کندی شود کدام‌اند و با چه دستوری هر کدام را جدا اندازه می‌گیری؟

---

## فصل ۲: HTTP/1.1 — پیام، متد، کد وضعیت و هدر

### مقدمه

HTTP یک پروتکل **متنی، بی‌حالت (stateless)، درخواست-پاسخ** است. همین سادگی، دلیل موفقیتش است: با `nc` می‌توانی دستی با هر سروری در دنیا حرف بزنی. ولی همین سادگی، تله‌هایی دارد که سال‌ها طول می‌کشد تا یاد بگیری — از `Content-Length` و `chunked` تا `keep-alive` و `Host`.

### مفاهیم اساسی

**۱) ساختار پیام**

```
درخواست:                          پاسخ:
┌────────────────────────┐        ┌────────────────────────┐
│ METHOD /path HTTP/1.1  │        │ HTTP/1.1 200 OK        │
│ Header: value          │        │ Header: value          │
│ Header: value          │        │ Header: value          │
│ (خط خالی)              │        │ (خط خالی)              │
│ [body]                 │        │ [body]                 │
└────────────────────────┘        └────────────────────────┘
```

- خط پایان هدرها: `\r\n\r\n` (CRLF خالی).
- هدرها: `Name: value`، نام‌ها **حساس به حروف نیستند**، مقدارها هستند.

**۲) متدها**

| متد | کار | Idempotent | Safe | بدنه |
|---|---|---|---|---|
| `GET` | خواندن | ✅ | ✅ | معمولاً نه |
| `HEAD` | فقط هدر | ✅ | ✅ | ❌ |
| `POST` | ایجاد | ❌ | ❌ | ✅ |
| `PUT` | جایگزینی کامل | ✅ | ❌ | ✅ |
| `PATCH` | تغییر جزئی | ❌ | ❌ | ✅ |
| `DELETE` | حذف | ✅ | ❌ | اختیاری |
| `OPTIONS` | قابلیت‌ها/CORS | ✅ | ✅ | ❌ |
| `TRACE` | بازتاب درخواست | ✅ | ✅ | ❌ (امنیت: خاموش) |
| `CONNECT` | تونل (پروکسی) | ❌ | ❌ | ❌ |

**Idempotent** یعنی «تکرارش همان اثر یک‌بار را دارد». مهم است برای retry خودکار.

**۳) کدهای وضعیت — پنج خانواده**

| خانواده | معنا | نمونه |
|---|---|---|
| `1xx` | اطلاعاتی | `100 Continue`، `101 Switching Protocols` |
| `2xx` | موفق | `200 OK`، `201 Created`، `204 No Content`، `206 Partial Content` |
| `3xx` | تغییر مسیر | `301`، `302`، `304`، `307`، `308` |
| `4xx` | خطای کلاینت | `400`، `401`، `403`، `404`، `405`، `413`، `429` |
| `5xx` | خطای سرور | `500`، `502`، `503`، `504` |

کدهایی که در وب‌سرورها زیاد می‌بینی:

| کد | معنا | معمولاً از کجا |
|---|---|---|
| `301` | انتقال دائمی | `rewrite`/`return 301` |
| `302` | موقت | اپ |
| `304` | تغییر نکرده (کش) | `If-None-Match`/`If-Modified-Since` |
| `400` | درخواست بد | هدر نامعتبر، `Host` نبودن |
| `403` | ممنوع | مجوز فایل، `deny`، `autoindex off` |
| `404` | پیدا نشد | مسیر اشتباه، `root` غلط |
| `413` | بدنه‌ی بزرگ | `client_max_body_size` |
| `429` | درخواست زیاد | `limit_req` |
| `499` | کلاینت بست (Nginx) | timeout کلاینت |
| `500` | خطای داخلی | اپ، PHP fatal |
| `502` | پاسخ بد از upstream | اپ خاموش، سوکت غلط |
| `503` | سرویس ناموجود | maintenance، `limit_conn` |
| `504` | timeout upstream | اپ کند |

**۴) هدرهای مهم درخواست**

| هدر | کار |
|---|---|
| `Host` | دامنه‌ی مقصد (اجباری در HTTP/1.1) |
| `User-Agent` | کلاینت |
| `Accept` | نوع محتوای قابل قبول |
| `Accept-Encoding` | فشرده‌سازی قابل قبول (`gzip`, `br`) |
| `Accept-Language` | زبان |
| `Cookie` | کوکی‌ها |
| `Authorization` | اعتبارنامه (`Basic`، `Bearer`) |
| `If-None-Match` / `If-Modified-Since` | کش شرطی |
| `Range` | درخواست بخشی از فایل |
| `Referer` | صفحه‌ی مبدأ |
| `X-Forwarded-For` / `X-Real-IP` | IP اصلی پشت پروکسی (→ فصل ۳۴) |
| `Connection` | `keep-alive` یا `close` |

**۵) هدرهای مهم پاسخ**

| هدر | کار |
|---|---|
| `Content-Type` | نوع محتوا + charset |
| `Content-Length` | طول بدنه (یا `Transfer-Encoding: chunked`) |
| `Content-Encoding` | `gzip`، `br`، `zstd` |
| `Cache-Control` | سیاست کش |
| `ETag` / `Last-Modified` | اعتبارسنجی کش |
| `Location` | مقصد redirect |
| `Set-Cookie` | تنظیم کوکی |
| `Vary` | کش بر اساس کدام هدر (مثلاً `Accept-Encoding`) |
| `Strict-Transport-Security` | HSTS (→ فصل ۴۴) |
| `Server` | نام/نسخه‌ی سرور (بهتر: حذف یا مبهم) |

**۶) `keep-alive` و هزینه‌ی اتصال**

هر اتصال TCP یک `handshake` سه‌مرحله‌ای دارد (و در TLS، چند رفت‌وبرگشت بیشتر). `Connection: keep-alive` اجازه می‌دهد چند درخواست روی یک اتصال بروند.

```
بدون keep-alive:            با keep-alive:
conn1 → req1 → resp1 → ✂    conn1 → req1 → resp1
conn2 → req2 → resp2 → ✂           → req2 → resp2
conn3 → req3 → resp3 → ✂           → req3 → resp3 → ✂
```

ولی HTTP/1.1 فقط **یک درخواست در هر لحظه** روی هر اتصال دارد (head-of-line blocking). به همین دلیل مرورگرها ۶ اتصال موازی به هر دامنه باز می‌کنند. این محدودیت، دلیل اصلی تولد HTTP/2 است (فصل ۳).

**۷) `chunked` و `Content-Length`**

اگر طول بدنه از قبل معلوم نباشد (مثلاً خروجی استریم)، سرور از `Transfer-Encoding: chunked` استفاده می‌کند. اگر هر دو باشند، `Content-Length` برنده است و `chunked` نادیده گرفته می‌شود — که در پروکسی‌ها می‌تواند به `request smuggling` منجر شود.

**۸) `Range` — درخواست بخشی**

```http
GET /video.mp4 HTTP/1.1
Range: bytes=0-1023
```

پاسخ: `206 Partial Content` با `Content-Range: bytes 0-1023/10485760`. برای ویدیو، دانلود ادامه‌دار و PDF ضروری است. Nginx به‌طور پیش‌فرض پشتیبانی می‌کند.

### مثال‌های عملی

**Lab Scenario: یک درخواست کامل، دستی**

```bash
# درخواست با هدرهای واقعی
printf 'GET /index.html HTTP/1.1\r\nHost: lab.example.com\r\nAccept: */*\r\nConnection: close\r\n\r\n' \
  | nc lab.example.com 80
```

**تست کدهای وضعیت با Nginx:**

```nginx
# /etc/nginx/conf.d/lab-status.conf
server {
    listen 8080;
    server_name _;

    location /ok      { return 200 "OK\n"; }
    location /created { return 201 "Created\n"; }
    location /nocontent { return 204; }
    location /redirect { return 301 /ok; }
    location /temp    { return 302 /ok; }
    location /forbidden { return 403 "no\n"; }
    location /missing { return 404 "not found\n"; }
    location /toomany { return 429 "slow down\n"; }
    location /oops    { return 500 "server error\n"; }
    location /unavail { return 503 "maintenance\n"; }
}
```

```bash
nginx -t && nginx -s reload
for p in ok created nocontent redirect temp forbidden missing toomany oops unavail; do
  printf '%-12s → ' "$p"
  curl -s -o /dev/null -w '%{http_code}\n' "http://localhost:8080/$p"
done
```

**تست کش شرطی:**

```bash
# اولین درخواست
curl -sSI http://localhost:8080/index.html | grep -Ei 'etag|last-modified'

# درخواست با ETag
ETAG=$(curl -sSI http://localhost:8080/index.html | awk -F': ' 'tolower($1)=="etag"{print $2}' | tr -d '\r')
curl -sS -o /dev/null -w '%{http_code}\n' -H "If-None-Match: $ETAG" http://localhost:8080/index.html
# 304
```

**تست `Range`:**

```bash
curl -sS -r 0-99 -D - -o /dev/null http://localhost:8080/index.html | grep -Ei 'content-range|content-length'
# Content-Range: bytes 0-99/...
# HTTP/1.1 206 Partial Content
```

**بررسی هدرهای امنیتی موجود:**

```bash
curl -sSI https://example.com | grep -Ei 'strict-transport|content-security|x-frame|x-content-type|referrer-policy|permissions-policy'
```

### مقایسه و الگوهای مختلف

**HTTP/1.0 در برابر HTTP/1.1**

| | 1.0 | 1.1 |
|---|---|---|
| `Host` | اختیاری | **اجباری** |
| `keep-alive` | ❌ | ✅ پیش‌فرض |
| `chunked` | ❌ | ✅ |
| `Range` | ⚠️ | ✅ |
| `Pipelining` | ❌ | ⚠️ (عملاً استفاده نمی‌شود) |

**سه روش انتقال بدنه**

| روش | چه‌وقت | هدر |
|---|---|---|
| طول مشخص | فایل استاتیک | `Content-Length` |
| `chunked` | خروجی استریم | `Transfer-Encoding: chunked` |
| بستن اتصال | قدیمی/ساده | `Connection: close` |

**کدهای ۳۰۱ و ۳۰۲ و ۳۰۷ و ۳۰۸**

| کد | دائمی؟ | متد حفظ می‌شود؟ | کاربرد |
|---|---|---|---|
| `301` | ✅ | ⚠️ (مرورگرها `POST`→`GET`) | تغییر دامنه |
| `302` | ❌ | ⚠️ | ریدایرکت موقت |
| `307` | ❌ | ✅ | موقت با حفظ متد |
| `308` | ✅ | ✅ | دائمی با حفظ متد |

**نکته‌ی مهم:** برای `POST` که نباید به `GET` تبدیل شود، `307`/`308` بگذار، نه `301`/`302`.

### اشکال شایع

**۱) `301` برای `POST`**
مرورگر `POST` را به `GET` تبدیل می‌کند و بدنه از دست می‌رود. `308`.

**۲) `Host` نبودن**
HTTP/1.1 بدون `Host` → `400 Bad Request`. در تست دستی با `nc` فراموش می‌شود.

**۳) `Content-Length` اشتباه در پروکسی**
اگر پروکسی بدنه را تغییر دهد (مثلاً فشرده کند) ولی `Content-Length` را به‌روز نکند، پاسخ بریده می‌شود. Nginx خودش مدیریت می‌کند، ولی `proxy_set_header Content-Length ""` در بعضی سناریوها لازم است.

**۴) کش نکردن فایل استاتیک**
بدون `Cache-Control`/`ETag`، هر بازدید، دانلود کامل. (→ فصل ۵۰)

**۵) `Vary` فراموش‌شده با gzip**
کش میانی نسخه‌ی gzip را به کلاینتی می‌دهد که پشتیبانی نمی‌کند. `Vary: Accept-Encoding`.

**۶) `HEAD` که بدنه برمی‌گرداند**
برخی اپ‌ها `HEAD` را مثل `GET` پیاده می‌کنند. Nginx خودش بدنه را حذف می‌کند.

**۷) `TRACE` باز**
خطر `XST`. در Nginx/Apache/IIS غیرفعال کن.

**۸) `Server` با نسخه‌ی دقیق**
اطلاعات رایگان به مهاجم. `server_tokens off;`.

**۹) `OPTIONS` و CORS اشتباه**
دو `Access-Control-Allow-Origin` تکراری → مرورگر رد می‌کند.

### نکات مهم (Tips & Tricks)

- **`curl -sSI`** فقط هدرهای پاسخ (HEAD).
- **`curl -D -`** هدرها را روی خروجی استاندارد بده.
- **`curl --http1.1`** برای اجبار نسخه.
- **`curl -H 'Host: ...'`** برای تست virtual host بدون DNS.
- **`curl --compressed`** تا `Accept-Encoding` بفرستد و پاسخ فشرده را باز کند.
- **`curl -w '%{http_code} %{size_download} %{time_total}'`** خلاصه‌ی یک‌خطی.
- **`httpie`** اگر خروجی رنگی و خوانا می‌خواهی.
- **`nghttp`** برای تست HTTP/2.

### خلاصه فصل

- HTTP متنی، بی‌حالت، درخواست-پاسخ است.
- `Host` در HTTP/1.1 اجباری است.
- `301`/`302` متد را ممکن است عوض کنند؛ `307`/`308` نه.
- `keep-alive` هزینه‌ی اتصال را کم می‌کند، ولی head-of-line blocking دارد.
- `chunked` برای بدنه‌ی با طول نامعلوم؛ `Range` برای بخشی از فایل.
- `Vary: Accept-Encoding` با فشرده‌سازی الزامی است.

**سؤال مروری:** چرا `301` برای ریدایرکت یک `POST` خطرناک است و `308` چه تفاوتی دارد؟

---

## فصل ۳: HTTP/2 و HTTP/3 — چه چیزی واقعاً عوض شد

### مقدمه

HTTP/1.1 در ۱۹۹۷ استاندارد شد و محدودیت اصلی‌اش ساده است: **یک درخواست در هر اتصال در هر لحظه**. مرورگرها با باز کردن ۶ اتصال موازی به هر دامنه دورش زدند، ولی این راه‌حل، هزینه دارد (۶ handshake TLS، ۶ بار کندی شروع). HTTP/2 (۲۰۱۵) و HTTP/3 (۲۰۲۲) این محدودیت را از ریشه حل کردند.

### مفاهیم اساسی

**۱) مشکل HTTP/1.1: head-of-line blocking**

```
اتصال ۱: [req1 ████████][req2 ███][req3 ██████]
اتصال ۲: [req4 ██][req5 ██████████]
...
اتصال ۶: ...

اگر req1 کند باشد، req2 و req3 پشتش می‌مانند.
```

راه‌حل‌های عجیب دوران HTTP/1.1: `domain sharding` (چند زیردامنه)، `sprite sheet`، `concat` فایل‌ها، `inlining`. همه‌ی این‌ها **anti-pattern** در HTTP/2 هستند.

**۲) HTTP/2 — multiplexing**

```
یک اتصال TCP:
┌──────────────────────────────────────────┐
│ stream 1: [HEADERS][DATA....]            │
│ stream 3:    [HEADERS][DATA..]           │
│ stream 5:       [HEADERS][DATA.......]   │
│ stream 7:          [HEADERS][DATA.]      │
└──────────────────────────────────────────┘
```

| ویژگی | توضیح |
|---|---|
| **Multiplexing** | چند stream روی یک اتصال |
| **Binary framing** | باینری، نه متن (کمتر خطا، سریع‌تر پارس) |
| **Header compression (HPACK)** | هدرهای تکراری یک‌بار فرستاده می‌شوند |
| **Server Push** | سرور منابع را قبل از درخواست می‌فرستد (عملاً مرده) |
| **Prioritization** | اولویت streamها (در عمل کم‌استفاده) |
| **Stream reset** | لغو یک stream بدون بستن اتصال |

**۳) مشکل باقی‌مانده‌ی HTTP/2: TCP head-of-line blocking**

HTTP/2 روی **یک** اتصال TCP کار می‌کند. اگر یک بسته‌ی TCP گم شود، **همه‌ی** streamها منتظر retransmit می‌مانند. این دقیقاً همان مشکل، یک لایه پایین‌تر.

**۴) HTTP/3 — QUIC روی UDP**

```
HTTP/2:  HTTP → TLS → TCP → IP
HTTP/3:  HTTP → QUIC (شامل TLS 1.3) → UDP → IP
```

| ویژگی | مزیت |
|---|---|
| UDP به‌جای TCP | بدون HOL blocking در لایه‌ی انتقال |
| TLS 1.3 اجباری | handshake یک‌مرحله‌ای (0-RTT ممکن) |
| Connection ID | تغییر شبکه (Wi-Fi → LTE) بدون قطع اتصال |
| QPACK | فشرده‌سازی هدر، بدون HOL |
| پیاده‌سازی در userspace | به‌روزرسانی سریع‌تر |

**۵) کدام نسخه را استفاده کن**

| سناریو | توصیه |
|---|---|
| سایت عمومی | HTTP/2 + HTTP/3 (اگر ممکن) |
| API داخلی | HTTP/2 |
| فایل بزرگ | HTTP/2 حداقل |
| کلاینت قدیمی | HTTP/1.1 fallback |

**۶) `Alt-Svc` — چطور مرورگر HTTP/3 را پیدا می‌کند**

```http
Alt-Svc: h3=":443"; ma=86400
```

سرور روی HTTP/2 این هدر را می‌دهد؛ مرورگر دفعه‌ی بعد QUIC را امتحان می‌کند.

**۷) `ALPN` — انتخاب پروتکل در TLS**

در TLS handshake، کلاینت لیست پروتکل‌ها را می‌فرستد (`h2`, `http/1.1`) و سرور یکی را انتخاب می‌کند. اگر `h2` توافق نشود، `http/1.1`.

**۸) محدودیت‌های HTTP/2 که باید بدانی**

- **`Server Push` مرده است**: کروم در ۲۰۲۲ حذفش کرد. از `103 Early Hints` یا `preload` استفاده کن.
- **`domain sharding` مضر است**: همه‌ی منابع از یک دامنه بیایند.
- **`concat` مضر است**: cache granularity را خراب می‌کند.
- **`inlining` مضر است**: کش نمی‌شود.

### مثال‌های عملی

**Lab Scenario: فعال‌سازی HTTP/2 و HTTP/3 در Nginx**

```nginx
# /etc/nginx/conf.d/lab-http2.conf
server {
    listen 443 ssl;
    http2 on;                       # Nginx 1.25.1+ (قبلاً: listen 443 ssl http2;)
    server_name lab.example.com;

    ssl_certificate     /etc/letsencrypt/live/lab.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/lab.example.com/privkey.pem;
    ssl_protocols       TLSv1.2 TLSv1.3;

    add_header Alt-Svc 'h3=":443"; ma=86400' always;

    root  /var/www/lab;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

```bash
# تست
curl -sSI --http2 https://lab.example.com/ | head -3
# HTTP/2 200

# بررسی ALPN
openssl s_client -connect lab.example.com:443 -alpn h2,http/1.1 </dev/null 2>/dev/null \
  | grep -i "ALPN protocol"
# ALPN protocol: h2
```

**بررسی HTTP/3 با `curl` (نیاز به build با HTTP/3):**

```bash
curl --http3 -sSI https://lab.example.com/ | head -3
# HTTP/3 200
```

اگر `curl` پشتیبانی ندارد:

```bash
curl -V | grep -i http3
```

**مقایسه‌ی تعداد اتصال در HTTP/1.1 و HTTP/2:**

```bash
# HTTP/1.1: مرورگر ۶ اتصال باز می‌کند
curl -sS --http1.1 -o /dev/null -w 'version=%{http_version} connects=%{num_connects}\n' \
  https://lab.example.com/ https://lab.example.com/ https://lab.example.com/

# HTTP/2: یک اتصال
curl -sS --http2 -o /dev/null -w 'version=%{http_version} connects=%{num_connects}\n' \
  https://lab.example.com/ https://lab.example.com/ https://lab.example.com/
```

**Apache:**

```apache
# /etc/apache2/sites-available/lab-ssl.conf
<VirtualHost *:443>
    ServerName lab.example.com
    Protocols h2 http/1.1
    SSLEngine on
    SSLCertificateFile      /etc/letsencrypt/live/lab.example.com/fullchain.pem
    SSLCertificateKeyFile   /etc/letsencrypt/live/lab.example.com/privkey.pem
    DocumentRoot /var/www/lab
</VirtualHost>
```

```bash
sudo a2enmod http2 ssl
sudo apachectl configtest && sudo systemctl reload apache2
curl -sSI --http2 https://lab.example.com/ | head -1
```

**IIS (ویندوز سرور ۲۰۱۶+):**

```powershell
# HTTP/2 به‌طور پیش‌فرض روی HTTPS فعال است
# بررسی:
curl.exe -sSI --http2 https://lab.example.com/ | Select-Object -First 1

# غیرفعال/فعال‌سازی در رجیستری
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\HTTP\Parameters" `
  -Name "EnableHttp2Tls" -Value 1 -PropertyType DWord -Force
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\HTTP\Parameters" `
  -Name "EnableHttp2Cleartext" -Value 0 -PropertyType DWord -Force
```

### مقایسه و الگوهای مختلف

**سه نسخه، سه مدل**

| معیار | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---|---|---|---|
| انتقال | TCP | TCP | QUIC/UDP |
| Multiplexing | ❌ | ✅ | ✅ |
| HOL در انتقال | ✅ | ✅ (TCP) | ❌ |
| فشرده‌سازی هدر | ❌ | HPACK | QPACK |
| TLS | اختیاری | عملاً اجباری (مرورگر) | اجباری |
| handshake | ۲ RTT + TLS | ۲ RTT + TLS | ۱ RTT (یا 0-RTT) |
| پیچیدگی استقرار | ✅ | ✅ | ⚠️ (UDP، فایروال) |

**HTTP/2 در برابر HTTP/3 در عمل**

| | HTTP/2 | HTTP/3 |
|---|---|---|
| پشتیبانی مرورگر | ✅ کامل | ✅ کامل |
| فایروال/UDP | ✅ | ⚠️ (بعضی شبکه‌ها UDP را می‌بندند) |
| ابزار تست | ✅ | ⚠️ (`curl` با HTTP/3) |
| بهبود واقعی | ✅ بزرگ نسبت به 1.1 | ⚠️ متوسط در شبکه‌ی خوب، بزرگ در شبکه‌ی بد |

**دیدگاه «HTTP/2 همه‌ی مشکلات را حل کرد»**: نه. HOL در TCP باقی ماند و همین دلیل HTTP/3 است. ولی در شبکه‌ی با loss کم، تفاوت HTTP/2 و ۳ کم است.

**دیدگاه «HTTP/3 همیشه سریع‌تر است»**: در شبکه‌ی موبایل و loss بالا، بله. در دیتاسنتر، تفاوت ناچیز و پیچیدگی بیشتر است.

### اشکال شایع

**۱) `domain sharding` باقی‌مانده**
در HTTP/2، چند دامنه یعنی چند اتصال و از دست رفتن multiplexing. همه از یک دامنه.

**۲) `concat` و `sprite`**
کش را خراب می‌کند. در HTTP/2، فایل‌های کوچک جدا بهترند.

**۳) `Server Push` استفاده‌شده**
مرده. `103 Early Hints` یا `<link rel=preload>`.

**۴) `listen 443 ssl http2;` در Nginx جدید**
از ۱.۲۵.۱ منسوخ شده؛ `http2 on;` جدا. هشدار می‌دهد.

**۵) HTTP/2 بدون TLS**
مرورگرها `h2c` (cleartext) را پشتیبانی نمی‌کنند. HTTP/2 عملاً یعنی HTTPS.

**۶) UDP بسته در فایروال**
HTTP/3 کار نمی‌کند، ولی fallback به HTTP/2 بی‌صدا انجام می‌شود. تست کن، نه فرض.

**۷) `Alt-Svc` با پورت اشتباه**
مرورگر به پورت اشتباه QUIC می‌زند و شکست می‌خورد.

**۸) `proxy_pass` به upstream با HTTP/1.0**
Nginx به‌طور پیش‌فرض HTTP/1.0 به upstream می‌فرستد. برای `keepalive` و WebSocket، `proxy_http_version 1.1;` لازم است.

### نکات مهم (Tips & Tricks)

- **`curl --http2`** برای تست؛ `--http1.1` برای مقایسه.
- **`openssl s_client -alpn h2`** برای دیدن پروتکل توافق‌شده.
- **`nghttp -nv https://...`** جزئیات streamها را نشان می‌دهد.
- **`chrome://net-export`** برای دیباگ عمیق مرورگر.
- **`Alt-Svc`** با `ma` کوتاه در تست، بلند در تولید.
- **`http2 on;`** در Nginx 1.25.1+.
- **`proxy_http_version 1.1;`** برای upstream.

### خلاصه فصل

- HTTP/1.1: یک درخواست در هر اتصال، HOL blocking.
- HTTP/2: multiplexing، باینری، HPACK؛ ولی HOL در TCP باقی است.
- HTTP/3: QUIC روی UDP، بدون HOL، handshake سریع‌تر.
- `Server Push` مرده؛ `domain sharding` و `concat` مضرند.
- HTTP/2 عملاً نیاز به TLS دارد.
- `Alt-Svc` پل بین HTTP/2 و HTTP/3 است.

**سؤال مروری:** چرا HTTP/2 مشکل head-of-line blocking را کامل حل نکرد و HTTP/3 چه کاری متفاوت انجام داد؟

---

## فصل ۴: HTTPS، TLS و گواهی — از handshake تا مرورگر

### مقدمه

HTTPS یعنی HTTP روی TLS. TLS سه چیز می‌دهد: **محرمانگی** (کسی نمی‌تواند بخواند)، **یکپارچگی** (کسی نمی‌تواند تغییر دهد) و **احراز هویت** (سرور همان است که ادعا می‌کند). این فصل، از ریاضیات تا پیکربندی، ولی به زبان شهودی.

### مفاهیم اساسی

**۱) رمزنگاری متقارن و نامتقارن**

| نوع | کلید | سرعت | کاربرد در TLS |
|---|---|---|---|
| متقارن (AES) | یک کلید مشترک | ✅ سریع | رمزنگاری داده |
| نامتقارن (RSA/ECDSA) | جفت کلید | ❌ کند | تبادل کلید، امضا |

TLS هر دو را ترکیب می‌کند: با نامتقارن، کلید متقارن را رد و بدل می‌کند؛ بعد با متقارن، داده را سریع رمز می‌کند.

**۲) گواهی (Certificate) چیست**

یک فایل امضاشده که می‌گوید:

```
«کلید عمومی X متعلق به دامنه‌ی example.com است.»
امضا: CA (مرجع صدور گواهی)
اعتبار: از ... تا ...
```

مرورگر به CA اعتماد دارد (از طریق ریشه‌های نصب‌شده در سیستم‌عامل). اگر امضا معتبر باشد و دامنه مطابقت کند، قفل سبز/قفل معمولی نشان می‌دهد.

**۳) زنجیره‌ی اعتماد**

```
Root CA (در مرورگر/سیستم‌عامل)
  └── Intermediate CA
        └── گواهی دامنه‌ی شما (leaf)
```

اگر intermediate را در سرور نفرستی، بعضی کلاینت‌ها خطای «unable to verify» می‌دهند. به همین دلیل `fullchain.pem` استفاده می‌شود، نه فقط `cert.pem`.

**۴) انواع گواهی**

| نوع | اعتبارسنجی | کاربرد |
|---|---|---|
| **DV** (Domain Validated) | فقط کنترل دامنه | اکثر سایت‌ها، Let's Encrypt |
| **OV** (Organization Validated) | + سازمان | سازمانی |
| **EV** (Extended Validation) | + بررسی عمیق | بانک (ولی مرورگرها دیگر متمایز نمی‌کنند) |
| **Wildcard** | `*.example.com` | چند زیردامنه |
| **Multi-domain (SAN)** | چند دامنه در یک گواهی | چند سایت |

**۵) TLS handshake (نسخه ۱.۳، ساده‌شده)**

```
Client                                Server
  │                                      │
  │── ClientHello ──────────────────────▶│  (versions, ciphers, SNI, ALPN)
  │                                      │
  │◀─────────────── ServerHello ─────────│  (chosen version/cipher)
  │◀─────────────── {Certificate} ───────│
  │◀─────────────── {CertificateVerify} ─│
  │◀─────────────── {Finished} ──────────│
  │                                      │
  │── {Finished} ───────────────────────▶│
  │                                      │
  │◀═══════ Application Data ═══════════▶│
```

TLS 1.3 در **یک RTT** تمام می‌شود (در برابر دو RTT در 1.2). با `0-RTT` (session resumption) می‌تواند حتی بدون RTT شروع کند — به قیمت ریسک replay.

**۶) `SNI` — چطور یک IP چند گواهی دارد**

کلاینت در `ClientHello` نام دامنه را می‌فرستد (`SNI`). سرور گواهی مناسب را انتخاب می‌کند. بدون SNI، فقط یک گواهی روی آن IP ممکن بود.

**۷) `ALPN` — انتخاب HTTP/2**

کلاینت لیست پروتکل‌ها را می‌فرستد (`h2`, `http/1.1`)؛ سرور یکی را انتخاب می‌کند.

**۸) Cipher Suite**

مجموعه‌ای از الگوریتم‌ها: تبادل کلید + احراز هویت + رمزنگاری + MAC.

```
TLS_AES_256_GCM_SHA384
    └── AES-256-GCM (رمزنگاری) + SHA384 (هش)
```

در TLS 1.3، مجموعه‌ها ساده‌تر شدند و الگوریتم‌های ضعیف (RC4، 3DES، CBC) حذف.

**۹) `HSTS` — چطور از downgrade جلوگیری کنیم**

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

مرورگر بعد از دیدن این هدر، **هرگز** با `http://` به این دامنه وصل نمی‌شود.

**۱۰) `OCSP Stapling`**

سرور وضعیت ابطال گواهی را از CA می‌گیرد و به کلاینت می‌دهد؛ کلاینت لازم نیست به CA وصل شود (سریع‌تر، خصوصی‌تر).

### مثال‌های عملی

**Lab Scenario: از صفر تا HTTPS با Let's Encrypt**

```bash
# ۱) پیش‌نیاز: دامنه به IP سرور اشاره کند، پورت ۸۰ باز باشد
dig +short lab.example.com

# ۲) نصب certbot
sudo apt update && sudo apt install -y certbot python3-certbot-nginx

# ۳) گرفتن گواهی (Nginx خودش config را می‌سازد)
sudo certbot --nginx -d lab.example.com --redirect --agree-tos -m admin@example.com

# ۴) بررسی
sudo certbot certificates
curl -sSI https://lab.example.com/ | head -5
```

**پیکربندی دستی Nginx:**

```nginx
server {
    listen 80;
    server_name lab.example.com;
    return 301 https://$host$request_uri;      # اجبار HTTPS
}

server {
    listen 443 ssl;
    http2 on;
    server_name lab.example.com;

    ssl_certificate     /etc/letsencrypt/live/lab.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/lab.example.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/lab.example.com/chain.pem;

    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_ciphers         ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305;
    ssl_prefer_server_ciphers off;      # در TLS 1.3 بی‌اثر است
    ssl_session_cache   shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;

    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 1.1.1.1 8.8.8.8 valid=300s;
    resolver_timeout 5s;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    root /var/www/lab;
    index index.html;
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx
```

**Apache:**

```apache
<VirtualHost *:80>
    ServerName lab.example.com
    Redirect permanent / https://lab.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName lab.example.com
    SSLEngine on
    SSLCertificateFile    /etc/letsencrypt/live/lab.example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/lab.example.com/privkey.pem

    SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
    SSLCipherSuite ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384
    SSLHonorCipherOrder off
    SSLSessionTickets off

    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"

    DocumentRoot /var/www/lab
</VirtualHost>
```

```bash
sudo a2enmod ssl headers
sudo apachectl configtest && sudo systemctl reload apache2
```

**IIS (PowerShell):**

```powershell
# ساخت binding با گواهی
$cert = Get-ChildItem Cert:\LocalMachine\My | Where-Object { $_.Subject -like "*lab.example.com*" }
New-WebBinding -Name "Lab" -Protocol https -Port 443 -HostHeader "lab.example.com" -SslFlags 1
(Get-WebBinding -Name "Lab" -Protocol https).AddSslCertificate($cert.Thumbprint, "my")

# اجبار HTTPS با URL Rewrite
```

```xml
<!-- web.config -->
<system.webServer>
  <rewrite>
    <rules>
      <rule name="Force HTTPS" stopProcessing="true">
        <match url="(.*)" />
        <conditions>
          <add input="{HTTPS}" pattern="off" />
        </conditions>
        <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" redirectType="Permanent" />
      </rule>
    </rules>
  </rewrite>
</system.webServer>
```

**تست گواهی و پیکربندی:**

```bash
# ۱) زنجیره و تاریخ انقضا
echo | openssl s_client -connect lab.example.com:443 -servername lab.example.com 2>/dev/null \
  | openssl x509 -noout -subject -issuer -dates

# ۲) پروتکل‌ها
nmap --script ssl-enum-ciphers -p 443 lab.example.com

# ۳) ابزار جامع
docker run --rm -it drwetter/testssl.sh --quiet https://lab.example.com

# ۴) فقط خطاها
docker run --rm -it drwetter/testssl.sh --quiet --severity HIGH https://lab.example.com
```

**بررسی HSTS:**

```bash
curl -sSI https://lab.example.com/ | grep -i strict-transport
```

**تمدید خودکار:**

```bash
# certbot خودش timer می‌سازد
systemctl list-timers | grep certbot
sudo certbot renew --dry-run
```

### مقایسه و الگوهای مختلف

**TLS 1.2 در برابر TLS 1.3**

| | 1.2 | 1.3 |
|---|---|---|
| RTT | ۲ | ۱ |
| Cipher suite | پیچیده، شامل ضعیف | ساده، فقط AEAD |
| 0-RTT | ⚠️ | ✅ |
| Forward secrecy | ⚠️ (بستگی به suite) | ✅ اجباری |
| پشتیبانی | ✅ همه | ✅ مدرن |

**سه مدل صدور گواهی**

| مدل | هزینه | خودکار | مناسب |
|---|---|---|---|
| Let's Encrypt (ACME) | ۰ | ✅ | اکثر سایت‌ها |
| تجاری (DigiCert، Sectigo) | 💰 | ⚠️ | OV/EV، پشتیبانی |
| Self-signed | ۰ | ✅ | داخلی، تست |

**سه مدل پایان‌دهی TLS**

| مدل | توضیح | مناسب |
|---|---|---|
| TLS در وب‌سرور | ساده‌ترین | تک‌سرور |
| TLS در LB، HTTP به backend | رایج در ابر | مقیاس |
| mTLS انتها-به-انتها | امن‌ترین | سرویس‌به‌سرویس |

**دیدگاه «TLS کند است»**: در ۲۰۱۰ درست بود. با TLS 1.3، AES-NI و session resumption، هزینه‌ی TLS روی CPU معمولاً ۱ تا ۳٪ است — بسیار کمتر از هزینه‌ی عدم استفاده.

### اشکال شایع

**۱) فقط `cert.pem` فرستادن**
زنجیره ناقص → خطای «unable to verify» در بعضی کلاینت‌ها (مخصوصاً اندروید قدیمی). `fullchain.pem`.

**۲) فراموش کردن `ssl_certificate_key` با مجوز درست**
`chmod 600`، مالک `root`.

**۳) `ssl_protocols TLSv1 TLSv1.1;`**
ضعیف و غیرقابل قبول در انطباق. فقط `TLSv1.2 TLSv1.3`.

**۴) HSTS با `preload` قبل از تست**
اگر اشتباه پیکربندی کنی، ماه‌ها نمی‌توانی برگردی. اول بدون `preload`.

**۵) `includeSubDomains` بدون پوشش همه‌ی زیردامنه‌ها**
زیردامنه‌ای که HTTPS ندارد، از دسترس خارج می‌شود.

**۶) گواهی منقضی**
`certbot renew` شکست خورده و کسی ندیده. مانیتور انقضا.

**۷) `Mixed Content`**
صفحه HTTPS که منبع `http://` بار می‌کند. مرورگر بلاک می‌کند. `Content-Security-Policy: upgrade-insecure-requests`.

**۸) `SNI` و کلاینت قدیمی**
ویندوز XP / جاوا قدیمی بدون SNI، گواهی اشتباه می‌گیرند. امروز نادر است.

**۹) `ssl_stapling` بدون `resolver`**
خطا یا بی‌اثر. `resolver` لازم است.

**۱۰) redirect loop**
`http → https` در جایی که خودش پشت پروکسی HTTPS است و `X-Forwarded-Proto` را نمی‌خواند. (→ فصل ۳۴)

### نکات مهم (Tips & Tricks)

- **`fullchain.pem`** همیشه.
- **`TLSv1.2 TLSv1.3`** فقط.
- **`testssl.sh`** قبل از تحویل.
- **`certbot renew --dry-run`** ماهانه.
- **`openssl s_client -servername`** برای تست SNI.
- **`HSTS` را مرحله‌ای** (اول `max-age=300`، بعد بلند).
- **`ssl_session_tickets off`** برای forward secrecy بهتر.
- **`OCSP stapling`** با `resolver`.

### خلاصه فصل

- TLS: محرمانگی + یکپارچگی + احراز هویت.
- گواهی، کلید عمومی را به دامنه گره می‌زند؛ زنجیره تا Root CA.
- `fullchain.pem` برای زنجیره‌ی کامل.
- TLS 1.3 در یک RTT، با forward secrecy اجباری.
- HSTS از downgrade جلوگیری می‌کند، ولی برگشت‌ناپذیر است.
- `testssl.sh` و `openssl s_client` ابزارهای تأیید.

**سؤال مروری:** اگر سرور فقط `cert.pem` (بدون intermediate) بفرستد، چرا بعضی مرورگرها خطا می‌دهند و بعضی نه؟

---

## منابع و مراجع

**مستندات رسمی**
- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
- [RFC 9112 — HTTP/1.1](https://www.rfc-editor.org/rfc/rfc9112.html)
- [RFC 9113 — HTTP/2](https://www.rfc-editor.org/rfc/rfc9113.html)
- [RFC 9114 — HTTP/3](https://www.rfc-editor.org/rfc/rfc9114.html)
- [RFC 8446 — TLS 1.3](https://www.rfc-editor.org/rfc/rfc8446.html)
- [MDN — HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [Nginx — HTTP/2 module](https://nginx.org/en/docs/http/ngx_http_v2_module.html)
- [Apache — HTTP/2 Guide](https://httpd.apache.org/docs/2.4/howto/http2.html)

**آموزش و ابزار**
- [High Performance Browser Networking — Ilya Grigorik](https://hpbn.co/) — رایگان و مرجع
- [Cloudflare Learning Center — What is HTTP/3?](https://www.cloudflare.com/learning/performance/what-is-http3/)
- [testssl.sh](https://testssl.sh/)
- [SSL Labs Server Test](https://www.ssllabs.com/ssltest/)
- [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/)
- [Let's Encrypt Docs](https://letsencrypt.org/docs/)

---

**بعدی:** [`02-Networking-Basics.md`](02-Networking-Basics.md) — TCP، پورت، DNS، پروکسی و WebSocket
