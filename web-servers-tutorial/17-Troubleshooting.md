# بخش ۱۷: عیب‌یابی

**مدت یادگیری**: ۴ تا ۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۰۷ (خطاها)، بخش ۱۱ (لاگ)

---

## فهرست محتوا

- [فصل ۷۲: `502`، `504`، `refused` و `timeout`](#فصل-۷۲)
- [فصل ۷۳: `403`، `404`، `500` و مجوز/SELinux](#فصل-۷۳)
- [فصل ۷۴: TLS، گواهی و DNS](#فصل-۷۴)
- [فصل ۷۵: جعبه‌ابزار — `curl`، `openssl`، `tcpdump`، `strace`](#فصل-۷۵)

---

## فصل ۷۲: `502`، `504`، `refused` و `timeout`

### مقدمه

چهار خطای «شبکه‌ای» که ۸۰٪ حادثه‌ها را می‌سازند. هر کدام یک معنا و یک مسیر تشخیص دارد.

### مفاهیم اساسی

**۱) تفکیک**

| خطا | معنا | لایه |
|---|---|---|
| `Connection refused` | پورت گوش نمی‌دهد | TCP |
| `Timeout` | بسته drop می‌شود | شبکه |
| `502 Bad Gateway` | upstream پاسخ بد داد | HTTP |
| `504 Gateway Timeout` | upstream کند بود | HTTP |

**۲) `502` — پنج علت**

| علت | تشخیص |
|---|---|
| اپ خاموش | `ss -ltnp` |
| پورت اشتباه | `proxy_pass` |
| socket اشتباه | `ls -l /run/php/*.sock` |
| SELinux | `audit.log` |
| FPM pool پر | `pm.max_children` |

**۳) `504` — سه علت**

| علت | تشخیص |
|---|---|
| اپ کند | لاگ اپ، `slowlog` |
| `proxy_read_timeout` کم | config |
| DB کند | `pg_stat_statements` |

**۴) `Connection refused`**

```bash
ss -ltnp | grep :8000        # گوش می‌دهد؟
curl -v http://127.0.0.1:8000/   # از خود سرور
sudo systemctl status app
sudo journalctl -u app -n 50
```

**۵) `Timeout`**

```bash
nc -zv host 443
traceroute host
sudo tcpdump -i any port 443 -c 20
```

**۶) `502` در Nginx — لاگ**

```bash
sudo tail -20 /var/log/nginx/error.log
# connect() failed (111: Connection refused) while connecting to upstream
# upstream prematurely closed connection
# no live upstreams
```

**۷) `499` (Nginx)**

کلاینت قبل از پاسخ، اتصال را بست. معمولاً اپ کند یا timeout کلاینت.

**۸) `upstream timed out`**

```
upstream timed out (110: Connection timed out) while reading response header from upstream
```

`proxy_read_timeout`.

**۹) `no live upstreams`**

همه‌ی upstreamها `max_fails` را رد کرده‌اند.

**۱۰) `recv() failed (104: Connection reset by peer)`**

اپ اتصال را بست. کرش یا timeout داخلی.

### مثال‌های عملی

**Lab Scenario: بازتولید و تشخیص چهار خطا**

```bash
# ۱) Connection refused
python3 -m http.server 8000 --bind 127.0.0.1 &
curl -sS http://127.0.0.1:8000/ | head -1
kill %1
curl -sS -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8000/
# 000 (refused)
```

```nginx
# ۲) 502 با upstream خاموش
location /api/ {
    proxy_pass http://127.0.0.1:8000/;
}
```

```bash
curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/api/
# 502
sudo tail -2 /var/log/nginx/error.log
# connect() failed (111: Connection refused)
```

```bash
# ۳) 504 با اپ کند
# اپ با sleep 60
python3 -c "
from http.server import BaseHTTPRequestHandler, HTTPServer
import time
class H(BaseHTTPRequestHandler):
    def do_GET(self):
        time.sleep(60)
        self.send_response(200); self.end_headers()
HTTPServer(('127.0.0.1', 8000), H).serve_forever()
" &
```

```nginx
location /slow/ {
    proxy_pass http://127.0.0.1:8000/;
    proxy_read_timeout 5s;
}
```

```bash
time curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/slow/
# 504 بعد از ۵ ثانیه
sudo tail -2 /var/log/nginx/error.log
# upstream timed out (110: Connection timed out)
```

```bash
# ۴) 499
# کلاینت با timeout کوتاه
curl -sS --max-time 1 -o /dev/null -w '%{http_code}\n' http://localhost:8080/slow/
# 000 در سمت کلاینت، 499 در لاگ Nginx
sudo grep ' 499 ' /var/log/nginx/access.log | tail -1
```

**درخت تشخیص `502`:**

