# فایل ۲� — محاسبات بدون‌سرور (Serverless)

**مدت یادگیری**: ۱۰ ساعت
**سطح سختی**: متوسط تا سخت
**پیش‌نیاز**: فایل ۱۸ (AWS)، فایل ۱۱ (Kubernetes)، فایل ۲۲ (Observability)، فایل ۱۵ (IaC/Terraform)

---

## فهرست محتوا

- [فصل ۱۲۵: مبانی Serverless و فلسفهٔ آن](#فصل-۱۲۵-مبانی-serverless-و-فلسفهٔ-آن)
- [فصل ۱۲۶: AWS Lambda — معماری و جزئیات](#فصل-۱۲۶-aws-lambda--معماری-و-جزئیات)
- [فصل ۱۲۷: معماری Serverless و الگوها](#فصل-۱۲۷-معماری-serverless-و-الگوها)
- [فصل ۱۲۸: Serverless در K8s (Knative) و چالش‌ها](#فصل-۱۲۸-serverless-در-k8s-knative-و-چالشها)
- [فصل ۱۲۹: سناریوی عملی و بهترین روش‌ها](#فصل-۱۲۹-سناریوی-عملی-و-بهترین-روشها)

---

## فصل ۱۲۵: مبانی Serverless و فلسفهٔ آن

### مقدمه

واژهٔ **Serverless** گمراه‌کننده است — سرور هست، اما شما آن را مدیریت نمی‌کنید. ارائه‌دهنده مقیاس‌بندی، پچ و زمان‌بندی را بر عهده دارد؛ شما فقط کد (تابع) یا سرویس مدیریت‌شده را می‌دهید و **بر اساس مصرف** پرداخت می‌کنید. این فصل تفاوت FaaS/BaaS، cold start و کاربردها را می‌سازد.

### مفاهیم اساسی

**دو مدل Serverless:**
- **FaaS (Function as a Service):** اجرای تابع در پاسخ به رویداد. مثال: AWS Lambda، Azure Functions، GCP Cloud Functions.
- **BaaS (Backend as a Service):** سرویس‌های مدیریت‌شده (Database/Auth/Storage) — مثلاً Firebase، DynamoDB، S3.

**ویژگی‌های کلیدی:**
- **Scale-to-zero:** وقتی ترافیک نیست، هزینه/منابع صفر است.
- **Event-driven:** تابع در پاسخ به رویداد (HTTP، پیام، آپلود فایل) اجرا می‌شود.
- **Pay-per-use:** پرداخت فقط بابت میلی‌ثانیهٔ اجرا + تعداد فراخوانی.
- **No server management:** نه OS، نه پچ، نه نگهداری.

**Cold Start (شروع سرد):**
- اولین فراخوان پس از بیکاری، زمان صرف مقداردهی محیط (بالا آوردن runtime) می‌شود → تأخیر اضافه (صدم‌ثانیه تا چند ثانیه).
- عوامل: زبان (Node/Python سریع‌تر از Java/.NET)، اندازهٔ بسته، حجم حافظه.

**کاربردهای مناسب:**
- پردازش پس‌زمینه (image resize، ویدیو ترنسکد).
- وب‌هوک‌ها و API سبک.
- پردازش جریان داده (stream processing).
- اتوماسیون/CRON جایگزین.

**کاربردهای نامناسب:**
- کارهای طولانی (محدودیت زمان اجرا).
- بار پیوسته با ترافیک ثابت (EC2/Containers ارزان‌تر).
- نیاز به state محلی پایدار.

**ارائه‌دهندگان:** AWS Lambda، Azure Functions، GCP Cloud Functions / **Cloud Run** (کانتینر سرورلس)، **Knative** (خودمیزبان)، Cloudflare Workers.

### مثال‌های عملی

**الف) معماری Serverless ساده:**
```
Upload File → S3 → triggers → Lambda (resize) → saves thumbnail → SQS notify
هیچ سروری دستی روشن نمی‌شود؛ فقط بابت اجرای Lambda پرداخت می‌شود
```

**ب) مقایسه هزینه (مفهومی):**
```
سرویس پیوسته (۱۰۰۰ RPS همیشه):  EC2 ۲۴/۷ ≈ ۷۳$/ماه
همان بار اما ناپیوسته (۴ ساعت/روز): Lambda ≈ ۸$/ماه (scale-to-zero در باقی‌مانده)
```

**ج) اندازه‌گیری cold start (مفهومی):**
```
اولین درخواست پس از ۱۰ دقیقه بیکاری: ۸۰۰ms (شامل cold start)
درخواست‌های بعدی: ۲۰ms
```

### مقایسه و دیدگاه‌های مختلف

**Serverless در برابر Containers (Fargate/ECS):**
- Serverless: مصرف محور، مدیریت صفر، برای رویدادمحور.
- Containers: کنترل بیشتر، بار پیوسته، runtime آزاد.

**FaaS در برابر Cloud Run (کانتینر سرورلس):**
- FaaS: تابع کوچک، محدودیت زمان، رویدادمحور خالص.
- Cloud Run: کانتینر کامل، HTTP-focus، scale-to-zero، بسیار انعطاف‌پذیر.

**دیدگاه‌ها:**
- **مبلغان:** سرعت نوآوری، هزینهٔ کم برای بار متغیر.
- **منتقدان:** vendor lock-in، debugging سخت‌تر، cold start، محدودیت زمان.
- **رویکرد عملی:** «بار متغیر/رویدادمحور → Serverless؛ بار پیوسته/حساس به latency → Container/VM».

### اشتباهات رایج

1. **استفاده از Lambda برای بار ۲۴/۷ پایدار:** گران‌تر از EC2/Container.
2. **نادیده گرفتن cold start:** API حساس به تأخیر با اولین درخواست کند مواجه می‌شود.
3. **قرار دادن state در تابع:** توابع بی‌حالت (stateless) باشند؛ state در BaaS (DB/Cache).
4. **بستهٔ بزرگ (large deployment):** حجم کد/dependency زیاد → cold start بدتر.
5. **فراموشی محدودیت زمان:** ۱۵ دقیقه سقف Lambda → وظایف طولانی شکست می‌خورند.

### نکات مهم و ترفندها

- **Provisioned Concurrency** (AWS): نگه داشتن تعدادی نمونهٔ گرم → حذف cold start برای بار حساس.
- **تابع را کوچک و stateless نگه دارید**؛ منطق stateful را به سرویس مدیریت‌شده بسپارید.
- **منابع (حافظه) را tun کنید:** حافظه بیشتر = CPU بیشتر = اجرای سریع‌تر (گاهی ارزان‌تر از آنچه فکر می‌کنید چون زمان کم می‌شود).
- **Keep-warm hacks** را جایگزین Provisioned Concurrency کنید (تمیزتر).
- **Step Functions** برای اورکستراسیون چند تابع (فصل بعد).

### خلاصه فصل

**Serverless** (FaaS/BaaS) اجرای کد بر اساس رویداد با **scale-to-zero** و **پرداخت بر اساس مصرف** است — بدون مدیریت سرور. **Cold Start** (تأخیر اولین اجرا) مهم‌ترین ضعف آن است. مناسب برای بار متغیر/رویدادمحور، نامناسب برای بار پیوسته یا وظایف طولانی. اشتباهات: استفاده برای بار ۲۴/۷ و نگه داشتن state در تابع. در فصل بعد AWS Lambda را عمیق‌تر می‌بینیم.

---

## فصل ۱۲۶: AWS Lambda — معماری و جزئیات

### مدمه

**AWS Lambda** پیشگام Serverless است. درک triggerها، نسخه‌ها (versions/aliases)، لایه‌ها (layers)، event source mapping و محدودیت‌ها برای استفادهٔ واقعی ضروری است. این فصل جزئیات فنی Lambda را بررسی می‌کند.

### مفاهیم اساسی

**اجزا:**
- **Function:** کد + runtime (Python/Node/Java/Go/...) + حافظه (۱۲۸MB–۱۰GB) + timeout (تا ۱۵ دقیقه).
- **Trigger (محرک):** منبع فراخوان — API Gateway، S3، SQS، EventBridge، DynamoDB Streams، و غیره.
- **Event:** آبجکت JSON که هنگام فراخوان به تابع داده می‌شود (شامل `event` و `context`).
- **Layer:** بستهٔ کد/کتابخانهٔ مشترک قابل اشتراک میان چند تابع (کاهش حجم).
- **Version:** نسخهٔ غیرقابل‌تغییر از تابع (`:1`, `:2`).
- **Alias:** اشاره‌گر به نسخه (مثل `prod` → `:2`) — برای Blue-Green/Canary بدون تغییر فراخواننده.

**انواع فراخوان:**
- **Synchronous (همگام):** API Gateway → انتظار پاسخ (مثل HTTP).
- **Asynchronous (ناهمگام):** S3/SNS → صف داخلی Lambda، پاسخ فوری نمی‌دهد؛ در صورت خطا **DLQ** (Dead Letter Queue).
- **Stream (جریان):** DynamoDB/Kinesis → **Event Source Mapping** پارتیشن‌بندی را مدیریت می‌کند.

**محدودیت‌ها:**
- زمان اجرا ۱۵ دقیقه، حافظه ۱۰GB، اندازهٔ بسته ۲۵۰MB (unzipped).
- هم‌روندی (concurrency) پیش‌فرض حساب‌محور (می‌توان محدود کرد تا Downstream را protection کند).

**بهینه‌سازی:**
- **Provisioned Concurrency:** نمونه‌های گرم از پیش.
- **SnapStart** (Java): کاهش چشمگیر cold start.
- **Container Images:** اجرای Lambda از image تا ۱۰GB (برای وابستگی سنگین).

### مثال‌های عملی

**الف) تابع Python (handler):**
```python
import json
def lambda_handler(event, context):
    # event: دیکشنری JSON از Trigger
    user_id = event.get("user_id")
    # منطق...
    return {
        "statusCode": 200,
        "body": json.dumps({"user_id": user_id, "ok": True})
    }
# handler: "filename.lambda_handler"
```

**ب) نسخه + Alias (Blue-Green):**
```bash
aws lambda publish-version --function-name orders
aws lambda create-alias --function-name orders --name prod --function-version 2
# فراخواننده به 'orders:prod' اشاره می‌کند، نه نسخهٔ سخت‌کد
# ارتقا: فقط alias را به نسخهٔ ۳ آپدیت کنید (بدون تغییر فراخواننده)
```

