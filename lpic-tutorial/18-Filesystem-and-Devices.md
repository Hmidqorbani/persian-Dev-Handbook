# فایل ۱۸ — فایل‌سیستم و دستگاه‌ها

**مدت یادگیری**: ۶ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: فایل ۰۷–۰۸ (فایل‌سیستم پایه)

---

## فهرست محتوا

- [فصل ۶۴: کار با فایل‌سیستم لینوکس](#فصل-۶۴-کار-با-فایلسیستم-لینوکس)
- [فصل ۶۵: نگهداری فایل‌سیستم لینوکس](#فصل-۶۵-نگهداری-فایلسیستم-لینوکس)
- [فصل ۶۶: ساخت و پیکربندی گزینه‌های فایل‌سیستم](#فصل-۶۶-ساخت-و-پیکربندی-گزینههای-فایلسیستم)

---

## فصل ۶۴: کار با فایل‌سیستم لینوکس

### مقدمه

در LPIC-1 پایهٔ فایل‌سیستم را دیدیم. در LPIC-2 عمیق‌تر می‌شویم: تنظیم پارامترهای پیشرفته، بهینه‌سازی و عیب‌یابی دقیق.

تصور کنید یک فایل‌سیستم پر از فایل‌های کوچک است و فضای inode تمام شده، یا می‌خواهید کارایی را با تنظیم پارامترها بالا ببرید.

### مفاهیم اساسی

**ساختار داخلی ext4:**

- **inode:** متادیتای فایل (مجوز، مالک، بلوک‌ها).
- **block:** واحد ذخیره‌سازی (معمولاً ۴KB).
- **superblock:** اطلاعات کلی.
- **block group:** گروهی از بلوک‌ها.
- **journal:** لاگ تراکنش‌ها.

**مفاهیم پیشرفته:**

- **tune2fs:** تنظیم پارامترهای ext.
- **dumpe2fs:** نمایش جزئیات.
- **resize2fs:** تغییر اندازه.
- **e2fsck:** بررسی.

**پارامترهای ext4:**

| پارامتر | معنا |
|---|---|
| Block size | اندازهٔ بلوک |
| Inode ratio | نسبت inode به فضا |
| Reserved blocks | فضای رزرو ریشه |
| Journal | لاگ تراکنش |
| Mount count | شمارندهٔ mount |
| Check interval | فاصلهٔ بررسی |

### مثال‌های عملی

**اطلاعات دقیق فایل‌سیستم:**

```bash
# اطلاعات ext
sudo dumpe2fs /dev/sda1
sudo dumpe2fs -h /dev/sda1    # فقط superblock
sudo tune2fs -l /dev/sda1

# پارامترهای کلیدی
sudo tune2fs -l /dev/sda1 | grep -E "Block count|Inode count|Reserved"

# inodeها
df -i
sudo tune2fs -l /dev/sda1 | grep -i inode
```

**تنظیم ext4:**

```bash
# کاهش فضای رزرو ریشه (از ۵٪ به ۱٪)
sudo tune2fs -m 1 /dev/sda1

# تغییر برچسب
sudo tune2fs -L mydata /dev/sda1
sudo e2label /dev/sda1 mydata

# فعال/غیرفعال journaling
sudo tune2fs -O ^has_journal /dev/sda1    # غیرفعال (احتیاط)
sudo tune2fs -O has_journal /dev/sda1     # فعال

# تنظیم بررسی خودکار
sudo tune2fs -c 30 -i 90d /dev/sda1

# غیرفعال کردن بررسی
sudo tune2fs -c 0 -i 0 /dev/sda1

# تنظیم ویژگی‌ها
sudo tune2fs -O dir_index /dev/sda1
```

**تغییر اندازه:**

```bash
# بزرگ‌کردن (بعد از بزرگ کردن پارتیشن)
sudo resize2fs /dev/sda1
sudo resize2fs /dev/sda1 20G    # اندازهٔ مشخص

# کوچک‌کردن (اول fsck)
sudo umount /dev/sda1
sudo e2fsck -f /dev/sda1
sudo resize2fs /dev/sda1 10G
```

**XFS tools:**

```bash
# اطلاعات
sudo xfs_info /dev/sdb1
sudo xfs_db -r -c "sb 0" -c "print" /dev/sdb1

# بزرگ‌کردن (فقط بزرگ!)
sudo xfs_growfs /mnt/xfs
sudo xfs_growfs -d /mnt/xfs    # تا انتهای دیسک

# تعمیر
sudo xfs_repair /dev/sdb1
sudo xfs_repair -n /dev/sdb1   # بررسی بدون تغییر

# تنظیم
sudo xfs_admin -L mylabel /dev/sdb1
```

### مقایسه و الگوهای مختلف

**پارامترهای مهم tune2fs:**

| پارامتر | اثر |
|---|---|
| `-m N` | درصد رزرو ریشه |
| `-L label` | برچسب |
| `-c N` | بررسی هر N mount |
| `-i N` | بررسی هر N روز |
| `-O feature` | فعال/غیرفعال ویژگی |
| `-U uuid` | تغییر UUID |
| `-e behavior` | رفتار در خطا |

**ویژگی‌های ext4:**

| ویژگی | کار |
|---|---|
| `has_journal` | journaling |
| `extent` | اکستنت (کارایی) |
| `dir_index` | ایندکس پوشه |
| `flex_bg` | گروه‌بندی انعطاف |
| `64bit` | فایل‌سیستم بزرگ |

**ابزارهای XFS:**

| ابزار | کار |
|---|---|
| `xfs_info` | اطلاعات |
| `xfs_growfs` | بزرگ‌کردن |
| `xfs_repair` | تعمیر |
| `xfs_admin` | تنظیم |
| `xfs_db` | اشکال‌زدایی |

### اشکال شایع

**اشکال ۱: کوچک کردن ext بدون fsck.**
خطا می‌دهد. راه‌حل: اول `e2fsck -f`.

**اشکال ۲: کوچک کردن XFS.**
XFS نمی‌تواند کوچک شود! راه‌حل: فایل‌سیستم جدید.

**اشکال ۳: resize بدون بزرگ کردن پارتیشن.**
ابتدا پارتیشن، سپس فایل‌سیستم. راه‌حل: ترتیب درست.

**اشکال ۴: غیرفعال کردن journal.**
خطر از دست دادن داده. راه‌حل: نگه دارید.

**اشکال ۵: رزرو ریشه اشتباه.**
فضای هدر یا نبود فضای ریشه. راه‌حل: `-m 1`.

**اشکال ۶: درون inode اشتباه.**
فایل‌های کوچک زیاد → inode بیشتر. راه‌حل: `-i` هنگام ساخت.

### نکات مهم (Tips & Tricks)

- **`resize2fs` می‌تواند آنلاین بزرگ کند (ext4).**
- **همیشه قبل از resize بکاپ بگیرید.**
- **`-m 1` فضای مفید را در سرورهای بزرگ افزایش می‌دهد.**
- **`dumpe2fs | grep -i "block count"` حجم را نشان می‌دهد.**
- **برای XFS فقط بزرگ‌کردن ممکن است.**
- **`e2fsck -f` قبل از resize اجباری است.**
- **`xfs_growfs -d` تا انتهای دیسک.**
- **`tune2fs -O ^has_journal` فقط برای سیستم‌های خاص.**
- **`df -i` مصرف inode را نشان می‌دهد.**
- **`tune2fs -c 0` بررسی خودکار را غیرفعال می‌کند.**
- **`debugfs` ابزار پیشرفتهٔ ext.**
- **`xfs_repair -n` بررسی بدون تغییر.**

### خلاصه فصل

ext4 از inode، block، superblock و journal ساخته شده. `tune2fs` پارامترها را تنظیم و `dumpe2fs` جزئیات را نشان می‌دهد. `resize2fs` اندازه را تغییر می‌دهد (آنلاین برای بزرگ‌کردن). XFS فقط بزرگ می‌شود. همیشه قبل از resize بکاپ و fsck. `-m 1` فضای رزرو ریشه را کم می‌کند و `df -i` مصرف inode را نشان می‌دهد.

---

## فصل ۶۵: نگهداری فایل‌سیستم لینوکس

### مقدمه

نگهداری منظم فایل‌سیستم از خرابی و از دست رفتن داده جلوگیری می‌کند. تصور کنید دیسکی در حال خرابی است اما SMART هشدار داده — با پایش منظم، قبل از فاجعه اقدام می‌کنید.

این فصل روش‌های نگهداری پیشرفته را پوشش می‌دهد.

### مفاهیم اساسی

**کارهای نگهداری:**

- بررسی دوره‌ای (fsck).
- پایش فضای آزاد و inode.
- چرخش لاگ (logrotate).
- پایش سلامت دیسک (SMART).
- بکاپ منظم.

**مفاهیم کلیدی:**

- **SMART:** فناوری خودپایش دیسک.
- **badblocks:** بلوک‌های خراب.
- **logrotate:** چرخش لاگ.
- **fstrim:** TRIM برای SSD.

### مثال‌های عملی

**بررسی سلامت دیسک:**

```bash
# نصب smartmontools
sudo apt install smartmontools

# وضعیت کلی
sudo smartctl -H /dev/sda

# همهٔ اطلاعات
sudo smartctl -a /dev/sda

# تست کوتاه/بلند
sudo smartctl -t short /dev/sda
sudo smartctl -t long /dev/sda

# بررسی نتیجهٔ تست
sudo smartctl -l selftest /dev/sda

# برای NVMe
sudo nvme smart-log /dev/nvme0
```

**بررسی بلوک‌های خراب:**

```bash
sudo badblocks -sv /dev/sda1

# بررسی فقط‌خواندنی
sudo badblocks -nsv /dev/sda1

# با fsck
sudo e2fsck -c /dev/sda1
```

**مدیریت فضای دیسک:**

```bash
# پرمصرف‌ترین پوشه‌ها
du -sh /* 2>/dev/null | sort -rh | head -10
du -h --max-depth=1 /var | sort -h

# فایل‌های بزرگ
find / -type f -size +100M 2>/dev/null
find / -type f -size +1G 2>/dev/null

# فایل‌های حذف‌شده ولی باز
sudo lsof | grep deleted

# تحلیل تعاملی
ncdu /
```

**logrotate:**

```bash
# /etc/logrotate.conf
# /etc/logrotate.d/

# نمونه /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0640 www-data adm
    sharedscripts
    postrotate
        systemctl reload myapp > /dev/null 2>&1 || true
    endscript
}

# تست
sudo logrotate -d /etc/logrotate.conf    # dry-run
sudo logrotate -f /etc/logrotate.conf    # اجرای اجباری
```

**TRIM برای SSD:**

```bash
# فعال‌سازی timer
sudo systemctl enable --now fstrim.timer

# اجرای دستی
sudo fstrim -v /
sudo fstrim -av

# بررسی
systemctl status fstrim.timer
```

### مقایسه و الگوهای مختلف

**ابزارهای پایش دیسک:**

| ابزار | کار |
|---|---|
| `smartctl` | سلامت SMART |
| `badblocks` | بلوک خراب |
| `fsck` | یکپارچگی |
| `dstat`/`iostat` | I/O |
| `nvme` | NVMe |

**سیاست‌های logrotate:**

| گزینه | اثر |
|---|---|
| `daily/weekly/monthly` | فرکانس |
| `rotate N` | تعداد نگهداری |
| `compress` | فشرده‌سازی |
| `maxsize` | حداکثر حجم |
| `postrotate` | دستور بعد از چرخش |
| `copytruncate` | کپی + خالی (برای سرویس‌های بدون reload) |

**ابزارهای تحلیل فضا:**

| ابزار | ویژگی |
|---|---|
| `du` | استاندارد |
| `ncdu` | تعاملی |
| `duc` | سریع، database |
| `baobab` | گرافیکی |

### اشکال شایع

**اشکال ۱: نادیده گرفتن SMART.**
خرابی دیسک پیش‌بینی‌نشده. راه‌حل: `smartctl` دوره‌ای.

**اشکال ۲: نبود logrotate.**
دیسک با لاگ پر می‌شود. راه‌حل: logrotate.

**اشکال ۳: badblocks نوشتنی روی دیسک فعال.**
داده خراب می‌شود. راه‌حل: فقط روی دیسک خام.

**اشکال ۴: نبود بکاپ.**
نگهداری جای بکاپ را نمی‌گیرد. راه‌حل: بکاپ.

**اشکال ۵: نبود TRIM روی SSD.**
SSD کند می‌شود. راه‌حل: `fstrim.timer`.

**اشکال ۶: فایل‌های حذف‌شدهٔ باز.**
فضا آزاد نمی‌شود. راه‌حل: restart سرویس.

### نکات مهم (Tips & Tricks)

- **SMART را دوره‌ای بررسی کنید (cron).**
- **`smartd` daemon هشدار خودکار می‌دهد.**
- **`logrotate` معمولاً با cron اجرا می‌شود.**
- **`du -sh * | sort -h` پرمصرف‌ها را پیدا می‌کند.**
- **فایل‌های حذف‌شدهٔ باز فضا را آزاد نمی‌کنند؛ سرویس را restart کنید.**
- **`ncdu` ابزار تعاملی تحلیل فضاست.**
- **`smartctl -a` همهٔ پارامترهای SMART.**
- **`fstrim.timer` TRIM دوره‌ای.**
- **`logrotate -d` حالت dry-run.**
- **`/etc/logrotate.d/` برای هر سرویس.**
- **`e2fsck -c` بلوک‌های خراب را پیدا و علامت می‌زند.**
- **`tune2fs -l | grep -i "block count"` حجم را نشان می‌دهد.**

### خلاصه فصل

نگهداری فایل‌سیستم شامل بررسی دوره‌ای (fsck)، پایش SMART (smartctl)، بررسی badblocks، مدیریت فضا (du/find/ncdu) و logrotate است. `smartd` هشدار خودکار می‌دهد و `fstrim.timer` TRIM برای SSD. فایل‌های حذف‌شدهٔ باز فضا آزاد نمی‌کنند. نگهداری جای بکاپ را نمی‌گیرد.

---

## فصل ۶۶: ساخت و پیکربندی گزینه‌های فایل‌سیستم

### مقدمه

انتخاب و پیکربندی درست فایل‌سیستم بر کارایی و قابلیت اطمینان اثر می‌گذارد. تصور کنید یک سرور با بار سنگین I/O دارید — انتخاب فایل‌سیستم و گزینه‌های mount می‌تواند تفاوت زیادی ایجاد کند.

در این فصل ساخت و تنظیم پیشرفته را یاد می‌گیرید.

### مفاهیم اساسی

**گزینه‌های ساخت (mkfs):**

| گزینه | اثر |
|---|---|
| `-t type` | نوع فایل‌سیستم |
| `-L label` | برچسب |
| `-b size` | اندازهٔ بلوک |
| `-i bytes` | نسبت inode |
| `-m percent` | رزرو ریشه |
| `-O feature` | ویژگی‌ها |

**گزینه‌های mount:**

| گزینه | اثر |
|---|---|
| `noatime` | بدون به‌روزرسانی زمان دسترسی |
| `nodiratime` | فقط پوشه‌ها |
| `data=writeback/ordered/journal` | حالت journaling |
| `discard` | TRIM برای SSD |
| `nobarrier` | حذف barrier (احتیاط) |
| `noexec` | ممنوعیت اجرا |
| `nosuid` | نادیده گرفتن SUID |

### مثال‌های عملی

**ساخت با گزینه:**

```bash
# ext4 با تنظیمات بهینه
sudo mkfs.ext4 -L data -b 4096 -m 1 /dev/sdb1
sudo mkfs.ext4 -O ^has_journal /dev/sdb1    # بدون journal

# XFS با حجم بزرگ
sudo mkfs.xfs -f -L data -d agcount=8 /dev/sdb1
sudo mkfs.xfs -b size=4096 /dev/sdb1

# Btrfs
sudo mkfs.btrfs -L data /dev/sdb1

# بررسی
sudo tune2fs -l /dev/sdb1 | head
sudo xfs_info /dev/sdb1
```

**بهینه‌سازی برای SSD:**

```bash
# TRIM دوره‌ای
sudo systemctl enable fstrim.timer
sudo fstrim -v /

# mount با discard
# /etc/fstab: UUID=... / ext4 defaults,discard 0 1

# یا noatime
# UUID=... / ext4 defaults,noatime 0 1
```

**پیکربندی fstab پیشرفته:**

```bash
# /etc/fstab
UUID=xxx  /         ext4  defaults,noatime,errors=remount-ro  0 1
UUID=yyy  /data     xfs   defaults,noatime,inode64            0 2
tmpfs     /tmp      tmpfs defaults,noatime,size=2G            0 0
UUID=zzz  /home     ext4  defaults,nosuid,nodev               0 2
```

**ایجاد swap بهینه:**

```bash
# فایل swap
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# تنظیم swappiness
echo "vm.swappiness=10" | sudo tee /etc/sysctl.d/99-swap.conf
```

### مقایسه و الگوهای مختلف

**انتخاب فایل‌سیستم:**

| کاربرد | پیشنهاد |
|---|---|
| عمومی | ext4 |
| فایل بزرگ | XFS |
| snapshot | Btrfs |
| SSD | ext4/XFS + discard |
| فلش/USB | ext4 یا exFAT |
| اشتراک ویندوز | NTFS/exFAT |

**حالت‌های journaling:**

| حالت | امنیت | سرعت |
|---|---|---|
| `journal` | بالا | کم |
| `ordered` | متوسط (پیش‌فرض) | متوسط |
| `writeback` | کم | بالا |

**اندازهٔ بلوک:**

| اندازه | مناسب |
|---|---|
| 1KB | فایل‌های کوچک |
| 4KB | عمومی (پیش‌فرض) |
| 16KB+ | فایل‌های بزرگ |

### اشکال شایع

**اشکال ۱: discard روی همه SSDها.**
بعضی SSDها بهتر است `fstrim` دوره‌ای داشته باشند. راه‌حل: fstrim.timer.

**اشکال ۲: اندازهٔ بلوک اشتباه.**
برای فایل‌های کوچک زیاد، بلوک کوچک بهتر. راه‌حل: بلوک مناسب.

**اشکال ۳: tmpfs بزرگ.**
حافظه را اشغال می‌کند. راه‌حل: اندازهٔ محدود.

**اشکال ۴: writeback بدون درک.**
خطر از دست دادن داده. راه‌حل: ordered.

**اشکال ۵: noexec روی پوشه‌ای که اسکریپت دارد.**
اسکریپت اجرا نمی‌شود. راه‌حل: گزینه را بردارید.

**اشکال ۶: inode کم.**
فایل‌های کوچک زیاد → inode تمام. راه‌حل: `-i` هنگام ساخت.

### نکات مهم (Tips & Tricks)

- **برای SSD، `fstrim.timer` را فعال کنید.**
- **`noatime` کارایی را بهبود می‌دهد.**
- **`inode64` برای XFS در فایل‌سیستم‌های بزرگ.**
- **`tmpfs` برای `/tmp` سرعت را بالا می‌برد (اما فرار است).**
- **`errors=remount-ro` ایمنی را افزایش می‌دهد.**
- **برای فایل‌سیستم‌های اشتراکی، `user_xattr` و ACL را فعال کنید.**
- **`data=writeback` سرعت بیشتر اما امنیت کمتر.**
- **`nobarrier` فقط اگر باتری RAID دارید.**
- **`discard` آنلاین TRIM می‌زند (یا fstrim).**
- **`vm.swappiness=10` swap را کم می‌کند.**
- **`noexec` روی `/tmp` امنیت را بالا می‌برد.**
- **`commit=60` فرکانس sync را کم می‌کند.**

### خلاصه فصل

انتخاب فایل‌سیستم (ext4/XFS/Btrfs) و گزینه‌های ساخت (`-b`, `-m`, `-i`, `-O`) و mount (`noatime`, `discard`, `data=`) بر کارایی اثر می‌گذارند. برای SSD از `fstrim.timer` استفاده کنید. `errors=remount-ro` ایمنی می‌دهد. `tmpfs` برای فایل‌های موقت سریع است. حالت journaling تعادل امنیت/سرعت را تعیین می‌کند.

---

## منابع و مراجع

- man pages: `man tune2fs`, `man resize2fs`, `man smartctl`, `man logrotate`, `man mkfs.ext4`, `man xfs_growfs`
- ext4 Wiki: https://ext4.wiki.kernel.org
- smartmontools: https://www.smartmontools.org
- XFS Documentation: https://xfs.org
- Btrfs Wiki: https://btrfs.wiki.kernel.org

**بعدی:** [فایل ۱۹ — ذخیره‌سازی پیشرفته: RAID و LVM](19-Advanced-Storage-RAID-LVM.md)
