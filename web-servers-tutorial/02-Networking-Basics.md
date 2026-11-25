# بخش ۰۲: شبکه برای وب‌سرور

**مدت یادگیری**: ۴ تا ۵ ساعت
**سطح سختی**: مبتدی
**پیش‌نیاز**: بخش ۰۱؛ آشنایی مقدماتی با IP

---

## فهرست محتوا

- [فصل ۵: TCP، سوکت، پورت و صف اتصال](#فصل-۵)
- [فصل ۶: DNS — از دامنه تا IP](#فصل-۶)
- [فصل ۷: پروکسی، NAT، فایروال و `X-Forwarded-*`](#فصل-۷)
- [فصل ۸: WebSocket، SSE و اتصال‌های طولانی](#فصل-۸)

---

## فصل ۵: TCP، سوکت، پورت و صف اتصال

### مقدمه

وب‌سرور روی یک **سوکت** گوش می‌دهد: ترکیب `IP:Port`. قبل از هر بایت HTTP، یک اتصال TCP برقرار می‌شود. اگر این لایه را نفهمی، خطاهای `connection refused`، `timeout`، `too many open files` و `backlog overflow` برایت جادو می‌شوند.

### مفاهیم اساسی

**۱) سوکت چیست**

یک سوکت، شناسه‌ای برای «یک سرِ ارتباط» است:

```
سرور:  192.168.56.10:443   (listen)
کلاینت: 203.0.113.45:51234 (ephemeral)
اتصال یکتا: (src_ip, src_port, dst_ip, dst_port)
```

**۲) سه‌مرحله‌ی برقراری (three-way handshake)**

```
Client                          Server
  │──── SYN (seq=x) ────────────▶│
  │◀─── SYN-ACK (seq=y, ack=x+1)─│
  │──── ACK (ack=y+1) ──────────▶│
  │                              │
  │◀═══════ data ═══════════════▶│
```

هزینه: یک RTT قبل از اولین بایت داده. به همین دلیل `keep-alive` مهم است.

**۳) پورت‌ها**

| پورت | سرویس |
|---|---|
| ۲۲ | SSH |
| ۲۵ | SMTP |
| ۵۳ | DNS |
| ۸۰ | HTTP |
| ۴۴۳ | HTTPS |
| ۳۳۰۶ | MySQL |
| ۵۴۳۲ | PostgreSQL |
| ۶۳۷۹ | Redis |
| ۸۰۸۰ / ۸۴۴۳ | HTTP/HTTPS جایگزین |
| ۳۰۰۰ / ۵۰۰۰ / ۸۰۰۰ | اپلیکیشن‌های توسعه |

**پورت‌های زیر ۱۰۲۴** نیاز به دسترسی `root`/`CAP_NET_BIND_SERVICE` دارند.

**۴) `listen` و `backlog`**

وقتی سرور `listen(port, backlog)` می‌زند، کرنل یک صف برای اتصال‌های نیمه‌باز و کامل نگه می‌دارد:

```
┌──────────────────────────────────────┐
│  SYN queue (نیمه‌باز)                 │
│  accept queue (کامل، منتظر accept)   │
└──────────────────────────────────────┘
         │
         ▼
   وب‌سرور accept() می‌کند
```

اگر صف پر شود، اتصال‌های جدید **drop** می‌شوند (کلاینت `timeout` می‌بیند، نه خطای واضح).

| پارامتر | معنا | پیش‌فرض |
|---|---|---|
| `net.core.somaxconn` | حداکثر backlog | ۴۰۹۶ (جدید) |
| `net.ipv4.tcp_max_syn_backlog` | صف SYN | ۱۰۲۴ |
| `listen ... backlog=511` (Nginx) | backlog برنامه | ۵۱۱ |

**۵) `TIME_WAIT` — چرا زیاد می‌شود**

بعد از بستن اتصال، سمت فعال‌کننده‌ی بستن، وارد `TIME_WAIT` می‌شود (۲×MSL، معمولاً ۶۰ ثانیه). در سرور پرترافیک، هزاران سوکت در این حالت دیده می‌شود. طبیعی است، ولی اگر پورت‌های ephemeral تمام شوند، مشکل می‌شود.

```bash
ss -tan state time-wait | wc -l
cat /proc/sys/net/ipv4/ip_local_port_range
```

**۶) `SO_REUSEPORT`**

چند پروسه می‌توانند روی یک پورت گوش دهند و کرنل بار را پخش می‌کند. Nginx از ۱.۹.۱ پشتیبانی می‌کند:

```nginx
listen 80 reuseport;
```

مزیت: پخش بار در لایه‌ی کرنل، بدون قفل مشترک. عیب: در reload، سوکت‌های قدیمی و جدید همزمان هستند (معمولاً مشکلی نیست).

**۷) `keepalive` در سه لایه**

| لایه | پارامتر | اثر |
|---|---|---|
| TCP (کلاینت↔سرور) | `keepalive_timeout` (Nginx) | اتصال بی‌استفاده چقدر باز بماند |
| TCP (سرور↔upstream) | `keepalive` در `upstream` | استفاده‌ی مجدد از اتصال به اپ |
| HTTP | `Connection: keep-alive` | چند درخواست روی یک اتصال |

**۸) `tcp_nodelay` و `tcp_nopush`**

- `tcp_nodelay on`: بسته‌های کوچک را فوراً بفرست (کم‌کردن تأخیر). برای API و WebSocket.
- `tcp_nopush on`: بسته‌ها را جمع کن و با هم بفرست (کارایی برای فایل بزرگ). با `sendfile` کار می‌کند.

