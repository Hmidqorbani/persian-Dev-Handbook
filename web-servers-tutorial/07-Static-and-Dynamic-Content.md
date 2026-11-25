# بخش ۰۷: سرو محتوای استاتیک و داینامیک

**مدت یادگیری**: ۴ تا ۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۰۵ (Nginx، `location`)، بخش ۰۴ (Apache)

---

## فهرست محتوا

- [فصل ۲۷: سرو فایل استاتیک — مسیر، MIME، `index` و `autoindex`](#فصل-۲۷)
- [فصل ۲۸: PHP-FPM با Nginx و Apache](#فصل-۲۸)
- [فصل ۲۹: Python و Node.js — WSGI/ASGI و reverse proxy](#فصل-۲۹)
- [فصل ۳۰: خطاهای ۴۰۳، ۴۰۴ و ۵۰۰ — دیباگ مرحله‌به‌مرحله](#فصل-۳۰)

---

## فصل ۲۷: سرو فایل استاتیک — مسیر، MIME، `index` و `autoindex`

### مقدمه

سرو فایل استاتیک، ساده‌ترین کار وب‌سرور و در عین حال، پرتکرارترین منبع خطاهای `403` و `404`. سه چیز باید درست باشد: **مسیر** (کدام فایل)، **مجوز** (می‌توانم بخوانم؟)، **MIME** (با چه نوعی بفرستم؟).

### مفاهیم اساسی

**۱) زنجیره‌ی حل مسیر**

```
URI: /assets/css/app.css
  │
  ├── location تطبیق → /assets/
  ├── root/alias → /var/www/lab/static/css/app.css
  ├── مجوز فایل (www-data:r)
  ├── symlink (اگر FollowSymLinks)
  └── MIME → text/css
```

هر مرحله می‌تواند شکست بخورد و کد خطای متفاوتی بدهد.

**۲) MIME type**

| پسوند | MIME |
|---|---|
| `.html` | `text/html; charset=utf-8` |
| `.css` | `text/css` |
| `.js` | `text/javascript` (یا `application/javascript`) |
| `.json` | `application/json` |
| `.svg` | `image/svg+xml` |
| `.woff2` | `font/woff2` |
| `.webp` | `image/webp` |
| `.wasm` | `application/wasm` |

MIME اشتباه → مرورگر فایل را اجرا/نمایش نمی‌دهد. مثال کلاسیک: `.js` با `text/plain` → `SyntaxError` یا بلاک.

**۳) `index`**

```nginx
index index.html index.htm;
```

اگر URI به پوشه اشاره کند و `index` وجود نداشته باشد:
- `autoindex off` → `403`
- `autoindex on` → لیست فایل

**۴) `autoindex`**

```nginx
location /files/ {
    autoindex on;
    autoindex_exact_size off;
    autoindex_localtime on;
}
```

> در تولید، فقط پشت احراز هویت یا IP محدود.

**۵) `charset`**

```nginx
charset utf-8;
charset_types text/html text/css application/javascript application/json;
```

بدون `charset`، مرورگر ممکن است صفحه‌ی فارسی را با encoding اشتباه نشان دهد.

**۶) `sendfile` و `aio`**

```nginx
sendfile on;
sendfile_max_chunk 1m;     # جلوگیری از اشغال worker با فایل بزرگ
aio threads;               # برای فایل بزرگ روی فایل‌سیستم کند
directio 4m;               # دور زدن کش OS برای فایل بزرگ
```

**۷) `Range` و فایل بزرگ**

Nginx به‌طور پیش‌فرض `Range` را پشتیبانی می‌کند. برای ویدیو و PDF ضروری است. اگر غیرفعال شد:

```nginx
max_ranges 1;     # محدود کردن
# یا حذف کامل با proxy
```

**۸) `expires` و `Cache-Control`**

```nginx
location ~* \.(css|js|jpg|jpeg|png|gif|ico|svg|woff2)$ {
    expires 30d;
    add_header Cache-Control "public, immutable";
    access_log off;
}
```

**۹) فشرده‌سازی فایل استاتیک**

```nginx
gzip_static on;     # فایل .gz از قبل ساخته‌شده
brotli_static on;   # ماژول brotli
```

`gzip_static` فایل `.gz` را مستقیم می‌فرستد (بدون فشرده‌سازی در هر درخواست).

**۱۰) Apache: `mod_mime` و `AddType`**

```apache
AddType font/woff2 .woff2
AddType image/webp .webp
AddType application/wasm .wasm
AddDefaultCharset UTF-8
```

**۱۱) IIS: `staticContent` و `mimeMap`**

```xml
<staticContent>
  <remove fileExtension=".woff2" />
  <mimeMap fileExtension=".woff2" mimeType="font/woff2" />
</staticContent>
```

### مثال‌های عملی

**Lab Scenario: سایت استاتیک کامل**

ساختار:

```
/var/www/lab/
├── index.html
├── assets/
│   ├── app.css
│   ├── app.js
│   └── logo.svg
├── media/
│   └── demo.mp4
└── files/           (autoindex محدود)
    ├── a.pdf
    └── b.zip
```

```bash
sudo mkdir -p /var/www/lab/{assets,media,files}
echo '<!doctype html><html><body><h1>Lab</h1><link rel=stylesheet href=/assets/app.css></body></html>' \
  | sudo tee /var/www/lab/index.html
echo 'body{font-family:sans-serif}' | sudo tee /var/www/lab/assets/app.css
echo 'console.log("ok")' | sudo tee /var/www/lab/assets/app.js
sudo chown -R www-data:www-data /var/www/lab
sudo find /var/www/lab -type d -exec chmod 755 {} \;
sudo find /var/www/lab -type f -exec chmod 644 {} \;
```

