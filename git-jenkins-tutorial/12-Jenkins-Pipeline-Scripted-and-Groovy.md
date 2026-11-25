# بخش ۱۲: Scripted Pipeline و Groovy

**مدت یادگیری**: ۳ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۱۱ (Declarative Pipeline)

---

## فهرست محتوا

- [فصل ۱: Scripted Pipeline](#فصل-۱-scripted-pipeline)
- [فصل ۲: مبانی Groovy](#فصل-۲-مبانی-groovy)
- [فصل ۳: node، stage و step](#فصل-۳-node-stage-و-step)
- [فصل ۴: متغیرها و توابع](#فصل-۴-متغیرها-و-توابع)
- [فصل ۵: مدیریت استثنا](#فصل-۵-مدیریت-استثنا)
- [فصل ۶: Declarative vs Scripted](#فصل-۶-declarative-vs-scripted)
- [فصل ۷: بهترین روش‌ها](#فصل-۷-بهترین-روشها)

---

## فصل ۱: Scripted Pipeline

### مقدمه

**Scripted Pipeline** سبک قدیمی‌تر و انعطاف‌پذیرتر Jenkins است. برخلاف Declarative که ساختار ثابتی دارد، Scripted یک اسکریپت Groovy خالص است که با `node {}` شروع می‌شود.

### مفاهیم اساسی

حداقل Scripted Pipeline:

```groovy
node {
    stage('Checkout') {
        checkout scm
    }
    stage('Build') {
        sh 'make build'
    }
    stage('Test') {
        sh 'make test'
    }
}
```

تفاوت بنیادی: در Scripted، شما آزادید هر منطق Groovy را بنویسید — حلقه، شرط، تابع، try/catch.

### مقایسه و الگوهای مختلف

| جنبه | Declarative | Scripted |
|------|-------------|----------|
| شروع | `pipeline {}` | `node {}` |
| ساختار | ثابت | آزاد |
| یادگیری | آسان‌تر | سخت‌تر |
| انعطاف | محدودتر | نامحدود |
| توصیه | پیش‌فرض | موارد پیچیده |

### اشکال شایع

- **اشتباه رایج**: انتخاب Scripted برای کارهای ساده. Declarative کافی است.
- **اشتباه رایج**: فراموش کردن `node {}`. بدون آن هیچ کاری اجرا نمی‌شود.

### نکات مهم (Tips & Tricks)

- Scripted را وقتی انتخاب کنید که به منطق پیچیده (حلقه، شرط پویا) نیاز دارید.
- می‌توانید در Declarative هم با `script {}` از Groovy استفاده کنید.

### خلاصه فصل

Scripted Pipeline یک اسکریپت Groovy خالص است که با `node {}` شروع می‌شود. انعطاف بیشتری دارد اما ساختار کمتری ارائه می‌دهد.

---

## فصل ۲: مبانی Groovy

### مقدمه

Groovy زبانی است که Jenkins Pipeline بر پایه آن ساخته شده. آشنایی با مبانی آن ضروری است.

### مفاهیم اساسی

**متغیرها:**
```groovy
def name = 'Ali'
String city = 'Tehran'
int count = 5
def list = [1, 2, 3]
def map = [name: 'Ali', age: 30]
```

**رشته‌ها:**
```groovy
def a = 'single quotes'
def b = "double quotes with ${a}"
def multiline = """
    خط اول
    خط دوم
"""
```

**شرط‌ها:**
```groovy
if (env.BRANCH_NAME == 'main') {
    echo 'On main branch'
} else if (env.BRANCH_NAME == 'develop') {
    echo 'On develop'
} else {
    echo 'Other branch'
}
```

**حلقه‌ها:**
```groovy
for (int i = 0; i < 3; i++) {
    echo "Iteration $i"
}

[1, 2, 3].each { item ->
    echo "Item: $item"
}
```

**توابع:**
```groovy
def greet(String name) {
    return "Hello, ${name}!"
}

echo greet('Jenkins')
```

### مثال‌های عملی

```groovy
node {
    stage('Dynamic Stages') {
        def environments = ['dev', 'staging', 'prod']
        for (env_name in environments) {
            stage("Deploy to ${env_name}") {
                echo "Deploying to ${env_name}"
            }
        }
    }
}
```

### اشکال شایع

- **اشتباه رایج**: استفاده از `def` برای متغیرهای محیطی. `def x = 'val'` متغیر محلی Groovy است، نه env.
- **اشتباه رایج**: تداخل نوع داده. Groovy پویا است؛ مراقب باشید.

### نکات مهم (Tips & Tricks)

- Groovy در Jenkins به‌صورت sandboxed اجرا می‌شود؛ برخی متدها نیاز به تأیید ادمین دارند.
- از `def` برای متغیرهای محلی استفاده کنید.
- `env.VAR` برای متغیرهای محیطی.

### خلاصه فصل

Groovy زبان پایه Pipeline است. متغیر، رشته، شرط، حلقه و تابع را یاد بگیرید. `def` برای متغیر محلی و `env` برای محیطی است.

---

## فصل ۳: node، stage و step

### مقدمه

سه بلوک اصلی Scripted Pipeline را می‌شناسیم.

### مفاهیم اساسی

**node:** تخصیص یک Executor روی Agent.

```groovy
node('linux') {          // روی Agent با برچسب linux
    // کار
}

node {                   // هر Agent موجود
    // کار
}
```

**stage:** گروه‌بندی منطقی (برای نمایش).

```groovy
stage('Build') {
    // steps
}
```

**step:** دستور واقعی.

```groovy
sh 'make'                // اجرای shell
echo 'message'           // چاپ
git 'https://...'        // Git
checkout scm             // Checkout
```

### مثال‌های عملی

```groovy
node('linux') {
    stage('Checkout') {
        checkout scm
    }

    stage('Build') {
        sh 'docker build -t myapp .'
    }

    stage('Test') {
        sh 'docker run myapp pytest'
    }
}
```

### اشکال شایع

- **اشتباه رایج**: همه کارها در یک `node`. برای کارهای طولانی، node را مدیریت کنید.
- **اشتباه رایج**: نگه‌داشتن Executor در حین انتظار (input). از `input` خارج از node استفاده کنید.

### نکات مهم (Tips & Tricks)

- `node('label')` برای انتخاب Agent خاص.
- Workspace در node ساخته می‌شود.
- برای کارهای موازی، از `parallel` استفاده کنید.

### خلاصه فصل

node یک Executor تخصیص می‌دهد، stage گروه‌بندی می‌کند و step کار واقعی را انجام می‌دهد. برای کارهای طولانی، زمان نگه‌داری node را کم کنید.

---

## فصل ۴: متغیرها و توابع

### مقدمه

در Scripted Pipeline می‌توانید توابع و متغیرهای سفارشی بسازید.

### مفاهیم اساسی

**متغیرهای محیطی:**
```groovy
env.MY_VAR = 'value'
echo env.MY_VAR

// متغیرهای از پیش موجود
echo env.BUILD_NUMBER
echo env.BRANCH_NAME
echo env.WORKSPACE
echo env.JOB_NAME
```

**توابع سفارشی:**
```groovy
def buildAndTest(String service) {
    stage("Build ${service}") {
        sh "docker build -t ${service} ./services/${service}"
    }
    stage("Test ${service}") {
        sh "docker run ${service} pytest"
    }
}

node {
    ['auth', 'payment', 'gateway'].each { svc ->
        buildAndTest(svc)
    }
}
```

**بازگشت مقدار:**
```groovy
def getVersion() {
    return sh(script: 'cat version.txt', returnStdout: true).trim()
}

def version = getVersion()
echo "Version: ${version}"
```

### مثال‌های عملی

```groovy
def runTests(String type) {
    sh "pytest tests/${type}"
    return currentBuild.result ?: 'SUCCESS'
}

node {
    def result = runTests('unit')
    echo "Unit tests: ${result}"
}
```

### اشکال شایع

- **اشتباه رایج**: استفاده از توابع بدون `def`. باعث مشکل scope می‌شود.
- **اشتباه رایج**: فراموش کردن `.trim()` روی خروجی `sh`. شامل newline است.

### نکات مهم (Tips & Tricks)

- برای منطق پیچیده، توابع بسازید.
- `returnStdout: true` برای گرفتن خروجی دستور.
- `returnStatus: true` برای گرفتن کد خروجی.

### خلاصه فصل

متغیرهای محیطی با `env` و توابع سفارشی با `def` تعریف می‌شوند. توابع، Pipeline را تمیز و قابل استفاده مجدد می‌کنند.

---

## فصل ۵: مدیریت استثنا

### مقدمه

در Scripted Pipeline، کنترل خطا با `try/catch/finally` انجام می‌شود.

### مفاهیم اساسی

```groovy
node {
    try {
        stage('Deploy') {
            sh './deploy.sh'
        }
        currentBuild.result = 'SUCCESS'
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        echo "Error: ${e.message}"
        // اعلان خطا
        slackSend message: "Deploy failed: ${e.message}"
        throw e     // اگر می‌خواهید Pipeline شکست بخورد
    } finally {
        // همیشه اجرا می‌شود
        cleanWs()
    }
}
```

### مثال‌های عملی

```groovy
node {
    def deployed = false
    try {
        stage('Deploy') {
            sh './deploy.sh'
            deployed = true
        }
    } catch (e) {
        echo 'Deployment failed, rolling back...'
        sh './rollback.sh'
    } finally {
        if (!deployed) {
            echo 'Not deployed'
        }
    }
}
```

### مقایسه و الگوهای مختلف

| بلوک | زمان اجرا |
|------|-----------|
| try | همیشه |
| catch | هنگام خطا |
| finally | همیشه (حتی بعد از خطا) |

### اشکال شایع

- **اشتباه رایج**: `catch` کردن بدون لاگ. خطا را پنهان می‌کنید.
- **اشتباه رایج**: فراموش کردن `finally` برای پاک‌سازی.

### نکات مهم (Tips & Tricks)

- `currentBuild.result` را تنظیم کنید.
- `throw e` را دوباره پرتاب کنید تا Pipeline قرمز شود.
- `finally` برای cleanup عالی است.

### خلاصه فصل

try/catch/finally خطاها را مدیریت می‌کند. `finally` همیشه اجرا می‌شود. خطاها را لاگ کنید و در صورت نیاز دوباره پرتاب کنید.

---

## فصل ۶: Declarative vs Scripted

### مقدمه

کدام سبک را انتخاب کنیم؟ پاسخ به پیچیدگی نیاز بستگی دارد.

### مفاهیم اساسی

**Declarative را انتخاب کنید اگر:**
- Pipeline استاندارد و ساخت‌یافته است.
- می‌خواهید خوانایی بالا داشته باشید.
- تیم شما تازه‌کار است.
- به قابلیت‌های داخلی (post، when، matrix) نیاز دارید.

**Scripted را انتخاب کنید اگر:**
- منطق پیچیده و پویا دارید (حلقه‌های تودرتو، شرط‌های پیچیده).
- به انعطاف کامل Groovy نیاز دارید.
- در حال مهاجرت از Pipelineهای قدیمی هستید.

**راه سوم:** Declarative با `script {}` — بهترین هر دو جهان:

```groovy
stage('Complex') {
    steps {
        script {
            def branches = ['a', 'b']
            branches.each { b ->
                echo "Processing ${b}"
            }
        }
    }
}
```

### مقایسه و الگوهای مختلف

| نیاز | توصیه |
|------|-------|
| Pipeline ساده | Declarative |
| منطق پویا | Declarative + script |
| پیچیدگی زیاد | Scripted |
| کتابخانه مشترک | Groovy (هر دو) |

### اشکال شایع

- **اشتباه رایج**: استفاده از Scripted وقتی Declarative کافی است.
- **اشتباه رایج**: نوشتن کل Pipeline در `script {}`. Declarative را از دست می‌دهید.

### نکات مهم (Tips & Tricks)

- توصیه مدرن: **Declarative با script در جاهای لازم**.
- کد مشترک را به Shared Library منتقل کنید (بخش ۱۳).

### خلاصه فصل

Declarative ساخت‌یافته و خوانا، Scripted انعطاف‌پذیر و قدرتمند است. راه میانه: Declarative با `script {}` برای موارد پیچیده.

---

## فصل ۷: بهترین روش‌ها

### مقدمه

چند اصل که Pipeline شما را حرفه‌ای می‌کند.

### مفاهیم اساسی

**Best Practices:**
1. Pipeline را در Git نگه دارید (`Jenkinsfile`).
2. از Declarative شروع کنید.
3. Secretها را در Credentials بگذارید.
4. از Shared Library برای کد تکراری استفاده کنید.
5. timeout و discard بگذارید.
6. Workspace را پاک کنید (`cleanWs()`).
7. توابع کوچک و متمرکز بسازید.
8. لاگ معنادار بنویسید.
9. Pipeline را بازبینی کنید.
10. شکست‌ها را سریع رفع کنید.

**ضدالگوها:**
- Hardcode کردن رمز و مسیر.
- Pipeline غول‌پیکر بدون مرحله‌بندی.
- `sleep` طولانی به‌جای polling درست.
- نادیده گرفتن نتیجه تست.
- کپی-پیست کد بین Pipelineها.

### مثال‌های عملی

ساختار تمیز:

```groovy
@Library('my-shared-lib') _

pipeline {
    agent any
    stages {
        stage('CI') {
            steps {
                script {
                    def builder = new com.example.Builder(this)
                    builder.buildAndTest()
                }
            }
        }
    }
}
```

### نکات مهم (Tips & Tricks)

- از `Pipeline Syntax` برای تولید قطعه‌کد استفاده کنید.
- Lint کردن Jenkinsfile با `jenkinsfile-runner` یا IDE.
- نسخه‌بندی Shared Library.

### خلاصه فصل

بهترین روش‌ها: Pipeline در Git، Declarative، Credentials، Shared Library، timeout و cleanup. از ضدالگوها مثل Hardcode و کپی-پیست پرهیز کنید.

---

## منابع و مراجع

- Scripted Pipeline: [https://www.jenkins.io/doc/book/pipeline/#scripted-pipeline-fundamentals](https://www.jenkins.io/doc/book/pipeline/#scripted-pipeline-fundamentals)
- Groovy Documentation: [https://groovy-lang.org/documentation.html](https://groovy-lang.org/documentation.html)
- Pipeline Best Practices: [https://www.jenkins.io/doc/book/pipeline/best-practices/](https://www.jenkins.io/doc/book/pipeline/best-practices/)
- Jenkins Pipeline Steps Reference: [https://www.jenkins.io/doc/pipeline/steps/](https://www.jenkins.io/doc/pipeline/steps/)
