# بخش ۱۸: پروژه‌ی کامل — استقرار یک سرویس واقعی

**مدت یادگیری**: ۶ تا ۸ ساعت
**سطح سختی**: پروژه (جمع‌بندی همه‌ی بخش‌ها)
**پیش‌نیاز**: بخش ۰۱ تا ۱۷

---

## فهرست محتوا

- [فصل ۷۶: سناریو، نیازمندی و معماری](#فصل-۷۶)
- [فصل ۷۷: پیاده‌سازی گام‌به‌گام](#فصل-۷۷)
- [فصل ۷۸: امنیت و بهینه‌سازی](#فصل-۷۸)
- [فصل ۷۹: مانیتورینگ، لاگ و بکاپ](#فصل-۷۹)
- [فصل ۸۰: تست بار، تحویل و چک‌لیست](#فصل-۸۰)

---

## فصل ۷۶: سناریو، نیازمندی و معماری

### مقدمه

این پروژه، همه‌ی تکه‌ها را کنار هم می‌گذارد: یک سرویس وب واقعی با Nginx به‌عنوان لبه، یک API، یک SPA، TLS، کش، rate limit، مانیتورینگ و استقرار قابل تکرار. هدف: «می‌توانم از صفر، یک سرویس آماده‌ی تولید بالا بیاورم».

### مفاهیم اساسی

**۱) سناریو**

> یک فروشگاه کوچک می‌خواهد سرویس «وضعیت سفارش» را آنلاین کند. کاربران با موبایل وارد می‌شوند، سفارش را پیگیری می‌کنند، و گاهی فایل فاکتور دانلود می‌کنند. ترافیک: ۵۰ درخواست/ثانیه در اوج، ۵۰۰ هزار بازدید ماهانه.

**۲) نیازمندی‌ها**

| # | نیازمندی | معیار |
|---|---|---|
| ۱ | HTTPS اجباری | `A` در SSL Labs |
| ۲ | SPA + API | مسیر جدا، بدون تداخل |
| ۳ | API عمومی کش‌شده | `HIT` بالای ۵۰٪ |
| ۴ | API خصوصی محافظت‌شده | بدون کش، rate limit |
| ۵ | دانلود فاکتور | فقط کاربر مالک |
| ۶ | پایداری | بدون `5xx` در تست بار |
| ۷ | مشاهده‌پذیری | لاگ + متریک + هشدار |
| ۸ | استقرار | اسکریپت تک‌دستوری |
| ۹ | بکاپ | روزانه + تست بازیابی |

**۳) معماری**

```
                    ┌───────────────────────────────┐
   کاربر (موبایل)    │  CDN (اختیاری)                │
        │           └───────────────┬───────────────┘
        │                           │
        ▼                           ▼
┌─────────────────────────────────────────────────────────┐
│  web01 — Nginx 1.26                                     │
│  ├── :80   → redirect 443                               │
│  ├── :443  → TLS, HTTP/2, gzip, security headers        │
│  ├── /            → SPA (static, fallback)              │
│  ├── /assets/     → static, immutable cache             │
│  ├── /api/public/ → proxy + micro-cache 5s              │
│  ├── /api/        → proxy + rate limit                  │
│  ├── /internal/   → X-Accel-Redirect (فاکتور)           │
│  └── /health      → 200                                 │
└──────────────┬──────────────────────────────────────────┘
               │ 127.0.0.1:8000
               ▼
┌─────────────────────────────────────────────────────────┐
│  app — FastAPI + Uvicorn (systemd)                      │
│  ├── /health (DB + Redis)                               │
│  ├── /public/stats                                      │
│  ├── /orders/{id}  (auth)                               │
│  └── /invoices/{id} → X-Accel-Redirect                  │
└──────────────┬──────────────────────────────────────────┘
               │
       ┌───────┴────────┐
       ▼                ▼
┌────────────┐   ┌────────────┐
│ PostgreSQL │   │  Redis     │
└────────────┘   └────────────┘
```

**۴) تصمیم‌های کلیدی**

| تصمیم | انتخاب | چرا |
|---|---|---|
| لبه | Nginx | استاتیک، پروکسی، کش |
| اپ | FastAPI + Uvicorn | async، ساده |
| session | JWT + Redis (revoke) | بدون sticky |
| کش API عمومی | `proxy_cache` ۵ ثانیه | کاهش بار |
| فاکتور | `X-Accel-Redirect` | فایل از Nginx، دسترسی کنترل‌شده |
| TLS | Let's Encrypt + `certbot` | رایگان، خودکار |
| استقرار | systemd + اسکریپت | ساده، بدون K8s |
| مانیتورینگ | `stub_status` + `pg_stat` + `prometheus` | کافی |

**۵) ساختار فایل**

```
/srv/lab/
├── app/
│   ├── venv/
│   ├── app.py
│   └── requirements.txt
├── spa/
│   ├── index.html
│   └── assets/
├── private/
│   └── invoices/
├── nginx/
│   ├── nginx.conf
│   ├── conf.d/
│   └── snippets/
├── scripts/
│   ├── deploy.sh
│   ├── backup.sh
│   └── healthcheck.sh
└── .env
```

**۶) پیش‌نیاز محیط**

| مورد | نسخه |
|---|---|
| Ubuntu | 24.04 |
| Nginx | 1.26 |
| Python | 3.12 |
| PostgreSQL | 16+ |
| Redis | 7 |
| certbot | 2.x |

### مثال‌های عملی

**Lab Scenario: آماده‌سازی VM**

```bash
# ۱) به‌روزرسانی
sudo apt update && sudo apt upgrade -y

# ۲) بسته‌ها
sudo apt install -y nginx python3-venv python3-pip \
  postgresql redis-server certbot python3-certbot-nginx \
  curl jq ufw fail2ban

# ۳) کاربر اپ
sudo useradd --system --home /srv/lab --shell /usr/sbin/nologin lab || true

# ۴) پوشه‌ها
sudo mkdir -p /srv/lab/{app,spa/assets,private/invoices,nginx/conf.d,nginx/snippets,scripts}
sudo chown -R lab:lab /srv/lab

# ۵) فایروال
sudo ufw default deny incoming
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw --force enable
sudo ufw status verbose

# ۶) بررسی
nginx -v
python3 -V
psql --version
redis-cli ping
```

**۶) `hosts` برای آزمایشگاه:**

```bash
echo "192.168.56.10  lab.example.com" | sudo tee -a /etc/hosts
```

### مقایسه و الگوهای مختلف

**تک‌سرور در برابر چندسرور**

| | تک‌سرور | چندسرور |
|---|---|---|
| پیچیدگی | ✅ | ❌ |
| HA | ❌ | ✅ |
| مناسب | MVP | ✅ تولید |

**systemd در برابر Docker در برابر K8s**

| | systemd | Docker | K8s |
|---|---|---|---|
| سادگی | ✅✅ | ✅ | ❌ |
| بازتولید | ⚠️ | ✅ | ✅ |
| مقیاس | ❌ | ⚠️ | ✅ |
| انتخاب این پروژه | ✅ | — | — |

**`proxy_cache` در برابر Redis**

| | `proxy_cache` | Redis |
|---|---|---|
| لایه | HTTP | اپ |
| سادگی | ✅ | ⚠️ |
| مناسب | پاسخ کامل | داده‌ی ساختاریافته |

