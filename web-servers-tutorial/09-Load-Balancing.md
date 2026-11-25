# بخش ۰۹: Load Balancing

**مدت یادگیری**: ۵ تا ۶ ساعت
**سطح سختی**: متوسط تا پیشرفته
**پیش‌نیاز**: بخش ۰۸ (Reverse Proxy، upstream)

---

## فهرست محتوا

- [فصل ۳۶: مفاهیم — L4 در برابر L7 و الگوریتم‌ها](#فصل-۳۶)
- [فصل ۳۷: Nginx — `upstream`، `keepalive` و health check](#فصل-۳۷)
- [فصل ۳۸: HAProxy و `mod_proxy_balancer`](#فصل-۳۸)
- [فصل ۳۹: Sticky Session و اشتراک Session](#فصل-۳۹)
- [فصل ۴۰: ARR در IIS و Load Balancer ابری](#فصل-۴۰)

---

## فصل ۳۶: مفاهیم — L4 در برابر L7 و الگوریتم‌ها

### مقدمه

یک سرور، هزار درخواست در ثانیه. دو سرور، دو هزار. ولی توزیع درست، ساده‌تر از «نصف نصف» است: باید بدانی چه چیزی را توزیع می‌کنی (اتصال یا درخواست)، با چه الگوریتمی، و اگر یک سرور مرد چه می‌کنی.

### مفاهیم اساسی

**۱) L4 در برابر L7**

```
L4 (Transport):
کلاینت ──TCP──▶ LB ──TCP──▶ سرور
         (بدون دیدن HTTP)

L7 (Application):
کلاینت ──HTTP──▶ LB ──HTTP──▶ سرور
         (مسیر، هدر، کوکی را می‌بیند)
```

| | L4 | L7 |
|---|---|---|
| لایه | TCP/UDP | HTTP |
| سرعت | ✅✅ | ✅ |
| محتوای مسیر | ❌ | ✅ |
| TLS termination | ⚠️ (passthrough) | ✅ |
| sticky با کوکی | ❌ | ✅ |
| نمونه | AWS NLB، HAProxy `mode tcp` | Nginx، ALB، HAProxy `mode http` |

**۲) دیاگرام**

```
                ┌──────────────────────────┐
    کلاینت ────▶│  Load Balancer (L7)      │
                │  - TLS termination       │
                │  - health check          │
                │  - algorithm             │
                └────┬────────┬────────┬───┘
                     │        │        │
                     ▼        ▼        ▼
                  ┌─────┐  ┌─────┐  ┌─────┐
                  │app 1│  │app 2│  │app 3│
                  └─────┘  └─────┘  └─────┘
                     │        │        │
                     └────────┼────────┘
                              ▼
                        ┌──────────┐
                        │ DB / Cache│
                        └──────────┘
```

**۳) الگوریتم‌ها**

| الگوریتم | رفتار | مناسب |
|---|---|---|
| **Round Robin** | نوبتی | درخواست هم‌زمان یکسان |
| **Weighted RR** | نوبتی با وزن | سرور با ظرفیت متفاوت |
| **Least Connections** | کمترین اتصال فعال | زمان پاسخ متغیر |
| **Least Response Time** | کمترین زمان پاسخ | نیاز به معیار کیفیت |
| **IP Hash** | هش IP | sticky بدون کوکی |
| **Consistent Hash** | هش کلید (URI/هدر) | کش |
| **Random** | تصادفی | ساده، مقیاس بزرگ |

**۴) Health Check**

| نوع | معنا |
|---|---|
| **Passive** | بر اساس خطای واقعی (۵۰۲، timeout) |
| **Active** | درخواست دوره‌ای به `/health` |

`Active` سریع‌تر خطا را می‌بیند؛ `Passive` سربار ندارد.

**۵) `/health` درست**

```python
@app.get("/health")
def health():
    # فقط «زنده بودن» را چک نکن
    db_ok = check_db()
    cache_ok = check_cache()
    if not (db_ok and cache_ok):
        raise HTTPException(503)
    return {"status": "ok"}
```

> `health` باید وابستگی‌های حیاتی را چک کند، نه فقط `return 200`.

**۶) `readiness` در برابر `liveness`**

| | liveness | readiness |
|---|---|---|
| سؤال | زنده‌ای؟ | آماده‌ی ترافیک؟ |
| خطا | restart | حذف از LB |
| مثال | deadlock | DB قطع |

**۷) `connection draining`**

قبل از حذف سرور، اتصال‌های فعال تمام شوند:

```nginx
# Nginx Plus: slow_start، max_conns
# HAProxy: `disable` روی server، سپس `shutdown-sessions`
```

**۸) `max_conns` و `max_fails`**

```nginx
upstream backend {
    server 10.0.0.11:8000 max_conns=200;
    server 10.0.0.12:8000 max_conns=200;
    server 10.0.0.13:8000 backup;
}
```

**۹) `slow start`**

سروری که تازه اضافه شده، آرام‌آرام ترافیک بگیرد (Nginx Plus، HAProxy):

```
سرور جدید → ۱۰٪ ترافیک → ۵۰٪ → ۱۰۰٪ در ۶۰ ثانیه
```

**۱۰) `session` و state**

اگر session در حافظه‌ی سرور است، LB باید sticky باشد. اگر session در Redis است، هر سروری می‌تواند پاسخ دهد.

### مثال‌های عملی

**Lab Scenario: سه بک‌اند با وزن متفاوت**

```bash
# سه نمونه اپ روی پورت‌های مختلف
for p in 8000 8001 8002; do
  PORT=$p nohup python3 -m http.server $p --bind 127.0.0.1 >/dev/null 2>&1 &
done

# یا با یک اپ که نام نمونه را برمی‌گرداند
```

```nginx
upstream app {
    least_conn;
    server 127.0.0.1:8000 weight=3 max_fails=3 fail_timeout=10s;
    server 127.0.0.1:8001 weight=1;
    server 127.0.0.1:8002 backup;
    keepalive 32;
}

server {
    listen 8080;
    location / {
        proxy_pass http://app;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        add_header X-Upstream $upstream_addr always;
        add_header X-Upstream-Status $upstream_status always;
    }
}
```

```bash
# توزیع
for i in $(seq 1 40); do
  curl -sSI http://localhost:8080/ | grep -i x-upstream
done | sort | uniq -c
```

