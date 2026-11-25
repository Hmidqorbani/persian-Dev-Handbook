# فایل ۲۵ — سرویس مش (Service Mesh: Istio و Linkerd)

**مدت یادگیری**: ۱۱ ساعت
**سطح سختی**: سخت
**پیش‌نیاز**: فایل ۱۱/۱۲ (Kubernetes پیشرفته)، فایل ۲۰/۲۲ (Observability)، فایل ۲۴ (امنیت/Kyverno)

---

## فهرست محتوا

- [فصل ۱۲۰: مبانی Service Mesh و مسئلهٔ آن](#فصل-۱۲۰-مبانی-service-mesh-و-مسئلهٔ-آن)
- [فصل ۱۲۱: Istio — معماری و مدیریت ترافیک](#فصل-۱۲۱-istio--معماری-و-مدیریت-ترافیک)
- [فصل ۱۲۲: Linkerd — جایگزین سبک](#فصل-۱۲۲-linkerd--جایگزین-سبک)
- [فصل ۱۲۳: قابلیت‌های کلیدی (mTLS، ترافیک، مشاهده‌پذیری)](#فصل-۱۲۳-قابلیتهای-کلیدی-mtls-ترافیک-مشاهدهپذیری)
- [فصل ۱۲۴: سناریوی عملی — پیاده‌سازی Mesh + راهنمای انتخاب](#فصل-۱۲۴-سناریوی-عملی--پیادهسازی-mesh--راهنمای-انتخاب)

---

## فصل ۱۲۰: مبانی Service Mesh و مسئلهٔ آن

### مقدمه

وقتی ۳ سرویس دارید، ارتباطشان ساده است. وقتی ۵۰ سرویس (میکروسرویس) دارید، سوال‌ها زیاد می‌شوند: ترافیک را چطور مسیریابی کنیم؟ چطور همه‌جا mTLS داشته باشیم؟ چطور retry/circuit-breaker یکسان در همه اعمال شود؟ **Service Mesh** این «منطق ارتباطی» را از کد جدا کرده و به لایه‌ای زیربنایی می‌سپارد. این فصل ضرورت و معماری را می‌سازد.

### مفاهیم اساسی

**مسئله (Problem Space):**
در میکروسرویس، هر سرویس باید: احراز هویت متقابل (mTLS)، retry، timeout، circuit breaking، نظارت ترافیک، و تزریق خطا (fault injection) داشته باشد. نوشتن این در هر زبان/سرویس = تکرار و ناسازگاری.

**تعریف Service Mesh:**
لایهٔ زیرساختی تخصصی برای **ارتباط سرویس‌به‌سرویس (service-to-service)**. منطق شبکه را از کد جدا می‌کند.

**الگوی Sidecar:**
هر Pod یک پروکسی کوچک (مثل Envoy) کنار کانتینر اصلی تزریق می‌شود. تمام ترافیک ورودی/خروجی از پروکسی عبور می‌کند.

**معماری دو لایه:**
- **Data Plane (صفحهٔ داده):** پروکسی‌های sidecar که ترافیک واقعی را مدیریت می‌کنند.
- **Control Plane (صفحهٔ کنترل):** مغز مدیریت — تنظیمات را به پروکسی‌ها توزیع می‌کند، سیاست‌ها را اعمال می‌کند.

**مسئولیت‌های Mesh:**
- **Traffic Management:** مسیریابی، تقسیم ترافیک (traffic split)، retry/timeout.
- **Security:** mTLS خودکار، AuthorizationPolicy.
- **Observability:** متریک/تریس هر درخواست سرویس‌به‌سرویس (یکپارچه با فصل ۲۲).
- **Resilience:** circuit breaking، fault injection، rate limiting.

**زمانی که نیاز نیست:** اپلیکیشن تک‌سرویسه یا چند سرویس ساده → Mesh اضافه‌بار (overhead) دارد.

### مثال‌های عملی

**الف) معماری Sidecar (نمای کلی):**
```
Pod:
 ├─ container: orders (کد برنامه)
 └─ container: envoy (sidecar proxy)  ← همهٔ ترافیک از اینجا رد می‌شود
ترافیک orders→payment واقعاً orders→envoy→envoy(payment)→payment است
```

**ب) بدون Mesh (در کد):**
```python
# هر سرویس باید این را خودش پیاده کند:
session = requests.Session()
session.mount('https://', RetryAdapter(max_retries=3))
# + کد mTLS + circuit breaker ... در ۵۰ سرویس تکرار!
```

**ج) با Mesh (خارج از کد):**
```
تنظیم یک بار در Control Plane → روی همهٔ سرویس‌ها اعمال می‌شود
کد فقط منطق کسب‌وکار را می‌نویسد
```

### مقایسه و دیدگاه‌های مختلف

**Service Mesh در برابر API Gateway:**
- Gateway: ورودی خارجی (north-south traffic، اینترنت→سرویس).
- Mesh: ترافیک داخلی (east-west، سرویس→سرویس).
- هر دو مکمل‌اند (معمولاً Gateway + Mesh با هم).

**Mesh در برابر Ingress Controller:**
- Ingress: مسیریابی HTTP ورودی به K8s.
- Mesh: مدیریت داخل خوشه.

**دیدگاه‌ها:**
- **طرفداران:** یکپارچگی امنیت/مشاهده‌پذیری، کاهش کد تکراری.
- **منتقدان:** پیچیدگی، latency اضافه (~ms)، یادگیری سخت.
- **رویکرد عملی:** «اگر >۱۰ سرویس و نیاز به mTLS/observability یکپارچه دارید → Mesh؛ وگرنه Ingress + App-level کافی است».

### اشتباهات رایج

1. **نصب Mesh برای اپلیکیشن تک‌سرویسه:** overhead بی‌دلیل (پیچیدگی + latency).
2. **فراموشی overhead شبکه:** هر درخواست حالا از دو پروکسی رد می‌شود → تأخیر جزئی.
3. **عدم درک Data/Control Plane:** وقتی Control Plane خراب شود، ترافیک ممکن است مختل شود.
4. **پیچیدگی بیش از نیاز:** همهٔ قابلیت‌ها را روشن کردن → گیج‌کننده.
5. **عدم مانیتورینگ خودِ Mesh:** خودِ sidecarها متریک/تریس تولید می‌کنند — باید جمع‌آوری شوند.

### نکات مهم و ترفندها

- **شروع تدریجی:** فقط mTLS + observability فعال کنید؛ بقیه بعداً.
- **متریک‌های Mesh را به Prometheus وصل کنید** (فصل ۹۷) — sidecarها `/stats` درخور دارند.
- **تریس را با OTel یکپارچه کنید** (فصل ۱۰۸) — Mesh تریس هر hop را می‌دهد.
- **Canary با Mesh:** تقسیم ترافیک ۹۰/۱۰ برای انتشار تدریجی (فصل ۴/۱۴).
- **mTLS را به‌صورت strict در تولید فعال کنید** (تمام ترافیک داخل رمزنگاری شود).

### خلاصه فصل

**Service Mesh** منطق ارتباط سرویس‌به‌سرویس (mTLS، retry، مسیریابی، نظارت) را از کد جدا کرده و به **Sidecar proxy** (Data Plane) تحت کنترل **Control Plane** می‌سپارد. برای میکروسرویس‌های بزرگ ضروری است؛ برای سیستم کوچک overhead بی‌دلیل. اشتباه کلیدی: نصب بی‌مورد و نادیده گرفتن overhead. در فصل بعد Istio (محبوب‌ترین) را بررسی می‌کنیم.

---

## فصل ۱۲۱: Istio — معماری و مدیریت ترافیک

### مدمه

**Istio** پرکاربردترین Service Mesh است (مبتنی بر Envoy). قابلیت‌های غنی برای ترافیک، امنیت و مشاهده‌پذیری دارد. این فصل معماری (istiod + Envoy) و ابزارهای مدیریت ترافیک (VirtualService/DestinationRule) را بررسی می‌کند.

### مفاهیم اساسی

**معماری Istio:**
- **Envoy:** پروکسی sidecar (Data Plane) — تمام ترافیک از آن عبور می‌کند.
- **istiod (Control Plane):** ترکیب قدیمِ Pilot + Citadel + Galley. مسئول: توزیع تنظیمات به Envoy، صدور گواهی mTLS، اعتبارسنجی CRDها.
- **CRDهای کلیدی:**
  - **VirtualService (VS):** قوانین مسیریابی (مسیر/هدر/وزن).
  - **DestinationRule (DR):** سیاست مقصد (load balancing، subsetها، circuit breaker).
  - **Gateway:** نقطهٔ ورود ترافیک خارجی به Mesh.
  - **PeerAuthentication / AuthorizationPolicy:** مقررات mTLS و دسترسی.

**مدیریت ترافیک (Traffic Management):**
- **Routing پیشرفته:** بر اساس مسیر (`/api/v1`)، هدر، وزن.
- **Traffic Split (کاناری):** ۹۰٪ → v1، ۱۰٪ → v2.
- **Retry / Timeout:** در سطح mesh (نه کد).
- **Circuit Breaking:** قطع درخواست به مقصدِ ناسالم.

**نصب:**
- روش قدیم: `istioctl install`.
- جدید (Ambient Mesh): بدون sidecar (layer 4/7 جداگانه) — کاهش overhead.

### مثال‌های عملی

**الف) نصب Istio (istioctl):**
```bash
istioctl install --set profile=demo -y
kubectl label namespace default istio-injection=enabled
# پادهای جدید به‌طور خودکار sidecar می‌گیرند
```

**ب) VirtualService — کاناری ۹۰/۱۰:**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata: { name: orders-route }
spec:
  hosts: ["orders.local"]
  http:
    - route:
        - destination: { host: orders, subset: v1 }
          weight: 90
        - destination: { host: orders, subset: v2 }
          weight: 10
