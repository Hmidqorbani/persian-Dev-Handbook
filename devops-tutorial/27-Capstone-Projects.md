# فایل ۲۷ — پروژه‌های Capstone (ترکیب همه)

**مدت یادگیری**: ۱۴ ساعت (پروژه‌محور)
**سطح سختی**: سخت (ترکیبی)
**پیش‌نیاز**: تمام فایل‌های ۰۱ تا ۲۶

---

## فهرست محتوا

- [فصل ۱۳۰: معماری مرجع End-to-End (Reference Architecture)](#فصل-۱۳۰-معماری-مرجع-end-to-end-reference-architecture)
- [فصل ۱۳۱: پروژه ۱ — خط لوله CI/CD و کانتینر](#فصل-۱۳۱-پروژه-۱--خط-لوله-cicd-و-کانتینر)
- [فصل ۱۳۲: پروژه ۲ — GitOps و استقرار در Kubernetes](#فصل-۱۳۲-پروژه-۲--gitops-و-استقرار-در-kubernetes)
- [فصل ۱۳۳: پروژه ۳ — Observability و SRE](#فصل-۱۳۳-پروژه-۳--observability-و-sre)
- [فصل ۱۳۴: پروژه ۴ — DevSecOps، Multi-cloud و ساخت Portfolio](#فصل-۱۳۴-پروژه-۴--devsecops-multi-cloud-و-ساخت-portfolio)

---

## فصل ۱۳۰: معماری مرجع End-to-End (Reference Architecture)

### مقدمه

این فصل همهٔ مفاهیم کتاب (۲۹ فایل) را در یک «معماری مرجع» واحد ترکیب می‌کند — از لحظه‌ای که توسعه‌دهنده کد می‌نویسد تا لحظه‌ای که در تولید با نظارت کامل اجرا می‌شود. این نقشهٔ راهی است که در پروژه‌های ۱ تا ۴ خواهید ساخت.

### مفاهیم اساسی

**جریان کلی (End-to-End Flow):**
```
۱. Source:   Git (GitHub/GitLab) + Branching (GitFlow/Trunk)
۲. CI:       Pipeline (GitHub Actions/GitLab CI/Jenkins)
             → build → test → SAST/SCA/Secret scan → image build
۳. Registry:  Push image به Harbor/ECR (اسکن + امضا با cosign)
۴. IaC:      Terraform برای زیربنا (VPC/Cluster)
۵. Config:   Helm/Kustomize برای تنظیمات
۶. GitOps:   ArgoCD/Flux مشاهدهٔ repo و همگام‌سازی با خوشه
۷. Runtime:  Kubernetes (Deployments/Services/Ingress)
۸. Mesh:     Istio/Linkerd (mTLS + ترافیک)
۹. Observ:   Prometheus + Loki + Tempo + OTel → Grafana
۱۰. SRE:     SLO/Error Budget + Alertmanager + on-call + Postmortem
۱۱. Sec:     Vault (رمز) + Kyverno (سیاست) + Zero Trust
۱۲. Cloud:   AWS/Azure/GCP (Terraform multi-cloud)
```

**اصول راهنما:**
- **همه چیز کد (Everything as Code):** کد، زیربنا، پیکربندی، سیاست، داشبورد.
- **Pull-based GitOps:** خوشه خودش از git می‌کشد (فصل ۱۴) — امن‌تر از push.
- **Shift-Left Security:** اسکن در CI (فصل ۱۱۷).
- **Observability-first:** بدون داده هیچ تصمیمی (فصل ۲۰–۲۲).
- **Progressive Delivery:** انتشار تدریجی با Canary (فصل ۴/۱۴).

**تقسیم لایه‌ها:**
- **Dev (توسعه):** Git + local (Docker/K3s).
- **CI/CD:** Pipeline + Registry.
- **Platform (پلتفرم):** K8s + GitOps + Mesh + Observability (تیم پلتفرم).
- **Product (محصول):** اپلیکیشن‌های تجاری.

### مثال‌های عملی

**الف) مثال مخزن (repo structure):**
```
my-platform/
├── app/                 # کد برنامه + Dockerfile
├── charts/orders/       # Helm chart
├── k8s/base/            # Kustomize base
├── terraform/           # IaC (VPC, EKS)
├── .github/workflows/   # CI pipeline
├── argocd/              # GitOps manifests
└── policies/            # Kyverno/OPA
```

**ب) نمودار جریان (متنی):**
```
Developer ─commit─▶ Git ─webhook─▶ CI ─build/scan─▶ Image
                                                     │
                                              ┌──────▼──────┐
                                              │   Registry  │ (cosign signed)
                                              └──────┬──────┘
                                                     │ git update (Helm values)
                                                     ▼
                                              ArgoCD ─pull─▶ Kubernetes
                                                     │
              ┌──────────────────────────────────────┼──────────────────────┐
              ▼                                      ▼                      ▼
        Prometheus/Loki/Tempo                Istio mesh              Vault (secrets)
              │                                      │                      │
              └──────────────▶ Grafana ◀──────────────┘◀──── OTel ──────────┘
```

**ج) فایل `.github/workflows/ci.yml` (خلاصه):**
```yaml
on: [push]
jobs:
  build-test-scan:
    steps:
      - checkout
      - run: docker build -t app:${{ github.sha }} .
      - trivy image app:${{ github.sha }}
      - semgrep scan
      - gitleaks
      - docker push registry/app:${{ github.sha }}
```

### مقایسه و دیدگاه‌های مختلف

**معماری کامل در برابر حداقل کافی (MVP):**
- کامل: یادگیری/سازمان بزرگ — همه لایه‌ها.
- MVP: استارتاپ — Git + CI + K8s + Prometheus ساده.

**تیم واحد در برابر Platform Team:**
- تیم واحد: همه چیز را یک تیم (ساده‌تر در کوچک).
- Platform Engineering: تیم پلتفرم ابزار می‌دهد، محصول جدا (مقیاس‌پذیرتر).

**دیدگاه:** «معماری مرجع یک هدف است نه نقطهٔ شروع — با لایه‌های ضروری شروع کنید و بقیه را تکاملی بیفزایید».

### اشتباهات رایج

1. **ساخت همه چیز دفعهٔ اول:** پیچیدگی انفجاری → شکست. تکاملی بسازید.
2. **Push-based deployment بدون GitOps:** خطای انسانی در kubectl در تولید.
3. **فراموشی Observability تا بعد از حادثه:** «چرا کند است؟» بدون داده.
4. **عدم جداسازی Secret از کد:** در معماری مرجع Vault اجباری است.
5. **بیش‌مهندسی (over-engineering):** ابزارهایی که تیم نمی‌تواند نگه دارد.

### نکات مهم و ترفندها

- **شروع با MVP:** Git + CI + K8s + Prometheus؛ سپس GitOps، سپس Mesh/Sec.
- **هر لایه را با یک ابزار شروع کنید** (نه سه جایگزین).
- **مستندسازی معماری** در README (فایل ۲۸ چیت‌شیت دارد).
- **تمرین در محیط آزمایشی:** Kind/Minikube/K3s برای تمرین رایگان.
- **GitOps repo جداگانه** برای تنظیمات تولید (separate from app code).

### خلاصه فصل

معماری مرجع جریانی پیوسته از **Source → CI → Registry → IaC → GitOps → K8s → Mesh → Observability → SRE → Security** می‌سازد که همهٔ مفاهیم کتاب را یکپارچه می‌کند. اصول: Everything-as-Code، Pull-based GitOps، Shift-Left Security، و Observability-first. اشتباه کلیدی: ساخت همه‌چیز دفعهٔ اول (بیش‌مهندسی). رویکرد: MVP تکاملی. در فصل‌های بعد چهار پروژهٔ عملی روی این معماری می‌سازیم.

---

## فصل ۱۳۱: پروژه ۱ — خط لوله CI/CD و کانتینر

### مقدمه

**پروژه ۱:** یک خط لولهٔ واقعی بسازید که کد را build، تست، اسکن و به تصویر کانتینر تبدیل کند و در registry با امضا منتشر کند. خروجی: یک image آمادهٔ استقرار. (پیوند به فصل ۰۴/۰۸/۱۱۷.)

### مفاهیم اساسی

**هدف پروژه:**
- مخزن Git با برنامهٔ ساده (مثلاً Flask/FastAPI).
- Pipeline که روی هر PR: lint → test → build image → scan → push → sign.
- Dockerfile چندمرحله‌ای و کوچک (فصل ۳۸).
- ثبت در Registry (ECR/Harbor) با امضای cosign (فصل ۴۵/۱۱۸).

**ابزارها:** GitHub Actions (یا GitLab CI)، Docker، Trivy، cosign، gitleaks.

**معیار موفقیت:** `git push` → چند دقیقه بعد image امضا‌شده در registry آماده است؛ هشدار اسکن در صورت CVE حیاتی.

### مثال‌های عملی

**الف) Dockerfile (multi-stage، distroless):**
```dockerfile
FROM python:3.12-slim AS build
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt
COPY . .
RUN python -m compileall .

FROM gcr.io/distroless/python3-debian12
COPY --from=build /root/.local /root/.local
COPY --from=build /app /app
WORKDIR /app
ENV PATH=/root/.local/bin:$PATH
USER nonroot
CMD ["app.py"]
```

**ب) GitHub Actions (`ci.yml`):**
```yaml
name: ci
on: [push, pull_request]
jobs:
  pipeline:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Lint & Test
        run: |
          pip install -r requirements.txt
          flake8 . && pytest
      - name: Build
        run: docker build -t app:${{ github.sha }} .
      - name: Scan
        uses: aquasecurity/trivy-action@master
        with: { scan-type: image, severity: CRITICAL, exit-code: 1 }
      - name: Push & Sign
        run: |
          docker push $REG/app:${{ github.sha }}
          cosign sign --yes $REG/app:${{ github.sha }}
        env:
          REG: ${{ secrets.REGISTRY }}
```

