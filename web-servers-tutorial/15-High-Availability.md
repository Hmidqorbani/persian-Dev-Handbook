# بخش ۱۵: دسترس‌پذیری بالا (High Availability)

**مدت یادگیری**: ۵ تا ۶ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: بخش ۰۹ (Load Balancing)، بخش ۱۰ (TLS)

---

## فهرست محتوا

- [فصل ۶۴: HA برای وب‌سرور — active-passive و active-active](#فصل-۶۴)
- [فصل ۶۵: Keepalived، VIP و health check](#فصل-۶۵)
- [فصل ۶۶: Kubernetes Ingress و Ingress Controller](#فصل-۶۶)
- [فصل ۶۷: CDN و لبه — کش، WAF و DDoS](#فصل-۶۷)

---

## فصل ۶۴: HA برای وب‌سرور — active-passive و active-active

### مقدمه

یک وب‌سرور، یک نقطه‌ی شکست است. دو وب‌سرور، اگر درست طراحی شوند، «بدون قطعی» می‌دهند. ولی «دو سرور» به‌تنهایی کافی نیست: باید بدانی چه چیزی مشترک است، چه چیزی نیست، و خرابی چطور تشخیص داده می‌شود.

### مفاهیم اساسی

**۱) سه مدل**

```
Active-Passive:
کلاینت → VIP → [active] 
                [passive] (آماده)

Active-Active:
کلاینت → LB → [web1] [web2]

Active-Active با DNS:
کلاینت → DNS (چند A) → [web1] [web2]
```

| مدل | استفاده از منابع | پیچیدگی | RTO |
|---|---|---|---|
| Active-Passive | ⚠️ نصف | ✅ | ⚠️ (failover) |
| Active-Active (LB) | ✅ کامل | ⚠️ | ✅ صفر |
| DNS round-robin | ✅ | ✅ | ⚠️ (کش DNS) |

**۲) چه چیزی باید مشترک باشد**

| جزء | راه‌حل |
|---|---|
| محتوای استاتیک | فایل مشترک (NFS/object storage) یا deploy یکسان |
| session | Redis/DB |
| آپلود | object storage |
| TLS | گواهی یکسان روی هر سرور |
| config | IaC (Ansible/Terraform) |
| لاگ | متمرکز |

**۳) چه چیزی نباید مشترک باشد**

- `PGDATA` (اگر DB روی همان سرور است، جدا)
- `node_modules`/`vendor` (deploy جدا، ولی یکسان)

**۴) `health check` درست**

```
/health → 200 اگر:
  - وب‌سرور پاسخ می‌دهد
  - اپ متصل است
  - DB در دسترس است
  - (اختیاری) cache در دسترس است
```

**۵) `failover` و `failback`**

- **failover**: انتقال به سرور دوم.
- **failback**: بازگشت به سرور اول (خودکار یا دستی).

`failback` خودکار می‌تواند «flapping» بسازد: سرور ناپایدار، مدام جابه‌جا.

**۶) `quorum` و `split-brain`**

با دو سرور، اگر ارتباط قطع شود، هر دو فکر می‌کنند دیگری مرده. راه‌حل: `witness`/`quorum disk` یا سه گره.

**۷) `VIP` (Virtual IP)**

```
کلاینت → 192.168.56.100 (VIP)
              │
       ┌──────┴──────┐
       ▼             ▼
   [web1]        [web2]
   (master)      (backup)
```

`VIP` با `Keepalived`/`VRRP` جابه‌جا می‌شود.

**۸) `DNS failover`**

`TTL` کوتاه (۶۰s) + health check در DNS. عیب: کش DNS و تأخیر.

**۹) `Anycast`**

همان IP روی چند نقطه. مسیریابی با BGP. برای CDN و DNS رایج.

**۱۰) اندازه‌گیری `RTO`**

```
RTO = تشخیص + تصمیم + جابه‌جایی + گرم شدن
```

`health check` سریع‌تر = `RTO` کمتر، ولی ریسک false positive بیشتر.

### مثال‌های عملی

**Lab Scenario: Active-Passive با Keepalived**

```
web1: 192.168.56.11 (MASTER, priority 100)
web2: 192.168.56.12 (BACKUP, priority 90)
VIP:  192.168.56.100
```

```bash
# روی هر دو سرور
sudo apt install -y keepalived nginx
```

```nginx
# /etc/nginx/sites-available/lab
server {
    listen 80;
    server_name lab.example.com;
    root /var/www/lab;

    location = /health {
        access_log off;
        default_type text/plain;
        return 200 "ok\n";
    }

    location / {
        try_files $uri $uri/ =404;
    }
}
```

```bash
sudo ln -sf /etc/nginx/sites-available/lab /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t && sudo systemctl reload nginx
```

