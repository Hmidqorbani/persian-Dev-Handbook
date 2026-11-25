# بخش ۰۷: مهندسی انتشار

**مدت یادگیری**: ۳ تا ۴ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**:
- بخش ۰۶ (CI/CD)

---
## فهرست محتوا
- [فصل ۳۶: مهندسی انتشار چیست](#فصل-۳۶-مهندسی-انتشار-چیست)
- [فصل ۳۷: استراتژی‌های استقرار](#فصل-۳۷-استراتژیهای-استقرار)
- [فصل ۳۸: Feature Flag و مدیریت تغییر رفتار](#فصل-۳۸-feature-flag-و-مدیریت-تغییر-رفتار)
- [فصل ۳۹: نسخه‌گذاری و مدیریت Artifact](#فصل-۳۹-نسخهگذاری-و-مدیریت-artifact)
- [فصل ۴۰: Rollback و مدیریت شکست](#فصل-۴۰-rollback-و-مدیریت-شکست)

---

## فصل ۳۶: مهندسی انتشار چیست

### مقدمه

**مهندسی انتشار (Release Engineering)** رشته‌ای است که بین «کد آماده» و «کاربر راضی» می‌نشیند. کارش این است که انتشار **قابل پیش‌بینی، تکرارپذیر و قابل بازگشت** باشد.

این رشته در گوگل و سازمان‌های بزرگ، تیم و فرآیند مستقل دارد. در تیم‌های کوچک، بخشی از کار DevOps است.

### مفاهیم اساسی

**چهار مسئولیت مهندسی انتشار:**

| مسئولیت | سؤال |
|---|---|
| **قابلیت تکرار** | همان artifact، همان نتیجه؟ |
| **قابلیت ردیابی** | این نسخه از کجا آمد؟ |
| **قابلیت بازگشت** | چطور برگردیم؟ |
| **هماهنگی** | چه کسی، کِی، چطور؟ |

**سه مفهوم کلیدی:**

| مفهوم | معنا |
|---|---|
| **Artifact** | خروجی فیزیکی build (ایمیج، بسته) |
| **Release** | مجموعه‌ای از artifactها که با هم منتشر می‌شوند |
| **Deployment** | نصب یک release روی محیط |

**نکته مهم**: یک Release می‌تواند چند Deployment داشته باشد (staging، production، مناطق مختلف).

**«قابلیت ردیابی» — چرا حیاتی است:**

```
سؤال: "چه نسخه‌ای روی production است؟"
❌ پاسخ بد: "آخرین نسخه"
✅ پاسخ خوب: "v1.4.2، commit a1b2c3d، build #892، digest sha256:abc..."

سؤال: "این باگ از کِی هست؟"
✅ پاسخ: "از v1.4.0، commit f4e5d6c، MR #234"
```

بدون ردیابی، دیباگ production حدس زدن است.

### مثال‌های عملی

**زنجیره ردیابی کامل:**

```
ایده → issue #142
  ↓
branch: feat/142-add-expiry
  ↓
commit: a1b2c3d
  ↓
MR: !234
  ↓
pipeline: #1892
  ↓
artifact: registry/app:a1b2c3d
  ↓
digest: sha256:9f8e7d6...
  ↓
release: v1.4.2
  ↓
deployment: production, 1405/07/15 09:30
```

**هر حلقه به قبلی وصل است.** از هر نقطه می‌توانی به هر نقطه برسی.

**چک‌لیست انتشار (حداقلی):**

```
قبل:
  [ ] artifact در registry موجود و digest ثبت شده
  [ ] migration سازگار با نسخه قبل
  [ ] rollback تست شده
  [ ] feature flag تنظیم شده (اگر لازم)
  [ ] تیم در دسترس است

در حین:
  [ ] canary/rolling با متریک
  [ ] smoke test
  [ ] بررسی نرخ خطا و latency

بعد:
  [ ] تأیید سلامت
  [ ] ثبت نسخه در changelog
  [ ] اطلاع‌رسانی
  [ ] پاک‌سازی منابع قدیمی
```

### مقایسه و الگوهای مختلف

**سه مدل انتشار:**

| مدل | توضیح | مناسب |
|---|---|---|
| **Big Bang** | همه‌چیز یک‌باره | ❌ |
| **Phased** | تدریجی، گروه‌به‌گروه | ✅ |
| **Continuous** | پیوسته، کوچک | ✅✅ |

**Release Train در برابر On-demand:**

| معیار | Train | On-demand |
|---|---|---|
| زمان‌بندی | ثابت (مثلاً دوهفته) | هر وقت آماده |
| پیش‌بینی | ✅ | ❌ |
| انعطاف | ❌ | ✅ |
| مناسب | محصول با نسخه | SaaS |

**سه سطح بلوغ:**

| سطح | ویژگی |
|---|---|
| ۱ | release دستی، بدون ردیابی |
| ۲ | artifact نسخه‌دار، deploy خودکار |
| ۳ | ✅ canary، flag، ردیابی کامل |

### اشکال شایع

**اشکال ۱: «آخرین نسخه» به‌جای نسخه دقیق**

نمی‌دانی چه چیزی روی production است. همیشه digest یا نسخه دقیق.

**اشکال ۲: artifact بازسازی‌شده**

«همان کد را دوباره build کردیم» — ممکن است متفاوت باشد. یک‌بار build.

**اشکال ۳: release بدون changelog**

کاربر نمی‌داند چه عوض شد. و تو نمی‌دانی چه منتشر کردی.

**اشکال ۴: migration بدون rollback plan**

اگر migration برگشت‌پذیر نباشد، rollback غیرممکن است.

**اشکال ۵: release جمعه شب**

تیم در دسترس نیست. قاعده: پنجشنبه/جمعه بدون on-call کامل، نه.

### نکات مهم (Tips & Tricks)

- **digest را ثبت کن**، نه فقط tag.
- **زنجیره ردیابی** از issue تا deployment.
- **changelog خودکار** از commit message.
- **rollback را تمرین کن** — نه فقط طراحی.
- **migration با expand-contract.**
- **چک‌لیست انتشار** در MR template.

### خلاصه فصل

- مهندسی انتشار: تکرارپذیری، ردیابی، بازگشت، هماهنگی.
- Release ≠ Deployment.
- زنجیره ردیابی از issue تا production.
- digest را ثبت کن.
- rollback را تمرین کن.

---

## فصل ۳۷: استراتژی‌های استقرار

### مقدمه

انتخاب استراتژی استقرار، مستقیماً روی ریسک و سرعت اثر می‌گذارد. این فصل چهار استراتژی اصلی را با شرایط استفاده‌شان مقایسه می‌کند.

### مفاهیم اساسی

**چهار استراتژی:**

| استراتژی | مکانیزم | ریسک | منابع |
|---|---|---|---|
| **Recreate** | توقف، نصب، شروع | ❌❌ downtime | ✅ کم |
| **Rolling** | تدریجی، نسخه به نسخه | ✅ | ✅ کم |
| **Blue-Green** | دو محیط، سوییچ | ✅✅ | ❌ ۲× |
| **Canary** | درصد ترافیک | ✅✅✅ | ❌ ۲× |

**جزئیات هر کدام:**

**Recreate:**
```
۱. همه instanceهای قدیم را متوقف کن
۲. نسخه جدید را نصب کن
۳. شروع کن
⏱️ downtime: دقیقه تا ساعت
```

**Rolling:**
```
۱. یک instance را با نسخه جدید جایگزین کن
۲. منتظر آماده شدن بمان
۳. تکرار تا همه
⏱️ downtime: صفر (اگر health check درست باشد)
```

**Blue-Green:**
```
۱. Green (نسخه جدید) را کنار Blue (فعلی) بالا بیاور
۲. تست Green
۳. ترافیک را به Green سوییچ کن
۴. Blue را نگه دار (rollback فوری)
⏱️ downtime: صفر، rollback: ثانیه
```

**Canary:**
```
۱. نسخه جدید را با ۵٪ ترافیک بالا بیاور
۲. متریک را ببین
۳. اگر سالم، ۲۵٪ → ۵۰٪ → ۱۰۰٪
⏱️ downtime: صفر، ریسک: محدود
```

### مثال‌های عملی

**مقایسه عددی — یک باگ در نسخه جدید:**

| استراتژی | کاربران تحت تأثیر | زمان کشف | زمان rollback |
|---|---|---|---|
| Recreate | ۱۰۰٪ | فوری (همه خراب) | ۱۰-۳۰ دقیقه |
| Rolling | ۱۰۰٪ (تدریجی) | دقیقه | ۵-۱۰ دقیقه |
| Blue-Green | ۱۰۰٪ | دقیقه | ✅ ۳۰ ثانیه |
| Canary | ✅ ۵-۲۵٪ | دقیقه | ✅ ۳۰ ثانیه |

**Canary به‌طور واضح برنده است** — اگر متریک داشته باشی.

**بدون متریک، Canary بی‌فایده است:**

```
canary بدون متریک = deploy با تأخیر
```

باید بدانی «سالم» چه معنایی دارد: نرخ خطا، latency، و آستانه.

**آستانه Canary:**

| متریک | آستانه rollback |
|---|---|
| نرخ خطا (5xx) | > ۱٪ |
| latency p95 | > ۲× نسخه قبل |
| نرخ موفقیت کسب‌وکار | < ۹۸٪ |

**Canary تدریجی — زمان‌بندی:**

```
۰۹:۰۰  ۵٪   → ۱۵ دقیقه صبر
۰۹:۱۵  ۲۵٪  → ۱۵ دقیقه صبر
۰۹:۳۰  ۵۰٪  → ۱۵ دقیقه صبر
۰۹:۴۵  ۱۰۰٪
```

**نکته**: هر گام باید **به‌اندازه کافی طولانی** باشد تا متریک معنا پیدا کند. ۳۰ ثانیه کافی نیست.

### مقایسه و الگوهای مختلف

**جدول انتخاب:**

| اگر… | استراتژی |
|---|---|
| downtime مجاز است (dev/test) | Recreate |
| اکثر سرویس‌ها | Rolling |
| rollback فوری حیاتی | Blue-Green |
| ریسک باید محدود شود | ✅ Canary |
| نیاز به کنترل رفتار | + Feature Flag |

**Rolling در برابر Blue-Green:**

| معیار | Rolling | Blue-Green |
|---|---|---|
| منابع | ✅ ۱× | ❌ ۲× |
| سرعت deploy | متوسط | ✅ سریع |
| rollback | متوسط | ✅ فوری |
| تست پیش از سوییچ | ❌ | ✅ |

**Canary در برابر A/B:**

| معیار | Canary | A/B |
|---|---|---|
| هدف | کاهش ریسک | یادگیری |
| معیار | خطا، latency | رفتار کاربر |
| مدت | دقیقه | روز |
| تصمیم | ادامه/rollback | کدام بهتر |

**ترکیب توصیه‌شده:**

```
Canary + Feature Flag:
  - Canary: کاهش ریسک فنی
  - Flag: کنترل رفتار کسب‌وکار
  - نتیجه: دو لایه ایمنی
```

### اشکال شایع

**اشکال ۱: Canary بدون متریک**

«canary گذاشتیم» ولی نمی‌دانی سالم است. باید دستی چک کنی — که انجام نمی‌شود.

**اشکال ۲: Blue-Green بدون تست Green**

ترافیک را سوییچ کردی، Green خراب بود. باید قبل از سوییچ تست کنی.

**اشکال ۳: Rolling بدون health check**

instance جدید بالا نمی‌آید ولی ترافیک می‌گیرد. downtime.

**اشکال ۴: migration ناسازگار**

Blue و Green با schema یکسان کار نمی‌کنند. expand-contract لازم.

**اشکال ۵: استراتژی پیچیده برای سرویس کوچک**

Blue-Green برای یک سرویس داخلی با ۱۰ کاربر = overkill.

**اشکال ۶: فراموش کردن session**

کاربران در Blue لاگین کرده‌اند، سوییچ به Green = خروج. session باید مشترک باشد.

### نکات مهم (Tips & Tricks)

- **Canary + Flag** بهترین ترکیب.
- **Canary بدون متریک، بی‌فایده است.**
- **هر گام canary، حداقل ۱۵ دقیقه.**
- **health check درست** برای Rolling.
- **session مشترک** برای Blue-Green.
- **استراتژی را با مقیاس تطبیق بده** — overkill نکن.

### خلاصه فصل

- چهار استراتژی: Recreate، Rolling، Blue-Green، Canary.
- Canary کم‌ترین ریسک را دارد — اگر متریک باشد.
- Blue-Green سریع‌ترین rollback.
- Rolling پیش‌فرض sensible است.
- Canary + Flag، دو لایه ایمنی.
- استراتژی را با مقیاس تطبیق بده.

---

## فصل ۳۸: Feature Flag و مدیریت تغییر رفتار

### مقدمه

**Feature Flag** (یا Feature Toggle) ساده‌ترین و قدرتمندترین ابزار مهندسی انتشار است: کد را deploy کن، ولی رفتار را با یک کلید کنترل کن.

این فصل انواع flag، خطراتش و مدیریتش را می‌آورد.

### مفاهیم اساسی

**چهار نوع flag:**

| نوع | عمر | کاربرد |
|---|---|---|
| **Release Toggle** | روز-هفته | انتشار تدریجی قابلیت |
| **Experiment Toggle** | هفته-ماه | A/B testing |
| **Ops Toggle** | دائمی | خاموش کردن مسیر در حادثه |
| **Permission Toggle** | دائمی | قابلیت برای مشتری خاص |

**نکته**: هر نوع، **عمر متفاوت** دارد. Release Toggle باید حذف شود، Ops Toggle می‌ماند.

**سه سطح فعال‌سازی:**

| سطح | مثال |
|---|---|
| Boolean | روشن/خاموش |
| درصدی | ۱۰٪ کاربران |
| هدفمند | کاربران یک سازمان، یک منطقه |

**چرا flag مهم است:**

```
بدون flag:  deploy = تغییر رفتار برای همه
با flag:    deploy = کد روی سرور (رفتار بدون تغییر)

نتیجه: deploy می‌تواند هر روز باشد، تغییر رفتار کنترل‌شده
```

### مثال‌های عملی

**پیاده‌سازی ساده:**

```python
"""مدیریت feature flag با پشتیبانی درصدی و هدفمند."""
import hashlib
import os
from dataclasses import dataclass


@dataclass
class Flag:
    """تعریف یک feature flag."""
    name: str
    enabled: bool = False
    rollout_percent: int = 0
    allowed_users: frozenset[str] = frozenset()


FLAGS: dict[str, Flag] = {
    "new_checkout": Flag(name="new_checkout", enabled=True, rollout_percent=10),
    "beta_dashboard": Flag(
        name="beta_dashboard",
        enabled=True,
        allowed_users=frozenset({"alice", "bob"}),
    ),
}


def is_enabled(flag_name: str, user_id: str | None = None) -> bool:
    """آیا قابلیت برای این کاربر فعال است؟

    Args:
        flag_name: نام flag
        user_id: شناسه کاربر (برای rollout درصدی)

    Returns:
        True اگر فعال باشد
    """
    flag = FLAGS.get(flag_name)
    if flag is None or not flag.enabled:
        return False

    # کاربران صریحاً مجاز
    if user_id and user_id in flag.allowed_users:
        return True

    # rollout درصدی — پایدار برای هر کاربر
    if flag.rollout_percent > 0 and user_id:
        bucket = int(hashlib.sha256(f"{flag_name}:{user_id}".encode()).hexdigest(), 16)
        return (bucket % 100) < flag.rollout_percent

    return False
```

**نکته مهم**: استفاده از hash، باعث می‌شود یک کاربر همیشه در همان bucket باشد — تجربه پایدار.

**الگوی «خاموش کردن در حادثه»:**

```python
"""Ops toggle برای خاموش کردن مسیر پرخطر در حادثه."""
if is_enabled("use_cache_layer"):
    return read_from_cache(key)
return read_from_database(key)
```

در حادثه (مثلاً cache خراب است)، flag را خاموش کن — بدون deploy. **ثانیه‌ها به‌جای دقیقه‌ها.**

**چرخه حیات flag:**

```
۱. ساخت: flag با rollout 0٪
۲. deploy: کد روی production، رفتار عوض نشده
۳. dogfooding: تیم داخلی (allowed_users)
۴. canary: ۵٪ → ۲۵٪ → ۵۰٪
۵. کامل: ۱۰۰٪
۶. تثبیت: ۲ هفته بدون مشکل
۷. حذف: flag و کد قدیمی پاک شود
```

**نکته حیاتی**: مرحله ۷ را **فراموش نکن**. flag بدون حذف = بدهی فنی.

**مدیریت بدهی flag:**

```
قاعده: هر Release Toggle تاریخ انقضا دارد (حداکثر ۹۰ روز)
```

```python
# در کد
# TODO(2026-12-01): حذف flag new_checkout پس از rollout کامل
if is_enabled("new_checkout", user.id):
    ...
```

و یک job دوره‌ای که flagهای منقضی را گزارش کند.

### مقایسه و الگوهای مختلف

**Flag در برابر branch:**

| معیار | Flag | Branch |
|---|---|---|
| ریسک merge | ✅ صفر | ❌ تجمعی |
| تست | ✅ در production | ❌ جدا |
| پیچیدگی کد | ❌ بیشتر | ✅ کمتر |
| مناسب | انتشار تدریجی | توسعه |

**سه روش ذخیره flag:**

| روش | مناسب |
|---|---|
| متغیر محیطی | ✅ ساده، ops toggle |
| فایل config | متوسط |
| سرویس flag (LaunchDarkly و...) | ✅ مقیاس، UI |

**چهار ضدالگوی flag:**

| ضدالگو | مشکل |
|---|---|
| flag در flag | غیرقابل درک |
| flag بدون مستند | کسی نمی‌داند چه می‌کند |
| flag بدون انقضا | بدهی فنی |
| flag برای همه‌چیز | پیچیدگی انفجاری |

### اشکال شایع

**اشکال ۱: flag هرگز حذف نشد**

پس از ۲ سال، ۵۰ flag، کد غیرقابل درک. تاریخ انقضا بگذار.

**اشکال ۲: flag در چند جا چک می‌شود**

نتیجه ناسازگار. یک تابع مرکزی.

**اشکال ۳: flag بدون تست**

مسیر قدیم و جدید باید هر دو تست شوند. اگر flag خاموش باشد و مسیر قدیم خراب، متوجه نمی‌شوی.

**اشکال ۴: flag به‌عنوان جایگزین طراحی**

«به‌جای refactor، flag بگذاریم» — بدهی فنی انباشته.

**اشکال ۵: flag درصدی بدون پایداری**

`random.random() < 0.1` → هر درخواست متفاوت. کاربر تجربه ناسازگار دارد. از hash استفاده کن.

**اشکال ۶: flag در کد کلاینت بدون cache**

هر درخواست به سرور flag = latency. cache کن.

### نکات مهم (Tips & Tricks)

- **هر flag تاریخ انقضا** داشته باشد.
- **hash برای rollout درصدی** — نه random.
- **یک تابع مرکزی** برای چک کردن.
- **هر دو مسیر را تست کن.**
- **Ops Toggle را جدا نگه دار** — عمر متفاوت.
- **job گزارش flagهای منقضی.**
- **flag را در کد کامنت کن** — چرا و کِی حذف.

### خلاصه فصل

- چهار نوع flag: Release، Experiment، Ops، Permission.
- flag کد را از رفتار جدا می‌کند.
- rollout درصدی با hash، نه random.
- چرخه هفت‌مرحله‌ای با حذف در انتها.
- هر flag تاریخ انقضا دارد.
- flag بدون حذف = بدهی فنی.

---

## فصل ۳۹: نسخه‌گذاری و مدیریت Artifact

### مقدمه

بدون نسخه‌گذاری درست، نمی‌دانی چه چیزی روی production است، نمی‌توانی rollback کنی، و نمی‌توانی باگ را ردیابی کنی.

این فصل اصول نسخه‌گذاری و مدیریت artifact را می‌آورد.

### مفاهیم اساسی

**سه نوع نسخه:**

| نوع | مثال | تغییرپذیر |
|---|---|---|
| **SemVer** | `v1.4.2` | ❌ (باید ثابت باشد) |
| **CalVer** | `2026.10.1` | ❌ |
| **Build number** | `#1892` | ❌ |
| **Tag mutable** | `latest`، `stable` | ✅ |

**SemVer — سه عدد:**

```
MAJOR.MINOR.PATCH
  │      │     └── رفع باگ (سازگار)
  │      └──────── قابلیت جدید (سازگار)
  └─────────────── تغییر ناسازگار
```

**قاعده**: تغییر ناسازگار در MAJOR. اگر مشتری روی API تو کد نوشته، نباید بشکند.

**«Artifact Immutable» — اصل بنیادی:**

```
❌ tag را بازنویسی نکن
✅ نسخه جدید بساز
```

اگر `v1.4.2` را بازنویسی کنی:
- staging و production ممکن است متفاوت باشند
- rollback به نسخه اشتباه می‌رود
- audit غیرممکن می‌شود

**Digest — دقیق‌ترین ارجاع:**

```
tag:    app:v1.4.2
digest: app@sha256:9f8e7d6c5b4a...
```

digest به **محتوای دقیق** اشاره می‌کند. حتی اگر tag بازنویسی شود، digest همان است.

### مثال‌های عملی

**استراتژی تگ‌گذاری کامل:**

| موقعیت | تگ‌ها | تغییرپذیر |
|---|---|---|
| هر build | `sha-a1b2c3d` | ❌ |
| main | + `main` | ✅ |
| tag | + `v1.4.2`، `stable` | ❌ / ✅ |
| nightly | `nightly-20261017` | ❌ |

**قاعده**: deploy همیشه با **تگ immutable** یا **digest**.

**ذخیره metadata همراه artifact:**

```json
{
  "version": "v1.4.2",
  "commit": "a1b2c3d4e5f6...",
  "branch": "main",
  "build_number": 1892,
  "built_at": "2026-10-17T09:30:00Z",
  "built_by": "pipeline-1892",
  "digest": "sha256:9f8e7d6c...",
  "base_image": "python:3.12.7-slim@sha256:abc...",
  "dependencies_hash": "sha256:def..."
}
```

**این فایل، پاسخ همه سؤالات audit است.**

**مدیریت نسخه در چند محیط:**

```
staging:    v1.4.2 (digest sha256:9f8e...)
production: v1.4.1 (digest sha256:7c6b...)

سؤال: "چرا staging و production متفاوت‌اند؟"
پاسخ: ✅ "v1.4.2 هنوز به production نرفته"

بدون نسخه‌گذاری:
پاسخ: ❌ "نمی‌دانم"
```

**پاک‌سازی نسخه‌های قدیمی — سیاست:**

| نسخه | نگه‌داری |
|---|---|
| ۱۰ نسخه آخر | ✅ همیشه |
| releaseها (v*.*.*) | ✅ ۱ سال |
| `latest`، `stable` | ✅ همیشه |
| sha-* قدیمی‌تر از ۳۰ روز | ❌ پاک |

**نکته**: قبل از پاک‌سازی، مطمئن شو که rollback به نسخه‌های لازم ممکن است.

### مقایسه و الگوهای مختلف

**SemVer در برابر CalVer:**

| معیار | SemVer | CalVer |
|---|---|---|
| معنا | سازگاری | زمان |
| مناسب | کتابخانه، API | محصول SaaS |
| مثال | v2.1.0 | 2026.10.1 |

**Tag در برابر Digest:**

| معیار | Tag | Digest |
|---|---|---|
| خوانایی | ✅ | ❌ |
| تغییرناپذیری | ⚠️ | ✅✅ |
| مناسب انسان | ✅ | ❌ |
| مناسب deploy | ⚠️ | ✅✅ |

**توصیه**: tag برای انسان، digest برای ماشین.

**سه سطح بلوغ نسخه‌گذاری:**

| سطح | ویژگی |
|---|---|
| ۱ | `latest`، بدون نسخه |
| ۲ | SemVer، تگ immutable |
| ۳ | ✅ + digest + metadata |

### اشکال شایع

**اشکال ۱: `latest` در deploy**

غیرقابل تکرار. rollback ممکن نیست. همیشه نسخه دقیق.

**اشکال ۲: بازنویسی tag**

`v1.4.2` دو بار build شد. کدام روی production است؟

**اشکال ۳: بدون metadata**

نمی‌دانی artifact از کدام commit ساخته شد.

**اشکال ۴: نسخه‌گذاری دستی**

«یادم رفت نسخه را عوض کنم» → دو نسخه متفاوت با یک شماره.

**اشکال ۵: پاک کردن نسخه‌های لازم**

rollback به نسخه‌ای که پاک شده. سیاست پاک‌سازی با rollback هم‌راستا باشد.

**اشکال ۶: base image بدون pin**

`python:latest` → build دوباره، نتیجه متفاوت.

### نکات مهم (Tips & Tricks)

- **Tag immutable + digest.**
- **metadata همراه artifact.**
- **SemVer برای API**، CalVer برای محصول.
- **`latest` فقط برای انسان**، نه deploy.
- **سیاست پاک‌سازی با rollback هم‌راستا.**
- **base image با digest pin.**

### خلاصه فصل

- سه نوع نسخه: SemVer، CalVer، build number.
- Artifact باید immutable باشد.
- digest دقیق‌ترین ارجاع است.
- metadata پاسخ همه audit است.
- tag برای انسان، digest برای ماشین.
- پاک‌سازی با rollback هم‌راستا.

---

## فصل ۴۰: Rollback و مدیریت شکست

### مقدمه

هر deploy می‌تواند شکست بخورد. سؤال این نیست «چطور جلوگیری کنم»، بلکه «چطور در ۲ دقیقه برگردم».

این فصل rollback را به یک **قابلیت تمرین‌شده** تبدیل می‌کند، نه یک آرزو.

### مفاهیم اساسی

**چهار نوع rollback:**

| نوع | سرعت | روش |
|---|---|---|
| **Redeploy نسخه قبل** | ۵-۱۵ دقیقه | pipeline با نسخه قبلی |
| **Platform undo** | ✅ ۳۰ ثانیه | `kubectl rollout undo` |
| **Traffic switch** | ✅ ۳۰ ثانیه | Blue-Green |
| **Feature flag off** | ✅✅ ۰ ثانیه | خاموش کردن |

**قاعده طلایی**: **اول rollback، بعد دیباگ.**

```
❌ بد: "بگذار بفهمم چرا" (در حالی که سرویس خراب است)
✅ خوب: "rollback کن، بعد با آرامش دیباگ کن"
```

کاربران نباید منتظر بمانند تا تو بفهمی.

**سه شرط rollback مؤثر:**

| شرط | چرا |
|---|---|
| **artifact قبلی موجود** | بدون آن، rollback ممکن نیست |
| **migration سازگار** | بدون آن، rollback داده را می‌شکند |
| **تمرین‌شده** | بدون آن، در حادثه فراموش می‌کنی |

**مهم‌ترین: migration سازگار.**

اگر migration ستونی را حذف کند، نسخه قدیم نمی‌تواند کار کند. راه‌حل: **expand-contract**.

### مثال‌های عملی

**Expand-Contract — سه deploy:**

```
Deploy ۱ (expand):
  - ستون جدید `email_verified` اضافه شود (nullable)
  - کد جدید هم `email` و هم `email_verified` را می‌نویسد
  - کد قدیم فقط `email` (کار می‌کند)

Deploy ۲ (migrate):
  - داده قدیمی پر شود
  - کد جدید فقط `email_verified` بخواند

Deploy ۳ (contract):
  - ستون `email` حذف شود
  - کد قدیم دیگر کار نمی‌کند — ولی ما در نسخه جدید هستیم
```

**در هر لحظه، rollback به نسخه قبلی ممکن است** — چون schema سازگار است.

**rollback خودکار بر اساس متریک:**

```python
"""rollback خودکار اگر نرخ خطا بالا رفت."""
import time


ERROR_THRESHOLD = 0.01       # ۱٪
CHECK_DURATION = 300         # ۵ دقیقه


def deploy_with_auto_rollback(deploy_fn, rollback_fn, metric_fn) -> bool:
    """استقرار با بررسی خودکار و rollback.

    Returns:
        True اگر موفق، False اگر rollback شد
    """
    deploy_fn()

    for _ in range(CHECK_DURATION // 30):
        time.sleep(30)
        error_rate = metric_fn()

        if error_rate > ERROR_THRESHOLD:
            print(f"❌ نرخ خطا {error_rate:.2%} — rollback")
            rollback_fn()
            return False

    print("✅ سالم")
    return True
```

**نکته**: این الگو باید **خودکار** باشد، نه دستی. در ساعت ۳ بامداد، کسی نیست.

**تمرین rollback — الزامی:**

```
هر ماه:
  ۱. در staging، نسخه جدید deploy کن
  ۲. rollback کن
  ۳. زمان بگیر
  ۴. مستند کن

هدف: < ۵ دقیقه، بدون مشکل
```

**rollback تمرین‌نشده، rollback نیست.**

**پس از rollback — پنج گام:**

| گام | هدف |
|---|---|
| ۱. تأیید سلامت | سرویس برگشت؟ |
| ۲. اطلاع‌رسانی | تیم و ذی‌نفعان بدانند |
| ۳. جمع‌آوری شواهد | لاگ، متریک، نسخه |
| ۴. ریشه‌یابی | با آرامش، بدون فشار |
| ۵. postmortem | یادگیری |

**نکته**: شواهد را **قبل** از پاک کردن محیط جمع کن.

### مقایسه و الگوهای مختلف

**rollback در برابر roll-forward:**

| معیار | Rollback | Roll-forward |
|---|---|---|
| سرعت | ✅ سریع | ❌ کند |
| ریسک | ✅ کم | ❌ بیشتر |
| مناسب | حادثه | باگ کوچک با fix آماده |

**قاعده**: در حادثه، rollback. roll-forward فقط اگر fix در ۵ دقیقه آماده است.

**سه سطح rollback:**

| سطح | زمان | روش |
|---|---|---|
| Code | ۳۰ ثانیه | flag off |
| Deployment | ۳۰ ثانیه | platform undo |
| Data | ❌ ساعت‌ها | restore backup |

**نکته**: rollback داده، **همیشه** گران است. برای همین migration سازگار حیاتی است.

**Rollback در برابر Failover:**

| معیار | Rollback | Failover |
|---|---|---|
| هدف | برگشت نسخه | تغییر مسیر |
| trigger | باگ | خرابی |
| مناسب | deploy | زیرساخت |

### اشکال شایع

**اشکال ۱: artifact قبلی پاک شده**

rollback ممکن نیست. سیاست نگه‌داری با rollback هم‌راستا.

**اشکال ۲: migration برگشت‌ناپذیر**

ستون حذف شد، نسخه قدیم کار نمی‌کند. expand-contract.

**اشکال ۳: rollback هرگز تمرین نشده**

در حادثه، دستور را فراموش می‌کنی. ماهانه تمرین کن.

**اشکال ۴: دیباگ قبل از rollback**

سرویس ۳۰ دقیقه خراب می‌ماند. اول rollback.

**اشکال ۵: rollback بدون اطلاع‌رسانی**

تیم دیگر فکر می‌کند deploy موفق بود.

**اشکال ۶: rollback بدون جمع‌آوری شواهد**

محیط پاک شد، ریشه‌یابی غیرممکن.

**اشکال ۷: rollback دستی در ساعت ۳ بامداد**

خودکار کن. آستانه و trigger مشخص.

### نکات مهم (Tips & Tricks)

- **اول rollback، بعد دیباگ.**
- **migration با expand-contract** — شرط لازم.
- **rollback خودکار** با آستانه متریک.
- **ماهانه تمرین کن.**
- **شواهد را قبل از پاک کردن جمع کن.**
- **artifact قبلی را نگه دار.**
- **flag off سریع‌ترین rollback است.**

### خلاصه فصل

- چهار نوع rollback؛ flag off سریع‌ترین.
- اول rollback، بعد دیباگ.
- سه شرط: artifact، migration سازگار، تمرین.
- expand-contract برای migration.
- rollback خودکار با آستانه.
- تمرین ماهانه الزامی است.

---

## منابع و مراجع

**کتاب‌ها:**
- [Continuous Delivery](https://continuousdelivery.com/) — Humble, Farley
- [Release It!](https://pragprog.com/titles/mnee2/release-it-second-edition/) — Michael Nygard
- [The DevOps Handbook](https://itrevolution.com/product/the-devops-handbook/)

**مقالات:**
- [Martin Fowler: Feature Toggles](https://martinfowler.com/articles/feature-toggles.html)
- [Martin Fowler: Parallel Change](https://martinfowler.com/bliki/ParallelChange.html)
- [Martin Fowler: Blue-Green Deployment](https://martinfowler.com/bliki/BlueGreenDeployment.html)
- [Martin Fowler: Canary Release](https://martinfowler.com/bliki/CanaryRelease.html)

**مستندات:**
- [Semantic Versioning](https://semver.org/)
- [Google SRE: Release Engineering](https://sre.google/sre-book/release-engineering/)

---

**بعدی:** [بخش ۰۸: زیرساخت به‌عنوان کد](08-Infrastructure-as-Code.md)
