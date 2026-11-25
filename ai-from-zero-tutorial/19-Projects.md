# بخش ۱۹: پنج پروژه‌ی عملی

**مدت یادگیری تقریبی**: ۳ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل‌های ۰۶، ۰۸، ۱۳، ۱۴، ۱۶، ۱۷

---

## فهرست محتوا

- [فصل ۸۷: پروژه ۱: دسته‌بندی گل‌ها (Iris)](#فصل-۸۷-پروژه-۱-دستهبندی-گلها-iris)
- [فصل ۸۸: پروژه ۲: پیش‌بینی قیمت خانه](#فصل-۸۸-پروژه-۲-پیشبینی-قیمت-خانه)
- [فصل ۸۹: پروژه ۳: تشخیص الگو در تصاویر](#فصل-۸۹-پروژه-۳-تشخیص-الگو-در-تصاویر)
- [فصل ۹۰: پروژه ۴: تحلیل احساسات متن](#فصل-۹۰-پروژه-۴-تحلیل-احساسات-متن)
- [فصل ۹۱: پروژه ۵: دستیار ساده با API مدل زبانی](#فصل-۹۱-پروژه-۵-دستیار-ساده-با-api-مدل-زبانی)

---

## مقدمه‌ی بخش: چرا پروژه؟

تا اینجا ۸۶ فصل خوانده‌ای. اما **خواندن، یادگیری نیست**. یادگیری وقتی اتفاق می‌افتد که خودت از صفر بنویسی و خطا بخوری.

این پنج پروژه، **به ترتیب سختی** چیده شده‌اند و هر کدام یک مهارت متفاوت را تمرین می‌دهند:

| پروژه | مهارت اصلی | زمان |
|---|---|---|
| ۱. Iris | چرخه‌ی کامل ML | ۳۰ دقیقه |
| ۲. قیمت خانه | مهندسی ویژگی + رگرسیون | ۴۵ دقیقه |
| ۳. ارقام دست‌نویس | شبکه عصبی + ارزیابی تصویر | ۴۵ دقیقه |
| ۴. تحلیل احساسات | NLP + embedding | ۳۰ دقیقه |
| ۵. دستیار LLM | API + prompt + اعتبارسنجی | ۳۰ دقیقه |

**قاعده:** هر پروژه را **اول خودت بنویس**، بعد با کد این فصل مقایسه کن.

---

## فصل ۸۷: پروژه ۱: دسته‌بندی گل‌ها (Iris)

### مقدمه

Iris، معروف‌ترین دیتاست یادگیری ماشین است: ۱۵۰ گل، ۴ ویژگی (طول و عرض کاسبرگ و گلبرگ)، ۳ گونه.

**چرا این پروژه؟** چون کوچک است (سریع اجرا می‌شود)، تمیز است (بدون گمشده) و **کل چرخه‌ی ML** را در ۵۰ خط نشان می‌دهد. هدف، دقت ۱۰۰٪ نیست؛ **اجرای درست چرخه** است.

### مفاهیم اساسی

**۱. چرخه‌ی هفت‌مرحله‌ای (تکرار از فایل ۰۸):**

```
بارگذاری → کاوش → تقسیم → Pipeline → آموزش → ارزیابی → تفسیر
```

**۲. معیارها برای این مسئله:**
- سه کلاس متعادل (۵۰/۵۰/۵۰) → accuracy معنی‌دار است.
- اما `macro F1` بهتر است (به همه‌ی کلاس‌ها وزن برابر می‌دهد).
- ماتریس درهم‌ریختگی نشان می‌دهد کدام کلاس‌ها اشتباه می‌شوند.

**۳. نکته‌ی مهم:** در Iris، دو کلاس اول تقریباً همیشه جدا می‌شوند؛ سختی در تمایز **versicolor** و **virginica** است.

### مثال‌ها و نمونه‌های عملی

**کد کامل پروژه:**

```python
"""
پروژه ۱: دسته‌بندی گل‌های Iris
اجرا: python iris_project.py
"""
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split, cross_val_score, StratifiedKFold
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier
from sklearn.dummy import DummyClassifier
from sklearn.metrics import (classification_report, confusion_matrix,
                             accuracy_score, f1_score)

# ============ ۱. بارگذاری ============
iris = load_iris()
X = pd.DataFrame(iris.data, columns=iris.feature_names)
y = pd.Series(iris.target, name="species")

print(f"شکل داده: {X.shape}")
print(f"توزیع کلاس:\n{y.value_counts().rename(index=dict(enumerate(iris.target_names)))}")

# ============ ۲. کاوش سریع ============
print("\n--- آمار توصیفی ---")
print(X.describe().T.round(2))

print("\n--- میانگین ویژگی‌ها به تفکیک گونه ---")
print(X.groupby(y).mean().round(2))

# نمودار جفت‌ها
df_plot = X.copy()
df_plot["species"] = y.map(dict(enumerate(iris.target_names)))
sns.pairplot(df_plot, hue="species", height=1.8, diag_kind="kde")
plt.suptitle("Iris — نمای کلی", y=1.02)
plt.show()

# ============ ۳. تقسیم ============
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42
)
print(f"\ntrain: {len(X_train)} | test: {len(X_test)}")

# ============ ۴. Baseline ============
baseline = DummyClassifier(strategy="most_frequent").fit(X_train, y_train)
print(f"\nBaseline accuracy: {baseline.score(X_test, y_test):.3f}")
print("(همیشه پرتکرارترین کلاس را می‌گوید — کف انتظار)")

# ============ ۵. مقایسه‌ی مدل‌ها با CV ============
cv = StratifiedKFold(5, shuffle=True, random_state=42)

candidates = {
    "Logistic Regression": Pipeline([
        ("scaler", StandardScaler()),
        ("clf", LogisticRegression(max_iter=1000, random_state=42)),
    ]),
    "Random Forest": RandomForestClassifier(n_estimators=200, random_state=42),
}

cv_results = {}
print(f"\n{'مدل':>22} {'CV F1 (macro)':>18}")
for name, model in candidates.items():
    scores = cross_val_score(model, X_train, y_train, cv=cv, scoring="f1_macro")
    cv_results[name] = scores
    print(f"{name:>22} {scores.mean():>10.3f} ± {scores.std():.3f}")

best_name = max(cv_results, key=lambda k: cv_results[k].mean())
print(f"\nبهترین مدل: {best_name}")

# ============ ۶. آموزش نهایی و ارزیابی ============
final_model = candidates[best_name].fit(X_train, y_train)
y_pred = final_model.predict(X_test)

print(f"\n--- ارزیابی روی test ---")
print(f"Accuracy: {accuracy_score(y_test, y_pred):.3f}")
print(f"F1 (macro): {f1_score(y_test, y_pred, average='macro'):.3f}")
print("\n" + classification_report(y_test, y_pred, target_names=iris.target_names, digits=3))

# ماتریس درهم‌ریختگی
cm = confusion_matrix(y_test, y_pred)
plt.figure(figsize=(5.5, 4.5))
sns.heatmap(cm, annot=True, fmt="d", cmap="Blues",
            xticklabels=iris.target_names, yticklabels=iris.target_names)
plt.xlabel("پیش‌بینی")
plt.ylabel("واقعیت")
plt.title(f"ماتریس درهم‌ریختگی — {best_name}")
plt.tight_layout()
plt.show()

# ============ ۷. تفسیر ============
if best_name == "Logistic Regression":
    clf = final_model.named_steps["clf"]
    coefs = pd.DataFrame(clf.coef_.T,
                         index=iris.feature_names,
                         columns=iris.target_names)
    print("\n--- ضرایب مدل (تأثیر هر ویژگی) ---")
    print(coefs.round(2))
    print("\nتفسیر: ضریب مثبت بزرگ = ویژگی مهم برای آن کلاس")

# ============ ۸. گزارش نهایی ============
print("\n" + "=" * 55)
print("گزارش نهایی")
print("=" * 55)
print(f"دیتاست: Iris ({len(X)} نمونه، {X.shape[1]} ویژگی، ۳ کلاس)")
print(f"مدل: {best_name}")
print(f"CV F1: {cv_results[best_name].mean():.3f}")
print(f"Test F1: {f1_score(y_test, y_pred, average='macro'):.3f}")
print(f"Baseline: {baseline.score(X_test, y_test):.3f}")
print(f"بهبود: {(f1_score(y_test, y_pred, average='macro') - baseline.score(X_test, y_test)):.3f}")
print("\nمحدودیت: ۱۵۰ نمونه، داده‌ی تمیز و متعادل — آسان‌تر از دنیای واقعی.")
```

**نتیجه‌ی معمول:** F1 ≈ ۰.۹۵–۱.۰. اگر گرفتی، خوب است — اما به یاد داشته باش که Iris **آسان** است.

### مقایسه و دیدگاه‌های مختلف

| رویکرد | F1 (تقریبی) | نکته |
|---|---|---|
| Baseline (most_frequent) | ۰.۲۲ | کف |
| Logistic | ۰.۹۷ | ساده و تفسیرپذیر |
| Random Forest | ۰.۹۶ | قوی |
| KNN (k=5) | ۰.۹۶ | نیاز به مقیاس‌بندی |
| SVM (RBF) | ۰.۹۷ | خوب |

**درس:** در مسائل ساده، همه‌ی مدل‌ها نزدیک‌اند. **تفاوت در مسائل سخت و داده‌ی واقعی ظاهر می‌شود.**

### اشتباهات رایج

**۱. گزارش accuracy بدون F1.** با کلاس‌های متعادل تفاوت کم است، اما عادت بدی است.
**۲. نبود baseline.** نمی‌دانی مدلت ارزش دارد یا نه.
**۳. مقیاس‌بندی نکردن برای Logistic/KNN.**
**۴. تفسیر ضرایب بدون استانداردسازی.**
**۵. نتیجه‌گیری از یک تقسیم.** CV لازم است.

### نکات مهم و ترفندها

- **Pipeline را از ابتدا بگذار.** عادت درست.
- **`stratify=y` را فراموش نکن.**
- **`pairplot` برای داده‌ی کوچک فوق‌العاده است.**
- **در گزارش نهایی، محدودیت‌ها را بنویس.** حرفه‌ای‌گری است.
- **میان‌بر:** داده → کاوش → تقسیم → baseline → CV → مدل نهایی → گزارش.

### خلاصه فصل

- Iris، چرخه‌ی کامل ML را در ۵۰ خط نشان می‌دهد.
- baseline و CV، دو چیز غیرقابل حذف‌اند.
- ماتریس درهم‌ریختگی، الگوی خطا را نشان می‌دهد.
- در مسائل ساده، تفاوت مدل‌ها کم است.
- گزارش با محدودیت‌ها، حرفه‌ای است.

**تمرین:**
1. KNN و SVM را اضافه کن و مقایسه کن.
2. `max_depth` درخت را تنظیم کن.
3. یک ویژگی جدید بساز (`petal_area = petal_length × petal_width`) و اثرش را ببین.

---

## فصل ۸۸: پروژه ۲: پیش‌بینی قیمت خانه

### مقدمه

این پروژه، **رگرسیون واقعی** است: هدف پیوسته، ویژگی‌های مختلط (عددی + دسته‌ای)، و داده‌ی کثیف.

مهارت‌های جدید: مهندسی ویژگی، مدیریت گمشده، مقیاس‌بندی، و **انتخاب معیار درست**.

### مفاهیم اساسی

**۱. تفاوت با Iris:**
- هدف پیوسته (قیمت) نه کلاس.
- داده‌ی گمشده و پرت دارد.
- ویژگی‌های دسته‌ای دارد.
- **دقت ۱۰۰٪ ممکن نیست** — باید بدانی «چقدر خطا قابل قبول است».

**۲. معیار انتخاب:**
- **MAE:** «میانگین خطا چند تومان است؟» — قابل فهم برای کسب‌وکار.
- **RMSE:** جریمه‌ی خطاهای بزرگ.
- **R²:** «چقدر بهتر از میانگین؟»
- **MAPE:** درصد خطا.

**۳. مهندسی ویژگی کلیدی:**
- `age = current_year - year_built`
- `price_per_sqm = price / area` (اگر هدف نباشد!)
- `rooms_per_area = rooms / area`
- `total_rooms = bedrooms + bathrooms`

**۴. تله‌ی مهم:** ویژگی‌ای که در زمان تصمیم موجود نیست (مثل `final_price`) نشت می‌دهد.

### مثال‌ها و نمونه‌های عملی

**کد کامل پروژه:**

```python
"""
پروژه ۲: پیش‌بینی قیمت خانه
داده‌ی مصنوعی واقع‌گرا (چون دیتاست واقعی در دسترس نیست)
"""
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

from sklearn.model_selection import train_test_split, KFold, cross_val_score
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.linear_model import Ridge
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor
from sklearn.dummy import DummyRegressor
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score

# ============ ۱. ساخت داده‌ی واقع‌گرا ============
rng = np.random.default_rng(42)
n = 2000

df = pd.DataFrame({
    "area":      rng.normal(110, 35, n).clip(30, 400).round(0),
    "rooms":     rng.integers(1, 6, n),
    "age":       rng.integers(0, 50, n),
    "floor":     rng.integers(0, 20, n),
    "district":  rng.choice(["شمال", "مرکز", "جنوب", "غرب"], n,
                            p=[0.3, 0.35, 0.2, 0.15]),
    "has_parking": rng.integers(0, 2, n),
})

# قیمت پایه: متراژ × قیمت منطقه − استهلاک + نویز
district_price = {"شمال": 90, "مرکز": 60, "جنوب": 35, "غرب": 50}
base = df["area"] * df["district"].map(district_price)
depreciation = 1 - df["age"] * 0.008
premium = 1 + df["has_parking"] * 0.06 + df["floor"] * 0.003
df["price"] = (base * depreciation * premium *
               rng.normal(1, 0.08, n)).round(0) / 1000   # میلیون تومان

# ایجاد گمشدگی و پرت مصنوعی (مثل داده‌ی واقعی)
df.loc[rng.choice(n, 80, replace=False), "area"] = np.nan
df.loc[rng.choice(n, 40, replace=False), "district"] = np.nan
df.loc[rng.choice(n, 10, replace=False), "price"] *= 4     # پرت

print(f"شکل: {df.shape}")
print(f"گمشدگی:\n{df.isna().sum()[df.isna().sum() > 0]}")

# ============ ۲. کاوش ============
print("\n--- آمار قیمت ---")
print(df["price"].describe().round(1))
print(f"\nمیانگین vs میانه: {df['price'].mean():.1f} vs {df['price'].median():.1f}")
print("(تفاوت زیاد = وجود پرت یا چولگی)")

fig, axes = plt.subplots(1, 2, figsize=(12, 4))
axes[0].hist(df["price"].dropna(), bins=50, edgecolor="white")
axes[0].set_title("توزیع قیمت")
axes[1].scatter(df["area"], df["price"], alpha=0.3, s=10)
axes[1].set_xlabel("متراژ"); axes[1].set_ylabel("قیمت (میلیون)")
axes[1].set_title("متراژ در برابر قیمت")
plt.tight_layout(); plt.show()

# ============ ۳. حذف پرت از هدف ============
q_low, q_high = df["price"].quantile([0.005, 0.995])
df = df[(df["price"] >= q_low) & (df["price"] <= q_high)].copy()
print(f"\nپس از حذف پرت: {len(df)} ردیف")

# ============ ۴. مهندسی ویژگی ============
df["area_per_room"] = df["area"] / df["rooms"]
df["is_high_floor"] = (df["floor"] > 10).astype(int)
df["district"] = df["district"].fillna("نامشخص")

print("\nستون‌های نهایی:", list(df.columns))

# ============ ۵. تقسیم ============
X = df.drop(columns="price")
y = df["price"]

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

num_cols = X.select_dtypes("number").columns.tolist()
cat_cols = ["district"]

print(f"عددی: {num_cols}")
print(f"دسته‌ای: {cat_cols}")

# ============ ۶. Pipeline ============
preprocessor = ColumnTransformer([
    ("num", Pipeline([
        ("impute", SimpleImputer(strategy="median")),
        ("scale", StandardScaler()),
    ]), num_cols),
    ("cat", Pipeline([
        ("impute", SimpleImputer(strategy="constant", fill_value="نامشخص")),
        ("onehot", OneHotEncoder(handle_unknown="ignore", sparse_output=False)),
    ]), cat_cols),
])

# ============ ۷. Baseline ============
baseline = DummyRegressor(strategy="median").fit(X_train, y_train)
base_mae = mean_absolute_error(y_test, baseline.predict(X_test))
print(f"\nBaseline (میانه) MAE: {base_mae:.1f} میلیون")

# ============ ۸. مقایسه‌ی مدل‌ها ============
models = {
    "Ridge": Ridge(alpha=10.0),
    "Random Forest": RandomForestRegressor(n_estimators=300, random_state=42, n_jobs=-1),
    "Gradient Boosting": GradientBoostingRegressor(n_estimators=300, learning_rate=0.05,
                                                   max_depth=4, random_state=42),
}

kf = KFold(5, shuffle=True, random_state=42)
print(f"\n{'مدل':>20} {'CV MAE':>12} {'CV R²':>10}")
results = {}

for name, model in models.items():
    pipe = Pipeline([("prep", preprocessor), ("model", model)])
    mae = -cross_val_score(pipe, X_train, y_train, cv=kf,
                           scoring="neg_mean_absolute_error", n_jobs=-1)
    r2 = cross_val_score(pipe, X_train, y_train, cv=kf, scoring="r2", n_jobs=-1)
    results[name] = (mae.mean(), r2.mean())
    print(f"{name:>20} {mae.mean():>12.1f} {r2.mean():>10.3f}")

best_name = min(results, key=lambda k: results[k][0])
print(f"\nبهترین: {best_name}")

# ============ ۹. آموزش نهایی ============
final = Pipeline([("prep", preprocessor), ("model", models[best_name])])
final.fit(X_train, y_train)
pred = final.predict(X_test)

mae = mean_absolute_error(y_test, pred)
rmse = np.sqrt(mean_squared_error(y_test, pred))
r2 = r2_score(y_test, pred)
mape = np.mean(np.abs((y_test - pred) / y_test)) * 100

print("\n--- ارزیابی روی test ---")
print(f"MAE:  {mae:.1f} میلیون تومان")
print(f"RMSE: {rmse:.1f} میلیون تومان")
print(f"R²:   {r2:.3f}")
print(f"MAPE: {mape:.1f}%")
print(f"\nبهبود نسبت به baseline: {(base_mae - mae) / base_mae:.1%}")

# ============ ۱۰. تحلیل خطا ============
residuals = y_test - pred
fig, axes = plt.subplots(1, 2, figsize=(12, 4))

axes[0].scatter(pred, residuals, alpha=0.4, s=12)
axes[0].axhline(0, color="red", ls="--")
axes[0].set_xlabel("پیش‌بینی"); axes[0].set_ylabel("باقیمانده")
axes[0].set_title("باقیمانده در برابر پیش‌بینی")

axes[1].hist(residuals, bins=40, edgecolor="white")
axes[1].set_title("توزیع باقیمانده")
plt.tight_layout(); plt.show()

print("\n--- تفسیر باقیمانده ---")
print("اگر الگو (منحنی) داری → رابطه‌ی غیرخطی مدل نشده")
print("اگر پهنای نوار با پیش‌بینی زیاد شود → واریانس ناهمگون")

# ============ ۱۱. اهمیت ویژگی ============
if best_name == "Random Forest":
    rf = final.named_steps["model"]
    feature_names = final.named_steps["prep"].get_feature_names_out()
    imp = pd.Series(rf.feature_importances_, index=feature_names).nlargest(8)
    print("\n--- مهم‌ترین ویژگی‌ها ---")
    print(imp.round(3))

# ============ ۱۲. پیش‌بینی نمونه‌ی جدید ============
new_house = pd.DataFrame([{
    "area": 120, "rooms": 3, "age": 5, "floor": 4,
    "district": "شمال", "has_parking": 1,
    "area_per_room": 40, "is_high_floor": 0,
}])
print(f"\nپیش‌بینی برای خانه‌ی نمونه: {final.predict(new_house)[0]:.1f} میلیون تومان")
```

### مقایسه و دیدگاه‌های مختلف

| مدل | MAE (تقریبی) | نکته |
|---|---|---|
| Baseline (میانه) | بالا | کف |
| Ridge | متوسط | سریع، تفسیرپذیر |
| Random Forest | بهتر | مقاوم |
| Gradient Boosting | بهترین | نیاز به تنظیم |

**درس:** در داده‌ی جدولی، **boosting** معمولاً برنده است — اما تفاوت با Ridge در داده‌ی ساده، کم است.

### اشتباهات رایج

**۱. حذف ردیف‌های با گمشده بدون بررسی.** ۸۰ ردیف = ۴٪ داده.
**۲. پرکردن متراژ با صفر.** باید میانه یا مدل‌محور.
**۳. نادیده گرفتن پرت در هدف.** MAE و RMSE را خراب می‌کند.
**۴. One-Hot بدون `handle_unknown`.** در test، دسته‌ی جدید خطا می‌دهد.
**۵. ارزیابی با R² تنها.** برای کسب‌وکار، MAE گویاتر است.
**۶. نبود تحلیل باقیمانده.** الگو در باقیمانده = مدل ناقص.

### نکات مهم و ترفندها

- **ColumnTransformer + Pipeline، استاندارد است.**
- **MAE را به کسب‌وکار گزارش کن** («میانگین ۱۲ میلیون خطا»).
- **باقیمانده‌ها را رسم کن.** بیشتر از R² می‌گوید.
- **قبل از هر چیز، baseline.** با میانگین/میانه مقایسه کن.
- **میان‌بر:** داده → پاکسازی → ویژگی → Pipeline → CV → ارزیابی → تحلیل خطا.

### خلاصه فصل

- رگرسیون واقعی: هدف پیوسته، داده‌ی کثیف، معیار چندگانه.
- مهندسی ویژگی، بیشترین اثر را دارد.
- Pipeline از نشت جلوگیری می‌کند.
- MAE برای کسب‌وکار، RMSE برای پرت.
- تحلیل باقیمانده، بخشی از ارزیابی است.

**تمرین:**
1. `district` را با Target Encoding کدگذاری کن و مقایسه کن.
2. ویژگی `price_per_sqm` را از داده‌ی *آموزش* بساز (مراقب نشت باش).
3. `RandomForest` را با `GridSearchCV` تنظیم کن.

---

## فصل ۸۹: پروژه ۳: تشخیص الگو در تصاویر

### مقدمه

این پروژه، **بینایی ماشین واقعی** است: تشخیص ارقام دست‌نویس (MNIST). اما به جای آموزش از صفر، از **transfer learning** استفاده می‌کنیم — چون این، روش استاندارد صنعت است.

مهارت‌های جدید: پردازش تصویر، CNN، augmentation، ارزیابی چندکلاسه.

### مفاهیم اساسی

**۱. MNIST:** ۷۰٬۰۰۰ تصویر ۲۸×۲۸ خاکستری، ۱۰ کلاس (۰–۹). آسان است — دقت بالای ۹۹٪ ممکن است.

**۲. چرا transfer learning؟** چون در دنیای واقعی، داده‌ی کم داری و مدل‌های از پیش آموزش‌دیده بهتر شروع می‌کنند.

**۳. سه مرحله‌ی پردازش تصویر:**

```python
transforms.Compose([
    transforms.Resize(224),           # اندازه‌ی مدل
    transforms.ToTensor(),            # 0-255 → 0-1
    transforms.Normalize(mean, std),  # آمار ImageNet
])
```

**۴. نکته‌ی مهم:** MNIST تک‌کاناله است؛ مدل‌های ImageNet سه‌کاناله‌اند. باید تکرار کانال کنی یا مدل را تغییر دهی.

**۵. معیارها:** accuracy (متعادل)، ماتریس درهم‌ریختگی (کدام ارقام اشتباه می‌شوند؟)، و **نگاه کردن به نمونه‌های اشتباه**.

### مثال‌ها و نمونه‌های عملی

**کد کامل پروژه (بدون دانلود، با داده‌ی ساختگی برای اجرای تضمینی):**

```python
"""
پروژه ۳: تشخیص الگو در تصاویر
دو نسخه: (الف) با sklearn روی داده‌ی ساختگی — همیشه اجرا می‌شود
         (ب) با PyTorch CNN روی MNIST — نیاز به دانلود
"""
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import load_digits
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.neural_network import MLPClassifier
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report, confusion_matrix, accuracy_score

# ============ الف) نسخه‌ی sklearn (سریع و بدون دانلود) ============
digits = load_digits()
X, y = digits.data, digits.target

print(f"شکل داده: {X.shape}")      # (1797, 64) — ۸×۸ تصویر
print(f"کلاس‌ها: {np.unique(y)}")

# نمایش چند نمونه
fig, axes = plt.subplots(2, 5, figsize=(10, 4))
for ax, idx in zip(axes.ravel(), range(10)):
    ax.imshow(digits.images[idx], cmap="gray_r")
    ax.set_title(f"برچسب: {digits.target[idx]}")
    ax.axis("off")
plt.suptitle("نمونه‌های ارقام")
plt.tight_layout(); plt.show()

# تقسیم
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42
)

# مقایسه‌ی مدل‌ها
models = {
    "Logistic": LogisticRegression(max_iter=2000, random_state=42),
    "Random Forest": RandomForestClassifier(n_estimators=300, random_state=42, n_jobs=-1),
    "MLP (128,64)": MLPClassifier(hidden_layer_sizes=(128, 64), max_iter=500,
                                  random_state=42),
}

print(f"\n{'مدل':>16} {'test acc':>12}")
best_acc, best_name, best_pred = 0, None, None

for name, model in models.items():
    model.fit(X_train, y_train)
    pred = model.predict(X_test)
    acc = accuracy_score(y_test, pred)
    print(f"{name:>16} {acc:>12.4f}")
    if acc > best_acc:
        best_acc, best_name, best_pred = acc, name, pred

print(f"\nبهترین: {best_name} ({best_acc:.4f})")

# گزارش کامل
print("\n" + classification_report(y_test, best_pred, digits=3))

# ماتریس درهم‌ریختگی
cm = confusion_matrix(y_test, best_pred)
plt.figure(figsize=(7, 6))
plt.imshow(cm, cmap="Blues")
plt.colorbar()
plt.xlabel("پیش‌بینی"); plt.ylabel("واقعیت")
plt.title(f"ماتریس درهم‌ریختگی — {best_name}")
plt.xticks(range(10)); plt.yticks(range(10))
for i in range(10):
    for j in range(10):
        if cm[i, j] > 0:
            plt.text(j, i, str(cm[i, j]), ha="center", va="center",
                     color="white" if cm[i, j] > cm.max() / 2 else "black", fontsize=8)
plt.tight_layout(); plt.show()

# ============ تحلیل خطا: نگاه به نمونه‌های اشتباه ============
errors = np.where(best_pred != y_test)[0]
print(f"\nتعداد خطا: {len(errors)} از {len(y_test)}")

if len(errors) > 0:
    fig, axes = plt.subplots(1, min(6, len(errors)), figsize=(12, 2.5))
    if len(errors) == 1:
        axes = [axes]
    for ax, idx in zip(np.atleast_1d(axes), errors[:6]):
        ax.imshow(X_test[idx].reshape(8, 8), cmap="gray_r")
        ax.set_title(f"واقعی: {y_test[idx]}\nپیش‌بینی: {best_pred[idx]}", fontsize=9)
        ax.axis("off")
    plt.suptitle("نمونه‌های اشتباه — چرا؟")
    plt.tight_layout(); plt.show()

    print("\nالگوهای رایج خطا:")
    print("- ۳ و ۸ (شکل مشابه در ۸×۸)")
    print("- ۴ و ۹ (بالای باز)")
    print("- ۱ و ۷ (خط عمودی مشابه)")
```

**نسخه‌ی PyTorch CNN (برای اجرای واقعی):**

```python
"""
نسخه‌ی CNN با PyTorch — نیاز به: pip install torch torchvision
دانلود MNIST در اجرای اول انجام می‌شود (~۱۱ مگابایت)
"""
try:
    import torch
    import torch.nn as nn
    from torch.utils.data import DataLoader
    from torchvision import datasets, transforms

    device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

    transform_train = transforms.Compose([
        transforms.RandomAffine(degrees=10, translate=(0.1, 0.1)),
        transforms.ToTensor(),
        transforms.Normalize((0.1307,), (0.3081,)),
    ])
    transform_test = transforms.Compose([
        transforms.ToTensor(),
        transforms.Normalize((0.1307,), (0.3081,)),
    ])

    train_ds = datasets.MNIST("./data", train=True, download=True, transform=transform_train)
    test_ds = datasets.MNIST("./data", train=False, download=True, transform=transform_test)

    train_loader = DataLoader(train_ds, batch_size=128, shuffle=True, num_workers=0)
    test_loader = DataLoader(test_ds, batch_size=512, num_workers=0)

    class CNN(nn.Module):
        def __init__(self):
            super().__init__()
            self.features = nn.Sequential(
                nn.Conv2d(1, 32, 3, padding=1), nn.BatchNorm2d(32), nn.ReLU(),
                nn.Conv2d(32, 32, 3, padding=1), nn.ReLU(),
                nn.MaxPool2d(2),                                    # 28 → 14
                nn.Conv2d(32, 64, 3, padding=1), nn.BatchNorm2d(64), nn.ReLU(),
                nn.MaxPool2d(2),                                    # 14 → 7
                nn.AdaptiveAvgPool2d(1),
            )
            self.head = nn.Sequential(nn.Flatten(), nn.Dropout(0.3), nn.Linear(64, 10))

        def forward(self, x):
            return self.head(self.features(x))

    model = CNN().to(device)
    criterion = nn.CrossEntropyLoss()
    optimizer = torch.optim.AdamW(model.parameters(), lr=1e-3, weight_decay=1e-4)

    print(f"دستگاه: {device} | پارامترها: {sum(p.numel() for p in model.parameters()):,}")

    for epoch in range(3):
        model.train()
        for xb, yb in train_loader:
            xb, yb = xb.to(device), yb.to(device)
            optimizer.zero_grad()
            loss = criterion(model(xb), yb)
            loss.backward()
            optimizer.step()

        model.eval()
        correct = total = 0
        with torch.no_grad():
            for xb, yb in test_loader:
                xb, yb = xb.to(device), yb.to(device)
                correct += (model(xb).argmax(1) == yb).sum().item()
                total += len(yb)
        print(f"epoch {epoch+1}: test acc = {correct/total:.4f}")

except ImportError:
    print("برای نسخه‌ی CNN نصب کن: pip install torch torchvision")
except Exception as e:
    print(f"خطا (احتمالاً دانلود): {type(e).__name__}")
```

### مقایسه و دیدگاه‌های مختلف

| مدل | دقت (۸×۸ sklearn) | دقت (۲۸×۲۸ MNIST) |
|---|---|---|
| Logistic | ~۰.۹۶ | ~۰.۹۲ |
| Random Forest | ~۰.۹۷ | ~۰.۹۷ |
| MLP | ~۰.۹۸ | ~۰.۹۸ |
| CNN | — | ~۰.۹۹+ |

**درس:** CNN برای تصویر بهتر است، اما روی داده‌ی کوچک و ساده، MLP هم نزدیک می‌شود. **ساختار داده مهم است، نه فقط مدل.**

### اشتباهات رایج

**۱. نرمال‌سازی نکردن تصویر.** آموزش ناپایدار.
**۲. augmentation در test.** ارزیابی ناپایدار.
**۳. نگاه نکردن به نمونه‌های اشتباه.** مهم‌ترین بخش تحلیل.
**۴. آموزش از صفر با داده‌ی کم.** transfer learning.
**۵. فراموش کردن `model.eval()`.** BatchNorm رفتار غلط.
**۶. نادیده گرفتن عدم توازن کلاس.** در MNIST کم است، در داده‌ی واقعی زیاد.

### نکات مهم و ترفندها

- **همیشه نمونه‌های اشتباه را ببین.** الگوی خطا، ارزشمندتر از عدد دقت است.
- **augmentation را متناسب انتخاب کن.** چرخش ۱۸۰ درجه برای رقم، ۶ را ۹ می‌کند.
- **`AdaptiveAvgPool2d(1)`** پارامترها را کم می‌کند.
- **BatchNorm در CNN تقریباً واجب است.**
- **میان‌بر:** تصویر → نرمال‌سازی → CNN → ارزیابی → تحلیل خطا.

### خلاصه فصل

- تشخیص ارقام، مسیر استاندارد بینایی ماشین است.
- CNN برای تصویر بهترین انتخاب است.
- augmentation با داده‌ی کم حیاتی است.
- ماتریس درهم‌ریختگی، الگوی خطا را نشان می‌دهد.
- نگاه کردن به نمونه‌های اشتباه، بخشی از کار است.

**تمرین:**
1. `RandomAffine` را حذف کن و تفاوت دقت را ببین.
2. یک لایه‌ی Conv اضافه کن.
3. مدل را روی Fashion-MNIST (سخت‌تر) امتحان کن.

---

## فصل ۹۰: پروژه ۴: تحلیل احساسات متن

### مقدمه

این پروژه، **NLP عملی** است: تشخیص مثبت/منفی بودن متن. اما به جای آموزش از صفر، از **embedding از پیش آموزش‌دیده** استفاده می‌کنیم — روش استاندارد صنعت.

مهارت‌های جدید: پردازش متن، TF-IDF، embedding، ارزیابی NLP.

### مفاهیم اساسی

**۱. سه رویکرد به متن:**

| رویکرد | روش | داده‌ی لازم | کیفیت |
|---|---|---|---|
| **Bag of Words / TF-IDF** | شمارش کلمات | کم | خوب برای موضوع |
| **Embedding + مدل ساده** | بردار معنادار + Logistic | کم | خوب |
| **Fine-tune ترانسفورمر** | BERT فارسی | متوسط | عالی |

**۲. TF-IDF — چرا بهتر از شمارش خام؟**

```
TF = فراوانی کلمه در سند
IDF = log(تعداد اسناد / تعداد اسناد حاوی کلمه)
TF-IDF = TF × IDF
```
کلمات پرتکرار در همه‌ی اسناد («و»، «است») وزن کم می‌گیرند.

**۳. چالش‌های فارسی:**
- **نیم‌فاصله:** «می‌رود» vs «میرود».
- **ی/ک عربی:** «كتاب» vs «کتاب».
- **اعراب:** «کِتاب».
- **توکن‌سازی:** بدون فاصله‌ی صریح، سخت‌تر.

**۴. پیش‌پردازش حداقلی برای فارسی:**

```python
def normalize_persian(text):
    text = text.replace("ي", "ی").replace("ك", "ک")
    text = re.sub(r"[\u064B-\u0652]", "", text)      # اعراب
    text = text.replace("\u200c", " ")                # نیم‌فاصله
    text = re.sub(r"[^\w\s]", " ", text)              # نشانه‌ها
    return re.sub(r"\s+", " ", text).strip()
```

### مثال‌ها و نمونه‌های عملی

**کد کامل پروژه:**

```python
"""
پروژه ۴: تحلیل احساسات
دو نسخه: (الف) TF-IDF + Logistic (همیشه اجرا می‌شود)
         (ب) Embedding چندزبانه (نیاز به دانلود مدل)
"""
import re
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.pipeline import Pipeline
from sklearn.metrics import classification_report, confusion_matrix
from sklearn.naive_bayes import MultinomialNB
from sklearn.svm import LinearSVC

# ============ ۱. داده‌ی نمونه (فارسی) ============
texts = [
    "این فیلم فوق‌العاده بود، واقعاً لذت بردم",
    "خیلی بد بود، وقت تلف کردم",
    "بازیگرها عالی بودند اما فیلمنامه ضعیف",
    "بهترین فیلمی که دیدم",
    "افتضاح بود، نیمه‌کاره رها کردم",
    "داستان جذاب و کارگردانی خوب",
    "حوصله‌ام سر رفت",
    "شاهکار بود، حتماً ببینید",
    "پول و وقت هدر رفت",
    "قشنگ بود ولی چیز خاصی نداشت",
    "عالی، عالی، عالی",
    "ضعیف‌ترین فیلم سال",
    "دوستش داشتم، بازی‌ها طبیعی بود",
    "فیلمنامه پر از کلیشه بود",
    "لذت بردم از دیدنش",
    "خیلی خسته‌کننده بود",
    "یکی از بهترین‌ها در ژانر خودش",
    "ناامید شدم",
    "ارزش دیدن داشت",
    "بد نبود ولی انتظار بیشتری داشتم",
]
labels = [1, 0, 1, 1, 0, 1, 0, 1, 0, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0]

df = pd.DataFrame({"text": texts, "sentiment": labels})
print(f"تعداد نمونه: {len(df)} | مثبت: {df.sentiment.sum()} | منفی: {(1-df.sentiment).sum()}")

# ============ ۲. پیش‌پردازش فارسی ============
def normalize_persian(text):
    text = text.replace("ي", "ی").replace("ك", "ک")
    text = re.sub(r"[\u064B-\u0652]", "", text)     # حذف اعراب
    text = text.replace("\u200c", " ")               # نیم‌فاصله → فاصله
    text = re.sub(r"[^\w\s]", " ", text)             # حذف نشانه‌ها
    return re.sub(r"\s+", " ", text).strip()

df["clean"] = df["text"].apply(normalize_persian)
print("\nنمونه‌ی پاکسازی:")
for i in range(2):
    print(f"  قبل: {df['text'][i]}")
    print(f"  بعد: {df['clean'][i]}")

# ============ ۳. تقسیم ============
X_train, X_test, y_train, y_test = train_test_split(
    df["clean"], df["sentiment"], test_size=0.25, stratify=df["sentiment"],
    random_state=42
)

# ============ ۴. مدل‌ها ============
models = {
    "TF-IDF + Logistic": Pipeline([
        ("tfidf", TfidfVectorizer(ngram_range=(1, 2), min_df=1)),
        ("clf", LogisticRegression(max_iter=1000, C=1.0)),
    ]),
    "TF-IDF + NaiveBayes": Pipeline([
        ("tfidf", TfidfVectorizer(ngram_range=(1, 2))),
        ("clf", MultinomialNB()),
    ]),
    "TF-IDF + LinearSVC": Pipeline([
        ("tfidf", TfidfVectorizer(ngram_range=(1, 2))),
        ("clf", LinearSVC(C=1.0, max_iter=5000)),
    ]),
}

print(f"\n{'مدل':>22} {'CV acc':>10}")
for name, model in models.items():
    # با داده‌ی کوچک، CV=3
    scores = cross_val_score(model, X_train, y_train, cv=3)
    print(f"{name:>22} {scores.mean():>10.3f}")

# ============ ۵. آموزش نهایی ============
best = models["TF-IDF + Logistic"].fit(X_train, y_train)
pred = best.predict(X_test)

print("\n" + classification_report(y_test, pred,
                                   target_names=["منفی", "مثبت"], digits=3))
print("ماتریس درهم‌ریختگی:")
print(confusion_matrix(y_test, pred))

# ============ ۶. تحلیل: مهم‌ترین کلمات ============
tfidf = best.named_steps["tfidf"]
clf = best.named_steps["clf"]
feature_names = tfidf.get_feature_names_out()

coefs = pd.DataFrame({
    "word": feature_names,
    "weight": clf.coef_[0],
}).sort_values("weight")

print("\n--- کلمات منفی‌ترین ---")
print(coefs.head(5).to_string(index=False))
print("\n--- کلمات مثبت‌ترین ---")
print(coefs.tail(5).to_string(index=False))

# ============ ۷. پیش‌بینی متن جدید ============
new_texts = [
    "فیلم خوبی بود، پیشنهاد می‌کنم",
    "کاملاً بی‌ارزش بود",
    "نمی‌دانم، نظری ندارم",
]
new_clean = [normalize_persian(t) for t in new_texts]
probs = best.predict_proba(new_clean)[:, 1]

print("\n--- پیش‌بینی متن جدید ---")
for t, p in zip(new_texts, probs):
    label = "مثبت" if p > 0.5 else "منفی"
    print(f"{t!r:45s} → {label} (p={p:.2f})")

print("\n⚠️ توجه: مدل با ۲۰ نمونه آموزش دیده — فقط نمایش مکانیزم است.")
print("برای نتیجه‌ی واقعی، هزاران نمونه‌ی برچسب‌خورده لازم است.")
```

**نسخه‌ی embedding (دقیق‌تر):**

```python
"""
نسخه‌ی embedding چندزبانه — نیاز به: pip install sentence-transformers
"""
try:
    from sentence_transformers import SentenceTransformer
    from sklearn.linear_model import LogisticRegression
    from sklearn.model_selection import cross_val_score
    import numpy as np

    encoder = SentenceTransformer("paraphrase-multilingual-MiniLM-L12-v2")

    # داده‌ی بیشتر برای نتیجه‌ی معنادار
    texts_big = texts * 10
    labels_big = labels * 10

    X_emb = encoder.encode(texts_big, show_progress_bar=False)

    clf = LogisticRegression(max_iter=1000)
    scores = cross_val_score(clf, X_emb, labels_big, cv=5)
    print(f"دقت با embedding: {scores.mean():.3f} ± {scores.std():.3f}")

    # شباهت معنایی
    from sklearn.metrics.pairwise import cosine_similarity
    pairs = [
        ("فیلم عالی بود", "فیلم فوق‌العاده بود"),
        ("فیلم عالی بود", "فیلم بد بود"),
    ]
    for a, b in pairs:
        sim = cosine_similarity(encoder.encode([a]), encoder.encode([b]))[0][0]
        print(f"شباهت {a!r} و {b!r}: {sim:.3f}")
except ImportError:
    print("نصب کن: pip install sentence-transformers")
```

### مقایسه و دیدگاه‌های مختلف

| رویکرد | داده‌ی لازم | دقت | سرعت | تفسیر |
|---|---|---|---|---|
| TF-IDF + NB | کم | متوسط | خیلی سریع | بالا |
| TF-IDF + Logistic | کم | خوب | سریع | بالا |
| Embedding + Logistic | کم | خوب | متوسط | متوسط |
| Fine-tune BERT | متوسط | عالی | کند | پایین |
| LLM + prompt | صفر | متغیر | کند | پایین |

**درس:** برای شروع، **TF-IDF + Logistic** بهترین نسبت کیفیت/هزینه است. embedding وقتی مفید است که داده کم و متن پیچیده باشد.

### اشتباهات رایج

**۱. نبود پیش‌پردازش فارسی.** «می‌رود» و «میرود» دو توکن متفاوت.
**۲. حذف stopwords بدون بررسی.** «نه» و «نیست» سیگنال احساس‌اند!
**۳. ارزیابی با داده‌ی کم.** ۲۰ نمونه، هیچ چیزی را ثابت نمی‌کند.
**۴. `ngram_range=(1,1)` همیشه.** bigram برای احساسات بهتر است («خیلی خوب» vs «خوب»).
**۵. نادیده گرفتن نفی.** «بد نبود» مثبت است.
**۶. تفسیر ضرایب به‌عنوان علیت.** وزن بالا یعنی همبستگی.

### نکات مهم و ترفندها

- **`ngram_range=(1, 2)` را امتحان کن.** برای احساسات، معمولاً بهتر.
- **stopwords فارسی را با احتیاط حذف کن.** کلمات منفی مهم‌اند.
- **`min_df=1` برای داده‌ی کم، `min_df=2` برای داده‌ی زیاد.**
- **embedding برای متن کوتاه و پیچیده بهتر است.**
- **میان‌بر:** متن → نرمال‌سازی → TF-IDF → Logistic → ارزیابی.

### خلاصه فصل

- TF-IDF + Logistic، baseline قوی NLP است.
- پیش‌پردازش فارسی (ی/ک، نیم‌فاصله، اعراب) واجب است.
- bigram برای احساسات کمک می‌کند.
- embedding برای معنای پیچیده‌تر.
- با داده‌ی کم، نتیجه قابل اعتماد نیست.

**تمرین:**
1. `ngram_range` را تغییر بده و مقایسه کن.
2. stopwords را اضافه کن و ببین چه اتفاقی می‌افتد.
3. یک دیتاست واقعی فارسی (مثل SnappFood) پیدا کن و اجرا کن.

---

## فصل ۹۱: پروژه ۵: دستیار ساده با API مدل زبانی

### مقدمه

این پروژه، **مهندسی با LLM** است، نه آموزش مدل. هدف: ساخت یک دستیار که به سؤالات **بر اساس اسناد خودت** پاسخ می‌دهد (RAG ساده).

مهارت‌های جدید: طراحی prompt، مدیریت context، اعتبارسنجی خروجی، مدیریت خطا.

### مفاهیم اساسی

**۱. معماری RAG ساده:**

```
سؤال
  → بازیابی اسناد مرتبط (جست‌وجوی متنی/برداری)
  → ساخت prompt با اسناد
  → فراخوانی LLM
  → اعتبارسنجی و بازگشت پاسخ
```

**۲. سه لایه‌ی دفاعی:**

1. **Prompt:** «فقط از اسناد پاسخ بده.»
2. **اعتبارسنجی:** آیا پاسخ در اسناد هست؟
3. **Fallback:** «نمی‌دانم» به جای حدس.

**۳. مدیریت خطا:**
- Timeout
- Rate limit
- پاسخ خالی
- JSON نامعتبر

**۴. نکته‌ی مهم:** این پروژه **بدون API key** هم اجرا می‌شود — با یک «مدل ساختگی» که مکانیزم را نشان می‌دهد.

### مثال‌ها و نمونه‌های عملی

**کد کامل پروژه:**

```python
"""
پروژه ۵: دستیار ساده با RAG
اجرا بدون API key (حالت mock) یا با OpenAI/متغیر محیطی
"""
import os
import json
import re
from dataclasses import dataclass, field
from typing import List, Optional

# ============ ۱. پایگاه دانش ============
@dataclass
class Document:
    id: str
    title: str
    content: str

KB = [
    Document("1", "هوش مصنوعی",
             "هوش مصنوعی شاخه‌ای از علوم کامپیوتر است که سیستم‌هایی می‌سازد "
             "که وظایفی مانند یادگیری، استدلال و تصمیم‌گیری انجام می‌دهند."),
    Document("2", "یادگیری ماشین",
             "یادگیری ماشین زیرشاخه‌ای از هوش مصنوعی است که در آن مدل‌ها "
             "به جای قاعده‌نویسی دستی، از داده الگو یاد می‌گیرند."),
    Document("3", "یادگیری عمیق",
             "یادگیری عمیق زیرشاخه‌ای از یادگیری ماشین است که از شبکه‌های "
             "عصبی چندلایه استفاده می‌کند و برای داده‌های خام مثل تصویر و متن مناسب است."),
    Document("4", "مدل زبانی بزرگ",
             "مدل زبانی بزرگ (LLM) مدلی است که با پیش‌بینی توکن بعدی آموزش می‌بیند "
             "و می‌تواند متن تولید کند، ترجمه کند و به سؤال پاسخ دهد."),
    Document("5", "بیش‌برازش",
             "بیش‌برازش زمانی رخ می‌دهد که مدل داده‌ی آموزش را حفظ می‌کند "
             "و روی داده‌ی جدید ضعیف عمل می‌کند. نشانه‌ی آن شکاف بین دقت آموزش و آزمون است."),
]

# ============ ۲. بازیابی (Retrieval) ============
def tokenize(text: str) -> set:
    text = re.sub(r"[^\w\s]", " ", text.lower())
    return set(text.split())

def retrieve(query: str, kb: List[Document], k: int = 2) -> List[Document]:
    """بازیابی ساده با هم‌پوشانی کلمات (baseline قابل فهم)."""
    q_tokens = tokenize(query)
    scored = []
    for doc in kb:
        d_tokens = tokenize(doc.title + " " + doc.content)
        overlap = len(q_tokens & d_tokens)
        # نرمال‌سازی با طول سند
        score = overlap / (len(d_tokens) ** 0.5 + 1e-9)
        if overlap > 0:
            scored.append((score, doc))
    scored.sort(key=lambda x: -x[0])
    return [doc for _, doc in scored[:k]]

# ============ ۳. ساخت prompt ============
SYSTEM_PROMPT = """تو یک دستیار پاسخ‌گو هستی که فقط بر اساس اسناد ارائه‌شده پاسخ می‌دهد.

قواعد سخت:
۱. فقط از محتوای اسناد استفاده کن. دانش بیرونی اضافه نکن.
۲. اگر پاسخ در اسناد نبود، دقیقاً بنویس: «در اسناد موجود نیست.»
۳. بعد از هر ادعا، شناسه‌ی سند را در [] بیاور: [1]
۴. پاسخ کوتاه و دقیق باشد.
۵. حدس نزن."""

def build_prompt(query: str, docs: List[Document]) -> str:
    context = "\n\n".join(
        f"[سند {d.id}] {d.title}\n{d.content}" for d in docs
    )
    return f"""{SYSTEM_PROMPT}

=== اسناد ===
{context}

=== سؤال ===
{query}

=== پاسخ ==="""

# ============ ۴. فراخوانی مدل ============
def call_llm_mock(prompt: str) -> str:
    """
    مدل ساختگی: پاسخ را از اسناد استخراج می‌کند.
    برای اجرا بدون API key — منطق RAG را نشان می‌دهد.
    """
    # استخراج اسناد از prompt
    docs_part = prompt.split("=== اسناد ===")[1].split("=== سؤال ===")[0]
    question = prompt.split("=== سؤال ===")[1].split("=== پاسخ ===")[0].strip()

    entries = re.findall(r"\[سند (\d+)\] ([^\n]+)\n([^\[]+)", docs_part)
    if not entries:
        return "در اسناد موجود نیست."

    q_tokens = tokenize(question)
    best, best_score = None, 0
    for doc_id, title, content in entries:
        content_tokens = tokenize(content)
        score = len(q_tokens & content_tokens)
        if score > best_score:
            best_score, best = score, (doc_id, title, content.strip())

    if best is None:
        return "در اسناد موجود نیست."

    doc_id, title, content = best
    return f"{content} [{doc_id}]"

def call_llm_openai(prompt: str) -> str:
    """نسخه‌ی واقعی با OpenAI — نیاز به OPENAI_API_KEY."""
    from openai import OpenAI
    client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])
    resp = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
        temperature=0.0,
        max_tokens=400,
    )
    return resp.choices[0].message.content.strip()

def call_llm(prompt: str) -> str:
    if os.environ.get("OPENAI_API_KEY"):
        try:
            return call_llm_openai(prompt)
        except Exception as e:
            print(f"⚠️ خطای API ({type(e).__name__}) — استفاده از حالت mock")
    return call_llm_mock(prompt)

# ============ ۵. اعتبارسنجی ============
def validate_answer(answer: str, docs: List[Document]) -> dict:
    """بررسی این‌که پاسخ به اسناد متکی است."""
    doc_ids = {d.id for d in docs}
    cited = set(re.findall(r"\[(\d+)\]", answer))

    checks = {
        "has_answer": bool(answer.strip()),
        "not_empty": answer.strip() not in ("", "در اسناد موجود نیست."),
        "cites_valid_doc": bool(cited & doc_ids) if cited else False,
        "no_invalid_citation": cited.issubset(doc_ids) if cited else True,
    }
    checks["passed"] = checks["has_answer"] and (
        not checks["not_empty"] or checks["cites_valid_doc"]
    )
    return checks

# ============ ۶. خط لوله‌ی کامل ============
@dataclass
class Assistant:
    kb: List[Document] = field(default_factory=lambda: KB)
    history: List[dict] = field(default_factory=list)

    def ask(self, question: str, k: int = 2, verbose: bool = True) -> dict:
        docs = retrieve(question, self.kb, k=k)

        if not docs:
            return {"answer": "در اسناد موجود نیست.", "docs": [], "valid": True}

        prompt = build_prompt(question, docs)
        answer = call_llm(prompt)
        checks = validate_answer(answer, docs)

        result = {
            "question": question,
            "answer": answer,
            "docs": [d.id for d in docs],
            "valid": checks["passed"],
            "checks": checks,
        }
        self.history.append(result)

        if verbose:
            print(f"\n❓ {question}")
            print(f"📚 اسناد بازیابی‌شده: {[d.id for d in docs]}")
            print(f"💬 {answer}")
            print(f"✅ معتبر: {checks['passed']}")

        return result

# ============ ۷. اجرا ============
if __name__ == "__main__":
    assistant = Assistant()

    questions = [
        "تفاوت هوش مصنوعی و یادگیری ماشین چیست؟",
        "یادگیری عمیق چه زمانی مناسب است؟",
        "بیش‌برازش چه نشانه‌ای دارد؟",
        "قیمت دلار امروز چقدر است؟",        # خارج از پایگاه دانش
        "LLM چگونه آموزش می‌بیند؟",
    ]

    for q in questions:
        assistant.ask(q)

    # خلاصه
    print("\n" + "=" * 60)
    print(f"تعداد سؤال: {len(assistant.history)}")
    valid = sum(1 for r in assistant.history if r["valid"])
    print(f"پاسخ‌های معتبر: {valid}/{len(assistant.history)}")
    out_of_scope = sum(1 for r in assistant.history
                       if r["answer"] == "در اسناد موجود نیست.")
    print(f"رد صحیح سؤال خارج از دامنه: {out_of_scope}")
    print("\nنکته: رد کردن سؤال خارج از دامنه، یک موفقیت است، نه شکست.")
```

**خروجی مورد انتظار:**

```
❓ تفاوت هوش مصنوعی و یادگیری ماشین چیست؟
📚 اسناد بازیابی‌شده: ['2', '1']
💬 یادگیری ماشین زیرشاخه‌ای از هوش مصنوعی است... [2]
✅ معتبر: True

❓ قیمت دلار امروز چقدر است؟
📚 اسناد بازیابی‌شده: []
💬 در اسناد موجود نیست.
✅ معتبر: True
```

### مقایسه و دیدگاه‌های مختلف

| روش بازیابی | دقت | سرعت | پیچیدگی |
|---|---|---|---|
| کلمه‌ای (TF-IDF) | متوسط | سریع | کم |
| برداری (embedding) | خوب | متوسط | متوسط |
| ترکیبی (hybrid) | بهترین | کندتر | زیاد |
| Reranker | عالی | کند | زیاد |

**دو دیدگاه:**
- **«RAG جای fine-tune را می‌گیرد.»** برای دانش، بله. برای سبک و رفتار، نه.
- **«RAG مشکل hallucination را حل می‌کند.»** کاهش می‌دهد، نه حذف. مدل می‌تواند از سند هم اشتباه نقل کند.

### اشتباهات رایج

**۱. نبود محدودیت در prompt.** مدل دانش بیرونی اضافه می‌کند.
**۲. نبود اعتبارسنجی.** پاسخ غلط بدون استناد برمی‌گردد.
**۳. بازیابی ضعیف.** اگر سند درست پیدا نشود، پاسخ درست ممکن نیست.
**۴. context بیش از حد.** اسناد زیاد، مدل را گیج می‌کند.
**۵. نبود fallback.** خطای API، تجربه‌ی کاربر را می‌شکند.
**۶. اعتماد به خروجی بدون بازبینی.** در محصول واقعی، خطرناک.

### نکات مهم و ترفندها

- **«اگر نمی‌دانی بگو نمی‌دانم» را در prompt بگذار.** مهم‌ترین قاعده.
- **استناد اجباری** را در prompt و اعتبارسنجی پیاده کن.
- **`temperature=0`** برای پاسخ‌های واقعیتی.
- **برای بازیابی واقعی، embedding + vector DB** (Chroma، FAISS).
- **همیشه مسیر fallback داشته باش.**
- **میان‌بر:** بازیابی → prompt محدود → فراخوانی → اعتبارسنجی → پاسخ.

### خلاصه فصل

- RAG، LLM را به دانش خودت متصل می‌کند.
- سه لایه‌ی دفاعی: prompt، اعتبارسنجی، fallback.
- «نمی‌دانم» پاسخ درست برای سؤال خارج از دامنه است.
- بازیابی خوب، پیش‌نیاز پاسخ خوب است.
- مدیریت خطا، بخشی از طراحی است.

**تمرین:**
1. بازیابی را با embedding جایگزین کن.
2. یک سؤال چندبخشی اضافه کن (نیاز به دو سند).
3. یک تست خودکار بنویس که ۲۰ سؤال و پاسخ مورد انتظار را بررسی کند.

---

## جمع‌بندی بخش پروژه‌ها

| پروژه | چه یاد گرفتی | چه چیزی را تمرین کن |
|---|---|---|
| ۱. Iris | چرخه‌ی کامل ML | baseline، CV، گزارش |
| ۲. قیمت خانه | رگرسیون و مهندسی ویژگی | Pipeline، تحلیل باقیمانده |
| ۳. ارقام | بینایی ماشین | CNN، augmentation، تحلیل خطا |
| ۴. احساسات | NLP | TF-IDF، embedding، پیش‌پردازش فارسی |
| ۵. دستیار | مهندسی LLM | RAG، prompt، اعتبارسنجی |

**قدم بعدی:** یکی از این پنج را انتخاب کن، داده‌ی خودت را جایگزین کن، و **از صفر** بازنویسی کن. این، تنها راه تبدیل دانش به مهارت است.

---

## منابع و مراجع

**دیتاست‌ها**
- UCI Machine Learning Repository
- Kaggle Datasets
- Hugging Face Datasets
- scikit-learn `datasets` module

**پروژه‌های پیشنهادی بیشتر**
- Kaggle — *Titanic*, *House Prices*, *Digit Recognizer* (با kernelهای آموزشی)
- fast.ai — *Practical Deep Learning* (پروژه‌محور)

**ابزارها**
- `ydata-profiling` — گزارش خودکار EDA
- `Optuna` — تنظیم فراپارامتر
- `MLflow` — ردیابی آزمایش‌ها
- `Streamlit` — ساخت رابط سریع برای مدل

---

**بعدی:** [`20-Roadmap-Troubleshooting.md`](./20-Roadmap-Troubleshooting.md) — فصل ۹۲: عیب‌یابی خطاهای درک مفاهیم
