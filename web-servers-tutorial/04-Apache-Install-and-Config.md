# بخش ۰۴: Apache — نصب و پیکربندی

**مدت یادگیری**: ۵ تا ۶ ساعت
**سطح سختی**: مبتدی
**پیش‌نیاز**: بخش ۰۳ (معماری Apache، MPM)

---

## فهرست محتوا

- [فصل ۱۳: نصب روی Ubuntu/Debian و RHEL](#فصل-۱۳)
- [فصل ۱۴: نصب روی ویندوز و تفاوت‌ها](#فصل-۱۴)
- [فصل ۱۵: ساختار پیکربندی و سینتکس](#فصل-۱۵)
- [فصل ۱۶: `VirtualHost` — میزبانی چند سایت](#فصل-۱۶)
- [فصل ۱۷: `Directory`، `.htaccess` و `AllowOverride`](#فصل-۱۷)

---

## فصل ۱۳: نصب روی Ubuntu/Debian و RHEL

### مقدمه

نصب Apache در دو خانواده‌ی لینوکس، دو دنیای متفاوت است: Debian/Ubuntu با `a2enmod`/`a2ensite` و ساختار `sites-available`، و RHEL/Rocky با `httpd.conf` و `conf.d/`. اگر این تفاوت را ندانی، دستورالعمل‌های اینترنت گاهی کار می‌کنند و گاهی نه.

### مفاهیم اساسی

**۱) بسته‌ها**

| توزیع | بسته | سرویس | کاربر |
|---|---|---|---|
| Ubuntu/Debian | `apache2` | `apache2` | `www-data` |
| RHEL/Rocky/Alma | `httpd` | `httpd` | `apache` |
| SUSE | `apache2` | `apache2` | `wwwrun` |

**۲) مسیرها**

| مورد | Debian/Ubuntu | RHEL |
|---|---|---|
| config اصلی | `/etc/apache2/apache2.conf` | `/etc/httpd/conf/httpd.conf` |
| سایت‌ها | `/etc/apache2/sites-available/` | `/etc/httpd/conf.d/` |
| ماژول‌ها | `/etc/apache2/mods-available/` | `/etc/httpd/conf.modules.d/` |
| لاگ | `/var/log/apache2/` | `/var/log/httpd/` |
| `DocumentRoot` | `/var/www/html` | `/var/www/html` |
| ابزار فعال‌سازی | `a2enmod`، `a2ensite` | دستی |

**۳) دستورهای مدیریت**

```bash
# Debian/Ubuntu
sudo systemctl start|stop|restart|reload apache2
sudo apachectl configtest
sudo a2enmod rewrite
sudo a2ensite lab
sudo a2dissite 000-default

# RHEL
sudo systemctl start|stop|restart|reload httpd
sudo apachectl configtest
sudo httpd -M
```

**۴) فایروال و SELinux**

```bash
# Ubuntu
sudo ufw allow 'Apache Full'      # 80 + 443

# RHEL
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload

# SELinux (مهم!)
sudo setsebool -P httpd_can_network_connect 1     # برای proxy
sudo semanage fcontext -a -t httpd_sys_content_t "/srv/lab(/.*)?"
sudo restorecon -Rv /srv/lab
```

**۵) `DocumentRoot` و مجوزها**

```bash
sudo mkdir -p /var/www/lab
sudo chown -R www-data:www-data /var/www/lab
sudo chmod -R 755 /var/www/lab
```

> فایل‌ها `644`، پوشه‌ها `755`. `777` هرگز.

**۶) تست اولیه**

```bash
curl -sSI http://localhost/ | head -3
# HTTP/1.1 200 OK
# Server: Apache/2.4.58 (Ubuntu)
```

**۷) حذف صفحه‌ی پیش‌فرض**

```bash
# Debian
sudo a2dissite 000-default
sudo rm /var/www/html/index.html

# RHEL
sudo sed -i 's/^#ServerName.*/ServerName lab.example.com/' /etc/httpd/conf/httpd.conf
```

### مثال‌های عملی

**Lab Scenario: نصب کامل روی Ubuntu 24.04**

```bash
# ۱) نصب
sudo apt update
sudo apt install -y apache2 apache2-utils

# ۲) فعال‌سازی ماژول‌های لازم
sudo a2enmod rewrite headers ssl proxy proxy_http proxy_fcgi expires deflate http2 remoteip

# ۳) تنظیم ServerName (جلوگیری از هشدار)
echo "ServerName lab.example.com" | sudo tee /etc/apache2/conf-available/servername.conf
sudo a2enconf servername

# ۴) تست و restart
sudo apachectl configtest
sudo systemctl restart apache2

# ۵) بررسی
sudo systemctl status apache2 --no-pager
sudo apachectl -V | head -5
sudo apachectl -M | wc -l
curl -sSI http://localhost/
```

**نصب روی Rocky Linux 9:**

```bash
# ۱) نصب
sudo dnf install -y httpd httpd-tools

# ۲) فعال‌سازی
sudo systemctl enable --now httpd

# ۳) فایروال
sudo firewall-cmd --permanent --add-service={http,https}
sudo firewall-cmd --reload

# ۴) SELinux برای پوشه‌ی سفارشی
sudo semanage fcontext -a -t httpd_sys_content_t "/srv/lab(/.*)?"
sudo restorecon -Rv /srv/lab

# ۵) تست
curl -sSI http://localhost/
```

**پیکربندی پایه برای تولید:**

```apache
# /etc/apache2/conf-available/hardening.conf
ServerTokens Prod
ServerSignature Off
TraceEnable Off
Timeout 30
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5

<IfModule mod_headers.c>
    Header always unset X-Powered-By
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
</IfModule>

<Directory />
    Options -Indexes -FollowSymLinks
    AllowOverride None
    Require all denied
</Directory>
```

```bash
sudo a2enconf hardening
sudo apachectl configtest && sudo systemctl reload apache2
```

**لاگ چرخشی:**

```bash
# Debian: از قبل تنظیم است
cat /etc/logrotate.d/apache2

# RHEL
sudo tee /etc/logrotate.d/httpd >/dev/null <<'EOF'
/var/log/httpd/*log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    sharedscripts
    postrotate
        /bin/systemctl reload httpd.service > /dev/null 2>/dev/null || true
    endscript
}
EOF
```

### مقایسه و الگوهای مختلف

**Debian در برابر RHEL**

