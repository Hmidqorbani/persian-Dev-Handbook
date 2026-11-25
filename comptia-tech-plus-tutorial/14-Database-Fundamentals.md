# مبانی پایگاه داده

**مدت یادگیری**: ۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۵ (داده) و فایل ۱۲ (انواع داده)

---

## فهرست محتوا

- [فصل ۶۰: پایگاه داده چیست — DBMS و مزایا](#ch-60)
- [فصل ۶۱: مدل رابطه‌ای — جدول، رکورد، فیلد، کلید](#ch-61)
- [فصل ۶۲: SQL — چهار عمل اصلی و JOIN](#ch-62)
- [فصل ۶۳: طراحی پایگاه داده — نرمال‌سازی و یکپارچگی](#ch-63)
- [فصل ۶۴: NoSQL، انبار داده و مقیاس‌پذیری](#ch-64)

---

<a name="ch-60"></a>
## فصل ۶۰: پایگاه داده چیست — DBMS و مزایا

### مقدمه

فایل‌های Excel برای چند صد ردیف عالی‌اند. برای میلیون ردیف و چند کاربر همزمان، نه. اینجاست که پایگاه داده وارد می‌شود.

دامنهٔ «مبانی داده و پایگاه داده» (۱۳٪) در Tech+ یکی از شش حوزهٔ آزمون است — و بخش زیادی از سؤالات از مفاهیم پایه می‌آید.

### مفاهیم اساسی

**پایگاه داده چیست؟**

مجموعه‌ای سازمان‌یافته از داده که برای بازیابی، مدیریت و به‌روزرسانی طراحی شده.

**DBMS — سیستم مدیریت**

نرم‌افزاری که بین کاربر و داده می‌نشیند.

```
کاربر/برنامه → DBMS → داده روی دیسک
```

| کار DBMS | توضیح |
|---|---|
| ذخیره‌سازی | مدیریت فایل |
| بازیابی | کوئری |
| امنیت | مجوز |
| همزمانی | چند کاربر |
| پشتیبان | Backup/Restore |
| یکپارچگی | قواعد |

**مزایای پایگاه داده بر فایل**

| مزیت | توضیح |
|---|---|
| **کاهش تکرار** | داده یک‌جا |
| **یکپارچگی** | قواعد اعمال می‌شود |
| **همزمانی** | چند کاربر |
| **امنیت** | مجوز سطح ردیف/ستون |
| **پشتیبان‌گیری** | ابزار |
| **بازیابی** | تراکنش |

**فایل در برابر پایگاه داده**

| معیار | فایل (Excel/CSV) | پایگاه داده |
|---|---|---|
| حجم | محدود | بزرگ |
| کاربر همزمان | ۱ | هزاران |
| یکپارچگی | دستی | خودکار |
| کوئری | دستی | SQL |
| امنیت | فایل | نقش |

**انواع پایگاه داده**

| نوع | مدل | مثال |
|---|---|---|
| **Relational (RDBMS)** | جدول | MySQL، PostgreSQL، SQL Server، Oracle |
| **NoSQL — Document** | سند | MongoDB |
| **NoSQL — Key-Value** | کلید-مقدار | Redis |
| **NoSQL — Column** | ستونی | Cassandra |
| **NoSQL — Graph** | گراف | Neo4j |

**ACID — چهار تضمین تراکنش**

| حرف | معنی | مثال |
|---|---|---|
| **A**tomicity | همه یا هیچ | انتقال پول |
| **C**onsistency | قواعد حفظ شود | موجودی منفی نشود |
| **I**solation | تراکنش‌ها مستقل | دو خرید همزمان |
| **D**urability | پس از commit پایدار | قطع برق |

**مثال ACID:** انتقال ۱۰۰ تومان از حساب A به B:

```
BEGIN
  A ← A − 100
  B ← B + 100
COMMIT
```

اگر بین دو خط خطا رخ دهد، هیچ‌کدام اعمال نمی‌شود (Atomicity).

**CAP Theorem**

در سیستم توزیع‌شده، فقط دو مورد از سه را می‌توانی داشته باشی:

| حرف | معنی |
|---|---|
| **C**onsistency | همه یک داده را ببینند |
| **A**vailability | همیشه پاسخ |
| **P**artition tolerance | تحمل قطعی شبکه |

**نکته:** در عمل، P اجتناب‌ناپذیر است؛ انتخاب بین C و A.

**OLTP در برابر OLAP**

| معیار | OLTP | OLAP |
|---|---|---|
| کار | تراکنش روزانه | تحلیل |
| کوئری | کوتاه، زیاد | بلند، کم |
| داده | جاری | تاریخی |
| مثال | فروشگاه | انبار داده |

### مثال‌های عملی

**سناریوی Lab: از فایل به پایگاه داده**

**وضعیت:** فروشگاه کوچک با فایل Excel.

| مشکل | اثر |
|---|---|
| دو نفر همزمان ویرایش | داده گم می‌شود |
| ۵۰٬۰۰۰ ردیف | کند |
| نام مشتری تکراری | ناسازگاری |
| بدون مجوز | همه می‌بینند |

**راه‌حل: PostgreSQL**

```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(id),
    total NUMERIC(12,2) NOT NULL,
    created_at TIMESTAMP DEFAULT now()
);
```

**نتیجه:**

| معیار | قبل | بعد |
|---|---|---|
| کاربر همزمان | ۱ | چند |
| یکپارچگی | دستی | خودکار |
| سرعت کوئری | کند | ایندکس |
| امنیت | فایل | نقش |

**سناریوی دوم: انتخاب نوع پایگاه داده**

| نیاز | نوع |
|---|---|
| تراکنش مالی | RDBMS |
| لاگ و رویداد | NoSQL (Document) |
| کش | Key-Value |
| شبکهٔ اجتماعی | Graph |
| تحلیل تاریخی | انبار داده |

**سناریوی سوم: تراکنش در عمل**

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

اگر بین دو `UPDATE` خطا رخ دهد، `ROLLBACK` خودکار یا دستی.

### مقایسه و الگوهای مختلف

**سه مدل داده**

| مدل | ساختار | مناسب |
|---|---|---|
| Relational | جدول | دادهٔ ساختاریافته |
| Document | JSON | دادهٔ نیمه‌ساختاریافته |
| Graph | گره/یال | روابط پیچیده |

**الگوی «RDBMS پیش‌فرض»**

اگر شک داری، RDBMS انتخاب کن. ۸۰٪ پروژه‌ها با آن حل می‌شوند.

### اشکال شایع

**۱) استفاده از Excel برای دادهٔ بزرگ**
کندی و ناسازگاری.

**۲) بدون کلید اصلی**
ردیف تکراری.

**۳) بدون تراکنش**
دادهٔ نیمه‌کاره.

**۴) نادیده گرفتن ACID**
انتقال پول ناقص.

**۵) انتخاب NoSQL بدون نیاز**
پیچیدگی بی‌دلیل.

