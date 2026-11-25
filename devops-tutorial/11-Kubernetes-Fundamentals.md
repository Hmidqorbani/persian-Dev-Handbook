# فایل ۱۱ — مبانی کوبرنتیز (Kubernetes)

**مدت یادگیری**: ۱۲ ساعت
**سطح سختی**: متوسط–پیشرفته
**پیش‌نیاز**: فایل ۰۸، ۰۹، ۱۰ (داکر و پادمن)، فایل ۰۴ (CI/CD)

---

## فهرست محتوا

- [فصل ۵۱: معماری Kubernetes (Control Plane و Node)](#فصل-۵۱-معماری-kubernetes-control-plane-و-node)
- [فصل ۵۲: اشیاء پایه (Pod، ReplicaSet، Deployment، Service)](#فصل-۵۲-اشیاء-پایه-pod-replicaset-deployment-service)
- [فصل ۵۳: پیکربندی و تنظیمات (ConfigMap، Secret، Volume، PV/PVC)](#فصل-۵۳-پیکربندی-و-تنظیمات-configmap-secret-volume-pvpvc)
- [فصل ۵۴: kubectl و عیب‌یابی](#فصل-۵۴-kubectl-و-عیبیابی)
- [فصل ۵۵: شبکه و سرویس‌دهی (Service و Ingress)](#فصل-۵۵-شبکه-و-سرویسدهی-service-و-ingress)

---

## فصل ۵۱: معماری Kubernetes (Control Plane و Node)

### مقدمه

وقتی کانتینرها زیاد می‌شوند، مدیریت دستی آن‌ها غیرممکن است. **Kubernetes** (K8s) استاندارد صنعت برای **ارکستراسیون کانتینر** (Orchestration) است — زمان‌بندی، مقیاس‌پذیری، خودترمیمی (self-healing) و شبکه را خودکار می‌کند. در این فصل معماری کلی آن را کالبدشکافی می‌کنیم.

### مفاهیم اساسی

**Kubernetes چیست؟**
یک سیستم متن‌باز برای **اتوماسیون استقرار، مقیاس‌بندی و مدیریت اپلیکیشن‌های کانتینری**. مدل «desired state» (وضعیت مطلوب) دارد: شما می‌گویید «۳ نمونه بخواهم» و K8s آن را تأمین و حفظ می‌کند.

**معماری کلی:**

```
┌─────────────────────────────────────┐
│         Control Plane (Master)       │
│  API Server | Scheduler | Controller│
│  etcd (database)                    │
└──────────────┬──────────────────────┘
               │ (watch/notify)
   ┌───────────┼───────────┐
┌──▼──┐    ┌────▼────┐  ┌────▼────┐
│ Node │    │  Node   │  │  Node   │
│ kubelet│  │ kubelet │  │ kubelet │
│ kube-proxy│ container│  │         │
└──────┘    └─────────┘  └─────────┘
```

**اجزای Control Plane:**
- **API Server:** درگاه ارتباط (REST) — همه چیز از اینجا می‌گذرد.
- **etcd:** پایگاه دادهٔ کلید-مقدار (state کلاستر).
- **Scheduler:** تصمیم می‌گیرد هر Pod کجا اجرا شود.
- **Controller Manager:** حلقه‌های کنترل (ReplicaSet، Node، و...) را اجرا می‌کند.
- **(Cloud) Controller Manager:** یکپارچه‌سازی با ابر (LB، disks).

**اجزای Node (Worker):**
- **kubelet:** عامل روی هر نود که Podها را مدیریت می‌کند.
- **kube-proxy:** قوانین شبکه (iptables/IPVS) برای Service.
- **Container Runtime:** containerd یا cri-o (داکر دیگر پیش‌فرض نیست).

**Cluster چیست؟** مجموعه‌ای از نودها (ماشین‌ها) که با هم یک سیستم یکپارچه می‌سازند.

**Namespace:** فضای نام برای تفکیک منطقی منابع (مثل `default`، `kube-system`، `production`).

### مثال‌های عملی

**۱. نصب محلی (kind / minikube):**

```bash
# با kind
kind create cluster --name dev
kubectl cluster-info

# با minikube
minikube start --driver=docker
```

**۲. بررسی اجزا:**

```bash
kubectl get nodes
kubectl get pods -n kube-system
kubectl api-resources   # لیست انواع اشیاء
```

**۳. ساختار Control Plane (مفهومی):**

```bash
# etcd را مستقیماً نبینید؛ اما وضعیت کلاستر:
kubectl get componentstatuses
```

**۴. افزودن Node (مفهومی):**

```
Master ارسال توکن → Worker اجرای kubeadm join → kubelet ثبت می‌شود
```

### مقایسه و دیدگاه‌های مختلف

| ابزار ارکستراسیون | توضیح |
|---|---|
| Kubernetes | استاندارد صنعت، غنی |
| Docker Swarm | ساده، سبک، کم‌ویژگی |
| Nomad (HashiCorp) | ساده‌تر، multi-workload |
| ECS (AWS) | مدیریت‌شده AWS |

**دیدگاه‌ها:**
- K8s پیچیده اما قدرتمندترین است؛ برای تولید مقیاس‌پذیر استاندارد است.
- برای تیم‌های کوچک، managed K8s (EKS/GKE/AKS) بهتر از self-hosted است.
- Swarm برای موارد بسیار ساده کافی است اما اکوسیستم کوچک دارد.

**مدیریت‌شده vs خودمیزبان:** EKS/GKE/AKS درد نگهداری Control Plane را می‌گیرند — توصیه برای اکثر.

### اشتباهات رایج

- **داکر به عنوان runtime:** از K8s ۱.۲۴ به بعد، داکر مستقیم پشتیبانی نمی‌شود (containerd/cri-o استفاده کنید).
- **دستکاری مستقیم etcd:** فقط از طریق API Server تغییر دهید.
- **اجرای workload روی master:** در تولید، master را taint کنید تا Pod کاربر روی آن نرود.
- **فهم نادرست desired state:** حذف Pod دستی → K8s دوباره می‌سازد (باید Deployment را حذف کنید).
- **نادیده گرفتن Namespace:** همه چیز در `default` → تداخل و عدم ایزولاسیون.

### نکات مهم و ترفندها

- **Kind/Minikube** برای یادگیری محلی عالی‌اند.
- **Managed K8s:** EKS/GKE/AKS برای تولید (بدون نگهداری etcd).
- **kubectl autocomplete:** `source <(kubectl completion bash)`.
- **`--dry-run=client -o yaml`:** تولید منیفست بدون اعمال.
- **Contexts:** `kubectl config` برای سوییچ بین کلاسترها.
- **k9s:** ابزار TUI برای مدیریت بصری کلاستر.

### خلاصه فصل

Kubernetes سیستم ارکستراسیون کانتینر با مدل desired state است. Control Plane (API Server، etcd، Scheduler، Controller) وضعیت را مدیریت می‌کند؛ Nodeها (kubelet، kube-proxy، containerd) کار را اجرا می‌کنند. Namespace منابع را تفکیک می‌کند. کلید: مدیریت‌شده (EKS/GKE/AKS) برای تولید، containerd به جای داکر، و درک اینکه حذف Pod دستی باعث بازسازی می‌شود (Deployment را حذف کنید).

---

## فصل ۵۲: اشیاء پایه (Pod، ReplicaSet، Deployment، Service)

### مقدمه

هر چیزی در K8s یک «شیء» (Object) است که با فایل YAML توصیف می‌شود. درک اشیاء پایه — Pod، ReplicaSet، Deployment، Service — پیش‌نیاز هر کار عملی است. در این فصل این اشیاء و رابطه‌شان را یاد می‌گیرید.

### مفاهیم اساسی

**ساختار منیفست YAML:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: web
          image: nginx:alpine
          ports:
            - containerPort: 80
```

**Pod:**
کوچک‌ترین واحد قابل استقرار. یک یا چند کانتینر با شبکه/حجم مشترک. معمولاً Pod را مستقیم نمی‌سازید — از Deployment استفاده کنید.

**ReplicaSet:**
تضمین می‌کند N نمونه از Pod همیشه در حال اجرا باشد (self-healing و scaling).

**Deployment:**
نسخهٔ سطح‌بالای ReplicaSet. مدیریت **rollout** (انتشار تدریجی) و **rollback** را دارد. پرکاربردترین شیء.

**Service:**
آدرس پایدار (ثابت) برای گروهی از Podها. Podها می‌آیند و می‌روند (IP عوض می‌شود) اما Service ثابت می‌ماند. انواع: ClusterIP، NodePort، LoadBalancer، ExternalName.

**Label & Selector:**
برچسب‌ها (Labels) کلید اتصال اشیاء هستند. Service با `selector` Podهای مناسب را پیدا می‌کند.

### مثال‌های عملی

**۱. ساخت Deployment:**

```bash
kubectl create deployment myapp --image=nginx:alpine --replicas=3
kubectl get deployments
kubectl get pods -o wide
```

**۲. اکسپوز کردن با Service:**

```bash
kubectl expose deployment myapp --port=80 --type=ClusterIP
# یا مستقیم در YAML (فصل ۵۵)
```

**۳. مقیاس‌بندی:**

```bash
kubectl scale deployment myapp --replicas=5
# یا با تغییر YAML + kubectl apply
```

**۴. Rollout و Rollback:**

```bash
kubectl set image deployment/myapp nginx=nginx:1.25
kubectl rollout status deployment/myapp
# خراب شد؟ برگردان:
kubectl rollout undo deployment/myapp
# یا به نسخهٔ خاص:
kubectl rollout undo deployment/myapp --to-revision=2
```

**۵. نمونهٔ کامل YAML:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: web }
spec:
  replicas: 3
  selector: { matchLabels: { app: web } }
  template:
    metadata: { labels: { app: web } }
    spec:
      containers:
        - name: web
          image: nginx:alpine
          resources:
            requests: { cpu: 100m, memory: 128Mi }
            limits:   { cpu: 500m, memory: 256Mi }
```

### مقایسه و دیدگاه‌های مختلف

| شیء | کاربرد | توصیه |
|---|---|---|
| Pod | تک‌کانتینر (آزمایش) | نه در تولید |
| ReplicaSet | تکرار Pod | غیرمستقیم (via Deployment) |
| Deployment | اپلیکیشن stateless | ✅ استاندارد |
| StatefulSet | داده‌دار (DB) | برای stateful apps |
| DaemonSet | یک نمونه در هر نود | لاگ/مانیتورینگ |
| Service | اتصال پایدار | ضروری |

**دیدگاه:** همیشه از Deployment (نه Pod مستقیم) برای اپلیکیشن‌های وب استفاده کنید. برای داده‌دار (database) از StatefulSet + PVC استفاده کنید.

**Imperative vs Declarative:** `kubectl run` (imperative) برای تست سریع؛ `kubectl apply -f` (declarative) برای تولید (GitOps — فصل ۱۴).

### اشتباهات رایج

- **ساخت Pod مستقیم در تولید:** با خرابی حذف می‌شود و بازسازی نمی‌شود.
- **فراموش `selector` matchLabels:** Deployment Pod را پیدا نمی‌کند.
- **عدم تنظیم resources:** Podها منابع نامحدود می‌گیرند → noisy neighbor.
- **تغییر imperative بعد از declarative:** با `apply` تداخل می‌کند؛ همیشه YAML نگه دارید.
- **حذف Pod به جای Deployment:** Pod دوباره ساخته می‌شود.

### نکات مهم و ترفندها

- **`kubectl apply -f dir/`:** اعمال همهٔ YAMLهای یک پوشه.
- **`--record`:** تاریخچه rollout را با دستور ثبت کنید (برای دیباگ).
- **`kubectl diff -f`:** قبل از apply تفاوت را ببینید.
- **Labels استاندارد:** `app`, `tier`, `env` را استفاده کنید.
- **Resource quotas:** در Namespace محدودیت بگذارید.
- **Liveness/Readiness probes:** (فصل ۱۲) برای سلامت ضروری.

### خلاصه فصل

اشیاء پایه K8s: Pod (واحد کوچک)، ReplicaSet (تکرار)، Deployment (مدیریت rollout/rollback — استاندارد تولید)، و Service (آدرس پایدار). همه با YAML توصیف می‌شوند و با Labels/Selectors به هم متصل می‌شوند. کلید: همیشه Deployment برای stateless، تنظیم resources (requests/limits)، و استفاده از `apply` (declarative) به جای دستورات imperative در تولید.

---

## فصل ۵۳: پیکربندی و تنظیمات (ConfigMap، Secret، Volume، PV/PVC)

### مدمه

کد نباید تنظیمات را سخت‌کد (hard-code) کند. K8s ابزارهای جدا برای **پیکربندی** (ConfigMap)، **رمز** (Secret) و **ذخیرهٔ دائم** (PV/PVC) دارد. در این فصل این اشیاء را برای جداسازی کانفیگ از کد یاد می‌گیرید.

### مفاهیم اساسی

**ConfigMap:**
ذخیرهٔ دادهٔ غیرحساس (مثل تنظیمات اپلیکیشن، فایل‌های conf). به Pod به صورت متغیر محیطی یا فایل (volume) تزریق می‌شود.

**Secret:**
مثل ConfigMap اما برای دادهٔ حساس (رمز، توکن، کلید). در etcd **رمزنگاری‌شده نیست** (base64 است — نه امن!) مگر با Encryption-at-Rest فعال شود. همیشه با RBAC و Encryption محافظت کنید.

**Volume:**
دیسک قابل اتصال به Pod. انواع: `emptyDir` (موقت)، `hostPath` (پوشه هاست — خطرناک)، `configMap`/`secret` (تزریق تنظیمات)، `persistentVolumeClaim`.

**PV (PersistentVolume) و PVC (PersistentVolumeClaim):**
- **PV:** منبع ذخیرهٔ واقعی (دیسک ابری، NFS) — توسط ادمین یا dynamic provisioning.
- **PVC:** درخواست کاربر برای فضا («۵ گیگ بده»). K8s PV مناسب را متصل می‌کند.
- **StorageClass:** تعریف نوع ذخیره (مثل `gp3` در AWS) برای ایجاد خودکار PV.

**Best practice:** کد را از کانفیگ جدا کنید (12-factor app). رمز هرگز در image یا ConfigMap.

### مثال‌های عملی

**۱. ساخت ConfigMap:**

```bash
kubectl create configmap app-config \
  --from-literal=LOG_LEVEL=info \
  --from-file=config.yaml=./app.conf
```

```yaml
# یا YAML
apiVersion: v1
kind: ConfigMap
metadata: { name: app-config }
data:
  LOG_LEVEL: "info"
  config.yaml: |
    server:
      port: 8080
```

**۲. استفاده در Pod:**

```yaml
env:
  - name: LOG_LEVEL
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: LOG_LEVEL
# یا به صورت volume
volumeMounts:
  - name: cfg
    mountPath: /etc/app
volumes:
  - name: cfg
    configMap:
      name: app-config
```

**۳. Secret:**

```bash
kubectl create secret generic db-secret \
  --from-literal=password=supersecret
```

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: password
```

**۴. PV/PVC:**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata: { name: db-pvc }
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 10Gi
  storageClassName: gp3
---
# در Pod:
volumes:
  - name: data
    persistentVolumeClaim:
      claimName: db-pvc
```

### مقایسه و دیدگاه‌های مختلف

| شیء | داده | رمزنگاری |
|---|---|---|
| ConfigMap | غیرحساس | نه |
| Secret | حساس | base64 (نه رمزنگاری!) |
| PV/PVC | دیسک دائم | بسته به زیرساخت |

**دیدگاه‌ها:**
- Secret در K8s فقط مخفی‌سازی سطحی است؛ حتماً **Encryption-at-Rest** در etcd فعال کنید.
- برای رمزهای واقعی، از **External Secrets Operator** (یکپارچه با Vault/AWS Secrets Manager) استفاده کنید (فصل ۱۷/IaC).
- ConfigMap در تست/تولید متفاوت باشد — جداسازی محیط.

**Sealed Secrets / SOPS:** رمز را در Git (encrypted) نگه دارید و در کلاستر باز کنید (GitOps-safe).

### اشتباهات رایج

- **رمز در ConfigMap:** اشتباه امنیتی؛ از Secret استفاده کنید.
- **فکر کردن base64 = رمزنگاری:** base64 فقط encoding است، هر کسی می‌تواند decode کند.
- **hostPath در تولید:** وابستگی به نود خاص → Pod جابه‌جا شود، داده گم می‌شود.
- **PVC بدون StorageClass:** در کلاستر بدون default، Pending می‌ماند.
- **حجم زیاد در Secret:** محدودیت اندازه دارد (~۱MB)؛ برای فایل بزرگ از mount خارجی.

### نکات مهم و ترفندها

- **Encryption-at-Rest:** پیکربندی `kube-apiserver --encryption-provider-config`.
- **External Secrets Operator:** همگام‌سازی Vault/AWS SM با K8s Secret.
- **ConfigMap as volume hot-reload:** برخی اپلیکیشن‌ها تغییر فایل را خودکار می‌خوانند.
- **`kubectl exec` برای تست:** متغیر محیطی را چک کنید.
- **ResourceQuota + LimitRange:** محدودیت منابع در Namespace.
- **SubPath:** فقط یک فایل از ConfigMap را mount کنید بدون بازنویسی پوشه.

### خلاصه فصل

ConfigMap برای تنظیمات غیرحساس، Secret برای رمزها (فقط base64 — نه واقعی رمزنگاری‌شده؛ Encryption-at-Rest فعال کنید)، و PV/PVC برای ذخیرهٔ دائم (ایده‌آل برای stateful). کد را از کانفیگ جدا کنید (12-factor). کلید: هرگز رمز در ConfigMap/Image، از External Secrets/Vault برای رمز واقعی، و اجتناب از hostPath در تولید. این پایهٔ DevSecOps و GitOps است.

---

## فصل ۵۴: kubectl و عیب‌یابی

### مقدمه

`kubectl` چاقوی سویسی شما در K8s است. اما دانستن دستورات اجرایی کافی نیست — عیب‌یابی واقعی نیازمند رویکرد لایه‌ای است. در این فصل دستورات کلیدی و متدولوژی دیباگ سیستماتیک را می‌آموزید.

### مفاهیم اساسی

**دستورات پایه:**

| دستور | کاربرد |
|---|---|
| `kubectl get` | لیست اشیاء |
| `kubectl describe` | جزئیات + Events |
| `kubectl logs` | لاگ کانتینر |
| `kubectl exec` | اجرای دستور درون کانتینر |
| `kubectl apply` | اعمال YAML |
| `kubectl delete` | حذف |
| `kubectl explain` | توضیح فیلدهای منیفست |

**جریان عیب‌یابی لایه‌ای:**
1. **Pod status:** `kubectl get pods` → وضعیت (Pending/Running/CrashLoopBackOff).
2. **Events:** `kubectl describe pod` → چرا schedule نشد؟ (منابع؟ node؟).
3. **Logs:** `kubectl logs` → خطای اپلیکیشن.
4. **Exec:** `kubectl exec -it` → بررسی محیط درونی.
5. **Network:** `kubectl port-forward` یا تست از Pod دیگر.
6. **Cluster:** `kubectl get events --sort-by=.lastTimestamp`.

**وضعیت‌های رایج Pod:**
- `Pending`: منتظر نود/منابع.
- `ImagePullBackOff`: image پیدا نشد یا احراز اشتباه.
- `CrashLoopBackOff`: اپلیکیشن مدام crash می‌کند.
- `OOMKilled`: حافظه بیش از limit.
- `Running` اما `0/1 Ready`: readiness probe شکست.

**Port-forward:** دسترسی محلی به سرویس برای تست:
```bash
kubectl port-forward svc/myapp 8080:80
```

### مثال‌های عملی

**۱. عیب‌یابی Pod گیر کرده:**

```bash
kubectl get pods
# NAME     READY  STATUS             RESTARTS
# web-xxx  0/1    CrashLoopBackOff  5

kubectl describe pod web-xxx
# Events: FailedScheduling, Back-off restarting

kubectl logs web-xxx --previous   # لاگ قبل از کرش
kubectl logs web-xxx -c init-container  # اگر چند کانتینر
```

**۲. اجرای دستور درون کانتینر:**

```bash
kubectl exec -it web-xxx -- sh
# بررسی:
#   env | grep DB
#   cat /etc/app/config.yaml
#   curl http://localhost:8080/healthz
```

**۳. مشاهده Events کلاستر:**

```bash
kubectl get events --sort-by=.lastTimestamp
kubectl get events -n production --field-selector type=Warning
```

**۴. تست شبکه داخلی:**

```bash
kubectl run debug --rm -it --image=busybox -- sh
# درون: wget -qO- http://myapp.default.svc.cluster.local
```

**۵. تغییر سریع برای تست:**

```bash
kubectl set env deployment/web LOG_LEVEL=debug
kubectl rollout restart deployment/web
```

### مقایسه و دیدگاه‌های مختلف

| ابزار | کاربرد |
|---|---|
| kubectl | استاندارد CLI |
| k9s | TUI تعاملی |
| Lens / Octant | GUI دسکتاپ/وب |
| kubectl-tree | نمایش سلسله‌مراتب اشیاء |
| stern | لاگ چند Pod هم‌زمان |

**دیدگاه:** kubectl برای خودکارسازی/اسکریپت ضروری است؛ GUIها (Lens/k9s) برای درک بصری و دیباگ سریع. هر دو را بلد باشید.

**متدولوژی:** همیشه از «وضعیت» شروع کنید، نه از حدس. `describe` + `logs` ۹۰٪ مشکلات را حل می‌کنند.

### اشتباهات رایج

- **فقط `logs` دیدن بدون `describe`:** علت (مثل FailedScheduling) را از دست می‌دهید.
- **`--previous` فراموش کردن:** لاگ کرش قبلی برای دیباگ حیاتی است.
- **exec روی Pod که Running نیست:** باید حداقل یک بار start شود؛ از initContainer دیباگ کنید.
- **حذف Pod به جای Deployment:** دوباره ساخته می‌شود.
- **فرض شبکه درست:** بدون تست connectivity از Pod دیگر، DNS مشکل را نادیده می‌گیرید.

### نکات مهم و ترفندها

- **`kubectl explain pod.spec.containers`:** مستندات فیلد را در CLI ببینید.
- **`kubectl debug`:** ساخت Pod دیباگ با ابزار اضافه (ephemeral debug container).
- **`kubectl get all`:** دید کلی (اما Secrets را نشان نمی‌دهد — عمدی).
- **`--field-selector`:** فیلتر رویدادها/Podها.
- **Aliases:** `k=kubectl`, `kgp='kubectl get pods'` و غیره.
- **Context switch:** `kubectl config use-context prod` برای محیط‌های مختلف.

### خلاصه فصل

`kubectl` ابزار اصلی مدیریت K8s است. عیب‌یابی لایه‌ای: ابتدا `get pods` (وضعیت)، سپس `describe` (Events)، `logs`/`--previous` (خروجی اپ)، `exec` (محیط درونی)، و تست شبکه. وضعیت‌های رایج: Pending (منابع)، ImagePullBackOff (احراز/image)، CrashLoopBackOff (اپ crash)، OOMKilled (حافظه). کلید: همیشه از وضعیت شروع کنید نه حدس، و `describe`+`logs` ۹۰٪ مشکلات را حل می‌کنند.

---

## فصل ۵۵: شبکه و سرویس‌دهی (Service و Ingress)

### مدمه

در K8s، Podها Ephemeral هستند و IPشان عوض می‌شود. **Service** آدرس پایدار می‌دهد، و **Ingress** ترافیک خارجی را مسیریابی می‌کند. درک شبکه در K8s برای هر DevOps ضروری است. این فصل انواع Service و Ingress را بررسی می‌کند.

### مفاهیم اساسی

**انواع Service:**

| نوع | دسترسی | کاربرد |
|---|---|---|
| **ClusterIP** | فقط داخل کلاستر | ارتباط داخلی (پیش‌فرض) |
| **NodePort** | `<NodeIP>:<Port>` از خارج | تست/دسترس ساده |
| **LoadBalancer** | IP خارجی (ابر) | تولید ساده (یک IP برای هر سرویس) |
| **ExternalName** | DNS alias | اشاره به سرویس خارجی |

**Service چطور کار می‌کند؟**
با `selector` Podهای مطابق label را پیدا می‌کند و Endpoint می‌سازد. kube-proxy ترافیک را به Podهای سالم (طبق readiness) هدایت می‌کند (load balancing داخلی).

**Ingress:**
به جای یک LoadBalancer برای هر سرویس (گران)، Ingress یک **نقطهٔ ورودی واحد** است که ترافیک را بر اساس **مسیر (path)** یا **دامنه (host)** به سرویس‌های مختلف هدایت می‌کند. نیازمند **Ingress Controller** (مثل NGINX، Traefik، HAProxy).

**DNS داخلی:**
سرویس‌ها با `<service>.<namespace>.svc.cluster.local` قابل دسترس‌اند.

**NetworkPolicy:**
فایروال سطح کلاستر — کنترل کند کدام Pod با کدام صحبت کند (پیش‌فرض همه به هم وصل‌اند).

### مثال‌های عملی

**۱. Service از نوع ClusterIP:**

```yaml
apiVersion: v1
kind: Service
metadata: { name: web }
spec:
  selector: { app: web }
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

**۲. NodePort:**

```yaml
spec:
  type: NodePort
  ports:
    - port: 80
      nodePort: 30080
```

**۳. LoadBalancer (در ابر):**

```yaml
spec:
  type: LoadBalancer
# ابر یک IP خارجی می‌دهد
```

**۴. Ingress:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api
                port: { number: 80 }
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port: { number: 80 }
```

**۵. NetworkPolicy (محدود کردن ترافیک):**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: { name: db-only-from-api }
spec:
  podSelector: { matchLabels: { app: db } }
  policyTypes: [Ingress]
  ingress:
    - from:
        - podSelector:
            matchLabels: { app: api }
      ports:
        - port: 5432
```

### مقایسه و دیدگاه‌های مختلف

| روش دسترسی | هزینه | مناسب برای |
|---|---|---|
| ClusterIP | رایگان | داخلی |
| NodePort | رایگان | تست |
| LoadBalancer | یک IP/سرویس | تولید ساده |
| Ingress | یک IP کل | تولید چندسرویس |

**دیدگاه:** برای تولید، **Ingress + Ingress Controller** استاندارد است (صرفه‌جویی در IP). LoadBalancer فقط برای سرویس‌های بسیار حساس یا موارد خاص. NetworkPolicy باید همیشه تعریف شود (zero-trust داخلی).

**Service Mesh (فصل ۲۵):** برای ترافیک پیچیده (mTLS، retries)، Istio/Linkerd جایگزین/مکمل Ingress است.

### اشتباهات رایج

- **فراموش `selector` در Service:** Endpoint ساخته نمی‌شود (ترافیک بلوک).
- **targetPort اشتباه:** ترافیک به پورت اشتباه Pod می‌رود.
- **LoadBalancer برای همه:** هزینه و پیچیدگی زیاد؛ از Ingress استفاده کنید.
- **بدون Ingress Controller:** Ingress فقط تعریف است؛ Controller باید نصب باشد.
- **عدم تنظیم NetworkPolicy:** همه Podها به هم دسترس دارند (ریسک).

### نکات مهم و ترفندها

- **Headless Service** (`clusterIP: None`): برای سرویس‌های stateful (Direct Pod DNS).
- **`externalTrafficPolicy: Local`:** حفظ IP کاربر اصلی در لاگ.
- **TLS در Ingress:** با cert-manager گواهی خودکار (Let's Encrypt).
- **Port-forward برای تست:** `kubectl port-forward svc/web 8080:80`.
- **`kubectl get endpoints`:** بررسی Podهای متصل به Service.
- **مولتی-Ingress:** ingressClassName برای جدا کردن کنترلرها.

### خلاصه فصل

Service آدرس پایدار برای Podهای متغیر است: ClusterIP (داخلی)، NodePort (تست)، LoadBalancer (IP خارجی در ابر)، ExternalName (DNS alias). Ingress یک نقطهٔ ورودی واحد برای مسیریابی بر اساس host/path است که Ingress Controller (NGINX/Traefik) آن را پیاده می‌کند — صرفه‌جویی در IP. NetworkPolicy ترافیک داخلی را محدود می‌کند (zero-trust). کلید: همیشه selector+targetPort درست، Ingress برای تولید چندسرویس، و NetworkPolicy برای امنیت.

---

## منابع و مراجع

- Kubernetes Documentation: https://kubernetes.io/docs/
- Kubernetes Concepts: https://kubernetes.io/docs/concepts/
- kubectl Reference: https://kubernetes.io/docs/reference/kubectl/
- Podman/K8s relationship: https://kubernetes.io/docs/setup/production-environment/container-runtimes/
- Ingress Controllers: https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/
- NetworkPolicy: https://kubernetes.io/docs/concepts/services-networking/network-policy/
- kind (local K8s): https://kind.sigs.k8s.io
- minikube: https://minikube.sigs.k8s.io
- K8s API Reference (kubectl explain): built-in

**بعدی:** [فایل ۱۲ — کوبرنتیز پیشرفته (Probes، HPA، RBAC، Secret Management)](12-Kubernetes-Advanced.md)
