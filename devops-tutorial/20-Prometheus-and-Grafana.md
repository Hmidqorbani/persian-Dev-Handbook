# فایل ۲۰ — Prometheus و Grafana (Observability بخش ۱)

**مدت یادگیری**: ۱۱ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۱۱ (Kubernetes مبانی)، فایل ۱۸ (مفاهیم ابر)، آشنایی با مفاهیم شبکه/HTTP

---

## فهرست محتوا

- [فصل ۹۶: مبانی Observability و فلسفهٔ مانیتورینگ](#فصل-۹۶-مبانی-observability-و-فلسفهٔ-مانیتورینگ)
- [فصل ۹۷: معماری Prometheus و جمع‌آوری داده](#فصل-۹۷-معماری-prometheus-و-جمعآوری-داده)
- [فصل ۹۸: PromQL و هشدارها (Alerting)](#فصل-۹۸-promql-و-هشدارها-alerting)
- [فصل ۹۹: Grafana — داشبورد و تصویرسازی](#فصل-۹۹-grafana--داشبورد-و-تصویرسازی)
- [فصل ۱۰۰: سناریوی عملی — مانیتورینگ کامل یک اپلیکیشن](#فصل-۱۰۰-سناریوی-عملی--مانیتورینگ-کامل-یک-اپلیکیشن)

---

## فصل ۹۶: مبانی Observability و فلسفهٔ مانیتورینگ

### مقدمه

«سیستم کار می‌کند» را چگونه می‌فهمیم؟ وقتی کاربران شکایت می‌کنند دیر شده است. **Observability (مشاهده‌پذیری)** یعنی توانایی پاسخ به هر سوالی دربارهٔ وضعیت سیستم از روی خروجی‌های آن — بدون دانش قبلی از باگ. این فصل تفاوت مانیتورینگ و Observability، سه ستون داده و متدولوژی‌های اندازه‌گیری (RED / USE / Four Golden Signals) را می‌سازد که پایهٔ Prometheus و Grafana است.

### مفاهیم اساسی

**تفاوت Monitoring و Observability:**
- **Monitoring:** جمع‌آوری متریک‌های از پیش تعریف‌شده و هشدار روی آستانه (threshold) — پاسخ به «آیا X خراب است؟»
- **Observability:** توانایی کاوش سیستم برای یافتن مشکل ناشناخته — پاسخ به «چرا کند است؟»

**سه ستون Observability:**
1. **Metrics (متریک):** اعداد عددی بازه‌مند (time series) — مثلاً «تأخیر ۲۰۰ms». ارزان، کلی.
2. **Logs (لاگ):** رویدادهای متنی — مثلاً «خطای ۵۰۰ در درخواست /api». جزئی، حجیم.
3. **Traces (ردیابی):** مسیر یک درخواست در میان سرویس‌ها — برای معماری توزیع‌شده (فایل ۲۲).

**متدولوژی‌های اندازه‌گیری:**
- **RED method** (برای سرویس‌های وب): Rate (نرخ درخواست)، Errors (نرخ خطا)، Duration (تأخیر).
- **USE method** (برای منابع): Utilization، Saturation، Errors — برای CPU/RAM/دیسک.
- **Four Golden Signals** (گوگل SRE): Latency، Traffic، Errors، Saturation.

**مدل‌های جمع‌آوری:**
- **Pull (کشش):** مانیتور سرور به مقصد می‌رود و داده را می‌کشد (Prometheus style) — کنترل مرکزی، مناسب شبکه‌های محدود.
- **Push (رانش):** مقصد داده را به سرور می‌فرستد (StatsD، Graphite) — مناسب محیط‌های ephemeral (مثل Function).

**مفهوم Time Series (سری زمانی):**
هر متریک = (نام متریک) + (برچسب‌های/labelهای بعدی) + (مقدار) + (timestamp). مثال:
```
http_requests_total{method="GET", status="200"}  1234 @ 1700000000
```

### مثال‌های عملی

**الف) تفکیک سه ستون برای یک درخواست:**
```
۱. Metrics:  نرخ ۵۰۰=۰.۳٪ ، p99 latency=۱.۲s   → آیا مشکل داریم؟
۲. Logs:     "ERROR db timeout at query user_orders"  → کجا؟
۳. Traces:   span "checkout" ۹۰۰ms در "db.query"  → چرا کند؟
```

**ب) محاسبه RED برای یک endpoint (مفهومی):**
```
Rate    = requests/sec
Errors  = (5xx responses) / (total responses)
Duration= histogram of response time
```

**ج) فرق Pull/Push (نمایش):**
```
Pull:  Prometheus ──GET /metrics──▶ App:9090/metrics
Push:  App ──POST metric──▶ Pushgateway ──▶ Prometheus
```

### مقایسه و دیدگاه‌های مختلف

**Metrics در برابر Logs:**
- Metrics: فشرده، مناسب هشدار و روند بلندمدت؛ جزئیات ندارد.
- Logs: جزئی، برای debug؛ حجم بالا و گران.
- **ترکیب:** هشدار از روی Metrics، ریشه‌یابی از روی Logs/Traces.

**Pull در برابر Push:**
- Pull (Prometheus): کشف خودکار (service discovery)، مقیاس بهتر، تشخیص targetهای مرده آسان.
- Push (StatsD): برای محیط‌هایی که target قابل کشش نیست (مثل batch job کوتاه).

**دیدگاه مدرن:** «Metrics برای هشدار، Logs برای تحلیل، Traces برای توزیع‌شده». ابزارهای جدید (OpenTelemetry، فایل ۲۲) تلاش می‌کنند هر سه را یکپارچه کنند.

### اشتباهات رایج

1. **فقط Logs (بدون Metrics):** داشبورد ندارید، هشدار ندارید، فقط وقتی خراب شد متوجه می‌شوید.
2. **هشدار روی لاگ به‌جای متریک:** هشدار باید از دادهٔ عددی سریع باشد نه متن.
3. **آستانهٔ ثابت (static threshold) نامناسب:** مثلاً «CPU>۸۰٪» همیشه در ساعت شلوغ هشدار می‌دهد → از روش‌های dynamic/error-rate استفاده کنید.
4. **جمع‌آوری بیش از حد متریک:** cardinality انفجاری (مثل label با user_id) → TSDB کرش می‌کند.
5. **عدم تعریف SLO قبل از مانیتورینگ:** بدون هدف، نمی‌دانید چه چیزی مهم است.

### نکات مهم و ترفندها

- **ابتدا SLO/SLA تعریف کنید** (فایل ۲۳)، سپس متریک‌های مرتبط را جمع‌آوری کنید — نه برعکس.
- **Cardinality را کنترل کنید:** labelها را به مقادیر محدود (status, method) نگه دارید؛ از مقادیر high-cardinality (IP, user_id, request_id) بپرهیزید.
- **روش RED/USE را به عنوان چک‌لیست داشبورد استفاده کنید.**
- **یک داشبورد «Overview» برای مدیر + داشبورد «Detail» برای on-call** طراحی کنید.
- **Alert fatigue را جدی بگیرید:** هشدار فقط برای مواردی که نیاز به عمل (action) دارند.

### خلاصه فصل

Observability توانایی پاسخ به سوالات ناشناخته دربارهٔ سیستم است و بر سه ستون **Metrics/Logs/Traces** استوار است. متدولوژی‌های **RED/USE/Four Golden Signals** چارچوب اندازه‌گیری را می‌دهند. مدل **Pull** (Prometheus) در مقابل **Push** (StatsD) ترجیحات متفاوتی دارد. پیش‌نیاز همه: تعریف SLO و کنترل cardinality. در فصل بعد وارد معماری Prometheus می‌شویم.

---

## فصل ۹۷: معماری Prometheus و جمع‌آوری داده

### مقدمه

**Prometheus** استاندارد صنعتی مانیتورینگ متریک‌محور (مخصوصاً در دنیای Kubernetes) است. طراحی Pull-based دارد: خودش به برنامه‌ها می‌رود و اندپوینت `/metrics` را می‌خواند. این فصل اجزای معماری، مدل داده، انواع متریک و فرآیند scrape را بررسی می‌کند.

### مفاهیم اساسی

**اجزای معماری Prometheus:**
- **Prometheus Server:** هسته — شامل Retrieval (scraper)، TSDB (پایگاه دادهٔ سری‌زمانی)، HTTP API.
- **Exporters:** کتابخانه‌هایی که سیستم‌های بدون `/metrics` را به فرمت Prometheus تبدیل می‌کنند (node_exporter، mysqld_exporter).
- **Pushgateway:** برای کارهای کوتاه (batch) که نمی‌توانند منتظر pull بمانند.
- **Alertmanager:** مدیریت و مسیریابی هشدارها (ایمیل/Slack/PagerDuty).
- **Service Discovery:** کشف خودکار targetها از Kubernetes، Consul، DNS و غیره.
- **Client Libraries:** برای instrument کردن کد (Go/Java/Python/...).

**مدل داده:**
- همه چیز **Time Series** است: `(metric_name{labels} value timestamp)`.
- **Labels** (برچسب) ابعاد را می‌سازند — پایهٔ کوئری قدرتمند.

**انواع متریک (بسیار مهم):**
1. **Counter (شمارنده):** فقط افزایشی — مثلاً تعداد درخواست کل. برای rate مناسب.
2. **Gauge (سنج):** مقدار لحظه‌ای بالا/پایین — مثلاً استفادهٔ حافظه، دما.
3. **Histogram (هیستوگرام):** توزیع مقادیر در bucketها — مثلاً تأخیر درخواست (برای percentile).
4. **Summary:** مشابه Histogram اما محاسبهٔ quantile در سمت کلاینت.

**فرآیند Scrape:**
Prometheus هر `scrape_interval` (مثلاً ۱۵s) به آدرس target می‌رود، متن `/metrics` را می‌خواند و در TSDB ذخیره می‌کند.

### مثال‌های عملی

**الف) فایل پیکربندی `prometheus.yml` (حداقل):**
```yaml
global:
  scrape_interval: 15s
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
```

**ب) اندپوینت `/metrics` ساده (خروجی مثال):**
```
# HELP http_requests_total Total HTTP requests
# TYPE http_requests_total counter
http_requests_total{method="GET",status="200"} 1027
http_requests_total{method="POST",status="500"} 3
# TYPE memory_usage_bytes gauge
memory_usage_bytes 512000
# HELP request_duration_seconds Request latency
# TYPE request_duration_seconds histogram
request_duration_seconds_bucket{le="0.1"} 800
request_duration_seconds_bucket{le="0.5"} 950
request_duration_seconds_bucket{le="+Inf"} 1000
```