**۶) بدون پشتیبان**
فاجعه.

**۷) بدون ایندکس**
کوئری کند.

**۸) مجوز همه‌کاره**
امنیت.

### نکات مهم (Tips & Tricks)

- **DBMS = لایهٔ مدیریت.**
- **ACID** را حفظ کن.
- **RDBMS پیش‌فرض.**
- **تراکنش برای عملیات چندمرحله‌ای.**
- **کلید اصلی همیشه.**
- **ایندکس برای کوئری.**
- **CAP: دو از سه.**

### خلاصه فصل

- پایگاه داده، داده را سازمان‌یافته نگه می‌دارد؛ DBMS آن را مدیریت می‌کند.
- مزایا: کاهش تکرار، یکپارچگی، همزمانی، امنیت.
- RDBMS با جدول؛ NoSQL با مدل‌های دیگر.
- ACID چهار تضمین تراکنش است.
- CAP می‌گوید دو از سه.
- OLTP برای تراکنش، OLAP برای تحلیل.

---

<a name="ch-61"></a>
## فصل ۶۱: مدل رابطه‌ای — جدول، رکورد، فیلد، کلید

### مقدمه

مدل رابطه‌ای، ساده‌ترین و پرکاربردترین مدل داده است: همه‌چیز در جدول. اما پشت این سادگی، مفاهیمی هست که طراحی درست را ممکن می‌کند.

### مفاهیم اساسی

**اجزای جدول**

```
        ستون (Field/Column)
        ↓
   ┌────┬────────┬─────────┐
   │ id │ name   │ email   │  ← سرستون
   ├────┼────────┼─────────┤
   │ 1  │ Ali    │ a@b.com │  ← ردیف (Row/Record)
   │ 2  │ Sara   │ s@b.com │
   └────┴────────┴─────────┘
```

| اصطلاح | معنی |
|---|---|
| **Table** | جدول |
| **Row/Record** | ردیف |
| **Column/Field** | ستون |
| **Value** | مقدار خانه |
| **Schema** | ساختار جدول |

**کلید اصلی (Primary Key)**

ستونی که هر ردیف را یکتا شناسایی می‌کند.

| ویژگی | توضیح |
|---|---|
| یکتا | تکراری نیست |
| غیر Null | همیشه مقدار دارد |
| تغییرناپذیر | معمولاً |

```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);
```

**کلید خارجی (Foreign Key)**

ستونی که به کلید اصلی جدول دیگر اشاره می‌کند.

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(id)
);
```

**چرا؟** یکپارچگی ارجاعی: نمی‌توانی سفارش برای مشتری ناموجود بسازی.

**کلید مرکب (Composite Key)**

ترکیب دو یا چند ستون به‌عنوان کلید.

```sql
CREATE TABLE enrollments (
    student_id INTEGER,
    course_id INTEGER,
    grade NUMERIC(4,2),
    PRIMARY KEY (student_id, course_id)
);
```

**کلید طبیعی در برابر مصنوعی**

| نوع | مثال | مزیت | عیب |
|---|---|---|---|
| **Natural** | کد ملی | معنا دارد | ممکن است تغییر کند |
| **Surrogate** | `id` خودکار | پایدار | بی‌معنا |

**توصیه:** Surrogate برای کلید اصلی، Natural برای کلید یکتا.

**روابط**

| نوع | مثال | پیاده‌سازی |
|---|---|---|
| **One-to-One** | کاربر/پروفایل | FK یکتا |
| **One-to-Many** | مشتری/سفارش | FK در «چند» |
| **Many-to-Many** | دانشجو/درس | جدول واسط |

**Many-to-Many — جدول واسط**

```
students ──┐
           ├── enrollments (student_id, course_id)
