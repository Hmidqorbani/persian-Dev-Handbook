# بخش ۰۳: معماری سه وب‌سرور و انتخاب درست

**مدت یادگیری**: ۴ تا ۵ ساعت
**سطح سختی**: مبتدی تا متوسط
**پیش‌نیاز**: بخش ۰۲ (شبکه، سوکت، `epoll`)

---

## فهرست محتوا

- [فصل ۹: Apache — MPM، ماژول و `.htaccess`](#فصل-۹)
- [فصل ۱۰: Nginx — master/worker و event loop](#فصل-۱۰)
- [فصل ۱۱: IIS — `HTTP.sys` و Application Pool](#فصل-۱۱)
- [فصل ۱۲: مقایسه، انتخاب و گزینه‌های دیگر](#فصل-۱۲)

---

## فصل ۹: Apache — MPM، ماژول و `.htaccess`

### مقدمه

Apache در ۱۹۹۵ شروع شد و برای سال‌ها ۷۰٪ وب را می‌گرداند. طراحی‌اش بر «انعطاف» است، نه «حداکثر کارایی»: هر قابلیت یک ماژول است، پیکربندی می‌تواند در `.htaccess` بدون restart تغییر کند، و هر اتصال می‌تواند یک پروسه باشد. همین انعطاف، آن را برای هاست اشتراکی و وردپرس ایده‌آل کرده.

### مفاهیم اساسی

**۱) MPM — Multi-Processing Module**

Apache سه MPM دارد و انتخابش، سرنوشت کارایی را تعیین می‌کند:

| MPM | مدل | مناسب |
|---|---|---|
| `prefork` | پروسه به ازای اتصال | `mod_php`، کد غیر-thread-safe |
| `worker` | پروسه + ترد | عمومی، `php-fpm` |
| `event` | ترد + `epoll` برای keep-alive | **پیش‌فرض مدرن** |

```
prefork:
┌────────┐ ┌────────┐ ┌────────┐
│process │ │process │ │process │   هر اتصال یک پروسه
└────────┘ └────────┘ └────────┘

worker:
┌──────────────────┐
│ process          │
│ ├─thread ├─thread│  چند ترد در هر پروسه
└──────────────────┘

event:
┌──────────────────────────┐
│ process                  │
│ ├─thread ├─thread        │
│ └─ listener (epoll)      │  اتصال‌های idle جدا مدیریت می‌شوند
└──────────────────────────┘
```

**۲) `event` MPM چطور کار می‌کند**

در `worker`، هر اتصال `keep-alive` یک ترد را اشغال می‌کند، حتی وقتی کاری نمی‌کند. در `event`، یک ترد اختصاصی (`listener`) اتصال‌های idle را با `epoll` نگه می‌دارد و فقط وقتی درخواستی بیاید، آن را به یک ترد کاری می‌دهد. نتیجه: با همان حافظه، چند برابر اتصال.

**۳) ماژول‌ها**

| ماژول | کار |
|---|---|
| `mod_ssl` | TLS |
| `mod_rewrite` | بازنویسی URL |
| `mod_proxy`، `mod_proxy_http` | reverse proxy |
| `mod_proxy_fcgi` | PHP-FPM |
| `mod_headers` | دستکاری هدر |
| `mod_deflate` | gzip |
| `mod_expires` | `Cache-Control` |
| `mod_security` | WAF |
| `mod_remoteip` | IP واقعی پشت پروکسی |
| `mod_status` | مانیتورینگ |
| `mod_ratelimit` | محدودیت نرخ |

**۴) ساختار پیکربندی (تفاوت توزیع‌ها)**

| توزیع | فایل اصلی | سایت‌ها |
|---|---|---|
| Debian/Ubuntu | `/etc/apache2/apache2.conf` | `/etc/apache2/sites-available/` + `sites-enabled/` |
| RHEL/Rocky | `/etc/httpd/conf/httpd.conf` | `/etc/httpd/conf.d/*.conf` |

**۵) `.htaccess` — قدرت و خطر**

`.htaccess` فایلی در `DocumentRoot` است که پیکربندی را **در زمان درخواست** بازنویسی می‌کند.

| مزیت | عیب |
|---|---|
| بدون restart | ❌ کندی (هر درخواست، خواندن فایل) |
| کنترل برای کاربر بدون دسترسی root | ❌ خطر امنیتی |
| رایج در هاست اشتراکی | ❌ `AllowOverride All` گران است |

**۶) `AllowOverride`**

```apache
<Directory /var/www/html>
    AllowOverride None      # .htaccess نادیده (سریع‌ترین)
    # AllowOverride All     # همه‌چیز مجاز (کند)
    # AllowOverride FileInfo Indexes   # فقط دستورهای مشخص
</Directory>
```

**۷) `Directory`، `Location`، `Files`**

| بخش | روی چه اثر می‌گذارد |
|---|---|
| `<Directory /path>` | مسیر فیزیکی |
| `<Location /url>` | مسیر URL |
| `<Files "*.php">` | نام فایل |
| `<IfModule>` | اگر ماژول فعال بود |

**۸) ترتیب پردازش**

```
۱. <Directory> (پوشه‌ی فایل)
۲. .htaccess
۳. <Directory> داخل .htaccess
۴. <Files>
۵. <Location>
```

`Location` بعد از `Directory` اعمال می‌شود، پس می‌تواند آن را override کند.

**۹) `mod_php` در برابر `php-fpm`**

| | `mod_php` | `php-fpm` |
|---|---|---|
| MPM | `prefork` اجباری | `event`/`worker` |
| حافظه | ❌ در هر پروسه | ✅ استخر جدا |
| کارایی | ⚠️ | ✅ |
| `.htaccess` | ✅ | ⚠️ (بخشی) |
| توصیه | legacy | ✅ مدرن |

**۱۰) `apachectl` و `systemctl`**

```bash
sudo apachectl configtest      # تست سینتکس
sudo apachectl -M              # لیست ماژول‌ها
sudo apachectl -S              # لیست vhostها
sudo apachectl -V              # نسخه و مسیرها
sudo systemctl reload apache2  # اعمال بدون قطع
```

### مثال‌های عملی