```bash
#!/usr/bin/env bash
set -uo pipefail

UPSTREAM=${1:-127.0.0.1:8000}

echo "== 1) پورت"
ss -ltnp | grep -F "$UPSTREAM" || echo "  ✗ گوش نمی‌دهد"

echo "== 2) اتصال مستقیم"
curl -sS -o /dev/null -w '  HTTP %{http_code}\n' "http://$UPSTREAM/" || echo "  ✗ refused"

echo "== 3) از طریق proxy"
curl -sS -o /dev/null -w '  HTTP %{http_code}\n' http://localhost:8080/api/ || true

echo "== 4) لاگ"
sudo tail -3 /var/log/nginx/error.log

echo "== 5) SELinux"
command -v getenforce >/dev/null && getenforce
sudo ausearch -m avc -ts recent 2>/dev/null | tail -5 || true
```

**SELinux و `502`:**

```bash
sudo ausearch -m avc -ts recent
# type=AVC ... denied { name_connect } for ... tcp_socket ... dest=8000
sudo setsebool -P httpd_can_network_connect 1
```

**PHP-FPM `502`:**

```bash
sudo systemctl status php8.3-fpm
sudo tail -20 /var/log/php8.3-fpm.log
ls -l /run/php/php8.3-fpm.sock
# srw-rw---- 1 www-data www-data
```

اگر socket نبود:

```bash
sudo systemctl restart php8.3-fpm
sudo systemctl enable php8.3-fpm
```

**`502` از pool پر:**

```bash
sudo grep 'pm.max_children' /var/log/php8.3-fpm.log | tail -3
# WARNING: [pool www] server reached pm.max_children setting (25)
```

**`504` از DB:**

```bash
# در اپ
sudo journalctl -u lab-api -n 50 | grep -i timeout
# در DB
psql -c "SELECT pid, now()-query_start, left(query,60) FROM pg_stat_activity WHERE state='active' ORDER BY 2 DESC LIMIT 5;"
```

**`upstream prematurely closed`:**

```bash
# اپ کرش کرده
sudo journalctl -u lab-api --since "10 min ago" | grep -iE 'error|panic'
```

### مقایسه و الگوهای مختلف

**`502` در برابر `503`**

| | `502` | `503` |
|---|---|---|
| معنا | upstream بد | سرویس ناموجود |
| علت | کرش، پاسخ بد | maintenance، overload |
| `Retry-After` | ❌ | ✅ |

**`504` در برابر `499`**

| | `504` | `499` |
|---|---|---|
| چه کسی بست | Nginx (timeout) | کلاینت |
| لاگ | `error.log` | `access.log` |
| علت | اپ کند | کلاینت صبر نکرد |

**`refused` در برابر `timeout`**

| | refused | timeout |
|---|---|---|
| معنا | پورت بسته | بسته drop |
| علت | سرویس خاموش | فایروال/شبکه |
| تشخیص | `ss` | `tcpdump` |

**دیدگاه «`502` یعنی Nginx خراب است»**: تقریباً همیشه upstream. Nginx فقط گزارش می‌دهد.

### اشکال شایع

**۱) `proxy_pass` به `localhost` و IPv6**
`localhost` ممکن است `::1` باشد، اپ روی `127.0.0.1`. `127.0.0.1` صریح.

**۲) اپ روی `0.0.0.0` و فایروال**
`refused` از بیرون. `listen` درست.

**۳) `max_fails` و `fail_timeout`**
upstream سالم، غیرفعال. تنظیم.

**۴) `proxy_read_timeout` پیش‌فرض ۶۰s**
گزارش ۵ دقیقه‌ای، `504`. افزایش.

**۵) `client_max_body_size`**
`413`، نه `502`. تفکیک.

**۶) SELinux**
`502` بی‌توضیح. `audit.log`.

**۷) `pm.max_children` کم**
`502` زیر بار. افزایش.

**۸) `upstream prematurely closed`**
کرش اپ. لاگ اپ.

**۹) `499` و فرض خطای سرور**
کلاینت قطع کرده.

**۱۰) لاگ بدون `$upstream_addr`**
نمی‌دانی کدام upstream. (فصل ۴۶)

### نکات مهم (Tips & Tricks)

- **`curl` مستقیم به upstream**.
- **`ss -ltnp`** برای پورت.
- **`error.log`** با `$upstream_addr`.
- **`ausearch`** برای SELinux.
- **`127.0.0.1`** به‌جای `localhost`.
- **`proxy_connect_timeout 5s`**.
- **`X-Upstream`** هدر برای دیباگ.

### خلاصه فصل

- `refused` = پورت بسته، `timeout` = drop.
- `502` = upstream خاموش/خطا.
- `504` = upstream کند.
- `499` = کلاینت بست.
- SELinux و FPM pool، علت‌های پنهان.
- `127.0.0.1` صریح، نه `localhost`.

