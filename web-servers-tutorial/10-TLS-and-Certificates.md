# بخش ۱۰: TLS و مدیریت گواهی

**مدت یادگیری**: ۵ تا ۶ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۰۱ (فصل ۴ — مبانی TLS)، بخش ۰۵ (Nginx `server`)

---

## فهرست محتوا

- [فصل ۴۱: TLS 1.2 و 1.3 — handshake و cipher suite](#فصل-۴۱)
- [فصل ۴۲: Let's Encrypt و ACME با Certbot](#فصل-۴۲)
- [فصل ۴۳: پیکربندی TLS در Nginx، Apache و IIS](#فصل-۴۳)
- [فصل ۴۴: HSTS، OCSP Stapling، Redirect و mTLS](#فصل-۴۴)
- [فصل ۴۵: مدیریت گواهی، چرخش و تست](#فصل-۴۵)

---

## فصل ۴۱: TLS 1.2 و 1.3 — handshake و cipher suite

### مقدمه

TLS یک «قرارداد» است: کلاینت و سرور توافق می‌کنند کدام نسخه، کدام الگوریتم و کدام کلید. جزئیات این توافق، تعیین می‌کند که اتصال امن است یا فقط «به نظر امن». این فصل، آنچه باید بدانی تا پیکربندی درست بنویسی.

### مفاهیم اساسی

**۱) TLS 1.2 handshake (دو RTT)**

```
Client                                    Server
  │──── ClientHello ────────────────────▶│
  │     (versions, ciphers, random, SNI)  │
  │                                       │
  │◀─── ServerHello ──────────────────────│
  │◀─── Certificate ──────────────────────│
  │◀─── ServerKeyExchange ────────────────│
  │◀─── ServerHelloDone ──────────────────│
  │                                       │
  │──── ClientKeyExchange ───────────────▶│
  │──── ChangeCipherSpec ────────────────▶│
  │──── Finished ────────────────────────▶│
  │                                       │
  │◀─── ChangeCipherSpec ─────────────────│
  │◀─── Finished ─────────────────────────│
  │                                       │
  │◀═══════ Application Data ════════════▶│
```

**۲) TLS 1.3 handshake (یک RTT)**

```
Client                                    Server
  │──── ClientHello ────────────────────▶│
  │     (key_share, supported_versions)   │
  │                                       │
  │◀─── ServerHello ──────────────────────│
  │◀─── {EncryptedExtensions} ────────────│
  │◀─── {Certificate} ────────────────────│
  │◀─── {CertificateVerify} ──────────────│
  │◀─── {Finished} ───────────────────────│
  │                                       │
  │──── {Finished} ──────────────────────▶│
  │                                       │
  │◀═══════ Application Data ════════════▶│
```

`{}` یعنی رمزنگاری‌شده. TLS 1.3 بخش بیشتری از handshake را رمز می‌کند (خصوصی‌تر).

**۳) `0-RTT` (TLS 1.3)**

با `session resumption`، کلاینت می‌تواند داده را در همان اولین بسته بفرستد. مزیت: تأخیر صفر. عیب: **replay attack** ممکن است — فقط برای درخواست‌های idempotent.

**۴) Cipher Suite**

TLS 1.2:

```
ECDHE-RSA-AES128-GCM-SHA256
│     │   │      │    │
│     │   │      │    └── MAC/PRF
│     │   │      └── حالت (GCM)
│     │   └── الگوریتم رمز (AES-128)
│     └── احراز هویت (RSA)
└── تبادل کلید (ECDHE = forward secrecy)
```

TLS 1.3:

```
TLS_AES_128_GCM_SHA256
TLS_AES_256_GCM_SHA384
TLS_CHACHA20_POLY1305_SHA256
```

ساده‌تر، همه با forward secrecy.

**۵) Forward Secrecy**

`ECDHE` کلید جلسه را طوری می‌سازد که حتی اگر کلید خصوصی سرور لو برود، اتصال‌های گذشته قابل رمزگشایی نباشند. `RSA` key exchange این را ندارد.

**۶) `SNI`**

کلاینت نام دامنه را در `ClientHello` می‌فرستد. سرور گواهی مناسب را انتخاب می‌کند.

**۷) `ALPN`**

انتخاب `h2`/`http/1.1` در همان handshake.

**۸) `OCSP Stapling`**

سرور وضعیت ابطال را از CA می‌گیرد و در handshake می‌فرستد. کلاینت لازم نیست به CA وصل شود.

**۹) `Certificate Transparency` (CT)**

گواهی‌ها در لاگ عمومی ثبت می‌شوند. مرورگر گواهی بدون SCT را رد می‌کند (برای گواهی‌های جدید).

**۱۰) `TLS termination` در برابر `passthrough`**

| مدل | توضیح |
|---|---|
| Termination | LB/proxy رمزگشایی، HTTP به backend |
| Passthrough | LB فقط TCP، TLS در backend |
| Re-encryption | رمزگشایی و دوباره رمزنگاری |

### مثال‌های عملی

**Lab Scenario: دیدن handshake از نزدیک**

```bash
# TLS 1.2
openssl s_client -connect lab.example.com:443 -tls1_2 -servername lab.example.com </dev/null 2>&1 \
  | grep -E "Protocol|Cipher|Server public key|Verify return"

# TLS 1.3
openssl s_client -connect lab.example.com:443 -tls1_3 -servername lab.example.com </dev/null 2>&1 \
  | grep -E "Protocol|Cipher|Verify return"
```

**دیدن cipher suiteها:**

```bash
# TLS 1.2
openssl ciphers -v 'ECDHE+AESGCM:ECDHE+CHACHA20' | head -10

# TLS 1.3
openssl ciphers -v -s -tls1_3 | head
```

**تست `0-RTT`:**

```bash
openssl s_client -connect lab.example.com:443 -tls1_3 -early_data </dev/null 2>&1 | head -20
```

**بررسی `ALPN`:**

```bash
openssl s_client -connect lab.example.com:443 -alpn h2,http/1.1 </dev/null 2>&1 \
  | grep -i "ALPN protocol"
# ALPN protocol: h2
```

**بررسی `OCSP Stapling`:**

```bash
openssl s_client -connect lab.example.com:443 -status -servername lab.example.com </dev/null 2>&1 \
  | grep -A3 "OCSP Response Status"
# OCSP Response Status: successful (0x0)
```

**بررسی `CT`:**

