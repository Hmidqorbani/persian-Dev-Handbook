# بخش ۰۵: Nginx — نصب و پیکربندی

**مدت یادگیری**: ۵ تا ۶ ساعت
**سطح سختی**: مبتدی
**پیش‌نیاز**: بخش ۰۳ (معماری Nginx، event loop)

---

## فهرست محتوا

- [فصل ۱۸: نصب Nginx روی لینوکس و ویندوز](#فصل-۱۸)
- [فصل ۱۹: ساختار `nginx.conf` و contextها](#فصل-۱۹)
- [فصل ۲۰: `server` block و میزبانی چند سایت](#فصل-۲۰)
- [فصل ۲۱: `location` — تطبیق، اولویت و تله‌ها](#فصل-۲۱)
- [فصل ۲۲: `root`/`alias`، `try_files`، `internal` و `error_page`](#فصل-۲۲)

---

## فصل ۱۸: نصب Nginx روی لینوکس و ویندوز

### مقدمه

Nginx در مخازن رسمی اکثر توزیع‌ها هست، ولی نسخه‌ی مخزن معمولاً قدیمی است و ماژول‌های جدید (مثل HTTP/3) را ندارد. برای تولید، مخزن رسمی Nginx بهتر است. این فصل، هر سه مسیر را می‌گوید: مخزن توزیع، مخزن رسمی، و ویندوز.

### مفاهیم اساسی

**۱) سه منبع نصب**

| منبع | نسخه | مزیت | عیب |
|---|---|---|---|
| مخزن توزیع (`apt install nginx`) | قدیمی‌تر | به‌روزرسانی امنیتی توزیع | بدون ماژول جدید |
| مخزن رسمی `nginx.org` | جدید | آخرین نسخه و ماژول‌ها | باید مخزن اضافه کنی |
| کامپایل از سورس | دلخواه | کنترل کامل | نگهداری دستی |
| `nginx-extras` (Debian) | جدیدتر | ماژول‌های اضافه | حجم بیشتر |

**۲) مسیرها**

| مورد | Debian/Ubuntu | RHEL |
|---|---|---|
| config اصلی | `/etc/nginx/nginx.conf` | `/etc/nginx/nginx.conf` |
| سایت‌ها | `/etc/nginx/sites-available/` | `/etc/nginx/conf.d/` |
| لاگ | `/var/log/nginx/` | `/var/log/nginx/` |
| `DocumentRoot` | `/var/www/html` | `/usr/share/nginx/html` |
| کاربر | `www-data` | `nginx` |

**۳) سرویس**

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl reload nginx     # بدون قطع
sudo systemctl restart nginx    # قطع کوتاه
sudo nginx -t                   # تست config
sudo nginx -T                   # تست + چاپ کل config
```

**۴) نسخه و ماژول‌ها**

```bash
nginx -v                 # نسخه
nginx -V 2>&1 | head -3  # نسخه + configure args
nginx -V 2>&1 | tr ' ' '\n' | grep -E 'module|http_v[23]' | sort
```

**۵) `nginx -t` چه چک می‌کند**

- سینتکس
- وجود فایل‌های `ssl_certificate`
- دسترسی به `pid` و لاگ
- `include`های ناموجود

**۶) فایروال**

```bash
# Ubuntu
sudo ufw allow 'Nginx Full'    # 80 + 443
sudo ufw allow 'Nginx HTTP'

# RHEL
sudo firewall-cmd --permanent --add-service={http,https}
sudo firewall-cmd --reload
```

**۷) SELinux (RHEL)**

```bash
# اجازه‌ی proxy به شبکه
sudo setsebool -P httpd_can_network_connect 1

# پوشه‌ی سفارشی
sudo semanage fcontext -a -t httpd_sys_content_t "/srv/lab(/.*)?"
sudo restorecon -Rv /srv/lab

# پورت غیراستاندارد
sudo semanage port -a -t http_port_t -p tcp 8080
```

**۸) ویندوز**

Nginx روی ویندوز اجرا می‌شود، ولی:

- فقط یک worker مؤثر (بدون `epoll`، از `select` استفاده می‌کند).
- `reuseport` ندارد.
- کارایی پایین‌تر، مخصوصاً با اتصال زیاد.
- برای توسعه یا پروکسی سبک مناسب است، نه تولید سنگین.

```powershell
# دانلود
Invoke-WebRequest -Uri "https://nginx.org/download/nginx-1.26.2.zip" -OutFile "$env:TEMP\nginx.zip"
Expand-Archive "$env:TEMP\nginx.zip" -DestinationPath "C:\"
Rename-Item "C:\nginx-1.26.2" "C:\nginx"

# اجرا
cd C:\nginx
.\nginx.exe
.\nginx.exe -s reload
.\nginx.exe -s stop

# تست
Invoke-WebRequest http://localhost/ -UseBasicParsing | Select-Object StatusCode
```

**۹) نصب به‌عنوان سرویس ویندوز**

```powershell
# با NSSM یا WinSW
# مثال با nssm:
nssm install nginx "C:\nginx\nginx.exe"
nssm set nginx AppDirectory "C:\nginx"
nssm start nginx
```

**۱۰) `nginx.conf` پیش‌فرض Debian**

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
}

http {
    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    ssl_protocols TLSv1.2 TLSv1.3;
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    gzip on;
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

### مثال‌های عملی

**Lab Scenario: نصب از مخزن رسمی روی Ubuntu 24.04**

```bash
# ۱) پیش‌نیاز
sudo apt install -y curl gnupg2 ca-certificates lsb-release ubuntu-keyring

# ۲) کلید و مخزن
curl -fsSL https://nginx.org/keys/nginx_signing.key \
  | sudo gpg --dearmor -o /usr/share/keyrings/nginx-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/ubuntu $(lsb_release -cs) nginx" \
  | sudo tee /etc/apt/sources.list.d/nginx.list

# ۳) اولویت مخزن رسمی
sudo tee /etc/apt/preferences.d/99nginx >/dev/null <<'EOF'
Package: *
Pin: origin nginx.org
Pin-Priority: 900

Package: nginx
Pin-Priority: 900
EOF

# ۴) نصب
sudo apt update
sudo apt install -y nginx

# ۵) بررسی
nginx -v
sudo systemctl enable --now nginx
sudo systemctl status nginx --no-pager
curl -sSI http://localhost/
```

**نصب روی Rocky Linux 9:**

```bash
sudo tee /etc/yum.repos.d/nginx.repo >/dev/null <<'EOF'
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
EOF

sudo dnf install -y nginx
sudo systemctl enable --now nginx
sudo firewall-cmd --permanent --add-service=http --add-service=https
sudo firewall-cmd --reload
```

**پیکربندی پایه‌ی تولید:**

```nginx
# /etc/nginx/nginx.conf
user  www-data;
worker_processes  auto;
worker_rlimit_nofile 65535;

error_log  /var/log/nginx/error.log warn;
pid        /run/nginx.pid;

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
                    '"$http_user_agent" "$http_x_forwarded_for" '
                    'rt=$request_time urt=$upstream_response_time';

    access_log  /var/log/nginx/access.log main;

    sendfile           on;
    tcp_nopush         on;
    tcp_nodelay        on;
    keepalive_timeout  65;
    server_tokens      off;
    client_max_body_size 16m;
    client_body_timeout  12s;
    client_header_timeout 12s;

    gzip              on;
    gzip_vary         on;
    gzip_min_length   1024;
    gzip_comp_level   6;
    gzip_types        text/plain text/css text/xml application/json
                      application/javascript application/xml image/svg+xml;

    include /etc/nginx/conf.d/*.conf;
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx
```

**تست:**

```bash
curl -sSI http://localhost/
# Server: nginx   (بدون نسخه، به‌خاطر server_tokens off)
```

**چک‌لیست بعد از نصب:**

```bash
# ۱) نسخه و ماژول
nginx -V 2>&1 | tr ' ' '\n' | grep -c module

# ۲) config سالم
nginx -t

# ۳) سرویس فعال و enabled
systemctl is-active nginx && systemctl is-enabled nginx

# ۴) پورت
ss -ltnp | grep nginx

# ۵) صفحه‌ی پیش‌فرض
curl -sS http://localhost/ | head -5

# ۶) لاگ
sudo tail -5 /var/log/nginx/access.log
```

### مقایسه و الگوهای مختلف

**مخزن توزیع در برابر مخزن رسمی**

| | توزیع | رسمی |
|---|---|---|
| نسخه | قدیمی‌تر | جدید |
| HTTP/3 | ⚠️ | ✅ (1.25+) |
| پشتیبانی امنیتی | ✅ توزیع | ✅ nginx.org |
| تعارض | ❌ | ⚠️ (pin لازم) |

**`nginx` در برابر `nginx-extras`**

| | `nginx` | `nginx-extras` |
|---|---|---|
| ماژول | پایه | + `auth_pam`، `geoip`، `xslt` |
| حجم | ✅ | ⚠️ |
| مناسب | اکثر | نیاز خاص |

**Nginx روی ویندوز در برابر IIS**

| | Nginx (Win) | IIS |
|---|---|---|
| worker مؤثر | ۱ | چند |
| کارایی | ⚠️ | ✅ |
| config | ✅ | ⚠️ |
| توصیه | dev/proxy سبک | ✅ تولید ویندوز |

**دیدگاه «Nginx روی ویندوز برای تولید»**: برای ترافیک کم و پروکسی ساده، کار می‌کند. برای بار جدی، IIS یا یک VM لینوکس.

### اشکال شایع

**۱) `nginx: [emerg] unknown directive "http2"`**
نسخه‌ی قدیمی. `listen 443 ssl http2;` (سبک قدیمی) یا ارتقا به 1.25.1+.

**۲) `403 Forbidden` بعد از تغییر `root`**
مجوز فایل یا SELinux. `ls -lZ`.

**۳) پورت ۸۰ اشغال**
Apache یا IIS. `ss -ltnp`.

**۴) `nginx -t` موفق، ولی سایت قدیمی**
`reload` نزده‌ای، یا `include` فایل اشتباه. `nginx -T`.

**۵) `worker_connections are not enough`**
در `error.log`. `worker_connections` و `worker_rlimit_nofile`.

**۶) SELinux و پورت غیراستاندارد**
`semanage port -a -t http_port_t -p tcp 8080`.

**۷) صفحه‌ی پیش‌فرض Nginx**
`sites-enabled/default` هنوز فعال است. حذف یا تغییر.

**۸) `user` اشتباه**
`www-data` در Debian، `nginx` در RHEL. `ps aux | grep nginx`.

**۹) ویندوز: `nginx.exe` در PowerShell بسته می‌شود**
با `Start-Process` یا `nssm` به‌عنوان سرویس.

**۱۰) فایروال و IPv6**
`ufw` فقط IPv4؟ `ip6tables` را چک کن.

### نکات مهم (Tips & Tricks)

- **`nginx -T`** کل config بعد از include.
- **`nginx -V`** ماژول‌های کامپایل‌شده.
- **`server_tokens off`** برای پنهان کردن نسخه.
- **`worker_processes auto`**.
- **`worker_rlimit_nofile`** برای اتصال زیاد.
- **`systemctl reload`** به‌جای `restart`.
- **`sites-enabled/default`** را حذف کن.

### خلاصه فصل

- مخزن رسمی برای نسخه‌ی جدید و HTTP/3.
- مسیرها در Debian و RHEL متفاوت‌اند (`sites-available` در برابر `conf.d`).
- `nginx -t`/`-T`/`-V` ابزارهای روزمره.
- ویندوز فقط برای توسعه یا پروکسی سبک.
- SELinux نیاز به تنظیم پوشه و پورت دارد.

**سؤال مروری:** چرا `nginx -t` می‌گوید config سالم است ولی بعد از reload سایت تغییر نکرده؟

---

## فصل ۱۹: ساختار `nginx.conf` و contextها

### مقدمه

Nginx یک زبان پیکربندی سلسله‌مراتبی دارد: `main` → `events`/`http` → `server` → `location`. هر دستور، دامنه‌ی اعتبار مشخصی دارد و اگر در context اشتباه بنویسی، خطای `directive is not allowed here` می‌گیری. این فصل، نقشه‌ی کامل است.

### مفاهیم اساسی

**۱) درخت context**

```
main
├── events { ... }              ← تنظیمات اتصال
├── http {                      ← تنظیمات HTTP
│   ├── upstream backend { }    ← گروه سرورها
│   ├── map $a $b { }           ← متغیر مشتق
│   ├── server {                ← یک سایت
│   │   ├── location / { }      ← مسیر
│   │   └── location /api/ { }
│   └── server { }
└── stream { }                  ← TCP/UDP proxy (بدون HTTP)
```

**۲) دستورهای `main`**

```nginx
user  www-data;
worker_processes  auto;
worker_rlimit_nofile 65535;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log warn;
load_module modules/ngx_http_geoip_module.so;
```

**۳) `events`**

```nginx
events {
    worker_connections  16384;
    multi_accept        on;
    use                 epoll;
    accept_mutex        off;      # با reuseport
}
```

**۴) `http` — دستورهای مهم**

| دستور | کار |
|---|---|
| `sendfile on` | کپی کرنل |
| `tcp_nopush on` | جمع بسته‌ها |
| `tcp_nodelay on` | ارسال فوری |
| `keepalive_timeout 65` | اتصال idle |
| `keepalive_requests 1000` | حداکثر درخواست روی اتصال |
| `client_max_body_size 16m` | حداکثر بدنه |
| `client_body_timeout 12s` | زمان خواندن بدنه |
| `server_tokens off` | پنهان کردن نسخه |
| `types_hash_max_size 2048` | اندازه‌ی جدول MIME |
| `default_type` | MIME پیش‌فرض |
| `log_format` | قالب لاگ |
| `include` | فایل دیگر |

**۵) `server` — دستورهای مهم**

| دستور | کار |
|---|---|
| `listen 80` | پورت |
| `server_name` | دامنه |
| `root` | ریشه‌ی فایل |
| `index` | فایل پیش‌فرض |
| `access_log`/`error_log` | لاگ per-site |
| `ssl_certificate` | گواهی |
| `return` | پاسخ مستقیم |
| `rewrite` | بازنویسی |
| `location` | مسیر |

**۶) `location` — دستورهای مهم**

| دستور | کار |
|---|---|
| `root`/`alias` | مسیر فایل |
| `try_files` | ترتیب تلاش |
| `proxy_pass` | پروکسی |
| `fastcgi_pass` | PHP-FPM |
| `uwsgi_pass`/`grpc_pass` | پروتکل دیگر |
| `expires` | کش |
| `add_header` | هدر |
| `deny`/`allow` | دسترسی |
| `limit_req`/`limit_conn` | محدودیت |
| `internal` | فقط داخلی |

**۷) ارث‌بری (inheritance)**

```nginx
http {
    add_header X-Global "1";       # ارث می‌رسد
    server {
        add_header X-Server "2";   # ❌ X-Global را حذف می‌کند!
    }
}
```

> **تله‌ی معروف:** `add_header` در سطح فرزند، **همه‌ی** `add_header`های والد را نادیده می‌گیرد. اگر هدر امنیتی در `http` داری و در `location` یک `add_header` می‌گذاری، هدرهای امنیتی از دست می‌روند. راه‌حل: `include` مشترک یا تکرار.

دستورهایی که ارث می‌برند: `root`، `index`، `proxy_*`، `gzip_*`، `client_*`.
دستورهایی که override کامل می‌کنند: `add_header`، `proxy_set_header` (در سطح‌های مختلف).

**۸) `include` و ساختار فایل**

```nginx
include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-enabled/*;
include /etc/nginx/snippets/ssl.conf;
include /etc/nginx/snippets/security-headers.conf;
```

**۹) `map` — متغیر مشتق**

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

map $request_uri $is_api {
    default 0;
    ~^/api/ 1;
}

map $http_user_agent $is_bot {
    default 0;
    ~*(bot|crawler|spider) 1;
}
```

**۱۰) `upstream`**

```nginx
upstream backend {
    least_conn;
    server 127.0.0.1:3000 weight=3 max_fails=3 fail_timeout=30s;
    server 127.0.0.1:3001;
    keepalive 32;
}
```

**۱۱) `stream` — TCP/UDP proxy**

```nginx
stream {
    upstream db {
        server 10.0.0.11:5432;
        server 10.0.0.12:5432;
    }
    server {
        listen 5432;
        proxy_pass db;
    }
}
```

**۱۲) `nginx -T` — دیدن config نهایی**

```bash
sudo nginx -T | less
```

همه‌ی `include`ها را باز می‌کند و با کامنت مسیر فایل نشان می‌دهد. ابزار اصلی دیباگ.

### مثال‌های عملی

**Lab Scenario: ساختار ماژولار کامل**

```
/etc/nginx/
├── nginx.conf
├── conf.d/
│   ├── 00-logging.conf
│   ├── 10-gzip.conf
│   └── 20-security.conf
├── snippets/
│   ├── ssl.conf
│   ├── proxy-common.conf
│   └── security-headers.conf
├── sites-available/
│   ├── lab.conf
│   └── api.conf
└── sites-enabled/
    ├── lab.conf -> ../sites-available/lab.conf
    └── api.conf -> ../sites-available/api.conf
```

```nginx
# /etc/nginx/nginx.conf
user  www-data;
worker_processes  auto;
worker_rlimit_nofile 65535;

error_log  /var/log/nginx/error.log warn;
pid        /run/nginx.pid;

events {
    worker_connections  16384;
    multi_accept        on;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" rt=$request_time';

    access_log /var/log/nginx/access.log main;

    sendfile        on;
    tcp_nopush      on;
    tcp_nodelay     on;
    keepalive_timeout 65;
    server_tokens   off;
    client_max_body_size 16m;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

```nginx
# /etc/nginx/snippets/security-headers.conf
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
```

```nginx
# /etc/nginx/snippets/ssl.conf
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
ssl_prefer_server_ciphers off;
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 1d;
ssl_session_tickets off;
ssl_stapling on;
ssl_stapling_verify on;
resolver 1.1.1.1 8.8.8.8 valid=300s;
```

```nginx
# /etc/nginx/sites-available/lab.conf
server {
    listen 80;
    server_name lab.example.com;

    root /var/www/lab;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

```bash
# فعال‌سازی
sudo ln -s /etc/nginx/sites-available/lab.conf /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx

# دیدن config نهایی
sudo nginx -T | grep -n "security-headers" -A5
```

**دیباگ ارث‌بری `add_header`:**

```nginx
server {
    listen 8080;
    server_name _;
    add_header X-A "1";

    location /with-header {
        add_header X-B "2";      # ❌ X-A حذف می‌شود
        return 200 "ok\n";
    }

    location /inherit {
        return 200 "ok\n";       # ✅ X-A ارث می‌رسد
    }
}
```

```bash
curl -sSI http://localhost:8080/with-header | grep -i x-
# x-b: 2       (X-A نیست!)

curl -sSI http://localhost:8080/inherit | grep -i x-
# x-a: 1
```

**راه‌حل:** `include` مشترک در هر دو سطح:

```nginx
location /with-header {
    include snippets/security-headers.conf;
    add_header X-B "2" always;
    return 200 "ok\n";
}
```

**`map` در عمل:**

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 8080;
    location /ws {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade    $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
    }
}
```

**`stream` برای TCP:**

```nginx
# /etc/nginx/nginx.conf — خارج از http
stream {
    log_format basic '$remote_addr [$time_local] $protocol $status $bytes_sent $bytes_received';
    access_log /var/log/nginx/stream.log basic;

    upstream pg {
        server 10.0.0.11:5432;
        server 10.0.0.12:5432;
    }

    server {
        listen 5432;
        proxy_pass pg;
        proxy_timeout 10m;
    }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx
psql "host=127.0.0.1 port=5432 dbname=shop_db user=app" -c "SELECT 1;"
```

### مقایسه و الگوهای مختلف

**`root` در `http` در برابر `server` در برابر `location`**

| سطح | کاربرد |
|---|---|
| `http` | مسیر مشترک همه‌ی سایت‌ها (نادر) |
| `server` | ریشه‌ی سایت (رایج) |
| `location` | ریشه‌ی مسیر خاص |

**`map` در برابر `if`**

| | `map` | `if` |
|---|---|---|
| سرعت | ✅ (زمان config) | ❌ (زمان درخواست) |
| امنیت | ✅ | ❌ (`if is evil`) |
| مناسب | متغیر مشتق | شرط ساده |

> **`if is evil`** یک قانون معروف Nginx است: `if` در `location` رفتار غیرمنتظره دارد. تا جای ممکن `map`، `try_files`، یا `location` جدا استفاده کن.

**`include` در برابر کپی**

| | `include` | کپی |
|---|---|---|
| نگهداری | ✅ | ❌ |
| خوانایی | ✅ | ⚠️ |
| دیباگ | ⚠️ (`nginx -T`) | ✅ |

**دیدگاه «همه‌چیز در یک فایل»**: برای یک سایت کوچک کار می‌کند. با چند سایت و هدرهای مشترک، `snippets` و `sites-available` ضروری است.

### اشکال شایع

**۱) `directive is not allowed here`**
دستور در context اشتباه. `server` داخل `server` نه.

**۲) `add_header` که هدرهای والد را حذف می‌کند**
تله‌ی ارث‌بری. `include` یا تکرار.

**۳) `root` در `location` که به مسیر اضافه می‌شود**
`root` = `root + uri`؛ `alias` = جایگزین. (فصل ۲۲)

**۴) `if` در `location`**
رفتار غیرمنتظره. `map`/`try_files`.

**۵) `include` با glob خالی**
خطا نمی‌دهد (glob خالی مجاز است)، ولی فایل انتظاری لود نمی‌شود. `nginx -T`.

**۶) `upstream` بدون `keepalive`**
اتصال جدید برای هر درخواست. `keepalive 32`.

**۷) `stream` داخل `http`**
خطا. `stream` در سطح `main`.

**۸) `worker_processes` بیشتر از هسته**
`context switch` و کندی. `auto`.

**۹) `log_format` بعد از `access_log`**
خطا. ترتیب مهم است.

**۱۰) `map` با متغیر ناموجود**
مقدار `default`. تست با `add_header X-Debug $var always`.

### نکات مهم (Tips & Tricks)

- **`nginx -T`** همه‌ی config.
- **`map`** به‌جای `if`.
- **`snippets/`** برای هدرها و SSL.
- **`include`** برای تکرار.
- **`add_header ... always`** برای پاسخ‌های خطا.
- **`upstream keepalive`** برای کارایی.
- **`stream`** برای TCP/UDP.

### خلاصه فصل

- Nginx سلسله‌مراتب `main` → `events`/`http` → `server` → `location` دارد.
- `add_header` در فرزند، والد را حذف می‌کند.
- `map` جای `if` است.
- `include` و `snippets` برای ماژولار بودن.
- `upstream` با `keepalive` برای پروکسی کارآمد.
- `nginx -T` ابزار دیباگ نهایی.

**سؤال مروری:** چرا `add_header X-Frame-Options` در `http` و `add_header Cache-Control` در `location` باعث می‌شود `X-Frame-Options` ارسال نشود؟

---

## فصل ۲۰: `server` block و میزبانی چند سایت

### مقدمه

`server` block معادل `VirtualHost` در Apache است. تفاوت مهم: Nginx با `server_name` تطبیق می‌دهد و اگر هیچ‌کدام مطابقت نکند، **اولین** `server` روی آن پورت (یا `default_server`) پاسخ می‌دهد.

### مفاهیم اساسی

**۱) ساختار**

```nginx
server {
    listen 80;
    server_name lab.example.com www.lab.example.com;
    root /var/www/lab;
    index index.html;
}
```

**۲) `listen`**

```nginx
listen 80;                      # همه‌ی IPها، IPv4
listen [::]:80;                 # IPv6
listen 192.168.56.10:80;        # IP خاص
listen 80 default_server;       # default برای پورت ۸۰
listen 443 ssl;                 # HTTPS
listen 443 ssl http2;           # سبک قدیمی (تا 1.25.0)
listen 443 ssl; http2 on;       # سبک جدید (1.25.1+)
listen 80 reuseport;            # پخش بار کرنل
```

**۳) `server_name` — انواع تطبیق**

| نوع | مثال | اولویت |
|---|---|---|
| دقیق | `lab.example.com` | ۱ |
| wildcard شروع | `*.example.com` | ۲ |
| wildcard پایان | `mail.*` | ۳ |
| regex | `~^api\d+\.example\.com$` | ۴ |
| `_` | `server_name _;` | default (بدون تطبیق) |

**۴) `default_server`**

```nginx
server {
    listen 80 default_server;
    server_name _;
    return 444;          # بستن اتصال بدون پاسخ (Nginx-specific)
}
```

`444` = بستن بدون پاسخ. برای درخواست‌های ناشناس (اسکنرها) عالی است.

**۵) `server_name` با چند نام**

```nginx
server_name lab.example.com www.lab.example.com lab.local;
```

**۶) HTTPS با SNI**

```nginx
server {
    listen 443 ssl;
    http2 on;
    server_name lab.example.com;
    ssl_certificate     /etc/letsencrypt/live/lab.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/lab.example.com/privkey.pem;
}

server {
    listen 443 ssl;
    http2 on;
    server_name api.lab.example.com;
    ssl_certificate     /etc/letsencrypt/live/api.lab.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.lab.example.com/privkey.pem;
}
```

Nginx با SNI گواهی درست را انتخاب می‌کند.

**۷) redirect www/non-www**

```nginx
# www → non-www
server {
    listen 80;
    server_name www.lab.example.com;
    return 301 https://lab.example.com$request_uri;
}

# non-www → www
server {
    listen 80;
    server_name lab.example.com;
    return 301 https://www.lab.example.com$request_uri;
}
```

**۸) redirect HTTP → HTTPS**

```nginx
server {
    listen 80;
    server_name lab.example.com www.lab.example.com;
    return 301 https://$host$request_uri;
}
```

**۹) `$host` در برابر `$http_host`**

| متغیر | مقدار |
|---|---|
| `$host` | `server_name` یا `Host` (بدون پورت)، lowercase |
| `$http_host` | `Host` خام (با پورت) |
| `$server_name` | `server_name` اول |

برای redirect، `$host` امن‌تر است (بدون تزریق پورت).

**۱۰) `server` برای پورت داخلی**

```nginx
server {
    listen 127.0.0.1:8081;
    server_name _;
    location /status { stub_status; allow 127.0.0.1; deny all; }
}
```

### مثال‌های عملی

**Lab Scenario: سه سایت + default + redirect**

```nginx
# /etc/nginx/sites-available/000-default.conf
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;
    return 444;
}
```

```nginx
# /etc/nginx/sites-available/lab.conf
server {
    listen 80;
    listen [::]:80;
    server_name lab.example.com www.lab.example.com;

    return 301 https://lab.example.com$request_uri;
}

server {
    listen 443 ssl;
    http2 on;
    server_name lab.example.com;

    include snippets/ssl.conf;
    include snippets/security-headers.conf;

    ssl_certificate     /etc/letsencrypt/live/lab.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/lab.example.com/privkey.pem;

    root /var/www/lab;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

# www → non-www (فقط اگر گواهی www هم داری)
server {
    listen 443 ssl;
    http2 on;
    server_name www.lab.example.com;

    include snippets/ssl.conf;
    ssl_certificate     /etc/letsencrypt/live/lab.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/lab.example.com/privkey.pem;

    return 301 https://lab.example.com$request_uri;
}
```

```nginx
# /etc/nginx/sites-available/api.conf
server {
    listen 80;
    server_name api.lab.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    http2 on;
    server_name api.lab.example.com;

    include snippets/ssl.conf;
    include snippets/security-headers.conf;

    ssl_certificate     /etc/letsencrypt/live/api.lab.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.lab.example.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/000-default.conf /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/lab.conf /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/api.conf /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

**تست تطبیق `server_name`:**

```bash
for h in lab.example.com www.lab.example.com api.lab.example.com unknown.example.com; do
  printf '%-25s → ' "$h"
  curl -s -o /dev/null -w '%{http_code} %{redirect_url}\n' -H "Host: $h" http://127.0.0.1/
done
```

خروجی نمونه:

```
lab.example.com           → 301 https://lab.example.com/
www.lab.example.com       → 301 https://lab.example.com/
api.lab.example.com       → 301 https://api.lab.example.com/
unknown.example.com       → 000 (اتصال بسته شد)
```

**بررسی گواهی هر دامنه (SNI):**

```bash
echo | openssl s_client -connect 192.168.56.10:443 -servername lab.example.com 2>/dev/null \
  | openssl x509 -noout -subject

echo | openssl s_client -connect 192.168.56.10:443 -servername api.lab.example.com 2>/dev/null \
  | openssl x509 -noout -subject
```

**`server_name` با regex:**

```nginx
server {
    listen 80;
    server_name ~^(?<sub>[a-z0-9-]+)\.lab\.example\.com$;

    root /var/www/$sub;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

**`server_name` با wildcard:**

```nginx
server {
    listen 80;
    server_name *.lab.example.com;
    return 301 https://$host$request_uri;
}
```

**بررسی config مؤثر:**

```bash
sudo nginx -T | grep -E "server_name|listen" | head -30
```

### مقایسه و الگوهای مختلف

**`server_name` در برابر `VirtualHost`**

| | Nginx | Apache |
|---|---|---|
| تطبیق دقیق | ✅ | ✅ |
| wildcard | ✅ | ✅ |
| regex | ✅ (`~`) | ⚠️ (`ServerAlias` با wildcard) |
| default | `default_server` | اولین vhost |
| بدون تطبیق | `444` ممکن | default |

**سه روش redirect**

| روش | مثال |
|---|---|
| `return 301` | `return 301 https://$host$request_uri;` |
| `rewrite ... permanent` | `rewrite ^ https://$host$request_uri permanent;` |
| `if` + `return` | ❌ (توصیه نمی‌شود) |

**`$host` در برابر `$http_host` در redirect**

| | `$host` | `$http_host` |
|---|---|---|
| پورت | ❌ | ✅ |
| امنیت | ✅ | ⚠️ |
| lowercase | ✅ | ❌ |
| توصیه | ✅ | ❌ |

**دیدگاه «`default_server` لازم نیست»**: بدون آن، اولین `server` روی پورت، default است و ممکن است سایت اشتباه به اسکنرها پاسخ دهد. `444` یا `deny` بهتر است.

### اشکال شایع

**۱) `conflicting server name` در لاگ**
دو `server` با یک `server_name` روی یک پورت. `nginx -T | grep server_name`.

**۲) `default_server` تکراری**
`a duplicate default server for 0.0.0.0:80`. فقط یکی.

**۳) سایت اول همه‌چیز را می‌گیرد**
`default_server` تعریف نشده. `444`.

**۴) redirect loop**
`return 301 https://...` در `server` پورت ۴۴۳ هم هست. فقط ۸۰.

**۵) `www` بدون گواهی**
`www` به HTTPS می‌رود ولی گواهی ندارد → خطای نام. گواهی با SAN یا redirect در ۸۰.

**۶) `$host` خالی**
HTTP/1.0 بدون `Host`. `server_name` استفاده می‌شود.

**۷) `listen 443 ssl http2;` در نسخه‌ی جدید**
هشدار منسوخ. `http2 on;`.

**۸) `server_name` با نقطه‌ی اضافه**
`lab.example.com.` با نقطه‌ی پایانی، تطبیق نمی‌کند.

**۹) `return` داخل `location` با `proxy_pass`**
`return` برنده است. ترتیب و سطح مهم.

**۱۰) `server` بدون `listen`**
پورت پیش‌فرض ۸۰. صریح بنویس.

### نکات مهم (Tips & Tricks)

- **`default_server` با `444`**.
- **`$host`** در redirect.
- **`nginx -T | grep server_name`** برای تعارض.
- **SNI** با `openssl s_client -servername`.
- **regex با named capture** برای زیردامنه‌ی پویا.
- **`include snippets/ssl.conf`** برای تکرار.
- **`http2 on;`** جدا در نسخه‌ی جدید.

### خلاصه فصل

- `server` block = سایت؛ `server_name` تطبیق می‌دهد.
- بدون تطبیق، `default_server` یا اولین `server` پاسخ می‌دهد.
- `444` اتصال را می‌بندد.
- SNI گواهی درست را انتخاب می‌کند.
- `$host` امن‌تر از `$http_host` در redirect.
- `nginx -T` تعارض‌ها را نشان می‌دهد.

**سؤال مروری:** چرا تعریف `default_server` با `return 444` از نظر امنیتی بهتر از نداشتن آن است؟

---

## فصل ۲۱: `location` — تطبیق، اولویت و تله‌ها

### مقدمه

`location` پرکاربردترین و گیج‌کننده‌ترین بخش Nginx است. سه نوع تطبیق (دقیق، پیشوندی، regex) با اولویت غیرشهودی وجود دارد و بیشتر باگ‌های «چرا این مسیر به آن‌جا می‌رود» از همین‌جا می‌آید.

### مفاهیم اساسی

**۱) چهار نوع**

| سینتکس | نوع | مثال |
|---|---|---|
| `location = /path` | دقیق | `location = /health` |
| `location /path` | پیشوندی | `location /api/` |
| `location ^~ /path` | پیشوندی با توقف regex | `location ^~ /static/` |
| `location ~ regex` | regex (حساس) | `location ~ \.php$` |
| `location ~* regex` | regex (غیرحساس) | `location ~* \.(jpg\|png)$` |
| `location @name` | named (داخلی) | `location @fallback` |

**۲) الگوریتم تطبیق (به ترتیب)**

```
۱. دقیق (=): اگر تطبیق کرد → همان. تمام.
۲. پیشوندی: بلندترین پیشوند را پیدا کن.
   - اگر با ^~ شروع شود → همان. regex اجرا نمی‌شود. تمام.
۳. regex (~ و ~*): به ترتیب تعریف. اولین تطبیق برنده.
۴. اگر regex تطبیق نکرد → بلندترین پیشوند (از مرحله ۲).
```

**۳) مثال عددی**

```nginx
location = /exact       { return 200 "exact\n"; }
location ^~ /static/    { return 200 "static\n"; }
location ~ \.php$       { return 200 "php\n"; }
location /api/          { return 200 "api\n"; }
location /               { return 200 "root\n"; }
```

| درخواست | پاسخ | چرا |
|---|---|---|
| `/exact` | `exact` | تطبیق دقیق |
| `/exact/` | `root` | دقیق نیست، پیشوندی `/` |
| `/static/a.php` | `static` | `^~` regex را متوقف می‌کند |
| `/api/a.php` | `php` | regex قبل از پیشوندی (بدون `^~`) |
| `/api/x` | `api` | پیشوندی بلندتر |
| `/other` | `root` | پیشوندی `/` |

**۴) تله‌ی اصلی: regex بر پیشوندی مقدم است**

```nginx
location /api/   { proxy_pass http://api; }
location ~ \.php$ { return 403; }
```

درخواست `/api/index.php` → `403` (regex برنده). اگر می‌خواهی `/api/` برنده شود:

```nginx
location ^~ /api/ { proxy_pass http://api; }
```

**۵) `location /` و `try_files`**

```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

**۶) `location` تودرتو**

```nginx
location /api/ {
    proxy_pass http://api;

    location ~ \.php$ {
        return 403;      # داخل /api/، regex
    }
}
```

regex تودرتو فقط داخل پیشوندی والد بررسی می‌شود.

**۷) `location @named`**

```nginx
location / {
    try_files $uri $uri/ @backend;
}

location @backend {
    proxy_pass http://127.0.0.1:3000;
}
```

**۸) `location` با `alias`**

```nginx
location /static/ {
    alias /var/www/static/;    # /static/a.css → /var/www/static/a.css
}

location /files/ {
    root /var/www;             # /files/a.css → /var/www/files/a.css
}
```

**۹) `internal`**

```nginx
location /internal/ {
    internal;                  # فقط از rewrite/proxy داخلی
}
```

**۱۰) `location` و `proxy_pass` با/بدون `/`**

```nginx
location /api/ {
    proxy_pass http://backend;      # /api/users → /api/users
}

location /api/ {
    proxy_pass http://backend/;     # /api/users → /users
}
```

**تله‌ی کلاسیک:** `/` پایانی، مسیر `location` را حذف می‌کند.

### مثال‌های عملی

**Lab Scenario: نقشه‌ی کامل مسیرها**

```nginx
server {
    listen 8080;
    server_name _;
    root /var/www/lab;

    # ۱) health check (دقیق)
    location = /health {
        access_log off;
        default_type text/plain;
        return 200 "ok\n";
    }

    # ۲) استاتیک (^~ برای جلوگیری از regex)
    location ^~ /static/ {
        alias /var/www/static/;
        expires 30d;
        add_header Cache-Control "public, immutable";
        access_log off;
    }

    # ۳) API (^~ برای جلوگیری از regex)
    location ^~ /api/ {
        proxy_pass http://127.0.0.1:3000/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # ۴) PHP (regex)
    location ~ \.php$ {
        return 403;
    }

    # ۵) فایل‌های مخفی
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }

    # ۶) SPA fallback
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx

# تست هر مسیر
for p in /health /static/app.css /api/users /index.php /.env /some/route; do
  printf '%-20s → ' "$p"
  curl -s -o /dev/null -w '%{http_code}\n' "http://localhost:8080$p"
done
```

**دیباگ تطبیق با `add_header`:**

```nginx
location = /debug {
    default_type text/plain;
    return 200 "matched: exact\n";
}

location /debug/ {
    default_type text/plain;
    return 200 "matched: prefix\n";
}
```

یا با `error_log debug`:

```nginx
error_log /var/log/nginx/debug.log debug;
```

```bash
curl -s http://localhost:8080/debug
sudo grep -i "using configuration" /var/log/nginx/debug.log | tail -5
# using configuration "/etc/nginx/...":123
```

**`location` برای فایل استاتیک با `try_files`:**

```nginx
location ~* \.(css|js|jpg|jpeg|png|gif|ico|svg|woff2)$ {
    expires 30d;
    add_header Cache-Control "public, immutable";
    try_files $uri =404;
    access_log off;
}
```

**`location` برای WebSocket:**

```nginx
location /ws {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_read_timeout 3600s;
}
```

**`location` برای محدودیت:**

```nginx
location /login {
    limit_req zone=login burst=5 nodelay;
    proxy_pass http://127.0.0.1:3000;
}
```

**تست اولویت با `curl`:**

```bash
# جدول اولویت را عملاً ببین
curl -s http://localhost:8080/api/test.php     # api (چون ^~)
curl -s http://localhost:8080/other/test.php   # 403 (regex php)
curl -s http://localhost:8080/.git/config      # 403 (deny)
```

### مقایسه و الگوهای مختلف

**چهار نوع `location`**

| نوع | سرعت | اولویت | کاربرد |
|---|---|---|---|
| `=` | ✅✅ | ۱ | health، دقیق |
| `^~` | ✅ | ۲ | استاتیک، API |
| `~`/`~*` | ⚠️ | ۳ | پسوند، الگو |
| پیشوندی | ✅ | ۴ (fallback) | عمومی |

**`root` در برابر `alias`**

| درخواست | `root /var/www;` | `alias /var/www/static/;` |
|---|---|---|
| `/a.css` | `/var/www/a.css` | — |
| `/static/a.css` | `/var/www/static/a.css` | `/var/www/static/a.css` |
| مسیر کامل | `root + uri` | `alias` جایگزین پیشوند |

**`try_files` در برابر `if`**

| | `try_files` | `if` |
|---|---|---|
| امن | ✅ | ❌ |
| سرعت | ✅ | ⚠️ |
| مناسب | فایل/fallback | شرط ساده |

**دیدگاه «regex راحت‌تر است»**: regex در هر درخواست اجرا می‌شود و اولویت را غیرشهودی می‌کند. تا جای ممکن `=` و `^~` استفاده کن.

### اشکال شایع

**۱) regex بر پیشوندی مقدم است**
`/api/x.php` به `location ~ \.php$` می‌رود. `^~`.

**۲) `proxy_pass` با `/` پایانی**
مسیر حذف می‌شود. تفاوت را بفهم.

**۳) `root` به‌جای `alias`**
مسیر دوبار تکرار می‌شود. `alias`.

**۴) `location /` بدون `try_files`**
`404` برای SPA. `try_files`.

**۵) `location` تودرتو با regex**
فقط داخل والد بررسی می‌شود. `^~` در والد.

**۶) `location` بدون `listen`/`server`**
خطا. داخل `server` بنویس.

**۷) `location = /path` با query string**
`=` فقط مسیر را می‌بیند، نه query. `/path?a=1` تطبیق می‌کند.

**۸) `deny all` در `location ~ /\.`**
`.well-known` را هم می‌بندد (Let's Encrypt). استثنا بگذار:

```nginx
location ~ /\.(?!well-known) { deny all; }
```

**۹) `location` با `alias` و `try_files`**
`alias` + `try_files` می‌تواند مسیر را غلط بسازد. `root` امن‌تر.

**۱۰) `location` با کاراکتر خاص**
`location /a b/` نیاز به quote. `location "/a b/"`.

### نکات مهم (Tips & Tricks)

- **`=` برای health**.
- **`^~` برای استاتیک و API**.
- **`error_log debug`** برای دیدن تطبیق.
- **`nginx -T`** برای دیدن ترتیب.
- **`~ /\.(?!well-known)`** برای فایل‌های مخفی.
- **`try_files`** به‌جای `if`.
- **`alias` با `/` پایانی**.

### خلاصه فصل

- چهار نوع `location`: دقیق، پیشوندی، `^~`، regex.
- اولویت: `=` → `^~` → regex → پیشوندی بلندتر.
- regex بر پیشوندی مقدم است؛ `^~` آن را متوقف می‌کند.
- `root` اضافه می‌کند، `alias` جایگزین.
- `proxy_pass` با `/` مسیر را حذف می‌کند.
- `error_log debug` تطبیق را نشان می‌دهد.

**سؤال مروری:** چرا `/api/index.php` با `location /api/` و `location ~ \.php$` به `403` می‌رسد و چطور آن را به `proxy_pass` می‌فرستی؟

---

## فصل ۲۲: `root`/`alias`، `try_files`، `internal` و `error_page`

### مقدمه

این چهار دستور، ستون فقرات سرو فایل و مدیریت مسیر در Nginx هستند. تفاوت `root` و `alias`، ترتیب `try_files`، و رفتار `internal`، منبع بیشترین باگ‌های `404`/`403` است.

### مفاهیم اساسی

**۱) `root` — اضافه کردن**

```nginx
server {
    root /var/www/lab;
    location /images/ {
        # /images/a.png → /var/www/lab/images/a.png
    }
}
```

`root` همیشه `root + uri` می‌سازد.

**۲) `alias` — جایگزینی**

```nginx
location /static/ {
    alias /var/www/assets/;
    # /static/a.css → /var/www/assets/a.css
}
```

`alias` پیشوند `location` را با مسیر داده‌شده جایگزین می‌کند.

**۳) تله‌ی `/` پایانی در `alias`**

```nginx
location /static {
    alias /var/www/assets/;     # /static/a.css → /var/www/assets//a.css (کار می‌کند ولی زشت)
}

location /static/ {
    alias /var/www/assets;      # /static/a.css → /var/www/assetsa.css (خراب!)
}
```

قاعده: `location` و `alias` یا هر دو `/` داشته باشند یا هیچ‌کدام.

**۴) `try_files` — ترتیب تلاش**

```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

| مرحله | معنا |
|---|---|
| `$uri` | فایل دقیق |
| `$uri/` | پوشه (با `index`) |
| `/index.html` | fallback داخلی |
| `=404` | خطای صریح |

**۵) `try_files` با named location**

```nginx
location / {
    try_files $uri $uri/ @app;
}

location @app {
    proxy_pass http://127.0.0.1:3000;
}
```

**۶) `try_files` با `=code`**

```nginx
location /downloads/ {
    try_files $uri =404;
}
```

`=404` از `index` و `autoindex` جلوگیری می‌کند.

**۷) `internal` — فقط دسترسی داخلی**

```nginx
location /internal/metrics {
    internal;
    proxy_pass http://127.0.0.1:9090/metrics;
}
```

`internal` یعنی: درخواست مستقیم از کلاینت → `404`؛ درخواست از `rewrite`/`X-Accel-Redirect`/`error_page` → مجاز.

**۸) `X-Accel-Redirect`**

اپ می‌تواند به Nginx بگوید «این فایل را خودت بفرست»:

```python
# در اپ
response.headers['X-Accel-Redirect'] = '/internal/files/report.pdf'
response.headers['Content-Disposition'] = 'attachment; filename=report.pdf'
```

```nginx
location /internal/files/ {
    internal;
    alias /var/private/;
}
```

مزیت: فایل از Nginx سرو می‌شود (سریع)، ولی دسترسی کنترل‌شده است.

**۹) `error_page`**

```nginx
error_page 404 /404.html;
error_page 500 502 503 504 /50x.html;

location = /404.html { internal; }
location = /50x.html { internal; root /var/www/errors; }
```

**۱۰) `error_page` با redirect**

```nginx
error_page 404 =301 https://example.com/new-location;
```

**۱۱) `error_page` با named location**

```nginx
error_page 404 @fallback;

location @fallback {
    proxy_pass http://127.0.0.1:3000;
}
```

**۱۲) `index` و `autoindex`**

```nginx
index index.html index.htm;

location /files/ {
    autoindex on;              # ❌ در تولید خطرناک
    autoindex_exact_size off;
    autoindex_localtime on;
}
```

### مثال‌های عملی

**Lab Scenario: سرو فایل با کنترل دسترسی**

ساختار:

```
/var/www/lab/
├── index.html
├── static/
│   └── app.css
└── uploads/          (خارج از دسترس مستقیم)
/var/private/
└── reports/
    └── q3.pdf
```

```nginx
server {
    listen 8080;
    server_name _;
    root /var/www/lab;

    # ۱) استاتیک با alias
    location /assets/ {
        alias /var/www/lab/static/;
        expires 7d;
        add_header Cache-Control "public";
        try_files $uri =404;
    }

    # ۲) فایل‌های خصوصی (فقط از X-Accel-Redirect)
    location /internal/reports/ {
        internal;
        alias /var/private/reports/;
        add_header Content-Disposition "attachment";
    }

    # ۳) SPA
    location / {
        try_files $uri $uri/ /index.html;
    }

    # ۴) خطاها
    error_page 404 /404.html;
    location = /404.html { internal; }

    error_page 500 502 503 504 /50x.html;
    location = /50x.html { internal; root /var/www/errors; }
}
```

```bash
mkdir -p /var/www/lab/static /var/www/errors /var/private/reports
echo 'body{}' > /var/www/lab/static/app.css
echo '<h1>404</h1>' > /var/www/lab/404.html
echo '<h1>50x</h1>' > /var/www/errors/50x.html
echo 'PDF' > /var/private/reports/q3.pdf
sudo chown -R www-data:www-data /var/www /var/private

sudo nginx -t && sudo systemctl reload nginx

# تست
curl -sI http://localhost:8080/assets/app.css
curl -s -o /dev/null -w '%{http_code}\n' http://localhost:8080/internal/reports/q3.pdf   # 404
curl -s http://localhost:8080/nonexistent                                                # صفحه 404
```

**تست `X-Accel-Redirect` با اپ:**

```javascript
// server.js
import express from 'express';
const app = express();

app.get('/download/:file', (req, res) => {
  // بررسی احراز هویت/مجوز
  if (req.query.token !== 'secret') return res.status(403).end();

  res.setHeader('X-Accel-Redirect', `/internal/reports/${req.params.file}`);
  res.setHeader('Content-Disposition', `attachment; filename="${req.params.file}"`);
  res.end();
});

app.listen(3000);
```

```nginx
location /api/ {
    proxy_pass http://127.0.0.1:3000/;
}

location /internal/reports/ {
    internal;
    alias /var/private/reports/;
}
```

```bash
curl -sI 'http://localhost:8080/api/download/q3.pdf?token=secret'
# X-Accel-Redirect... سپس Nginx فایل را می‌فرستد
curl -s 'http://localhost:8080/api/download/q3.pdf?token=secret'
# PDF
```

**`try_files` برای PHP fallback:**

```nginx
location / {
    try_files $uri $uri/ /index.php?$query_string;
}

location ~ \.php$ {
    fastcgi_pass unix:/run/php/php8.3-fpm.sock;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
```

**`try_files` با named location:**

```nginx
location / {
    try_files $uri $uri/ @spa;
}

location @spa {
    root /var/www/lab;
    try_files /index.html =404;
}
```

**`error_page` با JSON برای API:**

```nginx
location /api/ {
    proxy_pass http://127.0.0.1:3000/;
    proxy_intercept_errors on;
    error_page 502 504 = @api_error;
}

location @api_error {
    default_type application/json;
    return 503 '{"error":"upstream unavailable"}';
}
```

**`autoindex` امن:**

```nginx
location /files/ {
    alias /var/files/;
    autoindex on;
    autoindex_format json;
    allow 10.0.0.0/8;
    deny all;
}
```

### مقایسه و الگوهای مختلف

**`root` در برابر `alias`**

| | `root` | `alias` |
|---|---|---|
| رفتار | `root + uri` | جایگزینی پیشوند |
| `/` پایانی | بی‌اهمیت | ⚠️ حیاتی |
| مناسب | ساختار یکسان | پوشه‌ی متفاوت |

**`try_files` در برابر `rewrite`**

| | `try_files` | `rewrite` |
|---|---|---|
| سرعت | ✅ | ⚠️ |
| خوانایی | ✅ | ⚠️ |
| مناسب | فایل/fallback | الگوی URL |

**`internal` در برابر `deny`**

| | `internal` | `deny all` |
|---|---|---|
| دسترسی مستقیم | `404` | `403` |
| از `X-Accel` | ✅ | ❌ |
| مناسب | فایل خصوصی | IP |

**`error_page` در برابر `return`**

| | `error_page` | `return` |
|---|---|---|
| واکنش به خطا | ✅ | ❌ |
| پاسخ مستقیم | ❌ | ✅ |
| مناسب | خطای upstream | redirect/پاسخ ثابت |

**دیدگاه «`autoindex on` راحت است»**: لیست فایل‌ها را به همه نشان می‌دهد. فقط پشت احراز هویت و IP.

### اشکال شایع

**۱) `alias` بدون `/` هماهنگ**
مسیر دوبار/غلط. هر دو با `/` یا هر دو بدون.

**۲) `try_files` با `alias`**
مسیر غلط. `root` استفاده کن.

**۳) `try_files` بدون fallback**
`404` برای SPA. `/index.html`.

**۴) `internal` و `404` بی‌توضیح**
درخواست مستقیم `404` می‌دهد (نه `403`). طبیعی است.

**۵) `X-Accel-Redirect` با `proxy_pass`**
Nginx هدر را فقط از `fastcgi`/`uwsgi`/`proxy` می‌پذیرد اگر `proxy_ignore_headers` نباشد. `X-Accel-Redirect` باید از upstream بیاید.

**۶) `error_page` و `proxy_intercept_errors`**
بدون آن، خطای upstream به کلاینت می‌رود. `on`.

**۷) `error_page` با `internal` فراموش‌شده**
صفحه‌ی خطا مستقیم قابل دسترسی. `internal`.

**۸) `autoindex` روشن**
افشای ساختار. خاموش.

**۹) `root` در `location` که والد را override می‌کند**
`root` ارث می‌رسد ولی در `location` می‌تواند عوض شود. دقت کن.

**۱۰) `try_files $uri/` بدون `index`**
`403` یا `404`. `index` بگذار.

### نکات مهم (Tips & Tricks)

- **`root` + `try_files`** امن‌تر از `alias` + `try_files`.
- **`alias` با `/` هماهنگ**.
- **`internal`** برای فایل خصوصی.
- **`X-Accel-Redirect`** برای سرو فایل توسط Nginx.
- **`error_page ... internal`**.
- **`proxy_intercept_errors on`** برای API.
- **`autoindex` خاموش** در تولید.

### خلاصه فصل

- `root` اضافه می‌کند، `alias` جایگزین.
- `/` پایانی در `alias` حیاتی است.
- `try_files` ترتیب تلاش را تعیین می‌کند.
- `internal` دسترسی مستقیم را می‌بندد.
- `X-Accel-Redirect` فایل را از Nginx سرو می‌کند.
- `error_page` با `internal` و `proxy_intercept_errors`.

**سؤال مروری:** چرا `location /static/ { alias /var/www/assets; }` باعث `404` می‌شود و اصلاحش چیست؟

---

## منابع و مراجع

**مستندات رسمی**
- [Nginx Documentation](https://nginx.org/en/docs/)
- [ngx_http_core_module — location](https://nginx.org/en/docs/http/ngx_http_core_module.html#location)
- [ngx_http_core_module — root/alias](https://nginx.org/en/docs/http/ngx_http_core_module.html#root)
- [ngx_http_core_module — try_files](https://nginx.org/en/docs/http/ngx_http_core_module.html#try_files)
- [ngx_http_headers_module](https://nginx.org/en/docs/http/ngx_http_headers_module.html)
- [Nginx on Windows](https://nginx.org/en/docs/windows.html)

**آموزش**
- [Nginx Admin Guide](https://docs.nginx.com/nginx/admin-guide/)
- [DigitalOcean — Nginx server blocks](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-virtual-hosts-on-ubuntu-16-04)
- [Nginx location matching — explained](https://www.digitalocean.com/community/tutorials/understanding-nginx-server-and-location-block-selection-algorithms)

**ابزار**
- [nginx -T output visualizer](https://nginxconfig.io/)
- [Gixy — Nginx config analyzer](https://github.com/yandex/gixy)

---

**بعدی:** [`06-IIS-Install-and-Config.md`](06-IIS-Install-and-Config.md) — نصب، `App Pool`، `web.config` و ASP.NET Core
