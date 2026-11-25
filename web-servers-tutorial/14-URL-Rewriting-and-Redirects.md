# بخش ۱۴: بازنویسی URL و Redirect

**مدت یادگیری**: ۴ تا ۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۰۴ (Apache)، بخش ۰۵ (Nginx `location`)

---

## فهرست محتوا

- [فصل ۶۰: `mod_rewrite` در Apache و `rewrite`/`return` در Nginx](#فصل-۶۰)
- [فصل ۶۱: Redirect، اسلش پایانی و canonical](#فصل-۶۱)
- [فصل ۶۲: SPA و الگوی front controller](#فصل-۶۲)
- [فصل ۶۳: URL Rewrite در IIS](#فصل-۶۳)

---

## فصل ۶۰: `mod_rewrite` در Apache و `rewrite`/`return` در Nginx

### مقدمه

بازنویسی URL، قدرتمندترین و خطرناک‌ترین بخش پیکربندی است. یک قاعده‌ی اشتباه، حلقه‌ی بی‌پایان یا ۴۰۴ برای همه‌چیز می‌سازد. تفاوت مهم: **`rewrite` داخلی** (کاربر نمی‌بیند) در برابر **`redirect`** (مرورگر مسیر جدید را می‌بیند).

### مفاهیم اساسی

**۱) تفاوت بنیادی**

```
Rewrite (داخلی):  /old  →  /new     (URL مرورگر: /old)
Redirect:         /old  →  301 /new (URL مرورگر: /new)
```

**۲) Apache `mod_rewrite` — اجزا**

```apache
RewriteEngine On
RewriteBase /
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^blog/(.*)$ index.php?slug=$1 [L,QSA]
```

| جزء | کار |
|---|---|
| `RewriteEngine` | فعال‌سازی |
| `RewriteBase` | پایه (در `.htaccess`) |
| `RewriteCond` | شرط |
| `RewriteRule` | قاعده |
| `[L]` | توقف |
| `[QSA]` | الحاق query string |
| `[R=301]` | redirect |
| `[NC]` | غیرحساس به حروف |
| `[F]` | 403 |
| `[P]` | پروکسی |

**۳) `%{...}` در `RewriteCond`**

| متغیر | معنا |
|---|---|
| `%{REQUEST_URI}` | مسیر |
| `%{QUERY_STRING}` | query |
| `%{HTTP_HOST}` | دامنه |
| `%{HTTPS}` | `on`/`off` |
| `%{REQUEST_FILENAME}` | مسیر فایل |
| `%{HTTP_USER_AGENT}` | UA |
| `%{REMOTE_ADDR}` | IP |
| `%{REQUEST_METHOD}` | متد |

**۴) پرچم‌ها**

| پرچم | معنا |
|---|---|
| `L` | last (توقف) |
| `END` | توقف کامل (بدون بازنویسی بیشتر) |
| `QSA` | الحاق query |
| `R=301` | redirect دائمی |
| `R=302` | موقت |
| `NC` | case-insensitive |
| `F` | forbidden |
| `G` | gone |
| `P` | proxy |
| `NE` | no escape |
| `S=2` | skip 2 rules |
| `C` | chain |

**۵) Nginx — `return` در برابر `rewrite`**

```nginx
# return: سریع‌تر، بدون regex (اگر ممکن)
return 301 https://example.com$request_uri;
return 200 "ok\n";
return 404;

# rewrite: برای regex
rewrite ^/old/(.*)$ /new/$1 permanent;
```

`permanent` = 301، `last` = توقف، `break` = توقف بدون تغییر location.

**۶) `rewrite` در `server` در برابر `location`**

- در `server`: قبل از انتخاب `location`.
- در `location`: بعد از انتخاب.

**۷) `try_files` به‌جای `rewrite`**

```nginx
# ❌ rewrite برای SPA
rewrite ^ /index.html last;

# ✅ try_files
try_files $uri $uri/ /index.html;
```

`try_files` سریع‌تر و امن‌تر است.

**۸) `if` — «شیطانی»**

```nginx
# ❌ if در location
if ($host = www.example.com) { rewrite ^ https://example.com$request_uri permanent; }

# ✅ server جدا
server { listen 80; server_name www.example.com; return 301 https://example.com$request_uri; }
```

**۹) `$request_uri` در برابر `$uri`**

| متغیر | محتوا |
|---|---|
| `$request_uri` | مسیر کامل + query (خام) |
| `$uri` | مسیر نرمال‌شده (بدون query) |

برای redirect، `$request_uri` (حفظ query).

**۱۰) `break` در برابر `last`**

| | `break` | `last` |
|---|---|---|
| توقف rewrite | ✅ | ✅ |
| انتخاب location جدید | ❌ | ✅ |
| مناسب | داخل location | server |

### مثال‌های عملی

**Lab Scenario: قواعد رایج در هر دو سرور**

**۱) HTTPS اجباری — Apache:**

```apache
<VirtualHost *:80>
    ServerName lab.example.com
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
</VirtualHost>
```

**Nginx:**

```nginx
server {
    listen 80;
    server_name lab.example.com;
    return 301 https://$host$request_uri;
}
```

