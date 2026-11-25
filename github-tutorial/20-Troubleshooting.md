# بخش ۲۰: عیب‌یابی — خطاهای رایج Git و GitHub

**مدت یادگیری**: ۴ تا ۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۰۴–۰۵ (remote، بازنویسی تاریخ)، بخش ۱۰–۱۲ (Actions)

---

## فهرست محتوا

- [فصل ۸۹: خطاهای احراز هویت و دسترسی](#فصل-۸۹)
- [فصل ۹۰: خطاهای merge، rebase و وضعیت branch](#فصل-۹۰)
- [فصل ۹۱: خطاهای Actions و CI](#فصل-۹۱)
- [فصل ۹۲: خطاهای حجم، کارایی و مخزن بزرگ](#فصل-۹۲)
- [فصل ۹۳: بازیابی فاجعه](#فصل-۹۳)

---

## فصل ۸۹: خطاهای احراز هویت و دسترسی

### مقدمه

این خانواده از خطاها بیشترین سؤال را می‌سازند، چون پیام‌ها گنگ‌اند: `403`، `404`، `Permission denied`. و علت معمولاً یکی از پنج چیز است.

**چرا مهم است؟** بدون تشخیص درست، آدم ساعت‌ها دنبال مشکل می‌گردد در حالی که جواب در `git remote -v` است.

**کاربرد عملی:** این فصل یک درخت تصمیم می‌دهد.

### مفاهیم اساسی

#### درخت تصمیم سریع

```
خطای دسترسی؟
├── 403 Forbidden
│   ├── push به branch محافظت‌شده → PR بزن
│   ├── توکن منقضی → gh auth login
│   ├── scope ناکافی → gh auth refresh -s ...
│   └── SSO سازمان → authorize token
├── 404 Not Found
│   ├── مخزن خصوصی و دسترسی نداری (GitHub 404 می‌دهد نه 403)
│   ├── URL اشتباه / typo
│   └── توکن بدون دسترسی به آن مخزن
└── Permission denied (publickey)
    ├── کلید در GitHub نیست
    ├── ssh-agent ندارد
    ├── remote با HTTPS است
    └── مجوز فایل اشتباه
```

#### نکته‌ی مهم: 404 به‌جای 403

GitHub برای مخزن خصوصی که دسترسی نداری، **404** می‌دهد نه 403 — تا وجود مخزن لو نرود. اگر مطمئنی مخزن هست ولی 404 می‌گیری، مشکل دسترسی است.

### مثال‌های عملی

#### مثال ۱: `Permission denied (publickey)`

```bash
# ۱) آیا SSH کار می‌کند؟
ssh -T git@github.com
# Hi ali! You've successfully authenticated...

# اگر نه، verbose:
ssh -vT git@github.com 2>&1 | grep -E "Offering|Authenticated|denied"

# ۲) remote چیست؟
git remote -v
# اگر https:// بود، به SSH تغییر بده:
git remote set-url origin git@github.com:OWNER/REPO.git

# ۳) کلید در agent هست؟
ssh-add -l
# اگر خالی: ssh-add ~/.ssh/id_ed25519

# ۴) مجوز فایل‌ها
ls -la ~/.ssh/
# باید: drwx------ .ssh، -rw------- id_ed25519، -rw-r--r-- id_ed25519.pub
chmod 700 ~/.ssh && chmod 600 ~/.ssh/id_ed25519

# ۵) تست با کلید مشخص
ssh -i ~/.ssh/id_ed25519 -T git@github.com
```

#### مثال ۲: `Support for password authentication was removed`

```bash
# راه ۱: SSH
git remote set-url origin git@github.com:OWNER/REPO.git

# راه ۲: PAT
git remote set-url origin https://github.com/OWNER/REPO.git
# سپس username = نام کاربری، password = PAT
```

#### مثال ۳: `403 Forbidden` روی push

```bash
# آیا branch محافظت شده؟
gh api repos/OWNER/REPO/branches/main/protection --jq '.required_pull_request_reviews'

# آیا دسترسی write داری؟
gh api repos/OWNER/REPO --jq '.permissions'
# {"admin":false,"maintain":false,"push":true,"triage":false,"pull":true}
# push:true یعنی write داری
```

اگر branch محافظت شده، PR بزن — راه دیگری نیست.

#### مثال ۴: `remote: Permission to X denied to Y`

```bash
# با کدام حساب authenticate شده‌ای؟
gh auth status
git config user.email
ssh -T git@github.com

# اگر حساب اشتباه است:
gh auth switch
# یا کلید SSH درست را تنظیم کن (~/.ssh/config)
```

#### مثال ۵: SSO — `Resource protected by organization SAML enforcement`

```bash
# توکن باید authorize شود
# در مرورگر: Settings → Organizations → org → SSO authorize
# یا:
gh auth refresh -s read:org
```

#### مثال ۶: `fatal: could not read Username`

```bash
# در CI: توکن در URL
git clone https://x-access-token:${GITHUB_TOKEN}@github.com/OWNER/REPO.git

# یا gh:
GH_TOKEN=$GITHUB_TOKEN gh repo clone OWNER/REPO
```

**هشدار:** توکن در URL در `.git/config` ذخیره می‌شود. در CI مشکلی نیست (مخزن موقتی)، ولی محلی نه.

#### مثال ۷: `Host key verification failed`

```bash
ssh-keyscan github.com >> ~/.ssh/known_hosts
# یا در CI:
ssh-keyscan -t ed25519 github.com >> ~/.ssh/known_hosts
```

**هشدار:** `ssh-keyscan` بدون بررسی، در معرض MITM است. برای CI، fingerprint را از مستندات GitHub بگیر.

#### مثال ۸: `bad credentials` در gh

```bash
gh auth status
gh auth login
# یا
gh auth refresh
```

#### مثال ۹: `Your token has not been granted the required scopes`

```bash
gh auth refresh -s workflow,write:packages
```

scopeهای رایج: `repo`، `workflow`، `write:packages`، `read:org`، `admin:public_key`.

#### مثال ۱۰: `Repository not found` در Actions

اگر workflow به مخزن دیگر دسترسی می‌خواهد:

```yaml
- uses: actions/checkout@<sha>
  with:
    repository: other/repo
    token: ${{ secrets.PAT_FOR_OTHER_REPO }}
```

`GITHUB_TOKEN` فقط به مخزن خودش دسترسی دارد.

#### مثال ۱۱: `Resource not accessible by integration`

در Actions، `GITHUB_TOKEN` دسترسی کافی ندارد:

```yaml
permissions:
  issues: write        # ← اضافه کن
  pull-requests: write
```

### مقایسه و الگوهای مختلف

#### 401 در برابر 403 در برابر 404

| کد | معنی | معمولاً |
|---|---|---|
| 401 | احراز هویت نشده | توکن نبود/باطل |
| 403 | احراز شده ولی مجاز نیست | scope/permission/ruleset |
| 404 | پیدا نشد (یا مخفی) | دسترسی/URL |

#### SSH در برابر HTTPS برای debug

| | SSH | HTTPS |
|---|---|---|
| خطا | `publickey` | `403`/`401` |
| دیباگ | `ssh -vT` | `GIT_CURL_VERBOSE=1` |
| معمولاً | کلید | توکن |

#### `gh auth` در برابر `git config`

| | gh auth | git config |
|---|---|---|
| توکن | ✅ مدیریت | ❌ |
| SSH | ✅ setup | ❌ |
| بررسی | `gh auth status` | `git config --list` |

### اشکال شایع

#### ۱. حدس زدن به‌جای دیباگ

```bash
# به‌جای ۱۰ بار امتحان، این سه را بزن:
gh auth status
git remote -v
ssh -T git@github.com
```

#### ۲. توکن در URL

```bash
✗ git remote set-url origin https://ghp_xxx@github.com/...
```

در `.git/config` ذخیره می‌شود و در `git remote -v` دیده می‌شود. از credential helper استفاده کن.

#### ۳. `StrictHostKeyChecking=no`

```bash
✗ ssh -o StrictHostKeyChecking=no
```

MITM ممکن است. `known_hosts` درست بگذار.

#### ۴. مجوز فایل SSH

اگر `.ssh` مجوز ۷۷۷ داشته باشد، SSH کلید را نادیده می‌گیرد.

#### ۵. کلید اشتباه با چند حساب

`~/.ssh/config` با `IdentitiesOnly yes` و `Host`های جدا.

### نکات مهم (Tips & Tricks)

- **`gh auth status`** اولین دستور.
- **`git remote -v`** دوم.
- **`ssh -vT`** برای SSH.
- **`GIT_CURL_VERBOSE=1`** برای HTTPS.
- **404 در GitHub معمولاً دسترسی است، نه نبود.**
- **`permissions:` را در workflow صریح بنویس.**

### خلاصه فصل

- سه خطای اصلی: `publickey`، `403`، `404`.
- `gh auth status` + `git remote -v` + `ssh -T` = تشخیص سریع.
- branch محافظت‌شده → PR.
- `GITHUB_TOKEN` فقط به مخزن خودش.
- `permissions:` صریح.

**سؤالات مروری:**
1. چرا GitHub برای مخزن خصوصی 404 می‌دهد نه 403؟
2. سه دستور اول در هر مشکل دسترسی چیست؟
3. چرا `GITHUB_TOKEN` به مخزن دیگر دسترسی ندارد؟

---

## فصل ۹۰: خطاهای merge، rebase و وضعیت branch

### مقدمه

این خانواده از خطاها بیشتر «گیج‌کننده» هستند تا «خطرناک»: `detached HEAD`، `diverged`، `conflict`، `not fully merged`. هر کدام یک دلیل مشخص و یک راه‌حل مشخص دارند.

**چرا مهم است؟** اگر بدانی هر وضعیت چه معنایی دارد، در ۳۰ ثانیه حل می‌شود. اگر نه، ممکن است کارت را از دست بدهی.

**کاربرد عملی:** این فصل یک مرجع سریع است.

### مفاهیم اساسی

#### وضعیت‌های branch

```bash
git status -sb
## main...origin/main                 ← همگام
## main...origin/main [ahead 2]       ← ۲ commit push نشده
## main...origin/main [behind 3]      ← ۳ commit روی سرور
## main...origin/main [ahead 2, behind 3]  ← diverged
## HEAD (no branch)                   ← detached
```

#### درخت تصمیم

```
merge/rebase مشکل دارد؟
├── CONFLICT (content)
│   └── حل کن → add → continue/commit
├── diverged (ahead + behind)
│   └── pull --rebase یا merge
├── detached HEAD
│   └── switch -c name (اگر کار ارزش دارد)
├── branch is not fully merged
│   └── بررسی کن → -D یا merge
└── unrelated histories
    └── --allow-unrelated-histories
```

### مثال‌های عملی

#### مثال ۱: `CONFLICT (content): Merge conflict in X`

```bash
# ۱) کدام فایل‌ها؟
git status
# both modified: src/auth.js

# ۲) ببین
git diff src/auth.js
# <<<<<<< HEAD
# ...
# =======
# ...
# >>>>>>> feature/x

# ۳) حل کن (ادیتور یا VS Code)
code src/auth.js

# ۴) stage
git add src/auth.js

# ۵) ادامه
git commit                 # برای merge
git rebase --continue      # برای rebase
git cherry-pick --continue # برای cherry-pick

# یا لغو
git merge --abort
git rebase --abort
git cherry-pick --abort
```

#### مثال ۲: `Your local changes would be overwritten by checkout`

```bash
# ۱) تغییرات چیست؟
git status
git diff

# ۲) سه راه:
git stash              # ذخیره و بعد pop
git commit -m "wip"    # commit موقت
git restore .          # دور ریختن (خطرناک!)
```

#### مثال ۳: `diverged` — `ahead 2, behind 3`

```bash
# راه ۱: rebase (تاریخ خطی)
git pull --rebase

# راه ۲: merge
git pull --no-rebase

# راه ۳: دستی
git fetch
git log HEAD..origin/main --oneline    # چه چیزی روی سرور است
git rebase origin/main
```

#### مثال ۴: `fatal: refusing to merge unrelated histories`

```bash
# معمولاً وقتی دو مخزن مستقل را merge می‌کنی
git pull origin main --allow-unrelated-histories

# یا merge بین دو branch مستقل
git merge other --allow-unrelated-histories
```

**بهتر:** از اول یک مخزن داشته باش.

#### مثال ۵: `detached HEAD`

```bash
git status
# HEAD detached at a1b2c3d

# اگر کاری نکرده‌ای:
git switch main

# اگر commit زده‌ای و می‌خواهی نگه داری:
git switch -c rescue-branch

# اگر گم کردی:
git reflog
git switch -c rescue a1b2c3d
```

#### مثال ۶: `error: The branch 'x' is not fully merged`

```bash
# ببین چه چیزی merge نشده
git log main..x --oneline

# اگر لازم نیست:
git branch -D x

# اگر لازم است:
git switch main && git merge x
```

#### مثال ۷: `fatal: Not possible to fast-forward, aborting`

```bash
# pull --ff-only با تاریخ واگرا
git pull --rebase
# یا
git merge origin/main
```

#### مثال ۸: `error: cannot lock ref`

```bash
# معمولاً ref خراب یا فرآیند هم‌زمان
# ۱) فرآیند Git دیگر را ببند
# ۲) اگر ref خراب است:
rm .git/refs/heads/broken-branch
git fetch
# ۳) اگر index قفل است:
rm .git/index.lock
```

**هشدار:** `index.lock` را فقط وقتی پاک کن که مطمئنی هیچ Git دیگری در حال اجرا نیست.

#### مثال ۹: `fatal: bad object HEAD`

```bash
# مخزن خراب
git fsck --full
# اگر HEAD به commit ناموجود اشاره می‌کند:
cat .git/HEAD
git reflog
git reset --hard <valid-commit>
```

#### مثال ۱۰: `error: pathspec 'x' did not match any file(s) known to git`

```bash
# فایل وجود ندارد یا نام اشتباه
git ls-files | grep x
git status
```

#### مثال ۱۱: `warning: CRLF will be replaced by LF`

```bash
# هشدار، نه خطا. برای رفع:
git config --global core.autocrlf input   # مک/لینوکس
git config --global core.autocrlf true    # ویندوز

# یا بهتر:
echo "* text=auto eol=lf" > .gitattributes
git add --renormalize .
git commit -m "chore: normalize line endings"
```

#### مثال ۱۲: `fatal: unable to access ... SSL certificate problem`

```bash
# در شبکه‌ی شرکتی با proxy
git config --global http.proxy http://proxy:8080
git config --global https.proxy http://proxy:8080

# یا certificate شرکتی
git config --global http.sslCAInfo /path/to/cert.pem

# ❌ هرگز: http.sslVerify false
```

#### مثال ۱۳: `fatal: The current branch x has no upstream branch`

```bash
git push -u origin x
# یا یک‌بار برای همیشه:
git config --global push.autoSetupRemote true
```

#### مثال ۱۴: `git push` rejected — `non-fast-forward`

```bash
git pull --rebase
git push
# هرگز --force روی branch مشترک
```

#### مثال ۱۵: `fatal: ambiguous argument 'HEAD~2'`

```bash
# commit کافی وجود ندارد
git log --oneline
# از تعداد موجود استفاده کن یا از هش
```

### مقایسه و الگوهای مختلف

#### `--abort` در برابر حل کردن

| | abort | حل |
|---|---|---|
| سرعت | ✅ فوری | کند |
| از دست رفتن کار | ✅ (کار merge) | ❌ |
| مناسب | وقتی گیج شده‌ای | وقتی می‌دانی چه می‌خواهی |

#### `pull --rebase` در برابر `pull`

| | rebase | merge |
|---|---|---|
| تاریخ | خطی | merge commit |
| conflict | per-commit | یک‌بار |
| مناسب | branch شخصی | branch مشترک |

#### `-d` در برابر `-D`

| | `-d` | `-D` |
|---|---|---|
| ایمنی | ✅ چک می‌کند | ❌ اجباری |
| مناسب | merge شده | رها شده |

### اشکال شایع

#### ۱. `git reset --hard` برای حل همه‌چیز

کار uncommitted از بین می‌رود. اول `git stash` یا `commit`.

#### ۲. `git push --force` برای حل rejection

کار دیگران از بین می‌رود. `--force-with-lease` یا pull.

#### ۳. پاک کردن `index.lock` بدون بررسی

اگر Git دیگری در حال اجراست، مخزن خراب می‌شود.

#### ۴. `http.sslVerify false`

امنیت را کاملاً از بین می‌برد. هرگز.

#### ۵. حذف branch بدون بررسی

```bash
git log main..branch --oneline   # اول ببین
git branch -D branch
```

#### ۶. commit کردن conflict markers

```bash
git grep -n "<<<<<<<"
```

در CI هم چک کن.

### نکات مهم (Tips & Tricks)

- **`git status` قبل و بعد از هر عملیات.**
- **`git merge --abort` / `rebase --abort`** — اولین کار در گیجی.
- **`git reflog`** — شبکه‌ی نجات.
- **`git log main..branch --oneline`** قبل از حذف.
- **`--force-with-lease`** همیشه، `--force` هرگز.
- **`git grep "<<<<<<<"`** قبل از commit.

### خلاصه فصل

- اکثر خطاها یک دلیل مشخص دارند.
- `--abort` برای لغو، `--continue` برای ادامه.
- `diverged` → `pull --rebase`.
- `detached HEAD` → `switch -c`.
- `--allow-unrelated-histories` برای مخزن مستقل.
- `index.lock` را با احتیاط پاک کن.

**سؤالات مروری:**
1. تفاوت `merge --abort` و `reset --hard` چیست؟
2. چرا `detached HEAD` خطرناک است و چطور نجات پیدا می‌کند؟
3. چرا `http.sslVerify false` خطرناک است؟

---

## فصل ۹۱: خطاهای Actions و CI

### مقدمه

Actions خطاهایش گاهی گنگ است: `Resource not accessible`، `Expected — Waiting for status`، `YAML syntax error`. این فصل یک مرجع سریع است.

**چرا مهم است؟** CI خطا خورد، تیم گیر می‌کند. تشخیص سریع = جریان کار ادامه دارد.

**کاربرد عملی:** ۱۵ خطای رایج با راه‌حل.

### مفاهیم اساسی

#### دسته‌بندی خطاها

| دسته | نمونه |
|---|---|
| **Syntax** | YAML نامعتبر، کلید اشتباه |
| **Permission** | `Resource not accessible` |
| **Trigger** | workflow اجرا نمی‌شود |
| **Check name** | `Expected — Waiting for status` |
| **Environment** | `command not found`، نسخه‌ی اشتباه |
| **Timeout** | job گیر کرده |
| **Secret** | خالی یا ناموجود |

#### ابزارها

```bash
actionlint .github/workflows/*.yml     # syntax + context
zizmor .github/workflows/              # امنیت
gh run view <id> --log-failed          # خطا
gh run rerun <id> --failed --debug     # تلاش دوم با debug
```

### مثال‌های عملی

#### مثال ۱: `Invalid workflow file` — YAML

```
Invalid workflow file: .github/workflows/ci.yml#L12
You have an error in your yaml syntax on line 12
```

```bash
# راه‌حل: actionlint
actionlint .github/workflows/ci.yml
```

**خطاهای رایج:**

```yaml
✗ runs-on: ubuntu-latest    (در سطح jobs)
✗ steps:                    (indentation)
✗ on: [push, pull_request   (براکت بسته نشده)
✗ - uses: actions/checkout@v4
    with:                    (indentation)
```

#### مثال ۲: workflow اجرا نمی‌شود

چک‌لیست:

1. فایل در `.github/workflows/` است؟ (نه `.github/workflow`)
2. YAML معتبر است؟ (GitHub در تب Actions خطا نشان می‌دهد)
3. `on:` شرط را برآورده می‌کند؟ (`branches`، `paths`)
4. workflow در branch پیش‌فرض است؟ (بعضی eventها فقط از branch پیش‌فرض trigger می‌شوند)
5. workflow disable نشده؟ (`gh workflow list`)

#### مثال ۳: `Expected — Waiting for status to be reported`

required check هست ولی اجرا نمی‌شود.

```bash
# ۱) اسم دقیق check
gh pr checks 42

# ۲) آیا workflow روی این event اجرا می‌شود؟
grep -A5 "^on:" .github/workflows/ci.yml

# ۳) آیا اسم job عوض شده؟
# ruleset را ببین:
gh api repos/OWNER/REPO/rulesets --jq '.[] | .name'
```

راه‌حل بلندمدت: `ci-success` job (فصل ۴۰).

#### مثال ۴: `Resource not accessible by integration`

```yaml
✗ permissions:
    contents: read
  # ولی workflow می‌خواهد comment بگذارد
```

```yaml
✓ permissions:
    contents: read
    pull-requests: write
```

#### مثال ۵: `command not found`

```yaml
✗ - run: npm ci
  # bash: npm: command not found
```

`setup-node` نبوده، یا در job اشتباه هستی (هر job محیط جدا دارد).

#### مثال ۶: `Permission denied` روی اسکریپت

```yaml
- run: chmod +x scripts/deploy.sh && ./scripts/deploy.sh
```

یا در Git:

```bash
git update-index --chmod=+x scripts/deploy.sh
git commit -m "chore: اجرایی کردن اسکریپت"
```

#### مثال ۷: `npm ci` با lock ناسازگار

```
npm error: `npm ci` can only install packages when your package.json
and package-lock.json are in sync
```

```bash
# محلی:
npm install
git add package-lock.json
git commit -m "chore: به‌روزرسانی lock"
```

#### مثال ۸: timeout

```
The job running on runner ... has exceeded the maximum execution time of 360 minutes.
```

- `timeout-minutes` بگذار
- job را بشکن
- self-hosted runner (بدون محدودیت ۶ ساعت)

#### مثال ۹: `No space left on device`

```yaml
- name: آزاد کردن فضا
  run: |
    sudo rm -rf /usr/share/dotnet /usr/local/lib/android /opt/ghc
    df -h
```

#### مثال ۱۰: secret خالی

```yaml
- run: echo "len=${#TOKEN}"
  env:
    TOKEN: ${{ secrets.MY_TOKEN }}
```

اگر `len=0`:

- secret تعریف نشده
- در fork (طراحی)
- در `if:` استفاده شده (ممنوع)
- نام اشتباه (حساس به حرف بزرگ/کوچک)

#### مثال ۱۱: `This request has been automatically failed because it uses a deprecated version of actions/upload-artifact: v3`

```yaml
✗ uses: actions/upload-artifact@v3
✓ uses: actions/upload-artifact@v4
```

همچنین `download-artifact` را هم به v4 ارتقا بده.

#### مثال ۱۲: `Error: Conflict: an artifact with this name already exists`

در matrix، نام artifact باید یکتا باشد:

```yaml
✗ name: coverage
✓ name: coverage-${{ matrix.node }}
```

#### مثال ۱۳: `Process completed with exit code 1` بدون جزئیات

```yaml
- run: npm test
  # فقط exit code 1
```

راه‌حل: `--verbose`، یا `continue-on-error: true` برای دیدن خروجی، یا `ACTIONS_STEP_DEBUG=true`.

#### مثال ۱۴: `denied: permission_denied: write_package`

```yaml
✗ permissions:
    contents: read
✓ permissions:
    contents: read
    packages: write
```

#### مثال ۱۵: `GraphQL: Resource not accessible by integration (addComment)`

در PR از fork، `GITHUB_TOKEN` فقط read است. راه‌حل:

```yaml
- if: github.event.pull_request.head.repo.fork == false
  uses: actions/github-script@<sha>
```

#### مثال ۱۶: `fatal: could not read Username for 'https://github.com'`

`actions/checkout` نبوده یا `persist-credentials: false` گذاشته‌ای و بعد `git push` می‌زنی.

#### مثال ۱۷: `The process '/usr/bin/git' failed with exit code 128`

معمولاً checkout با ref ناموجود:

```yaml
✗ ref: feature/nonexistent
```

یا submodule بدون token.

### مقایسه و الگوهای مختلف

#### `--log-failed` در برابر `--log`

| | failed | log |
|---|---|---|
| حجم | کم | زیاد |
| سرعت | ✅ | کند |
| مناسب | خطا | بررسی کامل |

#### `rerun --failed` در برابر `rerun`

| | failed | همه |
|---|---|---|
| سرعت | ✅ | کند |
| مناسب | خطای گذرا | تغییر تنظیمات |

#### `continue-on-error` در برابر حذف

| | continue | حذف |
|---|---|---|
| job سبز می‌شود | ✅ | — |
| اطلاعات | ✅ | ❌ |
| مناسب | تست غیرحیاتی | مرحله‌ی بی‌فایده |

### اشکال شایع

#### ۱. حدس زدن

`actionlint` + `gh run view --log-failed` = ۸۰٪ جواب.

#### ۲. `--no-verify` معادل در Actions

`continue-on-error: true` برای همه‌چیز = CI بی‌فایده.

#### ۳. required check با نام اشتباه

همیشه از `gh pr checks` کپی کن.

#### ۴. `permissions` ننوشتن

پیش‌فرض ممکن است کم یا زیاد باشد. صریح بنویس.

#### ۵. workflow در branch اشتباه

workflow باید در branch پیش‌فرض باشد برای بعضی eventها.

#### ۶. کش کهنه

`key` ثابت = وابستگی قدیمی. `hashFiles`.

#### ۷. `timeout` نبودن

job گیرکرده ۶ ساعت. `timeout-minutes`.

### نکات مهم (Tips & Tricks)

- **`actionlint` در pre-commit.**
- **`gh run view --log-failed`** اولین دستور.
- **`rerun --failed --debug`** تلاش دوم.
- **`ci-success` job** برای required پایدار.
- **`permissions:` صریح.**
- **`timeout-minutes` در هر job.**

### خلاصه فصل

- دسته‌ها: syntax، permission، trigger، check name، environment، timeout، secret.
- `actionlint` + `--log-failed` = تشخیص سریع.
- `ci-success` job ruleset را پایدار می‌کند.
- `permissions` صریح.
- `timeout-minutes` همیشه.

**سؤالات مروری:**
1. چرا `Expected — Waiting for status` رخ می‌دهد؟
2. چطور فقط job شکست‌خورده را با debug دوباره اجرا کنی؟
3. چرا نام artifact در matrix باید یکتا باشد؟

---

## فصل ۹۲: خطاهای حجم، کارایی و مخزن بزرگ

### مقدمه

مخزنی که ۲GB شده، clone در ۱۰ دقیقه، و `git status` در ۳۰ ثانیه. این فصل درباره‌ی تشخیص و رفع این مشکلات است.

**چرا مهم است؟** مخزن بزرگ، بهره‌وری تیم را می‌کشد و CI را کند می‌کند.

**کاربرد عملی:** از تشخیص تا کوچک‌سازی.

### مفاهیم اساسی

#### محدودیت‌های GitHub

| | محدودیت |
|---|---|
| فایل | ۱۰۰MB (خطا) |
| مخزن (توصیه) | ۱GB |
| مخزن (سخت) | ۵GB |
| push | ۲GB در یک بار |
| LFS | سهمیه |

#### منابع حجم

| منبع | معمول |
|---|---|
| `node_modules` | ۵۰-۵۰۰MB |
| فایل build (`dist`، `.next`) | ۱۰-۲۰۰MB |
| binary (تصویر، ویدیو) | متغیر |
| dataset | متغیر |
| تاریخ طولانی با فایل بزرگ | متغیر |

### مثال‌های عملی

#### مثال ۱: تشخیص حجم

```bash
# حجم .git
du -sh .git

# بزرگ‌ترین فایل‌های تاریخ
git rev-list --objects --all \
  | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' \
  | awk '/^blob/ {print $3, $4}' \
  | sort -rn \
  | head -20 \
  | numfmt --to=iec --field=1
```

#### مثال ۲: پیدا کردن فایل‌هایی که باید حذف شوند

```bash
# بزرگ‌ترین پوشه‌های working dir
du -sh * | sort -rh | head -10

# آیا در Git هستند؟
git ls-files | grep -E "node_modules|dist|\.next" | head

# اگر هستند، حذف از ردیابی:
git rm -r --cached node_modules dist .next
echo -e "node_modules/\ndist/\n.next/" >> .gitignore
git commit -m "chore: توقف ردیابی فایل‌های build"
```

#### مثال ۳: clone سریع (shallow)

```bash
git clone --depth 1 https://github.com/OWNER/BIG-REPO.git

# بعداً تاریخ کامل (اگر لازم شد)
git fetch --unshallow

# یا فقط چند commit
git clone --depth 50 ...
```

#### مثال ۴: clone تک‌branch

```bash
git clone --single-branch --branch main https://github.com/OWNER/BIG-REPO.git
```

#### مثال ۵: partial clone (blobless)

```bash
git clone --filter=blob:none https://github.com/OWNER/BIG-REPO.git
```

فایل‌ها فقط وقتی لازم شوند دانلود می‌شوند. برای مخزن بزرگ عالی است.

#### مثال ۶: sparse checkout

```bash
git clone --filter=blob:none --sparse https://github.com/OWNER/MONOREPO.git
cd MONOREPO
git sparse-checkout set packages/my-package
```

فقط پوشه‌ی مورد نظر. برای monorepo حیاتی.

#### مثال ۷: کوچک‌سازی تاریخ با `git filter-repo`

```bash
# نصب
pip install git-filter-repo

# پشتیبان (مهم!)
git clone --mirror git@github.com:OWNER/REPO.git repo-backup.git

# حذف یک فایل از کل تاریخ
cd REPO
git filter-repo --path path/to/big-file.zip --invert-paths

# حذف با الگو
git filter-repo --path-glob '*.psd' --invert-paths

# حذف فایل‌های بزرگ‌تر از ۱۰MB
git filter-repo --strip-blobs-bigger-than 10M

# force push
git push --force --all
git push --force --tags
```

**هشدارها:**

- تاریخ کاملاً بازنویسی می‌شود
- همه باید دوباره clone کنند
- forkها نسخه‌ی قدیمی دارند
- GitHub ممکن است نسخه‌ی قدیمی را در کش داشته باشد (support)

#### مثال ۸: `git gc` — فشرده‌سازی

```bash
git gc --aggressive --prune=now
# می‌تواند ساعت‌ها طول بکشد، ولی فضا آزاد می‌کند
```

#### مثال ۹: `git count-objects`

```bash
git count-objects -vH
# count: 0
# size: 0 bytes
# in-pack: 45231
# packs: 1
# size-pack: 245.32 MiB
# prune-packable: 0
# garbage: 0
# size-garbage: 0 bytes
```

#### مثال ۱۰: کند بودن `git status`

```bash
# ۱) تعداد فایل
git ls-files | wc -l

# ۲) untracked فایل‌های زیاد؟
git status --porcelain | wc -l

# ۳) .gitignore کامل؟
# ۴) fsmonitor (Git 2.37+):
git config core.fsmonitor true
git config core.untrackedCache true
```

#### مثال ۱۱: کند بودن checkout

```bash
# ۱) core.preloadIndex
git config core.preloadIndex true

# ۲) sparse checkout
git sparse-checkout init --cone
git sparse-checkout set src/

# ۳) partial clone
```

#### مثال ۱۲: فایل بیش از ۱۰۰MB

```
remote: error: GH001: Large files detected.
remote: error: File big.zip is 150.00 MB; this exceeds GitHub's file size limit of 100.00 MB
```

راه‌حل:

1. اگر commit **نشده**: از LFS استفاده کن یا فایل را حذف
2. اگر commit **شده** ولی push **نشده**: `git reset --soft HEAD~1`، حذف، دوباره commit
3. اگر push **شده**: `git filter-repo`

#### مثال ۱۳: مخزن به ۵GB رسیده

```
remote: error: GH002: Repository size limit exceeded
```

باید تاریخ را کوچک کنی. `git filter-repo` + هماهنگی با support.

#### مثال ۱۴: CI کند به‌خاطر checkout

```yaml
- uses: actions/checkout@<sha>
  with:
    fetch-depth: 1        # نه 0، اگر تاریخ کامل لازم نیست
    sparse-checkout: |
      packages/my-package
```

#### مثال ۱۵: بررسی حجم در CI

```yaml
- name: بررسی حجم مخزن
  run: |
    SIZE=$(du -sm .git | cut -f1)
    echo "حجم .git: ${SIZE}MB"
    if [ "$SIZE" -gt 1024 ]; then
      echo "::warning::مخزن بزرگ است (${SIZE}MB)"
    fi
```

### مقایسه و الگوهای مختلف

#### shallow در برابر partial در برابر sparse

| | shallow | partial (blobless) | sparse |
|---|---|---|---|
| چه چیزی کم می‌شود | تاریخ | محتوا | فایل‌ها |
| `git log` | محدود | ✅ کامل | ✅ |
| `git blame` | ❌ | ✅ (دانلود) | محدود |
| مناسب | CI | مخزن بزرگ | monorepo |

#### `filter-repo` در برابر `filter-branch`

| | filter-repo | filter-branch |
|---|---|---|
| سرعت | ✅✅ | کند |
| ایمنی | ✅ | ⚠️ |
| توصیه | ✅ | ❌ (deprecated) |

#### LFS در برابر حذف

| | LFS | حذف |
|---|---|---|
| فایل می‌ماند | ✅ | ❌ |
| حجم مخزن | کم | کم |
| سهمیه | محدود | — |
| مناسب | asset لازم | فایل قدیمی |

### اشکال شایع

#### ۱. `filter-repo` بدون پشتیبان

اگر اشتباه کنی، تاریخ رفته. `git clone --mirror` قبل از هر کاری.

#### ۲. force push بدون اطلاع تیم

همه باید دوباره clone کنند. اگر کسی push کند، تاریخ برمی‌گردد.

#### ۳. فراموش کردن forkها

forkها نسخه‌ی قدیمی دارند. باید حذف/بازسازی شوند.

#### ۴. `--depth 1` در CI که تاریخ لازم دارد

semantic-release، `git describe`، `git log` کامل — همه به تاریخ نیاز دارند.

#### ۵. `node_modules` در تاریخ

حتی اگر الان حذف شده باشد، در تاریخ هست. `filter-repo`.

#### ۶. `git gc --aggressive` زیاد

ساعت‌ها طول می‌کشد و فایده‌ی محدود. یک بار در سال کافی است.

#### ۷. `core.fsmonitor` در شبکه

روی فایل‌سیستم شبکه‌ای (NFS) ممکن است بدتر کند.

#### ۸. انتظار کوچک شدن فوری روی GitHub

بعد از force push، GitHub ممکن است نسخه‌ی قدیمی را نگه دارد. با support تماس بگیر.

### نکات مهم (Tips & Tricks)

- **`du -sh .git`** — تشخیص سریع.
- **`git rev-list --objects --all | ...`** — بزرگ‌ترین فایل‌ها.
- **`--filter=blob:none`** — clone سریع.
- **`git filter-repo`** — کوچک‌سازی (با پشتیبان).
- **`--depth 1` در CI** اگر تاریخ لازم نیست.
- **`.gitignore` از روز اول.**

### خلاصه فصل

- مخزن بزرگ = clone کند، CI کند، تیم کند.
- `shallow`، `partial`، `sparse` برای clone سریع.
- `git filter-repo` برای کوچک‌سازی (با پشتیبان و اطلاع تیم).
- فایل > ۱۰۰MB → LFS یا حذف.
- `.gitignore` از روز اول.

**سؤالات مروری:**
1. تفاوت shallow و partial clone چیست؟
2. چرا قبل از `filter-repo` باید پشتیبان گرفت؟
3. چرا `--depth 1` می‌تواند semantic-release را بشکند؟

---

## فصل ۹۳: بازیابی فاجعه

### مقدمه

«همه‌چیز را از دست دادم.» تقریباً همیشه غلط است. Git تقریباً هیچ‌وقت چیزی را فوراً پاک نمی‌کند. این فصل پروتکل بازیابی است.

**چرا مهم است؟** در لحظه‌ی بحران، کسی باید بداند چه کند. بدون پروتکل، آدم `reset --hard` می‌زند و وضع را بدتر می‌کند.

**کاربرد عملی:** این فصل را یک بار بخوان، و در بحران به آن برگرد.

### مفاهیم اساسی

#### پروتکل کلی

```
۱. دست نگه دار — هیچ دستور مخربی نزن
۲. وضعیت را ثبت کن (git status, git log, git reflog)
۳. یک پشتیبان بساز (cp -r یا git bundle)
۴. علت را تشخیص بده
۵. بازیابی کن
۶. تأیید کن
```

#### منابع بازیابی

| منبع | چه چیزی |
|---|---|
| `git reflog` | حرکت HEAD (۹۰ روز) |
| `git fsck --lost-found` | اشیاء orphan |
| `git stash list` | stashها |
| GitHub | commitهای push‌شده، PRها، releaseها |
| forkها | نسخه‌ی کامل |
| `ORIG_HEAD` | قبل از merge/rebase/reset |
| بکاپ محلی | اگر داشته باشی |

#### ابزار `git bundle` — پشتیبان کامل

```bash
# ساخت
git bundle create repo-backup.bundle --all

# بازیابی
git clone repo-backup.bundle repo-restored
```

یک فایل، کل تاریخ. عالی برای پشتیبان قبل از عملیات خطرناک.

### مثال‌های عملی

#### مثال ۱: `reset --hard` اشتباه

```bash
# ۱) دست نگه دار
git reflog
# a1b2c3d HEAD@{0}: reset: moving to HEAD~3
# e4f5g6h HEAD@{1}: commit: feat: کار مهم
# ...

# ۲) پشتیبان از وضعیت فعلی
git branch backup-$(date +%s)

# ۳) بازیابی
git reset --hard e4f5g6h

# ۴) تأیید
git log --oneline -3
```

#### مثال ۲: branch حذف‌شده

```bash
git branch -D feature/important
# Deleted branch feature/important (was e4f5g6h)

git switch -c feature/important e4f5g6h
```

اگر هش را یادت نیست:

```bash
git reflog --all --since="1 day ago" | grep feature
# یا
git fsck --lost-found | grep commit
```

#### مثال ۳: rebase خراب

```bash
git reset --hard ORIG_HEAD
# یا
git reflog
git reset --hard <hash-قبل-از-rebase>
```

#### مثال ۴: commit در detached HEAD

```bash
git switch --detach HEAD~5
# کار کن، commit بزن
git switch main
# commit گم شد

git reflog
# a1b2c3d HEAD@{3}: commit: کار من

git switch -c rescue a1b2c3d
```

#### مثال ۵: force push اشتباه (روی سرور)

```bash
# ۱) اگر commit را در reflog خودت داری:
git reflog
git push --force-with-lease origin <hash>:main

# ۲) اگر نداری، از GitHub Events API:
gh api repos/OWNER/REPO/events --jq '.[] | select(.type=="PushEvent") | {before: .payload.before, head: .payload.head}'

# ۳) اگر پیدا نشد: support.github.com (تا ۹۰ روز)
```

#### مثال ۶: فایل حذف‌شده

```bash
# حذف شده ولی commit نشده
git restore path/to/file

# حذف شده و commit شده
git restore --source=HEAD~1 path/to/file

# پیدا کردن commit که حذف کرد
git log --oneline --diff-filter=D -- path/to/file
```

#### مثال ۷: مخزن خراب (`fatal: bad object`)

```bash
# ۱) پشتیبان
cp -r . ../repo-broken-backup

# ۲) بررسی
git fsck --full

# ۳) اگر فقط چند شیء خراب است:
git fetch origin
git reset --hard origin/main

# ۴) اگر بدتر شد: clone جدید از GitHub
git clone git@github.com:OWNER/REPO.git repo-new
# و تغییرات محلی uncommitted را دستی منتقل کن
```

#### مثال ۸: `.git` پاک شده

اگر `.git` رفته، تاریخ رفته. ولی:

```bash
# اگر GitHub داری:
git clone git@github.com:OWNER/REPO.git
# و فایل‌های محلی را کپی کن

# اگر GitHub نداری ولی commitها push شده‌اند به remote دیگر:
git clone <remote>
```

**درس:** همیشه push کن. commit محلی = تنها نسخه.

#### مثال ۹: `index.lock` گیرکرده

```bash
# ۱) آیا Git در حال اجراست؟
ps aux | grep git

# ۲) اگر نه:
rm .git/index.lock

# ۳) اگر مشکل ادامه دارد:
git reset
```

#### مثال ۱۰: stash گم‌شده

```bash
git fsck --unreachable | grep commit
# یا
git log -g refs/stash
git stash list
```

#### مثال ۱۱: مخزن با تاریخ واگرا بعد از force push دیگران

```bash
git fetch
git log HEAD..origin/main --oneline     # جدید
git log origin/main..HEAD --oneline     # قدیمی (که دیگر روی سرور نیست)
git reset --hard origin/main
# یا اگر کارت مهم است:
git rebase origin/main
```

#### مثال ۱۲: پشتیبان قبل از عملیات خطرناک

```bash
# روش ۱: branch
git branch backup-$(date +%Y%m%d-%H%M)

# روش ۲: bundle
git bundle create /tmp/backup-$(date +%s).bundle --all

# روش ۳: کپی کامل
cp -r ~/projects/app /tmp/app-backup
```

#### مثال ۱۳: بازیابی از GitHub پس از فاجعه‌ی محلی

```bash
cd ~/projects
mv app app-broken
git clone git@github.com:OWNER/REPO.git app
cd app

# تغییرات uncommitted را از app-broken کپی کن
# (با دقت، فایل به فایل)
```

#### مثال ۱۴: مخزن با فایل‌های حساس در تاریخ

```bash
# ۱) فوراً کلید را revoke کن (نه فقط حذف)
# ۲) پشتیبان
git clone --mirror git@github.com:OWNER/REPO.git backup.git
# ۳) حذف از تاریخ
git filter-repo --path secrets.env --invert-paths
# ۴) force push
git push --force --all
# ۵) اطلاع به تیم: دوباره clone کنند
```

#### مثال ۱۵: چک‌لیست پیشگیری

```
□ push منظم (روزانه)
□ branch پشتیبان قبل از rebase/filter-repo
□ 2FA روی حساب GitHub
□ بکاپ محلی هفتگی (bundle)
□ GitHub Actions برای بکاپ خودکار
□ آموزش تیم: reflog را بشناسند
```

#### مثال ۱۶: بکاپ خودکار با Actions

```yaml
name: Backup

on:
  schedule:
    - cron: "0 2 * * 0"    # یکشنبه‌ها
  workflow_dispatch:

jobs:
  backup:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
        with:
          fetch-depth: 0
      - run: |
          git bundle create repo-$(date +%Y%m%d).bundle --all
      - uses: actions/upload-artifact@<sha>
        with:
          name: backup-${{ github.run_id }}
          path: repo-*.bundle
          retention-days: 90
```

### مقایسه و الگوهای مختلف

#### `reflog` در برابر `fsck`

| | reflog | fsck |
|---|---|---|
| چه چیزی | حرکت HEAD | اشیاء orphan |
| عمر | ۹۰ روز | تا gc |
| سرعت | ✅ | کند |
| مناسب | اول | دوم |

#### `reset` در برابر `restore` در بازیابی

| | reset | restore |
|---|---|---|
| branch | ✅ | ❌ |
| فایل | ✅ | ✅ |
| ایمنی | ⚠️ | ✅ |

#### branch پشتیبان در برابر bundle

| | branch | bundle |
|---|---|---|
| سرعت | ✅ فوری | ثانیه |
| انتقال | ❌ | ✅ فایل |
| کامل | ✅ | ✅ |

### اشکال شایع

#### ۱. زدن `reset --hard` در بحران

بدتر می‌کند. **اول reflog، بعد تصمیم.**

#### ۲. نداشتن پشتیبان قبل از `filter-repo`

اگر اشتباه کنی، برگشت سخت است. `git clone --mirror` همیشه.

#### ۳. force push روی مخزن بدون اطلاع

تاریخ دیگران را می‌شکند.

#### ۴. commit نکردن برای مدت طولانی

اگر لپ‌تاپ بسوزد، کار از دست می‌رود. commit محلی حداقل.

#### ۵. push نکردن

commit محلی = تنها نسخه. **push کن.**

#### ۶. انتظار بازیابی بعد از `gc`

اگر `git gc --prune=now` اجرا شده و reflog پاک شده، commitها رفته‌اند.

#### ۷. پاک کردن `.git` برای «شروع تازه»

تاریخ می‌رود. اگر لازم است، اول `git bundle`.

#### ۸. فراموش کردن fork

اگر مخزن fork شده، fork نسخه‌ی قدیمی دارد و می‌تواند منبع بازیابی باشد.

### نکات مهم (Tips & Tricks)

- **`git reflog`** اولین دستور در هر فاجعه.
- **`git bundle create --all`** پشتیبان کامل در یک فایل.
- **branch پشتیبان** قبل از هر rebase/filter.
- **`ORIG_HEAD`** میان‌بر لغو merge/rebase.
- **push منظم** — تنها راه واقعی محافظت.
- **در بحران، دست نگه دار** — هیچ دستور مخربی نزن.

### خلاصه فصل

- Git تقریباً هیچ‌وقت فوری پاک نمی‌کند.
- `reflog` شبکه‌ی نجات (۹۰ روز).
- `git bundle` پشتیبان کامل.
- در بحران: دست نگه دار، ثبت کن، پشتیبان بگیر، بازیابی کن.
- **push منظم** بهترین محافظت.
- `filter-repo` بدون پشتیبان = خطر.

**سؤالات مروری:**
1. اولین دستور در هر فاجعه چیست؟
2. چطور کل تاریخ را در یک فایل پشتیبان بگیری؟
3. چرا `reset --hard` در بحران بد است؟

---

**بعدی:** [`21-Complete-Project.md`](21-Complete-Project.md)

---

## منابع و مراجع

- GitHub Docs: Troubleshooting connectivity — https://docs.github.com/en/authentication/troubleshooting-ssh
- GitHub Docs: Troubleshooting workflows — https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows
- Pro Git، فصل ۷ (ابزارها) — https://git-scm.com/book/fa/v2
- `git filter-repo` — https://github.com/newren/git-filter-repo
- `git bundle` — https://git-scm.com/docs/git-bundle
- `git fsck` — https://git-scm.com/docs/git-fsck
- `actionlint` — https://github.com/rhysd/actionlint
- GitHub Status — https://www.githubstatus.com/