**دیدگاه «پروژه باید K8s باشد»**: برای این مقیاس، systemd ساده‌تر و قابل‌اعتمادتر است. K8s پیچیدگی اضافه می‌کند بدون سود.

### اشکال شایع

**۱) شروع با K8s برای ۵۰ rps**
پیچیدگی بی‌دلیل.

**۲) نبود `hosts`/DNS**
تست محلی، گواهی شکست.

**۳) `ufw` قبل از SSH**
قفل شدن بیرون. اول ۲۲.

**۴) پوشه با مالک `root`**
`403`. `chown lab`.

**۵) نبود `venv`**
تعارض بسته. `python3 -m venv`.

**۶) `redis` بدون رمز و روی `0.0.0.0`**
خطر. `127.0.0.1`.

**۷) `postgres` با رمز پیش‌فرض**
خطر.

**۸) نبود `.env`**
رمز در کد.

### نکات مهم (Tips & Tricks)

- **`ufw` اول SSH**.
- **`/srv/lab`** برای همه‌چیز.
- **`venv`** برای Python.
- **`127.0.0.1`** برای Redis/Postgres.
- **`.env`** برای رمز.
- **`hosts`** برای آزمایشگاه.
- **systemd** برای پایداری.

### خلاصه فصل

- سناریو: SPA + API + دانلود + TLS.
- Nginx لبه، FastAPI اپ، Postgres/Redis داده.
- `X-Accel-Redirect` برای فایل خصوصی.
- کش ۵ ثانیه برای API عمومی.
- systemd برای استقرار ساده.
- امنیت از روز اول.

**سؤال مروری:** چرا برای فاکتور، `X-Accel-Redirect` بهتر از خواندن فایل در اپ است؟

---

## فصل ۷۷: پیاده‌سازی گام‌به‌گام

### مقدمه

این فصل، کد و config کامل و قابل اجراست. ترتیب: داده → اپ → Nginx → TLS.

### مفاهیم اساسی

**۱) ترتیب**

```
۱. PostgreSQL (DB + schema)
۲. Redis
۳. اپ (FastAPI)
۴. systemd
۵. Nginx (HTTP)
۶. TLS
۷. SPA
```

**۲) `requirements.txt`**

```
fastapi==0.115.0
uvicorn[standard]==0.30.6
gunicorn==22.0.0
psycopg[binary,pool]==3.2.3
redis==5.0.8
python-jose[cryptography]==3.3.0
pydantic-settings==2.5.2
```

**۳) `.env`**

```ini
DATABASE_URL=postgresql://lab:CHANGE_ME@127.0.0.1:5432/lab
REDIS_URL=redis://127.0.0.1:6379/0
JWT_SECRET=CHANGE_ME_LONG_RANDOM
JWT_ALG=HS256
INVOICE_DIR=/srv/lab/private/invoices
```

**۴) schema**

```sql
CREATE TABLE customers (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email TEXT NOT NULL UNIQUE,
    full_name TEXT NOT NULL
);

CREATE TABLE orders (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id BIGINT NOT NULL REFERENCES customers(id),
    status TEXT NOT NULL DEFAULT 'pending'
        CHECK (status IN ('pending','paid','shipped','delivered','cancelled')),
    total NUMERIC(12,2) NOT NULL CHECK (total >= 0),
    placed_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX ix_orders_customer ON orders (customer_id, placed_at DESC);

CREATE TABLE invoices (
    order_id BIGINT PRIMARY KEY REFERENCES orders(id),
    file_name TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

**۵) اپ — ساختار**

```python
# app.py
from fastapi import FastAPI, HTTPException, Depends, Header
from fastapi.responses import JSONResponse
...
```

**۶) `X-Accel-Redirect`**

```python
return Response(
    headers={
        "X-Accel-Redirect": f"/internal/invoices/{name}",
        "Content-Disposition": f'attachment; filename="{name}"',
    }
)
```

**۷) Nginx — `location /internal/`**

```nginx
location /internal/invoices/ {
    internal;
    alias /srv/lab/private/invoices/;
    add_header Content-Disposition "attachment" always;
}
```

**۸) systemd**

```ini
[Service]
User=lab
WorkingDirectory=/srv/lab/app
EnvironmentFile=/srv/lab/.env
ExecStart=/srv/lab/app/venv/bin/gunicorn app:app \
  -k uvicorn.workers.UvicornWorker -w 4 -b 127.0.0.1:8000 --timeout 30
Restart=always
```

**۹) SPA**

```html
<!-- /srv/lab/spa/index.html -->
<!doctype html>
<html lang="fa" dir="rtl">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>پیگیری سفارش</title>
  <link rel="stylesheet" href="/assets/app.css">
</head>
<body>
  <div id="app">در حال بارگذاری…</div>
  <script src="/assets/app.js"></script>
</body>
</html>
```

**۱۰) Nginx — fallback**

```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

### مثال‌های عملی

**Lab Scenario: کد و config کامل**

**۱) دیتابیس:**

```bash
sudo -u postgres psql <<'SQL'
CREATE ROLE lab WITH LOGIN PASSWORD 'CHANGE_ME';
CREATE DATABASE lab OWNER lab ENCODING 'UTF8' LC_COLLATE 'C.UTF-8' LC_CTYPE 'C.UTF-8' TEMPLATE template0;
SQL

sudo -u postgres psql -d lab <<'SQL'
CREATE TABLE customers (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email TEXT NOT NULL UNIQUE,
    full_name TEXT NOT NULL
);

CREATE TABLE orders (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id BIGINT NOT NULL REFERENCES customers(id),
    status TEXT NOT NULL DEFAULT 'pending'
        CHECK (status IN ('pending','paid','shipped','delivered','cancelled')),
    total NUMERIC(12,2) NOT NULL CHECK (total >= 0),
    placed_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX ix_orders_customer ON orders (customer_id, placed_at DESC);

CREATE TABLE invoices (
    order_id BIGINT PRIMARY KEY REFERENCES orders(id),
    file_name TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

INSERT INTO customers (email, full_name) VALUES
  ('maryam@example.com', 'مریم رضایی'),
  ('ali@example.com',    'علی کریمی');

INSERT INTO orders (customer_id, status, total) VALUES
  (1, 'paid', 2500000),
  (1, 'shipped', 1800000),
  (2, 'pending', 900000);
SQL
```

**۲) Redis:**

```bash
sudo sed -i 's/^# requirepass .*/requirepass CHANGE_ME/' /etc/redis/redis.conf
sudo sed -i 's/^bind .*/bind 127.0.0.1 ::1/' /etc/redis/redis.conf
sudo systemctl restart redis-server
redis-cli -a CHANGE_ME ping
```

**۳) اپ:**

```bash
sudo -u lab bash -c '
cd /srv/lab/app
python3 -m venv venv
./venv/bin/pip install -q -r requirements.txt
'
```

