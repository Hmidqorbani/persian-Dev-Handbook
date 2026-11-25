# بخش ۱۱: Declarative Pipeline

**مدت یادگیری**: ۳.۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۱۰ (Jobها و Freestyle)

---

## فهرست محتوا

- [فصل ۱: Pipeline as Code](#فصل-۱-pipeline-as-code)
- [فصل ۲: ساختار Declarative](#فصل-۲-ساختار-declarative)
- [فصل ۳: environment، options و parameters](#فصل-۳-environment-options-و-parameters)
- [فصل ۴: موازی‌سازی و matrix](#فصل-۴-موازیسازی-و-matrix)
- [فصل ۵: Credentials در Pipeline](#فصل-۵-credentials-در-pipeline)
- [فصل ۶: کنترل خطا و input](#فصل-۶-کنترل-خطا-و-input)
- [فصل ۷: اولین Pipeline کامل](#فصل-۷-اولین-pipeline-کامل)

---

## فصل ۱: Pipeline as Code

### مقدمه

**Pipeline as Code** یعنی تعریف فرآیند CI/CD در یک فایل متنی (`Jenkinsfile`) که در مخزن Git نگه‌داری می‌شود. این رویکرد انقلابی است: Pipeline شما هم نسخه‌بندی، هم بازبینی و هم قابل بازتولید می‌شود.

### اهمیت این موضوع

Pipeline گرافیکی (Freestyle) در Jenkins ذخیره می‌شود، نه در Git. اما `Jenkinsfile` بخشی از پروژه است: با کد سفر می‌کند، در PR بازبینی می‌شود و اگر Jenkins از دست رفت، Pipeline باقی می‌ماند.

### مفاهیم اساسی

**مزایا:**
- نسخه‌بندی خودکار در Git
- بازبینی در Pull Request
- قابل استفاده مجدد در شاخه‌ها
- قابل بازتولید و انتقال
- مستندسازی زنده

**دو سبک:**
- **Declarative**: ساخت‌یافته، ساده‌تر (این بخش).
- **Scripted**: Groovy خالص، انعطاف‌پذیر (بخش ۱۲).

### مثال‌های عملی

حداقل `Jenkinsfile`:

```groovy
pipeline {
    agent any
    stages {
        stage('Hello') {
            steps {
                echo 'Hello, Jenkins!'
            }
        }
    }
}
```

### اشکال شایع

- **اشتباه رایج**: نگه‌داشتن Pipeline در رابط Jenkins به‌جای Git. همیشه از SCM استفاده کنید.
- **اشتباه رایج**: نبودن Jenkinsfile در همه شاخه‌ها.

### نکات مهم (Tips & Tricks)

- `Jenkinsfile` را در ریشه مخزن بگذارید.
- آن را مثل کد بازبینی کنید.
- از `Pipeline Syntax` (ابزار داخلی Jenkins) برای تولید قطعه‌کد استفاده کنید.

### خلاصه فصل

Pipeline as Code یعنی تعریف CI/CD در `Jenkinsfile` که در Git نگه‌داری می‌شود. مزایا: نسخه‌بندی، بازبینی و بازتولیدپذیری.

---

## فصل ۲: ساختار Declarative

### مقدمه

Declarative Pipeline ساختار مشخصی دارد که یادگیری و خواندن را ساده می‌کند.

### مفاهیم اساسی

ساختار کامل:

```groovy
pipeline {
    agent any

    environment {
        APP_NAME = 'myapp'
    }

    options {
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    parameters {
        choice(name: 'ENV', choices: ['dev', 'prod'], description: 'Target')
    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'make build'
            }
        }
    }

    post {
        always { echo 'Done' }
        success { echo 'Success!' }
        failure { echo 'Failed!' }
    }
}
```

**اجزای اصلی:**
- `agent`: کجا اجرا شود.
- `stages`: مراحل.
- `steps`: دستورات.
- `post`: کارهای پایانی.
- `environment`, `options`, `parameters`, `triggers`, `tools`.

### مثال‌های عملی

`agent` با برچسب:

```groovy
agent { label 'linux && docker' }

// یا با Docker
agent {
    docker {
        image 'python:3.12'
        args '-v /tmp:/tmp'
    }
}
```

### اشکال شایع

- **اشتباه رایج**: استفاده از کد Groovy خالص در Declarative بدون `script {}`. در Declarative باید بلوک‌های Groovy داخل `script {}` باشند.
- **اشتباه رایج**: فراموش کردن `agent` در سطح pipeline.

### نکات مهم (Tips & Tricks)

- همیشه `options` برای timeout و discard بگذارید.
- `post` برای اعلان‌ها و پاک‌سازی عالی است.
- ساختار را خوانا نگه دارید.

### خلاصه فصل

Declarative Pipeline ساخت‌یافته است: pipeline → agent/stages/steps/post. هر بخش نقش مشخصی دارد و خوانایی بالایی می‌دهد.

---

## فصل ۳: environment، options و parameters

### مقدمه

این بخش‌ها Pipeline را پیکربندی‌پذیر و قابل کنترل می‌کنند.

### مفاهیم اساسی

**environment:** متغیرهای محیطی.

```groovy
environment {
    APP = 'web'
    VERSION = '1.0.0'
    REGISTRY = 'registry.example.com'
}
```

**options:** تنظیمات رفتار Pipeline.

```groovy
options {
    timeout(time: 30, unit: 'MINUTES')
    buildDiscarder(logRotator(numToKeepStr: '20'))
    disableConcurrentBuilds()
    skipDefaultCheckout()
    timestamps()
    retry(3)
}
```

**parameters:** ورودی‌ها.

```groovy
parameters {
    string(name: 'BRANCH', defaultValue: 'main', description: 'Branch')
    choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: 'Env')
    booleanParam(name: 'SKIP_TESTS', defaultValue: false, description: 'Skip?')
}
```

**tools:** ابزارهای موردنیاز.

```groovy
tools {
    maven 'maven-3.9'
    jdk 'jdk17'
}
```

### مثال‌های عملی

```groovy
pipeline {
    agent any
    environment {
        DOCKER_REGISTRY = 'docker.io'
    }
    options {
        timeout(time: 20, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '15'))
    }
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'prod'], description: 'Deploy target')
    }
    stages {
        stage('Info') {
            steps {
                echo "Deploying to ${params.ENVIRONMENT}"
                sh 'echo $DOCKER_REGISTRY'
            }
        }
    }
}
```

### اشکال شایع

- **اشتباه رایج**: استفاده از `params.X` قبل از تعریف parameters. خطا می‌دهد.
- **اشتباه رایج**: نداشتن timeout. Pipeline می‌تواند بی‌نهایت گیر کند.

### نکات مهم (Tips & Tricks)

- `disableConcurrentBuilds()` از اجرای همزمان جلوگیری می‌کند.
- برای Secretها از `credentials()` استفاده کنید، نه `environment`.
- `timestamps()` لاگ‌ها را زمان‌دار می‌کند.

### خلاصه فصل

environment متغیرها، options تنظیمات، parameters ورودی‌ها و tools ابزارها را تعریف می‌کنند. همیشه timeout و discard بگذارید.

---

## فصل ۴: موازی‌سازی و matrix

### مقدمه

موازی‌سازی زمان Pipeline را به‌شدت کم می‌کند.

### مفاهیم اساسی

**parallel:** اجرای همزمان مراحل.

```groovy
stage('Tests') {
    parallel {
        stage('Unit') {
            steps { sh 'pytest tests/unit' }
        }
        stage('Integration') {
            steps { sh 'pytest tests/integration' }
        }
        stage('Lint') {
            steps { sh 'flake8 .' }
        }
    }
}
```

**matrix:** اجرا روی ترکیب‌های مختلف (نسخه زبان، OS).

```groovy
stage('Test Matrix') {
    matrix {
        axes {
            axis {
                name 'PYTHON_VERSION'
                values '3.10', '3.11', '3.12'
            }
            axis {
                name 'OS'
                values 'linux', 'windows'
            }
        }
        agent {
            docker { image "python:${PYTHON_VERSION}" }
        }
        stages {
            stage('Test') {
                steps {
                    sh 'pytest'
                }
            }
        }
    }
}
```

### مقایسه و الگوهای مختلف

| روش | کاربرد |
|-----|--------|
| parallel | مراحل مستقل |
| matrix | ترکیب پارامترها |
| sequential (پیش‌فرض) | وابستگی |

### اشکال شایع

- **اشتباه رایج**: موازی کردن مراحل وابسته. فقط مستقل‌ها را موازی کنید.
- **اشتباه رایج**: استفاده زیاد از parallel و اشباع Agentها.

### نکات مهم (Tips & Tricks)

- `failFast true` در parallel: با اولین شکست، بقیه را متوقف می‌کند.
- matrix برای تست سازگاری عالی است.
- تعداد Executor را متناسب با parallel تنظیم کنید.

### خلاصه فصل

parallel مراحل مستقل را همزمان اجرا می‌کند و matrix ترکیب پارامترها را پوشش می‌دهد. این دو زمان Pipeline را به‌شدت کاهش می‌دهند.

---

## فصل ۵: Credentials در Pipeline

### مقدمه

Pipelineها اغلب به رمز، توکن و کلید نیاز دارند. مدیریت امن آن‌ها حیاتی است.

### مفاهیم اساسی

```groovy
environment {
    // اعتبارنامه از نوع Secret Text
    API_TOKEN = credentials('api-token-id')

    // نام کاربری/رمز (دو متغیر می‌سازد: _USR و _PSW)
    DOCKER_CREDS = credentials('docker-hub')
}
```

`credentials('id')` برای Username/Password سه متغیر می‌سازد:
- `VAR` = `username:password`
- `VAR_USR` = username
- `VAR_PSW` = password

**انواع Credentials:** Secret Text، Username/Password، SSH Key، Secret File، Certificate.

### مثال‌های عملی

```groovy
pipeline {
    agent any
    environment {
        DOCKER_CREDS = credentials('dockerhub-creds')
    }
    stages {
        stage('Login & Push') {
            steps {
                sh '''
                    echo "$DOCKER_CREDS_PSW" | docker login -u "$DOCKER_CREDS_USR" --password-stdin
                    docker push myapp:latest
                '''
            }
        }
    }
}
```

استفاده در `withCredentials`:

```groovy
withCredentials([string(credentialsId: 'api-token', variable: 'TOKEN')]) {
    sh 'curl -H "Authorization: Bearer $TOKEN" https://api.example.com'
}
```

### اشکال شایع

- **اشتباه رایج**: Hardcode کردن رمز در Jenkinsfile. هرگز!
- **اشتباه رایج**: echo کردن رمز در لاگ. Jenkins آن را mask می‌کند، اما محتاط باشید.

### نکات مهم (Tips & Tricks)

- Secretها را در `Manage Credentials` بسازید.
- از `withCredentials` برای محدود کردن دامنه استفاده کنید.
- Credentials ID را معنادار بگذارید.

### خلاصه فصل

Credentials رمزها را امن نگه می‌دارد. با `credentials()` در environment یا `withCredentials` استفاده کنید. هرگز رمز را Hardcode نکنید.

---

## فصل ۶: کنترل خطا و input

### مقدمه

Pipeline باید در برابر خطا مقاوم باشد و گاهی منتظر تأیید انسان بماند.

### مفاهیم اساسی

**post:** کارهای پس از اجرا.

```groovy
post {
    always { echo 'Always runs' }
    success { echo 'Success' }
    failure { echo 'Failed' }
    unstable { echo 'Unstable' }
    changed { echo 'Status changed' }
    cleanup { deleteDir() }
}
```

**retry:** تلاش مجدد.

```groovy
retry(3) {
    sh 'flaky-command'
}
```

**timeout:**

```groovy
timeout(time: 5, unit: 'MINUTES') {
    sh 'long-command'
}
```

**input:** تأیید دستی.

```groovy
stage('Deploy to Prod') {
    steps {
        input message: 'Deploy to production?', ok: 'Deploy'
        sh './deploy.sh prod'
    }
}
```

### مثال‌های عملی

```groovy
stage('Deploy') {
    when { branch 'main' }
    steps {
        timeout(time: 10, unit: 'MINUTES') {
            script {
                if (params.ENVIRONMENT == 'prod') {
                    input 'Approve production deployment?'
                }
            }
            sh './deploy.sh'
        }
    }
    post {
        failure {
            slackSend channel: '#alerts', message: "Deploy failed!"
        }
    }
}
```

### اشکال شایع

- **اشتباه رایج**: input در Pipeline خودکار. جلوی اتوماسیون را می‌گیرد.
- **اشتباه رایج**: نداشتن cleanup. فایل‌های موقت باقی می‌مانند.

### نکات مهم (Tips & Tricks)

- input را فقط برای محیط تولید بگذارید.
- `when` برای شرطی کردن مرحله.
- `cleanup` همیشه اجرا می‌شود، حتی بعد از شکست.

### خلاصه فصل

post، retry، timeout و input ابزارهای کنترل خطا و جریان‌اند. `when` مرحله را شرطی می‌کند. همیشه `cleanup` بگذارید.

---

## فصل ۷: اولین Pipeline کامل

### مقدمه

بیایید همه‌چیز را در یک Pipeline واقعی ترکیب کنیم.

### مثال‌های عملی

`Jenkinsfile` کامل:

```groovy
pipeline {
    agent any

    environment {
        APP_NAME = 'myapp'
        DOCKER_IMAGE = "myapp:${env.BUILD_NUMBER}"
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '20'))
        timestamps()
        disableConcurrentBuilds()
    }

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Deploy target')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'git log --oneline -3'
            }
        }

        stage('Build & Test') {
            parallel {
                stage('Lint') {
                    steps { sh 'echo "Running linter..."' }
                }
                stage('Unit Tests') {
                    steps { sh 'echo "Running unit tests..."' }
                }
            }
        }

        stage('Package') {
            steps {
                sh 'echo "Building Docker image ${DOCKER_IMAGE}"'
            }
        }

        stage('Deploy') {
            when { branch 'main' }
            steps {
                script {
                    if (params.ENVIRONMENT == 'prod') {
                        input 'Deploy to production?'
                    }
                }
                sh "echo 'Deploying ${APP_NAME} to ${params.ENVIRONMENT}'"
            }
        }
    }

    post {
        always {
            echo "Pipeline finished with status: ${currentBuild.currentResult}"
        }
        success {
            echo '✅ Deployment succeeded'
        }
        failure {
            echo '❌ Deployment failed'
        }
        cleanup {
            cleanWs()
        }
    }
}
```

### اشکال شایع

- **اشتباه رایج**: Pipeline غول‌پیکر بدون تست مرحله‌به‌مرحله. تدریجی بسازید.
- **اشتباه رایج**: نداشتن `when` برای محافظت از تولید.

### نکات مهم (Tips & Tricks)

- Pipeline را در Git نگه دارید و در PR بازبینی کنید.
- از `cleanWs()` برای پاک‌سازی Workspace استفاده کنید.
- Pipeline را با تغییرات کوچک تست کنید.

### خلاصه فصل

یک Pipeline کامل شامل Checkout، Build/Test موازی، Package، Deploy شرطی و post است. این ساختار را می‌توانید برای پروژه‌های واقعی گسترش دهید.

---

## منابع و مراجع

- Declarative Pipeline Syntax: [https://www.jenkins.io/doc/book/pipeline/syntax/](https://www.jenkins.io/doc/book/pipeline/syntax/)
- Pipeline Syntax Generator: [http://localhost:8080/pipeline-syntax](http://localhost:8080/pipeline-syntax)
- Credentials in Pipeline: [https://www.jenkins.io/doc/book/pipeline/jenkinsfile/#handling-credentials](https://www.jenkins.io/doc/book/pipeline/jenkinsfile/#handling-credentials)
- Best Practices: [https://www.jenkins.io/doc/book/pipeline/best-practices/](https://www.jenkins.io/doc/book/pipeline/best-practices/)
