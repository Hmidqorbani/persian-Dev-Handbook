# بخش ۰۹: محافظت از شاخه و Rulesets

**مدت یادگیری**: ۳ تا ۴ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۰۸ (PR، review)

---

## فهرست محتوا

- [فصل ۳۸: Branch protection کلاسیک](#فصل-۳۸)
- [فصل ۳۹: Rulesets — نسل جدید حاکمیت](#فصل-۳۹)
- [فصل ۴۰: Required checks و تعامل با Actions](#فصل-۴۰)
- [فصل ۴۱: محافظت از تگ‌ها و فایل‌های حساس](#فصل-۴۱)

---

## فصل ۳۸: Branch protection کلاسیک

### مقدمه

Branch protection قواعدی است که می‌گوید چه کسی و چطور می‌تواند به یک branch بنویسد. بدون آن، هر کسی با دسترسی write می‌تواند مستقیم روی `main` push کند، تست‌ها را دور بزند و کد ناشکسته را بشکند.

**چرا مهم است؟** تمام تضمین‌های کیفیت تیم (review، CI، تست) فقط وقتی معنی دارند که **دور زدنشان ممکن نباشد**. Branch protection همان قفل است.

**کاربرد عملی:** یک بار روی `main` تنظیم می‌کنی؛ از آن به بعد، هیچ PRی نمی‌تواند بدون review و CI merge شود — حتی PR خودت.

### مفاهیم اساسی

#### گزینه‌های اصلی

| گزینه | کار |
|---|---|
| **Require a pull request before merging** | هیچ push مستقیمی به branch ممکن نیست |
| ↳ Required approvals | حداقل تعداد approve |
| ↳ Dismiss stale approvals | با commit جدید، approveها باطل شوند |
| ↳ Require review from Code Owners | CODEOWNERS باید approve کنند |
| ↳ Require approval of the most recent reviewable push | آخرین push باید approve شود |
| **Require status checks to pass** | CI باید سبز باشد |
| ↳ Require branches to be up to date | branch باید با base همگام باشد |
| **Require conversation resolution** | همه‌ی threadها resolve شده باشند |
| **Require signed commits** | commitها امضا داشته باشند |
| **Require linear history** | merge commit ممنوع (فقط squash/rebase) |
| **Require deployments to succeed** | محیط deploy باید موفق باشد |
| **Lock branch** | branch فقط خواندنی |
| **Do not allow bypassing** | حتی admin هم نتواند دور بزند |
| **Restrict who can push** | فقط افراد/تیم‌های مشخص |
| **Allow force pushes** | ❌ (همیشه خاموش) |
| **Allow deletions** | ❌ (همیشه خاموش) |

#### ترکیب پیشنهادی برای `main`

```
☑ Require a pull request before merging
    Required approvals: 1
    ☑ Dismiss stale approvals
    ☑ Require review from Code Owners
☑ Require status checks to pass
    ☑ Require branches to be up to date
    checks: lint, test, build
☑ Require conversation resolution
☑ Require linear history
☐ Allow force pushes
☐ Allow deletions
```

### مثال‌های عملی

#### مثال ۱: تنظیم با UI

**Settings → Branches → Add branch protection rule** (یا **Add classic branch protection rule**):

```
Branch name pattern: main

☑ Require a pull request before merging
   Required number of approvals: 1
   ☑ Dismiss stale pull request approvals when new commits are pushed
   ☑ Require review from Code Owners
   ☐ Require approval of the most recent reviewable push

☑ Require status checks to pass before merging
   ☑ Require branches to be up to date before merging
   [جست‌وجو و انتخاب: ci / test، ci / lint]

☑ Require conversation resolution before merging
☑ Require linear history
☐ Require signed commits

☐ Allow force pushes
☐ Allow deletions

☑ Do not allow bypassing the above settings
```

#### مثال ۲: تنظیم با API

```bash
gh api -X PUT repos/OWNER/REPO/branches/main/protection \
  --input - <<'JSON'
{
  "required_status_checks": {
    "strict": true,
    "contexts": ["ci / test", "ci / lint"]
  },
  "enforce_admins": true,
  "required_pull_request_reviews": {
    "dismiss_stale_reviews": true,
    "require_code_owner_reviews": true,
    "required_approving_review_count": 1
  },
  "required_conversation_resolution": true,
  "required_linear_history": true,
  "allow_force_pushes": false,
  "allow_deletions": false,
  "restrictions": null
}
JSON
```

#### مثال ۳: بررسی وضعیت فعلی

```bash
gh api repos/OWNER/REPO/branches/main/protection | jq

# یا خلاصه
gh api repos/OWNER/REPO/branches/main/protection \
  --jq '{
    reviews: .required_pull_request_reviews.required_approving_review_count,
    strict: .required_status_checks.strict,
    checks: .required_status_checks.contexts,
    admins: .enforce_admins.enabled
  }'
```

#### مثال ۴: branch pattern — چند branch با یک قاعده

```
main
release/*
v*.*.*
```

می‌توانی یک قاعده برای `main`، یکی برای `release/*` و یکی برای تگ‌ها بسازی.

**توجه:** الگوها glob هستند. `release/*` فقط یک سطح را می‌گیرد، `release/**` همه‌ی سطوح را.

#### مثال ۵: دسترسی push محدود

```
☑ Restrict who can push to matching branches
  Teams: @myorg/release-managers
  Apps: (هیچ)
```

این برای `main` در سازمان‌های بزرگ مفید است: حتی با دسترسی write، فقط تیم release می‌تواند push کند.

**نکته:** با «Require a pull request» فعال، این گزینه عملاً برای bypass است.

### مقایسه و الگوهای مختلف

#### Branch protection در برابر Rulesets

| | Branch protection | Rulesets |
|---|---|---|
| چند branch با یک قاعده | ❌ (per-branch) | ✅ |
| تگ‌ها | ❌ | ✅ |
| لایه‌بندی (چند ruleset) | ❌ | ✅ |
| Bypass list | محدود | ✅ صریح |
| وضعیت | legacy (ولی کار می‌کند) | ✅ توصیه‌شده |
| API | REST | REST + GraphQL |

**توصیه:** برای پروژه‌ی جدید، از Rulesets استفاده کن (فصل ۳۹). برای پروژه‌ی موجود، مهاجرت تدریجی.

#### Require linear history — روشن یا خاموش؟

| | روشن | خاموش |
|---|---|---|
| merge commit | ❌ ممنوع | ✅ مجاز |
| تاریخ | خطی | شاخه‌دار |
| bisect | ساده | پیچیده‌تر |
| revert merge | لازم نیست | مشکل‌دار |
| مناسب | تیم با squash | تیم با merge commit |

اگر `merge --no-ff` می‌خواهی، این را خاموش کن.

#### Dismiss stale approvals

| | روشن | خاموش |
|---|---|---|
| امنیت | ✅ (کد جدید = review جدید) | ❌ |
| راحتی | ❌ (باید دوباره approve) | ✅ |
| مناسب | کد حساس | مستندات |

### اشکال شایع

#### ۱. «Do not allow bypassing» را روشن نکردی

اگر خاموش باشد، adminها می‌توانند دور بزنند. در تیم کوچک شاید مشکلی نباشد، ولی در سازمان یعنی قاعده‌ای وجود ندارد.

#### ۲. اسم check اشتباه

```
Required: "test"
Actual:   "CI / test"
```

اسم دقیق را از صفحه‌ی PR کپی کن. اگر matrix داری: `test (18.x)`، `test (20.x)`.

#### ۳. `Require branches to be up to date` + تیم فعال = صف rebase

هر merge، بقیه را باطل می‌کند. راه‌حل: Merge Queue (فصل ۳۶).

#### ۴. `Require signed commits` بدون تنظیم امضا

همه‌ی PRها block می‌شوند. اول تیم را آموزش بده و امضا را تنظیم کن (فصل ۸۸).

#### ۵. `Lock branch` به‌جای Archive

`Lock` فقط نوشتن را ممنوع می‌کند، branch هنوز هست. برای پروژه‌ی تمام‌شده، Archive مخزن بهتر است.

#### ۶. فراموش کردن branchهای `release/*`

اگر فقط `main` محافظت شده باشد، `release/1.4` آزاد است — که همان‌قدر خطرناک است.

### نکات مهم (Tips & Tricks)

- **`enforce_admins: true`** — بدون آن، قاعده تشریفاتی است.
- **`Require conversation resolution`** را روشن کن — threadهای باز، محل باگ‌های فراموش‌شده‌اند.
- **الگوی `*` را برای همه‌ی branchها با قاعده‌ی سبک بگذار** (مثلاً فقط منع force push).
- **قبل از روشن کردن `Require signed commits`، یک هفته اطلاع‌رسانی کن.**
- **`gh api` را برای backup تنظیمات استفاده کن** — یک فایل JSON از protection بساز و در مخزن نگه دار.

### خلاصه فصل

- Branch protection تضمین می‌کند review و CI دور زده نشوند.
- ترکیب پیشنهادی: PR اجباری + ۱ approve + status checks + conversation resolution + linear history.
- **`enforce_admins: true`** حیاتی است.
- اسم required check باید دقیقاً با اسم job بخواند.
- برای پروژه‌ی جدید، Rulesets بهتر است.

**سؤالات مروری:**
1. چرا `enforce_admins` مهم است؟
2. چرا `Require branches to be up to date` با تیم فعال مشکل‌ساز است؟
3. چطور اسم دقیق required check را پیدا کنی؟

---

## فصل ۳۹: Rulesets — نسل جدید حاکمیت

### مقدمه

Rulesets نسخه‌ی بازنویسی‌شده و بسیار قدرتمندتر Branch Protection است. سه مزیت کلیدی: **چند branch با یک قاعده**، **محافظت از تگ**، و **لایه‌بندی** (چند ruleset با هم).

**چرا مهم است؟** در یک سازمان با ۵۰ مخزن، تنظیم branch protection روی هر کدام یک کابوس است. Rulesets در سطح سازمان یک بار تنظیم می‌شود و روی همه اعمال می‌شود.

**کاربرد عملی:** اگر بیش از ۵ مخزن داری یا می‌خواهی از تگ‌های release محافظت کنی، Rulesets پاسخ است.

### مفاهیم اساسی

#### ساختار

```
Ruleset
├── Target (branch یا tag)
├── Enforcement status (Active / Evaluate / Disabled)
├── Bypass list (actors)
├── Target branches/tags (patterns)
└── Rules (قواعد)
    ├── Restrict creations / updates / deletions
    ├── Require pull request
    ├── Require status checks
    ├── Require signed commits
    ├── Require linear history
    ├── Require deployments
    ├── Block force pushes
    ├── Require code scanning results
    └── Require code quality results
```

#### سه سطح اعمال

| سطح | معنی | مناسب |
|---|---|---|
| **Active** | قاعده اجرا می‌شود | تولید |
| **Evaluate** | فقط گزارش می‌دهد (بدون مسدود کردن) | آزمایش قبل از اعمال |
| **Disabled** | خاموش | موقت |

**`Evaluate` یک قابلیت فوق‌العاده است:** می‌توانی یک قاعده را یک هفته در حالت observe بگذاری، ببینی چند PR را می‌شکند، بعد فعال کنی.

#### Rulesets در برابر Branch protection

| ویژگی | Branch protection | Rulesets |
|---|---|---|
| چند branch | ❌ | ✅ (glob patterns) |
| تگ | ❌ | ✅ |
| لایه‌بندی | ❌ | ✅ (چند ruleset) |
| Bypass صریح | محدود | ✅ (actor list) |
| حالت Evaluate | ❌ | ✅ |
| سطح سازمان | محدود | ✅ |
| Code scanning required | ❌ | ✅ |
| Import از protection | — | ✅ |

### مثال‌های عملی

#### مثال ۱: Ruleset برای main

**Settings → Rules → Rulesets → New ruleset → New branch ruleset**

```
Name: Protect main
Enforcement status: Active

Target branches:
  ☑ Include default branch
  (یا) Add target → Include by pattern: main

Bypass list:
  - Organization admin (اگر واقعاً لازم است)

Rules:
  ☑ Restrict deletions
  ☑ Block force pushes
  ☑ Require a pull request before merging
      Required approvals: 1
      ☑ Dismiss stale approvals
      ☑ Require review from Code Owners
      ☑ Require approval of the most recent reviewable push
  ☑ Require status checks to pass
      ☑ Require branches to be up to date
      checks: ci / test, ci / lint
  ☑ Require conversation resolution
  ☑ Require linear history
  ☐ Require signed commits
```

#### مثال ۲: Ruleset برای همه‌ی branchها (سبک)

```
Name: Baseline
Target: Include all branches (*)

Rules:
  ☑ Restrict deletions
  ☑ Block force pushes
```

این جلوی فاجعه‌های ساده را می‌گیرد (حذف branch، force push) بدون اینکه جریان کار را کند کند.

#### مثال ۳: Ruleset برای تگ‌ها

```
Name: Protect releases
Target: Tags
Pattern: v*.*.*

Rules:
  ☑ Restrict creations
  ☑ Restrict updates
  ☑ Restrict deletions

Bypass: @myorg/release-managers
```

حالا کسی نمی‌تواند تگ `v1.0.0` را جابه‌جا یا پاک کند — که برای پکیج‌های منتشرشده حیاتی است.

#### مثال ۴: Ruleset در سطح سازمان

در **Organization Settings → Repository → Rulesets**:

```
Name: Org baseline
Enforcement: Active
Target: All repositories
  (یا) Selected repositories

Rules:
  ☑ Block force pushes
  ☑ Restrict deletions
```

هر مخزن جدید خودکار این قاعده را می‌گیرد. **این بزرگ‌ترین مزیت Rulesets است.**

#### مثال ۵: تنظیم با API

```bash
# ساخت ruleset
gh api -X POST repos/OWNER/REPO/rulesets --input - <<'JSON'
{
  "name": "protect-main",
  "target": "branch",
  "enforcement": "active",
  "conditions": {
    "ref_name": {
      "include": ["~DEFAULT_BRANCH"],
      "exclude": []
    }
  },
  "rules": [
    { "type": "deletion" },
    { "type": "non_fast_forward" },
    {
      "type": "pull_request",
      "parameters": {
        "required_approving_review_count": 1,
        "dismiss_stale_reviews_on_push": true,
        "require_code_owner_review": true,
        "require_last_push_approval": false,
        "required_review_thread_resolution": true
      }
    },
    {
      "type": "required_status_checks",
      "parameters": {
        "strict_required_status_checks_policy": true,
        "required_status_checks": [
          { "context": "ci / test" },
          { "context": "ci / lint" }
        ]
      }
    }
  ],
  "bypass_actors": []
}
JSON
```

#### مثال ۶: لیست و بررسی

```bash
gh api repos/OWNER/REPO/rulesets | jq '.[] | {id, name, target, enforcement}'

# جزئیات
gh api repos/OWNER/REPO/rulesets/12345 | jq

# قوانین فعال روی یک branch
gh api repos/OWNER/REPO/rules/branches/main | jq '.[].type'
```

#### مثال ۷: Evaluate mode — روش امن

```
Enforcement: Evaluate

# یک هفته بگذار کار کند
# بعد ببین چند PR را می‌شکست:
gh api repos/OWNER/REPO/rulesets/12345/rule-suites | jq '.[] | {result, evaluated_at}'

# اگر مشکلی نبود:
# Enforcement → Active
```

#### مثال ۸: Bypass list

```
Bypass list:
  - Organization admin        (همه‌ی adminها)
  - Repository admin
  - Team: @myorg/release-bots
  - App: dependabot
```

**نکته‌ی مهم:** `bypass_actors` در API با `actor_id` و `actor_type` کار می‌کند:

```json
"bypass_actors": [
  { "actor_id": 5, "actor_type": "RepositoryRole", "bypass_mode": "always" },
  { "actor_id": 123, "actor_type": "Team", "bypass_mode": "pull_request" }
]
```

`bypass_mode: "pull_request"` یعنی این actor باید PR بزند (نه push مستقیم) — برای ربات‌ها مفید.

### مقایسه و الگوهای مختلف

#### استراتژی لایه‌بندی

```
Ruleset 1 (Org, all repos):     Block force push + delete
Ruleset 2 (Repo, main):         Require PR + 1 approve
Ruleset 3 (Repo, main):         Require status checks
Ruleset 4 (Repo, release/*):    Require 2 approves + signed commits
Ruleset 5 (Repo, tags v*):      Block update/delete
```

هر ruleset یک نگرانی دارد. تغییر یکی، بقیه را نمی‌شکند.

#### `~DEFAULT_BRANCH` و الگوهای ویژه

| الگو | معنی |
|---|---|
| `~DEFAULT_BRANCH` | branch پیش‌فرض (main) |
| `~ALL` | همه‌ی branchها |
| `main` | دقیقاً main |
| `release/*` | یک سطح زیر release |
| `release/**` | همه‌ی سطوح |

#### Bypass: چه کسی؟

| Actor | مناسب |
|---|---|
| Organization admin | ⚠️ فقط اگر واقعاً لازم است |
| Repository admin | ⚠️ |
| Team (مثلاً release-managers) | ✅ |
| GitHub App (dependabot، renovate) | ✅ |
| Deploy key | ✅ (برای automation) |

**اصل:** حداقل bypass. اگر همه bypass دارند، ruleset وجود ندارد.

### اشکال شایع

#### ۱. Ruleset و Branch protection هم‌زمان فعال

هر دو اعمال می‌شوند و نتیجه ممکن است غیرمنتظره باشد (سخت‌گیرانه‌ترین برنده است). **قاعده:** یکی را انتخاب کن. اگر مهاجرت می‌کنی، اول protection را بردار، بعد ruleset را فعال کن.

#### ۲. Bypass برای admin و بعد فراموش کردن

اگر `Organization admin` را bypass بگذاری، خودت می‌توانی دور بزنی — که خوب است برای اضطراری، بد است برای انضباط. لاگ audit را چک کن.

#### ۳. الگوی `*` همه‌چیز را می‌گیرد

`*` در Rulesets همه‌ی branchها را می‌گیرد (برخلاف `.gitignore`). اگر می‌خواهی فقط `main`، صریح بنویس.

#### ۴. Required check با نام اشتباه

مثل branch protection. اسم را از صفحه‌ی PR کپی کن.

#### ۵. Ruleset روی تگ، بدون pattern

اگر `v*.*.*` را نگذاری، تگ‌های دیگر محافظت نمی‌شوند.

#### ۶. فراموش کردن `non_fast_forward`

این قاعده force push را ممنوع می‌کند. بدون آن، تمام محافظت‌های دیگر قابل دور زدن است (force push تاریخ را بازنویسی می‌کند).

#### ۷. Evaluate را جدی نگرفتن

`Evaluate` یک هفته وقت می‌گیرد ولی جلوی شکستن جریان کار تیم را می‌گیرد. استفاده کن.

### نکات مهم (Tips & Tricks)

- **`~DEFAULT_BRANCH`** به‌جای `main` — اگر روزی نام عوض شد، ruleset می‌شکند.
- **`non_fast_forward`** را در همه‌ی rulesetها بگذار — ارزان‌ترین محافظت.
- **Ruleset سطح سازمان** برای baseline، Ruleset مخزن برای جزئیات.
- **`Evaluate` یک هفته، بعد `Active`.**
- **Bypass را مکتوب کن** — در `CONTRIBUTING.md` بنویس چه کسی و چرا.
- **`gh api repos/.../rulesets` را در backup بگذار** — تنظیمات را در مخزن `infra` نگه دار.

### خلاصه فصل

- Rulesets جانشین مدرن Branch protection: چند branch، تگ، لایه‌بندی، Evaluate.
- **سطح سازمان** = تنظیم یک‌بار برای همه‌ی مخزن‌ها.
- `non_fast_forward` و `deletion` ارزان‌ترین و مهم‌ترین قواعد.
- `Evaluate` قبل از `Active` — روش امن.
- Bypass را حداقل نگه دار و مکتوب کن.

**سؤالات مروری:**
1. سه مزیت Rulesets نسبت به Branch protection چیست؟
2. حالت `Evaluate` چه کاری می‌کند؟
3. چرا `non_fast_forward` حیاتی است؟

---

## فصل ۴۰: Required checks و تعامل با Actions

### مقدمه

Required status checks تضمین می‌کنند که یک PR فقط وقتی merge شود که CI سبز است. ساده به نظر می‌رسد، ولی ۹۰٪ «PR گیر کرده»ها از ناهماهنگی اسم check و job می‌آید.

**چرا مهم است؟** بدون required checks، CI فقط یک اطلاع است — می‌شود نادیده گرفت. با آن، CI یک دروازه است.

**کاربرد عملی:** بعد از این فصل می‌دانی چطور اسم check را دقیق پیدا کنی، چرا matrix اسم را عوض می‌کند، و چطور با skip کردن کنار بیایی.

### مفاهیم اساسی

#### check name از کجا می‌آید

```
.github/workflows/ci.yml
  name: CI                    ← نام workflow
  jobs:
    test:                     ← نام job
```

اسم check در GitHub:

- اگر job `name:` نداشته باشد: `<workflow name> / <job id>` → `CI / test`
- اگر job `name:` داشته باشد: `<workflow name> / <job name>` → `CI / تست واحد`
- با matrix: `CI / test (18.x)`، `CI / test (20.x)`

**قاعده:** اسم دقیق را از تب **Checks** در PR یا از `gh pr checks` کپی کن.

#### انواع check

| نوع | منبع |
|---|---|
| **Actions** | workflow |
| **Third-party** | GitHub App (Vercel، Netlify، CircleCI) |
| **Commit status** | API `POST /repos/:owner/:repo/statuses/:sha` |

همه‌ی اینها می‌توانند required باشند.

#### وضعیت‌های check

| وضعیت | معنی |
|---|---|
| `queued` | در صف |
| `in_progress` | در حال اجرا |
| `completed / success` | سبز |
| `completed / failure` | قرمز |
| `completed / neutral` | خنثی (مثل skip) |
| `completed / skipped` | رد شده |

**نکته‌ی مهم:** `skipped` معمولاً **قبول** می‌شود (به‌عنوان success در نظر گرفته می‌شود). `neutral` هم.

### مثال‌های عملی

#### مثال ۱: پیدا کردن اسم دقیق

```bash
gh pr checks 42
# All checks were successful
# ✓  CI / test (18.x)   1m23s
# ✓  CI / test (20.x)   1m31s
# ✓  CI / lint          32s
# ✓  Vercel Preview     45s

# اسم دقیق برای required:
# "CI / test (18.x)"
# "CI / test (20.x)"
# "CI / lint"
```

#### مثال ۲: workflow با اسم‌های واضح

```yaml
name: CI

on:
  pull_request:
  merge_group:

jobs:
  lint:
    name: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: npm }
      - run: npm ci
      - run: npm run lint

  test:
    name: test (${{ matrix.node }})
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        node: [20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: ${{ matrix.node }}, cache: npm }
      - run: npm ci
      - run: npm test

  build:
    name: build
    needs: [lint, test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: npm }
      - run: npm ci
      - run: npm run build
```

اسم‌های تولیدشده: `CI / lint`، `CI / test (20)`، `CI / test (22)`، `CI / build`.

#### مثال ۳: مشکل matrix و required checks

اگر matrix را از `[18, 20]` به `[20, 22]` تغییر دهی، اسم check از `test (18)` به `test (22)` عوض می‌شود. required check قدیمی دیگر وجود ندارد → **PRها برای همیشه pending می‌مانند**.

**راه‌حل ۱:** یک job جمع‌کننده بساز:

```yaml
jobs:
  test:
    strategy:
      matrix:
        node: [20, 22]
    # ...

  ci-success:
    name: ci-success        # ← فقط این required است
    needs: [lint, test, build]
    if: always()
    runs-on: ubuntu-latest
    steps:
      - name: بررسی نتیجه
        run: |
          if [ "${{ contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled') }}" = "true" ]; then
            echo "یکی از jobها شکست خورد"
            exit 1
          fi
          echo "همه سبز"
```

حالا فقط `CI / ci-success` را required کن. matrix هر چه باشد، این job ثابت است.

**راه‌حل ۲:** از `paths-ignore` استفاده نکن برای jobهای required.

#### مثال ۴: skip کردن هوشمند

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    if: |
      !contains(github.event.head_commit.message, '[skip ci]') &&
      !contains(github.event.pull_request.title, '[skip ci]')
    steps:
      - run: npm test
```

**تله:** اگر job به‌خاطر `if` اجرا نشود، check با وضعیت `skipped` ثبت می‌شود — که required را پاس می‌کند! یعنی کسی می‌تواند با `[skip ci]` CI را دور بزند.

**راه‌حل امن:** `[skip ci]` را فقط برای branchهای غیر-main مجاز کن، یا اصلاً نگذار.

#### مثال ۵: check از سرویس خارجی

Vercel، Netlify، CircleCI و ... check ثبت می‌کنند:

```
Vercel – my-app        (Deployment)
Vercel Preview Comments
```

اینها هم می‌توانند required باشند. اسم را از تب Checks کپی کن.

#### مثال ۶: check که هرگز اجرا نمی‌شود

اگر required check را تنظیم کردی ولی workflow روی آن event اجرا نمی‌شود:

```yaml
✗ on:
    push:
      branches: [main]
```

PR هیچ‌وقت check نمی‌گیرد. باید `pull_request` هم باشد.

#### مثال ۷: `paths-ignore` و required check

```yaml
on:
  pull_request:
    paths-ignore:
      - 'docs/**'
```

اگر PR فقط `docs/` را عوض کند، workflow اجرا نمی‌شود، check ثبت نمی‌شود، و اگر required باشد، **PR برای همیشه pending می‌ماند**.

**راه‌حل:** job جمع‌کننده (`ci-success`) با `if: always()` که وقتی workflow اجرا نشد هم... نمی‌شود، چون workflow اجرا نشده.

**راه‌حل واقعی:** `paths-ignore` را با required check ترکیب نکن. یا از job-level `if` استفاده کن، یا required را بردار.

### مقایسه و الگوهای مختلف

#### یک check در برابر چند check

| | یک check (`ci-success`) | چند check |
|---|---|---|
| مدیریت ruleset | ✅ ساده | ❌ با هر تغییر matrix می‌شکند |
| دیدن جزئیات | نیاز به باز کردن | ✅ مستقیم |
| توصیه | ✅ | برای تیم کوچک |

#### `strict` (up to date) در برابر غیر strict

| | strict | غیر strict |
|---|---|---|
| CI روی کد نهایی | ✅ | ❌ (ممکن است base عوض شده باشد) |
| نیاز به sync | هر merge | فقط conflict |
| مناسب | تیم کوچک | با Merge Queue |

#### `skipped` در برابر `neutral` در برابر `success`

| نتیجه | required را پاس می‌کند؟ |
|---|---|
| `success` | ✅ |
| `skipped` | ✅ |
| `neutral` | ✅ |
| `failure` | ❌ |
| `cancelled` | ❌ |
| `timed_out` | ❌ |
| `action_required` | ❌ |

**تله:** `skipped` پاس می‌کند — پس `if` شرطی می‌تواند CI را دور بزند.

### اشکال شایع

#### ۱. «Expected — Waiting for status to be reported»

یعنی اسم check در ruleset هست ولی هیچ‌وقت گزارش نمی‌شود. دلایل:

- workflow روی آن event اجرا نمی‌شود
- اسم job عوض شده
- matrix عوض شده
- workflow در branch دیگری است

#### ۲. matrix را عوض کردم و PR گیر کرد

راه‌حل بلندمدت: `ci-success` job. راه‌حل فوری: ruleset را ویرایش کن.

#### ۳. check سبز است ولی merge نمی‌شود

- `strict: true` و branch عقب است → sync کن
- review لازم است
- conversation resolution

#### ۴. check از fork اجرا نمی‌شود

PR از fork، workflow نیاز به تأیید maintainer دارد. `Settings → Actions → Fork pull request workflows`.

#### ۵. required check روی branch اشتباه

ruleset برای `main` است ولی PR به `develop` می‌رود.

#### ۶. `if: always()` در ci-success اشتباه

```yaml
✗ if: always()   # حتی وقتی lint شکست خورد، موفق می‌شود؟
```

باید نتیجه را چک کنی:

```yaml
- if: contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')
  run: exit 1
```

### نکات مهم (Tips & Tricks)

- **`ci-success` job** بهترین راه‌حل برای matrix و required checks.
- **اسم check را از `gh pr checks` کپی کن**، نه از حافظه.
- **`paths-ignore` + required = دردسر.** اجتناب کن.
- **`skipped` پاس می‌کند** — `if` شرطی می‌تواند دروازه را باز کند.
- **`workflow_dispatch`** به workflowهای required اضافه کن — برای اجرای دستی در اضطراری.
- **بعد از تغییر matrix، ruleset را چک کن.**

### خلاصه فصل

- اسم check = `<workflow name> / <job name>` (+ matrix).
- matrix اسم را عوض می‌کند → از `ci-success` استفاده کن.
- `skipped` و `neutral` required را پاس می‌کنند — مراقب `if` باش.
- `paths-ignore` با required check سازگار نیست.
- اسم دقیق را از `gh pr checks` بگیر.

**سؤالات مروری:**
1. چرا تغییر matrix می‌تواند PRها را برای همیشه pending بگذارد؟
2. `ci-success` job چه مشکلی را حل می‌کند؟
3. چرا `[skip ci]` می‌تواند یک حفره‌ی امنیتی باشد؟

---

## فصل ۴۱: محافظت از تگ‌ها و فایل‌های حساس

### مقدمه

Branchها فقط نیمی از داستان‌اند. تگ‌ها (که releaseها به آنها وصل‌اند) و فایل‌های حساس (workflowها، CODEOWNERS، migrationها) هم نیاز به محافظت دارند.

**چرا مهم است؟** اگر کسی بتواند تگ `v1.0.0` را جابه‌جا کند، تمام کسانی که آن نسخه را نصب کرده‌اند، چیز متفاوتی از آنچه فکر می‌کنند دارند. و اگر کسی بتواند `.github/workflows/` را تغییر دهد، می‌تواند secrets را بدزدد.

**کاربرد عملی:** این فصل لایه‌ی نهایی محافظت است.

### مفاهیم اساسی

#### چه چیزی محافظت لازم دارد

| هدف | چرا |
|---|---|
| **تگ‌های release** | تغییر تگ = تغییر نسخه‌ی منتشرشده |
| **`.github/workflows/`** | دسترسی به secrets و deploy |
| **`.github/CODEOWNERS`** | تعیین reviewer |
| **`CODEOWNERS` در ریشه** | همان |
| **migrationهای DB** | تغییر schema در production |
| **`SECURITY.md`** | کانال گزارش آسیب‌پذیری |
| **فایل‌های infra (Terraform، K8s)** | دسترسی زیرساخت |
| **`package.json` / lock** | زنجیره‌ی تأمین |

#### سه لایه‌ی محافظت

```
۱. Ruleset روی branch (main) — همه‌ی تغییرات از PR
۲. CODEOWNERS — متخصص هر مسیر باید approve کند
۳. Ruleset روی tag — تگ‌های منتشرشده قابل تغییر نیستند
```

### مثال‌های عملی

#### مثال ۱: Ruleset برای تگ‌ها

```
Name: Protect release tags
Target: Tags
Pattern: v*.*.*

Rules:
  ☑ Restrict creations       (فقط release-managers)
  ☑ Restrict updates         (هیچ‌کس)
  ☑ Restrict deletions       (هیچ‌کس)

Bypass:
  - Team: @myorg/release-managers  (فقط برای creation)
```

**توجه:** `Restrict updates` و `Restrict deletions` نباید bypass داشته باشند — تگ منتشرشده تغییرناپذیر است.

#### مثال ۲: CODEOWNERS برای فایل‌های حساس

`.github/CODEOWNERS`:

```gitignore
# پیش‌فرض
*                           @myorg/core-team

# workflowها — دسترسی به secrets
/.github/workflows/         @myorg/security-team @myorg/core-team
/.github/actions/           @myorg/security-team

# CODEOWNERS خودش
/.github/CODEOWNERS         @myorg/security-team

# زیرساخت
/infra/                     @myorg/platform-team
/terraform/                 @myorg/platform-team
/k8s/                       @myorg/platform-team

# دیتابیس
/db/migrations/             @myorg/dba-team
*.sql                       @myorg/dba-team

# امنیت
SECURITY.md                 @myorg/security-team

# وابستگی‌ها
package.json                @myorg/core-team
package-lock.json           @myorg/core-team
requirements.txt            @myorg/core-team
```

و در Ruleset: **Require review from Code Owners** ✅

#### مثال ۳: جلوگیری از تغییر workflow توسط PR خارجی

یک لایه‌ی اضافی: workflow که چک می‌کند آیا PR فایل‌های حساس را عوض کرده و آیا نویسنده مجاز است:

```yaml
name: محافظت از فایل‌های حساس

on:
  pull_request:
    paths:
      - '.github/workflows/**'
      - '.github/actions/**'
      - '.github/CODEOWNERS'

permissions:
  pull-requests: write
  contents: read

jobs:
  guard:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            const author = context.payload.pull_request.user.login;
            const { data } = await github.rest.repos.getCollaboratorPermissionLevel({
              ...context.repo, username: author,
            });

            const allowed = ["admin", "maintain"];
            if (!allowed.includes(data.permission)) {
              await github.rest.pulls.createReview({
                ...context.repo,
                pull_number: context.payload.pull_request.number,
                event: "REQUEST_CHANGES",
                body: `این PR فایل‌های حساس CI را تغییر می‌دهد.
                فقط maintainerها می‌توانند این کار را بکنند (@${author} دسترسی ${data.permission} دارد).`,
              });
            }
```

**نکته:** این جایگزین CODEOWNERS نیست، مکمل است (لایه‌ی دفاعی).

#### مثال ۴: محافظت از environment secrets

**Settings → Environments → production**:

```
Required reviewers: @myorg/release-managers
Wait timer: 5 minutes
Deployment branches: Selected branches → main
```

حالا هر deploy به production نیاز به تأیید انسانی دارد — حتی اگر workflow خودکار باشد.

#### مثال ۵: branch موقت برای release

اگر می‌خواهی یک release branch داشته باشی که فقط hotfix بگیرد:

```
Name: Release branches
Target: release/*
Rules:
  ☑ Require a pull request
      Required approvals: 2
  ☑ Require status checks
  ☑ Require signed commits
```

#### مثال ۶: بررسی اینکه چه چیزی محافظت شده

```bash
# قوانین روی main
gh api repos/OWNER/REPO/rules/branches/main --jq '.[].type'

# قوانین روی یک تگ
gh api repos/OWNER/REPO/rules/tags/v1.0.0 --jq '.[].type'

# همه‌ی rulesetها
gh api repos/OWNER/REPO/rulesets --jq '.[] | "\(.name): \(.target) — \(.enforcement)"'
```

### مقایسه و الگوهای مختلف

#### محافظت از تگ: Rulesets در برابر Environment

| | Ruleset (tag) | Environment |
|---|---|---|
| تغییر تگ | ممنوع | ممنوع نیست |
| تأیید deploy | ❌ | ✅ |
| مناسب | نسخه‌های منتشرشده | محیط deploy |

**بهترین:** هر دو. Ruleset تگ را قفل می‌کند، Environment deploy را کنترل می‌کند.

#### CODEOWNERS در برابر workflow سفارشی

| | CODEOWNERS | workflow |
|---|---|---|
| نیاز به کد | ❌ | ✅ |
| اجباری با Ruleset | ✅ | ❌ (فقط کامنت) |
| سرعت | فوری | بعد از اجرا |
| توصیه | ✅ اول | مکمل |

#### چند لایه برای فایل‌های حساس

```
لایه ۱: Ruleset — همه از PR
لایه ۲: CODEOWNERS — security-team approve
لایه ۳: Environment — deploy نیاز به تأیید
لایه ۴: Audit log — بررسی دوره‌ای
```

### اشکال شایع

#### ۱. تگ بدون محافظت

هر کسی با write می‌تواند `git push --force origin v1.0.0`. اگر آن نسخه در npm منتشر شده، فاجعه.

#### ۲. CODEOWNERS بدون Ruleset

فقط یک اطلاع است. باید **Require review from Code Owners** روشن باشد.

#### ۳. `CODEOWNERS` در مسیر اشتباه

سه مسیر معتبر:

```
CODEOWNERS
.github/CODEOWNERS
docs/CODEOWNERS
```

`.github/CODEOWNERS` توصیه‌شده است (با بقیه‌ی تنظیمات).

#### ۴. تیم CODEOWNERS بدون دسترسی write

اگر team دسترسی write نداشته باشد، نمی‌تواند approve کند. `Settings → Access` را چک کن.

#### ۵. محافظت از `main` ولی نه از `release/*`

مهاجم می‌تواند به `release/1.4` push کند و از آنجا deploy شود.

#### ۶. Environment بدون required reviewers

اگر required reviewer نداشته باشد، workflow خودکار deploy می‌کند — که برای production خطرناک است.

#### ۷. فراموش کردن اینکه Ruleset روی tag، creation را هم محدود می‌کند

اگر `Restrict creations` را با bypass نگذاری، **هیچ‌کس** نمی‌تواند تگ بسازد. باید release-managers را bypass کنی.

### نکات مهم (Tips & Tricks)

- **همه‌ی تگ‌های release را با Ruleset قفل کن** — ارزان و حیاتی.
- **`.github/workflows/` را در CODEOWNERS بگذار** — دسترسی به secrets.
- **Environment برای production** با required reviewers.
- **`CODEOWNERS` خودش را هم محافظت کن** — وگرنه کسی می‌تواند خودش را owner کند.
- **Audit log را ماهانه چک کن** (Enterprise) — ببین چه کسی bypass کرده.
- **محافظت را در `SECURITY.md` مستند کن** — تا محقق امنیتی بداند چه چیزی تست کند.

### خلاصه فصل

- تگ‌های release باید قفل شوند: `Restrict updates/deletions`.
- `.github/workflows/` هدف اصلی حمله است — CODEOWNERS + Ruleset.
- **سه لایه:** Ruleset، CODEOWNERS، Environment.
- Environment با required reviewers، deploy را کنترل می‌کند.
- CODEOWNERS بدون Ruleset فقط اطلاع است.

**سؤالات مروری:**
1. چرا تغییر تگ منتشرشده خطرناک است؟
2. چطور از `.github/workflows/` محافظت کنی؟
3. تفاوت Ruleset روی تگ و Environment چیست؟

---

**بعدی:** [`10-Actions-Basics.md`](10-Actions-Basics.md)

---

## منابع و مراجع

- GitHub Docs: About branch protection — https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches
- GitHub Docs: About rulesets — https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets
- GitHub Docs: Available rules for rulesets — https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets
- GitHub Docs: About status checks — https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/collaborating-on-repositories-with-code-quality-features/about-status-checks
- GitHub Docs: Deployment environments — https://docs.github.com/en/actions/deployment/targeting-different-environments
- REST API: Rules — https://docs.github.com/en/rest/repos/rules