**تست failover:**

```bash
# بستن 8000
kill %1

for i in $(seq 1 10); do
  curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:8080/
done | sort | uniq -c
# همه 200، از 8001 یا 8002
```

**مقایسه‌ی الگوریتم‌ها:**

```bash
# round-robin
sed -i 's/least_conn;//' /etc/nginx/conf.d/lb.conf
sudo nginx -s reload
for i in $(seq 1 30); do curl -sSI http://localhost:8080/ | grep -i x-upstream; done | sort | uniq -c

# least_conn
sed -i 's/upstream app {/upstream app {\n    least_conn;/' /etc/nginx/conf.d/lb.conf
sudo nginx -s reload
for i in $(seq 1 30); do curl -sSI http://localhost:8080/ | grep -i x-upstream; done | sort | uniq -c
```

**health check فعال با HAProxy (فصل ۳۸):**

```haproxy
backend app
    option httpchk GET /health
    http-check expect status 200
    server app1 10.0.0.11:8000 check inter 2s fall 3 rise 2
    server app2 10.0.0.12:8000 check inter 2s fall 3 rise 2
```

### مقایسه و الگوهای مختلف

**L4 در برابر L7**

| معیار | L4 | L7 |
|---|---|---|
| throughput | ✅✅ | ✅ |
| تأخیر | ✅ | ⚠️ |
| مسیریابی بر اساس مسیر | ❌ | ✅ |
| TLS termination | ⚠️ | ✅ |
| WebSocket | ✅ (passthrough) | ✅ |
| کش | ❌ | ✅ |

**Round Robin در برابر Least Conn**

| | RR | Least Conn |
|---|---|---|
| توزیع | مساوی | بر اساس بار |
| درخواست سریع/کند مخلوط | ❌ | ✅ |
| سادگی | ✅ | ✅ |
| توصیه | بار یکنواخت | ✅ عمومی |

**Passive در برابر Active health check**

| | Passive | Active |
|---|---|---|
| تشخیص | بعد از خطا | قبل از خطا |
| سربار | ✅ صفر | ⚠️ |
| نیاز به endpoint | ❌ | ✅ |
| توصیه | مکمل | ✅ اصلی |

**دیدگاه «LB فقط round-robin»**: برای درخواست‌های هم‌زمان یکسان خوب است. برای API با زمان متغیر، `least_conn` بهتر.

### اشکال شایع

**۱) `health` که فقط `200` می‌دهد**
سرور با DB قطع، ترافیک می‌گیرد. وابستگی‌ها را چک کن.

**۲) `health` روی مسیر پرهزینه**
هر ۲ ثانیه، کوئری سنگین. `/health` سبک باشد.

**۳) `max_fails` پیش‌فرض**
در Nginx OSS، health check فعال نیست؛ فقط passive. برای فعال، Plus یا HAProxy.

**۴) `backup` همیشه فعال**
اگر `backup` ترافیک می‌گیرد، تنظیم اشتباه است.

**۵) بدون `keepalive`**
اتصال جدید برای هر درخواست. `keepalive 32`.

**۶) `least_conn` با `weight` ناهماهنگ**
رفتار پیچیده. تست کن.

**۷) `ip_hash` با NAT**
همه‌ی کاربران پشت NAT، یک سرور. `hash $cookie_session`.

**۸) بدون `connection draining`**
restart سرور، درخواست‌های فعال قطع.

**۹) `proxy_next_upstream` روی `POST`**
تکرار درخواست. محدود کن.

**۱۰) LB بدون مانیتورینگ**
نمی‌دانی کدام سرور چند درخواست دارد. `stub_status`/metrics.

### نکات مهم (Tips & Tricks)

- **`X-Upstream`** هدر برای دیباگ.
- **`least_conn`** پیش‌فرض خوب.
- **`keepalive`** در upstream.
- **`/health` سبک و واقعی**.
- **`max_conns`** برای جلوگیری از اشباع.
- **`backup`** برای DR.
- **`stub_status`** برای مانیتورینگ.

### خلاصه فصل

- L4 اتصال، L7 درخواست را توزیع می‌کند.
- الگوریتم‌ها: RR، وزن‌دار، least conn، hash.
- health check فعال سریع‌تر، passive سبک‌تر.
- `keepalive` کارایی را چند برابر می‌کند.
- `connection draining` برای restart بی‌درد.
- `/health` باید وابستگی را چک کند.

**سؤال مروری:** چه زمانی `least_conn` از round-robin بهتر است و چرا؟

---

## فصل ۳۷: Nginx — `upstream`، `keepalive` و health check

### مقدمه

Nginx OSS health check **فعال** ندارد؛ فقط passive (بر اساس `max_fails`). برای active، Nginx Plus یا ماژول ثالث لازم است. این محدودیت مهم است و باید بدانی چطور دورش بزنی.

### مفاهیم اساسی

**۱) `upstream` کامل**

```nginx
upstream backend {
    least_conn;
    server 10.0.0.11:8000 weight=3 max_conns=200 max_fails=3 fail_timeout=10s;
    server 10.0.0.12:8000 weight=1 max_conns=200;
    server 10.0.0.13:8000 backup;
    keepalive 64;
    keepalive_requests 1000;
    keepalive_timeout 60s;
    zone backend_zone 64k;
}
```

`zone` برای اشتراک state بین workerها لازم است (در Plus برای health check).

**۲) `keepalive` — چرا مهم**

بدون `keepalive`:

```
هر درخواست: TCP handshake به upstream (۱ RTT)
۱۰۰۰ درخواست/ثانیه = ۱۰۰۰ اتصال جدید
```

با `keepalive 64`:

```
۶۴ اتصال باز، استفاده‌ی مجدد
```

**شرط:** `proxy_http_version 1.1` و `proxy_set_header Connection ""`.

**۳) Passive health check**

```nginx
upstream backend {
    server 10.0.0.11:8000 max_fails=3 fail_timeout=10s;
    server 10.0.0.12:8000 max_fails=3 fail_timeout=10s;
}
```

- `max_fails=3`: بعد از ۳ خطا
- `fail_timeout=10s`: به مدت ۱۰ ثانیه غیرفعال

**۴) `proxy_next_upstream`**

