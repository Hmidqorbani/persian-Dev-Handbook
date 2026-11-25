# بخش ۰۹: نصب و راه‌اندازی Jenkins

**مدت یادگیری**: ۳ ساعت
**سطح سختی**: مبتدی
**پیش‌نیاز**: بخش ۰۸ (مبانی CI/CD)، آشنایی با Docker

---

## فهرست محتوا

- [فصل ۱: Jenkins چیست؟](#فصل-۱-jenkins-چیست)
- [فصل ۲: پیش‌نیازها](#فصل-۲-پیشنیازها)
- [فصل ۳: روش‌های نصب](#فصل-۳-روشهای-نصب)
- [فصل ۴: راه‌اندازی اولیه](#فصل-۴-راهاندازی-اولیه)
- [فصل ۵: آشنایی با رابط کاربری](#فصل-۵-آشنایی-با-رابط-کاربری)
- [فصل ۶: پیکربندی سراسری](#فصل-۶-پیکربندی-سراسری)
- [فصل ۷: ساختار JENKINS_HOME](#فصل-۷-ساختار-jenkins_home)

---

## فصل ۱: Jenkins چیست؟

### مقدمه

**Jenkins** یک سرور اتوماسیون متن‌باز و مبتنی بر Java است که برای CI/CD استفاده می‌شود. Jenkins فرآیندهای ساخت، تست و استقرار را خودکار می‌کند.

### اهمیت این موضوع

Jenkins از سال ۲۰۱۱ (انشعاب از Hudson) به استاندارد صنعت تبدیل شده و با بیش از ۱۸۰۰ افزونه، تقریباً هر سناریویی را پوشش می‌دهد.

### مفاهیم اساسی

**معماری Jenkins:**
- **Controller (Master)**: مغز سیستم؛ زمان‌بندی Jobها، مدیریت رابط کاربری، ذخیره پیکربندی.
- **Agent (Node/Slave)**: ماشین‌هایی که کار واقعی (Build) روی آن‌ها اجرا می‌شود.
- **Executor**: یک اسلات اجرای کار روی یک Agent.

```
        ┌──────────────┐
        │  Controller  │
        └──────┬───────┘
     ┌─────────┼─────────┐
     ↓         ↓         ↓
  Agent1    Agent2    Agent3
 (Linux)   (Windows) (Docker)
```

**مفاهیم کلیدی:**
- **Job/Project**: یک واحد کار (Build، Test، Deploy).
- **Build**: یک اجرای خاص از یک Job.
- **Workspace**: پوشه کاری Job روی Agent.
- **Plugin**: افزونه‌ای که قابلیت اضافه می‌کند.

### اشکال شایع

- **اشتباه رایج**: فکر کردن Jenkins یک ابزار ساده است. Jenkins یک پلتفرم کامل با پیچیدگی‌های خودش است.
- **اشتباه رایج**: اجرای همه کارها روی Controller. این کار امنیت و کارایی را کاهش می‌دهد.

### نکات مهم (Tips & Tricks)

- همیشه Controller را از Agentها جدا کنید (بخش ۱۴).
- Jenkins را با Docker اجرا کنید تا نصب و به‌روزرسانی ساده باشد.

### خلاصه فصل

Jenkins یک سرور اتوماسیون مبتنی بر Java است با معماری Controller/Agent. با افزونه‌ها گسترش می‌یابد و استاندارد صنعت CI/CD است.

---

## فصل ۲: پیش‌نیازها

### مقدمه

قبل از نصب، باید بدانید چه چیزی نیاز دارید.

### مفاهیم اساسی

**حداقل منابع:**
- **Java**: Jenkins 2.4xx به Java 11 یا 17 نیاز دارد (نسخه‌های جدید 17/21).
- **RAM**: حداقل ۲ گیگابایت، توصیه ۴+ گیگابایت.
- **دیسک**: حداقل ۱۰ گیگابایت آزاد.
- **CPU**: ۲ هسته توصیه‌شده.

**بررسی Java:**
```bash
java -version
# باید نسخه 11، 17 یا 21 را نشان دهد
```

نصب Java 17 (Ubuntu):
```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
```

### اشکال شایع

- **اشتباه رایج**: نصب Jenkins روی سیستمی با Java قدیمی. خطای سازگاری می‌دهد.
- **اشتباه رایج**: منابع ناکافی. Jenkins با Buildهای سنگین به‌سرعت کم‌حافظه می‌شود.

### نکات مهم (Tips & Tricks)

- با Docker، نیازی به نصب دستی Java نیست؛ image آن را دارد.
- برای محیط تولید، از یک ماشین اختصاصی استفاده کنید.

### خلاصه فصل

Jenkins به Java (11/17/21)، حداقل ۲ گیگ RAM و ۱۰ گیگ دیسک نیاز دارد. Docker ساده‌ترین راه برای شروع است.

---

## فصل ۳: روش‌های نصب

### مقدمه

Jenkins را به چند روش می‌توان نصب کرد. هر روش مزایا و معایب خود را دارد.

### مفاهیم اساسی

**۱. Docker (توصیه‌شده برای یادگیری):**
```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts-jdk17
```

**۲. WAR file (انعطاف‌پذیر):**
```bash
wget https://get.jenkins.io/war-stable/latest/jenkins.war
java -jar jenkins.war --httpPort=8080
```

**۳. بسته سیستمی (Ubuntu/Debian):**
```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/" | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins -y
sudo systemctl start jenkins
```

**۴. Helm (Kubernetes):**
```bash
helm repo add jenkins https://charts.jenkins.io
helm repo update
helm install jenkins jenkins/jenkins
```

### مقایسه و الگوهای مختلف

| روش | مزیت | عیب |
|-----|------|-----|
| Docker | ساده، ایزوله | نیاز به Docker |
| WAR | بدون نصب | اجرای دستی |
| بسته سیستمی | سرویس systemd | وابسته به OS |
| Helm | مقیاس‌پذیر | پیچیده |

### اشکال شایع

- **اشتباه رایج**: نصب Docker بدون volume. با حذف container، همه داده از بین می‌رود. همیشه `-v` بدهید.
- **اشتباه رایج**: باز کردن پورت ۵۰۰۰۰ به اینترنت. فقط داخلی باشد.

### نکات مهم (Tips & Tricks)

- برای یادگیری، Docker بهترین گزینه است.
- برای تولید، بسته سیستمی یا Kubernetes.
- همیشه `JENKINS_HOME` را روی volume پایدار قرار دهید.

### خلاصه فصل

Jenkins را با Docker، WAR، بسته سیستمی یا Helm نصب می‌کنید. Docker برای شروع و Kubernetes برای مقیاس توصیه می‌شود. همیشه داده را پایدار نگه دارید.

---

## فصل ۴: راه‌اندازی اولیه

### مقدمه

بعد از نصب، Jenkins را باید راه‌اندازی اولیه کنید.

### مفاهیم اساسی

**مراحل:**
1. باز کردن `http://localhost:8080`.
2. وارد کردن رمز اولیه (Initial Admin Password).
3. نصب افزونه‌های پیشنهادی.
4. ساخت کاربر ادمین.
5. تعیین URL.

**یافتن رمز اولیه:**

Docker:
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Linux:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

**نصب افزونه‌ها:** «Install suggested plugins» را انتخاب کنید (شامل Git، Pipeline، Credentials و...).

### مثال‌های عملی

یک اسکریپت راه‌اندازی Docker کامل:

```bash
docker run -d --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  --restart=on-failure \
  jenkins/jenkins:lts-jdk17

# صبر کنید تا بالا بیاید
sleep 30
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### اشکال شایع

- **اشتباه رایج**: رد کردن نصب افزونه‌های پیشنهادی. بعداً مجبور می‌شوید دستی نصب کنید.
- **اشتباه رایج**: فراموش کردن رمز ادمین. در `JENKINS_HOME` قابل بازیابی نیست؛ باید ریست کنید.

### نکات مهم (Tips & Tricks)

- افزونه‌های پیشنهادی را نصب کنید؛ بعداً می‌توانید حذف کنید.
- URL را درست تنظیم کنید (مخصوصاً اگر پشت proxy هستید).
- از همان ابتدا کاربر ادمین قوی بسازید.

### خلاصه فصل

راه‌اندازی اولیه شامل رمز اولیه، نصب افزونه‌ها و ساخت ادمین است. با Docker این فرآیند چند دقیقه‌ای است.

---

## فصل ۵: آشنایی با رابط کاربری

### مقدمه

رابط کاربری Jenkins ساده اما پر از امکانات است. در این فصل بخش‌های اصلی را می‌شناسیم.

### مفاهیم اساسی

**منوی اصلی:**
- **New Item**: ساخت Job جدید.
- **Build Executor Status**: نمایش Executorها.
- **Build Queue**: صف Buildها.
- **Manage Jenkins**: پیکربندی سیستم.
- **My Views / Dashboard**: نمای Jobها.

**زیرمجموعه‌های Manage Jenkins:**
- System Configuration
- Global Tool Configuration
- Manage Plugins
- Manage Credentials
- Manage Nodes and Clouds
- Security

### مثال‌های عملی

ساخت اولین Job:
1. New Item
2. نام و نوع (Freestyle یا Pipeline)
3. OK
4. پیکربندی و Save
5. Build Now

### اشکال شایع

- **اشتباه رایج**: سردرگمی در منوی Manage Jenkins. بخش‌ها را یک‌بار مرور کنید.
- **اشتباه رایج**: نداشتن View مرتب. با Folderها سازماندهی کنید.

### نکات مهم (Tips & Tricks)

- از Folder برای گروه‌بندی Jobها استفاده کنید.
- Blue Ocean (افزونه) رابط کاربری مدرن‌تری ارائه می‌دهد.
- `Manage Jenkins → System Log` برای عیب‌یابی.

### خلاصه فصل

رابط کاربری Jenkins شامل Dashboard، New Item و Manage Jenkins است. با Folderها سازماندهی کنید و Blue Ocean را امتحان کنید.

---

## فصل ۶: پیکربندی سراسری

### مقدمه

پیکربندی سراسری، ابزارهای موردنیاز Jobها را تعریف می‌کند.

### مفاهیم اساسی

**Global Tool Configuration:**
- **JDK**: مسیر Java برای Build.
- **Git**: نصب Git روی Agent.
- **Maven**: برای پروژه‌های Java.
- **Gradle، NodeJS، Docker**: ابزارهای دیگر.

**System Configuration:**
- مسیر Jenkins URL
- تعداد Executor
- تنظیمات ایمیل (SMTP)
- تنظیمات Proxy
- مسیر Home

### مثال‌های عملی

افزودن Maven:

1. Manage Jenkins → Global Tool Configuration
2. Maven → Add Maven
3. نام: `maven-3.9`
4. Install automatically
5. Save

در Pipeline:
```groovy
tools {
    maven 'maven-3.9'
    jdk 'jdk17'
}
```

### اشکال شایع

- **اشتباه رایج**: Hardcode کردن مسیر ابزارها در Jobها. از Global Tools استفاده کنید.
- **اشتباه رایج**: فراموش کردن تنظیم SMTP برای اعلان‌ها.

### نکات مهم (Tips & Tricks)

- نام‌های معنادار برای ابزارها بگذارید.
- «Install automatically» را فعال کنید تا Jenkins خودش نصب کند.
- ایمیل را برای اعلان شکست Build تنظیم کنید.

### خلاصه فصل

Global Tool Configuration ابزارهای مشترک (JDK، Git، Maven) را تعریف می‌کند. System Configuration تنظیمات کلی سیستم را مدیریت می‌کند.

---

## فصل ۷: ساختار JENKINS_HOME

### مقدمه

`JENKINS_HOME` تمام داده Jenkins را نگه می‌دارد. شناخت ساختار آن برای پشتیبان‌گیری و عیب‌یابی ضروری است.

### مفاهیم اساسی

```
JENKINS_HOME/
├── config.xml              # پیکربندی اصلی
├── jobs/                   # تعریف Jobها
│   └── my-job/
│       ├── config.xml
│       └── builds/         # تاریخچه Build
├── plugins/                # افزونه‌های نصب‌شده
├── users/                  # کاربران
├── secrets/                # کلیدها و رمزهای حساس
├── nodes/                  # تعریف Agentها
├── workspace/              # فضای کاری Jobها
├── logs/                   # لاگ‌ها
└── updates/                # اطلاعات به‌روزرسانی
```

### مثال‌های عملی

پشتیبان‌گیری ساده:

```bash
# توقف Jenkins (اختیاری)
docker stop jenkins

# پشتیبان‌گیری از volume
docker run --rm \
  -v jenkins_home:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/jenkins-backup.tar.gz -C /data .

docker start jenkins
```

### اشکال شایع

- **اشتباه رایج**: حذف تصادفی `JENKINS_HOME`. همه‌چیز از بین می‌رود.
- **اشتباه رایج**: پشتیبان‌نگرفتن. حتماً برنامه پشتیبان‌گیری داشته باشید.

### نکات مهم (Tips & Tricks)

- `JENKINS_HOME` را روی یک volume پایدار نگه دارید.
- پشتیبان‌گیری منظم (بخش ۱۶).
- `secrets/` بسیار حساس است؛ محافظت کنید.

### خلاصه فصل

`JENKINS_HOME` شامل config، jobs، plugins، users و secrets است. شناخت آن برای پشتیبان‌گیری و عیب‌یابی حیاتی است.

---

## منابع و مراجع

- نصب Jenkins: [https://www.jenkins.io/doc/book/installing/](https://www.jenkins.io/doc/book/installing/)
- Docker image رسمی: [https://hub.docker.com/r/jenkins/jenkins](https://hub.docker.com/r/jenkins/jenkins)
- Jenkins Handbook: [https://www.jenkins.io/doc/book/](https://www.jenkins.io/doc/book/)
- JENKINS_HOME: [https://www.jenkins.io/doc/book/managing/](https://www.jenkins.io/doc/book/managing/)
