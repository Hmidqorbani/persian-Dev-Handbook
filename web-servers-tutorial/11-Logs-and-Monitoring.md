# بخش ۱۱: لاگ و مانیتورینگ

**مدت یادگیری**: ۴ تا ۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۰۵ (Nginx)، بخش ۰۴ (Apache)، بخش ۰۶ (IIS)

---

## فهرست محتوا

- [فصل ۴۶: لاگ در Nginx — فرمت، متغیر و چرخش](#فصل-۴۶)
- [فصل ۴۷: لاگ در Apache و IIS](#فصل-۴۷)
- [فصل ۴۸: تحلیل لاگ — `GoAccess`، `awk` و ELK](#فصل-۴۸)
- [فصل ۴۹: مانیتورینگ — `stub_status` و متریک](#فصل-۴۹)

---

## فصل ۴۶: لاگ در Nginx — فرمت، متغیر و چرخش

### مقدمه

لاگ، حافظه‌ی وب‌سرور است. بدون آن، هر حادثه یک معما است. با آن، می‌توانی بگویی «ساعت ۱۴:۳۲، از IP فلان، `POST /api/orders` با ۵۰۲ و زمان ۳۰ ثانیه». این فصل، طراحی لاگ است، نه فقط روشن کردنش.

### مفاهیم اساسی

**۱) دو لاگ**

| لاگ | محتوا |
|---|---|
| `access.log` | هر درخواست |
| `error.log` | خطا و هشدار |

**۲) `log_format` — متغیرهای کلیدی**

| متغیر | معنا |
|---|---|
| `$remote_addr` | IP کلاینت |
| `$remote_user` | کاربر (Basic auth) |
| `$time_local` | زمان محلی |
| `$request` | خط درخواست |
| `$status` | کد وضعیت |
| `$body_bytes_sent` | حجم پاسخ |
| `$http_referer` | ارجاع‌دهنده |
| `$http_user_agent` | UA |
| `$request_time` | زمان کل (ثانیه) |
| `$upstream_response_time` | زمان upstream |
| `$upstream_addr` | آدرس upstream |
| `$upstream_status` | کد upstream |
| `$request_id` | شناسه‌ی یکتا |
| `$ssl_protocol` | نسخه‌ی TLS |
| `$ssl_cipher` | cipher |
| `$server_name` | دامنه |
| `$host` | Host |
| `$request_length` | حجم درخواست |

**۳) فرمت پیشنهادی**

```nginx
log_format main '$remote_addr - $remote_user [$time_local] '
                '"$request" $status $body_bytes_sent '
                '"$http_referer" "$http_user_agent" '
                'rt=$request_time urt=$upstream_response_time '
                'ua=$upstream_addr us=$upstream_status '
                'rid=$request_id';

log_format json_combined escape=json
'{'
  '"time":"$time_iso8601",'
  '"remote_addr":"$remote_addr",'
  '"method":"$request_method",'
  '"uri":"$request_uri",'
  '"status":$status,'
  '"bytes":$body_bytes_sent,'
  '"rt":$request_time,'
  '"urt":"$upstream_response_time",'
  '"upstream":"$upstream_addr",'
  '"ua":"$http_user_agent",'
  '"ref":"$http_referer",'
  '"host":"$host",'
  '"rid":"$request_id",'
  '"proto":"$server_protocol"'
'}';
```

**۴) `$request_time` در برابر `$upstream_response_time`**

```
$request_time = کل زمان از دریافت اولین بایت تا ارسال آخرین بایت
$upstream_response_time = زمان صرف‌شده در upstream
```

اگر `$request_time` بزرگ و `$upstream_response_time` کوچک است، مشکل در Nginx/شبکه/کلاینت است. برعکسش، مشکل در اپ.

**۵) `$upstream_addr` با چند تلاش**

```
upstream_addr: "10.0.0.11:8000, 10.0.0.12:8000"
upstream_status: "502, 200"
```

اولی شکست، دومی موفق. برای دیباگ `proxy_next_upstream` حیاتی.

**۶) `error_log` — سطح‌ها**

```nginx
error_log /var/log/nginx/error.log warn;
```

| سطح | کاربرد |
|---|---|
| `debug` | دیباگ عمیق (حجم زیاد) |
| `info` | اطلاعات |
| `notice` | ✅ تولید |
| `warn` | ✅ تولید (پیش‌فرض) |
| `error` | خطا |
| `crit` | بحرانی |

**۷) `error_log` per-site**

```nginx
server {
    error_log /var/log/nginx/lab-error.log warn;
    access_log /var/log/nginx/lab-access.log main;
}
```

**۸) `access_log off` برای استاتیک**

```nginx
location ~* \.(css|js|jpg|png|woff2)$ {
    access_log off;
}
```

**۹) `log_not_found off`**

```nginx
log_not_found off;   # برای favicon و ربات‌ها
```

**۱۰) `open_log_file_cache`**

```nginx
open_log_file_cache max=1000 inactive=20s valid=1m min_uses=2;
```

**۱۱) چرخش با `logrotate`**

```bash
sudo tee /etc/logrotate.d/nginx >/dev/null <<'EOF'
/var/log/nginx/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 www-data adm
    sharedscripts
    postrotate
        [ -f /var/run/nginx.pid ] && kill -USR1 $(cat /var/run/nginx.pid)
    endscript
}
EOF
```

`kill -USR1` به Nginx می‌گوید فایل لاگ را باز کن (بدون restart).