```nginx
proxy_next_upstream error timeout http_502 http_503 http_504;
proxy_next_upstream_tries 3;
proxy_next_upstream_timeout 15s;
```

**۵) `slow_start` (Plus)**

```nginx
server 10.0.0.13:8000 slow_start=30s;
```

**۶) `resolve` (1.27.3+)**

```nginx
resolver 127.0.0.11 valid=10s;
upstream backend {
    zone backend 64k;
    server app.internal:8000 resolve;
}
```

**۷) `stub_status` و `upstream` state**

Nginx OSS فقط `stub_status` کلی دارد. برای per-upstream، Plus یا `nginx-module-vts`.

**۸) `proxy_next_upstream_tries`**

تعداد تلاش کل. `3` یعنی حداکثر ۳ سرور.

**۹) `max_conns`**

محدودیت اتصال همزمان به یک upstream. برای جلوگیری از اشباع.

**۱۰) `backup` و `down`**

```nginx
server 10.0.0.13:8000 backup;    # فقط اگر بقیه خراب
server 10.0.0.14:8000 down;      # دستی غیرفعال (maintenance)
```

### مثال‌های عملی

**Lab Scenario: LB با سه اپ و health check غیرفعال‌سازی دستی**

```bash
# سه اپ ساده که نام نمونه را برمی‌گردانند
for p in 8000 8001 8002; do
  mkdir -p /tmp/app$p
  cat > /tmp/app$p/server.py <<PY
from http.server import BaseHTTPRequestHandler, HTTPServer
import json, os

class H(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/health':
            self.send_response(200); self.end_headers(); self.wfile.write(b'ok'); return
        body = json.dumps({"instance": $p}).encode()
        self.send_response(200)
        self.send_header('Content-Type','application/json')
        self.send_header('Content-Length', str(len(body)))
        self.end_headers()
        self.wfile.write(body)
    def log_message(self, *a): pass

HTTPServer(('127.0.0.1', $p), H).serve_forever()
PY
  nohup python3 /tmp/app$p/server.py >/dev/null 2>&1 &
done

sleep 1
curl -sS http://127.0.0.1:8000/
```

```nginx
upstream app {
    least_conn;
    server 127.0.0.1:8000 max_fails=2 fail_timeout=10s;
    server 127.0.0.1:8001 max_fails=2 fail_timeout=10s;
    server 127.0.0.1:8002 backup;
    keepalive 32;
}

server {
    listen 8080;
    location / {
        proxy_pass http://app;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        proxy_next_upstream error timeout http_502 http_503 http_504;
        proxy_next_upstream_tries 3;
        add_header X-Upstream $upstream_addr always;
    }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx

# توزیع
for i in $(seq 1 30); do
  curl -sS http://localhost:8080/ | python -c 'import sys,json;print(json.load(sys.stdin)["instance"])'
done | sort | uniq -c
# 15 → 8000
# 15 → 8001
```

**failover خودکار:**

```bash
# بستن 8001
pkill -f 'http.server 8001' || kill %2

for i in $(seq 1 10); do
  curl -sS http://localhost:8080/ | python -c 'import sys,json;print(json.load(sys.stdin)["instance"])'
done | sort | uniq -c
# همه از 8000 (8001 غیرفعال شده)
```

**`backup`:**

```bash
# بستن هر دو
kill %1 %2

for i in $(seq 1 5); do
  curl -sS http://localhost:8080/ | python -c 'import sys,json;print(json.load(sys.stdin)["instance"])'
done | sort | uniq -c
# همه از 8002
```

**health check فعال با ماژول ثالث:**

```bash
# nginx-module-vts یا nginx_upstream_check_module
# یا Nginx Plus:
#   health_check interval=2s fails=3 passes=2 uri=/health;
```

**`stub_status` برای مانیتورینگ:**

```nginx
server {
    listen 127.0.0.1:8081;
    location /status {
        stub_status;
        allow 127.0.0.1;
        deny all;
    }
}
```

```bash
curl -s http://127.0.0.1:8081/status
```

**`proxy_next_upstream` برای `POST`:**

```nginx
location /api/ {
    proxy_pass http://app;
    proxy_next_upstream error timeout;      # فقط error/timeout، نه 5xx
    proxy_next_upstream_non_idempotent off; # POST تکرار نشود
}
```

**`max_conns`:**

```nginx
upstream app {
    server 127.0.0.1:8000 max_conns=100;
    server 127.0.0.1:8001 max_conns=100;
}
```

### مقایسه و الگوهای مختلف

**Nginx OSS در برابر Plus در health check**

| | OSS | Plus |
|---|---|---|
| Passive | ✅ | ✅ |
| Active | ❌ | ✅ |
| `slow_start` | ❌ | ✅ |
| `zone` + state | ⚠️ | ✅ |
| هزینه | ۰ | 💰 |

**`keepalive` کم در برابر زیاد**

| | ۱۶ | ۶۴ | ۲۵۶ |
|---|---|---|---|
| اتصال idle | ✅ | ✅ | ⚠️ |
| throughput | ⚠️ | ✅ | ✅✅ |
| حافظه | ✅ | ✅ | ⚠️ |

**`proxy_next_upstream` محافظه‌کارانه در برابر تهاجمی**

| | فقط error/timeout | + 5xx |
|---|---|---|
| POST امن | ✅ | ⚠️ |
| failover | ⚠️ | ✅ |
| توصیه | ✅ پیش‌فرض | idempotent |

**دیدگاه «Nginx OSS برای LB کافی است»**: برای passive failover، بله. برای active health check و `slow_start`، HAProxy یا Plus.

### اشکال شایع

**۱) `keepalive` بدون `proxy_http_version 1.1`**
بی‌اثر.

**۲) `Connection: upgrade` hardcode**
keepalive خراب. `Connection ""`.

**۳) `max_fails=0`**
هیچ‌وقت غیرفعال نمی‌شود.

**۴) `fail_timeout` کوتاه**
سرور سالم، غیرفعال می‌شود. `10s`.

**۵) `proxy_next_upstream` روی `POST`**
تکرار. `non_idempotent off`.

**۶) `backup` با `least_conn`**
رفتار پیچیده.

**۷) `stub_status` باز**
اطلاعات لو می‌رود. `allow`/`deny`.

**۸) `zone` بدون `keepalive`**
state مشترک ندارد.

**۹) بدون `X-Upstream`**
دیباگ سخت.