**ج) آزمایش محلی:**
```bash
docker build -t app:test .
docker run -p 8080:8080 app:test
curl localhost:8080/health
```

### مقایسه و دیدگاه‌های مختلف

**GitHub Actions در برابر GitLab CI:**
- GitHub: اکوسیستم بزرگ، ماتریکس آسان.
- GitLab: یکپارچگی با repo، DAG (needs) قوی.

**Push روی هر commit در برابر فقط PR:**
- هر commit: بازخورد سریع اما هزینه CI بالا.
- فقط PR: کنترل بهتر، مناسب تیم.

**دیدگاه:** «Pipeline باید در کمتر از ۱۰ دقیقه تمام شود — اگر کند است، کش (cache) و موازی‌سازی اضافه کنید».

### اشتباهات رایج

1. **تصویر حجیم (full OS):** بوت کند + سطح حمله بزرگ.
2. **عدم کش (cache) در CI:** هر بار نصب دوبارهٔ وابستگی → کند.
3. **push بدون اسکن:** CVE وارد registry می‌شود.
4. **رمز در secret ولی لاگ کردن آن:** در CI لاگ‌ها عمومی‌اند.
5. **تست نکردن محلی قبل از CI:** چرخهٔ اشکال‌زدایی کند.

### نکات مهم و ترفندها