**Lab Scenario: تغییر MPM و اندازه‌گیری**

```bash
# ۱) MPM فعلی
sudo apachectl -V | grep -i mpm
# Server MPM: event

# ۲) ماژول‌های فعال
sudo apachectl -M | sort | head -40
```

**پیکربندی `event` برای سرور ۸ هسته / ۱۶ گیگ:**

```apache
# /etc/apache2/mods-available/mpm_event.conf
<IfModule mpm_event_module>
    StartServers              4
    MinSpareThreads           75
    MaxSpareThreads           250
    ThreadsPerChild           32
    MaxRequestWorkers         800
    MaxConnectionsPerChild    10000
    ServerLimit               25
</IfModule>
```

```bash
sudo a2dismod mpm_prefork 2>/dev/null
sudo a2enmod mpm_event
sudo apachectl configtest && sudo systemctl restart apache2
```

> `MaxRequestWorkers` = `ServerLimit × ThreadsPerChild`. اگر بیشتر بگذاری، Apache هشدار می‌دهد و کم می‌کند.

**پیکربندی `prefork` (اگر `mod_php` داری):**

```apache
<IfModule mpm_prefork_module>
    StartServers          8
    MinSpareServers       8
    MaxSpareServers       32
    MaxRequestWorkers     256
    MaxConnectionsPerChild 5000
</IfModule>
```

**`.htaccess` نمونه (وردپرس):**

```apache
# BEGIN WordPress
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
</IfModule>
# END WordPress
```

**معادل در vhost (سریع‌تر از `.htaccess`):**

```apache
<Directory /var/www/wordpress>
    AllowOverride None
    <IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteBase /
        RewriteRule ^index\.php$ - [L]
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteCond %{REQUEST_FILENAME} !-d
        RewriteRule . /index.php [L]
    </IfModule>
</Directory>
```

**تست تفاوت کارایی `.htaccess`:**

```bash
# با AllowOverride All
ab -n 5000 -c 50 http://lab.example.com/

# با AllowOverride None (بعد از انتقال قواعد به vhost)
ab -n 5000 -c 50 http://lab.example.com/
```

معمولاً ۵ تا ۲۰٪ تفاوت، بسته به تعداد فایل‌های `.htaccess` در مسیر.

**بررسی `mod_status`:**

```apache
<Location /server-status>
    SetHandler server-status
    Require local
    # یا: Require ip 10.0.0.0/8
</Location>
```

```bash
curl -s http://localhost/server-status?auto
# Total Accesses: 1234
# Total kBytes: 5678
# BusyWorkers: 3
# IdleWorkers: 47
```

### مقایسه و الگوهای مختلف

**سه MPM**

| معیار | prefork | worker | event |
|---|---|---|---|
| مدل | پروسه | پروسه+ترد | پروسه+ترد+epoll |
| حافظه در ۱۰k اتصال | ❌❌ | ⚠️ | ✅ |
| `mod_php` | ✅ | ❌ | ❌ |
| thread-safe لازم | ❌ | ✅ | ✅ |
| توصیه | legacy | ⚠️ | ✅ |

**`.htaccess` در برابر vhost**

| | `.htaccess` | vhost |
|---|---|---|
| سرعت | ❌ | ✅ |
| نیاز به restart | ❌ | ✅ |
| کنترل کاربر | ✅ | ❌ |
| امنیت | ⚠️ | ✅ |
| مناسب | هاست اشتراکی | تولید |

**`mod_php` در برابر `php-fpm`**

| | `mod_php` | `php-fpm` |
|---|---|---|
| MPM | prefork | event |
| حافظه | ❌ | ✅ |
| opcache مشترک | ⚠️ | ✅ |
| تنظیم per-pool | ❌ | ✅ |
| توصیه | ❌ | ✅ |

**دیدگاه «Apache کند است»**: با `event` MPM و `php-fpm`، Apache برای اکثر بارها کافی است. کندی معمولاً از `prefork` + `mod_php` + `AllowOverride All` می‌آید، نه از خود Apache.

### اشکال شایع

**۱) `prefork` + `event` همزمان**
Apache بالا نمی‌آید: `Apache is running a threaded MPM, but your PHP Module is not compiled to be threadsafe`. یکی را انتخاب کن.

**۲) `MaxRequestWorkers` بیشتر از `ServerLimit × ThreadsPerChild`**
هشدار و کاهش خودکار. فرمول را رعایت کن.

**۳) `AllowOverride All` در تولید**
کندی و خطر. قواعد را به vhost منتقل کن.

**۴) `.htaccess` با خطای سینتکس**
`500 Internal Server Error` بدون پیام واضح. `error_log` را ببین.

**۵) فراموش کردن `a2enmod`**
`Invalid command 'RewriteEngine'`. ماژول فعال نیست.

**۶) `Directory` بدون `/` پایانی**
`<Directory /var/www/html>` درست است، `<Directory /var/www/html/>` هم کار می‌کند ولی در بعضی نسخه‌ها رفتار متفاوت. یکدست باش.

**۷) `Require all granted` فراموش‌شده**
`403 Forbidden` در Apache 2.4 (در 2.2 با `Order allow,deny` کار می‌کرد).

**۸) `KeepAlive Off`**
هر درخواست، اتصال جدید. `KeepAlive On` + `MaxKeepAliveRequests 100`.

**۹) لاگ بدون `%D`**
زمان پاسخ را نمی‌بینی. `LogFormat` را کامل کن (→ فصل ۴۷).

**۱۰) reload بدون `configtest`**
سایت با config خراب، از دسترس خارج می‌شود. همیشه `apachectl configtest` اول.

### نکات مهم (Tips & Tricks)

- **`apachectl -S`** همه‌ی vhostها و مسیر config را نشان می‌دهد.
- **`apachectl -M`** ماژول‌های فعال.
- **`apachectl configtest`** قبل از هر reload.
- **`event` + `php-fpm`** ترکیب مدرن.
- **`AllowOverride None`** در تولید.
- **`mod_status`** برای مانیتورینگ سبک.
- **`LogLevel warn`** به‌جای `debug` در تولید.

### خلاصه فصل

