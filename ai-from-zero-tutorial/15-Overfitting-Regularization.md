# بخش ۱۵: بیش‌برازش و تنظیم

**مدت یادگیری تقریبی**: ۲ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۱۳ (آموزش) و ۱۴ (ارزیابی)

---

## فهرست محتوا

- [فصل ۶۶: overfitting و underfitting: تعریف و تشخیص](#فصل-۶۶-overfitting-و-underfitting-تعریف-و-تشخیص)
- [فصل ۶۷: منحنی یادگیری و تفسیر آن](#فصل-۶۷-منحنی-یادگیری-و-تفسیر-آن)
- [فصل ۶۸: Regularization: L1، L2، Dropout](#فصل-۶۸-regularization-l1-l2-dropout)
- [فصل ۶۹: تنظیم فراپارامترها](#فصل-۶۹-تنظیم-فراپارامترها)
- [فصل ۷۰: مثلث تنظیم: epoch، batch size، learning rate](#فصل-۷۰-مثلث-تنظیم-epoch-batch-size-learning-rate)

---

## فصل ۶۶: overfitting و underfitting: تعریف و تشخیص

### مقدمه

**بیش‌برازش (Overfitting)** و **کم‌برازش (Underfitting)** دو روی یک سکه‌اند: مدل یا خیلی ساده است یا خیلی پیچیده.

این فصل، مهم‌ترین مهارت تشخیصی در ML را می‌دهد: **با یک نگاه به اعداد train و test، بفهمی کدام مشکل را داری.**

### مفاهیم اساسی

**۱. سه حالت:**

| حالت | train | test | تشخیص |
|---|---|---|---|
| **Underfitting** | پایین | پایین | مدل خیلی ساده است |
| **خوب (Good fit)** | بالا | بالا و نزدیک به train | هدف |
| **Overfitting** | خیلی بالا | پایین (شکاف بزرگ) | مدل خیلی پیچیده است |

**۲. تعریف شهودی:**

- **Underfitting:** دانش‌آموزی که کتاب را نخوانده — در تمرین و امتحان هر دو ضعیف.
- **Overfitting:** دانش‌آموزی که فقط پاسخ‌های تمرین را حفظ کرده — در تمرین ۱۰۰٪، در امتحان ۴۰٪.
- **Good fit:** دانش‌آموزی که مفهوم را فهمیده — در هر دو خوب.

**۳. منبع overfitting:**

مدل به‌قدری پیچیده است که **نویز** داده‌ی آموزش را هم یاد می‌گیرد، نه فقط الگو را.

```
y = الگو + نویز
مدل خوب: الگو را یاد می‌گیرد
مدل overfit: الگو + نویز را حفظ می‌کند
```

**۴. منبع underfitting:**
- مدل خیلی ساده (رگرسیون خطی برای رابطه‌ی پیچیده)
- ویژگی‌های ناکافی
- آموزش ناکافی (epoch کم، lr خیلی کوچک)
- منظم‌سازی بیش از حد

**۵. تعادل بایاس-واریانس (Bias-Variance Tradeoff).**

```
خطای کل = بایاس² + واریانس + خطای برگشت‌ناپذیر
```

- **بایاس:** خطای ناشی از فرض‌های ساده‌ی مدل. بایاس بالا → underfitting.
- **واریانس:** حساسیت مدل به داده‌ی آموزش. واریانس بالا → overfitting.

**نمودار ذهنی:**
```
خطا
 │  underfit        overfit
 │  \                  /
 │   \                /
 │    \______________/
 │         sweet spot
 └──────────────────────────► پیچیدگی مدل
```

**۶. هفت نشانه‌ی overfitting:**

1. شکاف بزرگ بین train و validation.
2. loss روی validation بعد از مدتی **بالا می‌رود** (در حالی که train پایین می‌آید).
3. مدل روی داده‌ی جدید، عملکرد بسیار بد.
4. وزن‌های بسیار بزرگ.
5. درخت با عمق بی‌نهایت.
6. حساسیت شدید به نویز.
7. performance عالی روی train با داده‌ی کم.

**۷. شش راه‌حل overfitting:**

| راه‌حل | توضیح |
|---|---|
| **داده‌ی بیشتر** | بهترین راه، اگر ممکن است |
| **کاهش پیچیدگی** | عمق کمتر، ویژگی کمتر |
| **Regularization** | جریمه‌ی وزن‌های بزرگ |
| **Early Stopping** | توقف قبل از overfit |
| **Dropout** | در شبکه‌ی عصبی |
| **Data Augmentation** | داده‌ی مصنوعی |

**۸. سه راه‌حل underfitting:**

| راه‌حل | توضیح |
|---|---|
| **مدل پیچیده‌تر** | ویژگی بیشتر، عمق بیشتر |
| **ویژگی بهتر** | مهندسی ویژگی |
| **کاهش منظم‌سازی** | کمتر محدود کن |

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — دیدن overfitting با درجه‌ی چندجمله‌ای:**

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression
from sklearn.pipeline import make_pipeline
from sklearn.metrics import mean_squared_error

rng = np.random.default_rng(0)
# تابع واقعی: sin(x) با نویز
X = np.sort(rng.uniform(0, 2 * np.pi, 60)).reshape(-1, 1)
y = np.sin(X.ravel()) + rng.normal(0, 0.25, 60)

X_plot = np.linspace(0, 2 * np.pi, 300).reshape(-1, 1)
y_true = np.sin(X_plot.ravel())

fig, axes = plt.subplots(1, 3, figsize=(15, 4))

for ax, degree in zip(axes, [1, 3, 15]):
    model = make_pipeline(PolynomialFeatures(degree), LinearRegression()).fit(X, y)
    pred = model.predict(X_plot)

    train_rmse = np.sqrt(mean_squared_error(y, model.predict(X)))
    test_rmse = np.sqrt(mean_squared_error(y_true, pred))

    ax.scatter(X, y, s=18, alpha=0.6, label="داده")
    ax.plot(X_plot, y_true, "g--", label="واقعی")
    ax.plot(X_plot, pred, "r-", label=f"درجه {degree}")
    ax.set_title(f"درجه {degree}\ntrain RMSE={train_rmse:.2f} | true RMSE={test_rmse:.2f}")
    ax.legend(fontsize=8)

plt.tight_layout()
plt.show()
```

**خروجی:**
- درجه ۱: underfit (خط راست برای سینوس).
- درجه ۳: خوب.
- درجه ۱۵: overfit (منحنی پیچیده که نویز را دنبال می‌کند).

**مثال ۲ — جدول عددی تشخیص:**

```python
from sklearn.tree import DecisionTreeClassifier
from sklearn.model_selection import train_test_split
from sklearn.datasets import make_classification
from sklearn.metrics import accuracy_score

X, y = make_classification(n_samples=1000, n_features=20, n_informative=6,
                           n_redundant=10, flip_y=0.1, random_state=42)
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.25, random_state=42)

print(f"{'max_depth':>10} {'train':>8} {'test':>8} {'شکاف':>8} {'تشخیص':>14}")
for depth in [1, 2, 3, 5, 8, 12, 20, None]:
    t = DecisionTreeClassifier(max_depth=depth, random_state=42).fit(X_tr, y_tr)
    tr = accuracy_score(y_tr, t.predict(X_tr))
    te = accuracy_score(y_te, t.predict(X_te))
    gap = tr - te

    if tr < 0.75:
        verdict = "underfit"
    elif gap > 0.12:
        verdict = "overfit"
    else:
        verdict = "خوب"
    print(f"{str(depth):>10} {tr:>8.3f} {te:>8.3f} {gap:>8.3f} {verdict:>14}")
```

**این جدول، بهترین ابزار تشخیص است.** به دنبال «شکاف» بگرد.

**مثال ۳ — overfitting با داده‌ی کم:**

```python
from sklearn.neighbors import KNeighborsClassifier

for n_train in [20, 50, 200, 800]:
    X_s, y_s = make_classification(n_samples=n_train + 300, n_features=20,
                                   n_informative=6, random_state=42)
    X_tr, X_te = X_s[:n_train], X_s[n_train:]
    y_tr, y_te = y_s[:n_train], y_s[n_train:]

    knn = KNeighborsClassifier(n_neighbors=1).fit(X_tr, y_tr)
    print(f"n_train={n_train:>4} | train={knn.score(X_tr, y_tr):.3f} | "
          f"test={knn.score(X_te, y_te):.3f}")
```

**k=1 همیشه train=1.0 می‌دهد** (هر نقطه، نزدیک‌ترین همسایه‌ی خودش است). با داده‌ی کم، test فاجعه است.

### مقایسه و دیدگاه‌های مختلف

| | Underfitting | Overfitting |
|---|---|---|
| بایاس | بالا | پایین |
| واریانس | پایین | بالا |
| train score | پایین | خیلی بالا |
| test score | پایین | پایین |
| راه‌حل | پیچیده‌تر | ساده‌تر + داده بیشتر |

**یک دیدگاه مهم:** «در عمل، overfitting خطر کمتری از underfitting ندارد.» بعضی‌ها فکر می‌کنند overfitting بدتر است. اما مدل underfit، **هیچ ارزشی ندارد**؛ مدل overfit حداقل روی داده‌ی آموزش کار می‌کند و با تنظیم قابل نجات است. **هر دو بد هستند، اما underfitting اغلب نادیده گرفته می‌شود.**

### اشتباهات رایج

**۱. نگاه کردن فقط به train accuracy.** همیشه test را ببین.
**۲. تلاش برای دقت ۱۰۰٪ روی train.** نشانه‌ی قطعی overfit.
**۳. اضافه کردن داده بدون بررسی کیفیت.** داده‌ی نویزی، overfit را بدتر می‌کند.
**۴. Regularization خیلی قوی.** underfit می‌شود.
**۵. نتیجه‌گیری از یک تقسیم.** CV لازم است.
**۶. نادیده گرفتن نویز برچسب.** با ۱۰٪ برچسب غلط، سقف دقت ~۹۰٪ است.

### نکات مهم و ترفندها

- **همیشه train و validation را با هم ببین.** یک عدد، هیچ می‌گوید.
- **شکاف بیش از ۱۰٪ = هشدار.** بیش از ۲۰٪ = overfit جدی.
- **اول داده‌ی بیشتر، بعد Regularization.** داده، بهترین درمان است.
- **`learning_curve` را رسم کن.** می‌گوید داده‌ی بیشتر کمک می‌کند یا نه (فصل بعد).
- **میان‌بر:** train بالا + test پایین = overfit · هر دو پایین = underfit.

### خلاصه فصل

- Overfitting: حفظ کردن داده‌ی آموزش. Underfitting: یاد نگرفتن الگو.
- تعادل بایاس-واریانس، چارچوب نظری است.
- نشانه‌ی اصلی overfit، شکاف بین train و test است.
- شش راه‌حل: داده بیشتر، سادگی، regularization، early stopping، dropout، augmentation.
- Underfitting هم به همان اندازه مهم است.

**سؤالات مروری:**
1. تفاوت overfitting و underfitting در جدول train/test چیست؟
2. بایاس و واریانس را تعریف کن.
3. سه راه‌حل overfitting و سه راه‌حل underfitting بگو.

---

## فصل ۶۷: منحنی یادگیری و تفسیر آن

### مقدمه

دو منحنی وجود دارد که **تشخیص را از حدس به علم** تبدیل می‌کنند:

- **منحنی یادگیری (Learning Curve):** عملکرد در برابر **حجم داده‌ی آموزش**.
- **منحنی آموزش (Training Curve):** loss در برابر **epoch**.

این فصل، خواندن هر دو را یاد می‌دهد. این مهارت، در پروژه‌های واقعی **هر روز** به کار می‌آید.

### مفاهیم اساسی

**۱. منحنی آموزش (Loss vs Epoch) — چهار الگو:**

```
الف) خوب:                    ب) Overfit:
train ↘، val ↘ سپس ↗          train ↘↘، val ↘ سپس ↗↗
                                (شکاف بزرگ)

ج) Underfit:                 د) مشکل داده/آموزش:
هر دو بالا و صاف              loss می‌پرد، NaN
```

**۲. تشخیص از روی منحنی آموزش:**

| الگو | تشخیص | اقدام |
|---|---|---|
| train↓ val↓ | سالم | ادامه بده |
| train↓ val↓ سپس ↑ | overfit | early stopping |
| هر دو بالا و ثابت | underfit یا lr خیلی کم | مدل بزرگ‌تر، lr بیشتر |
| loss می‌پرد | lr خیلی بزرگ | lr را کم کن |
| loss = NaN | lr بزرگ یا باگ | lr را ۱۰× کم کن |
| train صفر، val بد | نشت داده یا باگ | داده را بررسی کن |

**۳. Early Stopping.** بهترین راه جلوگیری از overfit:

```python
best_val, patience_counter = inf, 0
for epoch in range(max_epochs):
    train(...)
    val_loss = evaluate(...)
    if val_loss < best_val:
        best_val, patience_counter = val_loss, 0
        save_model()
    else:
        patience_counter += 1
        if patience_counter >= patience:
            break