```

**ج) DestinationRule — subsets + circuit breaker:**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata: { name: orders-dr }
spec:
  host: orders
  trafficPolicy:
    connectionPool:
      http: { http1MaxPendingRequests: 100, maxRequestsPerConnection: 10 }
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 5m
  subsets:
    - name: v1
      labels: { version: v1 }
    - name: v2
      labels: { version: v2 }
```

**د) PeerAuthentication (mTLS strict):**
```yaml
apiVersion: security.istio.io/v1
kind: PeerAuthentication
metadata: { name: default }
spec:
  mtls: { mode: STRICT }   # تمام ترافیک داخل رمزنگاری اجباری
```

### مقایسه و دیدگاه‌های مختلف

**Istio در برابر راه‌حل‌های App-level:**
- Istio: یکبار تنظیم، روی همه اعمال، زبان‌مستقل.
- App-level: کنترل دقیق‌تر اما تکرار در هر سرویس.

**Sidecar در برابر Ambient (Istio جدید):**
- Sidecar: بالغ، اما یک کانتینر اضافه به هر Pod.
- Ambient: بدون sidecar، سبک‌تر، اما جدیدتر (بررسی پایداری).

**VirtualService در برابر Ingress:**
- VS: مسیریابی درون mesh (host/path based).
- Ingress/Gateway: ورودی خارجی.

