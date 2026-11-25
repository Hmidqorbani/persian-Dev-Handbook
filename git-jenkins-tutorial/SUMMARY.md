# فهرست کامل (SUMMARY)

> فهرست جامع تمام فایل‌ها و فصل‌های کتابچه. از این صفحه به عنوان نقشه راه استفاده کنید.

---

## بخش اول: پایه‌های Git

### [۰۱ — مقدمه و نقشه راه](01-Introduction.md)
- فصل ۱: کنترل نسخه چیست و چرا به آن نیاز داریم؟
- فصل ۲: انواع سیستم‌های کنترل نسخه (متمرکز vs توزیع‌شده)
- فصل ۳: Git چیست و چه تفاوتی با SVN دارد؟
- فصل ۴: اکوسیستم Git (GitHub، GitLab، Bitbucket)
- فصل ۵: CI/CD چیست و چرا Git + Jenkins مکمل هم‌اند؟
- فصل ۶: نقشه راه یادگیری و آماده‌سازی محیط کار

### [۰۲ — مبانی Git](02-Git-Basics.md)
- فصل ۱: نصب و پیکربندی Git
- فصل ۲: ساختار داخلی Git
- فصل ۳: اولین مخزن (init و clone)
- فصل ۴: ثبت تغییرات
- فصل ۵: مدیریت فایل‌ها و .gitignore
- فصل ۶: بررسی تاریخچه

### [۰۳ — شاخه‌بندی و ادغام](03-Git-Branching-and-Merging.md)
- فصل ۱: مفهوم Branch
- فصل ۲: کار با شاخه‌ها
- فصل ۳: ادغام (Merge)
- فصل ۴: حل تعارض (Conflict)
- فصل ۵: ابزارهای بصری Merge
- فصل ۶: استراتژی‌های Merge

### [۰۴ — مخازن راه دور و همکاری](04-Git-Remote-and-Collaboration.md)
- فصل ۱: مخازن راه دور
- فصل ۲: fetch، pull و push
- فصل ۳: کار با GitHub
- فصل ۴: Pull Request و بازبینی کد
- فصل ۵: Fork و مدل مشارکت
- فصل ۶: upstream و مدیریت شاخه‌های راه دور

---

## بخش دوم: Git حرفه‌ای

### [۰۵ — Git پیشرفته](05-Git-Advanced.md)
- فصل ۱: rebase و بازنویسی تاریخچه
- فصل ۲: Rebase تعاملی
- فصل ۳: cherry-pick
- فصل ۴: stash
- فصل ۵: reset و revert
- فصل ۶: reflog و نجات داده‌ها
- فصل ۷: bisect، Tags و Submodules

### [۰۶ — استراتژی‌های Branching](06-Branching-Strategies.md)
- فصل ۱: چرا استراتژی مهم است؟
- فصل ۲: GitFlow
- فصل ۳: GitHub Flow
- فصل ۴: Trunk-Based Development
- فصل ۵: Release Branch و Hotfix
- فصل ۶: مقایسه و انتخاب

### [۰۷ — بهترین روش‌ها و عیب‌یابی Git](07-Git-Best-Practices-Hooks-and-Troubleshooting.md)
- فصل ۱: Conventional Commits
- فصل ۲: Git Hooks
- فصل ۳: ابزارهای کیفیت کد
- فصل ۴: امنیت و امضای commit
- فصل ۵: عیب‌یابی خطاهای رایج
- فصل ۶: بازیابی از فاجعه و چک‌لیست

---

## بخش سوم: مبانی CI/CD و Jenkins

### [۰۸ — مبانی CI/CD](08-CI-CD-Fundamentals.md)
- فصل ۱: CI، CD و تفاوت‌ها
- فصل ۲: آناتومی یک Pipeline
- فصل ۳: چرا اتوماسیون؟
- فصل ۴: ابزارهای CI/CD و جایگاه Jenkins
- فصل ۵: مفاهیم Build تا Rollback
- فصل ۶: اصول طراحی Pipeline مؤثر

### [۰۹ — نصب و راه‌اندازی Jenkins](09-Jenkins-Installation-and-Setup.md)
- فصل ۱: Jenkins چیست؟
- فصل ۲: پیش‌نیازها
- فصل ۳: روش‌های نصب
- فصل ۴: راه‌اندازی اولیه
- فصل ۵: آشنایی با رابط کاربری
- فصل ۶: پیکربندی سراسری
- فصل ۷: ساختار JENKINS_HOME

### [۱۰ — Jobها و Freestyle](10-Jenkins-Jobs-and-Freestyle.md)
- فصل ۱: انواع Job
- فصل ۲: اولین Freestyle Job
- فصل ۳: مدیریت کد منبع
- فصل ۴: Build Triggers
- فصل ۵: Build Steps و Post-build
- فصل ۶: Jobهای پارامتریک
- فصل ۷: لاگ و مدیریت Build

