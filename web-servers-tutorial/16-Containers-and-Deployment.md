# بخش ۱۶: کانتینر و استقرار

**مدت یادگیری**: ۵ تا ۶ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۰۷ (اپ داینامیک)، بخش ۰۸ (پروکسی)

---

## فهرست محتوا

- [فصل ۶۸: Docker برای Nginx و Apache](#فصل-۶۸)
- [فصل ۶۹: Compose — Nginx + app + Certbot](#فصل-۶۹)
- [فصل ۷۰: Kubernetes برای وب‌سرور](#فصل-۷۰)
- [فصل ۷۱: CI/CD، config as code و blue/green](#فصل-۷۱)

---

## فصل ۶۸: Docker برای Nginx و Apache

### مقدمه

Docker وب‌سرور را «قابل بازتولید» می‌کند: همان image، همان رفتار، هر جا. ولی کانتینر، مدل ذهنی متفاوتی دارد: config immutable، لاگ به stdout، و شبکه‌ی داخلی.

### مفاهیم اساسی

**۱) image رسمی**

```bash
docker pull nginx:1.26-alpine
docker pull httpd:2.4-alpine
```

| image | حجم | مناسب |
|---|---|---|
| `nginx:1.26-alpine` | ~۵۰MB | ✅ تولید |
| `nginx:1.26` | ~۱۸۰MB | دیباگ |
| `httpd:2.4-alpine` | ~۶۰MB | Apache |

**۲) مسیرهای مهم**

| مسیر | محتوا |
|---|---|
| `/etc/nginx/nginx.conf` | config اصلی |
| `/etc/nginx/conf.d/` | سایت‌ها |
| `/usr/share/nginx/html` | `DocumentRoot` |
| `/var/log/nginx/` | لاگ (symlink به stdout) |

**۳) لاگ به stdout**

image رسمی Nginx، `access.log` و `error.log` را به `/dev/stdout` و `/dev/stderr` symlink می‌کند. `docker logs` کار می‌کند.

**۴) config immutable**

```
❌ docker exec -it nginx vi /etc/nginx/nginx.conf
✅ Dockerfile یا volume + reload
```

**۵) `Dockerfile` — استاتیک**

```dockerfile
FROM nginx:1.26-alpine

RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d/app.conf
COPY dist/ /usr/share/nginx/html/

EXPOSE 80
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -qO- http://127.0.0.1/health || exit 1
```

**۶) `Dockerfile` — با config کامل**

```dockerfile
FROM nginx:1.26-alpine

RUN apk add --no-cache curl

COPY nginx/nginx.conf /etc/nginx/nginx.conf
COPY nginx/conf.d/ /etc/nginx/conf.d/
COPY public/ /var/www/html/

RUN nginx -t

STOPSIGNAL SIGQUIT
EXPOSE 80 443
```

**۷) `nginx -t` در build**

اگر config خراب باشد، build شکست می‌خورد، نه production.

**۸) `STOPSIGNAL SIGQUIT`**

`SIGQUIT` = graceful shutdown. `SIGTERM` = سریع. برای Nginx، `SIGQUIT` بهتر.

**۹) غیر-root**

```dockerfile
FROM nginx:1.26-alpine

RUN addgroup -S app && adduser -S app -G app \
 && chown -R app:app /var/cache/nginx /var/run /var/log/nginx

USER app
```

> image رسمی Nginx با `root` شروع می‌شود (برای bind ۸۰). برای غیر-root، پورت `8080` و config سفارشی.

**۱۰) `HEALTHCHECK`**

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget -qO- http://127.0.0.1/health || exit 1
```

### مثال‌های عملی

**Lab Scenario: image تولیدی Nginx**

ساختار:

```
.
├── Dockerfile
├── nginx/
│   ├── nginx.conf
│   └── conf.d/
│       └── app.conf
└── public/
    ├── index.html
    └── assets/app.css
```

```dockerfile
# Dockerfile
FROM nginx:1.26-alpine

RUN apk add --no-cache curl

COPY nginx/nginx.conf /etc/nginx/nginx.conf
COPY nginx/conf.d/ /etc/nginx/conf.d/
COPY public/ /var/www/html/

RUN nginx -t

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -fsS http://127.0.0.1/health || exit 1