**ج) Event Source Mapping (DynamoDB Stream → Lambda):**
```bash
aws lambda create-event-source-mapping \
  --function-name orders-processor \
  --event-source-arn arn:aws:dynamodb:...:table/Orders/stream/... \
  --starting-position LATEST
```

**د) DLQ برای فراخوان ناهمگام:**
```bash
aws lambda update-function-event-invoke-config \
  --function-name orders \
  --destination-config '{"OnFailure":{"Destination":"arn:aws:sqs:...:dlq"}}'
```

### مقایسه و دیدگاه‌های مختلف

**Synchronous در برابر Asynchronous:**
- Sync: نیاز به پاسخ فوری (API)؛ خطا → ۵xx به کاربر.
- Async: پردازش پس‌زمینه؛ خطا → retry + DLQ.

**Alias در برابر Version مستقیم:**
- Alias: تجرید برای Canary/Blue-Green (توصیه).
- Version مستقیم: شکستن فراخواننده هنگام ارتقا.

**Container Image در برابر ZIP:**
- Image: وابستگی سنگین، یکپارچگی با ECR، تا ۱۰GB.
- ZIP: سبک، سریع آپلود، مناسب اکثر موارد.

**دیدگاه:** «Lambda عالی است وقتی زمان اجرا کوتاه و بار متغیر است؛ برای هر چیز دیگر، فکر دوباره کنید».

