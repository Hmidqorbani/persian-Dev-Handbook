# فایل ۲۱ — لاگینگ، Loki و ELK (Observability بخش ۲)

**مدت یادگیری**: ۱۰ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۲۰ (Prometheus/Grafana)، فایل ۱۱ (Kubernetes مبانی)، فایل ۱۸ (مفاهیم ابر)

---

## فهرست محتوا

- [فصل ۱۰۱: مبانی Logging و مدیریت لاگ](#فصل-۱۰۱-مبانی-logging-و-مدیریت-لاگ)
- [فصل ۱۰۲: ELK Stack (Elasticsearch، Logstash، Kibana)](#فصل-۱۰۲-elk-stack-elasticsearch-logstash-kibana)
- [فصل ۱۰۳: Grafana Loki و LogQL](#فصل-۱۰۳-grafana-loki-و-logql)
- [فصل ۱۰۴: جمع‌آوری و ارسال لاگ در عمل (Agents)](#فصل-۱۰۴-جمعآوری-و-ارسال-لاگ-در-عمل-agents)
- [فصل ۱۰۵: سناریوی عملی — متمرکزسازی لاگ‌ها](#فصل-۱۰۵-سناریوی-عملی--متمرکزسازی-لاگها)

---

## فصل ۱۰۱: مبانی Logging و مدیریت لاگ

### مقدمه

وقتی متریک می‌گوید «سیستم کند است»، لاگ می‌گوید «چرا». **Logging** ثبت رویدادهای متنی سیستم است — ستون دوم Observability (فصل ۹۶). اما لاگ‌های پراکنده روی صدها سرور قابل استفاده نیستند؛ نیاز به **متمرکزسازی (Centralized Logging)** داریم. این فصل اصول لاگ‌نویسی، سطوح، فرمت و چرخهٔ حیات لاگ را می‌سازد.

### مفاهیم اساسی

**چرا لاگ متمرکز؟**
- در محیط کانتینر/K8s، لاگ روی دیسک ناپایدارِ پاد حذف می‌شود (ephemeral).
- ده‌ها سرور = ده‌ها محل دیدن لاگ → غیرممکن بدون تجمیع.
- برای تحلیل الگو، جست‌وجوی متقاطع (cross-service) و نگهداری بلندمدت.

**سطوح لاگ (Log Levels):**
- **DEBUG:** جزئیات توسعه (فقط هنگام debug).
- **INFO:** رویدادهای عادی (startup، درخواست پردازش شد).
- **WARN:** مشکل احتمالی اما سیستم کار می‌کند.
- **ERROR:** شکست عملیات (مثلاً خطای DB).
- **FATAL/CRITICAL:** شکست سیستم (باید صفحه‌کوب شود).

**لاگ ساختاریافته (Structured) در برابر متن ساده:**
- **Unstructured:** `"User 42 logged in"` — انسان‌خوانا، ماشین سخت‌پرداز.
- **Structured (JSON):** `{"ts":"2026-...","level":"INFO","user_id":42,"event":"login","ip":"1.2.3.4"}` — قابل فیلتر/جست‌وجو در ابزار.

**چرخهٔ حیات لاگ:**
```
تولید (app) → جمع‌آوری (agent) → انتقال (shipping) →
ذخیره (store/index) → جست‌وجو/تحلیل (search) → نگهداری/حذف (retention)
```

**مفاهیم کلیدی:**
- **Log Rotation:** چرخش فایل لاگ بر اساس اندازه/زمان (جلوگیری از پر شدن دیسک).
- **Sampling:** نمونه‌برداری از لاگ‌های پرحجم برای کاهش هزینه.
- **Context/Trace ID:** شناسهٔ یکتا برای پیوند لاگ‌های یک درخواست (پل به Tracing، فایل ۲۲).
- **PII Redaction:** حذف داده‌های حساس (رمز/کارت) از لاگ.

### مثال‌های عملی

**الف) لاگ ساختاریافته (Python logging با JSON):**
```python
import logging, json, sys
class JsonFormatter(logging.Formatter):
    def format(self, rec):
        return json.dumps({
            "ts": self.formatTime(rec),
            "level": rec.levelname,
            "service": "orders",
            "msg": rec.getMessage(),
            "trace_id": getattr(rec, "trace_id", None)
        })
logger = logging.getLogger()
handler = logging.StreamHandler(sys.stdout)
handler.setFormatter(JsonFormatter())
logger.addHandler(handler)
logger.setLevel(logging.INFO)
logger.info("order created", extra={"trace_id": "abc123"})
```

**ب) خروجی (یک خط JSON در stdout):**
```
{"ts":"2026-10-17 12:00:00","level":"INFO","service":"orders","msg":"order created","trace_id":"abc123"}
```

**ج) سطوح مناسب (مثال اشتباه/درست):**
```
اشتباه: logger.error("user logged in")   # سطح اشتباه
درست:   logger.info("user logged in")    # INFO کافی است
```

### مقایسه و دیدگاه‌های مختلف

**Structured در برابر Unstructured:**
- Structured: هزینهٔ کمی در تولید اما قدرت تحلیل بالا — استاندارد مدرن.
- Unstructured: راحت برای چشم، غیرقابل جست‌وجوی ماشینی در مقیاس.

**لاگ در برابر متریک (تکرار از فصل ۹۶):**
- لاگ: «چه اتفاقی افتاد» (جزئی، پرهزینه).
- متریک: «چند بار افتاد» (فشرده، ارزان).
- نگهدارید لاگ را هدفمند — لاگ نویسی بی‌رویه هزینه و نویز ایجاد می‌کند.

**دیدگاه:** «لاگ برای انسان (debug)، متریک برای ماشین (هشدار)». برخی تیم‌ها لاگ را فقط برای خطا نگه می‌دارند و بقیه را با متریک پوشش می‌دهند (cost-effective).

### اشتباهات رایج

1. **لاگ‌نویسی در سطح DEBUG در تولید:** حجم عظیم و ریزش عملکرد.
2. **لاگ غیرساختاریافته:** `print("user=" + id)` → جست‌وجوی `user_id=42` غیرممکن.
3. **نوشتن رمز/دادهٔ حساس در لاگ:** نقض امنیت و قوانین (GDPR).
4. **فقدان Trace ID:** ردیابی یک درخواست در میان ۱۰ سرویس غیرممکن.
5. **عدم تنظیم Retention:** لاگ برای همیشه نگه داشته می‌شود → هزینهٔ انباره.
6. **لاگ روی دیسک محلی در K8s:** با restart پاد حذف می‌شود.

### نکات مهم و ترفندها

- **Structured Logging را از روز اول عادت کنید** — تغییر بعدی سخت است.
- **استاندارد OpenTelemetry Logs (فایل ۲۲):** برای یکپارچگی متریک/لاگ/تریس.
- **Log Level پیکربندی‌پذیر:** در تولید INFO، در debug موقت DEBUG.
- **Redact قبل از ارسال:** لایهٔ agent (Fluent Bit) فیلدهای حساس را حذف کند.
- **یک schema مشترک:** `ts, level, service, trace_id, msg, ...` در همهٔ سرویس‌ها.
- **هشدار روی الگوی لاگ:** مثلاً «۵ خطا در ۱ دقیقه» (error rate از لاگ).

### خلاصه فصل

لاگ ستون دوم Observability است و برای محیط‌های توزیع‌شده باید **متمرکز** شود. **لاگ ساختاریافته (JSON)** با سطوح استاندارد (DEBUG→CRITICAL) و **Trace ID** پیونددهنده به Tracing است. چرخهٔ حیات شامل تولید/جمع‌آوری/ذخیره/جست‌وجو/retention می‌شود. اشتباهات کلیدی: لاگ حساس، بدون Trace ID، و نگهداری بی‌مرز. در فصول بعد دو سیستم اصلی ذخیره/جست‌وجو — ELK و Loki — را بررسی می‌کنیم.

---

## فصل ۱۰۲: ELK Stack (Elasticsearch، Logstash، Kibana)

### مقدمه

**ELK** قدیمی‌ترین و قدرتمندترین پشتهٔ متن‌باز لاگ است: **E**lasticsearch (ذخیره/جست‌وجو)، **L**ogstash (پردازش)، **K**ibana (تصویرسازی). همراه با **Beats** (جمع‌آوری)، پشتهٔ کاملی می‌سازد. این فصل معماری و نحوهٔ کار آن را بررسی می‌کند.

### مفاهیم اساسی

**معماری ELK:**
```
App → Filebeat (Beats) → Logstash (فیلتر/پارس) → Elasticsearch (index) → Kibana (جست‌وجو/داشبورد)
```
- **Beats:** عوامل سبک جمع‌آوری (Filebeat برای لاگ، Metricbeat برای متریک).
- **Logstash:** موتور پردازش با pipeline سه مرحله‌ای: Input → Filter (grok/parse) → Output.
- **Elasticsearch:** پایگاه دادهٔ جست‌وجومحور توزیع‌شده؛ داده در **Index** (مثل جدول) و **Document** (مثل ردیف) ذخیره می‌شود. قدرت در full-text search و aggregation.
- **Kibana:** رابط وب برای جست‌وجو (Query DSL)، داشبورد، Discover، alerting.

**شاخص‌گذاری (Indexing):** Elasticsearch هر فیلد را index می‌کند → جست‌وجوی سریع اما **هزینهٔ ذخیره بالا** (لاگ‌های متنی حجیم گران می‌شوند).

**Grok Pattern:** زبان استخراج فیلد از متن (مثل تبدیل لاگ nginx به فیلدهای structured).

**تکامل نام:** امروز «Elastic Stack» شامل همه است؛ نسخهٔ مدیریت‌شده (Elastic Cloud / AWS OpenSearch) هم موجود است (OpenSearch فورک متن‌باز پس از تغییر لایسنس Elastic).

### مثال‌های عملی

**الف) فایل پیکربندی Logstash (`logstash.conf`):**
```
input {
  beats { port => 5044 }
}
filter {
  grok {
    match => { "message" => "%{TIMESTAMP_ISO8601:ts} %{LOGLEVEL:level} %{GREEDYDATA:msg}" }
  }
  date { match => [ "ts", "ISO8601" ] }
}
output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
}
```

**ب) Filebeat (`filebeat.yml`):**
```yaml
filebeat.inputs:
  - type: log
    paths:
      - /var/log/app/*.log
output.logstash:
  hosts: ["logstash:5044"]
```