**ج) instrument کردن در Python (client library):**
```python
from prometheus_client import Counter, start_http_server
REQUESTS = Counter('http_requests_total', 'Total requests', ['method', 'status'])

start_http_server(8000)   # اندپوینت /metrics روی پورت ۸۰۰۰

# در هر درخواست:
REQUESTS.labels(method='GET', status='200').inc()
```

**د) scrape از Kubernetes (service discovery خلاصه):**
```yaml
- job_name: 'kubernetes-pods'
  kubernetes_sd_configs:
    - role: pod
  relabel_configs:
    - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
      regex: "true"
      action: keep
```

### مقایسه و دیدگاه‌های مختلف

**Prometheus در برابر Graphite/StatsD:**
- Prometheus: Pull-based، مدل دادهٔ لیبل‌دار قدرتمند، PromQL. مناسب K8s.
- Graphite: Push-based، ساده‌تر اما فاقد label dimensions پیشرفته.

**Prometheus در برابر سیستم‌های تجاری (Datadog/New Relic):**
- Prometheus: متن‌باز، رایگان، کنترل کامل، اما نیاز به نگهداری خودتان.
- تجاری: مدیریت‌شده، یکپارچگی آسان، اما هزینه و lock-in.

**Counter در برابر Gauge:**
- Counter: برای نرخ (rate) — همیشه صعودی.
- Gauge: برای وضعیت لحظه‌ای.

