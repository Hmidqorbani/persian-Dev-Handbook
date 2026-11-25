# بخش ۰۷: تحلیل اکتشافی و تجسم داده

**مدت یادگیری تقریبی**: ۱.۵ ساعت
**سطح سختی**: مبتدی
**پیش‌نیاز**: فایل ۰۶ (Pandas)

---

## فهرست محتوا

- [فصل ۳۰: EDA چیست و چرا ۷۰٪ کار واقعی است](#فصل-۳۰-eda-چیست-و-چرا-۷۰-کار-واقعی-است)
- [فصل ۳۱: تجسم با Matplotlib](#فصل-۳۱-تجسم-با-matplotlib)
- [فصل ۳۲: تجسم آماری با Seaborn](#فصل-۳۲-تجسم-آماری-با-seaborn)
- [فصل ۳۳: از نمودار تا بینش: داستان‌سرایی داده](#فصل-۳۳-از-نمودار-تا-بینش-داستانسرایی-داده)

---

## فصل ۳۰: EDA چیست و چرا ۷۰٪ کار واقعی است

### مقدمه

EDA (Exploratory Data Analysis — تحلیل اکتشافی داده) یعنی «قبل از ساختن مدل، داده را بشناس». نه با نگاه کردن به جدول، بلکه با **پرسیدن سؤال و کشیدن نمودار**.

چرا مهم است؟ چون مدل‌ها بی‌رحمانه‌اند: اگر داده اشکال داشته باشد، مدل **بی‌صدا** آن اشکال را یاد می‌گیرد و بعد با اطمینان کامل اشتباه می‌کند. EDA تنها فرصت توست برای این‌که قبل از مدل‌سازی، اشکال را ببینی.

### مفاهیم اساسی

**۱. چهار سؤال بنیادی EDA:**

| سؤال | ابزار | چه چیزی پیدا می‌کنی |
|---|---|---|
| داده چه شکلی است؟ | `info`, `shape`, `head` | نوع ستون، حجم، ساختار |
| مقادیر چطور پخش شده‌اند؟ | `describe`, هیستوگرام | توزیع، پرت، چولگی |
| چه چیزی گم است؟ | `isna()` | درصد گمشدگی، الگوی آن |
| چه چیزی با هدف مرتبط است؟ | همبستگی، نمودار گروهی | ویژگی‌های مفید |

**۲. سه سطح EDA:**

- **سطح ۱ — تک‌متغیره (Univariate):** هر ستون را جدا ببین. توزیع، پرت، گمشده.
- **سطح ۲ — دومتغیره (Bivariate):** رابطه‌ی هر ویژگی با هدف. `x` در برابر `y`.
- **سطح ۳ — چندمتغیره (Multivariate):** همبستگی بین ویژگی‌ها، تعامل‌ها، گروه‌بندی.

**۳. نمودار درست برای سؤال درست.**

| نوع داده | هدف | نمودار |
|---|---|---|
| عددی پیوسته | توزیع | هیستوگرام، KDE، boxplot |
| عددی پیوسته | رابطه با عددی | scatter |
| عددی | رابطه با دسته‌ای | boxplot، violin |
| دسته‌ای | فراوانی | bar chart |
| دسته‌ای | رابطه با دسته‌ای | جدول توافقی، heatmap |
| زمانی | روند | line chart |
| ماتریس عددی | همبستگی | heatmap |

**۴. تله‌های رایج EDA که باید بشناسی:**

- **پارادوکس سیمپسون:** رابطه‌ای که در کل داده مثبت است، در هر زیرگروه منفی می‌شود (یا برعکس). همیشه داده را به تفکیک گروه هم ببین.
- **چولگی (Skewness):** میانگین گمراه‌کننده می‌شود.
- **همبستگی جعلی:** دو متغیر بی‌ربط که تصادفاً همبسته‌اند (خصوصاً در نمونه‌های کوچک).
- **نشت داده در EDA:** اگر از کل داده برای محاسبه‌ی آمار استفاده کنی و بعد تقسیم کنی، ارزیابی‌ات خوش‌بینانه می‌شود.

**۵. خروجی EDA چیست؟** نه یک نمودار، بلکه یک **فهرست تصمیم**:
- کدام ستون حذف شود؟
- کدام ستون گمشده‌اش پر شود و با چه روشی؟
- کدام ویژگی جدید ساخته شود؟
- کدام مدل مناسب است؟
- چه معیاری برای ارزیابی درست است؟

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — چک‌لیست EDA در ۱۰ خط:**

```python
import pandas as pd

def quick_eda(df, target=None):
    """گزارش سریع اکتشافی — اولین چیزی که روی هر دیتاست می‌زنی."""
    print(f"شکل: {df.shape[0]:,} ردیف × {df.shape[1]} ستون\n")

    print("=== انواع و گمشدگی ===")
    summary = pd.DataFrame({
        "dtype": df.dtypes.astype(str),
        "non_null": df.notna().sum(),
        "missing_%": (df.isna().mean() * 100).round(1),
        "unique": df.nunique(),
    })
    print(summary)

    print("\n=== ستون‌های عددی ===")
    print(df.describe().T.round(2))

    print("\n=== ستون‌های دسته‌ای (۵ مقدار پرتکرار) ===")
    for col in df.select_dtypes(include="object").columns[:5]:
        print(f"\n{col}:")
        print(df[col].value_counts().head())

    if target:
        print(f"\n=== توازن هدف ({target}) ===")
        print(df[target].value_counts(normalize=True).round(3))

    print(f"\n=== تکراری: {df.duplicated().sum()} ردیف ===")
```

**مثال ۲ — پارادوکس سیمپسون در عمل:**

```python
import pandas as pd

# داده‌ی مصنوعی: در کل، درمان بهتر به‌نظر می‌رسد
# اما در هر گروه سنی، درمان بدتر است!
df = pd.DataFrame({
    "age_group": ["جوان"] * 40 + ["مسن"] * 60,
    "treatment": ["A"] * 10 + ["B"] * 30 + ["A"] * 50 + ["B"] * 10,
    "recovered": ([1]*9 + [0]*1) + ([1]*18 + [0]*12) + ([1]*25 + [0]*25) + ([1]*2 + [0]*8),
})

overall = df.groupby("treatment")["recovered"].mean()
by_group = df.groupby(["age_group", "treatment"])["recovered"].mean()

print("کل:\n", overall.round(3))
print("\nبه تفکیک گروه سنی:\n", by_group.round(3))
```

خروجی نشان می‌دهد درمان A در کل بهتر است، اما در **هر گروه سنی** بدتر. علت: A بیشتر به جوان‌ها داده شده. **این تله را در داده‌ی واقعی هم زیاد می‌بینی.**

**مثال ۳ — تحلیل دومتغیره با هدف:**

```python
import pandas as pd
import numpy as np

rng = np.random.default_rng(0)
df = pd.DataFrame({
    "age": rng.integers(18, 70, 500),
    "income": rng.normal(15, 5, 500).round(1),
    "churn": rng.integers(0, 2, 500),
})
df.loc[df["churn"] == 1, "income"] -= 3      # ریزش با درآمد کمتر مرتبط است

print(df.groupby("churn")[["age", "income"]].mean().round(2))
print("\nهمبستگی با هدف:")
print(df.corr(numeric_only=True)["churn"].sort_values(ascending=False).round(3))
```

### مقایسه و دیدگاه‌های مختلف

| رویکرد | مزیت | عیب |
|---|---|---|
| **EDA دستی و شهودی** | کشف الگوهای غیرمنتظره | وقت‌بر، غیرقابل تکرار |
| **گزارش خودکار (ydata-profiling)** | سریع، جامع | سطحی، بدون تفسیر |
| **ترکیبی (توصیه‌شده)** | گزارش خودکار برای نمای کلی + بررسی دستی نقاط مشکوک | — |

**یک دیدگاه مهم:** بعضی‌ها EDA را «وقت تلف‌کردن» می‌دانند و مستقیم مدل می‌سازند. تجربه نشان می‌دهد همین افراد بیشترین وقت را در دیباگ مدل‌های شکست‌خورده تلف می‌کنند. **EDA، سرمایه‌گذاری است، نه هزینه.**

### اشتباهات رایج

**۱. نگاه نکردن به داده.** فقط `describe()` زدن و مستقیم مدل ساختن.
**۲. اعتماد به میانگین بدون دیدن توزیع.** داده‌ی دوقله‌ای، میانگینی می‌دهد که هیچ‌کس ندارد.
**۳. نادیده گرفتن گروه‌ها.** پارادوکس سیمپسون.
**۴. EDA روی کل داده و بعد تقسیم.** نشت اطلاعات.
**۵. نمودار بدون محور و واحد.** نموداری که خودت هم بعد از یک هفته نمی‌فهمی.
**۶. نتیجه‌گیری علی از همبستگی.** EDA همبستگی نشان می‌دهد، نه علت.

### نکات مهم و ترفندها

- **همیشه با `shape`, `info()`, `head()`, `describe()` شروع کن.** چهار دستور، ۵ دقیقه، تصویر اولیه.
- **به تفکیک هدف نگاه کن، نه فقط کل.** `df.groupby("target").mean()` طلاست.
- **نمودار بکش، حتی اگر «لازم نیست».** یک scatter می‌تواند چیزی نشان دهد که ۱۰۰ عدد پنهان کرده‌اند.
- **هر یافته را بنویس.** یک فایل `findings.md` بساز: «ستون X، ۴۰٪ گمشده»، «رابطه‌ی Y و Z غیرخطی». این یادداشت‌ها در مدل‌سازی نجاتت می‌دهند.
- **میان‌بر:** EDA = چهار سؤال (شکل، توزیع، گمشده، رابطه) + یک گزارش مکتوب.

### خلاصه فصل

- EDA یعنی شناختن داده قبل از مدل‌سازی؛ سه سطح دارد: تک‌متغیره، دومتغیره، چندمتغیره.
- نمودار درست برای سؤال درست، نیمی از کار است.
- پارادوکس سیمپسون و چولگی، دو تله‌ی کلاسیک‌اند.
- خروجی EDA یک فهرست تصمیم است، نه یک نمودار.
- EDA سرمایه‌گذاری است، نه هزینه.

**سؤالات مروری:**
1. چهار سؤال بنیادی EDA چیست؟
2. پارادوکس سیمپسون را با یک مثال توضیح بده.
3. چرا EDA باید قبل از تقسیم داده انجام شود (یا حداقل آمار آن از train گرفته شود)؟

---

## فصل ۳۱: تجسم با Matplotlib

### مقدمه

Matplotlib کتابخانه‌ی پایه‌ی تجسم در پایتون است. Seaborn و Pandas روی آن ساخته شده‌اند. یادگیری‌اش کمی سخت‌تر از بقیه است، اما یک بار که بلد شوی، **هر نموداری** می‌توانی بسازی.

هدف این فصل، «هنر» نیست؛ **انتقال اطلاعات** است. نمودار خوب، یک پیام روشن دارد.

### مفاهیم اساسی

**۱. ساختار: Figure و Axes.**

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots(figsize=(8, 5))
ax.plot([1, 2, 3], [4, 5, 6])
ax.set_title("عنوان")
ax.set_xlabel("محور X")
ax.set_ylabel("محور Y")
plt.show()
```

- **Figure:** کل تصویر (بوم).
- **Axes:** یک نمودار داخل بوم.
- با `subplots(2, 2)` چهار Axes می‌گیری.

**۲. شش نمودار که ۹۵٪ کار را انجام می‌دهند:**

```python
import numpy as np
import matplotlib.pyplot as plt

rng = np.random.default_rng(0)
x = rng.normal(size=200)
y = x * 2 + rng.normal(scale=0.5, size=200)

fig, axes = plt.subplots(2, 3, figsize=(14, 8))

axes[0, 0].hist(x, bins=20, edgecolor="white")
axes[0, 0].set_title("هیستوگرام — توزیع")

axes[0, 1].scatter(x, y, s=15, alpha=0.6)
axes[0, 1].set_title("Scatter — رابطه")

axes[0, 2].boxplot(x, vert=True)
axes[0, 2].set_title("Boxplot — پرت و چارک")

axes[1, 0].bar(["الف", "ب", "ج"], [3, 7, 5])
axes[1, 0].set_title("Bar — مقایسه‌ی دسته‌ها")

axes[1, 1].plot(np.cumsum(rng.normal(size=100)), linewidth=2)
axes[1, 1].set_title("Line — روند")

axes[1, 2].pie([40, 35, 25], labels=["A", "B", "C"], autopct="%1.0f%%")
axes[1, 2].set_title("Pie — سهم (با احتیاط)")

plt.tight_layout()
plt.show()
```

**۳. تنظیمات مهم ظاهری.**

```python
ax.grid(alpha=0.3)                       # شبکه‌ی کم‌رنگ
ax.legend(loc="upper right")             # راهنما
ax.set_xlim(0, 10)                       # محدوده
ax.tick_params(axis="x", rotation=45)    # چرخش برچسب‌ها
plt.style.use("seaborn-v0_8-whitegrid")  # سبک آماده
```

**۴. ذخیره‌ی نمودار.**

```python
fig.savefig("plot.png", dpi=150, bbox_inches="tight")
```

`bbox_inches="tight"` از بریده شدن برچسب‌ها جلوگیری می‌کند.

**۵. نمودار برای متن فارسی.**

```python
import matplotlib
# اگر فونت فارسی داری (مثل Vazirmatn):
matplotlib.rcParams["font.family"] = "Tahoma"      # ویندوز
matplotlib.rcParams["axes.unicode_minus"] = False  # علامت منفی درست
```

بدون تنظیم فونت، فارسی به مربع تبدیل می‌شود. `Tahoma` در ویندوز تقریباً همیشه موجود است.

**۶. `tight_layout` و `constrained_layout`.** جلوگیری از هم‌پوشانی. اگر چیدمان به‌هم ریخت، `fig, ax = plt.subplots(layout="constrained")`.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — داشبورد چهارگانه برای EDA:**

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

rng = np.random.default_rng(42)
df = pd.DataFrame({
    "amount": rng.lognormal(10, 0.8, 500),
    "age": rng.integers(18, 70, 500),
    "city": rng.choice(["تهران", "شیراز", "مشهد"], 500),
    "churn": rng.integers(0, 2, 500),
})

fig, axes = plt.subplots(2, 2, figsize=(13, 8))

# ۱. توزیع مبلغ
axes[0, 0].hist(df["amount"], bins=40, color="#4C72B0", edgecolor="white")
axes[0, 0].axvline(df["amount"].median(), color="red", ls="--", label="میانه")
axes[0, 0].set_title("توزیع مبلغ سفارش (چوله به راست)")
axes[0, 0].legend()

# ۲. رابطه‌ی سن و مبلغ
axes[0, 1].scatter(df["age"], df["amount"], alpha=0.4, s=12)
axes[0, 1].set_xlabel("سن")
axes[0, 1].set_ylabel("مبلغ")
axes[0, 1].set_title("سن در برابر مبلغ")

# ۳. مقایسه‌ی شهرها
city_avg = df.groupby("city")["amount"].mean().sort_values()
axes[1, 0].barh(city_avg.index, city_avg.values, color="#55A868")
axes[1, 0].set_title("میانگین مبلغ به تفکیک شهر")
axes[1, 0].set_xlabel("میانگین مبلغ")

# ۴. نرخ ریزش به تفکیک شهر
churn_rate = df.groupby("city")["churn"].mean() * 100
axes[1, 1].bar(churn_rate.index, churn_rate.values, color="#C44E52")
axes[1, 1].set_title("نرخ ریزش (%)")
axes[1, 1].set_ylim(0, 100)

plt.tight_layout()
plt.show()
```

**مثال ۲ — مقایسه‌ی توزیع دو گروه:**

```python
fig, ax = plt.subplots(figsize=(8, 4))
ax.hist(df.loc[df["churn"] == 0, "amount"], bins=30, alpha=0.6, label="ماند")
ax.hist(df.loc[df["churn"] == 1, "amount"], bins=30, alpha=0.6, label="ریزش")
ax.set_title("توزیع مبلغ: ماند در برابر ریزش")
ax.legend()
plt.show()
```

**همپوشانی توزیع‌ها** یکی از مهم‌ترین چیزهایی است که در EDA می‌بینی: اگر دو گروه توزیع یکسان داشته باشند، آن ویژگی قدرت تفکیک ندارد.

### مقایسه و دیدگاه‌های مختلف

| کتابخانه | سطح | مزیت | عیب |
|---|---|---|---|
| Matplotlib | پایین | کنترل کامل، هر نموداری | کد زیاد |
| Seaborn | متوسط | زیبا، آماری | کمتر انعطاف‌پذیر |
| Pandas `.plot()` | بالا | یک خط | محدود |
| Plotly | بالا | تعاملی، تحت وب | حجم بیشتر |
| Altair | بالا | گرامر نمودار | اکوسیستم کوچک‌تر |

**توصیه:** Matplotlib را برای کنترل پایه یاد بگیر، Seaborn را برای کار روزمره، Plotly را وقتی تعامل لازم است.

### اشتباهات رایج

**۱. `plt.plot()` بدون `fig, ax`.** در نمودارهای چندگانه، حالت جهانی (`plt`) گیج‌کننده می‌شود. همیشه شیءگرا بنویس.
**۲. فراموش کردن `plt.show()` یا `tight_layout()`.** نمودار به‌هم‌ریخته یا نامرئی.
**۳. نمودار پای بدون برچسب محور.** نمودار بدون واحد، بی‌معنی است.
**۴. محور Y از صفر شروع نمی‌کند (برای bar).** تفاوت‌ها را بزرگ‌تر از واقعیت نشان می‌دهد.
**۵. نمودار دایره‌ای با ۱۰ دسته.** خواندنش غیرممکن است. Bar بهتر است.
**۶. فونت فارسی تنظیم نشده.** مربع به جای حرف.

### نکات مهم و ترفندها

- **`fig, ax = plt.subplots()` را عادت کن.** همیشه. حتی برای یک نمودار.
- **`alpha` برای شفافیت در نقاط هم‌پوشان.** بدون آن، scatter با ۱۰٬۰۰۰ نقطه یک لکه‌ی توپر می‌شود.
- **`s` (اندازه) و `bins` را تنظیم کن.** نمودار بد، نتیجه‌گیری بد می‌دهد.
- **رنگ را معنادار انتخاب کن.** در بازار ایران: افزایش قرمز، کاهش سبز.
- **میان‌بر:** `subplots` → `ax.plot/hist/scatter/bar` → `set_title/xlabel/ylabel` → `tight_layout` → `show`.

### خلاصه فصل

- Matplotlib با Figure و Axes کار می‌کند؛ سبک شیءگرا بهتر است.
- شش نمودار اصلی: hist، scatter، boxplot، bar، line، pie.
- تنظیم فونت برای فارسی و `axes.unicode_minus` لازم است.
- همپوشانی توزیع‌ها، قدرت تفکیک ویژگی را نشان می‌دهد.
- نمودار بدون برچسب و واحد، بی‌معنی است.

**سؤالات مروری:**
1. تفاوت Figure و Axes چیست؟
2. کِی boxplot از هیستوگرام مناسب‌تر است؟
3. چرا برای bar chart محور Y باید از صفر شروع شود؟

---

## فصل ۳۲: تجسم آماری با Seaborn

### مقدمه

Seaborn روی Matplotlib ساخته شده و دو کار را عالی انجام می‌دهد: **نمودارهای آماری با یک خط** و **زیبایی پیش‌فرض**. اگر Matplotlib «ابزار» است، Seaborn «میان‌بر» است.

در EDA، ۸۰٪ نمودارهایت با Seaborn یک خط می‌شوند.

### مفاهیم اساسی

**۱. سبک و تنظیمات.**

```python
import seaborn as sns
import matplotlib.pyplot as plt

sns.set_theme(style="whitegrid", context="notebook")
```

**۲. نمودارهای توزیع.**

```python
sns.histplot(data=df, x="amount", bins=30, kde=True)
sns.kdeplot(data=df, x="amount", hue="churn", fill=True)
sns.boxplot(data=df, x="city", y="amount")
sns.violinplot(data=df, x="city", y="amount", inner="quart")
```

**۳. نمودارهای رابطه.**

```python
sns.scatterplot(data=df, x="age", y="amount", hue="churn", alpha=0.6)
sns.lineplot(data=df, x="date", y="amount")
sns.regplot(data=df, x="age", y="amount")          # با خط رگرسیون
sns.lmplot(data=df, x="age", y="amount", hue="churn")
```

**۴. نمودارهای دسته‌ای.**

```python
sns.barplot(data=df, x="city", y="amount", errorbar="sd")   # میانگین + خطا
sns.countplot(data=df, x="city", hue="churn")
sns.boxenplot(data=df, x="city", y="amount")
```

**۵. Heatmap — همبستگی.**

```python
corr = df.select_dtypes("number").corr()
sns.heatmap(corr, annot=True, fmt=".2f", cmap="coolwarm",
            center=0, square=True, linewidths=0.5)
```

`center=0` رنگ صفر را خنثی می‌کند. `annot=True` عدد را می‌نویسد.

**۶. Pairplot — نمای کلی همه‌ی جفت‌ها.**

```python
sns.pairplot(df[["age", "amount", "churn"]], hue="churn", diag_kind="kde")
```

برای دیتاست کوچک (زیر ۱۰ ویژگی) فوق‌العاده است. برای دیتاست بزرگ، کند و شلوغ.

**۷. `FacetGrid` — نمودار شرطی.**

```python
g = sns.FacetGrid(df, col="city", hue="churn", height=3.5, aspect=1.2)
g.map(sns.histplot, "amount", bins=20, alpha=0.6)
g.add_legend()
```

**۸. `heatmap` جدول توافقی.**

```python
pd.crosstab(df["city"], df["churn"], normalize="index")
sns.heatmap(pd.crosstab(df["city"], df["churn"], normalize="index"),
            annot=True, fmt=".2f", cmap="Blues")
```

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — یک گزارش EDA کامل با Seaborn:**

```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt

sns.set_theme(style="whitegrid")

rng = np.random.default_rng(7)
n = 400
df = pd.DataFrame({
    "age": rng.integers(18, 70, n),
    "income": rng.normal(15, 5, n).clip(1),
    "city": rng.choice(["تهران", "شیراز", "مشهد"], n),
    "churn": rng.integers(0, 2, n),
})
df.loc[df["churn"] == 1, "income"] -= 3

fig, axes = plt.subplots(2, 2, figsize=(13, 9))

sns.histplot(df["income"], bins=30, kde=True, ax=axes[0, 0], color="#4C72B0")
axes[0, 0].set_title("توزیع درآمد")

sns.boxplot(data=df, x="churn", y="income", ax=axes[0, 1])
axes[0, 1].set_title("درآمد به تفکیک ریزش")

sns.scatterplot(data=df, x="age", y="income", hue="churn",
                alpha=0.6, ax=axes[1, 0])
axes[1, 0].set_title("سن در برابر درآمد")

corr = df.select_dtypes("number").corr()
sns.heatmap(corr, annot=True, fmt=".2f", cmap="coolwarm",
            center=0, square=True, ax=axes[1, 1])
axes[1, 1].set_title("ماتریس همبستگی")

plt.tight_layout()
plt.show()
```

**مثال ۲ — کشف رابطه‌ی غیرخطی:**

```python
# همبستگی خطی ضعیف، اما رابطه‌ی قوی غیرخطی
x = np.linspace(-3, 3, 300)
y = x ** 2 + rng.normal(scale=0.3, size=300)
tmp = pd.DataFrame({"x": x, "y": y})

print("همبستگی پیرسون:", round(tmp["x"].corr(tmp["y"]), 3))   # نزدیک صفر!

sns.scatterplot(data=tmp, x="x", y="y", alpha=0.5)
plt.title("همبستگی صفر، رابطه‌ی قوی")
plt.show()
```

**این نمودار، درس مهمی می‌دهد:** اگر فقط به عدد همبستگی نگاه کنی، این رابطه را از دست می‌دهی. **همیشه نمودار بکش.**

**مثال ۳ — مقایسه‌ی گروهی با violin:**

```python
sns.violinplot(data=df, x="city", y="income", hue="churn",
               split=True, inner="quart", cut=0)
plt.title("توزیع درآمد: شهر × ریزش")
plt.show()
```

`split=True` دو گروه را در یک ویولن کنار هم می‌گذارد — برای مقایسه‌ی مستقیم عالی است.

### مقایسه و دیدگاه‌های مختلف

| نمودار | چه می‌گوید | کِی |
|---|---|---|
| `histplot` | توزیع | تک‌متغیره |
| `kdeplot` | توزیع نرم‌شده | مقایسه‌ی گروه‌ها |
| `boxplot` | میانه، چارک، پرت | مقایسه‌ی سریع |
| `violinplot` | توزیع کامل + چگالی | مقایسه‌ی دقیق‌تر |
| `scatterplot` | رابطه‌ی دو عددی | دومتغیره |
| `heatmap` | ماتریس | همبستگی، توافقی |
| `pairplot` | همه‌ی جفت‌ها | نگاه اولیه |

**یک هشدار:** Seaborn نمودارهای زیبا می‌سازد، اما زیبایی می‌تواند گمراه کند. `barplot` پیش‌فرض میانگین را با فاصله‌ی اطمینان نشان می‌دهد — اگر داده چوله است، میانه بهتر است. `estimator="median"` بگذار.

### اشتباهات رایج

**۱. `barplot` روی داده‌ی چوله.** میانگین گمراه‌کننده. از `estimator="median"` یا boxplot استفاده کن.
**۲. `pairplot` روی ۳۰ ویژگی.** کند و ناخوانا. فقط ویژگی‌های مهم را بده.
**۳. `heatmap` بدون `center=0`.** رنگ‌ها معنای متقارن ندارند.
**۴. `countplot` به جای `barplot`.** اولی شمارش می‌کند، دومی تجمیع.
**۵. نادیده گرفتن `hue`.** مقایسه‌ی گروهی، اغلب مهم‌ترین بینش را می‌دهد.
**۶. اعتماد به Seaborn بدون تنظیم فونت فارسی.** همان مشکل Matplotlib.

### نکات مهم و ترفندها

- **`sns.set_theme(style="whitegrid")` را یک بار در ابتدا بگذار.** همه‌ی نمودارها زیبا می‌شوند.
- **`hue` را جدی بگیر.** همان محور سومی است که بینش را می‌سازد.
- **`corr()` + `heatmap` = نقشه‌ی سریع ویژگی‌ها.** همبستگی بالای ۰.۹ = ویژگی تکراری.
- **`violinplot(split=True)`** برای مقایسه‌ی دو گروه، بهترین انتخاب است.
- **میان‌بر:** توزیع → hist/kde · مقایسه → box/violin · رابطه → scatter/reg · ماتریس → heatmap.

### خلاصه فصل

- Seaborn نمودارهای آماری را یک‌خطی می‌کند و زیبایی پیش‌فرض دارد.
- hist/kde برای توزیع، box/violin برای مقایسه، scatter/reg برای رابطه، heatmap برای ماتریس.
- `hue` بینش گروهی می‌سازد.
- همبستگی صفر می‌تواند با رابطه‌ی قوی غیرخطی همراه باشد؛ نمودار بکش.
- `barplot` روی داده‌ی چوله گمراه‌کننده است.

**سؤالات مروری:**
1. تفاوت `boxplot` و `violinplot` چیست؟
2. چرا `center=0` در heatmap مهم است؟
3. کِی `pairplot` مناسب است و کِی نه؟

---

## فصل ۳۳: از نمودار تا بینش: داستان‌سرایی داده

### مقدمه

نمودار زیبا که پیام ندارد، هنر است نه مهندسی. **بینش (Insight)** یعنی: «یک جمله‌ی قابل اقدام که از داده بیرون آمده.» این فصل، پل بین «نمودار کشیدن» و «تصمیم گرفتن» است.

مهارت اصلی این فصل، **انتخاب** است: از ۲۰ نمودار، کدام ۳ تا را نشان بدهی؟

### مفاهیم اساسی

**۱. ساختار یک بینش:**

```
[مشاهده] + [مقایسه/بافت] + [پیامد/اقدام]
```

مثال ضعیف: «نرخ ریزش ۲۳٪ است.»
مثال قوی: «نرخ ریزش در مشتریان با درآمد پایین ۳۱٪ است، در برابر ۱۴٪ در بقیه — یعنی تمرکز کمپین نگهداشت باید روی این گروه باشد.»

**۲. سه اصل نمودار مؤثر:**

- **یک پیام در هر نمودار.** اگر دو پیام داری، دو نمودار بساز.
- **مقایسه، نه عدد مطلق.** «۳۱٪ در برابر ۱۴٪» از «۳۱٪» قوی‌تر است.
- **حذف نویز بصری.** هر خط، رنگ و برچسبی که به پیام کمک نمی‌کند، حذف شود.

**۳. سلسله‌مراتب بصری.** چشم انسان اول چه می‌بیند؟
1. رنگ متضاد
2. موقعیت (بالا-چپ در فرهنگ ما)
3. اندازه
4. متن

پس مهم‌ترین چیز را با رنگ متضاد نشان بده، نه با ۱۰ رنگ مختلف.

**۴. انتخاب نمودار برای پیام:**

| پیام | نمودار |
|---|---|
| «A بیشتر از B است» | bar |
| «روند در زمان» | line |
| «توزیع متفاوت است» | box/violin/hist |
| «رابطه وجود دارد» | scatter + خط روند |
| «سهم‌ها» | bar انباشته (نه pie، اگر بیش از ۳ دسته) |
| «یک نقطه متفاوت است» | scatter با برجسته‌سازی |

**۵. آنچه نباید در نمودار باشد:**
- ۳بعدی (اعوجاج ادراکی)
- Pie با بیش از ۵ دسته
- محور Y بریده‌شده در bar (اغلب)
- رنگ‌های رنگین‌کمانی برای داده‌ی ترتیبی
- نمودار دو محور Y بدون هشدار (dual axis، منبع سوءتفسیر)

**۶. جدول در برابر نمودار.** اگر دقیقاً ۵ عدد داری که مخاطب می‌خواهد بخواند، جدول بهتر است. نمودار برای **الگو** است، جدول برای **مقدار دقیق**.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — از نمودار خام به نمودار با پیام:**

```python
import matplotlib.pyplot as plt
import pandas as pd

churn = pd.DataFrame({
    "segment": ["درآمد کم", "درآمد متوسط", "درآمد بالا"],
    "rate": [31, 19, 14],
})

fig, ax = plt.subplots(figsize=(8, 4.5))

colors = ["#C44E52", "#8C8C8C", "#8C8C8C"]     # فقط مهم‌ترین میله رنگی
bars = ax.barh(churn["segment"], churn["rate"], color=colors)

ax.set_xlim(0, 40)
ax.set_xlabel("نرخ ریزش (%)")
ax.set_title("ریزش در مشتریان کم‌درآمد دو برابر بیشتر است", loc="right", fontsize=12)

# برچسب مقدار روی میله
for bar, rate in zip(bars, churn["rate"]):
    ax.text(rate + 1, bar.get_y() + bar.get_height() / 2,
            f"{rate}%", va="center", fontsize=10)

ax.spines[["top", "right"]].set_visible(False)   # حذف کادر اضافه
plt.tight_layout()
plt.show()
```

**تفاوت‌ها با نمودار خام:**
- عنوان، **پیام** است نه «نمودار نرخ ریزش».
- فقط میله‌ی مهم رنگی است.
- مقدارها روی میله نوشته شده‌اند (بدون نیاز به خواندن محور).
- خطوط اضافی حذف شده‌اند.

**مثال ۲ — برجسته‌سازی در scatter:**

```python
import numpy as np
import matplotlib.pyplot as plt

rng = np.random.default_rng(3)
x = rng.normal(size=300)
y = x * 1.5 + rng.normal(scale=1.0, size=300)
outlier_idx = np.argsort(y)[-3:]     # ۳ نقطه‌ی بالاترین

fig, ax = plt.subplots(figsize=(8, 5))
ax.scatter(x, y, alpha=0.35, s=25, color="#4C72B0")
ax.scatter(x[outlier_idx], y[outlier_idx], color="#C44E52", s=90,
           edgecolor="black", linewidth=0.8, label="نقاط نیازمند بررسی")
ax.legend()
ax.set_title("سه مشاهده از الگوی کلی فاصله دارند", loc="right")
ax.spines[["top", "right"]].set_visible(False)
plt.tight_layout()
plt.show()
```

**مثال ۳ — گزارش متنی همراه نمودار (مهم‌ترین بخش):**

```python
def insight_report(df, target):
    """تولید خودکار بینش‌های متنی — الگوی گزارش‌دهی."""
    lines = []
    base = df[target].mean()

    lines.append(f"نرخ پایه‌ی {target}: {base:.1%}")

    for col in df.select_dtypes("number").columns:
        if col == target:
            continue
        corr = df[col].corr(df[target])
        if abs(corr) > 0.2:
            direction = "افزایش" if corr > 0 else "کاهش"
            lines.append(
                f"با {direction} «{col}» احتمال {target} بیشتر می‌شود "
                f"(همبستگی {corr:+.2f})"
            )

    return "\n".join(lines)

# print(insight_report(df, "churn"))
```

**چرا این مهم است؟** چون بیشتر مخاطبان تو (مدیر، ذی‌نفع) نمودار را نمی‌خوانند؛ **جمله** را می‌خوانند.

### مقایسه و دیدگاه‌های مختلف

| مخاطب | چه می‌خواهد | چه چیزی نشان بده |
|---|---|---|
| مدیر | تصمیم | یک نمودار + یک جمله + پیشنهاد |
| همکار فنی | جزئیات | جدول معیارها، نمودار خطا |
| ذی‌نفع خارجی | اطمینان | محدودیت‌ها، دامنه‌ی اعتبار |
| خودت (آینده) | بازتولید | کد، داده، نسخه |

**دو دیدگاه درباره‌ی «زیبایی»:**
- **«نمودار باید زیبا باشد تا جدی گرفته شود.»** درست است: نمودار زشت، اعتماد را کم می‌کند.
- **«زیبایی مهم نیست، دقت مهم است.»** هم درست است: نمودار زیبای گمراه‌کننده، بدتر از نمودار زشت درست است.
**جمع:** درست اول، زیبا دوم. هرگز برعکس.

### اشتباهات رایج

**۱. عنوان توصیفی به جای پیام.** «نمودار فروش» ← «فروش زمستان ۳۰٪ کمتر از تابستان است».
**۲. نمایش همه‌ی ۲۰ نمودار.** مخاطب هیچ‌چیز یاد نمی‌گیرد. سه نمودار، سه پیام.
**۳. محور Y بریده در bar.** تفاوت‌ها را بزرگ‌نمایی می‌کند (فریب بصری).
**۴. رنگ‌های بی‌معنا.** ۷ رنگ مختلف که هیچ چیزی را کد نمی‌کنند.
**۵. نتیجه‌گیری علی از همبستگی.** «X باعث Y می‌شود» در حالی که فقط همبستگی دیدی.
**۶. پنهان کردن محدودیت‌ها.** نمونه‌ی کوچک، بازه‌ی زمانی محدود، داده‌ی سوگیرانه — همه باید ذکر شوند.

### نکات مهم و ترفندها

- **قاعده‌ی «یک نمودار، یک پیام».** اگر نمی‌توانی پیام را در یک جمله بنویسی، نمودار آماده نیست.
- **عنوان را پیام بنویس، زیرنویس را توضیح.** عنوان = نتیجه، زیرنویس = روش.
- **رنگ را فقط برای برجسته‌سازی استفاده کن.** بقیه خاکستری.
- **`ax.spines[["top","right"]].set_visible(False)`** فوراً نمودار را حرفه‌ای‌تر می‌کند.
- **میان‌بر:** مشاهده + مقایسه + اقدام = بینش. یک نمودار، یک پیام.

### خلاصه فصل

- بینش = مشاهده + مقایسه + اقدام؛ نه یک عدد خام.
- یک نمودار، یک پیام؛ عنوان، خودِ پیام است.
- رنگ برای برجسته‌سازی، نه تزئین.
- برای مخاطب غیرفنی، جمله مهم‌تر از نمودار است.
- محدودیت‌ها را پنهان نکن؛ اعتماد از صداقت می‌آید.

**سؤالات مروری:**
1. تفاوت «مشاهده» و «بینش» چیست؟
2. چرا بریدن محور Y در bar chart مسئله‌دار است؟
3. برای مخاطب مدیریتی، چه چیزی را کنار نمودار می‌گذاری؟

---

## منابع و مراجع

**مستندات**
- Matplotlib Official — *Pyplot tutorial*, *Examples gallery*
- Seaborn Official — *API reference*, *Example gallery*
- Pandas — *Visualization* (`df.plot`)

**کتاب‌ها**
- Wilke, C. — *Fundamentals of Data Visualization* (رایگان آنلاین — بهترین کتاب در این موضوع)
- Cairo, A. — *The Truthful Art*
- Knaflic, C. — *Storytelling with Data*

**ابزارهای تکمیلی**
- `ydata-profiling` — گزارش خودکار EDA
- `missingno` — تجسم الگوی داده‌ی گمشده
- Plotly / Altair — نمودار تعاملی

---

**بعدی:** [`08-Machine-Learning-Basics.md`](./08-Machine-Learning-Basics.md) — فصل ۳۴: یادگیری ماشین چیست؟