- **Build cache:** در GitHub Actions از `actions/cache` برای pip/دایرکتوری.
- **Multi-stage حتماً:** حجم را ۵–۱۰ برابر کاهش دهید.
- **Cosign keyless (OIDC):** نیاز به مدیریت کلید کمتر.
- **SBOM** را با Syft تولید و پیوست کنید (فصل ۱۱۸).
- **جدا کردن stageها:** lint/test سریع قبل از build سنگین (شکست زودهنگام).
- **تمرین:** این پروژه را در GitHub رایگان انجام دهید.

### خلاصه فصل

پروژه ۱ یک **خط لولهٔ کامل CI/CD** ساخت: lint/test → build (multi-stage) → scan (Trivy) → push → sign (cosign). خروجی image امضا‌شده و اسکن‌شده در registry است. کلیدها: Dockerfile کوچک، کش در CI، و شکست زودهنگام. اشتباهات: تصویر حجیم و push بدون اسکن. در فصل بعد این image را با GitOps در K8s مستقر می‌کنیم.

---

## فصل ۱۳۲: پروژه ۲ — GitOps و استقرار در Kubernetes

### مدمه

**پروژه ۲:** تصویر پروژهٔ ۱ را با **ArgoCD** (یا Flux) در یک خوشهٔ K8s مستقر کنید و انتشار تدریجی (Canary) با **Argo Rollouts** پیاده کنید. (پیوند به فصل ۱۴/۱۳/۱۱.)