**Histogram در برابر Summary:**
- Histogram: bucketها در سمت سرور، انعطاف در کوئری (محاسبه percentile هر زمان) — توصیهٔ عمومی.
- Summary: quantile در سمت کلاینت، کارآمدتر اما غیرقابل تغییر بعداً.

### اشتباهات رایج

1. **استفاده از Gauge برای شمارنده:** مثلاً شمارش درخواست با Gauge → نمی‌توان rate گرفت، ریست سرور داده را از بین می‌برد.
2. **label با cardinality بالا:** `user_id` را به عنوان label قرار دادن → میلیون‌ها سری زمانی → OOM.
3. **فراموشی Pushgateway برای batch:** jobهای کوتاه نمی‌توانند منتظر scrape بمانند.
4. **scrape_interval بسیار کوتاه (۱s):** بار زیاد روی Prometheus و شبکه.
5. **عدم تنظیم retention:** دیسک پر می‌شود؛ TSDB خراب می‌شود.
6. **سرویس‌دهی `/metrics` روی اینترنت:** اندپوینت متریک باید محافظت شود.

### نکات مهم و ترفندها

- **متریک‌های استاندارد را دنبال کنید:** نام‌گذاری `_total` برای counter، واحد را در نام (`_seconds`، `_bytes`) بیاورید.
- **راهنمای exposed:** همیشه HELP و TYPE را بالای متریک بنویسید (ابزارها از آن استفاده می‌کنند).
- **Service Discovery در K8s:** اجازه دهید Prometheus خودش podها را کشف کند (افزونه kube-state-metrics).
- **remote_write:** برای بلندمدت، داده را به Thanos / Cortex / Mimir (ذخیره‌سازی توزیع‌شده) بفرستید.
- **Recoding rules:** کوئری‌های پرهزینه را پیش‌محاسبه (precompute) کنید.
- **/exporter شناخته‌شده:** node_exporter (سیستم)، blackbox_exporter (تست endpoint)، kube-state-metrics (کوبرنتیز).

