# فایل ۰۴ — مبانی CI/CD و خطوط لوله

**مدت یادگیری**: ۹ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۱، ۰۲، ۰۳ (Git و branching)

---

## فهرست محتوا

- [فصل ۱۶: مفاهیم پایه CI/CD](#فصل-۱۶-مفاهیم-پایه-cicd)
- [فصل ۱۷: معماری یک Pipeline (مراحل، Jobها و Artifactها)](#فصل-۱۷-معماری-یک-pipeline-مراحل-jobها-و-artifactها)
- [فصل ۱۸: استراتژی‌های Build و Caching](#فصل-۱۸-استراتژیهای-build-و-caching)
- [فصل ۱۹: تست در Pipeline (واحد، یکپارچه‌سازی، End-to-End)](#فصل-۱۹-تست-در-pipeline-واحد-یکپارچهسازی-end-to-end)
- [فصل ۲۰: استراتژی‌های استقرار (Blue-Green، Canary، Rolling)](#فصل-۲۰-استراتژیهای-استقرار-blue-green-canary-rolling)

---

## فصل ۱۶: مفاهیم پایه CI/CD

### مقدمه

همهٔ چیزی که تا اینجا یاد گرفتیم — Git، branching، review — به یک هدف ختم می‌شود: **رساندن کد به دست کاربر به صورت سریع و ایمن**. این دقیقاً کاری است که **CI/CD** انجام می‌دهد. CI/CD قلب تپندهٔ خودکارسازی DevOps است.

در این فصل تفاوت بین Continuous Integration، Continuous Delivery و Continuous Deployment را روشن می‌کنیم، و می‌فهمیم چرا این سه رکن تحویل نرم‌افزار مدرن هستند.

### مفاهیم اساسی

**Continuous Integration (CI — ادغام مداوم):**
تمرینی که در آن توسعه‌دهندگان تغییرات کد را **به صورت مکرر** (چند بار در روز) به مخزن مشترک ادغام می‌کنند. هر ادغام با یک **build خودکار** و **تست خودکار** اعتبارسنجی می‌شود تا ادغام‌های شکسته (broken integrations) سریع پیدا شوند.

هدف: شناسایی مشکلات ادغام زودهنگام — اصطلاحاً «Integration Hell» را حذف می‌کند.

**Continuous Delivery (CD — تحویل مداوم):**
توسعهٔ CI به این معنی که کد همیشه در وضعیتی است که **آمادهٔ انتشار به تولید** است. استقرار هنوز دستی (manual trigger) است اما هر لحظه می‌توان آن را انجام داد.

**Continuous Deployment (استقرار مداوم):**
مثل Continuous Delivery، اما استقرار به تولید **کاملاً خودکار** است — بدون دکمهٔ انسانی. هر کامیتی که تست‌ها را رد کند، به تولید می‌رود.

| مفهوم | ادغام خودکار | تست خودکار | استقرار به تولید |
|---|---|---|---|
| CI | ✅ | ✅ | ❌ |
| CD (Delivery) | ✅ | ✅ | ⏸ دستی |
| CD (Deployment) | ✅ | ✅ | ✅ خودکار |

**DORA Metrics (چهار شاخص کلیدی):**
1. **Deployment Frequency** — چند بار در روز/هفته مستقر می‌شوید؟
2. **Lead Time for Changes** — از کامیت تا تولید چقدر طول می‌کشد؟
3. **Change Failure Rate** — چند درصد استقرارها خراب می‌شوند؟
4. **Time to Restore Service** — پس از خرابی چقدر طول می‌کشد درست شود؟

سازمان‌های «elite» روزانه چندین بار مستقر می‌شوند و Lead Time آن‌ها کمتر از یک ساعت است.

### مثال‌های عملی

**۱. یک فایل CI ساده (مفهومی، فرامدل‌مستقل):**

```yaml
pipeline:
  - stage: build
    run: npm install && npm run build
  - stage: test
    run: npm test
  - stage: deploy
    when: manual   # در Continuous Delivery
    run: ./deploy.sh production
```

**۲. محاسبه Lead Time (مثال):**

```
کامیت: 09:00
build: 09:02 (۲ دقیقه)
test:  09:08 (۶ دقیقه)
deploy: 09:15 (۷ دقیقه)
Lead Time = 15 دقیقه ✅ (عالی)
```

**۳. نمونهٔ خروجی یک build شکست‌خورده:**

```
$ npm test
FAIL  src/auth.test.js
  ✕ should reject invalid token (42ms)
1 failed, 24 passed
Pipeline aborted. Fix and push again.
```

### مقایسه و دیدگاه‌های مختلف

**Delivery vs Deployment — کدام؟**
- **Continuous Delivery** برای اکثر تیم‌ها توصیه می‌شود: خودکارسازی کامل تا پیش از تولید، اما یک انسان تصمیم می‌گیرد «کِی» منتشر کنیم (مناسب برای کنترل ریسک، محصولات regulated).
- **Continuous Deployment** برای SaaSهای بالغ با تست بسیار قوی: سرعت حداکثری، اما نیازمند فرهنگ و ابزار فوق‌العاده.

**CI در مقابل Build Server سنتی:**
- سیستم شبانه (nightly build) قدیمی: کند، بازخورد دیرهنگام.
- CI مدرن: هر کامیت، بازخورد دقایقه‌ای.

**دیدگاه‌ها:** برخی معتقدند Deployment خودکار بی‌وقفه ریسک دارد؛ برخی (طرفدار Lean) معتقدند فقط با خودکارسازی کامل می‌توان سرعت و پایداری را هم‌زمان داشت.

### اشتباهات رایج

- **CI بدون تست:** فقط build کردن کافی نیست؛ CI بی‌معنی است اگر تست نداشته باشید.
- **اجازه دادن به شکستن `main`:** اگر `main` شکسته باشد، همه متوقف می‌شوند. همیشه build سبز (green) نگه دارید.
- **Pipelineهای بسیار کند:** اگر تست ۴۰ دقیقه طول بکشد، تیم منتظر می‌ماند. موازی‌سازی و caching کنید.
- **اشتباه گرفتن Delivery با Deployment:** فکر کردن که CD یعنی حتماً خودکار استقرار می‌شود.
- **نادیده گرفتن DORA:** بدون اندازه‌گیری، نمی‌دانید بهبود یافته‌اید یا نه.

### نکات مهم و ترفندها

- **Fail Fast:** تست‌های سریع و ارزان را اول بگذارید (lint → unit) تا سریع شکست ببینید.
- **سبز نگه داشتن main:** قانون تیم: `main` همیشه deployable.
- **Build Once, Deploy Many:** خروجی build (artifact) را یک بار بسازید و در محیط‌های مختلف (staging/production) همان را مستقر کنید — نه rebuild در هر محیط.
- **اندازه‌گیری DORA:** از ابزارهایی مثل `Four Keys` (Google) برای پایش استفاده کنید.
- **Commit often:** ادغام‌های کوچک مشکلات را کوچک و قابل‌حل نگه می‌دارند.

### خلاصه فصل

CI ادغام مکرر با build/test خودکار است؛ CD (Delivery) آماده‌سازی برای انتشار دستی؛ CD (Deployment) استقرار کاملاً خودکار. چهار شاخص DORA (فراوانی استقرار، Lead Time، نرخ خرابی، زمان بازیابی) بلوغ تیم را می‌سنجند. کلید: main همیشه سبز، Build Once/Deploy Many، و اندازه‌گیری مستمر.

---

## فصل ۱۷: معماری یک Pipeline (مراحل، Jobها و Artifactها)

### مقدمه

یک خط لوله (Pipeline) مجموعه‌ای از **مراحل (Stages)** و **Jobها** است که کد را از کامیت تا استقرار هدایت می‌کند. درک درست معماری Pipeline برای ساخت سیستمی که هم سریع و هم قابل‌اعتماد باشد ضروری است.

در این فصل اجزای Pipeline، مفهوم Artifact، وابستگی بین Jobها، و الگوهای موازی‌سازی را بررسی می‌کنیم.

### مفاهیم اساسی

**اجزای یک Pipeline:**

| جزء | تعریف |
|---|---|
| **Stage** | یک فاز منطقی (مثل build، test، deploy) |
| **Job** | واحد اجرایی در یک stage (مثلاً تست واحد) |
| **Step/Task** | کوچک‌ترین واحد (یک دستور) |
| **Runner/Agent** | ماشینی که Job را اجرا می‌کند |
| **Artifact** | خروجی قابل انتقال بین Stageها (مثل فایل `.tar`، image) |
| **Cache** | دادهٔ قابل بازاستفاده برای سرعت (مثل `node_modules`) |
| **Trigger** | چه زمانی Pipeline اجرا شود (push، PR، schedule) |

**جریان معمول:**

```
Trigger (push به main)
   ↓
[Stage: Build]  → build app → artifact
   ↓
[Stage: Test]   → unit (موازی) + integration
   ↓
[Stage: Package] → docker image / binary
   ↓
[Stage: Deploy] → staging (خودکار) → production (دستی/خودکار)
```

**Artifact vs Cache:**
- **Artifact:** خروجی که می‌خواهید نگه دارید و جابه‌جا کنید (مثلاً image داکر یا فایل `.jar`).
- **Cache:** ورودی‌های تکراری که سرعت می‌دهند (مثلاً دانلود dependencies) — پاک شدنش مشکلی ندارد.

**موازی‌سازی (Parallelism):**
Jobهای مستقل را هم‌زمان اجرا کنید. مثلاً تست واحد، lint و تست امنیت می‌توانند موازی باشند.

### مثال‌های عملی

**۱. ساختار Pipeline مفهومی (GitLab-style):**

```yaml
stages:
  - build
  - test
  - package
  - deploy

build_job:
  stage: build
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

unit_test:
  stage: test
  script: npm test -- --watchAll=false
  needs: [build_job]

lint:
  stage: test
  script: npm run lint
  needs: [build_job]    # موازی با unit_test

package_image:
  stage: package
  script: docker build -t myapp:$CI_COMMIT_SHA .
  needs: [unit_test, lint]

deploy_staging:
  stage: deploy
  script: ./deploy.sh staging
  environment: staging
  needs: [package_image]
```

**۲. انتقال Artifact بین Stage:**

```bash
# در build
tar -czf app.tar.gz dist/
# artifact آپلود می‌شود توسط سیستم CI

# در deploy
# artifact دانلود می‌شود
tar -xzf app.tar.gz
./run.sh
```

**۳. Trigger بر اساس شاخه:**

```yaml
workflow:
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH == "main"'
```

### مقایسه و دیدگاه‌های مختلف

| الگو | مزیت | معایب |
|---|---|---|
| **Linear (خطی)** | ساده، قابل فهم | کند (همهٔ سریال) |
| **Parallel Jobs** | سریع | پیچیدگی هماهنگی |
| **Fan-out/Fan-in** | مقیاس‌پذیر | سخت در دیباگ |
| **Matrix Build** | تست روی چند محیط (OS/نسخه) | مصرف منابع |

**دیدگاه‌ها:**
- تیم‌های کوچک: خطی ساده کافی است.
- تیم‌های بزرگ: موازی‌سازی سنگین و matrix برای پوشش کامل.

**Pipeline به عنوان Code:** همیشه Pipeline را در مخزن (مثل `.gitlab-ci.yml`، `.github/workflows/`) نگه دارید — همانند کد، version-controlled.

### اشتباهات رایج

- **عدم استفاده از Artifact:** rebuild کردن در هر stage (کند و inconsistent).
- **پیچیدگی بیش از حد:** Pipeline ۵۰ stage که کسی نمی‌فهمدش.
- **Jobهای وابسته غلط:** اجرای deploy قبل از تست به خاطر اشتباه در `needs`.
- **فراموش کردن cleanup:** artifactهای بی‌پایان فضا پر می‌کنند (`expire_in` بگذارید).
- **Runnerهای اشتراکی بدون isolation:** نشت state بین Jobها.

### نکات مهم و ترفندها

- **`needs` برای گراف اجرا:** به جای ترتیب stage، وابستگی دقیق تعریف کنید تا زودتر شروع شود.
- **Matrix برای چند نسخه:**
  ```yaml
  test:
    matrix:
      - PYTHON: ["3.10", "3.11", "3.12"]
  ```
- **Caching هوشمند:** کلید کش بر اساس hash of lockfile.
- **Secrets در Pipeline:** از متغیرهای محیطی رمزگذاری‌شده (CI secrets) استفاده کنید، هرگز در متن.
- **Pipeline visualization:** از نمودارهای UI برای درک جریان استفاده کنید.
- **Fast feedback loop:** Stage اول (lint) را زیر ۱ دقیقه نگه دارید.

### خلاصه فصل

Pipeline از Stage/Job/Step تشکیل شده، روی Runnerها اجرا می‌شود و Artifact/Cache جابه‌جا می‌کند. Artifact خروجی قابل انتقال است؛ Cache ورودی تکراری برای سرعت. موازی‌سازی و `needs` زمان را کم می‌کنند. کلید: Build Once/Deploy Many، Pipeline-as-Code، و secrets ایمن.

---

## فصل ۱۸: استراتژی‌های Build و Caching

### مقدمه

زمان Build اغلب بزرگ‌ترین عامل کندی Pipeline است. یک build ۲۰ دقیقه‌ای باعث می‌شود توسعه‌دهندگان context خود را گم کنند و بازخورد کند شود. **استراتژی‌های build و caching** مستقیماً روی Lead Time و بهره‌وری اثر می‌گذارند.

در این فصل انواع build (incremental، reproducible، hermetic)، لایه‌های کش (dependency، docker layer، CI cache) و ابزارهای build مدرن را بررسی می‌کنیم.

### مفاهیم اساسی

**انواع Build:**
- **Clean Build:** همه چیز از صفر (قابل اعتماد اما کند).
- **Incremental Build:** فقط تغییرات rebuild شوند (سریع، مثل `make`، `tsc --incremental`).
- **Reproducible Build:** خروجی دقیقاً یکسان با ورودی یکسان (مهم برای امنیت/قابلیت اعتماد).
- **Hermetic Build:** ایزوله از محیط (هیچ وابستگی به خارج) — توسط Bazel.

**لایه‌های Caching:**

| لایه | مثال | ضریب سرعت |
|---|---|---|
| Dependency cache | `node_modules`، `~/.m2` | بالا |
| Docker layer cache | لایه‌های image | بالا |
| CI artifact cache | خروجی بین stage | متوسط |
| Build tool cache | `target/`, `.next/cache` | متوسط |
| Remote cache | Bazel/Nx remote | بالا (تیمی) |

**Content-Addressable Cache:** کلید کش = hash محتوا (مثل lockfile). اگر محتوا عوض شد، کش باطل می‌شود.

**مفهوم Build Matrix / Layering:**
در داکر، دستورات را از کم‌تغییرترین به پرتغییرترین مرتب کنید تا لایه‌ها cached بمانند.

### مثال‌های عملی

**۱. Dockerfile بهینه (لایه‌بندی):**

```dockerfile
# ۱. base (کم‌تغییر)
FROM node:20-alpine AS base
WORKDIR /app

# ۲. dependencies (فقط وقتی package.json عوض شد)
COPY package*.json ./
RUN npm ci --omit=dev

# ۳. source (پرتغییر)
COPY . .
RUN npm run build

# ۴. runtime
CMD ["node", "dist/index.js"]
```

**۲. Caching در CI (GitLab):**

```yaml
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
    - .npm/
before_script:
  - npm ci --cache --prefer-offline
```

**۳. Build incremental با Nx (Monorepo):**

```bash
# فقط پکیج‌های تغییریافته + وابستگانشان
npx nx build myapp --skip-nx-cache=false
# remote cache برای اشتراک بین توسعه‌دهندگان
npx nx affected -t test --base=main
```

**۴. Reproducible build (Go):**

```bash
# خروجی یکسان در هر محیط
go build -trimpath -buildvcs=false -o app .
```

### مقایسه و دیدگاه‌های مختلف

| استراتژی | سرعت | قابلیت اعتماد | پیچیدگی |
|---|---|---|---|
| Clean build | کند | بالا | کم |
| Incremental | سریع | متوسط | متوسط |
| Hermetic (Bazel) | خیلی سریع (با کش) | بالا | بالا |
| Docker layer cache | سریع | بالا | کم |

**دیدگاه‌ها:**
- تیم‌های کوچک: Docker layer cache + dependency cache کافی است.
- تیم‌های بزرگ/مهندسی پلتفرم: Bazel برای reproducibility و remote cache.

**Trade-off:** Hermetic build سرعت و قطعیت می‌دهد اما هزینهٔ یادگیری و نگهداری بالایی دارد.

### اشتباهات رایج

- **COPY . . در اول داکر:** هر تغییر کوچک کل image را rebuild می‌کند (کش از بین می‌رود).
- **کش بدون کلید (key):** کش قدیمی باعث باگ‌های عجیب می‌شود؛ همیشه کلید = hash.
- **`npm install` به جای `npm ci`:** `ci` سریع‌تر و قطعی‌تر (از lockfile).
- **فراموش کردن purge کش:** کش فاسد = build شکست‌خورده مرموز.
- **Build غیرقابل تکرار:** تکیه بر زمان سیستم یا مسیر مطلق → خروجی‌های متفاوت.

### نکات مهم و ترفندها

- **Layer ordering:** در داکر، ثابت‌ترین چیزها را اول بیاورید.
- **`.dockerignore`:** مثل `.gitignore`، فایل‌های غیرضروری را از context حذف کنید (build سریع‌تر).
- **Remote cache:** در Monorepo از Nx Cloud / Bazel Remote اشتراک کش بین تیم.
- **`--mount=type=cache` در BuildKit:**
  ```dockerfile
  RUN --mount=type=cache,target=/root/.npm npm ci
  ```
- **SBOM (Software Bill of Materials):** در build تولید کنید (امنیت — فصل ۲۴).
- **Build timers:** زمان هر stage را لاگ کنید تا گلوگاه پیدا شود.

### خلاصه فصل

زمان Build مستقیماً Lead Time را تعیین می‌کند. استراتژی‌ها: Clean (قابل اعتماد/کند)، Incremental (سریع)، Hermetic/Bazel (سریع + reproducible اما پیچیده). لایه‌بندی داکر (ثابت اول) و dependency cache با کلید hash سرعت را چند برابر می‌کند. کلید: Build Once/Deploy Many، کش با کلید محتوا، و تولید SBOM.

---

## فصل ۱۹: تست در Pipeline (واحد، یکپارچه‌سازی، End-to-End)

### مقدمه

بدون تست، CI یک «build سریع که کد خراب را هم می‌پذیرد» بیش نیست. تست در Pipeline لایهٔ دفاعی است که کیفیت را تضمین می‌کند. اما نه همهٔ تست‌ها برابرند: تست واحد سریع است، E2E کند و شکننده.

در این فصل هرم تست (Testing Pyramid)، انواع تست، و نحوهٔ جای‌دهی آن‌ها در Pipeline را یاد می‌گیرید.

### مفاهیم اساسی

**هرم تست (Test Pyramid):**

```
        /\        E2E (کم، کند، شکننده)
       /  \       Integration (متوسط)
      /----\      Unit (زیاد، سریع، ارزان)
     /______\
```

قانون: تست واحد را زیاد بنویسید، E2E را کم.

**انواع تست:**

| نوع | دامنه | سرعت | هزینه نگهداری |
|---|---|---|---|
| **Unit** | یک تابع/کلاس | میلی‌ثانیه | کم |
| **Integration** | چند ماژول + DB/API | ثانیه‌ای | متوسط |
| **E2E** | کل سیستم از دید کاربر | دقیقه‌ای | بالا |
| **Contract** | قرارداد بین سرویس‌ها | ثانیه‌ای | متوسط |
| **Smoke** | حداقل عملکرد پس از deploy | ثانیه‌ای | کم |

**پوشش (Coverage):** درصد کدی که تست اجرا می‌شود. هدف ۸۰٪+ برای کد حیاتی (اما ۱۰۰٪ وسواس بی‌جا).

**Test Pyramid در DevOps:** تست‌ها باید سریع و در خط لوله اجرا شوند تا بازخورد سریع بدهند.

### مثال‌های عملی

**۱. تست واحد (Python / pytest):**

```python
# tests/test_calc.py
def add(a, b):
    return a + b

def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
```

```bash
pytest tests/ --cov=src --cov-report=term-missing
```

**۲. تست یکپارچه‌سازی (Docker Compose برای DB):**

```yaml
# docker-compose.test.yml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: test
```

```bash
docker compose -f docker-compose.test.yml up -d
pytest tests/integration/
docker compose down
```

**۳. تست E2E (Playwright):**

```typescript
// e2e/login.spec.ts
import { test, expect } from '@playwright/test';
test('user can login', async ({ page }) => {
  await page.goto('/login');
  await page.fill('#user', 'demo');
  await page.fill('#pass', 'secret');
  await page.click('button[type=submit]');
  await expect(page).toHaveURL('/dashboard');
});
```

**۴. جای‌دهی در Pipeline (موازی‌سازی):**

```yaml
test_unit:
  stage: test
  script: pytest tests/unit/ -x
  parallel: 4

test_integration:
  stage: test
  script: ./run_integration.sh

test_e2e:
  stage: test
  when: manual   # گران؛ دستی یا فقط روی main
```

### مقایسه و دیدگاه‌های مختلف

| رویکرد | مزیت | معایب |
|---|---|---|
| Pyramid (واحد سنگین) | سریع، پایدار | نیاز به طراحی خوب |
| Ice Cream Cone (E2E سنگین) | نزدیک به واقعیت | کند، شکننده |
| Trophy (عدم تست UI) | سریع | ریسک UI |

**دیدگاه‌ها:**
- طرفداران Pyramid: تست واحد ارزان‌ترین بازخورد است.
- برخی معتقدند در معماری Microservice، Contract Test از Unit مهم‌تر است (Consumer-Driven Contracts با Pact).

**Flaky tests:** تست‌های گاه‌به‌گاه شکست‌خورده سم هستند — یا درستشان کنید یا حذف. اجازه ندهید «retry بی‌پایان» استاندارد شود.

### اشتباهات رایج

- **E2E در stage اول:** کل pipeline را کند می‌کند؛ E2E را آخر یا دستی بگذارید.
- **تست‌های Flaky را نادیده گرفتن:** اعتماد به pipeline را از بین می‌برد.
- **تست روی دادهٔ واقعی:** همیشه محیط ایزوله (test DB) داشته باشید.
- **پوشش ۱۰۰٪ وسواس:** زمان هدر می‌دهد؛ روی کد حیاتی تمرکز کنید.
- **فراموش کردن تست Performance/امنیت:** فقط functional تست کردن کافی نیست.

### نکات مهم و ترفندها

- **Fail Fast:** lint و unit اول (ثانیه‌ای)، E2E آخر (دقیقه‌ای).
- **Test Containers:** برای یکپارچه‌سازی، DB را در کانتینر ایجاد کنید (Testcontainers).
- **Parallel sharding:** تست‌ها را بین چند runner تقسیم کنید.
- **Coverage gate:** pipeline را fail کنید اگر پوشش زیر آستانه افتاد (مثلاً < 70%).
- **Test Report:** خروجی JUnit XML را به CI بدهید تا نمایش گرافیکی دهد.
- **Mock/Stub:** وابستگی خارجی (API سوم‌شخص) را mock کنید تا تست پایدار بماند.

### خلاصه فصل

تست در Pipeline لایهٔ دفاع کیفیت است. هرم تست: واحد زیاد (سریع)، یکپارچه‌سازی متوسط، E2E کم (کند/شکننده). تست‌های Flaky سم هستند و باید اصلاح یا حذف شوند. جای‌دهی: lint/unit اول (fail fast)، E2E آخر یا دستی. کلید: محیط ایزوله، پوشش منطقی (۸۰٪+)، و گزارش JUnit.

---

## فصل ۲۰: استراتژی‌های استقرار (Blue-Green، Canary، Rolling)

### مقدمه

رسیدن کد به تولید لحظهٔ حساس است. استقرار بدون استراتژی یعنی خاموشی (downtime) و ریسک بالا. **استراتژی‌های استقرار (Deployment Strategies)** راه‌هایی برای انتشار کد جدید با کمترین ریسک و بیشترین در دسترس بودن (availability) هستند.

در این فصل چهار استراتژی اصلی — Rolling، Blue-Green، Canary و Shadow — را با مزایا، معایب و سناریوهای مناسب بررسی می‌کنیم.

### مفاهیم اساسی

**چرا استراتژی مهم است؟**
- **Zero downtime:** کاربر وقفه‌ای نبیند.
- **Rollback سریع:** در صورت خرابی فوراً برگردیم.
- **کاهش Blast Radius:** فقط بخش کوچکی تحت تأثیر قرار گیرد.

**۱. Rolling Update:** نسخهٔ جدید را کم‌کم جایگزین نسخهٔ قدیم می‌کند (مثلاً ۲ از ۱۰ پاد هر بار).

**۲. Blue-Green:** دو محیط یکسان — Blue (فعلی) و Green (جدید). ترافیک را یک‌باره یا تدریجی منحرف می‌کنید. در صورت خرابی، برمی‌گردید به Blue.

**۳. Canary:** نسخهٔ جدید را فقط به **درصد کوچکی از کاربران** (مثلاً ۵٪) می‌دهید. اگر سالم بود، افزایش می‌دهید (۱۰٪، ۵۰٪، ۱۰۰٪).

**۴. Shadow:** نسخهٔ جدید ترافیک را دریافت می‌کند اما پاسخش به کاربر نشان داده نمی‌شود (فقط برای تست بار/رفتار).

**مفهوم `Health Check`:** قبل از ارسال ترافیک، سیستم چک می‌کند سرویس زنده است (`/healthz`).

### مثال‌های عملی

**۱. Rolling Update در Kubernetes:**

```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # حداکثر ۱ پاد اضافه
      maxUnavailable: 0  # هیچ پادی از دسترس خارج نشود
```

**۲. Blue-Green با Load Balancer:**

```bash
# Green آماده شد
kubectl apply -f deploy-green.yaml
# چک سلامت
curl -f http://green/healthz
# سوییچ ترافیک
aws elbv2 modify-listener --listener-arn $L --default-actions TargetGroupArn=green-tg
# در صورت مشکل: برگرداندن به blue-tg
```

**۳. Canary با Argo Rollouts:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
spec:
  strategy:
    canary:
      steps:
        - setWeight: 5
        - pause: { duration: 5m }
        - setWeight: 25
        - pause: { duration: 5m }
        - setWeight: 100
```

**۴. برچسب‌گذاری نسخه برای Rollback:**

```bash
kubectl rollout history deployment/myapp
kubectl rollout undo deployment/myapp --to-revision=2
```

### مقایسه و دیدگاه‌های مختلف

| استراتژی | Downtime | سرعت برگشت | هزینه منابع | پیچیدگی |
|---|---|---|---|---|
| Rolling | صفر (با تنظیم) | متوسط | کم | کم |
| Blue-Green | صفر | فوری | ۲x (دو محیط) | متوسط |
| Canary | صفر | سریع (وزن کم) | کم | بالا |
| Shadow | صفر | n/a | ۲x | بالا |

**کدام را انتخاب کنیم؟**
- **Rolling:** پیش‌فرض ساده در K8s.
- **Blue-Green:** وقتی rollback فوری حیاتی است و بودجهٔ ۲x دارید.
- **Canary:** برای تغییرات پرریسک با نظارت دقیق (SLO — فصل ۲۳).
- **Shadow:** برای تست نسخهٔ جدید روی ترافیک واقعی بدون ریسک.

**دیدگاه DevOps:** Canary همراه با Service Mesh و مانیتورینگ SLO بهترین تعادل ریسک/سرعت را می‌دهد.

### اشتباهات رایج

- **استقرار بدون Health Check:** ترافیک به پاد نیمه‌آماده می‌رود → خرابی.
- **Blue-Green بدون همگام‌سازی DB:** نسخهٔ جدید schema قدیمی را نمی‌فهمد.
- **Canary بدون مانیتورینگ:** ۵٪ کاربر خراب می‌شوند و کسی نمی‌فهمد.
- **فراموش کردن Rollback plan:** «اگر خراب شد چه؟» باید قبل از استقرار جواب داشته باشید.
- **غفلت از Database Migration:** کد جدید اغلب نیاز به migration دارد (رویکرد expand/contract).

### نکات مهم و ترفندها

- **Database expand/contract:** ابتدا schema را «بسط» (ستون جدید اضافه، بدون حذف قدیمی)، کد جدید را مستقر کنید، سپس ستون قدیمی را «انقباض» (حذف) کنید — بدون خرابی.
- **Automated Rollback:** اگر خطای ۵xx > آستانه، خودکار برگردانید.
- **Feature Flag + Canary:** با پرچم، کد را در ۱۰۰٪ پخش کنید اما ویژگی را برای ۵٪ فعال کنید.
- **Smoke test پس از deploy:** یک درخواست `/healthz` یا E2E کوتاه.
- **Progressive Delivery:** Canary + Automation (Argo Rollouts / Flagger) برای انتشار تدریجی هوشمند.
- **حتماً نسخه را برچسب بزنید:** هر image باید tag منحصر‌به‌فرد (SHA) داشته باشد، نه `latest`.

### خلاصه فصل

استراتژی استقرار ریسک انتشار را کنترل می‌کند. Rolling (ساده)، Blue-Green (rollback فوری، ۲x منابع)، Canary (درصدی، نیازمند مانیتورینگ) و Shadow (تست بی‌خطر). قانون: همیشه Health Check داشته باشید، Database Migration با رویکرد expand/contract، و Rollback Plan از پیش آماده. Canary + Service Mesh + SLO بهترین تعادل است.

---

## منابع و مراجع

- Continuous Delivery (Jez Humble, David Farley)
- The DevOps Handbook (Gene Kim et al.)
- DORA Metrics / State of DevOps Report: https://dora.dev
- Google Four Keys: https://github.com/googleapis/go-genai (fourkeys project)
- Kubernetes Deployment Strategies: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
- Argo Rollouts: https://argoproj.github.io/rollouts/
- Test Pyramid (Martin Fowler): https://martinfowler.com/articles/practical-test-pyramid.html
- Pact (Contract Testing): https://pact.io
- Playwright: https://playwright.dev
- Bazel: https://bazel.build

**بعدی:** [فایل ۰۵ — Jenkins: استاد خطوط لوله](05-Jenkins.md)
