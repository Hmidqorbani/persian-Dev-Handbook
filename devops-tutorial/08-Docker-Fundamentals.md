# فایل ۰۸ — مبانی داکر و کانتینریزاسیون

**مدت یادگیری**: ۱۰ ساعت
**سطح سختی**: مبتدی–متوسط
**پیش‌نیاز**: فایل ۰۲ (لینوکس و شبکه)، فایل ۰۴ (CI/CD)

---

## فهرست محتوا

- [فصل ۳۶: مفاهیم کانتینر و داکر (Container در برابر VM)](#فصل-۳۶-مفاهیم-کانتینر-و-داکر-container-در-برابر-vm)
- [فصل ۳۷: Dockerfile و ساخت Image (چندمرحله‌ای و بهترین روش‌ها)](#فصل-۳۷-dockerfile-و-ساخت-image-چندمرحلهای-و-بهترین-روشها)
- [فصل ۳۸: مدیریت Container، Volume و شبکه](#فصل-۳۸-مدیریت-container-volume-و-شبکه)
- [فصل ۳۹: Docker Compose (اپلیکیشن‌های چندکانتینری)](#فصل-۳۹-docker-compose-اپلیکیشنهای-چندکانتینری)
- [فصل ۴۰: بهینه‌سازی و عیب‌یابی داکر](#فصل-۴۰-بهینهسازی-و-عیبیابی-داکر)

---

## فصل ۳۶: مفاهیم کانتینر و داکر (Container در برابر VM)

### مقدمه

انقلاب DevOps بدون **کانتینرها** غیرممکن بود. کانتینرها بسته‌بندی اپلیکیشن به همراه تمام وابستگی‌هایش را ممکن کردند — «روی ماشین من کار می‌کرد» را برای همیشه تمام کردند. **Docker** محبوب‌ترین ابزار کانتینریزاسیون است. در این فصل تفاوت کانتینر و ماشین مجازی، و معماری داکر را درک می‌کنیم.

### مفاهیم اساسی

**کانتینر چیست؟**
یک کانتینر واحدی ایزوله از نرم‌افزار است که شامل کد، runtime، کتابخانه‌ها و تنظیمات است — اما **بدون سیستم‌عامل کامل**. از هستهٔ (kernel) لینوکس هاست استفاده می‌کند (namespaces و cgroups).

**کانتینر در برابر VM:**

| ویژگی | VM | Container |
|---|---|---|
| ایزولاسیون | سخت‌افزاری (Hypervisor) | سطح فرآیند (kernel مشترک) |
| سیستم‌عامل | هر VM OS کامل | اشتراک kernel |
| حجم | گیگابایت | مگابایت |
| زمان بوت | دقیقه‌ای | میلی‌ثانیه‌ای |
| چگالی | کم | زیاد |

**معماری Docker:**

| جزء | نقش |
|---|---|
| **Docker Daemon** (`dockerd`) | مغز، مدیریت objects |
| **Docker CLI** (`docker`) | رابط کاربر (شما) |
| **Image** | قالب فقط‌خواندنی (الگو) |
| **Container** | نمونهٔ اجرایی از Image |
| **Registry** | انبار Image (Docker Hub) |
| **Dockerfile** | دستورالعمل ساخت Image |

**Union File System / Layers:**
هر Image از لایه‌های (layers) فقط‌خواندنی ساخته می‌شود. لایهٔ بالایی (container layer) قابل نوشتن است. لایه‌های مشترک بین Imageها بازاستفاده می‌شوند (صرفه‌جویی فضا).

**OCI (Open Container Initiative):** استانداردی برای فرمت Image و runtime که Docker، Podman و دیگران رعایت می‌کنند.

### مثال‌های عملی

**۱. اجرای اولین کانتینر:**

```bash
# دانلود image و اجرا
docker run hello-world

# اجرای یک وب‌سرور
docker run -d -p 8080:80 --name web nginx:alpine
curl http://localhost:8080
```

**۲. بررسی لایه‌ها:**

```bash
docker image history nginx:alpine
docker image inspect nginx:alpine
```

**۳. معماری daemon:**

```bash
# وضعیت daemon
docker info
docker version
# systemd
sudo systemctl status docker
```

**۴. تفاوت VM و Container (تصویرسازی):**

```
VM:  [App][Guest OS][Hypervisor][Host OS][Hardware]
Ctr: [App A][App B] → [Container Engine] → [Host OS][Hardware]
```

### مقایسه و دیدگاه‌های مختلف

| ابزار | نوع | تفاوت |
|---|---|---|
| Docker | Daemon-based | محبوب‌ترین، اکوسیستم بزرگ |
| Podman | Daemonless | بدون root، امن‌تر (فصل ۱۰) |
| containerd | Runtime | سطح پایین، در K8s |
| Buildah | Build-only | تمرکز روی ساخت |

**دیدگاه‌ها:**
- Docker برای توسعه و یادگیری بهترین است (مستندات فراوان).
- در تولید، اغلب runtime واقعی `containerd` یا `cri-o` است (از طریق Docker یا مستقیم).
- کانتینرها جایگزین VM نیستند بلکه مکمل‌اند (امنیت: VM برای ایزولاسیون سخت‌افزاری).

### اشتباهات رایج

- **اجرای همه چیز با `sudo`:** Docker گروه دارد؛ کاربر را به `docker` اضافه کنید.
- **داکر دایمون روی root:** پیش‌فرض امنیتی دارد اما دستریر root به daemon = دستریر root به سیستم.
- **کانتینر به عنوان VM:** کانتینرها ephemeral هستند؛ state را درونشان ذخیره نکنید (از Volume استفاده کنید).
- **استفاده از `latest`:** نسخه نامشخص → عدم تکرارپذیری (reproducibility).
- **اجرای چندین process با systemd در کانتینر:** ضدالگو؛ یک process اصلی (one process per container).

### نکات مهم و ترفندها

- **بدون sudo:** `sudo usermod -aG docker $USER` (سپس logout/login).
- **`.dockerignore`:** مثل `.gitignore`، فایل‌های غیرضروری را از build context حذف کنید (سرعت build).
- **Image digest:** به جای tag از `@sha256:...` برای قطعیت استفاده کنید.
- **Resource limits:** `docker run --memory=512m --cpus=1` برای جلوگیری از قورت دادن منابع.
- **Rootless Docker:** اجرای daemon بدون root (امنیت بیشتر).
- **Prune منظم:** `docker system prune` برای آزادسازی فضا.

### خلاصه فصل

کانتینر واحدی ایزوله با اشتراک kernel است (برخلاف VM که OS کامل دارد) — سبک‌تر و سریع‌تر. Docker از Daemon + CLI + Image + Registry تشکیل شده؛ Imageها لایه‌لایه (union FS) ساخته می‌شوند. کلید: از `latest` اجتناب کنید، state را در Volume نگه دارید، یک process در هر کانتینر، و `.dockerignore` برای سرعت build.

---

## فصل ۳۷: Dockerfile و ساخت Image (چندمرحله‌ای و بهترین روش‌ها)

### مقدمه

**Dockerfile** دستورالعمل گام‌به‌گام ساخت Image است. نوشتن آن با کیفیت مستقیماً روی **اندازهٔ Image**، **امنیت** و **سرعت build** اثر می‌گذارد. در این فصل Dockerfile اصولی، الگوی Multi-stage، و بهترین روش‌ها را یاد می‌گیرید.

### مفاهیم اساسی

**دستورات اصلی Dockerfile:**

| دستور | کاربرد |
|---|---|
| `FROM` | Image پایه (اولین دستور) |
| `RUN` | اجرای دستور در زمان build (لایه جدید) |
| `COPY` / `ADD` | کپی فایل از host به image |
| `WORKDIR` | تغییر دایرکتوری کاری |
| `ENV` | متغیر محیطی |
| `EXPOSE` | اعلام پورت (مستنداتی) |
| `CMD` | دستور پیش‌فرض اجرا |
| `ENTRYPOINT` | نقطهٔ ورود غیرقابل تغییر |
| `USER` | کاربر اجرا |
| `HEALTHCHECK` | بررسی سلامت |

**لایه‌بندی (Layering):**
هر دستور `RUN`/`COPY`/`ADD` یک لایه ایجاد می‌کند. دستورات کم‌تغییر را **اول** بیاورید تا کش (cache) حفظ شود.

**Multi-stage Build:**
چندین `FROM` در یک Dockerfile. بیلد در stage اول (سنگین، با کامپایلر)، خروجی فقط به stage آخر (سبک، runtime) کپی می‌شود. نتیجه: Image نهایی بسیار کوچک.

**الگوهای Image:**
- **Slim:** دبیان کوچک شده.
- **Alpine:** توزیع مینیمال (~۵MB) — مراقب glibc/musl باشید.
- **Distroless:** فقط اپلیکیشن و کتابخانه‌هایش (بدون shell) — امن‌ترین.

### مثال‌های عملی

**۱. Dockerfile پایه (Node.js):**

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
EXPOSE 3000
USER node
CMD ["node", "server.js"]
```

**۲. Multi-stage Build (Go):**

```dockerfile
# Stage 1: build
FROM golang:1.22 AS builder
WORKDIR /src
COPY . .
RUN CGO_ENABLED=0 go build -o /app/server

# Stage 2: runtime (distroless)
FROM gcr.io/distroless/static-debian12
COPY --from=builder /app/server /server
USER nonroot:nonroot
EXPOSE 8080
ENTRYPOINT ["/server"]
```

**۳. بهینه‌سازی لایه (Cache-friendly):**

```dockerfile
FROM python:3.12-slim
WORKDIR /app
# اول فقط dependency (کم‌تغییر)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
# بعد source (پرتغییر)
COPY . .
CMD ["python", "app.py"]
```

**۴. HEALTHCHECK:**

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:8080/healthz || exit 1
```

**۵. Build و تگ:**

```bash
docker build -t myapp:v1.0 -f Dockerfile .
docker build -t myapp:$GIT_SHA .
```

### مقایسه و دیدگاه‌های مختلف

| پایه | حجم | امنیت | سرعت build |
|---|---|---|---|
| ubuntu | بزرگ | متوسط | کند |
| debian-slim | متوسط | متوسط | متوسط |
| alpine | کوچک | متوسط | سریع |
| distroless | خیلی کوچک | بالا | (build جدا) |

**دیدگاه‌ها:**
- برای توسعه: alpine یا slim (راحتی دیباگ).
- برای تولید: distroless یا slim با `USER nonroot` (امنیت).
- Alpine با احتیاط: musl libc گاهی با باینری‌های glibc تداخل دارد.

**Multi-stage:** تقریباً همیشه توصیه می‌شود — Image نهایی ۱۰-۲۰ برابر کوچک‌تر.

### اشتباهات رایج

- **`COPY . .` در اول:** هر تغییر کوچک کل image را rebuild می‌کند (کش از بین).
- **اجرا به عنوان root:** حمله به کانتینر = دستریر root به درون (اگرچه ایزوله).
- **`RUN apt-get upgrade` بدون cleanup:** حجم اضافه؛ `rm -rf /var/lib/apt/lists/*` اضافه کنید.
- **`latest` در FROM:** Image غیرقابل تکرار.
- **یک Dockerfile برای dev و prod:** از multi-stage و build args جدا کنید.
- **فراموش `WORKDIR`:** فایل‌ها در `/` پراکنده می‌شوند.

### نکات مهم و ترفندها

- **`--no-cache-dir` در pip:** جلوگیری از ذخیرهٔ کش در image.
- **Build Args:** `ARG NODE_ENV=production` و در build: `--build-arg`.
- **`.dockerignore` قوی:** `node_modules`، `.git`، فایل‌های محلی را حذف کنید.
- **Scan image:** `docker scout cves myapp:v1` یا Trivy (فصل ۲۴).
- **LABEL:** متادیتا (نسخه، maintainer) اضافه کنید.
- **Minimal base:** وقتی امکان دارد از `distroless` استفاده کنید.

### خلاصه فصل

Dockerfile دستورالعمل ساخت Image است. کلید: لایه‌بندی هوشمند (ثابت اول)، Multi-stage برای کوچک‌سازی، `USER nonroot` برای امنیت، و `HEALTHCHECK` برای نظارت. پایه‌های سبک (alpine/distroless) حجم و سطح حمله را کم می‌کنند. کلید: `.dockerignore`، `--no-cache-dir`، و اجتناب از `latest`/root.

---

## فصل ۳۸: مدیریت Container، Volume و شبکه

### مقدمه

ساختن Image فقط نیمی از کار است. اجرا، مدیریت، ذخیرهٔ داده و اتصال کانتینرها به هم مهارت‌های روزانهٔ یک مهندس DevOps است. در این فصل چرخهٔ عمر کانتینر، Volumeها برای دادهٔ پایدار، و شبکه‌سازی داکر را بررسی می‌کنیم.

### مفاهیم اساسی

**چرخهٔ عمر Container:**
`created → running → paused → stopped → removed`

**دستورات پایه:**

| دستور | کاربرد |
|---|---|
| `docker run` | ساخت + اجرا |
| `docker ps` | لیست کانتینرهای در حال اجرا |
| `docker exec` | اجرای دستور در کانتینر در حال اجرا |
| `docker logs` | مشاهده لاگ |
| `docker stop/start` | توقف/شروع |
| `docker rm` | حذف |

**Volume (حجم):**
دیسک پایدار خارج از لایهٔ قابل‌نوشتن کانتینر. داده در آن بین restartها باقی می‌ماند. دو نوع:
- **Named volume:** مدیریت‌شده توسط داکر (`docker volume create`).
- **Bind mount:** نقشه‌برداری پوشهٔ host به کانتینر.

**شبکه (Networking):**
داکر انواع شبکه دارد:
- **bridge** (پیش‌فرض): کانتینرها در شبکهٔ داخلی، با هم صحبت می‌کنند.
- **host:** اشتراک شبکهٔ host (بدون NAT).
- **none:** ایزوله.
- **overlay**: برای چند هاست (Swarm/K8s).

**مفهوم مهم:** کانتینر ephemeral است — هرگز دادهٔ مهم را درونش ذخیره نکنید.

### مثال‌های عملی

**۱. اجرا با گزینه‌های مختلف:**

```bash
docker run -d \
  --name web \
  -p 8080:80 \
  -e ENV=production \
  -v web-data:/usr/share/nginx/html \
  --restart unless-stopped \
  nginx:alpine
```

**۲. مدیریت و عیب‌یابی:**

```bash
docker ps -a                    # همه (حتی متوقف)
docker logs web --tail 50 -f    # لاگ زنده
docker exec -it web sh          # شل درون کانتینر
docker stats web                # مصرف منابع
docker inspect web              # جزئیات JSON
```

**۳. Volume:**

```bash
docker volume create db-data
docker run -d -v db-data:/var/lib/postgresql/data postgres:16
# یا bind mount:
docker run -v $(pwd)/config:/app/config myapp
```

**۴. شبکه سفارشی (برای ارتباط چند کانتینر):**

```bash
docker network create mynet
docker run -d --network mynet --name db postgres:16
docker run -d --network mynet --name app myapp
# app می‌تواند db را با نام میزبان "db" ببیند
```

### مقایسه و دیدگاه‌های مختلف

| ذخیره‌سازی | پایداری | کاربرد |
|---|---|---|
| Container layer | نه | موقت |
| Named volume | بله | دادهٔ اپلیکیشن (DB) |
| Bind mount | بله | تنظیمات/کد توسعه |

| شبکه | کاربرد |
|---|---|
| bridge | پیش‌فرض، چند کانتینر روی یک هاست |
| host | عملکرد بالا، بدون NAT |
| overlay | multi-host (تولید واقعی در K8s) |

**دیدگاه:** برای توسعه، bind mount برای کد (hot reload) عالی است. برای تولید، named volume یا external storage (PVC در K8s). شبکهٔ سفارشی بهتر از پیش‌فرض برای نام‌گذاری سرویس.

**Stateful در تولید:** در K8s، از PersistentVolume استفاده کنید نه docker volume ساده.

### اشتباهات رایج

- **ذخیره داده در container layer:** با `docker rm` داده از دست می‌رود.
- **فراموش `--restart`:** کانتینر پس از reboot اجرا نمی‌شود.
- **bind mount اشتباه:** مسیر host اشتباه → خالی بودن.
- **پورت‌های درگیر:** `docker run -p 8080:80` وقتی ۸۰۸۰ اشغال است شکست می‌خورد.
- **دسترسی volume با permission:** کانتینر با UID متفاوت نمی‌تواند بنویسد.
- **`docker rm` بدون volume:** گاهی volume هم باید حذف شود (`-v`).

### نکات مهم و ترفندها

- **`--rm`:** برای کانتینرهای موقت (پس از اتمام حذف شود): `docker run --rm alpine echo hi`.
- **`docker compose` برای چند کانتینر:** (فصل ۳۹).
- **Resource limits:** همیشه `--memory` و `--cpus` بگذارید تا یک کانتینر سیستم را نگیرد.
- **Read-only rootfs:** `--read-only` برای امنیت (فقط volume قابل نوشتن).
- **Log rotation:** تنظیم `json-file` max-size برای جلوگیری از پر شدن دیسک.
- **Healthcheck در run:** `docker run --health-cmd=...` یا از Dockerfile.

### خلاصه فصل

مدیریت کانتینر شامل اجرا (`run` با `-p`, `-v`, `-e`, `--restart`)، نظارت (`ps`, `logs`, `exec`, `stats`) و حذف است. Volumeها دادهٔ پایدار (named volume برای تولید، bind mount برای توسعه) را فراهم می‌کنند — کانتینر خودش ephemeral است. شبکهٔ سفارشی اجازه می‌دهد کانتینرها با نام سرویس صحبت کنند. کلید: `--restart unless-stopped`، resource limits، و هرگز داده در container layer.

---

## فصل ۳۹: Docker Compose (اپلیکیشن‌های چندکانتینری)

### مقدمه

اکثر اپلیکیشن‌های واقعی چندین سرویس دارند: وب‌سرور، پایگاه داده، کش، صف پیام. **Docker Compose** ابزاری است که این چند کانتینر را با یک فایل YAML تعریف و با یک دستور مدیریت می‌کند. در این فصل Compose را از صفر یاد می‌گیرید.

### مفاهیم اساسی

**Docker Compose چیست؟**
ابزار تعریف و اجرای اپلیکیشن‌های چندکانتینری. فایل `docker-compose.yml` (یا `compose.yaml`) سرویس‌ها، شبکه‌ها و Volumeها را توصیف می‌کند.

**عناصر اصلی:**

| کلید | کاربرد |
|---|---|
| `services` | کانتینرهای اپلیکیشن |
| `image` / `build` | منبع Image |
| `ports` | نگاشت پورت |
| `volumes` | حجم‌ها |
| `environment` | متغیرها |
| `depends_on` | وابستگی راه‌اندازی |
| `networks` | شبکه‌های سفارشی |
| `profiles` | گروه‌بندی سرویس (اختیاری راه‌اندازی) |

**دستورات:**

```bash
docker compose up -d       # اجرا در پس‌زمینه
docker compose down       # توقف و حذف
docker compose ps         # وضعیت
docker compose logs -f    # لاگ
docker compose build      # بازسازی images
```

**`depends_on`:** فقط ترتیب راه‌اندازی را کنترل می‌کند، نه آمادگی سرویس — برای انتظار واقعی از `healthcheck` + `depends_on: condition` استفاده کنید.

### مثال‌های عملی

**۱. فایل `compose.yaml` کامل:**

```yaml
services:
  web:
    build: .
    ports:
      - "8080:3000"
    environment:
      - DATABASE_URL=postgres://db:5432/app
    depends_on:
      db:
        condition: service_healthy
    networks: [appnet]

  db:
    image: postgres:16-alpine
    environment:
      - POSTGRES_PASSWORD=secret
      - POSTGRES_DB=app
    volumes:
      - db-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5
    networks: [appnet]

  redis:
    image: redis:7-alpine
    networks: [appnet]

volumes:
  db-data:

networks:
  appnet:
    driver: bridge
```

**۲. اجرا:**

```bash
docker compose up -d --build
docker compose ps
docker compose logs -f web
```

**۳. پروفایل (سرویس اختیاری):**

```yaml
services:
  web:
    ...
  adminer:
    image: adminer
    ports: ["8081:8080"]
    profiles: ["debug"]    # فقط با: docker compose --profile debug up
```

**۴. متغیرها از فایل `.env`:**

```bash
# .env
DB_PASSWORD=mysecret
```
```yaml
environment:
  - POSTGRES_PASSWORD=${DB_PASSWORD}
```

### مقایسه و دیدگاه‌های مختلف

| ابزار | کاربرد |
|---|---|
| Docker Compose | تک‌هاست، توسعه/تست |
| Docker Swarm | تولید ساده، multi-host |
| Kubernetes | تولید مقیاس‌پذیر (فصل ۱۱+) |

**دیدگاه‌ها:**
- Compose برای local dev و CI عالی است.
- برای تولید واقعی، Compose کافی نیست (no auto-healing, scaling محدود) → K8s یا Swarm.
- Compose Spec اکنون استانداردی فراتر از داکر است (Podman Compose هم پشتیبانی می‌کند).

**`depends_on` + healthcheck:** ترکیب آن‌ها جایگزین ابزارهای انتظار (wait-for-it) می‌شود.

### اشتباهات رایج

- **`depends_on` بدون healthcheck:** web قبل از آمادگی db شروع می‌شود → خطا.
- **ورژن قدیمی `version: '3'`:** در Compose جدید نیاز نیست (حذف شود).
- **Volume ندادن به DB:** داده با `down` پاک می‌شود (از named volume استفاده کنید).
- **پورت‌های hard-code:** از `${PORT:-8080}` برای انعطاف.
- **اجرای `docker-compose` قدیم:** دستور جدید `docker compose` (بدون خط تیره) است.

### نکات مهم و ترفندها

- **`docker compose watch`** (جدید): hot-reload در توسعه.
- **Override file:** `docker-compose.override.yml` برای محیط محلی (پیش‌فرض بارگذاری می‌شود).
- **`scale`:** `docker compose up -d --scale web=3` (نیاز به load balancer).
- **`--build` همیشه در dev:** تغییرات را بازتاب دهد.
- **Secrets in compose:** از کلید `secrets` به جای `environment` برای رمز استفاده کنید (امنیت).
- **Resource limits در compose:**
  ```yaml
  deploy:
    resources:
      limits:
        memory: 512M
  ```

### خلاصه فصل

Docker Compose چند کانتینر را با `compose.yaml` یک‌جا تعریف و مدیریت می‌کند (`up`/`down`/`logs`). سرویس‌ها را با `depends_on` + `healthcheck` به ترتیب و پس از آمادگی راه‌اندازی کنید. Volume برای دادهٔ پایدار (DB) الزامی است. کلید: named volume برای state، `.env` برای متغیر، profiles برای سرویس اختیاری، و ترکیب healthcheck با depends_on به جای اسکریپت‌های wait.

---

## فصل ۴۰: بهینه‌سازی و عیب‌یابی داکر

### مقدمه

در دنیای واقعی، Imageهای بزرگ، buildهای کند، و باگ‌های عجیب کانتینر شما را متوقف می‌کنند. این فصل ابزارها و تکنیک‌های **بهینه‌سازی** (حجم، زمان build) و **عیب‌یابی** (داکر دایمون، شبکه، Docker-in-Docker) را جمع می‌کند.

### مفاهیم اساسی

**معیارهای بهینه‌سازی:**
- **Image size:** کوچک‌تر = pull سریع‌تر، سطح حمله کمتر.
- **Build time:** لایه‌بندی + cache.
- **Runtime footprint:** منابع (CPU/RAM) کانتینر.

**ابزارهای تحلیل:**
- `docker images` / `docker system df` (فضا).
- `docker scout` / `trivy` (امنیت/لایه).
- `dive` (تحلیل لایه‌ها).
- `docker stats` (منابع runtime).

**عیب‌یابی لایه‌ای:**
1. آیا daemon اجرا می‌شود؟ (`systemctl status docker`)
2. آیا image دانلود/ساخته شد؟ (`docker images`)
3. آیا کانتینر start می‌شود؟ (`docker ps -a`)
4. لاگ چه می‌گوید؟ (`docker logs`)
5. شبکه/volume درست است؟ (`docker inspect`)

**Docker-in-Docker (dind):** اجرای daemon داکر درون کانتینر (برای build در CI). نیازمند `privileged` — ریسک امنیتی؛ از **Rootless dind** یا **Kaniko/BuildKit** استفاده کنید.

### مثال‌های عملی

**۱. کوچک‌سازی حجم:**

```bash
# قبل و بعد
docker images myapp
# myapp:v1  1.2GB
# myapp:v2 (multi-stage) 24MB
```

**۲. تحلیل لایه با dive:**

```bash
dive myapp:v1
# نشان می‌دهد کدام لایه حجم اضافه دارد
```

**۳. BuildKit (سریع‌تر):**

```bash
export DOCKER_BUILDKIT=1
docker build --progress=plain -t myapp .
```

**۴. عیب‌یابی build شکست‌خورده:**

```bash
# اجرای مرحلهٔ میانی برای دیباگ
docker run -it --entrypoint sh myapp:broken
```

**۵. dind در CI (ایمن‌تر):**

```yaml
services:
  docker:
    image: docker:dind
    privileged: true   # فقط در محیط قابل اعتماد
```

### مقایسه و دیدگاه‌های مختلف

| چالش | راه‌حل |
|---|---|
| Image بزرگ | Multi-stage، distroless، alpine |
| Build کند | لایه‌بندی، BuildKit، cache |
| dind ناامن | Rootless dind، Kaniko، BuildKit |
| فضای دیسک پر | `docker system prune -a` |
| کانتینر crash | `docker logs` + `inspect` |

**دیدگاه:** در CI مدرن، به جای dind از **Kaniko** یا **BuildKit** استفاده کنید (بدون privileged). برای عیب‌یابی، `dive` و `docker scout` طلایی هستند.

**تجارت (Trade-off):** alpine سریع/کوچک اما ممکن است با glibc تداخل کند؛ distroless امن اما بدون shell (دیباگ سخت‌تر).

### اشتباهات رایج

- **`docker system prune` بدون احتیاط:** containerهای در حال اجرا را هم حذف نمی‌کند اما images بی‌استفاده را پاک می‌کند.
- **dind با privileged در محیط ناامن:** فرار از کانتینر ممکن است.
- **لاگ‌های بی‌انتها:** بدون log rotation دیسک پر می‌شود.
- **فراموش `--progress=plain`:** خطای build مخفی می‌ماند.
- **build بدون BuildKit:** کندتر از حد ممکن.

### نکات مهم و ترفندها

- **`docker system df`:** دید کلی از فضای مصرفی.
- **BuildKit secrets:** ارسال رمز در build بدون لایه شدن:
  ```dockerfile
  RUN --mount=type=secret,id=token cat /run/secrets/token
  ```
- **Kaniko:** build بدون daemon (در K8s ایده‌آل).
- **`.dockerignore` جامع:** کلید سرعت build.
- **Labels استاندارد:** OCI labels برای ردیابی.
- **`docker context`:** سوییچ بین daemonهای مختلف (مثل remote docker).

### خلاصه فصل

بهینه‌سازی داکر = کاهش حجم (multi-stage/distroless) + سرعت build (لایه‌بندی/BuildKit/cache) + کمینه کردن footprint. عیب‌یابی لایه‌ای: daemon → image → container → logs → network/volume. برای build در CI، dind با privileged ریسک دارد؛ از Kaniko یا BuildKit rootless استفاده کنید. ابزارها: `dive` (لایه)، `docker scout`/`trivy` (امنیت)، `docker system prune` (فضا).

---

## منابع و مراجع

- Docker Documentation: https://docs.docker.com
- Dockerfile Reference: https://docs.docker.com/reference/dockerfile/
- Best Practices for Dockerfile: https://docs.docker.com/build/building/best-practices/
- Docker Compose Spec: https://compose-spec.io
- Multi-stage Builds: https://docs.docker.com/build/building/multi-stage/
- Dive (image analysis): https://github.com/wagoodman/dive
- Docker Scout: https://docs.docker.com/scout/
- Kaniko: https://github.com/GoogleContainerTools/kaniko
- OCI Image Spec: https://github.com/opencontainers/image-spec

**بعدی:** [فایل ۰۹ — داکر پیشرفته و رجیستری](09-Docker-Advanced-and-Registry.md)