- Apache سه MPM دارد؛ `event` انتخاب مدرن است.
- `mod_php` نیاز به `prefork` دارد؛ `php-fpm` آزادتر است.
- `.htaccess` انعطاف می‌دهد ولی کند و پرخطر است.
- `Directory`/`Location`/`Files` سطوح مختلف پیکربندی‌اند.
- `apachectl -S`/`-M`/`configtest` ابزارهای روزمره.

**سؤال مروری:** چرا `event` MPM با `keep-alive` کارایی بهتری از `worker` دارد؟

---

## فصل ۱۰: Nginx — master/worker و event loop

### مقدمه

Nginx در ۲۰۰۴ برای حل مشکل C10k (ده هزار اتصال همزمان) نوشته شد. ایده‌ی اصلی: **یک حلقه‌ی رویداد، بدون ترد به ازای هر اتصال**. نتیجه: مصرف حافظه‌ی کم، تأخیر پایین، و توانایی سرو ده‌ها هزار اتصال روی یک سرور معمولی.

### مفاهیم اساسی

**۱) مدل پروسه**

```
┌───────────────────────────────┐
│ master process (root)         │
│  - خواندن config              │
│  - bind پورت‌ها                │
│  - مدیریت workerها            │
└──────────┬────────────────────┘
           │ fork
     ┌─────┴─────┬─────────┐
     ▼           ▼         ▼
┌─────────┐ ┌─────────┐ ┌─────────┐
│ worker 1│ │ worker 2│ │ worker 3│   (هر کدام یک هسته)
│ epoll   │ │ epoll   │ │ epoll   │
└─────────┘ └─────────┘ └─────────┘
```

- **master** با `root` اجرا می‌شود (برای bind پورت <۱۰۲۴) و بعد به کاربر کم‌دسترسی (`www-data`/`nginx`) تغییر می‌کند.
- **worker**ها با کاربر کم‌دسترسی اجرا می‌شوند.
- تعداد worker معمولاً = تعداد هسته (`worker_processes auto;`).

**۲) event loop**

```
while (true) {
    events = epoll_wait(fd_set);      // منتظر رویداد
    for (e : events) {
        if (e.type == READ)  read_request(e);
        if (e.type == WRITE) write_response(e);
    }
}
```

هیچ عملیاتی **بلاک** نمی‌کند. اگر I/O دیسک لازم است، Nginx از `aio`/`sendfile`/thread pool استفاده می‌کند.

**۳) `worker_connections`**

حداکثر اتصال هر worker. سقف کل ≈ `worker_processes × worker_connections`.

ولی هر اتصال می‌تواند **دو** سوکت باشد (کلاینت + upstream)، پس عملاً نصف.

**۴) `reuseport`**

```nginx
listen 80 reuseport;
```

کرنل بار را بین workerها پخش می‌کند؛ بدون قفل مشترک روی `accept`.

**۵) ماژول‌ها — کامپایل‌شده**

Nginx ماژول‌ها را در زمان build کامپایل می‌کند (برخلاف Apache). برای ماژول جدید، باید دوباره build کنی (یا از `nginx-extras`/`openresty` استفاده کنی).

```bash
nginx -V 2>&1 | tr ' ' '\n' | grep module
```

**۶) ساختار پیکربندی — context**

```nginx
# main context
user www-data;
worker_processes auto;

events {                       # context: events
    worker_connections 16384;
}

http {                         # context: http
    include /etc/nginx/mime.types;
    sendfile on;

    upstream backend {         # context: upstream
        server 127.0.0.1:3000;
    }

    server {                   # context: server
        listen 80;
        server_name lab.example.com;

        location / {           # context: location
            root /var/www/lab;
            try_files $uri $uri/ =404;
        }

        location /api/ {
            proxy_pass http://backend;
        }
    }
}
```

**۷) ارث‌بری (inheritance)**

بسیاری از دستورها از context والد ارث می‌برند. اگر در `location` تعریف شود، والد را override می‌کند. **`add_header` ارث نمی‌برد اگر در همان سطح `add_header` دیگری باشد** — تله‌ی معروف.

**۸) `include` و ساختار فایل**

```
/etc/nginx/
├── nginx.conf
├── conf.d/
│   └── *.conf
├── sites-available/
│   └── lab.conf
├── sites-enabled/
│   └── lab.conf -> ../sites-available/lab.conf
├── snippets/
│   ├── ssl.conf
│   └── security-headers.conf
└── mime.types
```

**۹) `nginx -t` و `-s reload`**

```bash
sudo nginx -t                 # تست config
sudo nginx -T                 # تست + چاپ کل config (با include)
sudo nginx -s reload          # reload بدون قطع
sudo nginx -s quit            # خروج تمیز
sudo nginx -s stop            # خروج فوری
```

**reload چطور کار می‌کند:** master config جدید را می‌خواند، workerهای جدید fork می‌کند، workerهای قدیمی `graceful shutdown` می‌شوند (اتصال‌های فعال تمام می‌شوند). **بدون قطع سرویس.**

**۱۰) `sendfile`، `tcp_nopush`، `tcp_nodelay`**

```nginx
sendfile        on;    # کپی مستقیم از دیسک به سوکت (kernel)
tcp_nopush      on;    # بسته‌ها را جمع کن (با sendfile)
tcp_nodelay     on;    # برای keep-alive، بسته‌های کوچک فوری
```

**۱۱) `open_file_cache`**

کش متادیتای فایل (نه محتوا):

```nginx
open_file_cache          max=10000 inactive=60s;
open_file_cache_valid    30s;
open_file_cache_min_uses 2;
open_file_cache_errors   on;
```

### مثال‌های عملی

**Lab Scenario: پیکربندی پایه و اندازه‌گیری**