courses  ──┘
```

**نمودار ER**

| نماد | معنی |
|---|---|
| مستطیل | موجودیت |
| بیضی | صفت |
| لوزی | رابطه |
| خط | اتصال |

**ایندکس (Index)**

ساختار کمکی برای سرعت جستجو.

```sql
CREATE INDEX idx_orders_customer ON orders(customer_id);
```

| مزیت | عیب |
|---|---|
| کوئری سریع | فضای بیشتر |
| مرتب‌سازی سریع | کندی در نوشتن |

**توصیه:** برای ستون‌های `WHERE` و `JOIN` ایندکس بگذار، نه همه‌چیز.

**محدودیت‌ها (Constraints)**

| محدودیت | کار |
|---|---|
| `NOT NULL` | خالی نباشد |
| `UNIQUE` | یکتا |
| `CHECK` | شرط |
| `DEFAULT` | مقدار پیش‌فرض |
| `FOREIGN KEY` | ارجاع معتبر |

### مثال‌های عملی

**سناریوی Lab: طراحی جدول فروشگاه**

**موجودیت‌ها:** مشتری، محصول، سفارش، آیتم سفارش.

```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT now()
);

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    price NUMERIC(12,2) NOT NULL CHECK (price >= 0),
    stock INTEGER NOT NULL DEFAULT 0
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customers(id),
    status TEXT NOT NULL DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT now()
);

CREATE TABLE order_items (
    order_id INTEGER REFERENCES orders(id) ON DELETE CASCADE,
    product_id INTEGER REFERENCES products(id),
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(12,2) NOT NULL,
    PRIMARY KEY (order_id, product_id)
);
```

**تحلیل:**

| تصمیم | چرا |
|---|---|
| `unit_price` در آیتم | قیمت تاریخی حفظ شود |
| `ON DELETE CASCADE` | حذف سفارش، آیتم‌ها را حذف کند |
| `CHECK` | جلوگیری از دادهٔ نامعتبر |
| کلید مرکب | یک محصول در یک سفارش یک بار |

**نکتهٔ مهم:** `unit_price` را از `products.price` کپی می‌کنیم، چون قیمت محصول ممکن است بعداً عوض شود ولی فاکتور قدیمی نباید تغییر کند.

**سناریوی دوم: رابطهٔ Many-to-Many**

```sql
CREATE TABLE students (id SERIAL PRIMARY KEY, name TEXT);
CREATE TABLE courses (id SERIAL PRIMARY KEY, title TEXT);

CREATE TABLE enrollments (
    student_id INTEGER REFERENCES students(id),
    course_id INTEGER REFERENCES courses(id),
    enrolled_at DATE DEFAULT CURRENT_DATE,
    PRIMARY KEY (student_id, course_id)
);
```

**سناریوی سوم: ایندکس**

```sql
-- بدون ایندکس: full scan
SELECT * FROM orders WHERE customer_id = 42;

