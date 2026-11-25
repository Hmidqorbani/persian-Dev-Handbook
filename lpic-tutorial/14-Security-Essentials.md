# فایل ۱۴ — مبانی امنیت

**مدت یادگیری**: ۷ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۱۳ (شبکه)، فایل ۱۱ (کاربران)

---

## فهرست محتوا

- [فصل ۵۲: وظایف مدیریت امنیت](#فصل-۵۲-وظایف-مدیریت-امنیت)
- [فصل ۵۳: امنیت میزبان](#فصل-۵۳-امنیت-میزبان)
- [فصل ۵۴: رمزنگاری داده](#فصل-۵۴-رمزنگاری-داده)
- [فصل ۵۵: جمع‌بندی LPIC-1 و آزمون تمرینی](#فصل-۵۵-جمعبندی-lpic-1-و-آزمون-تمرینی)

---

## فصل ۵۲: وظایف مدیریت امنیت

### مقدمه

امنیت یک فرآیند مداوم است، نه یک کار یک‌باره. تصور کنید یک کارمند جدید می‌آید — باید حداقل دسترسی را بدهید. کارمندی می‌رود — باید حسابش را قفل کنید. یک فایل مشکوک پیدا می‌شود — باید بررسی کنید.

در این فصل وظایف روزمرهٔ مدیریت امنیت را یاد می‌گیرید: مدیریت رمز، sudo، حسابرسی و بررسی سیستم.

### مفاهیم اساسی

**مفاهیم کلیدی:**

- **Principle of Least Privilege:** حداقل دسترسی لازم.
- **sudo:** اجرای دستور با دسترسی دیگر.
- **SUID/SGID:** اجرای فایل با دسترسی مالک/گروه.
- **Audit:** ثبت رویدادهای امنیتی.
- **Hardening:** کاهش سطح حمله.

**فایل‌های مهم:**

| فایل | کار |
|---|---|
| `/etc/sudoers` | پیکربندی sudo |
| `/etc/shadow` | رمزهای رمزنگاری‌شده |
| `/etc/security/` | پیکربندی امنیتی (PAM) |
| `/etc/security/limits.conf` | محدودیت منابع |
| `/var/log/auth.log` | لاگ احراز هویت |

**سطوح دسترسی:**

| سطح | توضیح |
|---|---|
| کاربر عادی | محدود |
| sudo | دستور مشخص |
| ریشه | کامل |

### مثال‌های عملی

**پیکربندی sudo:**

```bash
# ویرایش امن
sudo visudo
sudo visudo -f /etc/sudoers.d/myfile

# نمونه:
# user ALL=(ALL:ALL) ALL              ← دسترسی کامل
# %developers ALL=(ALL) /usr/bin/systemctl  ← فقط یک دستور
# user ALL=(ALL) NOPASSWD: ALL        ← بدون رمز (احتیاط)
# user ALL=(ALL) NOPASSWD: /usr/bin/apt update  ← فقط یک دستور بدون رمز

# بررسی
sudo -l
sudo -l -U ali
```

**مدیریت رمز:**

```bash
# تغییر رمز
passwd
sudo passwd user

# سیاست رمز
sudo chage -l user
sudo chage -M 90 user      # حداکثر ۹۰ روز
sudo chage -E 2027-01-01 user  # تاریخ انقضا

# قفل حساب
sudo passwd -l user
sudo usermod -L user

# وضعیت
sudo passwd -S user
```

**حسابرسی:**

```bash
# فایل‌های SUID (خطر امنیتی)
sudo find / -perm -4000 -type f 2>/dev/null

# فایل‌های SGID
sudo find / -perm -2000 -type f 2>/dev/null

# فایل‌های بدون مالک
sudo find / -nouser -o -nogroup 2>/dev/null

# پوشه‌های world-writable
sudo find / -type d -perm -0002 2>/dev/null

# آخرین ورودها
last
lastlog
who
w

# تلاش‌های ناموفق
sudo lastb

# اتصالات فعلی
ss -tunap
```

**محدودیت منابع (ulimit):**

```bash
# محدودیت فایل باز
ulimit -n
ulimit -n 4096     # تنظیم موقت

# محدودیت پروسه
ulimit -u

# در /etc/security/limits.conf
# user hard nproc 100
# @devs soft nofile 4096
```

### مقایسه و الگوهای مختلف

**روش‌های بالا بردن دسترسی:**

| روش | امنیت | کاربرد |
|---|---|---|
| `su` | نیاز به رمز ریشه | جلسهٔ کامل |
| `sudo` | قابل حسابرسی | دستور مشخص |
| SUID | خطرناک | خاص |

**su در برابر sudo:**

| ویژگی | su | sudo |
|---|---|---|
| رمز | ریشه | کاربر خودش |
| حسابرسی | محدود | کامل |
| دانه‌بندی | همه/هیچ | دقیق |
| توصیه | — | ✅ |

**سطوح sudo:**

| سطح | مثال |
|---|---|
| همه‌چیز | `ALL=(ALL) ALL` |
| گروه | `%admin ALL=(ALL) ALL` |
| دستور مشخص | `user ALL=(ALL) /bin/systemctl` |
| بدون رمز | `NOPASSWD: ...` |

### اشکال شایع

**اشکال ۱: ویرایش مستقیم sudoers.**
خراب شود، sudo از کار می‌افتد؛ از `visudo` استفاده کنید. راه‌حل: `visudo`.

**اشکال ۲: NOPASSWD برای همه.**
امنیت را نابود می‌کند. راه‌حل: فقط برای دستورهای خاص.

**اشکال ۳: SUID روی اسکریپت.**
خطرناک است؛ معمولاً اشتباه. راه‌حل: پرهیز.

**اشکال ۴: نادیده گرفتن حساب‌های بی‌استفاده.**
باید قفل شوند. راه‌حل: `passwd -l`.

**اشکال ۵: رمز بدون انقضا.**
خطر امنیتی. راه‌حل: `chage -M`.

**اشکال ۶: فراموش کردن حسابرسی SUID.**
مهاجم ممکن است SUID نصب کند. راه‌حل: `find -perm -4000`.

### نکات مهم (Tips & Tricks)

- **همیشه `visudo` برای ویرایش sudoers.**
- **`sudo -l` دسترسی‌های sudo شما را نشان می‌دهد.**
- **حساب‌های سیستمی باید shell غیرقابل ورود داشته باشند.**
- **`chage -l` سیاست رمز را نشان می‌دهد.**
- **`find -perm -4000` را دوره‌ای بررسی کنید.**
- **برای حسابرسی از `auditd` استفاده کنید.**
- **`/etc/sudoers.d/` برای فایل‌های جدا (نه ویرایش sudoers).**
- **`sudo -i` یک جلسهٔ ریشهٔ تعاملی.**
- **`su - user` جلسهٔ کامل کاربر.**
- **`lastb` تلاش‌های ناموفق ورود.**
- **`pam_faillock` قفل پس از تلاش‌های ناموفق.**
- **`lynis` حسابرسی امنیت سیستم.**

### خلاصه فصل

امنیت بر حداقل دسترسی استوار است. `sudo` (با `visudo`) دسترسی کنترل‌شده می‌دهد. رمزها با `passwd`/`chage` مدیریت می‌شوند. حسابرسی با `last`, `lastlog`, `find -perm -4000` انجام می‌شود. حساب‌های بی‌استفاده را قفل کنید و از ویرایش مستقیم sudoers پرهیز کنید. `sudo` امن‌تر و قابل حسابرسی‌تر از `su` است.

---

## فصل ۵۳: امنیت میزبان

### مقدمه

**امنیت میزبان (Host Security)** یعنی محافظت از خود سرور: کنترل دسترسی، فایروال، سرویس‌های امن و مانیتورینگ.

تصور کنید سروری با پورت‌های باز اضافی راه‌اندازی می‌کنید — هر پورت باز یک درِ ورود برای مهاجم است. کاهش سطح حمله، اصل اول امنیت میزبان است.

### مفاهیم اساسی

**مفاهیم کلیدی:**

- **TCP Wrappers:** کنترل دسترسی بر اساس host (قدیمی).
- **Firewall:** فیلتر بسته‌ها (iptables, firewalld, ufw).
- **Port Scanning:** کشف پورت‌های باز.
- **Shadow passwords:** رمز جدا از passwd.
- **Attack surface:** سطح حمله.

**فلسفهٔ امنیت:** کاهش سطح حمله — هر سرویس غیرضروری را خاموش کنید.

**لایه‌های دفاعی:**

```
۱. فایروال (شبکه)
۲. TCP Wrappers (host)
۳. سرویس‌های امن (SSH به‌جای telnet)
۴. مجوز فایل
۵. SELinux/AppArmor
```

### مثال‌های عملی

**فایروال ufw (Ubuntu):**

```bash
# فعال‌سازی
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing

# اجازه
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow from 192.168.1.0/24 to any port 3306

# رد
sudo ufw deny 23

# وضعیت
sudo ufw status verbose
sudo ufw status numbered

# حذف
sudo ufw delete allow 80/tcp
sudo ufw delete 3
```

**فایروال firewalld (RHEL):**

```bash
# فعال‌سازی
sudo systemctl enable --now firewalld

# اجازه
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --add-port=8080/tcp --permanent
sudo firewall-cmd --reload

# بررسی
sudo firewall-cmd --list-all
sudo firewall-cmd --list-services
sudo firewall-cmd --list-ports

# zone
sudo firewall-cmd --get-active-zones
```

**iptables (پایه):**

```bash
# لیست قوانین
sudo iptables -L -n -v

# اجازه SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# سیاست پیش‌فرض
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# ذخیره
sudo iptables-save > /etc/iptables/rules.v4
```

**بررسی پورت‌ها:**

```bash
# پورت‌های باز محلی
ss -tlnp

# اسکن از بیرون (nmap)
nmap -sT localhost
nmap -p- localhost      # همهٔ پورت‌ها
```

**سخت‌سازی:**

```bash
# سرویس‌های فعال
systemctl list-unit-files --state=enabled

# غیرفعال کردن سرویس‌های غیرضروری
sudo systemctl disable telnet.socket
sudo systemctl disable rsh.socket

# بررسی شنودها
ss -tulnp
```

**TCP Wrappers (قدیمی):**

```bash
# /etc/hosts.allow
sshd: 192.168.1.0/24

# /etc/hosts.deny
ALL: ALL
```

### مقایسه و الگوهای مختلف

**ابزارهای فایروال:**

| ابزار | سطح | مناسب |
|---|---|---|
| `ufw` | ساده | Ubuntu، مبتدی |
| `firewalld` | متوسط | RHEL، zone-based |
| `iptables`/`nftables` | پایین | کنترل کامل |

**توصیه:** برای مبتدی `ufw`، برای RHEL `firewalld`.

**سرویس‌های ناامن در برابر امن:**

| ناامن | امن |
|---|---|
| telnet | SSH |
| FTP | SFTP/FTPS |
| rsh/rlogin | SSH |
| HTTP | HTTPS |
| POP3 | POP3S |

**nftables در برابر iptables:**

| ویژگی | iptables | nftables |
|---|---|---|
| نسل | قدیمی | مدرن |
| کارایی | خوب | بهتر |
| syntax | پیچیده | ساده‌تر |
| پیشنهاد | legacy | آینده |

### اشکال شایع

**اشکال ۱: فایروال بدون قانون SSH.**
خودتان را قفل می‌کنید! راه‌حل: اول SSH را اجازه دهید.

**اشکال ۲: باز گذاشتن همه پورت‌ها.**
سطح حمله بالا می‌رود. راه‌حل: فقط لازم‌ها.

**اشکال ۳: نادیده گرفتن سرویس‌های پیش‌فرض.**
telnet, rsh ناامن‌اند. راه‌حل: غیرفعال کنید.

**اشکال ۴: فراموش کردن IPv6.**
فایروال باید هر دو را پوشش دهد. راه‌حل: `ip6tables`/`ufw`.

**اشکال ۵: قوانین ذخیره‌نشده.**
بعد از reboot پاک می‌شوند. راه‌حل: `iptables-save`.

**اشکال ۶: نبود مانیتورینگ.**
حمله دیده نمی‌شود. راه‌حل: `fail2ban`.

### نکات مهم (Tips & Tricks)

- **قبل از فعال‌سازی فایروال، قانون SSH را اضافه کنید.**
- **`ufw default deny incoming` سیاست پیش‌فرض امن است.**
- **`firewalld` با zone کار می‌کند (`--zone=public`).**
- **`ss -tlnp` اولین قدم برای دیدن سرویس‌های در معرض است.**
- **`nmap` برای اسکن از دید مهاجم.**
- **`lynis` ابزار حسابرسی امنیت است.**
- **`fail2ban` برای جلوگیری از brute-force.**
- **`iptables-save`/`iptables-restore` برای پایداری.**
- **`nft` جایگزین مدرن iptables.**
- **`systemctl list-unit-files --state=enabled` سرویس‌های فعال.**
- **`chkrootkit`/`rkhunter` برای تشخیص rootkit.**
- **`/etc/hosts.allow` و `deny` همچنان کار می‌کنند.**

### خلاصه فصل

امنیت میزبان شامل فایروال، کنترل دسترسی و کاهش سطح حمله است. `ufw` (ساده)، `firewalld` (zone-based) و `iptables`/`nftables` (پایه) ابزارهای فایروال‌اند. با `ss -tlnp` پورت‌های باز را ببینید و سرویس‌های غیرضروری را خاموش کنید. قبل از فعال‌سازی فایروال، قانون SSH را اضافه کنید. `fail2ban` و `lynis` ابزارهای مکمل امنیتی‌اند.

---

## فصل ۵۴: رمزنگاری داده

### مقدمه

**رمزنگاری (Encryption)** داده را در برابر دسترسی غیرمجاز محافظت می‌کند. تصور کنید یک لپ‌تاپ گم می‌شود — اگر دیسکش رمزنگاری شده باشد، داده در امان است. یا یک فایل حساس را می‌خواهید به کسی بفرستید — با GPG رمزنگاری می‌کنید.

در این فصل رمزنگاری فایل، انتقال امن و امضای دیجیتال را یاد می‌گیرید.

### مفاهیم اساسی

**دو نوع رمزنگاری:**

- **Symmetric (متقارن):** یک کلید برای رمز و رمزگشایی (AES).
- **Asymmetric (نامتقارن):** جفت کلید عمومی/خصوصی (RSA).

**ابزارها:**

- **GnuPG (gpg):** رمزنگاری و امضا.
- **OpenSSL:** رمزنگاری و گواهی.
- **dm-crypt/LUKS:** رمزنگاری دیسک.
- **ssh-keygen:** کلید SSH.

**مفاهیم کلیدی:**

- **Hash:** تابع یک‌طرفه (SHA-256).
- **Digital Signature:** تأیید اصالت.
- **Certificate:** گواهی کلید عمومی.
- **PGP:** Pretty Good Privacy.

**مقایسه symmetric و asymmetric:**

| ویژگی | Symmetric | Asymmetric |
|---|---|---|
| کلید | یکی | جفت |
| سرعت | سریع | کند |
| کاربرد | حجم زیاد | تبادل کلید |
| مثال | AES | RSA |

### مثال‌های عملی

**رمزنگاری فایل با GPG:**

```bash
# رمزنگاری متقارن (با رمز)
gpg -c file.txt              # تولید file.txt.gpg
gpg file.txt.gpg             # رمزگشایی

# تولید کلید
gpg --gen-key
gpg --full-generate-key

# فهرست کلیدها
gpg --list-keys
gpg --list-secret-keys

# رمزنگاری با کلید عمومی
gpg -e -r user@example.com file.txt
gpg -d file.txt.gpg          # رمزگشایی

# امضا
gpg --sign file.txt
gpg --detach-sign file.txt
gpg --verify file.txt.sig file.txt

# صادرات
gpg --export -a user@example.com > public.asc
gpg --import public.asc
```

**Hash:**

```bash
sha256sum file.iso
md5sum file.iso
sha1sum file.iso

# بررسی صحت
sha256sum -c checksum.txt
echo "hash  file" | sha256sum -c

# محاسبه
openssl dgst -sha256 file.txt
```

**OpenSSL:**

```bash
# رمزنگاری فایل
openssl enc -aes-256-cbc -salt -in file.txt -out file.enc
openssl enc -d -aes-256-cbc -in file.enc -out file.txt

# ساخت گواهی خودامضا
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365

# بررسی گواهی
openssl x509 -in cert.pem -text -noout

# تست SSL
openssl s_client -connect example.com:443
```

**رمزنگاری دیسک (LUKS):**

```bash
# ساخت
sudo cryptsetup luksFormat /dev/sdb1
sudo cryptsetup open /dev/sdb1 myvolume
sudo mkfs.ext4 /dev/mapper/myvolume
sudo mount /dev/mapper/myvolume /mnt

# بستن
sudo umount /mnt
sudo cryptsetup close myvolume

# مدیریت
sudo cryptsetup luksDump /dev/sdb1
sudo cryptsetup luksAddKey /dev/sdb1
sudo cryptsetup luksRemoveKey /dev/sdb1

# وضعیت
sudo cryptsetup status myvolume
```

### مقایسه و الگوهای مختلف

**ابزارهای رمزنگاری:**

| ابزار | کاربرد |
|---|---|
| `gpg` | رمزنگاری فایل/ایمیل، امضا |
| `openssl` | TLS، گواهی، رمزنگاری |
| `cryptsetup` | رمزنگاری دیسک |
| `ssh-keygen` | کلید SSH |

**توابع hash:**

| الگوریتم | طول | امنیت |
|---|---|---|
| MD5 | ۱۲۸ | شکسته (استفاده نکنید) |
| SHA-1 | ۱۶۰ | ضعیف |
| SHA-256 | ۲۵۶ | امن |
| SHA-512 | ۵۱۲ | بسیار امن |

**الگوریتم‌های رمزنگاری:**

| الگوریتم | نوع | امنیت |
|---|---|---|
| AES-256 | symmetric | عالی |
| RSA-4096 | asymmetric | عالی |
| ChaCha20 | symmetric | عالی |
| 3DES | symmetric | منسوخ |

### اشکال شایع

**اشکال ۱: فراموش کردن رمز کلید.**
داده برای همیشه از دست می‌رود. راه‌حل: رمز را در جای امن.

**اشکال ۲: استفاده از MD5.**
ناامن است. راه‌حل: SHA-256+.

**اشکال ۳: ذخیرهٔ کلید خصوصی ناامن.**
باید محافظت شود (chmod 600). راه‌حل: مجوز ۶۰۰.

**اشکال ۴: اشتباه symmetric/asymmetric.**
با یک کلید عمومی نمی‌توان رمزگشایی کرد. راه‌حل: درک تفاوت.

**اشکال ۵: LUKS بدون پشتیبان header.**
خرابی header = از دست رفتن داده. راه‌حل: `luksHeaderBackup`.

**اشکال ۶: رمز ضعیف.**
brute-force ممکن. راه‌حل: رمز قوی.

### نکات مهم (Tips & Tricks)

- **`gpg --list-keys` کلیدها را نشان می‌دهد.**
- **`ssh-keygen -t ed25519` کلید SSH مدرن می‌سازد.**
- **`sha256sum` برای تأیید دانلود.**
- **کلید خصوصی را هرگز به اشتراک نگذارید.**
- **`gpg --armor` خروجی متنی (base64) می‌سازد.**
- **برای LUKS، رمز قوی و پشتیبان header داشته باشید.**
- **`gpg --symmetric` رمزنگاری متقارن.**
- **`openssl enc` برای رمزنگاری سریع فایل.**
- **`sha256sum -c` فایل checksum را بررسی می‌کند.**
- **`cryptsetup luksHeaderBackup` پشتیبان header.**
- **`gpg-agent` کلیدها را cache می‌کند.**
- **برای رمزنگاری کل دیسک، LUKS + LVM.**

### خلاصه فصل

رمزنگاری متقارن (AES) و نامتقارن (RSA) دو نوع اصلی‌اند. `gpg` برای فایل/ایمیل، `openssl` برای TLS/گواهی و `cryptsetup` (LUKS) برای دیسک به‌کار می‌روند. `sha256sum` برای hash و بررسی صحت است. از MD5 پرهیز کنید و کلید خصوصی را محافظت کنید. فراموش کردن رمز = از دست رفتن داده. LUKS header را پشتیبان بگیرید.

---

## فصل ۵۵: جمع‌بندی LPIC-1 و آزمون تمرینی

### مقدمه

تبریک! شما تمام مباحث LPIC-1 را پوشش دادید. در این فصل مرور نهایی و یک آزمون تمرینی انجام می‌دهید.

### مفاهیم اساسی

**مرور مباحث LPIC-1:**

| Topic | موضوع | فایل |
|---|---|---|
| 101 | System Architecture | ۰۳ |
| 102 | Installation & Packages | ۰۴ |
| 103 | GNU/Unix Commands | ۰۵، ۰۶ |
| 104 | Devices & Filesystems | ۰۷، ۰۸ |
| 105 | Shells & Scripting | ۰۹ |
| 106 | User Interfaces | ۱۰ |
| 107 | Administrative Tasks | ۱۱ |
| 108 | System Services | ۱۲ |
| 109 | Networking | ۱۳ |
| 110 | Security | ۱۴ |

### مثال‌های عملی

**آزمون تمرینی (۱۵ سؤال):**

```
۱. کدام دستور فایل‌سیستم ext4 می‌سازد؟
   A) mkfs.ext4   B) fsck.ext4   C) e2fsck   D) format
   پاسخ: A

۲. برای افزودن کاربر به گروه docker بدون حذف گروه‌های قبلی:
   A) usermod -G docker ali   B) usermod -aG docker ali
   C) groupadd ali docker     D) adduser ali docker
   پاسخ: B

۳. کدام فایل رمزهای کاربران را نگه می‌دارد؟
   A) /etc/passwd   B) /etc/shadow   C) /etc/group   D) /etc/secure
   پاسخ: B

۴. خروجی کدام دستور فقط خطاها را دور می‌ریزد؟
   A) cmd > /dev/null   B) cmd 2> /dev/null
   C) cmd &> /dev/null  D) cmd < /dev/null
   پاسخ: B

۵. کدام target معادل runlevel 3 است؟
   A) rescue.target   B) multi-user.target
   C) graphical.target   D) basic.target
   پاسخ: B

۶. برای یافتن فایل‌های SUID:
   A) find / -perm -4000   B) find / -suid
   C) ls -l / | grep s     D) locate suid
   پاسخ: A

۷. crontab «0 2 * * *» چه زمانی اجرا می‌شود؟
   A) هر ۲ دقیقه   B) هر روز ساعت ۲ بامداد
   C) هر ۲ ساعت   D) روز دوم ماه
   پاسخ: B

۸. کدام لینک بین فایل‌سیستم‌ها کار می‌کند؟
   A) hard link   B) symbolic link
   C) هر دو   D) هیچ‌کدام
   پاسخ: B

۹. مجوز 644 یعنی:
   A) rwxr--r--   B) rw-r--r--   C) rw-rw-r--   D) r--r--r--
   پاسخ: B

۱۰. برای مشاهدهٔ لاگ‌های این بوت:
    A) journalctl -b   B) dmesg -f
    C) cat /var/log   D) tail -f /var/log
    پاسخ: A

۱۱. کدام فایل نقطهٔ mount دائمی را تعیین می‌کند؟
    A) /etc/mtab   B) /etc/fstab   C) /proc/mounts   D) /etc/mount
    پاسخ: B

۱۲. برای تغییر owner و group با هم:
    A) chmod user:group   B) chown user:group
    C) chgrp user:group   D) chmod 755
    پاسخ: B

۱۳. کدام ابزار پورت‌های باز را نشان می‌دهد؟
    A) ifconfig   B) ss -tlnp   C) route   D) ping
    پاسخ: B

۱۴. کدام دستور فایل‌سیستم را بررسی و تعمیر می‌کند؟
    A) mkfs   B) fsck   C) mount   D) df
    پاسخ: B

۱۵. کدام متغیر اولویت را در locale بازنویسی می‌کند؟
    A) LANG   B) LC_ALL   C) LC_TIME   D) LANGUAGE
    پاسخ: B
```

### مقایسه و الگوهای مختلف

**چک‌لیست آمادگی:**

```
☑ همهٔ ۱۰ Topic را مطالعه کرده‌ام
☑ هر Topic را در آزمایشگاه تمرین کرده‌ام
☑ دستورهای کلیدی را حفظ کرده‌ام
☑ با man pages راحت هستم
☑ آزمون تمرینی داده‌ام (بالای ۸۰٪)
☑ تفاوت Debian و RHEL را می‌دانم
☑ با systemd، شبکه و امنیت پایه آشنا هستم
```

**نقاط ضعف رایج:**

| Topic | اشتباه رایج |
|---|---|
| 102 | dpkg vs rpm |
| 103 | regex و sed/awk |
| 104 | mount و fstab |
| 105 | اسکریپت‌نویسی |
| 107 | usermod -aG |
| 109 | nmcli/netplan |
| 110 | مسیر فایل‌های امنیتی |

### اشکال شایع

**اشکال ۱: مطالعهٔ سطحی.**
باید عمق داشته باشید. راه‌حل: آزمایشگاه عملی.

**اشکال ۲: نادیده گرفتن تفاوت توزیع‌ها.**
هر دو خانواده مهم‌اند. راه‌حل: جدول مقایسه.

**اشکال ۳: حفظ بدون تمرین.**
دست‌ها باید دستورها را بلد باشند. راه‌حل: تمرین.

**اشکال ۴: عجله در آزمون.**
تا آماده نشده‌اید ثبت‌نام نکنید. راه‌حل: آزمون تمرینی.

**اشکال ۵: فراموش کردن مرور.**
فراموشی طبیعی است. راه‌حل: چیت‌شیت.

**اشکال ۶: نادیده گرفتن Topicهای پراهمیت.**
103 و 104 وزن بالا دارند. راه‌حل: وقت بیشتر.

### نکات مهم (Tips & Tricks)

- **آزمون‌های تمرینی بیشتری در فایل ۲۸ خواهید دید.**
- **نقاط ضعف خود را شناسایی و مرور کنید.**
- **با `man` و `--help` در آزمون هم می‌توانید (اگر terminal access باشد) — ولی معمولاً نیست.**
- **تفاوت‌های کلیدی Debian/RHEL را یک صفحه یادداشت کنید.**
- **استراحت کافی قبل از آزمون.**
- **چیت‌شیت (فایل ۲۹) را مرور کنید.**
- **سؤالات غلط را در دفتر یادداشت کنید.**
- **برای هر Topic یک آزمایشگاه خلاصه بزنید.**
- **نمرهٔ قبولی ~۵۰۰ است، نه ۸۰۰.**
- **آرامش و تمرکز مهم‌تر از دانش کامل است.**

### خلاصه فصل

LPIC-1 شامل ۱۰ Topic از معماری سیستم تا امنیت است. این آزمون تمرینی ۱۵ سؤال نمونه داشت. چک‌لیست آمادگی را کامل کنید و نقاط ضعف را مرور کنید. حالا برای LPIC-2 آماده‌اید.

---

## منابع و مراجع

- man pages: `man sudoers`, `man ufw`, `man gpg`, `man cryptsetup`, `man sha256sum`
- LPI Security Objectives: https://www.lpi.org
- GnuPG Documentation: https://www.gnupg.org/documentation/
- CIS Benchmarks: https://www.cisecurity.org/cis-benchmarks
- NIST Cybersecurity Framework: https://www.nist.gov/cyberframework

**بعدی:** [فایل ۱۵ — برنامه‌ریزی ظرفیت](15-Capacity-Planning.md)