### خلاصه فصل

Prometheus یک سیستم **Pull-based** است: Server با TSDB داده را از اندپوینت `/metrics` targetها می‌کشد. چهار نوع متریک — **Counter/Gauge/Histogram/Summary** — پایهٔ مدل داده‌اند؛ انتخاب درست آن‌ها (مثلاً Counter برای نرخ) حیاتی است. **Exporters** سیستم‌های قدیمی را متصل، **Pushgateway** jobهای کوتاه را پوشش، و **Service Discovery** در K8s خودکار است. رایج‌ترین اشتباه: label با cardinality بالا که TSDB را می‌کشد. در فصل بعد PromQL و هشدارها را یاد می‌گیریم.

---

## فصل ۹۸: PromQL و هشدارها (Alerting)

### مقدمه

جمع‌آوری داده بی‌فایده است مگر بتوانیم آن را پرس‌وجو و روی آن هشدار بدهیم. **PromQL (Prometheus Query Language)** زبانِ قدرتمندِ پرس‌وجوست و **Alertmanager** مسیریاب هشدارها. این فصل نحوهٔ نوشتن کوئری، محاسبهٔ rate/percentile، و تنظیم هشدارهای معنادار را پوشش می‌دهد.

### مفاهیم اساسی

**عملگرهای پایه PromQL:**
- **انتخاب (Instant Vector):** `http_requests_total` → همهٔ سری‌های فعلی.
- **فیلتر label:** `http_requests_total{status="500"}`.
- ** range vector:** `http_requests_total[5m]` → مقادیر ۵ دقیقهٔ گذشته.
- **rate():** نرخ رشد counter در بازه — `rate(http_requests_total[5m])`.
- **irate():** نرخ لحظه‌ای (حساس‌تر به نوسان).
- **increase():** افزایش مطلق در بازه.
- **histogram_quantile():** محاسبهٔ percentile از histogram — `histogram_quantile(0.99, rate(request_duration_seconds_bucket[5m]))` (p99).
- **sum() by (label):** تجمیع و گروه‌بندی.
- **avg, max, min, count, topk.**

**قواعد هشدار (Alerting Rules):**
- در فایل قواعد تعریف می‌شوند؛ Prometheus به‌طور دوره‌ای ارزیابی می‌کند.
- وقتی عبارت true شد، هشدار به Alertmanager فرستاده می‌شود.
- **for:** مدتی که شرط باید برقرار بماند تا هشدار فعال شود (جلوگیری از نویز لحظه‌ای).

**Alertmanager:**
- مسیریابی (Routing) بر اساس برچسب هشدار.
- گروه‌بندی (Grouping)، سکوت (Silence)، مهار (Inhibition).
- خروجی‌ها: Email، Slack، PagerDuty، Webhook.

### مثال‌های عملی

**الف) کوئری‌های پرکاربرد:**
```promql
# نرخ درخواست کل به صورت در ثانیه
sum(rate(http_requests_total[5m]))

# نرخ خطای ۵xx
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))

# p99 تأخیر
histogram_quantile(0.99, sum(rate(request_duration_seconds_bucket[5m])) by (le))

# استفادهٔ CPU (از node_exporter)
100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

**ب) تعریف قانون هشدار (`rules/alerts.yml`):**
```yaml
groups:
  - name: example
    rules:
      - alert: HighErrorRate
        expr: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) > 0.05
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "نرخ خطا بالای ۵٪ برای ۱۰ دقیقه"
          description: "مقدار فعلی {{ $value }} است."
