# فایل ۱۶ — کرنل لینوکس

**مدت یادگیری**: ۶ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: فایل ۰۳ (معماری سیستم)

---

## فهرست محتوا

- [فصل ۵۸: اجزای کرنل](#فصل-۵۸-اجزای-کرنل)
- [فصل ۵۹: کامپایل کرنل](#فصل-۵۹-کامپایل-کرنل)
- [فصل ۶۰: مدیریت و عیب‌یابی زمان اجرای کرنل](#فصل-۶۰-مدیریت-و-عیبیابی-زمان-اجرای-کرنل)

---

## فصل ۵۸: اجزای کرنل

### مقدمه

**کرنل (Kernel)** قلب سیستم‌عامل است — لایه‌ای که بین سخت‌افزار و برنامه‌ها قرار می‌گیرد و منابع را مدیریت می‌کند. تصور کنید یک برنامه می‌خواهد فایل بخواند — درخواست از طریق کرنل به سخت‌افزار می‌رسد.

فهم ساختار کرنل برای مدیریت پیشرفته ضروری است و در آزمون LPIC-2 می‌آید.

### مفاهیم اساسی

**وظایف کرنل:**

- مدیریت فرآیندها (زمان‌بندی).
- مدیریت حافظه.
- مدیریت دستگاه‌ها (درایورها).
- سیستم فایل.
- شبکه.
- امنیت.

**اجزای کرنل:**

| جزء | توضیح |
|---|---|
| **Kernel core** | هستهٔ اصلی |
| **Modules** | درایورهای قابل بارگذاری |
| **initramfs** | فایل‌سیستم موقت اولیه |
| **vmlinuz** | تصویر فشردهٔ کرنل |

**محل فایل‌ها:**

```
/boot/vmlinuz-<version>        ← کرنل
/boot/initrd.img-<version>     ← initramfs
/lib/modules/<version>/        ← ماژول‌ها
/boot/config-<version>         ← پیکربندی کرنل
/boot/System.map-<version>     ← نقشهٔ نمادها
```

**انواع کرنل:**

| نوع | توضیح | مزیت |
|---|---|---|
| **Monolithic** | همه‌چیز در کرنل | سرعت |
| **Microkernel** | حداقلی + سرویس‌ها | پایداری |
| **Hybrid** | ترکیبی | تعادل |

**نکته:** لینوکس monolithic است اما ماژولار پشتیبانی می‌کند.

### مثال‌های عملی

**اطلاعات کرنل:**

```bash
# نسخهٔ کرنل
uname -r
uname -a
uname -m              # معماری

# پارامترهای بوت
cat /proc/cmdline

# ماژول‌های بارگذاری‌شده
lsmod
cat /proc/modules

# پارامترهای کرنل
sysctl -a | less
sysctl kernel.hostname
sysctl -a | grep vm.swappiness
```

**مدیریت ماژول:**

```bash
# اطلاعات ماژول
modinfo ext4
modinfo -p ext4       # پارامترها

# بارگذاری/حذف
sudo modprobe ext4
sudo modprobe -r ext4

# وابستگی‌ها
modprobe --show-depends ext4
depmod -a             # بازسازی وابستگی‌ها

# پارامترهای ماژول
cat /sys/module/ext4/parameters/*
```

**پارامترهای sysctl:**

```bash
# دیدن
sysctl net.ipv4.ip_forward
sysctl -a

# تنظیم موقت
sudo sysctl -w net.ipv4.ip_forward=1

# تنظیم دائمی
echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/99-custom.conf
sudo sysctl -p /etc/sysctl.d/99-custom.conf

# پارامترهای مهم
# net.ipv4.ip_forward=1        ← routing
# vm.swappiness=10             ← تمایل به swap
# net.core.somaxconn=1024      ← backlog
# fs.file-max=100000           ← فایل باز
```

**مدیریت ماژول دائمی:**

```bash
# ماژول در بوت
echo "usb_storage" | sudo tee /etc/modules-load.d/usb.conf

# پارامتر ماژول
echo "options usbcore nousb=0" | sudo tee /etc/modprobe.d/usb.conf

# بلاک کردن ماژول
echo "blacklist floppy" | sudo tee /etc/modprobe.d/blacklist.conf
```

### مقایسه و الگوهای مختلف

**Monolithic در برابر Modular:**

| نوع | توضیح | مزیت |
|---|---|---|
| **Monolithic** | همه‌چیز در کرنل | سرعت |
| **Modular** | درایورها ماژول | انعطاف، حافظه کم |

**پارامترهای مهم sysctl:**

| پارامتر | کار |
|---|---|
| `net.ipv4.ip_forward` | routing |
| `vm.swappiness` | تمایل به swap (0-100) |
| `vm.dirty_ratio` | درصد dirty قبل از flush |
| `net.core.somaxconn` | backlog اتصالات |
| `fs.file-max` | حداکثر فایل باز |
| `kernel.pid_max` | حداکثر PID |

**پوشه‌های پیکربندی:**

| مسیر | کار |
|---|---|
| `/etc/modules-load.d/` | ماژول‌های بوت |
| `/etc/modprobe.d/` | گزینه‌های ماژول |
| `/etc/sysctl.d/` | پارامترهای کرنل |
| `/etc/sysctl.conf` | قدیمی |

### اشکال شایع

**اشکال ۱: نبود ماژول لازم در initramfs.**
سیستم بوت نمی‌شود (مثلاً دیسک NVMe). راه‌حل: بازتولید initramfs.

**اشکال ۲: بارگذاری دستی بدون نیاز.**
حافظه هدر می‌رود. راه‌حل: فقط لازم‌ها.

**اشکال ۳: پارامتر sysctl اشتباه.**
ممکن است سیستم ناپایدار شود. راه‌حل: مقادیر معتبر.

**اشکال ۴: حذف ماژول در حال استفاده.**
خطا می‌دهد. راه‌حل: سرویس را متوقف کنید.

**اشکال ۵: فراموش کردن `depmod`.**
ماژول جدید پیدا نمی‌شود. راه‌حل: `depmod -a`.

**اشکال ۶: swappiness اشتباه.**
swap بیش از حد/کم. راه‌حل: تنظیم مناسب.

### نکات مهم (Tips & Tricks)

- **`lsmod` ستون «Used by» وابستگی‌ها را نشان می‌دهد.**
- **`modprobe -r` فقط ماژول بی‌استفاده را حذف می‌کند.**
- **`/etc/modules-load.d/` ماژول‌های بوت را تعیین می‌کند.**
- **`/etc/modprobe.d/` گزینه‌های ماژول را تنظیم می‌کند.**
- **`sysctl` را همیشه در فایل `.conf` بنویسید (پایدار).**
- **`/proc/config.gz` پیکربندی کرنل فعلی است (اگر فعال باشد).**
- **`modinfo` پارامترهای ماژول را نشان می‌دهد.**
- **`sysctl -p` فایل را reload می‌کند.**
- **`/sys/module/` پارامترهای زمان اجرا.**
- **`lsmod | wc -l` تعداد ماژول‌ها.**
- **`dmesg | grep module` پیام‌های ماژول.**
- **`blacklist` ماژول را از بارگذاری خودکار منع می‌کند.**

### خلاصه فصل

کرنل قلب سیستم است و فرآیندها، حافظه، دستگاه‌ها، فایل‌سیستم و شبکه را مدیریت می‌کند. فایل‌های اصلی در `/boot/` و ماژول‌ها در `/lib/modules/` هستند. `lsmod`, `modprobe`, `modinfo`, `depmod` برای ماژول‌ها و `sysctl` برای پارامترهای کرنل به‌کار می‌روند. ماژول‌های بوت در `/etc/modules-load.d/` تعیین می‌شوند و پارامترها در `/etc/sysctl.d/`.

---

## فصل ۵۹: کامپایل کرنل

### مقدمه

گاهی نیاز دارید کرنل را خودتان **کامپایل (compile)** کنید — مثلاً برای پشتیبانی از سخت‌افزار خاص، بهینه‌سازی یا آزمایش. تصور کنید یک درایور جدید یا ویژگی آزمایشی می‌خواهید.

در این فصل فرآیند کامپایل را یاد می‌گیرید.

### مفاهیم اساسی

**چرا کامپایل کرنل؟**

- پشتیبانی از سخت‌افزار جدید.
- بهینه‌سازی برای سخت‌افزار خاص.
- حذف ویژگی‌های غیرضروری.
- فعال‌سازی گزینه‌های آزمایشی.
- یادگیری (آزمون).

**مراحل کلی:**

```
۱. دریافت سورس کرنل
۲. پیکربندی (.config)
۳. کامپایل
۴. نصب ماژول‌ها
۵. نصب کرنل
۶. به‌روزرسانی بوت‌لودر
```

**ابزارهای لازم:** `gcc`, `make`, `libncurses`, `flex`, `bison`, `libssl`, `libelf`, `bc`.

### مثال‌های عملی

**دریافت و آماده‌سازی:**

```bash
# نصب ابزارهای لازم
sudo apt install build-essential libncurses-dev flex bison \
  libssl-dev libelf-dev bc

# دریافت سورس از kernel.org
cd /usr/src
sudo wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.6.tar.xz
sudo tar -xf linux-6.6.tar.xz
cd linux-6.6
```

**پیکربندی:**

```bash
# کپی پیکربندی فعلی
cp /boot/config-$(uname -r) .config

# یا منوی تعاملی
make menuconfig

# یا تنظیم پیش‌فرض
make defconfig

# به‌روزرسانی گزینه‌های جدید
make oldconfig

# فقط ماژول‌های فعلی (کرنل کوچک)
make localmodconfig
```

**کامپایل و نصب:**

```bash
# کامپایل (با همهٔ هسته‌ها)
make -j$(nproc)

# نصب ماژول‌ها
sudo make modules_install

# نصب کرنل
sudo make install

# به‌روزرسانی GRUB
sudo update-grub
```

**پاک‌سازی:**

```bash
make clean            # حذف فایل‌های build
make mrproper         # حذف .config هم
```

### مقایسه و الگوهای مختلف

**روش‌های پیکربندی:**

| روش | توضیح |
|---|---|
| `make menuconfig` | منوی متنی تعاملی |
| `make xconfig` | منوی گرافیکی (Qt) |
| `make gconfig` | منوی گرافیکی (GTK) |
| `make oldconfig` | بر پایهٔ config قبلی |
| `make defconfig` | پیش‌فرض |
| `make localmodconfig` | فقط ماژول‌های فعلی |

**زمان کامپایل (تقریبی):**

| تعداد هسته | زمان |
|---|---|
| ۱ | ۲–۳ ساعت |
| ۴ | ۳۰–۴۵ دقیقه |
| ۸ | ۱۵–۲۰ دقیقه |
| ۱۶ | ۸–۱۰ دقیقه |

**روش‌های کامپایل در توزیع‌ها:**

| توزیع | روش |
|---|---|
| Debian/Ubuntu | `make` + `make install` |
| RHEL/Rocky | `rpmbuild` + SRPM |
| Arch | PKGBUILD |

### اشکال شایع

**اشکال ۱: نبود ابزارهای build.**
خطا در کامپایل. راه‌حل: نصب `build-essential`.

**اشکال ۲: فراموش کردن `modules_install`.**
ماژول‌ها نصب نمی‌شوند. راه‌حل: `make modules_install`.

**اشکال ۳: به‌روز نکردن GRUB.**
کرنل جدید در منو نمی‌آید. راه‌حل: `update-grub`.

**اشکال ۴: پیکربندی اشتباه.**
سیستم بوت نمی‌شود (کرنل قدیمی را نگه دارید!). راه‌حل: کرنل قبلی را نگه دارید.

**اشکال ۵: فضای ناکافی.**
کامپایل شکست می‌خورد. راه‌حل: ۲۰+ گیگ آزاد.

**اشکال ۶: کامپایل با هسته‌های بیش از حد.**
ممکن است کند شود. راه‌حل: `-j$(nproc)`.

### نکات مهم (Tips & Tricks)

- **همیشه کرنل قبلی را نگه دارید تا در صورت خطا برگردید.**
- **`make -j$(nproc)` سرعت را بالا می‌برد.**
- **`make localmodconfig` کرنل کوچک می‌سازد.**
- **از `make -j` با تعداد بیش از هسته استفاده نکنید (بی‌فایده).**
- **در VM با فضای کم، کامپایل ممکن است شکست بخورد.**
- **برای RHEL از `rpmbuild` و SRPM استفاده می‌شود.**
- **`make oldconfig` گزینه‌های جدید را می‌پرسد.**
- **`make menuconfig` نیاز به `libncurses-dev`.**
- **`make -j$(nproc) 2>&1 | tee build.log` لاگ می‌گیرد.**
- **`make localyesconfig` ماژول‌ها را در کرنل می‌گنجاند.**
- **`CONFIG_IKCONFIG` امکان دیدن config را می‌دهد.**
- **برای کرنل توزیع، `deb-pkg`/`rpm-pkg` پکیج می‌سازد.**

### خلاصه فصل

کامپایل کرنل شامل دریافت سورس، پیکربندی (`.config`)، کامپایل (`make`)، نصب ماژول‌ها و کرنل و به‌روزرسانی GRUB است. `make menuconfig` پیکربندی تعاملی می‌دهد و `make -j$(nproc)` سرعت را بالا می‌برد. همیشه کرنل قبلی را نگه دارید. ابزارهای build (gcc, make, libncurses) لازم‌اند. `make localmodconfig` کرنل کوچک می‌سازد.

---

## فصل ۶۰: مدیریت و عیب‌یابی زمان اجرای کرنل

### مقدمه

بعد از بوت، باید بدانید کرنل چطور رفتار می‌کند و چطور در صورت مشکل عیب‌یابی کنید. تصور کنید سیستم کرش می‌کند (kernel panic) و باید بفهمید چرا — بدون ابزار مناسب، سخت است.

این فصل مدیریت زمان اجرا و عیب‌یابی کرنل را پوشش می‌دهد.

### مفاهیم اساسی

**مفاهیم کلیدی:**

- **dmesg:** حلقهٔ پیام‌های کرنل (kernel ring buffer).
- **sysctl:** پارامترهای زمان اجرا.
- **/proc/sys/:** رابط فایل‌سیستم برای sysctl.
- **Oops/Panic:** خطای کرنل.
- **Module parameters:** تنظیمات ماژول.
- **kdump:** ذخیرهٔ حافظه در کرش.

**سطوح خطای کرنل:**

| سطح | معنا | اثر |
|---|---|---|
| **Warning** | هشدار | ادامه می‌دهد |
| **Oops** | خطا | ممکن است ادامه دهد |
| **Panic** | بحرانی | سیستم متوقف می‌شود |

### مثال‌های عملی

**پیام‌های کرنل:**

```bash
# همهٔ پیام‌ها
dmesg
dmesg -T              # با زمان خوانا
dmesg -w              # زنده
dmesg -l err,crit     # فقط خطاها
dmesg -H              # خوانا
dmesg | grep -i error
dmesg | grep -i usb

# پاک‌سازی
sudo dmesg -C
```

**پارامترهای زمان اجرا:**

```bash
# از /proc/sys
cat /proc/sys/net/ipv4/ip_forward
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# معادل sysctl
sudo sysctl -w net.ipv4.ip_forward=1

# در sysctl.conf دائمی
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
```

**عیب‌یابی کرنل:**

```bash
# اطلاعات کرنل
cat /proc/version
uname -r

# وضعیت ماژول
lsmod
cat /proc/modules

# بارگذاری با پارامتر
sudo modprobe usbcore nousb=0

# بررسی تیون‌های کرنل
sysctl -a | grep -E "panic|oops"
```

**kdump:**

```bash
# نصب
sudo apt install kdump-tools

# فعال‌سازی
sudo systemctl enable --now kdump-tools

# پیکربندی /etc/default/kdump-tools
# USE_KDUMP=1

# فایل dump
ls /var/crash/
```

### مقایسه و الگوهای مختلف

**ابزارهای لاگ کرنل:**

| ابزار | کار |
|---|---|
| `dmesg` | حلقهٔ پیام‌ها |
| `journalctl -k` | لاگ کرنل (systemd) |
| `/var/log/kern.log` | لاگ کرنل (فایل) |
| `/var/log/dmesg` | dmesg ذخیره‌شده |

**سطوح لاگ:**

| سطح | کاربرد |
|---|---|
| `err`, `crit` | خطاها |
| `warn` | هشدارها |
| `info` | اطلاعات |
| `debug` | اشکال‌زدایی |

**ابزارهای عیب‌یابی:**

| ابزار | کار |
|---|---|
| `dmesg` | پیام‌های کرنل |
| `sysctl` | پارامترها |
| `kdump`/`crash` | تحلیل panic |
| `trace-cmd` | trace |
| `perf` | profiling |

### اشکال شایع

**اشکال ۱: نبود لاگ کرنل.**
panic بعد از ری‌استارت از دست می‌رود؛ باید `kdump` فعال باشد. راه‌حل: kdump.

**اشکال ۲: تغییر موقت sysctl.**
بعد از ری‌استارت برمی‌گردد. راه‌حل: فایل `.conf`.

**اشکال ۳: نادیده گرفتن dmesg.**
منبع اصلی عیب‌یابی سخت‌افزار است. راه‌حل: عادت کنید.

**اشکال ۴: اشتباه پارامتر ماژول.**
ماژول بارگذاری نمی‌شود. راه‌حل: `modinfo`.

**اشکال ۵: حلقهٔ پیام پر.**
پیام‌های قدیمی از دست می‌روند. راه‌حل: `log_buf_len` بزرگ‌تر.

**اشکال ۶: panic بدون clue.**
نیاز به تحلیل. راه‌حل: kdump + crash.

### نکات مهم (Tips & Tricks)

- **`dmesg -T` زمان خوانا می‌دهد.**
- **`dmesg | grep -i error` خطاها را فیلتر می‌کند.**
- **`kdump`/`crash` برای تحلیل panic.**
- **`sysctl -a` همهٔ پارامترها را نشان می‌دهد.**
- **`/etc/sysctl.d/` برای تنظیمات ماژولار بهتر از `sysctl.conf` است.**
- **`journalctl -k -b -1` پیام‌های کرنل بوت قبلی.**
- **`dmesg -w` پایش زنده.**
- **`sysctl vm.swappiness=10` swap را کم می‌کند.**
- **`crash /var/crash/*/vmcore` تحلیل kdump.**
- **`echo t > /proc/sysrq-trigger` نمایش task.**
- **`sysrq` برای بازیابی سیستم hang.**
- **`perf` برای profiling کرنل.**

### خلاصه فصل

مدیریت زمان اجرای کرنل با `dmesg` (پیام‌ها)، `sysctl` (پارامترها) و مدیریت ماژول انجام می‌شود. سطوح خطا: Warning, Oops, Panic. برای عیب‌یابی panic از kdump استفاده کنید. تنظیمات sysctl را در `/etc/sysctl.d/` بنویسید تا پایدار بمانند. `sysrq` برای بازیابی سیستم hang مفید است.

---

## منابع و مراجع

- man pages: `man modprobe`, `man sysctl`, `man dmesg`, `man kdump`
- Kernel Documentation: https://www.kernel.org/doc/html/latest/
- Kernel Newbies: https://kernelnewbies.org
- The Linux Kernel Archives: https://www.kernel.org
- Kdump Documentation: https://www.kernel.org/doc/html/latest/admin-guide/kdump/

**بعدی:** [فایل ۱۷ — راه‌اندازی سیستم](17-System-Startup.md)