### اشتباهات رایج

1. **تابع بیش از ۱۵ دقیقه:** وظیفهٔ طولانی می‌شکند — از Step Functions/Container استفاده کنید.
2. **عدم استفاده از Alias:** ارتقای نسخه فراخوانندگان را می‌شکند.
3. **حذف نسخه‌های فعال:** نسخه غیرقابل‌تغییر است اما حذفش می‌تواند alias را بی‌ارزش کند.
4. **بستهٔ بزرگ بدون Layer:** کتابخانهٔ مشترک را در هر تابع تکرار → حجم/زمان بیشتر.
5. **عدم تنظیم DLQ:** خطاهای ناهمگام بی‌سر و صدا گم می‌شوند.
6. **Concurrency بی‌حد:** سیل درخواست → فشار روی DB پایین‌دستی.

### نکات مهم و ترفندها

- **Provisioned Concurrency** برای API حساس به latency.
- **Power Tuning:** ابزار `aws-lambda-power-tuning` برای یافتن حافظهٔ بهینه (هزینه/سرعت).
- **Lambda Layers** برای Shared code/SDK.
- **X-Ray** (AWS tracing) را برای دید تابع فعال کنید (معادل OTel فصل ۱۰۸).
- **Environment Variables** برای تنظیمات (نه رمز — از Secrets Manager فصل ۱۱۹).
- **Lambda Extensions** برای agentهای observability (Datadog/OTel).