### مفاهیم اساسی

**هدف:**
- مخزن جداگانهٔ GitOps (manifests/Helm values).
- ArgoCD Application که خوشه را با git همگام می‌کند (pull-based).
- Helm chart یا Kustomize برای تنظیمات.
- Argo Rollouts برای Canary (ترافیک ۱۰٪ → ۱۰۰٪ با تحلیل).

**جریان:**
```
git commit (values.yaml با image جدید) → ArgoCD می‌بیند →
همگام‌سازی با خوشه → رول‌اوت کاناری → صحت‌سنجی → تکمیل یا برگشت
```

**ابزارها:** kind/minikube (محلی)، ArgoCD، Argo Rollouts، Helm.

### مثال‌های عملی

**الف) نصب ArgoCD (محلی با kind):**
```bash
kind create cluster --name devops
kubectl create ns argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl -n argocd get pods
```

**ب) ArgoCD Application (`app.yaml`):**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata: { name: orders }
spec:
  destination: { namespace: app, server: https://kubernetes.default.svc }
  source:
    repoURL: https://github.com/you/gitops-repo
    path: charts/orders
  syncPolicy:
    automated: { prune: true, selfHeal: true }
```

**ج) Argo Rollouts (Canary — خلاصه):**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
spec:
  strategy:
    canary:
      steps:
        - setWeight: 10
        - pause: { duration: 2m }     # بررسی سلامت
        - setWeight: 50
        - pause: { duration: 2m }
        - setWeight: 100
```

**د) به‌روزرسانی انتشار:**
```bash
# فقط مقدار image را در git تغییر دهید؛ ArgoCD بقیه را انجام می‌دهد
sed -i 's|image:.*|image: app:v2|' gitops-repo/charts/orders/values.yaml
git commit -am "release v2" && git push
```

### مقایسه و دیدگاه‌های مختلف

**ArgoCD در برابر Flux:**
- ArgoCD: UI غنی، App-of-Apps، مناسب تیم.
- Flux: سبک‌تر، GitOps-native، بدون UI سنگین.

**Helm در برابر Kustomize (فصل ۶۱–۶۵):**
- Helm: templating، versioning، chart reuse.
- Kustomize: overlay ساده، بدون template.

**Canary در برابر Blue-Green:**
- Canary: ترافیک تدریجی، ریسک کمتر.
- Blue-Green: تعویض کامل، بازگشت سریع‌تر.

**دیدگاه:** «GitOps یعنی git منبع حقیقت (source of truth) است — kubectl در تولید ممنوع».

### اشتباهات رایج

1. **استفاده از kubectl apply در تولید:** تضاد با GitOps (drift).
2. **repo GitOps با دسترسی نوشتن باز:** هر کسی می‌تواند خوشه را تغییر دهد.
3. **عدم self-heal:** وقتی کسی دستی تغییر داد، ArgoCD اصلاح نمی‌کند.
4. **Canary بدون معیار سلامت:** وزن را زیاد می‌کند بدون چک کردن خطا.
5. **صلاح image با `latest`:** نسخه نامشخص → بازتولید غیرممکن.

### نکات مهم و ترفندها

