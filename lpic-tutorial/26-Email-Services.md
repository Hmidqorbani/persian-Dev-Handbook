# فایل ۲۶ — سرویس‌های ایمیل

**مدت یادگیری**: ۸ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: فایل ۱۲ (مبانی MTA)، فایل ۲۲ (DNS)

---

## فهرست محتوا

- [فصل ۹۰: استفاده از سرورهای ایمیل](#فصل-۹۰-استفاده-از-سرورهای-ایمیل)
- [فصل ۹۱: مدیریت تحویل ایمیل](#فصل-۹۱-مدیریت-تحویل-ایمیل)
- [فصل ۹۲: مدیریت دسترسی به صندوق پستی](#فصل-۹۲-مدیریت-دسترسی-به-صندوق-پستی)

---

## فصل ۹۰: استفاده از سرورهای ایمیل

### مقدمه

در LPIC-1 مبانی MTA را دیدیم. در LPIC-2 عمیق‌تر می‌شویم: پیکربندی کامل Postfix، مسیریابی ایمیل و اجزای سیستم.

تصور کنید می‌خواهید یک سرور ایمیل کامل راه‌اندازی کنید — با ارسال، دریافت و دسترسی. این فصل پایه‌های آن را می‌دهد.

### مفاهیم اساسی

**اجزای سیستم ایمیل:**

| جزء | کار | نمونه |
|---|---|---|
| **MTA** | انتقال | Postfix, Sendmail, Exim |
| **MDA** | تحویل محلی | Procmail, Dovecot-LDA |
| **MUA** | کلاینت | Thunderbird, mutt |
| **MSA** | ارسال از کلاینت | Postfix (submission) |

**پروتکل‌ها:**

- **SMTP (25):** انتقال بین سرورها.
- **Submission (587):** ارسال از کلاینت.
- **IMAP (143/993):** دسترسی به صندوق.
- **POP3 (110/995):** دانلود صندوق.

**رکوردهای DNS:**

- **MX:** سرور ایمیل.
- **SPF (TXT):** مجاز بودن سرور.
- **DKIM:** امضای دیجیتال.
- **DMARC:** سیاست احراز.

### مثال‌های عملی

**پیکربندی Postfix:**

```bash
# /etc/postfix/main.cf
myhostname = mail.example.com
mydomain = example.com
myorigin = $mydomain
inet_interfaces = all
inet_protocols = ipv4
mydestination = $myhostname, localhost.$mydomain, $mydomain
mynetworks = 127.0.0.0/8, 192.168.1.0/24
relay_domains = $mydestination
home_mailbox = Maildir/
message_size_limit = 10485760
mailbox_size_limit = 1073741824
```

**فعال‌سازی submission:**

```bash
# /etc/postfix/master.cf
submission inet n - y - - smtpd
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_tls_auth_only=yes
```

**رکوردهای DNS:**

```
example.com.  IN  MX  10  mail.example.com.
example.com.  IN  TXT  "v=spf1 mx -all"
mail._domainkey.example.com.  IN  TXT  "v=DKIM1; k=rsa; p=..."
_dmarc.example.com.  IN  TXT  "v=DMARC1; p=quarantine"
```

**بررسی:**

```bash
# پیکربندی
sudo postfix check
postconf -n

# وضعیت
systemctl status postfix
postqueue -p
```

### مقایسه و الگوهای مختلف

**MTAها:**

| MTA | مزیت | مناسب |
|---|---|---|
| **Postfix** | امن، سریع، محبوب | عمومی |
| **Sendmail** | کلاسیک، پیچیده | legacy |
| **Exim** | انعطاف | Debian |
| **OpenSMTPD** | ساده | ساده |

**پورت‌ها:**

| پورت | کاربرد |
|---|---|
| 25 | SMTP (سرور-سرور) |
| 587 | Submission (کلاینت) |
| 465 | SMTPS (قدیمی) |
| 143/993 | IMAP/IMAPS |
| 110/995 | POP3/POP3S |

**روش‌های احراز ارسال:**

| روش | امنیت |
|---|---|
| SASL + TLS | بالا |
| IP allowlist | متوسط |
| بدون احراز | ناامن |

### اشکال شایع

**اشکال ۱: Open Relay.**
سرور اسپم می‌فرستد. راه‌حل: `mynetworks` محدود.

**اشکال ۲: نبود SPF/DKIM.**
ایمیل به اسپم می‌رود. راه‌حل: DNS درست.

**اشکال ۳: اشتباه MX.**
ایمیل نمی‌رسد. راه‌حل: MX درست.

**اشکال ۴: نبود rDNS.**
بعضی سرورها رد می‌کنند. راه‌حل: rDNS با ISP.

**اشکال ۵: فایروال پورت ۲۵.**
ارسال شکست می‌خورد. راه‌حل: باز کردن.

**اشکال ۶: نبود TLS.**
رمزها متن‌رو. راه‌حل: TLS.

### نکات مهم (Tips & Tricks)

- **`postconf -n` تنظیمات غیرپیش‌فرض.**
- **`postfix check` پیکربندی را بررسی می‌کند.**
- **SPF، DKIM و DMARC را تنظیم کنید.**
- **rDNS را با ISP تنظیم کنید.**
- **submission (587) را با TLS+SASL فعال کنید.**
- **`mynetworks` را محدود کنید.**
- **`postmap` برای lookup table.**
- **`mailq` صف را نشان می‌دهد.**
- **`telnet mail.example.com 25` تست SMTP.**
- **`openssl s_client -connect host:587 -starttls smtp` تست TLS.**
- **`dkim-genkey`/`opendkim` برای DKIM.**
- **`pflogsumm` تحلیل لاگ.**

### خلاصه فصل

سیستم ایمیل از MTA، MDA و MUA ساخته می‌شود. SMTP ارسال، IMAP/POP3 دریافت و submission ارسال کلاینت است. Postfix محبوب‌ترین MTA است. MX، SPF، DKIM و DMARC در DNS حیاتی‌اند. Open Relay خطرناک است؛ `mynetworks` را محدود کنید و submission را با TLS+SASL فعال کنید.

---

## فصل ۹۱: مدیریت تحویل ایمیل

### مقدمه

تحویل ایمیل شامل صف، مسیریابی، alias و فیلتر است. تصور کنید صف ایمیل پر شده و باید مدیریت کنید، یا می‌خواهید ایمیل‌های یک آدرس به چند نفر برود.

در این فصل مدیریت آن را یاد می‌گیرید.

### مفاهیم اساسی

**مفاهیم کلیدی:**

- **Queue:** صف ایمیل‌های در انتظار.
- **Alias:** نام‌های مستعار.
- **Relay:** ارجاع به سرور دیگر.
- **Transport:** روش تحویل.
- **Virtual domain:** دامنه‌های میزبانی‌شده.

**فایل‌های مهم:**

- `/etc/aliases` — نام‌های مستعار.
- `/etc/postfix/main.cf` — پیکربندی.
- `/var/spool/postfix/` — صف.

**وضعیت‌های صف:**

| وضعیت | معنا |
|---|---|
| active | در حال پردازش |
| deferred | تأخیر |
| hold | نگه‌داشته |
| incoming | در حال دریافت |

### مثال‌های عملی

**مدیریت صف:**

```bash
# نمایش صف
mailq
postqueue -p

# پاک‌سازی
sudo postsuper -d ALL           # همه
sudo postsuper -d <queue_id>    # یکی
sudo postsuper -r ALL           # بازگشت به صف
sudo postsuper -h ALL           # نگه‌داشتن
sudo postsuper -H ALL           # آزاد کردن

# پاک‌سازی نگه‌داشته‌شده
sudo postsuper -d ALL hold

# پردازش فوری
sudo postqueue -f
```

**Alias:**

```bash
# /etc/aliases
postmaster: root
root: alice
admin: alice, bob
team: :include:/etc/mail/team-list
info: |/usr/local/bin/process-info

# اعمال
sudo newaliases
```

**Virtual domain:**

```
# /etc/postfix/main.cf
virtual_mailbox_domains = example.com, example.org
virtual_mailbox_base = /var/mail/vhosts
virtual_mailbox_maps = hash:/etc/postfix/vmailbox
virtual_alias_maps = hash:/etc/postfix/valias
```

**Relay:**

```bash
# /etc/postfix/main.cf
relayhost = [smtp.provider.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_use_tls = yes
```

```bash
# /etc/postfix/sasl_passwd
[smtp.provider.com]:587 user:password
sudo postmap /etc/postfix/sasl_passwd
sudo chmod 600 /etc/postfix/sasl_passwd
```

### مقایسه و الگوهای مختلف

**روش‌های تحویل:**

| روش | توضیح |
|---|---|
| Local | صندوق محلی |
| Relay | ارجاع به سرور دیگر |
| Virtual | دامنه‌های میزبانی‌شده |
| Transport map | مسیریابی سفارشی |

**فایل‌های lookup:**

| نوع | پسوند | کاربرد |
|---|---|---|
| hash | `.db` | سریع |
| LDAP | — | دایرکتوری |
| MySQL | — | دیتابیس |
| regexp | — | regex |
| pcre | — | regex پیشرفته |

**فرمت صندوق:**

| فرمت | مزیت |
|---|---|
| mbox | ساده |
| Maildir | سریع، امن |

### اشکال شایع

**اشکال ۱: فراموش کردن `newaliases`.**
alias اعمال نمی‌شود. راه‌حل: `newaliases`.

**اشکال ۲: صف پر.**
ایمیل‌ها دیر تحویل می‌شوند. راه‌حل: مدیریت صف.

**اشکال ۳: نبود postmap.**
فایل lookup کار نمی‌کند. راه‌حل: `postmap`.

**اشکال ۴: مجوز فایل رمز.**
باید `600` باشد. راه‌حل: chmod.

**اشکال ۵: حلقهٔ alias.**
ایمیل بی‌نهایت می‌چرخد. راه‌حل: بررسی.

**اشکال ۶: اشتباه در virtual.**
ایمیل تحویل نمی‌شود. راه‌حل: پیکربندی درست.

### نکات مهم (Tips & Tricks)

- **`postqueue -f` صف را فوراً پردازش می‌کند.**
- **`postsuper -r ALL` ایمیل‌ها را دوباره در صف می‌گذارد.**
- **`newaliases` بعد از تغییر aliases.**
- **`postmap` بعد از تغییر lookup.**
- **`maillog` را برای دیباگ ببینید.**
- **برای relay، SASL و TLS لازم است.**
- **`postsuper -h` ایمیل را نگه می‌دارد (برای بررسی).**
- **`qshape` تحلیل صف.**
- **`pfqueue` رابط تعاملی.**
- **`mailq | tail -1` خلاصه.**
- **`postcat -q <id>` محتوای ایمیل صف.**
- **`virtual_alias_maps` برای forward.**

### خلاصه فصل

تحویل ایمیل شامل صف، alias و relay است. `mailq`/`postqueue` صف را مدیریت می‌کنند و `postsuper` پاک/بازگشت/نگه‌داشت می‌دهد. `/etc/aliases` با `newaliases` اعمال می‌شود. relay با `relayhost`، SASL و TLS پیکربندی می‌شود. فایل‌های lookup با `postmap` ساخته می‌شوند. Virtual domain برای میزبانی چند دامنه.

---

## فصل ۹۲: مدیریت دسترسی به صندوق پستی

### مقدمه

دسترسی به صندوق پستی با پروتکل‌های IMAP و POP3 انجام می‌شود. **Dovecot** پیاده‌سازی محبوب است. تصور کنید می‌خواهید کاربران از هر دستگاهی ایمیل‌شان را ببینند — IMAP این کار را می‌کند.

در این فصل پیکربندی آن را یاد می‌گیرید.

### مفاهیم اساسی

**IMAP در برابر POP3:**

| ویژگی | IMAP | POP3 |
|---|---|---|
| نگهداری روی سرور | بله | خیر (معمولاً) |
| چند دستگاه | عالی | ضعیف |
| پوشه‌ها | بله | محدود |
| پورت | 143/993 | 110/995 |

**Dovecot:** MDA و IMAP/POP3 server.

**فرمت صندوق:** `mbox` (تک‌فایل) یا `Maildir` (پوشه‌ای).

**محل فایل‌ها:**

- `/etc/dovecot/dovecot.conf` — پیکربندی.
- `/etc/dovecot/conf.d/` — پیکربندی‌های جدا.

### مثال‌های عملی

**نصب و پیکربندی Dovecot:**

```bash
sudo apt install dovecot-core dovecot-imapd dovecot-pop3d
sudo systemctl enable --now dovecot
```

**پیکربندی (`/etc/dovecot/dovecot.conf`):**

```
protocols = imap pop3
mail_location = maildir:~/Maildir

# احراز هویت
disable_plaintext_auth = yes
auth_mechanisms = plain login
ssl = required
```

**TLS (`/etc/dovecot/conf.d/10-ssl.conf`):**

```
ssl = required
ssl_cert = </etc/ssl/certs/dovecot.pem
ssl_key = </etc/ssl/private/dovecot.key
ssl_min_protocol = TLSv1.2
```

**SASL با Postfix:**

```
# /etc/postfix/main.cf
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_auth_enable = yes
smtpd_sasl_security_options = noanonymous
```

```
# /etc/dovecot/conf.d/10-master.conf
service auth {
  unix_listener /var/spool/postfix/private/auth {
    mode = 0660
    user = postfix
    group = postfix
  }
}
```

**بررسی:**

```bash
sudo doveconf -n
sudo systemctl restart dovecot
sudo doveadm mailbox list -u alice
```

### مقایسه و الگوهای مختلف

**فرمت‌های صندوق:**

| فرمت | مزیت | معایب |
|---|---|---|
| **mbox** | ساده | قفل‌شدن، کندی |
| **Maildir** | سریع، امن | فایل‌های زیاد |

**سرورهای IMAP:**

| سرور | ویژگی |
|---|---|
| **Dovecot** | محبوب، سریع |
| **Cyrus** | سازمانی |
| **Courier** | کلاسیک |

**دستورهای doveadm:**

| دستور | کار |
|---|---|
| `mailbox list` | فهرست صندوق‌ها |
| `search` | جستجو |
| `fetch` | خواندن |
| `expunge` | پاک‌سازی |
| `who` | کاربران متصل |

### اشکال شایع

**اشکال ۱: نبود TLS.**
رمزها متن‌رو می‌روند. راه‌حل: `ssl = required`.

**اشکال ۲: mail_location اشتباه.**
صندوق پیدا نمی‌شود. راه‌حل: مسیر درست.

**اشکال ۳: نبود SASL.**
ارسال از کلاینت کار نمی‌کند. راه‌حل: SASL.

**اشکال ۴: مجوز پوشه.**
Dovecot به Maildir دسترسی ندارد. راه‌حل: chown.

**اشکال ۵: گواهی اشتباه.**
اتصال رد می‌شود. راه‌حل: گواهی درست.

**اشکال ۶: فایروال پورت ۱۴۳/۹۹۳.**
دسترسی نیست. راه‌حل: باز کردن.

### نکات مهم (Tips & Tricks)

- **`doveconf -n` پیکربندی مؤثر را نشان می‌دهد.**
- **`doveadm` ابزار مدیریت (کاربران، صندوق).**
- **Maildir برای کارایی بهتر.**
- **TLS را الزامی کنید.**
- **Dovecot را برای SASL به Postfix وصل کنید.**
- **`doveadm mailbox list -u user` صندوق‌ها را فهرست می‌کند.**
- **`doveadm who` کاربران متصل.**
- **`doveadm search` جستجو.**
- **`mail_location = maildir:~/Maildir` استاندارد.**
- **`/etc/dovecot/conf.d/` برای پیکربندی ماژولار.**
- **LMTP برای تحویل.**
- **`doveadm index` ایندکس.**

### خلاصه فصل

دسترسی به صندوق با IMAP (نگهداری سرور، چند دستگاه) و POP3 (دانلود) انجام می‌شود. Dovecot محبوب‌ترین سرور است. `mail_location` و فرمت (Maildir) مهم‌اند. TLS را الزامی کنید و Dovecot را برای SASL به Postfix وصل کنید. `doveconf` و `doveadm` ابزارهای مدیریت‌اند.

---

## منابع و مراجع

- man pages: `man postfix`, `man aliases`, `man postconf`, `man doveconf`, `man doveadm`
- Postfix Documentation: https://www.postfix.org/documentation.html
- Dovecot Documentation: https://doc.dovecot.org
- SPF/DKIM/DMARC: https://www.rfc-editor.org

**بعدی:** [فایل ۲۷ — امنیت سیستم (LPIC-2)](27-System-Security-LPIC2.md)