**ج) جست‌وجو در Kibana (KQL):**
```
level: ERROR and service: orders
status: 500 and url: "/api/checkout"
```

**د) ایجاد یک Index Pattern و داشبورد:** در Kibana → Stack Management → Index Patterns → `logs-*` سپس در Discover فیلتر کنید.

### مقایسه و دیدگاه‌های مختلف

**ELK در برابر Loki (پیش‌نمایش فصل بعد):**
- ELK: index کامل متن → جست‌وجوی قدرتمند، اما هزینه/منابع بالا.
- Loki: index فقط روی برچسب (label) → ارزان‌تر، اما جست‌وجوی متنی کُندتر/محدودتر.

**Logstash در برابر Beats:**
- Logstash: پردازش سنگین/پیچیده (grok، enrichment) — منبق‌خوار.
- Beats: سبک، فقط جمع‌آوری/ارسال ساده.

**Managed در برابر Self-hosted:**
- Elastic Cloud / AWS OpenSearch: نگهداری کم، هزینه بیشتر.
- Self-hosted: کنترل، اما نیاز به خوشه‌سازی Elasticsearch (سخت).

**دیدگاه:** ELK برای تیم‌هایی که جست‌وجوی پیشرفته و SIEM (امنیت) می‌خواهند عالی است؛ برای لاگ ساده/هزینه‌حساس، Loki سبک‌تر است.