**۱۲) `map` برای فیلد اضافه**

```nginx
map $http_x_request_id $rid {
    default $request_id;
    ~.+     $http_x_request_id;
}
```

### مثال‌های عملی

**Lab Scenario: لاگ ساختاریافته برای دیباگ**

```nginx
# /etc/nginx/conf.d/logging.conf
log_format json_main escape=json
'{'
  '"ts":"$time_iso8601",'
  '"ip":"$remote_addr",'
  '"xff":"$http_x_forwarded_for",'
  '"method":"$request_method",'
  '"uri":"$request_uri",'
  '"status":$status,'
  '"bytes":$body_bytes_sent,'
  '"rt":$request_time,'
  '"urt":"$upstream_response_time",'
  '"upstream":"$upstream_addr",'
  '"ustatus":"$upstream_status",'
  '"host":"$host",'
  '"ua":"$http_user_agent",'
  '"ref":"$http_referer",'
  '"rid":"$request_id",'
  '"proto":"$server_protocol"'
'}';

access_log /var/log/nginx/access.json.log json_main buffer=32k flush=5s;
error_log  /var/log/nginx/error.log warn;
```

> `buffer=32k flush=5s` نوشتن دیسک را کم می‌کند. در حادثه، ممکن است تا ۵ ثانیه لاگ نباشد.

```bash
sudo nginx -t && sudo systemctl reload nginx

curl -sS http://localhost:8080/ >/dev/null
curl -sS http://localhost:8080/missing >/dev/null

sudo tail -2 /var/log/nginx/access.json.log | python -m json.tool
```

خروجی نمونه:

```json
{
  "ts": "2026-10-22T14:30:01+03:30",
  "ip": "127.0.0.1",
  "method": "GET",
  "uri": "/",
  "status": 200,
  "bytes": 612,
  "rt": 0.001,
  "urt": "0.001",
  "upstream": "127.0.0.1:8000",
  "ustatus": "200",
  "host": "localhost",
  "rid": "..."
}
```

**پیدا کردن کندترین درخواست‌ها:**

```bash
# با jq
sudo cat /var/log/nginx/access.json.log | jq -r 'select(.rt > 1) | "\(.rt)\t\(.status)\t\(.uri)"' | sort -rn | head -10

# با awk روی فرمت متنی
sudo awk '{for(i=1;i<=NF;i++) if($i ~ /^rt=/) {split($i,a,"="); if(a[2]>1) print a[2], $0}}' \
  /var/log/nginx/access.log | sort -rn | head
```

**پیدا کردن ۵۰۲ها:**

```bash
sudo grep ' 502 ' /var/log/nginx/access.log | tail -20
sudo jq -r 'select(.status==502) | "\(.ts) \(.uri) up=\(.upstream) us=\(.ustatus)"' \
  /var/log/nginx/access.json.log | tail -20
```

**`request_id` برای ردیابی:**

```nginx
map $http_x_request_id $req_id {
    default $request_id;
    ~.+     $http_x_request_id;
}

log_format with_rid '... rid=$req_id ...';
add_header X-Request-ID $req_id always;
```

```bash
curl -sSI http://localhost:8080/ | grep -i x-request-id
# X-Request-ID: 6a1f...

sudo grep '6a1f' /var/log/nginx/access.json.log
```

**چرخش دستی:**

```bash
sudo mv /var/log/nginx/access.log /var/log/nginx/access.log.1
sudo kill -USR1 $(cat /var/run/nginx.pid)
ls -la /var/log/nginx/
```

**لاگ اشکال‌زدایی TLS:**

```nginx
log_format ssl '$remote_addr [$time_local] "$request" $status '
               'proto=$server_protocol tls=$ssl_protocol cipher=$ssl_cipher';
access_log /var/log/nginx/ssl.log ssl;
```

### مقایسه و الگوهای مختلف

**متنی در برابر JSON**

| | متنی | JSON |
|---|---|---|
| خوانایی انسان | ✅ | ⚠️ |
| پارس ماشین | ⚠️ | ✅ |
| حجم | ✅ | ⚠️ |
| مناسب | دیباگ سریع | ✅ تولید |

**`buffer` در برابر بدون بافر**

| | `buffer=32k` | بدون |
|---|---|---|
| I/O | ✅ کم | ❌ |
| تأخیر در لاگ | ⚠️ | ✅ |
| از دست رفتن در کرش | ⚠️ | ✅ کمتر |

**`$request_time` در برابر `$upstream_response_time`**

| حالت | تفسیر |
|---|---|
| `rt` بالا، `urt` کم | Nginx/شبکه/کلاینت |
| `rt` بالا، `urt` بالا | اپ کند |
| `rt` کم، `urt` بالا | کش یا `proxy_buffering` |

**دیدگاه «لاگ زیاد خوب است»**: لاگ زیاد، دیسک و نویز. فیلدهای لازم، حجم مدیریت‌شده.

### اشکال شایع

**۱) لاگ بدون `$request_time`**
نمی‌دانی کدام درخواست کند بود.

**۲) `error_log debug` در تولید**
دیسک پر و کندی.

**۳) `access_log` برای استاتیک**
حجم بی‌دلیل. `off`.

**۴) چرخش بدون `USR1`**
Nginx به فایل قدیمی می‌نویسد.

**۵) `log_not_found` روشن**
ربات‌ها لاگ را پر می‌کنند.