**سؤال مروری:** `502` می‌گیری. سه دستور اول به ترتیب چیست؟

---

## فصل ۷۳: `403`، `404`، `500` و مجوز/SELinux

### مقدمه

خطاهای «محتوایی» بیشتر از پیکربندی می‌آیند تا شبکه. ترتیب تشخیص: مسیر، مجوز، SELinux، config.

### مفاهیم اساسی

**۱) `403` — پنج علت**

| علت | تشخیص |
|---|---|
| مجوز فایل | `namei -l` |
| `autoindex off` + بدون `index` | پوشه |
| `deny`/`Require` | config |
| SELinux | `ls -Z` |
| `Directory Browsing` (IIS) | `403.14` |

**۲) `404` — چهار علت**

| علت | تشخیص |
|---|---|
| `root`/`alias` | `nginx -T` |
| `location` | `error_log debug` |
| اپ | لاگ اپ |
| case-sensitive | مسیر |

**۳) `500` — پنج علت**

| علت | تشخیص |
|---|---|
| PHP fatal | `php error log` |
| `.htaccess` | `error_log` |
| اپ | `journalctl` |
| مجوز نوشتن | `ls -l` |
| `web.config` | `500.19` |

**۴) `namei -l`**

```bash
namei -l /var/www/lab/assets/app.css
# drwxr-xr-x root root /
# drwxr-xr-x root root var
# drwxr-xr-x www-data www-data www
# drwxr-xr-x www-data www-data lab
# drwxr-xr-x www-data www-data assets
# -rw-r--r-- www-data www-data app.css
```

هر سطح باید `x` برای `www-data` داشته باشد (پوشه) و `r` (فایل).

**۵) SELinux context**

```bash
ls -Z /var/www/lab
# unconfined_u:object_r:httpd_sys_content_t:s0 index.html
```

اگر `user_home_t` بود:

```bash
sudo semanage fcontext -a -t httpd_sys_content_t "/srv/lab(/.*)?"
sudo restorecon -Rv /srv/lab
```

**۶) `audit.log`**

```bash
sudo ausearch -m avc -ts recent
# denied { read } for ... path="/var/www/lab/index.html"
```

**۷) `error_log` Nginx**

```bash
sudo tail -20 /var/log/nginx/error.log
# *1 open() "/var/www/lab/x" failed (13: Permission denied)
# directory index of "/var/www/lab/" is forbidden
# *2 open() "/var/www/lab/missing" failed (2: No such file or directory)
```

کد `13` = `EACCES`، کد `2` = `ENOENT`.

**۸) `strace` برای `EACCES`**

```bash
sudo strace -f -p $(pgrep -f 'nginx: worker' | head -1) -e trace=openat 2>&1 | grep -E 'EACCES|ENOENT' | tail
```

**۹) `500` در PHP**

```bash
sudo tail -30 /var/log/php/error.log
# PHP Fatal error: Uncaught Error: ...
```

اگر خالی بود:

```ini
log_errors = On
error_log = /var/log/php/error.log
display_errors = Off
```

**۱۰) `500` در `.htaccess`**

```bash
sudo tail -5 /var/log/apache2/error.log
# /var/www/lab/.htaccess: Invalid command 'RewriteEngine'
```

`AllowOverride` یا ماژول.

### مثال‌های عملی

**Lab Scenario: پنج خطا، پنج تشخیص**

```bash
sudo mkdir -p /var/www/lab/{assets,private,noindex,broken}
echo 'body{}' | sudo tee /var/www/lab/assets/app.css
echo 'secret' | sudo tee /var/www/lab/private/data.txt
sudo chmod 600 /var/www/lab/private/data.txt
sudo chown -R www-data:www-data /var/www/lab
sudo chown -R root:root /var/www/lab/private
```

```nginx
server {
    listen 8080;
    server_name _;
    root /var/www/lab;

    location /assets/ { try_files $uri =404; }
    location /private/ { try_files $uri =404; }
    location /noindex/ { try_files $uri $uri/ =404; }
    location /broken/ {
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
    location / { try_files $uri $uri/ =404; }
}
```

**۱) `403` از مجوز:**

```bash
curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/private/data.txt
# 403

namei -l /var/www/lab/private/data.txt
# -rw------- root root data.txt   ← www-data نمی‌تواند

sudo chmod 644 /var/www/lab/private/data.txt
sudo chown www-data:www-data /var/www/lab/private/data.txt
curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/private/data.txt
# 200
```

**۲) `403` از `autoindex`:**

```bash
curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/noindex/
# 403
sudo tail -2 /var/log/nginx/error.log
# directory index of "/var/www/lab/noindex/" is forbidden

# درمان
echo '<h1>idx</h1>' | sudo tee /var/www/lab/noindex/index.html
```

**۳) `404` از `alias`:**