**دیدگاه:** Istio قدرتمند اما «یادگیری دامنهٔ وسیع» دارد؛ تیم‌های کوچک ممکن است Linkerd را ترجیح دهند (فصل بعد).

### اشتباهات رایج

1. **فعال کردن mTLS STRICT قبل از آماده‌سازی همه:** سرویس‌های قدیمی بدون sidecar قطع می‌شوند.
2. **عدم درک weight در VS:** اشتباه `weight` باعث قطع ترافیک می‌شود.
3. **CRDهای ناسازگار:** نسخهٔ istiod و CRD باید هم‌راستا باشند.
4. **فراموشی منابع sidecar:** هر Pod حالا RAM/CPU بیشتری مصرف می‌کند.
5. **تنظیم circuit breaker خیلی تهاجمی:** خروج زودهنگام همهٔ پادها (cascading failure).

### نکات مهم و ترفندها

- **Kiali** را برای نقشهٔ ترافیک بصری نصب کنید (همراه Istio).
- **grafana add-on** متریک mesh را می‌دهد.
- **`istioctl analyze`** پیکربندی را قبل از اعمال اعتبارسنجی کند.
- **Canary را با Argo Rollouts ترکیب کنید** (فصل ۱۴) برای کاناری خودکار.
- **Ambient Mesh** را برای محیط جدید بررسی کنید (کاهش هزینه منابع).
- **ابتدا PERMISSIVE mTLS** (اجازه هر دو) سپس به STRICT بروید.

