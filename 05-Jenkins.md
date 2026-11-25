# فایل ۰۵ — Jenkins: استاد خطوط لوله

**مدت یادگیری**: ۸ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۱ تا ۰۴ (مفاهیم CI/CD)

---

## فهرست محتوا

- [فصل ۲۱: معماری Jenkins و نصب](#فصل-۲۱-معماری-jenkins-و-نصب)
- [فصل ۲۲: Jenkinsfile و Pipeline (Declarative در برابر Scripted)](#فصل-۲۲-jenkinsfile-و-pipeline-declarative-در-برابر-scripted)
- [فصل ۲۳: پلاگین‌ها، اعتبارنامه‌ها و مدیریت پیکربندی](#فصل-۲۳-پلاگینها-اعتبارنامهها-و-مدیریت-پیکربندی)
- [فصل ۲۴: تریگرها، Multibranch و یکپارچه‌سازی](#فصل-۲۴-تریگرها-multibranch-و-یکپارچهسازی)
- [فصل ۲۵: بهترین روش‌ها و عیب‌یابی Jenkins](#فصل-۲۵-بهترین-روشها-و-عیبیابی-jenkins)

---

## فصل ۲۱: معماری Jenkins و نصب

### مقدمه

Jenkins قدیمی‌ترین و محبوب‌ترین سرور CI/CD متن‌باز است. با وجود رقبای جدید (GitHub Actions، GitLab CI)، Jenkins هنوز در بسیاری از سازمان‌های بزرگ به دلیل **انعطاف‌پذیری بی‌نظیر** و **هزاران پلاگین** حکمرانی می‌کند. درک معماری و نصب درست آن پیش‌زمینهٔ هر استفادهٔ جدی است.

در این فصل اجزای Jenkins، تفاوت Controller/Agent، و روش‌های نصب (Docker، WAR، Kubernetes) را بررسی می‌کنیم.

### مفاهیم اساسی

**معماری Jenkins:**

| جزء | نقش |
|---|---|
| **Controller (Master)** | مغز پردازش: UI، scheduling، مدیریت پیکربندی |
| **Agent (Slave/Node)** | ماشینی که واقعاً Jobها را اجرا می‌کند |
| **Executor** | یک نخ اجرایی روی Agent (تعداد هم‌زمان Jobها) |
| **Job/Item** | یک تعریف کار (Freestyle، Pipeline، Multibranch) |
| **Workspace** | پوشهٔ محلی برای چک‌اوت کد روی Agent |

**چرا Agent جدا؟**
- مقیاس‌پذیری: Agentها را بر اساس بار اضافه می‌کنید.
- ایزولاسیون: هر Agent محیط متفاوت (OS/ابزار) دارد.
- امنیت: اجرای کد کاربر روی Controller خطرناک است.

**انواع نصب:**
- **WAR file:** `java -jar jenkins.war` — ساده‌ترین.
- **Docker:** کانتینرایز شده (توصیه برای توسعه).
- **Kubernetes (Jenkins Operator / JCasC):** در محیط ابری.

**LTS vs Weekly:** نسخهٔ LTS (Long-Term Support) برای تولید پایدار؛ Weekly برای ویژگی‌های جدید.

### مثال‌های عملی

**۱. اجرای Jenkins با Docker:**

```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

```bash
# گرفتن رمز اولیهٔ Administrator
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

**۲. اجرای WAR به صورت مستقل:**

```bash
java -jar jenkins.war --httpPort=8080
# مرورگر: http://localhost:8080
```

**۳. ثبت یک Agent (بر اساس SSH):**

```
Manage Jenkins → Nodes → New Node
- Name: agent-1
- Remote root directory: /home/jenkins
- Launch method: SSH (Host, Credentials)
```

**۴. Pipeline در Kubernetes (Pod Template):**

```groovy
podTemplate(containerTemplate(name: 'docker', image: 'docker:24')) {
  node(POD) {
    stage('Build') { container('docker') { sh 'docker build .' } }
  }
}
```

### مقایسه و دیدگاه‌های مختلف

| ویژگی | Jenkins | GitHub Actions | GitLab CI |
|---|---|---|---|
| انعطاف | بسیار بالا (Groovy) | متوسط | متوسط |
| پلاگین | هزاران | مارکت‌پلیس | built-in |
| یادگیری | سخت | آسان | آسان |
| نگهداری | سنگین | کم (SaaS) | کم (SaaS/self) |
| خودمیزبانی | عالی | نیاز به runner | نیاز به Omnibus |

**دیدگاه‌ها:**
- Jenkins برای سازمان‌هایی که نیاز به کنترل کامل و pipelineهای پیچیده دارند عالی است.
- تیم‌های کوچک/مدرن ترجیح می‌دهند GitHub Actions/GitLab CI (کمتر نگهداری).
- توصیه: Jenkins را با JCasC (تعریف به صورت کد) مدیریت کنید تا "Snowflake server" نشود.

### اشتباهات رایج

- **اجرای Job روی Controller:** بار را روی مغز می‌اندازید؛ Agent اضافه کنید.
- **فراموش کردن بکاپ `jenkins_home`:** همهٔ پیکربندی آنجاست؛ بدون بکاپ، فاجعه.
- **پلاگین‌های قدیمی/ناپایدار:** فقط پلاگین‌های نگهداری‌شده را نصب کنید.
- **نصب بدون Reverse Proxy و TLS:** Jenkins روی ۸۰۸۰ بدون HTTPS = ریسک امنیتی.
- **استفاده از Freestyle به جای Pipeline:** Freestyle قابلیت نسخه‌گذاری ندارد؛ همیشه Pipeline (Jenkinsfile) بنویسید.

### نکات مهم و ترفندها

- **JCasC (Jenkins Configuration as Code):** پیکربندی را در YAML تعریف کنید:
  ```yaml
  jenkins:
    securityRealm:
      local:
        allowsSignup: false
  ```
- **Lock down Controller:** اجازه ندهید Jobها روی master اجرا شوند (تنظیم executors = 0).
- **بکاپ خودکار:** `jenkins_home` را با cron بکاپ بگیرید.
- **Pipeline Shared Libraries:** کد Groovy مشترک را در repo جدا نگه دارید (فصل ۲۵).
- **Blue Ocean:** پلاگین UI مدرن برای نمایش pipeline (اگرچه deprecated، هنوز کاربرد دارد).
- **Agent labels:** Jobها را به Agent با label خاص (مثل `docker`, `linux`) هدایت کنید.

### خلاصه فصل

Jenkins از Controller (مغز) و Agent (اجراکننده) تشکیل شده. نصب با Docker/WAR/Kubernetes ممکن است. LTS برای تولید توصیه می‌شود. مزیت اصلی: انعطاف Groovy + هزاران پلاگین؛ هزینه: نگهداری سنگین. کلید: Agent جدا برای اجرا، بکاپ `jenkins_home`، و مدیریت با JCasC برای جلوگیری از Snowflake server.

---

## فصل ۲۲: Jenkinsfile و Pipeline (Declarative در برابر Scripted)

### مقدمه

**Pipeline-as-Code** قلب Jenkins مدرن است. به جای کلیک در UI، شما یک فایل `Jenkinsfile` در ریشهٔ مخزن می‌نویسید که خط لوله را تعریف می‌کند. این یعنی نسخه‌گذاری، بازبینی و بازاستفاده از pipeline مثل کد.

در این فصل دو سینتکس (Declarative و Scripted) را مقایسه می‌کنیم و یک Jenkinsfile واقعی می‌سازیم.

### مفاهیم اساسی

**دو نوع سینتکس:**

| ویژگی | Declarative | Scripted |
|---|---|---|
| ساختار | مبتنی بر بخش (sections) | Groovy آزاد |
| یادگیری | آسان‌تر | سخت‌تر |
| انعطاف | کمتر | نامحدود |
| استفاده | توصیهٔ رسمی | legacy/پیچیده |

**عناصر Declarative:**

```groovy
pipeline {
  agent any            // کجا اجرا شود
  options { ... }      // تنظیمات کلی
  stages {
    stage('Build') {
      steps { sh 'make' }
    }
  }
  post {
    success { ... }
    failure { ... }
  }
}
```

- `agent`: تعیین Agent (any، label، none، docker).
- `stages/stage`: مراحل منطقی.
- `steps`: دستورات (sh، bat، script).
- `post`: اقدامات پس از پایان (always/success/failure).
- `environment`: متغیرهای محیطی.
- `parameters`: ورودی در زمان اجرا.
- `triggers`: زمان‌بندی (cron، pollSCM).

### مثال‌های عملی

**۱. یک Jenkinsfile کامل (Declarative):**

```groovy
pipeline {
    agent any
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    environment {
        REGISTRY = 'registry.example.com'
        APP = 'myapp'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    docker.build("${REGISTRY}/${APP}:${env.BUILD_NUMBER}").push()
                }
            }
        }
        stage('Deploy') {
            when { branch 'main' }
            steps {
                sh './deploy.sh production'
            }
        }
    }
    post {
        success { slackSend channel: '#ci', message: '✅ Build OK' }
        failure { slackSend channel: '#ci', message: '❌ Build Failed' }
        always  { cleanWs() }
    }
}
```

**۲. استفاده از Docker Agent:**

```groovy
pipeline {
    agent { docker { image 'node:20' } }
    stages {
        stage('Test') { steps { sh 'npm test' } }
    }
}
```

**۳. Scripted (برای منطق پیچیده):**

```groovy
node('linux') {
    try {
        stage('Build') { sh 'make' }
        stage('Test')  { sh 'make test' }
    } catch (e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
```

**۴. ورودی پارامتر:**

```groovy
parameters {
    choice(name: 'ENV', choices: ['staging','production'], description: 'Target')
}
stage('Deploy') { steps { sh "./deploy.sh ${params.ENV}" } }
```

### مقایسه و دیدگاه‌های مختلف

**چرا Declarative توصیه می‌شود؟**
- خواناتر و استاندارد (schema validation دارد).
- خطای سینتکس زود هنگام گرفته می‌شود.
- مستندات رسمی روی آن متمرکز است.

**کی وقتی Scripted؟**
- وقتی منطق شرطی بسیار پیچیده یا حلقه‌های داینامیک نیاز است.
- اما حتی آن‌وقت، `script {}` داخل Declarative معمولاً کافی است.

**دیدگاه:** ترکیب Declarative + `script {}` برای ۹۹٪ موارد بهترین است.

### اشتباهات رایج

- **سینتکس Groovy نادرست:** فراموش کردن `;` یا اشتباه در closure باعث شکست می‌شود.
- **Hard-code کردن رمز:** رمز را در Jenkinsfile ننویسید؛ از Credentials استفاده کنید (`withCredentials`).
- **`agent any` روی Controller:** بار را روی master می‌اندازد.
- **فراموش کردن `cleanWs()`:** فایل‌های قدیمی بین buildها تداخل می‌کنند.
- **timeout نداشتن:** یک pipeline گیر کرده تا ابد منابع را اشغال می‌کند.

### نکات مهم و ترفندها

- **Shared Libraries:** کد تکراری را به `vars/` در repo جدا ببرید:
  ```groovy
  // vars/deploy.groovy
  def call(env) { sh "./deploy.sh ${env}" }
  ```
  سپس در Jenkinsfile: `deploy('production')`.
- **`withCredentials`:** ایمن خواندن رمز:
  ```groovy
  withCredentials([string(credentialsId: 'TOKEN', variable: 'TOKEN')]) {
      sh 'curl -H "Token: $TOKEN" ...'
  }
  ```
- **Stage View:** از UI برای دیدن زمان هر stage استفاده کنید.
- **Parallel stages:**
  ```groovy
  stage('Test') {
    parallel {
      stage('Unit') { steps { sh 'npm run unit' } }
      stage('Int')  { steps { sh 'npm run int' } }
    }
  }
  ```
- **Replay:** قبل از commit، Jenkinsfile را با «Replay» تست کنید.

### خلاصه فصل

Jenkinsfile = Pipeline-as-Code. Declarative (بخش‌محور، آسان، توصیه‌شده) در برابر Scripted (Groovy آزاد، قدرتمند اما سخت). عناصر اصلی: agent، stages/steps، environment، post، parameters، triggers. رمزها را با `withCredentials` ایمن کنید، `cleanWs()` و timeout فراموش نکنید، و کد تکراری را به Shared Library ببرید.

---

## فصل ۲۳: پلاگین‌ها، اعتبارنامه‌ها و مدیریت پیکربندی

### مقدمه

قدرت Jenkins از **پلاگین‌ها** می‌آید — اما همین پلاگین‌ها می‌توانند نقطهٔ ضعف (نگهداری، امنیت) باشند. مدیریت اعتبارنامه‌ها (credentials) و پیکربندی به صورت کد (JCasC) تفاوت بین یک Jenkins قابل اعتماد و یک فاجعهٔ دستی است.

در این فصل مدیریت پلاگین، سیستم Credentials، و JCasC را بررسی می‌کنیم.

### مفاهیم اساسی

**پلاگین‌ها (Plugins):**
Jenkins قابلیت‌هایش را از پلاگین می‌گیرد (Git، Docker، Pipeline، Kubernetes، Slack...). نصب از `Manage Jenkins → Plugins`.

**اعتبارنامه‌ها (Credentials):**
رمزها، توکن‌ها و کلیدها در یک **store رمزگذاری‌شده** نگهداری می‌شوند و با `credentialsId` فراخوانی می‌شوند — هرگز در متن Jenkinsfile نمی‌آیند.

انواع:
- **Username with password**
- **Secret text (token)**
- **SSH Username with private key**
- **Secret file** (مثل کلید JSON سرویس)

**JCasC (Jenkins Configuration as Code):**
پیکربندی Jenkins در یک فایل YAML تعریف می‌شود و هنگام راه‌اندازی اعمال می‌گردد. این یعنی Jenkins شما **قابل بازتولید** (reproducible) است.

**Folder-based credentials:** دامنهٔ اعتبارنامه را محدود کنید (فقط در پوشهٔ مربوطه در دسترس باشد).

### مثال‌های عملی

**۱. نصب پلاگین از CLI:**

```bash
java -jar jenkins-cli.jar -s http://localhost:8080/ install-plugin git docker-workflow
```

**۲. استفاده از Credentials در Pipeline:**

```groovy
pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub',
                    usernameVariable: 'U',
                    passwordVariable: 'P')]) {
                    sh 'echo $P | docker login -u $U --password-stdin'
                }
            }
        }
    }
}
```

**۳. فایل JCasC (`jenkins.yaml`):**

```yaml
jenkins:
  systemMessage: "CI Server - Managed by JCasC"
  securityRealm:
    local:
      allowsSignup: false
      users:
        - id: admin
          password: ${ADMIN_PASSWORD}
  authorizationStrategy:
    loggedInUsersCanDoAnything:
      allowAnonymousRead: false
credentials:
  system:
    domainCredentials:
      - credentials:
          - usernamePassword:
              scope: GLOBAL
              id: docker-hub
              username: admin
              password: ${DOCKER_PASSWORD}
```

**۴. بارگذاری JCasC در Docker:**

```bash
docker run -d \
  -v $(pwd)/jenkins.yaml:/var/jenkins_home/jenkins.yaml \
  -e CASC_JENKINS_CONFIG=/var/jenkins_home/jenkins.yaml \
  jenkins/jenkins:lts
```

### مقایسه و دیدگاه‌های مختلف

| رویکرد | مزیت | معایب |
|---|---|---|
| UI دستی | سریع برای تست | غیرقابل بازتولید، فراموش‌شدنی |
| JCasC | reproducible، version-controlled | منحنی یادگیری |
| Job DSL | انعطاف بالا | پیچیدگی |

**دیدگاه:** هر محیط تولید باید JCasC داشته باشد. "Click-ops" برای Jenkins توصیه نمی‌شود چون پیکربندی در `jenkins_home` پراکنده می‌شود.

**امنیت پلاگین:** پلاگین‌های مهجور (abandoned) را حذف کنید — آن‌ها حفرهٔ امنیتی هستند.

### اشتباهات رایج

- **پلاگین‌های بیش از حد:** هر پلاگین سطح حمله (attack surface) را بیشتر می‌کند.
- **رمز در Jenkinsfile:** با `withCredentials` جایگزین کنید.
- **عدم بکاپ Credentials store:** رمزها را از دست می‌دهید.
- **پلاگین ناسازگار با نسخه:** همیشه LTS را با پلاگین سازگار نگه دارید.
- **دسترسی anonymous read:** در تولید غیرفعال کنید.

### نکات مهم و ترفندها

- **Least privilege:** به کاربران فقط دسترسی لازم را بدهید (Matrix Authorization).
- **Folder credentials:** اعتبارنامه را در پوشهٔ پروژه قرار دهید نه Global.
- **Plugin update تدریجی:** همه را یک‌باره آپدیت نکنید؛ ابتدا در staging.
- **Secret در متغیر محیطی:** با `environment { TOKEN = credentials('id') }` ایمن بخوانید.
- **Audit log:** پلاگین Audit Trail برای ردیابی تغییرات.
- **Backup با ThinBackup:** پلاگینی برای بکاپ سریع پیکربندی.

### خلاصه فصل

پلاگین‌ها قدرت Jenkins هستند اما باید مدیریت شوند. Credentials در store رمزگذاری‌شده با `credentialsId` فراخوانی می‌شوند (هرگز در متن). JCasC پیکربندی را reproducible می‌کند — برای تولید ضروری است. کلید: Least privilege، Folder credentials، بکاپ منظم، و حذف پلاگین‌های مهجور.

---

## فصل ۲۴: تریگرها، Multibranch و یکپارچه‌سازی

### مقدمه

یک pipeline باید **خودکار** اجرا شود — نه با دکمهٔ دستی. **تریگرها (Triggers)** تعیین می‌کنند چه زمانی خط لوله اجرا شود: push جدید، PR، یا زمان‌بندی. **Multibranch Pipeline** امکان داشتن Jenkinsfile مجزا برای هر branch را می‌دهد — سنگ‌بنای CI مدرن.

در این فصل تریگرها، Webhookها، Multibranch و یکپارچه‌سازی با GitHub/GitLab را بررسی می‌کنیم.

### مفاهیم اساسی

**انواع تریگر:**

| تریگر | کاربرد |
|---|---|
| `pollSCM` | Jenkins هر چند دقیقه مخزن را چک می‌کند (قدیمی) |
| **Webhook** | مخزن هنگام push به Jenkins خبر می‌دهد (توصیه) |
| `cron` | زمان‌بندی (مثل شبانه) |
| `upstream` | پس از اتمام pipeline دیگر |
| **GitHub/GitLab Hook** | یکپارچه‌سازی مستقیم با پلتفرم |

**Multibranch Pipeline:**
یک نوع Job که برای **هر branch** که Jenkinsfile دارد，یک pipeline جدا می‌سازد. وقتی branch حذف شود，pipeline هم حذف می‌شود.

**Webhook چیست؟**
آدرسی که Git provider (GitHub/GitLab) با یک POST request هنگام رویداد (push/PR) فراخوانی می‌کند تا Jenkins را بیدار کند.

**Branch indexing:** Multibranch به صورت دوره‌ای branchها را اسکن می‌کند تا Jenkinsfile جدید را بیابد.

### مثال‌های عملی

**۱. تریگر با Webhook در Jenkinsfile:**

```groovy
pipeline {
    agent any
    triggers {
        githubPush()   // نیاز به پلاگین GitHub Integration
    }
    stages {
        stage('Build') { steps { sh 'make' } }
    }
}
```

**۲. Multibranch Pipeline (تعریف در UI یا JCasC):**

```yaml
# JCasC
jobs:
  - script: >
      multibranchPipelineJob('myapp') {
        branchSources {
          git {
            id('myapp')
            url('https://github.com/org/myapp.git')
          }
        }
        orphanedItemStrategy {
          discardOldItems { numToKeep(20) }
        }
      }
```

**۳. Webhook در GitHub:**

```
Repo → Settings → Webhooks → Add
Payload URL: http://jenkins.example.com/github-webhook/
Content type: application/json
Events: Just the push event
```

**۴. شرط اجرا بر اساس branch:**

```groovy
stage('Deploy Prod') {
    when { branch 'main' }
    steps { sh './deploy.sh prod' }
}
stage('Deploy Test') {
    when { branch 'develop' }
    steps { sh './deploy.sh test' }
}
```

### مقایسه و دیدگاه‌های مختلف

| روش | سرعت واکنش | نیاز به پیکربندی |
|---|---|---|
| pollSCM | کند (تأخیر چک) | کم |
| Webhook | فوری | متوسط (URL در Git) |
| GitHub App | فوری + امن | بالا |

**دیدگاه‌ها:**
- Webhook استاندارد طلایی است؛ pollSCM فقط برای مخازن بدون دسترسی webhook.
- GitHub App authentication امن‌تر از توکن ساده است.
- Multibranch برای هر تیمی که branching strategy دارد (فصل ۱۲) توصیه می‌شود.

### اشتباهات رایج

- **pollSCM با فاصلهٔ کم:** بار زیاد روی Git server؛ از webhook استفاده کنید.
- **Webhook به آدرس داخلی:** Jenkins باید از اینترنت (یا شبکهٔ سازمان) قابل دسترس باشد.
- **فراموش کردن `githubPush()` پلاگین:** تریگر کار نمی‌کند.
- **Multibranch بدون Branch Filter:** صدها pipeline بی‌استفاده برای branchهای قدیمی.
- **عدم پاکسازی orphaned:** pipelineهای branch حذف‌شده باید auto-cleanup شوند.

### نکات مهم و ترفندها

- **Branch filtering:** فقط `main`, `develop`, `feature/*` را اسکن کنید.
- **Webhook secret:** در GitHub secret بگذارید و در Jenkins تأیید کنید (امنیت).
- **Pipeline status به Git:** Jenkins نتیجه را به صورت commit status برمی‌گرداند (علامت سبز/قرمز کنار کامیت).
- **GitHub Checks API:** نمایش جزئیات تست مستقیم در PR.
- **Auto-cleanup:** `orphanedItemStrategy { discardOldItems { numToKeep(20) } }`.
- **Shared library + Multibranch:** pipeline یکسان برای همهٔ repoها.

### خلاصه فصل

تریگرها pipeline را خودکار می‌کنند: Webhook (فوری، توصیه) بهتر از pollSCM (کند). Multibranch Pipeline برای هر branch یک خط لوله می‌سازد — سنگ‌بنای CI مدرن. Webhook نیاز به URL در Git provider و پلاگین مربوطه دارد. کلید: Branch filter، Webhook secret، و auto-cleanup branchهای حذف‌شده. وضعیت build به صورت commit status به Git برگردانده شود.

---

## فصل ۲۵: بهترین روش‌ها و عیب‌یابی Jenkins

### مقدمه

Jenkins ابزاری قدرتمند اما «تله‌خیز» است. بدون رعایت بهترین روش‌ها，به سرعت به یک **Snowflake server** (سرور برفی یکتا و غیرقابل بازتولید) تبدیل می‌شود. در این فصل چک‌لیست بهترین روش‌ها و تکنیک‌های عیب‌یابی رایج را می‌آوریم.

### مفاهیم اساسی

**اصول طلایی Jenkins:**
1. **Pipeline-as-Code:** هیچ تنظیمی در UI نماند.
2. **JCasC:** تمام پیکربندی در YAML version-controlled.
3. **Shared Libraries:** کد تکراری در repo جدا.
4. **Agent-based:** اجرا روی Agent، نه Controller.
5. **Ephemeral Agents:** Agentهای موقت (Docker/K8s) بهتر از دائمی.
6. **Least Privilege:** دسترسی حداقلی.
7. **Backup:** `jenkins_home` بکاپ شود.

**عیب‌یابی لایه‌ای:**
- آیا Jenkins اصلاً اجرا می‌شود؟ (سرویس/لاگ)
- آیا Agent متصل است؟ (Nodes)
- آیا Job trigger شد؟ (Build history)
- آیا مرحلهٔ خاص شکست؟ (Console output)
- آیا محیط درست است؟ (ابزارها نصب‌اند؟)

### مثال‌های عملی

**۱. لاگ‌های مفید:**

```bash
# لاگ Jenkins (Docker)
docker logs -f jenkins

# لاگ سیستم (Linux service)
journalctl -u jenkins -f

# لاگ یک build خاص: در UI → Build → Console Output
```

**۲. عیب‌یابی Agent قطع:**

```
Manage Jenkins → Nodes → agent-1
Status: Offline
سبب رایج: SSH ناموفق، فضای دیسک پر، java قدیمی
```

**۳. بازیابی از بکاپ:**

```bash
# توقف Jenkins
docker stop jenkins
# بازگرداندن jenkins_home
cp -r /backup/jenkins_home /var/jenkins_home
docker start jenkins
```

**۴. پاکسازی فضای دیسک:**

```bash
# حذف workspaceهای قدیمی
docker exec jenkins sh -c 'rm -rf /var/jenkins_home/workspace/*'
# یا از پلاگین Workspace Cleanup در pipeline
```

### مقایسه و دیدگاه‌های مختلف

| مشکل | علت رایج | راه حل |
|---|---|---|
| Build هنگ می‌کند | timeout ندارد | `options { timeout }` |
| Agent offline | SSH/DNS | بررسی connectivity |
| پلاگین conflict | نسخه ناسازگار | به LTS برگردانید |
| رمز کار نمی‌کند | credentialsId اشتباه | چک کنید scope درست است |
| Disk full | artifact بی‌پایان | `expire_in` بگذارید |

**دیدگاه مهندسی پلتفرم:** Jenkins را به عنوان «زیرساخت» نگاه کنید نه «اپلیکیشن» — یعنی همه چیز کد باشد (IaC + JCasC).

### اشتباهات رایج

- **تنظیمات فقط در UI:** با خرابی سرور，همه چیز از دست می‌رود.
- **Controller اجراکننده:** بار سنگین روی مغز → کندی کل سیستم.
- **Shared library بدون versioning:** `library 'mylib'` بدون `@tag` → تغییر ناگهانی pipelineها.
- **عدم بکاپ:** فاجعهٔ بازگشت‌ناپذیر.
- **پلاگین‌های زیاد بی‌استفاده:** حفرهٔ امنیتی و کندی.

### نکات مهم و ترفندها

- **استفاده از `@Library` با تگ:**
  ```groovy
  @Library('my-shared-lib@v2.1') _
  ```
- **Pipeline lint:** قبل از commit، Jenkinsfile را با HTTP lint چک کنید.
- **Build discarders:**
  ```groovy
  options { buildDiscarder(logRotator(numToKeepStr: '20')) }
  ```
- **Notifications:** Slack/Email در `post` برای آگاهی تیم.
- **Matrix labels:** Job را به Agent مناسب با label هدایت کنید.
- **مهاجرت تدریجی:** اگر Jenkins قدیمی دارید，ابتدا JCasC را فعال کنید，سپس تنظیمات را به کد منتقل کنید.

### خلاصه فصل

Jenkins با رعایت بهترین روش‌ها (Pipeline-as-Code، JCasC، Shared Libraries، Agent-based، Ephemeral Agents، Least Privilege، Backup) از یک فاجعهٔ دستی به زیرساختی قابل بازتولید تبدیل می‌شود. عیب‌یابی لایه‌ای: سرویس → Agent → Trigger → Stage → Environment. کلید: هیچ تنظیمی در UI نماند، بکاپ `jenkins_home` منظم，و Shared Library همیشه با تگ نسخه.

---

## منابع و مراجع

- Jenkins User Documentation: https://www.jenkins.io/doc/
- Jenkins Configuration as Code (JCasC): https://github.com/jenkinsci/configuration-as-code-plugin
- Pipeline Syntax: https://www.jenkins.io/doc/book/pipeline/syntax/
- Shared Libraries: https://www.jenkins.io/doc/book/pipeline/shared-libraries/
- Docker Jenkins Image: https://hub.docker.com/r/jenkins/jenkins
- Jenkins Plugins Index: https://plugins.jenkins.io
- Blue Ocean: https://www.jenkins.io/projects/blueocean/

**بعدی:** [فایل ۰۶ — GitHub Actions: CI/CD در اکوسیستم GitHub](06-GitHub-Actions.md)
