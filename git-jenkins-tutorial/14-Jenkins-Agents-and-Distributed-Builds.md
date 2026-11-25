# بخش ۱۴: Agents و Build توزیع‌شده

**مدت یادگیری**: ۲.۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۱۳ (Multi-Branch و Shared Libraries)

---

## فهرست محتوا

- [فصل ۱: معماری Controller/Agent](#فصل-۱-معماری-controlleragent)
- [فصل ۲: روش‌های اتصال Agent](#فصل-۲-روشهای-اتصال-agent)
- [فصل ۳: Labels و انتخاب Agent](#فصل-۳-labels-و-انتخاب-agent)
- [فصل ۴: Agent داکری و Kubernetes](#فصل-۴-agent-داکری-و-kubernetes)
- [فصل ۵: توزیع بار](#فصل-۵-توزیع-بار)
- [فصل ۶: نگهداری و عیب‌یابی](#فصل-۶-نگهداری-و-عیبیابی)

---

## فصل ۱: معماری Controller/Agent

### مقدمه

در محیط تولید، همه Buildها نباید روی Controller اجرا شوند. معماری **Controller/Agent** کار را توزیع می‌کند.

### اهمیت این موضوع

Controller باید سبک و امن بماند. اگر Buildهای سنگین روی آن اجرا شوند، رابط کاربری کند می‌شود و در صورت آلودگی Build، کل Jenkins به خطر می‌افتد.

### مفاهیم اساسی

- **Controller**: مدیریت، زمان‌بندی، رابط کاربری، پیکربندی. کار سنگین نمی‌کند.
- **Agent (Node)**: اجرای واقعی Buildها.
- **Executor**: اسلات اجرا روی هر Agent (مثلاً ۲ Executor = ۲ Build همزمان).
- **Workspace**: پوشه کاری هر Job روی Agent.

```
        ┌──────────────┐
        │  Controller  │  (سبک، امن)
        └──────┬───────┘
    ┌──────────┼──────────┐
    ↓          ↓          ↓
Agent-Linux  Agent-Win  Agent-Docker
 (Java)      (Maven)    (Containers)
```

### اشکال شایع

- **اشتباه رایج**: اجرای Build روی Controller. همیشه `agent none` یا Agent جدا.
- **اشتباه رایج**: تعداد Executor زیاد روی Controller.

### نکات مهم (Tips & Tricks)

- Controller را با `numExecutors: 0` پیکربندی کنید.
- Agentها را با برچسب (Label) سازماندهی کنید.
- Workspace را روی هر Agent مدیریت کنید.

### خلاصه فصل

معماری Controller/Agent کار را توزیع می‌کند. Controller سبک و امن می‌ماند و Agentها Build را اجرا می‌کنند. تعداد Executor را درست تنظیم کنید.

---

## فصل ۲: روش‌های اتصال Agent

### مقدمه

Agentها به چند روش به Controller وصل می‌شوند.

### مفاهیم اساسی

**۱. SSH (توصیه‌شده):**
- Controller با SSH به Agent وصل می‌شود.
- نیاز به کلید SSH.
- امن و پایدار.

**۲. JNLP / Inbound:**
- Agent به Controller وصل می‌شود.
- مناسب محیط‌هایی که Controller نمی‌تواند به Agent دسترسی داشته باشد (مثلاً پشت NAT).

**۳. Docker Agent:**
- Agent موقت با Docker.
- سبک و ایزوله.

**۴. Kubernetes Agent:**
- Pod موقت در Kubernetes.
- مقیاس‌پذیر و مدرن.

### مثال‌های عملی

**افزودن SSH Agent:**
1. Manage Jenkins → Nodes → New Node
2. نام، Remote root directory
3. Launch method: Launch agents via SSH
4. Host، Credentials (SSH key)
5. Save

در Pipeline:
```groovy
pipeline {
    agent { label 'linux' }
    stages { ... }
}
```

### مقایسه و الگوهای مختلف

| روش | جهت اتصال | مناسب |
|-----|-----------|-------|
| SSH | Controller→Agent | شبکه باز |
| JNLP | Agent→Controller | پشت NAT |
| Docker | موقت | ایزوله |
| K8s | موقت | مقیاس بالا |

### اشکال شایع

- **اشتباه رایج**: کلید SSH با رمز. Jenkins نمی‌تواند رمز بدهد؛ کلید بدون رمز بسازید.
- **اشتباه رایج**: پورت ۵۰۰۰۰ مسدود. برای JNLP بازش کنید.

### نکات مهم (Tips & Tricks)

- SSH برای Agentهای دائمی، Docker/K8s برای موقت.
- کلید SSH را در Credentials ذخیره کنید.
- Remote root directory را پایدار انتخاب کنید.

### خلاصه فصل

Agentها با SSH، JNLP، Docker یا Kubernetes وصل می‌شوند. SSH برای دائمی، Docker/K8s برای موقت. کلید SSH بدون رمز بسازید.

---

## فصل ۳: Labels و انتخاب Agent

### مقدمه

**Label** برچسبی است که مشخص می‌کند Job روی کدام Agent اجرا شود.

### مفاهیم اساسی

```groovy
agent { label 'linux' }
agent { label 'linux && docker' }
agent { label 'windows || mac' }
agent { label 'maven && jdk17' }
```

عملگرها: `&&` (و)، `||` (یا)، `!` (نه).

### مثال‌های عملی

```groovy
pipeline {
    agent none
    stages {
        stage('Build') {
            agent { label 'linux && docker' }
            steps { sh 'make build' }
        }
        stage('Test on Windows') {
            agent { label 'windows' }
            steps { bat 'run_tests.bat' }
        }
        stage('Deploy') {
            agent { label 'deployer' }
            steps { sh './deploy.sh' }
        }
    }
}
```

`agent none` در سطح pipeline یعنی هر stage خودش Agent انتخاب کند.

### اشکال شایع

- **اشتباه رایج**: برچسب‌های مبهم. نام‌گذاری استاندارد کنید.
- **اشتباه رایج**: نبود Agent با برچسب موردنظر. Job در صف می‌ماند.

### نکات مهم (Tips & Tricks)

- برچسب‌ها را بر اساس قابلیت بگذارید: `docker`، `jdk17`، `maven`، `linux`.
- از `agent none` برای Pipelineهای چندمحیطی استفاده کنید.
- در کنسول، درخواست Agent و انتظار را ببینید.

### خلاصه فصل

Labels تعیین می‌کنند Job روی کدام Agent اجرا شود. با `&&`/`||` ترکیب کنید. `agent none` کنترل را به stageها می‌دهد.

---

## فصل ۴: Agent داکری و Kubernetes

### مقدمه

Agentهای موقت داکری و Kubernetes انعطاف و مقیاس‌پذیری بی‌نظیری می‌دهند.

### مفاهیم اساسی

**Docker Agent:**

```groovy
pipeline {
    agent {
        docker {
            image 'python:3.12'
            args '-u root'
        }
    }
    stages {
        stage('Test') {
            steps { sh 'python --version && pytest' }
        }
    }
}
```

**Dockerfile Agent (ساخت image سفارشی):**

```groovy
agent {
    dockerfile {
        filename 'Dockerfile.ci'
        dir 'ci'
        additionalBuildArgs '--build-arg VERSION=1.0'
    }
}
```

**Kubernetes Agent:**

```groovy
pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.9-eclipse-temurin-17
    command: ['cat']
    tty: true
  - name: docker
    image: docker:24
    command: ['cat']
    tty: true
'''
        }
    }
    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }
    }
}
```

### مقایسه و الگوهای مختلف

| نوع | عمر | مناسب |
|-----|-----|-------|
| Static Agent | دائمی | Buildهای تکراری |
| Docker Agent | موقت | ایزوله |
| K8s Pod | موقت | مقیاس بالا |

### اشکال شایع

- **اشتباه رایج**: image بدون ابزار لازم. image درست انتخاب کنید.
- **اشتباه رایج**: K8s بدون serviceAccount و RBAC. خطای دسترسی می‌دهد.

### نکات مهم (Tips & Tricks)

- Docker Agent محیط تمیز و تکرارپذیر می‌دهد.
- K8s برای بار متغیر عالی است.
- image را پین (pin) کنید (`python:3.12.1` نه `python:latest`).

### خلاصه فصل

Docker و Kubernetes Agentهای موقت و ایزوله می‌سازند. Docker برای تکرارپذیری و K8s برای مقیاس‌پذیری مناسب است. image را پین کنید.

---

## فصل ۵: توزیع بار

### مقدمه

با چند Agent، باید بار را متعادل توزیع کنید.

### مفاهیم اساسی

- **تعداد Executor**: هر Agent چند Build همزمان.
- **برچسب‌گذاری هوشمند**: توزیع بار بین Agentها.
- **Parallel**: استفاده از چند Agent همزمان.
- **Cloud**: Agentهای خودکار (Docker/K8s) که با تقاضا ساخته می‌شوند.

### مثال‌های عملی

```groovy
stage('Parallel Tests') {
    parallel {
        stage('Shard 1') {
            agent { label 'linux' }
            steps { sh 'pytest tests/ -k "test_1 or test_2"' }
        }
        stage('Shard 2') {
            agent { label 'linux' }
            steps { sh 'pytest tests/ -k "test_3 or test_4"' }
        }
    }
}
```

### اشکال شایع

- **اشتباه رایج**: Executor زیاد روی یک Agent ضعیف. Buildها کند می‌شوند.
- **اشتباه رایج**: عدم استفاده از Cloud برای بار متغیر.

### نکات مهم (Tips & Tricks)

- Executor را بر اساس منابع (CPU/RAM) تنظیم کنید.
- برای بار متغیر، Cloud (Docker/K8s) بهتر از Agent ثابت است.
- تست‌ها را Shard کنید تا موازی شوند.

### خلاصه فصل

توزیع بار با تعداد Executor، برچسب‌گذاری، parallel و Cloud انجام می‌شود. Cloud برای بار متغیر ایده‌آل است.

---

## فصل ۶: نگهداری و عیب‌یابی

### مقدمه

Agentها نیاز به نگهداری دارند.

### مفاهیم اساسی

**مشکلات رایج:**
- Agent offline.
- فضای دیسک پر (Workspaceهای قدیمی).
- نسخه ابزار ناهماهنگ.
- اتصال قطع.

**دستورات مفید:**

```bash
# بررسی وضعیت
java -jar agent.jar -jnlpUrl ... -secret ...

# فضای دیسک
df -h

# لاگ Agent
tail -f /var/log/jenkins/agent.log
```

### مثال‌های عملی

پاک‌سازی Workspaceهای قدیمی:

```groovy
post {
    cleanup {
        cleanWs()
    }
}
```

### اشکال شایع

- **اشتباه رایج**: نگه‌داشتن Workspaceهای قدیمی. دیسک پر می‌شود.
- **اشتباه رایج**: Agent بدون مانیتورینگ.

### نکات مهم (Tips & Tricks)

- `cleanWs()` بعد از هر Build.
- فضای دیسک Agentها را مانیتور کنید.
- Agentهای problem دار را mark offline کنید.
- نسخه ابزارها را استاندارد کنید.

### خلاصه فصل

نگهداری Agent شامل پاک‌سازی Workspace، مانیتورینگ دیسک و یکسان‌سازی ابزارهاست. `cleanWs()` عادت خوبی است.

---

## منابع و مراجع

- Distributed Builds: [https://www.jenkins.io/doc/book/managing/nodes/](https://www.jenkins.io/doc/book/managing/nodes/)
- Docker Agent: [https://www.jenkins.io/doc/book/pipeline/docker/](https://www.jenkins.io/doc/book/pipeline/docker/)
- Kubernetes Plugin: [https://plugins.jenkins.io/kubernetes/](https://plugins.jenkins.io/kubernetes/)
- Jenkins Agents: [https://www.jenkins.io/doc/book/using/using-agents/](https://www.jenkins.io/doc/book/using/using-agents/)
