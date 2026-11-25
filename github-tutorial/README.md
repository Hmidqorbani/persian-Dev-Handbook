# GitHub از صفر تا تولید

**راهنمای جامع از Git و همکاری تا Copilot، Actions و امنیت**

---

## این کتابچه برای کیست؟

اگر تا حالا با Git کار نکرده‌ای و فقط شنیده‌ای «کدت را push کن»، این کتابچه از همان‌جا شروع می‌کند. اگر هم چند سال است هر روز `git commit` می‌زنی ولی هیچ‌وقت وقت نکرده‌ای سراغ Actions، Rulesets، CodeQL و Copilot Agent بروی، فصل‌های میانی و انتهایی همان چیزی است که دنبالش هستی.

هیچ پیش‌نیاز سنگینی ندارد: فقط باید بدانی خط فرمان چیست و یک زبان برنامه‌نویسی را در حد «می‌توانم یک فایل بنویسم و اجرا کنم» بشناسی.

---

## چه چیزی یاد می‌گیری؟

- **Git از ریشه:** مدل snapshot، سه ناحیه، branch به‌عنوان اشاره‌گر، merge و rebase و اینکه چرا conflict پیش می‌آید.
- **همکاری واقعی:** fork، upstream، Pull Request، Code Review، CODEOWNERS، Merge Queue.
- **محافظت از شاخه:** Branch Protection و Rulesets نسل جدید، required checks، bypass.
- **GitHub Actions:** از یک workflow ساده تا reusable workflow، matrix، caching، OIDC و زنجیره‌ی تأمین امن.
- **Copilot:** Ask / Edit / Agent Mode، Cloud Agent، MCP، Custom Instructions و Code Review.
- **امنیت:** Dependabot، CodeQL، Secret Scanning + Push Protection، SBOM و Artifact Attestations.
- **اکوسیستم:** Projects، Packages، Pages، Codespaces، GitHub CLI.
- **تولید:** یک مخزن کامل با CI/CD، امنیت، مستندات و انتشار — از صفر تا merge نهایی.

---

## ساختار کتابچه

```
github-tutorial/
├── README.md                            ← همین فایل
├── SUMMARY.md                           ← فهرست کامل ۱۰۱ فصل با لینک
├── 01-Introduction-and-Ecosystem.md     ← Git و GitHub چیست، اکوسیستم، حساب
├── 02-Git-Fundamentals.md               ← نصب، init، commit، ignore، تاریخ
├── 03-Branching-and-Merging.md          ← branch، merge، rebase، استراتژی‌ها
├── 04-Remote-and-Collaboration.md       ← remote، fork، push، اولین PR
├── 05-History-Rewriting-and-Recovery.md ← amend، reset، revert، reflog، bisect
├── 06-Repository-Setup-and-Files.md     ← README، LICENSE، قالب‌ها، Markdown
├── 07-Issues-Labels-and-Projects.md     ← Issue، label، milestone، Projects
├── 08-Pull-Requests-and-Review.md       ← چرخه‌ی PR، review، merge queue
├── 09-Branch-Protection-and-Rulesets.md ← محافظت از شاخه و تگ
├── 10-Actions-Basics.md                 ← workflow، job، step، matrix، debug
├── 11-Actions-Advanced.md               ← cache، artifact، reusable، runner
├── 12-Actions-Security.md               ← permissions، OIDC، injection، pinning
├── 13-CI-CD-Patterns.md                 ← خط CI، Docker، deploy، انتشار
├── 14-Releases-Versioning-and-Packages.md ← tag، SemVer، Packages
├── 15-Copilot.md                        ← Ask/Edit/Agent، Cloud Agent، MCP
├── 16-Security-Features.md              ← Dependabot، CodeQL، Secret Scanning
├── 17-GitHub-CLI.md                     ← gh در ترمینال
├── 18-Pages-Codespaces-and-More.md      ← Pages، Codespaces، Discussions، Gist
├── 19-Advanced-Git.md                   ← worktree، submodule، LFS، hooks، signing
├── 20-Troubleshooting.md                ← خطاهای رایج و راه‌حل
├── 21-Complete-Project.md               ← پروژه‌ی کامل end-to-end
└── 22-Checklist-and-Glossary.md         ← چک‌لیست، واژه‌نامه، نقشه راه
```

---

## مسیرهای پیشنهادی مطالعه

### مسیر ۱ — «فقط می‌خواهم کدم را درست مدیریت کنم» (۱۲ تا ۱۵ ساعت)

```
01 → 02 → 03 → 04 → 05 → 06 → 20
```