### خلاصه فصل

**AWS Lambda** با **Function/Trigger/Event**، **Layers** (کتابخانهٔ مشترک)، و **Version/Alias** (برای Blue-Green/Canary) ساختار یافته است. فراخوان **Sync/Async/Stream** و **DLQ** برای قابلیت اطمینان حیاتی‌اند. محدودیت **۱۵ دقیقه/۱۰GB** و بهینه‌سازی با **Provisioned Concurrency/Power Tuning** کلید عملیات‌اند. اشتباهات: تابع طولانی و عدم استفاده از Alias/DLQ. در فصل بعد معماری و الگوهای Serverless را می‌بینیم.

---

## فصل ۱۲۷: معماری Serverless و الگوها

### مدمه

Serverless فقط «تابع» نیست — معماری کامل شامل گیت‌وی، صف، اورکستراسیون و IaC است. این فصل الگوهای معماری (API Gateway + Lambda، event-driven، Step Functions) و ابزارهای استقرار (Serverless Framework/SAM/CDK) را بررسی می‌کند.

### مفاهیم اساسی

**الگوی API Gateway + Lambda:**
- API Gateway (HTTP/REST) درخواست را به Lambda می‌فرستد؛ پاسخ برمی‌گردد. معادل سرور API بدون مدیریت سرور.
- **HTTP API** (سبک‌تر/ارزان‌تر) در مقابل **REST API** (ویژگی بیشتر).

**معماری Event-Driven:**
- **SQS** (صف پیام): صف بین اجزا، مقاومت در برابر spike.
- **SNS** (انتشار/عضویت): یک رویداد → چند مشترک.
- **EventBridge:** گذرگاه رویداد (event bus) برای معماری loosely-coupled.
- **Kinesis/Kafka:** جریان داده با بالا.

**اورکستراسیون: Step Functions:**
- ماشین وضعیت (state machine) برای هماهنگی چند تابع/سرویس با خطاگیری و بازپخش.
- مناسب جریان‌های چندمرحله‌ای (مثل پردازش سفارش).

**IaC برای Serverless:**
- **AWS SAM** (Serverless Application Model): قالب YAML ساده‌شده برای Lambda/API GW.
- **Serverless Framework:** چندابری، تجرید بالا.
- **AWS CDK** (فصل ۹۰): کد-محور.
- **Terraform** (فصل ۷۵): چندابری.

**الگوهای طراحی:**
- **Saga Pattern:** تراکنش توزیع‌شده با جبران (compensation).
- **Fan-out/Fan-in:** یک رویداد → چند پردازش موازی → تجمیع.
- **Strangler Fig:** مهاجرت تدریجی مونولیت به Serverless.