```conf
# /etc/keepalived/keepalived.conf — روی web1
global_defs {
    router_id web1
    enable_script_security
    script_user root
}

vrrp_script chk_nginx {
    script "/usr/bin/curl -fsS -m 2 http://127.0.0.1/health || exit 1"
    interval 2
    fall 3
    rise 2
    timeout 3
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass S3cret
    }

    virtual_ipaddress {
        192.168.56.100/24
    }

    track_script {
        chk_nginx
    }
}
```

```conf
# /etc/keepalived/keepalived.conf — روی web2
global_defs {
    router_id web2
    enable_script_security
    script_user root
}

vrrp_script chk_nginx {
    script "/usr/bin/curl -fsS -m 2 http://127.0.0.1/health || exit 1"
    interval 2
    fall 3
    rise 2
    timeout 3
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 90
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass S3cret
    }

    virtual_ipaddress {
        192.168.56.100/24
    }

    track_script {
        chk_nginx
    }
}
```

```bash
# روی هر دو
sudo systemctl enable --now keepalived
sudo systemctl status keepalived --no-pager

# بررسی VIP
ip addr show eth0 | grep 192.168.56.100

# تست
curl -sS http://192.168.56.100/health
```

**تست failover:**

```bash
# روی web1
sudo systemctl stop nginx

# بعد از ~۶ ثانیه (interval×fall)
ip addr show eth0 | grep 192.168.56.100
# VIP رفته

# روی web2
ip addr show eth0 | grep 192.168.56.100
# VIP آمده

curl -sS http://192.168.56.100/health
# ok
```

**`notify` scripts:**

```conf
vrrp_instance VI_1 {
    # ...
    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault  "/etc/keepalived/notify.sh fault"
}
```

```bash
#!/usr/bin/env bash
# /etc/keepalived/notify.sh
set -euo pipefail
STATE=$1
echo "$(date -Is) $STATE" >> /var/log/keepalived-state.log
case "$STATE" in
  master) systemctl start nginx ;;
  backup) : ;;
  fault)  : ;;
esac
```

**Active-Active با DNS:**

```
lab.example.com. 60 IN A 192.168.56.11
lab.example.com. 60 IN A 192.168.56.12
```

```bash
dig +short lab.example.com
# 192.168.56.11
# 192.168.56.12

for i in 1 2 3 4; do curl -sS http://lab.example.com/ -o /dev/null -w '%{remote_ip}\n'; done
```

**Active-Active با LB:**

```
کلاینت → HAProxy (VIP) → web1, web2
```

(فصل ۳۸)

### مقایسه و الگوهای مختلف

**Active-Passive در برابر Active-Active**

| | A-P | A-A |
|---|---|---|
| منابع | ⚠️ | ✅ |
| RTO | ⚠️ | ✅ |
| پیچیدگی | ✅ | ⚠️ |
| session | ⚠️ | ❌ لازم مشترک |
| مناسب | DB، اپ stateful | ✅ وب |

**`Keepalived` در برابر `Pacemaker` در برابر DNS**

| | Keepalived | Pacemaker | DNS |
|---|---|---|---|
| سادگی | ✅ | ❌ | ✅ |
| VIP | ✅ | ✅ | ❌ |
| quorum | ⚠️ | ✅ | ❌ |
| مناسب | ✅ دو سرور | چند سرویس | geo |

**`health check` سریع در برابر کند**

| | ۲s | ۱۰s |
|---|---|---|
| RTO | ✅ | ❌ |
| false positive | ⚠️ | ✅ |

**دیدگاه «دو سرور = HA»**: بدون VIP/LB، کلاینت باید IP را عوض کند. بدون session مشترک، کاربر logout می‌شود.

### اشکال شایع

**۱) `split-brain` با دو سرور**
هر دو MASTER. `priority` و `advert_int`.

**۲) `health check` فقط TCP**
اپ خراب، سرور سالم. `/health` واقعی.

**۳) session در حافظه**
failover، logout. Redis.

**۴) محتوای متفاوت**
web1 نسخه‌ی جدید، web2 قدیم. deploy یکسان.

**۵) `VIP` بدون `arp_ignore`**
ARP conflict. تنظیم sysctl.

**۶) `failback` خودکار**
flapping. `nopreempt`.

**۷) گواهی متفاوت**
خطای TLS بعد از failover. یکسان.

**۸) لاگ محلی**
دیباگ سخت. متمرکز.

**۹) بدون مانیتورینگ VIP**
نمی‌دانی کدام master است.

**۱۰) `keepalived` بدون `script_user`**
اسکریپت اجرا نمی‌شود. تنظیم.

### نکات مهم (Tips & Tricks)

- **`vrrp_script`** با `/health`.
- **`priority`** متفاوت.
- **`nopreempt`** برای جلوگیری از flapping.
- **session در Redis**.
- **config یکسان** با IaC.
- **`notify`** برای لاگ.
- **`ip addr`** برای بررسی VIP.

### خلاصه فصل