### خلاصه فصل

**Istio** با **Envoy (Data Plane)** و **istiod (Control Plane)** ترافیک، امنیت و نظارت میکروسرویس را مدیریت می‌کند. **VirtualService** (مسیریابی/وزن) و **DestinationRule** (subset/circuit breaker) ابزارهای اصلی‌اند؛ **PeerAuthentication** mTLS را اجباری می‌کند. اشتباهات: فعال‌سازی زودهنگام STRICT و تنظیمات تهاجمی circuit breaker. ابزارهای Kiali/Grafana دید بصری می‌دهند. در فصل بعد Linkerd (سبک‌تر) را می‌بینیم.

---

## فصل ۱۲۲: Linkerd — جایگزین سبک

### مقدمه

**Linkerd** قدیمی‌ترین Service Mesh است که روی «سادگی و سبکی» تمرکز دارد. برخلاف Istio که ویژگی‌های فراوان دارد، Linkerd «فقط کارهای ضروری را درست انجام می‌دهد» با overhead کمتر. این فصل معماری و زمانی که باید Linkerd را به Istio ترجیح داد بررسی می‌کند.

### مفاهیم اساسی

**فلسفه Linkerd:**
- «Mesh کوچک و ایمن» — کمترین پیچیدگی، کمترین منابع.
- تمرکز: mTLS خودکار، مشاهده‌پذیری (golden metrics)، و traffic split ساده.

**معماری:**
- **Micro-proxy (linkerd2-proxy):** پروکسی بسیار سبک (Rust، ~۱۰-۲۰MB RAM) به‌جای Envoy سنگین.
- **Control Plane:** اجزای کوچک (destination، identity، proxy-injector).
- **No CRD پیچیده:** تنظیمات با annotation ساده‌تر از Istio.

**قابلیت‌ها:**
- **Automatic mTLS:** همهٔ ترافیک TCP بین پادهای injected رمزنگاری می‌شود.
- **Golden Metrics:** Success rate، RPS، latency (p50/p95/p99) برای هر سرویس.
- **Traffic Split (CRD):** تقسیم ترافیک برای کاناری (مثل Flagger).
- **Retries / Timeouts:** ساده از طریق annotation.

**تفاوت کلیدی با Istio:**
- Linkerd: سبک‌تر، ساده‌تر، کمترین یادگیری، اما ویژگی کمتر (مثلاً WASM filter پیچیده ندارد).
- Istio: سنگین‌تر، غنی‌تر، انعطاف بیشتر.

### مثال‌های عملی

**الف) نصب Linkerd (CLI):**
```bash
linkerd install --crds | kubectl apply -f -
linkerd install | kubectl apply -f -
linkerd check   # تأیید سلامت
```

**ب) تزریق sidecar (annotation یا namespace):**
```bash
kubectl get deploy orders -n app -o yaml | linkerd inject - | kubectl apply -f -
# یا برچسب زدن namespace:
kubectl annotate namespace app linkerd.io/inject=enabled
```