```python
# /srv/lab/app/app.py
from __future__ import annotations

import os
from datetime import datetime, timedelta, timezone
from pathlib import Path

import psycopg
from fastapi import Depends, FastAPI, Header, HTTPException
from fastapi.responses import JSONResponse, Response
from jose import JWTError, jwt
from psycopg.rows import dict_row
from psycopg_pool import ConnectionPool
from pydantic import BaseModel
from redis import Redis

DATABASE_URL = os.environ["DATABASE_URL"]
REDIS_URL = os.environ["REDIS_URL"]
JWT_SECRET = os.environ["JWT_SECRET"]
JWT_ALG = os.environ.get("JWT_ALG", "HS256")
INVOICE_DIR = Path(os.environ.get("INVOICE_DIR", "/srv/lab/private/invoices"))

pool = ConnectionPool(DATABASE_URL, min_size=2, max_size=10, open=True)
redis = Redis.from_url(REDIS_URL, decode_responses=True)

app = FastAPI(title="Lab Shop API", docs_url=None, redoc_url=None)


class LoginIn(BaseModel):
    email: str


def make_token(email: str) -> str:
    payload = {
        "sub": email,
        "exp": datetime.now(timezone.utc) + timedelta(hours=8),
        "iat": datetime.now(timezone.utc),
    }
    return jwt.encode(payload, JWT_SECRET, algorithm=JWT_ALG)


def current_user(authorization: str | None = Header(default=None)) -> str:
    if not authorization or not authorization.startswith("Bearer "):
        raise HTTPException(401, "missing token")
    token = authorization.removeprefix("Bearer ").strip()
    try:
        payload = jwt.decode(token, JWT_SECRET, algorithms=[JWT_ALG])
    except JWTError:
        raise HTTPException(401, "invalid token")
    if redis.get(f"revoked:{token}"):
        raise HTTPException(401, "revoked token")
    return payload["sub"]


@app.get("/health")
def health():
    try:
        with pool.connection() as conn:
            conn.execute("SELECT 1")
        redis.ping()
    except Exception as exc:
        return JSONResponse({"status": "degraded", "error": str(exc)}, status_code=503)
    return {"status": "ok"}


@app.post("/login")
def login(body: LoginIn):
    with pool.connection() as conn:
        row = conn.execute(
            "SELECT id, email, full_name FROM customers WHERE email = %s",
            (body.email,),
        ).fetchone()
    if not row:
        raise HTTPException(404, "customer not found")
    return {"token": make_token(body.email), "name": row[2]}


@app.post("/logout")
def logout(authorization: str | None = Header(default=None)):
    if authorization and authorization.startswith("Bearer "):
        token = authorization.removeprefix("Bearer ").strip()
        redis.setex(f"revoked:{token}", 8 * 3600, "1")
    return {"ok": True}


@app.get("/public/stats")
def stats():
    """عمومی — کش می‌شود."""
    with pool.connection() as conn:
        row = conn.execute(
            "SELECT count(*) AS orders, coalesce(sum(total), 0) AS revenue FROM orders"
        ).fetchone()
    return {"orders": row[0], "revenue": str(row[1])}


@app.get("/orders")
def my_orders(user: str = Depends(current_user)):
    with pool.connection() as conn:
        with conn.cursor(row_factory=dict_row) as cur:
            cur.execute(
                """
                SELECT o.id, o.status, o.total, o.placed_at
                FROM orders o
                JOIN customers c ON c.id = o.customer_id
                WHERE c.email = %s
                ORDER BY o.placed_at DESC
                """,
                (user,),
            )
            return cur.fetchall()


@app.get("/invoices/{order_id}")
def invoice(order_id: int, user: str = Depends(current_user)):
    with pool.connection() as conn:
        row = conn.execute(
            """
            SELECT i.file_name
            FROM invoices i
            JOIN orders o ON o.id = i.order_id
            JOIN customers c ON c.id = o.customer_id
            WHERE i.order_id = %s AND c.email = %s
            """,
            (order_id, user),
        ).fetchone()

    if not row:
        raise HTTPException(404, "invoice not found")

    name = row[0]
    if not (INVOICE_DIR / name).is_file():
        raise HTTPException(404, "file missing")

    # فایل را Nginx می‌فرستد
    return Response(
        status_code=200,
        headers={
            "X-Accel-Redirect": f"/internal/invoices/{name}",
            "Content-Disposition": f'attachment; filename="{name}"',
            "Content-Type": "application/pdf",
        },
    )
```

```bash
# فایل نمونه
echo '%PDF-1.4 fake' | sudo tee /srv/lab/private/invoices/inv-1.pdf
sudo chown -R lab:lab /srv/lab/private
```

**۴) systemd:**

```ini
# /etc/systemd/system/lab-api.service
[Unit]
Description=Lab Shop API
After=network-online.target postgresql.service redis-server.service
Wants=network-online.target

[Service]
Type=simple
User=lab
Group=lab
WorkingDirectory=/srv/lab/app
EnvironmentFile=/srv/lab/.env
ExecStart=/srv/lab/app/venv/bin/gunicorn app:app \
    -k uvicorn.workers.UvicornWorker \
    -w 4 -b 127.0.0.1:8000 \
    --timeout 30 --graceful-timeout 30 \
    --access-logfile - --error-logfile -
Restart=always
RestartSec=3
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ReadWritePaths=/srv/lab/private

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now lab-api
sudo systemctl status lab-api --no-pager
curl -sS http://127.0.0.1:8000/health
```

**۵) Nginx — config کامل:**

```nginx
# /etc/nginx/snippets/ssl.conf
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305;
ssl_prefer_server_ciphers off;
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 1d;
ssl_session_tickets off;
ssl_stapling on;
ssl_stapling_verify on;
resolver 1.1.1.1 8.8.8.8 valid=300s;
resolver_timeout 5s;
```

```nginx
# /etc/nginx/snippets/security-headers.conf
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
```

```nginx
# /etc/nginx/conf.d/ratelimit.conf
limit_req_zone $binary_remote_addr zone=api:10m rate=20r/s;
limit_req_zone $binary_remote_addr zone=auth:10m rate=5r/m;
limit_conn_zone $binary_remote_addr zone=conn:10m;

proxy_cache_path /var/cache/nginx/public levels=1:2 keys_zone=public:16m
                 max_size=200m inactive=10m use_temp_path=off;
```

```nginx
# /etc/nginx/conf.d/lab.conf
upstream lab_api {
    server 127.0.0.1:8000;
    keepalive 32;
}

server {
    listen 80;
    server_name lab.example.com;

    location ^~ /.well-known/acme-challenge/ {
        root /var/www/certbot;
        default_type text/plain;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    http2 on;
    server_name lab.example.com;

    ssl_certificate     /etc/letsencrypt/live/lab.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/lab.example.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/lab.example.com/chain.pem;

    include snippets/ssl.conf;
    include snippets/security-headers.conf;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    root /srv/lab/spa;
    index index.html;
    server_tokens off;
    client_max_body_size 8m;

    # health
    location = /health {
        access_log off;
        proxy_pass http://lab_api/health;
    }

    # API عمومی با micro-cache
    location /api/public/ {
        limit_req zone=api burst=40 nodelay;

        proxy_cache public;
        proxy_cache_key "$scheme$request_method$host$request_uri";
        proxy_cache_valid 200 5s;
        proxy_cache_lock on;
        proxy_cache_use_stale error timeout updating;
        proxy_cache_background_update on;
        add_header X-Cache-Status $upstream_cache_status always;

        rewrite ^/api/(.*)$ /$1 break;
        proxy_pass http://lab_api;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # login — محدودیت سخت
    location = /api/login {
        limit_req zone=auth burst=3;

        rewrite ^/api/(.*)$ /$1 break;
        proxy_pass http://lab_api;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # API خصوصی
    location /api/ {
        limit_req zone=api burst=40 nodelay;
        limit_conn conn 20;

        rewrite ^/api/(.*)$ /$1 break;
        proxy_pass http://lab_api;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Authorization $http_authorization;

        proxy_read_timeout 30s;
        proxy_connect_timeout 5s;
    }

    # فایل خصوصی
    location /internal/invoices/ {
        internal;
        alias /srv/lab/private/invoices/;
        add_header Content-Disposition "attachment" always;
        add_header X-Content-Type-Options "nosniff" always;
    }

    # استاتیک
    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, max-age=31536000, immutable" always;
        access_log off;
        try_files $uri =404;
    }

    # HTML
    location = /index.html {
        add_header Cache-Control "no-cache" always;
    }

    # SPA
    location / {
        try_files $uri $uri/ /index.html;
    }

    # فایل‌های مخفی
    location ~ /\.(?!well-known) {
        deny all;
        access_log off;
        log_not_found off;
    }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx
```