```

**۴. منحنی یادگیری (Performance vs Training Size).**

```
داده را به ۱۰٪، ۲۰٪، ... ۱۰۰٪ تقسیم کن
برای هر کدام آموزش بده و train/val را بسنج
```

**سه الگو:**

| الگو | تشخیص | اقدام |
|---|---|---|
| **شکاف بزرگ، val پایین** | overfit | داده‌ی بیشتر کمک می‌کند |
| **هر دو نزدیک، هر دو پایین** | underfit | مدل پیچیده‌تر |
| **هر دو نزدیک، هر دو بالا** | خوب | تمام |

**این نمودار، به سؤال کلیدی جواب می‌دهد: «آیا جمع‌آوری داده‌ی بیشتر فایده دارد؟»** اگر منحنی val هنوز صعودی است، بله.

**۵. منحنی اعتبارسنجی (Validation Curve).** عملکرد در برابر یک فراپارامتر (مثلاً `max_depth`):

```
train ↘ (با پیچیدگی)، val ∩ (زنگوله)
```
نقطه‌ی اوج val، بهترین مقدار است.

**۶. نکته‌ی مهم:** منحنی‌ها را روی **validation** بکش، نه test.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — منحنی آموزش با sklearn:**

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.tree import DecisionTreeClassifier
from sklearn.metrics import log_loss

X, y = make_classification(n_samples=2000, n_features=20, n_informative=8,
                           flip_y=0.08, random_state=42)
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.25, random_state=42)

model = DecisionTreeClassifier(max_depth=None, random_state=42)

# شبیه‌سازی آموزش مرحله‌به‌مرحله با تغییر max_depth
depths = range(1, 25)
train_loss, val_loss = [], []

for d in depths:
    m = DecisionTreeClassifier(max_depth=d, random_state=42).fit(X_tr, y_tr)
    train_loss.append(log_loss(y_tr, m.predict_proba(X_tr)))
    val_loss.append(log_loss(y_te, m.predict_proba(X_te)))

best_d = list(depths)[int(np.argmin(val_loss))]
plt.figure(figsize=(9, 5))
plt.plot(list(depths), train_loss, "o-", label="train")
plt.plot(list(depths), val_loss, "s-", label="validation")
plt.axvline(best_d, color="red", ls="--", label=f"بهترین عمق={best_d}")
plt.xlabel("max_depth")
plt.ylabel("log loss")
plt.title("منحنی اعتبارسنجی — نقطه‌ی شیرین")
plt.legend()
plt.grid(alpha=0.3)
plt.show()

print(f"بهترین عمق: {best_d}")
print(f"train loss: {train_loss[best_d-1]:.3f} | val loss: {val_loss[best_d-1]:.3f}")
```