STOPSIGNAL SIGQUIT
EXPOSE 80
```

```nginx
# nginx/nginx.conf
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events { worker_connections 1024; }

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format main '$remote_addr - [$time_local] "$request" $status $body_bytes_sent rt=$request_time';
    access_log /var/log/nginx/access.log main;

    sendfile on;
    tcp_nopush on;
    keepalive_timeout 65;
    server_tokens off;
    gzip on;
    gzip_types text/css application/javascript application/json image/svg+xml;

    include /etc/nginx/conf.d/*.conf;
}
```

```nginx
# nginx/conf.d/app.conf
server {
    listen 80;
    server_name _;
    root /var/www/html;
    index index.html;

    location = /health {
        access_log off;
        default_type text/plain;
        return 200 "ok\n";
    }

    location ~* \.(css|js|woff2|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        access_log off;
    }

    location / { try_files $uri $uri/ =404; }
}
```

```bash
docker build -t lab-web:1.0 .
docker run -d --name lab-web -p 8080:80 lab-web:1.0
docker ps
docker logs lab-web
curl -sS http://localhost:8080/health
curl -sSI http://localhost:8080/assets/app.css | head -5
```

**Apache:**

```dockerfile
FROM httpd:2.4-alpine

RUN sed -i 's/^#LoadModule rewrite_module/LoadModule rewrite_module/' conf/httpd.conf \
 && sed -i 's/^#LoadModule headers_module/LoadModule headers_module/' conf/httpd.conf

COPY httpd.conf /usr/local/apache2/conf/httpd.conf
COPY public/ /usr/local/apache2/htdocs/

RUN httpd -t
EXPOSE 80
```

```bash
docker build -t lab-apache:1.0 .
docker run -d --name lab-apache -p 8081:80 lab-apache:1.0
curl -sSI http://localhost:8081/
```

**compose حداقلی:**

```yaml
services:
  web:
    build: .
    ports: ["8080:80"]
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-fsS", "http://127.0.0.1/health"]
      interval: 30s
      timeout: 3s
      retries: 3
```

**`.dockerignore`:**

```
.git
node_modules
*.log
.env
dist/**/*.map
```

**بررسی image:**

```bash
docker image ls lab-web
docker history lab-web:1.0
docker run --rm -it lab-web:1.0 sh
# nginx -t
# ls -la /etc/nginx/conf.d/
```

### مقایسه و الگوهای مختلف

**`COPY` در برابر `volume` برای config**

| | `COPY` | `volume` |
|---|---|---|
| immutable | ✅ | ❌ |
| تغییر بدون build | ❌ | ✅ |
| مناسب | ✅ تولید | توسعه |

**`alpine` در برابر `debian`**

| | alpine | debian |
|---|---|---|
| حجم | ✅ | ❌ |
| سازگاری | ⚠️ | ✅ |
| دیباگ | ⚠️ | ✅ |

**`root` در برابر غیر-root**

| | root | غیر-root |
|---|---|---|
| پورت ۸۰ | ✅ | ❌ (`8080`) |
| امنیت | ❌ | ✅ |
| پیچیدگی | ✅ | ⚠️ |

**دیدگاه «Nginx در Docker کند است»**: روی لینوکس، تفاوت ناچیز. روی macOS/Windows، شبکه و `fsync` کندتر است.

### اشکال شایع

**۱) `docker exec` برای تغییر config**
با `rm` می‌رود. `Dockerfile`.

**۲) `nginx -t` در build نبودن**
config خراب، production خراب.

**۳) `HEALTHCHECK` نبودن**
traffic به container ناسالم.

**۴) `SIGTERM` به‌جای `SIGQUIT`**
قطع اتصال. `STOPSIGNAL`.

**۵) لاگ به فایل**
`docker logs` خالی. symlink به stdout.

**۶) `alpine` و `glibc`**
بعضی ماژول‌ها. `debian`.

**۷) `node_modules` در image**
حجم. `.dockerignore`.

**۸) `latest` tag**
غیرقابل بازتولید. tag دقیق.

**۹) `USER root`**
امنیت. غیر-root.

**۱۰) `ports` به‌جای `expose`**
همه‌چیز باز. `expose` داخلی.

### نکات مهم (Tips & Tricks)

- **`nginx -t`** در build.
- **`STOPSIGNAL SIGQUIT`**.
- **`HEALTHCHECK`**.
- **`.dockerignore`**.
- **tag دقیق**.
- **غیر-root** اگر ممکن.
- **لاگ به stdout**.

### خلاصه فصل

- image رسمی، base خوب.
- config در image، نه `exec`.
- `nginx -t` در build.
- `HEALTHCHECK` و `STOPSIGNAL`.
- لاگ به stdout.
- tag دقیق برای بازتولید.

**سؤال مروری:** چرا `docker exec` برای تغییر config Nginx یک anti-pattern است؟

---

## فصل ۶۹: Compose — Nginx + app + Certbot

### مقدمه

Compose یک فایل YAML برای چند سرویس است. برای وب‌سرور + اپ + TLS + cache، ایده‌آل است. این فصل، یک استک کامل و قابل اجرا.

### مفاهیم اساسی

**۱) ساختار**

```yaml
services:
  web:      # Nginx
  app:      # اپ
  certbot:  # TLS
  redis:    # session/cache
volumes:
networks:
```

**۲) شبکه**

```yaml
networks:
  frontend:
  backend:

services:
  web:
    networks: [frontend, backend]
  app:
    networks: [backend]
