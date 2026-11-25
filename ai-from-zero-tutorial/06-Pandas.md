# بخش ۰۶: Pandas و کار با داده

**مدت یادگیری تقریبی**: ۲.۵ ساعت
**سطح سختی**: مبتدی
**پیش‌نیاز**: فایل ۰۵ (NumPy) و فایل ۰۲ (مفاهیم داده)

---

## فهرست محتوا

- [فصل ۲۵: Series و DataFrame: جدول قابل برنامه‌نویسی](#فصل-۲۵-series-و-dataframe-جدول-قابل-برنامهنویسی)
- [فصل ۲۶: خواندن و نوشتن داده](#فصل-۲۶-خواندن-و-نوشتن-داده)
- [فصل ۲۷: انتخاب، فیلتر و مرتب‌سازی](#فصل-۲۷-انتخاب-فیلتر-و-مرتب‌سازی)
- [فصل ۲۸: پاکسازی: مقدار گمشده، تکراری، نوع داده](#فصل-۲۸-پاکسازی-مقدار-گمشده-تکراری-نوع-داده)
- [فصل ۲۹: GroupBy، ادغام و جدول محوری](#فصل-۲۹-groupby-ادغام-و-جدول-محوری)

---

## فصل ۲۵: Series و DataFrame: جدول قابل برنامه‌نویسی

### مقدمه

NumPy آرایه‌ی عددی می‌دهد، اما داده‌ی واقعی **مختلط** است: یک ستون اسم، یکی عدد، یکی تاریخ، یکی خالی. Pandas برای همین ساخته شد: **جدولی که هم برچسب دارد، هم نوع مختلط، هم ابزار پرس‌وجو.**

اگر فقط دو مفهوم از Pandas بدانی، `Series` و `DataFrame` کافی است. بقیه، متدهایی روی همین دو هستند.

### مفاهیم اساسی

**۱. Series.** یک ستون با برچسب (index).

```python
import pandas as pd

s = pd.Series([10, 20, 30], index=["الف", "ب", "ج"], name="امتیاز")
print(s)
# الف    10
# ب      20
# ج      30
print(s["ب"])        # 20
print(s.values)      # array([10, 20, 30])
```

**۲. DataFrame.** جدول دو‌بعدی: مجموعه‌ای از Series که یک index مشترک دارند.

```python
df = pd.DataFrame({
    "name":  ["علی", "مریم", "رضا"],
    "age":   [28, 34, 41],
    "city":  ["تهران", "شیراز", "تهران"],
    "score": [85.5, 91.0, 78.5],
})

print(df)
print(df.shape)        # (3, 4)
print(df.columns)      # Index(['name', 'age', 'city', 'score'])
print(df.dtypes)       # نوع هر ستون
```

**۳. index — چیزی که Pandas را از NumPy جدا می‌کند.**

```python
df2 = df.set_index("name")
print(df2.loc["مریم"])       # دسترسی با برچسب
print(df2.loc["مریم", "score"])   # 91.0

df3 = df.reset_index()       # برگشت به index عددی
```

**نکته‌ی مهم:** index **خودکار هم‌تراز** می‌شود. اگر دو Series با index متفاوت جمع کنی، Pandas بر اساس برچسب جمع می‌زند، نه ترتیب. این هم قدرت است، هم منبع باگ.

**۴. `loc` در برابر `iloc` — تفاوت حیاتی.**

| | مبنا | مثال | نتیجه |
|---|---|---|---|
| `loc` | برچسب | `df.loc[0:2]` | ردیف‌های ۰ تا ۲ **شامل** ۲ |
| `iloc` | موقعیت عددی | `df.iloc[0:2]` | ردیف‌های ۰ و ۱ (**بدون** ۲) |

این تفاوت (شامل بودن در `loc`) یکی از رایج‌ترین منابع خطاست.

**۵. ساخت DataFrame از منابع مختلف.**

```python
# از دیکشنری لیست‌ها
pd.DataFrame({"a": [1, 2], "b": [3, 4]})

# از لیست دیکشنری‌ها (شبیه JSON)
pd.DataFrame([{"a": 1, "b": 3}, {"a": 2, "b": 4}])

# از آرایه‌ی NumPy
import numpy as np
pd.DataFrame(np.random.randn(3, 2), columns=["x", "y"])

# از فایل
# pd.read_csv("data.csv")
```

**۶. اطلاعات کلی — پنج دستور که همیشه اول می‌زنی.**

```python
df.head()          # ۵ ردیف اول
df.info()          # نوع ستون‌ها + تعداد غیرخالی
df.describe()      # آمار توصیفی ستون‌های عددی
df.isna().sum()    # تعداد گمشده در هر ستون
df["city"].value_counts()   # فراوانی مقادیر یکتا
```

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — ساخت و کاوش یک دیتاست کوچک:**

```python
import pandas as pd

df = pd.DataFrame({
    "order_id": [1, 2, 3, 4, 5, 6],
    "customer": ["علی", "مریم", "علی", "سارا", "رضا", "مریم"],
    "amount":   [120_000, 85_000, 240_000, 60_000, 310_000, 95_000],
    "city":     ["تهران", "شیراز", "تهران", "مشهد", "تهران", "شیراز"],
    "date":     pd.to_datetime(["2024-01-05", "2024-01-07", "2024-02-11",
                                "2024-02-14", "2024-03-01", "2024-03-09"]),
})

print(df.info())
print("\n", df.describe())
print("\nشهرها:\n", df["city"].value_counts())
```

**مثال ۲ — تفاوت loc و iloc:**

```python
df = pd.DataFrame({"a": [10, 20, 30], "b": [1, 2, 3]},
                  index=["x", "y", "z"])

print(df.loc["x":"y"])      # x و y (شامل)
print(df.iloc[0:2])         # x و y (بدون z)
print(df.iloc[1, 0])        # 20
print(df.loc["y", "a"])     # 20
```

**مثال ۳ — هم‌ترازی خودکار index (قدرت و خطر):**

```python
s1 = pd.Series([1, 2, 3], index=["a", "b", "c"])
s2 = pd.Series([10, 20], index=["b", "c"])

print(s1 + s2)
# a     NaN   ← در s2 نبود
# b    12.0
# c    23.0
```

اگر با NumPy کار می‌کردی، خطا می‌گرفتی. Pandas بی‌صدا `NaN` می‌گذارد. **همیشه بعد از عملیات، `isna()` را چک کن.**

### مقایسه و دیدگاه‌های مختلف

| ابزار | مزیت | عیب | کِی |
|---|---|---|---|
| `list`/`dict` | بدون وابستگی | بدون ابزار تحلیل | داده‌ی کوچک |
| `np.array` | سریع، عددی | بدون برچسب، هم‌نوع | محاسبات عددی |
| `pd.DataFrame` | برچسب، مختلط، ابزار غنی | کندتر، حافظه‌بر | داده‌ی جدولی |
| `polars` | خیلی سریع، موازی | اکوسیستم کوچک‌تر | داده‌ی بزرگ |

**یک دیدگاه عملی:** برای داده‌ی زیر ۱ میلیون ردیف، Pandas بی‌رقیب است. بالای ۱۰ میلیون، به Polars یا Dask فکر کن.

### اشتباهات رایج

**۱. `df["col"][0]` برای دسترسی.** از `df.loc[0, "col"]` استفاده کن؛ اولی ممکن است کپی بدهد و `SettingWithCopyWarning` بدهد.
**۲. `df.loc[0:5]` به انتظار ۵ ردیف.** ۶ ردیف می‌دهد (شامل).
**۳. `inplace=True` همه‌جا.** در Pandas 2+ بسیاری از `inplace`ها deprecated شده‌اند و رفتار غیرقابل پیش‌بینی دارند. `df = df.dropna()` بنویس.
**۴. index را جدی نگرفتن.** بعد از `concat` یا `merge`، index تکراری می‌شود و `.loc` رفتار عجیب می‌دهد.
**۵. مقایسه‌ی DataFrame با `==`.** خروجی DataFrame بولی است، نه `True/False`.

### نکات مهم و ترفندها

- **همیشه `df.info()` بزن.** یک خط، هم نوع‌ها را می‌گوید هم گمشده‌ها.
- **`df.head()` و `df.sample(5)`** — دومی برای دیدن ردیف‌های تصادفی بهتر است (ممکن است داده مرتب باشد).
- **`pd.set_option("display.max_columns", None)`** برای دیدن همه‌ی ستون‌ها در نوت‌بوک.
- **`df.memory_usage(deep=True).sum() / 1e6`** — چند مگابایت است؟ اگر زیاد است، `dtype` را کوچک کن.
- **میان‌بر:** DataFrame = جدول با برچسب · `loc` برچسبی · `iloc` عددی · `info()` اولین قدم.

### خلاصه فصل

- `Series` یک ستون برچسب‌دار، `DataFrame` یک جدول است.
- index، برچسب ردیف‌هاست و عملیات را هم‌تراز می‌کند.
- `loc` بر اساس برچسب (شامل انتها)، `iloc` بر اساس موقعیت (بدون انتها).
- `info`, `describe`, `isna().sum()`, `value_counts()` پنج دستور اول هر تحلیل‌اند.
- هم‌ترازی index می‌تواند بی‌صدا `NaN` بسازد.

**سؤالات مروری:**
1. تفاوت `df.loc[0:3]` و `df.iloc[0:3]` چیست؟
2. چرا `df["col"][0]` توصیه نمی‌شود؟
3. اگر دو Series با index متفاوت جمع شوند چه اتفاقی می‌افتد؟

---

## فصل ۲۶: خواندن و نوشتن داده

### مقدمه

هر پروژه با `read_csv` شروع می‌شود. اما همان خط اول، پر از تله است: کدگذاری، جداکننده، ستون index، نوع داده، فایل بزرگ. این فصل، آن خط را از «کار می‌کند» به «درست کار می‌کند» می‌برد.

### مفاهیم اساسی

**۱. `read_csv` با پارامترهای مهم.**

```python
import pandas as pd

df = pd.read_csv(
    "data.csv",
    encoding="utf-8",          # برای فارسی حیاتی
    sep=",",                   # جداکننده (گاهی ";" یا "\t")
    header=0,                  # ردیف عنوان
    index_col=None,            # کدام ستون index باشد
    usecols=["age", "city"],   # فقط ستون‌های لازم (حافظه)
    dtype={"age": "int32"},    # نوع صریح
    na_values=["", "NA", "-", "?"],   # چه چیزهایی NaN باشند
    nrows=1000,                # فقط ۱۰۰۰ ردیف (برای آزمایش)
)
```

**چرا `dtype` صریح مهم است؟** چون Pandas گاهی ستون عددی را `object` می‌خواند (مثلاً اگر یک مقدار متنی داخلش باشد) و بعد محاسباتت می‌شکند.

**۲. خواندن فایل‌های اکسل.**

```python
# نیاز به: pip install openpyxl
df = pd.read_excel("report.xlsx", sheet_name="Sheet1", engine="openpyxl")

# همه‌ی شیت‌ها
sheets = pd.read_excel("report.xlsx", sheet_name=None)   # dict از DataFrame
```

**۳. خواندن JSON.**

```python
# JSON ساده
df = pd.read_json("data.json")

# JSON تودرتو — نیاز به نرمال‌سازی
import json
with open("nested.json", encoding="utf-8") as f:
    raw = json.load(f)
df = pd.json_normalize(raw, sep="_")     # کلیدهای تودرتو → ستون‌های مسطح
```

**۴. خواندن از دیتابیس.**

```python
import sqlite3

conn = sqlite3.connect("shop.db")
df = pd.read_sql_query("SELECT * FROM orders WHERE amount > 1000", conn)
conn.close()
```

**۵. خواندن فایل بزرگ با chunking.**

```python
total_rows = 0
chunks = []
for chunk in pd.read_csv("big.csv", chunksize=200_000, encoding="utf-8"):
    # فیلتر یا تجمیع در هر تکه
    filtered = chunk[chunk["amount"] > 0]
    chunks.append(filtered.groupby("city")["amount"].sum())

result = pd.concat(chunks).groupby(level=0).sum()
print(result)
```

**۶. نوشتن.**

```python
df.to_csv("out.csv", index=False, encoding="utf-8-sig")   # utf-8-sig برای اکسل ویندوز
df.to_excel("out.xlsx", index=False)
df.to_json("out.json", orient="records", force_ascii=False, indent=2)
df.to_parquet("out.parquet")        # نیاز به pyarrow — سریع و فشرده
```

**نکته‌ی مهم درباره‌ی `utf-8-sig`:** اکسل ویندوز، UTF-8 بدون BOM را درست نمی‌خواند و فارسی را خراب نشان می‌دهد. `utf-8-sig` این مشکل را حل می‌کند.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — خواندن مقاوم با تشخیص کدگذاری:**

```python
import pandas as pd
from pathlib import Path

def read_any_csv(path, **kwargs):
    """تلاش با چند کدگذاری — الگوی عملی برای فایل‌های ناشناس."""
    p = Path(path)
    if not p.exists():
        raise FileNotFoundError(f"فایل نیست: {p.resolve()}")

    for enc in ["utf-8", "utf-8-sig", "cp1256", "latin-1"]:
        try:
            df = pd.read_csv(p, encoding=enc, **kwargs)
            print(f"✅ خوانده شد با {enc} — شکل: {df.shape}")
            return df
        except (UnicodeDecodeError, UnicodeError):
            continue
    raise ValueError("هیچ کدگذاری‌ای جواب نداد")

# df = read_any_csv("data.csv")
```

**مثال ۲ — ذخیره و بازخوانی با حفظ نوع‌ها:**

```python
import pandas as pd

df = pd.DataFrame({
    "id": [1, 2, 3],
    "name": ["علی", "مریم", "رضا"],
    "date": pd.to_datetime(["2024-01-01", "2024-02-01", "2024-03-01"]),
    "amount": [1200.5, 800.0, 3000.25],
})

# CSV نوع‌ها را حفظ نمی‌کند
df.to_csv("tmp.csv", index=False, encoding="utf-8-sig")
back = pd.read_csv("tmp.csv")
print(back.dtypes)      # date → object! (رشته)

# Parquet نوع‌ها را حفظ می‌کند
df.to_parquet("tmp.parquet")
back2 = pd.read_parquet("tmp.parquet")
print(back2.dtypes)     # date → datetime64[ns] ✅
```

**درس:** برای ذخیره‌ی میانی، Parquet بهتر است — سریع‌تر، کوچک‌تر، نوع‌ها حفظ می‌شوند.

**مثال ۳ — تبدیل فرمت و کاهش حجم:**

```python
import pandas as pd

df = pd.read_csv("big.csv", encoding="utf-8")

# کاهش حافظه با dtype مناسب
df["age"] = df["age"].astype("int8")            # ۱ بایت به جای ۸
df["city"] = df["city"].astype("category")      # رشته‌های تکراری → کد

print(f"حافظه: {df.memory_usage(deep=True).sum() / 1e6:.1f} MB")
df.to_parquet("big.parquet", compression="snappy")
```

**`category`** برای ستون‌های با مقادیر تکراری (مثل شهر، وضعیت) می‌تواند حافظه را ۱۰ برابر کم کند.

### مقایسه و دیدگاه‌های مختلف

| فرمت | سرعت خواندن | حجم | حفظ نوع | خوانا |
|---|---|---|---|---|
| CSV | کند | بزرگ | ❌ | ✅ |
| Excel | خیلی کند | بزرگ | متوسط | ✅ |
| JSON | متوسط | بزرگ | متوسط | ✅ |
| Parquet | خیلی سریع | کوچک | ✅ | ❌ |
| Feather | خیلی سریع | کوچک | ✅ | ❌ |

**توصیه‌ی عملی:**
- **تبادل با انسان** → CSV/Excel
- **ذخیره‌ی میانی و pipeline** → Parquet
- **API و تنظیمات** → JSON

### اشتباهات رایج

**۱. فراموش کردن `encoding`.** فارسی خراب یا `UnicodeDecodeError`.
**۲. `index=False` ننوشتن در `to_csv`.** یک ستون `Unnamed: 0` اضافه به فایل بعدی می‌رود.
**۳. باز کردن فایل ۵ گیگابایتی با `read_csv` یک‌جا.** `MemoryError`. از `chunksize` یا `usecols` استفاده کن.
**۴. اعتماد به تشخیص خودکار نوع.** Pandas ممکن است کد پستی `01234` را `1234` بخواند (صفر از دست می‌رود). `dtype={"zip": str}` بگذار.
**۵. `to_excel` روی فایل باز.** در ویندوز `PermissionError`. فایل را ببند.
**۶. نوشتن روی فایل اصلی داده.** همیشه روی نسخه‌ی جدید بنویس.

### نکات مهم و ترفندها

- **`usecols` و `nrows` را زودتر بگذار.** سرعت کاوش اولیه را چند برابر می‌کند.
- **`na_values` را صریح بده.** وگرنه `"?"` و `"-"` به‌عنوان رشته می‌مانند و محاسبات را خراب می‌کنند.
- **`utf-8-sig` برای فایل‌هایی که در اکسل باز می‌شوند.** یک عادت طلایی.
- **`pd.read_parquet` را جایگزین CSV کن** در pipelineهای تکراری. ۱۰ برابر سریع‌تر.
- **میان‌بر:** خواندن = `encoding` + `dtype` + `usecols` · نوشتن = `index=False` + `utf-8-sig`.

### خلاصه فصل

- `encoding="utf-8"` (یا `utf-8-sig`) برای فارسی ضروری است.
- `dtype` و `na_values` را صریح بده تا نوع‌ها و مقادیر گمشده درست خوانده شوند.
- فایل‌های بزرگ را با `chunksize` پردازش کن.
- Parquet برای ذخیره‌ی میانی بهترین است: سریع، فشرده، با حفظ نوع.
- `index=False` در نوشتن CSV فراموش نشود.

**سؤالات مروری:**
1. چرا `utf-8-sig` برای اکسل ویندوز لازم است؟
2. چه زمانی `chunksize` لازم می‌شود؟
3. چرا Parquet از CSV برای pipeline بهتر است؟

---

## فصل ۲۷: انتخاب، فیلتر و مرتب‌سازی

### مقدمه

این فصل، «SQL ذهنی» تو در Pandas است. تقریباً هر سؤالی که از داده داری، به یکی از این شکل‌هاست: «کدام ردیف‌ها؟»، «کدام ستون‌ها؟»، «به چه ترتیبی؟». این فصل جواب هر سه را می‌دهد.

### مفاهیم اساسی

**۱. انتخاب ستون.**

```python
df["age"]                  # Series
df[["age", "city"]]        # DataFrame — دو براکت!
df.loc[:, "age":"city"]    # با برش برچسبی
```

**۲. فیلتر با شرط.**

```python
df[df["age"] > 30]
df[(df["age"] > 30) & (df["city"] == "تهران")]     # و
df[(df["city"] == "تهران") | (df["city"] == "شیراز")]  # یا
df[~(df["city"] == "تهران")]                        # نقیض
```

**قواعد حیاتی:**
- از `&` و `|` استفاده کن، نه `and` و `or`.
- هر شرط را در پرانتز بگذار.
- برای «هست در لیست»: `df["city"].isin(["تهران", "شیراز"])`
- برای «نیست»: `~df["city"].isin([...])`

**۳. فیلتر با `query` — خواناتر.**

```python
df.query("age > 30 and city == 'تهران'")
df.query("amount > @threshold")     # @ برای متغیر پایتون
```

**۴. `loc` و `iloc` برای انتخاب هم‌زمان ردیف و ستون.**

```python
df.loc[df["age"] > 30, ["name", "age"]]
df.iloc[0:5, 0:3]
df.loc[2, "city"]
```

**۵. مرتب‌سازی.**

```python
df.sort_values("age")                        # صعودی
df.sort_values("age", ascending=False)       # نزولی
df.sort_values(["city", "age"], ascending=[True, False])
df.sort_values("age", na_position="first")   # NaN اول
```

**۶. `nlargest` و `nsmallest` — سریع‌تر از sort + head.**

```python
df.nlargest(3, "amount")
df.nsmallest(3, "amount")
```

**۷. انتخاب با نوع داده.**

```python
df.select_dtypes(include="number")     # فقط عددی
df.select_dtypes(include="object")     # فقط متنی
df.select_dtypes(exclude="number")
```

**۸. نمونه‌گیری تصادفی.**

```python
df.sample(5)                    # ۵ ردیف تصادفی
df.sample(frac=0.1, random_state=42)   # ۱۰٪
```

**۹. `where` و `mask` — جایگزینی شرطی.**

```python
df["age"].where(df["age"] > 0, other=0)     # اگر شرط False، ۰
df["age"].mask(df["age"] < 0, other=0)      # اگر شرط True، ۰
```

**۱۰. `np.where` برداری برای ستون جدید:**

```python
import numpy as np
df["is_adult"] = np.where(df["age"] >= 18, 1, 0)
```

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — یک تحلیل واقعی در ۶ خط:**

```python
import pandas as pd

df = pd.DataFrame({
    "order_id": range(1, 9),
    "customer": ["علی", "مریم", "علی", "سارا", "رضا", "مریم", "علی", "سارا"],
    "amount":   [120_000, 85_000, 240_000, 60_000, 310_000, 95_000, 150_000, 72_000],
    "city":     ["تهران", "شیراز", "تهران", "مشهد", "تهران", "شیراز", "تهران", "مشهد"],
    "status":   ["paid", "paid", "cancelled", "paid", "paid", "cancelled", "paid", "paid"],
})

# ۱. سفارش‌های پرداخت‌شده‌ی تهران
tehran_paid = df.query("city == 'تهران' and status == 'paid'")
print(tehran_paid[["customer", "amount"]])

# ۲. سه سفارش بزرگ
print(df.nlargest(3, "amount")[["order_id", "amount"]])

# ۳. سفارش‌های بین ۸۰ تا ۲۰۰ هزار
mid = df[df["amount"].between(80_000, 200_000)]
print("تعداد:", len(mid))
```

**مثال ۲ — ساختن ستون‌های جدید با شرط:**

```python
import numpy as np

df["is_large"] = df["amount"] > 100_000
df["size_label"] = np.select(
    [df["amount"] < 80_000, df["amount"] < 200_000],
    ["small", "medium"],
    default="large",
)
print(df[["amount", "size_label"]])
```

`np.select` برای «چند شرط پشت سر هم» خیلی خواناتر از `apply` است.

**مثال ۳ — فیلتر با تاریخ:**

```python
df["date"] = pd.to_datetime(df["date"])

mask = (df["date"] >= "2024-02-01") & (df["date"] < "2024-03-01")
feb = df[mask]

# یا با dt accessor
feb = df[df["date"].dt.month == 2]
```

### مقایسه و دیدگاه‌های مختلف

| روش | خوانایی | سرعت | کِی |
|---|---|---|---|
| `df[mask]` | متوسط | سریع | شرط ساده |
| `df.query()` | بالا | سریع | شرط پیچیده، نام ستون ساده |
| `df.loc[mask, cols]` | بالا | سریع | هم‌زمان ردیف و ستون |
| `df.eval()` | بالا | خیلی سریع | محاسبه‌ی عبارت روی ستون‌ها |
| `apply` | بالا | کند | منطق پیچیده‌ی ردیفی |

**قاعده:** تا جای ممکن از `apply` پرهیز کن. اگر مجبور شدی، بدان که ۱۰ تا ۱۰۰ برابر کندتر از عملیات برداری است.

### اشتباهات رایج

**۱. `and` به جای `&`.** `ValueError: The truth value of a Series is ambiguous`.
**۲. نگذاشتن پرانتز.** `df["a"] > 1 & df["b"] < 2` اشتباه تفسیر می‌شود.
**۳. `df[df["age"] > 30]["name"] = "x"`.** زنجیره‌ی index → `SettingWithCopyWarning` و احتمالاً تغییر نکردن. از `df.loc[mask, "name"] = "x"` استفاده کن.
**۴. `df.drop(inplace=True)` روی view.** رفتار غیرقابل پیش‌بینی.
**۵. مقایسه‌ی رشته با فاصله.** `"تهران "` با `"تهران"` برابر نیست. اول `.str.strip()` بزن.
**۶. فیلتر روی ستون `object` که عدد است.** اول `pd.to_numeric(errors="coerce")`.

### نکات مهم و ترفندها

- **`between`, `isin`, `isna`, `duplicated`** را حفظ کن. چهار متد که نصف فیلترها را حل می‌کنند.
- **`query` با `@`** متغیر پایتون را وارد می‌کند — برای توابع پارامتری عالی است.
- **`df.filter(like="amount")`** برای انتخاب ستون‌های با نام مشابه.
- **`np.select` را جای `apply` با چند `if` بگذار.** هم خواناتر، هم سریع‌تر.
- **میان‌بر:** شرط‌ها را با `&`/`|` و پرانتز بنویس · انتخاب هم‌زمان با `loc` · مرتب‌سازی با `sort_values`.

### خلاصه فصل

- `df[["a","b"]]` ستون، `df[mask]` ردیف.
- `&` و `|` با پرانتز، برای شرط‌های چندگانه.
- `loc` برای انتخاب هم‌زمان ردیف و ستون؛ `query` برای خوانایی.
- `nlargest`/`nsmallest` سریع‌تر از `sort_values().head()`.
- `apply` آخرین گزینه است، نه اولین.

**سؤالات مروری:**
1. چرا `df["a"] > 1 and df["b"] < 2` خطا می‌دهد؟
2. تفاوت `df[mask]["col"] = x` و `df.loc[mask, "col"] = x` چیست؟
3. کِی از `query` و کِی از ماسک بولی استفاده می‌کنی؟

---

## فصل ۲۸: پاکسازی: مقدار گمشده، تکراری، نوع داده

### مقدمه

این فصل، پرکاربردترین فصل Pandas در عمل است. طبق تجربه، **۵۰ تا ۷۰ درصد** وقت یک پروژه‌ی داده، صرف همین کارها می‌شود. اگر این فصل را مسلط شوی، سریع‌تر از ۹۰٪ همکارانت به مدل‌سازی می‌رسی.

### مفاهیم اساسی

**۱. شناسایی مقدار گمشده.**

```python
df.isna()                 # DataFrame بولی
df.isna().sum()           # تعداد هر ستون
df.isna().mean() * 100    # درصد هر ستون
df.isna().any(axis=1)     # ردیف‌هایی که حداقل یک NaN دارند
```

**۲. حذف.**

```python
df.dropna()                                  # هر ردیف با NaN
df.dropna(axis=1)                            # هر ستون با NaN
df.dropna(thresh=len(df) * 0.5, axis=1)      # ستون‌هایی که کمتر از ۵۰٪ NaN دارند
df.dropna(subset=["age", "income"])          # فقط بر اساس ستون‌های کلیدی
```

**۳. پرکردن.**

```python
df["age"].fillna(df["age"].median())
df["city"].fillna("نامشخص")
df["income"].fillna(df["income"].mean())

# پرکردن با مقدار قبلی/بعدی (برای سری زمانی)
df["price"].ffill()      # forward fill
df["price"].bfill()      # backward fill

# پرکردن گروهی (بهترین روش ساده)
df["income"] = df.groupby("city")["income"].transform(
    lambda s: s.fillna(s.median())
)
```

**مثال گروهی بالا را جدی بگیر:** پر کردن با میانه‌ی **شهر**، از میانه‌ی کل داده دقیق‌تر است، چون درآمد در شهرهای مختلف فرق دارد.

**۴. نشانگر گمشدگی — ترفند حرفه‌ای.**

```python
df["income_missing"] = df["income"].isna().astype(int)
df["income"] = df["income"].fillna(df["income"].median())
```

گاهی خودِ «گم‌بودن» سیگنال است. مثلاً مشتریانی که درآمدشان را وارد نمی‌کنند، ممکن است رفتار متفاوتی داشته باشند.

**۵. تکراری‌ها.**

```python
df.duplicated().sum()                    # تعداد ردیف‌های تکراری
df.duplicated(subset=["customer", "date"]).sum()   # تکراری بر اساس کلید
df.drop_duplicates()
df.drop_duplicates(subset=["order_id"], keep="last")   # نگه‌داشتن آخرین
```

**۶. نوع داده.**

```python
df.dtypes
df["age"] = df["age"].astype("int32")
df["date"] = pd.to_datetime(df["date"], errors="coerce")     # نامعتبر → NaT
df["amount"] = pd.to_numeric(df["amount"], errors="coerce")  # نامعتبر → NaN
df["city"] = df["city"].astype("category")
```

**`errors="coerce"`** کلید کار با داده‌ی کثیف است: هر چیزی که تبدیل نشود، `NaN` می‌شود و بعد می‌توانی بشماری.

**۷. پاکسازی متن.**

```python
df["city"] = (df["city"]
    .str.strip()                     # حذف فاصله
    .str.replace("\u200c", " ", regex=False)   # نیم‌فاصله
    .str.replace("ي", "ی")           # یکسان‌سازی عربی/فارسی
    .str.replace("ك", "ک")
    .str.lower()                     # برای انگلیسی
)
```

**این تکه کد برای داده‌ی فارسی حیاتی است.** «تهران» و «تهران » و «تهران» (با نیم‌فاصله) سه مقدار متفاوت در `value_counts` می‌شوند.

**۸. مقادیر پرت.**

```python
Q1, Q3 = df["amount"].quantile([0.25, 0.75])
IQR = Q3 - Q1
lower, upper = Q1 - 1.5 * IQR, Q3 + 1.5 * IQR

df["amount_clipped"] = df["amount"].clip(lower, upper)     # محدود کردن (نه حذف)
outliers = df[(df["amount"] < lower) | (df["amount"] > upper)]
```

**`clip` بهتر از حذف است:** داده را از دست نمی‌دهی، فقط اثر پرت را محدود می‌کنی.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — یک تابع پاکسازی کامل:**

```python
import pandas as pd
import numpy as np

def clean_dataframe(df):
    """پاکسازی استاندارد — گزارش هر مرحله."""
    report = {}
    df = df.copy()
    report["rows_start"] = len(df)

    # ۱. حذف تکراری
    before = len(df)
    df = df.drop_duplicates()
    report["duplicates_removed"] = before - len(df)

    # ۲. یکسان‌سازی متن
    for col in df.select_dtypes(include="object").columns:
        df[col] = df[col].str.strip()

    # ۳. تبدیل عددی
    for col in ["amount", "age"]:
        if col in df.columns:
            df[col] = pd.to_numeric(df[col], errors="coerce")

    # ۴. حذف ستون‌های بیش از ۵۰٪ خالی
    thresh = len(df) * 0.5
    drop_cols = df.columns[df.isna().sum() > thresh].tolist()
    df = df.drop(columns=drop_cols)
    report["columns_dropped"] = drop_cols

    # ۵. پرکردن
    for col in df.select_dtypes(include="number").columns:
        df[col] = df[col].fillna(df[col].median())
    for col in df.select_dtypes(include="object").columns:
        df[col] = df[col].fillna("نامشخص")

    report["rows_end"] = len(df)
    report["missing_left"] = int(df.isna().sum().sum())
    return df, report

df_raw = pd.DataFrame({
    "name":   ["علی", "مریم ", "علی", None, "رضا"],
    "age":    ["28", "34", "28", "نامعلوم", "41"],
    "amount": [1200, None, 1200, 800, 3000],
})

df_clean, rep = clean_dataframe(df_raw)
print(rep)
print(df_clean)
```

**مثال ۲ — مقایسه‌ی استراتژی‌های پرکردن:**

```python
import numpy as np
import pandas as pd

rng = np.random.default_rng(0)
df = pd.DataFrame({
    "city": rng.choice(["تهران", "شیراز", "مشهد"], 1000),
})
df["income"] = np.where(df["city"] == "تهران", 20_000, 10_000) + rng.normal(0, 2000, 1000)
df.loc[rng.choice(1000, 200, replace=False), "income"] = np.nan

global_median = df["income"].median()
group_median = df.groupby("city")["income"].transform("median")

err_global = (df["income"].fillna(global_median) - group_median).abs().mean()
err_group = (df["income"].fillna(group_median) - group_median).abs().mean()

print(f"خطای پرکردن با میانه‌ی کل: {err_global:.0f}")
print(f"خطای پرکردن گروهی:        {err_group:.0f}")
```

**مثال ۳ — تشخیص تکراری‌های ظریف:**

```python
df = pd.DataFrame({
    "email": ["a@x.com", "A@X.com", "a@x.com ", "b@x.com"],
    "amount": [100, 100, 100, 200],
})

# تکراری خام
print(df.duplicated().sum())        # 1

# تکراری پس از نرمال‌سازی
df["email_norm"] = df["email"].str.strip().str.lower()
print(df.duplicated(subset=["email_norm"]).sum())    # 2
```

**درس:** تکراری‌یابی بدون نرمال‌سازی، نصف تکراری‌ها را از دست می‌دهد.

### مقایسه و دیدگاه‌های مختلف

| استراتژی گمشده | مزیت | عیب | کِی |
|---|---|---|---|
| حذف ردیف | ساده، بدون فرض | از دست دادن داده | گمشدگی کم و تصادفی |
| میانگین/میانه | سریع | واریانس را کم می‌کند | عددی، گمشدگی کم |
| گروهی | دقیق‌تر | نیاز به ستون گروه | داده‌ی خوشه‌ای |
| KNN/Iterative | دقیق‌ترین | کند، پیچیده | داده‌ی مهم |
| نشانگر + پرکردن | اطلاعات گمشدگی را نگه می‌دارد | یک ستون بیشتر | همیشه (توصیه‌شده) |

**یک هشدار:** هر روش پرکردن، **واریانس را مصنوعاً کم می‌کند** و مدل را بیش‌اطمینان می‌سازد. اگر درصد گمشدگی بالاست، به‌جای پرکردن، به فکر حذف ستون یا جمع‌آوری داده باش.

### اشتباهات رایج

**۱. پرکردن با میانگین وقتی توزیع چوله است.** میانه بهتر است.
**۲. `fillna(0)` روی درآمد.** صفر یک مقدار معنادار است، نه «نمی‌دانم».
**۳. پرکردن قبل از تقسیم train/test.** نشت داده. اول تقسیم، بعد پرکردن با آمار train.
**۴. `dropna()` بدون بررسی.** ممکن است ۸۰٪ داده را از دست بدهی و متوجه نشوی.
**۵. `astype(int)` روی ستون با NaN.** `ValueError`. اول NaN را مدیریت کن، یا از `Int64` (با I بزرگ) استفاده کن.
**۶. نرمال‌نکردن متن فارسی.** «تهران» و «تهران » دو مقدار متفاوت می‌شوند.
**۷. حذف همه‌ی پرت‌ها.** گاهی پرت همان سیگنال است.

### نکات مهم و ترفندها

- **همیشه قبل و بعد، گزارش بگیر:** تعداد ردیف، درصد گمشدگی، تعداد تکراری. پاکسازی بدون گزارش، غیرقابل بازبینی است.
- **`clip` بهتر از حذف پرت است.** داده را نگه می‌دارد.
- **`transform` برای پرکردن گروهی.** `groupby().transform()` همان index را نگه می‌دارد.
- **`Int64` (با I بزرگ) نوع صحیح با پشتیبانی NaN است.** برای ستون‌های شمارشی با گمشده.
- **میان‌بر:** شناسایی (`isna`) → تصمیم (حذف/پرکردن) → گزارش. همیشه به همین ترتیب.

### خلاصه فصل

- `isna().sum()` و `isna().mean()` نقطه‌ی شروع هر پاکسازی است.
- پرکردن گروهی از پرکردن کلی دقیق‌تر است.
- نشانگر گمشدگی، اطلاعات را حفظ می‌کند.
- نرمال‌سازی متن فارسی (فاصله، نیم‌فاصله، ی/ك) واجب است.
- `clip` جای حذف پرت را می‌گیرد.

**سؤالات مروری:**
1. چرا پرکردن با میانه‌ی گروه بهتر از میانه‌ی کل است؟
2. چرا `fillna(0)` برای درآمد غلط است؟
3. چگونه تکراری‌های «ظریف» (با فاصله و حروف بزرگ) را پیدا می‌کنی؟

---

## فصل ۲۹: GroupBy، ادغام و جدول محوری

### مقدمه

اگر فصل قبل «پاکسازی» بود، این فصل «تحلیل» است. سه ابزار اصلی: `groupby` (تجمیع بر اساس گروه)، `merge` (ترکیب جدول‌ها، مثل JOIN در SQL)، و `pivot_table` (جدول محوری، مثل اکسل).

این سه، ۹۰٪ تحلیل‌های اکتشافی و بخش بزرگی از مهندسی ویژگی را می‌سازند.

### مفاهیم اساسی

**۱. GroupBy — «تقسیم، اعمال، ترکیب».**

```python
df.groupby("city")["amount"].sum()
df.groupby("city")["amount"].mean()
df.groupby("city").agg({"amount": ["sum", "mean", "count"]})
```

منطق سه‌مرحله‌ای:
1. **Split:** داده بر اساس کلید گروه‌بندی می‌شود.
2. **Apply:** تابع روی هر گروه اجرا می‌شود.
3. **Combine:** نتایج در یک جدول ترکیب می‌شوند.

**۲. تجمیع چندگانه.**

```python
df.groupby("city").agg(
    total=("amount", "sum"),
    avg=("amount", "mean"),
    n=("amount", "count"),
    max_amount=("amount", "max"),
)
```

این سبک (named aggregation) خروجی با نام ستون تمیز می‌دهد.

**۳. گروه‌بندی چندکلیدی.**

```python
df.groupby(["city", "status"])["amount"].sum()
df.groupby(["city", "status"])["amount"].sum().unstack()   # جدول دو‌بعدی
```

**۴. `transform` در برابر `agg` — تفاوت مهم.**

```python
# agg: یک مقدار برای هر گروه (تعداد ردیف کم می‌شود)
df.groupby("city")["amount"].agg("mean")     # یک ردیف برای هر شهر

# transform: یک مقدار برای هر ردیف (هم‌اندازه‌ی داده)
df["city_avg"] = df.groupby("city")["amount"].transform("mean")
df["diff_from_city_avg"] = df["amount"] - df["city_avg"]
```

**`transform` برای مهندسی ویژگی حیاتی است:** «این سفارش چقدر از میانگین شهرش بیشتر است؟» — یک ویژگی قدرتمند.

**۵. `merge` — JOIN.**

```python
pd.merge(orders, customers, on="customer_id", how="inner")
```

| نوع | نتیجه |
|---|---|
| `inner` | فقط کلیدهای مشترک |
| `left` | همه‌ی چپ + منطبق‌های راست |
| `right` | همه‌ی راست + منطبق‌های چپ |
| `outer` | همه‌ی هر دو |

**۶. `merge` با نام ستون متفاوت.**

```python
pd.merge(orders, customers, left_on="cust_id", right_on="id", how="left")
```

**۷. بررسی نتیجه‌ی merge (حیاتی):**

```python
merged = pd.merge(a, b, on="id", how="left", indicator=True)
print(merged["_merge"].value_counts())
# both          900
# left_only     100   ← در b نبود
```

`indicator=True` یک ستون `_merge` اضافه می‌کند که می‌گوید هر ردیف از کجا آمده. **همیشه بعد از merge این را چک کن** — وگرنه ممکن است بی‌صدا ردیف اضافه یا کم شده باشد.

**۸. `concat` — چسباندن.**

```python
pd.concat([df1, df2], axis=0, ignore_index=True)     # عمودی (ردیف)
pd.concat([df1, df2], axis=1)                        # افقی (ستون)
```

**۹. `pivot_table` — جدول محوری.**

```python
df.pivot_table(
    index="city",
    columns="status",
    values="amount",
    aggfunc="sum",
    fill_value=0,
    margins=True,        # جمع کل
)
```

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — تحلیل فروش کامل:**

```python
import pandas as pd

df = pd.DataFrame({
    "order_id": range(1, 11),
    "customer": ["علی", "مریم", "علی", "سارا", "رضا", "مریم", "علی", "سارا", "رضا", "مریم"],
    "city":     ["تهران", "شیراز", "تهران", "مشهد", "تهران", "شیراز", "تهران", "مشهد", "تهران", "شیراز"],
    "amount":   [120, 85, 240, 60, 310, 95, 150, 72, 88, 130],
    "status":   ["paid", "paid", "cancelled", "paid", "paid", "cancelled", "paid", "paid", "paid", "paid"],
})

# ۱. درآمد کل هر شهر (فقط پرداخت‌شده)
paid = df[df["status"] == "paid"]
print(paid.groupby("city")["amount"].sum().sort_values(ascending=False))

# ۲. خلاصه‌ی چندمعیاره
summary = paid.groupby("city").agg(
    revenue=("amount", "sum"),
    orders=("order_id", "count"),
    avg_order=("amount", "mean"),
).round(1)
summary["share_%"] = (summary["revenue"] / summary["revenue"].sum() * 100).round(1)
print(summary)

# ۳. سهم هر مشتری از درآمد
print(paid.groupby("customer")["amount"].sum().sort_values(ascending=False))

# ۴. جدول محوری شهر × وضعیت
print(df.pivot_table(index="city", columns="status", values="amount",
                     aggfunc="sum", fill_value=0, margins=True))
```

**مثال ۲ — مهندسی ویژگی با transform:**

```python
# ویژگی‌های قدرتمند برای مدل
df["city_avg"] = df.groupby("city")["amount"].transform("mean")
df["amount_vs_city"] = df["amount"] / df["city_avg"]
df["customer_total"] = df.groupby("customer")["amount"].transform("sum")
df["customer_n"] = df.groupby("customer")["order_id"].transform("count")
df["is_big_for_customer"] = df["amount"] > df["customer_total"] / df["customer_n"]

print(df[["customer", "amount", "city_avg", "customer_total", "is_big_for_customer"]])
```

این پنج ستون، از دو ستون خام ساخته شدند و **بسیار informative‌تر** از خودشان هستند.

**مثال ۳ — merge با بررسی صحت:**

```python
customers = pd.DataFrame({
    "customer_id": [1, 2, 3, 4],
    "name": ["علی", "مریم", "رضا", "سارا"],
    "tier": ["gold", "silver", "gold", "bronze"],
})
orders = pd.DataFrame({
    "order_id": [101, 102, 103, 104],
    "customer_id": [1, 2, 1, 9],      # ۹ وجود ندارد
    "amount": [120, 85, 240, 50],
})

merged = orders.merge(customers, on="customer_id", how="left", indicator=True)
print(merged["_merge"].value_counts())
print(merged[merged["_merge"] == "left_only"])     # سفارش‌های بی‌مشتری
print("تعداد ردیف تغییر کرد؟", len(merged) != len(orders))
```

**نکته:** اگر `merge` تعداد ردیف‌ها را زیاد کرد، یعنی جدول راست کلید تکراری دارد. این یک باگ خاموش است که داده را دو برابر می‌کند.

### مقایسه و دیدگاه‌های مختلف

| عملیات | Pandas | SQL | اکسل |
|---|---|---|---|
| تجمیع گروهی | `groupby().agg()` | `GROUP BY` | PivotTable |
| ترکیب جدول | `merge()` | `JOIN` | VLOOKUP |
| جدول محوری | `pivot_table()` | `GROUP BY` + `CASE` | PivotTable |
| مرتب‌سازی | `sort_values()` | `ORDER BY` | Sort |

**یک توصیه:** اگر SQL بلدی، Pandas را به‌عنوان «SQL در پایتون» ببین. `groupby` = `GROUP BY`، `merge` = `JOIN`. این نگاشت، یادگیری را نصف می‌کند.

### اشتباهات رایج

**۱. `merge` بدون بررسی تکراری بودن کلید.** ردیف‌ها دو برابر می‌شوند و متوجه نمی‌شوی.
**۲. `groupby` روی ستون با NaN.** NaN به‌طور پیش‌فرض حذف می‌شود؛ اگر لازم است، `dropna=False` بگذار.
**۳. `agg` به جای `transform` (یا برعکس).** اولی تعداد ردیف را کم می‌کند، دومی نگه می‌دارد.
**۴. `pivot_table` با `aggfunc="mean"` پیش‌فرض.** اگر جمع می‌خواستی، صریح بنویس.
**۵. `concat` بدون `ignore_index=True`.** index تکراری می‌شود و بعد `.loc` رفتار عجیب می‌دهد.
**۶. فراموش کردن `fill_value=0` در pivot.** NaN می‌گیری که در نمودار بد به‌نظر می‌رسد.

### نکات مهم و ترفندها

- **`indicator=True` را همیشه در merge بگذار.** یک ستون، جلوی ساعت‌ها دیباگ را می‌گیرد.
- **`transform` را برای ویژگی‌های «نسبت به گروه» استفاده کن.** قوی‌ترین ویژگی‌های جدولی از اینجا می‌آیند.
- **`named aggregation`** (`total=("amount","sum")`) خروجی خیلی تمیزتری از `agg(["sum"])` می‌دهد.
- **`sort_values(ascending=False).head()`** را با `nlargest` عوض کن.
- **میان‌بر:** groupby = تقسیم/اعمال/ترکیب · transform = ویژگی گروهی · merge = JOIN · pivot = جدول دو‌بعدی.

### خلاصه فصل

- `groupby` سه مرحله دارد: تقسیم، اعمال، ترکیب.
- `agg` خلاصه می‌دهد، `transform` هم‌اندازه‌ی داده برمی‌گرداند.
- `merge` انواع inner/left/right/outer دارد و باید با `indicator` بررسی شود.
- `pivot_table` جدول دو‌بعدی می‌سازد.
- مهندسی ویژگی با `transform`، قدرتمندترین کاربرد این فصل است.

**سؤالات مروری:**
1. تفاوت `agg` و `transform` چیست؟
2. چرا بعد از `merge` باید تعداد ردیف‌ها را چک کرد؟
3. چطور «سهم هر سفارش از کل خرید مشتری» را با `transform` بسازی؟

---

## منابع و مراجع

**مستندات**
- Pandas Official — *10 minutes to pandas*
- Pandas Official — *User Guide: Group by, Merge, Reshaping*
- Pandas Official — *Cookbook*

**کتاب‌ها**
- McKinney, W. — *Python for Data Analysis* (فصل ۵–۱۰)
- VanderPlas, J. — *Python Data Science Handbook*، فصل ۳

**تمرین**
- Kaggle Learn — *Pandas* (دوره‌ی کوتاه و عملی)
- Pandas Exercises (مخزن GitHub: guipsamora/pandas_exercises)

---

**بعدی:** [`07-EDA-Visualization.md`](./07-EDA-Visualization.md) — فصل ۳۰: EDA چیست؟