**مثال ۲ — منحنی یادگیری:**

```python
from sklearn.model_selection import learning_curve

train_sizes, train_scores, val_scores = learning_curve(
    DecisionTreeClassifier(max_depth=8, random_state=42),
    X, y, cv=5, scoring="accuracy",
    train_sizes=np.linspace(0.1, 1.0, 8),
    n_jobs=-1,
)

tr_mean, tr_std = train_scores.mean(1), train_scores.std(1)
va_mean, va_std = val_scores.mean(1), val_scores.std(1)

plt.figure(figsize=(9, 5))
plt.plot(train_sizes, tr_mean, "o-", label="train")
plt.fill_between(train_sizes, tr_mean - tr_std, tr_mean + tr_std, alpha=0.15)
plt.plot(train_sizes, va_mean, "s-", label="validation")
plt.fill_between(train_sizes, va_mean - va_std, va_mean + va_std, alpha=0.15)
plt.xlabel("تعداد نمونه‌ی آموزش")
plt.ylabel("accuracy")
plt.title("منحنی یادگیری")
plt.legend()
plt.grid(alpha=0.3)
plt.show()

print(f"شکاف در ۱۰٪ داده: {tr_mean[0] - va_mean[0]:.3f}")
print(f"شکاف در ۱۰۰٪ داده: {tr_mean[-1] - va_mean[-1]:.3f}")
```

**تفسیر:**
- اگر شکاف با افزایش داده کم می‌شود → داده‌ی بیشتر کمک می‌کند.
- اگر شکاف ثابت می‌ماند → مشکل مدل است، نه داده.

**مثال ۳ — early stopping دستی:**

```python
from sklearn.neural_network import MLPClassifier
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler().fit(X_tr)
X_tr_s, X_te_s = scaler.transform(X_tr), scaler.transform(X_te)

mlp = MLPClassifier(hidden_layer_sizes=(64, 32), max_iter=1, warm_start=True,
                    random_state=42, learning_rate_init=0.01)

best_val, patience, wait, best_iter = np.inf, 10, 0, 0
train_curve, val_curve = [], []

for epoch in range(300):
    mlp.fit(X_tr_s, y_tr)
    tr = log_loss(y_tr, mlp.predict_proba(X_tr_s))
    va = log_loss(y_te, mlp.predict_proba(X_te_s))
    train_curve.append(tr)
    val_curve.append(va)

    if va < best_val - 1e-4:
        best_val, wait, best_iter = va, 0, epoch
    else:
        wait += 1
        if wait >= patience:
            print(f"توقف در epoch {epoch} (بهترین: {best_iter})")
            break

plt.figure(figsize=(9, 4))
plt.plot(train_curve, label="train")
plt.plot(val_curve, label="validation")
plt.axvline(best_iter, color="red", ls="--", label="best")
plt.xlabel("epoch"); plt.ylabel("log loss"); plt.legend(); plt.grid(alpha=0.3)
plt.show()

print(f"بهترین val loss: {best_val:.4f}")
```

**مثال ۴ — تشخیص lr نامناسب از منحنی:**

```python
def diagnose_curve(train_losses, val_losses):
    """تشخیص خودکار از منحنی."""
    if any(not np.isfinite(v) for v in train_losses[-5:]):
        return "lr خیلی بزرگ (NaN)"
    if len(train_losses) > 3 and train_losses[-1] > train_losses[0]:
        return "lr خیلی بزرگ (واگرا)"
    if train_losses[-1] > 0.7 * train_losses[0]:
        return "lr خیلی کوچک یا مدل خیلی ضعیف"
    if val_losses[-1] > min(val_losses) * 1.05:
        return "overfitting — early stopping"
    return "سالم"

# آزمایش
rng = np.random.default_rng(0)
print(diagnose_curve(list(np.linspace(1, 0.1, 50)), list(np.linspace(1, 0.2, 50))))
print(diagnose_curve(list(np.linspace(1, 0.01, 50)), list(np.r_[np.linspace(1, 0.2, 30), np.linspace(0.2, 0.5, 20)])))
```

### مقایسه و دیدگاه‌های مختلف

| منحنی | چه می‌گوید | چه نمی‌گوید |
|---|---|---|
| Loss vs epoch | overfit/underfit، lr | آیا داده بیشتر کمک می‌کند |
| Performance vs size | ارزش داده‌ی بیشتر | کدام فراپارامتر بهتر است |
| Performance vs hyperparam | بهترین مقدار | آیا مدل مناسب است |

**یک دیدگاه:** «منحنی‌ها برای پژوهش‌اند، نه تولید.» اشتباه: در تولید هم برای پایش (drift) و تشخیص افت، منحنی لازم است.

### اشتباهات رایج

**۱. رسم منحنی روی test.** باید validation باشد.
**۲. نتیجه‌گیری از یک نقطه.** منحنی لازم است.
**۳. نادیده گرفتن نوسان.** loss پرنویز است؛ میانگین متحرک بگیر.
**۴. early stopping با patience=۰.** خیلی زود متوقف می‌شود.
**۵. تفسیر منحنی یادگیری صعودی به‌عنوان «داده بیشتر بی‌فایده».** ممکن است مدل ظرفیت نداشته باشد.
**۶. مقیاس نامناسب محور y.** تفاوت‌های کوچک را بزرگ نشان می‌دهد.

### نکات مهم و ترفندها

- **همیشه منحنی آموزش را رسم کن.** حتی در پروژه‌های کوچک.
- **`patience=10` تا `20`** برای early stopping معمول است.
- **میانگین متحرک (EMA) روی loss بگذار** تا نویز کم شود.
- **`learning_curve` به سؤال «داده بیشتر بخرم؟» جواب می‌دهد.**
- **میان‌بر:** loss↑val + train↓ = overfit · هر دو بالا = underfit · شکاف بزرگ = داده بیشتر.

