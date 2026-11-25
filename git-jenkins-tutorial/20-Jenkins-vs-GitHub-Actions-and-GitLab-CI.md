# بخش ۲۰: مقایسه Jenkins با GitHub Actions و GitLab CI

**مدت یادگیری**: ۲ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۱۹ (پروژه Java و Docker CI/CD)

---

## فهرست محتوا

- [فصل ۱: GitHub Actions](#فصل-۱-github-actions)
- [فصل ۲: GitLab CI/CD](#فصل-۲-gitlab-cicd)
- [فصل ۳: مقایسه معماری و هزینه](#فصل-۳-مقایسه-معماری-و-هزینه)
- [فصل ۴: جدول مقایسه و معیار انتخاب](#فصل-۴-جدول-مقایسه-و-معیار-انتخاب)
- [فصل ۵: مهاجرت و هم‌زیستی](#فصل-۵-مهاجرت-و-همزیستی)

---

## فصل ۱: GitHub Actions

### مقدمه

**GitHub Actions** سیستم CI/CD یکپارچه GitHub است که در سال ۲۰۱۹ معرفی شد.

### مفاهیم اساسی

- **Workflow**: فرآیند خودکار (فایل YAML در `.github/workflows/`).
- **Event**: رویدادی که Workflow را فعال می‌کند (push، PR).
- **Job**: واحد کاری.
- **Step**: گام در Job.
- **Runner**: ماشین اجرا (GitHub-hosted یا self-hosted).
- **Action**: کامپوننت قابل استفاده مجدد.

### مثال‌های عملی

`.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - run: pip install -r requirements-dev.txt
      - run: pytest --cov=app
      - run: flake8 app tests
```

### اشکال شایع

- **اشتباه رایج**: Workflowهای طولانی. Jobها را موازی کنید.
- **اشتباه رایج**: استفاده از Actionهای بدون نگهداری.

### نکات مهم (Tips & Tricks)

- بازار Actionها (Marketplace) بسیار غنی است.
- Runnerهای self-hosted برای کارهای سنگین.
- Secrets در تنظیمات مخزن.

### خلاصه فصل

GitHub Actions CI/CD یکپارچه GitHub با YAML است. Workflow، Job و Step اجزای آن‌اند. Marketplace Actionهای زیادی دارد.

---

## فصل ۲: GitLab CI/CD

### مقدمه

**GitLab CI/CD** بخشی از GitLab است و یکی از قدرتمندترین سیستم‌های CI/CD محسوب می‌شود.

### مفاهیم اساسی

- **Pipeline**: تعریف در `.gitlab-ci.yml`.
- **Stage**: مراحل (build، test، deploy).
- **Job**: کار در Stage.
- **Runner**: اجراکننده (shared یا specific).
- **Artifact**: خروجی Job.

### مثال‌های عملی

`.gitlab-ci.yml`:

```yaml
stages:
  - build
  - test
  - deploy

variables:
  PIP_CACHE_DIR: "$CI_PROJECT_DIR/.cache/pip"

build:
  stage: build
  image: python:3.12
  script:
    - pip install -r requirements-dev.txt
    - python -m build
  artifacts:
    paths:
      - dist/

test:
  stage: test
  image: python:3.12
  script:
    - pip install -r requirements-dev.txt
    - pytest --junitxml=report.xml --cov=app
  artifacts:
    reports:
      junit: report.xml

deploy_prod:
  stage: deploy
  script:
    - ./deploy.sh production
  only:
    - main
  when: manual
```

### اشکال شایع

- **اشتباه رایج**: نبود `only`/`rules`. همه Jobها روی همه شاخه‌ها اجرا می‌شوند.
- **اشتباه رایج**: نبود cache. هر بار وابستگی‌ها نصب می‌شوند.

### نکات مهم (Tips & Tricks)

- `rules` انعطاف بیشتری از `only` دارد.
- cache و artifact سرعت را زیاد می‌کنند.
- GitLab Runner را می‌توانید self-host کنید.

### خلاصه فصل

GitLab CI/CD با `.gitlab-ci.yml` تعریف می‌شود. Stage، Job و Runner اجزای آن‌اند. بسیار قدرتمند و یکپارچه با GitLab است.

---

## فصل ۳: مقایسه معماری و هزینه

### مقدمه

هر سه ابزار قوی‌اند؛ تفاوت در معماری و هزینه است.

### مفاهیم اساسی

**Jenkins:**
- خودمیزبان؛ شما سرور و نگهداری را مدیریت می‌کنید.
- رایگان (نرم‌افزار)، اما هزینه زیرساخت و نگهداری دارد.
- انعطاف بی‌نظیر.

**GitHub Actions:**
- ابری (با گزینه self-hosted).
- رایگان تا سقف مشخص (برای مخازن عمومی نامحدود).
- یکپارچه با GitHub.

**GitLab CI:**
- ابری یا self-hosted.
- رایگان تا سقف (دقیقه‌های CI).
- یکپارچه با GitLab.

### مقایسه و الگوهای مختلف

| معیار | Jenkins | GitHub Actions | GitLab CI |
|-------|---------|----------------|-----------|
| میزبانی | خودمیزبان | ابری | هر دو |
| هزینه | زیرساخت | دقیقه‌ای | دقیقه‌ای |
| نگهداری | بالا | صفر | کم |
| انعطاف | بی‌نظیر | بالا | بالا |
| پیکربندی | Groovy/YAML | YAML | YAML |
| اکوسیستم | ۱۸۰۰+ افزونه | Marketplace | Templates |

### اشکال شایع

- **اشتباه رایج**: مقایسه فقط بر اساس هزینه ظاهری. هزینه نگهداری Jenkins را فراموش نکنید.

### نکات مهم (Tips & Tricks)

- برای سازمان با کنترل کامل داده: Jenkins self-hosted.
- برای پروژه متن‌باز: GitHub Actions.
- برای تیم GitLab: GitLab CI.

### خلاصه فصل

Jenkins انعطاف و کنترل کامل می‌دهد اما نگهداری دارد. Actions و GitLab CI ابری و کم‌نگهداری‌اند. انتخاب به نیاز سازمان بستگی دارد.

---

## فصل ۴: جدول مقایسه و معیار انتخاب

### مقدمه

بیایید معیارهای انتخاب را دقیق مرور کنیم.

### مفاهیم اساسی

**جدول مقایسه جامع:**

| معیار | Jenkins | GitHub Actions | GitLab CI |
|-------|---------|----------------|-----------|
| یادگیری | متوسط | آسان | متوسط |
| انعطاف | ★★★★★ | ★★★★ | ★★★★ |
| نگهداری | ★★ | ★★★★★ | ★★★★ |
| یکپارچگی SCM | ★★★ | ★★★★★ | ★★★★★ |
| هزینه کل | متوسط/بالا | پایین | پایین |
| خودمیزبانی | بله | بله (Runner) | بله |
| مناسب متن‌باز | ★★★ | ★★★★★ | ★★★★ |
| مناسب سازمانی | ★★★★★ | ★★★★ | ★★★★★ |
| پلاگین/اکشن | ★★★★★ | ★★★★ | ★★★★ |
| مقیاس‌پذیری | ★★★★ | ★★★★★ | ★★★★★ |

**راهنمای انتخاب:**

- **Jenkins را انتخاب کنید اگر:** کنترل کامل می‌خواهید، سیستم‌های قدیمی دارید، نیاز به انعطاف بی‌نظیر دارید، داده باید داخل سازمان بماند.
- **GitHub Actions را انتخاب کنید اگر:** از GitHub استفاده می‌کنید، پروژه متن‌باز دارید، نگهداری کم می‌خواهید.
- **GitLab CI را انتخاب کنید اگر:** از GitLab استفاده می‌کنید، یکپارچگی کامل DevOps می‌خواهید.

### اشکال شایع

- **اشتباه رایج**: انتخاب بر اساس مد روز. نیاز واقعی را بسنجید.
- **اشتباه رایج**: فراموش کردن هزینه نگهداری.

### نکات مهم (Tips & Tricks)

- در محیط ترکیبی، هم‌زیستی ممکن است.
- مهاجرت هزینه دارد؛ عجولانه نباشید.

### خلاصه فصل

انتخاب ابزار به کنترل، هزینه، یکپارچگی و نیاز سازمان بستگی دارد. هیچ‌کدام مطلقاً بهتر نیست.

---

## فصل ۵: مهاجرت و هم‌زیستی

### مقدمه

مهاجرت بین ابزارها ممکن است لازم شود. همچنین هم‌زیستی چند ابزار ممکن است.

### مفاهیم اساسی

**استراتژی مهاجرت:**
1. Pipeline را مستند کنید.
2. معادل هر مرحله را بنویسید.
3. موازی اجرا کنید (مدت آزمایشی).
4. تدریجی مهاجرت کنید.
5. ابزار قدیمی را بازنشسته کنید.

**هم‌زیستی:**
- Jenkins برای کارهای پیچیده/قدیمی.
- Actions/GitLab برای پروژه‌های جدید.

### مثال‌های عملی

نقشه مهاجرت Jenkins → GitHub Actions:

| Jenkins | GitHub Actions |
|---------|----------------|
| `agent docker` | `runs-on` + container |
| `stage` | `job` |
| `sh 'cmd'` | `run: cmd` |
| `credentials()` | `secrets` |
| `post` | `if: always()` |

### اشکال شایع

- **اشتباه رایج**: مهاجرت کامل یک‌شبه. تدریجی و با تست.
- **اشتباه رایج**: رها کردن Pipelineهای قدیمی بدون مهاجرت.

### نکات مهم (Tips & Tricks)

- مدت آزمایشی موازی اجرا کنید.
- Pipeline را مثل کد مدیریت کنید.
- مستندسازی کنید.

### خلاصه فصل

مهاجرت باید تدریجی و با تست باشد. هم‌زیستی Jenkins و ابزارهای ابری ممکن و منطقی است. Pipeline را مثل کد مدیریت کنید.

---

## منابع و مراجع

- GitHub Actions: [https://docs.github.com/en/actions](https://docs.github.com/en/actions)
- GitLab CI/CD: [https://docs.gitlab.com/ee/ci/](https://docs.gitlab.com/ee/ci/)
- Jenkins: [https://www.jenkins.io/doc/](https://www.jenkins.io/doc/)
- Comparison articles: [https://www.jenkins.io/blog/](https://www.jenkins.io/blog/)