| | Debian/Ubuntu | RHEL |
|---|---|---|
| فعال‌سازی ماژول | `a2enmod` | `LoadModule` دستی |
| فعال‌سازی سایت | `a2ensite` | فایل در `conf.d/` |
| کاربر | `www-data` | `apache` |
| SELinux | ❌ (AppArmor) | ✅ |
| نسخه‌ی پیش‌فرض | جدیدتر | پایدارتر |

**`apt` در برابر `dnf` در برابر کامپایل از سورس**

| | `apt`/`dnf` | سورس |
|---|---|---|
| سرعت | ✅ | ❌ |
| به‌روزرسانی امنیتی | ✅ | ❌ دستی |
| ماژول سفارشی | ⚠️ | ✅ |
| توصیه | ✅ | فقط نیاز خاص |

**دیدگاه «کامپایل از سورس بهتر است»**: فقط اگر ماژول غیراستاندارد لازم داری (مثل `mod_pagespeed` نسخه‌ی خاص). وگرنه، بسته‌ی توزیع با به‌روزرسانی امنیتی، انتخاب درست است.

### اشکال شایع

**۱) `Could not reliably determine the server's fully qualified domain name`**
`ServerName` تنظیم نشده. یک `conf` اضافه کن.

**۲) `403 Forbidden` بعد از تغییر `DocumentRoot`**
مجوز فایل یا SELinux. `ls -lZ /srv/lab`.

**۳) `a2enmod: command not found`**
روی RHEL. `LoadModule` در `conf.modules.d/`.

**۴) پورت ۸۰ اشغال**
`ss -ltnp | grep :80`. ممکن است Nginx هم نصب باشد.

**۵) `AH00558` هشدار در لاگ**
همان `ServerName`. بی‌خطر ولی بهتر است رفع شود.

**۶) SELinux و proxy**
`httpd_can_network_connect` خاموش → `502`. `setsebool`.

**۷) `AppArmor` و پوشه‌ی سفارشی**
`/etc/apparmor.d/local/usr.sbin.apache2` را ویرایش کن.

**۸) `mod_rewrite` فعال نیست**
`Invalid command 'RewriteEngine'`. `a2enmod rewrite`.

**۹) `systemctl restart` در ساعات اوج**
اتصال‌ها قطع می‌شوند. `reload` (graceful) استفاده کن.

**۱۰) فراموش کردن `systemctl enable`**
بعد از reboot، سایت پایین است.

### نکات مهم (Tips & Tricks)

- **`apachectl -S`** همه‌ی vhostها.
- **`apachectl -M`** ماژول‌ها.
- **`apachectl configtest`** قبل از reload.
- **`a2enconf`** برای تنظیمات مشترک.
- **`ServerTokens Prod`** برای پنهان کردن جزئیات.
- **`systemctl reload`** به‌جای `restart` در تولید.
- **`logrotate`** برای لاگ.

### خلاصه فصل

- Debian با `a2enmod`/`a2ensite`، RHEL با `conf.d/`.
- کاربر `www-data` در Debian، `apache` در RHEL.
- SELinux در RHEL باید برای پوشه و proxy تنظیم شود.
- `DocumentRoot` نیاز به مجوز `755`/`644`.
- `reload` بدون قطع سرویس.

**سؤال مروری:** چرا بعد از تغییر `DocumentRoot` به `/srv/lab` روی RHEL، `403` می‌گیری و چطور حل می‌شود؟

---

## فصل ۱۴: نصب روی ویندوز و تفاوت‌ها

### مقدمه

Apache روی ویندوز کار می‌کند، ولی «کار کردن» با «تولید» تفاوت دارد. ویندوز برای Apache، محیط درجه‌دوم است: بدون `epoll`، بدون `fork` واقعی، و با مسیرهای متفاوت. اگر روی ویندوز Apache می‌خواهی، این فصل واقعیت‌ها را می‌گوید.

### مفاهیم اساسی

**۱) دو گزینه**

| گزینه | توضیح |
|---|---|
| بیلد رسمی Apache Lounge | `apachelounge.com` — باینری‌های Win64 |
| `winget`/Chocolatey | نصب سریع |

**۲) تفاوت‌های کلیدی**

| مورد | لینوکس | ویندوز |
|---|---|---|
| MPM | `event`/`worker`/`prefork` | `winnt` (فقط) |
| مسیرها | `/etc/apache2` | `C:\Apache24\conf` |
| سرویس | `systemctl` | `httpd.exe -k install` |
| کاربر | `www-data` | `LocalSystem` یا کاربر سرویس |
| مجوز | POSIX | ACL |
| `epoll` | ✅ | ❌ (IOCP در `winnt` MPM) |
| symlink | ✅ | ⚠️ (نیاز به privilege) |

**۳) `winnt` MPM**

```apache
<IfModule mpm_winnt_module>
    ThreadsPerChild      150
    MaxConnectionsPerChild 0
</IfModule>
```

یک پروسه‌ی والد + یک فرزند با تردها. `MaxConnectionsPerChild 0` یعنی هرگز بازیافت نشود (در ویندوز توصیه می‌شود، چون `fork` گران است).

**۴) نصب به‌عنوان سرویس**

```cmd
cd C:\Apache24\bin
httpd.exe -k install -n "Apache2.4"
httpd.exe -k start
httpd.exe -k restart
httpd.exe -k uninstall
```

**۵) فایروال ویندوز**

```powershell
New-NetFirewallRule -DisplayName "Apache HTTP" -Direction Inbound -Protocol TCP -LocalPort 80 -Action Allow
New-NetFirewallRule -DisplayName "Apache HTTPS" -Direction Inbound -Protocol TCP -LocalPort 443 -Action Allow
```

**۶) مسیرها با `/` یا `\`**

Apache روی ویندوز هر دو را می‌پذیرد، ولی در config از `/` استفاده کن:

```apache
DocumentRoot "C:/web/lab"
<Directory "C:/web/lab">
```

**۷) `httpd.conf` روی ویندوز**

```apache
ServerRoot "C:/Apache24"
Listen 80
ServerName lab.example.com:80

LoadModule rewrite_module modules/mod_rewrite.so
# ...

DocumentRoot "C:/web/lab"
<Directory "C:/web/lab">
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

**۸) محدودیت‌های واقعی**

- کارایی فایل استاتیک کمتر از لینوکس.
- بعضی ماژول‌ها (`mod_fcgid`، `mod_wsgi`) پشتیبانی محدود.
- `mod_php` روی ویندوز، قدیمی و ناپایدار.
- برای PHP روی ویندوز، `php-fpm` (از PHP 8 با `php-cgi` روی ویندوز محدود) یا IIS توصیه می‌شود.

