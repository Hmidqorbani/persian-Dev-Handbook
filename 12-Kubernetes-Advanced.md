# فایل ۱۲ — کوبرنتیز پیشرفته (Probes، HPA، RBAC، امنیت)

**مدت یادگیری**: ۱۲ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: فایل ۱۱ (مبانی کوبرنتیز)

---

## فهرست محتوا

- [فصل ۵۶: Probes و مدیریت منابع (Liveness، Readiness، QoS)](#فصل-۵۶-probes-و-مدیریت-منابع-liveness-readiness-qos)
- [فصل ۵۷: مقیاس‌پذیری (HPA، VPA، Cluster Autoscaler)](#فصل-۵۷-مقیاسپذیری-hpa-vpa-cluster-autoscaler)
- [فصل ۵۸: RBAC و امنیت کلاستر](#فصل-۵۸-rbac-و-امنیت-کلاستر)
- [فصل ۵۹: مدیریت رمز در کوبرنتیز (Vault، External Secrets)](#فصل-۵۹-مدیریت-رمز-در-کوبرنتیز-vault-external-secrets)
- [فصل ۶۰: استانداردهای امنیتی Pod و سیاست‌ها](#فصل-۶۰-استانداردهای-امنیتی-pod-و-سیاستها)

---

## فصل ۵۶: Probes و مدیریت منابع (Liveness، Readiness، QoS)

### مقدمه

در تولید، «در حال اجرا بودن» کافی نیست — کانتینر ممکن است زنده باشد اما کار نکند (deadlock)، یا هنوز آمادهٔ سرویس‌دهی نباشد. **Probes** به K8s می‌گویند کی Pod سالم، آماده، یا هنوز در حال بوت است. مدیریت **منابع** (CPU/RAM) هم برای ثبات کلاستر حیاتی است. این فصل هر دو را پوشش می‌دهد.

### مفاهیم اساسی

**سه نوع Probe:**

| Probe | سوال | اقدام در شکست |
|---|---|---|
| **Liveness** | «برنامه زنده است؟» | ری‌استارت کانتینر |
| **Readiness** | «آمادهٔ ترافیک است؟» | حذف از چرخهٔ Service (ترافیک نمی‌رسد) |
| **Startup** | «بوت اولیه تمام شد؟» | قبل از آن، liveness/readiness صبر می‌کنند |

**انواع بررسی (Probe mechanism):**
- `httpGet`: درخواست HTTP به مسیر (مثل `/healthz`).
- `exec`: اجرای دستور درون کانتینر.
- `tcpSocket`: چک باز بودن پورت.

**مدیریت منابع:**
- **requests:** حداقل تضمین‌شده (برای زمان‌بندی).
- **limits:** سقف مجاز (فراتر از آن → throttle یا kill).

**QoS (Quality of Service) Classes:**
- **Guaranteed:** requests == limits → اولویت بالا (آخرین حذف در کمبود).
- **Burstable:** requests < limits.
- **BestEffort:** بدون تنظیم → اولین حذف در کمبود منابع.

**OOMKilled:** اگر RAM از limit بگذرد، کانتینر کشته می‌شود (خروج ۱۳۷).

### مثال‌های عملی

**۱. تعریف Probes در Deployment:**

```yaml
containers:
  - name: web
    image: myapp:v1
    ports: [{ containerPort: 8080 }]
    livenessProbe:
      httpGet: { path: /healthz, port: 8080 }
      initialDelaySeconds: 10
      periodSeconds: 10
      failureThreshold: 3
    readinessProbe:
      httpGet: { path: /ready, port: 8080 }
      periodSeconds: 5
    startupProbe:
      httpGet: { path: /healthz, port: 8080 }
      failureThreshold: 30
      periodSeconds: 5   # ۱۵۰ ثانیه فرصت بوت
```

**۲. منابع و QoS:**

```yaml
resources:
  requests:
    cpu: 100m        # ۰.۱ هسته
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 256Mi
```

**۳. تست Probe محلی:**

```bash
kubectl exec web-xxx -- curl -f http://localhost:8080/healthz
# خروجی 200 = سالم
```

**۴. مشاهده وضعیت:**

```bash
kubectl get pods
# READY 1/1 یعنی readiness رد شده یا نه
kubectl describe pod web-xxx | grep -A5 Probe
```

### مقایسه و دیدگاه‌های مختلف

| سناریو | Probe لازم |
|---|---|
| اپلیکیشن وب معمولی | Liveness + Readiness |
| اپلیکیشن با بوت کند | + Startup |
| صف/worker (بدون HTTP) | Exec-based readiness |
| کرش تصادفی | Liveness (ری‌استارت) |

**دیدگاه‌ها:**
- Readiness مهم‌تر از Liveness برای تجربهٔ کاربر است (ترافیک به Pod نیمه‌آماده نرود).
- Startup probe جلوی «death loop» هنگام بوت کند را می‌گیرد.
- همیشه requests/limits بگذارید (BestEffort در تولید ممنوع).

**تجارت:** limits خیلی کم → OOMKill مکرر؛ خیلی زیاد → هدر رفت منابع.

### اشتباهات رایج

- **یکسان گرفتن Liveness و Readiness:** اگر readiness شکست، ترافیک قطع شود کافی است؛ نباید ری‌استارت شود (Liveness جدا باشد).
- **Liveness روی دیتابیس پایدار:** اگر DB چند ثانیه کند شود، Pod ری‌استارت می‌شود → بدتر.
- **فراموش initialDelay در Liveness:** قبل از راه‌اندازی اپ، پاد ری‌استارت می‌شود (یا از startup probe استفاده کنید).
- **بدون requests/limits:** Pod BestEffort می‌شود → اولین قربانی.
- **Probe مسیر اشتباه:** `/` را چک می‌کند که همیشه ۲۰۰ است (deadlock را نمی‌بیند).

### نکات مهم و ترفندها

- **Probe جداگانه برای عمق:** `/healthz` (زنده بودن) در برابر `/ready` (وابستگی‌ها اوکی‌اند؟).
- **exec probe برای workerها:** `pg_isready` برای DB، `redis-cli ping` برای Redis.
- **CPU limit احتیاطی:** throttle شدن CPU باعث latency می‌شود؛ برخی ترجیح می‌دهند فقط memory limit بگذارند.
- **VPA recommendation:** (فصل ۵۷) مقادیر مناسب را پیشنهاد می‌دهد.
- **Pod PriorityClass:** برای اپلیکیشن‌های حیاتی اولویت بالا بدهید.
- **`kubectl top pods`:** مصرف واقعی را مانیتور کنید.

### خلاصه فصل

Probes سلامت را تعریف می‌کنند: Liveness (ری‌استارت در شکست)، Readiness (خروج از چرخهٔ ترافیک)، Startup (فرصت بوت). مدیریت منابع با requests/limits کلاستر را پایدار نگه می‌دارد و QoS تعیین می‌کند چه کسی در کمبود حذف شود. کلید: Readiness و Liveness را جدا تعریف کنید، مسیرهای معنادار برای probe بدهید، و همیشه requests/limits (حداقل memory) تنظیم کنید تا OOMKill و BestEffort اجتناب شود.

---

## فصل ۵۷: مقیاس‌پذیری (HPA، VPA، Cluster Autoscaler)

### مدمه

ترافیک متغیر است — ساعت شلوغی ۱۰ برابر ساعت خلوت. **مقیاس‌پذیری خودکار** (Autoscaling) اجازه می‌دهد اپلیکیشن و کلاستر بدون دخالت انسان بزرگ یا کوچک شوند. در این فصل سه لایهٔ autoscaling را بررسی می‌کنیم: Pods، کلاستر، و منابع.

### مفاهیم اساسی

**سه لایه Autoscaling:**

| لایه | ابزار | مقیاس‌بندی |
|---|---|---|
| Pod | **HPA** (Horizontal Pod Autoscaler) | تعداد رپلیکا |
| Pod (منابع) | **VPA** (Vertical Pod Autoscaler) | CPU/RAM هر Pod |
| Node | **Cluster Autoscaler** / Karpenter | تعداد ماشین |

**HPA (افقی):**
بر اساس متریک (CPU٪، حافظه، یا سفارشی مثل RPS)، تعداد رپلیکا را بین `minReplicas` و `maxReplicas` تنظیم می‌کند. نیاز به **Metrics Server** دارد.

**VPA (عمودی):**
منابع (requests/limits) هر Pod را بالا/پایین می‌برد. معمولاً با restart Pod (در تولید با HPA تداخل دارد — معمولاً یکی را انتخاب کنید).

**Cluster Autoscaler:**
وقتی Podها به دلیل کمبود منابع Schedule نمی‌شوند، نود جدید اضافه می‌کند؛ وقتی نود خالی ماند، حذف می‌کند.

**Karpenter (جایگزین مدرن):** به جای گروه‌های نود ثابت، نود را بر اساس نیاز دقیق و به صورت سریع و ارزان‌تر provision می‌کند (AWS-originated، حالا multi-cloud).

**Metrics Server:** جمع‌آوری متریک‌های مصرف از kubelet (پیش‌نیاز HPA).

### مثال‌های عملی

**۱. نصب Metrics Server:**

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl top nodes   # تست
```

**۲. HPA بر اساس CPU:**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata: { name: web-hpa }
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

**۳. HPA بر اساس متریک سفارشی (مثل RPS):**

```yaml
metrics:
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "100"
```

**۴. Cluster Autoscaler (در ابر):**

```bash
# روی EKS/GKE معمولاً فعال است؛ پیکربندی از طریق node group
# وقتی Podها Pending ماندند، نود اضافه می‌شود
kubectl get nodes -w
```

### مقایسه و دیدگاه‌های مختلف

| ابزار | بهترین برای | نکته |
|---|---|---|
| HPA | ترافیک متغیر (وب) | نیاز به Metrics Server |
| VPA | بار پیش‌بینی‌پذیر | با HPA تداخل (mode: "Off" برای recommendations) |
| Cluster Autoscaler | مقیاس نودها | وابسته به ابر |
| Karpenter | بهینه‌سازی هزینه/سرعت | مدرن‌تر |

**دیدگاه:** معمولاً **HPA + Cluster Autoscaler** ترکیب استاندارد است. VPA بیشتر برای بهینه‌سازی منابع (توصیه) استفاده می‌شود تا autoscaling واقعی. Karpenter جایگزین قدرتمند Cluster Autoscaler در ابرهای پشتیبانی‌شده است.

**HPA vs VPA:** انتخاب یکی — هر دو با هم روی CPU تداخل ایجاد می‌کنند.

### اشتباهات رایج

- **HPA بدون requests:** HPA نمی‌داند ۷۰٪ از چیست (باید CPU request ست باشد).
- **minReplicas=1:** در ترافیک، یک نمونه زیر بار می‌شکند؛ حداقل ۲ بگذارید.
- **اهداف غیرواقعی:** target ۳۰٪ CPU باعث نوسان (thrashing) می‌شود.
- **فراموش Metrics Server:** HPA همیشه `<unknown>` نشان می‌دهد.
- **VPA و HPA هم‌زمان روی CPU:** تداخل → استفاده از VPA در mode Off.

### نکات مهم و ترفندها

- **`kubectl get hpa`:** وضعیت مقیاس‌بندی را ببینید.
- **Custom Metrics:** با Prometheus Adapter متریک سفارشی (مثل latency) بدهید.
- **PodDisruptionBudget:** هنگام autoscale-down، حداقل Pod در دسترس را تضمین کنید.
- **Karpenter:** برای کاهش هزینه و سرعت provisioning نود.
- **`behavior` در HPA:** سرعت scale-up/down را کنترل کنید (مثلاً آرام‌تر scale-down).
- **Load testing:** با `k6`/`locust` حد واقعی HPA را پیدا کنید.

### خلاصه فصل

Autoscaling سه لایه دارد: HPA (تعداد Pod بر اساس متریک)، VPA (منابع هر Pod)، و Cluster Autoscaler/Karpenter (تعداد نود). HPA نیازمند Metrics Server و تنظیم CPU request است. کلید: minReplicas ≥ ۲، target واقع‌بینانه (~۷۰٪)، PodDisruptionBudget برای جلوگیری از قطع سرویس هنگام scale-down، و انتخاب بین HPA یا VPA (نه هر دو روی CPU). Karpenter جایگزین مدرن برای نودها.

---

## فصل ۵۸: RBAC و امنیت کلاستر

### مدمه

هر کاربر یا سرویس در K8s باید فقط به آنچه نیاز دارد دسترسی داشته باشد (**Least Privilege**). **RBAC** (Role-Based Access Control) این را مدیریت می‌کند. در این فصل مدل دسترس K8s و چگونگی ایمن‌سازی آن را یاد می‌گیرید.

### مفاهیم اساسی

**موجودیت‌های RBAC:**

| شیء | کاربرد |
|---|---|
| **ServiceAccount (SA)** | هویت برای اپلیکیشن/Pod (نه انسان) |
| **Role** | مجوزهای در یک Namespace |
| **ClusterRole** | مجوزهای در کل کلاستر |
| **RoleBinding** | اتصال Role به کاربر/SA (در یک NS) |
| **ClusterRoleBinding** | اتصال ClusterRole به سطح کلاستر |

**جریان:** کاربر/SA ← (Binding) ← Role/ClusterRole ← (قوانین) ← منابع + عملیات.

**عملیات (verbs):** `get`, `list`, `watch`, `create`, `update`, `patch`, `delete`.

**گروه‌های داخلی:**
- `system:admin` (دسترس کامل).
- `view` (فقط خواندن).
- `edit` (ویرایش بدون مدیریت RBAC).
- `cluster-admin` (خطرناک — خیلی کم استفاده کنید).

**kubeconfig و احراز:**
انسان‌ها با kubeconfig (گواهی یا OIDC) احراز می‌شوند؛ Podها با ServiceAccount (توکن خودکار mount شده در `/var/run/secrets/...`).

**امنیت کلاستر:**
- **API Server exposure:** هرگز اینترنتی بدون AuthN/AuthZ.
- **Pod Security (فصل ۶۰):** محدود کردن قابلیت‌های کانتینر.
- **NetworkPolicy:** (فصل ۵۵) ایزولاسیون ترافیک.

### مثال‌های عملی

**۱. ساخت ServiceAccount و Role:**

```yaml
apiVersion: v1
kind: ServiceAccount
metadata: { name: deployer, namespace: app }
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata: { name: deployer-role, namespace: app }
rules:
  - apiGroups: [apps]
    resources: [deployments]
    verbs: [get, list, watch, update, patch]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata: { name: deployer-binding, namespace: app }
subjects:
  - kind: ServiceAccount
    name: deployer
    namespace: app
roleRef:
  kind: Role
  name: deployer-role
  apiGroup: rbac.authorization.k8s.io
```

**۲. استفاده در Pod:**

```yaml
spec:
  serviceAccountName: deployer   # به جای پیش‌فرض
  automountServiceAccountToken: false   # اگر نیاز ندارد
```

**۳. بررسی دسترس (مفهومی):**

```bash
kubectl auth can-i create deployments --as=system:serviceaccount:app:deployer
# yes
```

**۴. محدود کردن kubectl دستی:**

```bash
# به جای cluster-admin، از Role محدود برای CI استفاده کنید
kubectl create role ci-deployer --verb=get,list,watch,update,patch --resource=deployments -n app
```

### مقایسه و دیدگاه‌های مختلف

| دسترس | کاربرد | ریسک |
|---|---|---|
| cluster-admin | اضطراری فقط | بسیار بالا |
| edit | توسعه‌دهنده | متوسط |
| view | مشاهده | کم |
| ServiceAccount محدود | CI/CD | کم (ایده‌آل) |

**دیدگاه:** هرگز token cluster-admin را به CI یا اپلیکیشن ندهید. همیشه ServiceAccount با کمترین دسترس لازم. برای SSO، از OIDC (مثل GitHub/Google) با تعریف RoleBinding برای گروه‌ها استفاده کنید.

**معماری Zero-Trust:** فرض کنید هر Pod می‌تواند مخدوش شود → محدودترین RBAC + NetworkPolicy.

### اشتباهات رایج

- **استفاده از `cluster-admin` برای CI:** نشت توکن = کنترل کل کلاستر.
- **`automountServiceAccountToken: true` پیش‌فرض:** Podهایی که نیاز ندارند هم توکن دارند.
- **RoleBinding اشتباه namespace:** در NS اشتباه → دسترس نمی‌دهد.
- **گروه‌های بزرگ در Binding:** کاربران بیش از نیاز دسترس می‌گیرند.
- **فراموش حذف Binding قدیمی:** دسترس ماندگار می‌شود (دردسر امنیتی).

### نکات مهم و ترفندها

- **`kubectl auth can-i`** برای تست دسترس (قبل از اعطا).
- **محدودسازی با OIDC:** گروه‌های IdP را به RoleBinding نگاشت کنید.
- **Rotate توکن SA:** در K8s جدید خودکار (Bound Service Account Token).
- **Audit logging:** فعال کنید تا چه کسی چه کرد را ببینید.
- **`kubectl get rolebindings -A`:** پایش دسترس‌ها.
- **Pod Security Admission** (فصل ۶۰) مکمل RBAC برای ایمنی کانتینر.

### خلاصه فصل

RBAC دسترس را از طریق Role/ClusterRole و Binding به کاربر/ServiceAccount می‌دهد. اصل: همیشه Least Privilege — ServiceAccount با کمترین دسترس لازم برای CI/اپلیکیشن (نه cluster-admin). کلید: `automountServiceAccountToken: false` وقتی نیاز نیست، تست با `auth can-i`، و استفاده از OIDC برای SSO. RBAC + NetworkPolicy + Pod Security = دفاع در عمق (defense-in-depth).

---

## فصل ۵۹: مدیریت رمز در کوبرنتیز (Vault، External Secrets)

### مدمه

K8s Secret به تنهایی ناکافی است (فقط base64، خطر نشت در etcd و Git). برای تولید واقعی، سیستم‌های مدیریت رمز مثل **HashiCorp Vault** و الگوی **External Secrets** استاندارد هستند. در این فصل چرخهٔ ایمن رمز در K8s را می‌سازیم.

### مفاهیم اساسی

**مشکل K8s Secret:**
- فقط base64 (encoding، نه encryption).
- اگر Encryption-at-Rest غیرفعال باشد، plaintext در etcd.
- وقتی در Git (manifest) قرار گیرد، لو می‌رود.
- چرخش (rotation) دستی و دردناک.

**راه‌حل‌های لایه‌بندی شده:**
1. **Encryption-at-Rest:** رمزنگاری etcd (حداقل).
2. **Sealed Secrets:** رمز را encrypted در Git نگه می‌دارد؛ در کلاستر باز می‌شود (GitOps-safe).
3. **External Secrets Operator (ESO):** Secretهای خارجی (Vault، AWS SM، GCP SM) را به K8s Secret همگام‌سازی می‌کند.
4. **Vault:** سیستم مدیریت رمز کامل (صدور پویای توکن، دور زدن، audit).

**HashiCorp Vault:**
- انبار رمز متمرکز.
- **Dynamic Secrets:** توکن‌های کوتاه‌مدت (مثل کاربر DB موقت).
- **PKI:** صدور گواهی خودکار.
- **Integration:** از طریق Vault Agent یا CSI Provider توکن را به Pod می‌دهد.

**External Secrets Operator:**
- `ExternalSecret` منبع رمز خارجی را تعریف می‌کند.
- به صورت دوره‌ای/رویدادیSecret را به‌روزرسانی می‌کند.
- اجازه می‌دهد رمزها در Git (به صورت reference) باشند بدون لو رفتن.

### مثال‌های عملی

**۱. فعال‌سازی Encryption-at-Rest:**

```bash
# در kube-apiserver: --encryption-provider-config=enc.yaml
# enc.yaml:
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources: [secrets]
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <BASE64_KEY>
```

**۲. External Secrets با AWS Secrets Manager:**

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata: { name: db-secret }
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-store
    kind: ClusterSecretStore
  data:
    - secretKey: password
      remoteRef:
        key: prod/db
        property: password
# نتیجه: یک K8s Secret به نام db-secret ساخته می‌شود
```

**۳. استفاده در Pod (مثل قبل):**

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef: { name: db-secret, key: password }
```

**۴. Vault Agent Injector (مفهومی):**

```yaml
annotations:
  vault.hashicorp.com/agent-inject: "true"
  vault.hashicorp.com/role: "myapp"
  vault.hashicorp.com/secret: "secret/data/myapp"
# Vault Agent توکن را به عنوان فایل تزریق می‌کند
```

### مقایسه و دیدگاه‌های مختلف

| رویکرد | مزیت | مناسب برای |
|---|---|---|
| K8s Secret + Encryption | ساده | شروع |
| Sealed Secrets | GitOps-safe | تیم‌های Git-based |
| External Secrets (ESO) | همگام با Vault/SM | تولید |
| Vault (کامل) | Dynamic, Audit, PKI | سازمانی پیچیده |

**دیدگاه:** برای اکثر، **ESO + Secrets Manager ابری** تعادل عالی است. Vault برای نیازهای پیشرفته (dynamic DB creds، PKI) ارزش دارد اما نگهداری سنگین. Sealed Secrets برای تیم‌هایی که GitOps دارند و ابر SM ندارند مناسب است.

**اصل:** رمز هرگز در Git plaintext نباشد؛ و چرخش خودکار داشته باشید.

### اشتباهات رایج

- **رمز در Helm values/plain YAML:** لو رفتن در Git.
- **فرض امن بودن K8s Secret:** بدون Encryption-at-Rest، etcd plaintext است.
- **توکن ثابت طولانی:** نشت = خطر دائمی؛ از dynamic/short-lived استفاده کنید.
- **ExternalSecret بدون refresh:** رمز چرخش‌یافته در Vault منعکس نمی‌شود.
- **دسترس بیش از حد به Vault:** هر Pod نباید همهٔ مسیرهای Vault را ببیند (policy جدا).

### نکات مهم و ترفندها

- **Sealed Secrets:** `kubeseal` رمز را encrypt می‌کند؛ فقط controller می‌تواند باز کند.
- **Vault CSI Provider:** mount کردن رمز به عنوان فایل (بدون env).
- **Rotation خودکار:** ESO `refreshInterval` را تنظیم کنید.
- **GitOps + SOPS:** رمز را در Git با SOPS (age/GPG) رمزنگاری کنید.
- **Audit Vault:** همهٔ دسترس‌ها لاگ شوند.
- **Namespaced SecretStore:** محدود کردن دسترس به منبع رمز.

### خلاصه فصل

K8s Secret به تنهایی ناامن است (base64، خطر etcd/Git). لایه‌های امنیت: Encryption-at-Rest (حداقل)، Sealed Secrets (GitOps-safe)، External Secrets Operator (همگام با Vault/AWS SM)، و Vault کامل (dynamic secrets، PKI، audit). کلید: رمز هرگز در Git plaintext، چرخش خودکار، و Least Privilege در دسترس به منبع رمز. ESO + Cloud SM برای اکثر تیم‌ها تعادل ایده‌آل؛ Vault برای نیازهای سازمانی پیچیده.

---

## فصل ۶۰: استانداردهای امنیتی Pod و سیاست‌ها

### مدمه

حتی با RBAC و Secret ایمن، یک Pod می‌تواند با دسترس بیش از حد (مثل اجرای privileged) سیستم را به خطر بیندازد. **Pod Security Standards (PSS)** و موتورهای سیاست (OPA/Gatekeeper) اجازه می‌دهند الزامات امنیتی را در سطح کلاستر اجباری کنیم. این فصل پایان‌بندی امنیت K8s است.

### مفاهیم اساسی

**Pod Security Standards (PSS):**
سه سطح سخت‌گیری:
- **Privileged:** آزاد (فقط سیستمی).
- **Baseline:** محدودیت‌های حداقلی (مقابله با شایع‌ترین آسیب‌پذیری).
- **Restricted:** سخت‌گیرانه (تولید توصیه‌شده).

**Pod Security Admission (PSA):**
جایگزین قدیمی **PodSecurityPolicy (PSP)** (حذف شده در ۱.۲۵). سطح PSS را در Namespace با label اعمال می‌کند:

```yaml
# Namespace
metadata:
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

**OPA Gatekeeper / Kyverno:**
موتورهای **Policy-as-Code**. قوانین سفارشی (مثل «همهٔ imageها باید از رجیستری داخلی باشند» یا «همه must runAsNonRoot») را اجباری می‌کنند. Gatekeeper از Rego (OPA) استفاده می‌کند؛ Kyverno YAML-friendly است.

**ایدهٔ دفاع در عمق (Defense in Depth):**
امنیت یک لایه نیست: RBAC + NetworkPolicy + PSS + Image Signing + Runtime Security همگی با هم.

**قوانین رایج Restricted:**
- `runAsNonRoot: true`
- `seccompProfile: RuntimeDefault`
- `allowPrivilegeEscalation: false`
- دسترس به `capabilities` محدود
- فقط `readOnlyRootFilesystem` (ترجیحاً)

### مثال‌های عملی

**۱. اعمال PSS روی Namespace:**

```bash
kubectl label ns production \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version=latest
# Pod ناامن رد می‌شود:
# Error: forbidden: violates PodSecurity "restricted:latest"
```

**۲. Pod امن (Restricted-compliant):**

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  allowPrivilegeEscalation: false
  seccompProfile:
    type: RuntimeDefault
containers:
  - name: app
    securityContext:
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
```

**۳. Kyverno Policy (مثال: اجبار non-root):**

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata: { name: require-non-root }
spec:
  validationFailureAction: enforce
  rules:
    - name: check-user
      match: { resources: { kinds: [Pod] } }
      validate:
        message: "Pods must run as non-root"
        pattern:
          spec:
            securityContext:
              runAsNonRoot: true
```

**۴. OPA Gatekeeper Constraint (مفهومی):**

```yaml
# Rego-based constraint: image فقط از رجیستری داخلی
# (نیاز به ConstraintTemplate + Constraint)
```

### مقایسه و دیدگاه‌های مختلف

| ابزار | زبان | کاربرد |
|---|---|---|
| Pod Security Admission | برچسب (ساده) | PSS پایه |
| Kyverno | YAML | سیاست‌های ساده/متوسط |
| OPA Gatekeeper | Rego | سیاست پیچیده/سازمانی |

**دیدگاه:** برای شروع، PSA با `restricted` کافی است. برای کنترل‌های سفارشی (مثل الزام label یا منبع image)، Kyverno (ساده‌تر) یا Gatekeeper (قدرتمندتر) اضافه کنید. دفاع در عمق: همه را با هم.

**تجارت:** Restricted ممکن است برخی اپلیکیشن‌های قدیمی را بشکند — آن‌ها را در Namespace جدا با Baseline قرار دهید.

### اشتباهات رایج

- **Privileged در تولید:** دستریر کامل هسته → escape آسان.
- **فراموش `runAsNonRoot`:** کانتینر با root اجرا می‌شود.
- **PSA روی `warn` only:** فقط هشدار می‌دهد، اجبار نمی‌کند (از `enforce` استفاده کنید).
- **یک سطح برای همه:** اپلیکیشن سیستمی (مثل CNI) نیاز به Privileged دارد → Namespace جدا.
- **فقط PSS، بدون NetworkPolicy:** ترافیک داخلی هنوز آزاد است.

### نکات مهم و ترفندها

- **Audit mode اول:** PSA را با `audit`/`warn` شروع کنید تا ببینید چه Podهایی می‌شکنند، سپس `enforce`.
- **Kyverno `validate`/`mutate`:** می‌تواند خودکار مقدار پیش‌فرض (مثل `runAsNonRoot`) بدهد (mutate).
- **ImagePolicy:** Kyverno/Gatekeeper اجبار کند image امضا شده (فصل ۴۳).
- **CIS Benchmark:** چک‌لیست کامل امنیت K8s.
- **Falco:** تشخیص نفوذ در زمان اجرا (runtime security).
- **Namespaces منطقی:** تفکیک `production` (restricted) از `system` (privileged).

### خلاصه فصل

Pod Security Standards سه سطح (Privileged/Baseline/Restricted) دارند که با Pod Security Admission برچسبی روی Namespace اعمال می‌شوند (جایگزین PSP حذف‌شده). Kyverno (YAML) و OPA Gatekeeper (Rego) سیاست‌های سفارشی را اجباری می‌کنند. کلید: `enforce: restricted` در production، Podهای compliant (runAsNonRoot، drop capabilities، seccomp، readOnlyRootFS)، و Audit-first برای جلوگیری از شکست ناگهانی. امنیت = دفاع در عمق (RBAC + NetworkPolicy + PSS + Signing + Runtime).

---

## منابع و مراجع

- Kubernetes Probes: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes
- HPA: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
- VPA: https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler
- Cluster Autoscaler: https://github.com/kubernetes/autoscaler
- Karpenter: https://karpenter.sh
- RBAC: https://kubernetes.io/docs/reference/access-authn-authz/rbac/
- Pod Security Standards: https://kubernetes.io/docs/concepts/security/pod-security-standards/
- External Secrets Operator: https://external-secrets.io
- HashiCorp Vault: https://www.vaultproject.io
- Kyverno: https://kyverno.io
- OPA Gatekeeper: https://open-policy-agent.github.io/gatekeeper/

**بعدی:** [فایل ۱۳ — Helm و Kustomize: مدیریت پیکربندی کوبرنتیز](13-Helm-and-Kustomize.md)