این دو با هم متناقض به نظر می‌رسند، ولی Nginx هر دو را در شرایط درست استفاده می‌کند.

**۹) IPv4 و IPv6**

```nginx
listen 80;          # IPv4 (و IPv6 اگر ipv6only=off)
listen [::]:80;     # IPv6 صریح
listen [::]:80 ipv6only=on;
```

اگر فقط `listen 80` بنویسی و سیستم `net.ipv6.bindv6only=0` داشته باشد، ممکن است هر دو را بگیرد. صریح بنویس تا غافلگیر نشوی.

### مثال‌های عملی

**Lab Scenario: بررسی سوکت‌ها و صف اتصال**

```bash
# ۱) چه چیزی روی کدام پورت گوش می‌دهد؟
sudo ss -ltnp
# State  Recv-Q Send-Q Local Address:Port  Process
# LISTEN 0      511    0.0.0.0:80            nginx
# LISTEN 0      511    0.0.0.0:443           nginx

# ۲) اتصال‌های فعال
ss -tan | awk 'NR>1 {print $1}' | sort | uniq -c | sort -rn
#   120 ESTAB
#    45 TIME-WAIT
#     3 LISTEN

# ۳) صف accept (Recv-Q در LISTEN = اتصال‌های منتظر)
ss -ltn 'sport = :80'
# اگر Recv-Q بزرگ باشد → وب‌سرور کند accept می‌کند
```

**تنظیم backlog و محدودیت فایل:**

```bash
# /etc/sysctl.d/99-web.conf
sudo tee /etc/sysctl.d/99-web.conf >/dev/null <<'EOF'
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 10240 65000
net.ipv4.tcp_fin_timeout = 15
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
EOF
sudo sysctl --system
```

```bash
# محدودیت فایل باز
sudo tee /etc/security/limits.d/99-web.conf >/dev/null <<'EOF'
www-data soft nofile 65535
www-data hard nofile 65535
EOF
```

```nginx
# Nginx
worker_rlimit_nofile 65535;
events {
    worker_connections 16384;
    multi_accept on;
    use epoll;
}
```

```apache
# Apache (mpm_event)
<IfModule mpm_event_module>
    StartServers             4
    MinSpareThreads          75
    MaxSpareThreads          250
    ThreadsPerChild          25
    MaxRequestWorkers        400
    MaxConnectionsPerChild   10000
</IfModule>
```

**تست باز کردن اتصال زیاد:**

```bash
# ۱۰۰۰ اتصال همزمان با hey (یا ab)
hey -n 20000 -c 1000 -disable-keepalive http://lab.example.com/

# بررسی شمارش سوکت در حین تست
watch -n1 "ss -s"
```

**تست `SYN flood` ساده (فقط در محیط خودت):**

```bash
sudo hping3 -S -p 80 --flood --rand-source 192.168.56.10
# سپس در سرور: ss -s، dmesg | tail
```

### مقایسه و الگوهای مختلف

**`select` / `poll` / `epoll` / `kqueue`**

| مکانیزم | پیچیدگی | مقیاس | سیستم‌عامل |
|---|---|---|---|
| `select` | O(n) | ~۱۰۲۴ | همه |
| `poll` | O(n) | بهتر | همه |
| `epoll` | O(1) | میلیون‌ها | لینوکس |
| `kqueue` | O(1) | بالا | BSD/macOS |
| `IOCP` | O(1) | بالا | ویندوز |

Nginx و IIS از مکانیزم O(1) استفاده می‌کنند؛ Apache prefork از مدل پروسه‌ای.

**`listen backlog` کم در برابر زیاد**

| | کم (۱۲۸) | زیاد (۶۵۵۳۵) |
|---|---|---|
| حافظه | ✅ | ⚠️ |
| تحمل burst | ❌ | ✅ |
| توصیه | ❌ | ✅ |

**سه حالت `keepalive_timeout`**

| مقدار | اثر |
|---|---|
| `0` | هر درخواست، اتصال جدید (کند) |
| `65s` (پیش‌فرض) | تعادل خوب |
| `300s` | اتصال بیشتر، مصرف fd بیشتر |

**دیدگاه «`TIME_WAIT` را باید حذف کرد»**: نه. `TIME_WAIT` بخشی از پروتکل است و از اختلال بسته‌های قدیمی جلوگیری می‌کند. اگر پورت تمام شد، `ip_local_port_range` را بزرگ کن، نه `tcp_tw_reuse` را روشن (که در بعضی سناریوها مشکل‌ساز است).

### اشکال شایع

**۱) `too many open files`**
هر اتصال یک fd است. `worker_rlimit_nofile` و `limits.conf`.

**۲) `Connection refused`**
سرور روی آن پورت گوش نمی‌دهد، یا فایروال `RST` می‌فرستد. `ss -ltnp`.

**۳) `Connection timed out`**
بسته drop می‌شود (فایروال، مسیر، backlog پر). `tcpdump`.

**۴) backlog کم و drop بی‌صدا**
کلاینت `timeout` می‌بیند، لاگ سرور چیزی ندارد. `somaxconn`.

**۵) `TIME_WAIT` زیاد و اتمام پورت**
`ip_local_port_range` بزرگ‌تر، `keepalive` بهتر.

**۶) `tcp_nodelay off` برای API**
تأخیر اضافه (Nagle). `tcp_nodelay on`.

**۷) `sendfile off` برای فایل استاتیک**
کپی اضافه در userspace. `sendfile on`.

**۸) `ipv6only` و دسترسی نداشتن**
`listen [::]:80` بدون `ipv6only=off` فقط IPv6. تست با `curl -6`.

**۹) `SO_REUSEPORT` با TLS و session**
در بعضی نسخه‌ها، session cache بین workerها پخش نمی‌شود. `ssl_session_cache shared:`.

**۱۰) فراموش کردن `multi_accept`**
هر بار یک اتصال از صف. `multi_accept on`.

### نکات مهم (Tips & Tricks)

- **`ss -s`** خلاصه‌ی وضعیت سوکت.
- **`ss -ltnp`** پورت‌های listen با پروسه.
- **`ss -tan state established '( dport = :443 )'`** اتصال‌های HTTPS.
- **`sysctl net.core.somaxconn`** را چک کن.
- **`worker_connections` × `worker_processes`** = سقف اتصال تقریبی.
- **`tcpdump -i any port 80 -c 20 -nn`** برای دیدن بسته‌ها.
- **`nstat -az`** آمار TCP (retransmit، drop).

### خلاصه فصل

- سوکت = `IP:Port`؛ اتصال یکتا با ۴تایی تعریف می‌شود.
- handshake یک RTT هزینه دارد؛ `keep-alive` آن را کم می‌کند.
- `backlog` و `somaxconn` از drop بی‌صدا جلوگیری می‌کنند.
- `TIME_WAIT` طبیعی است؛ پورت‌ها را زیاد کن، نه اینکه پروتکل را دور بزن.
- `sendfile`، `tcp_nodelay`، `tcp_nopush` برای کارایی.
- `epoll`/`IOCP` مقیاس بالا را ممکن می‌کند.

**سؤال مروری:** چرا وقتی backlog پر می‌شود، کلاینت `timeout` می‌بیند و سرور هیچ خطایی در لاگ ندارد؟

---

## فصل ۶: DNS — از دامنه تا IP

### مقدمه

قبل از هر اتصال TCP، باید `example.com` به IP تبدیل شود. DNS یک سیستم توزیع‌شده و **کش‌شده** است. بیشتر «سایت باز نمی‌شود»ها و «گواهی برای دامنه‌ی دیگر است»ها، ریشه در DNS دارند.

### مفاهیم اساسی

**۱) سلسله‌مراتب**

```
.  (root)
├── com.
│   └── example.com.
│       ├── www.example.com.
│       └── api.example.com.
└── ir.
    └── example.ir.
```

**۲) انواع رکورد**

| نوع | کار | مثال |
|---|---|---|
| `A` | نام → IPv4 | `example.com. 300 IN A 203.0.113.10` |
| `AAAA` | نام → IPv6 | `example.com. 300 IN AAAA 2001:db8::10` |
| `CNAME` | نام → نام دیگر | `www → example.com.` |
| `MX` | سرور ایمیل | `10 mail.example.com.` |
| `TXT` | متن (SPF، DKIM، تأیید دامنه) | `"v=spf1 -all"` |
| `NS` | سرورهای نام | `ns1.example.com.` |
| `SOA` | اطلاعات ناحیه | — |
| `SRV` | سرویس + پورت | — |
| `CAA` | کدام CA مجاز است | `0 issue "letsencrypt.org"` |

**۳) `TTL` و کش**

`TTL=300` یعنی «۳۰۰ ثانیه کش کن». برای مهاجرت، **قبل از** تغییر، TTL را کم کن (مثلاً ۶۰) تا تغییر سریع منتشر شود.

**۴) انواع سرور DNS**

| نوع | نقش |
|---|---|
| **Recursive resolver** | از طرف کلاینت می‌پرسد (ISP، 1.1.1.1، 8.8.8.8) |
| **Authoritative** | پاسخ نهایی برای یک ناحیه |
| **Forwarder** | رزولور محلی که به بالادست می‌فرستد |

**۵) مسیر حل نام**

```
مرورگر → کش مرورگر → کش OS → resolver → root → .com → authoritative
                                                    │
                                        پاسخ: A 203.0.113.10
```

**۶) `CNAME` و محدودیت‌ها**

- `CNAME` در ریشه‌ی دامنه (`example.com`) مجاز نیست (با `SOA`/`NS` تعارض دارد) — مگر با `ALIAS`/`ANAME` در بعضی سرویس‌ها.
- `CNAME` به `CNAME` مجاز است، ولی زنجیره‌ی طولانی، تأخیر می‌آورد.

**۷) `SNI` و DNS — چرا گواهی اشتباه می‌گیری**

اگر `api.example.com` به IP اشتباه اشاره کند، سرور گواهی `example.com` را می‌دهد (چون `SNI` را نمی‌شناسد یا default server است) → خطای نام.

**۸) `DNS` در محیط داخلی**

- `/etc/hosts` قبل از DNS خوانده می‌شود.
- `resolv.conf` سرورهای DNS را می‌دهد.
- `systemd-resolved` ممکن است کش محلی داشته باشد: `resolvectl flush-caches`.

**۹) `DNSSEC` و `CAA`**