```bash
openssl s_client -connect lab.example.com:443 -servername lab.example.com </dev/null 2>&1 \
  | openssl x509 -noout -text | grep -A2 "CT Precertificate SCTs"
```

**مقایسه‌ی زمان handshake:**

```bash
# TLS 1.2
curl -sS -o /dev/null -w 'tls12: %{time_appconnect}s\n' --tlsv1.2 --tls-max 1.2 https://lab.example.com/

# TLS 1.3
curl -sS -o /dev/null -w 'tls13: %{time_appconnect}s\n' --tlsv1.3 https://lab.example.com/
```

### مقایسه و الگوهای مختلف

**TLS 1.2 در برابر 1.3**

| | 1.2 | 1.3 |
|---|---|---|
| RTT | ۲ | ۱ |
| Cipher | پیچیده | ۳ مجموعه |
| Forward secrecy | ⚠️ اختیاری | ✅ اجباری |
| `0-RTT` | ❌ | ✅ |
| بخش رمزنگاری‌شده | کم | زیاد |
| پشتیبانی | ✅ همه | ✅ مدرن |

**سه سطح پیکربندی**

| سطح | cipher |
|---|---|
| Modern | فقط TLS 1.3 + AES-GCM/ChaCha |
| Intermediate | TLS 1.2 + 1.3، ECDHE |
| Old | + RSA، CBC (❌ توصیه نمی‌شود) |

**`Termination` در برابر `Passthrough`**

| | Termination | Passthrough |
|---|---|---|
| LB محتوا را می‌بیند | ✅ | ❌ |
| TLS در backend | ⚠️ | ✅ |
| کارایی | ✅ | ⚠️ |
| مناسب | اکثر | mTLS انتها-به-انتها |

**دیدگاه «TLS 1.2 قدیمی است»**: هنوز لازم است برای کلاینت‌های قدیمی. TLS 1.0/1.1 را حذف کن.

### اشکال شایع

**۱) `TLSv1`/`TLSv1.1` فعال**
غیرقابل قبول در انطباق. حذف.

**۲) `RSA` key exchange**
بدون forward secrecy. `ECDHE` فقط.

**۳) `3DES`/`RC4`/`CBC`**
ضعیف. حذف.

**۴) `ssl_prefer_server_ciphers on` در TLS 1.3**
بی‌اثر. `off`.

**۵) `SNI` خاموش**
گواهی اشتباه برای دامنه‌ی دوم.

**۶) `OCSP stapling` بدون `resolver`**
بی‌اثر. `resolver` لازم.

**۷) `0-RTT` برای `POST`**
replay. فقط `GET`/`HEAD`.

**۸) `session tickets` بدون چرخش**
forward secrecy ضعیف. `ssl_session_tickets off`.

**۹) گواهی بدون `SCT`**
مرورگر رد می‌کند (برای گواهی جدید).

**۱۰) `verify` خاموش در upstream TLS**
MITM. `proxy_ssl_verify on`.

### نکات مهم (Tips & Tricks)

- **`openssl s_client -tls1_3`** برای تست.
- **`-alpn h2`** برای HTTP/2.
- **`-status`** برای OCSP.
- **`testssl.sh`** برای بررسی جامع.
- **`ssl_session_tickets off`**.
- **`resolver`** برای stapling.
- **Mozilla SSL Config Generator** برای شروع.

### خلاصه فصل

- TLS 1.3 در یک RTT، با forward secrecy اجباری.
- Cipher suite در 1.3 ساده شد.
- `ECDHE` برای forward secrecy.
- `SNI` گواهی درست را انتخاب می‌کند.
- `OCSP stapling` سرعت و خصوصی را بهتر می‌کند.
- `0-RTT` فقط برای idempotent.

**سؤال مروری:** چرا `ECDHE-RSA-AES128-GCM-SHA256` از `AES128-SHA` امن‌تر است؟

---

## فصل ۴۲: Let's Encrypt و ACME با Certbot

### مقدمه

قبل از ۲۰۱۵، گواهی SSL هزینه داشت و دستی تمدید می‌شد. Let's Encrypt آن را رایگان و خودکار کرد. امروز، گواهی منقضی، یک اشتباه غیرقابل توجیه است.

### مفاهیم اساسی

**۱) ACME — پروتکل**

```
کلاینت (certbot) ──▶ ACME server (Let's Encrypt)
                      │
                      ├── چالش اعتبارسنجی
                      └── صدور گواهی
```

**۲) سه نوع چالش**

| چالش | روش | نیاز |
|---|---|---|
| `HTTP-01` | فایل در `/.well-known/acme-challenge/` | پورت ۸۰ باز |
| `DNS-01` | رکورد `TXT` | دسترسی DNS API |
| `TLS-ALPN-01` | ALPN روی ۴۴۳ | پورت ۴۴۳ |

**۳) `HTTP-01` — چطور کار می‌کند**

```
۱. certbot از ACME توکن می‌گیرد
۲. فایل /.well-known/acme-challenge/<token> را می‌سازد
۳. ACME از http://domain/.well-known/... می‌خواند
۴. اگر مطابق بود → صدور
```

**۴) `DNS-01` — برای wildcard**

`*.example.com` فقط با `DNS-01` ممکن است.

**۵) نصب Certbot**

```bash
sudo apt install -y certbot python3-certbot-nginx
# یا
sudo apt install -y certbot python3-certbot-apache
```

**۶) حالت‌های اجرا**

| حالت | دستور |
|---|---|
| Nginx (خودکار) | `certbot --nginx -d domain` |
| Apache (خودکار) | `certbot --apache -d domain` |
| `webroot` | `certbot certonly --webroot -w /var/www -d domain` |
| `standalone` | `certbot certonly --standalone -d domain` |
| `DNS` | `certbot certonly --dns-cloudflare -d '*.domain'` |

**۷) تمدید خودکار**

```bash
systemctl list-timers | grep certbot
sudo certbot renew --dry-run
```

Certbot یک `systemd timer` می‌سازد که روزی دو بار بررسی می‌کند و ۳۰ روز قبل از انقضا تمدید می‌کند.

**۸) `--deploy-hook`**

```bash
sudo certbot renew --deploy-hook "systemctl reload nginx"
```

**۹) محدودیت‌ها**

| محدودیت | مقدار |
|---|---|
| گواهی برای یک دامنه‌ی یکتا | ۵۰ در هفته |
| `Failed validation` | ۵ در ساعت |
| دامنه در گواهی (SAN) | ۱۰۰ |
| `Duplicate certificate` | ۵ در هفته |