**۶) TLS:**

```bash
sudo mkdir -p /var/www/certbot
sudo certbot certonly --webroot -w /var/www/certbot \
  -d lab.example.com --agree-tos -m admin@example.com --no-eff-email
sudo nginx -t && sudo systemctl reload nginx
curl -sSI https://lab.example.com/health | head -3
```

**۷) SPA:**

```bash
sudo tee /srv/lab/spa/index.html >/dev/null <<'HTML'
<!doctype html>
<html lang="fa" dir="rtl">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>پیگیری سفارش</title>
<link rel="stylesheet" href="/assets/app.css">
</head>
<body>
<div id="app">در حال بارگذاری…</div>
<script src="/assets/app.js"></script>
</body>
</html>
HTML

sudo tee /srv/lab/spa/assets/app.css >/dev/null <<'CSS'
:root{--bg:#0f1115;--fg:#e8e8ea;--accent:#4f8cff}
*{box-sizing:border-box}
body{margin:0;font-family:system-ui,Tahoma,sans-serif;background:var(--bg);color:var(--fg)}
#app{max-width:720px;margin:48px auto;padding:24px}
button{background:var(--accent);border:0;color:#fff;padding:10px 16px;border-radius:8px;cursor:pointer}
.card{border:1px solid #2a2f3a;border-radius:12px;padding:16px;margin:12px 0}
CSS

sudo tee /srv/lab/spa/assets/app.js >/dev/null <<'JS'
const app = document.getElementById('app');
const api = async (p, opt = {}) => {
  const r = await fetch('/api' + p, opt);
  if (!r.ok) throw new Error(r.status);
  return r.json();
};

(async () => {
  try {
    const s = await api('/public/stats');
    app.innerHTML = `<h1>فروشگاه</h1>
      <div class="card">سفارش‌ها: ${s.orders}</div>
      <div class="card">درآمد: ${Number(s.revenue).toLocaleString('fa-IR')} ریال</div>`;
  } catch (e) {
    app.textContent = 'خطا: ' + e.message;
  }
})();
JS

sudo chown -R lab:lab /srv/lab/spa
```

```bash
curl -sS https://lab.example.com/ | head -5
curl -sS https://lab.example.com/api/public/stats
```

### مقایسه و الگوهای مختلف

**`rewrite ... break` در برابر `proxy_pass` با URI**

| | `rewrite break` | `proxy_pass URI` |
|---|---|---|
| خوانایی | ⚠️ | ✅ |
| کنترل | ✅ | ⚠️ |
| مناسب | مسیر پیچیده | ساده |

**`gunicorn` در برابر `uvicorn` مستقیم**

| | gunicorn | uvicorn |
|---|---|---|
| workers | ✅ | ⚠️ |
| graceful | ✅ | ✅ |
| توصیه | ✅ تولید | توسعه |

**`systemd` در برابر `supervisor`**

| | systemd | supervisor |
|---|---|---|
| بومی | ✅ | ❌ |
| لاگ | journal | فایل |
| توصیه | ✅ | legacy |

**دیدگاه «همه‌چیز در Docker»**: برای این مقیاس، systemd + venv ساده‌تر است. Docker برای بازتولید و CI.

### اشکال شایع

**۱) `proxy_pass` با `/` اضافه**
مسیر دوبار. `rewrite break`.

**۲) `X-Accel-Redirect` و `alias` بدون `/`**
`404`. `/` پایانی.

**۳) `gunicorn` بدون `--timeout`**
worker گیر. `30`.

**۴) `systemd` بدون `EnvironmentFile`**
رمز نیست. `EnvironmentFile`.

**۵) `ProtectSystem=strict` بدون `ReadWritePaths`**
اپ نمی‌نویسد. `ReadWritePaths`.

**۶) `limit_req` روی `/api/public/`**
کش و rate limit هم‌زمان، پیچیده. مراقب.

**۷) `JWT` بدون revoke**
logout کار نمی‌کند. Redis.

**۸) `invoices` بدون بررسی مالکیت**
IDOR. `JOIN` با `email`.

**۹) `root` به‌جای `alias` برای `/internal/`**
مسیر غلط.

**۱۰) `certbot` قبل از Nginx**
`404` چالش. `webroot`.

### نکات مهم (Tips & Tricks)

- **`rewrite ... break`** برای مسیر API.
- **`X-Accel-Redirect`** + `alias` با `/`.
- **`EnvironmentFile`**.
- **`ReadWritePaths`**.
- **بررسی مالکیت** در فایل خصوصی.
- **`proxy_cache_lock`**.
- **`health`** با DB + Redis.

### خلاصه فصل

- DB → Redis → اپ → systemd → Nginx → TLS.
- `rewrite break` مسیر را کنترل می‌کند.
- `X-Accel-Redirect` فایل را از Nginx می‌فرستد.
- `systemd` با `EnvironmentFile` و `ProtectSystem`.
- کش و rate limit در `location` جدا.
- بررسی مالکیت برای داده‌ی خصوصی.

**سؤال مروری:** چرا `X-Accel-Redirect` باید در `location` با `internal` باشد؟

---

## فصل ۷۸: امنیت و بهینه‌سازی

### مقدمه

سیستم کار می‌کند. حالا باید «امن» و «سریع» شود. این فصل، سخت‌سازی و تنظیم.

### مفاهیم اساسی

**۱) چک‌لیست امنیتی**

| مورد | وضعیت |
|---|---|
| `server_tokens off` | ✅ |
| TLS 1.2+ | ✅ |
| HSTS | ✅ |
| هدرهای امنیتی | ✅ |
| rate limit | ✅ |
| `fail2ban` | ⬜ |
| `.git`/`.env` بسته | ✅ |
| `admin` محدود | ⬜ |
| `modsecurity` | ⬜ |
| `unattended-upgrades` | ⬜ |

**۲) `fail2ban`**

```ini
[nginx-limit-req]
enabled = true
filter  = nginx-limit-req
logpath = /var/log/nginx/error.log
maxretry = 5
findtime = 600
bantime  = 3600
```

**۳) `unattended-upgrades`**

```bash
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

**۴) بهینه‌سازی Nginx**

```nginx
worker_processes auto;
worker_rlimit_nofile 65535;

events {
    worker_connections 16384;
    multi_accept on;
}