**۱۰) `resolve` بدون `zone`**
خطا در نسخه‌های جدید.

### نکات مهم (Tips & Tricks)

- **`keepalive 32`** حداقل.
- **`proxy_http_version 1.1` + `Connection ""`**.
- **`max_fails=2`/`fail_timeout=10s`**.
- **`X-Upstream`** برای دیباگ.
- **`proxy_next_upstream`** محافظه‌کارانه.
- **`stub_status`** محدود.
- **HAProxy** برای active health check.

### خلاصه فصل

- Nginx OSS فقط passive health check دارد.
- `keepalive` + `1.1` + `Connection ""` کارایی را چند برابر می‌کند.
- `max_fails`/`fail_timeout` غیرفعال‌سازی موقت.
- `backup` برای DR.
- `proxy_next_upstream` برای idempotent.
- `X-Upstream` برای دیباگ.

**سؤال مروری:** چرا `proxy_next_upstream` روی `POST` می‌تواند داده‌ی تکراری بسازد؟

---

## فصل ۳۸: HAProxy و `mod_proxy_balancer`

### مقدمه

HAProxy استاندارد صنعت برای load balancing است: active health check، آمار دقیق، `slow start`، `connection draining` و کارایی بالا. Apache هم با `mod_proxy_balancer` گزینه‌ای داخلی دارد.

### مفاهیم اساسی

**۱) HAProxy — ساختار config**

```
global
    ...
defaults
    mode http
    timeout connect 5s
    timeout client 30s
    timeout server 30s
frontend
    bind *:80
    default_backend app
backend
    server ...
```

**۲) `mode`**

| مقدار | لایه |
|---|---|
| `tcp` | L4 |
| `http` | L7 |

**۳) `balance`**

```haproxy
balance roundrobin
balance leastconn
balance source          # IP hash
balance uri             # بر اساس URI
balance hdr(X-Key)
```

**۴) `server` با پارامترها**

```haproxy
server app1 10.0.0.11:8000 check weight 3 maxconn 200 inter 2s fall 3 rise 2
server app2 10.0.0.12:8000 check weight 1 maxconn 200
server app3 10.0.0.13:8000 check backup
```

| پارامتر | معنا |
|---|---|
| `check` | health check فعال |
| `inter` | فاصله‌ی بررسی |
| `fall` | تعداد شکست برای غیرفعال |
| `rise` | تعداد موفقیت برای فعال |
| `maxconn` | سقف اتصال |
| `weight` | وزن |
| `backup` | پشتیبان |

**۵) `option httpchk`**

```haproxy
option httpchk GET /health
http-check expect status 200
```

**۶) `slow start`**

```haproxy
server app1 10.0.0.11:8000 check slowstart 30s
```

**۷) `stats`**

```haproxy
frontend stats
    bind *:8404
    stats enable
    stats uri /stats
    stats refresh 5s
    stats auth admin:password
```

**۸) `acl` و مسیریابی**

```haproxy
frontend http
    bind *:80
    acl is_api path_beg /api
    acl is_ws  path_beg /ws

    use_backend api if is_api
    use_backend ws  if is_ws
    default_backend web
```

**۹) Apache `mod_proxy_balancer`**

```apache
<Proxy "balancer://app">
    BalancerMember http://10.0.0.11:8000 route=node1
    BalancerMember http://10.0.0.12:8000 route=node2
    ProxySet lbmethod=bybusyness
</Proxy>

ProxyPass        / balancer://app/
ProxyPassReverse / balancer://app/
```

**۱۰) `mod_proxy_hcheck`**

```apache
<Proxy "balancer://app">
    BalancerMember http://10.0.0.11:8000 hcmethod=GET hcuri=/health hcinterval=5
    BalancerMember http://10.0.0.12:8000 hcmethod=GET hcuri=/health hcinterval=5
    ProxySet lbmethod=bybusyness hcheck=on
</Proxy>
```

### مثال‌های عملی

**Lab Scenario: HAProxy با health check و stats**

```haproxy
# /etc/haproxy/haproxy.cfg
global
    log /dev/log local0
    maxconn 4096
    daemon

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    option  forwardfor
    timeout connect 5s
    timeout client  30s
    timeout server  30s
    retries 3

frontend http_in
    bind *:8080
    default_backend app

backend app
    balance leastconn
    option httpchk GET /health
    http-check expect status 200

    server app1 127.0.0.1:8000 check inter 2s fall 3 rise 2 weight 3 maxconn 200
    server app2 127.0.0.1:8001 check inter 2s fall 3 rise 2 weight 1 maxconn 200
    server app3 127.0.0.1:8002 check inter 2s fall 3 rise 2 backup

frontend stats
    bind *:8404
    stats enable
    stats uri /stats
    stats refresh 5s
```

```bash
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
sudo systemctl restart haproxy
sudo systemctl status haproxy --no-pager

# تست
for i in $(seq 1 20); do
  curl -sS http://localhost:8080/ | python -c 'import sys,json;print(json.load(sys.stdin)["instance"])'
done | sort | uniq -c

# stats
curl -sS http://localhost:8404/stats
```

**تست active health check:**

```bash
# بستن app1
kill %1

# HAProxy بعد از ۳×۲ ثانیه، app1 را غیرفعال می‌کند
sleep 8
curl -sS http://localhost:8080/ | python -c 'import sys,json;print(json.load(sys.stdin)["instance"])'
# از app2

# در stats، app1 با رنگ قرمز
```

**مسیریابی با ACL:**

```haproxy
frontend http_in
    bind *:8080

    acl is_api path_beg /api
    acl is_static path_beg /assets

    use_backend api if is_api
    use_backend static if is_static
    default_backend web

backend api
    balance leastconn
    option httpchk GET /health
    server api1 127.0.0.1:8000 check
    server api2 127.0.0.1:8001 check

backend web
    balance roundrobin
    server web1 127.0.0.1:8002 check

backend static
    balance roundrobin
    server web1 127.0.0.1:8002 check
```

**`slow start`:**

```haproxy
server app1 127.0.0.1:8000 check slowstart 30s
```

**`connection draining`:**

```bash
# غیرفعال کردن دستی
echo "disable server app/app1" | sudo socat stdio /var/run/haproxy/admin.sock
# صبر تا اتمام اتصال‌ها
echo "shutdown sessions server app/app1" | sudo socat stdio /var/run/haproxy/admin.sock
```

