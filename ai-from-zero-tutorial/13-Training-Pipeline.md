# بخش ۱۳: فرآیند آموزش مدل — قلب یادگیری ماشین

**مدت یادگیری تقریبی**: ۳ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۴ (مشتق و گرادیان)، فایل ۰۵ (NumPy)، فایل ۰۸ (مبانی ML)

---

## فهرست محتوا

- [فصل ۵۵: چرخه‌ی کامل آموزش: نمای کلی](#فصل-۵۵-چرخهی-کامل-آموزش-نمای-کلی)
- [فصل ۵۶: آماده‌سازی داده و مهندسی ویژگی](#فصل-۵۶-آمادهسازی-داده-و-مهندسی-ویژگی)
- [فصل ۵۷: Forward Pass: مدل چگونه پیش‌بینی می‌کند](#فصل-۵۷-forward-pass-مدل-چگونه-پیشبینی-میکند)
- [فصل ۵۸: Loss Function: مدل چگونه اشتباهش را می‌فهمد](#فصل-۵۸-loss-function-مدل-چگونه-اشتباهش-را-میفهمد)
- [فصل ۵۹: Backpropagation: زنجیره‌ی مشتق به زبان ساده](#فصل-۵۹-backpropagation-زنجیرهی-مشتق-به-زبان-ساده)
- [فصل ۶۰: Gradient Descent و learning rate](#فصل-۶۰-gradient-descent-و-learning-rate)

---

## فصل ۵۵: چرخه‌ی کامل آموزش: نمای کلی

### مقدمه

این فایل، **قلب کتاب** است. تا اینجا یاد گرفتیم داده چیست، چه مدل‌هایی وجود دارد و چطور ارزیابی کنیم. حالا می‌بینیم **داخل جعبه چه خبر است**: وقتی `model.fit(X, y)` را صدا می‌زنی، دقیقاً چه اتفاقی می‌افتد؟

جواب، یک چرخه‌ی پنج‌مرحله‌ای است که هزاران بار تکرار می‌شود:

```
پیش‌بینی → محاسبه‌ی خطا → محاسبه‌ی گرادیان → اصلاح وزن‌ها → تکرار
```

اگر این چرخه را بفهمی، هر خطای آموزشی را می‌توانی دیباگ کنی.

### مفاهیم اساسی

**۱. نمای کلی چرخه:**

```
┌─────────────────────────────────────────────────┐
│  ۱. آماده‌سازی داده (یک بار)                     │
│     پاکسازی → ویژگی → تقسیم → مقیاس‌بندی        │
└────────────────────┬────────────────────────────┘
                     ▼
        ┌────────────────────────────┐
        │  برای هر epoch:             │
        │  ┌──────────────────────┐  │
        │  │ برای هر batch:        │  │
        │  │  ۱. Forward pass      │  │
        │  │  ۲. محاسبه‌ی loss      │  │
        │  │  ۳. Backpropagation   │  │
        │  │  ۴. به‌روزرسانی وزن   │  │
        │  └──────────────────────┘  │
        │  ارزیابی روی validation     │
        └────────────────────────────┘
                     ▼
               ذخیره‌ی بهترین مدل
```

**۲. چهار جزء که باید بشناسی:**

| جزء | نقش | مثال |
|---|---|---|
| **Forward pass** | محاسبه‌ی پیش‌بینی | `y_hat = W₂·relu(W₁·x + b₁) + b₂` |
| **Loss** | اندازه‌گیری بدی | MSE، cross-entropy |
| **Backward pass** | محاسبه‌ی گرادیان | قاعده‌ی زنجیره |
| **Optimizer** | اصلاح وزن‌ها | SGD، Adam |

**۳. چه چیزی «یاد گرفته می‌شود»؟** فقط **پارامترها** (وزن‌ها و بایاس‌ها). ساختار مدل، تابع فعال‌سازی، معماری — این‌ها **فراپارامتر** هستند و توسط تو انتخاب می‌شوند.

**۴. تفاوت آموزش «دسته‌ای» و «آنلاین»:**

| | Batch (آفلاین) | Online (جریانی) |
|---|---|---|
| داده | همه از قبل موجود | تدریجی می‌رسد |
| آموزش | یک بار (یا دوره‌ای) | مداوم |
| مثال | پیش‌بینی قیمت خانه | فیلتر اسپم |
| چالش | drift | مفهوم drift شدیدتر |

**۵. سه منبع اصلی شکست آموزش:**

1. **مشکل داده:** نویز، نشت، عدم توازن، حجم کم.
2. **مشکل مدل:** ظرفیت کم (underfit) یا زیاد (overfit).
3. **مشکل آموزش:** lr بد، gradient ناپایدار، باگ در پیاده‌سازی.

**تشخیص:** همیشه اول به loss نگاه کن. شکل منحنی loss، به تو می‌گوید کدام دسته است.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — چرخه‌ی کامل، دستی و بدون کتابخانه:**

```python
import numpy as np

# ---------- ۱. داده ----------
rng = np.random.default_rng(42)
n, d = 300, 3
X = rng.normal(size=(n, d))
true_w = np.array([2.0, -1.0, 0.5])
y = X @ true_w + 1.5 + rng.normal(scale=0.3, size=n)

# ---------- ۲. آماده‌سازی ----------
X_mean, X_std = X.mean(axis=0), X.std(axis=0)
X_norm = (X - X_mean) / X_std

split = int(0.8 * n)
X_tr, X_val = X_norm[:split], X_norm[split:]
y_tr, y_val = y[:split], y[split:]

# ---------- ۳. پارامترها ----------
w = np.zeros(d)
b = 0.0
lr = 0.05
history = {"train": [], "val": []}

# ---------- ۴. چرخه‌ی آموزش ----------
for epoch in range(200):
    # ---- forward ----
    y_hat = X_tr @ w + b

    # ---- loss ----
    error = y_hat - y_tr
    loss = np.mean(error ** 2)

    # ---- backward ----
    dw = (2 / len(X_tr)) * (X_tr.T @ error)
    db = (2 / len(X_tr)) * error.sum()

    # ---- update ----
    w -= lr * dw
    b -= lr * db

    # ---- ارزیابی ----
    if epoch % 20 == 0:
        val_pred = X_val @ w + b
        val_loss = np.mean((val_pred - y_val) ** 2)
        history["train"].append(loss)
        history["val"].append(val_loss)
        print(f"epoch {epoch:3d} | train={loss:7.4f} | val={val_loss:7.4f}")

print(f"\nw نهایی: {w.round(3)} (واقعی {true_w})")
print(f"b نهایی: {b:.3f} (واقعی 1.5)")
```

**این ۳۰ خط، هسته‌ی هر آموزش مدلی است.** هر چیزی که بعداً می‌بینی (PyTorch، TensorFlow) همین است، با اتوماسیون و بهینه‌سازی.

**مثال ۲ — همان چرخه با sklearn:**

```python
from sklearn.linear_model import SGDRegressor
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import make_pipeline

model = make_pipeline(
    StandardScaler(),
    SGDRegressor(loss="squared_error", learning_rate="constant",
                 eta0=0.01, max_iter=1000, random_state=42)
)
model.fit(X, y)
print("R²:", round(model.score(X, y), 4))
```

**نکته:** `SGDRegressor` دقیقاً همان چیزی است که دستی نوشتیم، اما بهینه‌شده و با گزینه‌های بیشتر.

### مقایسه و دیدگاه‌های مختلف

| سطح انتزاع | ابزار | کنترل | سرعت توسعه |
|---|---|---|---|
| خیلی پایین | NumPy دستی | کامل | کند |
| متوسط | sklearn | متوسط | سریع |
| بالا | PyTorch/TF | زیاد (با autograd) | متوسط |
| خیلی بالا | Keras / AutoML | کم | خیلی سریع |

**توصیه:** یک بار دستی بنویس (مثل مثال ۱)، بعد از کتابخانه استفاده کن. **دانستن این‌که پشت صحنه چه خبر است، تفاوت دیباگر و کاربر است.**

### اشتباهات رایج

**۱. شروع با معماری پیچیده.** اول ساده‌ترین مدل، بعد پیچیده‌تر.
**۲. نگاه نکردن به loss.** بدون منحنی loss، کوری.
**۳. ارزیابی نکردن روی validation در طول آموزش.** نمی‌دانی کِی متوقف کنی.
**۴. ذخیره نکردن مدل.** آموزش دو ساعته از دست می‌رود.
**۵. نبود بازتولیدپذیری.** بدون seed، نمی‌توانی دیباگ کنی.
**۶. آموزش روی داده‌ی تست.** نشت داده.

### نکات مهم و ترفندها

- **همیشه منحنی loss را رسم کن.** یک نگاه، بیشتر از ۱۰۰ عدد می‌گوید.
- **`random_state` / `seed` را از ابتدا بگذار.** بعداً اضافه کردنش، نتایج قبلی را بی‌اعتبار می‌کند.
- **مدل را با `checkpoint` ذخیره کن.** بهترین مدل روی validation را نگه دار، نه آخرین را.
- **میان‌بر:** forward → loss → backward → update → تکرار.

### خلاصه فصل

- آموزش، چرخه‌ی پنج‌مرحله‌ای است که هزاران بار تکرار می‌شود.
- چهار جزء: forward، loss، backward، optimizer.
- فقط پارامترها یاد گرفته می‌شوند؛ ساختار را تو انتخاب می‌کنی.
- سه منبع شکست: داده، مدل، آموزش.
- منحنی loss، بهترین ابزار تشخیص.

**سؤالات مروری:**
1. چهار جزء چرخه‌ی آموزش را نام ببر.
2. تفاوت پارامتر و فراپارامتر را در این چرخه نشان بده.
3. اگر loss کاهش نیابد، سه احتمال اصلی چیست؟

---

## فصل ۵۶: آماده‌سازی داده و مهندسی ویژگی

### مقدمه

در فایل‌های ۰۲ و ۰۶ درباره‌ی پاکسازی صحبت کردیم. این فصل، همان را از **دیدگاه آموزش مدل** می‌بیند: چه چیزی داده را «آماده‌ی آموزش» می‌کند و چه چیزی بی‌صدا مدل را خراب می‌کند.

قاعده‌ی کلی: **هر تبدیلی که پارامتری از داده یاد می‌گیرد، باید فقط روی train فیت شود.**

### مفاهیم اساسی

**۱. ترتیب صحیح (این ترتیب را حفظ کن):**

```
۱. حذف تکراری‌ها (روی کل داده — بی‌خطر)
۲. تقسیم train / val / test
۳. fit پیش‌پردازش روی train
۴. transform روی val و test
```

**۲. سه تبدیل حساس به نشت:**

| تبدیل | چه یاد می‌گیرد | نشت اگر... |
|---|---|---|
| `StandardScaler` | mean، std | روی کل داده fit کنی |
| `SimpleImputer` | median، mode | روی کل داده fit کنی |
| `TargetEncoder` | میانگین هدف | روی کل داده حساب کنی |

**۳. مقیاس‌بندی — کدام و کِی:**

| روش | فرمول | کِی |
|---|---|---|
| `StandardScaler` | `(x - μ) / σ` | پیش‌فرض، مدل‌های خطی و شبکه |
| `MinMaxScaler` | `(x - min) / (max - min)` | بازه‌ی مشخص لازم است |
| `RobustScaler` | `(x - median) / IQR` | داده‌ی پرت |
| `Normalizer` | نرمال‌سازی هر ردیف | متن (TF-IDF)، شباهت |

**۴. کدگذاری دسته‌ای:**

| روش | مناسب برای | خطر |
|---|---|---|
| One-Hot | اسمی، کاردینالیتی کم | انفجار ابعاد |
| Ordinal | ترتیبی | ترتیب جعلی برای اسمی |
| Target Encoding | کاردینالیتی بالا | نشت |
| Frequency | کاردینالیتی بالا | از دست دادن معنا |
| Embedding | کاردینالیتی خیلی بالا | نیاز به داده‌ی زیاد |

**۵. مهندسی ویژگی — پنج تکنیک پرثمر:**

1. **نسبت‌ها:** `درآمد / هزینه`، `کلیک / نمایش`
2. **تفاوت‌های زمانی:** `امروز - آخرین خرید`
3. **تجمیع گروهی:** `میانگین خرید مشتری`، `تعداد تراکنش در ۳۰ روز`
4. **تعامل:** `سن × درآمد`، `متراژ / اتاق`
5. **بینه‌سازی (Binning):** `سن → [۱۸-۲۵, ۲۶-۳۵, ...]`

**۶. مدیریت داده‌ی نامتوازن — فقط روی train:**

```python
# درست
X_train_res, y_train_res = smote.fit_resample(X_train, y_train)
# غلط: اعمال روی کل داده قبل از تقسیم
```

**۷. داده‌ی گمشده — کدام استراتژی:**

| درصد گمشدگی | استراتژی |
|---|---|
| <۵٪ | حذف ردیف یا میانه |
| ۵–۳۰٪ | imputation + نشانگر |
| ۳۰–۶۰٪ | imputation مدل‌محور یا حذف ستون |
| >۶۰٪ | حذف ستون (مگر سیگنال باشد) |

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — Pipeline کامل با ColumnTransformer:**

```python
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer
from sklearn.linear_model import LogisticRegression

rng = np.random.default_rng(42)
n = 800
df = pd.DataFrame({
    "age": rng.integers(18, 70, n).astype(float),
    "income": rng.lognormal(10, 0.7, n),
    "city": rng.choice(["تهران", "شیراز", "مشهد"], n),
    "plan": rng.choice(["basic", "pro", "enterprise"], n),
    "churn": rng.integers(0, 2, n),
})
# ایجاد گمشدگی مصنوعی
df.loc[rng.choice(n, 60, replace=False), "income"] = np.nan
df.loc[rng.choice(n, 30, replace=False), "city"] = np.nan

X = df.drop(columns="churn")
y = df["churn"]

num_cols = ["age", "income"]
cat_cols = ["city", "plan"]

numeric_pipe = Pipeline([
    ("impute", SimpleImputer(strategy="median")),
    ("scale", StandardScaler()),
])
categorical_pipe = Pipeline([
    ("impute", SimpleImputer(strategy="constant", fill_value="نامشخص")),
    ("onehot", OneHotEncoder(handle_unknown="ignore", sparse_output=False)),
])

preprocessor = ColumnTransformer([
    ("num", numeric_pipe, num_cols),
    ("cat", categorical_pipe, cat_cols),
])

full_pipeline = Pipeline([
    ("prep", preprocessor),
    ("clf", LogisticRegression(max_iter=1000, random_state=42)),
])

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42
)
full_pipeline.fit(X_train, y_train)
print("دقت:", round(full_pipeline.score(X_test, y_test), 3))

# نام ستون‌های خروجی
feature_names = full_pipeline.named_steps["prep"].get_feature_names_out()
print("تعداد ویژگی پس از پیش‌پردازش:", len(feature_names))
```

**این الگو، استاندارد صنعتی است.** یک شیء `Pipeline`، همه‌ی مراحل را در بر می‌گیرد و نشت را غیرممکن می‌کند.

**مثال ۲ — نشان دادن نشت داده:**

```python
from sklearn.preprocessing import StandardScaler

# ---------- اشتباه ----------
scaler_wrong = StandardScaler().fit(X[["age"]])          # روی کل داده!
X_tr_wrong = scaler_wrong.transform(X_train[["age"]])

# ---------- درست ----------
scaler_right = StandardScaler().fit(X_train[["age"]])    # فقط train
X_tr_right = scaler_right.transform(X_train[["age"]])

print("mean اشتباه:", round(scaler_wrong.mean_[0], 3))
print("mean درست:  ", round(scaler_right.mean_[0], 3))
print("تفاوت کوچک است، اما با داده‌ی کم و تقسیم نامتوازن، بزرگ می‌شود.")
```

**مثال ۳ — مهندسی ویژگی گروهی:**

```python
df2 = df.copy()
df2["income"] = df2["income"].fillna(df2["income"].median())

# ویژگی‌های گروهی
df2["city_avg_income"] = df2.groupby("city")["income"].transform("mean")
df2["income_vs_city"] = df2["income"] / df2["city_avg_income"]
df2["age_bin"] = pd.cut(df2["age"], bins=[17, 25, 35, 50, 70],
                        labels=["۱۸-۲۵", "۲۶-۳۵", "۳۶-۵۰", "۵۱-۷۰"])

print(df2[["age", "age_bin", "income", "city_avg_income", "income_vs_city"]].head())
```

**مثال ۴ — Target Encoding با cross-fitting (بدون نشت):**

```python
from sklearn.model_selection import KFold

def target_encode_oof(train_df, test_df, col, target, n_splits=5, smoothing=10):
    """Target encoding با out-of-fold — بدون نشت."""
    global_mean = train_df[target].mean()

    # برای train: out-of-fold
    encoded_train = np.zeros(len(train_df))
    kf = KFold(n_splits=n_splits, shuffle=True, random_state=42)
    for tr_idx, val_idx in kf.split(train_df):
        stats = train_df.iloc[tr_idx].groupby(col)[target].agg(["mean", "count"])
        smooth = (stats["mean"] * stats["count"] + global_mean * smoothing) / (stats["count"] + smoothing)
        encoded_train[val_idx] = train_df.iloc[val_idx][col].map(smooth).fillna(global_mean)

    # برای test: روی کل train
    stats = train_df.groupby(col)[target].agg(["mean", "count"])
    smooth = (stats["mean"] * stats["count"] + global_mean * smoothing) / (stats["count"] + smoothing)
    encoded_test = test_df[col].map(smooth).fillna(global_mean)

    return encoded_train, encoded_test
```

### مقایسه و دیدگاه‌های مختلف

| رویکرد | مزیت | عیب |
|---|---|---|
| Pipeline (sklearn) | ساختاری، بدون نشت | کمتر انعطاف‌پذیر |
| دستی و مرحله‌به‌مرحله | کنترل کامل | مستعد نشت |
| Feature Store | بازاستفاده، یکسان‌سازی | زیرساخت سنگین |

**یک دیدگاه:** «مهندسی ویژگی، هنر است، نه علم.» تا حدی درست — اما قاعده‌ی «زمان تصمیم» و «Pipeline» بخش علمی و قابل اتکای آن است.

### اشتباهات رایج

**۱. `fit` روی کل داده.** نشت.
**۲. `get_dummies` قبل از تقسیم.** ستون‌های متفاوت در train/test → خطا یا نشت.
**۳. One-Hot روی ستون با ۵۰۰۰ مقدار یکتا.** انفجار ابعاد. از Target/Frequency Encoding استفاده کن.
**۴. SMOTE قبل از تقسیم.** نشت شدید.
**۵. پر کردن گمشده با میانگین کل.** نشت + تخریب واریانس.
**۶. حذف ستون‌ها بر اساس همبستگی با هدف روی کل داده.** نشت انتخاب ویژگی.

### نکات مهم و ترفندها

- **`ColumnTransformer` + `Pipeline` = استاندارد.** یک بار یاد بگیری، همه‌جا استفاده می‌کنی.
- **`handle_unknown="ignore"` در OneHotEncoder واجب است.** وگرنه در test، دسته‌ی جدید خطا می‌دهد.
- **`SimpleImputer(strategy="median")` پیش‌فرض امن است.**
- **هر ویژگی جدید را با یک فرضیه بساز.** «فکر می‌کنم این مهم است چون...» — نه تصادفی.
- **میان‌بر:** تقسیم → fit روی train → transform روی همه → مدل.

### خلاصه فصل

- ترتیب: حذف تکراری → تقسیم → fit روی train → transform.
- هر تبدیل پارامتریک، مستعد نشت است.
- ColumnTransformer + Pipeline، راه استاندارد جلوگیری از نشت.
- مهندسی ویژگی: نسبت، زمان، تجمیع، تعامل، بینه‌سازی.
- Target Encoding باید out-of-fold باشد.

**سؤالات مروری:**
1. چرا `fit` روی کل داده نشت است؟
2. `handle_unknown="ignore"` چه مشکلی را حل می‌کند؟
3. سه ویژگی مشتق که از داده‌ی فروش می‌سازی را نام ببر.

---

## فصل ۵۷: Forward Pass: مدل چگونه پیش‌بینی می‌کند

### مقدمه

Forward pass، **مسیر داده از ورودی به خروجی** است. ساده به نظر می‌رسد: چند ضرب و جمع. اما همین مسیر، تمام «دانش» مدل را حمل می‌کند.

درک دقیق forward pass، برای سه کار لازم است: **دیباگ شکل‌ها**، **فهم معماری**، و **پیاده‌سازی backprop**.

### مفاهیم اساسی

**۱. مدل خطی — ساده‌ترین forward:**

```
z = w₁x₁ + w₂x₂ + ... + wₙxₙ + b = w·x + b
ŷ = z                      (رگرسیون)
ŷ = σ(z)                   (دسته‌بندی دودویی)
ŷ = softmax(z)             (دسته‌بندی چندکلاسه)
```

**۲. شبکه‌ی چندلایه — forward سلسله‌مراتبی:**

```
h₁ = activation(W₁·x + b₁)
h₂ = activation(W₂·h₁ + b₂)
ŷ  = W₃·h₂ + b₃
```

هر لایه، یک تبدیل خطی + یک غیرخطی.

**۳. شکل‌ها (shape) — مهم‌ترین چیزی که باید پیگیری کنی:**

```
X:  (batch_size, n_features)
W₁: (n_features, hidden_size)
b₁: (hidden_size,)
h₁: (batch_size, hidden_size)
W₂: (hidden_size, n_classes)
ŷ:  (batch_size, n_classes)
```

**قاعده:** `(m, n) @ (n, p) = (m, p)`. بعد داخلی باید یکی باشد.

**۴. چرا غیرخطی لازم است؟**

بدون تابع فعال‌سازی، ترکیب چند لایه‌ی خطی، **یک لایه‌ی خطی** است:

```
W₂(W₁x) = (W₂W₁)x = W'x
```

یعنی ۱۰۰ لایه‌ی خطی، معادل ۱ لایه. **غیرخطی بودن، تنها دلیلی است که «عمق» معنا دارد.**

**۵. توابع فعال‌سازی رایج:**

| تابع | فرمول | مزیت | عیب |
|---|---|---|---|
| **ReLU** | `max(0, x)` | سریع، بدون محوشدگی مثبت | مرگ نورون |
| **Leaky ReLU** | `max(0.01x, x)` | بدون مرگ | کمی کندتر |
| **Sigmoid** | `1/(1+e⁻ˣ)` | خروجی احتمال | محوشدگی گرادیان |
| **Tanh** | `(eˣ-e⁻ˣ)/(eˣ+e⁻ˣ)` | مرکز صفر | محوشدگی |
| **GELU** | نرم‌شده‌ی ReLU | استاندارد در ترانسفورمر | کندتر |
| **Softmax** | نرمال‌سازی نمایی | خروجی احتمال چندکلاسه | فقط لایه‌ی آخر |

**۶. Dropout — ترفند آموزش، نه استنتاج.**

در آموزش، تصادفی بعضی نورون‌ها را صفر می‌کند (مثلاً ۲۰٪). در استنتاج، همه فعال‌اند.

```python
# در PyTorch:
model.train()   # dropout فعال
model.eval()    # dropout خاموش
```

**۷. Batch Normalization.** نرمال‌سازی خروجی هر لایه در هر batch. آموزش را سریع‌تر و پایدارتر می‌کند.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — forward pass از صفر (یک لایه):**

```python
import numpy as np

def relu(x):
    return np.maximum(0, x)

def softmax(z):
    z = z - z.max(axis=1, keepdims=True)
    e = np.exp(z)
    return e / e.sum(axis=1, keepdims=True)

# داده: ۴ نمونه، ۳ ویژگی
X = np.array([
    [1.0, 2.0, -1.0],
    [0.5, -0.5, 1.0],
    [-1.0, 1.0, 0.0],
    [2.0, 0.0, -2.0],
])

# پارامترها
W1 = np.array([[0.5, -0.2, 0.3, 0.1],
               [0.1,  0.4, -0.3, 0.2],
               [-0.2, 0.1, 0.5, -0.4]])
b1 = np.array([0.1, 0.0, -0.1, 0.2])

W2 = np.array([[0.3, -0.5],
               [0.2,  0.4],
               [-0.1, 0.6],
               [0.5, -0.2]])
b2 = np.array([0.0, 0.1])

# forward
Z1 = X @ W1 + b1
H1 = relu(Z1)
Z2 = H1 @ W2 + b2
probs = softmax(Z2)

print("شکل X:", X.shape)
print("شکل Z1/H1:", Z1.shape)
print("شکل Z2:", Z2.shape)
print("احتمال‌ها:\n", probs.round(3))
print("جمع هر ردیف:", probs.sum(axis=1).round(6))
```

**مثال ۲ — forward شبکه‌ی عمیق‌تر با ذخیره‌ی مقادیر میانی (لازم برای backprop):**

```python
def forward(X, params):
    """forward + ذخیره‌ی cache برای backprop."""
    cache = {"X": X}
    h = X

    for i in range(len(params) // 2):
        W, b = params[f"W{i+1}"], params[f"b{i+1}"]
        z = h @ W + b
        cache[f"z{i+1}"] = z
        cache[f"h{i}"] = h
        h = relu(z) if i < len(params) // 2 - 1 else z   # آخرین لایه: بدون فعال‌سازی

    return h, cache

params = {
    "W1": np.random.randn(3, 5) * 0.5, "b1": np.zeros(5),
    "W2": np.random.randn(5, 4) * 0.5, "b2": np.zeros(4),
    "W3": np.random.randn(4, 2) * 0.5, "b3": np.zeros(2),
}

out, cache = forward(X, params)
print("خروجی:", out.shape)
print("کلیدهای cache:", list(cache.keys()))
```

**چرا cache؟** چون backprop برای محاسبه‌ی گرادیان، به مقادیر میانی (مثل `h` و `z`) نیاز دارد. ذخیره‌ی آن‌ها در forward، استاندارد است.

**مثال ۳ — اثر غیرخطی بودن:**

```python
rng = np.random.default_rng(0)
Xr = rng.normal(size=(100, 3))

# بدون غیرخطی: ۳ لایه = ۱ لایه
W1, W2, W3 = [rng.normal(size=(3, 3)) for _ in range(3)]
deep_linear = (Xr @ W1) @ W2 @ W3
single = Xr @ (W1 @ W2 @ W3)

print("خطای تفاوت:", np.abs(deep_linear - single).max())
# تقریباً صفر → ۳ لایه‌ی خطی = ۱ لایه
```

**این آزمایش را خودت اجرا کن.** دیدن عدد `~1e-15`، بهترین درس درباره‌ی ضرورت غیرخطی است.

**مثال ۴ — همان شبکه با PyTorch:**

```python
try:
    import torch
    import torch.nn as nn

    net = nn.Sequential(
        nn.Linear(3, 5), nn.ReLU(),
        nn.Linear(5, 4), nn.ReLU(),
        nn.Linear(4, 2),
    )
    x_t = torch.tensor(X, dtype=torch.float32)
    print(net(x_t).shape)
    print(net)
except ImportError:
    print("نصب کن: pip install torch")
```

### مقایسه و دیدگاه‌های مختلف

| فعال‌سازی | سرعت | محوشدگی گرادیان | کاربرد |
|---|---|---|---|
| ReLU | خیلی سریع | ندارد (برای x>0) | پیش‌فرض CNN/MLP |
| Leaky ReLU | سریع | ندارد | وقتی نورون می‌میرد |
| GELU | متوسط | ندارد | ترانسفورمر |
| Sigmoid | کند | شدید | لایه‌ی خروجی دودویی |
| Tanh | متوسط | شدید | RNN |

**یک دیدگاه:** «ReLU همه‌جا کافی است.» تقریباً درست برای MLP و CNN. در ترانسفورمر، GELU استاندارد است.

### اشتباهات رایج

**۱. نادرست بودن شکل‌ها.** رایج‌ترین خطا. `print(x.shape)` بعد از هر لایه.
**۲. فراموش کردن `+ b`.** بدون بایاس، مدل نمی‌تواند جابه‌جا شود.
**۳. فعال‌سازی در لایه‌ی خروجی رگرسیون.** خروجی باید خطی باشد.
**۴. sigmoid در لایه‌ی پنهان عمیق.** محوشدگی گرادیان. ReLU استفاده کن.
**۵. `model.train()` را در استنتاج فراموش کردن.** Dropout فعال می‌ماند و نتایج ناپایدار می‌شود.
**۶. عدم ذخیره‌ی cache در backprop دستی.** گرادیان قابل محاسبه نیست.

### نکات مهم و ترفندها

- **`print(x.shape)` را عادت کن.** در هر لایه، یک بار.
- **ReLU را پیش‌فرض بگیر**؛ اگر نورون مرد، Leaky ReLU.
- **`model.eval()` و `torch.no_grad()`** در استنتاج: سریع‌تر و درست‌تر.
- **BatchNorm بعد از لایه‌ی خطی و قبل از فعال‌سازی** (قرارداد رایج).
- **میان‌بر:** `h = act(W·x + b)` · بدون غیرخطی، عمق بی‌معنی است.

### خلاصه فصل

- forward pass، مسیر داده از ورودی به خروجی است.
- هر لایه = تبدیل خطی + غیرخطی.
- بدون غیرخطی، چند لایه معادل یک لایه است.
- ReLU پیش‌فرض است؛ GELU در ترانسفورمر.
- cache مقادیر میانی برای backprop لازم است.

**سؤالات مروری:**
1. چرا ترکیب چند لایه‌ی خطی معادل یک لایه است؟
2. چرا ReLU در لایه‌ی پنهان بهتر از sigmoid است؟
3. Dropout در آموزش و استنتاج چه تفاوتی دارد؟

---

## فصل ۵۸: Loss Function: مدل چگونه اشتباهش را می‌فهمد

### مقدمه

Loss، **تنها سیگنال یادگیری** است. مدل هیچ چیز دیگری نمی‌داند جز این‌که loss را کم کند. اگر loss را اشتباه انتخاب کنی، مدل دقیقاً همان چیز اشتباه را بهینه می‌کند.

این فصل، مهم‌ترین تصمیم بعد از انتخاب مسئله است.

### مفاهیم اساسی

**۱. سه خانواده‌ی اصلی:**

| مسئله | Loss | فرمول |
|---|---|---|
| **رگرسیون** | MSE | `(1/n)Σ(y-ŷ)²` |
| **رگرسیون (مقاوم به پرت)** | MAE / Huber | `(1/n)Σ\|y-ŷ\|` |
| **دسته‌بندی دودویی** | Binary Cross-Entropy | `-[y·log p + (1-y)·log(1-p)]` |
| **دسته‌بندی چندکلاسه** | Categorical Cross-Entropy | `-Σ yᵢ·log pᵢ` |
| **دسته‌بندی نامتوازن** | Focal Loss | `-α(1-p)ᵞ·log p` |

**۲. MSE در برابر MAE:**

| | MSE | MAE |
|---|---|---|
| حساسیت به پرت | **زیاد** | کم |
| مشتق در صفر | صاف | غیرصاف |
| تفسیر | واحد مربع | واحد اصلی |
| کِی | خطاهای بزرگ مهم‌اند | پرت داریم |

**Huber** ترکیب هر دو است: برای خطاهای کوچک مثل MSE، برای بزرگ مثل MAE.

**۳. چرا cross-entropy برای دسته‌بندی؟**

- **MSE با softmax** گرادیان کوچک می‌دهد وقتی مدل خیلی اشتباه است (چون softmax اشباع می‌شود) → یادگیری کند.
- **Cross-entropy** گرادیان قوی می‌دهد: `∂L/∂z = p - y`. ساده و مؤثر.

**این یکی از زیباترین نتایج ریاضی در ML است:** ترکیب softmax + cross-entropy، گرادیانی به این سادگی می‌دهد.

**۴. Label Smoothing.** به جای برچسب سخت `[0, 0, 1]`، از `[0.05, 0.05, 0.9]` استفاده کن. جلوی بیش‌اطمینانی را می‌گیرد.

**۵. Focal Loss.** برای داده‌ی نامتوازن شدید:

```
L = -α(1-p)ᵞ · log(p)
```
- `(1-p)ᵞ`: نمونه‌های آسان (p بالا) وزن کم می‌گیرند.
- `α`: وزن کلاس‌ها.

**۶. Regularization در loss:**

```
L_total = L_data + λ·||w||²        (L2)
L_total = L_data + λ·||w||₁        (L1)
```
این، همان چیزی است که Ridge و Lasso انجام می‌دهند (فایل ۱۵).

**۷. loss در برابر metric — دوباره:**

مدل loss را کم می‌کند. تو metric را می‌سنجی. **اگر این دو ناهم‌راستا باشند، مدل ممکن است metric را خراب کند.** مثال: loss = MSE، metric = «تعداد پیش‌بینی‌های با خطای بیش از ۱۰٪».

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — پیاده‌سازی و مقایسه‌ی lossها:**

```python
import numpy as np

def mse(y, p):
    return np.mean((y - p) ** 2)

def mae(y, p):
    return np.mean(np.abs(y - p))

def huber(y, p, delta=1.0):
    d = y - p
    return np.mean(np.where(np.abs(d) <= delta,
                            0.5 * d ** 2,
                            delta * (np.abs(d) - 0.5 * delta)))

y = np.array([1.0, 2.0, 3.0, 4.0, 100.0])      # یک پرت
p_good = np.array([1.1, 1.9, 3.2, 3.8, 100.5]) # پیش‌بینی خوب
p_bad  = np.array([1.1, 1.9, 3.2, 3.8, 50.0])  # پرت را بد پیش‌بینی کرد

print(f"{'':8} {'MSE':>10} {'MAE':>10} {'Huber':>10}")
for name, pred in [("خوب", p_good), ("بد", p_bad)]:
    print(f"{name:8} {mse(y, pred):>10.3f} {mae(y, pred):>10.3f} {huber(y, pred):>10.3f}")
```

**درس:** با یک پرت، MSE منفجر می‌شود اما MAE تقریباً ثابت می‌ماند. **انتخاب loss، انتخاب رفتار مدل است.**

**مثال ۲ — cross-entropy و گرادیانش:**

```python
def softmax(z):
    z = z - z.max(axis=1, keepdims=True)
    e = np.exp(z)
    return e / e.sum(axis=1, keepdims=True)

def cross_entropy(y_onehot, probs):
    eps = 1e-12
    return -np.mean(np.sum(y_onehot * np.log(probs + eps), axis=1))

# ۳ نمونه، ۳ کلاس
logits = np.array([[2.0, 1.0, 0.1],
                   [0.5, 2.5, 0.3],
                   [0.1, 0.2, 3.0]])
y_true = np.array([[1, 0, 0],
                   [0, 1, 0],
                   [0, 0, 1]])

probs = softmax(logits)
print("probs:\n", probs.round(3))
print("cross-entropy:", round(cross_entropy(y_true, probs), 4))

# گرادیان ساده‌شده: p - y  (بعد از softmax)
grad = probs - y_true
print("گرادیان (p - y):\n", grad.round(3))
```

**این `p - y`، سادگی شگفت‌انگیز softmax + cross-entropy را نشان می‌دهد.** به همین دلیل این ترکیب استاندارد است.

**مثال ۳ — Focal Loss برای نامتوازن:**

```python
def focal_loss(y, p, alpha=0.25, gamma=2.0):
    eps = 1e-12
    p = np.clip(p, eps, 1 - eps)
    ce = -(y * np.log(p) + (1 - y) * np.log(1 - p))
    pt = np.where(y == 1, p, 1 - p)
    alpha_t = np.where(y == 1, alpha, 1 - alpha)
    return np.mean(alpha_t * (1 - pt) ** gamma * ce)

y = np.array([1, 1, 1, 0, 0])
p = np.array([0.95, 0.6, 0.3, 0.9, 0.5])   # نمونه‌ی اول آسان، سوم سخت

print("cross-entropy ساده:", round(-np.mean(y*np.log(p+1e-12) + (1-y)*np.log(1-p+1e-12)), 4))
print("focal loss:         ", round(focal_loss(y, p), 4))
```

**اثر:** نمونه‌ی آسان (p=0.95) تقریباً نادیده گرفته می‌شود؛ نمونه‌ی سخت (p=0.3) وزن بیشتری می‌گیرد.

**مثال ۴ — اثر label smoothing:**

```python
def smooth_labels(y_onehot, eps=0.1):
    n_classes = y_onehot.shape[1]
    return y_onehot * (1 - eps) + eps / n_classes

print("برچسب سخت:\n", y_true[0])
print("برچسب نرم:\n", smooth_labels(y_true)[0].round(3))
```

### مقایسه و دیدگاه‌های مختلف

| Loss | مناسب | نامناسب |
|---|---|---|
| MSE | رگرسیون، خطای متقارن | داده‌ی پرت |
| MAE | داده‌ی پرت | نیاز به مشتق صاف |
| Huber | تعادل | تنظیم `delta` |
| BCE | دودویی | چندکلاسه |
| CE | چندکلاسه | رگرسیون |
| Focal | نامتوازن شدید | داده‌ی متعادل |
| Dice/IoU | تقسیم تصویر (segmentation) | دسته‌بندی ساده |

**یک دیدگاه مهم:** «loss را بر اساس هزینه‌ی کسب‌وکار انتخاب کن، نه راحتی ریاضی.» اگر خطای ۱۰۰ واحدی ۱۰ برابر خطای ۱۰ واحدی هزینه دارد، MSE منطقی است. اگر یک خطای بزرگ فاجعه است، MAE.

### اشتباهات رایج

**۱. MSE برای دسته‌بندی.** گرادیان ضعیف، آموزش کند.
**۲. cross-entropy با خروجی غیر softmax.** باید logits را به softmax بدهی.
**۳. MSE روی داده‌ی پرت بدون بررسی.** مدل به پرت‌ها وسواس پیدا می‌کند.
**۴. `log(0)` بدون `eps`.** `inf` و NaN.
**۵. بهینه‌سازی metric به جای loss.** اگر metric مشتق‌پذیر نیست، از surrogate loss استفاده کن و metric را جدا بسنج.
**۶. نادیده گرفتن ناهم‌راستایی loss و metric.** مدل ممکن است metric را بدتر کند در حالی که loss بهتر می‌شود.

### نکات مهم و ترفندها

- **رگرسیون با پرت:** Huber یا MAE.
- **دسته‌بندی نامتوازن:** Focal Loss یا `class_weight`.
- **`eps=1e-12` در log، همیشه.**
- **Label smoothing برای مدل‌های بزرگ** (خصوصاً ترانسفورمر) مفید است.
- **میان‌بر:** رگرسیون → MSE/Huber · دسته‌بندی → cross-entropy · نامتوازن → focal.

### خلاصه فصل

- Loss، تنها سیگنال یادگیری است؛ انتخابش رفتار مدل را تعیین می‌کند.
- MSE به پرت حساس، MAE مقاوم، Huber تعادل.
- cross-entropy گرادیان ساده و قوی می‌دهد: `p - y`.
- Focal Loss برای داده‌ی نامتوازن شدید.
- loss و metric می‌توانند ناهم‌راستا باشند.

**سؤالات مروری:**
1. چرا cross-entropy برای دسته‌بندی بهتر از MSE است؟
2. کِی MAE را به MSE ترجیح می‌دهی؟
3. Focal Loss چه مشکلی را حل می‌کند؟

---

## فصل ۵۹: Backpropagation: زنجیره‌ی مشتق به زبان ساده

### مقدمه

Backpropagation، **ترسناک‌ترین نام** در یادگیری ماشین است. اما ایده‌اش ساده است:

> اگر می‌خواهی بدانی هر وزن چقدر در خطا نقش دارد، از خروجی به عقب برگرد و در هر لایه، سهم را حساب کن.

این، فقط **قاعده‌ی زنجیره** است که در فایل ۰۴ یاد گرفتیم — با سازمان‌دهی هوشمند.

### مفاهیم اساسی

**۱. قاعده‌ی زنجیره، یک بار دیگر:**

```
اگر L = f(g(h(w)))، آنگاه dL/dw = f'·g'·h'
```

هر لایه، مشتق خودش را به مشتق لایه‌ی بعدی **ضرب** می‌کند.

**۲. چرا «عقب»؟**

چون خطا در خروجی محاسبه می‌شود و باید به لایه‌های داخلی برسد. Backprop، گرادیان را از آخر به اول منتقل می‌کند و **مقادیر مشترک را دوباره استفاده می‌کند** (به همین دلیل کارآمد است).

**۳. چهار معادله‌ی backprop (برای یک شبکه‌ی ساده):**

```
۱. خطای لایه‌ی خروجی:   δ_L = ∂L/∂ŷ · act'(z_L)
۲. خطای لایه‌ی قبل:     δ_l = (W_{l+1}ᵀ · δ_{l+1}) ⊙ act'(z_l)
۳. گرادیان وزن:         ∂L/∂W_l = h_{l-1}ᵀ · δ_l
۴. گرادیان بایاس:       ∂L/∂b_l = Σ δ_l
```

**تفسیر شهودی:**

- `δ` = «چقدر این لایه در خطا نقش دارد».
- `Wᵀ·δ` = «سهم این لایه، از طریق وزن‌هایش به لایه‌ی بعد».
- `⊙ act'` = «فقط جایی که نورون فعال بوده، عبور کن» (ReLU: مشتق ۱ برای مثبت، ۰ برای منفی).

**۴. گرادیان توابع فعال‌سازی:**

| تابع | مشتق |
|---|---|
| ReLU | `1 if x > 0 else 0` |
| Sigmoid | `σ(x)(1-σ(x))` |
| Tanh | `1 - tanh²(x)` |
| Linear | `1` |

**۵. محوشدگی گرادیان (Vanishing Gradient).**

اگر مشتق‌ها همه کوچک‌تر از ۱ باشند (مثل sigmoid: حداکثر ۰.۲۵)، ضرب ۱۰ لایه یعنی `0.25¹⁰ ≈ 1e-6`. گرادیان صفر می‌شود و لایه‌های اولیه یاد نمی‌گیرند.

**راه‌حل‌ها:**
- ReLU (مشتق ۱ برای مثبت)
- Residual connections (ResNet): `h = f(x) + x` — گرادیان مسیر میان‌بر دارد.
- BatchNorm
- مقداردهی اولیه‌ی مناسب (He، Xavier)

**۶. انفجار گرادیان (Exploding Gradient).** برعکس: مشتق‌ها بزرگ‌تر از ۱، ضرب مکرر → `inf`. راه‌حل: **gradient clipping**.

```python
torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
```

**۷. Gradient Checking.** مقایسه‌ی گرادیان تحلیلی با عددی:

```python
(grad_analytic - grad_numeric) / (grad_analytic + grad_numeric) < 1e-6
```

ابزار طلایی برای دیباگ backprop دستی.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — backprop کامل برای یک شبکه‌ی دو‌لایه:**

```python
import numpy as np

def relu(x):
    return np.maximum(0, x)

def relu_grad(x):
    return (x > 0).astype(float)

def mse_loss(y, p):
    return np.mean((y - p) ** 2)

def mse_grad(y, p):
    return 2 * (p - y) / y.size

rng = np.random.default_rng(0)

# داده: ۵ نمونه، ۳ ویژگی → ۱ خروجی
X = rng.normal(size=(5, 3))
y = rng.normal(size=(5, 1))

# پارامترها
W1 = rng.normal(size=(3, 4)) * 0.5; b1 = np.zeros(4)
W2 = rng.normal(size=(4, 1)) * 0.5; b2 = np.zeros(1)

# ---------- FORWARD ----------
z1 = X @ W1 + b1
h1 = relu(z1)
z2 = h1 @ W2 + b2
loss = mse_loss(y, z2)

# ---------- BACKWARD ----------
# لایه‌ی خروجی
dz2 = mse_grad(y, z2)                    # ∂L/∂z2 (لایه‌ی خروجی خطی است)
dW2 = h1.T @ dz2
db2 = dz2.sum(axis=0)

# لایه‌ی پنهان
dh1 = dz2 @ W2.T
dz1 = dh1 * relu_grad(z1)
dW1 = X.T @ dz1
db1 = dz1.sum(axis=0)

print(f"loss: {loss:.4f}")
print("شکل گرادیان‌ها:", dW1.shape, db1.shape, dW2.shape, db2.shape)

# ---------- بررسی با مشتق عددی ----------
def numeric_grad(param, idx, eps=1e-6):
    original = param[idx]
    param[idx] = original + eps
    z1p = X @ W1 + b1; h1p = relu(z1p); lp = mse_loss(y, h1p @ W2 + b2)
    param[idx] = original - eps
    z1m = X @ W1 + b1; h1m = relu(z1m); lm = mse_loss(y, h1m @ W2 + b2)
    param[idx] = original
    return (lp - lm) / (2 * eps)

for idx in [(0, 0), (1, 2), (2, 3)]:
    num = numeric_grad(W1, idx)
    ana = dW1[idx]
    print(f"W1{idx}: تحلیلی={ana:+.6f} عددی={num:+.6f} | تفاوت={abs(ana-num):.2e}")
```

**اگر تفاوت‌ها در حد `1e-8` باشند، backprop تو درست است.** این تکنیک (gradient checking) را برای هر پیاده‌سازی دستی استفاده کن.

**مثال ۲ — آموزش کامل با backprop دستی:**

```python
def train(X, y, epochs=500, lr=0.1):
    W1 = rng.normal(size=(3, 4)) * 0.5; b1 = np.zeros(4)
    W2 = rng.normal(size=(4, 1)) * 0.5; b2 = np.zeros(1)
    losses = []

    for _ in range(epochs):
        # forward
        z1 = X @ W1 + b1; h1 = relu(z1)
        z2 = h1 @ W2 + b2
        loss = mse_loss(y, z2); losses.append(loss)

        # backward
        dz2 = mse_grad(y, z2)
        dW2 = h1.T @ dz2; db2 = dz2.sum(axis=0)
        dh1 = dz2 @ W2.T
        dz1 = dh1 * relu_grad(z1)
        dW1 = X.T @ dz1; db1 = dz1.sum(axis=0)

        # update
        W1 -= lr * dW1; b1 -= lr * db1
        W2 -= lr * dW2; b2 -= lr * db2

    return losses

losses = train(X, y)
print(f"loss اول: {losses[0]:.4f} | loss آخر: {losses[-1]:.4f}")
```

**مثال ۳ — اثر عمق بر محوشدگی:**

```python
def simulate_vanishing(depth, act_grad_max=0.25):
    """شبیه‌سازی ضرب مشتق‌ها در عمق‌های مختلف."""
    return act_grad_max ** depth

for depth in [1, 5, 10, 20, 50]:
    print(f"عمق {depth:>2}: گرادیان ≈ {simulate_vanishing(depth):.2e}")
```

**با sigmoid (حداکثر مشتق ۰.۲۵):**
- عمق ۵ → `1e-3`
- عمق ۱۰ → `1e-6`
- عمق ۲۰ → `1e-12` (عملاً صفر)

**این، دلیل اصلی انقلاب ReLU است.**

**مثال ۴ — همان آموزش با autograd:**

```python
try:
    import torch

    X_t = torch.tensor(X, dtype=torch.float32)
    y_t = torch.tensor(y, dtype=torch.float32)

    model = torch.nn.Sequential(
        torch.nn.Linear(3, 4), torch.nn.ReLU(), torch.nn.Linear(4, 1)
    )
    optimizer = torch.optim.SGD(model.parameters(), lr=0.1)
    criterion = torch.nn.MSELoss()

    for epoch in range(200):
        optimizer.zero_grad()
        pred = model(X_t)
        loss = criterion(pred, y_t)
        loss.backward()          # ← backprop خودکار!
        optimizer.step()

    print("loss نهایی:", round(loss.item(), 4))
except ImportError:
    print("pip install torch")
```

**`loss.backward()`، همان ۱۰ خط دستی مثال قبل است** — اما برای هر معماری‌ای کار می‌کند.

### مقایسه و دیدگاه‌های مختلف

| روش | مزیت | عیب | کِی |
|---|---|---|---|
| Backprop دستی | درک کامل | خطاپذیر | یادگیری، دیباگ |
| Autograd (PyTorch) | دقیق، عمومی | کندتر از دستی بهینه | همیشه در عمل |
| گرادیان عددی | ساده، بدون مشتق | خیلی کند، تقریبی | gradient checking |
| گرادیان نمادین | دقیق | پیچیده | تحقیق |

**یک دیدگاه:** «لازم نیست backprop را دستی بنویسی.» درست، اما **اگر یک بار ننویسی، هرگز نمی‌فهمی چرا مدلت آموزش نمی‌بیند.**

### اشتباهات رایج

**۱. فراموش کردن `optimizer.zero_grad()`.** گرادیان‌ها جمع می‌شوند.
**۲. `backward()` روی مقدار غیر scalar بدون `retain_graph`.** خطا.
**۳. مشتق ReLU در صفر.** قرارداد: ۰.
**۴. نادرست بودن ترانهاده در گرادیان.** `W.T` و `h.T` را چک کن.
**۵. استفاده از sigmoid در شبکه‌ی عمیق.** محوشدگی.
**۶. نبود gradient clipping در RNN/Transformer.** انفجار گرادیان.
**۷. اعتماد به backprop دستی بدون gradient checking.** همیشه چک کن.

### نکات مهم و ترفندها

- **همیشه gradient checking کن** وقتی backprop دستی می‌نویسی.
- **`zero_grad()` را اولین خط حلقه بگذار.**
- **ReLU + He initialization** برای شبکه‌های عمیق.
- **`clip_grad_norm_` برای RNN و Transformer.**
- **اگر loss ناپایدار است، اول به گرادیان نگاه کن:** `print(grad.norm())`.
- **میان‌بر:** `δ` از آخر به اول · `dW = hᵀ·δ` · `dh = δ·Wᵀ`.

### خلاصه فصل

- backprop، قاعده‌ی زنجیره است که از خروجی به ورودی اعمال می‌شود.
- `δ` سهم هر لایه در خطا را می‌سنجد.
- `dW = hᵀ·δ` و `dh = Wᵀ·δ`.
- محوشدگی گرادیان، مشکل شبکه‌های عمیق با sigmoid است؛ ReLU و residual حلش می‌کنند.
- gradient checking، ابزار طلایی دیباگ.

**سؤالات مروری:**
1. چرا backprop از خروجی شروع می‌شود؟
2. محوشدگی گرادیان چیست و دو راه‌حل بگو.
3. گرادیان وزن در یک لایه چگونه از `δ` محاسبه می‌شود؟

---

## فصل ۶۰: Gradient Descent و learning rate

### مقدمه

اگر backprop بگوید «کدام جهت»، gradient descent می‌گوید «چقدر دور برو». این «چقدر»، همان **learning rate** است — و مهم‌ترین فراپارامتر آموزش.

این فصل، از ساده‌ترین شکل (SGD) تا مدرن‌ترین (Adam) را پوشش می‌دهد و نشان می‌دهد چرا انتخاب optimizer، گاهی تفاوت بین شکست و موفقیت است.

### مفاهیم اساسی

**۱. سه نوع gradient descent:**

| نوع | داده در هر گام | مزیت | عیب |
|---|---|---|---|
| **Batch GD** | کل داده | گرادیان دقیق، مسیر صاف | کند، حافظه‌بر |
| **SGD** | یک نمونه | سریع، از کمینه‌ی محلی فرار می‌کند | پرنویز |
| **Mini-batch** | ۳۲–۲۵۶ نمونه | تعادل | نیاز به تنظیم batch size |

**در عمل، همیشه mini-batch.** وقتی می‌گوییم SGD، منظور mini-batch SGD است.

**۲. قانون به‌روزرسانی:**

```
w ← w - lr · ∇L(w)
```

**۳. Momentum — «توده‌ی حرکتی».**

مشکل SGD: در دره‌های باریک، نوسان می‌کند. راه‌حل: سرعت قبلی را نگه دار.

```
v ← β·v + ∇L
w ← w - lr·v
```
`β` معمولاً ۰.۹. نتیجه: حرکت در جهت پایدار، نوسان کمتر.

**۴. RMSProp و AdaGrad — نرخ یادگیری تطبیقی.**

ایده: وزن‌هایی که گرادیان بزرگ دارند، lr کوچک‌تر؛ وزن‌های با گرادیان کوچک، lr بزرگ‌تر.

```
RMSProp: s ← β·s + (1-β)·(∇L)²
         w ← w - lr·∇L / (√s + ε)
```

**۵. Adam — ترکیب Momentum و RMSProp.**

```
m ← β₁·m + (1-β₁)·∇L          (momentum)
v ← β₂·v + (1-β₂)·(∇L)²       (RMSProp)
m̂ = m / (1-β₁ᵗ)               (تصحیح بایاس)
v̂ = v / (1-β₂ᵗ)
w ← w - lr·m̂ / (√v̂ + ε)
```

پیش‌فرض‌ها: `β₁=0.9`, `β₂=0.999`, `ε=1e-8`, `lr=0.001`.

**Adam تقریباً همیشه کار می‌کند** — به همین دلیل پیش‌فرض است.

**۶. انتخاب learning rate — مهم‌ترین تصمیم:**

| مقدار | نشانه |
|---|---|
| خیلی کوچک | loss به‌آرامی کاهش می‌یابد، ساعت‌ها طول می‌کشد |
| خیلی بزرگ | loss می‌پرد، NaN می‌شود |
| مناسب | کاهش پایدار و سریع |

**۷. Learning Rate Scheduling:**

- **Step decay:** هر ۱۰ epoch، نصف کن.
- **Cosine annealing:** کاهش نرم و دوره‌ای.
- **Warmup:** از lr کوچک شروع، بعد افزایش، بعد کاهش (استاندارد ترانسفورمر).
- **ReduceLROnPlateau:** اگر validation بهتر نشد، lr را کم کن.

**۸. Batch size و lr به هم وابسته‌اند.**
قاعده‌ی سرانگشتی: اگر batch را ۲ برابر کردی، lr را هم ۲ برابر کن (linear scaling rule).

**۹. Warmup — چرا لازم است؟**
در ابتدای آموزش، گرادیان‌ها پرنویزند. lr بزرگ، مدل را منفجر می‌کند. Warmup، چند صد قدم اول را با lr کوچک شروع می‌کند.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — مقایسه‌ی lrها:**

```python
import numpy as np

def f(w):        return (w - 3) ** 2 + 1
def grad(w):     return 2 * (w - 3)

def run(lr, steps=30, start=10.0):
    w, path = start, [start]
    for _ in range(steps):
        w -= lr * grad(w)
        path.append(w)
        if not np.isfinite(w) or abs(w) > 1e6:
            return path, "واگرا"
    return path, "همگرا" if abs(path[-1] - 3) < 0.01 else "نزدیک نشد"

for lr in [0.001, 0.01, 0.1, 0.5, 0.9, 1.1]:
    path, status = run(lr)
    print(f"lr={lr:<6} | w نهایی={path[-1]:>12.4f} | {status}")
```

**مثال ۲ — SGD در برابر Momentum در یک دره‌ی باریک:**

```python
def rosenbrock(w, lr=0.001, steps=2000, momentum=0.0):
    """تابع روزنبروک — دره‌ی باریک، چالش کلاسیک بهینه‌سازی."""
    x, y = -1.0, 1.0
    vx = vy = 0.0
    path = [(x, y)]

    for _ in range(steps):
        dx = -2 * (1 - x) - 400 * x * (y - x ** 2)
        dy = 200 * (y - x ** 2)

        vx = momentum * vx + lr * dx
        vy = momentum * vy + lr * dy
        x -= vx
        y -= vy
        path.append((x, y))

        if not (np.isfinite(x) and np.isfinite(y)):
            break

    return path

for mom in [0.0, 0.9]:
    path = rosenbrock(momentum=mom)
    final = path[-1]
    print(f"momentum={mom} | نقطه‌ی نهایی=({final[0]:.3f}, {final[1]:.3f}) "
          f"| فاصله از بهینه={( (final[0]-1)**2 + (final[1]-1)**2 )**0.5:.4f}")
```

**مثال ۳ — Adam از صفر:**

```python
class Adam:
    def __init__(self, lr=0.001, b1=0.9, b2=0.999, eps=1e-8):
        self.lr, self.b1, self.b2, self.eps = lr, b1, b2, eps
        self.m = {}
        self.v = {}
        self.t = 0

    def step(self, params, grads):
        self.t += 1
        for key in params:
            g = grads[key]
            self.m[key] = self.b1 * self.m.get(key, 0) + (1 - self.b1) * g
            self.v[key] = self.b2 * self.v.get(key, 0) + (1 - self.b2) * g ** 2

            m_hat = self.m[key] / (1 - self.b1 ** self.t)
            v_hat = self.v[key] / (1 - self.b2 ** self.t)

            params[key] -= self.lr * m_hat / (np.sqrt(v_hat) + self.eps)
        return params

params = {"w": np.array([0.0, 0.0])}
opt = Adam(lr=0.1)

for step in range(200):
    grads = {"w": 2 * (params["w"] - np.array([3.0, -2.0]))}
    params = opt.step(params, grads)

print("نتیجه:", params["w"].round(4), "(هدف: [3, -2])")
```

**مثال ۴ — LR scheduler:**

```python
def cosine_schedule(step, total_steps, lr_max=1e-3, lr_min=1e-5):
    return lr_min + 0.5 * (lr_max - lr_min) * (1 + np.cos(np.pi * step / total_steps))

def warmup_cosine(step, total_steps, warmup_steps=100, lr_max=1e-3):
    if step < warmup_steps:
        return lr_max * step / warmup_steps
    progress = (step - warmup_steps) / max(1, total_steps - warmup_steps)
    return 1e-5 + 0.5 * (lr_max - 1e-5) * (1 + np.cos(np.pi * progress))

print(f"{'step':>6} {'cosine':>12} {'warmup+cos':>12}")
for step in [0, 50, 100, 250, 500, 1000]:
    print(f"{step:>6} {cosine_schedule(step, 1000):>12.6f} {warmup_cosine(step, 1000):>12.6f}")
```

**مثال ۵ — PyTorch: تنظیم optimizer و scheduler:**

```python
try:
    import torch
    import torch.nn as nn

    model = nn.Sequential(nn.Linear(10, 32), nn.ReLU(), nn.Linear(32, 1))
    optimizer = torch.optim.AdamW(model.parameters(), lr=1e-3, weight_decay=1e-4)
    scheduler = torch.optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=100)

    # در حلقه:
    # optimizer.zero_grad()
    # loss = criterion(model(X), y)
    # loss.backward()
    # torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
    # optimizer.step()
    # scheduler.step()
    print("optimizer:", optimizer.__class__.__name__)
    print("lr اولیه:", optimizer.param_groups[0]["lr"])
except ImportError:
    print("pip install torch")
```

### مقایسه و دیدگاه‌های مختلف

| Optimizer | سرعت همگرایی | حافظه | تنظیم | توصیه |
|---|---|---|---|---|
| SGD | کند | کم | سخت | با momentum، برای CNN |
| SGD+Momentum | متوسط | کم | متوسط | استاندارد CNN |
| RMSProp | سریع | متوسط | آسان | RNN |
| Adam | سریع | ۲× مدل | آسان | پیش‌فرض عمومی |
| AdamW | سریع | ۲× مدل | آسان | ترانسفورمر |

**دو دیدگاه:**
- **«Adam همیشه بهتر است.»** برای شروع، بله. اما در بعضی مسائل، SGD+momentum بهتر generalize می‌کند.
- **«SGD+momentum برای بینایی بهتر است.»** در مقالات CNN، شواهدی وجود دارد. اما تفاوت اغلب کوچک است.

**توصیه‌ی عملی:** با Adam شروع کن، اگر نتیجه خوب بود تمام. اگر دنبال آخرین درصد دقت هستی، SGD+momentum را امتحان کن.

### اشتباهات رایج

**۱. lr پیش‌فرض بدون تنظیم.** ۰.۰۰۱ برای همه‌ی مسائل مناسب نیست.
**۲. lr بزرگ.** loss می‌پرد یا NaN می‌شود.
**۳. نبود warmup در ترانسفورمر.** آموزش ناپایدار.
**۴. `scheduler.step()` در جای اشتباه.** در PyTorch، بعد از `optimizer.step()`.
**۵. batch بزرگ + lr کوچک.** همگرایی بسیار کند.
**۶. فراموش کردن `weight_decay`.** بیش‌برازش.
**۷. تغییر optimizer در وسط آموزش.** وضعیت (momentum، moments) از دست می‌رود.

### نکات مهم و ترفندها

- **اگر loss NaN شد، اول lr را ۱۰ برابر کم کن.**
- **`clip_grad_norm_(model.parameters(), 1.0)` را در ترانسفورمر و RNN بگذار.**
- **`AdamW` به `Adam` ترجیح داده می‌شود** (weight decay درست‌تر).
- **`ReduceLROnPlateau` ساده‌ترین scheduler است:** اگر بهتر نشد، lr کم کن.
- **منحنی loss را با lrهای مختلف رسم کن** — سریع‌ترین راه پیدا کردن lr مناسب.
- **میان‌بر:** `w ← w - lr·∇L` · lr بزرگ = واگرا · lr کوچک = کند · Adam = تطبیقی.

### خلاصه فصل

- gradient descent جهت و اندازه‌ی قدم را تعیین می‌کند.
- SGD پرنویز، Batch کند، mini-batch تعادل.
- Momentum نوسان را کم می‌کند، Adam نرخ یادگیری را تطبیقی می‌کند.
- lr مهم‌ترین فراپارامتر است؛ خیلی بزرگ واگرا، خیلی کوچک کند.
- Scheduler و warmup، آموزش مدل‌های بزرگ را ممکن می‌کنند.

**سؤالات مروری:**
1. چرا mini-batch استاندارد است؟
2. Momentum چه مشکلی را حل می‌کند؟
3. اگر loss بعد از چند قدم NaN شد، اولین کاری که می‌کنی چیست؟

---

## منابع و مراجع

**کتاب‌ها**
- Goodfellow, I. et al. — *Deep Learning*، فصل ۶ (Backprop)، ۷ (Regularization)، ۸ (Optimization)
- Géron, A. — *Hands-On ML*، فصل ۴ (Training Models)، ۱۱ (Training Deep Nets)
- Nielsen, M. — *Neural Networks and Deep Learning*، فصل ۲ (بهترین توضیح backprop)

**مقالات کلیدی**
- Rumelhart, D. et al. (1986) — *Learning representations by back-propagating errors*
- Kingma, D. & Ba, J. (2014) — *Adam: A Method for Stochastic Optimization*
- He, K. et al. (2015) — *Delving Deep into Rectifiers* (He initialization)
- Ioffe, S. & Szegedy, C. (2015) — *Batch Normalization*
- Loshchilov, I. & Hutter, F. (2017) — *Decoupled Weight Decay Regularization* (AdamW)

**ابزارها**
- `torch.autograd` — مستندات رسمی
- Andrej Karpathy — *micrograd* (پیاده‌سازی autograd در ۱۵۰ خط — فوق‌العاده برای یادگیری)

---

**بعدی:** [`14-Evaluation-Metrics.md`](./14-Evaluation-Metrics.md) — فصل ۶۱: چرا دقت کافی نیست