**۶) JSON بدون `escape=json`**
JSON نامعتبر با کاراکتر خاص.

**۷) `$upstream_addr` چند مقدار**
پارس ساده شکست می‌خورد. جداکننده را بشناس.

**۸) لاگ بدون `Host`**
چند سایت در یک لاگ، قابل تفکیک نیست. `$host`.

**۹) `buffer` بزرگ و حادثه**
۵ ثانیه آخر لاگ نیست. `flush=1s` در حادثه.

**۱۰) PII در لاگ**
`Authorization`، کوکی. حذف/ماسک.

### نکات مهم (Tips & Tricks)

- **JSON + `jq`** برای تحلیل.
- **`$request_id`** برای ردیابی.
- **`$request_time`** و `$upstream_response_time`** همیشه.
- **`buffer=32k flush=5s`**.
- **`logrotate` + `USR1`**.
- **`access_log off`** برای استاتیک.
- **PII را ماسک کن**.

### خلاصه فصل

- دو لاگ: access و error.
- `$request_time`/`$upstream_response_time` تفکیک گلوگاه.
- `$upstream_addr` چند تلاش را نشان می‌دهد.
- JSON برای تحلیل ماشینی.
- چرخش با `USR1` بدون restart.
- `buffer` سرعت می‌دهد، ولی تأخیر در حادثه.

**سؤال مروری:** اگر `$request_time = 3s` و `$upstream_response_time = 0.05s` باشد، گلوگاه کجاست؟

---

## فصل ۴۷: لاگ در Apache و IIS

### مقدمه

Apache با `LogFormat` و `CustomLog` کار می‌کند؛ IIS با فایل‌های `W3C` در `LogFiles`. هر دو، اطلاعات مشابهی دارند، ولی نام و ساختار متفاوت.

### مفاهیم اساسی

**۱) Apache `LogFormat`**

```apache
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %D" combined_time
CustomLog ${APACHE_LOG_DIR}/access.log combined_time
```

| کد | معنا |
|---|---|
| `%h` | IP (یا نام میزبان) |
| `%l` | identity (معمولاً `-`) |
| `%u` | کاربر |
| `%t` | زمان |
| `%r` | خط درخواست |
| `%>s` | کد وضعیت نهایی |
| `%b` | حجم پاسخ (`-` برای صفر) |
| `%B` | حجم پاسخ (۰ برای صفر) |
| `%D` | زمان (میکروثانیه) |
| `%T` | زمان (ثانیه) |
| `%{X}i` | هدر درخواست |
| `%{X}o` | هدر پاسخ |
| `%{X}e` | متغیر محیطی |
| `%p` | پورت |
| `%v` | نام سرور |

**۲) `%D` در برابر `%T`**

- `%D`: میکروثانیه (دقت بالا)
- `%T`: ثانیه (بدون اعشار در بعضی نسخه‌ها)

برای اندازه‌گیری، `%D` بهتر.

**۳) `%{ms}T` (2.4.18+)**

میلی‌ثانیه با اعشار:

```apache
LogFormat "%h %t \"%r\" %>s %b %{ms}T" perf
```

**۴) لاگ per-vhost**

```apache
<VirtualHost *:443>
    ErrorLog  ${APACHE_LOG_DIR}/lab-error.log
    CustomLog ${APACHE_LOG_DIR}/lab-access.log combined
</VirtualHost>
```

**۵) `LogLevel` per-module**

```apache
LogLevel warn
LogLevel info ssl:warn rewrite:trace3
```

**۶) `ErrorLogFormat`**

```apache
ErrorLogFormat "[%t] [%l] [pid %P] %F: %E: [client %a] %M"
```

**۷) IIS — W3C**

```
#Software: Microsoft Internet Information Services 10.0
#Version: 1.0
#Date: 2026-10-22 00:00:00
#Fields: date time s-ip cs-method cs-uri-stem cs-uri-query s-port cs-username c-ip cs(User-Agent) cs(Referer) sc-status sc-substatus sc-win32-status time-taken
2026-10-22 14:30:01 192.168.56.10 GET / - 443 - 203.0.113.45 Mozilla/5.0 - 200 0 0 123
```

| فیلد | معنا |
|---|---|
| `s-ip` | IP سرور |
| `cs-method` | متد |
| `cs-uri-stem` | مسیر |
| `c-ip` | IP کلاینت |
| `sc-status` | کد |
| `sc-substatus` | زیرکد (مثل `403.14`) |
| `time-taken` | میلی‌ثانیه |

**۸) IIS — محل و تنظیم**

```powershell
Get-WebConfigurationProperty -PSPath "IIS:\Sites\lab" `
  -Filter "system.webServer/httpLogging" -Name "directory"
# C:\inetpub\logs\LogFiles

# فرمت
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.applicationHost/sites/siteDefaults/logFile" `
  -Name "logFormat" -Value "W3C"
```

**۹) IIS — لاگ پیشرفته**

```powershell
# نصب Enhanced Logging
# https://www.iis.net/downloads/microsoft/advanced-logging
# فیلدهای اضافه: time-taken، cs-host، ...
```

**۱۰) چرخش**

- Apache: `logrotate` (Debian از پیش تنظیم)
- IIS: `logFile` با `localTimeRollover` و `truncateSize`، یا اسکریپت

```powershell
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.applicationHost/sites/siteDefaults/logFile" `
  -Name "truncateSize" -Value 10485760
