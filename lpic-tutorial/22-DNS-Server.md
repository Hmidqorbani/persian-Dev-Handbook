# فایل ۲۲ — سرور DNS

**مدت یادگیری**: ۸ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: فایل ۱۳ (شبکه)، فایل ۲۰ (پیکربندی شبکه)

---

## فهرست محتوا

- [فصل ۷۷: پیکربندی پایهٔ سرور DNS](#فصل-۷۷-پیکربندی-پایهٔ-سرور-dns)
- [فصل ۷۸: ساخت و نگهداری Zones](#فصل-۷۸-ساخت-و-نگهداری-zones)
- [فصل ۷۹: امن‌سازی سرور DNS](#فصل-۷۹-امنسازی-سرور-dns)

---

## فصل ۷۷: پیکربندی پایهٔ سرور DNS

### مقدمه

**DNS (Domain Name System)** نام دامنه‌ها را به آدرس IP ترجمه می‌کند — «دفترچه تلفن اینترنت». تصور کنید یک شرکت دارید و می‌خواهید سرور DNS داخلی راه‌اندازی کنید تا کامپیوترها نام‌های داخلی را resolve کنند.

در این فصل راه‌اندازی سرور DNS با **BIND** را یاد می‌گیرید. این Topic در آزمون LPIC-2 وزن دارد.

### مفاهیم اساسی

**انواع سرور DNS:**

| نوع | توضیح |
|---|---|
| **Authoritative** | مرجع یک zone |
| **Recursive** | پاسخ از منابع دیگر |
| **Caching** | کش نتایج |
| **Forwarder** | ارجاع به سرور دیگر |

**اجزای BIND:**

- **named:** daemon.
- **named.conf:** پیکربندی اصلی.
- **zone files:** فایل‌های منطقه.
- **rndc:** کنترل از راه دور.

**انواع رکورد:**

| رکورد | معنا |
|---|---|
| SOA | Start of Authority |
| NS | Name Server |
| A | IPv4 |
| AAAA | IPv6 |
| CNAME | نام مستعار |
| MX | سرور ایمیل |
| PTR | Reverse |
| TXT | متن (SPF و غیره) |
| SRV | سرویس |

**محل فایل‌ها:**

| مسیر | محتوا |
|---|---|
| `/etc/bind/named.conf` | پیکربندی (Debian) |
| `/etc/named.conf` | پیکربندی (RHEL) |
| `/var/cache/bind/` | کش |
| `/var/named/` | zone files (RHEL) |

### مثال‌های عملی

**نصب BIND:**

```bash
# Debian/Ubuntu
sudo apt install bind9 bind9utils bind9-doc

# RHEL/Rocky
sudo dnf install bind bind-utils

# فعال‌سازی
sudo systemctl enable --now named
```

**پیکربندی پایه (`/etc/bind/named.conf.options`):**

```
options {
    directory "/var/cache/bind";
    recursion yes;
    allow-recursion { 192.168.1.0/24; localhost; };
    forwarders {
        8.8.8.8;
        1.1.1.1;
    };
    dnssec-validation auto;
    listen-on { any; };
    listen-on-v6 { any; };
};
```

**بررسی:**

```bash
# بررسی syntax
sudo named-checkconf

# وضعیت
sudo systemctl status named
sudo rndc status
sudo rndc reload
```

**پیکربندی کلاینت:**

```bash
# /etc/resolv.conf
nameserver 192.168.1.10
```

### مقایسه و الگوهای مختلف

**سرورهای DNS:**

| سرور | ویژگی |
|---|---|
| **BIND** | استاندارد، کامل |
| **dnsmasq** | سبک، برای شبکهٔ کوچک |
| **Unbound** | resolver امن |
| **PowerDNS** | با backend دیتابیس |
| **Knot** | سریع، مدرن |

**نقش‌ها:**

| نقش | پیکربندی |
|---|---|
| Caching only | forwarders |
| Authoritative | zones |
| Forwarder | forward only |

**BIND در برابر dnsmasq:**

| ویژگی | BIND | dnsmasq |
|---|---|---|
| کامل بودن | بالا | ساده |
| پیکربندی | پیچیده | ساده |
| مناسب | سرور DNS | شبکهٔ کوچک |

### اشکال شایع

**اشکال ۱: Open Recursion.**
سرور برای همه پاسخ می‌دهد (خطر abuse). راه‌حل: `allow-recursion`.

**اشکال ۲: فراموش کردن allow-recursion.**
یا همه یا هیچ‌کس. راه‌حل: تنظیم دقیق.

**اشکال ۳: خطای syntax در named.conf.**
named بالا نمی‌آید؛ `named-checkconf`. راه‌حل: بررسی.

**اشکال ۴: فایروال پورت ۵۳.**
DNS کار نمی‌کند. راه‌حل: باز کردن (TCP+UDP).

**اشکال ۵: SELinux در RHEL.**
BIND نمی‌تواند فایل بخواند. راه‌حل: context.

**اشکال ۶: فراموش کردن reload.**
تغییرات اعمال نمی‌شود. راه‌حل: `rndc reload`.

### نکات مهم (Tips & Tricks)

- **`named-checkconf` قبل از restart.**
- **`rndc reload` پیکربندی را بدون restart بازخوانی می‌کند.**
- **`dig @localhost example.com` تست محلی.**
- **`dnsmasq` برای شبکهٔ کوچک ساده‌تر است.**
- **recursion را محدود به شبکهٔ داخلی کنید.**
- **DNSSEC را فعال نگه دارید.**
- **`named-checkzone` zone را بررسی می‌کند.**
- **`rndc status` وضعیت را نشان می‌دهد.**
- **`journalctl -u named` لاگ.**
- **`/var/log/named/` لاگ (RHEL).**
- **برای caching، `forwarders` کافی است.**
- **`chroot` امنیت BIND را بالا می‌برد.**

### خلاصه فصل

DNS نام را به IP ترجمه می‌کند. BIND استاندارد است و شامل named، named.conf و zone files می‌شود. سرورها می‌توانند authoritative، recursive یا caching باشند. با `named-checkconf` و `rndc` مدیریت کنید. recursion را محدود کنید تا Open Resolver نشود. فایروال پورت ۵۳ (TCP+UDP) لازم است.

---

## فصل ۷۸: ساخت و نگهداری Zones

### مقدمه

**Zone** بخشی از فضای نام DNS است که یک سرور مرجع آن است. تصور کنید می‌خواهید دامنهٔ `example.com` را میزبانی کنید — باید zone بسازید.

ساخت و نگهداری zone files قلب کار سرور DNS است و در آزمون می‌آید.

### مفاهیم اساسی

**انواع zone:**

- **Forward zone:** نام → IP.
- **Reverse zone:** IP → نام (in-addr.arpa).

**فایل zone شامل:** SOA، NS، A، CNAME، MX، PTR و غیره.

**SOA record:** اطلاعات کلیدی منطقه:

```
@ IN SOA ns1.example.com. admin.example.com. (
    2026101601  ; Serial
    3600        ; Refresh
    1800        ; Retry
    604800      ; Expire
    86400 )     ; Minimum TTL
```

**Serial:** شمارندهٔ نسخه — **هر بار تغییر باید افزایش یابد**.

### مثال‌های عملی

**تعریف zone در `named.conf.local`:**

```
zone "example.com" {
    type master;
    file "/etc/bind/db.example.com";
};

zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.1";
};
```

**فایل zone forward (`/etc/bind/db.example.com`):**

```
$TTL    86400
@   IN  SOA ns1.example.com. admin.example.com. (
        2026101601  ; Serial
        3600        ; Refresh
        1800        ; Retry
        604800      ; Expire
        86400 )     ; Minimum TTL
;
@       IN  NS      ns1.example.com.
@       IN  NS      ns2.example.com.
@       IN  A       192.168.1.10
@       IN  MX  10  mail.example.com.
ns1     IN  A       192.168.1.10
ns2     IN  A       192.168.1.11
www     IN  A       192.168.1.20
mail    IN  A       192.168.1.30
ftp     IN  CNAME   www.example.com.
sub     IN  A       192.168.1.40
```

**فایل zone reverse (`/etc/bind/db.192.168.1`):**

```
$TTL    86400
@   IN  SOA ns1.example.com. admin.example.com. (
        2026101601 3600 1800 604800 86400 )
@       IN  NS      ns1.example.com.
10      IN  PTR     ns1.example.com.
20      IN  PTR     www.example.com.
30      IN  PTR     mail.example.com.
```

**بررسی و بارگذاری:**

```bash
# بررسی zone
sudo named-checkzone example.com /etc/bind/db.example.com
sudo named-checkzone 1.168.192.in-addr.arpa /etc/bind/db.192.168.1

# بارگذاری مجدد
sudo rndc reload
sudo systemctl reload named

# تست
dig @localhost www.example.com
dig @localhost -x 192.168.1.20
```

### مقایسه و الگوهای مختلف

**افزایش Serial:**

| روش | مثال |
|---|---|
| `YYYYMMDDnn` | 2026101601 |
| شمارنده | ۱، ۲، ۳... |
| Unix timestamp | 1729000000 |

نکته: هر بار تغییر zone، **Serial باید افزایش یابد**، وگرنه slave به‌روز نمی‌شود.

**انواع zone:**

| نوع | توضیح |
|---|---|
| `master`/`primary` | مرجع اصلی |
| `slave`/`secondary` | کپی از master |
| `forward` | ارجاع |
| `stub` | فقط NS |
| `hint` | ریشه |

**رکوردها:**

| رکورد | کاربرد |
|---|---|
| A | IPv4 |
| AAAA | IPv6 |
| CNAME | نام مستعار (نه برای دامنهٔ ریشه) |
| MX | ایمیل (با اولویت) |
| TXT | SPF، تأیید |
| SRV | سرویس |

### اشکال شایع

**اشکال ۱: فراموش کردن نقطهٔ انتهایی.**
`www.example.com` (بدون نقطه) به `www.example.com.example.com` تبدیل می‌شود! راه‌حل: نقطهٔ انتهایی.

**اشکال ۲: افزایش ندادن Serial.**
slave به‌روز نمی‌شود. راه‌حل: Serial را افزایش دهید.

**اشکال ۳: خطای syntax.**
`named-checkzone` بگیرید. راه‌حل: بررسی.

**اشکال ۴: PTR نادرست.**
reverse lookup کار نمی‌کند. راه‌حل: PTR درست.

**اشکال ۵: CNAME برای دامنهٔ ریشه.**
مجاز نیست. راه‌حل: A record.

**اشکال ۶: فراموش کردن `$TTL`.**
خطای syntax. راه‌حل: `$TTL` در ابتدا.

### نکات مهم (Tips & Tricks)

- **همیشه نقطهٔ انتهایی در FQDN بگذارید.**
- **`named-checkzone` قبل از reload.**
- **Serial را خودکار کنید (مثلاً تاریخ).**
- **`dig -x 192.168.1.20` تست reverse.**
- **TTL را بر اساس نیاز تنظیم کنید (کوتاه برای تغییرات مکرر).**
- **slave برای افزونگی.**
- **`$ORIGIN` و `$INCLUDE` برای ساده‌سازی.**
- **CNAME را برای دامنهٔ ریشه استفاده نکنید.**
- **MX با اولویت (کمتر = بالاتر).**
- **`dig +norecurse` query بدون recursion.**
- **zone file را در `/etc/bind/` (Debian) یا `/var/named/` (RHEL).**
- **`rndc reload zone` فقط یک zone را reload.**

### خلاصه فصل

Zone بخشی از فضای نام است که سرور مرجع آن است (forward و reverse). فایل zone شامل SOA، NS، A، MX و PTR است. هر تغییر نیاز به افزایش Serial دارد. همیشه نقطهٔ انتهایی FQDN بگذارید و با `named-checkzone` بررسی کنید. slave افزونگی می‌دهد. CNAME برای دامنهٔ ریشه مجاز نیست.

---

## فصل ۷۹: امن‌سازی سرور DNS

### مقدمه

سرور DNS در معرض حملات متعددی است: cache poisoning، amplification، tunneling. تصور کنید سرور DNS شما برای حملهٔ DDoS سوءاستفاده می‌شود — بدون امن‌سازی، این اتفاق می‌افتد.

امن‌سازی حیاتی است و در آزمون می‌آید.

### مفاهیم اساسی

**تهدیدات:**

- **Cache Poisoning:** تزریق پاسخ جعلی.
- **DNS Amplification:** سوءاستفاده برای DDoS.
- **DNS Tunneling:** انتقال داده از طریق DNS.
- **Zone Transfer سرقت:** کپی غیرمجاز zone.
- **DDoS:** حملهٔ حجمی.

**راهکارها:** DNSSEC، TSIG، ACL، Rate Limiting، RPZ.

### مثال‌های عملی

**محدودسازی recursion:**

```
options {
    recursion yes;
    allow-recursion { localhost; 192.168.1.0/24; };
    allow-query { any; };
    allow-transfer { 192.168.1.11; };   // فقط slave
};
```

**TSIG برای zone transfer امن:**

```bash
# تولید کلید
tsig-keygen -a hmac-sha256 transfer-key > /etc/bind/transfer.key

# در named.conf
include "/etc/bind/transfer.key";
zone "example.com" {
    type master;
    file "/etc/bind/db.example.com";
    allow-transfer { key "transfer-key"; };
};
```

**Rate Limiting:**

```
options {
    rate-limit {
        responses-per-second 10;
        window 5;
        ipv4-prefix-length 24;
    };
};
```

**DNSSEC:**

```
options {
    dnssec-validation auto;
    dnssec-enable yes;
};
```

**Response Policy Zone (RPZ):**

```
zone "rpz.local" {
    type master;
    file "/etc/bind/db.rpz";
};

options {
    response-policy { zone "rpz.local"; };
};
```

### مقایسه و الگوهای مختلف

**روش‌های امن‌سازی:**

| روش | محافظت از |
|---|---|
| ACL | دسترسی غیرمجاز |
| DNSSEC | جعل پاسخ |
| TSIG | zone transfer |
| Rate Limit | amplification |
| RPZ | محتوای مخرب |

**انواع حمله و دفاع:**

| حمله | دفاع |
|---|---|
| Cache poisoning | DNSSEC |
| Amplification | Rate Limit، ACL |
| Zone transfer | TSIG، allow-transfer |
| Tunneling | پایش، RPZ |

**Recursion در برابر Authoritative:**

| نقش | امنیت |
|---|---|
| Authoritative (عمومی) | recursion خاموش |
| Recursive (داخلی) | ACL محدود |

### اشکال شایع

**اشکال ۱: Open Resolver.**
سرور برای همه پاسخ می‌دهد. راه‌حل: `allow-recursion`.

**اشکال ۲: allow-transfer باز.**
zone سرقت می‌شود. راه‌حل: محدود به slave.

**اشکال ۳: نبود rate limiting.**
سرور برای amplification سوءاستفاده می‌شود. راه‌حل: rate-limit.

**اشکال ۴: نبود DNSSEC.**
پاسخ‌های جعلی پذیرفته می‌شوند. راه‌حل: DNSSEC.

**اشکال ۵: سرور authoritative با recursion.**
خطر. راه‌حل: جدا کنید.

**اشکال ۶: نبود پایش.**
حمله دیده نمی‌شود. راه‌حل: لاگ + مانیتورینگ.

### نکات مهم (Tips & Tricks)

- **recursion را فقط برای شبکهٔ داخلی.**
- **allow-transfer را محدود به slave.**
- **TSIG کلید امن zone transfer.**
- **rate-limit برای جلوگیری از amplification.**
- **DNSSEC را فعال کنید.**
- **`dig +dnssec` وضعیت DNSSEC را نشان می‌دهد.**
- **سرور را از اینترنت برای recursion باز نگذارید.**
- **authoritative و recursive را جدا کنید.**
- **`rndc` را با کلید محافظت کنید.**
- **RPZ برای فیلترینگ.**
- **پایش query rate.**
- **`dnssec-keygen`/`dnssec-signzone` برای امضا.**

### خلاصه فصل

امن‌سازی DNS شامل محدودسازی recursion (ACL)، TSIG برای zone transfer، rate limiting، DNSSEC و RPZ است. Open Resolver خطرناک است. `allow-transfer` را محدود کنید و DNSSEC را فعال نگه دارید. سرور authoritative و recursive را جدا کنید. پایش query rate برای تشخیص حمله.

---

## منابع و مراجع

- man pages: `man named`, `man named.conf`, `man rndc`, `man dig`, `man named-checkzone`
- BIND 9 Documentation: https://bind9.readthedocs.io
- DNSSEC Guide: https://www.isc.org/dnssec/
- RFC 1034/1035 (DNS): https://www.rfc-editor.org
- dnsmasq: https://thekelleys.org.uk/dnsmasq/doc.html

**بعدی:** [فایل ۲۳ — سرویس‌های HTTP](23-HTTP-Services.md)