```

`app` از اینترنت پنهان است.

**۳) `depends_on` با `condition`**

```yaml
depends_on:
  app:
    condition: service_healthy
```

**۴) Certbot در Compose**

```yaml
certbot:
  image: certbot/certbot:latest
  volumes:
    - ./certbot/conf:/etc/letsencrypt
    - ./certbot/www:/var/www/certbot
  entrypoint: >
    /bin/sh -c "trap exit TERM;
    while :; do certbot renew --webroot -w /var/www/certbot --quiet;
    sleep 12h & wait $${!}; done;"
```

**۵) `webroot` برای چالش**

```nginx
location ^~ /.well-known/acme-challenge/ {
    root /var/www/certbot;
    default_type text/plain;
}
```

**۶) bootstrap گواهی**

```bash
# اولین بار: Nginx با HTTP-only
docker compose up -d web
docker compose run --rm certbot certonly --webroot -w /var/www/certbot \
  -d lab.example.com --agree-tos -m admin@example.com --no-eff-email

# سپس Nginx با TLS
docker compose restart web
```

**۷) reload بعد از تمدید**

```yaml
certbot:
  entrypoint: >
    /bin/sh -c "trap exit TERM;
    while :; do
      certbot renew --webroot -w /var/www/certbot --quiet --deploy-hook 'touch /var/www/certbot/.reload';
      sleep 12h & wait $${!};
    done;"
```

و Nginx با `inotify` یا `docker kill -s HUP`.

**۸) `env_file` و `secrets`**

```yaml
services:
  app:
    env_file: [.env]
    secrets: [db_password]
secrets:
  db_password:
    file: ./secrets/db_password.txt
```

**۹) `profiles`**

```yaml
services:
  prometheus:
    profiles: [monitoring]
```

```bash
docker compose --profile monitoring up -d
```

**۱۰) `deploy.resources`**

```yaml
deploy:
  resources:
    limits: { memory: 512M, cpus: '1.0' }
    reservations: { memory: 128M }
```

### مثال‌های عملی

**Lab Scenario: استک کامل**

ساختار:

```
.
├── compose.yaml
├── nginx/
│   ├── conf.d/app.conf
│   └── nginx.conf
├── app/
│   ├── Dockerfile
│   └── server.js
├── certbot/
│   ├── conf/
│   └── www/
└── public/
```

```yaml
# compose.yaml
name: lab

services:
  web:
    image: nginx:1.26-alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - ./public:/var/www/html:ro
      - ./certbot/conf:/etc/letsencrypt:ro
      - ./certbot/www:/var/www/certbot:ro
    depends_on:
      app:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://127.0.0.1/health"]
      interval: 30s
      timeout: 3s
      retries: 3
    networks: [frontend, backend]

  app:
    build: ./app
    restart: unless-stopped
    environment:
      NODE_ENV: production
      REDIS_URL: redis://redis:6379
    expose: ["3000"]
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://127.0.0.1:3000/health"]
      interval: 15s
      timeout: 3s
      retries: 3
    networks: [backend]

  redis:
    image: redis:7-alpine
    restart: unless-stopped
    command: ["redis-server", "--appendonly", "yes"]
    volumes:
      - redis:/data
    networks: [backend]

  certbot:
    image: certbot/certbot:latest
    volumes:
      - ./certbot/conf:/etc/letsencrypt
      - ./certbot/www:/var/www/certbot
    entrypoint: >
      /bin/sh -c "trap exit TERM;
      while :; do
        certbot renew --webroot -w /var/www/certbot --quiet;
        sleep 12h & wait $${!};
      done;"

volumes:
  redis:

networks:
  frontend:
  backend:
    internal: true
```

```nginx
# nginx/conf.d/app.conf
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
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_session_tickets off;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;

    root /var/www/html;
    index index.html;

    location = /health {
        access_log off;
        default_type text/plain;
        return 200 "ok\n";
    }

    location /api/ {
        proxy_pass http://app:3000/;
        proxy_http_version 1.1;
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 60s;
    }

    location / { try_files $uri $uri/ =404; }
}
```

```dockerfile
# app/Dockerfile
FROM node:20-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .

ENV NODE_ENV=production
EXPOSE 3000
HEALTHCHECK --interval=15s --timeout=3s \
  CMD wget -qO- http://127.0.0.1:3000/health || exit 1

USER node
CMD ["node", "server.js"]
```

```javascript
// app/server.js
import express from 'express';
import { createClient } from 'redis';

const app = express();
const redis = createClient({ url: process.env.REDIS_URL });
await redis.connect();

app.get('/health', async (req, res) => {
  try {
    await redis.ping();
    res.json({ status: 'ok' });
  } catch {
    res.status(503).json({ status: 'degraded' });
  }
});

app.get('/api/hello', async (req, res) => {
  const n = await redis.incr('hits');
  res.json({ hello: 'world', hits: n });
});