```nginx
# /etc/nginx/nginx.conf
user  www-data;
worker_processes  auto;
worker_rlimit_nofile 65535;

events {
    worker_connections  16384;
    multi_accept        on;
    use                 epoll;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" rt=$request_time urt=$upstream_response_time';

    access_log /var/log/nginx/access.log main;
    error_log  /var/log/nginx/error.log warn;

    sendfile        on;
    tcp_nopush      on;
    tcp_nodelay     on;
    keepalive_timeout  65;
    types_hash_max_size 2048;
    server_tokens   off;

    client_max_body_size 16m;

    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml image/svg+xml;
    gzip_min_length 1024;

    include /etc/nginx/conf.d/*.conf;
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx

# بررسی تعداد worker
ps -eo pid,ppid,cmd | grep 'nginx: worker'

# وضعیت
sudo systemctl status nginx
curl -sSI http://localhost/ | head -3
```

**`stub_status` برای مانیتورینگ:**

```nginx
server {
    listen 127.0.0.1:8081;
    location /nginx_status {
        stub_status;
        allow 127.0.0.1;
        deny all;
    }
}
```

```bash
curl -s http://127.0.0.1:8081/nginx_status
# Active connections: 3
# server accepts handled requests
#  1234 1234 5678
# Reading: 0 Writing: 1 Waiting: 2
```

| فیلد | معنا |
|---|---|
| `accepts` | کل اتصال‌های پذیرفته |
| `handled` | موفق (اگر کمتر از `accepts`، محدودیت منابع) |
| `requests` | کل درخواست |
| `Reading` | در حال خواندن هدر |
| `Writing` | در حال نوشتن پاسخ |
| `Waiting` | idle (keep-alive) |

**تست بار:**

```bash
# نصب
sudo apt install -y apache2-utils

# ۱۰ هزار درخواست، ۱۰۰ همزمان، با keep-alive
ab -k -n 10000 -c 100 http://lab.example.com/

# با hey (خروجی بهتر)
hey -n 50000 -c 200 http://lab.example.com/
```

**مقایسه‌ی `worker_processes`:**

```bash
# ۱ worker
sudo sed -i 's/worker_processes.*/worker_processes 1;/' /etc/nginx/nginx.conf
sudo nginx -s reload && ab -k -n 10000 -c 100 http://localhost/ | grep 'Requests per second'

# auto (تعداد هسته)
sudo sed -i 's/worker_processes.*/worker_processes auto;/' /etc/nginx/nginx.conf
sudo nginx -s reload && ab -k -n 10000 -c 100 http://localhost/ | grep 'Requests per second'
```

### مقایسه و الگوهای مختلف

**Nginx در برابر Apache (معماری)**

| معیار | Nginx | Apache (event) |
|---|---|---|
| مدل | event loop | ترد + epoll |
| ماژول | کامپایل | پویا (`a2enmod`) |
| پیکربندی | متمرکز | متمرکز + `.htaccess` |
| reload | ✅ بدون قطع | ✅ بدون قطع |
| فایل استاتیک | ✅✅ | ✅ |
| پروکسی | ✅✅ | ✅ |
| `.htaccess` | ❌ | ✅ |
| پیچیدگی config | ⚠️ (`location` semantics) | ✅ ساده‌تر |

**`worker_processes`**

| مقدار | مناسب |
|---|---|
| `1` | تست، دیباگ |
| `auto` | ✅ تولید |
| `2 × cores` | ⚠️ (برای I/O-bound، گاهی) |

**`worker_connections`**

| مقدار | اتصال تقریبی (۴ worker) |
|---|---|
| ۱۰۲۴ | ~۲۰۰۰ |
| ۱۶۳۸۴ | ~۳۲٬۰۰۰ |
| ۶۵۵۳۵ | ~۱۳۰٬۰۰۰ |

**دیدگاه «Nginx همیشه سریع‌تر است»**: برای فایل استاتیک و پروکسی، بله. برای محتوای داینامیک با `.htaccess` پیچیده، تفاوت کم است. انتخاب بر اساس نیاز، نه بنچمارک خام.

### اشکال شایع

**۱) `nginx -t` موفق ولی reload خطا**
`include` فایلی که وجود ندارد، فقط در reload دیده می‌شود. `nginx -T` بزن.

**۲) `worker_connections` کم و `worker_connections are not enough`**
در `error.log`. بالا ببر + `worker_rlimit_nofile`.

**۳) `Too many open files`**
`worker_rlimit_nofile` و `limits.conf` (فصل ۵).

**۴) `add_header` ارث نمی‌برد**
اگر در `server` و `location` هر دو `add_header` باشد، فقط `location` اعمال می‌شود. `always` و تکرار در هر سطح.

**۵) `root` در `location` اشتباه**
`root` به مسیر اضافه می‌شود (`root + uri`)، `alias` جایگزین می‌کند. تفاوت مهم (فصل ۲۲).

**۶) `try_files` با `proxy_pass`**
ترتیب اشتباه، `404`. (فصل ۲۲)

**۷) `proxy_pass` با `/` پایانی**
`proxy_pass http://backend/;` مسیر را جایگزین می‌کند؛ بدون `/`، مسیر کامل را می‌فرستد. تله‌ی کلاسیک (فصل ۳۲).

**۸) config بدون `events`**
`nginx: [emerg] no "events" section`. ساختار را رعایت کن.

**۹) `user` اشتباه**
`403 Forbidden` چون worker نمی‌تواند فایل را بخواند.

**۱۰) reload بدون `-t`**
config خراب، workerهای جدید بالا نمی‌آیند و قدیمی‌ها می‌مانند (سرویس قطع نمی‌شود، ولی تغییر اعمال نمی‌شود). همیشه `-t` اول.

### نکات مهم (Tips & Tricks)

- **`nginx -T`** کل config بعد از `include` را چاپ می‌کند.
- **`nginx -V`** ماژول‌های کامپایل‌شده.
- **`worker_processes auto`**.
- **`reuseport`** برای پخش بار.
- **`stub_status`** برای مانیتورینگ سبک.
- **`open_file_cache`** برای فایل استاتیک زیاد.
- **`server_tokens off`** برای پنهان کردن نسخه.

### خلاصه فصل

- Nginx: master + worker، event loop، بدون بلاک.
- `worker_connections × worker_processes` سقف اتصال است.
- ماژول‌ها کامپایل می‌شوند، نه پویا.
- `location` و ارث‌بری، منبع بیشترین اشتباه‌اند.
- `reload` بدون قطع سرویس کار می‌کند.
- `nginx -t`/`-T`/`-V` ابزارهای روزمره.

