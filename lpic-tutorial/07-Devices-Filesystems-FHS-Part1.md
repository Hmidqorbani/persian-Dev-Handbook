# فایل ۰۷ — دستگاه‌ها، فایل‌سیستم و FHS — بخش اول

**مدت یادگیری**: ۶ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۴ (چیدمان دیسک)، فایل ۰۵–۰۶ (دستورات)

---

## فهرست محتوا

- [فصل ۲۸: ساخت پارتیشن و فایل‌سیستم](#فصل-۲۸-ساخت-پارتیشن-و-فایلسیستم)
- [فصل ۲۹: حفظ یکپارچگی فایل‌سیستم](#فصل-۲۹-حفظ-یکپارچگی-فایلسیستم)
- [فصل ۳۰: کنترل Mount و Unmount](#فصل-۳۰-کنترل-mount-و-unmount)

---

## فصل ۲۸: ساخت پارتیشن و فایل‌سیستم

### مقدمه

بعد از ساخت پارتیشن (فصل ۱۴)، باید یک **فایل‌سیستم (filesystem)** روی آن بسازید تا بتوان داده ذخیره کرد. این فرآیند را **فرمت کردن (formatting)** می‌گویند.

تصور کنید یک دیسک جدید خریدید و می‌خواهید آن را برای داده استفاده کنید. ابتدا پارتیشن‌بندی، سپس فرمت، سپس mount. این فصل گام دوم را پوشش می‌دهد.

### مفاهیم اساسی

**فایل‌سیستم چیست؟**

ساختاری که مشخص می‌کند داده چطور روی دیسک سازمان‌دهی شود — مثل سیستمی از پوشه‌ها و فایل‌ها با متادیتا (مجوز، مالک، زمان). بدون فایل‌سیستم، دیسک فقط یک سری بایت خام است.

**اجزای یک فایل‌سیستم:**

- **Superblock:** اطلاعات کلی (اندازه، تعداد inode، ...).
- **Inode:** متادیتای هر فایل.
- **Data blocks:** محتوای فایل.
- **Journal:** لاگ تراکنش‌ها (برای بازیابی سریع).
- **Block group:** گروهی از بلوک‌ها.

**انواع فایل‌سیستم:**

| فایل‌سیستم | توضیح | کاربرد | حداکثر حجم فایل |
|---|---|---|---|
| **ext4** | استاندارد لینوکس | عمومی، پایدار | ۱۶ TB |
| **XFS** | با کارایی بالا | سرورهای بزرگ | ۸ EB |
| **Btrfs** | مدرن، snapshot | پیشرفته | ۱۶ EB |
| **vfat/FAT32** | سازگاری | فلش، UEFI | ۴ GB |
| **exFAT** | سازگاری مدرن | فلش بزرگ | ۱۶ EB |
| **swap** | حافظهٔ مجازی | — | — |
| **NTFS** | ویندوز | اشتراک با ویندوز | ۱۶ EB |

**مفاهیم کلیدی:**

- **mkfs:** Make FileSystem (ابزار ساخت).
- **Label:** برچسب فایل‌سیستم.
- **UUID:** شناسهٔ یکتا.
- **Journaling:** ثبت تغییرات قبل از اعمال.

### مثال‌های عملی

**ساخت فایل‌سیستم:**

```bash
# ext4
sudo mkfs.ext4 /dev/sdb1
sudo mkfs -t ext4 /dev/sdb1     # معادل

# XFS
sudo mkfs.xfs /dev/sdb1

# vfat (برای EFI)
sudo mkfs.vfat /dev/sdb1
sudo mkfs.vfat -F32 /dev/sdb1   # FAT32

# Btrfs
sudo mkfs.btrfs /dev/sdb1

# با برچسب
sudo mkfs.ext4 -L mydata /dev/sdb1

# با اندازهٔ بلوک
sudo mkfs.ext4 -b 4096 /dev/sdb1

# با نسبت inode
sudo mkfs.ext4 -i 8192 /dev/sdb1

# بررسی قبل از ساخت
lsblk -f
```

**ساخت swap:**

```bash
# با پارتیشن
sudo mkswap /dev/sdb2
sudo mkswap -L swap /dev/sdb2
sudo swapon /dev/sdb2

# یا فایل swap
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# دائمی در fstab
echo "/swapfile none swap sw 0 0" | sudo tee -a /etc/fstab

# بررسی
swapon --show
free -h
```

**بررسی فایل‌سیستم:**

```bash
# اطلاعات فایل‌سیستم ext
sudo tune2fs -l /dev/sdb1
sudo dumpe2fs -h /dev/sdb1

# اطلاعات XFS
sudo xfs_info /dev/sdb1

# حجم و فضای آزاد
df -h
df -T                          # با نوع فایل‌سیستم
df -i                          # inodeها

# مصرف پوشه
du -sh /var/log
du -h --max-depth=1 /var | sort -h
```

**برچسب و UUID:**

```bash
# دیدن
blkid /dev/sdb1
lsblk -f

# تغییر برچسب ext
sudo e2label /dev/sdb1 newlabel
sudo tune2fs -L newlabel /dev/sdb1

# تغییر برچسب XFS
sudo xfs_admin -L newlabel /dev/sdb1

# تغییر برچسب vfat
sudo fatlabel /dev/sdb1 newlabel
```

### مقایسه و الگوهای مختلف

**مقایسهٔ ext4، XFS و Btrfs:**

| ویژگی | ext4 | XFS | Btrfs |
|---|---|---|---|
| حداکثر فایل | ۱۶ TB | ۸ EB | ۱۶ EB |
| حداکثر فایل‌سیستم | ۱ EB | ۸ EB | ۱۶ EB |
| snapshot | خیر | خیر | بله |
| بزرگ‌شدن آنلاین | بله | بله | بله |
| کوچک‌شدن | بله | خیر | بله |
| journaling | بله | بله | بله (COW) |
| مناسب برای | عمومی | فایل بزرگ | snapshot |

**توصیه:** برای سرور عمومی ext4 یا XFS؛ برای قابلیت snapshot از Btrfs.

**journaling در برابر non-journaling:**

| ویژگی | journaling | non-journaling |
|---|---|---|
| بازیابی | سریع | کند (fsck کامل) |
| کارایی | کمی کندتر | سریع‌تر |
| امنیت داده | بالا | پایین |
| مثال | ext4, XFS | ext2 |

**ابزارهای ساخت:**

| ابزار | فایل‌سیستم |
|---|---|
| `mkfs.ext4` | ext2/3/4 |
| `mkfs.xfs` | XFS |
| `mkfs.btrfs` | Btrfs |
| `mkfs.vfat` | FAT |
| `mkfs.exfat` | exFAT |
| `mkswap` | swap |

### اشکال شایع

**اشکال ۱: فرمت کردن پارتیشن اشتباه.**
داده از دست می‌رود! راه‌حل: همیشه `lsblk` را قبل از `mkfs` چک کنید.

**اشکال ۲: فراموش کردن `swapon`.**
بعد از `mkswap` باید swap را فعال کنید. راه‌حل: `swapon`.

**اشکال ۳: فرمت اشتباه برای EFI.**
پارتیشن EFI باید `vfat` باشد. راه‌حل: `mkfs.vfat`.

**اشکال ۴: اشتباه گرفتن `df` و `du`.**
`df` فضای فایل‌سیستم، `du` مصرف پوشه. راه‌حل: به خاطر بسپارید.

**اشکال ۵: نبود journal در ext4.**
بازیابی کند می‌شود. راه‌حل: journaling را فعال نگه دارید.

**اشکال ۶: `mkfs` روی دیسک mount‌شده.**
خطرناک است. راه‌حل: اول `umount`.

### نکات مهم (Tips & Tricks)

- **قبل از `mkfs` حتماً `lsblk -f` بزنید** تا مطمئن شوید پارتیشن درست است.
- **برای ext4، `-m 1` فضای رزرو ریشه را کم می‌کند.**
- **`tune2fs -c 0` بررسی خودکار fsck را غیرفعال می‌کند (سرور).**
- **swap در `/etc/fstab` برای فعال‌سازی دائمی.**
- **`e2label` و `xfs_admin` برچسب فایل‌سیستم را تغییر می‌دهند.**
- **`blkid` UUID و نوع را نشان می‌دهد.**
- **`df -hT` نوع و حجم را با هم نشان می‌دهد.**
- **`du -h --max-depth=1` مصرف سطح اول را نشان می‌دهد.**
- **برای XFS، `mkfs.xfs -f` روی دیسک موجود.**
- **`file -s /dev/sdb1` نوع فایل‌سیستم را تشخیص می‌دهد.**

### خلاصه فصل

بعد از پارتیشن‌بندی، با `mkfs` فایل‌سیستم بسازید. انواع اصلی: ext4 (عمومی)، XFS (کارایی)، Btrfs (snapshot)، vfat (EFI) و swap. با `df -h` فضای فایل‌سیستم و با `du` مصرف پوشه را ببینید. همیشه قبل از `mkfs` پارتیشن را بررسی کنید تا داده از دست نرود. journaling بازیابی سریع می‌دهد. برچسب و UUID با ابزارهای مختلف تغییر می‌کنند.

---

## فصل ۲۹: حفظ یکپارچگی فایل‌سیستم

### مقدمه

تصور کنید برق ناگهان قطع می‌شود و سرور خاموش می‌شود. بعد از روشن شدن، ممکن است فایل‌سیستم آسیب دیده باشد. اگر journaling داشته باشید، بازیابی سریع است؛ اما گاهی نیاز به `fsck` دارید.

فایل‌سیستم ممکن است به دلیل قطع برق، خرابی دیسک یا خاموشی نادرست آسیب ببیند. ابزارهای **fsck** و بکاپ‌گیری به حفظ یکپارچگی کمک می‌کنند.

### مفاهیم اساسی

**یکپارچگی فایل‌سیستم یعنی:** ساختار داخلی (inodeها، بلوک‌ها، جدول‌ها) سالم باشد.

**مفاهیم کلیدی:**

- **inode:** ساختاری که متادیتای فایل را نگه می‌دارد (نه نام).
- **superblock:** اطلاعات کلی فایل‌سیستم.
- **journaling:** ثبت تغییرات قبل از اعمال (بازیابی سریع).
- **fsck:** File System Check (بررسی و تعمیر).
- **orphan inode:** inode بدون نام.
- **lost+found:** پوشهٔ فایل‌های بازیابی‌شده.

**انواع خرابی:**

| نوع | علت | راه‌حل |
|---|---|---|
| Orphan inode | خاموشی ناگهانی | fsck |
| Bad block | خرابی سخت‌افزار | badblocks + fsck |
| Corrupt superblock | خرابی جدی | کپی پشتیبان superblock |
| Inconsistent metadata | قطع برق | journaling/fsck |

### مثال‌های عملی

**بررسی فایل‌سیستم:**

```bash
# بررسی (باید unmount باشد)
sudo umount /dev/sdb1
sudo fsck /dev/sdb1

# بررسی اجباری
sudo fsck -f /dev/sdb1

# بررسی خودکار (بدون پرسش)
sudo fsck -a /dev/sdb1
sudo fsck -y /dev/sdb1

# فقط بررسی بدون تعمیر
sudo fsck -n /dev/sdb1

# بررسی ext با e2fsck
sudo e2fsck -f /dev/sdb1
sudo e2fsck -p /dev/sdb1    # خودکار
```

**بررسی root در بوت:**

```bash
# در بوت، فایل‌سیستم root را نمی‌توان unmount کرد
# با forcefsck یا گزینهٔ کرنل:
sudo touch /forcefsck
sudo reboot

# یا با پارامتر کرنل fsck.mode=force
```

**مدیریت زمان بررسی خودکار (ext):**

```bash
# دیدن تنظیمات
sudo tune2fs -l /dev/sda1 | grep -i "mount count"
sudo tune2fs -l /dev/sda1 | grep -i "check"

# تنظیم بررسی هر N mount
sudo tune2fs -c 30 /dev/sda1

# غیرفعال کردن بررسی زمان‌محور
sudo tune2fs -i 0 /dev/sda1

# تنظیم بررسی هر N روز
sudo tune2fs -i 90d /dev/sda1
```

**بررسی bad block:**

```bash
# بررسی فقط‌خواندنی
sudo badblocks -nsv /dev/sdb1

# بررسی نوشتنی (خطرناک!)
sudo badblocks -sv /dev/sdb1

# با fsck
sudo e2fsck -c /dev/sdb1    # بررسی bad block
```

**بازیابی superblock خراب:**

```bash
# دیدن کپی‌های superblock
sudo dumpe2fs /dev/sdb1 | grep -i superblock

# بازیابی از کپی
sudo e2fsck -b 32768 /dev/sdb1
```

### مقایسه و الگوهای مختلف

**ابزارهای بررسی:**

| فایل‌سیستم | ابزار |
|---|---|
| ext2/3/4 | `fsck.ext4` / `e2fsck` |
| XFS | `xfs_repair` |
| Btrfs | `btrfs check` |
| vfat | `fsck.vfat` / `dosfsck` |

**فلگ‌های fsck:**

| فلگ | اثر |
|---|---|
| `-f` | بررسی اجباری |
| `-a` | خودکار (بدون پرسش) |
| `-y` | پاسخ بله به همه |
| `-n` | بدون تغییر (فقط بررسی) |
| `-p` | پیش‌فرض خودکار |
| `-c` | بررسی bad block |
| `-v` | نمایش |

**نکتهٔ مهم:** `fsck` فقط روی فایل‌سیستم **unmount‌شده** امن است. برای root باید از rescue mode یا live CD استفاده کنید.

### اشکال شایع

**اشکال ۱: `fsck` روی فایل‌سیستم mount‌شده.**
می‌تواند داده را خراب کند! راه‌حل: اول unmount.

**اشکال ۲: فراموش کردن `-f`.**
fsck بدون `-f` ممکن است بررسی را رد کند (اگر «clean» علامت‌خورده باشد). راه‌حل: `-f` بزنید.

**اشکال ۳: بررسی root بدون rescue mode.**
باید از حالت تک‌کاربره یا live CD استفاده کنید. راه‌حل: rescue.

**اشکال ۴: نادیده گرفتن خطاهای دیسک.**
اگر fsck خطای سخت‌افزاری داد، دیسک را با `smartctl` بررسی کنید. راه‌حل: SMART.

**اشکال ۵: badblocks نوشتنی روی دیسک فعال.**
داده خراب می‌شود. راه‌حل: فقط روی دیسک خام.

**اشکال ۶: نبود بکاپ.**
بازیابی محدود می‌شود. راه‌حل: همیشه بکاپ.

### نکات مهم (Tips & Tricks)

- **همیشه قبل از fsck بکاپ بگیرید.**
- **`smartctl -a /dev/sda` سلامت دیسک را نشان می‌دهد.**
- **برای ext4، journaling را با `tune2fs -O has_journal` مدیریت کنید.**
- **`dumpe2fs` اطلاعات superblock و گروه‌ها را نشان می‌دهد.**
- **برای بازیابی فایل حذف‌شده از `extundelete` یا `photorec` استفاده کنید.**
- **`e2fsck -b 32768` از superblock پشتیبان بازیابی می‌کند.**
- **`lost+found` را دوره‌ای بررسی کنید.**
- **`tune2fs -C 0` شمارندهٔ mount را ریست می‌کند.**
- **`fsck` را در fstab با ستون pass تنظیم کنید.**
- **برای XFS، `xfs_repair -n` اول بررسی کنید.**

### خلاصه فصل

یکپارچگی فایل‌سیستم با inode، superblock و journaling حفظ می‌شود. `fsck` (و معادل‌های xfs_repair/btrfs check) برای بررسی و تعمیر است، اما فقط روی فایل‌سیستم unmount‌شده. بررسی خودکار با `tune2fs` تنظیم می‌شود. برای root از rescue mode استفاده کنید و همیشه بکاپ داشته باشید. `badblocks` و `smartctl` سلامت سخت‌افزار را بررسی می‌کنند.

---

## فصل ۳۰: کنترل Mount و Unmount

### مقدمه

در لینوکس، فایل‌سیستم‌ها برای دسترسی باید **mount** شوند — یعنی به یک پوشه (mount point) متصل شوند. تصور کنید یک USB وصل می‌کنید؛ تا وقتی mount نشود، نمی‌توانید به فایل‌هایش دسترسی داشته باشید.

این فصل mount، unmount و پیکربندی دائمی آن‌ها را پوشش می‌دهد — یکی از پرکاربردترین مباحث آزمون.

### مفاهیم اساسی

**Mount چیست؟**

اتصال یک فایل‌سیستم (روی دیسک یا شبکه) به یک پوشه در درخت فایل.

**مفاهیم کلیدی:**

- **mount point:** پوشه‌ای که فایل‌سیستم به آن متصل می‌شود.
- **`/etc/fstab`:** فایل پیکربندی mountهای دائمی.
- **UUID:** شناسهٔ یکتای فایل‌سیستم (پایدارتر از نام دستگاه).
- **bind mount:** mount یک پوشه در پوشهٔ دیگر.
- **loop device:** mount یک فایل ISO به‌عنوان دیسک.

**ستون‌های `/etc/fstab`:**

```
<device>    <mount>    <type>    <options>    <dump>  <pass>
UUID=...    /          ext4      defaults     0       1
```

| ستون | معنا |
|---|---|
| device | دستگاه (UUID توصیه‌شده) |
| mount | نقطهٔ mount |
| type | نوع فایل‌سیستم |
| options | گزینه‌ها |
| dump | پشتیبان‌گیری (۰ = نه) |
| pass | ترتیب fsck (۰ = نه، ۱ = ریشه) |

### مثال‌های عملی

**Mount دستی:**

```bash
# mount ساده
sudo mount /dev/sdb1 /mnt

# با نوع مشخص
sudo mount -t ext4 /dev/sdb1 /mnt

# فقط‌خواندنی
sudo mount -o ro /dev/sdb1 /mnt

# با گزینه‌ها
sudo mount -o rw,noatime /dev/sdb1 /mnt

# mount از fstab (فقط با نام)
sudo mount /mnt
sudo mount /dev/sdb1

# دیدن mountهای فعال
mount
findmnt
findmnt -t ext4
cat /proc/mounts
```

**Unmount:**

```bash
sudo umount /mnt
sudo umount /dev/sdb1

# اگر busy است
sudo umount -l /mnt      # lazy
sudo umount -f /mnt      # force (برای NFS)
sudo fuser -m /mnt       # ببینید چه کسی استفاده می‌کند
sudo lsof +D /mnt
```

**پیکربندی دائمی (`/etc/fstab`):**

```
# <device>        <mount>   <type>  <options>         <dump> <pass>
UUID=xxxx-xxxx    /         ext4    errors=remount-ro 0      1
UUID=yyyy-yyyy    /home     ext4    defaults          0      2
UUID=zzzz-zzzz    none      swap    sw                0      0
/dev/sdb1         /mnt/data ext4    defaults,noatime  0      2
tmpfs             /tmp      tmpfs   defaults,size=2G  0      0
```

**دیدن UUID:**

```bash
blkid
blkid /dev/sdb1
lsblk -f
sudo tune2fs -l /dev/sdb1 | grep UUID
```

**Loop mount (ISO):**

```bash
sudo mount -o loop file.iso /mnt
sudo umount /mnt

# ساخت فایل‌سیستم در فایل
dd if=/dev/zero of=disk.img bs=1M count=100
mkfs.ext4 disk.img
sudo mount -o loop disk.img /mnt
```

**Bind mount:**

```bash
sudo mount --bind /src /dst
sudo mount --bind /var/www /mnt/www
```

### مقایسه و الگوهای مختلف

**گزینه‌های مهم mount:**

| گزینه | معنا |
|---|---|
| `defaults` | rw, suid, dev, exec, auto, nouser, async |
| `ro` / `rw` | فقط‌خواندنی / خواندنی-نوشتنی |
| `noatime` | به‌روزرسانی نکن زمان دسترسی (کارایی) |
| `nodiratime` | فقط پوشه‌ها |
| `noexec` | ممنوعیت اجرای فایل |
| `nosuid` | نادیده گرفتن SUID |
| `nodev` | نادیده گرفتن فایل‌های دستگاه |
| `bind` | mount یک پوشه در پوشهٔ دیگر |
| `_netdev` | منتظر شبکه (برای NFS) |
| `loop` | فایل به‌عنوان دیسک |
| `user` / `users` | اجازهٔ mount به کاربر |
| `auto` / `noauto` | mount خودکار در بوت |

**مشخص کردن دستگاه:**

| روش | مثال | پایداری |
|---|---|---|
| نام | `/dev/sdb1` | کم (ممکن است تغییر کند) |
| UUID | `UUID=...` | زیاد |
| LABEL | `LABEL=data` | متوسط |
| PARTUUID | `PARTUUID=...` | زیاد |

**گزینه‌های امنیتی:**

| گزینه | کاربرد |
|---|---|
| `noexec` | `/tmp`, `/var/tmp` |
| `nosuid` | `/home`, `/tmp` |
| `nodev` | پوشه‌های داده |

### اشکال شایع

**اشکال ۱: استفاده از `/dev/sdX` در fstab.**
نام‌ها ممکن است تغییر کنند؛ از UUID استفاده کنید. راه‌حل: `blkid` و UUID.

**اشکال ۲: mount روی پوشهٔ غیرخالی.**
محتوای پوشه پنهان می‌شود. راه‌حل: پوشهٔ خالی.

**اشکال ۳: اشتباه در fstab.**
سیستم بوت نمی‌شود (به rescue mode می‌رود). راه‌حل: `mount -a` قبل از reboot.

**اشکال ۴: فراموش کردن `_netdev` برای NFS.**
بوت کند می‌شود. راه‌حل: `_netdev`.

**اشکال ۵: unmount در حال استفاده.**
خطای «target is busy» می‌دهد. راه‌حل: `fuser`/`lsof` برای پیدا کردن.

**اشکال ۶: noexec روی پوشه‌ای که اسکریپت دارد.**
اسکریپت اجرا نمی‌شود. راه‌حل: گزینه را بردارید.

### نکات مهم (Tips & Tricks)

- **بعد از ویرایش fstab، با `sudo mount -a` تست کنید (قبل از reboot).**
- **`findmnt` نمایش درختی mountها می‌دهد.**
- **`noatime` کارایی را در سرورهای پرمصرف بهبود می‌دهد.**
- **برای bind mount: `mount --bind /src /dst`.**
- **`systemd` mount units را با `systemctl list-units --type=mount` ببینید.**
- **`findmnt --verify` صحت fstab را بررسی می‌کند.**
- **`mount | column -t` خروجی خوانا.**
- **`/etc/mtab` (لینک به `/proc/mounts`) mountهای فعال.**
- **`mount -o remount,rw /` برای تغییر حالت ریشه.**
- **`udisksctl` برای mount بدون ریشه (دسکتاپ).**
- **`autofs` برای mount خودکار بر تقاضا.**
- **ستون pass در fstab ترتیب fsck را تعیین می‌کند (۱ برای ریشه).**

### خلاصه فصل

Mount اتصال فایل‌سیستم به درخت فایل است. با `mount`/`umount` دستی و با `/etc/fstab` دائمی می‌شود. از UUID به‌جای نام دستگاه استفاده کنید. گزینه‌های مهم: `defaults`, `noatime`, `ro`, `noexec`, `_netdev`. بعد از ویرایش fstab با `mount -a` تست کنید. Loop mount برای ISO و bind mount برای پوشه‌ها کاربرد دارند. ستون pass ترتیب fsck را تعیین می‌کند.

---

## منابع و مراجع

- man pages: `man mkfs`, `man fsck`, `man mount`, `man fstab`, `man blkid`, `man tune2fs`
- Filesystem Hierarchy Standard: https://refspecs.linuxfoundation.org/FHS_3.0/fhs-3.0.html
- ext4 Documentation: https://www.kernel.org/doc/html/latest/filesystems/ext4/
- XFS Documentation: https://xfs.org

**بعدی:** [فایل ۰۸ — دستگاه‌ها، فایل‌سیستم و FHS — بخش دوم](08-Devices-Filesystems-FHS-Part2.md)