```haproxy
# فعال‌سازی socket
global
    stats socket /var/run/haproxy/admin.sock mode 660 level admin
```

**Apache `mod_proxy_balancer`:**

```bash
sudo a2enmod proxy proxy_http proxy_balancer lbmethod_bybusyness proxy_hcheck
sudo apachectl configtest && sudo systemctl reload apache2
```

```apache
<Proxy "balancer://app">
    BalancerMember http://127.0.0.1:8000 route=node1 hcmethod=GET hcuri=/health hcinterval=5
    BalancerMember http://127.0.0.1:8001 route=node2 hcmethod=GET hcuri=/health hcinterval=5
    ProxySet lbmethod=bybusyness hcheck=on
</Proxy>

<Location />
    ProxyPass        balancer://app/
    ProxyPassReverse balancer://app/
</Location>

<Location /balancer-manager>
    SetHandler balancer-manager
    Require ip 127.0.0.1
</Location>
```

```bash
curl -sS http://localhost:8080/balancer-manager | head -20
```

### مقایسه و الگوهای مختلف

**HAProxy در برابر Nginx در برابر Apache balancer**

| | HAProxy | Nginx OSS | Apache |
|---|---|---|---|
| Active health | ✅ | ❌ | ✅ (`hcheck`) |
| Stats | ✅✅ | ⚠️ | ✅ (`balancer-manager`) |
| `slow start` | ✅ | ❌ | ⚠️ |
| L4 | ✅ | ⚠️ (`stream`) | ❌ |
| Config | ✅ | ✅ | ⚠️ |
| کارایی | ✅✅ | ✅ | ⚠️ |
| مناسب | LB تخصصی | همه‌کاره | اکوسیستم Apache |

**`balance` در HAProxy**

| الگوریتم | مناسب |
|---|---|
| `roundrobin` | بار یکنواخت |
| `leastconn` | زمان متغیر |
| `source` | sticky |
| `uri` | کش |

**`option httpchk` در برابر `check` ساده**

| | `check` | `httpchk` |
|---|---|---|
| بررسی | TCP | HTTP + وضعیت |
| دقت | ⚠️ | ✅ |
| سربار | ✅ کم | ⚠️ |

**دیدگاه «HAProxy فقط برای LB بزرگ»**: برای هر اپ با بیش از یک سرور، HAProxy ساده و قدرتمند است. config کوتاه، stats عالی.

### اشکال شایع

**۱) `timeout` بدون تنظیم**
HAProxy شروع نمی‌شود. `connect`/`client`/`server`.

**۲) `check` بدون `inter`**
پیش‌فرض ۲ ثانیه، ولی صریح بنویس.

**۳) `httpchk` به `/health` سنگین**
بار اضافه. سبک باشد.

**۴) `stats` بدون `auth`**
اطلاعات لو می‌رود.

**۵) `mode tcp` و انتظار `X-Forwarded-For`**
L4 هدر نمی‌بیند. `mode http` یا `send-proxy`.

**۶) `balance source` با NAT**
همه به یک سرور.

**۷) `maxconn` کم**
`503`. افزایش یا `queue`.

**۸) `haproxy -c` نزده**
config خراب، سرویس بالا نمی‌آید.

**۹) `balancer-manager` باز**
دسترسی به تنظیمات. `Require ip`.

**۱۰) `hcheck` بدون `mod_proxy_hcheck`**
`Invalid command`.

### نکات مهم (Tips & Tricks)

- **`haproxy -c -f`** قبل از restart.
- **`stats socket`** برای `disable`/`enable`.
- **`option httpchk`** با `/health`.
- **`slowstart`** برای سرور جدید.
- **`stats auth`**.
- **`mode http`** برای `X-Forwarded-For`.
- **`balancer-manager`** محدود.

### خلاصه فصل

- HAProxy استاندارد LB با active health check.
- `balance` الگوریتم را تعیین می‌کند.
- `check inter fall rise` health check را تنظیم می‌کند.
- `slowstart` سرور جدید را آرام اضافه می‌کند.
- `stats` برای مانیتورینگ.
- Apache `mod_proxy_balancer` گزینه‌ی داخلی است.

**سؤال مروری:** تفاوت `check` و `option httpchk` در HAProxy چیست و چرا دومی دقیق‌تر است؟

---

## فصل ۳۹: Sticky Session و اشتراک Session

### مقدمه

اگر session در حافظه‌ی سرور باشد، کاربر باید همیشه به همان سرور برود. این «sticky» است. ولی sticky، مشکل‌حل موقت است: با restart سرور، کاربر logout می‌شود. راه درست، session مشترک است.

### مفاهیم اساسی

**۱) سه مدل session**

| مدل | محل | sticky لازم؟ |
|---|---|---|
| در حافظه‌ی سرور | `w3wp`/Gunicorn | ✅ |
| در کوکی (امضاشده) | کلاینت | ❌ |
| در Redis/DB | مشترک | ❌ |

**۲) sticky با کوکی**

```nginx
upstream app {
    server 10.0.0.11:8000;
    server 10.0.0.12:8000;
    sticky cookie srv_id expires=1h path=/;
}
```

`sticky` ماژول Plus است. در OSS:

```nginx
upstream app {
    hash $cookie_sessionid consistent;
    server 10.0.0.11:8000;
    server 10.0.0.12:8000;
}
```

**۳) sticky با IP**

```nginx
upstream app {
    ip_hash;
    server 10.0.0.11:8000;
    server 10.0.0.12:8000;
}
```

مشکل: NAT، موبایل، IPv6.

**۴) HAProxy sticky**

```haproxy
backend app
    balance roundrobin
    cookie SRV insert indirect nocache
    server app1 10.0.0.11:8000 check cookie a
    server app2 10.0.0.12:8000 check cookie b
```

**۵) ARR affinity (IIS)**

```powershell
# در IIS Manager → ARR → Server Farms → lab-farm → Load Balance
# → Client Affinity: Cookie
```

**۶) session در Redis**

```python
# Flask
from flask_session import Session
import redis

app.config['SESSION_TYPE'] = 'redis'
app.config['SESSION_REDIS'] = redis.from_url('redis://127.0.0.1:6379')
Session(app)
```