```nginx
server {
    listen 8080;
    server_name _;
    root /var/www/lab;
    index index.html;

    charset utf-8;

    # فایل‌های استاتیک با کش
    location /assets/ {
        expires 30d;
        add_header Cache-Control "public, immutable";
        access_log off;
        try_files $uri =404;
    }

    # رسانه با Range
    location /media/ {
        expires 7d;
        add_header Accept-Ranges bytes;
        try_files $uri =404;
    }

    # لیست فایل محدود
    location /files/ {
        autoindex on;
        autoindex_exact_size off;
        allow 192.168.56.0/24;
        deny all;
    }

    location / {
        try_files $uri $uri/ =404;
    }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx

# تست MIME
curl -sSI http://localhost:8080/assets/app.css | grep -i content-type
# Content-Type: text/css

curl -sSI http://localhost:8080/assets/app.js | grep -i content-type
# Content-Type: application/javascript

# تست Range
curl -sS -r 0-9 -D - -o /dev/null http://localhost:8080/media/demo.mp4 | grep -Ei 'content-range|content-length|HTTP/'
# HTTP/1.1 206 Partial Content
# Content-Range: bytes 0-9/...

# تست autoindex
curl -s http://localhost:8080/files/ | head -5
# <html><head><title>Index of /files/</title>...
```

**بررسی MIME اشتباه:**

```bash
# اگر .js به‌عنوان text/plain سرو شود
curl -sSI http://localhost:8080/assets/app.js | grep -i content-type
# Content-Type: text/plain    ← مشکل

# در Nginx: include mime.types در http
# در Apache: AddType application/javascript .js
# در IIS: mimeMap
```

**فشرده‌سازی استاتیک از قبل:**

```bash
# ساخت فایل .gz برای هر فایل استاتیک
cd /var/www/lab/assets
for f in *.css *.js *.svg; do gzip -9 -k -f "$f"; done
ls -la
# app.css  app.css.gz

# در Nginx
# gzip_static on;
```

```bash
curl -sSI -H 'Accept-Encoding: gzip' http://localhost:8080/assets/app.css | grep -Ei 'content-encoding|content-length'
# Content-Encoding: gzip
```

**Apache معادل:**

```apache
<VirtualHost *:80>
    ServerName lab.example.com
    DocumentRoot /var/www/lab

    AddDefaultCharset UTF-8
    AddType font/woff2 .woff2
    AddType image/webp .webp

    <Directory /var/www/lab>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    <Directory /var/www/lab/assets>
        ExpiresActive On
        ExpiresDefault "access plus 30 days"
        Header set Cache-Control "public, immutable"
    </Directory>

    <Directory /var/www/lab/files>
        Options +Indexes
        IndexOptions FancyIndexing
        Require ip 192.168.56.0/24
    </Directory>
</VirtualHost>
```

**IIS معادل:**

```xml
<system.webServer>
  <staticContent>
    <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="30.00:00:00" />
    <remove fileExtension=".woff2" />
    <mimeMap fileExtension=".woff2" mimeType="font/woff2" />
    <remove fileExtension=".wasm" />
    <mimeMap fileExtension=".wasm" mimeType="application/wasm" />
  </staticContent>
</system.webServer>
```

### مقایسه و الگوهای مختلف

**`root` در برابر `alias` برای استاتیک**

| | `root` | `alias` |
|---|---|---|
| ساختار | یکسان با URL | متفاوت |
| `/` پایانی | بی‌اهمیت | ⚠️ حیاتی |
| توصیه | ✅ پیش‌فرض | پوشه‌ی خارجی |

**`autoindex` روشن در برابر خاموش**

| | `on` | `off` |
|---|---|---|
| امنیت | ❌ | ✅ |
| راحتی | ✅ | ❌ |
| توصیه | فقط محدود | ✅ |

**`gzip_static` در برابر `gzip`**

| | `gzip_static` | `gzip` |
|---|---|---|
| CPU | ✅ صفر | ⚠️ هر درخواست |
| نیاز به فایل `.gz` | ✅ | ❌ |
| توصیه | ✅ استاتیک | داینامیک |

**سه روش کش استاتیک**

| روش | هدر |
|---|---|
| `expires 30d` | `Expires` + `Cache-Control: max-age` |
| `add_header Cache-Control` | `Cache-Control` |
| `immutable` | `Cache-Control: public, max-age=31536000, immutable` |

**دیدگاه «`autoindex` برای دیباگ خوب است»**: در توسعه بله. در تولید، ساختار را لو می‌دهد.

### اشکال شایع

**۱) `403 Forbidden` برای فایل**
مجوز `644` و پوشه `755`. `namei -l /var/www/lab/assets/app.css`.

**۲) `404` با وجود فایل**
`root`/`alias` اشتباه. `nginx -T` و `error_log`.

**۳) MIME `text/plain` برای `.js`**
`include mime.types` نبودن. مرورگر بلاک می‌کند.

**۴) فایل فارسی خراب**
`charset` تنظیم نشده. `charset utf-8`.

**۵) `autoindex` روشن**
افشا. `-Indexes`/`autoindex off`.

**۶) `sendfile` با `alias` و symlink**
مشکل در بعضی نسخه‌ها. `disable_symlinks`.

**۷) فایل بزرگ و `worker` اشغال**
`sendfile_max_chunk 1m` و `aio threads`.

**۸) `Range` کار نمی‌کند**
`max_ranges 0` یا پروکسی که هدر را حذف می‌کند.

**۹) `Cache-Control` روی HTML**
کش صفحه‌ی داینامیک. `no-cache` برای HTML.

**۱۰) فایل `.gz` قدیمی**
`gzip_static` نسخه‌ی قدیمی می‌فرستد. در build جدید بساز.

### نکات مهم (Tips & Tricks)

- **`namei -l`** برای دیباگ مسیر و مجوز.
- **`curl -sSI | grep -i content-type`** برای MIME.
- **`gzip_static`** برای استاتیک.
- **`immutable`** برای فایل با hash در نام.
- **`sendfile_max_chunk`** برای فایل بزرگ.
- **`charset utf-8`** برای فارسی.
- **`autoindex` فقط محدود**.

### خلاصه فصل

- سه شرط: مسیر، مجوز، MIME.
- `403` = مجوز/`autoindex`، `404` = مسیر.
- MIME اشتباه، فایل را غیرقابل استفاده می‌کند.
- `Range` برای ویدیو و PDF.
- `gzip_static` فشرده‌سازی را از CPU خارج می‌کند.
- `charset utf-8` برای فارسی الزامی است.

