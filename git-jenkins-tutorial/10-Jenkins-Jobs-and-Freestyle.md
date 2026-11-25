# بخش ۱۰: Jobها و Freestyle

**مدت یادگیری**: ۲ ساعت
**سطح سختی**: مبتدی
**پیش‌نیاز**: بخش ۰۹ (نصب و راه‌اندازی Jenkins)

---

## فهرست محتوا

- [فصل ۱: انواع Job](#فصل-۱-انواع-job)
- [فصل ۲: اولین Freestyle Job](#فصل-۲-اولین-freestyle-job)
- [فصل ۳: مدیریت کد منبع](#فصل-۳-مدیریت-کد-منبع)
- [فصل ۴: Build Triggers](#فصل-۴-build-triggers)
- [فصل ۵: Build Steps و Post-build](#فصل-۵-build-steps-و-post-build)
- [فصل ۶: Jobهای پارامتریک](#فصل-۶-jobهای-پارامتریک)
- [فصل ۷: لاگ و مدیریت Build](#فصل-۷-لاگ-و-مدیریت-build)

---

## فصل ۱: انواع Job

### مقدمه

Jenkins چند نوع Job دارد. شناخت آن‌ها کمک می‌کند مناسب‌ترین را انتخاب کنید.

### مفاهیم اساسی

| نوع | توضیح | کاربرد |
|-----|-------|--------|
| **Freestyle** | Job ساده با رابط گرافیکی | شروع سریع، کارهای ساده |
| **Pipeline** | تعریف با کد (Jenkinsfile) | فرآیندهای پیچیده |
| **Multi-Branch Pipeline** | کشف خودکار شاخه‌ها | پروژه‌های چندشاخه |
| **Folder** | گروه‌بندی Jobها | سازماندهی |
| **Multijob** | زنجیره Jobها | وابستگی بین Jobها |
| **Organization Folder** | اسکن سازمان | مدیریت خودکار |

### مقایسه و الگوهای مختلف

| معیار | Freestyle | Pipeline |
|-------|-----------|----------|
| پیکربندی | گرافیکی | کد (Groovy) |
| نسخه‌بندی | ندارد | در Git |
| پیچیدگی | کم | بالا |
| توصیه امروز | فقط کارهای ساده | استاندارد |

### اشکال شایع

- **اشتباه رایج**: استفاده از Freestyle برای فرآیندهای پیچیده. Pipeline بهتر است.
- **اشتباه رایج**: ندانستن تفاوت Multi-Branch با Pipeline.

### نکات مهم (Tips & Tricks)

- امروزه **Pipeline** استاندارد است؛ Freestyle را برای کارهای کوچک نگه دارید.
- Freestyle برای یادگیری مفاهیم پایه عالی است.

### خلاصه فصل

Jenkins چند نوع Job دارد: Freestyle (ساده)، Pipeline (کد)، Multi-Branch (چندشاخه). امروزه Pipeline استاندارد صنعت است.

---

## فصل ۲: اولین Freestyle Job

### مقدمه

بیایید اولین Job خود را بسازیم تا با مفاهیم آشنا شویم.

### مثال‌های عملی

**مراحل ساخت:**
1. New Item → نام: `hello-world` → Freestyle project → OK
2. **Build Steps** → Add build step → Execute shell
3. دستور:
   ```bash
   echo "Hello from Jenkins!"
   date
   whoami
   ```
4. Save → Build Now

**مشاهده نتیجه:** روی Build → Console Output کلیک کنید.

خروجی نمونه:
```
Started by user admin
Building in workspace /var/jenkins_home/workspace/hello-world
[hello-world] $ /bin/sh -xe /tmp/jenkins123.sh
+ echo Hello from Jenkins!
Hello from Jenkins!
+ date
Thu Oct 15 03:30:00 UTC 2026
Finished: SUCCESS
```

### اشکال شایع

- **اشتباه رایج**: استفاده از ویندوز بدون تنظیم shell. روی Linux از `Execute shell` و روی Windows از `Execute Windows batch command` استفاده کنید.
- **اشتباه رایج**: انتظار خروجی در رابط. خروجی در Console Output است.

### نکات مهم (Tips & Tricks)

- همیشه خروجی Console را بعد از Build بخوانید.
- Job را از یک منبع Git بسازید تا واقعی‌تر شود.

### خلاصه فصل

اولین Freestyle Job با New Item و Execute shell ساخته می‌شود. نتیجه در Console Output دیده می‌شود.

---

## فصل ۳: مدیریت کد منبع

### مقدمه

Job واقعی کد را از Git می‌گیرد. این فصل اتصال SCM را آموزش می‌دهد.

### مفاهیم اساسی

**پیکربندی:**
- Source Code Management → Git
- Repository URL
- Credentials
- Branches to build
- Additional Behaviours

### مثال‌های عملی

1. New Item → Freestyle
2. Source Code Management → Git
3. Repository URL: `https://github.com/username/repo.git`
4. Credentials: افزودن (Username/Password یا SSH)
5. Branch Specifier: `*/main`
6. Build Step:
   ```bash
   ls -la
   git log --oneline -5
   ```

**Poll SCM** برای بررسی تغییرات.

### اشکال شایع

- **اشتباه رایج**: نداشتن Credentials. برای مخازن خصوصی الزامی است.
- **اشتباه رایج**: Branch Specifier اشتباه. `*/main` یا `*/master`.

### نکات مهم (Tips & Tricks)

- برای GitHub از Personal Access Token یا Deploy Key استفاده کنید.
- Webhook سریع‌تر از Poll SCM است (بخش ۱۷).

### خلاصه فصل

Job با SCM Git کد را می‌گیرد. Repository URL، Credentials و Branch را پیکربندی کنید. Webhook بهترین روش Trigger است.

---

## فصل ۴: Build Triggers

### مقدمه

Build Trigger تعیین می‌کند Job کِی اجرا شود.

### مفاهیم اساسی

| Trigger | توضیح |
|---------|-------|
| **Build periodically** | زمان‌بندی Cron |
| **Poll SCM** | بررسی دوره‌ای تغییرات |
| **GitHub hook trigger** | Webhook |
| **Build after other projects** | بعد از Job دیگر |
| **Build when a change is pushed** | Webhook عمومی |
| **Trigger builds remotely** | با API |

**نحو Cron Jenkins:**
```
MINUTE HOUR DOM MONTH DOW
H/15 * * * *     # هر ۱۵ دقیقه
H 2 * * *        # هر روز ساعت ۲
H 9 * * 1-5      # هر روز کاری ساعت ۹
```

`H` یعنی «هش» — توزیع تصادفی برای جلوگیری از همزمانی.

### مثال‌های عملی

Poll SCM:
```
# هر ۵ دقیقه تغییرات را چک کن
H/5 * * * *
```

### اشکال شایع

- **اشتباه رایج**: Poll SCM با فاصله کوتاه. بار زیادی روی Git می‌آورد. Webhook بهتر است.
- **اشتباه رایج**: زمان‌بندی Cron اشتباه (فراموشی فیلد).

### نکات مهم (Tips & Tricks)

- از `H` استفاده کنید تا بار پخش شود.
- Webhook سریع‌ترین و کارآمدترین Trigger است.
- در محیط تست، Trigger را دستی نگه دارید.

### خلاصه فصل

Build Trigger انواع مختلفی دارد: زمان‌بندی، Poll SCM، Webhook. Webhook بهترین گزینه برای CI است.

---

## فصل ۵: Build Steps و Post-build

### مقدمه

Build Step کار واقعی است و Post-build Actions بعد از آن اجرا می‌شوند.

### مفاهیم اساسی

**Build Steps:**
- Execute shell / batch
- Invoke Maven / Gradle
- Run Docker
- Publish JUnit results

**Post-build Actions:**
- Archive the artifacts
- Publish JUnit test results
- Send email notification
- Trigger other projects
- Build other projects

### مثال‌های عملی

یک Job پایتونی کامل:

```bash
# Build Step
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
pytest --junitxml=results.xml
```

Post-build:
- Publish JUnit test result report → `results.xml`
- Archive artifacts → `dist/*`

### اشکال شایع

- **اشتباه رایج**: نداشتن Post-build برای گزارش تست. نتایج را از دست می‌دهید.
- **اشتباه رایج**: آرشیو نکردن Artifact. نمی‌توانید بعداً دانلود کنید.

### نکات مهم (Tips & Tricks)

- همیشه نتایج تست را Publish کنید.
- Artifactها را Archive کنید.
- اعلان ایمیل شکست را فعال کنید.

### خلاصه فصل

Build Step کار اصلی و Post-build Actions کارهای پس از آن (گزارش، آرشیو، اعلان) را انجام می‌دهند. هر دو را پیکربندی کنید.

---

## فصل ۶: Jobهای پارامتریک

### مقدمه

Job پارامتریک به کاربر اجازه می‌دهد ورودی بدهد؛ مثلاً انتخاب محیط استقرار.

### مفاهیم اساسی

**انواع پارامتر:**
- String
- Boolean
- Choice (لیست)
- Password
- File
- Git Parameter

### مثال‌های عملی

پارامتر `ENVIRONMENT` از نوع Choice: `dev`, `staging`, `prod`.

در Build Step:
```bash
echo "Deploying to $ENVIRONMENT"
if [ "$ENVIRONMENT" = "prod" ]; then
    echo "⚠️ Production deployment!"
fi
```

پارامتر Boolean `RUN_TESTS`:
```bash
if [ "$RUN_TESTS" = "true" ]; then
    pytest
fi
```

### اشکال شایع

- **اشتباه رایج**: نداشتن مقدار پیش‌فرض. همیشه یک مقدار امن بگذارید.
- **اشتباه رایج**: پارامتر Password در لاگ نمایش داده شود.

### نکات مهم (Tips & Tricks)

- برای انتخاب شاخه از Git Parameter Plugin استفاده کنید.
- مقادیر پیش‌فرض امن بگذارید (مثلاً `dev` نه `prod`).

### خلاصه فصل

Jobهای پارامتریک ورودی می‌گیرند. انواع مختلف پارامتر (String، Choice، Boolean) برای سناریوهای مختلف مناسب‌اند. مقادیر پیش‌فرض امن بگذارید.

---

## فصل ۷: لاگ و مدیریت Build

### مقدمه

مدیریت Buildها برای عیب‌یابی و نگهداری مهم است.

### مفاهیم اساسی

- **Console Output**: لاگ کامل Build.
- **Build History**: تاریخچه Buildها.
- **Discard Old Builds**: حذف خودکار Buildهای قدیمی.
- **Rebuild**: اجرای مجدد.

**Discard Old Builds:**
```
Strategy: Log Rotation
Days to keep builds: 30
Max # of builds to keep: 50
```

### مثال‌های عملی

مشاهده لاگ با API:

```bash
# دانلود Console Output
curl -u user:token http://localhost:8080/job/hello-world/lastBuild/consoleText
```

### اشکال شایع

- **اشتباه رایج**: نگه‌داشتن همه Buildها. دیسک پر می‌شود. از Discard Old Builds استفاده کنید.
- **اشتباه رایج**: نداشتن لاگ‌های ساخت‌یافته.

### نکات مهم (Tips & Tricks)

- Discard Old Builds را برای همه Jobها فعال کنید.
- از Timestamper Plugin برای زمان‌بندی لاگ استفاده کنید.
- Build شماره‌گذاری خودکار دارد.

### خلاصه فصل

Console Output، Build History و Discard Old Builds ابزارهای مدیریت Build هستند. حتماً سیاست نگه‌داری Buildها را تنظیم کنید.

---

## منابع و مراجع

- Jenkins Freestyle: [https://www.jenkins.io/doc/book/using/](https://www.jenkins.io/doc/book/using/)
- Build Triggers: [https://www.jenkins.io/doc/book/pipeline/syntax/#triggers](https://www.jenkins.io/doc/book/pipeline/syntax/#triggers)
- Cron syntax: [https://www.jenkins.io/doc/book/pipeline/syntax/#cron-syntax](https://www.jenkins.io/doc/book/pipeline/syntax/#cron-syntax)
- Parameterized Builds: [https://plugins.jenkins.io/git-parameter/](https://plugins.jenkins.io/git-parameter/)
