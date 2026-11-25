# فایل ۰۲ — لینوکس، شل و شبکه برای DevOps

**مدت یادگیری**: ۸ ساعت
**سطح سختی**: مبتدی–متوسط
**پیش‌نیاز**: فایل ۰۱، آشنایی مقدماتی با خط فرمان

---

## فهرست محتوا

- [فصل ۶: لینوکس برای DevOps](#فصل-۶-لینوکس-برای-devops)
- [فصل ۷: اسکریپت‌نویسی شل و خودکارسازی](#فصل-۷-اسکریپتنویسی-شل-و-خودکارسازی)
- [فصل ۸: شبکه برای DevOps](#فصل-۸-شبکه-برای-devops)
- [فصل ۹: امنیت پایه و مدیریت دسترسی](#فصل-۹-امنیت-پایه-و-مدیریت-دسترسی)
- [فصل ۱۰: عیب‌یابی سیستمی و ابزارهای CLI](#فصل-۱۰-عیبیابی-سیستمی-و-ابزارهای-cli)

---

## فصل ۶: لینوکس برای DevOps

### مقدمه

لینوکس، ستون فقرات DevOps است. تقریباً همهٔ سرورها، کانتینرها و ابزارهای ابری روی لینوکس اجرا می‌شوند. تصور کنید می‌خواهید یک سرور را پیکربندی کنید، یک کانتینر بسازید یا یک pipeline بنویسید — همه‌جا لینوکس حاضر است.

در این فصل مباحث لینوکسی که یک مهندس DevOps بیشتر به آن نیاز دارد را مرور می‌کنیم: مدیریت فرآیند، سرویس‌ها، پکیج، مجوز و ابزارهای کلیدی.

### مفاهیم اساسی

**توزیع‌های مهم برای DevOps:**

| توزیع | کاربرد |
|---|---|
| **Ubuntu/Debian** | رایج‌ترین برای ابر و CI |
| **RHEL/Rocky/Alma** | سازمانی |
| **Alpine** | سبک، برای کانتینر |
| **Amazon Linux** | AWS |
| **Distroless** | حداقلی، امن |

**مدیریت فرآیند:**

| دستور | کار |
|---|---|
| `ps aux` | فهرست فرآیندها |
| `top`/`htop` | پایش زنده |
| `kill` | ارسال سیگنال |
| `systemctl` | مدیریت سرویس |
| `journalctl` | لاگ سرویس |

**مدیریت سرویس با systemd:**

| دستور | کار |
|---|---|
| `systemctl start X` | شروع |
| `systemctl enable X` | فعال در بوت |
| `systemctl status X` | وضعیت |
| `systemctl reload X` | بازخوانی |

**مدیریت پکیج:**

```bash
# Debian/Ubuntu
apt update && apt install -y nginx

# RHEL/Rocky
dnf install -y nginx

# Alpine (برای کانتینر)
apk add --no-cache nginx
```

**ساختار فایل‌سیستم مهم:**

| مسیر | محتوا |
|---|---|
| `/etc` | پیکربندی |
| `/var/log` | لاگ |
| `/var/lib` | داده |
| `/usr/bin` | دستورها |
| `/opt` | نرم‌افزار اختیاری |
| `/proc`, `/sys` | مجازی |

### مثال‌های عملی

**مثال ۱: اسکریپت بررسی سرویس**

```bash
#!/bin/bash
# check-service.sh — بررسی وضعیت سرویس
SERVICE="nginx"

if systemctl is-active --quiet "$SERVICE"; then
  echo "✓ $SERVICE در حال اجراست"
else
  echo "✗ $SERVICE متوقف است — تلاش برای شروع..."
  sudo systemctl start "$SERVICE"
fi
```

**مثال ۲: یک سرویس systemd سفارشی**

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/start
Restart=on-failure
RestartSec=5
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
systemctl status myapp
```

**مثال ۳: مدیریت مجوز در کانتینر**

```bash
# در Dockerfile، کاربر غیر-ریشه بسازید
RUN adduser --disabled-password --gecos "" appuser
USER appuser
```

### مقایسه و الگوهای مختلف

**systemd در برابر init سنتی:**

| ویژگی | systemd | SysV init |
|---|---|---|
| راه‌اندازی | موازی | سری |
| وابستگی | دارد | ندارد |
| لاگ | journald | فایل |
| وضعیت | مدرن | منسوخ |

**توزیع‌های سبک برای کانتینر:**

| توزیع | حجم | کاربرد |
|---|---|---|
| Alpine | ~5 MB | کانتینر عمومی |
| Distroless | ~2 MB | امنیت |
| Scratch | 0 | static binary |
| Ubuntu | ~70 MB | عمومی |

**مدیران پکیج:**

| توزیع | مدیر | فرمت |
|---|---|---|
| Debian | apt/dpkg | .deb |
| RHEL | dnf/rpm | .rpm |
| Alpine | apk | .apk |
| Arch | pacman | .pkg.tar.zst |

### اشکال شایع

**اشکال ۱: استفاده از root در کانتینر.**
خطر امنیتی. راه‌حل: USER غیر-ریشه.

**اشکال ۲: تصویر پایهٔ بزرگ.**
تصویر سنگین و کند. راه‌حل: Alpine/distroless.

**اشکال ۳: فراموش کردن `daemon-reload`.**
سرویس اعمال نمی‌شود. راه‌حل: daemon-reload.

**اشکال ۴: نبود logrotate.**
دیسک پر می‌شود. راه‌حل: logrotate.

**اشکال ۵: نادیده گرفتن محدودیت‌های هسته.**
`ulimit` و `sysctl`. راه‌حل: تنظیم.

**اشکال ۶: اشتباه در مسیرها.**
مسیر نسبی در سرویس. راه‌حل: مسیر مطلق.

### نکات مهم (Tips & Tricks)

- **Alpine را برای کانتینر یاد بگیرید.**
- **`systemctl` را عمیق یاد بگیرید.**
- **از کاربر غیر-ریشه استفاده کنید.**
- **`journalctl -u service -f` برای دیباگ.**
- **`ss -tlnp` پورت‌های باز.**
- **`strace`/`ltrace` برای دیباگ عمیق.**
- **`lsof` فایل‌های باز.**
- **`dmesg` پیام‌های کرنل.**
- **`/etc/os-release` نسخهٔ توزیع.**
- **`uname -a` اطلاعات کرنل.**

### خلاصه فصل

لینوکس ستون DevOps است. مدیریت فرآیند (ps, kill)، سرویس (systemctl)، پکیج (apt/dnf/apk) و مجوز پایه‌اند. توزیع‌های سبک (Alpine، distroless) برای کانتینر مهم‌اند. سرویس‌های systemd سفارشی و کاربر غیر-ریشه امنیت و مدیریت را بهتر می‌کنند. ابزارهای دیباگ (journalctl, strace, lsof) ضروری‌اند.

---

## فصل ۷: اسکریپت‌نویسی شل و خودکارسازی

### مقدمه

خودکارسازی، قلب DevOps است. تصور کنید باید روزانه ۱۰ سرور را پیکربندی کنید — دستی غیرممکن است. اسکریپت شل، ساده‌ترین و قدرتمندترین ابزار خودکارسازی است که همه‌جا در دسترس است.

در این فصل اسکریپت‌نویسی حرفه‌ای برای DevOps را یاد می‌گیرید.

### مفاهیم اساسی

**ساختار اسکریپت:**

```bash
#!/usr/bin/env bash
set -euo pipefail   # ایمنی

# متغیرها
readonly APP_NAME="myapp"

# توابع
main() {
  echo "شروع $APP_NAME"
}

main "$@"
```

**`set -euo pipefail`:**

| گزینه | اثر |
|---|---|
| `-e` | توقف در خطا |
| `-u` | خطا در متغیر تعریف‌نشده |
| `-o pipefail` | خطای لوله |

**پردازش متن (حیاتی برای DevOps):**

```bash
grep, sed, awk, cut, sort, uniq, jq, yq
```

**ابزارهای مهم DevOps:**

| ابزار | کار |
|---|---|
| `jq` | پردازش JSON |
| `yq` | پردازش YAML |
| `curl`/`wget` | HTTP |
| `xargs` | پردازش آرگومان |
| `parallel` | اجرای موازی |

### مثال‌های عملی

**مثال ۱: اسکریپت deploy**

```bash
#!/usr/bin/env bash
set -euo pipefail

APP="myapp"
VERSION="${1:?نسخه لازم است}"
REGISTRY="registry.example.com"

log() { echo "[$(date '+%H:%M:%S')] $*"; }

log "ساخت image $APP:$VERSION"
docker build -t "$REGISTRY/$APP:$VERSION" .

log "push به registry"
docker push "$REGISTRY/$APP:$VERSION"

log "استقرار روی Kubernetes"
kubectl set image deployment/"$APP" \
  "$APP=$REGISTRY/$APP:$VERSION"

log "انتظار برای rollout"
kubectl rollout status deployment/"$APP"

log "✓ deploy موفق"
```

**مثال ۲: پردازش JSON با jq**

```bash
# استخراج نام سرویس‌ها از خروجی API
curl -s https://api.example.com/services | jq -r '.[].name'

# فیلتر
curl -s https://api.example.com/services | \
  jq '.[] | select(.status == "unhealthy") | .name'

# ساختاردهی مجدد
cat data.json | jq '{total: length, names: [.[].name]}'
```

**مثال ۳: پردازش موازی**

```bash
#!/usr/bin/env bash
# اجرای موازی روی چند سرور
SERVERS=(server1 server2 server3 server4)

check_server() {
  local host="$1"
  if ssh -o ConnectTimeout=5 "$host" "uptime" &>/dev/null; then
    echo "✓ $host"
  else
    echo "✗ $host"
  fi
}
export -f check_server

printf '%s\n' "${SERVERS[@]}" | \
  xargs -P 4 -I {} bash -c 'check_server "$@"' _ {}
```

### مقایسه و الگوهای مختلف

**Bash در برابر Python برای خودکارسازی:**

| جنبه | Bash | Python |
|---|---|---|
| سادگی | برای کارهای ساده | ساختارمند |
| پردازش متن | عالی | خوب |
| منطق پیچیده | سخت | آسان |
| وابستگی | ندارد | نیاز به Python |
| مناسب | pipeline، CLI | منطق پیچیده |

**توصیه:** Bash برای گلوی کارها، Python برای منطق پیچیده.

**ابزارهای پردازش ساختاریافته:**

| فرمت | ابزار |
|---|---|
| JSON | `jq` |
| YAML | `yq` |
| TOML | `tomlq` |
| CSV | `csvkit`, `awk` |

### اشکال شایع

**اشکال ۱: نبود `set -e`.**
خطاها نادیده می‌مانند. راه‌حل: `set -euo pipefail`.

**اشکال ۲: متغیر بدون کوتیشن.**
فاصله‌ها خطا می‌سازند. راه‌حل: `"$var"`.

**اشکال ۳: مسیر نسبی.**
در cron/pipeline خطا می‌دهد. راه‌حل: مسیر مطلق.

**اشکال ۴: نبود لاگ.**
دیباگ سخت. راه‌حل: تابع log.

**اشکال ۵: parse کردن JSON با grep/sed.**
شکننده. راه‌حل: `jq`.

**اشکال ۶: نبود بررسی خطا.**
اسکریپت ادامه می‌دهد. راه‌حل: بررسی `$?`.

### نکات مهم (Tips & Tricks)

- **همیشه `set -euo pipefail`.**
- **`jq` و `yq` را یاد بگیرید.**
- **`shellcheck` برای بررسی.**
- **توابع برای تکرارنشدن.**
- **`trap` برای پاک‌سازی.**
- **`mktemp` برای فایل موقت.**
- **مسیر مطلق.**
- **لاگ با timestamp.**
- **متغیرها در کوتیشن.**
- **`getopts` برای آرگومان.**
- **`xargs -P` برای موازی.**
- **اسکریپت‌ها را در Git نگه دارید.**

### خلاصه فصل

اسکریپت شل ساده‌ترین ابزار خودکارسازی است. `set -euo pipefail` ایمنی می‌دهد. پردازش متن (grep/sed/awk) و ساختاریافته (jq/yq) حیاتی است. Bash برای گلوی کارها و Python برای منطق پیچیده. همیشه کوتیشن، مسیر مطلق، لاگ و بررسی خطا. `shellcheck` کیفیت را بالا می‌برد.

---

## فصل ۸: شبکه برای DevOps

### مقدمه

شبکه، پایهٔ ارتباط سرویس‌هاست. تصور کنید سرویس‌ها همدیگر را پیدا نمی‌کنند یا TLS خطا می‌دهد — بدون فهم شبکه، عیب‌یابی غیرممکن است.

در این فصل مباحث شبکه‌ای که یک مهندس DevOps به آن نیاز دارد را یاد می‌گیرید.

### مفاهیم اساسی

**مدل TCP/IP و پورت‌ها:**

| لایه | پروتکل‌ها |
|---|---|
| Application | HTTP، DNS، SSH |
| Transport | TCP، UDP |
| Internet | IP، ICMP |
| Link | Ethernet |

**پورت‌های مهم:**

| سرویس | پورت |
|---|---|
| HTTP | 80 |
| HTTPS | 443 |
| SSH | 22 |
| DNS | 53 |
| PostgreSQL | 5432 |
| Redis | 6379 |
| MySQL | 3306 |

**DNS در Kubernetes:**

```
service.namespace.svc.cluster.local
myapp.default.svc.cluster.local
```

**Load Balancing:**

| نوع | لایه | ابزار |
|---|---|---|
| L4 | TCP/UDP | HAProxy، NLB |
| L7 | HTTP | Nginx، ALB، Ingress |

**TLS/SSL:**

- رمزنگاری ارتباط.
- گواهی (certificate).
- Let's Encrypt، ACME.
- Termination در LB/Ingress.

### مثال‌های عملی

**مثال ۱: عیب‌یابی اتصال**

```bash
# ۱. تست دسترسی
ping -c 3 8.8.8.8

# ۲. تست DNS
dig +short example.com

# ۳. تست پورت
nc -zv host 443

# ۴. تست HTTP
curl -v https://example.com

# ۵. بررسی گواهی
openssl s_client -connect example.com:443

# ۶. ردیابی مسیر
mtr example.com
```

**مثال ۲: تنظیم Nginx به‌عنوان reverse proxy**

```nginx
server {
    listen 443 ssl;
    server_name app.example.com;

    ssl_certificate /etc/ssl/certs/app.crt;
    ssl_certificate_key /etc/ssl/private/app.key;

    location / {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**مثال ۳: Service در Kubernetes**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

### مقایسه و الگوهای مختلف

**L4 در برابر L7 Load Balancing:**

| جنبه | L4 | L7 |
|---|---|---|
| لایه | TCP/UDP | HTTP |
| آگاهی | ندارد | دارد (URL، header) |
| سرعت | سریع‌تر | کمی کندتر |
| قابلیت | محدود | routing پیشرفته |

**روش‌های TLS Termination:**

| روش | توضیح |
|---|---|
| Edge | در LB/Ingress |
| Passthrough | مستقیم به backend |
| Re-encryption | دو بار |

**Service Types در Kubernetes:**

| نوع | کاربرد |
|---|---|
| ClusterIP | داخلی |
| NodePort | پورت روی نود |
| LoadBalancer | ابر |
| ExternalName | DNS |

### اشکال شایع

**اشکال ۱: فراموش کردن X-Forwarded-* هدرها.**
Backend IP واقعی را نمی‌بیند. راه‌حل: proxy_set_header.

**اشکال ۲: گواهی منقضی.**
سایت خطا می‌دهد. راه‌حل: تمدید خودکار.

**اشکال ۳: DNS اشتباه در cluster.**
سرویس پیدا نمی‌شود. راه‌حل: نام کامل.

**اشکال ۴: فایروال پورت.**
اتصال رد می‌شود. راه‌حل: باز کردن.

**اشکال ۵: MTU اشتباه در overlay.**
بسته‌های بزرگ drop. راه‌حل: تنظیم MTU.

**اشکال ۶: نبود health check.**
ترافیک به پاد خراب. راه‌حل: readiness probe.

### نکات مهم (Tips & Tricks)

- **`curl -v` برای دیباگ HTTP.**
- **`dig`/`nslookup` برای DNS.**
- **`nc -zv` برای تست پورت.**
- **`openssl s_client` برای TLS.**
- **`mtr` برای مسیر.**
- **`tcpdump` برای تحلیل بسته.**
- **X-Forwarded-* را در proxy بگذارید.**
- **health check را تنظیم کنید.**
- **DNS cluster را یاد بگیرید.**
- **گواهی را خودکار تمدید کنید.**

### خلاصه فصل

شبکه پایهٔ ارتباط سرویس‌هاست. مدل TCP/IP، پورت‌ها، DNS، Load Balancing (L4/L7) و TLS مفاهیم کلیدی‌اند. ابزارهای عیب‌یابی (curl, dig, nc, openssl, mtr, tcpdump) ضروری‌اند. در Kubernetes، Service و DNS cluster مهم‌اند. X-Forwarded-* و health check را فراموش نکنید.

---

## فصل ۹: امنیت پایه و مدیریت دسترسی

### مقدمه

امنیت در DevOps از روز اول مهم است (Shift Left). تصور کنید یک سرور بدون رمز قوی و بدون فایروال راه‌اندازی کنید — یک درِ باز برای مهاجم.

در این فصل مبانی امنیتی که یک مهندس DevOps باید بداند را یاد می‌گیرید.

### مفاهیم اساسی

**اصول امنیتی:**

- **Least Privilege:** حداقل دسترسی.
- **Defense in Depth:** لایه‌های دفاع.
- **Shift Left:** امنیت از ابتدا.
- **Zero Trust:** به هیچ‌چیز اعتماد نکن.

**مدیریت دسترسی:**

| مفهوم | توضیح |
|---|---|
| کاربر/گروه | UID/GID |
| sudo | اجرای ریشه |
| SSH keys | احراز کلید |
| RBAC | دسترسی نقش‌محور |

**مدیریت Secrets:**

- هرگز secret در کد/تصویر نباشد.
- ابزارها: Vault، SOPS، Sealed Secrets.
- متغیرهای محیطی، secret manager.

**فایروال:**

```bash
# ufw (ساده)
ufw allow 22/tcp
ufw allow 443/tcp
ufw enable

# firewalld (RHEL)
firewall-cmd --add-service=https --permanent
firewall-cmd --reload
```

### مثال‌های عملی

**مثال ۱: سخت‌سازی SSH**

```bash
# /etc/ssh/sshd_config
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
ClientAliveInterval 300
```

**مثال ۲: مدیریت secret در CI**

```yaml
# GitHub Actions — secret امن
jobs:
  deploy:
    steps:
      - name: Deploy
        env:
          API_KEY: ${{ secrets.API_KEY }}   # از Secrets
        run: ./deploy.sh
```

**مثال ۳: اسکن تصویر کانتینر**

```bash
# Trivy — اسکن آسیب‌پذیری
trivy image myapp:latest

# فقط آسیب‌پذیری‌های بحرانی
trivy image --severity CRITICAL,HIGH myapp:latest

# در pipeline
- name: Scan
  run: trivy image --exit-code 1 --severity CRITICAL myapp:${{ github.sha }}
```

### مقایسه و الگوهای مختلف

**روش‌های احراز:**

| روش | امنیت |
|---|---|
| رمز عبور | پایین |
| کلید SSH | بالا |
| MFA | بسیار بالا |
| Certificate | بسیار بالا |

**ابزارهای Secrets:**

| ابزار | ویژگی |
|---|---|
| HashiCorp Vault | جامع، پویا |
| AWS Secrets Manager | ابری |
| SOPS | رمزنگاری فایل |
| Sealed Secrets | برای K8s |

**فایروال‌ها:**

| ابزار | سطح |
|---|---|
| ufw | ساده |
| firewalld | متوسط |
| iptables/nftables | پایین |

### اشکال شایع

**اشکال ۱: secret در کد.**
در Git تاریخچه می‌ماند. راه‌حل: secret manager.

**اشکال ۲: root SSH فعال.**
خطر brute-force. راه‌حل: غیرفعال.

**اشکال ۳: نبود فایروال.**
پورت‌های باز اضافی. راه‌حل: فایروال.

**اشکال ۴: مجوز بیش از حد.**
`chmod 777`. راه‌حل: حداقل.

**اشکال ۵: نبود اسکن تصویر.**
آسیب‌پذیری منتشر می‌شود. راه‌حل: Trivy.

**اشکال ۶: گواهی self-signed در production.**
خطای امنیتی. راه‌حل: CA معتبر.

### نکات مهم (Tips & Tricks)

- **کلید SSH به‌جای رمز.**
- **secret در manager، نه کد.**
- **فایروال minimal.**
- **اسکن تصویر در pipeline.**
- **کمترین دسترسی.**
- **`fail2ban` برای SSH.**
- **گواهی خودکار.**
- **`gitleaks` برای پیدا کردن secret.**
- **به‌روزرسانی منظم.**
- **لاگ امنیتی.**

### خلاصه فصل

امنیت از ابتدا (Shift Left). اصول: Least Privilege، Defense in Depth، Zero Trust. مدیریت دسترسی با sudo/RBAC/SSH keys. Secrets در manager. فایروال minimal. اسکن تصویر (Trivy) و گواهی معتبر. secret هرگز در کد نباشد.

---

## فصل ۱۰: عیب‌یابی سیستمی و ابزارهای CLI

### مقدمه

عیب‌یابی مهارت کلیدی DevOps است. تصور کنید سروری کند شده یا کانتینری کرش می‌کند — باید سریع ریشه را پیدا کنید.

در این فصل رویکرد و ابزارهای عیب‌یابی را یاد می‌گیرید.

### مفاهیم اساسی

**رویکرد لایه‌ای:**

```
۱. منابع (CPU، RAM، دیسک)
۲. فرآیند
۳. شبکه
۴. اپلیکیشن
۵. لاگ
```

**ابزارهای کلیدی:**

| ابزار | کار |
|---|---|
| `top`/`htop` | منابع |
| `df`/`du` | دیسک |
| `free` | حافظه |
| `ss`/`netstat` | شبکه |
| `journalctl` | لاگ |
| `strace` | syscall |
| `lsof` | فایل باز |

**مدل USE:**

- **U**tilization: چقدر استفاده می‌شود.
- **S**aturation: چقدر صف دارد.
- **E**rrors: خطا دارد؟

### مثال‌های عملی

**مثال ۱: سرور کند**

```bash
# ۱. بار کلی
uptime                    # load average

# ۲. منابع
top -bn1 | head -20
free -h
df -h

# ۳. I/O
iostat -x 1 3

# ۴. شبکه
ss -s

# ۵. فرآیندهای پرمصرف
ps aux --sort=-%cpu | head
```

**مثال ۲: کانتینر کرش می‌کند**

```bash
# لاگ کانتینر
docker logs mycontainer

# وضعیت
docker inspect mycontainer | jq '.[0].State'

# در Kubernetes
kubectl logs pod-name
kubectl describe pod pod-name
kubectl get events --sort-by=.lastTimestamp

# اجرای shell در پاد
kubectl exec -it pod-name -- sh
```

**مثال ۳: مشکل شبکه**

```bash
# DNS
dig +short service-name

# اتصال
curl -v http://service:8080/health

# پورت
nc -zv service 8080

# بسته
tcpdump -i any port 8080 -c 10
```

### مقایسه و الگوهای مختلف

**رویکردها:**

| رویکرد | تمرکز |
|---|---|
| **USE** | منابع |
| **RED** | سرویس (Rate، Errors، Duration) |
| **Four Golden Signals** | Latency، Traffic، Errors، Saturation |

**ابزارهای عیب‌یابی:**

| لایه | ابزار |
|---|---|
| سیستم | top، iostat، vmstat |
| شبکه | ss، tcpdump، mtr |
| اپلیکیشن | لاگ، APM |
| کانتینر | docker/kubectl |

### اشکال شایع

**اشکال ۱: عیب‌یابی کورکورانه.**
بدون رویکرد. راه‌حل: لایه‌ای.

**اشکال ۲: نادیده گرفتن لاگ.**
لاگ منبع اصلی. راه‌حل: اول لاگ.

**اشکال ۳: تغییر همزمان چند چیز.**
نمی‌فهمید کدام حل کرد. راه‌حل: یکی‌یکی.

**اشکال ۴: نبود baseline.**
نمی‌دانید «عادی» چیست. راه‌حل: baseline.

**اشکال ۵: فراموش کردن دیسک/inode.**
`df -i`. راه‌حل: بررسی.

**اشکال ۶: نادیده گرفتن DNS.**
اغلب مشکل DNS است. راه‌حل: dig.

### نکات مهم (Tips & Tricks)

- **اول لاگ، بعد حدس.**
- **رویکرد لایه‌ای.**
- **`kubectl describe` برای events.**
- **`docker logs` برای کانتینر.**
- **`strace` برای syscall.**
- **`lsof` برای فایل باز.**
- **`ss -tlnp` پورت‌ها.**
- **`mtr` برای شبکه.**
- **`df -i` for inode.**
- **یکی‌یکی تغییر دهید.**
- **baseline بگیرید.**
- **`dmesg` برای کرنل.**

### خلاصه فصل

عیب‌یابی رویکرد لایه‌ای می‌خواهد: منابع → فرآیند → شبکه → اپلیکیشن → لاگ. ابزارهای کلیدی: top، df، ss، journalctl، strace، lsof. مدل‌های USE و RED و Four Golden Signals راهنماست. اول لاگ، بعد حدس. یکی‌یکی تغییر دهید و baseline داشته باشید.

---

## منابع و مراجع

- Linux Command Line (William Shotts)
- The Linux Programming Interface (Michael Kerrisk)
- Bash Reference Manual: https://www.gnu.org/software/bash/manual/
- ShellCheck: https://www.shellcheck.net
- jq Manual: https://jqlang.github.io/jq/manual/
- Trivy: https://trivy.dev
- HashiCorp Vault: https://www.vaultproject.io

**بعدی:** [فایل ۰۳ — Git و استراتژی‌های Branching](03-Git-and-Branching-Strategies.md)
