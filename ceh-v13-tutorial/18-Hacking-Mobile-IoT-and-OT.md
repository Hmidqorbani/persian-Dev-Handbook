# هک موبایل، IoT و OT

**مدت یادگیری**: ۵ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: فایل ۰۳ (آزمایشگاه) و ۱۵ (وب)

---

## فهرست محتوا

- [فصل ۱: امنیت موبایل (Android و iOS)](#فصل-۱-امنیت-موبایل-android-و-ios)
- [فصل ۲: بدافزار موبایل و MDM](#فصل-۲-بدافزار-موبایل-و-mdm)
- [فصل ۳: مقدمه IoT](#فصل-۳-مقدمه-iot)
- [فصل ۴: حملات IoT و AI](#فصل-۴-حملات-iot-و-ai)
- [فصل ۵: امنیت OT/SCADA/ICS](#فصل-۵-امنیت-otscadaics)
- [فصل ۶: آزمایشگاه و اقدامات متقابل](#فصل-۶-آزمایشگاه-و-اقدامات-متقابل)

---

## فصل ۱: امنیت موبایل (Android و iOS)

### مقدمه

گوشی‌های هوشمند حاوی حساس‌ترین داده‌های ما هستند. امنیت موبایل بخش مهمی از CEH است.

### مفاهیم اساسی

**معماری Android:**
- مبتنی بر Linux Kernel.
- **Sandbox:** هر اپ در محیط جداگانه.
- **Permissions:** دسترسی‌های اپ.
- **APK:** بسته نصب اپ.
- **Root:** دسترسی سطح بالا (خطر امنیتی).
- **Google Play Protect.**

**معماری iOS:**
- **Sandbox** سخت‌گیرانه.
- **Secure Enclave:** ذخیره امن کلیدها.
- **Code Signing** اجباری.
- **Jailbreak:** دسترسی سطح بالا.
- **App Store** بررسی‌شده.

**تهدیدات رایج:**
- اپ‌های مخرب
- Rooting/Jailbreaking
- شبکه‌های Wi-Fi ناامن
- مهندسی اجتماعی (Smishing)

### مثال عملی

```bash
# === تحلیل APK (Android) ===
# ابزارها: apktool, jadx, MobSF, Frida

# decompile با apktool
apktool d app.apk

# تحلیل با jadx (خواندن کد جاوا)
jadx-gui app.apk

# تحلیل ایستا با MobSF
# docker run -it -p 8000:8000 opensecurity/mobile-security-framework-mobsf
# → مرورگر: http://localhost:8000

# === تست با Frida (Instrumentation) ===
frida-ps -U                     # لیست پروسه‌های گوشی
frida -U -f com.target.app      # اتصال به اپ

# === بررسی امنیت ===
# - Hardcoded secrets در APK
# - Insecure storage (SharedPreferences)
# - عدم Certificate Pinning
# - دسترسی‌های اضافی
```

### اشکال شایع

- **مشکل:** APK با obfuscation → از ابزارهای Deobfuscation یا Dynamic Analysis.
- **مشکل:** اپ Root را تشخیص می‌دهد → با Magisk مخفی کنید.

### نکات مهم (Tips & Tricks)

- **MobSF** جامع‌ترین ابزار تحلیل موبایل است.
- **Frida** برای Dynamic Analysis عالی است.
- **Certificate Pinning** تحلیل ترافیک را سخت می‌کند.
- همیشه APK را از منبع رسمی بگیرید.

### خلاصه فصل

- Android (Sandbox/Permissions) و iOS (Secure Enclave/Code Signing).
- ابزارها: MobSF، jadx، Frida، apktool.
- Root/Jailbreak امنیت را کاهش می‌دهد.

---

## فصل ۲: بدافزار موبایل و MDM

### مقدمه

بدافزار موبایل می‌تواند پیام‌ها، موقعیت و بانک کاربر را هدف بگیرد.

### مفاهیم اساسی

**انواع بدافزار موبایل:**
- **Trojan:** اپ ظاهراً سالم (مثل بازی) با کد مخرب.
- **Spyware:** جاسوسی (Pegasus).
- **Banking Trojan:** سرقت اطلاعات بانکی.
- **Ransomware:** رمزنگاری گوشی.
- **Adware:** تبلیغات مزاحم.
- **SMS Trojan:** ارسال پیامک پرمخارج.

**کانال‌های نفوذ:** اپ‌های غیررسمی، Smishing، Wi-Fi عمومی، اپ‌های sideload.

**MDM (Mobile Device Management):** مدیریت و امنیت دستگاه‌های سازمانی.
- Enrollment، Policy Enforcement، Remote Wipe، Compliance.

### مثال عملی

```bash
# === تحلیل رفتار بدافزار ===
# در محیط ایزوله (emulator)

# اجرای Android emulator
emulator -avd test_device

# نصب APK مشکوک
adb install suspicious.apk

# مانیتور ترافیک
# با Burp/mitmproxy روی emulator

# بررسی مجوزها
adb shell dumpsys package suspicious.package | grep permission

# لاگ‌های سیستم
adb logcat
```

### اشکال شایع

- **مشکل:** بدافزار در emulator اجرا نمی‌شود (Anti-VM) → از دستگاه واقعی ایزوله استفاده کنید.
- **مشکل:** SSL Pinning مانع تحلیل است → با Frida آن را غیرفعال کنید.

### نکات مهم (Tips & Tricks)

- **Pegasus** نمونه پیشرفته جاسوس‌افزار (Zero-click) است.
- **MDM** ابزار کلیدی سازمان‌هاست.
- **BYOD** (Bring Your Own Device) چالش امنیتی بزرگ است.
- **sideload** کردن اپ خطرناک است.

### خلاصه فصل

- انواع: Trojan، Spyware، Banking، Ransomware.
- کانال نفوذ: اپ غیررسمی، Smishing، Wi-Fi.
- MDM مدیریت دستگاه‌های سازمانی.

---

## فصل ۳: مقدمه IoT

### مقدمه

**IoT (اینترنت اشیا)** به میلیاردها دستگاه متصل اشاره دارد: دوربین، ترموستات، حسگر، لوازم خانگی. امنیت ضعیف آن‌ها خطر بزرگی است.

> **مثال واقعی:** باتنت Mirai با آلوده‌کردن دوربین‌ها و روترهای IoT، حمله DDoS عظیمی انجام داد.

### مفاهیم اساسی

**معماری IoT:**
```
دستگاه‌های حسگر/عملگر
        │
    شبکه (Wi-Fi/Zigbee/LoRa)
        │
    Gateway / Hub
        │
    پلتفرم ابری
        │
    اپلیکیشن کاربر
```

**پروتکل‌های IoT:** MQTT، CoAP، Zigbee، Z-Wave، BLE، LoRaWAN.

**چالش‌های امنیتی IoT:**
- رمز پیش‌فرض
- Firmware قدیمی (بدون به‌روزرسانی)
- ارتباط بدون رمز
- منابع محدود (عدم پشتیبانی از رمزنگاری قوی)
- مدیریت متمرکز ضعیف

### مثال عملی

```bash
# === جستجوی دستگاه‌های IoT (Shodan) ===
# در Shodan: "webcam", "router", "port:554" (RTSP)

# === تحلیل Firmware ===
# دانلود firmware و استخراج
binwalk -e firmware.bin
# جستجوی رمز و کلید
strings firmware.bin | grep -i "password\|key\|admin"

# === تست پروتکل MQTT ===
mosquitto_sub -h 192.168.56.101 -t "#" -v
# اگر بدون احراز هویت باشد → همه پیام‌ها لو می‌رود

# === اسکن دستگاه IoT ===
nmap -sV -p- 192.168.56.101
```

### اشکال شایع

- **مشکل:** Firmware رمزنگاری‌شده است → سخت‌تر، اما کلید اغلب در دستگاه است.
- **مشکل:** دستگاه به‌روزرسانی نمی‌شود → چالش اصلی IoT؛ Segment‌بندی شبکه لازم است.

### نکات مهم (Tips & Tricks)

- **Shodan** موتور جستجوی دستگاه‌های IoT است.
- **MQTT بدون احراز هویت** بسیار رایج است.
- **binwalk** ابزار اصلی تحلیل Firmware.
- IoT را در **شبکه جدا** (VLAN) قرار دهید.

### خلاصه فصل

- IoT = میلیاردها دستگاه کم‌امنیت.
- پروتکل‌ها: MQTT، CoAP، Zigbee.
- رمز پیش‌فرض و Firmware قدیمی چالش‌های اصلی.

---

## فصل ۴: حملات IoT و AI

### مقدمه

IoT سطح حمله عظیمی ایجاد کرده و AI هم حمله و هم دفاع را تغییر داده است.

### مفاهیم اساسی

**حملات IoT:**
- **Default Credentials:** رمز پیش‌فرض (admin/admin).
- **Firmware Analysis:** استخراج رمز از Firmware.
- **Botnet Recruitment:** جذب به باتنت (Mirai).
- **Side-Channel:** تحلیل مصرف برق/صدا.
- **Physical:** دسترسی فیزیکی (JTAG، UART).
- **Protocol Attacks:** MQTT، CoAP بدون امنیت.
- **Man-in-the-Middle:** ارتباط بدون رمز.

**AI در IoT:**
- **سمت مهاجم:** کشف خودکار دستگاه آسیب‌پذیر، بهینه‌سازی باتنت.
- **سمت مدافع:** تشخیص ناهنجاری ترافیک IoT، رفتار دستگاه.

### مثال عملی

```bash
# === حمله رمز پیش‌فرض ===
hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.56.101 http-get /
# یا با ابزارهای مخصوص IoT

# === استخراج Firmware با binwalk ===
binwalk -e firmware.bin
cd _firmware.bin.extracted/
find . -name "*.conf" -exec grep -i "pass" {} \;

# === UART/JTAG (دسترسی فیزیکی) ===
# با ابزار سخت‌افزاری (Bus Pirate, JTAGulator)
# → دسترسی به console دستگاه

# === تحلیل ترافیک IoT ===
tcpdump -i eth0 -w iot.pcap
wireshark iot.pcap
```

### اشکال شایع

- **مشکل:** دستگاه از شبکه خارج است → دسترسی فیزیکی لازم است.
- **مشکل:** ترافیک رمزنگاری‌شده → کلید در Firmware جستجو کنید.

### نکات مهم (Tips & Tricks)

- **رمز پیش‌فرض** شایع‌ترین نقطه ورود IoT است.
- **Mirai** منبع الهام باتنت‌های IoT.
- **Segment‌بندی** بهترین دفاع سازمانی.
- AI برای **کشف خودکار** دستگاه‌های آسیب‌پذیر استفاده می‌شود.

### خلاصه فصل

- حملات: Default Cred، Firmware، Botnet، Physical.
- AI هم حمله و هم دفاع را تقویت می‌کند.
- Segment‌بندی شبکه دفاع کلیدی.

---

## فصل ۵: امنیت OT/SCADA/ICS

### مقدمه

**OT (Operational Technology)**، **ICS (Industrial Control Systems)** و **SCADA** سیستم‌های کنترل زیرساخت‌های حیاتی‌اند: برق، آب، نفت، تولید. حمله به آن‌ها می‌تواند فاجعه‌بار باشد.

> **مثال واقعی:** Stuxnet (2010) به سانتریفیوژهای هسته‌ای ایران آسیب زد و نشان داد OT چقدر آسیب‌پذیر است.

### مفاهیم اساسی

**تفاوت IT و OT:**

| ویژگی | IT | OT |
|-------|-----|-----|
| اولویت | محرمانگی | دسترس‌پذیری و ایمنی |
| به‌روزرسانی | منظم | کم (ریسک توقف) |
| طول عمر | ۳-۵ سال | ۱۰-۲۰ سال |
| پروتکل | TCP/IP | Modbus، DNP3، Profinet |

**اجزای ICS:** PLC، RTU، HMI، SCADA Server، Historian.
**پروتکل‌ها:** Modbus، DNP3، IEC 61850، Profinet، EtherNet/IP.
**مناطق (Purdue Model):** از سطح فیزیکی (۰) تا اینترنت (۵).

### مثال عملی

```bash
# === اسکن شبکه OT (با احتیاط!) ===
# هرگز اسکن تهاجمی در OT واقعی انجام ندهید
# در محیط آزمایشگاهی:
nmap -sV -p 502,102,44818 192.168.56.101   # Modbus, S7, EtherNet/IP

# === تعامل با Modbus ===
# ابزار: modbus-cli, mbtget
mbtget -a 1 -r 0 -n 10 -w 192.168.56.101
# اگر بدون احراز هویت باشد → کنترل PLC ممکن است

# === ابزارهای تحلیل OT ===
# - Wireshark با dissectorهای OT
# - GRASSMARLIN (نقشه‌برداری شبکه OT)
# - Shodan (جستجوی دستگاه‌های OT در اینترنت)
```

### اشکال شایع

- **مشکل:** OT نمی‌تواند اسکن شود (توقف تولید) → همیشه با احتیاط و هماهنگی.
- **مشکل:** پروتکل‌های OT بدون امنیت‌اند → درست است؛ رمزنگاری در OT نادر است.

### نکات مهم (Tips & Tricks)

- **ایمنی (Safety) مقدم بر امنیت (Security)** در OT است.
- هرگز اسکن تهاجمی در OT واقعی نکنید — می‌تواند تولید را متوقف کند.
- **Air Gap** (جداسازی فیزیکی) دفاع سنتی است (اما Stuxnet آن را رد کرد).
- **Purdue Model** برای طراحی دفاع لایه‌ای استفاده می‌شود.
- **Segment‌بندی** و **مانیتورینگ Passive** دفاع‌های اصلی‌اند.

### خلاصه فصل

- OT/ICS/SCADA زیرساخت‌های حیاتی را کنترل می‌کنند.
- پروتکل‌های OT اغلب بدون امنیت‌اند.
- Stuxnet نمونه حمله پیشرفته OT است.
- ایمنی و احتیاط اولویت اول است.

---

## فصل ۶: آزمایشگاه و اقدامات متقابل

### مقدمه

یک آزمایشگاه امن برای تمرین موبایل و IoT.

### مثال عملی — آزمایشگاه

```bash
# === موبایل ===
# ۱. نصب Android emulator یا Genymotion
# ۲. دانلود یک APK آزمایشی (مثل InsecureBankv2)
# ۳. تحلیل با MobSF:
docker run -it -p 8000:8000 opensecurity/mobile-security-framework-mobsf
# ۴. تست پویا با Frida

# === IoT ===
# ۱. نصب OWASP IoTGoat (firmware آسیب‌پذیر)
# ۲. تحلیل Firmware با binwalk
binwalk -e IoTGoat.bin
# ۳. راه‌اندازی در VM و تست

# === OT ===
# ۱. نصب ابزار شبیه‌سازی (مثل Conpot)
pip install conpot
conpot --template default
# ۲. تحلیل ترافیک Modbus با Wireshark
```

### اشکال شایع

- **مشکل:** ابزار شبیه‌سازی OT پیچیده است → با Conpot شروع کنید.
- **مشکل:** Emulator کند است → منابع کافی اختصاص دهید.

### نکات مهم (Tips & Tricks)

- **InsecureBankv2** و **DIVA** اپ‌های تمرینی موبایل‌اند.
- **OWASP IoTGoat** firmware آسیب‌پذیر برای تمرین.
- **Conpot** PLC شبیه‌سازی‌شده برای تمرین OT.
- همیشه در محیط ایزوله.

### اقدامات متقابل (Countermeasures)

| حوزه | راهکار |
|------|--------|
| Mobile | MDM، App Store رسمی، رمزنگاری، Biometric |
| IoT | تغییر رمز پیش‌فرض، به‌روزرسانی Firmware، Segment‌بندی |
| IoT Protocol | MQTT با احراز هویت، TLS |
| OT/ICS | Air Gap، Segment‌بندی، مانیتورینگ Passive، Purdue Model |
| همه | آموزش، Least Privilege، مانیتورینگ |

### خلاصه فصل

- آزمایشگاه: emulator موبایل، IoTGoat، Conpot.
- ابزارهای تمرینی امن موجودند.
- دفاع: Segment‌بندی + به‌روزرسانی + احراز هویت.

---

## منابع و مراجع

- **OWASP Mobile Security:** https://owasp.org/www-project-mobile-security/
- **MobSF:** https://github.com/MobSF/Mobile-Security-Framework-MobSF
- **Frida:** https://frida.re/
- **OWASP IoT:** https://owasp.org/www-project-internet-of-things/
- **OWASP IoTGoat:** https://github.com/OWASP/IoTGoat
- **Shodan:** https://www.shodan.io/
- **Conpot:** https://github.com/mushorg/conpot
- **NIST ICS Security:** https://www.nist.gov/
- **MITRE ATT&CK for ICS:** https://collaborate.mitre.org/attackics/
