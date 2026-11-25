# فایل ۱۰ — Podman و Buildah: جایگزین‌های بدون دایمون

**مدت یادگیری**: ۷ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۸ و ۰۹ (داکر مبانی و پیشرفته)

---

## فهرست محتوا

- [فصل ۴۶: فلسفهٔ Daemonless و Rootless (Podman در برابر Docker)](#فصل-۴۶-فلسفهٔ-daemonless-و-rootless-podman-در-برابر-docker)
- [فصل ۴۷: Podman CLI و سازگاری با داکر](#فصل-۴۷-podman-cli-و-سازگاری-با-داکر)
- [فصل ۴۸: Podman Play و مفهوم Pod](#فصل-۴۸-podman-play-و-مفهوم-pod)
- [فصل ۴۹: Buildah و Skopeo (ساخت و جابه‌جایی Image)](#فصل-۴۹-buildah-و-skopeo-ساخت-و-جابهجایی-image)
- [فصل ۵۰: یکپارچه‌سازی Podman در CI/CD و بهترین روش‌ها](#فصل-۵۰-یکپارچهسازی-podman-در-cicd-و-بهترین-روشها)

---

## فصل ۴۶: فلسفهٔ Daemonless و Rootless (Podman در برابر Docker)

### مقدمه

داکر به دلیل معماری daemon-based خود، نقطهٔ ضعف امنیتی دارد: daemon با دستریر root اجرا می‌شود. **Podman** با شعار «Daemonless، Rootless، Docker-compatible» آمد تا این مشکلات را حل کند. در این فصل تفاوت‌های معماری و فلسفه را بررسی می‌کنیم.

### مفاهیم اساسی

**تفاوت معماری:**

| ویژگی | Docker | Podman |
|---|---|---|
| معماری | Client-Server (daemon `dockerd`) | Daemonless (client مستقیم به runc) |
| اجرا | نیاز به daemon root | بدون daemon، rootless |
| امنیت | daemon = root | هر کاربر کانتینر خودش |
| Pod | ندارد (تک کانتینر) | دارد (مثل K8s) |
| Socket | `/var/run/docker.sock` | بدون socket |

**چرا Daemonless؟**
در داکر، حتی اگر کانتینر شما غیر-root باشد، **daemon** با root اجرا می‌شود. اگر daemon آسیب‌پذیر باشد، کل هاست خطرناک است. Podman این را با حذف daemon از بین می‌برد — هر کاربر کانتینرهایش را با هویت خودش اجرا می‌کند (rootless).

**کلمهٔ Pod در Podman:**
Podman نامش را از **Pod** گرفته — گروهی از کانتینرهایی که شبکه و storage را **مشترک** دارند (دقیقاً مثل Pod در Kubernetes). این پل پیوند به K8s است.

**OCI استاندارد:**
هر دو از فرمت OCI استفاده می‌کنند؛ Image ساخته‌شده در Podman در داکر اجرا می‌شود و برعکس.

**libpod:** کتابخانهٔ پشت Podman که مدیریت کانتینر را انجام می‌دهد (مثل containerd در داکر).

### مثال‌های عملی

**۱. نصب Podman (لینوکس):**

```bash
# Ubuntu/Debian
sudo apt update && sudo apt install -y podman

# RHEL/Fedora
sudo dnf install -y podman

# بررسی
podman version
podman info
```

**۲. اجرای rootless (به عنوان کاربر عادی):**

```bash
# بدون sudo
podman run -d -p 8080:80 nginx:alpine
podman ps
# کانتینر با UID کاربر شما اجرا می‌شود
```

**۳. مقایسه با داکر (همان دستور):**

```bash
# در داکر: docker run ...
# در پادمن: podman run ...  (کاملاً مشابه)
podman run --rm alpine cat /etc/os-release
```

**۴. بررسی rootless:**

```bash
podman unshare cat /proc/self/uid_map
# نشان می‌دهد نگاشت UID کاربر به root مجازی درون کانتینر
```

### مقایسه و دیدگاه‌های مختلف

| سناریو | Docker | Podman |
|---|---|---|
| امنیت بالا (multi-tenant) | Rootless mode پیچیده | بومی rootless |
| یادگیری/توسعه | عالی | عالی (سازگار) |
| K8s-like pods | نه | بله |
| اکوسیستم ابزار | بزرگ‌تر | رو به رشد |
| CI/CD سنتی | رایج | نیاز به تنظیم |

**دیدگاه‌ها:**
- برای توسعه‌دهنده: Podman تقریباً جایگزین بی‌دردسر داکر است (حتی alias `docker=podman` می‌دهند).
- برای تولید/K8s: بسیاری runtime را به containerd یا cri-o (هم‌خانواده Podman) تغییر داده‌اند.
- Red Hat/CentOS/RHEL داکر را حذف و Podman را پیش‌فرض کرده‌اند.

**انتقال (Migration):** بسیاری از اسکریپت‌ها فقط با تغییر `docker` به `podman` کار می‌کنند (alias پیشنهاد می‌شود).

### اشتباهات رایج

- **فرض root بودن:** در rootless، binding پورت زیر ۱۰۲۴ (مثل ۸۰) ممنوع است → از ۸۰۸۰ استفاده کنید یا `sudo setcap`.
- **فراموش `--rm` در rootless:** کانتینرهای زیاد فضای home کاربر را پر می‌کنند.
- **Volume با permission:** UID درون کانتینر (مثلاً ۰) با کاربر هاست متفاوت → خطای نوشتن.
- **Docker socket انتظار:** ابزارهایی که `docker.sock` می‌خواهند در Podman کار نمی‌کنند (از Podman socket فعال یا docksy استفاده کنید).
- **Cgroups v1 vs v2:** تنظیمات متفاوت در توزیع‌های قدیمی.

### نکات مول و ترفندها

- **Alias:** `alias docker=podman` برای سازگاری اسکریپت‌های قدیمی.
- **Podman socket برای Docker-compatible API:**
  ```bash
  systemctl --user enable --now podman.socket
  export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/podman/podman.sock
  ```
- **`podman machine` (مک/ویندوز):** مثل `docker machine`، VM لینوکس سبک می‌سازد.
- **Auto-update:** `podman auto-update` برای کانتینرهای با label.
- **Quadlet:** تعریف کانتینرها به صورت systemd unit (جایگزین compose برای تولید).
- **`podman generate kube`:** تبدیل کانتینر به YAML کوبرنتیز (پل به K8s).

### خلاصه فصل

Podman جایگزین daemonless و rootless داکر است — بدون daemon، هر کاربر کانتینر خودش را با هویت خودش اجرا می‌کند (امنیت بالاتر). سازگاری CLI کامل با داکر دارد (حتی alias). ویژگی منحصربه‌فرد: مفهوم **Pod** (مثل K8s) که پل به کوبرنتیز است. OCI-compliant → Imageها قابل تبادل. کلید: درک rootless (پورت <۱۰۲۴ ممنوع)، و استفاده از podman socket برای سازگاری ابزارهای قدیمی.

---

## فصل ۴۷: Podman CLI و سازگاری با داکر

### مقدمه

بزرگ‌ترین مزیت Podman برای پذیرش، **سازگاری تقریباً کامل با دستورات داکر** است. توسعه‌دهنده می‌تواند بدون یادگیری جدید مهاجرت کند. در این فصل نقشهٔ برابری دستورات و تفاوت‌های ظریف را می‌بینیم.

### مفاهیم اساسی

**نقشهٔ برابری (Command Parity):**

| Docker | Podman | توضیح |
|---|---|---|
| `docker run` | `podman run` | یکسان |
| `docker ps` | `podman ps` | یکسان |
| `docker build` | `podman build` | یکسان (یا via Buildah) |
| `docker images` | `podman images` | یکسان |
| `docker compose` | `podman compose` | بر پایهٔ اختیاری |
| `docker volume` | `podman volume` | یکسان |
| `docker network` | `podman network` | یکسان |

**تفاوت‌های کلیدی:**
- Podman پرچم‌های اضافه دارد: `--pod` (برای اضافه کردن به pod)، `--tls-verify=false` (برای رجیستری‌های خودامضا).
- در rootless، مسیر ذخیره در `~/.local/share/containers/` است نه `/var/lib/docker`.
- `podman` پرچم `--cgroup-manager` را برای systemd می‌دهد.

**Docker Compose سازگاری:**
Podman می‌تواند docker-compose را اجرا کند (با نصب پلاگین `podman-compose` یا `docker-compose` که به socket پادمن وصل شود).

**Podman Desktop:** رابط گرافیکی مشابه Docker Desktop برای مک/ویندوز/لینوکس.

### مثال‌های عملی

**۱. جایگزینی مستقیم:**

```bash
# همان خروجی
docker ps -a
podman ps -a

docker run -it --rm ubuntu bash
podman run -it --rm ubuntu bash
```

**۲. Build با Podman (rootless):**

```bash
podman build -t myapp:latest -f Containerfile .
# یا Dockerfile (نام یکسان پشتیبانی می‌شود)
```

**۳. Podman Compose:**

```bash
# نصب
pip install podman-compose
# اجرا (مثل docker-compose)
podman-compose up -d
```

**۴. اتصال ابزارهای داکر به Podman:**

```bash
# فعال‌سازی socket سازگار
systemctl --user start podman.socket
export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/podman/podman.sock
# حالا docker-compose، testcontainers و... کار می‌کنند
```

**۵. تفاوت در ذخیره:**

```bash
podman info | grep graphRoot
# graphRoot: /home/user/.local/share/containers/storage
```

### مقایسه و دیدگاه‌های مختلف

| جنبه | Docker | Podman |
|---|---|---|
| یادگیری | صفر (اگر داکر بلدید) | صفر (سازگار) |
| اسکریپت‌های موجود | کار می‌کند | با alias کار می‌کند |
| Compose | بومی | podman-compose/سازگار |
| Desktop GUI | Docker Desktop | Podman Desktop |

**دیدگاه:** برای مهاجرت سازمانی، Podman "drop-in replacement" داکر است. تنها جاهایی که تفاوت معنادار است: ابزارهایی که مستقیماً `docker.sock` را لمس می‌کنند (با socket سازگار حل می‌شود) و ویژگی‌های خاص داکر (مثل Swarm — Podman آن را ندارد، K8s را ترجیح می‌دهد).

**توصیه:** آلیاس `docker→podman` بزنید تا تیم عادت نکند به daemon.

### اشتباهات رایج

- **انتظار دستور Swarm:** Podman Swarm ندارد؛ برای ارکستراسیون از K8s استفاده کنید.
- **فراموش rootless در script:** `sudo podman` اشتباه است (مگر واقعاً بخواهید rootful).
- **عدم تنظیم DOCKER_HOST:** ابزارهای قدیمی socket نمی‌یابند.
- **`buildah` را جدا یاد نمی‌گیرند:** Podman build در واقع Buildah صدا می‌زند (فصل ۴۹).
- **مخلوط کردن daemon و rootless:** imageهای rootful و rootless در storage متفاوت‌اند.

### نکات مهم و ترفندها

- **`podman container cleanup`:** پاکسازی کانتینرهای مرده.
- **Shell completion:** فعال‌سازی برای bash/zsh/fish.
- **`podman system migrate`:** انتقال storage بین نسخه‌ها.
- **`--log-level=debug`:** برای عیب‌یابی rootless.
- **Podman on Windows/Mac:** `podman machine init` + `podman machine start`.
- **`podman search` / `podman pull`:** مستقیم از Docker Hub و OCI registries.

### خلاصه فصل

Podman CLI تقریباً کاملاً با داکر یکسان است (همان run/ps/build/compose). تفاوت‌های اصلی: rootless به‌طور پیش‌فرض (storage در home کاربر)، پشتیبانی از `--pod`، و نبود Swarm (K8s توصیه می‌شود). سازگاری با ابزارهای داکر از طریق `podman.socket` + `DOCKER_HOST` ممکن است. کلید: alias `docker=podman`، یادگیری صفر، و درک rootless storage.

---

## فصل ۴۸: Podman Play و مفهوم Pod

### مقدمه

تفاوت بنیادی Podman با داکر، مفهوم **Pod** است — گروهی از کانتینرها با شبکه و حجم مشترک (دقیقاً مثل Pod در Kubernetes). ابزار `podman play kube` حتی YAML کوبرنتیز را اجرا می‌کند. این فصل پل Podman → K8s را می‌سازد.

### مفاهیم اساسی

**Pod چیست؟**
یک واحد متشکل از یک یا چند کانتینر که:
- **شبکه مشترک** دارند (همه یک IP می‌بینند، مثل localhost).
- **حجم مشترک** (shared storage) دارند.
- با هم زنده/می‌میرند.

در داکر چنین چیزی ندارید (باید `docker-compose` با شبکهٔ مشترک شبیه‌سازی کنید).

**دستورات Pod:**

| دستور | کاربرد |
|---|---|
| `podman pod create` | ساخت pod خالی |
| `podman run --pod` | اضافه کردن کانتینر به pod |
| `podman pod ps` | لیست podها |
| `podman pod start/stop` | کنترل کل pod |
| `podman play kube` | اجرای YAML کوبرنتیز محلی |
| `podman generate kube` | تولید YAML از pod موجود |

**ساختار Pod:**

```
Pod (یک IP)
 ├── container: app  (main)
 ├── container: log-agent (sidecar)
 └── container: proxy
```

**Play Kube:**
`podman play kube file.yaml` فایل YAML کوبرنتیز (محدود به Pod/Deployment/Service) را روی ماشین محلی اجرا می‌کند — بدون کلاستر K8s. عالی برای تست قبل از استقرار واقعی.

### مثال‌های عملی

**۱. ساخت Pod و افزودن کانتینر:**

```bash
# ساخت pod با نام و پورت
podman pod create --name mypod -p 8080:80

# کانتینر اصلی
podman run -d --pod mypod --name web nginx:alpine

# sidecar (مثلاً لاگ‌بردار)
podman run -d --pod mypod --name logger fluentd:alpine

# مشاهده
podman pod ps
podman ps --pod
```

**۲. تولید YAML کوبرنتیز از Pod:**

```bash
podman generate kube mypod > mypod.yaml
# حالا mypod.yaml را می‌توان در K8s اجرا کرد (تقریباً)
```

**۳. اجرای YAML کوبرنتیز محلی:**

```bash
# فایل k8s-pod.yaml ساده
podman play kube k8s-pod.yaml
# کانتینرها طبق تعریف اجرا می‌شوند
```

**۴. Compose به Pod:**

```bash
# podman-compose در واقع پادها را می‌سازد
podman-compose up
podman pod ps   # هر سرویس در pod مشترک
```

### مقایسه و دیدگاه‌های مختلف

| مفهوم | Docker Compose | Podman Pod |
|---|---|---|
| واحد گروه‌بندی | Service | Pod |
| شبکه مشترک | network | pod network |
| مسیر به K8s | نیاز به تبدیل | `generate kube` نزدیک |
| Sidecar | غیرمستقیم | بومی |

**دیدگاه:** Podman Pod مدل ذهنی را به K8s نزدیک می‌کند. توسعه‌دهنده‌ای که روی Podman Pod کار می‌کند، انتقالش به K8s بی‌دردسر است. `generate kube`/`play kube` پل عملی بین local و cluster است.

**محدودیت:** `play kube` همهٔ ویژگی‌های K8s (مثل Ingress پیچیده) را پشتیبانی نمی‌کند — فقط Pod/Deployment/Service ساده.

### اشتباهات رایج

- **انتظار همهٔ K8s API:** `play kube` فقط زیرمجموعه کوچکی دارد.
- **پورت conflict در pod:** همه کانتینرها پورت داخلی را share می‌کنند — تکراری ندهید.
- **pod stopped اما container زنده:** وقتی pod را stop می‌کنید، همه کانتینرها می‌میرند (رفتار مورد انتظار).
- **فراموش `--pod` هنگام run:** کانتینر بیرون از pod می‌رود.
- **عدم درک sidecar pattern:** هنوز همه چیز را در یک کانتینر می‌ریزند.

### نکات مهم و ترفندها

- **Sidecar pattern:** در یک pod، کانتینر اصلی + کانتینر کمکی (مثل log shipper) همدیگر را در `localhost` می‌بینند.
- **`podman kube play --network`:** شبکهٔ سفارشی بدهید.
- **Systemd integration (Quadlet):** تعریف pod به صورت systemd unit برای بقای تولید.
- **`podman auto-update` با pod:** به‌روزرسانی خودکار بر اساس label.
- **Debug pod:** `podman pod inspect mypod` برای دیدن شبکه/IP.

### خلاصه فصل

Podman Pod گروهی از کانتینرها با شبکه/حجم مشترک است (مثل K8s Pod) — تفاوت بنیادی با داکر. `podman play kube` YAML کوبرنتیز ساده را محلی اجرا می‌کند و `generate kube` برعکس — پل واقعی به K8s. Sidecar pattern بومی پشتیبانی می‌شود. کلید: `--pod` هنگام run، درک اشتراک شبکه، و استفاده از play/generate kube برای انتقال نرم به کوبرنتیز.

---

## فصل ۴۹: Buildah و Skopeo (ساخت و جابه‌جایی Image)

### مقدمه

Podman خودش برای build از **Buildah** استفاده می‌کند، اما Buildah ابزاری مستقل و قدرتمند برای ساخت Image **بدون Dockerfile اجباری** و با کنترل ظریف است. **Skopeo** هم برای کپی/بررسی Image بین رجیستری‌ها بدون دانلود کامل است. این فصل این دو ابزار کلیدی اکوسیستم daemonless را بررسی می‌کند.

### مفاهیم اساسی

**Buildah چیست؟**
ابزار ساخت OCI Imageها. برخلاف `docker build` که همه چیز را در یک Dockerfile انجام می‌دهد، Buildah اجازه می‌دهد Image را **گام‌به‌گام در شل** بسازید (scriptable). مزایا:
- بدون daemon (rootless).
- Image نهایی بدون لایهٔ اضافه (تمیزتر).
- کنترل دقیق روی هر لایه.
- می‌تواند از Dockerfile هم بخواند (`buildah bud`).

**Skopeo چیست؟**
ابزار کپی و بازرسی Image. ویژگی‌های کلیدی:
- **Copy بدون دانلود کامل:** مستقیم از رجیستری A به B (registry-to-registry).
- **Inspect بدون pull:** متادیتا را بدون دانلود کل image ببینید.
- پشتیبانی فرمت‌های مختلف (oci، docker، containers-storage).

**تفاوت با داکر:**
- داکر build/push یکجا؛ Buildah/Skopeo تقسیم‌شده و ماژولار.
- Buildah برای تیم‌های امنیتی/پلتفرم جذاب‌تر (لایه‌های کمتر = سطح حمله کمتر).

**چرخه:**

```
Buildah (ساخت) → Skopeo (کپی/تأیید) → Registry
```

### مثال‌های عملی

**۱. ساخت با Buildah از Dockerfile:**

```bash
buildah bud -t myapp:v1 -f Dockerfile .
# یا با podman (که buildah را صدا می‌زند)
podman build -t myapp:v1 .
```

**۲. ساخت scriptable (بدون Dockerfile):**

```bash
#!/bin/bash
container=$(buildah from node:20-alpine)
buildah run $container -- npm ci --omit=dev
buildah copy $container ./dist /app/dist
buildah config --cmd '["node","/app/dist/server.js"]' $container
buildah config --user 1000 $container
buildah commit $container myapp:v1
# ذخیره در storage محلی یا push
buildah push myapp:v1 docker://registry/myapp:v1
```

**۳. Skopeo برای بازرسی (بدون pull):**

```bash
# دیدن لایه‌ها و متادیتا بدون دانلود کل image
skopeo inspect docker://docker.io/library/nginx:alpine
```

**۴. کپی بین رجیستری‌ها (بدون دانلود محلی):**

```bash
skopeo copy \
  docker://docker.io/library/nginx:alpine \
  docker://registry.internal/team/nginx:alpine
# یا فرمت oci-archive
skopeo copy docker://myapp:v1 oci-archive:myapp.tar
```

**۵. تبدیل فرمت:**

```bash
skopeo copy docker://myapp:v1 containers-storage:myapp:v1
```

### مقایسه و دیدگاه‌های مختلف

| ابزار | Docker معادل | تفاوت |
|---|---|---|
| `buildah bud` | `docker build` | daemonless، لایهٔ تمیزتر |
| `buildah from/run/commit` | (ندارد) | scriptable build |
| `skopeo copy` | `docker pull` + `docker push` | registry-to-registry مستقیم |
| `skopeo inspect` | `docker inspect` (بعد از pull) | بدون دانلود |

**دیدگاه:** Buildah برای خطوط لولهٔ امنیت‌محور (مثلاً تولید Image بدون shell، با فقط بایری درونی) ایده‌آل است. Skopeo در مهاجرت رجیستری و بازرسی امنیتی بی‌رقیب است. با هم، جایگزین کامل داکر CLI برای عملیات image هستند.

**CI/CD:** در محیط rootless CI، Buildah + Skopeo ترکیب استاندارد است (بسیار سبک‌تر از dind).

### اشتباهات رایج

- **فراموش `buildah commit`:** اگر commit نکنید، Image ذخیره نمی‌شود.
- **لایه‌های زیاد در scriptable:** هر `buildah run` لایه می‌سازد؛ با `--layers` یا ترکیب دستورات بهینه کنید.
- **Skopeo بدون احراز:** برای رجیستری خصوصی، `--creds` یا `--authfile` بدهید.
- **عدم توجه به فرمت:** `containers-storage` برای ذخیرهٔ محلی پادمن است نه `docker://`.
- **اجرای buildah به عنوان root در CI:** مزیت rootless را از دست می‌دهید.

### نکات مهم و ترفندها

- **`buildah --layers`:** استفاده از کش لایه (سرعت build).
- **Buildah در CI rootless:**
  ```yaml
  - name: Build
    run: buildah bud --layers -t myapp:${{ sha }} .
  ```
- **Skopeo sync:** همگام‌سازی کل repo بین رجیستری‌ها.
- **`buildah unshare`:** اجرای فرمان با نگاشت UID ریشه‌ای مجازی (برای عملیات نیازمند root درون کانتینر).
- **Manifest lists:** `buildah manifest` برای imageهای multi-arch (مثل arm64/amd64).
- **Sign با cosign روی خروجی Buildah:** زنجیره تأمین کامل.

### خلاصه فصل

Buildah ابزار ساخت OCI Image به صورت daemonless و scriptable است (با یا بدون Dockerfile) — لایه‌های تمیزتر و کنترل ظریف. Skopeo برای کپی registry-to-registry بدون دانلود کامل و بازرسی متادیتا استفاده می‌شود. با هم، جایگزین کامل داکر برای عملیات image هستند (مخصوصاً در CI rootless). کلید: `buildah commit` فراموش نشود، Skopeo برای مهاجرت/بازرسی، و استفاده از `--layers` برای سرعت.

---

## فصل ۵۰: یکپارچه‌سازی Podman در CI/CD و بهترین روش‌ها

### مقدمه

در محیط‌های CI/CD، داکر معمولاً با مشکل `privileged` در dind روبه‌روست. Podman با رویکرد **rootless** و **daemonless** جایگزین ایمن‌تری ارائه می‌دهد. در این فصل پیکربندی Podman در خط لوله، و چک‌لیست بهترین روش‌ها را می‌بینیم.

### مفاهیم اساسی

**چرا Podman در CI؟**
- **بدون privileged:** دیگر نیازی به `docker:dind --privileged` نیست (که ریسک امنیتی بزرگی در اجرای کد کاربر دارد).
- **سبک‌تر:** بدون daemon اضافه.
- **ایزولاسیون کاربر:** هر Job با UID خودش.

**چالش‌ها در CI:**
- rootless نیاز به تنظیم cgroups/user namespaces دارد (در runnerهای مدرن حل شده).
- برخی runnerها هنوز rootful می‌خواهند.

**الگوهای اجرا در CI:**
1. **Podman rootless روی runner:** مستقیم `podman build`.
2. **Podman در کانتینر (rootless):** با تصویر `podman` رسمی.
3. **Buildah در Kubernetes:** بدون داکر دایمون (Kaniko-like اما با Buildah).

** بهترین روش‌ها (CI/CD):**
- Image را با Buildah بسازید.
- با Skopeo به رجیستری پوش کنید (یا `podman push`).
- SBOM با Syft + امضا با cosign (فصل ۴۳).
- Image را با tag SHA تگ کنید (فصل ۴۲).

### مثال‌های عملی

**۱. GitLab CI با Podman (rootless runner):**

```yaml
build:
  image: quay.io/podman/stable
  script:
    - podman build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - podman push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  variables:
    STORAGE_DRIVER: vfs   # برای محیط‌های بدون overlay
```

**۲. GitHub Actions با Podman:**

```yaml
- name: Setup Podman
  run: |
    sudo apt install -y podman
- name: Build
  run: podman build -t myapp:${{ github.sha }} .
- name: Push
  run: |
    echo ${{ secrets.TOKEN }} | podman login -u user --password-stdin registry
    podman push myapp:${{ github.sha }}
```

**۳. Buildah در Kubernetes (Tekton مثال):**

```yaml
# TaskRun در Tekton مستقیماً Buildah را صدا می‌زند
steps:
  - name: build
    image: quay.io/buildah/stable
    script: buildah bud -t myapp .
```

**۴. ذخیرهٔ rootless در CI:**

```bash
export XDG_RUNTIME_DIR=/tmp/runtime-$UID
export TMPDIR=/tmp
podman --root /tmp/storage build -t myapp .
```

### مقایسه و دیدگاه‌های مختلف

| رویکرد CI | امنیت | پیچیدگی |
|---|---|---|
| docker:dind --privileged | کم | کم |
| Podman rootless | بالا | متوسط |
| Kaniko (داکر-محور) | بالا | کم |
| Buildah in K8s (Tekton) | بالا | متوسط |

**دیدگاه:** برای امنیت بالا در CI، Podman/Buildah rootless استاندارد طلایی است. dind مگر در محیط کاملاً قابل اعتماد اجتناب شود. Tekton (مبتنی بر K8s) به طور بومی از Buildah استفاده می‌کند.

**عملی:** اگر CI شما هنوز docker می‌خواهد، کافی است `DOCKER_HOST` را به podman socket بدهید (فصل ۴۷) — بدون تغییر اسکریپت.

### اشتباهات رایج

- **فراموش `STORAGE_DRIVER=vfs` در محیط محدود:** overlay در برخی runnerها کار نمی‌کند.
- **push بدون login:** Skopeo/podman نیاز به احراز دارند.
- **rootful در CI:** مزیت امنیتی را از بین می‌برد.
- **عدم پاکسازی:** imageهای ساخته‌شده فضای runner را پر می‌کنند.
- **اختلاف UID volume:** در rootless، نوشتن روی volume با permission mismatch.

### نکات مهم و ترفندها

- **`podman system prune -f`** در پایان CI برای آزادسازی فضا.
- **Cache در CI:** برای Buildah، لایه‌های قبلی را mount کنید (`--layers` + volume cache).
- **`podman login` با سرویس توکن:** از OIDC/IAM به جای رمز ثابت.
- **Multi-arch:** `buildah manifest` برای ساخت image برای arm64/amd64.
- **Tekton/Pipeline:** اکوسیستم K8s-native که Podman/Buildah را ترجیح می‌دهد.
- **Podman Quadlet در تولید:** برای اجرای long-running containers به عنوان systemd.

### خلاصه فصل

Podman/Buildah در CI/CD جایگزین ایمن‌تر dind هستند — بدون privileged، daemonless، و rootless. چالش اصلی تنظیم cgroups/user-ns در runner است (در runnerهای مدرن حل‌شده). الگو: Buildah برای build، Skopeo/podman push برای انتقال، با SBOM+cosign برای زنجیرهٔ تأمین. کلید: `STORAGE_DRIVER=vfs` در محیط محدود، پاکسازی منظم، و استفاده از OIDC برای login. ابزارهای قدیمی با `DOCKER_HOST`→podman socket سازگار می‌شوند.

---

## منابع و مراجع

- Podman Documentation: https://docs.podman.io
- Buildah: https://buildah.io
- Skopeo: https://github.com/containers/skopeo
- Podman Tutorials: https://podman.io/getting-started
- Podman vs Docker (Red Hat): https://www.redhat.com/en/topics/containers/podman-vs-docker
- Rootless Containers: https://rootlesscontaine.rs
- Quadlet: https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html
- Tekton (Buildah-native CI): https://tekton.dev

**بعدی:** [فایل ۱۱ — مبانی کوبرنتیز (Kubernetes)](11-Kubernetes-Fundamentals.md)