هدف: Git را در سطح روزمره و بدون ترس بلد شوی؛ بتوانی conflict حل کنی، commit اشتباه را برگردانی، و از دست دادن کار برایت اتفاق نیفتد.

### مسیر ۲ — «می‌خواهم در یک تیم کار کنم» (۲۵ تا ۳۰ ساعت)

```
01 → 02 → 03 → 04 → 06 → 07 → 08 → 09 → 17
```

هدف: Pull Request، Code Review، Issue، Projects و قواعد محافظت از شاخه را در سطح تیم حرفه‌ای بلد شوی.

### مسیر ۳ — «می‌خواهم CI/CD و امنیت راه بیندازم» (۳۵ تا ۴۵ ساعت)

```
02 → 03 → 04 → 08 → 09 → 10 → 11 → 12 → 13 → 14 → 16 → 21
```

هدف: خط لوله‌ی تست و استقرار، اسکن امنیتی خودکار، انتشار نسخه و مدیریت وابستگی‌ها — همان چیزی که از یک مخزن production انتظار می‌رود.

### مسیر ۴ — کامل (۴۵ تا ۶۰ ساعت)

همه‌ی ۲۲ فایل، به ترتیب. برای تدریس یا مرجع دائمی.

---

## قراردادهای این کتابچه

- **سطح‌بندی:** هر فایل با `مدت یادگیری`، `سطح سختی` و `پیش‌نیاز` شروع می‌شود.
- **ساختار هر فصل ثابت است:** مقدمه → مفاهیم اساسی → مثال‌های عملی → مقایسه و الگوهای مختلف → اشکال شایع → نکات مهم → خلاصه فصل.
- **شماره‌ی فصل‌ها سراسری است:** فصل ۱ تا ۱۰۱، پشت سر هم در کل کتابچه — نه از نو در هر فایل.
- **همه‌ی دستورها قابل اجرا هستند.** هر جا دستور یا فایل کامل نوشته شده، فرض بر این است که می‌توانی کپی کنی و اجرا کنی.
- **مثال‌ها با نام‌های ایرانی و سناریوهای واقعی** (فروشگاه، تیم پشتیبانی، پروژه‌ی دانشگاهی) نوشته شده‌اند تا ملموس باشند.
- **نسخه‌ها:** محتوا بر مبنای وضعیت GitHub در سال ۲۰۲۶ نوشته شده. جایی که قابلیتی سریع تغییر می‌کند (مثل Copilot، MCP، Agent)، صریح هشدار داده شده که «مستندات همان روز را چک کن».

---

## چطور از این کتابچه استفاده کنم؟

**اگر تازه‌کاری:** از `01-Introduction-and-Ecosystem.md` شروع کن و به ترتیب برو. هر فصل را که خواندی، همان دستورها را در یک پوشه‌ی خالی روی سیستم خودت اجرا کن. Git با خواندن یاد گرفته نمی‌شود؛ با خراب کردن و درست کردن یاد گرفته می‌شود.

**اگر وقت کم داری:** `SUMMARY.md` را باز کن، فصل‌های مربوط به مشکل امروزت را پیدا کن و همان‌ها را بخوان. هر فصل مستقل قابل خواندن است.

**اگر می‌خواهی تدریس کنی:** هر فایل یک جلسه‌ی کلاس است. بخش «مثال‌های عملی» را به‌عنوان تمرین سر کلاس بده و بخش «اشکال شایع» را به‌عنوان «تله‌های امتحان» استفاده کن.

---

## پیشنهاد: یک پوشه‌ی تمرین بساز

قبل از فصل ۲، این کار را بکن:

```bash
# ویندوز (Git Bash) / مک / لینوکس
mkdir -p ~/github-lab && cd ~/github-lab
git --version
gh --version   # اگر نصب نیست، در فصل ۱۷ نصب می‌کنیم
```

همه‌ی مثال‌های کتابچه را داخل همین پوشه اجرا کن. اگر خراب شد، پاکش کن و از نو بساز — این ارزان‌ترین راه یاد گرفتن است.

---

## منابع رسمی که همیشه باید کنار دستت باشد

| موضوع | لینک |
|---|---|
| مستندات Git | https://git-scm.com/doc |
| کتاب Pro Git (رایگان) | https://git-scm.com/book/fa/v2 |
| مستندات GitHub | https://docs.github.com |
| Changelog گیت‌هاب | https://github.blog/changelog/ |
| Actions Marketplace | https://github.com/marketplace?type=actions |
| GitHub Skills (آموزش تعاملی) | https://skills.github.com |

---

**شروع از:** [`01-Introduction-and-Ecosystem.md`](01-Introduction-and-Ecosystem.md) — یا [`SUMMARY.md`](SUMMARY.md) برای فهرست کامل.