```

**ج) پیکربندی Alertmanager (مسیریابی به Slack):**
```yaml
route:
  receiver: 'slack-notifications'
  group_by: ['alertname', 'service']
receivers:
  - name: 'slack-notifications'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/XXX'
        channel: '#alerts'
```

**د) بارگذاری قواعد در `prometheus.yml`:**
```yaml
rule_files:
  - "rules/alerts.yml"
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']
```

### مقایسه و دیدگاه‌های مختلف

**rate در برابر irate:**
- `rate`: میانگین در بازه — پایدارتر، برای داشبورد.
- `irate`: بر اساس آخرین دو نمونه — حساس به جهش، برای تشخیص سریع.

**هشدار روی symptom در برابر cause:**
- Symptom (توصیه): «نرخ خطا بالاست» (مشتری تأثیر می‌بیند).
- Cause: «دیسک پر است» — مفید اما نباید جایگزین symptom شود.
- **قانون:** هشدار روی آنچه مشتری حس می‌کند (symptom)، لاگ/تله‌متری برای cause.

**Alertmanager در برابر هشدار در Grafana:**
- Alertmanager: هشدار متمرکز Prometheus، مناسب infrastructure.
- Grafana Alerting: قواعد روی داده‌های چندمنبعی (حتی غیر Prometheus).

### اشتباهات رایج

1. **استفاده از rate روی Gauge:** `rate(memory_usage_bytes[5m])` بی‌معناست — rate فقط برای counter.
2. **هشدار بدون `for`:** نویز لحظه‌ای باعث هشدارهای کاذب می‌شود.
3. **آستانهٔ ثابت سخت:** «CPU>۸۰» در ترافیک نرمال همیشه آتش می‌گیرد؛ از نرخ خطا/SLO استفاده کنید.
4. **بیش از حد هشدار (alert fatigue):** همه چیز را هشدار ندهید؛ فقط action-ableها.
5. **فراموشی Inhibition:** هشدار «سرویس X Down» باید هشدارهای وابسته را سرکوب کند.
6. **عدم تست قواعد:** قبل از تولید با `promtool check rules` اعتبارسنجی کنید.

### نکات مهم و ترفندها

- **promtool:** ابزار اعتبارسنجی — `promtool check rules alerts.yml` و `promtool test rules`.
- **Unit متریک را در نام نگه دارید** تا در کوئری اشتباه نکنید.
- **Recording Rules:** کوئری‌های پیچیده (مثل p99) را پیش‌محاسبه کنید تا داشبورد سریع شود.
- **متغیر `$value` و `$labels`** در annotation برای پیام هوشمند.
- **Silence موقت:** موقع تعمیرات برنامه‌ریزی‌شده هشدار را silence کنید تا نویز ندهد.
- **Multi-tenancy:** با label `tenant` هشدارها را جدا کنید.

### خلاصه فصل

**PromQL** زبان پرس‌وجوی Prometheus است؛ `rate()` برای نرخ counter، `histogram_quantile()` برای percentile (مثل p99)، و `sum() by()` برای تجمیع ابزارهای اصلی‌اند. **Alerting Rules** با فیلد `for` از نویز جلوگیری می‌کنند و **Alertmanager** هشدارها را مسیریابی/گروه‌بندی می‌کند. اشتباه کلیدی: rate گرفتن از Gauge یا هشدار بدون `for`. هشدار باید روی **symptom** (تأثیر مشتری) باشد نه فقط cause. در فصل بعد Grafana را برای تصویرسازی وصل می‌کنیم.

---

## فصل ۹۹: Grafana — داشبورد و تصویرسازی

### مقدمه

Prometheus داده را ذخیره می‌کند؛ **Grafana** آن را به داشبوردهای زیبا و قابل فهم تبدیل می‌کند. Grafana یک لایهٔ visualization چندمنبعی است (نه فقط Prometheus — می‌تواند از Loki، Elasticsearch، CloudWatch هم بخواند). این فصل اتصال، ساخت داشبورد، متغیرها و provisioning را بررسی می‌کند.

### مفاهیم اساسی

**معماری Grafana:**
- **Data Source:** اتصال به Prometheus / Loki / Tempo و غیره.
- **Dashboard:** مجموعه‌ای از Panelها با layout شبکه‌ای.
- **Panel:** هر نمودار/جدول/گیج (gauge). انواع: Time series، Stat، Heatmap، Table، Gauge، Bar chart.
- **Variables (متغیرها):** ورودی‌های پویا (مثل انتخاب `$datacenter`، `$service`) برای فیلتر داشبورد.
- **Folder / Provisioning:** مدیریت داشبورد به‌صورت کد (JSON) به‌جای کلیک دستی.
- **Alerting (Grafana native):** قواعد هشدار روی هر منبع داده.

**انواع نمودار مناسب:**
- **Time series:** برای متریک‌های بازه‌مند (تأخیر، ترافیک).
- **Stat / Gauge:** برای عدد واحد (نرخ خطا، وضعیت).
- **Heatmap:** برای توزیع (مثل histogram تأخیر).
- **Table:** برای لیست (top ۱۰ endpoint کند).

**Provisioning as Code:**
داشبوردها و منابع داده را با فایل YAML/JSON تعریف می‌کنید — قابل بازتولید و نسخه‌کنترل.

### مثال‌های عملی

**الف) تعریف Data Source (Provisioning):**
```yaml
# datasources/prometheus.yml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    url: http://prometheus:9090
    isDefault: true