**سؤال مروری:** فایل `app.js` با MIME `text/plain` سرو می‌شود. مرورگر چه می‌کند و چطور تشخیص می‌دهی؟

---

## فصل ۲۸: PHP-FPM با Nginx و Apache

### مقدمه

PHP روی وب‌سرور، دو راه دارد: `mod_php` (داخل Apache) و `php-fpm` (پروسه‌ی جدا با FastCGI). دومی استاندارد امروز است: جداسازی، opcache مشترک، و سازگاری با `event` MPM و Nginx.

### مفاهیم اساسی

**۱) FastCGI چیست**

```
Nginx/Apache ──FastCGI──▶ PHP-FPM (master)
                              ├── pool: www
                              │    ├── worker 1
                              │    └── worker 2
                              └── pool: api
                                   └── worker
```

FastCGI یک پروتکل باینری است که درخواست را با پارامترها (`SCRIPT_FILENAME`، `QUERY_STRING`، ...) به PHP می‌فرستد.

**۲) `php-fpm` pool**

```ini
; /etc/php/8.3/fpm/pool.d/www.conf
[www]
user = www-data
group = www-data
listen = /run/php/php8.3-fpm.sock
listen.owner = www-data
listen.group = www-data
listen.mode = 0660

pm = dynamic
pm.max_children = 25
pm.start_servers = 5
pm.min_spare_servers = 3
pm.max_spare_servers = 10
pm.max_requests = 500
```

| `pm` | رفتار |
|---|---|
| `static` | تعداد ثابت |
| `dynamic` | بر اساس بار |
| `ondemand` | فقط در صورت نیاز |

**۳) `pm.max_children` — محاسبه**

```
max_children ≈ (RAM اختصاصی به PHP) / (میانگین حافظه‌ی هر worker)
```

مثال: ۲ گیگ برای PHP، هر worker ۵۰ مگ → ۴۰ worker.

**۴) Nginx + PHP-FPM**

```nginx
location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/run/php/php8.3-fpm.sock;
}
```

`fastcgi-php.conf` شامل:

```nginx
fastcgi_split_path_info ^(.+?\.php)(/.*)$;
set $path_info $fastcgi_path_info;
try_files $fastcgi_script_name =404;
include fastcgi_params;
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
fastcgi_param PATH_INFO $path_info;
fastcgi_index index.php;
```

**۵) تله‌ی `SCRIPT_FILENAME`**

اگر `SCRIPT_FILENAME` اشتباه باشد: `Primary script unknown` یا `File not found`. باید `$document_root$fastcgi_script_name` باشد.

**۶) Apache + PHP-FPM**

```apache
<VirtualHost *:80>
    ServerName lab.example.com
    DocumentRoot /var/www/lab

    <Directory /var/www/lab>
        AllowOverride None
        Require all granted
        <FilesMatch \.php$>
            SetHandler "proxy:unix:/run/php/php8.3-fpm.sock|fcgi://localhost/"
        </FilesMatch>
    </Directory>
</VirtualHost>
```

```bash
sudo a2enmod proxy proxy_fcgi
sudo a2dismod php8.3 mpm_prefork 2>/dev/null
sudo a2enmod mpm_event
sudo apachectl configtest && sudo systemctl restart apache2 php8.3-fpm
```

**۷) `php.ini` و `opcache`**

```ini
; /etc/php/8.3/fpm/php.ini
memory_limit = 256M
upload_max_filesize = 16M
post_max_size = 16M
max_execution_time = 30
display_errors = Off
log_errors = On
error_log = /var/log/php/error.log

opcache.enable = 1
opcache.memory_consumption = 128
opcache.max_accelerated_files = 10000
opcache.validate_timestamps = 0    ; در تولید، ۰ برای سرعت
```

**۸) `.user.ini` — تنظیم per-directory**

```ini
; /var/www/lab/.user.ini
upload_max_filesize = 32M
post_max_size = 32M
```

**۹) `php_value` در `.htaccess`**

فقط با `mod_php`. با `php-fpm` کار نمی‌کند (→ فصل ۱۷).

**۱۰) لاگ PHP-FPM**

```ini
; pool.d/www.conf
access.log = /var/log/php/fpm-access.log
slowlog = /var/log/php/fpm-slow.log
request_slowlog_timeout = 5s
catch_workers_output = yes
```

### مثال‌های عملی

**Lab Scenario: Nginx + PHP-FPM کامل**

```bash
# ۱) نصب
sudo apt install -y php8.3-fpm php8.3-cli php8.3-mbstring php8.3-xml php8.3-curl php8.3-mysql

# ۲) بررسی
systemctl status php8.3-fpm --no-pager
ls -l /run/php/php8.3-fpm.sock

# ۳) تست
sudo -u www-data php -v
```

```bash
# ۴) فایل تست
sudo tee /var/www/lab/info.php >/dev/null <<'PHP'
<?php
header('Content-Type: text/plain; charset=utf-8');
echo "PHP ", PHP_VERSION, "\n";
echo "SAPI: ", php_sapi_name(), "\n";
echo "IP: ", $_SERVER['REMOTE_ADDR'] ?? '-', "\n";
PHP
sudo chown www-data:www-data /var/www/lab/info.php
```

```nginx
server {
    listen 8080;
    server_name _;
    root /var/www/lab;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;

        fastcgi_param HTTPS $https if_not_empty;
        fastcgi_param HTTP_X_FORWARDED_PROTO $scheme;
        fastcgi_read_timeout 60s;
        fastcgi_buffers 16 16k;
        fastcgi_buffer_size 32k;
    }

    location ~ /\.(?!well-known) {
        deny all;
    }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx
curl -sS http://localhost:8080/info.php
# PHP 8.3.x
# SAPI: fpm-fcgi
# IP: 127.0.0.1
```

**بهینه‌سازی pool:**

