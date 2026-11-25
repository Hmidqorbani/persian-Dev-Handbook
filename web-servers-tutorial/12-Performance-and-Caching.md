# بخش ۱۲: کارایی، کش و فشرده‌سازی

**مدت یادگیری**: ۶ تا ۷ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: بخش ۰۸ (پروکسی)، بخش ۱۱ (لاگ و متریک)

---

## فهرست محتوا

- [فصل ۵۰: کش — مرورگر، پروکسی و micro-cache](#فصل-۵۰)
- [فصل ۵۱: `proxy_cache` و `fastcgi_cache` در Nginx](#فصل-۵۱)
- [فصل ۵۲: فشرده‌سازی — `gzip`، Brotli و `zstd`](#فصل-۵۲)
- [فصل ۵۳: HTTP/2 و HTTP/3 در عمل](#فصل-۵۳)
- [فصل ۵۴: تنظیم — `keepalive`، بافر، worker و `fd`](#فصل-۵۴)

---

## فصل ۵۰: کش — مرورگر، پروکسی و micro-cache

### مقدمه

سریع‌ترین درخواست، درخواستی است که فرستاده نمی‌شود. کش، این را ممکن می‌کند. ولی کش اشتباه، بدترین باگ است: کاربر داده‌ی قدیمی یا داده‌ی کاربر دیگر را می‌بیند.

### مفاهیم اساسی

**۱) چهار لایه‌ی کش**

```
مرورگر → CDN → Reverse Proxy (micro-cache) → اپ → DB
  ↑        ↑         ↑                          ↑
  کش      کش        کش                        query cache (ندارد)
```

| لایه | کنترل |
|---|---|
| مرورگر | `Cache-Control`، `ETag` |
| CDN | هدرها + تنظیمات CDN |
| Proxy | `proxy_cache` |
| اپ | Redis/Memcached |

**۲) هدرهای کش**

| هدر | معنا |
|---|---|
| `Cache-Control: max-age=3600` | کش ۱ ساعت |
| `Cache-Control: no-store` | هرگز کش نکن |
| `Cache-Control: no-cache` | کش کن، ولی هر بار تأیید بگیر |
| `Cache-Control: private` | فقط مرورگر |
| `Cache-Control: public` | هر کش میانی |
| `Cache-Control: immutable` | هرگز تغییر نمی‌کند |
| `ETag` | شناسه‌ی نسخه |
| `Last-Modified` | زمان تغییر |
| `Vary` | کش بر اساس کدام هدر |
| `Expires` | قدیمی، `Cache-Control` مقدم |

**۳) `no-cache` در برابر `no-store`**

| | `no-cache` | `no-store` |
|---|---|---|
| ذخیره | ✅ | ❌ |
| تأیید | ✅ هر بار | — |
| مناسب | HTML داینامیک | داده‌ی حساس |

**۴) کش شرطی**

```
کلاینت: GET /app.css
        If-None-Match: "abc123"

سرور:   304 Not Modified   (اگر ETag یکی بود)
```

صرفه‌جویی: بدنه فرستاده نمی‌شود.

**۵) `Vary` — چرا حیاتی**

```http
Vary: Accept-Encoding
```

بدون آن، کش نسخه‌ی gzip را به کلاینتی می‌دهد که پشتیبانی نمی‌کند.

`Vary: Cookie` = کش تقریباً بی‌فایده (هر کاربر متفاوت).

**۶) `stale-while-revalidate`**

```http
Cache-Control: max-age=60, stale-while-revalidate=300
```

نسخه‌ی قدیمی فوراً، بازسازی در پس‌زمینه. تجربه‌ی کاربری عالی.

**۷) `stale-if-error`**

```http
Cache-Control: max-age=60, stale-if-error=86400
```

اگر اپ خراب شد، نسخه‌ی کش‌شده تا ۲۴ ساعت سرو شود.

**۸) micro-cache**

کش کوتاه (۱–۵ ثانیه) روی HTML داینامیک:

```nginx
proxy_cache_valid 200 5s;
```

با ۱۰۰۰ درخواست/ثانیه، بار اپ ۱۰۰۰ برابر کم می‌شود.

**۹) کش استاتیک با hash در نام**

```
/app.3f2a9c.css   → max-age=31536000, immutable
```

تغییر فایل = تغییر نام = کش جدید.

**۱۰) `Cache-Control` پیشنهادی**

| نوع | هدر |
|---|---|
| HTML | `no-cache` (یا `max-age=0, must-revalidate`) |
| CSS/JS با hash | `public, max-age=31536000, immutable` |
| تصویر | `public, max-age=2592000` |
| API عمومی | `public, max-age=60, stale-while-revalidate=300` |
| API خصوصی | `private, no-store` |

### مثال‌های عملی

**Lab Scenario: کش سه‌لایه**

```nginx
server {
    listen 8080;
    server_name lab.example.com;
    root /var/www/lab;

    # ۱) HTML: همیشه تأیید
    location = /index.html {
        add_header Cache-Control "no-cache" always;
        add_header ETag "" always;
    }

    # ۲) استاتیک با hash: کش دائمی
    location ~* \.[0-9a-f]{8}\.(css|js)$ {
        expires 1y;
        add_header Cache-Control "public, max-age=31536000, immutable";
        access_log off;
    }

    # ۳) تصویر: ۳۰ روز
    location ~* \.(jpg|jpeg|png|gif|webp|svg|ico)$ {
        expires 30d;
        add_header Cache-Control "public";
        access_log off;
    }

    # ۴) فونت: ۱ سال
    location ~* \.(woff2?|ttf|otf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        add_header Access-Control-Allow-Origin "*";
        access_log off;
    }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx

curl -sSI http://localhost:8080/index.html | grep -Ei 'cache-control|etag'
curl -sSI http://localhost:8080/app.3f2a9c.css | grep -Ei 'cache-control|expires'
```

**کش شرطی:**

```bash
ETAG=$(curl -sSI http://localhost:8080/index.html | awk -F': ' 'tolower($1)=="etag"{print $2}' | tr -d '\r')
echo "etag=$ETAG"

curl -sS -o /dev/null -w '%{http_code}\n' -H "If-None-Match: $ETAG" http://localhost:8080/index.html
# 304
```

**micro-cache برای API:**

```nginx
proxy_cache_path /var/cache/nginx/micro levels=1:2 keys_zone=micro:10m
                 max_size=200m inactive=10m use_temp_path=off;

server {
    location /api/public/ {
        proxy_cache micro;
        proxy_cache_valid 200 5s;
        proxy_cache_lock on;
        proxy_cache_use_stale updating error timeout;
        proxy_cache_background_update on;
        add_header X-Cache-Status $upstream_cache_status always;
        proxy_pass http://127.0.0.1:8000/;
    }
}
```

