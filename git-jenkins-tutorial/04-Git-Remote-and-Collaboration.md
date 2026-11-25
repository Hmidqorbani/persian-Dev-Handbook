# بخش ۰۴: مخازن راه دور و همکاری

**مدت یادگیری**: ۳ ساعت
**سطح سختی**: مبتدی
**پیش‌نیاز**: بخش ۰۳ (شاخه‌بندی و ادغام)

---

## فهرست محتوا

- [فصل ۱: مخازن راه دور](#فصل-۱-مخازن-راه-دور)
- [فصل ۲: fetch، pull و push](#فصل-۲-fetch-pull-و-push)
- [فصل ۳: کار با GitHub](#فصل-۳-کار-با-github)
- [فصل ۴: Pull Request و بازبینی کد](#فصل-۴-pull-request-و-بازبینی-کد)
- [فصل ۵: Fork و مدل مشارکت](#فصل-۵-fork-و-مدل-مشارکت)
- [فصل ۶: upstream و مدیریت شاخه‌های راه دور](#فصل-۶-upstream-و-مدیریت-شاخههای-راه-دور)

---

## فصل ۱: مخازن راه دور

### مقدمه

تا حالا همه‌چیز محلی بود. اما همکاری تیمی نیازمند یک نقطه مشترک است: **مخزن راه دور (Remote Repository)**. این مخزن روی سروری مثل GitHub قرار دارد و همه اعضای تیم تغییرات خود را با آن همگام می‌کنند.

### مفاهیم اساسی

**`git remote`**: مدیریت مخازن راه دور.
```bash
git remote -v                      # لیست مخازن راه دور
git remote add origin <url>        # افزودن مخزن
git remote remove origin           # حذف
git remote rename origin upstream  # تغییر نام
git remote show origin             # جزئیات
```

**`origin`**: نام پیش‌فرض مخزن راه دور اصلی. نامی قراردادی است، نه جادویی.

**آدرس‌های اتصال:**
- HTTPS: `https://github.com/user/repo.git` (نیاز به توکن/رمز)
- SSH: `git@github.com:user/repo.git` (نیاز به کلید SSH)

### مثال‌های عملی

```bash
# ساخت مخزن محلی
git init
git add .
git commit -m "chore: initial commit"

# اتصال به GitHub
git remote add origin git@github.com:username/my-project.git
git remote -v

# ارسال اولین بار
git push -u origin main
```

### اشکال شایع

- **اشتباه رایج**: اشتباه گرفتن نام remote با URL. `origin` فقط یک برچسب است.
- **اشتباه رایج**: نداشتن کلید SSH و وارد کردن مکرر رمز.

### نکات مهم (Tips & Tricks)

- برای چند remote (مثلاً fork)، نام‌های معنادار بگذارید: `origin` و `upstream`.
- با `git remote set-url origin <new-url>` آدرس را تغییر دهید.

### خلاصه فصل

مخزن راه دور نقطه هماهنگی تیم است. `origin` نام پیش‌فرض آن است. با `git remote` آن‌ها را مدیریت می‌کنید.

---

## فصل ۲: fetch، pull و push

### مقدمه

سه دستور اصلی همگام‌سازی: `fetch`، `pull` و `push`. تفاوت آن‌ها برای جلوگیری از سردرگمی حیاتی است.

### مفاهیم اساسی

- **`git fetch`**: تغییرات راه دور را می‌گیرد اما **اعمال نمی‌کند**. فقط شاخه‌های راه دور (`origin/main`) را به‌روز می‌کند.
- **`git pull`**: `fetch` + `merge` (یا rebase). تغییرات را می‌گیرد و در شاخه محلی ادغام می‌کند.
- **`git push`**: commitهای محلی را به راه دور می‌فرستد.

```
راه دور:  A ← B ← C ← D
محلی:     A ← B ← C ← E

git fetch  → شاخه origin/main به D می‌رسد، اما main محلی همان E است.
git pull   → E با D ادغام می‌شود.
git push   → E به راه دور می‌رود (اگر D ادغام شده باشد).
```

### مثال‌های عملی

```bash
# گرفتن بدون اعمال
git fetch origin
git log HEAD..origin/main --oneline   # دیدن تغییرات جدید

# اعمال
git pull origin main

# pull با rebase
git pull --rebase origin main

# ارسال
git push origin main
git push origin feature/login         # شاخه دیگر
git push -u origin feature/login      # تنظیم upstream
```

### مقایسه و الگوهای مختلف

| دستور | اثر | ایمنی |
|-------|-----|-------|
| `fetch` | فقط دانلود | کاملاً ایمن |
| `pull` | دانلود + ادغام | ممکن است تعارض بدهد |
| `push` | ارسال | ممکن است رد شود |

### اشکال شایع

- **اشتباه رایج**: `git push -f` روی شاخه مشترک. این کار تاریخچه دیگران را خراب می‌کند.
- **اشتباه رایج**: `pull` بدون commit کردن کارها. تعارض می‌گیرید.
- **اشتباه رایج**: عدم fetch قبل از شروع کار.

### نکات مهم (Tips & Tricks)

- عادت کنید صبح‌ها اول `git fetch` بزنید.
- `git config --global pull.rebase true` تا pullها خطی بمانند.
- اگر push رد شد، اول `pull --rebase` و بعد push.

### خلاصه فصل

`fetch` فقط می‌گیرد، `pull` می‌گیرد و ادغام می‌کند، `push` می‌فرستد. تفاوت این سه را بدانید تا از سردرگمی و از دست رفتن کار جلوگیری کنید.

---

## فصل ۳: کار با GitHub

### مقدمه

GitHub محبوب‌ترین میزبان Git است. در این فصل کارهای اصلی را یاد می‌گیریم.

### مفاهیم اساسی

- **ساخت مخزن**: از رابط وب یا با `gh` (GitHub CLI).
- **کلید SSH**: برای اتصال امن بدون رمز.
- **توکن (PAT)**: برای HTTPS و CI/CD.
- **`.gitignore` و README**: استاندارد هر مخزن.

**ساخت کلید SSH:**
```bash
ssh-keygen -t ed25519 -C "you@example.com"
cat ~/.ssh/id_ed25519.pub
# محتوا را در GitHub → Settings → SSH Keys اضافه کنید
ssh -T git@github.com   # تست
```

**GitHub CLI:**
```bash
gh auth login
gh repo create my-project --public --source=. --push
```

### مثال‌های عملی

```bash
# کلون با SSH
git clone git@github.com:username/repo.git

# ساخت توکن و استفاده در HTTPS
git remote set-url origin https://<TOKEN>@github.com/username/repo.git
```

### اشکال شایع

- **اشتباه رایج**: commit کردن توکن یا کلید خصوصی. هرگز این کار را نکنید.
- **اشتباه رایج**: استفاده از رمز عبور به‌جای PAT. GitHub دیگر رمز عبور را برای Git نمی‌پذیرد.

### نکات مهم (Tips & Tricks)

- از SSH استفاده کنید تا امن و بدون رمز کار کنید.
- توکن‌ها را با حداقل دسترسی (scope) بسازید.
- برای CI/CD از **Deploy Key** یا **GitHub App** استفاده کنید.

### خلاصه فصل

GitHub مخزن راه دور شماست. با SSH یا PAT به آن وصل می‌شوید. هرگز رمزها را commit نکنید.

---

## فصل ۴: Pull Request و بازبینی کد

### مقدمه

**Pull Request (PR)** قلب همکاری در GitHub است. یک PR پیشنهاد ادغام یک شاخه در شاخه دیگر است که امکان بحث، بازبینی و تست خودکار را فراهم می‌کند.

### مفاهیم اساسی

**چرخه یک PR:**
1. ساخت شاخه: `git switch -c feature/x`.
2. commit و push.
3. باز کردن PR در GitHub.
4. بازبینی توسط همکاران.
5. اجرای CI (تست‌های خودکار).
6. ادغام (Merge).
7. حذف شاخه.

**اجزای PR:** عنوان، توضیحات، Reviewers، Labels، Linked Issues، Checks.

### مثال‌های عملی

```bash
git switch -c feature/user-profile
# ... کار و commit ...
git push -u origin feature/user-profile
# حالا در GitHub دکمه "Compare & pull request" ظاهر می‌شود
```

با GitHub CLI:
```bash
gh pr create --title "Add user profile" --body "Closes #42"
gh pr list
gh pr view 1
gh pr merge 1 --squash
```

### اشکال شایع

- **اشتباه رایج**: PRهای بزرگ با صدها تغییر. PRهای کوچک سریع‌تر بازبینی می‌شوند.
- **اشتباه رایج**: توضیحات مبهم. بنویسید «چه» و «چرا».
- **اشتباه رایج**: نادیده گرفتن شکست CI.

### نکات مهم (Tips & Tricks)

- هر PR را به یک Issue مرتبط کنید: `Closes #42`.
- از قالب PR (`.github/pull_request_template.md`) استفاده کنید.
- قبل از درخواست بازبینی، خودتان تغییرات را مرور کنید.

### خلاصه فصل

PR مکانیزم همکاری GitHub است. شاخه بسازید، push کنید، PR باز کنید، بازبینی و CI را رد کنید و merge کنید. PRهای کوچک و واضح بهترین‌اند.

---

## فصل ۵: Fork و مدل مشارکت

### مقدمه

در پروژه‌های متن‌باز، شما دسترسی نوشتن به مخزن اصلی ندارید. راه‌حل: **Fork**.

### مفاهیم اساسی

**Fork**: کپی مخزن دیگری در حساب شما. سپس از fork خود clone می‌کنید، شاخه می‌سازید و PR می‌فرستید.

```
مخزن اصلی (upstream)
      ↓ Fork
مخزن شما (origin)
      ↓ clone
مخزن محلی
      ↓ PR
مخزن اصلی
```

**دو remote:** `origin` (fork شما) و `upstream` (مخزن اصلی).

### مثال‌های عملی

```bash
# clone از fork خودتان
git clone git@github.com:yourname/repo.git
cd repo

# افزودن upstream
git remote add upstream git@github.com:original/repo.git
git remote -v

# همگام‌سازی با upstream
git fetch upstream
git switch main
git merge upstream/main
git push origin main
```

### اشکال شایع

- **اشتباه رایج**: فراموش کردن همگام‌سازی fork با upstream. fork شما قدیمی می‌شود.
- **اشتباه رایج**: ارسال PR از شاخه main خودتان. همیشه شاخه feature بسازید.

### نکات مهم (Tips & Tricks)

- هر بار قبل از شروع کار، upstream را fetch و merge کنید.
- یک شاخه به‌ازای هر PR بسازید.
- شاخه‌های feature را در fork خود حذف کنید تا تمیز بماند.

### خلاصه فصل

Fork کپی مخزن در حساب شماست. با دو remote (`origin` و `upstream`) کار می‌کنید و از طریق PR به مخزن اصلی مشارکت می‌کنید.

---

## فصل ۶: upstream و مدیریت شاخه‌های راه دور

### مقدمه

برای کار روان، باید بدانید هر شاخه محلی به کدام شاخه راه دور متصل است (upstream/tracking branch).

### مفاهیم اساسی

```bash
git branch -vv                    # نمایش upstream هر شاخه
git branch -u origin/main main    # تنظیم upstream
git push -u origin feature        # تنظیم هنگام push
git branch -r                     # شاخه‌های راه دور
git branch -a                     # همه شاخه‌ها
```

**حذف شاخه راه دور:**
```bash
git push origin --delete feature/old
git fetch --prune                 # حذف ارجاعات شاخه‌های حذف‌شده
```

### مثال‌های عملی

```bash
# بعد از merge یک PR در GitHub، شاخه محلی و راه دور را پاک کنید
git switch main
git pull
git branch -d feature/done
git push origin --delete feature/done
```

### اشکال شایع

- **اشتباه رایج**: انبوه شاخه‌های قدیمی. مرتباً prune کنید.
- **اشتباه رایج**: نداشتن upstream و زدن مکرر `git push origin <branch>`.

### نکات مهم (Tips & Tricks)

- `git config --global fetch.prune true` تا حذف‌ها خودکار همگام شوند.
- `git remote prune origin` ارجاعات کهنه را پاک می‌کند.

### خلاصه فصل

upstream ارتباط شاخه محلی با راه دور را مشخص می‌کند. با `git branch -vv` وضعیت را ببینید و شاخه‌های قدیمی را مرتباً پاک کنید.

---

## منابع و مراجع

- مستندات Remote: [https://git-scm.com/docs/git-remote](https://git-scm.com/docs/git-remote)
- راهنمای Pull Request: [https://docs.github.com/en/pull-requests](https://docs.github.com/en/pull-requests)
- GitHub Flow: [https://docs.github.com/en/get-started/using-github/github-flow](https://docs.github.com/en/get-started/using-github/github-flow)
- راهنمای Fork: [https://docs.github.com/en/get-started/quickstart/fork-a-repo](https://docs.github.com/en/get-started/quickstart/fork-a-repo)
- GitHub CLI: [https://cli.github.com/](https://cli.github.com/)