```ini
; /etc/php/8.3/fpm/pool.d/www.conf
[www]
user = www-data
group = www-data
listen = /run/php/php8.3-fpm.sock
listen.owner = www-data
listen.group = www-data
listen.mode = 0660

pm = dynamic
pm.max_children = 40
pm.start_servers = 8
pm.min_spare_servers = 4
pm.max_spare_servers = 16
pm.max_requests = 500

request_terminate_timeout = 60s
request_slowlog_timeout = 5s
slowlog = /var/log/php/fpm-slow.log

php_admin_value[memory_limit] = 256M
php_admin_value[upload_max_filesize] = 16M
php_admin_value[post_max_size] = 16M
php_admin_value[display_errors] = Off
php_admin_flag[log_errors] = On
```

```bash
sudo systemctl restart php8.3-fpm

# بررسی وضعیت
sudo systemctl status php8.3-fpm --no-pager
ps -eo pid,rss,cmd | grep 'php-fpm: pool' | awk '{sum+=$2} END {print "total RSS MB:", sum/1024}'
```

**Apache + PHP-FPM:**

```bash
sudo apt install -y libapache2-mod-fcgid
sudo a2enmod proxy proxy_fcgi setenvif
sudo a2dismod mpm_prefork
sudo a2enmod mpm_event
```

```apache
<VirtualHost *:80>
    ServerName lab.example.com
    DocumentRoot /var/www/lab

    <Directory /var/www/lab>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted

        <FilesMatch "\.php$">
            SetHandler "proxy:unix:/run/php/php8.3-fpm.sock|fcgi://localhost/"
        </FilesMatch>
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/lab-error.log
    CustomLog ${APACHE_LOG_DIR}/lab-access.log combined
</VirtualHost>
```

```bash
sudo apachectl configtest && sudo systemctl restart apache2
curl -sS -H 'Host: lab.example.com' http://127.0.0.1/info.php
```

**IIS + PHP (FastCGI):**

```powershell
# ۱) نصب PHP (مثلاً با winget یا دانلود zip)
# ۲) نصب CGI
Install-WindowsFeature Web-CGI

# ۳) ثبت handler
& $env:windir\system32\inetsrv\appcmd.exe set config /section:system.webServer/fastCgi `
  /+"[fullPath='C:\php\php-cgi.exe']"

& $env:windir\system32\inetsrv\appcmd.exe set config /section:handlers `
  /+"[name='PHP',path='*.php',verb='*',modules='FastCgiModule',scriptProcessor='C:\php\php-cgi.exe',resourceType='Either']"
```

**دیباگ خطاهای PHP:**

```bash
# ۱) خطای FPM
sudo tail -50 /var/log/php8.3-fpm.log

# ۲) خطای PHP
sudo tail -50 /var/log/php/error.log

# ۳) تست مستقیم socket
sudo cgi-fcgi -bind -connect /run/php/php8.3-fpm.sock

# ۴) بررسی handler
curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/info.php
```

### مقایسه و الگوهای مختلف

**`mod_php` در برابر `php-fpm`**

| | `mod_php` | `php-fpm` |
|---|---|---|
| MPM | `prefork` | `event` |
| حافظه | ❌ per-process | ✅ shared |
| opcache | ⚠️ per-process | ✅ shared |
| تنظیم per-pool | ❌ | ✅ |
| `.htaccess` `php_value` | ✅ | ❌ |
| توصیه | legacy | ✅ |

**`static` در برابر `dynamic` در برابر `ondemand`**

| | `static` | `dynamic` | `ondemand` |
|---|---|---|---|
| حافظه | ❌ ثابت | ⚠️ | ✅ کم |
| تأخیر | ✅ صفر | ✅ | ❌ cold start |
| مناسب | بار ثابت بالا | ✅ عمومی | بار کم |

**`unix socket` در برابر TCP**

| | socket | TCP `127.0.0.1:9000` |
|---|---|---|
| سرعت | ✅ | ⚠️ |
| جداسازی | ⚠️ (هم‌میزبان) | ✅ (شبکه) |
| توصیه | ✅ هم‌میزبان | چند سرور |

**دیدگاه «`pm.max_children` هرچه بیشتر بهتر»**: هر worker حافظه می‌گیرد. `OOM` و `swap` بدتر از صف است.

### اشکال شایع

**۱) `502 Bad Gateway`**
`php-fpm` خاموش یا socket اشتباه. `systemctl status php8.3-fpm`.

**۲) `Primary script unknown`**
`SCRIPT_FILENAME` اشتباه. `$document_root$fastcgi_script_name`.

**۳) `File not found`**
`root`/`alias` یا `try_files` اشتباه.

**۴) `413 Request Entity Too Large`**
`client_max_body_size` (Nginx) یا `post_max_size`/`upload_max_filesize` (PHP).

**۵) `504 Gateway Timeout`**
`fastcgi_read_timeout` کم یا اپ کند. `slowlog`.

**۶) `php_value` در `.htaccess` بی‌اثر**
`php-fpm` آن را نمی‌خواند. `.user.ini` یا pool config.

**۷) `opcache.validate_timestamps = 0` و تغییر کد**
کد جدید دیده نمی‌شود. `systemctl reload php8.3-fpm`.

**۸) `pm.max_children` کم**
`502` زیر بار. لاگ: `server reached pm.max_children`.

**۹) مجوز socket**
`502` و `Permission denied`. `listen.owner`/`listen.mode`.

**۱۰) `display_errors = On` در تولید**
افشای مسیر و کد. `Off` + `log_errors`.

### نکات مهم (Tips & Tricks)

- **`fastcgi-php.conf`** استاندارد.
- **`unix socket`** سریع‌تر.
- **`pm.max_children`** بر اساس رم.
- **`slowlog`** برای کوئری کند.
- **`opcache.validate_timestamps = 0`** در تولید + reload.
- **`display_errors = Off`**.
- **`client_max_body_size`** هماهنگ با PHP.

### خلاصه فصل

