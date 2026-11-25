# Git & Jenkins از صفر تا تولید

**راهنمای جامع کنترل نسخه و CI/CD خودکار**

---

## این کتابچه چیست؟

این یک کتابچه آموزشی کامل، ساده و عملی است که شما را از **صفر** تا **استقرار خودکار در محیط تولید (Production)** همراهی می‌کند. در پایان این مسیر، شما نه‌فقط با دستورات Git و Jenkins آشنا می‌شوید، بلکه می‌توانید برای پروژه‌های واقعی یک خط لوله (Pipeline) حرفه‌ای CI/CD طراحی و پیاده‌سازی کنید.

هدف ما این است که مفاهیم را **از ریشه** توضیح دهیم؛ به‌جای اینکه فقط بگوییم «چه دستوری بزن»، توضیح می‌دهیم «چرا این کار را می‌کنیم» و «زیر پوست Git و Jenkins چه خبر است».

---

## مخاطب این کتابچه

- **مبتدی**: کسی که تازه با کنترل نسخه و CI/CD آشنا می‌شود و می‌خواهد اصولی یاد بگیرد.
- **متوسط**: توسعه‌دهنده یا DevOpsی که می‌خواهد دانش پراکنده‌اش را منسجم کند و به سطح حرفه‌ای برسد.

**پیش‌نیازها:**
- آشنایی مقدماتی با خط فرمان Linux (کار با `cd`, `ls`, `cat`, `chmod`)
- آشنایی با حداقل یک زبان برنامه‌نویسی (Python یا Java)
- مفاهیم پایه توسعه نرم‌افزار (کامپایل، تست، وابستگی‌ها)

---

## چه چیزی یاد می‌گیرید؟

- **Git** از `init` تا `rebase` تعاملی، `bisect` و `reflog`
- **استراتژی‌های Branching**: GitFlow، GitHub Flow و Trunk-Based Development
- **Jenkins** از نصب تا Pipeline as Code، Multi-Branch و Shared Libraries
- **یکپارچگی Git + Jenkins**: Webhook، SCM Polling و Auto-Trigger
- تفاوت **Declarative vs Scripted Pipeline** و بهترین روش‌ها
- **Jenkins Agents، Security، Credentials و RBAC**
- **پروژه‌های واقعی CI/CD** با Python، Java (Maven) و Docker
- **عیب‌یابی** خطاهای رایج Git و Jenkins
- مقایسه Jenkins با **GitHub Actions** و **GitLab CI**

---

## ساختار مسیر یادگیری

```
فاز ۱ — پایه‌های Git            (فایل‌های ۰۱ تا ۰۴)
   ↓
فاز ۲ — Git حرفه‌ای              (فایل‌های ۰۵ تا ۰۷)
   ↓
فاز ۳ — مبانی CI/CD و Jenkins   (فایل‌های ۰۸ تا ۱۰)
   ↓
فاز ۴ — Pipeline حرفه‌ای         (فایل‌های ۱۱ تا ۱۶)
   ↓
فاز ۵ — یکپارچگی و پروژه‌ها      (فایل‌های ۱۷ تا ۲۱)
```

---

## نحوه استفاده

1. **به ترتیب بخوانید.** فایل‌ها پیش‌رونده (Progressive) هستند و هر فایل بر فایل قبل سوار می‌شود.
2. **کدها را اجرا کنید.** همه مثال‌ها قابل اجرا هستند؛ فقط خواندن کافی نیست.
3. **تمرین‌ها را انجام دهید.** در پایان هر فصل، نکات و تمرین‌های پیشنهادی وجود دارد.
4. **از چک‌لیست‌ها استفاده کنید.** فایل ۲۱ شامل چک‌لیست‌های عملی برای محیط واقعی است.

> **نکته:** برای یادگیری بهتر، یک مخزن تمرینی بسازید و همه دستورات را در آن اجرا کنید.

---

## فهرست فایل‌ها

| # | فایل | موضوع |
|---|------|-------|
| ۰۱ | [Introduction](01-Introduction.md) | مقدمه و نقشه راه |
| ۰۲ | [Git Basics](02-Git-Basics.md) | مبانی Git |
| ۰۳ | [Branching & Merging](03-Git-Branching-and-Merging.md) | شاخه‌بندی و ادغام |
| ۰۴ | [Remote & Collaboration](04-Git-Remote-and-Collaboration.md) | مخازن راه دور و همکاری |
| ۰۵ | [Git Advanced](05-Git-Advanced.md) | Git پیشرفته |
| ۰۶ | [Branching Strategies](06-Branching-Strategies.md) | استراتژی‌های شاخه‌بندی |
| ۰۷ | [Best Practices & Troubleshooting](07-Git-Best-Practices-Hooks-and-Troubleshooting.md) | بهترین روش‌ها و عیب‌یابی Git |
| ۰۸ | [CI/CD Fundamentals](08-CI-CD-Fundamentals.md) | مبانی CI/CD |
| ۰۹ | [Jenkins Installation](09-Jenkins-Installation-and-Setup.md) | نصب و راه‌اندازی Jenkins |
| ۱۰ | [Jobs & Freestyle](10-Jenkins-Jobs-and-Freestyle.md) | Jobها و Freestyle |
| ۱۱ | [Declarative Pipeline](11-Jenkins-Pipeline-Declarative.md) | Pipeline اعلانی |
| ۱۲ | [Scripted Pipeline & Groovy](12-Jenkins-Pipeline-Scripted-and-Groovy.md) | Pipeline اسکریپتی |
| ۱۳ | [Multibranch & Shared Libraries](13-Jenkins-Multibranch-and-Shared-Libraries.md) | Multi-Branch و کتابخانه‌ها |
| ۱۴ | [Agents & Distributed Builds](14-Jenkins-Agents-and-Distributed-Builds.md) | Agentها و Build توزیع‌شده |
| ۱۵ | [Security & Credentials](15-Jenkins-Security-Credentials-and-RBAC.md) | امنیت و مجوزها |
| ۱۶ | [Plugins, Backup & Scaling](16-Jenkins-Plugins-Backup-and-Scaling.md) | افزونه‌ها و مقیاس‌پذیری |
| ۱۷ | [Git + Jenkins Integration](17-Git-Jenkins-Integration.md) | یکپارچگی Git و Jenkins |
| ۱۸ | [Project: Python CI/CD](18-Project-Python-CI-CD.md) | پروژه عملی Python |
| ۱۹ | [Project: Java + Docker CI/CD](19-Project-Java-Maven-and-Docker-CI-CD.md) | پروژه عملی Java و Docker |
| ۲۰ | [Jenkins vs GitHub Actions vs GitLab CI](20-Jenkins-vs-GitHub-Actions-and-GitLab-CI.md) | مقایسه ابزارها |
| ۲۱ | [Checklists, Glossary & Resources](21-Checklists-Glossary-and-Resources.md) | چک‌لیست، واژه‌نامه و منابع |

---

## پیشنهاد مسیر مطالعه

- **اگر عجله دارید:** فایل‌های ۰۱، ۰۲، ۰۵، ۰۸، ۱۱، ۱۷، ۱۸ را بخوانید.
- **برای یادگیری کامل:** همه فایل‌ها را به ترتیب بخوانید.
- **برای مرور سریع:** فایل ۲۱ (چک‌لیست و واژه‌نامه) را باز کنید.

---

## مشارکت

اگر خطایی دیدید یا پیشنهادی برای بهتر شدن محتوا داشتید، خوشحال می‌شویم از طریق Issue یا Pull Request اطلاع دهید.

---

## مجوز

این محتوا برای استفاده آموزشی آزاد است.
