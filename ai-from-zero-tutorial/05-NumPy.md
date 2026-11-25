# بخش ۰۵: NumPy و محاسبات عددی

**مدت یادگیری تقریبی**: ۲ ساعت
**سطح سختی**: مبتدی
**پیش‌نیاز**: فایل ۰۳ (پایتون) و فایل ۰۴ (بردار و ماتریس)

---

## فهرست محتوا

- [فصل ۲۱: NumPy چیست و چرا هزار برابر سریع‌تر است](#فصل-۲۱-numpy-چیست-و-چرا-هزار-برابر-سریعتر-است)
- [فصل ۲۲: آرایه‌ها: ساخت، شکل، ایندکس و برش](#فصل-۲۲-آرایهها-ساخت-شکل-ایندکس-و-برش)
- [فصل ۲۳: عملیات برداری و Broadcasting](#فصل-۲۳-عملیات-برداری-و-broadcasting)
- [فصل ۲۴: توابع آماری، جبر خطی و تصادفی](#فصل-۲۴-توابع-آماری-جبر-خطی-و-تصادفی)

---

## فصل ۲۱: NumPy چیست و چرا هزار برابر سریع‌تر است

### مقدمه

اگر یک کتابخانه را باید خوب بلد باشی، NumPy است. نه چون اسمش در رزومه خوب است، بلکه چون **همه‌چیز روی آن ساخته شده**: Pandas روی NumPy، scikit-learn روی NumPy، PyTorch و TensorFlow هم رابطشان شبیه NumPy است.

این فصل توضیح می‌دهد چرا NumPy سریع است — و همین توضیح، تو را از یک «کاربر کتابخانه» به کسی تبدیل می‌کند که می‌داند کِی و چرا از آن استفاده کند.

### مفاهیم اساسی

**۱. مشکل لیست پایتون.**

```python
# با لیست پایتون
squares = [x**2 for x in range(1_000_000)]
```
هر عدد یک شیء کامل پایتون است: نوع، مقدار، شمارنده‌ی مرجع. برای یک میلیون عدد، ده‌ها مگابایت و میلی‌ثانیه‌ها زمان. برای ضرب دو آرایه‌ی یک میلیونی، فاجعه.

**۲. راه‌حل NumPy: آرایه‌ی هم‌نوع (homogeneous array).**

`np.array` همه‌ی اعداد را در یک بلوک پیوسته‌ی حافظه، با یک نوع ثابت (`dtype`) ذخیره می‌کند. نتیجه:
- **حافظه‌ی کمتر:** ۸ بایت برای هر `float64`، به جای ~۳۲ بایت شیء پایتون.
- **سرعت بیشتر:** عملیات در حلقه‌ی C اجرا می‌شود، نه در مفسر پایتون.
- **عملیات برداری:** `a + b` روی کل آرایه، بدون نوشتن حلقه.

**۳. برداری‌سازی (Vectorization).** یعنی «بدون حلقه بنویس، کتابخانه حلقه را در C بزند».

```python
# کند (پایتون)
result = [a[i] + b[i] for i in range(len(a))]

# سریع (NumPy)
result = a + b
```

**چرا این‌قدر سریع است؟** سه دلیل:
1. **حلقه در C**، نه در مفسر پایتون (هر خط پایتون هزینه‌ی تفسیر دارد).
2. **SIMD:** پردازنده می‌تواند چند عدد را هم‌زمان در یک دستور پردازش کند.
3. **حافظه‌ی پیوسته:** دسترسی به حافظه‌ی پشت‌سرهم، cache-friendly است.

**۴. اندازه‌گیری واقعی:**

```python
import numpy as np
import time

n = 1_000_000
py_list = list(range(n))
np_arr = np.arange(n)

# لیست
t0 = time.perf_counter()
py_result = [x * 2 + 1 for x in py_list]
t1 = time.perf_counter()

# NumPy
t0n = time.perf_counter()
np_result = np_arr * 2 + 1
t1n = time.perf_counter()

print(f"لیست:  {(t1-t0)*1000:8.2f} ms")
print(f"NumPy: {(t1n-t0n)*1000:8.2f} ms")
print(f"نسبت:  {(t1-t0)/(t1n-t0n):8.1f}x")
```

معمولاً نسبت بین ۲۰ تا ۱۰۰ برابر است. روی عملیات سنگین‌تر (مثل ضرب ماتریسی)، هزاران برابر.

**۵. `dtype` — نوع داده.**

| dtype | بایت | بازه/دقت | کاربرد |
|---|---|---|---|
| `int8/16/32/64` | ۱/۲/۴/۸ | صحیح | برچسب، شمارش |
| `float32` | ۴ | ~۷ رقم | شبکه‌ی عصبی (پیش‌فرض GPU) |
| `float64` | ۸ | ~۱۵ رقم | محاسبات عمومی |
| `bool` | ۱ | True/False | ماسک |

**نکته‌ی مهم:** `float32` نصف حافظه‌ی `float64` است و در GPU دو برابر سریع‌تر. برای آموزش شبکه‌ی عصبی، `float32` استاندارد است. برای آمار دقیق، `float64`.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — مقایسه‌ی حافظه:**

```python
import numpy as np
import sys

n = 100_000
py_list = list(range(n))
np_arr = np.arange(n, dtype=np.int64)
np_arr32 = np.arange(n, dtype=np.int32)

print(f"لیست پایتون: {sys.getsizeof(py_list) + sum(sys.getsizeof(x) for x in py_list[:100]) * n // 100:,} بایت (تقریبی)")
print(f"NumPy int64: {np_arr.nbytes:,} بایت")
print(f"NumPy int32: {np_arr32.nbytes:,} بایت")
```

**مثال ۲ — ضرب ماتریسی: قلب AI:**

```python
import numpy as np
import time

n = 500
A = np.random.rand(n, n)
B = np.random.rand(n, n)

t0 = time.perf_counter()
C = A @ B                    # عملگر @ برای ضرب ماتریسی
t1 = time.perf_counter()

print(f"ضرب {n}×{n}: {(t1-t0)*1000:.1f} ms")
print("شکل نتیجه:", C.shape)
```

**مثال ۳ — یک لایه‌ی شبکه‌ی عصبی در یک خط:**

```python
import numpy as np

# ۱۰۰۰ نمونه، ۲۰ ویژگی
X = np.random.randn(1000, 20)
W = np.random.randn(20, 64) * 0.1     # ۲۰ ورودی → ۶۴ نورون
b = np.zeros(64)

Z = X @ W + b                          # forward pass — یک خط!
print("شکل خروجی:", Z.shape)            # (1000, 64)
```

در فایل ۱۶ همین را از صفر با حلقه می‌نویسیم و تفاوت سرعت را می‌بینیم.

### مقایسه و دیدگاه‌های مختلف

| ساختار | سرعت | حافظه | انعطاف | کِی |
|---|---|---|---|---|
| `list` | کند | زیاد | بالا (نوع مختلط) | داده‌ی غیرعددی، منطق |
| `np.array` | سریع | کم | فقط هم‌نوع | همه‌ی محاسبات عددی |
| `pd.Series/DataFrame` | متوسط | متوسط | بالا (برچسب، نوع مختلط) | داده‌ی جدولی |
| `torch.Tensor` | سریع + GPU | کم | متوسط | یادگیری عمیق |

**یک قاعده:** اگر داده‌ات **هم‌نوع و عددی** است، NumPy. اگر **جدولی و مختلط** است، Pandas. اگر **می‌خواهی روی GPU و با گرادیان خودکار** کار کنی، PyTorch.

### اشتباهات رایج

**۱. استفاده از حلقه برای عملیات آرایه‌ای.** `for i in range(len(a)): c[i] = a[i]+b[i]` — صد برابر کندتر از `c = a+b`.
**۲. `np.append` در حلقه.** هر بار کل آرایه کپی می‌شود: O(n²). به جای آن، لیست بساز و در آخر `np.array` کن، یا از `np.concatenate` یک‌باره استفاده کن.
**۳. انتظار نوع مختلط.** `np.array([1, "a"])` همه را رشته می‌کند. NumPy هم‌نوع است.
**۴. فراموش کردن این‌که برش، view است نه کپی.** `b = a[1:3]` و بعد تغییر `b` → `a` هم عوض می‌شود.
**۵. `dtype=object`.** NumPy را به لیست پایتون تبدیل می‌کند و همه‌ی مزیت سرعت از بین می‌رود.

### نکات مهم و ترفندها

- **قاعده‌ی طلایی: اگر حلقه‌ی `for` روی آرایه نوشتی، توقف کن و فکر کن.** معمولاً نسخه‌ی برداری وجود دارد.
- **`%timeit` در Jupyter** سریع‌ترین راه اندازه‌گیری است: `%timeit a @ b`.
- **`dtype` را آگاهانه انتخاب کن.** `float32` برای شبکه‌ی عصبی، `float64` برای آمار، `int8` برای برچسب.
- **میان‌بر:** NumPy = آرایه‌ی هم‌نوع + عملیات برداری + حلقه در C.

### خلاصه فصل

- NumPy آرایه را در حافظه‌ی پیوسته و هم‌نوع ذخیره می‌کند؛ همین، سرعت را می‌سازد.
- برداری‌سازی یعنی حذف حلقه‌ی پایتون به نفع حلقه‌ی C.
- `dtype` هم حافظه و هم سرعت را تعیین می‌کند.
- `A @ B` ضرب ماتریسی است — هسته‌ی هر لایه‌ی شبکه‌ی عصبی.
- NumPy پایه‌ی Pandas، scikit-learn و چارچوب‌های یادگیری عمیق است.

**سؤالات مروری:**
1. سه دلیل سرعت NumPy نسبت به لیست پایتون چیست؟
2. چرا `np.append` در حلقه بد است؟
3. تفاوت `float32` و `float64` کِی مهم می‌شود؟

---

## فصل ۲۲: آرایه‌ها: ساخت، شکل، ایندکس و برش

### مقدمه

این فصل، «دستور زبان» NumPy است. اگر `shape`، ایندکس‌گذاری و برش را خوب بلد باشی، ۸۰٪ خطاهای ابعادی — که رایج‌ترین خطای مبتدی‌هاست — برایت پیش نمی‌آید.

### مفاهیم اساسی

**۱. ساخت آرایه.**

```python
import numpy as np

np.array([1, 2, 3])                    # از لیست
np.zeros((2, 3))                        # همه صفر
np.ones((2, 3))                         # همه یک
np.full((2, 3), 7)                      # همه ۷
np.arange(0, 10, 2)                     # [0 2 4 6 8]
np.linspace(0, 1, 5)                    # [0. 0.25 0.5 0.75 1.]
np.eye(3)                               # ماتریس همانی ۳×۳
np.random.randn(2, 3)                   # نرمال استاندارد
np.random.rand(2, 3)                    # یکنواخت [0,1)
```

**۲. شکل (Shape) و ابعاد (ndim).**

```python
a = np.array([1, 2, 3])
print(a.shape)      # (3,)      ← یک‌بعدی
print(a.ndim)       # 1
print(a.size)       # 3

b = np.array([[1, 2, 3], [4, 5, 6]])
print(b.shape)      # (2, 3)    ← دوبعدی: ۲ ردیف، ۳ ستون
print(b.ndim)       # 2
```

**تله‌ی کلاسیک:** `(3,)` با `(3, 1)` فرق دارد.

```python
v = np.array([1, 2, 3])         # shape (3,)
c = np.array([[1], [2], [3]])   # shape (3, 1)
r = np.array([[1, 2, 3]])       # shape (1, 3)
```
اولی بردار، دومی ستون، سومی ردیف. در ضرب ماتریسی، این تفاوت همه‌چیز را عوض می‌کند.

**۳. تغییر شکل (reshape).**

```python
a = np.arange(12)               # [0..11]
b = a.reshape(3, 4)             # ۳×۴
c = a.reshape(2, -1)            # -1 = «خودت حساب کن» → (2, 6)
d = a.reshape(-1, 1)            # ستون: (12, 1)
```

**قاعده:** تعداد کل عناصر باید حفظ شود. `12 = 3×4 = 2×6 = 12×1`.

**۴. `flatten` و `ravel`.** مسطح کردن.

```python
b = np.arange(6).reshape(2, 3)
b.flatten()     # کپی — تغییرش روی b اثر ندارد
b.ravel()       # view — اگر ممکن باشد، بدون کپی
```

**۵. ایندکس‌گذاری.**

```python
a = np.array([[10, 20, 30],
              [40, 50, 60]])

a[0, 1]         # 20 — ردیف ۰، ستون ۱
a[1]            # [40, 50, 60] — کل ردیف
a[:, 2]         # [30, 60] — کل ستون ۲
a[-1, -1]       # 60 — آخرین
```

**۶. برش (Slicing).**

```python
a[0:2, 1:3]     # ردیف ۰..۱، ستون ۱..۲
a[::2]          # هر دو ردیف
a[::-1]         # معکوس
a[:, ::-1]      # ستون‌های معکوس
```

**۷. ایندکس‌گذاری بولی (Boolean Masking) — پرکاربردترین تکنیک.**

```python
scores = np.array([85, 42, 91, 67, 78])

mask = scores >= 70
print(mask)              # [ True False  True False  True]
print(scores[mask])      # [85 91 78]

# ترکیب شرط‌ها
print(scores[(scores >= 60) & (scores < 90)])   # [85 67 78]
```

**تله:** از `&` و `|` استفاده کن، نه `and` و `or`. و پرانتز بگذار، چون اولویت عملگرها غافلگیرکننده است.

**۸. ایندکس‌گذاری fancy.**

```python
a = np.array([10, 20, 30, 40, 50])
print(a[[0, 2, 4]])       # [10 30 50]
print(a[np.array([1, 1, 3])])   # [20 20 40] — تکرار مجاز
```

**۹. view در برابر copy — تله‌ی مهم.**

```python
a = np.arange(10)
b = a[2:5]          # view
b[0] = 999
print(a)            # [0 1 999 3 4 ...] ← a تغییر کرد!

c = a[2:5].copy()   # کپی مستقل
c[0] = 0
print(a)            # a دست‌نخورده
```

**قاعده:** برش (slicing) = view؛ ایندکس بولی و fancy = copy. وقتی شک داری، `.copy()` بزن.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — آماده‌سازی داده‌ی تصویری:**

```python
import numpy as np

# یک «تصویر» مصنوعی ۴×۴ با سه کانال رنگ
image = np.random.randint(0, 256, size=(4, 4, 3), dtype=np.uint8)
print("شکل:", image.shape)          # (4, 4, 3)

# نرمال‌سازی به [0,1] برای شبکه‌ی عصبی
normalized = image.astype(np.float32) / 255.0
print("بازه:", normalized.min(), "تا", normalized.max())

# تبدیل به شکل مورد انتظار PyTorch: (C, H, W)
chw = normalized.transpose(2, 0, 1)
print("شکل PyTorch:", chw.shape)    # (3, 4, 4)

# افزودن بعد batch: (N, C, H, W)
batch = chw[np.newaxis, ...]        # یا chw[None, :, :, :]
print("شکل batch:", batch.shape)    # (1, 3, 4, 4)
```

**مثال ۲ — پاکسازی داده با ماسک:**

```python
import numpy as np

ages = np.array([25, -3, 34, 200, 29, 41, -1, 38])

# مقدارهای غیرممکن را NaN کن
clean = ages.astype(float)
clean[(clean < 0) | (clean > 120)] = np.nan
print(clean)

# میانگین با نادیده گرفتن NaN
print("میانگین:", np.nanmean(clean))

# جایگزینی NaN با میانه
median = np.nanmedian(clean)
clean = np.where(np.isnan(clean), median, clean)
print(clean)
```

**مثال ۳ — تقسیم داده به train/test:**

```python
import numpy as np

X = np.random.randn(1000, 5)
y = np.random.randint(0, 2, 1000)

rng = np.random.default_rng(seed=42)     # مولد تصادفی مدرن (توصیه‌شده)
indices = rng.permutation(len(X))

split = int(0.8 * len(X))
train_idx, test_idx = indices[:split], indices[split:]

X_train, X_test = X[train_idx], X[test_idx]
y_train, y_test = y[train_idx], y[test_idx]

print(X_train.shape, X_test.shape)       # (800, 5) (200, 5)
```

**نکته:** `np.random.default_rng()` جای `np.random.seed()` را گرفته. API قدیمی هنوز کار می‌کند اما مولد جدید بهتر و سریع‌تر است.

### مقایسه و دیدگاه‌های مختلف

| عملیات | نتیجه | کپی؟ |
|---|---|---|
| `a[1:3]` | برش | view |
| `a[[0,2]]` | fancy index | copy |
| `a[a>0]` | boolean mask | copy |
| `a.reshape(...)` | تغییر شکل | view (اگر ممکن) |
| `a.flatten()` | مسطح | copy |
| `a.ravel()` | مسطح | view (اگر ممکن) |
| `a.T` | ترانهاده | view |

**یک توصیه:** وقتی نمی‌دانی view است یا کپی، `.copy()` بزن. کمی حافظه بیشتر، اما باگ کمتر.

### اشتباهات رایج

**۱. اشتباه گرفتن `(3,)` با `(3,1)`.** رایج‌ترین خطای ابعادی.
**۲. تغییر view و تعجب از تغییر آرایه‌ی اصلی.**
**۳. `and` به جای `&` در ماسک.** `and` روی آرایه، `ValueError` می‌دهد.
**۴. فراموش کردن پرانتز:** `a > 1 & a < 5` اشتباه تفسیر می‌شود؛ بنویس `(a > 1) & (a < 5)`.
**۵. `reshape` با تعداد عناصر ناسازگار.** `12` را نمی‌شود `5×3` کرد.
**۶. ایندکس با float.** `a[1.0]` خطا می‌دهد؛ ایندکس باید صحیح باشد.

### نکات مهم و ترفندها

- **`shape` را چاپ کن.** در ۹۰٪ موارد، باگ ابعادی با یک `print(x.shape)` حل می‌شود.
- **`None` یا `np.newaxis` برای افزودن بعد:** `x[None, :]` → `(1, n)`.
- **`squeeze()` برای حذف بعدهای تک:** `(1, 5, 1)` → `(5,)`.
- **`np.where(شرط, الف, ب)`** معادل برداری `if/else` است.
- **میان‌بر:** shape را بلد باش، برش را بلد باش، ماسک بولی را بلد باش — بقیه جزئیات است.

### خلاصه فصل

- `shape` مهم‌ترین ویژگی آرایه است؛ `(3,)` و `(3,1)` فرق دارند.
- `reshape` شکل را عوض می‌کند، `-1` یعنی «خودت حساب کن».
- برش = view، ماسک بولی = copy.
- ماسک بولی، پرکاربردترین ابزار فیلتر داده است.
- `np.random.default_rng()` روش مدرن تولید اعداد تصادفی است.

**سؤالات مروری:**
1. `np.arange(12).reshape(3, -1)` چه شکلی دارد؟
2. تفاوت `flatten()` و `ravel()` چیست؟
3. چرا `a[1:3]` می‌تواند آرایه‌ی اصلی را تغییر بدهد؟

---

## فصل ۲۳: عملیات برداری و Broadcasting

### مقدمه

Broadcasting یکی از آن قابلیت‌های NumPy است که وقتی بفهمی، **کدت نصف می‌شود و سرعتت چند برابر**. و وقتی نفهمی، خطاهای ابعادی مرموز می‌گیری که ساعت‌ها وقتت را می‌گیرد.

این فصل، همان «فهمیدن» است.

### مفاهیم اساسی

**۱. عملیات مؤلفه‌به‌مؤلفه (Element-wise).**

```python
import numpy as np

a = np.array([1, 2, 3])
b = np.array([10, 20, 30])

print(a + b)      # [11 22 33]
print(a * b)      # [10 40 90]   ← مؤلفه‌به‌مؤلفه، نه ضرب داخلی!
print(a ** 2)     # [1 4 9]
print(a / 2)      # [0.5 1.  1.5]
```

**تله‌ی شماره یک:** `a * b` ضرب داخلی **نیست**. برای ضرب داخلی: `a @ b` یا `np.dot(a, b)`.

**۲. توابع جهانی (Universal Functions / ufunc).** روی کل آرایه کار می‌کنند:

```python
a = np.array([1, 4, 9, 16])
np.sqrt(a)        # [1. 2. 3. 4.]
np.log(a)         # لگاریتم طبیعی
np.exp(a)
np.abs(np.array([-1, 2, -3]))
np.clip(a, 2, 10) # محدود کردن به بازه
np.round(a / 3, 2)
```

**۳. Broadcasting — قانون اصلی.**

وقتی دو آرایه با شکل‌های متفاوت عمل می‌کنند، NumPy ابعاد را «پخش» می‌کند. قاعده:

> از راست به چپ، ابعاد را مقایسه کن. دو بعد سازگارند اگر برابر باشند یا یکی از آن‌ها ۱ باشد. بعدِ ۱ به اندازه‌ی بعدِ دیگر تکرار می‌شود.

```
A: (3, 4)
B:    (4,)     →  B به (3,4) پخش می‌شود  ✅

A: (3, 4)
B: (3,)        →  از راست: 4 vs 3 ✗      ❌ خطا

A: (3, 1)
B: (1, 4)      →  نتیجه (3,4)             ✅
```

**۴. مثال‌های broadcasting که هر روز لازم می‌شوی:**

```python
X = np.random.randn(100, 5)          # ۱۰۰ نمونه، ۵ ویژگی
mean = X.mean(axis=0)                # (5,) — میانگین هر ستون
std = X.std(axis=0)                  # (5,)

# استانداردسازی — یک خط، بدون حلقه
X_scaled = (X - mean) / std          # (100,5) - (5,) → پخش می‌شود
print(X_scaled.mean(axis=0).round(10))   # نزدیک صفر
print(X_scaled.std(axis=0).round(10))    # نزدیک یک
```

**این یک خط، کاری است که در Pandas با `StandardScaler` می‌کنیم.** حالا می‌دانی پشت صحنه چه خبر است.

**۵. `axis` — مهم‌ترین پارامتر گمراه‌کننده.**

```python
a = np.array([[1, 2, 3],
              [4, 5, 6]])

a.sum()            # 21  — همه
a.sum(axis=0)      # [5 7 9]     — جمع ستون‌ها (رو به پایین)
a.sum(axis=1)      # [6 15]      — جمع ردیف‌ها (رو به راست)
```

**قاعده‌ی حفظی:** `axis=0` یعنی «این محور را خرد کن» → نتیجه یک بعد کمتر، به اندازه‌ی ستون‌ها.
`axis=1` یعنی «ستون‌ها را خرد کن» → نتیجه به اندازه‌ی ردیف‌ها.

**۶. `keepdims=True` — برای حفظ ابعاد.**

```python
a = np.array([[1, 2, 3], [4, 5, 6]])
m = a.mean(axis=1)                    # (2,)
m_keep = a.mean(axis=1, keepdims=True)  # (2,1)

print((a - m_keep).shape)             # (2,3) — broadcasting درست
# print((a - m).shape)                # خطا! (2,3) - (2,) ناسازگار
```

**این ترفند، جان هزاران کد را نجات داده.**

**۷. `einsum` — برای عملیات پیچیده.** یک بار ببین، بعداً لازم می‌شود:

```python
A = np.random.rand(3, 4)
B = np.random.rand(4, 5)
print(np.einsum("ij,jk->ik", A, B).shape)   # (3,5) — همان A @ B
```

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — محاسبه‌ی فاصله‌ی همه‌ی جفت‌ها (KNN):**

```python
import numpy as np

X_train = np.random.randn(100, 3)     # ۱۰۰ نقطه‌ی آموزش
x_query = np.random.randn(3)          # یک نقطه‌ی پرسش

# روش حلقه‌ای (کند)
dists_loop = [np.sqrt(((X_train[i] - x_query) ** 2).sum())
              for i in range(len(X_train))]

# روش برداری (سریع)
diffs = X_train - x_query             # (100,3) - (3,) → پخش
dists_vec = np.sqrt((diffs ** 2).sum(axis=1))

print("یکسان؟", np.allclose(dists_loop, dists_vec))
print("نزدیک‌ترین:", dists_vec.argmin())
```

**مثال ۲ — ماتریس فاصله‌ی کامل (بدون حلقه):**

```python
A = np.random.randn(50, 3)
B = np.random.randn(30, 3)

# (50,1,3) - (1,30,3) → (50,30,3) → sum → (50,30)
diffs = A[:, None, :] - B[None, :, :]
D = np.sqrt((diffs ** 2).sum(axis=2))
print(D.shape)     # (50, 30)
```

**این ترفند `[:, None, :]` را حفظ کن.** برای هر «مقایسه‌ی همه با همه» لازم می‌شود.

**مثال ۳ — پیاده‌سازی softmax به‌صورت برداری:**

```python
def softmax_batch(Z):
    """Z: (n_samples, n_classes) → احتمال‌ها."""
    Z_shift = Z - Z.max(axis=1, keepdims=True)   # پایداری عددی
    expZ = np.exp(Z_shift)
    return expZ / expZ.sum(axis=1, keepdims=True)

Z = np.random.randn(4, 3)
P = softmax_batch(Z)
print(P.round(3))
print("جمع هر ردیف:", P.sum(axis=1).round(6))   # [1. 1. 1. 1.]
```

**مثال ۴ — یک لایه‌ی کامل با فعال‌سازی:**

```python
def relu(x):
    return np.maximum(0, x)

X = np.random.randn(32, 10)      # batch ۳۲ نمونه، ۱۰ ویژگی
W1 = np.random.randn(10, 16) * 0.1
b1 = np.zeros(16)
W2 = np.random.randn(16, 1) * 0.1
b2 = np.zeros(1)

h = relu(X @ W1 + b1)            # لایه‌ی پنهان
out = h @ W2 + b2                # لایه‌ی خروجی
print(h.shape, out.shape)        # (32,16) (32,1)
```

### مقایسه و دیدگاه‌های مختلف

| روش | کد | سرعت | خوانایی |
|---|---|---|---|
| حلقه‌ی پایتون | `for i in range(n): ...` | کند | بالا برای مبتدی |
| Broadcasting | `X - mean` | سریع | بالا اگر بلد باشی |
| `einsum` | `np.einsum("ij,jk->ik", A, B)` | سریع | متوسط |
| تابع آماده | `np.linalg.norm(...)` | سریع | بالا |

**یک هشدار:** broadcasting می‌تواند **بی‌صدا اشتباه** شود. اگر شکل‌ها تصادفاً سازگار باشند، کد اجرا می‌شود اما نتیجه غلط است. مثال:

```python
a = np.random.randn(3, 3)
b = np.random.randn(3)      # قرار بود ستون باشد
print((a - b).shape)        # (3,3) — اجرا می‌شود، اما b روی ستون‌ها پخش شده، نه ردیف‌ها
```

همیشه شکل‌ها را چاپ کن.

### اشتباهات رایج

**۱. `a * b` به‌جای `a @ b`.** ضرب مؤلفه‌به‌مؤلفه با ضرب داخلی فرق دارد.
**۲. `axis` اشتباه.** `axis=0` ستون‌ها را جمع می‌کند، نه ردیف‌ها.
**۳. فراموش کردن `keepdims`.** بعد از `mean(axis=1)` شکل به `(n,)` تبدیل می‌شود و broadcasting می‌شکند.
**۴. broadcasting بی‌صدا غلط.** شکل‌ها سازگارند اما معنایشان نه.
**۵. `np.exp` بدون شیفت.** برای اعداد بزرگ، `inf`. همیشه `x - max(x)`.
**۶. مقایسه‌ی float با `==`.** از `np.allclose` یا `np.isclose` استفاده کن.

### نکات مهم و ترفندها

- **`[:, None]` و `[None, :]` را حفظ کن.** این دو، کلید ساختن ابعاد برای broadcasting‌اند.
- **`keepdims=True` را در `mean`/`sum` روی محور، همیشه در نظر بگیر.** جلوگیری از ۹۰٪ خطاهای ابعادی.
- **`np.allclose(a, b)` برای مقایسه‌ی آرایه‌ها.** `==` روی float، تقریباً همیشه غلط است.
- **`axis` را با یک مثال کوچک تست کن.** یک آرایه‌ی ۲×۳ بساز و ببین کدام جهت خرد می‌شود.
- **میان‌بر:** broadcasting = کشیدن بعدهای ۱ به اندازه‌ی بعد دیگر. `axis=0` = ستون‌ها.

### خلاصه فصل

- عملیات `+`, `*`, `/` مؤلفه‌به‌مؤلفه‌اند؛ ضرب داخلی `@` است.
- Broadcasting ابعاد با اندازه‌ی ۱ را پخش می‌کند و حلقه را حذف می‌کند.
- `axis=0` ستون‌ها را خرد می‌کند، `axis=1` ردیف‌ها را.
- `keepdims=True` شکل را برای broadcasting بعدی حفظ می‌کند.
- broadcasting می‌تواند بی‌صدا غلط شود؛ همیشه شکل را چاپ کن.

**سؤالات مروری:**
1. `(100,5) - (5,)` چه نتیجه‌ای می‌دهد و چرا؟
2. تفاوت `a.sum(axis=0)` و `a.sum(axis=1)` روی ماتریس ۲×۳ چیست؟
3. چرا `keepdims=True` در `mean` مهم است؟

---

## فصل ۲۴: توابع آماری، جبر خطی و تصادفی

### مقدمه

این فصل، جعبه‌ابزار کامل NumPy را می‌دهد: آمار (برای تحلیل داده)، جبر خطی (برای مدل‌ها)، و تولید تصادفی (برای بازتولیدپذیری). این‌ها همان توابعی هستند که Pandas و scikit-learn زیر پوشش استفاده می‌کنند.

### مفاهیم اساسی

**۱. توابع آماری.**

```python
import numpy as np

a = np.array([[1, 2, 3], [4, 5, 6]])

a.mean()          # 3.5
a.mean(axis=0)    # [2.5 3.5 4.5]
a.std()           # انحراف معیار
a.var()           # واریانس
a.min(), a.max()
a.argmin(), a.argmax()    # ایندکس کمینه/بیشینه
np.median(a)
np.percentile(a, [25, 50, 75])
```

**۲. توابع آماری مقاوم به NaN.**

```python
a = np.array([1.0, np.nan, 3.0, 4.0])
print(a.mean())          # nan ← آلوده می‌شود
print(np.nanmean(a))     # 2.666
print(np.nanmedian(a))   # 3.0
print(np.nansum(a))      # 8.0
```

**قاعده:** هر تابع آماری، نسخه‌ی `nan`-دار دارد: `nanmean`, `nansum`, `nanstd`, `nanpercentile`.

**۳. شمارش و یکتا.**

```python
a = np.array([1, 2, 2, 3, 3, 3])
values, counts = np.unique(a, return_counts=True)
print(values)     # [1 2 3]
print(counts)     # [1 2 3]

# احتمال تجربی هر مقدار
probs = counts / counts.sum()
print(probs)      # [0.167 0.333 0.5  ]
```

**۴. جبر خطی (`np.linalg`).**

```python
A = np.array([[1., 2.], [3., 4.]])
b = np.array([5., 6.])

np.linalg.det(A)          # دترمینان
np.linalg.inv(A)          # معکوس
np.linalg.solve(A, b)     # حل Ax = b (بهتر از inv)
np.linalg.norm(b)         # نرم L2
np.linalg.matrix_rank(A)  # رتبه
```

**نکته‌ی مهم:** برای حل `Ax = b`، از `solve` استفاده کن، نه `inv(A) @ b`. `solve` پایدارتر و سریع‌تر است.

**۵. تجزیه‌ها (Decompositions).**

```python
A = np.random.randn(5, 3)
U, S, Vt = np.linalg.svd(A)     # تجزیه‌ی مقدار منفرد — پایه‌ی PCA
Q, R = np.linalg.qr(A)          # تجزیه‌ی QR
eigvals, eigvecs = np.linalg.eig(np.array([[2., 0.], [0., 3.]]))
```

**کاربرد SVD:** کاهش بعد، فشرده‌سازی، سیستم‌های توصیه‌گر. مقادیر `S` نشان می‌دهند هر مؤلفه چقدر «انرژی» دارد (فایل ۱۰).

**۶. تولید اعداد تصادفی — و اهمیت seed.**

```python
rng = np.random.default_rng(seed=42)

rng.random(5)                     # یکنواخت [0,1)
rng.normal(0, 1, 5)               # نرمال
rng.integers(0, 10, 5)            # صحیح
rng.choice([1, 2, 3], size=5)     # نمونه‌گیری با جایگذاری
rng.permutation(10)               # جایگشت
rng.shuffle(arr)                  # درجا (in-place)
```

**چرا seed مهم است؟** چون **بازتولیدپذیری** (reproducibility) پایه‌ی علم و مهندسی است. اگر نتیجه‌ات را نتوانی بازتولید کنی، نمی‌توانی دیباگ کنی.

**۷. توزیع‌های آماده.**

```python
rng = np.random.default_rng(0)
rng.binomial(n=10, p=0.3, size=5)     # دوجمله‌ای
rng.poisson(lam=3, size=5)            # پواسون
rng.uniform(0, 1, 5)                  # یکنواخت
rng.exponential(scale=1.0, size=5)    # نمایی
```

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — رگرسیون خطی با فرمول بسته (Normal Equation):**

```python
import numpy as np

rng = np.random.default_rng(42)
X = rng.normal(size=(100, 1))
y = 3 * X[:, 0] + 2 + rng.normal(scale=0.1, size=100)

# افزودن ستون بایاس
X_b = np.c_[np.ones(len(X)), X]         # (100, 2)

# حل تحلیلی: θ = (XᵀX)⁻¹ Xᵀ y
theta = np.linalg.solve(X_b.T @ X_b, X_b.T @ y)
print("وزن‌ها:", theta.round(3))         # نزدیک [2, 3]
```

**این سه خط، رگرسیون خطی است — بدون هیچ کتابخانه‌ی ML.**

**مثال ۲ — پیاده‌سازی PCA با SVD:**

```python
import numpy as np

rng = np.random.default_rng(0)
X = rng.normal(size=(200, 5)) @ rng.normal(size=(5, 5))   # داده‌ی همبسته

# ۱. مرکز کردن
X_centered = X - X.mean(axis=0)

# ۲. SVD
U, S, Vt = np.linalg.svd(X_centered, full_matrices=False)

# ۳. نسبت واریانس توضیح‌داده‌شده
explained = S**2 / (S**2).sum()
print("نسبت واریانس:", explained.round(3))
print("واریانس تجمعی:", np.cumsum(explained).round(3))

# ۴. کاهش به ۲ بعد
X_2d = X_centered @ Vt[:2].T
print("شکل جدید:", X_2d.shape)      # (200, 2)
```

**مثال ۳ — ارزیابی مدل با آمار:**

```python
import numpy as np

y_true = np.array([3.0, 5.0, 7.0, 9.0])
y_pred = np.array([2.5, 5.5, 7.5, 8.0])

mae = np.abs(y_true - y_pred).mean()
mse = ((y_true - y_pred) ** 2).mean()
rmse = np.sqrt(mse)
ss_res = ((y_true - y_pred) ** 2).sum()
ss_tot = ((y_true - y_true.mean()) ** 2).sum()
r2 = 1 - ss_res / ss_tot

print(f"MAE={mae:.3f}  MSE={mse:.3f}  RMSE={rmse:.3f}  R²={r2:.3f}")
```

**مثال ۴ — بازتولیدپذیری:**

```python
def make_data(seed=42, n=10):
    rng = np.random.default_rng(seed)
    return rng.normal(size=n)

print(make_data())          # همیشه یکسان
print(make_data())          # همیشه یکسان
print(make_data(seed=7))    # متفاوت
```

### مقایسه و دیدگاه‌های مختلف

| API تصادفی | وضعیت | توصیه |
|---|---|---|
| `np.random.seed()` + `np.random.rand()` | قدیمی (legacy) | استفاده نکن در کد جدید |
| `np.random.default_rng()` | مدرن، سریع‌تر | ✅ توصیه‌شده |
| `random.seed()` (پایتون) | برای غیرعددی | فقط برای منطق عمومی |

**دو دیدگاه درباره‌ی seed:**
- **«seed را ثابت کن تا نتایج بازتولید شوند.»** برای دیباگ، تست و مقاله ضروری.
- **«seed را در تولید ثابت نکن.»** برای داده‌ی تصادفی واقعی (مثل انتخاب نمونه در A/B test) نباید ثابت باشد.
**قاعده:** در **آزمایش**، seed ثابت؛ در **تولید**، متفاوت.

### اشتباهات رایج

**۱. `np.linalg.inv(A) @ b` به جای `np.linalg.solve(A, b)`.** پایدارتر و سریع‌تر است.
**۲. `np.random.seed()` در وسط کد.** حالت جهانی را عوض می‌کند و نتایج بخش‌های دیگر را تغییر می‌دهد. از `rng` محلی استفاده کن.
**۳. نادیده گرفتن NaN.** `mean` با یک NaN، کل نتیجه را NaN می‌کند.
**۴. محاسبه‌ی واریانس با `ddof=0` در نمونه.** پیش‌فرض NumPy `ddof=0` است (واریانس جامعه). برای نمونه، `ddof=1` بگذار.
**۵. مقایسه‌ی `np.float32` و `np.float64` با `==`.** دقت متفاوت است؛ `np.allclose` با `rtol` مناسب.
**۶. استفاده از `svd` روی داده‌ی مرکز‌نشده برای PCA.** نتیجه غلط می‌شود؛ اول `X - X.mean(axis=0)`.

### نکات مهم و ترفندها

- **`np.c_` و `np.r_`** برای الحاق سریع: `np.c_[ones, X]` ستون اضافه می‌کند.
- **`ddof` را آگاهانه انتخاب کن.** `ddof=0` برای جامعه، `ddof=1` برای نمونه.
- **`rng = np.random.default_rng(seed)` را یک بار بساز و پاس بده.** نه این‌که در هر تابع `np.random` صدا بزنی.
- **`np.linalg.solve` را به `inv` ترجیح بده.** همیشه.
- **میان‌بر:** آمار → `mean/std/percentile` · جبر خطی → `linalg.solve/svd` · تصادفی → `default_rng(seed)`.

### خلاصه فصل

- توابع آماری NumPy، پایه‌ی تحلیل و ارزیابی‌اند؛ نسخه‌ی `nan`-دار را بشناس.
- `linalg.solve` برای حل معادلات، `svd` برای کاهش بعد.
- PCA با SVD در چند خط پیاده می‌شود.
- `default_rng(seed)` روش مدرن و بازتولیدپذیر تولید تصادفی است.
- seed در آزمایش ثابت، در تولید متفاوت.

**سؤالات مروری:**
1. چرا `solve` بهتر از `inv` است؟
2. `ddof` در `std` چه اثری دارد؟
3. چرا در PCA باید اول داده را مرکز کرد؟

---

## منابع و مراجع

**مستندات**
- NumPy Official Docs — *NumPy: the absolute basics for beginners*
- NumPy Docs — *Broadcasting* (بخش Rules، خواندنش واجب است)
- NumPy Docs — *Linear Algebra* (`numpy.linalg`)

**کتاب‌ها و دوره‌ها**
- McKinney, W. — *Python for Data Analysis*، فصل ۴ (NumPy Basics)
- VanderPlas, J. — *Python Data Science Handbook*، فصل ۲ (رایگان آنلاین)

**تمرین**
- 100 NumPy Exercises (مخزن GitHub: rougier/numpy-100)

---

**بعدی:** [`06-Pandas.md`](./06-Pandas.md) — فصل ۲۵: Series و DataFrame