app.listen(3000, '0.0.0.0');
```

**راه‌اندازی:**

```bash
# ۱) build
docker compose build

# ۲) اولین بار: HTTP-only (بدون گواهی)
# موقتاً بلوک ۴۴۳ را کامنت کن یا یک config موقت بگذار
docker compose up -d web app redis

# ۳) گواهی
docker compose run --rm certbot certonly --webroot -w /var/www/certbot \
  -d lab.example.com --agree-tos -m admin@example.com --no-eff-email

# ۴) config کامل + restart
docker compose restart web

# ۵) بررسی
docker compose ps
curl -sS https://lab.example.com/health
curl -sS https://lab.example.com/api/hello
```

**تست:**

```bash
docker compose logs -f web
docker compose exec web nginx -t
docker compose exec app wget -qO- http://127.0.0.1:3000/health
```

**تمدید:**

```bash
docker compose run --rm certbot renew --dry-run
docker compose exec web nginx -s reload
```

**`nginx -s reload` بعد از تمدید خودکار:**

```yaml
# راه: sidecar با inotify
  reloader:
    image: alpine:3.20
    command: >
      sh -c "apk add --no-cache inotify-tools &&
      while inotifywait -e close_write /etc/letsencrypt/live/lab.example.com/; do
        nginx -s reload; done"
    volumes:
      - ./certbot/conf:/etc/letsencrypt:ro
```

یا `docker kill -s HUP <container>` از cron.

### مقایسه و الگوهای مختلف

**`volume` برای config در برابر `COPY` در image**

| | volume | COPY |
|---|---|---|
| توسعه | ✅ | ❌ |
| تولید | ⚠️ | ✅ |
| immutable | ❌ | ✅ |

**Certbot در Compose در برابر روی host**

| | Compose | host |
|---|---|---|
| جداسازی | ✅ | ⚠️ |
| سادگی | ⚠️ | ✅ |
| مناسب | ✅ | تک‌سرور |

**`internal: true`**

| | `internal` | عادی |
|---|---|---|
| دسترسی اینترنت | ❌ | ✅ |
| امنیت | ✅ | ⚠️ |
| مناسب | ✅ backend | frontend |

**دیدگاه «Compose برای تولید»**: برای تک‌سرور با `restart` و `healthcheck`، کار می‌کند. برای HA، K8s/Swarm.

### اشکال شایع

**۱) `depends_on` بدون `healthcheck`**
اپ قبل از آماده شدن. `condition`.

**۲) `internal: true` برای web**
web به اینترنت دسترسی ندارد. فقط backend.

**۳) گواهی قبل از Nginx**
Nginx بالا نمی‌آید. bootstrap.

**۴) `certbot renew` بدون reload**
گواهی جدید، Nginx قدیمی.

**۵) `volume` با مجوز اشتباه**
`403`. `chown`.

**۶) `ports` برای app**
app در اینترنت. `expose`.

**۷) `latest` tag**
غیرقابل بازتولید.

**۸) `redis` بدون `appendonly`**
داده با restart می‌رود.

**۹) `nginx.conf` بدون `include conf.d`**
سایت لود نمی‌شود.

**۱۰) `docker compose down -v`**
volume داده می‌رود.

### نکات مهم (Tips & Tricks)

- **`healthcheck` + `condition`**.
- **`internal: true`** برای backend.
- **bootstrap گواهی**.
- **reload بعد از تمدید**.
- **`expose`** به‌جای `ports` برای app.
- **tag دقیق**.
- **`docker compose config`** برای بررسی.

### خلاصه فصل

- Compose استک کامل می‌دهد.
- `healthcheck` ترتیب را تضمین می‌کند.
- Certbot در Compose با `webroot`.
- `internal` backend را می‌بندد.
- reload بعد از تمدید ضروری.
- `expose` برای سرویس داخلی.

**سؤال مروری:** چرا Nginx بدون گواهی موجود، با config TLS بالا نمی‌آید و bootstrap چطور حلش می‌کند؟

---

## فصل ۷۰: Kubernetes برای وب‌سرور

### مقدمه

در Kubernetes، وب‌سرور یک Deployment است، نه یک سرور. `ConfigMap` برای config، `Secret` برای گواهی، `Ingress` برای مسیریابی، و `HPA` برای مقیاس.

### مفاهیم اساسی

**۱) اجزا**

```
Ingress → Service → Deployment (Pods: nginx)
                     ├── ConfigMap (nginx.conf)
                     ├── Secret (TLS)
                     └── PVC (فایل استاتیک، اگر لازم)
```

**۲) `ConfigMap`**

```bash
kubectl create configmap nginx-conf --from-file=nginx.conf
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf
  namespace: lab
data:
  nginx.conf: |
    server {
      listen 80;
      server_name _;
      root /usr/share/nginx/html;
      location /health { return 200 "ok\n"; }
    }