```

**ب) متغیر داشبورد (برای انتخاب سرویس):**
```yaml
# در تنظیمات داشبورد → Variables
Name: service
Query: label_values(http_requests_total, service)   # لیست سرویس‌ها از Prometheus
```

**ج) یک Panel (Time series) با PromQL:**
```
Panel Title: "Request Rate by Status"
Query: sum(rate(http_requests_total[5m])) by (status)
Legend: {{status}}
```

**د) Provisioning داشبورد (فایل JSON یا مرجع):**
```yaml
# dashboards/dashboards.yml
apiVersion: 1
providers:
  - name: 'Default'
    folder: ''
    options:
      path: /var/lib/grafana/dashboards
```
فایل JSON داشبورد در آن مسیر قرار می‌گیرد و بدون کلیک بارگذاری می‌شود.

### مقایسه و دیدگاه‌های مختلف

**Grafana در برابر Prometheus UI:**
- Prometheus UI: کوئری سریع و تست، بدون زیبایی/داشبورد پیچیده.
- Grafana: تصویرسازی حرفه‌ای، اشتراک‌گذاری، متغیر پویا.

**داشبورد دستی در برابر Provisioning:**
- دستی: سریع برای یادگیری، اما با restart/فقدان نسخه از دست می‌رود.
- Provisioning (توصیه): بخشی از GitOps، بازتولیدپذیر.

**Grafana در برابر ابزار تجاری (Datadog dashboard):**
- Grafana: رایگان/متن‌باز، چندمنبعی (حتی دادهٔ رقیب را نشان می‌دهد).
- Datadog: یکپارچه اما قفل‌شده و گران.

### اشتباهات رایج

1. **داشبورد شلوغ (spaghetti):** ۵۰ پنل در یک صفحه → کسی نمی‌خواند. خلاصه + جزئی جدا کنید.
2. **واحد (Unit) نادرست در Panel:** نمایش latency به صورت «بایت» به‌جای «ثانیه» → اشتباه تفسیر.
3. **اکسپورت نکردن داشبورد به کد:** داشبورد در Grafana می‌ماند و با حذف Pod از بین می‌رود.
4. **عدم استفاده از متغیر:** کپی کردن داشبورد برای هر سرویس به‌جای یک داشبورد با `$service`.
5. **هشدار در Grafana بدون Alertmanager:** برای infrastructure بهتر است هشدار متمرکز باشد.

### نکات مهم و ترفندها

- **Dashboard as Code:** از Grafana provisioning یا ابزار **Grafana Operator** (در K8s) استفاده کنید.
- **JSONnet / Grizzly:** برای تولید داشبوردهای پیچیده به‌صورت برنامه‌نویسی.
- **Templating:** متغیرهای `$__interval` و `$datasource` برای داشبوردهای reusable.
- **Annotations:** رویدادهایی مثل «deploy آخر» را روی نمودار نشان دهید تا همبستگی ببینید.
- **Community Dashboards:** Grafana.com/dashboards هزاران داشبورد آماده (مثل node_exporter full) دارد — از آن‌ها شروع کنید.
- **Row/Collapsible:** داشبورد را با Row سازماندهی کنید تا خوانا بماند.

### خلاصه فصل

**Grafana** لایهٔ visualization چندمنبعی است که Prometheus (و Loki/Tempo و غیره) را به داشبورد تبدیل می‌کند. **Panelها** (Time series/Stat/Heatmap)، **متغیرها** (برای فیلتر پویا مثل `$service`) و **Provisioning as Code** اجزای کلیدی‌اند. اشتباه رایج: داشبورد شلوغ و عدم اکسپورت به کد. توصیه: همیشه داشبورد را provisioning کنید و از داشبوردهای آمادهٔ جامعه شروع کنید. در فصل بعد یک سناریوی end-to-end می‌سازیم.

---

## فصل ۱۰۰: سناریوی عملی — مانیتورینگ کامل یک اپلیکیشن

### مقدمه

تا اینجا قطعات را جدا یاد گرفتیم. حالا آن‌ها را در یک سناریوی واقعی ترکیب می‌کنیم: یک اپلیکیشن وب (Python Flask) با Prometheus + Grafana + Alertmanager + exporters، از instrument تا داشبورد و هشدار. این فصل «چگونه در عمل» است.

### مفاهیم اساسی

**معماری سناریو:**
```
Flask App (instrumented) ──/metrics──▶ Prometheus ──▶ Grafana (dashboard)
                                          │
                                          └──▶ Alertmanager ──▶ Slack
