# فایل ۱۹ — ذخیره‌سازی پیشرفته: RAID و LVM

**مدت یادگیری**: ۸ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: فایل ۱۸ (فایل‌سیستم و دستگاه‌ها)

---

## فهرست محتوا

- [فصل ۶۷: پیکربندی RAID](#فصل-۶۷-پیکربندی-raid)
- [فصل ۶۸: تنظیم دسترسی دستگاه ذخیره‌سازی](#فصل-۶۸-تنظیم-دسترسی-دستگاه-ذخیرهسازی)
- [فصل ۶۹: Logical Volume Manager](#فصل-۶۹-logical-volume-manager)
- [فصل ۷۰: آزمایشگاه: RAID و LVM به‌صورت عملی](#فصل-۷۰-آزمایشگاه-raid-و-lvm-بهصورت-عملی)

---

## فصل ۶۷: پیکربندی RAID

### مقدمه

**RAID (Redundant Array of Independent Disks)** ترکیب چند دیسک برای افزایش کارایی، افزونگی (redundancy) یا هر دو است. تصور کنید یک سرور با یک دیسک دارید و آن دیسک خراب می‌شود — همهٔ داده از دست می‌رود. با RAID 1، یک دیسک می‌تواند خراب شود و داده سالم بماند.

در سرورها برای محافظت از داده و بهبود سرعت حیاتی است و در آزمون LPIC-2 وزن دارد.

### مفاهیم اساسی

**سطوح RAID:**

| سطح | نام | حداقل دیسک | تحمل خرابی | حجم مفید |
|---|---|---|---|---|
| **RAID 0** | Striping | ۲ | ۰ | ۱۰۰٪ |
| **RAID 1** | Mirroring | ۲ | ۱ | ۵۰٪ |
| **RAID 5** | Striping + Parity | ۳ | ۱ | (N-1)/N |
| **RAID 6** | Double Parity | ۴ | ۲ | (N-2)/N |
| **RAID 10** | Mirror + Stripe | ۴ | ۱ در هر آینه | ۵۰٪ |

**مفاهیم کلیدی:**

- **Software RAID:** با `mdadm` (کرنل).
- **Hardware RAID:** کارت RAID اختصاصی.
- **Hot Spare:** دیسک یدکی که خودکار جایگزین می‌شود.
- **Rebuild:** بازسازی پس از جایگزینی دیسک.
- **Degraded:** آرایه با دیسک خراب (اما فعال).

**مقایسه با LVM:** RAID افزونگی می‌دهد، LVM انعطاف. اغلب با هم استفاده می‌شوند.

### مثال‌های عملی

**ساخت Software RAID با mdadm:**

```bash
# نصب
sudo apt install mdadm

# RAID 1 از دو دیسک
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# RAID 5 از سه دیسک
sudo mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd

# RAID 10 از چهار دیسک
sudo mdadm --create /dev/md0 --level=10 --raid-devices=4 /dev/sd[b-e]

# با hot spare
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 \
  --spare-devices=1 /dev/sdb /dev/sdc /dev/sdd

# بررسی
cat /proc/mdstat
sudo mdadm --detail /dev/md0

# ساخت فایل‌سیستم
sudo mkfs.ext4 /dev/md0

# mount
sudo mount /dev/md0 /mnt/raid
```

**مدیریت:**

```bash
# ذخیرهٔ پیکربندی
sudo mdadm --detail --scan >> /etc/mdadm/mdadm.conf
sudo update-initramfs -u

# شبیه‌سازی خرابی
sudo mdadm --fail /dev/md0 /dev/sdc
sudo mdadm --remove /dev/md0 /dev/sdc

# افزودن دیسک جدید
sudo mdadm --add /dev/md0 /dev/sdd

# متوقف کردن
sudo mdadm --stop /dev/md0

# پاک‌سازی superblock
sudo mdadm --zero-superblock /dev/sdb /dev/sdc

# بزرگ کردن (grow)
sudo mdadm --grow /dev/md0 --raid-devices=3

# پایش
sudo mdadm --monitor --scan --daemonise
```

**بررسی وضعیت:**

```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
sudo mdadm --examine /dev/sdb
lsblk
```

### مقایسه و الگوهای مختلف

**مقایسهٔ سطوح:**

| معیار | RAID 0 | RAID 1 | RAID 5 | RAID 6 | RAID 10 |
|---|---|---|---|---|---|
| کارایی خواندن | عالی | خوب | خوب | خوب | عالی |
| کارایی نوشتن | عالی | متوسط | متوسط | کم | خوب |
| افزونگی | ندارد | بالا | متوسط | بالا | بالا |
| هزینه | کم | زیاد | متوسط | زیاد | زیاد |
| ظرفیت | ۱۰۰٪ | ۵۰٪ | (N-1)/N | (N-2)/N | ۵۰٪ |
| حداقل دیسک | ۲ | ۲ | ۳ | ۴ | ۴ |

**توصیه:** RAID 1 برای سیستمی، RAID 10 برای کارایی+افزونگی، RAID 5 برای تعادل هزینه/ظرفیت، RAID 6 برای حجم زیاد.

**Software در برابر Hardware RAID:**

| ویژگی | Software (mdadm) | Hardware |
|---|---|---|
| هزینه | رایگان | کارت گران |
| کارایی | خوب | بهتر |
| CPU | مصرف می‌کند | کارت |
| انعطاف | بالا | محدود |

**RAID در برابر LVM:**

| ویژگی | RAID | LVM |
|---|---|---|
| افزونگی | بله | با mirror/thin |
| انعطاف اندازه | محدود | بالا |
| snapshot | خیر | بله |
| ترکیب | عالی با هم | عالی با هم |

### اشکال شایع

**اشکال ۱: RAID به‌عنوان جایگزین بکاپ.**
RAID خرابی دیسک را پوشش می‌دهد، نه حذف اشتباهی! راه‌حل: بکاپ جدا.

**اشکال ۲: نبود hot spare.**
در خرابی، بازیابی دستی لازم است. راه‌حل: hot spare.

**اشکال ۳: RAID 5 با دیسک‌های بزرگ.**
بازسازی طولانی و پرخطر (URE). راه‌حل: RAID 6 یا 10.

**اشکال ۴: فراموش کردن ذخیرهٔ پیکربندی mdadm.**
بعد از reboot، RAID شناسایی نمی‌شود. راه‌حل: `mdadm.conf`.

**اشکال ۵: اشتباه در اندازهٔ chunk.**
کارایی پایین. راه‌حل: chunk مناسب.

**اشکال ۶: نبود پایش.**
خرابی دیسک دیده نمی‌شود. راه‌حل: `mdadm --monitor`.

### نکات مهم (Tips & Tricks)

- **RAID بکاپ نیست — همیشه بکاپ جدا داشته باشید.**
- **`/proc/mdstat` وضعیت RAID را نشان می‌دهد.**
- **`mdadm --monitor` برای هشدار خرابی.**
- **برای RAID نرم، از پارتیشن‌های نوع `fd` (Linux RAID) استفاده کنید.**
- **بازسازی (rebuild) زمان‌بر است؛ صبر کنید.**
- **hot spare کار را خودکار می‌کند.**
- **`mdadm --detail --scan` پیکربندی را تولید می‌کند.**
- **`update-initramfs` بعد از تغییر mdadm.conf.**
- **`--bitmap` بازسازی سریع‌تر.**
- **`echo 200000 > /proc/sys/dev/raid/speed_limit_max` سرعت rebuild.**
- **`--assume-clean` برای ساخت سریع (احتیاط).**
- **`mdadm --grow` آرایه را بزرگ می‌کند.**

### خلاصه فصل

RAID چند دیسک را برای کارایی/افزونگی ترکیب می‌کند. سطوح اصلی: 0 (سرعت)، 1 (آینه)، 5 (parity)، 6 (double parity)، 10 (ترکیبی). `mdadm` برای Software RAID است. RAID جایگزین بکاپ نیست. پیکربندی را در `mdadm.conf` ذخیره کنید و برای هشدار از `--monitor` استفاده کنید. hot spare بازیابی را خودکار می‌کند.

---

## فصل ۶۸: تنظیم دسترسی دستگاه ذخیره‌سازی

### مقدمه

دسترسی به دستگاه‌های ذخیره‌سازی و بهینه‌سازی آن‌ها بر کارایی اثر می‌گذارد. تصور کنید یک SSD دارید اما I/O scheduler اشتباه، کارایی را نصف کرده.

این فصل تنظیمات پیشرفتهٔ دسترسی را پوشش می‌دهد.

### مفاهیم اساسی

**مفاهیم کلیدی:**

- **I/O Scheduler:** الگوریتم زمان‌بندی درخواست‌های دیسک.
- **Queue Depth:** عمق صف درخواست‌ها.
- **Read-ahead:** خواندن پیش‌دستانه.
- **SATA/SCSI/NVMe:** رابط‌های ذخیره‌سازی.

**I/O Schedulers:**

| Scheduler | مناسب |
|---|---|
| `none`/`noop` | SSD، NVMe |
| `mq-deadline` | عمومی، سرور |
| `bfq` | دسکتاپ |
| `kyber` | سرور سریع |

### مثال‌های عملی

**دیدن و تنظیم scheduler:**

```bash
# دیدن scheduler فعلی
cat /sys/block/sda/queue/scheduler

# تغییر (موقت)
echo mq-deadline | sudo tee /sys/block/sda/queue/scheduler

# دائمی (GRUB)
# GRUB_CMDLINE_LINUX="elevator=mq-deadline"

# برای NVMe
cat /sys/block/nvme0n1/queue/scheduler

# udev rule برای دائمی
# /etc/udev/rules.d/60-scheduler.rules
# ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/scheduler}="mq-deadline"
```

**تنظیم read-ahead:**

```bash
# دیدن
cat /sys/block/sda/queue/read_ahead_kb

# تنظیم
echo 2048 | sudo tee /sys/block/sda/queue/read_ahead_kb
```

**اطلاعات دستگاه:**

```bash
# فهرست دستگاه‌های بلوکی
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT
lsblk -o NAME,ROTA,SIZE    # ROTA=0 SSD، ROTA=1 HDD

# اطلاعات دیسک
sudo hdparm -I /dev/sda

# تست سرعت
sudo hdparm -tT /dev/sda

# اطلاعات NVMe
sudo nvme list
sudo nvme smart-log /dev/nvme0
sudo nvme id-ctrl /dev/nvme0
```

**پایش I/O:**

```bash
iostat -x 1 5
iotop
sudo iotop -o       # فقط فعال‌ها
pidstat -d 1
```

### مقایسه و الگوهای مختلف

**انتخاب scheduler:**

| دستگاه | Scheduler |
|---|---|
| SSD/NVMe | none / mq-deadline |
| HDD سرور | mq-deadline |
| دسکتاپ | bfq |
| RAID | mq-deadline |

**ابزارهای تست:**

| ابزار | کار |
|---|---|
| `hdparm -tT` | سرعت دیسک |
| `dd` | تست نوشتن |
| `fio` | تست جامع I/O |
| `iostat` | پایش I/O |
| `bonnie++` | تست جامع |

**نمونهٔ تست با dd:**

```bash
# تست نوشتن
dd if=/dev/zero of=/tmp/test bs=1M count=1024 conv=fdatasync

# تست خواندن
dd if=/tmp/test of=/dev/null bs=1M
```

**نمونهٔ تست با fio:**

```bash
fio --name=test --rw=randread --bs=4k --size=1G \
  --numjobs=4 --runtime=30 --group_reporting
```

### اشکال شایع

**اشکال ۱: scheduler اشتباه برای SSD.**
`bfq` روی SSD کارایی را کم می‌کند. راه‌حل: `none`/`mq-deadline`.

**اشکال ۲: read-ahead زیاد.**
حافظه هدر می‌رود. راه‌حل: مقدار مناسب.

**اشکال ۳: تست نوشتنی روی دیسک تولیدی.**
داده خراب می‌شود. راه‌حل: روی فایل موقت.

**اشکال ۴: نادیده گرفتن TRIM.**
SSD کند می‌شود. راه‌حل: fstrim.

**اشکال ۵: اشتباه در queue depth.**
کارایی پایین. راه‌حل: تنظیم.

**اشکال ۶: نبود پایش I/O.**
گلوگاه دیده نمی‌شود. راه‌حل: iostat/iotop.

### نکات مهم (Tips & Tricks)

- **برای NVMe، scheduler معمولاً `none` بهترین است.**
- **`fstrim.timer` را برای SSD فعال کنید.**
- **`lsblk -o NAME,ROTA` تشخیص HDD (۱) از SSD (۰).**
- **`hdparm -B 254` برای جلوگیری از پارک هد (HDD).**
- **`iotop` برای دیدن I/O هر فرآیند.**
- **`blkdiscard` برای پاک‌سازی امن SSD.**
- **`udev` rules برای تنظیم دائمی.**
- **`fio` تست جامع I/O.**
- **`iostat -x` ستون `%util` اشباع.**
- **`nvme` برای مدیریت NVMe.**
- **`smartctl` سلامت.**
- **برای RAID، chunk size مهم است.**

### خلاصه فصل

تنظیم دسترسی ذخیره‌سازی شامل انتخاب I/O Scheduler (مناسب SSD/HDD)، read-ahead و مدیریت TRIM است. `lsblk`, `hdparm`, `nvme` اطلاعات دستگاه را می‌دهند. برای SSD `none`/`mq-deadline` و برای HDD `mq-deadline` مناسب است. TRIM را برای SSD فعال نگه دارید. `fio` و `dd` ابزارهای تست‌اند.

---

## فصل ۶۹: Logical Volume Manager

### مقدمه

**LVM (Logical Volume Manager)** لایه‌ای بین فایل‌سیستم و دیسک فیزیکی ایجاد می‌کند و انعطاف فوق‌العاده‌ای می‌دهد: بزرگ/کوچک کردن آنلاین، snapshot و ترکیب دیسک‌ها.

تصور کنید یک سرور دارید و فضای یک پارتیشن تمام شده. با پارتیشن سنتی، باید سیستم را خاموش کنید و پارتیشن را بزرگ کنید. با LVM، آنلاین و در چند ثانیه.

### مفاهیم اساسی

**سه لایهٔ LVM:**

| لایه | نام | توضیح |
|---|---|---|
| **PV** | Physical Volume | دیسک/پارتیشن فیزیکی |
| **VG** | Volume Group | گروهی از PVها |
| **LV** | Logical Volume | حجم منطقی (مثل پارتیشن) |

**نمودار:**

```
دیسک فیزیکی → PV → VG → LV → فایل‌سیستم
/dev/sdb       /dev/vg0/data
```

**مزیت کلیدی:** می‌توانید LV را بدون unmount بزرگ کنید (اگر فضای VG باشد).

**مفاهیم پیشرفته:**

- **Thin provisioning:** تخصیص بر تقاضا.
- **Snapshot:** تصویر لحظه‌ای.
- **pvmove:** انتقال داده بین PVها.

### مثال‌های عملی

**ساخت LVM:**

```bash
# نصب
sudo apt install lvm2

# ۱. ساخت PV
sudo pvcreate /dev/sdb /dev/sdc

# ۲. ساخت VG
sudo vgcreate vg_data /dev/sdb /dev/sdc

# ۳. ساخت LV
sudo lvcreate -L 10G -n lv_app vg_data
sudo lvcreate -l 100%FREE -n lv_data vg_data

# ۴. فایل‌سیستم
sudo mkfs.ext4 /dev/vg_data/lv_app

# ۵. mount
sudo mount /dev/vg_data/lv_app /mnt/app
```

**بزرگ‌کردن LV (آنلاین):**

```bash
# افزودن فضای بیشتر به VG
sudo pvcreate /dev/sdd
sudo vgextend vg_data /dev/sdd

# بزرگ کردن LV
sudo lvextend -L +5G /dev/vg_data/lv_app
sudo lvextend -l +100%FREE /dev/vg_data/lv_data

# بزرگ کردن فایل‌سیستم
sudo resize2fs /dev/vg_data/lv_app    # ext4 (آنلاین)
sudo xfs_growfs /mnt/app               # XFS

# یا با هم
sudo lvextend --resizefs -L +5G /dev/vg_data/lv_app
```

**Snapshot:**

```bash
# ساخت snapshot
sudo lvcreate -L 5G -s -n lv_app_snap /dev/vg_data/lv_app

# mount snapshot
sudo mount /dev/vg_data/lv_app_snap /mnt/snap

# بازگردانی (بعد از unmount)
sudo umount /mnt/app
sudo lvconvert --merge /dev/vg_data/lv_app_snap

# حذف
sudo lvremove /dev/vg_data/lv_app_snap
```

**اطلاعات و مدیریت:**

```bash
# فهرست
pvs
vgs
lvs
pvdisplay
vgdisplay
lvdisplay

# کوچک کردن (اول umount)
sudo umount /mnt/app
sudo e2fsck -f /dev/vg_data/lv_app
sudo resize2fs /dev/vg_data/lv_app 5G
sudo lvreduce -L 5G /dev/vg_data/lv_app

# انتقال داده
sudo pvmove /dev/sdb /dev/sdd

# حذف
sudo lvremove /dev/vg_data/lv_app
sudo vgremove vg_data
sudo pvremove /dev/sdb
```

### مقایسه و الگوهای مختلف

**دستورهای LVM:**

| لایه | ساخت | نمایش | گسترش |
|---|---|---|---|
| PV | `pvcreate` | `pvs`/`pvdisplay` | — |
| VG | `vgcreate` | `vgs`/`vgdisplay` | `vgextend` |
| LV | `lvcreate` | `lvs`/`lvdisplay` | `lvextend` |

**LVM در برابر پارتیشن سنتی:**

| ویژگی | پارتیشن | LVM |
|---|---|---|
| تغییر اندازه | سخت | آسان |
| snapshot | خیر | بله |
| ترکیب دیسک | خیر | بله |
| مهاجرت | سخت | `pvmove` |
| آنلاین | خیر | بله |

**انواع LV:**

| نوع | توضیح |
|---|---|
| linear | پیش‌فرض |
| striped | کارایی (مثل RAID 0) |
| mirrored | افزونگی (مثل RAID 1) |
| thin | تخصیص بر تقاضا |

**LVM در برابر RAID:**

| سناریو | ابزار |
|---|---|
| افزونگی دیسک | RAID |
| انعطاف اندازه | LVM |
| هر دو | RAID + LVM |

### اشکال شایع

**اشکال ۱: فراموش کردن resize فایل‌سیستم.**
`lvextend` تنها کافی نیست؛ باید `resize2fs`/`xfs_growfs` هم بزنید. راه‌حل: `--resizefs`.

**اشکال ۲: کوچک کردن LV بدون umount.**
خطرناک است. راه‌حل: umount + fsck.

**اشکال ۳: نبود فضای VG.**
باید `vgextend` کنید. راه‌حل: `pvs` بررسی.

**اشکال ۴: snapshot بدون فضای کافی.**
پر می‌شود و بی‌اعتبار می‌شود. راه‌حل: فضای کافی.

**اشکال ۵: اشتباه در نام LV.**
`/dev/vg/lv` یا `/dev/mapper/vg-lv`. راه‌حل: `lvs`.

**اشکال ۶: حذف PV در حال استفاده.**
داده از دست می‌رود. راه‌حل: `pvmove` اول.

### نکات مهم (Tips & Tricks)

- **`lvs`, `vgs`, `pvs` خلاصهٔ سریع می‌دهند.**
- **`lvextend -l +100%FREE` تمام فضای آزاد را می‌گیرد.**
- **snapshot برای بکاپ سازگار (consistent) عالی است.**
- **`pvmove /dev/sdb` داده را بین PVها منتقل می‌کند.**
- **برای swap در LVM، `lvcreate -L 2G -n swap` و سپس `mkswap`.**
- **`lvresize --resizefs` هم LV و هم فایل‌سیستم را با هم تغییر می‌دهد.**
- **`lvcreate -s` snapshot می‌سازد.**
- **`vgcfgbackup` پیکربندی VG را بکاپ می‌گیرد.**
- **`lvconvert --merge` snapshot را ادغام می‌کند.**
- **Thin pool برای تخصیص بر تقاضا.**
- **`/etc/lvm/lvm.conf` پیکربندی.**
- **`vgscan`/`pvscan` LVM را اسکن می‌کند.**

### خلاصه فصل

LVM با سه لایه (PV → VG → LV) انعطاف ذخیره‌سازی می‌دهد. `pvcreate`, `vgcreate`, `lvcreate` ساخت و `lvextend`+`resize2fs` گسترش می‌دهند. snapshot برای بکاپ سازگار عالی است. `pvmove` داده را منتقل می‌کند. بعد از `lvextend` حتماً فایل‌سیستم را resize کنید (یا `--resizefs`). LVM و RAID با هم ترکیب عالی می‌سازند.

---

## فصل ۷۰: آزمایشگاه: RAID و LVM به‌صورت عملی

### مقدمه

در این آزمایشگاه ترکیبی، RAID و LVM را با هم تمرین می‌کنیم — الگویی که در سرورهای واقعی رایج است.

هدف: ساخت یک RAID 1 از دو دیسک، سپس استفاده از آن به‌عنوان PV برای LVM.

### مفاهیم اساسی

```
/dev/sdb + /dev/sdc → mdadm RAID 1 → /dev/md0 → PV → VG → LV → ext4
```

### مثال‌های عملی

**گام ۱: آماده‌سازی دیسک‌ها**

```bash
# سه دیسک مجازی اضافه کنید (۵GB هر کدام)
lsblk

# پاک‌سازی
sudo wipefs -a /dev/sdb /dev/sdc /dev/sdd
```

**گام ۲: ساخت RAID 1**

```bash
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# بررسی
cat /proc/mdstat
sudo mdadm --detail /dev/md0
```

**گام ۳: ساخت LVM روی RAID**

```bash
# PV روی RAID
sudo pvcreate /dev/md0

# VG
sudo vgcreate vg_lab /dev/md0

# LV
sudo lvcreate -L 3G -n lv_data vg_lab

# فایل‌سیستم
sudo mkfs.ext4 /dev/vg_lab/lv_data

# mount
sudo mkdir -p /mnt/lab
sudo mount /dev/vg_lab/lv_data /mnt/lab

# تست
echo "test" | sudo tee /mnt/lab/test.txt
df -h /mnt/lab
```

**گام ۴: گسترش LVM با دیسک سوم**

```bash
# افزودن /dev/sdd به RAID
sudo mdadm --add /dev/md0 /dev/sdd
sudo mdadm --grow /dev/md0 --raid-devices=3

# یا افزودن مستقیم به VG
sudo pvcreate /dev/sdd
sudo vgextend vg_lab /dev/sdd

# بزرگ کردن LV
sudo lvextend -L +2G /dev/vg_lab/lv_data
sudo resize2fs /dev/vg_lab/lv_data

df -h /mnt/lab
```

**گام ۵: تست خرابی**

```bash
# شبیه‌سازی خرابی یک دیسک
sudo mdadm --fail /dev/md0 /dev/sdb
cat /proc/mdstat        # دیسک failed نشان داده می‌شود

# جایگزینی
sudo mdadm --remove /dev/md0 /dev/sdb
sudo mdadm --add /dev/md0 /dev/sdb
cat /proc/mdstat        # rebuild شروع می‌شود
```

**گام ۶: Snapshot LVM**

```bash
# ساخت snapshot
sudo lvcreate -L 1G -s -n lv_snap /dev/vg_lab/lv_data

# mount
sudo mkdir /mnt/snap
sudo mount /dev/vg_lab/lv_snap /mnt/snap
ls /mnt/snap

# حذف snapshot
sudo umount /mnt/snap
sudo lvremove /dev/vg_lab/lv_snap
```

### مقایسه و الگوهای مختلف

**پیکربندی دائمی:**

```bash
# ذخیرهٔ RAID
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u

# fstab
echo "/dev/vg_lab/lv_data /mnt/lab ext4 defaults 0 2" | sudo tee -a /etc/fstab

# تست
sudo mount -a
```

**پاک‌سازی آزمایشگاه:**

```bash
sudo umount /mnt/lab
sudo lvremove /dev/vg_lab/lv_data
sudo vgremove vg_lab
sudo pvremove /dev/md0
sudo mdadm --stop /dev/md0
sudo mdadm --zero-superblock /dev/sdb /dev/sdc /dev/sdd
```

### اشکال شایع

**اشکال ۱: فراموش کردن resize بعد از lvextend.**
فضای اضافه استفاده نمی‌شود. راه‌حل: `resize2fs`.

**اشکال ۲: نبود پیکربندی mdadm.**
RAID بعد از reboot شناسایی نمی‌شود. راه‌حل: `mdadm.conf`.

**اشکال ۳: خطای sync در RAID.**
تا اتمام rebuild صبر کنید. راه‌حل: `cat /proc/mdstat`.

**اشکال ۴: اشتباه در پاک‌سازی.**
ترتیب مهم است (LV → VG → PV → md). راه‌حل: ترتیب.

**اشکال ۵: snapshot بدون فضای کافی.**
بی‌اعتبار می‌شود. راه‌حل: فضای کافی.

**اشکال ۶: فراموش کردن update-initramfs.**
RAID در بوت شناسایی نمی‌شود. راه‌حل: update.

### نکات مهم (Tips & Tricks)

- **همیشه `cat /proc/mdstat` را بعد از هر عملیات چک کنید.**
- **برای بازسازی سریع‌تر، `--bitmap` را در نظر بگیرید.**
- **LVM روی RAID ترکیب قدرتمندی است.**
- **`lvdisplay`, `vgdisplay`, `pvdisplay` جزئیات کامل.**
- **در پایان، پیکربندی را در `mdadm.conf` و `fstab` ذخیره کنید.**
- **`systemctl restart mdmonitor` برای پایش.**
- **snapshot برای بکاپ سازگار.**
- **`pvmove` انتقال داده.**
- **`wipefs` پاک‌سازی امضا.**
- **`vgs`/`lvs`/`pvs` خلاصه.**
- **`mdadm --grow` آرایه را بزرگ می‌کند.**
- **`update-initramfs -u` بعد از تغییر mdadm.**

### خلاصه فصل

در این آزمایشگاه RAID 1 (با mdadm) + LVM (روی /dev/md0) را ترکیب کردیم: ساخت PV/VG/LV، گسترش آنلاین با `lvextend`+`resize2fs`، snapshot و تست خرابی/بازیابی. نکات کلیدی: ذخیرهٔ پیکربندی در `mdadm.conf`، resize فایل‌سیستم بعد از lvextend و ترتیب صحیح پاک‌سازی.

---

## منابع و مراجع

- man pages: `man mdadm`, `man lvm`, `man pvcreate`, `man lvcreate`, `man hdparm`, `man fio`
- mdadm Wiki: https://raid.wiki.kernel.org
- LVM HOWTO: https://tldp.org/HOWTO/LVM-HOWTO/
- Red Hat LVM Guide: https://access.redhat.com/documentation/
- fio Documentation: https://fio.readthedocs.io

**بعدی:** [فایل ۲۰ — پیکربندی شبکه](20-Networking-Configuration.md)
