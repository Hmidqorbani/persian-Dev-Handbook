# بخش ۱۵: امنیت، Credentials و RBAC

**مدت یادگیری**: ۲.۵ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: بخش ۱۴ (Agents و Build توزیع‌شده)

---

## فهرست محتوا

- [فصل ۱: مدل امنیتی Jenkins](#فصل-۱-مدل-امنیتی-jenkins)
- [فصل ۲: انواع Credentials](#فصل-۲-انواع-credentials)
- [فصل ۳: Credentials Binding](#فصل-۳-credentials-binding)
- [فصل ۴: احراز هویت](#فصل-۴-احراز-هویت)
- [فصل ۵: RBAC](#فصل-۵-rbac)
- [فصل ۶: سخت‌سازی](#فصل-۶-سختسازی)
- [فصل ۷: بهترین روش‌ها](#فصل-۷-بهترین-روشها)

---

## فصل ۱: مدل امنیتی Jenkins

### مقدمه

Jenkins دسترسی به کد، سرورها و Secretها دارد؛ پس هدف جذابی برای حمله است. امنیت آن حیاتی است.

### اهمیت این موضوع

یک Jenkins ناامن می‌تواند به کل زیرساخت سازمان دسترسی بدهد. Pipelineها می‌توانند کد دلخواه اجرا کنند؛ پس کنترل دسترسی و Secretها ضروری است.

### مفاهیم اساسی

لایه‌های امنیتی:
1. **احراز هویت (Authentication)**: شما کی هستید؟
2. **مجوزدهی (Authorization)**: چه کاری می‌توانید بکنید؟
3. **مدیریت Secret (Credentials)**: رمزها کجا نگه داشته می‌شوند؟
4. **امنیت شبکه**: چه کسی می‌تواند به Jenkins وصل شود؟
5. **امنیت Agent**: Agentها چقدر قابل اعتمادند؟
6. **Sandbox اسکریپت**: چه کدی می‌تواند اجرا شود؟

### اشکال شایع

- **اشتباه رایج**: Jenkins بدون احراز هویت روی اینترنت. فاجعه!
- **اشتباه رایج**: دادن دسترسی admin به همه.

### نکات مهم (Tips & Tricks)

- Jenkins را هرگز مستقیماً روی اینترنت باز نکنید؛ از VPN/reverse proxy استفاده کنید.
- اصل حداقل دسترسی (Least Privilege) را رعایت کنید.

### خلاصه فصل

امنیت Jenkins شش لایه دارد: احراز هویت، مجوزدهی، Credentials، شبکه، Agent و Sandbox. Jenkins را هرگز بدون احراز هویت روی اینترنت باز نکنید.

---

## فصل ۲: انواع Credentials

### مقدمه

Jenkins انواع مختلفی از Secretها را مدیریت می‌کند.

### مفاهیم اساسی

| نوع | کاربرد |
|-----|--------|
| **Username with password** | Git، Registry |
| **SSH Username with private key** | SSH، Git |
| **Secret text** | API token |
| **Secret file** | kubeconfig، .env |
| **Certificate** | PKI، mTLS |

**دامنه (Scope):**
- **Global**: در همه Jobها.
- **System**: فقط برای Jenkins خودش.
- **(Folder)**: محدود به Folder.

### مثال‌های عملی

افزودن Credential:
1. Manage Jenkins → Credentials → System → Global credentials
2. Add Credentials
3. نوع: Secret text
4. ID: `github-token`
5. Secret: توکن
6. Save

استفاده:
```groovy
withCredentials([string(credentialsId: 'github-token', variable: 'TOKEN')]) {
    sh 'curl -H "Authorization: Bearer $TOKEN" ...'
}
```

### اشکال شایع

- **اشتباه رایج**: ID نامشخص. نام‌گذاری استاندارد کنید.
- **اشتباه رایج**: Scope اشتباه. Credential حساس را Folder-scoped کنید.

### نکات مهم (Tips & Tricks)

- ID معنادار بگذارید: `dockerhub-prod`، `k8s-staging`.
- برای هر محیط Credential جدا بسازید.
- Credentials را در Folders سازماندهی کنید.

### خلاصه فصل

Jenkins پنج نوع Credential دارد: Username/Password، SSH Key، Secret Text، Secret File و Certificate. با Scope درست و ID معنادار مدیریت کنید.

---

## فصل ۳: Credentials Binding

### مقدمه

**Credentials Binding** روش امن استفاده از Secretها در Pipeline است.

### مفاهیم اساسی

```groovy
withCredentials([
    string(credentialsId: 'api-token', variable: 'API_TOKEN'),
    usernamePassword(credentialsId: 'db-creds', usernameVariable: 'DB_USER', passwordVariable: 'DB_PASS'),
    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG'),
    sshUserPrivateKey(credentialsId: 'deploy-key', keyFileVariable: 'SSH_KEY')
]) {
    sh '''
        echo "Connecting as $DB_USER"
        kubectl apply -f deployment.yaml
    '''
}
```

**environment**:
```groovy
environment {
    DB_CREDS = credentials('db-creds')
    // DB_CREDS, DB_CREDS_USR, DB_CREDS_PSW
}
```

### مثال‌های عملی

```groovy
pipeline {
    agent any
    environment {
        DOCKER = credentials('dockerhub')
    }
    stages {
        stage('Push') {
            steps {
                sh 'echo $DOCKER_PSW | docker login -u $DOCKER_USR --password-stdin'
                sh 'docker push myapp:latest'
            }
        }
    }
}
```

### اشکال شایع

- **اشتباه رایج**: echo کردن Secret. Jenkins آن را `****` می‌کند اما در `set -x` ممکن است لو برود.
- **اشتباه رایج**: استفاده از Secret در آرگومان دستور (در `ps` دیده می‌شود).

### نکات مهم (Tips & Tricks)

- Secret را از stdin بدهید، نه آرگومان.
- `withCredentials` دامنه را محدود می‌کند؛ بهتر از `environment` است.
- Jenkins Secretها را در لاگ mask می‌کند.

### خلاصه فصل

withCredentials و environment روش‌های امن استفاده از Secretها هستند. Secret را از stdin بدهید و از echo پرهیز کنید.

---

## فصل ۴: احراز هویت

### مقدمه

Jenkins باید بداند شما کی هستید.

### مفاهیم اساسی

گزینه‌های احراز هویت:
- **Jenkins' own user database**: ساده، برای شروع.
- **LDAP / Active Directory**: سازمانی.
- **GitHub / GitLab OAuth**: یکپارچه با SCM.
- **SAML / OIDC**: مدرن و سازمانی.

**پیکربندی:**
Manage Jenkins → Security → Authentication → انتخاب روش.

### مثال‌های عملی

برای LDAP:
```
Server: ldap://ldap.example.com
Root DN: dc=example,dc=com
User search base: ou=users
User search filter: (uid={0})
```

### اشکال شایع

- **اشتباه رایج**: فراموش کردن رمز admin و قفل شدن. از `JENKINS_HOME` قابل ریست است (با توقف Jenkins).
- **اشتباه رایج**: استفاده از دیتابیس داخلی در سازمان بزرگ.

### نکات مهم (Tips & Tricks)

- برای سازمان، از LDAP/OAuth استفاده کنید.
- همیشه یک admin پشتیبان داشته باشید.
- MFA را اگر ممکن است فعال کنید.

### خلاصه فصل

احراز هویت با دیتابیس داخلی، LDAP/AD یا OAuth انجام می‌شود. برای سازمان LDAP/OAuth توصیه می‌شود. همیشه admin پشتیبان داشته باشید.

---

## فصل ۵: RBAC

### مقدمه

**RBAC (Role-Based Access Control)** تعیین می‌کند هر کاربر چه کاری می‌تواند بکند.

### مفاهیم اساسی

با **Role Strategy Plugin**:
- **Global roles**: admin، read-only، developer.
- **Project roles**: دسترسی به Jobهای خاص.
- **Node roles**: دسترسی به Agentها.

**مثال نقش‌ها:**

| نقش | دسترسی |
|-----|--------|
| admin | همه‌چیز |
| developer | ساخت/اجرای Job، بدون مدیریت سیستم |
| viewer | فقط مشاهده |
| deployer | استقرار در تولید |

### مثال‌های عملی

پیکربندی:
1. نصب Role Strategy Plugin
2. Manage Jenkins → Security → Authorization → Role-Based Strategy
3. Manage and Assign Roles
4. تعریف Global roles (admin، developer، viewer)
5. تعریف Project roles با Pattern (مثلاً `prod-.*`)
6. Assign Roles به کاربران/گروه‌ها

### اشکال شایع

- **اشتباه رایج**: دادن admin به همه. اصل حداقل دسترسی را رعایت کنید.
- **اشتباه رایج**: Pattern اشتباه در Project roles.

### نکات مهم (Tips & Tricks)

- نقش‌ها را بر اساس وظیفه تعریف کنید.
- از گروه‌های LDAP استفاده کنید، نه کاربران فردی.
- به‌طور دوره‌ای دسترسی‌ها را بازبینی کنید.

### خلاصه فصل

RBAC با Role Strategy Plugin پیاده می‌شود. Global، Project و Node roles تعریف کنید. اصل حداقل دسترسی را رعایت کنید.

---

## فصل ۶: سخت‌سازی

### مقدمه

**Hardening** یعنی Jenkins را در برابر حملات مقاوم کنیم.

### مفاهیم اساسی

اقدامات سخت‌سازی:
1. **CSRF Protection**: فعال (پیش‌فرض).
2. **Agent → Controller Access Control**: محدود.
3. **Script Security**: Sandbox.
4. **CLI**: غیرفعال یا محدود.
5. **HTTP**: فقط HTTPS.
6. **پورت‌ها**: محدود به شبکه داخلی.
7. **به‌روزرسانی**: منظم.

**پیکربندی:**
Manage Jenkins → Security:
- CSRF Protection ✅
- Agent → Controller Access Control: محدود به Agentهای مشخص
- Markup Formatter: Plain text (جلوگیری از XSS)

### مثال‌های عملی

محدود کردن CLI:

```bash
# در jenkins.xml یا JAVA_OPTS
-Dhudson.CLI.overTcpPort=disable
```

HTTPS با reverse proxy (Nginx):

```nginx
server {
    listen 443 ssl;
    server_name jenkins.example.com;

    ssl_certificate /etc/ssl/cert.pem;
    ssl_certificate_key /etc/ssl/key.pem;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### اشکال شایع

- **اشتباه رایج**: غیرفعال کردن CSRF برای راحتی. خطر امنیتی جدی.
- **اشتباه رایج**: باز گذاشتن Jenkins روی اینترنت.

### نکات مهم (Tips & Tricks)

- از reverse proxy با HTTPS استفاده کنید.
- Script Security را جدی بگیرید؛ Pipelineهای نامعتبر را تأیید نکنید.
- Jenkins و افزونه‌ها را منظم به‌روز کنید.

### خلاصه فصل

سخت‌سازی شامل CSRF، کنترل Agent، Script Security، HTTPS و به‌روزرسانی است. Jenkins را پشت reverse proxy و خارج از اینترنت نگه دارید.

---

## فصل ۷: بهترین روش‌ها

### مقدمه

چکیده بهترین شیوه‌های امنیتی.

### مفاهیم اساسی

**چک‌لیست امنیتی:**
- [ ] احراز هویت فعال (LDAP/OAuth)
- [ ] RBAC پیاده‌شده
- [ ] CSRF فعال
- [ ] HTTPS اجباری
- [ ] Jenkins خارج از اینترنت مستقیم
- [ ] Secretها در Credentials (نه در کد)
- [ ] Secretها از stdin
- [ ] Agent→Controller محدود
- [ ] Script Security جدی
- [ ] افزونه‌ها منظم به‌روز
- [ ] پشتیبان‌گیری منظم
- [ ] لاگ و مانیتورینگ
- [ ] بازبینی دوره‌ای دسترسی‌ها
- [ ] حداقل دسترسی

### نکات مهم (Tips & Tricks)

- امنیت فرآیندی مداوم است، نه یک‌بار.
- هشدارهای امنیتی Jenkins را دنبال کنید.
- افزونه‌های قدیمی/بدون نگهداری را حذف کنید.

### خلاصه فصل

بهترین روش‌ها: احراز هویت، RBAC، HTTPS، Credentials امن و به‌روزرسانی منظم. امنیت یک فرآیند مداوم است.

---

## منابع و مراجع

- Securing Jenkins: [https://www.jenkins.io/doc/book/security/](https://www.jenkins.io/doc/book/security/)
- Credentials: [https://www.jenkins.io/doc/book/using/using-credentials/](https://www.jenkins.io/doc/book/using/using-credentials/)
- Role Strategy Plugin: [https://plugins.jenkins.io/role-strategy/](https://plugins.jenkins.io/role-strategy/)
- Jenkins Security Advisories: [https://www.jenkins.io/security/](https://www.jenkins.io/security/)
