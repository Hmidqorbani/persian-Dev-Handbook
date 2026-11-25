# بخش ۰۸: Reverse Proxy

**مدت یادگیری**: ۵ تا ۶ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۰۵ (`location`)، بخش ۰۷ (اپ داینامیک)

---

## فهرست محتوا

- [فصل ۳۱: Reverse Proxy چیست و چرا لازم است](#فصل-۳۱)
- [فصل ۳۲: `proxy_pass` و `upstream` در Nginx](#فصل-۳۲)
- [فصل ۳۳: `mod_proxy` در Apache](#فصل-۳۳)
- [فصل ۳۴: هدرها، `X-Forwarded-*`، `real_ip` و timeout](#فصل-۳۴)
- [فصل ۳۵: WebSocket و SSE از پشت پروکسی](#فصل-۳۵)

---

## فصل ۳۱: Reverse Proxy چیست و چرا لازم است

### مقدمه

در معماری مدرن، تقریباً هیچ اپلیکیشنی مستقیم به اینترنت وصل نمی‌شود. یک **reverse proxy** جلوی آن می‌نشیند و کارهایی را انجام می‌دهد که اپ نباید انجام دهد: TLS، فشرده‌سازی، کش، rate limit، مسیریابی، و مخفی کردن ساختار داخلی.

### مفاهیم اساسی

**۱) Forward در برابر Reverse**

```
Forward Proxy:
کلاینت ──▶ پروکسی ──▶ اینترنت
         (به نمایندگی کلاینت)
         مثال: VPN، پروکسی سازمانی

Reverse Proxy:
اینترنت ──▶ پروکسی ──▶ سرورهای داخلی
          (به نمایندگی سرور)
          مثال: Nginx جلوی اپ
```

**۲) دیاگرام کامل**

```
                          ┌─────────────────────────────┐
   کلاینت                  │  Reverse Proxy (Nginx)      │
     │                     │  :443                       │
     │  HTTPS              │  ├── TLS termination        │
     ├────────────────────▶│  ├── static files           │
     │                     │  ├── rate limit             │
     │                     │  ├── cache                  │
     │                     │  └── routing                │
     │                     └───────┬─────────┬───────────┘
     │                             │         │
     │                     HTTP    │         │  HTTP
     │                             ▼         ▼
     │                     ┌──────────┐  ┌──────────┐
     │                     │ app:8000 │  │ app:8001 │
     │                     └──────────┘  └──────────┘
     │                          │
     │                          ▼
     │                     ┌──────────┐
     │                     │ DB       │
     │                     └──────────┘
```

**۳) ده کاری که proxy انجام می‌دهد**

| کار | چرا در proxy |
|---|---|
| TLS termination | گواهی در یک جا، نه در هر اپ |
| فشرده‌سازی | یک‌بار، برای همه |
| کش | کاهش بار اپ |
| rate limit | جلوگیری از سوءاستفاده |
| مسیریابی | `/api` به اپ، `/` به SPA |
| فایل استاتیک | سریع‌تر از اپ |
| هدرهای امنیتی | یک‌جا |
| health check | حذف upstream خراب |
| load balancing | توزیع بار |
| مخفی کردن ساختار | اپ در شبکه‌ی خصوصی |

**۴) مزیت امنیتی**

```
بدون proxy:
اینترنت ──▶ app:3000 (Node)
            ├── TLS? خودت
            ├── rate limit? خودت
            ├── static? خودت
            └── باگ در app = سطح حمله

با proxy:
اینترنت ──▶ proxy:443 ──▶ app:3000 (فقط 127.0.0.1)
            ├── TLS ✅
            ├── rate limit ✅
            └── app در شبکه‌ی خصوصی
```

**۵) `proxy_pass` — قلب کار**

```nginx
location /api/ {
    proxy_pass http://127.0.0.1:8000/;
}
```

Nginx درخواست را می‌گیرد، هدرها را (با تنظیمات) بازنویسی می‌کند، به upstream می‌فرستد و پاسخ را برمی‌گرداند.

**۶) `upstream` — گروه سرورها**

```nginx
upstream backend {
    server 127.0.0.1:8000;
    server 127.0.0.1:8001;
    keepalive 32;
}
```

**۷) `proxy_set_header` — چه چیزی به upstream می‌رود**

به‌طور پیش‌فرض، Nginx هدرهای `Host` و `Connection` را تغییر می‌دهد. برای اپ، باید صریح تنظیم کنی:

```nginx
proxy_set_header Host              $host;
proxy_set_header X-Real-IP         $remote_addr;
proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-Forwarded-Host  $host;
proxy_set_header X-Forwarded-Port  $server_port;
```

**۸) `proxy_http_version`**

پیش‌فرض `1.0` است. برای `keepalive`، WebSocket و `chunked`، باید `1.1` باشد.

**۹) `proxy_buffering`**

| مقدار | رفتار |
|---|---|
| `on` (پیش‌فرض) | پاسخ را در بافر می‌گیرد، بعد به کلاینت می‌دهد |
| `off` | stream مستقیم |

برای SSE، دانلود بزرگ و streaming، `off`.

**۱۰) `proxy_cache`**

```nginx
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=api:10m max_size=1g inactive=60m;

location /api/public/ {
    proxy_cache api;
    proxy_cache_valid 200 5m;
    proxy_cache_use_stale error timeout updating;
    add_header X-Cache-Status $upstream_cache_status;
    proxy_pass http://backend/;
}
```

### مثال‌های عملی

**Lab Scenario: سه سرویس پشت یک proxy**

```nginx
# /etc/nginx/sites-available/gateway.conf
upstream app_api {
    server 127.0.0.1:8000;
    keepalive 32;
}

upstream app_ws {
    server 127.0.0.1:3000;
    keepalive 16;
}

map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 8080;
    server_name lab.example.com;

    # ۱) SPA
    location / {
        root /var/www/spa;
        try_files $uri $uri/ /index.html;
    }

    # ۲) API
    location /api/ {
        proxy_pass http://app_api/;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 5s;
        proxy_read_timeout    60s;
        proxy_send_timeout    60s;
    }

    # ۳) WebSocket
    location /ws {
        proxy_pass http://app_ws;
        proxy_http_version 1.1;
        proxy_set_header Upgrade    $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host       $host;
        proxy_read_timeout 3600s;
    }

    # ۴) فایل استاتیک
    location /assets/ {
        alias /var/www/spa/assets/;
        expires 30d;
        add_header Cache-Control "public, immutable";
        access_log off;
    }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx

curl -sS http://localhost:8080/api/health
curl -sS http://localhost:8080/ | head -3
```

**بررسی هدرهای رسیده به اپ:**