```

### مثال‌های عملی

**Lab Scenario: لاگ کامل Apache**

```apache
# /etc/apache2/conf-available/logging.conf
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %D %{X-Forwarded-For}i" full
LogFormat "%h %t \"%r\" %>s %b %D" perf
LogFormat "{\"ts\":\"%{%Y-%m-%dT%H:%M:%S%z}t\",\"ip\":\"%a\",\"method\":\"%m\",\"uri\":\"%U%q\",\"status\":%>s,\"bytes\":%B,\"us\":%D}" json

CustomLog ${APACHE_LOG_DIR}/access.log full
ErrorLog ${APACHE_LOG_DIR}/error.log
LogLevel warn
```

```bash
sudo a2enconf logging
sudo apachectl configtest && sudo systemctl reload apache2

curl -sS http://localhost/ >/dev/null
sudo tail -1 /var/log/apache2/access.log
```

**پیدا کردن کندترین‌ها (میکروثانیه):**

```bash
sudo awk '{print $NF, $0}' /var/log/apache2/access.log | sort -rn | head -5
# یا با فیلد مشخص (بسته به فرمت)
sudo awk -F'"' '{split($0,a," "); print a[length(a)-2], $0}' /var/log/apache2/access.log | sort -rn | head
```

**لاگ JSON در Apache:**

```apache
LogFormat "{\"ts\":\"%{%Y-%m-%dT%H:%M:%S%z}t\",\"ip\":\"%a\",\"host\":\"%V\",\"method\":\"%m\",\"uri\":\"%U%q\",\"status\":%>s,\"bytes\":%B,\"us\":%D,\"ua\":\"%{User-Agent}i\"}" json
CustomLog ${APACHE_LOG_DIR}/access.json.log json
```

> `%a` (IP) بهتر از `%h` است (بدون reverse DNS، سریع‌تر).

**IIS — خواندن لاگ:**

```powershell
$log = Get-ChildItem C:\inetpub\logs\LogFiles\W3SVC*\*.log | Sort-Object LastWriteTime -Descending | Select-Object -First 1
Get-Content $log.FullName -Tail 20
```

**IIS — پارس و تحلیل:**

```powershell
$files = Get-ChildItem C:\inetpub\logs\LogFiles\W3SVC*\*.log
$lines = $files | ForEach-Object { Get-Content $_.FullName } | Where-Object { $_ -notmatch '^#' }

# ۱۰ درخواست کندتر از ۱ ثانیه
$lines | ForEach-Object {
  $f = $_ -split ' '
  [pscustomobject]@{
    time   = "$($f[0]) $($f[1])"
    method = $f[3]
    uri    = $f[4]
    status = $f[11]
    ms     = [int]$f[14]
  }
} | Where-Object { $_.ms -gt 1000 } | Sort-Object ms -Descending | Select-Object -First 10 |
  Format-Table -AutoSize
```

**IIS — شمارش کدها:**

```powershell
$lines | ForEach-Object { ($_ -split ' ')[11] } | Group-Object | Sort-Object Count -Descending |
  Select-Object Name, Count
```

**IIS — فعال‌سازی فیلد `cs-host`:**

```powershell
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.applicationHost/sites/siteDefaults/logFile" `
  -Name "logTargetW3C" -Value "File"
```

**چرخش IIS:**

```powershell
# روزانه + سقف حجم
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.applicationHost/sites/siteDefaults/logFile" `
  -Name "period" -Value "Daily"
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.applicationHost/sites/siteDefaults/logFile" `
  -Name "truncateSize" -Value 104857600

# پاک کردن قدیمی‌ها
Get-ChildItem C:\inetpub\logs\LogFiles -Recurse -Filter *.log |
  Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays(-30) } |
  Remove-Item -Force
```

### مقایسه و الگوهای مختلف

**Nginx در برابر Apache در برابر IIS**

| | Nginx | Apache | IIS |
|---|---|---|---|
| فرمت | `log_format` | `LogFormat` | W3C |
| زمان | `$request_time` (s) | `%D` (µs) | `time-taken` (ms) |
| upstream | ✅ | ⚠️ (`%{...}e`) | ⚠️ |
| چرخش | `USR1` + logrotate | logrotate | خودکار |
| JSON | ✅ | ✅ | ⚠️ (افزونه) |

**`%h` در برابر `%a`**

| | `%h` | `%a` |
|---|---|---|
| reverse DNS | ✅ (کند) | ❌ |
| سرعت | ❌ | ✅ |
| توصیه | ❌ | ✅ |

**`%D` در برابر `%{ms}T`**

| | `%D` | `%{ms}T` |
|---|---|---|
| واحد | µs | ms |
| دقت | ✅ | ✅ |
| نسخه | همه | 2.4.18+ |

**دیدگاه «لاگ IIS کافی است»**: W3C پایه است، ولی برای upstream و زمان تفکیکی، `Enhanced Logging` یا ARR log لازم است.

### اشکال شایع

**۱) `%h` با reverse DNS کند**
`HostnameLookups On`. `%a` استفاده کن.

**۲) `%b` با `-` برای صفر**
پارس عددی می‌شکند. `%B`.

**۳) `LogLevel debug`**
دیسک پر. `warn`.

**۴) لاگ مشترک چند vhost**
قابل تفکیک نیست. `%v` یا لاگ جدا.

**۵) IIS بدون `time-taken`**
فیلد پیش‌فرض هست، ولی اگر فرمت دستی عوض شود، می‌رود.

**۶) IIS `period` پیش‌فرض روزانه**
حجم زیاد. `truncateSize`.

**۷) چرخش بدون reload**
Apache به فایل قدیمی می‌نویسد. `graceful`.

**۸) `ErrorLog` و `CustomLog` یکی**
قاطی شدن خطا و دسترسی.

**۹) `%{X-Forwarded-For}i` بدون `remoteip`**
IP واقعی نیست.

**۱۰) PII در `%r`**
توکن در query string. ماسک.

### نکات مهم (Tips & Tricks)

- **`%a`** به‌جای `%h`.
- **`%D`** برای دقت.
- **`%v`** برای چند سایت.
- **`LogLevel warn`**.
- **`logrotate`** + `graceful`.
- **IIS `truncateSize`**.
- **JSON** برای تحلیل.

### خلاصه فصل

- Apache با `LogFormat`/`CustomLog`، IIS با W3C.
- `%D` میکروثانیه، `time-taken` میلی‌ثانیه.
- `%a` سریع‌تر از `%h`.
- `%v` برای تفکیک سایت.
- چرخش: `logrotate` (Apache)، خودکار (IIS).
- PII را ماسک کن.

**سؤال مروری:** چرا `%h` در Apache می‌تواند لاگ را کند کند و جایگزینش چیست؟

---

## فصل ۴۸: تحلیل لاگ — `GoAccess`، `awk` و ELK

### مقدمه

لاگ خام، داده است. تحلیل، بینش. سه سطح ابزار: `awk` برای یک سؤال سریع، `GoAccess` برای داشبورد زنده، ELK/Loki برای جستجوی تاریخی.

### مفاهیم اساسی

**۱) `awk` — سؤال سریع**

```bash
# تعداد درخواست به ازای کد وضعیت
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