-- با ایندکس: سریع
CREATE INDEX idx_orders_customer ON orders(customer_id);
```

| تعداد ردیف | بدون ایندکس | با ایندکس |
|---|---|---|
| ۱٬۰۰۰ | ~۱ms | ~۱ms |
| ۱٬۰۰۰٬۰۰۰ | ~۵۰۰ms | ~۲ms |

### مقایسه و الگوهای مختلف

**سه نوع کلید**

| نوع | کار | مثال |
|---|---|---|
| Primary | یکتایی | `id` |
| Foreign | ارجاع | `customer_id` |
| Candidate | گزینهٔ کلید | `email` |

**الگوی «کلید مصنوعی + کلید یکتا»**

```sql
id SERIAL PRIMARY KEY,
email TEXT UNIQUE NOT NULL
```

`id` برای ارجاع، `email` برای یکتایی منطقی.

### اشکال شایع

**۱) بدون کلید اصلی**
ردیف تکراری و غیرقابل ارجاع.

**۲) کلید طبیعی متغیر**
کد ملی/ایمیل ممکن است عوض شود.

**۳) بدون کلید خارجی**
دادهٔ یتیم.

**۴) ایندکس روی همه ستون‌ها**
کندی نوشتن.

**۵) بدون `CHECK`**
قیمت منفی.

**۶) `ON DELETE` بدون تصمیم**
حذف آبشاری ناخواسته.

**۷) نام‌گذاری ناسازگار**
`CustomerID` و `customer_id`.

**۸) بدون `NOT NULL`**
خانهٔ خالی.

### نکات مهم (Tips & Tricks)

- **کلید اصلی همیشه.**
- **Surrogate برای PK، Natural برای UNIQUE.**
- **Many-to-Many با جدول واسط.**
- **ایندکس برای `WHERE` و `JOIN`.**
- **`CHECK` برای اعتبارسنجی.**
- **`ON DELETE` را آگاهانه انتخاب کن.**
- **قیمت تاریخی را کپی کن.**

### خلاصه فصل

- جدول از ردیف و ستون ساخته می‌شود.
- کلید اصلی یکتایی را تضمین می‌کند.
- کلید خارجی یکپارچگی ارجاعی می‌سازد.
- Many-to-Many با جدول واسط پیاده می‌شود.
- ایندکس سرعت خواندن را بالا و نوشتن را پایین می‌آورد.
- محدودیت‌ها دادهٔ نامعتبر را رد می‌کنند.

---

<a name="ch-62"></a>
## فصل ۶۲: SQL — چهار عمل اصلی و JOIN

### مقدمه

SQL زبان پایگاه داده است. چهار دستور اصلی دارد و یک مفهوم کلیدی: JOIN. اگر این پنج چیز را بدانی، ۸۰٪ کارها را انجام می‌دهی.

### مفاهیم اساسی

**چهار دستهٔ دستور**

| دسته | دستورها | کار |
|---|---|---|
| **DQL** | `SELECT` | خواندن |
| **DML** | `INSERT`، `UPDATE`، `DELETE` | تغییر داده |
| **DDL** | `CREATE`، `ALTER`، `DROP` | تغییر ساختار |
| **DCL** | `GRANT`، `REVOKE` | مجوز |

**SELECT — خواندن**

```sql
SELECT name, email
FROM customers
WHERE created_at > '2026-01-01'
ORDER BY name ASC
LIMIT 10;
```

| بخش | کار |
|---|---|
| `SELECT` | ستون‌ها |
| `FROM` | جدول |
| `WHERE` | فیلتر |
| `ORDER BY` | مرتب‌سازی |
| `LIMIT` | تعداد |

**عملگرهای WHERE**

| عملگر | مثال |
|---|---|
| `=` | `status = 'paid'` |
| `>` `<` | `total > 100` |
| `BETWEEN` | `total BETWEEN 10 AND 100` |
| `IN` | `status IN ('paid','shipped')` |
| `LIKE` | `name LIKE 'A%'` |
| `IS NULL` | `email IS NULL` |

**توابع تجمیعی**

| تابع | کار |
|---|---|
| `COUNT` | تعداد |
| `SUM` | جمع |
| `AVG` | میانگین |
| `MIN`/`MAX` | کمینه/بیشینه |

```sql
SELECT status, COUNT(*) AS orders, SUM(total) AS revenue
FROM orders
GROUP BY status
HAVING SUM(total) > 1000;
```

**نکته:** `WHERE` قبل از گروه‌بندی، `HAVING` بعد از آن.

**INSERT**

```sql
INSERT INTO customers (name, email)
VALUES ('Ali', 'ali@example.com');
```

**UPDATE**

```sql
UPDATE orders
SET status = 'shipped'
WHERE id = 42;
```

**هشدار:** `UPDATE` بدون `WHERE`، همهٔ ردیف‌ها را عوض می‌کند.

**DELETE**

```sql
DELETE FROM orders WHERE id = 42;
```

**هشدار:** `DELETE` بدون `WHERE`، همه را پاک می‌کند.

**JOIN — اتصال جدول‌ها**

```sql
SELECT c.name, o.id, o.total
FROM customers c
JOIN orders o ON o.customer_id = c.id;
```

| نوع JOIN | نتیجه |
|---|---|
| `INNER JOIN` | فقط تطابق‌ها |
| `LEFT JOIN` | همهٔ چپ + تطابق راست |
| `RIGHT JOIN` | همهٔ راست + تطابق چپ |
| `FULL JOIN` | همه از هر دو |

**تشبیه:**

```
INNER: فقط مشتریانی که سفارش دارند
LEFT:  همهٔ مشتریان، حتی بدون سفارش
```

**مثال LEFT JOIN:**

```sql
SELECT c.name, COUNT(o.id) AS order_count
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id
GROUP BY c.name;
```

مشتری بدون سفارش، `0` می‌گیرد.

**Subquery**

```sql
SELECT name FROM customers
WHERE id IN (
    SELECT customer_id FROM orders WHERE total > 1000
);
```

**View**

```sql
CREATE VIEW big_orders AS
SELECT * FROM orders WHERE total > 1000;
```

مزیت: کوئری پیچیده با نام ساده.

### مثال‌های عملی

**سناریوی Lab: گزارش فروش**

**سؤال:** پنج مشتری برتر بر اساس مجموع خرید.

```sql
SELECT
    c.name,
    COUNT(o.id) AS orders,
    SUM(o.total) AS revenue
