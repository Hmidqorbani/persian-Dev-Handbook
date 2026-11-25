# بخش ۱۶: شبکه عصبی از صفر

**مدت یادگیری تقریبی**: ۳.۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۱۳ (آموزش)، فایل ۰۵ (NumPy)، فایل ۰۴ (مشتق)

---

## فهرست محتوا

- [فصل ۷۱: نورون مصنوعی: از یک خط تا یک تصمیم](#فصل-۷۱-نورون-مصنوعی-از-یک-خط-تا-یک-تصمیم)
- [فصل ۷۲: تابع فعال‌سازی: چرا خطی بودن کافی نیست](#فصل-۷۲-تابع-فعالسازی-چرا-خطی-بودن-کافی-نیست)
- [فصل ۷۳: شبکه‌ی چندلایه و معماری](#فصل-۷۳-شبکهی-چندلایه-و-معماری)
- [فصل ۷۴: ساخت شبکه عصبی با NumPy](#فصل-۷۴-ساخت-شبکه-عصبی-با-numpy)
- [فصل ۷۵: آموزش شبکه در عمل](#فصل-۷۵-آموزش-شبکه-در-عمل)
- [فصل ۷۶: مشکلات آموزش شبکه و راه‌حل‌ها](#فصل-۷۶-مشکلات-آموزش-شبکه-و-راهحلها)

---

## فصل ۷۱: نورون مصنوعی: از یک خط تا یک تصمیم

### مقدمه

شبکه‌ی عصبی از میلیون‌ها پارامتر ساخته شده، اما **همه‌اش از یک واحد کوچک شروع می‌شود: نورون**.

اگر نورون را بفهمی، شبکه‌ی ۱۰۰ لایه هم چیز جدیدی ندارد — فقط تکرار سازمان‌یافته‌ی همان است.

### مفاهیم اساسی

**۱. نورون مصنوعی — سه مرحله:**

```
۱. جمع وزن‌دار:  z = w₁x₁ + w₂x₂ + ... + wₙxₙ + b = w·x + b
۲. فعال‌سازی:    a = σ(z)
۳. خروجی:        a (به نورون‌های بعدی می‌رود)
```

**۲. تشبیه زیستی (و تفاوت مهم).**

| مغز | نورون مصنوعی |
|---|---|
| دندریت‌ها ورودی می‌گیرند | `x` |
| سیناپس‌ها وزن دارند | `w` |
| جسم سلولی جمع می‌زند | `w·x + b` |
| آستانه‌ی فعال‌سازی | `σ` |
| آکسون خروجی می‌دهد | `a` |

**تفاوت مهم:** نورون زیستی بسیار پیچیده‌تر است (زمانی، شیمیایی، بازخورد). نورون مصنوعی یک **تقریب بسیار ساده** است. اما همین سادگی، مقیاس‌پذیری می‌دهد.

**۳. نورون به‌عنوان «مرز تصمیم».**

یک نورون با `w·x + b`، یک **خط** (در ۲ بعد) یا **صفحه** (در n بعد) تعریف می‌کند:
```
w·x + b = 0  ← مرز
```
- `w·x + b > 0`: یک طرف
- `w·x + b < 0`: طرف دیگر

**پس یک نورون = یک طبقه‌بند خطی.** با sigmoid، این می‌شود رگرسیون لجستیک.

**۴. چرا «شبکه»؟**

یک نورون فقط خطی جدا می‌کند. برای مسائل پیچیده، چند نورون را کنار هم می‌گذاریم و خروجی‌شان را به نورون‌های بعدی می‌دهیم. **ترکیب چند مرز خطی، مرزهای پیچیده می‌سازد.**

**۵. تعداد پارامترها:**

```
لایه با n ورودی و m نورون:  n×m + m پارامتر
```
مثال: ۷۸۴ ورودی، ۱۲۸ نورون → `784×128 + 128 = 100,480` پارامتر فقط در یک لایه.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — یک نورون، گام‌به‌گام:**

```python
import numpy as np

def sigmoid(z):
    return 1 / (1 + np.exp(-np.clip(z, -500, 500)))

# یک نمونه با ۳ ویژگی
x = np.array([0.5, -1.2, 2.0])
w = np.array([1.0, 0.5, -0.3])
b = 0.2

# مرحله ۱: جمع وزن‌دار
z = np.dot(w, x) + b
print(f"z = {w} · {x} + {b} = {z:.3f}")

# مرحله ۲: فعال‌سازی
a = sigmoid(z)
print(f"a = sigmoid({z:.3f}) = {a:.3f}")
print(f"تفسیر: احتمال {a:.1%} برای کلاس مثبت")
```

**مثال ۲ — نورون به‌عنوان مرز تصمیم (بصری):**

```python
import matplotlib.pyplot as plt

# مرز: w1*x1 + w2*x2 + b = 0
w1, w2, b = 1.0, 1.0, -1.0

x1 = np.linspace(-1, 2, 100)
x2 = -(w1 * x1 + b) / w2      # حل برای x2

plt.figure(figsize=(6, 5))
plt.plot(x1, x2, "r-", label="مرز تصمیم")
plt.fill_between(x1, x2, 2.5, alpha=0.15, color="blue", label="کلاس +")
plt.fill_between(x1, -1.5, x2, alpha=0.15, color="orange", label="کلاس −")
plt.xlim(-1, 2); plt.ylim(-1.5, 2.5)
plt.xlabel("x₁"); plt.ylabel("x₂")
plt.legend(); plt.grid(alpha=0.3)
plt.title("یک نورون = یک مرز خطی")
plt.show()
```

**مثال ۳ — محدودیت نورون تک: مسئله‌ی XOR:**

```python
# XOR: با یک خط جدا نمی‌شود
X_xor = np.array([[0, 0], [0, 1], [1, 0], [1, 1]])
y_xor = np.array([0, 1, 1, 0])

plt.figure(figsize=(5, 5))
for (x1, x2), label in zip(X_xor, y_xor):
    plt.scatter(x1, x2, s=200, c="blue" if label else "red",
                marker="o" if label else "x", linewidths=2)
plt.title("XOR — یک خط نمی‌تواند جدا کند")
plt.xlabel("x₁"); plt.ylabel("x₂")
plt.grid(alpha=0.3)
plt.show()

print("هر خطی که بکشی، یا یک ۰ و یک ۱ را با هم می‌گذارد یا هیچ.")
print("راه‌حل: ترکیب چند نورون.")
```

**مثال ۴ — حل XOR با دو نورون + یک نورون خروجی:**

```python
def relu(z):
    return np.maximum(0, z)

def solve_xor(X):
    # نورون ۱: OR
    h1 = relu(X @ np.array([1.0, 1.0]) - 0.5)
    # نورون ۲: AND
    h2 = relu(X @ np.array([1.0, 1.0]) - 1.5)
    # خروجی: OR - AND = XOR
    out = h1 - h2
    return out

print(f"{'x1':>3} {'x2':>3} {'y':>3} {'پیش‌بینی':>10}")
for (x1, x2), y in zip(X_xor, y_xor):
    pred = solve_xor(np.array([[x1, x2]]))[0]
    print(f"{x1:>3} {x2:>3} {y:>3} {pred:>10.1f}")

print("\nنتیجه: ۳ نورون، XOR را کامل حل کرد — چیزی که یک نورون نمی‌توانست.")
```

**این، مهم‌ترین درس این فصل است:** ترکیب نورون‌ها، قدرت بیان را زیاد می‌کند.

### مقایسه و دیدگاه‌های مختلف

| مدل | مرز | پارامتر |
|---|---|---|
| نورون خطی | خط | n+1 |
| نورون + sigmoid | خط (با احتمال) | n+1 |
| ۲ لایه (۲ نورون پنهان) | قطعه‌خطی | ~۱۰ |
| شبکه‌ی عمیق | هر شکل پیچیده | میلیون‌ها |

**یک دیدگاه:** «شبکه‌ی عصبی، جادو نیست؛ رگرسیون لجستیک است که چند بار تکرار شده.» در سطح ریاضی، تقریباً درست است — و همین، درک آن را ممکن می‌کند.

### اشتباهات رایج

**۱. فراموش کردن بایاس.** بدون `b`، مرز همیشه از مبدأ می‌گذرد.
**۲. انتظار از یک نورون برای مسئله‌ی غیرخطی.** XOR را نمی‌تواند حل کند.
**۳. تشبیه بیش‌ازحد به مغز.** نورون مصنوعی خیلی ساده‌تر است.
**۴. نادیده گرفتن تعداد پارامترها.** یک لایه می‌تواند میلیون‌ها پارامتر داشته باشد.
**۵. فعال‌سازی خطی در همه‌جا.** معادل یک لایه.

### نکات مهم و ترفندها

- **نورون = `σ(w·x + b)`.** همین را حفظ کن.
- **بایاس، جابه‌جایی مرز است.** حذفش نکن.
- **یک نورون، یک خط.** برای غیرخطی، چند نورون لازم است.
- **میان‌بر:** نورون = جمع وزن‌دار + فعال‌سازی.

### خلاصه فصل

- نورون: جمع وزن‌دار + بایاس + تابع فعال‌سازی.
- یک نورون، یک مرز خطی می‌سازد.
- XOR با یک نورون حل نمی‌شود؛ با سه نورون حل می‌شود.
- تعداد پارامترها = `n×m + m` در هر لایه.
- شبکه، ترکیب سازمان‌یافته‌ی نورون‌هاست.

**سؤالات مروری:**
1. سه مرحله‌ی یک نورون را بنویس.
2. چرا XOR با یک نورون حل نمی‌شود؟
3. یک لایه با ۱۰۰ ورودی و ۶۴ نورون چند پارامتر دارد؟

---

## فصل ۷۲: تابع فعال‌سازی: چرا خطی بودن کافی نیست

### مقدمه

تابع فعال‌سازی، **تنها منبع غیرخطی بودن** در شبکه است. حذفش کن، شبکه به یک رگرسیون خطی بزرگ تبدیل می‌شود.

این فصل، توابع اصلی، مزایا و معایبشان، و انتخاب درست را می‌دهد.

### مفاهیم اساسی

**۱. اثبات ساده‌ی ضرورت غیرخطی:**

```
لایه ۱: h = W₁x
لایه ۲: y = W₂h = W₂W₁x = W'x
```
هر تعداد لایه‌ی خطی = یک لایه‌ی خطی. **بدون غیرخطی، «عمق» بی‌معنی است.**

**۲. شش تابع اصلی:**

| تابع | فرمول | بازه | مشتق |
|---|---|---|---|
| **Linear** | `x` | (-∞,∞) | `1` |
| **ReLU** | `max(0,x)` | [0,∞) | `1 if x>0 else 0` |
| **Leaky ReLU** | `max(αx, x)` | (-∞,∞) | `1 if x>0 else α` |
| **Sigmoid** | `1/(1+e⁻ˣ)` | (0,1) | `σ(1-σ)` |
| **Tanh** | `(eˣ-e⁻ˣ)/(eˣ+e⁻ˣ)` | (-1,1) | `1-tanh²` |
| **GELU** | `x·Φ(x)` | (-∞,∞) | نرم |

**۳. ReLU — چرا استاندارد است؟**

- **سریع:** فقط یک مقایسه.
- **بدون محوشدگی مثبت:** مشتق ۱ برای `x>0`.
- **پراکندگی:** حدود نیمی از نورون‌ها صفر می‌شوند → شبکه‌ی تنک.

**عیب: «نورون مرده».** اگر یک نورون همیشه `x<0` ببیند، گرادیانش صفر می‌شود و هرگز بازنمی‌گردد.

**۴. Leaky ReLU — راه‌حل نورون مرده.**

```
f(x) = x if x > 0 else 0.01x
```
شیب کوچک در ناحیه‌ی منفی، گرادیان را زنده نگه می‌دارد.

**۵. Sigmoid — کجا هنوز لازم است؟**
- لایه‌ی خروجی دودویی (احتمال).
- دروازه‌ها در LSTM/GRU.
**نه** در لایه‌های پنهان عمیق (محوشدگی گرادیان).

**۶. Tanh — مرکز صفر.**

مزیت بر sigmoid: خروجی متقارن حول صفر → آموزش سریع‌تر. عیب: همان محوشدگی.

**۷. GELU — استاندارد ترانسفورمر.**

```
GELU(x) ≈ 0.5x(1 + tanh(√(2/π)(x + 0.044715x³)))
```
نرم‌تر از ReLU، در BERT و GPT استفاده می‌شود.

**۸. Softmax — فقط لایه‌ی خروجی.**

تبدیل بردار به توزیع احتمال. برای چندکلاسه.

**۹. انتخاب — جدول تصمیم:**

| موقعیت | انتخاب |
|---|---|
| لایه‌ی پنهان MLP/CNN | ReLU (یا Leaky ReLU) |
| لایه‌ی پنهان ترانسفورمر | GELU |
| لایه‌ی خروجی دودویی | Sigmoid |
| لایه‌ی خروجی چندکلاسه | Softmax |
| لایه‌ی خروجی رگرسیون | بدون فعال‌سازی |
| RNN | Tanh |

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — پیاده‌سازی و رسم همه‌ی توابع:**

```python
import numpy as np
import matplotlib.pyplot as plt

def relu(x):        return np.maximum(0, x)
def leaky_relu(x, a=0.1): return np.where(x > 0, x, a * x)
def sigmoid(x):     return 1 / (1 + np.exp(-np.clip(x, -500, 500)))
def tanh(x):        return np.tanh(x)
def gelu(x):        return 0.5 * x * (1 + np.tanh(np.sqrt(2/np.pi) * (x + 0.044715 * x**3)))

x = np.linspace(-5, 5, 400)
functions = {"ReLU": relu, "Leaky ReLU": leaky_relu,
             "Sigmoid": sigmoid, "Tanh": tanh, "GELU": gelu}

fig, axes = plt.subplots(1, 5, figsize=(18, 3.2))
for ax, (name, fn) in zip(axes, functions.items()):
    ax.plot(x, fn(x), lw=2)
    ax.axhline(0, color="gray", lw=0.5)
    ax.axvline(0, color="gray", lw=0.5)
    ax.set_title(name)
    ax.grid(alpha=0.3)
plt.tight_layout()
plt.show()
```

**مثال ۲ — محوشدگی گرادیان: مقایسه‌ی عددی:**

```python
def max_gradient(fn_grad, x_range=(-5, 5), n=1000):
    xs = np.linspace(*x_range, n)
    return np.max(np.abs(fn_grad(xs)))

sigmoid_grad = lambda x: sigmoid(x) * (1 - sigmoid(x))
tanh_grad = lambda x: 1 - np.tanh(x) ** 2
relu_grad = lambda x: (x > 0).astype(float)

print(f"{'تابع':>12} {'حداکثر مشتق':>14} {'مشتق در ۱۰ لایه':>18}")
for name, g in [("Sigmoid", sigmoid_grad), ("Tanh", tanh_grad), ("ReLU", relu_grad)]:
    m = max_gradient(g)
    print(f"{name:>12} {m:>14.3f} {m**10:>18.2e}")

print("\nSigmoid: ۰.۲۵^۱۰ ≈ ۱e-6 → گرادیان عملاً صفر")
print("ReLU: ۱^۱۰ = ۱ → گرادیان حفظ می‌شود")
```

**مثال ۳ — نورون مرده:**

```python
rng = np.random.default_rng(0)
# ورودی‌هایی که همه منفی‌اند
X = rng.normal(-5, 1, (100, 5))
W = rng.normal(size=(5, 10))
z = X @ W

print(f"درصد نورون‌های مرده (z<0 همیشه): {(z < 0).mean():.1%}")
print("→ با ReLU، این نورون‌ها گرادیان صفر دارند و هرگز یاد نمی‌گیرند.")
print("→ راه‌حل: Leaky ReLU یا مقداردهی بهتر.")
```

**مثال ۴ — انتخاب اشتباه فعال‌سازی:**

```python
from sklearn.neural_network import MLPClassifier
from sklearn.datasets import make_moons
from sklearn.model_selection import train_test_split

X, y = make_moons(n_samples=1000, noise=0.2, random_state=42)
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.25, random_state=42)

for act in ["identity", "logistic", "tanh", "relu"]:
    mlp = MLPClassifier(hidden_layer_sizes=(32, 32), activation=act,
                        max_iter=1000, random_state=42).fit(X_tr, y_tr)
    print(f"{act:>10}: test acc = {mlp.score(X_te, y_te):.3f}")

print("\nidentity = خطی → معادل رگرسیون لجستیک، نمی‌تواند ماه را یاد بگیرد.")
```

### مقایسه و دیدگاه‌های مختلف

| تابع | سرعت | محوشدگی | نورون مرده | استاندارد |
|---|---|---|---|---|
| ReLU | ⚡⚡⚡ | ندارد (x>0) | ✅ دارد | MLP/CNN |
| Leaky ReLU | ⚡⚡⚡ | ندارد | ❌ ندارد | جایگزین ReLU |
| Sigmoid | ⚡ | ⚠️ شدید | ❌ | خروجی دودویی |
| Tanh | ⚡⚡ | ⚠️ شدید | ❌ | RNN |
| GELU | ⚡⚡ | ندارد | ❌ | ترانسفورمر |
| Swish | ⚡⚡ | ندارد | ❌ | جایگزین مدرن |

**یک دیدگاه:** «ReLU بهترین است.» تقریباً درست برای MLP/CNN. اما در ترانسفورمر، GELU بهتر عمل می‌کند. **تابع فعال‌سازی، یک انتخاب مهندسی است، نه یک حقیقت مطلق.**

### اشتباهات رایج

**۱. فعال‌سازی خطی در لایه‌های پنهان.** شبکه معادل یک لایه.
**۲. sigmoid در شبکه‌ی عمیق.** محوشدگی گرادیان.
**۳. softmax در لایه‌های پنهان.** فقط خروجی.
**۴. فعال‌سازی در لایه‌ی خروجی رگرسیون.** خروجی محدود می‌شود.
**۵. ReLU بدون توجه به نورون مرده.** اگر >۳۰٪ مرده‌اند، Leaky ReLU.
**۶. تغییر فعال‌سازی بدون تغییر مقداردهی اولیه.** He برای ReLU، Xavier برای tanh.

### نکات مهم و ترفندها

- **ReLU پیش‌فرض.** اگر مشکل شد، Leaky ReLU.
- **مقداردهی اولیه را با فعال‌سازی هماهنگ کن:** He برای ReLU، Xavier برای tanh/sigmoid.
- **`nn.ReLU(inplace=True)`** در PyTorch حافظه کم می‌کند.
- **GELU برای ترانسفورمر.**
- **میان‌بر:** پنهان → ReLU · خروجی دودویی → sigmoid · خروجی چندکلاسه → softmax · رگرسیون → خطی.

### خلاصه فصل

- بدون غیرخطی، عمق بی‌معنی است.
- ReLU استاندارد لایه‌های پنهان است؛ سریع و بدون محوشدگی مثبت.
- Leaky ReLU مشکل نورون مرده را حل می‌کند.
- Sigmoid فقط در خروجی دودویی و دروازه‌ها.
- GELU استاندارد ترانسفورمر است.

**سؤالات مروری:**
1. چرا بدون فعال‌سازی غیرخطی، ۱۰ لایه معادل ۱ لایه است؟
2. نورون مرده چیست و چطور حل می‌شود؟
3. برای خروجی رگرسیون چه فعال‌سازی‌ای استفاده می‌کنی؟

---

## فصل ۷۳: شبکه‌ی چندلایه و معماری

### مقدمه

یک نورون، خط می‌کشد. چند نورون، چند خط. چند **لایه**، ترکیب خطوط — و همین، مرزهای پیچیده می‌سازد.

این فصل، از معماری تا عمق و عرض را پوشش می‌دهد: **چند لایه، چند نورون، و چرا.**

### مفاهیم اساسی

**۱. ساختار MLP (Multi-Layer Perceptron):**

```
ورودی (n ویژگی)
   ↓
لایه‌ی پنهان ۱ (h₁ نورون) + فعال‌سازی
   ↓
لایه‌ی پنهان ۲ (h₂ نورون) + فعال‌سازی
   ↓
...
   ↓
لایه‌ی خروجی (k نورون)
```

**۲. سه نوع لایه:**

| لایه | نقش | فعال‌سازی |
|---|---|---|
| **ورودی** | دریافت ویژگی | ندارد |
| **پنهان** | تبدیل و ترکیب | ReLU/GELU |
| **خروجی** | تولید پیش‌بینی | sigmoid/softmax/خطی |

**۳. عمق در برابر عرض:**

| | عمق بیشتر | عرض بیشتر |
|---|---|---|
| توان بیان | بالاتر (سلسله‌مراتبی) | متوسط |
| پارامترها | کمتر برای همان توان | بیشتر |
| آموزش | سخت‌تر (محوشدگی) | آسان‌تر |
| سرعت | کندتر | سریع‌تر |

**قضیه‌ی تقریب جهانی:** یک شبکه با **یک لایه‌ی پنهان و به‌اندازه‌ی کافی نورون**، می‌تواند هر تابع پیوسته را تقریب بزند. **اما** تعداد نورون‌های لازم می‌تواند نجومی باشد. **عمق، کارآمدتر است.**

**۴. چرا عمق بهتر است؟**

شبکه‌ی عمیق، **ویژگی‌های سلسله‌مراتبی** می‌سازد:
- لایه ۱: لبه
- لایه ۲: بافت
- لایه ۳: قسمت
- لایه ۴: شیء

هر لایه، از ترکیب لایه‌ی قبل ساخته می‌شود. این «ترکیب‌پذیری»، بازدهی نمایی می‌دهد.

**۵. انتخاب معماری — راهنمای عملی:**

| نوع داده | معماری پیشنهادی |
|---|---|
| جدولی | MLP کوچک (۲–۳ لایه، ۳۲–۲۵۶ نورون) یا boosting |
| تصویر | CNN (ResNet، EfficientNet) |
| متن | Transformer |
| سری زمانی | RNN/LSTM، Transformer، یا boosting با lag |
| صدا | CNN + RNN، یا Whisper |

**۶. تعداد پارامترها — محاسبه:**

```
لایه ۱: n_in × h + h
لایه ۲: h × h + h
لایه ۳: h × n_out + n_out
```

مثال: ۷۸۴ → ۲۵۶ → ۱۲۸ → ۱۰:
```
784×256 + 256 = 200,960
256×128 + 128 = 32,896
128×10 + 10   = 1,290
جمع ≈ 235,146
```

**۷. قانون سرانگشتی برای اندازه:**
- داده‌ی کم (<۱۰۰۰): شبکه‌ی کوچک (۱–۲ لایه، ۱۶–۶۴ نورون) یا مدل کلاسیک.
- داده‌ی متوسط (۱۰k–۱۰۰k): ۲–۳ لایه، ۶۴–۲۵۶.
- داده‌ی زیاد (>۱۰۰k): عمیق‌تر، یا transfer learning.

**۸. معماری‌های خاص:**
- **CNN:** برای تصویر (فایل ۱۷).
- **RNN/LSTM:** برای دنباله.
- **Transformer:** برای متن و هر چیز ترتیبی.
- **Autoencoder:** برای فشرده‌سازی و تشخیص ناهنجاری.
- **GAN:** برای تولید.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — محاسبه‌ی پارامترها:**

```python
def count_params(layers):
    """layers: لیستی از (n_in, n_out) برای هر لایه‌ی خطی."""
    total = 0
    details = []
    for i, (n_in, n_out) in enumerate(layers, 1):
        p = n_in * n_out + n_out
        total += p
        details.append(f"لایه {i}: {n_in}×{n_out} + {n_out} = {p:,}")
    return total, details

architectures = {
    "کوچک (784-64-10)": [(784, 64), (64, 10)],
    "متوسط (784-256-128-10)": [(784, 256), (256, 128), (128, 10)],
    "بزرگ (784-512-256-128-10)": [(784, 512), (512, 256), (256, 128), (128, 10)],
}

for name, layers in architectures.items():
    total, details = count_params(layers)
    print(f"\n{name}: {total:,} پارامتر")
    for d in details:
        print("  " + d)
```

**مثال ۲ — اثر عمق بر توان بیان (شبکه‌های مختلف روی داده‌ی غیرخطی):**

```python
from sklearn.neural_network import MLPClassifier
from sklearn.datasets import make_circles
from sklearn.model_selection import cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import make_pipeline

X, y = make_circles(n_samples=1000, factor=0.4, noise=0.1, random_state=42)

architectures = {
    "بدون لایه پنهان": (),
    "1×8": (8,),
    "1×32": (32,),
    "2×32": (32, 32),
    "3×64": (64, 64, 64),
}

for name, hidden in architectures.items():
    pipe = make_pipeline(StandardScaler(),
                         MLPClassifier(hidden_layer_sizes=hidden, max_iter=2000,
                                       random_state=42))
    score = cross_val_score(pipe, X, y, cv=5).mean()
    n_params = sum(pipe.named_steps["mlpclassifier"].coefs_[i].size +
                   pipe.named_steps["mlpclassifier"].intercepts_[i].size
                   for i in range(len(pipe.named_steps["mlpclassifier"].coefs_)))
    print(f"{name:>18}: acc={score:.3f} | پارامتر={n_params:>6,}")
```

**خروجی معمول:** بدون لایه پنهان ~۰.۵ (نمی‌تواند)، با ۲ لایه ~۰.۹۸.

**مثال ۳ — معماری با PyTorch (شفاف و خوانا):**

```python
try:
    import torch
    import torch.nn as nn

    class MLP(nn.Module):
        def __init__(self, n_in, hidden=(256, 128), n_out=10, dropout=0.2):
            super().__init__()
            layers = []
            prev = n_in
            for h in hidden:
                layers += [nn.Linear(prev, h), nn.ReLU(), nn.Dropout(dropout)]
                prev = h
            layers.append(nn.Linear(prev, n_out))
            self.net = nn.Sequential(*layers)

        def forward(self, x):
            return self.net(x)

    model = MLP(784)
    print(model)
    print(f"\nتعداد پارامترها: {sum(p.numel() for p in model.parameters()):,}")
except ImportError:
    print("pip install torch")
```

**مثال ۴ — انتخاب معماری بر اساس حجم داده:**

```python
def suggest_architecture(n_samples, n_features, n_classes, task="tabular"):
    if task == "image":
        return "CNN (ResNet18 با transfer learning)"
    if task == "text":
        return "Transformer (fine-tune مدل از پیش آموزش‌دیده)"

    if n_samples < 1000:
        return "MLP کوچک: (32,) یا (64, 32) — یا بهتر: Gradient Boosting"
    if n_samples < 20_000:
        return "MLP متوسط: (128, 64) با dropout=0.2"
    if n_samples < 200_000:
        return "MLP بزرگ: (256, 128, 64) با dropout=0.3 + BatchNorm"
    return "MLP عمیق یا Transformer + transfer learning"

for n in [500, 5000, 50_000, 500_000]:
    print(f"{n:>8} نمونه → {suggest_architecture(n, 20, 2)}")
```

### مقایسه و دیدگاه‌های مختلف

| معماری | پارامتر (تقریبی) | توان | کِی |
|---|---|---|---|
| Logistic | n+1 | خطی | baseline |
| MLP 1 لایه | 10³ | متوسط | داده‌ی کوچک |
| MLP 3 لایه | 10⁵ | بالا | داده‌ی متوسط |
| ResNet-50 | 25M | خیلی بالا | تصویر |
| GPT-3 | 175B | عظیم | متن |

**دو دیدگاه:**
- **«عمیق‌تر همیشه بهتر است.»** نه: با داده‌ی کم، عمق بیشتر = overfit سریع‌تر.
- **«یک لایه‌ی پنهان کافی است.»** از نظر تئوری درست (تقریب جهانی)، اما در عمل ناکارآمد.

**جمع:** عمق را با حجم داده تنظیم کن، نه با جاه‌طلبی.

### اشتباهات رایج

**۱. معماری بزرگ برای داده‌ی کوچک.** overfit فوری.
**۲. معماری کوچک برای مسئله‌ی پیچیده.** underfit.
**۳. فراموش کردن لایه‌ی خروجی مناسب.** `n_classes` نورون با softmax.
**۴. نبود dropout/BatchNorm در شبکه‌ی عمیق.** آموزش ناپایدار.
**۵. کپی معماری مقاله بدون توجه به داده.** آن‌ها ۱.۲M تصویر داشتند، تو ۲۰۰۰.
**۶. محاسبه‌ی نادرست پارامترها.** همیشه `sum(p.numel())` بزن.

### نکات مهم و ترفندها

- **از کوچک شروع کن، بزرگ کن.** برعکسش سخت‌تر است.
- **قاعده: پارامترها را با حجم داده متناسب کن.** ۱۰ پارامتر به ازای هر نمونه، سقف تقریبی.
- **BatchNorm بعد از Linear، قبل از ReLU.**
- **Dropout در لایه‌های با پارامتر زیاد.**
- **میان‌بر:** ورودی → (Linear+ReLU)* → Linear → خروجی.

### خلاصه فصل

- MLP از لایه‌های خطی + غیرخطی ساخته می‌شود.
- تقریب جهانی با یک لایه ممکن است، اما عمق کارآمدتر است.
- عمق، ویژگی‌های سلسله‌مراتبی می‌سازد.
- معماری را با حجم داده تنظیم کن.
- تعداد پارامتر = `Σ(n_in×n_out + n_out)`.

**سؤالات مروری:**
1. چرا عمق از عرض کارآمدتر است؟
2. یک شبکه‌ی ۲۰ → ۱۲۸ → ۶۴ → ۳ چند پارامتر دارد؟
3. برای ۵۰۰ نمونه، چه معماری‌ای پیشنهاد می‌کنی؟

---

## فصل ۷۴: ساخت شبکه عصبی با NumPy

### مقدمه

این فصل، **قلب عملی بخش شبکه عصبی** است: یک شبکه‌ی کامل، از صفر، با NumPy. بدون PyTorch، بدون TensorFlow.

اگر این فصل را کامل اجرا کنی، دیگر هیچ کتابخانه‌ای برایت جادو نیست.

### مفاهیم اساسی

**۱. طراحی کلاس:**

```
Network
├── __init__(layer_sizes)      ← ساخت وزن‌ها
├── forward(X)                 ← پیش‌بینی + cache
├── backward(y)                ← گرادیان‌ها
├── step(lr)                   ← به‌روزرسانی
├── fit(X, y, epochs)          ← حلقه‌ی آموزش
└── predict(X)                 ← استنتاج
```

**۲. مقداردهی اولیه — مهم است.**

- **صفر:** فاجعه (همه نورون‌ها یکسان می‌مانند).
- **تصادفی کوچک:** کار می‌کند.
- **Xavier/Glorot:** `σ = √(2/(n_in+n_out))` — برای tanh/sigmoid.
- **He:** `σ = √(2/n_in)` — برای ReLU.

**۳. شکل‌ها:**

```
W[l]: (n_in, n_out)
b[l]: (n_out,)
Z[l]: (batch, n_out)
A[l]: (batch, n_out)
```

**۴. الگوریتم کامل:**

```
for epoch:
  for batch:
    ۱. A = forward(X_batch)          # ذخیره‌ی Z و A
    ۲. loss = cross_entropy(y, A)
    ۳. dZ = A - y_onehot             # گرادیان ساده‌شده
    ۴. for l از آخر به اول:
         dW[l] = A[l-1].T @ dZ[l]
         db[l] = sum(dZ[l])
         dA[l-1] = dZ[l] @ W[l].T
         dZ[l-1] = dA[l-1] * relu_grad(Z[l-1])
    ۵. W -= lr * dW
```

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — شبکه‌ی کامل، کلاس‌محور:**

```python
import numpy as np

class NeuralNetwork:
    """شبکه‌ی عصبی چندلایه از صفر — پشتیبانی از ReLU و softmax."""

    def __init__(self, layer_sizes, seed=42):
        rng = np.random.default_rng(seed)
        self.params = {}
        self.L = len(layer_sizes) - 1

        for l in range(1, self.L + 1):
            n_in, n_out = layer_sizes[l - 1], layer_sizes[l]
            # مقداردهی He برای ReLU
            self.params[f"W{l}"] = rng.normal(0, np.sqrt(2 / n_in), (n_in, n_out))
            self.params[f"b{l}"] = np.zeros(n_out)

    # ---------- فعال‌سازی ----------
    @staticmethod
    def relu(z):
        return np.maximum(0, z)

    @staticmethod
    def relu_grad(z):
        return (z > 0).astype(float)

    @staticmethod
    def softmax(z):
        z = z - z.max(axis=1, keepdims=True)
        e = np.exp(z)
        return e / e.sum(axis=1, keepdims=True)

    # ---------- forward ----------
    def forward(self, X):
        cache = {"A0": X}
        A = X
        for l in range(1, self.L + 1):
            Z = A @ self.params[f"W{l}"] + self.params[f"b{l}"]
            cache[f"Z{l}"] = Z
            A = self.softmax(Z) if l == self.L else self.relu(Z)
            cache[f"A{l}"] = A
        return A, cache

    # ---------- loss ----------
    @staticmethod
    def cross_entropy(y_onehot, probs):
        eps = 1e-12
        return -np.mean(np.sum(y_onehot * np.log(probs + eps), axis=1))

    # ---------- backward ----------
    def backward(self, y_onehot, cache):
        grads = {}
        m = y_onehot.shape[0]

        # گرادیان لایه‌ی خروجی (softmax + CE → A - y)
        dZ = (cache[f"A{self.L}"] - y_onehot) / m

        for l in range(self.L, 0, -1):
            grads[f"dW{l}"] = cache[f"A{l-1}"].T @ dZ
            grads[f"db{l}"] = dZ.sum(axis=0)

            if l > 1:
                dA_prev = dZ @ self.params[f"W{l}"].T
                dZ = dA_prev * self.relu_grad(cache[f"Z{l-1}"])

        return grads

    # ---------- update ----------
    def step(self, grads, lr):
        for l in range(1, self.L + 1):
            self.params[f"W{l}"] -= lr * grads[f"dW{l}"]
            self.params[f"b{l}"] -= lr * grads[f"db{l}"]

    # ---------- آموزش ----------
    def fit(self, X, y, epochs=100, lr=0.1, batch_size=64, verbose=True):
        n_classes = self.params[f"W{self.L}"].shape[1]
        y_onehot = np.eye(n_classes)[y]
        history = []
        rng = np.random.default_rng(0)

        for epoch in range(epochs):
            idx = rng.permutation(len(X))
            epoch_loss = 0.0
            n_batches = 0

            for start in range(0, len(X), batch_size):
                batch_idx = idx[start:start + batch_size]
                Xb, yb = X[batch_idx], y_onehot[batch_idx]

                probs, cache = self.forward(Xb)
                epoch_loss += self.cross_entropy(yb, probs)
                n_batches += 1

                grads = self.backward(yb, cache)
                self.step(grads, lr)

            avg_loss = epoch_loss / n_batches
            history.append(avg_loss)
            if verbose and epoch % 10 == 0:
                acc = self.score(X, y)
                print(f"epoch {epoch:3d} | loss={avg_loss:.4f} | acc={acc:.3f}")

        return history

    def predict_proba(self, X):
        probs, _ = self.forward(X)
        return probs

    def predict(self, X):
        return self.predict_proba(X).argmax(axis=1)

    def score(self, X, y):
        return (self.predict(X) == y).mean()
```

**مثال ۲ — آموزش روی داده‌ی واقعی:**

```python
from sklearn.datasets import load_digits
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

digits = load_digits()
X, y = digits.data, digits.target      # ۱۷۹۷ نمونه، ۶۴ ویژگی، ۱۰ کلاس

X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)

scaler = StandardScaler().fit(X_tr)
X_tr_s, X_te_s = scaler.transform(X_tr), scaler.transform(X_te)

net = NeuralNetwork([64, 128, 64, 10])
history = net.fit(X_tr_s, y_tr, epochs=100, lr=0.1, batch_size=64)

print(f"\nدقت train: {net.score(X_tr_s, y_tr):.3f}")
print(f"دقت test:  {net.score(X_te_s, y_te):.3f}")
```

**انتظار:** دقت تست بالای ۰.۹۵. **این شبکه، از صفر نوشته شده و هیچ کتابخانه‌ای ندارد.**

**مثال ۳ — بررسی درستی گرادیان (gradient checking):**

```python
def gradient_check(net, X, y, eps=1e-6):
    """مقایسه‌ی گرادیان تحلیلی با عددی."""
    n_classes = net.params[f"W{net.L}"].shape[1]
    y_onehot = np.eye(n_classes)[y]

    probs, cache = net.forward(X)
    grads = net.backward(y_onehot, cache)

    key = "W1"
    W = net.params[key]
    analytic = grads["dW1"]

    # چند عنصر تصادفی را بررسی کن
    rng = np.random.default_rng(0)
    errors = []
    for _ in range(5):
        i, j = rng.integers(0, W.shape[0]), rng.integers(0, W.shape[1])
        original = W[i, j]

        W[i, j] = original + eps
        lp = net.cross_entropy(y_onehot, net.forward(X)[0])
        W[i, j] = original - eps
        lm = net.cross_entropy(y_onehot, net.forward(X)[0])
        W[i, j] = original

        numeric = (lp - lm) / (2 * eps)
        rel_err = abs(numeric - analytic[i, j]) / max(1e-8, abs(numeric) + abs(analytic[i, j]))
        errors.append(rel_err)

    return max(errors)

net_small = NeuralNetwork([64, 16, 10])
err = gradient_check(net_small, X_tr_s[:64], y_tr[:64])
print(f"حداکثر خطای نسبی: {err:.2e}")
print("✅ درست است" if err < 1e-6 else "❌ گرادیان اشتباه است")
```

**این تست، طلای دیباگ backprop دستی است.** اگر عدد بزرگ بود، ترانهاده یا علامت را چک کن.

**مثال ۴ — رسم منحنی loss:**

```python
import matplotlib.pyplot as plt

plt.figure(figsize=(8, 4))
plt.plot(history, lw=2)
plt.xlabel("epoch")
plt.ylabel("cross-entropy loss")
plt.title("منحنی آموزش شبکه از صفر")
plt.grid(alpha=0.3)
plt.show()
```

### مقایسه و دیدگاه‌های مختلف

| سطح | کد | سرعت | کنترل |
|---|---|---|---|
| NumPy دستی | ~۱۰۰ خط | کند | کامل |
| PyTorch | ~۱۵ خط | سریع (GPU) | زیاد |
| Keras | ~۵ خط | سریع | کم |

**یک دیدگاه:** «چرا از صفر بنویسم وقتی PyTorch هست؟» چون **یک بار نوشتن، درک دائمی می‌دهد**. بعد از این، `loss.backward()` یک دستور جادویی نیست؛ می‌دانی چه می‌کند.

### اشتباهات رایج

**۱. مقداردهی صفر.** همه‌ی نورون‌ها یکسان می‌مانند.
**۲. فراموش کردن `/m` در گرادیان.** گرادیان به batch size وابسته می‌شود.
**۳. اشتباه در ترانهاده.** `A.T @ dZ` درست است، نه `A @ dZ`.
**۴. اعمال softmax و cross-entropy جداگانه.** گرادیان `A - y` ساده‌تر و پایدارتر است.
**۵. نبود cache.** نمی‌توان گرادیان را حساب کرد.
**۶. lr بزرگ.** loss می‌پرد.
**۷. استفاده از این شبکه در تولید.** کند و بدون بهینه‌سازی. فقط برای یادگیری.

### نکات مهم و ترفندها

- **مقداردهی He برای ReLU.** `√(2/n_in)`.
- **`dZ = A - y` برای softmax+CE.** ساده و درست.
- **Gradient checking را یک بار انجام بده.** نجات‌دهنده است.
- **batch_size=64، lr=0.1** نقطه‌ی شروع خوبی است.
- **میان‌بر:** forward → loss → backward → step → تکرار.

### خلاصه فصل

- یک شبکه‌ی کامل با ~۱۰۰ خط NumPy نوشته می‌شود.
- مقداردهی He، برای ReLU حیاتی است.
- `dZ = A - y` گرادیان softmax+CE است.
- gradient checking درستی backprop را تأیید می‌کند.
- بعد از این فصل، کتابخانه‌ها دیگر جادو نیستند.

**سؤالات مروری:**
1. چرا مقداردهی صفر بد است؟
2. گرادیان softmax + cross-entropy چه شکل ساده‌ای دارد؟
3. gradient checking چه کاری انجام می‌دهد؟

---

## فصل ۷۵: آموزش شبکه در عمل

### مقدمه

نوشتن شبکه، نیمی از کار است. **آموزش دادنش**، نیمه‌ی دیگر — و بخش سخت‌تر.

این فصل، مهارت‌های عملی آموزش را می‌دهد: از انتخاب lr تا تشخیص مشکلات.

### مفاهیم اساسی

**۱. دستور کار استاندارد:**

```
۱. داده را مقیاس‌بندی کن (StandardScaler)
۲. شبکه‌ی کوچک بساز
۳. lr را با range test پیدا کن
۴. با early stopping آموزش بده
۵. منحنی loss را ببین
۶. اگر overfit → regularization
   اگر underfit → بزرگ‌تر کن
```

**۲. هفت مشکل و نشانه‌هایشان:**

| نشانه | مشکل | راه‌حل |
|---|---|---|
| loss ثابت | lr خیلی کم، یا شبکه‌ی خراب | lr ↑، مقداردهی را چک کن |
| loss می‌پرد | lr بزرگ | lr ↓ (۱۰×) |
| loss = NaN | lr بزرگ، تقسیم بر صفر | lr ↓، gradient clipping |
| train خوب، test بد | overfit | dropout، داده بیشتر |
| هر دو بد | underfit | شبکه بزرگ‌تر، ویژگی بهتر |
| loss کاهش می‌یابد اما کند | lr کوچک | lr ↑ |
| دقت ثابت روی یک کلاس | داده‌ی نامتوازن | class_weight |

**۳. مقیاس‌بندی — چرا حیاتی است؟**

بدون مقیاس‌بندی، ویژگی‌های با بازه‌ی بزرگ، گرادیان‌های بزرگ تولید می‌کنند و آموزش ناپایدار می‌شود.

**همیشه `StandardScaler` (یا `MinMaxScaler` برای تصویر).**

**۴. Batch Normalization.**

```python
nn.BatchNorm1d(hidden_size)
```
- آموزش را سریع‌تر می‌کند.
- به lr حساسیت را کم می‌کند.
- اثر regularization دارد.

**۵. Early Stopping با patience.**

```python
if val_loss < best - min_delta:
    best = val_loss; wait = 0
else:
    wait += 1
    if wait >= patience: break
```

**۶. ذخیره‌ی بهترین مدل، نه آخرین.**

```python
if val_loss < best_val:
    torch.save(model.state_dict(), "best.pt")
```

**۷. بازتولیدپذیری:**

```python
torch.manual_seed(42)
np.random.seed(42)
random.seed(42)
torch.backends.cudnn.deterministic = True
```

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — حلقه‌ی آموزش کامل و درست (PyTorch):**

```python
try:
    import torch
    import torch.nn as nn
    from torch.utils.data import DataLoader, TensorDataset
    import copy

    def train_model(model, X_train, y_train, X_val, y_val,
                    epochs=100, lr=1e-3, batch_size=64, patience=10, device="cpu"):
        model = model.to(device)
        train_ds = TensorDataset(
            torch.tensor(X_train, dtype=torch.float32),
            torch.tensor(y_train, dtype=torch.long),
        )
        val_X = torch.tensor(X_val, dtype=torch.float32).to(device)
        val_y = torch.tensor(y_val, dtype=torch.long).to(device)
        loader = DataLoader(train_ds, batch_size=batch_size, shuffle=True)

        criterion = nn.CrossEntropyLoss()
        optimizer = torch.optim.AdamW(model.parameters(), lr=lr, weight_decay=1e-4)

        best_val, best_state, wait = float("inf"), None, 0
        history = {"train": [], "val": []}

        for epoch in range(epochs):
            # ---- آموزش ----
            model.train()
            total_loss = 0.0
            for xb, yb in loader:
                xb, yb = xb.to(device), yb.to(device)
                optimizer.zero_grad()
                loss = criterion(model(xb), yb)
                loss.backward()
                torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
                optimizer.step()
                total_loss += loss.item() * len(xb)

            train_loss = total_loss / len(train_ds)

            # ---- ارزیابی ----
            model.eval()
            with torch.no_grad():
                val_loss = criterion(model(val_X), val_y).item()
                val_acc = (model(val_X).argmax(1) == val_y).float().mean().item()

            history["train"].append(train_loss)
            history["val"].append(val_loss)

            # ---- early stopping ----
            if val_loss < best_val - 1e-4:
                best_val, wait = val_loss, 0
                best_state = copy.deepcopy(model.state_dict())
            else:
                wait += 1

            if epoch % 10 == 0:
                print(f"epoch {epoch:3d} | train={train_loss:.4f} | "
                      f"val={val_loss:.4f} | val_acc={val_acc:.3f}")

            if wait >= patience:
                print(f"⏹ early stopping در epoch {epoch} (بهترین val={best_val:.4f})")
                break

        model.load_state_dict(best_state)
        return model, history
except ImportError:
    print("pip install torch")
```

**این تابع، الگوی استاندارد آموزش است.** هر بخشش را باید بتوانی توضیح بدهی.

**مثال ۲ — تشخیص مشکلات از منحنی:**

```python
def diagnose(history):
    train, val = history["train"], history["val"]

    if any(not np.isfinite(v) for v in train):
        return "🚨 NaN — lr را ۱۰ برابر کم کن"
    if train[-1] > train[0] * 0.95:
        return "⚠️ loss تقریباً ثابت — lr خیلی کم یا مشکل داده"
    if val[-1] > min(val) * 1.2:
        return "⚠️ overfitting — early stopping، dropout، داده بیشتر"
    if train[-1] > 0.5:
        return "⚠️ underfitting — شبکه بزرگ‌تر یا ویژگی بهتر"
    return "✅ آموزش سالم"

# مثال
fake = {"train": list(np.linspace(1.5, 0.1, 50)),
        "val": list(np.r_[np.linspace(1.6, 0.3, 30), np.linspace(0.3, 0.6, 20)])}
print(diagnose(fake))
```

**مثال ۳ — LR Range Test:**

```python
def lr_finder(model, loader, criterion, start=1e-6, end=1.0, steps=100):
    optimizer = torch.optim.SGD(model.parameters(), lr=start)
    mult = (end / start) ** (1 / steps)
    lr, results = start, []
    best_loss = float("inf")

    for step, (xb, yb) in enumerate(loader):
        if step >= steps:
            break
        for g in optimizer.param_groups:
            g["lr"] = lr

        optimizer.zero_grad()
        loss = criterion(model(xb), yb)
        loss.backward()
        optimizer.step()

        results.append((lr, loss.item()))
        if loss.item() < best_loss:
            best_loss = loss.item()
        if loss.item() > best_loss * 4:
            break        # loss منفجر شد → توقف
        lr *= mult

    return results

# استفاده: بهترین lr ≈ lr در کمترین loss، تقسیم بر ۱۰
```

**مثال ۴ — چک‌لیست پیش از آموزش:**

```python
def pre_training_checklist(X_train, y_train, X_val, y_val, model):
    checks = []

    # ۱. مقیاس
    means = np.abs(X_train.mean(axis=0)).max()
    stds = X_train.std(axis=0)
    checks.append(("✅ مقیاس‌بندی" if means < 0.5 and stds.max() < 3
                   else "⚠️ داده مقیاس‌بندی نشده", f"max|mean|={means:.2f}, max std={stds.max():.2f}"))

    # ۲. گمشده
    n_missing = int(np.isnan(X_train).sum())
    checks.append(("✅ بدون NaN" if n_missing == 0 else f"🚨 {n_missing} NaN",
                   "imputation لازم است"))

    # ۳. توازن کلاس
    counts = np.bincount(y_train)
    ratio = counts.min() / counts.max()
    checks.append(("✅ متعادل" if ratio > 0.3 else "⚠️ نامتوازن",
                   f"نسبت={ratio:.2f}"))

    # ۴. تعداد پارامتر
    n_params = sum(p.numel() for p in model.parameters())
    ratio_p = n_params / len(X_train)
    checks.append(("✅ اندازه مناسب" if ratio_p < 50 else "⚠️ پارامتر زیاد",
                   f"{n_params:,} پارامتر / {len(X_train):,} نمونه"))

    # ۵. همپوشانی train/val
    checks.append(("ℹ️", f"train={len(X_train)}, val={len(X_val)}"))

    for status, detail in checks:
        print(f"{status:35s} {detail}")

# pre_training_checklist(X_tr_s, y_tr, X_te_s, y_te, model)
```

### مقایسه و دیدگاه‌های مختلف

| ترفند | اثر | هزینه |
|---|---|---|
| BatchNorm | آموزش سریع‌تر، پایدارتر | کمی کندتر در استنتاج |
| Gradient clipping | جلوگیری از انفجار | صفر |
| Early stopping | جلوگیری از overfit | صفر |
| LR scheduler | همگرایی بهتر | صفر |
| Mixed precision | ۲× سریع‌تر | کمی پیچیده |

**یک دیدگاه:** «آموزش شبکه، هنر است.» نیمه‌درست: تجربه مهم است، اما **چک‌لیست و منحنی**، آن را از هنر به مهندسی تبدیل می‌کند.

### اشتباهات رایج

**۱. فراموش کردن `model.train()` / `model.eval()`.** BatchNorm و Dropout رفتار غلط.
**۲. مقیاس‌بندی نکردن.** آموزش ناپایدار.
**۳. lr پیش‌فرض بدون تست.** ممکن است ۱۰۰ برابر دور از بهینه باشد.
**۴. ذخیره‌ی مدل آخر به جای بهترین.** overfit.
**۵. نبود validation جدا.** نمی‌دانی کِی متوقف کنی.
**۶. `zero_grad()` فراموش شده.** گرادیان‌ها انباشته می‌شوند.
**۷. نادیده گرفتن زمان.** مدلی که ۱۰ ساعت آموزش می‌بیند و ۱٪ بهتر است، ارزش ندارد.

### نکات مهم و ترفندها

- **مقیاس‌بندی، اولین و مهم‌ترین قدم.**
- **LR range test، سریع‌ترین راه پیدا کردن lr.**
- **`clip_grad_norm_(params, 1.0)` را همیشه بگذار.** تقریباً رایگان است.
- **`AdamW` + `weight_decay=1e-4`** نقطه‌ی شروع خوبی است.
- **همیشه best model را ذخیره کن.**
- **میان‌بر:** مقیاس‌بندی → lr → early stopping → منحنی → تنظیم.

### خلاصه فصل

- آموزش، چرخه‌ای است که باید با چک‌لیست اجرا شود.
- مقیاس‌بندی، پیش‌نیاز پایداری است.
- lr مهم‌ترین فراپارامتر است؛ با range test پیدا کن.
- Early stopping و ذخیره‌ی بهترین مدل، استاندارد است.
- منحنی loss، تشخیص را ممکن می‌کند.

**سؤالات مروری:**
1. چرا مقیاس‌بندی برای شبکه‌ی عصبی حیاتی است؟
2. اگر loss NaN شد، سه اقدام به ترتیب چیست؟
3. چرا باید بهترین مدل را ذخیره کرد، نه آخرین؟

---

## فصل ۷۶: مشکلات آموزش شبکه و راه‌حل‌ها

### مقدمه

این فصل، **کتابچه‌ی عیب‌یابی** شبکه‌های عصبی است. هر مشکلی که در آموزش می‌بینی، احتمالاً یکی از این ده مورد است.

هدف: وقتی مدلت آموزش نمی‌بیند، بدانی **کجا را نگاه کنی** — نه این‌که تصادفی همه‌چیز را عوض کنی.

### مفاهیم اساسی

**۱. محوشدگی گرادیان (Vanishing Gradient).**

- **نشانه:** لایه‌های اولیه تغییر نمی‌کنند؛ loss کند کاهش می‌یابد.
- **علت:** ضرب مشتق‌های کوچک در عمق.
- **راه‌حل:** ReLU، BatchNorm، residual connection، مقداردهی He.

**۲. انفجار گرادیان (Exploding Gradient).**

- **نشانه:** loss = NaN یا نوسان شدید.
- **علت:** ضرب مشتق‌های بزرگ.
- **راه‌حل:** gradient clipping، lr کوچک‌تر، مقداردهی بهتر.

**۳. نورون مرده (Dying ReLU).**

- **نشانه:** بخش زیادی از نورون‌ها همیشه صفر؛ loss گیر می‌کند.
- **علت:** `z<0` همیشگی → گرادیان صفر.
- **راه‌حل:** Leaky ReLU، lr کوچک‌تر، مقداردهی بهتر.

**۴. هم‌انطباقی (Co-adaptation).**

- **نشانه:** overfit سریع با داده‌ی کم.
- **راه‌حل:** Dropout، weight decay، داده بیشتر.

**۵. عدم تعادل کلاس.**

- **نشانه:** مدل همه را کلاس اکثریت پیش‌بینی می‌کند.
- **راه‌حل:** `class_weight`, focal loss, resampling.

**۶. اشباع sigmoid/tanh.**

- **نشانه:** گرادیان نزدیک صفر در لایه‌های عمیق.
- **راه‌حل:** ReLU، BatchNorm.

**۷. مشکل مقداردهی اولیه.**

- **نشانه:** loss از ابتدا خیلی بالا یا NaN.
- **راه‌حل:** He/Xavier، نه صفر و نه خیلی بزرگ.

**۸. lr نامناسب.**

- **نشانه:** واگرایی یا همگرایی بسیار کند.
- **راه‌حل:** LR range test.

**۹. overfitting.**

- **نشانه:** شکاف train/val.
- **راه‌حل:** فایل ۱۵.

**۱۰. باگ در کد.**

- **نشانه:** رفتار غیرقابل توضیح، نتایج متناقض.
- **راه‌حل:** gradient checking، داده‌ی ساختگی، تست کوچک.

**۱۱. جدول عیب‌یابی جامع:**

| نشانه | احتمال ۱ | احتمال ۲ | احتمال ۳ |
|---|---|---|---|
| loss ثابت | lr کم | داده‌ی خراب | باگ در گرادیان |
| loss NaN | lr بزرگ | log(0) | تقسیم بر صفر |
| train↑ val↑ | overfit | lr بزرگ | داده‌ی نویزی |
| دقت ۵۰٪ در دودویی | برچسب‌ها برعکس | ویژگی بی‌ربط | نشت معکوس |
| همه یک کلاس | نامتوازن | lr بزرگ | آستانه‌ی غلط |
| نوسان شدید | batch کوچک | lr بزرگ | داده‌ی ناهمگون |

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — تشخیص محوشدگی گرادیان:**

```python
import numpy as np

def grad_norms_by_layer(model_params, grads):
    """نرم گرادیان هر لایه — ابزار تشخیص محوشدگی."""
    for l in range(1, len(model_params) // 2 + 1):
        key = f"dW{l}"
        if key in grads:
            print(f"لایه {l}: ||dW|| = {np.linalg.norm(grads[key]):.2e}")

# اگر نرم‌ها از آخر به اول به‌شدت کم شوند → محوشدگی
```

**مثال ۲ — انفجار گرادیان و clipping:**

```python
def clip_gradients(grads, max_norm=1.0):
    """مقیاس‌بندی گرادیان‌ها اگر نرم کل از max_norm بیشتر باشد."""
    total_norm = np.sqrt(sum(np.sum(g ** 2) for g in grads.values()))
    if total_norm > max_norm:
        scale = max_norm / (total_norm + 1e-6)
        grads = {k: v * scale for k, v in grads.items()}
        print(f"⚠️ clipping: {total_norm:.2f} → {max_norm}")
    return grads

grads = {"dW1": np.random.randn(10, 10) * 100}
grads = clip_gradients(grads)
```

**مثال ۳ — نورون مرده و راه‌حل:**

```python
def dead_neuron_ratio(activations):
    """نسبت نورون‌هایی که در کل batch صفر بوده‌اند."""
    return (activations.max(axis=0) == 0).mean()

rng = np.random.default_rng(0)
# سناریو ۱: ورودی نرمال
h_normal = np.maximum(0, rng.normal(0, 1, (1000, 64)))
# سناریو ۲: ورودی با بایاس منفی بزرگ (شبکه‌ی بد مقداردهی‌شده)
h_bad = np.maximum(0, rng.normal(-5, 1, (1000, 64)))

print(f"ورودی نرمال: {dead_neuron_ratio(h_normal):.1%} نورون مرده")
print(f"ورودی بد:    {dead_neuron_ratio(h_bad):.1%} نورون مرده")
```

**مثال ۴ — تست با داده‌ی ساختگی (روش طلایی دیباگ):**

```python
def sanity_check(model, n=64):
    """اگر مدل نمی‌تواند ۶۴ نمونه را حفظ کند، مشکلی اساسی دارد."""
    rng = np.random.default_rng(0)
    X_tiny = rng.normal(size=(n, 20))
    y_tiny = rng.integers(0, 2, n)

    # آموزش روی داده‌ی کوچک — باید به دقت ~۱.۰ برسد
    return X_tiny, y_tiny

print("قاعده: مدل باید بتواند ۶۴ نمونه را حفظ کند (دقت ۱۰۰٪).")
print("اگر نتوانست → باگ در کد، نه overfitting.")
```

**این تست، ۸۰٪ باگ‌های آموزش را پیدا می‌کند.** اگر مدل روی ۶۴ نمونه هم overfit نمی‌شود، مشکل معماری یا کد است.

**مثال ۵ — جدول عیب‌یابی تعاملی:**

```python
def troubleshoot(train_loss, val_loss, train_acc, val_acc):
    """تشخیص خودکار از اعداد نهایی."""
    issues = []

    if train_loss > 1.0 and val_loss > 1.0:
        issues.append("underfit: مدل بزرگ‌تر، ویژگی بهتر، lr بیشتر")
    if train_acc - val_acc > 0.15:
        issues.append("overfit: dropout، weight decay، داده بیشتر")
    if abs(train_acc - 0.5) < 0.05:
        issues.append("مدل در حد تصادفی: برچسب‌ها یا گرادیان را چک کن")
    if train_acc > 0.99 and val_acc < 0.7:
        issues.append("حفظ کردن: مدل خیلی بزرگ برای داده")
    if not issues:
        issues.append("سالم")

    return issues

print(troubleshoot(0.9, 0.95, 0.62, 0.60))
print(troubleshoot(0.05, 0.85, 0.99, 0.72))
```

**مثال ۶ — بررسی گرادیان در PyTorch:**

```python
try:
    import torch
    import torch.nn as nn

    model = nn.Sequential(nn.Linear(20, 64), nn.ReLU(), nn.Linear(64, 2))
    X = torch.randn(32, 20)
    y = torch.randint(0, 2, (32,))

    loss = nn.CrossEntropyLoss()(model(X), y)
    loss.backward()

    for name, p in model.named_parameters():
        if p.grad is not None:
            print(f"{name:15s} | grad norm = {p.grad.norm().item():.4e}")

    print("\nاگر گرادیان لایه‌ی اول خیلی کوچک‌تر از لایه‌ی آخر است → محوشدگی")
except ImportError:
    print("pip install torch")
```

### مقایسه و دیدگاه‌های مختلف

| مشکل | تشخیص سریع | راه‌حل اصلی |
|---|---|---|
| محوشدگی | نرم گرادیان لایه‌ها | ReLU + BatchNorm |
| انفجار | loss NaN | clipping |
| نورون مرده | نسبت فعال‌سازی صفر | Leaky ReLU |
| overfit | شکاف train/val | regularization |
| underfit | هر دو بالا | مدل بزرگ‌تر |
| باگ | تست ۶۴ نمونه | gradient check |

**یک دیدگاه:** «بیشتر مشکلات آموزش، مشکل داده است، نه الگوریتم.» در تجربه، درست است: نشت، برچسب غلط، توزیع نامناسب — این‌ها بیشتر از هر هایپرپارامتری دردسر می‌سازند.

### اشتباهات رایج

**۱. تغییر تصادفی هایپرپارامترها.** بدون تشخیص، وقت تلف می‌شود.
**۲. شروع با مدل بزرگ.** اول کوچک، بعد بزرگ.
**۳. نادیده گرفتن داده.** ۸۰٪ مشکلات از داده است.
**۴. نبود بازتولیدپذیری.** نمی‌توانی بفهمی چه چیزی عوض شد.
**۵. اعتماد به یک اجرا.** RL و شبکه‌های عصبی پرنویزند.
**۶. نگاه نکردن به گرادیان‌ها.** اولین ابزار تشخیص.
**۷. توقف زودهنگام.** گاهی فقط patience بیشتر لازم است.

### نکات مهم و ترفندها

- **اول تست ۶۴ نمونه.** اگر مدل آن را حفظ نکرد، باگ داری.
- **نرم گرادیان هر لایه را چاپ کن.** محوشدگی فوراً دیده می‌شود.
- **`clip_grad_norm_` را همیشه بگذار.** ارزان و مؤثر.
- **lr را ۱۰× کم کن وقتی شک داری.** تقریباً همیشه کمک می‌کند.
- **یک تغییر در هر زمان.** وگرنه نمی‌دانی کدام مؤثر بود.
- **میان‌بر:** مشکل؟ اول داده، بعد lr، بعد معماری، آخر regularization.

### خلاصه فصل

- ده مشکل رایج آموزش و نشانه‌هایشان را بشناس.
- محوشدگی و انفجار گرادیان، دو مشکل اصلی عمق‌اند.
- تست ۶۴ نمونه، سریع‌ترین راه تشخیص باگ.
- نرم گرادیان هر لایه، ابزار تشخیص محوشدگی است.
- بیشتر مشکلات، از داده می‌آید نه الگوریتم.

**سؤالات مروری:**
1. محوشدگی گرادیان چه نشانه‌ای دارد و سه راه‌حل بگو.
2. تست ۶۴ نمونه چه چیزی را ثابت می‌کند؟
3. اگر loss NaN شد، سه اقدام به ترتیب چیست؟

---

## منابع و مراجع

**کتاب‌ها**
- Nielsen, M. — *Neural Networks and Deep Learning* (رایگان — بهترین منبع برای backprop)
- Goodfellow, I. et al. — *Deep Learning*، فصل ۶–۸
- Prince, S. — *Understanding Deep Learning* (رایگان: udlbook.github.io)

**مقالات کلیدی**
- He, K. et al. (2015) — *Delving Deep into Rectifiers* (He init)
- Glorot, X. & Bengio, Y. (2010) — *Understanding the difficulty of training deep feedforward networks* (Xavier)
- Ioffe, S. & Szegedy, C. (2015) — *Batch Normalization*
- He, K. et al. (2016) — *Deep Residual Learning* (ResNet)
- Srivastava, N. et al. (2014) — *Dropout*

**ابزارها و دوره‌ها**
- Andrej Karpathy — *Neural Networks: Zero to Hero* (ساخت micrograd و makemore از صفر)
- PyTorch — *Learn the Basics*

---

**بعدی:** [`17-Deep-Learning-Frameworks.md`](./17-Deep-Learning-Frameworks.md) — فصل ۷۷: یادگیری عمیق چیست؟