**۲) حذف `www` — Apache:**

```apache
RewriteEngine On
RewriteCond %{HTTP_HOST} ^www\.(.+)$ [NC]
RewriteRule ^ https://%1%{REQUEST_URI} [L,R=301]
```

**Nginx:**

```nginx
server {
    listen 443 ssl;
    server_name www.lab.example.com;
    return 301 https://lab.example.com$request_uri;
}
```

**۳) URL تمیز — Apache:**

```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^([a-z0-9-]+)/?$ index.php?page=$1 [L,QSA]
```

**Nginx:**

```nginx
location / {
    try_files $uri $uri/ @app;
}

location @app {
    rewrite ^/([a-z0-9-]+)/?$ /index.php?page=$1 last;
}
```

**۴) حذف `.php` از URL — Apache:**

```apache
RewriteCond %{REQUEST_FILENAME}.php -f
RewriteRule ^(.+)$ $1.php [L]
```

**Nginx:**

```nginx
location / {
    try_files $uri $uri.php $uri/ =404;
}
```

**۵) بلاک مسیر — Apache:**

```apache
RewriteRule ^(\.git|\.env) - [F,L]
```

**Nginx:**

```nginx
location ~ /\.(?!well-known) { deny all; }
```

**۶) redirect مسیر قدیمی — Apache:**

```apache
RewriteRule ^old-blog/(.*)$ /blog/$1 [R=301,L]
```

**Nginx:**

```nginx
location /old-blog/ {
    rewrite ^/old-blog/(.*)$ /blog/$1 permanent;
}
```

**تست:**

```bash
curl -sSI http://lab.example.com/old-blog/post-1 | grep -Ei 'HTTP/|location'
# HTTP/1.1 301
# Location: https://lab.example.com/blog/post-1
```

**۷) دیباگ `mod_rewrite`:**

```apache
LogLevel alert rewrite:trace3
```

```bash
sudo tail -f /var/log/apache2/error.log | grep rewrite
# [rewrite:trace2] ... applying pattern '^blog/(.*)$' to uri 'blog/post-1'
# [rewrite:trace3] ... [perdir /var/www/lab/] strip per-dir prefix: blog/post-1 -> blog/post-1
# [rewrite:trace1] ... [perdir /var/www/lab/] internal redirect with /index.php [INTERNAL REDIRECT]
```

**۸) دیباگ Nginx:**

```nginx
rewrite_log on;
error_log /var/log/nginx/rewrite.log notice;
```

```bash
sudo tail -f /var/log/nginx/rewrite.log
# *1 "^/old/(.*)$" matches "/old/x", client: 127.0.0.1
```

**۹) `QSA` در عمل:**

```apache
RewriteRule ^search$ /search.php [L,QSA]
# /search?q=test → /search.php?q=test
```

**۱۰) `NE` برای `#`:**

```apache
RewriteRule ^tag/(.*)$ /tags?name=$1 [R=301,NE,L]
# بدون NE، # به %23 تبدیل می‌شود
```

### مقایسه و الگوهای مختلف

**Apache `mod_rewrite` در برابر Nginx `rewrite`**

| | Apache | Nginx |
|---|---|---|
| شرط | `RewriteCond` | `if`/`map` |
| regex | PCRE | PCRE |
| `[L]` | `last` | `last` |
| redirect | `[R=301]` | `permanent` |
| داخلی | پیش‌فرض | `last`/`break` |
| `.htaccess` | ✅ | ❌ |

**`return` در برابر `rewrite` در Nginx**

| | `return` | `rewrite` |
|---|---|---|
| سرعت | ✅✅ | ✅ |
| regex | ❌ | ✅ |
| query | `$request_uri` | `$args` |
| توصیه | ✅ اگر ممکن | regex |

**`break` در برابر `last`**

| | `break` | `last` |
|---|---|---|
| location جدید | ❌ | ✅ |
| کاربرد | داخل `location` | `server` |
| خطر | ⚠️ حلقه | ⚠️ حلقه |

**دیدگاه «`mod_rewrite` برای همه‌چیز»**: `RewriteRule` آخرین راه است. `Redirect`، `Alias`، `FallbackResource` ساده‌تر و سریع‌ترند.

### اشکال شایع

**۱) حلقه‌ی بی‌پایان**
`RewriteRule ^(.*)$ /index.php` بدون شرط `!-f`. `[L]` و شرط.

**۲) `RewriteBase` اشتباه**
مسیر دوبار. `/`.

**۳) `QSA` فراموش‌شده**
query از دست می‌رود.

**۴) `%{REQUEST_URI}` در `RewriteRule`**
باید `%{REQUEST_URI}` در `Cond` و `$1` در `Rule`.

**۵) `if` در Nginx**
رفتار غیرمنتظره. `map`/`server` جدا.

**۶) `rewrite` بدون `last`**
ادامه‌ی پردازش.

**۷) `$uri` به‌جای `$request_uri` در redirect**
query از دست می‌رود.

**۸) `[R=301]` برای تست**
مرورگر کش می‌کند. `302` برای تست.