```

**۳) `Deployment`**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: lab
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate: { maxSurge: 1, maxUnavailable: 0 }
  selector:
    matchLabels: { app: web }
  template:
    metadata:
      labels: { app: web }
    spec:
      containers:
        - name: nginx
          image: nginx:1.26-alpine
          ports: [{ containerPort: 80 }]
          volumeMounts:
            - name: conf
              mountPath: /etc/nginx/conf.d
          readinessProbe:
            httpGet: { path: /health, port: 80 }
            initialDelaySeconds: 3
            periodSeconds: 5
          livenessProbe:
            httpGet: { path: /health, port: 80 }
            initialDelaySeconds: 10
            periodSeconds: 10
          resources:
            requests: { cpu: 50m, memory: 64Mi }
            limits:   { cpu: 500m, memory: 256Mi }
          securityContext:
            runAsNonRoot: true
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities: { drop: ["ALL"] }
      volumes:
        - name: conf
          configMap: { name: nginx-conf }
```

**۴) `readinessProbe` در برابر `livenessProbe`**

| | readiness | liveness |
|---|---|---|
| خطا | حذف از Service | restart |
| مناسب | آماده‌ی ترافیک | قفل‌شدن |

**۵) `securityContext`**

| تنظیم | اثر |
|---|---|
| `runAsNonRoot` | کاربر غیر-root |
| `readOnlyRootFilesystem` | فایل‌سیستم فقط‌خواندنی |
| `drop: ALL` | حذف capability |
| `allowPrivilegeEscalation: false` | بدون escalation |

> Nginx برای `/var/cache/nginx` و `/var/run` نیاز به نوشتن دارد. `emptyDir` mount کن.

**۶) `emptyDir` برای پوشه‌های قابل نوشتن**

```yaml
volumeMounts:
  - { name: cache, mountPath: /var/cache/nginx }
  - { name: run,   mountPath: /var/run }
volumes:
  - { name: cache, emptyDir: {} }
  - { name: run,   emptyDir: {} }
```

**۷) `HPA`**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web
  namespace: lab
spec:
  scaleTargetRef: { apiVersion: apps/v1, kind: Deployment, name: web }
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target: { type: Utilization, averageUtilization: 70 }
```

**۸) `PodDisruptionBudget`**

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web
  namespace: lab
spec:
  minAvailable: 2
  selector:
    matchLabels: { app: web }
```

**۹) `NetworkPolicy`**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-allow
  namespace: lab
spec:
  podSelector:
    matchLabels: { app: web }
  policyTypes: [Ingress]
  ingress:
    - from:
        - namespaceSelector:
            matchLabels: { name: ingress-nginx }
      ports:
        - protocol: TCP
          port: 80
```

**۱۰) `Service`**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
  namespace: lab
spec:
  selector: { app: web }
  ports: [{ port: 80, targetPort: 80 }]
```

### مثال‌های عملی

**Lab Scenario: Nginx در K8s با config و TLS**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: web-conf
  namespace: lab