```javascript
// Express
import session from 'express-session';
import RedisStore from 'connect-redis';
import { createClient } from 'redis';

const redis = createClient();
await redis.connect();

app.use(session({
  store: new RedisStore({ client: redis }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: { secure: true, httpOnly: true, sameSite: 'lax', maxAge: 3600_000 },
}));
```

**۷) JWT — session بی‌حالت**

```
Authorization: Bearer eyJhbGciOi...
```

مزیت: بدون sticky، بدون Redis.
عیب: revoke سخت، اندازه‌ی بزرگ، نیاز به چرخش کلید.

**۸) کوکی امن**

```javascript
cookie: {
  secure: true,        // فقط HTTPS
  httpOnly: true,      // بدون JS
  sameSite: 'lax',     // CSRF
  maxAge: 3600_000,
  domain: 'example.com',
  path: '/',
}
```

**۹) `session fixation`**

بعد از login، session id را عوض کن:

```python
session.regenerate()   # یا session.clear() + new
```

**۱۰) `session` در چند دامنه**

`domain=.example.com` برای زیردامنه‌ها. `SameSite` و `CORS` را هماهنگ کن.

### مثال‌های عملی

**Lab Scenario: sticky با Nginx و بعد Redis**

```bash
# اپ Flask با session در حافظه
pip install flask
```

```python
# app.py
from flask import Flask, session, jsonify
import os, socket

app = Flask(__name__)
app.secret_key = os.environ.get("SECRET", "dev-secret")

@app.get("/login")
def login():
    session["user"] = "alice"
    session["count"] = session.get("count", 0) + 1
    return jsonify(instance=socket.gethostname(), session=dict(session))

@app.get("/me")
def me():
    return jsonify(instance=socket.gethostname(), session=dict(session))
```

```bash
# دو نمونه روی پورت‌های مختلف
PORT=8000 python app.py &
PORT=8001 python app.py &
```

```nginx
upstream app {
    hash $cookie_session consistent;
    server 127.0.0.1:8000;
    server 127.0.0.1:8001;
}

server {
    listen 8080;
    location / {
        proxy_pass http://app;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        add_header X-Upstream $upstream_addr always;
    }
}
```

```bash
# login و نگه‌داشتن کوکی
curl -sS -c /tmp/cj -b /tmp/cj http://localhost:8080/login
curl -sS -c /tmp/cj -b /tmp/cj http://localhost:8080/me
curl -sS -c /tmp/cj -b /tmp/cj http://localhost:8080/me
# همه از یک instance (sticky)
```

**بدون sticky:**

```bash
# حذف hash
# upstream app { server ...; server ...; }

curl -sS -c /tmp/cj2 -b /tmp/cj2 http://localhost:8080/login
curl -sS -c /tmp/cj2 -b /tmp/cj2 http://localhost:8080/me
curl -sS -c /tmp/cj2 -b /tmp/cj2 http://localhost:8080/me
# ممکن است session خالی شود (instance دیگر)
```

**با Redis:**

```bash
sudo apt install -y redis-server
pip install flask-session redis
```

```python
from flask import Flask, session, jsonify
from flask_session import Session
import redis, os

app = Flask(__name__)
app.secret_key = os.environ["SECRET"]
app.config["SESSION_TYPE"] = "redis"
app.config["SESSION_REDIS"] = redis.from_url("redis://127.0.0.1:6379/0")
Session(app)

@app.get("/login")
def login():
    session["user"] = "alice"
    return jsonify(ok=True)

@app.get("/me")
def me():
    return jsonify(session=dict(session))
```

```bash
# بدون sticky، هر دو instance همان session را می‌بینند
curl -sS -c /tmp/cj3 -b /tmp/cj3 http://localhost:8080/login
curl -sS -c /tmp/cj3 -b /tmp/cj3 http://localhost:8080/me
curl -sS -c /tmp/cj3 -b /tmp/cj3 http://localhost:8080/me
```

**HAProxy sticky:**

```haproxy
backend app
    balance roundrobin
    cookie SRV insert indirect nocache
    option httpchk GET /health
    server app1 127.0.0.1:8000 check cookie a
    server app2 127.0.0.1:8001 check cookie b
```

```bash
curl -sSI http://localhost:8080/login | grep -i set-cookie
# Set-Cookie: SRV=a; path=/
```

**تست session fixation:**

```bash
# قبل از login
SID1=$(curl -sSI http://localhost:8080/ | grep -i set-cookie | sed 's/.*session=//;s/;.*//')

# بعد از login
SID2=$(curl -sSI -X POST http://localhost:8080/login | grep -i set-cookie | sed 's/.*session=//;s/;.*//')

[ "$SID1" != "$SID2" ] && echo "OK: rotated" || echo "FAIL: same session id"
```

### مقایسه و الگوهای مختلف

**سه مدل session**

| | حافظه | کوکی/JWT | Redis |
|---|---|---|---|
| sticky | ✅ لازم | ❌ | ❌ |
| revoke | ✅ | ❌ | ✅ |
| مقیاس | ❌ | ✅ | ✅ |
| state در سرور | ✅ | ❌ | ⚠️ |
| مناسب | تک‌سرور | API | ✅ عمومی |

**`ip_hash` در برابر `hash $cookie`**

| | `ip_hash` | `hash $cookie` |
|---|---|---|
| NAT | ❌ | ✅ |
| موبایل | ❌ | ✅ |
| سادگی | ✅ | ✅ |

**`cookie` در برابر `JWT`**

| | cookie session | JWT |
|---|---|---|
| revoke | ✅ | ⚠️ (blacklist) |
| اندازه | ✅ کوچک | ⚠️ بزرگ |
| CSRF | ⚠️ (نیاز به token) | ✅ (header) |
| مناسب | وب | API |

**دیدگاه «sticky کافی است»**: برای شروع بله، ولی با restart سرور، همه logout می‌شوند. Redis/DB راه درست است.

### اشکال شایع

**۱) sticky بدون `path=/`**
کوکی برای مسیر خاص. `path=/`.

**۲) `ip_hash` با NAT**
همه به یک سرور. `hash $cookie`.

**۳) session در حافظه با `least_conn`**
کاربر بین سرورها می‌پرد. sticky یا Redis.

**۴) کوکی بدون `secure`**
روی HTTP فرستاده می‌شود. `secure: true`.