```bash
# اولین: MISS
curl -sSI http://localhost:8080/api/public/stats | grep -i x-cache-status
# MISS

# بعدی: HIT
curl -sSI http://localhost:8080/api/public/stats | grep -i x-cache-status
# HIT

# بعد از ۵ ثانیه: STALE/REVALIDATED
sleep 6
curl -sSI http://localhost:8080/api/public/stats | grep -i x-cache-status
```

**کش امن (بدون افشا):**

```nginx
location /api/me {
    proxy_cache off;                  # داده‌ی کاربر، کش نکن
    proxy_no_cache 1;
    proxy_cache_bypass 1;
    proxy_pass http://127.0.0.1:8000/me;
}
```

یا:

```nginx
map $http_authorization $no_cache {
    default 1;
    ""      0;
}

proxy_cache_bypass $no_cache;
proxy_no_cache     $no_cache;
```

**`Vary` با gzip:**

```nginx
gzip_vary on;
```

```bash
curl -sSI -H 'Accept-Encoding: gzip' http://localhost:8080/app.css | grep -i vary
# Vary: Accept-Encoding
```

**تست کش با `curl`:**

```bash
# بدون کش
curl -sS -H 'Cache-Control: no-cache' -D - -o /dev/null http://localhost:8080/app.css | grep -i cache

# با کش
curl -sS -D - -o /dev/null http://localhost:8080/app.css | grep -iE 'cache-control|expires|etag'
```

### مقایسه و الگوهای مختلف

**چهار استراتژی کش**

| استراتژی | هدر | مناسب |
|---|---|---|
| دائمی | `immutable, max-age=1y` | فایل با hash |
| بلند | `max-age=30d` | تصویر |
| کوتاه | `max-age=60` | API عمومی |
| بدون کش | `no-store` | داده‌ی کاربر |

**`no-cache` در برابر `must-revalidate`**

| | `no-cache` | `must-revalidate` |
|---|---|---|
| کش | ✅ | ✅ |
| تأیید | ✅ همیشه | بعد از انقضا |
| مناسب | HTML پویا | استاتیک با نسخه |

**micro-cache در برابر کش کامل**

| | micro (۵s) | کامل (۱h) |
|---|---|---|
| تازگی | ✅ | ❌ |
| بار اپ | ⚠️ | ✅✅ |
| مناسب | داشبورد، فید | محتوای ثابت |

**دیدگاه «کش همه‌چیز»**: داده‌ی کاربر و پاسخ احراز هویت‌شده را کش نکن. `private, no-store`.

### اشکال شایع

**۱) کش HTML با `max-age` بلند**
کاربر تغییر نمی‌بیند. `no-cache`.

**۲) کش API احراز هویت‌شده**
افشای داده. `private, no-store` یا `proxy_cache_bypass`.

**۳) `Vary` فراموش‌شده با gzip**
پاسخ اشتباه. `gzip_vary on`.

**۴) `Vary: Cookie`**
کش بی‌فایده. `Vary` حداقلی.

**۵) `Expires` و `Cache-Control` متناقض**
`Cache-Control` برنده است، ولی گیج‌کننده. یکی.

**۶) کش استاتیک بدون نسخه**
فایل جدید، کش قدیمی. hash در نام.

**۷) `immutable` برای فایل بدون hash**
کاربر تا ۱ سال نسخه‌ی قدیمی می‌بیند.

**۸) کش `POST`**
معمولاً کش نمی‌شود. `proxy_cache_methods GET HEAD`.

**۹) کش `Set-Cookie`**
پاسخ با کوکی، کش نشود. `proxy_ignore_headers Set-Cookie` ممنوع.

**۱۰) `proxy_cache_key` پیش‌فرض**
`$scheme$proxy_host$request_uri` — ممکن است کوکی/هدر را نادیده بگیرد. صریح تنظیم کن.

### نکات مهم (Tips & Tricks)

- **`immutable`** فقط با hash.
- **`no-cache`** برای HTML.
- **`Vary: Accept-Encoding`**.
- **`X-Cache-Status`** برای دیباگ.
- **`stale-while-revalidate`** برای UX.
- **`proxy_cache_bypass`** برای داده‌ی کاربر.
- **hash در نام فایل** برای کش دائمی.

### خلاصه فصل

- چهار لایه‌ی کش: مرورگر، CDN، proxy، اپ.
- `Cache-Control` حاکم است.
- `no-cache` تأیید می‌خواهد، `no-store` ذخیره نمی‌کند.
- `Vary` با gzip الزامی.
- micro-cache بار اپ را به‌شدت کم می‌کند.
- داده‌ی کاربر را کش نکن.

**سؤال مروری:** چرا `Cache-Control: public, max-age=3600` برای `/api/me` یک آسیب‌پذیری است؟

---

## فصل ۵۱: `proxy_cache` و `fastcgi_cache` در Nginx

### مقدمه

`proxy_cache` پاسخ upstream را ذخیره می‌کند؛ `fastcgi_cache` خروجی PHP-FPM. هر دو یک مکانیزم دارند، با تفاوت در `*_cache_*` prefix.

### مفاهیم اساسی

**۱) `proxy_cache_path`**

```nginx
proxy_cache_path /var/cache/nginx/app
                 levels=1:2
                 keys_zone=app_cache:20m
                 max_size=2g
                 inactive=60m
                 use_temp_path=off;
```

| پارامتر | معنا |
|---|---|
| `levels=1:2` | ساختار پوشه (۱۶×۲۵۶) |
| `keys_zone` | حافظه‌ی متادیتا |
| `max_size` | سقف دیسک |
| `inactive` | حذف بعد از بی‌استفاده |
| `use_temp_path=off` | نوشتن مستقیم (سریع‌تر) |

**۲) `proxy_cache_key`**

```nginx
proxy_cache_key "$scheme$request_method$host$request_uri";
```

اگر کاربران مختلف پاسخ متفاوت می‌گیرند، کلید باید شامل آن تفاوت باشد (مثلاً `$cookie_user`).

**۳) `proxy_cache_valid`**

```nginx
proxy_cache_valid 200 302 10m;
proxy_cache_valid 404 1m;
proxy_cache_valid any 1m;
```

**۴) `proxy_cache_bypass` و `proxy_no_cache`**

```nginx
proxy_cache_bypass $cookie_nocache $arg_nocache;
proxy_no_cache     $cookie_nocache $arg_nocache;
```

- `bypass`: از کش نخوان.
- `no_cache`: در کش ننویس.