data:
  default.conf: |
    server {
        listen 80;
        server_name lab.example.com;
        root /usr/share/nginx/html;
        index index.html;

        location = /health { access_log off; return 200 "ok\n"; }

        location /api/ {
            proxy_pass http://api:80/;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location / { try_files $uri $uri/ /index.html; }
    }
---
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
        - name: nginx
          image: nginx:1.26-alpine
          ports: [{ containerPort: 80 }]
          volumeMounts:
            - { name: conf,  mountPath: /etc/nginx/conf.d }
            - { name: cache, mountPath: /var/cache/nginx }
            - { name: run,   mountPath: /var/run }
          readinessProbe:
            httpGet: { path: /health, port: 80 }
            initialDelaySeconds: 2
            periodSeconds: 5
          livenessProbe:
            httpGet: { path: /health, port: 80 }
            initialDelaySeconds: 10
            periodSeconds: 10
          resources:
            requests: { cpu: 50m, memory: 64Mi }
            limits:   { cpu: 300m, memory: 192Mi }
      volumes:
        - name: conf
          configMap: { name: web-conf }
        - { name: cache, emptyDir: {} }
        - { name: run,   emptyDir: {} }
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
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web
  namespace: lab
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
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
kubectl apply -f web.yaml
kubectl -n lab rollout status deploy/web
kubectl -n lab get pods,svc,ingress
kubectl -n lab logs deploy/web --tail=20

# تست
curl -sS -H 'Host: lab.example.com' https://<INGRESS-IP>/health
```

**`ConfigMap` تغییر و reload:**

```bash
kubectl -n lab create configmap web-conf --from-file=default.conf \
  --dry-run=client -o yaml | kubectl apply -f -

kubectl -n lab rollout restart deploy/web
kubectl -n lab rollout status deploy/web
```

> Nginx config را در startup می‌خواند؛ `rollout restart` لازم است. یا `reloader` (stakater) با annotation.

**`reloader`:**

```yaml
metadata:
  annotations:
    reloader.stakater.com/auto: "true"
```

**مقیاس:**

```bash
kubectl -n lab scale deploy/web --replicas=5
kubectl -n lab get hpa
```

**`rollout` و rollback:**

```bash
kubectl -n lab set image deploy/web nginx=nginx:1.27-alpine
kubectl -n lab rollout status deploy/web
kubectl -n lab rollout undo deploy/web
```

**`NetworkPolicy`:**

```bash
kubectl -n lab apply -f netpol.yaml
kubectl -n lab describe networkpolicy web-allow
```

**بررسی منابع:**

```bash
kubectl -n lab top pods
kubectl -n lab describe pod <pod>
```

### مقایسه و الگوهای مختلف

**`ConfigMap` در برابر `image` برای config**

| | ConfigMap | image |
|---|---|---|
| تغییر بدون build | ✅ | ❌ |
| بازتولید | ⚠️ | ✅ |
| مناسب | تولید | ✅ CI/CD |

**`Deployment` در برابر `DaemonSet` در برابر `StatefulSet`**

| | Deployment | DaemonSet | StatefulSet |
|---|---|---|---|
| هر node | ❌ | ✅ | ❌ |
| هویت پایدار | ❌ | — | ✅ |
| مناسب | ✅ وب | agent | DB |

**`readinessProbe` در برابر `livenessProbe`**

| | readiness | liveness |
|---|---|---|
| خطا | حذف از LB | restart |
| مناسب | آماده‌بودن | قفل |

**دیدگاه «Nginx در K8s لازم نیست»**: اگر `Ingress` داری، Nginx داخلی برای استاتیک و SPA مفید است. برای API خالص، `Ingress` کافی است.

### اشکال شایع

**۱) `readOnlyRootFilesystem` بدون `emptyDir`**
Nginx بالا نمی‌آید. mount.

**۲) `runAsNonRoot` با پورت ۸۰**
`Permission denied`. `8080` یا `NET_BIND_SERVICE`.

**۳) `ConfigMap` بدون `rollout restart`**
config قدیمی.

**۴) `livenessProbe` روی `/health` وابسته به DB**
restart حلقه‌ای. `readiness` برای DB، `liveness` سبک.

**۵) `resources` نبودن**
`OOMKill` یا `eviction`.

**۶) `NetworkPolicy` بدون `Ingress` از ingress-nginx**
ترافیک بلاک.

**۷) `Ingress` بدون `ingressClassName`**
نادیده.

**۸) `Secret` در git**
خطر. `SealedSecret`/`ExternalSecret`.

**۹) `latest` tag**
غیرقابل بازتولید.

**۱۰) `PDB` نبودن**
`drain` همه‌ی podها را می‌برد.

### نکات مهم (Tips & Tricks)

- **`readinessProbe`**.
- **`emptyDir`** برای پوشه‌ی نوشتنی.
- **`resources`**.
- **`PDB`**.
- **`NetworkPolicy`**.
- **`rollout undo`**.
- **`reloader`**.

### خلاصه فصل

- Nginx در K8s یک Deployment است.
- `ConfigMap` برای config، `Secret` برای TLS.
- `readiness`/`liveness` تفاوت دارند.
- `emptyDir` برای پوشه‌های قابل نوشتن.
- `HPA` و `PDB` برای پایداری.
- `NetworkPolicy` سطح حمله را کم می‌کند.

**سؤال مروری:** چرا `livenessProbe` روی `/health` که DB را چک می‌کند، می‌تواند restart حلقه‌ای بسازد؟

---

## فصل ۷۱: CI/CD، config as code و blue/green

### مقدمه

استقرار دستی، خطا و «works on my machine» می‌آورد. سه اصل: config در git، build یکسان، و استقرار تدریجی.

### مفاهیم اساسی

**۱) `config as code`**

```
repo/
├── nginx/
│   ├── nginx.conf
│   └── conf.d/
├── ansible/
│   └── playbook.yml
├── k8s/
│   └── *.yaml
└── .github/workflows/deploy.yml
```

**۲) سه استراتژی استقرار**

| استراتژی | قطعی | rollback |
|---|---|---|
| Recreate | ❌ | ⚠️ |
| Rolling | ✅ | ✅ |
| Blue/Green | ✅ | ✅✅ |
| Canary | ✅ | ✅ |

**۳) Rolling update**

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate: { maxSurge: 1, maxUnavailable: 0 }
```

**۴) Blue/Green**

```
LB → Blue (v1)  [فعال]
     Green (v2) [آماده]

بعد از تست: LB → Green
Rollback: LB → Blue
```

**۵) Canary**

```
۹۰٪ → v1
۱۰٪ → v2
```

**۶) `nginx` با blue/green**