**سؤال مروری:** چرا `worker_connections = 16384` با ۴ worker به معنای ۶۵٬۵۳۶ اتصال همزمان نیست؟

---

## فصل ۱۱: IIS — `HTTP.sys` و Application Pool

### مقدمه

IIS (Internet Information Services) وب‌سرور مایکروسافت است و روی ویندوز سرور اجرا می‌شود. تفاوت بنیادی‌اش با Nginx/Apache: بخش زیادی از HTTP در **کرنل** (`HTTP.sys`) پیاده شده، نه در userspace. نتیجه: کارایی خوب برای استاتیک و یکپارچگی عمیق با ASP.NET، ولی وابستگی به ویندوز.

### مفاهیم اساسی

**۱) معماری**

```
┌──────────────────────────────────────────────┐
│  HTTP.sys (کرنل)                             │
│  - گوش دادن به پورت‌ها                        │
│  - پارس HTTP، کش پاسخ استاتیک                 │
│  - صف درخواست برای هر Application Pool        │
└───────────────┬──────────────────────────────┘
                │
                ▼
┌──────────────────────────────────────────────┐
│  WAS (Windows Process Activation Service)     │
│  - مدیریت Application Poolها                  │
└───────────────┬──────────────────────────────┘
                │
                ▼
┌──────────────────────────────────────────────┐
│  w3wp.exe (worker process)                   │
│  - ASP.NET / ASP.NET Core / PHP / static     │
└──────────────────────────────────────────────┘
```

**۲) Application Pool**

یک `App Pool` = یک یا چند `w3wp.exe` با:
- هویت (`ApplicationPoolIdentity` یا کاربر دیگر)
- تنظیمات .NET
- محدودیت‌های CPU/حافظه
- `Idle Timeout`، `Recycling`

**چرا مهم است:** جداسازی. دو سایت در دو pool، خرابی یکی به دیگری سرایت نمی‌کند.

**۳) Site و Binding**

```
Site "Lab"
├── Binding: http  *:80:lab.example.com
├── Binding: https *:443:lab.example.com (SNI)
└── Physical Path: C:\inetpub\lab
```

`Binding` = `protocol + IP + port + hostname`. چند سایت روی یک IP با hostname متفاوت (مثل virtual host).

**۴) `web.config`**

معادل `.htaccess`، ولی XML و در ریشه‌ی اپ:

```xml
<configuration>
  <system.webServer>
    <defaultDocument>
      <files>
        <clear />
        <add value="index.html" />
      </files>
    </defaultDocument>
    <httpErrors errorMode="DetailedLocalOnly" />
  </system.webServer>
</configuration>
```

**۵) ماژول‌های IIS**

| ماژول | کار |
|---|---|
| `StaticFileModule` | فایل استاتیک |
| `DefaultDocumentModule` | `index.html` |
| `DirectoryBrowsingModule` | لیست پوشه (خاموش کن) |
| `HttpRedirectionModule` | redirect |
| `UrlRewriteModule` | بازنویسی (افزونه) |
| `HttpCompressionModule` | gzip/brotli |
| `WebSocketsModule` | WebSocket |
| `AspNetCoreModuleV2` | ASP.NET Core |
| `WebDAVModule` | WebDAV (خاموش کن) |

**۶) `appcmd` — مدیریت خط فرمان**

```cmd
%windir%\system32\inetsrv\appcmd.exe list sites
%windir%\system32\inetsrv\appcmd.exe list apppool
%windir%\system32\inetsrv\appcmd.exe list wp
```

**۷) PowerShell و `WebAdministration`**

```powershell
Import-Module WebAdministration
Get-Website
Get-WebApplication
Get-IISAppPool
```

**۸) ASP.NET Core و `ANCM`**

`AspNetCoreModuleV2` یک reverse proxy سبک است: IIS درخواست را به `Kestrel` (روی پورت داخلی یا named pipe) می‌فرستد.

```
کلاینت → HTTP.sys → IIS (ANCM) → Kestrel → اپ
```

در تولید، IIS جلوی Kestrel می‌نشیند (TLS، لاگ، rate limit، static).

**۹) `Recycling`**

`App Pool` بعد از مدت/تعداد درخواست، ری‌استارت می‌شود. دلایل: نشت حافظه، بازنویسی فایل.

| تنظیم | پیش‌فرض |
|---|---|
| `Regular Time Interval` | ۱۷۴۰ دقیقه |
| `Idle Time-out` | ۲۰ دقیقه |
| `Private Memory Limit` | ۰ (نامحدود) |

**۱۰) `HTTP.sys` و `netsh http`**

```cmd
netsh http show servicestate
netsh http show urlacl
netsh http add urlacl url=http://+:8080/ user="Everyone"
```

### مثال‌های عملی

**Lab Scenario: سایت کامل با PowerShell**

```powershell
# ۱) نصب IIS
Install-WindowsFeature -Name Web-Server -IncludeManagementTools

# ۲) نصب ماژول‌های لازم
Install-WindowsFeature -Name Web-Http-Redirect, Web-WebSockets, Web-Mgmt-Console, Web-IP-Security

# ۳) ساخت پوشه
New-Item -ItemType Directory -Path C:\inetpub\lab -Force | Out-Null
Set-Content -Path C:\inetpub\lab\index.html -Value '<h1>Lab OK</h1>'

# ۴) ساخت App Pool
New-WebAppPool -Name "lab-pool"
Set-ItemProperty -Path "IIS:\AppPools\lab-pool" -Name managedRuntimeVersion -Value ""

# ۵) ساخت Site
New-Website -Name "lab" -PhysicalPath C:\inetpub\lab `
  -ApplicationPool "lab-pool" -Port 80 -HostHeader "lab.example.com"

# ۶) بررسی
Get-Website -Name "lab" | Format-List Name, State, PhysicalPath, Bindings
Invoke-WebRequest -Uri http://lab.example.com/ -UseBasicParsing | Select-Object -ExpandProperty Content
```

**افزودن HTTPS با SNI:**

```powershell
$cert = Get-ChildItem Cert:\LocalMachine\My | Where-Object { $_.Subject -like "*lab.example.com*" }