FROM customers c
JOIN orders o ON o.customer_id = c.id
WHERE o.status = 'paid'
GROUP BY c.name
ORDER BY revenue DESC
LIMIT 5;
```

**تحلیل:**

| بخش | کار |
|---|---|
| `JOIN` | اتصال مشتری و سفارش |
| `WHERE` | فقط پرداخت‌شده |
| `GROUP BY` | گروه بر اساس مشتری |
| `SUM` | جمع |
| `ORDER BY DESC` | نزولی |
| `LIMIT 5` | پنج تا |

**سناریوی دوم: پیدا کردن دادهٔ یتیم**

```sql
-- سفارش‌هایی که مشتری‌شان حذف شده
SELECT o.*
FROM orders o
LEFT JOIN customers c ON c.id = o.customer_id
WHERE c.id IS NULL;
```

**درس:** `LEFT JOIN` + `IS NULL` الگوی کلاسیک پیدا کردن رکوردهای بی‌مرجع است.

**سناریوی سوم: به‌روزرسانی امن**

```sql
-- ❌ خطرناک
UPDATE products SET price = price * 1.1;

-- ✅ امن
BEGIN;
UPDATE products SET price = price * 1.1 WHERE category = 'book';
SELECT * FROM products WHERE category = 'book';   -- بررسی
COMMIT;
```

**سناریوی چهارم: SQL Injection**

```python
# ❌ آسیب‌پذیر
query = f"SELECT * FROM users WHERE name = '{name}'"
# ورودی: ' OR '1'='1  → همهٔ کاربران

# ✅ پارامتری
cursor.execute("SELECT * FROM users WHERE name = %s", (name,))
```

**درس:** هرگز رشته را به SQL نچسبان. پارامتر استفاده کن.

### مقایسه و الگوهای مختلف

**سه سطح کوئری**

| سطح | مثال |
|---|---|
| ساده | `SELECT * FROM t` |
| با JOIN | چند جدول |
| با Subquery/CTE | پیچیده |

**الگوی «CTE برای خوانایی»**

```sql
WITH paid_orders AS (
    SELECT * FROM orders WHERE status = 'paid'
)
SELECT customer_id, SUM(total)
FROM paid_orders
GROUP BY customer_id;
```

### اشکال شایع

**۱) `SELECT *`**
ستون‌های اضافه، کندی. ستون را نام ببر.

**۲) `UPDATE`/`DELETE` بدون `WHERE`**
فاجعه.

**۳) SQL Injection**
پارامتر.

**۴) `WHERE` به‌جای `HAVING`**
خطا در تجمیع.

**۵) `INNER JOIN` به‌جای `LEFT`**
مشتری بدون سفارش گم می‌شود.

**۶) بدون `LIMIT`**
کوئری سنگین.

**۷) مقایسه با `NULL`**
`= NULL` کار نمی‌کند. `IS NULL`.

**۸) بدون ایندکس روی `JOIN`**
کندی.

### نکات مهم (Tips & Tricks)

- **چهار دستهٔ SQL** را حفظ کن.
- **`WHERE` قبل، `HAVING` بعد.**
- **`LEFT JOIN` + `IS NULL`** برای یتیم.
- **`UPDATE`/`DELETE` با `WHERE`.**
- **پارامتر، نه چسباندن رشته.**
- **`SELECT` ستون‌های لازم.**
- **`LIMIT` برای امنیت.**

### خلاصه فصل

- SQL چهار دسته دارد: DQL، DML، DDL، DCL.
- `SELECT` با `WHERE`، `GROUP BY`، `HAVING`، `ORDER BY`.
- `JOIN` جدول‌ها را وصل می‌کند؛ `LEFT` همهٔ چپ را نگه می‌دارد.
- `UPDATE`/`DELETE` بدون `WHERE` فاجعه است.
- SQL Injection با پارامتر جلوگیری می‌شود.
- `IS NULL` برای مقایسهٔ NULL.

---

<a name="ch-63"></a>
## فصل ۶۳: طراحی پایگاه داده — نرمال‌سازی و یکپارچگی

### مقدمه

یک پایگاه دادهٔ بدطراحی، بعد از شش ماه غیرقابل نگه‌داری می‌شود: داده تکراری، ناسازگار، و کوئری‌های کند. نرمال‌سازی، راه‌حل است.

### مفاهیم اساسی

**چرا طراحی مهم است؟**

| مشکل | اثر |
|---|---|
| تکرار داده | ناسازگاری |
| جدول غول‌پیکر | کوئری کند |
| بدون کلید | دادهٔ یتیم |
| نوع اشتباه | خطا |

**آنومالی‌ها**

| نوع | مثال |
|---|---|
| **Insert** | نمی‌توانی مشتری بدون سفارش ثبت کنی |
| **Update** | تغییر آدرس در ۱۰ ردیف |
| **Delete** | حذف سفارش، اطلاعات مشتری را هم می‌برد |

**نرمال‌سازی**

فرآیند سازمان‌دهی برای کاهش تکرار.

**۱NF — فرم اول**

> هر خانه، یک مقدار اتمیک.

| ❌ بد | ✅ خوب |
|---|---|
| `phones: "0912..., 0935..."` | جدول جدا برای تلفن |

**۲NF — فرم دوم**

> هر ستون غیرکلیدی، به **کل** کلید اصلی وابسته باشد.

| ❌ بد | ✅ خوب |
|---|---|
| `(order_id, product_id, product_name, quantity)` — `product_name` فقط به `product_id` وابسته | جدول `products` جدا |

**۳NF — فرم سوم**

> هیچ وابستگی транзитивی نباشد.

| ❌ بد | ✅ خوب |
|---|---|
| `(id, customer_id, customer_city, customer_zip)` — شهر به zip وابسته | جدول `customers` جدا |

**مثال کامل**

**جدول غیرنرمال:**

```
orders(id, customer_name, customer_city, product_name, product_price, qty)
```

**مشکلات:**
- نام مشتری در هر سفارش تکرار.
- قیمت محصول در هر سفارش تکرار.
- تغییر شهر، چند ردیف.

**بعد از نرمال‌سازی:**

```
customers(id, name, city)
products(id, name, price)
orders(id, customer_id, created_at)
order_items(order_id, product_id, qty, unit_price)
```

**Denormalization — چرا گاهی تکرار می‌کنیم؟**

| مزیت | عیب |
|---|---|
| کوئری سریع‌تر | دادهٔ تکراری |
| کمتر JOIN | ناسازگاری ممکن |

**مثال:** در انبار داده، تکرار عمدی است.

**یکپارچگی داده**

| نوع | معنی |
|---|---|
| **Entity** | کلید اصلی یکتاست |
| **Referential** | کلید خارجی معتبر |
| **Domain** | مقدار در محدوده |
| **Business** | قاعدهٔ کسب‌وکار |

**محدودیت‌ها در عمل**

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    price NUMERIC(12,2) NOT NULL CHECK (price >= 0),
    category TEXT NOT NULL,
    sku TEXT UNIQUE NOT NULL
);
```