**۹) `NE` نبودن**
`#` escape می‌شود.

**۱۰) `rewrite_log` در تولید**
حجم لاگ. فقط دیباگ.

### نکات مهم (Tips & Tricks)

- **`return`** به‌جای `rewrite` در Nginx.
- **`try_files`** به‌جای rewrite برای SPA.
- **`[L,QSA]`** در Apache.
- **`$request_uri`** در redirect.
- **`302` برای تست**، `301` برای تولید.
- **`rewrite_log`** برای دیباگ.
- **`RewriteCond %{REQUEST_FILENAME} !-f`**.

### خلاصه فصل

- rewrite داخلی، redirect قابل مشاهده.
- Apache با `RewriteCond`/`RewriteRule`، Nginx با `return`/`rewrite`.
- `return` سریع‌تر از `rewrite`.
- `try_files` برای SPA.
- `[L,QSA]` و `$request_uri` را فراموش نکن.
- حلقه، شایع‌ترین خطاست.

**سؤال مروری:** تفاوت `rewrite ... last` و `return 301` در Nginx چیست؟

---

## فصل ۶۱: Redirect، اسلش پایانی و canonical

### مقدمه

یک صفحه با دو URL، مشکل SEO و analytics است. canonical، یک URL را «رسمی» می‌کند و بقیه را redirect.

### مفاهیم اساسی

**۱) کدهای redirect**

| کد | دائمی | متد | کش |
|---|---|---|---|
| `301` | ✅ | ممکن است `GET` شود | ✅ |
| `302` | ❌ | ممکن است `GET` شود | ⚠️ |
| `303` | ❌ | `GET` | ❌ |
| `307` | ❌ | حفظ | ❌ |
| `308` | ✅ | حفظ | ✅ |

**۲) canonical در HTML**

```html
<link rel="canonical" href="https://lab.example.com/product/123">
```

مکمل redirect، نه جایگزین.

**۳) اسلش پایانی**

```
/product/123   در برابر   /product/123/
```

انتخاب یکی و redirect دیگری:

```nginx
# حذف اسلش
server {
    listen 443 ssl;
    server_name lab.example.com;
    rewrite ^/(.*)/$ /$1 permanent;
}
```

```apache
RewriteRule ^(.*)/$ /$1 [R=301,L]
```

**مراقب:** برای پوشه‌ها، اسلش لازم است. قاعده را دقیق کن.

**۴) `www` در برابر non-www**

```nginx
# non-www رسمی
server {
    listen 443 ssl;
    server_name www.lab.example.com;
    return 301 https://lab.example.com$request_uri;
}
```

**۵) حروف بزرگ/کوچک**

```
/Product/123 → /product/123
```

```nginx
# در صورت نیاز (نادر)
```

> بهتر: URLها را از ابتدا lowercase بساز.

**۶) `trailing slash` با query**

```
/search/?q=x  →  /search?q=x
```

```apache
RewriteCond %{REQUEST_URI} ^(.+)/$
RewriteRule ^(.+)/$ /$1 [R=301,L,QSA]
```

**۷) `HSTS` و redirect**

`HSTS` بعد از اولین بازدید HTTPS، مرورگر را از HTTP بازمی‌دارد. redirect برای بازدید اول.

**۸) `canonical` در سرور**

```nginx
# دامنه‌ی اصلی
server {
    listen 443 ssl;
    server_name lab.example.com www.lab.example.com;
    # ...
}
```

یا redirect.

**۹) `X-Robots-Tag` برای صفحه‌های موقت**

```nginx
add_header X-Robots-Tag "noindex" always;
```

**۱۰) `410 Gone`**

```nginx
location /old-product/ {
    return 410;
}
```

بهتر از `404` برای محتوای حذف‌شده (سریع‌تر از ایندکس حذف می‌شود).

### مثال‌های عملی

**Lab Scenario: canonical کامل**

```nginx
# ۱) HTTP → HTTPS
server {
    listen 80;
    server_name lab.example.com www.lab.example.com;
    return 301 https://lab.example.com$request_uri;
}

# ۲) www → non-www
server {
    listen 443 ssl;
    http2 on;
    server_name www.lab.example.com;

    ssl_certificate     /etc/letsencrypt/live/lab.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/lab.example.com/privkey.pem;

    return 301 https://lab.example.com$request_uri;
}

# ۳) اصلی
server {
    listen 443 ssl;
    http2 on;
    server_name lab.example.com;

    ssl_certificate     /etc/letsencrypt/live/lab.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/lab.example.com/privkey.pem;

    root /var/www/lab;

    # حذف اسلش پایانی (به‌جز ریشه)
    location ~ ^(.+)/$ {
        return 301 $1$is_args$args;
    }

    location / {
        try_files $uri $uri/ =404;
    }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx

for u in \
  "http://lab.example.com/product/1" \
  "https://www.lab.example.com/product/1" \
  "https://lab.example.com/product/1/" \
  "https://lab.example.com/product/1/?a=1"
do
  printf '%-45s → ' "$u"
  curl -sSI "$u" | awk 'BEGIN{IGNORECASE=1} /^HTTP|^location/{printf "%s ", $0} END{print ""}'
done
```