- A-P ساده، A-A کارآمد.
- VIP با Keepalived جابه‌جا می‌شود.
- session و محتوا باید مشترک/یکسان باشند.
- `health check` واقعی، نه TCP.
- `split-brain` با priority/quorum.
- `failback` خودکار می‌تواند مضر باشد.

**سؤال مروری:** چرا دو سرور بدون VIP، «HA» نیست؟

---

## فصل ۶۵: Keepalived، VIP و health check

### مقدمه

`Keepalived` پیاده‌سازی `VRRP` (Virtual Router Redundancy Protocol) در لینوکس است. یک IP مجازی بین سرورها جابه‌جا می‌شود، بدون تغییر در کلاینت.

### مفاهیم اساسی

**۱) `VRRP` چطور کار می‌کند**

```
MASTER ──advert (هر ۱s)──▶ multicast 224.0.0.18
BACKUP ──(منتظر)──────────▶
   │
   └── اگر ۳ advert نیامد → BACKUP خود را MASTER می‌کند
```

**۲) `virtual_router_id`**

شناسه‌ی گروه VRRP. باید روی همه‌ی سرورها یکسان و با شبکه‌ی دیگر متفاوت باشد (۱–۲۵۵).

**۳) `priority`**

بالاتر = MASTER. `100` و `90`.

**۴) `nopreempt`**

بعد از بازگشت MASTER اصلی، VIP برنمی‌گردد (جلوگیری از flapping).

**۵) `track_script`**

اسکریپت دوره‌ای؛ اگر شکست بخورد، priority کم می‌شود.

```conf
vrrp_script chk_http {
    script "/usr/bin/curl -fsS -m 2 http://127.0.0.1/health"
    interval 2
    weight -20
    fall 3
    rise 2
}
```

**۶) `weight`**

`-20` = کاهش priority به اندازه‌ی ۲۰.

**۷) `track_interface`**

اگر اینترفیس down شد، priority کم شود.

```conf
track_interface {
    eth0 weight -30
}
```

**۸) `unicast_peer`**

برای محیط‌هایی که multicast مجاز نیست (ابر).

```conf
vrrp_instance VI_1 {
    # ...
    unicast_src_ip 10.0.0.11
    unicast_peer {
        10.0.0.12
    }
}
```

**۹) `sync_group`**

چند `vrrp_instance` با هم جابه‌جا شوند.

```conf
vrrp_sync_group VG1 {
    group {
        VI_1
        VI_2
    }
}
```

**۱۰) `arp_ignore` و `arp_announce`**

```bash
# /etc/sysctl.d/99-keepalived.conf
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
net.ipv4.conf.eth0.arp_ignore = 1
net.ipv4.conf.eth0.arp_announce = 2
```

### مثال‌های عملی

**Lab Scenario: HA کامل با unicast و health check**

```conf
# /etc/keepalived/keepalived.conf — web1 (MASTER)
global_defs {
    router_id web1
    enable_script_security
    script_user root
}

vrrp_script chk_nginx {
    script "/usr/bin/curl -fsS -m 2 http://127.0.0.1/health || exit 1"
    interval 2
    timeout 3
    fall 3
    rise 2
    weight -20
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    nopreempt

    unicast_src_ip 192.168.56.11
    unicast_peer {
        192.168.56.12
    }

    authentication {
        auth_type PASS
        auth_pass S3cret
    }

    virtual_ipaddress {
        192.168.56.100/24 dev eth0
    }

    track_script {
        chk_nginx
    }

    track_interface {
        eth0 weight -30
    }

    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault  "/etc/keepalived/notify.sh fault"
}
```

```conf
# web2 (BACKUP) — تفاوت‌ها
vrrp_instance VI_1 {
    state BACKUP
    priority 90
    unicast_src_ip 192.168.56.12
    unicast_peer {
        192.168.56.11
    }
    # بقیه یکسان
}
```

```bash
sudo systemctl restart keepalived
sudo journalctl -u keepalived -n 30 --no-pager
```

**بررسی وضعیت:**

```bash
# VIP کجاست؟
ip -4 addr show eth0 | grep -A1 inet

# وضعیت VRRP
sudo journalctl -u keepalived | grep -E "Entering|MASTER|BACKUP" | tail
```

**تست failover با `weight`:**

```bash
# روی web1، اپ را خراب کن
sudo systemctl stop nginx

# priority از 100 به 80 می‌رسد (< 90)
# web2 MASTER می‌شود
sleep 8
curl -sS http://192.168.56.100/health

# بازگرداندن
sudo systemctl start nginx
# با nopreempt، web2 MASTER می‌ماند
```

**تست `split-brain`:**

```bash
# روی web1 فایروال VRRP را ببند
sudo iptables -A INPUT -p vrrp -j DROP
sudo iptables -A OUTPUT -p vrrp -j DROP

# هر دو MASTER می‌شوند (خطر!)
# راه‌حل: unicast + فایروال/شبکه‌ی قابل اعتماد
```