### مثال‌های عملی

**Lab Scenario: نصب Apache روی Windows Server 2022**

```powershell
# ۱) دانلود Apache Lounge (Win64)
$url = "https://www.apachelounge.com/download/VS17/binaries/httpd-2.4.62-250425-win64-VS17.zip"
Invoke-WebRequest -Uri $url -OutFile "$env:TEMP\apache.zip"

# ۲) استخراج به C:\
Expand-Archive -Path "$env:TEMP\apache.zip" -DestinationPath "C:\" -Force
# نتیجه: C:\Apache24

# ۳) نصب سرویس
cd C:\Apache24\bin
.\httpd.exe -k install -n "Apache2.4"

# ۴) تست config
.\httpd.exe -t

# ۵) شروع
Start-Service Apache2.4
Get-Service Apache2.4

# ۶) فایروال
New-NetFirewallRule -DisplayName "Apache HTTP" -Direction Inbound -Protocol TCP -LocalPort 80 -Action Allow

# ۷) تست
Invoke-WebRequest http://localhost/ -UseBasicParsing | Select-Object StatusCode
```

**پیکربندی سایت:**

```apache
# C:/Apache24/conf/extra/httpd-vhosts.conf
<VirtualHost *:80>
    ServerName lab.example.com
    DocumentRoot "C:/web/lab"

    <Directory "C:/web/lab">
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog "logs/lab-error.log"
    CustomLog "logs/lab-access.log" combined
</VirtualHost>
```

```apache
# در httpd.conf، این خط را از کامنت خارج کن
Include conf/extra/httpd-vhosts.conf
```

```powershell
# hosts برای تست محلی
Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value "127.0.0.1  lab.example.com"
.\httpd.exe -t
Restart-Service Apache2.4
Invoke-WebRequest http://lab.example.com/ -UseBasicParsing
```

**TLS روی ویندوز:**

```apache
LoadModule ssl_module modules/mod_ssl.so
Listen 443

<VirtualHost *:443>
    ServerName lab.example.com
    SSLEngine on
    SSLCertificateFile "C:/certs/lab.crt"
    SSLCertificateKeyFile "C:/certs/lab.key"
    SSLCertificateChainFile "C:/certs/chain.crt"
    DocumentRoot "C:/web/lab"
</VirtualHost>
```

**لاگ و چرخش:**

```powershell
# Apache روی ویندوز، rotatelogs دارد
CustomLog "|bin/rotatelogs.exe logs/access-%Y%m%d.log 86400" combined
ErrorLog  "|bin/rotatelogs.exe logs/error-%Y%m%d.log 86400"
```

### مقایسه و الگوهای مختلف

**Apache روی ویندوز در برابر IIS**

| معیار | Apache (Win) | IIS |
|---|---|---|
| کارایی استاتیک | ⚠️ | ✅ (`HTTP.sys`) |
| `.htaccess` | ✅ | ⚠️ (`web.config`) |
| PHP | ⚠️ | ⚠️ |
| ASP.NET | ❌ | ✅✅ |
| مدیریت | ⚠️ (فایل) | ✅ (GUI) |
| توصیه | انتقال از لینوکس | ✅ بومی |

**`winnt` MPM در برابر `event`**

| | `winnt` | `event` |
|---|---|---|
| سیستم‌عامل | ویندوز | لینوکس |
| مدل | پروسه + ترد | پروسه + ترد + epoll |
| کارایی keep-alive | ⚠️ | ✅ |

**دیدگاه «Apache روی ویندوز برای تولید»**: اگر مجبوری (انتقال از لینوکس، ماژول خاص)، کار می‌کند. ولی برای پروژه‌ی جدید روی ویندوز، IIS انتخاب طبیعی‌تر است.

### اشکال شایع

**۱) `httpd.exe -k install` بدون Administrator**
`Access is denied`. PowerShell را as Administrator باز کن.

**۲) پورت ۸۰ اشغال (`PID 4`)**
`HTTP.sys` (IIS یا `World Wide Web Publishing Service`) پورت را گرفته. `netstat -ano | findstr :80` → اگر PID 4 بود، سرویس را متوقف کن یا پورت Apache را عوض کن.

**۳) `DocumentRoot` با `\`**
بعضی نسخه‌ها مشکل دارند. `/` استفاده کن.

**۴) `403` با وجود مجوز**
ACL ویندوز: `icacls C:\web\lab /grant "IIS_IUSRS:(OI)(CI)R"` یا کاربر سرویس.

**۵) `mod_rewrite` لود نشده**
`LoadModule rewrite_module modules/mod_rewrite.so` بدون `#`.

**۶) سرویس شروع نمی‌شود**
`httpd.exe -t` و `logs/error.log`. معمولاً config یا پورت.

**۷) `SSLCertificateKeyFile` با مسیر اشتباه**
`\` و فاصله در مسیر. quote بگذار.

**۸) لاگ پر می‌شود**
بدون `rotatelogs`. تنظیم کن.

### نکات مهم (Tips & Tricks)

- **`httpd.exe -t`** تست config.
- **`httpd.exe -k restart`** برای reload.
- **`netstat -ano | findstr :80`** برای تعارض پورت.
- **`icacls`** برای مجوز.
- **`rotatelogs.exe`** برای چرخش لاگ.
- **`/` در مسیرها**.
- **IIS** برای پروژه‌ی جدید ویندوزی.

### خلاصه فصل

- Apache روی ویندوز فقط `winnt` MPM دارد.
- نصب به‌عنوان سرویس با `httpd.exe -k install`.
- `HTTP.sys` می‌تواند پورت ۸۰ را بگیرد (PID 4).
- ACL ویندوز جای مجوز POSIX را می‌گیرد.
- برای پروژه‌ی جدید ویندوزی، IIS انتخاب طبیعی‌تر است.

**سؤال مروری:** اگر `netstat` نشان دهد PID 4 روی پورت ۸۰ گوش می‌دهد، چه چیزی باعثش شده و چطور حل می‌کنی؟

---

## فصل ۱۵: ساختار پیکربندی و سینتکس

### مقدمه

Apache بیش از ۲۰۰ دستور پیکربندی دارد و در ده‌ها فایل پخش شده‌اند. اگر ساختار و ترتیب را نفهمی، «تنظیم کردم ولی اثر نکرد» تبدیل به تجربه‌ی روزمره می‌شود.

### مفاهیم اساسی

**۱) ساختار درخت**

```apache
# سطح سرور (global)
ServerRoot "/etc/apache2"
Listen 80
User www-data
Group www-data
ServerName lab.example.com
Timeout 30
KeepAlive On