```nginx
location /static/ {
    alias /var/www/lab/assets;   # بدون / پایانی
}
```

```bash
curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/static/app.css
# 404

sudo nginx -T | grep -A3 'location /static/'
ls -l /var/www/lab/assets/app.css
# فایل هست؛ alias مسیر را غلط ساخته

# درمان: alias /var/www/lab/assets/;
```

**۴) `500` از PHP:**

```bash
echo '<?php undefined_fn();' | sudo tee /var/www/lab/broken/index.php
sudo chown www-data:www-data /var/www/lab/broken/index.php

curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/broken/
# 500

sudo tail -5 /var/log/php/error.log
# PHP Fatal error: Uncaught Error: Call to undefined function undefined_fn()
```

**۵) `404` از case:**

```bash
curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/Assets/app.css
# 404 (حساس به حروف در لینوکس)
```

**SELinux:**

```bash
sudo semanage fcontext -l | grep /srv/lab
sudo restorecon -Rv /srv/lab
sudo ausearch -m avc -ts today | grep lab | tail -5
```

**`strace` برای `EACCES`:**

```bash
PID=$(pgrep -f 'nginx: worker' | head -1)
sudo strace -f -p "$PID" -e trace=openat 2>&1 | grep -E 'EACCES|ENOENT' | tail -5
```

**IIS `403.14`:**

```powershell
Get-WebConfigurationProperty -PSPath "IIS:\Sites\lab" `
  -Filter "system.webServer/defaultDocument" -Name "enabled"
# باید True باشد
```

**IIS `500.19`:**

```powershell
& $env:windir\system32\inetsrv\appcmd.exe list config "lab" /section:system.webServer/security/requestFiltering
# اگر خطا داد، web.config نامعتبر
```

### مقایسه و الگوهای مختلف

**`403` در برابر `404` برای فایل حساس**

| | `403` | `404` |
|---|---|---|
| معنا | وجود دارد، ممنوع | وجود ندارد |
| افشا | ✅ وجود را می‌گوید | ✅ پنهان |
| توصیه | ⚠️ | ✅ فایل حساس |

**`root` در برابر `alias` در خطا**

| | `root` | `alias` |
|---|---|---|
| `404` رایج | مسیر اضافه | `/` پایانی |
| `403` رایج | مجوز | مجوز |

**`namei` در برابر `ls -l`**

| | `namei` | `ls` |
|---|---|---|
| همه‌ی سطوح | ✅ | ❌ |
| سرعت | ✅ | ✅ |
| توصیه | ✅ | تک‌فایل |

**دیدگاه «`chmod 777` حل می‌کند»**: حل می‌کند و امنیت را نابود. `644`/`755` + مالک درست.

### اشکال شایع

**۱) `chmod 777`**
امنیت. `644`/`755`.

**۲) `www-data` بدون `x` روی پوشه‌ی والد**
`403`. `namei`.

**۳) `alias` بدون `/`**
`404`.

**۴) SELinux**
`403` بی‌توضیح. `ls -Z`.

**۵) `index` نبودن**
`403`. `index` یا `autoindex`.

**۶) case-sensitive**
`404`. مسیر دقیق.

**۷) `display_errors = Off` بدون لاگ**
`500` بی‌توضیح. `log_errors`.

**۸) `.htaccess` با ماژول غیرفعال**
`500`. `a2enmod`.

**۹) `open_basedir` محدود**
`500`/`403`. مسیر.

**۱۰) `root` به‌جای `alias`**
مسیر دوبار. `alias`.

### نکات مهم (Tips & Tricks)

- **`namei -l`** اول.
- **`ls -Z`** برای SELinux.
- **`error.log`** کد `13`/`2`.
- **`strace`** برای `EACCES`.
- **`nginx -T`** برای `alias`.
- **`log_errors`** برای PHP.
- **`404` برای فایل حساس**.

### خلاصه فصل

- `403` = مجوز/`autoindex`/SELinux.
- `404` = مسیر/`alias`/case.
- `500` = اپ/config.
- `namei -l` همه‌ی سطوح را نشان می‌دهد.
- `strace` علت دقیق را می‌دهد.
- `chmod 777` هرگز.

**سؤال مروری:** فایل وجود دارد و `644` است، ولی `403` می‌گیری. دو علت ممکن؟

---

## فصل ۷۴: TLS، گواهی و DNS

### مقدمه

خطاهای TLS و DNS، پیام‌های گمراه‌کننده دارند. «certificate verify failed» می‌تواند ده علت داشته باشد.

### مفاهیم اساسی

**۱) خطاهای رایج TLS**

| خطا | علت |
|---|---|
| `unable to verify the first certificate` | زنجیره ناقص (`cert.pem` به‌جای `fullchain`) |
| `certificate has expired` | منقضی |
| `certificate is not valid for this domain` | SAN/CN اشتباه |
| `self signed certificate` | self-signed |
| `no alternative certificate subject name` | دامنه در SAN نیست |
| `wrong version number` | HTTP روی پورت HTTPS |
| `handshake failure` | cipher/protocol ناسازگار |
| `tlsv1 alert protocol version` | پروتکل قدیمی |

**۲) `wrong version number`**

```
کلاینت: openssl s_client -connect host:443
سرور:   HTTP/1.1 400 Bad Request  ← HTTP روی ۴۴۳!
```

یعنی پورت ۴۴۳ به HTTP وصل است، نه TLS.

**۳) `unable to verify`**

```bash
echo | openssl s_client -connect lab.example.com:443 -servername lab.example.com 2>&1 | grep -A2 'Verify return'
# Verify return code: 21 (unable to verify the first certificate)
```

`fullchain.pem` استفاده کن.

**۴) بررسی زنجیره**

```bash
openssl s_client -connect lab.example.com:443 -servername lab.example.com -showcerts </dev/null 2>/dev/null \
  | grep -E 'depth=|s:|i:'