New-WebBinding -Name "lab" -Protocol https -Port 443 `
  -HostHeader "lab.example.com" -SslFlags 1     # 1 = SNI

(Get-WebBinding -Name "lab" -Protocol https -HostHeader "lab.example.com").AddSslCertificate($cert.Thumbprint, "my")
```

**`web.config` کامل:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    <defaultDocument enabled="true">
      <files>
        <clear />
        <add value="index.html" />
        <add value="default.aspx" />
      </files>
    </defaultDocument>

    <directoryBrowse enabled="false" />

    <httpProtocol>
      <customHeaders>
        <remove name="X-Powered-By" />
        <add name="X-Content-Type-Options" value="nosniff" />
        <add name="X-Frame-Options" value="SAMEORIGIN" />
        <add name="Referrer-Policy" value="strict-origin-when-cross-origin" />
        <add name="Strict-Transport-Security" value="max-age=31536000; includeSubDomains" />
      </customHeaders>
    </httpProtocol>

    <httpErrors errorMode="DetailedLocalOnly" existingResponse="Replace">
      <remove statusCode="404" />
      <error statusCode="404" path="/404.html" responseMode="File" />
    </httpErrors>

    <security>
      <requestFiltering>
        <requestLimits maxAllowedContentLength="16777216" />
        <hiddenSegments>
          <add segment=".git" />
          <add segment="appsettings.json" />
        </hiddenSegments>
        <fileExtensions>
          <add fileExtension=".config" allowed="false" />
        </fileExtensions>
      </requestFiltering>
    </security>

    <staticContent>
      <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="7.00:00:00" />
      <remove fileExtension=".woff2" />
      <mimeMap fileExtension=".woff2" mimeType="font/woff2" />
    </staticContent>

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
</configuration>
```

**ASP.NET Core روی IIS:**

```powershell
# ۱) نصب Hosting Bundle (شامل ANCM)
# دانلود از microsoft.com/dotnet → ASP.NET Core Hosting Bundle

# ۲) publish
dotnet publish -c Release -o C:\inetpub\lab-api

# ۳) App Pool بدون managed runtime
New-WebAppPool -Name "lab-api-pool"
Set-ItemProperty "IIS:\AppPools\lab-api-pool" managedRuntimeVersion ""

# ۴) Site
New-Website -Name "lab-api" -PhysicalPath C:\inetpub\lab-api `
  -ApplicationPool "lab-api-pool" -Port 8080 -HostHeader "api.lab.example.com"

# ۵) بررسی
Invoke-WebRequest http://api.lab.example.com/health -UseBasicParsing
```

**مانیتورینگ با `appcmd`:**

```cmd
appcmd list wp
appcmd list requests
appcmd list apppool /state:Started
```

**لاگ IIS:**

```powershell
Get-WebConfigurationProperty -PSPath "IIS:\Sites\lab" -Filter "system.webServer/httpLogging" -Name *
# directory: %SystemDrive%\inetpub\logs\LogFiles
```

### مقایسه و الگوهای مختلف

**IIS در برابر Nginx در برابر Apache**

| معیار | IIS | Nginx | Apache |
|---|---|---|---|
| سیستم‌عامل | ویندوز | لینوکس/ویندوز | لینوکس/ویندوز |
| HTTP در کرنل | ✅ (`HTTP.sys`) | ❌ | ❌ |
| فایل استاتیک | ✅ (کش کرنل) | ✅✅ | ✅ |
| `.htaccess` | ⚠️ (`web.config`) | ❌ | ✅ |
| ASP.NET | ✅✅ | ⚠️ | ⚠️ |
| PHP | ⚠️ | ✅ (fpm) | ✅ |
| هزینه | 💰 (لایسنس ویندوز) | ۰ | ۰ |
| مدیریت GUI | ✅✅ | ⚠️ | ⚠️ |

**`App Pool` در برابر `worker` Nginx**

| | App Pool | worker |
|---|---|---|
| جداسازی | ✅ (per site) | ❌ (مشترک) |
| بازیافت خودکار | ✅ | ⚠️ (reload) |
| محدودیت حافظه | ✅ | ❌ |
| هویت جدا | ✅ | ❌ |

**`web.config` در برابر `.htaccess`**

| | `web.config` | `.htaccess` |
|---|---|---|
| فرمت | XML | متن Apache |
| سرعت | ⚠️ (cache دارد) | ❌ |
| قدرت | ✅ (بیشتر) | ✅ |
| قابل انتقال | ❌ | ⚠️ |

**دیدگاه «IIS فقط برای ASP.NET»**: IIS برای PHP و استاتیک هم کار می‌کند، ولی اکوسیستم و ابزارش حول .NET است. اگر روی ویندوز هستی و .NET داری، IIS انتخاب طبیعی است.

### اشکال شایع

**۱) `500.19` — config error**
`web.config` نامعتبر یا ماژول نصب‌نشده. `%windir%\system32\inetsrv\config` را چک کن.

**۲) `500.30` — ASP.NET Core start failure**
اپ بالا نمی‌آید. `stdout` log را در `logs/` ببین، یا `ASPNETCORE_ENVIRONMENT=Development`.

**۳) `502.5` — process failure**
`ANCM` نمی‌تواند `dotnet` را اجرا کند. Hosting Bundle نصب نیست.

**۴) `403.14` — Directory listing denied**
`defaultDocument` تنظیم نشده یا فایل `index` نیست.

**۵) `404.0` — not found**
مسیر فیزیکی یا `handler` اشتباه.

**۶) `App Pool` متوقف**
`Idle Time-out` یا خطای اپ. `appcmd list apppool`.

**۷) `Directory Browsing` روشن**
افشای ساختار. خاموش کن.

**۸) `WebDAV` روشن**
روش‌های `PUT`/`DELETE` فعال. حذف ماژول.

**۹) `X-Powered-By` و `Server`**
نسخه لو می‌رود. `customHeaders` + `removeServerHeader`.

**۱۰) `HTTP.sys` و پورت اشغال**
سرویس دیگری (`SQL Server Reporting`، `BranchCache`) روی ۸۰. `netstat -ano | findstr :80`.