**ج) Traffic Split (کاناری):**
```yaml
apiVersion: split.smi-spec.io/v1alpha2
kind: TrafficSplit
metadata: { name: orders-split }
spec:
  service: orders
  splits:
    - { weight: 90, service: orders-v1 }
    - { weight: 10, service: orders-v2 }
```

**د) مشاهدهٔ متریک (Linkerd dashboard):**
```bash
linkerd dashboard &   # نمودار success rate/latency هر سرویس
linkerd stat deploy   # خروجی متنی متریک‌ها
```

### مقایسه و دیدگاه‌های مختلف

**Linkerd در برابر Istio:**

| معیار | Linkerd | Istio |
|---|---|---|
| حجم پروکسی | بسیار سبک (~۱۵MB) | سنگین‌تر (Envoy) |
| یادگیری | ساده | دامنهٔ وسیع |
| ویژگی‌ها | ضروری | بسیار غنی |
| mTLS | خودکار/پیش‌فرض | پیکربندی بیشتر |
| مناسب برای | تیم کوچک/سادگی | سازمانی/پیچیده |

**انتخاب:**
- نیاز به «فقط mTLS + نظارت + کاناری ساده» و منابع کم → Linkerd.
- نیاز به «سیاست پیچیده، WASM، گیت‌وی قدرتمند» → Istio.

**دیدگاه:** «Linkerd برای کسانی است که می‌خواهند Mesh داشته باشند بدون دردسر؛ Istio برای کسانی است که می‌خواهند همه‌چیز را کنترل کنند».

### اشتباهات رایج

1. **انتظار همهٔ ویژگی‌های Istio از Linkerd:** مثلاً فیلترهای سفارشی پیچیده (WASM) ندارد.
2. **عدم اجرای `linkerd check` پیش از تولید:** مشکلات زیربنایی پنهان می‌مانند.
3. **فراموشی منابع پروکسی:** حتی سبک، باز هم RAM اضافه دارد.
4. **مخلوط کردن دو Mesh:** Istio و Linkerd روی یک namescape = تداخل.

### نکات مهم و ترفندها

- **Buoyant (شرکت پشت Linkerd)** ابزارهای تجاری (Buoyant Cloud) برای SLO ارائه می‌دهد.
- **همبستگی با Flagger** (فصل ۱۴) برای کاناری خودکار عالی است.
- **Multicluster Linkerd:** اتصال دو خوشه با mTLS (برای فایل ۲۸).
- **ابزار `linkerd viz`** برای تریس/متریک سریع.
- **شروع با یک namespace** (نه کل خوشه) برای یادگیری ایمن.

### خلاصه فصل

**Linkerd** بر سادگی و سبکی (micro-proxy Rust) تمرکز دارد — mTLS خودکار، golden metrics، و traffic split ساده با یادگیری کم. در مقایسه با Istio، ویژگی کمتر اما پیچیدگی بسیار کمتر دارد. انتخاب میان آن‌ها بر اساس نیاز به غنای ویژگی در برابر سادگی است. اشتباه: انتظار قابلیت‌های پیشرفتهٔ Istio. در فصل بعد قابلیت‌های مشترک (mTLS/ترافیک/مشاهده‌پذیری) را عمیق‌تر می‌بینیم.

---

## فصل ۱۲۳: قابلیت‌های کلیدی (mTLS، ترافیک، مشاهده‌پذیری)

### مقدمه

چه Istio چه Linkerd، سه قابلیت را ارائه می‌دهند که بدون Mesh سخت است: **امنیت (mTLS)**، **انعطاف ترافیک**، و **مشاهده‌پذیری سرویس‌به‌سرویس**. این فصل این سه را با مثال عملی و پیوند به فصل‌های قبل (۲۰/۲۲) بررسی می‌کند.

### مفاهیم اساسی