خروجی:

```
http://lab.example.com/product/1               → HTTP/1.1 301  Location: https://lab.example.com/product/1
https://www.lab.example.com/product/1          → HTTP/1.1 301  Location: https://lab.example.com/product/1
https://lab.example.com/product/1/             → HTTP/1.1 301  Location: https://lab.example.com/product/1
https://lab.example.com/product/1/?a=1         → HTTP/1.1 301  Location: https://lab.example.com/product/1?a=1
```

**Apache معادل:**

```apache
<VirtualHost *:80>
    ServerName lab.example.com
    ServerAlias www.lab.example.com
    Redirect permanent / https://lab.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName www.lab.example.com
    SSLEngine on
    # ...
    Redirect permanent / https://lab.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName lab.example.com
    DocumentRoot /var/www/lab

    RewriteEngine On

    # حذف اسلش پایانی
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^(.+)/$ /$1 [R=301,L,QSA]

    <Directory /var/www/lab>
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>
```

**canonical در HTML:**

```html
<!doctype html>
<html lang="fa">
<head>
  <meta charset="utf-8">
  <link rel="canonical" href="https://lab.example.com/product/123">
  <meta property="og:url" content="https://lab.example.com/product/123">
</head>
```

**redirect صفحه‌های حذف‌شده:**

```nginx
# محصول حذف‌شده → 410
location ~ ^/product/(999|1000)$ {
    return 410;
}

# مسیر قدیمی → جدید
location /old-category/ {
    rewrite ^/old-category/(.*)$ /category/$1 permanent;
}
```

**`X-Robots-Tag` برای staging:**

```nginx
server {
    listen 443 ssl;
    server_name staging.lab.example.com;

    add_header X-Robots-Tag "noindex, nofollow" always;
    # ...
}
```

**تست کامل canonical:**

```bash
#!/usr/bin/env bash
BASE=https://lab.example.com

check() {
  local url=$1 expect=$2
  code=$(curl -sS -o /dev/null -w '%{http_code}' "$url")
  loc=$(curl -sSI "$url" | awk 'BEGIN{IGNORECASE=1} /^location:/{print $2}' | tr -d '\r')
  printf '%-45s %s %s\n' "$url" "$code" "$loc"
}

check "http://lab.example.com/x" 301
check "https://www.lab.example.com/x" 301
check "https://lab.example.com/x/" 301
check "https://lab.example.com/x" 200
```

### مقایسه و الگوهای مختلف

**`301` در برابر `302`**

| | `301` | `302` |
|---|---|---|
| دائمی | ✅ | ❌ |
| کش مرورگر | ✅ | ⚠️ |
| مناسب | canonical | تست/موقت |

**redirect در سرور در برابر `meta refresh`**

| | سرور | meta |
|---|---|---|
| سرعت | ✅ | ❌ |
| SEO | ✅ | ⚠️ |
| توصیه | ✅ | ❌ |

**canonical در HTML در برابر redirect**

| | canonical | redirect |
|---|---|---|
| URL مرورگر | تغییر نمی‌کند | تغییر می‌کند |
| SEO | ✅ | ✅ |
| analytics | ⚠️ (دو URL) | ✅ |
| توصیه | مکمل | ✅ اصلی |

**دیدگاه «redirect زیاد بد است»**: هر redirect یک RTT اضافه است. زنجیره‌ی ۳ redirect، ۳ RTT. تا حد ممکن یک redirect.

### اشکال شایع

**۱) زنجیره‌ی redirect**
`http → https → www → /path/`. یک قاعده.

**۲) `301` برای تست**
کش مرورگر، برگشت سخت. `302`.

**۳) اسلش پایانی برای پوشه**
`/dir` → `/dir/` حذف می‌شود، پوشه ۴۰۴. شرط `!-d`.

**۴) `www` بدون گواهی**
`www` به HTTPS می‌رود، گواهی ندارد. SAN.

**۵) `Location` نسبی**
بعضی کلاینت‌ها مشکل دارند. مطلق.

**۶) `canonical` و redirect متناقض**
`canonical` به URL دیگری که redirect می‌شود. یکی.

**۷) `X-Robots-Tag` روی staging فراموش‌شده**
ایندکس staging. `noindex`.

**۸) `410` به‌جای `404` برای محتوای موقت**
حذف دائمی از ایندکس. `404`.

**۹) حروف بزرگ/کوچک**
دو URL. lowercase.

**۱۰) `RewriteRule` با `[R]` بدون کد**
`302` پیش‌فرض. صریح `301`.

### نکات مهم (Tips & Tricks)

- **یک redirect** برای هر زنجیره.
- **`301`** برای canonical، **`302`** برای تست.
- **`$request_uri`** برای حفظ query.
- **`!-d`** برای اسلش پوشه.
- **`canonical`** مکمل.
- **`410`** برای حذف دائمی.
- **`X-Robots-Tag`** برای staging.

### خلاصه فصل