- `php-fpm` استاندارد؛ `mod_php` legacy.
- `SCRIPT_FILENAME` مهم‌ترین پارامتر FastCGI.
- `pm.max_children` بر اساس رم محاسبه می‌شود.
- `opcache` در تولید با `validate_timestamps = 0`.
- `.user.ini` جای `php_value`.
- `502` معمولاً FPM خاموش یا socket اشتباه است.

**سؤال مروری:** چرا `php_value upload_max_filesize` در `.htaccess` با `php-fpm` کار نمی‌کند و جایگزینش چیست؟

---

## فصل ۲۹: Python و Node.js — WSGI/ASGI و reverse proxy

### مقدمه

اپلیکیشن‌های Python و Node.js یک سرور HTTP داخلی دارند (`Gunicorn`، `Uvicorn`، `Node http`). این سرورها برای «منطق» ساخته شده‌اند، نه برای TLS، فایل استاتیک، rate limit و اتصال زیاد. راه درست: وب‌سرور جلوی آن‌ها.

### مفاهیم اساسی

**۱) WSGI و ASGI**

| | WSGI | ASGI |
|---|---|---|
| مدل | همزمان | async |
| فریم‌ورک | Django (سنتی)، Flask | FastAPI، Starlette، Django 4+ |
| سرور | Gunicorn، uWSGI | Uvicorn، Hypercorn |
| WebSocket | ❌ | ✅ |

**۲) معماری استاندارد**

```
کلاینت → Nginx (TLS، static، rate limit) → Gunicorn/Uvicorn (127.0.0.1:8000) → اپ
```

**۳) Gunicorn**

```bash
gunicorn app:app \
  --bind 127.0.0.1:8000 \
  --workers 4 \
  --threads 2 \
  --timeout 30 \
  --access-logfile - \
  --error-logfile -
```

`workers` معمولاً `2 × cores + 1` برای بار CPU-محور.

**۴) Uvicorn**

```bash
uvicorn app:app --host 127.0.0.1 --port 8000 --workers 4
```

یا `gunicorn -k uvicorn.workers.UvicornWorker`.

**۵) Node.js**

```javascript
// server.js
import express from 'express';
const app = express();
app.get('/health', (req, res) => res.send('ok'));
app.listen(3000, '127.0.0.1');
```

**۶) `PM2` برای Node**

```bash
pm2 start server.js -i max --name lab-api
pm2 save
pm2 startup
```

`-i max` = cluster mode با تعداد هسته.

**۷) Nginx برای Python/Node**

```nginx
location / {
    proxy_pass http://127.0.0.1:8000;
    proxy_http_version 1.1;
    proxy_set_header Host              $host;
    proxy_set_header X-Real-IP         $remote_addr;
    proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Host  $host;
    proxy_read_timeout 60s;
}
```

**۸) `proxy_pass` با/بدون `/`**

```nginx
location /api/ {
    proxy_pass http://127.0.0.1:8000;    # /api/users → /api/users
}
location /api/ {
    proxy_pass http://127.0.0.1:8000/;   # /api/users → /users
}
```

**۹) `X-Forwarded-*` و اپ**

Django:

```python
USE_X_FORWARDED_HOST = True
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
```

FastAPI:

```python
from uvicorn.middleware.proxy_headers import ProxyHeadersMiddleware
app.add_middleware(ProxyHeadersMiddleware, trusted_hosts="127.0.0.1")
```

Express:

```javascript
app.set('trust proxy', 'loopback');
```

**۱۰) `keepalive` به upstream**

```nginx
upstream app {
    server 127.0.0.1:8000;
    keepalive 32;
}

location / {
    proxy_pass http://app;
    proxy_http_version 1.1;
    proxy_set_header Connection "";
}
```

`Connection ""` مهم است؛ وگرنه `keepalive` بی‌اثر می‌شود.

**۱۱) systemd برای اپ**

```ini
# /etc/systemd/system/lab-api.service
[Unit]
Description=Lab API
After=network.target

[Service]
Type=simple
User=www-data
Group=www-data
WorkingDirectory=/srv/lab-api
Environment="DATABASE_URL=postgresql://..."
ExecStart=/srv/lab-api/venv/bin/gunicorn app:app \
    --bind 127.0.0.1:8000 --workers 4 --timeout 30 \
    --access-logfile - --error-logfile -
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

### مثال‌های عملی

**Lab Scenario: FastAPI + Nginx**

```bash
# ۱) اپ
sudo mkdir -p /srv/lab-api && cd /srv/lab-api
python3 -m venv venv
./venv/bin/pip install fastapi uvicorn gunicorn

cat > app.py <<'PY'
from fastapi import FastAPI, Request

app = FastAPI()

@app.get("/health")
def health():
    return {"status": "ok"}

@app.get("/whoami")
def whoami(request: Request):
    return {
        "client": request.client.host,
        "xff": request.headers.get("x-forwarded-for"),
        "proto": request.headers.get("x-forwarded-proto"),
    }
PY

sudo chown -R www-data:www-data /srv/lab-api
```

```ini
# /etc/systemd/system/lab-api.service
[Unit]
Description=Lab API
After=network.target

[Service]
User=www-data
WorkingDirectory=/srv/lab-api
ExecStart=/srv/lab-api/venv/bin/gunicorn app:app -k uvicorn.workers.UvicornWorker \
    --bind 127.0.0.1:8000 --workers 4 --timeout 30
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now lab-api
sudo systemctl status lab-api --no-pager
curl -sS http://127.0.0.1:8000/health
```

```nginx
# /etc/nginx/sites-available/api.conf
upstream lab_api {
    server 127.0.0.1:8000;
    keepalive 32;
}

