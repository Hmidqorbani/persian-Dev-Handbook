# بخش ۲۱: بهترین روش‌ها، چک‌لیست و واژه‌نامه

**مدت یادگیری**: ۲ تا ۳ ساعت
**سطح سختی**: همه سطوح
**پیش‌نیاز**:
- بخش ۰۱ تا ۲۰

---
## فهرست محتوا
- [فصل ۱۱۵: چک‌لیست تیم](#فصل-۱۱۵-چکلیست-تیم)
- [فصل ۱۱۶: چک‌لیست سازمان](#فصل-۱۱۶-چکلیست-سازمان)
- [فصل ۱۱۷: چک‌لیست SRE و پایداری](#فصل-۱۱۷-چکلیست-sre-و-پایداری)
- [فصل ۱۱۸: واژه‌نامه جامع](#فصل-۱۱۸-واژهنامه-جامع)
- [فصل ۱۱۹: تمرین‌های عملی](#فصل-۱۱۹-تمرینهای-عملی)
- [فصل ۱۲۰: مسیر یادگیری](#فصل-۱۲۰-مسیر-یادگیری)
- [فصل ۱۲۱: منابع نهایی](#فصل-۱۲۱-منابع-نهایی)

---

## فصل ۱۱۵: چک‌لیست تیم

### مقدمه

این چک‌لیست‌ها برای **مراجعه سریع** هستند. هر مورد از یک درد واقعی آمده.

### چک‌لیست روزانه تیم

**جریان:**
```
[ ] WIP زیر حد تعیین‌شده است
[ ] هیچ کاری بیش از ۳ روز در جریان نیست
[ ] هیچ MRی بیش از ۲۴ ساعت منتظر review نیست
[ ] هیچ آیتمی «بلوکه» بدون اقدام نیست
```

**کیفیت:**
```
[ ] main سبز است
[ ] هیچ تست flaky در pipeline اصلی نیست
[ ] هیچ هشدار کاذب در ۲۴ ساعت گذشته نبوده
```

### چک‌لیست هفتگی

```
[ ] retro برگزار شد، با حداکثر ۲ اقدام
[ ] متریک‌های جریان بررسی شد (WIP، Cycle Time)
[ ] یافته‌های امنیتی جدید triage شد
[ ] اقدامات هفته قبل مرور شد
[ ] دانش جدید مستند شد (اگر بود)
```

### چک‌لیست ماهانه

```
[ ] متریک‌های DORA بررسی شد
[ ] روند نسبت به ماه قبل
[ ] یک اقدام برای ماه بعد (با مالک)
[ ] postmortem‌های ماه مرور شد
[ ] هزینه ابر بررسی شد
[ ] چک‌لیست دسترسی‌ها (کسی نرفته؟)
```

### چک‌لیست سرویس جدید

```
مخزن:
[ ] README با دستور اجرا
[ ] .gitignore و .dockerignore
[ ] CODEOWNERS
[ ] CONTRIBUTING.md

CI/CD:
[ ] pipeline با lint + test
[ ] cache تنظیم‌شده
[ ] artifact با expire_in
[ ] deploy به staging خودکار

کیفیت:
[ ] تست واحد (پوشش ≥ ۷۰٪)
[ ] تست integration (اگر لازم)
[ ] تست در staging

امنیت:
[ ] SAST در pipeline
[ ] Dependency scanning
[ ] Secret detection
[ ] هیچ رازی در مخزن

عملیات:
[ ] health endpoint
[ ] متریک (چهار سیگنال)
[ ] لاگ ساختاریافته
[ ] alert پایه
[ ] runbook

مالکیت:
[ ] مالک مشخص
[ ] on-call تعیین‌شده
[ ] SLO (اگر حیاتی)
```

### چک‌لیست MR

```
نویسنده:
[ ] MR زیر ۴۰۰ خط
[ ] توضیح کامل (چه، چرا، چطور، ریسک)
[ ] Closes #issue
[ ] self-review انجام شد
[ ] pipeline سبز
[ ] تست اضافه شد

reviewer:
[ ] آیا مشکل درست را حل می‌کند؟
[ ] آیا تست کافی است؟
[ ] آیا خطاها مدیریت شده؟
[ ] آیا امنیت رعایت شده؟
[ ] آیا سازگاری عقب‌رو حفظ شده؟
```

### چک‌لیست release

```
قبل:
[ ] artifact در registry، digest ثبت
[ ] migration backward-compatible
[ ] rollback تست‌شده
[ ] feature flag تنظیم‌شده
[ ] تیم در دسترس

در حین:
[ ] canary/rolling با متریک
[ ] smoke test
[ ] بررسی نرخ خطا

بعد:
[ ] تأیید سلامت
[ ] changelog به‌روز
[ ] اطلاع‌رسانی
[ ] پاک‌سازی منابع قدیمی
```

---

## فصل ۱۱۶: چک‌لیست سازمان

### چک‌لیست ساختار

```
[ ] تیم‌ها ۵-۹ نفره
[ ] هر تیم، دامنه روشن (نه فناوری)
[ ] هر سرویس، یک مالک
[ ] تیم‌ها end-to-end (build تا run)
[ ] حالت تعامل تعریف‌شده (Service/Collaboration)
[ ] Platform team وجود دارد (اگر ۵+ تیم)
[ ] Enabling team موقت است
```

### چک‌لیست انگیزه

```
[ ] KPI تیم‌ها متناقض نیست
[ ] KPI مشترک برای Dev و Ops
[ ] پاداش برای انتقال دانش (نه فقط حل)
[ ] پاداش برای گزارش خطا
[ ] پاداش برای بهبود (نه فقط تحویل)
[ ] بودجه تیمی (نه فقط متمرکز)
```

### چک‌لیست دسترسی و اختیار

```
[ ] تیم می‌تواند بدون تأیید deploy کند
[ ] تیم می‌تواند محیط بسازد (self-service)
[ ] تیم می‌تواند ابزار انتخاب کند (در محدوده)
[ ] تعداد تأییدها حداقل (۱-۲)
[ ] هیچ تصمیم فنی نیاز به مدیر ندارد
```

### چک‌لیست اندازه‌گیری

```
[ ] Lead Time اندازه‌گیری می‌شود
[ ] Deploy Frequency اندازه‌گیری می‌شود
[ ] CFR اندازه‌گیری می‌شود
[ ] MTTR اندازه‌گیری می‌شود
[ ] امتیاز فرهنگ سنجیده می‌شود
[ ] گزارش ماهانه به مدیریت
[ ] متریک برای بهبود، نه ارزیابی
```

### چک‌لیست امنیت و انطباق

```
[ ] 2FA اجباری برای Maintainer+
[ ] عضویت با تاریخ انقضا
[ ] audit events فعال
[ ] secret detection در CI
[ ] security gate برای Critical
[ ] دسترسی‌ها فصلی بازبینی می‌شوند
[ ] SBOM تولید می‌شود
[ ] base image با digest
```

### چک‌لیست پایداری تغییر

```
[ ] تغییر در onboarding
[ ] تغییر در Definition of Done
[ ] تغییر در ارزیابی عملکرد
[ ] تغییر در ابزار (guardrail)
[ ] مالک برای هر عنصر
[ ] متریک ماهانه ادامه دارد
[ ] نشانه‌های رگرسیون پایش می‌شود
```

---

## فصل ۱۱۷: چک‌لیست SRE و پایداری

### چک‌لیست سرویس حیاتی

```
SLO:
[ ] SLO تعریف‌شده (availability + latency)
[ ] SLI از دید کاربر
[ ] Error budget محاسبه می‌شود
[ ] burn rate alert تنظیم‌شده
[ ] سیاست توقف انتشار توافق‌شده

Observability:
[ ] چهار سیگنال طلایی
[ ] داشبورد با مالک
[ ] لاگ ساختاریافته با request_id
[ ] تریس (اگر توزیع‌شده)
[ ] alert فقط برای اقدام فوری

عملیات:
[ ] runbook برای هر alert
[ ] on-call با rotation
[ ] escalation policy
[ ] game day فصلی
[ ] postmortem در ۴۸ ساعت
```

### چک‌لیست حادثه

```
در حادثه:
[ ] تأیید واقعی بودن
[ ] IC تعیین شد
[ ] کانال جدا برای دیباگ
[ ] به‌روزرسانی هر ۵ دقیقه
[ ] کاهش اثر قبل از ریشه‌یابی
[ ] اطلاع‌رسانی به کاربر (اگر لازم)

بعد از حادثه:
[ ] postmortem در ۴۸ ساعت
[ ] خط زمانی ثبت
[ ] علت ریشه‌ای (۳ چرا)
[ ] اقدام با مالک و مهلت
[ ] «چه خوب پیش رفت» نوشته شد
[ ] جلسه مرور برگزار شد
```

### چک‌لیست toil

```
[ ] toil اندازه‌گیری می‌شود (درصد ماهانه)
[ ] toil < ۵۰٪
[ ] لیست کارهای تکراری
[ ] اولویت‌بندی با ROI
[ ] خودکارسازی در جریان
[ ] self-service برای درخواست‌های رایج
```

### چک‌لیست ظرفیت

```
[ ] utilization پایش می‌شود
[ ] headroom ≥ ۲۰٪
[ ] ظرفیت برای ۶ ماه آینده برنامه‌ریزی شده
[ ] load test سالانه
[ ] soak test برای نشت حافظه
[ ] گلوگاه وابستگی شناسایی شده
```

---

## فصل ۱۱۸: واژه‌نامه جامع

### مقدمه

اصطلاحات کلیدی DevOps، به ترتیب الفبای انگلیسی، با معادل فارسی و توضیح کوتاه.

### واژه‌نامه

| اصطلاح | فارسی | توضیح |
|---|---|---|
| **Agile** | چابک | روش کار تکراری و افزایشی |
| **Alert fatigue** | خستگی هشدار | بی‌توجهی به‌خاطر هشدار زیاد |
| **Artifact** | خروجی ساخت | فایل تولیدشده از build (ایمیج، بسته) |
| **Automation** | خودکارسازی | حذف کار دستی تکراری |
| **Availability** | دسترس‌پذیری | درصد زمانی که سرویس کار می‌کند |
| **Blameless** | بدون سرزنش | بررسی حادثه بدون یافتن مقصر |
| **Blue-Green** | آبی-سبز | دو محیط کامل، سوییچ ترافیک |
| **Build** | ساخت | تبدیل کد به artifact |
| **Burn rate** | نرخ مصرف | سرعت مصرف بودجه خطا |
| **Bus factor** | ضریب اتوبوس | چند نفر باید بروند تا پروژه متوقف شود |
| **CALMS** | — | Culture، Automation، Lean، Measurement، Sharing |
| **Canary** | قناری | انتشار به درصدی از کاربران |
| **Capacity** | ظرفیت | حداکثر بار قابل تحمل |
| **CFR** | نرخ شکست تغییر | درصد deployهایی که حادثه می‌سازند |
| **Chaos engineering** | مهندسی آشوب | تزریق خرابی برای آزمودن مقاومت |
| **CI** | یکپارچه‌سازی پیوسته | ادغام مکرر با تست خودکار |
| **CD (Delivery)** | تحویل پیوسته | همیشه آماده انتشار، با تأیید |
| **CD (Deployment)** | استقرار پیوسته | انتشار خودکار |
| **Cognitive load** | بار شناختی | حجم ذهنی لازم برای کار |
| **Collaboration** | همکاری | حالت تعامل نزدیک بین تیم‌ها |
| **Complicated subsystem** | زیرسیستم پیچیده | تیمی برای بخش نیازمند تخصص عمیق |
| **Conway's Law** | قانون کانوی | معماری آینه ساختار سازمان است |
| **Cycle time** | زمان چرخه | از شروع تا پایان یک آیتم |
| **DAG** | گراف جهت‌دار | مدل وابستگی jobها |
| **Deployment** | استقرار | نصب نسخه روی محیط |
| **DevOps** | — | فرهنگ و روش تحویل سریع و پایدار |
| **DevSecOps** | — | DevOps با امنیت جاری |
| **DORA** | — | چهار متریک عملکرد تحویل |
| **Drift** | انحراف | تفاوت بین کد و واقعیت |
| **Dwell time** | زمان انتظار | زمان در صف |
| **Enabling team** | تیم توانمندساز | تیم موقت انتقال مهارت |
| **Error budget** | بودجه خطا | ۱۰۰٪ منهای SLO |
| **Escalation** | ارجاع | انتقال به سطح بالاتر |
| **Feature flag** | کلید قابلیت | خاموش/روشن کردن رفتار بدون deploy |
| **Flow** | جریان | حرکت روان کار |
| **Flow efficiency** | بازده جریان | نسبت زمان ارزش‌افزا به کل |
| **Gameday** | روز بازی | تمرین حادثه |
| **Golden path** | مسیر طلایی | راه توصیه‌شده و آسان |
| **Golden signals** | سیگنال‌های طلایی | latency، traffic، errors، saturation |
| **Goodhart's Law** | قانون گودهارت | معیاری که هدف شود، خراب می‌شود |
| **Guardrail** | نرده محافظ | مسیر امن، بدون بلاک |
| **IaC** | زیرساخت به‌عنوان کد | تعریف زیرساخت در کد |
| **IDP** | پلتفرم توسعه داخلی | لایه انتزاع برای توسعه‌دهنده |
| **Idempotent** | خودتکرار | اجرای دوباره، نتیجه یکسان |
| **Immutable infrastructure** | زیرساخت تغییرناپذیر | جایگزینی، نه تغییر |
| **Incident** | حادثه | اختلال در سرویس |
| **Incident Commander** | فرمانده حادثه | مسئول هماهنگی حادثه |
| **Just culture** | فرهنگ منصفانه | مسئولیت + یادگیری |
| **Kaizen** | بهبود مستمر | بهبود کوچک روزانه |
| **Kanban** | — | سیستم جریان با محدودیت WIP |
| **Latency** | تأخیر | زمان پاسخ |
| **Lead time** | زمان تحویل | از commit تا production |
| **Lean** | ناب | حذف اتلاف |
| **Little's Law** | قانون لیتل | Cycle Time = WIP / Throughput |
| **MTTR** | زمان بازیابی | میانگین زمان رفع حادثه |
| **MVP** | حداقل محصول قابل استفاده | کوچک‌ترین نسخه ارزشمند |
| **Observability** | قابلیت مشاهده | توانایی پاسخ به سؤال‌های جدید |
| **On-call** | کشیک | آماده‌باش برای حادثه |
| **OPA** | — | موتور سیاست (Open Policy Agent) |
| **Orchestration** | ارکستراسیون | هماهنگی خودکار سرویس‌ها |
| **Paved road** | جاده آسفالت | همان Golden Path |
| **Platform team** | تیم پلتفرم | تیم سرویس‌دهنده به تیم‌های محصول |
| **Postmortem** | بررسی پس از حادثه | تحلیل بدون سرزنش |
| **Psychological safety** | ایمنی روانی | باور به امن بودن سؤال/خطا |
| **Pull request** | درخواست ادغام | معادل Merge Request |
| **Queue** | صف | آیتم‌های منتظر |
| **Rollback** | بازگشت | برگشت به نسخه قبل |
| **Runbook** | دفترچه اجرا | دستور گام‌به‌گام برای عملیات |
| **SLA** | توافق سطح سرویس | تعهد قراردادی |
| **SLI** | شاخص سطح سرویس | اندازه‌گیری کمی |
| **SLO** | هدف سطح سرویس | هدف داخلی |
| **SBOM** | فهرست مواد نرم‌افزاری | لیست اجزای ایمیج |
| **Shift left** | انتقال به چپ | کشف زودتر |
| **SRE** | مهندسی قابلیت اطمینان | مهندسی عملیات |
| **Stream-aligned team** | تیم هم‌راستا با جریان | تیم مالک یک جریان ارزش |
| **Technical debt** | بدهی فنی | هزینه آینده تصمیم امروز |
| **Team Topologies** | — | مدل چهار نوع تیم |
| **Throughput** | توان عبور | تعداد تحویل در بازه |
| **Toil** | کار طاقت‌فرسا | کار دستی تکراری بی‌ارزش |
| **Trunk-based** | مبتنی بر تنه | کار مستقیم روی main |
| **Value stream** | جریان ارزش | زنجیره فعالیت‌های ارزش‌ساز |
| **Vanity metric** | متریک خودنمایی | عدد خوب، بی‌معنا |
| **WIP** | کار در جریان | آیتم‌های نیمه‌تمام |
| **X-as-a-Service** | — | حالت تعامل سرویس‌محور |

---

## فصل ۱۱۹: تمرین‌های عملی

### مقدمه

یادگیری بدون تمرین، فراموش می‌شود. این تمرین‌ها برای **تثبیت** هستند.

### تمرین‌های فاز ۱ (مفاهیم)

**تمرین ۱: محاسبه هزینه شکاف**

```
در تیم خودت:
۱. تعداد نفرات را بشمار
۲. تخمین بزن چند درصد وقت صرف هماهنگی می‌شود
۳. هزینه ماهانه را حساب کن
۴. یک راه‌حل پیشنهاد بده با ROI
```

**تمرین ۲: نقشه جریان ارزش**

```
۱. یک قابلیت اخیر را انتخاب کن
۲. مسیرش را از ایده تا production بنویس
۳. زمان هر مرحله را تخمین بزن
۴. Flow Efficiency را حساب کن
۵. گلوگاه را شناسایی کن
```

**تمرین ۳: سه راه**

```
برای یک مشکل واقعی تیم:
۱. راه اول (جریان) چه راه‌حلی دارد؟
۲. راه دوم (بازخورد) چه؟
۳. راه سوم (یادگیری) چه؟
```

### تمرین‌های فاز ۲ (CI/CD)

**تمرین ۴: محاسبه متریک‌ها**

```
۱. Lead Time ۱۰ MR اخیر را حساب کن
۲. Deploy Frequency ماه گذشته را بشمار
۳. CFR را حساب کن
۴. MTTR را حساب کن
۵. با DORA مقایسه کن
```

**تمرین ۵: تحلیل pipeline**

```
۱. زمان هر مرحله pipeline را بنویس
۲. کدام مرحله بیشترین زمان را می‌گیرد؟
۳. کدام مرحله بیشترین flaky دارد؟
۴. کدام مرحله کسی نمی‌داند چرا هست؟
۵. یک اقدام برای هر مورد
```

**تمرین ۶: طراحی Golden Path**

```
۱. رایج‌ترین نوع سرویس در سازمانت را انتخاب کن
۲. مراحل ساخت آن را بنویس (دستی)
۳. کدام مراحل قابل خودکارسازی است؟
۴. Golden Path را طراحی کن
۵. Time to First Deploy را تخمین بزن
```

### تمرین‌های فاز ۳ (SRE)

**تمرین ۷: تعریف SLO**

```
برای یک سرویس:
۱. SLI انتخاب کن (availability? latency?)
۲. از دید کاربر بنویس
۳. SLO پیشنهاد بده
۴. downtime مجاز را حساب کن
۵. آیا واقع‌بینانه است؟
```

**تمرین ۸: تحلیل حادثه**

```
۱. آخرین حادثه را انتخاب کن
۲. خط زمانی بنویس
۳. سه «چرا» بپرس
۴. اقدام با مالک تعریف کن
۵. آیا postmortem فعلی کافی بود؟
```

**تمرین ۹: اندازه‌گیری toil**

```
۱. یک هفته کارهای تکراری را ثبت کن
۲. ساعت‌ها را جمع بزن
۳. درصد toil را حساب کن
۴. سه مورد با بهترین ROI را انتخاب کن
۵. یک مورد را خودکار کن
```

### تمرین‌های فاز ۴ (سازمان)

**تمرین ۱۰: نقشه تعامل**

```
۱. تیم‌های سازمانت را بنویس
۲. بین هر جفت، حالت تعامل فعلی چیست؟
۳. کدام Collaboration باید Service شود؟
۴. کدام Service باید Collaboration شود؟
```

**تمرین ۱۱: بازطراحی مرز**

```
۱. یک تیم بزرگ (۱۰+) انتخاب کن
۲. بر اساس دامنه تقسیم کن (نه فناوری)
۳. مرزها را بنویس
۴. چه چیزی سخت می‌شود؟
```

**تمرین ۱۲: نقشه راه ۹۰ روزه**

```
۱. سطح بلوغ فعلی را ارزیابی کن
۲. سه گلوگاه اصلی را بنویس
۳. برای هر ماه یک خروجی تعیین کن
۴. متریک موفقیت را بنویس
۵. چه کسی مالک است؟
```

---

## فصل ۱۲۰: مسیر یادگیری

### مقدمه

این کتابچه پایه را داد. این فصل مسیر ادامه را می‌گوید — بر اساس نقشی که می‌خواهی.

### مسیر ۱: مهندس DevOps

| مرحله | چه چیزی |
|---|---|
| ۱ | CI/CD عمیق (pipeline، cache، artifact) |
| ۲ | Docker و Kubernetes |
| ۳ | IaC (Terraform) |
| ۴ | Observability (Prometheus، Grafana) |
| ۵ | GitOps (ArgoCD) |

**پروژه تمرینی**: pipeline کامل برای یک سرویس + deploy روی K8s.

### مسیر ۲: مهندس SRE

| مرحله | چه چیزی |
|---|---|
| ۱ | SLO و Error Budget عمیق |
| ۲ | Observability (سه ستون) |
| ۳ | Incident management |
| ۴ | Chaos engineering |
| ۵ | ظرفیت و کارایی |

**پروژه تمرینی**: تعریف SLO برای سه سرویس + dashboard + runbook.

### مسیر ۳: مهندس پلتفرم

| مرحله | چه چیزی |
|---|---|
| ۱ | Team Topologies |
| ۲ | Backstage / IDP |
| ۳ | Golden path |
| ۴ | Self-service |
| ۵ | سنجش پلتفرم |

**پروژه تمرینی**: یک golden path برای یک نوع سرویس.

### مسیر ۴: DevSecOps

| مرحله | چه چیزی |
|---|---|
| ۱ | OWASP Top 10 |
| ۲ | SAST/DAST عمیق |
| ۳ | Supply chain (SBOM، signing) |
| ۴ | Threat modeling |
| ۵ | Compliance |

**پروژه تمرینی**: security gate با allowlist برای یک پروژه.

### مسیر ۵: رهبر فنی

| مرحله | چه چیزی |
|---|---|
| ۱ | DORA و سنجش |
| ۲ | Team Topologies |
| ۳ | مدیریت تغییر |
| ۴ | ADR و تصمیم‌گیری |
| ۵ | Mentoring |

**پروژه تمرینی**: نقشه راه ۹۰ روزه برای یک تیم.

### منابع کلیدی هر مسیر

| موضوع | منبع |
|---|---|
| CI/CD | Continuous Delivery (Humble, Farley) |
| K8s | Kubernetes Up & Running |
| SRE | Google SRE Books (رایگان) |
| Platform | Team Topologies |
| امنیت | OWASP ASVS، NIST SSDF |
| سازمان | Accelerate |

### گام بعدی فوری — سه کار

```
۱. یکی از ۱۲ تمرین فصل ۱۱۹ را انجام بده
۲. Lead Time تیمت را اندازه بگیر
۳. یک اقدام کوچک شروع کن (این هفته)
```

**نکته**: یادگیری با عمل تثبیت می‌شود، نه با خواندن.

---

## فصل ۱۲۱: منابع نهایی

### کتاب‌های ضروری

**مفاهیم پایه:**
- [The Phoenix Project](https://itrevolution.com/product/the-phoenix-project/) — Gene Kim (رمان، بهترین شروع)
- [The DevOps Handbook](https://itrevolution.com/product/the-devops-handbook/) — Kim, Debois, Willis, Humble
- [Accelerate](https://itrevolution.com/product/accelerate/) — Forsgren, Humble, Kim (پایه علمی)
- [Continuous Delivery](https://continuousdelivery.com/) — Humble, Farley

**تخصصی:**
- [Site Reliability Engineering](https://sre.google/sre-book/table-of-contents/) — Google (رایگان)
- [The Site Reliability Workbook](https://sre.google/workbook/table-of-contents/) — Google (رایگان)
- [Team Topologies](https://teamtopologies.com/) — Skelton, Pais
- [Thinking in Systems](https://www.chelseagreen.com/product/thinking-in-systems/) — Donella Meadows
- [The Goal](https://www.goldratt.com/) — Goldratt (تئوری محدودیت‌ها)
- [Release It!](https://pragprog.com/titles/mnee2/release-it-second-edition/) — Michael Nygard
- [Observability Engineering](https://www.oreilly.com/library/view/observability-engineering/9781492076438/) — Majors و همکاران
- [The Fearless Organization](https://fearlessorganization.com/) — Amy Edmondson

### پژوهش و داده

- [DORA](https://dora.dev/) — مرجع متریک‌ها
- [State of DevOps Report](https://cloud.google.com/devops/state-of-devops/)
- [Google Cloud: DevOps](https://cloud.google.com/architecture/devops)
- [DORA Capabilities](https://dora.dev/devops-capabilities/)

### استانداردها

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/)
- [NIST SSDF](https://csrc.nist.gov/Projects/ssdf)
- [SLSA](https://slsa.dev/)
- [The Twelve-Factor App](https://12factor.net/)
- [OpenTelemetry](https://opentelemetry.io/docs/)

### جامعه

- [DevOpsDays](https://devopsdays.org/)
- [Platform Engineering](https://platformengineering.org/)
- [CNCF](https://www.cncf.io/)
- [r/devops](https://www.reddit.com/r/devops/)
- [SRE Weekly](https://sreweekly.com/)

### دوره‌ها و مسیرها

- [DevOps Roadmap](https://roadmap.sh/devops)
- [Google Cloud Skills Boost](https://www.cloudskillsboost.google/)
- [Linux Foundation Training](https://training.linuxfoundation.org/)

### ابزارها (مفهومی)

**CI/CD:** GitLab CI، GitHub Actions، Jenkins، ArgoCD
**Containers:** Docker، Podman، Kubernetes
**IaC:** Terraform، OpenTofu، Ansible، Pulumi
**Observability:** Prometheus، Grafana، Loki، Tempo، Jaeger
**Security:** Semgrep، Trivy، Syft، Grype، Cosign، gitleaks
**Policy:** OPA، Kyverno، Sentinel
**Platform:** Backstage، Port، Crossplane

### چک‌لیست نهایی

```
[ ] یک تمرین از فصل ۱۱۹ انجام دادم
[ ] Lead Time تیمم را اندازه گرفتم
[ ] یک اقدام کوچک شروع کردم
[ ] یک کتاب از لیست انتخاب کردم
[ ] یک مسیر یادگیری انتخاب کردم
```

**پایان کتابچه.**
از اینجا به بعد، بهترین کار این است که یکی از تمرین‌ها را **همین هفته** انجام دهی.

---

## منابع و مراجع

**کتاب‌ها:**
- [The Phoenix Project](https://itrevolution.com/product/the-phoenix-project/)
- [The DevOps Handbook](https://itrevolution.com/product/the-devops-handbook/)
- [Accelerate](https://itrevolution.com/product/accelerate/)
- [Continuous Delivery](https://continuousdelivery.com/)
- [Site Reliability Engineering](https://sre.google/sre-book/table-of-contents/)
- [Team Topologies](https://teamtopologies.com/)
- [Thinking in Systems](https://www.chelseagreen.com/product/thinking-in-systems/)

**آنلاین:**
- [DORA](https://dora.dev/)
- [Google SRE](https://sre.google/)
- [OWASP](https://owasp.org/)
- [The Twelve-Factor App](https://12factor.net/)
- [DevOps Roadmap](https://roadmap.sh/devops)

---

**بازگشت به:** [فهرست کامل](SUMMARY.md)