- `DNSSEC`: امضای رکوردها، جلوگیری از جعل.
- `CAA`: فقط CAهای مجاز می‌توانند گواهی صادر کنند (بهبود امنیت Let's Encrypt).

### مثال‌های عملی

**Lab Scenario: دیباگ DNS برای یک دامنه**

```bash
# ۱) رکوردهای اصلی
dig +short A    lab.example.com
dig +short AAAA lab.example.com
dig +short CNAME www.lab.example.com

# ۲) پاسخ کامل با TTL
dig lab.example.com A

# ۳) از سرور نام معتبر
dig @ns1.example.com lab.example.com A +norecurse

# ۴) مسیر کامل
dig +trace lab.example.com

# ۵) بررسی CAA (مهم برای Let's Encrypt)
dig CAA lab.example.com +short

# ۶) بررسی TXT
dig TXT lab.example.com +short
```

**تست بدون تغییر DNS (برای مهاجرت):**

```bash
# curl با resolve
curl -sSI --resolve lab.example.com:443:192.168.56.20 https://lab.example.com/

# یا با هدر Host روی HTTP
curl -sSI -H 'Host: lab.example.com' http://192.168.56.20/
```

**`/etc/hosts` برای آزمایشگاه:**

```bash
echo "192.168.56.10  lab.example.com www.lab.example.com" | sudo tee -a /etc/hosts
```

**تنظیم `CAA` (مثال در ناحیه):**

```
lab.example.com.  IN  CAA  0 issue "letsencrypt.org"
lab.example.com.  IN  CAA  0 iodef "mailto:security@example.com"
```

**پاک کردن کش:**

```bash
# systemd-resolved
sudo resolvectl flush-caches

# nscd
sudo systemctl restart nscd

# ویندوز
ipconfig /flushdns
```

**بررسی DNS از دید سرور (چه چیزی می‌بیند):**

```bash
# از داخل کانتینر (ممکن است DNS متفاوت باشد)
docker run --rm alpine nslookup lab.example.com

# از سرور
getent hosts lab.example.com
```

### مقایسه و الگوهای مختلف

**`A` در برابر `CNAME` در برابر `ALIAS`**

| | `A` | `CNAME` | `ALIAS`/`ANAME` |
|---|---|---|---|
| ریشه‌ی دامنه | ✅ | ❌ | ✅ |
| تغییر IP مقصد | ⚠️ دستی | ✅ خودکار | ✅ |
| استاندارد | ✅ | ✅ | ❌ (اختصاصی) |
| مناسب | سرور ثابت | زیردامنه به سرویس | ریشه به CDN |

**سه مدل توزیع بار DNS**

| مدل | مزیت | عیب |
|---|---|---|
| چند `A` | ساده، round-robin | بدون health check |
| `GeoDNS` | نزدیک‌ترین | پیچیدگی |
| `Anycast` | تأخیر کم، DDoS-resistant | نیاز به BGP |

**دیدگاه «DNS فوری منتشر می‌شود»**: نه. `TTL` حاکم است. برای مهاجرت، TTL را ۲۴ ساعت قبل کم کن.

### اشکال شایع

**۱) `NXDOMAIN` برای دامنه‌ای که وجود دارد**
`dig @8.8.8.8` بزن. اگر جواب داد، resolver محلی مشکل دارد.

**۲) `SERVFAIL`**
اغلب `DNSSEC` شکسته یا سرور نام معتبر پاسخ‌گو نیست.

**۳) گواهی برای دامنه‌ی اشتباه**
`SNI` به سروری می‌رسد که default server است. `server_name`/`binding` را چک کن.

**۴) `CNAME` در ریشه**
بعضی ارائه‌دهنده‌ها اجازه می‌دهند ولی رفتار غیرقابل پیش‌بینی است. `ALIAS`.

**۵) `TTL` بلند و مهاجرت دردناک**
قبل از تغییر، TTL را کم کن.

**۶) `/etc/hosts` فراموش‌شده**
تست محلی جواب می‌دهد، تولید نه. `getent hosts`.

**۷) `CAA` که Let's Encrypt را بلاک می‌کند**
`CAA` فقط برای CA دیگری تنظیم شده. `dig CAA`.

**۸) `AAAA` اشتباه**
مرورگر IPv6 را ترجیح می‌دهد و به سرور اشتباه می‌رود. `curl -4` برای تست.

**۹) کش مرورگر**
`chrome://net-internals/#dns` → Clear host cache.

**۱۰) split-horizon DNS**
داخل شبکه یک IP، بیرون IP دیگر. تست از هر دو سمت.

### نکات مهم (Tips & Tricks)

- **`dig +short`** خلاصه.
- **`dig +trace`** مسیر کامل.
- **`dig @server`** پرسش از سرور مشخص.
- **`curl --resolve`** برای تست بدون DNS.
- **`getent hosts`** همان چیزی که اپ می‌بیند.
- **`dig CAA`** قبل از درخواست گواهی.
- **`resolvectl flush-caches`** برای پاک کردن کش.

### خلاصه فصل

- DNS سلسله‌مراتبی و کش‌شده است؛ `TTL` حاکم است.
- `A`/`AAAA` برای IP، `CNAME` برای نام دیگر، `CAA` برای محدود کردن CA.
- `CNAME` در ریشه مجاز نیست.
- گواهی اشتباه معمولاً از `SNI` + default server می‌آید.
- `dig +trace` و `curl --resolve` ابزارهای اصلی.

**سؤال مروری:** چرا قبل از مهاجرت سرور باید `TTL` را کم کرد و اگر نکنی چه اتفاقی می‌افتد؟

---

## فصل ۷: پروکسی، NAT، فایروال و `X-Forwarded-*`

### مقدمه

در معماری واقعی، درخواست مستقیم از کلاینت به وب‌سرور نمی‌رسد: از CDN، LB، فایروال و NAT عبور می‌کند. نتیجه: **IP واقعی کلاینت گم می‌شود** و **پروتکل واقعی (http/https) گم می‌شود**. این فصل، راه حل استاندارد است.

### مفاهیم اساسی

**۱) Forward Proxy در برابر Reverse Proxy**

```
Forward Proxy:  کلاینت → پروکسی → اینترنت     (به نمایندگی کلاینت)
Reverse Proxy:  اینترنت → پروکسی → سرورها     (به نمایندگی سرور)
```

**۲) NAT — چرا IP عوض می‌شود**

```
کلاینت 10.0.0.5:51234  →  [NAT]  →  203.0.113.7:40001  →  سرور
```

