# فایل ۲۵ — مدیریت کلاینت شبکه

**مدت یادگیری**: ۸ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: فایل ۱۳ (شبکه)، فایل ۱۱ (کاربران)

---

## فهرست محتوا

- [فصل ۸۶: پیکربندی DHCP](#فصل-۸۶-پیکربندی-dhcp)
- [فصل ۸۷: احراز هویت PAM](#فصل-۸۷-احراز-هویت-pam)
- [فصل ۸۸: استفاده از LDAP سمت کلاینت](#فصل-۸۸-استفاده-از-ldap-سمت-کلاینت)
- [فصل ۸۹: پیکربندی سرور OpenLDAP](#فصل-۸۹-پیکربندی-سرور-openldap)

---

## فصل ۸۶: پیکربندی DHCP

### مقدمه

**DHCP (Dynamic Host Configuration Protocol)** به‌طور خودکار به کلاینت‌ها آدرس IP، gateway و DNS می‌دهد. تصور کنید ۱۰۰ کامپیوتر در یک شبکه دارید — بدون DHCP باید همه را دستی پیکربندی کنید.

برای مدیریت شبکه‌های بزرگ حیاتی است و در آزمون می‌آید.

### مفاهیم اساسی

**مفاهیم کلیدی:**

- **Lease:** اجارهٔ آدرس (مدت‌دار).
- **Scope:** محدودهٔ آدرس‌ها.
- **Reservation:** آدرس ثابت برای MAC خاص.
- **Relay:** ارسال DHCP بین شبکه‌ها.
- **Option:** پارامتر اضافی.

**فایل‌های مهم:**

- `/etc/dhcp/dhcpd.conf` — پیکربندی.
- `/var/lib/dhcp/dhcpd.leases` — اجاره‌ها.

**پورت‌ها:** ۶۷ (سرور)، ۶۸ (کلاینت) — UDP.

### مثال‌های عملی

**نصب:**

```bash
sudo apt install isc-dhcp-server    # Debian
sudo dnf install dhcp-server        # RHEL
```

**پیکربندی `/etc/dhcp/dhcpd.conf`:**

```
option domain-name "example.com";
option domain-name-servers 192.168.1.10, 8.8.8.8;
option routers 192.168.1.1;
default-lease-time 600;
max-lease-time 7200;
authoritative;

subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option broadcast-address 192.168.1.255;
}

# رزرو برای MAC خاص
host printer {
    hardware ethernet 00:11:22:33:44:55;
    fixed-address 192.168.1.50;
}

# گروه
group {
    option domain-name-servers 192.168.1.10;
    host client1 { hardware ethernet AA:BB:CC:DD:EE:01; }
    host client2 { hardware ethernet AA:BB:CC:DD:EE:02; }
}
```

**فعال‌سازی:**

```bash
# Debian: تعیین رابط
# /etc/default/isc-dhcp-server: INTERFACESv4="eth0"

# بررسی
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf

# شروع
sudo systemctl restart isc-dhcp-server

# لاگ
tail -f /var/log/syslog | grep dhcp
```

**dnsmasq (سبک):**

```
# /etc/dnsmasq.conf
interface=eth0
dhcp-range=192.168.1.100,192.168.1.200,12h
dhcp-option=3,192.168.1.1
dhcp-option=6,8.8.8.8
dhcp-host=00:11:22:33:44:55,192.168.1.50
```

### مقایسه و الگوهای مختلف

**سرورهای DHCP:**

| سرور | ویژگی |
|---|---|
| **ISC DHCP** | کلاسیک، کامل |
| **dnsmasq** | سبک، DHCP+DNS |
| **Kea** | مدرن، جایگزین ISC |

**انواع تخصیص:**

| نوع | توضیح |
|---|---|
| Dynamic | از range |
| Automatic | اولین بار |
| Manual | رزرو MAC |

**گزینه‌های مهم:**

| Option | کار |
|---|---|
| 3 | gateway |
| 6 | DNS |
| 15 | domain name |
| 51 | lease time |
| 66/67 | boot server/file |

### اشکال شایع

**اشکال ۱: دو DHCP در شبکه.**
تداخل و اختلال. راه‌حل: فقط یکی.

**اشکال ۲: scope اشتباه.**
آدرس‌های خارج از شبکه. راه‌حل: subnet درست.

**اشکال ۳: نبود range.**
هیچ آدرسی توزیع نمی‌شود. راه‌حل: `range`.

**اشکال ۴: فایروال پورت ۶۷/۶۸.**
DHCP کار نمی‌کند. راه‌حل: باز کردن.

**اشکال ۵: نبود `authoritative`.**
سرور پاسخ نمی‌دهد. راه‌حل: `authoritative`.

**اشکال ۶: اشتباه در MAC رزرو.**
رزرو اعمال نمی‌شود. راه‌حل: MAC درست.

### نکات مهم (Tips & Tricks)

- **`dhcpd -t` پیکربندی را بررسی می‌کند.**
- **`dhcpd.leases` اجاره‌های فعلی را نشان می‌دهد.**
- **برای چند subnet، relay لازم است.**
- **`dhcp-host` برای رزرو ثابت.**
- **`dnsmasq` برای شبکهٔ کوچک ساده‌تر است.**
- **فقط یک DHCP در هر broadcast domain.**
- **`omapi` مدیریت آنلاین.**
- **`dhcp-lease-list` (Debian) اجاره‌ها را نشان می‌دهد.**
- **Option 66/67 برای PXE boot.**
- **`authoritative` در شبکهٔ واحد.**
- **`deny unknown-clients` امنیت.**
- **Kea جایگزین مدرن ISC.**

### خلاصه فصل

DHCP آدرس IP، gateway و DNS را خودکار توزیع می‌کند. `/etc/dhcp/dhcpd.conf` شامل subnet، range و reservation است. `dhcpd -t` بررسی و `dhcpd.leases` اجاره‌ها را نشان می‌دهد. فقط یک DHCP در هر شبکه. `dnsmasq` جایگزین سبک است و Kea مدرن.

---

## فصل ۸۷: احراز هویت PAM

### مقدمه

**PAM (Pluggable Authentication Modules)** چارچوبی برای احراز هویت در لینوکس است. تصور کنید می‌خواهید برای SSH احراز دو-عاملی یا محدودیت زمانی اضافه کنید — PAM این کار را ممکن می‌کند.

در این فصل کار با PAM را یاد می‌گیرید.

### مفاهیم اساسی

**چهار نوع ماژول PAM:**

| نوع | کار |
|---|---|
| `auth` | احراز هویت |
| `account` | اعتبار حساب |
| `password` | تغییر رمز |
| `session` | تنظیم جلسه |

**فایل‌های PAM:**

- `/etc/pam.d/` — پیکربندی هر سرویس.
- `/etc/security/` — محدودیت‌ها.

**کنترل‌ها:**

| کنترل | معنا |
|---|---|
| `required` | باید موفق باشد |
| `requisite` | اگر شکست، فوراً رد |
| `sufficient` | اگر موفق، کافی |
| `optional` | اختیاری |
| `include` | شامل فایل دیگر |

### مثال‌های عملی

**دیدن پیکربندی:**

```bash
cat /etc/pam.d/login
cat /etc/pam.d/su
cat /etc/pam.d/sshd
cat /etc/pam.d/common-auth    # Debian
```

**محدودیت منابع (`/etc/security/limits.conf`):**

```
# <domain>  <type>  <item>   <value>
*           soft    nproc    100
*           hard    nproc    200
@developers soft    nofile   4096
alice       hard    core     0
```

**محدودیت ورود (`/etc/security/access.conf`):**

```
# کاربر    type    list
+ : root : LOCAL
+ : @admin : ALL
- : ALL : ALL
```

**سیاست رمز (`/etc/security/pwquality.conf`):**

```
minlen = 12
dcredit = -1
ucredit = -1
ocredit = -1
lcredit = -1
```

**قفل پس از تلاش ناموفق:**

```
# /etc/pam.d/common-auth
auth required pam_faillock.so preauth
auth [default=die] pam_faillock.so authfail
account required pam_faillock.so
```

```bash
faillock --user alice
faillock --user alice --reset
```

### مقایسه و الگوهای مختلف

**ماژول‌های رایج:**

| ماژول | کار |
|---|---|
| `pam_unix.so` | رمز یونیکس |
| `pam_limits.so` | محدودیت منابع |
| `pam_faillock`/`pam_tally2` | قفل |
| `pam_access.so` | کنترل دسترسی |
| `pam_pwquality.so` | کیفیت رمز |
| `pam_env.so` | متغیر محیطی |
| `pam_succeed_if.so` | شرط |

**ترتیب کنترل:**

| ترتیب | رفتار |
|---|---|
| `required` | اجرا و بررسی |
| `requisite` | توقف در خطا |
| `sufficient` | موفق = کافی |
| `optional` | نادیده اگر نبود |

**ابزارها:**

| ابزار | کار |
|---|---|
| `pam-auth-update` | پیکربندی (Debian) |
| `authconfig` | پیکربندی (RHEL) |
| `faillock` | مدیریت قفل |

### اشکال شایع

**اشکال ۱: خطای PAM = قفل شدن سیستم.**
همیشه یک جلسهٔ ریشه باز نگه دارید! راه‌حل: جلسهٔ پشتیبان.

**اشکال ۲: ترتیب اشتباه ماژول‌ها.**
احراز هویت می‌شکند. راه‌حل: ترتیب درست.

**اشکال ۳: `required` به‌جای `sufficient`.**
رفتار متفاوت. راه‌حل: آگاه باشید.

**اشکال ۴: قفل حساب ریشه.**
خودتان را قفل می‌کنید. راه‌حل: احتیاط.

**اشکال ۵: خطای syntax.**
سرویس کار نمی‌کند. راه‌حل: بررسی.

**اشکال ۶: فراموش کردن module.**
ماژول نصب نیست. راه‌حل: نصب.

### نکات مهم (Tips & Tricks)

- **قبل از تغییر PAM، یک ترمینال ریشه باز نگه دارید.**
- **`pam-auth-update` (Debian) پیکربندی تعاملی.**
- **`faillock --user alice` وضعیت قفل را نشان می‌دهد.**
- **`/etc/security/limits.conf` برای محدودیت.**
- **`pam_access` برای محدودیت ورود بر اساس IP.**
- **تغییرات PAM فوری اعمال می‌شوند.**
- **`pam_tally2` نسخهٔ قدیمی `pam_faillock`.**
- **`limits.d/` برای فایل‌های جدا.**
- **`pam_env` متغیر محیطی.**
- **`pam_mkhomedir` پوشهٔ خانه خودکار.**
- **`pam_sss` برای SSSD/LDAP.**
- **`pamtester` تست PAM.**

### خلاصه فصل

PAM چارچوب احراز هویت است با چهار نوع ماژول (auth, account, password, session). `/etc/pam.d/` پیکربندی هر سرویس و `/etc/security/` محدودیت‌ها. کنترل‌های required/requisite/sufficient/optional ترتیب اجرا را تعیین می‌کنند. قبل از تغییر PAM یک جلسهٔ ریشه باز نگه دارید. `pam_faillock` و `pam_limits` ابزارهای مهم‌اند.

---

## فصل ۸۸: استفاده از LDAP سمت کلاینت

### مقدمه

**LDAP (Lightweight Directory Access Protocol)** سرویس دایرکتوری متمرکز برای کاربران و منابع است. تصور کنید ۱۰۰ سرور دارید و می‌خواهید کاربران مرکزی مدیریت شوند — LDAP این کار را می‌کند.

کلاینت از آن برای احراز هویت مرکزی استفاده می‌کند.

### مفاهیم اساسی

**مفاهیم کلیدی:**

- **DIT:** درخت دایرکتوری.
- **DN:** Distinguished Name (مسیر یکتا).
- **Base DN:** ریشهٔ جستجو.
- **Bind:** احراز هویت به دایرکتوری.
- **Schema:** تعریف ساختار.

**نمونهٔ DN:**

```
cn=alice,ou=people,dc=example,dc=com
```

**ساختار DN:**

```
dc=example,dc=com          ← دامنه
ou=people                  ← واحد سازمانی
uid=alice                  ← کاربر
```

### مثال‌های عملی

**نصب ابزار کلاینت:**

```bash
sudo apt install ldap-utils libnss-ldapd libpam-ldapd
sudo dnf install openldap-clients nss-pam-ldapd

# SSSD (مدرن)
sudo apt install sssd sssd-ldap
```

**پیکربندی کلاینت:**

```bash
# /etc/ldap/ldap.conf
BASE    dc=example,dc=com
URI     ldap://ldap.example.com
TLS_CACERT /etc/ssl/certs/ca.crt
```

**جستجو:**

```bash
# جستجوی ساده
ldapsearch -x -b "dc=example,dc=com" "(uid=alice)"

# با bind
ldapsearch -x -D "cn=admin,dc=example,dc=com" -W -b "dc=example,dc=com"

# محدود کردن
ldapsearch -x -b "dc=example,dc=com" "(objectClass=person)" cn mail

# LDAPS
ldapsearch -H ldaps://ldap.example.com -x -b "dc=example,dc=com"
```

**احراز هویت سیستم با LDAP (`/etc/nsswitch.conf`):**

```
passwd:  files ldap
group:   files ldap
shadow:  files ldap
```

**SSSD (مدرن):**

```ini
# /etc/sssd/sssd.conf
[sssd]
services = nss, pam
domains = example.com

[domain/example.com]
id_provider = ldap
auth_provider = ldap
ldap_uri = ldap://ldap.example.com
ldap_search_base = dc=example,dc=com
```

### مقایسه و الگوهای مختلف

**ابزارهای LDAP:**

| ابزار | کار |
|---|---|
| `ldapsearch` | جستجو |
| `ldapadd` | افزودن |
| `ldapmodify` | تغییر |
| `ldapdelete` | حذف |
| `ldapwhoami` | شناسایی |
| `ldapvi` | ویرایش تعاملی |

**nsswitch در برابر SSSD:**

| ویژگی | nsswitch | SSSD |
|---|---|---|
| نسخه | قدیمی | مدرن |
| کش | ندارد | دارد |
| offline | ندارد | دارد |
| توصیه | — | ✅ |

**LDAP در برابر فایل محلی:**

| ویژگی | فایل | LDAP |
|---|---|---|
| مقیاس | محدود | بزرگ |
| متمرکز | خیر | بله |
| مدیریت | تکی | مرکزی |

### اشکال شایع

**اشکال ۱: Base DN اشتباه.**
جستجو نتیجه نمی‌دهد. راه‌حل: Base DN درست.

**اشکال ۲: نبود nsswitch.**
سیستم کاربران LDAP را نمی‌بیند. راه‌حل: `nsswitch.conf`.

**اشکال ۳: TLS نادرست.**
اتصال رد می‌شود. راه‌حل: گواهی درست.

**اشکال ۴: bind اشتباه.**
احراز هویت شکست می‌خورد. راه‌حل: DN/رمز درست.

**اشکال ۵: نبود schema.**
افزودن شکست می‌خورد. راه‌حل: schema مناسب.

**اشکال ۶: کارایی پایین.**
جستجو کند. راه‌حل: SSSD کش.

### نکات مهم (Tips & Tricks)

- **`ldapsearch -x` برای جستجوی بدون احراز (anonymous).**
- **`ldapwhoami` اتصال را تست می‌کند.**
- **همیشه از TLS (ldaps://) استفاده کنید.**
- **`/etc/nsswitch.conf` را برای کاربران/گروه‌ها تنظیم کنید.**
- **`getent passwd` کاربران LDAP را نشان می‌دهد.**
- **`sssd` جایگزین مدرن برای یکپارچگی است.**
- **`getent hosts` هم شامل LDAP می‌شود.**
- **`ldapsearch -LLL` خروجی بدون کامنت.**
- **`-ZZ` برای StartTLS.**
- **`pam_ldap`/`pam_sss` برای احراز.**
- **`nslcd` سرویس nsswitch.**
- **`authselect` (RHEL) پیکربندی.**

### خلاصه فصل

LDAP دایرکتوری متمرکز کاربران است. DN مسیر یکتا و Base DN ریشهٔ جستجو است. `ldapsearch` جستجو و `ldapadd`/`ldapmodify` تغییر می‌دهند. `/etc/nsswitch.conf` سیستم را به LDAP وصل می‌کند. `ldapwhoami` اتصال را تست می‌کند و `getent` کاربران را نشان می‌دهد. SSSD جایگزین مدرن است.

---

## فصل ۸۹: پیکربندی سرور OpenLDAP

### مقدمه

**OpenLDAP** پیاده‌سازی متن‌باز سرور LDAP است. تصور کنید می‌خواهید یک دایرکتوری مرکزی برای کل سازمان بسازید — OpenLDAP این کار را می‌کند.

در این فصل راه‌اندازی و مدیریت آن را یاد می‌گیرید.

### مفاهیم اساسی

**اجزای OpenLDAP:**

- **slapd:** daemon سرور.
- **slapadd:** افزودن bulk.
- **slapcat:** بکاپ.
- **Schema:** تعریف ساختار.
- **cn=config:** پیکربندی مدرن.

**محل فایل‌ها:**

- `/etc/ldap/slapd.d/` — پیکربندی (cn=config).
- `/var/lib/ldap/` — داده.
- `/etc/ldap/schema/` — schemaها.

**Schemaهای رایج:**

| Schema | کاربرد |
|---|---|
| `core` | پایه |
| `cosine` | عمومی |
| `nis` | کاربران یونیکس |
| `inetOrgPerson` | افراد |

### مثال‌های عملی

**نصب و راه‌اندازی:**

```bash
sudo apt install slapd ldap-utils    # Debian
sudo dpkg-reconfigure slapd           # پیکربندی تعاملی

sudo dnf install openldap-servers openldap-clients   # RHEL
```

**ساخت ساختار پایه:**

```bash
# فایل base.ldif
cat > base.ldif <<'EOF'
dn: ou=people,dc=example,dc=com
objectClass: organizationalUnit
ou: people

dn: ou=groups,dc=example,dc=com
objectClass: organizationalUnit
ou: groups
EOF

ldapadd -x -D "cn=admin,dc=example,dc=com" -W -f base.ldif
```

**افزودن کاربر:**

```bash
cat > user.ldif <<'EOF'
dn: uid=alice,ou=people,dc=example,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: alice
cn: Alice Rezaei
sn: Rezaei
uidNumber: 10001
gidNumber: 10001
homeDirectory: /home/alice
loginShell: /bin/bash
mail: alice@example.com
userPassword: {SSHA}...
EOF

ldapadd -x -D "cn=admin,dc=example,dc=com" -W -f user.ldif
```

**تولید رمز:**

```bash
slappasswd -s mypassword
slappasswd -h '{SSHA}' -s mypassword
```

**مدیریت:**

```bash
# بکاپ
sudo slapcat -l backup.ldif

# بازیابی
sudo slapadd -l backup.ldif

# تغییر
ldapmodify -x -D "cn=admin,dc=example,dc=com" -W -f change.ldif

# حذف
ldapdelete -x -D "cn=admin,dc=example,dc=com" -W "uid=alice,ou=people,dc=example,dc=com"
```

### مقایسه و الگوهای مختلف

**ابزارهای مدیریت:**

| ابزار | کار |
|---|---|
| `slapadd` | افزودن از LDIF (آفلاین) |
| `slapcat` | بکاپ کامل |
| `slapindex` | ایندکس |
| `ldapadd` | افزودن (آنلاین) |
| `ldapmodify` | تغییر |
| `ldapdelete` | حذف |

**slapd.conf در برابر cn=config:**

| ویژگی | slapd.conf | cn=config |
|---|---|---|
| نسل | قدیمی | مدرن |
| فرمت | متنی | LDIF |
| تغییر | restart | آنلاین |
| توصیه | — | ✅ |

**پیکربندی در برابر داده:**

| مسیر | محتوا |
|---|---|
| `/etc/ldap/slapd.d/` | پیکربندی |
| `/var/lib/ldap/` | داده |

### اشکال شایع

**اشکال ۱: DN اشتباه.**
افزودن شکست می‌خورد. راه‌حل: DN درست.

**اشکال ۲: نبود parent.**
باید OU والد وجود داشته باشد. راه‌حل: ساخت والد.

**اشکال ۳: رمز بدون hash.**
`slappasswd` استفاده کنید. راه‌حل: hash.

**اشکال ۴: نبود ایندکس.**
جستجو کند می‌شود. راه‌حل: `olcDbIndex`.

**اشکال ۵: نبود schema.**
objectClass ناشناخته. راه‌حل: schema.

**اشکال ۶: مجوز فایل.**
slapd نمی‌تواند بخواند. راه‌حل: `chown openldap`.

### نکات مهم (Tips & Tricks)

- **`slapcat` بکاپ کامل می‌دهد.**
- **`slapadd` برای بازیابی سریع.**
- **`ldapsearch -b "" -s base` ریشه را نشان می‌دهد.**
- **برای امنیت، از TLS استفاده کنید.**
- **`olcAccess` دسترسی را کنترل می‌کند.**
- **`ldapvi` ویرایشگر تعاملی است.**
- **`slappasswd` رمز hash می‌کند.**
- **`olcDbIndex` ایندکس را تعریف می‌کند.**
- **`slapindex` ایندکس را بازسازی می‌کند.**
- **`phpLDAPadmin` رابط وب.**
- **`ldapdomaindump` تحلیل.**
- **`olcLogLevel` لاگ.**

### خلاصه فصل

OpenLDAP سرور LDAP است. `slapd` daemon و `slapd.d` پیکربندی است. ساختار با OUها ساخته می‌شود و کاربران با `ldapadd` اضافه می‌شوند. `slappasswd` رمز hash می‌کند و `slapcat` بکاپ می‌گیرد. همیشه TLS و ایندکس را در نظر بگیرید. `cn=config` روش مدرن پیکربندی است.

---

## منابع و مراجع

- man pages: `man dhcpd.conf`, `man pam.d`, `man limits.conf`, `man ldapsearch`, `man slapd.conf`
- ISC DHCP: https://www.isc.org/dhcp/
- PAM Documentation: http://www.linux-pam.org
- OpenLDAP Admin Guide: https://www.openldap.org/doc/
- SSSD: https://sssd.io

**بعدی:** [فایل ۲۶ — سرویس‌های ایمیل](26-Email-Services.md)