# depth=0 CN = lab.example.com
# depth=1 C = US, O = Let's Encrypt, CN = R3
```

اگر فقط `depth=0` بود، intermediate نیست.

**۵) `SAN` بررسی**

```bash
echo | openssl s_client -connect lab.example.com:443 -servername lab.example.com 2>/dev/null \
  | openssl x509 -noout -ext subjectAltName
# X509v3 Subject Alternative Name:
#     DNS:lab.example.com, DNS:www.lab.example.com
```

**۶) DNS — خطاهای رایج**

| خطا | علت |
|---|---|
| `NXDOMAIN` | رکورد نیست |
| `SERVFAIL` | DNSSEC/سرور نام |
| `Timeout` | resolver |
| IP اشتباه | رکورد قدیمی/کش |

**۷) `curl` با `--resolve`**

```bash
curl -sSI --resolve lab.example.com:443:192.168.56.10 https://lab.example.com/
```

بدون تغییر DNS.

**۸) `dig +trace`**

```bash
dig +trace lab.example.com
```

مسیر کامل حل نام.

**۹) `openssl s_client` با `-servername`**

بدون `-servername`، SNI فرستاده نمی‌شود و ممکن است گواهی پیش‌فرض بگیری.

**۱۰) `curl -k`**

`-k` تأیید گواهی را نادیده می‌گیرد. **فقط برای تست**، هرگز در کد.

### مثال‌های عملی

**Lab Scenario: پنج خطای TLS**

```bash
# ۱) زنجیره ناقص
# nginx با cert.pem به‌جای fullchain.pem
echo | openssl s_client -connect lab.example.com:443 -servername lab.example.com 2>&1 | grep 'Verify return code'
# 21 (unable to verify the first certificate)

# درمان
# ssl_certificate .../fullchain.pem;
```

**۲) دامنه در SAN نیست:**

```bash
echo | openssl s_client -connect lab.example.com:443 -servername api.lab.example.com 2>/dev/null \
  | openssl x509 -noout -ext subjectAltName
# DNS:lab.example.com   ← api نیست

# درمان: گواهی با SAN یا wildcard
```

**۳) `wrong version number`:**

```bash
curl -sSI https://lab.example.com:8080/
# curl: (35) error:0A00010B:SSL routines::wrong version number
```

یعنی `:8080` HTTP است.

**۴) پروتکل قدیمی:**

```bash
openssl s_client -connect lab.example.com:443 -tls1 </dev/null 2>&1 | head -3
# tlsv1 alert protocol version
```

TLS 1.0 غیرفعال است (خوب).

**۵) منقضی:**

```bash
echo | openssl s_client -connect lab.example.com:443 -servername lab.example.com 2>/dev/null \
  | openssl x509 -noout -dates
# notAfter=Sep  1 00:00:00 2026 GMT   ← گذشته
```

**تست DNS:**

```bash
# ۱) رکورد
dig +short lab.example.com

# ۲) از سرور نام معتبر
dig @ns1.example.com lab.example.com +norecurse

# ۳) مسیر
dig +trace lab.example.com | tail -20

# ۴) CAA
dig CAA lab.example.com +short

# ۵) کش محلی
sudo resolvectl flush-caches

# ۶) از دید سرور
getent hosts lab.example.com
```

**تست TLS کامل:**

```bash
#!/usr/bin/env bash
set -uo pipefail
HOST=${1:?host}

echo "== resolve"
getent hosts "$HOST" || echo "  ✗ DNS"

echo "== connect"
timeout 5 bash -c "</dev/tcp/$HOST/443" && echo "  ok" || echo "  ✗ port"

echo "== chain"
echo | openssl s_client -connect "$HOST:443" -servername "$HOST" 2>&1 \
  | grep -E 'Verify return code|depth='