**`notify.sh`:**

```bash
#!/usr/bin/env bash
# /etc/keepalived/notify.sh
set -euo pipefail
STATE=${1:-unknown}
LOG=/var/log/keepalived-state.log

echo "$(date -Is) state=$STATE host=$(hostname)" >> "$LOG"

case "$STATE" in
  master)
    # اطمینان از سرویس
    systemctl is-active --quiet nginx || systemctl start nginx
    ;;
  backup|fault)
    ;;
esac
```

```bash
sudo chmod +x /etc/keepalived/notify.sh
```

**`VRRP` در ابر (بدون multicast):**

```conf
vrrp_instance VI_1 {
    # ...
    unicast_src_ip 10.0.0.11
    unicast_peer {
        10.0.0.12
    }
}
```

> در AWS، VIP با VRRP کار نمی‌کند (محدودیت شبکه). از NLB یا `Elastic IP` با اسکریپت استفاده کن.

**`keepalived` + `HAProxy`:**

```
VIP → HAProxy (web1) → app1, app2
VIP → HAProxy (web2) → app1, app2
```

`keepalived` فقط VIP را جابه‌جا می‌کند؛ `HAProxy` بار را توزیع می‌کند.

### مقایسه و الگوهای مختلف

**Multicast در برابر Unicast**

| | multicast | unicast |
|---|---|---|
| پیکربندی | ✅ ساده | ⚠️ |
| ابر | ❌ | ✅ |
| امنیت | ⚠️ | ✅ |

**`nopreempt` در برابر پیش‌فرض**

| | `nopreempt` | پیش‌فرض |
|---|---|---|
| failback | ❌ | ✅ |
| flapping | ✅ کم | ⚠️ |
| مناسب | ✅ | — |

**`weight -20` در برابر `-100`**

| | `-20` | `-100` |
|---|---|---|
| کاهش | جزئی | کامل |
| مناسب | چند سرویس | سرویس اصلی |

**دیدگاه «`Keepalived` برای ابر»**: VRRP در AWS/Azure محدود است. NLB/ALB یا VIP با API.

### اشکال شایع

**۱) `virtual_router_id` تکراری**
دو گروه قاطی. شناسه‌ی یکتا.

**۲) `auth_pass` متفاوت**
VIP جابه‌جا نمی‌شود. یکسان.

**۳) `health check` بدون `timeout`**
اسکریپت گیر می‌کند.

**۴) `script_user` نبودن**
اسکریپت اجرا نمی‌شود.

**۵) `split-brain`**
دو MASTER. `unicast` + مانیتورینگ.

**۶) `arp_ignore` تنظیم نشده**
ARP conflict.

**۷) VIP روی اینترفیس اشتباه**
`interface eth0` اشتباه.

**۸) `priority` یکسان**
تصادفی. تفاوت.

**۹) `failback` خودکار در سرویس حساس**
flapping. `nopreempt`.

**۱۰) بدون لاگ state**
نمی‌دانی کِی جابه‌جا شد. `notify`.

### نکات مهم (Tips & Tricks)

- **`unicast_peer`** در ابر.
- **`nopreempt`** برای پایداری.
- **`track_script`** با `/health`.
- **`arp_ignore`**.
- **`notify`** برای لاگ.
- **`ip addr`** برای بررسی.
- **`journalctl -u keepalived`**.

### خلاصه فصل

- `Keepalived` با VRRP، VIP را جابه‌جا می‌کند.
- `priority` و `advert_int` تعیین‌کننده.
- `track_script` سلامت را چک می‌کند.
- `unicast` برای ابر.
- `nopreempt` از flapping جلوگیری می‌کند.
- `split-brain` با unicast و مانیتورینگ.

**سؤال مروری:** چرا `nopreempt` می‌تواند از flapping جلوگیری کند؟

---

## فصل ۶۶: Kubernetes Ingress و Ingress Controller

### مقدمه

در Kubernetes، سرویس‌ها IP داخلی دارند. `Ingress` لایه‌ی HTTP است که ترافیک بیرونی را به سرویس‌ها می‌رساند: host-based، path-based، TLS.

### مفاهیم اساسی

**۱) اجزا**

```
اینترنت → LoadBalancer Service → Ingress Controller (Nginx) → Service → Pod
```

| جزء | کار |
|---|---|
| `Ingress` | قواعد مسیریابی (منبع K8s) |
| `Ingress Controller` | پیاده‌سازی (Nginx، Traefik، HAProxy) |
| `Service` | IP پایدار + load balancing داخلی |
| `Pod` | اپ |

**۲) `Ingress` نمونه**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: lab
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
    - hosts: [lab.example.com]
      secretName: lab-tls
  rules:
    - host: lab.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 80