```python
# app.py
from fastapi import FastAPI, Request
app = FastAPI()

@app.get("/debug/headers")
def headers(request: Request):
    return dict(request.headers)
```

```bash
curl -sS http://localhost:8080/api/debug/headers | python -m json.tool
```

باید `host`، `x-real-ip`، `x-forwarded-for`، `x-forwarded-proto` را ببینی.

**کش با `X-Cache-Status`:**

```nginx
proxy_cache_path /var/cache/nginx/api levels=1:2 keys_zone=api_cache:10m
                 max_size=500m inactive=30m use_temp_path=off;

server {
    location /api/public/ {
        proxy_cache api_cache;
        proxy_cache_key "$scheme$request_method$host$request_uri";
        proxy_cache_valid 200 302 5m;
        proxy_cache_valid 404 1m;
        proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
        proxy_cache_background_update on;
        proxy_cache_lock on;
        add_header X-Cache-Status $upstream_cache_status always;
        proxy_pass http://app_api/public/;
    }
}
```

```bash
for i in 1 2 3; do
  curl -sSI http://localhost:8080/api/public/data | grep -i x-cache-status
done
# MISS
# HIT
# HIT
```

### مقایسه و الگوهای مختلف

**Reverse proxy در برابر Load Balancer**

| | Reverse Proxy | Load Balancer |
|---|---|---|
| تمرکز | TLS، کش، routing | توزیع بار |
| لایه | L7 | L4/L7 |
| ابزار | Nginx، Apache، Envoy | Nginx، HAProxy، ALB |
| هم‌پوشانی | ✅ (Nginx هر دو) | ✅ |

**`proxy_buffering on` در برابر `off`**

| | `on` | `off` |
|---|---|---|
| حافظه | ⚠️ | ✅ |
| تأخیر TTFB | ⚠️ | ✅ |
| streaming | ❌ | ✅ |
| توصیه | API معمولی | SSE، دانلود |

**سه مدل استقرار**

1. **Proxy روی همان سرور اپ**: ساده، تک‌سرور.
2. **Proxy جدا**: جداسازی، مقیاس.
3. **Proxy در لبه (CDN/LB)**: مقیاس بزرگ.

**دیدگاه «اپ خودش TLS داشته باشد»**: هر اپ باید گواهی، فشرده‌سازی و rate limit را دوباره پیاده کند. proxy این‌ها را یک‌جا می‌دهد.

### اشکال شایع

**۱) `proxy_set_header Host $host` فراموش‌شده**
اپ `Host` اشتباه می‌بیند → redirect و لینک غلط.

**۲) `proxy_http_version 1.0`**
`keepalive` بی‌اثر، WebSocket شکسته.

**۳) `X-Forwarded-For` بدون `real_ip`**
لاگ IP اشتباه.

**۴) `proxy_buffering on` برای SSE**
پیام‌ها جمع می‌شوند.

**۵) `proxy_pass` با `/` اشتباه**
مسیر دوبار/حذف.

**۶) timeout پیش‌فرض ۶۰s**
اپ کند → `504`.

**۷) کش بدون `proxy_cache_key`**
کاربر A پاسخ کاربر B را می‌بیند (اگر احراز هویت در کوکی باشد). `proxy_cache_bypass $http_authorization` یا `proxy_no_cache`.

**۸) `proxy_cache` روی `/api/` احراز هویت‌شده**
افشای داده. فقط public.

**۹) upstream `127.0.0.1` و اپ روی `0.0.0.0`**
کار می‌کند، ولی اپ در معرض دسترسی مستقیم است. bind به `127.0.0.1`.

**۱۰) `client_max_body_size` کم**
`413` برای آپلود.

### نکات مهم (Tips & Tricks)

- **`proxy_set_header Host $host`** همیشه.
- **`proxy_http_version 1.1`** + `Connection ""` برای keepalive.
- **`X-Forwarded-*`** کامل.
- **`proxy_buffering off`** برای stream.
- **`X-Cache-Status`** برای دیباگ کش.
- **`proxy_connect_timeout 5s`**.
- **bind اپ به `127.0.0.1`**.

### خلاصه فصل

- Reverse proxy جلوی اپ می‌نشیند و TLS/کش/امنیت را یک‌جا می‌دهد.
- `proxy_pass` قلب کار است؛ `/` پایانی مسیر را عوض می‌کند.
- `upstream` + `keepalive` برای کارایی.
- `X-Forwarded-*` برای IP و پروتکل.
- `proxy_buffering` برای stream باید `off`.
- کش فقط برای محتوای عمومی.

**سؤال مروری:** چرا گذاشتن اپ روی `0.0.0.0:3000` و proxy روی همان سرور، از `127.0.0.1:3000` خطرناک‌تر است؟

---

## فصل ۳۲: `proxy_pass` و `upstream` در Nginx

### مقدمه

`proxy_pass` ساده به نظر می‌رسد: یک URL بده. ولی رفتارش با `/` پایانی، `location`، `rewrite` و `upstream` ترکیب می‌شود و نتیجه می‌تواند غیرشهودی باشد.

### مفاهیم اساسی

**۱) قاعده‌ی `/` پایانی**

```nginx
location /api/ {
    proxy_pass http://backend;     # مسیر کامل حفظ می‌شود
}
# /api/users → http://backend/api/users

location /api/ {
    proxy_pass http://backend/;    # پیشوند location حذف می‌شود
}
# /api/users → http://backend/users
```

**۲) `proxy_pass` با URI در `location` regex**

```nginx
location ~ ^/api/(.*)$ {
    proxy_pass http://backend/$1;   # صریح
}
```

> در `location` regex، `proxy_pass` **باید** URI داشته باشد، وگرنه خطا.

**۳) `upstream` — تنظیمات**

```nginx
upstream backend {
    least_conn;
    server 10.0.0.11:8000 weight=3 max_fails=3 fail_timeout=30s;
    server 10.0.0.12:8000;
    server 10.0.0.13:8000 backup;
    keepalive 32;
    keepalive_requests 1000;
    keepalive_timeout 60s;
}
```

| پارامتر | معنا |
|---|---|
| `weight` | وزن |
| `max_fails` | تعداد شکست قبل از غیرفعال |
| `fail_timeout` | مدت غیرفعالی |
| `backup` | فقط اگر بقیه خراب |
| `down` | دستی غیرفعال |
| `keepalive` | اتصال idle به upstream |

**۴) `least_conn`، `ip_hash`، `hash`**

```nginx
upstream backend {
    least_conn;                    # کمترین اتصال فعال
}
upstream backend {
    ip_hash;                       # sticky بر اساس IP
}
upstream backend {
    hash $request_uri consistent;  # بر اساس URI (کش)
}
```