**۵) `proxy_cache_use_stale`**

```nginx
proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
```

اگر upstream خراب، نسخه‌ی قدیمی سرو شود.

**۶) `proxy_cache_lock`**

```nginx
proxy_cache_lock on;
proxy_cache_lock_timeout 5s;
```

فقط یک درخواست به upstream می‌رود، بقیه منتظر می‌مانند. از «thundering herd» جلوگیری می‌کند.

**۷) `proxy_cache_background_update`**

```nginx
proxy_cache_background_update on;
```

بازسازی در پس‌زمینه، پاسخ قدیمی فوری.

**۸) `fastcgi_cache`**

```nginx
fastcgi_cache_path /var/cache/nginx/php levels=1:2 keys_zone=php_cache:20m max_size=1g inactive=30m;

location ~ \.php$ {
    fastcgi_cache php_cache;
    fastcgi_cache_valid 200 10m;
    fastcgi_cache_key "$scheme$request_method$host$request_uri";
    fastcgi_cache_use_stale error timeout updating;
    fastcgi_cache_background_update on;
    add_header X-FastCGI-Cache $upstream_cache_status always;

    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/run/php/php8.3-fpm.sock;
}
```

**۹) `proxy_cache_revalidate`**

```nginx
proxy_cache_revalidate on;
```

با `If-None-Match`/`If-Modified-Since` به upstream، به‌جای دانلود کامل.

**۱۰) `proxy_cache_min_uses`**

```nginx
proxy_cache_min_uses 2;
```

فقط بعد از ۲ درخواست کش کن (جلوگیری از کش یک‌بارمصرف).

### مثال‌های عملی

**Lab Scenario: کش API با کلید امن**

```nginx
proxy_cache_path /var/cache/nginx/api levels=1:2 keys_zone=api_cache:32m
                 max_size=1g inactive=30m use_temp_path=off;

map $http_authorization $api_no_cache {
    default 1;      # با Authorization → کش نکن
    ""      0;
}

server {
    listen 8080;
    server_name api.lab.example.com;

    location /v1/public/ {
        proxy_cache api_cache;
        proxy_cache_key "$scheme$request_method$host$request_uri";
        proxy_cache_valid 200 60s;
        proxy_cache_valid 404 10s;
        proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
        proxy_cache_lock on;
        proxy_cache_lock_timeout 5s;
        proxy_cache_background_update on;
        proxy_cache_revalidate on;
        proxy_cache_min_uses 2;

        add_header X-Cache-Status $upstream_cache_status always;
        add_header Age $upstream_http_age always;

        proxy_pass http://127.0.0.1:8000/public/;
        proxy_set_header Host $host;
    }

    location /v1/private/ {
        proxy_cache_bypass $api_no_cache;
        proxy_no_cache     $api_no_cache;
        proxy_cache_valid 200 5s;      # micro-cache برای کاهش بار

        add_header X-Cache-Status $upstream_cache_status always;
        proxy_pass http://127.0.0.1:8000/private/;
        proxy_set_header Host $host;
        proxy_set_header Authorization $http_authorization;
    }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx

# public: کش
for i in 1 2 3; do
  curl -sSI http://localhost:8080/v1/public/stats | grep -i x-cache-status
done
# MISS, HIT, HIT

# private با auth: micro-cache
for i in 1 2 3; do
  curl -sSI -H 'Authorization: Bearer token' http://localhost:8080/v1/private/me | grep -i x-cache-status
done
# MISS, HIT, HIT
```

> **توجه:** micro-cache برای داده‌ی کاربر با `Authorization` خطرناک است اگر کلید شامل توکن نباشد. کلید را با `$http_authorization` ترکیب کن:

```nginx
proxy_cache_key "$scheme$request_method$host$request_uri$http_authorization";
```

**پاک کردن کش:**

```bash
# کل کش
sudo rm -rf /var/cache/nginx/api/*

# با ماژول cache_purge (ثالث)
# PURGE /v1/public/stats
```

**`fastcgi_cache` برای وردپرس:**

```nginx
fastcgi_cache_path /var/cache/nginx/wp levels=1:2 keys_zone=wp:100m
                   max_size=2g inactive=60m use_temp_path=off;

map $http_cookie $wp_skip_cache {
    default 0;
    ~*wordpress_logged_in 1;
    ~*comment_author      1;
    ~*woocommerce_cart    1;
}

server {
    location ~ \.php$ {
        fastcgi_cache wp;
        fastcgi_cache_key "$scheme$request_method$host$request_uri";
        fastcgi_cache_valid 200 60m;
        fastcgi_cache_valid 404 5m;
        fastcgi_cache_bypass $wp_skip_cache;
        fastcgi_no_cache     $wp_skip_cache;
        fastcgi_cache_use_stale error timeout updating;
        fastcgi_cache_background_update on;

        add_header X-FastCGI-Cache $upstream_cache_status always;

        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
    }
}
```

**`stale` در عمل:**

```bash
# کش را پر کن
curl -sS http://localhost:8080/v1/public/stats >/dev/null

# اپ را خاموش کن
kill %1

# هنوز پاسخ از کش
curl -sSI http://localhost:8080/v1/public/stats | grep -iE 'x-cache-status|HTTP/'
# HTTP/1.1 200
# X-Cache-Status: STALE
```

**مانیتورینگ کش:**

```bash
# حجم
du -sh /var/cache/nginx/api

# تعداد فایل
find /var/cache/nginx/api -type f | wc -l

# نسبت HIT با awk روی لاگ
awk '{for(i=1;i<=NF;i++) if($i ~ /^x-cache/) print}' access.log
```

### مقایسه و الگوهای مختلف

**`proxy_cache` در برابر `fastcgi_cache` در برابر `uwsgi_cache`**

| | proxy | fastcgi | uwsgi |
|---|---|---|---|
| upstream | HTTP | PHP-FPM | uWSGI |
| prefix | `proxy_` | `fastcgi_` | `uwsgi_` |
| مناسب | API، Node | PHP | Python (legacy) |

**`proxy_cache_lock on` در برابر `off`**

| | `on` | `off` |
|---|---|---|
| thundering herd | ✅ جلوگیری | ❌ |
| تأخیر اولین | ⚠️ | ✅ |
| مناسب | ✅ پرترافیک | کم‌ترافیک |

**`bypass` در برابر `no_cache`**

| | `bypass` | `no_cache` |
|---|---|---|
| خواندن از کش | ❌ | ✅ |
| نوشتن در کش | ✅ | ❌ |
| کاربرد | refresh اجباری | داده‌ی خصوصی |