# ماژول‌ها
LoadModule rewrite_module modules/mod_rewrite.so

# بخش‌های شرطی
<IfModule mod_ssl.c>
    Listen 443
</IfModule>

# پیکربندی per-directory
<Directory /var/www>
    Options -Indexes
    AllowOverride None
    Require all granted
</Directory>

# vhost
<VirtualHost *:80>
    ServerName lab.example.com
    DocumentRoot /var/www/lab
</VirtualHost>
```

**۲) دستورها و دامنه‌ی اثر**

| دامنه | مثال | اثر |
|---|---|---|
| سراسری | `Timeout 30` | همه‌جا |
| `<Directory>` | مسیر فیزیکی | فایل‌ها |
| `<Location>` | مسیر URL | درخواست‌ها |
| `<Files>` | نام فایل | فایل‌های مشخص |
| `<VirtualHost>` | دامنه/پورت | یک سایت |
| `.htaccess` | پوشه | زیرشاخه |

**۳) ترتیب merge**

```
سرور → VirtualHost → Directory → .htaccess → Location → Files
```

هر سطح، سطح قبلی را override می‌کند (بسته به دستور).

**۴) `Options` — چه چیزی مجاز است**

| گزینه | معنا | توصیه |
|---|---|---|
| `Indexes` | لیست پوشه | ❌ خاموش |
| `FollowSymLinks` | دنبال کردن symlink | ⚠️ |
| `SymLinksIfOwnerMatch` | symlink با مالک یکسان | ✅ |
| `ExecCGI` | اجرای CGI | ❌ |
| `Includes` | SSI | ❌ |
| `MultiViews` | محتوای مذاکره‌ای | ❌ (خطر) |
| `-Indexes +FollowSymLinks` | خاموش/روشن | ✅ |

**۵) `Require` — کنترل دسترسی (2.4)**

```apache
Require all granted
Require all denied
Require ip 10.0.0.0/8
Require host example.com
Require valid-user

# ترکیب
<RequireAll>
    Require ip 10.0.0.0/8
    Require not ip 10.0.0.99
</RequireAll>
```

**۶) `IfModule` و `IfDefine`**

```apache
<IfModule mod_headers.c>
    Header set X-Test "1"
</IfModule>

<IfDefine PRODUCTION>
    LogLevel warn