### اشتباهات رایج

1. **عدم تنظیم ILM (Index Lifecycle Management):** indexها رشد بی‌مرز → دیسک پر → خوشه کرش.
2. **ارسال لاگ خام بدون پارس:** جست‌وجو فقط روی فیلد `message` → کُند و بی‌فایده.
3. **Grokk پیچیده نادرست:** الگوی غلط باعث `_grokparsefailure` و از دست رفتن داده.
4. **نادیده گرفتن Shard بیش‌ازحد:** هزاران shard کوچک → فشار روی کلاستر.
5. **عدم امنیت‌سازی (X-Pack/TLS):** Elasticsearch بدون احراز در دسترس عمومی → نشت داده.

### نکات مهم و ترفندها

- **ILM را فعال کنید:** Hot→Warm→Cold→Delete برای کنترل هزینه و دیسک.
- **ECS (Elastic Common Schema):** فرمت استاندارد فیلدها را دنبال کنید تا همهٔ منابع هم‌زبان شوند.
- **Ingest Node Pipelines:** پارس را به جای Logstash روی Elasticsearch ببرید (سبک‌تر).
- **Alerting در Kibana:** روی الگوی خطا هشدار بسازید.
- **OpenSearch:** اگر لایسنس Elastic برایتان مشکل است، OpenSearch (AWS) جایگزین سازگار است.
- **Synthetic monitoring:** Kibana Heartbeat/Uptime برای بررسی در دسترس بودن.