**نوع دادهٔ درست**

| داده | نوع |
|---|---|
| پول | `NUMERIC(12,2)` — نه `FLOAT` |
| تاریخ | `DATE`/`TIMESTAMP` |
| بولین | `BOOLEAN` |
| متن کوتاه | `VARCHAR(n)` |
| متن بلند | `TEXT` |
| JSON | `JSONB` (PostgreSQL) |

**هشدار:** `FLOAT` برای پول، خطای گردکردن می‌آورد.

### مثال‌های عملی

**سناریوی Lab: نرمال‌سازی یک جدول**

**جدول اولیه:**

```sql
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    customer_name TEXT,
    customer_phone TEXT,
    product_name TEXT,
    product_category TEXT,
    quantity INTEGER,
    price NUMERIC
);
```

**گام ۱ — ۱NF:** تلفن‌های چندگانه → جدول جدا.

**گام ۲ — ۲NF:** `product_name` و `product_category` فقط به محصول وابسته‌اند → جدول `products`.

**گام ۳ — ۳NF:** `product_category` به `product_name` وابسته است → جدول `categories`.

**نتیجه:**

```sql
CREATE TABLE categories (id SERIAL PRIMARY KEY, name TEXT UNIQUE);
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    category_id INTEGER REFERENCES categories(id),
    price NUMERIC(12,2) CHECK (price >= 0)
);
CREATE TABLE customers (id SERIAL PRIMARY KEY, name TEXT NOT NULL);
CREATE TABLE customer_phones (
    customer_id INTEGER REFERENCES customers(id),
    phone TEXT,
    PRIMARY KEY (customer_id, phone)
);
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(id),
    product_id INTEGER REFERENCES products(id),
    quantity INTEGER CHECK (quantity > 0),
    unit_price NUMERIC(12,2) NOT NULL
);
```

**سناریوی دوم: تست یکپارچگی**

```sql
-- باید خطا بدهد: قیمت منفی
INSERT INTO products (name, price) VALUES ('x', -5);

-- باید خطا بدهد: سفارش برای مشتری ناموجود
INSERT INTO sales (customer_id, product_id, quantity, unit_price)
VALUES (9999, 1, 1, 10);
```

**درس:** محدودیت‌ها، دادهٔ نامعتبر را در **زمان نوشتن** رد می‌کنند، نه بعداً.

**سناریوی سوم: تشخیص نرمال‌نبودن**

| نشانه | احتمال |
|---|---|
| ستون با کاما | ۱NF |
| ستون وابسته به بخشی از کلید | ۲NF |
| ستون وابسته به ستون غیرکلیدی | ۳NF |
| جدول با ۳۰ ستون | احتمالاً چند موجودیت |

### مقایسه و الگوهای مختلف

**سه سطح نرمال‌سازی**

| سطح | قاعده | کاربرد |
|---|---|---|
| ۱NF | اتمیک | همه‌جا |
| ۲NF | وابستگی کامل | تراکنشی |
| ۳NF | بدون транзитив | تراکنشی |
| BCNF+ | پیشرفته | خاص |

**الگوی «نرمال برای OLTP، دنرمال برای OLAP»**

