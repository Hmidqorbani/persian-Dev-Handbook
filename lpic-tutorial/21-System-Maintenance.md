# فایل ۲۱ — نگهداری سیستم

**مدت یادگیری**: ۶ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: فایل ۱۸ (فایل‌سیستم)

---

## فهرست محتوا

- [فصل ۷۴: ساخت و نصب برنامه از سورس](#فصل-۷۴-ساخت-و-نصب-برنامه-از-سورس)
- [فصل ۷۵: عملیات پشتیبان‌گیری](#فصل-۷۵-عملیات-پشتیبانگیری)
- [فصل ۷۶: اطلاع‌رسانی به کاربران دربارهٔ رویدادهای سیستم](#فصل-۷۶-اطلاعرسانی-به-کاربران-دربارهٔ-رویدادهای-سیستم)

---

## فصل ۷۴: ساخت و نصب برنامه از سورس

### مقدمه

گاهی برنامه‌ای در مخازن نیست یا نسخهٔ خاصی لازم دارید. تصور کنید یک نرم‌افزار آخرین نسخه را می‌خواهید اما مخزن قدیمی است. در این صورت باید از **سورس (source)** بسازید.

این فصل فرآیند را پوشش می‌دهد و در آزمون LPIC-2 می‌آید.

### مفاهیم اساسی

**فرآیند استاندارد:**

```
۱. دریافت سورس (tar.gz)
۲. استخراج
۳. ./configure (بررسی پیش‌نیازها)
۴. make (کامپایل)
۵. make install (نصب)
```

**ابزارهای لازم:** `gcc`, `make`, `autoconf`, `automake`, `libtool`.

**سه روش ساخت:**

| روش | فایل |
|---|---|
| Autotools | `configure` + `Makefile` |
| CMake | `CMakeLists.txt` |
| Meson | `meson.build` |

**محل نصب:**

| prefix | مسیر |
|---|---|
| `/usr/local` | نرم‌افزار محلی (توصیه) |
| `/opt` | نرم‌افزار اختیاری |
| `/usr` | سیستمی (تداخل!) |

### مثال‌های عملی

**روش Autotools:**

```bash
# نصب ابزارها
sudo apt install build-essential

# دریافت و استخراج
wget https://example.com/app-1.0.tar.gz
tar -xzf app-1.0.tar.gz
cd app-1.0

# مطالعهٔ README و INSTALL
cat README
cat INSTALL

# پیکربندی
./configure --prefix=/usr/local
./configure --help    # گزینه‌ها
./configure --prefix=/usr/local --enable-feature --disable-other

# کامپایل
make -j$(nproc)

# تست (اختیاری)
make check

# نصب
sudo make install

# پاک‌سازی فایل‌های کامپایل
make clean
```

**روش CMake:**

```bash
mkdir build && cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local
make -j$(nproc)
sudo make install
```

**روش Meson:**

```bash
meson setup build --prefix=/usr/local
cd build
ninja
sudo ninja install
```

**حذف برنامهٔ نصب‌شده از سورس:**

```bash
# اگر Makefile دارد
sudo make uninstall

# یا از فهرست نصب
cat install_manifest.txt | sudo xargs rm

# با checkinstall
sudo checkinstall    # به‌جای make install
sudo dpkg -r app     # حذف با پکیج
```

### مقایسه و الگوهای مختلف

**روش‌های ساخت:**

| روش | مزیت | معایب |
|---|---|---|
| Autotools | همه‌جا | پیچیده |
| CMake | مدرن، چندسکویی | یادگیری |
| Meson | سریع | جدید |
| `checkinstall` | ساخت .deb/.rpm | محدود |

**نصب از سورس در برابر پکیج:**

| ویژگی | سورس | پکیج |
|---|---|---|
| نسخه | آخرین | مخزن |
| مدیریت | دستی | خودکار |
| وابستگی | دستی | خودکار |
| به‌روزرسانی | دستی | خودکار |
| حذف | سخت | آسان |

**ساخت پکیج با checkinstall:**

```bash
sudo apt install checkinstall
sudo checkinstall --pkgname=app --pkgversion=1.0
```

### اشکال شایع

**اشکال ۱: نبود پیش‌نیازها.**
`./configure` خطا می‌دهد؛ `-dev`ها را نصب کنید. راه‌حل: نصب پیش‌نیاز.

**اشکال ۲: نبود `-j`.**
کامپایل کند می‌شود. راه‌حل: `make -j$(nproc)`.

**اشکال ۳: نصب در `/usr`.**
ممکن است با پکیج‌های سیستم تداخل کند؛ `/usr/local` بهتر. راه‌حل: prefix درست.

**اشکال ۴: نبود uninstall.**
پیگیری نصب سخت می‌شود. راه‌حل: checkinstall.

**اشکال ۵: فراموش کردن `ldconfig`.**
کتابخانه جدید پیدا نمی‌شود. راه‌حل: `ldconfig`.

**اشکال ۶: نبود فضای کافی.**
کامپایل شکست می‌خورد. راه‌حل: فضا.

### نکات مهم (Tips & Tricks)

- **همیشه `--prefix=/usr/local` بگذارید.**
- **`checkinstall` پکیج قابل حذف می‌سازد.**
- **`make -j$(nproc)` سرعت را بالا می‌برد.**
- **پیش‌نیازها را با `apt build-dep package` نصب کنید.**
- **`make clean` فایل‌های موقت را پاک می‌کند.**
- **برای حذف کامل، فهرست فایل‌ها را نگه دارید.**
- **`./configure --help` گزینه‌ها را نشان می‌دهد.**
- **`make -j$(nproc) 2>&1 | tee build.log` لاگ.**
- **`ldconfig` بعد از نصب کتابخانه.**
- **`DESTDIR=/tmp/pkg make install` نصب موقت.**
- **`PKG_CONFIG_PATH` برای کتابخانه‌های سفارشی.**
- **`stow` مدیریت نصب از سورس.**

### خلاصه فصل

ساخت از سورس شامل استخراج، `./configure`، `make` و `make install` است. ابزارهای build ضروری‌اند. از `--prefix=/usr/local` استفاده کنید تا با سیستم تداخل نکنید. `checkinstall` پکیج قابل حذف می‌سازد. `make -j$(nproc)` سرعت را بالا می‌برد. CMake و Meson روش‌های مدرن‌اند.

---

## فصل ۷۵: عملیات پشتیبان‌گیری

### مقدمه

**بکاپ (Backup)** مهم‌ترین کار نگهداری است. تصور کنید دیسک سرور خراب می‌شود و آخرین بکاپ یک ماه پیش است — یک ماه داده از دست می‌رود. بدون بکاپ، هر خرابی می‌تواند فاجعه باشد.

در این فصل ابزارها و استراتژی‌های بکاپ را یاد می‌گیرید.

### مفاهیم اساسی

**انواع بکاپ:**

| نوع | توضیح |
|---|---|
| **Full** | کامل |
| **Incremental** | فقط تغییرات از آخرین بکاپ |
| **Differential** | تغییرات از آخرین full |

**قانون 3-2-1:** ۳ نسخه، ۲ رسانهٔ مختلف، ۱ نسخهٔ خارج از محل.

**ابزارها:** `tar`, `rsync`, `dd`, `dump`/`restore`, `borg`, `restic`.

**استراتژی‌های بکاپ:**

| استراتژی | توضیح |
|---|---|
| Full روزانه | ساده، فضای زیاد |
| Full هفتگی + Incremental | تعادل |
| Snapshot (LVM) | سازگار، سریع |
| Dedup (borg) | فضای کم |

### مثال‌های عملی

**بکاپ با tar:**

```bash
# بکاپ کامل
tar -czf backup-$(date +%F).tar.gz /home/user

# بکاپ با حفظ مجوز
sudo tar -czpf backup.tar.gz /etc

# استخراج
tar -xzf backup.tar.gz
tar -xzf backup.tar.gz -C /restore

# فقط فایل‌های تغییر‌یافته (با find)
find /home -mtime -1 | tar -czf incr.tar.gz -T -

# بکاپ به سرور دیگر
tar -czf - /home | ssh user@remote "cat > /backup/home.tar.gz"
```

**بکاپ با rsync:**

```bash
# بکاپ محلی (آینه)
rsync -av --delete /home/user/ /backup/user/

# بکاپ از راه دور
rsync -avz -e ssh /home/user/ user@remote:/backup/

# با حذف فایل‌های حذف‌شده
rsync -av --delete /src/ /dst/

# فقط تغییرات (incremental با link)
rsync -av --link-dest=/backup/last /home/ /backup/current/

# dry-run
rsync -av --dry-run /src/ /dst/

# با پیشرفت
rsync -av --progress /src/ /dst/

# محدود کردن پهنای باند
rsync -av --bwlimit=1000 /src/ /dst/
```

**بکاپ با dd:**

```bash
# تصویر کامل دیسک
sudo dd if=/dev/sda of=/backup/disk.img bs=4M status=progress

# بازگردانی
sudo dd if=/backup/disk.img of=/dev/sda bs=4M

# فشرده
sudo dd if=/dev/sda bs=4M | gzip > disk.img.gz
```

**بکاپ با borg:**

```bash
# نصب
sudo apt install borgbackup

# ساخت مخزن
borg init --encryption=repokey /backup/borg

# بکاپ
borg create /backup/borg::home-{now} /home

# فهرست
borg list /backup/borg

# بازیابی
borg extract /backup/borg::home-2026-10-16

# پاک‌سازی
borg prune --keep-daily=7 --keep-weekly=4 /backup/borg
```

### مقایسه و الگوهای مختلف

**ابزارهای بکاپ:**

| ابزار | نوع | ویژگی |
|---|---|---|
| `tar` | آرشیو | ساده، همه‌جا |
| `rsync` | همگام‌سازی | کارا، incremental |
| `dd` | تصویر | سطح پایین |
| `borg` | dedup | فشرده، نسخه‌بندی |
| `restic` | dedup | رمزنگاری |
| `dump` | سنتی | filesystem-aware |

**tar در برابر rsync:**

| ویژگی | tar | rsync |
|---|---|---|
| سرعت incremental | کند | سریع |
| فشرده‌سازی | بله | با -z |
| بکاپ شبکه | با ssh | بومی |
| بازیابی جزئی | سخت | آسان |

**borg در برابر restic:**

| ویژگی | borg | restic |
|---|---|---|
| زبان | Python | Go |
| dedup | بله | بله |
| رمزنگاری | بله | بله |
| سرعت | خوب | خوب |

### اشکال شایع

**اشکال ۱: نبود بکاپ آفلاین.**
بکاپ روی همان سرور کافی نیست. راه‌حل: 3-2-1.

**اشکال ۲: بکاپ تست‌نشده.**
بکاپی که بازیابی نشود بی‌ارزش است. راه‌حل: تست بازیابی.

**اشکال ۳: اشتباه `--delete` در rsync.**
می‌تواند داده را پاک کند. راه‌حل: `--dry-run` اول.

**اشکال ۴: نبود رمزنگاری.**
بکاپ حساس باید رمزنگاری شود. راه‌حل: borg/restic.

**اشکال ۵: بکاپ در حال تغییر.**
داده ناسازگار. راه‌حل: snapshot.

**اشکال ۶: نبود خودکارسازی.**
بکاپ فراموش می‌شود. راه‌حل: cron.

### نکات مهم (Tips & Tricks)

- **قانون 3-2-1 را رعایت کنید.**
- **بکاپ‌ها را دوره‌ای تست بازیابی کنید.**
- **`rsync -avz --dry-run` قبل از اجرای واقعی.**
- **برای بکاپ سازگار از snapshot (LVM) استفاده کنید.**
- **`borg` و `restic` dedup + رمزنگاری دارند.**
- **بکاپ را خودکار کنید (cron).**
- **`--link-dest` فضای زیادی ذخیره می‌کند.**
- **`tar -czpf` مجوزها را حفظ می‌کند.**
- **`dd` برای تصویر دیسک.**
- **`rsync --bwlimit` پهنای باند را محدود می‌کند.**
- **`borg prune` بکاپ‌های قدیمی را پاک می‌کند.**
- **`restic check` صحت مخزن را بررسی می‌کند.**

### خلاصه فصل

بکاپ سه نوع دارد: full، incremental، differential. قانون 3-2-1 را رعایت کنید. `tar` آرشیو، `rsync` همگام‌سازی، `dd` تصویر و `borg`/`restic` بکاپ dedup می‌سازند. بکاپ را تست بازیابی کنید و خودکار (cron) کنید. `rsync --dry-run` قبل از اجرای واقعی. snapshot برای بکاپ سازگار.

---

## فصل ۷۶: اطلاع‌رسانی به کاربران دربارهٔ رویدادهای سیستم

### مقدمه

گاهی باید به کاربران سیستم اطلاع دهید — قبل از ری‌استارت، در زمان قطعی یا برای اعلان‌های مهم. تصور کنید می‌خواهید سرور را برای تعمیرات ری‌استارت کنید — کاربران باید بدانند تا کار خود را ذخیره کنند.

این فصل ابزارهای اطلاع‌رسانی را پوشش می‌دهد.

### مفاهیم اساسی

**ابزارهای اطلاع‌رسانی:**

| ابزار | کار |
|---|---|
| `wall` | پیام به همهٔ کاربران وارد‌شده |
| `write` | پیام به یک کاربر |
| `mesg` | کنترل دریافت پیام |
| `motd` | پیام ورود |
| `/etc/issue` | پیام قبل از ورود |

**محل پیام‌ها:**

| فایل | زمان |
|---|---|
| `/etc/issue` | قبل از ورود (console) |
| `/etc/issue.net` | قبل از ورود (telnet/ssh) |
| `/etc/motd` | بعد از ورود |
| `/etc/update-motd.d/` | MOTD پویا |

### مثال‌های عملی

**پیام به همه:**

```bash
# wall
wall "سیستم تا ۵ دقیقهٔ دیگر ری‌استارت می‌شود"

# با فایل
wall < notice.txt

# echo pipe
echo "هشدار!" | wall
```

**پیام به یک کاربر:**

```bash
# write
write user1
# سپس متن را بنویسید و Ctrl+D بزنید

# از فایل
write user1 < message.txt
```

**کنترل پیام:**

```bash
# وضعیت
mesg

# غیرفعال کردن دریافت پیام
mesg n

# فعال
mesg y
```

**پیام ورود (MOTD):**

```bash
# /etc/motd (استاتیک)
echo "خوش آمدید به سرور" | sudo tee /etc/motd

# /etc/update-motd.d/ (پویا - Debian/Ubuntu)
ls /etc/update-motd.d/
sudo vim /etc/update-motd.d/99-custom

# مثال اسکریپت
#!/bin/sh
echo "سیستم: $(uptime)"
echo "به‌روزرسانی‌های امنیتی: $(apt list --upgradable 2>/dev/null | wc -l)"
```

**اطلاع‌رسانی در shutdown:**

```bash
sudo shutdown -r +10 "سیستم تا ۱۰ دقیقه ری‌استارت می‌شود"
# shutdown خودکار به کاربران پیام می‌دهد
```

### مقایسه و الگوهای مختلف

**ابزارهای پیام:**

| ابزار | دامنه | نیاز |
|---|---|---|
| `wall` | همهٔ کاربران | ریشه |
| `write` | یک کاربر | — |
| `talk` | تعاملی (قدیمی) | — |
| `motd` | در ورود | — |

**پیام در برابر ایمیل:**

| ویژگی | wall | ایمیل |
|---|---|---|
| فوری | بله | خیر |
| نیاز به ورود | بله | خیر |
| لاگ | خیر | بله |

**سطوح اطلاع‌رسانی:**

| سطح | ابزار |
|---|---|
| فوری | wall |
| ورود | motd |
| اعلان | ایمیل |

### اشکال شایع

**اشکال ۱: wall در اسکریپت cron.**
ممکن است کاربری وارد نباشد. راه‌حل: ترکیب با ایمیل.

**اشکال ۲: mesg n روی root.**
ریشه باید پیام بگیرد. راه‌حل: `mesg y`.

**اشکال ۳: MOTD طولانی.**
ورود را کند می‌کند. راه‌حل: مختصر.

**اشکال ۴: فراموش کردن اطلاع‌رسانی قبل از shutdown.**
کاربران داده از دست می‌دهند. راه‌حل: `shutdown +N`.

**اشکال ۵: نبود `/etc/issue` سفارشی.**
اطلاعات ورود ندارد. راه‌حل: سفارشی کنید.

**اشکال ۶: MOTD پویا کند.**
اسکریپت سنگین. راه‌حل: بهینه.

### نکات مهم (Tips & Tricks)

- **`shutdown` خودکار به کاربران پیام می‌دهد.**
- **`wall` را قبل از کارهای مخرب استفاده کنید.**
- **`/etc/motd` را برای اطلاعیه‌های مهم نگه دارید.**
- **`update-motd.d` پیام پویا می‌سازد.**
- **`mesg n` جلوی مزاحمت را می‌گیرد.**
- **در systemd، `wall` همچنان کار می‌کند.**
- **`wall -n` بدون هدر.**
- **`/etc/issue` کاراکترهای ویژه دارد (`\n`، `\l`).**
- **`figlet`/`banner` برای متن بزرگ.**
- **`fortune` برای MOTD جالب.**
- **برای اعلان گرافیکی، `notify-send`.**
- **`wall` فقط به کاربران وارد‌شده پیام می‌دهد.**

### خلاصه فصل

اطلاع‌رسانی به کاربران با `wall` (همه)، `write` (یک نفر) و `motd`/`issue` (در ورود) انجام می‌شود. `mesg` دریافت پیام را کنترل می‌کند. `shutdown` خودکار پیام می‌دهد. قبل از کارهای مخرب، `wall` بزنید. `update-motd.d` پیام پویا می‌سازد. `notify-send` برای اعلان گرافیکی است.

---

## منابع و مراجع

- man pages: `man make`, `man tar`, `man rsync`, `man dd`, `man wall`, `man borg`
- GNU Make Manual: https://www.gnu.org/software/make/manual/
- rsync Documentation: https://rsync.samba.org/documentation.html
- BorgBackup: https://borgbackup.readthedocs.io
- restic: https://restic.net

**بعدی:** [فایل ۲۲ — سرور DNS](22-DNS-Server.md)