**۱۰) staging**

```bash
sudo certbot --nginx -d lab.example.com --staging
```

برای تست، از staging استفاده کن تا محدودیت تولید را نسوزانی.

### مثال‌های عملی

**Lab Scenario: از HTTP تا HTTPS با Certbot**

```bash
# ۰) پیش‌نیاز
dig +short lab.example.com
# باید IP سرور را بدهد

sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# ۱) نصب
sudo apt update && sudo apt install -y certbot python3-certbot-nginx

# ۲) گرفتن گواهی (Nginx)
sudo certbot --nginx -d lab.example.com -d www.lab.example.com \
  --agree-tos -m admin@example.com --redirect

# ۳) بررسی
sudo certbot certificates
curl -sSI https://lab.example.com/ | head -5
```

**حالت `webroot` (بدون تغییر config):**

```bash
sudo mkdir -p /var/www/html/.well-known/acme-challenge

sudo certbot certonly --webroot \
  -w /var/www/html \
  -d lab.example.com \
  --agree-tos -m admin@example.com
```

```nginx
# باید مسیر چالش در دسترس باشد
location ^~ /.well-known/acme-challenge/ {
    root /var/www/html;
    default_type text/plain;
}
```

**حالت `standalone` (بدون وب‌سرور):**

```bash
sudo systemctl stop nginx
sudo certbot certonly --standalone -d lab.example.com --agree-tos -m admin@example.com
sudo systemctl start nginx
```

**wildcard با Cloudflare DNS:**

```bash
sudo apt install -y python3-certbot-dns-cloudflare

sudo tee /root/.secrets/cloudflare.ini >/dev/null <<'EOF'
dns_cloudflare_api_token = YOUR_TOKEN
EOF
sudo chmod 600 /root/.secrets/cloudflare.ini

sudo certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials /root/.secrets/cloudflare.ini \
  -d '*.lab.example.com' -d 'lab.example.com' \
  --agree-tos -m admin@example.com
```

**تمدید و reload خودکار:**

```bash
# تست
sudo certbot renew --dry-run

# deploy hook
sudo tee /etc/letsencrypt/renewal-hooks/deploy/reload-nginx.sh >/dev/null <<'EOF'
#!/bin/sh
nginx -t && systemctl reload nginx
EOF
sudo chmod +x /etc/letsencrypt/renewal-hooks/deploy/reload-nginx.sh

# timer
systemctl list-timers | grep certbot
```

**بررسی تاریخ انقضا:**

```bash
sudo openssl x509 -in /etc/letsencrypt/live/lab.example.com/cert.pem -noout -dates

# یا
echo | openssl s_client -connect lab.example.com:443 -servername lab.example.com 2>/dev/null \
  | openssl x509 -noout -enddate
```

**مانیتورینگ انقضا (اسکریپت):**

```bash
#!/usr/bin/env bash
set -euo pipefail

DAYS=30
for cert in /etc/letsencrypt/live/*/cert.pem; do
  end=$(openssl x509 -in "$cert" -noout -enddate | cut -d= -f2)
  end_epoch=$(date -d "$end" +%s)
  now_epoch=$(date +%s)
  left=$(( (end_epoch - now_epoch) / 86400 ))
  name=$(basename "$(dirname "$cert")")
  if [ "$left" -lt "$DAYS" ]; then
    echo "WARN: $name expires in $left days"
  else
    echo "OK: $name ($left days)"
  fi
done
```

### مقایسه و الگوهای مختلف

**سه چالش ACME**

| | HTTP-01 | DNS-01 | TLS-ALPN-01 |
|---|---|---|---|
| wildcard | ❌ | ✅ | ❌ |
| پورت ۸۰ | ✅ لازم | ❌ | ❌ |
| API DNS | ❌ | ✅ | ❌ |
| مناسب | اکثر | wildcard، داخلی | محدود |

**Certbot در برابر acme.sh در برابر lego**

| | Certbot | acme.sh | lego |
|---|---|---|---|
| زبان | Python | Shell | Go |
| سادگی | ✅ | ✅ | ⚠️ |
| DNS API | ✅ | ✅✅ | ✅ |
| مناسب | ✅ عمومی | اسکریپت | Go |

**`webroot` در برابر `standalone`**

| | `webroot` | `standalone` |
|---|---|---|
| توقف وب‌سرور | ❌ | ✅ |
| نیاز به config | ✅ | ❌ |
| مناسب | ✅ تولید | سرور بدون وب‌سرور |

**دیدگاه «گواهی تجاری بهتر است»**: برای DV، Let's Encrypt کاملاً کافی است. برای OV/EV و پشتیبانی، تجاری.

### اشکال شایع

**۱) `DNS problem: NXDOMAIN`**
دامنه به سرور اشاره نمی‌کند. `dig +short`.

**۲) `Timeout during connect`**
پورت ۸۰ بسته. فایروال.

**۳) `too many failed authorizations`**
محدودیت ۵ در ساعت. صبر یا staging.

**۴) `Certificate not yet due for renewal`**
۳۰ روز قبل. طبیعی.

**۵) `nginx: [emerg]` بعد از certbot**
config دستکاری‌شده. `nginx -t`.

**۶) گواهی برای `www` نیست**
`-d www.example.com` را فراموش کردی.

**۷) تمدید خودکار کار نمی‌کند**
timer خاموش یا `webroot` مسیر اشتباه. `certbot renew --dry-run`.

**۸) `CAA` مانع صدور**
`dig CAA example.com`. رکورد را اضافه/اصلاح کن.

**۹) گواهی wildcard بدون DNS API**
`HTTP-01` پشتیبانی نمی‌کند. `DNS-01`.

**۱۰) `--staging` فراموش‌شده**
گواهی تست، مرورگر رد می‌کند.

### نکات مهم (Tips & Tricks)

- **`certbot --dry-run`** ماهانه.
- **`--deploy-hook`** برای reload.
- **`--staging`** برای تست.
- **`DNS-01`** برای wildcard.
- **`CAA`** را چک کن.
- **مانیتور انقضا**.
- **`renewal-hooks`** برای خودکارسازی.

### خلاصه فصل

- ACME گواهی را خودکار می‌کند.
- سه چالش: HTTP-01، DNS-01، TLS-ALPN-01.
- wildcard فقط با DNS-01.
- Certbot تمدید خودکار با timer.
- `--dry-run` و `--staging` برای تست.
- `CAA` و DNS پیش‌نیاز صدورند.