### نکات مهم (Tips & Tricks)

- **`appcmd list site`** سریع‌ترین راه.
- **`Get-Website`/`Get-IISAppPool`** در PowerShell.
- **`netsh http show servicestate`** برای `HTTP.sys`.
- **`%SystemDrive%\inetpub\logs\LogFiles`** محل لاگ.
- **`failed request tracing`** برای دیباگ عمیق.
- **`ANCM`** نیاز به Hosting Bundle دارد.
- **`removeServerHeader="true"`** در `requestFiltering`.

### خلاصه فصل

- IIS از `HTTP.sys` در کرنل استفاده می‌کند.
- `App Pool` جداسازی و بازیافت می‌دهد.
- `Binding` = پروتکل + IP + پورت + hostname.
- `web.config` معادل `.htaccess` است.
- ASP.NET Core با `ANCM` پشت IIS می‌نشیند.
- `appcmd` و PowerShell ابزارهای مدیریت.

**سؤال مروری:** چرا دو سایت در دو `App Pool` جدا، امن‌تر و پایدارتر از یک pool هستند؟

---

## فصل ۱۲: مقایسه، انتخاب و گزینه‌های دیگر

### مقدمه

سه فصل قبل، سه معماری را دیدی. این فصل، تصمیم می‌گیرد: برای پروژه‌ی تو، کدام؟ و چهار گزینه‌ی دیگر که ارزش شناختن دارند: Caddy، LiteSpeed، Traefik و Envoy.

### مفاهیم اساسی

**۱) جدول تصمیم**

| نیاز | انتخاب اول |
|---|---|
| فایل استاتیک + reverse proxy | Nginx |
| وردپرس با `.htaccess` | Apache |
| ASP.NET / ویندوز | IIS |
| TLS خودکار و سادگی | Caddy |
| Kubernetes ingress | Traefik / Nginx Ingress |
| Service mesh | Envoy |
| هاست اشتراکی | LiteSpeed / Apache |

**۲) Nginx — چکیده**

- ✅ استاتیک، پروکسی، حافظه کم، HTTP/2/3
- ❌ `.htaccess` ندارد، `location` پیچیده، ماژول کامپایل

**۳) Apache — چکیده**

- ✅ انعطاف، `.htaccess`، ماژول پویا، مستندات غنی
- ❌ حافظه بیشتر، config پراکنده، `prefork` legacy

**۴) IIS — چکیده**

- ✅ ویندوز/`.NET`، GUI، `HTTP.sys`، App Pool
- ❌ لایسنس، فقط ویندوز، config XML پیچیده

**۵) Caddy — سادگی مدرن**

```caddyfile
lab.example.com {
    root * /var/www/lab
    file_server
    reverse_proxy /api/* localhost:3000
    encode gzip zstd
}
```

TLS خودکار (Let's Encrypt + ZeroSSL)، HTTP/3 پیش‌فرض، config کوتاه.

**۶) LiteSpeed**

سازگار با `.htaccess` و `mod_rewrite`، کارایی بالا، LSCache. در هاست اشتراکی رایج.

**۷) Traefik**

برای Docker/Kubernetes: کشف خودکار سرویس از label، TLS خودکار.

```yaml
labels:
  - "traefik.http.routers.lab.rule=Host(`lab.example.com`)"
  - "traefik.http.services.lab.loadbalancer.server.port=80"
```

**۸) Envoy**

Proxy L7 با focus روی observability و service mesh. پیچیده‌تر، برای زیرساخت بزرگ.

**۹) معیارهای انتخاب**

| معیار | وزن |
|---|---|
| کارایی | ⚠️ (اغلب گلوگاه نیست) |
| تیم و مهارت | ✅✅ |
| اکوسیستم اپ | ✅✅ |
| عملیات (لاگ، مانیتورینگ) | ✅ |
| هزینه | ✅ |
| قابلیت انتقال | ⚠️ |

**۱۰) معماری ترکیبی رایج**

```
اینترنت
  │
  ▼
CDN (Cloudflare)
  │
  ▼
Nginx (TLS، rate limit، static، proxy)
  │
  ├── PHP-FPM
  └── Node/Python app
```

یا در ویندوز:

```
اینترنت → IIS (ARR) → Kestrel / ASP.NET
```

### مثال‌های عملی

**Lab Scenario: همان سایت، چهار سرور**

هدف: یک سایت استاتیک + یک API، روی چهار سرور، با config مقایسه‌ای.

**Nginx:**

```nginx
server {
    listen 443 ssl;
    http2 on;
    server_name lab.example.com;

    ssl_certificate     /etc/ssl/lab/fullchain.pem;
    ssl_certificate_key /etc/ssl/lab/privkey.pem;

    root /var/www/lab;

    location / {
        try_files $uri $uri/ =404;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:3000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    gzip on;
    gzip_types text/css application/javascript application/json;
}
```

**Apache:**

```apache
<VirtualHost *:443>
    ServerName lab.example.com
    SSLEngine on
    SSLCertificateFile    /etc/ssl/lab/fullchain.pem
    SSLCertificateKeyFile /etc/ssl/lab/privkey.pem

    DocumentRoot /var/www/lab

    <Directory /var/www/lab>
        AllowOverride None
        Require all granted
        FallbackResource /index.html
    </Directory>

    ProxyPass        /api/ http://127.0.0.1:3000/
    ProxyPassReverse /api/ http://127.0.0.1:3000/

    <IfModule mod_deflate.c>
        AddOutputFilterByType DEFLATE text/css application/javascript application/json
    </IfModule>
</VirtualHost>
```

**IIS (`web.config`):**

```xml
<configuration>
  <system.webServer>
    <staticContent>
      <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="7.00:00:00" />
    </staticContent>
    <rewrite>
      <rules>
        <rule name="API" stopProcessing="true">
          <match url="^api/(.*)" />
          <action type="Rewrite" url="http://127.0.0.1:3000/{R:1}" />
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
  </httpProtocol>
  </system.webServer>
</configuration>
```

**Caddy (`Caddyfile`):**