</IfDefine>
```

`IfDefine` با `-D PRODUCTION` در startup.

**۷) `Include` و `IncludeOptional`**

```apache
Include /etc/apache2/conf-enabled/*.conf
IncludeOptional /etc/apache2/sites-enabled/*.conf
```

`IncludeOptional` اگر فایل نبود، خطا نمی‌دهد.

**۸) متغیرها (`Define`)**

```apache
Define DOCROOT /var/www/lab
DocumentRoot "${DOCROOT}"
```

**۹) `LogLevel`**

```apache
LogLevel warn
# یا per-module
LogLevel info ssl:warn rewrite:trace3
```

| سطح | کاربرد |
|---|---|
| `emerg`/`alert`/`crit` | بحرانی |
| `error` | خطا |
| `warn` | ✅ تولید |
| `notice` | اطلاعات |
| `info` | دیباگ |
| `debug`/`trace1..8` | دیباگ عمیق |

**۱۰) `apachectl -S` و `-t -D DUMP_VHOSTS`**

```bash
sudo apachectl -S
# VirtualHost configuration:
# *:80   is a NameVirtualHost
#          default server lab.example.com (/etc/apache2/sites-enabled/lab.conf:1)
#          port 80 namevhost lab.example.com (/etc/apache2/sites-enabled/lab.conf:1)
#                  alias www.lab.example.com
```

### مثال‌های عملی

**Lab Scenario: پیکربندی تمیز و ماژولار**

ساختار پیشنهادی:

```
/etc/apache2/
├── apache2.conf
├── ports.conf
├── conf-available/
│   ├── security.conf
│   ├── compression.conf
│   └── charset.conf
├── conf-enabled/        (symlink)
├── mods-available/
├── mods-enabled/        (symlink)
├── sites-available/
│   ├── lab.conf
│   └── api.conf
└── sites-enabled/       (symlink)
```

```apache
# /etc/apache2/conf-available/security.conf
ServerTokens Prod
ServerSignature Off
TraceEnable Off

<Directory />
    Options -Indexes -FollowSymLinks
    AllowOverride None
    Require all denied
</Directory>

<Directory /var/www>
    Options -Indexes +FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>

<IfModule mod_headers.c>
    Header always unset X-Powered-By
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set Permissions-Policy "geolocation=(), microphone=(), camera=()"
</IfModule>
```

```apache
# /etc/apache2/conf-available/compression.conf
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html text/plain text/css text/xml
    AddOutputFilterByType DEFLATE application/javascript application/json
    AddOutputFilterByType DEFLATE application/xml image/svg+xml
    DeflateCompressionLevel 6
</IfModule>

<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType image/jpeg  "access plus 30 days"
    ExpiresByType image/png   "access plus 30 days"
    ExpiresByType image/svg+xml "access plus 30 days"
    ExpiresByType text/css    "access plus 7 days"
    ExpiresByType application/javascript "access plus 7 days"
    ExpiresByType text/html   "access plus 0 seconds"
</IfModule>
```

```bash
sudo a2enconf security compression
sudo apachectl configtest && sudo systemctl reload apache2
```

**دیباگ ترتیب config:**

```bash
# همه‌ی vhostها
sudo apachectl -S

# تست با define
sudo apachectl -t -D DUMP_RUN_CFG

# دیدن مقدار مؤثر یک دستور
sudo apachectl -t -D DUMP_MODULES | grep -i rewrite

# لاگ با trace برای rewrite
# در config:
# LogLevel alert rewrite:trace3
sudo tail -f /var/log/apache2/error.log | grep rewrite
```

**تست `Options -Indexes`:**

```bash
mkdir -p /var/www/lab/empty
curl -sSI http://lab.example.com/empty/
# HTTP/1.1 403 Forbidden
```

**تست `Require ip`:**

```apache
<Location /admin>
    Require ip 192.168.56.0/24
</Location>
```

```bash
curl -sSI http://lab.example.com/admin/
# از داخل شبکه: 200
# از بیرون: 403
```

### مقایسه و الگوهای مختلف

**`Directory` در برابر `Location`**

| | `<Directory>` | `<Location>` |
|---|---|---|
| ورودی | مسیر فایل | URL |
| بعد از rewrite | ❌ | ✅ |
| کاربرد | مجوز فایل | کنترل URL |
| خطر | ⚠️ | ⚠️ (regex پیچیده) |

**`AllowOverride None` در برابر `All`**

| | `None` | `All` |
|---|---|---|
| سرعت | ✅ | ❌ |
| انعطاف | ❌ | ✅ |
| امنیت | ✅ | ⚠️ |
| توصیه | ✅ تولید | هاست اشتراکی |

**`Require` 2.2 در برابر 2.4**

| 2.2 | 2.4 |
|---|---|
| `Order allow,deny` | `Require all granted` |
| `Allow from all` | `Require ip ...` |
| `Deny from all` | `Require all denied` |

**دیدگاه «`Options All` راحت است»**: `All` شامل `Indexes`، `Includes`، `ExecCGI` و `MultiViews` است — هر کدام یک سطح حمله. صریح بنویس.

### اشکال شایع

**۱) `Invalid command 'Header'`**
`mod_headers` فعال نیست. `a2enmod headers`.

**۲) `Options Indexes` روشن**
لیست فایل‌ها. `-Indexes`.

**۳) `MultiViews` روشن**
مسیرهای عجیب و دور زدن قواعد. خاموش.

**۴) `FollowSymLinks` بدون `SymLinksIfOwnerMatch`**
خطر symlink. `SymLinksIfOwnerMatch`.

**۵) `Require all granted` در `<Directory />`**
همه‌چیز باز. `Require all denied` در ریشه و استثنا در پوشه‌ی سایت.

**۶) `LogLevel debug` در تولید**
دیسک پر و کندی. `warn`.

**۷) `Include` با glob خالی**
خطا. `IncludeOptional`.

**۸) ترتیب `Include`**
فایل `security.conf` بعد از `sites-enabled` لود شود، می‌تواند override کند. ترتیب را در `apache2.conf` ببین.

**۹) `IfModule` که ماژول را نمی‌بیند**
نام ماژول اشتباه (`mod_ssl.c` در برابر `ssl_module`). `apachectl -M`.

**۱۰) `Define` قبل از استفاده**
`${VAR}` قبل از `Define`، خطا.

### نکات مهم (Tips & Tricks)

- **`apachectl -S`** همه‌ی vhostها.
- **`apachectl -t -D DUMP_RUN_CFG`** تنظیمات مؤثر.
- **`LogLevel alert rewrite:trace3`** برای دیباگ rewrite.
- **`IncludeOptional`** به‌جای `Include` برای پوشه‌های خالی.
- **`Define`** برای مسیرهای تکراری.
- **`conf-available`** برای تنظیمات مشترک.
- **`ServerTokens Prod`**.

### خلاصه فصل

- Apache سه سطح دارد: سرور، vhost، directory.
- `<Directory>` روی فایل، `<Location>` روی URL.
- `Options` را صریح بنویس، `All` نه.
- `Require` در 2.4 جای `Order`/`Allow` را گرفت.
- `IncludeOptional` و `Define` برای تمیزی.
- `apachectl -S` و `-t -D DUMP_*` ابزار دیباگ.

**سؤال مروری:** تفاوت `<Directory>` و `<Location>` چیست و کدام بعد از `mod_rewrite` اعمال می‌شود؟

---

## فصل ۱۶: `VirtualHost` — میزبانی چند سایت

### مقدمه

یک سرور، یک IP، ده سایت. `VirtualHost` این را ممکن می‌کند. سه روش وجود دارد و انتخاب اشتباه، باعث می‌شود سایت اول، همه‌ی درخواست‌های ناشناس را بگیرد.

### مفاهیم اساسی

**۱) سه نوع**

| نوع | مبنای تشخیص | مثال |
|---|---|---|
| **Name-based** | هدر `Host` | `lab.example.com` و `api.example.com` روی یک IP |
| **IP-based** | IP مقصد | دو IP روی یک سرور |
| **Port-based** | پورت | `:80` و `:8080` |

**Name-based** رایج‌ترین است (چون IPv4 کم است).

**۲) چطور کار می‌کند**

```
کلاینت: GET / HTTP/1.1
        Host: api.example.com
              │
              ▼
       Apache هدر Host را می‌خواند
              │
       ┌──────┴──────┐
       ▼             ▼
  lab.example.com  api.example.com
  DocumentRoot A   DocumentRoot B
```

**۳) `ServerName` و `ServerAlias`**

```apache
<VirtualHost *:80>
    ServerName lab.example.com
    ServerAlias www.lab.example.com lab.local
    DocumentRoot /var/www/lab
</VirtualHost>
```

**۴) default server**

اولین `VirtualHost` برای هر پورت، **default** است. اگر `Host` با هیچ‌کدام مطابقت نکند، default پاسخ می‌دهد.

```apache
# default برای رد کردن یا صفحه‌ی خاص
<VirtualHost *:80>
    ServerName default.invalid
    DocumentRoot /var/www/default
    <Location />
        Require all denied
    </Location>
</VirtualHost>
```

**۵) `_default_` و `*`**

```apache
<VirtualHost _default_:80>     # همه‌ی IPها، پورت ۸۰
<VirtualHost *:80>             # همان
<VirtualHost 192.168.56.10:80> # IP مشخص
```

**۶) SSL و SNI**

```apache
<VirtualHost *:443>
    ServerName lab.example.com
    SSLEngine on
    SSLCertificateFile    /etc/ssl/lab/fullchain.pem
    SSLCertificateKeyFile /etc/ssl/lab/privkey.pem
</VirtualHost>

<VirtualHost *:443>
    ServerName api.example.com
    SSLEngine on
    SSLCertificateFile    /etc/ssl/api/fullchain.pem
    SSLCertificateKeyFile /etc/ssl/api/privkey.pem
</VirtualHost>
```

Apache با `SNI` گواهی درست را انتخاب می‌کند (نیاز به `SSLStrictSNIVHostCheck off` برای کلاینت‌های قدیمی).

**۷) `NameVirtualHost` (قدیمی)**

در 2.2 لازم بود؛ در 2.4 حذف شده. اگر در config قدیمی دیدی، حذفش کن.

**۸) ترتیب و اولویت**

- اولین vhost برای هر پورت = default.
- تطابق دقیق `ServerName` > `ServerAlias` > default.
- `_default_` صریح، اولویت دارد.

**۹) `ServerAdmin` و لاگ per-vhost**

```apache
<VirtualHost *:80>
    ServerName lab.example.com
    ServerAdmin admin@example.com
    ErrorLog  ${APACHE_LOG_DIR}/lab-error.log
    CustomLog ${APACHE_LOG_DIR}/lab-access.log combined
</VirtualHost>
```

**۱۰) `Redirect` و `Rewrite` در vhost**

```apache
<VirtualHost *:80>
    ServerName lab.example.com
    Redirect permanent / https://lab.example.com/
</VirtualHost>
```

### مثال‌های عملی

**Lab Scenario: سه سایت روی یک سرور**

```apache
# /etc/apache2/sites-available/000-default.conf
<VirtualHost *:80>
    ServerName default.invalid
    DocumentRoot /var/www/default
    <Location />
        Require all denied
    </Location>
    ErrorLog ${APACHE_LOG_DIR}/default-error.log
    CustomLog ${APACHE_LOG_DIR}/default-access.log combined
</VirtualHost>
```

```apache
# /etc/apache2/sites-available/lab.conf
<VirtualHost *:80>
    ServerName lab.example.com
    ServerAlias www.lab.example.com
    ServerAdmin admin@example.com
    DocumentRoot /var/www/lab

    <Directory /var/www/lab>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    ErrorLog  ${APACHE_LOG_DIR}/lab-error.log
    CustomLog ${APACHE_LOG_DIR}/lab-access.log combined
</VirtualHost>
```

```apache
# /etc/apache2/sites-available/api.conf
<VirtualHost *:80>
    ServerName api.lab.example.com
    DocumentRoot /var/www/api

    <Directory /var/www/api>
        Options -Indexes
        AllowOverride None
        Require all granted
    </Directory>

    ProxyPreserveHost On
    ProxyPass        /v1/ http://127.0.0.1:3000/
    ProxyPassReverse /v1/ http://127.0.0.1:3000/

    ErrorLog  ${APACHE_LOG_DIR}/api-error.log
    CustomLog ${APACHE_LOG_DIR}/api-access.log combined
</VirtualHost>
```

```bash
# ساخت پوشه‌ها
sudo mkdir -p /var/www/{default,lab,api}
echo '<h1>Lab</h1>'  | sudo tee /var/www/lab/index.html
echo '<h1>API</h1>'  | sudo tee /var/www/api/index.html
echo 'denied'        | sudo tee /var/www/default/index.html
sudo chown -R www-data:www-data /var/www

# فعال‌سازی
sudo a2ensite 000-default lab api
sudo a2enmod proxy proxy_http
sudo apachectl configtest
sudo systemctl reload apache2

# بررسی
sudo apachectl -S

# تست
for h in lab.example.com api.lab.example.com unknown.example.com; do
  printf '%-25s → ' "$h"
  curl -s -o /dev/null -w '%{http_code}\n' -H "Host: $h" http://127.0.0.1/
done
```

خروجی:

```
lab.example.com           → 200
api.lab.example.com       → 200
unknown.example.com       → 403      (default deny)
```

**افزودن HTTPS:**

```apache
# /etc/apache2/sites-available/lab-ssl.conf
<VirtualHost *:443>
    ServerName lab.example.com
    ServerAlias www.lab.example.com
    DocumentRoot /var/www/lab

    SSLEngine on
    SSLCertificateFile    /etc/letsencrypt/live/lab.example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/lab.example.com/privkey.pem
    SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1

    <Directory /var/www/lab>
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
</VirtualHost>
```

```apache
# ریدایرکت در vhost پورت ۸۰
<VirtualHost *:80>
    ServerName lab.example.com
    ServerAlias www.lab.example.com
    Redirect permanent / https://lab.example.com/
</VirtualHost>
```

**IP-based (برای مقایسه):**

```apache
<VirtualHost 192.168.56.10:80>
    ServerName lab.example.com
    DocumentRoot /var/www/lab
</VirtualHost>

<VirtualHost 192.168.56.11:80>
    ServerName api.example.com
    DocumentRoot /var/www/api
</VirtualHost>
```

**تست SNI:**

```bash
# گواهی هر دامنه
echo | openssl s_client -connect 192.168.56.10:443 -servername lab.example.com 2>/dev/null \
  | openssl x509 -noout -subject

echo | openssl s_client -connect 192.168.56.10:443 -servername api.lab.example.com 2>/dev/null \
  | openssl x509 -noout -subject
```

### مقایسه و الگوهای مختلف

**Name-based در برابر IP-based در برابر Port-based**

| | Name-based | IP-based | Port-based |
|---|---|---|---|
| نیاز به چند IP | ❌ | ✅ | ❌ |
| SSL (با SNI) | ✅ | ✅ | ✅ |
| کلاینت قدیمی بدون SNI | ⚠️ | ✅ | ✅ |
| توصیه | ✅ | ⚠️ | ⚠️ |

**`Redirect` در برابر `RewriteRule`**

| | `Redirect` | `RewriteRule` |
|---|---|---|
| سادگی | ✅ | ⚠️ |
| regex | ❌ | ✅ |
| query string | ⚠️ | ✅ |
| مناسب | کل دامنه/مسیر | قواعد پیچیده |

**`ServerAlias` در برابر `ServerName`**

| | `ServerName` | `ServerAlias` |
|---|---|---|
| تعداد | یک | چند |
| نقش | نام اصلی | نام‌های دیگر |
| redirect canonical | ✅ | — |

**دیدگاه «همه‌چیز در یک vhost»**: با یک سایت کوچک کار می‌کند. با چند سایت، جداسازی لاگ و `DocumentRoot` ضروری است.

### اشکال شایع

**۱) سایت اول، همه‌چیز را می‌گیرد**
default vhost تنظیم نشده. اولین vhost = default. یک `000-default` با `deny` بگذار.

**۲) `ServerName` بدون `ServerAlias`**
`www.` کار نمی‌کند. اضافه کن.

**۳) `NameVirtualHost` در 2.4**
منسوخ و خطا. حذف.

**۴) `a2ensite` فراموش‌شده**
فایل در `sites-available` هست ولی symlink ندارد. `apachectl -S` نشان می‌دهد.

**۵) SSL و `SNI` خاموش**
`SSLStrictSNIVHostCheck` یا کلاینت قدیمی. تست با `openssl s_client -servername`.

**۶) `DocumentRoot` مشترک**
دو سایت، یک پوشه. لاگ و مجوز قاطی می‌شود.

**۷) `ProxyPass` در vhost بدون `mod_proxy`**
`Invalid command`. `a2enmod proxy proxy_http`.

**۸) redirect loop**
`Redirect permanent / https://...` در vhost ۴۴۳ هم هست. فقط در ۸۰.

**۹) `ServerAlias` با wildcard اشتباه**
`*.example.com` مجاز است، ولی با دامنه‌های دیگر تعارض می‌کند.

**۱۰) لاگ مشترک**
`lab-error.log` و `api-error.log` جدا بگذار.

### نکات مهم (Tips & Tricks)

- **`apachectl -S`** ترتیب و default را نشان می‌دهد.
- **`000-default` با deny** برای جلوگیری از افشا.
- **`ServerAlias`** برای `www`.
- **`a2ensite`/`a2dissite`** برای مدیریت.
- **لاگ per-vhost**.
- **`Redirect`** برای HTTPS.
- **`openssl s_client -servername`** برای تست SNI.

### خلاصه فصل

- `VirtualHost` با `Host` (name-based)، IP یا پورت کار می‌کند.
- اولین vhost هر پورت، default است.
- `ServerAlias` نام‌های دیگر را پوشش می‌دهد.
- SNI گواهی درست را انتخاب می‌کند.
- `apachectl -S` ابزار اصلی دیباگ.

**سؤال مروری:** اگر `Host` با هیچ `ServerName`/`ServerAlias` مطابقت نکند، کدام vhost پاسخ می‌دهد و چطور از افشای اطلاعات جلوگیری می‌کنی؟

---

## فصل ۱۷: `Directory`، `.htaccess` و `AllowOverride`

### مقدمه

`.htaccess` قدرتمندترین و خطرناک‌ترین ویژگی Apache است. قدرتمند چون بدون دسترسی root، پیکربندی را عوض می‌کند. خطرناک چون در **هر درخواست** خوانده می‌شود و می‌تواند تنظیمات امنیتی سراسری را override کند.

### مفاهیم اساسی

**۱) `.htaccess` چه می‌تواند بکند**

| کار | مثال |
|---|---|
| rewrite | `RewriteRule` |
| احراز هویت | `AuthType Basic` |
| کنترل دسترسی | `Require ip` |
| MIME | `AddType` |
| خطا | `ErrorDocument` |
| فشرده‌سازی | `AddOutputFilterByType` |
| هدر | `Header set` |
| PHP | `php_value` |

**۲) `AllowOverride` — چه چیزی مجاز است**

```apache
<Directory /var/www/lab>
    AllowOverride None       # هیچ
    # AllowOverride All      # همه
    # AllowOverride FileInfo Indexes Limit
</Directory>
```

| گروه | شامل |
|---|---|
| `AuthConfig` | `AuthType`، `Require` |
| `FileInfo` | `RewriteRule`، `AddType`، `ErrorDocument` |
| `Indexes` | `DirectoryIndex`، `Options Indexes` |
| `Limit` | `Require`، `Allow`/`Deny` |
| `Options` | `Options` |
| `All` | همه |

**۳) هزینه‌ی `.htaccess`**

```
هر درخواست:
  ۱. جستجوی .htaccess در هر پوشه از ریشه تا فایل
  ۲. پارس و اعمال
  ۳. (اگر تغییر کرده) کش
```

با `AllowOverride All` و ۵ سطح پوشه، هر درخواست ۵ فایل را چک می‌کند. در بار بالا، تفاوت ۱۰ تا ۳۰٪.

**۴) `DirectoryIndex`**

```apache
DirectoryIndex index.html index.php
```

**۵) احراز هویت با `.htaccess`**

```apache
AuthType Basic
AuthName "Restricted"
AuthUserFile /etc/apache2/.htpasswd
Require valid-user
```

```bash
sudo htpasswd -c /etc/apache2/.htpasswd admin
sudo htpasswd /etc/apache2/.htpasswd user2
```

> `Basic` رمز را base64 می‌فرستد؛ **فقط روی HTTPS**.

**۶) `ErrorDocument`**

```apache
ErrorDocument 404 /404.html
ErrorDocument 403 /403.html
ErrorDocument 500 "خطای داخلی"
```

**۷) `Options` در `.htaccess`**

```apache
Options -Indexes
Options +FollowSymLinks
```

**۸) `php_value` در `.htaccess`**

```apache
php_value upload_max_filesize 16M
php_value post_max_size 16M
php_value memory_limit 256M
```

فقط با `mod_php`؛ با `php-fpm` کار نمی‌کند (→ فصل ۲۸).

**۹) `mod_rewrite` در `.htaccess`**

```apache
RewriteEngine On
RewriteBase /
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```

**۱۰) `Require` در `.htaccess`**

```apache
<RequireAll>
    Require ip 192.168.56.0/24
    Require valid-user
</RequireAll>
```

### مثال‌های عملی

**Lab Scenario: انتقال `.htaccess` به vhost**

**قبل (کند):**

```apache
# /etc/apache2/sites-available/lab.conf
<VirtualHost *:80>
    ServerName lab.example.com
    DocumentRoot /var/www/lab
    <Directory /var/www/lab>
        AllowOverride All
    </Directory>
</VirtualHost>
```

```apache
# /var/www/lab/.htaccess
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.php [L]

Options -Indexes
ErrorDocument 404 /404.html
```

**بعد (سریع):**

```apache
# /etc/apache2/sites-available/lab.conf
<VirtualHost *:80>
    ServerName lab.example.com
    DocumentRoot /var/www/lab

    <Directory /var/www/lab>
        AllowOverride None
        Options -Indexes +FollowSymLinks
        Require all granted

        <IfModule mod_rewrite.c>
            RewriteEngine On
            RewriteCond %{REQUEST_FILENAME} !-f
            RewriteCond %{REQUEST_FILENAME} !-d
            RewriteRule ^ index.php [L]
        </IfModule>

        ErrorDocument 404 /404.html
    </Directory>
</VirtualHost>
```

```bash
sudo apachectl configtest && sudo systemctl reload apache2

# بنچمارک
ab -k -n 10000 -c 100 http://lab.example.com/
```

**`.htaccess` برای محافظت از فایل‌ها:**

```apache
# /var/www/lab/.htaccess
<FilesMatch "^\.">
    Require all denied
</FilesMatch>

<FilesMatch "\.(env|ini|log|sql|bak|old|sh)$">
    Require all denied
</FilesMatch>

<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteRule ^(.*/)?\.git/ - [F,L]
    RewriteRule ^(.*/)?\.env$ - [F,L]