### خلاصه فصل

**ELK/Elastic Stack** پشتهٔ بالغ لاگ با Elasticsearch (جست‌وجوی قدرتمند full-text)، Logstash (پردازش/گروک)، Kibana (تصویرسازی) و Beats (جمع‌آوری) است. نقطهٔ قوتش جست‌وجوی پیشرفته و SIEM است؛ نقطهٔ ضعفش هزینهٔ ذخیره‌سازی (index کامل). کلید موفقیت: **ILM** برای چرخهٔ حیات و **grok/ECS** برای پارس استاندارد. اشتباه بحرانی: عدم مدیریت lifecycle که کلاستر را از کار می‌اندازد. در فصل بعد Loki (رویکرد ارزان‌تر) را می‌بینیم.

---

## فصل ۱۰۳: Grafana Loki و LogQL

### مقدمه

**Loki** محصول Grafana Labs است که با شعار «مثل Prometheus برای لاگ‌ها» طراحی شده. برخلاف ELK که متن را کاملاً index می‌کند، Loki فقط روی **برچسب‌ها (labels)** index می‌کند و خود متن را فشرده نگه می‌دارد → بسیار **ارزان‌تر**. این فصل معماری Loki و زبان **LogQL** را بررسی می‌کند.

### مفاهیم اساسی

**فلسفه Loki — «Label-based»:**
- شبیه Prometheus: لاگ‌ها با **labels** (مثل `{app="orders", env="prod"}`) گروه‌بندی می‌شوند.
- متن لاگ index نمی‌شود؛ فقط برای جست‌وجو در صورت فیلتر شدن با label خوانده می‌شود.
- نتیجه: ذخیره‌سازی ارزان (تا ۱۰–۵۰ برابر کمتر از ELK)، اما جست‌وجوی متنی آزاد کُندتر.

**معماری:**
```
App → Promtail/Fluent Bit (جمع‌آوری + الصاق label) → Loki (ذخیره) → Grafana (جست‌وجو/داشبورد)
```
- **Promtail:** agent رسمی Loki (مثل Filebeat) — لاگ را می‌خواند، label می‌زند، می‌فرستد.
- **Loki:** اجزا: Distributor → Ingester → Store (S3/محلی) → Querier.
- **Object Storage:** Loki متن لاگ را در S3/GCS/MinIO نگه می‌دارد → ارزان و مقیاس‌پذیر.

**LogQL (زبان پرس‌وجو):**
- انتخاب: `{app="orders"}`.
- فیلتر خط: `{app="orders"} |= "error"` (شامل)، `!=` (فاقد)، `|~` (regex).
- استخراج/محاسبه: `|= "status=500" | json | status="500"`.
- تجمیع: `count_over_time({app="orders"}[5m])` — مشابه PromQL.

**یکپارچگی با Grafana:** لاگ و متریک در یک داشبورد (همان Grafana فصل ۹۹) — قدرت بزرگ.

### مثال‌های عملی

**الف) پیکربندی Promtail (`promtail.yml` خلاصه):**
```yaml
scrape_configs:
  - job_name: app
    static_configs:
      - targets: [localhost]
        labels:
          job: app
          app: orders
          __path__: /var/log/app/*.log
```

**ب) کوئری‌های LogQL:**
```logql
# همهٔ لاگ‌های سرویس orders
{app="orders"}

# خطاهای حاوی "timeout"
{app="orders"} |= "timeout" | json

# تعداد خطا در ۵ دقیقه
count_over_time({app="orders"} |= "ERROR" [5m])

# استخراج فیلد status و شمارش ۵۰۰
{app="orders"} | json | status="500" | count_over_time([5m])

# نرخ خطا مشابه متریک
sum(count_over_time({app="orders"} |= "ERROR" [1m]))
```

