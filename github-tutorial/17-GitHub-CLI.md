# بخش ۱۷: GitHub CLI (`gh`)

**مدت یادگیری**: ۳ تا ۴ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۰۴ (PR)، بخش ۱۰ (Actions)

---

## فهرست محتوا

- [فصل ۷۶: نصب و احراز هویت `gh`](#فصل-۷۶)
- [فصل ۷۷: `gh repo`، `gh pr`، `gh issue` — کار روزمره](#فصل-۷۷)
- [فصل ۷۸: `gh run` و `gh workflow` — CI از ترمینال](#فصل-۷۸)
- [فصل ۷۹: `gh api`، `gh alias` و extensionها](#فصل-۷۹)

---

## فصل ۷۶: نصب و احراز هویت `gh`

### مقدمه

`gh` ابزار رسمی خط فرمان GitHub است. هر کاری که در UI می‌کنی، می‌توانی در ترمینال بکنی — سریع‌تر، قابل اسکریپت، و بدون قطع جریان فکری.

**چرا مهم است؟** ۸۰٪ کارهای روزمره (PR، Issue، CI، release) در `gh` سریع‌تر است. و برای اسکریپت/اتوماسیون، تنها راه.

**کاربرد عملی:** بعد از این فصل، احتمالاً دیگر برای PR به مرورگر نمی‌روی.

### مفاهیم اساسی

#### نصب

| سیستم | دستور |
|---|---|
| macOS | `brew install gh` |
| Windows | `winget install GitHub.cli` |
| Debian/Ubuntu | `sudo apt install gh` |
| Fedora | `sudo dnf install gh` |
| Arch | `sudo pacman -S github-cli` |
| از سورس | `go install github.com/cli/cli/v2/cmd/gh@latest` |

#### احراز هویت

```bash
gh auth login
```

سه انتخاب مهم:

1. **GitHub.com** یا **Enterprise Server**
2. **HTTPS** یا **SSH**
3. **Login with a web browser** یا **Paste an authentication token**

#### ذخیره‌ی credential

`gh` توکن را در یکی از این‌ها ذخیره می‌کند:

| سیستم | محل |
|---|---|
| macOS | Keychain |
| Windows | Credential Manager |
| Linux | `~/.config/gh/hosts.yml` (اگر keyring نباشد) |

**هشدار:** روی Linux بدون keyring، توکن در فایل متنی است. مجوز فایل را چک کن.

#### چند حساب

```bash
gh auth login --hostname github.com --git-protocol ssh
gh auth login --hostname github.com --git-protocol ssh --web

gh auth status
# github.com
#   ✓ Logged in to github.com account ali (keyring)
#   - Active account: true
# - Logged in to github.com account ali-work (keyring)
#   - Active account: false

gh auth switch
```

### مثال‌های عملی

#### مثال ۱: نصب و راه‌اندازی

```bash
# نصب (macOS)
brew install gh

# بررسی نسخه
gh --version
# gh version 2.63.0 (2026-...)

# ورود
gh auth login
# ? What account do you want to log into? GitHub.com
# ? What is your preferred protocol for Git operations? SSH
# ? Generate a new SSH key to add to your GitHub account? Yes
# ? Enter a passphrase for your new SSH key? (empty for no passphrase)
# ? How would you like to authenticate GitHub CLI? Login with a web browser

# بررسی
gh auth status
```

#### مثال ۲: احراز هویت با token (CI)

```bash
echo "$GITHUB_TOKEN" | gh auth login --with-token
gh auth status
```

**در CI** لازم نیست لاگین کنی — `GH_TOKEN` را در env بگذار:

```yaml
- run: gh pr list
  env:
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

#### مثال ۳: دسترسی‌ها

```bash
gh auth refresh -s write:packages,read:org
```

scopeهای رایج: `repo`، `workflow`، `write:packages`، `read:org`، `admin:public_key`.

#### مثال ۴: چند حساب

```bash
# افزودن حساب دوم
gh auth login --hostname github.com

# جابه‌جایی
gh auth switch --hostname github.com --user ali-work

# یا موقت با env
GH_TOKEN=$(gh auth token --user ali-work) gh pr list
```

#### مثال ۵: تنظیمات

```bash
# ادیتور
gh config set editor "code --wait"

# پروتکل پیش‌فرض
gh config set git_protocol ssh

# مرورگر
gh config set browser firefox

# pager
gh config set pager "less -FRX"

# مشاهده
gh config list
```

#### مثال ۶: استفاده در اسکریپت

```bash
#!/usr/bin/env bash
set -euo pipefail

export GH_TOKEN="${GH_TOKEN:?GH_TOKEN لازم است}"

REPO="myorg/backend"
gh pr list --repo "$REPO" --state open --json number,title \
  --jq '.[] | "\(.number)\t\(.title)"'
```

#### مثال ۷: `gh` در برابر `curl`

| کار | gh | curl |
|---|---|---|
| لیست PR | `gh pr list` | `curl -H "Authorization: Bearer $T" https://api.github.com/repos/.../pulls` |
| ساخت Issue | `gh issue create --title x` | POST با JSON |
| pagination | خودکار | دستی |
| خروجی | جدول/JSON | JSON خام |

**`gh api`** وسط این دو است: قدرت API با راحتی gh.

### مقایسه و الگوهای مختلف

#### `gh` در برابر Git

| | `gh` | `git` |
|---|---|---|
| دامنه | GitHub | Git |
| PR/Issue/Actions | ✅ | ❌ |
| commit/branch | محدود | ✅ |
| نیاز به شبکه | ✅ | ❌ |

**`gh` جایگزین `git` نیست.** کنار هم کار می‌کنند.

#### `GH_TOKEN` در برابر `gh auth login`

| | env | login |
|---|---|---|
| برای CI | ✅ | ❌ |
| ماندگار | ❌ | ✅ |
| چند حساب | ✅ (هر بار) | ✅ (switch) |

#### keyring در برابر فایل

| | keyring | فایل |
|---|---|---|
| امنیت | ✅ | ⚠️ |
| نیاز به نصب | ✅ (gnome-keyring) | ❌ |
| در container | ❌ | ✅ |

### اشکال شایع

#### ۱. `gh: command not found`

نصب نشده یا در PATH نیست. `which gh` و `echo $PATH`.

#### ۲. `gh auth login` در محیط بدون مرورگر

```bash
gh auth login --with-token < token.txt
```

#### ۳. `HTTP 401: Bad credentials`

توکن منقضی یا باطل. `gh auth login` دوباره.

#### ۴. `gh` با حساب اشتباه

```bash
gh auth status
gh auth switch
```

#### ۵. `gh` در container

keyring نیست. `GH_TOKEN` در env بگذار.

#### ۶. مجوز فایل hosts.yml

```bash
chmod 600 ~/.config/gh/hosts.yml
```

#### ۷. `gh` کند است

بعضی دستورها pagination دارند. `--limit` بگذار.

### نکات مهم (Tips & Tricks)

- **`gh auth status`** اولین دستور در هر مشکل احراز هویت.
- **`GH_TOKEN` در CI** — نه `gh auth login`.
- **`gh config set editor "code --wait"`** — برای دستورهایی که ادیتور باز می‌کنند.
- **`gh auth switch`** برای چند حساب.
- **`--json` + `--jq`** برای اسکریپت.

### خلاصه فصل

- `gh` ابزار رسمی CLI، مکمل `git` است.
- `gh auth login` برای setup؛ `GH_TOKEN` برای CI.
- چند حساب با `gh auth switch`.
- `gh config set` برای تنظیمات.
- keyring بهتر از فایل است.

**سؤالات مروری:**
1. چرا `gh` جایگزین `git` نیست؟
2. در CI چطور احراز هویت کنی؟
3. چطور بین دو حساب جابه‌جا شوی؟

---

## فصل ۷۷: `gh repo`، `gh pr`، `gh issue` — کار روزمره

### مقدمه

این سه خانواده، ۹۰٪ استفاده‌ی روزمره را پوشش می‌دهند: ساخت مخزن، مدیریت PR، مدیریت Issue.

**چرا مهم است؟** PR زدن از ترمینال، ۱۰ ثانیه است؛ از مرورگر، ۲ دقیقه. در روز ۵ بار، ۱۰ دقیقه صرفه‌جویی.

**کاربرد عملی:** این فصل یک مرجع دستوری عملی است.

### مفاهیم اساسی

#### خانواده‌ها

| دستور | چه می‌کند |
|---|---|
| `gh repo` | ساخت، clone، fork، view، edit، delete |
| `gh pr` | ساخت، لیست، view، checkout، review، merge |
| `gh issue` | ساخت، لیست، view، close، develop |
| `gh release` | ساخت، لیست، دانلود |
| `gh workflow` / `gh run` | CI (فصل ۷۸) |
| `gh secret` / `gh variable` | secret و متغیر |
| `gh label` | برچسب |
| `gh search` | جست‌وجو |

#### الگوی مشترک

```
gh <resource> <action> [flags]

مثال:
gh pr create --title "..." --body "..."
gh pr list --state open --label bug
gh pr view 42 --json state
gh pr merge 42 --squash --delete-branch
```

#### `--json` + `--jq`

```bash
gh pr list --json number,title,author --jq '.[] | "\(.number) \(.title)"'
```

`--json` فیلدهای موجود را می‌دهد. برای دیدن لیست:

```bash
gh pr view --json 2>&1 | head -20
# یا
gh pr list --json 2>&1 | head
```

### مثال‌های عملی

#### مثال ۱: `gh repo`

```bash
# ساخت
gh repo create my-project --public --clone
gh repo create myorg/backend --private --description "API فروشگاه"

# از پوشه‌ی موجود
gh repo create my-project --source=. --public --push

# از template
gh repo create my-project --template myorg/template --private --clone

# fork
gh repo fork original/project --clone

# view
gh repo view
gh repo view --web
gh repo view myorg/backend --json name,description,stargazerCount

# edit
gh repo edit --description "..." --add-topic api --add-topic python
gh repo edit --enable-wiki --enable-discussions
gh repo edit --enable-squash-merge --enable-merge-commit=false

# لیست
gh repo list myorg --limit 20
gh repo list --fork

# clone
gh repo clone myorg/backend
gh repo clone myorg/backend -- --depth 1

# sync fork
gh repo sync ali/project --source original/project

# archive
gh repo archive myorg/old-project --yes
```

#### مثال ۲: `gh pr` — چرخه‌ی کامل

```bash
# ساخت
gh pr create --fill
gh pr create --title "feat: ..." --body-file pr.md --reviewer maryam --label feature
gh pr create --draft
gh pr create --base develop

# لیست
gh pr list
gh pr list --state all --limit 50
gh pr list --author @me
gh pr list --review-requested @me
gh pr list --search "is:open label:bug"

# view
gh pr view
gh pr view 42
gh pr view 42 --web
gh pr view 42 --json state,mergeable,mergeStateStatus,reviewDecision
gh pr diff 42
gh pr checks 42
gh pr checks 42 --watch

# checkout
gh pr checkout 42
gh pr checkout 42 --branch my-local-name

# review
gh pr review 42 --approve
gh pr review 42 --request-changes --body "کوئری باید پارامتری شود"
gh pr review 42 --comment --body "سؤال: چرا cache نکردی؟"

# merge
gh pr merge 42 --squash --delete-branch
gh pr merge 42 --merge
gh pr merge 42 --rebase
gh pr merge 42 --auto --squash

# edit
gh pr edit 42 --add-reviewer ali --add-label bug
gh pr edit 42 --title "..."

# ready
gh pr ready 42

# close/reopen
gh pr close 42 --comment "در #57 حل شد"
gh pr reopen 42
```

#### مثال ۳: `gh issue`

```bash
# ساخت
gh issue create --title "خطای 500" --body-file bug.md --label bug --assignee @me
gh issue create --title "..." --milestone "v2.4.2" --project "نقشه راه"

# لیست
gh issue list
gh issue list --label bug --state open
gh issue list --assignee @me
gh issue list --search "no:assignee label:bug"
gh issue list --json number,title,labels --jq '.[] | "\(.number) \(.title)"'

# view
gh issue view 42
gh issue view 42 --comments
gh issue view 42 --web

# edit
gh issue edit 42 --add-label "priority:high" --add-assignee maryam
gh issue edit 42 --milestone "v2.4.2"

# branch از Issue
gh issue develop 42 --checkout
gh issue develop 42 --name "fix/42-login-error" --base main

# close/reopen
gh issue close 42 --comment "در 2.4.2 رفع شد"
gh issue close 42 --reason "not planned"
gh issue reopen 42

# transfer
gh issue transfer 42 other/repo
```

#### مثال ۴: `gh release`

```bash
gh release create v1.4.0 --generate-notes
gh release create v1.4.0 dist/*.zip --title "v1.4.0" --notes-file notes.md
gh release create v1.5.0-rc.1 --prerelease
gh release list
gh release view v1.4.0
gh release download v1.4.0 --pattern "*.zip"
gh release delete v1.4.0 --yes --cleanup-tag
```

#### مثال ۵: `gh label` و `gh secret`

```bash
gh label list
gh label create "priority:high" --color d93f0b --description "این هفته"
gh label clone myorg/template

gh secret set API_KEY --body "xxx"
gh secret set PROD_KEY --env production --body "xxx"
gh secret list
gh secret delete OLD_KEY

gh variable set AWS_REGION --body "eu-central-1"
gh variable list
```

#### مثال ۶: `gh search`

```bash
gh search repos "fastapi template" --limit 10
gh search code "TODO" --repo myorg/backend
gh search issues --assignee @me --state open
gh search prs --review-requested @me
gh search commits "fix: login" --repo myorg/backend
```

#### مثال ۷: جریان روزمره‌ی کامل

```bash
#!/usr/bin/env bash
set -euo pipefail

# صبح: چه چیزی روی من است؟
echo "=== PRهایی که باید review کنم ==="
gh pr list --review-requested @me --json number,title,author \
  --jq '.[] | "#\(.number) \(.title) (@\(.author.login))"'

echo "=== Issueهای من ==="
gh issue list --assignee @me --json number,title \
  --jq '.[] | "#\(.number) \(.title)"'

echo "=== CI شکست‌خورده ==="
gh run list --status failure --limit 5 --json databaseId,displayTitle \
  --jq '.[] | "\(.databaseId) \(.displayTitle)"'
```

#### مثال ۸: اسکریپت merge خودکار

```bash
#!/usr/bin/env bash
set -euo pipefail

# PRهایی که approve شده، CI سبز است، و conflict ندارند
gh pr list --state open --json number,reviewDecision,mergeStateStatus \
  --jq '.[] | select(.reviewDecision=="APPROVED" and .mergeStateStatus=="CLEAN") | .number' \
  | while read -r n; do
      echo "merge #$n"
      gh pr merge "$n" --squash --delete-branch
    done
```

**هشدار:** قبل از اتوماسیون merge، مطمئن شو که شرط‌ها دقیق‌اند.

#### مثال ۹: ساخت PR با body از فایل

```bash
cat > /tmp/pr-body.md <<'EOF'
## چه چیزی عوض شد
...

## چطور تست شد
...

Closes #42
EOF

gh pr create --title "feat(profile): ..." --body-file /tmp/pr-body.md
```

### مقایسه و الگوهای مختلف

#### `gh pr create --fill` در برابر `--title/--body`

| | `--fill` | صریح |
|---|---|---|
| سرعت | ✅ | کند |
| کیفیت | متوسط | ✅ |
| مناسب | commit تمیز | PR مهم |

#### `gh pr merge` گزینه‌ها

| گزینه | کار |
|---|---|
| `--squash` | یک commit |
| `--merge` | merge commit |
| `--rebase` | rebase |
| `--auto` | وقتی آماده شد |
| `--delete-branch` | پاک کردن branch |
| `--admin` | bypass ruleset (خطرناک) |

#### `gh issue develop` در برابر branch دستی

| | `gh issue develop` | دستی |
|---|---|---|
| اتصال به Issue | ✅ | ❌ |
| نام‌گذاری | خودکار | دستی |
| نمایش در Issue | ✅ | ❌ |

### اشکال شایع

#### ۱. `gh pr create` بدون commit

```
Error: no commits between main and feature
```

commit نزده‌ای یا push نکرده‌ای.

#### ۲. `--fill` با commit نامناسب

پیام commit «wip» → عنوان PR «wip». صریح بنویس.

#### ۳. `gh pr merge` روی PR با conflict

```
GraphQL: Pull request is not mergeable (mergePullRequest)
```

اول rebase.

#### ۴. `--admin` بدون فکر

```bash
✗ gh pr merge 42 --admin
```

ruleset را دور می‌زند. فقط در اضطراری، و در audit log ثبت می‌شود.

#### ۵. `gh issue develop` که branch نمی‌سازد

اگر Issue در مخزن دیگری است یا دسترسی نداری.

#### ۶. pagination

```bash
✗ gh pr list
✓ gh pr list --limit 200
```

پیش‌فرض ۳۰ است.

#### ۷. `--json` فیلد ناموجود

```
Unknown JSON field: "foo"
```

فیلدها را از مستندات یا با `gh pr view --json` (بدون مقدار) چک کن.

### نکات مهم (Tips & Tricks)

- **`gh pr checks --watch`** — تا پایان CI صبر می‌کند.
- **`gh pr checkout`** — بهترین راه review محلی.
- **`gh issue develop`** — branch را به Issue وصل می‌کند.
- **`--json` + `--jq`** — اسکریپت‌پذیر.
- **`--body-file`** برای بدنه‌ی طولانی.
- **`--limit`** برای pagination.

### خلاصه فصل

- `gh repo`، `gh pr`، `gh issue`، `gh release` — ۹۰٪ کار روزمره.
- `--json` + `--jq` برای اسکریپت.
- `gh pr checkout` برای review محلی.
- `gh issue develop` branch می‌سازد و لینک می‌کند.
- `--limit` را فراموش نکن.

**سؤالات مروری:**
1. چطور یک PR را محلی review کنی؟
2. چطور branch را به Issue وصل کنی؟
3. چرا `--limit` مهم است؟

---

## فصل ۷۸: `gh run` و `gh workflow` — CI از ترمینال

### مقدمه

وقتی CI شکست می‌خورد، رفتن به مرورگر و پیدا کردن لاگ کند است. `gh run` همه‌چیز را در ترمینال می‌دهد — و قابل اسکریپت است.

**چرا مهم است؟** دیباگ CI سریع‌تر، و امکان اتوماسیون (مثلاً «اگر شبانه شکست خورد، Issue بساز»).

**کاربرد عملی:** این فصل یک جعبه‌ابزار دیباگ CI است.

### مفاهیم اساسی

#### تفاوت `workflow` و `run`

| | workflow | run |
|---|---|---|
| چیست | تعریف (فایل YAML) | یک اجرای مشخص |
| مثال | `ci.yml` | «اجرای #۱۲۳ از ci.yml» |
| دستور | `gh workflow` | `gh run` |

#### دستورهای اصلی

```bash
gh workflow list          # workflowها
gh workflow view ci.yml   # جزئیات
gh workflow run ci.yml    # اجرای دستی (workflow_dispatch)
gh workflow enable/disable

gh run list               # اجراها
gh run view <id>          # جزئیات
gh run watch <id>         # زنده
gh run view <id> --log    # لاگ کامل
gh run view <id> --log-failed  # فقط شکست
gh run rerun <id>         # اجرای مجدد
gh run cancel <id>        # لغو
gh run download <id>      # artifact
```

### مثال‌های عملی

#### مثال ۱: بررسی وضعیت CI

```bash
gh run list --limit 10
# STATUS  TITLE                        WORKFLOW  BRANCH  EVENT  ID          ELAPSED  AGE
# ✓       feat: افزودن پروفایل          CI        main    push   1234567890  3m12s    5m
# X       fix: رفع تست                  CI        feat/x  pull   1234567891  2m45s    1h
# *       docs: به‌روزرسانی              CI        main    push   1234567892  1m       1m
```

نمادها: `✓` موفق، `X` شکست، `*` در حال اجرا، `-` لغو.

#### مثال ۲: دیدن لاگ شکست

```bash
gh run view 1234567891 --log-failed
# test (20.x)	Run npm test	2026-10-23T10:15:32.123Z
# FAIL src/auth.test.ts
#   ● login › should normalize email
#     expect(received).toBe(expected)
```

**`--log-failed`** سریع‌ترین راه — فقط بخش شکست.

#### مثال ۳: لاگ کامل با جست‌وجو

```bash
gh run view 1234567891 --log > full.log
grep -i "error\|fail" full.log | head -30
```

#### مثال ۴: watch زنده

```bash
gh run watch
# یا
gh run watch 1234567891
# ✓ test (20.x)  1m23s
# * build        0m45s
```

#### مثال ۵: اجرای دستی

```bash
gh workflow list
gh workflow run ci.yml
gh workflow run deploy.yml -f environment=staging -f dry_run=true
gh workflow run ci.yml --ref feature/x
```

#### مثال ۶: اجرای مجدد

```bash
# فقط jobهای شکست‌خورده
gh run rerun 1234567891 --failed

# همه
gh run rerun 1234567891

# با debug logging
gh run rerun 1234567891 --debug
```

`--debug` معادل `ACTIONS_STEP_DEBUG=true` است — بدون نیاز به secret.

#### مثال ۷: دانلود artifact

```bash
gh run view 1234567891 --json artifacts --jq '.artifacts[] | .name'
gh run download 1234567891
gh run download 1234567891 -n coverage
gh run download 1234567891 -n dist -D ./output
```

#### مثال ۸: لیست jobها

```bash
gh run view 1234567891 --json jobs \
  --jq '.jobs[] | "\(.conclusion // .status)\t\(.name)"'
# success	test (20.x)
# failure	test (22.x)
# success	build
```

#### مثال ۹: پیدا کردن آخرین اجرای شکست‌خورده

```bash
gh run list --status failure --limit 1 --json databaseId --jq '.[0].databaseId'
```

#### مثال ۱۰: اسکریپت گزارش CI

```bash
#!/usr/bin/env bash
set -euo pipefail

REPO="${1:-.}"

echo "## وضعیت CI — $(date '+%Y-%m-%d %H:%M')"
echo

echo "### اجراهای امروز"
gh run list --repo "$REPO" --limit 20 \
  --json conclusion,displayTitle,workflowName,createdAt \
  --jq '.[] | select(.createdAt > (now - 86400 | todate)) |
        "\(.conclusion // "running")\t\(.workflowName)\t\(.displayTitle)"'

echo
echo "### شکست‌ها (۷ روز)"
gh run list --repo "$REPO" --status failure --limit 20 \
  --json displayTitle,workflowName,createdAt,databaseId \
  --jq '.[] | "- \(.workflowName): \(.displayTitle) (\(.databaseId))"'
```

#### مثال ۱۱: Issue خودکار برای CI شبانه

```yaml
name: Nightly failure report

on:
  workflow_run:
    workflows: [Nightly]
    types: [completed]

permissions:
  issues: write
  actions: read

jobs:
  report:
    if: github.event.workflow_run.conclusion == 'failure'
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/github-script@<sha>
        with:
          script: |
            const run = context.payload.workflow_run;
            await github.rest.issues.create({
              ...context.repo,
              title: `CI شبانه شکست خورد — ${run.created_at.slice(0,10)}`,
              body: `[اجرا](${run.html_url}) شکست خورد.`,
              labels: ["ci", "bug"],
            });
```

#### مثال ۱۲: بررسی وضعیت PR از gh

```bash
gh pr checks 42 --watch
# ✓  CI / lint        32s
# ✓  CI / test (20)   1m23s
# X  CI / test (22)   1m31s
```

### مقایسه و الگوهای مختلف

#### `gh run view --log` در برابر `--log-failed`

| | `--log` | `--log-failed` |
|---|---|---|
| حجم | زیاد | کم |
| سرعت | کند | ✅ |
| مناسب | دیباگ عمیق | خطا |

#### `gh run watch` در برابر مرورگر

| | watch | مرورگر |
|---|---|---|
| جریان کار | ✅ (در ترمینال) | ❌ |
| جزئیات | محدود | ✅ |
| مناسب | انتظار | بررسی |

#### `rerun --failed` در برابر `rerun`

| | `--failed` | همه |
|---|---|---|
| سرعت | ✅ | کند |
| هزینه | کم | زیاد |
| مناسب | خطای گذرا | تغییر تنظیمات |

### اشکال شایع

#### ۱. `--log` خیلی طولانی است

از `--log-failed` یا `grep` استفاده کن.

#### ۲. `gh run view` بدون ID

آخرین اجرای branch جاری را می‌گیرد. اگر روی branch دیگری هستی، ID بده.

#### ۳. `gh workflow run` کار نمی‌کند

workflow باید `workflow_dispatch` داشته باشد.

#### ۴. `gh run download` خالی است

artifact منقضی شده (۹۰ روز پیش‌فرض) یا نام اشتباه.

#### ۵. `rerun` دوباره شکست می‌خورد

اگر خطا قطعی است (نه گذرا)، rerun بی‌فایده است. لاگ را بخوان.

#### ۶. `--debug` در rerun

```
Error: --debug is not supported with --job
```

`--debug` فقط برای کل run است.

#### ۷. workflow disable شده

```
workflow is disabled
```

`gh workflow enable ci.yml`.

### نکات مهم (Tips & Tricks)

- **`--log-failed`** — سریع‌ترین دیباگ.
- **`gh run rerun --failed --debug`** — تلاش دوم با لاگ کامل.
- **`gh run watch`** — انتظار بدون مرورگر.
- **`--json` + `--jq`** برای گزارش.
- **`gh workflow run -f key=value`** برای inputها.
- **`gh pr checks --watch`** — وضعیت PR.

### خلاصه فصل

- `gh workflow` تعریف را مدیریت می‌کند؛ `gh run` اجراها را.
- `--log-failed` سریع‌ترین راه دیباگ.
- `rerun --failed --debug` برای تلاش دوم.
- `gh run download` برای artifact.
- `--json` + `--jq` برای اسکریپت.

**سؤالات مروری:**
1. تفاوت `gh workflow` و `gh run` چیست؟
2. چطور فقط jobهای شکست‌خورده را دوباره اجرا کنی؟
3. چطور artifact یک اجرا را دانلود کنی؟

---

## فصل ۷۹: `gh api`، `gh alias` و extensionها

### مقدمه

`gh api` دسترسی کامل به REST و GraphQL API می‌دهد — با احراز هویت خودکار و خروجی قابل `jq`. `gh alias` میان‌بر می‌سازد. Extensionها قابلیت اضافه می‌کنند.

**چرا مهم است؟** بعضی کارها UI ندارند یا در `gh` نیستند. `gh api` همه‌چیز را باز می‌کند.

**کاربرد عملی:** بعد از این فصل، هیچ محدودیتی در GitHub API نداری.

### مفاهیم اساسی

#### `gh api` — سینتکس

```bash
gh api <endpoint> [flags]
```

| Flag | کار |
|---|---|
| `-X, --method` | HTTP method (GET پیش‌فرض) |
| `-f, --field key=value` | فیلد string |
| `-F, --raw-field key=value` | فیلد typed (عدد، boolean، @file) |
| `--input <file>` | body از فایل JSON |
| `-H, --header` | هدر |
| `--paginate` | همه‌ی صفحه‌ها |
| `--jq` | فیلتر jq |
| `--template` | قالب Go |
| `--silent` | بدون خروجی |
| `-i, --include` | شامل هدرهای response |

#### REST در برابر GraphQL

| | REST | GraphQL |
|---|---|---|
| سینتکس | `gh api repos/o/r` | `gh api graphql -f query='...'` |
| خروجی | JSON کامل | دقیقاً آنچه خواستی |
| Pagination | `--paginate` | cursor دستی |
| مناسب | عملیات ساده | کوئری پیچیده |

#### `-f` در برابر `-F`

| | `-f` | `-F` |
|---|---|---|
| نوع | همیشه string | typed |
| عدد | `"123"` | `123` |
| boolean | `"true"` | `true` |
| فایل | ❌ | `@file.json` |

### مثال‌های عملی

#### مثال ۱: GET ساده

```bash
gh api repos/cli/cli
gh api repos/cli/cli --jq '.stargazers_count'
gh api user --jq '.login'
gh api rate_limit --jq '.rate'
```

#### مثال ۲: POST

```bash
gh api -X POST repos/OWNER/REPO/issues \
  -f title="عنوان" \
  -f body="توضیح" \
  -f 'labels[]=bug'
```

#### مثال ۳: PATCH

```bash
gh api -X PATCH repos/OWNER/REPO \
  -f description="توضیح جدید" \
  -F has_wiki=true
```

#### مثال ۴: DELETE

```bash
gh api -X DELETE repos/OWNER/REPO/issues/42
```

#### مثال ۵: pagination

```bash
# همه‌ی Issueها
gh api repos/OWNER/REPO/issues --paginate --jq '.[].number'

# با limit
gh api --paginate --slurp repos/OWNER/REPO/issues --jq 'add | length'
```

`--slurp` همه‌ی صفحه‌ها را در یک آرایه می‌گذارد.

#### مثال ۶: GraphQL

```bash
gh api graphql -f query='
query($owner: String!, $repo: String!) {
  repository(owner: $owner, name: $repo) {
    pullRequests(first: 10, states: OPEN) {
      nodes {
        number
        title
        author { login }
        reviewDecision
      }
    }
  }
}' -f owner=myorg -f repo=backend --jq '.data.repository.pullRequests.nodes'
```

#### مثال ۷: GraphQL با متغیر

```bash
gh api graphql \
  -f query='
    query($owner: String!, $repo: String!, $number: Int!) {
      repository(owner: $owner, name: $repo) {
        pullRequest(number: $number) {
          title
          additions
          deletions
          files(first: 100) { nodes { path additions } }
        }
      }
    }' \
  -f owner=myorg -f repo=backend -F number=42 \
  --jq '.data.repository.pullRequest | {title, additions, deletions}'
```

#### مثال ۸: Projects (GraphQL اجباری)

```bash
# لیست Projectها
gh api graphql -f query='
query($org: String!) {
  organization(login: $org) {
    projectsV2(first: 10) {
      nodes { id number title }
    }
  }
}' -f org=myorg

# افزودن آیتم
gh api graphql -f query='
mutation($project: ID!, $content: ID!) {
  addProjectV2ItemById(input: {projectId: $project, contentId: $content}) {
    item { id }
  }
}' -f project=PVT_xxx -f content=I_xxx
```

#### مثال ۹: آپلود فایل (release asset)

```bash
gh api -X POST \
  -H "Content-Type: application/octet-stream" \
  "https://uploads.github.com/repos/OWNER/REPO/releases/123/assets?name=app.zip" \
  --input app.zip
```

#### مثال ۱۰: `gh alias`

```bash
gh alias set prs "pr list --state open --limit 50"
gh alias set mywork "pr list --author @me"
gh alias set reviews "pr list --review-requested @me"
gh alias set co "pr checkout"
gh alias set bugs "issue list --label bug --state open"

# با آرگومان
gh alias set --shell prc 'gh pr create --fill --web'

# لیست
gh alias list

# حذف
gh alias delete prs
```

`--shell` اجازه می‌دهد دستور shell بنویسی:

```bash
gh alias set --shell stale 'gh pr list --json number,updatedAt --jq ".[] | select(.updatedAt < (now - 2592000 | todate)) | .number"'
```

#### مثال ۱۱: Extensionها

```bash
# لیست
gh extension list

# نصب
gh extension install github/gh-copilot
gh extension install dlvhdr/gh-dash
gh extension install mislav/gh-branch
gh extension install yuler/gh-download

# استفاده
gh copilot suggest "list files by size"
gh dash
gh branch

# حذف
gh extension remove gh-copilot

# ساخت extension خودت
gh extension create my-extension
```

#### مثال ۱۲: extensionهای مفید

| Extension | کار |
|---|---|
| `gh-dash` | داشبورد TUI برای PR/Issue |
| `gh-copilot` | Copilot در ترمینال |
| `gh-branch` | انتخاب branch تعاملی |
| `gh-download` | دانلود فایل از مخزن |
| `gh-poi` | پاک کردن branchهای مرده |
| `gh-stack` | stacked PRs |
| `gh-notify` | اعلان‌ها |

#### مثال ۱۳: اسکریپت با `gh api`

```bash
#!/usr/bin/env bash
set -euo pipefail

# گزارش هفتگی: PRهای merge‌شده با آمار
gh api "search/issues?q=repo:myorg/backend+is:pr+is:merged+merged:>=$(date -d '7 days ago' +%Y-%m-%d)" \
  --paginate \
  --jq '.items[] | "\(.number)\t\(.title)\t@\(.user.login)"'
```

#### مثال ۱۴: rate limit

```bash
gh api rate_limit --jq '.resources.core'
# {"limit":5000,"used":123,"remaining":4877,"reset":...}

# اگر نزدیک limit هستی، از GraphQL استفاده کن (سهمیه‌ی جدا)
```

#### مثال ۱۵: خطاها

```bash
gh api repos/nonexistent/repo 2>&1
# gh: Not Found (HTTP 404)
```

برای دیدن جزئیات:

```bash
gh api repos/nonexistent/repo -i 2>&1 | head -20
```

### مقایسه و الگوهای مختلف

#### `gh api` در برابر `curl`

| | gh api | curl |
|---|---|---|
| احراز هویت | ✅ خودکار | دستی |
| pagination | ✅ | ❌ |
| jq | ✅ داخلی | جدا |
| مناسب | اکثر کارها | کنترل کامل |

#### alias در برابر shell function

| | alias | function |
|---|---|---|
| سینتکس | `gh alias set` | `.bashrc` |
| دامنه | فقط gh | همه |
| آرگومان | محدود | ✅ |

#### extension در برابر script

| | extension | script |
|---|---|---|
| توزیع | `gh extension install` | کپی فایل |
| به‌روزرسانی | `gh extension upgrade` | دستی |
| مناسب | ابزار عمومی | کار شخصی |

### اشکال شایع

#### ۱. `-f` برای عدد

```bash
✗ gh api ... -f number="42"    # string
✓ gh api ... -F number=42      # عدد
```

بعضی endpointها نوع را چک می‌کنند.

#### ۲. آرایه با `-f`

```bash
✗ -f labels="bug,enhancement"
✓ -f 'labels[]=bug' -f 'labels[]=enhancement'
```

#### ۳. GraphQL بدون `query=`

```bash
✗ gh api graphql -f 'query { viewer { login } }'
✓ gh api graphql -f query='query { viewer { login } }'
```

#### ۴. pagination فراموش شد

```bash
✗ gh api repos/o/r/issues         # فقط ۳۰
✓ gh api repos/o/r/issues --paginate
```

#### ۵. rate limit

اگر ۵۰۰۰ درخواست در ساعت تمام شد، ۴۰۳ می‌گیری. `--paginate` روی مخزن بزرگ می‌تواند سریع تمام کند.

#### ۶. مسیر اشتباه

```
✗ repos/o/r/pulls/42/comments     # برای review comment
✓ repos/o/r/pulls/42/comments     # درست است، ولی:
✓ repos/o/r/pulls/42/reviews      # برای review
```

مستندات REST را چک کن.

#### ۷. `gh alias` با آرگومان

aliasهای `gh` آرگومان اضافه را به انتها می‌چسبانند:

```bash
gh alias set prs "pr list --state open"
gh prs --limit 100    # کار می‌کند
```

#### ۸. extension نیاز به dependency

بعضی extensionها به `fzf`، `jq`، یا ابزار دیگر نیاز دارند. `README` را بخوان.

### نکات مهم (Tips & Tricks)

- **`-F` برای عدد/boolean، `-f` برای string.**
- **`--paginate`** را فراموش نکن.
- **`--jq`** داخلی است — نیازی به pipe نیست.
- **`gh api graphql`** برای Projects و کوئری پیچیده.
- **`gh alias set --shell`** برای دستورهای پیچیده.
- **`gh-dash`** را امتحان کن — داشبورد TUI عالی.

### خلاصه فصل

- `gh api` دسترسی کامل REST/GraphQL با احراز هویت خودکار.
- `-f` string، `-F` typed.
- `--paginate` برای همه‌ی صفحه‌ها.
- GraphQL برای Projects و کوئری دقیق.
- `gh alias` میان‌بر می‌سازد.
- Extensionها قابلیت اضافه می‌کنند.

**سؤالات مروری:**
1. تفاوت `-f` و `-F` چیست؟
2. چرا Projects نیاز به GraphQL دارد؟
3. چطور یک alias با دستور shell بسازی؟

---

**بعدی:** [`18-Pages-Codespaces-and-More.md`](18-Pages-Codespaces-and-More.md)

---

## منابع و مراجع

- GitHub CLI manual — https://cli.github.com/manual/
- GitHub CLI repo — https://github.com/cli/cli
- GitHub REST API — https://docs.github.com/en/rest
- GitHub GraphQL API — https://docs.github.com/en/graphql
- `gh` extensions — https://github.com/topics/gh-extension
- `gh-dash` — https://github.com/dlvhdr/gh-dash
- `jq` manual — https://jqlang.github.io/jq/manual/