**۱. امنیت — mTLS خودکار:**
- Mesh گواهی‌ها را صادر و می‌چرخاند (بدون مدیریت دستی).
- هر درخواست سرویس‌به‌سرویس رمزنگاری و احراز هویت می‌شود (Zero Trust درون خوشه).
- **مزیت:** حتی اگر ترافیک sniff شود، خوانا نیست.

**۲. انعطاف ترافیک (Resilience):**
- **Retry:** تلاش مجدد خودکار روی خطای موقت (با «بازنشستگی نمایی» برای جلوگیری از طوفان).
- **Timeout:** محدود کردن زمان انتظار.
- **Circuit Breaker:** خروج مقصدِ ناسالم از چرخش (جلوگیری از cascade failure).
- **Fault Injection:** تزریق خطا (۵۰۰/تأخیر) برای تست مقاومت (هم‌ارزش chaos محدود).

**۳. مشاهده‌پذیری (Observability):**
- هر درخواست سرویس‌به‌سرویس متریک (RED: rate/error/duration) و تریس تولید می‌کند.
- **یکپارچگی با Prometheus/Grafana** (فصل ۹۷/۹۹): متریک mesh به همان داشبورد.
- **یکپارچگی با OTel/Tempo** (فصل ۱۰۸): تریس هر hop (سرویس→سرویس) دیده می‌شود.
- **Golden Metrics:** success rate، RPS، latency per service — تشخیص سریع bottleneck.

**۴. Traffic Split / Canary:**
- تقسیم ترافیک برای انتشار تدریجی (فصل ۴/۱۴).

### مثال‌های عملی

**الف) retry + timeout در Istio (VirtualService):**
```yaml
http:
  - route:
      - destination: { host: payment }
    retries:
      attempts: 3
      perTryTimeout: 2s
      retryOn: 5xx,reset
    timeout: 5s
```

**ب) fault injection (تست مقاومت):**
```yaml
http:
  - fault:
      delay: { percent: 50, fixedDelay: 3s }   # ۵۰٪ درخواست ۳ ثانیه تأخیر
      abort: { percent: 10, httpStatus: 500 }   # ۱۰٪ خطای ۵۰۰ تزریق
    route:
      - destination: { host: payment }
```

**ج) مشاهده در Grafana (متریک mesh):**
```
# نرخ موفقیت سرویس orders
sum(rate(istio_requests_total{destination_service="orders"}[5m])) by (response_code)
# یا linkerd_success_total
```

**د) تریس با OTel (نمای کلی):**
```
Gateway → [envoy span] → orders → [envoy span] → payment
هر hop یک span در Tempo (فصل ۱۰۹)
```

### مقایسه و دیدگاه‌های مختلف

**Mesh mTLS در برابر App-level mTLS:**
- Mesh: یکبار تنظیم، روی همه اعمال، چرخش خودکار.
- App: کنترل دقیق اما تکرار و خطای انسانی.

**Retry در Mesh در برابر در کد:**
- Mesh: یکجا، زبان‌مستقل.
- کد: دقیق‌تر اما در ۵۰ سرویس تکرار.

**دیدگاه:** «Mesh مشاهده‌پذیری را از «سیاه‌چالهٔ شبکه» به «نمودار شفاف» تبدیل می‌کند» — ارزش اصلی آن همین دید است.

### اشتباهات رایج

1. **retry بدون backoff:** طوفان درخواست روی سرویس نیمه‌شکسته.
2. **fault injection در تولید بدون هدف:** اختلال واقعی ایجاد می‌کند (فقط در تست/staging).
3. **عدم جمع‌آوری متریک mesh:** داشبورد خالی → ارزش mesh دیده نمی‌شود.
4. **timeout خیلی کوتاه:** درخواست‌های موفق قطع می‌شوند.
5. **circuit breaker بدون آستانهٔ بازگشت:** نیمی از ترافیک همیشه رد می‌شود.

### نکات مهم و ترفندها