echo "== dates"
echo | openssl s_client -connect "$HOST:443" -servername "$HOST" 2>/dev/null \
  | openssl x509 -noout -dates

echo "== SAN"
echo | openssl s_client -connect "$HOST:443" -servername "$HOST" 2>/dev/null \
  | openssl x509 -noout -ext subjectAltName

echo "== protocols"
nmap --script ssl-enum-ciphers -p 443 "$HOST" 2>/dev/null | grep -E 'TLSv1\.[0-9]:'

echo "== HTTP"
curl -sSI "https://$HOST/" | head -1
```

**`curl -k` فقط تست:**

```bash
# ❌ در کد
curl -k https://api.example.com/

# ✅ در تست محلی با گواهی self-signed
curl -k --resolve lab.local:443:192.168.56.10 https://lab.local/
```

**گواهی self-signed در تست:**

```bash
openssl req -x509 -newkey rsa:2048 -nodes -days 1 \
  -keyout /tmp/test.key -out /tmp/test.crt -subj "/CN=lab.local"

# curl با CA
curl --cacert /tmp/test.crt https://lab.local/
```

**DNS split-horizon:**

```bash
# از داخل شبکه
dig @10.0.0.1 lab.example.com +short
# 192.168.56.10

# از بیرون
dig @8.8.8.8 lab.example.com +short
# 203.0.113.10
```

### مقایسه و الگوهای مختلف

**`curl -k` در برابر `--cacert`**

| | `-k` | `--cacert` |
|---|---|---|
| تأیید | ❌ | ✅ |
| امنیت | ❌ | ✅ |
| مناسب | تست | ✅ CI داخلی |

**`-servername` در برابر بدون**

| | با | بدون |
|---|---|---|
| SNI | ✅ | ❌ |
| گواهی درست | ✅ | ⚠️ |
| توصیه | ✅ | ❌ |

**`dig` در برابر `nslookup` در برابر `host`**

| | `dig` | `nslookup` | `host` |
|---|---|---|---|
| جزئیات | ✅✅ | ⚠️ | ✅ |
| توصیه | ✅ | ❌ | ✅ |

**دیدگاه «`-k` مشکل را حل می‌کند»**: مشکل را پنهان می‌کند. گواهی را درست کن.

### اشکال شایع

**۱) `cert.pem` بدون زنجیره**
`unable to verify`. `fullchain`.

**۲) `-servername` نبودن**
گواهی پیش‌فرض. اضافه کن.

**۳) `curl -k` در کد**
امنیت. حذف.

**۴) DNS کش‌شده**
`flush-caches` و `TTL`.

**۵) `CAA` مانع صدور**
`dig CAA`.

**۶) `AAAA` اشتباه**
IPv6 اشتباه. `curl -4`.

**۷) گواهی self-signed**
`--cacert` یا CA عمومی.

**۸) پروتکل قدیمی**
`tlsv1 alert`. ارتقا.

**۹) `wrong version number`**
HTTP روی HTTPS. پورت.

**۱۰) گواهی منقضی**
`certbot renew`.

### نکات مهم (Tips & Tricks)

- **`openssl s_client -servername`**.
- **`-showcerts`** برای زنجیره.
- **`-ext subjectAltName`**.
- **`dig +trace`**.
- **`curl --resolve`**.
- **`nmap ssl-enum-ciphers`**.
- **`testssl.sh`** برای جامع.

### خلاصه فصل

- `unable to verify` = زنجیره ناقص.
- `wrong version number` = HTTP روی HTTPS.
- `-servername` برای SNI.
- DNS: `dig`/`getent`/`resolvectl`.
- `-k` فقط تست.
- `testssl.sh` برای بررسی جامع.

**سؤال مروری:** چرا `openssl s_client` بدون `-servername` می‌تواند گواهی اشتباه نشان دهد؟

---

## فصل ۷۵: جعبه‌ابزار — `curl`، `openssl`، `tcpdump`، `strace`

### مقدمه

چهار ابزار که ۹۰٪ عیب‌یابی را ممکن می‌کنند. هر کدام یک لایه را می‌بیند.

### مفاهیم اساسی

**۱) `curl` — لایه‌ی HTTP**

```bash
curl -sS -v https://lab.example.com/ 2>&1 | head -30
curl -sSI https://lab.example.com/              # فقط هدر
curl -sS -o /dev/null -w '%{http_code} %{time_total}\n' URL
curl -sS --http2 -I URL
curl -sS -L URL                                  # دنبال redirect
curl -sS --resolve host:443:IP URL
curl -sS -H 'Host: x' http://IP/
curl -sS -x http://proxy:3128 URL                # از طریق پروکسی
```

**۲) `-w` — قالب زمان‌بندی**

```bash
curl -sS -o /dev/null -w '
dns=%{time_namelookup}
connect=%{time_connect}
tls=%{time_appconnect}
ttfb=%{time_starttransfer}
total=%{time_total}
code=%{http_code}
ip=%{remote_ip}
size=%{size_download}
' https://lab.example.com/
```

**۳) `openssl` — لایه‌ی TLS**

```bash
openssl s_client -connect host:443 -servername host -showcerts </dev/null
openssl s_client -connect host:443 -tls1_3 </dev/null
openssl s_client -connect host:443 -alpn h2 </dev/null | grep ALPN
openssl s_client -connect host:443 -status </dev/null | grep -A2 OCSP
openssl x509 -in cert.pem -noout -text
openssl x509 -in cert.pem -noout -dates -subject -issuer
openssl verify -CAfile ca.crt cert.pem
```

**۴) `tcpdump` — لایه‌ی شبکه**

```bash
sudo tcpdump -i any -nn port 80 -c 20
sudo tcpdump -i any -nn -A port 80 -c 5
sudo tcpdump -i any -nn 'tcp[tcpflags] & (tcp-syn|tcp-rst) != 0' -c 20
sudo tcpdump -i any -nn -w /tmp/cap.pcap port 443
```

`-A` = ASCII (برای HTTP)، `-w` = فایل برای Wireshark.

**۵) `strace` — لایه‌ی syscall**

```bash
sudo strace -f -p PID -e trace=openat,stat,connect 2>&1 | tail -20
sudo strace -f -e trace=network -p PID
```

**۶) `ss` — سوکت‌ها**

```bash
ss -ltnp
ss -tan state established
ss -s
ss -tan '( dport = :443 )'
```

**۷) `lsof` — فایل‌های باز**

```bash
sudo lsof -i :80
sudo lsof -p PID | grep log
```

**۸) `journalctl`**

```bash
sudo journalctl -u nginx --since "10 min ago"
sudo journalctl -u nginx -f
sudo journalctl -u nginx -p err
```

**۹) `dmesg`**

```bash
sudo dmesg | tail -20
sudo dmesg | grep -i 'oom\|segfault'
```

**۱۰) `mtr`**

```bash
mtr -rw -c 10 lab.example.com
```

ترکیب `ping` و `traceroute`، برای تشخیص مسیر.

### مثال‌های عملی

**Lab Scenario: یک حادثه، چهار ابزار**

سناریو: `502` روی `/api/`.

```bash
# ۱) curl — کد و زمان
curl -sS -o /dev/null -w 'code=%{http_code} ttfb=%{time_starttransfer} total=%{time_total}\n' \
  http://localhost:8080/api/health
