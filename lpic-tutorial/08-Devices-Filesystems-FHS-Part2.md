# فایل ۰۸ — دستگاه‌ها، فایل‌سیستم و FHS — بخش دوم

**مدت یادگیری**: ۶ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۷ (فایل‌سیستم و mount)

---

## فهرست محتوا

- [فصل ۳۱: مجوزها و مالکیت فایل](#فصل-۳۱-مجوزها-و-مالکیت-فایل)
- [فصل ۳۲: لینک سخت و نمادین](#فصل-۳۲-لینک-سخت-و-نمادین)
- [فصل ۳۳: یافتن فایل‌های سیستم و استاندارد FHS](#فصل-۳۳-یافتن-فایلهای-سیستم-و-استاندارد-fhs)
- [فصل ۳۴: آزمایشگاه: کار با دیسک، پارتیشن و ساختار FHS](#فصل-۳۴-آزمایشگاه-کار-با-دیسک-پارتیشن-و-ساختار-fhs)

---

## فصل ۳۱: مجوزها و مالکیت فایل

### مقدمه

امنیت لینوکس بر پایهٔ **مجوزها (permissions)** بنا شده است. تصور کنید یک فایل کلید SSH دارید — اگر مجوزش اشتباه باشد، SSH کار نمی‌کند یا امنیت به خطر می‌افتد. هر فایل مشخص می‌کند چه کسی می‌تواند بخواند، بنویسد یا اجرا کند.

تسلط بر این مفهوم برای هر مدیر سیستمی حیاتی است و در آزمون LPIC وزن بالایی دارد.

### مفاهیم اساسی

**سه نوع دسترسی:**

| نماد | معنا | برای فایل | برای پوشه |
|---|---|---|---|
| `r` | خواندن | دیدن محتوا | فهرست کردن |
| `w` | نوشتن | تغییر محتوا | ساخت/حذف فایل |
| `x` | اجرا | اجرای برنامه | ورود به پوشه |

**سه دستهٔ کاربر:**

- **u (user/owner):** مالک فایل.
- **g (group):** گروه مالک.
- **o (others):** بقیه.

**نمایش مجوز:**

```
-rwxr-xr--  1 user group 1234 Oct 16 10:00 file.sh
│└┬┘└┬┘└┬┘
│ │  │  └── others: r--  (4)
│ │  └───── group:  r-x  (5)
│ └──────── owner:  rwx  (7)
└────────── نوع: - (فایل)، d (پوشه)، l (لینک)، c (character)، b (block)
```

**مقدار عددی:** `r=4`, `w=2`, `x=1` → مجموع برای هر دسته.

| Octal | نمادین | معنا |
|---|---|---|
| 7 | rwx | خواندن+نوشتن+اجرا |
| 6 | rw- | خواندن+نوشتن |
| 5 | r-x | خواندن+اجرا |
| 4 | r-- | فقط خواندن |
| 0 | --- | هیچ |

### مثال‌های عملی

**دیدن مجوز:**

```bash
ls -l file.txt
ls -ld mydir
ls -la
stat file.txt        # نمایش octal و نمادین
stat -c "%a %n" file.txt   # فقط octal
namei -l /var/www/html     # مجوز کل مسیر
```

**تغییر مجوز با chmod:**

```bash
# نمادین
chmod u+x script.sh
chmod go-w file.txt
chmod a+r file.txt
chmod u=rwx,g=rx,o= file.txt
chmod +x script.sh     # همه
chmod -R u+r dir/      # بازگشتی

# عددی
chmod 644 file.txt     # rw-r--r--
chmod 755 script.sh    # rwxr-xr-x
chmod 700 private/     # rwx------
chmod 600 secret.key   # rw-------
chmod -R 755 /var/www  # بازگشتی

# با مرجع
chmod --reference=file1 file2
```

**تغییر مالکیت با chown/chgrp:**

```bash
sudo chown user file.txt
sudo chown user:group file.txt
sudo chown :group file.txt          # فقط گروه
sudo chown -R www-data:www-data /var/www
sudo chgrp developers project/
sudo chown -h user link             # خود لینک (نه هدف)
```

**مجوزهای ویژه:**

```bash
# SUID (اجرا با دسترسی مالک)
chmod u+s file
chmod 4755 file

# SGID (اجرا با دسترسی گروه)
chmod g+s dir
chmod 2755 dir

# Sticky (فقط مالک حذف می‌کند)
chmod +t shared/
chmod 1777 shared/

# دیدن
ls -l    # s در جای x، t در جای دیگر
```

**umask:**

```bash
# دیدن
umask          # مثلاً 0022

# تنظیم
umask 027      # فایل: 640، پوشه: 750

# محاسبه: 666 - umask = مجوز فایل
#         777 - umask = مجوز پوشه
```

**ACL (دسترسی پیشرفته):**

```bash
# نصب
sudo apt install acl

# تنظیم
setfacl -m u:alice:rw file.txt
setfacl -m g:devs:rx dir/

# دیدن
getfacl file.txt

# حذف
setfacl -x u:alice file.txt
setfacl -b file.txt
```

### مقایسه و الگوهای مختلف

**مجوزهای رایج:**

| مجوز | Octal | کاربرد |
|---|---|---|
| `rwx------` | 700 | فایل/پوشهٔ خصوصی |
| `rwxr-xr-x` | 755 | اسکریپت/پوشهٔ عمومی |
| `rw-r--r--` | 644 | فایل معمولی |
| `rw-------` | 600 | کلید/رمز |
| `rw-rw-r--` | 664 | فایل گروهی |
| `rwxrwxr-x` | 775 | پوشهٔ گروهی |
| `rwxrwxrwx` | 777 | همه‌چیز (خطرناک!) |

**مجوزهای ویژه:**

| مجوز | Octal | نمایش | اثر |
|---|---|---|---|
| **SUID** | 4000 | `s` در جای owner x | اجرا با دسترسی مالک |
| **SGID** | 2000 | `s` در جای group x | اجرا با دسترسی گروه |
| **Sticky** | 1000 | `t` در جای others x | فقط مالک حذف می‌کند |

**نمایش s/t:**

| حالت | نمایش | معنا |
|---|---|---|
| SUID + x | `rws` | SUID با اجرا |
| SUID بدون x | `rwS` | SUID بدون اجرا |
| Sticky + x | `rwt` | Sticky با اجرا |
| Sticky بدون x | `rwT` | Sticky بدون اجرا |

**مثال‌های واقعی SUID:**

| فایل | چرا SUID؟ |
|---|---|
| `/usr/bin/passwd` | کاربر عادی بتواند رمز خود را تغییر دهد |
| `/usr/bin/sudo` | کاربر مجاز بتواند دستور ریشه اجرا کند |
| `/bin/ping` | کاربر عادی بتواند ICMP بفرستد |

### اشکال شایع

**اشکال ۱: `chmod 777`.**
امنیت را نابود می‌کند. راه‌حل: حداقل مجوز لازم را بدهید.

**اشکال ۲: اشتباه محاسبهٔ octal.**
`rw-r--r--` = ۶۴۴ (نه ۶۴۲). راه‌حل: ۴+۲+۰=۶، ۴+۰+۰=۴، ۴+۰+۰=۴.

**اشکال ۳: `chmod -R` بی‌احتیاط.**
روی پوشه‌های سیستمی خطرناک است. راه‌حل: فقط پوشهٔ هدف.

**اشکال ۴: فراموش کردن `x` روی پوشه.**
بدون `x` نمی‌توان وارد پوشه شد. راه‌حل: `x` بدهید.

**اشکال ۵: `chown` بدون sudo.**
مالکیت تغییر نمی‌کند. راه‌حل: `sudo`.

**اشکال ۶: فراموش کردن مجوز کلید SSH.**
SSH کلید با مجوز باز را رد می‌کند. راه‌حل: `chmod 600`.

### نکات مهم (Tips & Tricks)

- **برای پوشه‌ها، `x` یعنی «ورود»؛ بدون آن `ls` کار نمی‌کند.**
- **`umask` مجوز پیش‌فرض فایل‌های جدید را تعیین می‌کند.**
- **SUID را با `find / -perm -4000` پیدا کنید.**
- **`chmod --reference=file1 file2` مجوز را کپی می‌کند.**
- **`setfacl`/`getfacl` برای ACLهای پیشرفته.**
- **`ls -l` ستون دوم تعداد لینک سخت است.**
- **`stat -c "%a"` فقط octal می‌دهد.**
- **`install -m 755 src dst` کپی با مجوز.**
- **`chmod -R` را با `find -type` ترکیب کنید** برای دقت.
- **`umask 077` برای فایل‌های خصوصی.**
- **`chattr +i file` فایل را غیرقابل تغییر می‌کند.**
- **`lsattr` ویژگی‌های ویژه را نشان می‌دهد.**

### خلاصه فصل

مجوزها (r/w/x) برای سه دسته (u/g/o) تعیین می‌شوند. `chmod` (نمادین یا octal)، `chown` و `chgrp` مجوز و مالکیت را تغییر می‌دهند. مجوزهای ویژه SUID/SGID/Sticky کارهای خاص می‌کنند. از `chmod 777` پرهیز کنید و حداقل مجوز لازم را بدهید. `umask` پیش‌فرض‌ها را تعیین می‌کند و ACL کنترل پیشرفته می‌دهد. مجوز ۶۰۰ برای کلیدها ضروری است.

---

## فصل ۳۲: لینک سخت و نمادین

### مقدمه

تصور کنید می‌خواهید یک برنامه را از دو مسیر مختلف در دسترس داشته باشید، یا یک فایل بزرگ را بدون کپی کردن در چند جا ارجاع دهید. راه‌حل: **لینک (link)**.

در لینوکس دو نوع لینک وجود دارد: سخت (hard) و نمادین (symbolic). درک تفاوت آن‌ها برای مدیریت فایل و عیب‌یابی مهم است و در آزمون می‌آید.

### مفاهیم اساسی

**Hard Link:**

- یک نام دیگر برای همان **inode** (همان داده).
- حذف یکی، داده باقی می‌ماند (تا آخرین لینک).
- نمی‌تواند بین فایل‌سیستم‌های مختلف باشد.
- نمی‌تواند به پوشه اشاره کند.

**Symbolic Link (soft link):**

- یک فایل جدا که به **مسیر** فایل دیگر اشاره می‌کند.
- مثل «میان‌بر».
- می‌تواند بین فایل‌سیستم‌ها باشد.
- اگر هدف حذف شود، لینک «شکسته» می‌شود.

**inode چیست؟** ساختاری که متادیتای فایل (مجوز، مالک، زمان، مکان داده) را نگه می‌دارد. نام فایل جدا از inode است.

### مثال‌های عملی

**ساخت لینک سخت:**

```bash
ln original.txt hardlink.txt

# بررسی inode
ls -li original.txt hardlink.txt
# هر دو یک inode دارند (شمارهٔ یکسان)

# تعداد لینک در ستون دوم ls -l
ls -l original.txt
# -rw-r--r-- 2 user group ... (عدد ۲ = دو لینک)
```

**ساخت لینک نمادین:**

```bash
ln -s /path/to/target symlink.txt

# نمونه
ln -s /var/log/nginx/access.log ~/access.log

# لینک به پوشه
ln -s /data/projects ~/projects

# لینک نسبی
ln -s ../shared/file.txt ./link.txt

# بازنویسی لینک موجود
ln -sf /new/target link.txt
```

**بررسی:**

```bash
ls -l              # لینک نمادین با -> نشان داده می‌شود
ls -li             # نمایش inode
readlink symlink.txt
readlink -f symlink.txt   # مسیر کامل (resolve)
stat symlink.txt
file symlink.txt
```

**حذف لینک:**

```bash
rm symlink.txt     # فقط لینک حذف می‌شود (نه هدف)
unlink symlink.txt
rm hardlink.txt    # داده باقی می‌ماند (تا آخرین لینک)
```

**پیدا کردن لینک‌های شکسته:**

```bash
find / -xtype l
find . -type l ! -exec test -e {} \; -print
```

### مقایسه و الگوهای مختلف

**تفاوت‌ها:**

| ویژگی | Hard Link | Symbolic Link |
|---|---|---|
| inode | یکسان | متفاوت |
| بین فایل‌سیستم | خیر | بله |
| به پوشه | خیر | بله |
| حذف هدف | لینک کار می‌کند | لینک شکسته می‌شود |
| اندازه | = فایل | = طول مسیر |
| مسیر مطلق/نسبی | — | هر دو |
| مجوز | = فایل اصلی | `lrwxrwxrwx` |

**نمایش در `ls -l`:**

```
-rw-r--r-- 2 user group 100 ... file.txt       ← لینک سخت (count=2)
lrwxrwxrwx 1 user group  10 ... link -> file    ← لینک نمادین
```

**کاربردها:**

| کاربرد | نوع لینک |
|---|---|
| میانبر به فایل/پوشه | نمادین |
| نسخه‌بندی (`python` → `python3`) | نمادین |
| صرفه‌جویی فضا | سخت |
| بکاپ (rsync --link-dest) | سخت |
| پیکربندی سایت (nginx sites-enabled) | نمادین |

### اشکال شایع

**اشکال ۱: لینک سخت به پوشه.**
مجاز نیست. راه‌حل: از لینک نمادین استفاده کنید.

**اشکال ۲: لینک سخت بین فایل‌سیستم‌ها.**
مجاز نیست. راه‌حل: لینک نمادین.

**اشکال ۳: لینک نمادین نسبی اشتباه.**
مسیر نسبی نسبت به محل لینک است، نه محل فعلی. راه‌حل: از مسیر مطلق استفاده کنید.

**اشکال ۴: شکستن لینک نمادین.**
اگر هدف جابجا/حذف شود، لینک شکسته می‌شود. راه‌حل: هدف را حفظ کنید.

**اشکال ۵: `rm -r` روی پوشهٔ لینک‌شده.**
با `/` انتهایی، محتوای هدف پاک می‌شود! راه‌حل: `rm link` نه `rm link/`.

**اشکال ۶: انتظار حذف هدف با حذف لینک.**
حذف لینک، هدف را حذف نمی‌کند. راه‌حل: آگاه باشید.

### نکات مهم (Tips & Tricks)

- **`ln -s` همیشه با مسیر مطلق امن‌تر است.**
- **برای پیدا کردن لینک‌های شکسته: `find / -xtype l`.**
- **تعداد لینک سخت در ستون دوم `ls -l` دیده می‌شود.**
- **لینک نمادین در `rsync` با `-l` یا `-a` حفظ می‌شود.**
- **برای ساختار پروژه، لینک نمادین به پوشه بسیار مفید است.**
- **`readlink -f` مسیر نهایی را resolve می‌کند.**
- **`ln -sf` لینک موجود را بازنویسی می‌کند.**
- **`cp -d` لینک را حفظ می‌کند.**
- **`cp -L` لینک را دنبال می‌کند (کپی محتوا).**
- **در بکاپ، `--link-dest` فضای زیادی ذخیره می‌کند.**
- **`ln -sr` لینک نسبی هوشمند می‌سازد.**
- **`tar -h` لینک را دنبال می‌کند (پیش‌فرض حفظ).**

### خلاصه فصل

Hard link نام دومی برای همان inode است (فقط در یک فایل‌سیستم، بدون پوشه). Symbolic link یک فایل جدا با اشاره به مسیر است (بین فایل‌سیستم‌ها، به پوشه). با `ln` و `ln -s` ساخته می‌شوند. لینک نمادین با حذف هدف می‌شکند؛ لینک سخت تا آخرین نام باقی می‌ماند. در `rsync --link-dest` و پیکربندی سایت (sites-enabled) کاربرد دارند.

---

## فصل ۳۳: یافتن فایل‌های سیستم و استاندارد FHS

### مقدمه

تصور کنید باید یک فایل پیکربندی را پیدا کنید اما نمی‌دانید کجاست. اگر **FHS (Filesystem Hierarchy Standard)** را بلد باشید، می‌دانید که پیکربندی در `/etc` است. اگر فایلی گم شده، با `find` یا `locate` پیدا می‌کنید.

FHS استانداردی است که مشخص می‌کند هر نوع فایل در کدام پوشه قرار گیرد. این دانش به شما کمک می‌کند فایل‌ها را سریع پیدا کنید و بدانید هر چیز کجاست.

### مفاهیم اساسی

**ساختار FHS:**

| مسیر | محتوا |
|---|---|
| `/` | ریشه |
| `/bin` | دستورهای پایه (در توزیع‌های جدید → `/usr/bin`) |
| `/sbin` | دستورهای مدیریتی (→ `/usr/sbin`) |
| `/etc` | فایل‌های پیکربندی |
| `/home` | پوشه‌های کاربران |
| `/root` | پوشهٔ خانهٔ ریشه |
| `/var` | دادهٔ متغیر (لاگ، صف، کش) |
| `/tmp` | فایل‌های موقت |
| `/usr` | برنامه‌ها و کتابخانه‌ها |
| `/lib` | کتابخانه‌های اشتراکی |
| `/opt` | نرم‌افزارهای اختیاری |
| `/mnt` , `/media` | نقاط mount |
| `/dev` | فایل‌های دستگاه |
| `/proc` , `/sys` | فایل‌سیستم‌های مجازی |
| `/boot` | فایل‌های بوت |
| `/srv` | دادهٔ سرویس‌ها |
| `/run` | دادهٔ زمان اجرا |

**زیرپوشه‌های `/usr`:**

| مسیر | محتوا |
|---|---|
| `/usr/bin` | دستورهای کاربر |
| `/usr/sbin` | دستورهای مدیریتی |
| `/usr/lib` | کتابخانه‌ها |
| `/usr/share` | دادهٔ مشترک (man, doc) |
| `/usr/local` | نرم‌افزار نصب‌شدهٔ دستی |
| `/usr/include` | هدرهای C |

**زیرپوشه‌های `/var`:**

| مسیر | محتوا |
|---|---|
| `/var/log` | لاگ‌ها |
| `/var/spool` | صف (چاپ، ایمیل) |
| `/var/cache` | کش |
| `/var/lib` | دادهٔ وضعیت |
| `/var/tmp` | فایل‌های موقت ماندگار |

### مثال‌های عملی

**یافتن فایل با `find`:**

```bash
# بر اساس نام
find / -name "*.conf"
find /etc -name "*.conf"

# بی‌توجه به حروف
find / -iname "README*"

# بر اساس نوع
find /var -type d          # پوشه‌ها
find /var -type f          # فایل‌ها
find / -type l             # لینک‌ها

# بر اساس اندازه
find / -size +100M         # بزرگ‌تر از ۱۰۰ مگ
find / -size -1k           # کوچک‌تر از ۱ کیلوبایت
find / -size +1G -size -2G # بین ۱ و ۲ گیگ

# بر اساس زمان
find / -mtime -7           # تغییر در ۷ روز اخیر
find / -atime +30          # دسترسی قدیمی‌تر از ۳۰ روز
find / -mmin -60           # تغییر در ۶۰ دقیقه اخیر

# بر اساس مالک
find / -user alice
find / -group devs
find / -nouser             # بدون مالک

# بر اساس مجوز
find / -perm -4000         # فایل‌های SUID
find / -perm 644           # دقیقاً ۶۴۴

# با اجرای دستور
find /tmp -name "*.tmp" -exec rm {} \;
find /tmp -name "*.tmp" -delete
find . -type f -exec chmod 644 {} +

# محدود کردن عمق
find / -maxdepth 2 -name "*.conf"

# فقط یک فایل‌سیستم
find / -xdev -name "*.log"
```

**یافتن فایل با `locate`:**

```bash
# باید ابتدا database را به‌روز کنید
sudo updatedb
locate nginx.conf
locate -i readme       # بی‌توجه به حروف
locate -c "*.conf"     # شمارش
```

**جستجوی دستور:**

```bash
which ls           # مسیر دستور
whereis ls         # مسیر + man
type ls            # نوع (alias/function/builtin)
command -v ls      # مسیر یا نوع
```

**جستجو در محتوا:**

```bash
grep -r "keyword" /etc
grep -rl "keyword" /etc     # فقط نام فایل
```

### مقایسه و الگوهای مختلف

**`find` در برابر `locate`:**

| ویژگی | find | locate |
|---|---|---|
| سرعت | کند (جستجوی زنده) | سریع (database) |
| به‌روز بودن | همیشه | نیاز به updatedb |
| انعطاف | بسیار زیاد | محدود |
| جستجو بر اساس ویژگی | بله | خیر |

**مسیرهای مهم پیکربندی:**

| سرویس | مسیر |
|---|---|
| کاربران | `/etc/passwd`, `/etc/shadow` |
| گروه‌ها | `/etc/group` |
| شبکه | `/etc/network/` یا `/etc/sysconfig/network-scripts/` |
| سرویس‌ها | `/etc/systemd/system/` |
| لاگ‌ها | `/var/log/` |
| کرنل | `/boot/` |
| DNS | `/etc/resolv.conf` |
| fstab | `/etc/fstab` |

**مسیرهای لاگ مهم:**

| لاگ | مسیر (Debian) | مسیر (RHEL) |
|---|---|---|
| عمومی | `/var/log/syslog` | `/var/log/messages` |
| احراز هویت | `/var/log/auth.log` | `/var/log/secure` |
| کرنل | `/var/log/kern.log` | `/var/log/dmesg` |
| نصب پکیج | `/var/log/dpkg.log` | `/var/log/dnf.log` |

### اشکال شایع

**اشکال ۱: `find /` بدون محدودیت.**
بسیار کند است؛ مسیر را محدود کنید. راه‌حل: مسیر یا `-maxdepth`.

**اشکال ۲: `locate` قدیمی.**
database به‌روز نیست؛ `updatedb` بزنید. راه‌حل: `sudo updatedb`.

**اشکال ۳: `-exec` بدون `\;`.**
خطای syntax می‌دهد. راه‌حل: `\;` یا `+`.

**اشکال ۴: جستجو در `/proc` و `/sys`.**
این‌ها مجازی‌اند؛ `-xdev` یا حذف کنید. راه‌حل: `-xdev`.

**اشکال ۵: `find -delete` بدون `-type`.**
ممکن است پوشه‌ها را هم حذف کند. راه‌حل: `-type f` اضافه کنید.

**اشکال ۶: اشتباه در `-size`.**
`+100M` یعنی بزرگ‌تر از ۱۰۰ مگ. راه‌حل: علامت‌ها را بشناسید.

### نکات مهم (Tips & Tricks)

- **`find ... -delete` به‌جای `-exec rm`.**
- **`find -maxdepth 2` عمق جستجو را محدود می‌کند.**
- **`locate -i` بی‌توجه به حروف.**
- **`find . -name "*.log" -mtime +30 -delete` پاک‌سازی لاگ‌های قدیمی.**
- **`type` نشان می‌دهد یک نام دستور است، alias یا builtin.**
- **`find -exec {} +` سریع‌تر از `{} \;` است.**
- **`find -newer file` فایل‌های جدیدتر از یک فایل.**
- **`find -empty` فایل‌ها/پوشه‌های خالی.**
- **`find -links +1` فایل‌های دارای لینک سخت.**
- **`updatedb` را در cron بگذارید** تا locate به‌روز بماند.
- **`mlocate`/`plocate` سریع‌تر از locate قدیمی.**
- **`fzf` ابزار جستجوی تعاملی عالی است.**

### خلاصه فصل

FHS استاندارد محل فایل‌هاست: `/etc` پیکربندی، `/var` دادهٔ متغیر، `/home` کاربران، `/usr` برنامه‌ها، `/boot` بوت. با `find` (زنده، انعطاف‌پذیر) و `locate` (سریع، database) فایل پیدا کنید. `which`, `whereis`, `type` مسیر دستورها را نشان می‌دهند. مسیرها را محدود کنید تا جستجو سریع باشد. مسیرهای لاگ بین Debian و RHEL متفاوت‌اند.

---

## فصل ۳۴: آزمایشگاه: کار با دیسک، پارتیشن و ساختار FHS

### مقدمه

در این آزمایشگاه، تمام مفاهیم فایل ۰۷ و ۰۸ را در عمل ترکیب می‌کنیم: پارتیشن‌بندی، فرمت، mount، مجوز و لینک.

هدف: شبیه‌سازی یک سناریوی واقعی — افزودن یک دیسک جدید به سرور و پیکربندی کامل آن.

### مفاهیم اساسی

هدف آزمایشگاه:
1. یک دیسک مجازی جدید اضافه کنید.
2. پارتیشن بسازید و فرمت کنید.
3. mount کنید و در fstab بگذارید.
4. مجوزها و لینک‌ها را تمرین کنید.
5. ساختار FHS را کاوش کنید.

### مثال‌های عملی

**گام ۱: افزودن دیسک**

در VirtualBox: Settings → Storage → Add Hard Disk → Create new (مثلاً ۵ GB).

```bash
# بررسی دیسک جدید
lsblk
# مثلاً /dev/sdb ظاهر می‌شود

# پاک‌سازی (اگر قبلاً استفاده شده)
sudo wipefs -a /dev/sdb
```

**گام ۲: پارتیشن‌بندی و فرمت**

```bash
# ساخت پارتیشن
sudo fdisk /dev/sdb
#   n → p → 1 → Enter → Enter → w

# یا با parted
sudo parted /dev/sdb mklabel gpt
sudo parted /dev/sdb mkpart primary ext4 1MiB 100%

# فرمت
sudo mkfs.ext4 -L labdata /dev/sdb1

# بررسی
lsblk -f
sudo blkid /dev/sdb1
```

**گام ۳: Mount**

```bash
# ساخت نقطهٔ mount
sudo mkdir -p /mnt/labdata

# mount
sudo mount /dev/sdb1 /mnt/labdata

# بررسی
df -h /mnt/labdata
findmnt /mnt/labdata
mount | grep labdata
```

**گام ۴: پیکربندی دائمی**

```bash
# گرفتن UUID
sudo blkid /dev/sdb1

# افزودن به fstab
echo "UUID=$(sudo blkid -s UUID -o value /dev/sdb1) /mnt/labdata ext4 defaults,noatime 0 2" | sudo tee -a /etc/fstab

# تست بدون reboot
sudo umount /mnt/labdata
sudo mount -a
findmnt /mnt/labdata

# بررسی خطاهای fstab
sudo findmnt --verify
```

**گام ۵: مجوز و لینک**

```bash
# ساخت ساختار
sudo mkdir -p /mnt/labdata/{shared,private,data}
sudo chown $USER:$USER /mnt/labdata/shared
sudo chown root:root /mnt/labdata/private
chmod 750 /mnt/labdata/private
chmod 770 /mnt/labdata/shared

# فایل تست
echo "test" > /mnt/labdata/shared/test.txt

# لینک نمادین
ln -s /mnt/labdata/shared ~/shared-data
ls -l ~/shared-data

# لینک سخت
ln /mnt/labdata/shared/test.txt /mnt/labdata/data/test-hard.txt
ls -li /mnt/labdata/shared/test.txt /mnt/labdata/data/test-hard.txt
```

**گام ۶: کاوش FHS**

```bash
ls -la /
cat /etc/os-release
ls /etc | head -20
ls /var/log | head -20
ls /usr/bin | wc -l
cat /proc/cpuinfo | head -5
mount | column -t
```

**گام ۷: تمرین find**

```bash
# پیدا کردن فایل‌های SUID
sudo find / -perm -4000 -type f 2>/dev/null

# فایل‌های بزرگ
sudo find / -type f -size +50M 2>/dev/null

# لاگ‌های قدیمی
sudo find /var/log -type f -mtime +30 2>/dev/null

# فایل‌های بدون مالک
sudo find / -nouser 2>/dev/null
```

### مقایسه و الگوهای مختلف

**پاک‌سازی آزمایشگاه:**

| کار | دستور |
|---|---|
| unmount | `sudo umount /mnt/labdata` |
| حذف از fstab | ویرایش `/etc/fstab` |
| پاک کردن لینک | `rm ~/shared-data` |
| پاک کردن پارتیشن | `sudo wipefs -a /dev/sdb` |

**بررسی نهایی:**

```bash
# چک‌لیست
df -hT                    # فایل‌سیستم‌ها
lsblk -f                  # دیسک‌ها
findmnt --verify          # صحت fstab
cat /etc/fstab            # محتوای fstab
```

### اشکال شایع

**اشکال ۱: اشتباه در fstab.**
اگر بوت به rescue رفت، فایل را اصلاح کنید. راه‌حل: `mount -a` قبل از reboot.

**اشکال ۲: فراموش کردن unmount قبل از تغییر.**
ممکن است خطا بدهد. راه‌حل: اول umount.

**اشکال ۳: mount روی پوشهٔ غیرخالی.**
محتوا پنهان می‌شود. راه‌حل: پوشهٔ خالی.

**اشکال ۴: حذف لینک با `/` انتهایی.**
`rm link` درست است، نه `rm link/`. راه‌حل: بدون `/`.

**اشکال ۵: `find` کند.**
`/proc` و `/sys` را شامل می‌شود. راه‌حل: `-xdev` یا محدود کردن مسیر.

**اشکال ۶: مجوز اشتباه لینک.**
مجوز لینک نمادین همیشه `lrwxrwxrwx` است. راه‌حل: مجوز هدف را تغییر دهید.

### نکات مهم (Tips & Tricks)

- **همیشه `mount -a` را قبل از reboot تست کنید.**
- **`blkid` UUID را می‌دهد؛ در fstab از آن استفاده کنید.**
- **برای آزمایشگاه، برچسب (`-L`) را هم تمرین کنید.**
- **`findmnt --verify` صحت fstab را بررسی می‌کند.**
- **اسنپ‌شات قبل از آزمایش بگیرید.**
- **`wipefs` برای پاک‌سازی امضای فایل‌سیستم.**
- **`namei -l` مجوز کل مسیر را نشان می‌دهد.**
- **`stat` زمان‌های فایل را نشان می‌دهد.**
- **`lsblk -o NAME,SIZE,FSTYPE,LABEL,UUID,MOUNTPOINT` همه‌چیز را نشان می‌دهد.**
- **در پایان، آزمایشگاه را پاک‌سازی کنید.**

### خلاصه فصل

در این آزمایشگاه دیسک اضافه، پارتیشن‌بندی، فرمت، mount و پیکربندی دائمی (fstab با UUID) را تمرین کردیم. مجوزها و لینک‌های سخت/نمادین را ساختیم و ساختار FHS را کاوش کردیم. نکات کلیدی: تست `mount -a` قبل از reboot، استفاده از UUID در fstab، و `findmnt --verify` برای بررسی صحت.

---

## منابع و مراجع

- man pages: `man chmod`, `man chown`, `man ln`, `man find`, `man fstab`, `man locate`
- Filesystem Hierarchy Standard 3.0: https://refspecs.linuxfoundation.org/FHS_3.0/fhs-3.0.html
- POSIX File Permissions: https://pubs.opengroup.org/onlinepubs/9699919799/
- ACL Documentation: https://wiki.archlinux.org/title/Access_Control_List

**بعدی:** [فایل ۰۹ — شل‌ها و اسکریپت‌نویسی](09-Shells-and-Shell-Scripting.md)