# ۱۰ IP پرترافیک
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# ۱۰ مسیر پرترافیک
awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# درخواست‌های ۵xx
awk '$9 ~ /^5/ {print}' /var/log/nginx/access.log | tail -20
```

**۲) `GoAccess` — داشبورد**

```bash
sudo apt install -y goaccess

# HTML
sudo goaccess /var/log/nginx/access.log \
  --log-format=COMBINED \
  -o /var/www/report.html --real-time-html

# ترمینال
sudo goaccess /var/log/nginx/access.log --log-format=COMBINED
```

**۳) `GoAccess` با فرمت سفارشی**

```bash
goaccess access.log \
  --log-format='%h %^[%d:%t %^] "%r" %s %b "%R" "%u" %T' \
  --date-format='%d/%b/%Y' \
  --time-format='%H:%M:%S'
```

**۴) `lnav` — لاگ‌خوان ترمینال**

```bash
lnav /var/log/nginx/access.log
```

با `SQL` روی لاگ:

```
:select count(*) from access_log where status >= 500
```

**۵) ELK — Elasticsearch + Logstash + Kibana**

```
Filebeat → Logstash → Elasticsearch → Kibana
```

**۶) Loki + Grafana — سبک‌تر**

```
Promtail → Loki → Grafana
```

**۷) `jq` برای JSON**

```bash
jq -r 'select(.status >= 500) | "\(.ts) \(.uri)"' access.json.log | tail
jq -r '.rt' access.json.log | awk '{s+=$1; n++} END {print "avg:", s/n, "n:", n}'
```

**۸) `sort`/`uniq` برای شمارش**

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -20
```

**۹) تحلیل حمله**

```bash
# تلاش برای مسیرهای حساس
grep -E '/(\.env|\.git|wp-login|admin|phpmyadmin)' access.log | wc -l

# User-Agent مشکوک
awk -F'"' '{print $6}' access.log | sort | uniq -c | sort -rn | head
```

**۱۰) `goaccess` با `--persist`**

```bash
goaccess access.log --persist --db-path /var/lib/goaccess
```

برای نگه‌داشتن تاریخچه بین اجراها.

### مثال‌های عملی

**Lab Scenario: از `awk` تا داشبورد**

```bash
# ۱) خلاصه
echo "=== status"
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

echo "=== top IP"
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

echo "=== top paths"
awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

echo "=== 5xx"
awk '$9 ~ /^5/' /var/log/nginx/access.log | wc -l

echo "=== slowest (rt)"
awk '{for(i=1;i<=NF;i++) if($i ~ /^rt=/){split($i,a,"="); print a[2], $7, $9}}' \
  /var/log/nginx/access.log | sort -rn | head -10
```

**گزارش HTML با GoAccess:**

```bash
sudo mkdir -p /var/www/reports
sudo goaccess /var/log/nginx/access.log \
  --log-format=COMBINED \
  --output=/var/www/reports/index.html \
  --html-prefs='{"theme":"darkGray"}'

# سرو با Nginx
sudo tee /etc/nginx/sites-available/reports >/dev/null <<'EOF'
server {
    listen 8081;
    server_name _;
    root /var/www/reports;
    location / {
        allow 192.168.56.0/24;
        deny all;
        try_files $uri $uri/ =404;
    }
}
EOF
sudo ln -sf /etc/nginx/sites-available/reports /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx

curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8081/
```

**گزارش زنده:**

```bash
sudo goaccess /var/log/nginx/access.log \
  --log-format=COMBINED \
  --real-time-html \
  --output=/var/www/reports/live.html \
  --ws-url=ws://lab.example.com:8081/ws \
  --port=7890 &

# Nginx برای WebSocket
# location /ws { proxy_pass http://127.0.0.1:7890; ... }
```