http {
    sendfile on;
    sendfile_max_chunk 1m;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    keepalive_requests 1000;
    client_body_buffer_size 16k;
    large_client_header_buffers 4 8k;
    open_file_cache max=10000 inactive=60s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;
    gzip on;
    gzip_vary on;
    gzip_comp_level 6;
    gzip_min_length 1024;
    gzip_static on;
}
```

**۵) `gzip_static`**

```bash
find /srv/lab/spa/assets -type f \( -name '*.css' -o -name '*.js' \) \
  -exec gzip -9 -k -f {} \;
```

**۶) کش استاتیک**

```nginx
location /assets/ {
    expires 1y;
    add_header Cache-Control "public, max-age=31536000, immutable" always;
}
```

> نام فایل باید hash داشته باشد. در build، `app.[hash].js`.

**۷) `PgBouncer` (اختیاری)**

اگر اتصال زیاد شد:

```ini
[databases]
lab = host=127.0.0.1 port=5432 dbname=lab

[pgbouncer]
listen_addr = 127.0.0.1
listen_port = 6432
auth_type = scram-sha-256
pool_mode = transaction
default_pool_size = 20
max_client_conn = 200
```

**۸) `postgresql.conf`**

```ini
shared_buffers = 1GB
effective_cache_size = 3GB
work_mem = 8MB
maintenance_work_mem = 256MB
max_wal_size = 4GB
random_page_cost = 1.1
```

**۹) `redis`**

```ini
maxmemory 256mb
maxmemory-policy allkeys-lru
save 900 1
appendonly yes
```

**۱۰) اندازه‌گیری قبل/بعد**

```bash
hey -z 30s -c 100 https://lab.example.com/api/public/stats
hey -z 30s -c 100 https://lab.example.com/
```

### مثال‌های عملی

**Lab Scenario: سخت‌سازی و تنظیم**

```bash
# ۱) fail2ban
sudo tee /etc/fail2ban/jail.d/lab.conf >/dev/null <<'EOF'
[DEFAULT]
bantime  = 3600
findtime = 600
maxretry = 5
backend  = systemd
ignoreip = 127.0.0.1/8 ::1

[sshd]
enabled = true

[nginx-limit-req]
enabled  = true
filter   = nginx-limit-req
logpath  = /var/log/nginx/error.log
maxretry = 5
EOF

sudo systemctl restart fail2ban
sudo fail2ban-client status
```

```bash
# ۲) unattended-upgrades
sudo tee /etc/apt/apt.conf.d/20auto-upgrades >/dev/null <<'EOF'
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
EOF
sudo systemctl restart unattended-upgrades
```

```bash
# ۳) gzip_static
find /srv/lab/spa/assets -type f \( -name '*.css' -o -name '*.js' \) \
  -exec gzip -9 -k -f {} \;
ls -la /srv/lab/spa/assets/
```

```bash
# ۴) بهینه‌سازی Nginx
sudo tee /etc/nginx/nginx.conf >/dev/null <<'EOF'
user www-data;
worker_processes auto;
worker_rlimit_nofile 65535;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log warn;

events {
    worker_connections 16384;
    multi_accept on;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - [$time_local] "$request" $status '
                    '$body_bytes_sent rt=$request_time urt=$upstream_response_time '
                    'ua=$upstream_addr us=$upstream_status rid=$request_id';
    access_log /var/log/nginx/access.log main;

    sendfile on;
    sendfile_max_chunk 1m;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    keepalive_requests 1000;

    client_body_buffer_size 16k;
    large_client_header_buffers 4 8k;

    open_file_cache max=10000 inactive=60s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;

    server_tokens off;
    client_max_body_size 8m;

    gzip on;
    gzip_vary on;
    gzip_comp_level 6;
    gzip_min_length 1024;
    gzip_static on;
    gzip_types text/plain text/css text/xml application/javascript
               application/json application/xml image/svg+xml;

    include /etc/nginx/conf.d/*.conf;
}
EOF

sudo nginx -t && sudo systemctl reload nginx
```

```bash
# ۵) PostgreSQL
sudo tee /etc/postgresql/16/main/conf.d/lab.conf >/dev/null <<'EOF'
shared_buffers = 1GB
effective_cache_size = 3GB
work_mem = 8MB
maintenance_work_mem = 256MB
max_wal_size = 4GB
random_page_cost = 1.1
log_min_duration_statement = 500ms
EOF
sudo systemctl restart postgresql
```

```bash
# ۶) Redis
sudo sed -i 's/^# maxmemory .*/maxmemory 256mb/' /etc/redis/redis.conf
sudo sed -i 's/^# maxmemory-policy .*/maxmemory-policy allkeys-lru/' /etc/redis/redis.conf
sudo systemctl restart redis-server
redis-cli -a CHANGE_ME info memory | head -5
```

**تست امنیت:**

```bash
# هدرها
curl -sSI https://lab.example.com/ | grep -Ei 'strict|x-frame|x-content|referrer|permissions|server'

# فایل حساس
for p in .env .git/config index.php.bak; do
  printf '%-18s → ' "$p"
  curl -sS -o /dev/null -w '%{http_code}\n' "https://lab.example.com/$p"
done

# rate limit
for i in $(seq 1 30); do
  curl -s -o /dev/null -w '%{http_code} ' https://lab.example.com/api/public/stats
done; echo

# TLS
docker run --rm -it drwetter/testssl.sh --quiet --severity HIGH https://lab.example.com
```

**تست کارایی:**

```bash
# قبل از gzip_static
hey -z 20s -c 50 https://lab.example.com/assets/app.css > /tmp/a.txt

# بعد
hey -z 20s -c 50 https://lab.example.com/assets/app.css > /tmp/b.txt

grep 'Requests/sec' /tmp/a.txt /tmp/b.txt
```

### مقایسه و الگوهای مختلف

**`gzip` در برابر `gzip_static`**

| | `gzip` | `gzip_static` |
|---|---|---|
| CPU | ⚠️ | ✅ |
| نیاز به `.gz` | ❌ | ✅ |
| مناسب | API | ✅ استاتیک |

**`fail2ban` در برابر `limit_req`**

| | fail2ban | `limit_req` |
|---|---|---|
| لایه | IP | نرخ |
| تأخیر | ✅ | ✅ |
| مناسب | تکرار | حجم |

**`unattended-upgrades` در برابر دستی**

| | auto | دستی |
|---|---|---|
| امنیت | ✅ | ⚠️ |
| کنترل | ⚠️ | ✅ |
| توصیه | ✅ امنیتی | — |

**دیدگاه «بهینه‌سازی قبل از اندازه‌گیری»**: حدس. `hey`/`ab` و `strace`.

### اشکال شایع

**۱) `gzip_static` بدون `.gz`**
بی‌اثر. `gzip -k`.

**۲) `immutable` بدون hash**
نسخه‌ی قدیمی.

**۳) `fail2ban` خودش را بلاک می‌کند**
`ignoreip`.

**۴) `unattended-upgrades` و reboot**
kernel. `needrestart`.

**۵) `maxmemory` بدون policy**
`OOM`. `allkeys-lru`.

**۶) `work_mem` بالا**
`OOM`. ۸MB.

**۷) `worker_connections` بدون `rlimit`**
`Too many open files`.

**۸) `open_file_cache` بدون `valid`**
متادیتای قدیمی.

**۹) `sendfile_max_chunk` نامحدود**
worker اشغال.

**۱۰) `log_min_duration_statement = 0`**
دیسک پر.

### نکات مهم (Tips & Tricks)

- **`gzip_static`** برای استاتیک.
- **`fail2ban` + `ignoreip`**.
- **`unattended-upgrades`**.
- **`open_file_cache`**.
- **`hey`** برای اندازه‌گیری.
- **`testssl.sh`** قبل از تحویل.
- **`pgbouncer`** اگر اتصال زیاد.

### خلاصه فصل

- سخت‌سازی: `fail2ban`، `unattended-upgrades`، هدرها.
- `gzip_static` CPU را حذف می‌کند.
- `immutable` فقط با hash.
- `open_file_cache` برای استاتیک.
- `Redis` با `maxmemory-policy`.
- اندازه‌گیری، نه حدس.

**سؤال مروری:** چرا `immutable` برای فایل بدون hash خطرناک است؟

---

## فصل ۷۹: مانیتورینگ، لاگ و بکاپ

### مقدمه

سیستم امن و سریع است. حالا باید «قابل مشاهده» و «قابل بازیابی» شود.

### مفاهیم اساسی

**۱) سه لاگ**

| لاگ | محتوا |
|---|---|
| Nginx access | هر درخواست |
| Nginx error | خطا |
| اپ (journal) | خطای اپ |

**۲) `stub_status`**

```nginx
server {
    listen 127.0.0.1:8081;
    location /status { stub_status; allow 127.0.0.1; deny all; }
}
```

**۳) متریک‌های کلیدی**

| متریک | منبع |
|---|---|
| RPS | `stub_status` |
| 5xx rate | لاگ/اپ |
| p95 latency | `$request_time` |
| cache hit | `$upstream_cache_status` |
| DB conn | `pg_stat_activity` |
| Redis mem | `redis-cli info` |

**۴) `logrotate`**

```bash
/var/log/nginx/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 www-data adm
    sharedscripts
    postrotate
        [ -f /run/nginx.pid ] && kill -USR1 $(cat /run/nginx.pid)
    endscript
}
```

**۵) `pg_dump` روزانه**

```bash
pg_dump -Fc -Z 6 -d lab -f /backup/lab_$(date +%F).dump
```

**۶) بکاپ فایل خصوصی**

```bash
tar czf /backup/private_$(date +%F).tar.gz -C /srv/lab private
```

**۷) `restore` تست**

```bash
createdb lab_restore
pg_restore -d lab_restore /backup/lab_2026-10-22.dump
psql -d lab_restore -c "SELECT count(*) FROM orders;"
```

**۸) هشدارها**

| هشدار | شرط |
|---|---|
| 5xx | >۲٪ در ۵ دقیقه |
| p95 | >۱s |
| cache hit | <۳۰٪ |
| دیسک | <۱۵٪ |
| گواهی | <۱۴ روز |

**۹) `GoAccess`**

```bash
goaccess /var/log/nginx/access.log --log-format=COMBINED -o /srv/lab/report.html
```

**۱۰) `prometheus` + `node_exporter` + `nginx_exporter`**

(فصل ۴۹)

### مثال‌های عملی

**Lab Scenario: مانیتورینگ و بکاپ**

```bash
# ۱) stub_status
sudo tee /etc/nginx/conf.d/status.conf >/dev/null <<'EOF'
server {
    listen 127.0.0.1:8081;
    server_name _;
    location /status {
        stub_status;
        allow 127.0.0.1;
        deny all;
        access_log off;
    }
}
EOF
sudo nginx -t && sudo systemctl reload nginx
curl -s http://127.0.0.1:8081/status
```

```bash
# ۲) اسکریپت مانیتورینگ
sudo tee /srv/lab/scripts/healthcheck.sh >/dev/null <<'EOF'
#!/usr/bin/env bash
set -uo pipefail
FAIL=0