### خلاصه فصل

- منحنی آموزش، overfit/underfit را نشان می‌دهد.
- منحنی یادگیری، ارزش داده‌ی بیشتر را می‌سنجد.
- منحنی اعتبارسنجی، بهترین فراپارامتر را پیدا می‌کند.
- Early stopping ساده‌ترین و مؤثرترین راه جلوگیری از overfit است.
- شکل منحنی، تشخیص را از حدس به علم تبدیل می‌کند.

**سؤالات مروری:**
1. چهار الگوی منحنی آموزش را نام ببر.
2. منحنی یادگیری با شکاف بزرگ چه می‌گوید؟
3. Early stopping چگونه کار می‌کند؟

---

## فصل ۶۸: Regularization: L1، L2، Dropout

### مقدمه

Regularization یعنی **«مدل را عمداً محدود کن تا ساده‌تر شود»**. ایده‌ی متناقض‌نما: اگر مدل را کمی «ضعیف‌تر» کنی، روی داده‌ی جدید بهتر عمل می‌کند.

این فصل، چهار ابزار اصلی را می‌دهد: L1، L2، Dropout و Early Stopping.

### مفاهیم اساسی

**۱. L2 (Ridge) — جریمه‌ی مربع وزن‌ها.**

```
L_total = L_data + λ·Σw²
```

**اثر:** وزن‌ها را کوچک می‌کند، اما صفر نمی‌کند.

**گرادیان:** `∂/∂w = 2λw` — هر وزن، به نسبت اندازه‌اش، به سمت صفر کشیده می‌شود.

**۲. L1 (Lasso) — جریمه‌ی قدر مطلق.**

```
L_total = L_data + λ·Σ|w|
```

**اثر:** بعضی وزن‌ها را **دقیقاً صفر** می‌کند → انتخاب ویژگی خودکار.

**چرا؟** گرادیان L1 ثابت است (`λ·sign(w)`)، پس وزن‌های کوچک را کاملاً به صفر می‌رساند.

**۳. ElasticNet — ترکیب.**

```
L_total = L_data + λ₁·Σ|w| + λ₂·Σw²
```
وقتی ویژگی‌های همبسته زیادند، بهتر از Lasso تنها.

**۴. انتخاب λ:**
- `λ = 0`: بدون منظم‌سازی (overfit).
- `λ` بزرگ: underfit.
- انتخاب با CV.

در sklearn: `alpha` = λ. و `C` = 1/λ (در SVM و Logistic).

**۵. Dropout — منظم‌سازی در شبکه‌ی عصبی.**

در هر گام آموزش، تصادفی `p` درصد نورون‌ها را صفر می‌کند.

**چرا کار می‌کند؟** سه تفسیر:
- **جلوگیری از هم‌انطباقی (co-adaptation):** نورون‌ها نمی‌توانند به هم وابسته شوند.
- **میانگین‌گیری از مدل‌ها:** مثل آموزش هزاران زیرشبکه.
- **نویز:** مدل را مقاوم می‌کند.

**نکته‌ی حیاتی:** Dropout فقط در آموزش. در استنتاج، همه فعال‌اند.

**۶. Early Stopping.** توقف وقتی validation بهتر نمی‌شود. ساده‌ترین و ارزان‌ترین regularization.

**۷. Data Augmentation.** تولید داده‌ی مصنوعی از داده‌ی موجود:
- **تصویر:** چرخش، برش، تغییر رنگ، flip.
- **متن:** جایگزینی کلمه، back-translation.
- **صدا:** تغییر سرعت، نویز.

**۸. سایر روش‌ها:**
- **BatchNorm:** اثر جانبی regularization دارد.
- **Weight decay:** نام دیگر L2 در بهینه‌سازها.
- **Label smoothing:** جلوگیری از بیش‌اطمینانی.
- **Noise injection:** افزودن نویز به ورودی یا وزن.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — مقایسه‌ی L1 و L2:**

```python
import numpy as np
from sklearn.datasets import make_regression
from sklearn.linear_model import LinearRegression, Ridge, Lasso, ElasticNet
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import make_pipeline
from sklearn.model_selection import cross_val_score

X, y = make_regression(n_samples=300, n_features=50, n_informative=10,
                       noise=15, random_state=42)

models = {
    "Linear (λ=0)": LinearRegression(),
    "Ridge α=1": Ridge(alpha=1.0),
    "Ridge α=100": Ridge(alpha=100.0),
    "Lasso α=1": Lasso(alpha=1.0, max_iter=10_000),
    "Lasso α=10": Lasso(alpha=10.0, max_iter=10_000),
    "ElasticNet": ElasticNet(alpha=1.0, l1_ratio=0.5, max_iter=10_000),
}

print(f"{'مدل':>14} {'CV R²':>10} {'ویژگی صفر':>12} {'||w||':>10}")
for name, m in models.items():
    pipe = make_pipeline(StandardScaler(), m)
    r2 = cross_val_score(pipe, X, y, cv=5, scoring="r2").mean()
    pipe.fit(X, y)
    coefs = pipe.named_steps[list(pipe.named_steps)[-1]].coef_
    print(f"{name:>14} {r2:>10.3f} {int((coefs == 0).sum()):>12} {np.linalg.norm(coefs):>10.3f}")
```

**خروجی معمول:**
- Linear: R² خوب اما همه‌ی ۵۰ ویژگی غیرصفر.
- Ridge: وزن‌های کوچک‌تر، R² مشابه یا بهتر.
- Lasso α=10: بیشتر ویژگی‌ها صفر → مدل ساده‌تر.

**مثال ۲ — انتخاب alpha با CV:**

```python
from sklearn.linear_model import RidgeCV, LassoCV

ridge_cv = make_pipeline(StandardScaler(),
                         RidgeCV(alphas=np.logspace(-3, 3, 20), cv=5)).fit(X, y)
print("بهترین alpha برای Ridge:", ridge_cv.named_steps["ridgecv"].alpha_)

lasso_cv = make_pipeline(StandardScaler(),
                         LassoCV(alphas=np.logspace(-3, 3, 20), cv=5,
                                 max_iter=20_000)).fit(X, y)
print("بهترین alpha برای Lasso:", lasso_cv.named_steps["lassocv"].alpha_)
```

**مثال ۳ — اثر Dropout در PyTorch:**

```python
try:
    import torch
    import torch.nn as nn
    from torch.utils.data import DataLoader, TensorDataset

    X_t = torch.tensor(X, dtype=torch.float32)
    y_t = torch.tensor(y, dtype=torch.float32).view(-1, 1)
    X_t = (X_t - X_t.mean(0)) / (X_t.std(0) + 1e-8)

    def make_model(dropout):
        return nn.Sequential(
            nn.Linear(50, 128), nn.ReLU(), nn.Dropout(dropout),
            nn.Linear(128, 64), nn.ReLU(), nn.Dropout(dropout),
            nn.Linear(64, 1),
        )

    loader = DataLoader(TensorDataset(X_t, y_t), batch_size=32, shuffle=True)

    for p in [0.0, 0.2, 0.5]:
        torch.manual_seed(0)
        model = make_model(p)
        opt = torch.optim.Adam(model.parameters(), lr=1e-3)
        loss_fn = nn.MSELoss()

        for epoch in range(150):
            model.train()
            for xb, yb in loader:
                opt.zero_grad()
                loss = loss_fn(model(xb), yb)
                loss.backward()
                opt.step()

        model.eval()
        with torch.no_grad():
            train_mse = loss_fn(model(X_t), y_t).item()
        print(f"dropout={p:<4} | train MSE={train_mse:8.2f}")
except ImportError:
    print("pip install torch")
```

