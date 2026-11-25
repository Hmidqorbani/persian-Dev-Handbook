# بخش ۰۸: Pull Request و Code Review حرفه‌ای

**مدت یادگیری**: ۵ تا ۶ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۰۴ (اولین PR)، بخش ۰۷ (Issue)

---

## فهرست محتوا

- [فصل ۳۳: چرخه‌ی حیات PR — از draft تا merge](#فصل-۳۳)
- [فصل ۳۴: Code Review مؤثر — چه چیزی را ببینیم](#فصل-۳۴)
- [فصل ۳۵: ابزارهای review — suggestion، batch، resolve](#فصل-۳۵)
- [فصل ۳۶: Merge Queue و PRهای پشت‌سرهم](#فصل-۳۶)
- [فصل ۳۷: PRهای بزرگ — چرا بد هستند و چطور بشکنیم](#فصل-۳۷)

---

## فصل ۳۳: چرخه‌ی حیات PR — از draft تا merge

### مقدمه

در فصل ۱۸ PR اولت را ساختی. حالا وقت آن است که چرخه‌ی کامل را ببینی: چه اتفاقی از لحظه‌ی «Create pull request» تا «Merged» می‌افتد، چه وضعیت‌هایی وجود دارد، و چه چیزی می‌تواند گیر کند.

**چرا مهم است؟** ۹۰٪ «PR من گیر کرده»ها یکی از ده وضعیت شناخته‌شده است. اگر وضعیت‌ها را بشناسی، در ۳۰ ثانیه می‌فهمی چه باید بکنی.

**کاربرد عملی:** بعد از این فصل می‌توانی هر PR را از هر وضعیتی که هست، به merge برسانی.

### مفاهیم اساسی

#### وضعیت‌های PR

```
Draft
  ↓ (Ready for review)
Open
  ├── Checks pending  → منتظر Actions
  ├── Checks failed   → باید fix کنی
  ├── Conflicts       → باید rebase/merge کنی
  ├── Review required → منتظر reviewer
  ├── Changes requested → باید اصلاح کنی
  └── Approved        → آماده‌ی merge
       ↓
  Merged  |  Closed (بدون merge)
```

#### جدول کامل وضعیت‌ها

| وضعیت | معنی | چه کنی |
|---|---|---|
| **Draft** | کار نیمه‌تمام | `gh pr ready` |
| **Open** | آماده‌ی بررسی | منتظر یا reviewer پیدا کن |
| **Checks pending** | Actions در حال اجرا | صبر کن |
| **Checks failed** | تست/لینت شکست | لاگ را ببین، fix کن |
| **Conflicts** | تعارض با base | rebase کن |
| **Review required** | Ruleset review اجباری کرده | reviewer اضافه کن |
| **Changes requested** | reviewer اصلاح خواسته | اصلاح کن، دوباره review بخواه |
| **Approved** | تأیید شده | merge کن |
| **Blocked** | شرطی برآورده نشده | پیام GitHub را بخوان |
| **Merged** | تمام | branch را پاک کن |
| **Closed** | رد شد | اگر لازم شد، reopen |

#### چه چیزی PR را «blocked» می‌کند

GitHub در پایین PR یک باکس نشان می‌دهد: **«Merging is blocked»** با دلیل. دلایل رایج:

1. Required review نداری
2. Required status check پاس نشده
3. Conflict داری
4. Branch protection: باید به‌روز باشد (`Require branches to be up to date`)
5. Merge queue فعال است و PR در صف است
6. Ruleset: امضا لازم است (commit signing)

### مثال‌های عملی

#### مثال ۱: بررسی وضعیت PR با gh

```bash
gh pr view 42
# title:	feat(profile): افزودن صفحه‌ی پروفایل
# state:	OPEN
# author:	ali
# labels:	feature
# assignees:	ali
# reviewers:	maryam-ahmadi (Pending)
# projects:	نقشه راه محصول
# milestone:	v2.4.2
# number:	42
# url:		https://github.com/myorg/backend/pull/42
# additions:	142
# deletions:	8
# auto-merge:	disabled
# --

# وضعیت checks
gh pr checks 42
# All checks were successful
# 0 failing, 3 successful, 0 skipped, and 1 pending

# نمای JSON برای اسکریپت
gh pr view 42 --json state,mergeable,mergeStateStatus,reviewDecision
# {"mergeStateStatus":"BLOCKED","mergeable":"MERGEABLE","reviewDecision":"REVIEW_REQUIRED","state":"OPEN"}
```

`mergeStateStatus` مهم‌ترین فیلد است:

| مقدار | معنی |
|---|---|
| `CLEAN` | آماده‌ی merge |
| `BLOCKED` | شرطی برآورده نشده |
| `BEHIND` | base جلو رفته، باید sync کنی |
| `DIRTY` | conflict |
| `UNSTABLE` | checks در حال اجرا یا شکست |
| `DRAFT` | draft است |
| `HAS_HOOKS` | pre-receive hook |

#### مثال ۲: از Draft به Ready

```bash
gh pr create --draft --title "WIP: بازطراحی پرداخت"
# ... کار ...
gh pr ready 42
gh pr edit 42 --add-reviewer maryam-ahmadi
```

#### مثال ۳: fix کردن check شکست‌خورده

```bash
gh pr checks 42
# X  test (ubuntu-latest)   failing  2m
# ✓  lint                   passing  45s

gh run list --branch feat/profile --limit 3
gh run view <run-id> --log-failed
```

`--log-failed` فقط بخش شکست‌خورده را نشان می‌دهد — بسیار سریع‌تر از خواندن کل لاگ.

```bash
# بعد از fix
git commit -am "fix: رفع تست شکسته"
git push
```

#### مثال ۴: حل conflict و sync

```bash
git fetch origin
git switch feat/profile
git rebase origin/main
# حل conflict
git add .
git rebase --continue
git push --force-with-lease
```

اگر در طول review هستی، به‌جای rebase از merge استفاده کن (فصل ۱۹).

#### مثال ۵: درخواست review مجدد

```bash
gh pr edit 42 --add-reviewer maryam-ahmadi
# یا در UI: دکمه‌ی "Re-request review" کنار نام reviewer
```

#### مثال ۶: Auto-merge

اگر همه‌ی شرایط برآورده شد، خودکار merge شود:

```bash
gh pr merge 42 --squash --auto
```

**چرا مفید است؟** PR را می‌سازی، auto-merge را روشن می‌کنی، می‌روی. وقتی CI سبز شد و review آمد، خودش merge می‌شود.

**نیاز:** مخزن باید auto-merge را اجازه دهد (**Settings → General → Allow auto-merge**).

#### مثال ۷: بستن PR بدون merge

```bash
gh pr close 42 --comment "به‌جای این، در #57 حل شد"
gh pr reopen 42   # اگر پشیمان شدی
```

#### مثال ۸: PR از fork — تأیید workflow

اگر PR از fork باشد، GitHub ممکن است منتظر تأیید maintainer برای اجرای workflow بماند:

```
This workflow requires approval from a maintainer.
```

در UI: **Approve and run workflows**.

### مقایسه و الگوهای مختلف

#### Auto-merge در برابر merge دستی

| | Auto-merge | دستی |
|---|---|---|
| نیاز به حضور | ❌ | ✅ |
| سرعت | به‌محض آماده شدن | وقتی یادت بیاید |
| کنترل | کم | کامل |
| مناسب | PRهای روتین | PRهای حساس |

#### `Require branches to be up to date` — روشن یا خاموش؟

| | روشن | خاموش |
|---|---|---|
| اطمینان | CI روی کد نهایی اجرا می‌شود | CI ممکن است قدیمی باشد |
| نیاز به sync | هر بار base عوض شد | فقط وقتی conflict |
| سرعت تیم | کندتر | سریع‌تر |
| مناسب | تیم کوچک، deploy سریع | تیم بزرگ، PR زیاد |

**راه میانه:** روشن کن + **Merge Queue** فعال کن (فصل ۳۶). Merge queue این مشکل را حل می‌کند.

#### `mergeStateStatus` — جدول تصمیم

| مقدار | اقدام |
|---|---|
| `CLEAN` | merge کن |
| `BEHIND` | `git pull --rebase` و push |
| `DIRTY` | conflict را حل کن |
| `BLOCKED` | دلیل را در UI بخوان |
| `UNSTABLE` | منتظر checks یا fix |

### اشکال شایع

#### ۱. «Merge button خاکستری است»

باکس «Merging is blocked» را بخوان — دلیل دقیقاً نوشته شده. ۹۰٪: review یا check.

#### ۲. «This branch has conflicts that must be resolved»

```bash
git fetch origin && git rebase origin/main
```

یا در UI دکمه‌ی **Resolve conflicts** (ادیتور ساده‌ی GitHub).

#### ۳. «Required status check "X" is expected»

اسم check در Ruleset با اسم job در workflow نمی‌خواند. باید دقیقاً یکی باشند:

```yaml
jobs:
  test:            # ← این اسم باید در Ruleset باشد
    runs-on: ubuntu-latest
```

اگر matrix استفاده می‌کنی، اسم check شامل matrix می‌شود: `test (18)`، `test (20)`.

#### ۴. «At least 1 approving review is required»

reviewer پیدا کن. اگر تنها کار می‌کنی، Ruleset را برای خودت bypass کن (فصل ۳۹) — ولی بهتر: یک همکار پیدا کن.

#### ۵. PR بعد از force push، review را از دست داد

GitHub با تنظیم **Dismiss stale pull request approvals when new commits are pushed** تأییدها را باطل می‌کند. این امنیتی است، نه باگ.

#### ۶. «Pull request is not mergeable» بدون دلیل

صفحه را رفرش کن. گاهی GitHub در حال محاسبه است.

#### ۷. Merge کردم ولی Issue بسته نشد

`Closes #42` باید در **توضیح** PR باشد (نه در کامنت). و Issue باید در همان مخزن باشد (یا با `owner/repo#42`).

### نکات مهم (Tips & Tricks)

- **`gh pr status`** وضعیت PRهای خودت را خلاصه می‌دهد — بهترین دستور صبحگاهی.
- **`gh pr checks --watch`** تا تمام شدن checks صبر می‌کند.
- **`gh run view --log-failed`** فقط شکست را نشان می‌دهد.
- **`gh pr merge --auto --squash`** را عادت کن — از انتظار بی‌فایده جلوگیری می‌کند.
- **قبل از درخواست review، `git diff origin/main...HEAD`** را کامل بخوان.
- **`gh pr edit --add-label`** برای دسته‌بندی PRها.

### خلاصه فصل

- PR ده وضعیت دارد؛ هر وضعیت یک اقدام مشخص.
- `mergeStateStatus` مهم‌ترین فیلد برای تشخیص مشکل است.
- **Auto-merge** انتظار را حذف می‌کند.
- اسم required check باید دقیقاً با اسم job بخواند.
- `Closes #N` در توضیح، Issue را می‌بندد.

**سؤالات مروری:**
1. تفاوت `BLOCKED` و `BEHIND` در `mergeStateStatus` چیست؟
2. چرا اسم check با اسم job باید یکی باشد؟
3. چطور بدون حضور، PR را merge کنی؟

---

## فصل ۳۴: Code Review مؤثر — چه چیزی را ببینیم

### مقدمه

Code Review یکی از پرهزینه‌ترین فعالیت‌های تیم است: دو نفر، نیم ساعت، برای هر PR. اگر مؤثر نباشد، هم وقت تلف می‌شود هم کیفیت بالا نمی‌رود.

**چرا مهم است؟** review خوب باگ می‌گیرد، دانش را پخش می‌کند، و استانداردها را زنده نگه می‌دارد. review بد (نیت‌پاکانه ولی بی‌فایده) فقط تأخیر ایجاد می‌کند.

**کاربرد عملی:** این فصل دو دیدگاه می‌دهد: چطور review کنم و چطور review بگیرم.

### مفاهیم اساسی

#### چه چیزی را در review ببینیم — به ترتیب اولویت

| اولویت | چه چیزی | چرا |
|---|---|---|
| ۱ | **درستی** | آیا کد کاری که می‌گوید را می‌کند؟ |
| ۲ | **امنیت** | ورودی اعتبارسنجی شده؟ secret لو نمی‌رود؟ |
| ۳ | **مرزهای داده** | حالت‌های خالی، null، خطا |
| ۴ | **تست** | آیا رفتار جدید تست دارد؟ |
| ۵ | **خوانایی** | آیا ۶ ماه بعد قابل فهم است؟ |
| ۶ | **کارایی** | آیا N+1 query، loop سنگین، یا حافظه‌ی زیاد دارد؟ |
| ۷ | **سبک** | نام‌گذاری، فرمت — که باید خودکار باشد |

**نکته‌ی مهم:** سبک (۷) نباید در review انسانی بحث شود. linter و formatter باید خودکار اجرا شوند. اگر درباره‌ی فاصله بحث می‌کنی، ابزارت کم است.

#### چه چیزی را **نبینیم**

- سبک کد (کار linter)
- چیزهایی که کامپایلر/type checker می‌گیرد
- سلیقه‌ی شخصی بدون دلیل
- بازنویسی کامل به سبک خودت («من این‌طور می‌نوشتم»)

#### سه نوع نظر

| نوع | مثال | کِی |
|---|---|---|
| **Blocking** | «این کوئری SQL injection دارد» | باید fix شود |
| **Suggestion** | «می‌شود از `map` استفاده کرد» | اختیاری |
| **Question** | «چرا اینجا `setTimeout` گذاشتی؟» | برای فهمیدن |

**مهم:** نوع را صریح مشخص کن. «این باید عوض شود» با «این را می‌شود بهتر کرد» تفاوت دارد.

#### قاعده‌ی «چه چیزی را نگه داریم»

- اگر PR ۸۰٪ کار درست است، approve کن با چند suggestion — نه request changes برای یک نکته‌ی کوچک.
- اگر یک مسئله‌ی امنیتی یا درستی دارد، request changes.
- اگر سبک است و linter نگرفته، comment بگذار ولی approve کن.

### مثال‌های عملی

#### مثال ۱: review یک تابع — چک‌لیست ذهنی

```javascript
// PR اضافه کرده:
export async function getUser(req, res) {
  const user = await db.query(`SELECT * FROM users WHERE id = ${req.params.id}`);
  res.json(user);
}
```

**مسائل:**

1. **SQL Injection** (blocking): `${req.params.id}` مستقیم در کوئری.
2. **بدون خطا** (blocking): اگر کاربر نبود؟
3. **`SELECT *`** (suggestion): ستون‌های حساس (password_hash) هم برمی‌گردند.
4. **بدون احراز هویت** (blocking): هر کسی می‌تواند هر کاربری را ببیند؟ (IDOR)
5. **بدون تست** (blocking اگر سیاست تیم است).

**کامنت خوب:**

```markdown
**Blocking — امنیت:** `req.params.id` مستقیم در کوئری می‌رود → SQL injection.
از پارامتر استفاده کن:

```suggestion
  const user = await db.query("SELECT id, email, name FROM users WHERE id = $1", [req.params.id]);
```

**Blocking:** اگر کاربر پیدا نشد، `user` خالی است و `res.json(undefined)` خطا می‌دهد.
**Question:** آیا این endpoint احراز هویت دارد؟ الان هر کسی می‌تواند هر id را بخواند.
```

#### مثال ۲: کامنت با suggested change

در UI، روی خط کلیک کن → آیکون `±` → پیشنهاد بنویس:

````markdown
```suggestion
const normalized = email.trim().toLowerCase();
```
````

نویسنده‌ی PR می‌تواند با یک کلیک **Commit suggestion** بزند.

#### مثال ۳: batch review

در GitHub می‌توانی چند کامنت بگذاری و در پایان یک review واحد submit کنی:

1. **Files changed** → کامنت بگذار (Add single comment / Start a review)
2. کامنت‌ها در حالت **Pending** جمع می‌شوند
3. **Review changes** → Comment / Approve / Request changes → **Submit review**

**مزیت:** نویسنده یک ایمیل می‌گیرد، نه ده تا. و می‌تواند همه را با هم ببیند.

#### مثال ۴: review با gh

```bash
gh pr diff 42                    # diff کامل
gh pr diff 42 --name-only        # فقط فایل‌ها

# approve
gh pr review 42 --approve --body "LGTM، فقط یک نکته‌ی کوچک در auth.js"

# request changes
gh pr review 42 --request-changes --body "کوئری SQL باید پارامتری شود"

# comment
gh pr review 42 --comment --body "سؤال: چرا از cache استفاده نکردی؟"

# checkout برای تست محلی
gh pr checkout 42
npm test
```

#### مثال ۵: خواندن diff هوشمندانه

```bash
# فقط فایل‌های منطقی، بدون lock file
git diff origin/main...HEAD -- . ':!package-lock.json' ':!*.snap'

# فقط تغییرات یک پوشه
git diff origin/main...HEAD -- src/auth/

# با آمار
git diff --stat origin/main...HEAD
```

**تکنیک:** اول `--stat` را ببین. اگر ۴۰ فایل است، دنبال فایل‌های اصلی بگرد و بقیه را سریع رد کن.

#### مثال ۶: تست PR دیگران به‌صورت محلی

```bash
gh pr checkout 42
npm install
npm test
npm run dev
# دستی تست کن
```

برای PRهای UI، این تنها راه واقعی ارزیابی است.

#### مثال ۷: نوشتن PR که راحت review شود

```markdown
## چه چیزی عوض شد
افزودن صفحه‌ی پروفایل با نمایش و ویرایش نام.

## چطور review کنی (راهنما)
۱. `src/pages/profile.tsx` — ساختار صفحه (اصلی)
۲. `src/api/profile.ts` — endpoint (کوچک)
۳. `src/lib/validate.ts` — اعتبارسنجی (کپی از `auth.ts` با تغییر کوچک)
۴. بقیه: تست‌ها و snapshot

## چه چیزی را نادیده بگیر
- `*.snap` — snapshotهای خودکار
- `package-lock.json` — فقط نصب zod

## سؤالات باز
آیا باید آواتار را در همین PR بگذاریم یا جدا؟
```

این «راهنمای review» نرخ و کیفیت review را چند برابر می‌کند.

### مقایسه و الگوهای مختلف

#### Review سبک در برابر عمیق

| | سبک (۵ دقیقه) | عمیق (۳۰ دقیقه) |
|---|---|---|
| مناسب | PR کوچک، تغییر مستندات | تغییر منطق حساس |
| چه می‌بیند | ساختار کلی، نام‌ها | درستی، امنیت، edge case |
| خطر | باگ‌های ظریف | کندی تیم |

**توصیه:** PR کوچک → سبک. PR حساس → عمیق. **همه را یکسان review نکن.**

#### Approve در برابر Request changes

| | Approve | Request changes |
|---|---|---|
| چه زمانی | قابل merge است، نکات جزئی | مسئله‌ی درستی/امنیت |
| اثر | merge ممکن | merge مسدود |
| خطر | مسئله‌ی جدی رد شود | تأخیر برای چیز جزئی |

**قاعده:** Request changes را برای چیزهایی نگه دار که **واقعاً** باید عوض شوند.

#### کامنت در برابر suggested change

| | کامنت | Suggestion |
|---|---|---|
| سرعت اعمال | نویسنده باید تایپ کند | یک کلیک |
| مناسب | توضیح، سؤال | تغییر کوچک مشخص |
| محدودیت | — | فقط خطوط موجود |

### اشکال شایع

#### ۱. «LGTM» بدون خواندن

اگر واقعاً نخوانده‌ای، approve نکن. بگو «وقت ندارم، فردا».

#### ۲. Bike-shedding

بحث ۲۰ کامنتی درباره‌ی نام یک متغیر، و نادیده گرفتن یک باگ امنیتی. اولویت‌ها را یادت باشد.

#### ۳. Request changes برای سبک

اگر linter نمی‌گیرد، پس مهم نیست. comment بگذار، approve کن.

#### ۴. review شخصی

```
✗ "این کد اصلاً خوب نیست"
✓ "این تابع در حالت X خطا می‌دهد، چون ..."
```

به کد نقد کن، نه به نویسنده.

#### ۵. PR بی‌پاسخ

اگر reviewer ۳ روز جواب نداد، در PR کامنت بزن یا در Slack پیام بده. سکوت = گیر کردن.

#### ۶. approve کردن PR خودت

GitHub اجازه می‌دهد (در بعضی تنظیمات)، ولی بی‌معنی است. Ruleset می‌تواند جلوگیری کند.

#### ۷. review بدون تست

برای تغییر منطقی، PR را checkout کن و تست کن. خواندن diff کافی نیست.

### نکات مهم (Tips & Tricks)

- **اول توضیح PR را بخوان، بعد diff.** بدون فهم «چرا»، review کردن «چه» بی‌فایده است.
- **`gh pr checkout`** + تست محلی، بهترین review.
- **کامنت را با برچسب نوع شروع کن:** `[blocking]`، `[suggestion]`، `[question]`، `[nit]`.
- **در review، چیزی که خوب است را هم بگو.** فقط انتقاد، انگیزه را می‌کشد.
- **PR را در ۲۴ ساعت review کن.** بیشتر از این، هزینه‌ی context switch برای نویسنده زیاد می‌شود.
- **`Require review from Code Owners`** را در Ruleset روشن کن تا متخصص هر بخش review کند.

### خلاصه فصل

- اولویت review: درستی → امنیت → مرزهای داده → تست → خوانایی → کارایی → سبک.
- **سبک را ابزار باید بگیرد، نه انسان.**
- سه نوع نظر: blocking، suggestion، question — نوع را صریح کن.
- **Batch review** بهتر از ده کامنت جدا.
- Request changes را برای مسائل واقعی نگه دار.
- `gh pr checkout` + تست محلی = بهترین review.

**سؤالات مروری:**
1. چرا بحث درباره‌ی سبک در review نشانه‌ی ابزار ناکافی است؟
2. تفاوت `[blocking]` و `[suggestion]` در کامنت چیست؟
3. چطور PR را طوری بنویسی که review سریع‌تر شود؟

---

## فصل ۳۵: ابزارهای review — suggestion، batch، resolve

### مقدمه

GitHub در سال‌های اخیر ابزارهای review را بسیار بهتر کرده: suggested changes، multi-line comments، file filtering، resolve conversation، و Copilot code review. این فصل ابزارها را عملی نشان می‌دهد.

**چرا مهم است؟** ابزار درست، تفاوت بین «۳۰ کامنت پراکنده» و «یک review منظم» است. و برای نویسنده، تفاوت بین «۲۰ دقیقه تایپ» و «۳ کلیک».

**کاربرد عملی:** اینها را یک بار یاد بگیری، در هر PR استفاده می‌کنی.

### مفاهیم اساسی

#### قابلیت‌های کلیدی

| قابلیت | کار |
|---|---|
| **Suggested change** | پیشنهاد کد قابل اعمال با یک کلیک |
| **Multi-line comment** | کامنت روی چند خط |
| **File filter** | دیدن فقط فایل‌های خاص |
| **Viewed** | علامت‌گذاری فایل دیده‌شده |
| **Resolve conversation** | بستن thread حل‌شده |
| **Pending review** | جمع کردن کامنت‌ها و submit یکجا |
| **Copilot review** | review خودکار با AI |

#### Suggested change — سینتکس

در بلوک ` ```suggestion ` محتوا را بنویس. GitHub آن را به‌جای خط(های) انتخاب‌شده پیشنهاد می‌دهد.

**محدودیت:** فقط روی خطوطی که در diff هستند (added یا context) کار می‌کند.

### مثال‌های عملی

#### مثال ۱: suggested change ساده

diff:

```diff
-const email = req.body.email;
+const email = req.body.email.trim();
```

کامنت:

````markdown
برای اطمینان بیشتر، lowercase هم کن:

```suggestion
const email = req.body.email.trim().toLowerCase();
```
````

نویسنده: **Commit suggestion** → یک commit جدید در PR.

#### مثال ۲: suggested change چندخطی

روی چند خط انتخاب کن (کلیک + shift کلیک) و:

````markdown
```suggestion
export async function getUser(id) {
  const { rows } = await db.query(
    "SELECT id, email, name FROM users WHERE id = $1",
    [id]
  );
  return rows[0] ?? null;
}
```
````

#### مثال ۳: کامنت روی خطی که در diff نیست

GitHub اجازه نمی‌دهد روی خط خارج از diff کامنت بگذاری. راه‌حل: کامنت روی نزدیک‌ترین خط، یا در توضیح کلی PR.

#### مثال ۴: resolve conversation

بعد از حل، دکمه‌ی **Resolve conversation** روی thread. این:

- thread را جمع می‌کند
- اگر Ruleset «Require conversation resolution» روشن باشد، merge را ممکن می‌کند

**قاعده:** نویسنده resolve کند، نه reviewer (مگر reviewer مطمئن باشد حل شده).

#### مثال ۵: Viewed — علامت‌گذاری فایل

در **Files changed**، هر فایل یک چک‌باکس **Viewed** دارد. برای PRهای بزرگ حیاتی است: فایل‌های دیده‌شده جمع می‌شوند و می‌توانی فردا ادامه بدهی.

#### مثال ۶: فیلتر فایل‌ها

```
Files changed → Filter files... → src/auth/
```

یا:

```
*.ts !*test*
```

#### مثال ۷: Copilot Code Review

اگر Copilot داری:

1. در PR، کنار **Reviewers** → **Copilot**
2. Copilot یک review با کامنت‌های پیشنهادی می‌دهد
3. می‌توانی هر کامنت را dismiss یا اعمال کنی

**نکته:** Copilot review جایگزین review انسانی نیست — مکمل است. برای چیزهای مکانیکی (خطاهای رایج، الگوهای ناامن) خوب است.

می‌توانی خودکار کنی:

```yaml
# .github/workflows/copilot-review.yml
name: Copilot Review
on:
  pull_request:
    types: [opened, synchronize]
jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            await github.rest.pulls.requestReviewers({
              ...context.repo,
              pull_number: context.payload.pull_request.number,
              reviewers: ["copilot-pull-request-reviewer[bot]"],
            });
```

**هشدار:** قابلیت Copilot review سریع تغییر می‌کند؛ نام bot و روش فعال‌سازی را در مستندات همان روز چک کن.

#### مثال ۸: پیشنهاد چندتایی در gh

`gh pr review` کامنت خطی نمی‌گذارد. برای کامنت خطی از API استفاده کن:

```bash
gh api -X POST repos/OWNER/REPO/pulls/42/reviews \
  -f event=REQUEST_CHANGES \
  -f body="چند نکته" \
  -f 'comments[][path]=src/auth.js' \
  -f 'comments[][line]=12' \
  -f 'comments[][body]=این باید پارامتری شود'
```

#### مثال ۹: پیشنهاد در Git — `--fixup`

اگر reviewer خودت هستی و می‌خواهی تغییر کوچک بدهی:

```bash
gh pr checkout 42
# تغییر بده
git commit --fixup=<commit-hash>
git push
```

### مقایسه و الگوهای مختلف

#### کامنت در برابر suggestion

| | کامنت | Suggestion |
|---|---|---|
| نیاز به تایپ نویسنده | ✅ | ❌ |
| مناسب تغییر چندخطی | ✅ | ✅ |
| مناسب توضیح | ✅ | ❌ |
| اعمال گروهی | ❌ | ✅ (Add suggestion to batch) |

#### Review انسانی در برابر Copilot

| | انسان | Copilot |
|---|---|---|
| درک زمینه‌ی کسب‌وکار | ✅ | محدود |
| خطاهای مکانیکی | متوسط | ✅ |
| امنیت | خوب | خوب |
| سرعت | کند | سریع |
| هزینه | وقت تیم | اشتراک |

**بهترین ترکیب:** Copilot اول (مکانیکی) → انسان بعد (منطق و زمینه).

#### Thread resolution

| | Resolve | Reply |
|---|---|---|
| اثر | thread جمع می‌شود | ادامه‌ی بحث |
| کِی | حل شد | هنوز باز است |
| اگر Ruleset دارد | لازم برای merge | کافی نیست |

### اشکال شایع

#### ۱. Suggestion روی خط اشتباه

اگر خطی که انتخاب می‌کنی در diff نباشد، GitHub اجازه نمی‌دهد. اگر خطوط را جابه‌جا کنی، suggestion ممکن است context را از دست بدهد.

#### ۲. Commit suggestion همه را یکجا

دکمه‌ی **Add suggestion to batch** را بزن تا چند suggestion را در یک commit اعمال کنی. وگرنه ۱۰ commit «Apply suggestion» داری.

#### ۳. Resolve کردن thread بدون حل

اگر Ruleset «Require conversation resolution» دارد، کسی می‌تواند بدون حل کردن resolve کند و merge کند. **قاعده‌ی تیمی:** نویسنده resolve کند، با کامنت تأیید.

#### ۴. Copilot review را جدی گرفتن

Copilot می‌تواند چیزهای بی‌ربط پیشنهاد دهد. هر کامنت را خودت ارزیابی کن.

#### ۵. نادیده گرفتن `Viewed`

در PR بزرگ، `Viewed` تنها راه پیگیری است. استفاده کن.

#### ۶. کامنت روی فایل حذف‌شده

اگر فایل حذف شده، GitHub اجازه‌ی کامنت نمی‌دهد. در توضیح کلی بنویس.

### نکات مهم (Tips & Tricks)

- **`Add suggestion to batch`** را یاد بگیر — یک commit به‌جای ده.
- **`Viewed` + فیلتر** برای PR بزرگ، تفاوت بین ۱ ساعت و ۱۰ دقیقه.
- **`Resolve conversation`** را به عادت تبدیل کن — threadهای باز، PR را شلوغ می‌کنند.
- **در Ruleset، `Require conversation resolution`** را روشن کن تا threadهای حل‌نشده مانع merge شوند.
- **Copilot review** را برای PRهای روتین خودکار کن، برای PRهای حساس خودت هم بخوان.
- **کامنت `[nit]`** برای چیزهای جزئی که نباید merge را مسدود کنند.

### خلاصه فصل

- **Suggested change** با ` ```suggestion ` و یک کلیک اعمال می‌شود.
- **Batch** کامنت‌ها و suggestionها را یکجا submit می‌کند.
- **Viewed** و فیلتر، PR بزرگ را قابل مدیریت می‌کند.
- **Resolve conversation** + Ruleset = threadهای باز مانع merge.
- **Copilot review** مکمل است، نه جایگزین.

**سؤالات مروری:**
1. چطور چند suggestion را در یک commit اعمال کنی؟
2. چرا resolve کردن thread توسط نویسنده بهتر است؟
3. Copilot review کجا خوب است و کجا نه؟

---

## فصل ۳۶: Merge Queue و PRهای پشت‌سرهم

### مقدمه

دو مشکل واقعی در تیم‌های پرکار:

1. **«Require branches to be up to date»** باعث می‌شود هر PR قبل از merge نیاز به sync داشته باشد — و با ۱۰ PR باز، این یک مسابقه‌ی بی‌پایان است.
2. **PRهای زنجیره‌ای** (stacked PRs): ویژگی بزرگ به ۳ PR تقسیم می‌شود، هر کدام به قبلی وابسته.

**چرا مهم است؟** Merge Queue این مسابقه را حل می‌کند و stacked PRها به تیم‌های بالغ اجازه می‌دهند PR کوچک بفرستند بدون اینکه منتظر merge بمانند.

**کاربرد عملی:** اگر روزی ۵+ PR merge می‌کنی، Merge Queue ضروری است.

### مفاهیم اساسی

#### مشکل: مسابقه‌ی merge

```
main:        A
PR#1 (from A):  → approved ✓, CI ✓
PR#2 (from A):  → approved ✓, CI ✓

PR#1 merge → main: A ← B
PR#2: حالا base عوض شده. اگر "Require up to date" باشد،
      باید rebase کند و CI دوباره اجرا شود.
```

با ۱۰ PR هم‌زمان، هر merge بقیه را باطل می‌کند. تیم در حلقه‌ی بی‌پایان rebase گیر می‌کند.

#### Merge Queue چطور حل می‌کند

```
PRها در صف قرار می‌گیرند:
  [PR#3] [PR#7] [PR#9]

Queue یک branch موقت می‌سازد:
  main + PR#3  → CI اجرا → سبز → merge به main
  main' + PR#7 → CI اجرا → سبز → merge
  main'' + PR#9 → CI اجرا → سبز → merge
```

هر PR روی **نتیجه‌ی PRهای قبلی** تست می‌شود، نه روی main قدیمی. اگر PR#7 با PR#3 تعارض داشت، در صف معلوم می‌شود، نه در main.

#### پیش‌نیازها

| نیاز | چرا |
|---|---|
| Ruleset روی `main` | Merge Queue بخشی از Ruleset است |
| Required status checks | Queue باید بداند چه چیزی را چک کند |
| GitHub Team یا بالاتر | Merge Queue در پلن رایگان برای مخزن عمومی هست، خصوصی نیاز به Team |
| `merge_group` trigger در workflow | وگرنه checks در صف اجرا نمی‌شوند |

#### `merge_group` — مهم‌ترین نکته

workflowهای CI باید علاوه بر `pull_request`، روی `merge_group` هم trigger شوند:

```yaml
on:
  pull_request:
  merge_group:
```

بدون این، checks در صف اجرا نمی‌شوند و PRها گیر می‌کنند.

### مثال‌های عملی

#### مثال ۱: فعال‌سازی Merge Queue

**Settings → Rules → Rulesets** (یا Branch protection):

```
☑ Require merge queue
  Merge method: Squash
  Build concurrency: 5
  Minimum group size: 1
  Maximum group size: 5
  Wait time: 5 minutes
```

یا با API:

```bash
gh api -X PUT repos/OWNER/REPO/branches/main/protection \
  -f 'required_status_checks[strict]=true' \
  -f 'required_status_checks[contexts][]=test' \
  -F 'required_linear_history=false' \
  -F 'enforce_admins=true' \
  -F 'required_pull_request_reviews[required_approving_review_count]=1' \
  -F 'restrictions=null'
```

**توجه:** API برای Merge Queue محدود است؛ از UI یا Ruleset استفاده کن.

#### مثال ۲: workflow سازگار با Queue

`.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
  merge_group:      # ← حیاتی
  push:
    branches: [main]

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm test
```

#### مثال ۳: دیدن صف

```bash
gh api repos/OWNER/REPO/merge-queue --jq '.[] | "\(.position). \(.pull_request.number) \(.state)"'
```

یا در UI: تب **Merge queue** در صفحه‌ی مخزن.

#### مثال ۴: PRهای زنجیره‌ای (stacked)

```
main
 └── feat/part-1   (PR #10 → main)
      └── feat/part-2   (PR #11 → feat/part-1)
           └── feat/part-3   (PR #12 → feat/part-2)
```

```bash
git switch main
git switch -c feat/part-1
# ... commit ...
git push -u origin feat/part-1
gh pr create --base main --title "بخش ۱: مدل داده"

git switch -c feat/part-2      # از feat/part-1
# ... commit ...
git push -u origin feat/part-2
gh pr create --base feat/part-1 --title "بخش ۲: API"

git switch -c feat/part-3
# ...
gh pr create --base feat/part-2 --title "بخش ۳: UI"
```

بعد از merge شدن #10، GitHub خودکار base ی #11 را به `main` تغییر می‌دهد.

#### مثال ۵: به‌روزرسانی زنجیره بعد از تغییر در پایه

```bash
# اگر part-1 عوض شد
git switch feat/part-1
git commit -am "fix: اصلاح مدل"
git push

# حالا part-2 و part-3 را rebase کن
git switch feat/part-2
git rebase feat/part-1
git push --force-with-lease

git switch feat/part-3
git rebase feat/part-2
git push --force-with-lease
```

#### مثال ۶: ابزارهای stacked PR

| ابزار | کار |
|---|---|
| **`gh stack`** (extension) | مدیریت زنجیره |
| **Graphite** | ابزار اختصاصی stacked PR |
| **git-branchless** | تاریخ و زنجیره |
| **spr** (Stacked PRs) | ابزار متن‌باز |

```bash
# نصب gh extension
gh extension install github/gh-stack

# استفاده
gh stack submit
gh stack sync
```

**هشدار:** ابزارهای stacked PR سریع تغییر می‌کنند؛ قبل از اتکا، مستندات را چک کن.

### مقایسه و الگوهای مختلف

#### Merge Queue در برابر Require up-to-date

| | Merge Queue | Require up-to-date |
|---|---|---|
| نیاز به rebase دستی | ❌ | ✅ |
| CI روی کد نهایی | ✅ | ✅ |
| سرعت تیم | بالا | پایین |
| پیچیدگی setup | متوسط | کم |
| مناسب | تیم فعال | تیم کوچک |

#### Stacked در برابر یک PR بزرگ

| | Stacked (۳ PR) | یک PR بزرگ |
|---|---|---|
| review | ✅ سریع | ❌ کند |
| ریسک | کم | زیاد |
| پیچیدگی مدیریت | بالا | کم |
| نیاز به ابزار | ✅ | ❌ |
| مناسب | ویژگی بزرگ | تغییر کوچک |

#### Merge method با Queue

| روش | با Queue |
|---|---|
| Squash | ✅ توصیه‌شده |
| Merge commit | ✅ ممکن |
| Rebase | ✅ ممکن |

Squash با Queue ساده‌ترین است چون یک commit در main می‌گذارد.

### اشکال شایع

#### ۱. فراموش کردن `merge_group` در workflow

PR وارد صف می‌شود، checks اجرا نمی‌شوند، PR برای همیشه pending می‌ماند.

```
✗ on: pull_request
✓ on: [pull_request, merge_group]
```

#### ۲. `concurrency` که صف را می‌شکند

```yaml
✗ concurrency:
    group: ci-${{ github.ref }}
    cancel-in-progress: true
```

در `merge_group`، `github.ref` برای همه‌ی آیتم‌های گروه یکی است → هر گروه گروه قبلی را cancel می‌کند. راه‌حل:

```yaml
concurrency:
  group: ci-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}
```

#### ۳. Queue با test کند

اگر CI ۲۰ دقیقه است و ۵ PR در صف، آخرین PR یک ساعت منتظر می‌ماند. CI را سریع کن (فصل ۶۱).

#### ۴. Stacked PR بدون ابزار

با ۳ PR دستی می‌شود؛ با ۶ تا، rebase دستی فاجعه است. ابزار استفاده کن.

#### ۵. فراموش کردن به‌روزرسانی زنجیره

اگر پایه عوض شد و بقیه را rebase نکنی، PRهای بعدی conflict می‌دهند.

#### ۶. PR زنجیره‌ای که در ترتیب اشتباه merge می‌شود

اگر #12 قبل از #11 merge شود، #12 شامل تغییرات #11 نیست (چون base اش #11 است ولی merge به main می‌رود). GitHub معمولاً جلوی این را می‌گیرد (چون base merge نشده)، ولی حواست باشد.

### نکات مهم (Tips & Tricks)

- **`merge_group` را به همه‌ی workflowهای required اضافه کن.**
- **`concurrency` را برای `merge_group` اصلاح کن** — تله‌ی رایج.
- **Build concurrency** را بالا ببر (۵-۱۰) اگر CI سریع است.
- **PRهای زنجیره‌ای را کوچک نگه دار** — هر کدام حداکثر ۳۰۰ خط.
- **`gh stack` یا Graphite** را امتحان کن اگر زیاد stacked کار می‌کنی.
- **بعد از merge، branchهای زنجیره را پاک کن** — وگرنه انبار می‌شوند.

### خلاصه فصل

- **Merge Queue** مسابقه‌ی merge را حل می‌کند: هر PR روی نتیجه‌ی قبلی تست می‌شود.
- نیاز به `merge_group` trigger در workflow دارد.
- **Stacked PRs** ویژگی بزرگ را به PRهای کوچک می‌شکند.
- هر PR در زنجیره، base اش PR قبلی است.
- بعد از merge پایه، بقیه را rebase کن.
- `concurrency` باید با `merge_group` سازگار باشد.

**سؤالات مروری:**
1. Merge Queue چه مشکلی را حل می‌کند؟
2. چرا `merge_group` در workflow لازم است؟
3. چطور یک زنجیره‌ی سه‌تایی PR بسازی؟

---

## فصل ۳۷: PRهای بزرگ — چرا بد هستند و چطور بشکنیم

### مقدمه

همه می‌دانیم PR بزرگ بد است. باز هم می‌سازیم، چون شکستن سخت است. این فصل راه‌های عملی برای شکستن می‌دهد.

**چرا مهم است؟** اعداد واقعی: reviewer بعد از ~۴۰۰ خط، توان تشخیص باگ به‌شدت افت می‌کند. PR ۲۰۰۰ خطی عملاً review نمی‌شود — فقط approve می‌شود.

**کاربرد عملی:** سه تکنیک برای شکستن PR، و یک تکنیک برای وقتی که مجبوری بزرگ بفرستی.

### مفاهیم اساسی

#### چرا PR بزرگ بد است

| مشکل | توضیح |
|---|---|
| **review سطحی** | بعد از ۴۰۰ خط، تمرکز می‌رود |
| **conflict** | هر روز که می‌گذرد، احتمال conflict بیشتر |
| **revert سخت** | revert یک PR بزرگ، چیزهای خوب را هم می‌برد |
| **bisect درشت** | یک commit غول‌پیکر، bisect را بی‌فایده می‌کند |
| **context switch** | نویسنده منتظر می‌ماند، کار دیگر را نمی‌تواند شروع کند |
| **بحث طولانی** | ۵۰ کامنت، ۳ جلسه، هفته‌ها تأخیر |

#### چه چیزی «بزرگ» است

| اندازه | خط تغییر | وضعیت |
|---|---|---|
| XS | < ۵۰ | ایده‌آل |
| S | ۵۰–۲۰۰ | خوب |
| M | ۲۰۰–۴۰۰ | قابل قبول |
| L | ۴۰۰–۸۰۰ | باید توجیه شود |
| XL | > ۸۰۰ | بشکن |

**نکته:** تعداد فایل هم مهم است. ۵۰۰ خط در ۳ فایل، بهتر از ۵۰۰ خط در ۴۰ فایل است.

#### چرا PR بزرگ ساخته می‌شود

1. **کار بزرگ است** — «باید کل ماژول را عوض کنم»
2. **ترس از نیمه‌کاره** — «اگر merge شود، همه‌چیز می‌شکند»
3. **عجله** — «وقت ندارم سه PR بسازم»
4. **نبود feature flag** — «نمی‌توانم کد ناتمام را merge کنم»

### مثال‌های عملی

#### تکنیک ۱: Refactor جدا از Feature

```
✗ یک PR: تغییر نام ۲۰ تابع + افزودن ویژگی جدید

✓ PR #1: refactor — تغییر نام (خالص، بدون تغییر رفتار)
✓ PR #2: feat — افزودن ویژگی (روی کد تمیز)
```

PR #1 با تست‌های موجود تأیید می‌شود که رفتار عوض نشده. PR #2 کوچک و واضح است.

#### تکنیک ۲: Feature Flag

```javascript
// flags.js
export const flags = {
  newCheckout: process.env.FF_NEW_CHECKOUT === "true",
};

// CheckoutPage.jsx
export default function CheckoutPage() {
  return flags.newCheckout ? <NewCheckout /> : <LegacyCheckout />;
}
```

حالا می‌توانی:
- PR #1: ساختار و flag (بدون UI جدید) — merge بی‌خطر
- PR #2: کامپوننت جدید — merge، flag خاموش
- PR #3: اتصال به API — merge، flag خاموش
- PR #4: تست e2e — merge
- PR #5: روشن کردن flag (یک خط!)

هر PR کوچک و قابل review.

#### تکنیک ۳: لایه‌ای کردن (Vertical Slicing)

```
✗ یک PR: "صفحه‌ی گزارش" با ۱۵۰۰ خط

✓ PR #1: API endpoint + تست (۳۰۰ خط)
✓ PR #2: کامپوننت جدول + story (۲۵۰ خط)
✓ PR #3: فیلترها (۲۰۰ خط)
✓ PR #4: خروجی CSV (۱۵۰ خط)
✓ PR #5: اتصال به صفحه (۱۰۰ خط)
```

هر کدام مستقل merge می‌شود (اگر ترتیب درست باشد).

#### تکنیک ۴: وقتی مجبوری بزرگ بفرستی

گاهی واقعاً نمی‌شود شکست (مثلاً migration دیتابیس). راه‌حل:

**الف) راهنمای review بنویس:**

```markdown
## راهنمای review — این PR ۱٬۸۰۰ خط است

### نقشه
| فایل | خطوط | اهمیت | توضیح |
|---|---|---|---|
| `db/migrations/001.sql` | ۱۲۰ | 🔴 بالا | تغییر schema — با دقت بخوان |
| `src/repo/*.ts` | ۴۵۰ | 🔴 بالا | کوئری‌های جدید |
| `src/api/*.ts` | ۳۰۰ | 🟡 متوسط | endpointها |
| `src/__tests__/*` | ۷۰۰ | 🟢 پایین | تست‌ها — سریع رد کن |
| `*.snap` | ۲۰۰ | ⚪ نادیده | snapshot |

### ترتیب پیشنهادی review
۱. migration (۱۲۰ خط) — آیا reversible است؟
۲. repo — آیا N+1 دارد؟
۳. api — آیا اعتبارسنجی دارد؟
۴. بقیه را نگاه کن
```

**ب) جلسه‌ی review زنده:** ۳۰ دقیقه با screen share، سریع‌تر از ۵۰ کامنت.

**ج) review تدریجی:** reviewer هر روز یک بخش را review کند و `Viewed` بزند.

**د) خودت اول review کن:** `git diff` را کامل بخوان و کامنت‌های خودت را بگذار. نصف مسائل را خودت پیدا می‌کنی.

#### مثال ۵: شکستن یک PR موجود

```bash
# فرض: branch feat/big با ۲۰ commit و ۱۵۰۰ خط
git switch feat/big

# ۱) branch اول: refactor
git switch -c feat/big-part1 main
git cherry-pick <hash-refactor-1> <hash-refactor-2>
git push -u origin feat/big-part1
gh pr create --base main --title "refactor: آماده‌سازی برای گزارش"

# ۲) branch دوم: API (روی part1)
git switch -c feat/big-part2 feat/big-part1
git cherry-pick <hash-api-1> <hash-api-2> <hash-api-3>
git push -u origin feat/big-part2
gh pr create --base feat/big-part1 --title "feat: API گزارش"

# ۳) و به همین ترتیب...
```

#### مثال ۶: کاهش diff با `.gitattributes`

```
*.snap -diff
package-lock.json -diff
dist/* -diff
```

حالا lock file و snapshotها diff را شلوغ نمی‌کنند.

#### مثال ۷: جدا کردن تغییرات فرمت

اگر formatter را عوض کردی، **یک PR جدا** بزن:

```bash
# PR جدا فقط برای فرمت
npx prettier --write .
git commit -am "style: اجرای prettier روی کل پروژه"
```

و در `.git-blame-ignore-revs` ثبت کن تا `git blame` آن را نادیده بگیرد:

```bash
echo "<commit-hash>" >> .git-blame-ignore-revs
git config blame.ignoreRevsFile .git-blame-ignore-revs
```

GitHub هم از این فایل پشتیبانی می‌کند و blame را تمیز نگه می‌دارد.

### مقایسه و الگوهای مختلف

#### Feature flag در برابر branch بلندعمر

| | Feature flag | branch بلندعمر |
|---|---|---|
| merge مکرر | ✅ | ❌ |
| conflict | کم | زیاد |
| کد مرده در main | ✅ (موقت) | ❌ |
| نیاز به انضباط | ✅ (پاک کردن flag) | ❌ |

#### Refactor-first در برابر Feature-first

| | Refactor-first | Feature-first |
|---|---|---|
| PR اول | تغییر ساختار | ویژگی |
| review | آسان (رفتار ثابت) | سخت‌تر |
| ریسک | کم | متوسط |
| توصیه | ✅ | فقط اگر refactor کوچک است |

#### ابزارهای اندازه‌گیری

| ابزار | کار |
|---|---|
| **`gh pr view --json additions,deletions`** | اندازه |
| **Danger JS** | هشدار خودکار در PR بزرگ |
| **`pr-size-labeler`** | برچسب `size/L`، `size/XL` |
| **Reviewable** | review تدریجی |

```yaml
# برچسب خودکار اندازه
- uses: pascalgn/size-label-action@v0.5.5
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    sizes: >
      {
        "0": "XS",
        "40": "S",
        "100": "M",
        "200": "L",
        "500": "XL"
      }
```

### اشکال شایع

#### ۱. «بعداً می‌شکنم»

هرگز نمی‌شود. PR بزرگ‌تر می‌شود، conflict بیشتر، انگیزه کمتر. **همین حالا بشکن.**

#### ۲. شکستن به PRهای غیرقابل merge

اگر PR #1 قابل merge نباشد (چون کد ناتمام است)، کل زنجیره گیر می‌کند. **PR اول باید مستقل و بی‌خطر باشد.**

#### ۳. Feature flag که پاک نمی‌شود

```
✗ if (flags.newCheckout) ...   // ۲ سال بعد
```

برای هر flag یک Issue با تاریخ انقضا بساز.

#### ۴. Diff پر از فایل تولیدشده

`dist/`، `*.snap`، `package-lock.json` — همه در diff. `.gitattributes` با `-diff` حلش می‌کند.

#### ۵. PR بزرگ بدون راهنمای review

reviewer باز می‌کند، ۴۰ فایل می‌بیند، می‌بندد. راهنما بنویس.

#### ۶. merge سریع PR بزرگ بدون review

بدترین حالت: «فوری بود، merge کردم». بعداً باید کل PR را revert کنی.

### نکات مهم (Tips & Tricks)

- **قاعده‌ی ۴۰۰ خط:** اگر بیشتر شد، بشکن.
- **`refactor` را همیشه اول** — کد تمیز، feature راحت‌تر.
- **Feature flag** برای هر چیزی که بیش از ۲ روز طول می‌کشد.
- **`.git-blame-ignore-revs`** برای PRهای فرمت.
- **راهنمای review** در توضیح PR، نرخ review را چند برابر می‌کند.
- **`gh pr view --json additions`** را در اسکریپت چک کن و اگر > ۸۰۰ بود، هشدار بده.

### خلاصه فصل

- PR بزرگ review نمی‌شود؛ ۴۰۰ خط سقف عملی است.
- سه تکنیک شکستن: refactor جدا، feature flag، vertical slicing.
- اگر مجبوری بزرگ بفرستی: راهنمای review + جلسه‌ی زنده.
- `.gitattributes` و `.git-blame-ignore-revs` diff را تمیز می‌کنند.
- **PR اول زنجیره باید مستقل و بی‌خطر باشد.**

**سؤالات مروری:**
1. چرا ۴۰۰ خط سقف عملی review است؟
2. Feature flag چطور اجازه‌ی merge کد ناتمام را می‌دهد؟
3. چطور یک PR ۱۵۰۰ خطی را به سه PR بشکنی؟

---

**بعدی:** [`09-Branch-Protection-and-Rulesets.md`](09-Branch-Protection-and-Rulesets.md)

---

## منابع و مراجع

- GitHub Docs: About pull requests — https://docs.github.com/en/pull-requests
- GitHub Docs: Reviewing changes in pull requests — https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests
- GitHub Docs: Merge queue — https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue
- GitHub Docs: Auto-merge — https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/automatically-merging-a-pull-request
- Google Engineering Practices: Code Review — https://google.github.io/eng-practices/review/
- "Best Kept Secrets of Peer Code Review" (SmartBear) — https://smartbear.com/
- `gh-stack` — https://github.com/github/gh-stack