**دیدگاه «کش را دستی پاک می‌کنیم»**: برای محتوای وابسته، `bypass` با پارامتر یا `purge` بهتر است.

### اشکال شایع

**۱) `proxy_cache_key` بدون کوکی**
کاربر A پاسخ B را می‌بیند. کلید را کامل کن.

**۲) `proxy_cache` روی `Set-Cookie`**
کش پاسخ شخصی. `proxy_ignore_headers Set-Cookie` ممنوع.

**۳) `inactive` کوتاه**
کش مدام خالی. تنظیم.

**۴) `max_size` کم**
کش سریع پر و LRU. افزایش.

**۵) `proxy_cache_valid any`**
خطاها هم کش می‌شوند. `200` فقط.

**۶) `X-Cache-Status` نبودن**
نمی‌دانی کش کار می‌کند. هدر بگذار.

**۷) `fastcgi_cache` برای صفحه‌ی لاگین**
session قاطی. `map` روی کوکی.

**۸) `stale` بدون `updating`**
`UPDATING` به کلاینت می‌رسد. `proxy_cache_use_stale updating`.

**۹) پاک کردن کش با `rm` در حین سرویس**
Nginx فایل‌های باز را نگه می‌دارد. `reload` بعد از پاک.

**۱۰) کش روی `POST`**
پیش‌فرض `GET`/`HEAD`. `proxy_cache_methods`.

### نکات مهم (Tips & Tricks)

- **`X-Cache-Status`** همیشه.
- **`proxy_cache_key`** با همه‌ی متغیرهای مؤثر.
- **`proxy_cache_lock`** برای پرترافیک.
- **`stale-while-revalidate`** برای UX.
- **`map` روی کوکی** برای PHP.
- **`min_uses 2`** برای جلوگیری از کش یک‌بارمصرف.
- **`use_temp_path=off`**.

### خلاصه فصل

- `proxy_cache`/`fastcgi_cache` پاسخ upstream را کش می‌کنند.
- کلید کش باید همه‌ی تفاوت‌ها را شامل شود.
- `lock` از thundering herd جلوگیری می‌کند.
- `stale` در خطا، UX را نجات می‌دهد.
- `bypass`/`no_cache` برای داده‌ی خصوصی.
- `X-Cache-Status` برای دیباگ.

**سؤال مروری:** چرا `proxy_cache_key` بدون `$http_authorization` می‌تواند داده‌ی کاربر A را به کاربر B بدهد؟

---

## فصل ۵۲: فشرده‌سازی — `gzip`، Brotli و `zstd`

### مقدمه

HTML، CSS و JS، ۷۰ تا ۹۰٪ قابل فشرده‌سازی هستند. یعنی ۱ مگابایت به ۱۵۰ کیلوبایت. این تفاوت، روی موبایل یعنی ثانیه‌ها. ولی فشرده‌سازی اشتباه، CPU را می‌خورد یا فایل را خراب می‌کند.

### مفاهیم اساسی

**۱) سه الگوریتم**

| الگوریتم | نسبت | سرعت | پشتیبانی |
|---|---|---|---|
| `gzip` | ۳–۵× | ✅ | ✅✅ |
| Brotli | ۴–۶× | ⚠️ (سطح بالا) | ✅ مدرن |
| `zstd` | ۴–۶× | ✅✅ | ⚠️ جدید |

**۲) `gzip` در Nginx**

```nginx
gzip on;
gzip_vary on;
gzip_comp_level 6;
gzip_min_length 1024;
gzip_proxied any;
gzip_types
    text/plain
    text/css
    text/xml
    text/javascript
    application/javascript
    application/json
    application/xml
    application/rss+xml
    image/svg+xml;
```

| پارامتر | معنا |
|---|---|
| `gzip_comp_level` | ۱–۹ (۶ تعادل) |
| `gzip_min_length` | حداقل حجم |
| `gzip_vary` | `Vary: Accept-Encoding` |
| `gzip_proxied` | فشرده‌سازی برای پروکسی |
| `gzip_static` | فایل `.gz` از قبل |

**۳) `gzip_static`**

```nginx
gzip_static on;
```

فایل `.gz` را مستقیم می‌فرستد (بدون CPU).

```bash
# ساخت فایل‌های .gz در build
find /var/www/lab -type f \( -name '*.css' -o -name '*.js' -o -name '*.svg' \) \
  -exec gzip -9 -k -f {} \;
```

**۴) Brotli**

```nginx
# ماژول: ngx_brotli (کامپایل یا بسته)
brotli on;
brotli_comp_level 5;
brotli_static on;
brotli_types text/plain text/css application/javascript application/json image/svg+xml;
```

Brotli برای فایل استاتیک بهتر از gzip است (۵–۲۰٪ کوچک‌تر).

**۵) `zstd`**

```nginx
# ماژول: ngx_http_zstd_module (1.25.3+ / ثالث)
zstd on;
zstd_comp_level 3;
zstd_static on;
zstd_types application/javascript text/css application/json;
```

**۶) ترتیب انتخاب**

```
کلاینت: Accept-Encoding: br, gzip, zstd
سرور:   Content-Encoding: br   (اگر پشتیبانی)
```

Nginx اولویت را خودش تعیین می‌کند (br > zstd > gzip در ماژول‌های مربوط).

**۷) چه چیزی فشرده نکن**

| نوع | چرا |
|---|---|
| JPEG/PNG/WebP | از قبل فشرده |
| MP4/MP3 | از قبل فشرده |
| ZIP/GZ | از قبل فشرده |
| WOFF2 | از قبل فشرده |
| فایل کوچک (<۱KB) | سربار بیشتر از سود |

**۸) `gzip` در Apache**

```apache
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html text/plain text/css text/xml
    AddOutputFilterByType DEFLATE application/javascript application/json application/xml
    AddOutputFilterByType DEFLATE image/svg+xml
    DeflateCompressionLevel 6
</IfModule>

<IfModule mod_brotli.c>
    AddOutputFilterByType BROTLI_COMPRESS text/html text/css application/javascript application/json
    BrotliCompressionQuality 5
</IfModule>
```

**۹) `gzip` در IIS**

```xml
<httpCompression directory="%SystemDrive%\inetpub\temp\IIS Temporary Compressed Files">
  <scheme name="gzip" dll="%Windir%\system32\inetsrv\gzip.dll" staticCompressionLevel="9" />
  <dynamicTypes>
    <add mimeType="text/*" enabled="true" />
    <add mimeType="application/json" enabled="true" />
    <add mimeType="application/javascript" enabled="true" />
    <add mimeType="*/*" enabled="false" />
  </dynamicTypes>
  <staticTypes>
    <add mimeType="text/*" enabled="true" />
    <add mimeType="application/javascript" enabled="true" />
    <add mimeType="image/svg+xml" enabled="true" />
    <add mimeType="*/*" enabled="false" />
  </staticTypes>
</httpCompression>
<urlCompression doStaticCompression="true" doDynamicCompression="true" />
```

**۱۰) اندازه‌گیری**

```bash
curl -sSI -H 'Accept-Encoding: gzip' https://lab.example.com/app.css | grep -Ei 'content-encoding|content-length'
```

### مثال‌های عملی

**Lab Scenario: gzip + brotli + static**

```bash
# ۱) ساخت فایل‌های از پیش فشرده
cd /var/www/lab
for f in $(find . -type f \( -name '*.css' -o -name '*.js' -o -name '*.svg' -o -name '*.json' \)); do
  gzip -9 -k -f "$f"
  [ -x "$(command -v brotli)" ] && brotli -q 11 -k -f "$f"
done
ls -la assets/
```

```nginx
http {
    gzip on;
    gzip_vary on;
    gzip_comp_level 6;
    gzip_min_length 1024;
    gzip_proxied any;
    gzip_static on;
    gzip_types text/plain text/css text/xml application/javascript application/json application/xml image/svg+xml;

    # اگر ماژول brotli نصب است
    brotli on;
    brotli_comp_level 5;
    brotli_static on;
    brotli_types text/plain text/css application/javascript application/json image/svg+xml;
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx

# gzip
curl -sSI -H 'Accept-Encoding: gzip' http://localhost:8080/assets/app.css \
  | grep -Ei 'content-encoding|content-length|vary'

# brotli
curl -sSI -H 'Accept-Encoding: br' http://localhost:8080/assets/app.css \
  | grep -Ei 'content-encoding|content-length'

# بدون فشرده‌سازی
curl -sSI http://localhost:8080/assets/app.css | grep -Ei 'content-encoding|content-length'
```

**مقایسه‌ی حجم:**

```bash
ORIG=$(stat -c%s /var/www/lab/assets/app.css)
GZ=$(stat -c%s /var/www/lab/assets/app.css.gz)
echo "orig=$ORIG gz=$GZ ratio=$(echo "scale=2; $ORIG/$GZ" | bc)"

# با brotli
[ -f /var/www/lab/assets/app.css.br ] && {
  BR=$(stat -c%s /var/www/lab/assets/app.css.br)
  echo "br=$BR ratio=$(echo "scale=2; $ORIG/$BR" | bc)"
}
```

**`gzip` داینامیک برای API:**

```nginx
location /api/ {
    gzip on;
    gzip_min_length 512;
    gzip_types application/json;

    proxy_pass http://127.0.0.1:8000/;
}
```

> برای پاسخ‌های کوچک JSON، فشرده‌سازی می‌تواند بزرگ‌تر کند. `min_length` را تنظیم کن.

**Apache:**

```bash
sudo a2enmod deflate brotli
sudo apachectl configtest && sudo systemctl reload apache2
```

```apache
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html text/plain text/css text/xml application/json application/javascript
    DeflateCompressionLevel 6
</IfModule>
```

```bash
curl -sSI -H 'Accept-Encoding: gzip' -H 'Host: lab.example.com' http://127.0.0.1/app.css \
  | grep -Ei 'content-encoding|vary'
```

**IIS:**

```powershell
# بررسی
Get-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/httpCompression" -Name "."

# فعال‌سازی
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/urlCompression" -Name "doStaticCompression" -Value $true
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/urlCompression" -Name "doDynamicCompression" -Value $true

# پاک کردن کش IIS
& $env:windir\system32\inetsrv\appcmd.exe set config -section:system.webServer/httpCompression /+"dynamicTypes.[mimeType='application/json',enabled='true']" /commit:apphost
```

### مقایسه و الگوهای مختلف

**`gzip` در برابر Brotli در برابر `zstd`**

| | gzip | Brotli | zstd |
|---|---|---|---|
| نسبت (استاتیک) | ۴× | ۵.۵× | ۵× |
| نسبت (داینامیک) | ۴× | ۴.۵× | ۴.۵× |
| CPU | ✅ | ⚠️ | ✅ |
| پشتیبانی مرورگر | ✅✅ | ✅ | ⚠️ |
| مناسب | ✅ همه | ✅ استاتیک | API |

**`gzip_static` در برابر `gzip`**

| | `static` | `gzip` |
|---|---|---|
| CPU | ✅ صفر | ⚠️ |
| نیاز به `.gz` | ✅ | ❌ |
| مناسب | ✅ استاتیک | داینامیک |

**`comp_level` ۱ در برابر ۶ در برابر ۹**

| | ۱ | ۶ | ۹ |
|---|---|---|---|
| سرعت | ✅✅ | ✅ | ❌ |
| نسبت | ⚠️ | ✅ | ✅ (کم) |
| توصیه | داینامیک پرترافیک | ✅ | استاتیک از پیش |

**دیدگاه «Brotli همیشه بهتر است»**: برای استاتیک بله. برای داینامیک با سطح بالا، CPU زیاد. `comp_level 4–5`.

### اشکال شایع

**۱) `gzip_types` نبودن**
فقط `text/html` فشرده می‌شود. اضافه کن.

**۲) `gzip_vary off`**
کش نسخه‌ی اشتباه. `on`.

**۳) فشرده‌سازی تصویر**
CPU هدر، حجم کم. `gzip_types` تصویر ندارد.

**۴) `gzip_comp_level 9` داینامیک**
CPU بالا. ۶.

**۵) `gzip_static` بدون فایل `.gz`**
بی‌اثر (به `gzip` برمی‌گردد).

**۶) `brotli` بدون ماژول**
`unknown directive`. کامپایل.

**۷) `zstd` در Nginx قدیمی**
پشتیبانی ندارد. ماژول/ارتقا.

**۸) فشرده‌سازی `HEAD`**
بدنه ندارد، ولی هدر ممکن است. طبیعی.

**۹) `Content-Length` اشتباه**
پروکسی که فشرده می‌کند، باید `Content-Length` را حذف/به‌روز کند. Nginx خودش.

**۱۰) `gzip` روی `proxy_pass` بدون `gzip_proxied`**
پاسخ upstream فشرده نمی‌شود. `any`.

### نکات مهم (Tips & Tricks)

- **`gzip_vary on`** الزامی.
- **`gzip_static on`** برای استاتیک.
- **`comp_level 6`** تعادل.
- **Brotli** برای استاتیک.
- **`min_length 1024`**.
- **تصویر را فشرده نکن**.
- **`curl -H 'Accept-Encoding'`** برای تست.

