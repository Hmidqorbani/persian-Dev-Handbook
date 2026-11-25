# بخش ۰۷: بهترین روش‌ها، Hooks و عیب‌یابی Git

**مدت یادگیری**: ۲.۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۰۶ (استراتژی‌های Branching)

---

## فهرست محتوا

- [فصل ۱: Conventional Commits](#فصل-۱-conventional-commits)
- [فصل ۲: Git Hooks](#فصل-۲-git-hooks)
- [فصل ۳: ابزارهای کیفیت کد](#فصل-۳-ابزارهای-کیفیت-کد)
- [فصل ۴: امنیت و امضای commit](#فصل-۴-امنیت-و-امضای-commit)
- [فصل ۵: عیب‌یابی خطاهای رایج](#فصل-۵-عیبیابی-خطاهای-رایج)
- [فصل ۶: بازیابی از فاجعه و چک‌لیست](#فصل-۶-بازیابی-از-فاجعه-و-چکلیست)

---

## فصل ۱: Conventional Commits

### مقدمه

پیام commit خوب، تاریخچه‌ای خوانا می‌سازد که می‌توانید با آن نسخه‌بندی خودکار و changelog بسازید. **Conventional Commits** یک استاندارد جهانی است.

### مفاهیم اساسی

ساختار:
```
<type>(<scope>): <description>

[body]

[footer]
```

**انواع اصلی:**

| نوع | کاربرد |
|-----|--------|
| `feat` | ویژگی جدید |
| `fix` | رفع باگ |
| `docs` | تغییر مستندات |
| `style` | فرمت‌دهی (بدون تغییر منطق) |
| `refactor` | بازنویسی کد |
| `perf` | بهبود کارایی |
| `test` | افزودن/اصلاح تست |
| `build` | تغییرات build/وابستگی |
| `ci` | تغییرات CI |
| `chore` | کارهای متفرقه |
| `revert` | بازگرداندن commit |

### مثال‌های عملی

```
feat(auth): add JWT token validation
fix(api): handle null response from payment gateway
docs(readme): update installation steps
refactor(utils): extract date formatting helper
ci(jenkins): add staging deployment stage

BREAKING CHANGE: the /api/v1/users endpoint now requires auth header
```

### مقایسه و الگوهای مختلف

| بد | خوب |
|----|-----|
| `update` | `feat(ui): add dark mode toggle` |
| `fix bug` | `fix(cart): correct total price calculation` |
| `asdf` | `chore: bump dependencies` |

### اشکال شایع

- **اشتباه رایج**: پیام‌های مبهم. همیشه «چه» و «چرا» را بنویسید.
- **اشتباه رایج**: استفاده از زمان گذشته (`added`) به‌جای امر (`add`).

### نکات مهم (Tips & Tricks)

- Conventional Commits پایه **Semantic Versioning** خودکار است: `feat` → minor، `fix` → patch، `BREAKING CHANGE` → major.
- ابزار `commitizen` پیام‌ها را راهنمایی می‌کند.

### خلاصه فصل

Conventional Commits ساختار `<type>(<scope>): <desc>` را تعریف می‌کند. این استاندارد نسخه‌بندی خودکار و changelog را ممکن می‌سازد.

---

## فصل ۲: Git Hooks

### مقدمه

**Git Hooks** اسکریپت‌هایی هستند که Git در نقاط مشخصی اجرا می‌کند (مثلاً قبل از commit). با آن‌ها می‌توانید کیفیت را خودکار تضمین کنید.

### مفاهیم اساسی

Hooks در `.git/hooks/` قرار دارند. مهم‌ترین‌ها:

**Client-side:**
- `pre-commit`: قبل از commit (lint، تست سریع).
- `commit-msg`: بررسی فرمت پیام.
- `pre-push`: قبل از push (تست کامل).
- `post-commit`: بعد از commit.

**Server-side:**
- `pre-receive`: قبل از پذیرش push.
- `update`: به‌ازای هر شاخه.
- `post-receive`: بعد از پذیرش (deploy، notification).

### مثال‌های عملی

`pre-commit` ساده:

```bash
#!/bin/sh
# .git/hooks/pre-commit
echo "Running linter..."
python -m flake8 . || {
  echo "Lint failed! Commit aborted."
  exit 1
}
```

`commit-msg` برای Conventional Commits:

```bash
#!/bin/sh
# .git/hooks/commit-msg
MSG=$(cat "$1")
PATTERN="^(feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert)(\(.+\))?: .+"
if ! echo "$MSG" | grep -qE "$PATTERN"; then
  echo "Invalid commit message format!"
  exit 1
fi
```

### مقایسه و الگوهای مختلف

| Hook | زمان اجرا | کاربرد |
|------|-----------|--------|
| pre-commit | قبل از commit | lint/test |
| commit-msg | هنگام commit | بررسی پیام |
| pre-push | قبل از push | تست کامل |
| post-receive | روی سرور | deploy |

### اشکال شایع

- **اشتباه رایج**: hooks در `.git/` هستند و **commit نمی‌شوند**. اعضای تیم آن‌ها را ندارند. راه‌حل: `core.hooksPath` یا ابزارهایی مثل Husky.

### نکات مهم (Tips & Tricks)

```bash
git config core.hooksPath .githooks
mkdir .githooks
# اسکریپت‌ها را اینجا بگذارید و commit کنید
```

- hooks باید **سریع** باشند؛ pre-commit طولانی، توسعه را کند می‌کند.

### خلاصه فصل

Hooks اسکریپت‌هایی در نقاط کلیدی Git هستند. برای lint، بررسی پیام و تست استفاده می‌شوند. با `core.hooksPath` می‌توانید آن‌ها را در تیم به اشتراک بگذارید.

---

## فصل ۳: ابزارهای کیفیت کد

### مقدمه

ترکیب Git با ابزارهای کیفیت کد، از ورود کد بی‌کیفیت جلوگیری می‌کند.

### مفاهیم اساسی

- **pre-commit** (فریم‌ورک پایتونی): مدیریت آسان hooks.
- **Husky** (Node.js): مدیریت hooks برای پروژه‌های JS.
- **commitlint**: بررسی پیام commit.
- **lint-staged**: اجرای linter فقط روی فایل‌های staged.
- **pre-commit-ci**: یکپارچگی با GitHub Actions.

### مثال‌های عملی

`.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
  - repo: https://github.com/psf/black
    rev: 24.1.0
    hooks:
      - id: black
  - repo: https://github.com/pycqa/flake8
    rev: 7.0.0
    hooks:
      - id: flake8
```

```bash
pip install pre-commit
pre-commit install
pre-commit run --all-files
```

### اشکال شایع

- **اشتباه رایج**: اجرای linter روی کل پروژه در هر commit. کند است. از lint-staged استفاده کنید.
- **اشتباه رایج**: نادیده گرفتن هشدارها با `--no-verify`.

### نکات مهم (Tips & Tricks)

- `git commit --no-verify` hooks را دور می‌زند؛ فقط در اضطرار استفاده کنید.
- pre-commit را در CI هم اجرا کنید تا مطمئن شوید همه hooks دارند.

### خلاصه فصل

ابزارهایی مثل pre-commit، Husky و commitlint کیفیت کد را خودکار تضمین می‌کنند. آن‌ها را در hooks محلی و CI هر دو اجرا کنید.

---

## فصل ۴: امنیت و امضای commit

### مقدمه

امنیت Git شامل محافظت از رمزها و اطمینان از اصالت commitهاست.

### مفاهیم اساسی

- **هرگز رمز/کلید را commit نکنید.**
- **`.gitignore`** اولین خط دفاعی است.
- **امضای GPG**: تأیید می‌کند commit از شماست.
- **`git-secrets` / `gitleaks`**: اسکن مخزن برای رمزها.

**راه‌اندازی امضای GPG:**
```bash
gpg --full-generate-key
gpg --list-secret-keys --keyid-format=long
git config --global user.signingkey <KEY_ID>
git config --global commit.gpgsign true
```

### مثال‌های عملی

اسکن رمزها:

```bash
# نصب gitleaks
gitleaks detect --source . --verbose

# یا با git-secrets
git secrets --install
git secrets --register-aws
git secrets --scan
```

اگر رمزی را اشتباهی commit کردید:

```bash
# ۱. رمز را فوراً باطل/تغییر دهید (مهم‌ترین قدم!)
# ۲. از تاریخچه حذف کنید (git filter-repo)
git filter-repo --path secrets.env --invert-paths
# ۳. force push (با هماهنگی تیم)
```

### اشکال شایع

- **اشتباه رایج**: فقط حذف رمز از commit جدید. تاریخچه هنوز آن را دارد.
- **اشتباه رایج**: فراموش کردن باطل کردن رمز لو رفته. تغییر تاریخچه کافی نیست؛ رمز باید باطل شود.

### نکات مهم (Tips & Tricks)

- از **pre-commit hook** برای اسکن رمز استفاده کنید.
- امضای GPG را اجباری کنید (GitHub نمایش «Verified» می‌دهد).
- به‌جای رمز در کد، از متغیرهای محیطی یا Secret Manager استفاده کنید.

### خلاصه فصل

امنیت Git یعنی محافظت از رمزها و تأیید اصالت. `.gitignore`، gitleaks و امضای GPG ابزارهای اصلی‌اند. اگر رمزی لو رفت، اول آن را باطل کنید.

---

## فصل ۵: عیب‌یابی خطاهای رایج

### مقدمه

در کار با Git حتماً با خطاهایی مواجه می‌شوید. این فصل مهم‌ترین‌ها را با راه‌حل بررسی می‌کند.

### مفاهیم اساسی و راه‌حل‌ها

**۱. Detached HEAD**
```
You are in 'detached HEAD' state.
```
علت: روی یک commit مستقیم هستید، نه شاخه.
راه‌حل:
```bash
git switch -c new-branch     # ساخت شاخه از اینجا
# یا
git switch main              # بازگشت
```

**۲. Push Rejected (non-fast-forward)**
```
! [rejected] main -> main (non-fast-forward)
```
علت: راه دور commitهایی دارد که شما ندارید.
راه‌حل:
```bash
git pull --rebase origin main
git push origin main
```

**۳. Merge Conflict**
```
CONFLICT (content): Merge conflict in file.txt
```
راه‌حل: (فصل ۴ بخش ۰۳) ویرایش، `add`، `commit`.

**۴. فراموشی commit در شاخه اشتباه**
```bash
git switch correct-branch
git cherry-pick <hash>       # انتقال commit
git switch wrong-branch
git reset --hard HEAD~1      # حذف از شاخه اشتباه
```

**۵. فایل بزرگ اشتباهاً commit شده**
```bash
git rm --cached bigfile.zip
echo "bigfile.zip" >> .gitignore
git commit --amend
```

**۶. خطای مجوز SSH**
```
Permission denied (publickey)
```
راه‌حل:
```bash
ssh-add ~/.ssh/id_ed25519
ssh -T git@github.com
```

**۷. اشتباه در پیام commit آخر**
```bash
git commit --amend -m "corrected message"
```

### اشکال شایع

- **اشتباه رایج**: استفاده از `--force` برای حل همه مشکلات push. خطرناک است.
- **اشتباه رایج**: حذف `.git` برای حل مشکل. همه تاریخچه از بین می‌رود.

### نکات مهم (Tips & Tricks)

- `git status` همیشه اولین قدم عیب‌یابی است.
- پیام خطای Git معمولاً راه‌حل را پیشنهاد می‌دهد؛ بخوانید!
- قبل از عملیات خطرناک، `git branch backup` بسازید.

### خلاصه فصل

بیشتر خطاهای Git راه‌حل ساده دارند: detached HEAD با `switch -c`، push rejected با `pull --rebase`، تعارض با ویرایش دستی. قبل از عملیات خطرناک پشتیبان بگیرید.

---

## فصل ۶: بازیابی از فاجعه و چک‌لیست

### مقدمه

گاهی اتفاق بدی می‌افتد: حذف شاخه، reset اشتباه، یا حتی خرابی مخزن. این فصل نشان می‌دهد چطور نجات پیدا کنید.

### مفاهیم اساسی و سناریوها

**حذف شاخه اشتباهی:**
```bash
git reflog
# پیدا کردن آخرین commit شاخه
git branch recovered-branch <hash>
```

**reset --hard اشتباه:**
```bash
git reflog
git reset --hard HEAD@{1}
```

**commit گمشده:**
```bash
git fsck --lost-found
# commitهای dangling را در .git/lost-found می‌یابید
```

**مخزن خراب:**
```bash
git fsck --full
git gc --aggressive
```

**بازگردانی فایل حذف‌شده:**
```bash
git checkout <commit> -- path/to/file
```

### نکات مهم (Tips & Tricks)

- **پشتیبان‌گیری منظم** از مخزن راه دور؛ هر clone یک نسخه کامل است.
- قبل از عملیات پرخطر: `git branch backup-$(date +%Y%m%d)`.
- `git reflog` تقریباً همیشه نجات‌دهنده است.

### چک‌لیست حرفه‌ای Git

- [ ] نام و ایمیل پیکربندی شده
- [ ] `.gitignore` مناسب
- [ ] پیام‌های Conventional Commits
- [ ] hooks کیفیت فعال
- [ ] امضای GPG (اختیاری)
- [ ] شاخه‌های feature کوتاه‌عمر
- [ ] قبل از push: `git pull --rebase`
- [ ] بازبینی PR قبل از merge
- [ ] حذف شاخه‌های merge‌شده
- [ ] پشتیبان‌گیری منظم

### خلاصه فصل

reflog، fsck و backup ابزارهای نجات‌اند. با پشتیبان‌گیری و احتیاط، می‌توانید از هر فاجعه‌ای بازیابی کنید. چک‌لیست حرفه‌ای را رعایت کنید.

---

## منابع و مراجع

- Conventional Commits: [https://www.conventionalcommits.org/](https://www.conventionalcommits.org/)
- Git Hooks: [https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)
- pre-commit: [https://pre-commit.com/](https://pre-commit.com/)
- gitleaks: [https://github.com/gitleaks/gitleaks](https://github.com/gitleaks/gitleaks)
- Git Security: [https://git-scm.com/book/en/v2/Git-Tools-Signing-Your-Work](https://git-scm.com/book/en/v2/Git-Tools-Signing-Your-Work)