```

**۳) `pathType`**

| مقدار | معنا |
|---|---|
| `Prefix` | پیشوند مسیر |
| `Exact` | دقیق |
| `ImplementationSpecific` | به controller |

**۴) Ingress Controller — نصب**

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

**۵) TLS secret**

```bash
kubectl create secret tls lab-tls \
  --cert=fullchain.pem --key=privkey.pem -n lab
```

یا با `cert-manager`:

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: lab-tls
  namespace: lab
spec:
  secretName: lab-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
    - lab.example.com
```

**۶) Annotationهای مهم**

| annotation | کار |
|---|---|
| `ssl-redirect` | HTTPS اجباری |
| `proxy-body-size` | حجم بدنه |
| `proxy-read-timeout` | timeout |
| `limit-rps` | نرخ |
| `configuration-snippet` | nginx خام |
| `rewrite-target` | بازنویسی |
| `auth-type`/`auth-secret` | Basic auth |

**۷) `rewrite-target`**

```yaml
annotations:
  nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
    - http:
        paths:
          - path: /api(/|$)(.*)
            pathType: ImplementationSpecific
            backend:
              service: { name: api, port: { number: 80 } }
```

**۸) `default backend`**

```yaml
spec:
  defaultBackend:
    service:
      name: fallback
      port:
        number: 80
```

**۹) `IngressClass`**

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: nginx
spec:
  controller: k8s.io/ingress-nginx
```

**۱۰) `Gateway API`**

جانشین آینده‌ی `Ingress` با انعطاف بیشتر (`HTTPRoute`، `Gateway`).

### مثال‌های عملی

**Lab Scenario: استقرار کامل**

```yaml
# namespace
apiVersion: v1
kind: Namespace
metadata:
  name: lab
---
# web
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: lab
spec:
  replicas: 2
  selector:
    matchLabels: { app: web }
  template:
    metadata:
      labels: { app: web }
    spec:
      containers:
        - name: web
          image: nginx:1.26-alpine
          ports: [{ containerPort: 80 }]
          readinessProbe:
            httpGet: { path: /, port: 80 }
            initialDelaySeconds: 3
---
apiVersion: v1
kind: Service
metadata:
  name: web
  namespace: lab
spec:
  selector: { app: web }
  ports: [{ port: 80, targetPort: 80 }]
---
# api
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  namespace: lab
spec:
  replicas: 2
  selector:
    matchLabels: { app: api }
  template:
    metadata:
      labels: { app: api }
    spec:
      containers:
        - name: api
          image: hashicorp/http-echo:1.0
          args: ["-text=api ok"]
          ports: [{ containerPort: 5678 }]
          readinessProbe:
            httpGet: { path: /, port: 5678 }
            initialDelaySeconds: 2
---
apiVersion: v1
kind: Service
metadata:
  name: api
  namespace: lab
spec:
  selector: { app: api }
  ports: [{ port: 80, targetPort: 5678 }]
```

```bash
kubectl apply -f lab.yaml
kubectl -n lab get pods,svc
```

```yaml
# ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: lab
  namespace: lab
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "16m"
spec:
  ingressClassName: nginx
  tls:
    - hosts: [lab.example.com]
      secretName: lab-tls
  rules:
    - host: lab.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service: { name: api, port: { number: 80 } }
          - path: /
            pathType: Prefix
            backend:
              service: { name: web, port: { number: 80 } }
```

```bash
kubectl apply -f ingress.yaml
kubectl -n lab get ingress
kubectl -n lab describe ingress lab

# آدرس
kubectl -n ingress-nginx get svc ingress-nginx-controller

# تست
curl -sS -H 'Host: lab.example.com' http://<EXTERNAL-IP>/api
curl -sS -H 'Host: lab.example.com' http://<EXTERNAL-IP>/
```

**TLS با `cert-manager`:**

```bash
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager --create-namespace \
  --set installCRDs=true
```

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
      - http01:
          ingress:
            class: nginx
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: lab
  namespace: lab
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
    - hosts: [lab.example.com]
      secretName: lab-tls
  rules:
    - host: lab.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service: { name: web, port: { number: 80 } }
```

```bash
kubectl -n lab get certificate,secret
kubectl -n lab describe certificate lab-tls
```

**Basic auth:**

```bash
htpasswd -c auth admin
kubectl -n lab create secret generic basic-auth --from-file=auth
```

```yaml
annotations:
  nginx.ingress.kubernetes.io/auth-type: basic
  nginx.ingress.kubernetes.io/auth-secret: basic-auth
  nginx.ingress.kubernetes.io/auth-realm: "Authentication Required"
```

**rate limit:**

```yaml
annotations:
  nginx.ingress.kubernetes.io/limit-rps: "10"
  nginx.ingress.kubernetes.io/limit-burst-multiplier: "5"
  nginx.ingress.kubernetes.io/limit-connections: "20"
```

**canary:**