**۵) `proxy_next_upstream`**

```nginx
proxy_next_upstream error timeout http_502 http_503 http_504;
proxy_next_upstream_tries 2;
proxy_next_upstream_timeout 10s;
```

اگر upstream اول خطا داد، به بعدی برو.

**۶) `proxy_intercept_errors`**

```nginx
proxy_intercept_errors on;
error_page 502 504 = @fallback;
```

بدون `on`، خطای upstream مستقیم به کلاینت می‌رود.

**۷) `proxy_redirect`**

اگر اپ `Location: http://127.0.0.1:8000/login` بدهد، کلاینت به آدرس داخلی می‌رود:

```nginx
proxy_redirect http://127.0.0.1:8000/ /;
# یا
proxy_redirect default;
```

**۸) `proxy_cookie_path` و `proxy_cookie_domain`**

```nginx
proxy_cookie_path /app/ /;
proxy_cookie_domain internal.example.com example.com;
```

**۹) `proxy_ssl_*` برای upstream HTTPS**

```nginx
proxy_pass https://backend;
proxy_ssl_verify on;
proxy_ssl_trusted_certificate /etc/ssl/certs/ca.pem;
proxy_ssl_server_name on;
```

**۱۰) `resolver` برای upstream داینامیک**

```nginx
resolver 127.0.0.11 valid=10s ipv6=off;
set $backend "http://app:8000";
proxy_pass $backend;
```

برای Docker/K8s که IP عوض می‌شود.

### مثال‌های عملی

**Lab Scenario: مقایسه‌ی چهار حالت `proxy_pass`**

```nginx
server {
    listen 8080;
    server_name _;

    # ۱) بدون / — مسیر کامل
    location /a/ {
        proxy_pass http://127.0.0.1:8000;
    }

    # ۲) با / — حذف پیشوند
    location /b/ {
        proxy_pass http://127.0.0.1:8000/;
    }

    # ۳) با URI کامل
    location /c/ {
        proxy_pass http://127.0.0.1:8000/v1/;
    }

    # ۴) regex
    location ~ ^/d/(.*)$ {
        proxy_pass http://127.0.0.1:8000/x/$1;
    }
}
```

```bash
# اپ باید مسیر دریافتی را برگرداند
# app: @app.get("/{path:path}") → {"path": path}

for p in /a/users /b/users /c/users /d/users; do
  printf '%-12s → ' "$p"
  curl -sS "http://localhost:8080$p"
  echo
done
```

خروجی نمونه:

```
/a/users     → {"path": "a/users"}
/b/users     → {"path": "users"}
/c/users     → {"path": "v1/users"}
/d/users     → {"path": "x/users"}
```

**`upstream` با weight و backup:**

```nginx
upstream api {
    least_conn;
    server 127.0.0.1:8000 weight=3 max_fails=2 fail_timeout=10s;
    server 127.0.0.1:8001 weight=1;
    server 127.0.0.1:8002 backup;
    keepalive 32;
}

server {
    listen 8080;
    location / {
        proxy_pass http://api;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
    }
}
```

```bash
# توزیع
for i in $(seq 1 20); do
  curl -sS http://localhost:8080/instance; echo
done | sort | uniq -c
# 15 → :8000
#  5 → :8001
```

**`proxy_next_upstream`:**

```nginx
upstream flaky {
    server 127.0.0.1:8001;    # خاموش
    server 127.0.0.1:8000;    # سالم
}

location /failover/ {
    proxy_pass http://flaky/;
    proxy_next_upstream error timeout http_502 http_503;
    proxy_next_upstream_tries 2;
}
```

```bash
curl -sS http://localhost:8080/failover/health
# پاسخ از 8000، بدون 502
```

**`proxy_redirect`:**

```nginx
location /app/ {
    proxy_pass http://127.0.0.1:8000/;
    proxy_redirect http://127.0.0.1:8000/ /app/;
}
```

بدون آن:

```bash
curl -sSI http://localhost:8080/app/login | grep -i location
# Location: http://127.0.0.1:8000/login   ← اشتباه
```

با آن:

```bash
# Location: /app/login
```

**upstream داینامیک (Docker):**

```nginx
resolver 127.0.0.11 valid=10s ipv6=off;

server {
    listen 8080;
    set $upstream_app "http://app:8000";
    location / {
        proxy_pass $upstream_app;
        proxy_set_header Host $host;
    }
}
```

> با `$variable`، Nginx در هر درخواست DNS را resolve می‌کند.

**`proxy_ssl` برای upstream HTTPS:**

```nginx
location /secure/ {
    proxy_pass https://internal.example.com/;
    proxy_ssl_server_name on;
    proxy_ssl_verify on;
    proxy_ssl_trusted_certificate /etc/ssl/certs/internal-ca.pem;
}
```

### مقایسه و الگوهای مختلف

**چهار حالت `proxy_pass`**

| `location` | `proxy_pass` | `/api/users` → |
|---|---|---|
| `/api/` | `http://b` | `/api/users` |
| `/api/` | `http://b/` | `/users` |
| `/api/` | `http://b/v1/` | `/v1/users` |
| `~ ^/api/(.*)$` | `http://b/$1` | `/users` |

**`least_conn` در برابر `ip_hash` در برابر `hash`**

| | توزیع | sticky | مناسب |
|---|---|---|---|
| پیش‌فرض (round-robin) | ✅ | ❌ | API بی‌حالت |
| `least_conn` | ✅✅ | ❌ | اپ با زمان متغیر |
| `ip_hash` | ⚠️ | ✅ | session در حافظه |
| `hash $uri` | ✅ | ✅ (per URI) | کش |

**`proxy_buffering`**

| | `on` | `off` |
|---|---|---|
| TTFB | ⚠️ | ✅ |
| حافظه | ⚠️ | ✅ |
| streaming | ❌ | ✅ |

**دیدگاه «`ip_hash` برای sticky کافی است»**: با NAT و موبایل، IP عوض می‌شود و session می‌پرد. بهتر: session در Redis.

### اشکال شایع

**۱) `proxy_pass` با `/` ناخواسته**
مسیر حذف می‌شود. `nginx -T` و `curl -v`.

**۲) `proxy_pass` در `location` regex بدون URI**
`proxy_pass cannot have URI part in location given by regular expression`. URI بده.

**۳) `upstream` با `keepalive` و `proxy_http_version 1.0`**
keepalive بی‌اثر. `1.1`.

**۴) `proxy_next_upstream` روی `POST`**
درخواست تکراری می‌شود. `proxy_next_upstream` را برای `POST` محدود کن یا `non_idempotent` را نگذار.

