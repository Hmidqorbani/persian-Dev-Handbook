# بخش ۰۵: Git پیشرفته

**مدت یادگیری**: ۳.۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۰۴ (مخازن راه دور و همکاری)

---

## فهرست محتوا

- [فصل ۱: rebase و بازنویسی تاریخچه](#فصل-۱-rebase-و-بازنویسی-تاریخچه)
- [فصل ۲: Rebase تعاملی](#فصل-۲-rebase-تعاملی)
- [فصل ۳: cherry-pick](#فصل-۳-cherry-pick)
- [فصل ۴: stash](#فصل-۴-stash)
- [فصل ۵: reset و revert](#فصل-۵-reset-و-revert)
- [فصل ۶: reflog و نجات داده‌ها](#فصل-۶-reflog-و-نجات-دادهها)
- [فصل ۷: bisect، Tags و Submodules](#فصل-۷-bisect-tags-و-submodules)

---

## فصل ۱: rebase و بازنویسی تاریخچه

### مقدمه

**Rebase** یکی از قدرتمندترین و در عین حال خطرناک‌ترین دستورات Git است. تفاوت اصلی آن با merge در شکل تاریخچه است: rebase تاریخچه را **خطی و تمیز** می‌کند، در حالی که merge ساختار شاخه‌ای را حفظ می‌کند.

### مفاهیم اساسی

تصور کنید:

```
main:     A ← B ← C
                  ↑
feature:           D ← E
```

با merge:

```
main:     A ← B ← C ← M   (M = merge commit)
                  \     /
feature:           D ← E
```

با rebase:

```
main:     A ← B ← C
feature:              D' ← E'   (D و E بازنویسی شده‌اند)
```

rebase commitهای feature را برمی‌دارد و روی نوک main «بازنشانی» می‌کند. هش commitها **تغییر می‌کند** (D → D').

```bash
git switch feature
git rebase main
```

### مثال‌های عملی

```bash
# همگام‌سازی feature با main به‌روز
git switch feature
git fetch origin
git rebase origin/main

# اگر تعارض داد
# فایل‌ها را حل کنید، سپس:
git add .
git rebase --continue
# یا انصراف:
git rebase --abort
```

### مقایسه و الگوهای مختلف

| ویژگی | Merge | Rebase |
|-------|-------|--------|
| تاریخچه | شاخه‌ای | خطی |
| هش‌ها | حفظ می‌شود | تغییر می‌کند |
| ایمنی | بالا | خطر روی شاخه عمومی |
| خوانایی | گاهی شلوغ | تمیز |

**قاعده طلایی:** روی شاخه‌های **عمومی/مشترک** rebase نکنید. فقط روی شاخه‌های محلی یا feature خودتان.

### اشکال شایع

- **اشتباه رایج**: rebase کردن `main` که دیگران رویش کار می‌کنند. این کار تاریخچه مشترک را نابود می‌کند.
- **اشتباه رایج**: rebase بعد از push. نیاز به force push دارد که خطرناک است.

### نکات مهم (Tips & Tricks)

- قاعده: **rebase برای پاک‌سازی محلی، merge برای ادغام عمومی.**
- `git pull --rebase` جایگزین امن‌تری برای pull معمولی است.
- اگر اشتباه rebase کردید، با `git reflog` برمی‌گردید.

### خلاصه فصل

rebase تاریخچه را خطی می‌کند و commitها را روی نوک شاخه دیگر می‌نشاند. هش‌ها تغییر می‌کنند؛ پس روی شاخه‌های عمومی استفاده نکنید. برای تمیز کردن شاخه feature قبل از PR عالی است.

---

## فصل ۲: Rebase تعاملی

### مقدمه

**Interactive Rebase** به شما اجازه می‌دهد چند commit را بازنویسی، ترکیب، حذف یا مرتب کنید. این ابزار برای تمیز کردن تاریخچه قبل از PR بی‌نظیر است.

### مفاهیم اساسی

```bash
git rebase -i HEAD~3      # ۳ commit آخر
git rebase -i <hash>      # از یک commit به بعد
```

ادیتور باز می‌شود با لیست commitها و دستورات:

| دستور | معنی |
|-------|------|
| `pick` | استفاده از commit |
| `reword` | تغییر پیام |
| `edit` | توقف برای ویرایش |
| `squash` | ترکیب با commit قبلی |
| `fixup` | ترکیب بدون حفظ پیام |
| `drop` | حذف commit |
| `reorder` | با جابه‌جایی خطوط |

### مثال‌های عملی

```bash
git rebase -i HEAD~4
```

ادیتور باز می‌شود:
```
pick a1b2c3d feat: add login form
squash e4f5g6h fix typo
squash i7j8k9l fix typo again
reword m0n1o2p feat: add validation
```

نتیجه: دو commit اول در یک commit فشرده می‌شوند و پیام commit چهارم ویرایش می‌شود.

### مقایسه و الگوهای مختلف

| هدف | دستور |
|-----|-------|
| فشرده کردن commitها | squash / fixup |
| اصلاح پیام | reword |
| تقسیم یک commit | edit + reset |
| حذف commit | drop |

### اشکال شایع

- **اشتباه رایج**: حذف ناخواسته commit مهم. قبل از rebase یک شاخه پشتیبان بسازید.
- **اشتباه رایج**: squash کردن commitهایی که منطقاً جدا هستند.

### نکات مهم (Tips & Tricks)

- قبل از rebase تعاملی: `git branch backup-feature` بسازید.
- `git rebase -i --autosquash` با commitهای `fixup!` کار می‌کند.
- `git commit --fixup=<hash>` برای آماده‌سازی fixup استفاده کنید.

### خلاصه فصل

rebase تعاملی ابزار تمیزکاری تاریخچه است. با squash/fixup فشرده کنید، با reword پیام را اصلاح کنید و با drop حذف کنید. همیشه پشتیبان بگیرید.

---

## فصل ۳: cherry-pick

### مقدمه

گاهی فقط یک commit خاص از یک شاخه را می‌خواهید، نه همه آن. **cherry-pick** دقیقاً همین کار را می‌کند.

### مفاهیم اساسی

```bash
git cherry-pick <hash>
git cherry-pick <hash1> <hash2>
git cherry-pick <hash1>..<hash2>    # محدوده
```

cherry-pick یک کپی از commit را روی شاخه فعلی می‌سازد (هش جدید).

### مثال‌های عملی

```bash
# سناریو: باگ مهم در develop رفع شده، می‌خواهیم در main هم باشد
git switch main
git log develop --oneline
# فرض: abc123 fix: critical security bug

git cherry-pick abc123
# حالا commit روی main کپی شده است
```

### مقایسه و الگوهای مختلف

| روش | کاربرد |
|-----|--------|
| merge | ادغام کل شاخه |
| cherry-pick | یک commit خاص |
| rebase | بازنشانی کل شاخه |

### اشکال شایع

- **اشتباه رایج**: cherry-pick مکرر که تاریخچه را تکراری می‌کند. برای رفع باگ‌های hotfix مناسب است، نه برای توسعه معمول.
- **اشتباه رایج**: تعارض‌های مکرر به‌خاطر وابستگی commitها.

### نکات مهم (Tips & Tricks)

- برای hotfix روی شاخه release عالی است.
- `git cherry-pick -x <hash>` منبع اصلی را در پیام ثبت می‌کند.
- اگر تعارض داد: حل کنید، `git add` و `git cherry-pick --continue`.

### خلاصه فصل

cherry-pick یک commit خاص را روی شاخه فعلی کپی می‌کند. برای انتقال hotfix بین شاخه‌ها مناسب است. از استفاده بی‌رویه پرهیز کنید.

---

## فصل ۴: stash

### مقدمه

گاهی وسط کار باید سریع شاخه عوض کنید یا pull بزنید، اما کارتان نیمه‌تمام است. **stash** تغییرات را موقتاً کنار می‌گذارد.

### مفاهیم اساسی

```bash
git stash                       # ذخیره تغییرات tracked
git stash -u                    # شامل untracked
git stash -a                    # شامل ignored
git stash list                  # لیست stashها
git stash pop                   # بازیابی و حذف
git stash apply                 # بازیابی بدون حذف
git stash drop                  # حذف آخرین
git stash clear                 # حذف همه
```

### مثال‌های عملی

```bash
# وسط کار روی feature، باید hotfix بزنید
git stash push -m "work on login form"

git switch main
git switch -c hotfix/urgent
# ... رفع باگ و commit ...

git switch feature
git stash pop
```

### مقایسه و الگوهای مختلف

| دستور | اثر |
|-------|-----|
| `stash pop` | اعمال + حذف از لیست |
| `stash apply` | اعمال + نگه‌داشتن |
| `stash push -m` | ذخیره با پیام |

### اشکال شایع

- **اشتباه رایج**: فراموش کردن stashها. `git stash list` را مرتب چک کنید.
- **اشتباه رایج**: stash بدون پیام؛ بعداً نمی‌دانید کدام چیست.

### نکات مهم (Tips & Tricks)

- همیشه `-m "پیام"` بدهید.
- `git stash branch <name>` یک شاخه جدید از stash می‌سازد.
- `git stash show -p` محتوای stash را نشان می‌دهد.

### خلاصه فصل

stash تغییرات نیمه‌تمام را موقتاً ذخیره می‌کند. با `push -m` و `pop` مدیریتش کنید. مرتب لیست را بررسی کنید تا گم نشوند.

---

## فصل ۵: reset و revert

### مقدمه

دو دستور برای «برگرداندن» تغییرات: `reset` (بازنویسی تاریخچه محلی) و `revert` (ساخت commit معکوس). تفاوت آن‌ها حیاتی است.

### مفاهیم اساسی

**`git reset`** سه حالت دارد:

| حالت | HEAD | Staging | Working Dir |
|------|------|---------|-------------|
| `--soft` | جابه‌جا | حفظ | حفظ |
| `--mixed` (پیش‌فرض) | جابه‌جا | پاک | حفظ |
| `--hard` | جابه‌جا | پاک | پاک |

```bash
git reset --soft HEAD~1    # undo commit، تغییرات در Staging
git reset HEAD~1           # undo commit، تغییرات در Working Dir
git reset --hard HEAD~1    # حذف کامل commit و تغییرات ⚠️
```

**`git revert`** یک commit معکوس می‌سازد (ایمن برای شاخه عمومی):
```bash
git revert <hash>
git revert HEAD
```

### مثال‌های عملی

```bash
# اشتباه در پیام آخرین commit
git commit --amend -m "پیام درست"

# undo commit آخر اما نگه‌داشتن تغییرات
git reset --soft HEAD~1

# برگرداندن یک commit که قبلاً push شده (ایمن)
git revert abc123
git push
```

### مقایسه و الگوهای مختلف

| معیار | reset | revert |
|-------|-------|--------|
| تاریخچه | بازنویسی | حفظ + commit جدید |
| مناسب برای | commit محلی | commit عمومی |
| خطر | `--hard` داده از دست می‌دهد | کم |

### اشکال شایع

- **اشتباه رایج**: `git reset --hard` بدون فکر. تغییرات uncommitted نابود می‌شوند.
- **اشتباه رایج**: reset روی شاخه push شده. از revert استفاده کنید.

### نکات مهم (Tips & Tricks)

- قاعده: **reset برای محلی، revert برای عمومی.**
- قبل از `--hard` مطمئن شوید کارتان commit یا stash شده.
- اگر `--hard` اشتباه زدید، `git reflog` نجاتتان می‌دهد.

### خلاصه فصل

reset تاریخچه محلی را بازنویسی می‌کند (soft/mixed/hard). revert یک commit معکوس می‌سازد و برای شاخه‌های عمومی ایمن است. با `--hard` بسیار محتاط باشید.

---

## فصل ۶: reflog و نجات داده‌ها

### مقدمه

**reflog** دفترچه‌ای است که هر حرکت `HEAD` را ثبت می‌کند. اگر commit یا شاخه‌ای را گم کردید، reflog نجات‌دهنده است.

### مفاهیم اساسی

```bash
git reflog
# خروجی نمونه:
# abc1234 HEAD@{0}: reset: moving to HEAD~1
# def5678 HEAD@{1}: commit: feat: add feature
# ...
```

هر ورودی نشان می‌دهد `HEAD` کجا بوده. می‌توانید به هر نقطه برگردید.

### مثال‌های عملی

```bash
# فرض: با git reset --hard یک commit مهم را پاک کردید
git reflog
# def5678 HEAD@{1}: commit: feat: important work

# بازگردانی
git reset --hard def5678
# یا ساخت شاخه از آن
git branch recovered def5678
```

### اشکال شایع

- **اشتباه رایج**: فکر کردن داده برای همیشه از دست رفته. reflog معمولاً نجات می‌دهد.
- **اشتباه رایج**: نادیده گرفتن reflog در عیب‌یابی.

### نکات مهم (Tips & Tricks)

- reflog به‌طور پیش‌فرض ۹۰ روز نگه‌داری می‌کند.
- `git reflog show <branch>` تاریخچه یک شاخه خاص.
- برای commitهای گمشده (dangling): `git fsck --lost-found`.

### خلاصه فصل

reflog تاریخچه حرکت‌های HEAD را نگه می‌دارد و ابزار نجات داده است. تقریباً همیشه می‌توانید کار ازدست‌رفته را با reflog بازیابی کنید.

---

## فصل ۷: bisect، Tags و Submodules

### مقدمه

سه ابزار پیشرفته که در موقعیت‌های خاص بسیار مفیدند: `bisect` برای یافتن باگ، `tag` برای نسخه‌بندی و `submodule` برای پروژه‌های وابسته.

### مفاهیم اساسی

**bisect** — جست‌وجوی دودویی برای یافتن commit معیوب:
```bash
git bisect start
git bisect bad                  # نسخه فعلی خراب است
git bisect good v1.0            # این نسخه سالم بود
# Git وسط را checkout می‌کند؛ تست کنید و بگویید good یا bad
git bisect good   # یا
git bisect bad
# ... تکرار تا یافتن commit
git bisect reset
```

**Tags** — نشانه‌گذاری نسخه‌ها:
```bash
git tag v1.0.0                          # سبک
git tag -a v1.0.0 -m "Release 1.0"      # annotated (توصیه‌شده)
git tag                                 # لیست
git push origin v1.0.0                  # ارسال
git push origin --tags                  # همه
git tag -d v1.0.0                       # حذف محلی
```

**Submodules** — مخزن درون مخزن:
```bash
git submodule add <url> path
git submodule update --init --recursive
git clone --recurse-submodules <url>
```

**Git LFS** — برای فایل‌های بزرگ:
```bash
git lfs install
git lfs track "*.psd"
git add .gitattributes
```

### مثال‌های عملی

یافتن باگ با bisect:

```bash
git bisect start
git bisect bad HEAD
git bisect good v2.0
# Git: Bisecting: 5 revisions left
python -m pytest        # تست
git bisect good         # یا bad
# تا پیدا شدن commit معیوب
```

### مقایسه و الگوهای مختلف

| ابزار | کاربرد |
|-------|--------|
| bisect | یافتن commit معیوب |
| tag | نسخه‌بندی/Release |
| submodule | وابستگی مخزن خارجی |
| LFS | فایل‌های بزرگ |

### اشکال شایع

- **اشتباه رایج**: فراموش کردن `git bisect reset` و گیر افتادن در حالت bisect.
- **اشتباه رایج**: استفاده از submodule بدون درک پیچیدگی‌های آن. اغلب پکیج‌منیجرها بهترند.
- **اشتباه رایج**: commit کردن فایل بزرگ بدون LFS.

### نکات مهم (Tips & Tricks)

- `git bisect run <script>` خودکار باگ را پیدا می‌کند.
- tagهای annotated تاریخ، پیام و امضای GPG دارند.
- برای فایل‌های بزرگ از LFS استفاده کنید تا مخزن سبک بماند.

### خلاصه فصل

bisect با جست‌وجوی دودویی commit معیوب را می‌یابد. tagها نسخه‌ها را نشانه‌گذاری می‌کنند. submodule و LFS برای پروژه‌های خاص کاربرد دارند.

---

## منابع و مراجع

- Rebase: [https://git-scm.com/docs/git-rebase](https://git-scm.com/docs/git-rebase)
- Cherry-pick: [https://git-scm.com/docs/git-cherry-pick](https://git-scm.com/docs/git-cherry-pick)
- Reset/Revert: [https://git-scm.com/docs/git-reset](https://git-scm.com/docs/git-reset)
- Reflog: [https://git-scm.com/docs/git-reflog](https://git-scm.com/docs/git-reflog)
- Bisect: [https://git-scm.com/docs/git-bisect](https://git-scm.com/docs/git-bisect)
- Git LFS: [https://git-lfs.github.com/](https://git-lfs.github.com/)