**ج) داشبورد ترکیبی Grafana:** پنل متریک (Prometheus) کنار پنل لاگ (Loki) با متغیر `$trace_id` — کلیک روی نمودار خطا → لاگ‌های مربوطه.

### مقایسه و دیدگاه‌های مختلف

**Loki در برابر ELK:**

| معیار | Loki | ELK |
|---|---|---|
| مدل index | فقط labels | full-text |
| هزینهٔ ذخیره | بسیار کم | بالا |
| جست‌وجوی متن آزاد | محدود/کُند | قدرتمند/سریع |
| یکپارچگی با Grafana | بومی | نیاز به data source |
| مناسب برای | لاگ حجیم/هزینه‌حساس | جست‌وجوی پیشرفته/SIEM |

**Promtail در برابر Fluent Bit:** هر دو کار می‌کنند؛ Fluent Bit سبک‌تر و چندمقصدی (به Loki و ELK همزمان).

**دیدگاه:** بسیاری تیم‌ها که از قبل Prometheus/Grafana دارند، Loki را ترجیح می‌دهند (یک پنجرهٔ واحد، هزینه کم). تیم‌های امنیتی/SIEM هنوز ELK/OpenSearch را می‌خواهند.

### اشتباهات رایج

1. **بیش از حد label (high-cardinality):** label مثل `user_id` → میلیون‌ها جریان (stream) → Loki کرش. فقط labelهای کم‌تعداد (app/env/host).
2. **انتظار جست‌وجوی متنی سریع مثل ELK:** Loki برای آن طراحی نشده؛ فیلتر با label شروع کنید.
3. **عدم استفاده از object storage:** نگه داشتن در فایل محلی → از دست رفتن داده با restart.
4. **فشرده‌سازی نادرست:** تنظیمات chunk/compression را برای بهینه‌سازی چک کنید.
5. **Promtail نخواندن مسیر درست:** `__path__` اشتباه → لاگ نمی‌آید.

### نکات مهم و ترفندها

- **Label کم، مقدار کم:** فقط `app, env, host, level` — نه مقادیر پویا.
- **Object Storage پایدار:** Loki را به S3/MinIO وصل کنید (در K8s با Helm values).
- **LogQL خط‌لوله (pipeline):** `| json | status="500"` برای فیلتر پس از parse.
- **همبستگی متریک-لاگ:** در Grafana با متغیر `$trace_id` لاگ و متریک را پیوند دهید.
- **Loki Canary / Promtail mixin:** مانیتورینگ خود Loki.
- **Grafana Alloy:** نسل بعدی agent (جایگزین Promtail) که OpenTelemetry را هم پشتیبانی می‌کند.

### خلاصه فصل

**Loki** رویکرد «ارزان» به لاگ است: فقط روی **labels** index می‌کند و متن را فشرده در object storage نگه می‌دارد — هزینهٔ ذخیره بسیار پایین. **LogQL** شبیه PromQL اجازهٔ فیلتر/تجمیع می‌دهد و یکپارچگی با **Grafana** (در کنار Prometheus) یک پنجرهٔ واحد می‌سازد. اشتباه کلیدی: label با cardinality بالا که Loki را از کار می‌اندازد. برای جست‌وجوی متنی سنگین/SIEM، ELK هنوز برتر است. در فصل بعد agents جمع‌آوری را عمیق‌تر می‌بینیم.

---

## فصل ۱۰۴: جمع‌آوری و ارسال لاگ در عمل (Agents)

### مقدمه

چه کسی لاگ را از روی سرور/کانتینر جمع می‌کند و به سیستم مرکزی می‌فرستد؟ **Log Agentها**. انتخاب و استقرار درست آن‌ها — به‌ویژه در Kubernetes — تعیین‌کنندهٔ موفقیت centralized logging است. این فصل گزینه‌ها (Fluent Bit، Vector، Promtail) و الگوهای استقرار (DaemonSet/Sidecar) را بررسی می‌کند.

### مفاهیم اساسی

