# بخش ۰۹: یادگیری با ناظر

**مدت یادگیری تقریبی**: ۳ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۸ (مبانی ML)، فایل ۰۴ (ریاضی)

---

## فهرست محتوا

- [فصل ۳۹: رگرسیون خطی: پیش‌بینی عدد](#فصل-۳۹-رگرسیون-خطی-پیشبینی-عدد)
- [فصل ۴۰: رگرسیون لجستیک: پیش‌بینی دسته](#فصل-۴۰-رگرسیون-لجستیک-پیشبینی-دسته)
- [فصل ۴۱: KNN و درخت تصمیم](#فصل-۴۱-knn-و-درخت-تصمیم)
- [فصل ۴۲: جنگل تصادفی و تقویت گرادیانی](#فصل-۴۲-جنگل-تصادفی-و-تقویت-گرادیانی)
- [فصل ۴۳: ماشین بردار پشتیبان و کرنل](#فصل-۴۳-ماشین-بردار-پشتیبان-و-کرنل)

---

## فصل ۳۹: رگرسیون خطی: پیش‌بینی عدد

### مقدمه

رگرسیون خطی، **«سلام دنیا»ی یادگیری ماشین** است. ساده، سریع، تفسیرپذیر و شگفت‌آور قدرتمند. حتی اگر بعداً از مدل‌های پیچیده استفاده کنی، رگرسیون خطی دو نقش همیشگی دارد: **baseline** و **ابزار فهم داده**.

اگر این فصل را خوب بفهمی، بقیه‌ی الگوریتم‌ها نسخه‌های تعمیم‌یافته‌ی همین ایده‌اند.

### مفاهیم اساسی

**۱. مدل.**

```
ŷ = w₁x₁ + w₂x₂ + ... + wₙxₙ + b
```

- `xᵢ` = ویژگی‌ها
- `wᵢ` = وزن‌ها (ضریب)
- `b` = عرض از مبدأ (bias)
- `ŷ` = پیش‌بینی

**تفسیر `wᵢ`:** «اگر `xᵢ` یک واحد زیاد شود، `ŷ` به اندازه‌ی `wᵢ` تغییر می‌کند، با ثابت بودن بقیه.» این تفسیر، **طلای رگرسیون خطی** است.

**۲. تابع خطا (MSE).**

```
MSE = (1/n) Σ (yᵢ - ŷᵢ)²
```

چرا مربع؟
- مقادیر مثبت و منفی خنثی نشوند.
- خطاهای بزرگ، جریمه‌ی بیشتری بگیرند.
- مشتق‌پذیر باشد (برای گرادیان نزولی).

**۳. دو راه حل:**

- **تحلیلی (Normal Equation):** `θ = (XᵀX)⁻¹Xᵀy` — دقیق، اما برای ویژگی‌های زیاد کند و ناپایدار.
- **گرادیان نزولی:** تکراری، مقیاس‌پذیر، پایه‌ی همه‌ی مدل‌های بعدی.

**۴. چهار فرض رگرسیون خطی (که اغلب نقض می‌شوند):**

| فرض | معنا | نقض چه می‌کند |
|---|---|---|
| **خطی بودن** | رابطه خطی است | پیش‌بینی سوگیرانه |
| **استقلال خطاها** | خطاها به هم وابسته نیستند | سری زمانی |
| **هم‌واریانسی** | واریانس خطا ثابت است | بازه‌های اطمینان غلط |
| **نرمال بودن خطاها** | خطاها نرمال‌اند | آزمون‌های آماری |

**نکته‌ی مهم:** نقض این فرض‌ها، **پیش‌بینی** را همیشه خراب نمی‌کند، اما **تفسیر و استنتاج** را خراب می‌کند. اگر فقط پیش‌بینی می‌خواهی، مهم‌تر است که روی داده‌ی test خوب عمل کند.

**۵. R² — ضریب تعیین.**

```
R² = 1 - SS_res / SS_tot
```
- `R² = 1`: پیش‌بینی کامل
- `R² = 0`: به اندازه‌ی پیش‌بینی با میانگین
- `R² < 0`: بدتر از میانگین (بله، ممکن است!)

**۶. منظم‌سازی (Regularization).** وقتی ویژگی‌های زیاد و همبسته داری:

- **Ridge (L2):** جریمه‌ی `α·Σw²`. وزن‌ها را کوچک می‌کند، هیچ‌کدام را صفر نمی‌کند.
- **Lasso (L1):** جریمه‌ی `α·Σ|w|`. بعضی وزن‌ها را **دقیقاً صفر** می‌کند → انتخاب ویژگی خودکار.
- **ElasticNet:** ترکیب هر دو.

**۷. مهندسی ویژگی برای خطی:** رگرسیون خطی فقط خطی می‌بیند. راه‌حل: ویژگی‌های چندجمله‌ای.

```python
from sklearn.preprocessing import PolynomialFeatures
# x → [x, x², x³]
```

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — رگرسیون خطی از صفر با گرادیان نزولی:**

```python
import numpy as np

def linear_regression_gd(X, y, lr=0.01, epochs=1000):
    """رگرسیون خطی چندویژگی از صفر."""
    n, d = X.shape
    w = np.zeros(d)
    b = 0.0
    history = []

    for _ in range(epochs):
        y_hat = X @ w + b
        error = y_hat - y
        loss = np.mean(error ** 2)
        history.append(loss)

        dw = (2 / n) * (X.T @ error)
        db = (2 / n) * error.sum()
        w -= lr * dw
        b -= lr * db

    return w, b, history

rng = np.random.default_rng(42)
X = rng.normal(size=(500, 3))
true_w = np.array([2.0, -1.5, 0.5])
y = X @ true_w + 3.0 + rng.normal(scale=0.2, size=500)

w, b, hist = linear_regression_gd(X, y)
print("وزن‌های یادگرفته‌شده:", w.round(3))
print("وزن‌های واقعی:        ", true_w)
print("بایاس:", round(b, 3), "(واقعی 3.0)")
print("loss اول:", round(hist[0], 2), "| loss آخر:", round(hist[-1], 4))
```

**مثال ۲ — با sklearn، همراه با مقایسه‌ی Ridge و Lasso:**

```python
import numpy as np
import pandas as pd
from sklearn.datasets import make_regression
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.linear_model import LinearRegression, Ridge, Lasso
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import make_pipeline
from sklearn.metrics import mean_absolute_error, r2_score

X, y = make_regression(n_samples=500, n_features=20, n_informative=8,
                       noise=10, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

models = {
    "Linear": LinearRegression(),
    "Ridge":  Ridge(alpha=1.0),
    "Lasso":  Lasso(alpha=1.0, max_iter=10_000),
}

for name, m in models.items():
    pipe = make_pipeline(StandardScaler(), m)
    pipe.fit(X_train, y_train)
    pred = pipe.predict(X_test)
    print(f"{name:7s} | MAE={mean_absolute_error(y_test, pred):6.2f} | R²={r2_score(y_test, pred):.3f}")

# تعداد ویژگی‌های صفرشده توسط Lasso
lasso = make_pipeline(StandardScaler(), Lasso(alpha=5.0, max_iter=10_000)).fit(X_train, y_train)
coefs = lasso.named_steps["lasso"].coef_
print(f"\nLasso: {np.sum(coefs == 0)} از {len(coefs)} ویژگی صفر شد")
```

**مثال ۳ — رابطه‌ی غیرخطی و چندجمله‌ای:**

```python
from sklearn.preprocessing import PolynomialFeatures
from sklearn.pipeline import make_pipeline

# رابطه‌ی سهمی
x = np.linspace(-3, 3, 200).reshape(-1, 1)
y = 2 * x.ravel() ** 2 + x.ravel() + np.random.default_rng(0).normal(0, 1, 200)

linear = LinearRegression().fit(x, y)
poly = make_pipeline(PolynomialFeatures(degree=2), LinearRegression()).fit(x, y)

print(f"خطی R²: {linear.score(x, y):.3f}")
print(f"درجه ۲ R²: {poly.score(x, y):.3f}")
```

**درس:** اگر رابطه غیرخطی است، یا ویژگی چندجمله‌ای بساز، یا سراغ مدل غیرخطی (درخت، شبکه) برو.

**مثال ۴ — تشخیص بیش‌برازش با درجه‌ی چندجمله‌ای:**

```python
from sklearn.model_selection import cross_val_score

for degree in [1, 2, 5, 15]:
    model = make_pipeline(PolynomialFeatures(degree), LinearRegression())
    train_r2 = model.fit(x, y).score(x, y)
    cv_r2 = cross_val_score(model, x, y, cv=5, scoring="r2").mean()
    print(f"درجه {degree:2d} | train R²={train_r2:.3f} | CV R²={cv_r2:.3f}")
```

خروجی معمولاً نشان می‌دهد: درجه‌ی بالا، train را عالی و CV را فاجعه می‌کند. **این، تعریف عملی overfitting است.**

### مقایسه و دیدگاه‌های مختلف

| مدل | مزیت | عیب | کِی |
|---|---|---|---|
| Linear | تفسیرپذیر، سریع | فقط خطی | baseline، داده‌ی خطی |
| Ridge | پایدار با ویژگی همبسته | هیچ ضریبی صفر نمی‌شود | ویژگی زیاد، همبسته |
| Lasso | انتخاب ویژگی خودکار | با ویژگی همبسته ناپایدار | ویژگی زیاد، پرت |
| ElasticNet | تعادل L1/L2 | دو فراپارامتر | داده‌ی پیچیده |
| Polynomial + Linear | رابطه‌ی غیرخطی ساده | بیش‌برازش با درجه‌ی بالا | رابطه‌ی هموار |

**یک دیدگاه مهم:** «همیشه با رگرسیون خطی شروع کن.» نه چون بهترین است، بلکه چون **سریع‌ترین baseline** و **بهترین ابزار فهم** است. اگر خطی جواب داد، تمام؛ اگر نه، می‌دانی مشکل «خطی نبودن» است.

### اشتباهات رایج

**۱. مقایسه‌ی ضرایب بدون استانداردسازی.** ویژگی با مقیاس بزرگ، ضریب کوچک می‌گیرد. برای مقایسه، اول `StandardScaler`.
**۲. تفسیر ضریب به‌عنوان علیت.** ضریب می‌گوید «همبستگی»، نه «اثر علّی».
**۳. نادیده گرفتن هم‌خطی (multicollinearity).** دو ویژگی همبسته‌ی ۰.۹۹، ضرایب ناپایدار می‌سازند.
**۴. R² بالا روی train را موفقیت دانستن.** باید روی test ببینی.
**۵. استفاده از رگرسیون خطی برای هدف دودویی.** از لجستیک استفاده کن.
**۶. Lasso بدون استانداردسازی.** جریمه به مقیاس ویژگی وابسته است و نتیجه غلط می‌دهد.

### نکات مهم و ترفندها

- **همیشه `StandardScaler` قبل از Ridge/Lasso.** بدون آن، جریمه معنای یکسانی ندارد.
- **`alpha` را با CV تنظیم کن.** `RidgeCV` و `LassoCV` خودکار انجام می‌دهند.
- **باقیمانده‌ها را رسم کن.** اگر الگو دارند، رابطه خطی نیست.
- **برای تفسیر، ضرایب استانداردشده را ببین.** `coef_` بعد از `StandardScaler`.
- **میان‌بر:** `ŷ = w·x + b` · MSE · Ridge = کوچک کردن وزن · Lasso = صفر کردن وزن.

### خلاصه فصل

- رگرسیون خطی: ترکیب خطی ویژگی‌ها با وزن.
- MSE تابع خطاست؛ دو راه حل: تحلیلی و گرادیان نزولی.
- R² نسبت واریانس توضیح‌داده‌شده است.
- Ridge وزن‌ها را کوچک می‌کند، Lasso بعضی را صفر.
- رابطه‌ی غیرخطی را با ویژگی چندجمله‌ای یا مدل غیرخطی حل کن.

**سؤالات مروری:**
1. تفسیر ضریب `w` در رگرسیون خطی چیست؟
2. چرا Lasso انتخاب ویژگی انجام می‌دهد؟
3. اگر R² روی train = 0.99 و روی test = 0.4 باشد، چه اتفاقی افتاده؟

---

## فصل ۴۰: رگرسیون لجستیک: پیش‌بینی دسته

### مقدمه

نامش گمراه‌کننده است: **رگرسیون لجستیک، یک مدل دسته‌بندی است**، نه رگرسیون. دلیل نامش این است که از یک تابع خطی شروع می‌کند و بعد آن را به احتمال تبدیل می‌کند.

این مدل، **پرکاربردترین مدل دسته‌بندی در صنعت** است: سریع، تفسیرپذیر، پایدار. در بانک، بیمه و پزشکی، هنوز انتخاب اول است — چون می‌توانی توضیح بدهی چرا تصمیم گرفتی.

### مفاهیم اساسی

**۱. از خط به احتمال.**

```
z = w·x + b            ← امتیاز خام (بی‌کران)
p = σ(z) = 1/(1+e⁻ᶻ)   ← احتمال (بین ۰ و ۱)
ŷ = 1 if p > 0.5 else 0
```

تابع `σ` (سیگموید) هر عددی را به بازه‌ی (۰,۱) می‌برد.

**۲. تابع خطا: cross-entropy (log loss).**

```
L = -[y·log(p) + (1-y)·log(1-p)]
```

چرا نه MSE؟ چون با سیگموید، MSE یک تابع غیرمحدب می‌دهد و گرادیان نزولی گیر می‌کند. cross-entropy محدب است و آموزش پایدار.

**۳. مرز تصمیم (Decision Boundary).** مرز خطی است: `w·x + b = 0`. در دو بعد، یک خط؛ در سه بعد، یک صفحه.

**۴. آستانه (Threshold) — تصمیم مهندسی، نه پیش‌فرض.**

پیش‌فرض ۰.۵ است، اما:
- در تشخیص بیماری: آستانه را **پایین** بیاور (۰.۳) تا هیچ بیمار را از دست ندهی.
- در فیلتر اسپم: آستانه را **بالا** ببر (۰.۸) تا ایمیل مهم را حذف نکنی.

**آستانه، معامله‌ی precision/recall است** (فایل ۱۴).

**۵. کلاس‌های چندگانه.**

- **One-vs-Rest (OvR):** یک مدل برای هر کلاس.
- **Softmax (Multinomial):** یک مدل با خروجی softmax.

sklearn به‌طور پیش‌فرض برای چندکلاسه از multinomial استفاده می‌کند.

**۶. `class_weight` — ابزار داده‌ی نامتوازن.**

```python
LogisticRegression(class_weight="balanced")
```
وزن کلاس اقلیت را به نسبت معکوس فراوانی بالا می‌برد.

**۷. تعبیر ضرایب.** `exp(wᵢ)` = **نسبت شانس (odds ratio)**: «با یک واحد افزایش `xᵢ`، شانس کلاس مثبت چند برابر می‌شود.» این تفسیر، در پزشکی و بانک بسیار ارزشمند است.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — پیاده‌سازی از صفر:**

```python
import numpy as np

def sigmoid(z):
    return 1 / (1 + np.exp(-np.clip(z, -500, 500)))   # clip برای پایداری

def logistic_regression_gd(X, y, lr=0.1, epochs=2000):
    n, d = X.shape
    w = np.zeros(d)
    b = 0.0
    losses = []

    for _ in range(epochs):
        p = sigmoid(X @ w + b)
        eps = 1e-12
        loss = -np.mean(y * np.log(p + eps) + (1 - y) * np.log(1 - p + eps))
        losses.append(loss)

        grad = p - y
        w -= lr * (X.T @ grad) / n
        b -= lr * grad.mean()

    return w, b, losses

rng = np.random.default_rng(0)
X = rng.normal(size=(400, 2))
true_w = np.array([1.5, -2.0])
z = X @ true_w + 0.5
y = (sigmoid(z) > rng.random(400)).astype(int)   # برچسب با نویز

w, b, losses = logistic_regression_gd(X, y, lr=0.5)
print("وزن‌های یادگرفته‌شده:", w.round(2), "| بایاس:", round(b, 2))
print("loss اول:", round(losses[0], 3), "| loss آخر:", round(losses[-1], 3))
```

**مثال ۲ — با sklearn و تنظیم آستانه:**

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import precision_score, recall_score, f1_score, roc_auc_score

X, y = make_classification(n_samples=2000, n_features=10, n_informative=6,
                           weights=[0.9], flip_y=0.02, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.25, stratify=y, random_state=42
)

model = make_pipeline(
    StandardScaler(),
    LogisticRegression(max_iter=1000, class_weight="balanced", random_state=42),
).fit(X_train, y_train)

probs = model.predict_proba(X_test)[:, 1]
print(f"ROC-AUC: {roc_auc_score(y_test, probs):.3f}\n")

print(f"{'آستانه':>8} {'precision':>10} {'recall':>8} {'f1':>6}")
for th in [0.2, 0.3, 0.5, 0.7, 0.8]:
    pred = (probs >= th).astype(int)
    print(f"{th:>8.1f} {precision_score(y_test, pred, zero_division=0):>10.3f} "
          f"{recall_score(y_test, pred):>8.3f} {f1_score(y_test, pred):>6.3f}")
```

**این جدول، مهم‌ترین درس این فصل است:** آستانه را بر اساس **هزینه‌ی کسب‌وکار** انتخاب کن، نه پیش‌فرض.

**مثال ۳ — تفسیر ضرایب به‌صورت نسبت شانس:**

```python
import pandas as pd
import numpy as np

coefs = model.named_steps["logisticregression"].coef_[0]
names = [f"feature_{i}" for i in range(len(coefs))]

table = pd.DataFrame({
    "feature": names,
    "coef": coefs.round(3),
    "odds_ratio": np.exp(coefs).round(3),
}).sort_values("coef", key=abs, ascending=False)

print(table.head(6).to_string(index=False))
print("\nتفسیر: odds_ratio = 2 یعنی با یک واحد افزایش، شانس مثبت‌شدن دو برابر می‌شود.")
```

**مثال ۴ — مقایسه‌ی وزن‌دهی:**

```python
from sklearn.metrics import classification_report

for cw in [None, "balanced"]:
    m = make_pipeline(
        StandardScaler(),
        LogisticRegression(max_iter=1000, class_weight=cw, random_state=42)
    ).fit(X_train, y_train)
    print(f"\n=== class_weight={cw} ===")
    print(classification_report(y_test, m.predict(X_test), digits=3))
```

### مقایسه و دیدگاه‌های مختلف

| مدل | تفسیرپذیری | سرعت | دقت | کِی |
|---|---|---|---|---|
| Logistic | بالا | خیلی سریع | خوب | baseline، نیاز به توضیح |
| Random Forest | متوسط | سریع | بهتر | داده‌ی جدولی |
| Gradient Boosting | متوسط | متوسط | عالی | رقابت، دقت بالا |
| Neural Net | پایین | کند | عالی با داده‌ی زیاد | تصویر، متن |

**یک دیدگاه مهم:** «در محیط‌های تنظیم‌شده (بانک، بیمه، پزشکی) مدل باید قابل توضیح باشد.» همین، دلیل محبوبیت ماندگار رگرسیون لجستیک است. اگر مدل تو باید به مشتری توضیح داده شود («چرا وام رد شد؟»)، لجستیک انتخاب طبیعی است.

### اشتباهات رایج

**۱. «لجستیک، رگرسیون است.»** دسته‌بندی است.
**۲. استفاده از MSE برای آموزش.** cross-entropy درست است.
**۳. آستانه‌ی ۰.۵ بدون فکر.** باید بر اساس هزینه‌ی خطا تنظیم شود.
**۴. تفسیر ضریب بدون استانداردسازی.** مقایسه‌ی ضرایب ویژگی‌های با مقیاس متفاوت، بی‌معنی است.
**۵. نادیده گرفتن داده‌ی نامتوازن.** بدون `class_weight`، مدل همه را منفی پیش‌بینی می‌کند.
**۶. فرض خطی بودن مرز.** اگر مرز پیچیده است، ویژگی تعاملی بساز یا مدل غیرخطی بزن.

### نکات مهم و ترفندها

- **`class_weight="balanced"` را برای داده‌ی نامتوازن امتحان کن.** رایگان است و معمولاً کمک می‌کند.
- **آستانه را روی validation تنظیم کن، نه test.**
- **`exp(coef)` را گزارش کن، نه `coef`.** برای مخاطب غیرفنی، نسبت شانس قابل فهم‌تر است.
- **`C` (معکوس قدرت منظم‌سازی) را با CV تنظیم کن.** `C` کوچک = منظم‌سازی قوی.
- **میان‌بر:** z = w·x + b → sigmoid → p → آستانه → کلاس.

### خلاصه فصل

- رگرسیون لجستیک، دسته‌بندی با تبدیل خطی به احتمال است.
- cross-entropy تابع خطاست؛ محدب و پایدار.
- آستانه، تصمیم مهندسی است و باید با هزینه‌ی خطا تنظیم شود.
- `class_weight` برای داده‌ی نامتوازن.
- `exp(coef)` نسبت شانس می‌دهد — ابزار تفسیر قدرتمند.

**سؤالات مروری:**
1. چرا رگرسیون لجستیک از MSE استفاده نمی‌کند؟
2. کِی آستانه را پایین می‌آوری و کِی بالا؟
3. تفسیر `odds_ratio = 1.8` چیست؟

---

## فصل ۴۱: KNN و درخت تصمیم

### مقدمه

این فصل، دو مدل کاملاً متفاوت را معرفی می‌کند:

- **KNN:** «بگو همسایه‌هایت چه کسانی هستند، تا بگویم تو چه هستی.» ساده‌ترین ایده‌ی ممکن.
- **درخت تصمیم:** «سری سؤال بله/خیر بپرس تا به جواب برسی.» شبیه‌ترین مدل به طرز فکر انسانی.

هر دو، شهود مهمی می‌دهند که در مدل‌های پیچیده‌تر هم به کار می‌آید.

### مفاهیم اساسی

**KNN — سه ایده‌ی ساده:**

1. برای پیش‌بینی یک نمونه، نزدیک‌ترین `k` نمونه‌ی آموزش را پیدا کن.
2. **دسته‌بندی:** رأی اکثریت. **رگرسیون:** میانگین.
3. تمام. هیچ «آموزشی» در کار نیست.

**چهار نکته‌ی حیاتی KNN:**

- **مقیاس‌بندی واجب است.** فاصله‌ی اقلیدسی بدون مقیاس، تحت سلطه‌ی ویژگی با بازه‌ی بزرگ است (مثلاً درآمد در برابر سن).
- **انتخاب `k`:** k کوچک = بیش‌برازش (نویز)، k بزرگ = کم‌برازش (همه‌چیز یکسان). مقدار فرد، معمولاً ۳ تا ۱۵.
- **کندی پیش‌بینی:** KNN در آموزش سریع است (هیچ کاری نمی‌کند) اما در پیش‌بینی کند (باید همه‌ی فاصله‌ها را حساب کند).
- **لعنت ابعاد (Curse of Dimensionality):** در ابعاد بالا، همه‌ی نقاط تقریباً هم‌فاصله‌اند و KNN بی‌معنی می‌شود.

**درخت تصمیم — مفاهیم:**

**۱. ساختار:** هر گره داخلی یک سؤال (`age > 30?`)، هر برگ یک تصمیم.

**۲. معیار انتخاب تقسیم:**
- **Gini Impurity:** `1 - Σpᵢ²` — احتمال اشتباه طبقه‌بندی تصادفی.
- **Entropy:** `-Σpᵢ log(pᵢ)` — عدم قطعیت.
- **Information Gain:** کاهش آنتروپی پس از تقسیم.

**۳. هایپرپارامترهای کلیدی:**

| پارامتر | اثر | نکته |
|---|---|---|
| `max_depth` | عمق درخت | کوچک = ساده‌تر، مقاوم‌تر |
| `min_samples_split` | حداقل نمونه برای تقسیم | بزرگ = ساده‌تر |
| `min_samples_leaf` | حداقل نمونه در برگ | بزرگ = هموارتر |
| `ccp_alpha` | هرس هزینه-پیچیدگی | هرس خودکار |

**۴. مزیت بزرگ: تفسیرپذیری.** می‌توانی درخت را رسم کنی و به مدیر نشان بدهی.

**۵. عیب بزرگ: ناپایداری.** تغییر کوچک در داده، درخت کاملاً متفاوتی می‌سازد. همین، دلیل وجود جنگل تصادفی است.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — KNN: اثر مقیاس‌بندی (مهم‌ترین آزمایش):**

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.neighbors import KNeighborsClassifier
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler

X, y = make_classification(n_samples=600, n_features=5, n_informative=3, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.25, random_state=42)

# بدون مقیاس‌بندی
knn_raw = KNeighborsClassifier(n_neighbors=5).fit(X_train, y_train)
print(f"بدون مقیاس‌بندی: {knn_raw.score(X_test, y_test):.3f}")

# با مقیاس‌بندی
knn_scaled = make_pipeline(StandardScaler(), KNeighborsClassifier(n_neighbors=5))
knn_scaled.fit(X_train, y_train)
print(f"با مقیاس‌بندی:   {knn_scaled.score(X_test, y_test):.3f}")

# با یک ویژگی در مقیاس بزرگ
X_big = X.copy()
X_big[:, 0] *= 1000
Xtr, Xte, ytr, yte = train_test_split(X_big, y, test_size=0.25, random_state=42)
print(f"با ویژگی بزرگ بدون مقیاس: {KNeighborsClassifier().fit(Xtr, ytr).score(Xte, yte):.3f}")
```

**مثال ۲ — انتخاب `k`:**

```python
import matplotlib.pyplot as plt

ks = range(1, 31)
scores = []
for k in ks:
    pipe = make_pipeline(StandardScaler(), KNeighborsClassifier(n_neighbors=k))
    scores.append(cross_val_score(pipe, X, y, cv=5).mean())

best_k = list(ks)[int(np.argmax(scores))]
print(f"بهترین k: {best_k} | دقت: {max(scores):.3f}")

plt.figure(figsize=(8, 4))
plt.plot(list(ks), scores, marker="o", ms=3)
plt.axvline(best_k, color="red", ls="--")
plt.xlabel("k")
plt.ylabel("CV accuracy")
plt.title("انتخاب k در KNN")
plt.show()
```

**مثال ۳ — درخت تصمیم و تفسیر آن:**

```python
from sklearn.tree import DecisionTreeClassifier, export_text
from sklearn.datasets import load_iris

iris = load_iris()
X, y = iris.data, iris.target
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.25, stratify=y, random_state=42
)

tree = DecisionTreeClassifier(max_depth=3, random_state=42).fit(X_train, y_train)
print(f"دقت: {tree.score(X_test, y_test):.3f}\n")
print(export_text(tree, feature_names=list(iris.feature_names), decimals=2))
```

**خروجی `export_text`، یک درخت قابل خواندن است.** این، تفاوت درخت با مدل‌های جعبه‌سیاه است.

**مثال ۴ — اثر عمق درخت (بیش‌برازش):**

```python
for depth in [1, 3, 5, 10, None]:
    t = DecisionTreeClassifier(max_depth=depth, random_state=42)
    train = t.fit(X_train, y_train).score(X_train, y_train)
    test = t.score(X_test, y_test)
    print(f"max_depth={str(depth):>4} | train={train:.3f} | test={test:.3f}")
```

خروجی معمول: `max_depth=None` → train = 1.000، test کمتر. **این، حفظ کردن داده است.**

**مثال ۵ — اهمیت ویژگی:**

```python
import pandas as pd

importance = pd.Series(
    tree.feature_importances_, index=iris.feature_names
).sort_values(ascending=False)
print(importance.round(3))
```

### مقایسه و دیدگاه‌های مختلف

| معیار | KNN | درخت تصمیم |
|---|---|---|
| نیاز به مقیاس‌بندی | **بله، واجب** | خیر |
| سرعت آموزش | صفر | سریع |
| سرعت پیش‌بینی | کند | سریع |
| تفسیرپذیری | متوسط | بالا |
| داده‌ی با ابعاد بالا | ضعیف | متوسط |
| نویز | حساس | متوسط |
| مرز تصمیم | پیچیده، محلی | پله‌ای، محوری |

**دو دیدگاه:**
- **«KNN منسوخ شده است.»** برای داده‌ی جدولی بزرگ، تا حدی درست. اما برای سیستم‌های توصیه‌گر (شباهت کاربران) و جست‌وجوی برداری، هنوز هسته‌ی کار است.
- **«درخت تنها کافی نیست.»** درست: درخت تنها ناپایدار است. اما **به‌عنوان پایه‌ی جنگل و boosting**، مهم‌ترین الگوریتم دنیاست.

### اشتباهات رایج

**۱. KNN بدون مقیاس‌بندی.** رایج‌ترین خطا.
**۲. `k` زوج در دسته‌بندی دودویی.** رأی مساوی ممکن می‌شود. k فرد بگذار.
**۳. KNN روی داده‌ی با ابعاد بالا.** لعنت ابعاد. اول PCA.
**۴. درخت بدون محدودیت عمق.** بیش‌برازش کامل.
**۵. اعتماد به یک درخت تنها.** ناپایدار است؛ از جنگل استفاده کن.
**۶. تفسیر `feature_importances_` به‌عنوان علیت.** اهمیت، همبستگی است.
**۷. نادیده گرفتن این‌که `feature_importances_` به همبستگی بین ویژگی‌ها حساس است.** دو ویژگی تکراری، اهمیت را نصف می‌کنند.

### نکات مهم و ترفندها

- **KNN: همیشه `Pipeline([("scaler", StandardScaler()), ("knn", KNeighborsClassifier())])`.**
- **KNN با `weights="distance"`:** همسایه‌های نزدیک‌تر رأی بیشتری دارند. معمولاً بهتر است.
- **درخت: `max_depth` و `min_samples_leaf` را تنظیم کن.** بیشترین اثر را دارند.
- **`export_text` برای نمایش درخت.** ابزار عالی برای توضیح به غیرفنی‌ها.
- **میان‌بر:** KNN = همسایه‌ها + فاصله · درخت = سؤال‌های بله/خیر + هرس.

### خلاصه فصل

- KNN بر اساس نزدیک‌ترین همسایه‌ها تصمیم می‌گیرد؛ مقیاس‌بندی واجب است.
- `k` کوچک = بیش‌برازش، `k` بزرگ = کم‌برازش.
- درخت تصمیم با تقسیم‌های بازگشتی ساخته می‌شود و بسیار تفسیرپذیر است.
- `max_depth` و `min_samples_leaf` کلیدی‌ترین هایپرپارامترها هستند.
- درخت تنها ناپایدار است؛ جنگل حلش می‌کند.

**سؤالات مروری:**
1. چرا KNN بدون مقیاس‌بندی تقریباً بی‌فایده است؟
2. Gini impurity چه چیزی را می‌سنجد؟
3. چرا درخت با `max_depth=None` بیش‌برازش می‌کند؟

---

## فصل ۴۲: جنگل تصادفی و تقویت گرادیانی

### مقدمه

این فصل، **دو الگوریتم برنده‌ی داده‌ی جدولی** را معرفی می‌کند. اگر یک جمله بخواهی: برای داده‌ی جدولی با اندازه‌ی متوسط، **Gradient Boosting (XGBoost/LightGBM) معمولاً بهترین انتخاب است** — و Random Forest، انتخاب مطمئن و کم‌دردسر.

هر دو از درخت می‌سازند، اما به دو روش کاملاً متفاوت.

### مفاهیم اساسی

**۱. Bagging و Random Forest.**

- **Bagging:** چند مدل روی نمونه‌های Bootstrap (نمونه‌گیری با جایگذاری) آموزش بده، سپس رأی‌گیری/میانگین.
- **Random Forest:** Bagging + **انتخاب تصادفی زیرمجموعه‌ی ویژگی‌ها** در هر تقسیم.

**چرا کار می‌کند؟** کاهش واریانس. خطاهای تصادفی درخت‌ها همدیگر را خنثی می‌کنند.

```
خطای جنگل ≈ خطای یک درخت / √(تعداد درخت‌ها)   (اگر مستقل باشند)
```
به همین دلیل ۱۰۰ درخت خیلی بهتر از ۱ درخت است — اما ۱۰۰۰ درخت فقط کمی بهتر از ۱۰۰.

**۲. Boosting — ایده‌ی متفاوت.**

به جای «چند مدل مستقل»، **مدل‌های متوالی که هر کدام خطای قبلی را اصلاح می‌کند**.

```
F₀ = میانگین
F₁ = F₀ + lr × درخت₁(خطای F₀)
F₂ = F₁ + lr × درخت₂(خطای F₁)
...
```

**تفاوت بنیادی:**

| | Bagging (RF) | Boosting (GBM) |
|---|---|---|
| آموزش | موازی | ترتیبی |
| هدف | کاهش واریانس | کاهش بایاس |
| بیش‌برازش | مقاوم | حساس (نیاز به تنظیم) |
| درخت‌ها | عمیق | کم‌عمق (weak learners) |
| تنظیم | ساده | دقیق‌تر لازم |

**۳. هایپرپارامترهای Random Forest:**

| پارامتر | اثر |
|---|---|
| `n_estimators` | تعداد درخت (بیشتر = پایدارتر، تا حدی) |
| `max_depth` | عمق هر درخت |
| `max_features` | ویژگی‌های تصادفی در هر تقسیم (`sqrt` پیش‌فرض) |
| `min_samples_leaf` | حداقل نمونه در برگ |
| `class_weight` | وزن کلاس‌ها |

**۴. هایپرپارامترهای Gradient Boosting:**

| پارامتر | اثر |
|---|---|
| `n_estimators` | تعداد مرحله |
| `learning_rate` | سهم هر درخت (کوچک = نیاز به تعداد بیشتر) |
| `max_depth` | عمق (معمولاً ۳–۸) |
| `subsample` | نسبت نمونه‌گیری (stochastic GB) |
| `reg_lambda` | منظم‌سازی L2 |

**قاعده‌ی مهم:** `learning_rate` کوچک + `n_estimators` بزرگ = معمولاً بهترین نتیجه، اما کندتر.

**۵. سه پیاده‌سازی محبوب:**
- **sklearn `GradientBoostingClassifier`** — آموزشی، کند.
- **XGBoost** — سریع، منظم‌سازی قوی، استاندارد صنعتی.
- **LightGBM** — سریع‌ترین، حافظه‌ی کم، عالی برای داده‌ی بزرگ.
- **CatBoost** — بهترین برای ویژگی دسته‌ای.

**۶. اهمیت ویژگی (Feature Importance):**
- **Impurity-based:** سریع، اما سوگیرانه (به ویژگی‌های با کاردینالیتی بالا).
- **Permutation:** قابل اعتمادتر، کندتر.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — Random Forest در برابر درخت تنها:**

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import f1_score

X, y = make_classification(n_samples=2000, n_features=20, n_informative=10,
                           n_redundant=5, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.25, stratify=y, random_state=42
)

models = {
    "Tree (depth=5)": DecisionTreeClassifier(max_depth=5, random_state=42),
    "Tree (full)": DecisionTreeClassifier(random_state=42),
    "RandomForest 100": RandomForestClassifier(n_estimators=100, random_state=42),
    "RandomForest 500": RandomForestClassifier(n_estimators=500, random_state=42),
}

for name, m in models.items():
    m.fit(X_train, y_train)
    train_f1 = f1_score(y_train, m.predict(X_train))
    test_f1 = f1_score(y_test, m.predict(X_test))
    print(f"{name:18s} | train F1={train_f1:.3f} | test F1={test_f1:.3f}")
```

خروجی معمول: درخت کامل، train=1.0 و test پایین‌تر؛ جنگل، شکاف کوچک‌تر.

**مثال ۲ — Gradient Boosting با تنظیم:**

```python
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.model_selection import GridSearchCV

param_grid = {
    "n_estimators": [100, 300],
    "learning_rate": [0.05, 0.1],
    "max_depth": [3, 5],
}

gs = GridSearchCV(
    GradientBoostingClassifier(random_state=42),
    param_grid, cv=3, scoring="f1", n_jobs=-1, verbose=1
)
gs.fit(X_train, y_train)

print("بهترین پارامترها:", gs.best_params_)
print(f"بهترین CV F1: {gs.best_score_:.3f}")
print(f"test F1: {f1_score(y_test, gs.predict(X_test)):.3f}")
```

**مثال ۳ — XGBoost / LightGBM (اگر نصب باشد):**

```python
try:
    from xgboost import XGBClassifier
    xgb = XGBClassifier(
        n_estimators=500, learning_rate=0.05, max_depth=5,
        subsample=0.8, colsample_bytree=0.8,
        reg_lambda=1.0, eval_metric="logloss", random_state=42,
    )
    xgb.fit(X_train, y_train)
    print(f"XGBoost test F1: {f1_score(y_test, xgb.predict(X_test)):.3f}")
except ImportError:
    print("نصب کن: pip install xgboost")

try:
    from lightgbm import LGBMClassifier
    lgbm = LGBMClassifier(n_estimators=500, learning_rate=0.05, num_leaves=31,
                          random_state=42, verbose=-1)
    lgbm.fit(X_train, y_train)
    print(f"LightGBM test F1: {f1_score(y_test, lgbm.predict(X_test)):.3f}")
except ImportError:
    print("نصب کن: pip install lightgbm")
```

**مثال ۴ — اهمیت ویژگی با permutation (قابل اعتمادتر):**

```python
from sklearn.inspection import permutation_importance
import pandas as pd

rf = RandomForestClassifier(n_estimators=300, random_state=42).fit(X_train, y_train)

perm = permutation_importance(rf, X_test, y_test, n_repeats=10,
                              random_state=42, scoring="f1")

imp = pd.DataFrame({
    "feature": [f"f{i}" for i in range(X.shape[1])],
    "importance": perm.importances_mean,
    "std": perm.importances_std,
}).sort_values("importance", ascending=False)

print(imp.head(8).round(4).to_string(index=False))
```

**مثال ۵ — مقایسه‌ی نهایی چند مدل:**

```python
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier

comparison = {}
for name, clf in [
    ("Logistic", LogisticRegression(max_iter=1000, random_state=42)),
    ("RF", RandomForestClassifier(n_estimators=300, random_state=42)),
    ("GBM", GradientBoostingClassifier(n_estimators=200, learning_rate=0.05,
                                       max_depth=3, random_state=42)),
]:
    scores = cross_val_score(clf, X_train, y_train, cv=5, scoring="f1")
    comparison[name] = (scores.mean(), scores.std())
    print(f"{name:9s} | F1 = {scores.mean():.3f} ± {scores.std():.3f}")
```

### مقایسه و دیدگاه‌های مختلف

| معیار | Random Forest | Gradient Boosting | XGBoost / LightGBM |
|---|---|---|---|
| دقت | خوب | عالی | عالی |
| سرعت آموزش | سریع (موازی) | کند | خیلی سریع |
| تنظیم | آسان | متوسط | نیاز به تجربه |
| بیش‌برازش | مقاوم | حساس | مقاوم (با منظم‌سازی) |
| تفسیر | متوسط | متوسط | متوسط |
| داده‌ی نامتوازن | `class_weight` | `sample_weight` | `scale_pos_weight` |

**یک دیدگاه رایج در مسابقات:** «اگر داده جدولی است، اول XGBoost/LightGBM را امتحان کن.» این تقریباً همیشه درست است. اما در تولید، Random Forest اغلب به‌خاطر **سادگی تنظیم و پایداری** ترجیح داده می‌شود.

### اشتباهات رایج

**۱. تنظیم `n_estimators` بدون `learning_rate`.** این دو به هم وابسته‌اند.
**۲. `max_depth` بزرگ در Boosting.** بیش‌برازش سریع. ۳–۸ کافی است.
**۳. اعتماد به `feature_importances_` خام.** سوگیرانه است؛ از permutation استفاده کن.
**۴. نادیده گرفتن `n_jobs=-1`.** استفاده نکردن از همه‌ی هسته‌ها، اتلاف وقت.
**۵. Boosting بدون validation.** زودتر از RF بیش‌برازش می‌کند.
**۶. فراموش کردن `random_state`.** نتایج غیرقابل بازتولید.

### نکات مهم و ترفندها

- **RF: `n_estimators=300` نقطه‌ی شیرین است.** بیشتر، سود کمی دارد.
- **Boosting: `learning_rate` را اول کوچک بگذار (۰.۰۵)، بعد `n_estimators` را زیاد کن.**
- **`early_stopping` در XGBoost/LightGBM** جلوی بیش‌برازش را می‌گیرد و وقت صرفه‌جویی می‌کند.
- **`permutation_importance` را روی test حساب کن، نه train.**
- **میان‌بر:** RF = چند درخت مستقل + رأی · Boosting = درخت‌های زنجیره‌ای + اصلاح خطا.

### خلاصه فصل

- Bagging واریانس را کم می‌کند؛ Boosting بایاس را.
- Random Forest = Bagging + انتخاب تصادفی ویژگی.
- Gradient Boosting = مدل‌های متوالی روی باقیمانده‌ها.
- `learning_rate` و `n_estimators` به هم وابسته‌اند.
- XGBoost/LightGBM استاندارد داده‌ی جدولی‌اند.

**سؤالات مروری:**
1. تفاوت اصلی Bagging و Boosting چیست؟
2. چرا Random Forest از یک درخت بهتر است؟
3. چرا `learning_rate` کوچک نیاز به `n_estimators` بزرگ دارد؟

---

## فصل ۴۳: ماشین بردار پشتیبان و کرنل

### مقدمه

SVM یک ایده‌ی هندسی زیبا دارد: **«بهترین مرز، مرزی است که بیشترین فاصله را از نزدیک‌ترین نقاط دو کلاس داشته باشد.»** این «حاشیه‌ی حداکثری» (maximum margin)، به مدل مقاومت در برابر نویز می‌دهد.

SVM در دهه‌ی ۲۰۰۰ پرکاربردترین مدل بود. امروز کمتر استفاده می‌شود، اما **مفهوم کرنل** آن، پایه‌ی درک بسیاری از روش‌های مدرن است.

### مفاهیم اساسی

**۱. حاشیه (Margin).** فاصله‌ی مرز تصمیم تا نزدیک‌ترین نمونه‌ها. SVM این فاصله را بیشینه می‌کند.

**۲. بردارهای پشتیبان (Support Vectors).** نمونه‌هایی که روی یا نزدیک حاشیه‌اند. **فقط این‌ها** مدل را تعیین می‌کنند؛ بقیه‌ی نقاط اثری ندارند. این، ویژگی متمایز SVM است.

**۳. حاشیه‌ی نرم (Soft Margin) و پارامتر C.**

داده‌ی واقعی همیشه جدا شدنی نیست. `C` تعادل بین «حاشیه‌ی بزرگ» و «خطای کم» است:

| `C` | اثر |
|---|---|
| کوچک | حاشیه‌ی بزرگ، تحمل خطای بیشتر → منظم‌سازی قوی، مقاوم به بیش‌برازش |
| بزرگ | حاشیه‌ی کوچک، تلاش برای جداسازی کامل → بیش‌برازش |

**۴. کرنل (Kernel) — ترفند کلیدی.**

اگر داده با خط جدا نشود، SVM آن را به فضای بالاتر **می‌نگارد** تا جدا شدنی شود.

```
کرنل خطی:      K(x, x') = x·x'
کرنل چندجمله‌ای: K(x, x') = (γ·x·x' + r)^d
کرنل RBF:      K(x, x') = exp(-γ·||x - x'||²)
کرنل سیگموید:  K(x, x') = tanh(γ·x·x' + r)
```

**نکته‌ی شگفت‌انگیز:** کرنل، ضرب داخلی در فضای بالا را **بدون محاسبه‌ی صریح** آن فضا حساب می‌کند. این «ترفند کرنل» است.

**۵. پارامتر `γ` (gamma) در RBF.**

| `γ` | اثر |
|---|---|
| کوچک | هر نقطه، تأثیر گسترده → مرز نرم، کم‌برازش |
| بزرگ | تأثیر محلی → مرز پیچیده، بیش‌برازش |

**۶. SVM برای رگرسیون: SVR.** همان ایده، اما با «لوله‌ی تحمل» به جای حاشیه.

**۷. چرا SVM امروز کمتر استفاده می‌شود؟**
- روی داده‌ی بزرگ کند است (O(n²) تا O(n³)).
- به مقیاس‌بندی و تنظیم حساس است.
- درخت‌ها و boosting، ساده‌تر و اغلب بهترند.
- **اما:** برای داده‌ی متوسط با ابعاد بالا (مثل متن با TF-IDF)، هنوز عالی است.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — SVM خطی و اثر C:**

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.svm import SVC
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler

X, y = make_classification(n_samples=800, n_features=2, n_informative=2,
                           n_redundant=0, n_clusters_per_class=1,
                           class_sep=0.8, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.25, stratify=y, random_state=42
)

for C in [0.01, 0.1, 1, 10, 100]:
    m = make_pipeline(StandardScaler(), SVC(kernel="linear", C=C))
    m.fit(X_train, y_train)
    print(f"C={C:>5} | train={m.score(X_train, y_train):.3f} | test={m.score(X_test, y_test):.3f}")
```

**مثال ۲ — مقایسه‌ی کرنل‌ها:**

```python
for kernel in ["linear", "poly", "rbf", "sigmoid"]:
    m = make_pipeline(StandardScaler(), SVC(kernel=kernel, C=1.0))
    m.fit(X_train, y_train)
    print(f"{kernel:8s} | test={m.score(X_test, y_test):.3f}")
```

**مثال ۳ — داده‌ی غیرخطی و کرنل RBF:**

```python
from sklearn.datasets import make_circles

# داده‌ی دایره‌ای — با خط جدا نمی‌شود
X_c, y_c = make_circles(n_samples=600, factor=0.4, noise=0.08, random_state=42)
Xtr, Xte, ytr, yte = train_test_split(X_c, y_c, test_size=0.25, stratify=y_c, random_state=42)

for kernel in ["linear", "rbf"]:
    m = make_pipeline(StandardScaler(), SVC(kernel=kernel, C=1.0, gamma="scale"))
    m.fit(Xtr, ytr)
    print(f"{kernel:7s}: {m.score(Xte, yte):.3f}")
```

کرنل خطی حدود ۰.۵ (تصادفی) و RBF نزدیک ۱.۰ می‌دهد. **این، قدرت کرنل را نشان می‌دهد.**

**مثال ۴ — اثر gamma:**

```python
for gamma in [0.01, 0.1, 1, 10, 100]:
    m = make_pipeline(StandardScaler(), SVC(kernel="rbf", C=1.0, gamma=gamma))
    m.fit(Xtr, ytr)
    print(f"gamma={gamma:>6} | train={m.score(Xtr, ytr):.3f} | test={m.score(Xte, yte):.3f}")
```

خروجی معمول: gamma بزرگ → train = ۱.۰، test افت می‌کند. **بیش‌برازش.**

**مثال ۵ — SVM برای متن (کاربرد واقعی امروز):**

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.pipeline import Pipeline

# نمونه‌ی کوچک
texts = [
    "این فیلم عالی بود", "خیلی بد بود", "عالی و دیدنی",
    "افتضاح", "لذت بردم", "وقت تلف کردن بود",
]
labels = [1, 0, 1, 0, 1, 0]

text_model = Pipeline([
    ("tfidf", TfidfVectorizer()),
    ("svm", SVC(kernel="linear", C=1.0)),
])
text_model.fit(texts, labels)
print("پیش‌بینی:", text_model.predict(["فیلم خوبی بود"]))
```

**نکته:** با داده‌ی کم، این فقط نمایش است. برای نتیجه‌ی واقعی، هزاران نمونه لازم است.

### مقایسه و دیدگاه‌های مختلف

| کرنل | مزیت | عیب | کِی |
|---|---|---|---|
| Linear | سریع، تفسیرپذیر | فقط خطی | ابعاد بالا (متن) |
| Polynomial | رابطه‌ی چندجمله‌ای | حساس به پارامتر | ویژگی تعاملی |
| RBF | عمومی، قوی | نیاز به تنظیم γ | پیش‌فرض مناسب |
| Sigmoid | شبیه شبکه‌ی عصبی | ناپایدار | کمتر استفاده می‌شود |

| مدل | داده‌ی بزرگ | ابعاد بالا | تفسیر |
|---|---|---|---|
| SVM | ضعیف | خوب | متوسط |
| Random Forest | خوب | متوسط | متوسط |
| Logistic | عالی | عالی | بالا |

**دو دیدگاه:**
- **«SVM منسوخ شده است.»** برای داده‌ی بزرگ و جدولی، تا حد زیادی درست.
- **«SVM برای متن و ابعاد بالا هنوز بهترین است.»** در برخی موارد درست: SVM خطی روی TF-IDF، سریع و دقیق است.

### اشتباهات رایج

**۱. SVM بدون مقیاس‌بندی.** مثل KNN، فاجعه.
**۲. `C` خیلی بزرگ.** حاشیه‌ی صفر، بیش‌برازش.
**۳. `gamma` خیلی بزرگ.** مرز پیچیده‌ی محلی، بیش‌برازش.
**۴. استفاده از SVM روی ۱ میلیون نمونه.** ساعت‌ها طول می‌کشد. `LinearSVC` یا SGD بهتر است.
**۵. فراموش کردن `probability=True` برای `predict_proba`.** پیش‌فرض خاموش است (چون کند است).
**۶. انتظار اهمیت ویژگی از کرنل غیرخطی.** `coef_` فقط برای کرنل خطی معنی دارد.

### نکات مهم و ترفندها

- **همیشه `StandardScaler` قبل از SVM.**
- **از RBF به‌عنوان پیش‌فرض شروع کن، بعد خطی را امتحان کن.**
- **`gamma="scale"`** پیش‌فرض معقولی است (۱/(n_features × X.var())).
- **برای داده‌ی بزرگ، `LinearSVC` یا `SGDClassifier`** به جای `SVC`.
- **میان‌بر:** SVM = حاشیه‌ی حداکثری · C = تحمل خطا · gamma = شعاع تأثیر · کرنل = نگاشت به فضای بالاتر.

### خلاصه فصل

- SVM مرزی با بیشترین حاشیه می‌سازد؛ فقط بردارهای پشتیبان تعیین‌کننده‌اند.
- `C` تعادل حاشیه/خطاست.
- کرنل، جداسازی غیرخطی را با نگاشت به فضای بالاتر ممکن می‌کند.
- `gamma` در RBF، شعاع تأثیر هر نقطه را تعیین می‌کند.
- SVM برای داده‌ی متوسط و ابعاد بالا (متن) هنوز ارزشمند است.

**سؤالات مروری:**
1. بردار پشتیبان چیست و چرا مهم است؟
2. اثر `C` کوچک و بزرگ چیست؟
3. چرا کرنل RBF روی داده‌ی دایره‌ای کار می‌کند اما کرنل خطی نه؟

---

## منابع و مراجع

**مستندات**
- scikit-learn — *Supervised learning* (بخش‌های Linear Models، Trees، Ensembles، SVM)
- scikit-learn — *Model selection: GridSearchCV, cross_val_score*

**کتاب‌ها**
- Géron, A. — *Hands-On Machine Learning*، فصل ۳–۷
- Hastie, T. et al. — *The Elements of Statistical Learning* (فصل ۳، ۹، ۱۲، ۱۵)
- James, G. et al. — *An Introduction to Statistical Learning*، فصل ۳، ۸

**مقالات کلیدی**
- Breiman, L. (2001) — *Random Forests*
- Friedman, J. (2001) — *Greedy Function Approximation: A Gradient Boosting Machine*
- Cortes, C. & Vapnik, V. (1995) — *Support-Vector Networks*

---

**بعدی:** [`10-Unsupervised-Learning.md`](./10-Unsupervised-Learning.md) — فصل ۴۴: خوشه‌بندی K-Means