- **همیشه mTLS را در تولید STRICT کنید** (پس از آماده‌سازی کامل).
- **Retry با exponential backoff + jitter** (پیش‌فرض Istio خوب است).
- **متریک mesh را به همان Prometheus وصل کنید** که فصل ۹۷ ساختید.
- **تریس mesh را به Tempo/OTel بفرستید** (فصل ۱۰۸) — دید کامل هر hop.
- **Fault injection را در محیط تست با Game Day ترکیب کنید** (فصل ۱۱۴).
- **SLO روی متریک mesh بسازید** (فصل ۱۱۲) — success rate سرویس‌به‌سرویس.

### خلاصه فصل

سه قابلیت محوری Mesh — **mTLS خودکار** (Zero Trust درون خوشه)، **انعطاف ترافیک** (retry/timeout/circuit breaker/fault injection)، و **مشاهده‌پذیری سرویس‌به‌سرویس** (golden metrics + تریس هر hop) — بدون آن در میکروسرویس بسیار سخت‌اند. همه با **Prometheus/Grafana/OTel** (فصل‌های ۲۰/۲۲) یکپارچه می‌شوند. اشتباهات: retry بدون backoff و fault injection در تولید. در فصل بعد سناریوی end-to-end و راهنمای انتخاب را می‌بینیم.

---

## فصل ۱۲۴: سناریوی عملی — پیاده‌سازی Mesh + راهنمای انتخاب

### مقدمه

همه را در یک سناریو ترکیب می‌کنیم: نصب Linkerd/Istio، فعال‌سازی mTLS، اجرای کاناری با traffic split، و مشاهده با Grafana/Tempo. سپس راهنمای انتخاب میان ابزارها را ارائه می‌دهیم.

### مفاهیم اساسی

**گام‌های سناریو (با Linkerd برای سادگی):**
1. نصب Linkerd + برچسب‌زنی namespace.
2. تزریق sidecar در سرویس‌های `orders` و `payment`.
3. فعال‌سازی mTLS (پیش‌فرض روشن است).
4. اعمال TrafficSplit برای کاناری نسخهٔ جدید.
5. مشاهدهٔ متریک در Linkerd dashboard + Grafana.
6. اجرای fault injection (در staging) برای تست مقاومت.

**ارزیابی موفقیت:** ترافیک رمزنگاری‌شده، success rate قابل مشاهده، و انتشار کاناری بدون قطع.

**راهنمای انتخاب Mesh:**
- **Linkerd:** تیم کوچک، می‌خواهید سادگی، منابع کم، mTLS+نظارت سریع.
- **Istio:** سازمان بزرگ، نیاز به گیت‌وی قدرتمند، سیاست پیچیده، WASM.
- **Consul Connect (HashiCorp):** اگر قبلاً Consul دارید / multi-cloud خارج از K8s.
- **Cilium Service Mesh:** مبتنی بر eBPF (بدون sidecar) — نوظهور و سریع.
- **بدون Mesh:** سیستم کوچک → Ingress + App-level کافی.

### مثال‌های عملی

**الف) سناریوی کامل (Linkerd):**
```bash
# ۱. نصب
linkerd install | kubectl apply -f -
# ۲. تزریق
kubectl annotate ns app linkerd.io/inject=enabled
kubectl apply -f orders.yaml -n app
# ۳. بررسی mTLS
linkerd edges deploy -n app   # ستون "MESHED" و "TLS"
# ۴. کاناری
kubectl apply -f traffic-split.yaml   # ۹۰/۱۰ بین v1/v2
# ۵. مشاهده
linkerd dashboard
```

**ب) بررسی سلامت (CLI):**
```bash
linkerd stat deploy -n app
# خروجی:
# NAME     SUCCESS   RPS   LATENCY_P50   LATENCY_P99
# orders   100.0%    12.3  2ms          18ms
# payment  99.8%     11.9  3ms          25ms
```

**ج) پیوند به Grafana (متریک mesh):**
```
در Grafana دادهٔ source جدید: Prometheus در حال scrape لینکرد
نمودار: sum(rate(linkerd_proxy_requests_total[5m])) by (dst_service)
```