server {
    listen 8080;
    server_name api.lab.example.com;

    location /api/ {
        proxy_pass http://lab_api/;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 60s;
        proxy_connect_timeout 5s;
    }

    location = /health {
        proxy_pass http://lab_api/health;
        access_log off;
    }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx

curl -sS http://localhost:8080/api/health
# {"status":"ok"}

curl -sS -H 'X-Forwarded-For: 203.0.113.45' http://localhost:8080/api/whoami
# {"client":"127.0.0.1","xff":"203.0.113.45","proto":"http"}
```

**Node.js + PM2:**

```bash
sudo mkdir -p /srv/lab-node && cd /srv/lab-node
npm init -y
npm i express

cat > server.js <<'JS'
import express from 'express';
const app = express();

app.get('/health', (req, res) => res.json({ status: 'ok' }));
app.get('/whoami', (req, res) => res.json({
  ip: req.ip,
  xff: req.headers['x-forwarded-for'] ?? null,
  proto: req.headers['x-forwarded-proto'] ?? null,
}));

app.listen(3000, '127.0.0.1', () => console.log('up'));
JS

sudo npm i -g pm2
sudo -u www-data pm2 start server.js -i max --name lab-node
sudo -u www-data pm2 save
sudo env PATH=$PATH pm2 startup systemd -u www-data --hp /var/www
```

```nginx
upstream lab_node {
    server 127.0.0.1:3000;
    keepalive 64;
}

server {
    listen 8080;
    server_name node.lab.example.com;

    location / {
        proxy_pass http://lab_node;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**WebSocket برای Node:**

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

location /ws {
    proxy_pass http://lab_node;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_read_timeout 3600s;
}
```

**`trust proxy` در Express:**

```javascript
app.set('trust proxy', 'loopback');   // فقط 127.0.0.1
```

بدون آن، `req.ip` همیشه `127.0.0.1` است.

**تست بار و `keepalive`:**

```bash
# بدون keepalive
hey -n 20000 -c 200 -disable-keepalive http://localhost:8080/api/health

# با keepalive
hey -n 20000 -c 200 http://localhost:8080/api/health
```

### مقایسه و الگوهای مختلف

**Gunicorn در برابر Uvicorn در برابر uWSGI**

| | Gunicorn | Uvicorn | uWSGI |
|---|---|---|---|
| WSGI | ✅ | ❌ | ✅ |
| ASGI | ⚠️ (worker) | ✅ | ⚠️ |
| سادگی | ✅ | ✅ | ⚠️ |
| مناسب | Django/Flask | FastAPI | legacy |

**PM2 در برابر systemd**

| | PM2 | systemd |
|---|---|---|
| cluster | ✅ | ❌ (نیاز به چند unit) |
| لاگ | ✅ | ✅ (journal) |
| وابستگی | ❌ | ✅ بومی |
| توصیه | Node | ✅ عمومی |

**`proxy_pass` با `/` در برابر بدون `/`**

| | `http://app` | `http://app/` |
|---|---|---|
| `/api/users` | `/api/users` | `/users` |
| مناسب | اپ مسیر کامل می‌خواهد | اپ در ریشه |

**دیدگاه «اپ را مستقیم روی ۸۰ بگذاریم»**: بدون TLS، rate limit، لاگ، static و graceful restart. وب‌سرور جلوی آن بگذار.

### اشکال شایع

**۱) `502` با اپ در حال اجرا**
`proxy_pass` به پورت اشتباه یا `127.0.0.1` در برابر `0.0.0.0`. `ss -ltnp`.

**۲) `504`**
`proxy_read_timeout` کم یا اپ کند. لاگ اپ.

**۳) `req.ip` همیشه `127.0.0.1`**
`trust proxy` تنظیم نشده.

**۴) `redirect` به `http` در اپ پشت HTTPS**
`X-Forwarded-Proto` تنظیم نشده یا اپ آن را نمی‌خواند.

**۵) `keepalive` بی‌اثر**
`proxy_http_version 1.1` یا `Connection ""` فراموش شده.

**۶) WebSocket قطع می‌شود**
`Upgrade`/`Connection` یا timeout.

**۷) `workers` زیاد و `OOM`**
هر worker حافظه می‌گیرد. `2 × cores + 1` برای CPU-محور، کمتر برای حافظه-محور.

**۸) `gunicorn` بدون `--timeout`**
worker گیر می‌کند. `30`.

**۹) `PM2` بعد از reboot بالا نمی‌آید**
`pm2 save` + `pm2 startup`.

**۱۰) `proxy_buffering on` برای SSE**
پیام‌ها جمع می‌شوند. `off` + `X-Accel-Buffering: no`.

### نکات مهم (Tips & Tricks)

- **`127.0.0.1`** برای bind اپ، نه `0.0.0.0`.
- **`keepalive`** در upstream + `Connection ""`.
- **`trust proxy`** در اپ.
- **`X-Forwarded-*`** همه.
- **systemd** برای پایداری.
- **`proxy_connect_timeout`** کوتاه (۵s).
- **`proxy_read_timeout`** بر اساس اپ.

### خلاصه فصل

- اپ Python/Node سرور داخلی دارد، ولی وب‌سرور جلوی آن لازم است.
- `Gunicorn`/`Uvicorn` برای Python، `PM2` برای Node.
- `proxy_pass` با `/` مسیر را حذف می‌کند.
- `X-Forwarded-*` + `trust proxy` برای IP و پروتکل.
- `keepalive` نیاز به `proxy_http_version 1.1` و `Connection ""`.
- WebSocket نیاز به `Upgrade` و timeout طولانی.

**سؤال مروری:** چرا اپ را روی `0.0.0.0:8000` بایند نمی‌کنیم و `127.0.0.1` بهتر است؟

---

## فصل ۳۰: خطاهای ۴۰۳، ۴۰۴ و ۵۰۰ — دیباگ مرحله‌به‌مرحله

### مقدمه

هر کد خطا، یک مسیر تشخیص دارد. اگر ترتیب را بدانی، در چند دقیقه به علت می‌رسی؛ اگر ندانی، ساعت‌ها حدس می‌زنی. این فصل، درخت تشخیص عملی است.

### مفاهیم اساسی

**۱) `403 Forbidden` — چهار علت**

| علت | نشانه | درمان |
|---|---|---|
| مجوز فایل | `ls -l` | `chmod 644`/`755` |
| `autoindex off` + بدون `index` | پوشه | `index` یا `autoindex` |
| `deny`/`Require` | config | بررسی قاعده |
| SELinux | `ls -Z` | `semanage`/`restorecon` |
| `Directory Browsing` (IIS) | `403.14` | `defaultDocument` |

