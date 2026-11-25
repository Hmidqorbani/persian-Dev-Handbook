# فایل ۰۴ — نصب و مدیریت پکیج

**مدت یادگیری**: ۸ ساعت
**سطح سختی**: مبتدی–متوسط
**پیش‌نیاز**: فایل ۰۲ (آزمایشگاه)، فایل ۰۳ (معماری سیستم)

---

## فهرست محتوا

- [فصل ۱۴: طراحی چیدمان دیسک](#فصل-۱۴-طراحی-چیدمان-دیسک)
- [فصل ۱۵: نصب Boot Manager](#فصل-۱۵-نصب-boot-manager)
- [فصل ۱۶: مدیریت کتابخانه‌های اشتراکی](#فصل-۱۶-مدیریت-کتابخانههای-اشتراکی)
- [فصل ۱۷: مدیریت پکیج Debian (dpkg/apt)](#فصل-۱۷-مدیریت-پکیج-debian-dpkgapt)
- [فصل ۱۸: مدیریت پکیج RPM (rpm/yum/dnf)](#فصل-۱۸-مدیریت-پکیج-rpm-rpmyumdnf)
- [فصل ۱۹: لینوکس به‌عنوان مهمان مجازی‌سازی](#فصل-۱۹-لینوکس-بهعنوان-مهمان-مجازیسازی)

---

## فصل ۱۴: طراحی چیدمان دیسک

### مقدمه

قبل از نصب لینوکس، باید تصمیم بگیرید که دیسک چطور پارتیشن‌بندی شود. این تصمیم روی کارایی، امنیت و قابلیت بازیابی اثر می‌گذارد. تصور کنید سروری دارید که لاگ‌هایش `/var` را پر می‌کند — اگر `/var` پارتیشن جدا نباشد، کل سیستم از کار می‌افتد.

طراحی چیدمان دیسک یک تصمیم معماری است. در این فصل اصول آن را یاد می‌گیرید: چه پارتیشن‌هایی لازم است، هر کدام چقدر فضا می‌خواهد، و چرا.

### مفاهیم اساسی

**مفاهیم پایه:**

- **Partition (پارتیشن):** بخشی از دیسک که جدا مدیریت می‌شود.
- **MBR (Master Boot Record):** جدول پارتیشن قدیمی.
- **GPT (GUID Partition Table):** جدول پارتیشن جدید.
- **Filesystem (فایل‌سیستم):** ساختاری که روی پارتیشن داده ذخیره می‌کند.
- **Mount point:** پوشه‌ای که فایل‌سیستم به آن متصل می‌شود.

**مقایسه MBR و GPT:**

| ویژگی | MBR | GPT |
|---|---|---|
| حداکثر پارتیشن اصلی | ۴ | ۱۲۸ |
| حداکثر حجم دیسک | ۲ TB | ۹.۴ ZB |
| پشتیبانی UEFI | خیر | بله |
| پشتیبانی BIOS | بله | بله (با BIOS boot partition) |
| افزونگی | خیر | بله (کپی پشتیبان در انتهای دیسک) |
| CRC | ندارد | دارد (تشخیص خرابی) |

**پارتیشن‌های پیشنهادی برای سرور:**

| mount point | حجم پیشنهادی | کاربرد | چرا جدا؟ |
|---|---|---|---|
| `/` (root) | ۱۰–۲۰ GB | سیستم پایه | — |
| `/home` | بقیه یا زیاد | داده‌های کاربران | حفظ داده در reinstall |
| `/var` | ۱۰–۲۰ GB | لاگ‌ها و دادهٔ سرویس‌ها | جلوگیری از پر شدن ریشه |
| `swap` | ۱–۲ برابر رم | حافظهٔ مجازی | — |
| `/boot` | ۵۰۰ MB | کرنل و GRUB | برای LVM/رمزنگاری |
| `/boot/efi` | ۲۰۰–۵۰۰ MB | برای UEFI | الزامی در UEFI |
| `/tmp` | ۱–۵ GB | فایل‌های موقت | جلوگیری از پر شدن ریشه |

**چرا جدا کردن پارتیشن‌ها مهم است؟**

- **امنیت:** می‌توانید `/home` یا `/tmp` را با `noexec`، `nosuid` mount کنید.
- **پایداری:** پر شدن `/var` (لاگ‌ها) سیستم را از کار نمی‌اندازد.
- **بازیابی:** می‌توانید سیستم را بدون از دست دادن `/home` دوباره نصب کنید.
- **بکاپ:** پارتیشن‌های داده را راحت‌تر بکاپ می‌گیرید.
- **کارایی:** پارتیشن‌های مختلف می‌توانند روی دیسک‌های مختلف باشند.

### مثال‌های عملی

**دیدن ساختار دیسک:**

```bash
# فهرست دیسک‌ها و پارتیشن‌ها
lsblk
lsblk -f    # با فایل‌سیستم

# اطلاعات کامل
sudo fdisk -l

# ساختار GPT
sudo gdisk -l /dev/sda

# ابزار parted
sudo parted -l
```

**ایجاد پارتیشن با `fdisk` (تعاملی):**

```bash
sudo fdisk /dev/sdb
# داخل fdisk:
#   m → راهنما
#   p → نمایش پارتیشن‌های فعلی
#   n → پارتیشن جدید
#     p → primary (یا e برای extended در MBR)
#     1 → شمارهٔ پارتیشن
#     Enter → شروع پیش‌فرض
#     +2G → اندازه ۲ گیگابایت
#   t → تغییر نوع (مثلاً 82 برای swap، 8e برای LVM)
#   w → نوشتن و خروج
#   q → خروج بدون ذخیره
```

**ایجاد پارتیشن با `parted` (مدرن‌تر، اسکریپت‌پذیر):**

```bash
# ایجاد جدول GPT
sudo parted /dev/sdb mklabel gpt

# ساخت پارتیشن ۲ گیگابایتی
sudo parted /dev/sdb mkpart primary ext4 1MiB 2GiB

# ساخت پارتیشن swap
sudo parted /dev/sdb mkpart primary linux-swap 2GiB 4GiB

# نمایش
sudo parted /dev/sdb print
```

**ایجاد پارتیشن با `gdisk` (برای GPT):**

```bash
sudo gdisk /dev/sdb
#   n → پارتیشن جدید
#   1 → شماره
#   Enter → شروع پیش‌فرض
#   +5G → اندازه
#   8300 → کد نوع (Linux filesystem)
#   w → نوشتن
```

**محاسبهٔ اندازهٔ swap:**

```bash
# قاعدهٔ سنتی: ۲ برابر رم (برای hibernation)
# قاعدهٔ مدرن: نصف رم یا ۲ گیگ (برای سیستم بدون hibernation)
free -h
```

### مقایسه و الگوهای مختلف

**ابزارهای پارتیشن‌بندی:**

| ابزار | کاربرد | تعاملی؟ |
|---|---|---|
| `fdisk` | MBR/GPT، سنتی | بله |
| `gdisk` | مخصوص GPT | بله |
| `parted` | مدرن، اسکریپت‌پذیر | هر دو |
| `sfdisk` | اسکریپت‌پذیر، غیرتعاملی | خیر |
| `cfdisk` | رابط متنی زیبا | بله |

**طرح‌های چیدمان رایج:**

| طرح | مناسب |
|---|---|
| ساده (فقط `/` + swap) | VM آزمایشگاهی |
| استاندارد (`/`, `/home`, swap) | دسکتاپ |
| سرور (`/`, `/home`, `/var`, `/boot`, swap) | سرور |
| امن (`/`, `/home`, `/var`, `/tmp`, `/boot`, swap جدا) | سرور تولیدی |
| LVM (همه روی LVM) | انعطاف‌پذیر |

### اشکال شایع

**اشکال ۱: نداشتن swap.**
بدون swap، سیستم در فشار حافظه کند یا کرش می‌کند. راه‌حل: swap بگذارید (حداقل ۲ گیگ).

**اشکال ۲: `/boot` خیلی کوچک.**
با چند کرنل، ۵۰۰ MB لازم است. راه‌حل: حداقل ۵۰۰ MB.

**اشکال ۳: نادیده گرفتن `/var` جدا.**
لاگ‌های پرحجم می‌توانند ریشه را پر کنند. راه‌حل: `/var` جدا بگذارید.

**اشکال ۴: پارتیشن‌بندی دستی بدون برنامه.**
برای آزمایشگاه، خودکار کافی است؛ برای سرور، برنامهٔ مشخص لازم است. راه‌حل: طرح از قبل طراحی کنید.

**اشکال ۵: اشتباه نوع پارتیشن.**
پارتیشن swap باید نوع `82` (MBR) یا `8200` (GPT) باشد. راه‌حل: نوع درست انتخاب کنید.

**اشکال ۶: نبود پارتیشن EFI در UEFI.**
سیستم بوت نمی‌شود. راه‌حل: پارتیشن EFI (vfat) بسازید.

### نکات مهم (Tips & Tricks)

- **برای سرور تولیدی، `/var` و `/home` را جدا کنید.**
- **اندازهٔ swap برای سیستم بدون hibernation می‌تواند نصف رم باشد.**
- **`lsblk -f` فایل‌سیستم هر پارتیشن را نشان می‌دهد.**
- **برای دیسک‌های بزرگ (>۲TB) حتماً GPT استفاده کنید.**
- **قبل از پارتیشن‌بندی، از داده‌های مهم بکاپ بگیرید.**
- **`parted` برای اسکریپت‌نویسی بهتر است.**
- **`partprobe` بعد از تغییر پارتیشن‌ها، کرنل را به‌روز می‌کند.**
- **با `sfdisk --dump` می‌توانید جدول پارتیشن را بکاپ/بازیابی کنید.**
- **برای VM آزمایشگاهی، «Use an entire disk» ساده‌ترین است.**
- **با `df -h` بعد از نصب، چیدمان را بررسی کنید.**

### خلاصه فصل

چیدمان دیسک شامل انتخاب جدول پارتیشن (MBR/GPT) و ساخت پارتیشن‌های منطقی است. برای سرور، جدا کردن `/`, `/home`, `/var`, `/boot` و swap توصیه می‌شود. ابزارهای `fdisk`, `gdisk`, `parted` برای پارتیشن‌بندی به‌کار می‌روند. برای دیسک بزرگ GPT و برای UEFI پارتیشن EFI لازم است. جدا کردن پارتیشن‌ها امنیت، پایداری، بازیابی و کارایی را بهبود می‌دهد.

---

## فصل ۱۵: نصب Boot Manager

### مقدمه

**Boot Manager** (مدیر بوت) برنامه‌ای است که پس از BIOS/UEFI اجرا می‌شود و سیستم‌عامل را بارگذاری می‌کند. اگر GRUB خراب شود، سیستم بوت نمی‌شود. پس فهم نصب و پیکربندی آن حیاتی است.

در لینوکس، استاندارد امروز **GRUB2** است. این فصل نصب، پیکربندی و عیب‌یابی آن را پوشش می‌دهد.

### مفاهیم اساسی

**GRUB2 از کجا نصب می‌شود؟**

- در حالت **BIOS/MBR:** GRUB در **MBR** دیسک نصب می‌شود (۵۱۲ بایت اول).
- در حالت **UEFI:** GRUB در **EFI System Partition** (`/boot/efi`) نصب می‌شود.

**فایل‌های مهم GRUB2:**

| مسیر | محتوا | ویرایش؟ |
|---|---|---|
| `/boot/grub/grub.cfg` | پیکربندی نهایی (تولیدشده) | هرگز مستقیم |
| `/etc/default/grub` | تنظیمات اصلی | بله |
| `/etc/grub.d/` | اسکریپت‌های تولید | در صورت نیاز |
| `/boot/grub/` | فایل‌های GRUB | — |

**مراحل بوت GRUB:**

```
۱. BIOS/UEFI، GRUB را از MBR یا ESP بارگذاری می‌کند
۲. GRUB فایل grub.cfg را می‌خواند
۳. منو نمایش داده می‌شود
۴. کاربر (یا پیش‌فرض) انتخاب می‌کند
۵. کرنل + initramfs بارگذاری می‌شوند
۶. کنترل به کرنل منتقل می‌شود
```

### مثال‌های عملی

**نصب GRUB:**

```bash
# Debian/Ubuntu
sudo apt install grub-pc        # برای BIOS
sudo apt install grub-efi-amd64 # برای UEFI

# نصب روی دیسک
sudo grub-install /dev/sda

# برای UEFI
sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi

# RHEL/Rocky
sudo dnf install grub2
sudo grub2-install /dev/sda
```

**پیکربندی:**

```bash
# ویرایش تنظیمات
sudo vim /etc/default/grub

# نمونهٔ محتوا:
GRUB_TIMEOUT=5
GRUB_DEFAULT=0
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX=""

# بازتولید پیکربندی
sudo update-grub                              # Debian/Ubuntu
sudo grub2-mkconfig -o /boot/grub2/grub.cfg   # RHEL
sudo grub-mkconfig -o /boot/grub/grub.cfg     # عمومی
```

**افزودن ورودی سفارشی:**

```bash
# /etc/grub.d/40_custom
menuentry "My Custom Entry" {
    set root=(hd0,1)
    linux /vmlinuz root=/dev/sda1 ro
    initrd /initrd.img
}

# سپس
sudo update-grub
```

**بررسی و بازیابی:**

```bash
# بررسی پیکربندی
sudo grub-script-check /boot/grub/grub.cfg

# نصب مجدد GRUB از live CD
sudo mount /dev/sda1 /mnt
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo chroot /mnt
grub-install /dev/sda
update-grub
exit
```

### مقایسه و الگوهای مختلف

**مدیران بوت:**

| مدیر بوت | مناسب | مزیت | معایب |
|---|---|---|---|
| **GRUB2** | همه‌جا | انعطاف، پشتیبانی گسترده | پیچیده |
| **systemd-boot** | UEFI | ساده، سریع | فقط UEFI |
| **LILO** | قدیمی | ساده | منسوخ |
| **rEFInd** | UEFI | زیبا، خودکار | فقط UEFI |

**پارامترهای کلیدی `/etc/default/grub`:**

| پارامتر | اثر |
|---|---|
| `GRUB_TIMEOUT` | مدت نمایش منو (ثانیه) |
| `GRUB_DEFAULT` | ورودی پیش‌فرض (شماره یا نام) |
| `GRUB_CMDLINE_LINUX_DEFAULT` | پارامترهای کرنل پیش‌فرض |
| `GRUB_DISABLE_OS_PROBER` | غیرفعال‌سازی تشخیص سیستم‌عامل‌های دیگر |
| `GRUB_TERMINAL` | نوع ترمینال (console/serial) |
| `GRUB_GFXMODE` | رزولوشن گرافیکی |

**نصب در BIOS در برابر UEFI:**

| ویژگی | BIOS | UEFI |
|---|---|---|
| محل نصب | MBR دیسک | EFI System Partition |
| دستور | `grub-install /dev/sda` | `grub-install --target=x86_64-efi` |
| مدیریت ترتیب | ترتیب دیسک در BIOS | `efibootmgr` |
| پارتیشن لازم | ندارد | `/boot/efi` (vfat) |

### اشکال شایع

**اشکال ۱: ویرایش `grub.cfg`.**
این فایل تولیدشده است؛ با `update-grub` بازنویسی می‌شود. راه‌حل: از `/etc/default/grub` استفاده کنید.

**اشکال ۲: فراموش کردن بازتولید.**
بعد از ویرایش `/etc/default/grub` حتماً `update-grub` بزنید. راه‌حل: همیشه بازتولید کنید.

**اشکال ۳: نصب GRUB روی پارتیشن اشتباه.**
`grub-install /dev/sda` (دیسک) نه `/dev/sda1` (پارتیشن). راه‌حل: به آدرس دقت کنید.

**اشکال ۴: نبود OS Prober.**
سیستم‌های دیگر (مثل ویندوز) در منو دیده نمی‌شوند. راه‌حل: `os-prober` را نصب کنید.

**اشکال ۵: خرابی GRUB بعد از به‌روزرسانی ویندوز.**
ویندوز MBR را بازنویسی می‌کند. راه‌حل: با live CD، GRUB را دوباره نصب کنید.

**اشکال ۶: اشتباه در UEFI.**
اگر پارتیشن EFI وجود نداشته باشد، نصب شکست می‌خورد. راه‌حل: پارتیشن EFI (vfat) بسازید.

### نکات مهم (Tips & Tricks)

- **برای تشخیص سیستم‌عامل‌های دیگر، `os-prober` را نصب کنید.**
- **`GRUB_TIMEOUT=0` برای بوت سریع سرور.**
- **با نگه‌داشتن `Shift` در بوت (BIOS) منو را ببینید.**
- **برای UEFI، `efibootmgr` ترتیب بوت را مدیریت می‌کند.**
- **همیشه یک rescue disk/USB داشته باشید.**
- **`grub-mkconfig -o /boot/grub/grub.cfg` معادل `update-grub` است.**
- **`GRUB_DEFAULT=saved` + `grub-reboot` بوت یک‌باره به ورودی دیگر.**
- **برای رمزگذاری منو، `grub-mkpasswd-pbkdf2` و `set superusers`.**
- **`grub-mkrescue` یک ISO بوت‌پذیر می‌سازد.**
- **بعد از نصب کرنل جدید، `update-grub` خودکار اجرا می‌شود.**

### خلاصه فصل

GRUB2 استاندارد مدیریت بوت لینوکس است. در BIOS در MBR و در UEFI در EFI System Partition نصب می‌شود. پیکربندی از `/etc/default/grub` و با `update-grub`/`grub2-mkconfig` بازتولید می‌شود. هرگز `grub.cfg` را مستقیم ویرایش نکنید. برای بازیابی، از live CD با `chroot` و `grub-install` استفاده کنید. همیشه rescue disk داشته باشید.

---

## فصل ۱۶: مدیریت کتابخانه‌های اشتراکی

### مقدمه

تصور کنید برنامه‌ای را اجرا می‌کنید و خطای `error while loading shared libraries: libssl.so.1.1` می‌گیرید. این خطا یعنی برنامه به یک کتابخانهٔ اشتراکی نیاز دارد که پیدا نمی‌شود. بدون فهم کتابخانه‌های اشتراکی، حل این خطا دشوار است.

در لینوکس، برنامه‌ها از **کتابخانه‌های اشتراکی (shared libraries)** استفاده می‌کنند تا کد مشترک یکبار در حافظه بارگذاری شود. فهم این مفهوم برای عیب‌یابی ضروری است.

### مفاهیم اساسی

**کتابخانهٔ اشتراکی چیست؟**

فایلی با پسوند `.so` (shared object) که چند برنامه از آن استفاده می‌کنند. مثلاً `libc.so.6` توسط تقریباً همهٔ برنامه‌ها استفاده می‌شود.

**دو نوع کتابخانه:**

| نوع | پسوند | زمان پیوند | مزیت |
|---|---|---|---|
| **Static (ایستا)** | `.a` | زمان کامپایل | مستقل، بدون وابستگی |
| **Shared (اشتراکی)** | `.so` | زمان اجرا | صرفه‌جویی حافظه |

**مسیرهای اصلی کتابخانه‌ها:**

| مسیر | محتوا |
|---|---|
| `/lib/` , `/lib64/` | کتابخانه‌های سیستم |
| `/usr/lib/` , `/usr/lib64/` | کتابخانه‌های برنامه‌ها |
| `/usr/local/lib/` | کتابخانه‌های سفارشی |
| `/etc/ld.so.conf` | پیکربندی مسیرهای جستجو |
| `/etc/ld.so.conf.d/` | فایل‌های مسیر اضافی |
| `/etc/ld.so.cache` | کش کتابخانه‌ها |

**چطور کرنل کتابخانه را پیدا می‌کند؟**

```
۱. کرنل برنامه را اجرا می‌کند
۲. لودر دینامیک (/lib/ld-linux.so.2) اجرا می‌شود
۳. لودر از /etc/ld.so.cache مسیرها را می‌خواند
۴. کتابخانه‌ها بارگذاری می‌شوند
```

**متغیرهای محیطی:**

- **`LD_LIBRARY_PATH`:** مسیرهای اضافی برای جستجو (موقت).
- **`LD_PRELOAD`:** کتابخانه‌ای که قبل از بقیه بارگذاری می‌شود.
- **`LD_DEBUG`:** دیباگ لودر.

### مثال‌های عملی

**مشاهدهٔ وابستگی‌های یک برنامه:**

```bash
# کتابخانه‌های موردنیاز یک برنامه
ldd /bin/ls

# نمونه خروجی:
#   linux-vdso.so.1
#   libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1
#   libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6
#   /lib64/ld-linux-x86-64.so.2

# کتابخانه‌های یک فایل .so
objdump -p /lib/x86_64-linux-gnu/libc.so.6 | grep NEEDED

# با readelf
readelf -d /bin/ls | grep NEEDED
```

**مدیریت مسیرهای کتابخانه:**

```bash
# افزودن مسیر جدید
echo "/opt/myapp/lib" | sudo tee /etc/ld.so.conf.d/myapp.conf

# بازسازی کش
sudo ldconfig

# دیدن کش
ldconfig -p | grep libssl

# دیدن مسیرهای اسکن‌شده
ldconfig -v | head
```

**متغیر LD_LIBRARY_PATH (موقت):**

```bash
# برای یک اجرا
LD_LIBRARY_PATH=/opt/myapp/lib ./myapp

# برای جلسه
export LD_LIBRARY_PATH=/opt/myapp/lib
```

**دیباگ لودر:**

```bash
# دیدن مسیر جستجو
LD_DEBUG=libs ./myapp 2>&1 | head

# دیدن همه
LD_DEBUG=all ./myapp 2>&1 | head
```

**ابزارهای تحلیل:**

```bash
# نمادهای یک کتابخانه
nm -D /lib/x86_64-linux-gnu/libc.so.6 | head

# اطلاعات ELF
readelf -h /bin/ls

# اندازه
size /bin/ls
```

### مقایسه و الگوهای مختلف

**ابزارهای کار با کتابخانه:**

| ابزار | کاربرد |
|---|---|
| `ldd` | نمایش وابستگی‌های دینامیک |
| `ldconfig` | بازسازی کش کتابخانه |
| `nm` | نمایش symbolها |
| `objdump` | تحلیل فایل اجرایی |
| `readelf` | نمایش headerهای ELF |
| `ld.so` / `ld-linux.so` | لودر دینامیک |

**Static در برابر Shared:**

| ویژگی | Static | Shared |
|---|---|---|
| حجم برنامه | بزرگ | کوچک |
| حافظه | زیاد (هر برنامه جدا) | کم (مشترک) |
| به‌روزرسانی | نیاز به کامپایل مجدد | فقط کتابخانه |
| سرعت اجرا | کمی سریع‌تر | کمی کندتر |
| وابستگی | ندارد | دارد |

**روش‌های حل خطای «library not found»:**

| روش | دائمی؟ |
|---|---|
| `LD_LIBRARY_PATH` | خیر |
| افزودن به `ld.so.conf` | بله |
| نصب پکیج کتابخانه | بله |
| ساخت symlink | بله |

### اشکال شایع

**اشکال ۱: خطای `error while loading shared libraries`.**
کتابخانه در مسیر جستجو نیست. راه‌حل: افزودن به `ld.so.conf` و `ldconfig`.

**اشکال ۲: فراموش کردن `ldconfig`.**
بعد از افزودن مسیر، بدون `ldconfig` اعمال نمی‌شود. راه‌حل: همیشه `ldconfig` بزنید.

**اشکال ۳: تنظیم `LD_LIBRARY_PATH` اشتباه.**
فقط برای تست موقت؛ برای دائمی از `ld.so.conf` استفاده کنید. راه‌حل: مسیر دائمی.

**اشکال ۴: نصب کتابخانه از منبع نامعتبر.**
می‌تواند امنیت را به خطر بیندازد. راه‌حل: از مخازن رسمی نصب کنید.

**اشکال ۵: نسخهٔ اشتباه کتابخانه.**
برنامه به `libssl.so.1.1` نیاز دارد اما `libssl.so.3` نصب است. راه‌حل: نسخهٔ درست را نصب کنید.

**اشکال ۶: اجرای `ldd` روی فایل نامعتبر.**
خطر امنیتی (کد اجرا می‌شود). راه‌حل: فقط روی فایل‌های مطمئن.

### نکات مهم (Tips & Tricks)

- **`ldd` را روی فایل‌های نامعتبر اجرا نکنید** (خطر امنیتی).
- **برای دیباگ، `LD_DEBUG=libs ./program` مسیر جستجو را نشان می‌دهد.**
- **`ldconfig -v` مسیرهای اسکن‌شده را نشان می‌دهد.**
- **کتابخانه‌های سفارشی را در `/usr/local/lib/` بگذارید.**
- **در ۶۴-بیتی، `/lib64/` و `/usr/lib64/` برای کتابخانه‌های ۶۴-بیتی است.**
- **`ldconfig -p` همهٔ کتابخانه‌های کش‌شده را نشان می‌دهد.**
- **`/etc/ld.so.conf.d/` بهتر از ویرایش `ld.so.conf` است.**
- **`ldconfig` را بعد از هر نصب کتابخانه اجرا کنید.**
- **`objdump -T` نمادهای دینامیک را نشان می‌دهد.**
- **`ldd -r` خطاهای نماد حل‌نشده را نشان می‌دهد.**

### خلاصه فصل

کتابخانه‌های اشتراکی (`.so`) کد مشترک را در حافظه به اشتراک می‌گذارند. `ldd` وابستگی‌ها را نشان می‌دهد و `ldconfig` کش مسیرها را می‌سازد. مسیرهای اضافی را در `/etc/ld.so.conf.d/` بگذارید و با `ldconfig` اعمال کنید. خطای «library not found» معمولاً با افزودن مسیر و اجرای `ldconfig` حل می‌شود. `LD_LIBRARY_PATH` موقت است؛ برای دائمی از `ld.so.conf` استفاده کنید.

---

## فصل ۱۷: مدیریت پکیج Debian (dpkg/apt)

### مقدمه

نصب نرم‌افزار در لینوکس با «پکیج» انجام می‌شود. تصور کنید می‌خواهید یک وب‌سرور نصب کنید — با یک دستور (`apt install nginx`) انجام می‌شود. اما پشت صحنه، سیستم وابستگی‌ها را حل می‌کند، مخزن را جستجو می‌کند و فایل‌ها را می‌نصبند.

خانوادهٔ Debian (شامل Ubuntu) از دو ابزار اصلی برای مدیریت پکیج استفاده می‌کند: **dpkg** (سطح پایین) و **apt** (سطح بالا). فهم هر دو برای آزمون LPIC ضروری است.

### مفاهیم اساسی

**تفاوت dpkg و apt:**

| ابزار | سطح | کار | حل وابستگی؟ |
|---|---|---|---|
| **dpkg** | پایین | نصب/حذف فایل `.deb`، مدیریت محلی | خیر |
| **apt** | بالا | دریافت از مخزن، حل وابستگی | بله |

**مخازن (Repositories):**

مخازن محل ذخیرهٔ پکیج‌ها هستند. پیکربندی در:
- `/etc/apt/sources.list` (سنتی)
- `/etc/apt/sources.list.d/` (جدید، deb822)

**ساختار یک مخزن:**

```
deb http://deb.debian.org/debian bookworm main contrib non-free
│   │                             │        │    │
│   │                             │        │    └── components
│   │                             │        └── release
│   │                             └── URL
│   └── نوع (deb=sourced, deb-src)
```

**فایل‌های مهم:**

| فایل | محتوا |
|---|---|
| `/var/lib/dpkg/status` | وضعیت پکیج‌های نصب‌شده |
| `/var/lib/apt/lists/` | فهرست پکیج‌های مخازن |
| `/var/cache/apt/archives/` | پکیج‌های دانلودشده |
| `/var/log/dpkg.log` | لاگ نصب |
| `/etc/apt/apt.conf.d/` | پیکربندی apt |

### مثال‌های عملی

**کار با apt:**

```bash
# به‌روزرسانی فهرست پکیج‌ها
sudo apt update

# ارتقای پکیج‌های نصب‌شده
sudo apt upgrade

# ارتقای کامل (شامل حذف پکیج‌های مشکل‌ساز)
sudo apt full-upgrade    # یا dist-upgrade

# نصب
sudo apt install nginx

# نصب نسخهٔ خاص
sudo apt install nginx=1.18.0-0ubuntu1

# حذف (نگه‌داشتن پیکربندی)
sudo apt remove nginx

# حذف کامل (شامل پیکربندی)
sudo apt purge nginx

# حذف پکیج‌های بی‌استفاده
sudo apt autoremove

# پاک کردن کش
sudo apt clean
sudo apt autoclean

# جستجو
apt search nginx

# نمایش اطلاعات
apt show nginx

# فهرست پکیج‌های نصب‌شده
apt list --installed

# فهرست قابل ارتقا
apt list --upgradable

# نسخه‌های موجود
apt-cache policy nginx
```

**کار با dpkg:**

```bash
# نصب یک فایل .deb محلی
sudo dpkg -i package.deb

# حل وابستگی‌های شکسته
sudo apt install -f

# حذف
sudo dpkg -r package

# حذف کامل
sudo dpkg -P package

# فهرست فایل‌های یک پکیج نصب‌شده
dpkg -L nginx

# پیدا کردن پکیج متعلق به یک فایل
dpkg -S /usr/sbin/nginx

# وضعیت پکیج
dpkg -l | grep nginx

# اطلاعات پکیج
dpkg -s nginx

# پکیج‌های نصب‌شده
dpkg-query -W

# فقط نام
dpkg-query -W -f='${Package}\n'
```

**تغییر مخزن:**

```bash
# Debian: ویرایش sources.list
sudo vim /etc/apt/sources.list

# Ubuntu جدید: deb822
sudo vim /etc/apt/sources.list.d/ubuntu.sources

# افزودن PPA (Ubuntu)
sudo add-apt-repository ppa:user/ppa-name
```

**رفع خطاهای رایج:**

```bash
# قفل dpkg
sudo rm /var/lib/dpkg/lock-frontend
sudo rm /var/lib/apt/lists/lock
sudo dpkg --configure -a

# مخزن خراب
sudo apt update --fix-missing

# بستهٔ نیمه‌نصب
sudo apt --fix-broken install
```

### مقایسه و الگوهای مختلف

**دستورهای معادل apt و dpkg:**

| کار | apt | dpkg |
|---|---|---|
| نصب | `apt install X` | `dpkg -i X.deb` |
| حذف | `apt remove X` | `dpkg -r X` |
| حذف کامل | `apt purge X` | `dpkg -P X` |
| فهرست فایل‌ها | `dpkg -L X` | `dpkg -L X` |
| جستجوی فایل | `apt-file search` | `dpkg -S` |
| اطلاعات | `apt show X` | `dpkg -s X` |
| پاک کردن کش | `apt clean` | — |

**apt در برابر apt-get:**

| ویژگی | apt | apt-get |
|---|---|---|
| رابط | کاربرپسند | اسکریپت‌پسند |
| خروجی | رنگی، پیشرفت | ساده |
| پایدار | کمتر (برای اسکریپت) | بیشتر |
| مناسب | تعامل انسانی | اسکریپت/CI |

**update در برابر upgrade:**

| دستور | کار |
|---|---|
| `apt update` | فقط فهرست را به‌روز می‌کند |
| `apt upgrade` | پکیج‌ها را ارتقا می‌دهد |
| `apt full-upgrade` | ارتقا + حذف/نصب لازم |

### اشکال شایع

**اشکال ۱: `dpkg -i` بدون حل وابستگی.**
اگر وابستگی نباشد، پکیج نیمه‌نصب می‌ماند. راه‌حل: `apt install -f` یا `apt install ./package.deb`.

**اشکال ۲: فراموش کردن `apt update`.**
بدون آن، فهرست پکیج‌ها قدیمی است و پکیج پیدا نمی‌شود. راه‌حل: همیشه `apt update` اول.

**اشکال ۳: قفل فایل dpkg.**
اگر پیام «Could not get lock» دیدید، یعنی apt دیگری در حال اجراست. راه‌حل: صبر کنید یا فرآیند را ببندید.

**اشکال ۴: حذف بدون purge.**
فایل‌های پیکربندی باقی می‌مانند. راه‌حل: برای حذف کامل `purge` استفاده کنید.

**اشکال ۵: مخزن اشتباه.**
ممکن است پکیج ناسازگار نصب شود. راه‌حل: فقط مخازن معتبر.

**اشکال ۶: نادیده گرفتن `autoremove`.**
پکیج‌های بی‌استفاده فضا می‌گیرند. راه‌حل: دوره‌ای `autoremove`.

### نکات مهم (Tips & Tricks)

- **همیشه قبل از نصب، `apt update` بزنید.**
- **`apt list --upgradable` پکیج‌های قابل ارتقا را نشان می‌دهد.**
- **`apt-cache policy X` نسخه‌های موجود را نشان می‌دهد.**
- **`apt-get` و `apt` تفاوت‌های جزئی دارند؛ `apt` برای تعامل انسانی بهتر است.**
- **لاگ نصب در `/var/log/dpkg.log` و `/var/log/apt/history.log` است.**
- **`apt-mark hold X` پکیج را از ارتقا نگه می‌دارد.**
- **`apt-mark showhold` پکیج‌های نگه‌داشته‌شده را نشان می‌دهد.**
- **`apt download X` فقط دانلود می‌کند بدون نصب.**
- **`apt-file` محتوای پکیج‌های نصب‌نشده را جستجو می‌کند.**
- **`apt --reinstall install X` پکیج را دوباره نصب می‌کند.**

### خلاصه فصل

خانوادهٔ Debian از dpkg (سطح پایین) و apt (سطح بالا) استفاده می‌کند. apt وابستگی‌ها را خودکار حل می‌کند؛ dpkg برای فایل‌های `.deb` محلی است. دستورهای کلیدی: `apt update/install/remove/purge/autoremove` و `dpkg -i/-r/-P/-L/-S`. مخازن در `/etc/apt/sources.list` و `sources.list.d/` هستند. همیشه قبل از نصب `apt update` بزنید و برای حذف کامل `purge` استفاده کنید.

---

## فصل ۱۸: مدیریت پکیج RPM (rpm/yum/dnf)

### مقدمه

خانوادهٔ Red Hat (RHEL، Rocky، Alma، CentOS، Fedora) از **rpm** (سطح پایین) و **yum/dnf** (سطح بالا) استفاده می‌کند. در نسخه‌های جدید، **dnf** جایگزین yum شده است.

آزمون LPIC هر دو خانواده را می‌سنجد. اگر فقط `apt` بلدید، باید `dnf`/`rpm` را هم یاد بگیرید. این فصل ابزارهای این خانواده را پوشش می‌دهد.

### مفاهیم اساسی

**تفاوت rpm و dnf:**

| ابزار | سطح | کار | حل وابستگی؟ |
|---|---|---|---|
| **rpm** | پایین | نصب فایل `.rpm`، مدیریت محلی | خیر |
| **dnf** | بالا | دریافت از مخزن، حل وابستگی | بله |
| **yum** | بالا | نسخهٔ قدیمی‌تر dnf (همچنان alias) | بله |

**مخازن:**

پیکربندی در `/etc/yum.repos.d/*.repo`.

**ساختار یک فایل `.repo`:**

```
[baseos]
name=Rocky Linux $releasever - BaseOS
baseurl=https://dl.rockylinux.org/$contentdir/$releasever/BaseOS/$basearch/os/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-rockyofficial
```

**فایل‌های مهم:**

| فایل | محتوا |
|---|---|
| `/etc/yum.repos.d/` | فایل‌های مخزن |
| `/var/lib/rpm/` | پایگاه دادهٔ RPM |
| `/var/cache/dnf/` | کش |
| `/var/log/dnf.log` | لاگ |

### مثال‌های عملی

**کار با dnf:**

```bash
# جستجو
dnf search nginx

# اطلاعات
dnf info nginx

# نصب
sudo dnf install nginx

# نصب گروه
sudo dnf group install "Development Tools"

# حذف
sudo dnf remove nginx

# به‌روزرسانی همه
sudo dnf update

# فقط بررسی به‌روزرسانی
dnf check-update

# فهرست نصب‌شده
dnf list installed

# فهرست موجود
dnf list available

# تاریخچهٔ تراکنش‌ها
dnf history

# اطلاعات یک تراکنش
dnf history info <ID>

# بازگردانی یک تراکنش
sudo dnf history undo <ID>

# گروه‌ها
dnf group list

# مخازن
dnf repolist

# پکیج‌های ارائه‌دهندهٔ یک فایل
dnf provides /usr/bin/nginx

# پاک‌سازی کش
sudo dnf clean all
```

**کار با rpm:**

```bash
# نصب
sudo rpm -ivh package.rpm

# ارتقا
sudo rpm -Uvh package.rpm

# حذف
sudo rpm -e package

# فهرست فایل‌ها
rpm -ql nginx

# پیدا کردن پکیج یک فایل
rpm -qf /usr/sbin/nginx

# اطلاعات
rpm -qi nginx

# بررسی امضای پکیج
rpm --checksig package.rpm

# فهرست همهٔ پکیج‌ها
rpm -qa

# بررسی تغییرات فایل‌ها
rpm -Va

# وابستگی‌ها
rpm -qR nginx
```

**افزودن مخزن:**

```bash
# EPEL (Extra Packages for Enterprise Linux)
sudo dnf install epel-release

# مخزن دستی
sudo tee /etc/yum.repos.d/myrepo.repo <<'EOF'
[myrepo]
name=My Repository
baseurl=https://example.com/repo
enabled=1
gpgcheck=0
EOF

# بازخوانی
sudo dnf makecache
```

**رفع خطا:**

```bash
# پاک‌سازی کش
sudo dnf clean all
sudo dnf makecache

# بازسازی پایگاه دادهٔ RPM
sudo rpm --rebuilddb

# حل وابستگی شکسته
sudo dnf install --skip-broken
```

### مقایسه و الگوهای مختلف

**دستورهای معادل دو خانواده:**

| کار | Debian (apt) | Red Hat (dnf) |
|---|---|---|
| نصب | `apt install X` | `dnf install X` |
| حذف | `apt remove X` | `dnf remove X` |
| جستجو | `apt search X` | `dnf search X` |
| به‌روزرسانی فهرست | `apt update` | `dnf check-update` |
| ارتقای همه | `apt upgrade` | `dnf update` |
| اطلاعات | `apt show X` | `dnf info X` |
| فهرست نصب‌شده | `apt list --installed` | `dnf list installed` |
| تاریخچه | `/var/log/apt/` | `dnf history` |
| گروه | `tasksel` | `dnf group` |
| ارائه‌دهندهٔ فایل | `apt-file search` | `dnf provides` |

**dnf در برابر yum:**

| ویژگی | dnf | yum |
|---|---|---|
| نسخه | جدید | قدیمی |
| کارایی | سریع‌تر | کندتر |
| وابستگی | libsolv | قدیمی |
| پایتون | ۳ | ۲ |
| سازگاری | yum alias به dnf | — |

**rpm در برابر dpkg:**

| ویژگی | rpm | dpkg |
|---|---|---|
| فهرست فایل | `rpm -ql` | `dpkg -L` |
| مالک فایل | `rpm -qf` | `dpkg -S` |
| اطلاعات | `rpm -qi` | `dpkg -s` |
| بررسی تغییرات | `rpm -Va` | `debsums` |

### اشکال شایع

**اشکال ۱: `rpm -ivh` بدون حل وابستگی.**
مثل dpkg، پکیج نیمه‌نصب می‌ماند. راه‌حل: از `dnf install ./package.rpm` استفاده کنید.

**اشکال ۲: مخلوط کردن rpm و dnf.**
همیشه از dnf برای نصب معمولی استفاده کنید. راه‌حل: dnf.

**اشکال ۳: نبود مخزن EPEL.**
بعضی پکیج‌ها در مخزن اصلی نیستند. راه‌حل: `dnf install epel-release`.

**اشکال ۴: اشکال در SELinux بعد از نصب.**
در RHEL باید context را بررسی کنید. راه‌حل: `restorecon`.

**اشکال ۵: پایگاه دادهٔ خراب RPM.**
dnf کار نمی‌کند. راه‌حل: `rpm --rebuilddb`.

**اشکال ۶: کش قدیمی.**
پکیج جدید پیدا نمی‌شود. راه‌حل: `dnf clean all && dnf makecache`.

### نکات مهم (Tips & Tricks)

- **`dnf provides <file>` پیدا می‌کند کدام پکیج فایل را دارد.**
- **EPEL را با `dnf install epel-release` اضافه کنید.**
- **`dnf history info <ID>` جزئیات یک تراکنش را نشان می‌دهد.**
- **`dnf clean all` کش را پاک می‌کند.**
- **`rpm -Va` فایل‌های تغییر‌یافتهٔ پکیج‌ها را بررسی می‌کند.**
- **`dnf history undo` تراکنش را برمی‌گرداند** (برای برگشت نصب).
- **`dnf module list` ماژول‌های AppStream را نشان می‌دهد** (برای نسخه‌های مختلف).
- **`rpm -q --whatrequires X` وابستگی‌های معکوس را نشان می‌دهد.**
- **`dnf repoquery` اطلاعات پیشرفته می‌دهد.**
- **`dnf update --security` فقط به‌روزرسانی‌های امنیتی را نصب می‌کند.**

### خلاصه فصل

خانوادهٔ Red Hat از rpm (سطح پایین) و dnf (سطح بالا) استفاده می‌کند. dnf وابستگی‌ها را خودکار حل می‌کند. دستورهای کلیدی: `dnf install/remove/update/history` و `rpm -ivh/-e/-ql/-qf/-Va`. مخازن در `/etc/yum.repos.d/` هستند و EPEL مخزن مکمل مهمی است. جدول مقایسهٔ apt/dnf را حفظ کنید — در آزمون حیاتی است.

---

## فصل ۱۹: لینوکس به‌عنوان مهمان مجازی‌سازی

### مقدمه

امروزه اکثر سرورها **مجازی (virtual)** هستند. تصور کنید وارد یک شرکت می‌شوید و سرورها روی VMware یا KVM اجرا می‌شوند. یک مدیر سیستم باید بداند سیستم مجازی چطور کار می‌کند، چه ابزارهایی دارد و چطور آن را بهینه کند.

در این فصل، مفاهیم مجازی‌سازی، تشخیص محیط مجازی و بهینه‌سازی مهمان را یاد می‌گیرید.

### مفاهیم اساسی

**دو نوع مجازی‌سازی:**

- **Full Virtualization:** سخت‌افزار کامل شبیه‌سازی می‌شود (KVM، VMware). مهمان نمی‌داند مجازی است.
- **Paravirtualization:** مهمان می‌داند مجازی است و مستقیم با hypervisor کار می‌کند (بهینه‌تر).

**Hypervisor چیست؟**

لایه‌ای که ماشین‌های مجازی را مدیریت می‌کند:

| نوع | توضیح | مثال |
|---|---|---|
| **Type 1 (Bare-metal)** | مستقیم روی سخت‌افزار | KVM, Xen, ESXi |
| **Type 2 (Hosted)** | روی سیستم‌عامل | VirtualBox, VMware Workstation |

**virtio:** مجموعه درایورهای بهینه برای مهمان لینوکس (شبکه، دیسک). بسیار سریع‌تر از شبیه‌سازی کامل.

**نام دستگاه‌ها بر اساس فناوری:**

| دیسک | نوع |
|---|---|
| `/dev/sda` | SATA/SCSI مجازی |
| `/dev/vda` | virtio (KVM) |
| `/dev/xvda` | Xen |
| `/dev/nvme0n1` | NVMe |

### مثال‌های عملی

**تشخیص محیط مجازی:**

```bash
# بررسی مجازی بودن
systemd-detect-virt
# خروجی: kvm, oracle, vmware, docker, none

# اطلاعات سخت‌افزار مجازی
sudo dmidecode -s system-manufacturer
sudo dmidecode -s system-product-name

# دستگاه‌های virtio
lsblk | grep vd      # vda = virtio disk
ip link | grep ens    # ممکن است virtio باشد

# بررسی /proc
cat /proc/cpuinfo | grep hypervisor
```

**نصب ابزارهای مهمان:**

```bash
# Debian/Ubuntu
sudo apt install qemu-guest-agent

# RHEL/Rocky
sudo dnf install qemu-guest-agent

# فعال‌سازی
sudo systemctl enable --now qemu-guest-agent

# برای VMware
sudo apt install open-vm-tools

# برای VirtualBox
sudo apt install virtualbox-guest-utils
```

**بهینه‌سازی:**

```bash
# بررسی درایور دیسک
lsblk -o NAME,ROTA,SIZE
# ROTA=0 یعنی SSD، ROTA=1 یعنی HDD

# بررسی رابط شبکه
ip -br link

# بررسی حافظه
free -h

# بررسی CPU
lscpu | grep -i hypervisor
```

### مقایسه و الگوهای مختلف

**فناوری‌های مجازی‌سازی:**

| فناوری | نوع | مزیت | معایب |
|---|---|---|---|
| **KVM** | Type 1 (داخل کرنل) | متن‌باز، سریع | نیاز به پشتیبانی CPU |
| **Xen** | Type 1 | ایزوله قوی | پیچیده |
| **VirtualBox** | Type 2 | ساده، رایگان | کندتر |
| **VMware** | Type 1/2 | پایدار، تجاری | هزینه |
| **Docker** | کانتینر | سبک، سریع | کرنل مشترک |

**ابزارهای مهمان:**

| Hypervisor | ابزار |
|---|---|
| KVM/QEMU | qemu-guest-agent |
| VMware | open-vm-tools |
| VirtualBox | virtualbox-guest-utils |
| Xen | xe-guest-utilities |
| Hyper-V | linux-azure |

**virtio در برابر emulated:**

| ویژگی | virtio | emulated |
|---|---|---|
| سرعت | عالی | کند |
| CPU | کم | زیاد |
| سازگاری | نیاز به درایور | همه‌جا |

### اشکال شایع

**اشکال ۱: نبود درایور virtio.**
کارایی پایین می‌شود. راه‌حل: در تنظیمات VM virtio را فعال کنید.

**اشکال ۲: فراموش کردن guest agent.**
ابزارهای مدیریت (شات‌داون تمیز، sync زمان) کار نمی‌کنند. راه‌حل: qemu-guest-agent نصب کنید.

**اشکال ۳: اشتباه گرفتن کانتینر با VM.**
کانتینر کرنل مستقل ندارد. راه‌حل: تفاوت را بشناسید.

**اشکال ۴: تنظیم زمان اشتباه.**
در VM زمان ممکن است drift کند. راه‌حل: از guest agent یا chrony استفاده کنید.

**اشکال ۵: تشخیص ندادن نوع مجازی‌سازی.**
اشتباه در بهینه‌سازی. راه‌حل: `systemd-detect-virt`.

**اشکال ۶: نصب ابزار اشتباه.**
مثلاً open-vm-tools روی KVM بی‌فایده است. راه‌حل: ابزار متناسب با hypervisor.

### نکات مهم (Tips & Tricks)

- **`systemd-detect-virt` سریع‌ترین راه تشخیص است.**
- **برای کارایی بهتر دیسک/شبکه، از virtio استفاده کنید.**
- **`qemu-guest-agent` را همیشه نصب کنید.**
- **برای sync زمان در VM از `chrony` استفاده کنید.**
- **در ابر (cloud)، از cloud-init برای پیکربندی اولیه استفاده کنید.**
- **`lsblk -o NAME,ROTA` تشخیص SSD از HDD می‌دهد.**
- **`dmidecode -t system` اطلاعات سیستم مجازی می‌دهد.**
- **برای کپی/پیست، Guest Additions/VMware Tools نصب کنید.**
- **`lscpu` وجود hypervisor را نشان می‌دهد.**
- **برای snapshot، از ابزار hypervisor استفاده کنید (نه داخل مهمان).**

### خلاصه فصل

لینوکس اغلب به‌عنوان مهمان مجازی اجرا می‌شود. Hypervisorها دو نوع‌اند (Type 1 و Type 2) و virtio کارایی را بهبود می‌دهد. با `systemd-detect-virt` محیط را تشخیص دهید و `qemu-guest-agent` (یا معادل) را نصب کنید. نام دیسک‌ها بسته به فناوری متفاوت است (`vda` برای KVM، `sda` برای SATA). برای sync زمان از chrony استفاده کنید.

---

## منابع و مراجع

- man pages: `man fdisk`, `man parted`, `man grub-install`, `man ldd`, `man apt`, `man dnf`, `man rpm`
- Debian Package Management: https://www.debian.org/doc/manuals/debian-faq/pkg-basics.en.html
- RPM Documentation: https://rpm.org/documentation.html
- GNU GRUB Manual: https://www.gnu.org/software/grub/manual/
- LVM and Disk Layout (Red Hat): https://access.redhat.com/documentation/

**بعدی:** [فایل ۰۵ — دستورات GNU/Unix — بخش اول](05-GNU-Unix-Commands-Part1.md)