### مثال‌های عملی

**الف) SAM Template (خلاصه):**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Resources:
  OrdersApi:
    Type: AWS::Serverless::Function
    Properties:
      Handler: app.lambda_handler
      Runtime: python3.12
      Events:
        Api:
          Type: Api
          Properties: { Path: /orders, Method: post }
```
استقرار: `sam build && sam deploy --guided`

**ب) Step Functions (تعاریف مفهومی):**
```
OrderReceived → [Lambda: Validate] → [Lambda: Charge] → [Lambda: Ship]
                                            ↓ (خطا)
                                       [Lambda: Refund]  (compensation)
```

**ج) EventBridge Rule:**
```bash
aws events put-rule --name order-created --event-pattern '{"source":["orders"]}'
aws events put-targets --rule order-created --targets "Id=1,Arn=arn:aws:lambda:...:notify"
```

**د) Serverless Framework (`serverless.yml`):**
```yaml
service: orders-svc
provider:
  name: aws
  runtime: python3.12
functions:
  create:
    handler: handler.create
    events:
      - http: POST /orders
```

### مقایسه و دیدگاه‌های مختلف

**SAM در برابر Serverless Framework:**
- SAM: بومی AWS، یکپارچه با ابزار AWS.
- Serverless Framework: چندابری، اکوسیستم پلاگین بزرگ.

**Step Functions در برابر Orchestrator دستی:**
- Step Functions: مدیریت state/retry/visual، اما هزینه و یادگیری.
- دستی (در کد): ساده برای مورد کوچک.

**API Gateway در برابر ALB برای Lambda:**
- API GW: RESTful، ویژگی‌های API (auth/throttle).
- ALB: ساده‌تر برای HTTP، ارزان‌تر در برخی موارد.

**دیدگاه:** «Serverless معماری است نه فقط تابع — اگر فقط تابع می‌نویسید، نیمی از ارزش را از دست داده‌اید».

### اشتباهات رایج

1. **Monolith در تابع:** یک تابع غول‌آسا → کند، سخت‌调试، نقض روح Serverless.
2. **عدم استفاده از صف (SQS) برای spike:** ترافیک ناگهانی مستقیماً DB را می‌کشد.
3. **IaC دستی در کنسول:** استقرار غیرقابل بازتولید.
4. **گره خوردن به یک ابر بدون الگوی جداگانه:** lock-in شدید (استفاده از event سفارشی).
5. **فراموشی هزینهٔ API GW:** فراخوان زیاد API Gateway می‌تواند گران شود.

### نکات مهم و ترفندها

- **توابع را تک‌منظوره (single-purpose) نگه دارید** (یک تابع = یک عمل).
- **SQS برای جذب spike** بین Gateway و پردازش.
- **EventBridge** را برای coupling کم ترجیح دهید.
- **SAM/Serverless Framework** را برای IaC تک‌فرمانی استفاده کنید.
- **X-Ray/OTel** برای ردیابی درخت تابع (فصل ۱۰۸).
- **تست محلی:** `sam local invoke` / `serverless invoke local` قبل از استقرار.

### خلاصه فصل

معماری Serverless فراتر از تابع است: **API Gateway** ورودی، **SQS/SNS/EventBridge** صف/رویداد، **Step Functions** اورکستراسیون، و **SAM/Serverless Framework/CDK** استقرار (IaC) را شامل می‌شود. الگوهای **Saga/Fan-out/Strangler** معماری توزیع‌شده را می‌سازند. اشتباهات: تابع مونولیتی و عدم استفاده از صف برای جذب spike. توصیه: توابع تک‌منظوره + IaC یکپارچه. در فصل بعد Serverless در K8s و چالش‌ها را می‌بینیم.

---

## فصل ۱۲۸: Serverless در K8s (Knative) و چالش‌ها

### مدمه

اگر نمی‌خواهید به ابر عمومی قفل شوید، می‌توانید Serverless را روی خود Kubernetes اجرا کنید. **Knative** استاندارد این کار است. این فصل Knative، scale-to-zero در K8s، و چالش‌های کلی Serverless (lock-in، debugging، state، هزینه) را بررسی می‌کند.

### مفاهیم اساسی

**Knative چیست؟**
مجموعه‌ای از اجزا روی K8s برای اجرای بارهای Serverless:
- **Knative Serving:** مقیاس‌بندی خودکار (تا صفر)، مسیریابی ترافیک، نسخه‌گذاری.
- **Knative Eventing:** تحویل رویداد (CloudEvents) — معماری event-driven در K8s.

**چگونه کار می‌کند:**
- شما یک **Service** (Knative) تعریف می‌کنید (شبیه Deployment اما با ویژگی Serverless).
- وقتی درخواستی نیست → تعداد پاد به **صفر** می‌رسد (scale-to-zero).
- درخواست جدید → Knative پاد را بالا می‌آورد (cold start در K8s بسته به تصویر).
- **Activator** ترافیک را میان پادهای صفر/فعال مدیریت می‌کند.

**مقایسه با ابر عمومی:**
- مزیت: چندابری، کنترل کامل، بدون lock-in به Lambda.
- ضعف: شما مسئول نگهداری K8s + Knative (هزینهٔ عملیاتی).

**چالش‌های کلی Serverless:**
- **Vendor lock-in:** هر ابر API رویداد خاص خود را دارد (استفاده از CloudEvents کمک می‌کند).
- **Debugging:** اجرای توزیع‌شده + ephemeral → سخت‌تر از ماشین واحد.
- **State:** توابع بی‌حالت باشند؛ state در DB/Cache بیرونی.
- **Cold start:** هنوز وجود دارد (در K8s بسته به تصویر).
- **هزینهٔ غیرشفاف:** بار پیوسته ممکن است غیرمنتظره گران شود.
- **زمان اجرا:** محدودیت (Lambda ۱۵دقیقه؛ Knative انعطاف بیشتر).

**پیوند به Observability (فصل ۲۲):**
در Serverless، **Distributed Tracing (OTel)** حیاتی است چون هر درخواست از چند تابع/سرویس رد می‌شود.

### مثال‌های عملی

**الف) Knative Service (YAML):**
```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata: { name: orders }
spec:
  template:
    spec:
      containers:
        - image: registry/orders:latest
          env:
            - { name: LOG_LEVEL, value: info }
      # autoscaling
      autoscaler:
        minScale: 0      # scale-to-zero فعال
        maxScale: 10