node_exporter (روی هاست) ──▶ Prometheus
```
ما: (۱) اپلیکیشن را instrument می‌کنیم، (۲) Prometheus را تنظیم، (۳) داشبورد Grafana می‌سازیم، (۴) هشدار روی نرخ خطا می‌دهیم، (۵) مشکل را شبیه‌سازی و حل می‌کنیم.

**اکوسیستم Exporter:**
- `node_exporter`: متریک‌های OS (CPU/RAM/دیسک/شبکه).
- `blackbox_exporter`: تست در دسترس بودن URL/پورت از بیرون.
- `kube-state-metrics` + `cadvisor`: متریک‌های K8s (فایل ۱۱).

### مثال‌های عملی

**الف) اپلیکیشن Flask instrument شده:**
```python
from flask import Flask
from prometheus_client import Counter, Histogram, generate_latest, CONTENT_TYPE_LATEST
import time

app = Flask(__name__)
REQUESTS = Counter('http_requests_total', 'Total', ['method', 'endpoint', 'status'])
LATENCY = Histogram('http_request_duration_seconds', 'Latency', ['endpoint'])

@app.route('/api/orders')
def orders():
    with LATENCY.labels('/api/orders').time():
        status = '200'
        # ... منطق ...
        REQUESTS.labels('GET', '/api/orders', status).inc()
        return {'ok': True}, 200

@app.route('/metrics')
def metrics():
    return generate_latest(), 200, {'Content-Type': CONTENT_TYPE_LATEST}
```

**ب) `prometheus.yml` کامل:**
```yaml
global:
  scrape_interval: 15s
scrape_configs:
  - job_name: 'flask-app'
    static_configs:
      - targets: ['flask-app:8000']
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
```

**ج) هشدار نرخ خطا:**
```yaml
groups:
  - name: app
    rules:
      - alert: High5xxErrorRate
        expr: sum(rate(http_requests_total{status="500"}[5m])) / sum(rate(http_requests_total[5m])) > 0.02
        for: 5m
        labels: { severity: page }
        annotations:
          summary: "نرخ ۵xx بالای ۲٪"
```

**د) داشبورد Grafana (پنل‌های پیشنهادی):**
```
۱. Stat:     نرخ درخواست کل (sum rate)
۲. TimeSeries: نرخ ۵xx (rate ... {status="500"})
۳. TimeSeries: p99 latency (histogram_quantile(0.99, ...))
۴. TimeSeries: CPU استفاده (node_exporter)
۵. Heatmap:  توزیع latency
```

**ه) اجرای محلی با Docker Compose (نمای کلی):**
```yaml
services:
  flask-app:    { build: ., ports: ["8000:8000"] }
  prometheus:   { image: prom/prometheus, volumes: [./prometheus.yml:/etc/prometheus/prometheus.yml] }
  grafana:      { image: grafana/grafana, ports: ["3000:3000"] }
  alertmanager: { image: prom/alertmanager }
  node-exporter:{ image: prom/node-exporter }