**سؤال مروری:** چرا گواهی wildcard فقط با `DNS-01` صادر می‌شود؟

---

## فصل ۴۳: پیکربندی TLS در Nginx، Apache و IIS

### مقدمه

گواهی گرفتی؛ حالا باید درست پیکربندی شود. یک خط اشتباه، یا اتصال را می‌شکند یا امنیت را نصف می‌کند. این فصل، پیکربندی «modern» هر سه سرور است.

### مفاهیم اساسی

**۱) Nginx — حداقل**

```nginx
server {
    listen 443 ssl;
    http2 on;
    server_name lab.example.com;

    ssl_certificate     /etc/letsencrypt/live/lab.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/lab.example.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
}
```

**۲) Nginx — کامل**

```nginx
server {
    listen 443 ssl;
    http2 on;
    server_name lab.example.com;

    ssl_certificate     /etc/letsencrypt/live/lab.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/lab.example.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/lab.example.com/chain.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305;
    ssl_prefer_server_ciphers off;
    ssl_ecdh_curve X25519:prime256v1:secp384r1;

    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;

    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 1.1.1.1 8.8.8.8 valid=300s;
    resolver_timeout 5s;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    root /var/www/lab;
    location / { try_files $uri $uri/ =404; }
}
```

**۳) Apache — کامل**

```apache
<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerName lab.example.com
    DocumentRoot /var/www/lab

    SSLEngine on
    SSLCertificateFile      /etc/letsencrypt/live/lab.example.com/fullchain.pem
    SSLCertificateKeyFile   /etc/letsencrypt/live/lab.example.com/privkey.pem
    SSLCertificateChainFile /etc/letsencrypt/live/lab.example.com/chain.pem

    SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
    SSLCipherSuite ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384
    SSLHonorCipherOrder off

    SSLSessionTickets off
    SSLSessionCache "shmcb:/var/run/apache2/ssl_scache(512000)"

    SSLUseStapling on
    SSLStaplingCache "shmcb:logs/ssl_stapling(32768)"

    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"

    Protocols h2 http/1.1
</VirtualHost>
</IfModule>
```

```bash
sudo a2enmod ssl headers http2
sudo apachectl configtest && sudo systemctl reload apache2
```

**۴) IIS — PowerShell**

```powershell
# ۱) گواهی نصب‌شده
$cert = Get-ChildItem Cert:\LocalMachine\My | Where-Object { $_.Subject -like "*lab.example.com*" }

# ۲) binding
New-WebBinding -Name "lab" -Protocol https -Port 443 -HostHeader "lab.example.com" -SslFlags 1
(Get-WebBinding -Name "lab" -Protocol https -HostHeader "lab.example.com").
  AddSslCertificate($cert.Thumbprint, "my")

# ۳) پروتکل‌ها (رجیستری)
$path = "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols"
foreach ($p in @("TLS 1.0","TLS 1.1")) {
    New-Item -Path "$path\$p\Server" -Force | Out-Null
    New-ItemProperty -Path "$path\$p\Server" -Name "Enabled" -Value 0 -PropertyType DWord -Force | Out-Null
    New-ItemProperty -Path "$path\$p\Server" -Name "DisabledByDefault" -Value 1 -PropertyType DWord -Force | Out-Null
}
foreach ($p in @("TLS 1.2","TLS 1.3")) {
    New-Item -Path "$path\$p\Server" -Force | Out-Null
    New-ItemProperty -Path "$path\$p\Server" -Name "Enabled" -Value 1 -PropertyType DWord -Force | Out-Null
    New-ItemProperty -Path "$path\$p\Server" -Name "DisabledByDefault" -Value 0 -PropertyType DWord -Force | Out-Null
}

# ۴) cipher suite (با IISCrypto ساده‌تر است)
```

> برای IIS، ابزار `IIS Crypto` (Nartac) امن‌ترین راه تنظیم cipher suite و پروتکل است.

**۵) `ssl_certificate` در برابر `fullchain`**

| فایل | محتوا |
|---|---|
| `cert.pem` | فقط گواهی |
| `chain.pem` | intermediate |
| `fullchain.pem` | گواهی + intermediate ✅ |
| `privkey.pem` | کلید خصوصی |

همیشه `fullchain`.

**۶) `ssl_trusted_certificate`**

برای `OCSP stapling` و `verify` کلاینت، Nginx باید زنجیره را بداند.

**۷) `ssl_session_cache`**

| مقدار | معنا |
|---|---|
| `off` | بدون کش |
| `builtin:1000` | کش per-worker |
| `shared:SSL:10m` | ✅ مشترک |

**۸) `ssl_session_tickets`**

بدون چرخش کلید، forward secrecy را ضعیف می‌کند. `off` امن‌تر.

**۹) `ssl_ecdh_curve`**

`X25519` سریع‌ترین و امن.

**۱۰) `ssl_buffer_size`**

برای فایل بزرگ، `4k` می‌تواند TTFB را بهتر کند.

### مثال‌های عملی

**Lab Scenario: پیکربندی کامل Nginx با تست**

```bash
# ۱) گواهی
sudo certbot certonly --nginx -d lab.example.com

# ۲) config
sudo tee /etc/nginx/sites-available/lab-tls.conf >/dev/null <<'EOF'
server {
    listen 80;
    server_name lab.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    http2 on;
    server_name lab.example.com;

    ssl_certificate         /etc/letsencrypt/live/lab.example.com/fullchain.pem;
    ssl_certificate_key     /etc/letsencrypt/live/lab.example.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/lab.example.com/chain.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305;
    ssl_prefer_server_ciphers off;
    ssl_ecdh_curve X25519:prime256v1:secp384r1;

    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;

    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 1.1.1.1 8.8.8.8 valid=300s;
    resolver_timeout 5s;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;

    root /var/www/lab;
    index index.html;
    location / { try_files $uri $uri/ =404; }
}
EOF

sudo ln -sf /etc/nginx/sites-available/lab-tls.conf /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t && sudo systemctl reload nginx
```

