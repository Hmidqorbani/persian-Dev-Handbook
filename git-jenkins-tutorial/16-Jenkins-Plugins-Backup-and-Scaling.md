# بخش ۱۶: افزونه‌ها، پشتیبان‌گیری و مقیاس‌پذیری

**مدت یادگیری**: ۲ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۱۵ (امنیت، Credentials و RBAC)

---

## فهرست محتوا

- [فصل ۱: اکوسیستم Plugin](#فصل-۱-اکوسیستم-plugin)
- [فصل ۲: افزونه‌های کلیدی](#فصل-۲-افزونههای-کلیدی)
- [فصل ۳: پشتیبان‌گیری و بازیابی](#فصل-۳-پشتیبانگیری-و-بازیابی)
- [فصل ۴: JCasC (Configuration as Code)](#فصل-۴-jcasc-configuration-as-code)
- [فصل ۵: مقیاس‌پذیری](#فصل-۵-مقیاسپذیری)
- [فصل ۶: به‌روزرسانی و مهاجرت](#فصل-۶-بهروزرسانی-و-مهاجرت)

---

## فصل ۱: اکوسیستم Plugin

### مقدمه

قدرت اصلی Jenkins در اکوسیستم افزونه‌های آن است: بیش از ۱۸۰۰ افزونه.

### مفاهیم اساسی

**مدیریت افزونه‌ها:**
Manage Jenkins → Plugins:
- **Available**: نصب افزونه جدید.
- **Installed**: افزونه‌های نصب‌شده.
- **Updates**: به‌روزرسانی‌ها.
- **Advanced**: آپلود دستی، URL آپدیت.

**انواع افزونه:**
- SCM (Git، GitHub)
- Build Tools (Maven، Gradle)
- Notifications (Slack، Email)
- Reporting (JUnit، Coverage)
- Deployment
- Security

### اشکال شایع

- **اشتباه رایج**: نصب افزونه‌های زیاد و بی‌استفاده. Jenkins کند می‌شود.
- **اشتباه رایج**: نصب افزونه بدون بررسی سازگاری. تعارض ایجاد می‌کند.

### نکات مهم (Tips & Tricks)

- فقط افزونه‌های لازم را نصب کنید.
- قبل از نصب، سازگاری نسخه را چک کنید.
- افزونه‌های قدیمی/بدون نگهداری را حذف کنید.

### خلاصه فصل

اکوسیستم Plugin قلب Jenkins است. فقط افزونه‌های لازم را نصب کنید و سازگاری را بررسی کنید.

---

## فصل ۲: افزونه‌های کلیدی

### مقدمه

برخی افزونه‌ها تقریباً در هر Jenkins لازم‌اند.

### مفاهیم اساسی

| افزونه | کاربرد |
|--------|--------|
| **Git / GitHub** | یکپارچگی SCM |
| **Pipeline** | Pipeline as Code |
| **Blue Ocean** | UI مدرن |
| **Credentials Binding** | مدیریت Secret |
| **JUnit** | گزارش تست |
| **HTML Publisher** | گزارش HTML |
| **Docker Pipeline** | ساخت/اجرای Docker |
| **Kubernetes** | Agent داینامیک |
| **Slack / Email-ext** | اعلان |
| **Role Strategy** | RBAC |
| **Build Timestamp** | زمان‌دار کردن Build |
| **Workspace Cleanup** | `cleanWs()` |
| **AnsiColor** | رنگ در لاگ |
| **Git Parameter** | انتخاب شاخه |

### مثال‌های عملی

نصب از CLI:

```bash
java -jar jenkins-cli.jar -s http://localhost:8080/ \
  -auth user:token install-plugin git
```

### اشکال شایع

- **اشتباه رایج**: نادیده گرفتن افزونه‌های وابسته. برخی افزونه‌ها نیاز به افزونه دیگر دارند.
- **اشتباه رایج**: نصب Blue Ocean در Jenkins قدیمی. سازگاری ندارد.

### نکات مهم (Tips & Tricks)

- Blue Ocean نمای بصری Pipeline عالی می‌دهد.
- Email-ext اعلان‌های پیشرفته می‌سازد.
- Docker Pipeline برای پروژه‌های کانتینری ضروری است.

### خلاصه فصل

افزونه‌های کلیدی شامل Git، Pipeline، Blue Ocean، Credentials و Docker Pipeline هستند. فقط موارد لازم را نصب کنید.

---

## فصل ۳: پشتیبان‌گیری و بازیابی

### مقدمه

بدون پشتیبان، از دست دادن `JENKINS_HOME` یعنی از دست دادن همه‌چیز.

### مفاهیم اساسی

**چه چیزی پشتیبان بگیریم:**
- `config.xml`
- `jobs/`
- `users/`
- `credentials.xml` + `secrets/`
- `plugins/` (اختیاری)
- `nodes/`

**روش‌ها:**
- کپی دستی volume.
- ThinBackup Plugin.
- Snapshot در سطح ماشین/ابر.

### مثال‌های عملی

**ThinBackup:**
1. نصب ThinBackup Plugin
2. Manage Jenkins → ThinBackup → Settings
3. تعیین مسیر پشتیبان
4. Backup Now / زمان‌بندی

**Docker:**
```bash
docker run --rm \
  -v jenkins_home:/data \
  -v /backup:/backup \
  alpine tar czf /backup/jenkins-$(date +%F).tar.gz -C /data .
```

**بازیابی:**
```bash
docker run --rm \
  -v jenkins_home:/data \
  -v /backup:/backup \
  alpine sh -c "rm -rf /data/* && tar xzf /backup/jenkins-2026-10-15.tar.gz -C /data"
```

### اشکال شایع

- **اشتباه رایج**: پشتیبان‌گیری بدون تست بازیابی. پشتیبان بی‌ارزش است اگر بازیابی نشود.
- **اشتباه رایج**: نادیده گرفتن `secrets/`.

### نکات مهم (Tips & Tricks)

- پشتیبان‌گیری خودکار روزانه.
- بازیابی را در محیط تست تمرین کنید.
- نسخه‌های پشتیبان را خارج از سرور نگه دارید.

### خلاصه فصل

پشتیبان‌گیری از JENKINS_HOME حیاتی است. از ThinBackup یا کپی volume استفاده کنید. بازیابی را تمرین کنید.

---

## فصل ۴: JCasC (Configuration as Code)

### مقدمه

**JCasC** به شما اجازه می‌دهد پیکربندی Jenkins را در یک فایل YAML تعریف و نسخه‌بندی کنید.

### مفاهیم اساسی

مزایا:
- پیکربندی در Git (نسخه‌بندی، بازبینی).
- بازتولیدپذیری کامل.
- استقرار خودکار Jenkins جدید.

**نصب:** Configuration as Code Plugin.

### مثال‌های عملی

`jenkins.yaml`:

```yaml
jenkins:
  systemMessage: "Jenkins managed by JCasC"
  numExecutors: 0
  securityRealm:
    local:
      allowsSignup: false
      users:
        - id: admin
          password: "${ADMIN_PASSWORD}"
  authorizationStrategy:
    roleBased:
      roles:
        global:
          - name: admin
            permissions:
              - "Overall/Administer"
            assignments:
              - admin

tool:
  git:
    installations:
      - name: Default
        home: /usr/bin/git

credentials:
  system:
    domainCredentials:
      - credentials:
          - string:
              id: github-token
              secret: "${GITHUB_TOKEN}"
```

اجرا:
```bash
export CASC_JENKINS_CONFIG=/var/jenkins_home/casc/jenkins.yaml
```

### اشکال شایع

- **اشتباه رایج**: Hardcode کردن Secret در YAML. از متغیر محیطی استفاده کنید.
- **اشتباه رایج**: YAML اشتباه. با `jenkins-plugin-cli` یا ابزار lint بررسی کنید.

### نکات مهم (Tips & Tricks)

- JCasC استاندارد مدرن پیکربندی Jenkins است.
- فایل را در Git نگه دارید.
- Secretها را از متغیر محیطی یا Credentials بدهید.

### خلاصه فصل

JCasC پیکربندی Jenkins را به کد تبدیل می‌کند. مزایا: نسخه‌بندی، بازتولیدپذیری و استقرار خودکار. Secretها را از متغیر محیطی بدهید.

---

## فصل ۵: مقیاس‌پذیری

### مقدمه

با رشد تیم، Jenkins باید مقیاس بگیرد.

### مفاهیم اساسی

راهکارهای مقیاس:
1. **Agentهای بیشتر**: توزیع بار.
2. **Cloud Agent**: Docker/K8s برای بار متغیر.
3. **Controller قوی‌تر**: RAM و CPU بیشتر.
4. **پاک‌سازی منظم**: Buildهای قدیمی، Workspaceها.
5. **جدا کردن Jobها**: Folderها.
6. **High Availability**: (پیچیده در Jenkins).

**تنظیم منابع:**
```bash
# JAVA_OPTS
-Xms2g -Xmx4g -XX:MaxMetaspaceSize=512m
```

### مثال‌های عملی

Kubernetes برای مقیاس خودکار:

```yaml
# در Jenkins Configuration
jenkins:
  clouds:
    - kubernetes:
        name: "kubernetes"
        serverUrl: "https://k8s.example.com"
        namespace: "jenkins"
        templates:
          - name: "default"
            label: "k8s"
            containers:
              - name: "jnlp"
                image: "jenkins/inbound-agent:latest"
```

### اشکال شایع

- **اشتباه رایج**: Controller بیش‌بار. Buildها را به Agent منتقل کنید.
- **اشتباه رایج**: نگه‌داشتن همه Buildها. دیسک پر می‌شود.

### نکات مهم (Tips & Tricks)

- Discard Old Builds را برای همه Jobها فعال کنید.
- Cloud Agent برای بار متغیر عالی است.
- منابع Controller را مانیتور کنید.

### خلاصه فصل

مقیاس‌پذیری با Agentهای بیشتر، Cloud، منابع کافی و پاک‌سازی منظم انجام می‌شود. K8s برای بار متغیر ایده‌آل است.

---

## فصل ۶: به‌روزرسانی و مهاجرت

### مقدمه

Jenkins و افزونه‌ها باید منظم به‌روز شوند — هم برای امنیت، هم برای قابلیت‌ها.

### مفاهیم اساسی

**استراتژی به‌روزرسانی:**
1. **پشتیبان بگیرید.**
2. **در محیط تست به‌روز کنید.**
3. **افزونه‌ها را بررسی کنید.**
4. **Jenkins را به‌روز کنید.**
5. **تست کنید.**
6. **در تولید اعمال کنید.**

**مهاجرت:**
- از سرور قدیمی به جدید.
- از Jenkins قدیمی به جدید.
- تغییر image Docker.

### مثال‌های عملی

به‌روزرسانی Docker:

```bash
docker stop jenkins
docker pull jenkins/jenkins:lts-jdk17
docker rm jenkins
docker run -d --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts-jdk17
```

### اشکال شایع

- **اشتباه رایج**: به‌روزرسانی بدون پشتیبان. اگر خراب شود، فاجعه.
- **اشتباه رایج**: به‌روزرسانی همه افزونه‌ها همزمان. تعارض ایجاد می‌کند.

### نکات مهم (Tips & Tricks)

- LTS (Long-Term Support) را ترجیح دهید.
- به‌روزرسانی را در بازه کم‌ترافیک انجام دهید.
- Release notes را بخوانید (breaking changes).

### خلاصه فصل

به‌روزرسانی باید با پشتیبان، تست و تدریجی باشد. LTS را ترجیح دهید و Release notes را بخوانید.

---

## منابع و مراجع

- Jenkins Plugins: [https://plugins.jenkins.io/](https://plugins.jenkins.io/)
- ThinBackup: [https://plugins.jenkins.io/thinBackup/](https://plugins.jenkins.io/thinBackup/)
- JCasC: [https://www.jenkins.io/projects/jcasc/](https://www.jenkins.io/projects/jcasc/)
- Jenkins Scaling: [https://www.jenkins.io/doc/book/scaling/](https://www.jenkins.io/doc/book/scaling/)
