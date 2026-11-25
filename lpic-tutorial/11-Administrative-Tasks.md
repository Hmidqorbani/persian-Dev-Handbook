# فایل ۱۱ — وظایف مدیریتی

**مدت یادگیری**: ۷ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۹ (شل و اسکریپت)

---

## فهرست محتوا

- [فصل ۴۱: مدیریت کاربران، گروه‌ها و فایل‌های سیستمی](#فصل-۴۱-مدیریت-کاربران-گروهها-و-فایلهای-سیستمی)
- [فصل ۴۲: خودکارسازی با cron و at](#فصل-۴۲-خودکارسازی-با-cron-و-at)
- [فصل ۴۳: بومی‌سازی و بین‌المللی‌سازی](#فصل-۴۳-بومی‌سازی-و-بین‌المللی‌سازی)

---

## فصل ۴۱: مدیریت کاربران، گروه‌ها و فایل‌های سیستمی

### مقدمه

هر کاربر در لینوکس یک حساب با نام کاربری، رمز و دسترسی‌های مشخص دارد. تصور کنید کارمند جدیدی وارد شرکت می‌شود — باید حساب بسازید، به گروه‌ها اضافه کنید و دسترسی بدهید. یا کارمندی خارج می‌شود — باید حسابش را قفل کنید.

مدیریت کاربران یکی از مهم‌ترین وظایف مدیر سیستم است و در آزمون LPIC وزن دارد.

### مفاهیم اساسی

**فایل‌های کلیدی:**

| فایل | محتوا | دسترسی |
|---|---|---|
| `/etc/passwd` | اطلاعات پایهٔ کاربران | خوانا برای همه |
| `/etc/shadow` | رمزهای رمزنگاری‌شده | فقط ریشه |
| `/etc/group` | اطلاعات گروه‌ها | خوانا برای همه |
| `/etc/gshadow` | رمز گروه‌ها | فقط ریشه |
| `/etc/skel/` | الگوی پوشهٔ خانهٔ کاربر جدید | — |
| `/etc/login.defs` | پیش‌فرض‌های ساخت کاربر | — |
| `/etc/default/useradd` | پیش‌فرض‌های useradd | — |

**ساختار `/etc/passwd`:**

```
name:x:UID:GID:comment:home:shell
user:x:1000:1000:User Name:/home/user:/bin/bash
 │  │  │   │    │        │          │
 │  │  │   │    │        │          └── شل ورود
 │  │  │   │    │        └── پوشهٔ خانه
 │  │  │   │    └── توضیح (GECOS)
 │  │  │   └── GID گروه اصلی
 │  │  └── UID
 │  └── رمز (x = در shadow)
 └── نام کاربری
```

**ساختار `/etc/shadow`:**

```
name:password:lastchange:min:max:warn:inactive:expire:reserved
user:$6$xxx:19000:0:99999:7:::
```

**محدودهٔ UID:**

| محدوده | نوع |
|---|---|
| 0 | ریشه |
| 1–999 | کاربران سیستمی |
| 1000+ | کاربران عادی |

**گروه‌های مهم:**

| گروه | دسترسی |
|---|---|
| `sudo` / `wheel` | اجرای دستور به‌عنوان ریشه |
| `docker` | مدیریت Docker |
| `adm` | خواندن لاگ‌ها |
| `www-data` | وب‌سرور |

### مثال‌های عملی

**مدیریت کاربر:**

```bash
# ساخت کاربر
sudo useradd -m -s /bin/bash ali
sudo useradd -m -c "Ali Rezaei" -G sudo,developers ali
sudo useradd -m -u 1500 -d /home/ali -s /bin/bash ali

# با رمز
sudo passwd ali

# یا همه‌چیز یکجا (Debian)
sudo adduser ali

# حذف
sudo userdel ali
sudo userdel -r ali      # با پوشهٔ خانه

# تغییر
sudo usermod -aG docker ali   # افزودن به گروه (بدون حذف قبلی)
sudo usermod -s /bin/zsh ali  # تغییر شل
sudo usermod -d /new/home -m ali  # تغییر پوشه
sudo usermod -l newname oldname  # تغییر نام
sudo usermod -c "New Name" ali   # تغییر توضیح
sudo usermod -u 1500 ali         # تغییر UID

# قفل/باز کردن
sudo usermod -L ali
sudo usermod -U ali
sudo passwd -l ali
sudo passwd -u ali

# انقضا
sudo usermod -e 2027-01-01 ali
sudo chage -E 2027-01-01 ali
```

**مدیریت گروه:**

```bash
sudo groupadd developers
sudo groupadd -g 1500 developers
sudo groupdel developers
sudo groupmod -n newname oldname
sudo groupmod -g 1600 developers

# عضویت
sudo gpasswd -a ali developers   # افزودن
sudo gpasswd -d ali developers   # حذف
sudo gpasswd -A ali developers   # مدیر گروه

# یا
sudo usermod -aG developers ali
```

**اطلاعات:**

```bash
id ali
groups ali
getent passwd ali
getent group developers
finger ali         # نیاز به نصب
cat /etc/passwd | grep ali
```

**مدیریت رمز:**

```bash
# تغییر رمز
passwd
sudo passwd ali

# سیاست رمز
sudo chage -l ali
sudo chage -M 90 ali      # حداکثر ۹۰ روز
sudo chage -m 7 ali       # حداقل ۷ روز
sudo chage -W 14 ali      # هشدار ۱۴ روز قبل
sudo chage -I 30 ali      # غیرفعال ۳۰ روز بعد
sudo chage -E 2027-01-01 ali  # تاریخ انقضا
```

**ویرایش امن فایل‌ها:**

```bash
sudo vipw        # ویرایش /etc/passwd
sudo vipw -s     # ویرایش /etc/shadow
sudo vigr        # ویرایش /etc/group
sudo vigr -s     # ویرایش /etc/gshadow
```

### مقایسه و الگوهای مختلف

**ابزارهای سطح بالا و پایین:**

| ابزار | سطح | مزیت |
|---|---|---|
| `useradd` | پایین | کنترل کامل |
| `adduser` | بالا | تعاملی، آسان (Debian) |
| `usermod` | — | تغییر |
| `userdel` | — | حذف |

**`useradd -G` در برابر `usermod -aG`:**

| دستور | اثر |
|---|---|
| `useradd -G g1,g2 u` | فقط این گروه‌ها |
| `usermod -G g1,g2 u` | جایگزینی همهٔ گروه‌ها! |
| `usermod -aG g3 u` | افزودن (نگه‌داشتن قبلی‌ها) |

**توصیه:** برای افزودن به گروه همیشه `-aG`.

**روش‌های ساخت کاربر:**

| روش | مزیت | معایب |
|---|---|---|
| `useradd` | کنترل کامل | پیچیده |
| `adduser` | ساده | فقط Debian |
| ویرایش دستی | — | خطرناک |

### اشکال شایع

**اشکال ۱: `useradd` بدون `-m`.**
پوشهٔ خانه ساخته نمی‌شود. راه‌حل: `-m` اضافه کنید.

**اشکال ۲: `usermod -G` بدون `-a`.**
کاربر از گروه‌های قبلی حذف می‌شود! راه‌حل: `-aG`.

**اشکال ۳: ویرایش دستی `/etc/passwd`.**
ممکن است خراب شود؛ از `vipw` استفاده کنید. راه‌حل: `vipw`.

**اشکال ۴: فراموش کردن گروه sudo.**
کاربر نمی‌تواند sudo بزند. راه‌حل: `-G sudo`.

**اشکال ۵: رمز بدون انقضا.**
خطر امنیتی. راه‌حل: `chage -M 90`.

**اشکال ۶: حساب کاربر سیستمی با shell.**
خطر امنیتی. راه‌حل: `/usr/sbin/nologin`.

### نکات مهم (Tips & Tricks)

- **برای افزودن به گروه از `-aG` (append) استفاده کنید، نه `-G`.**
- **`vipw` و `vigr` ویرایش امن فایل‌های کاربری.**
- **`/etc/skel/` را برای فایل‌های پیش‌فرض کاربر جدید سفارشی کنید.**
- **`chage -l ali` وضعیت انقضای رمز را نشان می‌دهد.**
- **`passwd -S ali` وضعیت رمز را نشان می‌دهد.**
- **برای قفل کامل حساب از `usermod -L -e 1` استفاده کنید.**
- **حساب‌های سیستمی باید shell غیرقابل ورود داشته باشند.**
- **`getent passwd` از NSS استفاده می‌کند (شامل LDAP).**
- **`/etc/login.defs` پیش‌فرض‌ها را تعیین می‌کند.**
- **`pwck` و `grpck` صحت فایل‌ها را بررسی می‌کنند.**
- **`last` آخرین ورودها را نشان می‌دهد.**
- **`faillock --user ali` قفل شدن حساب را نشان می‌دهد.**

### خلاصه فصل

کاربران در `/etc/passwd` و رمزها در `/etc/shadow` ذخیره می‌شوند. با `useradd`/`adduser` بسازید، با `usermod` تغییر دهید و با `userdel` حذف کنید. گروه‌ها در `/etc/group` هستند. برای افزودن گروه از `-aG` استفاده کنید (نه `-G`). فایل‌های سیستمی را با `vipw`/`vigr` ویرایش کنید. `chage` سیاست رمز را مدیریت می‌کند و حساب‌های سیستمی باید shell غیرقابل ورود داشته باشند.

---

## فصل ۴۲: خودکارسازی با cron و at

### مقدمه

برای اجرای خودکار کارها در زمان مشخص، لینوکس ابزارهای **cron** (تکرارشونده) و **at** (یک‌باره) را دارد. تصور کنید هر شب ساعت ۲ بامداد باید بکاپ بگیرید یا هر ۵ دقیقه یک سرویس را بررسی کنید. cron این کار را خودکار می‌کند.

این‌ها ستون فقرات خودکارسازی سیستم‌اند و در آزمون LPIC می‌آیند.

### مفاهیم اساسی

**cron:** زمان‌بند تکرارشونده. **at:** زمان‌بند یک‌باره.

**ساختار crontab (پنج فیلد + دستور):**

```
* * * * * command
│ │ │ │ │
│ │ │ │ └── روز هفته (0-7، 0=یکشنبه، 7=یکشنبه)
│ │ │ └──── ماه (1-12)
│ │ └────── روز ماه (1-31)
│ └──────── ساعت (0-23)
└────────── دقیقه (0-59)
```

**کاراکترهای ویژه:**

| نماد | معنا |
|---|---|
| `*` | هر مقدار |
| `,` | لیست (مثلاً `1,3,5`) |
| `-` | محدوده (مثلاً `1-5`) |
| `/` | گام (مثلاً `*/15`) |

**میان‌برها:**

| میان‌بر | معادل |
|---|---|
| `@reboot` | در بوت |
| `@yearly` / `@annually` | `0 0 1 1 *` |
| `@monthly` | `0 0 1 * *` |
| `@weekly` | `0 0 * * 0` |
| `@daily` / `@midnight` | `0 0 * * *` |
| `@hourly` | `0 * * * *` |

### مثال‌های عملی

**crontab کاربر:**

```bash
# ویرایش
crontab -e

# فهرست
crontab -l

# حذف
crontab -r

# ویرایش crontab کاربر دیگر (ریشه)
sudo crontab -u alice -e
```

**مثال‌های crontab:**

```bash
# هر روز ساعت ۲ بامداد
0 2 * * * /home/user/backup.sh

# هر ۵ دقیقه
*/5 * * * * /usr/bin/check.sh

# هر دوشنبه ساعت ۹
0 9 * * 1 /usr/bin/report.sh

# روز اول هر ماه
0 0 1 * * /usr/bin/monthly.sh

# هر ۱۵ دقیقه در ساعات کاری (۹-۱۷، دوشنبه-جمعه)
*/15 9-17 * * 1-5 /usr/bin/task.sh

# با هدایت خروجی به لاگ
0 3 * * * /usr/bin/job.sh >> /var/log/job.log 2>&1

# در بوت
@reboot /usr/local/bin/startup.sh
```

**cron سیستمی:**

```bash
# فایل‌های سیستمی
/etc/crontab
/etc/cron.d/
/etc/cron.daily/
/etc/cron.hourly/
/etc/cron.weekly/
/etc/cron.monthly/

# مثال /etc/crontab (شامل ستون user):
# m h dom mon dow user command
0 3 * * * root /usr/local/bin/cleanup.sh
```

**at (یک‌باره):**

```bash
# نصب
sudo apt install at
sudo systemctl enable --now atd

# زمان‌بندی
echo "/home/user/script.sh" | at 15:00
at now + 1 hour
at 10:00 AM tomorrow

# داخل at:
at> /home/user/backup.sh
at> Ctrl+D

# فهرست
atq

# حذف
atrm 5

# دیدن محتوای یک job
at -c 5
```

**کنترل دسترسی:**

```bash
# /etc/cron.allow (فقط این‌ها)
# /etc/cron.deny (همه جز این‌ها)
# اگر allow باشد، deny نادیده گرفته می‌شود
```

### مقایسه و الگوهای مختلف

**cron در برابر systemd timer:**

| ویژگی | cron | systemd timer |
|---|---|---|
| فایل | crontab | `.timer` unit |
| لاگ | `/var/log/syslog` | `journalctl` |
| دقت | دقیقه | ثانیه |
| وابستگی | ندارد | دارد |
| جبران | ندارد | `Persistent=true` |

**مثال systemd timer:**

```bash
# /etc/systemd/system/backup.service
[Unit]
Description=Daily Backup

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh

# /etc/systemd/system/backup.timer
[Unit]
Description=Daily Backup Timer

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable --now backup.timer
systemctl list-timers
```

**فایل‌های cron:**

| مسیر | ستون user؟ |
|---|---|
| `/etc/crontab` | بله |
| `/etc/cron.d/` | بله |
| `/etc/cron.daily/` | خیر (اسکریپت) |
| `crontab -e` | خیر |

### اشکال شایع

**اشکال ۱: PATH محدود در cron.**
cron PATH مینیمال دارد؛ از مسیر مطلق استفاده کنید. راه‌حل: مسیر مطلق.

**اشکال ۲: نبود لاگ.**
cron خروجی را ایمیل می‌کند؛ برای لاگ `>> log 2>&1` بگذارید. راه‌حل: هدایت خروجی.

**اشکال ۳: اشتباه در syntax.**
با `crontab.guru` تست کنید. راه‌حل: تست syntax.

**اشکال ۴: فراموش کردن `atd`.**
at بدون سرویس اجرا نمی‌شود. راه‌حل: `systemctl enable --now atd`.

**اشکال ۵: نبود newline انتهایی.**
بعضی نسخه‌های cron آخرین خط بدون newline را اجرا نمی‌کنند. راه‌حل: newline اضافه کنید.

**اشکال ۶: اجرای cron با محیط اشتباه.**
متغیرهای محیطی تعریف نشده‌اند. راه‌حل: در crontab تعریف کنید.

### نکات مهم (Tips & Tricks)

- **از `crontab.guru` برای تست syntax استفاده کنید.**
- **همیشه مسیر مطلق دستورها را بنویسید.**
- **خروجی را به لاگ هدایت کنید: `cmd >> /var/log/x.log 2>&1`.**
- **`@reboot` برای اجرا در هر بوت.**
- **`@daily`, `@weekly`, `@monthly` میان‌برهای مفیدند.**
- **برای کارهای پیچیده، اسکریپت بسازید و آن را در cron صدا بزنید.**
- **`MAILTO=""` ایمیل را غیرفعال می‌کند.**
- **`SHELL=/bin/bash` در crontab شل را تعیین می‌کند.**
- **`systemd timer` جایگزین مدرن با لاگ بهتر است.**
- **`anacron` برای سیستم‌هایی که همیشه روشن نیستند.**
- **`at` برای کارهای یک‌باره در زمان خاص.**
- **`/etc/cron.d/` برای jobهای سیستمی سفارشی.**

### خلاصه فصل

cron برای کارهای تکرارشونده و at برای یک‌باره است. crontab پنج فیلد زمان + دستور دارد. cron سیستمی در `/etc/cron.*/` و `/etc/crontab` است. systemd timer جایگزین مدرن با لاگ بهتر و `Persistent=true` است. همیشه مسیر مطلق و هدایت خروجی به لاگ را رعایت کنید. `MAILTO` و `SHELL` را در crontab تنظیم کنید.

---

## فصل ۴۳: بومی‌سازی و بین‌المللی‌سازی

### مقدمه

**بومی‌سازی (Localization)** یعنی تطبیق سیستم با زبان، منطقه و فرهنگ کاربر. **بین‌المللی‌سازی (Internationalization)** یعنی آماده‌سازی نرم‌افزار برای پشتیبانی چند زبان.

تصور کنید لاگ‌ها با تاریخ آمریکایی هستند اما کاربر ایرانی می‌خواهد تاریخ شمسی. یا سیستم در منطقهٔ زمانی اشتباه است. این فصل این موضوعات را پوشش می‌دهد.

### مفاهیم اساسی

**Locale:** مجموعه‌ای از تنظیمات منطقه‌ای (زبان، تاریخ، عدد، ارز).

**ساختار locale:**

```
<language>_<TERRITORY>.<codeset>@<modifier>
fa_IR.UTF-8
en_US.UTF-8
de_DE.UTF-8
```

**متغیرهای مهم:**

| متغیر | کار |
|---|---|
| `LANG` | locale پیش‌فرض |
| `LC_ALL` | بازنویسی همه (بالاترین اولویت) |
| `LC_TIME` | فرمت تاریخ/زمان |
| `LC_NUMERIC` | فرمت اعداد |
| `LC_MESSAGES` | زبان پیام‌ها |
| `LC_MONETARY` | فرمت ارز |
| `LC_CTYPE` | کاراکترها |
| `LC_COLLATE` | مرتب‌سازی |

**اولویت:** `LC_ALL` > `LC_*` > `LANG`.

**منطقهٔ زمانی:**

```
/usr/share/zoneinfo/Asia/Tehran
/usr/share/zoneinfo/UTC
```

### مثال‌های عملی

**دیدن locale:**

```bash
locale
locale -a          # localeهای موجود
locale -a | grep fa
localectl status   # (systemd)
```

**تنظیم locale:**

```bash
# موقت
export LANG=fa_IR.UTF-8
export LC_ALL=fa_IR.UTF-8

# فقط تاریخ
export LC_TIME=fa_IR.UTF-8

# دائمی (Debian/Ubuntu)
sudo dpkg-reconfigure locales
# یا ویرایش /etc/default/locale

# RHEL/Rocky
sudo localectl set-locale LANG=fa_IR.UTF-8
localectl status
```

**ساخت locale:**

```bash
# Debian/Ubuntu
sudo locale-gen fa_IR.UTF-8
sudo update-locale LANG=fa_IR.UTF-8

# ویرایش /etc/locale.gen و uncomment
sudo vim /etc/locale.gen
sudo locale-gen

# RHEL
sudo localedef -i fa_IR -f UTF-8 fa_IR.UTF-8
```

**تنظیم زمان:**

```bash
# timedatectl
timedatectl
sudo timedatectl set-timezone Asia/Tehran
sudo timedatectl set-time "2026-10-16 10:00:00"
sudo timedatectl set-ntp true

# فایل
ls /usr/share/zoneinfo/
sudo ln -sf /usr/share/zoneinfo/Asia/Tehran /etc/localtime

# متغیر
export TZ=Asia/Tehran
```

**تبدیل کدگذاری:**

```bash
# تبدیل ISO-8859-1 به UTF-8
iconv -f ISO-8859-1 -t UTF-8 in.txt > out.txt

# فهرست کدگذاری‌ها
iconv -l

# بررسی نوع
file -i file.txt
```

### مقایسه و الگوهای مختلف

**ابزارهای تنظیم:**

| توزیع | ابزار |
|---|---|
| Debian/Ubuntu | `dpkg-reconfigure`, `locale-gen`, `update-locale` |
| RHEL/Rocky | `localectl` |
| عمومی | ویرایش `/etc/default/locale` یا `/etc/locale.conf` |

**فرمت تاریخ در localeهای مختلف:**

| Locale | نمونه |
|---|---|
| en_US | 10/16/2026 |
| fa_IR | ۱۴۰۵/۰۷/۲۴ |
| de_DE | 16.10.2026 |
| ja_JP | 2026年10月16日 |

**فرمت عدد:**

| Locale | 1234.56 |
|---|---|
| en_US | 1,234.56 |
| de_DE | 1.234,56 |
| fa_IR | ۱٬۲۳۴٫۵۶ |

**سطح تنظیم locale:**

| سطح | فایل |
|---|---|
| سیستم | `/etc/default/locale` |
| کاربر | `~/.bashrc` |
| جلسه | `export` |

### اشکال شایع

**اشکال ۱: locale نصب‌نشده.**
خطای «Cannot set LC_ALL». راه‌حل: `locale-gen`.

**اشکال ۲: `LC_ALL` اشتباه.**
همهٔ تنظیمات را بازنویسی می‌کند. راه‌حل: فقط در صورت نیاز.

**اشکال ۳: فراموش کردن `locale-gen`.**
تغییر در `/etc/locale.gen` اعمال نمی‌شود. راه‌حل: `locale-gen`.

**اشکال ۴: اشتباه در منطقهٔ زمانی.**
زمان لاگ‌ها اشتباه می‌شود. راه‌حل: `timedatectl`.

**اشکال ۵: کدگذاری اشتباه فایل.**
کاراکترهای عجیب نمایش داده می‌شوند. راه‌حل: `iconv`.

**اشکال ۶: locale انگلیسی روی سرور فارسی.**
پیام‌ها انگلیسی می‌مانند. راه‌حل: `LC_MESSAGES`.

### نکات مهم (Tips & Tricks)

- **`LC_ALL` بر همه اولویت دارد؛ در عیب‌یابی موقتاً از آن استفاده کنید.**
- **`export LC_TIME=fa_IR.UTF-8` فقط فرمت تاریخ را تغییر می‌دهد.**
- **`timedatectl` هم زمان و هم منطقهٔ زمانی را مدیریت می‌کند.**
- **برای سرور، معمولاً `C.UTF-8` یا `en_US.UTF-8` توصیه می‌شود.**
- **`iconv` برای تبدیل کدگذاری فایل‌ها: `iconv -f X -t UTF-8 in > out`.**
- **`localectl` (systemd) روش مدرن.**
- **`locale -a` localeهای نصب‌شده را نشان می‌دهد.**
- **`LC_ALL=C` همه را به پیش‌فرض می‌برد (برای اسکریپت).**
- **`/etc/locale.gen` در Debian.**
- **`date` با `TZ` منطقهٔ موقت تعیین می‌کند.**
- **`chardet`/`enca` تشخیص کدگذاری.**
- **`tzselect` انتخاب منطقهٔ زمانی تعاملی.**

### خلاصه فصل

Locale تنظیمات منطقه‌ای (زبان، تاریخ، عدد) را تعیین می‌کند. `LANG` پیش‌فرض و `LC_ALL` بازنویسی‌کنندهٔ همه است. با `locale-gen`/`dpkg-reconfigure` (Debian) یا `localectl` (RHEL) تنظیم می‌شود. `timedatectl` زمان و منطقهٔ زمانی را مدیریت می‌کند. `iconv` برای تبدیل کدگذاری مفید است. اولویت: `LC_ALL` > `LC_*` > `LANG`.

---

## منابع و مراجع

- man pages: `man useradd`, `man usermod`, `man crontab`, `man at`, `man locale`, `man timedatectl`
- crontab.guru: https://crontab.guru
- systemd Timers: https://www.freedesktop.org/software/systemd/man/systemd.timer.html
- glibc Locales: https://www.gnu.org/software/libc/manual/html_node/Locales.html

**بعدی:** [فایل ۱۲ — سرویس‌های ضروری سیستم](12-Essential-System-Services.md)