**۵) کوکی بدون `httpOnly`**
XSS می‌تواند بخواند.

**۶) `sameSite: none` بدون `secure`**
مرورگر رد می‌کند.

**۷) session fixation**
id عوض نمی‌شود. `regenerate`.

**۸) Redis بدون رمز**
هر کسی می‌خواند. `requirepass` + شبکه‌ی خصوصی.

**۹) `maxAge` خیلی بلند**
session دزدیده‌شده، عمر بلند. ۱ ساعت.

**۱۰) `domain=.example.com` بدون کنترل زیردامنه**
زیردامنه‌ی آلوده، کوکی را می‌خواند.

### نکات مهم (Tips & Tricks)

- **Redis** برای session مشترک.
- **`hash $cookie`** به‌جای `ip_hash`.
- **`secure`/`httpOnly`/`sameSite`**.
- **`regenerate`** بعد از login.
- **`maxAge` کوتاه** + refresh.
- **`cookie SRV insert indirect`** در HAProxy.
- **بدون sticky با JWT**.

### خلاصه فصل

- sticky فقط وقتی session در حافظه است لازم می‌شود.
- `ip_hash` شکننده است؛ `hash $cookie` بهتر.
- Redis session را مشترک می‌کند.
- JWT بی‌حالت است، ولی revoke سخت.
- کوکی امن: `secure`، `httpOnly`، `sameSite`.
- session fixation با `regenerate` حل می‌شود.

**سؤال مروری:** چرا `ip_hash` برای کاربران پشت NAT بد است و چه جایگزینی داری؟

---

## فصل ۴۰: ARR در IIS و Load Balancer ابری

### مقدمه

در ویندوز، ARR سرور فارم می‌سازد. در ابر، ALB/NLB (AWS)، Application Gateway (Azure) و Cloud Load Balancing (GCP) همین کار را مدیریت‌شده انجام می‌دهند. این فصل، هر دو را با تمرکز بر تفاوت‌ها می‌گوید.

### مفاهیم اساسی

**۱) ARR — سرور فارم**

```
کلاینت → IIS + ARR → سرور فارم (چند IIS/اپ) → ...
```

```powershell
New-WebFarm -Name "lab-farm" -Servers @("10.0.0.11","10.0.0.12")
Add-WebFarmServer -Name "lab-farm" -ServerName "10.0.0.13"
Get-WebFarm
```

**۲) ARR — health check**

```powershell
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "webFarms/webFarm[@name='lab-farm']/applicationRequestRouting" `
  -Name "healthCheck.url" -Value "/health"

Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "webFarms/webFarm[@name='lab-farm']/applicationRequestRouting" `
  -Name "healthCheck.interval" -Value "00:00:10"
```

**۳) ARR — load balance algorithm**

```powershell
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "webFarms/webFarm[@name='lab-farm']/applicationRequestRouting" `
  -Name "loadBalancingAlgorithm" -Value "LeastCurrentRequest"
```

| الگوریتم | معنا |
|---|---|
| `WeightedRoundRobin` | نوبتی وزن‌دار |
| `LeastCurrentRequest` | کمترین درخواست فعال |
| `LeastResponseTime` | کمترین زمان پاسخ |

**۴) ARR — affinity**

```powershell
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "webFarms/webFarm[@name='lab-farm']/applicationRequestRouting" `
  -Name "clientAffinity.provider" -Value "Cookie"
```

**۵) AWS ALB — مفاهیم**

| مفهوم | معنا |
|---|---|
| Listener | پورت + پروتکل |
| Target Group | گروه مقصد با health check |
| Rule | مسیریابی بر اساس path/host/header |
| Stickiness | کوکی `AWSALB` |

**۶) AWS NLB در برابر ALB**

| | NLB | ALB |
|---|---|---|
| لایه | L4 | L7 |
| کارایی | ✅✅ | ✅ |
| مسیریابی path | ❌ | ✅ |
| TLS termination | ✅ | ✅ |
| IP ثابت | ✅ | ⚠️ |
| WebSocket | ✅ | ✅ |

**۷) Azure Application Gateway**

L7 با WAF، path-based routing، TLS termination، autoscaling.

**۸) GCP Cloud Load Balancing**

Global anycast، L4/L7، backend service با health check.

**۹) `X-Forwarded-For` در ابر**

ALB/AGW هدر را اضافه می‌کنند. سرور باید `set_real_ip_from` با CIDR آن‌ها داشته باشد.

**۱۰) `sticky` در ابر**

ALB: `stickiness.enabled=true`، `cookie.name=AWSALB`، `duration=1h`.
AGW: `CookieBasedAffinity`.

### مثال‌های عملی

**Lab Scenario: ARR با دو سرور**

```powershell
# ۱) نصب ARR و URL Rewrite (دستی از iis.net)

# ۲) فعال‌سازی proxy
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/proxy" -Name "enabled" -Value $true

# ۳) سرور فارم
New-WebFarm -Name "lab-farm" -Servers @("10.0.0.11","10.0.0.12")

# ۴) health check
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "webFarms/webFarm[@name='lab-farm']/applicationRequestRouting" `
  -Name "healthCheck.url" -Value "/health"
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "webFarms/webFarm[@name='lab-farm']/applicationRequestRouting" `
  -Name "healthCheck.interval" -Value "00:00:10"

# ۵) الگوریتم
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "webFarms/webFarm[@name='lab-farm']/applicationRequestRouting" `
  -Name "loadBalancingAlgorithm" -Value "LeastCurrentRequest"

# ۶) سایت با rewrite به فارم
New-Website -Name "lab-lb" -PhysicalPath C:\inetpub\empty `
  -Port 8080 -HostHeader "lab.example.com"
```

```xml
<!-- C:\inetpub\empty\web.config -->
<configuration>
  <system.webServer>
    <rewrite>
      <rules>
        <rule name="LB" stopProcessing="true">
          <match url="(.*)" />
          <action type="Rewrite" url="http://lab-farm/{R:1}" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
```

```powershell
# ۷) تست
1..10 | ForEach-Object {
  (Invoke-WebRequest http://lab.example.com:8080/ -UseBasicParsing).Content
}

# ۸) وضعیت
Get-WebFarm
Get-WebFarmServer -Name "lab-farm"
```

**AWS ALB با Terraform (نمونه):**

