# بخش ۱۷: یکپارچگی Git + Jenkins

**مدت یادگیری**: ۳ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۱۶ (افزونه‌ها، پشتیبان‌گیری و مقیاس‌پذیری)

---

## فهرست محتوا

- [فصل ۱: اتصال Git به Jenkins](#فصل-۱-اتصال-git-به-jenkins)
- [فصل ۲: Webhook](#فصل-۲-webhook)
- [فصل ۳: SCM Polling vs Webhook](#فصل-۳-scm-polling-vs-webhook)
- [فصل ۴: Auto-Trigger](#فصل-۴-auto-trigger)
- [فصل ۵: استراتژی شاخه](#فصل-۵-استراتژی-شاخه)
- [فصل ۶: Status Check و گزارش](#فصل-۶-status-check-و-گزارش)
- [فصل ۷: سناریوهای واقعی](#فصل-۷-سناریوهای-واقعی)

---

## فصل ۱: اتصال Git به Jenkins

### مقدمه

حالا وقت ترکیب همه‌چیز است: Git کد را نگه می‌دارد و Jenkins آن را خودکار می‌سازد، تست و مستقر می‌کند.

### مفاهیم اساسی

**گام‌های اتصال:**
1. نصب Git Plugin (پیش‌فرض هست).
2. پیکربندی Git در Global Tools.
3. افزودن Credentials (SSH یا Token).
4. ساخت Job با SCM Git.
5. تعیین Branch.
6. تعیین Trigger.

### مثال‌های عملی

**Credential SSH:**
```bash
ssh-keygen -t ed25519 -f ~/.ssh/jenkins_deploy -N ""
cat ~/.ssh/jenkins_deploy.pub
# کلید عمومی را به GitHub → Deploy Keys اضافه کنید
# کلید خصوصی را در Jenkins Credentials (SSH Key) وارد کنید
```

**Pipeline با Git:**
```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-deploy-key',
                    url: 'git@github.com:myorg/myrepo.git'
            }
        }
    }
}
```

### اشکال شایع

- **اشتباه رایج**: کلید SSH با رمز. Jenkins نمی‌تواند رمز بدهد.
- **اشتباه رایج**: Deploy Key با دسترسی فقط خواندن اما نیاز به push. دسترسی را تنظیم کنید.

### نکات مهم (Tips & Tricks)

- برای checkout از `checkout scm` در Multi-Branch استفاده کنید.
- Shallow clone (`--depth 1`) برای سرعت.
- Deploy Key اختصاصی برای Jenkins بسازید.

### خلاصه فصل

اتصال Git به Jenkins با نصب Git Plugin، Credentials و Job انجام می‌شود. از Deploy Key اختصاصی استفاده کنید.

---

## فصل ۲: Webhook

### مقدمه

**Webhook** سریع‌ترین راه اطلاع‌دهی تغییرات به Jenkins است. GitHub با هر push، یک درخواست HTTP به Jenkins می‌فرستد.

### مفاهیم اساسی

**جریان:**
```
Developer push → GitHub → Webhook (HTTP POST) → Jenkins → Build
```

**پیکربندی در GitHub:**
1. Settings → Webhooks → Add webhook
2. Payload URL: `http://jenkins.example.com/github-webhook/`
3. Content type: `application/json`
4. Events: Just the push event (یا PR)
5. Active ✅

**پیکربندی در Jenkins:**
- Job → Build Triggers → ✅ GitHub hook trigger for GITScm polling
- (برای Multi-Branch) خودکار فعال است.

### مثال‌های عملی

تست Webhook با ngrok (برای محیط محلی):

```bash
ngrok http 8080
# URL عمومی بگیرید: https://abc123.ngrok.io
# در GitHub: https://abc123.ngrok.io/github-webhook/
```

### اشکال شایع

- **اشتباه رایج**: Jenkins پشت فایروال و بدون دسترسی عمومی. از ngrok یا reverse proxy استفاده کنید.
- **اشتباه رایج**: URL اشتباه. باید به `/github-webhook/` ختم شود.

### نکات مهم (Tips & Tricks)

- GitHub دکمه «Redeliver» برای تست دارد.
- Recent Deliveries را برای عیب‌یابی ببینید.
- برای GitLab: `/project/<project>` و برای Bitbucket مسیر متفاوت.

### خلاصه فصل

Webhook سریع‌ترین Trigger است. GitHub با push یک POST به Jenkins می‌فرستد. URL باید به `/github-webhook/` ختم شود.

---

## فصل ۳: SCM Polling vs Webhook

### مقدمه

دو روش اطلاع از تغییرات: Polling و Webhook.

### مفاهیم اساسی

**SCM Polling:** Jenkins دوره‌ای (مثلاً هر ۵ دقیقه) مخزن را چک می‌کند.

```
Jenkins → (هر ۵ دقیقه) → Git: تغییری هست؟
```

**Webhook:** Git فعالانه به Jenkins اطلاع می‌دهد.

```
Git → (فوری) → Jenkins: تغییر!
```

### مقایسه و الگوهای مختلف

| معیار | Polling | Webhook |
|-------|---------|---------|
| سرعت | تأخیر (دقیقه) | فوری |
| بار روی Git | زیاد | کم |
| نیاز به دسترسی عمومی | ندارد | دارد |
| پیچیدگی | کم | متوسط |
| توصیه | فایروال | استاندارد |

### مثال‌های عملی

Polling:
```groovy
triggers {
    pollSCM('H/5 * * * *')
}
```

### اشکال شایع

- **اشتباه رایج**: Polling با فاصله کوتاه. بار زیاد.
- **اشتباه رایج**: Webhook بدون دسترسی عمومی. کار نمی‌کند.

### نکات مهم (Tips & Tricks)

- Webhook را ترجیح دهید؛ Polling را فقط اگر دسترسی عمومی ندارید.
- اگر هر دو را دارید، Polling را غیرفعال کنید.
- از `H` در Cron استفاده کنید.

### خلاصه فصل

Polling دوره‌ای چک می‌کند (کند، پر بار) و Webhook فوری اطلاع می‌دهد (سریع، کم‌بار). Webhook استاندارد است.

---

## فصل ۴: Auto-Trigger

### مقدمه

**Auto-Trigger** یعنی Build بدون دخالت انسان شروع شود.

### مفاهیم اساسی

سناریوهای Trigger:
- Push به شاخه → Build.
- باز شدن PR → Build + Test.
- Merge به main → Build + Deploy.
- Tag → Release.
- زمان‌بندی → کارهای دوره‌ای.

### مثال‌های عملی

Multi-Branch با شرط:

```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            steps { sh 'pytest' }
        }
        stage('Deploy Staging') {
            when { branch 'develop' }
            steps { sh './deploy.sh staging' }
        }
        stage('Deploy Prod') {
            when {
                allOf {
                    branch 'main'
                    not { changeRequest() }
                }
            }
            steps {
                input 'Deploy to production?'
                sh './deploy.sh prod'
            }
        }
    }
}
```

### اشکال شایع

- **اشتباه رایج**: Trigger کردن Deploy تولید روی هر push. خطرناک.
- **اشتباه رایج**: نبود شرط شاخه. همه شاخه‌ها Deploy می‌کنند.

### نکات مهم (Tips & Tricks)

- با `when` مرحله‌ها را شرطی کنید.
- برای PR فقط Test، برای main Deploy.
- `changeRequest()` برای PRها.

### خلاصه فصل

Auto-Trigger با Webhook/Polling و شرط‌های `when` کار می‌کند. Deploy را فقط برای شاخه‌های مشخص فعال کنید.

---

## فصل ۵: استراتژی شاخه

### مقدمه

Jenkins باید با استراتژی Branching شما (بخش ۰۶) هماهنگ باشد.

### مفاهیم اساسی

| استراتژی | Jenkins |
|-----------|---------|
| GitHub Flow | Multi-Branch + PR build |
| GitFlow | Multi-Branch با شرط develop/main |
| Trunk-Based | main + PR build سریع |

**نگاشت محیط‌ها:**
- `feature/*` → Build + Test (بدون Deploy)
- `develop` → Deploy به Staging
- `main` → Deploy به Production
- `release/*` → Deploy به Staging + تأیید

### مثال‌های عملی

```groovy
stage('Deploy') {
    steps {
        script {
            switch (env.BRANCH_NAME) {
                case 'develop':
                    sh './deploy.sh staging'
                    break
                case 'main':
                    input 'Deploy to prod?'
                    sh './deploy.sh prod'
                    break
                default:
                    echo 'Feature branch: build only'
            }
        }
    }
}
```

### اشکال شایع

- **اشتباه رایج**: Deploy تولید از شاخه feature.
- **اشتباه رایج**: ناهماهنگی استراتژی Git و Jenkins.

### نکات مهم (Tips & Tricks)

- نگاشت شاخه→محیط را مستند کنید.
- از محیط‌های مجزا برای Staging و Prod.
- Deploy تولید همیشه با تأیید.

### خلاصه فصل

Jenkins باید با استراتژی Branching هماهنگ باشد: feature→test، develop→staging، main→production. نگاشت را مستند کنید.

---

## فصل ۶: Status Check و گزارش

### مقدمه

گزارش وضعیت Build به GitHub، بازبینی PR را مؤثر می‌کند.

### مفاهیم اساسی

**GitHub Status API:** Jenkins نتیجه Build را به PR گزارش می‌دهد (✅/❌).

**Branch Protection:** GitHub می‌تواند merge را تا موفقیت CI مسدود کند.

**افزونه‌ها:**
- GitHub Plugin (Status).
- Coverage/Test Report.

### مثال‌های عملی

```groovy
post {
    success {
        echo 'CI passed'
        // Jenkins به‌طور خودکار status می‌فرستد اگر GitHub Plugin نصب باشد
    }
    failure {
        echo 'CI failed'
    }
}
```

Branch Protection در GitHub:
- Settings → Branches → Add rule
- Require status checks to pass ✅
- انتخاب `continuous-integration/jenkins/pr-merge`

### اشکال شایع

- **اشتباه رایج**: نبود Status Check. PRها بدون تست merge می‌شوند.
- **اشتباه رایج**: Status اشتباه (CI موفق ولی واقعاً شکست).

### نکات مهم (Tips & Tricks)

- Branch Protection را برای main فعال کنید.
- Status Check را الزامی کنید.
- گزارش Coverage را در PR بگذارید.

### خلاصه فصل

Status Check نتیجه Build را به PR گزارش می‌دهد و Branch Protection merge را تا موفقیت CI مسدود می‌کند. این ترکیب کیفیت را تضمین می‌کند.

---

## فصل ۷: سناریوهای واقعی

### مقدمه

بیایید چند سناریوی واقعی را کنار هم ببینیم.

### مثال‌های عملی

**سناریو ۱: Feature Branch → PR**
```
push feature → Webhook → Jenkins Build+Test → Status به PR
→ بازبینی → merge → Deploy Staging
```

**سناریو ۲: GitFlow**
```
feature/* → develop → (تست) → release/* → (تأیید) → main → (tag) → Deploy Prod
```

**سناریو ۳: Trunk-Based**
```
commit کوچک به main → Build+Test سریع → Deploy Prod (feature flag)
```

**Pipeline کامل یکپارچگی:**

```groovy
pipeline {
    agent any
    triggers { githubPush() }
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build & Test') {
            parallel {
                stage('Lint') { steps { sh 'flake8' } }
                stage('Unit') { steps { sh 'pytest tests/unit' } }
                stage('Integration') { steps { sh 'pytest tests/integration' } }
            }
        }
        stage('Deploy') {
            when { branch 'main' }
            steps {
                input 'Deploy to production?'
                sh './deploy.sh prod'
            }
        }
    }
    post {
        always { cleanWs() }
        failure { echo 'Notify team!' }
    }
}
```

### اشکال شایع

- **اشتباه رایج**: Pipelineهای متفاوت برای هر شاخه. از Multi-Branch استفاده کنید.
- **اشتباه رایج**: نداشتن Post برای اعلان.

### نکات مهم (Tips & Tricks)

- یک Jenkinsfile برای همه شاخه‌ها، با شرط `when`.
- اعلان شکست را فعال کنید.
- لاگ‌ها را برای عیب‌یابی نگه دارید.

### خلاصه فصل

سناریوهای واقعی ترکیبی از Webhook، Multi-Branch و شرط‌های `when` هستند. یک Jenkinsfile برای همه شاخه‌ها کافی است.

---

## منابع و مراجع

- GitHub Webhook: [https://docs.github.com/en/webhooks](https://docs.github.com/en/webhooks)
- Jenkins GitHub Plugin: [https://plugins.jenkins.io/github/](https://plugins.jenkins.io/github/)
- GitHub Branch Protection: [https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository)
- Jenkins Triggers: [https://www.jenkins.io/doc/book/pipeline/syntax/#triggers](https://www.jenkins.io/doc/book/pipeline/syntax/#triggers)