```bash
# ۳) تست
curl -sSI http://lab.example.com/ | head -3
# HTTP/1.1 301 Moved Permanently
# Location: https://lab.example.com/

curl -sSI https://lab.example.com/ | head -8
# HTTP/2 200
# strict-transport-security: max-age=31536000; includeSubDomains

# ۴) پروتکل‌ها
nmap --script ssl-enum-ciphers -p 443 lab.example.com | grep -E "TLSv|ciphers"

# ۵) testssl
docker run --rm -it drwetter/testssl.sh --quiet https://lab.example.com
```

**Apache:**

```bash
sudo tee /etc/apache2/sites-available/lab-tls.conf >/dev/null <<'EOF'
<VirtualHost *:80>
    ServerName lab.example.com
    Redirect permanent / https://lab.example.com/
</VirtualHost>

<IfModule mod_ssl.c>
<VirtualHost *:443>
    ServerName lab.example.com
    DocumentRoot /var/www/lab

    SSLEngine on
    SSLCertificateFile      /etc/letsencrypt/live/lab.example.com/fullchain.pem
    SSLCertificateKeyFile   /etc/letsencrypt/live/lab.example.com/privkey.pem
    SSLCertificateChainFile /etc/letsencrypt/live/lab.example.com/chain.pem

    SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
    SSLCipherSuite ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384
    SSLHonorCipherOrder off
    SSLSessionTickets off

    SSLUseStapling on
    SSLStaplingCache "shmcb:logs/ssl_stapling(32768)"

    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"

    Protocols h2 http/1.1
</VirtualHost>
</IfModule>
EOF

sudo a2ensite lab-tls
sudo apachectl configtest && sudo systemctl reload apache2
```

**IIS:**

```powershell
# بررسی
Get-WebBinding -Name "lab" | Format-Table protocol, bindingInformation, sslFlags
Get-ChildItem Cert:\LocalMachine\My | Select-Object Subject, NotAfter

# تست
curl.exe -sSI https://lab.example.com/ | Select-Object -First 5
```

### مقایسه و الگوهای مختلف

**سه سطح Mozilla**

| | Modern | Intermediate | Old |
|---|---|---|---|
| TLS 1.2 | ❌ | ✅ | ✅ |
| TLS 1.3 | ✅ | ✅ | ✅ |
| کلاینت | مدرن | اکثر | قدیمی |
| توصیه | API داخلی | ✅ عمومی | ❌ |

**`ssl_prefer_server_ciphers on` در برابر `off`**

| | `on` | `off` |
|---|---|---|
| TLS 1.2 | سرور انتخاب | کلاینت |
| TLS 1.3 | بی‌اثر | بی‌اثر |
| توصیه | ⚠️ | ✅ `off` |

**`session_tickets off` در برابر `on`**

| | `off` | `on` |
|---|---|---|
| forward secrecy | ✅ | ⚠️ |
| سرعت handshake مجدد | ⚠️ | ✅ |
| توصیه | ✅ | با چرخش |

**دیدگاه «پیکربندی TLS پیچیده است»**: از Mozilla SSL Config Generator شروع کن، بعد تست. بیشتر خطاها از گواهی ناقص و پروتکل قدیمی می‌آید.

### اشکال شایع

**۱) `cert.pem` به‌جای `fullchain.pem`**
زنجیره ناقص، خطا در بعضی کلاینت‌ها.

**۲) `ssl_protocols TLSv1`**
ضعیف. حذف.

**۳) `ssl_ciphers` قدیمی**
`3DES`/`RC4`. حذف.

**۴) `ssl_stapling on` بدون `resolver`**
خطا در لاگ، بی‌اثر.

**۵) `listen 443 ssl http2;` در نسخه‌ی جدید**
هشدار. `http2 on;`.

**۶) `SSLCertificateChainFile` در Apache 2.4.8+**
منسوخ؛ `fullchain` در `SSLCertificateFile`.

**۷) IIS و TLS 1.0**
رجیستری. `IIS Crypto`.

**۸) `HSTS` قبل از تست**
برگشت‌ناپذیر. اول `max-age=300`.

**۹) گواهی منقضی**
`certbot renew` شکست خورده.

**۱۰) `resolver` در Nginx و `OCSP` کند**
`resolver_timeout 5s`.

### نکات مهم (Tips & Tricks)

- **`fullchain.pem`** همیشه.
- **`TLSv1.2 TLSv1.3`**.
- **`ssl_session_tickets off`**.
- **`resolver`** برای stapling.
- **Mozilla SSL Config Generator**.
- **`testssl.sh`** قبل از تحویل.
- **`IIS Crypto`** برای ویندوز.

### خلاصه فصل

- `fullchain.pem` برای زنجیره‌ی کامل.
- فقط TLS 1.2 و 1.3.
- `ECDHE` + AES-GCM/ChaCha.
- `session_tickets off` برای forward secrecy.
- `OCSP stapling` با `resolver`.
- IIS با رجیستری و `IIS Crypto`.

**سؤال مروری:** چرا `ssl_prefer_server_ciphers on` در TLS 1.3 بی‌اثر است؟

---

## فصل ۴۴: HSTS، OCSP Stapling، Redirect و mTLS

### مقدمه

TLS بدون این چهار، نصفه است: **HSTS** از downgrade جلوگیری می‌کند، **OCSP** سرعت و خصوصی می‌دهد، **redirect** کاربر را به HTTPS می‌برد، و **mTLS** هویت کلاینت را تأیید می‌کند.

### مفاهیم اساسی

**۱) HSTS**

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

| پارامتر | معنا |
|---|---|
| `max-age` | مدت اعتبار (ثانیه) |
| `includeSubDomains` | همه‌ی زیردامنه‌ها |
| `preload` | درخواست ورود به لیست preload مرورگر |

**۲) خطر HSTS**

اگر `includeSubDomains` بگذاری و زیردامنه‌ای HTTPS نداشته باشد، از دسترس خارج می‌شود. و `preload` **برگشت‌ناپذیر** است (ماه‌ها).

**۳) استراتژی HSTS**

```
هفته ۱: max-age=300
هفته ۲: max-age=86400
هفته ۳: max-age=31536000
هفته ۴: + includeSubDomains
هفته ۵: + preload (اختیاری)
```

**۴) OCSP Stapling**

```nginx
ssl_stapling on;
ssl_stapling_verify on;
resolver 1.1.1.1 8.8.8.8 valid=300s;
ssl_trusted_certificate /etc/letsencrypt/live/lab.example.com/chain.pem;
```

**۵) `Must-Staple`**

گواهی می‌تواند اعلام کند «stapling اجباری است». مرورگر بدون staple، اتصال را رد می‌کند.