**توجه:** با dropout زیاد، train MSE بالاتر می‌رود اما generalization بهتر می‌شود. **همیشه روی validation بسنج، نه train.**

**مثال ۴ — Data Augmentation برای تصویر:**

```python
try:
    from torchvision import transforms

    train_transform = transforms.Compose([
        transforms.RandomResizedCrop(224),
        transforms.RandomHorizontalFlip(),
        transforms.RandomRotation(15),
        transforms.ColorJitter(brightness=0.2, contrast=0.2, saturation=0.2),
        transforms.ToTensor(),
        transforms.Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225]),
    ])

    val_transform = transforms.Compose([
        transforms.Resize(256),
        transforms.CenterCrop(224),
        transforms.ToTensor(),
        transforms.Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225]),
    ])

    print("augmentation آموزش:", train_transform)
    print("\nتوجه: در ارزیابی، augmentation تصادفی نداریم!")
except ImportError:
    print("pip install torchvision")
```

**نکته‌ی مهم:** هرگز در validation/test از augmentation تصادفی استفاده نکن. فقط resize و normalize.

**مثال ۵ — جدول تصمیم:**

```python
def suggest_regularization(gap, train_score, has_nn=False):
    if train_score < 0.7:
        return "underfit — regularization را کم کن، مدل را پیچیده‌تر کن"
    if gap < 0.05:
        return "شکاف کم — regularization لازم نیست"
    if gap < 0.15:
        base = "L2 متوسط (alpha=1)"
    else:
        base = "L2 قوی (alpha=10) + early stopping"
    return base + (" + Dropout 0.3" if has_nn else "")

print(suggest_regularization(0.02, 0.85))
print(suggest_regularization(0.12, 0.92))
print(suggest_regularization(0.25, 0.99, has_nn=True))
```

### مقایسه و دیدگاه‌های مختلف

| روش | اثر | هزینه | کِی |
|---|---|---|---|
| L2 | وزن‌های کوچک | صفر | پیش‌فرض |
| L1 | وزن‌های صفر (انتخاب ویژگی) | صفر | ویژگی زیاد |
| Dropout | مقاوم‌سازی شبکه | کمی کندتر | شبکه‌ی عصبی |
| Early Stopping | توقف به‌موقع | صفر | همیشه |
| Augmentation | داده‌ی بیشتر | زمان آموزش | تصویر/متن/صدا |
| BatchNorm | پایدارسازی | کم | شبکه‌ی عمیق |

**یک دیدگاه:** «Dropout منسوخ شده است.» در ترانسفورمرهای مدرن، کمتر استفاده می‌شود (به‌جایش weight decay و label smoothing). اما در MLP و CNN کلاسیک، هنوز مفید است.

### اشتباهات رایج

**۱. Dropout در استنتاج.** `model.eval()` فراموش شود.
**۲. L1 بدون استانداردسازی.** جریمه به مقیاس وابسته است.
**۳. λ خیلی بزرگ.** underfit.
**۴. Augmentation در validation.** ارزیابی را ناپایدار می‌کند.
**۵. Augmentation نامناسب.** چرخش ۱۸۰ درجه برای تشخیص رقم، «۶» را «۹» می‌کند.
**۶. تکیه بر یک روش.** ترکیب چند روش، معمولاً بهتر است.
**۷. Regularization برای جبران داده‌ی بد.** داده‌ی نویزی را درست نمی‌کند.

### نکات مهم و ترفندها

- **L2 را پیش‌فرض بگذار.** ارزان و مؤثر.
- **L1 برای انتخاب ویژگی** وقتی ویژگی زیاد و تفسیر مهم است.
- **Dropout ۰.۲–۰.۵** معمول است. بیشتر، underfit.
- **`weight_decay=1e-4`** در AdamW، معادل L2 است.
- **Early stopping را همیشه فعال کن.** رایگان است.
- **میان‌بر:** L1 = صفر کردن · L2 = کوچک کردن · Dropout = تصادفی خاموش کردن.

### خلاصه فصل

- Regularization، مدل را ساده‌تر می‌کند تا بهتر generalize کند.
- L1 وزن‌ها را صفر می‌کند (انتخاب ویژگی)، L2 کوچک.
- Dropout در آموزش تصادفی نورون‌ها را خاموش می‌کند.
- Early stopping ساده‌ترین و ارزان‌ترین روش است.
- Augmentation، داده‌ی مصنوعی می‌سازد.

**سؤالات مروری:**
1. چرا L1 وزن‌ها را دقیقاً صفر می‌کند و L2 نه؟
2. Dropout چرا فقط در آموزش اعمال می‌شود؟
3. کِی از Augmentation استفاده می‌کنی و کِی نه؟

---

## فصل ۶۹: تنظیم فراپارامترها

### مقدمه

فراپارامترها، تنظیمات مدل قبل از آموزش‌اند: `max_depth`، `n_estimators`، `alpha`، `learning_rate`. **انتخاب درست آن‌ها، اغلب تفاوت بین مدل متوسط و مدل عالی است.**

این فصل، از جست‌وجوی ساده تا Bayesian optimization را پوشش می‌دهد.

### مفاهیم اساسی

**۱. چهار روش جست‌وجو:**

| روش | ایده | تعداد ارزیابی | کِی |
|---|---|---|---|
| **Manual** | حدس و آزمایش | کم | شروع |
| **Grid Search** | همه‌ی ترکیب‌ها | انفجاری | فضای کوچک |
| **Random Search** | نمونه‌گیری تصادفی | قابل تنظیم | فضای متوسط/بزرگ |
| **Bayesian (Optuna)** | مدل‌سازی تابع هدف | کم و هوشمند | فضای بزرگ |

**۲. چرا Random بهتر از Grid است؟**

تحقیق Bergstra & Bengio (2012): اگر فقط ۲ فراپارامتر از ۱۰ فراپارامتر مهم باشند، Grid بیشتر وقتش را روی فراپارامترهای بی‌اثر تلف می‌کند. Random، فضای مهم را بهتر پوشش می‌دهد.

**۳. مقیاس لگاریتمی.**

برای `alpha`، `learning_rate`، `C`، `gamma` — از `logspace` استفاده کن:

```python
np.logspace(-4, 1, 20)     # 0.0001 تا 10
```
نه `linspace`، چون ۰.۰۰۰۱ و ۰.۰۰۱ تفاوت مهمی دارند اما `linspace` آن‌ها را نادیده می‌گیرد.

**۴. فراپارامترهای کلیدی هر مدل:**

| مدل | مهم‌ترین فراپارامترها |
|---|---|
| Random Forest | `n_estimators`, `max_depth`, `max_features`, `min_samples_leaf` |
| Gradient Boosting | `learning_rate`, `n_estimators`, `max_depth`, `subsample` |
| SVM | `C`, `gamma`, `kernel` |
| Logistic | `C`, `penalty` |
| KNN | `n_neighbors`, `weights`, `metric` |
| Neural Net | `lr`, `hidden_size`, `dropout`, `batch_size` |