```

**ب) مسیریابی کاناری (ترافیک ۸۰/۲۰):**
```yaml
spec:
  traffic:
    - { revisionName: orders-v1, percent: 80 }
    - { revisionName: orders-v2, percent: 20 }
```

**ج) فعال‌سازی Knative:**
```bash
kubectl apply -f https://github.com/knative/serving/releases/download/.../serving-crds.yaml
# یا با Operator
```

**د) CloudEvents (استاندارد رویداد):**
```json
{
  "specversion": "1.0",
  "type": "order.created",
  "source": "orders-service",
  "id": "evt-123",
  "data": { "order_id": 42 }
}
```

### مقایسه و دیدگاه‌های مختلف

**Knative در برابر Lambda:**
- Knative: کنترل/چندابری/بدون lock-in، اما نگهداری شما.
- Lambda: مدیریت‌شده کامل، اما قفل به AWS.

**Knative در برابر KEDA (مقیاس‌بندی معمولی):**
- KEDA: مقیاس Deployment بر اساس متریک (SQS/Kafka) اما scale-to-zero ضعیف‌تر از Knative.
- Knative: Serverless-native از ابتدا.

**دیدگاه:** «Serverless در K8s برای کسانی است که می‌خواهند مدل مصرف ابری را بدون رها کردن زیرساخت خودشان».

### اشتباهات رایج

1. **نصب Knative بدون درک بار عملیاتی:** خود K8s + Knative نگهداری می‌خواهد.
2. **توقع cold start صفر:** باز هم تأخیر اولیه هست (تصویر بزرگ = بدتر).
3. **فراموشی Observability:** توابع توزیع‌شده بدون تریس غیرقابل دیباگ.
4. **state در پاد:** با scale-to-zero پاد حذف → دادهٔ محلی گم می‌شود.
5. **قفل شدن روی event فرمت خاص:** از CloudEvents استفاده کنید تا قابل جابه‌جایی بماند.

### نکات مهم و ترفندها

- **تصویر کوچک (distroless)** برای cold start بهتر در Knative.
- **CloudEvents** را استاندارد رویداد قرار دهید (lock-in کمتر).
- **OTel/تریس** را اجباری کنید (فصل ۱۰۸) — در Serverless حیاتی.
- **minScale: 1** برای سرویس حساس به latency (جلوگیری از cold start، هزینه کمی بیشتر).
- **KEDA** را برای مقیاس‌بندی بر اساس صف بررسی کنید اگر Knative سنگین است.
- **هزینه را مانیتور کنید:** بار پیوسته در Serverless ممکن است گران‌تر از Container شود.

### خلاصه فصل

**Knative** Serverless را روی K8s می‌آورد (Serving با scale-to-zero + Eventing با CloudEvents) — راهی برای مدل مصرف ابری بدون lock-in به ابر عمومی، اما با هزینهٔ نگهداری خودتان. چالش‌های کلی Serverless شامل **lock-in، debugging سخت، state، cold start و هزینهٔ غیرشفاف**اند. **OTel/Tracing** در این محیط غیرقابل‌جایگزین است. اشتباهات: انتظار cold start صفر و نگه داشتن state در پاد. در فصل بعد سناریو و بهترین روش‌ها را می‌بینیم.

---

## فصل ۱۲۹: سناریوی عملی و بهترین روش‌ها

### مدمه

همهٔ مفاهیم Serverless را در یک سناریو ترکیب می‌کنیم: یک API سرورلس (API GW + Lambda) با استقرار از طریق SAM/Serverless Framework، مانیتورینگ با OTel، و رعایت بهترین روش‌ها. سپس فهرست pitfalls را مرور می‌کنیم.

### مفاهیم اساسی

**گام‌های سناریو:**
1. طراحی تابع کوچک و stateless (Python).
2. تعریف منابع با SAM (`template.yaml`).
3. استقرار با `sam deploy`.
4. فعال‌سازی X-Ray/OTel برای تریس.
5. تنظیم Provisioned Concurrency برای API حساس.
6. DLQ برای پردازش ناهمگام.
7. تست بار (k6) و بررسی هزینه/عملکرد.

**ارزیابی موفقیت:** انتشار یک‌فرمانی، تریس کامل هر درخواست، و هزینهٔ بهینه در بار متغیر.

**بهترین روش‌ها (Top Practices):**
- توابع single-purpose و stateless.
- از BaaS (DynamoDB/S3) برای state.
- IaC اجباری (SAM/Serverless/CDK).
- Observability (متریک/لاگ/تریس) از روز اول.
- محدود کردن concurrency برای محافظت Downstream.
- استفاده از Alias برای انتشار ایمن.

### مثال‌های عملی

**الف) سناریوی کامل (SAM):**
```yaml
# template.yaml
Resources:
  ApiFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: app.handler
      Runtime: python3.12
      MemorySize: 512
      Timeout: 10
      Events:
        Api: { Type: Api, Properties: { Path: /items, Method: get } }
      Environment: { Variables: { TABLE: items } }