**۲) `404 Not Found` — سه علت**

| علت | نشانه | درمان |
|---|---|---|
| مسیر فایل اشتباه | `root`/`alias` | `nginx -T` |
| `location` اشتباه | تطبیق | `error_log debug` |
| اپ مسیر را نمی‌شناسد | `proxy_pass` | لاگ اپ |

**۳) `500 Internal Server Error` — پنج علت**

| علت | نشانه | درمان |
|---|---|---|
| خطای PHP | `php error log` | رفع کد |
| `.htaccess` خراب | `error_log` | `apachectl configtest` |
| خطای اپ (Python/Node) | `journalctl`/PM2 | لاگ اپ |
| مجوز نوشتن | `ls -l` | `chmod`/`chown` |
| `web.config` خراب | `500.19` | `appcmd` |

**۴) `502 Bad Gateway` — upstream**

| علت | نشانه | درمان |
|---|---|---|
| اپ خاموش | `ss -ltnp` | start |
| پورت/socket اشتباه | `proxy_pass` | اصلاح |
| SELinux | `audit.log` | `setsebool` |
| timeout اتصال | `proxy_connect_timeout` | افزایش |
| FPM pool پر | `pm.max_children` | افزایش |

**۵) `504 Gateway Timeout`**

| علت | درمان |
|---|---|
| اپ کند | بهینه‌سازی |
| `proxy_read_timeout` کم | افزایش |
| دیتابیس کند | `pg_stat_statements` |

**۶) `413 Request Entity Too Large`**

| لایه | تنظیم |
|---|---|
| Nginx | `client_max_body_size` |
| Apache | `LimitRequestBody` |
| IIS | `maxAllowedContentLength` |
| PHP | `upload_max_filesize`، `post_max_size` |
| Python | `client_max_body_size` + اپ |

**۷) `499` (Nginx)**
کلاینت قبل از پاسخ، اتصال را بست. معمولاً timeout کلاینت یا اپ کند.

**۸) ترتیب دیباگ**

```
۱. کد وضعیت را ببین (curl -I)
۲. لاگ وب‌سرور (error.log)
۳. لاگ اپ
۴. مجوز فایل (namei -l)
۵. config مؤثر (nginx -T / apachectl -S)
۶. تست مستقیم upstream (curl 127.0.0.1:PORT)
```

**۹) `error_log` با سطح debug**

```nginx
error_log /var/log/nginx/debug.log debug;
```

```bash
sudo tail -f /var/log/nginx/debug.log | grep -E "using config|open\(\)|stat\(\)"
```

**۱۰) `strace` برای فراخوانی سیستمی**

```bash
sudo strace -f -p $(pgrep -f 'nginx: worker' | head -1) -e trace=openat,stat 2>&1 | grep -i 'app.css'
```

### مثال‌های عملی

**Lab Scenario: پنج خطا، پنج تشخیص**

```bash
# آماده‌سازی
sudo mkdir -p /var/www/lab/{ok,noindex,private,broken}
echo 'ok' | sudo tee /var/www/lab/ok/index.html
echo 'secret' | sudo tee /var/www/lab/private/data.txt
sudo chmod 600 /var/www/lab/private/data.txt
sudo chown -R root:root /var/www/lab/private
```

```nginx
server {
    listen 8080;
    server_name _;
    root /var/www/lab;

    location /ok/ { try_files $uri $uri/ =404; }
    location /noindex/ { try_files $uri $uri/ =404; }
    location /private/ { try_files $uri =404; }
    location /missing/ { try_files $uri =404; }
}
```

**خطای ۱: `403` از مجوز**

```bash
curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/private/data.txt
# 403

# تشخیص
namei -l /var/www/lab/private/data.txt
# drwxr-xr-x root root /var/www/lab/private
# -rw------- root root data.txt      ← www-data نمی‌تواند بخواند

# درمان
sudo chmod 644 /var/www/lab/private/data.txt
sudo chown www-data:www-data /var/www/lab/private/data.txt
curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/private/data.txt
# 200
```

**خطای ۲: `403` از `autoindex`**

```bash
curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/noindex/
# 403 (پوشه خالی، بدون index، autoindex off)

# تشخیص
sudo tail -5 /var/log/nginx/error.log
# directory index of "/var/www/lab/noindex/" is forbidden

# درمان: index یا autoindex محدود
```

**خطای ۳: `404` از `alias` اشتباه**

```nginx
location /static/ {
    alias /var/www/lab/static;    # ❌ بدون / پایانی
}
```

```bash
curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/static/app.css
# 404

# تشخیص
sudo nginx -T | grep -A2 "location /static/"
ls -l /var/www/lab/static/app.css
# فایل هست، ولی alias مسیر را غلط ساخته

# درمان
# alias /var/www/lab/static/;
```

**خطای ۴: `502` از upstream خاموش**

```nginx
location /api/ {
    proxy_pass http://127.0.0.1:9999/;    # پورت اشتباه
}
```

```bash
curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/api/health
# 502

# تشخیص
ss -ltnp | grep 9999
# (خالی)
sudo tail -3 /var/log/nginx/error.log
# connect() failed (111: Connection refused) while connecting to upstream

# درمان: پورت درست یا start اپ
```

**خطای ۵: `500` از PHP**

```php
<?php
// /var/www/lab/broken/index.php
undefined_function();   // خطای fatal
```

```bash
curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/broken/
# 500

# تشخیص
sudo tail -20 /var/log/php/error.log
# PHP Fatal error: Uncaught Error: Call to undefined function undefined_function()

# درمان: رفع کد
```

**خطای ۶: `413`**

```bash
# ساخت فایل ۲۰ مگابایتی
dd if=/dev/zero of=/tmp/big.bin bs=1M count=20

curl -sS -o /dev/null -w '%{http_code}\n' -X POST --data-binary @/tmp/big.bin http://localhost:8080/upload
# 413

# تشخیص
grep client_max_body_size /etc/nginx/nginx.conf
# client_max_body_size 16m;

# درمان
# client_max_body_size 32m;
# و در PHP: upload_max_filesize = 32M، post_max_size = 32M
```

