# بخش ۰۳: پایتون برای هوش مصنوعی

**مدت یادگیری تقریبی**: ۴ ساعت
**سطح سختی**: مبتدی
**پیش‌نیاز**: فایل ۰۱ و ۰۲ — هیچ پیش‌نیاز برنامه‌نویسی اجباری نیست، اما اگر هرگز کد ننوشته‌ای، این فصل را آهسته‌تر بخوان

---

## فهرست محتوا

- [فصل ۱۱: چرا پایتون؟ آماده‌سازی محیط و Jupyter](#فصل-۱۱-چرا-پایتون-آمادهسازی-محیط-و-jupyter)
- [فصل ۱۲: متغیرها، انواع داده و ساختارهای داده](#فصل-۱۲-متغیرها-انواع-داده-و-ساختارهای-داده)
- [فصل ۱۳: کنترل جریان، توابع و ماژول‌ها](#فصل-۱۳-کنترل-جریان-توابع-و-ماژولها)
- [فصل ۱۴: کلاس‌ها و شیءگرایی به‌اندازه‌ی نیاز](#فصل-۱۴-کلاسها-و-شیءگرایی-بهاندازهی-نیاز)
- [فصل ۱۵: کار با فایل، JSON و مدیریت خطا](#فصل-۱۵-کار-با-فایل-json-و-مدیریت-خطا)

---

## فصل ۱۱: چرا پایتون؟ آماده‌سازی محیط و Jupyter

### مقدمه

چرا تقریباً همه‌ی دنیای AI با پایتون کار می‌کند؟ نه چون سریع‌ترین زبان است (نیست)، نه چون زیباترین است (بحث‌برانگیز). چون **اکوسیستم** دارد: هر ایده‌ای که در مقاله‌ای منتشر می‌شود، چند هفته بعد یک کتابخانه‌ی پایتون دارد.

این فصل کوتاه اما حیاتی است: اگر محیطت درست تنظیم نشود، ۴۰٪ وقتت را صرف حل خطاهای نصب می‌کنی و فکر می‌کنی «من برای این کار ساخته نشده‌ام». مشکل، تو نیستی؛ محیط است.

### مفاهیم اساسی

**۱. چرا پایتون؟**

| دلیل | توضیح |
|---|---|
| **اکوسیستم** | NumPy، Pandas، scikit-learn، PyTorch، TensorFlow — همه پایتون |
| **جامعه** | هر خطایی که بزنی، کسی قبلاً در Stack Overflow زده |
| **خوانایی** | کد شبیه انگلیسی است؛ تمرکز روی مسئله، نه syntax |
| **چسب‌کنندگی** | به C/C++/CUDA وصل می‌شود؛ کندی‌اش را با کتابخانه‌های بومی جبران می‌کند |
| **نوت‌بوک** | Jupyter اجازه می‌دهد کد، خروجی و توضیح را کنار هم ببینی |

**نکته‌ی مهم:** پایتون «کند» است، اما کتابخانه‌هایش (NumPy و...) در واقع کد C را اجرا می‌کنند. پس محاسبات سنگین در پایتون خام انجام نمی‌شود؛ فقط **هماهنگ** می‌شود. این را «two-language problem» می‌گویند: پایتون برای نوشتن، C برای اجرا.

**۲. محیط مجازی (Virtual Environment).** مهم‌ترین مفهوم این فصل.

مشکل: پروژه‌ی A به `numpy==1.24` نیاز دارد، پروژه‌ی B به `numpy==2.0`. اگر همه‌چیز را گلوبال نصب کنی، یکی می‌شکند.
راه‌حل: **هر پروژه، یک محیط مجازی جدا** با کتابخانه‌های خودش.

```bash
# ساخت
python -m venv venv

# فعال‌سازی — ویندوز (Git Bash)
source venv/Scripts/activate
# فعال‌سازی — ویندوز (CMD)
venv\Scripts\activate.bat
# فعال‌سازی — مک/لینوکس
source venv/bin/activate

# وقتی فعال است، ابتدای prompt می‌بینی: (venv)
# نصب کتابخانه فقط داخل این محیط انجام می‌شود
pip install numpy pandas matplotlib scikit-learn jupyter

# خروج
deactivate
```

**۳. pip و requirements.txt.**
`pip` نصب‌کننده‌ی بسته است. برای این‌که پروژه‌ات قابل بازتولید باشد:

```bash
pip freeze > requirements.txt     # ذخیره‌ی نسخه‌های دقیق
pip install -r requirements.txt   # نصب دقیقاً همان‌ها روی ماشین دیگر
```

**۴. Jupyter Notebook.** فایلی با پسوند `.ipynb` که ترکیبی از سلول‌های کد و متن است.
- **سلول کد:** با `Shift+Enter` اجرا می‌شود؛ خروجی زیرش می‌ماند.
- **سلول Markdown:** برای توضیح، فرمول، عنوان.
- **مزیت:** حالت اکتشافی. می‌توانی یک سلول را ده بار تغییر بدهی و اجرا کنی بدون این‌که کل برنامه را از نو اجرا کنی.
- **عیب:** ترتیب اجرا مهم است و اگر سلول‌ها را نامرتب اجرا کنی، نتیجه گمراه‌کننده می‌شود. **قاعده: وقتی به نتیجه رسیدی، `Restart & Run All` بزن.** اگر بدون خطا اجرا شد، نتیجه معتبر است.

**۵. VS Code یا JupyterLab؟** هر دو خوب‌اند. VS Code با افزونه‌ی Python و Jupyter، تجربه‌ی یکپارچه‌ای می‌دهد و برای پروژه‌های بزرگ بهتر است. JupyterLab در مرورگر ساده‌تر است.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — اولین بررسی محیط:**

```python
import sys
import platform

print("نسخه‌ی پایتون:", sys.version)
print("سیستم‌عامل:", platform.system(), platform.release())

# بررسی نصب کتابخانه‌ها
libs = ["numpy", "pandas", "matplotlib", "sklearn"]
for lib in libs:
    try:
        module = __import__(lib)
        print(f"✅ {lib}: {getattr(module, '__version__', 'نصب شده')}")
    except ImportError:
        print(f"❌ {lib}: نصب نشده — اجرا کن: pip install {lib}")
```

**مثال ۲ — یک نوت‌بوک نمونه (ساختار پیشنهادی):**

```
سلول ۱ (Markdown): # تحلیل داده‌ی فروش
سلول ۲ (Code):     import pandas as pd
سلول ۳ (Code):     df = pd.read_csv("sales.csv")
سلول ۴ (Code):     df.head()
سلول ۵ (Markdown): ## مشاهده: ستون price دارای مقدار منفی است
سلول ۶ (Code):     df = df[df["price"] > 0]
```

**مثال ۳ — خطای رایج و پیام آن:**

```
ModuleNotFoundError: No module named 'pandas'
```
یعنی: یا نصب نشده، یا محیط مجازی فعال نیست. **۹۰٪ مواقع، مورد دوم است.** چک کن `(venv)` در ابتدای prompt باشد.

### مقایسه و دیدگاه‌های مختلف

| ابزار | مزیت | عیب | مناسب برای |
|---|---|---|---|
| **Jupyter Notebook** | اکتشافی، خروجی کنار کد | ترتیب اجرا شکننده | یادگیری، تحلیل |
| **VS Code + Python** | دیباگر قوی، Git یکپارچه | نیاز به تنظیم اولیه | پروژه‌ی واقعی |
| **Google Colab** | GPU رایگان، بدون نصب | محدودیت زمان، وابسته به اینترنت | آموزش، آزمایش سریع |
| **اسکریپت `.py`** | قابل اجرا در تولید، تمیز | بدون خروجی فوری | pipeline، سرویس |

**دو دیدگاه درباره‌ی نوت‌بوک:**
- **«نوت‌بوک برای یادگیری و کاوش است، نه تولید.»** درست است: کد تولیدی باید ماژولار و تست‌پذیر باشد.
- **«نوت‌بوک همه‌کاره است.»** برای تحلیل یک‌باره بله، برای سرویس نه.
**توصیه‌ی من:** با نوت‌بوک یاد بگیر و کاوش کن؛ وقتی منطق تثبیت شد، آن را به توابع در فایل `.py` منتقل کن.

### اشتباهات رایج

**۱. «همه‌چیز را گلوبال نصب می‌کنم.»** چند ماه بعد، هیچ پروژه‌ای اجرا نمی‌شود.
**۲. «محیط مجازی را فعال نمی‌کنم.»** بعد `pip install` را گلوبال می‌زنی و تعجب می‌کنی چرا کار نمی‌کند.
**۳. «سلول‌ها را نامرتب اجرا می‌کنم و نتیجه را باور می‌کنم.»** همیشه در پایان `Restart & Run All`.
**۴. «نسخه‌ی کتابخانه‌ها را ثبت نمی‌کنم.»** شش ماه بعد، کدت روی ماشین خودت هم اجرا نمی‌شود.
**۵. «از `pip install` در وسط نوت‌بوک استفاده می‌کنم.»** با `!pip install` کار می‌کند اما محیط را آلوده می‌کند؛ در ترمینال انجام بده.

### نکات مهم و ترفندها

- **یک پوشه برای هر پروژه، یک venv برای هر پروژه.** این عادت، ۸۰٪ دردسرها را حذف می‌کند.
- **`pip list` و `pip freeze` را بشناس.** اولی می‌گوید چه داری، دومی نسخه‌ها را برای بازتولید می‌نویسد.
- **در Colab، `!pip install` با `!` کار می‌کند** و محیط از تو جدا است؛ نگران آلودگی نباش.
- **میان‌بر:** محیط درست = venv فعال + کتابخانه‌ها نصب + `Restart & Run All` بدون خطا.

### خلاصه فصل

- پایتون به‌خاطر اکوسیستم و جامعه‌اش زبان استاندارد AI است؛ کندی‌اش با کتابخانه‌های بومی جبران می‌شود.
- محیط مجازی، جداسازی وابستگی‌ها را ممکن می‌کند؛ برای هر پروژه یکی بساز.
- `requirements.txt` بازتولیدپذیری را تضمین می‌کند.
- Jupyter برای کاوش عالی است؛ در پایان `Restart & Run All` بزن.
- نوت‌بوک برای یادگیری، اسکریپت ماژولار برای تولید.

**سؤالات مروری:**
1. چرا پایتون با این‌که کند است، برای AI انتخاب شده؟
2. محیط مجازی چه مشکلی را حل می‌کند؟
3. تفاوت `pip list` و `pip freeze` چیست؟

---

## فصل ۱۲: متغیرها، انواع داده و ساختارهای داده

### مقدمه

اگر از قبل پایتون بلدی، این فصل را سریع بخوان و مستقیم برو سراغ بخش «ساختارهای داده» — چون ۹۰٪ کدهای AI با لیست، دیکشنری و آرایه نوشته می‌شوند. اگر بلد نیستی، این فصل پایه‌ی همه‌چیز است.

هدف این فصل، «آموزش کامل پایتون» نیست. هدف این است که **دقیقاً آن بخشی از پایتون را بدانی که در AI لازم می‌شود**.

### مفاهیم اساسی

**۱. متغیر و نوع داده.** پایتون نوع‌دهی پویا دارد: لازم نیست بگویی `int x`؛ فقط مقدار بده.

```python
age = 28              # int — عدد صحیح
price = 12.5          # float — عدد اعشاری
name = "علی"          # str — متن
is_active = True      # bool — درست/غلط
nothing = None        # NoneType — «هیچ مقداری»
```

**نکته‌ی حیاتی برای AI:** `None` با `0` و `""` فرق دارد. `None` یعنی «مقدار نداریم» — همان چیزی که در داده‌ی گمشده دیدیم (فایل ۰۲). در Pandas این به `NaN` تبدیل می‌شود.

**۲. اعداد و دقت.** یک تله‌ی کلاسیک:

```python
print(0.1 + 0.2)          # 0.30000000000000004
print(0.1 + 0.2 == 0.3)   # False
```
دلیل: اعداد اعشاری در حافظه‌ی دودویی دقیق ذخیره نمی‌شوند. **هرگز اعداد اعشاری را با `==` مقایسه نکن.** از `math.isclose(a, b)` یا `abs(a-b) < 1e-9` استفاده کن.

**۳. رشته‌ها (String) — ابزار اصلی کار با متن.**

```python
text = "  هوش مصنوعی جذاب است  "
print(text.strip())                 # حذف فاصله‌های اضافه
print(text.replace("جذاب", "شگفت")) # جایگزینی
print(text.split())                 # تقسیم به کلمات
print("AI" in text)                 # بررسی وجود
print(f"طول متن: {len(text)}")      # f-string
print(text.lower())                 # برای متن فارسی بی‌اثر، برای انگلیسی مفید
```

**۴. لیست (list).** مجموعه‌ی مرتب و قابل تغییر.

```python
scores = [85, 92, 78, 90]
scores.append(88)          # افزودن به آخر
scores[0]                  # 85 — ایندکس از صفر
scores[-1]                 # 88 — آخرین عضو
scores[1:3]                # [92, 78] — برش
sorted(scores, reverse=True)  # مرتب‌سازی نزولی
len(scores), sum(scores), max(scores)
```

**۵. تاپل (tuple).** مثل لیست، اما **تغییرناپذیر**. برای داده‌ی ثابت (مثل ابعاد تصویر) مناسب است.

```python
image_shape = (224, 224, 3)   # قابل تغییر نیست — خوب برای ثابت‌ها
```

**۶. دیکشنری (dict).** جفت کلید-مقدار. **پرکاربردترین ساختار در AI**، چون شبیه یک ردیف داده است.

```python
sample = {
    "age": 28,
    "city": "تهران",
    "churn": 0,
}
print(sample["age"])              # 28
print(sample.get("income", 0))    # 0 — مقدار پیش‌فرض اگر کلید نبود
sample["income"] = 5000           # افزودن
for key, value in sample.items():
    print(key, "=", value)
```

**۷. مجموعه (set).** بدون ترتیب، بدون تکرار. برای حذف تکراری‌ها و عملیات مجموعه‌ای.

```python
words = ["ai", "ml", "ai", "dl"]
print(set(words))                 # {'ai', 'ml', 'dl'}
print(len(set(words)))            # 3 — تعداد یکتا
```

**۸. List Comprehension.** میان‌بر پایتون برای ساخت لیست. در کد AI همه‌جا می‌بینی.

```python
squares = [x**2 for x in range(5)]              # [0, 1, 4, 9, 16]
evens = [x for x in range(10) if x % 2 == 0]    # [0, 2, 4, 6, 8]
names = [s["name"].upper() for s in samples]    # استخراج و تبدیل
```

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — ساختن یک دیتاست کوچک با دیکشنری و لیست:**

```python
# ساختار رایج داده در AI: لیستی از دیکشنری‌ها (شبیه JSON)
dataset = [
    {"id": 1, "age": 25, "city": "تهران",  "bought": 1},
    {"id": 2, "age": 34, "city": "شیراز",  "bought": 0},
    {"id": 3, "age": 29, "city": "تهران",  "bought": 1},
    {"id": 4, "age": None, "city": "مشهد", "bought": 0},
]

# ۱. میانگین سن، با نادیده گرفتن None
ages = [r["age"] for r in dataset if r["age"] is not None]
print("میانگین سن:", sum(ages) / len(ages))

# ۲. نرخ خرید
bought = [r["bought"] for r in dataset]
print("نرخ خرید:", sum(bought) / len(bought))

# ۳. شهرهای یکتا
print("شهرها:", sorted({r["city"] for r in dataset}))

# ۴. فیلتر: مشتریان تهرانی که خرید کرده‌اند
tehran_buyers = [
    r for r in dataset
    if r["city"] == "تهران" and r["bought"] == 1
]
print("تعداد:", len(tehran_buyers))
```

**مثال ۲ — تابع آماری ساده (بدون NumPy):**

```python
def describe(values):
    """خلاصه‌ی آماری یک لیست عددی — معادل کوچکِ pandas.describe()."""
    n = len(values)
    if n == 0:
        return None
    mean = sum(values) / n
    variance = sum((x - mean) ** 2 for x in values) / n
    return {
        "n": n,
        "mean": round(mean, 2),
        "min": min(values),
        "max": max(values),
        "std": round(variance ** 0.5, 2),
    }

print(describe([12, 15, 18, 22, 30]))
```

**مثال ۳ — کپی سطحی در برابر عمیق (تله‌ای که همه می‌خورند):**

```python
import copy

original = [[1, 2], [3, 4]]
shallow = original.copy()      # کپی سطحی: لیست بیرونی جدید، درونی‌ها مشترک
deep = copy.deepcopy(original) # کپی عمیق: همه‌چیز جدید

shallow[0][0] = 999
print("original:", original)   # [[999, 2], [3, 4]] ← تغییر کرد!
print("deep:", deep)           # [[1, 2], [3, 4]] ← سالم
```
**قاعده:** در پردازش داده، وقتی می‌خواهی داده‌ی اصلی را دست‌نخورده نگه داری، `deepcopy` یا `df.copy()` استفاده کن.

### مقایسه و دیدگاه‌های مختلف

| ساختار | ترتیب | تغییرپذیر | تکراری مجاز | سرعت جست‌وجو | کاربرد در AI |
|---|---|---|---|---|---|
| `list` | ✅ | ✅ | ✅ | کند O(n) | نگه‌داری نمونه‌ها |
| `tuple` | ✅ | ❌ | ✅ | کند | ابعاد، پارامترهای ثابت |
| `dict` | ✅ (از ۳.۷) | ✅ | کلید نه | سریع O(1) | یک ردیف داده، پارامترها |
| `set` | ❌ | ✅ | ❌ | سریع O(1) | یکتا کردن، عملیات مجموعه‌ای |
| `numpy.array` | ✅ | ✅ | ✅ | خیلی سریع (برداری) | همه‌ی محاسبات عددی |

**یک توصیه‌ی عملی:** برای داده‌ی عددی، **هرگز از لیست پایتون برای محاسبات استفاده نکن**. لیست برای نگه‌داری خوب است؛ برای محاسبه، NumPy (فایل ۰۵) ده تا صد برابر سریع‌تر است.

### اشتباهات رایج

**۱. `list1 = list2` به جای `list.copy()`.** هر دو به یک شیء اشاره می‌کنند؛ تغییر یکی، دیگری را عوض می‌کند.
**۲. دسترسی به ایندکس خارج از محدوده.** `IndexError`. همیشه `len()` را چک کن یا از `try/except` استفاده کن.
**۳. `dict["missing_key"]`.** `KeyError`. از `.get()` استفاده کن.
**۴. مقایسه‌ی `None` با `==`.** از `is None` استفاده کن (درست‌تر و سریع‌تر).
**۵. تغییر لیست در حین پیمایش آن.** نتیجه غیرقابل پیش‌بینی است؛ روی کپی پیمایش کن.
**۶. استفاده از `float` برای پول.** خطای گردکردن جمع می‌شود؛ از `decimal.Decimal` یا محاسبه به سنت استفاده کن.

### نکات مهم و ترفندها

- **f-string بهترین دوستت است:** `f"دقت: {acc:.2%}"` → `دقت: 92.35%`. در گزارش‌دهی مدل، بی‌نهایت کاربرد دارد.
- **`enumerate` و `zip` را یاد بگیر:** دو تابعی که کد را نصف می‌کنند.
  ```python
  for i, (x, y) in enumerate(zip(xs, ys)):
      ...
  ```
- **`type()` و `isinstance()`:** وقتی خطای عجیب گرفتی، اول نوع داده را چک کن. نیمی از خطاهای «عجیب» از نوع اشتباه است (مثلاً `str` به جای `int`).
- **میان‌بر:** ۹۰٪ کد AI = لیست + دیکشنری + حلقه + شرط. بقیه‌اش کتابخانه است.

### خلاصه فصل

- پایتون نوع‌دهی پویا دارد؛ `None` یعنی «مقدار نداریم»، نه صفر.
- اعداد اعشاری را با `==` مقایسه نکن.
- چهار ساختار اصلی: لیست (مرتب، تغییرپذیر)، تاپل (ثابت)، دیکشنری (کلید-مقدار)، مجموعه (یکتا).
- List comprehension کد را کوتاه و خوانا می‌کند.
- کپی سطحی در برابر عمیق، یک تله‌ی کلاسیک است.

**سؤالات مروری:**
1. تفاوت `list.copy()` و `copy.deepcopy()` چیست و کِی مهم می‌شود؟
2. چرا `0.1 + 0.2 == 0.3` نتیجه‌ی `False` می‌دهد؟
3. برای نگه‌داری یک ردیف داده (مثلاً یک مشتری)، چه ساختاری مناسب‌تر است و چرا؟

---

## فصل ۱۳: کنترل جریان، توابع و ماژول‌ها

### مقدمه

در AI، کدت معمولاً یک الگوی تکراری دارد: **«برای هر نمونه چیزی محاسبه کن، اگر شرطی برقرار بود کاری بکن، نتیجه را جمع کن.»** این فصل دقیقاً همان ابزارها را می‌دهد: `if`، `for`، تابع، و ماژول.

اگر این چهار را خوب بلد باشی، می‌توانی هر الگوریتمی را از صفر پیاده کنی — همان کاری که در فایل ۱۶ با شبکه‌ی عصبی می‌کنیم.

### مفاهیم اساسی

**۱. شرط (`if / elif / else`).**

```python
def classify_score(score):
    if score >= 90:
        return "عالی"
    elif score >= 75:
        return "خوب"
    elif score >= 50:
        return "متوسط"
    else:
        return "ضعیف"

print(classify_score(88))   # خوب
```

**نکته‌ی AI:** همین ساختار، قلب «آستانه‌گذاری» (thresholding) در دسته‌بندی است. مدل یک عدد بین ۰ و ۱ می‌دهد؛ `if p > 0.5` تصمیم می‌سازد. انتخاب آستانه، یک تصمیم مهندسی است (فایل ۱۴).

**۲. حلقه‌ها.**

```python
# for روی لیست
for x in [1, 2, 3]:
    print(x)

# for با range
for i in range(3):        # 0, 1, 2
    print(i)

# while — وقتی تعداد تکرار معلوم نیست
error = 1.0
step = 0
while error > 0.01 and step < 1000:
    error *= 0.9          # کاهش تدریجی خطا (شبیه آموزش مدل)
    step += 1
print(f"بعد از {step} قدم، خطا: {error:.4f}")
```

**۳. `break`، `continue`، `else` روی حلقه.**

```python
for x in [3, 7, 12, 5]:
    if x % 2 == 0:
        print("اولین عدد زوج:", x)
        break          # خروج از حلقه
```
`else` روی حلقه، وقتی اجرا می‌شود که `break` رخ نداده باشد — کمتر شناخته‌شده اما مفید است.

**۴. تابع (Function).** واحد پایه‌ی بازاستفاده.

```python
def accuracy(y_true, y_pred):
    """نسبت پیش‌بینی‌های درست — ساده‌ترین معیار دسته‌بندی."""
    correct = sum(1 for t, p in zip(y_true, y_pred) if t == p)
    return correct / len(y_true)

print(accuracy([1, 0, 1, 1], [1, 0, 0, 1]))   # 0.75
```

**پارامتر پیش‌فرض، آرگومان کلیدی، و `*args`:**

```python
def train_summary(name, epochs=10, lr=0.01, **metrics):
    """خلاصه‌ی یک اجرای آموزش — الگوی رایج در کد AI."""
    print(f"مدل: {name} | epoch: {epochs} | lr: {lr}")
    for k, v in metrics.items():
        print(f"  {k}: {v}")

train_summary("mlp", epochs=20, lr=0.001, accuracy=0.91, loss=0.23)
```

**قاعده‌ی مهم:** **هرگز از مقدار پیش‌فرض تغییرپذیر (`def f(x=[])`) استفاده نکن.** لیست پیش‌فرض بین فراخوانی‌ها مشترک می‌ماند و باگ‌های عجیب می‌سازد.

**۵. Scope (دامنه).** متغیر داخل تابع، بیرون دیده نمی‌شود. اگر می‌خواهی تابع چیزی را عوض کند، **بازگردان** (`return`) نه این‌که به `global` دست بزنی.

**۶. ماژول و import.**

```python
# فایل my_utils.py
def normalize(values):
    lo, hi = min(values), max(values)
    return [(v - lo) / (hi - lo) for v in values]

# فایل main.py
from my_utils import normalize
print(normalize([10, 20, 30]))   # [0.0, 0.5, 1.0]
```

انواع import:
```python
import math                  # کل ماژول
from math import sqrt        # یک تابع
import numpy as np           # با نام مستعار (استاندارد در AI)
from sklearn.metrics import accuracy_score
```

**۷. نام‌گذاری استاندارد در AI (این را حفظ کن):**

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import torch                    # PyTorch
import tensorflow as tf
from sklearn.model_selection import train_test_split
```
این نام‌ها آن‌قدر استانداردند که اگر `import numpy as num` بنویسی، هر کسی کدت را می‌بیند تعجب می‌کند.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — پیاده‌سازی یک epoch آموزش (بدون کتابخانه):**

```python
def train_one_epoch(weights, X, y, lr=0.01):
    """
    یک دور آموزش برای رگرسیون خطی ساده.
    weights: لیست [w, b]
    X: لیست ویژگی‌ها، y: لیست هدف‌ها
    """
    w, b = weights
    n = len(X)
    total_loss = 0.0

    for xi, yi in zip(X, y):
        # ۱. پیش‌بینی (forward)
        y_hat = w * xi + b
        # ۲. خطا
        error = y_hat - yi
        total_loss += error ** 2
        # ۳. به‌روزرسانی (gradient descent ساده)
        w -= lr * 2 * error * xi
        b -= lr * 2 * error

    return [w, b], total_loss / n

# آزمایش: داده‌ی مصنوعی با رابطه‌ی y = 3x + 2
X = [1, 2, 3, 4, 5]
y = [5, 8, 11, 14, 17]
weights = [0.0, 0.0]          # شروع از صفر

for epoch in range(200):
    weights, loss = train_one_epoch(weights, X, y, lr=0.01)
    if epoch % 50 == 0:
        print(f"epoch {epoch:3d} | loss={loss:8.4f} | w={weights[0]:.3f} b={weights[1]:.3f}")

print("\nنتیجه‌ی نهایی:", [round(v, 3) for v in weights])   # نزدیک [3, 2]
```

این ۲۰ خط، **کل ایده‌ی یادگیری ماشین** است: پیش‌بینی → خطا → اصلاح. بقیه‌ی کتاب، بزرگ‌کردن همین ایده است.

**مثال ۲ — تابع ارزیابی با خروجی ساختاریافته:**

```python
def evaluate(y_true, y_pred):
    tp = sum(1 for t, p in zip(y_true, y_pred) if t == 1 and p == 1)
    tn = sum(1 for t, p in zip(y_true, y_pred) if t == 0 and p == 0)
    fp = sum(1 for t, p in zip(y_true, y_pred) if t == 0 and p == 1)
    fn = sum(1 for t, p in zip(y_true, y_pred) if t == 1 and p == 0)

    precision = tp / (tp + fp) if (tp + fp) else 0.0
    recall    = tp / (tp + fn) if (tp + fn) else 0.0
    f1        = (2 * precision * recall / (precision + recall)
                 if (precision + recall) else 0.0)

    return {
        "tp": tp, "tn": tn, "fp": fp, "fn": fn,
        "accuracy": round((tp + tn) / len(y_true), 3),
        "precision": round(precision, 3),
        "recall": round(recall, 3),
        "f1": round(f1, 3),
    }

print(evaluate([1,0,1,1,0,0,1], [1,0,0,1,0,1,1]))
```

**مثال ۳ — تقسیم داده به‌صورت دستی (تا بفهمی کتابخانه چه می‌کند):**

```python
import random

def train_test_split_manual(X, y, test_ratio=0.2, seed=42):
    """تقسیم تصادفی داده — همان کاری که sklearn می‌کند (ساده‌شده)."""
    random.seed(seed)
    indices = list(range(len(X)))
    random.shuffle(indices)

    n_test = int(len(X) * test_ratio)
    test_idx = indices[:n_test]
    train_idx = indices[n_test:]

    X_train = [X[i] for i in train_idx]
    y_train = [y[i] for i in train_idx]
    X_test  = [X[i] for i in test_idx]
    y_test  = [y[i] for i in test_idx]
    return X_train, X_test, y_train, y_test

X = list(range(100))
y = [x * 2 for x in X]
X_tr, X_te, y_tr, y_te = train_test_split_manual(X, y)
print(len(X_tr), len(X_te))   # 80 20
```

### مقایسه و دیدگاه‌های مختلف

| سبک کد | مثال | خوانایی | سرعت | کِی |
|---|---|---|---|---|
| حلقه‌ی صریح | `for x in X: ...` | بالا | کند | یادگیری، منطق پیچیده |
| Comprehension | `[f(x) for x in X]` | متوسط | متوسط | تبدیل ساده |
| برداری (NumPy) | `X * 2` | بالا (اگر بلد باشی) | خیلی سریع | محاسبات عددی |
| تابع کتابخانه | `model.fit(X, y)` | بالا | بهینه | همیشه، اگر موجود است |

**یک بحث همیشگی:** «اول از صفر بنویس یا مستقیم از کتابخانه استفاده کن؟»
- **از صفر:** می‌فهمی چه اتفاقی می‌افتد؛ برای یادگیری ضروری است.
- **کتابخانه:** بهینه، تست‌شده، قابل اعتماد؛ برای تولید ضروری است.
**توصیه:** هر الگوریتم اصلی را **یک بار** از صفر بنویس (مثل مثال ۱)، بعد برای همیشه از کتابخانه استفاده کن.

### اشتباهات رایج

**۱. `def f(x, items=[])`.** لیست پیش‌فرض مشترک می‌ماند.
**۲. تغییر لیست در حین پیمایش.** روی کپی پیمایش کن یا لیست جدید بساز.
**۳. استفاده از `global`.** تابع باید ورودی بگیرد و خروجی بدهد؛ نه این‌که متغیر جهانی را دست بزند.
**۴. تقسیم بر صفر.** در محاسبه‌ی precision/recall حتماً حالت صفر را هندل کن (`if denominator else 0`).
**۵. نام‌گذاری بد.** `x1, x2, temp, data2` بعد از دو هفته غیرقابل خواندن می‌شود. `X_train, y_train, val_loss` بنویس.
**۶. کپی-پیست کردن بلوک کد.** اگر سه جا تکرار شد، تابع بساز.

### نکات مهم و ترفندها

- **هر تابع را کوچک نگه دار.** اگر بیشتر از ۳۰ خط شد، احتمالاً باید به دو تابع شکسته شود.
- **Docstring بنویس، حتی یک خط.** شش ماه بعد، خودت مشتری اصلی‌اش هستی.
- **`print` را در تابع نگذار، `return` بده.** چاپ، مسئولیت فراخواننده است. این کار تابع را تست‌پذیر می‌کند.
- **در حلقه‌های سنگین، `tqdm` اضافه کن:** `from tqdm import tqdm` و `for x in tqdm(items)` — می‌بینی کجایی و چقدر مانده.
- **میان‌بر:** تابع خوب = یک کار + ورودی روشن + خروجی روشن + بدون اثر جانبی.

### خلاصه فصل

- `if` آستانه‌گذاری را ممکن می‌کند؛ `for` روی داده می‌چرخد.
- تابع، واحد بازاستفاده است؛ ورودی بگیر، خروجی بده، اثر جانبی نداشته باش.
- مقدار پیش‌فرض تغییرپذیر، یک باگ کلاسیک است.
- import با نام مستعار استاندارد (`np`, `pd`, `plt`) یک قرارداد جامعه است.
- با ۲۰ خط حلقه و تابع می‌توان یک epoch آموزش نوشت — کل ML همین است.

**سؤالات مروری:**
1. چرا `def f(items=[])` خطرناک است؟
2. تفاوت `break` و `continue` را با مثال بگو.
3. در تابع `evaluate`، چرا باید حالت مخرج صفر را هندل کرد؟

---

## فصل ۱۴: کلاس‌ها و شیءگرایی به‌اندازه‌ی نیاز

### مقدمه

در AI، شیءگرایی (OOP) دو کاربرد اصلی دارد:
1. **کتابخانه‌ها شیءگرا هستند:** `model = LinearRegression()`, `model.fit(X, y)`, `model.predict(X)`. اگر مفهوم کلاس و متد را نفهمی، این سه خط جادو به‌نظر می‌رسد.
2. **ساختن pipelineهای قابل نگهداری:** هر مرحله‌ی پردازش می‌تواند یک کلاس با `fit` و `transform` باشد.

این فصل کوتاه است و فقط آن مقدار از OOP را می‌دهد که واقعاً لازم داری. لازم نیست وراثت چندسطحی و metaclass یاد بگیری.

### مفاهیم اساسی

**۱. کلاس چیست؟** یک «قالب» (blueprint). شیء (object) یک «نمونه» از آن قالب.

```python
class Model:
    def __init__(self, learning_rate=0.01):
        """سازنده — هنگام ساخت شیء اجرا می‌شود."""
        self.learning_rate = learning_rate   # ویژگی (attribute)
        self.weights = None                  # هنوز آموزش ندیده

    def fit(self, X, y):
        """آموزش — فعلاً ساده‌شده."""
        self.weights = [0.0] * len(X[0])
        # ... منطق آموزش ...
        return self

    def predict(self, X):
        """پیش‌بینی با وزن‌های آموزش‌دیده."""
        if self.weights is None:
            raise RuntimeError("اول fit را صدا بزن")
        return [sum(w * xi for w, xi in zip(self.weights, row)) for row in X]

m = Model(learning_rate=0.001)
print(m.learning_rate)   # 0.001
```

**`self` چیست؟** ارجاع به خود شیء. پایتون آن را خودکار می‌فرستد. `self.weights` یعنی «وزنِ این نمونه».

**۲. چرا `fit` مقدار `self` برمی‌گرداند؟** برای زنجیره‌سازی (method chaining). scikit-learn این کار را نمی‌کند، اما الگوی خوبی است.

**۳. `__init__` و `__repr__`:**

```python
class Dataset:
    def __init__(self, X, y, name="unnamed"):
        self.X = X
        self.y = y
        self.name = name

    def __repr__(self):
        """نمایش خوانا هنگام print — برای دیباگ حیاتی است."""
        return f"Dataset(name={self.name!r}, n={len(self.X)}, features={len(self.X[0])})"

    def __len__(self):
        return len(self.X)

ds = Dataset([[1, 2], [3, 4]], [0, 1], name="train")
print(ds)          # Dataset(name='train', n=2, features=2)
print(len(ds))     # 2
```

**۴. روش‌های جادویی (dunder) که واقعاً لازم داری:**

| متد | چه‌کار می‌کند | چرا مهم است |
|---|---|---|
| `__init__` | ساخت شیء | تنظیم اولیه |
| `__repr__` | نمایش در دیباگ | بدون آن، `print` می‌دهد `<object at 0x...>` |
| `__len__` | `len(obj)` | داده‌ی قابل شمارش |
| `__getitem__` | `obj[i]` | دسترسی به نمونه |
| `__call__` | `obj(x)` | شیء مثل تابع عمل می‌کند |

**۵. سه اصل مفید OOP در AI:**

- **کپسوله‌سازی (Encapsulation):** داده و عملیات مربوط به هم در یک جا. مدل، وزن‌هایش را خودش نگه می‌دارد.
- **ترکیب (Composition):** «یک pipeline، چند مرحله دارد». بیشتر از وراثت استفاده می‌شود.
- **وراثت (Inheritance):** کمتر از آنچه فکر می‌کنی لازم است. یک سطح وراثت برای رابط مشترک کافی است.

**۶. الگوی `fit / transform` (الگوی کلیدی scikit-learn):**

```python
class StandardScaler:
    """استانداردساز دستی — دقیقاً همان قرارداد sklearn."""
    def __init__(self):
        self.mean_ = None
        self.std_ = None

    def fit(self, X):
        n = len(X)
        d = len(X[0])
        self.mean_ = [sum(row[j] for row in X) / n for j in range(d)]
        self.std_ = [
            (sum((row[j] - self.mean_[j]) ** 2 for row in X) / n) ** 0.5
            for j in range(d)
        ]
        return self

    def transform(self, X):
        return [
            [(row[j] - self.mean_[j]) / (self.std_[j] or 1) for j in range(len(row))]
            for row in X
        ]

    def fit_transform(self, X):
        return self.fit(X).transform(X)

X = [[1.0, 100.0], [2.0, 200.0], [3.0, 300.0]]
sc = StandardScaler().fit(X)
print(sc.mean_)                 # [2.0, 200.0]
print(sc.transform([[4.0, 400.0]]))
```

**چرا این الگو مهم است؟** چون `fit` فقط روی داده‌ی آموزش صدا زده می‌شود و `transform` روی هر داده‌ای. همین جداسازی، از نشت داده جلوگیری می‌کند (فایل ۰۲ و ۱۴).

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — یک مدل کامل با کلاس:**

```python
class SimpleLinearRegression:
    """رگرسیون خطی تک‌ویژگی با گرادیان نزولی — از صفر."""

    def __init__(self, lr=0.01, epochs=1000):
        self.lr = lr
        self.epochs = epochs
        self.w = 0.0
        self.b = 0.0
        self.history = []          # تاریخ خطا — برای رسم منحنی یادگیری

    def fit(self, X, y):
        n = len(X)
        for _ in range(self.epochs):
            dw = db = 0.0
            loss = 0.0
            for xi, yi in zip(X, y):
                y_hat = self.w * xi + self.b
                err = y_hat - yi
                loss += err ** 2
                dw += 2 * err * xi
                db += 2 * err
            self.w -= self.lr * dw / n
            self.b -= self.lr * db / n
            self.history.append(loss / n)
        return self

    def predict(self, X):
        return [self.w * x + self.b for x in X]

    def score(self, X, y):
        """R² — ضریب تعیین."""
        preds = self.predict(X)
        y_mean = sum(y) / len(y)
        ss_res = sum((yi - pi) ** 2 for yi, pi in zip(y, preds))
        ss_tot = sum((yi - y_mean) ** 2 for yi in y)
        return 1 - ss_res / ss_tot

    def __repr__(self):
        return f"SimpleLinearRegression(w={self.w:.3f}, b={self.b:.3f})"

# آزمایش
X = [1, 2, 3, 4, 5, 6, 7, 8]
y = [3 * x + 2 + (0.3 if x % 2 else -0.3) for x in X]   # با کمی نویز

model = SimpleLinearRegression(lr=0.01, epochs=2000).fit(X, y)
print(model)                       # نزدیک w=3, b=2
print("R²:", round(model.score(X, y), 4))
print("پیش‌بینی x=10:", round(model.predict([10])[0], 2))
```

**مثال ۲ — pipeline با ترکیب:**

```python
class Pipeline:
    """ترکیب چند مرحله — دقیقاً الگوی sklearn.pipeline."""
    def __init__(self, steps):
        self.steps = steps        # لیستی از (نام، شیء با fit/transform)

    def fit(self, X, y=None):
        for name, step in self.steps:
            X = step.fit_transform(X) if hasattr(step, "fit_transform") else step.fit(X).transform(X)
        return self

    def transform(self, X):
        for name, step in self.steps:
            X = step.transform(X)
        return X

# pipe = Pipeline([("scale", StandardScaler()), ("model", SimpleLinearRegression())])
```

**نکته:** این ۱۰ خط، همان ایده‌ای است که `sklearn.pipeline.Pipeline` با ۵۰۰ خط پیاده کرده. حالا وقتی از کتابخانه استفاده می‌کنی، می‌دانی پشت صحنه چه خبر است.

### مقایسه و دیدگاه‌های مختلف

| سبک | مزیت | عیب | کِی |
|---|---|---|---|
| **تابع‌محور** | ساده، تست‌پذیر، بدون حالت | برای وضعیت پیچیده مناسب نیست | تحلیل، اسکریپت |
| **شیءگرا** | حالت و رفتار کنار هم، قابل ترکیب | پیچیدگی بیشتر | مدل، pipeline |
| **تابع خالص (functional)** | بدون اثر جانبی، قابل موازی‌سازی | گاهی ناخوانا | پردازش داده |

**یک دیدگاه اقلیت اما مهم:** بسیاری از مهندسان ارشد ML می‌گویند «کد ML را تا جای ممکن تابع‌محور بنویس؛ کلاس فقط جایی که کتابخانه اجبار می‌کند یا حالت واقعاً داری.» دلیل: کلاس‌های زیاد با `self` پر، دیباگ را سخت می‌کنند.

### اشتباهات رایج

**۱. فراموش کردن `self` در تعریف متد.** `def fit(X, y):` به جای `def fit(self, X, y):` → `TypeError`.
**۲. استفاده از متغیر کلاس به جای متغیر نمونه.** `weights = []` بیرون از `__init__` بین همه‌ی نمونه‌ها مشترک است.
**۳. صدا زدن `predict` قبل از `fit`.** همیشه یک `raise` بگذار (مثل مثال `Model`).
**۴. وراثت بی‌مورد.** اگر فقط می‌خواهی کد مشترک داشته باشی، تابع بساز، نه کلاس پایه.
**۵. `__repr__` ننوشتن.** دیباگ بدون آن، عذاب است.
**۶. نگه‌داشتن داده‌ی بزرگ در `self`.** اگر مدل ۱۰ گیگابایت داده را در خودش نگه دارد، ذخیره و انتقالش غیرممکن می‌شود. فقط پارامترهای لازم را نگه دار.

### نکات مهم و ترفندها

- **قرارداد sklearn را یاد بگیر و رعایت کن:** `fit`, `predict`, `transform`, `fit_transform`, `score`. اگر کلاس تو این‌ها را داشته باشد، با `Pipeline` و `GridSearchCV` کار می‌کند.
- **پارامترهای آموزش‌دیده را با `_` تمام کن** (`coef_`, `mean_`) — قرارداد sklearn برای جدا کردن «پارامتر ورودی» از «چیز یادگرفته‌شده».
- **`@property` را برای محاسبات مشتق‌شده بشناس:**
  ```python
  @property
  def n_params(self):
      return len(self.weights)
  ```
- **میان‌بر:** کلاس = داده + رفتار. اگر کلاس فقط یک تابع است و حالت ندارد، تابع بنویس.

### خلاصه فصل

- کلاس، قالب است؛ شیء، نمونه. `self` ارجاع به خود شیء است.
- `__init__` برای تنظیم اولیه، `__repr__` برای دیباگ خوانا.
- الگوی `fit / transform` قلب طراحی scikit-learn است و از نشت داده جلوگیری می‌کند.
- ترکیب (composition) بیشتر از وراثت به کار می‌آید.
- قرارداد sklearn را رعایت کن تا کدت با ابزارهای استاندارد کار کند.

**سؤالات مروری:**
1. چرا `fit` فقط روی داده‌ی آموزش صدا زده می‌شود؟
2. تفاوت متغیر کلاس و متغیر نمونه چیست؟
3. اگر کلاسی هیچ حالتی نداشته باشد، بهتر است کلاس باشد یا تابع؟ چرا؟

---

## فصل ۱۵: کار با فایل، JSON و مدیریت خطا

### مقدمه

در پروژه‌های واقعی، داده از جایی می‌آید: فایل CSV، پاسخ API به شکل JSON، فایل تنظیمات YAML. و همیشه چیزی خراب می‌شود: فایل نیست، JSON نامعتبر است، شبکه قطع است، ستون مورد نظر وجود ندارد.

**تفاوت کد آماتور و حرفه‌ای، در همین فصل است:** کد آماتور فرض می‌کند همه‌چیز درست است؛ کد حرفه‌ای انتظار خرابی دارد.

### مفاهیم اساسی

**۱. خواندن و نوشتن فایل متنی.**

```python
# روش توصیه‌شده: با بلوک with (فایل خودکار بسته می‌شود)
with open("notes.txt", "w", encoding="utf-8") as f:
    f.write("خط اول\n")
    f.write("خط دوم\n")

with open("notes.txt", "r", encoding="utf-8") as f:
    content = f.read()          # همه‌ی متن
    # یا خط به خط:
    # for line in f: ...

print(content)
```

**همیشه `encoding="utf-8"` بنویس.** در ویندوز، پیش‌فرض `cp1252` است و متن فارسی خراب می‌شود. این یکی از رایج‌ترین باگ‌های فارسی‌زبان‌ها است.

**۲. مسیرها — `pathlib` بهتر از `os.path`:**

```python
from pathlib import Path

data_dir = Path("data")
data_dir.mkdir(exist_ok=True)          # ساخت پوشه اگر نبود

file_path = data_dir / "train.csv"     # الحاق مسیر با /
print(file_path.exists())              # بررسی وجود
print(file_path.suffix)                # .csv
print(file_path.stem)                  # train

# همه‌ی فایل‌های csv در یک پوشه
for p in data_dir.glob("*.csv"):
    print(p.name)
```

**چرا `pathlib`؟** چون `/` روی ویندوز و لینوکس کار می‌کند، و متدهای مفید دارد (`exists`, `suffix`, `glob`, `read_text`).

**۳. JSON — زبان تبادل داده.**

```python
import json

data = {
    "model": "logistic_regression",
    "params": {"C": 1.0, "max_iter": 1000},
    "metrics": {"accuracy": 0.91, "f1": 0.88},
    "tags": ["baseline", "v1"],
}

# نوشتن
with open("config.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=2)

# خواندن
with open("config.json", "r", encoding="utf-8") as f:
    loaded = json.load(f)

print(loaded["metrics"]["accuracy"])   # 0.91
```

**`ensure_ascii=False`** لازم است تا فارسی به `\u0645...` تبدیل نشود. `indent=2` فایل را خوانا می‌کند.

**۴. CSV — پرکاربردترین فرمت داده.**

```python
import csv

# خواندن
with open("data.csv", "r", encoding="utf-8", newline="") as f:
    reader = csv.DictReader(f)        # هر ردیف → دیکشنری
    rows = list(reader)

# نوشتن
with open("out.csv", "w", encoding="utf-8", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "score"])
    writer.writeheader()
    writer.writerows([{"name": "علی", "score": 90}])
```

در عمل از Pandas استفاده می‌کنی (`pd.read_csv`)، اما دانستن نسخه‌ی خام مفید است — و برای فایل‌های خیلی بزرگ که در حافظه جا نمی‌شوند، `csv` با پیمایش خط‌به‌خط تنها راه است.

**۵. مدیریت خطا — `try / except / else / finally`.**

```python
try:
    with open("missing.csv", encoding="utf-8") as f:
        data = f.read()
except FileNotFoundError:
    print("فایل پیدا نشد — مسیر را چک کن")
    data = ""
except PermissionError:
    print("دسترسی به فایل نداری")
    data = ""
except UnicodeDecodeError:
    print("کدگذاری فایل درست نیست — احتمالاً utf-8 نیست")
    data = ""
else:
    print("بدون خطا خوانده شد")
finally:
    print("این بلوک همیشه اجرا می‌شود")
```

**قاعده‌ی طلایی:** `except` خالی (bare except) **هرگز** ننویس. اگر همه‌ی خطاها را ببلعد، باگ واقعی را پنهان می‌کنی و بعداً دیباگش غیرممکن می‌شود.

**۶. خطاهای رایج در AI و معنی‌شان:**

| خطا | معنی | راه‌حل |
|---|---|---|
| `FileNotFoundError` | مسیر غلط | `Path.exists()` چک کن |
| `UnicodeDecodeError` | کدگذاری اشتباه | `encoding="utf-8"` یا `errors="ignore"` |
| `KeyError` | کلید/ستون نیست | `df.columns` را چاپ کن |
| `ValueError: could not convert string to float` | داده‌ی متنی در ستون عددی | `pd.to_numeric(errors="coerce")` |
| `MemoryError` | داده بزرگ‌تر از RAM | `chunksize` یا `dtype` کوچک‌تر |
| `ModuleNotFoundError` | نصب نیست / venv فعال نیست | `pip install` و فعال‌سازی venv |

**۷. کار با فایل‌های بزرگ (chunking):**

```python
import pandas as pd

total = 0
for chunk in pd.read_csv("big.csv", chunksize=100_000, encoding="utf-8"):
    total += len(chunk)
    # پردازش هر تکه: پاکسازی، تجمیع، ذخیره
print("کل ردیف‌ها:", total)
```

این الگو، پردازش فایل ۱۰ گیگابایتی با ۴ گیگابایت RAM را ممکن می‌کند.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — ذخیره و بازخوانی نتیجه‌ی آموزش (الگوی واقعی):**

```python
import json
from pathlib import Path
from datetime import datetime

def save_experiment(results, path="experiments.jsonl"):
    """ذخیره‌ی هر اجرا در یک خط JSON (JSONL) — قابل افزودن، قابل خواندن تکه‌تکه."""
    record = {
        "timestamp": datetime.now().isoformat(timespec="seconds"),
        **results,
    }
    with open(path, "a", encoding="utf-8") as f:
        f.write(json.dumps(record, ensure_ascii=False) + "\n")

def load_experiments(path="experiments.jsonl"):
    p = Path(path)
    if not p.exists():
        return []
    with open(p, encoding="utf-8") as f:
        return [json.loads(line) for line in f if line.strip()]

save_experiment({"model": "rf", "accuracy": 0.91, "params": {"n_estimators": 100}})
save_experiment({"model": "lr", "accuracy": 0.86, "params": {"C": 1.0}})

for r in load_experiments():
    print(r["model"], r["accuracy"])
```

**چرا JSONL و نه یک JSON بزرگ؟** چون می‌توانی بدون خواندن کل فایل، یک خط اضافه کنی. برای لاگ آزمایش‌ها ایده‌آل است.

**مثال ۲ — تابع مقاوم برای خواندن داده:**

```python
import pandas as pd

def safe_read_csv(path, required_cols=None):
    """خواندن CSV با بررسی‌های لازم — الگوی تولیدی."""
    p = Path(path)
    if not p.exists():
        raise FileNotFoundError(f"فایل داده پیدا نشد: {p.resolve()}")

    try:
        df = pd.read_csv(p, encoding="utf-8")
    except UnicodeDecodeError:
        df = pd.read_csv(p, encoding="latin-1")   # تلاش دوم

    if df.empty:
        raise ValueError("فایل خوانده شد اما خالی است")

    if required_cols:
        missing = set(required_cols) - set(df.columns)
        if missing:
            raise ValueError(f"ستون‌های لازم نیست: {missing} | موجود: {list(df.columns)}")

    return df

# df = safe_read_csv("data/train.csv", required_cols=["age", "target"])
```

**مثال ۳ — کلاس خطای سفارشی:**

```python
class DataValidationError(Exception):
    """خطای اعتبارسنجی داده — قابل تشخیص از خطاهای دیگر."""
    pass

def validate_range(series, name, lo, hi):
    out_of_range = ((series < lo) | (series > hi)).sum()
    if out_of_range > 0:
        raise DataValidationError(
            f"ستون {name}: {out_of_range} مقدار خارج از بازه‌ی [{lo}, {hi}]"
        )
```

**چرا خطای سفارشی؟** چون بعداً می‌توانی `except DataValidationError` بزنی و بدانی مشکل از داده است، نه از کد. این تفکیک، در پروژه‌های بزرگ حیاتی است.

### مقایسه و دیدگاه‌های مختلف

| فرمت | خوانا برای انسان | ساختار تودرتو | حجم | کاربرد |
|---|---|---|---|---|
| CSV | ✅ | ❌ | متوسط | داده‌ی جدولی |
| JSON | ✅ | ✅ | بزرگ‌تر | تنظیمات، API |
| JSONL | ✅ | ✅ (هر خط) | متوسط | لاگ، داده‌ی جریانی |
| YAML | ✅✅ | ✅ | متوسط | تنظیمات پروژه |
| Parquet | ❌ | محدود | کوچک | داده‌ی حجیم |
| Pickle | ❌ | ✅ | کوچک | ذخیره‌ی مدل پایتون (**ناامن برای فایل ناشناس**) |

**هشدار امنیتی مهم:** هرگز فایل `pickle` یا `joblib` از منبع ناشناس باز نکن. pickle می‌تواند کد دلخواه اجرا کند. برای تبادل داده از JSON/Parquet استفاده کن؛ pickle فقط برای مدل‌های خودت.

### اشتباهات رایج

**۱. فراموش کردن `encoding="utf-8"`.** متن فارسی خراب می‌شود (یا `UnicodeDecodeError`).
**۲. `except:` خالی.** باگ‌ها را پنهان می‌کند.
**۳. ساختن مسیر با `+` یا `\`.** از `Path` و `/` استفاده کن.
**۴. باز نکردن فایل با `with`.** فایل باز می‌ماند، در ویندوز قفل می‌شود و نمی‌توانی پاکش کنی.
**۵. نوشتن روی فایل اصلی داده.** همیشه روی کپی بنویس؛ خام را دست نزن (فایل ۰۲).
**۶. ذخیره‌ی مسیر مطلق در کد.** `C:\Users\...\data.csv` روی ماشین دیگر کار نمی‌کند. مسیر نسبی + `Path(__file__).parent`.
**۷. `pd.read_csv` روی فایل ۱۰ گیگابایتی.** `MemoryError`. از `chunksize` استفاده کن.

### نکات مهم و ترفندها

- **همیشه `encoding="utf-8"` بنویس.** یک عادت که ساعت‌ها صرفه‌جویی می‌کند.
- **مسیرها را نسبی و از محل فایل بساز:**
  ```python
  BASE_DIR = Path(__file__).resolve().parent
  DATA = BASE_DIR / "data"
  ```
- **برای تنظیمات، JSON یا YAML؛ برای داده، Parquet/CSV؛ برای مدل، joblib.**
- **`try/except` را دور «مرزهای سیستم» بگذار:** خواندن فایل، درخواست شبکه، ورودی کاربر. نه دور هر خط کد.
- **میان‌بر:** کد خوب = `with` + `encoding` + `Path` + `except` مشخص.

### خلاصه فصل

- همیشه با `with` فایل باز کن و `encoding="utf-8"` بگذار.
- `pathlib` جای `os.path` را گرفته؛ مسیرها را با `/` بساز.
- JSON برای تنظیمات و تبادل، JSONL برای لاگ، Parquet برای داده‌ی حجیم.
- `except` خالی ننویس؛ خطاهای مشخص را بگیر و پیام معنادار بده.
- فایل‌های بزرگ را با `chunksize` پردازش کن.

**سؤالات مروری:**
1. چرا `encoding="utf-8"` برای متن فارسی حیاتی است؟
2. تفاوت JSON و JSONL چیست و چرا برای لاگ آزمایش‌ها JSONL بهتر است؟
3. چرا باز کردن pickle از منبع ناشناس خطرناک است؟

---

## منابع و مراجع

**کتاب‌ها**
- Sweigart, A. — *Automate the Boring Stuff with Python* (فصل‌های ۱–۹، رایگان آنلاین)
- McKinney, W. — *Python for Data Analysis* (نوشته‌ی خالق Pandas)
- Ramalho, L. — *Fluent Python* (برای سطح بالاتر)

**منابع آنلاین**
- مستندات رسمی پایتون: `docs.python.org/3/tutorial/`
- Real Python — بخش‌های Python Basics، Pathlib، Exception Handling
- scikit-learn — *Developing scikit-learn estimators* (برای قرارداد fit/transform)
- PEP 8 — راهنمای سبک کد پایتون

**ابزارهای پیشنهادی**
- VS Code + افزونه‌ی Python
- Jupyter Lab
- `black` (فرمت‌کننده)، `ruff` (linter)

---

**بعدی:** [`04-Math-for-AI.md`](./04-Math-for-AI.md) — فصل ۱۶: بردار و ماتریس