</IfModule>
```

**احراز هویت برای یک مسیر:**

```apache
# /var/www/lab/admin/.htaccess
AuthType Basic
AuthName "Admin Area"
AuthUserFile /etc/apache2/.htpasswd
Require valid-user
```

```bash
sudo htpasswd -c /etc/apache2/.htpasswd admin
curl -sSI http://lab.example.com/admin/
# HTTP/1.1 401 Unauthorized
# WWW-Authenticate: Basic realm="Admin Area"

curl -sSI -u admin:password http://lab.example.com/admin/
# HTTP/1.1 200 OK
```

**تست `AllowOverride`:**

```bash
# با AllowOverride None، .htaccess نادیده گرفته می‌شود
echo 'ErrorDocument 404 "custom"' > /var/www/lab/.htaccess
curl -sS http://lab.example.com/nonexistent
# پاسخ پیش‌فرض Apache

# با AllowOverride All
curl -sS http://lab.example.com/nonexistent
# custom
```

**`htaccess` و `mod_php`:**

```apache
# فقط با mod_php
php_value upload_max_filesize 16M
php_value post_max_size 16M
php_flag display_errors Off
```

با `php-fpm`:

```ini
; /etc/php/8.3/fpm/php.ini
upload_max_filesize = 16M
post_max_size = 16M
display_errors = Off
```

### مقایسه و الگوهای مختلف

**`AllowOverride None` در برابر `All` در برابر گروه‌های خاص**

| | `None` | `All` | `FileInfo` |
|---|---|---|---|
| سرعت | ✅✅ | ❌ | ✅ |
| rewrite | ❌ | ✅ | ✅ |
| auth | ❌ | ✅ | ❌ |
| توصیه | ✅ تولید | هاست اشتراکی | سایت با rewrite |

**`.htaccess` در برابر vhost در برابر `conf.d`**

| | `.htaccess` | vhost | `conf.d` |
|---|---|---|---|
| محل | `DocumentRoot` | `sites-available` | `conf.d` |
| restart | ❌ | ✅ | ✅ |
| سرعت | ❌ | ✅ | ✅ |
| دسترسی | کاربر | root | root |

**`Basic` در برابر `Digest`**

| | Basic | Digest |
|---|---|---|
| رمز در شبکه | base64 (قابل خواندن) | هش |
| نیاز به HTTPS | ✅ | ⚠️ |
| پشتیبانی | ✅ | ⚠️ |
| توصیه | ✅ (با TLS) | ❌ |

**دیدگاه «`.htaccess` را حذف کنیم»**: در تولید، بله — قواعد را به vhost ببر و `AllowOverride None` بگذار. در هاست اشتراکی که دسترسی root نداری، چاره‌ای نیست.

### اشکال شایع

**۱) `.htaccess` با خطای سینتکس**
`500 Internal Server Error` بدون جزئیات. `error_log` را ببین.

**۲) `AllowOverride None` و انتظار rewrite**
`.htaccess` نادیده گرفته می‌شود. `FileInfo`.

**۳) `.htaccess` در پوشه‌ی والد و اثر روی فرزند**
قواعد ارث می‌رسند. `RewriteBase` و ترتیب مهم است.

**۴) `php_value` با `php-fpm`**
`500` یا بی‌اثر. `.user.ini` یا `php.ini`.

**۵) `.htaccess` قابل خواندن از وب**
اگر `AllowOverride` و `Options Indexes` بد باشد، خود فایل لو می‌رود. `<Files ".htaccess">Require all denied</Files>`.

**۶) `Require valid-user` بدون `AuthUserFile`**
`500`. فایل تعریف نشده.

**۷) `AuthType Basic` بدون TLS**
رمز در شبکه. فقط HTTPS.

**۸) `Options +Indexes` در `.htaccess`**
لیست فایل. `-Indexes`.

**۹) `RewriteRule` بدون `RewriteEngine On`**
بی‌اثر.

**۱۰) کش `.htaccess`**
`MaxKeepAliveRequests` و `mmap` می‌توانند تغییر را دیر نشان دهند. `reload`.

### نکات مهم (Tips & Tricks)

- **`AllowOverride None`** در تولید.
- **`<Files ".ht*">Require all denied</Files>`**.
- **`htpasswd`** برای Basic.
- **`LogLevel alert rewrite:trace3`** برای دیباگ rewrite.
- **`.user.ini`** برای PHP-FPM.
- **`ab`** برای اندازه‌گیری تفاوت.
- **`apachectl configtest`** بعد از هر تغییر vhost.

### خلاصه فصل

- `.htaccess` بدون restart پیکربندی را عوض می‌کند، به قیمت سرعت.
- `AllowOverride` تعیین می‌کند چه دستورهایی مجازند.
- در تولید، قواعد را به vhost منتقل کن.
- `Basic` auth فقط با HTTPS.
- `php_value` فقط با `mod_php`.
- `.htaccess` خودش باید محافظت شود.

**سؤال مروری:** چرا `AllowOverride All` در هر درخواست هزینه دارد و چطور آن را حذف می‌کنی بدون از دست دادن قواعد؟

---

## منابع و مراجع

**مستندات رسمی**
- [Apache 2.4 Documentation](https://httpd.apache.org/docs/2.4/)
- [VirtualHost Examples](https://httpd.apache.org/docs/2.4/vhosts/examples.html)
- [AllowOverride](https://httpd.apache.org/docs/2.4/mod/core.html#allowoverride)
- [Apache on Windows](https://httpd.apache.org/docs/2.4/platform/windows.html)
- [Apache Lounge (Windows builds)](https://www.apachelounge.com/)

**آموزش**
- [DigitalOcean — Apache Virtual Hosts](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-22-04)
- [Apache Week](https://www.apacheweek.com/)
- [SELinux and Apache](https://wiki.centos.org/HowTos/SELinux)

**ابزار**
- [ab](https://httpd.apache.org/docs/2.4/programs/ab.html)
- [htpasswd](https://httpd.apache.org/docs/2.4/programs/htpasswd.html)

---

**بعدی:** [`05-Nginx-Install-and-Config.md`](05-Nginx-Install-and-Config.md) — نصب، `server`، `location` و `try_files`
