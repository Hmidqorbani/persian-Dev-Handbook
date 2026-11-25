# فهرست کامل — DevOps از صفر تا تولید

## بخش ۰ — مقدمه و پایه‌ها

### فایل ۰۱ — [مقدمه بر DevOps](01-Introduction-to-DevOps.md)
- فصل ۱: DevOps چیست و چرا؟
- فصل ۲: فرهنگ DevOps و CALMS
- فصل ۳: چرخهٔ حیات DevOps و جریان ارزش
- فصل ۴: نقش‌ها، تیم‌ها و تحول سازمانی
- فصل ۵: چشم‌انداز ابزارها و نقشهٔ راه یادگیری

### فایل ۰۲ — [لینوکس، شل و شبکه برای DevOps](02-Linux-Shell-and-Networking.md)
- فصل ۶: لینوکس برای DevOps
- فصل ۷: اسکریپت‌نویسی شل و خودکارسازی
- فصل ۸: شبکه برای DevOps
- فصل ۹: امنیت پایه و مدیریت دسترسی
- فصل ۱۰: عیب‌یابی سیستمی و ابزارهای CLI

### فایل ۰۳ — [Git و استراتژی‌های Branching](03-Git-and-Branching-Strategies.md)
- فصل ۱۱: Git از پایه تا پیشرفته
- فصل ۱۲: استراتژی‌های Branching
- فصل ۱۳: Pull Request و Code Review
- فصل ۱۴: Git Hooks و خودکارسازی
- فصل ۱۵: Monorepo، Submodule و مدیریت مخزن

---

## بخش ۱ — CI/CD

### فایل ۰۴ — [مبانی CI/CD](04-CI-CD-Fundamentals.md)
- فصل ۱۶: CI/CD چیست؟
- فصل ۱۷: ساختار یک Pipeline
- فصل ۱۸: Artifacts و مدیریت نسخه
- فصل ۱۹: محیط‌ها و استقرار
- فصل ۲۰: استراتژی‌های استقرار

### فایل ۰۵ — [Jenkins](05-Jenkins.md)
- فصل ۲۱: نصب و معماری Jenkins
- فصل ۲۲: Pipeline as Code و Jenkinsfile
- فصل ۲۳: Plugins، Agents و Distributed Build
- فصل ۲۴: Jenkins در عمل و Best Practices

### فایل ۰۶ — [GitHub Actions](06-GitHub-Actions.md)
- فصل ۲۵: مفاهیم GitHub Actions
- فصل ۲۶: Workflows، Events و Jobs
- فصل ۲۷: Actions، Secrets و Runners
- فصل ۲۸: Matrix Build، Caching و Reusable Workflows

### فایل ۰۷ — [GitLab CI](07-GitLab-CI.md)
- فصل ۲۹: مفاهیم GitLab CI/CD
- فصل ۳۰: .gitlab-ci.yml، Stages و Jobs
- فصل ۳۱: Runners، Cache و Artifacts
- فصل ۳۲: Environments، Review Apps و Deploy

---

## بخش ۲ — کانتینر

### فایل ۰۸ — [مبانی Docker](08-Docker-Fundamentals.md)
- فصل ۳۳: کانتینر چیست؟
- فصل ۳۴: Image و Container
- فصل ۳۵: Dockerfile و Best Practices
- فصل ۳۶: Volume و Network
- فصل ۳۷: عیب‌یابی Docker

### فایل ۰۹ — [Docker پیشرفته و Registry](09-Docker-Advanced-and-Registry.md)
- فصل ۳۸: Docker Compose
- فصل ۳۹: Container Registry
- فصل ۴۰: امنیت کانتینر
- فصل ۴۱: بهینه‌سازی Image و Multi-stage Build

### فایل ۱۰ — [Podman و Buildah](10-Podman-and-Buildah.md)
- فصل ۴۲: Podman و اجرای rootless
- فصل ۴۳: Buildah و Skopeo
- فصل ۴۴: Podman در برابر Docker

---

## بخش ۳ — Kubernetes

### فایل ۱۱ — [مبانی Kubernetes](11-Kubernetes-Fundamentals.md)
- فصل ۴۵: Kubernetes چیست؟
- فصل ۴۶: معماری و اجزا
- فصل ۴۷: Pod
- فصل ۴۸: Deployment و ReplicaSet
- فصل ۴۹: Service و شبکهٔ داخلی
- فصل ۵۰: kubectl و چرخهٔ کاری

### فایل ۱۲ — [Kubernetes پیشرفته](12-Kubernetes-Advanced.md)
- فصل ۵۱: ConfigMap و Secret
- فصل ۵۲: Ingress و Gateway API
- فصل ۵۳: Storage و PersistentVolume
- فصل ۵۴: StatefulSet و DaemonSet
- فصل ۵۵: RBAC و امنیت
- فصل ۵۶: Resource Management و Autoscaling

### فایل ۱۳ — [Helm و Kustomize](13-Helm-and-Kustomize.md)
- فصل ۵۷: Helm و Chart
- فصل ۵۸: Values، Templates و Release
- فصل ۵۹: Kustomize
- فصل ۶۰: مدیریت Manifest و مقایسه

### فایل ۱۴ — [GitOps با ArgoCD و Flux](14-GitOps-ArgoCD-and-Flux.md)
- فصل ۶۱: GitOps چیست؟
- فصل ۶۲: ArgoCD
- فصل ۶۳: Flux
- فصل ۶۴: GitOps در عمل