```
OLTP: ۳NF (بدون تکرار، تراکنش سالم)
OLAP: Star Schema (تکرار عمدی، کوئری سریع)
```

### اشکال شایع

**۱) جدول با ۵۰ ستون**
چند موجودیت در یک جدول.

**۲) ستون با کاما**
۱NF نقض.

**۳) `FLOAT` برای پول**
خطای گردکردن.

**۴) بدون کلید خارجی**
دادهٔ یتیم.

**۵) دنرمال بدون دلیل**
ناسازگاری.

**۶) بدون `CHECK`**
قیمت منفی.

**۷) نام ستون مبهم**
`data1`, `data2`.

**۸) نرمال‌سازی افراطی**
JOIN زیاد و کندی. تعادل.

### نکات مهم (Tips & Tricks)

- **سه فرم اول** را حفظ کن.
- **۱NF: اتمیک.**
- **۲NF: وابسته به کل کلید.**
- **۳NF: بدون транзитив.**
- **`NUMERIC` برای پول.**
- **`CHECK` و `FK` برای یکپارچگی.**
- **دنرمال آگاهانه، نه تصادفی.**

### خلاصه فصل

- طراحی بد، آنومالی Insert/Update/Delete می‌سازد.
- ۱NF اتمیک، ۲NF وابستگی کامل، ۳NF بدون транзитив.
- دنرمال‌سازی برای سرعت، با هزینهٔ تکرار.
- چهار نوع یکپارچگی: entity، referential، domain، business.
- `NUMERIC` برای پول، `CHECK` برای محدوده.
- نرمال برای OLTP، دنرمال برای OLAP.

---

<a name="ch-64"></a>
## فصل ۶۴: NoSQL، انبار داده و مقیاس‌پذیری

### مقدمه

RDBMS برای ۸۰٪ کارها کافی است. اما ۲۰٪ باقی‌مانده — دادهٔ حجیم، نیمه‌ساختاریافته، یا با بار بسیار بالا — به گزینه‌های دیگر نیاز دارد.

### مفاهیم اساسی

**چهار نوع NoSQL**

| نوع | ساختار | مثال | مناسب |
|---|---|---|---|
| **Document** | JSON | MongoDB | محتوا، کاتالوگ |
| **Key-Value** | کلید-مقدار | Redis | کش، نشست |
| **Column-family** | ستونی | Cassandra | سری زمانی، لاگ |
| **Graph** | گره/یال | Neo4j | شبکهٔ اجتماعی |

**Document DB**

```json
{
  "_id": "abc123",
  "name": "Ali",
  "orders": [
    {"id": 1, "total": 100},
    {"id": 2, "total": 250}
  ]
}
```

| مزیت | عیب |
|---|---|
| انعطاف schema | بدون JOIN قوی |
| طبیعی برای JSON | یکپارچگی دستی |

**Key-Value**

```
SET user:42 "Ali"
GET user:42     → "Ali"
```

| مزیت | عیب |
|---|---|
| بسیار سریع | بدون کوئری پیچیده |
| ساده | بدون ساختار |

**Graph DB**

```
(Ali)-[:FRIEND]->(Sara)-[:WORKS_AT]->(Company)
```

مناسب برای روابط پیچیده: توصیه، تقلب، مسیریابی.

**SQL در برابر NoSQL**

| معیار | SQL | NoSQL |
|---|---|---|
| Schema | ثابت | انعطاف |
| JOIN | ✅ | محدود |
| ACID | ✅ | متغیر (BASE) |
| مقیاس | عمودی | افقی |
| مناسب | تراکنش | حجم/تنوع |

**BASE در برابر ACID**

| ACID | BASE |
|---|---|
| Atomicity | Basically Available |
| Consistency | Soft state |
| Isolation | Eventually consistent |
| Durability | — |

**مقیاس عمودی در برابر افقی**

| نوع | کار | محدودیت |
|---|---|---|
| **Vertical** | سخت‌افزار قوی‌تر | سقف فیزیکی |
| **Horizontal** | سرور بیشتر | پیچیدگی |

**Sharding**

تقسیم داده بین سرورها:

```
shard 1: users 1–1M
shard 2: users 1M–2M
```

**Replication**

کپی داده برای تحمل خرابی و خواندن بیشتر.

| نوع | کار |
|---|---|
| **Primary-Replica** | نوشتن روی primary |
| **Multi-Primary** | نوشتن روی چند |

**انبار داده (Data Warehouse)**

| معیار | OLTP | انبار داده |
|---|---|---|
| هدف | تراکنش | تحلیل |
| Schema | نرمال | Star/Snowflake |
| داده | جاری | تاریخی |
| کوئری | کوتاه | بلند |

**Star Schema**

```
        [Dim Date]
             │
[Dim Customer] ─ [Fact Sales] ─ [Dim Product]
             │
        [Dim Store]
```

| جزء | کار |
|---|---|
| **Fact** | اعداد (مبلغ، تعداد) |
| **Dimension** | توصیف (زمان، مشتری) |

**Data Lake**

