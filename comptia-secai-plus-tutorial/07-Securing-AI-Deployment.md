# امن‌سازی استقرار AI: از معماری تا کنترل هزینه

**مدت یادگیری**: ۶ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۳ (مدل تهدید) و ۰۶ (حملات مدل)

---

## فهرست محتوا

- [فصل ۳۵: معماری استقرار — on-prem، cloud، hybrid و لایهٔ استنتاج](#ch-35)
- [فصل ۳۶: هویت و دسترسی برای AI — Least Privilege در عمل](#ch-36)
- [فصل ۳۷: کنترل شبکه و مرزها — egress، private endpoint، جداسازی](#ch-37)
- [فصل ۳۸: حفاظت از مدل، کلید و مصنوعات](#ch-38)
- [فصل ۳۹: لاگ، پایش و اتصال به SIEM](#ch-39)
- [فصل ۴۰: Rate Limit، Token Quota و کنترل هزینه](#ch-40)

---

<a name="ch-35"></a>
## فصل ۳۵: معماری استقرار — on-prem، cloud، hybrid و لایهٔ استنتاج

### مقدمه

امنیت AI، از **معماری** شروع می‌شود، نه از فیلتر. اگر معماری، دسترسی اضافی بدهد، هیچ Guardrail‌ای نجاتش نمی‌دهد.

این فصل، سه مدل استقرار و لایه‌های یک سیستم AI را از دید امنیتی می‌شکافد.

### مفاهیم اساسی

**سه مدل استقرار**

| مدل | کنترل | هزینه | مناسب |
|---|---|---|---|
| **SaaS** | کم | کم | سریع، دادهٔ عمومی |
| **PaaS/API** | متوسط | متوسط | تعادل |
| **Self-hosted** | بالا | زیاد | دادهٔ حساس |

**نکتهٔ مهم:** «ابر» و «امنیت» متضاد نیستند. **پیکربندی** تعیین‌کننده است. یک S3 باز در ابر، از یک سرور داخلی باز، امن‌تر نیست.

**پنج لایهٔ معماری**

```
لایه ۵: رابط کاربری
لایه ۴: Gateway / API
لایه ۳: Orchestration (Guardrail، RAG، ابزار)
لایه ۲: مدل (inference)
لایه ۱: داده و ذخیره‌سازی
```

| لایه | کنترل‌های کلیدی |
|---|---|
| **۵. رابط** | احراز هویت، CSRF، نرخ |
| **۴. Gateway** | کلید، Rate limit، لاگ |
| **۳. Orchestration** | Guardrail، فیلتر، سیاست |
| **۲. مدل** | دسترسی، ایزولاسیون، نسخه |
| **۱. داده** | رمزنگاری، طبقه‌بندی، مجوز |

**مفهوم «جداسازی لایهٔ استنتاج»**

لایهٔ استنتاج (Inference) باید **ایزوله** باشد:

| ایزولاسیون | چرا |
|---|---|
| **شبکه** | مدل نباید به اینترنت باز دسترسی داشته باشد |
| **پردازش** | کانتینر جدا، بدون دسترسی به میزبان |
| **حافظه** | جلوگیری از نشت بین درخواست‌ها |
| **زمان** | محدودیت اجرا (جلوگیری از DoS) |

**نکتهٔ کلیدی:** مدل، کد اجرا نمی‌کند — ولی **ابزار** ممکن است. ایزولاسیون ابزار، مهم‌تر از ایزولاسیون مدل است.

**مفهوم «معماری مرجع امن»**

```
[کاربر]
   ↓ HTTPS + MFA
[WAF / API Gateway]  ← نرخ، احراز هویت، لاگ
   ↓
[Orchestrator]       ← Guardrail ورودی
   ↓
[RAG Service]        ← فیلتر دسترسی، بازبینی
   ↓
[Model Endpoint]     ← private network، بدون egress
   ↓
[Guardrail خروجی]    ← PII، سیاست
   ↓
[کاربر]
```

**هر پیکان، یک مرز اعتماد است.**

### مثال‌های عملی

**Lab Scenario 35-A: ارزیابی معماری**

```markdown
# ارزیابی معماری — چت‌بات پشتیبانی

| لایه | وضعیت فعلی | ریسک | توصیه |
|---|---|---|---|
| رابط | بدون MFA | متوسط | MFA برای ادمین |
| Gateway | کلید ثابت مشترک | بالا | کلید per-client |
| Orchestrator | Guardrail ورودی | متوسط | + خروجی |
| RAG | بدون فیلتر دسترسی | **بالا** | فیلتر نقش |
| مدل | دسترسی اینترنت باز | **بالا** | قطع egress |
| داده | رمزنگاری در سکون | کم | — |
```

**Lab Scenario 35-B: تصمیم استقرار**

| معیار | SaaS | PaaS | Self-hosted |
|---|---|---|---|
| دادهٔ PII | ⚠️ | ⚠️ | ✅ |
| زمان راه‌اندازی | روز | هفته | ماه |
| کنترل نسخه | کم | متوسط | کامل |
| هزینهٔ اولیه | کم | متوسط | زیاد |
| نیاز به تیم | کم | متوسط | زیاد |

**قاعدهٔ تصمیم:**

```text
اگر دادهٔ سطح «بسیار حساس» + الزام اقامت داده → Self-hosted
اگر دادهٔ «محرمانه» + تیم کوچک → PaaS با قرارداد
اگر دادهٔ «عمومی/داخلی» → SaaS سازمانی
```

**Lab Scenario 35-C: ایزولاسیون استنتاج**

```yaml
# نمونهٔ پیکربندی کانتینر استنتاج (سخت‌سازی)
apiVersion: v1
kind: Pod
metadata:
  name: inference
spec:
  containers:
    - name: model
      image: registry.internal/model:2.3.0
      securityContext:
        runAsNonRoot: true
        readOnlyRootFilesystem: true
        allowPrivilegeEscalation: false
        capabilities:
          drop: ["ALL"]
      resources:
        limits:
          cpu: "4"
          memory: "16Gi"
      volumeMounts:
        - name: model-cache
          mountPath: /models
          readOnly: true
  volumes:
    - name: model-cache
      emptyDir: {}
```

**نکته:** `readOnlyRootFilesystem` و `drop: ALL` دو کنترل ارزان با اثر بالا هستند.

### مقایسه و الگوهای مختلف

**مقایسهٔ سه مدل از دید امنیت**

| بُعد | SaaS | PaaS | Self-hosted |
|---|---|---|---|
| سطح حمله | کم (وابسته به فروشنده) | متوسط | بالا |
| کنترل | کم | متوسط | کامل |
| مسئولیت | مشترک | مشترک | کامل |
| مناسب | سریع | تعادل | حساس |

**سه الگوی معماری**

| الگو | مزیت | عیب |
|---|---|---|
| **Monolith** | ساده | سطح حملهٔ بزرگ |
| **Microservice** | ایزولاسیون | پیچیدگی |
| **Serverless** | مقیاس | کنترل کمتر |

### اشکال شایع

**اشتباه ۱: «ابر امن نیست»**
پیکربندی، نه محل.

**اشتباه ۲: مدل با دسترسی اینترنت**
درِ خروج داده (exfiltration) باز است.

**اشتباه ۳: نادیده گرفتن لایهٔ orchestration**
Guardrail درست، مهم‌تر از مدل قوی.

**اشتباه ۴: کانتینر با دسترسی root**
یک آسیب‌پذیری، کل میزبان را می‌دهد.

**اشتباه ۵: نبود مرز اعتماد**
اگر همه «داخلی» است، هیچ کنترلی نیست.

### نکات مهم (Tips & Tricks)

- **مدل را بدون egress اجرا کن.**
- **کانتینر: non-root، read-only، بدون capability.**
- **هر پیکان در نمودار، یک کنترل داشته باشد.**
- **برای دادهٔ حساس، self-hosted یا قرارداد سخت.**
- **معماری را قبل از Guardrail طراحی کن.**

### خلاصه فصل

- سه مدل: **SaaS، PaaS، Self-hosted** — با مسئولیت متفاوت.
- پنج لایه: **رابط، Gateway، Orchestration، مدل، داده**.
- **ایزولاسیون استنتاج** (شبکه، پردازش، حافظه، زمان).
- پیکربندی، نه محل، امنیت را تعیین می‌کند.
- Guardrail جای معماری را نمی‌گیرد.

---

<a name="ch-36"></a>
## فصل ۳۶: هویت و دسترسی برای AI — Least Privilege در عمل

### مقدمه

در سیستم‌های AI، هویت‌ها بیشتر از انسان‌ها هستند: سرویس مدل، سرویس RAG، Agent، پایپ‌لاین آموزش. هر کدام یک هویت است و هر هویت، یک ریسک.

### مفاهیم اساسی

**پنج نوع هویت**

| هویت | مثال | ریسک |
|---|---|---|
| **کاربر انسانی** | تحلیلگر، مشتری | سرقت اعتبار |
| **حساب سرویس** | inference-svc | دسترسی بیش‌ازحد |
| **کلید API** | کلید مشتری | نشت |
| **Agent** | دستیار با ابزار | اقدام ناخواسته |
| **پایپ‌لاین** | CI/CD | نفوذ زنجیره |

**چهار اصل**

| اصل | معنا |
|---|---|
| **حداقل دسترسی** | فقط آنچه لازم است |
| **تفکیک وظایف** | ساخت ≠ استقرار ≠ تأیید |
| **کوتاه‌مدت بودن** | توکن موقت، نه دائمی |
| **قابل ابطال** | چرخش و لغو سریع |

**مفهوم «هویت Agent»**

Agent، هویت مستقل می‌خواهد — نه هویت کاربر:

| مدل | مزیت | عیب |
|---|---|---|
| **Act-as-user** | ساده، مجوز کاربر | دسترسی کامل کاربر |
| **Dedicated identity** ⭐ | حداقل دسترسی | پیچیدگی |
| **On-behalf-of** | تعادل | نیاز به token exchange |

**توصیه:** Agent، هویت اختصاصی با **زیرمجموعهٔ** مجوزهای کاربر داشته باشد، نه کل آن.

**مفهوم «کلید API به‌عنوان رمز»**

| اشتباه | درست |
|---|---|
| کلید در کد | Secret Manager |
| کلید مشترک تیمی | کلید per-client |
| بدون انقضا | چرخش دوره‌ای |
| بدون لاگ | لاگ استفاده |

**مفهوم «دسترسی به داده در RAG»**

مهم‌ترین کنترل دسترسی در سیستم‌های AI:

```
مجوز سند ← از سیستم منبع
مجوز کاربر ← از SSO
فیلتر ← در لایهٔ بازیابی
```

**اشتباه رایج:** فیلتر در رابط کاربری. مدل، محدودیت UI را نمی‌شناسد.

### مثال‌های عملی

**Lab Scenario 36-A: ماتریس دسترسی**

```markdown
# ماتریس دسترسی — سرویس‌های AI

| هویت | مدل | RAG | دادهٔ خام | ابزار نوشتنی | لاگ |
|---|---|---|---|---|---|
| inference-svc | ✅ invoke | ✅ read | ❌ | ❌ | ✅ write |
| rag-svc | ❌ | ✅ read/write | ❌ | ❌ | ✅ write |
| agent-svc | ✅ invoke | ✅ read | ❌ | ⚠️ با تأیید | ✅ write |
| training-job | ✅ train | ❌ | ✅ read | ❌ | ✅ write |
| analyst (human) | ❌ | ✅ read | ⚠️ | ❌ | ✅ read |
```

**درس:** هر هویت، حداقل. `inference-svc` نباید به دادهٔ خام دسترسی داشته باشد.

**Lab Scenario 36-B: سیاست حداقل دسترسی (نمونه)**

```yaml
# policy.yaml — نمونهٔ مفهومی
roles:
  inference_service:
    allow:
      - "model:invoke"
      - "log:write"
    deny:
      - "dataset:*"
      - "model:update"
      - "tool:*"

  rag_service:
    allow:
      - "vector:read"
      - "vector:write"
      - "doc:read"
    conditions:
      - "doc.classification in user.clearance"

  agent_service:
    allow:
      - "model:invoke"
      - "vector:read"
      - "tool:read:*"
    require_approval:
      - "tool:write:*"
      - "payment:*"
```

**Lab Scenario 36-C: چرخش کلید**

```python
"""
مدیریت چرخهٔ عمر کلید — نمونهٔ ساختاری
"""
from dataclasses import dataclass
from datetime import date, timedelta

@dataclass
class ApiKey:
    key_id: str
    owner: str
    created: date
    ttl_days: int = 90

    @property
    def expires(self) -> date:
        return self.created + timedelta(days=self.ttl_days)

    def status(self, today: date | None = None) -> str:
        today = today or date.today()
        left = (self.expires - today).days
        if left < 0:
            return "EXPIRED"
        if left <= 14:
            return f"EXPIRING ({left}d)"
        return f"OK ({left}d)"

k = ApiKey("k-1042", "support-bot", date(2026, 9, 1))
print(k.key_id, k.expires, k.status(date(2026, 11, 12)))
```

**درس:** کلید بدون انقضا، بدهی امنیتی است. TTL + هشدار ۱۴ روزه، حداقل.

### مقایسه و الگوهای مختلف

**مقایسهٔ سه مدل هویت Agent**

| | Act-as-user | Dedicated | On-behalf-of |
|---|---|---|---|
| حداقل دسترسی | ❌ | ✅ | ✅ |
| پیچیدگی | کم | متوسط | بالا |
| مناسب | MVP | تولید | سازمانی |

**مقایسهٔ دو نوع اعتبار**

| | کلید ثابت | توکن موقت |
|---|---|---|
| نشت | دائمی | محدود |
| مدیریت | ساده | پیچیده |
| توصیه | کم‌ریسک | حساس |

### اشکال شایع

**اشتباه ۱: حساب سرویس با دسترسی ادمین**
یک آسیب‌پذیری = کل سیستم.

**اشتباه ۲: کلید در متغیر محیطی کانتینر**
در لاگ و crash dump ظاهر می‌شود.

**اشتباه ۳: Agent با هویت کاربر**
دسترسی کامل کاربر، شامل اقدامات حساس.

**اشتباه ۴: فیلتر دسترسی در UI**
مدل، آن را نمی‌بیند.

**اشتباه ۵: کلید بدون لاگ**
نمی‌فهمی چه کسی استفاده کرد.

### نکات مهم (Tips & Tricks)

- **ماتریس دسترسی را روی یک صفحه بکش.**
- **هر سرویس، هویت جدا.**
- **TTL برای هر کلید** + هشدار.
- **Agent، هویت اختصاصی.**
- **فیلتر دسترسی در لایهٔ بازیابی.**

### خلاصه فصل

- پنج نوع هویت؛ هر کدام ریسک خودش.
- چهار اصل: **حداقل، تفکیک، کوتاه‌مدت، قابل ابطال**.
- Agent، هویت اختصاصی می‌خواهد.
- کلید = رمز؛ Secret Manager + TTL.
- فیلتر دسترسی، در بازیابی نه UI.

---

<a name="ch-37"></a>
## فصل ۳۷: کنترل شبکه و مرزها — egress، private endpoint، جداسازی

### مقدمه

در حمله‌های AI، **خروج داده** هدف نهایی است. اگر مدل نتواند به بیرون وصل شود، حتی تزریق موفق هم نمی‌تواند داده را بفرستد.

این فصل، ارزان‌ترین و مؤثرترین کنترل‌ها را می‌دهد.

### مفاهیم اساسی

**چهار کنترل شبکه**

| کنترل | اثر |
|---|---|
| **Egress filtering** | جلوگیری از خروج داده |
| **Private endpoint** | حذف مسیر اینترنت |
| **Segmentation** | محدودسازی حرکت جانبی |
| **DNS allowlist** | جلوگیری از C2 |

**Egress — مهم‌ترین کنترل**

| سناریو | بدون egress | با egress |
|---|---|---|
| Injection موفق | داده ارسال می‌شود | مسدود |
| مدل آلوده | تماس با C2 | مسدود |
| ابزار مخرب | دانلود payload | مسدود |

**قاعده:** مدل استنتاج، **نباید** به اینترنت دسترسی داشته باشد. اگر ابزار جست‌وجو لازم است، از یک **پروکسی واسط** با allowlist استفاده کن.

**مفهوم «private endpoint»**

| مدل | مسیر | ریسک |
|---|---|---|
| Public endpoint | اینترنت | بالا |
| Private endpoint | شبکهٔ خصوصی | کم |
| VPC peering | شبکهٔ خصوصی | متوسط |

**مفهوم «جداسازی محیط»**

| محیط | داده | دسترسی |
|---|---|---|
| **توسعه** | ساختگی | آزادتر |
| **آزمون** | ساختگی | محدود |
| **تولید** | واقعی | حداقل |

**خطای رایج:** یک حساب/شبکه برای همه. یک نفوذ در توسعه، تولید را می‌دهد.

**مفهوم «DNS به‌عنوان کنترل»**

مهاجم برای exfiltration می‌تواند از DNS استفاده کند (درخواست به دامنهٔ حاوی داده).

| کنترل | اثر |
|---|---|
| DNS allowlist | بالا |
| ثبت همهٔ queryها | تشخیص |
| مسدودسازی دامنهٔ ناشناس | بالا |

### مثال‌های عملی

**Lab Scenario 37-A: سیاست شبکه**

```yaml
# network-policy.yaml — نمونهٔ مفهومی
egress:
  default: deny
  rules:
    - to: "vector-db.internal"
      port: 443
      protocol: tcp
    - to: "log-collector.internal"
      port: 514
      protocol: tcp
    - to: "proxy.internal"
      port: 8080
      protocol: tcp
      note: "تنها مسیر مجاز به اینترنت، با allowlist"

ingress:
  default: deny
  rules:
    - from: "api-gateway"
      port: 8443
```

**نکته:** `default: deny` — اصل بنیادی. هر استثنا، مستند.

**Lab Scenario 37-B: پروکسی با allowlist**

```python
"""
بررسی allowlist دامنه — نمونهٔ منطق
"""
from urllib.parse import urlparse

ALLOWED_DOMAINS = {
    "docs.internal.example",
    "api.partner.example",
}

def allowed(url: str) -> bool:
    try:
        host = urlparse(url).hostname or ""
    except ValueError:
        return False
    # تطبیق دقیق، نه endswith (جلوگیری از evil-docs.internal.example.attacker.com)
    return host in ALLOWED_DOMAINS

tests = [
    "https://docs.internal.example/page",
    "https://api.partner.example/v1",
    "https://evil.example/collect",
    "https://docs.internal.example.attacker.com/x",
]
for t in tests:
    print(f"{allowed(t)!s:5s} | {t}")
```

**درس:** `endswith` یا `startswith`، دامنهٔ جعلی را عبور می‌دهد. تطبیق **دقیق** لازم است.

**Lab Scenario 37-C: جداسازی شبکه**

```
┌──────────────┐     ┌──────────────┐
│  DMZ         │     │  App         │
│  WAF/Gateway │────▶│  Orchestrator│
└──────────────┘     └──────┬───────┘
                            │
                     ┌──────▼───────┐
                     │  AI Subnet   │
                     │  model + rag │  ← egress: deny
                     └──────┬───────┘
                            │
                     ┌──────▼───────┐
                     │  Data Subnet │
                     │  vector/db   │
                     └──────────────┘
```

**درس:** AI Subnet، فقط با App و Data صحبت می‌کند. نه با اینترنت.

### مقایسه و الگوهای مختلف

**سه سطح کنترل شبکه**

| سطح | کنترل | اثر |
|---|---|---|
| **پایه** | فایروال | متوسط |
| **متوسط** | + private endpoint | بالا |
| **پیشرفته** | + egress deny + DNS allowlist | بالاترین |

**مقایسهٔ دو رویکرد**

| | blacklist | allowlist |
|---|---|---|
| امنیت | کم | بالا |
| نگهداری | کم | بیشتر |
| توصیه | ❌ | ✅ |

### اشکال شایع

**اشتباه ۱: مدل با اینترنت باز**
در exfiltration باز است.

**اشتباه ۲: allowlist با `endswith`**
دامنهٔ جعلی عبور می‌کند.

**اشتباه ۳: یک شبکه برای همهٔ محیط‌ها**
توسعه → تولید.

**اشتباه ۴: نادیده گرفتن DNS**
مسیر exfiltration پنهان.

**اشتباه ۵: استثنای بدون مستند**
شش ماه بعد، کسی نمی‌داند چرا.

### نکات مهم (Tips & Tricks)

- **`default: deny` برای egress.**
- **تطبیق دقیق دامنه، نه substring.**
- **محیط‌ها را جدا کن.**
- **DNS را لاگ کن.**
- **هر استثنا، تاریخ و مالک.**

### خلاصه فصل

- چهار کنترل: **egress، private endpoint، segmentation، DNS**.
- **Egress deny** مؤثرترین کنترل ضد exfiltration.
- allowlist با تطبیق دقیق، نه `endswith`.
- جداسازی محیط، الزامی.
- DNS، مسیر پنهان exfiltration.

---

<a name="ch-38"></a>
## فصل ۳۸: حفاظت از مدل، کلید و مصنوعات

### مقدمه

مدل، یک **مصنوع** است: فایل، قابل کپی، قابل تعویض. مثل یک باینری تولیدی با آن رفتار کن.

### مفاهیم اساسی

**چهار مصنوع**

| مصنوع | ریسک | کنترل |
|---|---|---|
| **وزن مدل** | سرقت، تعویض | رمزنگاری، امضا |
| **کلید API** | سوءاستفاده | Secret Manager |
| **پیکربندی** | تغییر رفتار | Git، بازبینی |
| **دادهٔ مرجع** | نشت، آلودگی | طبقه‌بندی |

**مفهوم «مخزن مصنوع» (Artifact Registry)**

| ویژگی | چرا |
|---|---|
| **احراز هویت** | فقط سرویس مجاز |
| **نسخه‌بندی** | بازگشت‌پذیری |
| **immutable** | جلوگیری از overwrite |
| **امضا** | اصالت |
| **لاگ دسترسی** | ردیابی |

**مفهوم «امضای مدل»**

| روش | اثر |
|---|---|
| **Hash + ثبت جدا** | تشخیص تغییر |
| **امضای دیجیتال (Sigstore و مشابه)** | اصالت + یکپارچگی |
| **SBOM/ML-BOM** | فهرست اجزا |

**نکته:** hash بدون محل امن، بی‌فایده است. مهاجم هر دو را عوض می‌کند.

**مفهوم «Secret Manager»**

| اشتباه | درست |
|---|---|
| کلید در `.env` | Secret Manager |
| کلید در کد | ارجاع به secret |
| کلید در لاگ | ماسک کردن |
| کلید مشترک | per-service |

**الگوی «کوتاه‌مدت»:**

```
سرویس → درخواست اعتبار → Secret Manager → توکن موقت (۱۵ دقیقه)
```

### مثال‌های عملی

**Lab Scenario 38-A: ثبت و تأیید مصنوع**

```python
"""
ثبت manifest برای مصنوعات مدل
"""
import hashlib, json, os
from datetime import datetime, timezone

def sha256(path: str) -> str:
    h = hashlib.sha256()
    with open(path, "rb") as f:
        for chunk in iter(lambda: f.read(8192), b""):
            h.update(chunk)
    return h.hexdigest()

def manifest(model_dir: str) -> dict:
    files = {}
    for name in sorted(os.listdir(model_dir)):
        p = os.path.join(model_dir, name)
        if os.path.isfile(p):
            files[name] = {"sha256": sha256(p), "size": os.path.getsize(p)}
    return {
        "created": datetime.now(timezone.utc).isoformat(),
        "files": files,
        "count": len(files),
    }

# نمونه (نیاز به پوشهٔ واقعی)
# m = manifest("./model")
# print(json.dumps(m, indent=2))
print("تابع آماده است — روی پوشهٔ مدل اجرا کن.")
```

**Lab Scenario 38-B: ماسک کردن راز در لاگ**

```python
"""
حذف راز از لاگ — قبل از نوشتن
"""
import re

PATTERNS = [
    (r"(sk-[A-Za-z0-9]{8})[A-Za-z0-9]+", r"\1***"),
    (r"(api[_-]?key\s*[:=]\s*)\S+", r"\1***"),
    (r"(Bearer\s+)[A-Za-z0-9._-]+", r"\1***"),
    (r"(password\s*[:=]\s*)\S+", r"\1***"),
]

def scrub(line: str) -> str:
    for pat, repl in PATTERNS:
        line = re.sub(pat, repl, line, flags=re.I)
    return line

samples = [
    "key=sk-abcdefgh12345678",
    "Authorization: Bearer eyJhbGciOi.xyz",
    "api_key: 9f8e7d6c",
    "user logged in",
]
for s in samples:
    print(scrub(s))
```

**درس:** لاگ، شایع‌ترین مسیر نشت کلید است. ماسک کردن، ارزان و ضروری.

**Lab Scenario 38-C: کنترل دسترسی مخزن**

| نقش | خواندن | نوشتن | حذف |
|---|---|---|---|
| training-job | ✅ | ✅ | ❌ |
| inference-svc | ✅ | ❌ | ❌ |
| engineer | ✅ | ⚠️ با تأیید | ❌ |
| admin | ✅ | ✅ | ⚠️ با تأیید |

**نکته:** حذف مدل، اقدام برگشت‌ناپذیر است. تأیید دو نفره.

### مقایسه و الگوهای مختلف

**سه سطح حفاظت مصنوع**

| سطح | کنترل | اثر |
|---|---|---|
| **پایه** | رمزنگاری در سکون | متوسط |
| **متوسط** | + hash + مخزن خصوصی | بالا |
| **پیشرفته** | + امضا + immutable + لاگ | بالاترین |

**مقایسهٔ دو نوع اعتبار**

| | کلید ثابت | توکن موقت |
|---|---|---|
| عمر | طولانی | کوتاه |
| نشت | خطر بالا | خطر کم |
| پیچیدگی | کم | متوسط |

### اشکال شایع

**اشتباه ۱: کلید در `.env` مخزن**
در تاریخچهٔ Git می‌ماند.

**اشتباه ۲: مخزن قابل overwrite**
مهاجم مدل را عوض می‌کند.

**اشتباه ۳: لاگ بدون ماسک**
کلید در SIEM.

**اشتباه ۴: hash بدون محل امن**
بی‌فایده.

**اشتباه ۵: حذف مدل بدون تأیید**
برگشت‌ناپذیر.

### نکات مهم (Tips & Tricks)

- **Secret Manager از روز اول.**
- **مخزن immutable + امضا.**
- **لاگ را ماسک کن، قبل از نوشتن.**
- **حذف مدل، تأیید دو نفره.**
- **manifest را خودکار تولید کن.**

### خلاصه فصل

- چهار مصنوع: **وزن، کلید، پیکربندی، داده**.
- مخزن: **احراز هویت، نسخه، immutable، امضا، لاگ**.
- hash بدون محل امن، بی‌فایده است.
- Secret Manager + توکن موقت.
- لاگ بدون ماسک، نشت است.

---

<a name="ch-39"></a>
## فصل ۳۹: لاگ، پایش و اتصال به SIEM

### مقدمه

بدون لاگ، نه تشخیص داری، نه پاسخ‌گویی، نه انطباق. در سیستم‌های AI، لاگ باید **سه چیز** را ثبت کند: ورودی، خروجی، و تصمیم.

### مفاهیم اساسی

**چهار سطح لاگ**

| سطح | چه ثبت می‌شود | حجم |
|---|---|---|
| **دسترسی** | کی، کِی، از کجا | کم |
| **تعامل** | پرامپت، پاسخ | **زیاد** |
| **تصمیم** | Guardrail، مسیریابی | متوسط |
| **عملیات** | مدل، نسخه، خطا | کم |

**مفهوم «لاگ تعامل، پرخطر است»**

| ریسک | توضیح |
|---|---|
| **PII** | پرامپت کاربر حاوی دادهٔ شخصی |
| **راز** | کلید، رمز |
| **حجم** | هزینهٔ ذخیره |
| **دسترسی** | لاگ، هدف مهاجم |

**راه‌حل:** لاگ **فراداده** به‌جای محتوا، یا محتوای ماسک‌شده با نگهداری کوتاه.

**طرح پیشنهادی**

| فیلد | مثال | نگهداری |
|---|---|---|
| `trace_id` | uuid | ۹۰ روز |
| `user_id` | hash | ۹۰ روز |
| `model` | assistant-v3 | ۱ سال |
| `prompt_hash` | sha256 | ۹۰ روز |
| `prompt_text` | (ماسک‌شده) | **۷ روز** |
| `guardrail_action` | block/allow | ۱ سال |
| `latency_ms` | ۸۴۰ | ۳۰ روز |
| `tokens` | ۱۲۰۰ | ۳۰ روز |

**مفهوم «اتصال به SIEM»**

| منبع | چه می‌فرستد |
|---|---|
| API Gateway | دسترسی، نرخ |
| Guardrail | مسدودسازی |
| Model endpoint | خطا، تأخیر |
| RAG | بازیابی، دسترسی |
| Secret Manager | استفاده از کلید |

**هشدارهای کلیدی**

| هشدار | آستانه |
|---|---|
| نرخ مسدودسازی | > ۳ برابر میانگین |
| حجم توکن | > ۲ برابر روزانه |
| خطای مدل | > ۵٪ |
| دسترسی به سند محرمانه | هر مورد |
| Query تکراری | > ۱۰۰ از یک کاربر |

### مثال‌های عملی

**Lab Scenario 39-A: ساختار لاگ**

```python
"""
ساخت رکورد لاگ با ماسک و hash
"""
import hashlib, json, uuid
from datetime import datetime, timezone

def h(text: str) -> str:
    return hashlib.sha256(text.encode()).hexdigest()[:16]

def mask(text: str) -> str:
    import re
    text = re.sub(r"[\w\.-]+@[\w\.-]+\.\w+", "[EMAIL]", text)
    text = re.sub(r"(?<!\d)\d{10,16}(?!\d)", "[NUM]", text)
    return text[:500]

def log_record(user: str, prompt: str, model: str, action: str, tokens: int) -> dict:
    return {
        "trace_id": str(uuid.uuid4()),
        "ts": datetime.now(timezone.utc).isoformat(),
        "user_hash": h(user),
        "model": model,
        "prompt_hash": h(prompt),
        "prompt_masked": mask(prompt),
        "guardrail_action": action,
        "tokens": tokens,
    }

print(json.dumps(
    log_record("u-1042", "ایمیل من ali@example.com است، کمک کن", "assistant-v3", "allow", 420),
    ensure_ascii=False, indent=2
))
```

**Lab Scenario 39-B: هشدار نرخ مسدودسازی**

```python
"""
تشخیص انحراف نرخ مسدودسازی — ساده
"""
from statistics import mean, pstdev

baseline = [0.02, 0.03, 0.02, 0.04, 0.03, 0.02, 0.03]
current = 0.15

mu, sd = mean(baseline), pstdev(baseline)
z = (current - mu) / sd if sd else 0

print(f"میانگین={mu:.3f} انحراف={sd:.3f} z={z:.1f}")
print("هشدار" if z > 3 else "عادی")
```

**درس:** z-score ساده، برای شروع کافی است. برای تولید، از مدل‌های anomaly detection استفاده کن.

**Lab Scenario 39-C: نگهداری و حذف**

```yaml
retention:
  access_log: 365d
  interaction_meta: 90d
  interaction_text: 7d          # ماسک‌شده
  guardrail_decisions: 365d
  model_metrics: 30d

deletion:
  on_user_request: 30d
  on_contract_end: 30d
  verification: "گواهی حذف"
```

### مقایسه و الگوهای مختلف

**مقایسهٔ دو رویکرد لاگ**

| | محتوا کامل | فراداده + ماسک |
|---|---|---|
| تشخیص | بالا | متوسط |
| حریم خصوصی | کم | بالا |
| هزینه | بالا | کم |
| توصیه | محیط تست | تولید |

**سه سطح پایش**

| سطح | چه می‌سنجد |
|---|---|
| **فنی** | تأخیر، خطا، توکن |
| **امنیتی** | مسدودسازی، دسترسی |
| **کیفی** | رضایت، دقت |

### اشکال شایع

**اشتباه ۱: لاگ کامل پرامپت برای همیشه**
نشت + هزینه.

**اشتباه ۲: نبود trace_id**
نمی‌توانی رخداد را بازسازی کنی.

**اشتباه ۳: لاگ بدون ماسک**
کلید و PII در SIEM.

**اشتباه ۴: هشدار بدون baseline**
نمی‌دانی «عادی» چیست.

**اشتباه ۵: نبود سیاست حذف**
دادهٔ قدیمی، ریسک انباشته.

### نکات مهم (Tips & Tricks)

- **`trace_id` در همه‌چیز.**
- **فراداده + ماسک، پیش‌فرض.**
- **متن کامل، حداکثر ۷ روز.**
- **baseline بگیر، بعد هشدار بگذار.**
- **سیاست حذف مکتوب.**

### خلاصه فصل

- چهار سطح: **دسترسی، تعامل، تصمیم، عملیات**.
- لاگ تعامل، پرخطر است — ماسک و نگهداری کوتاه.
- SIEM، پنج منبع دارد.
- هشدار بدون baseline، بی‌فایده.
- `trace_id`، ستون بازسازی.

---

<a name="ch-40"></a>
## فصل ۴۰: Rate Limit، Token Quota و کنترل هزینه

### مقدمه

**مصرف بی‌حد** (LLM10) یکی از ده تهدید OWASP است. دلایلش سه‌گانه است: هزینه، دسترس‌پذیری، و **پیش‌نیاز حملات دیگر** (Extraction به Query زیاد نیاز دارد).

این فصل، ارزان‌ترین کنترل‌ها را می‌دهد.

### مفاهیم اساسی

**چهار کنترل**

| کنترل | هدف |
|---|---|
| **Rate limit** | تعداد درخواست در زمان |
| **Token quota** | مجموع توکن |
| **Max tokens** | طول هر پاسخ |
| **Budget alert** | هشدار هزینه |

**سه سطح Rate limit**

| سطح | مثال | مناسب |
|---|---|---|
| **به‌ازای کاربر** | ۶۰/دقیقه | پایه |
| **به‌ازای IP** | ۲۰۰/دقیقه | ضد ربات |
| **به‌ازای سازمان** | ۱۰٬۰۰۰/روز | قرارداد |

**نکته:** Rate limit **سراسری** کافی نیست. یک کاربر می‌تواند سهمیهٔ همه را مصرف کند.

**مفهوم «الگوریتم‌های Rate limit»**

| الگوریتم | مزیت | عیب |
|---|---|---|
| **Fixed window** | ساده | انفجار در مرز |
| **Sliding window** | دقیق‌تر | حافظه |
| **Token bucket** | اجازهٔ burst | تنظیم |
| **Leaky bucket** | نرخ ثابت | تأخیر |

**مفهوم «سهمیهٔ توکن»**

```
هزینه ≈ (توکن ورودی × نرخ ورودی) + (توکن خروجی × نرخ خروجی)
```

| کنترل | اثر |
|---|---|
| `max_tokens` | سقف هر پاسخ |
| سهمیهٔ روزانه | سقف کاربر |
| سقف سازمانی | سقف کل |

**مفهوم «حملهٔ اقتصادی»**

| روش | اثر |
|---|---|
| پرامپت غول‌آسا | هزینهٔ بالا |
| درخواست انبوه | اشباع |
| خروجی طولانی | هزینهٔ خروجی |
| حلقهٔ Agent | هزینهٔ تصاعدی |

### مثال‌های عملی

**Lab Scenario 40-A: Token Bucket**

```python
"""
Token bucket — پیاده‌سازی ساده و قابل اجرا
"""
import time
from dataclasses import dataclass, field

@dataclass
class TokenBucket:
    capacity: int
    refill_per_sec: float
    tokens: float = field(init=False)
    last: float = field(default_factory=time.monotonic, init=False)

    def __post_init__(self):
        self.tokens = float(self.capacity)

    def allow(self, cost: int = 1) -> bool:
        now = time.monotonic()
        elapsed = now - self.last
        self.tokens = min(self.capacity, self.tokens + elapsed * self.refill_per_sec)
        self.last = now
        if self.tokens >= cost:
            self.tokens -= cost
            return True
        return False

bucket = TokenBucket(capacity=5, refill_per_sec=1)
results = [bucket.allow() for _ in range(8)]
print("درخواست‌ها:", results)
print("پذیرفته:", sum(results), "از", len(results))
```

**تفسیر:** ۵ درخواست فوری مجاز، بعد محدود تا پر شدن دوباره. `burst` کنترل‌شده.

**Lab Scenario 40-B: محاسبهٔ هزینه**

```python
"""
برآورد هزینهٔ ماهانه
"""
def monthly_cost(
    requests_per_day: int,
    avg_input_tokens: int,
    avg_output_tokens: int,
    price_in_per_1k: float,
    price_out_per_1k: float,
    days: int = 30,
) -> dict:
    in_cost = requests_per_day * avg_input_tokens / 1000 * price_in_per_1k * days
    out_cost = requests_per_day * avg_output_tokens / 1000 * price_out_per_1k * days
    return {
        "input_cost": round(in_cost, 2),
        "output_cost": round(out_cost, 2),
        "total": round(in_cost + out_cost, 2),
    }

print(monthly_cost(2000, 800, 300, 0.003, 0.015))
print(monthly_cost(2000, 800, 3000, 0.003, 0.015))   # خروجی طولانی‌تر
```

**درس:** خروجی، اغلب گران‌تر از ورودی است. `max_tokens` کنترل مستقیم هزینه است.

**Lab Scenario 40-C: سیاست سهمیه**

```yaml
quota_policy:
  per_user:
    requests_per_minute: 30
    requests_per_day: 500
    tokens_per_day: 200_000
    max_output_tokens: 1500
  per_tenant:
    requests_per_day: 20_000
    tokens_per_day: 8_000_000
  alerts:
    - at: 70%
      action: notify_owner
    - at: 90%
      action: notify_security
    - at: 100%
      action: throttle
```

**Lab Scenario 40-D: تشخیص الگوی استخراج**

```python
"""
تشخیص Query انبوه و تکراری — نشانهٔ Extraction
"""
from collections import Counter
import hashlib

def signature(text: str) -> str:
    return hashlib.sha256(text.strip().lower().encode()).hexdigest()[:12]

queries = [
    "قیمت محصول الف", "قیمت محصول ب", "قیمت محصول ج",
    "قیمت محصول الف", "قیمت محصول الف", "قیمت محصول د",
]

c = Counter(signature(q) for q in queries)
top = c.most_common(1)[0]

print("تکرار بالا:", top[1], "بار")
print("هشدار" if top[1] >= 3 else "عادی")
```

**نکته:** مهاجم حرفه‌ای، Queryها را متنوع می‌کند. برای تشخیص، به **توزیع** نگاه کن، نه تکرار ساده.

### مقایسه و الگوهای مختلف

**مقایسهٔ سه الگوریتم**

| الگوریتم | burst | دقت | پیچیدگی |
|---|---|---|---|
| Fixed window | بالا | کم | کم |
| Sliding window | کم | بالا | متوسط |
| Token bucket | کنترل‌شده | بالا | متوسط |

**مقایسهٔ دو رویکرد**

| | سخت‌گیر | متعادل |
|---|---|---|
| تجربه | بد | خوب |
| امنیت | بالا | متوسط |
| توصیه | پرریسک | عمومی |

### اشکال شایع

**اشتباه ۱: Rate limit سراسری**
یک کاربر، سهمیهٔ همه.

**اشتباه ۲: نبود سهمیهٔ توکن**
تعداد درخواست کم، ولی هر کدام ۱۰۰k توکن.

**اشتباه ۳: `max_tokens` بزرگ**
هزینه و نشت.

**اشتباه ۴: هشدار بدون اقدام**
گزارش بدون throttle، بی‌فایده.

**اشتباه ۵: نادیده گرفتن Agent**
حلقهٔ Agent، هزینهٔ تصاعدی.

### نکات مهم (Tips & Tricks)

- **Rate limit به‌ازای کاربر، نه سراسری.**
- **سهمیهٔ توکن جدا از تعداد درخواست.**
- **`max_tokens` را کم بگذار.**
- **هشدار + اقدام (throttle).**
- **Agent: سقف مرحله و هزینه.**

### خلاصه فصل

- چهار کنترل: **Rate، Quota، Max tokens، Budget alert**.
- سه سطح Rate limit؛ سراسری کافی نیست.
- **خروجی**، محرک اصلی هزینه.
- سهمیهٔ توکن، جدا از تعداد.
- Agent، سقف مرحله و هزینه می‌خواهد.

---

## منابع و مراجع

- [OWASP — LLM10: Unbounded Consumption](https://genai.owasp.org/) — تهدید مصرف
- [NIST SP 800-207 — Zero Trust Architecture](https://csrc.nist.gov/pubs/sp/800/207/final) — معماری
- [CIS Benchmarks — Kubernetes](https://www.cisecurity.org/benchmark/kubernetes) — سخت‌سازی
- [NIST SP 800-92 — Log Management](https://csrc.nist.gov/pubs/sp/800/92/final) — لاگ
- [Sigstore](https://www.sigstore.dev/) — امضای مصنوعات
- [Google — SAIF](https://saif.google/) — چارچوب امنیت AI
- [MITRE ATLAS — Exfiltration](https://atlas.mitre.org/) — تاکتیک

> **راستی‌آزمایی:** نمونهٔ YAML پیکربندی، **مفهومی** است و باید با پلتفرم واقعی (Kubernetes، ابر خاص) تطبیق داده شود؛ نام فیلدها بین نسخه‌ها و ارائه‌دهندگان متفاوت است. قیمت‌های توکن در کد نمونه، **فرضی** هستند و باید با تعرفهٔ جاری فروشنده جایگزین شوند. آستانه‌های هشدار، تجربی‌اند و باید با baseline سازمان تنظیم شوند.

**بعدی:** [فایل ۰۸ — امنیت زنجیرهٔ تأمین AI](08-AI-Supply-Chain-Security.md)