- **App-of-Apps** برای مدیریت چند اپلیکیشن در ArgoCD.
- **selfHeal + prune** را فعال کنید (اصلاح drift خودکار).
- **Image tag دقیق** (نه latest) در git.
- **ArgoCD Notifications** برای هشدار موفقیت/شکست همگام‌سازی.
- **Rollback:** با `argocd app rollback` یا تغییر git به نسخهٔ قبل.
- **تمرین در kind:** رایگان و سریع برای یادگیری.

### خلاصه فصل

پروژه ۲ استقرار را از push به **Pull-based GitOps** تبدیل کرد: ArgoCD خوشه را با git همگام می‌کند و **Argo Rollouts** انتشار کاناری (۱۰٪→۱۰۰٪) با توقف برای بررسی سلامت می‌دهد. کلیدها: git به‌عنوان source of truth، selfHeal فعال، و image tag دقیق. اشتباهات: kubectl دستی در تولید و Canary بدون معیار. در فصل بعد نظارت و SRE را اضافه می‌کنیم.

---

## فصل ۱۳۳: پروژه ۳ — Observability و SRE

### مدمه

**پروژه ۳:** پشتهٔ مشاهده‌پذیری (Prometheus + Loki + Tempo + OTel) را روی خوشهٔ پروژهٔ ۲ نصب کنید، یک SLO تعریف کنید، و یک تمرین حادثه (Game Day) اجرا کنید. (پیوند به فصل ۲۰–۲۳/۱۰۷.)

### مفاهیم اساسی

**هدف:**
- نصب kube-prometheus-stack + Loki + Tempo + OTel Collector (Helm).
- instrument کردن اپلیکیشن با OTel SDK (فصل ۱۰۸).
- تعریف SLO (مثلاً ۹۹.۹٪ availability) + هشدار burn rate.
- اجرای Game Day: حذف پاد یا تزریق خطا → تمرین پاسخ.
- نوشتن یک Blameless Postmortem (فصل ۱۱۳).

**ابزارها:** Helm، Prometheus، Loki، Tempo، Grafana، OTel Collector، Chaos Mesh (اختیاری).

### مثال‌های عملی

**الف) نصب با Helm:**
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install mon prometheus-community/kube-prometheus-stack -n monitoring --create-namespace

helm repo add grafana https://grafana.github.io/helm-charts
helm install loki grafana/loki-stack -n monitoring
helm install tempo grafana/tempo -n monitoring
```

**ب) اپلیکیشن instrument شده (OTel):**
```python
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
# در app.py:
FlaskInstrumentor().instrument_app(app)
# exporter OTLP به Collector (فصل ۱۰۷)
```

**ج) SLO + هشدار (Prometheus Rule — خلاصه):**
```yaml
groups:
  - name: orders-slo
    rules:
      - alert: OrdersErrorBudgetBurn
        expr: |
          (1 - sum(rate(http_requests_total{service="orders",code=~"2.."}[1h]))
               / sum(rate(http_requests_total{service="orders"}[1h]))) > (14.4 * (1-0.999))
        for: 2m
        labels: { severity: page }