سرور فقط `203.0.113.7` را می‌بیند. برای لاگ و rate limit، باید IP واقعی را از هدر بگیرد.

**۳) هدرهای استاندارد**

| هدر | چه می‌گوید |
|---|---|
| `X-Forwarded-For` | زنجیره‌ی IPها (چپ‌ترین = اصلی) |
| `X-Forwarded-Proto` | `http` یا `https` |
| `X-Forwarded-Host` | دامنه‌ی اصلی |
| `X-Forwarded-Port` | پورت اصلی |
| `X-Real-IP` | IP کلاینت (Nginx) |
| `Forwarded` (RFC 7239) | استاندارد جدیدتر |

**۴) خطر: هدر قابل جعل است**

اگر کلاینت مستقیم به سرور وصل شود و `X-Forwarded-For: 1.2.3.4` بفرستد، سرور آن را باور می‌کند. راه‌حل: **فقط به پروکسی‌های شناخته‌شده اعتماد کن**.

```nginx
set_real_ip_from 10.0.0.0/8;
set_real_ip_from 192.168.56.0/24;
real_ip_header X-Forwarded-For;
real_ip_recursive on;
```

**۵) `X-Forwarded-For` با چند IP**

```
X-Forwarded-For: 203.0.113.45, 10.0.0.5, 10.0.0.6
```

- `203.0.113.45` = کلاینت اصلی
- بقیه = پروکسی‌های میانی

`real_ip_recursive on` از راست شروع می‌کند و اولین IP غیرمعتمد را به‌عنوان کلاینت می‌گیرد — **درست‌ترین رفتار**.

**۶) فایروال — سه لایه**

| لایه | ابزار |
|---|---|
| شبکه | Security Group، فایروال سخت‌افزاری |
| سیستم‌عامل | `iptables`/`nftables`، `ufw`، Windows Firewall |
| اپلیکیشن | `allow`/`deny` در Nginx/Apache، IIS IP Restrictions |

**۷) `ufw` — ساده و کافی برای اکثر موارد**

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status verbose
```

**۸) `nftables` — مدرن و قدرتمند**

```bash
sudo nft add table inet filter
sudo nft add chain inet filter input '{ type filter hook input priority 0; policy drop; }'
sudo nft add rule inet filter input ct state established,related accept
sudo nft add rule inet filter input iif lo accept
sudo nft add rule inet filter input tcp dport {22,80,443} accept
sudo nft add rule inet filter input icmp type echo-request accept
```

**۹) `Windows Firewall`**

```powershell
New-NetFirewallRule -DisplayName "HTTP" -Direction Inbound -Protocol TCP -LocalPort 80 -Action Allow
New-NetFirewallRule -DisplayName "HTTPS" -Direction Inbound -Protocol TCP -LocalPort 443 -Action Allow
Get-NetFirewallRule -Direction Inbound -Enabled True | Select DisplayName, LocalPort
```

### مثال‌های عملی

**Lab Scenario: پشت پروکسی، IP واقعی را ببین**

```nginx
# /etc/nginx/conf.d/lab-realip.conf
server {
    listen 8080;
    server_name _;

    # اعتماد فقط به پروکسی داخلی
    set_real_ip_from 127.0.0.1;
    set_real_ip_from 172.16.0.0/12;
    real_ip_header X-Forwarded-For;
    real_ip_recursive on;

    location /whoami {
        default_type text/plain;
        return 200 "remote_addr=$remote_addr\nx_forwarded_for=$http_x_forwarded_for\nscheme=$scheme\n";
    }
}
```

```bash
curl -s http://localhost:8080/whoami
# remote_addr=127.0.0.1
# x_forwarded_for=

# شبیه‌سازی پروکسی
curl -s -H 'X-Forwarded-For: 203.0.113.45, 10.0.0.5' http://localhost:8080/whoami
# remote_addr=203.0.113.45      ← IP واقعی
# x_forwarded_for=203.0.113.45, 10.0.0.5
```

**بدون `set_real_ip_from` (خطر):**

```bash
curl -s -H 'X-Forwarded-For: 1.2.3.4' http://localhost:8080/whoami
# اگر real_ip فعال نبود: remote_addr=127.0.0.1 (امن)
# اگر real_ip بدون set_real_ip_from بود: remote_addr=1.2.3.4 (جعل!)
```

**Apache:**

```apache
# mod_remoteip
RemoteIPHeader X-Forwarded-For
RemoteIPTrustedProxy 10.0.0.0/8
RemoteIPTrustedProxy 192.168.56.0/24
```

```apache
# لاگ با IP واقعی
LogFormat "%a %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %D" realip
CustomLog ${APACHE_LOG_DIR}/access.log realip
```

**IIS:**

```xml
<!-- web.config -->
<system.webServer>
  <proxy enabled="true" preserveHostHeader="true" />
  <security>
    <ipSecurity allowUnlisted="true">
      <add ipAddress="203.0.113.0" subnetMask="255.255.255.0" allowed="false" />
    </ipSecurity>
  </security>
</system.webServer>
```

```powershell
# نصب ماژول
Install-WindowsFeature Web-IP-Security
# تنظیم ARR برای X-Forwarded-For
```

**فایروال با محدودیت نرخ (nftables):**

```bash
# حداکثر ۲۰ اتصال جدید در دقیقه از هر IP
sudo nft add rule inet filter input tcp dport 443 \
  ct state new limit rate over 20/minute counter drop
```

**تست فایروال از بیرون:**

```bash
# از ماشین دیگر
nc -zv 203.0.113.10 80
nc -zv 203.0.113.10 22

