# بخش ۱۴: ارزیابی مدل و معیارها

**مدت یادگیری تقریبی**: ۲ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۸ (مبانی ML)، فایل ۱۳ (آموزش)

---

## فهرست محتوا

- [فصل ۶۱: چرا «دقت» کافی نیست](#فصل-۶۱-چرا-دقت-کافی-نیست)
- [فصل ۶۲: معیارهای دسته‌بندی](#فصل-۶۲-معیارهای-دستهبندی)
- [فصل ۶۳: معیارهای رگرسیون](#فصل-۶۳-معیارهای-رگرسیون)
- [فصل ۶۴: Cross-Validation و انتخاب مدل](#فصل-۶۴-cross-validation-و-انتخاب-مدل)
- [فصل ۶۵: نشت داده و دام‌های اعتبارسنجی](#فصل-۶۵-نشت-داده-و-دامهای-اعتبارسنجی)

---

## فصل ۶۱: چرا «دقت» کافی نیست

### مقدمه

این جمله را زیاد می‌شنوی: «مدل ما ۹۹٪ دقت دارد.» اما ۹۹٪ در چه داده‌ای؟ با چه توزیعی؟ با چه هزینه‌ای برای خطا؟

**دقت (accuracy)، فریبنده‌ترین معیار در یادگیری ماشین است.** این فصل نشان می‌دهد چرا، و چه چیزی باید جایش استفاده کنی.

### مفاهیم اساسی

**۱. پارادوکس دقت — مثال کلاسیک.**

فرض کن سیستم تشخیص تقلب داری. در ۱۰٬۰۰۰ تراکنش، ۱۰۰ تراکنش تقلبی است (۱٪).

مدلی که **همه را سالم** پیش‌بینی کند:
```
دقت = ۹٬۹۰۰ / ۱۰٬۰۰۰ = ۹۹٪
```
اما این مدل **هیچ تقلبی را پیدا نمی‌کند**. کاملاً بی‌فایده است.

**این، پارادوکس دقت است:** در داده‌ی نامتوازن، دقت بالا می‌تواند با عملکرد فاجعه‌بار هم‌زمان باشد.

**۲. چرا دقت کافی نیست؟**

| مشکل | توضیح |
|---|---|
| **نامتوازنی** | کلاس اکثریت، همه‌چیز را تعیین می‌کند |
| **هزینه‌ی نامتقارن** | خطای مثبت کاذب و منفی کاذب، هزینه‌ی متفاوت دارند |
| **آستانه** | دقت به آستانه‌ی ۰.۵ وابسته است، که ممکن است غلط باشد |
| **توزیع** | دقت روی داده‌ی تست، توزیع واقعی را نشان نمی‌دهد |
| **اطمینان** | دقت نمی‌گوید مدل چقدر مطمئن است |

**۳. ماتریس درهم‌ریختگی (Confusion Matrix) — پایه‌ی همه‌چیز.**

|  | پیش‌بینی مثبت | پیش‌بینی منفی |
|---|---|---|
| **واقعاً مثبت** | TP (درست مثبت) | FN (منفی کاذب) |
| **واقعاً منفی** | FP (مثبت کاذب) | TN (درست منفی) |

چهار عدد، همه‌چیز را می‌گویند. هر معیاری از این چهار عدد ساخته می‌شود.

**۴. معیارها از ماتریس:**

```
Accuracy  = (TP + TN) / (TP + TN + FP + FN)
Precision = TP / (TP + FP)          ← «از میان آن‌هایی که مثبت گفتم، چند تا واقعاً مثبت بودند؟»
Recall    = TP / (TP + FN)          ← «از میان مثبت‌های واقعی، چند تا را پیدا کردم؟»
F1        = 2·(P·R) / (P + R)       ← میانگین هارمونیک
Specificity = TN / (TN + FP)        ← نرخ درست منفی
```

**۵. کدام معیار، کِی؟**

| سناریو | معیار مهم | چرا |
|---|---|---|
| تشخیص سرطان | **Recall** | از دست دادن بیمار، فاجعه است |
| فیلتر اسپم | **Precision** | حذف ایمیل مهم، بدتر از دیدن یک اسپم |
| تشخیص تقلب | **Recall** (با بررسی انسانی) | از دست دادن تقلب گران است |
| جست‌وجو | **Precision@k** | نتایج اول باید مرتبط باشند |
| دسته‌بندی متعادل | **F1 یا Accuracy** | هر دو معقول |
| رتبه‌بندی | **ROC-AUC / PR-AUC** | مستقل از آستانه |

**۶. معیارهای مستقل از آستانه:**

- **ROC-AUC:** مساحت زیر منحنی TPR در برابر FPR. ۰.۵ = تصادفی، ۱.۰ = کامل.
- **PR-AUC:** مساحت زیر منحنی Precision-Recall. **برای داده‌ی نامتوازن، بهتر از ROC-AUC.**
- **Log Loss:** کیفیت احتمالات، نه فقط تصمیم‌ها.

**۷. چرا PR-AUC برای داده‌ی نامتوازن بهتر است؟**
چون ROC-AUC با TN زیاد، خوش‌بینانه می‌شود. PR-AUC فقط روی کلاس مثبت تمرکز می‌کند.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — محاسبه‌ی دستی همه‌ی معیارها:**

```python
import numpy as np

def all_metrics(y_true, y_pred):
    y_true, y_pred = np.asarray(y_true), np.asarray(y_pred)
    tp = int(((y_true == 1) & (y_pred == 1)).sum())
    tn = int(((y_true == 0) & (y_pred == 0)).sum())
    fp = int(((y_true == 0) & (y_pred == 1)).sum())
    fn = int(((y_true == 1) & (y_pred == 0)).sum())

    eps = 1e-12
    precision = tp / (tp + fp + eps)
    recall = tp / (tp + fn + eps)
    f1 = 2 * precision * recall / (precision + recall + eps)
    accuracy = (tp + tn) / (tp + tn + fp + fn)
    specificity = tn / (tn + fp + eps)

    return {
        "TP": tp, "TN": tn, "FP": fp, "FN": fn,
        "accuracy": round(accuracy, 3),
        "precision": round(precision, 3),
        "recall": round(recall, 3),
        "f1": round(f1, 3),
        "specificity": round(specificity, 3),
    }

# مدل «همه سالم» روی داده‌ی نامتوازن
y_true = [0] * 990 + [1] * 10
y_pred = [0] * 1000
print("مدل همه-منفی:", all_metrics(y_true, y_pred))
# accuracy=0.99 اما recall=0.0 → بی‌فایده!
```

**مثال ۲ — مقایسه‌ی مدل‌ها با آستانه‌های مختلف:**

```python
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import precision_recall_fscore_support, roc_auc_score, average_precision_score
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import make_pipeline

X, y = make_classification(n_samples=5000, n_features=15, n_informative=8,
                           weights=[0.97], flip_y=0.01, random_state=42)
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.3, stratify=y, random_state=42)

model = make_pipeline(
    StandardScaler(),
    LogisticRegression(max_iter=1000, class_weight="balanced", random_state=42)
).fit(X_tr, y_tr)

probs = model.predict_proba(X_te)[:, 1]

print(f"{'آستانه':>8} {'P':>7} {'R':>7} {'F1':>7} {'تعداد مثبت':>12}")
for th in [0.1, 0.2, 0.3, 0.5, 0.7]:
    pred = (probs >= th).astype(int)
    p, r, f1, _ = precision_recall_fscore_support(y_te, pred, average="binary", zero_division=0)
    print(f"{th:>8.1f} {p:>7.3f} {r:>7.3f} {f1:>7.3f} {pred.sum():>12}")

print(f"\nROC-AUC: {roc_auc_score(y_te, probs):.3f}")
print(f"PR-AUC:  {average_precision_score(y_te, probs):.3f}")
```

**این جدول، درس اصلی فصل است:** با تغییر آستانه، P و R معامله می‌شوند. **آستانه، انتخاب کسب‌وکار است، نه پیش‌فرض کتابخانه.**

**مثال ۳ — ماتریس درهم‌ریختگی بصری:**

```python
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.metrics import confusion_matrix

pred = (probs >= 0.3).astype(int)
cm = confusion_matrix(y_te, pred)

plt.figure(figsize=(5, 4))
sns.heatmap(cm, annot=True, fmt="d", cmap="Blues",
            xticklabels=["سالم", "تقلب"], yticklabels=["سالم", "تقلب"])
plt.xlabel("پیش‌بینی")
plt.ylabel("واقعیت")
plt.title("ماتریس درهم‌ریختگی (آستانه ۰.۳)")
plt.show()
```

**مثال ۴ — انتخاب آستانه بر اساس هزینه:**

```python
def cost_based_threshold(probs, y_true, cost_fp=10, cost_fn=100):
    """آستانه‌ای که هزینه‌ی کل را کمینه می‌کند."""
    best_th, best_cost = 0.5, float("inf")

    for th in np.arange(0.01, 1.0, 0.01):
        pred = (probs >= th).astype(int)
        fp = ((pred == 1) & (y_true == 0)).sum()
        fn = ((pred == 0) & (y_true == 1)).sum()
        cost = fp * cost_fp + fn * cost_fn
        if cost < best_cost:
            best_cost, best_th = cost, th

    return round(best_th, 2), best_cost

# اگر از دست دادن تقلب ۱۰ برابر گران‌تر از هشدار کاذب باشد:
th, cost = cost_based_threshold(probs, y_te.values, cost_fp=10, cost_fn=100)
print(f"آستانه‌ی بهینه: {th} | هزینه‌ی کل: {cost}")
```

**این، روش درست انتخاب آستانه است:** هزینه‌ی هر نوع خطا را بنویس، آستانه‌ی کمینه‌کننده‌ی هزینه را پیدا کن.

### مقایسه و دیدگاه‌های مختلف

| معیار | مقاوم به نامتوازنی | مستقل از آستانه | تفسیر | کِی |
|---|---|---|---|---|
| Accuracy | ❌ | ❌ | آسان | داده‌ی متعادل |
| Precision | ✅ | ❌ | متوسط | هزینه‌ی FP بالا |
| Recall | ✅ | ❌ | متوسط | هزینه‌ی FN بالا |
| F1 | ✅ | ❌ | متوسط | تعادل P/R |
| ROC-AUC | متوسط | ✅ | سخت | مقایسه‌ی مدل‌ها |
| PR-AUC | ✅ | ✅ | متوسط | نامتوازن شدید |
| MCC | ✅ | ❌ | سخت | معیار متعادل تک‌عددی |

**یک دیدگاه:** «F1 همیشه بهترین است.» نه: F1 فرض می‌کند P و R هم‌ارزش‌اند. در تشخیص بیماری، recall خیلی مهم‌تر است.

### اشتباهات رایج

**۱. گزارش فقط accuracy.** در داده‌ی نامتوازن، بی‌معنی.
**۲. آستانه‌ی ۰.۵ بدون بررسی.** تقریباً همیشه اشتباه است.
**۳. استفاده از ROC-AUC روی داده‌ی خیلی نامتوازن.** PR-AUC بهتر است.
**۴. مقایسه‌ی مدل‌ها با معیارهای متفاوت.** باید یک معیار را ثابت کنی.
**۵. بهینه‌سازی معیار روی test.** نشت.
**۶. نادیده گرفتن هزینه‌ی خطا.** ۱٪ خطا در تشخیص سرطان با ۱٪ خطا در فیلتر اسپم برابر نیست.
**۷. گزارش یک عدد بدون بازه‌ی اطمینان.** با ۱۰۰ نمونه، ±۵٪ طبیعی است.

### نکات مهم و ترفندها

- **اول ماتریس درهم‌ریختگی، بعد هر معیاری.** چهار عدد، همه‌چیز را می‌گویند.
- **آستانه را روی validation انتخاب کن، روی test گزارش بده.**
- **`average_precision_score` را برای نامتوازن جدی بگیر.**
- **هزینه‌ی خطا را با ذی‌نفع بنویس.** «هر FN چقدر هزینه دارد؟»
- **میان‌بر:** Precision = «چقدر از حرف‌هایم درست بود» · Recall = «چقدر از واقعیت را گفتم».

### خلاصه فصل

- accuracy در داده‌ی نامتوازن گمراه‌کننده است.
- ماتریس درهم‌ریختگی، پایه‌ی همه‌ی معیارهاست.
- Precision و Recall معامله می‌شوند؛ انتخاب، تابع هزینه است.
- PR-AUC برای داده‌ی نامتوازن بهتر از ROC-AUC است.
- آستانه را بر اساس هزینه‌ی کسب‌وکار انتخاب کن.

**سؤالات مروری:**
1. چرا مدل «همه منفی» روی داده‌ی ۱٪ مثبت، accuracy=0.99 دارد؟
2. کِی Precision را ترجیح می‌دهی و کِی Recall؟
3. چرا PR-AUC برای داده‌ی نامتوازن مناسب‌تر است؟

---

## فصل ۶۲: معیارهای دسته‌بندی

### مقدمه

این فصل، عمیق‌تر به معیارهای دسته‌بندی می‌رود: از ماتریس درهم‌ریختگی تا منحنی‌های ROC و PR، و معیارهای خاص چندکلاسه.

هدف: بتوانی **دقیقاً** بگویی مدلت در چه شرایطی خوب است و در چه شرایطی نه.

### مفاهیم اساسی

**۱. TPR و FPR:**

```
TPR (Recall) = TP / (TP + FN)     ← نرخ تشخیص مثبت‌های واقعی
FPR = FP / (FP + TN)              ← نرخ هشدار کاذب
```

**۲. منحنی ROC.** TPR را در برابر FPR برای همه‌ی آستانه‌ها رسم می‌کند.

- نقطه‌ی (0,0): هیچ‌چیز مثبت نمی‌گوید.
- نقطه‌ی (1,1): همه را مثبت می‌گوید.
- خط مورب: تصادفی.
- گوشه‌ی بالا-چپ: ایده‌آل.

**۳. منحنی PR.** Precision در برابر Recall.

**برای داده‌ی نامتوازن، PR گویاتر است** چون FPR با TN زیاد، کوچک و بی‌اثر می‌شود.

**۴. معیارهای چندکلاسه — سه حالت میانگین:**

| حالت | روش | کِی |
|---|---|---|
| **macro** | میانگین ساده‌ی معیار هر کلاس | همه‌ی کلاس‌ها هم‌ارزش |
| **weighted** | میانگین وزن‌دار به فراوانی | فراوانی مهم است |
| **micro** | محاسبه روی همه‌ی نمونه‌ها | معادل accuracy در تک‌برچسبی |

**تله:** macro و weighted می‌توانند خیلی متفاوت باشند. همیشه هر دو را گزارش کن.

**۵. Cohen's Kappa.** توافق با پیش‌بینی، منهای توافق تصادفی. برای داده‌ی نامتوازن مفید.

```
κ = (p_o - p_e) / (1 - p_e)
```

**۶. Matthews Correlation Coefficient (MCC).** معیار متعادل بین -۱ و ۱ که همه‌ی چهار خانه را در نظر می‌گیرد. **بهترین معیار تک‌عددی برای داده‌ی نامتوازن.**

**۷. Brier Score.** کیفیت **کالیبراسیون** احتمالات:

```
Brier = (1/n)Σ(p - y)²
```
اگر مدل بگوید ۰.۸ و در ۸۰٪ موارد درست باشد، کالیبره است.

**۸. منحنی کالیبراسیون (Reliability Diagram).**
پیش‌بینی‌شده در برابر فراوانی واقعی. مدل کالیبره، روی خط ۴۵ درجه است.

**چرا مهم است؟** مدل‌های عمیق معمولاً **بیش‌اطمینان** هستند: می‌گویند ۰.۹۹، اما دقت ۰.۸۵.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — ROC و PR برای دو مدل:**

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import roc_curve, precision_recall_curve, auc, average_precision_score
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import make_pipeline

X, y = make_classification(n_samples=4000, n_features=15, n_informative=8,
                           weights=[0.95], flip_y=0.01, random_state=42)
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.3, stratify=y, random_state=42)

models = {
    "Logistic": make_pipeline(StandardScaler(),
                              LogisticRegression(max_iter=1000, class_weight="balanced")),
    "Forest": RandomForestClassifier(n_estimators=300, class_weight="balanced",
                                     random_state=42),
}

fig, axes = plt.subplots(1, 2, figsize=(13, 5))

for name, m in models.items():
    m.fit(X_tr, y_tr)
    probs = m.predict_proba(X_te)[:, 1]

    fpr, tpr, _ = roc_curve(y_te, probs)
    axes[0].plot(fpr, tpr, label=f"{name} (AUC={auc(fpr, tpr):.3f})")

    prec, rec, _ = precision_recall_curve(y_te, probs)
    axes[1].plot(rec, prec, label=f"{name} (AP={average_precision_score(y_te, probs):.3f})")

axes[0].plot([0, 1], [0, 1], "k--", label="تصادفی")
axes[0].set_xlabel("FPR"); axes[0].set_ylabel("TPR"); axes[0].set_title("ROC")
axes[0].legend()

axes[1].set_xlabel("Recall"); axes[1].set_ylabel("Precision"); axes[1].set_title("Precision-Recall")
axes[1].legend()

plt.tight_layout()
plt.show()
```

**مثال ۲ — گزارش کامل چندکلاسه:**

```python
from sklearn.datasets import load_iris
from sklearn.metrics import classification_report, cohen_kappa_score, matthews_corrcoef

iris = load_iris()
X_i, y_i = iris.data, iris.target
X_tr, X_te, y_tr, y_te = train_test_split(X_i, y_i, test_size=0.3, stratify=y_i, random_state=42)

clf = RandomForestClassifier(n_estimators=200, random_state=42).fit(X_tr, y_te if False else y_tr)
pred = clf.predict(X_te)

print(classification_report(y_te, pred, target_names=iris.target_names, digits=3))
print("Cohen's Kappa:", round(cohen_kappa_score(y_te, pred), 3))
print("MCC:", round(matthews_corrcoef(y_te, pred), 3))
```

**نکته:** در `classification_report`، به تفاوت `macro avg` و `weighted avg` دقت کن. اگر کلاس کوچک ضعیف باشد، `macro` آن را نشان می‌دهد اما `weighted` پنهانش می‌کند.

**مثال ۳ — کالیبراسیون:**

```python
from sklearn.calibration import calibration_curve

model = models["Forest"]
probs = model.predict_proba(X_te)[:, 1]

prob_true, prob_pred = calibration_curve(y_te, probs, n_bins=10, strategy="quantile")

plt.figure(figsize=(6, 5))
plt.plot(prob_pred, prob_true, "o-", label="مدل")
plt.plot([0, 1], [0, 1], "k--", label="کالیبره‌ی کامل")
plt.xlabel("احتمال پیش‌بینی‌شده")
plt.ylabel("فراوانی واقعی")
plt.title("منحنی کالیبراسیون")
plt.legend()
plt.show()

print("اگر نقاط زیر خط باشند → بیش‌اطمینان")
print("اگر بالای خط باشند → کم‌اطمینان")
```

**مثال ۴ — کالیبراسیون با Platt Scaling:**

```python
from sklearn.calibration import CalibratedClassifierCV

calibrated = CalibratedClassifierCV(
    RandomForestClassifier(n_estimators=200, random_state=42),
    method="sigmoid", cv=3
).fit(X_tr, y_tr)

p_cal = calibrated.predict_proba(X_te)[:, 1]
print("Brier خام:", round(np.mean((probs - y_te) ** 2), 4))
print("Brier کالیبره:", round(np.mean((p_cal - y_te) ** 2), 4))
```

### مقایسه و دیدگاه‌های مختلف

| معیار | دامنه | نامتوازن | کالیبراسیون | تفسیر |
|---|---|---|---|---|
| Accuracy | [0,1] | ❌ | ❌ | آسان |
| F1 | [0,1] | ✅ | ❌ | متوسط |
| ROC-AUC | [0,1] | متوسط | ❌ | سخت |
| PR-AUC | [0,1] | ✅ | ❌ | متوسط |
| MCC | [-1,1] | ✅ | ❌ | سخت |
| Brier | [0,1] | ✅ | ✅ | متوسط |
| Log Loss | [0,∞) | ✅ | ✅ | سخت |

**یک دیدگاه:** «MCC بهترین معیار تک‌عددی است.» در بسیاری از مقالات، بله — چون همه‌ی چهار خانه را متعادل در نظر می‌گیرد. اما برای گزارش به کسب‌وکار، precision/recall قابل فهم‌ترند.

### اشتباهات رایج

**۱. گزارش فقط ROC-AUC.** در داده‌ی خیلی نامتوازن، خوش‌بینانه است.
**۲. نادیده گرفتن کالیبراسیون.** احتمال ۰.۹ ممکن است ۰.۷ باشد.
**۳. مقایسه‌ی macro و weighted بدون توجه.** نتیجه‌گیری متفاوت می‌دهند.
**۴. `predict` به جای `predict_proba` وقتی آستانه مهم است.**
**۵. استفاده از مدل کالیبره‌نشده برای تصمیم‌گیری ریسک.** در بیمه و پزشکی خطرناک است.
**۶. اعتماد به AUC بالای یک مدل بدون بررسی منحنی.** AUC می‌تواند خوب باشد اما در ناحیه‌ی مهم (recall بالا) بد عمل کند.

### نکات مهم و ترفندها

- **همیشه ماتریس + چند معیار + منحنی** را با هم گزارش کن.
- **برای داده‌ی نامتوازن: PR-AUC و MCC.**
- **برای تصمیم‌گیری ریسک: کالیبراسیون و Brier.**
- **`CalibratedClassifierCV` را برای مدل‌های درختی امتحان کن** (اغلب بیش‌اطمینان‌اند).
- **میان‌بر:** ROC = تشخیص · PR = نامتوازن · Brier = کالیبراسیون.

### خلاصه فصل

- ROC منحنی TPR/FPR است؛ PR برای داده‌ی نامتوازن گویاتر.
- macro/weighted/micro سه حالت میانگین چندکلاسه‌اند.
- MCC معیار متعادل تک‌عددی است.
- Brier و منحنی کالیبراسیون، کیفیت احتمال را می‌سنجند.
- مدل‌های عمیق و درختی معمولاً بیش‌اطمینان‌اند.

**سؤالات مروری:**
1. تفاوت ROC-AUC و PR-AUC چیست؟
2. کالیبراسیون یعنی چه و چرا مهم است؟
3. تفاوت `macro avg` و `weighted avg` را با مثال توضیح بده.

---

## فصل ۶۳: معیارهای رگرسیون

### مقدمه

در رگرسیون، «خطا» یک عدد است، اما **چطور آن را خلاصه کنیم**، تصمیم مهمی است. MSE، MAE، RMSE و R² هر کدام داستان متفاوتی می‌گویند.

این فصل کوتاه است، اما هر عددی که در گزارش مدل رگرسیونی می‌نویسی، از اینجا می‌آید.

### مفاهیم اساسی

**۱. چهار معیار اصلی:**

```
MAE  = (1/n)Σ|yᵢ - ŷᵢ|                  ← میانگین خطای مطلق (واحد اصلی)
MSE  = (1/n)Σ(yᵢ - ŷᵢ)²                 ← میانگین مربع خطا
RMSE = √MSE                              ← ریشه‌ی MSE (واحد اصلی)
R²   = 1 - SS_res/SS_tot                 ← نسبت واریانس توضیح‌داده‌شده
```

**۲. تفاوت‌های کلیدی:**

| معیار | واحد | حساسیت به پرت | تفسیر |
|---|---|---|---|
| MAE | همان واحد y | کم | «میانگین خطا چقدر است؟» |
| MSE | مربع واحد | **زیاد** | جریمه‌ی خطاهای بزرگ |
| RMSE | همان واحد y | زیاد | مثل MAE اما جریمه‌ی پرت |
| R² | بی‌واحد | متوسط | «چقدر بهتر از میانگین؟» |

**۳. R² — تفسیر دقیق:**

- `R² = 1`: پیش‌بینی کامل.
- `R² = 0`: به اندازه‌ی پیش‌بینی با میانگین.
- `R² < 0`: **بدتر از میانگین.** بله ممکن است! (روی test با مدل بد)

**تله:** R² با افزایش تعداد ویژگی‌ها **همیشه** بالا می‌رود (حتی ویژگی‌های بی‌معنی). برای مقایسه‌ی مدل‌ها با تعداد ویژگی متفاوت، از **Adjusted R²** استفاده کن:

```
R²_adj = 1 - (1-R²)(n-1)/(n-p-1)
```

**۴. MAPE — خطای درصدی:**

```
MAPE = (100/n)Σ|yᵢ - ŷᵢ|/|yᵢ|
```
- **مزیت:** قابل فهم برای کسب‌وکار («میانگین ۸٪ خطا»).
- **عیب:** با `y` نزدیک صفر، منفجر می‌شود.

**۵. معیارهای دیگر:**
- **MedAE:** میانه‌ی خطای مطلق — مقاوم‌تر به پرت.
- **RMSLE:** برای هدف‌های با دامنه‌ی بزرگ (لگاریتمی).
- **Quantile Loss:** برای پیش‌بینی چندک (مثلاً صدک ۹۰).

**۶. کدام معیار، کِی؟**

| سناریو | معیار | چرا |
|---|---|---|
| خطای متقارن، بدون پرت | MAE | تفسیر مستقیم |
| خطای بزرگ مهم است | RMSE | جریمه‌ی پرت |
| گزارش به مدیر | MAPE | درصد، قابل فهم |
| مقایسه‌ی مدل‌ها | R² | بی‌واحد |
| داده‌ی پرت | MedAE | مقاوم |
| قیمت (دامنه‌ی بزرگ) | RMSLE | مقیاس لگاریتمی |

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — همه‌ی معیارها با هم:**

```python
import numpy as np

def regression_metrics(y_true, y_pred):
    y_true, y_pred = np.asarray(y_true, float), np.asarray(y_pred, float)
    err = y_true - y_pred

    mae = np.mean(np.abs(err))
    mse = np.mean(err ** 2)
    rmse = np.sqrt(mse)
    medae = np.median(np.abs(err))

    ss_res = np.sum(err ** 2)
    ss_tot = np.sum((y_true - y_true.mean()) ** 2)
    r2 = 1 - ss_res / ss_tot if ss_tot else 0.0

    mask = y_true != 0
    mape = np.mean(np.abs(err[mask] / y_true[mask])) * 100 if mask.any() else np.nan

    return {
        "MAE": round(mae, 3), "MSE": round(mse, 3), "RMSE": round(rmse, 3),
        "MedAE": round(medae, 3), "R2": round(r2, 4), "MAPE%": round(mape, 2),
    }

y_true = np.array([100, 200, 300, 400, 500, 600])
y_pred = np.array([110, 190, 310, 380, 520, 900])     # یک خطای بزرگ

print(regression_metrics(y_true, y_pred))
```

**مثال ۲ — اثر پرت بر معیارها:**

```python
y_true = np.array([100, 200, 300, 400, 500])
y_pred_clean = np.array([105, 195, 310, 390, 510])       # خطاهای کوچک
y_pred_outlier = np.array([105, 195, 310, 390, 900])     # یک خطای بزرگ

print("بدون پرت:", regression_metrics(y_true, y_pred_clean))
print("با پرت:  ", regression_metrics(y_true, y_pred_outlier))
```

**نتیجه‌ی معمول:**
- MAE از ~۸ به ~۹۰ می‌رود (۱۰ برابر).
- RMSE از ~۹ به ~۱۸۰ می‌رود (۲۰ برابر).
- **RMSE بسیار حساس‌تر است.**

**مثال ۳ — R² منفی (چرا ممکن است):**

```python
y_true = np.array([10, 20, 30, 40, 50])
y_bad = np.array([100, 100, 100, 100, 100])     # پیش‌بینی ثابت و غلط
y_mean = np.full_like(y_true, y_true.mean())

print("پیش‌بینی میانگین:", round(regression_metrics(y_true, y_mean)["R2"], 3))   # 0.0
print("پیش‌بینی بد:     ", round(regression_metrics(y_true, y_bad)["R2"], 3))    # منفی!
```

**مثال ۴ — Adjusted R²:**

```python
def adjusted_r2(r2, n, p):
    return 1 - (1 - r2) * (n - 1) / (n - p - 1)

print("R²=0.90, n=100, p=5 →", round(adjusted_r2(0.90, 100, 5), 4))
print("R²=0.90, n=100, p=50 →", round(adjusted_r2(0.90, 100, 50), 4))
# با ۵۰ ویژگی، R² تعدیل‌شده خیلی کمتر است → هشدار بیش‌برازش
```

**مثال ۵ — MAPE و تله‌ی صفر:**

```python
y_true = np.array([0.0, 100.0, 200.0])
y_pred = np.array([5.0, 110.0, 190.0])
# MAPE روی مقدار صفر، تقسیم بر صفر → inf یا nan
```

**همیشه قبل از MAPE، صفرها را بررسی کن.**

### مقایسه و دیدگاه‌های مختلف

| معیار | واحدپذیر | پرت | تفسیر کسب‌وکار |
|---|---|---|---|
| MAE | ✅ | مقاوم | خوب |
| RMSE | ✅ | حساس | خوب |
| MAPE | درصد | متوسط | عالی |
| R² | ❌ | متوسط | متوسط |
| MedAE | ✅ | خیلی مقاوم | خوب |

**یک دیدگاه:** «MAE را گزارش کن، RMSE را برای آموزش.» دلیل: MAE برای انسان قابل فهم است («میانگین ۱۲ هزار تومان خطا»)، اما RMSE برای بهینه‌سازی مناسب‌تر است (مشتق‌پذیر و جریمه‌ی پرت).

### اشتباهات رایج

**۱. مقایسه‌ی RMSE بین دیتاست‌های مختلف.** واحدها متفاوت‌اند.
**۲. MAPE روی داده‌ی با صفر یا نزدیک صفر.** منفجر می‌شود.
**۳. R² بالا با ویژگی‌های زیاد.** Adjusted R² استفاده کن.
**۴. نادیده گرفتن جهت خطا.** MAE می‌گوید «چقدر»، نه «کجا». باقیمانده‌ها را رسم کن.
**۵. گزارش یک عدد بدون توزیع خطا.** میانگین خطا می‌تواند الگوی مهمی را پنهان کند.
**۶. مقایسه‌ی مدل روی train.** باید test باشد.

### نکات مهم و ترفندها

- **همیشه MAE و RMSE را با هم گزارش کن.** تفاوت زیادشان = پرت.
- **باقیمانده‌ها را رسم کن:** `residual = y_true - y_pred`. الگو در باقیمانده = مدل ناقص.
- **برای کسب‌وکار، MAPE بده** (با محافظت از صفر).
- **`np.sqrt(mean_squared_error(...))`** برای RMSE؛ sklearn معادل مستقیم ندارد.
- **میان‌بر:** MAE = میانگین خطا · RMSE = جریمه‌ی پرت · R² = بهتر از میانگین.

### خلاصه فصل

- MAE میانگین خطای مطلق، RMSE جریمه‌ی پرت را بیشتر می‌کند.
- R² نسبت واریانس توضیح‌داده‌شده است و می‌تواند منفی شود.
- MAPE برای کسب‌وکار خوب است، اما با صفر می‌شکند.
- Adjusted R² برای مقایسه‌ی مدل با تعداد ویژگی متفاوت.
- باقیمانده‌ها را رسم کن؛ عدد تنها کافی نیست.

**سؤالات مروری:**
1. چرا RMSE به پرت حساس‌تر از MAE است؟
2. R² منفی چه معنایی دارد؟
3. چرا MAPE روی داده‌ی با صفر مشکل‌دار است؟

---

## فصل ۶۴: Cross-Validation و انتخاب مدل

### مقدمه

یک تقسیم train/test، یک **برآورد تصادفی** می‌دهد. اگر داده را جور دیگری تقسیم کنی، ممکن است نتیجه ۵٪ تفاوت کند. **Cross-Validation** این وابستگی به شانس را کم می‌کند.

این فصل، ابزار درست انتخاب مدل را می‌دهد.

### مفاهیم اساسی

**۱. K-Fold Cross-Validation:**

```
داده را به K بخش تقسیم کن
برای i = 1..K:
    آموزش روی K-1 بخش
    ارزیابی روی بخش i
نتیجه: میانگین ± انحراف معیار
```

**۲. انواع CV:**

| نوع | کِی | نکته |
|---|---|---|
| **KFold** | رگرسیون، داده‌ی متعادل | تصادفی |
| **StratifiedKFold** | دسته‌بندی نامتوازن | حفظ نسبت کلاس |
| **GroupKFold** | داده‌ی گروهی (بیمار، کاربر) | گروه‌ها جدا |
| **TimeSeriesSplit** | سری زمانی | گذشته → آینده |
| **LeaveOneOut** | داده‌ی خیلی کم | K=n، کند |
| **RepeatedKFold** | برآورد پایدارتر | چند بار تکرار |

**۳. انتخاب K:**
- `K=5` یا `K=10` استاندارد است.
- `K` بزرگ‌تر: برآورد دقیق‌تر، محاسبه‌ی بیشتر، واریانس بیشتر در هر fold.
- `K=n` (LOO): تقریباً بدون بایاس، اما واریانس بالا و بسیار کند.

**۴. تنظیم فراپارامتر با CV:**

```python
GridSearchCV(model, param_grid, cv=5, scoring="f1")
```
- **Grid Search:** همه‌ی ترکیب‌ها. کامل، کند.
- **Random Search:** نمونه‌گیری تصادفی. سریع‌تر، اغلب به‌قدر کافی خوب.
- **Bayesian Optimization:** هوشمند. بهترین برای فضای بزرگ (Optuna).

**۵. قاعده‌ی «یک استاندارد طلایی» (One Standard Error Rule).**
اگر چند مدل در یک انحراف معیار از بهترین هستند، **ساده‌ترین** را انتخاب کن. پیچیدگی اضافه، ارزشش را ندارد.

**۶. Nested CV.** وقتی هم تنظیم و هم ارزیابی می‌خواهی:
- حلقه‌ی بیرونی: ارزیابی
- حلقه‌ی درونی: تنظیم
**بدون آن، برآوردت خوش‌بینانه است** (چون فراپارامتر را روی همان داده‌ای تنظیم کردی که ارزیابی می‌کنی).

**۷. مقایسه‌ی آماری مدل‌ها.**
اگر دو مدل ۰.۸۵ و ۰.۸۶ دارند، آیا تفاوت معنادار است؟ با انحراف معیار ±۰.۰۳، نه. **همیشه میانگین ± انحراف را گزارش کن.**

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — CV پایه:**

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.model_selection import cross_val_score, StratifiedKFold
from sklearn.ensemble import RandomForestClassifier

X, y = make_classification(n_samples=1500, n_features=15, n_informative=8,
                           weights=[0.85], random_state=42)

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(
    RandomForestClassifier(n_estimators=200, random_state=42),
    X, y, cv=cv, scoring="f1"
)

print("نتایج هر fold:", scores.round(3))
print(f"میانگین: {scores.mean():.3f} ± {scores.std():.3f}")
print(f"بازه‌ی اطمینان ۹۵٪: [{scores.mean()-2*scores.std():.3f}, {scores.mean()+2*scores.std():.3f}]")
```

**مثال ۲ — GridSearchCV با چند معیار:**

```python
from sklearn.model_selection import GridSearchCV

param_grid = {
    "n_estimators": [100, 300],
    "max_depth": [None, 5, 10],
    "min_samples_leaf": [1, 5],
}

gs = GridSearchCV(
    RandomForestClassifier(random_state=42),
    param_grid,
    cv=StratifiedKFold(5, shuffle=True, random_state=42),
    scoring={"f1": "f1", "roc_auc": "roc_auc", "recall": "recall"},
    refit="f1",
    n_jobs=-1,
    verbose=1,
)
gs.fit(X, y)

print("بهترین پارامترها:", gs.best_params_)
print(f"بهترین F1 (CV): {gs.best_score_:.3f}")
print(f"ROC-AUC (CV):  {gs.cv_results_['mean_test_roc_auc'][gs.best_index_]:.3f}")
```

**مثال ۳ — مقایسه‌ی مدل‌ها با خطای استاندارد:**

```python
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler

candidates = {
    "Logistic": make_pipeline(StandardScaler(), LogisticRegression(max_iter=1000)),
    "Forest": RandomForestClassifier(n_estimators=300, random_state=42),
    "GBM": GradientBoostingClassifier(n_estimators=200, learning_rate=0.05,
                                      max_depth=3, random_state=42),
}

results = {}
for name, m in candidates.items():
    s = cross_val_score(m, X, y, cv=cv, scoring="f1", n_jobs=-1)
    results[name] = (s.mean(), s.std())
    print(f"{name:9s}: {s.mean():.3f} ± {s.std():.3f}")

best = max(results, key=lambda k: results[k][0])
best_mean, best_std = results[best]
print(f"\nبهترین: {best}")
print("مدل‌های در یک خطای استاندارد از بهترین (One-SE):")
for name, (m, s) in results.items():
    if best_mean - m <= best_std:
        print(f"  - {name}: {m:.3f}")
```

**قاعده:** از میان مدل‌های درون یک SE، ساده‌ترین را انتخاب کن.

**مثال ۴ — TimeSeriesSplit:**

```python
from sklearn.model_selection import TimeSeriesSplit
import numpy as np

tscv = TimeSeriesSplit(n_splits=4, test_size=50)
X_ts = np.arange(300).reshape(-1, 1)
y_ts = np.arange(300) * 1.5 + np.random.default_rng(0).normal(0, 5, 300)

for i, (tr, te) in enumerate(tscv.split(X_ts)):
    print(f"Fold {i+1}: train={tr[0]}..{tr[-1]} ({len(tr)}) | "
          f"test={te[0]}..{te[-1]} ({len(te)})")
```

**مثال ۵ — Nested CV (ارزیابی صادقانه):**

```python
from sklearn.model_selection import cross_val_score

inner_cv = StratifiedKFold(3, shuffle=True, random_state=1)
outer_cv = StratifiedKFold(5, shuffle=True, random_state=2)

nested_scores = cross_val_score(
    GridSearchCV(RandomForestClassifier(random_state=42),
                 {"max_depth": [3, 5, 10]},
                 cv=inner_cv, scoring="f1"),
    X, y, cv=outer_cv, scoring="f1", n_jobs=-1
)

print(f"Nested CV F1: {nested_scores.mean():.3f} ± {nested_scores.std():.3f}")
print("این عدد، برآورد صادقانه‌تری از عملکرد واقعی است.")
```

### مقایسه و دیدگاه‌های مختلف

| روش | سرعت | دقت برآورد | کِی |
|---|---|---|---|
| Hold-out | خیلی سریع | پایین | داده‌ی بزرگ |
| 5-Fold CV | متوسط | خوب | پیش‌فرض |
| 10-Fold CV | کندتر | بهتر | داده‌ی متوسط |
| Repeated CV | خیلی کند | بهترین | تصمیم نهایی |
| Nested CV | خیلی خیلی کند | صادقانه‌ترین | گزارش نهایی |

**یک دیدگاه:** «CV برای همه‌چیز کافی است.» نه: برای داده‌ی زمانی یا گروهی، CV معمولی **نشت** می‌دهد.

### اشتباهات رایج

**۱. `KFold` روی سری زمانی.** آینده در train.
**۲. `KFold` روی داده‌ی گروهی.** نشت بین گروه‌ها.
**۳. پیش‌پردازش قبل از CV.** باید داخل Pipeline باشد.
**۴. گزارش بهترین نتیجه‌ی یک fold.** باید میانگین ± انحراف.
**۵. تنظیم و ارزیابی روی یک CV.** خوش‌بینانه. Nested CV لازم است.
**۶. `shuffle=True` در سری زمانی.** فاجعه.
**۷. مقایسه‌ی مدل‌ها با CV متفاوت.** باید یک CV ثابت.

### نکات مهم و ترفندها

- **`StratifiedKFold` را برای دسته‌بندی پیش‌فرض بگیر.**
- **`n_jobs=-1` را فراموش نکن.** چند برابر سریع‌تر.
- **`return_train_score=True`** برای تشخیص بیش‌برازش.
- **`RandomizedSearchCV` را برای فضای بزرگ امتحان کن** (با `n_iter=50`).
- **Optuna را برای تنظیم جدی یاد بگیر.**
- **میان‌بر:** K-Fold + میانگین ± انحراف = برآورد قابل اعتماد.

### خلاصه فصل

- CV برآورد پایدارتری از یک تقسیم می‌دهد.
- انواع CV برای انواع داده: stratified، group، time series.
- Grid/Random/Bayesian برای تنظیم فراپارامتر.
- Nested CV برای ارزیابی صادقانه.
- همیشه میانگین ± انحراف را گزارش کن.

**سؤالات مروری:**
1. چرا KFold برای سری زمانی غلط است؟
2. Nested CV چه مشکلی را حل می‌کند؟
3. قاعده‌ی One-SE چیست؟

---

## فصل ۶۵: نشت داده و دام‌های اعتبارسنجی

### مقدمه

**نشت داده (Data Leakage)**، قاتل خاموش پروژه‌های ML است. مدل در آزمایش عالی کار می‌کند، در تولید فاجعه است — و کسی نمی‌داند چرا.

این فصل، جامع‌ترین فهرست دام‌ها را می‌دهد. اگر فقط یک فصل از این بخش را دوبار بخوانی، همین باشد.

### مفاهیم اساسی

**۱. تعریف.** نشت داده یعنی **اطلاعاتی از داده‌ی آزمون (یا آینده) در آموزش وارد شود**.

**۲. هفت نوع نشت:**

| نوع | مثال | نشانه |
|---|---|---|
| **نشت هدف** | ویژگی‌ای که جواب را لو می‌دهد | دقت غیرواقعی بالا (۹۹٪+) |
| **نشت پیش‌پردازش** | `fit` روی کل داده | تفاوت کوچک اما واقعی |
| **نشت تقسیم** | رکورد تکراری در train و test | ارزیابی خوش‌بینانه |
| **نشت زمانی** | استفاده از داده‌ی آینده | عملکرد عالی آفلاین، بد آنلاین |
| **نشت گروهی** | چند رکورد از یک موجودیت در دو طرف | ارزیابی خوش‌بینانه |
| **نشت انتخاب ویژگی** | انتخاب ویژگی روی کل داده | دقت متورم |
| **نشت هدف‌محور در کدگذاری** | Target Encoding روی کل داده | بهبود غیرواقعی |

**۳. نشت هدف — خطرناک‌ترین.**

قاعده: **برای هر ویژگی بپرس «در لحظه‌ی تصمیم‌گیری، این مقدار را دارم؟»**

مثال‌های واقعی:
- پیش‌بینی ریزش با `cancellation_date` (فقط بعد از ریزش ثبت می‌شود).
- پیش‌بینی تقلب با `manual_review_status` (فقط بعد از بررسی).
- پیش‌بینی بیماری با `prescribed_drug` (بعد از تشخیص تجویز شده).
- پیش‌بینی قیمت خانه با `final_sale_price` (البته!).

**۴. دام‌های اعتبارسنجی:**

- **آلودگی test:** هر بار که مدل را با test تنظیم کنی، test آلوده می‌شود.
- **تکرار استفاده از test:** «test را ۵۰ بار نگاه کردم» = overfitting به test.
- **توزیع متفاوت:** test از توزیع دیگری (مثلاً سال دیگر) — عملکرد افت می‌کند.
- **نمونه‌ی کوچک:** ۳۰ نمونه، تفاوت ۵٪ معنادار نیست.
- **benchmark سوگیرانه:** مدل‌ها روی همان معیار تنظیم می‌شوند که گزارش می‌کنند.

**۵. نشانه‌های نشت:**
- دقت ۹۹٪+ در مسئله‌ی سخت.
- شکاف بزرگ بین CV و عملکرد واقعی.
- مدل ساده‌ای که «خیلی خوب» کار می‌کند.
- اهمیت بالای یک ویژگی عجیب.

**۶. چگونه نشت را پیدا کنیم؟**
1. **بازبینی فهرست ویژگی‌ها** با سؤال «زمان تصمیم».
2. **حذف ویژگی مشکوک و مقایسه‌ی عملکرد.** اگر افت شدید داشت، نشت بود.
3. **بررسی اهمیت ویژگی.** ویژگی با اهمیت غیرمنتظره = مشکوک.
4. **تست با داده‌ی جدید** (مثلاً ماه بعد).
5. **Pipeline اجباری** برای جلوگیری ساختاری.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — نمایش نشت هدف:**

```python
import numpy as np
import pandas as pd
from sklearn.model_selection import cross_val_score
from sklearn.ensemble import RandomForestClassifier

rng = np.random.default_rng(42)
n = 1000

df = pd.DataFrame({
    "age": rng.integers(18, 70, n),
    "usage": rng.normal(50, 15, n),
    "churn": rng.integers(0, 2, n),
})
# ویژگی نشت‌دهنده: «تاریخ لغو» — فقط برای کسانی که لغو کرده‌اند پر است
df["cancellation_reason"] = np.where(df["churn"] == 1, "price", None)
df["has_cancellation"] = df["cancellation_reason"].notna().astype(int)

X_leak = df[["age", "usage", "has_cancellation"]]
X_clean = df[["age", "usage"]]
y = df["churn"]

for name, X in [("با نشت", X_leak), ("بدون نشت", X_clean)]:
    s = cross_val_score(RandomForestClassifier(n_estimators=100, random_state=42),
                        X, y, cv=5, scoring="f1")
    print(f"{name:12s}: F1 = {s.mean():.3f}")
```

**خروجی معمول:** با نشت، F1 نزدیک ۱.۰؛ بدون نشت، حدود ۰.۵. **این، چهره‌ی نشت است.**

**مثال ۲ — نشت پیش‌پردازش:**

```python
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.pipeline import make_pipeline
from sklearn.model_selection import train_test_split

X = rng.normal(size=(500, 10))
y = rng.integers(0, 2, 500)

# ---------- اشتباه ----------
scaler = StandardScaler().fit(X)              # نشت: روی کل داده
X_scaled = scaler.transform(X)
X_tr, X_te, y_tr, y_te = train_test_split(X_scaled, y, test_size=0.2, random_state=42)
wrong = LogisticRegression(max_iter=1000).fit(X_tr, y_tr).score(X_te, y_te)

# ---------- درست ----------
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, random_state=42)
right = make_pipeline(StandardScaler(), LogisticRegression(max_iter=1000))
right.fit(X_tr, y_tr)
right_score = right.score(X_te, y_te)

print(f"اشتباه: {wrong:.4f} | درست: {right_score:.4f}")
print("تفاوت ممکن است کوچک باشد، اما در داده‌ی کم و ویژگی زیاد، بزرگ می‌شود.")
```

**مثال ۳ — نشت زمانی:**

```python
import pandas as pd

dates = pd.date_range("2023-01-01", periods=730, freq="D")
df_ts = pd.DataFrame({
    "date": dates,
    "value": np.cumsum(rng.normal(0, 1, 730)) + 100,
})
df_ts["target"] = df_ts["value"].shift(-1)      # پیش‌بینی فردا

# اشتباه: تصادفی
X_wrong = df_ts[["value"]].dropna()
y_wrong = df_ts["target"].dropna()

# درست: زمانی
split = int(len(df_ts) * 0.8)
train = df_ts.iloc[:split]
test = df_ts.iloc[split:]

print("تعداد train:", len(train), "| test:", len(test))
print("بازه‌ی train:", train["date"].min().date(), "→", train["date"].max().date())
print("بازه‌ی test: ", test["date"].min().date(), "→", test["date"].max().date())
```

**مثال ۴ — نشت گروهی:**

```python
from sklearn.model_selection import GroupShuffleSplit

# چند تصویر از هر بیمار
patient_ids = np.repeat(np.arange(100), 5)
X_g = rng.normal(size=(500, 8))
y_g = rng.integers(0, 2, 500)

# اشتباه: تصادفی → تصاویر یک بیمار در دو طرف
from sklearn.model_selection import train_test_split
X_tr, X_te, y_tr, y_te = train_test_split(X_g, y_g, test_size=0.2, random_state=42)

# درست: گروهی
gss = GroupShuffleSplit(n_splits=1, test_size=0.2, random_state=42)
tr_idx, te_idx = next(gss.split(X_g, y_g, groups=patient_ids))
print("تعداد بیمار در train:", len(set(patient_ids[tr_idx])))
print("تعداد بیمار در test: ", len(set(patient_ids[te_idx])))
print("همپوشانی بیماران:", len(set(patient_ids[tr_idx]) & set(patient_ids[te_idx])))
# باید صفر باشد
```

**مثال ۵ — چک‌لیست نشت:**

```python
def leakage_checklist(df, target, time_col=None):
    """چک‌لیست سریع پیش از مدل‌سازی."""
    issues = []

    # ۱. ستون‌های با نام مشکوک
    suspicious = ["date", "id", "result", "status", "reason", "final", "after"]
    for col in df.columns:
        if col == target:
            continue
        if any(s in col.lower() for s in suspicious):
            issues.append(f"⚠️ ستون مشکوک: {col} — آیا در زمان تصمیم موجود است؟")

    # ۲. همبستگی بسیار بالا با هدف
    for col in df.select_dtypes("number").columns:
        if col == target:
            continue
        corr = abs(df[col].corr(df[target]))
        if corr > 0.95:
            issues.append(f"🚨 همبستگی {corr:.3f} با هدف: {col} — احتمال نشت بالا")

    # ۳. تکراری‌ها
    if df.duplicated().sum() > 0:
        issues.append(f"⚠️ {df.duplicated().sum()} ردیف تکراری — قبل از تقسیم حذف کن")

    # ۴. ترتیب زمانی
    if time_col and not df[time_col].is_monotonic_increasing:
        issues.append(f"ℹ️ ستون زمانی مرتب نیست — بررسی کن")

    return issues if issues else ["✅ مشکل واضحی پیدا نشد (بازبینی دستی همچنان لازم است)"]

# print("\n".join(leakage_checklist(df, "churn")))
```

### مقایسه و دیدگاه‌های مختلف

| نوع نشت | سختی تشخیص | اثر | راه‌حل |
|---|---|---|---|
| هدف | آسان (اگر بدانی) | فاجعه‌بار | حذف ویژگی |
| پیش‌پردازش | متوسط | کوچک تا متوسط | Pipeline |
| تقسیم | متوسط | متوسط | حذف تکراری قبل از تقسیم |
| زمانی | سخت | بزرگ | TimeSeriesSplit |
| گروهی | سخت | بزرگ | GroupKFold |

**یک دیدگاه مهم:** «اگر نتیجه خیلی خوب به‌نظر می‌رسد، احتمالاً نشت دارد.» این شهود، در ۹۰٪ موارد درست است.

### اشتباهات رایج

**۱. اعتماد به دقت ۹۹٪.** اول نشت را رد کن.
**۲. `fit` روی کل داده.** همیشه Pipeline.
**۳. تقسیم تصادفی سری زمانی.**
**۴. نادیده گرفتن گروه.** بیمار، کاربر، دستگاه.
**۵. حذف تکراری بعد از تقسیم.**
**۶. تنظیم مکرر روی test.**
**۷. ارزیابی روی داده‌ای که توزیعش با تولید متفاوت است.**

### نکات مهم و ترفندها

- **قاعده‌ی زمان تصمیم را برای هر ویژگی اجرا کن.** تنها دفاع مطمئن.
- **همیشه Pipeline.** نشت پیش‌پردازش را ساختاراً حذف می‌کند.
- **اگر دقت مشکوک است، ویژگی‌ها را یکی‌یکی حذف کن** و افت را ببین.
- **داده‌ی test را در یک «گاوصندوق» بگذار.** یک بار در پایان.
- **میان‌بر:** نشت = اطلاعات آینده در حال. همیشه بپرس «الان این را می‌دانم؟»

### خلاصه فصل

- نشت داده، شایع‌ترین علت شکست مدل در تولید است.
- هفت نوع اصلی: هدف، پیش‌پردازش، تقسیم، زمانی، گروهی، انتخاب ویژگی، کدگذاری.
- نشانه‌ی اصلی: عملکرد غیرواقعی خوب.
- Pipeline و قاعده‌ی «زمان تصمیم» دو دفاع اصلی‌اند.
- test را فقط یک بار استفاده کن.

**سؤالات مروری:**
1. سه نوع نشت داده را با مثال توضیح بده.
2. چرا `StandardScaler().fit(X)` قبل از تقسیم نشت است؟
3. اگر مدلی ۹۹.۵٪ دقت روی مسئله‌ای سخت گرفت، اولین فرضت چیست؟

---

## منابع و مراجع

**مستندات**
- scikit-learn — *Metrics and scoring*, *Cross-validation*, *Tuning hyperparameters*
- scikit-learn — *Common pitfalls and recommended practices* (بخش leakage — خواندنش واجب)

**کتاب‌ها**
- Géron, A. — *Hands-On ML*، فصل ۲ و ۳ (ارزیابی، داده‌ی نشت‌دهنده)
- Kaufman, S. et al. — *Leakage in Data Mining: Formulation, Detection, and Avoidance*

**مقالات**
- Saito, T. & Rehmsmeier, M. (2015) — *The Precision-Recall Plot Is More Informative than the ROC Plot...*
- Chicco, D. & Jurman, G. (2020) — *The advantages of the Matthews correlation coefficient (MCC)*
- Guo, C. et al. (2017) — *On Calibration of Modern Neural Networks*

---

**بعدی:** [`15-Overfitting-Regularization.md`](./15-Overfitting-Regularization.md) — فصل ۶۶: overfitting و underfitting