```

### مقایسه و دیدگاه‌های مختلف

**محلی در برابر Managed (AMP/Mimir):**
- محلی: یادگیری آسان، کنترل کامل، اما نگهداری و دوام (durability) خودتان.
- **Amazon Managed Prometheus (AMP)** / **Grafana Cloud** / **Mimir**: ذخیرهٔ بلندمدت و مقیاس بالا، بدون نگهداری.

**یک داشبورد بزرگ در برابر چند داشبورد:**
- **توصیه:** یک Overview (برای همه) + داشبوردهای تخصصی سرویس (برای on-call).

**دیدگاه SRE:** «اگر هشدار نمی‌دهد، مانیتورینگ ندارید؛ اگر هشدار بی‌ربط می‌دهد، on-call نمی‌خواند».

### اشتباهات رایج

1. **فقط متریک اپلیکیشن، بدون node_exporter:** CPU هاست پر شده اما در داشبورد نیست.
2. **هشدار روی متریکی که وجود ندارد:** اشتباه تایپ نام متریک در expr → هشدار هرگز آتش نمی‌گیرد.
3. **باز گذاشتن `/metrics` روی اینترنت:** نشت اطلاعات عملیاتی.
4. **عدم تست هشدار:** قبل از تولید با خطای عمدی مطمئن شوید Slack پیام می‌گیرد.
5. **TSDB روی دیسک موقت (ephemeral):** ری‌استارت Prometheus همهٔ تاریخ را پاک می‌کند — volume پایدار وصل کنید.

### نکات مهم و ترفندها

- **همیشه volume پایدار برای Prometheus** (`/prometheus`) در K8s (PVC) یا Docker mount کنید.
- **ابزار `promtool`** را برای تست قواعد در CI اجرا کنید.
- **Blackbox exporter** را برای بررسی در دسترس بودن از دید کاربر اضافه کنید.
- **Grafana OnCall / Incident** برای مدیریت حادثه (فایل ۲۳) یکپارچه است.
- **ابزار kube-prometheus-stack:** در K8s، این Helm chart همه (Prometheus+Grafana+Alertmanager+exporters) را یکجا نصب می‌کند — از آن استفاده کنید.
- **SLO-based alerting:** هشدار را بر اساس نرخ خطای مجاز SLO بگذارید (فایل ۲۳).

### خلاصه فصل

سناریوی end-to-end نشان داد چگونه Flask instrument شده + Prometheus (scrape) + Grafana (داشبورد) + Alertmanager (هشدار Slack) یک سیستم مانیتورینگ کامل می‌سازند. کلیدها: instrument کردن با client library، پیوستن exporterها (node_exporter)، نوشتن هشدار روی نرخ خطا (نه آستانهٔ سخت)، و provisioning داشبورد. اشتباهات بحرانی شامل نبود volume پایدار و عدم تست هشدارند. در محیط K8s، **kube-prometheus-stack** کار را خلاصه می‌کند. با این فایل بخش اول Observability (متریک‌ها) کامل شد؛ فایل بعدی به Logs (Loki/ELK) می‌پردازد.

---

## منابع و مراجع

- مستندات Prometheus — https://prometheus.io/docs/
  - Querying (PromQL): https://prometheus.io/docs/prometheus/latest/querying/basics/
  - Alerting: https://prometheus.io/docs/alerting/latest/overview/
  - Exporters: https://prometheus.io/docs/instrumenting/exporters/
- Grafana Labs Docs — https://grafana.com/docs/
  - Provisioning: https://grafana.com/docs/grafana/latest/administration/provisioning/
  - Dashboards: https://grafana.com/docs/grafana/latest/dashboards/
- prometheus/client_python — https://github.com/prometheus/client_python
- «Prometheus: Up & Running» (کتاب O'Reilly)
- Google SRE Book — Monitoring distributed systems: https://sre.google/sre-book/monitoring-distributed-systems/
- kube-prometheus-stack (Helm) — https://github.com/prometheus-community/helm-charts
- Grafana Community Dashboards — https://grafana.com/grafana/dashboards/

**بعدی:** [فایل ۲۱ — لاگینگ، Loki و ELK (Observability بخش ۲)](21-Logging-Loki-and-ELK.md)