**۵) `proxy_redirect` فراموش‌شده**
redirect به IP داخلی.

**۶) `resolver` نبودن با upstream داینامیک**
`no resolver defined`. `resolver` بگذار.

**۷) `weight` و `least_conn` با هم**
`least_conn` وزن را نادیده نمی‌گیرد، ولی رفتار پیچیده می‌شود.

**۸) `max_fails=0`**
upstream هرگز غیرفعال نمی‌شود.

**۹) `proxy_ssl_verify off`**
MITM ممکن. `on` + CA.

**۱۰) `Host` به upstream با پورت**
`$host` بدون پورت است؛ `$http_host` با پورت. بسته به اپ.

### نکات مهم (Tips & Tricks)

- **`/` پایانی** را آگاهانه انتخاب کن.
- **`proxy_http_version 1.1` + `Connection ""`**.
- **`proxy_next_upstream`** برای idempotent.
- **`proxy_redirect`** برای اپ با redirect داخلی.
- **`resolver`** برای Docker/K8s.
- **`X-Cache-Status`** برای کش.
- **`proxy_connect_timeout 5s`**.

### خلاصه فصل

- `/` پایانی در `proxy_pass` مسیر را حذف می‌کند.
- regex نیاز به URI صریح دارد.
- `upstream` وزن، backup، keepalive و health دارد.
- `proxy_next_upstream` برای failover.
- `proxy_redirect` برای اصلاح `Location`.
- `resolver` برای upstream داینامیک.

**سؤال مروری:** تفاوت `proxy_pass http://b;` و `proxy_pass http://b/;` در `location /api/` چیست؟

---

## فصل ۳۳: `mod_proxy` در Apache

### مقدمه

Apache با `mod_proxy` و خانواده‌اش (`mod_proxy_http`، `mod_proxy_fcgi`، `mod_proxy_wstunnel`، `mod_proxy_balancer`) همان کار Nginx را می‌کند، با سینتکس متفاوت و انعطاف بیشتر در `Location`.

### مفاهیم اساسی

**۱) ماژول‌ها**

| ماژول | کار |
|---|---|
| `mod_proxy` | هسته |
| `mod_proxy_http` | HTTP/HTTPS |
| `mod_proxy_fcgi` | PHP-FPM |
| `mod_proxy_wstunnel` | WebSocket |
| `mod_proxy_balancer` | load balancing |
| `mod_proxy_hcheck` | health check |
| `mod_remoteip` | IP واقعی |

```bash
sudo a2enmod proxy proxy_http proxy_wstunnel proxy_balancer lbmethod_byrequests headers remoteip
sudo apachectl configtest && sudo systemctl reload apache2
```

**۲) `ProxyPass` و `ProxyPassReverse`**

```apache
ProxyPass        /api/ http://127.0.0.1:8000/
ProxyPassReverse /api/ http://127.0.0.1:8000/
```

- `ProxyPass`: درخواست را به upstream بفرست.
- `ProxyPassReverse`: هدر `Location` در پاسخ را بازنویسی کن.

**بدون `ProxyPassReverse`:** redirect به IP داخلی می‌رود.

**۳) `/` پایانی**

```apache
ProxyPass /api/ http://127.0.0.1:8000/     # /api/users → /users
ProxyPass /api/ http://127.0.0.1:8000      # /api/users → /api/users
```

**۴) `ProxyPreserveHost`**

```apache
ProxyPreserveHost On
```

`Host` اصلی حفظ می‌شود (به‌جای `127.0.0.1:8000`).

**۵) `<Location>` و `ProxyPass`**

```apache
<Location /api/>
    ProxyPass http://127.0.0.1:8000/
    ProxyPassReverse http://127.0.0.1:8000/
    ProxyPreserveHost On
    RequestHeader set X-Forwarded-Proto "https"
</Location>
```

**۶) `BalancerMember`**

```apache
<Proxy "balancer://mycluster">
    BalancerMember http://127.0.0.1:8000 route=node1
    BalancerMember http://127.0.0.1:8001 route=node2
    ProxySet lbmethod=byrequests
</Proxy>

ProxyPass        / balancer://mycluster/
ProxyPassReverse / balancer://mycluster/
```

| `lbmethod` | الگوریتم |
|---|---|
| `byrequests` | round-robin |
| `bytraffic` | بر اساس ترافیک |
| `bybusyness` | کمترین اشتغال |
| `heartbeat` | با health check |

**۷) `ProxyPass` با timeout**

```apache
ProxyPass /api/ http://127.0.0.1:8000/ connectiontimeout=5 timeout=60 retry=10
```

**۸) `mod_proxy_hcheck`**

```apache
<Proxy "balancer://mycluster">
    BalancerMember http://127.0.0.1:8000 hcmethod=GET hcuri=/health hcinterval=10
    BalancerMember http://127.0.0.1:8001 hcmethod=GET hcuri=/health hcinterval=10
    ProxySet lbmethod=byrequests hcheck=on
</Proxy>
```

**۹) `ProxyPassMatch` برای regex**

```apache
ProxyPassMatch "^/api/v([0-9]+)/(.*)$" "http://127.0.0.1:8000/v$1/$2"
```

**۱۰) `ProxyRequests Off`**

`ProxyRequests On` = forward proxy (خطرناک، open proxy). همیشه `Off`.

### مثال‌های عملی

**Lab Scenario: Apache به‌عنوان gateway**

```apache
# /etc/apache2/sites-available/gateway.conf
<VirtualHost *:8080>
    ServerName lab.example.com
    ProxyRequests Off
    ProxyPreserveHost On

    DocumentRoot /var/www/spa

    <Directory /var/www/spa>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    # ۱) API
    <Location /api/>
        ProxyPass        http://127.0.0.1:8000/
        ProxyPassReverse http://127.0.0.1:8000/
        RequestHeader set X-Forwarded-Proto "https"
        RequestHeader set X-Real-IP "%{REMOTE_ADDR}s"
    </Location>

    # ۲) WebSocket
    <Location /ws>
        ProxyPass        ws://127.0.0.1:3000/ws
        ProxyPassReverse ws://127.0.0.1:3000/ws
    </Location>

    # ۳) SPA fallback
    <Location />
        FallbackResource /index.html
    </Location>

    # ۴) استاتیک
    <Location /assets/>
        ProxyPass !           # هرگز پروکسی نکن (از دیسک)
    </Location>

    ErrorLog ${APACHE_LOG_DIR}/gateway-error.log
    CustomLog ${APACHE_LOG_DIR}/gateway-access.log combined
</VirtualHost>
```

```bash
sudo a2ensite gateway
sudo apachectl configtest && sudo systemctl reload apache2

curl -sS http://localhost:8080/api/health
curl -sSI http://localhost:8080/ | head -3
```

