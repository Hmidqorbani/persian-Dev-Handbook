# بخش ۱۰: یادگیری بدون ناظر

**مدت یادگیری تقریبی**: ۲ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۸ (مبانی ML)، فایل ۰۵ (NumPy)، فایل ۰۴ (جبر خطی)

---

## فهرست محتوا

- [فصل ۴۴: خوشه‌بندی K-Means](#فصل-۴۴-خوشهبندی-k-means)
- [فصل ۴۵: خوشه‌بندی سلسله‌مراتبی و DBSCAN](#فصل-۴۵-خوشهبندی-سلسلهمراتبی-و-dbscan)
- [فصل ۴۶: کاهش بعد: PCA و t-SNE](#فصل-۴۶-کاهش-بعد-pca-و-t-sne)
- [فصل ۴۷: تشخیص ناهنجاری و کاربردهای واقعی](#فصل-۴۷-تشخیص-ناهنجاری-و-کاربردهای-واقعی)

---

## فصل ۴۴: خوشه‌بندی K-Means

### مقدمه

تصور کن ۵۰۰۰ مشتری داری و هیچ برچسبی نداری. می‌خواهی بدانی «چند نوع مشتری» داری. این، مسئله‌ی خوشه‌بندی است: **گروه‌بندی بدون دانستن گروه‌ها**.

K-Means ساده‌ترین و پرکاربردترین الگوریتم خوشه‌بندی است. اگر مفهومش را بفهمی، بقیه‌ی روش‌های خوشه‌بندی برایت قابل فهم می‌شوند.

### مفاهیم اساسی

**۱. الگوریتم در چهار قدم:**

```
۱. k مرکز تصادفی انتخاب کن
۲. هر نقطه را به نزدیک‌ترین مرکز نسبت بده
۳. هر مرکز را به میانگین نقاط خودش منتقل کن
۴. تا تغییر نکردن تکرار کن
```

**همین.** این سادگی، دلیل محبوبیتش است.

**۲. تابع هدف: inertia (WCSS).**

```
inertia = Σ (فاصله‌ی هر نقطه تا مرکز خوشه‌اش)²
```

K-Means این را کم می‌کند. **همیشه با افزایش k کاهش می‌یابد** — پس نمی‌توانی از آن به‌تنهایی برای انتخاب k استفاده کنی.

**۳. انتخاب k — سه روش:**

- **Elbow Method:** نمودار inertia در برابر k. «آرنج» نقطه‌ی خوبی است.
- **Silhouette Score:** بین -۱ و ۱. بالاتر = خوشه‌بندی بهتر. **قابل اعتمادتر از elbow.**
- **دانش دامنه:** «ما ۴ بخش بازار داریم» — بهترین روش، اگر موجود باشد.

**۴. چهار محدودیت مهم K-Means:**

| محدودیت | توضیح |
|---|---|
| **k را باید بدانی** | الگوریتم خودش تعداد خوشه را پیدا نمی‌کند |
| **خوشه‌های کروی** | خوشه‌های کشیده یا حلقه‌ای را نمی‌بیند |
| **حساس به مقدار اولیه** | `n_init=10` (چند شروع) لازم است |
| **حساس به مقیاس** | باید استانداردسازی کنی |

**۵. K-Means در برابر K-Medoids:** دومی از نقطه‌ی واقعی به‌عنوان مرکز استفاده می‌کند (مقاوم‌تر به پرت).

**۶. MiniBatchKMeans:** نسخه‌ی سریع برای داده‌ی بزرگ. کیفیت کمی کمتر، سرعت خیلی بیشتر.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — K-Means از صفر:**

```python
import numpy as np

def kmeans(X, k, n_iter=100, seed=0):
    rng = np.random.default_rng(seed)
    # ۱. مراکز اولیه: k نقطه‌ی تصادفی از داده
    centers = X[rng.choice(len(X), k, replace=False)]

    for _ in range(n_iter):
        # ۲. تخصیص: هر نقطه به نزدیک‌ترین مرکز
        distances = ((X[:, None, :] - centers[None, :, :]) ** 2).sum(axis=2)
        labels = distances.argmin(axis=1)

        # ۳. به‌روزرسانی مراکز
        new_centers = np.array([
            X[labels == j].mean(axis=0) if np.any(labels == j) else centers[j]
            for j in range(k)
        ])

        if np.allclose(new_centers, centers):
            break
        centers = new_centers

    inertia = ((X - centers[labels]) ** 2).sum()
    return labels, centers, inertia

rng = np.random.default_rng(42)
X = np.vstack([
    rng.normal([0, 0], 0.7, (150, 2)),
    rng.normal([5, 5], 0.7, (150, 2)),
    rng.normal([0, 6], 0.7, (150, 2)),
])

labels, centers, inertia = kmeans(X, k=3)
print("مراکز:\n", centers.round(2))
print("inertia:", round(inertia, 1))
print("توزیع خوشه‌ها:", np.bincount(labels))
```

**مثال ۲ — انتخاب k با elbow و silhouette:**

```python
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.metrics import silhouette_score
from sklearn.preprocessing import StandardScaler

X_scaled = StandardScaler().fit_transform(X)

inertias, silhouettes = [], []
ks = range(2, 9)

for k in ks:
    km = KMeans(n_clusters=k, n_init=10, random_state=42).fit(X_scaled)
    inertias.append(km.inertia_)
    silhouettes.append(silhouette_score(X_scaled, km.labels_))

fig, axes = plt.subplots(1, 2, figsize=(12, 4))
axes[0].plot(list(ks), inertias, marker="o")
axes[0].set_title("Elbow Method")
axes[0].set_xlabel("k")

axes[1].plot(list(ks), silhouettes, marker="o", color="green")
axes[1].set_title("Silhouette Score")
axes[1].set_xlabel("k")
plt.tight_layout()
plt.show()

best_k = list(ks)[int(np.argmax(silhouettes))]
print(f"بهترین k بر اساس silhouette: {best_k}")
```

**مثال ۳ — بخش‌بندی مشتریان (کاربرد واقعی):**

```python
import pandas as pd
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler

rng = np.random.default_rng(7)
customers = pd.DataFrame({
    "recency_days": rng.integers(1, 365, 500),
    "frequency":    rng.integers(1, 40, 500),
    "monetary":     rng.lognormal(10, 1, 500).round(0),
})

features = ["recency_days", "frequency", "monetary"]
Xc = StandardScaler().fit_transform(customers[features])

km = KMeans(n_clusters=4, n_init=10, random_state=42).fit(Xc)
customers["segment"] = km.labels_

# پروفایل هر بخش (روی داده‌ی اصلی، نه استانداردشده)
profile = customers.groupby("segment")[features].mean().round(1)
profile["count"] = customers["segment"].value_counts().sort_index()
print(profile)

# نام‌گذاری معنادار
# بخش با recency کم + frequency بالا + monetary بالا = «مشتریان وفادار ارزشمند»
```

**این الگو (RFM: Recency, Frequency, Monetary) یکی از پرکاربردترین تحلیل‌های کسب‌وکار است** و با K-Means چند خط بیشتر نیست.

### مقایسه و دیدگاه‌های مختلف

| الگوریتم | نیاز به k | شکل خوشه | سرعت | پرت |
|---|---|---|---|---|
| K-Means | بله | کروی | خیلی سریع | حساس |
| MiniBatch K-Means | بله | کروی | خیلی سریع‌تر | حساس |
| DBSCAN | خیر | دلخواه | متوسط | مقاوم |
| Hierarchical | خیر (بعداً برش می‌زنی) | دلخواه | کند | متوسط |
| GMM | بله | بیضوی | متوسط | متوسط |

**یک دیدگاه:** «K-Means همیشه جواب می‌دهد، اما همیشه جواب درست نمی‌دهد.» الگوریتم روی هر داده‌ای خوشه پیدا می‌کند — حتی اگر داده واقعاً خوشه نداشته باشد. **همیشه با silhouette و نمودار بررسی کن.**

### اشتباهات رایج

**۱. K-Means بدون استانداردسازی.** ویژگی با مقیاس بزرگ، همه‌چیز را تعیین می‌کند.
**۲. `n_init=1`.** نتیجه به شروع تصادفی وابسته می‌شود. `n_init=10` (پیش‌فرض جدید) بگذار.
**۳. انتخاب k فقط با inertia.** همیشه کاهش می‌یابد؛ بی‌معنی است.
**۴. تفسیر خوشه به‌عنوان «گروه واقعی».** خوشه، ساختار ریاضی است، نه حقیقت کسب‌وکار.
**۵. استفاده روی داده‌ی با ابعاد بالا بدون کاهش بعد.** فاصله‌ها بی‌معنی می‌شوند.
**۶. نادیده گرفتن مقادیر پرت.** K-Means مرکز را به سمت پرت می‌کشد.

### نکات مهم و ترفندها

- **`StandardScaler` واجب است.** بدون آن، نتیجه بی‌معنی.
- **`silhouette_score` را به elbow ترجیح بده.** عددی و قابل مقایسه است.
- **پروفایل خوشه‌ها را روی داده‌ی اصلی گزارش کن.** «خوشه ۲: میانگین خرید ۵۰۰ هزار، ۱۲ خرید در سال» — این برای کسب‌وکار مفید است.
- **`random_state` بگذار** تا خوشه‌ها بازتولید شوند.
- **میان‌بر:** k مرکز → تخصیص → به‌روزرسانی → تکرار. ساده و سریع، اما کروی‌پسند.

### خلاصه فصل

- K-Means با تکرار تخصیص و به‌روزرسانی مرکز کار می‌کند.
- inertia همیشه با k کاهش می‌یابد؛ برای انتخاب k از silhouette استفاده کن.
- استانداردسازی و `n_init` چندگانه واجب‌اند.
- K-Means خوشه‌های کروی و مشابه اندازه را ترجیح می‌دهد.
- RFM + K-Means، الگوی کلاسیک بخش‌بندی مشتری است.

**سؤالات مروری:**
1. چهار قدم K-Means را به ترتیب بگو.
2. چرا inertia برای انتخاب k مناسب نیست؟
3. چرا K-Means روی داده‌ی استاندارد‌نشده شکست می‌خورد؟

---

## فصل ۴۵: خوشه‌بندی سلسله‌مراتبی و DBSCAN

### مقدمه

K-Means دو ضعف بزرگ دارد: باید `k` را بدانی و فقط خوشه‌ی کروی می‌بیند. این فصل دو الگوریتم معرفی می‌کند که این محدودیت‌ها را برطرف می‌کنند:

- **سلسله‌مراتبی:** یک «درخت خوشه» می‌سازد که می‌توانی در هر سطحی برش بزنی.
- **DBSCAN:** خوشه را به‌عنوان «ناحیه‌ی متراکم» تعریف می‌کند و پرت‌ها را جدا می‌کند.

### مفاهیم اساسی

**۱. خوشه‌بندی سلسله‌مراتبی (Agglomerative).**

رویکرد «از پایین به بالا»:
```
۱. هر نقطه، یک خوشه‌ی جدا
۲. نزدیک‌ترین دو خوشه را ادغام کن
۳. تا رسیدن به یک خوشه ادامه بده
```

خروجی: **دندروگرام (Dendrogram)** — درختی که نشان می‌دهد کدام خوشه‌ها در چه فاصله‌ای ادغام شدند.

**معیار فاصله بین خوشه‌ها (linkage):**

| linkage | فاصله | ویژگی |
|---|---|---|
| `single` | نزدیک‌ترین زوج | خوشه‌های زنجیره‌ای، حساس به نویز |
| `complete` | دورترین زوج | خوشه‌های فشرده |
| `average` | میانگین همه‌ی زوج‌ها | تعادل |
| `ward` | افزایش واریانس | **پیش‌فرض و معمولاً بهترین** |

**۲. DBSCAN — سه مفهوم:**

- **eps (ε):** شعاع همسایگی.
- **min_samples:** حداقل تعداد نقطه در همسایگی.
- **سه نوع نقطه:**
  - **Core:** حداقل `min_samples` در شعاع `eps` دارد.
  - **Border:** در شعاع یک core است، اما خودش core نیست.
  - **Noise:** نه core، نه border → پرت.

**مزیت‌های DBSCAN:**
- نیاز به `k` ندارد.
- خوشه‌های با شکل دلخواه (حلقه، مارپیچ) را می‌بیند.
- پرت‌ها را **صریحاً** جدا می‌کند (برچسب `-1`).

**عیب‌ها:**
- حساس به `eps` و `min_samples`.
- در ابعاد بالا ضعیف (مشکل تراکم).
- اگر خوشه‌ها تراکم متفاوت داشته باشند، شکست می‌خورد.

**۳. HDBSCAN** — نسخه‌ی بهبودیافته که `eps` را خودکار تعیین می‌کند و با تراکم‌های متفاوت کار می‌کند. اگر کتابخانه‌اش را داری، به DBSCAN ترجیح بده.

**۴. انتخاب `eps`:** نمودار «فاصله تا k-امین همسایه» را مرتب کن؛ نقطه‌ی «زانو» مقدار خوبی برای `eps` است.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — دندروگرام:**

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.cluster.hierarchy import dendrogram, linkage
from sklearn.datasets import make_blobs

X, _ = make_blobs(n_samples=60, centers=3, cluster_std=0.7, random_state=42)

Z = linkage(X, method="ward")

plt.figure(figsize=(11, 4))
dendrogram(Z, no_labels=True)
plt.title("دندروگرام — ادغام خوشه‌ها")
plt.ylabel("فاصله")
plt.show()

# برش در ارتفاع مشخص → خوشه‌ها
from scipy.cluster.hierarchy import fcluster
labels = fcluster(Z, t=3, criterion="maxclust")
print("تعداد خوشه‌ها:", len(set(labels)))
```

**چگونه دندروگرام را بخوانیم؟** خط عمودی بلندتر = فاصله‌ی بیشتر بین خوشه‌ها = برش بهتر. اگر یک خط افقی بکشی، تعداد خوشه‌ها = تعداد خطوطی که قطع می‌کنی.

**مثال ۲ — DBSCAN روی داده‌ی حلقه‌ای:**

```python
from sklearn.datasets import make_circles, make_moons
from sklearn.cluster import KMeans, DBSCAN
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import silhouette_score

X_c, y_c = make_circles(n_samples=500, factor=0.4, noise=0.06, random_state=42)
X_c = StandardScaler().fit_transform(X_c)

km = KMeans(n_clusters=2, n_init=10, random_state=42).fit(X_c)
db = DBSCAN(eps=0.3, min_samples=5).fit(X_c)

print("K-Means silhouette:", round(silhouette_score(X_c, km.labels_), 3))
mask = db.labels_ != -1
print("DBSCAN silhouette:", round(silhouette_score(X_c[mask], db.labels_[mask]), 3))
print("DBSCAN: خوشه‌ها =", len(set(db.labels_)) - (1 if -1 in db.labels_ else 0),
      "| پرت =", (db.labels_ == -1).sum())
```

K-Means روی داده‌ی دایره‌ای، دو نیم‌دایره می‌دهد (غلط). DBSCAN دو حلقه را درست پیدا می‌کند.

**مثال ۳ — انتخاب eps:**

```python
from sklearn.neighbors import NearestNeighbors

k = 5
nbrs = NearestNeighbors(n_neighbors=k).fit(X_c)
distances, _ = nbrs.kneighbors(X_c)
k_dist = np.sort(distances[:, -1])

plt.figure(figsize=(8, 4))
plt.plot(k_dist)
plt.ylabel(f"فاصله تا {k}-امین همسایه")
plt.title("انتخاب eps از نقطه‌ی زانو")
plt.grid(alpha=0.3)
plt.show()
```

**مثال ۴ — تشخیص پرت با DBSCAN:**

```python
rng = np.random.default_rng(0)
X_in = rng.normal(0, 1, (300, 2))
X_out = rng.uniform(-8, 8, (20, 2))          # پرت‌های دور
X_all = np.vstack([X_in, X_out])

db = DBSCAN(eps=0.6, min_samples=8).fit(X_all)
n_noise = (db.labels_ == -1).sum()
print(f"خوشه‌ها: {len(set(db.labels_)) - 1} | نقاط پرت: {n_noise}")
print("آیا پرت‌های مصنوعی پیدا شدند؟", n_noise >= 15)
```

### مقایسه و دیدگاه‌های مختلف

| معیار | K-Means | سلسله‌مراتبی | DBSCAN |
|---|---|---|---|
| نیاز به k | بله | خیر | خیر |
| شکل خوشه | کروی | دلخواه | دلخواه |
| مقیاس‌پذیری | عالی | ضعیف (O(n²)) | متوسط |
| پرت | حساس | متوسط | جدا می‌کند |
| خروجی | برچسب | درخت + برچسب | برچسب + پرت |
| تفسیر | آسان | **عالی** (دندروگرام) | متوسط |

**یک دیدگاه:** «DBSCAN بهترین است.» نه همیشه. اگر داده تراکم یکنواخت ندارد، DBSCAN شکست می‌خورد. **HDBSCAN** این مشکل را حل می‌کند و انتخاب اول مدرن است.

### اشتباهات رایج

**۱. DBSCAN بدون استانداردسازی.** `eps` به مقیاس وابسته است.
**۲. `eps` ثابت برای همه‌ی مسائل.** باید از نمودار k-distance تعیین شود.
**۳. سلسله‌مراتبی روی ۱۰۰٬۰۰۰ نقطه.** O(n²) حافظه — غیرعملی.
**۴. تفسیر برچسب `-1` به‌عنوان خطا.** `-1` یعنی «پرت»، که گاهی همان هدف است.
**۵. `linkage="single"` بدون فکر.** خوشه‌های زنجیره‌ای می‌سازد.
**۶. فراموش کردن `silhouette` روی DBSCAN با پرت.** باید پرت‌ها را قبل از محاسبه حذف کنی.

### نکات مهم و ترفندها

- **دندروگرام را رسم کن، حتی اگر K-Means استفاده می‌کنی.** دید خوبی از ساختار می‌دهد.
- **`ward` را پیش‌فرض بگیر.**
- **`eps` را از k-distance graph انتخاب کن.** نه حدس.
- **HDBSCAN را امتحان کن** اگر تراکم‌ها متفاوت‌اند.
- **میان‌بر:** سلسله‌مراتبی = درخت + برش · DBSCAN = تراکم + پرت.

### خلاصه فصل

- سلسله‌مراتبی، درختی از خوشه‌ها می‌سازد و در هر سطحی قابل برش است.
- `ward` معمولاً بهترین linkage است.
- DBSCAN خوشه را ناحیه‌ی متراکم تعریف می‌کند و پرت‌ها را جدا می‌کند.
- `eps` و `min_samples` کلیدی‌اند و باید از داده تعیین شوند.
- HDBSCAN نسخه‌ی مدرن و مقاوم‌تر است.

**سؤالات مروری:**
1. چرا DBSCAN روی داده‌ی حلقه‌ای بهتر از K-Means است؟
2. سه نوع نقطه در DBSCAN چیست؟
3. چگونه `eps` مناسب را پیدا می‌کنی؟

---

## فصل ۴۶: کاهش بعد: PCA و t-SNE

### مقدمه

«لعنت ابعاد» یک مشکل واقعی است: در ۱۰۰۰ بُعد، همه‌ی نقاط تقریباً هم‌فاصله‌اند، مدل‌ها کند می‌شوند و نمودار کشیدن غیرممکن است. **کاهش بعد** یعنی فشرده کردن داده به ابعاد کمتر، با حفظ بیشترین اطلاعات.

دو هدف متفاوت وجود دارد:
- **PCA:** فشرده‌سازی برای مدل‌سازی (حفظ واریانس).
- **t-SNE / UMAP:** تجسم برای انسان (حفظ ساختار محلی).

**این دو را با هم اشتباه نگیر.**

### مفاهیم اساسی

**۱. PCA — ایده.**

پیدا کردن جهت‌هایی که داده بیشترین **واریانس** را در آن‌ها دارد. اولین مؤلفه = جهت بیشترین پراکندگی. دومین = عمود بر اولی، بیشترین پراکندگی باقی‌مانده، و...

**۲. مراحل PCA:**

```
۱. داده را مرکز کن (میانگین صفر)
۲. (اختیاری) استانداردسازی
۳. ماتریس کوواریانس را حساب کن
۴. مقادیر و بردارهای ویژه را پیدا کن (یا SVD)
۵. به k مؤلفه‌ی اول نگاشت کن
```

**۳. واریانس توضیح‌داده‌شده (Explained Variance).**
هر مؤلفه، درصدی از کل واریانس را توضیح می‌دهد. قاعده‌ی رایج: مؤلفه‌هایی را نگه دار که ۹۰–۹۵٪ واریانس را پوشش می‌دهند.

**۴. نکته‌ی حیاتی: مقیاس‌بندی.**

اگر ویژگی‌ها مقیاس متفاوت دارند، **باید استانداردسازی کنی**. وگرنه PCA مؤلفه‌ی اول را به سمت ویژگی با واریانس بزرگ می‌برد — که فقط یک اثر مقیاس است، نه اطلاعات.

**۵. PCA چه کاری نمی‌کند:**
- انتخاب ویژگی نمی‌کند (مؤلفه‌ها ترکیب خطی همه‌ی ویژگی‌ها هستند).
- روابط غیرخطی را نمی‌بیند.
- برای تجسم با ساختار پیچیده، مناسب نیست.

**۶. t-SNE — ایده.**

t-SNE سعی می‌کند **همسایگی‌های محلی** را حفظ کند: نقاطی که در فضای بالا نزدیک‌اند، در فضای پایین هم نزدیک باشند. برای تجسم فوق‌العاده است.

**محدودیت‌های t-SNE:**
- **فاصله‌های جهانی بی‌معنی‌اند.** فقط خوشه‌های محلی قابل تفسیرند.
- **پارامتر `perplexity`** (معمولاً ۵–۵۰) نتیجه را به‌شدت تغییر می‌دهد.
- **تصادفی است.** نتایج متفاوت با seed متفاوت.
- **کند** و برای داده‌ی جدید قابل استفاده نیست (non-parametric).

**۷. UMAP** — جایگزین مدرن: سریع‌تر، ساختار جهانی را بهتر حفظ می‌کند، و می‌تواند داده‌ی جدید را نگاشت کند.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — PCA از صفر با SVD:**

```python
import numpy as np

def pca_from_scratch(X, n_components):
    # ۱. مرکز کردن
    X_centered = X - X.mean(axis=0)

    # ۲. SVD
    U, S, Vt = np.linalg.svd(X_centered, full_matrices=False)

    # ۳. نسبت واریانس
    explained = (S ** 2) / (S ** 2).sum()

    # ۴. نگاشت
    X_reduced = X_centered @ Vt[:n_components].T

    return X_reduced, explained, Vt[:n_components]

rng = np.random.default_rng(0)
# داده با ۵ ویژگی که در واقع از ۲ مؤلفه ساخته شده
latent = rng.normal(size=(300, 2))
mixing = rng.normal(size=(2, 5))
X = latent @ mixing + rng.normal(scale=0.1, size=(300, 5))

X_pca, explained, components = pca_from_scratch(X, 2)
print("نسبت واریانس هر مؤلفه:", explained.round(3))
print("واریانس تجمعی:", np.cumsum(explained).round(3))
print("شکل جدید:", X_pca.shape)
```

**مثال ۲ — انتخاب تعداد مؤلفه:**

```python
import matplotlib.pyplot as plt
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import load_digits

digits = load_digits()
X_d, y_d = digits.data, digits.target      # ۱۷۹۷ نمونه، ۶۴ ویژگی

X_scaled = StandardScaler().fit_transform(X_d)
pca_full = PCA().fit(X_scaled)
cum_var = np.cumsum(pca_full.explained_variance_ratio_)

n_90 = int(np.argmax(cum_var >= 0.90) + 1)
print(f"تعداد مؤلفه برای ۹۰٪ واریانس: {n_90} (از ۶۴)")

plt.figure(figsize=(8, 4))
plt.plot(cum_var, marker="o", ms=3)
plt.axhline(0.9, color="red", ls="--", label="۹۰٪")
plt.axvline(n_90, color="gray", ls=":")
plt.xlabel("تعداد مؤلفه")
plt.ylabel("واریانس تجمعی")
plt.legend()
plt.show()
```

**مثال ۳ — PCA برای تجسم:**

```python
pca_2d = PCA(n_components=2).fit_transform(X_scaled)

plt.figure(figsize=(8, 6))
scatter = plt.scatter(pca_2d[:, 0], pca_2d[:, 1], c=y_d, cmap="tab10", s=8, alpha=0.7)
plt.colorbar(scatter, label="رقم")
plt.title("PCA — ۶۴ بعد به ۲ بعد")
plt.xlabel("مؤلفه ۱")
plt.ylabel("مؤلفه ۲")
plt.show()
```

**مثال ۴ — t-SNE در برابر PCA:**

```python
from sklearn.manifold import TSNE

tsne = TSNE(n_components=2, perplexity=30, random_state=42, init="pca")
X_tsne = tsne.fit_transform(X_scaled)

fig, axes = plt.subplots(1, 2, figsize=(13, 5))
for ax, data, title in [
    (axes[0], pca_2d, "PCA — ساختار خطی"),
    (axes[1], X_tsne, "t-SNE — خوشه‌های محلی"),
]:
    sc = ax.scatter(data[:, 0], data[:, 1], c=y_d, cmap="tab10", s=8, alpha=0.7)
    ax.set_title(title)
    ax.set_xticks([]); ax.set_yticks([])
plt.tight_layout()
plt.show()
```

**خروجی معمول:** t-SNE خوشه‌های جدا و واضح می‌دهد، PCA یک توده‌ی درهم. اما **فاصله‌های t-SNE را نمی‌توانی تفسیر کنی** — فقط جدایی خوشه‌ها معنا دارد.

**مثال ۵ — PCA به‌عنوان پیش‌پردازش برای مدل:**

```python
from sklearn.pipeline import make_pipeline
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import cross_val_score

for n in [10, 20, 30, 64]:
    pipe = make_pipeline(
        StandardScaler(), PCA(n_components=n),
        LogisticRegression(max_iter=2000, random_state=42)
    )
    score = cross_val_score(pipe, X_d, y_d, cv=3, n_jobs=-1).mean()
    print(f"{n:>2} مؤلفه → accuracy = {score:.3f}")
```

**درس:** کاهش بعد می‌تواند **دقت را حفظ یا حتی بهبود** بدهد (کاهش نویز) و سرعت را چند برابر کند.

### مقایسه و دیدگاه‌های مختلف

| روش | هدف | خطی؟ | ساختار حفظ‌شده | سرعت |
|---|---|---|---|---|
| PCA | فشرده‌سازی | بله | واریانس جهانی | سریع |
| t-SNE | تجسم | خیر | همسایگی محلی | کند |
| UMAP | تجسم + کاهش بعد | خیر | محلی + بخشی از جهانی | سریع |
| Autoencoder | فشرده‌سازی غیرخطی | خیر | یادگیری‌شده | متوسط |
| LDA | کاهش بعد با ناظر | بله | تفکیک کلاس‌ها | سریع |

**دو دیدگاه:**
- **«t-SNE برای تجسم، PCA برای مدل.»** قاعده‌ی عملی خوبی است.
- **«UMAP جای هر دو را می‌گیرد.»** تقریباً درست، اما UMAP هم فاصله‌های جهانی را دقیق حفظ نمی‌کند.

### اشتباهات رایج

**۱. PCA بدون استانداردسازی.** مؤلفه‌ی اول، فقط بزرگ‌ترین مقیاس را می‌بیند.
**۲. تفسیر مؤلفه‌ها به‌عنوان ویژگی‌های اصلی.** مؤلفه ترکیب خطی است، نه یک ویژگی.
**۳. استفاده از t-SNE به‌عنوان ورودی مدل.** برای تجسم است، نه پیش‌پردازش.
**۴. تفسیر فاصله‌ی بین خوشه‌ها در t-SNE.** بی‌معنی است.
**۵. `perplexity` نامناسب.** خیلی کوچک = خوشه‌های ریز، خیلی بزرگ = همه‌چیز یکی.
**۶. PCA روی داده‌ی گمشده بدون imputation.** خطا می‌دهد یا نتیجه غلط.

### نکات مهم و ترفندها

- **PCA: همیشه `StandardScaler` قبلش.**
- **t-SNE: `perplexity` را بین ۵ تا ۵۰ امتحان کن** و ببین کدام ساختار را بهتر نشان می‌دهد.
- **`init="pca"` در t-SNE** نتایج پایدارتری می‌دهد.
- **برای مدل، PCA؛ برای ارائه، t-SNE/UMAP.**
- **میان‌بر:** PCA = بیشترین واریانس · t-SNE = همسایگی محلی · هر دو = کاهش بعد، با اهداف متفاوت.

### خلاصه فصل

- PCA جهت‌های بیشترین واریانس را پیدا می‌کند و داده را فشرده می‌کند.
- استانداردسازی قبل از PCA واجب است.
- تعداد مؤلفه را با واریانس تجمعی (۹۰–۹۵٪) انتخاب کن.
- t-SNE برای تجسم است، نه مدل‌سازی؛ فاصله‌هایش بی‌معنی‌اند.
- کاهش بعد می‌تواند سرعت و حتی دقت را بهبود بدهد.

**سؤالات مروری:**
1. چرا PCA بدون استانداردسازی گمراه‌کننده است؟
2. چرا نمی‌توانی از t-SNE به‌عنوان ویژگی ورودی مدل استفاده کنی؟
3. اگر ۵۰ مؤلفه ۹۵٪ واریانس را توضیح دهند، چه می‌کنی؟

---

## فصل ۴۷: تشخیص ناهنجاری و کاربردهای واقعی

### مقدمه

«ناهنجاری» یعنی نمونه‌ای که با الگوی بقیه نمی‌خواند: تراکنش تقلبی، سنسور خراب، سرور نفوذشده، محصول معیوب. مشکل: **داده‌ی ناهنجار بسیار کم است** و اغلب برچسب ندارد.

این فصل، ابزارهای این مسئله را می‌دهد — و یک نکته‌ی مهم: **تشخیص ناهنجاری، اغلب صورت مسئله‌ی «یادگیری بدون ناظر» است، نه دسته‌بندی نامتوازن.**

### مفاهیم اساسی

**۱. سه رویکرد:**

| رویکرد | ایده | مثال |
|---|---|---|
| **آماری** | نقاط دور از توزیع | z-score، IQR |
| **فاصله‌محور** | نقاط دور از همسایه‌ها | KNN، LOF |
| **مدل‌محور** | نقاطی که مدل نمی‌تواند بازسازی کند | Autoencoder، Isolation Forest، One-Class SVM |

**۲. Isolation Forest — ایده‌ی زیبا.**

ناهنجاری‌ها **کم** و **متفاوت**اند، پس با تقسیم‌های تصادفی **سریع‌تر جدا می‌شوند**. اگر یک نقطه با چند تقسیم تصادفی جدا شود، ناهنجار است.

```python
IsolationForest(contamination=0.05)   # ۵٪ داده ناهنجار فرض می‌شود
```

**۳. Local Outlier Factor (LOF).** تراکم محلی هر نقطه را با همسایه‌هایش مقایسه می‌کند. اگر تراکمش خیلی کمتر باشد، ناهنجار است.

**۴. One-Class SVM.** مرزی دور داده‌ی «نرمال» می‌کشد؛ هر چیزی بیرون، ناهنجار.

**۵. Autoencoder.** شبکه‌ای که داده را فشرده و بازسازی می‌کند. روی داده‌ی نرمال آموزش می‌بیند؛ ناهنجاری، خطای بازسازی بالا دارد.

**۶. مسئله‌ی `contamination`.** درصد ناهنجاری مورد انتظار. اگر اشتباه تنظیم شود، یا پرت‌های زیاد می‌گیری یا هیچ.

**۷. ارزیابی بدون برچسب — سخت است.**
- با برچسب: precision/recall روی کلاس ناهنجار.
- بدون برچسب: بررسی دستی نمونه‌های علامت‌خورده، یا معیارهای مبتنی بر فاصله.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — سه روش، یک داده:**

```python
import numpy as np
import pandas as pd
from sklearn.ensemble import IsolationForest
from sklearn.neighbors import LocalOutlierFactor
from sklearn.svm import OneClassSVM
from sklearn.preprocessing import StandardScaler

rng = np.random.default_rng(42)
normal = rng.normal(0, 1, (500, 2))
outliers = rng.uniform(-6, 6, (25, 2))
X = np.vstack([normal, outliers])
y_true = np.array([0] * 500 + [1] * 25)

X_s = StandardScaler().fit_transform(X)

results = {}
results["IsolationForest"] = IsolationForest(
    contamination=0.05, random_state=42
).fit_predict(X_s)

results["LOF"] = LocalOutlierFactor(
    n_neighbors=20, contamination=0.05
).fit_predict(X_s)

results["OneClassSVM"] = OneClassSVM(
    nu=0.05, kernel="rbf", gamma="scale"
).fit_predict(X_s)

for name, pred in results.items():
    pred_bin = (pred == -1).astype(int)
    tp = ((pred_bin == 1) & (y_true == 1)).sum()
    fp = ((pred_bin == 1) & (y_true == 0)).sum()
    fn = ((pred_bin == 0) & (y_true == 1)).sum()
    precision = tp / (tp + fp) if tp + fp else 0
    recall = tp / (tp + fn) if tp + fn else 0
    print(f"{name:16s} | precision={precision:.2f} recall={recall:.2f} "
          f"| flags={pred_bin.sum()}")
```

**مثال ۲ — تشخیص تقلب با داده‌ی برچسب‌دار:**

```python
from sklearn.datasets import make_classification
from sklearn.ensemble import IsolationForest
from sklearn.metrics import classification_report
from sklearn.model_selection import train_test_split

X_f, y_f = make_classification(
    n_samples=5000, n_features=15, n_informative=8,
    weights=[0.98], flip_y=0.01, random_state=42
)
X_tr, X_te, y_tr, y_te = train_test_split(X_f, y_f, test_size=0.3, stratify=y_f, random_state=42)

# آموزش فقط روی داده‌ی نرمال (رویکرد بدون ناظر واقعی)
iso = IsolationForest(contamination=0.02, random_state=42)
iso.fit(X_tr[y_tr == 0])

pred = (iso.predict(X_te) == -1).astype(int)
print(classification_report(y_te, pred, target_names=["سالم", "تقلب"], digits=3))
```

**نکته:** `IsolationForest` فقط با داده‌ی نرمال آموزش دید و باز هم تقلب را پیدا می‌کند. این قدرت رویکرد بدون ناظر است.

**مثال ۳ — z-score ساده (همیشه اولین قدم):**

```python
import numpy as np
import pandas as pd

def zscore_outliers(series, threshold=3.0):
    z = (series - series.mean()) / series.std()
    return series.index[np.abs(z) > threshold]

df = pd.DataFrame({"amount": np.r_[np.random.default_rng(0).normal(100, 15, 500), [800, 950, 1200]]})
idx = zscore_outliers(df["amount"])
print("تعداد پرت با z-score:", len(idx))
print(df.loc[idx, "amount"].values)
```

**مثال ۴ — Autoencoder ساده (مفهومی با sklearn):**

```python
from sklearn.neural_network import MLPRegressor
from sklearn.preprocessing import StandardScaler

X_norm = X_s[y_true == 0]
scaler = StandardScaler().fit(X_norm)
X_norm_s = scaler.transform(X_norm)
X_all_s = scaler.transform(X_s)

# شبکه‌ای که ورودی را بازسازی می‌کند (۲ → ۸ → ۲)
ae = MLPRegressor(hidden_layer_sizes=(8,), max_iter=500, random_state=42)
ae.fit(X_norm_s, X_norm_s)          # هدف = خود ورودی!

recon_error = ((X_all_s - ae.predict(X_all_s)) ** 2).sum(axis=1)
threshold = np.percentile(recon_error[y_true == 0], 99)

flags = (recon_error > threshold).astype(int)
print(f"Autoencoder: {flags.sum()} نمونه علامت خورد "
      f"({((flags == 1) & (y_true == 1)).sum()} از {y_true.sum()} ناهنجاری واقعی)")
```

**این، ایده‌ی autoencoder در ۱۰ خط است.** در فایل ۱۷ با PyTorch نسخه‌ی واقعی‌اش را می‌بینیم.

### مقایسه و دیدگاه‌های مختلف

| روش | نیاز به برچسب | ابعاد بالا | تفسیر | سرعت |
|---|---|---|---|---|
| z-score | خیر | ضعیف | بالا | خیلی سریع |
| Isolation Forest | خیر | خوب | متوسط | سریع |
| LOF | خیر | ضعیف | متوسط | متوسط |
| One-Class SVM | خیر | متوسط | پایین | متوسط |
| Autoencoder | خیر | عالی | پایین | کند |

**یک دیدگاه مهم:** «اگر برچسب داری، از دسته‌بندی استفاده کن، نه تشخیص ناهنجاری.» درست است — اما اگر کلاس اقلیت فقط ۰.۱٪ باشد، مدل دسته‌بندی به‌سختی یاد می‌گیرد. **ترکیب هر دو** (ناهنجاری به‌عنوان ویژگی + دسته‌بندی) اغلب بهترین نتیجه را می‌دهد.

### اشتباهات رایج

**۱. `contamination` پیش‌فرض بدون بررسی.** نتیجه‌ی گمراه‌کننده.
**۲. آموزش روی داده‌ی آلوده.** اگر داده‌ی آموزش پرت دارد، مدل آن را نرمال می‌بیند. تا جای ممکن، فقط داده‌ی نرمال بده.
**۳. ارزیابی با accuracy.** با ۱٪ ناهنجاری، مدل «همه نرمال» دقت ۹۹٪ دارد و بی‌فایده است.
**۴. نادیده گرفتن هزینه‌ی هشدار کاذب.** در عمل، هر هشدار باید بررسی انسانی شود؛ هشدار زیاد = بی‌توجهی.
**۵. فرض این‌که ناهنجاری = تقلب.** ناهنجاری، «متفاوت» است، نه لزوماً «بد».
**۶. z-score روی داده‌ی چوله.** میانگین و انحراف معیار گمراه می‌شوند؛ از IQR یا روش‌های مقاوم استفاده کن.

### نکات مهم و ترفندها

- **اول z-score و IQR، بعد مدل.** ساده‌ترین روش‌ها اغلب کافی‌اند.
- **`IsolationForest` بهترین شروع** برای داده‌ی متوسط و ابعاد بالا.
- **`contamination` را از دانش دامنه بگیر:** «حدود ۲٪ تراکنش‌ها تقلبی است.»
- **آستانه را طوری تنظیم کن که تعداد هشدار قابل بررسی باشد.** ۵۰ هشدار در روز که کسی بررسی کند، بهتر از ۵۰۰۰ هشدار بی‌بررسی است.
- **میان‌بر:** ناهنجاری = دور از توده · بدون برچسب کار می‌کند · `contamination` را از دامنه بگیر.

### خلاصه فصل

- تشخیص ناهنجاری، اغلب بدون ناظر است چون برچسب کم است.
- Isolation Forest، LOF، One-Class SVM و Autoencoder ابزارهای اصلی‌اند.
- `contamination` تعیین‌کننده‌ی تعداد هشدار است.
- ارزیابی با accuracy گمراه‌کننده است؛ از precision/recall استفاده کن.
- ناهنجاری ≠ تقلب؛ «متفاوت» است، نه لزوماً «بد».

**سؤالات مروری:**
1. چرا Isolation Forest سریع‌تر ناهنجاری را جدا می‌کند؟
2. چرا باید مدل را فقط روی داده‌ی نرمال آموزش داد؟
3. `contamination` را از کجا تعیین می‌کنی؟

---

## منابع و مراجع

**مستندات**
- scikit-learn — *Clustering*, *Decomposition*, *Outlier detection*, *Manifold learning*
- scikit-learn — *Clustering performance evaluation* (silhouette)

**کتاب‌ها**
- Géron, A. — *Hands-On Machine Learning*، فصل ۸ (Dimensionality Reduction) و ۹ (Unsupervised)
- VanderPlas, J. — *Python Data Science Handbook*، فصل ۵

**مقالات کلیدی**
- MacQueen, J. (1967) — *K-Means*
- Ester, M. et al. (1996) — *DBSCAN*
- Liu, F. T. et al. (2008) — *Isolation Forest*
- van der Maaten, L. & Hinton, G. (2008) — *Visualizing Data using t-SNE*
- McInnes, L. et al. (2018) — *UMAP*

---

**بعدی:** [`11-Semi-Self-Supervised.md`](./11-Semi-Self-Supervised.md) — فصل ۴۸: یادگیری نیمه‌نظارتی