```yaml
# ingress اصلی
metadata:
  name: web
  annotations:
    nginx.ingress.kubernetes.io/canary: "false"
---
# canary
metadata:
  name: web-canary
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"
```

### مقایسه و الگوهای مختلف

**`Ingress` در برابر `Gateway API`**

| | Ingress | Gateway API |
|---|---|---|
| سادگی | ✅ | ⚠️ |
| انعطاف | ⚠️ | ✅✅ |
| maturity | ✅ | ⚠️ |
| مناسب | اکثر | آینده |

**`ingress-nginx` در برابر `Traefik` در برابر `HAProxy`**

| | nginx | Traefik | HAProxy |
|---|---|---|---|
| محبوبیت | ✅✅ | ✅ | ⚠️ |
| CRD | ⚠️ | ✅ | ✅ |
| Dashboard | ⚠️ | ✅ | ⚠️ |
| مناسب | ✅ عمومی | K8s-native | LB تخصصی |

**`pathType: Prefix` در برابر `Exact`**

| | Prefix | Exact |
|---|---|---|
| `/api` | `/api`, `/api/x` | فقط `/api` |
| مناسب | ✅ | مسیر دقیق |

**دیدگاه «`Ingress` برای همه‌چیز»**: برای HTTP عالی است. برای TCP/UDP، `Service: LoadBalancer` یا `Gateway API` (TCPRoute).

### اشکال شایع

**۱) `ingressClassName` نبودن**
Ingress نادیده گرفته می‌شود. تنظیم.

**۲) `path` بدون `/`**
خطا. `/api`.

**۳) `tls.secretName` ناموجود**
گواهی پیش‌فرض. ساخت secret.

**۴) `rewrite-target` بدون `pathType: ImplementationSpecific`**
کار نمی‌کند.

**۵) `proxy-body-size` پیش‌فرض ۱m**
`413`. annotation.

**۶) `readinessProbe` نبودن**
ترافیک به pod آماده‌نشده. probe.

**۷) `Host` بدون DNS**
`404 default backend`. DNS.

**۸) `cert-manager` و `ClusterIssuer` اشتباه**
گواهی صادر نمی‌شود. `describe certificate`.

**۹) `limit-rps` کم**
`503`. تنظیم.

**۱۰) چند Ingress با host یکسان**
تعارض. یکی.

### نکات مهم (Tips & Tricks)

- **`ingressClassName`** صریح.
- **`readinessProbe`** برای همه.
- **`cert-manager`** برای TLS.
- **`proxy-body-size`**.
- **`kubectl describe ingress`**.
- **`rewrite-target`** با regex.
- **canary** برای deploy تدریجی.

### خلاصه فصل

- `Ingress` قواعد HTTP، Controller پیاده‌سازی.
- `pathType` و `host` مسیریابی را تعیین می‌کنند.
- TLS با secret یا `cert-manager`.
- Annotationها تنظیمات Nginx را می‌دهند.
- `readinessProbe` برای ترافیک سالم.
- `Gateway API` آینده است.

**سؤال مروری:** چرا بدون `readinessProbe`، ترافیک به pod آماده‌نشده می‌رود؟

---

## فصل ۶۷: CDN و لبه — کش، WAF و DDoS

### مقدمه

CDN فایل‌ها را از نزدیک‌ترین نقطه به کاربر سرو می‌کند: تأخیر کمتر، بار کمتر روی origin، و محافظت در برابر DDoS. ولی پیکربندی اشتباه، کش داده‌ی شخصی یا IP اشتباه در لاگ می‌دهد.

### مفاهیم اساسی

**۱) معماری**

```
کاربر → Edge (نزدیک) → ... → Origin (سرور شما)
         ├── کش
         ├── TLS
         ├── WAF
         └── DDoS
```

**۲) چه چیزی کش شود**

| نوع | کش |
|---|---|
| تصویر، CSS، JS | ✅ |
| HTML | ⚠️ (کوتاه) |
| API عمومی | ✅ (کوتاه) |
| API خصوصی | ❌ |
| فایل کاربر | ❌ |

**۳) `Cache-Control` و CDN**

CDN از هدرهای origin تبعیت می‌کند (یا override می‌کند).

```http
Cache-Control: public, max-age=3600, s-maxage=86400
```

`s-maxage` مخصوص کش مشترک (CDN).

**۴) `stale-while-revalidate` در CDN**

```http
Cache-Control: public, max-age=60, s-maxage=300, stale-while-revalidate=60
```

**۵) IP واقعی**

```
کاربر → CDN → origin
        X-Forwarded-For: 203.0.113.45
        CF-Connecting-IP: 203.0.113.45
```

origin باید `set_real_ip_from` با CIDR CDN داشته باشد.

**۶) TLS در لبه**

| مدل | توضیح |
|---|---|
| Full (strict) | CDN → origin با TLS و تأیید |
| Full | TLS بدون تأیید |
| Flexible | HTTP به origin (❌ خطرناک) |
| End-to-end | TLS تا origin |

