# بخش ۰۷: Issue، برچسب و Projects

**مدت یادگیری**: ۳ تا ۴ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۰۶ (ساختار مخزن، قالب‌ها)

---

## فهرست محتوا

- [فصل ۲۹: Issue — نوشتن گزارش و درخواستی که جدی گرفته شود](#فصل-۲۹)
- [فصل ۳۰: برچسب، milestone، assignee و issue types](#فصل-۳۰)
- [فصل ۳۱: GitHub Projects — تخته، جدول و roadmap](#فصل-۳۱)
- [فصل ۳۲: خودکارسازی Issue و Projects](#فصل-۳۲)

---

## فصل ۲۹: Issue — نوشتن گزارش و درخواستی که جدی گرفته شود

### مقدمه

Issue ساده‌ترین و در عین حال کم‌استفاده‌ترین ابزار GitHub است. اکثر Issueها دو خط‌اند: «کار نمی‌کند». نتیجه: نگه‌دارنده نمی‌تواند کاری کند، سؤال می‌پرسد، دو روز طول می‌کشد، و طرف مقابل دلخور می‌شود.

**چرا مهم است؟** Issue خوب = رفع سریع‌تر + بحث کمتر + احتمال بیشتر حل شدن. Issue بد = دور ریختن وقت دو نفر.

**کاربرد عملی:** این فصل یک الگوی مشخص برای نوشتن Issue می‌دهد که هم برای گزارش باگ، هم برای درخواست ویژگی، هم برای وظیفه‌ی داخلی کار می‌کند.

### مفاهیم اساسی

#### سه نوع Issue

| نوع | هدف | مثال |
|---|---|---|
| **Bug** | چیزی خراب است | «دکمه‌ی پرداخت صفحه را سفید می‌کند» |
| **Feature** | چیزی نیست و باید باشد | «امکان خروجی CSV» |
| **Task** | کار داخلی | «ارتقای React به ۱۹» |

#### آناتومی Issue خوب

```
۱. عنوان گویا (چی + کجا)
۲. زمینه (چه کاری می‌خواستی بکنی)
۳. رفتار فعلی در برابر رفتار مورد انتظار
۴. مراحل بازتولید (برای باگ)
۵. محیط (نسخه، سیستم‌عامل)
۶. لاگ/اسکرین‌شات
۷. معیار پذیرش (Definition of Done)
```

#### عنوان: مهم‌ترین بخش

| ✗ | ✓ |
|---|---|
| `کار نمی‌کند` | `خطای 500 هنگام ثبت سفارش با کد تخفیف منقضی` |
| `باگ` | `جست‌وجو، نتایج فارسی را پیدا نمی‌کند` |
| `لطفاً اضافه کنید` | `افزودن خروجی Excel به گزارش فروش ماهانه` |

**فرمول:** `[محیط/بخش] — [رفتار مشاهده‌شده] در [شرایط]`

#### معیار پذیرش (Definition of Done)

بدون آن، Issue هیچ‌وقت تمام نمی‌شود. نمونه:

```markdown
## معیار پذیرش
- [ ] کاربر می‌تواند گزارش را به CSV دانلود کند
- [ ] فایل شامل ستون‌های: تاریخ، مبلغ، وضعیت
- [ ] برای بازه‌ی بیش از ۱۰۰۰ ردیف، خطا نمی‌دهد
- [ ] تست واحد برای تابع تبدیل
```

### مثال‌های عملی

#### مثال ۱: باگ‌ریپورت کامل

```markdown
**عنوان:** خطای 500 هنگام ثبت سفارش با کد تخفیف منقضی

## محیط
- نسخه: 2.4.1
- مرورگر: Chrome 130 / Windows 11
- محیط: production

## چه کاری می‌خواستم بکنم
ثبت سفارش با کد تخفیف `EXPIRED2025`.

## چه اتفاقی افتاد
صفحه‌ی سفید + خطای 500 در کنسول. سفارش ثبت نشد.

## چه انتظاری داشتم
پیام واضح: «کد تخفیف منقضی شده است» و امکان حذف کد و ادامه.

## مراحل بازتولید
1. محصولی به سبد اضافه کنید
2. در صفحه‌ی پرداخت، کد `EXPIRED2025` را وارد کنید
3. روی «ثبت سفارش» بزنید

## لاگ
```
TypeError: Cannot read properties of null (reading 'discount')
    at calculateTotal (src/cart/total.ts:42:18)
    at POST /api/orders (src/api/orders.ts:88:5)
```

## اثر
- کاربران: ~۳٪ از سفارش‌ها (کدهای منقضی)
- شدت: بالا — سفارش ثبت نمی‌شود

## راه‌حل پیشنهادی (اختیاری)
در `calculateTotal`، قبل از دسترسی به `discount`، وجود کد را چک کنید.
```

**چرا این خوب است؟** نگه‌دارنده می‌تواند بدون هیچ سؤالی بازتولید کند و حتی محل باگ را می‌بیند.

#### مثال ۲: درخواست ویژگی خوب

```markdown
**عنوان:** افزودن خروجی Excel به گزارش فروش ماهانه

## مسئله
هر ماه باید گزارش فروش را دستی از پنل کپی کنم و در Excel بچسبانم.
برای ۱۲ شعبه، حدود ۲ ساعت کار است و خطای انسانی دارد.

## راه‌حل پیشنهادی
دکمه‌ی «دانلود Excel» در بالای گزارش که فایل `.xlsx` با
همان ستون‌های نمایش‌داده‌شده تولید کند.

## جایگزین‌هایی که بررسی کردم
- CSV: کار می‌کند ولی کاربران با اعداد فارسی در Excel مشکل دارند
- API: نیاز به توسعه‌ی جدا دارد

## معیار پذیرش
- [ ] دکمه در صفحه‌ی گزارش
- [ ] فایل شامل ستون‌های تاریخ، شعبه، مبلغ، تعداد
- [ ] اعداد با فرمت فارسی
- [ ] حداکثر ۱۰ ثانیه برای ۱۰٬۰۰۰ ردیف

## اولویت
متوسط — کار ماهانه است، نه روزانه.
```

#### مثال ۳: Issue داخلی برای تسک فنی

```markdown
**عنوان:** ارتقای Node.js از ۱۸ به ۲۲ در سرویس API

## چرا
Node 18 در آوریل ۲۰۲۵ EOL شد. بدون patch امنیتی.

## دامنه
- `services/api/`
- `Dockerfile`
- `.github/workflows/ci.yml`

## کارها
- [ ] به‌روزرسانی `engines` در package.json
- [ ] به‌روزرسانی Dockerfile
- [ ] به‌روزرسانی CI matrix
- [ ] اجرای تست‌ها
- [ ] بررسی وابستگی‌های ناسازگار

## ریسک
- `node-fetch` نسخه‌ی قدیمی — باید حذف شود (fetch بومی)
- `bcrypt` نیاز به rebuild دارد

## معیار پذیرش
- [ ] CI روی Node 22 سبز
- [ ] staging یک هفته بدون خطا
- [ ] Docker image جدید deploy شد
```

#### مثال ۴: Issue با gh

```bash
gh issue create \
  --title "خطای 500 هنگام ثبت سفارش با کد تخفیف منقضی" \
  --body-file bug-report.md \
  --label "bug,severity:high" \
  --assignee maryam-ahmadi \
  --milestone "v2.4.2"

# لیست
gh issue list --label bug --state open
gh issue view 42
gh issue view 42 --web

# بستن
gh issue close 42 --comment "در نسخه 2.4.2 رفع شد"
```

#### مثال ۵: جست‌وجوی Issue

```
is:issue is:open label:bug
is:issue assignee:@me
is:pr is:open review-requested:@me
is:issue created:>2026-10-01
is:issue label:bug no:assignee
is:issue in:title "خطای 500"
is:issue comments:>10
```

این syntax در نوار جست‌وجوی GitHub و در `gh issue list --search` کار می‌کند.

### مقایسه و الگوهای مختلف

#### Issue در برابر Discussion در برابر PR

| | Issue | Discussion | PR |
|---|---|---|---|
| کار قابل بستن | ✅ | ❌ | ✅ |
| کد | ❌ | ❌ | ✅ |
| گفت‌وگوی بلند | متوسط | ✅ | متوسط |
| رأی‌گیری | 👍 (reaction) | ✅ (poll) | ❌ |
| در Projects | ✅ | محدود | ✅ |

**قاعده:** اگر جواب «بله/خیر» یا «انجام شد/نشد» دارد → Issue. اگر گفت‌وگوی باز است → Discussion.

#### عنوان: سه سبک

| سبک | نمونه | مناسب |
|---|---|---|
| Conventional | `fix(cart): خطای 500 با کد منقضی` | تیم فنی |
| طبیعی | `خطای 500 با کد تخفیف منقضی` | کاربران عادی |
| Ticket-style | `[CART-123] خطای 500` | اتصال به Jira |

### اشکال شایع

#### ۱. «کار نمی‌کند»

بدون جزئیات. نگه‌دارنده نمی‌تواند کاری کند.

#### ۲. چند مسئله در یک Issue

```
✗ عنوان: چند باگ در صفحه‌ی پرداخت
   - دکمه کار نمی‌کند
   - فونت اشتباه است
   - لاگ خطا می‌دهد
```

نمی‌شود بست، نمی‌شود اولویت‌بندی کرد. **یک Issue، یک مسئله.**

#### ۳. Issue بدون معیار پذیرش

هیچ‌وقت «تمام» نمی‌شود، یا با بحث دوباره باز می‌شود.

#### ۴. اسکرین‌شات بدون متن

تصویر خوب است ولی قابل جست‌وجو نیست. متن + تصویر.

#### ۵. گذاشتن اطلاعات حساس در Issue

```
✗ توکن: ghp_xxxxxxxx
✗ ایمیل کاربر: ali@example.com
✗ connection string با پسورد
```

Issue در مخزن عمومی برای همه قابل خواندن است. حتی در مخزن خصوصی، در ایمیل اطلاع‌رسانی کپی می‌شود.

#### ۶. Issueای که سال‌ها باز می‌ماند

اگر قرار نیست انجام شود، ببند با `wontfix` یا `not planned`. Issueهای باز بی‌پایان، اعتماد را از بین می‌برند.

#### ۷. استفاده از Issue برای پرسش

«چطور این را نصب کنم؟» → Discussions یا Stack Overflow.

### نکات مهم (Tips & Tricks)

- **عنوان را بعد از نوشتن بدنه بازنویسی کن.** بعد از فهمیدن مسئله، عنوان دقیق‌تری می‌نویسی.
- **`--body-file`** در gh، برای بدنه‌ی طولانی راحت‌تر از `--body` است.
- **در باگ، همیشه «چه انتظاری داشتی» را بنویس.** نصف سوءتفاهم‌ها همین‌جاست.
- **`is:issue no:assignee label:bug`** برای پیدا کردن Issueهای رهاشده.
- **در Issue template، فیلد «نسخه» را اجباری کن** — بیشترین سؤال تکراری.
- **اگر گزارش‌دهنده اطلاعات نداد، برچسب `needs-info` بزن** و بعد از ۱۴ روز ببند.

### خلاصه فصل

- Issue خوب: عنوان گویا + زمینه + رفتار فعلی/انتظاری + بازتولید + محیط + معیار پذیرش.
- **یک Issue، یک مسئله.**
- معیار پذیرش (DoD) بدون آن، Issue تمام نمی‌شود.
- `Closes #N` در PR، Issue را خودکار می‌بندد.
- جست‌وجوی `is:` syntax قدرتمند است.
- اطلاعات حساس را در Issue نگذار.

**سؤالات مروری:**
1. سه بخشی که هر باگ‌ریپورت باید داشته باشد چیست؟
2. چرا چند مسئله در یک Issue مشکل‌ساز است؟
3. چطور Issueهای بدون assignee را پیدا کنی؟

---

## فصل ۳۰: برچسب، milestone، assignee و issue types

### مقدمه

وقتی ۵۰ Issue باز داری، فقط با عنوان نمی‌توانی مدیریت کنی. سه ابزار ساده — label، milestone، assignee — و یک ابزار جدیدتر (issue types) به تو اجازه می‌دهند از «انبار» به «سیستم» برسی.

**چرا مهم است؟** تیمی که label ندارد، هر هفته درباره‌ی اولویت‌ها بحث می‌کند. تیمی که label دارد، در ۳۰ ثانیه می‌فهمد چه چیزی فوری است.

**کاربرد عملی:** یک مجموعه‌ی label خوب + یک milestone برای هر نسخه = مدیریت پروژه بدون ابزار اضافی.

### مفاهیم اساسی

#### Label — دسته‌بندی

پیش‌فرض‌های GitHub (ضعیف‌اند):

```
bug، documentation، duplicate، enhancement، good first issue، help wanted، invalid، question، wontfix
```

یک مجموعه‌ی خوب:

| گروه | Label | رنگ |
|---|---|---|
| **نوع** | `bug` | `#d73a4a` |
| | `enhancement` | `#a2eeef` |
| | `documentation` | `#0075ca` |
| | `refactor` | `#cfd3d7` |
| | `test` | `#c2e0c6` |
| **اولویت** | `priority:critical` | `#b60205` |
| | `priority:high` | `#d93f0b` |
| | `priority:medium` | `#fbca04` |
| | `priority:low` | `#0e8a16` |
| **وضعیت** | `status:blocked` | `#e99695` |
| | `status:needs-info` | `#fef2c0` |
| | `status:in-review` | `#c5def5` |
| **مهارت** | `good first issue` | `#7057ff` |
| | `help wanted` | `#008672` |
| **دامنه** | `area:api` | `#1d76db` |
| | `area:ui` | `#5319e7` |
| | `area:infra` | `#006b75` |

#### Milestone — نسخه یا بازه

یک milestone مجموعه‌ای از Issue/PR است که باید تا یک تاریخ تمام شوند:

```
v2.4.2 — موعد: ۲۰۲۶-۱۱-۰۵ — ۸ از ۱۲ بسته (۶۷٪)
```

GitHub نوار پیشرفت خودکار نشان می‌دهد.

#### Assignee

فرد مسئول. **قاعده:** هر Issue باز باید حداکثر یک assignee داشته باشد (مالکیت مشخص). بیش از یکی = هیچ‌کس مسئول نیست.

#### Issue types (قابلیت جدیدتر)

GitHub در حال جایگزینی «type» با label است:

| Type | معنی |
|---|---|
| **Bug** | چیزی خراب است |
| **Feature** | چیزی جدید |
| **Task** | کار داخلی |

مزیت: type در جست‌وجو و فیلتر یک فیلد جداگانه است، نه یک label شلوغ. اگر در سازمانت فعال است، از آن استفاده کن.

### مثال‌های عملی

#### مثال ۱: ساخت labelها با gh

```bash
# نوع
gh label create "bug" --color d73a4a --description "چیزی درست کار نمی‌کند"
gh label create "enhancement" --color a2eeef --description "ویژگی جدید"
gh label create "documentation" --color 0075ca --description "مستندات"

# اولویت
gh label create "priority:critical" --color b60205 --description "فوری، production خراب"
gh label create "priority:high" --color d93f0b --description "این هفته"
gh label create "priority:medium" --color fbca04 --description "این ماه"
gh label create "priority:low" --color 0e8a16 --description "هر وقت شد"

# وضعیت
gh label create "status:blocked" --color e99695 --description "منتظر چیز دیگر"
gh label create "status:needs-info" --color fef2c0 --description "اطلاعات ناقص"

# دامنه
gh label create "area:api" --color 1d76db
gh label create "area:ui" --color 5319e7
gh label create "area:infra" --color 006b75

# لیست
gh label list
```

#### مثال ۲: اسکریپت راه‌اندازی label برای مخزن جدید

`scripts/setup-labels.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

REPO="${1:?usage: $0 owner/repo}"

create() {
  gh label create "$1" --repo "$REPO" --color "$2" --description "$3" --force
}

create "bug"                "d73a4a" "چیزی درست کار نمی‌کند"
create "enhancement"        "a2eeef" "ویژگی جدید"
create "documentation"      "0075ca" "مستندات"
create "priority:critical"  "b60205" "production خراب"
create "priority:high"      "d93f0b" "این هفته"
create "priority:medium"    "fbca04" "این ماه"
create "priority:low"       "0e8a16" "هر وقت شد"
create "status:blocked"     "e99695" "منتظر وابستگی"
create "status:needs-info"  "fef2c0" "اطلاعات ناقص"
create "good first issue"   "7057ff" "مناسب تازه‌کار"

echo "labels برای $REPO ساخته شد"
```

`--force` باعث می‌شود اگر label وجود داشت، به‌روز شود.

#### مثال ۳: Milestone

```bash
# با API (gh milestone وجود ندارد، از api استفاده کن)
gh api repos/OWNER/REPO/milestones \
  -f title="v2.4.2" \
  -f description="رفع باگ‌های بحرانی" \
  -f due_on="2026-11-05T00:00:00Z" \
  -f state="open"

# لیست
gh api repos/OWNER/REPO/milestones --jq '.[] | "\(.title): \(.open_issues)/\(.open_issues + .closed_issues)"'
```

#### مثال ۴: اختصاص Issue

```bash
gh issue edit 42 --add-assignee maryam-ahmadi --add-label "priority:high"
gh issue edit 42 --milestone "v2.4.2"
```

#### مثال ۵: دیدن کارهای خودت

```bash
gh issue list --assignee @me --state open
gh pr list --assignee @me --state open

# همه‌چیز در یک نگاه
gh search issues --assignee @me --state open --limit 50
```

#### مثال ۶: باگ بحرانی — جریان کامل

```bash
# ۱) Issue بساز
gh issue create --title "خطای 500 در پرداخت" --label "bug,priority:critical" --body "..."

# ۲) به خودت اختصاص بده
gh issue edit 101 --add-assignee @me

# ۳) branch بساز
gh issue develop 101 --checkout
# branch با نام 101-... ساخته و checkout می‌شود

# ۴) fix، commit، PR
gh pr create --fill

# ۵) merge → Issue خودکار بسته می‌شود (اگر Closes #101 در توضیح باشد)
```

`gh issue develop` یکی از بهترین قابلیت‌های gh است: branch را با شماره‌ی Issue لینک می‌کند.

### مقایسه و الگوهای مختلف

#### Label در برابر Projects field

| | Label | Project field |
|---|---|---|
| دامنه | کل مخزن | یک Project |
| فیلتر در Issue list | ✅ | ❌ (باید در Project ببینی) |
| چند مقدار | ✅ (چند label) | معمولاً یک |
| مناسب | دسته‌بندی سراسری | وضعیت گردش کار |

**توصیه:** label برای «نوع/اولویت/دامنه»، Project field برای «وضعیت در تخته» (Todo/In Progress/Done).

#### Milestone در برابر Release

| | Milestone | Release |
|---|---|---|
| Issue/PR | ✅ | ❌ (به تگ وصل است) |
| تاریخ | ✅ | ❌ (تاریخ انتشار) |
| فایل | ❌ | ✅ (asset) |
| مناسب | برنامه‌ریزی | انتشار |

#### Assignee: یک نفر در برابر چند نفر

| | یک نفر | چند نفر |
|---|---|---|
| مسئولیت | ✅ واضح | ❌ مبهم |
| نمایش در Projects | ساده | پیچیده |
| توصیه | ✅ | فقط برای جفت‌کاری (با comment توضیح بده) |

### اشکال شایع

#### ۱. ۴۰ label بی‌نظم

اگر labelها را با حروف بزرگ/کوچک و فاصله‌های متفاوت بسازی (`Bug`، `bug`، `BUG`)، جست‌وجو خراب می‌شود. **قاعده:** همه lowercase، با `:` برای گروه‌بندی.

#### ۲. Milestone بدون تاریخ

milestone بدون `due_on` فقط یک برچسب است. تاریخ بگذار (حتی تقریبی).

#### ۳. Milestone با ۱۰۰ Issue

اگر milestone بیش از ۲۰-۳۰ آیتم دارد، احتمالاً یک «نسخه» نیست، یک «آرزو» است. بشکن.

#### ۴. Issue بدون assignee در Sprint

در Projects، Issue بدون assignee یعنی «هیچ‌کس مسئول نیست» — که در عمل یعنی «انجام نمی‌شود».

#### ۵. `good first issue` روی Issueی که سخت است

اگر تازه‌کار بیاید و ۳ روز گیر کند، تجربه‌ی بدی می‌گیرد و دیگر مشارکت نمی‌کند. این label مسئولیت دارد.

#### ۶. حذف label به‌جای تغییر نام

اگر label را حذف کنی، از همه‌ی Issueها پاک می‌شود. تغییر نام (`gh label edit old --name new`) تاریخ را حفظ می‌کند.

### نکات مهم (Tips & Tricks)

- **`gh label clone SOURCE/REPO`** — labelهای یک مخزن را به مخزن دیگر کپی می‌کند. برای راه‌اندازی سریع عالی است.
- **`priority:` با `:`** بهتر از `priority-high` است چون در جست‌وجو `label:priority:*` کار می‌کند.
- **در Projects، یک view برای «بدون assignee»** بساز — هشدار زودهنگام.
- **Milestone را در `gh release create` استفاده کن** تا release notes خودکار ساخته شود.
- **`gh issue develop`** بهترین راه اتصال Issue به branch.

### خلاصه فصل

- **Label** برای دسته‌بندی: نوع، اولویت، وضعیت، دامنه — با پیشوند گروه.
- **Milestone** برای نسخه/بازه با تاریخ و نوار پیشرفت.
- **Assignee** یک نفر، برای مسئولیت واضح.
- **Issue types** جایگزین مدرن label برای نوع است.
- `gh issue develop` Issue را به branch وصل می‌کند.

**سؤالات مروری:**
1. چرا `priority:high` بهتر از `High Priority` است؟
2. تفاوت milestone و release چیست؟
3. چرا هر Issue باید حداکثر یک assignee داشته باشد؟

---

## فصل ۳۱: GitHub Projects — تخته، جدول و roadmap

### مقدمه

Projects یک ابزار مدیریت کار است که مستقیم روی Issue و PR سوار می‌شود. سه نمای اصلی دارد: **Table** (مثل Excel)، **Board** (کانبان)، **Roadmap** (تایم‌لاین). همه از یک داده‌ی واحد ساخته می‌شوند.

**چرا مهم است؟** تیم‌هایی که Projects را جدی می‌گیرند، ابزار جدا (Jira، Trello) لازم ندارند. و مزیت اصلی: Issue و PR **خودشان** در تخته ظاهر می‌شوند — بدون کپی دستی.

**کاربرد عملی:** یک Project با ۳ نما، جای یک ابزار مدیریت پروژه را می‌گیرد.

### مفاهیم اساسی

#### ساختار

```
Project (در سطح organization یا کاربر)
├── Items (Issue، PR، draft issue)
├── Fields (Status، Assignee، Priority، Estimate، ...)
└── Views (Table، Board، Roadmap + فیلتر/گروه‌بندی)
```

#### سه نوع نما

| نما | شکل | مناسب |
|---|---|---|
| **Table** | جدول با ستون‌های قابل تنظیم | کار روزمره، فیلتر دقیق |
| **Board** | ستون‌های کانبان | جریان کار، وضعیت |
| **Roadmap** | تایم‌لاین افقی | برنامه‌ریزی بلندمدت |

#### فیلدها

| فیلد | نوع | نمونه |
|---|---|---|
| `Status` | single select | Todo / In Progress / In Review / Done |
| `Priority` | single select | P0 / P1 / P2 / P3 |
| `Size` | single select | XS / S / M / L / XL |
| `Estimate` | number | ۳ (روز) |
| `Sprint` | iteration | Sprint 12 |
| `Target date` | date | ۲۰۲۶-۱۱-۰۵ |
| `Notes` | text | یادداشت آزاد |

#### Draft issue

آیتمی که هنوز Issue واقعی نیست — فقط در Project وجود دارد. برای ایده‌های خام عالی است. با یک کلیک به Issue تبدیل می‌شود.

### مثال‌های عملی

#### مثال ۱: ساخت Project با gh

```bash
# Project در سطح organization
gh project create --owner myorg --title "نقشه راه محصول"

# لیست
gh project list --owner myorg

# شماره‌ی Project را بگیر (مثلاً 5)
```

#### مثال ۲: افزودن Issue به Project

```bash
# با شماره
gh project item-add 5 --owner myorg --url https://github.com/myorg/repo/issues/42

# با gh issue
gh issue create --title "..." --project "نقشه راه محصول"
```

#### مثال ۳: تنظیم فیلدها با GraphQL

Projects API از GraphQL استفاده می‌کند (REST محدود است):

```bash
# گرفتن ID فیلدها
gh api graphql -f query='
query {
  organization(login: "myorg") {
    projectV2(number: 5) {
      id
      fields(first: 20) {
        nodes {
          ... on ProjectV2Field { id name }
          ... on ProjectV2SingleSelectField { id name options { id name } }
        }
      }
    }
  }
}'
```

```bash
# تنظیم Status یک آیتم
gh api graphql -f query='
mutation {
  updateProjectV2ItemFieldValue(input: {
    projectId: "PVT_xxx"
    itemId: "PVTI_xxx"
    fieldId: "PVTSSF_xxx"
    value: { singleSelectOptionId: "option-in-progress" }
  }) {
    projectV2Item { id }
  }
}'
```

**نکته:** API Projects پیچیده است. برای کار روزمره از UI یا از actionهای آماده استفاده کن.

#### مثال ۴: نمای پیشنهادی برای تیم

```
View 1: "Board — Sprint جاری"
  Layout: Board
  Group by: Status
  Filter: sprint:@current

View 2: "Table — همه‌ی کارهای باز"
  Layout: Table
  Columns: Title, Status, Assignee, Priority, Target date
  Filter: -status:Done

View 3: "Roadmap — سه ماه آینده"
  Layout: Roadmap
  Date field: Target date
  Filter: status:Todo,In Progress

View 4: "My work"
  Layout: Table
  Filter: assignee:@me -status:Done
```

#### مثال ۵: تبدیل draft به Issue

در UI: روی draft کلیک → **Convert to issue** → انتخاب مخزن. تمام فیلدها حفظ می‌شوند.

#### مثال ۶: اتوماسیون داخلی Projects

Projects خودش چند automation ساده دارد (در منوی `...` → **Workflows**):

- Item closed → Status: Done
- Item reopened → Status: Todo
- Pull request merged → Status: Done
- Auto-add items from a repository (با فیلتر label)

این‌ها بدون کد کار می‌کنند و ۸۰٪ نیازها را پوشش می‌دهند.

### مقایسه و الگوهای مختلف

#### Projects در برابر Issue list

| | Issue list | Projects |
|---|---|---|
| فیلتر | ✅ | ✅ |
| فیلد سفارشی | ❌ | ✅ |
| نمای کانبان | ❌ | ✅ |
| نمای تایم‌لاین | ❌ | ✅ |
| آیتم بدون Issue | ❌ | ✅ (draft) |
| چند مخزن | ❌ | ✅ |

#### Projects در برابر Jira

| | Projects | Jira |
|---|---|---|
| راه‌اندازی | صفر | نیاز به تنظیم |
| یکپارچگی با کد | ✅ بومی | ✅ (با plugin) |
| گزارش‌گیری | محدود | قوی |
| پیچیدگی | کم | زیاد |
| مناسب | تیم‌های کوچک/متوسط | سازمان‌های بزرگ |

#### Board در برابر Table

| | Board | Table |
|---|---|---|
| دید کلی | ✅ | متوسط |
| جزئیات | ❌ | ✅ |
| برای standup | ✅ | ✅ |
| برای گزارش | ❌ | ✅ |

**توصیه:** Board برای جلسه، Table برای کار روزمره.

### اشکال شایع

#### ۱. Project بدون فیلد Status

اگر Status نداشته باشی، Board بی‌معنی است. Status اولین فیلد است.

#### ۲. ۵۰ آیتم در «In Progress»

یعنی WIP limit ندارد. قاعده: هر نفر حداکثر ۲ آیتم هم‌زمان در In Progress.

#### ۳. Project در سطح کاربر، نه سازمان

اگر Project شخصی بسازی و تیم عوض شود، دسترسی از دست می‌رود. برای تیم، Project سازمانی بساز.

#### ۴. کپی دستی Issue در Project

اگر دستی اضافه کنی، وقتی Issue بسته شود، آیتم Project به‌روز نمی‌شود. از auto-add workflow استفاده کن.

#### ۵. استفاده از Projects به‌جای Issue

Project جای Issue نیست. آیتم باید Issue باشد تا در مخزن قابل ارجاع باشد. Draft فقط برای ایده‌ی خام.

#### ۶. فراموش کردن visibility

Project می‌تواند public یا private باشد. Project عمومی + Issue خصوصی = نشت اطلاعات.

### نکات مهم (Tips & Tricks)

- **Auto-add workflow** را روشن کن: هر Issue با label `bug` خودکار به Project اضافه شود.
- **`assignee:@me`** در فیلتر، نمای شخصی می‌سازد — بیشترین استفاده را دارد.
- **Iteration field** برای sprintها بهتر از date است (خودکار جلو می‌رود).
- **در Roadmap، فقط آیتم‌های با Target date نمایش داده می‌شوند** — بدون تاریخ، خالی است.
- **برای گزارش هفتگی، نمای Table با `Group by: Assignee`** عالی است.
- **`gh project item-list 5 --owner myorg --format json`** برای خروجی گرفتن.

### خلاصه فصل

- Projects سه نما دارد: Table، Board، Roadmap — از یک داده.
- فیلدهای سفارشی (Status، Priority، Sprint) قلب Projects هستند.
- **Auto-add workflow** Issueها را خودکار وارد می‌کند.
- Draft issue برای ایده‌ی خام؛ تبدیل به Issue با یک کلیک.
- برای تیم، Project سازمانی بساز نه شخصی.

**سؤالات مروری:**
1. تفاوت Table و Board چیست و کدام برای جلسه بهتر است؟
2. چرا کپی دستی Issue در Project بد است؟
3. چطور کارهای خودت را در Projects فیلتر کنی؟

---

## فصل ۳۲: خودکارسازی Issue و Projects

### مقدمه

کارهای تکراری در مدیریت Issue — برچسب زدن، اختصاص دادن، پیام خوش‌آمد، بستن Issueهای قدیمی — می‌توانند خودکار شوند. GitHub سه راه دارد: **Projects workflows** (بدون کد)، **Actions** (کد)، و **`gh` + cron** (اسکریپت).

**چرا مهم است؟** یک تیم ۵ نفره با ۵۰ Issue در ماه، حدود ۴ ساعت در ماه صرف کارهای تکراری می‌کند. خودکارسازی این را به صفر می‌رساند.

**کاربرد عملی:** سه الگوی زیر ۹۰٪ نیازها را پوشش می‌دهد.

### مفاهیم اساسی

#### سه لایه‌ی خودکارسازی

| لایه | ابزار | نیاز به کد | مناسب |
|---|---|---|---|
| **Projects workflows** | UI | ❌ | auto-add، status خودکار |
| **Actions** | YAML | ✅ کم | برچسب، اختصاص، پیام |
| **Script + cron** | bash/python | ✅ | گزارش، پاکسازی دوره‌ای |

#### رویدادهای مفید برای Actions

```yaml
on:
  issues:
    types: [opened, labeled, closed, reopened, assigned]
  pull_request:
    types: [opened, synchronize, closed]
  issue_comment:
    types: [created]
  schedule:
    - cron: "0 6 * * 1"    # دوشنبه‌ها ۶ صبح
  workflow_dispatch:        # دستی
```

### مثال‌های عملی

#### مثال ۱: خوش‌آمد به اولین Issue

`.github/workflows/welcome.yml`:

```yaml
name: خوش‌آمد

on:
  issues:
    types: [opened]
  pull_request_target:
    types: [opened]

permissions:
  issues: write
  pull-requests: write

jobs:
  welcome:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            const isIssue = !!context.payload.issue;
            const author = isIssue
              ? context.payload.issue.user.login
              : context.payload.pull_request.user.login;

            // فقط برای اولین مشارکت
            const { data: issues } = await github.rest.search.issuesAndPullRequests({
              q: `repo:${context.repo.owner}/${context.repo.repo} author:${author} type:issue`,
            });
            if (issues.total_count > 1) return;

            const body = isIssue
              ? `سلام @${author}! 👋\n\nممنون که Issue باز کردی. یک نفر از تیم تا ۴۸ ساعت آینده جواب می‌دهد.\n\nدر این فاصله، [CONTRIBUTING.md](../blob/main/CONTRIBUTING.md) را ببین.`
              : `سلام @${author}! 👋\n\nممنون برای اولین PR. یک reviewer تا ۴۸ ساعت آینده آن را می‌بیند.`;

            if (isIssue) {
              await github.rest.issues.createComment({
                ...context.repo, issue_number: context.payload.issue.number, body,
              });
            } else {
              await github.rest.issues.createComment({
                ...context.repo, issue_number: context.payload.pull_request.number, body,
              });
            }
```

**نکته‌ی امنیتی:** `pull_request_target` در اینجا امن است چون کد PR را checkout یا اجرا نمی‌کنیم — فقط API صدا می‌زنیم. فصل ۵۴.

#### مثال ۲: برچسب خودکار بر اساس محتوا

```yaml
name: برچسب خودکار

on:
  issues:
    types: [opened, edited]

permissions:
  issues: write

jobs:
  label:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            const issue = context.payload.issue;
            const text = `${issue.title}\n${issue.body || ""}`.toLowerCase();
            const labels = [];

            if (/\b(خطا|error|bug|crash|broken|نمیشود|کار نمیکند)\b/.test(text)) labels.push("bug");
            if (/\b(اضافه|افزودن|feature|پیشنهاد|میخواهم)\b/.test(text)) labels.push("enhancement");
            if (/\b(مستند|docs|readme|توضیح)\b/.test(text)) labels.push("documentation");
            if (/\b(امنیت|security|xss|sql injection|نشت)\b/.test(text)) labels.push("priority:critical");

            if (labels.length) {
              await github.rest.issues.addLabels({
                ...context.repo,
                issue_number: issue.number,
                labels: [...new Set(labels)],
              });
            }
```

#### مثال ۳: بستن خودکار Issueهای قدیمی

```yaml
name: پاکسازی Issueهای قدیمی

on:
  schedule:
    - cron: "0 3 * * 1"   # دوشنبه‌ها ۳ بامداد UTC
  workflow_dispatch:

permissions:
  issues: write
  pull-requests: write

jobs:
  stale:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/stale@v9
        with:
          days-before-stale: 60
          days-before-close: 14
          stale-issue-label: "status:needs-info"
          stale-issue-message: |
            این Issue ۶۰ روز بدون فعالیت بوده. اگر هنوز مرتبط است،
            لطفاً یک کامنت بگذار. در غیر این صورت ۱۴ روز دیگر بسته می‌شود.
          close-issue-message: |
            به‌خاطر بی‌فعالیتی بسته شد. اگر لازم شد، دوباره باز کن.
          exempt-issue-labels: "priority:critical,priority:high,pinned"
          days-before-pr-stale: -1     # PRها را دست نزن
```

#### مثال ۴: اختصاص خودکار بر اساس label

```yaml
name: اختصاص خودکار

on:
  issues:
    types: [labeled]

permissions:
  issues: write

jobs:
  assign:
    if: github.event.label.name == 'area:api'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            await github.rest.issues.addAssignees({
              ...context.repo,
              issue_number: context.payload.issue.number,
              assignees: ["maryam-ahmadi"],
            });
```

#### مثال ۵: Project — auto-add

در UI: Project → `...` → **Workflows** → **Auto-add to project**:

```
Repository: myorg/backend
Filter: is:issue is:open label:bug
```

هر Issue با label `bug` خودکار به Project اضافه می‌شود.

#### مثال ۶: انتقال وضعیت بر اساس رویداد

در همان Workflows:

```
When: Pull request merged
Then: Set Status → Done

When: Issue closed
Then: Set Status → Done

When: Issue reopened
Then: Set Status → Todo
```

#### مثال ۷: گزارش هفتگی با اسکریپت

`scripts/weekly-report.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

REPO="${REPO:-myorg/backend}"
SINCE="$(date -d '7 days ago' +%Y-%m-%d 2>/dev/null || date -v-7d +%Y-%m-%d)"

echo "# گزارش هفتگی — $SINCE تا امروز"
echo

echo "## Issueهای بسته‌شده"
gh issue list --repo "$REPO" --state closed --search "closed:>=$SINCE" --limit 100 \
  --json number,title,closedAt \
  --jq '.[] | "- #\(.number) \(.title)"'

echo
echo "## PRهای merge‌شده"
gh pr list --repo "$REPO" --state merged --search "merged:>=$SINCE" --limit 100 \
  --json number,title,author \
  --jq '.[] | "- #\(.number) \(.title) (@\(.author.login))"'

echo
echo "## Issueهای باز بدون assignee"
gh issue list --repo "$REPO" --state open --search "no:assignee" --limit 50 \
  --json number,title \
  --jq '.[] | "- #\(.number) \(.title)"'
```

و یک workflow که هر دوشنبه اجرا می‌کند و به Slack می‌فرستد (یا به Issue پین‌شده کامنت می‌زند).

#### مثال ۸: بررسی معیار پذیرش

```yaml
name: چک Issue

on:
  issues:
    types: [opened]

permissions:
  issues: write

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            const body = context.payload.issue.body || "";
            const problems = [];

            if (body.length < 100) problems.push("توضیح خیلی کوتاه است (حداقل ۱۰۰ کاراکتر)");
            if (!/نسخه|version/i.test(body)) problems.push("نسخه ذکر نشده");
            if (!/مراحل|steps/i.test(body) && /bug/i.test(context.payload.issue.title)) {
              problems.push("مراحل بازتولید نوشته نشده");
            }

            if (problems.length) {
              await github.rest.issues.createComment({
                ...context.repo,
                issue_number: context.payload.issue.number,
                body: `ممنون! برای اینکه بتوانیم بررسی کنیم، این موارد را اضافه کن:\n\n${problems.map(p => `- ${p}`).join("\n")}`,
              });
              await github.rest.issues.addLabels({
                ...context.repo,
                issue_number: context.payload.issue.number,
                labels: ["status:needs-info"],
              });
            }
```

### مقایسه و الگوهای مختلف

#### Actions در برابر Projects workflows

| | Actions | Projects workflows |
|---|---|---|
| قدرت | نامحدود | محدود |
| نیاز به کد | ✅ | ❌ |
| زمان اجرا | ثانیه | فوری |
| مناسب | منطق پیچیده | تغییر وضعیت |

**توصیه:** اول Projects workflows را امتحان کن. اگر کافی نبود، Actions.

#### `issues` در برابر `issue_comment`

| Trigger | کِی اجرا می‌شود |
|---|---|
| `issues: [opened]` | Issue جدید |
| `issue_comment: [created]` | کامنت جدید |
| `issues: [labeled]` | برچسب اضافه شد |

#### Slash commands با Actions

می‌توانی دستورهای متنی بسازی:

```yaml
on:
  issue_comment:
    types: [created]

jobs:
  command:
    if: startsWith(github.event.comment.body, '/assign')
    runs-on: ubuntu-latest
    permissions:
      issues: write
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            const user = context.payload.comment.body.replace('/assign', '').trim();
            if (!user) return;
            await github.rest.issues.addAssignees({
              ...context.repo,
              issue_number: context.payload.issue.number,
              assignees: [user],
            });
```

**هشدار امنیتی:** هر کسی می‌تواند کامنت بگذارد. قبل از اجرای هر دستوری، مجوز کاربر را چک کن:

```javascript
const { data } = await github.rest.repos.getCollaboratorPermissionLevel({
  ...context.repo, username: context.payload.comment.user.login,
});
if (!["admin", "write", "maintain"].includes(data.permission)) return;
```

### اشکال شایع

#### ۱. حلقه‌ی بی‌پایان

workflow که Issue را برچسب می‌زند و برچسب، workflow را trigger می‌کند:

```
on: issues: [labeled] → addLabels → labeled → ...
```

راه‌حل: شرط `if` بگذار که فقط روی label مشخص اجرا شود، یا از `GITHUB_TOKEN` (که خودش trigger نمی‌کند) استفاده کن.

**نکته:** کارهایی که با `GITHUB_TOKEN` انجام می‌شوند، workflow جدید trigger **نمی‌کنند** (محافظت داخلی GitHub). ولی اگر از PAT استفاده کنی، حلقه می‌شود.

#### ۲. دسترسی کافی نیست

```
Resource not accessible by integration
```

`permissions:` را در workflow تنظیم کن:

```yaml
permissions:
  issues: write
```

#### ۳. `pull_request_target` + checkout کد PR = فاجعه

```yaml
✗ on: pull_request_target
  steps:
    - uses: actions/checkout@v4
      with: { ref: ${{ github.event.pull_request.head.sha }} }
    - run: npm test    # ← کد مهاجم اجرا می‌شود با دسترسی secrets
```

فصل ۵۴ کامل.

#### ۴. برچسب خودکار اشتباه

اگر regex خیلی وسیع باشد، همه‌چیز `bug` می‌شود. با `\b` (word boundary) دقیق‌تر کن و تست کن.

#### ۵. Stale action همه‌چیز را می‌بندد

اگر `exempt-issue-labels` را تنظیم نکنی، Issueهای مهم هم بسته می‌شوند. `priority:critical` و `pinned` را استثنا کن.

#### ۶. cron در GitHub Actions دقیق نیست

```
schedule: - cron: "0 3 * * 1"
```

ممکن است ۱۰-۳۰ دقیقه تأخیر داشته باشد (در ساعات شلوغ تا ۱ ساعت). برای کارهای حساس به زمان، cron را جای دیگر بگذار.

### نکات مهم (Tips & Tricks)

- **`actions/github-script`** ساده‌ترین راه نوشتن منطق در workflow است (JavaScript بدون setup).
- **`workflow_dispatch`** را به همه‌ی workflowهای نگهداری اضافه کن تا دستی اجرا شوند.
- **`permissions` را همیشه صریح بنویس** — پیش‌فرض ممکن است بیش از حد باشد.
- **قبل از فعال کردن stale، ۲ هفته در حالت `dry-run`** (`debug-only: true`).
- **`gh issue list --json`** برای گزارش‌گیری، بسیار انعطاف‌پذیر است.
- **هر خودکارسازی را در `CONTRIBUTING.md` مستند کن** — وگرنه تیم گیج می‌شود.

### خلاصه فصل

- سه لایه: Projects workflows (بدون کد)، Actions (YAML)، اسکریپت + cron.
- `actions/github-script` برای منطق ساده بهترین است.
- **`permissions` را صریح تنظیم کن.**
- `GITHUB_TOKEN` workflow جدید trigger نمی‌کند؛ PAT می‌کند (و حلقه می‌سازد).
- `pull_request_target` + checkout کد PR = آسیب‌پذیری.
- Stale action با `exempt-labels` ایمن‌تر است.

**سؤالات مروری:**
1. چرا `GITHUB_TOKEN` حلقه‌ی بی‌پایان نمی‌سازد؟
2. چطور به اولین مشارکت‌کننده پیام خوش‌آمد بفرستی؟
3. خطر `pull_request_target` با checkout چیست؟

---

**بعدی:** [`08-Pull-Requests-and-Review.md`](08-Pull-Requests-and-Review.md)

---

## منابع و مراجع

- GitHub Docs: About issues — https://docs.github.com/en/issues
- GitHub Docs: Managing labels — https://docs.github.com/en/issues/using-labels-and-milestones-to-track-work/managing-labels
- GitHub Docs: About Projects — https://docs.github.com/en/issues/planning-and-tracking-with-projects
- GitHub Docs: Projects automation — https://docs.github.com/en/issues/planning-and-tracking-with-projects/automating-your-project
- `actions/stale` — https://github.com/actions/stale
- `actions/github-script` — https://github.com/actions/github-script
- GitHub Docs: Issue forms — https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests
