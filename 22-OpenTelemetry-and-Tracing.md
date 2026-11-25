# فایل ۲۲ — OpenTelemetry و Tracing (Observability بخش ۳)

**مدت یادگیری**: ۱۰ ساعت
**سطح سختی**: متوسط تا سخت
**پیش‌نیاز**: فایل ۲۰ (Prometheus/Grafana)، فایل ۲۱ (Logging/Loki)، فایل ۱۱ (Kubernetes مبانی)

---

## فهرست محتوا

- [فصل ۱۰۶: مبانی Distributed Tracing و ضرورت آن](#فصل-۱۰۶-مبانی-distributed-tracing-و-ضرورت-آن)
- [فصل ۱۰۷: OpenTelemetry (OTel) — استاندارد یکپارچه](#فصل-۱۰۷-opentelemetry-otel--استاندارد-یکپارچه)
- [فصل ۱۰۸: Instrumentation عملی با OTel](#فصل-۱۰۸-instrumentation-عملی-با-otel)
- [فصل ۱۰۹: Tracing در معماری میکروسرویس](#فصل-۱۰۹-tracing-در-معماری-میکروسرویس)
- [فصل ۱۱۰: سناریوی عملی — Observability یکپارچه (Trilogy)](#فصل-۱۱۰-سناریوی-عملی--observability-یکپارچه-trilogy)

---

## فصل ۱۰۶: مبانی Distributed Tracing و ضرورت آن

### مقدمه

در یک معماری تک‌سرویسه، لاگ و متریک کافی‌اند. اما وقتی یک درخواست از **Gateway → Auth → Orders → Payment → DB** می‌گذرد، کجا کند است؟ **Distributed Tracing** مسیر کامل یک درخواست را در میان سرویس‌ها ترسیم می‌کند — ستون سوم Observability (پس از Metrics و Logs). این فصل مفاهیم trace/span و ضرورت آن را می‌سازد.

### مفاهیم اساسی

**چرا Tracing؟**
- در میکروسرویس، یک باگ ممکن است حاصل تعامل ۱۰ سرویس باشد؛ هیچ لاگ تک‌سرویسه پاسخ نمی‌دهد.
- شناسایی **bottleneck**: کدام سرویس ۸۰٪ زمان را می‌گیرد؟
- درک وابستگی‌ها (service dependency graph).

**مفاهیم کلیدی:**
- **Trace (ردیابی):** کل سفر یک درخواست از شروع تا پایان.
- **Span (گستره):** یک واحد کار در یک سرویس (مثلاً «کوئری DB»). شامل: نام، زمان شروع/پایان، parent span id، attributes (برچسب)، events.
- **Parent/Child:** spanها درختی‌اند؛ child نشان‌دهندهٔ زیربخش درخواست.
- **Context Propagation (اشاعهٔ بافتار):** انتقال شناسهٔ trace میان سرویس‌ها (معمولاً در هدر HTTP `traceparent` طبق استاندارد W3C).
- **Sampling (نمونه‌برداری):** ثبت همهٔ تریس‌ها در ترافیک بالا گران است؛ بخشی (مثلاً ۱۰٪) نمونه‌برداری می‌شود. Head-based (در ورودی) یا Tail-based (پس از دیدن کل تریس — برای خطاها ۱۰۰٪).

**تفاوت با Logs/Metrics:**
- متریک: «چند درخواست کند بود؟»
- لاگ: «چه خطایی در سرویس X رخ داد؟»
- تریس: «درخواست شماره ۴۲۹۱ دقیقاً کجا کند بود؟»

**سیستم‌های قدیمی:** Jaeger (اوبر)، Zipkin — پیش‌نیازهای OTel.

### مثال‌های عملی

**الف) ساختار یک Trace (نمایش درختی):**
```
Trace #abc
├─ Gateway        (5ms)
│  └─ Auth        (10ms)
│     └─ Orders   (120ms)  ← bottleneck
│        ├─ DB    (100ms)
│        └─ Payment (15ms)
```

**ب) هدر استاندارد W3C (`traceparent`):**
```
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
              │   │ trace-id (۱۶ بایت) │ span-id (۸ بایت) │ flags
```

**ج) کی زمان‌بر است؟** با دیدن درخت تریس، متوجه می‌شویم `Orders→DB` ۱۰۰ms از ۱۲۰ms را گرفته → مشکل در کوئری/ایندکس DB.

### مقایسه و دیدگاه‌های مختلف

**Tracing در برابر Logs:**
- تریس: دید کل‌نگر روی یک درخواست در میان سرویس‌ها.
- لاگ: دید جزئی در یک سرویس؛ با Trace ID پیوند می‌خورد.

**Tracing در برابر Metrics:**
- متریک: خلاصهٔ آماری (p99 کل سیستم).
- تریس: نمونهٔ واقعی یک درخواست خاص.

**Head-based در برابر Tail-based Sampling:**
- Head: تصمیم در شروع → ارزان، اما تریس خطا ممکن است نمونه نشود.
- Tail: تصمیم پس از پایان → می‌توانید «همهٔ خطاها + ۵٪ موفق» را نگه دارید (بهتر اما پرهزینه).

**دیدگاه:** «تریس برای زمانی است که متریک می‌گوید مشکل دارید و لاگ نمی‌گوید کجا». ابزار ریشه‌یابی، نه نظارت مداوم.

### اشتباهات رایج

1. **عدم انتقال Context (traceparent):** سرویس‌های بعدی تریس را ادامه نمی‌دهند → تریس شکسته (broken).
2. **نمونه‌برداری ۰٪ یا ۱۰۰٪ افراطی:** ۰٪ = هیچ داده؛ ۱۰۰٪ = هزینهٔ انفجاری.
3. **Span بیش از حد ریز:** هر خط کد span → نویز و هزینه.
4. **فراموشی پیوند لاگ به تریس:** لاگ‌ها بدون Trace ID → نمی‌توان از تریس به لاگ پرش کرد.
5. **عدم استفاده از استاندارد W3C:** هدر سفارشی → ابزارهای دیگر آن را نمی‌فهمند.

### نکات مهم و ترفندها

- **Trace ID را در لاگ قرار دهید** (فصل ۱۰۱) تا از تریس به لاگ پیوند داشته باشید.
- **Exemplars:** پیوند یک متریک histogram به یک تریس نمونه (Prometheus + Tempo پشتیبانی می‌کنند) — قدرت‌مند برای «این نقطهٔ نمودار مربوط به کدام درخواست واقعی است؟»
- **تنظیم Sample rate پویا:** در حالت عادی ۵٪، هنگام حادثه ۱۰۰٪.
- **Service Map:** ابزارها نقشهٔ وابستگی را خودکار می‌سازند — برای درک معماری بی‌نظیر است.
- **Span attributes معنادار:** `http.method`, `db.statement`, `error=true` را الصاق کنید.

### خلاصه فصل

**Distributed Tracing** مسیر کامل یک درخواست در میان سرویس‌ها را با **Trace/Span** ترسیم می‌کند و bottleneck را آشکار می‌سازد. **Context Propagation** (استاندارد W3C `traceparent`) پیوند سرویس‌ها را حفظ می‌کند و **Sampling** هزینه را کنترل می‌کند. اشتباه کلیدی: شکستن تریس با عدم انتقال context و نداشتن پیوند لاگ-تریس. در فصل بعد، OpenTelemetry را به عنوان استاندارد یکپارچه معرفی می‌کنیم.

---

## فصل ۱۰۷: OpenTelemetry (OTel) — استاندارد یکپارچه

### مقدمه

قبلاً برای هر ابزار (Jaeger، Zipkin، Prometheus) باید کد را جور دیگر instrument می‌کردید — **وابستگی به فروشنده (vendor lock-in)**. **OpenTelemetry (OTel)** پروژهٔ CNCF است که یک استاندارد واحد برای تولید متریک/لاگ/تریس (Signals) ارائه می‌دهد، مستقل از مقصد. این فصل اجزا و فلسفهٔ OTel را بررسی می‌کند.

### مفاهیم اساسی

**چرا OTel؟**
- **Vendor-neutral:** کد را یکبار instrument کنید، به هر مقصد (Jaeger، Tempo، Prometheus، Datadog) بفرستید.
- **سه سیگنال در یک:** Traces + Metrics + Logs (پروژهٔ «جامع» OTel).
- **استاندارد صنعتی:** حمایت همهٔ ابرها و ابزارها.

**اجزای OTel:**
- **API:** رابط برنامه‌نویسی برای ایجاد span/متریک (زبان‌های مختلف).
- **SDK:** پیاده‌سازی API + پیکربندی (sampler، exporter).
- **Instrumentation Libraries:** کتابخانه‌های آماده برای فریم‌ورک‌ها (Flask، Express، JDBC) — instrument خودکار.
- **OpenTelemetry Collector:** سرویس مستقل که داده را از منابع می‌گیرد (Receivers)، پردازش می‌کند (Processors) و می‌فرستد (Exporters). زبان‌مستقل (Go binary/container).
- **OTLP (OpenTelemetry Protocol):** پروتکل انتقال استاندارد (gRPC/HTTP) — جایگزین فرمت‌های قدیمی.

**Signalها:**
- **Traces:** همان فصل قبل.
- **Metrics:** OTel متریک را هم به فرمت Prometheus-compatible تولید می‌کند.
- **Logs:** OTel Logs API (در حال تکمیل) — یکپارچگی سه‌گانه.

**Collector Pipelines:**
```
Receivers (OTLP/Jaeger/Prometheus) → Processors (batch/retry) → Exporters (Tempo/Loki/Prometheus)
```

### مثال‌های عملی

**الف) معماری OTel Collector (نمای کلی):**
```
App (OTel SDK) ──OTLP──▶ Collector ──▶ Tempo (traces)
                                ├─────▶ Prometheus (metrics)
                                └─────▶ Loki (logs)
```

**ب) پیکربندی Collector (`otel-collector-config.yaml`):**
```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
processors:
  batch: {}
  memory_limiter:
    check_interval: 1s
    limit_percentage: 80
exporters:
  otlp/jaeger:
    endpoint: jaeger:4317
  prometheus:
    endpoint: 0.0.0.0:8889
  loki:
    endpoint: http://loki:3100/loki/api/v1/push
service:
  pipelines:
    traces:  { receivers: [otlp], processors: [batch], exporters: [otlp/jaeger] }
    metrics: { receivers: [otlp], processors: [batch], exporters: [prometheus] }
    logs:    { receivers: [otlp], processors: [batch], exporters: [loki] }
```

**ج) نصب OTel در K8s (Helm):**
```bash
helm install otel-collector open-telemetry/opentelemetry-collector \
  --set mode=deployment
```

### مقایسه و دیدگاه‌های مختلف

**OTel در برابر OpenTracing/OpenCensus:**
- OpenTracing + OpenCensus ادغام شدند → **OpenTelemetry** (جایگزین هر دو).
- دیگر از OpenTracing قدیم استفاده نکنید؛ OTel جانشین رسمی است.

**OTel در برابر ابزار اختصاصی (Datadog APM):**
- OTel: استاندارد آزاد، چندمقصدی، بدون lock-in.
- Datadog: یکپارچگی آسان‌تر اما قفل‌شده و گران.

**Collector در برابر مستقیم (SDK→Backend):**
- Collector: تجمیع مرکزی، پردازش، تغییر مقصد بدون تغییر کد — توصیهٔ قوی.
- مستقیم: ساده برای شروع، اما تغییر مقصد = تغییر کد.

**دیدگاه:** OTel «HTTP برای تله‌متری» است — استانداردی که همه قبول کرده‌اند. سرمایه‌گذاری روی آن قفل‌شدن آینده را کاهش می‌دهد.

### اشتباهات رایج

1. **نصب چندین SDK متناقض:** مثلاً Jaeger SDK قدیم + OTel همزمان → تداخل.
2. **عدم استفاده از Collector:** ارسال مستقیم از هر سرویس → تغییر مقصد سخت.
3. **Collector بدون resource limits:** در K8s ممکن است حافظه را ببلعد.
4. **فراموشی Memory Limiter Processor:** فشار ترافیک → OOM در Collector.
5. **version ناسازگار OTLP:** کلاینت و سرور نسخهٔ پروتکل متفاوت → خطای انکدر.

### نکات مهم و ترفندها

- **همیشه از OTel Collector استفاده کنید** (حتی برای یک مقصد) — انعطاف آینده.
- **memory_limiter + batch processors** را همیشه در Collector فعال کنید.
- **Auto-instrumentation:** برای زبان‌های پشتیبانی‌شده از SDK خودکار (بدون تغییر کد) شروع کنید.
- **OTLP را استاندارد قرار دهید:** همهٔ ارسال‌ها gRPC:4317.
- **Semantic Conventions:** از نام‌های استاندارد OTel (`http.request.method` نه `http_method`) استفاده کنید.
- **Collector作为一个 Deployment در K8s:** با HPA برای مقیاس‌بندی.

### خلاصه فصل

**OpenTelemetry** استاندارد واحد تله‌متری (traces/metrics/logs) است که وابستگی به فروشنده را از بین می‌برد. اجزای آن **API/SDK/Instrumentation/Collector/OTLP** هستند؛ **Collector** با پایپ‌لاین Receiver→Processor→Exporter داده را به هر مقصد می‌فرستد. اشتباهات کلیدی: چند SDK متناقض و عدم استفاده از Collector. توصیه: همیشه Collector با memory_limiter داشته باشید و OTLP را استاندارد کنید. در فصل بعد instrument عملی را انجام می‌دهیم.

---

## فصل ۱۰۸: Instrumentation عملی با OTel

### مقدمه

حالا عملی می‌کنیم: چگونه کد را با OTel instrument کنیم؟ دو روش **خودکار (auto)** و **دستی (manual)** داریم. این فصل مثال‌های Python/Node/Java، ایجاد span دستی، و پیکربندی Collector را با جزئیات کدنویسی بررسی می‌کند.

### مفاهیم اساسی

**دو روش instrument:**
- **Auto-instrumentation:** بدون تغییر کد — با agent/کتابخانهٔ wrapper فریم‌ورک را hook می‌کند (Flask، Express، Spring). سریع‌ترین راه.
- **Manual instrumentation:** کد خودتان span ایجاد می‌کند برای منطق کسب‌وکار خاص (مثل «پردازش سفارش»). دقیق‌تر.

**مفاهیم کدنویسی:**
- **Tracer:** شیء ایجاد span.
- **Span:** واحد کار؛ `with tracer.start_as_current_span("name") as span:`.
- **Attributes:** برچسب روی span (`span.set_attribute("order.id", 123)`).
- **Events:** رویداد لحظه‌ای در span (`span.add_event("cache miss")`).
- **Span Status/Error:** علامت‌گذاری خطا.
- **Context Propagation:** خودکار در هدر HTTP (W3C) وقتی از کتابخانهٔ OTel HTTP استفاده می‌کنید.

**Collector Exporters:**
- `otlp` → Tempo/Jaeger
- `prometheus` → Prometheus scrape
- `debug` → چاپ در لاگ (برای تست)

### مثال‌های عملی

**الف) Python — Auto-instrumentation (بدون تغییر کد):**
```bash
pip install opentelemetry-distro opentelemetry-exporter-otlp
opentelemetry-bootstrap -a install
OTEL_SERVICE_NAME=orders OTEL_EXPORTER_OTLP_ENDPOINT=http://collector:4317 \
opentelemetry-instrument python app.py
# همین! Flask/requests/sqlalchemy خودکار instrument شدند
```

**ب) Python — Manual span:**
```python
from opentelemetry import trace
tracer = trace.get_tracer("orders")

def process_order(order_id):
    with tracer.start_as_current_span("process_order") as span:
        span.set_attribute("order.id", order_id)
        span.set_attribute("order.amount", 99.9)
        try:
            charge_payment(order_id)
        except Exception as e:
            span.set_status(trace.Status(trace.StatusCode.ERROR))
            span.record_exception(e)
            raise
```

**ج) Node.js — Express auto + manual:**
```javascript
const { NodeTracerProvider } = require('@opentelemetry/sdk-node');
const { OTLPTraceExporter } = require('@opentelemetry/exporter-trace-otlp-grpc');
const provider = new NodeTracerProvider();
provider.addSpanProcessor(new BatchSpanProcessor(new OTLPTraceExporter({ url: 'http://collector:4317' })));
provider.register();

// بعد: هر درخواست Express خودکار span دارد؛ برای منطق خاص:
const span = trace.getTracer('app').startSpan('custom-work');
span.end();
```

**د) Java — auto با agent:**
```bash
java -javaagent:opentelemetry-javaagent.jar \
     -Dotel.service.name=checkout \
     -Dotel.exporter.otlp.endpoint=http://collector:4317 \
     -jar app.jar
```

**ه) تست با debug exporter در Collector:**
```yaml
exporters:
  debug:
    verbosity: detailed
service:
  pipelines:
    traces: { receivers: [otlp], processors: [batch], exporters: [debug] }
# خروجی تریس‌ها را در لاگ Collector می‌بینید
```

### مقایسه و دیدگاه‌های مختلف

**Auto در برابر Manual:**
- Auto: سریع، پوشش فریم‌ورک، اما روی منطق کسب‌وکار کنترل ندارد.
- Manual: دقیق، اما نیاز به کدنویسی و نگهداری.
- **ترکیبی (توصیه):** Auto برای زیربنا + Manual برای نقاط کلیدی کسب‌وکار.

**زبان‌ها:** Python/Java/Node/Go/.NET همگی پشتیبانی کامل دارند؛ زبان‌های دیگر جزئی‌تر.

**دیدگاه:** «با Auto شروع کن، با Manual عمق ببخش». بیشتر تیم‌ها ۸۰٪ را Auto و ۲۰٪ را Manual می‌کنند.

### اشتباهات رایج

1. **فراموشی تنظیم `OTEL_SERVICE_NAME`:** همهٔ سرویس‌ها «unknown» نمایش داده می‌شوند.
2. **عدم ثبت خطا در span:** استثنا throw می‌شود اما span وضعیت ERROR نمی‌گیرد → در UI نمی‌درخشد.
3. **Span attributes بیش از حد:** cardinality انفجاری (مثل user_id) → فشار روی backend.
4. **اختلاف نسخه SDK و Collector:** خطای سازگاری پروتکل.
5. **ارسال مستقیم به backend بدون Collector در K8s:** هر پاد باید پیکربندی مقصد را بداند → تغییر سخت.

### نکات مهم و ترفندها

- **Environment variables استاندارد OTel** (`OTEL_*`): `OTEL_SERVICE_NAME`, `OTEL_EXPORTER_OTLP_ENDPOINT`, `OTEL_TRACES_SAMPLER`.
- **Sampler:** `OTEL_TRACES_SAMPLER=parentbased_traceidratio` با `OTEL_TRACES_SAMPLER_ARG=0.1` (۱۰٪).
- **Resource attributes:** `OTEL_RESOURCE_ATTRIBUTES=service.namespace=shop,deployment.environment=prod` برای گروه‌بندی.
- **auto-instrument در CI:** از `opentelemetry-bootstrap` برای نصب خودکار استفاده کنید.
- **Correlation:** OTel log appender لاگ را با Trace ID پیوند می‌زند (پل به فصل ۱۰۱).
- **Sampling هوشمند:** خطاها را ۱۰۰٪ نگه دارید (tail-based در Collector).

### خلاصه فصل

Instrument با OTel دو مسیر دارد: **Auto** (بدون تغییر کد، سریع) و **Manual** (span دستی برای منطق کلیدی). در Python با `opentelemetry-instrument`، در Java با `-javaagent`، و در Node با SDK آغاز می‌شود. کلیدها: تنظیم `OTEL_SERVICE_NAME`، ثبت خطا در span، و کنترل attributes. اشتباه بحرانی: سرویس بی‌نام و عدم پیوند لاگ-تریس. ترکیب Auto+Manual رویکرد عملی است. در فصل بعد tracing را در میکروسرویس واقعی می‌بینیم.

---

## فصل ۱۰۹: Tracing در معماری میکروسرویس

### مقدمه

چالش واقعی وقتی است که درخواست از ۵ سرویس عبور می‌کند. **Propogation** (اشاعه) صحیح context و **مشاهدهٔ سفر درخواست** تفاوت بین «حدس زدن» و «دانستن» است. این فصل پیوند سرویس‌ها، استاندارد W3C، baggage، و تحلیل bottleneck را در میکروسرویس بررسی می‌کند.

### مفاهیم اساسی

**اشاعهٔ بافتار (Context Propagation):**
- وقتی سرویس A درخواست به B می‌فرستد، باید `traceparent` (و اختیاری `tracestate`) را در هدر HTTP بگذارد.
- کتابخانهٔ OTel HTTP (requests/axios) این را **خودکار** انجام می‌دهد — به شرط استفاده از همان.
- نتیجه: spanهای B به A وصل می‌شوند (درخت یکپارچه).

**Baggage (بار اضافی):**
- انتقال داده‌های سفارشی میان سرویس‌ها (مثل `user.id` یا `tenant`) در هدر `baggage`.
- برای فیلتر/مسیریابی مفید (مثلاً «تریس‌های tenant=X»).

**مشاهدهٔ سفر:**
- ابزار (Tempo/Jaeger) درخت تریس را نمایش می‌دهد — زمان هر سرویس رنگی.
- **Service Map:** نمودار وابستگی خودکار (کدام سرویس به کدام زنگ می‌زند).

**زبان پرس‌وجو:**
- **TraceQL (Tempo):** پرس‌وجو روی ویژگی‌های span — `{ resource.service.name="orders" && span.duration > 500ms }`.
- **Jaeger Query:** جست‌وجو بر اساس service/operation/tag.

**تحلیل Bottleneck:**
- span طولانی‌تر = گلوگاه. مثلاً `payment` ۲s از ۲.۳s کل → مشکل در Payment.

### مثال‌های عملی

**الف) انتقال خودکار context (Python requests + OTel):**
```python
from opentelemetry.instrumentation.requests import RequestsInstrumentor
RequestsInstrumentor().instrument()
import requests
# هدر traceparent خودکار به درخواست اضافه می‌شود:
requests.get("http://payment:8080/charge")  # span در payment به orders وصل می‌شود
```

**ب) انتقال دستی (اگر کتابخانه خودکار نباشد):**
```python
from opentelemetry.propagate import inject
headers = {}
inject(headers)   # پر کردن traceparent در headers
requests.get(url, headers=headers)
# در سمت گیرنده: extract(headers) برای ادامهٔ تریس
```

**ج) TraceQL (Tempo) — یافتن کندها:**
```
{ resource.service.name = "orders" && span.duration > 1s }
# یا خطاها:
{ status = error }
```

**د) نمایش درخت تریس (مفهومی):**
```
Gateway 2ms → Auth 5ms → Orders 300ms
                                └─ DB 250ms  (bottleneck!)
                                └─ Cache 20ms
                             → Payment 1200ms (بسیار کند!)
```

### مقایسه و دیدگاه‌های مختلف

**W3C traceparent در برابر هدر سفارشی:**
- W3C: استاندارد جهانی، همهٔ ابزار می‌فهمند.
- سفارشی: قفل شدن، عدم همکاری ابزارهای دیگر.

**Tempo در برابر Jaeger:**
- Tempo: ذخیرهٔ ارزان (object storage)، یکپارچه با Grafana، TraceQL قدرتمند.
- Jaeger: بالغ، مستقل، اما ذخیره‌سازی سنگین‌تر.

**In-process در برابر Cross-process:**
- درون یک سرویس: spanها خودکار تو در تو می‌شوند.
- میان سرویس‌ها: نیاز به propagation صریح (اگر auto نباشد).

**دیدگاه:** بزرگ‌ترین ارزش tracing در میکروسرویس است — «بدون آن، عیب‌یابی توزیع‌شده شبیه تاریکی است».

### اشتباهات رایج

1. **عدم instrument کردن کلاینت HTTP:** درخواست به سرویس بعدی تریس را ادامه نمی‌دهد → درخت شکسته.
2. **استفاده از پروتکل‌های بدون هدر (مثل gRPC قدیم/کاستوم):** باید propagation را دستی اضافه کرد.
3. **Message queue بدون propagation:** وقتی درخواست از طریق Kafka/RabbitMQ می‌رود، context را در header پیام بگذارید.
4. **بیش‌درختی (over-nesting):** span در span بی‌دلیل → نویز.
5. **عدم نگاه به Service Map:** فرصت درک وابستگی‌ها را از دست می‌دهید.

### نکات مهم و ترفندها

- **همهٔ کلاینت‌های شبکه را instrument کنید** (requests، axios، grpc، kafka) تا زنجیره کامل شود.
- **TraceQL را بیاموزید:** قدرت فیلتر روی span attributes بی‌نظیر است.
- **Exemplars:** متریک histogram را به تریس وصل کنید تا از نمودار به درخواست واقعی بپرید.
- **Service Map** را برای On-call به عنوان نقشهٔ راه قرار دهید.
- **بازنویسی Baggage با احتیاط:** دادهٔ زیاد در baggage → سربار هدر.
- **Tempo + Grafana:** تریس، متریک و لاگ در یک داشبورد (فصل بعد).

### خلاصه فصل

در میکروسرویس، **اشاعهٔ context** (W3C `traceparent`) زنجیرهٔ سرویس‌ها را می‌سازد — مشروط بر اینکه همهٔ کلاینت‌های شبکه (HTTP/gRPC/Kafka) instrument شده باشند. **Baggage** دادهٔ سفارشی منتقل می‌کند و **TraceQL/Service Map** تحلیل bottleneck و وابستگی را ممکن می‌سازد. اشتباه کلیدی: درخواست‌های بین‌سرویسی که تریس را ادامه نمی‌دهند (درخت شکسته). در فصل بعد هر سه ستون را در یک سناریو یکپارچه می‌کنیم.

---

## فصل ۱۱۰: سناریوی عملی — Observability یکپارچه (Trilogy)

### مقدمه

اوج کار: سیستمی بسازیم که **متریک + لاگ + تریس** را یکجا (در Grafana) با پیوند متقابل (exemplars/trace_id) نشان دهد. **OpenTelemetry Collector** داده را از اپلیکیشن می‌گیرد و به **Prometheus (متریک)**، **Tempo (تریس)** و **Loki (لاگ)** می‌فرستد. این فصل نسخهٔ کامل عملیاتی trilogy است.

### مفاهیم اساسی

**معماری یکپارچه (Single Collector → سه مقصد):**
```
App (OTel SDK + structured logs)
   │ OTLP (traces+metrics+logs)
   ▼
OTel Collector
   ├─ traces  ─▶ Tempo ─┐
   ├─ metrics ─▶ Prometheus ─┼─▶ Grafana (یک داشبورد، سه منبع)
   └─ logs    ─▶ Loki  ─┘
```
**پیوندهای متقابل:**
- **Trace → Logs:** در تریس روی یک span، لاگ‌های همان `trace_id` را ببینید.
- **Metrics → Traces (Exemplars):** روی یک نقطهٔ نمودار p99، تریس نمونه را باز کنید.
- **Logs → Traces:** از لاگ روی `trace_id` کلیک کنید → تریس کامل.

این «چرخهٔ کاوش» (investigation loop) قدرت واقعی Observability است: متریک می‌گوید «کجا»، لاگ می‌گوید «چه»، تریس می‌گوید «چرا/کجای مسیر».

### مثال‌های عملی

**الف) OTel Collector — سه پایپ‌لاین (خلاصه):**
```yaml
receivers:
  otlp:
    protocols: { grpc: { endpoint: 0.0.0.0:4317 } }
processors:
  batch: {}
  memory_limiter: { check_interval: 1s, limit_percentage: 75 }
exporters:
  otlp/tempo: { endpoint: tempo:4317 }
  prometheus: { endpoint: 0.0.0.0:8889 }
  loki: { endpoint: http://loki:3100/loki/api/v1/push }
service:
  pipelines:
    traces:  { receivers: [otlp], processors: [memory_limiter, batch], exporters: [otlp/tempo] }
    metrics: { receivers: [otlp], processors: [memory_limiter, batch], exporters: [prometheus] }
    logs:    { receivers: [otlp], processors: [memory_limiter, batch], exporters: [loki] }
```

**ب) اپلیکیشن Python (تریس + لاگ با trace_id):**
```python
from opentelemetry import trace
from opentelemetry.sdk._logs import LoggingHandler
tracer = trace.get_tracer("orders")
# LoggingHandler لاگ را با trace_id به OTel می‌فرستد
import logging
logging.getLogger().addHandler(LoggingHandler())
# حالا لاگ و تریس با هم در Collector می‌رسند
```

**ج) داشبورد Grafana یکپارچه (پنل‌ها):**
```
۱. TimeSeries: p99 latency (Prometheus) + Exemplar→Tempo
۲. TraceQL: { status = error } (Tempo)
۳. Logs: {service="orders"} |= "ERROR" (Loki)
۴. دکمهٔ پیوند: از نمودار → تریس، از تریس → لاگ
```

**د) نصب یکجا در K8s (Helm):**
```bash
helm install tempo grafana/tempo
helm install loki grafana/loki-stack
helm install prometheus prometheus-community/kube-prometheus-stack
helm install otel open-telemetry/opentelemetry-collector
# Grafana همه را به‌عنوان data source می‌بیند
```