- redirect مرورگر را می‌برد، rewrite نه.
- `301` دائمی، `302` موقت.
- canonical یک URL رسمی می‌سازد.
- زنجیره‌ی redirect را حذف کن.
- اسلش پایانی را یکدست کن.
- `410` برای حذف دائمی.

**سؤال مروری:** چرا `301` برای تست می‌تواند ساعت‌ها وقت تلف کند؟

---

## فصل ۶۲: SPA و الگوی front controller

### مقدمه

در SPA (React/Vue/Angular)، مسیریابی در مرورگر است، نه سرور. اگر سرور `/dashboard` را نشناسد، `404` می‌دهد. راه‌حل: همه‌ی مسیرهای غیرفایل به `index.html`.

### مفاهیم اساسی

**۱) الگوی fallback**

```
/dashboard     → فایل نیست → index.html
/api/users     → API (proxy)
/assets/app.js → فایل
```

**۲) Nginx**

```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

**۳) Apache**

```apache
<Directory /var/www/spa>
    RewriteEngine On
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^ index.html [L]
</Directory>
```

یا ساده‌تر:

```apache
FallbackResource /index.html
```

**۴) IIS**

```xml
<rule name="SPA" stopProcessing="true">
  <match url=".*" />
  <conditions logicalGrouping="MatchAll">
    <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
    <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
  </conditions>
  <action type="Rewrite" url="/index.html" />
</rule>
```

**۵) جداسازی API از SPA**

```nginx
server {
    root /var/www/spa;

    location /api/ {
        proxy_pass http://127.0.0.1:8000/;
    }

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

**مهم:** `location /api/` قبل از `location /` بررسی می‌شود (پیشوندی بلندتر). ولی اگر regex داشته باشی، مراقب باش.

**۶) front controller (PHP)**

```
/index.php?route=/product/123
```

```nginx
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```

```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.php [L,QSA]
```

**۷) `404` واقعی برای API**

SPA fallback نباید API را ببلعد:

```nginx
location /api/ {
    proxy_pass http://127.0.0.1:8000/;
    # بدون try_files → 404 از اپ
}
```

**۸) `base href` در SPA**

اگر در زیرمسیر deploy می‌کنی:

```html
<base href="/app/">
```

```nginx
location /app/ {
    alias /var/www/spa/;
    try_files $uri $uri/ /app/index.html;
}
```

**۹) کش `index.html`**

```nginx
location = /index.html {
    add_header Cache-Control "no-cache" always;
}
```

فایل‌های hash‌دار، کش بلند.

**۱۰) `history mode` در برابر `hash mode`**

| | history | hash |
|---|---|---|
| URL | `/dashboard` | `/#/dashboard` |
| نیاز به fallback | ✅ | ❌ |
| SEO | ✅ | ⚠️ |

### مثال‌های عملی

**Lab Scenario: SPA + API + استاتیک**

```bash
sudo mkdir -p /var/www/spa/assets
cat > /var/www/spa/index.html <<'HTML'
<!doctype html>
<html lang="fa">
<head>
  <meta charset="utf-8">
  <title>SPA</title>
  <link rel="stylesheet" href="/assets/app.3f2a9c.css">
</head>
<body>
  <div id="app"></div>
  <script src="/assets/app.9b1d4e.js"></script>
</body>
</html>
HTML

echo 'body{font-family:sans-serif}' > /var/www/spa/assets/app.3f2a9c.css
echo 'console.log("spa")' > /var/www/spa/assets/app.9b1d4e.js
sudo chown -R www-data:www-data /var/www/spa
```

```nginx
upstream spa_api {
    server 127.0.0.1:8000;
    keepalive 16;
}

server {
    listen 8080;
    server_name spa.lab.example.com;
    root /var/www/spa;
    index index.html;

    # API
    location /api/ {
        proxy_pass http://spa_api/;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # استاتیک hash‌دار
    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        try_files $uri =404;
        access_log off;
    }

    # HTML بدون کش
    location = /index.html {
        add_header Cache-Control "no-cache" always;
    }

    # fallback
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx

# تست
curl -sS -o /dev/null -w 'root:   %{http_code}\n' http://localhost:8080/
curl -sS -o /dev/null -w 'route:  %{http_code}\n' http://localhost:8080/dashboard
curl -sS -o /dev/null -w 'asset:  %{http_code}\n' http://localhost:8080/assets/app.3f2a9c.css
curl -sS -o /dev/null -w 'api404: %{http_code}\n' http://localhost:8080/api/missing
# 200, 200, 200, 404 (از اپ)
```

**Apache:**

```apache
<VirtualHost *:8080>
    ServerName spa.lab.example.com
    DocumentRoot /var/www/spa

    ProxyPass        /api/ http://127.0.0.1:8000/
    ProxyPassReverse /api/ http://127.0.0.1:8000/

    <Directory /var/www/spa>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted

        RewriteEngine On
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteCond %{REQUEST_FILENAME} !-d
        RewriteRule ^ index.html [L]
    </Directory>

    <Directory /var/www/spa/assets>
        Header set Cache-Control "public, max-age=31536000, immutable"
    </Directory>
</VirtualHost>
```

**IIS:**