# اسکن پورت (فقط روی سرور خودت)
nmap -Pn -p 1-1024 203.0.113.10
```

### مقایسه و الگوهای مختلف

**سه روش انتقال IP واقعی**

| روش | هدر | نیاز به اعتماد |
|---|---|---|
| `X-Forwarded-For` | زنجیره | ✅ |
| `X-Real-IP` | تک | ✅ |
| `Forwarded` (RFC 7239) | استاندارد | ✅ |
| `PROXY protocol` | سطح TCP | ✅ (پورت اختصاصی) |

**`PROXY protocol` — وقتی هدر کافی نیست**

در L4 (TCP)، هدر HTTP وجود ندارد. `PROXY protocol` یک خط قبل از داده می‌فرستد:

```
PROXY TCP4 203.0.113.45 10.0.0.5 51234 443\r\n
```

```nginx
# Nginx به‌عنوان مقصد
listen 80 proxy_protocol;
set_real_ip_from 10.0.0.0/8;
real_ip_header proxy_protocol;
```

**`ufw` در برابر `nftables` در برابر `iptables`**

| | `ufw` | `nftables` | `iptables` |
|---|---|---|---|
| سادگی | ✅✅ | ⚠️ | ⚠️ |
| قدرت | ⚠️ | ✅✅ | ✅ |
| آینده | ✅ (روی nft) | ✅ | ❌ (legacy) |
| مناسب | اکثر | پیچیده | قدیمی |

**دیدگاه «فایروال شبکه کافی است»**: نه. دفاع لایه‌ای: Security Group + فایروال OS + `allow/deny` اپ. هر لایه، خطای لایه‌ی دیگر را می‌گیرد.

### اشکال شایع

**۱) باور `X-Forwarded-For` بدون اعتماد**
جعل IP، دور زدن rate limit، آلودگی لاگ. `set_real_ip_from`.

**۲) `real_ip_recursive off`**
با چند پروکسی، IP اشتباه. `on`.

**۳) `set_real_ip_from 0.0.0.0/0`**
همه را باور می‌کند. فاجعه.

**۴) `X-Forwarded-Proto` نادیده**
`redirect http→https` بی‌پایان (loop). (→ فصل ۳۴)

**۵) فایروال که ۸۰ را باز کرده ولی ۴۴۳ نه**
HTTPS timeout. `ufw status`.

**۶) `iptables` و `ufw` همزمان**
تعارض. یکی را انتخاب کن.

**۷) `fail2ban` که `iptables` را عوض می‌کند**
با `nftables` ممکن است کار نکند. نسخه‌ی جدید `fail2ban` از `nftables` پشتیبانی می‌کند.

**۸) `allow/deny` ترتیب اشتباه**
در Nginx، `allow`/`deny` به ترتیب بررسی می‌شوند؛ `deny all` آخر.

**۹) NAT و `hairpin`**
دسترسی به IP عمومی از داخل شبکه، ممکن است کار نکند. split-horizon DNS.

**۱۰) IPv6 بدون فایروال**
فایروال فقط IPv4 تنظیم شده و IPv6 باز است. `ip6tables`/`nft` را چک کن.

### نکات مهم (Tips & Tricks)

- **`set_real_ip_from`** فقط برای پروکسی‌های خودت.
- **`real_ip_recursive on`**.
- **`X-Forwarded-Proto`** را در redirect استفاده کن.
- **`ufw status verbose`** بعد از هر تغییر.
- **`nft list ruleset`** برای دیدن قواعد.
- **`tcpdump -nn -i any port 80`** برای دیدن IP واقعی در بسته.
- **`fail2ban-client status`** برای وضعیت بلاک.

### خلاصه فصل

- NAT و پروکسی، IP و پروتکل اصلی را پنهان می‌کنند.
- `X-Forwarded-For`/`Proto`/`Host` استانداردند، ولی قابل جعل.
- `set_real_ip_from` فقط به پروکسی‌های شناخته‌شده اعتماد می‌کند.
- `real_ip_recursive on` با چند پروکسی درست کار می‌کند.
- فایروال سه لایه: شبکه، OS، اپ.
- `PROXY protocol` برای L4.

**سؤال مروری:** چرا `set_real_ip_from 0.0.0.0/0` یک آسیب‌پذیری امنیتی است؟

---

## فصل ۸: WebSocket، SSE و اتصال‌های طولانی

### مقدمه

HTTP برای «درخواست-پاسخ کوتاه» طراحی شد. ولی چت، نوتیفیکیشن، تخته‌ی همکاری و داشبورد زنده، به اتصال طولانی نیاز دارند. سه راه وجود دارد و هر کدام، پیکربندی متفاوتی در وب‌سرور می‌خواهد.

### مفاهیم اساسی

**۱) سه راه ارتباط زنده**

| روش | جهت | پروتکل | پیچیدگی |
|---|---|---|---|
| Polling | کلاینت → سرور | HTTP | ✅ ساده |
| SSE | سرور → کلاینت | HTTP | ✅ متوسط |
| WebSocket | دوطرفه | HTTP upgrade → WS | ⚠️ |

**۲) WebSocket — upgrade**

```http
GET /chat HTTP/1.1
Host: lab.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

پاسخ:

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

از این لحظه، اتصال TCP یک تونل دوطرفه است، نه HTTP.

**۳) چرا پروکسی، WebSocket را می‌شکند**

| مشکل | علت | راه‌حل |
|---|---|---|
| `400` یا `426` | `Upgrade` حذف شده | `proxy_set_header Upgrade $http_upgrade;` |
| قطع بعد از ۶۰ ثانیه | `proxy_read_timeout` کم | افزایش timeout |
| بافر شدن پیام‌ها | `proxy_buffering on` | `off` |
| `Connection: close` | HTTP/1.0 به upstream | `proxy_http_version 1.1;` |