`Flexible` باعث redirect loop و ناامنی می‌شود.

**۷) WAF در لبه**

قواعد OWASP، rate limit، bot management.

**۸) DDoS**

L3/L4 در لبه، L7 با rate limit و challenge.

**۹) پنهان کردن origin**

اگر IP origin لو برود، CDN بی‌فایده است:

- فایروال origin فقط CIDR CDN
- DNS فقط از CDN
- بدون IP عمومی دیگر

**۱۰) `Vary` و CDN**

```http
Vary: Accept-Encoding
```

بدون آن، نسخه‌ی gzip به کلاینت ناسازگار می‌رسد.

### مثال‌های عملی

**Lab Scenario: Cloudflare جلوی Nginx**

```nginx
# origin: فقط Cloudflare
# /etc/nginx/conf.d/cloudflare.conf
# لیست CIDR از https://www.cloudflare.com/ips/

set_real_ip_from 173.245.48.0/20;
set_real_ip_from 103.21.244.0/22;
set_real_ip_from 103.22.200.0/22;
set_real_ip_from 103.31.4.0/22;
set_real_ip_from 141.101.64.0/18;
set_real_ip_from 108.162.192.0/18;
set_real_ip_from 190.93.240.0/20;
set_real_ip_from 188.114.96.0/20;
set_real_ip_from 197.234.240.0/22;
set_real_ip_from 198.41.128.0/17;
set_real_ip_from 162.158.0.0/15;
set_real_ip_from 104.16.0.0/13;
set_real_ip_from 104.24.0.0/14;
set_real_ip_from 172.64.0.0/13;
set_real_ip_from 131.0.72.0/22;
set_real_ip_from 2400:cb00::/32;
set_real_ip_from 2606:4700::/32;
set_real_ip_from 2803:f800::/32;
set_real_ip_from 2405:b500::/32;
set_real_ip_from 2405:8100::/32;
set_real_ip_from 2a06:98c0::/29;
set_real_ip_from 2c0f:f248::/32;

real_ip_header CF-Connecting-IP;
real_ip_recursive on;
```

```nginx
# فقط Cloudflare
server {
    listen 443 ssl;
    http2 on;
    server_name lab.example.com;

    allow 173.245.48.0/20;
    allow 103.21.244.0/22;
    # ... بقیه
    deny all;

    ssl_certificate     /etc/ssl/lab/fullchain.pem;
    ssl_certificate_key /etc/ssl/lab/privkey.pem;

    root /var/www/lab;
    location / { try_files $uri $uri/ =404; }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx

# تست IP واقعی
curl -sS -H 'CF-Connecting-IP: 203.0.113.45' https://lab.example.com/whoami
```

**هدرهای کش برای CDN:**

```nginx
location ~* \.(css|js|jpg|png|woff2)$ {
    expires 1y;
    add_header Cache-Control "public, max-age=31536000, immutable" always;
}

location = /index.html {
    add_header Cache-Control "no-cache" always;
}

location /api/public/ {
    add_header Cache-Control "public, max-age=60, s-maxage=300, stale-while-revalidate=60" always;
}

location /api/private/ {
    add_header Cache-Control "private, no-store" always;
}
```

**purge کش CDN:**

```bash
# Cloudflare API
curl -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/purge_cache" \
  -H "Authorization: Bearer $CF_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"files":["https://lab.example.com/app.css"]}'
```

**origin با فایروال:**

```bash
# فقط Cloudflare به ۴۴۳
sudo ufw allow from 173.245.48.0/20 to any port 443
# ... بقیه CIDR
sudo ufw deny 443
```

**AWS CloudFront — نمونه:**

```hcl
resource "aws_cloudfront_distribution" "lab" {
  enabled = true
  aliases = ["lab.example.com"]

  origin {
    domain_name = "origin.lab.example.com"
    origin_id   = "origin"

    custom_origin_config {
      http_port              = 80
      https_port             = 443
      origin_protocol_policy = "https-only"
      origin_ssl_protocols   = ["TLSv1.2"]
    }
  }

  default_cache_behavior {
    target_origin_id       = "origin"
    viewer_protocol_policy = "redirect-to-https"
    allowed_methods        = ["GET", "HEAD", "OPTIONS"]
    cached_methods         = ["GET", "HEAD"]
    compress               = true

    forwarded_values {
      query_string = false
      cookies { forward = "none" }
    }

    min_ttl     = 0
    default_ttl = 3600
    max_ttl     = 86400
  }

  ordered_cache_behavior {
    path_pattern           = "/api/*"
    target_origin_id       = "origin"
    viewer_protocol_policy = "https-only"
    allowed_methods        = ["GET", "HEAD", "OPTIONS", "PUT", "POST", "PATCH", "DELETE"]
    cached_methods         = ["GET", "HEAD"]
    min_ttl                = 0
    default_ttl            = 0
    max_ttl                = 0

    forwarded_values {
      query_string = true
      headers      = ["Authorization"]
      cookies { forward = "all" }
    }
  }

  viewer_certificate {
    acm_certificate_arn = var.cert_arn
    ssl_support_method  = "sni-only"
  }
}
```