```xml
<system.webServer>
  <rewrite>
    <rules>
      <rule name="API" stopProcessing="true">
        <match url="^api/(.*)" />
        <action type="Rewrite" url="http://127.0.0.1:8000/{R:1}" />
      </rule>
      <rule name="SPA" stopProcessing="true">
        <match url=".*" />
        <conditions logicalGrouping="MatchAll">
          <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
          <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
        </conditions>
        <action type="Rewrite" url="/index.html" />
      </rule>
    </rules>
  </rewrite>
</system.webServer>
```

**PHP front controller:**

```nginx
server {
    listen 8080;
    server_name app.lab.example.com;
    root /var/www/app/public;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
    }

    location ~ /\.(?!well-known) { deny all; }
}
```

**تست fallback:**

```bash
for p in / /dashboard /users/42 /settings/profile; do
  printf '%-20s → ' "$p"
  curl -sS -o /dev/null -w '%{http_code}\n' "http://localhost:8080$p"
done
```

**تست عدم بلعیدن API:**

```bash
curl -sS -i http://localhost:8080/api/nonexistent | head -5
# باید 404 از اپ، نه index.html
```

### مقایسه و الگوهای مختلف

**`try_files` در برابر `rewrite` برای SPA**

| | `try_files` | `rewrite` |
|---|---|---|
| سرعت | ✅ | ⚠️ |
| امنیت | ✅ | ⚠️ |
| خوانایی | ✅ | ⚠️ |
| توصیه | ✅ | ❌ |

**`FallbackResource` در برابر `RewriteRule` (Apache)**

| | `FallbackResource` | `RewriteRule` |
|---|---|---|
| سادگی | ✅✅ | ⚠️ |
| انعطاف | ⚠️ | ✅ |
| سرعت | ✅ | ✅ |

**history در برابر hash**

| | history | hash |
|---|---|---|
| URL | ✅ تمیز | ⚠️ |
| fallback | ✅ لازم | ❌ |
| SEO | ✅ | ⚠️ |

**دیدگاه «SPA نیازی به وب‌سرور ندارد»**: `npm start` برای توسعه است. تولید: وب‌سرور برای TLS، کش، فشرده‌سازی، API proxy.

### اشکال شایع

**۱) fallback بدون `!-f`**
همه‌چیز `index.html`، حتی CSS.

**۲) fallback برای API**
`404` API به `index.html`. `location` جدا.

**۳) کش `index.html`**
نسخه‌ی قدیمی. `no-cache`.

**۴) asset بدون hash با `immutable`**
نسخه‌ی قدیمی تا ۱ سال.

**۵) `base href` اشتباه**
مسیرهای نسبی خراب.

**۶) `location /` قبل از `/api/`**
regex می‌تواند API را بگیرد. `^~`.

**۷) `try_files` با `alias`**
مسیر غلط. `root`.

**۸) `index.html` در `location /assets/`**
فایل ناموجود، `index.html` برمی‌گردد. `=404`.

**۹) `rewrite` بی‌نهایت**
`/index.html` خودش fallback می‌شود. `try_files`.

**۱۰) `X-Robots-Tag` روی SPA**
ایندکس نشدن. مراقب.

### نکات مهم (Tips & Tricks)

- **`try_files $uri $uri/ /index.html`**.
- **`location /api/` جدا**.
- **`no-cache` برای HTML**.
- **`immutable` برای hash**.
- **`^~` برای API**.
- **`FallbackResource`** در Apache.
- **`base href`** برای زیرمسیر.

### خلاصه فصل

- SPA به fallback نیاز دارد.
- `try_files` بهترین ابزار.
- API را از fallback جدا کن.
- HTML بدون کش، asset با کش دائمی.
- `hash` در نام فایل، شرط `immutable`.
- `FallbackResource` ساده‌ترین در Apache.

**سؤال مروری:** چرا `try_files $uri $uri/ /index.html` بدون `!-f` خطرناک است؟

---

## فصل ۶۳: URL Rewrite در IIS

### مقدمه

IIS برای بازنویسی، ماژول جداگانه‌ی `URL Rewrite` دارد. سینتکس XML است و در `web.config` می‌نشیند. برای ASP.NET Core و SPA، ضروری است.

### مفاهیم اساسی

**۱) نصب**

```
https://www.iis.net/downloads/microsoft/url-rewrite
```

```powershell
Get-WebGlobalModule | Where-Object Name -eq "RewriteModule"
```

**۲) ساختار**

```xml
<system.webServer>
  <rewrite>
    <rules>
      <rule name="..." stopProcessing="true">
        <match url="pattern" />
        <conditions>
          <add input="{...}" pattern="..." negate="false" />
        </conditions>
        <action type="Redirect|Rewrite|AbortRequest|CustomResponse"
                url="..." redirectType="Permanent|Found|SeeOther|Temporary" />
      </rule>
    </rules>
  </rewrite>
</system.webServer>
```

**۳) `match url` — بدون `/` اول**

```
درخواست: /product/123
match url: "product/([0-9]+)"    ← بدون /
{R:1} = 123
```

**۴) `{R:n}` و `{C:n}`**