**الگوهای استقرار در Kubernetes:**
- **DaemonSet:** یک agent روی هر نود (مثل Promtail/Fluent Bit) → لاگ همهٔ پادهای نود را می‌خواند. کارآمد، استاندارد.
- **Sidecar:** یک container لاگ در کنار هر اپلیکیشن → جداسازی بهتر اما منابع بیشتر.
- **Node-level (stdout):** کوبرنتیز لاگ کانتینر را به stdout می‌نویسد؛ agent از `/var/log/pods` می‌خواند (رایج‌ترین).

**گزینه‌های Agent:**
- **Fluent Bit:** بسیار سبک (C)، چندورودی/چندخروجی (به Loki، ELK، Splunk). محبوب‌ترین در K8s.
- **Vector (توسط Datadog):** سبک و قدرتمند، پردازش پیشرفته، مقصدهای متعدد.
- **Promtail:** مخصوص Loki (ساده اما محدود به Loki).
- **Fluentd:** نسخهٔ بزرگتر/سنگین‌تر Fluent Bit (Ruby).
- **Logstash Beats (Filebeat):** در اکوسیستم ELK.

**پردازش (Processing):**
- **Parse:** تبدیل متن به structured (JSON/grok).
- **Enrich:** الصاق فیلد (namespace، pod_name، cluster).
- **Filter/Redact:** حذف PII، فیلتر خطوط بی‌ربط.
- **Route:** ارسال به مقصد مناسب (مثلاً لاگ حساس به سیستم جداگانه).

**تفاوت Pull/Push در لاگ:** برخلاف متریک (Prometheus Pull)، لاگ معمولاً **Push** است (agent می‌فرستد).

### مثال‌های عملی