```hcl
resource "aws_lb" "lab" {
  name               = "lab-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.lb.id]
  subnets            = var.public_subnets
}

resource "aws_lb_target_group" "app" {
  name     = "lab-app"
  port     = 8000
  protocol = "HTTP"
  vpc_id   = var.vpc_id

  health_check {
    path                = "/health"
    interval            = 15
    timeout             = 5
    healthy_threshold   = 2
    unhealthy_threshold = 3
    matcher             = "200"
  }

  stickiness {
    type            = "lb_cookie"
    enabled         = true
    cookie_duration = 3600
  }
}

resource "aws_lb_listener" "https" {
  load_balancer_arn = aws_lb.lab.arn
  port              = 443
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-TLS13-1-2-2021-06"
  certificate_arn   = var.cert_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}

resource "aws_lb_listener_rule" "api" {
  listener_arn = aws_lb_listener.https.arn
  priority     = 10

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.api.arn
  }

  condition {
    path_pattern {
      values = ["/api/*"]
    }
  }
}
```

**Nginx پشت ALB:**

```nginx
# ALB در subnet 10.0.0.0/16
set_real_ip_from 10.0.0.0/16;
real_ip_header X-Forwarded-For;
real_ip_recursive on;
```

**تست health check از دید ALB:**

```bash
# باید سریع و سبک باشد
curl -sS -o /dev/null -w '%{http_code} %{time_total}\n' https://app.example.com/health
# 200 0.003
```

**مقایسه‌ی هزینه/پیچیدگی:**

```bash
# NLB: ارزان، L4
# ALB: گران‌تر، L7، path routing
# Nginx روی EC2: کنترل کامل، نگهداری
```

### مقایسه و الگوهای مختلف

**ARR در برابر Nginx در برابر ALB**

| | ARR | Nginx | ALB |
|---|---|---|---|
| مدیریت | ✅ GUI | ⚠️ | ✅ |
| path routing | ⚠️ | ✅ | ✅ |
| health check | ✅ | ❌ (OSS) | ✅ |
| WAF | ⚠️ | ⚠️ | ✅ |
| هزینه | 💰 (ویندوز) | ۰ | 💰 |
| مناسب | ویندوز | همه‌جا | ابر AWS |

**NLB در برابر ALB**

| | NLB | ALB |
|---|---|---|
| لایه | 4 | 7 |
| path | ❌ | ✅ |
| WebSocket | ✅ | ✅ |
| IP ثابت | ✅ | ❌ |
| کارایی | ✅✅ | ✅ |

**سه مدل TLS در LB**

| مدل | توضیح |
|---|---|
| Termination | LB رمزگشایی، HTTP به backend |
| Passthrough | LB فقط TCP، TLS در backend |
| Re-encryption | LB رمزگشایی و دوباره رمزنگاری |

**دیدگاه «ALB برای همه‌چیز»**: ALB برای HTTP عالی است. برای TCP خام (database، MQ)، NLB.

### اشکال شایع

**۱) ARR بدون `proxy enabled`**
`502`. `Set-WebConfigurationProperty`.

**۲) health check به `/` که ۳۰۱ می‌دهد**
سرور سالم، unhealthy. `/health` با `200`.

**۳) `X-Forwarded-For` بدون `real_ip`**
لاگ IP اشتباه.

**۴) ALB و `Host` هدر**
ALB `Host` را حفظ می‌کند، ولی Nginx باید `proxy_set_header Host $host`.

**۵) sticky با `lb_cookie` و session در Redis**
دو لایه، بی‌دلیل. یکی کافی است.

**۶) `target group` با پورت اشتباه**
`502`. بررسی `port`.

**۷) health check interval خیلی کم**
بار اضافه. ۱۵–۳۰ ثانیه.

**۸) `deregistration delay` صفر**
قطع درخواست فعال. ۳۰ ثانیه.

**۹) ARR و WebSocket**
`webSocket enabled` + timeout.

**۱۰) `cross-zone load balancing` خاموش**
توزیع نامتوازن. `on`.

### نکات مهم (Tips & Tricks)

- **`/health` سبک** برای همه‌ی LBها.
- **`set_real_ip_from`** با CIDR ابر.
- **`deregistration delay`** ۳۰s.
- **`cross-zone`** روشن.
- **ARR: `LeastCurrentRequest`**.
- **ALB: `stickiness`** فقط اگر لازم.
- **`haproxy -c`**/`nginx -t` قبل از تغییر.

### خلاصه فصل

- ARR سرور فارم در IIS می‌سازد.
- ALB L7 با path routing، NLB L4 با کارایی بالا.
- health check سبک و واقعی.
- `X-Forwarded-For` نیاز به `real_ip` دارد.
- `deregistration delay` برای restart بی‌درد.
- sticky و session مشترک را با هم انتخاب کن، نه هر دو.

**سؤال مروری:** چه زمانی NLB را به ALB ترجیح می‌دهی؟

---

## منابع و مراجع

**مستندات رسمی**
- [Nginx — ngx_http_upstream_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html)
- [HAProxy Configuration Manual](https://docs.haproxy.org/2.9/configuration.html)
- [Apache — mod_proxy_balancer](https://httpd.apache.org/docs/2.4/mod/mod_proxy_balancer.html)
- [Apache — mod_proxy_hcheck](https://httpd.apache.org/docs/2.4/mod/mod_proxy_hcheck.html)
- [IIS ARR — Server Farms](https://learn.microsoft.com/en-us/iis/extensions/configuring-application-request-routing-arr/creating-a-forward-proxy-using-application-request-routing)
- [AWS ALB](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/)
- [AWS NLB](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/)

**آموزش**
- [HAProxy — best practices](https://www.haproxy.com/blog/the-four-essential-sections-of-an-haproxy-configuration)
- [Load balancing algorithms explained](https://www.nginx.com/blog/choosing-the-right-load-balancing-algorithm/)
- [Session management patterns](https://12factor.net/processes)

**ابزار**
- [haproxyctl](https://github.com/FlorentinDUBOIS/haproxyctl)
- [hey](https://github.com/rakyll/hey)
- [vegeta](https://github.com/tsenart/vegeta)

---

**بعدی:** [`10-TLS-and-Certificates.md`](10-TLS-and-Certificates.md) — TLS 1.3، ACME، HSTS، mTLS و تست
