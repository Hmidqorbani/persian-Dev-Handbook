# فایل ۱۳ — Helm و Kustomize: مدیریت پیکربندی کوبرنتیز

**مدت یادگیری**: ۱۰ ساعت
**سطح سختی**: متوسط–پیشرفته
**پیش‌نیاز**: فایل ۱۱، ۱۲ (مبانی و پیشرفته کوبرنتیز)

---

## فهرست محتوا

- [فصل ۶۱: چالش پیکربندی K8s و معرفی ابزارها](#فصل-۶۱-چالش-پیکربندی-k8s-و-معرفی-ابزارها)
- [فصل ۶۲: Kustomize (Overlay، Base و بازنویسی)](#فصل-۶۲-kustomize-overlay-base-و-بازنویسی)
- [فصل ۶۳: Helm مبانی (Chart، Template و values)](#فصل-۶۳-helm-مبانی-chart-template-و-values)
- [فصل ۶۴: Helm پیشرفته (Hooks، Dependencies، Helmfile)](#فصل-۶۴-helm-پیشرفته-hooks-dependencies-helmfile)
- [فصل ۶۵: مقایسه و انتخاب (Kustomize در برابر Helm)](#فصل-۶۵-مقایسه-و-انتخاب-kustomize-در-برابر-helm)

---

## فصل ۶۱: چالش پیکربندی K8s و معرفی ابزارها

### مدمه

وقتی یک اپلیکیشن چندین منیفست (Deployment، Service، Ingress، ConfigMap...) دارد و باید در محیط‌های مختلف (dev/staging/prod) با تفاوت‌های کوچک اجرا شود، مدیریت دستی YAMLها به کابوس تبدیل می‌شود. **Kustomize** و **Helm** دو رویکرد متفاوت برای حل این مشکل هستند. این فصل مسئله را تعریف می‌کند.

### مفاهیم اساسی

**مشکلات YAML دستی:**
- **تکرار (Duplication):** همان Deployment در dev/prod با تفاوت جزئی.
- **مدیریت محیط:** تغییر image tag، replica count، resource limits در هر محیط.
- **نسخه‌بندی:** بازگشت به نسخهٔ قبلی سخت است.
- **بسته‌بندی:** توزیع یک اپلیکیشن به دیگران (مثل نرم‌افزار متن‌باز).

**دو فلسفه:**

| رویکرد | ابزار | روش |
|---|---|---|
| **Templating** (الگوسازی) | **Helm** | متغیرها در template → YAML تولید می‌شود |
| **Overlay/Patch** (لایه‌بندی) | **Kustomize** | YAML پایه + پچ‌های محیطی → خروجی |

**Helm = Package Manager:**
مثل `apt`/`npm` برای K8s. یک **Chart** (بسته) شامل templates + values است. جامعهٔ بزرگ (Artifact Hub) دارد.

**Kustomize = YAML-native:**
بدون زبان قالب‌بندی جدید — فقط YAML + پچ. بخشی از `kubectl` (از ۱.۱۴ به بعد `kubectl apply -k`).

**Declarative vs Templated:** Kustomize کاملاً declarative (شما می‌بینید خروجی چیست)؛ Helm از طریق template (خروجی را قبل از apply باید رندر ببینید).

### مثال‌های عملی

**۱. سناریوی بدون ابزار (تکرار):**

```
deploy/dev/web.yaml     # replica: 1, image: myapp:dev
deploy/prod/web.yaml    # replica: 5, image: myapp:1.2.3, resources...
# تغییر در یکی → باید در هر دو اصلاح شود (فراموش‌شدنی)
```

**۲. Helm حل می‌کند:**

```bash
helm install myapp ./chart --set replicaCount=5 --set image.tag=1.2.3
```

**۳. Kustomize حل می‌کند:**

```bash
kubectl apply -k overlays/prod/   # خروجی ترکیب base + patch
```

**۴. بررسی ابزارها:**

```bash
kubectl kustomize overlays/prod/   # فقط نمایش (بدون apply)
helm template myapp ./chart        # رندر template بدون install
```

### مقایسه و دیدگاه‌های مختلف

| معیار | Helm | Kustomize |
|---|---|---|
| یادگیری | متوسط (Go template) | آسان (YAML) |
| بسته‌بندی | عالی (chart) | ضعیف |
| محیط‌های چندگانه | values/override | overlay |
| خوانایی خروجی | نیاز به template | مستقیم |
| اکوسیستم | Artifact Hub | کتابخانه محلی |

**دیدگاه‌ها:**
- Kustomize برای تیم‌هایی که می‌خواهند «فقط YAML» و کنترل کامل داشته باشند ایده‌آل است.
- Helm برای توزیع نرم‌افزار (مثل Databases، Prometheus) و اکوسیستم آماده عالی است.
- بسیاری هر دو را ترکیب می‌کنند: Helm chart را با Kustomize post-render می‌کنند.

**GitOps:** هر دو با ArgoCD/Flux (فصل ۱۴) عالی کار می‌کنند.

### اشتباهات رایج

- **کپی‌کردن YAML برای هر محیط:** تکرار = انحراف (drift).
- **فراموش تنظیم image tag:** `latest` در prod (فصل ۴۲).
- **Helm بدون `helm template` قبل از apply:** خطای template در کلاستر کشف می‌شود.
- **Kustomize پیچیده با jsonPatch عجیب:** خوانایی از بین می‌رود.
- **عدم نسخه‌بندی values:** بازگشت به نسخهٔ پیکربندی سخت.

### نکات مهم و ترفندها

- **`helm template | kubectl apply`:** رویکرد declarative با Helm (بدون Tiller).
- **Kustomize در kubectl:** نیاز به ابزار جدا ندارد (`-k` flag).
- **GitOps:** هر دو را در Git نگه دارید (source of truth).
- **`helm-docs`:** مستندسازی خودکار values.
- **Kustomize `configMapGenerator`:** هش خودکار برای rollout هنگام تغییر ConfigMap.

### خلاصه فصل

مدیریت YAML دستی در محیط‌های متعدد منجر به تکرار و انحراف می‌شود. Helm (templating/package manager) و Kustomize (overlay/patch) دو راه‌حل هستند. Helm برای بسته‌بندی و توزیع (Artifact Hub) قوی است؛ Kustomize برای YAML-native و کنترل محیطی ساده‌تر. کلید: از تکرار اجتناب کنید، خروجی را قبل از apply ببینید (template/kustomize)، و همه را در Git نگه دارید (GitOps).

---

## فصل ۶۲: Kustomize (Overlay، Base و بازنویسی)

### مدمه

Kustomize رویکرد **پچ‌محور** دارد: شما یک **Base** (منیفست‌های مشترک) دارید و **Overlays** (لایه‌های محیطی) که آن را بازنویسی می‌کنند. هیچ زبان template جدیدی یاد نمی‌گیرید — فقط YAML. در این فصل ساختار و کاربرد عملی را می‌بینیم.

### مفاهیم اساسی

**ساختار پوشه:**

```
app/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── replica-patch.yaml
    └── prod/
        ├── kustomization.yaml
        └── resources-patch.yaml
```

**`kustomization.yaml` (در base):**
لیست منیفست‌ها و منابع مشترک.

**`kustomization.yaml` (در overlay):**
به base اشاره می‌کند (`resources: [../../base]`) و پچ‌ها را اعمال می‌کند.

**انواع بازنویسی (Patch):**
- **`patchesStrategicMerge`:** ادغام هوشمند (مثل merge در kubectl).
- **`patchesJson6902`:** پچ JSON استاندارد (add/replace/remove).
- **`patches:` (جدید):** ترکیبی، ساده‌تر.

**قابلیت‌های کلیدی:**
- **`namePrefix` / `nameSuffix`:** پیشوند/پسوند به نام همهٔ منابع.
- **`commonLabels`:** برچسب مشترک.
- **`configMapGenerator` / `secretGenerator`:** تولید خودکار با هش (برای rollout).
- **`images`:** تغییر tag image در همه جا.

### مثال‌های عملی

**۱. Base `kustomization.yaml`:**

```yaml
# base/kustomization.yaml
resources:
  - deployment.yaml
  - service.yaml
```

**۲. Overlay dev:**

```yaml
# overlays/dev/kustomization.yaml
resources:
  - ../../base
namePrefix: dev-
patches:
  - target:
      kind: Deployment
      name: web
    patch: |
      - op: replace
        path: /spec/replicas
        value: 1
images:
  - name: myapp
    newTag: dev
```

**۳. Overlay prod (با منابع بیشتر):**

```yaml
# overlays/prod/kustomization.yaml
resources:
  - ../../base
namePrefix: prod-
patches:
  - target: { kind: Deployment, name: web }
    patch: |
      - op: replace
        path: /spec/replicas
        value: 5
  - target: { kind: Deployment, name: web }
    patch: |
      - op: add
        path: /spec/template/spec/containers/0/resources
        value:
          requests: { cpu: 200m, memory: 256Mi }
          limits:   { cpu: 500m, memory: 512Mi }
images:
  - name: myapp
    newTag: "1.2.3"
```

**۴. اعمال:**

```bash
kubectl apply -k overlays/prod/
# یا فقط دیدن:
kubectl kustomize overlays/prod/
```

### مقایسه و دیدگاه‌های مختلف

| ویژگی | Kustomize | Helm |
|---|---|---|
| زبان | YAML خالص | Go template |
| بسته‌بندی | خیر | بله (chart) |
| یادگیری | آسان | متوسط |
| Debugging | مستقیم (kustomize) | نیاز به template |

**دیدگاه:** Kustomize برای تیم‌های پلتفرم که می‌خواهند Git را source of truth داشته باشند و از پیچیدگی template دوری کنند، بهترین است. Overlay pattern برای چند محیط بی‌نظیر است.

**محدودیت:** منطق شرطی پیچیده (if/else) در Kustomize سخت است — برای آن Helm بهتر است.

### اشتباهات رایج

- **مسیرهای اشتباه در overlay:** `../../base` باید نسبت به overlay باشد.
- **فراموش `resources` در kustomization:** خروجی خالی می‌شود.
- **استفاده از `replace` روی فیلد موجود نیست:** خطا؛ از `add` استفاده کنید.
- **پچ شکسته YAML:** تورفتگی در patch باید دقیق باشد.
- **عدم استفاده از `configMapGenerator`:** تغییر ConfigMap باعث رول‌اوت خودکار نمی‌شود.

### نکات مهم و ترفندها

- **`kubectl kustomize` برای پیش‌نمایش:** همیشه قبل از apply ببینید.
- **`namePrefix` برای تفکیک:** dev-web، prod-web از تداخل جلوگیری می‌کند.
- **Base به اشتراک‌گذاری:** چند اپلیکیشن می‌توانند یک base داشته باشند.
- **`vars` (deprecated):** از `replacements` جدید استفاده کنید.
- **GitOps:** ArgoCD/Flux مستقیماً kustomization را می‌خوانند.
- **Generator options:** `disableNameSuffixHash: true` برای پایداری نام.

### خلاصه فصل

Kustomize با الگوی Base + Overlays پیکربندی محیطی را بدون زبان template جدید مدیریت می‌کند. Base منیفست‌های مشترک، Overlay پچ‌های محیطی (replicas، resources، image tag) را اعمال می‌کند. ابزارهایی مثل `namePrefix` و `configMapGenerator` (با هش خودکار) کار را ساده می‌کنند. کلید: ساختار پوشه درست، `kubectl kustomize` برای پیش‌نمایش، و اجتناب از منطق شرطی پیچیده (برای آن Helm). با GitOps عالی ترکیب می‌شود.

---

## فصل ۶۳: Helm مبانی (Chart، Template و values)

### مدمه

Helm «apt برای Kubernetes» است. یک **Chart** بسته‌ای شامل templates (با Go template syntax) و مقادیر (values) است که خروجی YAML می‌دهد. در این فصل ساختار Chart و نحوهٔ پارامتریک کردن منیفست‌ها را یاد می‌گیرید.

### مفاهیم اساسی

**ساختار Chart:**

```
mychart/
├── Chart.yaml          # متادیتا (نام، نسخه، dependencies)
├── values.yaml         # مقادیر پیش‌فرض
├── charts/             # زیرچارت‌ها (subcharts)
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── _helpers.tpl    # تعاریف مشترک (template partials)
│   └── NOTES.txt       # پیام پس از install
└── templates/tests/    # تست‌ها
```

**مفاهیم Go Template در Helm:**
- `{{ .Values.replicaCount }}` مقدار را از values می‌خواند.
- `{{ .Release.Name }}` نام release فعلی.
- `{{ .Chart.Version }}` نسخهٔ chart.
- دستورات کنترل: `{{- if .Values.ingress.enabled }}` ... `{{- end }}`.
- توابع: `default`, `quote`, `indent`, `toYaml`, `required`.

**Render flow:**
`values.yaml` (پیش‌فرض) ← `--set` / `-f values-prod.yaml` (override) → Template رندر → YAML → `kubectl apply`.

**Release:** یک نمونهٔ نصب‌شده از chart با نام خاص (مثل `helm install myapp ./chart`).

**Helm Repository:** مخزن chartها (مثل Artifact Hub، ChartMuseum).

### مثال‌های عملی

**۱. Chart.yaml:**

```yaml
apiVersion: v2
name: myapp
description: A Helm chart for MyApp
version: 0.1.0
appVersion: "1.2.3"
```

**۲. values.yaml:**

```yaml
replicaCount: 1
image:
  repository: myapp
  tag: "1.0.0"
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
  port: 80
resources: {}
ingress:
  enabled: false
```

**۳. templates/deployment.yaml (بخش):**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-app
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: app
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          {{- if .Values.resources }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          {{- end }}
```

**۴. نصب و بررسی:**

```bash
helm install myapp ./mychart
helm upgrade myapp ./mychart --set replicaCount=3
helm template myapp ./mychart   # دیدن خروجی
helm list                         # لیست releases
helm uninstall myapp
```

### مقایسه و دیدگاه‌های مختلف

| ویژگی | Helm | Kustomize |
|---|---|---|
| Template engine | Go template (قدرتمند) | پچ (محدود) |
| Conditionals | `if/else` قوی | سخت |
| Packaging | chart + repo | فقط فایل |
| Loops | `range` | نه |

**دیدگاه:** Helm برای نرم‌افزارهای پیچیده با منطق شرطی و توزیع بین تیم‌ها عالی است. Chartهای آماده (Prometheus، Postgres) ماه‌ها کار را ذخیره می‌کنند. اما برای اپلیکیشن داخلی ساده، Kustomize سبک‌تر است.

**Best practice:** Chartهای خودتان را کوچک و قابل فهم نگه دارید؛ از `_helpers.tpl` برای تکرار استفاده کنید.

### اشتباهات رایج

- **فراموش `--namespace` هنگام install:** در `default` نصب می‌شود.
- **مقادیر حساس در values.yaml:** رمز را در chart نگذارید (از Secret/External Secrets استفاده کنید — فصل ۵۹).
- **تورفتگی اشتباه در template:** `nindent`/`indent` اشتباه → YAML خراب.
- **`helm install` دوباره با همان نام:** خطای "already exists"؛ از `upgrade --install` استفاده کنید.
- **عدم استفاده از `helm template`:** خطاها در کلاستر کشف می‌شوند.

### نکات مهم و ترفندها

- **`helm upgrade --install`:** idempotent (نصب یا ارتقا در یک دستور).
- **`helm lint`:** اعتبارسنجی chart قبل از انتشار.
- **`helm dependency`:** مدیریت subcharts (`helm dependency build`).
- **`required` function:** اجبار مقادیر حیاتی: `{{ required "image.tag is required" .Values.image.tag }}`.
- **`helm secrets`:** (پلاگین) مدیریت values رمزنگاری‌شده.
- **Artifact Hub:** https://artifacthub.io برای چارت‌های آماده.

### خلاصه فصل

Helm با Chart (بستهٔ templates + values) پیکربندی را پارامتریک می‌کند. Go template syntax (`{{ .Values.x }}`, `if/range`) قدرت بالایی برای منطق شرطی/حلقه می‌دهد. جریان: values (پیش‌فرض + override) → template رندر → apply. کلید: `helm template`/`lint` قبل از apply، `upgrade --install` برای idempotency، رمز هرگز در values، و استفاده از `_helpers.tpl` برای DRY. Helm برای بسته‌بندی و توزیع (Artifact Hub) بی‌رقیب است.

---

## فصل ۶۴: Helm پیشرفته (Hooks، Dependencies، Helmfile)

### مدمه

Helm فراتر از نصب ساده است: **Hooks** اجازه می‌دهد در مراحل خاص (قبل/بعد از install) کارهایی (مثل migration DB) انجام دهید؛ **Dependencies** چارت‌های ترکیبی می‌سازد؛ و **Helmfile** مدیریت چندین release را خودکار می‌کند. این فصل سطح پیشرفته را پوشش می‌دهد.

### مفاهیم اساسی

**Helm Hooks:**
مثل lifecycle eventها. با annotation `helm.sh/hook` روی منیفست تعریف می‌شوند:
- `pre-install` / `post-install`
- `pre-upgrade` / `post-upgrade`
- `pre-delete` / `post-delete`
- `crashloopbackoff` / `test` (برای تست release)

معمولاً برای **Database Migration** (قبل از ارتقا) یا **Job**های یک‌باره استفاده می‌شود.

**Chart Dependencies (Subcharts):**
یک chart می‌تواند chart دیگر را به عنوان وابستگی داشته باشد (مثل اپلیکیشن شما + PostgreSQL). در `Chart.yaml` تعریف می‌شود:
```yaml
dependencies:
  - name: postgresql
    version: 12.x.x
    repository: https://charts.bitnami.com/bitnami
```

**Helmfile:**
ابزار خارج از Helm برای مدیریت **چندین release** با یک فایل YAML. برای محیط‌های پیچیده (مثل GitOps یا CI) عالی است:
```yaml
releases:
  - name: web
    chart: ./charts/web
    values: [values/web.yaml]
  - name: api
    chart: ./charts/api
```

**Library Charts:** چارت‌هایی که فقط `_helpers.tpl` دارند (برای اشتراک منطق بین چارت‌ها).

**Helm Plugins:** `helm secrets` (رمزنگاری values)، `helm-diff` (مقایسه قبل/بعد)، `helm-git` (چارت از Git).

### مثال‌های عملی

**۱. Hook برای Migration DB:**

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migrate
  annotations:
    "helm.sh/hook": pre-upgrade
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      containers:
        - name: migrate
          image: myapp:migrate
          command: ["alembic", "upgrade", "head"]
      restartPolicy: Never
```

**۲. Dependency در Chart.yaml:**

```yaml
apiVersion: v2
name: myapp
version: 0.2.0
dependencies:
  - name: postgresql
    version: "12.4.2"
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled
```
```bash
helm dependency build   # دانلود subcharts به charts/
```

**۳. Helmfile:**

```yaml
environments:
  prod:
    values:
      - env/prod.yaml
releases:
  - name: myapp
    namespace: production
    chart: ./charts/myapp
    values:
      - values/common.yaml
      - values/prod.yaml
  - name: monitoring
    chart: prometheus-community/kube-prometheus-stack
```

اجرا:
```bash
helmfile sync   # نصب/ارتقای همه
helmfile diff    # پیش‌نمایش تغییرات
```

### مقایسه و دیدگاه‌های مختلف

| قابلیت | کاربرد | جایگزین |
|---|---|---|
| Hooks | migration، setup | initContainer |
| Subcharts | چارت ترکیبی | Kustomize base |
| Helmfile | چند release | ArgoCD App-of-Apps |
| helm-diff | پیش‌نمایش | `kubectl diff` |

**دیدگاه:** برای تیم‌های بزرگ، Helmfile یا GitOps (ArgoCD) مدیریت release را متمرکز می‌کند. Hooks برای migration ضروری است اما باید idempotent باشند (چند بار اجرا بی‌خطا). Subcharts برای ترکیب نرم‌افزار آماده (DB) عالی‌اند.

**تجارت:** چارت پیچیده = نگهداری سخت. ساده نگه دارید.

### اشتباهات رایج

- **Hook غیر idempotent:** اجرای دوباره → خطا (مثلاً migration دوباره).
- **فراموش `hook-delete-policy`:** Jobهای hook در کلاستر باقی می‌مانند.
- **Dependency بدون `helm dependency build`:** chart فرعی پیدا نمی‌شود.
- **Helmfile بدون `helmfile diff` قبل:** تغییر ناخواسته در تولید.
- **ورژن subchart قفل نشده:** آپدیت ناگهانی chart فرعی → شکست.

### نکات مهم و ترفندها

- **`helm-diff` plugin:** همیشه قبل از upgrade بزنید.
- **`helm secrets`:** values رمزنگاری‌شده با SOPS.
- **Library chart:** منطق تکراری را در `_helpers.tpl` یک چارت مشترک بگذارید.
- **`helm test`:** تست release پس از نصب (smoke test).
- **Version pinning:** در dependencies همیشه نسخه دقیق بدهید.
- **`--atomic` / `--wait`:** اطمینان از موفقیت کامل یا rollback خودکار.

### خلاصه فصل

Helm پیشرفته شامل Hooks (برای migration/setup در lifecycle)، Dependencies (ترکیب subcharts مثل DB) و Helmfile (مدیریت چند release) است. Hooks باید idempotent باشند و با `hook-delete-policy` پاک شوند. Subcharts نسخه‌بندی‌شده برای ترکیب نرم‌افزار آماده عالی‌اند. Helmfile/ArgoCD برای مدیریت متمرکز release در محیط‌های پیچیده. کلید: `helm-diff`/`helm-test` قبل از تغییر، `helm upgrade --atomic` برای rollback خودکار، و نسخه‌بندی دقیق dependency.

---

## فصل ۶۵: مقایسه و انتخاب (Kustomize در برابر Helm)

### مدمه

«کدام را انتخاب کنیم؟» سؤالی است که هر تیم DevOps با آن روبه‌رو می‌شود. پاسخ قطعی نیست — بستگی به بافتار (context) دارد. در این فصل راهنمای تصمیم‌گیری و الگوهای ترکیبی را ارائه می‌دهیم.

### مفاهیم اساسی

**ماتریس تصمیم:**

| سناریو | توصیه |
|---|---|
| اپلیکیشن داخلی، چند محیط | Kustomize |
| توزیع نرم‌افزار به دیگران | Helm |
| منطق شرطی پیچیده | Helm |
| تیم می‌خواهد فقط YAML | Kustomize |
| استفاده از چارت‌های آماده | Helm |
| GitOps ساده | Kustomize یا Helm (هر دو OK) |

**ترکیب (Hybrid):**
بسیاری Helm chart را نصب می‌کنند و سپس با Kustomize **post-render** پچ می‌زنند (مخصوصاً در ArgoCD با `kustomize` build). یا برعکس: Kustomize base را با مقادیر از Helm منابع می‌کنند.

**عوامل فرهنگی:**
- یادگیری تیم: Kustomize (YAML) آسان‌تر از Helm (Go template).
- اکوسیستم: Helm Artifact Hub غنی‌تر.
- کنترل: Kustomize خروجی را مستقیم نشان می‌دهد (declarative خالص).

**GitOps و ابزارها:**
- **ArgoCD:** هر دو را پشتیبانی می‌کند (Kustomize native، Helm با موتور خودش).
- **Flux:** هر دو را پشتیبانی می‌کند.

### مثال‌های عملی

**۱. انتخاب بر اساس تیم:**

```
تیم کوچک، اپ داخلی، ۳ محیط → Kustomize (ساده)
تیم بزرگ، محصول SaaS، توزیع به مشتری → Helm (chart + values)
```

**۲. ترکیب Helm + Kustomize (ArgoCD):**

```yaml
# argocd Application
spec:
  source:
    path: charts/myapp
    # ArgoCD می‌تواند Helm را render و سپس Kustomize apply کند
```

**۳. مهاجرت تدریجی:**

```bash
# Helm chart موجود را به Kustomize تبدیل کنید:
helm template myapp ./chart > rendered.yaml
# سپس در base/kustomization استفاده کنید (البته dynamic نیست)
```

**۴. چک‌لیست انتخاب:**

```
[ ] آیا نرم‌افزار را توزیع می‌کنید؟ → Helm
[ ] آیا فقط محیط‌های داخلی دارید؟ → Kustomize
[ ] آیا منطق if/else پیچیده دارید؟ → Helm
[ ] آیا تیم Go template بلد نیست؟ → Kustomize
```

### مقایسه و دیدگاه‌های مختلف

| دیدگاه | طرفداران |
|---|---|
| "Kustomize کافی است" | تیم‌های پلتفرم، طرفدار declarative خالص |
| "Helm ضروری است" | تیم‌های محصول، استفاده‌کنندگان چارت آماده |
| "هر دو با هم" | تیم‌های بالغ با GitOps |

**دیدگاه نهایی:** انتخاب «درست» وجود ندارد، انتخاب «مناسب تیم شما» وجود دارد. شروع با Kustomize (یادگیری کمتر) و اضافه کردن Helm هنگام نیاز به بسته‌بندی، رویکردی محافظه‌کارانه و بی‌خطر است.

**Anti-pattern:** هر دو را برای همان مشکل موازی استفاده نکنید (confusion).

### اشتباهات رایج

- **انتخاب Helm فقط چون "محبوب‌تر":** شاید تیم به Kustomize نیاز داشته باشد.
- **ترکیب بی‌برنامه:** Helm و Kustomize را روی هم بدون معماری مشخص → هرج‌ومرج.
- **فراموش GitOps:** هر دو ابزار در Git نگه داشته شوند (source of truth).
- **چارت پیچیده بیش از نیاز:** سادگی را فدای انعطاف نکنید.
- **عدم مستندسازی values:** تیم نمی‌داند کدام پارامتر چه می‌کند.

### نکات مهم و ترفندها

- **شروع ساده:** Kustomize → اگر نیاز شد، Helm اضافه کنید.
- **`helm-docs` + README:** مستندسازی values.
- **ArgoCD Kustomize+Helm:** ابزار هر دو را یکجا مدیریت می‌کند.
- **Standardize:** در سازمان یکی را به عنوان استاندارد انتخاب کنید (یا الگوی ترکیبی مشخص).
- **Templates validation:** `kubeconform`/`kubeval` خروجی را validate کنید.

### خلاصه فصل

انتخاب بین Helm و Kustomize به بافتار بستگی دارد: Kustomize برای اپلیکیشن داخلی و کنترل YAML محض، Helm برای بسته‌بندی/توزیع و منطق پیچیده. رویکرد ترکیبی (Helm + Kustomize post-render) در تیم‌های بالغ رایج است. کلید: تصمیم را بر اساس نیاز تیم بگیرید نه محبوبیت، همه چیز را در Git (GitOps) نگه دارید، و از ابزار validate (kubeconform) استفاده کنید. شروع با Kustomize و اضافه کردن Helm هنگام نیاز، رویکرد محافظه‌کارانه است.

---

## منابع و مراجع

- Kustomize Documentation: https://kustomize.io
- Kustomize Built-ins: https://kubectl.docs.kubernetes.io/references/kustomize/
- Helm Documentation: https://helm.sh/docs/
- Helm Chart Best Practices: https://helm.sh/docs/chart_best_practices/
- Artifact Hub: https://artifacthub.io
- Helmfile: https://github.com/helmfile/helmfile
- Go Template: https://pkg.go.dev/text/template
- kubeconform: https://github.com/yannh/kubeconform
- ArgoCD (Helm/Kustomize): https://argo-cd.readthedocs.io

**بعدی:** [فایل ۱۴ — GitOps: ArgoCD و Flux](14-GitOps-ArgoCD-and-Flux.md)