| متغیر | معنا |
|---|---|
| `{R:0}` | کل تطبیق |
| `{R:1}` | گروه اول |
| `{C:1}` | گروه شرط |
| `{HTTP_HOST}` | دامنه |
| `{HTTPS}` | `on`/`off` |
| `{REQUEST_URI}` | مسیر |
| `{QUERY_STRING}` | query |
| `{REQUEST_FILENAME}` | مسیر فایل |

**۵) `type`**

| نوع | معنا |
|---|---|
| `Rewrite` | داخلی |
| `Redirect` | 301/302 |
| `AbortRequest` | بستن |
| `CustomResponse` | پاسخ سفارشی |

**۶) `redirectType`**

| مقدار | کد |
|---|---|
| `Permanent` | 301 |
| `Found` | 302 |
| `SeeOther` | 303 |
| `Temporary` | 307 |

**۷) `stopProcessing`**

`true` = قاعده‌های بعدی اجرا نشوند.

**۸) `serverVariables`**

```xml
<serverVariables>
  <set name="HTTP_X_FORWARDED_PROTO" value="{HTTPS}" />
</serverVariables>
```

نیاز به `allowedServerVariables`:

```powershell
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/rewrite/allowedServerVariables" `
  -Name "." -Value @{name='HTTP_X_FORWARDED_PROTO'}
```

**۹) `outboundRules`**

برای بازنویسی پاسخ (مثلاً لینک‌های داخلی).

**۱۰) `rewriteMaps`**

```xml
<rewrite>
  <rewriteMaps>
    <rewriteMap name="OldUrls">
      <add key="/old-a" value="/new-a" />
      <add key="/old-b" value="/new-b" />
    </rewriteMap>
  </rewriteMaps>
  <rules>
    <rule name="Map" stopProcessing="true">
      <match url=".*" />
      <conditions>
        <add input="{OldUrls:{REQUEST_URI}}" pattern="(.+)" />
      </conditions>
      <action type="Redirect" url="{C:1}" redirectType="Permanent" />
    </rule>
  </rules>
</rewrite>
```

### مثال‌های عملی

**Lab Scenario: قواعد کامل IIS**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    <rewrite>
      <rules>

        <!-- ۱) HTTPS -->
        <rule name="Force HTTPS" stopProcessing="true">
          <match url="(.*)" />
          <conditions>
            <add input="{HTTPS}" pattern="off" />
          </conditions>
          <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" redirectType="Permanent" />
        </rule>

        <!-- ۲) حذف www -->
        <rule name="Remove WWW" stopProcessing="true">
          <match url="(.*)" />
          <conditions>
            <add input="{HTTP_HOST}" pattern="^www\.(.+)$" />
          </conditions>
          <action type="Redirect" url="https://{C:1}/{R:1}" redirectType="Permanent" />
        </rule>

        <!-- ۳) API (قبل از SPA) -->
        <rule name="API Proxy" stopProcessing="true">
          <match url="^api/(.*)" />
          <action type="Rewrite" url="http://127.0.0.1:8000/{R:1}" />
        </rule>

        <!-- ۴) SPA fallback -->
        <rule name="SPA" stopProcessing="true">
          <match url=".*" />
          <conditions logicalGrouping="MatchAll">
            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
            <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
          </conditions>
          <action type="Rewrite" url="/index.html" />
        </rule>

      </rules>
    </rewrite>

    <httpProtocol>
      <customHeaders>
        <remove name="X-Powered-By" />
        <add name="X-Content-Type-Options" value="nosniff" />
      </customHeaders>
    </httpProtocol>

    <security>
      <requestFiltering removeServerHeader="true" />
    </security>
  </system.webServer>
</configuration>
```

```powershell
iisreset /noforce

# تست
curl.exe -sSI http://lab.example.com/ | Select-Object -First 3
curl.exe -sS -o NUL -w "%{http_code}`n" http://lab.example.com/dashboard
```

**بلاک مسیر:**

```xml
<rule name="Block Sensitive" stopProcessing="true">
  <match url="^\.(git|env|htaccess)" />
  <action type="AbortRequest" />
</rule>
```

**پاسخ سفارشی:**

```xml
<rule name="Maintenance" stopProcessing="true">
  <match url=".*" />
  <conditions>
    <add input="{REQUEST_URI}" pattern="^/health" negate="true" />
  </conditions>
  <action type="CustomResponse" statusCode="503"
          statusReason="Maintenance" statusDescription="Site is under maintenance" />
</rule>
```

**`rewriteMap` برای redirectهای زیاد:**

```xml
<rewrite>
  <rewriteMaps>
    <rewriteMap name="Legacy">
      <add key="/p/1" value="/product/100" />
      <add key="/p/2" value="/product/200" />
    </rewriteMap>
  </rewriteMaps>
  <rules>
    <rule name="Legacy Redirect" stopProcessing="true">
      <match url=".*" />
      <conditions>
        <add input="{Legacy:{REQUEST_URI}}" pattern="(.+)" />
      </conditions>
      <action type="Redirect" url="{C:1}" redirectType="Permanent" />
    </rule>
  </rules>