**۶) Redirect HTTP → HTTPS**

```nginx
server {
    listen 80;
    server_name lab.example.com;
    return 301 https://$host$request_uri;
}
```

**۷) `X-Forwarded-Proto` و redirect loop**

پشت پروکسی HTTPS، `$scheme` = `http` است. باید `X-Forwarded-Proto` را بخوانی (→ فصل ۳۴).

**۸) mTLS — احراز هویت کلاینت**

```nginx
server {
    listen 443 ssl;
    ssl_client_certificate /etc/nginx/client-ca.crt;
    ssl_verify_client on;
    ssl_verify_depth 2;

    location / {
        add_header X-Client-DN $ssl_client_s_dn always;
        add_header X-Client-Verify $ssl_client_verify always;
    }
}
```

| `ssl_verify_client` | معنا |
|---|---|
| `off` | بدون تأیید |
| `on` | اجباری |
| `optional` | اگر آمد، تأیید کن |
| `optional_no_ca` | اگر آمد، تأیید نکن (نادر) |

**۹) Apache mTLS**

```apache
SSLVerifyClient require
SSLVerifyDepth 2
SSLCACertificateFile /etc/apache/client-ca.crt
```

**۱۰) IIS mTLS**

```powershell
Set-WebConfigurationProperty -PSPath "IIS:\Sites\lab" `
  -Filter "system.webServer/security/access" `
  -Name "sslFlags" -Value "Ssl,SslNegotiateCert,SslRequireCert"
```

### مثال‌های عملی

**Lab Scenario: mTLS برای API داخلی**

```bash
# ۱) CA
openssl req -new -x509 -days 3650 -nodes -out ca.crt -keyout ca.key \
  -subj "/CN=Lab Internal CA"

# ۲) گواهی سرور
openssl req -new -nodes -out server.csr -keyout server.key -subj "/CN=api.lab.example.com"
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial \
  -out server.crt -days 365 -sha256

# ۳) گواهی کلاینت
openssl req -new -nodes -out client.csr -keyout client.key -subj "/CN=service-a"
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial \
  -out client.crt -days 365 -sha256
```

```nginx
server {
    listen 8443 ssl;
    server_name api.lab.example.com;

    ssl_certificate     /etc/ssl/lab/server.crt;
    ssl_certificate_key /etc/ssl/lab/server.key;

    ssl_client_certificate /etc/ssl/lab/ca.crt;
    ssl_verify_client on;
    ssl_verify_depth 2;

    location / {
        default_type text/plain;
        return 200 "client=$ssl_client_s_dn verify=$ssl_client_verify\n";
    }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx

# بدون گواهی
curl -sk https://localhost:8443/
# 400 No required SSL certificate was sent

# با گواهی
curl -sk --cert client.crt --key client.key https://localhost:8443/
# client=/CN=service-a verify=SUCCESS
```

**HSTS مرحله‌ای:**

```nginx
# مرحله ۱
add_header Strict-Transport-Security "max-age=300" always;

# مرحله ۲ (بعد از تست)
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

```bash
curl -sSI https://lab.example.com/ | grep -i strict-transport
```

**OCSP Stapling:**

```nginx
ssl_stapling on;
ssl_stapling_verify on;
resolver 1.1.1.1 8.8.8.8 valid=300s;
resolver_timeout 5s;
ssl_trusted_certificate /etc/letsencrypt/live/lab.example.com/chain.pem;
```

```bash
openssl s_client -connect lab.example.com:443 -status -servername lab.example.com </dev/null 2>&1 \
  | grep -A2 "OCSP Response Status"
# OCSP Response Status: successful (0x0)
```

**Redirect امن پشت پروکسی:**

```nginx
map $http_x_forwarded_proto $scheme_final {
    default $scheme;
    https   https;
}

server {
    listen 80;
    if ($scheme_final != "https") {
        return 301 https://$host$request_uri;
    }
}
```

**mTLS در Apache:**

```apache
<VirtualHost *:8443>
    ServerName api.lab.example.com
    SSLEngine on
    SSLCertificateFile    /etc/ssl/lab/server.crt
    SSLCertificateKeyFile /etc/ssl/lab/server.key

    SSLVerifyClient require
    SSLVerifyDepth 2
    SSLCACertificateFile /etc/ssl/lab/ca.crt

    <Location />
        Require ssl-verify-client
    </Location>
</VirtualHost>
```

**mTLS در IIS:**

```powershell
Set-WebConfigurationProperty -PSPath "IIS:\Sites\api" `
  -Filter "system.webServer/security/access" `
  -Name "sslFlags" -Value "Ssl,SslNegotiateCert,SslRequireCert"