**۵. Optuna — روش مدرن.**

```python
def objective(trial):
    params = {
        "n_estimators": trial.suggest_int("n_estimators", 100, 1000),
        "max_depth": trial.suggest_int("max_depth", 3, 20),
        "learning_rate": trial.suggest_float("learning_rate", 1e-4, 1e-1, log=True),
    }
    ...
    return score

study = optuna.create_study(direction="maximize")
study.optimize(objective, n_trials=50)
```

**۶. نکات مهم:**
- **همیشه CV، نه یک تقسیم.**
- **مقیاس لگاریتمی برای پارامترهای مثبت نامحدود.**
- **فضای جست‌وجو را منطقی محدود کن.** `max_depth` بین ۱ تا ۵۰، نه ۱ تا ۱۰۰۰۰.
- **`n_jobs=-1` را فراموش نکن.**
- **از بهترین نتیجه در test استفاده نکن.** روی validation تنظیم، روی test گزارش.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — Grid Search کامل:**

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split, GridSearchCV, StratifiedKFold
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report

X, y = make_classification(n_samples=2000, n_features=20, n_informative=10,
                           weights=[0.8], flip_y=0.05, random_state=42)
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.25, stratify=y, random_state=42)

param_grid = {
    "n_estimators": [100, 300],
    "max_depth": [5, 10, None],
    "min_samples_leaf": [1, 3, 5],
    "max_features": ["sqrt", "log2"],
}

cv = StratifiedKFold(5, shuffle=True, random_state=42)
gs = GridSearchCV(
    RandomForestClassifier(random_state=42, n_jobs=-1),
    param_grid, cv=cv, scoring="f1", n_jobs=-1, verbose=1
)
gs.fit(X_tr, y_tr)

print("بهترین:", gs.best_params_)
print(f"CV F1: {gs.best_score_:.3f}")
print(classification_report(y_te, gs.predict(X_te), digits=3))
```

**مثال ۲ — Random Search:**

```python
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import randint, loguniform

param_dist = {
    "n_estimators": randint(100, 800),
    "max_depth": randint(3, 25),
    "min_samples_leaf": randint(1, 10),
    "max_features": loguniform(0.1, 1.0),
}

rs = RandomizedSearchCV(
    RandomForestClassifier(random_state=42),
    param_dist, n_iter=40, cv=cv, scoring="f1",
    n_jobs=-1, random_state=42, verbose=1
)
rs.fit(X_tr, y_tr)

print("بهترین:", rs.best_params_)
print(f"CV F1: {rs.best_score_:.3f}")
print(f"بهبود نسبت به پیش‌فرض: {rs.best_score_ - 0.7:.3f}")
```

**مثال ۳ — Optuna:**

```python
try:
    import optuna
    optuna.logging.set_verbosity(optuna.logging.WARNING)

    from sklearn.ensemble import GradientBoostingClassifier
    from sklearn.model_selection import cross_val_score

    def objective(trial):
        params = {
            "n_estimators": trial.suggest_int("n_estimators", 50, 500),
            "learning_rate": trial.suggest_float("learning_rate", 1e-3, 0.3, log=True),
            "max_depth": trial.suggest_int("max_depth", 2, 10),
            "subsample": trial.suggest_float("subsample", 0.6, 1.0),
            "min_samples_leaf": trial.suggest_int("min_samples_leaf", 1, 20),
        }
        model = GradientBoostingClassifier(random_state=42, **params)
        return cross_val_score(model, X_tr, y_tr, cv=3, scoring="f1", n_jobs=-1).mean()

    study = optuna.create_study(direction="maximize")
    study.optimize(objective, n_trials=30, show_progress_bar=False)

    print(f"بهترین F1: {study.best_value:.3f}")
    print("بهترین پارامترها:", study.best_params)
    print(f"تعداد trial: {len(study.trials)}")
except ImportError:
    print("نصب کن: pip install optuna")
```

**Optuna با ۳۰ trial، معمولاً بهتر از Grid با ۱۰۰ ترکیب عمل می‌کند.**

**مثال ۴ — تنظیم مرحله‌ای (Coarse-to-Fine):**

```python
# مرحله ۱: جست‌وجوی درشت
coarse = {
    "n_estimators": [100, 300, 600],
    "max_depth": [5, 15, None],
}
gs1 = GridSearchCV(RandomForestClassifier(random_state=42),
                   coarse, cv=3, scoring="f1", n_jobs=-1).fit(X_tr, y_tr)
print("مرحله ۱:", gs1.best_params_, round(gs1.best_score_, 3))

# مرحله ۲: جست‌وجوی دقیق حول بهترین
best_depth = gs1.best_params_["max_depth"]
fine = {
    "max_depth": [best_depth] if best_depth is None else [best_depth - 2, best_depth, best_depth + 2],
    "min_samples_leaf": [1, 2, 4],
    "max_features": [0.3, 0.5, 0.8],
}
gs2 = GridSearchCV(RandomForestClassifier(
    n_estimators=gs1.best_params_["n_estimators"], random_state=42),
    fine, cv=5, scoring="f1", n_jobs=-1).fit(X_tr, y_tr)
print("مرحله ۲:", gs2.best_params_, round(gs2.best_score_, 3))
```

**این استراتژی، در عمل بسیار مؤثر است:** اول درشت، بعد دقیق.

**مثال ۵ — جدول خلاصه‌ی نتایج:**

```python
import pandas as pd

results = pd.DataFrame(gs.cv_results_)
cols = ["param_n_estimators", "param_max_depth", "param_min_samples_leaf",
        "mean_test_score", "std_test_score", "rank_test_score"]
