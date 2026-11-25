# بخش ۱۸: پروژه عملی — Python CI/CD

**مدت یادگیری**: ۳ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۱۷ (یکپارچگی Git و Jenkins)

---

## فهرست محتوا

- [فصل ۱: معرفی پروژه](#فصل-۱-معرفی-پروژه)
- [فصل ۲: محیط و وابستگی‌ها](#فصل-۲-محیط-و-وابستگیها)
- [فصل ۳: Jenkinsfile تست](#فصل-۳-jenkinsfile-تست)
- [فصل ۴: Artifact و نسخه‌بندی](#فصل-۴-artifact-و-نسخهبندی)
- [فصل ۵: استقرار و Rollback](#فصل-۵-استقرار-و-rollback)
- [فصل ۶: توسعه پروژه](#فصل-۶-توسعه-پروژه)

---

## فصل ۱: معرفی پروژه

### مقدمه

در این بخش یک پروژه کامل CI/CD برای یک اپلیکیشن پایتونی (FastAPI) می‌سازیم: از تست تا استقرار خودکار.

### مفاهیم اساسی

**سناریو:** یک API ساده با FastAPI که می‌خواهیم:
- با هر push تست شود.
- Lint و Coverage اجرا شود.
- Artifact ساخته شود.
- در staging و production مستقر شود.

**ساختار پروژه:**
```
myapp/
├── app/
│   ├── __init__.py
│   └── main.py
├── tests/
│   ├── test_main.py
│   └── __init__.py
├── requirements.txt
├── requirements-dev.txt
├── Dockerfile
├── .gitignore
├── pyproject.toml
├── Jenkinsfile
└── README.md
```

### مثال‌های عملی

`app/main.py`:

```python
from fastapi import FastAPI

app = FastAPI(title="MyApp")

@app.get("/")
def read_root():
    return {"message": "Hello, World!"}

@app.get("/health")
def health():
    return {"status": "healthy"}
```

`tests/test_main.py`:

```python
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_read_root():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Hello, World!"}

def test_health():
    response = client.get("/health")
    assert response.status_code == 200
    assert response.json()["status"] == "healthy"
```

### اشکال شایع

- **اشتباه رایج**: شروع با پروژه پیچیده. با MVP شروع کنید.
- **اشتباه رایج**: نبود تست. CI بدون تست بی‌معنی است.

### نکات مهم (Tips & Tricks)

- ساختار پروژه را از ابتدا تمیز نگه دارید.
- `.gitignore` پایتون را از ابتدا بگذارید.
- تست‌ها را کنار کد بنویسید.

### خلاصه فصل

پروژه یک API پایتونی (FastAPI) با تست است. ساختار تمیز و تست از ابتدا ضروری است.

---

## فصل ۲: محیط و وابستگی‌ها

### مقدمه

مدیریت وابستگی‌ها و محیط مجازی، پایه CI قابل تکرار است.

### مفاهیم اساسی

`requirements.txt`:
```
fastapi==0.109.0
uvicorn[standard]==0.27.0
```

`requirements-dev.txt`:
```
-r requirements.txt
pytest==8.0.0
pytest-cov==4.1.0
httpx==0.26.0
flake8==7.0.0
black==24.1.0
```

`.gitignore`:
```gitignore
__pycache__/
*.py[cod]
venv/
.env
.pytest_cache/
.coverage
htmlcov/
dist/
*.egg-info/
```

### مثال‌های عملی

اجرای محلی:

```bash
python -m venv venv
source venv/bin/activate      # Linux/Mac
# venv\Scripts\activate       # Windows
pip install -r requirements-dev.txt
pytest --cov=app --cov-report=xml --cov-report=html
flake8 app tests
```

### اشکال شایع

- **اشتباه رایج**: نبود پین نسخه. Buildها ناپایدار می‌شوند.
- **اشتباه رایج**: commit کردن `venv/`. همیشه `.gitignore`.

### نکات مهم (Tips & Tricks)

- نسخه‌ها را پین کنید (`==`).
- `requirements-dev.txt` را جدا نگه دارید.
- Coverage را با `--cov-report=xml` برای CI تولید کنید.

### خلاصه فصل

وابستگی‌ها را پین و در `requirements.txt`/`requirements-dev.txt` نگه دارید. محیط مجازی برای تکرارپذیری ضروری است.

---

## فصل ۳: Jenkinsfile تست

### مقدمه

Jenkinsfile قلب CI این پروژه است.

### مثال‌های عملی

`Jenkinsfile`:

```groovy
pipeline {
    agent {
        docker {
            image 'python:3.12-slim'
            args '-u root'
        }
    }

    options {
        timeout(time: 20, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '20'))
        timestamps()
    }

    environment {
        PIP_CACHE_DIR = "${env.WORKSPACE}/.pip-cache"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'python --version'
            }
        }

        stage('Install') {
            steps {
                sh '''
                    pip install --cache-dir=$PIP_CACHE_DIR -r requirements-dev.txt
                '''
            }
        }

        stage('Lint') {
            steps {
                sh 'flake8 app tests --max-line-length=100'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    pytest --junitxml=results.xml \
                           --cov=app \
                           --cov-report=xml \
                           --cov-report=term
                '''
            }
        }
    }

    post {
        always {
            junit 'results.xml'
            publishHTML(target: [
                reportDir: 'htmlcov',
                reportFiles: 'index.html',
                reportName: 'Coverage Report'
            ])
        }
        failure {
            echo 'Pipeline failed! Notify the team.'
        }
        cleanup {
            cleanWs()
        }
    }
}
```

### اشکال شایع

- **اشتباه رایج**: اجرای همه‌چیز روی Controller. از Docker Agent استفاده کنید.
- **اشتباه رایج**: نبود JUnit report. نتایج تست از دست می‌رود.

### نکات مهم (Tips & Tricks)

- Docker Agent محیط تمیز می‌دهد.
- `--junitxml` برای گزارش Jenkins.
- Coverage را Publish کنید.

### خلاصه فصل

Jenkinsfile با Docker Agent، مراحل Install/Lint/Test و گزارش JUnit/Coverage. این پایه CI است.

---

## فصل ۴: Artifact و نسخه‌بندی

### مقدمه

بعد از تست موفق، Artifact می‌سازیم و نسخه‌بندی می‌کنیم.

### مفاهیم اساسی

- **نسخه از Git tag**: `git describe --tags`.
- **نسخه از Build Number**: `${env.BUILD_NUMBER}`.
- **Artifact**: wheel، Docker image.

### مثال‌های عملی

افزودن مرحله Package:

```groovy
stage('Package') {
    steps {
        sh '''
            pip install build
            python -m build
        '''
    }
}

stage('Archive') {
    steps {
        archiveArtifacts artifacts: 'dist/*', fingerprint: true
    }
}
```

ساخت Docker image:

```groovy
stage('Build Image') {
    when { branch 'main' }
    steps {
        script {
            def tag = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.take(7)}"
            sh "docker build -t myapp:${tag} ."
        }
    }
}
```

### اشکال شایع

- **اشتباه رایج**: نسخه‌بندی مبهم. از ترکیب Build Number و Commit استفاده کنید.
- **اشتباه رایج**: آرشیو نکردن Artifact. نمی‌توانید دانلود کنید.

### نکات مهم (Tips & Tricks)

- `archiveArtifacts` برای نگه‌داری Artifact.
- نسخه را از Git tag یا Commit بسازید.
- image را با tag معنادار بسازید.

### خلاصه فصل

Artifact (wheel/image) را بسازید و با Build Number + Commit نسخه‌بندی کنید. با `archiveArtifacts` نگه دارید.

---

## فصل ۵: استقرار و Rollback

### مقدمه

استقرار خودکار در staging و production با امکان Rollback.

### مفاهیم اساسی

**استقرار Docker:**
- ساخت image.
- Push به Registry.
- Deploy با docker-compose یا kubectl.

**Rollback:**
- نگه‌داشتن نسخه قبلی.
- بازگشت در صورت خطا.

### مثال‌های عملی

```groovy
environment {
    DOCKER_CREDS = credentials('dockerhub')
    IMAGE = "myregistry/myapp"
}

stage('Push Image') {
    when { branch 'main' }
    steps {
        sh '''
            echo "$DOCKER_CREDS_PSW" | docker login -u "$DOCKER_CREDS_USR" --password-stdin
            docker tag myapp:${BUILD_NUMBER} ${IMAGE}:${BUILD_NUMBER}
            docker push ${IMAGE}:${BUILD_NUMBER}
        '''
    }
}

stage('Deploy Staging') {
    when { branch 'develop' }
    steps {
        sh '''
            ssh deployer@staging "
                docker pull ${IMAGE}:${BUILD_NUMBER}
                docker stop myapp || true
                docker rm myapp || true
                docker run -d --name myapp -p 8000:8000 ${IMAGE}:${BUILD_NUMBER}
            "
        '''
    }
}

stage('Deploy Production') {
    when { branch 'main' }
    steps {
        input 'Deploy to production?'
        sh './scripts/deploy-prod.sh ${IMAGE}:${BUILD_NUMBER}'
    }
}
```

`scripts/deploy-prod.sh` (با rollback):

```bash
#!/bin/bash
set -e
IMAGE=$1
CURRENT=$(docker inspect --format='{{.Config.Image}}' myapp 2>/dev/null || echo "")

echo "Deploying $IMAGE"
docker pull $IMAGE
docker stop myapp || true
docker rm myapp || true

if ! docker run -d --name myapp -p 80:8000 $IMAGE; then
    echo "Deploy failed! Rolling back..."
    if [ -n "$CURRENT" ]; then
        docker run -d --name myapp -p 80:8000 $CURRENT
    fi
    exit 1
fi

# Smoke test
sleep 5
if ! curl -f http://localhost/health; then
    echo "Health check failed! Rolling back..."
    docker stop myapp && docker rm myapp
    docker run -d --name myapp -p 80:8000 $CURRENT
    exit 1
fi

echo "Deploy successful"
```

### اشکال شایع

- **اشتباه رایج**: نبود Health Check بعد از Deploy.
- **اشتباه رایج**: نبود Rollback. اگر Deploy خراب شود، سرویس down می‌ماند.

### نکات مهم (Tips & Tricks)

- همیشه Smoke Test بعد از Deploy.
- نسخه قبلی را نگه دارید برای Rollback.
- Deploy تولید با تأیید دستی.

### خلاصه فصل

استقرار با Docker image و Health Check انجام می‌شود. Rollback خودکار در صورت شکست ضروری است. Deploy تولید با تأیید.

---

## فصل ۶: توسعه پروژه

### مقدمه

پروژه را می‌توانید گسترش دهید.

### مفاهیم اساسی

**گام‌های بعدی:**
- افزودن تست‌های Integration.
- اسکن امنیتی (Bandit، Safety).
- تحلیل کیفیت (SonarQube).
- استقرار Kubernetes.
- اعلان Slack.
- Blue-Green Deployment.

### مثال‌های عملی

افزودن اسکن امنیتی:

```groovy
stage('Security Scan') {
    steps {
        sh '''
            pip install bandit safety
            bandit -r app/ -f json -o bandit-report.json || true
            safety check --json > safety-report.json || true
        '''
    }
}
```

افزودن اعلان Slack:

```groovy
post {
    failure {
        slackSend(
            channel: '#ci-alerts',
            color: 'danger',
            message: "❌ ${env.JOB_NAME} #${env.BUILD_NUMBER} failed\n${env.BUILD_URL}"
        )
    }
}
```

### اشکال شایع

- **اشتباه رایج**: افزودن همه‌چیز همزمان. تدریجی پیش بروید.
- **اشتباه رایج**: اسکن امنیتی که کل Pipeline را می‌شکند. `|| true` بگذارید تا فقط گزارش دهد.

### نکات مهم (Tips & Tricks)

- Pipeline را مرحله‌به‌مرحله غنی کنید.
- اعلان‌ها را به Slack/Email وصل کنید.
- SonarQube کیفیت کد را می‌سنجد.

### خلاصه فصل

پروژه را با تست‌های بیشتر، اسکن امنیتی، SonarQube، Kubernetes و اعلان‌ها گسترش دهید. تدریجی پیش بروید.

---

## منابع و مراجع

- FastAPI: [https://fastapi.tiangolo.com/](https://fastapi.tiangolo.com/)
- pytest: [https://docs.pytest.org/](https://docs.pytest.org/)
- Docker in Jenkins: [https://www.jenkins.io/doc/book/pipeline/docker/](https://www.jenkins.io/doc/book/pipeline/docker/)
- Bandit: [https://bandit.readthedocs.io/](https://bandit.readthedocs.io/)