# CA باید در Trusted Root نصب باشد
Import-Certificate -FilePath C:\certs\ca.crt -CertStoreLocation Cert:\LocalMachine\Root
```

### مقایسه و الگوهای مختلف

**`on` در برابر `optional` در `ssl_verify_client`**

| | `on` | `optional` |
|---|---|---|
| بدون گواهی | ❌ رد | ✅ قبول |
| مناسب | API داخلی | وب + API |

**HSTS `includeSubDomains`**

| | بدون | با |
|---|---|---|
| امنیت | ⚠️ | ✅ |
| خطر | ✅ کم | ❌ بالا (زیردامنه‌ی HTTP) |
| توصیه | شروع | بعد از اطمینان |

**OCSP Stapling در برابر OCSP کلاینت**

| | Stapling | کلاینت مستقیم |
|---|---|---|
| سرعت | ✅ | ❌ |
| خصوصی | ✅ | ❌ |
| نیاز به `resolver` | ✅ | ❌ |

**دیدگاه «mTLS جای توکن را می‌گیرد»**: mTLS هویت سرویس را در لایه‌ی TLS می‌دهد، ولی authorization و انقضا نیاز به لایه‌ی اپ دارد. مکمل، نه جایگزین.

### اشکال شایع

**۱) `preload` قبل از تست**
برگشت‌ناپذیر. اول `max-age=300`.

**۲) `includeSubDomains` با زیردامنه‌ی HTTP**
از دسترس خارج می‌شود.

**۳) `ssl_stapling` بدون `ssl_trusted_certificate`**
بی‌اثر یا خطا.

**۴) `resolver` نبودن**
stapling کار نمی‌کند.

**۵) redirect loop پشت پروکسی**
`X-Forwarded-Proto` نادیده.

**۶) `ssl_verify_client on` برای همه**
کاربر عادی نمی‌تواند. `optional` یا `location` جدا.

**۷) CA در `ssl_client_certificate` ناقص**
زنجیره. `ca.crt` کامل.

**۸) `ssl_verify_depth` کم**
زنجیره‌ی میانی رد. `2`.

**۹) HSTS روی HTTP**
هدر فقط روی HTTPS اثر دارد.

**۱۰) `Must-Staple` بدون stapling**
مرورگر رد می‌کند.

### نکات مهم (Tips & Tricks)

- **HSTS مرحله‌ای**.
- **`resolver`** برای stapling.
- **`ssl_trusted_certificate`**.
- **`X-Forwarded-Proto`** برای redirect.
- **`optional`** برای mTLS تدریجی.
- **`ssl_verify_depth 2`**.
- **`curl --cert`** برای تست mTLS.

### خلاصه فصل

- HSTS از downgrade جلوگیری می‌کند، ولی برگشت‌ناپذیر است.
- OCSP stapling نیاز به `resolver` و زنجیره دارد.
- redirect پشت پروکسی نیاز به `X-Forwarded-Proto`.
- mTLS هویت کلاینت را در TLS تأیید می‌کند.
- `optional` برای مهاجرت تدریجی.
- `ssl_verify_depth` و CA کامل الزامی.

**سؤال مروری:** چرا `includeSubDomains` در HSTS می‌تواند یک زیردامنه را از دسترس خارج کند؟

---

## فصل ۴۵: مدیریت گواهی، چرخش و تست

### مقدمه

گواهی یک فایل ثابت نیست؛ چرخه دارد: صدور، نصب، تمدید، چرخش، ابطال. و هر مرحله می‌تواند شکست بخورد. این فصل، فرایند و ابزار تست است.

### مفاهیم اساسی

**۱) چرخه‌ی عمر**

```
صدور → نصب → فعال → تمدید (۳۰ روز قبل) → چرخش → ابطال (در صورت نیاز)
```

**۲) ابطال (Revocation)**

| روش | توضیح |
|---|---|
| CRL | لیست ابطال (فایل) |
| OCSP | پرسش آنلاین |
| OCSP Stapling | سرور پاسخ را می‌دهد |

**۳) `certbot revoke`**

```bash
sudo certbot revoke --cert-path /etc/letsencrypt/live/lab.example.com/cert.pem
sudo certbot delete --cert-name lab.example.com
```

**۴) چرخش بدون قطع**

```
۱. گواهی جدید بگیر (نام دیگر یا همان)
۲. config را به گواهی جدید اشاره بده
۳. reload
۴. گواهی قدیمی را حذف کن (بعد از مدتی)
```

**۵) `multi-domain` و `SAN`**

یک گواهی برای چند دامنه. مدیریت ساده‌تر، ولی انقضای مشترک.

**۶) `wildcard`**

`*.example.com` فقط یک سطح را پوشش می‌دهد (`a.example.com`، نه `a.b.example.com`).

**۷) تست — سه سطح**

| سطح | ابزار |
|---|---|
| سریع | `openssl s_client` |
| جامع | `testssl.sh` |
| آنلاین | SSL Labs |

**۸) `testssl.sh`**

```bash
docker run --rm -it drwetter/testssl.sh https://lab.example.com
docker run --rm -it drwetter/testssl.sh --severity HIGH https://lab.example.com
```

**۹) SSL Labs**

نمره‌ی A+ نیاز به: TLS 1.2+، HSTS، forward secrecy، بدون آسیب‌پذیری شناخته‌شده.

**۱۰) مانیتورینگ انقضا**

```bash
# ساده
for c in /etc/letsencrypt/live/*/cert.pem; do
  echo "$(basename $(dirname $c)): $(openssl x509 -in $c -noout -enddate | cut -d= -f2)"
done
```

یا با Prometheus `blackbox_exporter` (ماژول `https` + `certificate`).

### مثال‌های عملی

**Lab Scenario: چرخه‌ی کامل و تست**

```bash
# ۱) وضعیت
sudo certbot certificates
# Certificate Name: lab.example.com
#   Domains: lab.example.com www.lab.example.com
#   Expiry Date: 2026-12-20
#   Certificate Path: /etc/letsencrypt/live/lab.example.com/fullchain.pem

# ۲) تست تمدید
sudo certbot renew --dry-run

# ۳) force renew (برای تست)
sudo certbot renew --force-renewal --cert-name lab.example.com

# ۴) بررسی پس از تمدید
sudo openssl x509 -in /etc/letsencrypt/live/lab.example.com/cert.pem -noout -dates
sudo systemctl reload nginx
```

**تست جامع:**

```bash
# ۱) زنجیره و تاریخ
echo | openssl s_client -connect lab.example.com:443 -servername lab.example.com 2>/dev/null \
  | openssl x509 -noout -subject -issuer -dates -ext subjectAltName

# ۲) پروتکل‌ها
nmap --script ssl-enum-ciphers -p 443 lab.example.com

# ۳) testssl
docker run --rm -it drwetter/testssl.sh --quiet --severity HIGH https://lab.example.com

# ۴) HSTS
curl -sSI https://lab.example.com/ | grep -i strict-transport

# ۵) OCSP
openssl s_client -connect lab.example.com:443 -status -servername lab.example.com </dev/null 2>&1 \
  | grep -A2 "OCSP Response Status"

# ۶) HTTP/2
curl -sSI --http2 https://lab.example.com/ | head -1
```

**چرخش گواهی (بدون قطع):**

```bash
# ۱) گواهی جدید با نام دیگر
sudo certbot certonly --nginx -d lab.example.com -d www.lab.example.com \
  --cert-name lab-new --force-renewal

# ۲) تست config با گواهی جدید (بدون reload)
sudo sed -i 's|/etc/letsencrypt/live/lab.example.com/|/etc/letsencrypt/live/lab-new/|g' \
  /etc/nginx/sites-available/lab-tls.conf
sudo nginx -t

# ۳) reload
sudo systemctl reload nginx

# ۴) بررسی
curl -sSI https://lab.example.com/ | head -1

# ۵) حذف گواهی قدیمی
sudo certbot delete --cert-name lab.example.com
```

**مانیتورینگ انقضا با اسکریپت:**

```bash
#!/usr/bin/env bash
set -euo pipefail

WARN_DAYS=30
FAIL=0

for cert in /etc/letsencrypt/live/*/cert.pem; do
  [ -f "$cert" ] || continue
  name=$(basename "$(dirname "$cert")")
  end=$(openssl x509 -in "$cert" -noout -enddate | cut -d= -f2)
  end_epoch=$(date -d "$end" +%s)
  now=$(date +%s)
  left=$(( (end_epoch - now) / 86400 ))

  if [ "$left" -lt "$WARN_DAYS" ]; then
    echo "CRITICAL: $name expires in $left days ($end)"
    FAIL=1
  else
    echo "OK: $name ($left days)"
  fi