top = results[cols].sort_values("rank_test_score").head(8)
print(top.to_string(index=False))
```

### مقایسه و دیدگاه‌های مختلف

| روش | تعداد ارزیابی لازم | کیفیت | پیچیدگی |
|---|---|---|---|
| Manual | ۵–۱۰ | پایین | صفر |
| Grid | ۵۰–۵۰۰ | خوب | کم |
| Random | ۳۰–۱۰۰ | خوب | کم |
| Bayesian | ۲۰–۵۰ | عالی | متوسط |
| Hyperband | ۲۰–۵۰ | عالی (با pruning) | متوسط |

**دو دیدگاه:**
- **«Grid برای همه‌چیز کافی است.»** برای فضای کوچک (زیر ۱۰۰ ترکیب)، بله.
- **«Bayesian همیشه بهتر است.»** برای فضای بزرگ، بله. اما برای ۲ پارامتر، Grid ساده‌تر است.

**توصیه:** Manual → Random → Optuna. Grid فقط برای فضای کوچک.

### اشتباهات رایج

**۱. Grid روی فضای بزرگ.** `10×10×10×10 = 10,000` آموزش.
**۲. `linspace` به جای `logspace`** برای `alpha` و `lr`.
**۳. تنظیم روی test.** نشت.
**۴. استفاده از یک تقسیم به جای CV.** نتیجه‌ی شانسی.
**۵. تنظیم همه‌ی پارامترها هم‌زمان.** مرحله‌ای بهتر است.
**۶. نادیده گرفتن زمان آموزش.** مدلی با F1 بهتر اما ۱۰ برابر کندتر، ممکن است ارزش نداشته باشد.
**۷. انتخاب بهترین پارامتر بدون بررسی پایداری.** اگر دو پارامتر نزدیک، نتایج خیلی متفاوت دارند، مدل ناپایدار است.

### نکات مهم و ترفندها

- **`logspace` برای پارامترهای مقیاسی.** `linspace` برای پارامترهای شمارشی.
- **مرحله‌ای تنظیم کن:** اول مهم‌ترین‌ها، بعد جزئیات.
- **`n_jobs=-1` + `verbose=1`** ترکیب خوبی است.
- **`Optuna` با `pruning`** می‌تواند ۵۰٪ وقت صرفه‌جویی کند.
- **همیشه `std_test_score` را ببین.** اگر زیاد است، نتیجه ناپایدار.
- **میان‌بر:** Random برای شروع، Optuna برای جدی، Grid برای کوچک.

### خلاصه فصل

- فراپارامتر، تنظیمات قبل از آموزش است.
- Random Search از Grid بهتر است در فضای بزرگ.
- مقیاس لگاریتمی برای پارامترهای مثبت نامحدود.
- Optuna/Bayesian، جست‌وجوی هوشمند انجام می‌دهد.
- تنظیم مرحله‌ای (درشت → دقیق) در عمل مؤثرترین است.

**سؤالات مروری:**
1. چرا Random Search از Grid بهتر است؟
2. چه زمانی `logspace` لازم است؟
3. چرا تنظیم فراپارامتر روی test اشتباه است؟

---

## فصل ۷۰: مثلث تنظیم: epoch، batch size، learning rate

### مقدمه

سه فراپارامتر وجود دارند که **به هم وابسته‌اند** و انتخاب اشتباه هر کدام، آموزش را خراب می‌کند:

- **learning rate:** اندازه‌ی قدم
- **batch size:** تعداد نمونه در هر قدم
- **epoch:** تعداد بار عبور از داده

این فصل، رابطه‌ی بین این سه و راه‌های تنظیمشان را می‌دهد.

### مفاهیم اساسی

**۱. رابطه‌ی ریاضی:**

```
iterations_per_epoch = n_samples / batch_size
total_updates = iterations_per_epoch × epochs
```

مثال: ۱۰٬۰۰۰ نمونه، batch=100 → ۱۰۰ به‌روزرسانی در epoch. با ۵۰ epoch → ۵٬۰۰۰ به‌روزرسانی.

**۲. اثر batch size:**

| batch کوچک | batch بزرگ |
|---|---|
| نویز بیشتر در گرادیان | گرادیان دقیق‌تر |
| از کمینه‌ی محلی فرار می‌کند | پایدارتر |
| کندتر (بیشتر به‌روزرسانی) | سریع‌تر (موازی‌سازی بهتر) |
| generalization بهتر (اغلب) | generalization کمی ضعیف‌تر |
| حافظه‌ی کمتر | حافظه‌ی بیشتر |

**۳. قاعده‌ی مقیاس خطی (Linear Scaling Rule).**

اگر batch را ۲ برابر کردی، **lr را هم ۲ برابر کن**. دلیل: با batch بزرگ‌تر، گرادیان میانگین‌گیری بیشتری دارد و نویز کمتری دارد، پس می‌توانی قدم بزرگ‌تری برداری.

```
lr_new = lr_base × (batch_new / batch_base)
```

**۴. Warmup — چرا لازم است؟**

در ابتدای آموزش، مدل تصادفی است و گرادیان‌ها پرنویز. lr بزرگ، آموزش را منفجر می‌کند. Warmup از lr صفر شروع می‌کند و به‌تدریج بالا می‌برد.

**۵. Early stopping در برابر epoch ثابت.**

- **epoch ثابت:** ساده، اما ممکن است underfit یا overfit.
- **early stopping:** تطبیقی، بهتر. patience=۱۰ تا ۲۰.

**۶. راهنمای انتخاب:**

| حجم داده | batch size | lr | epoch |
|---|---|---|---|
| کوچک (<۱۰۰۰) | ۱۶–۳۲ | 1e-3 | ۱۰۰–۵۰۰ |
| متوسط | ۳۲–۱۲۸ | 1e-3 | ۵۰–۲۰۰ |
| بزرگ (>۱۰۰k) | ۲۵۶–۱۰۲۴ | 1e-2 (با warmup) | ۱۰–۵۰ |
| ترانسفورمر | ۳۲–۵۱۲ | 1e-4 تا 5e-4 (با warmup) | ۳–۱۰ |

**۷. ترفند: Gradient Accumulation.**
اگر حافظه برای batch بزرگ کافی نیست، گرادیان چند batch کوچک را جمع کن و یک بار به‌روزرسانی کن. **اثر batch بزرگ، با حافظه‌ی کم.**

```python
accumulation_steps = 4
for i, batch in enumerate(loader):
    loss = model(batch) / accumulation_steps
    loss.backward()
    if (i + 1) % accumulation_steps == 0:
        optimizer.step()
        optimizer.zero_grad()
```

**۸. نشانه‌های انتخاب اشتباه:**

| نشانه | مشکل |
|---|---|
| loss می‌پرد | lr بزرگ |
| loss خیلی کند کم می‌شود | lr کوچک یا batch بزرگ |
| loss نویزی و ناپایدار | batch کوچک |
| Out of Memory | batch بزرگ |
| overfit سریع | epoch زیاد |
| underfit بعد از آموزش | epoch کم |

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — محاسبه‌ی به‌روزرسانی‌ها:**

```python
def training_stats(n_samples, batch_size, epochs):
    steps_per_epoch = int(n_samples / batch_size)
    total = steps_per_epoch * epochs
    return {
        "steps/epoch": steps_per_epoch,
        "total_updates": total,
        "samples_seen": n_samples * epochs,
    }

for bs in [16, 32, 128, 512]:
    s = training_stats(10_000, bs, 20)
    print(f"batch={bs:>4} → {s['steps/epoch']:>4} گام/epoch | "
          f"{s['total_updates']:>6} به‌روزرسانی کل")
```

**نکته:** با batch=16، ۱۲ برابر بیشتر از batch=512 به‌روزرسانی داری — یعنی آموزش **بسیار** کندتر (اگرچه هر قدم ارزان‌تر).

**مثال ۲ — اثر lr و batch با هم:**

```python
import numpy as np