**تست کش CDN:**

```bash
# هدرهای CDN
curl -sSI https://lab.example.com/app.css | grep -Ei 'cf-cache-status|age|x-cache'

# cf-cache-status: HIT / MISS / EXPIRED
```

**تست IP واقعی:**

```bash
# لاگ باید IP کاربر را نشان دهد، نه CDN
sudo tail -1 /var/log/nginx/access.log
```

### مقایسه و الگوهای مختلف

**Cloudflare در برابر CloudFront در برابر Fastly**

| | Cloudflare | CloudFront | Fastly |
|---|---|---|---|
| راه‌اندازی | ✅✅ | ⚠️ | ⚠️ |
| WAF | ✅ | ✅ | ✅ |
| قیمت | ✅ | ⚠️ | 💰 |
| مناسب | ✅ عمومی | AWS | VOD/edge compute |

**`Full (strict)` در برابر `Flexible`**

| | Full strict | Flexible |
|---|---|---|
| امنیت | ✅ | ❌ |
| redirect loop | ❌ | ✅ ممکن |
| توصیه | ✅ | ❌ |

**کش CDN در برابر کش origin**

| | CDN | origin |
|---|---|---|
| نزدیکی | ✅✅ | ⚠️ |
| بار origin | ✅ کم | ⚠️ |
| کنترل | ⚠️ | ✅ |

**دیدگاه «CDN فقط برای استاتیک»**: برای API هم می‌شود (کش کوتاه، `s-maxage`). ولی داده‌ی خصوصی نه.

### اشکال شایع

**۱) `Flexible` SSL**
داده بین CDN و origin رمزنگاری نمی‌شود. `Full (strict)`.

**۲) کش HTML با `max-age` بلند**
کاربر نسخه‌ی قدیمی. `no-cache`.

**۳) `Vary` نبودن**
gzip اشتباه. `gzip_vary on`.

**۴) IP origin لو رفته**
CDN بی‌فایده. فایروال.

**۵) `set_real_ip_from` نبودن**
لاگ IP اشتباه، rate limit غلط.

**۶) کش `Set-Cookie`**
داده‌ی شخصی. `private`.

**۷) purge نکردن بعد از deploy**
نسخه‌ی قدیمی. purge یا hash.

**۸) `stale-while-revalidate` نبودن**
تأخیر در بازسازی.

**۹) CDN و `Authorization`**
کش پاسخ شخصی. `Vary: Authorization` یا `private`.

**۱۰) بدون مانیتورینگ `cf-cache-status`**
نمی‌دانی کش کار می‌کند.

### نکات مهم (Tips & Tricks)

- **`Full (strict)`**.
- **`set_real_ip_from`** با CIDR CDN.
- **`s-maxage`** برای CDN.
- **`private, no-store`** برای داده‌ی کاربر.
- **فایروال origin** فقط CIDR CDN.
- **purge یا hash** بعد از deploy.
- **`cf-cache-status`** برای دیباگ.

### خلاصه فصل

- CDN تأخیر و بار را کم می‌کند.
- `Cache-Control` و `s-maxage` حاکم.
- IP واقعی با `set_real_ip_from`.
- `Flexible` SSL خطرناک است.
- origin را پنهان کن.
- purge یا hash برای نسخه‌ی جدید.

**سؤال مروری:** چرا `Flexible SSL` در Cloudflare می‌تواند هم امنیت و هم redirect loop ایجاد کند؟

---

## منابع و مراجع

**مستندات رسمی**
- [Keepalived](https://www.keepalived.org/doc/)
- [VRRP — RFC 5798](https://www.rfc-editor.org/rfc/rfc5798.html)
- [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [ingress-nginx](https://kubernetes.github.io/ingress-nginx/)
- [cert-manager](https://cert-manager.io/docs/)
- [Gateway API](https://gateway-api.sigs.k8s.io/)
- [Cloudflare — IP ranges](https://www.cloudflare.com/ips/)
- [AWS CloudFront](https://docs.aws.amazon.com/cloudfront/)

**آموزش**
- [Keepalived — practical guide](https://www.redhat.com/sysadmin/keepalived-basics)
- [Kubernetes Ingress — deep dive](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
- [CDN caching best practices](https://developers.cloudflare.com/cache/)

**ابزار**
- [kubectl](https://kubernetes.io/docs/reference/kubectl/)
- [helm](https://helm.sh/)
- [curl](https://curl.se/)

---

**بعدی:** [`16-Containers-and-Deployment.md`](16-Containers-and-Deployment.md) — Docker، Compose، K8s و CI/CD
