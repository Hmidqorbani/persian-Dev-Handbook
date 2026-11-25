# بخش ۱۲: یادگیری تقویتی

**مدت یادگیری تقریبی**: ۲ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۴ (احتمال)، فایل ۰۸ (مبانی ML)

---

## فهرست محتوا

- [فصل ۵۱: عامل، محیط، پاداش: بازی به‌عنوان قالب ذهنی](#فصل-۵۱-عامل-محیط-پاداش-بازی-بهعنوان-قالب-ذهنی)
- [فصل ۵۲: اکتشاف در برابر بهره‌برداری](#فصل-۵۲-اکتشاف-در-برابر-بهرهبرداری)
- [فصل ۵۳: Q-Learning و معادله‌ی بلمن](#فصل-۵۳-q-learning-و-معادلهی-بلمن)
- [فصل ۵۴: کاربردهای واقعی: بازی، رباتیک، پیشنهاددهی](#فصل-۵۴-کاربردهای-واقعی-بازی-رباتیک-پیشنهاددهی)

---

## فصل ۵۱: عامل، محیط، پاداش: بازی به‌عنوان قالب ذهنی

### مقدمه

سه نوع یادگیری قبلی، یک ویژگی مشترک داشتند: **جواب درست را می‌دانستیم** (یا حداقل ساختارش را). یادگیری تقویتی این را ندارد. فقط می‌داند «چقدر خوب بود»، آن هم **با تأخیر**.

این تفاوت، همه‌چیز را عوض می‌کند: مسئله به یک **فرآیند تصمیم‌گیری ترتیبی** تبدیل می‌شود که در آن، انتخاب امروز بر گزینه‌های فردا اثر می‌گذارد.

### مفاهیم اساسی

**۱. پنج جزء هر مسئله‌ی RL:**

| جزء | معنا | مثال شطرنج |
|---|---|---|
| **Agent (عامل)** | تصمیم‌گیرنده | بازیکن |
| **Environment (محیط)** | دنیایی که عامل در آن است | صفحه‌ی شطرنج + حریف |
| **State (وضعیت)** | تصویر فعلی دنیا | چیدمان مهره‌ها |
| **Action (کنش)** | کاری که عامل می‌کند | حرکت مهره |
| **Reward (پاداش)** | بازخورد عددی | +۱ برد، -۱ باخت، ۰ ادامه |

**۲. چرخه‌ی تعامل:**

```
وضعیت sₜ
  → عامل کنش aₜ انتخاب می‌کند
  → محیط پاداش rₜ و وضعیت جدید sₜ₊₁ می‌دهد
  → تکرار
```

**۳. سیاست (Policy) — چیزی که یاد گرفته می‌شود.**

```
π(a | s) = احتمال انتخاب کنش a در وضعیت s
```

هدف: پیدا کردن سیاستی که **بازده تجمعی (return)** را بیشینه کند:

```
Gₜ = rₜ + γ·rₜ₊₁ + γ²·rₜ₊₂ + ...
```

`γ` (ضریب تخفیف، بین ۰ و ۱): چقدر آینده برایمان مهم است.
- `γ = 0`: فقط پاداش فوری.
- `γ = 0.99`: آینده‌ی دور هم مهم است.

**۴. سه چالش بنیادی RL:**

| چالش | توضیح |
|---|---|
| **تأخیر پاداش** | نمی‌دانی کدام حرکت از ۵۰ حرکت قبل، باعث برد شد |
| **اکتشاف/بهره‌برداری** | آزمودن چیزهای جدید یا استفاده از دانسته‌ها؟ |
| **داده‌ی غیر i.i.d.** | برخلاف ML معمولی، داده‌ها به هم وابسته‌اند (چون تصمیم‌ها مسیر را عوض می‌کنند) |

**۵. سه خانواده‌ی روش:**

- **Value-based:** ارزش هر وضعیت/کنش را یاد بگیر (Q-Learning، DQN).
- **Policy-based:** مستقیماً سیاست را بهینه کن (REINFORCE، PPO).
- **Actor-Critic:** ترکیب هر دو (A2C، SAC).

**۶. تفاوت با با ناظر، در یک جمله:**

> در با ناظر می‌گوییم «این کار را بکن». در تقویتی می‌گوییم «این کار چقدر خوب بود» — و مدل باید خودش بفهمد کدام کار بهتر است.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — یک محیط ساده (Grid World):**

```python
import numpy as np

class GridWorld:
    """دنیای ۴×۴: رسیدن به هدف +۱، افتادن در چاله -۱."""
    def __init__(self, size=4):
        self.size = size
        self.start = (0, 0)
        self.goal = (size - 1, size - 1)
        self.holes = {(1, 1), (2, 2)}
        self.state = self.start
        self.actions = [(-1, 0), (1, 0), (0, -1), (0, 1)]   # بالا، پایین، چپ، راست

    def reset(self):
        self.state = self.start
        return self.state

    def step(self, action):
        r, c = self.state
        dr, dc = self.actions[action]
        nr = min(max(r + dr, 0), self.size - 1)
        nc = min(max(c + dc, 0), self.size - 1)
        self.state = (nr, nc)

        if self.state == self.goal:
            return self.state, 1.0, True
        if self.state in self.holes:
            return self.state, -1.0, True
        return self.state, -0.01, False      # جریمه‌ی کوچک برای هر قدم

env = GridWorld()
state = env.reset()
print("وضعیت شروع:", state)

for step in range(5):
    a = np.random.default_rng(step).integers(0, 4)
    state, reward, done = env.step(a)
    print(f"کنش {a} → وضعیت {state} | پاداش {reward} | پایان: {done}")
    if done:
        break
```

**مثال ۲ — بازده تجمعی با تخفیف:**

```python
def discounted_return(rewards, gamma=0.9):
    """Gₜ = Σ γᵏ rₜ₊ₖ — چرا پاداش دور، ارزش کمتری دارد."""
    total = 0.0
    for k, r in enumerate(rewards):
        total += (gamma ** k) * r
    return total

print(round(discounted_return([0, 0, 0, 1], gamma=0.9), 4))    # ۰.۷۲۹
print(round(discounted_return([1, 0, 0, 0], gamma=0.9), 4))    # ۱.۰
```

**درس:** پاداش فوری ارزشمندتر از پاداش دور است. همین، `γ` را معنا می‌دهد.

**مثال ۳ — سیاست تصادفی در برابر سیاست بهینه:**

```python
def run_episode(env, policy, max_steps=100):
    state = env.reset()
    total = 0.0
    for _ in range(max_steps):
        action = policy(state)
        state, reward, done = env.step(action)
        total += reward
        if done:
            break
    return total, state

# سیاست تصادفی
random_policy = lambda s: np.random.randint(0, 4)
scores = [run_episode(GridWorld(), random_policy)[0] for _ in range(100)]
print(f"سیاست تصادفی: میانگین بازده = {np.mean(scores):.3f}")

# سیاست دستی (همیشه راست و پایین — به هدف می‌رسد اما از چاله می‌گذرد)
def smart_policy(state):
    r, c = state
    if r < 3:
        return 1     # پایین
    return 3         # راست
scores2 = [run_episode(GridWorld(), smart_policy)[0] for _ in range(100)]
print(f"سیاست دستی:   میانگین بازده = {np.mean(scores2):.3f}")
```

### مقایسه و دیدگاه‌های مختلف

| معیار | با ناظر | تقویتی |
|---|---|---|
| بازخورد | جواب درست، فوری | پاداش، تأخیری |
| داده | مستقل | وابسته به مسیر |
| هدف | پیش‌بینی | بیشینه کردن بازده |
| ارزیابی | دقت | بازده تجمعی |
| نمونه | تشخیص اسپم | بازی، ربات |

**دو دیدگاه:**
- **«RL برای هر مسئله‌ی تصمیم مناسب است.»** نه: اگر بتوانی با ناظر حل کنی، RL پیچیدگی بی‌مورد است.
- **«RL فقط برای بازی است.»** نه: در رباتیک، کنترل صنعتی، تنظیم سیستم‌های توزیع‌شده و حتی RLHF (فایل ۱۸) استفاده می‌شود.

### اشتباهات رایج

**۱. استفاده از RL برای مسئله‌ای که با ناظر حل می‌شود.** RL داده‌بر، ناپایدار و سخت است.
**۲. `γ` نامناسب.** `γ=1` در مسائل بی‌پایان، بازده واگرا می‌شود.
**۳. فراموش کردن جریمه‌ی هر قدم.** عامل بی‌هدف می‌چرخد.
**۴. انتظار همگرایی سریع.** RL معمولاً هزاران اپیزود لازم دارد.
**۵. طراحی بد تابع پاداش.** عامل دقیقاً همان چیزی را بهینه می‌کند که گفتی، نه آنچه می‌خواستی (reward hacking).
**۶. نادیده گرفتن وابستگی داده.** نمونه‌ها i.i.d. نیستند؛ نمی‌توانی تصادفی shuffle کنی.

### نکات مهم و ترفندها

- **اول بپرس: «می‌توانم این را با ناظر حل کنم؟»** اگر بله، RL را رها کن.
- **تابع پاداش را دقیق طراحی کن.** بیشترین منبع شکست RL، پاداش بد است، نه الگوریتم.
- **با محیط ساده شروع کن** (Grid World، CartPole). RL روی مسائل پیچیده، بی‌رحمانه است.
- **`γ` را ۰.۹ تا ۰.۹۹ بگذار** برای اکثر مسائل.
- **میان‌بر:** عامل + محیط + وضعیت + کنش + پاداش = حلقه‌ی RL.

### خلاصه فصل

- RL با بازخورد تأخیری و ترتیبی کار می‌کند.
- پنج جزء: عامل، محیط، وضعیت، کنش، پاداش.
- سیاست، نگاشت وضعیت به کنش است؛ هدف، بیشینه کردن بازده تجمعی.
- `γ` اهمیت آینده را تنظیم می‌کند.
- سه خانواده: value-based، policy-based، actor-critic.

**سؤالات مروری:**
1. پنج جزء RL را نام ببر.
2. چرا `γ` کوچک‌تر از ۱ معمولاً لازم است؟
3. تفاوت اصلی RL با یادگیری با ناظر چیست؟

---

## فصل ۵۲: اکتشاف در برابر بهره‌برداری

### مقدمه

این فصل، **بنیادی‌ترین دوراهی RL** را توضیح می‌دهد:

> رستوران همیشگی‌ات را بروی (بهره‌برداری از دانسته‌ها)، یا رستوان جدید را امتحان کنی (اکتشاف)؟

اگر همیشه اولی را انتخاب کنی، ممکن است هرگز رستوران بهتری پیدا نکنی. اگر همیشه دومی، ممکن است گرسنه بمانی. **تعادل، کل مسئله است.**

### مفاهیم اساسی

**۱. تعریف دقیق:**

- **Exploitation:** انتخاب کنشی که **فعلاً** بهترین تخمین را دارد.
- **Exploration:** انتخاب کنشی که اطلاعات جدید می‌دهد.

**۲. مسئله‌ی باندیت چندبازویی (Multi-Armed Bandit).**

ساده‌ترین مسئله‌ی RL: `k` ماشین اسلات، هر کدام توزیع پاداش ناشناخته‌ای دارد. چند بار هر کدام را بکشی تا بفهمی کدام بهترین است؟

**این، مدل انتزاعی بسیاری از مسائل واقعی است:** کدام تبلیغ؟ کدام نسخه‌ی UI؟ کدام درمان؟

**۳. چهار استراتژی:**

| استراتژی | ایده | مزیت | عیب |
|---|---|---|---|
| **ε-greedy** | با احتمال ε تصادفی، وگرنه بهترین | ساده | اکتشاف بی‌هدف |
| **ε-decay** | ε به‌مرور کم شود | تعادل خودکار | نیاز به برنامه |
| **UCB** | «بهترین تخمین + عدم‌قطعیت» | اکتشاف هوشمند | نیاز به شمارش |
| **Thompson Sampling** | نمونه‌گیری از توزیع باور | Bayesian، عالی در عمل | پیچیده‌تر |

**۴. UCB (Upper Confidence Bound):**

```
انتخاب کن: argmax [ Q(a) + c·√(ln(t) / N(a)) ]
```
- `Q(a)`: پاداش تخمینی
- جمله‌ی دوم: **پاداش عدم‌قطعیت** — بازوهایی که کمتر آزموده شده‌اند، امتیاز اضافی می‌گیرند.

**این، «خوش‌بینی در برابر عدم‌قطعیت» است:** بازویی که نمی‌شناسی، ممکن است عالی باشد.

**۵. چرا اکتشاف در RL سخت‌تر از باندیت است؟**
چون اکتشاف، **مسیر** را عوض می‌کند. در باندیت، هر انتخاب مستقل است؛ در RL، انتخاب امروز، وضعیت فردا را تغییر می‌دهد.

**۶. روش‌های اکتشاف پیشرفته:**
- **ε-greedy با نویز:** ساده و مؤثر.
- **Entropy bonus:** پاداش برای تصادفی‌بودن سیاست (PPO).
- **Curiosity / Intrinsic reward:** پاداش برای دیدن وضعیت‌های جدید.
- **Count-based:** پاداش برای وضعیت‌های کم‌دیده.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — شبیه‌سازی باندیت با سه استراتژی:**

```python
import numpy as np

class Bandit:
    """k بازو با احتمال موفقیت ناشناخته."""
    def __init__(self, probs, seed=0):
        self.probs = np.array(probs)
        self.rng = np.random.default_rng(seed)

    def pull(self, arm):
        return float(self.rng.random() < self.probs[arm])

def run_epsilon_greedy(bandit, n_steps=2000, eps=0.1, seed=0):
    rng = np.random.default_rng(seed)
    k = len(bandit.probs)
    counts = np.zeros(k)
    values = np.zeros(k)
    rewards = []

    for _ in range(n_steps):
        if rng.random() < eps:
            arm = rng.integers(k)
        else:
            arm = int(np.argmax(values))

        r = bandit.pull(arm)
        counts[arm] += 1
        values[arm] += (r - values[arm]) / counts[arm]    # میانگین تدریجی
        rewards.append(r)

    return np.array(rewards), counts

def run_ucb(bandit, n_steps=2000, c=2.0):
    k = len(bandit.probs)
    counts = np.zeros(k)
    values = np.zeros(k)
    rewards = []

    for t in range(1, n_steps + 1):
        if 0 in counts:
            arm = int(np.argmin(counts))       # اول هر بازو را یک بار بکش
        else:
            ucb = values + c * np.sqrt(np.log(t) / counts)
            arm = int(np.argmax(ucb))

        r = bandit.pull(arm)
        counts[arm] += 1
        values[arm] += (r - values[arm]) / counts[arm]
        rewards.append(r)

    return np.array(rewards), counts

probs = [0.3, 0.5, 0.7, 0.4]
for name, runner in [
    ("ε=0.1", lambda b: run_epsilon_greedy(b, eps=0.1)),
    ("ε=0.01", lambda b: run_epsilon_greedy(b, eps=0.01)),
    ("UCB", run_ucb),
]:
    rewards, counts = runner(Bandit(probs))
    print(f"{name:8s} | میانگین پاداش={rewards.mean():.3f} | "
          f"کشش بازوها={counts.astype(int)}")
```

**خروجی معمول:** UCB بهترین است، ε=0.01 کمترین اکتشاف و بیشترین بهره‌برداری، ε=0.1 اکتشاف بیشتر. **بازوی ۳ (۰.۷) باید بیشترین کشش را داشته باشد.**

**مثال ۲ — اثر ε بر عملکرد (نمایش کمی):**

```python
for eps in [0, 0.01, 0.05, 0.1, 0.3, 0.5]:
    rewards, counts = run_epsilon_greedy(Bandit(probs), eps=eps)
    regret = 2000 * max(probs) - rewards.sum()
    print(f"ε={eps:<5} | پاداش کل={rewards.sum():>6.0f} | regret={regret:>6.0f}")
```

**`regret` (پشیمانی):** تفاوت بین بهترین عملکرد ممکن و عملکرد واقعی. **معیار استاندارد ارزیابی در RL.**

**مثال ۳ — ε-decay:**

```python
def run_epsilon_decay(bandit, n_steps=2000, eps_start=1.0, eps_end=0.01, decay=0.995):
    rng = np.random.default_rng(0)
    k = len(bandit.probs)
    counts = np.zeros(k); values = np.zeros(k)
    eps = eps_start
    rewards = []

    for _ in range(n_steps):
        arm = rng.integers(k) if rng.random() < eps else int(np.argmax(values))
        r = bandit.pull(arm)
        counts[arm] += 1
        values[arm] += (r - values[arm]) / counts[arm]
        rewards.append(r)
        eps = max(eps_end, eps * decay)

    return np.array(rewards), counts, eps

rewards, counts, final_eps = run_epsilon_decay(Bandit(probs))
print(f"میانگین پاداش: {rewards.mean():.3f} | ε نهایی: {final_eps:.4f}")
```

**منطق:** اول زیاد کاوش کن (چون چیزی نمی‌دانی)، بعد کم‌کم بهره‌برداری کن. **این، الگوی استاندارد در عمل است.**

### مقایسه و دیدگاه‌های مختلف

| روش | سرعت یادگیری | پایداری | پیچیدگی | محبوبیت |
|---|---|---|---|---|
| ε-greedy | متوسط | بالا | خیلی کم | بسیار زیاد |
| ε-decay | خوب | بالا | کم | بسیار زیاد |
| UCB | سریع | بالا | کم | متوسط |
| Thompson | سریع | بالا | متوسط | در حال رشد |

**دو دیدگاه:**
- **«ε-greedy کافی است.»** در عمل، برای اکثر مسائل، بله.
- **«اکتشاف هوشمند لازم است.»** در مسائل با پاداش پرنویز یا فضاهای بزرگ، بله.

### اشتباهات رایج

**۱. ε ثابت در کل آموزش.** در پایان، هنوز ۱۰٪ تصادفی عمل می‌کند. از decay استفاده کن.
**۲. ε = 0.** هرگز چیز جدید یاد نمی‌گیرد؛ در بهینه‌ی محلی گیر می‌کند.
**۳. ε = 1.** هیچ‌وقت از دانسته‌ها استفاده نمی‌کند.
**۴. اکتشاف در محیطی که ریسک فاجعه دارد.** در پزشکی، نمی‌توانی «تصادفی» درمان کنی. به safety constraints نیاز داری.
**۵. نادیده گرفتن `regret`.** دقت نهایی کافی نیست؛ سرعت یادگیری هم مهم است.
**۶. مقایسه‌ی الگوریتم‌ها با یک seed.** RL پرنویز است؛ چند seed لازم است.

### نکات مهم و ترفندها

- **ε-decay را پیش‌فرض بگیر.** از ۱.۰ به ۰.۰۱ در طول آموزش.
- **`regret` را اندازه بگیر، نه فقط پاداش نهایی.**
- **UCB برای مسائل با پاداش پایدار عالی است.**
- **در محیط‌های حساس، اکتشاف را محدود کن.** اکتشاف بی‌قید، خطرناک است.
- **میان‌بر:** اکتشاف = یادگیری · بهره‌برداری = سود · تعادل = سیاست.

### خلاصه فصل

- دوراهی اکتشاف/بهره‌برداری، هسته‌ی RL است.
- ε-greedy ساده‌ترین راه‌حل است؛ ε-decay در عمل بهتر.
- UCB با پاداش عدم‌قطعیت، اکتشاف هوشمند انجام می‌دهد.
- `regret` معیار استاندارد ارزیابی است.
- در محیط‌های پرخطر، اکتشاف باید محدود شود.

**سؤالات مروری:**
1. چرا ε = 0 و ε = 1 هر دو بد هستند؟
2. UCB چگونه اکتشاف را هوشمند می‌کند؟
3. `regret` چیست و چرا مهم است؟

---

## فصل ۵۳: Q-Learning و معادله‌ی بلمن

### مقدمه

Q-Learning، **مشهورترین الگوریتم RL** است. ساده، بدون نیاز به مدل محیط، و پایه‌ی Deep Q-Network (DQN) که در ۲۰۱۵ بازی Atari را از انسان بهتر بازی کرد.

ایده‌ی مرکزی، یک تابع است: `Q(s, a)` — «چقدر خوب است اگر در وضعیت `s` باشم و کنش `a` را انجام بدهم، و بعد بهترین کار را ادامه بدهم؟»

### مفاهیم اساسی

**۱. تابع Q (ارزش کنش-وضعیت).**

```
Q(s, a) = پاداش فوری + γ × بهترین Q در وضعیت بعدی
```

این، **معادله‌ی بلمن** است:

```
Q(s, a) ← Q(s, a) + α [ r + γ·max Q(s', a') − Q(s, a) ]
```

اجزا:
- `α`: نرخ یادگیری
- `r + γ·max Q(s',a')`: **هدف TD** (تخمین بهتر)
- `[...]`: **خطای TD** (تفاوت تخمین فعلی و بهتر)
- `max`: بهترین کنش در وضعیت بعدی

**۲. چرا این کار می‌کند؟** چون معادله‌ی بلمن یک **برابری** است در حالت بهینه. با تکرار، تخمین‌ها به هم نزدیک می‌شوند (contraction mapping).

**۳. الگوریتم Q-Learning:**

```
برای هر اپیزود:
  s ← وضعیت شروع
  تا پایان:
    a ← ε-greedy بر اساس Q(s, ·)
    s', r ← محیط
    Q(s,a) ← Q(s,a) + α[r + γ·max Q(s',·) − Q(s,a)]
    s ← s'
```

**نکته‌ی مهم: Q-Learning «off-policy» است.** از سیاست ε-greedy برای اکتشاف استفاده می‌کند، اما **بهترین** کنش را برای به‌روزرسانی (`max`) — یعنی یاد می‌گیرد بهترین سیاست ممکن را، حتی اگر خودش رفتار متفاوتی داشته باشد.

**۴. SARSA در برابر Q-Learning:**

| | Q-Learning | SARSA |
|---|---|---|
| به‌روزرسانی | `max Q(s',·)` | `Q(s',a')` که واقعاً انتخاب شد |
| نوع | Off-policy | On-policy |
| رفتار | جسور، بهینه | محافظه‌کار، ایمن‌تر |

**در محیط‌های پرخطر، SARSA بهتر است** چون مسیر واقعی خودش را یاد می‌گیرد، نه مسیر بهینه‌ای که ممکن است خطرناک باشد.

**۵. Deep Q-Network (DQN).** وقتی فضا بزرگ است (مثل پیکسل‌های تصویر)، جدول Q غیرممکن می‌شود. راه‌حل: شبکه‌ی عصبی که `Q(s,a)` را تقریب بزند.

سه ترفند کلیدی DQN:
- **Experience Replay:** نمونه‌ها را در buffer ذخیره کن و تصادفی نمونه‌گیری کن (شکستن همبستگی).
- **Target Network:** شبکه‌ی هدف که هر چند هزار قدم کپی می‌شود (پایداری).
- **Reward Clipping:** پاداش‌ها را به [-۱, ۱] محدود کن.

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — Q-Learning کامل روی Grid World:**

```python
import numpy as np

class GridWorld:
    def __init__(self, size=5):
        self.size = size
        self.goal = (size - 1, size - 1)
        self.holes = {(1, 1), (2, 3)}
        self.actions = [(-1, 0), (1, 0), (0, -1), (0, 1)]

    def reset(self):
        return (0, 0)

    def step(self, state, action):
        r, c = state
        dr, dc = self.actions[action]
        nr = min(max(r + dr, 0), self.size - 1)
        nc = min(max(c + dc, 0), self.size - 1)
        new_state = (nr, nc)

        if new_state == self.goal:
            return new_state, 1.0, True
        if new_state in self.holes:
            return new_state, -1.0, True
        return new_state, -0.01, False

def q_learning(env, episodes=3000, alpha=0.1, gamma=0.95,
               eps_start=1.0, eps_end=0.02, decay=0.995):
    n = env.size
    Q = np.zeros((n, n, 4))
    eps = eps_start
    rng = np.random.default_rng(0)
    returns = []

    for ep in range(episodes):
        s = env.reset()
        total = 0.0

        for _ in range(200):
            if rng.random() < eps:
                a = rng.integers(4)
            else:
                a = int(np.argmax(Q[s[0], s[1]]))

            s2, r, done = env.step(s, a)
            best_next = np.max(Q[s2[0], s2[1]])
            td_target = r if done else r + gamma * best_next
            Q[s[0], s[1], a] += alpha * (td_target - Q[s[0], s[1], a])

            total += r
            s = s2
            if done:
                break

        returns.append(total)
        eps = max(eps_end, eps * decay)

    return Q, returns

env = GridWorld()
Q, returns = q_learning(env)

print(f"میانگین بازده ۱۰۰ اپیزود اول: {np.mean(returns[:100]):.3f}")
print(f"میانگین بازده ۱۰۰ اپیزود آخر: {np.mean(returns[-100:]):.3f}")

# سیاست بهینه
arrows = ["↑", "↓", "←", "→"]
print("\nسیاست بهینه:")
for r in range(env.size):
    row = []
    for c in range(env.size):
        if (r, c) == env.goal:
            row.append(" G")
        elif (r, c) in env.holes:
            row.append(" X")
        else:
            row.append(" " + arrows[int(np.argmax(Q[r, c]))])
    print("".join(row))
```

**این ۴۰ خط، Q-Learning کامل است.** خروجی، یک سیاست بهینه روی شبکه است.

**مثال ۲ — اجرای سیاست یادگرفته‌شده:**

```python
def run_policy(env, Q, max_steps=50):
    s = env.reset()
    path = [s]
    for _ in range(max_steps):
        a = int(np.argmax(Q[s[0], s[1]]))
        s, r, done = env.step(s, a)
        path.append(s)
        if done:
            return path, r
    return path, None

path, final_reward = run_policy(env, Q)
print("مسیر:", path)
print("پاداش نهایی:", final_reward)
```

**مثال ۳ — اثر gamma:**

```python
for gamma in [0.5, 0.9, 0.99]:
    _, rets = q_learning(GridWorld(), episodes=1500, gamma=gamma)
    print(f"gamma={gamma:<5} | بازده میانگین (۱۰۰ آخر): {np.mean(rets[-100:]):.3f}")
```

`gamma` کوچک → عامل فقط به پاداش نزدیک فکر می‌کند و ممکن است در مسیرهای بی‌پایان گیر کند.

**مثال ۴ — SARSA در برابر Q-Learning (مفهوم):**

```python
def sarsa(env, episodes=1500, alpha=0.1, gamma=0.95, eps=0.1):
    Q = np.zeros((env.size, env.size, 4))
    rng = np.random.default_rng(0)
    returns = []

    for _ in range(episodes):
        s = env.reset()
        a = rng.integers(4) if rng.random() < eps else int(np.argmax(Q[s[0], s[1]]))
        total = 0.0

        for _ in range(200):
            s2, r, done = env.step(s, a)
            a2 = rng.integers(4) if rng.random() < eps else int(np.argmax(Q[s2[0], s2[1]]))
            target = r if done else r + gamma * Q[s2[0], s2[1], a2]
            Q[s[0], s[1], a] += alpha * (target - Q[s[0], s[1], a])
            total += r
            s, a = s2, a2
            if done:
                break

        returns.append(total)
    return Q, returns
```

### مقایسه و دیدگاه‌های مختلف

| روش | نیاز به مدل محیط | فضا | پایداری | نمونه |
|---|---|---|---|---|
| Q-Learning (جدولی) | خیر | کوچک | بالا | Grid World |
| SARSA | خیر | کوچک | بالا | محیط پرخطر |
| DQN | خیر | بزرگ | متوسط | Atari |
| PPO | خیر | بزرگ/پیوسته | بالا | رباتیک، LLM |
| SAC | خیر | پیوسته | بالا | کنترل |

**دو دیدگاه:**
- **«Q-Learning پایه است، ولی در عمل PPO/SAC بهترند.»** در مسائل پیچیده، درست.
- **«برای فهم RL، باید از Q-Learning شروع کنی.»** همیشه درست.

### اشتباهات رایج

**۱. `α` بزرگ.** Q واگرا می‌شود. ۰.۰۱ تا ۰.۱ امن است.
**۲. `γ` = ۱ در مسئله‌ی بی‌پایان.** بازده محدود نمی‌شود.
**۳. اکتشاف صفر.** Q هرگز وضعیت‌های جدید را نمی‌بیند.
**۴. `max` را روی همه‌ی کنش‌ها زدن در وضعیت پایانی.** باید `done` را جدا کنی (در کد بالا انجام شد).
**۵. نبود Experience Replay در DQN.** داده همبسته، آموزش را ناپایدار می‌کند.
**۶. فراموش کردن Target Network.** Q به خودش تکیه می‌کند و واگرا می‌شود.
**۷. مقایسه با یک seed.** RL پرنویز است.

### نکات مهم و ترفندها

- **از `done` در محاسبه‌ی هدف استفاده کن.** `target = r if done else r + γ·max Q(s')`.
- **`α` را کم شروع کن.** ۰.۱ نقطه‌ی خوبی است.
- **ε-decay واجب است.**
- **برای فضاهای بزرگ، DQN با replay و target network.**
- **`regret` و منحنی یادگیری را رسم کن.** یک عدد نهایی کافی نیست.
- **میان‌بر:** `Q ← Q + α[هدف − Q]` · هدف = پاداش + بهترین آینده.

### خلاصه فصل

- Q-Learning ارزش هر جفت (وضعیت، کنش) را یاد می‌گیرد.
- معادله‌ی بلمن، پایه‌ی به‌روزرسانی است.
- off-policy بودن، مزیت اصلی آن است.
- DQN با replay و target network، Q-Learning را به فضاهای بزرگ می‌برد.
- SARSA در محیط‌های پرخطر محافظه‌کارتر و ایمن‌تر است.

**سؤالات مروری:**
1. معادله‌ی به‌روزرسانی Q-Learning را بنویس و هر جزء را توضیح بده.
2. تفاوت Q-Learning و SARSA چیست؟
3. چرا DQN به Experience Replay نیاز دارد؟

---

## فصل ۵۴: کاربردهای واقعی: بازی، رباتیک، پیشنهاددهی

### مقدمه

RL در آزمایشگاه جذاب است؛ در دنیای واقعی، سخت. این فصل نشان می‌دهد RL **کجا واقعاً کار می‌کند**، کجا شکست می‌خورد، و چطور در محصولات واقعی استفاده می‌شود.

مهم‌ترین درس این فصل: **RL در مسائلی که «شبیه‌ساز» دارند، موفق است.**

### مفاهیم اساسی

**۱. پنج حوزه‌ی موفق:**

| حوزه | مثال | چرا RL؟ |
|---|---|---|
| **بازی** | AlphaGo، Dota، StarCraft | شبیه‌ساز بی‌نهایت، پاداش روشن |
| **رباتیک** | راه رفتن، گرفتن اشیا | تصمیم ترتیبی، بازخورد حسی |
| **سیستم‌ها** | خنک‌سازی دیتاسنتر (DeepMind) | بهینه‌سازی مصرف انرژی |
| **تبلیغات/پیشنهاد** | انتخاب محتوا | پاداش بلندمدت، نه کلیک فوری |
| **تنظیم مدل زبانی (RLHF)** | آموزش ChatGPT | هم‌راستا کردن با ترجیح انسانی |

**۲. چرا RL در تولید کم است؟**

| چالش | توضیح |
|---|---|
| **هزینه‌ی نمونه** | هر تعامل، هزینه دارد (کاربر واقعی، ربات واقعی) |
| **شبیه‌ساز ناکافی** | sim-to-real gap: مدل در شبیه‌ساز عالی، در واقعیت بد |
| **ناپایداری** | آموزش حساس به seed و هایپرپارامتر |
| **ارزیابی سخت** | نمی‌توانی A/B test کنی بدون ریسک |
| **پاداش دشوار** | تعریف پاداش درست، سخت‌ترین بخش |

**۳. سه راهکار صنعتی:**

- **Offline RL:** از داده‌ی لاگ‌شده یاد بگیر، بدون تعامل آنلاین.
- **Contextual Bandit:** نسخه‌ی ساده‌شده که در صنعت بسیار استفاده می‌شود (پیشنهاد، تبلیغ).
- **Simulation + Domain Randomization:** شبیه‌ساز را متنوع کن تا انتقال بهتر شود.

**۴. Contextual Bandit — پرکاربردترین شکل RL در صنعت.**

```
وضعیت (context) = پروفایل کاربر
کنش = کدام محتوا/تبلیغ
پاداش = کلیک / خرید
```
بدون حالت‌های ترتیبی پیچیده. ساده، قابل A/B test، و مؤثر.

**۵. RLHF — پیوند RL و مدل‌های زبانی.**

```
۱. مدل را با داده‌ی انسانی fine-tune کن (SFT)
۲. یک مدل پاداش از مقایسه‌های انسانی آموزش بده
۳. با PPO، مدل زبانی را برای بیشینه کردن پاداش تنظیم کن
```

این، همان روشی است که ChatGPT را «مؤدب و مفید» کرد (فایل ۱۸).

**۶. reward hacking — تله‌ی بزرگ.**

عامل دقیقاً همان چیزی را بهینه می‌کند که **گفتی**، نه آنچه **می‌خواستی**.
مثال کلاسیک: رباتی که برای «جمع کردن توپ» پاداش می‌گیرد، یاد می‌گیرد توپ را روی سرش نگه دارد و بی‌حرکت بماند — چون پاداش برای «نزدیک بودن» بود، نه «گذاشتن در سبد».

### مثال‌ها و نمونه‌های عملی

**مثال ۱ — Contextual Bandit با ε-greedy (شبیه‌سازی تبلیغات):**

```python
import numpy as np

class AdBandit:
    """سه تبلیغ با نرخ کلیک متفاوت برای دو گروه کاربری."""
    def __init__(self, seed=0):
        self.ctr = {
            "جوان": [0.05, 0.02, 0.08],
            "مسن": [0.07, 0.09, 0.03],
        }
        self.rng = np.random.default_rng(seed)

    def show(self, context, ad):
        return float(self.rng.random() < self.ctr[context][ad])

def run_contextual(n_steps=5000, eps=0.1):
    env = AdBandit()
    contexts = ["جوان", "مسن"]
    Q = {c: np.zeros(3) for c in contexts}
    N = {c: np.zeros(3) for c in contexts}
    rng = np.random.default_rng(0)
    total_reward = 0

    for _ in range(n_steps):
        c = contexts[rng.integers(2)]
        ad = rng.integers(3) if rng.random() < eps else int(np.argmax(Q[c]))
        r = env.show(c, ad)
        N[c][ad] += 1
        Q[c][ad] += (r - Q[c][ad]) / N[c][ad]
        total_reward += r

    return total_reward, Q, N

reward, Q, N = run_contextual()
print(f"کلیک کل: {int(reward)} از ۵۰۰۰ (CTR={reward/5000:.3%})")
for c in Q:
    print(f"{c}: بهترین تبلیغ = {int(np.argmax(Q[c]))} | Q={Q[c].round(3)} | N={N[c].astype(int)}")
```

**خروجی معمول:** برای «جوان» تبلیغ ۳ و برای «مسن» تبلیغ ۲ انتخاب می‌شود — همان بهینه. **این، ساده‌ترین و پرکاربردترین RL صنعتی است.**

**مثال ۲ — مقایسه با سیاست ثابت:**

```python
# سیاست ثابت: همیشه تبلیغ ۰
env = AdBandit(seed=1)
rng = np.random.default_rng(1)
fixed = sum(env.show(["جوان", "مسن"][rng.integers(2)], 0) for _ in range(5000))
print(f"سیاست ثابت: {int(fixed)} کلیک")
print(f"contextual:  {int(reward)} کلیک")
print(f"بهبود: {(reward - fixed) / max(fixed, 1):.1%}")
```

**مثال ۳ — شبیه‌سازی ساده‌ی RLHF (مفهومی):**

```python
import numpy as np

def simulate_rlhf():
    """مدل پاداش از مقایسه‌های انسانی + بهینه‌سازی سیاست."""
    rng = np.random.default_rng(0)

    # سه «پاسخ» با کیفیت واقعی ناشناخته
    true_quality = np.array([0.3, 0.8, 0.5])

    # ۱. جمع‌آوری مقایسه‌های انسانی → آموزش مدل پاداش
    n_pairs = 300
    pairs = rng.integers(0, 3, (n_pairs, 2))
    prefs = []
    for a, b in pairs:
        # انسان، با کمی نویز، کیفیت بالاتر را ترجیح می‌دهد
        p_a_wins = 1 / (1 + np.exp(-(true_quality[a] - true_quality[b]) * 5))
        prefs.append(1 if rng.random() < p_a_wins else 0)

    # ۲. مدل پاداش ساده: میانگین برد
    reward_model = np.zeros(3)
    counts = np.zeros(3)
    for (a, b), p in zip(pairs, prefs):
        reward_model[a] += p
        reward_model[b] += 1 - p
        counts[a] += 1
        counts[b] += 1
    reward_model = reward_model / np.maximum(counts, 1)

    # ۳. سیاست بهینه بر اساس مدل پاداش
    best = int(np.argmax(reward_model))
    return reward_model.round(3), best, int(np.argmax(true_quality))

rm, best, true_best = simulate_rlhf()
print("پاداش تخمینی:", rm)
print(f"بهترین انتخاب مدل: {best} | بهترین واقعی: {true_best} | درست؟ {best == true_best}")
```

**این، هسته‌ی RLHF در ۳۰ خط است.** پیچیدگی واقعی در مقیاس و PPO است، نه در ایده.

### مقایسه و دیدگاه‌های مختلف

| کاربرد | نوع RL | چالش اصلی |
|---|---|---|
| بازی | DQN/PPO | فضای جست‌وجوی عظیم |
| رباتیک | SAC/PPO | sim-to-real |
| تبلیغات | Contextual Bandit | پاداش تأخیری، تقلب |
| تنظیم LLM | PPO/DPO | هزینه‌ی ارزیابی انسانی |
| کنترل صنعتی | MPC + RL | ایمنی |

**دو دیدگاه:**
- **«RL در صنعت مرده است.»** برای مسائل پیچیده‌ی ترتیبی، تا حدی درست.
- **«Bandit همه‌جا هست.»** درست: از انتخاب تیتر تا تست A/B هوشمند، همه bandit هستند.

**جمع:** RL «کلاسیک» کم استفاده می‌شود؛ **RL «سبک» (bandit) همه‌جا هست**.

### اشتباهات رایج

**۱. استفاده از RL کامل وقتی bandit کافی است.** ۹۰٪ مسائل صنعتی، bandit هستند.
**۲. تعریف پاداش بدون فکر به reward hacking.** عامل راه میان‌بر پیدا می‌کند.
**۳. آموزش آنلاین روی کاربران واقعی.** ریسک اخلاقی و تجاری. اول آفلاین.
**۴. اعتماد به شبیه‌ساز.** sim-to-real gap واقعی است.
**۵. ارزیابی با یک معیار.** باید بازده، پایداری و ایمنی را با هم بسنجی.
**۶. نادیده گرفتن هزینه‌ی اکتشاف.** هر اشتباه، هزینه دارد.

### نکات مهم و ترفندها

- **اول bandit را امتحان کن.** اگر جواب داد، RL کامل لازم نیست.
- **پاداش را با دقت طراحی کن و به reward hacking فکر کن.** «عامل چه راه میان‌بری پیدا می‌کند؟»
- **Offline RL را برای شروع در نظر بگیر.** ایمن‌تر از آنلاین.
- **در سیستم‌های واقعی، human-in-the-loop بگذار.** RL به‌تنهایی، پرخطر است.
- **میان‌بر:** RL صنعتی = bandit + پاداش دقیق + ارزیابی محافظه‌کارانه.

### خلاصه فصل

- RL در بازی، رباتیک، سیستم‌ها و تنظیم LLM موفق بوده است.
- در تولید، چالش‌های هزینه، شبیه‌ساز و ارزیابی، مانع اصلی‌اند.
- Contextual Bandit، پرکاربردترین شکل RL صنعتی است.
- RLHF، پیوند RL و مدل‌های زبانی است.
- reward hacking، تله‌ی همیشگی طراحی پاداش است.

**سؤالات مروری:**
1. چرا RL در تولید کمتر از آزمایشگاه استفاده می‌شود؟
2. Contextual Bandit چه تفاوتی با RL کامل دارد؟
3. reward hacking را با یک مثال توضیح بده.

---

## منابع و مراجع

**کتاب‌ها (رایگان)**
- Sutton, R. & Barto, A. — *Reinforcement Learning: An Introduction* (2nd ed., incompleteideas.net — مرجع اصلی)
- Winder, P. — *Reinforcement Learning: Industrial Applications of Intelligent Agents*

**دوره‌ها**
- David Silver — *RL Course* (DeepMind/UCL، ویدئوهای یوتیوب)
- Hugging Face — *Deep RL Course* (عملی، با notebook)

**مقالات کلیدی**
- Watkins, C. & Dayan, P. (1992) — *Q-Learning*
- Mnih, V. et al. (2015) — *Human-level control through deep reinforcement learning* (DQN)
- Schulman, J. et al. (2017) — *Proximal Policy Optimization* (PPO)
- Ouyang, L. et al. (2022) — *Training language models to follow instructions with human feedback* (InstructGPT/RLHF)

**محیط‌های تمرین**
- Gymnasium (سابقاً OpenAI Gym) — `CartPole`, `FrozenLake`
- Stable-Baselines3 — پیاده‌سازی‌های آماده

---

**بعدی:** [`13-Training-Pipeline.md`](./13-Training-Pipeline.md) — فصل ۵۵: چرخه‌ی کامل آموزش