**۴) SSE — Server-Sent Events**

```http
GET /events HTTP/1.1
Accept: text/event-stream
```

```http
HTTP/1.1 200 OK
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive

data: {"temp": 22.5}

data: {"temp": 22.7}

id: 42
event: alert
data: {"msg": "دما بالا"}
```

مزیت: فقط HTTP، بدون upgrade، reconnect خودکار در `EventSource`.
عیب: یک‌طرفه (سرور → کلاینت).

**۵) Long Polling**

کلاینت درخواست می‌زند، سرور تا مدت طولانی نگه می‌دارد تا داده باشد:

```
کلاینت: GET /poll?since=42
سرور:  (نگه می‌دارد ۲۵ ثانیه)
سرور:  200 [{"id":43,...}]
کلاینت: GET /poll?since=43
```

عیب: هر پیام، یک درخواست HTTP جدید.

**۶) `Sec-WebSocket-*` و امنیت**

- `Origin` را چک کن (جلوگیری از CSWSH — Cross-Site WebSocket Hijacking).
- احراز هویت با کوکی/توکن در handshake.
- `wss://` (WebSocket over TLS) در تولید.

**۷) تنظیمات کلیدی Nginx**

```nginx
proxy_http_version 1.1;
proxy_set_header Upgrade    $http_upgrade;
proxy_set_header Connection $connection_upgrade;   # با map
proxy_read_timeout 3600s;
proxy_send_timeout 3600s;
proxy_buffering off;
```

**۸) `map` برای `Connection`**

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}
```

بدون این `map`، اگر `Connection: upgrade` را hardcode کنی، درخواست‌های غیر WebSocket هم `upgrade` می‌گیرند و بعضی upstreamها خطا می‌دهند.

**۹) `HTTP/2` و WebSocket**

WebSocket روی HTTP/2 (RFC 8441) وجود دارد (`:protocol`)، ولی پشتیبانی محدود است. عملاً WebSocket روی HTTP/1.1 می‌رود.

**۱۰) مقیاس**

| مسئله | راه‌حل |
|---|---|
| چند سرور | pub/sub (Redis) برای پخش پیام |
| sticky session | برای WebSocket لازم است (یا stateless design) |
| تعداد اتصال | `worker_connections` و fd |
| زمان | timeout طولانی، heartbeat |

### مثال‌های عملی

**Lab Scenario: WebSocket پشت Nginx**

```nginx
# /etc/nginx/conf.d/lab-ws.conf
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

upstream ws_backend {
    server 127.0.0.1:3000;
    keepalive 32;
}

server {
    listen 80;
    server_name lab.example.com;

    location /ws {
        proxy_pass http://ws_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade    $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host       $host;
        proxy_set_header X-Real-IP  $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_buffering off;
        proxy_read_timeout 3600s;
        proxy_send_timeout 3600s;
    }

    location / {
        proxy_pass http://ws_backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
    }
}
```

**سرور WebSocket حداقلی با Node.js:**

```javascript
// server.js — نیاز به: npm i ws
import { WebSocketServer } from 'ws';
import http from 'node:http';

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('HTTP OK\n');
});

const wss = new WebSocketServer({ server, path: '/ws' });

wss.on('connection', (ws, req) => {
  const ip = req.headers['x-real-ip'] ?? req.socket.remoteAddress;
  console.log('client connected:', ip);

  const timer = setInterval(() => {
    ws.send(JSON.stringify({ t: Date.now(), msg: 'tick' }));
  }, 1000);

  ws.on('message', (data) => {
    ws.send(JSON.stringify({ echo: data.toString() }));
  });

  ws.on('close', () => {
    clearInterval(timer);
    console.log('client disconnected');
  });
});

server.listen(3000, () => console.log('app on :3000'));
```

```bash
node server.js &
sudo nginx -t && sudo systemctl reload nginx
```

**تست WebSocket:**

```bash
# با websocat (نصب: cargo install websocat یا apt)
websocat ws://lab.example.com/ws
# tick هر ثانیه می‌آید

# یا با curl (فقط handshake)
curl -sS -i -N \
  -H 'Connection: Upgrade' \
  -H 'Upgrade: websocket' \
  -H 'Sec-WebSocket-Version: 13' \
  -H 'Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==' \
  http://lab.example.com/ws
# باید HTTP/1.1 101 Switching Protocols ببینی
```

**SSE پشت Nginx:**

```nginx
location /events {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;
    proxy_set_header Connection '';
    proxy_buffering off;
    proxy_cache off;
    chunked_transfer_encoding off;
    proxy_read_timeout 24h;
}
```

```javascript
// سرور SSE
app.get('/events', (req, res) => {
  res.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive',
    'X-Accel-Buffering': 'no',   // مهم برای Nginx
  });

  const timer = setInterval(() => {
    res.write(`data: ${JSON.stringify({ t: Date.now() })}\n\n`);
  }, 1000);

  req.on('close', () => clearInterval(timer));
});
```

```bash
curl -N http://lab.example.com/events
```

**Apache:**

```apache
# mod_proxy_wstunnel
<VirtualHost *:80>
    ServerName lab.example.com
    ProxyPass        /ws  ws://127.0.0.1:3000/ws
    ProxyPassReverse /ws  ws://127.0.0.1:3000/ws
    ProxyPass        /    http://127.0.0.1:3000/
    ProxyPassReverse /    http://127.0.0.1:3000/