check() {
  local name=$1 url=$2 expect=${3:-200}
  code=$(curl -sS -o /dev/null -m 5 -w '%{http_code}' "$url" || echo 000)
  if [ "$code" != "$expect" ]; then
    echo "FAIL $name ($code)"; FAIL=1
  else
    echo "OK   $name"
  fi
}

check "https"      "https://lab.example.com/health"
check "spa"        "https://lab.example.com/"
check "api-public" "https://lab.example.com/api/public/stats"
check "api-auth"   "https://lab.example.com/api/orders" 401

# دیسک
USE=$(df --output=pcent / | tail -1 | tr -dc '0-9')
[ "$USE" -gt 85 ] && { echo "FAIL disk ${USE}%"; FAIL=1; } || echo "OK   disk ${USE}%"

# گواهی
DAYS=$(( ( $(date -d "$(openssl x509 -in /etc/letsencrypt/live/lab.example.com/cert.pem -noout -enddate | cut -d= -f2)" +%s) - $(date +%s) ) / 86400 ))
[ "$DAYS" -lt 14 ] && { echo "FAIL cert ${DAYS}d"; FAIL=1; } || echo "OK   cert ${DAYS}d"

# DB
psql "postgresql://lab:CHANGE_ME@127.0.0.1/lab" -tAc "SELECT 1" >/dev/null \
  && echo "OK   db" || { echo "FAIL db"; FAIL=1; }

# Redis
redis-cli -a CHANGE_ME ping >/dev/null 2>&1 && echo "OK   redis" || { echo "FAIL redis"; FAIL=1; }

exit $FAIL
EOF

sudo chmod +x /srv/lab/scripts/healthcheck.sh
sudo -u lab /srv/lab/scripts/healthcheck.sh
```

```bash
# ۳) بکاپ
sudo tee /srv/lab/scripts/backup.sh >/dev/null <<'EOF'
#!/usr/bin/env bash
set -euo pipefail

DATE=$(date +%F_%H%M)
DEST=/backup/lab
mkdir -p "$DEST"

# DB
pg_dump -Fc -Z 6 -d lab -f "$DEST/db_$DATE.dump"
pg_restore -l "$DEST/db_$DATE.dump" > /dev/null

# فایل‌ها
tar czf "$DEST/files_$DATE.tar.gz" -C /srv/lab spa private nginx

# globals (نقش‌ها)
pg_dumpall --globals-only -f "$DEST/globals_$DATE.sql"

# retention
find "$DEST" -type f -mtime +14 -delete

echo "backup ok: $DEST/*_$DATE.*"
EOF

sudo chmod +x /srv/lab/scripts/backup.sh
sudo -u postgres /srv/lab/scripts/backup.sh
```

```bash
# ۴) cron
sudo tee /etc/cron.d/lab >/dev/null <<'EOF'
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

0 2 * * *  postgres  /srv/lab/scripts/backup.sh >> /var/log/lab-backup.log 2>&1
*/5 * * * * lab      /srv/lab/scripts/healthcheck.sh >> /var/log/lab-health.log 2>&1
EOF

