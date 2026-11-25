# بخش ۱۵: GitHub Copilot

**مدت یادگیری**: ۴ تا ۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۰۳ (branch، PR)

---

## فهرست محتوا

- [فصل ۶۶: Copilot چیست، چه نیست، و پلن‌ها](#فصل-۶۶)
- [فصل ۶۷: Completion، Chat و حالت‌های Ask/Edit/Agent](#فصل-۶۷)
- [فصل ۶۸: Copilot Cloud Agent — واگذاری کار روی GitHub](#فصل-۶۸)
- [فصل ۶۹: Custom Instructions و MCP](#فصل-۶۹)
- [فصل ۷۰: Copilot Code Review و استفاده‌ی مسئولانه](#فصل-۷۰)

> **هشدار نسخه:** قابلیت‌های Copilot سریع‌ترین بخش GitHub تغییر می‌کنند. نام حالت‌ها، پلن‌ها و روش فعال‌سازی ممکن است چند ماه بعد متفاوت باشد. هر جا عدد یا نام مشخصی می‌بینی، قبل از اتکا مستندات همان روز را چک کن.

---

## فصل ۶۶: Copilot چیست، چه نیست، و پلن‌ها

### مقدمه

Copilot یک دستیار کدنویسی مبتنی بر مدل زبانی است که در ادیتور، در GitHub.com، و در ترمینال کار می‌کند. ولی درک درست آن مهم است: Copilot **حدس می‌زند**، **نمی‌داند**.

**چرا مهم است؟** کسی که Copilot را «جعبه‌ی جادویی» ببیند، کد ناامن را بدون بررسی merge می‌کند. کسی که آن را «تکمیل‌کننده‌ی هوشمند با محدودیت» ببیند، ۳۰-۵۰٪ سریع‌تر کار می‌کند.

**کاربرد عملی:** این فصل نقشه‌ی ذهنی درست می‌دهد تا از بقیه‌ی فصل‌ها درست استفاده کنی.

### مفاهیم اساسی

#### چهار سطح تعامل

| سطح | کجا | چه می‌کند |
|---|---|---|
| **Completion** | ادیتور | پیشنهاد خط/بلوک در حین تایپ |
| **Chat (Ask)** | ادیتور / GitHub | سؤال، توضیح، پیشنهاد |
| **Edit** | ادیتور | تغییر چند فایل با دستور |
| **Agent** | ادیتور / GitHub | اجرای چندمرحله‌ای (خواندن، ویرایش، تست) |

#### Copilot چه می‌کند (خوب)

- تکمیل کد تکراری (boilerplate، تست، config)
- توضیح کد ناشناس
- نوشتن تست برای تابع موجود
- ترجمه‌ی کد بین زبان‌ها
- پیشنهاد refactor
- نوشتن commit message و PR description
- پاسخ به سؤال «این خطا چه معنایی دارد؟»

#### Copilot چه نمی‌کند (محدودیت‌ها)

- **نمی‌داند** کد تو چه کاری باید بکند (فقط حدس می‌زند)
- **نمی‌داند** قواعد کسب‌وکار تو
- **نمی‌داند** کتابخانه‌ی داخلی شرکتت
- **API جعل می‌کند** (hallucination) — تابعی که وجود ندارد
- **از کد فعلی کپی می‌کند** — باگ‌ها را تکرار می‌کند
- **کد ناامن تولید می‌کند** — اگر prompt ناامن باشد
- **license را نمی‌داند** — ممکن است کد GPL تولید کند

#### پلن‌ها (۲۰۲۶ — تقریبی)

| پلن | برای | ویژگی‌های کلیدی |
|---|---|---|
| **Free** | امتحان | completion محدود، chat محدود |
| **Pro** | فردی | completion نامحدود، chat، agent، مدل‌های بیشتر |
| **Pro+** | کاربر پرقدرت | مدل‌های پیشرفته، سهمیه‌ی بیشتر |
| **Business** | سازمان | مدیریت، policy، audit، IP indemnity |
| **Enterprise** | سازمان بزرگ | + customization، knowledge base |

**توجه:** اعداد و نام‌ها تغییر می‌کنند. صفحه‌ی pricing را چک کن.

#### مسائل حقوقی (مهم برای شرکت)

| موضوع | وضعیت |
|---|---|
| **کد تولیدشده** | مالکیت معمولاً به تو می‌رسد |
| **کد مشابه متن‌باز** | GitHub فیلتر دارد (code referencing) ولی تضمین نیست |
| **IP indemnity** | فقط در پلن‌های Business/Enterprise |
| **سیاست شرکت** | ممکن است استفاده را محدود کند |

**قاعده:** قبل از استفاده در کد تجاری، با تیم حقوقی/امنیت چک کن.

### مثال‌های عملی

#### مثال ۱: نصب و راه‌اندازی

```
۱. اشتراک Copilot بگیر (github.com/features/copilot)
۲. در VS Code: extension "GitHub Copilot"
۳. Sign in با حساب GitHub
۴. آیکون Copilot در status bar ظاهر می‌شود
```

بررسی:

```
VS Code → Command Palette → "Copilot: Check Status"
```

#### مثال ۲: تنظیمات مهم

```json
// .vscode/settings.json (مشترک تیم)
{
  "github.copilot.enable": {
    "*": true,
    "markdown": true,
    "yaml": true,
    "plaintext": false,
    "dotenv": false
  },
  "github.copilot.nextEditSuggestions.enabled": true,
  "editor.inlineSuggest.enabled": true
}
```

**`dotenv: false`** مهم است — جلوگیری از پیشنهاد در فایل‌های حاوی secret.

#### مثال ۳: بررسی اینکه Copilot کد تکراری نمی‌دهد

GitHub یک قابلیت **code referencing** دارد: اگر پیشنهاد با کد عمومی مطابقت داشته باشد، لینک می‌دهد. در تنظیمات:

```
Settings → Copilot → Code referencing
```

#### مثال ۴: محدود کردن Copilot در سازمان

```
Organization Settings → Copilot → Policies
  ☐ Allow Copilot in the CLI
  ☑ Allow Copilot Chat in the IDE
  ☑ Allow Copilot coding agent
  ☐ Allow Copilot code review
  ☐ Allow Bing search (برای context خارجی)
```

#### مثال ۵: `.copilotignore` (اگر پشتیبانی شود)

بعضی نسخه‌ها از `.copilotignore` پشتیبانی می‌کنند تا فایل‌های حساس از context حذف شوند. اگر در نسخه‌ی تو نیست، از `.gitignore` + تنظیمات ادیتور استفاده کن.

#### مثال ۶: سنجش واقعی بهره‌وری

```
قبل: ۱۰۰ خط کد در ۲ ساعت
بعد: ۱۰۰ خط کد در ۱.۵ ساعت + ۳۰ دقیقه review بیشتر
```

**نکته:** بهره‌وری خالص ممکن است کمتر از تصور باشد. ارزش اصلی در کارهای تکراری است، نه در منطق پیچیده.

### مقایسه و الگوهای مختلف

#### Copilot در برابر ChatGPT/Claude در مرورگر

| | Copilot | چت‌بات مرورگر |
|---|---|---|
| context کد | ✅ خودکار | ❌ باید کپی کنی |
| یکپارچگی ادیتور | ✅ | ❌ |
| حریم خصوصی | طبق سیاست GitHub | طبق سیاست سرویس |
| قیمت | اشتراک | متفاوت |

#### Copilot در برابر Cursor/Windsurf

| | Copilot | Cursor |
|---|---|---|
| یکپارچگی GitHub | ✅ بومی | متوسط |
| Agent | ✅ | ✅ |
| قیمت | متوسط | متوسط |
| مناسب | تیم روی GitHub | فردی، agent-first |

#### Free در برابر Pro

| | Free | Pro |
|---|---|---|
| completion | ۲۰۰۰/ماه | نامحدود |
| chat | ۵۰/ماه | نامحدود |
| agent | محدود | ✅ |
| مدل | پایه | چند مدل |

### اشکال شایع

#### ۱. اعتماد کامل به پیشنهاد

Copilot کد را «قانع‌کننده» می‌نویسد، حتی وقتی غلط است. **همیشه review کن.**

#### ۲. کد با API جعلی

```javascript
// Copilot پیشنهاد می‌دهد:
const user = await prisma.user.findUniqueOrThrow({ where: { id } });
// ولی در نسخه‌ی prisma تو، این متد وجود ندارد
```

همیشه compile/run کن.

#### ۳. کد ناامن

```javascript
// Copilot پیشنهاد می‌دهد:
db.query(`SELECT * FROM users WHERE id = ${id}`);
// SQL injection!
```

Copilot از کد اطراف یاد می‌گیرد — اگر کدت ناامن است، ناامن‌تر می‌شود.

#### ۴. استفاده در فایل‌های حساس

`.env`، `secrets.yaml`، `id_rsa` — Copilot ممکن است محتوا را به سرور بفرستد. در تنظیمات غیرفعال کن.

#### ۵. کد با license نامشخص

اگر کد مشابه GPL تولید شود و در پروژه‌ی تجاری استفاده کنی، مشکل حقوقی. code referencing را چک کن.

#### ۶. انتظار درست‌بودن در domain خاص

Copilot درباره‌ی کتابخانه‌ی داخلی شرکتت هیچ نمی‌داند. برای آن، custom instructions لازم است (فصل ۶۹).

#### ۷. نادیده گرفتن هزینه

Copilot Business برای ۱۰۰ نفر، هزینه‌ی ماهانه‌ی قابل توجهی است. ارزش را اندازه بگیر.

### نکات مهم (Tips & Tricks)

- **Copilot را برای boilerplate و تست استفاده کن**، نه برای منطق حساس.
- **`dotenv: false`** در تنظیمات.
- **code referencing** را روشن کن.
- **کد تولیدشده را مثل کد یک همکار تازه‌کار review کن.**
- **در سازمان، policy را تنظیم کن** — نه به انتخاب فردی.
- **قبل از استفاده در کد تجاری، مسائل حقوقی را چک کن.**

### خلاصه فصل

- Copilot چهار سطح دارد: completion، Ask، Edit، Agent.
- **حدس می‌زند، نمی‌داند** — همیشه review.
- API جعل می‌کند، کد ناامن می‌سازد، license را نمی‌داند.
- پلن‌ها: Free، Pro، Pro+، Business، Enterprise.
- IP indemnity فقط در Business/Enterprise.
- در فایل‌های حساس غیرفعال کن.

**سؤالات مروری:**
1. سه محدودیت اصلی Copilot چیست؟
2. چرا `dotenv: false` مهم است؟
3. چرا Copilot ممکن است کد ناامن تولید کند؟

---

## فصل ۶۷: Completion، Chat و حالت‌های Ask/Edit/Agent

### مقدمه

در ادیتور، Copilot سه حالت اصلی دارد: **Ask** (سؤال)، **Edit** (تغییر چند فایل)، **Agent** (اجرای چندمرحله‌ای با ابزار). انتخاب درست حالت، تفاوت بین «۵ دقیقه» و «۳۰ دقیقه» است.

**چرا مهم است؟** اگر برای یک سؤال ساده از Agent استفاده کنی، وقت و token تلف می‌شود. اگر برای یک بازآرایی چندفایلی از Ask استفاده کنی، باید ۲۰ بار کپی کنی.

**کاربرد عملی:** این فصل یک جدول تصمیم عملی می‌دهد.

### مفاهیم اساسی

#### سه حالت — جدول تصمیم

| سؤال | حالت |
|---|---|
| «این تابع چه کار می‌کند؟» | **Ask** |
| «این خطا چه معنایی دارد؟» | **Ask** |
| «چطور X را در این کتابخانه انجام دهم؟» | **Ask** |
| «این تابع را به async تبدیل کن» | **Edit** (تک‌فایل) |
| «نام این متغیر را در کل پروژه عوض کن» | **Edit** (چندفایل) |
| «تست برای این ماژول بنویس و اجرا کن» | **Agent** |
| «این باگ را پیدا کن و رفع کن» | **Agent** |
| «این ویژگی را پیاده کن» | **Agent** |

#### Ask — سؤال و جواب

- context: فایل باز، انتخاب، `@workspace`، `@terminal`
- خروجی: متن + بلوک کد (که باید کپی کنی)
- هزینه: کم
- خطر: کم

#### Edit — تغییر مستقیم

- context: فایل‌های انتخاب‌شده یا `@workspace`
- خروجی: diff قابل قبول/رد
- هزینه: متوسط
- خطر: متوسط (چند فایل عوض می‌شود)

#### Agent — اجرای چندمرحله‌ای

- context: workspace + ترمینال + ابزارها
- خروجی: تغییرات + اجرای دستور + تست
- هزینه: زیاد
- خطر: زیاد (دستور اجرا می‌کند)

#### context — چطور بده

| پیشوند | معنی |
|---|---|
| `#file:src/app.ts` | یک فایل خاص |
| `#selection` | انتخاب فعلی |
| `#codebase` | کل مخزن (index شده) |
| `#terminalLastCommand` | آخرین دستور |
| `@workspace` | جست‌وجو در workspace |
| `@vscode` | APIهای VS Code |
| `@terminal` | ترمینال |

### مثال‌های عملی

#### مثال ۱: Ask — توضیح کد

```
@workspace این تابع calculateTotal چه کاری می‌کند و چه edge caseهایی را
در نظر نگرفته؟ فایل #file:src/cart/total.ts
```

پاسخ خوب: توضیح + اشاره به `null`، تخفیف منفی، سرریز عددی.

#### مثال ۲: Ask — دیباگ خطا

```
این خطا را می‌گیرم:
TypeError: Cannot read properties of null (reading 'discount')

#terminalLastCommand را ببین. مشکل کجاست؟
```

#### مثال ۳: Edit — بازآرایی تک‌فایل

```
این فایل را بازآرایی کن:
- تابع validateEmail را به utils منتقل کن
- از async/await استفاده کن
- خطاها را با Result type برگردان، نه throw

#file:src/auth.ts
```

Copilot یک diff نشان می‌دهد. می‌توانی تکه‌تکه قبول/رد کنی.

#### مثال ۴: Edit — تغییر چندفایل

```
در کل پروژه، نام متغیر `usr` را به `user` تغییر بده.
فقط در فایل‌های TypeScript داخل src/.
```

#### مثال ۵: Agent — رفع باگ

```
باگ: وقتی کاربر کد تخفیف منقضی وارد می‌کند، خطای 500 می‌دهد.

۱. Issue را بازتولید کن (تست بنویس که شکست بخورد)
۲. علت را پیدا کن
۳. رفع کن
۴. تست را سبز کن
۵. خلاصه‌ی تغییرات را بنویس
```

Agent: فایل‌ها را می‌خواند، تست می‌نویسد، اجرا می‌کند، رفع می‌کند، دوباره اجرا می‌کند.

**نکته‌ی مهم:** Agent دستور در ترمینال اجرا می‌کند. **قبل از تأیید، دستور را بخوان.**

#### مثال ۶: Agent — با محدودیت

```
فقط فایل‌های داخل src/cart/ را تغییر بده.
هیچ وابستگی جدیدی اضافه نکن.
بعد از تغییر، `npm test -- src/cart` را اجرا کن.
```

محدودیت صریح، ریسک را کم می‌کند.

#### مثال ۷: Completion — ترفندها

```typescript
// تایپ کن: function validateEmail(
// Copilot امضا را کامل می‌کند
// Tab برای قبول

// برای پیشنهاد جایگزین: Alt+] (یا Alt+])
// برای باز کردن همه‌ی پیشنهادها: Ctrl+Enter
```

**ترفند:** کامنت بنویس، بعد Enter بزن — Copilot از کامنت کد می‌سازد:

```typescript
// یک تابع که ایمیل را اعتبارسنجی می‌کند و دلیل خطا را برمی‌گرداند
```

#### مثال ۸: تست نویسی با Agent

```
برای فایل #file:src/lib/discount.ts تست با vitest بنویس.
موارد:
- کد معتبر → تخفیف اعمال شود
- کد منقضی → خطای مشخص
- کد ناموجود → خطای مشخص
- تخفیف بیشتر از مبلغ → مبلغ صفر، نه منفی

تست‌ها را اجرا کن و اگر شکست خوردند، نشان بده کدام.
```

#### مثال ۹: Agent در GitHub.com (نه ادیتور)

در GitHub.com هم می‌توانی از Copilot در Issue/PR استفاده کنی (فصل ۶۸).

### مقایسه و الگوهای مختلف

#### Ask در برابر Edit در برابر Agent

| | Ask | Edit | Agent |
|---|---|---|---|
| سرعت | ✅ | ✅ | کند |
| هزینه | کم | متوسط | زیاد |
| تغییر فایل | ❌ | ✅ | ✅ |
| اجرای دستور | ❌ | ❌ | ✅ |
| مناسب | فهمیدن | تغییر مشخص | کار چندمرحله‌ای |

#### Agent در ادیتور در برابر Cloud Agent

| | Agent (IDE) | Cloud Agent |
|---|---|---|
| کجا | لپ‌تاپ تو | ابر GitHub |
| تعاملی | ✅ | نیمه (چت) |
| اجرا | فوری | صف |
| محدودیت زمان | ندارد | ۵۹ دقیقه |
| مناسب | کار روزمره | واگذاری |

#### context: `#codebase` در برابر `@workspace`

| | `#codebase` | `@workspace` |
|---|---|---|
| جست‌وجو | معنایی در index | ساختاری |
| سرعت | ✅ | متوسط |
| دقت | متوسط | ✅ |

### اشکال شایع

#### ۱. Agent بدون محدودیت

```
✗ "پروژه را بهتر کن"
```

Agent ممکن است ۵۰ فایل را عوض کند. **همیشه محدوده بده.**

#### ۲. تأیید دستور بدون خواندن

Agent پیشنهاد می‌دهد:

```bash
rm -rf node_modules && npm install
```

اگر `rm -rf /` باشد؟ همیشه بخوان.

#### ۳. Ask برای کار چندفایلی

۲۰ بار کپی/پیست. از Edit یا Agent استفاده کن.

#### ۴. context ندادن

```
✗ "این را رفع کن"
✓ "این را رفع کن. فایل #file:src/x.ts، خطا از #terminalLastCommand"
```

#### ۵. اعتماد به Agent در تست

Agent ممکن است تست را طوری بنویسد که پاس شود، نه اینکه درست باشد. تست را بخوان.

#### ۶. Agent که کد را «ساده» می‌کند

اگر بگویی «این را ساده کن»، ممکن است عملکرد را حذف کند. مشخص کن چه چیزی باید حفظ شود.

#### ۷. اجرای Agent روی branch اشتباه

قبل از Agent: `git status` و `git branch`. Agent تغییرات را در working directory می‌گذارد.

### نکات مهم (Tips & Tricks)

- **Ask برای فهمیدن، Edit برای تغییر مشخص، Agent برای کار چندمرحله‌ای.**
- **محدوده بده** — فایل‌ها، محدودیت‌ها، معیار موفقیت.
- **دستورهای Agent را بخوان** قبل از تأیید.
- **قبل از Agent، commit کن** — اگر خراب شد، `git restore`.
- **context صریح** بده: `#file`، `#selection`، `#terminalLastCommand`.
- **تست بنویس و اجرا کن** — Agent بدون تست، حدس می‌زند.

### خلاصه فصل

- سه حالت: Ask (فهمیدن)، Edit (تغییر)، Agent (چندمرحله‌ای).
- Agent دستور اجرا می‌کند — با محدودیت و نظارت.
- context صریح بده.
- قبل از Agent، commit کن.
- تست را بخوان، نه فقط سبز بودنش.

**سؤالات مروری:**
1. چه زمانی Agent و چه زمانی Ask؟
2. چرا باید قبل از Agent commit کرد؟
3. چرا تست تولیدشده توسط Agent باید خوانده شود؟

---

## فصل ۶۸: Copilot Cloud Agent — واگذاری کار روی GitHub

### مقدمه

Cloud Agent یک قابلیت متفاوت است: به‌جای کار در ادیتور، کار را **واگذار** می‌کنی. یک Issue یا درخواست می‌دهی، Copilot در یک محیط ابری موقت کار می‌کند، branch می‌سازد، commit می‌زند و PR باز می‌کند.

**چرا مهم است؟** این مدل کار جدیدی است: «همکار نیمه‌خودکار». می‌توانی کارهای تکراری (ارتقای وابستگی، رفع lint، نوشتن تست) را واگذار کنی و روی کار اصلی تمرکز کنی.

**کاربرد عملی:** این فصل می‌گوید چه کاری را واگذار کن، چطور، و چه محدودیت‌هایی دارد.

> **هشدار:** این قابلیت سریع تغییر می‌کند (نام، روش فعال‌سازی، محدودیت‌ها). مستندات روز را چک کن.

### مفاهیم اساسی

#### چطور کار می‌کند

```
۱. یک Issue یا درخواست می‌دهی
۲. Copilot در محیط ابری موقت (ephemeral) شروع می‌کند
۳. فایل‌ها را می‌خواند، برنامه می‌ریزد، کد می‌نویسد
۴. تست و linter اجرا می‌کند
۵. branch می‌سازد، commit می‌زند، push می‌کند
۶. PR باز می‌کند
۷. تو review می‌کنی (مثل هر PR دیگر)
```

#### راه‌های شروع

| راه | چطور |
|---|---|
| **از Issue** | Issue → Assignees → Copilot |
| **از agents panel** | پنل اختصاصی در GitHub.com |
| **از chat** | سؤال را به کار واگذار کن |
| **از PR** | درخواست اصلاح |
| **از security campaign** | alertهای code scanning را به Copilot بده |
| **از CLI** | remote control یک session CLI |

#### محدودیت‌ها (مهم)

| محدودیت | مقدار |
|---|---|
| زمان هر session | **۵۹ دقیقه** (سخت، قابل تمدید نیست) |
| دامنه‌ی مخزن | فقط مخزن مشخص‌شده در شروع |
| branch | یکی در هر زمان |
| PR | یکی در هر کار |
| مخزن | فقط GitHub-hosted |
| context | به‌طور پیش‌فرض فقط همان مخزن |

**۵۹ دقیقه** مهم‌ترین محدودیت است: کارهای بزرگ باید به قطعات کوچک شکسته شوند.

#### پیش‌نیازها

- Copilot پلن Pro/Pro+/Business/Enterprise (بسته به سیاست)
- فعال بودن در سطح سازمان/مخزن
- Ruleset سازگار (بعضی قوانین مثل محدودیت نویسنده‌ی commit، Copilot را بلاک می‌کند)
- `copilot-setup-steps.yml` برای سفارشی‌سازی محیط (اختیاری)

### مثال‌های عملی

#### مثال ۱: واگذاری یک Issue

```markdown
<!-- در Issue -->

## عنوان
افزودن اعتبارسنجی ایمیل در فرم ثبت‌نام

## توضیح
فرم فعلی هر ورودی را قبول می‌کند. باید:
- فرمت ایمیل بررسی شود
- ایمیل تکراری رد شود
- پیام خطای فارسی نمایش داده شود

## معیار پذیرش
- [ ] ایمیل نامعتبر → پیام خطا
- [ ] ایمیل تکراری → پیام خطا
- [ ] تست واحد برای validator
- [ ] تست e2e برای فرم

## محدوده
فقط `src/features/auth/` و `tests/auth/`
```

سپس: **Assignees → Copilot**.

#### مثال ۲: درخواست مستقیم

```
در agents panel:

"در فایل src/api/orders.ts، endpoint POST /orders را طوری تغییر بده که
کد تخفیف منقضی، خطای 422 با پیام واضح برگرداند نه 500.
تست مربوطه در tests/api/orders.spec.ts را هم به‌روز کن.
فقط این دو فایل را تغییر بده."
```

#### مثال ۳: اصلاح یک PR

```
در PR #42:

"reviewer گفت که کوئری N+1 دارد. در src/repo/users.ts،
مشکل را رفع کن و تست performance اضافه کن."
```

#### مثال ۴: واگذاری alert امنیتی

```
Security → Code scanning → یک alert → "Assign to Copilot"
```

Copilot تلاش می‌کند آسیب‌پذیری را رفع کند و PR بزند.

#### مثال ۵: سفارشی‌سازی محیط

`.github/copilot-setup-steps.yml` (اگر پشتیبانی شود):

```yaml
steps:
  - uses: actions/setup-node@v4
    with:
      node-version: "22"
      cache: npm
  - run: npm ci
  - run: npx prisma generate
```

**چرا؟** بدون این، Copilot نمی‌داند چطور محیط را آماده کند و تست‌ها شکست می‌خورند.

#### مثال ۶: کارهای مناسب برای واگذاری

| مناسب | نامناسب |
|---|---|
| ارتقای وابستگی | طراحی معماری |
| رفع lint/format | تصمیم محصول |
| نوشتن تست برای کد موجود | تغییر مدل کسب‌وکار |
| رفع باگ مشخص با بازتولید | باگ مبهم |
| افزودن endpoint ساده | بازنویسی کامل |
| به‌روزرسانی مستندات | تصمیم امنیتی |

#### مثال ۷: شکستن کار بزرگ

```
✗ "مهاجرت از REST به GraphQL"

✓ Issue 1: "افزودن schema GraphQL برای User" (۳۰ دقیقه)
✓ Issue 2: "افزودن resolver برای User" (۳۰ دقیقه)
✓ Issue 3: "افزودن schema برای Order" (۳۰ دقیقه)
✓ Issue 4: "مهاجرت صفحه‌ی پروفایل به GraphQL" (۴۵ دقیقه)
```

هر کدام زیر ۵۹ دقیقه.

#### مثال ۸: بررسی خروجی

```bash
# PR ساخته‌شده را ببین
gh pr list --author "copilot-swe-agent[bot]"

# diff را کامل بخوان
gh pr diff <number>

# تست محلی
gh pr checkout <number>
npm test
```

**هرگز PR Copilot را بدون review merge نکن.**

#### مثال ۹: محدود کردن دسترسی

```
Settings → Copilot → Coding agent
  ☑ Allow Copilot to create pull requests
  ☐ Allow Copilot to push to protected branches (خاموش)
  Bypass actors: (هیچ)
```

اگر Ruleset نیاز به review دارد، Copilot نمی‌تواند خودش merge کند — که خوب است.

### مقایسه و الگوهای مختلف

#### Cloud Agent در برابر Agent در ادیتور

| | Cloud | IDE |
|---|---|---|
| کجا | ابر | لپ‌تاپ |
| تعاملی | نیمه | ✅ کامل |
| زمان | ۵۹ دقیقه سخت | نامحدود |
| دسترسی به محیط محلی | ❌ | ✅ |
| مناسب | واگذاری | کار روزمره |

#### واگذاری در برابر انجام خودت

| | واگذاری | خودت |
|---|---|---|
| سرعت شروع | فوری (صف) | فوری |
| کیفیت | متوسط | بالا (اگر بلد باشی) |
| یادگیری | ❌ | ✅ |
| مناسب | کار تکراری | کار پیچیده |

#### Issue-driven در برابر chat-driven

| | Issue | Chat |
|---|---|---|
| ثبت | ✅ (audit) | ❌ |
| معیار پذیرش | ✅ | ❌ |
| مناسب | کار تیمی | کار فردی |

**توصیه:** از Issue شروع کن — هم audit دارد هم معیار پذیرش.

### اشکال شایع

#### ۱. کار بزرگ‌تر از ۵۹ دقیقه

session timeout می‌شود و کار نیمه‌کاره می‌ماند. **بشکن.**

#### ۲. Issue مبهم

```
✗ "بهبود performance"
✓ "کاهش زمان response endpoint /orders از ۸۰۰ms به زیر ۲۰۰ms
   با افزودن index روی (user_id, created_at)"
```

#### ۳. merge بدون review

Copilot می‌تواند کد ناامن یا نادرست تولید کند. **همیشه review.**

#### ۴. Ruleset ناسازگار

```
Copilot is unable to create commits because of branch rules
```

اگر Ruleset «محدودیت نویسنده‌ی commit» دارد، Copilot را به bypass اضافه کن (اگر سیاست اجازه می‌دهد).

#### ۵. انتظار دسترسی به سیستم‌های داخلی

Cloud Agent در محیط GitHub اجرا می‌شود، نه شبکه‌ی شرکت. به دیتابیس داخلی دسترسی ندارد.

#### ۶. چند مخزن در یک کار

Copilot فقط در مخزن مشخص‌شده کار می‌کند. برای تغییر در چند مخزن، چند Issue.

#### ۷. context ناکافی

اگر کد در مخزن دیگری است (کتابخانه‌ی مشترک)، Copilot نمی‌بیندش. لینک یا توضیح بده.

### نکات مهم (Tips & Tricks)

- **۵۹ دقیقه** — کارها را کوچک بشکن.
- **Issue با معیار پذیرش** بده، نه یک جمله.
- **محدوده‌ی فایل** را صریح بگو.
- **`copilot-setup-steps.yml`** برای محیط.
- **همیشه review** — مثل PR یک همکار تازه‌کار.
- **از Issue شروع کن** — audit و وضوح.

### خلاصه فصل

- Cloud Agent کار را در ابر GitHub انجام می‌دهد و PR می‌سازد.
- محدودیت سخت **۵۹ دقیقه** — کارها را بشکن.
- از Issue با معیار پذیرش شروع کن.
- فقط مخزن مشخص، فقط یک branch، فقط یک PR.
- **همیشه review کن.**
- مناسب کار تکراری، نه طراحی.

**سؤالات مروری:**
1. چرا محدودیت ۵۹ دقیقه مهم است؟
2. چه نوع کاری برای Cloud Agent مناسب است؟
3. چرا باید از Issue شروع کرد نه chat؟

---

## فصل ۶۹: Custom Instructions و MCP

### مقدمه

Copilot به‌طور پیش‌فرض نمی‌داند پروژه‌ی تو چه قواعدی دارد: از چه کتابخانه‌ای استفاده می‌کنی، نام‌گذاری چطور است، چه الگوهایی ممنوع است. **Custom Instructions** این را حل می‌کند. **MCP** دسترسی به ابزارهای خارجی می‌دهد.

**چرا مهم است؟** بدون instructions، هر بار باید توضیح دهی و Copilot باز هم کد ناهماهنگ می‌دهد. با instructions، از اولین پیشنهاد هماهنگ است.

**کاربرد عملی:** یک فایل ۵۰ خطی، کیفیت همه‌ی پیشنهادها را بالا می‌برد.

### مفاهیم اساسی

#### سه سطح instructions

| فایل | دامنه | کجا |
|---|---|---|
| `.github/copilot-instructions.md` | کل مخزن | GitHub.com + IDE |
| `.github/instructions/*.instructions.md` | مسیر مشخص (با `applyTo`) | IDE + code review |
| `AGENTS.md` | کل مخزن (استاندارد باز) | ابزارهای مختلف |
| `~/.copilot/instructions.md` | شخصی، همه‌ی مخزن‌ها | IDE |

**توجه:** نام فایل‌ها و پشتیبانی ممکن است تغییر کند. `AGENTS.md` یک استاندارد باز است که ابزارهای زیادی (نه فقط Copilot) پشتیبانی می‌کنند — انتخاب امن‌تری است.

#### چه چیزی در instructions بنویسیم

| خوب | بد |
|---|---|
| «از `Result<T, E>` استفاده کن، نه throw» | «کد خوب بنویس» |
| «نام فایل‌ها kebab-case» | «نام‌گذاری مناسب» |
| «هر endpoint باید zod schema داشته باشد» | «اعتبارسنجی کن» |
| «تست با vitest، نه jest» | «تست بنویس» |
| «از `any` استفاده نکن» | «تایپ قوی» |

**قاعده:** مشخص، قابل بررسی، با مثال.

#### MCP چیست

Model Context Protocol — یک استاندارد باز که به Copilot اجازه می‌دهد به ابزارها و داده‌های خارجی وصل شود:

```
Copilot ←→ MCP Server ←→ (دیتابیس، API، فایل، Jira، Slack، ...)
```

#### MCP serverهای رایج

| Server | چه می‌دهد |
|---|---|
| **GitHub MCP** | Issue، PR، Actions، code search |
| **Filesystem** | دسترسی به فایل‌های محلی |
| **Postgres/SQLite** | کوئری دیتابیس |
| **Playwright** | کنترل مرورگر |
| **Sentry** | خطاها |
| **Linear/Jira** | Issue tracker |
| **Slack** | پیام‌ها |

### مثال‌های عملی

#### مثال ۱: `copilot-instructions.md`

`.github/copilot-instructions.md`:

```markdown
# راهنمای پروژه برای Copilot

## زبان و فریم‌ورک
- TypeScript 5.x، strict mode
- React 19 با Server Components
- Next.js 16 (App Router)
- Tailwind CSS 4

## قواعد کد

### TypeScript
- `any` ممنوع. از `unknown` + type guard استفاده کن.
- تایپ‌ها را در `types/` تعریف کن، نه inline.
- از `satisfies` به‌جای `as` استفاده کن.
- تابع‌های خالص را در `lib/` بگذار.

### React
- Server Component پیش‌فرض است. `"use client"` فقط وقتی لازم است.
- از `useEffect` برای data fetching استفاده نکن — در Server Component fetch کن.
- نام کامپوننت‌ها PascalCase، فایل‌ها kebab-case.

### خطاها
- از `Result<T, E>` استفاده کن، نه throw (به‌جز در boundary).
- پیام خطا برای کاربر فارسی، برای لاگ انگلیسی.
- هر خطای غیرمنتظره باید log شود.

### تست
- vitest برای unit، Playwright برای e2e.
- هر تابع جدید باید تست داشته باشد.
- نام تست: `should <behavior> when <condition>`

### استایل
- Tailwind، بدون CSS module.
- رنگ‌ها از `theme.colors`، نه hex مستقیم.
- RTL پیش‌فرض است.

## ساختار
```
src/
├── app/          ← routes (Next.js App Router)
├── features/     ← feature-based
├── lib/          ← utils، helpers
├── components/   ← UI مشترک
└── types/        ← تایپ‌های مشترک
```

## ممنوع
- `console.log` در production code
- `dangerouslySetInnerHTML` بدون sanitize
- کوئری SQL با string concatenation
- `fetch` بدون timeout
- وابستگی جدید بدون بحث در Issue

## دستورها
- `npm run dev` — توسعه
- `npm test` — تست
- `npm run lint` — lint
- `npm run typecheck` — بررسی تایپ
```

#### مثال ۲: instruction برای مسیر خاص

`.github/instructions/api.instructions.md`:

```markdown
---
applyTo: "src/app/api/**/*.ts"
---

# قواعد API

- هر route باید zod schema برای input داشته باشد.
- خروجی همیشه `{ data }` یا `{ error: { code, message } }`.
- کدهای خطا: 400 (اعتبارسنجی)، 401 (احراز هویت)، 403 (مجوز)، 404 (یافت نشد)، 422 (منطق)، 500 (سرور).
- rate limit روی همه‌ی endpointهای عمومی.
- هیچ‌وقت stack trace را به کاربر نشان نده.
- هر endpoint باید در `docs/api.md` مستند شود.
```

#### مثال ۳: `AGENTS.md` (استاندارد باز)

`AGENTS.md` در ریشه:

```markdown
# AGENTS.md

این فایل برای همه‌ی دستیارهای AI (Copilot، Cursor، Claude Code، ...) است.

## شروع کار
1. `npm install`
2. `npm run dev`
3. تست: `npm test`

## قواعد
[همان محتوای copilot-instructions]

## چه کاری نکن
- فایل‌های `migrations/` را دستی ویرایش نکن
- `package-lock.json` را دستی ویرایش نکن
- به `main` مستقیم push نکن
```

#### مثال ۴: MCP — تنظیم در VS Code

`.vscode/mcp.json`:

```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://localhost:5432/dev"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/projects"]
    }
  }
}
```

**هشدار امنیتی:** MCP server می‌تواند به دیتابیس/فایل دسترسی داشته باشد. فقط serverهای قابل اعتماد و با دسترسی محدود.

#### مثال ۵: استفاده از MCP در chat

```
@github این PR را review کن
@postgres چند کاربر در ۲۴ ساعت گذشته ثبت‌نام کرده‌اند؟
```

#### مثال ۶: MCP server سفارشی

اگر API داخلی داری:

```javascript
// mcp-server.js
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server(
  { name: "internal-api", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler("tools/list", async () => ({
  tools: [
    {
      name: "get_customer",
      description: "اطلاعات مشتری را برمی‌گرداند",
      inputSchema: {
        type: "object",
        properties: { id: { type: "string" } },
        required: ["id"],
      },
    },
  ],
}));

server.setRequestHandler("tools/call", async (req) => {
  if (req.params.name === "get_customer") {
    const res = await fetch(`https://internal.api/customers/${req.params.arguments.id}`);
    return { content: [{ type: "text", text: await res.text() }] };
  }
});

await server.connect(new StdioServerTransport());
```

#### مثال ۷: بررسی اینکه instructions اعمال می‌شود

```
در Copilot Chat بپرس:
"چه قواعدی از copilot-instructions.md برای این پروژه داری؟"

اگر جواب داد، اعمال شده.
```

#### مثال ۸: instruction برای code review

اگر Copilot code review داری، instructions روی آن هم اثر می‌گذارد. می‌توانی اضافه کنی:

```markdown
## در code review به این‌ها دقت کن
- آیا ورودی کاربر اعتبارسنجی شده؟
- آیا secret در کد هست؟
- آیا کوئری N+1 دارد؟
- آیا تست برای رفتار جدید نوشته شده؟
- آیا `any` استفاده شده؟
```

### مقایسه و الگوهای مختلف

#### `copilot-instructions.md` در برابر `AGENTS.md`

| | copilot-instructions | AGENTS.md |
|---|---|---|
| پشتیبانی Copilot | ✅ | ✅ (اگر پشتیبانی شود) |
| پشتیبانی ابزار دیگر | ❌ | ✅ |
| استاندارد | اختصاصی GitHub | باز |
| توصیه | ✅ | ✅ (هر دو) |

**بهترین:** `AGENTS.md` به‌عنوان منبع اصلی، و `copilot-instructions.md` که به آن اشاره می‌کند:

```markdown
# راهنمای Copilot

قواعد پروژه در [AGENTS.md](../AGENTS.md) است. لطفاً آن را رعایت کن.
```

#### instruction در برابر prompt

| | instruction | prompt |
|---|---|---|
| دامنه | همه‌ی تعامل‌ها | یک تعامل |
| تکرار | خودکار | دستی |
| مناسب | قواعد پروژه | کار مشخص |

#### MCP در برابر custom action

| | MCP | Action |
|---|---|---|
| کاربرد | context و ابزار | CI/CD |
| در Copilot | ✅ | ❌ |
| استاندارد | باز | GitHub |

### اشکال شایع

#### ۱. instruction مبهم

```
✗ "کد تمیز بنویس"
✓ "توابع حداکثر ۳۰ خط، پارامترها حداکثر ۴"
```

#### ۲. instruction خیلی بلند

اگر ۵۰۰ خط باشد، Copilot بخشی را نادیده می‌گیرد. حداکثر ۱۰۰-۱۵۰ خط، مهم‌ترین‌ها.

#### ۳. instruction متناقض

```
✗ "همیشه از classes استفاده کن" + "از functional استفاده کن"
```

#### ۴. MCP server با دسترسی بیش از حد

```
✗ filesystem server با دسترسی به /
✓ با دسترسی به یک پوشه‌ی مشخص
```

#### ۵. MCP server غیرقابل اعتماد

MCP server کد اجرا می‌کند. فقط از منابع رسمی/بررسی‌شده.

#### ۶. secret در `mcp.json`

```json
✗ "env": { "DB_URL": "postgres://user:password@..." }
```

از متغیر محیطی استفاده کن:

```json
"env": { "DATABASE_URL": "${env:DATABASE_URL}" }
```

#### ۷. فراموش کردن به‌روزرسانی instructions

پروژه تغییر کرد، instructions نه. Copilot کد قدیمی می‌دهد. instructions را بخشی از PRها کن.

### نکات مهم (Tips & Tricks)

- **`AGENTS.md`** به‌عنوان منبع واحد، `copilot-instructions.md` اشاره‌گر.
- **مشخص و قابل بررسی** بنویس، با مثال.
- **حداکثر ۱۵۰ خط** — بیشتر نادیده گرفته می‌شود.
- **`applyTo`** برای قواعد مسیر-محور.
- **MCP با دسترسی محدود** — نه root، نه همه‌ی دیتابیس.
- **instructions را با کد به‌روز نگه دار.**

### خلاصه فصل

- Custom Instructions قواعد پروژه را به Copilot می‌دهد.
- `AGENTS.md` استاندارد باز، `copilot-instructions.md` اختصاصی GitHub.
- مشخص و قابل بررسی بنویس، با مثال.
- `applyTo` برای قواعد مسیر-محور.
- MCP دسترسی به ابزار خارجی — با دسترسی محدود.
- instructions را مثل کد نگهداری کن.

**سؤالات مروری:**
1. چرا instruction مبهم بی‌فایده است؟
2. تفاوت `AGENTS.md` و `copilot-instructions.md` چیست؟
3. چه خطری در MCP server با دسترسی زیاد وجود دارد؟

---

## فصل ۷۰: Copilot Code Review و استفاده‌ی مسئولانه

### مقدمه

Copilot می‌تواند PR را review کند: کامنت بگذارد، مشکل پیدا کند، پیشنهاد بدهد. این مکمل review انسانی است، نه جایگزین.

**چرا مهم است؟** Copilot review در پیدا کردن مسائل مکانیکی (خطاهای رایج، الگوهای ناامن، فراموشی تست) خوب است و وقت reviewer انسانی را آزاد می‌کند برای منطق و طراحی.

**کاربرد عملی:** این فصل نحوه‌ی استفاده، محدودیت‌ها، و استفاده‌ی مسئولانه را پوشش می‌دهد.

### مفاهیم اساسی

#### Copilot review چه می‌بیند

| خوب در | ضعیف در |
|---|---|
| خطاهای رایج زبان | درک زمینه‌ی کسب‌وکار |
| الگوهای ناامن (SQL injection، XSS) | معماری |
| فراموشی تست | تصمیم‌های طراحی |
| ناسازگاری با instructions | trade-offها |
| پیچیدگی غیرضروری | اولویت محصول |

#### چطور فعال می‌شود

| راه | چطور |
|---|---|
| **دستی** | PR → Reviewers → Copilot |
| **خودکار برای همه** | Ruleset → Automatically request Copilot review |
| **خودکار با شرط** | workflow با `actions/github-script` |

#### قابلیت‌های review

- کامنت خط‌به‌خط
- suggested change (قابل اعمال با یک کلیک)
- خلاصه‌ی کلی PR
- هشدار امنیتی
- بررسی مطابقت با instructions

### مثال‌های عملی

#### مثال ۱: درخواست دستی

در PR: **Reviewers** (سمت راست) → `Copilot` → منتظر review.

#### مثال ۲: خودکار برای همه‌ی PRها

```
Settings → Rules → Rulesets → New branch ruleset
☑ Automatically request Copilot code review
```

یا در Branch protection قدیمی.

#### مثال ۳: خودکار با workflow

```yaml
name: Copilot Review

on:
  pull_request:
    types: [opened, ready_for_review]

permissions:
  pull-requests: write

jobs:
  request:
    if: github.event.pull_request.draft == false
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/github-script@<sha>
        with:
          script: |
            try {
              await github.rest.pulls.requestReviewers({
                ...context.repo,
                pull_number: context.payload.pull_request.number,
                reviewers: ["copilot-pull-request-reviewer[bot]"],
              });
            } catch (e) {
              core.warning(`درخواست review ناموفق: ${e.message}`);
            }
```

**هشدار:** نام bot و روش فعال‌سازی ممکن است تغییر کند. مستندات را چک کن.

#### مثال ۴: فقط برای PRهای بزرگ

```yaml
  request:
    if: github.event.pull_request.additions > 100
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/github-script@<sha>
        with:
          script: |
            await github.rest.pulls.requestReviewers({
              ...context.repo,
              pull_number: context.payload.pull_request.number,
              reviewers: ["copilot-pull-request-reviewer[bot]"],
            });
```

**چرا؟** PR کوچک را انسان سریع‌تر review می‌کند.

#### مثال ۵: ارزیابی کامنت Copilot

```
کامنت Copilot:
"این کوئری ممکن است N+1 داشته باشد. پیشنهاد: از include استفاده کن."

ارزیابی:
- آیا واقعاً N+1 است؟ (شاید در این مورد خاص مهم نیست)
- آیا پیشنهاد درست است؟ (شاید ORM تو این را ندارد)
- آیا با معماری تو می‌خواند؟
```

**هر کامنت را خودت ارزیابی کن.** Dismiss بی‌دلیل و اعمال بی‌دلیل، هر دو بد.

#### مثال ۶: استفاده از suggested change

مثل review انسانی: **Commit suggestion** یا **Add to batch**.

**هشدار:** suggestion Copilot را هم بخوان — ممکن است کد را بشکند.

#### مثال ۷: ترکیب با review انسانی

```
۱. Copilot review (مکانیکی، ۲ دقیقه)
۲. رفع مسائل واضح
۳. reviewer انسانی (منطق، طراحی)
```

این ترتیب، وقت reviewer انسانی را برای چیزهای مهم آزاد می‌کند.

#### مثال ۸: استفاده‌ی مسئولانه — چک‌لیست

- [ ] هر کامنت Copilot را خوانده‌ام
- [ ] suggestionها را قبل از اعمال بررسی کرده‌ام
- [ ] تست‌ها بعد از اعمال سبزند
- [ ] کد را خودم می‌فهمم (نه فقط «کار می‌کند»)
- [ ] license کد تولیدشده را چک کرده‌ام (اگر حساس است)
- [ ] secret در prompt نگذاشته‌ام

#### مثال ۹: چیزهایی که در prompt Copilot نگذار

```
✗ کلید API واقعی
✗ داده‌ی واقعی مشتری
✗ کد proprietary حساس (بسته به سیاست)
✗ اطلاعات شخصی
```

#### مثال ۱۰: اندازه‌گیری ارزش

```
معیارهای قبل/بعد:
- زمان review هر PR
- تعداد باگ در production
- زمان تا اولین commit در پروژه‌ی جدید
- رضایت تیم (نظرسنجی)
```

**بدون اندازه‌گیری، نمی‌دانی ارزش دارد یا نه.**

### مقایسه و الگوهای مختلف

#### Copilot review در برابر review انسانی

| | Copilot | انسان |
|---|---|---|
| سرعت | ✅ فوری | کند |
| زمینه‌ی کسب‌وکار | ❌ | ✅ |
| معماری | ❌ | ✅ |
| مسائل مکانیکی | ✅ | متوسط |
| خستگی | ❌ | ✅ |
| هزینه | اشتراک | وقت |

**بهترین:** Copilot اول، انسان بعد.

#### خودکار برای همه در برابر انتخابی

| | همه | انتخابی |
|---|---|---|
| پوشش | ✅ | متوسط |
| نویز | زیاد | کم |
| هزینه | بیشتر | کمتر |
| مناسب | تیم کوچک | تیم بزرگ |

#### review در برابر agent

| | review | agent |
|---|---|---|
| تغییر می‌دهد | ❌ (پیشنهاد) | ✅ |
| سرعت | ✅ | کند |
| مناسب | بررسی | انجام |

### اشکال شایع

#### ۱. اعتماد کامل به review Copilot

Copilot ممکن است مسئله‌ی مهمی را نبیند یا مسئله‌ی nonexistent بسازد. **review انسانی جایگزین نمی‌شود.**

#### ۲. dismiss همه‌ی کامنت‌ها

اگر همیشه dismiss کنی، چرا فعال کرده‌ای؟ کامنت‌ها را بخوان و یاد بگیر.

#### ۳. اعمال suggestion بدون تست

Copilot می‌تواند کد را بشکند. بعد از اعمال، تست اجرا کن.

#### ۴. secret در prompt

اگر secret را در چت بگذاری، به سرور می‌رود. **هرگز.**

#### ۵. review خودکار برای draft

PR draft هنوز آماده نیست. `if: draft == false`.

#### ۶. نویز زیاد

اگر Copilot برای هر PR ۲۰ کامنت بگذارد، تیم نادیده می‌گیرد. با instructions دقیق‌تر، نویز را کم کن.

#### ۷. کد تولیدشده که نمی‌فهمی

اگر کد Copilot را merge کنی و نفهمی، بدهی فنی می‌سازی. **اگر نمی‌فهمی، merge نکن.**

#### ۸. license

اگر کد مشابه GPL تولید شد، در پروژه‌ی تجاری مشکل است. code referencing را چک کن.

### نکات مهم (Tips & Tricks)

- **Copilot اول، انسان بعد** — وقت reviewer را آزاد کن.
- **instructions دقیق** = نویز کمتر.
- **هر suggestion را بخوان** قبل از اعمال.
- **بعد از اعمال، تست.**
- **secret در prompt نگذار.**
- **اندازه‌گیری کن** — بدون داده، تصمیم نگیر.
- **اگر کد را نمی‌فهمی، merge نکن.**

### خلاصه فصل

- Copilot review مکمل است، نه جایگزین.
- در مکانیکی خوب، در زمینه و معماری ضعیف.
- خودکار برای همه یا انتخابی — بسته به اندازه‌ی تیم.
- هر کامنت و suggestion را ارزیابی کن.
- secret در prompt نگذار.
- **کدی که نمی‌فهمی را merge نکن.**

**سؤالات مروری:**
1. Copilot review کجا خوب است و کجا نه؟
2. چرا نباید suggestion را بدون تست اعمال کرد؟
3. چرا merge کردن کدی که نمی‌فهمی خطرناک است؟

---

**بعدی:** [`16-Security-Features.md`](16-Security-Features.md)

---

## منابع و مراجع

- GitHub Docs: About GitHub Copilot — https://docs.github.com/en/copilot
- GitHub Docs: Copilot cloud agent — https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent
- GitHub Docs: Enhancing Copilot agent mode with MCP — https://docs.github.com/en/copilot/tutorials/enhance-agent-mode-with-mcp
- GitHub Docs: Adding custom instructions — https://docs.github.com/en/copilot/how-tos/configure-custom-instructions
- GitHub Docs: Copilot code review — https://docs.github.com/en/copilot/using-github-copilot/code-review
- Model Context Protocol — https://modelcontextprotocol.io/
- AGENTS.md (استاندارد باز) — https://agents.md/