**`ProxyPass !` — استثنا**

```apache
ProxyPass /assets/ !
ProxyPass /api/ http://127.0.0.1:8000/
```

`!` یعنی «این مسیر را پروکسی نکن».

**Balancer با health check:**

```apache
<Proxy "balancer://app">
    BalancerMember http://127.0.0.1:8000 hcmethod=GET hcuri=/health
    BalancerMember http://127.0.0.1:8001 hcmethod=GET hcuri=/health
    ProxySet lbmethod=bybusyness hcheck=on
</Proxy>

<Location />
    ProxyPass        balancer://app/
    ProxyPassReverse balancer://app/
</Location>
```

```bash
# وضعیت balancer
<Location /balancer-manager>
    SetHandler balancer-manager
    Require ip 127.0.0.1
</Location>
```

```bash
curl -sS http://localhost:8080/balancer-manager | head -20
```

**`mod_remoteip`:**

```apache
RemoteIPHeader X-Forwarded-For
RemoteIPTrustedProxy 10.0.0.0/8
RemoteIPTrustedProxy 127.0.0.1
```

```apache
# لاگ با IP واقعی
LogFormat "%a %l %u %t \"%r\" %>s %b %D" realip
CustomLog ${APACHE_LOG_DIR}/access.log realip
```

**`ProxyPass` با timeout:**

```apache
<Location /slow/>
    ProxyPass http://127.0.0.1:8000/ connectiontimeout=3 timeout=30
</Location>
```

### مقایسه و الگوهای مختلف

**`mod_proxy` در برابر Nginx `proxy_pass`**

| | Apache | Nginx |
|---|---|---|
| سینتکس | `ProxyPass` | `proxy_pass` |
| `/` پایانی | ✅ مشابه | ✅ |
| `Host` | `ProxyPreserveHost` | `proxy_set_header` |
| WebSocket | `mod_proxy_wstunnel` | `Upgrade` header |
| Balancer | `BalancerMember` | `upstream` |
| Health check | `mod_proxy_hcheck` | passive/`plus` |
| Config per-location | ✅ | ✅ |

**`ProxyPass` در برابر `<Location>` + `ProxyPass`**

| | `ProxyPass` | `<Location>` |
|---|---|---|
| سادگی | ✅ | ⚠️ |
| هدرها | ⚠️ | ✅ |
| مناسب | ساده | نیاز به `RequestHeader` |

**`byrequests` در برابر `bybusyness`**

| | `byrequests` | `bybusyness` |
|---|---|---|
| توزیع | مساوی | بر اساس بار |
| مناسب | درخواست هم‌زمان یکسان | زمان پاسخ متغیر |

**دیدگاه «Apache برای proxy کند است»**: با `event` MPM، تفاوت با Nginx کم است. کندی معمولاً از `prefork` می‌آید.

### اشکال شایع

**۱) `ProxyPassReverse` فراموش‌شده**
redirect به IP داخلی.

**۲) `ProxyPreserveHost` خاموش**
اپ `Host` اشتباه می‌بیند.

**۳) `ProxyRequests On`**
open proxy. خطرناک. `Off`.

**۴) `mod_proxy_wstunnel` نبودن**
WebSocket `400`. `a2enmod`.

**۵) `ProxyPass` بدون `/`**
مسیر دوبار. (مثل Nginx)

**۶) `timeout` پیش‌فرض**
۳۰۰ ثانیه. برای اپ کند، `504` دیر می‌آید.

**۷) `RequestHeader` بدون `mod_headers`**
`Invalid command`. `a2enmod headers`.

**۸) balancer بدون `ProxyPassReverse`**
`Location` اشتباه.

**۹) `hcheck` بدون `mod_proxy_hcheck`**
`Invalid command`. `a2enmod proxy_hcheck`.

**۱۰) `Location` و `ProxyPass` با ترتیب اشتباه**
`ProxyPass` در `Location` و بیرون، تعارض. یکی را انتخاب کن.

### نکات مهم (Tips & Tricks)

- **`ProxyPassReverse`** همیشه با `ProxyPass`.
- **`ProxyPreserveHost On`**.
- **`ProxyPass !`** برای استثنا.
- **`balancer-manager`** برای دیباگ.
- **`mod_remoteip`** برای IP.
- **`connectiontimeout`** کوتاه.
- **`event` MPM** برای کارایی.

### خلاصه فصل

- `mod_proxy` + `mod_proxy_http` پایه است.
- `ProxyPassReverse` برای `Location`.
- `ProxyPreserveHost` برای `Host`.
- `BalancerMember` برای load balancing.
- `ProxyPass !` برای استثنا.
- `ProxyRequests Off` همیشه.

**سؤال مروری:** چرا `ProxyPass` بدون `ProxyPassReverse` باعث می‌شود کاربر به `127.0.0.1` هدایت شود؟

---

## فصل ۳۴: هدرها، `X-Forwarded-*`، `real_ip` و timeout

### مقدمه

پشت پروکسی، دو چیز گم می‌شود: **IP کلاینت** و **پروتکل اصلی**. اگر این‌ها را درست منتقل نکنی، لاگ بی‌معنا می‌شود، rate limit اشتباه کار می‌کند، و redirect بی‌پایان می‌شود.

### مفاهیم اساسی

**۱) هدرهای استاندارد**

| هدر | مقدار | مصرف‌کننده |
|---|---|---|
| `X-Forwarded-For` | `client, proxy1, proxy2` | لاگ، rate limit، geo |
| `X-Forwarded-Proto` | `http`/`https` | redirect، کوکی `Secure` |
| `X-Forwarded-Host` | دامنه‌ی اصلی | لینک‌سازی |
| `X-Forwarded-Port` | پورت اصلی | لینک‌سازی |
| `X-Real-IP` | IP کلاینت | Nginx، اپ |
| `Forwarded` (RFC 7239) | استاندارد | اپ مدرن |

**۲) `X-Forwarded-For` — زنجیره**

```
کلاینت (203.0.113.45) → CDN (10.0.0.5) → Nginx (10.0.0.6) → اپ
X-Forwarded-For: 203.0.113.45, 10.0.0.5
```

- چپ‌ترین = کلاینت اصلی (اگر همه‌ی پروکسی‌ها درست عمل کنند).
- راست‌ترین = آخرین پروکسی.

**۳) خطر جعل**

کلاینت می‌تواند `X-Forwarded-For` بفرستد. اگر سرور بدون اعتبارسنجی باورش کند، IP جعلی می‌گیرد.

```nginx
# ❌ خطرناک
proxy_set_header X-Real-IP $http_x_forwarded_for;
```

