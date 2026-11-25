# فایل ۱۲ — سرویس‌های ضروری سیستم

**مدت یادگیری**: ۷ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۱۱ (وظایف مدیریتی)

---

## فهرست محتوا

- [فصل ۴۴: نگهداری زمان سیستم](#فصل-۴۴-نگهداری-زمان-سیستم)
- [فصل ۴۵: لاگ سیستم](#فصل-۴۵-لاگ-سیستم)
- [فصل ۴۶: مبانی MTA](#فصل-۴۶-مبانی-mta)
- [فصل ۴۷: مدیریت چاپگر](#فصل-۴۷-مدیریت-چاپگر)

---

## فصل ۴۴: نگهداری زمان سیستم

### مقدمه

دقت زمان در سرورها حیاتی است — برای لاگ‌ها، گواهی‌ها (certificates)، احراز هویت و همگام‌سازی. تصور کنید زمان سرور ۵ دقیقه عقب باشد و گواهی SSL منقضی به نظر برسد، یا لاگ‌های دو سرور با هم نخوانند.

در این فصل مدیریت زمان و همگام‌سازی NTP را یاد می‌گیرید.

### مفاهیم اساسی

**دو ساعت در سیستم:**

- **RTC (Real-Time Clock):** ساعت سخت‌افزاری (روی مادربورد)، با باتری کار می‌کند.
- **System Clock:** ساعت نرم‌افزاری کرنل (در حافظه).

**مفاهیم کلیدی:**

- **UTC:** زمان جهانی هماهنگ (مبنای استاندارد).
- **NTP:** پروتکل همگام‌سازی زمان.
- **chrony:** پیاده‌سازی مدرن NTP.
- **Timezone:** منطقهٔ زمانی (مثل Asia/Tehran).
- **Drift:** انحراف ساعت.

**ابزارها:** `date`, `timedatectl`, `hwclock`, `chronyc`.

**مقایسه UTC و Local:**

| مفهوم | توضیح |
|---|---|
| UTC | مبنای جهانی |
| Local | UTC + offset منطقه |
| RTC در UTC | استاندارد سرورها |
| RTC در Local | برای دسکتاپ (dual-boot) |

### مثال‌های عملی

**دیدن و تنظیم زمان:**

```bash
# نمایش زمان
date
date -u              # UTC
date +"%Y-%m-%d %H:%M:%S"
date +%s             # Unix timestamp

# timedatectl
timedatectl
timedatectl status

# تنظیم منطقهٔ زمانی
sudo timedatectl set-timezone Asia/Tehran
timedatectl list-timezones | grep Tehran

# تنظیم دستی زمان
sudo timedatectl set-time "2026-10-16 10:00:00"

# فعال‌سازی NTP
sudo timedatectl set-ntp true
```

**همگام‌سازی NTP با chrony:**

```bash
# نصب
sudo apt install chrony        # Debian/Ubuntu
sudo dnf install chrony        # RHEL/Rocky

# فعال‌سازی
sudo systemctl enable --now chronyd

# بررسی
chronyc sources
chronyc sources -v
chronyc tracking

# همگام‌سازی فوری
sudo chronyc makestep

# فهرست سرورها
chronyc activity
```

**پیکربندی `/etc/chrony/chrony.conf`:**

```
pool ntp.ubuntu.com iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony
```

**ساعت سخت‌افزاری:**

```bash
# خواندن
sudo hwclock --show

# نوشتن ساعت سیستم به سخت‌افزار
sudo hwclock --systohc

# خواندن سخت‌افزار به سیستم
sudo hwclock --hctosys

# تنظیم UTC
sudo hwclock --systohc --utc
```

### مقایسه و الگوهای مختلف

**NTP daemonها:**

| ابزار | ویژگی | مناسب |
|---|---|---|
| **chrony** | مدرن، پیش‌فرض اکثر توزیع‌ها | سرور، دسکتاپ |
| **ntpd** | کلاسیک، قدیمی‌تر | legacy |
| **systemd-timesyncd** | سبک، ساده | دسکتاپ، سبک |

**مقایسه chrony و ntpd:**

| ویژگی | chrony | ntpd |
|---|---|---|
| همگام‌سازی سریع | بله | کندتر |
| شبکهٔ متغیر | عالی | ضعیف |
| مصرف منابع | کم | بیشتر |
| پیش‌فرض | اکثر توزیع‌ها | قدیمی |

**منابع زمان:**

| منبع | توضیح |
|---|---|
| pool.ntp.org | مخزن عمومی |
| سرور داخلی | دقت بالا |
| GPS | بسیار دقیق |
| PTP | دقت نانوثانیه |

### اشکال شایع

**اشکال ۱: drift زمان.**
بدون NTP، ساعت منحرف می‌شود. راه‌حل: NTP فعال.

**اشکال ۲: اشتباه UTC/Local.**
زمان لاگ‌ها گیج‌کننده می‌شود. راه‌حل: UTC استاندارد.

**اشکال ۳: NTP در VM.**
بعضی VMها زمان را کند/تند می‌کنند. راه‌حل: guest agent.

**اشکال ۴: فایروال مسدود NTP.**
پورت UDP 123 لازم است. راه‌حل: باز کردن پورت.

**اشکال ۵: گواهی SSL و زمان.**
زمان اشتباه باعث خطای گواهی می‌شود. راه‌حل: NTP دقیق.

**اشکال ۶: دو ساعت ناهمگام.**
RTC و System Clock فرق دارند. راه‌حل: `hwclock --systohc`.

### نکات مهم (Tips & Tricks)

- **`timedatectl set-ntp true` همگام‌سازی خودکار را فعال می‌کند.**
- **`chronyc sources -v` جزئیات سرورهای NTP را نشان می‌دهد.**
- **در VMها guest agent معمولاً زمان را sync می‌کند.**
- **`date -s` نیاز به ریشه دارد.**
- **برای عیب‌یابی، `timedatectl` اولین ابزار است.**
- **`chronyc makestep` همگام‌سازی فوری.**
- **`rtcsync` در chrony ساعت سخت‌افزاری را sync می‌کند.**
- **`/etc/adjtime` تنظیمات RTC را نگه می‌دارد.**
- **برای dual-boot با ویندوز، RTC را Local بگذارید.**
- **`ntpdate` منسوخ شده؛ از chrony استفاده کنید.**
- **`timedatectl show-timesync` وضعیت همگام‌سازی.**
- **NTP پورت UDP 123 استفاده می‌کند.**

### خلاصه فصل

سیستم دو ساعت دارد: RTC (سخت‌افزاری) و System Clock (نرم‌افزاری). `timedatectl` مدیریت زمان و منطقهٔ زمانی را ساده می‌کند. `chrony` (یا ntpd/systemd-timesyncd) زمان را با NTP همگام می‌کند. `hwclock` بین دو ساعت sync می‌کند. دقت زمان برای لاگ، گواهی و احراز هویت حیاتی است. NTP پورت UDP 123 لازم دارد.

---

## فصل ۴۵: لاگ سیستم

### مقدمه

**لاگ‌ها (Logs)** ثبت رویدادهای سیستم‌اند. تصور کنید سرور هک شده یا سرویسی کرش کرده — بدون لاگ، عیب‌یابی تقریباً غیرممکن است. لاگ‌ها تاریخچهٔ همه‌چیز را نگه می‌دارند.

در این فصل سیستم لاگ‌گیری لینوکس را می‌شناسید: syslog، journald و مدیریت آن‌ها.

### مفاهیم اساسی

**دو سیستم لاگ:**

- **syslog (سنتی):** rsyslog/syslog-ng → فایل‌های متنی.
- **journald (systemd):** لاگ باینری، با `journalctl` خوانده می‌شود.

**محل لاگ‌ها:** `/var/log/`.

| فایل | محتوا | توزیع |
|---|---|---|
| `/var/log/syslog` | لاگ عمومی | Debian |
| `/var/log/messages` | لاگ عمومی | RHEL |
| `/var/log/auth.log` | احراز هویت | Debian |
| `/var/log/secure` | احراز هویت | RHEL |
| `/var/log/kern.log` | لاگ کرنل | Debian |
| `/var/log/dpkg.log` | نصب پکیج | Debian |
| `/var/log/dnf.log` | نصب پکیج | RHEL |
| `/var/log/cron` | cron | RHEL |
| `/var/log/boot.log` | بوت | — |

**اولویت‌های syslog:**

| سطح | شماره | معنا |
|---|---|---|
| emerg | 0 | اضطراری |
| alert | 1 | هشدار |
| crit | 2 | بحرانی |
| err | 3 | خطا |
| warning | 4 | هشدار |
| notice | 5 | توجه |
| info | 6 | اطلاعات |
| debug | 7 | اشکال‌زدایی |

**facilityها:** kern, user, mail, daemon, auth, cron, ...

### مثال‌های عملی

**کار با journalctl:**

```bash
# همهٔ لاگ‌ها
journalctl

# لاگ‌های این بوت
journalctl -b

# بوت قبلی
journalctl -b -1

# فهرست بوت‌ها
journalctl --list-boots

# یک سرویس
journalctl -u nginx

# زنده
journalctl -f

# بر اساس زمان
journalctl --since "1 hour ago"
journalctl --since "2026-10-16" --until "2026-10-17"
journalctl --since "10:00" --until "11:00"

# بر اساس اولویت
journalctl -p err
journalctl -p warning

# بر اساس فرآیند
journalctl _PID=1234
journalctl _COMM=sshd

# خروجی JSON
journalctl -o json-pretty
journalctl -o short-precise
```

**مدیریت فضای journal:**

```bash
# حجم
journalctl --disk-usage

# پاک‌سازی
sudo journalctl --vacuum-size=100M
sudo journalctl --vacuum-time=7d
sudo journalctl --vacuum-files=5

# پیکربندی (/etc/systemd/journald.conf)
# SystemMaxUse=500M
# MaxRetentionSec=1month
```

**پیکربندی rsyslog (`/etc/rsyslog.conf`):**

```
# ارسال لاگ auth به فایل جداگانه
auth,authpriv.*   /var/log/auth.log

# همه‌چیز به syslog
*.*               /var/log/syslog

# ارسال به سرور مرکزی
*.* @@logserver:514
```

**لاگ دستی:**

```bash
logger "پیام آزمایشی"
logger -t myscript "پیام با برچسب"
logger -p local0.info "پیام با facility"
```

### مقایسه و الگوهای مختلف

**syslog در برابر journald:**

| ویژگی | syslog | journald |
|---|---|---|
| فرمت | متنی | باینری |
| ابزار | `cat`, `grep` | `journalctl` |
| ساختار | فایل | یکپارچه |
| فراداده | محدود | غنی |
| چرخش | logrotate | خودکار |

**توصیه:** journald + rsyslog با هم کار می‌کنند (journald به rsyslog هم می‌فرستد).

**سطح‌های لاگ:**

| سطح | کاربرد |
|---|---|
| emerg/alert/crit | مشکلات جدی |
| err | خطاها |
| warning | هشدارها |
| notice/info | اطلاعات |
| debug | اشکال‌زدایی |

**چرخش لاگ:**

| ابزار | توضیح |
|---|---|
| `logrotate` | چرخش سیستمی |
| `journalctl --vacuum` | چرخش journald |
| `systemd-tmpfiles` | پاک‌سازی موقت |

### اشکال شایع

**اشکال ۱: رشد بی‌رویهٔ لاگ‌ها.**
`/var` پر می‌شود. راه‌حل: logrotate.

**اشکال ۲: نبود لاگ یک سرویس.**
شاید journald آن را نگه نمی‌دارد. راه‌حل: بررسی پیکربندی.

**اشکال ۳: دسترسی به لاگ.**
بعضی لاگ‌ها فقط برای ریشه خوانا هستند. راه‌حل: `sudo`.

**اشکال ۴: حذف دستی لاگ فعال.**
سرویس باید reload شود. راه‌حل: `systemctl reload rsyslog`.

**اشکال ۵: journald بدون persistent.**
لاگ بعد از reboot می‌رود. راه‌حل: `Storage=persistent`.

**اشکال ۶: فراموش کردن logrotate.**
لاگ‌ها بی‌نهایت رشد می‌کنند. راه‌حل: cron.

### نکات مهم (Tips & Tricks)

- **`journalctl -f -u service` بهترین راه دیباگ زنده.**
- **`journalctl -p err -b` فقط خطاهای این بوت.**
- **`/etc/logrotate.conf` چرخش لاگ را مدیریت می‌کند.**
- **`journalctl --no-pager` بدون صفحه‌بندی.**
- **`logger "message"` یک لاگ دستی می‌نویسد.**
- **`journalctl -o json-pretty` خروجی ساختاریافته.**
- **`Storage=persistent` در journald.conf لاگ را ماندگار می‌کند.**
- **`rsyslog` می‌تواند به سرور مرکزی بفرستد.**
- **`logrotate -d` حالت dry-run.**
- **`/etc/rsyslog.d/` برای پیکربندی اضافی.**
- **`journalctl -k` فقط لاگ کرنل.**
- **`lastlog`/`last` ورودها را نشان می‌دهند.**

### خلاصه فصل

لاگ‌ها در `/var/log/` (syslog) و در journal (systemd) ذخیره می‌شوند. `journalctl` ابزار اصلی خواندن journal است (با `-b`, `-u`, `-f`, `-p`, `--since`). rsyslog فایل‌های متنی می‌سازد. با `logrotate` و `journalctl --vacuum` فضا را مدیریت کنید. `Storage=persistent` لاگ را ماندگار می‌کند. لاگ‌ها پایهٔ عیب‌یابی‌اند.

---

## فصل ۴۶: مبانی MTA

### مقدمه

**MTA (Mail Transfer Agent)** نرم‌افزاری است که ایمیل را جابجا می‌کند. حتی اگر سرور شما سرور ایمیل نیست، سرویس‌های سیستمی از ایمیل برای ارسال هشدار استفاده می‌کنند (مثلاً cron نتایج را ایمیل می‌کند).

در این فصل مبانی MTA و Postfix را یاد می‌گیرید.

### مفاهیم اساسی

**اجزای سیستم ایمیل:**

| جزء | کار | نمونه |
|---|---|---|
| **MTA** | انتقال ایمیل | Postfix, Sendmail, Exim |
| **MDA** | تحویل به صندوق | Procmail, Dovecot-LDA |
| **MUA** | کلاینت کاربر | Thunderbird, mutt |

**پروتکل‌ها:** SMTP (ارسال)، POP3/IMAP (دریافت).

**نقشهٔ DNS:** MX record مشخص می‌کند ایمیل به کدام سرور برود.

**مسیر یک ایمیل:**

```
MUA → MTA فرستنده → (DNS MX) → MTA گیرنده → MDA → صندوق
```

### مثال‌های عملی

**نصب و مدیریت Postfix:**

```bash
# Debian/Ubuntu
sudo apt install postfix
# در نصب، «Internet Site» را انتخاب کنید

# RHEL/Rocky
sudo dnf install postfix

# سرویس
sudo systemctl enable --now postfix
systemctl status postfix

# صف ایمیل
mailq
postqueue -p

# پاک‌سازی صف
sudo postsuper -d ALL
sudo postsuper -d <queue_id>

# پردازش فوری صف
sudo postqueue -f
```

**پیکربندی (`/etc/postfix/main.cf`):**

```
myhostname = mail.example.com
mydomain = example.com
myorigin = $mydomain
inet_interfaces = all
inet_protocols = ipv4
mydestination = $myhostname, localhost.$mydomain, $mydomain
mynetworks = 127.0.0.0/8, 192.168.1.0/24
relay_domains = $mydestination
home_mailbox = Maildir/
```

**ارسال ایمیل:**

```bash
# با mail
echo "متن پیام" | mail -s "موضوع" user@example.com
mail -s "موضوع" user@example.com < message.txt

# با sendmail
sendmail user@example.com < message.txt

# با mailx (پیوست)
echo "body" | mailx -s "subject" -a file.pdf user@example.com

# بررسی ارسال
tail -f /var/log/mail.log
```

**بررسی پیکربندی:**

```bash
# بررسی syntax
sudo postfix check

# تنظیمات غیرپیش‌فرض
postconf -n

# همهٔ تنظیمات
postconf -d
```

### مقایسه و الگوهای مختلف

**MTAها:**

| MTA | ویژگی | مناسب |
|---|---|---|
| **Postfix** | محبوب، امن، آسان | عمومی |
| **Sendmail** | قدیمی، پیچیده | legacy |
| **Exim** | پیش‌فرض Debian | انعطاف |
| **OpenSMTPD** | ساده، امن | ساده |

**ابزارهای خط فرمان:**

| ابزار | کار |
|---|---|
| `mail` | ارسال/خواندن ساده |
| `mailx` | نسخهٔ پیشرفته |
| `mutt` | کلاینت ترمینالی کامل |
| `sendmail` | ارسال مستقیم |
| `mailq` | نمایش صف |

**حالت‌های Postfix:**

| حالت | توضیح |
|---|---|
| Internet Site | ایمیل مستقیم |
| Internet with smarthost | از طریق relay |
| Satellite | فقط ارسال محلی |
| Local only | فقط محلی |

### اشکال شایع

**اشکال ۱: MTA پیکربندی‌نشده.**
ایمیل‌های سیستمی در صف می‌مانند. راه‌حل: پیکربندی.

**اشکال ۲: Open Relay.**
پیکربندی اشتباه باعث اسپم می‌شود. راه‌حل: `mynetworks` محدود.

**اشکال ۳: نبود rDNS/SPF.**
ایمیل به اسپم می‌رود. راه‌حل: DNS درست.

**اشکال ۴: صف پر.**
با `postsuper` پاک کنید. راه‌حل: مدیریت صف.

**اشکال ۵: فایروال پورت ۲۵.**
ارسال شکست می‌خورد. راه‌حل: باز کردن.

**اشکال ۶: اشتباه در mydestination.**
ایمیل برای دامنهٔ خودی رد می‌شود. راه‌حل: پیکربندی درست.

### نکات مهم (Tips & Tricks)

- **برای فقط ارسال محلی، `inet_interfaces = loopback-only`.**
- **`postconf -n` تنظیمات غیرپیش‌فرض را نشان می‌دهد.**
- **`postfix check` پیکربندی را بررسی می‌کند.**
- **لاگ در `/var/log/mail.log` (Debian) یا `/var/log/maillog` (RHEL).**
- **`mailq` وضعیت صف را نشان می‌دهد.**
- **`postqueue -f` صف را فوراً پردازش می‌کند.**
- **`mailx` پیوست پشتیبانی می‌کند.**
- **`newaliases` بعد از تغییر aliases.**
- **`postmap` بعد از تغییر lookup table.**
- **برای relay، SASL و TLS لازم است.**
- **`nullmailer`/`msmtp` جایگزین سبک برای فقط ارسال.**
- **`mail` به‌عنوان root کاراکترهای اضافه دارد؛ با `-r` حل می‌شود.**

### خلاصه فصل

MTA (مثل Postfix) ایمیل را جابجا می‌کند؛ MDA به صندوق تحویل می‌دهد و MUA کلاینت است. SMTP ارسال و POP3/IMAP دریافت را انجام می‌دهند. Postfix رایج‌ترین MTA است. `mail`, `sendmail`, `mailq`, `postqueue` ابزارهای خط فرمان‌اند. پیکربندی اشتباه می‌تواند Open Relay بسازد؛ `mynetworks` را محدود کنید.

---

## فصل ۴۷: مدیریت چاپگر

### مقدمه

در لینوکس، سیستم چاپ **CUPS (Common Unix Printing System)** است. تصور کنید باید یک چاپگر شبکه‌ای را برای دفتر راه‌اندازی کنید و از چند کامپیوتر به آن چاپ کنید.

در این فصل نصب و مدیریت چاپگر را یاد می‌گیرید.

### مفاهیم اساسی

**CUPS چیست؟**

سیستم چاپ استاندارد که چاپگرها را مدیریت و کارهای چاپ را صف‌بندی می‌کند.

**اجزا:**

- **cupsd:** daemon چاپ.
- **Port 631:** رابط وب CUPS.
- **PPD:** فایل توصیف چاپگر (PostScript Printer Description).
- **Printer drivers:** درایورهای چاپگر.
- **Spool:** صف چاپ.

**دستورها:** `lp`, `lpr`, `lpstat`, `lpadmin`, `cancel`, `lprm`.

### مثال‌های عملی

**نصب CUPS:**

```bash
# نصب
sudo apt install cups cups-client    # Debian
sudo dnf install cups                 # RHEL

# فعال‌سازی
sudo systemctl enable --now cups

# دسترسی وب
# http://localhost:631

# افزودن کاربر به گروه
sudo usermod -aG lpadmin $USER
```

**مدیریت چاپ:**

```bash
# چاپ فایل
lp file.txt
lpr file.txt

# چاپ با چاپگر مشخص
lp -d HP_LaserJet file.txt
lpr -P HP_LaserJet file.txt

# چاپ چند نسخه
lp -n 3 file.txt

# فهرست چاپگرها
lpstat -p -d
lpstat -t           # همه اطلاعات

# فهرست کارهای چاپ
lpq
lpstat -o

# لغو کار
cancel 5            # با ID
cancel -a           # همه
lprm -              # همه کارهای خودتان
lprm 5

# تنظیم چاپگر پیش‌فرض
lpoptions -d HP_LaserJet
```

**افزودن چاپگر:**

```bash
# از طریق رابط وب (http://localhost:631) یا:
lpadmin -p MyPrinter -E -v ipp://192.168.1.50/ipp/print -m everywhere

# USB
lpadmin -p LocalPrinter -E -v usb://HP/LaserJet -m everywhere

# تنظیم پیش‌فرض
lpadmin -d MyPrinter
```

**بررسی:**

```bash
# دستگاه‌های چاپ در دسترس
lpinfo -v

# درایورهای موجود
lpinfo -m

# فایل PPD
ls /etc/cups/ppd/

# پیکربندی
cat /etc/cups/cupsd.conf
```

### مقایسه و الگوهای مختلف

**دستورهای معادل:**

| BSD-style | System V-style |
|---|---|
| `lpr` | `lp` |
| `lpq` | `lpstat` |
| `lprm` | `cancel` |

**انواع اتصال چاپگر:**

| نوع | URI | توضیح |
|---|---|---|
| USB | `usb://...` | مستقیم |
| Network (IPP) | `ipp://...` | مدرن |
| Network (LPD) | `lpd://...` | قدیمی |
| SMB | `smb://...` | اشتراک ویندوز |
| Parallel | `parallel:/dev/lp0` | قدیمی |

**درایورها:**

| نوع | توضیح |
|---|---|
| `everywhere` | IPP Everywhere (بدون درایور) |
| PPD | فایل درایور |
| PostScript | مستقیم |

### اشکال شایع

**اشکال ۱: نبود درایور.**
چاپگر کار نمی‌کند؛ درایور یا `everywhere` نصب کنید. راه‌حل: `-m everywhere`.

**اشکال ۲: دسترسی نداشتن به CUPS.**
کاربر باید در گروه `lpadmin` باشد. راه‌حل: `usermod -aG lpadmin`.

**اشکال ۳: چاپگر offline.**
با `lpstat -p` بررسی کنید. راه‌حل: وضعیت چاپگر.

**اشکال ۴: فایروال پورت 631.**
دسترسی وب مسدود می‌شود. راه‌حل: باز کردن.

**اشکال ۵: کار گیر کرده.**
صف قفل شده. راه‌حل: `cancel -a` و restart cupsd.

**اشکال ۶: اشتباه در URI.**
چاپگر پیدا نمی‌شود. راه‌حل: `lpinfo -v` برای URI درست.

### نکات مهم (Tips & Tricks)

- **رابط وب CUPS (پورت 631) مدیریت گرافیکی می‌دهد.**
- **`lpstat -t` وضعیت کامل چاپ را نشان می‌دهد.**
- **`lpinfo -v` دستگاه‌های چاپ در دسترس را فهرست می‌کند.**
- **`lpadmin` برای افزودن/حذف چاپگر.**
- **فایل PPD در `/etc/cups/ppd/` است.**
- **`cupsd.conf` پیکربندی daemon است.**
- **`everywhere` چاپگرهای مدرن را بدون درایور پشتیبانی می‌کند.**
- **`enable`/`disable` چاپگر را فعال/غیرفعال می‌کند.**
- **`accept`/`reject` پذیرش کار را کنترل می‌کند.**
- **`cupsctl` پیکربندی را از خط فرمان تغییر می‌دهد.**
- **`lpmove` کارها را بین چاپگرها منتقل می‌کند.**
- **برای اشتراک چاپگر، CUPS از شبکه پشتیبانی می‌کند.**

### خلاصه فصل

CUPS سیستم چاپ استاندارد لینوکس است. `cupsd` روی پورت 631 اجرا می‌شود و رابط وب دارد. با `lp`/`lpr` چاپ، با `lpstat`/`lpq` وضعیت و با `cancel`/`lprm` لغو کنید. `lpadmin` چاپگر اضافه/حذف می‌کند. درایور مناسب (یا `everywhere`) و عضویت در گروه `lpadmin` لازم است. `lpinfo -v` URI درست را نشان می‌دهد.

---

## منابع و مراجع

- man pages: `man timedatectl`, `man chronyc`, `man journalctl`, `man postfix`, `man lp`, `man cupsd`
- chrony Documentation: https://chrony-project.org/documentation.html
- systemd journal: https://www.freedesktop.org/software/systemd/man/journalctl.html
- CUPS Documentation: https://www.cups.org/documentation.html
- Postfix Documentation: https://www.postfix.org/documentation.html

**بعدی:** [فایل ۱۳ — مبانی شبکه](13-Networking-Fundamentals.md)