ذخیرهٔ دادهٔ خام (ساختاریافته و غیرساختاریافته) بدون schema اولیه.

| مزیت | عیب |
|---|---|
| انعطاف | نیاز به حکمرانی |
| ارزان | «باتلاق داده» |

**ETL در برابر ELT**

| معیار | ETL | ELT |
|---|---|---|
| ترتیب | تبدیل قبل از بارگذاری | بارگذاری قبل از تبدیل |
| مناسب | انبار داده سنتی | Data Lake |

### مثال‌های عملی

**سناریوی Lab: انتخاب پایگاه داده**

**۱) فروشگاه آنلاین**
- تراکنش، موجودی، پرداخت.
- **RDBMS (PostgreSQL).**

**۲) لاگ اپلیکیشن (۱۰۰ میلیون رویداد در روز)**
- نوشتن سنگین، خواندن تحلیلی.
- **Column-family (Cassandra)** یا انبار داده.

**۳) کش صفحه**
- خواندن سریع، عمر کوتاه.
- **Redis.**

**۴) شبکهٔ اجتماعی حرفه‌ای**
- روابط چندلایه.
- **Graph DB** (یا RDBMS + جدول رابطه برای شروع).

**جدول تصمیم**

| اگر… | انتخاب |
|---|---|
| تراکنش و یکپارچگی | RDBMS |
| JSON و schema متغیر | Document |
| سرعت و کش | Key-Value |
| روابط پیچیده | Graph |
| تحلیل تاریخی | انبار داده |

**سناریوی دوم: محاسبهٔ ظرفیت**

```
۱۰۰ میلیون رویداد در روز
× ۱KB هر رویداد
= ۱۰۰ GB در روز
× ۳۶۵ = ۳۶.۵ TB در سال
```

با replication ×۳ = ۱۱۰ TB. این عدد، انتخاب زیرساخت را تعیین می‌کند.

**سناریوی سوم: Sharding**

```python
def shard_for(user_id: int, shards: int = 4) -> int:
    return user_id % shards
```

| مشکل | راه‌حل |
|---|---|
| توزیع نامتوازن | Consistent hashing |
| کوئری بین shardها | ناسازگاری |
| افزودن shard | بازتوزیع |

### مقایسه و الگوهای مختلف

**سه معماری داده**

| معماری | توضیح |
|---|---|
| RDBMS مرکزی | ساده |
| RDBMS + کش | خواندن سریع |
| Polyglot | هر کار، ابزار خودش |

**الگوی «Polyglot Persistence»**

```
تراکنش → PostgreSQL
کش → Redis
جستجو → Elasticsearch
تحلیل → انبار داده
```

مزیت: هر ابزار برای کار خودش. عیب: پیچیدگی عملیاتی.

### اشکال شایع

**۱) NoSQL به‌عنوان مد**
برای تراکنش مالی، RDBMS بهتر است.

**۲) بدون schema در Document DB**
دادهٔ ناسازگار. Validation.

**۳) Sharding زودهنگام**
پیچیدگی قبل از نیاز.

**۴) انبار داده با schema نرمال**
کوئری کند. Star Schema.

**۵) Data Lake بدون حکمرانی**
باتلاق داده.

**۶) بدون replication**
تک‌نقطهٔ خرابی.

**۷) انتخاب بر اساس حجم بدون محاسبه**
ظرفیت را حساب کن.

**۸) نادیده گرفتن هزینهٔ عملیاتی**
هر ابزار جدید، نگه‌داری می‌خواهد.

### نکات مهم (Tips & Tricks)

- **RDBMS پیش‌فرض.**
- **چهار نوع NoSQL** را با مثال حفظ کن.
- **ACID در برابر BASE.**
- **Sharding = مقیاس افقی.**
- **Star Schema برای تحلیل.**
- **Polyglot با احتیاط.**
- **ظرفیت را حساب کن.**

### خلاصه فصل

- NoSQL چهار نوع دارد: document، key-value، column، graph.
- ACID برای تراکنش، BASE برای دسترس‌پذیری.
- مقیاس افقی با sharding و replication.
- انبار داده برای تحلیل، با Star Schema.
- ETL در برابر ELT.
- Polyglot Persistence قدرت و پیچیدگی می‌آورد.

---

## منابع و مراجع

- CompTIA Tech+ exam objectives V6 — دامنهٔ ۵ (Data and database fundamentals): <https://www.comptia.org/en-us/certifications/tech/>
- PostgreSQL Documentation: <https://www.postgresql.org/docs/>
- MySQL Documentation: <https://dev.mysql.com/doc/>
- SQL Standard (ISO/IEC 9075): <https://www.iso.org/standard/76583.html>
- NIST — Database Security: <https://csrc.nist.gov/>
- OWASP — SQL Injection Prevention: <https://owasp.org/www-community/attacks/SQL_Injection>
- Kimball — Data Warehouse Toolkit (Star Schema)
- CAP Theorem (Brewer): <https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed/>

**بعدی:** [فایل ۱۵ — پشتیبان‌گیری و بازیابی](15-Backup-and-Recovery.md)
