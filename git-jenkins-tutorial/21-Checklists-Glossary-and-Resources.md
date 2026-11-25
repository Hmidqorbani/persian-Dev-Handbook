# بخش ۲۱: چک‌لیست‌ها، واژه‌نامه و منابع

**مدت یادگیری**: ۱.۵ ساعت
**سطح سختی**: همه سطوح (مرجع)
**پیش‌نیاز**: مرور کل کتابچه

---

## فهرست محتوا

- [فصل ۱: چک‌لیست Git](#فصل-۱-چکلیست-git)
- [فصل ۲: چک‌لیست Jenkins](#فصل-۲-چکلیست-jenkins)
- [فصل ۳: چک‌لیست امنیت CI/CD](#فصل-۳-چکلیست-امنیت-cicd)
- [فصل ۴: واژه‌نامه](#فصل-۴-واژهنامه)
- [فصل ۵: میانبرها](#فصل-۵-میانبرها)
- [فصل ۶: منابع و ادامه مسیر](#فصل-۶-منابع-و-ادامه-مسیر)

---

## فصل ۱: چک‌لیست Git

### مقدمه

این چک‌لیست برای استفاده روزمره و راه‌اندازی پروژه‌های جدید است.

### چک‌لیست راه‌اندازی مخزن

- [ ] `git config --global user.name` تنظیم شده
- [ ] `git config --global user.email` تنظیم شده
- [ ] شاخه پیش‌فرض `main` تنظیم شده
- [ ] `.gitignore` مناسب ساخته شده
- [ ] `README.md` نوشته شده
- [ ] کلید SSH یا توکن راه‌اندازی شده
- [ ] remote به‌درستی تنظیم شده (`git remote -v`)
- [ ] Branch Protection فعال شده

### چک‌لیست کار روزمره

- [ ] صبح: `git fetch` و `git pull --rebase`
- [ ] برای هر کار: شاخه جدید بساز
- [ ] قبل از commit: `git status` و `git diff`
- [ ] پیام commit به سبک Conventional Commits
- [ ] قبل از push: `git pull --rebase`
- [ ] PR کوچک و واضح بساز
- [ ] بعد از merge: شاخه را حذف کن
- [ ] رمز/Secret را commit نکن

### چک‌لیست قبل از PR

- [ ] تغییرات تست شده
- [ ] تست‌ها اجرا شده
- [ ] Lint پاس شده
- [ ] پیام commit تمیز
- [ ] توضیحات PR نوشته شده
- [ ] Issue مرتبط لینک شده
- [ ] خودت تغییرات را مرور کرده‌ای

### چک‌لیست عیب‌یابی سریع

| مشکل | راه‌حل |
|------|-------|
| Detached HEAD | `git switch -c new-branch` |
| Push rejected | `git pull --rebase` |
| Merge conflict | ویرایش، `add`، `commit` |
| Commit گم‌شده | `git reflog` |
| فایل اشتباه commit | `git rm --cached` |
| پیام اشتباه | `git commit --amend` |

### خلاصه فصل

چک‌لیست Git شامل راه‌اندازی، کار روزمره، قبل از PR و عیب‌یابی سریع است. آن‌ها را عادت کنید.

---

## فصل ۲: چک‌لیست Jenkins

### مقدمه

چک‌لیست راه‌اندازی و نگهداری Jenkins.

### چک‌لیست راه‌اندازی

- [ ] Jenkins نصب و در دسترس
- [ ] افزونه‌های پیشنهادی نصب شده
- [ ] کاربر ادمین قوی ساخته شده
- [ ] URL درست تنظیم شده
- [ ] Java/JDK در Global Tools
- [ ] Git در Global Tools
- [ ] Maven/NodeJS/Docker در Global Tools
- [ ] Credentials لازم ساخته شده
- [ ] SMTP برای اعلان تنظیم شده
- [ ] `JENKINS_HOME` روی volume پایدار

### چک‌لیست هر Job/Pipeline

- [ ] `Jenkinsfile` در Git
- [ ] `options`: timeout + buildDiscarder
- [ ] `timestamps()` فعال
- [ ] Secretها در Credentials
- [ ] گزارش JUnit منتشر می‌شود
- [ ] Artifact آرشیو می‌شود
- [ ] `cleanWs()` در cleanup
- [ ] اعلان شکست فعال
- [ ] شرط `when` برای Deploy
- [ ] Pipeline بازبینی شده

### چک‌لیست نگهداری دوره‌ای

- [ ] پشتیبان‌گیری از JENKINS_HOME
- [ ] به‌روزرسانی Jenkins و افزونه‌ها
- [ ] بررسی فضای دیسک
- [ ] پاک‌سازی Buildهای قدیمی
- [ ] بازبینی Agentها
- [ ] بررسی لاگ‌های خطا
- [ ] حذف افزونه‌های بی‌استفاده
- [ ] تست بازیابی پشتیبان

### خلاصه فصل

چک‌لیست Jenkins شامل راه‌اندازی، هر Pipeline و نگهداری دوره‌ای است. رعایت آن از بسیاری از مشکلات جلوگیری می‌کند.

---

## فصل ۳: چک‌لیست امنیت CI/CD

### مقدمه

امنیت در CI/CD حیاتی است.

### چک‌لیست امنیت

**Git:**
- [ ] Secret در کد نیست
- [ ] `.gitignore` کامل
- [ ] gitleaks/pre-commit فعال
- [ ] امضای commit (اختیاری)
- [ ] Branch Protection
- [ ] بازبینی PR اجباری

**Jenkins:**
- [ ] احراز هویت فعال (LDAP/OAuth)
- [ ] RBAC پیاده‌شده
- [ ] CSRF فعال
- [ ] HTTPS اجباری
- [ ] خارج از اینترنت مستقیم
- [ ] Agent→Controller محدود
- [ ] Script Security جدی
- [ ] افزونه‌ها به‌روز
- [ ] پشتیبان‌گیری منظم

**Pipeline:**
- [ ] Secret از Credentials
- [ ] Secret از stdin
- [ ] حداقل دسترسی
- [ ] اسکن امنیتی image (Trivy)
- [ ] اسکن وابستگی (Safety/npm audit)
- [ ] Deploy تولید با تأیید

**زیرساخت:**
- [ ] Agentها به‌روز
- [ ] شبکه محدود
- [ ] مانیتورینگ و لاگ
- [ ] بازبینی دسترسی‌ها

### خلاصه فصل

چک‌لیست امنیت چهار بخش دارد: Git، Jenkins، Pipeline و زیرساخت. اصل حداقل دسترسی و مدیریت Secret محور آن است.

---

## فصل ۴: واژه‌نامه

### مقدمه

واژه‌نامه فارسی–انگلیسی اصطلاحات کلیدی.

### واژه‌نامه

| فارسی | انگلیسی | توضیح |
|-------|----------|-------|
| کنترل نسخه | Version Control | مدیریت تغییرات فایل‌ها |
| مخزن | Repository | محل نگه‌داری کد و تاریخچه |
| شاخه | Branch | خط توسعه مستقل |
| ادغام | Merge | ترکیب شاخه‌ها |
| تعارض | Conflict | تغییر متناقض در یک خط |
| درخواست ادغام | Pull Request | پیشنهاد ادغام |
| ثبت | Commit | ذخیره تغییرات |
| ناحیه آماده‌سازی | Staging Area | محل آماده‌سازی commit |
| اشاره‌گر | HEAD | اشاره به شاخه/commit فعلی |
| بازنویسی تاریخچه | Rebase | انتقال commitها |
| برچسب | Tag | نشانه نسخه |
| ادغام مداوم | Continuous Integration | تست خودکار مکرر |
| تحویل مداوم | Continuous Delivery | نسخه همیشه آماده |
| استقرار مداوم | Continuous Deployment | انتشار خودکار |
| خط لوله | Pipeline | زنجیره مراحل خودکار |
| مرحله | Stage | بخش منطقی Pipeline |
| عامل | Agent | ماشین اجرای Build |
| مصنوع | Artifact | خروجی Build |
| اعتبارنامه | Credentials | رمز/کلید |
| افزونه | Plugin | قابلیت اضافه Jenkins |
| کتابخانه مشترک | Shared Library | کد مشترک Pipeline |
| استقرار آبی-سبز | Blue-Green Deployment | دو محیط موازی |
| انتشار کاناری | Canary Release | انتشار تدریجی |
| بازگشت | Rollback | برگشت به نسخه قبل |
| پرچم ویژگی | Feature Flag | فعال/غیرفعال کردن ویژگی |

### خلاصه فصل

واژه‌نامه مرجعی سریع برای اصطلاحات کلیدی است. هنگام مطالعه بخش‌های دیگر به آن مراجعه کنید.

---

## فصل ۵: میانبرها

### مقدمه

میانبرهای Git و Jenkins برای سرعت بیشتر.

### میانبرهای Git (Aliases)

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.lg "log --oneline --graph --all --decorate"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.unstage "reset HEAD --"
git config --global alias.undo "reset --soft HEAD~1"
```

### دستورات پرکاربرد Git

```bash
git status -s                    # وضعیت خلاصه
git log --oneline -10            # ۱۰ commit آخر
git diff --staged                # تغییرات staged
git stash push -m "msg"          # ذخیره موقت
git switch -c feature/x          # شاخه جدید
git branch -vv                   # وضعیت upstream
git push -u origin HEAD          # push با upstream
```

### متغیرهای محیطی Jenkins

| متغیر | معنی |
|-------|------|
| `BUILD_NUMBER` | شماره Build |
| `BUILD_URL` | URL Build |
| `JOB_NAME` | نام Job |
| `WORKSPACE` | مسیر Workspace |
| `BRANCH_NAME` | نام شاخه (Multi-Branch) |
| `GIT_COMMIT` | هش commit |
| `CHANGE_ID` | شماره PR |
| `NODE_NAME` | نام Agent |

### خلاصه فصل

Aliasهای Git و متغیرهای محیطی Jenkins سرعت کار را زیاد می‌کنند. آن‌ها را حفظ کنید یا به‌عنوان مرجع نگه دارید.

---

## فصل ۶: منابع و ادامه مسیر

### مقدمه

مسیر یادگیری اینجا تمام نمی‌شود.

### مسیرهای ادامه یادگیری

**عمیق‌تر در Git:**
- Git internals (objects، refs، packfiles)
- Git hooks پیشرفته
- Monorepo strategies

**عمیق‌تر در CI/CD:**
- Kubernetes و ArgoCD (GitOps)
- Terraform (Infrastructure as Code)
- Observability (Prometheus، Grafana)
- DevSecOps

**گواهی‌نامه‌ها:**
- Jenkins Certified Engineer
- CKA / CKAD (Kubernetes)
- AWS DevOps Engineer

### پروژه‌های پیشنهادی برای تمرین

1. یک پروژه با CI کامل روی GitHub + Jenkins.
2. استقرار خودکار با Docker Compose.
3. استقرار روی Kubernetes.
4. ساخت یک Shared Library سازمانی.
5. پیاده‌سازی GitOps با ArgoCD.

### منابع کلیدی

**Git:**
- Pro Git Book (رایگان): [https://git-scm.com/book/fa/v2](https://git-scm.com/book/fa/v2)
- Git Documentation: [https://git-scm.com/docs](https://git-scm.com/docs)
- Learn Git Branching (تعاملی): [https://learngitbranching.js.org/](https://learngitbranching.js.org/)

**Jenkins:**
- Jenkins Handbook: [https://www.jenkins.io/doc/book/](https://www.jenkins.io/doc/book/)
- Pipeline Syntax: [https://www.jenkins.io/doc/book/pipeline/syntax/](https://www.jenkins.io/doc/book/pipeline/syntax/)
- Plugins: [https://plugins.jenkins.io/](https://plugins.jenkins.io/)

**CI/CD:**
- Continuous Delivery (کتاب): [https://continuousdelivery.com/](https://continuousdelivery.com/)
- GitHub Actions Docs: [https://docs.github.com/en/actions](https://docs.github.com/en/actions)
- GitLab CI Docs: [https://docs.gitlab.com/ee/ci/](https://docs.gitlab.com/ee/ci/)

**Docker & Kubernetes:**
- Docker Docs: [https://docs.docker.com/](https://docs.docker.com/)
- Kubernetes Docs: [https://kubernetes.io/docs/](https://kubernetes.io/docs/)

### پیام پایانی

تبریک! شما مسیر کامل Git و Jenkins را طی کردید. حالا:
- Git را عمیق می‌شناسید.
- Pipeline حرفه‌ای می‌نویسید.
- پروژه‌های واقعی را مستقر می‌کنید.

**بهترین راه تثبیت یادگیری: تدریس و تمرین.** یک پروژه واقعی بسازید، Pipeline بنویسید و آن را به دیگران آموزش دهید.

موفق باشید! 🚀

### خلاصه فصل

مسیر ادامه شامل Git internals، Kubernetes، GitOps و DevSecOps است. با پروژه‌های واقعی تمرین کنید و یادگیری را تثبیت کنید.

---

## منابع و مراجع

- Pro Git Book: [https://git-scm.com/book/fa/v2](https://git-scm.com/book/fa/v2)
- Jenkins Documentation: [https://www.jenkins.io/doc/](https://www.jenkins.io/doc/)
- Learn Git Branching: [https://learngitbranching.js.org/](https://learngitbranching.js.org/)
- Continuous Delivery: [https://continuousdelivery.com/](https://continuousdelivery.com/)
- Twelve-Factor App: [https://12factor.net/](https://12factor.net/)
- Awesome DevOps: [https://github.com/awesome-foss/awesome-sysadmin](https://github.com/awesome-foss/awesome-sysadmin)