</VirtualHost>
```

```bash
sudo a2enmod proxy proxy_http proxy_wstunnel
sudo apachectl configtest && sudo systemctl reload apache2
```

**IIS (ARR + WebSocket):**

```powershell
# نصب WebSocket Protocol
Install-WindowsFeature Web-WebSockets

# ARR باید نصب باشد؛ در IIS Manager → Server → Application Request Routing Cache → Server Proxy Settings → Enable
# در site → URL Rewrite → Reverse Proxy rule به localhost:3000
```

```xml
<!-- web.config برای ARR -->
<system.webServer>
  <webSocket enabled="true" />
  <rewrite>
    <rules>
      <rule name="ReverseProxy" stopProcessing="true">
        <match url="(.*)" />
        <action type="Rewrite" url="http://127.0.0.1:3000/{R:1}" />
      </rule>
    </rules>
  </rewrite>
</system.webServer>
```

### مقایسه و الگوهای مختلف

**سه روش ارتباط زنده**

| معیار | Polling | SSE | WebSocket |
|---|---|---|---|
| جهت | یک‌طرفه | سرور→کلاینت | دوطرفه |
| overhead | ❌ زیاد | ✅ کم | ✅ کمترین |
| reconnect | دستی | ✅ خودکار | دستی |
| پروکسی | ✅ ساده | ⚠️ (buffer) | ❌ (upgrade) |
| مقیاس | ⚠️ | ✅ | ⚠️ (state) |
| مناسب | وضعیت ساده | نوتیفیکیشن، فید | چت، بازی، همکاری |

**WebSocket در برابر SSE در انتخاب**

| سؤال | WebSocket | SSE |
|---|---|---|
| کلاینت باید بفرستد؟ | ✅ | ❌ |
| نیاز به HTTP/2 multiplexing | ⚠️ | ✅ |
| فایروال سخت‌گیر | ⚠️ | ✅ |
| پیاده‌سازی ساده | ⚠️ | ✅ |

**دیدگاه «همیشه WebSocket»**: نه. برای نوتیفیکیشن یک‌طرفه، SSE ساده‌تر و مقیاس‌پذیرتر است. WebSocket برای تعامل دوطرفه‌ی واقعی.

### اشکال شایع

**۱) `proxy_set_header Upgrade` نبودن**
`400 Bad Request` یا پاسخ HTTP عادی. هدر لازم.

**۲) `proxy_http_version 1.0`**
WebSocket کار نمی‌کند. `1.1`.

**۳) `proxy_read_timeout 60s`**
اتصال هر دقیقه قطع می‌شود. `3600s` یا heartbeat.

**۴) `proxy_buffering on` برای SSE**
پیام‌ها جمع می‌شوند و یک‌جا می‌آیند. `off` + `X-Accel-Buffering: no`.

**۵) `Connection: upgrade` hardcode**
درخواست‌های عادی هم upgrade می‌شوند. `map`.

**۶) نبود heartbeat**
پروکسی‌ها اتصال بی‌صدا را می‌بندند. ping/pong هر ۳۰ ثانیه.

**۷) WebSocket بدون TLS**
`ws://` در تولید. `wss://`.

**۸) نبود `Origin` check**
CSWSH. سرور باید `Origin` را با دامنه‌ی مجاز مقایسه کند.

**۹) sticky session نبودن**
کاربر به سرور دیگر می‌رود و اتصال قطع می‌شود. `ip_hash` یا pub/sub.

**۱۰) `worker_connections` کم**
با ۱۰ هزار WebSocket، اتصال‌ها رد می‌شوند. `worker_connections` را بالا ببر.

### نکات مهم (Tips & Tricks)

- **`map $http_upgrade`** استاندارد.
- **`proxy_buffering off`** برای stream.
- **`X-Accel-Buffering: no`** از سمت اپ.
- **heartbeat** هر ۳۰ ثانیه.
- **`websocat`** برای تست.
- **`curl -N`** برای SSE.
- **`ss -tan state established '( sport = :443 )' | wc -l`** شمارش اتصال.

### خلاصه فصل

- WebSocket با `Upgrade` از HTTP شروع می‌شود و دوطرفه می‌شود.
- پروکسی باید `Upgrade`/`Connection` را پاس بدهد و timeout را بالا ببرد.
- SSE یک‌طرفه، ساده و مقاوم است؛ `buffering off` لازم دارد.
- `map` برای `Connection` از hardcode جلوگیری می‌کند.
- heartbeat برای جلوگیری از قطع بی‌صدا.
- `wss://` و بررسی `Origin` الزامی.

**سؤال مروری:** چرا `proxy_read_timeout 60s` باعث قطع شدن WebSocket می‌شود و دو راه حل چیست؟

---

## منابع و مراجع

**مستندات رسمی**
- [RFC 9293 — TCP](https://www.rfc-editor.org/rfc/rfc9293.html)
- [RFC 6455 — WebSocket](https://www.rfc-editor.org/rfc/rfc6455.html)
- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
- [MDN — Server-Sent Events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)
- [Nginx — WebSocket proxying](https://nginx.org/en/docs/http/websocket.html)
- [Apache — mod_proxy_wstunnel](https://httpd.apache.org/docs/2.4/mod/mod_proxy_wstunnel.html)
- [HAProxy — PROXY protocol](https://www.haproxy.org/download/2.9/doc/proxy-protocol.txt)

**ابزارها**
- [websocat](https://github.com/vi/websocat)
- [nmap](https://nmap.org/)
- [dig / drill](https://www.isc.org/bind/)
- [mtr](https://github.com/traviscross/mtr)

---

**بعدی:** [`03-Architectures-and-Comparison.md`](03-Architectures-and-Comparison.md) — Apache، Nginx، IIS و انتخاب درست
