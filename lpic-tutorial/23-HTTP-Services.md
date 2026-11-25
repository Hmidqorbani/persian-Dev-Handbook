# فایل ۲۳ — سرویس‌های HTTP

**مدت یادگیری**: ۸ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: فایل ۱۳ (شبکه)، فایل ۲۲ (DNS)

---

## فهرست محتوا

- [فصل ۸۰: پیکربندی پایهٔ Apache](#فصل-۸۰-پیکربندی-پایهٔ-apache)
- [فصل ۸۱: Apache و HTTPS](#فصل-۸۱-apache-و-https)
- [فصل ۸۲: Squid به‌عنوان پروکسی کش](#فصل-۸۲-squid-بهعنوان-پروکسی-کش)
- [فصل ۸۳: Nginx به‌عنوان وب‌سرور و Reverse Proxy](#فصل-۸۳-nginx-بهعنوان-وبسرور-و-reverse-proxy)

---

## فصل ۸۰: پیکربندی پایهٔ Apache

### مقدمه

**Apache HTTP Server** یکی از محبوب‌ترین وب‌سرورهای جهان است. تصور کنید می‌خواهید یک وب‌سایت یا API را روی سرور میزبانی کنید — Apache گزینه‌ای قدرتمند است.

در این فصل نصب، پیکربندی و مدیریت آن را یاد می‌گیرید.

### مفاهیم اساسی

**ساختار پیکربندی:**

| مسیر | محتوا | توزیع |
|---|---|---|
| `/etc/apache2/` | پیکربندی | Debian/Ubuntu |
| `/etc/httpd/` | پیکربندی | RHEL/Rocky |
| `apache2.conf`/`httpd.conf` | پیکربندی اصلی | — |
| `sites-available/` | سایت‌های موجود | Debian |
| `sites-enabled/` | سایت‌های فعال | Debian |
| `mods-available/` | ماژول‌ها | Debian |

**مفاهیم کلیدی:**

- **Virtual Host:** میزبانی چند سایت.
- **DocumentRoot:** پوشهٔ ریشهٔ سایت.
- **Module:** افزونه‌ها.
- **.htaccess:** پیکربندی سطح پوشه.
- **MPM:** Multi-Processing Module.

### مثال‌های عملی

**نصب و مدیریت:**

```bash
# Debian/Ubuntu
sudo apt install apache2
sudo systemctl enable --now apache2

# RHEL/Rocky
sudo dnf install httpd
sudo systemctl enable --now httpd

# تست پیکربندی
sudo apache2ctl configtest    # یا httpd -t

# ماژول‌ها
sudo a2enmod rewrite
sudo a2dismod status
sudo a2enmod ssl
apache2ctl -M                 # فهرست ماژول‌ها
```

**Virtual Host:**

```apache
# /etc/apache2/sites-available/example.conf
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example

    <Directory /var/www/example>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/example-error.log
    CustomLog ${APACHE_LOG_DIR}/example-access.log combined
</VirtualHost>
```

```bash
sudo a2ensite example
sudo systemctl reload apache2
```

**فعال/غیرفعال سایت:**

```bash
sudo a2ensite example
sudo a2dissite default
sudo apache2ctl -S        # فهرست vhostها
```

### مقایسه و الگوهای مختلف

**دستورهای مدیریت:**

| کار | Debian | RHEL |
|---|---|---|
| تست | `apache2ctl configtest` | `httpd -t` |
| فعال سایت | `a2ensite` | ویرایش مستقیم |
| فعال ماژول | `a2enmod` | ویرایش مستقیم |
| لاگ | `/var/log/apache2/` | `/var/log/httpd/` |
| سرویس | `apache2` | `httpd` |

**MPM modules:**

| MPM | مدل | مناسب |
|---|---|---|
| `prefork` | فرآیند | سازگاری |
| `worker` | فرآیند+thread | کارایی |
| `event` | بهینه (پیش‌فرض) | مدرن |

**Apache در برابر Nginx:**

| ویژگی | Apache | Nginx |
|---|---|---|
| معماری | process/thread | event-driven |
| فایل استاتیک | خوب | سریع‌تر |
| .htaccess | بله | خیر |
| ماژول داینامیک | کامل | محدود |

### اشکال شایع

**اشکال ۱: خطای syntax.**
`configtest` را فراموش نکنید. راه‌حل: تست.

**اشکال ۲: مجوز پوشه.**
DocumentRoot باید خوانا باشد. راه‌حل: مجوز.

**اشکال ۳: `.htaccess` غیرفعال.**
`AllowOverride` لازم است. راه‌حل: `AllowOverride All`.

**اشکال ۴: پورت اشغال.**
سرور دیگری روی ۸۰ است. راه‌حل: `ss -tlnp`.

**اشکال ۵: SELinux در RHEL.**
Apache نمی‌تواند فایل بخواند. راه‌حل: context.

**اشکال ۶: فایروال پورت ۸۰/۴۴۳.**
سایت در دسترس نیست. راه‌حل: باز کردن.

### نکات مهم (Tips & Tricks)

- **همیشه `configtest` قبل از reload.**
- **`a2enmod`/`a2ensite` کار را ساده می‌کند (Debian).**
- **لاگ‌ها را با `logrotate` مدیریت کنید.**
- **`Options -Indexes` جلوی فهرست‌شدن پوشه را می‌گیرد.**
- **برای کارایی از MPM `event` استفاده کنید.**
- **`apache2ctl -S` همهٔ vhostها را نشان می‌دهد.**
- **`apachectl -M` ماژول‌های فعال.**
- **`.htaccess` برای پیکربندی سطح پوشه.**
- **`mod_rewrite` برای URL rewriting.**
- **`mod_proxy` برای reverse proxy.**
- **`Redirect` برای تغییر مسیر.**
- **`Alias` برای نگاشت مسیر.**

### خلاصه فصل

Apache محبوب‌ترین وب‌سرور است. پیکربندی در `/etc/apache2/` (Debian) یا `/etc/httpd/` (RHEL) است. Virtual Host میزبانی چند سایت را ممکن می‌کند. با `a2ensite`/`a2enmod` (Debian) مدیریت کنید. همیشه `configtest` قبل از reload بزنید. MPM `event` کارایی بهتری می‌دهد.

---

## فصل ۸۱: Apache و HTTPS

### مقدمه

**HTTPS** نسخهٔ امن HTTP است که با **TLS/SSL** رمزنگاری می‌شود. تصور کنید کاربری اطلاعات حساس (رمز، کارت بانکی) وارد می‌کند — بدون HTTPS، این اطلاعات متن‌رو می‌روند.

امروزه تقریباً همهٔ سایت‌ها HTTPS دارند و در آزمون می‌آید.

### مفاهیم اساسی

**مفاهیم کلیدی:**

- **SSL/TLS:** پروتکل رمزنگاری.
- **Certificate:** گواهی دیجیتال.
- **CA:** مرجع صدور گواهی.
- **Self-signed:** گواهی خودامضا.
- **Let's Encrypt:** CA رایگان.
- **SNI:** Server Name Indication.

**ماژول‌های Apache:** `mod_ssl`, `mod_rewrite`.

**پورت‌ها:** ۴۴۳ (HTTPS).

### مثال‌های عملی

**فعال‌سازی SSL:**

```bash
sudo a2enmod ssl
sudo a2enmod rewrite
sudo systemctl restart apache2
```

**ساخت گواهی خودامضا:**

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/example.key \
  -out /etc/ssl/certs/example.crt \
  -subj "/CN=example.com"
```

**Virtual Host HTTPS:**

```apache
<VirtualHost *:443>
    ServerName example.com
    DocumentRoot /var/www/example

    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/example.crt
    SSLCertificateKeyFile /etc/ssl/private/example.key
    # SSLCertificateChainFile /etc/ssl/certs/chain.crt

    # پروتکل‌های امن
    SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
    SSLCipherSuite HIGH:!aNULL:!MD5

    # HSTS
    Header always set Strict-Transport-Security "max-age=31536000"
</VirtualHost>
```

**ریدایرکت HTTP به HTTPS:**

```apache
<VirtualHost *:80>
    ServerName example.com
    Redirect permanent / https://example.com/
</VirtualHost>
```

**Let's Encrypt (Certbot):**

```bash
sudo apt install certbot python3-certbot-apache
sudo certbot --apache -d example.com -d www.example.com
sudo certbot renew --dry-run
sudo certbot certificates
```

### مقایسه و الگوهای مختلف

**انواع گواهی:**

| نوع | اعتماد | کاربرد |
|---|---|---|
| Self-signed | ندارد | تست/داخلی |
| Let's Encrypt | دارد | سایت عمومی (رایگان) |
| تجاری | دارد | سازمانی |
| Wildcard | دارد | چند زیردامنه |

**TLS versions:**

| نسخه | وضعیت |
|---|---|
| SSLv3 | ناامن (غیرفعال) |
| TLS 1.0/1.1 | منسوخ |
| TLS 1.2 | امن |
| TLS 1.3 | مدرن |

**روش‌های دریافت گواهی:**

| روش | هزینه | اعتماد |
|---|---|---|
| Let's Encrypt | رایگان | دارد |
| تجاری | پرداخت | دارد |
| Self-signed | رایگان | ندارد |

### اشکال شایع

**اشکال ۱: گواهی منقضی.**
سایت هشدار می‌دهد؛ تمدید کنید. راه‌حل: تمدید خودکار.

**اشکال ۲: زنجیرهٔ ناقص.**
`SSLCertificateChainFile` لازم است. راه‌حل: chain.

**اشکال ۳: SSLv3 فعال.**
آسیب‌پذیر (POODLE). راه‌حل: غیرفعال.

**اشکال ۴: فراموش کردن ریدایرکت.**
کاربران HTTP بدون رمز می‌مانند. راه‌حل: Redirect.

**اشکال ۵: کلید خصوصی ناامن.**
مجوز ۶۰۰ لازم است. راه‌حل: مجوز.

**اشکال ۶: mismatch نام.**
گواهی برای دامنهٔ دیگر. راه‌حل: CN/SAN درست.

### نکات مهم (Tips & Tricks)

- **از Let's Encrypt برای گواهی رایگان.**
- **`certbot renew` را خودکار کنید (cron/systemd timer).**
- **TLS 1.2+ فقط.**
- **HSTS را فعال کنید.**
- **`openssl s_client -connect example.com:443` تست.**
- **`ssllabs.com` تست کیفیت SSL.**
- **`SSLCertificateChainFile` برای زنجیره.**
- **`certbot --nginx` برای Nginx.**
- **SAN برای چند دامنه.**
- **OCSP Stapling برای کارایی.**
- **`ssl_dhparam` برای DH params.**
- **`apachectl configtest` بعد از تغییر.**

### خلاصه فصل

HTTPS با TLS گواهی را رمزنگاری می‌کند. `mod_ssl` را فعال کنید و Virtual Host روی پورت ۴۴۳ بسازید. Let's Encrypt گواهی رایگان می‌دهد (Certbot). TLS 1.2+ استفاده کنید و HTTP را به HTTPS ریدایرکت کنید. گواهی را خودکار تمدید کنید. HSTS امنیت را بالا می‌برد.

---

## فصل ۸۲: Squid به‌عنوان پروکسی کش

### مقدمه

**Squid** یک **پروکسی کش** است که درخواست‌ها را ذخیره می‌کند تا سرعت را بالا ببرد و پهنای باند را کم کند. تصور کنید یک شرکت با ۱۰۰ کارمند دارید — با Squid، یک بار دانلود کافی است برای همه.

در این فصل پیکربندی آن را یاد می‌گیرید.

### مفاهیم اساسی

**پروکسی کش چیست؟** واسطه‌ای که درخواست‌های کاربران را می‌گیرد، پاسخ را کش می‌کند و در درخواست‌های بعدی سریع‌تر پاسخ می‌دهد.

**کاربردها:**

- کاهش پهنای باند.
- افزایش سرعت.
- فیلترینگ.
- لاگ دسترسی.
- ناشناسی (anonymity).

**فایل‌های مهم:**

- `/etc/squid/squid.conf` — پیکربندی.
- `/var/spool/squid/` — کش.
- `/var/log/squid/` — لاگ.

### مثال‌های عملی

**نصب و پیکربندی:**

```bash
sudo apt install squid
sudo systemctl enable --now squid

# پیکربندی
sudo vim /etc/squid/squid.conf

# بررسی
sudo squid -k parse
sudo systemctl reload squid
```

**پیکربندی پایه:**

```
http_port 3128

# ACL برای شبکهٔ داخلی
acl localnet src 192.168.1.0/24
http_access allow localnet
http_access deny all

# کش
cache_dir ufs /var/spool/squid 1000 16 256
cache_mem 256 MB

# لاگ
access_log /var/log/squid/access.log
```

**فیلترینگ:**

```
# بلاک کردن سایت
acl blocked_sites dstdomain .facebook.com
http_access deny blocked_sites

# بلاک کلمات
acl blocked_words url_regex -i sex
http_access deny blocked_words

# بلاک پسوند
acl blocked_ext urlpath_regex \.(mp3|mp4|avi)$
http_access deny blocked_ext

# بلاک IP
acl blocked_ips dst 10.0.0.0/8
http_access deny blocked_ips
```

**احراز هویت:**

```
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
acl authenticated proxy_auth REQUIRED
http_access allow authenticated
```

### مقایسه و الگوهای مختلف

**انواع پروکسی:**

| نوع | توضیح |
|---|---|
| **Forward** | برای کاربران داخلی |
| **Reverse** | جلوی سرورها |
| **Transparent** | بدون تنظیم کلاینت |

**ابزارهای مرتبط:**

| ابزار | کاربرد |
|---|---|
| `squidclient` | تست |
| `SquidGuard` | فیلترینگ پیشرفته |
| `DansGuardian` | فیلتر محتوا |
| `e2guardian` | فیلتر محتوا |

**ACL types:**

| نوع | کاربرد |
|---|---|
| `src` | آدرس منبع |
| `dst` | آدرس مقصد |
| `dstdomain` | دامنه |
| `url_regex` | URL |
| `urlpath_regex` | مسیر URL |
| `port` | پورت |
| `time` | زمان |
| `proxy_auth` | احراز |

### اشکال شایع

**اشکال ۱: deny all برای همه.**
هیچ‌کس دسترسی ندارد. راه‌حل: ترتیب ACL.

**اشکال ۲: کش در حافظهٔ کم.**
کندی سیستم. راه‌حل: `cache_mem` مناسب.

**اشکال ۳: نبود acl.**
پیکربندی خطا می‌دهد. راه‌حل: تعریف acl.

**اشکال ۴: پورت اشغال.**
پورت ۳۱۲۸ در استفاده است. راه‌حل: پورت دیگر.

**اشکال ۵: نبود `squid -z`.**
پوشهٔ کش ساخته نشده. راه‌حل: `squid -z`.

**اشکال ۶: ترتیب ACL اشتباه.**
قوانین به ترتیب اعمال می‌شوند. راه‌حل: ترتیب درست.

### نکات مهم (Tips & Tricks)

- **`squid -k parse` پیکربندی را بررسی می‌کند.**
- **`squid -k reconfigure` بازخوانی بدون restart.**
- **`acl` و `http_access` به ترتیب اعمال می‌شوند.**
- **برای کش مؤثر، `cache_dir` کافی بدهید.**
- **`access_log` برای تحلیل ترافیک.**
- **`refresh_pattern` رفتار کش را تنظیم می‌کند.**
- **`squid -z` پوشهٔ کش را می‌سازد.**
- **`cache_effective_user` کاربر اجرا.**
- **`http_access deny all` در انتها (پیش‌فرض امن).**
- **`squidclient -h host -p 3128 url` تست.**
- **`acl` را قبل از `http_access` تعریف کنید.**
- **برای شفاف، `http_port 3128 intercept`.**

### خلاصه فصل

Squid پروکسی کش است که سرعت را بالا و پهنای باند را کم می‌کند. پیکربندی با `acl` و `http_access` انجام می‌شود. `http_port`, `cache_dir`, `cache_mem` تنظیمات کلیدی‌اند. با `squid -k parse` بررسی و با `-k reconfigure` بازخوانی کنید. برای فیلترینگ از acl استفاده کنید. ترتیب ACL مهم است.

---

## فصل ۸۳: Nginx به‌عنوان وب‌سرور و Reverse Proxy

### مقدمه

**Nginx** یک وب‌سرور سریع و سبک است که به‌عنوان **Reverse Proxy** و **Load Balancer** هم عالی عمل می‌کند. امروزه بسیار محبوب است.

تصور کنید چند سرور backend دارید و می‌خواهید بار را توزیع کنید — Nginx این کار را عالی انجام می‌دهد.

### مفاهیم اساسی

**نقش‌ها:**

- **Web Server:** سرو فایل‌های استاتیک.
- **Reverse Proxy:** ارجاع به سرورهای backend.
- **Load Balancer:** توزیع بار.
- **TLS Termination:** خاتمهٔ SSL.
- **Caching:** کش پاسخ‌ها.

**فایل‌های مهم:**

- `/etc/nginx/nginx.conf` — پیکربندی اصلی.
- `/etc/nginx/sites-available/` — سایت‌ها (Debian).
- `/etc/nginx/conf.d/` — پیکربندی‌ها (RHEL).

### مثال‌های عملی

**نصب:**

```bash
sudo apt install nginx    # Debian/Ubuntu
sudo dnf install nginx    # RHEL/Rocky
sudo systemctl enable --now nginx
```

**وب‌سرور ساده:**

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/example;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

**Reverse Proxy:**

```nginx
server {
    listen 80;
    server_name app.example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Load Balancer:**

```nginx
upstream backend {
    least_conn;
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080 backup;
    keepalive 32;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

**HTTPS:**

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/ssl/certs/example.crt;
    ssl_certificate_key /etc/ssl/private/example.key;
    ssl_protocols TLSv1.2 TLSv1.3;

    location / {
        root /var/www/example;
    }
}
```

**بررسی:**

```bash
sudo nginx -t
sudo nginx -T          # نمایش پیکربندی کامل
sudo systemctl reload nginx
```

### مقایسه و الگوهای مختلف

**Nginx در برابر Apache:**

| ویژگی | Nginx | Apache |
|---|---|---|
| معماری | event-driven | process/thread |
| فایل استاتیک | سریع‌تر | خوب |
| .htaccess | ندارد | دارد |
| Reverse Proxy | عالی | خوب |
| ماژول داینامیک | محدود | کامل |
| حافظه | کم | زیاد |

**الگوریتم‌های Load Balancing:**

| الگوریتم | توضیح |
|---|---|
| round-robin | چرخشی (پیش‌فرض) |
| least_conn | کمترین اتصال |
| ip_hash | بر اساس IP |
| weight | وزنی |
| random | تصادفی |

**گزینه‌های proxy:**

| گزینه | کار |
|---|---|
| `proxy_pass` | مقصد |
| `proxy_set_header` | هدر |
| `proxy_cache` | کش |
| `proxy_buffering` | بافر |
| `proxy_timeout` | timeout |

### اشکال شایع

**اشکال ۱: خطای syntax.**
`nginx -t` بگیرید. راه‌حل: تست.

**اشکال ۲: نبود هدرهای proxy.**
backend IP واقعی را نمی‌بیند. راه‌حل: `proxy_set_header`.

**اشکال ۳: اشتباه upstream.**
۵۰۲ Bad Gateway. راه‌حل: بررسی backend.

**اشکال ۴: webroot اشتباه.**
۴۰۴. راه‌حل: `root` درست.

**اشکال ۵: SELinux.**
Nginx نمی‌تواند به backend وصل شود. راه‌حل: `setsebool`.

**اشکال ۶: نبود keepalive.**
کارایی پایین. راه‌حل: `keepalive`.

### نکات مهم (Tips & Tricks)

- **همیشه `nginx -t` قبل از reload.**
- **`proxy_set_header` برای IP واقعی ضروری است.**
- **`least_conn` برای بار نامتوازن بهتر.**
- **`gzip on` برای فشرده‌سازی.**
- **`keepalive` برای کارایی upstream.**
- **Nginx را جلوی Apache/Node بگذارید (reverse proxy).**
- **`nginx -T` پیکربندی کامل.**
- **`location` با regex.**
- **`try_files` برای SPA.**
- **`limit_req` برای rate limiting.**
- **`proxy_cache` برای کش.**
- **`upstream` با `backup` برای failover.**

### خلاصه فصل

Nginx وب‌سرور سریع و reverse proxy قدرتمند است. با `server` و `location` پیکربندی می‌شود. `proxy_pass` برای reverse proxy و `upstream` برای load balancing به‌کار می‌رود. `nginx -t` قبل از reload. Nginx برای فایل استاتیک و reverse proxy از Apache سریع‌تر است. `proxy_set_header` برای IP واقعی ضروری است.

---

## منابع و مراجع

- man pages: `man apache2`, `man nginx`, `man squid`, `man openssl`, `man certbot`
- Apache HTTP Server Docs: https://httpd.apache.org/docs/
- Nginx Documentation: https://nginx.org/en/docs/
- Squid Wiki: https://wiki.squid-cache.org
- Let's Encrypt: https://letsencrypt.org

**بعدی:** [فایل ۲۴ — اشتراک‌گذاری فایل](24-File-Sharing.md)
