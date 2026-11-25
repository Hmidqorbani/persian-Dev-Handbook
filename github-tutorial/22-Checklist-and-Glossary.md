# بخش ۲۲: چک‌لیست، واژه‌نامه و نقشه راه

**مدت یادگیری**: ۲ تا ۳ ساعت (مرجع)
**سطح سختی**: مرجع
**پیش‌نیاز**: بخش ۰۱ تا ۲۱

---

## فهرست محتوا

- [فصل ۹۹: چک‌لیست تولید برای هر مخزن](#فصل-۹۹)
- [فصل ۱۰۰: واژه‌نامه‌ی Git و GitHub](#فصل-۱۰۰)
- [فصل ۱۰۱: نقشه راه یادگیری و منابع](#فصل-۱۰۱)

---

## فصل ۹۹: چک‌لیست تولید برای هر مخزن

### مقدمه

این فصل یک چک‌لیست عملی است: از ساخت مخزن تا اولین release. می‌توانی آن را کپی کنی و در `docs/production-readiness.md` پروژه بگذاری.

**چرا مهم است؟** یک مخزن «تولیدی» با یک مخزن «تمرینی» تفاوتش در ۳۰ مورد کوچک است. چک‌لیست مطمئن می‌شود هیچ‌کدام فراموش نشوند.

**کاربرد عملی:** برای هر مخزن جدید، از بالا به پایین.

### مفاهیم اساسی

#### شش دسته

```
۱. پایه (repo، README، LICENSE)
۲. همکاری (CONTRIBUTING، templates، CODEOWNERS)
۳. CI/CD (tests، build، deploy)
۴. امنیت (alerts، rulesets، pinning)
۵. کیفیت (lint، hooks، docs)
۶. انتشار (release، SBOM، changelog)
```

### مثال‌های عملی

#### چک‌لیست ۱ — پایه

```
□ نام مخزن گویا و kebab-case
□ توضیح (description) یک‌خطی
□ topics (۵-۸)
□ README با: یک جمله، نصب، مثال، لینک به docs
□ LICENSE (MIT / Apache 2.0 / ...)
□ .gitignore کامل (زبان + IDE + OS + build)
□ .gitattributes (`* text=auto eol=lf`، `lock -diff`)
□ .editorconfig
□ .nvmrc یا معادل (نسخه‌ی زبان)
□ Social preview (۱۲۸۰×۶۴۰)
□ visibility درست (public/private)
```

#### چک‌لیست ۲ — همکاری

```
□ CONTRIBUTING.md (شاخه، commit، PR، تست)
□ CODE_OF_CONDUCT.md
□ .github/PULL_REQUEST_TEMPLATE.md (حداکثر ۱۵ خط)
□ .github/ISSUE_TEMPLATE/bug_report.yml
□ .github/ISSUE_TEMPLATE/feature_request.yml
□ .github/ISSUE_TEMPLATE/config.yml (contact_links)
□ .github/CODEOWNERS (شامل خودش و workflows)
□ برچسب‌ها (نوع، اولویت، وضعیت، دامنه)
□ milestone برای نسخه‌ی جاری
□ استراتژی شاخه مکتوب
□ استراتژی merge انتخاب‌شده (فقط یکی)
□ delete_branch_on_merge=true
```

#### چک‌لیست ۳ — CI/CD

```
□ ci.yml: lint، typecheck، test، build
□ ci-success job (برای required check پایدار)
□ cache (setup-node cache / actions/cache)
□ matrix فقط جایی که لازم است
□ services برای دیتابیس
□ TZ=UTC در تست
□ concurrency (cancel-in-progress روی PR)
□ timeout-minutes در هر job
□ artifact برای coverage و گزارش تست
□ e2e فقط روی main یا با label
□ deploy.yml با environment + approval
□ smoke test بعد از deploy
□ rollback مستند
□ merge_group در trigger
```

#### چک‌لیست ۴ — امنیت

```
□ 2FA روی همه‌ی حساب‌های تیم
□ Secret scanning فعال
□ Push protection فعال
□ Dependabot alerts فعال
□ Dependabot security updates فعال
□ dependabot.yml (گروه‌بندی، limit، ignore مستند)
□ automerge فقط برای patch
□ CodeQL (یا Semgrep) روی PR + هفتگی
□ Ruleset روی main:
    □ Require PR
    □ ۱+ approve
    □ Dismiss stale approvals
    □ Require Code Owners review
    □ Require status checks (ci-success)
    □ Require conversation resolution
    □ Require linear history
    □ Block force push
    □ Restrict deletion
    □ enforce_admins
□ Ruleset روی تگ v*.*.* (بدون update/delete)
□ Actions: default_workflow_permissions = read
□ Actions: can_approve_pull_request_reviews = false
□ Actions: allow-list
□ Actions: SHA pinning (یا sha_pinning_required)
□ zizmor در CI
□ gitleaks در pre-commit
□ SECURITY.md + private vulnerability reporting
□ هیچ secret در تاریخ (چک با gitleaks)
```

#### چک‌لیست ۵ — کیفیت

```
□ linter + formatter (ESLint/Prettier، Ruff، gofmt)
□ typecheck (tsc --noEmit، mypy)
□ hook (Husky/Lefthook/pre-commit) با lint-staged
□ commitlint (Conventional Commits)
□ coverage گزارش (بدون هدف اجباری سخت‌گیرانه)
□ docs/architecture.md (با Mermaid)
□ docs/api.md
□ docs/operations.md (RUNBOOK)
□ copilot-instructions.md / AGENTS.md
□ devcontainer.json (تست‌شده)
```

#### چک‌لیست ۶ — انتشار

```
□ tag annotated (نه lightweight)
□ release با notes (--generate-notes یا .github/release.yml)
□ SBOM در release
□ attestation برای artifact
□ CHANGELOG (خودکار یا دستی)
□ نسخه‌ی package.json با tag می‌خواند
□ smoke test نصب پکیج/image
□ rollback برنامه‌ریزی‌شده
```

#### اسکریپت بررسی خودکار

`scripts/readiness.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

REPO="${1:?usage: $0 owner/repo}"
FAIL=0
PASS=0

check() {
  local name="$1" result="$2"
  if [ "$result" = "ok" ]; then
    echo "✅ $name"; PASS=$((PASS+1))
  else
    echo "❌ $name"; FAIL=$((FAIL+1))
  fi
}

has_file() {
  gh api "repos/$REPO/contents/$1" -i 2>/dev/null | head -1 | grep -q 200 && echo ok || echo no
}

echo "=== فایل‌های ضروری ==="
for f in README.md LICENSE CONTRIBUTING.md SECURITY.md .gitignore .editorconfig; do
  check "$f" "$(has_file "$f")"
done

echo
echo "=== امنیت ==="
P=$(gh api "repos/$REPO" --jq '.security_and_analysis.secret_scanning_push_protection.status // "disabled"')
check "Push protection" "$([ "$P" = "enabled" ] && echo ok || echo no)"

W=$(gh api "repos/$REPO/actions/permissions/workflow" --jq '.default_workflow_permissions')
check "Actions default = read" "$([ "$W" = "read" ] && echo ok || echo no)"

R=$(gh api "repos/$REPO/rulesets" --jq '[.[] | select(.target=="branch")] | length')
check "Branch ruleset" "$([ "$R" -gt 0 ] && echo ok || echo no)"

T=$(gh api "repos/$REPO/rulesets" --jq '[.[] | select(.target=="tag")] | length')
check "Tag ruleset" "$([ "$T" -gt 0 ] && echo ok || echo no)"

D=$(gh api "repos/$REPO/dependabot/alerts?state=open" --jq 'length')
check "بدون Dependabot alert باز" "$([ "$D" = "0" ] && echo ok || echo no)"

echo
echo "=== CI ==="
S=$(gh run list --repo "$REPO" --branch main --limit 1 --json conclusion --jq '.[0].conclusion // "none"')
check "آخرین CI روی main سبز" "$([ "$S" = "success" ] && echo ok || echo no)"

echo
echo "=== تنظیمات ==="
M=$(gh api "repos/$REPO" --jq '.allow_merge_commit, .allow_squash_merge, .allow_rebase_merge' | grep -c true || true)
check "فقط یک روش merge" "$([ "$M" = "1" ] && echo ok || echo no)"

B=$(gh api "repos/$REPO" --jq '.delete_branch_on_merge')
check "delete_branch_on_merge" "$([ "$B" = "true" ] && echo ok || echo no)"

echo
echo "─────────────────────"
echo "✅ $PASS | ❌ $FAIL"
exit "$([ "$FAIL" -eq 0 ] && echo 0 || echo 1)"
```

#### چک‌لیست روزانه (برای توسعه‌دهنده)

```
□ git fetch
□ git status -sb (چقدر عقب/جلو)
□ PRهای review-requested
□ Issueهای assignee@me
□ CI شکست‌خورده
□ قبل از commit: git diff --staged
□ قبل از push: git log origin/branch..branch
```

```bash
# همه در یک دستور
gh pr list --review-requested @me && \
gh issue list --assignee @me && \
gh run list --status failure --limit 3
```

#### چک‌لیست هفتگی (برای نگه‌دارنده)

```
□ Dependabot PRها بررسی و merge
□ alertهای جدید امنیتی
□ Issueهای بدون assignee
□ Issueهای قدیمی (stale)
□ زمان CI (روند)
□ هزینه‌ی Actions
□ branchهای مرده
□ bypassهای ruleset (audit)
```

#### چک‌لیست قبل از release

```
□ main سبز
□ همه‌ی تست‌ها پاس
□ CHANGELOG به‌روز
□ نسخه در package.json
□ migrationها تست‌شده روی staging
□ RUNBOOK به‌روز
□ rollback تست‌شده
□ اطلاع به تیم
```

### مقایسه و الگوهای مختلف

#### چک‌لیست کامل در برابر مینیمال

| | کامل | مینیمال |
|---|---|---|
| زمان setup | ۲-۳ ساعت | ۲۰ دقیقه |
| مناسب | محصول تولیدی | پروژه‌ی شخصی |
| نگهداری | بیشتر | کمتر |

**توصیه:** برای پروژه‌ی جدید، مینیمال شروع کن (README، LICENSE، .gitignore، CI) و تدریجی اضافه کن.

#### اسکریپت در برابر سند

| | اسکریپت | سند |
|---|---|---|
| خودکار | ✅ | ❌ |
| فراموش‌شدنی | ❌ | ✅ |
| انعطاف | کم | ✅ |

**بهترین:** سند برای توضیح، اسکریپت برای بررسی.

### اشکال شایع

#### ۱. چک‌لیست بدون اجرا

سند نوشته می‌شود و کسی نمی‌خواند. اسکریپت بگذار.

#### ۲. همه‌چیز از روز اول

پروژه‌ی ۲۰ خطی با ۱۵ workflow. اضافه‌کاری. تدریجی.

#### ۳. فراموش کردن `delete_branch_on_merge`

بعد از ۱۰۰ PR، ۱۰۰ branch مرده.

#### ۴. Ruleset بدون `non_fast_forward`

force push تاریخ را بازنویسی می‌کند و همه‌ی محافظت‌ها را دور می‌زند.

#### ۵. CODEOWNERS بدون Ruleset

فقط اطلاع است، نه اجبار.

#### ۶. `@v4` به‌جای SHA

زنجیره‌ی تأمین.

#### ۷. SECURITY.md بدون کانال واقعی

ایمیل وجود ندارد. Private vulnerability reporting را روشن کن.

### نکات مهم (Tips & Tricks)

- **اسکریپت بررسی** — ماهانه اجرا کن.
- **تدریجی اضافه کن** — نه همه از روز اول.
- **`non_fast_forward`** ارزان‌ترین محافظت.
- **`delete_branch_on_merge`** یک تیک.
- **چک‌لیست روزانه** در یک alias.
- **قبل از release، RUNBOOK را بخوان** — نه در بحران.

### خلاصه فصل

- شش دسته: پایه، همکاری، CI/CD، امنیت، کیفیت، انتشار.
- اسکریپت بررسی خودکار بساز.
- تدریجی اضافه کن، نه یک‌جا.
- `non_fast_forward` و `delete_branch_on_merge` فراموش نشوند.
- چک‌لیست روزانه/هفتگی/قبل-release.

**سؤالات مروری:**
1. چرا `non_fast_forward` مهم‌ترین قاعده است؟
2. چرا چک‌لیست سند کافی نیست؟
3. چه چیزی را باید تدریجی اضافه کرد؟

---

## فصل ۱۰۰: واژه‌نامه‌ی Git و GitHub

### مقدمه

این فصل مرجع سریع اصطلاحات است. هر جا در کتابچه اصطلاحی دیدی و مطمئن نبودی، اینجا.

### مفاهیم اساسی

#### Git — پایه

| اصطلاح | معنی |
|---|---|
| **repository (repo)** | مخزن؛ پوشه‌ای با `.git` که تاریخ را نگه می‌دارد |
| **commit** | snapshot از کل درخت فایل‌ها + متادیتا + والدها |
| **hash / SHA** | شناسه‌ی یکتای شیء (۴۰ کاراکتر hex) |
| **blob** | محتوای یک فایل در Git |
| **tree** | یک پوشه در Git (فهرست blob/tree) |
| **ref** | اشاره‌گر به یک commit (branch، tag، HEAD) |
| **HEAD** | اشاره‌گر به جایی که الان هستی |
| **detached HEAD** | HEAD مستقیم به commit (نه branch) |
| **branch** | اشاره‌گر متحرک به یک commit (۴۱ بایت) |
| **tag** | برچسب ثابت روی یک commit |
| **annotated tag** | tag با متادیتا و امضا |
| **lightweight tag** | tag بدون متادیتا |
| **index / staging area** | فایل `index`؛ سبد خرید commit بعدی |
| **working directory** | فایل‌های واقعی روی دیسک |
| **tracked / untracked** | ردیابی‌شده / نشده |
| **origin** | نام قراردادی remote اصلی |
| **upstream** | branch مرجع (برای pull/push) یا مخزن اصلی (در fork) |
| **remote-tracking branch** | آینه‌ی محلی از branch روی سرور (`origin/main`) |
| **fast-forward** | merge بدون commit جدید |
| **merge base** | نزدیک‌ترین commit مشترک دو branch |
| **three-way merge** | merge با commit جدید (دو والد) |
| **octopus merge** | merge بیش از دو branch |
| **conflict** | تعارض؛ هر دو طرف یک خط را متفاوت عوض کرده‌اند |
| **rebase** | بازسازی commitها روی base جدید (هش عوض می‌شود) |
| **interactive rebase** | `rebase -i` برای squash/reword/drop/edit |
| **squash** | چند commit → یکی |
| **fixup** | commit که با `--autosquash` به هدفش می‌چسبد |
| **cherry-pick** | کپی یک commit به branch دیگر |
| **stash** | کنار گذاشتن تغییرات uncommitted |
| **reflog** | تاریخ حرکت HEAD (۹۰ روز) |
| **ORIG_HEAD** | موقعیت قبل از merge/rebase/reset |
| **bisect** | جست‌وجوی دودویی برای commit مقصر |
| **worktree** | پوشه‌ی دوم از همان مخزن با branch دیگر |
| **submodule** | اشاره‌گر به commit یک مخزن دیگر |
| **subtree** | کپی محتوای مخزن دیگر در این مخزن |
| **LFS** | Large File Storage؛ فایل بزرگ بیرون مخزن |
| **hook** | اسکریپت در نقاط مشخص Git |
| **bundle** | فایل پشتیبان کامل مخزن |

#### Git — عملیات

| اصطلاح | معنی |
|---|---|
| **checkout** | رفتن به branch/commit (قدیمی) |
| **switch** | عوض کردن branch (مدرن) |
| **restore** | بازگرداندن فایل (مدرن) |
| **reset --soft** | HEAD را جابه‌جا کن، staging/working دست‌نخورده |
| **reset --mixed** | + staging ریست |
| **reset --hard** | + working ریست (خطرناک) |
| **revert** | commit جدید که تغییرات قبلی را خنثی می‌کند |
| **amend** | اصلاح آخرین commit |
| **force push** | overwrite تاریخ remote (خطرناک) |
| **force-with-lease** | force با قفل ایمنی |
| **prune** | پاک کردن refهای مرده |
| **gc** | garbage collection (فشرده‌سازی) |
| **fsck** | بررسی سلامت مخزن |

#### GitHub — همکاری

| اصطلاح | معنی |
|---|---|
| **fork** | کپی سروری مخزن در حساب تو |
| **Pull Request (PR)** | پیشنهاد merge با فضای بحث |
| **draft PR** | PR نیمه‌کاره |
| **review** | بازبینی؛ approve / request changes / comment |
| **suggested change** | پیشنهاد کد قابل اعمال با یک کلیک |
| **CODEOWNERS** | تعیین reviewer بر اساس مسیر |
| **Issue** | کار قابل بستن |
| **Discussion** | گفت‌وگوی بلندمدت |
| **milestone** | نسخه/بازه با تاریخ |
| **label** | برچسب دسته‌بندی |
| **Projects** | تخته/جدول/roadmap |
| **merge queue** | صف merge با تست روی نتیجه‌ی قبلی |
| **stacked PR** | PRهای زنجیره‌ای |
| **squash merge** | یک commit از کل PR |
| **merge commit** | commit با دو والد |
| **rebase merge** | commitها بازنویسی و خطی |
| **auto-merge** | merge خودکار وقتی شرایط برآورده شد |

#### GitHub — حاکمیت

| اصطلاح | معنی |
|---|---|
| **branch protection** | محافظت کلاسیک از branch |
| **ruleset** | نسل جدید محافظت (branch + tag، لایه‌بندی) |
| **required check** | check اجباری برای merge |
| **status check** | نتیجه‌ی CI/سرویس خارجی |
| **bypass actor** | کسی که می‌تواند ruleset را دور بزند |
| **Evaluate mode** | ruleset فقط گزارش می‌دهد |
| **environment** | هدف deploy با approval و secret جدا |
| **deployment protection rule** | قاعده‌ی سفارشی قبل از deploy |
| **organization** | حساب سازمانی (مالک مخزن) |
| **team** | گروه کاربران با دسترسی مشترک |
| **role** | read / triage / write / maintain / admin |

#### GitHub — Actions

| اصطلاح | معنی |
|---|---|
| **workflow** | فرآیند خودکار (`.github/workflows/*.yml`) |
| **event / trigger** | چیزی که workflow را اجرا می‌کند (`on:`) |
| **job** | مجموعه‌ی stepها روی یک runner |
| **step** | یک کار (`run:` یا `uses:`) |
| **action** | واحد قابل استفاده‌ی مجدد |
| **runner** | ماشین اجرا (hosted یا self-hosted) |
| **matrix** | اجرای job با چند مقدار |
| **artifact** | فایل خروجی job |
| **cache** | داده‌ی تکراری برای سرعت |
| **composite action** | مجموعه‌ی stepها در یک action |
| **reusable workflow** | workflow قابل فراخوانی (`workflow_call`) |
| **GITHUB_TOKEN** | توکن خودکار هر job |
| **OIDC** | احراز هویت بدون کلید به ابر |
| **environment secret** | secret مخصوص یک environment |
| **concurrency** | کنترل اجرای هم‌زمان |
| **merge_group** | trigger برای merge queue |
| **self-hosted runner** | runner روی زیرساخت خودت |
| **ephemeral runner** | runner که بعد از یک job حذف می‌شود |
| **SHA pinning** | قفل کردن action به commit |
| **pull_request_target** | trigger خطرناک (دسترسی secrets) |
| **script injection** | تزریق کد از طریق `${{ }}` در `run:` |

#### GitHub — امنیت

| اصطلاح | معنی |
|---|---|
| **Dependabot** | به‌روزرسانی خودکار وابستگی |
| **security update** | PR برای رفع آسیب‌پذیری |
| **version update** | PR برای به‌روزرسانی معمولی |
| **CodeQL** | موتور تحلیل ایستای GitHub |
| **code scanning** | تحلیل ایستا برای آسیب‌پذیری |
| **SARIF** | فرمت استاندارد نتایج تحلیل |
| **secret scanning** | پیدا کردن کلید در مخزن |
| **push protection** | جلوگیری از push کلید |
| **validity check** | آیا کلید هنوز معتبر است |
| **security advisory** | اعلام هماهنگ آسیب‌پذیری |
| **CVE** | شناسه‌ی بین‌المللی آسیب‌پذیری |
| **GHSA** | شناسه‌ی GitHub برای advisory |
| **CVSS** | امتیاز شدت |
| **CWE** | دسته‌بندی ضعف |
| **SBOM** | فهرست اجزای نرم‌افزار |
| **SPDX / CycloneDX** | استانداردهای SBOM |
| **attestation** | گواهی امضاشده‌ی منشأ |
| **SLSA** | استاندارد سطوح زنجیره‌ی تأمین |
| **provenance** | منشأ build |

#### GitHub — اکوسیستم

| اصطلاح | معنی |
|---|---|
| **Copilot** | دستیار AI |
| **completion** | تکمیل کد در ادیتور |
| **Ask / Edit / Agent** | حالت‌های Copilot |
| **Cloud Agent** | اجرای کار در ابر GitHub |
| **MCP** | Model Context Protocol |
| **custom instructions** | قواعد پروژه برای Copilot |
| **Pages** | میزبانی سایت استاتیک |
| **Packages** | رجیستری پکیج |
| **GHCR** | GitHub Container Registry |
| **Codespaces** | محیط توسعه‌ی ابری |
| **devcontainer** | تعریف محیط توسعه به‌صورت کد |
| **prebuild** | image آماده برای Codespaces |
| **Gist** | قطعه‌کد اشتراکی |
| **Wiki** | مستندات سریع در مخزن |
| **Sponsors** | حمایت مالی |
| **gh** | GitHub CLI |

### مثال‌های عملی

#### تمرین: اصطلاح را پیدا کن

```
۱. «commit با دو والد» → ؟
۲. «اشاره‌گر ۴۱ بایتی» → ؟
۳. «تاریخ حرکت HEAD» → ؟
۴. «جست‌وجوی دودویی» → ؟
۵. «ماشین اجرای workflow» → ؟
۶. «گواهی امضاشده‌ی منشأ artifact» → ؟
۷. «صف merge با تست روی نتیجه‌ی قبلی» → ؟
۸. «کلید بدون ذخیره در secret» → ؟
```

پاسخ‌ها: ۱. merge commit ۲. branch ۳. reflog ۴. bisect ۵. runner ۶. attestation ۷. merge queue ۸. OIDC

### مقایسه و الگوهای مختلف

#### اصطلاحات اشتباه رایج

| ✗ اشتباه | ✓ درست |
|---|---|
| «Git را روی GitHub آپلود کردم» | «به GitHub push کردم» |
| «branch را ذخیره کردم» | «commit زدم / push کردم» |
| «GitHub merge کرد» | «PR merge شد» |
| «fork کردم یعنی clone کردم» | fork سروری است، clone محلی |
| «rebase همان merge است» | rebase تاریخ را بازنویسی می‌کند |
| «reset همان revert است» | reset بازنویسی، revert commit جدید |

### اشکال شایع

#### ۱. ترجمه‌ی اصطلاحات به فارسی در مکالمه‌ی فنی

```
✗ «شاخه‌ی من را با شاخه‌ی تو ادغام کن»
✓ «branch من را با branch تو merge کن»
```

در گفت‌وگوی فنی، اصطلاح انگلیسی را نگه دار و در پرانتز معنی بده (در اولین استفاده). ترجمه‌ی کامل، ارتباط را کند می‌کند — چون هر کس ترجمه‌ی خودش را می‌سازد.

#### ۲. اشتباه گرفتن سطح ابزار

| جمله | مشکل |
|---|---|
| «GitHub را ریست کردم» | GitHub ریست نمی‌شود؛ Git ریست می‌شود |
| «Git را fork کردم» | fork عملیات GitHub است |
| «git push به staging» | push به remote؛ staging محیط deploy است |

#### ۳. استفاده از «آپلود» برای هر چیز

```
✗ «فایل را آپلود کردم به مخزن»
✓ «commit کردم و push کردم»
```

«آپلود» فقط برای artifact، release asset، یا فایل در UI است.

#### ۴. «main شاخه‌ی اصلی Git است»

`main` فقط یک **نام قراردادی** است. Git هیچ branchی را مقدس نمی‌داند. «اصلی بودن» یک قرارداد تیمی است.

#### ۵. «PR یعنی merge»

PR یک **درخواست** است. تا وقتی کسی merge نکند، هیچ تغییری در `main` نیست.

#### ۶. «commit یعنی ذخیره»

commit در **مخزن محلی** ذخیره می‌کند، نه روی سرور. تا push نکنی، فقط روی سیستم توست.

### نکات مهم (Tips & Tricks)

- **اصطلاح انگلیسی را یاد بگیر، معنی فارسی را هم بدان.** `rebase` را «بازچینش» بگو اگر لازم شد، ولی در مکالمه `rebase` بگو.
- **واژه‌نامه را در `CONTRIBUTING.md` تیم بگذار** — تا همه یک زبان حرف بزنند.
- **وقتی شک داری، `git help <term>`** — مستندات تعریف دقیق می‌دهد.
- **در Issue و PR، اصطلاح دقیق بنویس.** «شاخه» در برابر «branch» می‌تواند سوءتفاهم بسازد.
- **اشتباهات اصطلاحی را تصحیح کن، ولی نه با تحقیر.** خیلی‌ها این‌ها را از تویتریال‌های نادرست یاد گرفته‌اند.

### خلاصه فصل

- مرجع سریع ۱۵۰+ اصطلاح.
- Git: پایه، اشیاء، عملیات.
- GitHub: همکاری، حاکمیت، Actions، امنیت، اکوسیستم.
- اشتباهات رایج اصطلاحی را بشناس.

**سؤالات مروری:**
1. تفاوت ref و branch چیست؟
2. تفاوت fork و clone چیست؟
3. تفاوت attestation و SBOM چیست؟

---

## فصل ۱۰۱: نقشه راه یادگیری و منابع

### مقدمه

این کتابچه پایه است، نه پایان. این فصل می‌گوید بعد از این چه بخوانی و چطور تمرین کنی.

**چرا مهم است؟** بدون نقشه، آدم در تویتریال‌های پراکنده گیر می‌کند. با نقشه، هر هفته یک قدم مشخص.

**کاربرد عملی:** یک برنامه‌ی ۱۲ هفته‌ای.

### مفاهیم اساسی

#### سه سطح بعدی

| سطح | چه چیزی | چرا |
|---|---|---|
| **۱. عمق Git** | internals، packfile، index format | دیباگ موقعیت‌های عجیب |
| **۲. پلتفرم** | GitHub Apps، webhooks، REST/GraphQL | اتوماسیون و یکپارچگی |
| **۳. مهندسی** | CI/CD پیشرفته، IaC، observability | تولید واقعی |

#### برنامه‌ی ۱۲ هفته‌ای

| هفته | تمرکز | تمرین |
|---|---|---|
| ۱ | Git پایه | ۲۰ commit در یک مخزن تمرینی، بدون GUI |
| ۲ | branch/merge/rebase | ۱۰ conflict حل کن، ۵ rebase |
| ۳ | بازنویسی تاریخ | reflog، reset، revert، bisect |
| ۴ | PR و review | ۵ PR در یک پروژه‌ی متن‌باز |
| ۵ | Actions پایه | CI برای یک پروژه‌ی خودت |
| ۶ | Actions پیشرفته | cache، matrix، reusable |
| ۷ | امنیت Actions | permissions، OIDC، SHA pinning |
| ۸ | امنیت مخزن | Dependabot، CodeQL، rulesets |
| ۹ | release و packages | semantic-release، GHCR |
| ۱۰ | Copilot | instructions، MCP، agent |
| ۱۱ | Pages/Codespaces | سایت مستندات + devcontainer |
| ۱۲ | پروژه‌ی کامل | مخزن تولیدی از صفر |

### مثال‌های عملی

#### تمرین ۱: بازسازی Git از صفر (درک internals)

```bash
mkdir git-internals && cd git-internals
git init

# blob بساز
echo "سلام" | git hash-object -w --stdin
# e7a3f9c1b2d4...

# tree بساز
git update-index --add --cacheinfo 100644,e7a3f9c1b2d4,hello.txt
TREE=$(git write-tree)
echo "tree: $TREE"

# commit بساز
COMMIT=$(git commit-tree "$TREE" -m "first")
echo "commit: $COMMIT"

# ref بساز
git update-ref refs/heads/main "$COMMIT"

git log --oneline
git cat-file -p HEAD
```

**چرا؟** بعد از این، Git جادو نیست — یک پایگاه داده‌ی ساده است.

#### تمرین ۲: حل ۱۰ conflict

```bash
# یک مخزن با دو branch که همان فایل را عوض می‌کنند
for i in $(seq 1 10); do
  git switch main
  echo "main-$i" >> "file$i.txt"
  git add . && git commit -m "main $i"

  git switch -c "feature-$i" main~1
  echo "feature-$i" >> "file$i.txt"
  git add . && git commit -m "feature $i"

  git switch main
  git merge "feature-$i" || true
  # حل کن
  git add "file$i.txt" && git commit --no-edit
done
```

#### تمرین ۳: PR در متن‌باز

```bash
# ۱) یک پروژه با `good first issue` پیدا کن
gh search issues --label "good first issue" --state open --limit 20

# ۲) fork
gh repo fork OWNER/REPO --clone
cd REPO

# ۳) branch
git switch -c fix/typo-in-docs

# ۴) تغییر، commit، push
# ۵) PR
gh pr create --repo OWNER/REPO --title "docs: ..." --body "..."
```

**نکته:** اول Issue باز کن و بپرس «آیا این تغییر مطلوب است؟» — قبل از ساعت‌ها کار.

#### تمرین ۴: CI برای پروژه‌ی خودت

```yaml
# حداقل: lint + test
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
      - run: echo "جای دستورهای خودت"
```

بعد تدریجی: cache، matrix، artifact، deploy.

#### تمرین ۵: بازسازی یک مخزن تولیدی

یک پروژه‌ی قدیمی بردار و:

```
۱. .gitignore کامل
۲. README + LICENSE + CONTRIBUTING + SECURITY
۳. CI
۴. Dependabot + CodeQL + push protection
۵. Ruleset
۶. release با tag
```

#### تمرین ۶: خواندن کد یک پروژه‌ی بزرگ

```bash
# یک پروژه‌ی محبوب clone کن
gh repo clone fastify/fastify
cd fastify

# تاریخ را کاوش کن
git log --oneline -20
git log --author="Matteo" --oneline -10
git log -S"contentTypeParser" --oneline
git log --graph --oneline --all -30
git shortlog -sn --all | head

# یک فایل را دنبال کن
git log --follow -p lib/route.js | head -100

# چه کسی این خط را نوشت
git blame -L 1,20 lib/route.js
```

**چرا؟** دیدن یک مخزن واقعی با ۱۰۰۰+ commit، درک بهتری می‌دهد از اینکه Git در مقیاس چطور کار می‌کند.

#### منابع به ترتیب اولویت

**۱. رسمی (همیشه اول):**

| منبع | برای |
|---|---|
| https://git-scm.com/book/fa/v2 | Pro Git (رایگان، فارسی) |
| https://docs.github.com | مستندات GitHub |
| https://github.blog/changelog/ | تغییرات جدید |
| https://cli.github.com/manual/ | `gh` |

**۲. تمرین تعاملی:**

| منبع | برای |
|---|---|
| https://skills.github.com | آموزش تعاملی GitHub |
| https://learngitbranching.js.org/?locale=fa | تمرین بصری branch |
| https://git-school.github.io/visualizing-git/ | تجسم Git |

**۳. عمیق‌تر:**

| منبع | برای |
|---|---|
| "Building Git" (James Coglan) | internals |
| https://git-scm.com/docs | مرجع دستورها |
| https://github.com/git/git | سورس Git |
| https://graphite.dev/guides | راهنماهای مدرن |

**۴. امنیت:**

| منبع | برای |
|---|---|
| https://owasp.org/www-project-top-ten/ | OWASP Top 10 |
| https://slsa.dev/ | زنجیره‌ی تأمین |
| https://securitylab.github.com/ | تحقیقات امنیتی GitHub |
| https://github.com/woodruffw/zizmor | امنیت Actions |

**۵. برای ادامه‌ی مسیر شغلی:**

| مسیر | چه بخوانی |
|---|---|
| **DevOps** | Terraform، Kubernetes، observability |
| **Platform Engineering** | Internal Developer Platform، Backstage |
| **Security** | SAST/DAST، threat modeling، supply chain |
| **Open Source Maintainer** | governance، community، licensing |

#### پروژه‌های پیشنهادی برای تمرین

| پروژه | چه تمرین می‌کنی |
|---|---|
| **مخزن template سازمانی** | `.github` repo با reusable workflows |
| **CLI با release خودکار** | semantic-release + GHCR + SBOM |
| **سایت مستندات** | Pages + Actions + custom domain |
| **پروژه‌ی چندزبانه** | devcontainer + matrix + cache |
| **کتابخانه‌ی داخلی** | GitHub Packages + provenance |
| **پروژه با تیم** | Projects + rulesets + CODEOWNERS |

### مقایسه و الگوهای مختلف

#### یادگیری با خواندن در برابر ساختن

| | خواندن | ساختن |
|---|---|---|
| سرعت | ✅ | کند |
| عمق | ❌ | ✅ |
| ماندگاری | کم | ✅ |

**قاعده:** ۲۰٪ خواندن، ۸۰٪ ساختن.

#### تمرین روی مخزن خودت در برابر متن‌باز

| | خودت | متن‌باز |
|---|---|---|
| آزادی | ✅ | محدود |
| بازخورد | ❌ | ✅ |
| یادگیری review | ❌ | ✅ |
| ترس از خرابی | کم | ✅ بیشتر (خوب است) |

**توصیه:** هر دو. مخزن خودت برای آزمایش، متن‌باز برای review واقعی.

#### تویتریال در برابر مستندات رسمی

| | تویتریال | مستندات |
|---|---|---|
| سرعت شروع | ✅ | کند |
| دقت | متغیر | ✅ |
| به‌روز بودن | ❌ | ✅ |

**قاعده:** تویتریال برای شروع، مستندات برای تصمیم.

### اشکال شایع

#### ۱. تویتریال‌بازی

۱۰ دوره، صفر پروژه. **یک پروژه بساز.**

#### ۲. ترس از خراب کردن مخزن واقعی

Git تقریباً همیشه قابل بازیابی است (`reflog`). جسور باش.

#### ۳. یاد نگرفتن `reflog`

بدون آن، هر اشتباه فاجعه به نظر می‌رسد. با آن، Git امن است.

#### ۴. استفاده از GUI بدون درک خط فرمان

GUI خوب است، ولی وقتی گیر می‌کند، باید بدانی چه دستوری پشتش است.

#### ۵. نادیده گرفتن امنیت

بعد از اولین نشت secret، یاد می‌گیری. بهتر: قبل.

#### ۶. کپی کورکورانه از workflowها

workflow از پروژه‌ی دیگر را کپی می‌کنی، `permissions` و `pull_request_target` را نمی‌فهمی. **بخوان، بعد کپی.**

#### ۷. نداشتن `AGENTS.md` / `copilot-instructions.md`

Copilot کد ناهماهنگ می‌دهد. یک فایل، کیفیت را چند برابر می‌کند.

### نکات مهم (Tips & Tricks)

- **۸۰٪ ساختن، ۲۰٪ خواندن.**
- **`reflog` را حفظ کن** — Git امن می‌شود.
- **یک پروژه‌ی واقعی بردار** و تولیدی‌اش کن.
- **در متن‌باز PR بده** — review واقعی، بهترین معلم.
- **Changelog را دنبال کن** — GitHub سریع تغییر می‌کند.
- **مستندات را بخوان، نه فقط تویتریال.**

### خلاصه فصل

- سه سطح بعدی: عمق Git، پلتفرم، مهندسی.
- برنامه‌ی ۱۲ هفته‌ای با تمرین.
- ۸۰٪ ساختن، ۲۰٪ خواندن.
- `reflog` را حفظ کن.
- در متن‌باز مشارکت کن.
- Changelog را دنبال کن.

**سؤالات مروری:**
1. چرا ۸۰٪ ساختن بهتر از ۸۰٪ خواندن است؟
2. چرا `reflog` مهم‌ترین دستور برای یادگیری است؟
3. بعد از این کتابچه، سه قدم بعدی چیست؟

---

**پایان کتابچه.**

از [`README.md`](README.md) شروع کن، یا به فهرست کامل در [`SUMMARY.md`](SUMMARY.md) برگرد.

---

## منابع و مراجع

- Pro Git (فارسی، رایگان) — https://git-scm.com/book/fa/v2
- GitHub Docs — https://docs.github.com
- GitHub Skills — https://skills.github.com
- Learn Git Branching (فارسی) — https://learngitbranching.js.org/?locale=fa
- Visualizing Git — https://git-school.github.io/visualizing-git/
- GitHub Changelog — https://github.blog/changelog/
- GitHub CLI Manual — https://cli.github.com/manual/
- OWASP Top 10 — https://owasp.org/www-project-top-ten/
- SLSA — https://slsa.dev/
- GitHub Security Lab — https://securitylab.github.com/
- "Building Git" — https://shop.jcoglan.com/building-git/
- Keep a Changelog — https://keepachangelog.com/
- Conventional Commits — https://www.conventionalcommits.org