**درست:** فقط به پروکسی‌های خودت اعتماد کن.

**۴) `real_ip` در Nginx**

```nginx
set_real_ip_from 10.0.0.0/8;
set_real_ip_from 127.0.0.1;
real_ip_header X-Forwarded-For;
real_ip_recursive on;
```

`real_ip_recursive on` از راست شروع می‌کند و اولین IP غیرمعتمد را کلاینت می‌گیرد.

**۵) `mod_remoteip` در Apache**

```apache
RemoteIPHeader X-Forwarded-For
RemoteIPTrustedProxy 10.0.0.0/8
```

**۶) IIS و `X-Forwarded-For`**

```xml
<system.webServer>
  <proxy enabled="true" preserveHostHeader="true" />
</system.webServer>
```

```csharp
// ASP.NET Core
app.UseForwardedHeaders(new ForwardedHeadersOptions {
    ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto
});
```

**۷) timeoutها**

| تنظیم | Nginx | Apache |
|---|---|---|
| اتصال | `proxy_connect_timeout 5s` | `connectiontimeout=5` |
| خواندن | `proxy_read_timeout 60s` | `timeout=60` |
| فرستادن | `proxy_send_timeout 60s` | — |
| کلاینت | `client_body_timeout 12s` | `RequestReadTimeout` |

**۸) `proxy_read_timeout` — انتخاب**

| اپ | مقدار |
|---|---|
| API سریع | `30s` |
| گزارش سنگین | `300s` |
| WebSocket | `3600s` |
| SSE | `24h` |

**۹) `proxy_buffering` و `proxy_request_buffering`**

```nginx
proxy_buffering off;            # پاسخ
proxy_request_buffering off;    # بدنه‌ی درخواست (آپلود stream)
```

**۱۰) `client_max_body_size` هماهنگ**

```nginx
client_max_body_size 32m;
proxy_read_timeout 300s;
```

و در اپ: `max_body_size`، PHP `upload_max_filesize`.

### مثال‌های عملی

**Lab Scenario: زنجیره‌ی دو پروکسی و IP واقعی**

```
کلاینت → proxy1 (Nginx :8080) → proxy2 (Nginx :8081) → اپ (:8000)
```

```nginx
# proxy1
server {
    listen 8080;
    location / {
        proxy_pass http://127.0.0.1:8081;
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```nginx
# proxy2
server {
    listen 8081;

    set_real_ip_from 127.0.0.1;
    real_ip_header X-Forwarded-For;
    real_ip_recursive on;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
curl -sS -H 'X-Forwarded-For: 203.0.113.45' http://localhost:8080/debug/headers
# {
#   "x-real-ip": "203.0.113.45",
#   "x-forwarded-for": "203.0.113.45, 127.0.0.1",
#   ...
# }
```

**تست جعل:**

```bash
# بدون real_ip: سرور IP جعلی را باور می‌کند
curl -sS -H 'X-Forwarded-For: 1.2.3.4' http://localhost:8080/debug/headers | grep -i real

# با real_ip و set_real_ip_from درست:
# 1.2.3.4 پذیرفته می‌شود چون 127.0.0.1 معتمد است (سناریوی تست)
```

> در تولید، `set_real_ip_from` باید فقط IP پروکسی واقعی باشد، نه `0.0.0.0/0`.

**`X-Forwarded-Proto` و redirect:**

```nginx
server {
    listen 80;
    server_name lab.example.com;

    # اگر پشت پروکسی HTTPS هستیم، redirect نکن
    if ($http_x_forwarded_proto = "https") {
        return 301 https://$host$request_uri;
    }
}
```

بهتر: `map`:

```nginx
map $http_x_forwarded_proto $forwarded_scheme {
    default $scheme;
    https   https;
}

server {
    listen 80;
    if ($forwarded_scheme != "https") {
        return 301 https://$host$request_uri;
    }
}
```

**اپ Python:**

```python
from fastapi import FastAPI, Request
from uvicorn.middleware.proxy_headers import ProxyHeadersMiddleware

app = FastAPI()
app.add_middleware(ProxyHeadersMiddleware, trusted_hosts="127.0.0.1")

@app.get("/whoami")
def whoami(request: Request):
    return {
        "client": request.client.host,
        "scheme": request.url.scheme,
        "host": request.headers.get("host"),
    }
```

**`proxy_read_timeout` و `504`:**

```nginx
location /slow/ {
    proxy_pass http://127.0.0.1:8000/;
    proxy_connect_timeout 3s;
    proxy_read_timeout 30s;
    proxy_send_timeout 30s;
}
```

```bash
# اپ با تأخیر ۴۰ ثانیه
curl -sS -o /dev/null -w '%{http_code} %{time_total}\n' http://localhost:8080/slow/delay/40
# 504 30.0
```

**آپلود stream:**

```nginx
location /upload/ {
    client_max_body_size 0;          # نامحدود (مراقب باش!)
    proxy_request_buffering off;
    proxy_pass http://127.0.0.1:8000/upload/;
}
```

### مقایسه و الگوهای مختلف

**`X-Forwarded-For` در برابر `X-Real-IP` در برابر `Forwarded`**

| | XFF | X-Real-IP | Forwarded |
|---|---|---|---|
| زنجیره | ✅ | ❌ | ✅ |
| استاندارد | ❌ | ❌ | ✅ |
| پشتیبانی | ✅✅ | ✅ | ⚠️ |
| توصیه | ✅ | مکمل | مدرن |

**`real_ip_recursive on` در برابر `off`**

| | `on` | `off` |
|---|---|---|
| چند پروکسی | ✅ | ❌ |
| امنیت | ✅ | ⚠️ |
| توصیه | ✅ | — |

**timeout کوتاه در برابر بلند**

| | کوتاه (۵s) | بلند (۳۰۰s) |
|---|---|---|
| `504` سریع | ✅ | ❌ |
| گزارش سنگین | ❌ | ✅ |
| توصیه | API | گزارش |

**دیدگاه «`X-Forwarded-For` را باور کن»**: فقط اگر همه‌ی ورودی‌ها از پروکسی خودت باشند. وگرنه، IP جعلی و دور زدن rate limit.

### اشکال شایع

**۱) `set_real_ip_from 0.0.0.0/0`**
همه را باور می‌کند. فاجعه.

**۲) `real_ip_recursive off` با چند پروکسی**
IP اشتباه.

**۳) `X-Forwarded-Proto` تنظیم نشده**
redirect loop.

**۴) `proxy_read_timeout 60s` برای گزارش ۵ دقیقه‌ای**
`504`. افزایش یا async job.

**۵) `client_max_body_size` و اپ ناهماهنگ**
`413` از Nginx، یا خطای اپ.

**۶) `proxy_request_buffering on` برای آپلود بزرگ**
حافظه/دیسک. `off`.

**۷) `Host` با پورت در اپ**
`$host` بدون پورت. `$http_host`.

**۸) `trust proxy` در Express نبودن**
`req.ip` = `127.0.0.1`.

**۹) `ForwardedHeaders` در ASP.NET Core نبودن**
`Request.Scheme` = `http`، redirect loop.

**۱۰) لاگ بدون IP واقعی**
تحلیل بی‌معنا. `LogFormat` با `%a` بعد از `remoteip`.

### نکات مهم (Tips & Tricks)

- **`set_real_ip_from`** فقط پروکسی خودت.
- **`real_ip_recursive on`**.
- **`X-Forwarded-Proto`** برای redirect.
- **`proxy_read_timeout`** بر اساس اپ.
- **`proxy_request_buffering off`** برای آپلود stream.
- **`trust proxy`** در اپ.
- **`curl -H 'X-Forwarded-For'`** برای تست.

### خلاصه فصل

- پشت پروکسی، IP و پروتکل گم می‌شوند.
- `X-Forwarded-*` استاندارد، ولی قابل جعل.
- `set_real_ip_from` + `real_ip_recursive on` راه درست.
- `X-Forwarded-Proto` از redirect loop جلوگیری می‌کند.
- timeout را با اپ هماهنگ کن.
- اپ باید `trust proxy` داشته باشد.

**سؤال مروری:** چرا `X-Forwarded-Proto` نبودن باعث redirect loop می‌شود؟

---

## فصل ۳۵: WebSocket و SSE از پشت پروکسی

### مقدمه

WebSocket با `Upgrade` از HTTP شروع می‌شود و پروکسی‌ها به‌طور پیش‌فرض آن را نمی‌فهمند. SSE هم با بافر شدن، بی‌صدا خراب می‌شود. این فصل، پیکربندی دقیق هر سه سرور است.

### مفاهیم اساسی

**۱) چرا WebSocket می‌شکند**

| مشکل | علت |
|---|---|
| `400`/`426` | `Upgrade`/`Connection` حذف شده |
| قطع بعد از ۶۰s | `proxy_read_timeout` |
| پیام‌ها دیر می‌رسند | `proxy_buffering on` |
| `502` | `proxy_http_version 1.0` |

**۲) Nginx — حداقل لازم**

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

location /ws {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade    $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_set_header Host       $host;
    proxy_read_timeout 3600s;
    proxy_send_timeout 3600s;
    proxy_buffering off;
}
```

**۳) چرا `map` لازم است**

اگر `Connection: upgrade` را hardcode کنی، درخواست‌های عادی هم `upgrade` می‌گیرند و بعضی upstreamها خطا می‌دهند. `map` مقدار مناسب را انتخاب می‌کند.

**۴) Apache — `mod_proxy_wstunnel`**

```apache
<Location /ws>
    ProxyPass        ws://127.0.0.1:3000/ws
    ProxyPassReverse ws://127.0.0.1:3000/ws
</Location>
```

**۵) IIS + ARR**

```xml
<system.webServer>
  <webSocket enabled="true" pingInterval="00:00:30" />
</system.webServer>
```

و ARR باید فعال باشد.

**۶) SSE — سه تنظیم**

```nginx
location /events {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_buffering off;
    proxy_cache off;
    chunked_transfer_encoding off;
    proxy_read_timeout 24h;
}
```

و از سمت اپ:

```javascript
res.setHeader('X-Accel-Buffering', 'no');
```

**۷) heartbeat**

پروکسی‌ها اتصال بی‌صدا را می‌بندند. هر ۳۰ ثانیه ping:

```javascript
setInterval(() => {
  if (ws.readyState === ws.OPEN) ws.ping();
}, 30000);
```

**۸) مقیاس WebSocket**

| مسئله | راه‌حل |
|---|---|
| چند سرور | pub/sub (Redis) |
| sticky | `ip_hash` یا stateless |
| تعداد اتصال | `worker_connections`، fd |
| timeout | heartbeat + timeout طولانی |

**۹) `Sec-WebSocket-*` و امنیت**

- `Origin` را چک کن (CSWSH).
- `wss://` در تولید.
- احراز هویت در handshake (کوکی/توکن).

**۱۰) `proxy_buffering off` — چرا**

با `on`، Nginx پاسخ را جمع می‌کند تا بافر پر شود. برای SSE و WebSocket، پیام باید فوری برسد.

### مثال‌های عملی

**Lab Scenario: چت ساده با Nginx**

```javascript
// server.js — npm i ws express
import express from 'express';
import http from 'node:http';
import { WebSocketServer } from 'ws';

const app = express();
app.get('/health', (req, res) => res.json({ ok: true }));

const server = http.createServer(app);
const wss = new WebSocketServer({ server, path: '/ws' });

wss.on('connection', (ws, req) => {
  const ip = req.headers['x-real-ip'] ?? req.socket.remoteAddress;
  console.log('connect', ip);

  const t = setInterval(() => {
    if (ws.readyState === ws.OPEN) ws.send(JSON.stringify({ t: Date.now() }));
  }, 1000);

  ws.on('message', (d) => ws.send(JSON.stringify({ echo: d.toString() })));
  ws.on('close', () => { clearInterval(t); console.log('disconnect', ip); });
});

server.listen(3000, '127.0.0.1');
```

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

upstream ws_app {
    server 127.0.0.1:3000;
    keepalive 16;
}

server {
    listen 8080;
    server_name _;

    location /ws {
        proxy_pass http://ws_app;
        proxy_http_version 1.1;
        proxy_set_header Upgrade    $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host       $host;
        proxy_set_header X-Real-IP  $remote_addr;
        proxy_read_timeout 3600s;
        proxy_send_timeout 3600s;
        proxy_buffering off;
    }

    location /events {
        proxy_pass http://ws_app;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_buffering off;
        proxy_cache off;
        chunked_transfer_encoding off;
        proxy_read_timeout 24h;
    }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx

# تست handshake
curl -sS -i -N \
  -H 'Connection: Upgrade' \
  -H 'Upgrade: websocket' \
  -H 'Sec-WebSocket-Version: 13' \
  -H 'Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==' \
  http://localhost:8080/ws | head -8
# HTTP/1.1 101 Switching Protocols
# Upgrade: websocket
# Connection: Upgrade

# با websocat
websocat ws://localhost:8080/ws
```

**SSE با اپ:**

```javascript
app.get('/events', (req, res) => {
  res.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive',
    'X-Accel-Buffering': 'no',
  });

  let id = 0;
  const t = setInterval(() => {
    res.write(`id: ${++id}\n`);
    res.write(`data: ${JSON.stringify({ t: Date.now() })}\n\n`);
  }, 1000);

  req.on('close', () => clearInterval(t));
});
```

```bash
curl -N http://localhost:8080/events
# id: 1
# data: {"t":...}
```

**Apache:**

```apache
<VirtualHost *:8080>
    ServerName lab.example.com

    <Location /ws>
        ProxyPass        ws://127.0.0.1:3000/ws
        ProxyPassReverse ws://127.0.0.1:3000/ws
    </Location>

    <Location /events>
        ProxyPass        http://127.0.0.1:3000/events
        ProxyPassReverse http://127.0.0.1:3000/events
        SetEnv proxy-sendchunked 1
        SetEnv proxy-sendcl 0
    </Location>