**تحلیل JSON با `jq`:**

```bash
# میانگین زمان پاسخ
jq -s 'map(.rt) | add / length' /var/log/nginx/access.json.log

# درصد ۵xx
jq -s '[.[] | select(.status >= 500)] | length' /var/log/nginx/access.json.log

# کندترین‌ها
jq -r 'select(.rt > 0.5) | "\(.rt)\t\(.status)\t\(.uri)"' /var/log/nginx/access.json.log |
  sort -rn | head

# به ازای upstream
jq -r '.upstream // "none"' /var/log/nginx/access.json.log | sort | uniq -c | sort -rn
```

**تحلیل حمله:**

```bash
# IPهایی با بیشترین ۴۰۴ (اسکن)
awk '$9 == 404 {print $1}' access.log | sort | uniq -c | sort -rn | head

# مسیرهای حساس
grep -cE '/(\.env|\.git|wp-login\.php|phpmyadmin|admin)' access.log

# User-Agent خالی یا مشکوک
awk -F'"' 'NF>=6 && length($6) < 5 {print $6}' access.log | sort | uniq -c | sort -rn | head
```

**`lnav` با SQL:**

```bash
lnav -n -c ';SELECT count(*) AS n, status FROM access_log GROUP BY status ORDER BY n DESC' \
  /var/log/nginx/access.log
```

**ELK — `filebeat` ساده:**

```yaml
filebeat.inputs:
  - type: filestream
    id: nginx-access
    paths:
      - /var/log/nginx/access.json.log
    parsers:
      - ndjson:
          target: ""
          overwrite_keys: true

output.elasticsearch:
  hosts: ["http://elasticsearch:9200"]
  index: "nginx-%{+yyyy.MM.dd}"
```

