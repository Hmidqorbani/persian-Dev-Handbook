# عملیات: هزینه، سرعت و Observability

**سطح**: سطح ۳
**مدت یادگیری**: ۳.۵ ساعت
**سطح سختی**: متوسط → پیشرفته
**پیش‌نیاز**: فایل‌های ۰۲ تا ۰۸ — به‌ویژه آناتومی API و usage (فایل ۰۲ فصل ۶)، گاردهای ایجنت (فایل ۰۶ فصل ۴) و معماری قابلیت (فایل ۰۸)

---

## فهرست محتوا

- [فصل ۱: اقتصاد توکن](#فصل-۱-اقتصاد-توکن)
- [فصل ۲: Caching و سرعت](#فصل-۲-caching-و-سرعت)
- [فصل ۳: Fallback و مقاوم‌سازی](#فصل-۳-fallback-و-مقاومسازی)
- [فصل ۴: Observability](#فصل-۴-observability)

---

## فصل ۱: اقتصاد توکن

> **سطح**: سطح ۳ — این فصل هزینه را از «سورپرایز فاکتور» به «متغیر مهندسی» تبدیل می‌کند: مکانیزم قیمت‌گذاری، راننده‌های هزینه‌ی واقعی، اهرم‌های کاهش با اثر/ریسک و بودجه به‌عنوان قرارداد کد.

### مقدمه

در دمو، مدل رایگان به نظر می‌آید؛ در launch، اولین فاکتور می‌آید و معمولاً «چرا این‌قدر؟» می‌پرسد. این فصل همان «چرا» را پاسخ می‌دهد و مهم‌تر — ابزار پیش‌بینی و کنترل می‌دهد، تا هزینه مثل هر منبع دیگر (CPU، حافظه، تیم) بودجه و سقف داشته باشد.

نگرش کلیدی: **هزینه‌ی AI، تابعی از تصمیم‌های مهندسی توست** — مدل انتخابی، طول زمینه (فایل ۰۳)، تعداد دورهای ایجنت (فایل ۰۶)، الگوی cache (فصل بعد). هیچ‌کدام «سرنوشت» نیستند؛ همه قابل مهارند. و مثل همیشه، مهار با **سنجش** شروع می‌شود: چیزی که اندازه نمی‌گیری، نمی‌توانی کمش کنی.

### مفاهیم اساسی

#### مکانیزم قیمت‌گذاری — سه عدد که باید بدانی

| عدد | معنا | نکته |
|---|---|---|
| قیمت ورودی | دلار بر میلیون توکن ورودی | زمینه‌ی تو (فایل ۰۳) این‌جا خرج می‌شود |
| قیمت خروجی | دلار بر میلیون توکن خروجی | **معمولاً ۳ تا ۵ برابر گران‌تر** — جواب‌های طولانی گران‌اند |
| تخفیف batch | برای کارهای غیرفوری (فایل ۰۸ فصل ۲) | معمولاً حدود نصف — برای asyncها طلا |

قیمت‌ها و اسم مدل‌ها مرتب تغییر می‌کنند؛ این فصل ساختار محاسبه را می‌دهد، اعداد روز را از صفحه‌ی قیمت رسمی بگیر (منابع) و در config نگه دار.

#### فرمول و راننده‌ها

```text
هزینه = (توکن_ورودی / ۱M) × قیمت_ورودی + (توکن_خروجی / ۱M) × قیمت_خروجی
```

راننده‌های واقعیِ بزرگ‌شدنِ این فرمول — همه از فایل‌های قبلی شناخته‌شده:

| راننده | ریشه | پیوند |
|---|---|---|
| تاریخچه‌ی چت هر بار دوباره ارسال می‌شود | گفتگوی بلند = ورودیِ صعودی | فایل ۰۲ فصل ۲ |
| زمینه‌ی پهن (k بزرگ در RAG، dump اسناد) | نویز هم گران است | فایل ۰۳ فصل ۳، فایل ۰۵ |
| دورهای ایجنت | هر دور = یک فراخوانی کامل | فایل ۰۶ فصل ۴ |
| خروجی حشو | توکن گرانِ خروجی | فایل ۰۳ (قالب و سقف) |
| مدلِ گران برای کارِ ساده | بدون router | فایل ۰۲ فصل ۵ |
| متن فارسی | توکن‌بُرد بیشتر (فصل ۱ فایل ۰۲) | — |

#### اهرم‌های کاهش — با اثر و ریسک

| اهرم | مکانیزم | اثر | ریسک/هزینه |
|---|---|---|---|
| **Model routing** | ساده‌ها به مدل ارزان، سخت‌ها به قوی (فایل ۰۲) | بزرگ | قاعده‌ی مسیریابی بد = افت کیفیت |
| **خلاصه/برش تاریخچه** | پیام‌های قدیمی فشرده (فایل ۰۲) | بزرگ در چت | جزئیات قدیمی ضایع می‌شود |
| **کاهش k و زمینه‌ی هدفمند** | فقط مرتبط‌ها (فایل ۰۵) | بزرگ | شانس حذفِ لازم |
| **سقف خروجی + پرامپت کوتاه‌کننده** | «حداکثر سه جمله» + max tokens | متوسط | بریدگی ناخواسته |
| **Prompt caching ارائه‌دهنده** (فصل ۲) | prefix ثابت ارزان‌تر | متوسط–بزرگ | طراحی پرامپت لازم |
| **Batch API** | غیرفوری‌ها شبانه | بزرگ در async | تأخیر ساعت‌ها |
| **semantic cache خودت** | جوابِ سؤال مشابه (فصل ۲) | بزرگ در FAQ | پاسخ کهنه/نابجای |

الگوی اعمال: **ترتیب اثر بر هزینه** — اول routing (بزرگ‌ترین اهرم)، بعد cache، بعد زمینه، بعد جزئیات. و هر اهرم با eval (فایل ۰۷) تأیید شود که کیفیت نیفتاده.

#### بودجه به‌عنوان قرارداد

سقف‌ها در سه لایه:

1. **سقف ارائه‌دهنده** (spend limit در داشبورد) — آخرین دیوار.
2. **سقف قابلیت در کد** — هر قابلیت سقف روزانه/هر درخواست؛ تمام شد → مسیر flag خاموش (فایل ۰۸ فصل ۲).
3. **ledger قابل گزارش** — فایل ۰۶ فصل ۴ را همین‌جا جدی می‌کنیم: بدون ledger، بودجه دکور است.

### مثال‌های عملی

#### مثال ۱: موتور هزینه — از usage تا گزارش قابلیت‌محور

```python
"""cost_engine.py — قیمت config، محاسبه‌ی فراخوانی، گزارش به‌ازای قابلیت."""
import json
from pathlib import Path

# قیمت‌ها (USD بر میلیون توکن) — از صفحه‌ی رسمی روز؛ در config بماند
PRICING = {
    "gpt-4o-mini": {"input": 0.15, "output": 0.60},
    "gpt-4o":      {"input": 2.50, "output": 10.00},
}

CALLS_LOG = Path("ai_calls.jsonl")   # لاگ ساخت‌یافته — فصل ۴ کاملش می‌کند


def call_cost(model: str, prompt_tokens: int, completion_tokens: int) -> float:
    pricing = PRICING.get(model)
    if pricing is None:
        raise KeyError(f"قیمت مدل در config نیست: {model} — اضافه‌اش کن")
    return (prompt_tokens / 1_000_000) * pricing["input"] + \
           (completion_tokens / 1_000_000) * pricing["output"]


def log_call(feature: str, model: str, usage, latency_ms: int,
             prompt_version: str = "v1") -> float:
    """پایان هر فراخوانی: هزینه‌ی حساب، رکورد لاگ، برگشت هزینه."""
    cost = call_cost(model, usage.prompt_tokens, usage.completion_tokens)
    record = {
        "feature": feature,
        "prompt_version": prompt_version,
        "model": model,
        "in_tok": usage.prompt_tokens,
        "out_tok": usage.completion_tokens,
        "cost_usd": round(cost, 6),
        "latency_ms": latency_ms,
        "ts": time.time(),
    }
    with CALLS_LOG.open("a", encoding="utf-8") as f:
        f.write(json.dumps(record, ensure_ascii=False) + "\n")
    return cost


def report_by_feature() -> None:
    """گزارش فصلی/روزانه: هر قابلیت چقدر خرج می‌کند؟ (ورودی داشبورد فصل ۴)"""
    if not CALLS_LOG.exists():
        print("هنوز لاگی نیست.")
        return
    agg: dict[str, dict] = {}
    for line in CALLS_LOG.read_text(encoding="utf-8").splitlines():
        r = json.loads(line)
        a = agg.setdefault(r["feature"], {"calls": 0, "cost": 0.0, "out_tok": 0})
        a["calls"] += 1
        a["cost"] += r["cost_usd"]
        a["out_tok"] += r["out_tok"]

    print(f"{'قابلیت':22} {'فراخوانی':>9} {'هزینه($)':>10} {'توکن خروجی':>12}")
    for feature, a in sorted(agg.items(), key=lambda kv: -kv[1]["cost"]):
        print(f"{feature:22} {a['calls']:>9} {a['cost']:>10.3f} {a['out_tok']:>12}")


if __name__ == "__main__":
    report_by_feature()
```

این گزارش، دقیقاً همان جدولی است که در جلسه‌ی بودجه لازم داری: **هر قابلیت چقدر خرج می‌کند** — و معمولاً سورپرایزها دو دسته‌اند: یک قابلیت بی‌سروصدا که ۶۰٪ فاکتور است، و یک خروجی حشو که سقف ندارد.

#### مثال ۲: سقف روزانه — بودجه به‌عنوان کد

```python
"""budget_guard.py — سقف روزانه در کد: تمام شد → مسیر خاموش (فایل ۰۸)."""
import json
import time
from datetime import datetime
from pathlib import Path
# از cost_engine: CALLS_LOG

def spend_today() -> float:
    today = datetime.now().date().isoformat()
    total = 0.0
    if CALLS_LOG.exists():
        for line in CALLS_LOG.read_text(encoding="utf-8").splitlines():
            r = json.loads(line)
            if datetime.fromtimestamp(r["ts"]).date().isoformat() == today:
                total += r["cost_usd"]
    return total


FEATURE_LIMITS = {
    "support_chat": 2.00,     # دلار در روز
    "ticket_summary": 1.00,
}

def allowed(feature: str) -> tuple[bool, str]:
    """چک قبل از هر فراخوانی: بودجه‌ی قابلیت تمام نشده؟"""
    spent_today = spend_today()   # نسخه‌ی جدی: سقف per-feature با فیلد feature
    limit = FEATURE_LIMITS.get(feature, 0.50)
    if spent_today >= limit:
        return False, f"بودجه‌ی روزانه‌ی «{feature}» تمام شد ({spent_today:.2f}/{limit})"
    return True, f"ok ({spent_today:.2f}/{limit})"

# در handler قابلیت:
# ok, msg = allowed("support_chat")
# if not ok:
#     return fallback_response(msg)   # پیام صادقانه به کاربر + مسیر ارجاع (فایل ۰۸ فصل ۱)
```

نسخه‌ی جدیِ این گارد، سقف را per-feature و per-hour می‌شمارد (از همان لاگ، با فیلتر) — ولی الگو همین است: **سقف در کد، قبل از فراخوانی، با مسیر جایگزین مشخص.**

### مقایسه و الگوهای مختلف

#### دو نگرش به هزینه — و چرا یکی می‌برد

| نگرش | رفتار | نتیجه |
|---|---|---|
| واکنشی | فاکتور آمد → «مدل را عوض کنیم» → کیفیت می‌ریزد → برمی‌گردیم | نوسان بدون کنترل |
| مهندسی‌شده (این فصل) | ledger + گزارش قابلیت‌محور + اهرم‌ها با eval | کنترل با حفظ کیفیت |

#### Routing: دو الگوی پیاده‌سازی

| الگو | مکانیزم | کِی |
|---|---|---|
| قاعده‌ای (فایل ۰۲ فصل ۵) | طول/کلیدواژه/دسته‌بندی ارزان → انتخاب مدل | شروع؛ شفاف و قابل دیباگ |
| دو‌سطحی با داوری | مدل ارزان پاسخ می‌دهد + judge (فایل ۰۷) اگر مشکوک → بازتولید با قوی | کیفیت‌محور؛ هزینه‌ی داوری را دارد |

### اشکال شایع

**۱. فقط «میلیون توکن» را نگاه کردن، نه usage واقعی.**
قیمت صفحه‌ی اول را می‌بینند و تخمین می‌زنند؛ واقعیت، usage هر فراخوانی است (فایل ۰۲ فصل ۶) که باید لاگ و جمع شود — مثال ۱.

**۲. ایجنت بدون بودجه.**
از فایل ۰۶: سقف دور و توکن بدون ledger پول — ده دورِ ایجنتِ پرمکالمه، هر دور تاریخچه‌ی رشد‌کننده. ledger + سقف، عضو ایجنت است.

**۳. خروجی بی‌سقف.**
مدلی که دو پاراگراف اضافه می‌نویسد، توکنِ گرانِ خروجی می‌خورَد. پرامپتِ قالب‌دار + سقف پارامتری — و اگر جواب ساخت‌یافته است، JSON فشرده به‌جای متن حشو.

**۴. ارزان‌تر کردنِ چیز اشتباه.**
اگر ۸۰٪ هزینه یکی از پنج قابلیت است، کش کردن FAQ به‌جای routingِ آن قابلیت، وقت‌تلف‌کردن است. گزارش مثال ۱ اول بیاید، بعد اهرم.

**۵. بدون مسیر خاموش.**
سقف تمام شد و سیستم فقط خطا می‌دهد — تجربه‌ی کاربر فرو می‌ریزد. هر سقف، مسیر جایگزین دارد (پاسخ قالبی، ارجاع انسانی، قابلیت غیرAI — فایل ۰۸ فصل ۲).

### نکته‌های مهم (Tips & Tricks)

- گزارش هزینه را **هفتگی** در جلسه/کانال تیم بگذار — روند، نه نقطه؛ ناگهان‌بودن هزینه را قبل از فاکتور می‌بینی.
- برای هر قابلیت، **هزینه‌ی به‌ازای تعامل کاربر** حساب کن (cost per use) — عدد واقعی تصمیم‌گیری است، نه کل فاکتور.
- prompt caching و batch را در محاسبه‌ی قیمت config لحاظ کن (ستون تخفیف) — اعداد واقعی‌تر، تصمیم بهتر.
- فارسی بودن داده‌ات را در تخمین فراموش نکن: **۱.۵ تا ۲ برابر توکن** نسبت به انگلیسی (فایل ۰۲) — تخمین انگلیسی‌محور، نصف واقعیت است.
- بودجه‌ی تست/توسعه را **از production جدا کن** (کلید و سقف جدا — فایل ۰۱)؛ evalهای فایل ۰۷ هم هزینه دارند و باید در بودجه‌شان باشند.

### خلاصه‌ی فصل

- فرمول ساده است: **in×قیمت + out×قیمت**؛ خروجی گران‌تر است و فارسی توکن‌بُرد بیشتر دارد.
- راننده‌های هزینه شناخته‌شده‌اند: **تاریخچه، زمینه‌ی پهن، دورهای ایجنت، حشوی خروجی، مدلِ اشتباه** — و همه مهارشدنی.
- اهرم‌ها به ترتیب اثر: **routing → cache → زمینه‌ی هدفمند → سقف خروجی → batch** — هر اهرم با eval تأیید.
- بودجه سه لایه: **سقف ارائه‌دهنده + سقف کد قابلیت + ledger گزارش‌پذیر** — و هر سقف، مسیر جایگزین.
- سنجش اول: **ledger و گزارش قابلیت‌محور** — قبل از هر بهینه‌سازی.

---

## فصل ۲: Caching و سرعت

> **سطح**: سطح ۳ — این فصل دو مکانیزم cache را می‌سازد: prompt caching ارائه‌دهنده (ارزان‌تر/سریع‌تر برای prefix ثابت) و semantic cache خودت (جوابِ سؤال مشابه) — با مرزهای درست هرکدام و ریسک‌هایی که بی‌توجه، اعتماد را می‌سوزانند.

### مقدمه

دو درد که در فصل ۱ دیدی (هزینه) و در فصل ۱ فایل ۰۸ دیدی (تأخیر)، یک درمان مشترک دارند: **نکردنِ کاری که قبلاً شده.** Cache یعنی دقیقاً همین — ولی در دنیای LLM دو شکل کاملاً متفاوت دارد که باید تفکیک‌شان کنی:

1. **Prompt caching ارائه‌دهنده**: اگر ابتدای پرامپت (prefix) بین درخواست‌ها ثابت باشد، سرویس آن را کش می‌کند — ارزان‌تر و سریع‌تر. کنترل تو فقط در **طراحی پرامپت** است (فایل ۰۳).
2. **Semantic cache خودت**: اگر سؤال جدید به سؤالی که قبلاً جواب دادی **معنایش** نزدیک است، جوابِ ذخیره‌شده را برگردان — بدون فراخوانی مدل. کنترل کامل توست، با ریسک‌های خودش.

هیچ‌کدام جایگزین دیگری نیستند؛ یکی ساختار پرامپت را بهینه می‌کند، دیگری تعداد فراخوانی‌ها را کم می‌کند. و هر دو یک قانون طلایی دارند: **cache فقط برای داده‌ای که تغییر نمی‌کند.**

### مفاهیم اساسی

#### Prompt caching ارائه‌دهنده

مکانیزم: سرویس، توکن‌های مشترکِ ابتدای پرامپت را بین درخواست‌های مشابه cache می‌کند (قیمت خواندن از cache کسری از قیمت عادی است؛ جزئیات در مستندات هر ارائه‌دهنده). شرط فعال شدن: **prefix ثابت** — پس ساختار پیام‌ها اهمیت پیدا می‌کند:

```text
✅ طراحی cache-پسند (فایل ۰۳ فصل ۱ را به‌خاطر بیاور):
   [system: قواعد ثابت]  ← همیشه یکسان → cache می‌خورد
   [زمینه‌ی پایدار: مستندات/نمونه‌ها]  ← کم‌تغییر → cache می‌خورد
   [داده‌ی متغیر: سؤال کاربر]  ← آخر، بیرون از cache

❌ طراحی cache-شکن:
   [system: قواعد + اسم کاربر + تاریخ امروز + …]  ← هر درخواست فرق دارد → هیچ‌چیز cache نمی‌شود
```

قاعده‌ی عملی: **از ثابت به متغیر بچین** — و چیزهای تغییرپذیر کوچک (تاریخ، شناسه session) را اگر لازم نیست، از پرامپت بیرون نگه‌دار. این یکی از جاهایی است که مهندسی پرامپت (فایل ۰۳) مستقیماً فاکتور را کم می‌کند.

#### Semantic cache خودت

مکانیزم: سؤال → embedding (فایل ۰۵ فصل ۲) → مقایسه با سؤال‌های کش‌شده → اگر شباهت بالای آستانه و cache تازه، جواب ذخیره‌شده برمی‌گردد.

شایسته برای: **پرسش‌های پرتکرار با جواب ثابت** — FAQ پشتیبانی، تعریف‌ها، دستورالعمل‌های ثابت.
نامناسب برای: داده‌ی زنده (قیمت، موجودی)، شخصی‌سازی‌شده (سؤالِ کاربرِ خاص)، زمان‌حساس («الان»)، و کارهای خلاقانه.

#### سه ریسک که باید با قاعده ببندی

| ریسک | سناریو | قاعده‌ی بستن |
|---|---|---|
| **پاسخ کهنه** | سیاست عوض شد، cache ۳۰ روزه هنوز جواب قدیمی می‌دهد | TTL کوتاه + **باطل‌سازی آگاهانه** (تغییر سند → پاک‌کردن cache مرتبط) |
| **پاسخ نابجا** | سؤالِ «شبیه» که در واقع فرق دارد، جوابِ سؤال قبلی را می‌گیرد | آستانه‌ی سخت‌گیرانه (شباهت بالا) + دامنه‌ی cache محدود |
| **شخصی‌سازی‌شده‌ی اشتراکی** | cache بین کاربران مشترک؛ داده‌ی کاربر A به کاربر B | کلید cache شامل context کاربر — یا cache فقط برای داده‌ی عمومی |

قاعده‌ی نهایی: semantic cache یک **بهینه‌سازی ریسک‌دار** است — با eval (فایل ۰۷) سنجیده شود (نرخ hit در برابر نرخ جواب نابجا) و اول فقط روی دامنه‌ی امن (FAQ) فعال شود.

#### Latency: اندازه‌گیری قبل از بهینه‌سازی

دو مؤلفه که در فایل ۰۲ فصل ۵ دیدی: **تأخیر اولین توکن** (UX streaming) و **سرعت تولید** (طول جواب). سنجش: `latency_ms` در لاگ (فصل ۴) + درصدی‌ها (p50/p95). و درمان‌های ترتیبی: streaming (فایل ۰۸ فصل ۱) → prompt caching → مدل سریع‌تر برای کارهای ساده (routing) → semantic cache. بدون اندازه‌گیری، این لیست حدس است.

### مثال‌های عملی

#### مثال ۱: semantic cache با آستانه و TTL — کامل

```python
"""semantic_cache.py — cache معنایی با شباهت، TTL و آمار hit/miss."""
import json
import time
import math
from openai import OpenAI

client = OpenAI()
EMBED_MODEL = "text-embedding-3-small"

SIMILARITY_THRESHOLD = 0.93   # سخت‌گیر — پاسخ نابجا از پاسخِ ازدست‌رفته بدتر است
TTL_SECONDS = 24 * 3600       # حداکثر یک روز؛ برای FAQ حتی این هم شاید زیاد باشد


def embed(texts: list[str]) -> list[list[float]]:
    response = client.embeddings.create(model=EMBED_MODEL, input=texts)
    return [d.embedding for d in response.data]


def cosine(a: list[float], b: list[float]) -> float:
    dot = sum(x * y for x, y in zip(a, b))
    return dot / ((sum(x * x for x in a) ** 0.5) * (sum(y * y for y in b) ** 0.5))


class SemanticCache:
    """دامنه‌ی امن: پرسش‌های عمومی و ثابت — نه داده‌ی زنده، نه شخصی‌سازی."""

    def __init__(self) -> None:
        self.entries: list[dict] = []      # {q, vec, answer, ts}
        self.stats = {"hit": 0, "miss": 0, "stale": 0}

    def lookup(self, question: str) -> str | None:
        now = time.time()
        q_vec = embed([question])[0]
        best, best_score = None, 0.0
        for e in self.entries:
            score = cosine(q_vec, e["vec"])
            if score > best_score:
                best, best_score = e, score
        if best is None or best_score < SIMILARITY_THRESHOLD:
            self.stats["miss"] += 1
            return None
        if now - best["ts"] > TTL_SECONDS:
            self.stats["stale"] += 1
            self.entries.remove(best)      # کهنه را دور بریز
            return None
        self.stats["hit"] += 1
        return best["answer"]

    def store(self, question: str, answer: str) -> None:
        self.entries.append({
            "q": question, "vec": embed([question])[0],
            "answer": answer, "ts": time.time(),
        })

    def invalidate(self) -> None:
        """باطل‌سازی آگاهانه: وقتی منبعِ جواب‌ها (مستندات/سیاست) عوض شد."""
        self.entries.clear()
```

و استفاده در قابلیت پشتیبانی:

```python
cache = SemanticCache()

def answer_support(question: str) -> str:
    cached = cache.lookup(question)
    if cached is not None:
        return cached                      # سؤالِ شناخته‌شده → بدون فراخوانی مدل

    answer = ask_llm(question)             # فراخوانی واقعی (هر کجای کتابچه)
    cache.store(question, answer)
    return answer

# و در مسیر به‌روزرسانی مستندات (فایل ۰۵):
# update_documents(...)
# cache.invalidate()   ← همین خط، ریسکِ «پاسخ کهنه» را می‌بندد
```

دو ترفند داخل کد: **آستانه‌ی سخت‌گیرانه** (۰.۹۳ — پاسخِ نابجا بدتر از فراخوانیِ اضافه است) و **invalidate آگاهانه** — cache بی‌باطل‌سازی، بمب ساعتی است.

#### مثال ۲: چیدن پیام‌ها برای prompt caching

```python
"""cache_friendly_messages.py — از ثابت به متغیر؛ prefix ثابت cache می‌خورد."""

# بخش ثابت — بین همه‌ی فراخوانی‌های این قابلیت یکسان:
SYSTEM_STABLE = """دستیار پشتیبانی «هفت‌لینک» هستی.
قواعد: فقط بر اساس منابع؛ ارجاع اجباری؛ اگر نیست صریح بگو.
قالب: حداکثر چهار جمله؛ لحن دوستانه‌مختصر."""

# بخش نیمه‌ثابت — با تغییر مستندات عوض می‌شود، نه با هر درخواست:
DOCS_PREFIX = "<sources>\n{docs}\n</sources>"

# بخش متغیر — هر درخواست:
def build_messages(docs: str, question: str, history: list[dict]) -> list[dict]:
    return [
        {"role": "system", "content": SYSTEM_STABLE},          # ثابت → cache
        {"role": "user", "content": DOCS_PREFIX.format(docs=docs)},  # نیمه‌ثابت
        *history,
        {"role": "user", "content": question},                 # متغیر → آخر
    ]
```

و سنجشش: در لاگ هر فراخوانی (فصل ۴)، **توکن‌های cache‌خورده** را ثبت کن (فیلد usage ارائه‌دهنده‌ها گزارشش می‌دهند) — گزارش هفتگی نشان می‌دهد طراحی prefix تو چند درصد را کش کرده.

#### مثال ۳: سنجش اثر cache — با همان لاگ‌ها

```python
"""cache_metrics.py — hit-rate و پس‌انداز؛ تصمیم با عدد."""
def cache_report(cache: SemanticCache, cost_per_call: float) -> None:
    s = cache.stats
    total = s["hit"] + s["miss"]
    if total == 0:
        print("هنوز داده نیست.")
        return
    hit_rate = s["hit"] / total
    saved = s["hit"] * cost_per_call
    print(f"hit-rate: {hit_rate:.0%} | miss: {s['miss']} | کهنه‌ی رد‌شده: {s['stale']}")
    print(f"پس‌انداز تخمینی: ${saved:.2f}")
    # قاعده‌ی تصمیم:
    # hit-rate بالا + شکایت «جواب نابجا» صفر → دامنه را بزرگ کن
    # شکایت نابجا > صفر → آستانه را سخت‌تر یا دامنه را کوچک‌تر کن
```

### مقایسه و الگوهای مختلف

#### سه نوع cache در یک جدول

| نوع | چه چیز را کش می‌کند | کنترل تو | ریسک اصلی | کِی |
|---|---|---|---|---|
| **Prompt caching (ارائه‌دهنده)** | توکن‌های prefix ثابت | طراحی پیام‌ها | تقریباً هیچ | همیشه — با چینش cache-پسند |
| **Exact cache خودت** | (سؤال، پارامترها) → پاسخ | کامل | پاسخ کهنه | خروجی قطعی‌شده‌ها (دما صفر + قالب ثابت) |
| **Semantic cache خودت** | معنای سؤال → پاسخ | کامل (آستانه، TTL، دامنه) | پاسخ نابجا/کهنه | پرتکرارهای ثابت (FAQ) |

#### جدول تصمیم: این داده cache‌پذیر است؟

| داده | cache؟ | چرا |
|---|---|---|
| «سیاست مرجوعی چیست؟» (از مستندات ثابت) | بله — semantic | جواب ثابت، پرتکرار |
| «قیمت امروز X؟» (ابزار زنده — فایل ۰۴) | نه | داده‌ی زنده |
| «سفارش‌های من؟» | نه (یا per-user با TTL کوتاه) | شخصی‌سازی‌شده |
| «الان ساعت چنده؟» | هرگز | زمان‌حساس |
| «خلاصه‌ی این سند» (همان سند، همان کاربر) | بله — exact با کلید (سند+نسخه) | ورودی قطعی |

### اشکال شایع

**۱. cache روی داده‌ی زنده.**
بزرگ‌ترین اشتباه semantic cache: «قیمت» یا «موجودی» یا «وضعیت سفارشِ من» کش می‌شود — جواب درستِ دیروز، دروغ امروز. جدول تصمیم بالا را ملاک بگیر.

**۲. cache مشترک بین کاربرانِ داده‌ی شخصی.**
پرس‌وجوی کاربر A کش می‌شود و کاربر B (با سؤال مشابه درباره‌ی داده‌ی خودش) جواب A را می‌بیند — نقض حریم خصوصی (فایل ۰۸ فصل ۳). cache عمومی یا per-user با کلید context.

**۳. TTL طولانی + بدون invalidate.**
«بگذار یک هفته بماند» — سیاست عوض می‌شود، cache نه. TTL متناسب با سرعت تغییر منبع + invalidate آگاهانه در مسیر تغییر (مثال ۱).

**۴. آستانه‌ی شل.**
شباهت ۰.۸ را «کافی» می‌گیرند → پاسخ‌های نابجا → اعتماد کاربر. پاسخ نابجا از پاسخِ ازدست‌رفته گران‌تر است؛ آستانه سخت‌گیر، گزارش نرخ نابجا با eval.

**۵. prefix ثابتِ واقعاً ثابت نبودن.**
تاریخ امروز، شناسه session، اسم کاربر داخل system → prefix هر بار متفاوت → cache ارائه‌دهنده بی‌اثر. چیزهای متغیر را از system بیرون ببر (مثال ۲).

**۶. بهینه‌سازی بدون سنجش.**
cache را می‌سازند، hit-rate را نمی‌بینند — شاید ۵٪ hit دارد و فقط پیچیدگی اضافه کرده. آمار (مثال ۳) عضو cache است، نه لوکس.

### نکته‌های مهم (Tips & Tricks)

- اول **prompt caching ارائه‌دهنده** را فعال‌سازی‌ذاتی کن (چیدن پیام‌ها) — بی‌ریسک‌ترین cache دنیا؛ بعد اگر جواب داد، semantic را با دامنه‌ی بسته اضافه کن.
- کلید exact cache را **شامل نسخه** کن: `(prompt_version, model, input-hash)` — تغییر پرامپت/مدل، cache قدیمی را خودکار بی‌اعتبار می‌کند (فایل ۰۳ فصل ۴).
- برای RAG، **embeddingهای اسناد را cache کن** (ایندکس فایل ۰۵ همین کار است) و embedding سؤال‌های پرتکرار را هم — دو cache، یک مکانیزم.
- stats را در لاگ ساخت‌یافته بنویس (فصل ۴) — hit-rate و نرخ نابجا در داشبورد، نه در حافظه‌ی متغیر.
- برای پاسخ‌های کش‌شده در UI، **شفاف باش** («پاسخ ذخیره‌شده») — کاربری که سیاست را عوض‌شده می‌بیند و جواب قدیمی می‌گیرد، از پاسخِ کش، بی‌خبرتر هم عصبانی‌تر است.
- در استریم (فایل ۰۸ فصل ۱)، hit اول cache یعنی **شروع فوری واقعی** — ترکیب cache+streaming بهترین تجربه‌ی سرعت است.

### خلاصه‌ی فصل

- دو cache متفاوت: **prompt caching ارائه‌دهنده** (چیدن پیام‌ها: ثابت اول، متغیر آخر) و **semantic cache خودت** (آستانه + TTL + دامنه‌ی محدود).
- قاعده‌ی طلایی: **cache فقط برای ثابت‌ها** — داده‌ی زنده، شخصی‌سازی‌شده و زمان‌حساس هرگز.
- سه ریسک با سه قاعده: **پاسخ کهنه → TTL + invalidate آگاهانه؛ نابجا → آستانه‌ی سخت‌گیر؛ نشت شخصی → دامنه/کلید.**
- cache بدون آمار، پیچیدگی بی‌فایده است: **hit-rate، نرخ نابجا، پس‌انداز** — در لاگ و داشبورد.
- ترتیب سرعت: **streaming → prompt caching → routing → semantic cache** — هر پله با سنجش.

---

## فصل ۳: Fallback و مقاوم‌سازی

> **سطح**: سطح ۳ — این فصل محصولت را در برابر بیماری‌های API مدل مقاوم می‌کند: تفکیک خطای گذرا از سیستمی، retry با backoff، زنجیره‌ی fallback، circuit breaker و اصل «قابلیت قابل‌خاموش‌شدن» — تا خرابیِ بیرونی، خرابیِ تو نشود.

### مقدمه

API مدل، یک وابستگی شبکه‌ای است — مثل دیتابیس یا هر سرویس بیرونی: گاهی کند است، گاهی 429 می‌دهد، گاهی ساعتی نمی‌آید. تفاوت با وابستگی‌های عادی: چون قابلیت AI جای حساسی در محصول نشسته، خرابیِ آن **ظاهرِ** خرابیِ کل محصول است.

پرسش این فصل: **وقتی مدل (یا مسیر مدل) در دسترس نیست، محصول تو چه می‌کند؟** سه جواب ممکن دارد — و فقط سومی مهندسی است:

1. خطای فنی به کاربر («5xx»).
2. قفل شدن (کاربر retry بی‌پایان می‌زند).
3. **مقاوم‌سازی**: retry هوشمند → مسیر جایگزین → degrade صادقانه → قابلیت خاموش با حالت قدیمی.

زنجیره‌ی ۳ را این فصل می‌سازیم. پایه‌هایش را قبلاً گذاشته‌ای: timeout صریح (فایل ۰۲ فصل ۶)، کدهای خطا (همان‌جا)، گاردهای ایجنت (فایل ۰۶)، «قابل‌خاموش‌شدن» (فایل ۰۸ فصل ۲) — این‌جا همه را در یک الگوی واحد جمع می‌کنیم.

### مفاهیم اساسی

#### اول: خطا را دسته‌بندی کن — هر کدام درمان متفاوت

| دسته | مثال‌ها | درمان |
|---|---|---|
| **گذرا (transient)** | 429 نرخ، 503 سرویس، timeout شبکه | retry با backoff + jitter |
| **قطعی/سیستمی** | 400 بدفرم، 401 کلید، مدلِ بازنشسته | retry بی‌معنا — رفع در کد/config؛ هشدار فوری |
| **سهمیه/پول** | 429 سهمیه، اعتبار تمام | fallback یا مسیر خاموش — با هشدار |
| **کیفیت** | جواب آمده ولی نالزو (eval فایل ۰۷) | بازتولید/داوری — نه retry خام |

قاعده‌ی کلیدی که در فایل ۰۲ فصل ۶ گذاشتیم این‌جا کامل می‌شود: **retry فقط برای گذرا.** retry روی 401، فقط سرویس لاگ را پر می‌کند.

#### retry درست: backoff + jitter + سقف

```text
تلاش‌ها: بلافاصله → صبر کوتاه → صبر بلندتر → … (سقف محدود)
backoff نمایی: ۱s، ۲s، ۴s، … تا سقف (مثلاً ۱۵s)
jitter: کمی تصادف در صبر — که هزار client هم‌زمان، طوفان هم‌زمان نسازند
```

و دو شرط پرش: **timeout صریح هر تلاش** (فایل ۰۲) و **مجموع بودجه‌ی زمان درخواست کاربر** — retry که کلِ صبرِ کاربر را بخورد، از شکست بدتر است. در مسیرهای sync کاربر، حداکثر ۲–۳ تلاش؛ در queue (فایل ۰۸ فصل ۲) تلاش‌های بیشتر مجاز — کاربر منتظر نیست.

#### fallback زنجیره‌ای: از قوی به ساده

```text
مسیر ۱: مدل اصلی (کیفیت هدف)
   ↓ شکست ماندگار
مسیر ۲: مدل جایگزین/ارزان‌تر (کیفیت کمی پایین‌تر — کار می‌کند)
   ↓ شکست ماندگار
مسیر ۳: degrade قابلیت (جواب قالبی + منابع خام + ارجاع انسانی)
   ↓
مسیر ۴: قابلیت خاموش (حالت بدون AI — فایل ۰۸ فصل ۲)
```

دو قاعده‌ی این زنجیره:

- **هر گام پایین‌تر، اعلام‌شده باشد**: پاسخِ مدلِ جایگزین در trace علامت بخورد و در UI در حد صادقانه مشخص باشد («پاسخ با کیفیت پایین‌تر» برای موارد حساس). کاربر نباید بی‌خبر مسیر بدهد — و سیستم تو نباید مسیرها را در متریک قاطی کند (فصل ۴).
- **degrade به ساده‌ترین شکل مفید**: به‌جای «جواب AI نیست»، بده: مستندات مرتبط (بازیابیِ فایل ۰۵ بدون تولید) + «کارشناس انسانی» — مفیدِ کمتر، بهتر از هیچ.

#### circuit breaker: جلوی طوفان

وقتی سرویس پایین است، retryهای همه‌ی درخواست‌ها فقط تأخیر و هزینه اضافه می‌کنند و سرویسِ در حال برگشت را خفه می‌کنند. **Circuit breaker**: بعد از n شکست پیاپی، مدتی «باز» می‌شود — درخواست‌ها بدون تلاش به مسیر fallback می‌روند؛ بعد از فاصله‌ی خنک، یک تلاش آزمایشی، و اگر سبز شد بسته می‌شود. ساده، و برای هر قابلیت AI که به کاربر خدمت می‌دهد، اجباری.

#### رصد ریشه: از خطا به درون

هر شکستِ نهایی (بعد از همه‌ی زنجیره) باید **با دسته و ریشه در لاگ** برود (فصل ۴) — و آستانه‌های هشدار داشته باشد (نرخ شکست، مدت باز بودن breaker). مقاوم‌سازی بدون رصد، پنهان‌کردن آتش است.

### مثال‌های عملی

#### مثال ۱: فراخوانی مقاوم — retry + fallback + breaker، کامل

```python
"""resilient_call.py — فراخوانی تولید‌آماده: retry گذرا، fallback مدل، circuit breaker."""
import random
import time
from openai import (
    OpenAI, RateLimitError, APIConnectionError,
    APITimeoutError, APIStatusError, AuthenticationError,
)

client = OpenAI(timeout=30.0)          # timeout صریح — فایل ۰۲ فصل ۶

MODEL_CHAIN = ["gpt-4o-mini", "gpt-4o"]  # مسیر ۱ → مسیر ۲
MAX_RETRIES = 3                          # per مدل — برای sync کاربر
MAX_WAIT = 15.0

# ── circuit breaker ساده ──
class CircuitBreaker:
    def __init__(self, threshold: int = 5, cooldown: float = 60.0) -> None:
        self.threshold = threshold
        self.cooldown = cooldown
        self.failures = 0
        self.opened_at: float | None = None

    def allow(self) -> bool:
        if self.opened_at is None:
            return True
        if time.time() - self.opened_at >= self.cooldown:
            self.opened_at = None        # تلاش آزمایشی
            self.failures = self.threshold - 1
            return True
        return False                     # باز: بدون تلاش، مستقیم fallback

    def record(self, ok: bool) -> None:
        if ok:
            self.failures = 0
            self.opened_at = None
        else:
            self.failures += 1
            if self.failures >= self.threshold:
                self.opened_at = time.time()

breaker = CircuitBreaker()


def resilient_chat(messages: list[dict], feature: str = "general") -> str:
    """برگرداند: (متن، مدلِ استفاده‌شده) — مدلِ مسیر را لاگ کن (فصل ۴)."""
    if not breaker.allow():
        return degrade(messages, feature, reason="circuit_open")

    for model in MODEL_CHAIN:
        for attempt in range(MAX_RETRIES):
            try:
                response = client.chat.completions.create(
                    model=model, messages=messages, temperature=0,
                )
                breaker.record(ok=True)
                if model != MODEL_CHAIN[0]:
                    log_fallback(feature, to_model=model)   # متریک fallback — فصل ۴
                return response.choices[0].message.content, model

            except AuthenticationError:
                breaker.record(ok=False)
                raise        # قطعی — retry/fallback بی‌معنا؛ هشدار فوری (فصل ۴)

            except (RateLimitError, APIConnectionError, APITimeoutError) as exc:
                breaker.record(ok=False)
                if attempt == MAX_RETRIES - 1:
                    break    # این مدل شکست ماندگار → مدل بعدی
                wait = min(2 ** attempt + random.random(), MAX_WAIT)   # backoff + jitter
                time.sleep(wait)

            except APIStatusError as exc:
                breaker.record(ok=False)
                break        # 400-family: همین مدل دوباره همین خطا می‌دهد

    return degrade(messages, feature, reason="all_paths_failed")


def degrade(messages: list[dict], feature: str, reason: str) -> str:
    """مسیر ۳: مفیدِ کمتر — منابع + ارجاع؛ صادق و بدون وعده."""
    log_degrade(feature, reason)             # متریک — فصل ۴
    return ("فعلاً نمی‌توانم پاسخ هوشمند بدهم. این‌ها مرتبط‌ترین بخش‌های مستندات است: "
            "[لینک‌های مرتبط] — یا به پشتیبانی انسانی ارجاع بدهید. "
            "با عذرخواهی از وقفه.")
```

چهار لایه در یک تابع: **timeout صریح → retry فقط گذرا با backoff+jitter → زنجیره‌ی مدل → degrade صادقانه** — و breaker بالای همه. توجه کن که `degrade` مفید است (منابع + ارجاع)، صادق است (وعده‌ی قطع نمی‌دهد) و در متریک ثبت می‌شود (فصل ۴).

#### مثال ۲: retry در صف — جایی که صبر آزاد است

در async (فایل ۰۸ فصل ۲، worker)، قواعد شل‌تر و عاقلانه‌ترند:

```python
"""queue_retry.py — retry تابع نمایی در worker؛ شکست نهایی → وضعیت failed با ریشه."""
import time
import json

MAX_ATTEMPTS = 5
BASE_WAIT = 30       # ثانیه — در صف، کوتاه‌مرزی ممنوع

def run_job_with_retry(job: dict, handler) -> dict:
    for attempt in range(1, MAX_ATTEMPTS + 1):
        try:
            result = handler(job["payload"])
            return {"status": "done", "result": result}
        except AuthenticationError:
            return {"status": "failed", "reason": "config_error"}   # retry بی‌معنا
        except Exception:
            if attempt == MAX_ATTEMPTS:
                return {"status": "failed", "reason": "exhausted"}
            wait = min(BASE_WAIT * (2 ** (attempt - 1)), 3600)      # تا یک ساعت
            time.sleep(wait)                                        # در worker واقعی: schedule دوباره، نه sleep
    return {"status": "failed", "reason": "unknown"}
```

تفاوت با sync: تلاش‌های بیشتر، صبرهای طولانی، و **شکست نهایی با ریشه‌ی ثبت‌شده** — کاربرِ job، وضعیت failed را می‌بیند (فایل ۰۸ فصل ۲) و متریک نرخ exhaustion در داشبورد (فصل ۴).

#### مثال ۳: آزمایش مقاوم‌سازی — chaos کوچک خودت

مقاوم‌سازی بدون تست، ادعاست. آزمایش ساده و صادقانه:

```python
"""chaos_check.py — سرویس را عمداً بد رفتار کن و ببین زنجیره می‌ایستد."""
from unittest.mock import patch

def test_fallback_happens():
    with patch("__main__.client.chat.completions.create",
               side_effect=RateLimitError("test", body=None, response=None)):
        # فراخوانی اول: ۳ retry مسیر ۱ → مسیر ۲ (باز هم خطا) → degrade
        # انتظار: پاسخ degrade با reason=all_paths_failed + لاگ degrade
        answer, model = resilient_chat([{"role": "user", "content": "سلام"}])
        assert "نمی‌توانم" in answer          # degrade، نه crash، نه hang

def test_breaker_opens():
    # ۵ شکست پیاپی → allow() باید False بدهد (بدون تلاش → degrade فوری)
    for _ in range(5):
        breaker.record(ok=False)
    assert breaker.allow() is False
```

سناریوهای مرزی که باید تست شوند — همین‌ها در eval معماری (فایل ۰۷ فصل ۲) ثبت شوند: **timeout در نیمه‌ی جواب، 429 پایدار، کلید باطل، breaker در باز**. مقاوم‌سازی، regression test می‌خواهد مثل همه‌چیز.

### مقایسه و الگوهای مختلف

#### استراتژی‌های شکست — از بد به خوب

| استراتژی | رفتار | حکم |
|---|---|---|
| Crash خام | exception به کاربر | هرگز |
| Hang | بدون timeout، بی‌پایان | هرگز (timeout از فایل ۰۲) |
| Retry بی‌سقف | تا ابد امتحان | هرگز |
| Retry + fallback + breaker + degrade | زنجیره‌ی این فصل | **پیش‌فرض production** |

#### degrade: دو الگوی پاسخ

| الگو | مکانیزم | کِی |
|---|---|---|
| **مفیدِ کمتر** | منابع مرتبط خام (بازیابی فایل ۰۵) + ارجاع انسانی | وقتی داده قابل بازیابی است |
| **قالبی + هشدار** | جواب آماده‌ی صادقانه + ثبت | وقتی حتی بازیابی هم وابسته به خط است |

و قاعده‌ی صادقگی (فایل ۰۸ فصل ۱): هر دو، در UI با لحن درست — «فعلاً…»، نه «به‌زودی همه‌چیز اوکی است».

### اشکال شایع

**۱. retry روی خطای قطعی.**
401/400 تا ده بار تلاش — لاگ پر، سرویس کند، مشکل سرِ جایش. دسته‌بندی فصل (گذرا/قطعی) را در exceptها کد کن (مثال ۱).

**۲. بدون jitter.**
هزار درخواست هم‌زمان، هم‌زمان retry می‌کنند → موج بعدی، سنگین‌تر از اولی. jitter یکی از ارزان‌ترین ثبات‌سازهای دنیاست.

**۳. breaker نداشتن.**
سرویس ۲۰ دقیقه پایین است؛ هر درخواست، ۳ تلاش × ۳ مدل — تأخیر تجمعی، فاکتور، لاگ. breaker «درد را تشخیص می‌دهد و استراحت می‌دهد».

**۴. fallback بی‌اعلام.**
پاسخ از مدل جایگزین می‌آید، متریک همان‌طور می‌شمارد و کاربر بی‌خبر — کیفیت افت کرده، تو نمی‌دانی کجا، کاربر هم نمی‌دانی چرا. مسیر fallback، داده است (متریک + UI حساس‌ها).

**۵. degrade بی‌دلیل خاص.**
«خطا رخ داد» به‌جای ریشه (breaker باز؟ همه‌ی مسیرها؟ 429 پایدار؟). بدون ریشه، صبحِ بعد نمی‌دانی چه تعمیری لازم است. reason در لاگ degrade، اجباری.

**۶. مقاوم‌سازی فقط برای مدل.**
ابزارهای ایجنت (فایل ۰۴) و بازیابی RAG (فایل ۰۵) هم شبکه‌ای‌اند و همان درمان را می‌خواهند: timeout، retry گذرا، degrade. الگو را به همه‌ی فراخوانی‌های بیرونی تعمیم بده.

### نکته‌های مهم (Tips & Tricks)

- زنجیره‌ی fallback را **در config** نگه‌دار (لیست مدل‌ها به‌ازای هر قابلیت) — تغییر زنجیره = config، نه deployِ منطق.
- برای هر قابلیت، **«مسیر ۴» را از اول طراحی کن**: حالت بدون AI چه شکلی است؟ (فایل ۰۸: قابلیت قابل‌خاموش‌شدن) — اگر جواب نداری، قابلیت آماده‌ی production نیست.
- در استریم (فایل ۰۸)، شکست **بعد از شروع جریان** هم مدیریت شود: پیام قطعِ وسط + «دوباره تلاش» — تجربه‌ی نصفه‌پاسخِ معلق، بدترین حالت.
- نرخ fallback و مدت باز بودن breaker را **در داشبورد (فصل ۴)** بگذار با هشدار — این‌ها سلامتِ وابستگی‌های بیرونی تو هستند.
- هر حادثه‌ی واقعی، **یک تست chaos جدید** شود — همان الگوی «هر باگ یک تست» فایل ۰۷، این‌بار برای زیرساخت.
- کلیدها را هم مقاوم کن: **چرخش کلید بدون downtime** (دو کلید فعال هم‌زمان در config) — خطای 401 وسط روز، معنایش این آماده‌بودن است.

### خلاصه‌ی فصل

- دسته‌بندی خطا، درمان را تعیین می‌کند: **گذرا → retry با backoff+jitter؛ قطعی → هشدار و رفع؛ سهمیه → fallback؛ کیفیت → داوری.**
- زنجیره‌ی مقاوم: **timeout → retry محدود → fallback مدل → degrade مفید و صادق → قابلیت خاموش** — با breaker بالای همه.
- **circuit breaker** جلوی طوفان retryها را می‌گیرد: باز → بدون تلاش، مستقیم fallback.
- هر گام پایین‌تر **اعلام‌شده و متریک‌دار** است — مسیر مخفی، یعنی کیفیت ناپدیدِ بی‌ردیب.
- مقاوم‌سازی با **chaos test** اثبات می‌شود؛ در صف، retry شل‌تر و صبر بلندتر؛ و الگو به همه‌ی فراخوانی‌های بیرونی تعمیم می‌یابد.

---

## فصل ۴: Observability

> **سطح**: سطح ۳ — این فصل «دید» را می‌سازد: لاگ ساخت‌یافته‌ی هر فراخوانی، متریک‌های سلامت (هزینه/تأخیر/خطا/کیفیت)، trace ایجنت و هشدارها — چون همه‌ی فصل‌های قبل، روی این دید تکیه کرده‌اند.

### مقدمه

در طول کتابچه، بی‌وقفه پیش‌نمایش این فصل را داده‌ایم: «usage را از روز اول لاگ کن» (فایل ۰۲)، «trace ایجنت» (فایل ۰۶)، «ledger هزینه» (فصل ۱ همین فایل)، «flags در لاگ» (فایل ۰۷). این فصل همه را در یک سیستم واحد جمع می‌کند — و دلیلِ این همه پیوند را هم می‌گوید:

> **در محصولات AI، «چیزی که نمی‌بینی» سه برابر خطرناک‌تر است**: کیفیت خروجی نامرئی است (فایل ۰۷)، هزینه نامرئی است (فصل ۱) و شکست‌ها نامرئی‌اند (فصل ۳). Observability یعنی هر سه را دیدنی کن.

و یک تفاوت با observability کلاسیک: در سیستم‌های معمولی، لاگ «چه شد» است؛ در AI، لاگ باید **چه گفته شد و با چه ورودی** هم باشد — چون خودِ رفتار سیستم (پرامپت/پاسخ) عضو مشکل می‌شود. البته با مرز حریم خصوصی (فایل ۰۸ فصل ۳): محتوا mask، متادیتا کامل.

### مفاهیم اساسی

#### سه ستون — برای AI

| ستون | چیست | مخصوصِ AI |
|---|---|---|
| **لاگ ساخت‌یافته** | رکورد هر رویداد (فراخوانی، ابزار، degrade) | پرامپت/پاسخ (mask‌شده) + نسخه‌ی پرامپت + flags |
| **متریک** | جمع‌های زمانی: نرخ، درصدی‌ها، جمع هزینه | هزینه per قابلیت، نرخ «نمی‌دانم»، نرخ fallback |
| **Trace** | مسیر کامل یک تعامل: گام‌به‌گام | trace ایجنت (فایل ۰۶) — دورها، ابزارها، تکرارها |

و یک ستون چهارمِ مخصوص AI که بقیه را زنده نگه می‌دارد: **نمره‌ی eval و feedback کاربر** (فایل ۰۷ فصل ۲ و فایل ۰۸ فصل ۱) — کیفیت، متریک زمانی هم هست: افت نمره = زنگ خطر.

#### لاگ هر فراخوانی — schema استاندارد

```json
{
  "ts": 1769900000,
  "feature": "support_chat",
  "session": "sess-42",
  "prompt_version": "v14",
  "model": "gpt-4o-mini",
  "in_tok": 842, "out_tok": 156,
  "cost_usd": 0.000219,
  "latency_ms": 1870,
  "status": "ok",             // ok | degraded | error | cancelled
  "fallback_from": null,
  "cache_hit": false,
  "flags": [],                // suspicious و… (فایل ۰۷)
  "request_ref": "req-9f3"    // پیوند به trace کامل (محتوا mask‌شده)
}
```

قاعده‌های ثابت: **هر فراخوانی یک رکورد؛ بدون PII در متن؛ نسخه‌ی پرامپت همیشه** (فایل ۰۳ فصل ۴ — بدون آن، سه ماه بعد «چه چیزی عوض شد؟» بی‌جواب است)؛ پیوند به trace برای جزئیات.

#### متریک‌های سلامت — پنج زنگ خطر

| متریک | زنگ خطر وقتی | ریشه‌ی معمول |
|---|---|---|
| **هزینه/روز per قابلیت** | جهش ناگهانی | حلقه/تکرار، خروجی حشو، سوءاستفاده (فایل ۰۷) |
| **latency p95** | دو برابر خط پایه | سرویس کند، زنجیره‌ی fallback فعال (فصل ۳) |
| **نرخ خطا/degrade** | بالای آستانه | وابستگی پایین، کلید/سهمیه |
| **نرخ «نمی‌دانم» (RAG)** | جهش | ایندکس کهنه/ناکامل (فایل ۰۵) |
| **نمره eval / feedback منفی** | افت روند | پرامپت/مدل عوض شد (فایل ۰۷، ۰۳) |

قاعده‌ی عملی: هر متریک، **آستانه + صاحب + مسیر دیباگ** — متریک بدون صاحب، دکور داشبورد است.

#### داشبورد و هشدار: از jsonl به دید

تا وقتی مقیاس کوچک است، همین فایل‌های jsonl که در فصل‌های قبل ساختیم کافی‌اند — داشبورد یعنی کوئری‌های جمعی روی آن‌ها. در مقیاس بعدی: سرویس لاگ و APM یا ابزارهای تخصصی LLM (Langfuse، LangSmith و مشابه‌ها — که trace پرامپت/پاسخ را بومی می‌دهند) یا OpenTelemetry. الگو و schema یکسان می‌ماند؛ فقط ابزار جمع و نمایش فرق می‌کند — همان قاعده‌ی همیشگی کتابچه.

### مثال‌های عملی

#### مثال ۱: logger مرکزی — همه‌ی فصل‌ها در یک تابع

```python
"""ai_logger.py — لاگ مرکزی هر فراخوانی؛ مصرف‌کننده‌ی همه‌ی فایل‌های قبلی."""
import json
import time
from pathlib import Path
# از cost_engine (فصل ۱): call_cost
# از resilient_call (فصل ۳): status/fallback

LOG = Path("ai_calls.jsonl")


def log_ai_call(
    feature: str,
    model: str,
    usage,                       # response.usage — فایل ۰۲ فصل ۶
    latency_ms: int,
    *,
    session: str = "",
    prompt_version: str = "v1",
    status: str = "ok",          # ok|degraded|error|cancelled
    fallback_from: str | None = None,
    cache_hit: bool = False,
    flags: list[str] | None = None,
    request_ref: str = "",
) -> float:
    cost = call_cost(model, usage.prompt_tokens, usage.completion_tokens)
    record = {
        "ts": time.time(),
        "feature": feature,
        "session": session,
        "prompt_version": prompt_version,
        "model": model,
        "in_tok": usage.prompt_tokens,
        "out_tok": usage.completion_tokens,
        "cost_usd": round(cost, 6),
        "latency_ms": latency_ms,
        "status": status,
        "fallback_from": fallback_from,
        "cache_hit": cache_hit,
        "flags": flags or [],
        "request_ref": request_ref,
    }
    with LOG.open("a", encoding="utf-8") as f:
        f.write(json.dumps(record, ensure_ascii=False) + "\n")
    return cost

# الگوی استفاده در هر قابلیت:
# start = time.perf_counter()
# response = resilient_chat(...)            # فصل ۳
# log_ai_call(feature="support_chat", model=model, usage=response.usage,
#             latency_ms=int((time.perf_counter() - start) * 1000),
#             session=session_id, prompt_version="v14",
#             status="ok" if model == "gpt-4o-mini" else "degraded")
```

نکته‌ی جالب این logger: **status از مسیر fallback می‌آید** (فصل ۳) — یعنی سلامتِ مقاوم‌سازی، خودکار متریک می‌شود.

#### مثال ۲: داشبورد کوئری‌محور — از همان jsonl

```python
"""dashboard.py — گزارش روزانه/فصلی: هزینه، تأخیر، خطا، fallback."""
import json
from datetime import datetime
from pathlib import Path

LOG = Path("ai_calls.jsonl")


def load() -> list[dict]:
    if not LOG.exists():
        return []
    return [json.loads(line) for line in LOG.read_text(encoding="utf-8").splitlines()]


def daily_summary() -> None:
    rows = load()
    if not rows:
        print("لاگی نیست.")
        return

    today = datetime.now().date().isoformat()
    today_rows = [r for r in rows
                  if datetime.fromtimestamp(r["ts"]).date().isoformat() == today]

    total_cost = sum(r["cost_usd"] for r in today_rows)
    per_feature: dict[str, dict] = {}
    for r in today_rows:
        a = per_feature.setdefault(r["feature"],
                                   {"calls": 0, "cost": 0.0, "err": 0, "lats": []})
        a["calls"] += 1
        a["cost"] += r["cost_usd"]
        a["lats"].append(r["latency_ms"])
        if r["status"] in ("error", "degraded"):
            a["err"] += 1

    print(f"=== {today} — کل: ${total_cost:.2f} ===")
    for feature, a in sorted(per_feature.items(), key=lambda kv: -kv[1]["cost"]):
        lats = sorted(a["lats"])
        p95 = lats[int(len(lats) * 0.95) - 1] if lats else 0
        err_rate = a["err"] / a["calls"]
        print(f"{feature:20} calls={a['calls']:>5} cost=${a['cost']:.3f} "
              f"p95={p95}ms err={err_rate:.1%}")

    # زنگ‌های خطر — آستانه‌ها در config، صاحبشان مشخص
    alerts = []
    if total_cost > 5.00:
        alerts.append("هزینه‌ی روز از سقف گذشت ($5)")
    for feature, a in per_feature.items():
        if a["err"] / a["calls"] > 0.05:
            alerts.append(f"نرخ خطا/بیهودگی «{feature}» > ۵٪")
    for alert in alerts:
        print(f"⚠️  {alert}")
```

این خروجی، صبح‌های توست: هزینه per قابلیت، p95 تأخیر، نرخ خطا و هشدارهای آستانه‌ای — همه از همان لاگ واحد. در مقیاس بعدی، همین کوئری‌ها به سرویس لاگ منتقل می‌شوند؛ ساختار گزارش، ثابت می‌ماند.

#### مثال ۳: پاسخ به دو سؤال کلاسیک — با ردگیری

```python
"""trace_questions.py — دو سؤالی که observability برایش ساخته شده."""

def why_did_cost_spike(feature: str, day: str) -> None:
    """«چرا فاکتور این قابلیت جهش کرد؟» — ردگیری با فیلدهای لاگ."""
    rows = [r for r in load() if r["feature"] == feature
            and datetime.fromtimestamp(r["ts"]).date().isoformat() == day]
    by_version: dict[str, dict] = {}
    for r in rows:
        a = by_version.setdefault(r["prompt_version"], {"calls": 0, "out": 0})
        a["calls"] += 1
        a["out"] += r["out_tok"]

    print("توکن خروجی به‌ازای نسخه‌ی پرامپت (مظنون ۱: حشوی نسخه‌ی جدید):")
    for v, a in by_version.items():
        print(f"  {v}: calls={a['calls']} out_avg={a['out'] / max(a['calls'], 1):.0f}")

    cache_misses = sum(1 for r in rows if not r["cache_hit"])
    print(f"cache misses: {cache_misses}/{len(rows)} (مظنون ۲: prefix شکسته — فصل ۲)")


def why_quality_dropped(feature: str) -> None:
    """«چرا feedback منفی زیاد شد؟» — پیوند لاگ با نسخه/مسیر."""
    rows = [r for r in load() if r["feature"] == feature]
    degraded = sum(1 for r in rows if r["status"] == "degraded")
    versions: dict[str, int] = {}
    for r in rows:
        versions[r["prompt_version"]] = versions.get(r["prompt_version"], 0) + 1
    print(f"degraded: {degraded}/{len(rows)} — پاسخ‌ها از مسیر fallback آمده‌اند؟")
    print(f"توزیع نسخه‌ی پرامپت: {versions} — افت از کدام نسخه شروع شد؟")
```

این دو تابع، حاصلی است که چراغ‌های خاموش نمی‌دهند: **پاسخ به «چرا» با ردگیری، در دقیقه‌ها** — به‌شرط schema ثابت از روز اول.

### مقایسه و الگوهای مختلف

#### مسیر بلوغ observability

| مرحله | ابزار | کِی |
|---|---|---|
| jsonl + اسکریپت‌ها (این فصل) | صفر وابستگی | شروع تا چند کاربر |
| سرویس لاگ/APM | جمع‌آوری، جست‌وجو، هشدار | مقیاس کاربر/سرورها |
| پلتفرم تخصصی LLM (Langfuse/LangSmith) | trace پرامپت/پاسخ بومی، مقایسه‌ی نسخه‌ها | تیم چندنفره + حجم تغییر پرامپت |
| OpenTelemetry | استاندارد باز در کنار بقیه‌ی سیستم | هم‌گرایی infra |

قاعده: **schema و متریک‌ها از امروز ثابت**؛ ابزار با مقیاس عوض می‌شود — داده‌هایت قابلیت مهاجرت دارند.

#### چه چیزی را لاگ کنم؟ — مرز حریم خصوصی

| محتوا | لاگ؟ |
|---|---|
| متادیتا (usage، latency، نسخه، وضعیت) | همیشه |
| پرامپت/پاسخ کامل | هرگز خام — mask (فایل ۰۸ فصل ۳) یا فقط request_ref |
| داده‌ی ابزار/trace ایجنت | mask + سقف retention |
| flags و نشانه‌های امنیتی (فایل ۰۷) | بله — بدون متن ورودی |

### اشکال شایع

**۱. لاگ PII خام.**
با دیباگِ راحت شروع می‌شود، با ممیزیِ ناتمام تمام — فایل ۰۸ فصل ۳: ماسک در همه‌ی مسیرهای خروج، شامل لاگ. logger مرکزی (مثال ۱) نقطه‌ی اعمال واحد است.

**۲. بدون نسخه‌ی پرامپت.**
«چند هفته‌ی پیش جواب‌ها بهتر بود» — با کدام پرامپت؟ بدون prompt_version در لاگ، روند کیفیت قابل ردگیری نیست و A/B ممکن نیست (فایل ۰۳ فصل ۴).

**۳. متریک بدون هشدار و صاحب.**
داشبورد قشنگ که کسی نمی‌بیند؛ متریک‌هایی که آستانه ندارند. هر متریک: آستانه، هشدار، صاحب — یا حذفش کن.

**۴. فقط aggregate، بدون نمونه.**
«p95 بد است» — کدام درخواست؟ کدام trace؟ متریک، در را به خطای فرد نشان می‌دهد؛ trace و نمونه‌های log، همان را باز می‌کنند. این دو، جفت‌اند.

**۵. لاگِ خودش هزینه.**
هر فراخوانی با پرامپت کاملِ mask‌نشده در JSON زیبا — حجم لاگ، هزینه و ریسک. سقف اندازه، retention مشخص (فایل ۰۸)، و محتوای سنگین فقط با reference.

**۶. بدون مالک.**
«observability را کسی اضافه کرد» — و هیچ‌کس آن را نگه نمی‌دارد: آستانه‌های قدیمی، داشبورد مرده. متریک‌ها مالک دارند، مثل کد.

### نکته‌های مهم (Tips & Tricks)

- logger مرکزی (مثال ۱) را از **اولین قابلیت** اجباری کن — «هر فراخوانی مدل باید از این‌جا بگذرد»؛ قاعده‌ای که بعداً اضافه شود، گورستان فراخوانی‌های لاگ‌نشده می‌سازد.
- `request_ref` را به trace کامل (فایل ۰۶) وصل کن — لاگ یک‌خطی، در را به کل مسیر باز می‌کند.
- گزارش روزانه (مثال ۲) را **خودکار به کانال تیم** بفرست — داشبوردی که صبح‌ها خودش می‌آید، بیشترین خواننده را دارد.
- نمره‌ی eval (فایل ۰۷) را در همان داشبورد بگذار — کیفیت و هزینه کنار هم؛ بهینه‌سازیِ یکی به قیمت دیگری، همان‌جا دیده می‌شود.
- برای ایجنت‌ها، متریکِ مخصوص اضافه کن: **توزیع تعداد دورها**، نرخ رسیدن به گاردها (فایل ۰۶ فصل ۴) — رشدِ میانگین دورها، از اولین علامت‌های گیرکردن است.
- فصلی یک‌بار، **یک حادثه‌ی شبیه‌سازی‌شده** را با داشبورد ردگیری کن (فصل ۳): اگر در ۱۰ دقیقه ریشه را نیافتی، schema/ابزار کم دارد.

### خلاصه‌ی فصل

- سه ستون برای AI: **لاگ ساخت‌یافته** (با نسخه‌ی پرامپت و mask)، **متریک** (هزینه/تأخیر/خطا/کیفیت با آستانه و صاحب)، **trace** (مسیر کامل ایجنت).
- لاگ یک فراخوانی، قرارداد ثابت دارد: **usage، هزینه، latency، status، fallback_from، cache_hit، prompt_version** — بدون PII، با request_ref.
- پنج زنگ خطر پایه: **جهش هزینه، p95، نرخ خطا/degrade، نرخ «نمی‌دانم»، افت eval/feedback.**
- داشبورد = **کوئری روی همان داده‌ها**؛ ابزار با مقیاس عوض می‌شود، schema و متریک‌ها ثابت می‌مانند.
- ارزش نهایی: **پاسخ به «چرا» در دقیقه‌ها** — جهش هزینه، افت کیفیت، حادثه‌ی فصل ۳ — همه با ردگیری.

---

## منابع و مراجع

**قیمت‌گذاری و cache (اعداد روز را از این‌جا)**:

- قیمت OpenAI و توضیح batch/prompt caching: <https://openai.com/api/pricing/>
- مستندات Prompt Caching در OpenAI: <https://platform.openai.com/docs/guides/prompt-caching>
- Prompt Caching در Anthropic: <https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching>
- Batch API در OpenAI: <https://platform.openai.com/docs/guides/batch>

**Retry و مقاوم‌سازی**:

- tenacity — کتابخانه‌ی retry/backoff پایتون: <https://tenacity.readthedocs.io>
- الگوی Circuit Breaker (مرجع کلاسیک): <https://martinfowler.com/bliki/CircuitBreaker.html>

**Observability**:

- OpenTelemetry — استاندارد باز trace/متریک: <https://opentelemetry.io/docs/>
- Langfuse — پلتفرم متن‌باز observability اختصاصی LLM: <https://langfuse.com/docs>
- LangSmith — trace و ارزیابی اکوسیستم LangChain: <https://docs.smith.langchain.com>

**پیش‌نگاه فایل‌های بعد**:

- فایل ۱۰ (سطح ۴) — چیزهایی که لازم نیست؛ مرزهای آگاهانه‌ی یادگیری
- فایل ۱۱ — پروژه‌ی کامل: همه‌ی الگوهای این فایل، اجراشده در یک قابلیت واقعی

> یادآوری: قیمت‌ها، فیلدهای usage و قابلیت‌های caching/providerها تغییر می‌کنند — اعداد و APIها را از مستندات رسمی روز بگیر؛ schema لاگ، متریک‌ها و زنجیره‌ی مقاوم‌سازی این فایل، مستقل از آن‌ها پایدارند.