# code=502 ttfb=0.001 total=0.001   ← سریع، یعنی اتصال نشد
```

```bash
# ۲) ss — upstream
ss -ltnp | grep 8000
# (خالی) → اپ گوش نمی‌دهد
```

```bash
# ۳) journalctl — اپ
sudo journalctl -u lab-api -n 20 --no-pager
# Failed to bind: Address already in use
```

```bash
# ۴) lsof — چه چیزی پورت را گرفته
sudo lsof -i :8000
# node 1234 ... TCP *:8000 (LISTEN)
```

**`strace` برای hang:**

```bash
PID=$(pgrep -f 'nginx: worker' | head -1)
sudo timeout 5 strace -f -p "$PID" -e trace=network,openat 2>&1 | tail -20
# connect(3, {sa_family=AF_INET, sin_port=htons(8000), ...}) = -1 ECONNREFUSED
```

**`tcpdump` برای timeout:**

```bash
sudo tcpdump -i any -nn 'host 203.0.113.45 and port 443' -c 20
# SYN → (بدون SYN-ACK) → SYN → ...
# یعنی drop در مسیر
```

**`curl` مرحله‌به‌مرحله:**

```bash
# DNS
dig +short lab.example.com

# TCP
nc -zv lab.example.com 443

# TLS
openssl s_client -connect lab.example.com:443 -servername lab.example.com </dev/null | head -5

# HTTP
curl -sSI https://lab.example.com/
```

**`mtr` برای مسیر:**

```bash
mtr -rw -c 10 lab.example.com
# Loss% در hop میانی → مشکل شبکه
```

**`ss` برای اتصال زیاد:**

```bash
ss -s
# TCP: 1234 (estab 100, closed 1100, orphaned 0, timewait 1000)

ss -tan state time-wait | wc -l
ss -tan state established '( sport = :443 )' | wc -l
```

**`lsof` برای فایل قفل‌شده:**

```bash
sudo lsof /var/log/nginx/access.log
# nginx 1234 www-data 5w REG ...
```

**اسکریپت تشخیص کامل:**

```bash
#!/usr/bin/env bash
set -uo pipefail
URL=${1:?url}
HOST=$(echo "$URL" | sed -E 's#https?://([^/:]+).*#\1#')

