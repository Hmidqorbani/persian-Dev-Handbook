# فایل ۱۷ — راه‌اندازی سیستم

**مدت یادگیری**: ۶ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: فایل ۰۳ (معماری سیستم)، فایل ۱۶ (کرنل)

---

## فهرست محتوا

- [فصل ۶۱: سفارشی‌سازی راه‌اندازی سیستم](#فصل-۶۱-سفارشیسازی-راهاندازی-سیستم)
- [فصل ۶۲: بازیابی سیستم](#فصل-۶۲-بازیابی-سیستم)
- [فصل ۶۳: بوت‌لودرهای جایگزین](#فصل-۶۳-بوتلودرهای-جایگزین)

---

## فصل ۶۱: سفارشی‌سازی راه‌اندازی سیستم

### مقدمه

در LPIC-1 فرآیند بوت را در سطح پایه دیدیم. در LPIC-2 عمیق‌تر می‌شویم: سفارشی‌سازی systemd، مدیریت سرویس‌ها و بهینه‌سازی بوت.

تصور کنید سرور کند بوت می‌شود و باید سرویس‌های غیرضروری را حذف کنید، یا یک سرویس سفارشی بسازید که در بوت اجرا شود.

### مفاهیم اساسی

**systemd:** مدیر سیستم و سرویس پیش‌فرض اکثر توزیع‌ها.

**واحدها (Units):**

| نوع unit | کار | پسوند |
|---|---|---|
| `.service` | سرویس | .service |
| `.target` | گروهی از units | .target |
| `.socket` | سوکت (فعال‌سازی بر تقاضا) | .socket |
| `.timer` | زمان‌بند | .timer |
| `.mount` | mount | .mount |
| `.path` | پایش مسیر | .path |
| `.device` | دستگاه | .device |

**محل units:**

- `/usr/lib/systemd/system/` — units پکیج‌ها.
- `/etc/systemd/system/` — units سفارشی و override.
- `/run/systemd/system/` — runtime.

**بخش‌های یک unit:**

```
[Unit]       ← توصیف، وابستگی
[Service]    ← نحوهٔ اجرا
[Install]    ← فعال‌سازی
```

### مثال‌های عملی

**مدیریت سرویس:**

```bash
# شروع/توقف/ری‌استارت
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx       # بازخوانی پیکربندی

# فعال/غیرفعال در بوت
sudo systemctl enable nginx
sudo systemctl disable nginx
sudo systemctl enable --now nginx  # enable + start

# وضعیت
systemctl status nginx
systemctl is-active nginx
systemctl is-enabled nginx
systemctl is-failed nginx

# فهرست
systemctl list-units --type=service
systemctl list-units --failed
systemctl list-unit-files --type=service
```

**ساخت unit سفارشی:**

```bash
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
After=network.target
Requires=network.target

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/start
ExecReload=/bin/kill -HUP $MAINPID
ExecStop=/opt/myapp/bin/stop
Restart=on-failure
RestartSec=5
Environment=NODE_ENV=production
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
```

**Override بدون ویرایش فایل اصلی:**

```bash
sudo systemctl edit nginx
# فایل override در /etc/systemd/system/nginx.service.d/ ساخته می‌شود
```

**تحلیل بوت:**

```bash
systemd-analyze
systemd-analyze blame
systemd-analyze critical-chain
systemd-analyze plot > boot.svg
```

### مقایسه و الگوهای مختلف

**انواع Service:**

| Type | توضیح | مثال |
|---|---|---|
| `simple` | فرآیند پیش‌زمینه | وب‌سرور |
| `forking` | فرآیند fork می‌کند | سرویس‌های قدیمی |
| `oneshot` | یک‌بار اجرا | اسکریپت راه‌اندازی |
| `notify` | با sd_notify اطلاع می‌دهد | سرویس‌های مدرن |
| `idle` | بعد از بقیه | کارهای تأخیری |
| `dbus` | فعال‌سازی با D-Bus | سرویس‌های D-Bus |

**`Restart` options:**

| مقدار | رفتار |
|---|---|
| `no` | هرگز |
| `on-failure` | فقط در خطا |
| `always` | همیشه |
| `on-abnormal` | در سیگنال/تایم‌اوت |
| `on-abort` | در abort |

**وابستگی‌ها:**

| دستور | معنا |
|---|---|
| `After=` | بعد از (ترتیب) |
| `Before=` | قبل از |
| `Requires=` | نیاز (اگر شکست، این هم) |
| `Wants=` | تمایل (اگر شکست، این ادامه) |
| `Conflicts=` | تعارض |

### اشکال شایع

**اشکال ۱: فراموش کردن `daemon-reload`.**
تغییرات unit اعمال نمی‌شود. راه‌حل: بعد از هر تغییر.

**اشکال ۲: مسیر نسبی در ExecStart.**
باید مطلق باشد. راه‌حل: مسیر کامل.

**اشکال ۳: نبود `[Install]`.**
`enable` کار نمی‌کند. راه‌حل: `[Install]` + `WantedBy`.

**اشکال ۴: وابستگی اشتباه در `After`.**
سرویس زودتر اجرا می‌شود. راه‌حل: `After`/`Requires`.

**اشکال ۵: سرویس در `/lib/` به‌جای `/etc/`.**
`/lib/` برای پکیج‌هاست. راه‌حل: `/etc/systemd/system/`.

**اشکال ۶: نبود User.**
سرویس با ریشه اجرا می‌شود (خطر). راه‌حل: `User=`.

### نکات مهم (Tips & Tricks)

- **`systemctl list-dependencies nginx` وابستگی‌ها را نشان می‌دهد.**
- **`systemctl cat nginx` محتوای کامل unit.**
- **`systemctl show nginx` همهٔ خواص.**
- **برای override از `systemctl edit` استفاده کنید (نه ویرایش مستقیم).**
- **`systemd-analyze critical-chain` زنجیرهٔ بوت را نشان می‌دهد.**
- **سرویس‌های سفارشی در `/etc/systemd/system/`.**
- **`systemctl mask X` سرویس را کاملاً غیرفعال می‌کند.**
- **`RestartSec` تأخیر restart.**
- **`Environment=` متغیر محیطی.**
- **`ExecStartPre`/`ExecStartPost` دستورهای قبل/بعد.**
- **`systemctl daemon-reexec` systemd را دوباره اجرا می‌کند.**
- **`journalctl -u X` لاگ سرویس.**

### خلاصه فصل

systemd با units (service, target, socket, timer) سرویس‌ها را مدیریت می‌کند. `systemctl` دستور اصلی است. units سفارشی در `/etc/systemd/system/` و override با `systemctl edit`. بعد از تغییر `daemon-reload` لازم است. `Type` و `Restart` رفتار سرویس را تعیین می‌کنند. `systemd-analyze` بوت را تحلیل می‌کند.

---

## فصل ۶۲: بازیابی سیستم

### مقدمه

گاهی سیستم بوت نمی‌شود یا دچار مشکل جدی می‌شود. تصور کنید رمز ریشه را فراموش کرده‌اید یا fstab خراب شده و سیستم بوت نمی‌شود. دانستن روش‌های **بازیابی (recovery)** برای هر مدیر سیستمی حیاتی است.

این فصل سناریوهای بازیابی را پوشش می‌دهد.

### مفاهیم اساسی

**سناریوهای بازیابی:**

- فراموش کردن رمز ریشه.
- خرابی fstab (بوت نمی‌شود).
- خرابی GRUB.
- خرابی فایل‌سیستم.
- خرابی سرویس‌های حیاتی.

**ابزارها:** rescue mode، single-user mode، live CD/USB، chroot.

**حالت‌های بازیابی:**

| حالت | توضیح |
|---|---|
| `rescue.target` | بازیابی سرویس‌ها |
| `emergency.target` | حداقل (فقط root fs) |
| `single-user` | تعمیرات |
| Live CD/USB | خرابی جدی |

### مثال‌های عملی

**بازیابی رمز ریشه (GRUB):**

```
۱. در منوی GRUB، کلید 'e' را بزنید
۲. در خط linux، 'ro quiet splash' را حذف و 'rw init=/bin/bash' اضافه کنید
۳. Ctrl+X یا F10 برای بوت
۴. حالا در shell ریشه هستید:
   mount -o remount,rw /
   passwd root
   exec /sbin/init      # یا reboot -f
```

**بازیابی fstab خراب:**

```
۱. در GRUB، 'e' بزنید
۲. 'rw init=/bin/bash' اضافه کنید
۳. بوت کنید
۴. /etc/fstab را اصلاح کنید:
   mount -o remount,rw /
   vi /etc/fstab
۵. ری‌استارت
```

**بازیابی GRUB:**

```bash
# از live CD
sudo mount /dev/sda1 /mnt          # پارتیشن root
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo chroot /mnt
grub-install /dev/sda
update-grub
exit
sudo reboot
```

**بازیابی فایل‌سیستم:**

```bash
# از live CD
sudo fsck -f /dev/sda1

# یا از rescue mode
sudo systemctl rescue
```

**بازیابی بوت (systemd):**

```bash
# بوت به rescue
sudo systemctl rescue
sudo systemctl emergency

# در GRUB: systemd.unit=rescue.target
```

**chroot برای تعمیر:**

```bash
# از live CD
sudo mount /dev/sda2 /mnt
sudo mount /dev/sda1 /mnt/boot
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo chroot /mnt
# حالا در محیط سیستم نصب‌شده هستید
```

### مقایسه و الگوهای مختلف

**روش‌های بازیابی:**

| روش | کاربرد | پیچیدگی |
|---|---|---|
| **rescue.target** | بازیابی سرویس‌ها | ساده |
| **emergency.target** | حداقل | متوسط |
| **single-user** | تعمیرات | ساده |
| **Live CD/USB** | خرابی جدی | متوسط |
| **chroot** | تعمیر از بیرون | پیشرفته |

**بوت به rescue:**

| روش | دستور |
|---|---|
| از GRUB | `systemd.unit=rescue.target` |
| از systemd | `systemctl rescue` |
| از single | `init 1` |

**ابزارهای تعمیر:**

| ابزار | کار |
|---|---|
| `chroot` | محیط تعمیر |
| `fsck` | بررسی فایل‌سیستم |
| `grub-install` | نصب مجدد GRUB |
| `rescue` mode | بازیابی |

### اشکال شایع

**اشکال ۱: فراموش کردن remount rw.**
فایل‌سیستم فقط‌خواندنی است؛ نمی‌توانید تغییر دهید. راه‌حل: `mount -o remount,rw /`.

**اشکال ۲: اشتباه در fstab دوباره.**
سیستم باز هم بوت نمی‌شود. راه‌حل: با دقت اصلاح کنید.

**اشکال ۳: نبود live USB.**
برای خرابی جدی به آن نیاز دارید. راه‌حل: همیشه آماده.

**اشکال ۴: نبود بکاپ.**
بدون بکاپ، بازیابی محدود است. راه‌حل: بکاپ.

**اشکال ۵: chroot بدون mount کردن /proc.**
بعضی دستورها کار نمی‌کنند. راه‌حل: mount --bind.

**اشکال ۶: اشتباه در پارتیشن.**
بوت نمی‌شود. راه‌حل: `lsblk` قبل از mount.

### نکات مهم (Tips & Tricks)

- **همیشه یک live USB آماده داشته باشید.**
- **قبل از تغییر fstab، از آن بکاپ بگیرید.**
- **`mount -o remount,rw /` اولین دستور در حالت بازیابی.**
- **`chroot` محیط تعمیر کامل می‌دهد.**
- **`fsck` را روی فایل‌سیستم unmount اجرا کنید.**
- **رمز ریشه را در جای امن نگه دارید.**
- **`systemd.unit=rescue.target` در GRUB.**
- **`init=/bin/bash` برای shell ریشه.**
- **`mount --bind /dev /mnt/dev` برای chroot.**
- **`rescue.target` سرویس‌های اساسی را بالا می‌آورد.**
- **`emergency.target` حداقلی‌تر.**
- **در chroot، `passwd` رمز را تغییر می‌دهد.**

### خلاصه فصل

بازیابی سیستم شامل رفع رمز فراموش‌شده، fstab خراب، GRUB آسیب‌دیده و فایل‌سیستم خراب است. با `init=/bin/bash` در GRUB وارد shell ریشه شوید و با `remount,rw` تغییر دهید. `rescue.target`/`emergency.target` و live CD/chroot ابزارهای اصلی‌اند. همیشه live USB و بکاپ داشته باشید.

---

## فصل ۶۳: بوت‌لودرهای جایگزین

### مقدمه

**GRUB2** رایج‌ترین بوت‌لودر است، اما جایگزین‌هایی هم وجود دارد: **systemd-boot**, **LILO**, **SYSLINUX**, **rEFInd**. در این فصل آن‌ها را می‌شناسید.

تصور کنید سیستم UEFI دارید و می‌خواهید بوت ساده‌تر و سریع‌تری داشته باشید — systemd-boot گزینهٔ خوبی است.

### مفاهیم اساسی

**انواع بوت‌لودر:**

| بوت‌لودر | ویژگی | فریمور |
|---|---|---|
| **GRUB2** | استاندارد، همه‌جا | BIOS + UEFI |
| **systemd-boot** | سبک، ساده | فقط UEFI |
| **LILO** | قدیمی (تاریخی) | BIOS |
| **SYSLINUX** | برای USB/CD | BIOS |
| **rEFInd** | UEFI، زیبا | UEFI |

**مفاهیم UEFI:**

- **EFI System Partition (ESP):** پارتیشن FAT32.
- **efibootmgr:** مدیریت ورودی‌های بوت UEFI.
- **Secure Boot:** تأیید امضای بوت‌لودر.

### مثال‌های عملی

**systemd-boot:**

```bash
# نصب (روی UEFI)
sudo bootctl install

# پیکربندی /boot/loader/loader.conf:
# default arch
# timeout 3
# editor no

# /boot/loader/entries/arch.conf:
# title Arch Linux
# linux /vmlinuz-linux
# initrd /initramfs-linux.img
# options root=UUID=... rw

# وضعیت
bootctl status
bootctl list
```

**GRUB (مرور):**

```bash
# نصب
sudo grub-install /dev/sda

# بازتولید پیکربندی
sudo update-grub

# UEFI
sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi
```

**LILO (تاریخی):**

```bash
# /etc/lilo.conf
# boot=/dev/sda
# image=/boot/vmlinuz
#   label=linux
#   root=/dev/sda1
sudo lilo -v
```

**efibootmgr:**

```bash
# نمایش ورودی‌ها
sudo efibootmgr
sudo efibootmgr -v

# تغییر ترتیب
sudo efibootmgr -o 0001,0002

# حذف ورودی
sudo efibootmgr -b 0001 -B

# افزودن
sudo efibootmgr -c -d /dev/sda -p 1 -L "Linux" -l '\EFI\arch\grubx64.efi'
```

### مقایسه و الگوهای مختلف

**مقایسهٔ بوت‌لودرها:**

| ویژگی | GRUB2 | systemd-boot | LILO |
|---|---|---|---|
| BIOS | بله | خیر | بله |
| UEFI | بله | بله | خیر |
| پیچیدگی | بالا | کم | کم |
| پشتیبانی | گسترده | systemd | منسوخ |
| منو | گرافیکی/متنی | متنی | متنی |
| پیکربندی | تولیدشده | دستی | دستی |

**توصیه:** GRUB2 برای همه‌منظوره، systemd-boot برای UEFI ساده.

**ابزارهای UEFI:**

| ابزار | کار |
|---|---|
| `efibootmgr` | ورودی‌های بوت |
| `bootctl` | systemd-boot |
| `efivar` | متغیرهای EFI |

### اشکال شایع

**اشکال ۱: نصب بوت‌لودر روی پارتیشن.**
باید روی دیسک (MBR) یا ESP (UEFI) نصب شود. راه‌حل: آدرس درست.

**اشکال ۲: پیکربندی اشتباه.**
سیستم بوت نمی‌شود. راه‌حل: تست و بازیابی.

**اشکال ۳: فراموش کردن ESP.**
در UEFI باید پارتیشن EFI mount باشد. راه‌حل: `mount /boot/efi`.

**اشکال ۴: نبود fallback.**
همیشه یک بوت‌لودر پشتیبان داشته باشید. راه‌حل: fallback.

**اشکال ۵: Secure Boot.**
بوت‌لودر بدون امضا رد می‌شود. راه‌حل: امضا یا غیرفعال.

**اشکال ۶: ESP کوچک.**
چند کرنل جا نمی‌شود. راه‌حل: ESP بزرگ‌تر (۵۰۰MB).

### نکات مهم (Tips & Tricks)

- **`efibootmgr` ترتیب بوت UEFI را مدیریت می‌کند.**
- **برای UEFI، `/boot/efi` باید `vfat` باشد.**
- **`bootctl` برای systemd-boot.**
- **در GRUB، `grub-mkconfig -o /boot/grub/grub.cfg`.**
- **همیشه یک USB قابل بوت به‌عنوان پشتیبان داشته باشید.**
- **`os-prober` سیستم‌عامل‌های دیگر را تشخیص می‌دهد.**
- **`bootctl install` systemd-boot را نصب می‌کند.**
- **`bootctl update` را بعد از به‌روزرسانی systemd.**
- **ESP باید FAT32 باشد.**
- **`efibootmgr -o` ترتیب را تغییر می‌دهد.**
- **برای Secure Boot، کرنل امضاشده.**
- **`refind` گزینهٔ گرافیکی UEFI.**

### خلاصه فصل

بوت‌لودرهای اصلی: GRUB2 (استاندارد)، systemd-boot (UEFI ساده)، LILO (تاریخی)، SYSLINUX (USB/CD) و rEFInd (UEFI زیبا). هر کدام مزایا و محدودیت‌هایی دارند. در UEFI، پارتیشن ESP و `efibootmgr` مهم‌اند. همیشه یک بوت‌لودر/دستگاه پشتیبان داشته باشید. ESP باید FAT32 و بزرگ کافی باشد.

---

## منابع و مراجع

- man pages: `man systemctl`, `man systemd.service`, `man bootctl`, `man grub`, `man efibootmgr`
- systemd Documentation: https://www.freedesktop.org/wiki/Software/systemd/
- systemd-boot: https://www.freedesktop.org/software/systemd/man/systemd-boot.html
- GNU GRUB Manual: https://www.gnu.org/software/grub/manual/
- UEFI Specification: https://uefi.org/specifications

**بعدی:** [فایل ۱۸ — فایل‌سیستم و دستگاه‌ها](18-Filesystem-and-Devices.md)
