# بخش ۱۱: یادگیری نیمه‌نظارتی و خودنظارتی

**مدت یادگیری تقریبی**: ۱.۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۹ و ۱۰ (یادگیری با/بدون ناظر)

---

## فهرست محتوا

- [فصل ۴۸: یادگیری نیمه‌نظارتی: وقتی برچسب کم است](#فصل-۴۸-یادگیری-نیمهنظارتی-وقتی-برچسب-کم-است)
- [فصل ۴۹: یادگیری خودنظارتی: مدل خودش برچسب می‌سازد](#فصل-۴۹-یادگیری-خودنظارتی-مدل-خودش-برچسب-میسازد)
- [فصل ۵۰: یادگیری انتقالی و مدل‌های از پیش آموزش‌دیده](#فصل-۵۰-یادگیری-انتقالی-و-مدلهای-از-پیش-آموزشدیده)

---

## فصل ۴۸: یادگیری نیمه‌نظارتی: وقتی برچسب کم است

### مقدمه

در دنیای واقعی، داده ارزان است و برچسب گران. یک بیمارستان می‌تواند ۱۰۰٬۰۰۰ تصویر رادیولوژی داشته باشد، اما برچسب‌گذاری هر تصویر نیم ساعت وقت متخصص می‌خواهد. نتیجه: ۵۰۰ تصویر برچسب‌خورده و ۹۹٬۵۰۰ بدون برچسب.

یادگیری نیمه‌نظارتی برای همین موقعیت ساخته شده: **از داده‌ی بی‌برچسب برای بهتر شدن استفاده کن.**

### مفاهیم اساسی

**۱. چرا داده‌ی بی‌برچسب کمک می‌کند؟**

داده‌ی بی‌برچسب، **ساختار** را نشان می‌دهد: کجا خوشه‌ها هستند، مرز طبیعی داده کجاست، کدام جهت‌ها مهم‌اند. این ساختار، حتی بدون برچسب هم وجود دارد.

**مثال شهودی:** اگر ۱۰ نقطه‌ی برچسب‌خورده در دو گروه داشته باشی، مرز را می‌توانی تقریباً بکشی. اما اگر ببینی ۱۰٬۰۰۰ نقطه در دو خوشه‌ی واضح قرار گرفته‌اند، مرز را دقیق‌تر می‌کشی — بدون این‌که برچسب‌های جدیدی داشته باشی.

**۲. سه فرض بنیادی که نیمه‌نظارتی بر آن‌ها تکیه می‌کند:**

| فرض | معنا | نقض |
|---|---|---|
| **پیوستگی (Continuity)** | نقاط نزدیک، برچسب مشابه دارند | مرز پیچیده در ناحیه‌ی متراکم |
| **خوشه (Cluster)** | یک خوشه، معمولاً یک کلاس است | خوشه‌های مختلط |
| **منیفولد (Manifold)** | داده روی سطحی با ابعاد کمتر است | داده‌ی پرنویز |

**اگر این فرض‌ها نقض شوند، نیمه‌نظارتی می‌تواند بدتر از با ناظر باشد.** این یک هشدار جدی است، نه جزئیات.

**۳. چهار خانواده‌ی روش:**

1. **Self-Training:** مدل را با داده‌ی برچسب‌دار آموزش بده، برای داده‌ی بی‌برچسب پیش‌بینی کن، مطمئن‌ترین‌ها را اضافه کن، تکرار کن.
2. **Label Propagation / Spreading:** برچسب‌ها روی گراف شباهت پخش می‌شوند.
3. **Generative Models:** مدل توزیع داده را یاد می‌گیرد؛ برچسب، بخشی از متغیرهای پنهان است.
4. **Consistency Regularization:** مدل باید برای یک نمونه و نسخه‌ی تغییریافته‌اش، همان خروجی را بدهد (پایه‌ی روش‌های مدرن مثل FixMatch).

**۴. Self-Training — ساده‌ترین و عملی‌ترین.**

```
۱. با داده‌ی برچسب‌دار آموزش بده
۲. برای داده‌ی بی‌برچسب احتمال پیش‌بینی کن
۳. نمونه‌های با احتمال بالا (مثلاً > ۰.۹۵) را با برچسب پیش‌بینی‌شده اضافه کن
۴. دوباره آموزش بده
۵. تکرار کن
```

**خطر:** تأیید خطا (confirmation bias). اگر مدل اشتباه پیش‌بینی کند و با اطمینان بالا، آن خطا وارد داده‌ی آموزش می‌شود و تقویت می‌شود.

**۵. چه زمانی نیمه‌نظارتی جواب می‌دهد؟**
- داده‌ی بی‌برچسب **بسیار** بیشتر از برچسب‌دار است (۱۰ تا ۱۰۰ برابر).
- داده‌ی بی‌برچسب از **همان توزیع** است.
- فرض‌های خوشه/منیفولد برقرارند.
- مدل پایه معقول است (نه تصادفی).

**۶. چه زمانی جواب نمی‌دهد؟**
- داده‌ی بی‌برچسب از توزیع متفاوت (out-of-distribution).
- کلاس‌ها در هم تنیده‌اند.
- داده‌ی برچسب‌دار خیلی کم است (زیر ۵۰ نمونه برای هر کلاس).

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — Self-Training دستی:**

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score
from sklearn.preprocessing import StandardScaler

rng = np.random.default_rng(42)
X, y = make_classification(n_samples=3000, n_features=10, n_informative=6,
                           n_classes=2, random_state=42)

# فقط ۵٪ برچسب داریم
n_labeled = 150
idx = rng.permutation(len(X))
labeled_idx, unlabeled_idx = idx[:n_labeled], idx[n_labeled:]

X_lab, y_lab = X[labeled_idx], y[labeled_idx]
X_unlab = X[unlabeled_idx]
y_unlab_true = y[unlabeled_idx]

scaler = StandardScaler().fit(X)
X_lab_s, X_unlab_s = scaler.transform(X_lab), scaler.transform(X_unlab)

# ---------- baseline: فقط داده‌ی برچسب‌دار ----------
base = LogisticRegression(max_iter=1000, random_state=42).fit(X_lab_s, y_lab)
print(f"فقط برچسب‌دار: {accuracy_score(y_unlab_true, base.predict(X_unlab_s)):.3f}")

# ---------- Self-Training ----------
model = base
X_cur, y_cur = X_lab_s.copy(), y_lab.copy()
remaining = np.ones(len(X_unlab_s), dtype=bool)

for iteration in range(5):
    probs = model.predict_proba(X_unlab_s[remaining])[:, 1]
    conf = np.abs(probs - 0.5) * 2          # اطمینان: ۰ = مردد، ۱ = مطمئن
    confident_local = conf > 0.95

    if confident_local.sum() == 0:
        print("هیچ نمونه‌ی مطمئنی نماند.")
        break

    # ایندکس‌های مطلق
    remaining_idx = np.where(remaining)[0]
    chosen = remaining_idx[confident_local]

    X_cur = np.vstack([X_cur, X_unlab_s[chosen]])
    y_cur = np.concatenate([y_cur, (probs[confident_local] > 0.5).astype(int)])
    remaining[chosen] = False

    model = LogisticRegression(max_iter=1000, random_state=42).fit(X_cur, y_cur)
    acc = accuracy_score(y_unlab_true[~remaining], model.predict(X_unlab_s[~remaining]))
    print(f"دور {iteration+1}: {chosen.size:>4} نمونه اضافه شد | "
          f"دقت روی همان‌ها: {acc:.3f} | آموزش: {len(X_cur)}")
```

**مثال ۲ — Label Propagation:**

```python
from sklearn.semi_supervised import LabelSpreading, LabelPropagation
from sklearn.datasets import load_digits
from sklearn.model_selection import train_test_split

digits = load_digits()
X_d, y_d = digits.data, digits.target
X_tr, X_te, y_tr, y_te = train_test_split(X_d, y_d, test_size=0.3, stratify=y_d, random_state=42)

# فقط ۲۰ نمونه از هر کلاس برچسب‌دار
rng = np.random.default_rng(0)
labeled_mask = np.zeros(len(X_tr), dtype=bool)
for cls in np.unique(y_tr):
    cls_idx = np.where(y_tr == cls)[0]
    labeled_mask[rng.choice(cls_idx, 20, replace=False)] = True

y_semi = np.where(labeled_mask, y_tr, -1)     # -۱ = بدون برچسب

for model in [LabelPropagation(kernel="knn", n_neighbors=7),
              LabelSpreading(kernel="knn", n_neighbors=7)]:
    model.fit(X_tr, y_semi)
    print(f"{model.__class__.__name__:18s}: {model.score(X_te, y_te):.3f}")

# مقایسه با فقط داده‌ی برچسب‌دار
sup = LogisticRegression(max_iter=3000).fit(X_tr[labeled_mask], y_tr[labeled_mask])
print(f"{'فقط برچسب‌دار':18s}: {sup.score(X_te, y_te):.3f}")
```

**نتیجه‌ی معمول:** نیمه‌نظارتی چند درصد بهتر است — با ۱۰ برابر داده‌ی کمتر برچسب‌خورده.

**مثال ۳ — پخش برچسب روی گراف (مفهوم):**

```python
import numpy as np
from sklearn.metrics.pairwise import rbf_kernel

# گراف شباهت
K = rbf_kernel(X_lab_s[:50], gamma=0.1)
# برچسب‌ها با وزن شباهت پخش می‌شوند:
# y_new = D⁻¹ K y   (تکرار تا همگرایی)
```

### مقایسه و دیدگاه‌های مختلف

| روش | مزیت | عیب | کِی |
|---|---|---|---|
| Self-Training | ساده، مدل‌آگنوستیک | تأیید خطا | مدل قوی داری |
| Label Spreading | نیازی به مدل ندارد | حافظه‌ی گراف | داده‌ی متوسط |
| Generative | پایه‌ی نظری قوی | پیچیده، کند | داده‌ی پیچیده |
| Consistency (FixMatch) | بهترین نتایج مدرن | نیاز به augmentation | تصویر، متن |

**یک دیدگاه مهم:** «نیمه‌نظارتی همیشه کمک نمی‌کند.» در برخی مقالات نشان داده شده که با نقض فرض‌ها، عملکرد **بدتر** می‌شود. **همیشه با baseline مقایسه کن** و اگر بهتر نشد، رها کن.

### اشتباهات رایج

**۱. فرض این‌که داده‌ی بی‌برچسب همیشه کمک می‌کند.** گاهی ضرر می‌زند.
**۲. آستانه‌ی اطمینان پایین در Self-Training.** خطاها انباشته می‌شوند.
**۳. داده‌ی بی‌برچسب از توزیع متفاوت.** مثلاً عکس‌های روز و شب. نتیجه بدتر می‌شود.
**۴. ارزیابی روی داده‌ی برچسب‌دار اولیه.** باید روی داده‌ی جداگانه‌ی برچسب‌دار (که در آموزش استفاده نشده) ارزیابی کنی.
**۵. نادیده گرفتن هزینه.** Self-Training چند بار آموزش می‌دهد؛ گاهی ارزان‌تر است که داده‌ی بیشتری برچسب بزنی.

### نکات مهم و ترفندها

- **همیشه baseline «فقط برچسب‌دار» را حساب کن.** بدون آن، نمی‌دانی نیمه‌نظارتی کمک کرد یا نه.
- **آستانه‌ی اطمینان را سخت بگیر** (۰.۹۵ یا بالاتر) در دورهای اول.
- **در هر دور، کم اضافه کن.** اضافه کردن یک‌باره‌ی همه، خطا را قفل می‌کند.
- **اگر داده‌ی بی‌برچسب خیلی زیاد است، زیرمجموعه بگیر.** لازم نیست همه را استفاده کنی.
- **میان‌بر:** کمی برچسب + داده‌ی زیاد بی‌برچسب = ساختار + نظارت.

### خلاصه فصل

- نیمه‌نظارتی وقتی است که برچسب کم و داده‌ی بی‌برچسب زیاد است.
- بر سه فرض تکیه می‌کند: پیوستگی، خوشه، منیفولد.
- Self-Training ساده‌ترین روش عملی است، با خطر تأیید خطا.
- همیشه با baseline مقایسه کن؛ نیمه‌نظارتی تضمینی نیست.
- داده‌ی بی‌برچسب باید از همان توزیع باشد.

**سؤالات مروری:**
1. سه فرض نیمه‌نظارتی را نام ببر.
2. خطر اصلی Self-Training چیست؟
3. چه زمانی نیمه‌نظارتی می‌تواند بدتر از با ناظر باشد؟

---

## فصل ۴۹: یادگیری خودنظارتی: مدل خودش برچسب می‌سازد

### مقدمه

این فصل، **مهم‌ترین ایده‌ی دهه‌ی اخیر در AI** را معرفی می‌کند. ایده ساده است:

> اگر برچسب نداریم، **خودِ داده را تبدیل به مسئله‌ی برچسب‌دار کنیم.**

مثال: یک جمله بردار، آخرین کلمه‌اش را پاک کن، مدل را آموزش بده آن را پیش‌بینی کند. **برچسب، خودِ کلمه‌ی حذف‌شده است.**

همین ایده، پشت GPT، BERT و تقریباً همه‌ی مدل‌های بزرگ امروزی است.

### مفاهیم اساسی

**۱. تفاوت با نیمه‌نظارتی:**
- نیمه‌نظارتی: کمی برچسب واقعی داری + داده‌ی بی‌برچسب.
- خودنظارتی: **هیچ برچسبی نداری**؛ برچسب را از ساختار داده می‌سازی.

**۲. چهار روش خودنظارتی کلاسیک:**

| روش | برچسب ساختگی | حوزه |
|---|---|---|
| **Masked Modeling** | کلمه/تکه‌ی پنهان‌شده | متن (BERT) |
| **Next-token Prediction** | توکن بعدی | متن (GPT) |
| **Contrastive Learning** | «کدام دو نمونه به هم شبیه‌اند؟» | تصویر (SimCLR)، متن (CLIP) |
| **Pretext Tasks** | چرخاندن تصویر، ترتیب پازل | تصویر (قدیمی‌تر) |

**۳. Masked Language Modeling (MLM) — منطق.**

```
ورودی:  "هوش مصنوعی [MASK] را تغییر می‌دهد"
هدف:    پیش‌بینی "جهان"
```
مدل باید از context دو طرف استفاده کند. این کار، **فهم معنایی** می‌سازد، نه فقط تولید.

**۴. Next-Token Prediction (Causal LM) — منطق.**

```
ورودی: "هوش مصنوعی جهان را"
هدف:    "تغییر"
```
مدل فقط از گذشته استفاده می‌کند. این، پایه‌ی مدل‌های تولیدی است (فایل ۱۸).

**۵. Contrastive Learning — منطق.**

```
دو نسخه از یک تصویر (کراپ، چرخش، تغییر رنگ) = جفت مثبت
تصویر دیگر = جفت منفی
هدف: بردارهای جفت مثبت نزدیک، منفی‌ها دور
```

**چرا کار می‌کند؟** مدل مجبور می‌شود ویژگی‌های **پایدار** (که با تغییر ظاهر عوض نمی‌شوند) را یاد بگیرد — مثل شکل جسم، نه رنگ پیکسل.

**۶. چرا خودنظارتی انقلابی بود؟**

- **مقیاس‌پذیر است:** نیازی به انسان ندارد؛ می‌توانی از کل اینترنت استفاده کنی.
- **نمایندگی عمومی می‌سازد:** مدل آموزش‌دیده با خودنظارتی، برای بسیاری از کارهای پایین‌دستی مفید است.
- **کاهش نیاز به برچسب:** بعد از پیش‌آموزش، با چند هزار نمونه‌ی برچسب‌دار، fine-tune می‌کنی.

**۷. هزینه:** خودنظارتی **گران** است. آموزش GPT-3، میلیون‌ها دلار GPU می‌خواست. برای همین، اکثر ما از مدل‌های از پیش آموزش‌دیده استفاده می‌کنیم، نه از صفر.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — MLM ساده از صفر (مفهومی):**

```python
import numpy as np

# واژگان کوچک
vocab = ["<mask>", "هوش", "مصنوعی", "جهان", "را", "تغییر", "می‌دهد", "است"]
word2idx = {w: i for i, w in enumerate(vocab)}

sentence = ["هوش", "مصنوعی", "جهان", "را", "تغییر", "می‌دهد"]

# ساخت نمونه‌ی masked
def make_mlm_sample(sentence, mask_idx=2):
    tokens = sentence.copy()
    target = tokens[mask_idx]
    tokens[mask_idx] = "<mask>"
    return tokens, target

inp, target = make_mlm_sample(sentence, mask_idx=2)
print("ورودی:", inp)
print("هدف:  ", target)
print("یعنی: مدل باید «جهان» را از context حدس بزند")
```

**این، کل ایده‌ی BERT در سه خط است.** پیچیدگی واقعی در مقیاس و معماری است، نه در ایده.

**مثال ۲ — Next-token prediction از صفر:**

```python
def make_causal_samples(tokens):
    """هر پیشوند → توکن بعدی. یک جمله، چند نمونه می‌سازد."""
    samples = []
    for i in range(1, len(tokens)):
        samples.append((tokens[:i], tokens[i]))
    return samples

for context, target in make_causal_samples(["هوش", "مصنوعی", "جهان", "را", "تغییر", "می‌دهد"]):
    print(f"{' '.join(context):40s} → {target}")
```

**درس مهم:** یک جمله‌ی ۶ کلمه‌ای، **۵ نمونه‌ی آموزشی** می‌سازد. این، راز مقیاس‌پذیری خودنظارتی است: داده‌ی محدود، نمونه‌ی نامحدود.

**مثال ۳ — شبیه‌سازی contrastive با بردارهای ساده:**

```python
import numpy as np

def contrastive_loss(anchor, positive, negatives, temperature=0.1):
    """NT-Xent ساده‌شده — تابع خطای SimCLR."""
    def cos(a, b):
        return a @ b / (np.linalg.norm(a) * np.linalg.norm(b) + 1e-12)

    pos_sim = cos(anchor, positive) / temperature
    neg_sims = np.array([cos(anchor, n) / temperature for n in negatives])

    logits = np.concatenate([[pos_sim], neg_sims])
    exp = np.exp(logits - logits.max())
    probs = exp / exp.sum()

    return -np.log(probs[0])     # -log(احتمال جفت مثبت)

rng = np.random.default_rng(0)
anchor = rng.normal(size=8)
positive = anchor + rng.normal(scale=0.1, size=8)      # شبیه
negatives = [rng.normal(size=8) for _ in range(5)]     # نامرتبط

print("loss:", round(contrastive_loss(anchor, positive, negatives), 4))
```

**مثال ۴ — استفاده از مدل از پیش آموزش‌دیده (روش عملی):**

```python
# با sentence-transformers: pip install sentence-transformers
try:
    from sentence_transformers import SentenceTransformer, util

    model = SentenceTransformer("paraphrase-multilingual-MiniLM-L12-v2")
    sentences = [
        "هوش مصنوعی چیست؟",
        "یادگیری ماشین چه کاربردی دارد؟",
        "قیمت دلار امروز چقدر است؟",
    ]
    emb = model.encode(sentences)
    sim = util.cos_sim(emb, emb)
    print(sim.round(3))
except ImportError:
    print("نصب کن: pip install sentence-transformers")
```

**نکته:** دو جمله‌ی اول درباره‌ی AI هستند و شباهت بالاتری دارند، هرچند کلمه‌ی مشترک کمی دارند. **این، قدرت embedding خودنظارتی است.**

### مقایسه و دیدگاه‌های مختلف

| رویکرد | داده‌ی لازم | هزینه | کیفیت |
|---|---|---|---|
| با ناظر از صفر | برچسب‌دار زیاد | متوسط | خوب با داده‌ی کافی |
| نیمه‌نظارتی | کمی برچسب + زیاد بی‌برچسب | متوسط | بهتر |
| خودنظارتی + fine-tune | بی‌برچسب عظیم + کمی برچسب | **بالا** | بهترین |
| خودنظارتی صرف (zero-shot) | بی‌برچسب عظیم | بالا | متغیر |

**دو دیدگاه:**
- **«خودنظارتی، مسیر رسیدن به AGI است.»** چون مقیاس‌پذیر است و نمایندگی عمومی می‌سازد.
- **«خودنظارتی فقط فشرده‌سازی است، نه فهم.»** چون هدف، بازسازی/پیش‌بینی سطحی است، نه مدل‌سازی علّی.

**هر دو دیدگاه قابل دفاع‌اند. مهم این است که بدانی مدل‌های امروز با این روش ساخته شده‌اند.**

### اشتباهات رایج

**۱. «خودنظارتی یعنی بدون ناظر.»** نه؛ برچسب دارد، اما **ساختگی** است.
**۲. تلاش برای آموزش خودنظارتی از صفر روی لپ‌تاپ.** هزینه‌ی نجومی. از مدل آماده استفاده کن.
**۳. فرض این‌که embedding عمومی برای همه‌ی دامنه‌ها خوب است.** برای دامنه‌ی تخصصی (پزشکی، حقوقی)، fine-tune لازم است.
**۴. اشتباه گرفتن MLM و causal LM.** اولی دوطرفه، دومی یک‌طرفه؛ کاربردهای متفاوت.
**۵. نادیده گرفتن هزینه‌ی استنتاج.** مدل بزرگ، کند و گران است.

### نکات مهم و ترفندها

- **برای متن، از embedding آماده استفاده کن** (`sentence-transformers`). نیازی به آموزش از صفر نیست.
- **برای تصویر، ResNet/EfficientNet از پیش آموزش‌دیده** (ImageNet) نقطه‌ی شروع عالی است.
- **fine-tune با داده‌ی کم، بهتر از آموزش از صفر با داده‌ی کم است.** این قاعده تقریباً همیشه درست است.
- **`util.cos_sim` ابزار سریع سنجش شباهت متنی است.** برای جست‌وجوی معنایی، خوشه‌بندی متن، حذف تکراری.
- **میان‌بر:** خودنظارتی = برچسب از خود داده · MLM = پنهان کن و حدس بزن · Contrastive = شبیه را نزدیک کن.

### خلاصه فصل

- خودنظارتی، برچسب را از ساختار داده می‌سازد؛ نیازی به انسان ندارد.
- MLM و next-token prediction، دو روش اصلی در متن‌اند.
- Contrastive learning، پایه‌ی مدل‌های چندوجهی مثل CLIP است.
- مقیاس‌پذیری، دلیل اصلی انقلاب مدل‌های بنیادی است.
- در عمل، از مدل‌های از پیش آموزش‌دیده استفاده کن، نه آموزش از صفر.

**سؤالات مروری:**
1. تفاوت خودنظارتی و نیمه‌نظارتی چیست؟
2. MLM چه برچسبی می‌سازد؟
3. چرا contrastive learning ویژگی‌های پایدار را یاد می‌گیرد؟

---

## فصل ۵۰: یادگیری انتقالی و مدل‌های از پیش آموزش‌دیده

### مقدمه

تصور کن بخواهی مدلی بسازی که عیب سطح محصول را در خط تولید تشخیص بدهد. داده: ۲۰۰۰ عکس. از صفر آموزش بدهی، شکست می‌خوری. اما اگر مدلی را که روی ۱.۲ میلیون تصویر آموزش دیده **fine-tune** کنی، با همان ۲۰۰۰ عکس نتیجه‌ی خوبی می‌گیری.

این، **یادگیری انتقالی** است: انتقال دانش از یک مسئله به مسئله‌ی دیگر. و این، **مهم‌ترین مهارت عملی** برای کسی است که نمی‌خواهد از صفر آموزش بدهد.

### مفاهیم اساسی

**۱. ایده‌ی اصلی.**

شبکه‌های عمیق، سلسله‌مراتبی یاد می‌گیرند:
- لایه‌های اول: لبه، رنگ، بافت
- لایه‌های میانی: شکل، قسمت
- لایه‌های آخر: مفاهیم خاص مسئله (گربه، سگ)

**لایه‌های اول تقریباً برای همه‌ی مسائل تصویری مفیدند.** پس می‌توانی آن‌ها را نگه داری و فقط لایه‌های آخر را برای مسئله‌ی خودت آموزش بدهی.

**۲. سه استراتژی:**

| استراتژی | چه می‌کنی | کِی |
|---|---|---|
| **Feature Extraction** | مدل پایه را فریز کن، فقط یک classifier جدید روی خروجی آموزش بده | داده‌ی خیلی کم (<۱۰۰۰) |
| **Fine-tuning جزئی** | چند لایه‌ی آخر را باز کن | داده‌ی متوسط (۱۰۰۰–۵۰٬۰۰۰) |
| **Fine-tuning کامل** | همه‌ی وزن‌ها را با lr کوچک آموزش بده | داده‌ی زیاد (>۵۰٬۰۰۰) |

**۳. نکته‌ی حیاتی: learning rate تفاضلی.**

در fine-tune، لایه‌های اولیه باید lr **خیلی کوچک** بگیرند (تغییر نکنند) و لایه‌های آخر lr بزرگ‌تر. وگرنه دانش قبلی از بین می‌رود — این را **catastrophic forgetting** می‌گویند.

**۴. `freeze` کردن در PyTorch:**

```python
for param in model.parameters():
    param.requires_grad = False     # فریز
```

**۵. مدل‌های پایه‌ی رایج:**

| حوزه | مدل‌ها |
|---|---|
| تصویر | ResNet، EfficientNet، ViT، ConvNeXt |
| متن | BERT، RoBERTa، DistilBERT، sentence-transformers |
| چندوجهی | CLIP، BLIP |
| صدا | Whisper، wav2vec |
| جدولی | TabNet، و در عمل: boosting از صفر بهتر است |

**۶. چرا برای داده‌ی جدولی، انتقال کمتر مفید است؟**
چون ویژگی‌های جدولی (سن، درآمد) معنای جهانی ندارند و ستون‌های هر دیتاست متفاوت‌اند. **برای جدول، از صفر با boosting بساز.**

**۷. هزینه‌ی fine-tune:** بسیار کمتر از پیش‌آموزش، اما بیشتر از feature extraction. روی CPU ممکن است ساعت‌ها طول بکشد؛ روی GPU رایگان Colab، چند دقیقه.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — Feature Extraction با ResNet (تصویر):**

```python
# pip install torch torchvision
import torch
import torch.nn as nn
from torchvision import models, transforms, datasets
from torch.utils.data import DataLoader

device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

# مدل از پیش آموزش‌دیده
weights = models.ResNet18_Weights.DEFAULT
backbone = models.resnet18(weights=weights)

# فریز همه‌ی لایه‌ها
for p in backbone.parameters():
    p.requires_grad = False

# جایگزینی لایه‌ی آخر
num_features = backbone.fc.in_features
backbone.fc = nn.Linear(num_features, 10)      # ۱۰ کلاس جدید
backbone = backbone.to(device)

# فقط پارامترهای لایه‌ی آخر آموزش می‌بینند
trainable = [p for p in backbone.parameters() if p.requires_grad]
print(f"پارامترهای آموزش‌پذیر: {sum(p.numel() for p in trainable):,}")
print(f"کل پارامترها: {sum(p.numel() for p in backbone.parameters()):,}")
```

**نکته:** فقط ~۰.۵٪ پارامترها آموزش می‌بینند، اما نتیجه عالی است.

**مثال ۲ — Fine-tuning با lr تفاضلی:**

```python
# باز کردن لایه‌های آخر
for name, p in backbone.named_parameters():
    if "layer4" in name or "fc" in name:
        p.requires_grad = True

optimizer = torch.optim.AdamW([
    {"params": backbone.layer4.parameters(), "lr": 1e-4},   # لایه‌ی آخر: سریع‌تر
    {"params": backbone.fc.parameters(),     "lr": 1e-3},   # classifier: سریع‌ترین
], weight_decay=1e-4)
```

**مثال ۳ — pipeline کامل آموزش (قابل اجرا):**

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader
from torchvision import datasets, transforms, models

transform = transforms.Compose([
    transforms.Resize(224),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                         std=[0.229, 0.224, 0.225]),
])

def train_one_epoch(model, loader, criterion, optimizer, device):
    model.train()
    total_loss, correct, n = 0.0, 0, 0
    for images, labels in loader:
        images, labels = images.to(device), labels.to(device)
        optimizer.zero_grad()
        outputs = model(images)
        loss = criterion(outputs, labels)
        loss.backward()
        optimizer.step()

        total_loss += loss.item() * labels.size(0)
        correct += (outputs.argmax(1) == labels).sum().item()
        n += labels.size(0)
    return total_loss / n, correct / n

# استفاده:
# train_ds = datasets.ImageFolder("data/train", transform=transform)
# loader = DataLoader(train_ds, batch_size=32, shuffle=True)
# model = models.resnet18(weights=models.ResNet18_Weights.DEFAULT)
# model.fc = nn.Linear(model.fc.in_features, len(train_ds.classes))
# ...
```

**مثال ۴ — Transfer Learning برای متن:**

```python
# pip install transformers
try:
    from transformers import pipeline

    classifier = pipeline(
        "sentiment-analysis",
        model="distilbert-base-uncased-finetuned-sst-2-english",
    )
    results = classifier([
        "This movie was fantastic!",
        "Worst purchase I have ever made.",
    ])
    for r in results:
        print(r)
except ImportError:
    print("نصب کن: pip install transformers")
```

**مثال ۵ — استخراج embedding و استفاده در مدل ساده:**

```python
try:
    from sentence_transformers import SentenceTransformer
    from sklearn.linear_model import LogisticRegression
    from sklearn.model_selection import cross_val_score

    encoder = SentenceTransformer("paraphrase-multilingual-MiniLM-L12-v2")

    texts = ["دوستش داشتم", "خیلی بد بود", "عالی بود", "افتضاح بود",
             "لذت بردم", "وقت تلف کردم", "شگفت‌انگیز", "ناامید شدم"]
    labels = [1, 0, 1, 0, 1, 0, 1, 0]

    X_emb = encoder.encode(texts)
    clf = LogisticRegression(max_iter=1000)
    scores = cross_val_score(clf, X_emb, labels, cv=2)
    print("دقت با embedding + لجستیک:", round(scores.mean(), 3))
except ImportError:
    print("نصب کن: pip install sentence-transformers")
```

**این الگو («embedding + مدل ساده») در صنعت بسیار رایج است** و اغلب از fine-tune کامل سریع‌تر و کافی‌تر است.

### مقایسه و دیدگاه‌های مختلف

| رویکرد | داده‌ی لازم | زمان آموزش | کیفیت | هزینه |
|---|---|---|---|---|
| از صفر | >۱۰۰٬۰۰۰ | ساعت‌ها | متغیر | بالا |
| Fine-tune کامل | >۵۰٬۰۰۰ | دقیقه–ساعت | بالا | متوسط |
| Fine-tune جزئی | ۱۰۰۰–۵۰٬۰۰۰ | دقیقه | خوب | کم |
| Feature extraction | <۱۰۰۰ | ثانیه | قابل قبول | خیلی کم |
| Embedding + مدل ساده | <۱۰۰۰ | ثانیه | خوب برای متن | خیلی کم |

**یک دیدگاه:** «یادگیری انتقالی، AI را دموکراتیزه کرد.» درست است: امروز یک نفر با لپ‌تاپ می‌تواند مدلی بسازد که ده سال پیش نیاز به تیم تحقیقاتی داشت.

### اشتباهات رایج

**۱. آموزش از صفر با داده‌ی کم.** تقریباً همیشه شکست.
**۲. فراموش کردن `requires_grad = False`.** همه‌چیز آموزش می‌بیند، دانش قبلی از بین می‌رود.
**۳. lr یکسان برای همه‌ی لایه‌ها.** لایه‌های اولیه خراب می‌شوند.
**۴. فراموش کردن `model.train()` / `model.eval()`.** BatchNorm و Dropout رفتار اشتباه می‌دهند.
**۵. استفاده از نرمال‌سازی متفاوت با پیش‌آموزش.** مدل با `mean=[0.485,...]` آموزش دیده؛ باید همان را بدهی.
**۶. fine-tune روی داده‌ی خیلی کم بدون augmentation.** بیش‌برازش سریع.
**۷. فرض این‌که transfer برای جدول هم عالی است.** نیست.

### نکات مهم و ترفندها

- **قاعده‌ی سرانگشتی:** داده‌ی کم → feature extraction؛ داده‌ی متوسط → fine-tune جزئی؛ داده‌ی زیاد → fine-tune کامل.
- **همیشه از `weights=DEFAULT` استفاده کن، نه `pretrained=True`** (API جدید torchvision).
- **`model.eval()` را در ارزیابی فراموش نکن.**
- **lr را برای لایه‌های اولیه ۱۰ برابر کوچک‌تر بگذار.**
- **برای متن، `sentence-transformers` نقطه‌ی شروع بی‌دردسر است.**
- **میان‌بر:** انتقال = دانش قبلی + کمی داده‌ی جدید · فریز = فقط آخر را آموزش بده.

### خلاصه فصل

- یادگیری انتقالی، دانش مدل آموزش‌دیده را به مسئله‌ی جدید منتقل می‌کند.
- سه استراتژی: feature extraction، fine-tune جزئی، fine-tune کامل.
- انتخاب استراتژی، تابع حجم داده است.
- lr تفاضلی از catastrophic forgetting جلوگیری می‌کند.
- برای داده‌ی جدولی، انتقال کمتر مفید است؛ از boosting استفاده کن.

**سؤالات مروری:**
1. سه استراتژی یادگیری انتقالی را با شرط استفاده‌ی هر کدام بگو.
2. catastrophic forgetting چیست و چطور جلوگیری می‌کنی؟
3. چرا برای داده‌ی جدولی، انتقال از تصویر مفید نیست؟

---

## منابع و مراجع

**مستندات**
- scikit-learn — *Semi-supervised learning* (`LabelPropagation`, `LabelSpreading`, `SelfTrainingClassifier`)
- PyTorch — *Transfer Learning tutorial* (Finetuning vs Feature Extraction)
- Hugging Face — *Transformers course*, *Sentence Transformers docs*

**مقالات کلیدی**
- Zhu, X. & Goldberg, A. (2009) — *Introduction to Semi-Supervised Learning*
- Devlin, J. et al. (2018) — *BERT: Pre-training of Deep Bidirectional Transformers*
- Radford, A. et al. (2021) — *Learning Transferable Visual Models From Natural Language Supervision* (CLIP)
- Chen, T. et al. (2020) — *A Simple Framework for Contrastive Learning* (SimCLR)
- He, K. et al. (2019) — *Momentum Contrast* (MoCo)

**کتاب‌ها**
- Prince, S. — *Understanding Deep Learning*، فصل ۱۱ (Self-supervised)

---

**بعدی:** [`12-Reinforcement-Learning.md`](./12-Reinforcement-Learning.md) — فصل ۵۱: عامل، محیط، پاداش