sudo chmod 644 /etc/cron.d/lab
```

```bash
# ۵) تست بازیابی
sudo -u postgres createdb lab_restore
sudo -u postgres pg_restore -d lab_restore /backup/lab/db_*.dump
sudo -u postgres psql -d lab_restore -c "SELECT count(*) AS orders FROM orders;"
sudo -u postgres dropdb lab_restore
```

```bash
# ۶) logrotate
sudo tee /etc/logrotate.d/nginx >/dev/null <<'EOF'
/var/log/nginx/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 www-data adm
    sharedscripts
    postrotate
        [ -f /run/nginx.pid ] && kill -USR1 $(cat /run/nginx.pid)
    endscript
}
EOF
sudo logrotate -d /etc/logrotate.d/nginx
```

```bash
# ۷) گزارش GoAccess
sudo apt install -y goaccess
sudo goaccess /var/log/nginx/access.log --log-format=COMBINED \
  -o /srv/lab/spa/report.html --html-prefs='{"theme":"darkGray"}'
```

**لاگ ساختاریافته:**

```nginx
log_format json_main escape=json
'{'
  '"ts":"$time_iso8601",'
  '"ip":"$remote_addr",'
  '"method":"$request_method",'
  '"uri":"$request_uri",'
  '"status":$status,'
  '"rt":$request_time,'
  '"urt":"$upstream_response_time",'
  '"up":"$upstream_addr",'
  '"cache":"$upstream_cache_status",'
  '"rid":"$request_id"'
'}';
access_log /var/log/nginx/access.json.log json_main buffer=32k flush=5s;
```

```bash
jq -r 'select(.status >= 500) | "\(.ts) \(.uri) up=\(.up)"' /var/log/nginx/access.json.log | tail
jq -r '.rt' /var/log/nginx/access.json.log | awk '{s+=$1;n++} END {print "avg rt:", s/n}'
jq -r '.cache' /var/log/nginx/access.json.log | sort | uniq -c
```

### مقایسه و الگوهای مختلف

**`stub_status` در برابر `nginx_exporter`**

| | `stub_status` | exporter |
|---|---|---|
| per-upstream | ❌ | ⚠️ |
| Prometheus | ❌ | ✅ |
| سادگی | ✅ | ⚠️ |

**بکاپ منطقی در برابر فیزیکی**

| | `pg_dump` | `pg_basebackup` |
|---|---|---|
| PITR | ❌ | ✅ |
| سادگی | ✅ | ⚠️ |
| مناسب | DB کوچک | ✅ بزرگ |

**`cron` در برابر `systemd timer`**

| | cron | timer |
|---|---|---|
| سادگی | ✅ | ⚠️ |
| لاگ | ⚠️ | ✅ journal |
| توصیه | ✅ | ✅ |

**دیدگاه «بکاپ بدون تست کافی است»**: بکاپ تست‌نشده، بکاپ نیست.

### اشکال شایع

**۱) `pg_dump` بدون `-Fc`**
حجم، بدون انتخابی.

**۲) `restore` تست‌نشده**
ماه‌ها بعد می‌فهمی.

**۳) `logrotate` بدون `USR1`**
Nginx به فایل قدیمی می‌نویسد.

**۴) `stub_status` باز**
اطلاعات لو. `allow`.

**۵) `healthcheck` فقط `SELECT 1`**
DB `read-only` را نمی‌بیند.

**۶) `cron` بدون `PATH`**
دستور پیدا نمی‌شود.

**۷) بکاپ روی همان دیسک**
دیسک بمیرد، هر دو.

**۸) `retention` بی‌نهایت**
دیسک پر.

**۹) `jq` روی فایل بزرگ**
حافظه. `-c`.

**۱۰) `access_log` بدون `$request_time`**
کندی پنهان.

### نکات مهم (Tips & Tricks)

- **`stub_status`** محدود.
- **`healthcheck.sh`** با DB/Redis/گواهی.
- **`pg_dump -Fc`** + `pg_restore -l`.
- **`restore` ماهانه**.
- **`logrotate` + `USR1`**.
- **`$request_time`** و `$upstream_cache_status`.
- **بکاپ در مقصد جدا**.

### خلاصه فصل

- سه لاگ + `stub_status`.
- `healthcheck` واقعی.
- `pg_dump` + `tar` + `globals`.
- `restore` تست‌شده.
- `logrotate` با `USR1`.
- متریک‌های کلیدی: RPS، 5xx، p95، cache hit.

**سؤال مروری:** چرا `healthcheck` که فقط `SELECT 1` می‌زند، می‌تواند خطای واقعی را پنهان کند؟

---

## فصل ۸۰: تست بار، تحویل و چک‌لیست

### مقدمه

آخرین قدم: اندازه‌گیری، مستندسازی، و تحویل. سیستم باید «قابل تحویل» باشد، نه فقط «کار می‌کند».

### مفاهیم اساسی

**۱) تست بار**

| ابزار | مناسب |
|---|---|
| `hey` | ساده، سریع |
| `wrk` | throughput بالا |
| `k6` | سناریو، threshold |
| `vegeta` | constant rate |

**۲) سه سناریو**

| سناریو | هدف |
|---|---|
| Smoke | ۱–۲ کاربر، درستی |
| Load | بار مورد انتظار |
| Stress | تا شکست |

**۳) معیار پذیرش**

| متریک | هدف |
|---|---|
| p95 latency | <۵۰۰ms |
| 5xx | <۰.۱٪ |
| throughput | >۵۰ rps |
| cache hit | >۵۰٪ |

**۴) `k6` نمونه**

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 20 },
    { duration: '1m',  target: 50 },
    { duration: '30s', target: 0 },
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],
    http_req_failed: ['rate<0.001'],
  },
};

export default function () {
  const r = http.get('https://lab.example.com/api/public/stats');
  check(r, { 'status 200': (x) => x.status === 200 });
  sleep(1);
}
```

**۵) `hey`**

```bash
hey -z 60s -c 50 https://lab.example.com/api/public/stats
hey -z 60s -c 50 -H 'Accept-Encoding: gzip' https://lab.example.com/
```

**۶) مستندسازی تحویل**

```
docs/
├── README.md        (معماری، اجرا)
├── runbook.md       (حادثه)
├── changelog.md
└── adr/             (تصمیم‌ها)
```

**۷) `runbook` — سه سناریو**

| سناریو | اقدام |
|---|---|
| `502` | `ss -ltnp`، `journalctl -u lab-api` |
| دیسک پر | `du`، `logrotate`، پاک‌سازی بکاپ |
| گواهی | `certbot renew`، `reload` |

**۸) تحویل**

| مورد | تحویل‌دادنی |
|---|---|
| کد | git repo |
| config | `/srv/lab` در git |
| اسکریپت | `deploy.sh`، `backup.sh` |
| مستندات | `README`، `runbook` |
| دسترسی | رمز در `Vault` |
| مانیتورینگ | داشبورد + هشدار |

**۹) `deploy.sh`**

```bash
#!/usr/bin/env bash
set -euo pipefail
# pull, install, migrate, restart, healthcheck, rollback
```

**۱۰) `smoke test` بعد از استقرار**

```bash
curl -fsS https://lab.example.com/health
curl -fsS -o /dev/null -w '%{http_code}\n' https://lab.example.com/
```

### مثال‌های عملی

**Lab Scenario: تست و تحویل**

```bash
# ۱) smoke
hey -n 100 -c 5 https://lab.example.com/health

# ۲) load
hey -z 60s -c 50 https://lab.example.com/api/public/stats
hey -z 60s -c 50 https://lab.example.com/

# ۳) gzip
hey -z 30s -c 50 -H 'Accept-Encoding: gzip' https://lab.example.com/assets/app.css

# ۴) rate limit
hey -z 20s -c 100 https://lab.example.com/api/login
# باید 429 ببینی
```

