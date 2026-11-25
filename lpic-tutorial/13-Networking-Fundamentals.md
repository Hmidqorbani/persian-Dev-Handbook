# فایل ۱۳ — مبانی شبکه

**مدت یادگیری**: ۸ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۱۲ (سرویس‌های سیستم)

---

## فهرست محتوا

- [فصل ۴۸: مبانی پروتکل‌های اینترنت](#فصل-۴۸-مبانی-پروتکلهای-اینترنت)
- [فصل ۴۹: پیکربندی پایدار شبکه](#فصل-۴۹-پیکربندی-پایدار-شبکه)
- [فصل ۵۰: عیب‌یابی پایهٔ شبکه](#فصل-۵۰-عیبیابی-پایهٔ-شبکه)
- [فصل ۵۱: پیکربندی DNS سمت کلاینت](#فصل-۵۱-پیکربندی-dns-سمت-کلاینت)

---

## فصل ۴۸: مبانی پروتکل‌های اینترنت

### مقدمه

شبکه‌ها بر پایهٔ **مدل TCP/IP** کار می‌کنند. تصور کنید می‌خواهید دو کامپیوتر را به هم وصل کنید — باید بدانید IP چطور کار می‌کند، subnet چیست و gateway چه نقشی دارد. این مفاهیم پایهٔ هر کار شبکه‌ای هستند.

فهم این مدل و مفاهیم آدرس‌دهی، پیش‌نیاز مباحث بعدی (پیکربندی، عیب‌یابی، DNS) است.

### مفاهیم اساسی

**مدل TCP/IP (چهار لایه):**

| لایه | کار | پروتکل‌ها |
|---|---|---|
| **Application** | سرویس‌ها | HTTP, DNS, SSH, SMTP, FTP |
| **Transport** | انتقال | TCP, UDP |
| **Internet** | آدرس‌دهی و مسیریابی | IP, ICMP, ARP |
| **Link** | فیزیکی | Ethernet, Wi-Fi |

**مفاهیم کلیدی:**

- **IP Address:** آدرس منحصربه‌فرد دستگاه.
- **Subnet Mask:** تعیین محدودهٔ شبکه.
- **Gateway:** دروازهٔ خروج به شبکه‌های دیگر.
- **DNS:** ترجمهٔ نام به IP.
- **MAC Address:** آدرس فیزیکی کارت شبکه.

**IPv4:** ۳۲ بیتی، مثل `192.168.1.10` (چهار اکتت).
**IPv6:** ۱۲۸ بیتی، مثل `2001:db8::1`.

**آدرس‌های خصوصی (Private):**

| محدوده | کلاس | تعداد |
|---|---|---|
| `10.0.0.0/8` | A | ۱۶ میلیون |
| `172.16.0.0/12` | B | ۱ میلیون |
| `192.168.0.0/16` | C | ۶۵ هزار |
| `127.0.0.0/8` | loopback | — |
| `169.254.0.0/16` | link-local | — |

**آدرس‌های خاص:**

| آدرس | معنا |
|---|---|
| `127.0.0.1` | localhost |
| `0.0.0.0` | همهٔ رابط‌ها |
| `255.255.255.255` | broadcast |
| `169.254.x.x` | DHCP شکست خورده |

### مثال‌های عملی

**دیدن تنظیمات شبکه:**

```bash
# آدرس‌های IP
ip addr show
ip a
ip -c a              # رنگی
ip -br a             # خلاصه

# مسیرها
ip route show
ip r

# رابط‌های شبکه
ip link show

# اطلاعات آماری
ip -s link

# ابزار قدیمی
ifconfig
netstat -rn
route -n
```

**محاسبهٔ شبکه:**

```bash
# ابزار محاسبه subnet
ipcalc 192.168.1.10/24
# خروجی: Network, Broadcast, HostMin, HostMax, Hosts/Net

# ابزار دیگر
sipcalc 192.168.1.0/24

# محاسبهٔ دستی
# /24 = 255.255.255.0 = 256 آدرس (254 قابل استفاده)
# /25 = 255.255.255.128 = 128 آدرس
# /16 = 255.255.0.0 = 65536 آدرس
```

**نمایش پورت‌ها:**

```bash
# پورت‌های در حال گوش دادن
ss -tlnp
ss -tlnp | grep :80
netstat -tlnp

# اتصالات فعال
ss -tunap
ss -s                # خلاصه

# UDP
ss -ulnp
```

**بررسی MAC و ARP:**

```bash
ip link show
ip neigh
arp -a
```

### مقایسه و الگوهای مختلف

**TCP در برابر UDP:**

| ویژگی | TCP | UDP |
|---|---|---|
| اتصال | connection-oriented | connectionless |
| اطمینان | تضمین تحویل | بدون تضمین |
| ترتیب | حفظ می‌شود | حفظ نمی‌شود |
| سرعت | کندتر | سریع‌تر |
| هدر | بزرگ | کوچک |
| کاربرد | HTTP, SSH, FTP | DNS, VoIP, بازی |

**کلاس‌های IPv4 (تاریخی):**

| کلاس | محدوده اولین اکتت | پیش‌فرض mask | کاربرد |
|---|---|---|---|
| A | 1–126 | /8 | شبکه‌های بزرگ |
| B | 128–191 | /16 | شبکه‌های متوسط |
| C | 192–223 | /24 | شبکه‌های کوچک |
| D | 224–239 | — | multicast |
| E | 240–255 | — | آزمایشی |

**ابزارهای شبکه:**

| ابزار | کار |
|---|---|
| `ip` | مدرن، جامع |
| `ifconfig` | قدیمی |
| `ss` | پورت‌ها (مدرن) |
| `netstat` | پورت‌ها (قدیمی) |
| `ipcalc` | محاسبه subnet |
| `ping` | تست دسترسی |

**IPv4 در برابر IPv6:**

| ویژگی | IPv4 | IPv6 |
|---|---|---|
| اندازه | ۳۲ بیت | ۱۲۸ بیت |
| نمایش | اعشاری نقطه‌ای | هگزادسیمال |
| تعداد | ۴.۳ میلیارد | ۳.۴×۱۰³⁸ |
| NAT | رایج | غیرضروری |
| مثال | 192.168.1.1 | 2001:db8::1 |

### اشکال شایع

**اشکال ۱: گیج شدن subnet mask و CIDR.**
`/24` = `255.255.255.0`. راه‌حل: به خاطر بسپارید.

**اشکال ۲: اشتباه gateway.**
بدون gateway، فقط شبکهٔ محلی در دسترس است. راه‌حل: gateway درست.

**اشکال ۳: تداخل IP.**
دو دستگاه با یک IP → اختلال شبکه. راه‌حل: IP یکتا.

**اشکال ۴: فراموش کردن loopback.**
`127.0.0.1` (localhost) باید همیشه فعال باشد. راه‌حل: بررسی.

**اشکال ۵: اشتباه در broadcast.**
broadcast اشتباه → ارتباط ناموفق. راه‌حل: محاسبه درست.

**اشکال ۶: گیج شدن با `0.0.0.0`.**
`0.0.0.0` = همهٔ رابط‌ها (برای listen) یا مسیر پیش‌فرض. راه‌حل: context.

### نکات مهم (Tips & Tricks)

- **`ip` جایگزین مدرن `ifconfig`/`route` است.**
- **`ss` جایگزین سریع‌تر `netstat` است.**
- **`ipcalc` برای محاسبهٔ سریع subnet عالی است.**
- **`ping 127.0.0.1` تست loopback است.**
- **`hostname -I` آدرس IP را نشان می‌دهد.**
- **`ip -c a` خروجی رنگی می‌دهد.**
- **`ip route get 8.8.8.8` مسیر به مقصد را نشان می‌دهد.**
- **`ip -br a` خلاصهٔ رابط‌ها.**
- **`/etc/services` نام پورت‌های استاندارد را دارد.**
- **`ss -tulnp` همهٔ پورت‌های شنود (TCP+UDP).**
- **`nmap` برای اسکن شبکه.**
- **`ip neigh flush all` ARP را پاک می‌کند.**

### خلاصه فصل

شبکه بر مدل TCP/IP (چهار لایه) کار می‌کند. IP آدرس‌دهی، subnet محدوده و gateway خروج را تعیین می‌کند. TCP اتصال‌گرا و قابل‌اعتماد، UDP سریع و بدون تضمین است. آدرس‌های خصوصی برای شبکه‌های داخلی‌اند. ابزارهای `ip`, `ss`, `ipcalc` پایهٔ کار شبکه‌اند. `/24` معادل `255.255.255.0` است.

---

## فصل ۴۹: پیکربندی پایدار شبکه

### مقدمه

تنظیمات شبکه باید **پایدار (persistent)** باشند — یعنی بعد از ری‌استارت باقی بمانند. تصور کنید IP را با `ip addr add` تنظیم می‌کنید؛ بعد از reboot از بین می‌رود. برای پایداری باید فایل پیکربندی استفاده کنید.

روش پیکربندی در توزیع‌های مختلف متفاوت است. این فصل همه را پوشش می‌دهد.

### مفاهیم اساسی

**دو رویکرد:**

- **NetworkManager:** مدرن، پویا (nmcli, nmtui).
- **فایل‌های پیکربندی:** سنتی (ifcfg, netplan, interfaces).

**تفاوت توزیع‌ها:**

| توزیع | سیستم |
|---|---|
| Debian قدیمی | `/etc/network/interfaces` |
| Ubuntu جدید | Netplan (`/etc/netplan/`) |
| RHEL/Rocky | `/etc/sysconfig/network-scripts/` یا NetworkManager |

**مفاهیم کلیدی:**

- **DHCP:** دریافت خودکار IP.
- **Static:** IP ثابت.
- **Interface:** رابط شبکه (eth0, ens33, enp0s3).

### مثال‌های عملی

**NetworkManager (nmcli):**

```bash
# نمایش وضعیت
nmcli device status
nmcli connection show
nmcli con show "eth0"

# تنظیم IP استاتیک
sudo nmcli connection modify "eth0" \
  ipv4.method manual \
  ipv4.addresses 192.168.1.10/24 \
  ipv4.gateway 192.168.1.1 \
  ipv4.dns "8.8.8.8 1.1.1.1"

# اعمال
sudo nmcli connection up "eth0"

# DHCP
sudo nmcli connection modify "eth0" ipv4.method auto
sudo nmcli connection up "eth0"

# افزودن اتصال جدید
sudo nmcli connection add type ethernet con-name myeth ifname eth0
```

**nmtui:** رابط متنی تعاملی (ساده‌تر).

**Netplan (Ubuntu):**

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      addresses: [192.168.1.10/24]
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```

```bash
sudo netplan apply
sudo netplan try      # تست با بازگشت خودکار
sudo netplan generate
```

**Debian (`/etc/network/interfaces`):**

```
auto eth0
iface eth0 inet static
    address 192.168.1.10
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 1.1.1.1
```

```bash
sudo systemctl restart networking
sudo ifup eth0
sudo ifdown eth0
```

**RHEL (ifcfg):**

```
# /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE=Ethernet
BOOTPROTO=none
IPADDR=192.168.1.10
PREFIX=24
GATEWAY=192.168.1.1
DNS1=8.8.8.8
ONBOOT=yes
```

```bash
sudo nmcli connection reload
sudo systemctl restart network
```

### مقایسه و الگوهای مختلف

**ابزارهای پیکربندی:**

| ابزار | مزیت | معایب |
|---|---|---|
| `nmcli` | اسکریپت‌پذیر، یکپارچه | منحنی یادگیری |
| `nmtui` | ساده، تعاملی | محدود |
| Netplan | YAML خوانا | فقط Ubuntu |
| ifcfg | مستقیم | منسوخ در حال حاضر |
| `interfaces` | ساده | Debian قدیمی |

**Static در برابر DHCP:**

| ویژگی | Static | DHCP |
|---|---|---|
| IP | ثابت | خودکار |
| مناسب | سرور | کلاینت |
| مدیریت | دستی | متمرکز |
| تعارض | کم | کم |

**روش‌های موقت و پایدار:**

| روش | پایداری |
|---|---|
| `ip addr add` | تا reboot |
| `nmcli` | پایدار |
| `netplan` | پایدار |
| `ifcfg` | پایدار |

### اشکال شایع

**اشکال ۱: اشتباه YAML در Netplan.**
فاصله‌ها حساس‌اند؛ `netplan try` استفاده کنید. راه‌حل: YAML درست.

**اشکال ۲: تداخل NetworkManager و ifcfg.**
یکی را انتخاب کنید. راه‌حل: یکسان.

**اشکال ۳: فراموش کردن ONBOOT.**
رابط در بوت فعال نمی‌شود. راه‌حل: `ONBOOT=yes`.

**اشکال ۴: gateway اشتباه.**
بدون آن، خروج به اینترنت نیست. راه‌حل: gateway درست.

**اشکال ۵: فراموش کردن apply.**
تغییرات اعمال نمی‌شود. راه‌حل: `netplan apply`/`nmcli con up`.

**اشکال ۶: DNS در جای اشتباه.**
`resolv.conf` ممکن است بازنویسی شود. راه‌حل: در ابزار پایدار.

### نکات مهم (Tips & Tricks)

- **`nmcli` با Tab خودکارسازی می‌شود.**
- **`netplan try` بعد از ۱۲۰ ثانیه خودکار برمی‌گردد (ایمن).**
- **برای تغییر سریع، `ip addr add` موقت است (نه پایدار).**
- **`nmcli connection reload` بعد از ویرایش دستی فایل.**
- **همیشه تنظیمات را با `ping` و `ip route` تست کنید.**
- **`nmcli con mod X ipv4.dns "8.8.8.8"` DNS تنظیم می‌کند.**
- **`netplan get` پیکربندی فعلی را نشان می‌دهد.**
- **`ip -br a` تأیید نهایی.**
- **برای سرور، static بهتر است.**
- **`nmtui` برای مبتدیان ساده‌تر.**
- **`systemd-networkd` گزینهٔ دیگر (سبک).**
- **همیشه یک راه دسترسی پشتیبان (console) داشته باشید.**

### خلاصه فصل

پیکربندی پایدار شبکه در توزیع‌های مختلف متفاوت است: NetworkManager (nmcli/nmtui)، Netplan (Ubuntu) و ifcfg (RHEL). `nmcli connection modify` روش مدرن و یکپارچه است. Netplan از YAML با `netplan apply` استفاده می‌کند. بعد از هر تغییر، با `ping` و `ip route` تست کنید. برای سرور static و برای کلاینت DHCP مناسب است.

---

## فصل ۵۰: عیب‌یابی پایهٔ شبکه

### مقدمه

وقتی شبکه کار نمی‌کند، باید سیستماتیک عیب‌یابی کنید. تصور کنید کاربر می‌گوید «اینترنت قطع است» — از کجا شروع می‌کنید؟ با یک رویکرد لایه‌ای، سریع ریشه را پیدا می‌کنید.

در این فصل ابزارها و روش‌های عیب‌یابی را یاد می‌گیرید.

### مفاهیم اساسی

**رویکرد لایه‌ای:** از پایین به بالا یا بالا به پایین:

```
۱. فیزیکی (کابل، رابط)
۲. IP (آدرس، subnet)
۳. مسیریابی (gateway، route)
۴. DNS (ترجمهٔ نام)
۵. سرویس (پورت، برنامه)
```

**جدول تشخیص:**

| علامت | لایه | احتمال |
|---|---|---|
| no link | 1 | کابل/رابط |
| ping IP کار، نام نه | 4 | DNS |
| ping gateway نه | 2/3 | IP/routing |
| پورت بسته | 5 | فایروال/سرویس |

### مثال‌های عملی

**تست اتصال:**

```bash
# loopback
ping 127.0.0.1

# شبکهٔ محلی
ping 192.168.1.1

# اینترنت (IP)
ping 8.8.8.8

# اینترنت (نام) — تست DNS
ping google.com

# تعداد مشخص
ping -c 4 google.com

# اندازهٔ بسته
ping -s 1400 google.com
```

**ردیابی مسیر:**

```bash
traceroute google.com
tracepath google.com
mtr google.com         # ترکیبی، زنده
mtr -r -c 10 google.com  # report
```

**بررسی پورت و سرویس:**

```bash
# پورت باز؟
ss -tlnp
telnet host 80
nc -zv host 443
nmap -p 80,443 host

# تست HTTP
curl -I http://example.com
curl -v http://example.com
wget --spider http://example.com
```

**دیدن پیکربندی:**

```bash
ip addr
ip route
cat /etc/resolv.conf
cat /etc/hosts
ss -tlnp
```

**تحلیل ترافیک:**

```bash
# tcpdump
sudo tcpdump -i eth0
sudo tcpdump -i eth0 port 80
sudo tcpdump -i eth0 host 192.168.1.10
sudo tcpdump -i eth0 -w capture.pcap

# آمار رابط
ip -s link
```

### مقایسه و الگوهای مختلف

**ابزارهای عیب‌یابی:**

| ابزار | کاربرد | لایه |
|---|---|---|
| `ping` | تست دسترسی | 3 |
| `traceroute` | مسیر بسته‌ها | 3 |
| `mtr` | traceroute زنده | 3 |
| `ss`/`netstat` | پورت‌ها | 4 |
| `dig`/`nslookup` | DNS | 7 |
| `tcpdump` | تحلیل بسته | همه |
| `curl`/`wget` | تست HTTP | 7 |
| `nmap` | اسکن پورت | 4 |

**تفسیر نتایج:**

| تست | نتیجه | معنا |
|---|---|---|
| ping 127.0.0.1 | کار | IP stack سالم |
| ping gateway | کار | شبکهٔ محلی سالم |
| ping 8.8.8.8 | کار | اینترنت سالم |
| ping google.com | نه | مشکل DNS |

**ابزارهای مسیر:**

| ابزار | ویژگی |
|---|---|
| `traceroute` | کلاسیک |
| `tracepath` | بدون ریشه |
| `mtr` | زنده، ترکیبی |

### اشکال شایع

**اشکال ۱: اشتباه گرفتن مشکل DNS با شبکه.**
اگر IP کار کند ولی نام نه، DNS است. راه‌حل: `dig`.

**اشکال ۲: فایروال داخلی.**
`ufw` یا `firewalld` پورت را می‌بندد. راه‌حل: بررسی فایروال.

**اشکال ۳: روت اشتباه.**
`ip route` را چک کنید. راه‌حل: مسیر درست.

**اشکال ۴: نبود ابزار.**
بعضی ابزارها (telnet, traceroute) باید نصب شوند. راه‌حل: نصب.

**اشکال ۵: ICMP مسدود.**
ping کار نمی‌کند ولی سرویس کار می‌کند. راه‌حل: تست با nc/curl.

**اشکال ۶: MTU اشتباه.**
بسته‌های بزرگ drop می‌شوند. راه‌حل: `ping -M do -s`.

### نکات مهم (Tips & Tricks)

- **`ping -c 4 host` فقط ۴ بسته.**
- **`mtr` بهترین ابزار برای تشخیص نقطهٔ قطع است.**
- **`ss -tlnp` پورت باز و برنامه را نشان می‌دهد.**
- **`curl -v` جزئیات اتصال HTTP را می‌دهد.**
- **`tcpdump -i eth0 port 80` تحلیل بسته.**
- **`dig +short example.com` پاسخ سریع DNS.**
- **`nc -zv host port` تست پورت.**
- **`ip -s link` خطاها و dropها را نشان می‌دهد.**
- **`mtr --report` خروجی متنی.**
- **`ss -s` خلاصهٔ آماری اتصالات.**
- **`traceroute -T` با TCP (عبور از فایروال).**
- **`nmap -sn 192.168.1.0/24` اسکن شبکه.**

### خلاصه فصل

عیب‌یابی شبکه را لایه‌ای انجام دهید: فیزیکی → IP → routing → DNS → سرویس. `ping`, `traceroute`, `mtr` دسترسی و مسیر را تست می‌کنند؛ `ss` پورت‌ها را؛ `dig`/`nslookup` DNS را؛ `curl`/`wget` HTTP را. اگر IP کار کند ولی نام نه، مشکل DNS است. `mtr` بهترین ابزار تشخیص است.

---

## فصل ۵۱: پیکربندی DNS سمت کلاینت

### مقدمه

**DNS** نام‌های دامنه را به آدرس IP ترجمه می‌کند. تصور کنید در مرورگر `google.com` می‌زنید — DNS آن را به IP تبدیل می‌کند. کلاینت باید بداند از کدام سرور DNS بپرسد.

در این فصل پیکربندی و عیب‌یابی DNS سمت کلاینت را یاد می‌گیرید.

### مفاهیم اساسی

**فایل‌های کلیدی:**

| فایل | کار |
|---|---|
| `/etc/resolv.conf` | سرورهای DNS |
| `/etc/hosts` | نگاشت محلی نام به IP |
| `/etc/nsswitch.conf` | ترتیب جستجو |
| `/etc/hostname` | نام میزبان |

**ساختار `/etc/resolv.conf`:**

```
nameserver 8.8.8.8
nameserver 1.1.1.1
search example.com
options timeout:2 attempts:3
```

**ترتیب در `nsswitch.conf`:** معمولاً `files dns` یعنی اول `/etc/hosts` بعد DNS.

**انواع رکورد:**

| رکورد | معنا |
|---|---|
| A | IPv4 |
| AAAA | IPv6 |
| MX | سرور ایمیل |
| NS | نام‌سرور |
| CNAME | نام مستعار |
| PTR | reverse (IP→نام) |
| TXT | متن (SPF و غیره) |
| SOA | اطلاعات منطقه |

### مثال‌های عملی

**پیکربندی:**

```bash
# /etc/resolv.conf
nameserver 8.8.8.8
nameserver 1.1.1.1
search example.com

# /etc/hosts
127.0.0.1   localhost
::1         localhost
192.168.1.50  server1 server1.local
```

**تست DNS:**

```bash
# dig
dig example.com
dig +short example.com
dig @8.8.8.8 example.com     # سرور مشخص
dig -x 8.8.8.8                # reverse lookup
dig example.com MX
dig example.com NS
dig +trace example.com        # کل مسیر
dig +dnssec example.com

# nslookup
nslookup example.com
nslookup example.com 8.8.8.8

# host
host example.com
host -t MX example.com

# getent (از طریق nsswitch)
getent hosts example.com
```

**مدیریت با systemd-resolved:**

```bash
resolvectl status
resolvectl query example.com
resolvectl statistics
systemctl status systemd-resolved
```

### مقایسه و الگوهای مختلف

**ابزارهای DNS:**

| ابزار | ویژگی |
|---|---|
| `dig` | قدرتمند، استاندارد |
| `nslookup` | ساده، قدیمی |
| `host` | سریع، خلاصه |
| `getent` | از nsswitch استفاده می‌کند |
| `resolvectl` | systemd-resolved |

**روش‌های مدیریت DNS:**

| روش | پایداری |
|---|---|
| ویرایش مستقیم `resolv.conf` | ممکن است بازنویسی شود |
| NetworkManager | پایدار |
| Netplan | پایدار |
| systemd-resolved | پایدار |
| `chattr +i` | قفل فایل |

**ترتیب resolve:**

```
/etc/hosts → /etc/resolv.conf → DNS servers
```

### اشکال شایع

**اشکال ۱: ویرایش دستی `resolv.conf`.**
ممکن است با NetworkManager/Netplan بازنویسی شود. راه‌حل: از ابزار پایدار.

**اشکال ۲: نبود search domain.**
نام‌های کوتاه resolve نمی‌شوند. راه‌حل: `search` اضافه کنید.

**اشکال ۳: ترتیب hosts و DNS.**
`/etc/hosts` اولویت دارد. راه‌حل: آگاه باشید.

**اشکال ۴: کش DNS.**
تغییرات فوری اعمال نمی‌شوند. راه‌حل: `resolvectl flush-caches`.

**اشکال ۵: سرور DNS اشتباه.**
نام resolve نمی‌شود. راه‌حل: سرور معتبر.

**اشکال ۶: IPv6 در اولویت.**
ممکن است پاسخ کند باشد. راه‌حل: پیکربندی.

### نکات مهم (Tips & Tricks)

- **`dig +trace` کل مسیر resolve را نشان می‌دهد.**
- **`resolvectl status` (systemd-resolved) وضعیت DNS را نشان می‌دهد.**
- **برای پایدار کردن `resolv.conf` از `chattr +i` یا systemd-resolved استفاده کنید.**
- **`dig +short` خروجی تمیز.**
- **`getent hosts` ترتیب واقعی resolve را نشان می‌دهد.**
- **`nslookup` با سرور مشخص: `nslookup name server`.**
- **`host -t TYPE` نوع رکورد مشخص.**
- **`dig -x IP` reverse lookup.**
- **`/etc/hosts` برای نگاشت محلی سریع.**
- **`resolvectl flush-caches` کش را پاک می‌کند.**
- **`systemd-resolve --status` (قدیمی).**
- **`dnsmasq` به‌عنوان DNS محلی/کش.**

### خلاصه فصل

DNS نام را به IP ترجمه می‌کند. `/etc/resolv.conf` سرورها، `/etc/hosts` نگاشت محلی و `/etc/nsswitch.conf` ترتیب جستجو را تعیین می‌کند. `dig`, `nslookup`, `host` برای تست و `getent` برای مسیر واقعی به‌کار می‌روند. ویرایش دستی `resolv.conf` ممکن است پایدار نباشد؛ از NetworkManager/Netplan/systemd-resolved استفاده کنید. `dig +trace` مسیر کامل را نشان می‌دهد.

---

## منابع و مراجع

- man pages: `man ip`, `man ss`, `man nmcli`, `man dig`, `man resolv.conf`, `man ipcalc`
- Netplan Documentation: https://netplan.io
- NetworkManager: https://networkmanager.dev
- RFC 1034/1035 (DNS): https://www.rfc-editor.org

**بعدی:** [فایل ۱۴ — مبانی امنیت](14-Security-Essentials.md)