**دیباگ با `curl` مرحله‌به‌مرحله:**

```bash
# ۱) کد و هدر
curl -sSI http://localhost:8080/api/health

# ۲) با Host درست
curl -sSI -H 'Host: api.lab.example.com' http://127.0.0.1:8080/api/health

# ۳) مستقیم به upstream
curl -sSI http://127.0.0.1:8000/health

# ۴) با verbose
curl -v http://localhost:8080/api/health 2>&1 | head -30

# ۵) زمان‌بندی
curl -sS -o /dev/null -w 'code=%{http_code} connect=%{time_connect} ttfb=%{time_starttransfer} total=%{time_total}\n' \
  http://localhost:8080/api/health
```

**دیباگ Apache:**

```bash
sudo apachectl configtest
sudo tail -30 /var/log/apache2/error.log
sudo apachectl -S
# LogLevel alert rewrite:trace3
```

**دیباگ IIS:**

```powershell
Get-Content C:\inetpub\logs\LogFiles\W3SVC1\*.log -Tail 20
Get-WinEvent -LogName Application -MaxEvents 20 |
  Where-Object { $_.ProviderName -like "*IIS*" }
# failed request tracing
```

### مقایسه و الگوهای مختلف

**`403` در برابر `401`**

| | `401` | `403` |
|---|---|---|
| معنا | احراز هویت لازم | احراز شده ولی ممنوع |
| هدر | `WWW-Authenticate` | — |
| درمان | login | مجوز |

**`404` در برابر `410`**

| | `404` | `410` |
|---|---|---|
| معنا | پیدا نشد | حذف شده (دائمی) |
| SEO | ⚠️ | ✅ حذف از ایندکس |

**`502` در برابر `503` در برابر `504`**

| | `502` | `503` | `504` |
|---|---|---|---|
| معنا | پاسخ بد از upstream | سرویس ناموجود | timeout upstream |
| علت | اپ خاموش/خطا | maintenance/overload | کندی |
| Retry-After | ❌ | ✅ | ❌ |

**سه سطح لاگ**

| سطح | تنظیم |
|---|---|
| وب‌سرور | `error_log` |
| اپ | `journalctl`/PM2/`stdout` |
| سیستم | `dmesg`، `audit.log` (SELinux) |

**دیدگاه «`display_errors = On` برای دیباگ»**: در توسعه بله. در تولید، `Off` + `log_errors`؛ وگرنه مسیر و کد لو می‌رود.

### اشکال شایع

**۱) حدس زدن بدون لاگ**
`error.log` همیشه اول.

**۲) `502` و فرض «Nginx مشکل دارد»**
معمولاً upstream. `curl 127.0.0.1:PORT`.

**۳) `403` و `chmod 777`**
امنیت را نابود می‌کند. `644`/`755` + مالک درست.

**۴) `404` و تغییر `location` بدون `-t`**
config اعمال نشده. `nginx -t`.

**۵) `500` و نبود `display_errors`**
لاگ PHP را ببین، نه صفحه.

**۶) `413` و فقط `client_max_body_size`**
PHP هم حد دارد. هر دو.

**۷) `504` و افزایش timeout**
مشکل را پنهان می‌کند. اپ را بهینه کن.

**۸) SELinux نادیده**
`audit.log` را ببین. `setsebool`.

**۹) `499` و فرض خطای سرور**
کلاینت قطع کرده. لاگ کلاینت.

**۱۰) `curl` بدون `-H Host`**
default server پاسخ می‌دهد. `--resolve`.

### نکات مهم (Tips & Tricks)

- **`curl -sSI`** کد و هدر.
- **`error_log`** همیشه.
- **`namei -l`** برای مسیر و مجوز.
- **`nginx -T`**/`apachectl -S` برای config مؤثر.
- **`curl 127.0.0.1:PORT`** برای upstream.
- **`ss -ltnp`** برای پورت.
- **`journalctl -u`** برای اپ systemd.

### خلاصه فصل

- `403` = مجوز/`autoindex`/SELinux.
- `404` = مسیر/`location`/اپ.
- `500` = اپ/config.
- `502` = upstream خاموش یا خطا.
- `504` = کندی اپ.
- ترتیب دیباگ: کد → لاگ وب‌سرور → لاگ اپ → مجوز → config.

**سؤال مروری:** `502` می‌گیری. سه دستوری که به ترتیب می‌زنی تا علت را پیدا کنی کدام‌اند؟

---

## منابع و مراجع

**مستندات رسمی**
- [Nginx — Serving Static Content](https://docs.nginx.com/nginx/admin-guide/web-server/serving-static-content/)
- [Nginx — FastCGI](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html)
- [Apache — mod_proxy_fcgi](https://httpd.apache.org/docs/2.4/mod/mod_proxy_fcgi.html)
- [PHP-FPM Configuration](https://www.php.net/manual/en/install.fpm.configuration.php)
- [Gunicorn Docs](https://docs.gunicorn.org/)
- [Uvicorn Docs](https://www.uvicorn.org/)
- [PM2 Docs](https://pm2.keymetrics.io/docs/)
- [IIS — Static Content](https://learn.microsoft.com/en-us/iis/configuration/system.webserver/staticcontent/)

**آموزش**
- [DigitalOcean — Nginx + PHP-FPM](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack)
- [Deploying FastAPI with Gunicorn + Nginx](https://fastapi.tiangolo.com/deployment/server-workers/)
- [Debugging 502 Bad Gateway](https://www.nginx.com/resources/wiki/start/topics/tutorials/debugging/)

**ابزار**
- [hey](https://github.com/rakyll/hey)
- [namei](https://man7.org/linux/man-pages/man1/namei.1.html)
- [strace](https://strace.io/)

---

**بعدی:** [`08-Reverse-Proxy.md`](08-Reverse-Proxy.md) — `proxy_pass`، `mod_proxy`، هدرها و WebSocket
