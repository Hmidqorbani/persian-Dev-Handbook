# بخش ۱۹: پروژه عملی — Java (Maven) + Docker CI/CD

**مدت یادگیری**: ۳.۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۱۸ (پروژه Python CI/CD)

---

## فهرست محتوا

- [فصل ۱: پروژه Maven](#فصل-۱-پروژه-maven)
- [فصل ۲: Build و Test با Maven](#فصل-۲-build-و-test-با-maven)
- [فصل ۳: نسخه‌بندی و Nexus](#فصل-۳-نسخهبندی-و-nexus)
- [فصل ۴: داکری‌سازی](#فصل-۴-داکریسازی)
- [فصل ۵: ساخت و انتشار Image](#فصل-۵-ساخت-و-انتشار-image)
- [فصل ۶: استقرار و Rollback](#فصل-۶-استقرار-و-rollback)
- [فصل ۷: پروژه کامل](#فصل-۷-پروژه-کامل)

---

## فصل ۱: پروژه Maven

### مقدمه

در این بخش یک اپلیکیشن Java Spring Boot را با Maven می‌سازیم و با Docker مستقر می‌کنیم.

### مفاهیم اساسی

**Maven** ابزار build و مدیریت وابستگی Java است. `pom.xml` قلب آن است.

**ساختار پروژه:**
```
spring-app/
├── src/
│   ├── main/
│   │   ├── java/com/example/app/
│   │   │   ├── Application.java
│   │   │   └── HelloController.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
│       └── java/com/example/app/
│           └── HelloControllerTest.java
├── Dockerfile
├── pom.xml
├── .gitignore
├── Jenkinsfile
└── README.md
```

### مثال‌های عملی

`pom.xml` (خلاصه):

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>spring-app</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

`HelloController.java`:

```java
package com.example.app;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/")
    public String hello() {
        return "Hello, World!";
    }

    @GetMapping("/health")
    public String health() {
        return "OK";
    }
}
```

### اشکال شایع

- **اشتباه رایج**: نبود `spring-boot-starter-parent`. مدیریت وابستگی سخت می‌شود.
- **اشتباه رایج**: نسخه Java ناهماهنگ با Jenkins.

### نکات مهم (Tips & Tricks)

- `pom.xml` را تمیز و خوانا نگه دارید.
- از Spring Boot Parent برای مدیریت نسخه استفاده کنید.
- `.gitignore` برای `target/`.

### خلاصه فصل

پروژه یک Spring Boot با Maven است. `pom.xml` وابستگی‌ها و build را مدیریت می‌کند. ساختار استاندارد Maven را رعایت کنید.

---

## فصل ۲: Build و Test با Maven

### مقدمه

Maven چرخه حیات مشخصی دارد: `validate → compile → test → package → verify → install → deploy`.

### مفاهیم اساسی

**فازهای مهم:**
- `mvn clean` — پاک کردن `target/`.
- `mvn compile` — کامپایل.
- `mvn test` — اجرای تست.
- `mvn package` — ساخت jar.
- `mvn install` — نصب در مخزن محلی.
- `mvn deploy` — انتشار در مخزن راه دور.

### مثال‌های عملی

اجرای محلی:

```bash
mvn clean package
mvn test
mvn clean verify
```

Jenkinsfile با Maven:

```groovy
pipeline {
    agent any
    tools {
        maven 'maven-3.9'
        jdk 'jdk17'
    }
    stages {
        stage('Build & Test') {
            steps {
                sh 'mvn clean verify'
            }
        }
    }
    post {
        always {
            junit '**/target/surefire-reports/*.xml'
        }
    }
}
```

### اشکال شایع

- **اشتباه رایج**: نبود `clean`. Buildهای قدیمی باقی می‌مانند.
- **اشتباه رایج**: نادیده گرفتن گزارش تست.

### نکات مهم (Tips & Tricks)

- `mvn verify` تست و بررسی‌های اضافه را اجرا می‌کند.
- گزارش Surefire را Publish کنید.
- از `-DskipTests` فقط در موارد خاص استفاده کنید.

### خلاصه فصل

Maven چرخه validate→compile→test→package→verify→install→deploy دارد. `mvn clean verify` استاندارد CI است.

---

## فصل ۳: نسخه‌بندی و Nexus

### مقدمه

انتشار Artifact در یک مخزن (Nexus/Artifactory) آن را قابل استفاده مجدد می‌کند.

### مفاهیم اساسی

**Nexus** یک مخزن Artifact است:
- **Snapshot**: نسخه‌های در حال توسعه.
- **Release**: نسخه‌های پایدار.

**نسخه‌بندی Maven:**
- `1.0.0-SNAPSHOT` — در حال توسعه.
- `1.0.0` — پایدار.

### مثال‌های عملی

پیکربندی `settings.xml`:

```xml
<settings>
    <servers>
        <server>
            <id>nexus</id>
            <username>${env.NEXUS_USER}</username>
            <password>${env.NEXUS_PASS}</password>
        </server>
    </servers>
</settings>
```

`distributionManagement` در `pom.xml`:

```xml
<distributionManagement>
    <repository>
        <id>nexus</id>
        <url>https://nexus.example.com/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>nexus</id>
        <url>https://nexus.example.com/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

Jenkins:

```groovy
stage('Deploy to Nexus') {
    when { branch 'main' }
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'nexus-creds',
            usernameVariable: 'NEXUS_USER',
            passwordVariable: 'NEXUS_PASS'
        )]) {
            sh 'mvn deploy -DskipTests'
        }
    }
}
```

### اشکال شایع

- **اشتباه رایج**: انتشار Snapshot در مخزن Release.
- **اشتباه رایج**: Hardcode کردن رمز Nexus.

### نکات مهم (Tips & Tricks)

- Snapshot برای توسعه، Release برای پایدار.
- از Credentials برای رمز Nexus.
- نسخه‌بندی معنادار (Semantic Versioning).

### خلاصه فصل

Nexus Artifactها را نگه می‌دارد. `mvn deploy` آن‌ها را منتشر می‌کند. Snapshot و Release را جدا نگه دارید.

---

## فصل ۴: داکری‌سازی

### مقدمه

Docker اپلیکیشن را قابل حمل می‌کند.

### مفاهیم اساسی

**Dockerfile چندمرحله‌ای (Multi-stage):**

```dockerfile
# مرحله Build
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# مرحله Runtime
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -qO- http://localhost:8080/health || exit 1
ENTRYPOINT ["java", "-jar", "app.jar"]
```

`.dockerignore`:
```
target/
.git/
.idea/
*.md
```

### مثال‌های عملی

ساخت و اجرا:

```bash
docker build -t spring-app:1.0.0 .
docker run -d -p 8080:8080 --name spring-app spring-app:1.0.0
curl http://localhost:8080/health
```

### اشکال شایع

- **اشتباه رایج**: image تک‌مرحله‌ای و بزرگ. Multi-stage حجم را کم می‌کند.
- **اشتباه رایج**: نبود `.dockerignore`. context بزرگ می‌شود.
- **اشتباه رایج**: اجرا با کاربر root.

### نکات مهم (Tips & Tricks)

- Multi-stage build را همیشه استفاده کنید.
- از base image سبک (alpine) استفاده کنید.
- HEALTHCHECK برای orchestration.
- کاربر non-root بسازید.

### خلاصه فصل

Multi-stage Dockerfile image کوچک و امن می‌سازد. `.dockerignore` و HEALTHCHECK ضروری‌اند. با کاربر non-root اجرا کنید.

---

## فصل ۵: ساخت و انتشار Image

### مقدمه

Jenkins image را می‌سازد و در Registry منتشر می‌کند.

### مثال‌های عملی

```groovy
pipeline {
    agent any
    tools { jdk 'jdk17' }

    environment {
        REGISTRY = 'registry.example.com'
        IMAGE_NAME = 'spring-app'
        DOCKER_CREDS = credentials('registry-creds')
    }

    stages {
        stage('Build JAR') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    def version = sh(
                        script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout',
                        returnStdout: true
                    ).trim()
                    env.VERSION = version
                    sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${version} ."
                    sh "docker tag ${REGISTRY}/${IMAGE_NAME}:${version} ${REGISTRY}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Push Image') {
            when { branch 'main' }
            steps {
                sh '''
                    echo "$DOCKER_CREDS_PSW" | docker login ${REGISTRY} -u "$DOCKER_CREDS_USR" --password-stdin
                    docker push ${REGISTRY}/${IMAGE_NAME}:${VERSION}
                    docker push ${REGISTRY}/${IMAGE_NAME}:latest
                '''
            }
        }
    }
}
```

### اشکال شایع

- **اشتباه رایج**: push با tag `latest` فقط. همیشه نسخه مشخص هم بزنید.
- **اشتباه رایج**: رمز Registry در کد.

### نکات مهم (Tips & Tricks)

- tag نسخه‌دار + `latest`.
- از Credentials استفاده کنید.
- image را اسکن کنید (Trivy).

### خلاصه فصل

Jenkins JAR را می‌سازد، image می‌سازد و به Registry push می‌کند. tag نسخه‌دار ضروری است. image را اسکن کنید.

---

## فصل ۶: استقرار و Rollback

### مقدمه

استقرار با Docker Compose یا Kubernetes و Rollback.

### مثال‌های عملی

`docker-compose.yml`:

```yaml
version: '3.8'
services:
  app:
    image: ${REGISTRY}/spring-app:${VERSION}
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:8080/health"]
      interval: 30s
      timeout: 3s
      retries: 3
```

`deploy.sh` با Rollback:

```bash
#!/bin/bash
set -e
VERSION=$1
REGISTRY=$2
PREVIOUS=$(cat /opt/app/.current-version 2>/dev/null || echo "latest")

echo "Deploying version $VERSION (previous: $PREVIOUS)"

export REGISTRY=$REGISTRY VERSION=$VERSION
docker-compose pull
docker-compose up -d --no-deps app

sleep 10
if ! curl -f http://localhost:8080/health; then
    echo "Health check failed! Rolling back to $PREVIOUS..."
    export VERSION=$PREVIOUS
    docker-compose up -d --no-deps app
    exit 1
fi

echo $VERSION > /opt/app/.current-version
echo "Deploy successful"
```

### اشکال شایع

- **اشتباه رایج**: نبود Health Check.
- **اشتباه رایج**: نبود Rollback. سرویس down می‌ماند.

### نکات مهم (Tips & Tricks)

- Health Check بعد از Deploy.
- نسخه فعلی را ذخیره کنید.
- Blue-Green برای downtime صفر.

### خلاصه فصل

استقرار با Docker Compose و Health Check انجام می‌شود. Rollback خودکار در صورت شکست. نسخه فعلی را ذخیره کنید.

---

## فصل ۷: پروژه کامل

### مقدمه

حالا همه‌چیز را در یک Jenkinsfile کامل ترکیب می‌کنیم.

### مثال‌های عملی

`Jenkinsfile` نهایی:

```groovy
pipeline {
    agent any
    tools {
        maven 'maven-3.9'
        jdk 'jdk17'
    }

    environment {
        REGISTRY = 'registry.example.com'
        IMAGE_NAME = 'spring-app'
        DOCKER_CREDS = credentials('registry-creds')
        NEXUS_CREDS = credentials('nexus-creds')
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '20'))
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean verify'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Code Quality') {
            steps {
                sh 'mvn sonar:sonar -Dsonar.projectKey=spring-app || true'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    env.VERSION = sh(
                        script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout',
                        returnStdout: true
                    ).trim()
                    sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${VERSION} ."
                }
            }
        }

        stage('Scan Image') {
            steps {
                sh "trivy image --exit-code 0 --severity HIGH,CRITICAL ${REGISTRY}/${IMAGE_NAME}:${VERSION} || true"
            }
        }

        stage('Push Image') {
            when { branch 'main' }
            steps {
                sh '''
                    echo "$DOCKER_CREDS_PSW" | docker login ${REGISTRY} -u "$DOCKER_CREDS_USR" --password-stdin
                    docker push ${REGISTRY}/${IMAGE_NAME}:${VERSION}
                '''
            }
        }

        stage('Deploy Staging') {
            when { branch 'develop' }
            steps {
                sh "./scripts/deploy.sh ${VERSION} ${REGISTRY} staging"
            }
        }

        stage('Deploy Production') {
            when { branch 'main' }
            steps {
                input 'Deploy to production?'
                sh "./scripts/deploy.sh ${VERSION} ${REGISTRY} production"
            }
        }
    }

    post {
        success {
            slackSend channel: '#ci', message: "✅ ${env.JOB_NAME} #${env.BUILD_NUMBER} succeeded"
        }
        failure {
            slackSend channel: '#ci', color: 'danger',
                message: "❌ ${env.JOB_NAME} #${env.BUILD_NUMBER} failed\n${env.BUILD_URL}"
        }
        cleanup { cleanWs() }
    }
}
```

### نکات مهم (Tips & Tricks)

- Pipeline را مثل کد بازبینی کنید.
- مراحل را مستقل نگه دارید.
- اعلان‌ها را فعال کنید.

### خلاصه فصل

پروژه کامل شامل Checkout، Build/Test، Quality، Build Image، Scan، Push، Deploy Staging/Production و اعلان‌هاست. این یک Pipeline حرفه‌ای تولیدی است.

---

## منابع و مراجع

- Maven: [https://maven.apache.org/guides/](https://maven.apache.org/guides/)
- Spring Boot: [https://spring.io/projects/spring-boot](https://spring.io/projects/spring-boot)
- Docker Multi-stage: [https://docs.docker.com/build/building/multi-stage/](https://docs.docker.com/build/building/multi-stage/)
- Nexus Repository: [https://help.sonatype.com/repomanager3](https://help.sonatype.com/repomanager3)
- Trivy: [https://trivy.dev/](https://trivy.dev/)