**الف) Fluent Bit به‌عنوان DaemonSet (نمای کلی Helm):**
```bash
helm repo add fluent https://fluent.github.io/helm-charts
helm install fluent-bit fluent/fluent-bit \
  --set backend.type=loki \
  --set loki.host=loki:3100
```
فایل پیکربندی `[INPUT]` (tail /var/log/containers/*.log) و `[OUTPUT]` (loki) را دارد.

**ب) فیلتر Redact در Fluent Bit:**
```ini
[FILTER]
    Name    grep
    Match   *
    Exclude password \*        # حذف خطوط حاوی رمز
```

**ج) Vector pipeline (نمای کلی):**
```toml
[sources.kubernetes]
type = "kubernetes_logs"

[transforms.parse]
type = "remap"
inputs = ["kubernetes"]
source = '''
. |= parse_json!(.message)
'''

[sinks.loki]
type = "loki"
inputs = ["parse"]
endpoint = "http://loki:3100"
```

**د) لاگ stdout در K8s (روش استاندارد):**
```yaml
# اپلیکیشن فقط به stdout می‌نویسد؛ kubelet آن را در
# /var/log/pods/<ns>_<pod>_<id>/<container>/0.log ذخیره می‌کند
# DaemonSet agent آن را می‌خواند
```

### مقایسه و دیدگاه‌های مختلف

**DaemonSet در برابر Sidecar:**
- DaemonSet: منابع کمتر، مدیریت متمرکز؛ اما لاگ همهٔ پادها در یک agent.
- Sidecar: ایزولاسیون/کنترل بیشتر؛ منابع دوبرابر.

**Fluent Bit در برابر Vector:**
- Fluent Bit: بسیار سبک، پایدار، جامعه بزرگ.
- Vector: انعطاف پردازش بالاتر، اما جدیدتر.

**Push لاگ در برابر Pull متریک:**
- لاگ ذاتاً رویدادمحور → Push (agent می‌فرستد).
- متریک بازه‌مند → Pull (Prometheus می‌کشد).

**دیدگاه:** بیشتر تیم‌ها **Fluent Bit به‌عنوان DaemonSet** را ترجیح می‌دهند (سبکی + چندمقصدی). اگر همه‌چیز Loki است، Promtail کافی است.

### اشتباهات رایج

1. **خواندن لاگ از فایل محلی اپلیکیشن (نه stdout) در K8s:** پاد ephemeral است؛ stdout را بنویسید.
2. **DaemonSet بدون resource limit:** agent ممکن است نود را اشغال کند.
3. **ارسال لاگ حساس بدون Redact:** نقض امنیت.
4. **تکرار (duplicate) لاگ:** هم Promtail هم Fluent Bit همزمان → دوبرابر هزینه/حجم.
5. **عدم فشار (backpressure) management:** مقصد کند → agent حافظه پر → OOM.

### نکات مهم و ترفندها

- **اپلیکیشن فقط stdout/err بنویسد**؛ اجازه دهید K8s + agent بقیه را انجام دهند.
- **Resource Limits روی DaemonSet** (مثلاً 100m CPU / 100Mi) بگذارید.
- **Buffer/Retry:** agent باید روی قطع مقصد لاگ را در buffer نگه دارد (از دست نرود).
- **Label consistency:** agent فیلدهای `namespace`, `pod`, `container` را الصاق کند تا در Loki فیلتر شوند.
- **Grafana Alloy:** agent جدید چندمنظوره (Prometheus+Loki+OTel) — آیندهٔ این فضا.
- **Log Volume Monitoring:** agent خودش را مانیتور کنید تا فشار شناسایی شود.

### خلاصه فصل

**Log Agent** (معمولاً Fluent Bit به‌عنوان DaemonSet در K8s) لاگ‌ها را از stdout کانتینرها می‌خواند، parse/redact می‌کند و به Loki/ELK می‌فرستد. کلیدها: اپلیکیشن فقط stdout بنویسد، agent سبک با resource limit و buffer داشته باشد، و فیلدهای استاندارد (namespace/pod) الصاق کند. اشتباه رایج: خواندن از فایل محلی (ناپایدار در K8s) و ارسال بدون Redact. در فصل بعد سناریوی end-to-end لاگ را می‌سازیم.

---

## فصل ۱۰۵: سناریوی عملی — متمرکزسازی لاگ‌ها

### مقدمه

حالا همه قطعات لاگ را در یک سناریو ترکیب می‌کنیم: اپلیکیشن Flask لاگ ساختاریافته در stdout می‌نویسد → Fluent Bit (DaemonSet) آن را به Loki می‌فرستد → Grafana لاگ را جست‌وجو و داشبورد می‌کند → هشدار روی الگوی خطا. این فصل نسخهٔ عملیاتی است.

### مفاهیم اساسی

**معماری سناریو (K8s):**
```
Pod (Flask, stdout JSON log)
   │
   ▼ (kubelet → /var/log/pods/...)
Fluent Bit (DaemonSet) ──parse/label──▶ Loki ──▶ Grafana
                                                (Explore + Alert)
```

**گام‌ها:**
1. اپلیکیشن لاگ JSON به stdout.
2. Fluent Bit با DaemonSet لاگ را می‌خواند، label می‌زند (`app`, `namespace`)، به Loki می‌فرستد.
3. Grafana → Loki data source → Explore جست‌وجو با LogQL.
4. هشدار: «بیش از ۱۰ خطای ERROR در ۱ دقیقه» → Slack.
5. Retention: Loki لاگ را ۳۰ روز در S3 نگه می‌دارد.

**چالش‌های واقعی:** فشار روی Loki هنگام spike ترافیک، لاگ حجیم، و همبستگی با تریس (فایل ۲۲).

### مثال‌های عملی

**الف) اپلیکیشن Flask (stdout JSON):**
```python
import logging, json, sys
logging.basicConfig(stream=sys.stdout, level=logging.INFO,
    format='%(message)s')
log = logging.getLogger()
def log_event(event, **fields):
    log.info(json.dumps({"event": event, "level": "INFO", **fields}))
# در درخواست:
log_event("order_created", order_id=123, user_id=42, trace_id="t-9")
```

**ب) Fluent Bit ConfigMap (خلاصه):**
```ini
[INPUT]
    Name              tail
    Path              /var/log/containers/*.log
    multiline.parser  docker

[FILTER]
    Name              kubernetes
    Match             *

[OUTPUT]
    Name              loki
    Match             *
    Url               http://loki:3100/loki/api/v1/push
    Labels            {job="fluentbit", app="$kubernetes['labels']['app']"}
```

**ج) جست‌وجو در Grafana Explore (LogQL):**
```
{app="orders"} |= "ERROR" | json | trace_id="t-9"
```

**د) هشدار لاگ در Grafana (LogQL-based):**
```
expr: sum(count_over_time({app="orders"} |= "ERROR" [1m])) > 10
→ مسیریابی به Slack channel #logs-alerts
```

**ه) Helm یک‌خط برای کل پشته (در K8s):**
```bash
helm install loki grafana/loki-stack --set fluent-bit.enabled=true,promtail.enabled=false
# شامل Loki + Fluent Bit + Grafana
```

### مقایسه و دیدگاه‌های مختلف

**Loki+Fluent Bit در برابر ELK در تولید:**
- Loki: ارزان، یکپارچه با Grafana/Prometheus، برای اکثر تیم‌ها کافی.
- ELK: جست‌وجوی سنگین/SIEM، تیم امنیتی، هزینه بیشتر.
- **ترکیبی:** برخی لاگ عملیاتی در Loki، لاگ امنیتی در ELK (Fluent Bit هر دو را تغذیه می‌کند).

**DaemonSet یکتا در برابر چند agent:**
- یک Fluent Bit برای همه مقاصد (کارآمد).
- چند agent جدا → هدر رفت منابع.

**دیدگاه:** «شروع با Loki؛ اگر به SIEM رسیدید، ELK/OpenSearch اضافه کنید».

### اشتباهات رایج

1. **عدم پیکربندی Retention در Loki:** S3 پر می‌شود → هزینهٔ نجومی.
2. **Fluent Bit نخواندن multiline:** استک‌تریس خطا به چند خط تقسیم می‌شود → تحلیل سخت.
3. **هشدار روی متن لاگ بدون فیلتر label:** حجم زیاد → alert fatigue.
4. **ارسال لاگ DEBUG در تولید:** حجم ۱۰ برابر.
5. **عدم تست رویداد:** مطمئن شوید لاگ واقعاً در Grafana ظاهر می‌شود (مرحله به مرحله).

### نکات مهم و ترفندها

- **multiline.parser** در Fluent Bit را برای استک‌تریس فعال کنید.
- **Retention/Compactor** در Loki ( Helm values `retentionPeriod`) تنظیم کنید.
- **Derived Fields در Grafana:** فیلد `trace_id` را به لینک Tempo (فایل ۲۲) تبدیل کنید.
- **Log Volume Alert:** هشدار روی حجم کل لاگ (ناگهان ۱۰x → احتمالاً باگ لاگ‌نویسی).
- **Sampling در agent:** برای لاگ INFO در ترافیک بالا sample بگیرید.
- **LogQL Live Tail:** در Grafana `Live` برای دیدن لاگ لحظه‌ای.

### خلاصه فصل

سناریوی متمرکزسازی لاگ نشان داد: اپلیکیشن stdout JSON می‌نویسد → **Fluent Bit (DaemonSet)** parse/label می‌کند → **Loki** ذخیره ارزان → **Grafana** جست‌وجو (LogQL) و هشدار. کلیدهای موفقیت: لاگ ساختاریافته، تنظیم retention، فعال‌سازی multiline، و تست پایان‌به‌پایان. اشتباه بحرانی: نبود retention (هزینهٔ S3) و لاگ DEBUG در تولید. با این فایل بخش دوم Observability (لاگ) کامل شد؛ فایل بعدی به Tracing و OpenTelemetry (پایهٔ سوم) می‌پردازد.

---

## منابع و مراجع

- Grafana Loki Documentation — https://grafana.com/docs/loki/
  - LogQL: https://grafana.com/docs/loki/latest/query/
  - Promtail: https://grafana.com/docs/loki/latest/clients/promtail/
- Elastic Stack Docs — https://www.elastic.co/guide/index.html
  - Elasticsearch: https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html
  - Logstash: https://www.elastic.co/guide/en/logstash/current/index.html
  - Kibana: https://www.elastic.co/guide/en/kibana/current/index.html
- Fluent Bit — https://docs.fluentbit.io/
- Vector — https://vector.dev/docs/
- OpenSearch — https://opensearch.org/docs/
- «Logging in Action» (کتاب منتشر شده دربارهٔ Loki/Fluent Bit)
- Elastic Common Schema (ECS) — https://www.elastic.co/guide/en/ecs/current/index.html

**بعدی:** [فایل ۲۲ — OpenTelemetry و Tracing (Observability بخش ۳)](22-OpenTelemetry-and-Tracing.md)