echo "=== DNS"
dig +short "$HOST" || echo "  ✗"

echo "=== TCP"
nc -zv "$HOST" 443 2>&1 | tail -1

echo "=== TLS"
echo | openssl s_client -connect "$HOST:443" -servername "$HOST" 2>&1 \
  | grep -E 'Verify return code|Protocol|Cipher' | head -3

echo "=== HTTP"
curl -sS -o /dev/null -w 'code=%{http_code} dns=%{time_namelookup} tls=%{time_appconnect} ttfb=%{time_starttransfer}\n' "$URL"

echo "=== HEADERS"
curl -sSI "$URL" | head -12

echo "=== NGINX ERRORS"
sudo tail -3 /var/log/nginx/error.log 2>/dev/null || true
```

### مقایسه و الگوهای مختلف

**چهار ابزار، چهار لایه**

| ابزار | لایه | چه می‌گوید |
|---|---|---|
| `curl` | HTTP | کد، زمان، هدر |
| `openssl` | TLS | زنجیره، cipher |
| `tcpdump` | TCP/IP | بسته‌ها |
| `strace` | syscall | چه فراخوانی‌ای شکست |

**`tcpdump` در برابر `Wireshark`**

| | `tcpdump` | Wireshark |
|---|---|---|
| سرور | ✅ | ⚠️ (GUI) |
| تحلیل | ⚠️ | ✅✅ |
| توصیه | ✅ capture | ✅ analyze |

**`strace` در برابر `ltrace`**

| | `strace` | `ltrace` |
|---|---|---|
| syscall | ✅ | ❌ |
| کتابخانه | ❌ | ✅ |
| توصیه | ✅ | نادر |

**دیدگاه «`strace` برای تولید خطرناک است»**: سربار دارد (۲–۵×)، ولی برای چند ثانیه بی‌خطر است. `timeout 5 strace ...`.

### اشکال شایع

**۱) `curl` بدون `-sS`**
خطا پنهان/نمایش اضافه. `-sS`.

**۲) `curl -I` روی `HEAD` نامعتبر**
بعضی سرورها `HEAD` را پشتیبانی نمی‌کنند. `-X GET -o /dev/null -D -`.

**۳) `openssl` بدون `-servername`**
گواهی اشتباه.

**۴) `tcpdump` بدون `-nn`**
DNS معکوس کند.

**۵) `tcpdump` بدون فیلتر**
فایل عظیم.

**۶) `strace` بدون `-f`**
فرزندها دیده نمی‌شوند.

**۷) `ss` بدون `-p`**
پروسه معلوم نیست.

**۸) `journalctl` بدون `-u`**
همه‌ی لاگ. `-u`.

**۹) `mtr` بدون `-r`**
interactive. `-rw`.

**۱۰) `lsof` بدون `sudo`**
فایل‌های پروسه‌های دیگر. `sudo`.

### نکات مهم (Tips & Tricks)

- **`curl -w`** برای زمان‌بندی.
- **`openssl -servername`**.
- **`tcpdump -nn`**.
- **`strace -f`**.
- **`ss -ltnp`**.
- **`journalctl -u`**.
- **`mtr -rw`**.

### خلاصه فصل

- `curl` برای HTTP، `openssl` برای TLS.
- `tcpdump` برای بسته، `strace` برای syscall.
- `-servername` و `-nn` را فراموش نکن.
- `strace` با `timeout`.
- `ss`/`lsof` برای منابع.
- `mtr` برای مسیر.

**سؤال مروری:** `502` می‌گیری و `curl` مستقیم به upstream هم `refused` می‌دهد. کدام ابزار می‌گوید چرا؟

---

## منابع و مراجع

**مستندات رسمی**
- [Nginx — Debugging](https://nginx.org/en/docs/debugging_log.html)
- [Apache — Troubleshooting](https://httpd.apache.org/docs/2.4/misc/troubleshooting.html)
- [curl — man page](https://curl.se/docs/manpage.html)
- [openssl s_client](https://www.openssl.org/docs/man3.0/man1/openssl-s_client.html)
- [tcpdump](https://www.tcpdump.org/manpages/tcpdump.1.html)
- [strace](https://strace.io/)

**آموزش**
- [Debugging 502/504](https://www.nginx.com/resources/wiki/start/topics/tutorials/debugging/)
- [TLS troubleshooting](https://www.feistyduck.com/library/openssl-cookbook/)
- [Linux performance tools](https://www.brendangregg.com/linuxperf.html)

**ابزار**
- [testssl.sh](https://testssl.sh/)
- [mtr](https://github.com/traviscross/mtr)
- [Wireshark](https://www.wireshark.org/)

---

**بعدی:** [`18-Complete-Lab-Project.md`](18-Complete-Lab-Project.md) — پروژه‌ی کامل