done

exit $FAIL
```

```bash
chmod +x /usr/local/bin/check-certs.sh
# cron روزانه
echo '0 6 * * * /usr/local/bin/check-certs.sh | mail -s "cert status" admin@example.com' | sudo crontab -
```

**Prometheus `blackbox_exporter`:**

```yaml
modules:
  https_cert:
    prober: http
    http:
      tls_config:
        ca_file: /etc/ssl/certs/ca-certificates.crt
      preferred_ip_protocol: ip4
```

```yaml
- job_name: certs
  metrics_path: /probe
  params:
    module: [https_cert]
  static_configs:
    - targets:
        - https://lab.example.com
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - target_label: __address__
      replacement: blackbox:9115
```

هشدار:

```yaml
- alert: CertificateExpiringSoon
  expr: probe_ssl_earliest_cert_expiry - time() < 86400 * 14
  for: 1h
  labels:
    severity: warning
```

**مقایسه‌ی گواهی‌ها:**

```bash
for d in lab.example.com api.lab.example.com; do
  echo "=== $d"
  echo | openssl s_client -connect "$d:443" -servername "$d" 2>/dev/null \
    | openssl x509 -noout -subject -enddate
done
```

### مقایسه و الگوهای مختلف

**گواهی تک‌دامنه در برابر SAN در برابر wildcard**

| | تک | SAN | wildcard |
|---|---|---|---|
| دامنه | ۱ | چند | همه‌ی زیردامنه‌ها |
| مدیریت | ⚠️ | ✅ | ✅ |
| انقضای مشترک | ✅ | ⚠️ | ⚠️ |
| انعطاف | ❌ | ✅ | ✅ |

**`certbot` در برابر `acme.sh` در برابر مدیریت‌شده**

| | certbot | acme.sh | ابری |
|---|---|---|---|
| سادگی | ✅ | ✅ | ✅✅ |
| DNS API | ✅ | ✅✅ | ✅ |
| کنترل | ✅ | ✅ | ⚠️ |
| مناسب | ✅ | اسکریپت | تیم کوچک |

**سه ابزار تست**

| ابزار | سرعت | عمق |
|---|---|---|
| `openssl s_client` | ✅ | ⚠️ |
| `testssl.sh` | ⚠️ | ✅✅ |
| SSL Labs | ⚠️ | ✅✅ |

**دیدگاه «تمدید خودکار کافی است»**: تمدید خودکار کار می‌کند، ولی اگر شکست بخورد، کسی نمی‌فهمد. مانیتورینگ لازم است.

### اشکال شایع

**۱) تمدید خودکار شکست خورده**
`certbot renew --dry-run` ماهانه.

**۲) `deploy-hook` نبودن**
گواهی تمدید شده، ولی Nginx reload نشده.

**۳) `webroot` مسیر اشتباه**
چالش شکست می‌خورد.

**۴) گواهی `staging` در تولید**
مرورگر رد می‌کند.

**۵) حذف گواهی فعال**
`certbot delete` بدون بررسی. `certbot certificates` اول.

**۶) SAN فراموش‌شده**
`www` پوشش داده نمی‌شود.

**۷) `wildcard` برای دو سطح**
`*.example.com` شامل `a.b.example.com` نیست.

**۸) بدون مانیتورینگ انقضا**
غافلگیری.

**۹) `openssl` قدیمی**
پروتکل‌های جدید را نمی‌بیند. به‌روزرسانی.

**۱۰) تست فقط با `curl`**
`curl` همه‌ی مشکلات TLS را نشان نمی‌دهد. `testssl`.

### نکات مهم (Tips & Tricks)

- **`certbot renew --dry-run`** ماهانه.
- **`deploy-hook`** برای reload.
- **`testssl.sh`** قبل از تحویل.
- **مانیتور انقضا**.
- **`openssl x509 -enddate`** سریع.
- **`blackbox_exporter`** برای Prometheus.
- **`certbot certificates`** برای وضعیت.

### خلاصه فصل

- گواهی چرخه دارد: صدور، تمدید، چرخش، ابطال.
- تمدید خودکار بدون مانیتورینگ کافی نیست.
- `deploy-hook` برای reload.
- `testssl.sh` و SSL Labs برای تست جامع.
- SAN و wildcard مدیریت را ساده می‌کنند.
- چرخش بدون قطع با گواهی جدید و reload.

**سؤال مروری:** چرا `certbot renew` موفق، ولی سایت هنوز گواهی قدیمی نشان می‌دهد؟

---

## منابع و مراجع

**مستندات رسمی**
- [RFC 8446 — TLS 1.3](https://www.rfc-editor.org/rfc/rfc8446.html)
- [Nginx — SSL module](https://nginx.org/en/docs/http/ngx_http_ssl_module.html)
- [Apache — mod_ssl](https://httpd.apache.org/docs/2.4/mod/mod_ssl.html)
- [Let's Encrypt Docs](https://letsencrypt.org/docs/)
- [Certbot Docs](https://eff-certbot.readthedocs.io/)
- [IIS — SSL/TLS settings](https://learn.microsoft.com/en-us/iis/configuring-ssl-on-iis)

**ابزار**
- [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/)
- [testssl.sh](https://testssl.sh/)
- [SSL Labs Server Test](https://www.ssllabs.com/ssltest/)
- [IIS Crypto](https://www.nartac.com/Products/IISCrypto)
- [acme.sh](https://github.com/acmesh-official/acme.sh)

**آموزش**
- [Bulletproof SSL and TLS — Ristic](https://www.feistyduck.com/books/bulletproof-ssl-and-tls/)
- [HSTS Preload](https://hstspreload.org/)
- [OCSP Stapling — Nginx](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_stapling)

---

**بعدی:** [`11-Logs-and-Monitoring.md`](11-Logs-and-Monitoring.md) — فرمت لاگ، تحلیل، `stub_status` و متریک
