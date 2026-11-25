# فایل ۰۳ — معماری سیستم

**مدت یادگیری**: ۶ ساعت
**سطح سختی**: مبتدی–متوسط
**پیش‌نیاز**: فایل ۰۲ (آماده‌سازی آزمایشگاه)

---

## فهرست محتوا

- [فصل ۱۰: تنظیمات سخت‌افزار و پیکربندی](#فصل-۱۰-تنظیمات-سختافزار-و-پیکربندی)
- [فصل ۱۱: بوت سیستم](#فصل-۱۱-بوت-سیستم)
- [فصل ۱۲: Runlevel / Boot Target و خاموش/ری‌استارت](#فصل-۱۲-runlevel--boot-target-و-خاموشریاستارت)
- [فصل ۱۳: آزمایشگاه: تحلیل زندهٔ فرآیند بوت و systemd](#فصل-۱۳-آزمایشگاه-تحلیل-زندهٔ-فرآیند-بوت-و-systemd)

---

## فصل ۱۰: تنظیمات سخت‌افزار و پیکربندی

### مقدمه

اولین قدم در درک لینوکس، فهمیدن این است که سیستم‌عامل چطور با سخت‌افزار ارتباط می‌گیرد. تصور کنید یک مدیر سیستم باید بفهمد چرا یک کارت شبکه کار نمی‌کند، یا چرا یک دیسک شناسایی نمی‌شود. بدون درک لایهٔ سخت‌افزار، عیب‌یابی تقریباً غیرممکن است.

در این فصل یاد می‌گیرید که سیستم چطور دستگاه‌ها را تشخیص می‌دهد، اطلاعات سخت‌افزاری را از کجا بخوانید، و ماژول‌های کرنل چطور مدیریت می‌شوند. این دانش پایهٔ تمام مباحث بعدی (دیسک، شبکه، بوت) است.

### مفاهیم اساسی

**کرنل و سخت‌افزار:**

کرنل لینوکس قلب سیستم است. وقتی سیستمی روشن می‌شود، کرنل سخت‌افزار را تشخیص می‌دهد و **درایور** مربوطه را بارگذاری می‌کند. درایور یک قطعهٔ کد است که به کرنل می‌گوید چطور با یک دستگاه خاص ارتباط بگیرد.

**سه مفهوم کلیدی:**

۱. **Device Files (فایل‌های دستگاه):** در لینوکس «همه‌چیز یک فایل است». سخت‌افزارها در `/dev/` به‌صورت فایل نمایش داده می‌شوند. مثلاً `/dev/sda` دیسک اول است.

۲. **Modules (ماژول‌ها):** درایورهایی که می‌توانند در زمان اجرا بارگذاری/حذف شوند (مثل `usb_storage`). این انعطاف باعث می‌شود کرنل کوچک بماند و فقط درایورهای لازم بارگذاری شوند.

۳. **udev:** سیستم مدیریت دستگاه که فایل‌های `/dev/` را خودکار می‌سازد. وقتی یک USB وصل می‌کنید، udev فایل دستگاه را می‌سازد.

**مسیرهای مهم:**

| مسیر | محتوا | نوع |
|---|---|---|
| `/proc/` | اطلاعات کرنل و فرآیندها | مجازی (RAM) |
| `/sys/` | اطلاعات دستگاه‌ها و درایورها | مجازی (RAM) |
| `/dev/` | فایل‌های دستگاه | واقعی (udev) |
| `/proc/cpuinfo` | اطلاعات CPU | مجازی |
| `/proc/meminfo` | اطلاعات حافظه | مجازی |
| `/proc/modules` | ماژول‌های بارگذاری‌شده | مجازی |

**انواع فایل دستگاه:**

| نوع | نمایش در ls | توضیح | مثال |
|---|---|---|---|
| **Block** | `b` | دسترسی بلوکی (دیسک) | `/dev/sda` |
| **Character** | `c` | دسترسی کاراکتری (ترمینال) | `/dev/tty0` |

**نام‌گذاری دیسک‌ها:**

| نام | نوع |
|---|---|
| `/dev/sda`, `/dev/sdb` | SATA/SCSI/USB |
| `/dev/vda`, `/dev/vdb` | virtio (KVM) |
| `/dev/xvda` | Xen |
| `/dev/nvme0n1` | NVMe |
| `/dev/sda1` | پارتیشن اول از sda |

### مثال‌های عملی

**دیدن اطلاعات CPU:**

```bash
# از /proc
cat /proc/cpuinfo

# با ابزار
lscpu
# خروجی: Architecture, CPU(s), Thread(s), Core(s), Model name, ...

# تعداد هسته
nproc
```

**دیدن اطلاعات حافظه:**

```bash
cat /proc/meminfo
free -h
# خروجی: total, used, free, shared, buff/cache, available
```

**دیدن دستگاه‌های PCI:**

```bash
lspci
# خروجی: کارت گرافیک، کارت شبکه، کنترلر USB و...

lspci -v    # با جزئیات
lspci -k    # با درایور در حال استفاده
```

**دیدن دستگاه‌های USB:**

```bash
lsusb
lsusb -v    # جزئیات (نیاز به ریشه)
```

**دیدن دستگاه‌های بلوکی:**

```bash
lsblk
# خروجی درختی: NAME, SIZE, TYPE, MOUNTPOINT

lsblk -f    # با فایل‌سیستم
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT
```

**دیدن اطلاعات دیسک:**

```bash
sudo fdisk -l
sudo parted -l
```

**مدیریت ماژول‌های کرنل:**

```bash
# فهرست ماژول‌های بارگذاری‌شده
lsmod
# خروجی: Module, Size, Used by

# اطلاعات یک ماژول
modinfo usb_storage
# خروجی: filename, description, author, license, parameters

# بارگذاری ماژول
sudo modprobe usb_storage

# حذف ماژول
sudo modprobe -r usb_storage

# بارگذاری با پارامتر
sudo modprobe usbcore nousb=0

# بررسی وابستگی‌ها
modprobe --show-depends ext4
```

**افزودن ماژول به بوت (دائمی):**

```bash
# روش ۱: /etc/modules-load.d/
echo "usb_storage" | sudo tee /etc/modules-load.d/usb.conf

# روش ۲: /etc/modules (Debian قدیمی)
echo "usb_storage" | sudo tee -a /etc/modules

# تنظیم پارامتر ماژول
echo "options usbcore nousb=0" | sudo tee /etc/modprobe.d/usb.conf
```

**پیام‌های کرنل (سخت‌افزار):**

```bash
dmesg | less
dmesg -T              # با زمان خوانا
dmesg | grep -i usb   # فقط USB
dmesg | grep -i error # فقط خطاها
```

### مقایسه و الگوهای مختلف

**روش‌های دیدن سخت‌افزار:**

| ابزار | کاربرد | نیاز به نصب |
|---|---|---|
| `lspci` | دستگاه‌های PCI | pciutils |
| `lsusb` | دستگاه‌های USB | usbutils |
| `lscpu` | اطلاعات CPU | util-linux |
| `lsblk` | دستگاه‌های بلوکی | util-linux |
| `dmidecode` | اطلاعات BIOS/سخت‌افزار | نیاز به ریشه |
| `hwinfo` | گزارش جامع | hwinfo |
| `lshw` | گزارش کامل | lshw |
| `inxi` | خلاصهٔ سیستم | inxi |

**Block در برابر Character device:**

| نوع | دسترسی | مثال | کاربرد |
|---|---|---|---|
| Block | بلوکی (بافر) | دیسک | فایل‌سیستم |
| Character | کاراکتری (استریم) | ترمینال، ماوس | ورودی/خروجی |

**insmod در برابر modprobe:**

| ابزار | وابستگی | مسیر |
|---|---|---|
| `insmod` | حل نمی‌کند | مسیر کامل فایل |
| `modprobe` | حل می‌کند | نام ماژول |

**توصیه:** همیشه `modprobe` (نه `insmod`).

### اشکال شایع

**اشکال ۱: گم شدن در `/proc` و `/sys`.**
این‌ها فایل‌های **مجازی** هستند؛ ویرایش دستی آن‌ها معمولاً اشتباه است و ممکن است سیستم را ناپایدار کند. راه‌حل: فقط بخوانید، از ابزارهای رسمی برای تغییر استفاده کنید.

**اشکال ۲: فراموش کردن `sudo` برای `modprobe`.**
بارگذاری ماژول نیاز به دسترسی ریشه دارد. راه‌حل: با `sudo` اجرا کنید.

**اشکال ۳: حذف ماژول در حال استفاده.**
اگر ماژول در حال استفاده باشد، `modprobe -r` خطا می‌دهد. راه‌حل: ابتدا سرویس مربوطه را متوقف کنید.

**اشکال ۴: اشتباه گرفتن `/dev/sda` با پارتیشن.**
`sda` کل دیسک است، `sda1` پارتیشن اول. راه‌حل: به شماره دقت کنید.

**اشکال ۵: فراموش کردن `depmod`.**
بعد از نصب ماژول جدید، اگر `modprobe` کار نکرد، `sudo depmod -a` بزنید.

**اشکال ۶: نادیده گرفتن `dmesg`.**
منبع اصلی عیب‌یابی سخت‌افزار است. راه‌حل: عادت کنید `dmesg | grep -i error` بزنید.

### نکات مهم (Tips & Tricks)

- **`lsblk` بهترین ابزار برای دیدن ساختار دیسک است** (درختی و خوانا).
- **`dmesg -T` زمان‌های خوانا نشان می‌دهد.**
- **`lshw -short` گزارش خلاصهٔ سخت‌افزار می‌دهد.**
- **ماژول‌ها را با `modprobe` (نه `insmod`) بارگذاری کنید** تا وابستگی‌ها خودکار حل شوند.
- **تنظیمات دائمی ماژول را در `/etc/modules-load.d/` بگذارید.**
- **`lsmod | head` را ببینید تا بفهمید چه ماژول‌هایی بارگذاری شده‌اند.**
- **`/sys/class/net/` رابط‌های شبکه را نشان می‌دهد.**
- **`/dev/disk/by-uuid/` لینک‌های UUID دیسک‌هاست** (برای fstab مفید).
- **`udevadm info /dev/sda` جزئیات یک دستگاه را نشان می‌دهد.**
- **برای سخت‌افزار مجازی، `systemd-detect-virt` نوع را نشان می‌دهد.**

### خلاصه فصل

لینوکس سخت‌افزار را از طریق کرنل، ماژول‌ها و فایل‌های دستگاه مدیریت می‌کند. `/proc` و `/sys` اطلاعات سیستم را در قالب فایل‌های مجازی نشان می‌دهند. ابزارهای `lspci`, `lsusb`, `lscpu`, `lsblk` برای دیدن سخت‌افزار و `lsmod`, `modprobe`, `modinfo` برای مدیریت ماژول‌ها به‌کار می‌روند. `/dev/` فایل‌های دستگاه (block و character) را دارد. `dmesg` منبع اصلی عیب‌یابی سخت‌افزار است.

---

## فصل ۱۱: بوت سیستم

### مقدمه

وقتی دکمهٔ پاور را می‌زنید تا وقتی وارد سیستم می‌شوید، یک زنجیرهٔ پیچیده از رویدادها رخ می‌دهد. تصور کنید سروری بوت نمی‌شود و باید بفهمید کجای زنجیره شکسته است. بدون درک این زنجیره، عیب‌یابی کورکورانه است.

فهم فرآیند بوت برای عیب‌یابی و مدیریت سیستم حیاتی است. در این فصل کل فرآیند بوت را باز می‌کنیم — از BIOS/UEFI تا systemd.

### مفاهیم اساسی

**زنجیرهٔ بوت (Boot Sequence) در سیستم‌های مدرن:**

```
۱. BIOS/UEFI → POST (Power-On Self-Test)
   - تست سخت‌افزار (CPU، RAM، دیسک)
   - اگر خطا باشد، بیپ یا پیام خطا

۲. BIOS/UEFI → انتخاب دستگاه بوت
   - ترتیب بوت (Boot Order): دیسک، USB، شبکه
   - انتخاب بوت‌لودر

۳. Bootloader (GRUB2) → نمایش منو، بارگذاری کرنل
   - کاربر می‌تواند سیستم‌عامل انتخاب کند
   - پارامترهای کرنل قابل ویرایش

۴. Kernel → بارگذاری در حافظه، تشخیص سخت‌افزار
   - کرنل به RAM بارگذاری می‌شود
   - سخت‌افزار تشخیص داده می‌شود

۵. initramfs → بارگذاری درایورهای لازم
   - سیستم فایل موقت با درایورهای اولیه
   - برای mount کردن ریشه واقعی

۶. Kernel → اجرای init (که در systemd معمولاً /sbin/init است)
   - انتقال کنترل به userspace

۷. systemd → رسیدن به target پیش‌فرض (مثلاً multi-user)
   - سرویس‌ها شروع می‌شوند
   - کاربر وارد می‌شود
```

**تفاوت BIOS و UEFI:**

| ویژگی | BIOS (قدیمی) | UEFI (جدید) |
|---|---|---|
| نوع فریمور | Legacy | مدرن |
| پارتیشن بوت | MBR | GPT + EFI System Partition |
| محدودیت دیسک | ۲ ترابایت | بدون محدودیت عملی |
| تعداد پارتیشن | ۴ اصلی | ۱۲۸ |
| سرعت بوت | کندتر | سریع‌تر |
| Secure Boot | ندارد | دارد |
| رابط | متنی | گرافیکی |

**مفاهیم کلیدی:**

- **POST:** تست خودکار سخت‌افزار در شروع.
- **MBR:** Master Boot Record (۵۱۲ بایت اول دیسک).
- **EFI System Partition (ESP):** پارتیشن کوچک FAT32 برای UEFI.
- **initramfs/initrd:** فایل‌سیستم موقت اولیه.
- **GRUB2:** بوت‌لودر استاندارد.

### مثال‌های عملی

**دیدن پیام‌های بوت:**

```bash
# پیام‌های کرنل این جلسه
dmesg
dmesg -T              # با زمان خوانا

# پیام‌های بوت این جلسه (systemd)
journalctl -b

# بوت‌های قبلی
journalctl --list-boots

# بوت قبلی
journalctl -b -1

# فقط خطاهای بوت
journalctl -b -p err
```

**دیدن اطلاعات بوت‌لودر:**

```bash
# فایل پیکربندی GRUB (تولیدشده — ویرایش نکنید)
cat /boot/grub/grub.cfg      # Debian/Ubuntu
cat /boot/grub2/grub.cfg     # RHEL

# ویرایش از اینجا:
sudo vim /etc/default/grub

# بازتولید پیکربندی
sudo update-grub              # Debian/Ubuntu
sudo grub2-mkconfig -o /boot/grub2/grub.cfg   # RHEL
```

**دیدن پارامترهای کرنل:**

```bash
cat /proc/cmdline
# خروجی: BOOT_IMAGE=/vmlinuz-... root=UUID=... ro quiet splash
```

**محتوای /etc/default/grub:**

```bash
GRUB_DEFAULT=0
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
GRUB_CMDLINE_LINUX=""
```

**دیدن فایل‌های بوت:**

```bash
ls -lh /boot/
# vmlinuz-*      ← کرنل
# initrd.img-*   ← initramfs
# config-*       ← پیکربندی کرنل
# System.map-*   ← نقشهٔ نمادها
```

### مقایسه و الگوهای مختلف

**مدیران بوت:**

| مدیر بوت | کاربرد | رابط |
|---|---|---|
| **GRUB2** | پیش‌فرض اکثر توزیع‌ها | منو متنی/گرافیکی |
| **systemd-boot** | سبک، برای UEFI | منو ساده |
| **LILO** | قدیمی (تاریخی) | — |
| **rEFInd** | UEFI زیبا | گرافیکی |

**پارامترهای مفید کرنل در خط بوت:**

| پارامتر | اثر |
|---|---|
| `single` / `1` | بوت به حالت تک‌کاربره |
| `systemd.unit=rescue.target` | بوت به حالت بازیابی |
| `systemd.unit=emergency.target` | بوت حداقلی |
| `quiet` | کاهش پیام‌های بوت |
| `splash` | نمایش گرافیکی بوت |
| `ro` / `rw` | mount فقط‌خواندنی/خواندنی-نوشتنی ریشه |
| `init=/bin/bash` | اجرای bash به‌جای init (بازیابی) |
| `nomodeset` | غیرفعال کردن درایور گرافیکی |

### اشکال شایع

**اشکال ۱: ویرایش مستقیم `grub.cfg`.**
این فایل تولیدشده است؛ با ویرایش بعدی بازنویسی می‌شود. راه‌حل: از `/etc/default/grub` استفاده کنید و `update-grub` بزنید.

**اشکال ۲: فراموش کردن `update-grub`.**
تغییرات `/etc/default/grub` تا بازتولید اعمال نمی‌شود. راه‌حل: بعد از ویرایش، `update-grub`.

**اشکال ۳: از دست دادن رمز ریشه.**
اگر رمز ریشه را فراموش کردید، با حالت تک‌کاربره می‌توانید بازیابی کنید. راه‌حل: در GRUB با `e` ویرایش کنید و `init=/bin/bash` اضافه کنید.

**اشکال ۴: اشتباه گرفتن initramfs با kernel.**
initramfs یک سیستم فایل موقت است که درایورهای اولیه را دارد، نه کرنل. راه‌حل: تفاوت را بشناسید.

**اشکال ۵: خرابی initramfs.**
اگر درایور دیسک در initramfs نباشد، سیستم بوت نمی‌شود (خطای «unable to find root»). راه‌حل: initramfs را بازتولید کنید (`mkinitramfs`/`dracut`).

**اشکال ۶: نبود GRUB.**
اگر MBR خراب شود، سیستم بوت نمی‌شود. راه‌حل: از live CD با `grub-install` تعمیر کنید.

### نکات مهم (Tips & Tricks)

- **`journalctl -b` بهترین راه دیدن پیام‌های بوت است.**
- **برای دیباگ، `quiet` را از پارامترهای کرنل حذف کنید** تا پیام‌ها را ببینید.
- **برای بازیابی رمز ریشه، در GRUB با `e` ویرایش کنید و `init=/bin/bash` اضافه کنید.**
- **`dmesg -T` زمان‌های خوانا نشان می‌دهد.**
- **initramfs را با `mkinitramfs` (Debian) یا `dracut` (RHEL) بازتولید کنید.**
- **`systemd-analyze` زمان بوت را تحلیل می‌کند.**
- **`systemd-analyze blame` سرویس‌های کند را نشان می‌دهد.**
- **`efibootmgr` ترتیب بوت UEFI را مدیریت می‌کند.**
- **همیشه یک live USB به‌عنوان ابزار بازیابی داشته باشید.**
- **`grub-mkconfig -o /boot/grub/grub.cfg` معادل `update-grub` است.**

### خلاصه فصل

فرآیند بوت از BIOS/UEFI → GRUB2 → کرنل → initramfs → systemd می‌گذرد. BIOS از MBR و UEFI از GPT+ESP استفاده می‌کند. GRUB2 از `/etc/default/grub` پیکربندی و با `update-grub` بازتولید می‌شود. پیام‌های بوت با `dmesg` و `journalctl -b` دیده می‌شوند. پارامترهای کرنل (مثل `single`, `init=/bin/bash`) برای بازیابی مفیدند. هرگز `grub.cfg` را مستقیم ویرایش نکنید.

---

## فصل ۱۲: Runlevel / Boot Target و خاموش/ری‌استارت

### مقدمه

لینوکس در هر لحظه در یک «حالت» مشخص اجرا می‌شود: تک‌کاربره (برای تعمیرات)، چندکاربره (بدون گرافیک)، گرافیکی (برای دسکتاپ) و غیره. در سیستم‌های قدیمی این حالت‌ها **Runlevel** نام داشتند و امروز **Target** نامیده می‌شوند.

تصور کنید سرور شما در حالت گرافیکی بوت می‌شود اما شما فضای کمی دارید و می‌خواهید فقط متنی باشد. یا می‌خواهید سیستم را برای تعمیر به حالت تک‌کاربره ببرید. این فصل به شما یاد می‌دهد چطور حالت‌ها را مدیریت کنید.

### مفاهیم اساسی

**نگاشت Runlevel به Target در systemd:**

| Runlevel | Target systemd | معنی |
|---|---|---|
| 0 | poweroff.target | خاموشی |
| 1 | rescue.target | تک‌کاربره (بازیابی) |
| 2 | multi-user.target | چندکاربره (بدون شبکه) |
| 3 | multi-user.target | چندکاربره متنی |
| 4 | multi-user.target | (رزرو) |
| 5 | graphical.target | چندکاربره گرافیکی |
| 6 | reboot.target | ری‌استارت |

**نکته:** در systemd، Runlevelها فقط برای سازگاری نگه داشته شده‌اند. مفهوم اصلی **Target** است.

**systemd چیست؟**

systemd سیستم init و مدیر سرویس‌هاست که در اکثر توزیع‌های مدرن استفاده می‌شود. جایگزین SysV init قدیمی است. مزایا: راه‌اندازی موازی، مدیریت وابستگی، لاگ متمرکز (journald).

**Targetهای اصلی:**

| Target | کاربرد |
|---|---|
| `poweroff.target` | خاموشی |
| `rescue.target` | بازیابی (تک‌کاربره) |
| `emergency.target` | حداقلی (فقط ریشه) |
| `multi-user.target` | چندکاربره متنی |
| `graphical.target` | گرافیکی |
| `reboot.target` | ری‌استارت |

### مثال‌های عملی

**مدیریت targetها:**

```bash
# target فعلی
systemctl get-default

# تغییر target پیش‌فرض به گرافیکی
sudo systemctl set-default graphical.target

# تغییر به متنی
sudo systemctl set-default multi-user.target

# تعویض فوری target (بدون ری‌استارت)
sudo systemctl isolate multi-user.target

# بوت به حالت بازیابی
sudo systemctl isolate rescue.target

# فهرست targetها
systemctl list-units --type=target
```

**خاموش و ری‌استارت:**

```bash
# خاموشی
sudo shutdown -h now
sudo poweroff
sudo systemctl poweroff

# ری‌استارت
sudo shutdown -r now
sudo reboot
sudo systemctl reboot

# خاموشی در ساعت مشخص
sudo shutdown -h 23:00
sudo shutdown -h +10       # ۱۰ دقیقه بعد

# با پیام
sudo shutdown -r +5 "سیستم برای تعمیرات ری‌استارت می‌شود"

# لغو یک shutdown زمان‌بندی‌شده
sudo shutdown -c
```

**بررسی حالت فعلی:**

```bash
# با systemd
systemctl get-default
systemctl list-units --type=target --state=active

# روش قدیمی
runlevel
who -r
```

### مقایسه و الگوهای مختلف

**روش‌های مختلف مدیریت حالت:**

| روش | سیستم | مثال |
|---|---|---|
| `systemctl` | systemd (مدرن) | `systemctl isolate rescue.target` |
| `telinit` | سازگاری | `sudo telinit 3` |
| `init` | سازگاری | `sudo init 3` |
| `runlevel` | نمایش | `runlevel` |
| `who -r` | نمایش | `who -r` |

**نکته:** `telinit` و `init` همچنان در systemd به‌عنوان لایهٔ سازگاری کار می‌کنند، اما روش توصیه‌شده `systemctl` است.

**isolate در برابر set-default:**

| دستور | اثر |
|---|---|
| `isolate` | فوری، فقط این جلسه |
| `set-default` | دائمی، بوت بعدی |

**دستورهای خاموشی:**

| دستور | توضیح |
|---|---|
| `shutdown` | انعطاف‌پذیر (زمان‌بندی، پیام) |
| `poweroff` | خاموشی فوری |
| `halt` | توقف (بدون خاموشی برق) |
| `reboot` | ری‌استارت فوری |

### اشکال شایع

**اشکال ۱: استفاده از `init 3` روی سیستم systemd.**
کار می‌کند ولی روش توصیه‌شده `systemctl isolate` است. راه‌حل: از `systemctl` استفاده کنید.

**اشکال ۲: اشتباه گرفتن `isolate` با `set-default`.**
`isolate` فوری است؛ `set-default` برای بوت بعدی. راه‌حل: هدف را مشخص کنید.

**اشکال ۳: `shutdown` بدون زمان.**
`shutdown` بدون آرگومان در برخی نسخه‌ها نیاز به زمان دارد. راه‌حل: `shutdown -h now` بزنید.

**اشکال ۴: فراموش کردن `sudo`.**
همهٔ این دستورها نیاز به دسترسی ریشه دارند. راه‌حل: `sudo` بزنید.

**اشکال ۵: خاموشی بدون هشدار به کاربران.**
کاربران داده از دست می‌دهند. راه‌حل: از `shutdown +5 "پیام"` استفاده کنید.

**اشکال ۶: بوت ناخواسته به rescue.**
اگر `set-default` اشتباه بزنید، سیستم در حالت بازیابی بوت می‌شود. راه‌حل: با `set-default` درست کنید.

### نکات مهم (Tips & Tricks)

- **`systemctl get-default` سریع‌ترین راه دیدن target فعلی است.**
- **برای بازیابی، از `rescue.target` استفاده کنید.**
- **`shutdown +10 "پیام"` به کاربران هشدار می‌دهد.**
- **`wall` برای ارسال پیام به همهٔ کاربران قبل از خاموشی مفید است.**
- **`systemctl poweroff` و `systemctl reboot` معادل مدرن هستند.**
- **`systemctl isolate` را با احتیاط در سرور استفاده کنید** (سرویس‌ها قطع می‌شوند).
- **`systemctl list-dependencies graphical.target` وابستگی‌ها را نشان می‌دهد.**
- **`emergency.target` حداقلی است** (فقط ریشه mount می‌شود).
- **`systemctl default` به target پیش‌فرض برمی‌گردد.**
- **`last -x | head` تاریخچهٔ خاموشی/ری‌استارت را نشان می‌دهد.**

### خلاصه فصل

حالت‌های سیستم از Runlevel (قدیمی) به Target (systemd) تکامل یافتند. نگاشت کلیدی: 3=multi-user، 5=graphical، 1=rescue، 0=poweroff، 6=reboot. با `systemctl set-default` حالت پیش‌فرض و با `systemctl isolate` حالت فوری را تغییر دهید. خاموش/ری‌استارت با `shutdown`, `reboot`, `poweroff` انجام می‌شود. `shutdown` زمان‌بندی و پیام پشتیبانی می‌کند. `telinit`/`init` همچنان به‌عنوان لایهٔ سازگاری کار می‌کنند.

---

## فصل ۱۳: آزمایشگاه: تحلیل زندهٔ فرآیند بوت و systemd

### مقدمه

در این فصل، همهٔ آنچه در فصل‌های ۱۰ تا ۱۲ یاد گرفتید را در یک آزمایشگاه عملی ترکیب می‌کنیم. هدف: دیدن زنجیرهٔ بوت به‌صورت زنده و درک systemd در عمل.

تصور کنید سروری کند بوت می‌شود و باید بفهمید کدام سرویس مقصر است. یا می‌خواهید یک سرویس سفارشی بسازید که در بوت اجرا شود. این آزمایشگاه دقیقاً همین مهارت‌ها را می‌سازد.

### مفاهیم اساسی

در این آزمایشگاه:
1. فرآیند بوت را از لاگ‌ها تحلیل می‌کنیم.
2. سرویس‌های systemd را بررسی می‌کنیم.
3. یک سرویس سفارشی می‌سازیم.
4. حالت‌های مختلف را تست می‌کنیم.
5. سخت‌افزار و ماژول‌ها را بررسی می‌کنیم.

### مثال‌های عملی

**گام ۱: تحلیل بوت**

```bash
# زمان کل بوت
systemd-analyze
# خروجی: Startup finished in Xs (firmware) + Ys (loader) + Zs (kernel) + Ws (userspace)

# ترتیب سرویس‌ها در بوت (کندترین‌ها)
systemd-analyze blame | head -20

# نمودار وابستگی
systemd-analyze critical-chain

# خروجی گرافیکی (SVG)
systemd-analyze plot > boot.svg

# لاگ بوت کامل
journalctl -b --no-pager | less
```

**گام ۲: بررسی سرویس‌ها**

```bash
# سرویس‌های در حال اجرا
systemctl list-units --type=service --state=running

# سرویس‌های شکست‌خورده
systemctl --failed

# همهٔ سرویس‌ها (فعال یا غیرفعال)
systemctl list-unit-files --type=service

# وضعیت یک سرویس
systemctl status sshd

# آیا در بوت فعال است؟
systemctl is-enabled sshd
```

**گام ۳: ساخت یک سرویس سفارشی**

```bash
# ساخت فایل سرویس
sudo tee /etc/systemd/system/hello.service <<'EOF'
[Unit]
Description=Hello Lab Service
After=network.target

[Service]
Type=oneshot
ExecStart=/bin/echo "Lab service started at boot"
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF

# بارگذاری مجدد (بعد از هر تغییر unit)
sudo systemctl daemon-reload

# فعال‌سازی (اجرا در بوت)
sudo systemctl enable hello.service

# اجرای فوری
sudo systemctl start hello.service

# بررسی
systemctl status hello.service
journalctl -u hello.service

# تست disable
sudo systemctl disable hello.service
```

**گام ۴: تست حالت‌ها**

```bash
# ببینید در کدام target هستید
systemctl get-default

# تعویض موقت (در VM امن است)
sudo systemctl isolate multi-user.target

# بررسی (در VM گرافیکی، باید به ترمینال برگردید)
systemctl get-default

# برگشت به گرافیکی
sudo systemctl isolate graphical.target
```

**گام ۵: بررسی سخت‌افزار و ماژول**

```bash
# سخت‌افزار
lscpu
free -h
lsblk
lspci | head

# ماژول‌ها
lsmod | head -20
modinfo ext4

# پیام‌های کرنل
dmesg -T | head -30
```

### مقایسه و الگوهای مختلف

**انواع سرویس systemd:**

| نوع | توضیح | مثال |
|---|---|---|
| `Type=simple` | فرآیند پیش‌زمینه اجرا می‌شود | وب‌سرور |
| `Type=oneshot` | یک‌بار اجرا و تمام | اسکریپت راه‌اندازی |
| `Type=forking` | فرآیند fork می‌کند | سرویس‌های قدیمی |
| `Type=notify` | با sd_notify اطلاع می‌دهد | سرویس‌های مدرن |
| `Type=idle` | بعد از بقیه | کارهای تأخیری |

**Restart options:**

| مقدار | رفتار |
|---|---|
| `no` | هرگز restart نمی‌شود |
| `on-failure` | فقط در خطا |
| `always` | همیشه |
| `on-abnormal` | در سیگنال/تایم‌اوت |

### اشکال شایع

**اشکال ۱: فراموش کردن `daemon-reload`.**
بعد از ساخت/ویرایش unit، باید `systemctl daemon-reload` بزنید. راه‌حل: بعد از هر تغییر unit، daemon-reload.

**اشکال ۲: `enable` بدون `start`.**
`enable` فقط برای بوت است؛ برای اجرای فوری `start` لازم است. راه‌حل: `systemctl enable --now` هر دو را انجام می‌دهد.

**اشکال ۳: خطای مسیر در `ExecStart`.**
باید مسیر مطلق باشد. راه‌حل: مسیر کامل بنویسید.

**اشکال ۴: نبود بخش `[Install]`.**
بدون آن، `enable` کار نمی‌کند. راه‌حل: `[Install]` با `WantedBy=multi-user.target` اضافه کنید.

**اشکال ۵: وابستگی اشتباه در `After`.**
سرویس زودتر از وابستگی اجرا می‌شود. راه‌حل: `After` و `Requires` را درست تنظیم کنید.

**اشکال ۶: سرویس در `/lib/` به‌جای `/etc/`.**
`/lib/systemd/system/` برای پکیج‌هاست؛ سرویس سفارشی در `/etc/systemd/system/`. راه‌حل: مسیر درست.

### نکات مهم (Tips & Tricks)

- **`systemd-analyze blame` بهترین ابزار برای پیدا کردن سرویس‌های کند بوت است.**
- **برای دیباگ سرویس، `journalctl -u <service> -f` را زنده دنبال کنید.**
- **`systemctl cat <service>` محتوای کامل unit را نشان می‌دهد.**
- **`systemctl edit <service>` برای override بدون ویرایش فایل اصلی است.**
- **سرویس‌های سفارشی را در `/etc/systemd/system/` بگذارید** (نه `/lib/systemd/system/`).
- **`systemctl list-dependencies <service>` وابستگی‌ها را نشان می‌دهد.**
- **`systemd-analyze plot > boot.svg` نمودار بصری بوت می‌دهد.**
- **`systemctl show <service>` همهٔ خواص را نشان می‌دهد.**
- **`systemctl mask <service>` سرویس را کاملاً غیرفعال می‌کند** (حتی اگر وابستگی باشد).
- **`journalctl -b -p err` فقط خطاهای بوت را نشان می‌دهد.**

### خلاصه فصل

در این آزمایشگاه، زنجیرهٔ بوت را با `systemd-analyze` و `journalctl` تحلیل کردیم، سرویس‌ها را بررسی کردیم، یک سرویس سفارشی ساختیم و حالت‌های مختلف را تست کردیم. نکات کلیدی: `daemon-reload` بعد از تغییر، مسیر مطلق در `ExecStart`، وجود بخش `[Install]` برای enable، و قرار دادن سرویس سفارشی در `/etc/systemd/system/`. `systemd-analyze blame` و `journalctl -u` ابزارهای اصلی دیباگ‌اند.

---

## منابع و مراجع

- man pages: `man systemd`, `man systemctl`, `man grub`, `man modprobe`, `man dmesg`, `man lsblk`
- systemd Documentation: https://www.freedesktop.org/wiki/Software/systemd/
- GNU GRUB Manual: https://www.gnu.org/software/grub/manual/
- Kernel Documentation: https://www.kernel.org/doc/html/latest/
- UEFI Specification: https://uefi.org/specifications

**بعدی:** [فایل ۰۴ — نصب و مدیریت پکیج](04-Installation-and-Package-Management.md)