```
```bash
sam build && sam deploy --guided
```

**ب) مانیتورینگ با OTel (در تابع):**
```python
from opentelemetry import trace
tracer = trace.get_tracer("api")
def handler(event, ctx):
    with tracer.start_as_current_span("get-items") as span:
        # ... منطق
        return {"items": []}
# OTel Collector (فصل ۱۰۷) تریس را به Tempo می‌فرستد
```

**ج) تست بار و بررسی (مفهومی):**
```bash
k6 run -u 50 -d 2m script.js
# مشاهده: p99 latency، هزینه تخمینی، خطای cold start در دقایق اول
```

**د) فهرست بازبینی (Checklist):**
```
[ ] تابع < ۱۵ دقیقه اجرا می‌شود؟
[ ] state در BaaS است نه در تابع؟
[ ] IaC (SAM/CDK) دارید؟
[ ] Alias برای انتشار ایمن؟
[ ] DLQ برای async تنظیم شد؟
[ ] OTel/X-Ray فعال است؟
[ ] Concurrency محدود شد؟
```

### مقایسه و دیدگاه‌های مختلف

**SAM در برابر Serverless Framework در تیم:**
- SAM: تیم AWS-محور، یکپارچگی ابزار AWS.
- Serverless Framework: تیم چندابری یا علاقه‌مند به پلاگین.

**Provisioned Concurrency در برابر Keep-warm:**
- Provisioned: رسمی، قابل اطمینان.
- Keep-warm (CRON ping): hack، پرهزینه‌تر و نامطمئن.

**دیدگاه:** «Serverless شتاب‌دهنده است نه جادو — بدون معماری درست، فقط هزینه و پیچیدگی جدید اضافه می‌کنید».

### اشتباهات رایج

1. **انتشار دستی از کنسول:** غیرقابل بازتولید.
2. **عدم محدود کردن concurrency:** سیل → خرابی DB.
3. **تست نکردن cold start:** کاربران اولیه تجربهٔ بد می‌بینند.
4. **نادیده گرفتن هزینهٔ API GW:** در ترافیک بالا گران.
5. **عدم تریس:** عیب‌یابی تابع در تولید غیرممکن.

### نکات مهم و ترفندها

- **LocalStack** برای شبیه‌سازی AWS محلی (تست بدون هزینه).
- **SAM Accelerate** برای چرخهٔ توسعه سریع‌تر.
- **Lambda Power Tuning** برای بهینه‌سازی هزینه/سرعت.
- **Structured logging** (فصل ۱۰۱) اجباری — در تابع پراکنده لاگ بی‌ساختار بی‌فایده است.
- **Canary Lambda alias** + CloudWatch alarms برای بازگشت خودکار.
- **هزینه را با AWS Cost Explorer برچسب‌گذاری کنید** (فصل ۸۶).

### خلاصه فصل

سناریوی Serverless نشان داد چگونه **SAM/Serverless Framework** استقرار یک‌فرمانی می‌دهد، **API GW + Lambda** یک API بی‌سرور می‌سازد، و **OTel/X-Ray** تریس کامل می‌دهد. کلیدها: تابع single-purpose/stateless، **Alias** برای انتشار ایمن، **DLQ** برای async، و **محدودسازی concurrency**. اشتباهات: انتشار دستی، عدم محدود کردن concurrency، و نادیده گرفتن cold start. با این فایل بخش Serverless کامل شد؛ فایل بعدی پروژه‌های Capstone (ترکیب همه) است.

---

## منابع و مراجع

- AWS Lambda Documentation — https://docs.aws.amazon.com/lambda/
  - Concepts: https://docs.aws.amazon.com/lambda/latest/dg/lambda-introduction.html
  - Versions/Aliases: https://docs.aws.amazon.com/lambda/latest/dg/configuration-versions.html
  - Provisioned Concurrency: https://docs.aws.amazon.com/lambda/latest/dg/provisioned-concurrency.html
- AWS SAM — https://docs.aws.amazon.com/serverless-application-model/
- Serverless Framework — https://www.serverless.com/framework/docs/
- AWS Step Functions — https://docs.aws.amazon.com/step-functions/
- Knative — https://knative.dev/docs/
- CloudEvents — https://cloudevents.io/
- Google Cloud Run — https://cloud.google.com/run/docs
- AWS Power Tuning — https://github.com/alexcasalbon/aws-lambda-power-tuning
- LocalStack — https://www.localstack.cloud/

**بعدی:** [فایل ۲۷ — پروژه‌های Capstone (ترکیب همه)](27-Capstone-Projects.md)
