# بخش ۱۳: الگوهای CI/CD

**مدت یادگیری**: ۵ تا ۶ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: بخش ۱۰–۱۲ (Actions، امنیت)

---

## فهرست محتوا

- [فصل ۵۷: طراحی خط CI — از lint تا build](#فصل-۵۷)
- [فصل ۵۸: ساخت و انتشار Docker image](#فصل-۵۸)
- [فصل ۵۹: استقرار — از SSH تا Kubernetes](#فصل-۵۹)
- [فصل ۶۰: انتشار خودکار نسخه و changelog](#فصل-۶۰)
- [فصل ۶۱: کیفیت و سرعت CI — زمان، flaky، هزینه](#فصل-۶۱)

---

## فصل ۵۷: طراحی خط CI — از lint تا build

### مقدمه

CI فقط «اجرای تست» نیست. یک خط CI خوب مثل یک صافی چندلایه است: هر لایه چیزی را می‌گیرد که لایه‌ی قبلی نگرفته، و ارزان‌ترین لایه اول اجرا می‌شود.

**چرا مهم است؟** ترتیب اشتباه = CI کند. اگر تست ۱۰ دقیقه‌ای را قبل از lint ۱۰ ثانیه‌ای اجرا کنی، هر خطای فرمت ۱۰ دقیقه وقت می‌گیرد.

**کاربرد عملی:** این فصل یک خط CI بهینه و مقیاس‌پذیر طراحی می‌کند.

### مفاهیم اساسی

#### اصل «ارزان اول»

```
۱. Syntax / YAML / فرمت        (ثانیه)
۲. Lint                         (۱۰ ثانیه)
۳. Type check                   (۳۰ ثانیه)
۴. Unit test                    (۱-۲ دقیقه)
۵. Integration test             (۳-۵ دقیقه)
۶. Build                        (۱-۳ دقیقه)
۷. E2E test                     (۵-۱۵ دقیقه)
۸. Deploy                       (متغیر)
```

هر لایه اگر شکست خورد، لایه‌های بعدی اجرا نشوند (یا فقط برای دیباگ).

#### اصل «سریع بازخورد بده»

| هدف | زمان |
|---|---|
| lint/format | < ۳۰ ثانیه |
| unit test | < ۲ دقیقه |
| کل CI روی PR | < ۱۰ دقیقه |
| E2E | < ۲۰ دقیقه (یا فقط روی main) |

اگر CI بیشتر از ۱۵ دقیقه است، توسعه‌دهنده‌ها آن را دور می‌زنند.

#### مرز CI و CD

| | CI | CD |
|---|---|---|
| چه | lint، test، build | deploy |
| کِی | هر PR | بعد از merge |
| هدف | اطمینان | تحویل |
| ریسک | کم | بالا (نیاز به approval) |

### مثال‌های عملی

#### مثال ۱: خط CI چندلایه

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
  merge_group:

permissions:
  contents: read

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}

jobs:
  # لایه ۱: سریع‌ترین
  static:
    name: static
    runs-on: ubuntu-24.04
    timeout-minutes: 5
    steps:
      - uses: actions/checkout@<sha>
      - uses: actions/setup-node@<sha>
        with: { node-version: "22", cache: npm }
      - run: npm ci
      - run: npm run format:check
      - run: npm run lint
      - run: npm run typecheck

  # لایه ۲: تست
  test:
    name: test
    needs: static
    runs-on: ubuntu-24.04
    timeout-minutes: 10
    strategy:
      fail-fast: false
      matrix:
        shard: [1, 2]
    steps:
      - uses: actions/checkout@<sha>
      - uses: actions/setup-node@<sha>
        with: { node-version: "22", cache: npm }
      - run: npm ci
      - run: npm test -- --shard=${{ matrix.shard }}/2 --coverage
      - uses: actions/upload-artifact@<sha>
        if: always()
        with:
          name: coverage-${{ matrix.shard }}
          path: coverage/

  # لایه ۳: build
  build:
    name: build
    needs: static
    runs-on: ubuntu-24.04
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@<sha>
      - uses: actions/setup-node@<sha>
        with: { node-version: "22", cache: npm }
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@<sha>
        with:
          name: dist
          path: dist/
          retention-days: 7

  # لایه ۴: E2E فقط روی main (کند)
  e2e:
    name: e2e
    if: github.ref == 'refs/heads/main'
    needs: [test, build]
    runs-on: ubuntu-24.04
    timeout-minutes: 20
    steps:
      - uses: actions/checkout@<sha>
      - uses: actions/download-artifact@<sha>
        with: { name: dist, path: dist/ }
      - run: npx playwright install --with-deps
      - run: npx playwright test
      - uses: actions/upload-artifact@<sha>
        if: always()
        with: { name: playwright-report, path: playwright-report/ }

  # دروازه
  ci-success:
    name: ci-success
    needs: [static, test, build]
    if: always()
    runs-on: ubuntu-24.04
    steps:
      - run: |
          if echo "${{ join(needs.*.result, ' ') }}" | grep -qE 'failure|cancelled'; then
            echo "::error::CI شکست خورد"
            exit 1
          fi
```

**نکته:** `build` و `test` هر دو به `static` وابسته‌اند و موازی اجرا می‌شوند.

#### مثال ۲: مرحله‌ی بررسی پیام commit

```yaml
  commitlint:
    name: commitlint
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
        with:
          fetch-depth: 0
      - uses: actions/setup-node@<sha>
        with: { node-version: "22", cache: npm }
      - run: npm ci
      - run: npx commitlint --from ${{ github.event.pull_request.base.sha }} --to ${{ github.event.pull_request.head.sha }} --verbose
```

#### مثال ۳: بررسی لینک‌های مستندات

```yaml
  docs:
    name: docs
    if: contains(github.event.pull_request.changed_files, '')  # همیشه
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
      - uses: lycheeverse/lychee-action@<sha>
        with:
          args: --verbose --no-progress '**/*.md'
          fail: true
```

#### مثال ۴: بررسی امنیت وابستگی

```yaml
  audit:
    name: audit
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
      - uses: actions/setup-node@<sha>
        with: { node-version: "22", cache: npm }
      - run: npm ci
      - run: npm audit --audit-level=high
```

**توجه:** `npm audit` گاهی false positive دارد. `--audit-level=high` نویز را کم می‌کند.

#### مثال ۵: CI برای مونوریپو (فقط پکیج‌های تغییر‌یافته)

```yaml
jobs:
  changes:
    runs-on: ubuntu-24.04
    outputs:
      api: ${{ steps.filter.outputs.api }}
      web: ${{ steps.filter.outputs.web }}
      shared: ${{ steps.filter.outputs.shared }}
    steps:
      - uses: actions/checkout@<sha>
      - uses: dorny/paths-filter@<sha>
        id: filter
        with:
          filters: |
            api: ['packages/api/**', 'packages/shared/**']
            web: ['packages/web/**', 'packages/shared/**']
            shared: ['packages/shared/**']

  api:
    needs: changes
    if: needs.changes.outputs.api == 'true'
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
      - run: pnpm --filter api test

  web:
    needs: changes
    if: needs.changes.outputs.web == 'true'
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
      - run: pnpm --filter web test
```

#### مثال ۶: matrix با OS فقط برای کتابخانه

```yaml
  cross-platform:
    if: github.ref == 'refs/heads/main'
    strategy:
      matrix:
        os: [ubuntu-24.04, windows-2022, macos-14]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@<sha>
      - run: npm ci
      - run: npm test
```

**چرا فقط روی main؟** PRهای معمولی روی Ubuntu کافی است؛ سازگاری کامل روی main تست می‌شود (صرفه‌جویی ۹۰٪ دقیقه).

#### مثال ۷: گزارش وضعیت در PR

```yaml
  report:
    needs: [test, build]
    if: always() && github.event_name == 'pull_request'
    runs-on: ubuntu-24.04
    permissions:
      pull-requests: write
    steps:
      - uses: actions/github-script@<sha>
        with:
          script: |
            const ok = "${{ needs.test.result }}" === "success";
            const icon = ok ? "✅" : "❌";
            await github.rest.issues.createComment({
              ...context.repo,
              issue_number: context.issue.number,
              body: `${icon} **CI**: test=${"${{ needs.test.result }}"}، build=${"${{ needs.build.result }}"}`,
            });
```

### مقایسه و الگوهای مختلف

#### CI روی هر push در برابر فقط PR

| | هر push | فقط PR |
|---|---|---|
| مصرف دقیقه | زیاد | کم |
| بازخورد | فوری | فوری |
| مناسب | تیم کوچک | تیم بزرگ |

**توصیه:** PR + `push` روی `main`. push روی branchهای feature لازم نیست (PR پوشش می‌دهد).

#### تست کامل در برابر تست انتخابی

| | کامل | انتخابی (paths-filter) |
|---|---|---|
| اطمینان | ✅ | متوسط |
| سرعت | کند | ✅ |
| خطر | — | تست ناکافی |

**راه میانه:** روی PR انتخابی، روی `main` کامل.

#### یک workflow در برابر چند

| | یک فایل | چند فایل |
|---|---|---|
| خوانایی | ❌ (بلند) | ✅ |
| کش | مشترک | مشترک |
| مدیریت | ساده | نیاز به نظم |

**توصیه:** `ci.yml` (تست)، `deploy.yml`، `release.yml`، `security.yml`.

### اشکال شایع

#### ۱. CI که ۳۰ دقیقه طول می‌کشد

توسعه‌دهنده‌ها دورش می‌زنند. بشکن: shard، cache، موازی، E2E فقط روی main.

#### ۲. اجرای همه‌چیز روی هر push

۱۰ branch × ۵ push در روز × ۱۰ دقیقه = ۵۰۰ دقیقه در روز. فقط PR و main.

#### ۳. تست بدون coverage

coverage را ببین ولی **هدف عددی نگذار** (مردم تست بی‌ارزش می‌نویسند تا عدد برسد). `--cov-fail-under` را محافظه‌کارانه بگذار.

#### ۴. build قبل از test

اگر build ۳ دقیقه است و test ۲ دقیقه، و تست شکست بخورد، ۳ دقیقه تلف شد. test اول (یا موازی).

#### ۵. E2E روی هر PR

E2E کند و flaky است. فقط روی main یا با label `run-e2e`.

#### ۶. CI روی draft PR

```yaml
jobs:
  test:
    if: github.event.pull_request.draft == false
```

صرفه‌جویی خوبی است — draft هنوز آماده نیست.

#### ۷. عدم اجرای CI روی `merge_group`

PR در merge queue گیر می‌کند. `merge_group` را اضافه کن.

### نکات مهم (Tips & Tricks)

- **ارزان اول** — ترتیب لایه‌ها.
- **`needs` برای موازی‌سازی هوشمند.**
- **shard برای تست‌های زیاد.**
- **E2E و cross-platform فقط روی main.**
- **`paths-filter` برای مونوریپو.**
- **`ci-success` job برای required check.**
- **draft PR را نادیده بگیر.**

### خلاصه فصل

- خط CI = صافی چندلایه، ارزان اول.
- هدف: PR زیر ۱۰ دقیقه.
- `needs` موازی‌سازی و ترتیب را کنترل می‌کند.
- E2E/cross-platform فقط روی main.
- `paths-filter` برای مونوریپو.
- `ci-success` برای required check پایدار.

**سؤالات مروری:**
1. چرا lint باید قبل از test اجرا شود؟
2. چرا E2E روی هر PR ایده‌ی بدی است؟
3. چطور در مونوریپو فقط پکیج تغییر‌یافته را تست کنی؟

---

## فصل ۵۸: ساخت و انتشار Docker image

### مقدمه

Docker و GitHub Actions زوج طبیعی‌اند: build در CI، push به registry، deploy. GitHub خودش یک registry دارد (GHCR) که با `GITHUB_TOKEN` کار می‌کند — بدون نیاز به secret جدا.

**چرا مهم است؟** image یکی از اصلی‌ترین artifactهای تولیدی است. build اشتباه = امنیت، حجم، و سرعت بد.

**کاربرد عملی:** این فصل یک pipeline کامل و بهینه برای Docker می‌سازد.

### مفاهیم اساسی

#### registryها

| Registry | آدرس | احراز هویت |
|---|---|---|
| **GHCR** | `ghcr.io` | `GITHUB_TOKEN` |
| Docker Hub | `docker.io` | username + token |
| AWS ECR | `<acct>.dkr.ecr.<region>.amazonaws.com` | OIDC |
| GCR/Artifact Registry | `<region>-docker.pkg.dev` | OIDC |
| Quay | `quay.io` | token |

**GHCR مزیت دارد:** با `GITHUB_TOKEN` کار می‌کند، permission با `packages: write`.

#### tag strategy

| tag | کِی | مثال |
|---|---|---|
| `sha-<short>` | هر build | `sha-a1b2c3d` |
| `latest` | branch main | `latest` |
| `v1.2.3` | release | `v1.2.3` |
| `v1.2` | release | `v1.2` |
| `v1` | release | `v1` |
| `main` | branch main | `main` |
| `pr-42` | PR | `pr-42` |

**قاعده:** حداقل `sha` (تغییرناپذیر) + یک tag خوانا.

#### multi-platform

```yaml
platforms: linux/amd64,linux/arm64
```

نیاز به `docker/setup-qemu-action` برای arm64 روی amd64.

### مثال‌های عملی

#### مثال ۱: build و push به GHCR

```yaml
name: Docker

on:
  push:
    branches: [main]
    tags: ["v*.*.*"]
  pull_request:

permissions:
  contents: read
  packages: write

jobs:
  build:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>

      - uses: docker/setup-buildx-action@<sha>

      - name: Login به GHCR
        if: github.event_name != 'pull_request'
        uses: docker/login-action@<sha>
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: متادیتا
        id: meta
        uses: docker/metadata-action@<sha>
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=semver,pattern={{major}}
            type=sha,prefix=sha-,format=short

      - name: Build و push
        uses: docker/build-push-action@<sha>
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          provenance: true
          sbom: true
```

`docker/metadata-action` خودش tagها و labelها را می‌سازد — استاندارد OCI.

#### مثال ۲: multi-platform

```yaml
      - uses: docker/setup-qemu-action@<sha>
      - uses: docker/setup-buildx-action@<sha>

      - uses: docker/build-push-action@<sha>
        with:
          platforms: linux/amd64,linux/arm64
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

**هزینه:** build multi-platform ۲-۳ برابر کندتر است. فقط برای release یا main.

#### مثال ۳: Dockerfile بهینه

```dockerfile
# syntax=docker/dockerfile:1.7

# ---- مرحله ۱: deps ----
FROM node:22-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci --omit=dev

# ---- مرحله ۲: build ----
FROM node:22-alpine AS build
WORKDIR /app
COPY package.json package-lock.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci
COPY . .
RUN npm run build

# ---- مرحله ۳: runtime ----
FROM node:22-alpine AS runtime
WORKDIR /app
ENV NODE_ENV=production

# کاربر غیر root
RUN addgroup -S app && adduser -S app -G app

COPY --from=deps  /app/node_modules ./node_modules
COPY --from=build /app/dist        ./dist
COPY package.json ./

USER app
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD node -e "fetch('http://localhost:3000/health').then(r=>process.exit(r.ok?0:1)).catch(()=>process.exit(1))"

CMD ["node", "dist/server.js"]
```

**نکات:**

- multi-stage → image کوچک‌تر
- `--mount=type=cache` → کش npm
- `USER app` → غیر root
- `HEALTHCHECK` → orchestrator می‌فهمد
- `.dockerignore` → context کوچک‌تر

#### مثال ۴: `.dockerignore`

```
node_modules
dist
.git
.github
*.md
!README.md
coverage
.env*
!.env.example
Dockerfile*
docker-compose*
```

بدون `.dockerignore`، context شامل `node_modules` می‌شود و build کند می‌شود.

#### مثال ۵: اسکن امنیتی image

```yaml
      - name: اسکن Trivy
        uses: aquasecurity/trivy-action@<sha>
        with:
          image-ref: ghcr.io/${{ github.repository }}:sha-${{ github.sha }}
          format: sarif
          output: trivy.sarif
          severity: CRITICAL,HIGH
          exit-code: "1"

      - name: آپلود به code scanning
        if: always()
        uses: github/codeql-action/upload-sarif@<sha>
        with:
          sarif_file: trivy.sarif
```

#### مثال ۶: push به AWS ECR با OIDC

```yaml
permissions:
  id-token: write
  contents: read

jobs:
  build:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>

      - uses: aws-actions/configure-aws-credentials@<sha>
        with:
          role-to-assume: arn:aws:iam::123:role/GitHubECR
          aws-region: eu-central-1

      - id: ecr
        uses: aws-actions/amazon-ecr-login@<sha>

      - uses: docker/build-push-action@<sha>
        with:
          push: true
          tags: ${{ steps.ecr.outputs.registry }}/myapp:${{ github.sha }}
          cache-from: type=registry,ref=${{ steps.ecr.outputs.registry }}/myapp:cache
          cache-to: type=registry,ref=${{ steps.ecr.outputs.registry }}/myapp:cache,mode=max
```

#### مثال ۷: PR preview با image

```yaml
on:
  pull_request:

jobs:
  preview:
    runs-on: ubuntu-24.04
    permissions:
      contents: read
      packages: write
      pull-requests: write
    steps:
      - uses: actions/checkout@<sha>
      - uses: docker/setup-buildx-action@<sha>
      - uses: docker/login-action@<sha>
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@<sha>
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:pr-${{ github.event.pull_request.number }}
      - uses: actions/github-script@<sha>
        with:
          script: |
            await github.rest.issues.createComment({
              ...context.repo,
              issue_number: context.issue.number,
              body: `image آماده: \`ghcr.io/${context.repo.owner}/${context.repo.repo}:pr-${context.issue.number}\``,
            });
```

#### مثال ۸: پاک کردن imageهای قدیمی

GHCR محدودیت حجم دارد. با `actions/delete-package-versions`:

```yaml
- uses: actions/delete-package-versions@<sha>
  with:
    package-name: myapp
    package-type: container
    min-versions-to-keep: 10
    delete-only-untagged-versions: true
```

#### مثال ۹: بررسی image قبل از deploy

```bash
docker run --rm ghcr.io/myorg/myapp:sha-a1b2c3d --version
docker inspect ghcr.io/myorg/myapp:sha-a1b2c3d | jq '.[0].Config.User'
# باید غیر root باشد
```

### مقایسه و الگوهای مختلف

#### GHCR در برابر Docker Hub

| | GHCR | Docker Hub |
|---|---|---|
| احراز هویت | `GITHUB_TOKEN` | token جدا |
| مخزن خصوصی رایگان | ✅ | محدود |
| rate limit pull | بالا | ۱۰۰/۶ ساعت (ناشناس) |
| یکپارچگی GitHub | ✅ | ❌ |

#### `latest` در برابر `sha`

| | `latest` | `sha-<hash>` |
|---|---|---|
| تغییرناپذیر | ❌ | ✅ |
| خوانایی | ✅ | ❌ |
| rollback | سخت | ✅ |
| برای deploy | ❌ | ✅ |

**قاعده:** `latest` را برای انسان بگذار، `sha` را برای deploy.

#### cache: `gha` در برابر `registry`

| | `type=gha` | `type=registry` |
|---|---|---|
| نیاز به registry | ❌ | ✅ |
| حجم | محدود (۱۰GB) | نامحدود |
| سرعت | ✅ | متوسط |
| بین repo | ❌ | ✅ |

### اشکال شایع

#### ۱. image غول‌پیکر (۱GB+)

- `node_modules` کپی شده
- بدون multi-stage
- base image بزرگ (`node:22` به‌جای `node:22-alpine`)

#### ۲. اجرا با root

```dockerfile
✗ USER root
✓ USER app
```

container با root، در صورت نفوذ، کل host را در خطر می‌گذارد.

#### ۳. secret در layer

```dockerfile
✗ RUN curl -H "Authorization: $TOKEN" ...
```

secret در layer می‌ماند، حتی اگر در `ENV` نباشد. از `--mount=type=secret` استفاده کن:

```dockerfile
RUN --mount=type=secret,id=npmrc \
    npm ci
```

```yaml
- uses: docker/build-push-action@<sha>
  with:
    secrets: npmrc=${{ secrets.NPMRC }}
```

#### ۴. `latest` برای deploy

```yaml
✗ image: ghcr.io/myorg/app:latest
```

rollback ممکن نیست و نمی‌دانی چه کدی در حال اجراست. از `sha` یا semver.

#### ۵. build کند

- cache ندارد
- `.dockerignore` ندارد
- ترتیب layer اشتباه (کد قبل از `npm ci`)

#### ۶. image با معماری اشتباه

```
exec format error
```

روی Apple Silicon build کردی، روی سرور amd64 اجرا می‌کنی. `platforms` را تنظیم کن.

#### ۷. `packages: write` فراموش شد

```
denied: permission_denied: write_package
```

#### ۸. tag تکراری

push با tag موجود، image قبلی را overwrite می‌کند (اگر immutable tag نباشد). GHCR می‌تواند immutable باشد.

### نکات مهم (Tips & Tricks)

- **multi-stage + alpine** — image کوچک.
- **`USER` غیر root** — همیشه.
- **`--mount=type=cache`** در Dockerfile — build سریع.
- **`provenance: true` و `sbom: true`** — زنجیره‌ی تأمین.
- **`sha` برای deploy، `latest` برای انسان.**
- **`.dockerignore`** — context کوچک.
- **Trivy در CI** — اسکن آسیب‌پذیری.

### خلاصه فصل

- GHCR با `GITHUB_TOKEN` کار می‌کند (`packages: write`).
- `docker/metadata-action` tagها را استاندارد می‌سازد.
- multi-stage + `USER` غیر root + `HEALTHCHECK`.
- cache `type=gha` ساده‌ترین است.
- `sha` برای deploy، `latest` نه.
- Trivy + SBOM + provenance برای امنیت.

**سؤالات مروری:**
1. چرا `latest` برای deploy مناسب نیست؟
2. چطور secret را در build Docker بدون گذاشتن در layer استفاده کنی؟
3. چرا `USER` غیر root مهم است؟

---

## فصل ۵۹: استقرار — از SSH تا Kubernetes

### مقدمه

استقرار (deploy) مرحله‌ای است که کد را از مخزن به محیط اجرا می‌برد. روش‌های زیادی وجود دارد و انتخاب اشتباه می‌تواند امنیت یا سرعت را نابود کند.

**چرا مهم است؟** deploy پرخطرترین بخش pipeline است: اشتباه در آن یعنی downtime. و اگر ناامن باشد، مهاجم می‌تواند از CI به production برسد.

**کاربرد عملی:** چهار روش رایج، با مزایا و معایب و کد کامل.

### مفاهیم اساسی

#### چهار روش اصلی

| روش | پیچیدگی | امنیت | مناسب |
|---|---|---|---|
| **SSH به سرور** | کم | متوسط | VPS، پروژه‌ی کوچک |
| **PaaS** (Vercel، Netlify، Fly) | کم | ✅ | فرانت/اپ کوچک |
| **Kubernetes** | زیاد | ✅ | تیم بالغ |
| **Cloud-native** (ECS، Cloud Run، App Service) | متوسط | ✅ | اکثر پروژه‌ها |

#### اصل: deploy از artifact، نه از کد

```
✓ build → artifact/image → deploy همان artifact
✗ deploy → build روی سرور
```

چرا؟ چون چیزی که تست شد، همان چیزی است که deploy می‌شود.

#### environment و approval

هر deploy به production باید:

1. از `main` باشد
2. approval داشته باشد
3. secret جدا داشته باشد

(فصل ۵۰)

### مثال‌های عملی

#### مثال ۱: SSH با rsync

```yaml
name: Deploy

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-24.04
    environment:
      name: production
      url: https://example.com
    steps:
      - uses: actions/checkout@<sha>

      - uses: actions/setup-node@<sha>
        with: { node-version: "22", cache: npm }

      - run: npm ci && npm run build

      - name: تنظیم SSH
        env:
          SSH_KEY: ${{ secrets.SSH_KEY }}
          KNOWN_HOSTS: ${{ secrets.SSH_KNOWN_HOSTS }}
        run: |
          mkdir -p ~/.ssh
          echo "$SSH_KEY" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          echo "$KNOWN_HOSTS" > ~/.ssh/known_hosts
          chmod 644 ~/.ssh/known_hosts

      - name: rsync
        run: |
          rsync -az --delete \
            -e "ssh -i ~/.ssh/deploy_key -o StrictHostKeyChecking=yes" \
            dist/ deploy@server:/var/www/app/

      - name: reload
        run: |
          ssh -i ~/.ssh/deploy_key deploy@server \
            "sudo systemctl reload nginx"
```

**نکات امنیتی:**

- `known_hosts` را از secret بده (جلوگیری از MITM)
- `StrictHostKeyChecking=yes`
- کاربر `deploy` با دسترسی محدود
- کلید جداگانه برای deploy (نه کلید شخصی)

#### مثال ۲: Docker Compose روی سرور

```yaml
      - name: deploy با compose
        env:
          IMAGE_TAG: ${{ github.sha }}
          REGISTRY_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          ssh -i ~/.ssh/deploy_key deploy@server bash -s <<EOF
            set -euo pipefail
            cd /opt/app
            echo "$REGISTRY_TOKEN" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
            export IMAGE_TAG="$IMAGE_TAG"
            docker compose pull
            docker compose up -d --remove-orphans
            docker image prune -f
          EOF
```

#### مثال ۳: Kubernetes

```yaml
jobs:
  deploy:
    runs-on: ubuntu-24.04
    environment: production
    permissions:
      contents: read
      id-token: write
    steps:
      - uses: actions/checkout@<sha>

      - uses: aws-actions/configure-aws-credentials@<sha>
        with:
          role-to-assume: arn:aws:iam::123:role/GitHubEKS
          aws-region: eu-central-1

      - run: aws eks update-kubeconfig --name prod-cluster --region eu-central-1

      - name: deploy
        run: |
          kubectl set image deployment/app \
            app=ghcr.io/myorg/app:${{ github.sha }} \
            -n production
          kubectl rollout status deployment/app -n production --timeout=5m

      - name: rollback در صورت شکست
        if: failure()
        run: kubectl rollout undo deployment/app -n production
```

**`rollout status`** حیاتی است: بدون آن، deploy «موفق» است حتی اگر pod بالا نیاید.

#### مثال ۴: Cloud Run (GCP)

```yaml
jobs:
  deploy:
    runs-on: ubuntu-24.04
    environment: production
    permissions:
      contents: read
      id-token: write
    steps:
      - uses: actions/checkout@<sha>

      - uses: google-github-actions/auth@<sha>
        with:
          workload_identity_provider: ${{ vars.GCP_WIF }}
          service_account: deploy@project.iam.gserviceaccount.com

      - uses: google-github-actions/deploy-cloudrun@<sha>
        with:
          service: myapp
          image: ghcr.io/myorg/app:${{ github.sha }}
          region: europe-west1
          flags: --allow-unauthenticated
```

#### مثال ۵: Vercel / Netlify

```yaml
      - uses: amondnet/vercel-action@<sha>
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: "--prod"
```

یا ساده‌تر: یکپارچگی بومی Vercel با GitHub (بدون workflow).

#### مثال ۶: deploy با feature flag (canary)

```yaml
jobs:
  canary:
    runs-on: ubuntu-24.04
    environment: production
    steps:
      - name: deploy ۱۰٪ ترافیک
        run: ./deploy.sh --canary 10
      - name: مانیتور ۵ دقیقه
        run: ./monitor.sh --duration 300 --error-threshold 1
      - name: ترویج به ۱۰۰٪
        run: ./deploy.sh --promote
```

**اگر monitor شکست خورد، deploy متوقف می‌شود** (step بعدی اجرا نمی‌شود).

#### مثال ۷: smoke test بعد از deploy

```yaml
      - name: Smoke test
        run: |
          for i in {1..10}; do
            if curl -fsS https://example.com/health | jq -e '.status == "ok"'; then
              echo "سالم"
              exit 0
            fi
            echo "تلاش $i شکست خورد، ۵ ثانیه صبر..."
            sleep 5
          done
          echo "::error::سلامت‌سنجی شکست خورد"
          exit 1
```

#### مثال ۸: rollback خودکار

```yaml
      - name: rollback
        if: failure()
        run: |
          echo "::warning::deploy شکست خورد، rollback به نسخه‌ی قبلی"
          kubectl rollout undo deployment/app -n production
```

#### مثال ۹: deploy بدون downtime (blue-green)

```
۱. نسخه‌ی جدید را روی محیط green بالا بیاور
۲. smoke test روی green
۳. اگر سالم بود، traffic را از blue به green ببر
۴. blue را نگه دار (برای rollback سریع)
```

با Kubernetes: دو Deployment + Service selector. با nginx: دو upstream + `reload`.

### مقایسه و الگوهای مختلف

#### SSH در برابر Kubernetes

| | SSH | K8s |
|---|---|---|
| راه‌اندازی | دقیقه | روز |
| rollback | دستی | `rollout undo` |
| مقیاس | دستی | خودکار |
| امنیت | کلید SSH | RBAC + OIDC |
| مناسب | VPS، پروژه‌ی کوچک | تیم بالغ |

#### Recreate در برابر Rolling در برابر Blue-Green در برابر Canary

| استراتژی | downtime | ریسک | پیچیدگی |
|---|---|---|---|
| Recreate | ✅ دارد | متوسط | کم |
| Rolling | ❌ | متوسط | کم |
| Blue-Green | ❌ | کم | متوسط |
| Canary | ❌ | ✅ کمترین | زیاد |

#### `environment` در برابر بدون

| | با environment | بدون |
|---|---|---|
| approval | ✅ | ❌ |
| secret جدا | ✅ | ❌ |
| audit | ✅ | محدود |
| URL در PR | ✅ | ❌ |

**همیشه environment برای production.**

### اشکال شایع

#### ۱. deploy از branch شخصی

```yaml
✗ on: push  # هر branch
✓ on:
    push:
      branches: [main]
```

#### ۲. کلید SSH شخصی برای deploy

اگر شخص برود، کلید باطل می‌شود (یا بدتر، معتبر می‌ماند). کلید مخصوص deploy با دسترسی محدود.

#### ۳. `StrictHostKeyChecking=no`

```
✗ ssh -o StrictHostKeyChecking=no
```

MITM ممکن است. `known_hosts` را در secret بگذار.

#### ۴. deploy بدون smoke test

«deploy موفق» یعنی «دستور بدون خطا اجرا شد»، نه «اپ کار می‌کند». همیشه health check.

#### ۵. بدون rollback

اگر deploy خراب شد، باید بتوانی در ۱ دقیقه برگردی. rollback را از قبل تست کن.

#### ۶. secret در سرور deploy

```
✗ ssh server "export DB_URL=postgres://..."
```

secret در history shell سرور می‌ماند. از env فایل با مجوز ۶۰۰ یا secret manager.

#### ۷. deploy هم‌زمان

دو workflow هم‌زمان deploy می‌کنند و همدیگر را خراب می‌کنند:

```yaml
concurrency:
  group: deploy-production
  cancel-in-progress: false
```

#### ۸. build روی سرور

```
✗ ssh server "git pull && npm install && npm run build"
```

- وابستگی به وضعیت سرور
- build کند روی سرور production
- نمی‌دانی چه کدی deploy شد

**همیشه artifact.**

#### ۹. فراموش کردن مهاجرت دیتابیس

اگر migration نیاز است، باید قبل از deploy کد جدید اجرا شود — و برگشت‌پذیر باشد.

### نکات مهم (Tips & Tricks)

- **deploy از artifact/image، نه از کد.**
- **environment + approval برای production.**
- **smoke test بعد از deploy.**
- **rollback خودکار در شکست.**
- **`concurrency: cancel-in-progress: false`.**
- **`known_hosts` در secret.**
- **`rollout status` در K8s** — بدون آن، deploy «موفق» است ولی خراب.

### خلاصه فصل

- چهار روش: SSH، PaaS، K8s، cloud-native.
- **از artifact deploy کن، نه از کد.**
- environment + approval + secret جدا برای production.
- smoke test + rollback خودکار.
- `concurrency` برای جلوگیری از deploy هم‌زمان.
- `known_hosts` و `StrictHostKeyChecking=yes`.

**سؤالات مروری:**
1. چرا deploy از artifact بهتر از build روی سرور است؟
2. چرا `rollout status` در K8s حیاتی است؟
3. سه محافظت لازم برای deploy به production چیست؟

---

## فصل ۶۰: انتشار خودکار نسخه و changelog

### مقدمه

انتشار نسخه (release) شامل: تعیین شماره‌ی نسخه، ساخت changelog، تگ زدن، ساخت release در GitHub، و انتشار پکیج. همه‌ی اینها می‌توانند خودکار شوند.

**چرا مهم است؟** انتشار دستی، خطا دارد: شماره‌ی اشتباه، changelog ناقص، تگ فراموش‌شده. خودکارسازی اینها را حذف می‌کند.

**کاربرد عملی:** دو رویکرد: **semantic-release** (کاملاً خودکار از commitها) و **changesets** (فایل تغییرات دستی + نسخه‌بندی خودکار).

### مفاهیم اساسی

#### Semantic Versioning

```
MAJOR.MINOR.PATCH

1.4.2
 │   │   └── رفع باگ (fix)
 │   └────── ویژگی جدید سازگار (feat)
 └────────── تغییر ناسازگار (BREAKING CHANGE)
```

| تغییر | نسخه |
|---|---|
| `fix:` | PATCH +1 |
| `feat:` | MINOR +1، PATCH = 0 |
| `BREAKING CHANGE:` | MAJOR +1، بقیه = 0 |

#### Conventional Commits → نسخه

| پیام | اثر |
|---|---|
| `fix: ...` | patch |
| `feat: ...` | minor |
| `feat!: ...` یا `BREAKING CHANGE:` در body | major |
| `docs:`، `chore:`، `test:`، `ci:` | هیچ |

#### دو رویکرد

| | semantic-release | changesets |
|---|---|---|
| ورودی | commit messages | فایل `.changeset/*.md` |
| خودکار بودن | ✅ کامل | نیمه (فایل دستی) |
| کنترل | کم | ✅ زیاد |
| monorepo | ضعیف | ✅ عالی |
| مناسب | کتابخانه‌ی ساده | monorepo، تیم بزرگ |

### مثال‌های عملی

#### مثال ۱: semantic-release

نصب:

```bash
npm install -D semantic-release \
  @semantic-release/changelog \
  @semantic-release/git \
  @semantic-release/github
```

`.releaserc.json`:

```json
{
  "branches": ["main"],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    ["@semantic-release/changelog", { "changelogFile": "CHANGELOG.md" }],
    ["@semantic-release/npm", { "npmPublish": true }],
    ["@semantic-release/git", {
      "assets": ["CHANGELOG.md", "package.json"],
      "message": "chore(release): ${nextRelease.version} [skip ci]\n\n${nextRelease.notes}"
    }],
    "@semantic-release/github"
  ]
}
```

workflow:

```yaml
name: Release

on:
  push:
    branches: [main]

permissions:
  contents: write
  issues: write
  pull-requests: write
  id-token: write     # برای provenance

jobs:
  release:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
        with:
          fetch-depth: 0          # semantic-release به تاریخ کامل نیاز دارد
          persist-credentials: false

      - uses: actions/setup-node@<sha>
        with:
          node-version: "22"
          cache: npm
          registry-url: "https://registry.npmjs.org"

      - run: npm ci

      - run: npx semantic-release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

**`fetch-depth: 0` حیاتی است** — بدون تاریخ کامل، semantic-release نمی‌تواند commitها را تحلیل کند.

#### مثال ۲: changesets

```bash
npm install -D @changesets/cli
npx changeset init
```

توسعه‌دهنده بعد از هر تغییر:

```bash
npx changeset
# می‌پرسد: کدام پکیج؟ major/minor/patch؟ توضیح؟
# فایل .changeset/abc123.md می‌سازد
```

`.changeset/abc123.md`:

```markdown
---
"@myorg/ui": minor
"@myorg/api": patch
---

افزودن کامپوننت Button و رفع خطای اعتبارسنجی در API
```

workflow:

```yaml
name: Release

on:
  push:
    branches: [main]

permissions:
  contents: write
  pull-requests: write

jobs:
  release:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
        with:
          fetch-depth: 0

      - uses: actions/setup-node@<sha>
        with:
          node-version: "22"
          cache: npm

      - run: npm ci

      - uses: changesets/action@<sha>
        with:
          publish: npm run release
          version: npm run version
          commit: "chore: انتشار نسخه"
          title: "chore: انتشار نسخه"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

`changesets/action` یک PR «Version Packages» می‌سازد. وقتی merge شود، منتشر می‌کند.

#### مثال ۳: ساخت release دستی با gh

```bash
# تگ annotated
git tag -a v1.4.0 -m "نسخه ۱.۴.۰"
git push origin v1.4.0

# release با notes خودکار
gh release create v1.4.0 \
  --title "v1.4.0" \
  --generate-notes \
  --verify-tag

# با فایل
gh release create v1.4.0 dist/*.zip --generate-notes
```

`--generate-notes` از PRها و commitها changelog می‌سازد.

#### مثال ۴: تنظیم دسته‌بندی release notes

`.github/release.yml`:

```yaml
changelog:
  exclude:
    labels:
      - ignore-for-release
      - dependencies
    authors:
      - dependabot
  categories:
    - title: "🚀 ویژگی‌های جدید"
      labels: [enhancement, feature]
    - title: "🐛 رفع باگ"
      labels: [bug, fix]
    - title: "📚 مستندات"
      labels: [documentation]
    - title: "🔧 سایر تغییرات"
      labels: ["*"]
```

حالا `gh release create --generate-notes` یا دکمه‌ی GitHub، changelog دسته‌بندی‌شده می‌سازد.

#### مثال ۵: release از tag

```yaml
name: Release

on:
  push:
    tags: ["v*.*.*"]

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
        with:
          fetch-depth: 0

      - run: npm ci && npm run build

      - uses: softprops/action-gh-release@<sha>
        with:
          generate_release_notes: true
          files: |
            dist/app-linux
            dist/app-macos
            dist/app.exe
          draft: false
          prerelease: ${{ contains(github.ref, '-rc') || contains(github.ref, '-beta') }}
```

#### مثال ۶: بررسی نسخه قبل از انتشار

```yaml
      - name: بررسی اینکه نسخه قبلاً منتشر نشده
        run: |
          NAME=$(node -p "require('./package.json').name")
          VERSION=$(node -p "require('./package.json').version")
          if npm view "$NAME@$VERSION" version 2>/dev/null; then
            echo "::error::نسخه $VERSION قبلاً منتشر شده"
            exit 1
          fi
```

#### مثال ۷: CHANGELOG دستی ولی خودکار

اگر نمی‌خواهی semantic-release، می‌توانی changelog را از PRها بسازی:

```yaml
- uses: actions/github-script@<sha>
  with:
    script: |
      const { data: prs } = await github.rest.pulls.list({
        ...context.repo, state: "closed", base: "main", per_page: 100,
      });
      const merged = prs.filter(p => p.merged_at);
      const lines = merged.map(p => `- ${p.title} (#${p.number}) @${p.user.login}`);
      require("fs").writeFileSync("CHANGELOG-draft.md", lines.join("\n"));
```

### مقایسه و الگوهای مختلف

#### semantic-release در برابر changesets

| | semantic-release | changesets |
|---|---|---|
| ورودی | commit | فایل |
| نسخه | خودکار | خودکار از فایل |
| نیاز به Conventional Commits | ✅ اجباری | ❌ |
| monorepo | ضعیف | ✅ |
| کنترل انسانی | کم | ✅ |
| مناسب | کتابخانه | monorepo |

#### tag annotated در برابر lightweight

| | annotated | lightweight |
|---|---|---|
| متادیتا | ✅ (نویسنده، پیام، تاریخ) | ❌ |
| `git describe` | ✅ | ❌ |
| امضا | ✅ | ❌ |
| توصیه | ✅ برای release | برای تگ موقت |

#### release در برابر tag

| | tag | release |
|---|---|---|
| Git | ✅ | ✅ (به tag وصل) |
| فایل | ❌ | ✅ |
| notes | ❌ | ✅ |
| در UI | کوچک | ✅ برجسته |

### اشکال شایع

#### ۱. `fetch-depth: 1` در semantic-release

```
Error: Cannot read properties of undefined
```

بدون تاریخ کامل، تحلیل commit ممکن نیست. `fetch-depth: 0`.

#### ۲. Conventional Commits رعایت نمی‌شود

semantic-release نسخه را اشتباه می‌زند (یا هیچ). `commitlint` را در CI بگذار.

#### ۳. `[skip ci]` در پیام release

اگر semantic-release کامیت release می‌زند، باید `[skip ci]` داشته باشد — وگرنه حلقه می‌شود.

#### ۴. tag جابه‌جا شده

اگر `v1.0.0` را حذف و دوباره بسازی، کسانی که نسخه‌ی قبلی را نصب کرده‌اند، چیز متفاوتی دارند. **هرگز tag منتشرشده را جابه‌جا نکن** — نسخه‌ی جدید بزن.

#### ۵. نسخه‌ی `0.x`

در `0.x`، semver می‌گوید هر تغییر می‌تواند ناسازگار باشد. semantic-release رفتار خاصی دارد. اگر API پایدار داری، به `1.0.0` برو.

#### ۶. NPM_TOKEN بدون 2FA bypass

اگر حساب npm شما 2FA دارد، token باید «Automation» type باشد، وگرنه publish رد می‌شود.

#### ۷. release بدون تست

اگر release از tag ساخته می‌شود، مطمئن شو CI روی همان tag سبز است:

```yaml
on:
  workflow_run:
    workflows: [CI]
    types: [completed]
    branches: ["v*"]
```

### نکات مهم (Tips & Tricks)

- **Conventional Commits + commitlint** — پایه‌ی همه‌چیز.
- **`fetch-depth: 0`** برای semantic-release.
- **`[skip ci]` در commit release** — جلوگیری از حلقه.
- **`.github/release.yml`** برای changelog دسته‌بندی‌شده.
- **tag annotated** برای release.
- **provenance** در npm publish (`--provenance`) — زنجیره‌ی تأمین.

### خلاصه فصل

- SemVer: MAJOR.MINOR.PATCH از روی نوع commit.
- **semantic-release** کاملاً خودکار از commitها.
- **changesets** برای monorepo و کنترل بیشتر.
- `fetch-depth: 0` الزامی است.
- `[skip ci]` در commit release.
- tag منتشرشده را هرگز جابه‌جا نکن.

**سؤالات مروری:**
1. چرا `fetch-depth: 0` برای semantic-release لازم است؟
2. تفاوت semantic-release و changesets چیست؟
3. چرا جابه‌جا کردن tag منتشرشده خطرناک است؟

---

## فصل ۶۱: کیفیت و سرعت CI — زمان، flaky، هزینه

### مقدمه

CI کند، بدتر از CI نبودن است: توسعه‌دهنده‌ها آن را دور می‌زنند. و CI flaky، اعتماد را نابود می‌کند: «فقط دوباره اجرا کن» تبدیل به عادت می‌شود.

**چرا مهم است؟** زمان CI مستقیماً روی زمان تحویل اثر دارد. یک تیم ۱۰ نفره با CI ۲۰ دقیقه‌ای، روزانه ۳ ساعت منتظر می‌ماند.

**کاربرد عملی:** این فصل یک راهنمای عملی برای سریع‌تر و پایدارتر کردن CI است.

### مفاهیم اساسی

#### هزینه‌ی پنهان CI کند

```
۱۰ توسعه‌دهنده × ۵ PR در روز × ۲۰ دقیقه انتظار
= ۱۰۰۰ دقیقه در روز
= ۱۶ ساعت در روز
= ۲ نفر تمام‌وقت منتظر
```

#### چهار اهرم سرعت

| اهرم | اثر |
|---|---|
| **کش** | ۳۰-۷۰٪ |
| **موازی‌سازی** | ۲-۴× |
| **انتخابی اجرا** | ۵۰-۸۰٪ |
| **کاهش کار** | متغیر |

#### منابع flakiness

| منبع | مثال |
|---|---|
| زمان | `setTimeout`، تاریخ |
| شبکه | API خارجی، DNS |
| ترتیب | تست‌های وابسته |
| وضعیت مشترک | دیتابیس مشترک |
| تصادفی | `Math.random`، UUID |
| محیط | timezone، locale، مسیر |
| منابع | پورت، فایل موقت |

### مثال‌های عملی

#### مثال ۱: اندازه‌گیری زمان

```bash
# زمان هر job
gh run view <run-id> --json jobs --jq '.jobs[] | {name, startedAt, completedAt}'

# زمان کل
gh run list --limit 20 --json displayTitle,createdAt,updatedAt \
  --jq '.[] | "\(.displayTitle): \((.updatedAt|fromdate) - (.createdAt|fromdate))s"'

# کندترین stepها (در لاگ)
gh run view <run-id> --log | grep -E "^##\[group\]" 
```

#### مثال ۲: کش (بزرگ‌ترین برد)

```yaml
- uses: actions/setup-node@<sha>
  with:
    node-version: "22"
    cache: npm
```

فصل ۴۷ کامل.

#### مثال ۳: موازی‌سازی با shard

```yaml
strategy:
  matrix:
    shard: [1, 2, 3, 4]
steps:
  - run: npm test -- --shard=${{ matrix.shard }}/4
```

۴ برابر سریع‌تر برای تست‌های زیاد.

**نکته:** shard باید متعادل باشد. اگر یک shard ۵ دقیقه و دیگری ۳۰ ثانیه طول بکشد، فایده کم است. Jest/Vitest بر اساس زمان تست‌های قبلی تقسیم می‌کنند.

#### مثال ۴: موازی‌سازی jobها

```yaml
jobs:
  lint:      { ... }
  typecheck: { ... }
  test:      { ... }
  # هر سه موازی، بدون needs
```

#### مثال ۵: اجرای انتخابی

```yaml
- uses: dorny/paths-filter@<sha>
  id: filter
  with:
    filters: |
      src: ['src/**']
      docs: ['**/*.md']
```

فصل ۵۷.

#### مثال ۶: فقط تست‌های مربوط به تغییر

```bash
# Jest
npx jest --onlyChanged

# Vitest
npx vitest related --run src/changed-file.ts

# pytest
pytest --testmon
```

**هشدار:** این روش می‌تواند تست‌های لازم را از دست بدهد. روی main کامل اجرا کن.

#### مثال ۷: کاهش کار

| کار | کاهش |
|---|---|
| E2E فقط روی main | ۹۰٪ |
| cross-platform فقط روی main | ۹۰٪ |
| draft PR را نادیده بگیر | ۳۰٪ |
| `--no-coverage` روی PR | ۲۰٪ |
| build فقط اگر test سبز | ۲۰٪ |

#### مثال ۸: تشخیص flaky test

```yaml
- name: اجرای تست ۳ بار برای تشخیص flaky
  if: github.event_name == 'pull_request'
  run: |
    for i in 1 2 3; do
      echo "اجرا $i"
      npm test || { echo "::warning::flaky در اجرا $i"; exit 1; }
    done
```

یا با ابزار:

```bash
# pytest-rerunfailures
pytest --reruns 2 --reruns-delay 1

# Jest retry
jest --retryTimes=2   # در config
```

**هشدار:** retry علائم را پنهان می‌کند. **درست کن، نه retry.**

#### مثال ۹: quarantine تست flaky

```javascript
// تست‌هایی که موقتاً غیرفعالند
const QUARANTINED = ["checkout.spec.ts", "payment.spec.ts"];

// در CI
test.skip(QUARANTINED.some(f => expect.getState().testPath.includes(f)));
```

با یک Issue برای رفع هر کدام.

#### مثال ۱۰: کنترل هزینه

```yaml
jobs:
  expensive:
    if: |
      github.ref == 'refs/heads/main' ||
      contains(github.event.pull_request.labels.*.name, 'run-expensive')
    runs-on: ubuntu-24.04
```

**الگو:** تست سنگین فقط با label `run-expensive` یا روی main.

#### مثال ۱۱: timeout دقیق

```yaml
jobs:
  test:
    timeout-minutes: 12    # زمان نرمال ۵ دقیقه → ۲.۴×
```

خیلی کوتاه = قطع job سالم. خیلی بلند = مصرف بی‌رویه در خطا.

#### مثال ۱۲: کاهش مصرف دقیقه

```yaml
on:
  push:
    branches: [main]      # نه همه‌ی branchها
  pull_request:

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true    # لغو اجرای قدیمی
```

**`cancel-in-progress`** یکی از بزرگ‌ترین صرفه‌جویی‌هاست: ۵ push پشت سر هم = ۱ اجرا (به‌جای ۵).

#### مثال ۱۳: مانیتورینگ زمان CI

```yaml
- name: گزارش زمان
  if: always()
  run: |
    DURATION=$(( $(date +%s) - ${{ github.event.repository.pushed_at && 0 || 0 }} ))
    echo "### ⏱️ زمان CI: ${DURATION}s" >> "$GITHUB_STEP_SUMMARY"
```

یا ابزار: `benchmark-action/github-action-benchmark` برای روند زمان.

### مقایسه و الگوهای مختلف

#### کش در برابر موازی‌سازی در برابر کاهش کار

| اهرم | هزینه | اثر | ریسک |
|---|---|---|---|
| کش | کم | ۳۰-۷۰٪ | کهنگی |
| موازی | متوسط (دقیقه) | ۲-۴× | پیچیدگی |
| کاهش کار | کم | متغیر | پوشش کمتر |

**ترتیب:** کاهش کار → کش → موازی.

#### retry در برابر رفع

| | retry | رفع |
|---|---|---|
| سرعت | ✅ فوری | کند |
| اعتماد | ❌ | ✅ |
| مناسب | موقت (با Issue) | همیشه |

#### تست کامل در برابر فقط تغییر‌یافته

| | کامل | تغییر‌یافته |
|---|---|---|
| اطمینان | ✅ | متوسط |
| سرعت | کند | ✅ |
| مناسب | main | PR |

### اشکال شایع

#### ۱. «فقط دوباره اجرا کن»

اگر تست ۱۰٪ مواقع شکست می‌خورد، تیم یاد می‌گیرد دوباره اجرا کند. بعد یک باگ واقعی هم نادیده گرفته می‌شود. **flaky را در ۴۸ ساعت رفع کن یا quarantine.**

#### ۲. کش کهنه

cache با key ثابت، وابستگی‌های قدیمی می‌دهد. `hashFiles`.

#### ۳. موازی‌سازی که کندتر می‌کند

اگر هر job ۳۰ ثانیه setup دارد و job اصلی ۱ دقیقه است، ۵ job = ۵ × ۱.۵ = ۷.۵ دقیقه دقیقه مصرف (ولی ۱.۵ دقیقه زمان). بستگی به هدف دارد: زمان یا هزینه.

#### ۴. timeout بلند

job گیرکرده ۶۰ دقیقه اجرا می‌شود. `timeout-minutes`.

#### ۵. اجرای CI روی draft PR

۵۰٪ دقیقه‌ها هدر می‌رود:

```yaml
if: github.event.pull_request.draft == false
```

#### ۶. تست E2E روی هر PR

کند و flaky. فقط main یا label.

#### ۷. coverage اجباری ۱۰۰٪

مردم تست بی‌ارزش می‌نویسند. ۷۰-۸۰٪ با تست‌های معنادار بهتر از ۱۰۰٪ با تست‌های الکی است.

#### ۸. بیت‌های گم‌شده

اگر `shard` را ۴ کنی ولی تست‌ها ۳ تاست، یک shard خالی اجرا می‌شود. تعداد را متناسب بگذار.

### نکات مهم (Tips & Tricks)

- **`cancel-in-progress`** — بزرگ‌ترین صرفه‌جویی.
- **کش با `hashFiles`** — نه key ثابت.
- **shard برای تست‌های زیاد.**
- **E2E/cross-platform فقط روی main.**
- **`timeout-minutes` = ۲× زمان نرمال.**
- **flaky را در ۴۸ ساعت رفع یا quarantine کن.**
- **draft PR را نادیده بگیر.**

### خلاصه فصل

- CI کند = تیم دور می‌زند. هدف: PR زیر ۱۰ دقیقه.
- چهار اهرم: کش، موازی، انتخابی، کاهش کار.
- **flaky را retry نکن، رفع کن.**
- `cancel-in-progress` و `timeout-minutes` بزرگ‌ترین صرفه‌جویی‌ها.
- E2E و cross-platform فقط روی main.
- draft PR را نادیده بگیر.

**سؤالات مروری:**
1. چرا retry کردن تست flaky خطرناک است؟
2. سه راه کاهش زمان CI به ترتیب اثر چیست؟
3. چرا `timeout-minutes` خیلی کوتاه هم مشکل است؟

---

**بعدی:** [`14-Releases-Versioning-and-Packages.md`](14-Releases-Versioning-and-Packages.md)

---

## منابع و مراجع

- GitHub Docs: Deploying with GitHub Actions — https://docs.github.com/en/actions/deployment
- GitHub Docs: Publishing Docker images — https://docs.github.com/en/actions/publishing-packages/publishing-docker-images
- GitHub Docs: Managing a release — https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository
- semantic-release — https://semantic-release.gitbook.io/
- changesets — https://github.com/changesets/changesets
- Docker: Multi-stage builds — https://docs.docker.com/build/building/multi-stage/
- Conventional Commits — https://www.conventionalcommits.org
- `docker/metadata-action` — https://github.com/docker/metadata-action