```nginx
upstream app {
    server 10.0.0.11:8000;   # blue
    # server 10.0.0.12:8000; # green
}
```

تغییر با ویرایش config و `reload`.

**۷) `include` برای سوییچ**

```nginx
# /etc/nginx/conf.d/upstream.conf
include /etc/nginx/active-backend.conf;
```

```bash
# active-backend.conf → blue.conf یا green.conf
sudo ln -sf /etc/nginx/blue.conf /etc/nginx/active-backend.conf
sudo nginx -t && sudo systemctl reload nginx
```

**۸) `symlink` release (Capistrano-style)**

```
/var/www/app/
├── releases/20261022-1400/
├── releases/20261022-1600/
└── current -> releases/20261022-1600/
```

**۹) CI/CD pipeline**

```
lint → test → build → scan → push → deploy staging → smoke → deploy prod
```

**۱۰) `GitHub Actions` نمونه**

```yaml
name: deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build image
        run: docker build -t ghcr.io/${{ github.repository }}:${{ github.sha }} .

      - name: Trivy scan
        uses: aquasecurity/trivy-action@0.24.0
        with:
          image-ref: ghcr.io/${{ github.repository }}:${{ github.sha }}
          severity: HIGH,CRITICAL
          exit-code: '1'

      - name: Push
        run: |
          echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
          docker push ghcr.io/${{ github.repository }}:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /srv/lab
            ./deploy.sh ${{ github.sha }}
```

**۱۱) `deploy.sh` با health check و rollback**

```bash
#!/usr/bin/env bash
set -euo pipefail

TAG=${1:?tag required}
PREV=$(cat .current 2>/dev/null || echo "")

echo "deploying $TAG (prev=$PREV)"

docker pull "ghcr.io/org/lab:$TAG"
echo "$TAG" > .next

# سوییچ
sed -i "s|lab:.*|lab:$TAG|" compose.yaml
docker compose up -d --no-deps web

# health check
for i in $(seq 1 30); do
  if curl -fsS http://127.0.0.1/health >/dev/null; then
    echo "OK"
    mv .next .current
    exit 0
  fi
  sleep 2
done

echo "FAILED, rollback to $PREV"
sed -i "s|lab:.*|lab:$PREV|" compose.yaml
docker compose up -d --no-deps web
exit 1
```

### مثال‌های عملی

**Lab Scenario: blue/green با Nginx**

```bash
# دو نسخه
docker run -d --name app-blue  -p 127.0.0.1:8000:3000 lab-app:1.0
docker run -d --name app-green -p 127.0.0.1:8001:3000 lab-app:2.0
```

```nginx
# /etc/nginx/conf.d/upstream.conf
upstream app {
    server 127.0.0.1:8000;   # blue
    keepalive 16;
}
```

```nginx
# /etc/nginx/conf.d/site.conf
server {
    listen 8080;
    server_name _;

    location / {
        proxy_pass http://app;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        add_header X-Backend "blue" always;
    }
}
```

```bash
sudo nginx -t && sudo systemctl reload nginx
curl -sSI http://localhost:8080/ | grep -i x-backend
# X-Backend: blue
```

**سوییچ:**

```bash
sudo sed -i 's/8000/8001/; s/# blue/# green/' /etc/nginx/conf.d/upstream.conf
sudo nginx -t && sudo systemctl reload nginx
curl -sSI http://localhost:8080/ | grep -i x-backend
# X-Backend: green
```

**Rollback:**

```bash
sudo sed -i 's/8001/8000/; s/# green/# blue/' /etc/nginx/conf.d/upstream.conf
sudo nginx -t && sudo systemctl reload nginx
```

**`symlink` release:**

```bash
sudo mkdir -p /var/www/app/releases
sudo cp -a ./build /var/www/app/releases/20261022-1600
sudo ln -sfn /var/www/app/releases/20261022-1600 /var/www/app/current

# rollback
sudo ln -sfn /var/www/app/releases/20261022-1400 /var/www/app/current
```

**canary با weight:**

```nginx
upstream app {
    server 127.0.0.1:8000 weight=9;   # 90%
    server 127.0.0.1:8001 weight=1;   # 10%
}
```

```bash
for i in $(seq 1 100); do
  curl -sSI http://localhost:8080/ | grep -i x-backend
done | sort | uniq -c
# 90 blue, 10 green
```

**`GitHub Actions` کامل:**

```yaml
name: ci-cd

on:
  push:
    branches: [main]
  pull_request:

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker run --rm -v "$PWD/nginx:/etc/nginx:ro" nginx:1.26-alpine nginx -t

  build:
    needs: lint
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v6
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: deploy
          key: ${{ secrets.SSH_KEY }}
          script: /srv/lab/deploy.sh ${{ github.sha }}

  smoke:
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - run: |
          curl -fsS https://staging.example.com/health
          curl -fsS -o /dev/null -w '%{http_code}\n' https://staging.example.com/

  deploy-prod:
    needs: smoke
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.PROD_HOST }}
          username: deploy
          key: ${{ secrets.SSH_KEY }}
          script: /srv/lab/deploy.sh ${{ github.sha }}
```