```caddyfile
lab.example.com {
    root * /var/www/lab
    encode gzip zstd

    handle /api/* {
        reverse_proxy localhost:3000
    }

    handle {
        try_files {path} {path}/ /index.html
        file_server
    }
}
```

```bash
caddy validate --config Caddyfile
sudo systemctl reload caddy
```

**بنچمارک مقایسه‌ای:**

```bash
# فایل استاتیک ۱۰۰ کیلوبایتی
hey -n 50000 -c 200 https://lab.example.com/static/test.bin

# API با تأخیر ۱۰ms
hey -n 10000 -c 100 https://lab.example.com/api/fast

# TLS handshake
openssl s_time -connect lab.example.com:443 -www /index.html -time 10
```

### مقایسه و الگوهای مختلف

**پنج گزینه، پنج جایگاه**

| | Nginx | Apache | IIS | Caddy | Traefik |
|---|---|---|---|---|---|
| استاتیک | ✅✅ | ✅ | ✅ | ✅ | ⚠️ |
| پروکسی | ✅✅ | ✅ | ✅ | ✅ | ✅✅ |
| TLS خودکار | ⚠️ (certbot) | ⚠️ | ⚠️ | ✅✅ | ✅✅ |
| `.htaccess` | ❌ | ✅ | ⚠️ | ❌ | ❌ |
| K8s-native | ⚠️ | ❌ | ❌ | ⚠️ | ✅✅ |
| config ساده | ⚠️ | ⚠️ | ❌ | ✅✅ | ✅ |
| HTTP/3 | ✅ (1.25+) | ⚠️ | ⚠️ | ✅ | ✅ |
| بالغ/پایدار | ✅✅ | ✅✅ | ✅✅ | ✅ | ✅ |

**سه استراتژی مهاجرت**

1. **Apache → Nginx**: قواعد `mod_rewrite` را به `return`/`rewrite` ترجمه کن؛ `.htaccess` را به vhost ببر.
2. **Nginx → Caddy**: config کوتاه‌تر، TLS خودکار؛ ولی `location` پیچیده نیاز به `handle` دارد.
3. **هر → Traefik**: در K8s، با `Ingress`/`IngressRoute`.

**دیدگاه «یک سرور برای همه‌چیز»**: در عمل، ترکیب رایج است: Nginx برای لبه، IIS برای .NET، Apache برای legacy. مهم، مستندسازی و مالکیت مشخص است.

### اشکال شایع

**۱) انتخاب بر اساس بنچمارک خام**
بنچمارک `hello world` ربطی به بار واقعی ندارد. با ترافیک و الگوی خودت تست کن.

**۲) مهاجرت بدون تست `mod_rewrite`**
قواعد پیچیده، بی‌صدا خراب می‌شوند. تست با URLهای واقعی از لاگ.

**۳) Caddy در محیطی که Let's Encrypt بلاک است**
TLS خودکار شکست می‌خورد. `tls internal` یا CA دیگر.

**۴) Traefik بدون درک labelها**
سرویس‌ها کشف نمی‌شوند. `traefik.http.routers...` را دقیق بنویس.

**۵) IIS بدون Hosting Bundle برای .NET**
`502.5`. نصب کن.

**۶) Nginx به‌عنوان «فقط پروکسی» و فراموش کردن TLS**
`proxy_pass` به `http://` در شبکه‌ی باز.

**۷) نبود مانیتورینگ در انتخاب**
سروری که نمی‌توانی مانیتور کنی، انتخاب بدی است.

**۸) نادیده گرفتن مهارت تیم**
Nginx عالی است، ولی اگر تیم فقط Apache می‌داند، هزینه‌ی آموزش واقعی است.

### نکات مهم (Tips & Tricks)

- **با ترافیک واقعی تست کن**، نه `hello world`.
- **`nginx -T`/`apachectl -S`** برای دیدن config مؤثر.
- **Caddy** برای پروژه‌های کوچک با TLS سریع.
- **Traefik** برای K8s.
- **مستندسازی config** در git.
- **یک لبه، یک مسئولیت**: TLS و rate limit در لبه.
- **مهاجرت مرحله‌ای**: اول پروکسی، بعد استاتیک.

### خلاصه فصل

- Nginx: استاتیک/پروکسی؛ Apache: انعطاف/`.htaccess`؛ IIS: ویندوز/.NET.
- Caddy: سادگی و TLS خودکار؛ Traefik: K8s؛ Envoy: service mesh.
- انتخاب بر اساس تیم، اکوسیستم و عملیات، نه بنچمارک خام.
- معماری ترکیبی رایج است.
- مهاجرت نیاز به تست URLهای واقعی دارد.

**سؤال مروری:** برای یک تیم کوچک با وردپرس و نیاز به TLS خودکار، کدام ترکیب را پیشنهاد می‌کنی و چرا؟

---

## منابع و مراجع

**مستندات رسمی**
- [Apache HTTP Server 2.4 Docs](https://httpd.apache.org/docs/2.4/)
- [Apache MPMs](https://httpd.apache.org/docs/2.4/mpm.html)
- [Nginx Docs](https://nginx.org/en/docs/)
- [Nginx — Connection processing methods](https://nginx.org/en/docs/events.html)
- [IIS Documentation](https://learn.microsoft.com/en-us/iis/)
- [Caddy Docs](https://caddyserver.com/docs/)
- [Traefik Docs](https://doc.traefik.io/traefik/)
- [Envoy Docs](https://www.envoyproxy.io/docs)

**مقالات**
- [The C10K problem](http://www.kegel.com/c10k.html) — مرجع تاریخی
- [Nginx vs Apache performance](https://www.nginx.com/blog/nginx-vs-apache-our-view/)
- [IIS architecture](https://learn.microsoft.com/en-us/iis/get-started/introduction-to-iis/introduction-to-iis-architecture)

**ابزار**
- [hey](https://github.com/rakyll/hey)
- [wrk](https://github.com/wg/wrk)
- [ab](https://httpd.apache.org/docs/2.4/programs/ab.html)

---

**بعدی:** [`04-Apache-Install-and-Config.md`](04-Apache-Install-and-Config.md) — نصب، `VirtualHost` و `.htaccess`
