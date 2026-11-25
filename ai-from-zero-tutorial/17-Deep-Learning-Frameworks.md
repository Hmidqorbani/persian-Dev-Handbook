# بخش ۱۷: چارچوب‌های یادگیری عمیق

**مدت یادگیری تقریبی**: ۲.۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۱۶ (شبکه عصبی از صفر)

---

## فهرست محتوا

- [فصل ۷۷: یادگیری عمیق چیست و چه زمانی سراغش برویم](#فصل-۷۷-یادگیری-عمیق-چیست-و-چه-زمانی-سراغش-برویم)
- [فصل ۷۸: Keras / TensorFlow: اولین شبکه در ۲۰ خط](#فصل-۷۸-keras--tensorflow-اولین-شبکه-در-۲۰-خط)
- [فصل ۷۹: PyTorch: تنسور، autograd، حلقه‌ی آموزش](#فصل-۷۹-pytorch-تنسور-autograd-حلقهی-آموزش)
- [فصل ۸۰: CNN: بینایی ماشین به زبان ساده](#فصل-۸۰-cnn-بینایی-ماشین-به-زبان-ساده)
- [فصل ۸۱: RNN و داده‌های ترتیبی](#فصل-۸۱-rnn-و-دادههای-ترتیبی)

---

## فصل ۷۷: یادگیری عمیق چیست و چه زمانی سراغش برویم

### مقدمه

«یادگیری عمیق» یعنی شبکه‌ی عصبی با **لایه‌های زیاد**. اما «زیاد» چقدر است؟ و مهم‌تر: **چه زمانی ارزشش را دارد؟**

این فصل، خط مرزی بین «شبکه‌ی کوچک» و «یادگیری عمیق» و شرایط استفاده از هر کدام را می‌دهد.

### مفاهیم اساسی

**۱. تعریف کاری.**

| نوع | لایه‌ها | پارامترها | مثال |
|---|---|---|---|
| شبکه‌ی کم‌عمق | ۱–۲ | ۱۰³ | MLP روی داده‌ی جدولی |
| شبکه‌ی متوسط | ۳–۱۰ | ۱۰⁵ | MNIST classifier |
| یادگیری عمیق | ۱۰–۱۰۰+ | ۱۰⁶–۱۰¹² | ResNet، GPT |

**۲. سه دلیل که عمق کار می‌کند:**

1. **سلسله‌مراتب ویژگی:** لبه → بافت → قسمت → شیء.
2. **ترکیب‌پذیری:** هر لایه از ترکیب لایه‌ی قبل استفاده می‌کند → بازدهی نمایی.
3. **نمایندگی توزیع‌شده:** هر مفهوم در چند نورون پخش می‌شود → مقاومت به نویز.

**۳. چه زمانی یادگیری عمیق؟**

| شرط | توضیح |
|---|---|
| داده‌ی خام و پیچیده | تصویر، صدا، متن |
| داده‌ی زیاد | حداقل ۱۰k نمونه برای شروع |
| ویژگی‌های دستی سخت | «چه ویژگی‌ای گربه را توصیف می‌کند؟» |
| GPU موجود | یا صبر کافی |

**۴. چه زمانی **نه**؟**

| شرط | جایگزین |
|---|---|
| داده‌ی جدولی | Gradient Boosting (XGBoost/LightGBM) |
| داده‌ی کم (<۱۰۰۰) | مدل کلاسیک یا transfer learning |
| تفسیرپذیری حیاتی | رگرسیون لجستیک، درخت |
| CPU-only و زمان محدود | مدل کلاسیک |

**قاعده‌ی طلایی صنعت:** «برای داده‌ی جدولی، اول boosting. اگر جواب نداد، شبکه.»

**۵. سه چارچوب اصلی:**

| چارچوب | مزیت | عیب | محبوبیت |
|---|---|---|---|
| **PyTorch** | انعطاف، دیباگ آسان | کد بیشتر | تحقیق + صنعت |
| **TensorFlow/Keras** | ساده، اکوسیستم تولید | انعطاف کمتر | صنعت |
| **JAX** | سریع، تابعی | اکوسیستم کوچک‌تر | تحقیق |

**۶. چرا PyTorch غالب شد؟**
- **Pythonic:** حلقه‌ی آموزش صریح است؛ می‌توانی breakpoint بگذاری.
- **Dynamic graph:** گراف در هر اجرا ساخته می‌شود → دیباگ آسان.
- **جامعه:** اکثر مقالات جدید با PyTorch.

**۷. سخت‌افزار:**
- **CPU:** کافی برای مدل‌های کوچک.
- **GPU:** لازم برای CNN/Transformer. Google Colab رایگان می‌دهد.
- **TPU/GPU ابری:** برای مدل‌های بزرگ.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — مقایسه‌ی روی داده‌ی جدولی:**

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.model_selection import cross_val_score
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.neural_network import MLPClassifier
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import make_pipeline

X, y = make_classification(n_samples=3000, n_features=30, n_informative=10,
                           n_redundant=10, flip_y=0.05, random_state=42)

models = {
    "GradientBoosting": GradientBoostingClassifier(n_estimators=300, random_state=42),
    "MLP (2×128)": make_pipeline(StandardScaler(),
                                  MLPClassifier(hidden_layer_sizes=(128, 128),
                                                max_iter=500, random_state=42)),
    "MLP (3×256)": make_pipeline(StandardScaler(),
                                  MLPClassifier(hidden_layer_sizes=(256, 256, 256),
                                                max_iter=500, random_state=42)),
}

for name, m in models.items():
    s = cross_val_score(m, X, y, cv=5, scoring="f1", n_jobs=-1)
    print(f"{name:20s}: F1 = {s.mean():.3f} ± {s.std():.3f}")

print("\nنتیجه: روی داده‌ی جدولی، boosting اغلب از شبکه بهتر و سریع‌تر است.")
```

**مثال ۲ — جدول تصمیم:**

```python
def choose_approach(n_samples, data_type, needs_interpretability=False, has_gpu=False):
    if needs_interpretability:
        return "مدل تفسیرپذیر: Logistic، درخت، یا boosting با SHAP"
    if data_type == "tabular":
        if n_samples < 5000:
            return "Gradient Boosting + مهندسی ویژگی"
        return "LightGBM/XGBoost (اول)، شبکه اگر بهتر شد"
    if data_type in ("image", "audio", "text"):
        if n_samples < 1000:
            return "Transfer learning از مدل پیش‌آموزش‌دیده"
        if n_samples < 50_000:
            return "Fine-tune مدل پیش‌آموزش‌دیده"
        return "آموزش کامل یا fine-tune عمیق" + ("" if has_gpu else " — GPU لازم است")
    return "مشخص نیست"

print(choose_approach(2000, "tabular"))
print(choose_approach(50_000, "tabular"))
print(choose_approach(500, "image"))
print(choose_approach(100_000, "text", has_gpu=True))
```

### مقایسه و دیدگاه‌های مختلف

| | ML کلاسیک | یادگیری عمیق |
|---|---|---|
| داده‌ی لازم | کم | زیاد |
| ویژگی | دستی | خودکار |
| سخت‌افزار | CPU | GPU |
| تفسیر | بالا | پایین |
| زمان توسعه | سریع | کند |
| بهترین برای | جدولی | خام و پیچیده |

**دو دیدگاه:**
- **«یادگیری عمیق همه‌چیز را حل می‌کند.»** برای تصویر/متن، تا حد زیادی. برای جدول، نه.
- **«یادگیری عمیق فقط تبلیغ است.»** غلط: در بینایی، ترجمه و تولید متن، انقلابی واقعی بود.

### اشتباهات رایج

**۱. شبکه برای داده‌ی جدولی کوچک.** boosting بهتر، سریع‌تر، تفسیرپذیرتر.
**۲. آموزش از صفر بدون transfer learning.** با داده‌ی کم، شکست.
**۳. نادیده گرفتن هزینه‌ی GPU.** مدل بزرگ، هزینه‌ی استنتاج هم دارد.
**۴. انتخاب چارچوب بر اساس مد.** هر دو کار می‌کنند؛ آن‌چه می‌دانی را انتخاب کن.
**۵. انتظار تفسیرپذیری.** شبکه‌ی عمیق جعبه‌سیاه است.

### نکات مهم و ترفندها

- **اول boosting، بعد شبکه.** قاعده‌ی صنعتی.
- **برای تصویر/متن، هرگز از صفر شروع نکن.** transfer learning.
- **PyTorch را یاد بگیر** اگر می‌خواهی مقالات جدید را دنبال کنی.
- **Colab/Kaggle GPU رایگان** برای یادگیری کافی است.
- **میان‌بر:** جدولی → boosting · تصویر/متن → DL + transfer.

### خلاصه فصل

- یادگیری عمیق، شبکه با لایه‌های زیاد است.
- عمق، سلسله‌مراتب ویژگی می‌سازد.
- برای داده‌ی جدولی، boosting اغلب بهتر است.
- برای داده‌ی خام و پیچیده، DL برنده است.
- PyTorch غالب تحقیق، TF/Keras قوی در تولید.

**سؤالات مروری:**
1. سه دلیل که عمق کار می‌کند چیست؟
2. چه زمانی یادگیری عمیق انتخاب اشتباهی است؟
3. چرا برای داده‌ی جدولی boosting ترجیح داده می‌شود؟

---

## فصل ۷۸: Keras / TensorFlow: اولین شبکه در ۲۰ خط

### مقدمه

Keras، ساده‌ترین راه ساخت شبکه است: چند خط، یک مدل کامل. برای شروع و پروتوتایپ عالی است.

این فصل، هم Keras را نشان می‌دهد و هم **محدودیت‌هایش** را — چون در نهایت به PyTorch نیاز پیدا می‌کنی.

### مفاهیم اساسی

**۱. سه سبک ساخت مدل در Keras:**

```python
# ۱. Sequential — ساده‌ترین
model = keras.Sequential([...])

# ۲. Functional API — چند ورودی/خروجی
inputs = keras.Input(shape=(10,))
x = layers.Dense(64, activation="relu")(inputs)
outputs = layers.Dense(1)(x)
model = keras.Model(inputs, outputs)

# ۳. Subclassing — کنترل کامل
class MyModel(keras.Model):
    def __init__(self):
        super().__init__()
        self.dense = layers.Dense(1)
    def call(self, x):
        return self.dense(x)
```

**۲. چرخه‌ی کار:**

```python
model.compile(optimizer, loss, metrics)
model.fit(X, y, epochs, batch_size, validation_data)
model.evaluate(X_test, y_test)
model.predict(X_new)
```

**۳. انتخاب loss و activation خروجی:**

| مسئله | خروجی | Loss |
|---|---|---|
| رگرسیون | ۱ نورون، خطی | `mse` |
| دودویی | ۱ نورون، sigmoid | `binary_crossentropy` |
| چندکلاسه (انحصاری) | k نورون، softmax | `categorical_crossentropy` |
| چندکلاسه (چندبرچسبی) | k نورون، sigmoid | `binary_crossentropy` |

**۴. Callbacks — ابزارهای آموزش:**

```python
keras.callbacks.EarlyStopping(patience=10, restore_best_weights=True)
keras.callbacks.ModelCheckpoint("best.keras", save_best_only=True)
keras.callbacks.ReduceLROnPlateau(factor=0.5, patience=5)
keras.callbacks.TensorBoard(log_dir="./logs")
```

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — شبکه‌ی کامل روی داده‌ی جدولی:**

```python
try:
    import tensorflow as tf
    from tensorflow import keras
    from tensorflow.keras import layers
    import numpy as np
    from sklearn.datasets import make_classification
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler

    tf.random.set_seed(42)

    X, y = make_classification(n_samples=5000, n_features=20, n_informative=12,
                               flip_y=0.05, random_state=42)
    X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2,
                                              stratify=y, random_state=42)

    scaler = StandardScaler().fit(X_tr)
    X_tr, X_te = scaler.transform(X_tr), scaler.transform(X_te)

    model = keras.Sequential([
        layers.Input(shape=(20,)),
        layers.Dense(128, activation="relu"),
        layers.Dropout(0.3),
        layers.Dense(64, activation="relu"),
        layers.Dropout(0.2),
        layers.Dense(1, activation="sigmoid"),
    ])

    model.compile(
        optimizer=keras.optimizers.Adam(1e-3),
        loss="binary_crossentropy",
        metrics=["accuracy", keras.metrics.AUC(name="auc")],
    )

    model.summary()

    history = model.fit(
        X_tr, y_tr,
        epochs=50,
        batch_size=64,
        validation_split=0.2,
        callbacks=[
            keras.callbacks.EarlyStopping(patience=8, restore_best_weights=True,
                                          monitor="val_auc", mode="max"),
            keras.callbacks.ReduceLROnPlateau(patience=3, factor=0.5),
        ],
        verbose=0,
    )

    loss, acc, auc = model.evaluate(X_te, y_te, verbose=0)
    print(f"\ntest loss={loss:.4f} | acc={acc:.3f} | auc={auc:.3f}")
    print(f"توقف در epoch: {len(history.history['loss'])}")
except ImportError:
    print("نصب کن: pip install tensorflow")
```

**این ۳۰ خط، یک مدل کامل با early stopping و کاهش lr است.**

**مثال ۲ — رسم منحنی آموزش:**

```python
try:
    import matplotlib.pyplot as plt

    fig, axes = plt.subplots(1, 2, figsize=(12, 4))

    axes[0].plot(history.history["loss"], label="train")
    axes[0].plot(history.history["val_loss"], label="val")
    axes[0].set_title("Loss")
    axes[0].legend(); axes[0].grid(alpha=0.3)

    axes[1].plot(history.history["accuracy"], label="train")
    axes[1].plot(history.history["val_accuracy"], label="val")
    axes[1].set_title("Accuracy")
    axes[1].legend(); axes[1].grid(alpha=0.3)

    plt.tight_layout()
    plt.show()
except Exception as e:
    print("رسم نمودار نیاز به history دارد:", e)
```

**مثال ۳ — Functional API برای چند ورودی:**

```python
try:
    # دو ورودی: عددی و دسته‌ای
    num_input = keras.Input(shape=(10,), name="numeric")
    cat_input = keras.Input(shape=(5,), name="categorical")

    x1 = layers.Dense(64, activation="relu")(num_input)
    x2 = layers.Dense(32, activation="relu")(cat_input)

    merged = layers.Concatenate()([x1, x2])
    x = layers.Dense(32, activation="relu")(merged)
    output = layers.Dense(1, activation="sigmoid")(x)

    model = keras.Model(inputs=[num_input, cat_input], outputs=output)
    model.summary()
except ImportError:
    print("pip install tensorflow")
```

**مثال ۴ — ذخیره و بارگذاری:**

```python
# ذخیره
# model.save("model.keras")

# بارگذاری
# loaded = keras.models.load_model("model.keras")

# فقط وزن‌ها
# model.save_weights("weights.weights.h5")
# model.load_weights("weights.weights.h5")
```

### مقایسه و دیدگاه‌های مختلف

| ویژگی | Keras | PyTorch |
|---|---|---|
| ساخت مدل | ۳ خط | ۱۰ خط |
| حلقه‌ی آموزش | خودکار (`fit`) | دستی |
| دیباگ | سخت‌تر | آسان‌تر |
| انعطاف | متوسط | بالا |
| تولید | عالی (TF Serving) | خوب (TorchServe) |

**یک دیدگاه:** «Keras برای شروع، PyTorch برای جدی.» تقریباً درست. اما Keras برای ۸۰٪ پروژه‌های کاربردی کافی است.

### اشتباهات رایج

**۱. فراموش کردن `Input(shape=...)`.** خطای شکل در اولین fit.
**۲. loss نامناسب.** `mse` برای دسته‌بندی، آموزش را کند می‌کند.
**۳. نبود `restore_best_weights`.** مدل overfit ذخیره می‌شود.
**۴. `validation_split` بعد از shuffle.** داده‌ی زمانی به‌هم می‌ریزد.
**۵. نادیده گرفتن `model.summary()`.** تعداد پارامترها را نمی‌دانی.
**۶. استفاده از `predict` به جای `predict_proba`.** در Keras، `predict` خودش احتمال می‌دهد (برای sigmoid/softmax).

### نکات مهم و ترفندها

- **`restore_best_weights=True` را همیشه بگذار.**
- **`model.summary()` را بعد از ساخت چاپ کن.**
- **`validation_split=0.2` برای شروع سریع؛ برای جدی، validation جدا.**
- **`tf.keras` را استفاده کن، نه `keras` مستقل** (در پروژه‌های جدید، `keras 3` هم کار می‌کند).
- **میان‌بر:** Sequential → compile → fit → evaluate.

### خلاصه فصل

- Keras ساده‌ترین راه ساخت شبکه است.
- سه سبک: Sequential، Functional، Subclassing.
- loss و activation خروجی باید با مسئله هماهنگ باشند.
- Callbacks آموزش را حرفه‌ای می‌کنند.
- برای کنترل کامل، PyTorch لازم می‌شود.

**سؤالات مروری:**
1. برای دسته‌بندی چندکلاسه، چه خروجی و چه loss؟
2. `restore_best_weights` چه کاری می‌کند؟
3. تفاوت Sequential و Functional API چیست؟

---

## فصل ۷۹: PyTorch: تنسور، autograd، حلقه‌ی آموزش

### مقدمه

PyTorch استاندارد تحقیق و بخش بزرگی از صنعت است. سه مفهوم اصلی دارد: **Tensor** (آرایه با GPU و گرادیان)، **autograd** (مشتق خودکار)، و **حلقه‌ی آموزش صریح**.

اگر فایل ۱۶ را خوانده باشی، PyTorch برایت آشناست — فقط `backward()` جای کد دستی را می‌گیرد.

### مفاهیم اساسی

**۱. Tensor — NumPy با دو قابلیت اضافه:**

```python
import torch

x = torch.tensor([1.0, 2.0, 3.0])
x = torch.randn(3, 4)
x = torch.zeros(2, 2)
x = torch.arange(10)

x.device      # cpu / cuda:0
x.dtype       # float32
x.shape       # torch.Size([...])
```

**۲. `requires_grad` — کلید autograd:**

```python
w = torch.randn(3, requires_grad=True)
y = (w ** 2).sum()
y.backward()
print(w.grad)      # dy/dw = 2w
```

**۳. سه مرحله‌ی استاندارد:**

```python
optimizer.zero_grad()      # ۱. پاک کردن گرادیان قبلی
loss = criterion(model(x), y)
loss.backward()            # ۲. محاسبه‌ی گرادیان
optimizer.step()           # ۳. به‌روزرسانی
```

**۴. `train()` و `eval()`:**

```python
model.train()      # Dropout فعال، BatchNorm با آمار batch
model.eval()       # Dropout خاموش، BatchNorm با آمار تجمعی
```

**۵. `no_grad()` برای استنتاج:**

```python
with torch.no_grad():
    preds = model(X)
```
حافظه کمتر، سرعت بیشتر.

**۶. Dataset و DataLoader:**

```python
class MyDataset(Dataset):
    def __init__(self, X, y): self.X, self.y = X, y
    def __len__(self): return len(self.X)
    def __getitem__(self, i): return self.X[i], self.y[i]

loader = DataLoader(ds, batch_size=32, shuffle=True, num_workers=0)
```

**۷. مدیریت device:**

```python
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
model.to(device)
x = x.to(device)
```
**خطای رایج:** مدل روی GPU، داده روی CPU → `RuntimeError`.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — autograd در عمل:**

```python
try:
    import torch

    # مشتق ساده
    x = torch.tensor(3.0, requires_grad=True)
    y = x ** 2 + 2 * x + 1
    y.backward()
    print(f"y = x²+2x+1 در x=3 → dy/dx = {x.grad.item()} (انتظار: 8)")

    # مشتق جزئی
    a = torch.tensor([1.0, 2.0], requires_grad=True)
    b = torch.tensor([3.0, 4.0], requires_grad=True)
    z = (a * b).sum()
    z.backward()
    print(f"dz/da = {a.grad} (انتظار: [3,4])")
    print(f"dz/db = {b.grad} (انتظار: [1,2])")
except ImportError:
    print("pip install torch")
```

**مثال ۲ — شبکه‌ی کامل، از تعریف تا ارزیابی:**

```python
try:
    import torch
    import torch.nn as nn
    from torch.utils.data import DataLoader, TensorDataset
    import numpy as np
    from sklearn.datasets import make_classification
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import StandardScaler

    torch.manual_seed(42)
    device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

    # داده
    X, y = make_classification(n_samples=5000, n_features=20, n_informative=12,
                               flip_y=0.05, random_state=42)
    X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2,
                                              stratify=y, random_state=42)
    scaler = StandardScaler().fit(X_tr)
    X_tr, X_te = scaler.transform(X_tr), scaler.transform(X_te)

    train_ds = TensorDataset(torch.tensor(X_tr, dtype=torch.float32),
                             torch.tensor(y_tr, dtype=torch.long))
    test_ds = TensorDataset(torch.tensor(X_te, dtype=torch.float32),
                            torch.tensor(y_te, dtype=torch.long))

    train_loader = DataLoader(train_ds, batch_size=64, shuffle=True)
    test_loader = DataLoader(test_ds, batch_size=256)

    # مدل
    class MLP(nn.Module):
        def __init__(self, n_in=20, n_out=2):
            super().__init__()
            self.net = nn.Sequential(
                nn.Linear(n_in, 128), nn.ReLU(), nn.Dropout(0.3),
                nn.Linear(128, 64), nn.ReLU(), nn.Dropout(0.2),
                nn.Linear(64, n_out),
            )

        def forward(self, x):
            return self.net(x)

    model = MLP().to(device)
    criterion = nn.CrossEntropyLoss()
    optimizer = torch.optim.AdamW(model.parameters(), lr=1e-3, weight_decay=1e-4)

    print(f"دستگاه: {device}")
    print(f"پارامترها: {sum(p.numel() for p in model.parameters()):,}")

    # آموزش
    for epoch in range(30):
        model.train()
        total_loss = 0.0
        for xb, yb in train_loader:
            xb, yb = xb.to(device), yb.to(device)
            optimizer.zero_grad()
            loss = criterion(model(xb), yb)
            loss.backward()
            optimizer.step()
            total_loss += loss.item() * len(xb)

        if epoch % 10 == 0:
            model.eval()
            correct = total = 0
            with torch.no_grad():
                for xb, yb in test_loader:
                    xb, yb = xb.to(device), yb.to(device)
                    preds = model(xb).argmax(1)
                    correct += (preds == yb).sum().item()
                    total += len(yb)
            print(f"epoch {epoch:2d} | loss={total_loss/len(train_ds):.4f} | "
                  f"test acc={correct/total:.3f}")
except ImportError:
    print("pip install torch")
```

**مثال ۳ — تنسور در برابر NumPy:**

```python
try:
    import torch
    import numpy as np

    a_np = np.array([1.0, 2.0, 3.0])
    a_pt = torch.from_numpy(a_np)

    print("NumPy:", a_np)
    print("Tensor:", a_pt)
    print("تبدیل برگشتی:", a_pt.numpy())

    # عملیات مشابه
    print("جمع:", (a_pt + 1).tolist())
    print("ضرب ماتریسی:", (torch.randn(2, 3) @ torch.randn(3, 4)).shape)
    print("GPU موجود؟", torch.cuda.is_available())
except ImportError:
    print("pip install torch")
```

**مثال ۴ — حلقه‌ی آموزش کامل با early stopping:**

```python
def train_loop(model, train_loader, val_loader, epochs=100, lr=1e-3,
               patience=10, device="cpu"):
    import copy
    criterion = torch.nn.CrossEntropyLoss()
    optimizer = torch.optim.AdamW(model.parameters(), lr=lr, weight_decay=1e-4)

    best_loss, best_state, wait = float("inf"), None, 0

    for epoch in range(epochs):
        model.train()
        for xb, yb in train_loader:
            xb, yb = xb.to(device), yb.to(device)
            optimizer.zero_grad()
            loss = criterion(model(xb), yb)
            loss.backward()
            torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
            optimizer.step()

        # validation
        model.eval()
        val_loss, n = 0.0, 0
        with torch.no_grad():
            for xb, yb in val_loader:
                xb, yb = xb.to(device), yb.to(device)
                val_loss += criterion(model(xb), yb).item() * len(xb)
                n += len(xb)
        val_loss /= n

        if val_loss < best_loss - 1e-4:
            best_loss, wait = val_loss, 0
            best_state = copy.deepcopy(model.state_dict())
        else:
            wait += 1
            if wait >= patience:
                print(f"early stop @ epoch {epoch}")
                break

    model.load_state_dict(best_state)
    return model, best_loss
```

**مثال ۵ — ذخیره و بارگذاری:**

```python
# torch.save(model.state_dict(), "model.pt")
# model.load_state_dict(torch.load("model.pt", map_location=device))
# model.eval()

# ذخیره‌ی کامل (برای ادامه‌ی آموزش)
# torch.save({
#     "model": model.state_dict(),
#     "optimizer": optimizer.state_dict(),
#     "epoch": epoch,
# }, "checkpoint.pt")
```

### مقایسه و دیدگاه‌های مختلف

| ویژگی | PyTorch | Keras |
|---|---|---|
| حلقه‌ی آموزش | صریح | خودکار |
| دیباگ | آسان (`print` در حلقه) | سخت‌تر |
| گراف | Dynamic | Static (TF) / Dynamic (Keras 3) |
| اکوسیستم پژوهش | غالب | کمتر |
| استقرار | TorchServe، ONNX | TF Serving، TFLite |

**یک دیدگاه:** «PyTorch برای تحقیق، TF برای تولید.» این مرز در حال محو شدن است — PyTorch در تولید هم رایج شده.

### اشتباهات رایج

**۱. `zero_grad()` فراموش شده.** گرادیان‌ها جمع می‌شوند.
**۲. `model.eval()` در استنتاج فراموش شده.** Dropout فعال می‌ماند.
**۳. نبود `no_grad()`.** حافظه‌ی اضافی مصرف می‌شود.
**۴. device ناهماهنگ.** مدل روی GPU، داده روی CPU.
**۵. `torch.tensor` روی داده‌ی numpy با dtype اشتباه.** `float64` به جای `float32` → خطا.
**۶. نبود `shuffle=True` در DataLoader آموزش.**
**۷. ذخیره‌ی مدل بدون `state_dict`.** pickle شکننده است.

### نکات مهم و ترفندها

- **`model.train()` / `model.eval()` را عادت کن.**
- **`optimizer.zero_grad()` اولین خط حلقه.**
- **`clip_grad_norm_` را همیشه بگذار.**
- **`torch.no_grad()` در ارزیابی، ۲ برابر سریع‌تر.**
- **`state_dict` را ذخیره کن، نه کل مدل.**
- **میان‌بر:** forward → loss → backward → step → zero_grad.

### خلاصه فصل

- Tensor، NumPy با GPU و گرادیان است.
- autograd مشتق را خودکار حساب می‌کند.
- سه مرحله: zero_grad → backward → step.
- train/eval و no_grad، برای درستی و سرعت لازم‌اند.
- Dataset/DataLoader، مدیریت batch را ساده می‌کنند.

**سؤالات مروری:**
1. `requires_grad=True` چه کاری می‌کند؟
2. چرا `zero_grad()` لازم است؟
3. تفاوت `model.train()` و `model.eval()` چیست؟

---

## فصل ۸۰: CNN: بینایی ماشین به زبان ساده

### مقدمه

شبکه‌ی عصبی معمولی (MLP) روی تصویر، دو مشکل دارد: **پارامترهای نجومی** و **نادیده گرفتن ساختار فضایی**.

CNN این دو را حل می‌کند: با **فیلترهای محلی** و **اشتراک وزن**. نتیجه: مدلی که لبه، بافت و شکل را در هر جای تصویر تشخیص می‌دهد — با پارامترهای بسیار کمتر.

### مفاهیم اساسی

**۱. مشکل MLP روی تصویر:**

یک تصویر ۲۲۴×۲۲۴×۳ = ۱۵۰٬۵۲۸ عدد. یک لایه با ۱۰۰۰ نورون = **۱۵۰ میلیون پارامتر**. غیرعملی.

**۲. سه ایده‌ی کلیدی CNN:**

- **اتصال محلی (Local Connectivity):** هر نورون فقط به یک ناحیه‌ی کوچک نگاه می‌کند. لبه‌ها محلی‌اند.
- **اشتراک وزن (Weight Sharing):** همان فیلتر در کل تصویر اعمال می‌شود. یک «تشخیص‌دهنده‌ی لبه» برای همه‌ی جای تصویر.
- **تجمع (Pooling):** کاهش ابعاد با حفظ اطلاعات مهم.

**۳. کانولوشن (Convolution) — عملیات اصلی:**

```
فیلتر ۳×۳ روی تصویر می‌لغزد و در هر موقعیت، ضرب داخلی می‌گیرد
→ نقشه‌ی ویژگی (feature map)
```

**پارامترها:**
- **kernel_size:** اندازه‌ی فیلتر (۳×۳ استاندارد).
- **stride:** گام (۱ = همان اندازه، ۲ = نصف).
- **padding:** حاشیه (`same` = حفظ اندازه، `valid` = بدون حاشیه).
- **filters:** تعداد فیلتر (هر فیلتر، یک الگو را یاد می‌گیرد).

**۴. Pooling:**

- **Max Pooling:** بیشترین مقدار در هر ناحیه. رایج‌ترین.
- **Average Pooling:** میانگین.
- **Global Average Pooling:** میانگین کل → یک عدد به ازای هر کانال. در معماری‌های مدرن.

**۵. محاسبه‌ی اندازه‌ی خروجی:**

```
out = (in - kernel + 2×padding) / stride + 1
```
مثال: ۳۲ ورودی، kernel=3، padding=1، stride=1 → `(32-3+2)/1+1 = 32`.

**۶. معماری کلاسیک CNN:**

```
تصویر (224×224×3)
  → Conv(32, 3×3) + ReLU
  → MaxPool(2×2)          → 112×112×32
  → Conv(64, 3×3) + ReLU
  → MaxPool(2×2)          → 56×56×64
  → Conv(128, 3×3) + ReLU
  → GlobalAvgPool         → 128
  → Dense(n_classes)
```

**۷. معماری‌های مهم:**

| معماری | نوآوری | سال |
|---|---|---|
| LeNet-5 | اولین CNN موفق | ۱۹۹۸ |
| AlexNet | ReLU + Dropout + GPU | ۲۰۱۲ |
| VGG | عمق با فیلتر ۳×۳ | ۲۰۱۴ |
| **ResNet** | اتصال باقی‌مانده (residual) | ۲۰۱۵ |
| EfficientNet | مقیاس‌بندی هوشمند | ۲۰۱۹ |
| ViT | ترانسفورمر برای تصویر | ۲۰۲۰ |

**۸. اتصال باقی‌مانده (Residual Connection) — مهم‌ترین نوآوری:**

```
out = F(x) + x
```
به جای یادگیری `F(x)`، مدل یاد می‌گیرد `F(x)` را به `x` **اضافه** کند. گرادیان مسیر مستقیم دارد → شبکه‌های ۱۰۰+ لایه ممکن شدند.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — کانولوشن دستی با NumPy:**

```python
import numpy as np

def conv2d(image, kernel, stride=1, padding=0):
    """کانولوشن ۲بعدی ساده — بدون کانال."""
    if padding > 0:
        image = np.pad(image, padding, mode="constant")

    h, w = image.shape
    kh, kw = kernel.shape
    out_h = (h - kh) // stride + 1
    out_w = (w - kw) // stride + 1
    out = np.zeros((out_h, out_w))

    for i in range(out_h):
        for j in range(out_w):
            patch = image[i*stride:i*stride+kh, j*stride:j*stride+kw]
            out[i, j] = np.sum(patch * kernel)

    return out

# تصویر ساده با یک لبه‌ی عمودی
image = np.array([
    [0, 0, 1, 1, 1],
    [0, 0, 1, 1, 1],
    [0, 0, 1, 1, 1],
    [0, 0, 1, 1, 1],
    [0, 0, 1, 1, 1],
], dtype=float)

# فیلتر تشخیص لبه‌ی عمودی (Sobel)
kernel = np.array([[-1, 0, 1],
                   [-1, 0, 1],
                   [-1, 0, 1]], dtype=float)

result = conv2d(image, kernel, padding=0)
print("خروجی کانولوشن (لبه‌ی عمودی):")
print(result)
print("\nمقادیر بزرگ = لبه‌ی قوی")
```

**مثال ۲ — محاسبه‌ی اندازه‌ها:**

```python
def conv_output_size(size, kernel, padding=0, stride=1):
    return (size - kernel + 2 * padding) // stride + 1

print(f"{'ورودی':>8} {'kernel':>8} {'pad':>5} {'stride':>7} {'خروجی':>8}")
for size, k, p, s in [(32, 3, 0, 1), (32, 3, 1, 1), (32, 3, 1, 2), (224, 7, 3, 2)]:
    print(f"{size:>8} {k:>8} {p:>5} {s:>7} {conv_output_size(size, k, p, s):>8}")
```

**مثال ۳ — CNN با PyTorch:**

```python
try:
    import torch
    import torch.nn as nn

    class SimpleCNN(nn.Module):
        def __init__(self, n_classes=10):
            super().__init__()
            self.features = nn.Sequential(
                nn.Conv2d(1, 32, 3, padding=1), nn.ReLU(),
                nn.MaxPool2d(2),                          # 28 → 14
                nn.Conv2d(32, 64, 3, padding=1), nn.ReLU(),
                nn.MaxPool2d(2),                          # 14 → 7
                nn.Conv2d(64, 128, 3, padding=1), nn.ReLU(),
                nn.AdaptiveAvgPool2d(1),                  # → 1×1
            )
            self.classifier = nn.Sequential(
                nn.Flatten(),
                nn.Dropout(0.3),
                nn.Linear(128, n_classes),
            )

        def forward(self, x):
            return self.classifier(self.features(x))

    model = SimpleCNN()
    x = torch.randn(8, 1, 28, 28)
    print("خروجی:", model(x).shape)      # (8, 10)
    print(f"پارامترها: {sum(p.numel() for p in model.parameters()):,}")
    print("\nتوجه: فقط ~۱۰۰ هزار پارامتر — در برابر ۱۵۰ میلیون MLP!")
except ImportError:
    print("pip install torch")
```

**مثال ۴ — آموزش CNN روی MNIST:**

```python
try:
    import torch
    import torch.nn as nn
    from torch.utils.data import DataLoader
    from torchvision import datasets, transforms

    transform = transforms.Compose([
        transforms.ToTensor(),
        transforms.Normalize((0.1307,), (0.3081,)),      # آمار MNIST
    ])

    train_ds = datasets.MNIST("./data", train=True, download=True, transform=transform)
    test_ds = datasets.MNIST("./data", train=False, download=True, transform=transform)

    train_loader = DataLoader(train_ds, batch_size=128, shuffle=True)
    test_loader = DataLoader(test_ds, batch_size=512)

    device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
    model = SimpleCNN().to(device)
    optimizer = torch.optim.AdamW(model.parameters(), lr=1e-3)
    criterion = nn.CrossEntropyLoss()

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
    print("pip install torch torchvision")
except Exception as e:
    print("دانلود MNIST نیاز به اینترنت دارد:", type(e).__name__)
```

**انتظار:** دقت بالای ۰.۹۸ بعد از ۳ epoch. **این، قدرت CNN است.**

**مثال ۵ — Transfer Learning (روش واقعی):**

```python
try:
    from torchvision import models

    # مدل از پیش آموزش‌دیده
    model = models.resnet18(weights=models.ResNet18_Weights.DEFAULT)

    # فریز کردن
    for p in model.parameters():
        p.requires_grad = False

    # جایگزینی لایه‌ی آخر
    model.fc = nn.Linear(model.fc.in_features, 10)
    print(model.fc)
    print(f"پارامتر آموزش‌پذیر: {sum(p.numel() for p in model.parameters() if p.requires_grad):,}")
except ImportError:
    print("pip install torch torchvision")
```

### مقایسه و دیدگاه‌های مختلف

| لایه | پارامتر | نقش |
|---|---|---|
| Conv2d(3→32, 3×3) | 896 | تشخیص الگوهای ساده |
| Conv2d(32→64, 3×3) | 18,496 | ترکیب الگوها |
| Linear(128→10) | 1,290 | تصمیم نهایی |
| MLP معادل | میلیون‌ها | — |

**یک دیدگاه:** «CNN منسوخ شده، ViT جایش را گرفته.» نه کاملاً: CNNها سریع‌تر و با داده‌ی کمتر بهترند. ViT با داده‌ی عظیم برنده است. **در عمل، هر دو استفاده می‌شوند** (و EfficientNet/ConvNeXt ترکیبی هستند).

### اشتباهات رایج

**۱. فراموش کردن `padding`.** ابعاد در هر لایه کوچک می‌شود.
**۲. نبود BatchNorm در CNN عمیق.** آموزش ناپایدار.
**۳. نبود augmentation.** با داده‌ی کم، overfit سریع.
**۴. نرمال‌سازی با آمار اشتباه.** برای ImageNet pretrained، باید `mean=[0.485,0.456,0.406]` باشد.
**۵. Flatten قبل از Global Pooling.** پارامترهای زیاد.
**۶. نبود `model.eval()`.** BatchNorm رفتار غلط.
**۷. آموزش از صفر با داده‌ی کم.** transfer learning استفاده کن.

### نکات مهم و ترفندها

- **kernel=3×3 با padding=1** استاندارد است.
- **MaxPool 2×2** برای نصف کردن ابعاد.
- **Global Average Pooling** به جای Flatten، پارامترها را کم می‌کند.
- **Data augmentation** برای تصویر تقریباً واجب است.
- **transfer learning** نقطه‌ی شروع همیشگی.
- **میان‌بر:** Conv → ReLU → Pool → ... → GlobalPool → Dense.

### خلاصه فصل

- CNN با اتصال محلی و اشتراک وزن، پارامترها را کم می‌کند.
- کانولوشن، فیلتر را روی تصویر می‌لغزاند.
- Pooling ابعاد را کم می‌کند.
- Residual connection شبکه‌های عمیق را ممکن کرد.
- Transfer learning، روش استاندارد برای داده‌ی کم است.

**سؤالات مروری:**
1. چرا CNN از MLP برای تصویر بهتر است؟
2. خروجی Conv با ورودی ۳۲، kernel ۳، padding ۱، stride ۲ چند است؟
3. Residual connection چه مشکلی را حل می‌کند؟

---

## فصل ۸۱: RNN و داده‌های ترتیبی

### مقدمه

تا اینجا، همه‌ی مدل‌ها فرض می‌کردند نمونه‌ها **مستقل** هستند. اما در متن، صدا و سری زمانی، **ترتیب مهم است**: «سگ گربه را گاز گرفت» با «گربه سگ را گاز گرفت» فرق دارد.

RNN برای همین ساخته شد. امروز ترانسفورمرها جایگزینش شده‌اند، اما **مفهوم حالت پنهان** همچنان بنیادی است.

### مفاهیم اساسی

**۱. ایده‌ی RNN: حافظه.**

```
h_t = tanh(W_h · h_{t-1} + W_x · x_t + b)
y_t = W_y · h_t
```

- `h_t`: حالت پنهان در زمان t (خلاصه‌ی گذشته)
- `x_t`: ورودی فعلی
- همان وزن‌ها در همه‌ی گام‌ها (اشتراک وزن زمانی)

**۲. مشکل RNN ساده: محوشدگی گرادیان.**

برای یادگیری وابستگی‌های دور (مثلاً کلمه‌ی اول جمله و کلمه‌ی آخر)، گرادیان باید از ۵۰ گام عبور کند. با tanh (مشتق <۱)، گرادیان صفر می‌شود.

**۳. LSTM — راه‌حل با «دروازه».**

سه دروازه:
- **Forget gate:** چه چیزی از حافظه فراموش شود؟
- **Input gate:** چه چیزی اضافه شود؟
- **Output gate:** چه چیزی خارج شود؟

```
f_t = σ(W_f·[h_{t-1}, x_t])
i_t = σ(W_i·[h_{t-1}, x_t])
o_t = σ(W_o·[h_{t-1}, x_t])
c_t = f_t ⊙ c_{t-1} + i_t ⊙ tanh(W_c·[h_{t-1}, x_t])
h_t = o_t ⊙ tanh(c_t)
```

**`c_t` (حالت سلول) مسیر مستقیمی دارد** → گرادیان بدون ضرب مکرر عبور می‌کند. این، راه‌حل محوشدگی است.

**۴. GRU — نسخه‌ی ساده‌تر LSTM.** دو دروازه به جای سه، پارامتر کمتر، عملکرد مشابه.

**۵. انواع معماری برای وظایف مختلف:**

| وظیفه | معماری |
|---|---|
| many-to-one | دسته‌بندی متن (یک برچسب برای کل جمله) |
| one-to-many | تولید متن از یک ورودی |
| many-to-many (هم‌طول) | برچسب‌گذاری هر کلمه (POS tagging) |
| seq2seq | ترجمه (encoder-decoder) |

**۶. چرا ترانسفورمر جایگزین شد؟**
- **موازی‌سازی:** RNN ترتیبی است (گام t به t-1 وابسته)؛ ترانسفورمر همه را هم‌زمان می‌بیند.
- **وابستگی دور:** attention، فاصله را حذف می‌کند.
- **مقیاس‌پذیری:** روی GPU خیلی سریع‌تر.

**اما:** برای سری زمانی کوچک، RNN/LSTM هنوز ساده‌تر و سبک‌تر است.

**۷. Brute-force جایگزین:** برای بسیاری از مسائل سری زمانی، **ویژگی‌های lag** + boosting بهتر از RNN کار می‌کنند:
```python
df["lag_1"] = df["value"].shift(1)
df["lag_7"] = df["value"].shift(7)
df["rolling_mean_7"] = df["value"].rolling(7).mean()
```
**این را قبل از RNN امتحان کن.**

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — RNN ساده از صفر:**

```python
import numpy as np

class SimpleRNN:
    """RNN تک‌لایه برای نمایش مفهوم."""
    def __init__(self, input_size, hidden_size, seed=0):
        rng = np.random.default_rng(seed)
        self.Wx = rng.normal(0, 0.1, (input_size, hidden_size))
        self.Wh = rng.normal(0, 0.1, (hidden_size, hidden_size))
        self.b = np.zeros(hidden_size)
        self.hidden_size = hidden_size

    def forward(self, X):
        """X: (seq_len, input_size) → خروجی: (seq_len, hidden_size)"""
        h = np.zeros(self.hidden_size)
        states = []
        for x_t in X:
            h = np.tanh(x_t @ self.Wx + h @ self.Wh + self.b)
            states.append(h)
        return np.array(states)

rnn = SimpleRNN(input_size=5, hidden_size=8)
X_seq = np.random.randn(10, 5)          # ۱۰ گام زمانی، ۵ ویژگی
states = rnn.forward(X_seq)
print("شکل حالت‌ها:", states.shape)     # (10, 8)
print("حالت آخر (خلاصه‌ی کل دنباله):", states[-1].round(2))
```

**مثال ۲ — LSTM با PyTorch:**

```python
try:
    import torch
    import torch.nn as nn

    class LSTMClassifier(nn.Module):
        def __init__(self, input_size, hidden_size=64, n_layers=1, n_classes=2):
            super().__init__()
            self.lstm = nn.LSTM(input_size, hidden_size, n_layers,
                                batch_first=True, dropout=0.2 if n_layers > 1 else 0)
            self.head = nn.Sequential(
                nn.Dropout(0.3),
                nn.Linear(hidden_size, n_classes),
            )

        def forward(self, x):
            # x: (batch, seq_len, input_size)
            out, (h_n, c_n) = self.lstm(x)
            last = out[:, -1, :]        # آخرین گام زمانی
            return self.head(last)

    model = LSTMClassifier(input_size=10)
    x = torch.randn(16, 30, 10)        # batch=16، seq=30، features=10
    print("خروجی:", model(x).shape)     # (16, 2)
    print(f"پارامترها: {sum(p.numel() for p in model.parameters()):,}")
except ImportError:
    print("pip install torch")
```

**مثال ۳ — پیش‌بینی سری زمانی با lag features (جایگزین ساده):**

```python
import numpy as np
import pandas as pd
from sklearn.ensemble import GradientBoostingRegressor
from sklearn.metrics import mean_absolute_error

rng = np.random.default_rng(0)
n = 500
t = np.arange(n)
series = 100 + 0.1 * t + 10 * np.sin(2 * np.pi * t / 30) + rng.normal(0, 2, n)

df = pd.DataFrame({"value": series})
for lag in [1, 2, 3, 7, 14, 30]:
    df[f"lag_{lag}"] = df["value"].shift(lag)
df["rolling_mean_7"] = df["value"].shift(1).rolling(7).mean()
df["rolling_std_7"] = df["value"].shift(1).rolling(7).std()
df["day_of_week"] = t % 7

df = df.dropna()
X = df.drop(columns="value")
y = df["value"]

split = int(len(df) * 0.8)
model = GradientBoostingRegressor(n_estimators=200, random_state=42)
model.fit(X.iloc[:split], y.iloc[:split])

pred = model.predict(X.iloc[split:])
print(f"MAE: {mean_absolute_error(y.iloc[split:], pred):.2f}")
print("\nمهم‌ترین ویژگی‌ها:")
print(pd.Series(model.feature_importances_, index=X.columns).nlargest(5).round(3))
```

**این روش، در عمل اغلب از RNN بهتر و سریع‌تر است.**

**مثال ۴ — متن با LSTM (مفهومی):**

```python
try:
    import torch
    import torch.nn as nn

    class TextClassifier(nn.Module):
        def __init__(self, vocab_size, embed_dim=64, hidden=64, n_classes=2):
            super().__init__()
            self.embedding = nn.Embedding(vocab_size, embed_dim, padding_idx=0)
            self.lstm = nn.LSTM(embed_dim, hidden, batch_first=True, bidirectional=True)
            self.fc = nn.Linear(hidden * 2, n_classes)

        def forward(self, x):
            emb = self.embedding(x)
            out, _ = self.lstm(emb)
            # میانگین‌گیری روی گام‌های غیرصفر
            mask = (x != 0).unsqueeze(-1).float()
            pooled = (out * mask).sum(1) / mask.sum(1).clamp(min=1)
            return self.fc(pooled)

    model = TextClassifier(vocab_size=10_000)
    tokens = torch.randint(1, 10_000, (8, 50))     # batch=8، طول=50
    print("خروجی:", model(tokens).shape)
except ImportError:
    print("pip install torch")
```

### مقایسه و دیدگاه‌های مختلف

| مدل | موازی‌سازی | وابستگی دور | پارامتر | کِی |
|---|---|---|---|---|
| RNN | ❌ | ضعیف | کم | آموزشی |
| LSTM/GRU | ❌ | خوب | متوسط | سری زمانی کوچک |
| Transformer | ✅ | عالی | زیاد | متن، دنباله‌ی بلند |
| Lag + Boosting | ✅ | محدود به lag | کم | سری زمانی جدولی |

**یک دیدگاه:** «RNN مرده است.» برای متن، تقریباً. برای سری زمانی کوچک و نه‌چندان طولانی، هنوز گزینه‌ی معقولی است.

### اشتباهات رایج

**۱. RNN ساده برای دنباله‌ی طولانی.** محوشدگی. LSTM استفاده کن.
**۲. فراموش کردن `batch_first=True`.** شکل‌ها جابه‌جا می‌شوند.
**۳. نبود `pack_padded_sequence`.** دنباله‌های با طول متفاوت، محاسبات را خراب می‌کنند.
**۴. نبود gradient clipping.** انفجار گرادیان در RNN شایع است.
**۵. RNN برای سری زمانی جدولی.** اول lag features را امتحان کن.
**۶. نادیده گرفتن ترتیب در تقسیم داده.** سری زمانی باید زمانی تقسیم شود.

### نکات مهم و ترفندها

- **`clip_grad_norm_(params, 1.0)` برای RNN واجب است.**
- **LSTM دوطرفه (bidirectional)** برای متن (نه برای سری زمانی آینده).
- **Lag features را قبل از RNN امتحان کن.**
- **برای متن امروز، از Transformer استفاده کن.** RNN برای یادگیری مفهوم خوب است.
- **میان‌بر:** حالت پنهان = حافظه · دروازه = کنترل جریان · attention = جایگزین مدرن.

### خلاصه فصل

- RNN با حالت پنهان، ترتیب را مدل می‌کند.
- محوشدگی گرادیان، مشکل اصلی RNN ساده است.
- LSTM با دروازه‌ها آن را حل می‌کند.
- ترانسفورمر با موازی‌سازی، جایگزین RNN شد.
- برای سری زمانی جدولی، lag + boosting اغلب بهتر است.

**سؤالات مروری:**
1. حالت پنهان در RNN چه چیزی را نگه می‌دارد؟
2. LSTM چگونه محوشدگی گرادیان را حل می‌کند؟
3. چرا ترانسفورمر از RNN سریع‌تر است؟

---

## منابع و مراجع

**مستندات**
- PyTorch — *Learn the Basics*, *Tutorials*
- TensorFlow/Keras — *Guide*, *API docs*
- torchvision — *Models and pretrained weights*

**کتاب‌ها**
- Géron, A. — *Hands-On ML*، فصل ۱۰ (ANN)، ۱۴ (CNN)، ۱۵ (RNN)
- Prince, S. — *Understanding Deep Learning*، فصل ۵–۹
- Goodfellow, I. et al. — *Deep Learning*، فصل ۹ (CNN)، ۱۰ (RNN)

**مقالات کلیدی**
- LeCun, Y. et al. (1998) — *Gradient-Based Learning Applied to Document Recognition* (LeNet)
- Krizhevsky, A. et al. (2012) — *AlexNet*
- He, K. et al. (2016) — *Deep Residual Learning* (ResNet)
- Hochreiter, S. & Schmidhuber, J. (1997) — *Long Short-Term Memory*
- Dosovitskiy, A. et al. (2020) — *An Image is Worth 16x16 Words* (ViT)

**دوره‌ها**
- PyTorch — *Deep Learning with PyTorch: Zero to GANs*
- fast.ai — *Practical Deep Learning for Coders*

---

**بعدی:** [`18-LLM-Generative-AI.md`](./18-LLM-Generative-AI.md) — فصل ۸۲: مدل زبانی چیست؟