</rewrite>
```

**ASP.NET Core و `UsePathBase`:**

```csharp
app.UsePathBase("/app");
```

```xml
<rule name="PathBase" stopProcessing="true">
  <match url="^app/(.*)" />
  <action type="Rewrite" url="app/{R:1}" />
</rule>
```

**دیباگ:**

```powershell
# failed request tracing
Install-WindowsFeature Web-Http-Tracing
Set-WebConfigurationProperty -PSPath "IIS:\Sites\lab" `
  -Filter "system.webServer/tracing/traceFailedRequests" -Name "." `
  -Value @{path='C:\inetpub\logs\FailedReqLogs'}
```

**بررسی config:**

```powershell
Get-WebConfiguration -PSPath "IIS:\Sites\lab" -Filter "system.webServer/rewrite/rules" |
  Select-Object -ExpandProperty Collection | Select-Object name, stopProcessing
```

### مقایسه و الگوهای مختلف

**IIS URL Rewrite در برابر Apache `mod_rewrite`**

| | IIS | Apache |
|---|---|---|
| سینتکس | XML | دستور |
| `match url` | بدون `/` | با `/` |
| گروه | `{R:1}` | `$1` |
| شرط | `conditions` | `RewriteCond` |
| map | ✅ | ⚠️ |

**`Rewrite` در برابر `Redirect` در IIS**

| | `Rewrite` | `Redirect` |
|---|---|---|
| URL مرورگر | تغییر نمی‌کند | تغییر می‌کند |
| کارایی | ✅ | ⚠️ |
| مناسب | SPA، proxy | canonical |

**`AbortRequest` در برابر `CustomResponse`**

| | `AbortRequest` | `CustomResponse` |
|---|---|---|
| پاسخ | بستن اتصال | کد + متن |
| مناسب | بلاک | maintenance |

**دیدگاه «IIS Rewrite پیچیده است»**: XML است، ولی همان مفاهیم Apache را دارد. برای SPA و redirect، ساده است.

### اشکال شایع

**۱) `match url` با `/` اول**
تطبیق نمی‌کند. بدون `/`.

**۲) ترتیب قاعده‌ها**
SPA قبل از API → API بلعیده می‌شود. API اول.

**۳) `stopProcessing` نبودن**
قاعده‌های بعدی هم اجرا. `true`.

**۴) `serverVariables` بدون `allowed`**
`500.50`. `allowedServerVariables`.

**۵) `redirectType` پیش‌فرض**
`302`. برای canonical، `Permanent`.

**۶) `AbortRequest` برای فایل حساس**
اتصال بسته، بدون لاگ. `CustomResponse` با 404.

**۷) `rewriteMap` با کلید اشتباه**
تطبیق نمی‌کند. `{REQUEST_URI}` با `/`.

**۸) `outboundRules` برای همه‌ی پاسخ‌ها**
CPU. محدود کن.

**۹) `UsePathBase` و rewrite**
دوبار. یکی.

**۱۰) `web.config` در زیرپوشه**
قواعد ارث می‌رسند. `<clear />` اگر لازم.

### نکات مهم (Tips & Tricks)

- **`match url` بدون `/`**.
- **ترتیب: API قبل از SPA**.
- **`stopProcessing="true"`**.
- **`allowedServerVariables`**.
- **`rewriteMap`** برای redirect زیاد.
- **`failed request tracing`**.
- **`AbortRequest`** برای بلاک.

### خلاصه فصل

- URL Rewrite ماژول جدا برای IIS.
- `match url` بدون `/` اول.
- `{R:n}`/`{C:n}` گروه‌ها.
- ترتیب قاعده‌ها حیاتی است.
- `rewriteMap` برای مقیاس.
- `AbortRequest` برای بلاک.

**سؤال مروری:** چرا `match url="^/api/(.*)"` در IIS هیچ‌وقت تطبیق نمی‌کند؟

---

## منابع و مراجع

**مستندات رسمی**
- [Apache — mod_rewrite](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html)
- [Apache — RewriteRule Flags](https://httpd.apache.org/docs/2.4/rewrite/flags.html)
- [Nginx — ngx_http_rewrite_module](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html)
- [Nginx — try_files](https://nginx.org/en/docs/http/ngx_http_core_module.html#try_files)
- [IIS — URL Rewrite Module](https://learn.microsoft.com/en-us/iis/extensions/url-rewrite-module/url-rewrite-module-configuration-reference)
- [RFC 9110 — Redirection](https://www.rfc-editor.org/rfc/rfc9110.html#name-redirection-3xx)

**آموزش**
- [Apache mod_rewrite cheat sheet](https://httpd.apache.org/docs/2.4/rewrite/remapping.html)
- [Nginx rewrite vs return](https://www.nginx.com/blog/creating-nginx-rewrite-rules/)
- [Canonical URLs — Google](https://developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls)

**ابزار**
- [Redirect Checker](https://httpstatus.io/)
- [rewrite tester](https://htaccess.madewithlove.com/)

---

**بعدی:** [`15-High-Availability.md`](15-High-Availability.md) — Keepalived، VIP، Ingress و CDN