**Loki — `promtail`:**

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: nginx
    static_configs:
      - targets: [localhost]
        labels:
          job: nginx
          __path__: /var/log/nginx/*.log
```

### مقایسه و الگوهای مختلف

**سه سطح ابزار**

| ابزار | سرعت | عمق | نصب |
|---|---|---|---|
| `awk`/`grep` | ✅✅ | ⚠️ | ✅ |
| `GoAccess` | ✅ | ✅ | ✅ |
| ELK/Loki | ⚠️ | ✅✅ | ❌ |

**ELK در برابر Loki**

| | ELK | Loki |
|---|---|---|
| منابع | ❌ زیاد | ✅ کم |
| جستجوی متن | ✅✅ | ⚠️ |
| یکپارچگی Grafana | ⚠️ | ✅ |
| مناسب | تحلیل عمیق | ✅ اکثر |

**`GoAccess` در برابر `AWStats`**

| | GoAccess | AWStats |
|---|---|---|
| real-time | ✅ | ❌ |
| نصب | ✅ | ⚠️ |
| فرمت | ✅ چند | ⚠️ |

**دیدگاه «ELK برای همه»**: ELK منابع زیادی می‌خواهد. برای اکثر پروژه‌ها، `GoAccess` + Loki کافی است.

### اشکال شایع

**۱) `awk` با فرمت اشتباه**
فیلدها جابه‌جا. `awk '{print NF}'` اول.

**۲) `GoAccess` با `log-format` اشتباه**
`0 requests`. فرمت را دقیق بده.

**۳) لاگ بدون `Host`**
چند سایت قاطی. `%v`/`$host`.

**۴) ELK بدون index template**
فیلدها `keyword` نمی‌شوند. template.

**۵) Loki بدون label**
جستجو کند. `job`/`host`.

**۶) `jq` روی فایل بزرگ**
حافظه. `jq -c` یا `stream`.

**۷) تحلیل بدون بازه‌ی زمانی**
داده‌ی قدیمی. `--since`.

**۸) نادیده گرفتن `time-taken`**
کندی پنهان.

**۹) PII در گزارش**
ماسک.

**۱۰) لاگ بدون چرخش**
دیسک پر، تحلیل کند.

### نکات مهم (Tips & Tricks)

- **`awk`** برای سؤال سریع.
- **`GoAccess`** برای داشبورد.
- **`jq`** برای JSON.
- **`lnav`** برای SQL روی لاگ.
- **Loki** سبک‌تر از ELK.
- **بازه‌ی زمانی** در تحلیل.
- **ماسک PII**.

### خلاصه فصل

- `awk`/`grep` برای سؤال فوری.
- `GoAccess` داشبورد HTML/ترمینال.
- `jq` برای لاگ JSON.
- ELK برای جستجوی عمیق، Loki سبک‌تر.
- تحلیل حمله با ۴۰۴ و UA.
- بازه‌ی زمانی و فرمت درست.

**سؤال مروری:** چرا `GoAccess` ممکن است `0 requests` نشان دهد و چطور رفعش می‌کنی؟

---

## فصل ۴۹: مانیتورینگ — `stub_status` و متریک

### مقدمه

لاگ، گذشته را می‌گوید. مانیتورینگ، حال را. چهار متریک پایه: اتصال فعال، نرخ درخواست، نرخ خطا، زمان پاسخ. این فصل، از `stub_status` تا Prometheus.

### مفاهیم اساسی

**۱) Nginx `stub_status`**

```nginx
server {
    listen 127.0.0.1:8081;
    location /status {
        stub_status;
        allow 127.0.0.1;
        deny all;
    }
}
```

```
Active connections: 291
server accepts handled requests
 16630948 16630948 31070465
Reading: 6 Writing: 179 Waiting: 106
```

| فیلد | معنا |
|---|---|
| `Active` | اتصال فعال |
| `accepts` | کل پذیرفته |
| `handled` | موفق |
| `requests` | کل درخواست |
| `Reading` | خواندن هدر |
| `Writing` | نوشتن پاسخ |
| `Waiting` | idle (keep-alive) |

`accepts - handled > 0` = محدودیت منابع (fd/backlog).

**۲) Apache `mod_status`**

```apache
<Location /server-status>
    SetHandler server-status
    Require ip 127.0.0.1
</Location>
ExtendedStatus On
```

```bash
curl -s http://localhost/server-status?auto
# Total Accesses: 1234
# Total kBytes: 5678
# BusyWorkers: 3
# IdleWorkers: 47
# Scoreboard: ...
```

**۳) IIS — Performance Counters**

```powershell
Get-Counter '\Web Service(_Total)\Current Connections', `
            '\Web Service(_Total)\Total Method Requests/sec', `
            '\Web Service(_Total)\Total Bytes Received/sec'
```

**۴) چهار متریک طلایی**

| متریک | منبع |
|---|---|
| نرخ درخواست | `requests` / زمان |
| نرخ خطا | `5xx` / کل |
| زمان پاسخ | `$request_time` p95/p99 |
| اشباع | اتصال فعال / سقف |

**۵) Prometheus — `nginx-prometheus-exporter`**

```bash
docker run -d --name nginx-exporter -p 9113:9113 \
  nginx/nginx-prometheus-exporter:1.1 \
  --nginx.scrape-uri=http://127.0.0.1:8081/status
```

```yaml
scrape_configs:
  - job_name: nginx
    static_configs:
      - targets: ['nginx-exporter:9113']
```

**۶) `nginx-vts` / `nginx-module-vts`**

برای متریک per-upstream، per-server. یا Nginx Plus API.

**۷) `apache_exporter`**

```bash
docker run -d -p 9117:9117 \
  -e APACHE_STATUS_URL=http://127.0.0.1/server-status?auto \
  lusotycoon/apache-exporter
```

**۸) `blackbox_exporter`**

```yaml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      valid_http_versions: ["HTTP/1.1","HTTP/2"]
      valid_status_codes: [200]
      method: GET
```

**۹) هشدارهای پیشنهادی**

| هشدار | شرط |
|---|---|
| نرخ ۵xx | `> 5%` برای ۵ دقیقه |
| اتصال | `> 80%` سقف |
| p99 | `> 2s` |
| گواهی | `< 14 روز` |
| upstream down | `nginx_upstream_server_up == 0` |
| دیسک | `< 15%` |

**۱۰) `access log` به متریک**

با `nginx-prometheus-exporter` فقط `stub_status` است. برای نرخ خطا بر اساس لاگ:

```
mtail / vector / fluent-bit → Prometheus
```

یا از `log_format` + `mtail`:

```
counter http_requests_total by status, method
/^(\S+) .* "(\S+) (\S+) [^"]*" (\d{3}) / {
    http_requests_total[$4][$2]++
}
```

### مثال‌های عملی

**Lab Scenario: از `stub_status` تا Grafana**

```nginx
server {
    listen 127.0.0.1:8081;
    server_name _;
    location /status {
        stub_status;
        allow 127.0.0.1;
        deny all;
        access_log off;
    }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx
curl -s http://127.0.0.1:8081/status
```

**تولید بار و مشاهده:**

```bash
hey -z 30s -c 50 http://localhost:8080/ &
for i in $(seq 1 10); do
  curl -s http://127.0.0.1:8081/status | head -1
  sleep 2
done
```

**exporter + Prometheus:**

```yaml
# docker-compose.yml
services:
  nginx-exporter:
    image: nginx/nginx-prometheus-exporter:1.1
    command:
      - --nginx.scrape-uri=http://host.docker.internal:8081/status
    ports: ["9113:9113"]

  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
    ports: ["9090:9090"]

  grafana:
    image: grafana/grafana
    ports: ["3000:3000"]
```

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: nginx
    static_configs:
      - targets: ['nginx-exporter:9113']

  - job_name: blackbox
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - http://host.docker.internal:8080/
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - target_label: __address__
        replacement: blackbox:9115
```

**کوئری‌های Prometheus:**

```promql
# نرخ درخواست
rate(nginx_http_requests_total[5m])

# اتصال فعال
nginx_connections_active

# نسبت ۵xx (با mtail/vector)
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))

# p99 زمان پاسخ
histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))

# گواهی
probe_ssl_earliest_cert_expiry - time() < 86400 * 14
```

**هشدار:**

```yaml
groups:
  - name: web
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m]))
          /
          sum(rate(http_requests_total[5m])) > 0.05
        for: 5m
        labels: { severity: critical }
        annotations:
          summary: "نرخ خطای ۵xx بالای ۵٪"

      - alert: NginxUpstreamDown
        expr: nginx_upstream_server_up == 0
        for: 2m
        labels: { severity: warning }

      - alert: CertExpiringSoon
        expr: probe_ssl_earliest_cert_expiry - time() < 86400 * 14
        for: 1h
        labels: { severity: warning }
```

**Apache `mod_status`:**

```apache
ExtendedStatus On
<Location /server-status>
    SetHandler server-status
    Require ip 127.0.0.1
</Location>
```

```bash
curl -s 'http://localhost/server-status?auto' | head -10
```

**IIS Performance Counter به Prometheus:**

```powershell
# windows_exporter با collector
# --collectors.enabled "cpu,cs,logical_disk,net,os,service,textfile,process,iis"
```

```yaml
- job_name: windows
  static_configs:
    - targets: ['win-server:9182']
```

**داشبورد Grafana — پنل‌های پیشنهادی:**

| پنل | کوئری |
|---|---|
| RPS | `rate(nginx_http_requests_total[1m])` |
| Active conn | `nginx_connections_active` |
| 5xx rate | `sum(rate(http_requests_total{status=~"5.."}[5m]))` |
| p95 latency | `histogram_quantile(0.95, ...)` |
| upstream up | `nginx_upstream_server_up` |
| cert days | `(probe_ssl_earliest_cert_expiry - time())/86400` |

### مقایسه و الگوهای مختلف

**`stub_status` در برابر `vts` در برابر Plus API**

| | `stub_status` | `vts` | Plus |
|---|---|---|---|
| per-upstream | ❌ | ✅ | ✅ |
| per-server | ❌ | ✅ | ✅ |
| نصب | ✅ | ⚠️ (ماژول) | 💰 |
| مناسب | پایه | ✅ OSS | سازمانی |

**Prometheus در برابر Zabbix در برابر Datadog**

| | Prometheus | Zabbix | Datadog |
|---|---|---|---|
| مدل | pull | push/pull | SaaS |
| هزینه | ✅ | ✅ | 💰 |
| اکوسیستم وب | ✅ | ⚠️ | ✅ |
| مناسب | K8s/ابری | سازمانی | تیم کوچک |

**چهار متریک طلایی**

| متریک | آستانه‌ی هشدار |
|---|---|
| نرخ درخواست | افت ناگهانی |
| نرخ خطا | >۵٪ |
| زمان پاسخ | p99 >۲s |
| اشباع | >۸۰٪ |

**دیدگاه «لاگ کافی است»**: لاگ برای دیباگ گذشته، متریک برای هشدار حال. هر دو لازم.

### اشکال شایع

**۱) `stub_status` باز**
اطلاعات لو می‌رود. `allow`/`deny`.

**۲) `accepts` > `handled`**
fd/backlog کم. `worker_rlimit_nofile`.

**۳) `Waiting` بالا**
keep-alive زیاد. طبیعی، ولی `worker_connections` را مصرف می‌کند.

**۴) متریک بدون label**
نمی‌دانی کدام سرور. `instance`/`job`.

**۵) هشدار روی میانگین**
p99 را پنهان می‌کند. از `histogram_quantile`.

**۶) `ExtendedStatus On` نبودن در Apache**
`server-status` کامل نیست.

**۷) exporter بدون دسترسی**
`403`. `allow`.

**۸) هشدار زیاد**
`alert fatigue`. ۵–۱۰ حیاتی.

**۹) گواهی بدون هشدار**
انقضا. `blackbox`.

**۱۰) داشبورد بدون بازه**
نمی‌دانی «الان» چطور است.

### نکات مهم (Tips & Tricks)

- **`stub_status`** محدود به localhost.
- **`nginx-prometheus-exporter`**.
- **`blackbox_exporter`** برای TLS و availability.
- **`histogram_quantile`** برای p95/p99.
- **۵–۱۰ هشدار** کافی.
- **`accepts`/`handled`** را چک کن.
- **IIS: `windows_exporter`**.

### خلاصه فصل

- `stub_status` پایه‌ی متریک Nginx است.
- چهار متریک طلایی: نرخ، خطا، زمان، اشباع.
- Prometheus + exporter + Grafana.
- `histogram_quantile` برای صدک.
- هشدارهای کم و دقیق.
- لاگ و متریک مکمل‌اند.

**سؤال مروری:** اگر `accepts` از `handled` بیشتر باشد، چه مشکلی داری و چطور حل می‌شود؟

---

## منابع و مراجع

**مستندات رسمی**
- [Nginx — ngx_http_log_module](https://nginx.org/en/docs/http/ngx_http_log_module.html)
- [Nginx — stub_status](https://nginx.org/en/docs/http/ngx_http_stub_status_module.html)
- [Apache — Log Files](https://httpd.apache.org/docs/2.4/logs.html)
- [Apache — mod_log_config](https://httpd.apache.org/docs/2.4/mod/mod_log_config.html)
- [IIS — Logging](https://learn.microsoft.com/en-us/iis/configuration/system.webserver/httplogging/)
- [Prometheus — best practices](https://prometheus.io/docs/practices/naming/)

**ابزار**
- [GoAccess](https://goaccess.io/)
- [lnav](https://lnav.org/)
- [nginx-prometheus-exporter](https://github.com/nginx/nginx-prometheus-exporter)
- [apache_exporter](https://github.com/Lusitaniae/apache_exporter)
- [blackbox_exporter](https://github.com/prometheus/blackbox_exporter)
- [Grafana](https://grafana.com/)

**آموزش**
- [Four Golden Signals — Google SRE](https://sre.google/sre-book/monitoring-distributed-systems/)
- [Logging best practices](https://www.nginx.com/blog/nginx-logging-best-practices/)

---

**بعدی:** [`12-Performance-and-Caching.md`](12-Performance-and-Caching.md) — کش، فشرده‌سازی، HTTP/2 و ۳، tuning
