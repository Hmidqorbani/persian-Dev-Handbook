# فایل ۲۰ — پیکربندی شبکه

**مدت یادگیری**: ۶ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: فایل ۱۳ (مبانی شبکه)

---

## فهرست محتوا

- [فصل ۷۱: پیکربندی پایهٔ شبکه](#فصل-۷۱-پیکربندی-پایهٔ-شبکه)
- [فصل ۷۲: پیکربندی پیشرفتهٔ شبکه](#فصل-۷۲-پیکربندی-پیشرفتهٔ-شبکه)
- [فصل ۷۳: عیب‌یابی مشکلات شبکه](#فصل-۷۳-عیبیابی-مشکلات-شبکه)

---

## فصل ۷۱: پیکربندی پایهٔ شبکه

### مقدمه

در LPIC-1 مبانی شبکه را دیدیم. در LPIC-2 پیکربندی پیشرفته، مسیریابی، پل‌سازی و ابزارهای حرفه‌ای را می‌آموزیم.

تصور کنید می‌خواهید یک سرور را به‌عنوان روتر یا gateway بین دو شبکه پیکربندی کنید. این فصل پایه‌های لازم را می‌دهد.

### مفاهیم اساسی

**مفاهیم کلیدی:**

- **Interface (رابط):** eth0, ens33, enp0s3.
- **Routing table:** جدول مسیرها.
- **Netmask/CIDR:** محدودهٔ شبکه.
- **MTU:** حداکثر اندازهٔ بسته (معمولاً ۱۵۰۰).
- **Link:** وضعیت فیزیکی.

**نام‌گذاری مدرن رابط‌ها:**

| نام | معنا |
|---|---|
| `eth0` | نام کلاسیک |
| `ens33` | بر اساس slot |
| `enp0s3` | بر اساس PCI |
| `eno1` | onboard |
| `wlan0` | بی‌سیم |

**فایل‌های پیکربندی:** بسته به توزیع متفاوت است (فایل ۱۳).

### مثال‌های عملی

**پیکربندی با `ip`:**

```bash
# افزودن آدرس (موقت)
sudo ip addr add 192.168.1.10/24 dev eth0

# بالا بردن رابط
sudo ip link set eth0 up
sudo ip link set eth0 down

# تغییر MAC
sudo ip link set eth0 address 00:11:22:33:44:55

# تغییر MTU
sudo ip link set eth0 mtu 9000

# افزودن مسیر
sudo ip route add 10.0.0.0/8 via 192.168.1.1
sudo ip route add 10.0.0.0/8 dev eth0

# مسیر پیش‌فرض
sudo ip route add default via 192.168.1.1

# حذف
sudo ip route del 10.0.0.0/8

# دیدن
ip addr show
ip route show
ip route get 8.8.8.8
```

**پیکربندی دائمی (nmcli):**

```bash
# نمایش اتصالات
nmcli con show
nmcli dev status

# تنظیم استاتیک
sudo nmcli con mod "eth0" \
  ipv4.addresses 192.168.1.10/24 \
  ipv4.gateway 192.168.1.1 \
  ipv4.dns "8.8.8.8" \
  ipv4.method manual

sudo nmcli con up "eth0"

# DHCP
sudo nmcli con mod "eth0" ipv4.method auto
```

**پیکربندی نام‌گذاری:**

```bash
# نام میزبان
hostnamectl set-hostname server1
hostnamectl status

# /etc/hosts
echo "192.168.1.10 server1 server1.local" | sudo tee -a /etc/hosts
```

### مقایسه و الگوهای مختلف

**ابزارها:**

| ابزار | سطح |
|---|---|
| `ip` | مدرن، توصیه‌شده |
| `ifconfig`/`route` | قدیمی (net-tools) |
| `nmcli`/`nmtui` | پایدار |
| `netplan` | Ubuntu |
| `systemd-networkd` | سبک |

**پیکربندی موقت در برابر پایدار:**

| نوع | دستور | بقا |
|---|---|---|
| موقت | `ip addr add` | تا reboot |
| پایدار | `nmcli`/netplan | دائمی |

**انواع آدرس‌دهی:**

| نوع | توضیح |
|---|---|
| Static | ثابت |
| DHCP | خودکار |
| SLAAC | IPv6 خودکار |
| Link-local | خودکار (169.254) |

### اشکال شایع

**اشکال ۱: تداخل ip و nmcli.**
استفادهٔ همزمان می‌تواند گیج‌کننده باشد. راه‌حل: یکی را انتخاب کنید.

**اشکال ۲: MTU اشتباه.**
برای VPN/tunnel مشکل‌ساز است. راه‌حل: MTU درست.

**اشکال ۳: نبود gateway.**
خروج به اینترنت نیست. راه‌حل: gateway درست.

**اشکال ۴: اشتباه در CIDR.**
آدرس/ماسک اشتباه → عدم دسترسی. راه‌حل: محاسبه درست.

**اشکال ۵: نام رابط اشتباه.**
دستور اعمال نمی‌شود. راه‌حل: `ip a` برای نام درست.

**اشکال ۶: فراموش کردن up.**
رابط خاموش است. راه‌حل: `ip link set up`.

### نکات مهم (Tips & Tricks)

- **`ip -c` خروجی رنگی می‌دهد.**
- **`ip route get 8.8.8.8` مسیر به یک مقصد خاص را نشان می‌دهد.**
- **`nmcli` با Tab خودکارسازی می‌شود.**
- **برای تغییر سریع، `ip`؛ برای پایدار، `nmcli`/netplan.**
- **`ethtool eth0` سرعت و duplex را نشان می‌دهد.**
- **`ip -br a` خلاصهٔ رابط‌ها.**
- **`ip link set eth0 mtu 9000` jumbo frame (اگر سوییچ پشتیبانی کند).**
- **`ip neigh` ARP را نشان می‌دهد.**
- **`hostnamectl` نام و اطلاعات سیستم.**
- **`/etc/hosts` برای نگاشت محلی.**
- **`ip addr flush dev eth0` آدرس‌ها را پاک می‌کند.**
- **`ip rule` policy routing.**

### خلاصه فصل

پیکربندی پایهٔ شبکه با `ip` (موقت) و `nmcli`/netplan (پایدار) انجام می‌شود. `ip addr`, `ip route`, `ip link` ابزارهای اصلی‌اند. برای پایدار ماندن از NetworkManager/netplan استفاده کنید. `ip route get` مسیر به مقصد را نشان می‌دهد و `ethtool` جزئیات رابط را. نام‌گذاری مدرن رابط‌ها (enp0s3) بر اساس موقعیت سخت‌افزاری است.

---

## فصل ۷۲: پیکربندی پیشرفتهٔ شبکه

### مقدمه

در این فصل مباحث پیشرفته: مسیریابی، NAT، پل‌سازی (bridging)، bonding و VLAN.

تصور کنید می‌خواهید یک سرور را به‌عنوان gateway بین اینترنت و شبکهٔ داخلی پیکربندی کنید — NAT و forwarding لازم است.

### مفاهیم اساسی

**مفاهیم پیشرفته:**

- **Routing:** هدایت بسته‌ها بین شبکه‌ها.
- **NAT:** ترجمهٔ آدرس شبکه.
- **IP Forwarding:** عبور بسته‌ها از سیستم.
- **Bonding/Teaming:** ترکیب چند رابط.
- **VLAN:** شبکهٔ مجازی.
- **Bridge:** اتصال لایه ۲.

**انواع NAT:**

| نوع | کاربرد |
|---|---|
| SNAT | خروج (source) |
| DNAT | ورود/port forward |
| MASQUERADE | IP پویا |

### مثال‌های عملی

**فعال‌سازی IP forwarding:**

```bash
# موقت
sudo sysctl -w net.ipv4.ip_forward=1

# دائمی
echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/99-forward.conf
sudo sysctl -p
```

**NAT با iptables:**

```bash
# MASQUERADE (خروج اینترنت)
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# FORWARD
sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT

# ذخیره
sudo iptables-save > /etc/iptables/rules.v4
sudo iptables-restore < /etc/iptables/rules.v4
```

**Port Forwarding (DNAT):**

```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 \
  -j DNAT --to-destination 192.168.1.20:80
```

**Bonding (ترکیب رابط‌ها):**

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  ethernets:
    eth0: {}
    eth1: {}
  bonds:
    bond0:
      interfaces: [eth0, eth1]
      parameters:
        mode: active-backup
        primary: eth0
      addresses: [192.168.1.10/24]
```

**VLAN:**

```bash
# ساخت VLAN
sudo ip link add link eth0 name eth0.10 type vlan id 10
sudo ip addr add 192.168.10.1/24 dev eth0.10
sudo ip link set eth0.10 up

# دیدن
ip -d link show eth0.10
```

**Bridge:**

```bash
# ساخت bridge
sudo ip link add name br0 type bridge
sudo ip link set eth0 master br0
sudo ip link set br0 up

# بررسی
bridge link
brctl show       # قدیمی
```

### مقایسه و الگوهای مختلف

**حالت‌های bonding:**

| حالت | توضیح | نیاز به سوییچ |
|---|---|---|
| `balance-rr` | چرخشی | LACP |
| `active-backup` | یکی فعال، یکی پشتیبان | خیر |
| `802.3ad` | LACP | بله |
| `balance-xor` | بر اساس هش | LACP |
| `broadcast` | همه | — |

**ابزارهای NAT:**

| ابزار | کاربرد |
|---|---|
| `iptables` | کلاسیک |
| `nftables` | مدرن |
| `firewalld` | ساده |

**Bridge در برابر Bond:**

| ویژگی | Bridge | Bond |
|---|---|---|
| هدف | اتصال لایه ۲ | افزونگی/سرعت |
| کاربرد | مجازی‌سازی | سرور |
| لایه | ۲ | ۲ |

### اشکال شایع

**اشکال ۱: فراموش کردن IP forwarding.**
NAT کار نمی‌کند. راه‌حل: فعال کنید.

**اشکال ۲: bonding بدون پشتیبانی سوییچ.**
LACP نیاز به پیکربندی سوییچ دارد. راه‌حل: active-backup یا سوییچ.

**اشکال ۳: rules ذخیره‌نشدهٔ iptables.**
بعد از reboot پاک می‌شود. راه‌حل: `iptables-save`.

**اشکال ۴: اشتباه در VLAN tagging.**
سوییچ باید trunk باشد. راه‌حل: پیکربندی سوییچ.

**اشکال ۵: bridge بدون STP.**
حلقه ایجاد می‌شود. راه‌حل: STP یا طراحی درست.

**اشکال ۶: FORWARD پیش‌فرض DROP.**
بسته‌ها عبور نمی‌کنند. راه‌حل: قانون FORWARD.

### نکات مهم (Tips & Tricks)

- **برای NAT، هم forwarding و هم MASQUERADE لازم است.**
- **`iptables-save`/`iptables-restore` برای پایداری.**
- **`nftables` جایگزین مدرن iptables است.**
- **برای bridge، `brctl` (قدیمی) یا `ip link` (جدید).**
- **bonding را با `cat /proc/net/bonding/bond0` بررسی کنید.**
- **در VM، شبکهٔ Bridged برای تست مناسب است.**
- **`ip -d link` جزئیات (VLAN/bridge) را نشان می‌دهد.**
- **`conntrack` اتصالات را ردیابی می‌کند.**
- **`firewalld` با zone کار می‌کند.**
- **`nft list ruleset` قوانین nftables.**
- **`ip link add` برای ساخت رابط مجازی.**
- **`vconfig`/`ip link` برای VLAN.**

### خلاصه فصل

پیکربندی پیشرفته شامل IP forwarding، NAT (iptables MASQUERADE)، bonding (ترکیب رابط‌ها)، VLAN و bridge است. برای NAT هم forwarding و هم MASQUERADE لازم است. حالت `active-backup` برای bonding ساده و `802.3ad` برای LACP است. قوانین iptables را ذخیره کنید. `nftables` جایگزین مدرن است.

---

## فصل ۷۳: عیب‌یابی مشکلات شبکه

### مقدمه

عیب‌یابی شبکه در سطح LPIC-2 عمیق‌تر است: تحلیل بسته، بررسی مسیر، تشخیص bottleneck و مشکلات پیچیده.

تصور کنید کاربر می‌گوید «شبکه کند است» — باید با ابزارهای پیشرفته گلوگاه را پیدا کنید.

### مفاهیم اساسی

**رویکرد سیستماتیک:**

```
۱. لایهٔ فیزیکی (کابل، رابط)
۲. لایهٔ داده (MAC, ARP)
۳. لایهٔ شبکه (IP, routing)
۴. لایهٔ انتقال (پورت، TCP/UDP)
۵. لایهٔ کاربرد (DNS, HTTP)
```

**ابزارها:** ping, traceroute, mtr, tcpdump, ss, dig, curl.

**جدول تشخیص:**

| علامت | لایه | ابزار |
|---|---|---|
| no link | 1 | `ethtool`, `ip link` |
| no ARP | 2 | `ip neigh` |
| no ping | 3 | `ping`, `ip route` |
| port closed | 4 | `ss`, `nc` |
| DNS fail | 7 | `dig` |

### مثال‌های عملی

**تحلیل بسته با tcpdump:**

```bash
# همهٔ بسته‌ها روی eth0
sudo tcpdump -i eth0

# فقط پورت ۸۰
sudo tcpdump -i eth0 port 80

# فقط یک host
sudo tcpdump -i eth0 host 192.168.1.10

# بین دو host
sudo tcpdump -i eth0 src 192.168.1.10 and dst 8.8.8.8

# ذخیره در فایل
sudo tcpdump -i eth0 -w capture.pcap

# خواندن
sudo tcpdump -r capture.pcap

# نمایش ASCII
sudo tcpdump -i eth0 -A port 80

# محدود کردن
sudo tcpdump -i eth0 -c 100
```

**بررسی مسیر:**

```bash
mtr google.com         # ترکیبی و زنده
mtr -r -c 10 google.com  # report
traceroute -T google.com   # TCP traceroute
tracepath google.com
```

**بررسی اتصالات:**

```bash
ss -tunap              # همه
ss -tlnp               # پورت‌های شنود
ss -s                  # خلاصهٔ آماری
ss -t state established
ss -tn dst 8.8.8.8
```

**بررسی ARP:**

```bash
ip neigh
arp -a
ip neigh show dev eth0
```

**بررسی رابط:**

```bash
ip -s link
ethtool eth0
ethtool -S eth0        # آمار
```

### مقایسه و الگوهای مختلف

**ابزارهای تحلیل:**

| ابزار | لایه | کاربرد |
|---|---|---|
| `ping` | 3 | دسترسی |
| `traceroute` | 3 | مسیر |
| `mtr` | 3 | مسیر زنده |
| `tcpdump` | همه | تحلیل بسته |
| `wireshark` | همه | تحلیل گرافیکی |
| `ss` | 4 | اتصالات |
| `nmap` | 4 | اسکن |

**mtr در برابر traceroute:**

| ویژگی | mtr | traceroute |
|---|---|---|
| زنده | بله | خیر |
| آمار | بله | خیر |
| ترکیبی | ping+trace | trace |

**تشخیص bottleneck:**

| ابزار | چه چیزی |
|---|---|
| `ip -s link` | خطا/drop |
| `ss -s` | اتصالات |
| `iftop` | پهنای باند |
| `nethogs` | پهنای باند هر فرآیند |

### اشکال شایع

**اشکال ۱: فراموش کردن فایروال محلی.**
`ufw status` را چک کنید. راه‌حل: بررسی فایروال.

**اشکال ۲: اشتباه MTU.**
بسته‌های بزرگ drop می‌شوند. راه‌حل: `ping -M do -s`.

**اشکال ۳: DNS کند.**
`dig` زمان پاسخ را نشان می‌دهد. راه‌حل: سرور DNS سریع.

**اشکال ۴: نبود ابزار.**
بعضی ابزارها باید نصب شوند. راه‌حل: نصب.

**اشکال ۵: اشتباه در tcpdump filter.**
خروجی زیاد/کم. راه‌حل: filter دقیق.

**اشکال ۶: نادیده گرفتن duplex mismatch.**
کارایی پایین. راه‌حل: `ethtool`.

### نکات مهم (Tips & Tricks)

- **`mtr` بهترین ابزار برای تشخیص نقطهٔ قطع است.**
- **`tcpdump -w file.pcap` را با Wireshark تحلیل کنید.**
- **`ss -s` خلاصهٔ سریع اتصالات.**
- **`ping -M do -s 1472 host` تست MTU.**
- **`ip -s link` خطاهای رابط (errors, drops) را نشان می‌دهد.**
- **`nmap` برای اسکن پورت از دید مهاجم.**
- **`iftop`/`nethogs` پهنای باند را نشان می‌دهند.**
- **`ethtool -S` آمار دقیق.**
- **`ss -tlnp` پورت باز و برنامه.**
- **`tcpdump -A` محتوای ASCII.**
- **`conntrack -L` اتصالات tracked.**
- **`ss -K` اتصالات را می‌کشد.**

### خلاصه فصل

عیب‌یابی شبکه لایه‌ای انجام می‌شود: فیزیکی → داده → شبکه → انتقال → کاربرد. `tcpdump` تحلیل بسته، `mtr` مسیر زنده، `ss` اتصالات و `ip neigh` ARP را بررسی می‌کند. فایروال محلی را فراموش نکنید و MTU را برای بسته‌های بزرگ چک کنید. `iftop`/`nethogs` پهنای باند را نشان می‌دهند.

---

## منابع و مراجع

- man pages: `man ip`, `man tcpdump`, `man mtr`, `man ss`, `man ethtool`, `man iptables`
- netplan Documentation: https://netplan.io
- nftables Wiki: https://wiki.nftables.org
- NetworkManager: https://networkmanager.dev

**بعدی:** [فایل ۲۱ — نگهداری سیستم](21-System-Maintenance.md)
