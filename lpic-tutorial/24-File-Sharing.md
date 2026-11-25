# فایل ۲۴ — اشتراک‌گذاری فایل

**مدت یادگیری**: ۶ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: فایل ۱۳ (شبکه)، فایل ۲۳ (HTTP)

---

## فهرست محتوا

- [فصل ۸۴: پیکربندی سرور Samba](#فصل-۸۴-پیکربندی-سرور-samba)
- [فصل ۸۵: پیکربندی سرور NFS](#فصل-۸۵-پیکربندی-سرور-nfs)

---

## فصل ۸۴: پیکربندی سرور Samba

### مقدمه

**Samba** پیاده‌سازی متن‌باز پروتکل **SMB/CIFS** است که اشتراک‌گذاری فایل بین لینوکس و ویندوز را ممکن می‌کند. تصور کنید یک دفتر با کامپیوترهای ویندوزی و یک سرور لینوکسی دارید — Samba پل بین آن‌هاست.

برای محیط‌های مختلط حیاتی است و در آزمون می‌آید.

### مفاهیم اساسی

**مفاهیم کلیدی:**

- **SMB/CIFS:** پروتکل اشتراک ویندوز.
- **smbd:** daemon فایل.
- **nmbd:** daemon نام‌نت‌بایوس.
- **Share:** پوشهٔ اشتراکی.
- **smb.conf:** پیکربندی.
- **Workgroup:** گروه کاری.

**فایل‌های مهم:**

- `/etc/samba/smb.conf` — پیکربندی.
- `/var/log/samba/` — لاگ.
- `/var/lib/samba/` — داده.

**سطوح امنیت:**

| مقدار | توضیح |
|---|---|
| `user` | نیاز به احراز (پیش‌فرض) |
| `share` | بدون احراز (قدیمی، ناامن) |
| `server` | سرور جدا |
| `domain` | دامنه (AD) |

### مثال‌های عملی

**نصب:**

```bash
sudo apt install samba        # Debian
sudo dnf install samba        # RHEL
sudo systemctl enable --now smbd nmbd
```

**پیکربندی `/etc/samba/smb.conf`:**

```
[global]
   workgroup = WORKGROUP
   server string = File Server
   security = user
   map to guest = bad user
   dns proxy = no
   log file = /var/log/samba/log.%m
   max log size = 1000

[shared]
   path = /srv/samba/shared
   browseable = yes
   read only = no
   guest ok = yes
   valid users = @smbusers
   create mask = 0664
   directory mask = 0775

[private]
   path = /srv/samba/private
   browseable = yes
   read only = no
   valid users = alice, bob
   write list = alice
   valid users = alice, bob
```

**مدیریت کاربر:**

```bash
# افزودن کاربر samba
sudo smbpasswd -a alice

# فعال/غیرفعال
sudo smbpasswd -e alice
sudo smbpasswd -d alice

# حذف
sudo smbpasswd -x alice

# فهرست
sudo pdbedit -L
```

**بررسی و تست:**

```bash
# تست پیکربندی
testparm

# وضعیت
sudo smbstatus

# اتصال از کلاینت
smbclient -L //server -U alice
smbclient //server/shared -U alice
mount -t cifs //server/shared /mnt -o username=alice
```

**مجوز فایل‌سیستم:**

```bash
sudo mkdir -p /srv/samba/shared
sudo chown -R nobody:nogroup /srv/samba/shared
sudo chmod 0775 /srv/samba/shared

# برای SELinux (RHEL)
sudo setsebool -P samba_export_all_rw on
sudo chcon -t samba_share_t /srv/samba/shared
```

### مقایسه و الگوهای مختلف

**Samba در برابر NFS:**

| ویژگی | Samba | NFS |
|---|---|---|
| سازگاری ویندوز | عالی | ضعیف |
| سازگاری لینوکس | خوب | عالی |
| پروتکل | SMB/CIFS | NFS |
| احراز هویت | کاربر/رمز | UID/GID |
| پورت | 139/445 | 2049 |

**نسخه‌های SMB:**

| نسخه | ویژگی |
|---|---|
| SMB1 | قدیمی، ناامن |
| SMB2 | مدرن |
| SMB3 | رمزنگاری |

**گزینه‌های مهم smb.conf:**

| گزینه | کار |
|---|---|
| `browseable` | نمایش در فهرست |
| `read only` | فقط‌خواندنی |
| `valid users` | کاربران مجاز |
| `write list` | کاربران نویسنده |
| `guest ok` | دسترسی مهمان |
| `create mask` | مجوز فایل جدید |

### اشکال شایع

**اشکال ۱: مجوز فایل‌سیستم.**
Samba نمی‌تواند بنویسد اگر مجوز سیستمی نباشد. راه‌حل: chown/chmod.

**اشکال ۲: SELinux در RHEL.**
context اشتباه دسترسی را می‌بندد. راه‌حل: `setsebool`.

**اشکال ۳: security=share.**
ناامن است. راه‌حل: `security=user`.

**اشکال ۴: کاربر samba وجود ندارد.**
باید با `smbpasswd -a` ساخته شود. راه‌حل: افزودن کاربر.

**اشکال ۵: فایروال پورت ۱۳۹/۴۴۵.**
اتصال نمی‌شود. راه‌حل: باز کردن.

**اشکال ۶: نام کارگروه اشتباه.**
در ویندوز دیده نمی‌شود. راه‌حل: workgroup یکسان.

### نکات مهم (Tips & Tricks)

- **`testparm` قبل از restart.**
- **`smbstatus` اتصالات فعال را نشان می‌دهد.**
- **برای SELinux: `setsebool -P samba_export_all_rw on`.**
- **`valid users` و `write list` دسترسی را کنترل می‌کنند.**
- **`smbclient -L //server` اشتراک‌ها را فهرست می‌کند.**
- **`log level` را برای دیباگ بالا ببرید.**
- **`pdbedit -L` کاربران samba.**
- **`map to guest = bad user` برای مهمان.**
- **`force user`/`force group` مجوز یکسان.**
- **SMB3 رمزنگاری دارد.**
- **`testparm -v` همهٔ تنظیمات.**
- **`smbpasswd` رمز جدا از سیستم.**

### خلاصه فصل

Samba اشتراک فایل SMB/CIFS برای سازگاری ویندوز است. با `smbd`/`nmbd` و `smb.conf` پیکربندی می‌شود. کاربران با `smbpasswd` ساخته می‌شوند. `testparm` و `smbstatus` ابزارهای بررسی‌اند. مجوز فایل‌سیستم و SELinux را فراموش نکنید. `security=user` امن‌تر از `share` است.

---

## فصل ۸۵: پیکربندی سرور NFS

### مقدمه

**NFS (Network File System)** پروتکل اشتراک فایل بومی یونیکس/لینوکس است. تصور کنید چند سرور لینوکسی دارید و می‌خواهید یک پوشهٔ مشترک داشته باشند — NFS سریع و کارا است.

برای اشتراک بین سیستم‌های لینوکسی عالی است و در آزمون می‌آید.

### مفاهیم اساسی

**مفاهیم کلیدی:**

- **export:** پوشهٔ اشتراکی.
- **nfsd:** daemon سرور.
- **/etc/exports:** پیکربندی export.
- **rpcbind:** نگاشت RPC.
- **UID/GID:** مبنای احراز.

**نسخه‌ها:** NFSv3 (کلاسیک)، NFSv4 (مدرن، امن‌تر).

**مقایسه نسخه‌ها:**

| ویژگی | NFSv3 | NFSv4 |
|---|---|---|
| پورت | چند پورت | فقط 2049 |
| فایروال | پیچیده | ساده |
| امنیت | ضعیف | Kerberos |
| ACL | محدود | کامل |

### مثال‌های عملی

**نصب:**

```bash
sudo apt install nfs-kernel-server    # Debian
sudo dnf install nfs-utils             # RHEL
sudo systemctl enable --now nfs-server
```

**پیکربندی `/etc/exports`:**

```
# پوشه              کلاینت(ها)        گزینه‌ها
/srv/nfs/shared     192.168.1.0/24(rw,sync,no_subtree_check)
/srv/nfs/public     *(ro,sync,no_subtree_check)
/srv/nfs/private    192.168.1.50(rw,sync,no_root_squash)
/srv/nfs/home       192.168.1.0/24(rw,sync,root_squash)
```

**اعمال و بررسی:**

```bash
# اعمال
sudo exportfs -ra

# فهرست exportها
sudo exportfs -v

# وضعیت
sudo systemctl status nfs-server
cat /proc/fs/nfs/exports
```

**کلاینت:**

```bash
# نصب
sudo apt install nfs-common

# mount
sudo mount -t nfs server:/srv/nfs/shared /mnt/nfs

# در fstab
server:/srv/nfs/shared /mnt/nfs nfs defaults,_netdev 0 0

# بررسی
showmount -e server
mount | grep nfs
```

### مقایسه و الگوهای مختلف

**گزینه‌های مهم exports:**

| گزینه | معنا |
|---|---|
| `rw` / `ro` | خواندن-نوشتن / فقط‌خواندن |
| `sync` / `async` | همگام / ناهمگام |
| `no_root_squash` | ریشهٔ کلاینت = ریشهٔ سرور (خطرناک) |
| `root_squash` | ریشهٔ کلاینت = nobody (امن، پیش‌فرض) |
| `all_squash` | همه به nobody |
| `no_subtree_check` | سرعت بیشتر |
| `noexec` | ممنوعیت اجرا |
| `anonuid`/`anongid` | UID/GID مهمان |

**NFS در برابر Samba:**

| ویژگی | NFS | Samba |
|---|---|---|
| سازگاری لینوکس | عالی | خوب |
| سازگاری ویندوز | ضعیف | عالی |
| پروتکل | NFS | SMB |
| پورت | 2049 | 139/445 |

**NFSv4 features:**

| ویژگی | توضیح |
|---|---|
| پورت واحد | 2049 |
| ACL | کامل |
| Kerberos | امنیت |
| Compound | کارایی |

### اشکال شایع

**اشکال ۱: فراموش کردن `exportfs -ra`.**
تغییرات اعمال نمی‌شود. راه‌حل: `exportfs -ra`.

**اشکال ۲: no_root_squash بدون درک.**
خطر امنیتی. راه‌حل: `root_squash`.

**اشکال ۳: فایروال چند پورت NFSv3.**
باید rpcbind و mountd هم باز باشند. راه‌حل: NFSv4 یا باز کردن.

**اشکال ۴: UID/GID ناهماهنگ.**
مجوزها اشتباه می‌شوند. راه‌حل: UID یکسان.

**اشکال ۵: نبود rpcbind.**
NFSv3 کار نمی‌کند. راه‌حل: `rpcbind`.

**اشکال ۶: `_netdev` نبودن در fstab.**
بوت کند می‌شود. راه‌حل: `_netdev`.

### نکات مهم (Tips & Tricks)

- **`exportfs -v` وضعیت exportها را نشان می‌دهد.**
- **`root_squash` (پیش‌فرض) امن است.**
- **NFSv4 فقط پورت ۲۰۴۹ نیاز دارد.**
- **`showmount -e server` exportهای سرور را فهرست می‌کند.**
- **برای کارایی، `async` (با احتیاط).**
- **`_netdev` در fstab برای انتظار شبکه.**
- **`nfsstat -s`/`-c` آمار را نشان می‌دهد.**
- **`rpcinfo -p server` پورت‌های RPC.**
- **`/etc/idmapd.conf` نگاشت UID.**
- **Kerberos برای NFSv4 امن.**
- **`autofs` برای mount بر تقاضا.**
- **`hard`/`soft` mount رفتار در قطعی.**

### خلاصه فصل

NFS اشتراک فایل بومی لینوکس است. `/etc/exports` پوشه‌ها و کلاینت‌ها را تعیین می‌کند. `exportfs -ra` اعمال می‌کند و `showmount -e` فهرست می‌دهد. NFSv4 ساده‌تر و امن‌تر است. `root_squash` امن است و UID/GID باید هماهنگ باشند. `_netdev` در fstab برای انتظار شبکه.

---

## منابع و مراجع

- man pages: `man smb.conf`, `man smbclient`, `man exports`, `man exportfs`, `man nfs`
- Samba Documentation: https://www.samba.org/samba/docs/
- NFS HOWTO: https://nfs.sourceforge.net/nfs-howto/
- RFC 3530 (NFSv4): https://www.rfc-editor.org

**بعدی:** [فایل ۲۵ — مدیریت کلاینت شبکه](25-Network-Client-Management.md)
