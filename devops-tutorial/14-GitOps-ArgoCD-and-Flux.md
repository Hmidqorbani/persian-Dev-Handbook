# فایل ۱۴ — GitOps: ArgoCD و Flux

**مدت یادگیری**: ۱۱ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: فایل ۱۱، ۱۲، ۱۳ (کوبرنتیز، Helm، Kustomize)، فایل ۰۴ (CI/CD)

---

## فهرست محتوا

- [فصل ۶۶: فلسفه GitOps (Git به عنوان منبع حقیقت)](#فصل-۶۶-فلسفه-gitops-git-به-عنوان-منبع-حقیقت)
- [فصل ۶۷: ArgoCD مبانی (معماری، Application و Sync)](#فصل-۶۷-argocd-مبانی-معماری-application-و-sync)
- [فصل ۶۸: ArgoCD پیشرفته (App-of-Apps، Rollback، سلامت)](#فصل-۶۸-argocd-پیشرفته-app-of-apps-rollback-سلامت)
- [فصل ۶۹: Flux (Controllers، Kustomize/Helm، Image Automation)](#فصل-۶۹-flux-controllers-kustomizehelm-image-automation)
- [فصل ۷۰: مقایسه ArgoCD و Flux و بهترین روش‌ها](#فصل-۷۰-مقایسه-argocd-و-flux-و-بهترین-روشها)

---

## فصل ۶۶: فلسفه GitOps (Git به عنوان منبع حقیقت)

### مدمه

در رویکرد سنتی CI/CD، خط لوله به طور **Push** تغییرات را به کلاستر می‌فرستد (`kubectl apply` در CI). در **GitOps**، جهت برعکس است: کلاستر به طور **Pull** از Git می‌خواند و خودش را با آن هم‌راستا (reconcile) می‌کند. این فصل انقلاب GitOps را توضیح می‌دهد.

### مفاهیم اساسی

**GitOps چیست؟**
رویکردی که در آن **وضعیت مطلوب (desired state) کلاستر در Git ذخیره می‌شود** و یک عامل (agent) در کلاستر به صورت مداوم تلاش می‌کند وضعیت واقعی را با Git هم‌راستا کند.

**اصول چهارگانه (Weaveworks):**
1. **Git as Source of Truth:** همه چیز در Git (declarative).
2. **Declarative:** منیفست‌های K8s (YAML/Helm/Kustomize).
3. **Versioned & Immutable:** تغییرات از طریق Git (audit trail کامل).
4. **Automated Reconciliation:** عامل کلاستر خودکار هم‌راستا می‌کند.

**Push vs Pull (معماری):**

```
سنتی (Push):  CI → kubectl apply → Cluster
GitOps (Pull): Git → (Agent در Cluster) → می‌خواند → apply می‌کند
```

**مزایا:**
- **Rollback فوری:** فقط کافیست به commit قبلی برگردید (Git revert).
- **Auditability:** هر تغییر در Git تاریخچه دارد.
- **امنیت:** CI نیازی به دسترس kubectl ندارد (agent در کلاستر دسترس دارد).
- **Consistency:** انحراف (drift) خودکار اصلاح می‌شود.

**Drift Detection:** اگر کسی دستی `kubectl edit` کند، عامل آن را با Git بازمی‌گرداند (دفاع در برابر تغییرات غیرمجاز).

### مثال‌های عملی

**۱. ساختار مخزن GitOps:**

```
gitops-repo/
├── apps/
│   ├── dev/
│   │   └── myapp/ (kustomize overlay یا helm values)
│   └── prod/
│       └── myapp/
├── clusters/
│   └── prod/ (تعریف cluster)
└── README.md
```

**۲. جریان کاری:**

```bash
# توسعه‌دهنده تغییر می‌دهد
git commit -am "bump image to v1.3"
git push
# عامل GitOps (ArgoCD/Flux) متوجه می‌شود
# کلاستر خودکار هم‌راستا می‌شود
```

**۳. Rollback (ساده):**

```bash
git revert HEAD
git push
# کلاستر به وضعیت قبلی برمی‌گردد
```

**۴. تفاوت با CI سنتی:**

```yaml
# سنتی: CI مستقیم apply می‌کند (نیاز به kubeconfig در CI)
deploy:
  script: kubectl apply -f k8s/

# GitOps: CI فقط به Git می‌زند
deploy:
  script: git commit -am "update" && git push
# عامل باقی‌مانده کار را می‌کند
```

### مقایسه و دیدگاه‌های مختلف

| معیار | Push (سنتی) | Pull (GitOps) |
|---|---|---|
| جهت | CI → Cluster | Cluster ← Git |
| دسترس CI | نیاز به kubeconfig | فقط Git write |
| Rollback | دستی/اسکریپت | git revert |
| Drift | نادیده گرفته | خودکار اصلاح |
| پیچیدگی | کمتر | نصب عامل |

**دیدگاه‌ها:**
- GitOps استاندارد طلایی برای K8s تولید است (بسیاری از شرکت‌ها پذیرفته‌اند).
- برای سیستم‌های غیرکوبرنتیز، GitOps کمتر کاربرد دارد (اما ابزارهایی مثل Argo CD برای هر منبعی هست).
- چالش: "Git becomes the bottleneck" — تغییرات کوچک هم باید از Git بگذرند (برای برخی آزاردهنده).

**Progressive Delivery:** GitOps + Argo Rollouts (فصل ۱۲/فصل پیش) برای Canary خودکار.

### اشتباهات رایج

- **Git را فقط برای کد، نه برای کانفیگ:** در GitOps، **همه** (شامل Secrets رمزنگاری‌شده) باید در Git باشد.
- **عامل با دسترس بیش از حد:** ArgoCD/Flux باید RBAC محدود داشته باشند.
- **فراموش Image Tag در Git:** اگر `latest` استفاده کنید، عامل نمی‌فهمد چه تغییری کرده (استفاده از SHA/tag صریح).
- **تغییر دستی در کلاستر:** با GitOps، `kubectl edit` بی‌فایده است (بازنویسی می‌شود) — از Git استفاده کنید.
- **عدم محافظت از شاخهٔ اصلی:** هر کسی بتواند push کند → کلاستر تحت تأثیر قرار می‌گیرد (PR + review اجباری).

### نکات مهم و ترفندها

- **Sealed Secrets / SOPS:** رمز را encrypted در Git نگه دارید (فصل ۵۹).
- **Branch strategy:** `main` = تولید؛ PR برای هر تغییر.
- **Image tags صریح:** همیشه نسخه/SHA در Git (هیچ `latest`).
- **CI مسئول build/push:** CI image را می‌سازد و به رجیستری می‌فرستد؛ GitOps آن را مستقر می‌کند (تفکیک واضح).
- **Notifications:** ArgoCD/Flux وضعیت sync را به Slack می‌فرستند.
- **Policy guardrails:** با OPA/Gatekeeper (فصل ۶۰) تغییرات را قبل از اعمال validate کنید.

### خلاصه فصل

GitOps مدل Pull است: Git منبع حقیقت است و عاملی در کلاستر وضعیت را با Git هم‌راستا می‌کند (reconciliation). مزایا: rollback فوری (git revert)، audit کامل، امنیت (CI بدون دسترس kubectl)، و drift detection خودکار. کلید: همه چیز (حتی Secrets رمزنگاری‌شده) در Git، Image tag صریح، محافظت از شاخهٔ اصلی با PR، و RBAC محدود برای عامل. GitOps استاندارد طلایی K8s تولید است.

---

## فصل ۶۷: ArgoCD مبانی (معماری، Application و Sync)

### مدمه

**ArgoCD** محبوب‌ترین ابزار GitOps برای Kubernetes است. UI زیبا، یکپارچگی عالی با Helm/Kustomize، و مفهوم **Application** برای هر منبع. در این فصل معماری و نحوهٔ تعریف اولین Application را یاد می‌گیرید.

### مفاهیم اساسی

**معماری ArgoCD:**

| جزء | نقش |
|---|---|
| **argocd-server** | UI/API |
| **argocd-repo-server** | کلون و رندر Git (Helm/Kustomize) |
| **argocd-application-controller** | عامل reconcile (مقایسه Git با کلاستر) |
| **argocd-dex-server** | احراز (SSO) اختیاری |

**مفهوم Application:**
یک منبع قابل استقرار (مسیر Git + مسیر/چارت + هدف namespace). هر Application یک کارت در UI دارد که وضعیت Sync/Health را نشان می‌دهد.

**حالت‌های Sync:**
- **Synced:** وضعیت کلاستر = Git.
- **OutOfSync:** تفاوت وجود دارد.
- **Unknown:** عامل نمی‌تواند بررسی کند.

**حالت‌های Health:**
- **Healthy / Progressing / Degraded / Missing / Suspended**

**Sync مodes:**
- **Manual:** فقط با کلیک/فرمان sync می‌شود.
- **Automatic:** به محض تغییر Git، خودکار sync (با گزینه‌های prune و self-heal).

**Source types:** Git (Kustomize/Helm/Plain YAML)، Helm Repository، OCI Helm.

### مثال‌های عملی

**۱. نصب ArgoCD:**

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
# دسترس به UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

**۲. ورود:**

```bash
argocd admin initial-password -n argocd   # رمز اولیه
argocd login localhost:8080
```

**۳. تعریف Application (CLI):**

```bash
argocd app create myapp \
  --repo https://github.com/org/gitops-repo.git \
  --path overlays/prod \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace production \
  --sync-policy automated
```

**۴. تعریف با Manifest (توصیه‌شده - GitOps!):**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/org/gitops-repo.git
    path: overlays/prod
    targetRevision: HEAD
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

**۵. بررسی وضعیت:**

```bash
argocd app get myapp
argocd app sync myapp    # اگر manual
```

### مقایسه و دیدگاه‌های مختلف

| ویژگی | ArgoCD | Flux |
|---|---|---|
| UI | عالی | نه (CLI/UI جداگانه) |
| مدل | Application-centric | Git-native/controller |
| یادگیری | متوسط | متوسط |
| App-of-Apps | بومی | Kustomize برای خودش |

**دیدگاه:** ArgoCD برای تیم‌هایی که UI و دید بصری می‌خواهند بهترین است. تعریف Application به صورت Manifest (نه فقط CLI) بسیار مهم است — خودتان هم GitOps باشید.

**App-of-Apps:** الگویی که یک Application والد، بقیه را مدیریت می‌کند (فصل ۶۸).

### اشتباهات رایج

- **فقط ایجاد با CLI (بدون Manifest):** اگر ArgoCD پاک شود، تعاریف از بین می‌روند؛ Application را در Git نگه دارید.
- **`prune: false` در auto:** منابع حذف‌شده در Git در کلاستر می‌مانند (زاید).
- **`selfHeal: false`:** drift اصلاح نمی‌شود.
- **مسیر اشتباه در `path`:** اشاره به پوشهٔ اشتباه → خطای رندر.
- **دسترس namespace اشتباه:** در فضای نام غلط مستقر می‌شود.

### نکات مهم و ترفندها

- **ApplicationSet:** برای تولید خودکار Application برای چند کلاستر/محیط (GitOps-at-scale).
- **Projects:** تفکیک دسترس و منابع در ArgoCD (مثل tenant isolation).
- **`argocd app diff`:** پیش‌نمایش تغییرات قبل از sync.
- **RBAC در ArgoCD:** کنترل چه کسی چه را sync کند.
- **Notifications:** پلاگین برای Slack/Teams.
- **Argo CD Image Updater:** به‌روزرسانی خودکار image tag در Git (فصل ۱۵).

### خلاصه فصل

ArgoCD عامل GitOps با UI قدرتمند و مفهوم Application (هر منبع استقرار) است. معماری شامل server/repo-server/controller است. Application با Manifest (توصیه) یا CLI تعریف می‌شود؛ sync می‌تواند manual یا automated (با prune/selfHeal) باشد. کلید: Application را در Git نگه دارید (نه فقط CLI)، `prune`+`selfHeal` را فعال کنید، و از ApplicationSet برای مقیاس. ArgoCD برای تیم‌هایی که دید بصری می‌خواهند ایده‌آل است.

---

## فصل ۶۸: ArgoCD پیشرفته (App-of-Apps، Rollback، سلامت)

### مدمه

برای مدیریت ده‌ها اپلیکیشن در چند محیط، تعریف تکی Application کافی نیست. **App-of-Apps**، **Rollbacks**، و **Health Customization** ابزارهای سطح تولید هستند. این فصل ArgoCD را برای مقیاس آماده می‌کند.

### مفاهیم اساسی

**App-of-Apps Pattern:**
یک Application که خودش مجموعه‌ای از Applicationهای دیگر را با Kustomize/Helm تعریف می‌کند. با یک sync، همه چیز مستقر می‌شود. برای بوتسترپ (bootstrap) کلاستر عالی است.

**Rollback در ArgoCD:**
هر sync یک **Revision** (تاریخچه Git) دارد. با `argocd app rollback <app> <revision>` به عقب برمی‌گردید — یا ساده‌تر، `git revert` در مخزن (GitOps واقعی).

**Health Assessment:**
ArgoCD به طور پیش‌فرض سلامت K8s اشیاء را می‌فهمد، اما برای منابع سفارشی (CRD)، **Health Check سفارشی** (Lua script) تعریف می‌کنید.

**Sync Waves & Phases:**
ترتیب استقرار منابع را کنترل می‌کند (مثلاً ابتدا Namespace، سپس DB، سپس اپلیکیشن). با annotation `argocd.argoproj.io/sync-wave`.

**Sync Hooks:**
دستوراتی که قبل/بعد/در زمان sync اجرا می‌شوند (مثل اجرای migration).

**Diffing Customization:**
برای منابعی که فیلدهای تغییرناپذیر دارند (مثل Secret که base64 می‌شود)، تنظیم کنید ArgoCD آن‌ها را نادیده بگیرد.

### مثال‌های عملی

**۱. App-of-Apps:**

```yaml
# apps.yaml (در Git)
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: apps-all
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/org/gitops-repo.git
    path: apps   # پوشه‌ای که شامل چند Application است
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated: { prune: true, selfHeal: true }
```

که در پوشه `apps/`، فایل‌هایی مثل `web.yaml`، `api.yaml` (هر کدام یک Application) هستند.

**۲. Sync Wave:**

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "1"   # بعد از wave 0
```

**۳. Rollback:**

```bash
argocd app history myapp
argocd app rollback myapp 3
# یا GitOps واقعی:
git revert <commit> && git push
```

**۴. Health Check سفارشی (مثال CRD):**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
# (در ConfigMap argocd-cm بخش resource.customisations)
# مثال Lua برای بررسی وضعیت یک CRD
```

### مقایسه و دیدگاه‌های مختلف

| ویژگی | کاربرد |
|---|---|
| App-of-Apps | بوتسترپ کلاستر |
| Sync Waves | ترتیب استقرار (مثل DB اول) |
| Rollback | بازگشت از خرابی |
| Health custom | CRDهای سفارشی |

**دیدگاه:** App-of-Apps برای "everything as code" ضروری است — کل کلاستر از یک نقطه بازسازی می‌شود. برای تیم‌های بزرگ، **ApplicationSet** (تولید خودکار بر اساس لیست کلاسترها) جایگزین دستی App-of-Apps است.

**ترجیح GitOps:** rollback با `git revert` بر `argocd app rollback` ترجیح داده می‌شود چون تاریخچه در Git باقی می‌ماند.

### اشتباهات رایج

- **App-of-Apps بدون prune:** اپلیکیشن‌های حذف‌شده در Git باقی می‌مانند.
- **Wave اشتباه:** اپلیکیشن قبل از DB آماده می‌شود → خطا.
- **فراموش Health برای CRD:** ArgoCD CRD را همیشه "Progressing" نشان می‌دهد.
- **Rollback با CLI بدون Git:** تاریخچه Git ناقص می‌شود.
- **Sync hook ناایمن:** اجرای دستور مخرب در hook.

### نکات مهم و ترفندها

- **ApplicationSet برای Scale:** تعریف یک الگو که برای هر کلاستر/محیط Application می‌سازد.
- **`argocd app wait`:** در CI/اسکریپت منتظر sync کامل بمانید.
- **Resource exclusions:** مواردی که نباید مدیریت شوند (مثل کلاستر scoped) را exclude کنید.
- **Notification با Slack:** وضعیت degraded را فوری ببینید.
- **Argo Rollouts integration:** برای Canary مستقیم در ArgoCD (فصل ۱۵/۲۰).

### خلاصه فصل

ArgoCD پیشرفته شامل App-of-Apps (بوتسترپ کل کلاستر از یک نقطه)، Sync Waves (ترتیب استقرار)، Rollback (ترجیحاً با git revert)، و Health Check سفارشی برای CRDها است. کلید: App-of-Apps با prune فعال، استفاده از Wave برای ترتیب صحیح (DB قبل از اپ)، و ترجیح Git-based rollback بر CLI. برای مقیاس واقعی، ApplicationSet جایگزین دستی App-of-Apps می‌شود. ArgoCD + Argo Rollouts = GitOps + Progressive Delivery کامل.

---

## فصل ۶۹: Flux (Controllers، Kustomize/Helm، Image Automation)

### مدمه

**Flux** ابزار GitOps دیگر (توسط CNCF، قبلاً Weaveworks) است که رویکرد **controller-based و Git-native** دارد — بدون UI مرکزی، همه چیز از طریق CRDها و Git. در این فصل Flux را از دید کنترلرها و قابلیت‌های منحصربه‌فردش (مثل Image Automation) بررسی می‌کنیم.

### مفاهیم اساسی

**معماری مبتنی بر Controller:**
Flux مجموعه‌ای از کنترلرهای کوچک است:
- **source-controller:** منابع (Git/Helm repo) را مدیریت می‌کند.
- **kustomize-controller:** منیفست‌های Kustomize را apply می‌کند.
- **helm-controller:** چارت‌های Helm را مدیریت می‌کند.
- **notification-controller:** رویدادها را به Slack و غیره می‌فرستد.
- **image-reflector-controller / image-automation-controller:** بررسی و به‌روزرسانی خودکار imageها.

**اشیاء اصلی Flux:**

| شیء | کاربرد |
|---|---|
| **GitRepository** | منبع Git |
| **Kustomization** | مسیر Kustomize برای apply |
| **HelmRepository** | رجیستری چارت |
| **HelmRelease** | نصب چارت |
| **ImagePolicy** | قانون به‌روزرسانی image |

**تفاوت با ArgoCD:**
- Flux **Git-native** است: همه چیز CRD در خود کلاستر (و Git) — UI ندارد (اگرچه Flux UI جدید هست).
- هر منبع یک شیء جدا (مثلاً GitRepository + Kustomization).
- **Image Automation:** ویژگی برجسته — Flux می‌تواند خودش image tag جدید را در Git کامیت کند (با PR یا مستقیم).

**Bootstrap:**
`flux bootstrap github` کل استقرار Flux و تنظیمات را در Git می‌نویسد (GitOps برای خودِ Flux).

### مثال‌های عملی

**۱. نصب/Bootstrap:**

```bash
flux bootstrap github \
  --owner=org \
  --repository=gitops-repo \
  --path=clusters/prod \
  --personal
```

**۲. GitRepository + Kustomization:**

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata: { name: apps, namespace: flux-system }
spec:
  interval: 1m
  ref: { branch: main }
  url: https://github.com/org/gitops-repo
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata: { name: apps, namespace: flux-system }
spec:
  interval: 5m
  path: "./apps/prod"
  sourceRef:
    kind: GitRepository
    name: apps
  prune: true
```

**۳. HelmRelease:**

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata: { name: bitnami, namespace: flux-system }
spec:
  interval: 30m
  url: https://charts.bitnami.com/bitnami
---
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata: { name: postgres, namespace: db }
spec:
  interval: 5m
  chart:
    spec:
      chart: postgresql
      sourceRef:
        kind: HelmRepository
        name: bitnami
      version: "12.x.x"
  values:
    replicaCount: 1
```

**۴. Image Automation (به‌روزرسانی خودکار):**

```yaml
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImagePolicy
metadata: { name: app-policy, namespace: flux-system }
spec:
  imageRepositoryRef: { name: myapp }
  policy:
    semver: "1.x"    # آخرین 1.x را انتخاب کن
---
# ImageUpdateAutomation کامیت تغییر را به Git می‌زند
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageUpdateAutomation
metadata: { name: app-auto, namespace: flux-system }
spec:
  interval: 5m
  git:
    checkout: { ref: { branch: main } }
  update:
    path: ./apps
    strategy: Setters
```

### مقایسه و دیدگاه‌های مختلف

| ویژگی | Flux | ArgoCD |
|---|---|---|
| معماری | کنترلرهای جدا | یکپارچه |
| UI | ضعیف/جدید | قوی |
| Image Automation | بومی عالی | با افزونه |
| Git-native | بله (CRD-based) | Application-based |
| یادگیری | متوسط | متوسط |

**دیدگاه:** Flux برای کسانی که می‌خواهند کاملاً در Git و K8s-native بمانند و قابلیت Image Automation قوی دارند، عالی است. ArgoCD برای UI/دید بصری و اکوسیستم (Argo Rollouts/Workflows) برتری دارد. هر دو CNCF graduated.

**انتخاب:** تیم بدون نیاز UI → Flux؛ تیم نیازمند دید بصری/SSO → ArgoCD.

### اشتباهات رایج

- **فراموش `prune: true`:** منابع حذف‌شده باقی می‌مانند.
- **Image automation بدون بررسی:** به‌روزرسانی خودکار image ممکن است نسخهٔ ناپایدار بیاورد (از semver range محدود استفاده کنید).
- **عدم bootstrap:** نصب دستی Flux → تنظیمات در Git نیست (دوباره با bootstrap بسازید).
- **مسیرهای اشتباه:** `path` در Kustomization باید نسبت به GitRepository باشد.
- **RBAC ضعیف برای automation:** ImageUpdateAutomation نیاز به دسترس write به Git دارد (کلید محافظت شود).

### نکات مهم و ترفندها

- **`fluxctl` / `flux` CLI:** برای دیباگ و trigger دستی.
- **`flux get kustomizations`:** وضعیت را ببینید.
- **Kustomize + Flux:** بهترین ترکیب برای Git-native.
- **Image Automation با PR:** به جای کامیت مستقیم، PR بسازید (بررسی انسانی).
- **Multi-tenancy:** Flux با Kustomize برای جدا کردن تیم‌ها.
- **Alerts:** notification-controller به Slack برای Degraded.

### خلاصه فصل

Flux رویکرد Git-native و controller-based به GitOps دارد (بدون UI مرکزی). کنترلرهای جداگانه (source/kustomize/helm/image) مسئولیت‌ها را تقسیم می‌کنند. ویژگی برجسته: **Image Automation** که خودکار image tag جدید را در Git کامیت می‌کند. کلید: bootstrap برای GitOps بودن خودِ Flux، `prune: true`، ImagePolicy با semver محدود، و استفاده از ImageUpdateAutomation با PR برای بررسی انسانی. Flux برای تیم‌های K8s-native و نیازمند خودکارسازی image ایده‌آل است.

---

## فصل ۷۰: مقایسه ArgoCD و Flux و بهترین روش‌ها

### مدمه

هر دو ابزار GitOps عالی هستند اما با فلسفه‌های متفاوت. این فصل راهنمای نهایی انتخاب و چک‌لیست بهترین روش‌های GitOps را ارائه می‌دهد تا بتوانید تصمیم آگاهانه بگیرید.

### مفاهیم اساسی

**ماتریس نهایی:**

| معیار | ArgoCD | Flux |
|---|---|---|
| UI/دید بصری | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| Git-native | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Image Automation | ⭐⭐ (افزونه) | ⭐⭐⭐⭐⭐ |
| اکوسیستم Argo | ⭐⭐⭐⭐⭐ (Rollouts/Workflows) | ⭐⭐ |
| یادگیری | متوسط | متوسط |
| Multi-cluster | عالی (ApplicationSet) | خوب (Kustomize) |

**رویکرد انتخاب:**
- **ArgoCD** اگر: تیم نیاز به UI دارد، از Argo Rollouts/Workflows استفاده می‌کند، یا چند کلاستر با مدیریت متمرکز می‌خواهد.
- **Flux** اگر: می‌خواهید کاملاً Git/K8s-native باشید، Image Automation قوی نیاز دارید، یا فلسفهٔ controller-based را ترجیح می‌دهید.

**بهترین روش‌های GitOps (Checklist):**
1. **Git = Source of Truth:** همه (کانفیگ + Secrets رمزنگاری‌شده).
2. **PR-based changes:** هیچ push مستقیم به main نباشد.
3. **Immutability:** Image tag صریح (SHA/SemVer)، نه `latest`.
4. **Automated sync + prune + selfHeal.**
5. **RBAC محدود** برای عامل.
6. **Drift detection و alerting.**
7. **Rollback plan** (git revert).
8. **Policy guardrails** (OPA/Gatekeeper).
9. **Separate repos** برای app-code و gitops-config (یا پوشهٔ جدا).
10. **Notifications** به Slack برای وضعیت.

**جدایی مخازن:** مخزن کد (app) جدا از مخزن GitOps (config) — CI image می‌سازد، GitOps مستقر می‌کند (تمیزترین جدایی).

### مثال‌های عملی

**۱. جدایی مخازن:**

```
org/app-code (CI: build+push image)
org/gitops-config (ArgoCD/Flux: sync به کلاستر)
```

**۲. GitOps Pipeline (CI فقط build):**

```yaml
# در app-code repo
build:
  script:
    - docker build -t registry/app:$SHA .
    - docker push registry/app:$SHA
    # Image Update Automation (Flux) یا Argo Image Updater
    # خودش GitOps-config را آپدیت می‌کند
```

**۳. محافظت شاخه:**

```
GitHub: Branch protection روی main
- Required review
- Required status checks
- No force push
```

**۴. Policy در GitOps:**

```yaml
# Gatekeeper constraint روی هر منبع توسط ArgoCD/Flux اعمال می‌شود
# قبل از sync، manifest بررسی می‌شود
```

### مقایسه و دیدگاه‌های مختلف

| دیدگاه | طرفداران |
|---|---|
| "ArgoCD برتر است" | به خاطر UI و اکوسیستم |
| "Flux ساده‌تر/原生" | به خاطر Git-native |
| "هر دو خوبند" | انتخاب بر اساس نیاز تیم |

**دیدگاه نهایی:** هیچ ابزار "بهتر" وجود ندارد؛ هر دو CNCF graduated و تولید-آماده‌اند. مهم‌تر از ابزار، **اعمال صحیح اصول GitOps** است (PR، immutable tags، automated sync، RBAC محدود). بسیاری با ArgoCD شروع می‌کنند (به خاطر UI) و بعد Flux اضافه می‌کنند (برای image automation).

**پاداش:** ArgoCD و Flux می‌توانند حتی در کنار هم روی کلاسترهای مختلف یا منابع متفاوت باشند (اما معمولاً یکی کافی است).

### اشتباهات رایج

- **استفاده از هر دو روی یک منابع:** تداخل reconcile.
- **GitOps بدون PR protection:** اشتباه در main → خرابی فوری تولید.
- **عامل با cluster-admin:** حمله به GitOps = کنترل کلاستر.
- **فراموش Notification:** کسی نمی‌فهمد sync شکست.
- **Image `latest` در GitOps:** عامل نمی‌فهمد چه تغییری کند.

### نکات مهم و ترفندها

- **شروع با ArgoCD** (UI کمک می‌کند)، سپس ارزیابی Flux.
- **Separate gitops repo** از app code.
- **Argo Image Updater / Flux Image Automation** برای به‌روزرسانی خودکار.
- **Test in staging first:** GitOps هم باید از dev→staging→prod عبور کند.
- **Disaster recovery:** کل کلاستر از Git بازسازی‌پذیر باشد (بوتسترپ).
- **Secret encryption:** Sealed Secrets / SOPS / External Secrets (فصل ۵۹).

### خلاصه فصل

ArgoCD (UI قوی، اکوسیستم Argo، ApplicationSet) و Flux (Git-native، Image Automation بومی، بدون UI) هر دو ابزارهای درجهٔ یک CNCF هستند. انتخاب بر اساس نیاز تیم است نه برتری مطلق. بهترین روش‌های GitOps: Git=Source of Truth، PR-based، immutable tags، automated sync+prune+selfHeal، RBAC محدود، drift alerting، و جدایی مخزن app-code از gitops-config. کلید: ابزار مهم‌تر از اصول نیست — اعمال صحیح اصول GitOps تعیین‌کنندهٔ موفقیت است.

---

## منابع و مراجع

- GitOps Principles (OpenGitOps): https://opengitops.dev
- ArgoCD Documentation: https://argo-cd.readthedocs.io
- ArgoCD ApplicationSet: https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/
- Flux Documentation: https://fluxcd.io
- Flux Image Automation: https://fluxcd.io/flux/guides/image-update/
- Argo Rollouts (Progressive Delivery): https://argoproj.github.io/rollouts/
- Sealed Secrets: https://github.com/bitnami-labs/sealed-secrets
- OPA Gatekeeper: https://open-policy-agent.github.io/gatekeeper/
- CNCF GitOps WG: https://github.com/cncf/tag-app-delivery

**بعدی:** [فایل ۱۵ — Terraform: زیرساخت به عنوان کد (IaC)](15-Terraform.md)