```

**د) Game Day (سناریو):**
```
۱. kubectl delete pod -l app=orders -n app   (خرابی مصنوعی)
۲. مشاهدهٔ هشدار در Alertmanager → Slack
۳. On-call بررسی داشبورد Grafana → می‌بیند replica جایگزین شده
۴. نوشتن Postmortem بی‌سرزنش
```

### مقایسه و دیدگاه‌های مختلف

**نصب Helm در برابر Manifest دستی:**
- Helm: سریع، نسخه‌بندی، rollback.
- دستی: کنترل کامل، اما خسته‌کننده.

**یک ابزار در برابر همه (OTel unified):**
- همه جدا: سخت‌تر اما انعطاف.
- OTel unified (فصل ۱۱۰): یک Collector برای هر سه → ساده‌تر.

**دیدگاه:** «بدون Game Day، SLO فقط عددی در برگه است — وقتی واقعاً خراب شود، می‌فهمید آماده‌اید یا نه».

### اشتباهات رایج

1. **نصب همه بدون persistent volume:** ری‌استارت دادهٔ متریک را پاک می‌کند.
2. **عدم instrument کردن app:** فقط متریک زیربنا (node) دارید، نه اپلیکیشن.
3. **SLO بدون هشدار:** عدد را تعریف می‌کنید اما کسی خبردار نمی‌شود.
4. **Game Day در تولید بدون هماهنگی:** حادثهٔ واقعی درست می‌کنید.
5. **Postmortem بدون action item:** یادگیری مستند نمی‌شود.

### نکات مهم و ترفندها

- **PersistentVolume** برای Prometheus/Tempo/Loki (در K8s PVC).
- **Grafana dashboards** آماده (node-exporter، k8s) را از grafana.com دانلود کنید.
- **Alertmanager به Slack/PagerDuty** وصل کنید.
- **تمرین Game Day فصلی** در تقویم.
- **Exemplars** را فعال کنید (متریک↔تریس، فصل ۱۱۰).
- **Sloth** برای تولید خودکار قواعد SLO.

### خلاصه فصل

پروژه ۳ پشتهٔ **Observability یکپارچه** (Prometheus/Loki/Tempo/OTel → Grafana) و **SRE** (SLO + هشدار burn rate + Game Day + Postmortem) را روی خوشه می‌آورد. کلیدها: persistent volume، instrument کردن app، و تمرین حادثه واقعی. اشتباهات: نصب بدون PV و SLO بدون هشدار. با این پروژه سیستم قابل اتکا ساخته شد. در فصل بعد لایهٔ امنیت و چندابری را می‌افزاییم.

---

## فصل ۱۳۴: پروژه ۴ — DevSecOps، Multi-cloud و ساخت Portfolio

### مدمه

**پروژه ۴ (نهایی):** لایهٔ امنیت (Vault، Kyverno، اسکن در CI) و چندابری (Terraform) را به معماری اضافه کنید، و از همه چیز یک **Portfolio** (گواهی کار) در GitHub بسازید. (پیوند به فصل ۲۴/۱۵/۱۹.)

### مفاهیم اساسی

**هدف:**
- **DevSecOps:** افزودن vault (رمز)، Kyverno (سیاست: فقط image امضا‌شده)، و اسکن در CI (قبلاً در پروژهٔ ۱ بود، حالا اجباری در gate).
- **Multi-cloud:** Terraform برای ایجاد VPC/Cluster در دو ابر (مثلاً AWS + GCP) — یک کد.
- **Platform Engineering:** ابزارها را به‌صورت self-service درآورید (مثلاً Template برای تیم محصول).
- **Portfolio:** مخازن GitHub با README خوب، داشبوردهای Grafana، و مستندات — برای نمایش مهارت.

**ارزیابی نهایی:** معماری کامل که کد → امن → مستقر → نظارت‌شده → قابل اتکا است، در یک یا دو ابر.

### مثال‌های عملی

**الف) Kyverno Policy (فقط image امضا‌شده):**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata: { name: require-signed-images }
spec:
  validationFailureAction: enforce
  rules:
    - name: check-signature
      match: { resources: { kinds: [Pod] } }
      verifyImages:
        - imageReferences: ["*"]
          attestors:
            - entries:
                - keys: { publicKeys: | {{ vault_public_key }} }}
```

**ب) Terraform چندابری (خلاصه):**
```hcl
provider "aws"   { region = "us-east-1" }
provider "google" { project = "my-proj" region = "us-central1" }
module "vpc_aws"   { source = "./modules/vpc"   cloud = "aws" }
module "vpc_gcp"   { source = "./modules/vpc"   cloud = "gcp" }
```

**ج) External Secrets (Vault → K8s):**
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata: { name: db-creds }
spec:
  data:
    - secretKey: password
      remoteRef: { key: secret/db, property: password }