---

## بخش چهارم: Pipeline حرفه‌ای

### [۱۱ — Declarative Pipeline](11-Jenkins-Pipeline-Declarative.md)
- فصل ۱: Pipeline as Code
- فصل ۲: ساختار Declarative
- فصل ۳: environment و options
- فصل ۴: موازی‌سازی و matrix
- فصل ۵: Credentials
- فصل ۶: کنترل خطا و input
- فصل ۷: اولین Pipeline کامل

### [۱۲ — Scripted Pipeline و Groovy](12-Jenkins-Pipeline-Scripted-and-Groovy.md)
- فصل ۱: Scripted Pipeline
- فصل ۲: مبانی Groovy
- فصل ۳: node، stage و step
- فصل ۴: متغیرها و توابع
- فصل ۵: مدیریت استثنا
- فصل ۶: Declarative vs Scripted
- فصل ۷: بهترین روش‌ها

### [۱۳ — Multibranch و Shared Libraries](13-Jenkins-Multibranch-and-Shared-Libraries.md)
- فصل ۱: Multi-Branch Pipeline
- فصل ۲: Organization Folders
- فصل ۳: ساختار Shared Library
- فصل ۴: Global Variables و Steps
- فصل ۵: بارگذاری داینامیک
- فصل ۶: طراحی کتابخانه سازمانی

### [۱۴ — Agents و Build توزیع‌شده](14-Jenkins-Agents-and-Distributed-Builds.md)
- فصل ۱: معماری Controller/Agent
- فصل ۲: روش‌های اتصال Agent
- فصل ۳: Labels و انتخاب Agent
- فصل ۴: Agent داکری و Kubernetes
- فصل ۵: توزیع بار
- فصل ۶: نگهداری و عیب‌یابی

### [۱۵ — امنیت، Credentials و RBAC](15-Jenkins-Security-Credentials-and-RBAC.md)
- فصل ۱: مدل امنیتی
- فصل ۲: انواع Credentials
- فصل ۳: Credentials Binding
- فصل ۴: احراز هویت
- فصل ۵: RBAC
- فصل ۶: سخت‌سازی
- فصل ۷: بهترین روش‌ها

### [۱۶ — افزونه‌ها، پشتیبان‌گیری و مقیاس‌پذیری](16-Jenkins-Plugins-Backup-and-Scaling.md)
- فصل ۱: اکوسیستم Plugin
- فصل ۲: افزونه‌های کلیدی
- فصل ۳: پشتیبان‌گیری و بازیابی
- فصل ۴: JCasC
- فصل ۵: مقیاس‌پذیری
- فصل ۶: به‌روزرسانی و مهاجرت

---

## بخش پنجم: یکپارچگی و پروژه‌ها

### [۱۷ — یکپارچگی Git + Jenkins](17-Git-Jenkins-Integration.md)
- فصل ۱: اتصال Git به Jenkins
- فصل ۲: Webhook
- فصل ۳: SCM Polling vs Webhook
- فصل ۴: Auto-Trigger
- فصل ۵: استراتژی شاخه
- فصل ۶: Status Check و گزارش
- فصل ۷: سناریوهای واقعی

### [۱۸ — پروژه عملی: Python CI/CD](18-Project-Python-CI-CD.md)
- فصل ۱: معرفی پروژه
- فصل ۲: محیط و وابستگی‌ها
- فصل ۳: Jenkinsfile تست
- فصل ۴: Artifact و نسخه‌بندی
- فصل ۵: استقرار و Rollback
- فصل ۶: توسعه پروژه

### [۱۹ — پروژه عملی: Java + Docker CI/CD](19-Project-Java-Maven-and-Docker-CI-CD.md)
- فصل ۱: پروژه Maven
- فصل ۲: Build و Test با Maven
- فصل ۳: نسخه‌بندی و Nexus
- فصل ۴: داکری‌سازی
- فصل ۵: ساخت و انتشار Image
- فصل ۶: استقرار و Rollback
- فصل ۷: پروژه کامل

### [۲۰ — مقایسه Jenkins با GitHub Actions و GitLab CI](20-Jenkins-vs-GitHub-Actions-and-GitLab-CI.md)
- فصل ۱: GitHub Actions
- فصل ۲: GitLab CI/CD
- فصل ۳: مقایسه معماری و هزینه
- فصل ۴: جدول مقایسه و معیار انتخاب
- فصل ۵: مهاجرت و هم‌زیستی

### [۲۱ — چک‌لیست‌ها، واژه‌نامه و منابع](21-Checklists-Glossary-and-Resources.md)
- فصل ۱: چک‌لیست Git
- فصل ۲: چک‌لیست Jenkins
- فصل ۳: چک‌لیست امنیت CI/CD
- فصل ۴: واژه‌نامه
- فصل ۵: میانبرها
- فصل ۶: منابع و ادامه مسیر