**`k6`:**

```bash
sudo apt install -y k6 || docker run --rm -i grafana/k6 run - <load.js
k6 run load.js
```

**تحلیل نتیجه:**

```bash
hey -z 60s -c 50 https://lab.example.com/api/public/stats | tail -20
# Requests/sec
# p95
# Status code distribution
```

**بررسی cache hit:**

```bash
jq -r '.cache' /var/log/nginx/access.json.log | sort | uniq -c
# 1200 HIT
#  300 MISS
```

**`runbook.md` نمونه:**

```markdown
# Runbook — Lab Shop

## 502 Bad Gateway
1. `systemctl status lab-api`
2. `ss -ltnp | grep 8000`
3. `journalctl -u lab-api -n 50`
4. اگر DB قطع: `systemctl status postgresql`

## دیسک پر
1. `df -h`
2. `du -sh /var/log/* | sort -h`
3. `logrotate -f /etc/logrotate.d/nginx`
4. پاک‌سازی بکاپ قدیمی

## گواهی منقضی
1. `certbot renew --dry-run`
2. `certbot renew`
3. `systemctl reload nginx`

## Rollback
1. `cd /srv/lab && git checkout <prev-tag>`
2. `./scripts/deploy.sh`
```

**`README.md` نمونه:**

```markdown
# Lab Shop

## معماری
Nginx → FastAPI → PostgreSQL/Redis

## اجرا
```bash
./scripts/deploy.sh
```

## مسیرها
- `/` SPA
- `/api/public/*` کش ۵s
- `/api/*` احراز هویت
- `/health`

## مانیتورینگ
- `http://127.0.0.1:8081/status`
- `journalctl -u lab-api`

## بکاپ
`/srv/lab/scripts/backup.sh` — روزانه ۲ بامداد
```

**`deploy.sh` کامل:**

```bash
#!/usr/bin/env bash
set -euo pipefail

APP_DIR=/srv/lab
cd "$APP_DIR"

PREV=$(git rev-parse --short HEAD 2>/dev/null || echo none)
echo "deploying (prev=$PREV)"

git pull --ff-only

sudo -u lab bash -c '
cd /srv/lab/app
./venv/bin/pip install -q -r requirements.txt
'

sudo nginx -t
sudo systemctl reload nginx
sudo systemctl restart lab-api

for i in $(seq 1 20); do
  if curl -fsS http://127.0.0.1:8000/health >/dev/null; then
    echo "OK"
    exit 0
  fi
  sleep 1
done

echo "FAILED — rolling back"
git reset --hard "$PREV"
sudo systemctl restart lab-api
exit 1
```

**چک‌لیست تحویل:**

```markdown
## امنیت
- [ ] TLS 1.2+ و HSTS
- [ ] هدرهای امنیتی
- [ ] rate limit روی login
- [ ] fail2ban
- [ ] `.env` خارج از webroot
- [ ] مجوز 644/755

## کارایی
- [ ] gzip_static
- [ ] cache استاتیک
- [ ] micro-cache API
- [ ] p95 < 500ms

## عملیات
- [ ] healthcheck
- [ ] لاگ + چرخش
- [ ] بکاپ + تست restore
- [ ] runbook
- [ ] deploy.sh با rollback

## مشاهده‌پذیری
- [ ] stub_status
- [ ] هشدار 5xx
- [ ] هشدار گواهی
- [ ] هشدار دیسک
```

### مقایسه و الگوهای مختلف

**`hey` در برابر `k6` در برابر `wrk`**

| | hey | k6 | wrk |
|---|---|---|---|
| سناریو | ⚠️ | ✅ | ❌ |
| threshold | ❌ | ✅ | ❌ |
| سادگی | ✅✅ | ⚠️ | ✅ |
| مناسب | سریع | ✅ CI | throughput |

**Load در برابر Stress**

| | Load | Stress |
|---|---|---|
| هدف | تأیید ظرفیت | یافتن شکست |
| شدت | مورد انتظار | تا شکست |

**`runbook` در برابر `wiki`**

| | runbook | wiki |
|---|---|---|
| دسترسی در حادثه | ✅ | ⚠️ |
| به‌روز | ⚠️ | ✅ |
| توصیه | ✅ چاپ | مکمل |

**دیدگاه «تست بار فقط قبل از launch»**: هر تغییر بزرگ، تست بار. رگرسیون کارایی واقعی است.

### اشکال شایع

**۱) تست روی محیط توسعه**
نتیجه بی‌معنا. staging مشابه تولید.

**۲) تست بدون warm-up**
نتیجه بدتر. `-w` یا warm.

**۳) `hey` بدون `-H 'Accept-Encoding'`**
gzip تست نمی‌شود.

**۴) نبود threshold**
«به‌نظر خوب» کافی نیست. عدد.

**۵) نبود runbook**
حادثه، هرج‌ومرج.

**۶) `deploy.sh` بدون rollback**
بحران.

**۷) نبود smoke بعد از deploy**
نسخه‌ی خراب.

**۸) مستندات بدون معماری**
تیم جدید گم می‌شود.

**۹) رمز در مستندات**
خطر. `Vault`.

**۱۰) چک‌لیست بدون مدرک**
تیک بی‌معنا.

### نکات مهم (Tips & Tricks)

- **`k6`** با threshold.
- **`hey`** برای سریع.
- **warm-up** قبل از تست.
- **`runbook`** چاپ‌شده.
- **`deploy.sh` با rollback**.
- **`smoke`** بعد از deploy.
- **چک‌لیست با مدرک**.

### خلاصه فصل

- سه سناریو: smoke، load، stress.
- معیار عددی: p95، 5xx، throughput.
- `k6` برای threshold در CI.
- `runbook` برای حادثه.
- `deploy.sh` با rollback.
- تحویل: کد، config، مستندات، دسترسی.

**سؤال مروری:** چرا تست بار روی محیط توسعه، تصمیم تولید را خراب می‌کند؟

---

## منابع و مراجع

**مستندات رسمی**
- [Nginx — Admin Guide](https://docs.nginx.com/nginx/admin-guide/)
- [FastAPI — Deployment](https://fastapi.tiangolo.com/deployment/)
- [Gunicorn — Settings](https://docs.gunicorn.org/en/stable/settings.html)
- [PostgreSQL — Backup](https://www.postgresql.org/docs/16/backup.html)
- [Redis — Configuration](https://redis.io/docs/management/config/)

**ابزار**
- [hey](https://github.com/rakyll/hey)
- [k6](https://k6.io/docs/)
- [wrk](https://github.com/wg/wrk)
- [GoAccess](https://goaccess.io/)
- [testssl.sh](https://testssl.sh/)

**آموزش**
- [The Twelve-Factor App](https://12factor.net/)
- [Google SRE Book](https://sre.google/sre-book/table-of-contents/)
- [Nginx caching guide](https://docs.nginx.com/nginx/admin-guide/content-cache/content-caching/)

---

**بعدی:** [`19-Checklist-and-Glossary.md`](19-Checklist-and-Glossary.md) — چک‌لیست تولید، واژه‌نامه و نقشه راه