### خلاصه فصل

- gzip/Brotli/zstd، ۷۰–۹۰٪ کاهش برای متن.
- `gzip_vary` برای کش درست.
- `gzip_static` CPU را حذف می‌کند.
- Brotli برای استاتیک بهتر، برای داینامیک سنگین‌تر.
- تصویر و ویدیو را فشرده نکن.
- `min_length` برای جلوگیری از سربار.

**سؤال مروری:** چرا `gzip_vary on` بدون آن می‌تواند باعث شود کاربر فایل خراب ببیند؟

---

## فصل ۵۳: HTTP/2 و HTTP/3 در عمل

### مقدمه

در فصل ۳ مفاهیم را دیدی. این فصل، پیکربندی و اندازه‌گیری است: چطور فعال کنی، چطور تأیید کنی، و چه چیزی در HTTP/2 مضر است.

### مفاهیم اساسی

**۱) Nginx — HTTP/2**

```nginx
server {
    listen 443 ssl;
    http2 on;                 # 1.25.1+
    # listen 443 ssl http2;   # سبک قدیمی
    server_name lab.example.com;
    ssl_certificate     /etc/letsencrypt/live/lab.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/lab.example.com/privkey.pem;
}
```

**۲) Nginx — HTTP/3**

نیاز به ماژول `ngx_http_v3_module` (1.25.0+، build با `--with-http_v3_module`).

```nginx
server {
    listen 443 ssl;
    listen 443 quic reuseport;      # HTTP/3
    http2 on;
    http3 on;

    ssl_certificate     /etc/letsencrypt/live/lab.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/lab.example.com/privkey.pem;

    # TLS 1.3 اجباری برای QUIC
    ssl_protocols TLSv1.2 TLSv1.3;

    add_header Alt-Svc 'h3=":443"; ma=86400' always;

    quic_retry on;
    ssl_early_data on;
}
```

**۳) فایروال UDP**

```bash
sudo ufw allow 443/udp
sudo firewall-cmd --permanent --add-port=443/udp
sudo firewall-cmd --reload
```

**۴) Apache — HTTP/2**

```apache
<VirtualHost *:443>
    Protocols h2 http/1.1
    # یا h2c برای cleartext (نادر)
</VirtualHost>
```

```bash
sudo a2enmod http2
```

**۵) IIS — HTTP/2**

روی HTTPS به‌طور پیش‌فرض فعال. بررسی:

```powershell
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\HTTP\Parameters" `
  -Name "EnableHttp2Tls" -ErrorAction SilentlyContinue