def simulate_training(lr, batch_size, n=2000, epochs=20, seed=0):
    """شبیه‌سازی ساده: اثر lr و batch بر همگرایی."""
    rng = np.random.default_rng(seed)
    w = 0.0
    steps_per_epoch = max(1, n // batch_size)
    losses = []

    for epoch in range(epochs):
        for _ in range(steps_per_epoch):
            # گرادیان با نویز متناسب با 1/sqrt(batch)
            noise = rng.normal(0, 1 / np.sqrt(batch_size))
            grad = 2 * (w - 3) + noise
            w -= lr * grad
            if not np.isfinite(w) or abs(w) > 1e6:
                return "واگرا", losses
        losses.append((w - 3) ** 2)

    return ("همگرا" if abs(w - 3) < 0.5 else "نزدیک نشد"), losses

print(f"{'lr':>8} {'batch':>7} {'نتیجه':>12} {'loss نهایی':>12}")
for lr, bs in [(0.001, 32), (0.01, 32), (0.1, 32),
               (0.01, 128), (0.04, 128), (0.1, 128), (0.5, 128)]:
    status, losses = simulate_training(lr, bs)
    final = f"{losses[-1]:.4f}" if losses else "-"
    print(f"{lr:>8} {bs:>7} {status:>12} {final:>12}")
```

**خروجی معمول:** lr=0.1 با batch=32 واگرا؛ با lr کوچک‌تر و batch بزرگ‌تر، پایدارتر.

**مثال ۳ — Warmup + Cosine در PyTorch:**

```python
try:
    import torch
    import torch.nn as nn
    import math

    def get_lr(step, warmup_steps, total_steps, base_lr, min_lr=1e-6):
        if step < warmup_steps:
            return base_lr * (step + 1) / warmup_steps
        progress = (step - warmup_steps) / max(1, total_steps - warmup_steps)
        return min_lr + 0.5 * (base_lr - min_lr) * (1 + math.cos(math.pi * progress))

    # نمایش
    print(f"{'step':>6} {'lr':>12}")
    for step in [0, 10, 50, 100, 500, 1000]:
        print(f"{step:>6} {get_lr(step, 100, 1000, 1e-3):>12.7f}")

    # در حلقه‌ی آموزش:
    # for step in range(total_steps):
    #     for g in optimizer.param_groups:
    #         g["lr"] = get_lr(step, 100, total_steps, 1e-3)
except ImportError:
    print("pip install torch")
```

**مثال ۴ — Gradient Accumulation:**

```python
def train_with_accumulation(model, loader, optimizer, criterion,
                            accumulation_steps=4, device="cpu"):
    """batch بزرگ مجازی با حافظه‌ی کم."""
    model.train()
    optimizer.zero_grad()
    running_loss = 0.0

    for i, (xb, yb) in enumerate(loader):
        xb, yb = xb.to(device), yb.to(device)
        loss = criterion(model(xb), yb) / accumulation_steps
        loss.backward()
        running_loss += loss.item()

        if (i + 1) % accumulation_steps == 0:
            optimizer.step()
            optimizer.zero_grad()

    return running_loss / max(1, len(loader))
```

**مثال ۵ — جست‌وجوی lr (LR Range Test):**

```python
def lr_range_test(model, loader, criterion, optimizer, start_lr=1e-5,
                  end_lr=1.0, steps=100):
    """lr را به‌تدریج زیاد کن و loss را ثبت کن — نقطه‌ی شیرین را پیدا کن."""
    mult = (end_lr / start_lr) ** (1 / steps)
    lr = start_lr
    history = []

    for step, (xb, yb) in enumerate(loader):
        if step >= steps:
            break
        for g in optimizer.param_groups:
            g["lr"] = lr

        optimizer.zero_grad()
        loss = criterion(model(xb), yb)
        loss.backward()
        optimizer.step()

        history.append((lr, loss.item()))
        lr *= mult

    return history

# بهترین lr ≈ lr در نقطه‌ای که loss کمترین است، تقسیم بر ۱۰
```

**این تکنیک (Leslie Smith)، سریع‌ترین راه پیدا کردن lr مناسب است.**

### مقایسه و دیدگاه‌های مختلف

| رویکرد | مزیت | عیب |
|---|---|---|
| epoch ثابت | ساده، قابل پیش‌بینی | overfit/underfit |
| Early stopping | تطبیقی | نیاز به validation |
| LR range test | سریع | نیاز به اجرای جداگانه |
| One-cycle | سریع‌ترین همگرایی | نیاز به تنظیم |

**یک دیدگاه:** «batch size بزرگ‌تر همیشه بهتر است.» نه: generalization کمی بدتر می‌شود و حافظه زیاد می‌خواهد. **batch متوسط (۳۲–۱۲۸) معمولاً بهترین تعادل است.**

### اشتباهات رایج

**۱. تغییر lr بدون تغییر batch.** قاعده‌ی مقیاس خطی را نقض می‌کند.
**۲. batch=1.** آموزش بسیار کند و ناپایدار.
**۳. batch بزرگ بدون warmup.** واگرایی.
**۴. epoch ثابت برای همه‌ی مدل‌ها.** باید با early stopping تنظیم شود.
**۵. فراموش کردن `optimizer.zero_grad()` در accumulation.** گرادیان‌ها انباشته می‌شوند.
**۶. اندازه‌گیری زمان بدون در نظر گرفتن batch.** batch کوچک، «سریع‌تر» به‌نظر می‌رسد اما به‌ازای هر نمونه کندتر است.

### نکات مهم و ترفندها

- **از batch=32 شروع کن.** نقطه‌ی امن.
- **lr=1e-3 با Adam** نقطه‌ی شروع خوبی است.
- **LR range test** را یک بار امتحان کن — سریع‌ترین راه.
- **`gradient_accumulation`** وقتی حافظه کم است.
- **همیشه early stopping.** epoch ثابت را فقط برای مقایسه استفاده کن.
- **میان‌بر:** batch ↑ → lr ↑ · epoch ↑ → overfit · warmup برای شروع پایدار.

### خلاصه فصل

- epoch، batch و lr به هم وابسته‌اند.
- قاعده‌ی مقیاس خطی: batch دو برابر → lr دو برابر.
- batch کوچک generalization بهتر، batch بزرگ سریع‌تر.
- Warmup آموزش مدل‌های بزرگ را ممکن می‌کند.
- Gradient accumulation، batch بزرگ با حافظه‌ی کم.

**سؤالات مروری:**
1. اگر batch را از ۳۲ به ۱۲۸ ببری، lr را چطور تغییر می‌دهی؟
2. چرا warmup لازم است؟
3. Gradient accumulation چه مشکلی را حل می‌کند؟

---

## منابع و مراجع

**کتاب‌ها**
- Goodfellow, I. et al. — *Deep Learning*، فصل ۷ (Regularization)
- Géron, A. — *Hands-On ML*، فصل ۴ و ۱۱
- Hastie, T. et al. — *The Elements of Statistical Learning*، فصل ۳ و ۷

**مقالات کلیدی**
- Srivastava, N. et al. (2014) — *Dropout: A Simple Way to Prevent Neural Networks from Overfitting*
- Bergstra, J. & Bengio, Y. (2012) — *Random Search for Hyper-Parameter Optimization*
- Loshchilov, I. & Hutter, F. (2016) — *SGDR: Stochastic Gradient Descent with Warm Restarts*
- Smith, L. (2018) — *A Disciplined Approach to Neural Network Hyper-Parameters* (LR range test)
- Goyal, P. et al. (2017) — *Accurate, Large Minibatch SGD* (linear scaling rule)

**ابزارها**
- Optuna — `optuna.visualization` برای نمودارهای مهم‌بودن پارامترها
- Weights & Biases / TensorBoard — ردیابی آزمایش‌ها

---

**بعدی:** [`16-Neural-Networks.md`](./16-Neural-Networks.md) — فصل ۷۱: نورون مصنوعی
