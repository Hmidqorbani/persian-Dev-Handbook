# Enumeration (شمارش سرویس‌ها)

**مدت یادگیری**: ۴ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۵ (اسکن شبکه)

---

## فهرست محتوا

- [فصل ۱: مقدمه Enumeration](#فصل-۱-مقدمه-enumeration)
- [فصل ۲: شمارش NetBIOS و SMB](#فصل-۲-شمارش-netbios-و-smb)
- [فصل ۳: شمارش SNMP، LDAP و NFS](#فصل-۳-شمارش-snmp-ldap-و-nfs)
- [فصل ۴: شمارش DNS، SMTP و NTP](#فصل-۴-شمارش-dns-smtp-و-ntp)
- [فصل ۵: شمارش هوشمند با AI](#فصل-۵-شمارش-هوشمند-با-ai)
- [فصل ۶: آزمایشگاه و اقدامات متقابل](#فصل-۶-آزمایشگاه-و-اقدامات-متقابل)

---

## فصل ۱: مقدمه Enumeration

### مقدمه

**Enumeration** مرحله بعد از اسکن است. در اینجا نه فقط پورت باز، بلکه **کاربران، اشتراک‌ها، منابع و اطلاعات دقیق** را از سرویس‌های فعال بیرون می‌کشیم. Enumeration یک تعامل **فعال** با هدف است.

> **تفاوت با اسکن:** اسکن می‌گوید «درِ ۴۴۵ باز است». Enumeration می‌گوید «درِ ۴۴۵ باز است، دو کاربر به نام‌های admin و guest دارد و اشتراک C$ قابل دسترسی است».

### مفاهیم اساسی

**سرویس‌های رایج برای Enumeration:**
- **NetBIOS/SMB** (پورت ۱۳۷، ۱۳۹، ۴۴۵)
- **SNMP** (پورت ۱۶۱)
- **LDAP** (پورت ۳۸۹)
- **NFS** (پورت ۲۰۴۹)
- **DNS** (پورت ۵۳)
- **SMTP** (پورت ۲۵)
- **NTP** (پورت ۱۲۳)
- **RPC** (پورت ۱۳۵)

**اطلاعاتی که به‌دست می‌آید:** نام کاربری، گروه‌ها، اشتراک‌های شبکه، نسخه سیستم، اطلاعات DNS، حساب‌های SNMP community.

### نکات مهم (Tips & Tricks)

- Enumeration فعال است و **در لاگ ثبت می‌شود**.
- اطلاعات کاربران، ورودی طلایی برای **کرک پسورد** (فایل ۰۸) است.
- هر سرویس ابزار مخصوص خود را دارد.

### خلاصه فصل

- Enumeration = استخراج اطلاعات دقیق از سرویس‌ها.
- فعال و قابل ثبت در لاگ.
- ورودی فاز بهره‌برداری است.

---

## فصل ۲: شمارش NetBIOS و SMB

### مقدمه

**SMB (Server Message Block)** پروتکل اشتراک فایل ویندوز است و یکی از پرمخاطره‌ترین سرویس‌ها در تست نفوذ.

### مفاهیم اساسی

**NetBIOS:** سرویس نام‌گذاری قدیمی ویندوز (پورت ۱۳۷-۱۳۹).
**SMB:** اشتراک فایل و چاپگر (پورت ۴۴۵). نسخه‌های SMBv1 (آسیب‌پذیر)، SMBv2، SMBv3.

**آسیب‌پذیری معروف:** EternalBlue (MS17-010) که در حمله WannaCry استفاده شد.

### مثال عملی

```bash
# --- enum4linux (ابزار جامع) ---
enum4linux -a 192.168.56.101
# -a : همه تست‌ها

# --- smbclient (لیست اشتراک‌ها) ---
smbclient -L //192.168.56.101 -N
# -N : بدون رمز

# --- اتصال به اشتراک ---
smbclient //192.168.56.101/share -N

# --- nmblookup (نام NetBIOS) ---
nmblookup -A 192.168.56.101

# --- Nmap SMB scripts ---
nmap --script smb-enum-shares,smb-enum-users -p 445 192.168.56.101

# --- بررسی آسیب‌پذیری EternalBlue ---
nmap --script smb-vuln-ms17-010 -p 445 192.168.56.101
```

### اشکال شایع

- **مشکل:** smbclient خطای پروتکل می‌دهد → نسخه SMB را با `--option='client min protocol=NT1'` تنظیم کنید.
- **مشکل:** enum4linux نصب نیست → `sudo apt install enum4linux`.

### نکات مهم (Tips & Tricks)

- اشتراک‌های `C$`، `ADMIN$`، `IPC$` اشتراک‌های پیش‌فرض مدیریتی‌اند.
- **Null Session** (بدون احراز هویت) در سیستم‌های بدپیکربندی پاسخ می‌دهد.
- همیشه بررسی MS17-010 (EternalBlue) در ویندوزهای قدیمی.

### خلاصه فصل

- SMB پرخطرترین سرویس ویندوز است.
- enum4linux، smbclient و NSE ابزارهای اصلی.
- EternalBlue نمونه کلاسیک آسیب‌پذیری SMB است.

---

## فصل ۳: شمارش SNMP، LDAP و NFS

### مقدمه

سه سرویس مهم دیگر که اطلاعات ارزشمندی لو می‌دهند.

### مفاهیم اساسی

**SNMP (پورت ۱۶۱):** مدیریت شبکه. **Community String** نقش رمز را دارد؛ پیش‌فرض `public` و `private`.

**LDAP (پورت ۳۸۹):** سرویس دایرکتوری (Active Directory). اطلاعات کاربران و گروه‌ها.

**NFS (پورت ۲۰۴۹):** اشتراک فایل لینوکس. اغلب بدون احراز هویت.

### مثال عملی

```bash
# --- SNMP ---
snmpwalk -v2c -c public 192.168.56.101
snmp-check 192.168.56.101 -c public
onesixtyone -c community.txt 192.168.56.101

# --- LDAP ---
ldapsearch -x -h 192.168.56.101 -b "dc=example,dc=com"
nmap --script ldap-search -p 389 192.168.56.101

# --- NFS ---
showmount -e 192.168.56.101
# mount کردن اشتراک NFS
sudo mount -t nfs 192.168.56.101:/share /mnt/nfs
```

### اشکال شایع

- **مشکل:** snmpwalk پاسخ نمی‌دهد → community string اشتباه است؛ `public` را امتحان کنید.
- **مشکل:** mount NFS خطا می‌دهد → پکیج `nfs-common` نصب کنید.

### نکات مهم (Tips & Tricks)

- Community string ضعیف SNMP = اطلاعات کامل شبکه لو می‌رود.
- NFS با `no_root_squash` اجازه دسترسی روت می‌دهد — بسیار خطرناک.
- LDAP Anonymous Bind گاهی فعال است و اطلاعات لو می‌دهد.

### خلاصه فصل

- SNMP: community string پیش‌فرض خطرناک.
- LDAP: منبع اطلاعات Active Directory.
- NFS: اغلب بدون احراز هویت.

---

## فصل ۴: شمارش DNS، SMTP و NTP

### مقدمه

این سه سرویس نیز اطلاعات مهمی افشا می‌کنند.

### مفاهیم اساسی

**DNS Enumeration:** کشف رکوردها، زیردامنه‌ها و Zone Transfer.
**SMTP Enumeration:** کشف کاربران با دستور `VRFY`.
**NTP Enumeration:** کشف نسخه و اطلاعات زمان‌بندی.

### مثال عملی

```bash
# --- DNS ---
dnsrecon -d example.com -t std
dnsenum example.com
dig axfr @ns1.example.com example.com   # Zone Transfer
nmap --script dns-zone-transfer -p 53 ns1.example.com

# --- SMTP ---
smtp-user-enum -M VRFY -U users.txt -t 192.168.56.101
# یا دستی:
nc 192.168.56.101 25
# VRFY root
# EXPN admin

# --- NTP ---
ntpq -p 192.168.56.101
nmap --script ntp-info -p 123 192.168.56.101
```

### اشکال شایع

- **مشکل:** Zone Transfer رد می‌شود → طبیعی است؛ بیشتر سرورها غیرفعال کرده‌اند.
- **مشکل:** VRFY غیرفعال است → `EXPN` یا `RCPT TO` را امتحان کنید.

### نکات مهم (Tips & Tricks)

- Zone Transfer موفق = لیست کامل زیردامنه‌ها (طلا!).
- VRFY/EXPN راهی کلاسیک برای کشف کاربران ایمیل.
- سرور NTP قدیمی می‌تواند هدف حمله Amplification (فایل ۱۲) باشد.

### خلاصه فصل

- DNS Enumeration: کشف رکورد و Zone Transfer.
- SMTP Enumeration: کشف کاربران با VRFY/EXPN.
- NTP Enumeration: اطلاعات نسخه و زمان.

---

## فصل ۵: شمارش هوشمند با AI

### مقدمه

هوش مصنوعی می‌تواند خروجی خام Enumeration را تحلیل، دسته‌بندی و اولویت‌بندی کند.

### مفاهیم اساسی

**کاربردهای AI در Enumeration:**
1. **تفسیر خروجی خام:** تبدیل لاگ‌های طولانی به بینش قابل‌فهم.
2. **کشف الگو:** یافتن نام‌های کاربری با الگوی مشابه.
3. **پیش‌بینی اعتبار:** حدس‌زدن کاربران معتبر.
4. **تولید لیست هدف:** ساخت فایل کاربران برای brute-force.

### مثال عملی

```text
# پرامپت نمونه برای تحلیل خروجی enum4linux:
"این خروجی enum4linux است: [متن].
لطفاً:
1. لیست کاربران و گروه‌ها را استخراج کن
2. اشتراک‌های قابل دسترسی بدون رمز را مشخص کن
3. آسیب‌پذیری‌های احتمالی را اولویت‌بندی کن
خروجی را در سه جدول جدا بده."
```

```bash
# تولید لیست کاربران برای brute-force
# پس از استخراج نام‌ها، فایل users.txt بسازید و در Hydra استفاده کنید
```

### اشکال شایع

- **مشکل:** AI نام کاربری جعلی تولید می‌کند → با منابع معتبر اعتبارسنجی کنید.
- **مشکل:** داده‌های حساس در سرویس عمومی → از مدل محلی استفاده کنید.

### نکات مهم (Tips & Tricks)

- AI برای **سازمان‌دهی** عالی است، اما تأیید نهایی با شما.
- از AI برای تولید wordlist سفارشی و هدفمند استفاده کنید.
- ترکیب Enumeration سنتی + AI = پوشش کامل.

### خلاصه فصل

- AI خروجی Enumeration را تحلیل و اولویت‌بندی می‌کند.
- تولید wordlist هدفمند یکی از کاربردهای مهم است.
- همیشه خروجی را اعتبارسنجی کنید.

---

## فصل ۶: آزمایشگاه و اقدامات متقابل

### مقدمه

بیایید یک Enumeration کامل روی Metasploitable2 انجام دهیم.

### مثال عملی — آزمایشگاه کامل

```bash
TARGET=192.168.56.101
mkdir -p enum/$TARGET

# --- SMB ---
enum4linux -a $TARGET | tee enum/$TARGET/enum4linux.txt
smbclient -L //$TARGET -N | tee enum/$TARGET/smbclient.txt

# --- SNMP ---
snmpwalk -v2c -c public $TARGET | tee enum/$TARGET/snmp.txt

# --- NFS ---
showmount -e $TARGET | tee enum/$TARGET/nfs.txt

# --- FTP ---
nmap --script ftp-anon -p 21 $TARGET

# --- Nmap جامع ---
sudo nmap -sV -sC -p- $TARGET -oA enum/$TARGET/nmap
```

### اشکال شایع

- **مشکل:** بعضی سرویس‌ها پاسخ نمی‌دهند → بررسی کنید سرویس فعال است (`nmap -sV`).
- **مشکل:** خروجی بسیار حجیم است → با `grep` و `awk` فیلتر کنید.

### نکات مهم (Tips & Tricks)

- همه خروجی‌ها را در پوشه سازمان‌یافته ذخیره کنید.
- یک **جدول خلاصه** از یافته‌ها بسازید.
- یافته‌های Enumeration را به فاز بهره‌برداری وصل کنید.

### اقدامات متقابل (Countermeasures)

| سرویس | راهکار دفاعی |
|-------|--------------|
| SMB | غیرفعال‌سازی SMBv1، محدودسازی اشتراک‌ها، پچ EternalBlue |
| SNMP | تغییر community string، استفاده از SNMPv3 |
| LDAP | غیرفعال‌سازی Anonymous Bind، LDAPS |
| NFS | محدودسازی به IP مجاز، حذف no_root_squash |
| DNS | غیرفعال‌سازی Zone Transfer |
| SMTP | غیرفعال‌سازی VRFY/EXPN |

### خلاصه فصل

- Enumeration کامل = ترکیب ابزارهای SMB/SNMP/NFS/DNS.
- مستندسازی ساختاریافته ضروری است.
- هر سرویس اقدام متقابل مشخص دارد.

---

## منابع و مراجع

- **enum4linux:** https://github.com/CiscoCXSecurity/enum4linux
- **Samba (smbclient):** https://www.samba.org/
- **snmpwalk (Net-SNMP):** http://www.net-snmp.org/
- **dnsrecon:** https://github.com/darkoperator/dnsrecon
- **smtp-user-enum:** https://github.com/pentestmonkey/smtp-user-enum
- **MS17-010 (EternalBlue):** https://docs.microsoft.com/en-us/security-updates/
- **HackTricks — SMB Enumeration:** https://book.hacktricks.xyz/