---

## بخش ۴ — IaC و Configuration Management

### فایل ۱۵ — [Terraform](15-Terraform.md)
- فصل ۶۵: IaC چیست؟
- فصل ۶۶: Terraform و زبان HCL
- فصل ۶۷: State و Backend
- فصل ۶۸: Modules، Providers و Workspaces
- فصل ۶۹: Terraform در عمل و Best Practices

### فایل ۱۶ — [Pulumi و دیگر ابزارهای IaC](16-Pulumi-and-Other-IaC.md)
- فصل ۷۰: Pulumi
- فصل ۷۱: AWS CDK و Crossplane
- فصل ۷۲: مقایسهٔ ابزارهای IaC

### فایل ۱۷ — [Ansible](17-Ansible.md)
- فصل ۷۳: Ansible و معماری agentless
- فصل ۷۴: Playbook، Task و Module
- فصل ۷۵: Role، Inventory و Ansible Galaxy
- فصل ۷۶: Ansible در عمل و Best Practices

---

## بخش ۵ — Cloud

### فایل ۱۸ — [مفاهیم ابر و AWS](18-Cloud-Fundamentals-and-AWS.md)
- فصل ۷۷: مفاهیم ابر و مدل‌های سرویس
- فصل ۷۸: AWS — محاسبات و ذخیره‌سازی
- فصل ۷۹: AWS — شبکه و پایگاه داده
- فصل ۸۰: AWS — مدیریت، IAM و IaC
- فصل ۸۱: هزینه و بهینه‌سازی

### فایل ۱۹ — [Azure و GCP](19-Azure-and-GCP.md)
- فصل ۸۲: Azure
- فصل ۸۳: Google Cloud Platform
- فصل ۸۴: مقایسهٔ AWS، Azure و GCP
- فصل ۸۵: Multi-cloud و مهاجرت ابری

---

## بخش ۶ — Observability

### فایل ۲۰ — [Prometheus و Grafana](20-Prometheus-and-Grafana.md)
- فصل ۸۶: Observability چیست؟
- فصل ۸۷: Prometheus
- فصل ۸۸: PromQL و Alerting
- فصل ۸۹: Grafana و Dashboard

### فایل ۲۱ — [لاگ‌گیری با Loki و ELK](21-Logging-Loki-and-ELK.md)
- فصل ۹۰: لاگ متمرکز
- فصل ۹۱: Loki و Promtail
- فصل ۹۲: ELK Stack

### فایل ۲۲ — [OpenTelemetry و Tracing](22-OpenTelemetry-and-Tracing.md)
- فصل ۹۳: Distributed Tracing
- فصل ۹۴: OpenTelemetry
- فصل ۹۵: Jaeger و Tempo

---

## بخش ۷ — SRE

### فایل ۲۳ — [SRE و مدیریت Incident](23-SRE-and-Incident-Management.md)
- فصل ۹۶: SRE چیست؟
- فصل ۹۷: SLI، SLO و SLA
- فصل ۹۸: Error Budget
- فصل ۹۹: Toil و خودکارسازی
- فصل ۱۰۰: مدیریت Incident
- فصل ۱۰۱: Postmortem و Chaos Engineering

---

## بخش ۸ — DevSecOps و پیشرفته

### فایل ۲۴ — [DevSecOps](24-DevSecOps.md)
- فصل ۱۰۲: DevSecOps چیست؟
- فصل ۱۰۳: SAST، DAST و SCA
- فصل ۱۰۴: مدیریت Secrets و Vault
- فصل ۱۰۵: امنیت Supply Chain

### فایل ۲۵ — [Service Mesh](25-Service-Mesh.md)
- فصل ۱۰۶: Service Mesh چیست؟
- فصل ۱۰۷: Istio
- فصل ۱۰۸: Linkerd

### فایل ۲۶ — [Serverless](26-Serverless.md)
- فصل ۱۰۹: Serverless و FaaS
- فصل ۱۱۰: Knative و OpenFaaS
- فصل ۱۱۱: Serverless در ابرها

---

## بخش ۹ — پروژه و جمع‌بندی

### فایل ۲۷ — [پروژه‌های عملی (Capstone)](27-Capstone-Projects.md)
- فصل ۱۱۲: پروژهٔ CI/CD Pipeline کامل
- فصل ۱۱۳: پروژهٔ Microservices روی Kubernetes
- فصل ۱۱۴: پروژهٔ Monitoring Stack
- فصل ۱۱۵: پروژهٔ IaC کامل

### فایل ۲۸ — [Best Practices، چیت‌شیت و واژه‌نامه](28-Best-Practices-CheatSheets-Glossary.md)
- فصل ۱۱۶: Best Practices سراسری
- فصل ۱۱۷: چیت‌شیت‌ها
- فصل ۱۱۸: عیب‌یابی خطاهای رایج
- فصل ۱۱۹: واژه‌نامه و منابع

---

## آمار کتابچه

| شاخص | مقدار |
|---|---|
| تعداد فایل | ۲۸ |
| تعداد فصل | ۱۱۹ |
| بخش‌ها | ۹ |
| مدت تخمینی مطالعه | ۶۰–۸۰ ساعت |
| سطح | مبتدی تا متوسط |