</VirtualHost>
```

```bash
sudo a2enmod proxy proxy_http proxy_wstunnel
sudo apachectl configtest && sudo systemctl reload apache2
```

**IIS:**

```xml
<system.webServer>
  <webSocket enabled="true" pingInterval="00:00:30" />
  <rewrite>
    <rules>
      <rule name="WS" stopProcessing="true">
        <match url="^ws$" />
        <action type="Rewrite" url="http://127.0.0.1:3000/ws" />
      </rule>
    </rules>
  </rewrite>
</system.webServer>
```

```powershell
# ARR و WebSocket باید نصب باشند
Install-WindowsFeature Web-WebSockets
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/proxy" -Name "enabled" -Value $true
```

**تست قطع بعد از ۶۰ ثانیه:**

```bash
# قبل از تنظیم timeout
timeout 120 websocat ws://localhost:8080/ws
# بعد از ~۶۰ ثانیه قطع می‌شود

# بعد از proxy_read_timeout 3600s
# پایدار می‌ماند
```

**مقیاس با Redis pub/sub:**

```javascript
import { createClient } from 'redis';
import { WebSocketServer } from 'ws';

const pub = createClient();
const sub = pub.duplicate();
await pub.connect();
await sub.connect();

const wss = new WebSocketServer({ port: 3000, path: '/ws' });

sub.subscribe('chat', (msg) => {
  for (const c of wss.clients) {
    if (c.readyState === c.OPEN) c.send(msg);
  }
});

wss.on('connection', (ws) => {
  ws.on('message', (d) => pub.publish('chat', d.toString()));
});
```

### مقایسه و الگوهای مختلف

**WebSocket در برابر SSE در پروکسی**

| | WebSocket | SSE |
|---|---|---|
| `Upgrade` | ✅ لازم | ❌ |
| `proxy_http_version` | `1.1` | `1.1` |
| `buffering` | `off` | `off` |
| timeout | طولانی | خیلی طولانی |
| reconnect | دستی | ✅ خودکار |

**سه راه sticky**

| راه | ابزار |
|---|---|
| IP hash | `ip_hash` (Nginx) |
| Cookie | `sticky` (Nginx Plus)، ARR affinity |
| Stateless | pub/sub، توکن |

**`proxy_read_timeout` برای WebSocket**

| مقدار | نتیجه |
|---|---|
| `60s` | قطع هر دقیقه |
| `3600s` | ✅ پایدار |
| `24h` | برای SSE |

**دیدگاه «WebSocket مقیاس‌پذیر نیست»**: با pub/sub و sticky درست، مقیاس‌پذیر است. مشکل، state در حافظه‌ی هر سرور است.

### اشکال شایع

**۱) `Upgrade` حذف‌شده**
`400`/`426`. هدر لازم.

**۲) `Connection: upgrade` hardcode**
درخواست عادی هم upgrade. `map`.

**۳) `proxy_http_version 1.0`**
WebSocket شکسته.

**۴) `proxy_read_timeout 60s`**
قطع هر دقیقه.

**۵) `proxy_buffering on` برای SSE**
پیام‌ها جمع می‌شوند.

**۶) `X-Accel-Buffering` نبودن**
Nginx بافر می‌کند. هدر از اپ.

**۷) نبود heartbeat**
پروکسی اتصال بی‌صدا را می‌بندد.

**۸) `Origin` چک نمی‌شود**
CSWSH.

**۹) `wss://` نبودن**
داده‌ی چت روی HTTP.

**۱۰) `worker_connections` کم**
۱۰ هزار WebSocket → اتصال رد.

### نکات مهم (Tips & Tricks)

- **`map $http_upgrade`** استاندارد.
- **`proxy_buffering off`** برای stream.
- **`X-Accel-Buffering: no`** از اپ.
- **heartbeat** هر ۳۰ ثانیه.
- **`websocat`** برای تست.
- **`curl -N`** برای SSE.
- **pub/sub** برای چند سرور.

### خلاصه فصل

- WebSocket با `Upgrade` شروع می‌شود؛ پروکسی باید هدر را پاس بدهد.
- `map` از hardcode `Connection` جلوگیری می‌کند.
- `proxy_read_timeout` طولانی + heartbeat.
- SSE نیاز به `buffering off` و `X-Accel-Buffering: no`.
- مقیاس با pub/sub و sticky.
- `wss://` و بررسی `Origin` الزامی.

**سؤال مروری:** چرا `proxy_buffering on` برای SSE باعث می‌شود پیام‌ها با تأخیر برسند؟

---

## منابع و مراجع

**مستندات رسمی**
- [Nginx — ngx_http_proxy_module](https://nginx.org/en/docs/http/ngx_http_proxy_module.html)
- [Nginx — WebSocket proxying](https://nginx.org/en/docs/http/websocket.html)
- [Nginx — reverse proxy guide](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
- [Apache — mod_proxy](https://httpd.apache.org/docs/2.4/mod/mod_proxy.html)
- [Apache — mod_proxy_wstunnel](https://httpd.apache.org/docs/2.4/mod/mod_proxy_wstunnel.html)
- [Apache — mod_remoteip](https://httpd.apache.org/docs/2.4/mod/mod_remoteip.html)
- [IIS — ARR](https://learn.microsoft.com/en-us/iis/extensions/planning-for-arr/application-request-routing-version-3)

**آموزش**
- [Nginx reverse proxy — DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-as-a-reverse-proxy)
- [X-Forwarded-For explained](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For)
- [RFC 7239 — Forwarded](https://www.rfc-editor.org/rfc/rfc7239.html)

**ابزار**
- [websocat](https://github.com/vi/websocat)
- [curl](https://curl.se/docs/)

---

**بعدی:** [`09-Load-Balancing.md`](09-Load-Balancing.md) — L4/L7، الگوریتم‌ها، HAProxy، ARR و sticky session
