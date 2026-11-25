# بخش ۱۳: Multi-Branch و Shared Libraries

**مدت یادگیری**: ۳ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۱۲ (Scripted Pipeline و Groovy)

---

## فهرست محتوا

- [فصل ۱: Multi-Branch Pipeline](#فصل-۱-multi-branch-pipeline)
- [فصل ۲: Organization Folders](#فصل-۲-organization-folders)
- [فصل ۳: ساختار Shared Library](#فصل-۳-ساختار-shared-library)
- [فصل ۴: Global Variables و Steps](#فصل-۴-global-variables-و-steps)
- [فصل ۵: بارگذاری داینامیک](#فصل-۵-بارگذاری-داینامیک)
- [فصل ۶: طراحی کتابخانه سازمانی](#فصل-۶-طراحی-کتابخانه-سازمانی)

---

## فصل ۱: Multi-Branch Pipeline

### مقدمه

**Multi-Branch Pipeline** یک نوع Job است که به‌طور خودکار تمام شاخه‌های مخزن را کشف می‌کند و برای هر شاخه (و PR) یک Pipeline جداگانه می‌سازد — به شرطی که `Jenkinsfile` در آن شاخه وجود داشته باشد.

### اهمیت این موضوع

در یک تیم، ده‌ها شاخه و PR وجود دارد. ساخت دستی Job برای هرکدام غیرممکن است. Multi-Branch این کار را خودکار می‌کند: با push یک شاخه جدید، Jenkins خودش آن را کشف و Pipeline را اجرا می‌کند.

### مفاهیم اساسی

ویژگی‌ها:
- کشف خودکار شاخه‌ها از مخزن.
- هر شاخه یک Pipeline جدا.
- کشف Pull Requestها (اگر افزونه نصب باشد).
- حذف خودکار Job شاخه‌های حذف‌شده.

```
Multi-Branch Job
├── main       → Jenkinsfile → Pipeline
├── develop    → Jenkinsfile → Pipeline
├── feature/A  → Jenkinsfile → Pipeline
└── PR-42      → Jenkinsfile → Pipeline
```

### مثال‌های عملی

**ساخت:**
1. New Item → نام → **Multibranch Pipeline**
2. Branch Sources → Add source → Git
3. Project Repository: URL مخزن
4. Credentials
5. Behaviours (Discover branches, Discover PRs)
6. Build Configuration → by Jenkinsfile
7. Save

Jenkinsfile در ریشه مخزن:

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo "Building branch: ${env.BRANCH_NAME}"
            }
        }
    }
}
```

### مقایسه و الگوهای مختلف

| نوع | کشف شاخه | مناسب برای |
|-----|-----------|-------------|
| Pipeline | دستی | یک شاخه |
| Multi-Branch | خودکار | همه شاخه‌ها |
| Organization Folder | خودکار | همه مخازن |

### اشکال شایع

- **اشتباه رایج**: نبودن Jenkinsfile در شاخه. آن شاخه نادیده گرفته می‌شود.
- **اشتباه رایج**: نداشتن دسترسی به PRها. افزونه GitHub Branch Source را نصب کنید.

### نکات مهم (Tips & Tricks)

- از `when { branch 'main' }` برای مرحله‌های مخصوص استفاده کنید.
- برای PRها از `when { changeRequest() }` استفاده کنید.
- `env.BRANCH_NAME` و `env.CHANGE_ID` را بشناسید.

### خلاصه فصل

Multi-Branch Pipeline به‌طور خودکار شاخه‌ها و PRها را کشف و برای هرکدام Pipeline اجرا می‌کند. شرط وجود Jenkinsfile در شاخه است.

---

## فصل ۲: Organization Folders

### مقدمه

**Organization Folder** یک سطح بالاتر از Multi-Branch است: تمام مخازن یک سازمان/کاربر GitHub را کشف می‌کند و برای هرکدام Multi-Branch می‌سازد.

### مفاهیم اساسی

```
Organization Folder (github.com/myorg)
├── repo-1
│   ├── main
│   └── feature/x
├── repo-2
│   ├── main
│   └── develop
└── repo-3
    └── main
```

### مثال‌های عملی

1. New Item → **Organization Folder**
2. Branch Sources → GitHub Organization
3. Owner: نام سازمان
4. Credentials (توکن با دسترسی repo)
5. Behaviours: کشف مخازن
6. Save → Scan Organization

### اشکال شایع

- **اشتباه رایج**: توکن با دسترسی ناکافی. نیاز به `repo` و `admin:org` دارد.
- **اشتباه رایج**: کشف مخازن نامرتبط. از فیلترها استفاده کنید.

### نکات مهم (Tips & Tricks)

- برای سازمان‌های بزرگ بسیار کارآمد است.
- با GitHub App (به‌جای توکن) امن‌تر است.
- Scan را زمان‌بندی کنید.

### خلاصه فصل

Organization Folder تمام مخازن یک سازمان را کشف می‌کند. برای مدیریت متمرکز تیم‌های بزرگ ایده‌آل است.

---

## فصل ۳: ساختار Shared Library

### مقدمه

**Shared Library** مکانیزمی است که کد Pipeline را بین پروژه‌ها به اشتراک می‌گذارد. به‌جای کپی-پیست، کد مشترک را یک‌جا نگه می‌دارید.

### مفاهیم اساسی

ساختار استاندارد:

```
(root)
├── src/                    # کلاس‌های Groovy
│   └── org/
│       └── example/
│           └── Utils.groovy
├── vars/                   # Global Variables (steps)
│   ├── buildApp.groovy
│   └── sayHello.groovy
├── resources/              # فایل‌های غیر-Groovy
│   └── config.json
└── README.md
```

### مثال‌های عملی

**پیکربندی در Jenkins:**
1. Manage Jenkins → System → Global Pipeline Libraries
2. Name: `my-shared-lib`
3. Default version: `main`
4. Retrieval method: Modern SCM → Git → URL
5. Save

**استفاده:**

```groovy
@Library('my-shared-lib') _

pipeline {
    agent any
    stages {
        stage('Hello') {
            steps {
                sayHello('World')
            }
        }
    }
}
```

### اشکال شایع

- **اشتباه رایج**: نبودن `_` بعد از `@Library`. بدون آن، خطا می‌دهد.
- **اشتباه رایج**: ساختار پوشه اشتباه. `vars/` و `src/` باید در ریشه باشند.

### نکات مهم (Tips & Tricks)

- Library را در یک مخزن Git جدا نگه دارید.
- نسخه‌بندی کنید (tag/branch).
- با `@Library('lib@v1.0')` نسخه خاص را قفل کنید.

### خلاصه فصل

Shared Library کد مشترک Pipeline را به اشتراک می‌گذارد. ساختار: `src/`، `vars/`، `resources/`. با `@Library` استفاده می‌شود.

---

## فصل ۴: Global Variables و Steps

### مقدمه

فایل‌های در `vars/` به‌صورت Global Variable در دسترس‌اند.

### مفاهیم اساسی

**`vars/sayHello.groovy`:**
```groovy
def call(String name = 'World') {
    echo "Hello, ${name}!"
}
```

استفاده: `sayHello('Ali')`.

**`vars/buildApp.groovy`:**
```groovy
def call(Map config = [:]) {
    def image = config.image ?: 'myapp'
    def tag = config.tag ?: 'latest'

    stage('Build') {
        sh "docker build -t ${image}:${tag} ."
    }
    stage('Push') {
        sh "docker push ${image}:${tag}"
    }
}
```

استفاده:
```groovy
buildApp(image: 'myapp', tag: env.BUILD_NUMBER)
```

### مثال‌های عملی

Step پیچیده با `withCredentials`:

```groovy
// vars/deployToK8s.groovy
def call(Map config) {
    withCredentials([file(credentialsId: config.kubeconfig, variable: 'KUBECONFIG')]) {
        sh """
            kubectl set image deployment/${config.deployment} \
                ${config.container}=${config.image}
        """
    }
}
```

### اشکال شایع

- **اشتباه رایج**: استفاده از `def` در بالای فایل vars. متد باید `call` باشد.
- **اشتباه رایج**: پارامترهای بدون مقدار پیش‌فرض. همیشه پیش‌فرض امن بگذارید.

### نکات مهم (Tips & Tricks)

- نام فایل = نام step. `buildApp.groovy` → `buildApp()`.
- برای پارامترهای متعدد از `Map` استفاده کنید.
- مستندسازی کنید (`README` و کامنت).

### خلاصه فصل

فایل‌های `vars/` به stepهای جهانی تبدیل می‌شوند. متد `call` نقطه ورود است. با Map پارامترهای متعدد بپذیرید.

---

## فصل ۵: بارگذاری داینامیک

### مقدمه

کلاس‌های در `src/` را می‌توانید در Pipeline استفاده کنید و حتی به‌صورت داینامیک بارگذاری کنید.

### مفاهیم اساسی

**`src/org/example/Docker.groovy`:**
```groovy
package org.example

class Docker implements Serializable {
    def script

    Docker(script) {
        this.script = script
    }

    def build(String image, String tag) {
        script.sh "docker build -t ${image}:${tag} ."
    }

    def push(String image, String tag) {
        script.sh "docker push ${image}:${tag}"
    }
}
```

استفاده:
```groovy
@Library('my-shared-lib') _

pipeline {
    agent any
    stages {
        stage('Docker') {
            steps {
                script {
                    def docker = new org.example.Docker(this)
                    docker.build('myapp', env.BUILD_NUMBER)
                    docker.push('myapp', env.BUILD_NUMBER)
                }
            }
        }
    }
}
```

**بارگذاری داینامیک:**
```groovy
def lib = library identifier: 'my-shared-lib@main', retriever: modernSCM(...)
```

### اشکال شایع

- **اشتباه رایج**: کلاس‌های non-Serializable. Pipeline ممکن است در restart شکست بخورد. همیشه `implements Serializable`.
- **اشتباه رایج**: استفاده مستقیم از `sh` در کلاس. باید `script.sh` باشد.

### نکات مهم (Tips & Tricks)

- کلاس‌ها باید `Serializable` باشند.
- `this` (شیء script) را به کلاس بدهید.
- برای منطق پیچیده، کلاس بهتر از step است.

### خلاصه فصل

کلاس‌های `src/` منطق پیچیده را کپسوله می‌کنند. باید `Serializable` باشند و شیء script را بگیرند. بارگذاری داینامیک انعطاف می‌دهد.

---

## فصل ۶: طراحی کتابخانه سازمانی

### مقدمه

یک Shared Library خوب، استاندارد سازمان شما را کد می‌کند.

### مفاهیم اساسی

اصول طراحی:
1. **تک‌مسئولیتی**: هر step یک کار.
2. **قابل پیکربندی**: با Map پارامتر بگیرید.
3. **مقدار پیش‌فرض امن**: مقادیر sensible.
4. **مستندسازی**: README برای هر step.
5. **نسخه‌بندی**: tag برای release.
6. **تست**: Library را تست کنید.
7. **بازبینی**: مثل کد معمولی.

### مثال‌های عملی

Pipeline استاندارد سازمان:

```groovy
@Library('company-pipeline-lib@v2.1.0') _

standardPipeline(
    language: 'python',
    testCommand: 'pytest --cov',
    dockerImage: 'python:3.12',
    deployEnvironments: ['staging', 'prod']
)
```

که در پشت صحنه، همه مراحل استاندارد (lint، test، build، scan، deploy) را اجرا می‌کند.

### اشکال شایع

- **اشتباه رایج**: Library غول‌پیکر و پیچیده. ماژولار بسازید.
- **اشتباه رایج**: نبود نسخه‌بندی. تغییرات ناگهانی همه پروژه‌ها را می‌شکند.

### نکات مهم (Tips & Tricks)

- Library را مثل یک محصول نگه‌داری کنید.
- تغییرات breaking را در major version بگذارید.
- مستندات و مثال بگذارید.
- از `retrieveMethod` برای نسخه‌های مختلف استفاده کنید.

### خلاصه فصل

Shared Library سازمانی استانداردهای تیم را کد می‌کند. اصول: تک‌مسئولیتی، قابل پیکربندی، نسخه‌بندی و مستندسازی. Library را مثل محصول نگه‌داری کنید.

---

## منابع و مراجع

- Multi-Branch: [https://www.jenkins.io/doc/book/pipeline/multibranch/](https://www.jenkins.io/doc/book/pipeline/multibranch/)
- Shared Libraries: [https://www.jenkins.io/doc/book/pipeline/shared-libraries/](https://www.jenkins.io/doc/book/pipeline/shared-libraries/)
- GitHub Branch Source: [https://plugins.jenkins.io/github-branch-source/](https://plugins.jenkins.io/github-branch-source/)
- Extending Pipeline: [https://www.jenkins.io/doc/book/pipeline/shared-libraries/](https://www.jenkins.io/doc/book/pipeline/shared-libraries/)