```

**د) ساختار Portfolio در GitHub:**
```
github.com/you/
├── devops-platform/        # معماری کامل (این پروژه)
│   ├── README.md           # نمودار + توضیح
│   ├── ci/                 # workflow ها
│   ├── terraform/          # IaC
│   ├── gitops/             # ArgoCD manifests
│   └── dashboards/         # JSON داشبورد Grafana
└── learnings/              # پست‌های وبلاگ/یادداشت
```

### مقایسه و دیدگاه‌های مختلف

**امنیت به‌عنوان افزونه در برابر built-in:**
- افزونه: بعداً اضافه می‌شود → اغلب فراموش می‌شود.
- built-in: از ابتدا در pipeline/gate (توصیه).

**یک ابر در برابر Multi-cloud در Portfolio:**
- یک ابر: عمق بیشتر، یادگیری متمرکز.
- Multi-cloud: نشان‌دهندهٔ انعطاف (اما پیچیدگی بیشتر).

**دیدگاه:** «Portfolio شما باید یک مسئلهٔ واقعی را حل کند — نه فقط 'HELLO WORLD' روی K8s».

### اشتباهات رایج

1. **امنیت را به انتها موکول کردن:** بازنویسی گران.
2. **Multi-cloud بدون دلیل:** پیچیدگی بی‌دلیل در Portfolio.
3. **README خالی در GitHub:** کسی نمی‌فهمد چه ساخته‌اید.
4. **رمز در repo Portfolio:** حتی نمونه‌ها را با fake جایگزین کنید.
5. **عدم نمایش داشبورد/نتیجه:** فقط کد، بدون اثبات کارکرد.

### نکات مهم و ترفندها

- **شروع Portfolio با یک پروژهٔ کوچک کامل** بهتر از ده پروژهٔ نیمه‌کاره.
- **README هر پروژه:** نمودار معماری + نحوهٔ اجرا + نتایج (اسکرین‌شات داشبورد).
- **Makefile/脚本** برای یک‌فرمانی کردن راه‌اندازی محلی.
- **نوشتن دربارهٔ چالش‌ها:** مستندسازی یادگیری ارزشمندتر از کد است.
- **مشارکت در پروژهٔ متن‌باز:** برای اعتبار حرفه‌ای.
- **شبیه‌سازی مصاحبه:** پروژه را طوری توضیح دهید که در مصاحبه بتوانید دفاع کنید.

### خلاصه فصل

پروژه ۴ لایه‌های نهایی — **DevSecOps** (Vault/Kyverno/اسکن اجباری)، **Multi-cloud** (Terraform یکسان) و **Platform Engineering** — را به معماری می‌افزاید و همه را در یک **Portfolio** در GitHub بسته‌بندی می‌کند. کلید موفقیت: امنیت built-in (نه الحاقی)، README قوی با نمودار/نتایج، و تمرکز روی حل مسئلهٔ واقعی. اشتباهات: موکول کردن امنیت و README خالی. با این پروژه، سفر عملی DevOps کامل شد. فایل بعدی (۲۸) بهترین روش‌ها، چیت‌شیت‌ها و واژه‌نامه را جمع‌بندی می‌کند.

---

## منابع و مراجع

- ArgoCD — https://argo-cd.readthedocs.io/  ·  Argo Rollouts — https://argoproj.github.io/rollouts/
- kube-prometheus-stack — https://github.com/prometheus-community/helm-charts
- HashiCorp Vault — https://developer.hashicorp.com/vault  ·  External Secrets — https://external-secrets.io/
- Kyverno — https://kyverno.io/  ·  OPA — https://www.openpolicyagent.org/
- Terraform — https://developer.hashicorp.com/terraform (فایل ۱۵)
- Kind (K8s محلی) — https://kind.sigs.k8s.io/
- CNCF Landscape — https://landscape.cncf.io/ (نقشهٔ کامل ابزارها)
- «Accelerate» (کتاب Forsgren et al. — DORA metrics)
- Building a Portfolio: مستندات GitHub Pages / README best practices

**بعدی:** [فایل ۲۸ — بهترین روش‌ها، چیت‌شیت‌ها و واژه‌نامه (جمع‌بندی)](28-Best-Practices-CheatSheets-Glossary.md)