### مقایسه و دیدگاه‌های مختلف

**یک Collector در برابر چند Collector:**
- یکی: ساده، نقطهٔ تمرکز؛ اما در مقیاس بزرگ نیاز به sharding.
- چندتا (بر اساس signal): جداسازی فشار، اما پیچیدگی بیشتر.

**Tempo در برابر Jaeger در این معماری:**
- Tempo: ذخیرهٔ ارزان + یکپارچگی Grafana بی‌نظیر (توصیه برای trilogy).
- Jaeger: مستقل، اما هزینهٔ ذخیره بیشتر.

**Trilogy در برابر ابزار تجاری (Datadog):**
- Trilogy (OTel+Prom+Loki+Tempo): رایگان/متن‌باز، کنترل کامل، یادگیری بیشتر.
- Datadog: یکپارچگی فوری، هزینه و lock-in.

**دیدگاه:** «اگر می‌خواهید مهندس Observability واقعی شوید، trilogy متن‌باز را بسازید؛ اگر فقط می‌خواهید کار کند، Datadog».

### اشتباهات رایج

1. **عدم فعال‌سازی Exemplars:** متریک و تریس از هم جدا می‌مانند → پیوند از دست می‌رود.
2. **سه سیستم جدا بدون یک Grafana:** تکه‌تکه بودن → کاوش سخت.
3. **حجم دادهٔ بی‌حد در Tempo/Loki:** نبود retention → هزینهٔ انفجاری.
4. **Collector بدون memory_limiter:** فشار → OOM → همهٔ تله‌متری می‌ریزد.
5. **عدم تست پیوند:** تا حادثه پیش نیاید کسی نمی‌فهمد کلیک «لاگ→تریس» کار نمی‌کند.

### نکات مهم و ترفندها

- **Exemplars** را در Prometheus و Tempo فعال کنید (`exemplars` در scrape config + Tempo exporter).
- **Derived Fields در Grafana:** فیلد `trace_id` در لاگ را به لینک Tempo تبدیل کنید.
- **Alert → runbook → trace:** هشدار پیوندی به داشبورد یکپارچه داشته باشد.
- **Cost Guardrails:** retention کوتاه‌تر برای تریس (مثلاً ۷ روز)، طولانی‌تر برای لاگ.
- **Synthetic Load:** با ابزار (k6/Locust) ترافیک مصنوعی بزنید تا پایپ‌لاین را پیش از تولید تست کنید.
- **On-call drill:** یک بار در ماه تمرین حادثه با trilogy انجام دهید.

### خلاصه فصل

سناریوی یکپارچه نشان داد چگونه **یک OTel Collector** متریک→Prometheus، تریس→Tempo و لاگ→Loki را تغذیه می‌کند و **Grafana** هر سه را با پیوند متقابل (exemplars/trace_id) نمایش می‌دهد — چرخهٔ کاوش کامل. کلیدهای موفقیت: فعال‌سازی Exemplars، یک Grafana واحد، memory_limiter در Collector، و retention مناسب. اشتباه بحرانی: پیوندهای متقابل غیرفعال که کاوش را ناممکن می‌کند. با این فصل، هر سه ستون Observability (Metrics/Logs/Traces) تکمیل شد. در فایل بعدی به SRE و مدیریت حادثه (که از همین داده‌ها تغذیه می‌کند) می‌رویم.

---

## منابع و مراجع

- OpenTelemetry Documentation — https://opentelemetry.io/docs/
  - Collector: https://opentelemetry.io/docs/collector/
  - Instrumentation (Python): https://opentelemetry.io/docs/instrumentation/python/
  - Semantic Conventions: https://opentelemetry.io/docs/specs/semconv/
- W3C Trace Context — https://www.w3.org/TR/trace-context/
- Grafana Tempo — https://grafana.com/docs/tempo/
  - TraceQL: https://grafana.com/docs/tempo/latest/traceql/
- Jaeger — https://www.jaegertracing.io/docs/
- CNCF Observability Whitepaper — https://github.com/cncf/tag-observability
- «Distributed Tracing in Practice» (کتاب O'Reilly)
- Exemplars in Prometheus — https://prometheus.io/docs/practices/exemplars/

**بعدی:** [فایل ۲۳ — SRE و مدیریت حادثه (Incident Management)](23-SRE-and-Incident-Management.md)