**`nginx -t` در CI:**

```yaml
- name: Validate nginx config
  run: |
    docker run --rm \
      -v "$PWD/nginx/nginx.conf:/etc/nginx/nginx.conf:ro" \
      -v "$PWD/nginx/conf.d:/etc/nginx/conf.d:ro" \
      nginx:1.26-alpine nginx -t
```

**`ansible` برای config:**

```yaml
- hosts: web
  become: true
  tasks:
    - name: install nginx
      apt: { name: nginx, state: present, update_cache: true }

    - name: config
      template:
        src: templates/site.conf.j2
        dest: /etc/nginx/sites-available/lab.conf
      notify: reload nginx

    - name: enable site
      file:
        src: /etc/nginx/sites-available/lab.conf
        dest: /etc/nginx/sites-enabled/lab.conf
        state: link
      notify: reload nginx

  handlers:
    - name: reload nginx
      service: { name: nginx, state: reloaded }
```

### مقایسه و الگوهای مختلف

**Rolling در برابر Blue/Green در برابر Canary**

| | Rolling | B/G | Canary |
|---|---|---|---|
| منابع | ✅ | ❌ دو برابر | ✅ |
| rollback | ✅ | ✅✅ | ✅ |
| تست قبل از ترافیک | ❌ | ✅ | ⚠️ |
| مناسب | K8s | VM | ✅ تدریجی |

**`symlink` در برابر `image tag`**

| | symlink | tag |
|---|---|---|
| rollback | ✅ فوری | ⚠️ |
| بازتولید | ⚠️ | ✅ |
| مناسب | VM | ✅ کانتینر |

**`GitHub Actions` در برابر `GitLab CI` در برابر `Jenkins`**

| | GHA | GitLab | Jenkins |
|---|---|---|---|
| راه‌اندازی | ✅✅ | ✅ | ❌ |
| خودمیزبان | ⚠️ | ✅ | ✅ |
| اکوسیستم | ✅ | ✅ | ✅ |

**دیدگاه «CI/CD برای وب‌سرور لازم نیست»**: config دستی، drift و خطا می‌آورد. حتی یک `nginx -t` در CI، از outage جلوگیری می‌کند.

### اشکال شایع

**۱) config دستی روی سرور**
drift. IaC.

**۲) `nginx -t` نبودن در CI**
outage.

**۳) بدون health check در deploy**
نسخه‌ی خراب، ترافیک می‌گیرد.

**۴) بدون rollback**
بحران. `.current`.

**۵) `latest` tag**
غیرقابل بازتولید.

**۶) `Secret` در repo**
خطر. `secrets`.

**۷) blue/green بدون تست**
هر دو خراب.

**۸) `sed` روی config در production**
خطا. template.

**۹) بدون smoke test**
deploy موفق، اپ خراب.

**۱۰) `reload` بدون `-t`**
config خراب، worker قدیمی.

### نکات مهم (Tips & Tricks)

- **`nginx -t` در CI**.
- **health check در deploy**.
- **rollback خودکار**.
- **tag با commit sha**.
- **blue/green برای rollback فوری**.
- **smoke test**.
- **`secrets`** برای رمز.

### خلاصه فصل

- config در git، build یکسان.
- rolling، blue/green، canary.
- `nginx -t` در CI.
- health check و rollback در deploy.
- `symlink`/tag برای نسخه.
- smoke test بعد از استقرار.

**سؤال مروری:** چرا blue/green برای rollback سریع‌تر از rolling است؟

---

## منابع و مراجع

**مستندات رسمی**
- [Docker — nginx official image](https://hub.docker.com/_/nginx)
- [Docker Compose file reference](https://docs.docker.com/compose/compose-file/)
- [Kubernetes — Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Kubernetes — ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Kubernetes — Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [certbot docker](https://hub.docker.com/r/certbot/certbot)
- [GitHub Actions](https://docs.github.com/en/actions)

**آموزش**
- [Nginx in Docker — best practices](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-docker/)
- [Blue/Green deployments](https://martinfowler.com/bliki/BlueGreenDeployment.html)
- [Canary release](https://martinfowler.com/bliki/CanaryRelease.html)
- [The Twelve-Factor App](https://12factor.net/)

**ابزار**
- [trivy](https://github.com/aquasecurity/trivy)
- [hadolint](https://github.com/hadolint/hadolint)
- [kubeconform](https://github.com/yannh/kubeconform)
- [stakater/reloader](https://github.com/stakater/Reloader)

---

**بعدی:** [`17-Troubleshooting.md`](17-Troubleshooting.md) — `502`/`504`، permission، TLS و ابزارها