```

**۶) چه چیزی در HTTP/2 مضر است**

| ضدالگو | چرا |
|---|---|
| `domain sharding` | چند اتصال، از دست رفتن multiplexing |
| `concat` فایل‌ها | کش دانه‌درشت |
| `sprite` | کش و نگهداری |
| `inlining` | کش نمی‌شود |
| `Server Push` | حذف‌شده از کروم |

**۷) چه چیزی مفید است**

| کار | چرا |
|---|---|
| فایل‌های کوچک جدا | کش دانه‌ریز |
| `preload` برای critical | شروع سریع |
| `103 Early Hints` | پیش‌بارگذاری |
| یک دامنه | یک اتصال |

**۸) `103 Early Hints`**

```nginx
location / {
    add_header Link "</app.css>; rel=preload; as=style" early;
    add_header Link "</app.js>; rel=preload; as=script" early;
    proxy_pass http://app;
}
```

**۹) `ssl_early_data` (0-RTT)**

```nginx
ssl_early_data on;
```

> فقط برای `GET`/`HEAD` idempotent. برای `POST` خطر replay.

**۱۰) اندازه‌گیری**

```bash
curl -sSI --http2 https://lab.example.com/ | head -1
curl --http3 -sSI https://lab.example.com/ | head -1
```

### مثال‌های عملی

**Lab Scenario: HTTP/2 + HTTP/3 کامل**

```nginx
server {
    listen 80;
    server_name lab.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    listen 443 quic reuseport;
    http2 on;
    http3 on;

    server_name lab.example.com;

    ssl_certificate     /etc/letsencrypt/live/lab.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/lab.example.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_session_tickets off;

    add_header Alt-Svc 'h3=":443"; ma=86400' always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    quic_retry on;
    quic_gso on;

    root /var/www/lab;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx

# HTTP/2
curl -sSI --http2 https://lab.example.com/ | head -1
# HTTP/2 200

# ALPN
openssl s_client -connect lab.example.com:443 -alpn h2,http/1.1 </dev/null 2>/dev/null | grep -i alpn

# Alt-Svc
curl -sSI https://lab.example.com/ | grep -i alt-svc

# HTTP/3 (اگر curl پشتیبانی کند)
curl --http3 -sSI https://lab.example.com/ | head -1
```

**بررسی پشتیبانی `curl` از HTTP/3:**

```bash
curl -V | grep -i http3
# اگر نبود:
# sudo apt install curl  (نسخه‌ی جدید)
# یا build از سورس
```

**مقایسه‌ی تعداد اتصال:**

```bash
curl -sS --http1.1 -o /dev/null -w 'v=%{http_version} conns=%{num_connects}\n' \
  https://lab.example.com/a.css https://lab.example.com/b.css https://lab.example.com/c.css

curl -sS --http2 -o /dev/null -w 'v=%{http_version} conns=%{num_connects}\n' \
  https://lab.example.com/a.css https://lab.example.com/b.css https://lab.example.com/c.css
```

**`103 Early Hints` با اپ:**

```javascript
app.use((req, res, next) => {
  if (req.path === '/') {
    res.writeEarlyHints({
      link: [
        '</app.css>; rel=preload; as=style',
        '</app.js>; rel=preload; as=script',
      ],
    });
  }
  next();
});
```

**`preload` در HTML:**

```html
<link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin>
<link rel="preload" href="/app.css" as="style">
```

**تست `0-RTT`:**

```bash
# اولین اتصال
curl -sSI --tlsv1.3 https://lab.example.com/ >/dev/null

# با session resumption
curl -sSI --tlsv1.3 --tls-max 1.3 https://lab.example.com/ | head -1
```

**IIS بررسی:**

```powershell
curl.exe -sSI --http2 https://lab.example.com/ | Select-Object -First 1
```

### مقایسه و الگوهای مختلف

**HTTP/1.1 در برابر HTTP/2 در برابر HTTP/3**

| | 1.1 | 2 | 3 |
|---|---|---|---|
| اتصال | ۶/دامنه | ۱ | ۱ |
| HOL | ❌ | ✅ (TCP) | ❌ |
| هدر | متن | HPACK | QPACK |
| TLS | اختیاری | عملاً اجباری | اجباری |
| پیچیدگی | ✅ | ✅ | ⚠️ |

**`listen 443 ssl http2` در برابر `http2 on`**

| | قدیمی | جدید |
|---|---|---|
| نسخه | <1.25.1 | 1.25.1+ |
| وضعیت | منسوخ | ✅ |

**`Server Push` در برابر `103 Early Hints`**

| | Push | 103 |
|---|---|---|
| پشتیبانی | ❌ حذف | ✅ |
| کنترل کلاینت | ❌ | ✅ |
| توصیه | ❌ | ✅ |

**دیدگاه «HTTP/3 همیشه سریع‌تر است»**: در شبکه‌ی با loss بالا، بله. در دیتاسنتر، تفاوت ناچیز و پیچیدگی بیشتر (UDP، فایروال).

### اشکال شایع

**۱) `http2 on` در نسخه‌ی قدیمی**
`unknown directive`. `listen 443 ssl http2;`.

**۲) HTTP/2 بدون TLS**
مرورگر پشتیبانی نمی‌کند. HTTPS لازم.

**۳) HTTP/3 بدون UDP باز**
fallback بی‌صدا به HTTP/2. تست کن.

**۴) `Alt-Svc` با پورت اشتباه**
مرورگر به پورت اشتباه می‌رود.

**۵) `domain sharding` باقی‌مانده**
multiplexing از دست می‌رود.

**۶) `concat` فایل‌ها**
کش دانه‌درشت.

**۷) `0-RTT` برای `POST`**
replay. فقط `GET`.

**۸) `quic_retry off`**
DDoS amplification. `on`.

**۹) `ssl_early_data` بدون محدودیت**
همان ریسک replay.

**۱۰) فرض پشتیبانی همه‌ی کلاینت‌ها**
HTTP/1.1 fallback همیشه.

### نکات مهم (Tips & Tricks)

- **`http2 on;`** در 1.25.1+.
- **`listen 443 quic`** + UDP باز.
- **`Alt-Svc`** برای کشف HTTP/3.
- **`103 Early Hints`** به‌جای Push.
- **یک دامنه**، فایل‌های کوچک.
- **`quic_retry on`**.
- **`curl --http3`** برای تست.

### خلاصه فصل

- HTTP/2 با `http2 on`، HTTP/3 با `quic`.
- UDP باید باز باشد.
- `Alt-Svc` پل کشف است.
- `Server Push` مرده؛ `103` جایگزین.
- `domain sharding` و `concat` مضرند.
- `0-RTT` فقط برای idempotent.

**سؤال مروری:** چرا `concat` فایل‌ها در HTTP/2 مضر است؟

---

## فصل ۵۴: تنظیم — `keepalive`، بافر، worker و `fd`

### مقدمه

کارایی وب‌سرور، مجموعه‌ای از تنظیمات کوچک است که با هم اثر می‌گذارند. این فصل، پارامترهای کلیدی هر سه سرور، با روش اندازه‌گیری.

### مفاهیم اساسی

**۱) Nginx — `worker`**

```nginx
worker_processes auto;
worker_rlimit_nofile 65535;

events {
    worker_connections 16384;
    multi_accept on;
    use epoll;
}
```

سقف اتصال ≈ `worker_processes × worker_connections` (و هر اتصال ممکن است ۲ fd بگیرد).

**۲) `keepalive_timeout`**

```nginx
keepalive_timeout 65;
keepalive_requests 1000;
```

`65s` تعادل خوب. `0` برای API بدون keepalive (نادر).

**۳) بافرها**

```nginx
client_body_buffer_size 16k;
client_header_buffer_size 1k;
large_client_header_buffers 4 8k;
output_buffers 2 32k;
postpone_output 1460;
```

| بافر | اثر |
|---|---|
| `client_body_buffer_size` | بدنه‌ی کوچک در حافظه |
| `large_client_header_buffers` | هدر بزرگ (کوکی) |
| `output_buffers` | پاسخ |

**۴) `proxy_buffers`**

```nginx
proxy_buffers 8 16k;
proxy_buffer_size 32k;
proxy_busy_buffers_size 64k;
```

**۵) `sendfile`/`aio`**

```nginx
sendfile on;
sendfile_max_chunk 1m;
aio threads;
directio 4m;
```

**۶) `open_file_cache`**

```nginx
open_file_cache max=10000 inactive=60s;
open_file_cache_valid 30s;
open_file_cache_min_uses 2;
open_file_cache_errors on;
```

**۷) `gzip` و کش**

(فصل ۵۲ و ۵۱)

**۸) Apache — MPM و بافر**

```apache
<IfModule mpm_event_module>
    StartServers 4
    MinSpareThreads 75
    MaxSpareThreads 250
    ThreadsPerChild 32
    MaxRequestWorkers 800
    MaxConnectionsPerChild 10000
</IfModule>

Timeout 30
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5
```

**۹) IIS — تنظیمات**

```powershell
# محدودیت اتصال
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.applicationHost/sites/siteDefaults/limits" `
  -Name "maxConnections" -Value 4294967295

# keep-alive
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/httpProtocol" `
  -Name "allowKeepAlive" -Value $true

# compression
# (فصل ۵۲)
```

**۱۰) سیستم‌عامل**

```bash
# /etc/sysctl.d/99-web.conf
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 10240 65000
net.ipv4.tcp_fin_timeout = 15
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216
fs.file-max = 2000000
```

```bash
sudo sysctl --system
```

### مثال‌های عملی

**Lab Scenario: تنظیم و اندازه‌گیری قبل/بعد**

```bash
# ۱) baseline
hey -z 30s -c 200 http://localhost:8080/ > /tmp/before.txt
grep -E 'Requests/sec|99%' /tmp/before.txt
```

```nginx
# ۲) تنظیم
worker_processes auto;
worker_rlimit_nofile 65535;

events {
    worker_connections 16384;
    multi_accept on;
}

http {
    sendfile on;
    sendfile_max_chunk 1m;
    tcp_nopush on;
    tcp_nodelay on;

    keepalive_timeout 65;
    keepalive_requests 1000;

    client_body_buffer_size 16k;
    large_client_header_buffers 4 8k;
    output_buffers 2 32k;

    open_file_cache max=10000 inactive=60s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;

    gzip on;
    gzip_vary on;
    gzip_comp_level 6;
    gzip_min_length 1024;
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx

# ۳) بعد
hey -z 30s -c 200 http://localhost:8080/ > /tmp/after.txt
grep -E 'Requests/sec|99%' /tmp/after.txt

# ۴) مقایسه
paste <(grep 'Requests/sec' /tmp/before.txt) <(grep 'Requests/sec' /tmp/after.txt)
```

**تست `worker_connections`:**

```bash
# قبل
curl -s http://127.0.0.1:8081/status | head -1

# با ۱۰۰۰ اتصال همزمان
hey -z 20s -c 1000 http://localhost:8080/ &
sleep 5
curl -s http://127.0.0.1:8081/status | head -4
# Active connections: 1001
# accepts/handled: اگر handled < accepts → کمبود

wait
```

**تست `open_file_cache`:**

```bash
# بدون کش
ab -k -n 20000 -c 100 http://localhost:8080/assets/app.css | grep 'Requests per second'

# با کش
# (بعد از تنظیم)
ab -k -n 20000 -c 100 http://localhost:8080/assets/app.css | grep 'Requests per second'
```

**`fd` و `ulimit`:**

```bash
# بررسی
ulimit -n
cat /proc/$(pgrep -f 'nginx: worker' | head -1)/limits | grep 'open files'

# افزایش
sudo tee /etc/security/limits.d/99-web.conf >/dev/null <<'EOF'
www-data soft nofile 65535
www-data hard nofile 65535
nginx    soft nofile 65535
nginx    hard nofile 65535
EOF
```

**`sendfile_max_chunk` برای فایل بزرگ:**

```bash
# فایل ۵۰۰ مگابایتی
fallocate -l 500M /var/www/lab/big.bin

# بدون محدودیت: یک worker اشغال
curl -sS -o /dev/null http://localhost:8080/big.bin &

# با sendfile_max_chunk 1m: منصفانه
```

**IIS — `maxConnections` و `keepAlive`:**

```powershell
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.applicationHost/sites/siteDefaults/limits" `
  -Name "connectionTimeout" -Value "00:02:00"

Get-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/httpProtocol" -Name "allowKeepAlive"
```

**اندازه‌گیری p99:**

```bash
hey -z 60s -c 200 http://localhost:8080/ | grep -A10 'Response time histogram' | tail -12
```

### مقایسه و الگوهای مختلف

**`worker_connections` کم در برابر زیاد**

| | ۱۰۲۴ | ۱۶۳۸۴ | ۶۵۵۳۵ |
|---|---|---|---|
| اتصال | ⚠️ | ✅ | ✅✅ |
| حافظه | ✅ | ✅ | ⚠️ |
| توصیه | ❌ | ✅ | سرور بزرگ |

**`keepalive_timeout` کوتاه در برابر بلند**

| | ۵s | ۶۵s | ۳۰۰s |
|---|---|---|---|
| fd | ✅ | ✅ | ⚠️ |
| تأخیر | ⚠️ | ✅ | ✅ |
| مناسب | API پرترافیک | ✅ عمومی | WebSocket |

**`sendfile` در برابر `aio`**

| | `sendfile` | `aio threads` |
|---|---|---|
| فایل کوچک | ✅ | ⚠️ |
| فایل بزرگ | ⚠️ | ✅ |
| پیچیدگی | ✅ | ⚠️ |

**دیدگاه «پارامتر جادویی»**: نه. `worker_connections`، `keepalive` و `sendfile` بیشترین اثر را دارند. بقیه، تنظیم دقیق.

### اشکال شایع

**۱) `worker_connections` کم**
`worker_connections are not enough`. افزایش.

**۲) `worker_rlimit_nofile` کم**
`Too many open files`.

**۳) `keepalive_timeout 0`**
اتصال جدید هر درخواست.

**۴) `client_header_buffer_size` کم**
`400 Request Header Or Cookie Too Large`. `large_client_header_buffers`.

**۵) `sendfile_max_chunk` نامحدود**
یک فایل بزرگ، worker را اشغال می‌کند.

**۶) `open_file_cache` بدون `valid`**
متادیتای قدیمی.

**۷) `somaxconn` کم**
backlog پر، drop بی‌صدا.

**۸) `MaxRequestWorkers` بیشتر از `ServerLimit × ThreadsPerChild`**
Apache هشدار و کاهش.

**۹) IIS `maxConnections` کم**
`503`. افزایش.

**۱۰) تنظیم بدون اندازه‌گیری**
حس، نه داده. `hey`/`ab`.

### نکات مهم (Tips & Tricks)

- **`worker_processes auto`**.
- **`worker_rlimit_nofile`** بالا.
- **`keepalive_timeout 65`**.
- **`sendfile_max_chunk 1m`**.
- **`open_file_cache`** برای استاتیک.
- **`somaxconn`** بالا.
- **`hey`/`ab`** برای اندازه‌گیری.

### خلاصه فصل

- `worker_connections × workers` سقف اتصال.
- `keepalive` تأخیر و fd را تعادل می‌دهد.
- `sendfile_max_chunk` از اشغال worker جلوگیری می‌کند.
- `open_file_cache` برای فایل زیاد.
- `somaxconn` و `fd` در سطح OS.
- اندازه‌گیری، نه حدس.

**سؤال مروری:** چرا `worker_connections 16384` با ۴ worker به معنای ۶۵٬۵۳۶ اتصال همزمان نیست؟

---

## منابع و مراجع

**مستندات رسمی**
- [Nginx — ngx_http_proxy_module (caching)](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache)
- [Nginx — ngx_http_fastcgi_module](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html)
- [Nginx — ngx_http_gzip_module](https://nginx.org/en/docs/http/ngx_http_gzip_module.html)
- [Nginx — ngx_http_v3_module](https://nginx.org/en/docs/http/ngx_http_v3_module.html)
- [Apache — mod_deflate](https://httpd.apache.org/docs/2.4/mod/mod_deflate.html)
- [IIS — httpCompression](https://learn.microsoft.com/en-us/iis/configuration/system.webserver/httpcompression/)
- [RFC 9111 — HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html)

**ابزار**
- [hey](https://github.com/rakyll/hey)
- [wrk](https://github.com/wg/wrk)
- [nghttp2 client](https://nghttp2.org/)
- [WebPageTest](https://www.webpagetest.org/)

**آموزش**
- [HTTP Caching — MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching)
- [Brotli vs gzip](https://blog.cloudflare.com/results-experimenting-brotli/)
- [Nginx caching guide](https://docs.nginx.com/nginx/admin-guide/content-cache/content-caching/)

---

**بعدی:** [`13-Security-and-Hardening.md`](13-Security-and-Hardening.md) — هدرها، rate limit، traversal، WAF و چک‌لیست