**د) فهرست تصمیم‌گیری (ماتریس):**
```
[ ] تعداد سرویس > ۱۰؟  → بله → Mesh در نظر بگیرید
[ ] منابع محدود؟        → بله → Linkerd / Cilium
[ ] سیاست پیچیده لازم؟  → بله → Istio
[ ] خارج از K8s هم هست؟  → بله → Consul
```

### مقایسه و دیدگاه‌های مختلف

**نصب مرحله‌ای در برابر یکجا:**
- مرحله‌ای (یک namespace): یادگیری ایمن، بازگشت آسان.
- یکجا (کل خوشه): سریع اما ریسک بالا.

**Cilium eBPF در برابر Sidecar:**
- eBPF: بدون sidecar، کارایی بالا، اما نوظهور (بررسی بلوغ).
- Sidecar: بالغ، مستندات فراوان.

**دیدگاه:** «Mesh ابزار است نه هدف — فقط وقتی نصبش کنید که مسئلهٔ واقعی حل کند (mTLS/نظارت یکپارچه در مقیاس)».

### اشتباهات رایج

1. **نصب روی کل خوشهٔ تولید دفعهٔ اول:** ریسک بالا؛ با staging/یک namespace شروع کنید.
2. **فراموشی به‌روزرسانی sidecar:** نسخهٔ جدید پاد نصب شد اما پروکسی قدیمی ماند.
3. **عدم تست بازگشت (rollback):** وقتی mesh مشکل ایجاد کرد، چگونه حذفش کنید؟
4. **انتخاب بر اساس «محبوب‌ترین» نه «نیاز»:** Istio همیشه بهترین نیست.

### نکات مهم و ترفندها

- **همیشه plan برای rollback داشته باشید** (برچسب injection را بردارید → بازسازی پاد بدون sidecar).
- **متریک/تریس mesh را به همان observability stack وصل کنید** (فصل ۲۲).
- **Canary را با Flagger خودکار کنید** (فصل ۱۴) — کاهش دخالت دستی.
- **Multicluster:** برای DR، Linkerd/Istio اتصال خوشه‌ای با mTLS دارند.
- **هزینهٔ منابع را اندازه بگیرید:** sidecarها RAM مصرف می‌کنند — بودجهٔ کافی بگذارید.

### خلاصه فصل

سناریوی عملی نشان داد چگونه نصب Linkerd/Istio → تزریق sidecar → mTLS خودکار → TrafficSplit کاناری → مشاهده با Grafana/Tempo یک «شبکهٔ هوشمند» می‌سازد. **راهنمای انتخاب:** Linkerd برای سادگی/منابع کم، Istio برای غنا، Cilium برای eBPF، و هیچ‌کدام برای سیستم کوچک. اشتباهات: نصب یکجا روی تولید و نداشتن plan بازگشت. با این فایل بخش شبکه پیشرفته (Service Mesh) کامل شد؛ فایل بعدی به Serverless می‌رویم.

---

## منابع و مراجع

- Istio Documentation — https://istio.io/latest/docs/
  - Traffic Management: https://istio.io/latest/docs/concepts/traffic-management/
  - Security (mTLS): https://istio.io/latest/docs/concepts/security/
  - Ambient Mesh: https://istio.io/latest/docs/ambient/
- Linkerd Documentation — https://linkerd.io/2.14/overview/
  - Getting Started: https://linkerd.io/2.14/getting-started/
  - Traffic Split: https://linkerd.io/2.14/features/traffic-split/
- Buoyant (Linkerd company) — https://buoyant.io/
- Cilium Service Mesh — https://cilium.io/use-cases/service-mesh/
- CNCF Service Mesh Comparison — https://layer5.io/learn/service-mesh-comparison
- «Istio: Up and Running» (O'Reilly)
- SMI (Service Mesh Interface) Spec — https://smi-spec.io/

**بعدی:** [فایل ۲۶ — محاسبات بدون‌سرور (Serverless)](26-Serverless.md)
