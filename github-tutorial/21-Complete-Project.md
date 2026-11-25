# بخش ۲۱: پروژه‌ی کامل — از صفر تا مخزن تولیدی

**مدت یادگیری**: ۶ تا ۸ ساعت
**سطح سختی**: پروژه
**پیش‌نیاز**: بخش ۰۱ تا ۲۰

---

## فهرست محتوا

- [فصل ۹۴: طراحی و اسکلت پروژه](#فصل-۹۴)
- [فصل ۹۵: CI/CD کامل — تست، build، انتشار](#فصل-۹۵)
- [فصل ۹۶: امنیت — Dependabot، CodeQL، Secret Scanning، Ruleset](#فصل-۹۶)
- [فصل ۹۷: Copilot و خودکارسازی](#فصل-۹۷)
- [فصل ۹۸: مستندات، release و تحویل](#فصل-۹۸)

---

## فصل ۹۴: طراحی و اسکلت پروژه

### مقدمه

این بخش یک پروژه‌ی واقعی می‌سازد: **«سفارش»** — یک API ساده برای مدیریت سفارش با TypeScript، Fastify، Prisma و PostgreSQL. هدف، کد نیست؛ **مخزن** است.

**چرا مهم است؟** تا اینجا هر قابلیت را جدا دیدی. اینجا همه را در یک مخزن واقعی کنار هم می‌گذاری — همان چیزی که در شرکت انجام می‌شود.

**کاربرد عملی:** بعد از این بخش، یک مخزن داری که می‌توانی به‌عنوان template استفاده کنی.

### مفاهیم اساسی

#### تصمیم‌های قبل از شروع

| تصمیم | انتخاب | چرا |
|---|---|---|
| زبان | TypeScript | type safety |
| فریم‌ورک | Fastify | سریع، ساده |
| ORM | Prisma | migration و type |
| دیتابیس | PostgreSQL 17 | استاندارد |
| تست | Vitest + Playwright | سریع |
| لینت | ESLint + Prettier | استاندارد |
| CI | GitHub Actions | همان‌جا |
| Deploy | Docker + Fly.io/Render | ساده |
| پکیج | npm عمومی | کتابخانه‌ی مشترک |
| استراتژی شاخه | GitHub Flow + Squash | ساده |
| نسخه‌بندی | semantic-release | خودکار |

#### ساختار مخزن

```
safaresh/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml
│   │   ├── release.yml
│   │   ├── deploy.yml
│   │   ├── codeql.yml
│   │   └── automerge.yml
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.yml
│   │   ├── feature_request.yml
│   │   └── config.yml
│   ├── actions/
│   │   └── setup/
│   │       └── action.yml
│   ├── dependabot.yml
│   ├── CODEOWNERS
│   ├── PULL_REQUEST_TEMPLATE.md
│   ├── copilot-instructions.md
│   └── release.yml
├── .devcontainer/
│   └── devcontainer.json
├── prisma/
│   ├── schema.prisma
│   └── migrations/
├── src/
│   ├── app.ts
│   ├── server.ts
│   ├── config.ts
│   ├── routes/
│   │   ├── orders.ts
│   │   └── health.ts
│   ├── services/
│   │   └── orders.ts
│   ├── lib/
│   │   ├── result.ts
│   │   └── logger.ts
│   └── types/
├── tests/
│   ├── unit/
│   └── e2e/
├── docs/
│   ├── architecture.md
│   └── api.md
├── scripts/
│   └── smoke.sh
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── .editorconfig
├── .gitattributes
├── .gitignore
├── .nvmrc
├── CHANGELOG.md
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── LICENSE
├── README.md
├── SECURITY.md
├── package.json
├── tsconfig.json
├── vitest.config.ts
└── eslint.config.js
```

### مثال‌های عملی

#### مثال ۱: ساخت مخزن

```bash
mkdir safaresh && cd safaresh
git init
git config --local user.email "ali@example.com"

# فایل‌های پایه
npx --yes create-ts-project . 2>/dev/null || true
# یا دستی:
npm init -y
```

#### مثال ۲: `package.json`

```json
{
  "name": "safaresh",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "engines": { "node": ">=20.9" },
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "tsc -p tsconfig.build.json",
    "start": "node dist/server.js",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "typecheck": "tsc --noEmit",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:e2e": "playwright test",
    "db:migrate": "prisma migrate deploy",
    "db:generate": "prisma generate",
    "prepare": "husky || true",
    "release": "semantic-release"
  },
  "dependencies": {
    "@fastify/cors": "^10.0.0",
    "@fastify/helmet": "^13.0.0",
    "@fastify/rate-limit": "^10.0.0",
    "@prisma/client": "^6.0.0",
    "fastify": "^5.0.0",
    "pino": "^9.0.0",
    "zod": "^3.23.0"
  },
  "devDependencies": {
    "@commitlint/cli": "^19.0.0",
    "@commitlint/config-conventional": "^19.0.0",
    "@playwright/test": "^1.48.0",
    "@semantic-release/changelog": "^6.0.0",
    "@semantic-release/git": "^10.0.0",
    "@types/node": "^22.0.0",
    "eslint": "^9.0.0",
    "husky": "^9.0.0",
    "lint-staged": "^15.0.0",
    "prettier": "^3.3.0",
    "prisma": "^6.0.0",
    "semantic-release": "^24.0.0",
    "tsx": "^4.19.0",
    "typescript": "^5.6.0",
    "vitest": "^2.1.0"
  }
}
```

#### مثال ۳: `.gitignore`

```gitignore
# وابستگی‌ها
node_modules/

# build
dist/
build/
*.tsbuildinfo

# محیط
.env
.env.local
.env.*.local
!.env.example

# تست
coverage/
playwright-report/
test-results/

# لاگ
logs/
*.log

# سیستم‌عامل
.DS_Store
Thumbs.db

# IDE
.vscode/*
!.vscode/extensions.json
!.vscode/settings.json
.idea/

# Prisma
prisma/*.db
prisma/*.db-journal
```

#### مثال ۴: `.gitattributes`

```gitattributes
* text=auto eol=lf

*.png binary
*.jpg binary
*.ico binary
*.woff2 binary

package-lock.json -diff
*.snap -diff
```

#### مثال ۵: `.editorconfig`

```ini
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 2

[*.md]
trim_trailing_whitespace = false

[Makefile]
indent_style = tab
```

#### مثال ۶: `.nvmrc`

```
22
```

#### مثال ۷: `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "lib": ["ES2023"],
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "exactOptionalPropertyTypes": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": false,
    "sourceMap": true
  },
  "include": ["src/**/*", "tests/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

`tsconfig.build.json`:

```json
{
  "extends": "./tsconfig.json",
  "include": ["src/**/*"],
  "exclude": ["tests", "**/*.test.ts"]
}
```

#### مثال ۸: `eslint.config.js`

```javascript
import js from "@eslint/js";
import tseslint from "typescript-eslint";

export default tseslint.config(
  { ignores: ["dist", "coverage", "node_modules"] },
  js.configs.recommended,
  ...tseslint.configs.strictTypeChecked,
  {
    languageOptions: {
      parserOptions: {
        projectService: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
    rules: {
      "@typescript-eslint/no-explicit-any": "error",
      "@typescript-eslint/no-unused-vars": [
        "error",
        { argsIgnorePattern: "^_" },
      ],
      "@typescript-eslint/consistent-type-imports": "error",
      "no-console": ["warn", { allow: ["error", "warn"] }],
    },
  },
  {
    files: ["tests/**/*"],
    rules: { "@typescript-eslint/no-non-null-assertion": "off" },
  },
);
```

#### مثال ۹: `README.md`

````markdown
# سفارش (Safaresh)

> API ساده و قابل اعتماد برای مدیریت سفارش.

[![CI](https://github.com/OWNER/safaresh/actions/workflows/ci.yml/badge.svg)](https://github.com/OWNER/safaresh/actions/workflows/ci.yml)
[![codecov](https://codecov.io/gh/OWNER/safaresh/branch/main/graph/badge.svg)](https://codecov.io/gh/OWNER/safaresh)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## درباره

«سفارش» یک API برای مدیریت سفارش‌های یک فروشگاه کوچک است:
ثبت سفارش، تغییر وضعیت، و گزارش روزانه.

## شروع سریع

```bash
git clone https://github.com/OWNER/safaresh.git
cd safaresh
npm ci
cp .env.example .env
docker compose up -d db
npm run db:migrate
npm run dev
```

سرور روی `http://localhost:3000` بالا می‌آید.

## مستندات

- [معماری](docs/architecture.md)
- [API](docs/api.md)
- [مشارکت](CONTRIBUTING.md)

## دستورها

| دستور | کار |
|---|---|
| `npm run dev` | توسعه (watch) |
| `npm test` | تست واحد |
| `npm run test:e2e` | تست end-to-end |
| `npm run lint` | لینت |
| `npm run typecheck` | بررسی تایپ |
| `npm run build` | build |

## لایسنس

[MIT](LICENSE)
````

#### مثال ۱۰: `CONTRIBUTING.md`

```markdown
# راهنمای مشارکت

## قبل از شروع
- برای تغییرات بزرگ، اول Issue باز کن.
- [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) را رعایت کن.

## محیط
```bash
npm ci
cp .env.example .env
docker compose up -d db
npm run db:migrate
```

## شاخه
- از `main`: `feat/<توضیح>`، `fix/<توضیح>`، `docs/<توضیح>`
- حداکثر ۵۰ کاراکتر

## Commit
Conventional Commits الزامی است:
```
feat(orders): افزودن endpoint ثبت سفارش
fix(auth): رفع نرمال‌سازی ایمیل
docs: به‌روزرسانی README
```

## قبل از PR
```bash
npm run lint
npm run typecheck
npm test
npm run build
```

## قواعد PR
- حداکثر ۴۰۰ خط تغییر
- تست برای هر رفتار جدید
- توضیح «چرا»، نه فقط «چه»
- یک approve حداقل

## Merge
Squash merge. branch بعد از merge خودکار پاک می‌شود.
```

#### مثال ۱۱: `SECURITY.md`

```markdown
# سیاست امنیتی

## نسخه‌های پشتیبانی‌شده

| نسخه | پشتیبانی |
|---|---|
| 1.x | ✅ |
| < 1.0 | ❌ |

## گزارش آسیب‌پذیری

**در Issue عمومی گزارش نکن.**

از [Private vulnerability reporting](https://github.com/OWNER/safaresh/security/advisories/new) استفاده کن.

## زمان‌بندی
- تأیید دریافت: ۴۸ ساعت
- ارزیابی: ۵ روز کاری
- fix (Critical): ۷ روز
- fix (High): ۳۰ روز

## محدوده
- ✅ کد این مخزن
- ❌ حملات DoS حجمی
- ❌ نسخه‌های EOL
- ❌ سرویس‌های شخص ثالث
```

#### مثال ۱۲: `CODEOWNERS`

```gitignore
*                       @myorg/core-team

/.github/workflows/     @myorg/security-team @myorg/core-team
/.github/CODEOWNERS     @myorg/security-team
/prisma/migrations/     @myorg/dba-team
/docker/                @myorg/platform-team
Dockerfile              @myorg/platform-team
SECURITY.md             @myorg/security-team
```

#### مثال ۱۳: `.github/PULL_REQUEST_TEMPLATE.md`

```markdown
## توضیح
<!-- چه چیزی و چرا -->

## نوع تغییر
- [ ] fix
- [ ] feat
- [ ] breaking change
- [ ] docs
- [ ] refactor
- [ ] chore

## چک‌لیست
- [ ] تست اضافه/به‌روز شد
- [ ] `npm run lint` پاس شد
- [ ] `npm run typecheck` پاس شد
- [ ] `npm test` پاس شد
- [ ] مستندات به‌روز شد
- [ ] خودم diff را کامل خواندم

## Issue
Closes #
```

#### مثال ۱۴: `docs/architecture.md`

````markdown
# معماری

## نمای کلی

```mermaid
flowchart LR
    C[کلاینت] -->|HTTP| A[Fastify API]
    A --> S[Services]
    S --> P[Prisma]
    P --> D[(PostgreSQL)]
    A --> L[Pino Logger]
```

## لایه‌ها

| لایه | مسئولیت | مثال |
|---|---|---|
| `routes/` | HTTP، اعتبارسنجی، پاسخ | `orders.ts` |
| `services/` | منطق کسب‌وکار | `createOrder` |
| `lib/` | ابزار مشترک | `Result`، `logger` |
| `prisma/` | دسترسی داده | `Order` model |

## قاعده‌ها

- `routes` هیچ‌وقت مستقیم به Prisma دست نمی‌زند.
- `services` هیچ‌وقت `req`/`res` نمی‌بیند.
- خطاها با `Result<T, E>` برمی‌گردند، نه `throw` (به‌جز boundary).

## تصمیم‌ها

| تصمیم | جایگزین ردشده | دلیل |
|---|---|---|
| Fastify | Express | ۲-۳× سریع‌تر، schema validation بومی |
| Prisma | Drizzle | migration بالغ‌تر |
| Result type | throw | خطاها صریح‌اند |
````

#### مثال ۱۵: اولین commit

```bash
git add .
git commit -m "chore: اسکلت اولیه‌ی پروژه"

# مخزن در GitHub
gh repo create safaresh --private --source=. --push
```

### مقایسه و الگوهای مختلف

#### Feature-based در برابر layer-based

| | feature-based | layer-based |
|---|---|---|
| ساختار | `features/orders/{route,service,test}` | `routes/`, `services/` |
| مقیاس | ✅ | ❌ |
| شروع | متوسط | ✅ ساده |
| این پروژه | — | ✅ (کوچک) |

#### `private` در برابر `public`

| | private | public |
|---|---|---|
| Actions | سهمیه | ✅ رایگان |
| Code Scanning | نیاز به لایسنس | ✅ رایگان |
| مناسب | محصول | متن‌باز |

### اشکال شایع

#### ۱. شروع با کد، نه با ساختار

اگر ساختار نباشد، بعداً مهاجرت دردناک است. **اول اسکلت.**

#### ۲. `.gitignore` ناقص

`node_modules` commit می‌شود و مخزن ۵۰۰MB. **اول `.gitignore`.**

#### ۳. بدون `CONTRIBUTING.md`

هر کس به سبک خودش. **قواعد را مکتوب کن.**

#### ۴. بدون `SECURITY.md`

گزارش‌دهنده مجبور است Issue عمومی بزند. **کانال بگذار.**

#### ۵. `tsconfig` بدون strict

`strict: true` از روز اول. بعداً مهاجرت سخت است.

#### ۶. تست بدون ساختار

`tests/unit/` و `tests/e2e/` از اول.

### نکات مهم (Tips & Tricks)

- **اول اسکلت، بعد کد.**
- **`strict: true` از روز اول.**
- **`.gitignore` قبل از اولین commit.**
- **`CONTRIBUTING.md` قواعد را مکتوب می‌کند.**
- **`docs/architecture.md` با Mermaid** — تصمیم‌ها را ثبت کن.
- **`CODEOWNERS` از اول** — بعداً اضافه کردن سخت است.

### خلاصه فصل

- تصمیم‌های فنی را قبل از شروع بگیر و مکتوب کن.
- ساختار پوشه، فایل‌های پایه، و مستندات.
- `strict: true`، `.gitignore` کامل، `CONTRIBUTING`، `SECURITY`.
- `CODEOWNERS` برای فایل‌های حساس.
- اولین commit و ساخت مخزن.

**سؤالات مروری:**
1. چرا `CODEOWNERS` را از اول بگذاریم؟
2. چرا `tsconfig` strict از روز اول مهم است؟
3. چرا `docs/architecture.md` تصمیم‌ها را ثبت می‌کند؟

---

## فصل ۹۵: CI/CD کامل — تست، build، انتشار

### مقدمه

این فصل خط CI/CD کامل پروژه را می‌سازد: از lint تا deploy، با cache، matrix، artifact، و release خودکار.

**چرا مهم است؟** CI/CD قلب یک مخزن تولیدی است. بدون آن، هر merge یک ریسک است.

**کاربرد عملی:** این فایل‌ها را می‌توانی در هر پروژه‌ی Node کپی کنی.

### مفاهیم اساسی

#### جریان کامل

```
PR:
  lint → typecheck → test → build → (e2e اگر label)
  ↓
merge به main:
  ci-success → release (semantic-release) → deploy staging
  ↓
tag:
  deploy production (با approval)
```

#### فایل‌ها

| فایل | کار |
|---|---|
| `ci.yml` | lint، typecheck، test، build |
| `release.yml` | semantic-release |
| `deploy.yml` | deploy با environment |
| `automerge.yml` | Dependabot patch |
| `codeql.yml` | اسکن امنیتی |

### مثال‌های عملی

#### مثال ۱: composite action برای setup

`.github/actions/setup/action.yml`:

```yaml
name: Setup
description: نصب Node و وابستگی‌ها

inputs:
  node-version:
    default: "22"

runs:
  using: composite
  steps:
    - uses: actions/setup-node@<sha>
      with:
        node-version: ${{ inputs.node-version }}
        cache: npm

    - shell: bash
      run: npm ci

    - shell: bash
      run: npx prisma generate
```

#### مثال ۲: `ci.yml`

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

env:
  NODE_VERSION: "22"

jobs:
  lint:
    name: lint
    runs-on: ubuntu-24.04
    timeout-minutes: 8
    steps:
      - uses: actions/checkout@<sha>
      - uses: ./.github/actions/setup
      - run: npm run format:check
      - run: npm run lint

  typecheck:
    name: typecheck
    runs-on: ubuntu-24.04
    timeout-minutes: 8
    steps:
      - uses: actions/checkout@<sha>
      - uses: ./.github/actions/setup
      - run: npm run typecheck

  test:
    name: test
    runs-on: ubuntu-24.04
    timeout-minutes: 12
    services:
      postgres:
        image: postgres:17
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test
        ports: ["5432:5432"]
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    env:
      DATABASE_URL: postgres://postgres:postgres@localhost:5432/test
      NODE_ENV: test
      TZ: UTC
    steps:
      - uses: actions/checkout@<sha>
      - uses: ./.github/actions/setup
      - run: npm run db:migrate
      - run: npm test -- --coverage
      - uses: actions/upload-artifact@<sha>
        if: always()
        with:
          name: coverage
          path: coverage/
          retention-days: 7

  build:
    name: build
    needs: [lint, typecheck]
    runs-on: ubuntu-24.04
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@<sha>
      - uses: ./.github/actions/setup
      - run: npm run build
      - uses: actions/upload-artifact@<sha>
        with:
          name: dist
          path: dist/
          retention-days: 7

  e2e:
    name: e2e
    if: contains(github.event.pull_request.labels.*.name, 'run-e2e') || github.ref == 'refs/heads/main'
    needs: [build]
    runs-on: ubuntu-24.04
    timeout-minutes: 20
    services:
      postgres:
        image: postgres:17
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test
        ports: ["5432:5432"]
        options: >-
          --health-cmd pg_isready --health-interval 10s --health-retries 5
    env:
      DATABASE_URL: postgres://postgres:postgres@localhost:5432/test
      NODE_ENV: test
    steps:
      - uses: actions/checkout@<sha>
      - uses: ./.github/actions/setup
      - run: npm run db:migrate
      - run: npx playwright install --with-deps chromium
      - run: npm run test:e2e
      - uses: actions/upload-artifact@<sha>
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 14

  commitlint:
    name: commitlint
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
        with:
          fetch-depth: 0
      - uses: ./.github/actions/setup
      - run: |
          npx commitlint \
            --from ${{ github.event.pull_request.base.sha }} \
            --to ${{ github.event.pull_request.head.sha }} \
            --verbose

  ci-success:
    name: ci-success
    needs: [lint, typecheck, test, build, e2e, commitlint]
    if: always()
    runs-on: ubuntu-24.04
    steps:
      - run: |
          echo "نتایج: ${{ join(needs.*.result, ', ') }}"
          if echo "${{ join(needs.*.result, ' ') }}" | grep -qE 'failure|cancelled'; then
            echo "::error::CI شکست خورد"
            exit 1
          fi
          echo "✅ همه سبز"
```

**نکته:** `e2e` و `commitlint` ممکن است `skipped` شوند (با `if`). `ci-success` با `always()` همه را چک می‌کند و `skipped` را قبول می‌کند.

#### مثال ۳: `release.yml`

```yaml
name: Release

on:
  push:
    branches: [main]

permissions:
  contents: write
  issues: write
  pull-requests: write
  id-token: write

concurrency:
  group: release
  cancel-in-progress: false

jobs:
  release:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
        with:
          fetch-depth: 0
          persist-credentials: false

      - uses: ./.github/actions/setup

      - run: npm test
      - run: npm run build

      - run: npx semantic-release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

`.releaserc.json`:

```json
{
  "branches": ["main"],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    ["@semantic-release/changelog", { "changelogFile": "CHANGELOG.md" }],
    ["@semantic-release/git", {
      "assets": ["CHANGELOG.md", "package.json"],
      "message": "chore(release): ${nextRelease.version} [skip ci]\n\n${nextRelease.notes}"
    }],
    "@semantic-release/github"
  ]
}
```

**نکته:** `private: true` در package.json یعنی semantic-release به npm منتشر نمی‌کند (فقط release در GitHub). اگر کتابخانه است، `private` را بردار و `@semantic-release/npm` اضافه کن.

#### مثال ۴: `deploy.yml`

```yaml
name: Deploy

on:
  push:
    tags: ["v*.*.*"]
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [staging, production]
        default: staging

permissions:
  contents: read
  packages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
      - uses: docker/setup-buildx-action@<sha>
      - uses: docker/login-action@<sha>
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/metadata-action@<sha>
        id: meta
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=semver,pattern={{version}}
            type=sha,prefix=sha-
            type=raw,value=latest,enable=${{ github.ref_type == 'tag' }}
      - uses: docker/build-push-action@<sha>
        with:
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          provenance: true
          sbom: true

  deploy:
    needs: build
    runs-on: ubuntu-24.04
    environment:
      name: ${{ inputs.environment || 'production' }}
      url: ${{ vars.APP_URL }}
    steps:
      - uses: actions/checkout@<sha>

      - name: deploy
        run: |
          echo "deploy به ${{ inputs.environment || 'production' }}"
          # مثال با Fly.io:
          # flyctl deploy --image ghcr.io/${{ github.repository }}:${{ github.ref_name }} \
          #   --app ${{ vars.FLY_APP }}
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}

      - name: smoke test
        run: |
          for i in {1..10}; do
            if curl -fsS "${{ vars.APP_URL }}/health" | grep -q '"status":"ok"'; then
              echo "سالم"
              exit 0
            fi
            echo "تلاش $i..."
            sleep 5
          done
          echo "::error::سلامت‌سنجی شکست خورد"
          exit 1
```

#### مثال ۵: `automerge.yml`

```yaml
name: Dependabot auto-merge

on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  automerge:
    if: github.actor == 'dependabot[bot]'
    runs-on: ubuntu-24.04
    steps:
      - id: meta
        uses: dependabot/fetch-metadata@<sha>
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}

      - if: steps.meta.outputs.update-type == 'version-update:semver-patch'
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

#### مثال ۶: `codeql.yml`

```yaml
name: CodeQL

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: "0 4 * * 1"

permissions:
  contents: read
  security-events: write
  actions: read

jobs:
  analyze:
    runs-on: ubuntu-24.04
    timeout-minutes: 20
    steps:
      - uses: actions/checkout@<sha>
      - uses: github/codeql-action/init@<sha>
        with:
          languages: javascript-typescript
          queries: security-extended
      - uses: github/codeql-action/analyze@<sha>
        with:
          category: "/language:javascript-typescript"
```

#### مثال ۷: `dependabot.yml`

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
    open-pull-requests-limit: 8
    groups:
      minor-and-patch:
        patterns: ["*"]
        update-types: ["minor", "patch"]
    labels: ["dependencies"]
    commit-message:
      prefix: "chore(deps)"

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
    commit-message:
      prefix: "ci"
    groups:
      actions:
        patterns: ["*"]

  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"
    commit-message:
      prefix: "build(docker)"
```

#### مثال ۸: `Dockerfile`

```dockerfile
# syntax=docker/dockerfile:1.7

FROM node:22-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci --omit=dev

FROM node:22-alpine AS build
WORKDIR /app
COPY package.json package-lock.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci
COPY prisma ./prisma
RUN npx prisma generate
COPY . .
RUN npm run build

FROM node:22-alpine AS runtime
WORKDIR /app
ENV NODE_ENV=production

RUN addgroup -S app && adduser -S app -G app

COPY --from=deps  /app/node_modules ./node_modules
COPY --from=build /app/node_modules/.prisma ./node_modules/.prisma
COPY --from=build /app/dist ./dist
COPY --from=build /app/prisma ./prisma
COPY package.json ./

USER app
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD node -e "fetch('http://localhost:3000/health').then(r=>process.exit(r.ok?0:1)).catch(()=>process.exit(1))"

CMD ["node", "dist/server.js"]
```

#### مثال ۹: `docker-compose.yml`

```yaml
services:
  db:
    image: postgres:17
    environment:
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: safaresh
    ports: ["5432:5432"]
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5

  app:
    build: .
    ports: ["3000:3000"]
    environment:
      DATABASE_URL: postgres://postgres:postgres@db:5432/safaresh
      NODE_ENV: development
    depends_on:
      db:
        condition: service_healthy

volumes:
  postgres-data:
```

#### مثال ۱۰: بررسی خط CI

```bash
# اجرای محلی
npm run lint && npm run typecheck && npm test && npm run build

# در CI
gh pr checks 42 --watch
gh run view --log-failed
```

### مقایسه و الگوهای مختلف

#### یک workflow در برابر چند

| | یک فایل | چند فایل |
|---|---|---|
| مدیریت | ساده | ✅ واضح |
| خوانایی | ❌ (بلند) | ✅ |
| کش | مشترک | مشترک |

**توصیه:** `ci.yml`، `release.yml`، `deploy.yml`، `codeql.yml` جدا.

#### `services` در برابر container

| | services | container |
|---|---|---|
| کجا | کنار job | داخل job |
| شبکه | localhost | localhost |
| مناسب | دیتابیس | محیط کامل |

#### semantic-release در برابر manual

| | semantic | manual |
|---|---|---|
| خودکار | ✅ | ❌ |
| کنترل | کم | ✅ |
| changelog | ✅ | ❌ |

### اشکال شایع

#### ۱. `e2e` روی هر PR

کند و flaky. فقط با label یا روی main.

#### ۲. `ci-success` که `skipped` را failure می‌بیند

`skipped` قبول است. فقط `failure` و `cancelled` را رد کن.

#### ۳. `services` بدون healthcheck

job قبل از آماده شدن دیتابیس اجرا می‌شود و شکست می‌خورد. `--health-cmd`.

#### ۴. `TZ` تنظیم نشده

تست‌های تاریخ در CI (UTC) و محلی (تهران) متفاوت. `TZ: UTC`.

#### ۵. `DATABASE_URL` در تست

باید به `localhost` اشاره کند (نه `db`)، چون از host اجرا می‌شود.

#### ۶. `prisma generate` فراموش شد

```
@prisma/client did not initialize yet
```

در `setup` action بگذار.

#### ۷. `fetch-depth: 0` در semantic-release

بدون آن، تحلیل commit ممکن نیست.

#### ۸. `GITHUB_TOKEN` برای trigger release

اگر release باید deploy را trigger کند، `GITHUB_TOKEN` trigger نمی‌کند. از GitHub App یا PAT.

#### ۹. `private: true` و semantic-release/npm

اگر `private` باشد و `@semantic-release/npm` هم داشته باشی، خطا. یکی را بردار.

### نکات مهم (Tips & Tricks)

- **composite action برای setup** — DRY.
- **`ci-success`** برای required پایدار.
- **`TZ: UTC`** در تست.
- **`services` با healthcheck.**
- **`e2e` فقط با label یا main.**
- **`concurrency` برای لغو اجراهای قدیمی.**
- **smoke test بعد از deploy.**

### خلاصه فصل

- CI: lint → typecheck → test → build → e2e (شرطی) → ci-success.
- Release: semantic-release با `fetch-depth: 0`.
- Deploy: Docker + GHCR + environment + smoke test.
- Dependabot با automerge patch.
- CodeQL هفتگی و روی PR.
- `ci-success` ruleset را پایدار می‌کند.

**سؤالات مروری:**
1. چرا `ci-success` job لازم است؟
2. چرا `TZ: UTC` در تست مهم است؟
3. چرا `fetch-depth: 0` برای semantic-release لازم است؟

---

## فصل ۹۶: امنیت — Dependabot، CodeQL، Secret Scanning، Ruleset

### مقدمه

این فصل لایه‌های امنیتی را روی مخزن فعال می‌کند: از هشدار وابستگی تا محافظت از branch و تگ.

**چرا مهم است؟** یک مخزن بدون این لایه‌ها، در عمل «کد است که کسی مراقبش نیست».

**کاربرد عملی:** چک‌لیست امنیتی که در هر مخزن جدید تکرار می‌شود.

### مفاهیم اساسی

#### پنج لایه

```
۱. Secret Scanning + Push Protection   ← جلوگیری از نشت
۲. Dependabot alerts + updates         ← وابستگی آسیب‌پذیر
۳. CodeQL / Code Scanning              ← آسیب‌پذیری در کد
۴. Rulesets                            ← حاکمیت (review، CI، امضا)
۵. Actions security                    ← permissions، pinning، allow-list
```

### مثال‌های عملی

#### مثال ۱: فعال‌سازی همه با gh

```bash
REPO="OWNER/safaresh"

# Dependabot
gh api -X PUT "repos/$REPO/vulnerability-alerts"
gh api -X PUT "repos/$REPO/automated-security-fixes"

# Secret scanning + push protection
gh api -X PATCH "repos/$REPO" \
  -f 'security_and_analysis[secret_scanning][status]=enabled' \
  -f 'security_and_analysis[secret_scanning_push_protection][status]=enabled'

# Actions: فقط read
gh api -X PUT "repos/$REPO/actions/permissions/workflow" \
  -f default_workflow_permissions=read \
  -F can_approve_pull_request_reviews=false

# Actions: allow-list
gh api -X PUT "repos/$REPO/actions/permissions" \
  -f enabled=true -f allowed_actions=selected

gh api -X PUT "repos/$REPO/actions/permissions/selected-actions" \
  -f github_owned_allowed=true \
  -f verified_allowed=true \
  -f 'patterns_allowed[]=actions/*' \
  -f 'patterns_allowed[]=docker/*'
```

#### مثال ۲: Ruleset برای main

```bash
gh api -X POST "repos/$REPO/rulesets" --input - <<'JSON'
{
  "name": "protect-main",
  "target": "branch",
  "enforcement": "active",
  "conditions": {
    "ref_name": { "include": ["~DEFAULT_BRANCH"], "exclude": [] }
  },
  "rules": [
    { "type": "deletion" },
    { "type": "non_fast_forward" },
    {
      "type": "pull_request",
      "parameters": {
        "required_approving_review_count": 1,
        "dismiss_stale_reviews_on_push": true,
        "require_code_owner_review": true,
        "require_last_push_approval": true,
        "required_review_thread_resolution": true
      }
    },
    {
      "type": "required_status_checks",
      "parameters": {
        "strict_required_status_checks_policy": true,
        "required_status_checks": [{ "context": "CI / ci-success" }]
      }
    },
    { "type": "required_linear_history" },
    { "type": "creation" },
    { "type": "update" }
  ],
  "bypass_actors": []
}
JSON
```

#### مثال ۳: Ruleset برای تگ‌ها

```bash
gh api -X POST "repos/$REPO/rulesets" --input - <<'JSON'
{
  "name": "protect-releases",
  "target": "tag",
  "enforcement": "active",
  "conditions": {
    "ref_name": { "include": ["v*.*.*"], "exclude": [] }
  },
  "rules": [
    { "type": "deletion" },
    { "type": "update" }
  ],
  "bypass_actors": []
}
JSON
```

#### مثال ۴: بررسی وضعیت امنیتی

```bash
echo "=== Dependabot alerts ==="
gh api "repos/$REPO/dependabot/alerts?state=open" --jq 'length'

echo "=== Secret scanning ==="
gh api "repos/$REPO/secret-scanning/alerts?state=open" --jq 'length'

echo "=== Code scanning ==="
gh api "repos/$REPO/code-scanning/alerts?state=open" --jq 'length'

echo "=== Rulesets ==="
gh api "repos/$REPO/rulesets" --jq '.[] | "\(.name): \(.enforcement)"'

echo "=== Actions permissions ==="
gh api "repos/$REPO/actions/permissions/workflow"
```

#### مثال ۵: اسکریپت چک‌لیست امنیتی

`scripts/security-check.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

REPO="${1:?usage: $0 owner/repo}"
FAIL=0

check() {
  local name="$1" result="$2"
  if [ "$result" = "ok" ]; then
    echo "✅ $name"
  else
    echo "❌ $name"
    FAIL=1
  fi
}

# Dependabot alerts
V=$(gh api "repos/$REPO/vulnerability-alerts" -i 2>/dev/null | head -1 | grep -c 204 || true)
check "Dependabot alerts" "$([ "$V" = "1" ] && echo ok || echo no)"

# Push protection
P=$(gh api "repos/$REPO" --jq '.security_and_analysis.secret_scanning_push_protection.status // "disabled"')
check "Push protection" "$([ "$P" = "enabled" ] && echo ok || echo no)"

# Actions default permission
W=$(gh api "repos/$REPO/actions/permissions/workflow" --jq '.default_workflow_permissions')
check "Actions default = read" "$([ "$W" = "read" ] && echo ok || echo no)"

# Ruleset روی main
R=$(gh api "repos/$REPO/rulesets" --jq '[.[] | select(.target=="branch")] | length')
check "Branch ruleset" "$([ "$R" -gt 0 ] && echo ok || echo no)"

# CODEOWNERS
C=$(gh api "repos/$REPO/contents/.github/CODEOWNERS" -i 2>/dev/null | head -1 | grep -c 200 || true)
check "CODEOWNERS" "$([ "$C" = "1" ] && echo ok || echo no)"

# SECURITY.md
S=$(gh api "repos/$REPO/contents/SECURITY.md" -i 2>/dev/null | head -1 | grep -c 200 || true)
check "SECURITY.md" "$([ "$S" = "1" ] && echo ok || echo no)"

exit "$FAIL"
```

#### مثال ۶: بررسی workflowها برای امنیت

```bash
# نصب
pipx install zizmor

# اجرا
zizmor .github/workflows/

# مواردی که پیدا می‌کند:
# - template injection
# - unpinned actions
# - excessive permissions
# - pull_request_target خطرناک
# - secrets در ورودی
```

#### مثال ۷: pre-commit با gitleaks

`.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.21.2
    hooks:
      - id: gitleaks

  - repo: https://github.com/rhysd/actionlint
    rev: v1.7.7
    hooks:
      - id: actionlint
```

#### مثال ۸: SBOM و attestation

```yaml
# در deploy.yml، بعد از build
- uses: anchore/sbom-action@<sha>
  with:
    image: ghcr.io/${{ github.repository }}:${{ github.sha }}
    format: spdx-json
    artifact-name: sbom.spdx.json

- uses: actions/attest-build-provenance@<sha>
  with:
    subject-name: ghcr.io/${{ github.repository }}
    subject-digest: ${{ steps.build.outputs.digest }}
    push-to-registry: true
```

#### مثال ۹: چک‌لیست امنیتی

```
□ 2FA روی حساب‌های تیم
□ Secret scanning + push protection
□ Dependabot alerts + security updates
□ Dependabot version updates با گروه‌بندی
□ CodeQL (یا Semgrep)
□ Ruleset روی main (PR + review + checks + linear history)
□ Ruleset روی تگ‌ها (بدون update/delete)
□ CODEOWNERS برای فایل‌های حساس
□ Actions: default_workflow_permissions = read
□ Actions: allow-list
□ Actions: SHA pinning (یا sha_pinning_required در org)
□ zizmor در CI
□ gitleaks در pre-commit
□ SECURITY.md + private vulnerability reporting
□ SBOM + attestation در release
□ audit log ماهانه (Enterprise)
```

### مقایسه و الگوهای مختلف

#### CodeQL در برابر Semgrep

| | CodeQL | Semgrep |
|---|---|---|
| عمق | ✅✅ | ✅ |
| سرعت | کند | ✅ |
| رایگان در خصوصی | ❌ | ✅ |
| مناسب | GitHub بومی | مکمل |

#### Ruleset در برابر Branch protection

| | Rulesets | Protection |
|---|---|---|
| تگ | ✅ | ❌ |
| چند branch | ✅ | ❌ |
| Evaluate | ✅ | ❌ |
| توصیه | ✅ | legacy |

#### push protection در برابر scanning

| | push protection | scanning |
|---|---|---|
| جلوگیری | ✅ | ❌ |
| پوشش تاریخ | ❌ | ✅ |
| هر دو لازم | ✅ | ✅ |

### اشکال شایع

#### ۱. alerts خاموش

اولین کاری که در مخزن جدید باید بکنی: روشن کردن.

#### ۲. Ruleset با bypass برای همه

اگر adminها bypass دارند، ruleset تشریفاتی است. حداقل نگه دار.

#### ۳. `write-all` در workflow

یک action آلوده می‌تواند کد را عوض کند. `contents: read` پیش‌فرض.

#### ۴. `@v4` به‌جای SHA

زنجیره‌ی تأمین. `sha_pinning_required` یا Dependabot برای actions.

#### ۵. CodeQL فقط هفتگی

PRهای جدید اسکن نمی‌شوند. `pull_request` هم trigger کن.

#### ۶. dismiss بی‌دلیل

بدهی امنیتی. هر dismiss را مستند کن.

#### ۷. بدون `SECURITY.md`

گزارش‌دهنده Issue عمومی می‌زند. کانال بگذار.

#### ۸. بدون SBOM

مشتری نمی‌تواند بررسی کند. برای محصول تجاری، اغلب الزام است.

### نکات مهم (Tips & Tricks)

- **همه‌ی alerts را در مخزن جدید روشن کن** — ۱۰ دقیقه.
- **`default_workflow_permissions=read`.**
- **Ruleset روی main و tag.**
- **`zizmor` در CI** — امنیت workflow.
- **`gitleaks` در pre-commit.**
- **SBOM + attestation** برای هر release.
- **چک‌لیست را ماهانه اجرا کن** (اسکریپت).

### خلاصه فصل

- پنج لایه: secret scanning، Dependabot، CodeQL، Rulesets، Actions security.
- `default_workflow_permissions=read` و SHA pinning.
- Ruleset روی main و tag.
- `zizmor` و `gitleaks` برای پیشگیری.
- SBOM + attestation برای زنجیره‌ی تأمین.
- چک‌لیست را خودکار کن.

**سؤالات مروری:**
1. چرا push protection و scanning هر دو لازم‌اند؟
2. چرا `write-all` خطرناک است؟
3. چرا Ruleset با bypass برای همه بی‌فایده است؟

---

## فصل ۹۷: Copilot و خودکارسازی

### مقدمه

این فصل Copilot را در پروژه تنظیم می‌کند و چند خودکارسازی مفید اضافه می‌کند: خوش‌آمد، برچسب خودکار، گزارش هفتگی، و Issue خودکار برای CI شکست‌خورده.

**چرا مهم است؟** یک مخزن بالغ، کارهای تکراری‌اش خودکار است.

**کاربرد عملی:** این فایل‌ها را می‌توانی کپی کنی.

### مفاهیم اساسی

#### چه چیزی خودکار کنیم

| کار | ابزار |
|---|---|
| خوش‌آمد به مشارکت‌کننده | Actions |
| برچسب خودکار بر اساس محتوا | Actions |
| بستن Issueهای قدیمی | `actions/stale` |
| گزارش هفتگی | Actions + `gh` |
| Issue خودکار برای CI شکست‌خورده | `workflow_run` |
| Copilot review | Ruleset یا Actions |
| Copilot instructions | فایل |

### مثال‌های عملی

#### مثال ۱: `copilot-instructions.md`

`.github/copilot-instructions.md`:

```markdown
# راهنمای پروژه برای Copilot

## زبان و ابزار
- TypeScript 5.6+، strict
- Fastify 5، Prisma 6، PostgreSQL 17
- Vitest برای unit، Playwright برای e2e

## قواعد
- `any` ممنوع. `unknown` + type guard.
- خطاها با `Result<T, E>` (نه throw)، به‌جز boundary.
- هر route باید zod schema داشته باشد.
- `routes/` مستقیم به Prisma دست نمی‌زند — از `services/`.
- نام فایل‌ها kebab-case، تایپ‌ها PascalCase.
- کامنت‌ها فارسی، کد انگلیسی.

## تست
- هر تابع جدید تست دارد.
- نام: `should <behavior> when <condition>`
- تست e2e فقط برای مسیرهای حیاتی.

## ممنوع
- `console.log` (از `logger` استفاده کن)
- کوئری SQL با string concatenation
- `fetch` بدون timeout
- وابستگی جدید بدون بحث

## دستورها
- `npm run dev`، `npm test`، `npm run lint`، `npm run typecheck`
```

#### مثال ۲: خوش‌آمد

`.github/workflows/welcome.yml`:

```yaml
name: Welcome

on:
  issues:
    types: [opened]
  pull_request_target:
    types: [opened]

permissions:
  issues: write
  pull-requests: write

jobs:
  welcome:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/github-script@<sha>
        with:
          script: |
            const isIssue = !!context.payload.issue;
            const author = isIssue
              ? context.payload.issue.user.login
              : context.payload.pull_request.user.login;

            const { data } = await github.rest.search.issuesAndPullRequests({
              q: `repo:${context.repo.owner}/${context.repo.repo} author:${author} type:issue`,
            });
            if (data.total_count > 1) return;

            const body = isIssue
              ? `سلام @${author}! 👋\n\nممنون برای اولین Issue. تا ۴۸ ساعت آینده جواب می‌دهیم.\n\n[CONTRIBUTING.md](../blob/main/CONTRIBUTING.md) را ببین.`
              : `سلام @${author}! 👋\n\nممنون برای اولین PR. یک reviewer تا ۴۸ ساعت آینده آن را می‌بیند.`;

            await github.rest.issues.createComment({
              ...context.repo,
              issue_number: (context.payload.issue ?? context.payload.pull_request).number,
              body,
            });
```

**نکته‌ی امنیتی:** `pull_request_target` در اینجا امن است چون کد PR را checkout/اجرا نمی‌کنیم.

#### مثال ۳: برچسب خودکار

`.github/workflows/labeler.yml`:

```yaml
name: Auto label

on:
  issues:
    types: [opened, edited]
  pull_request:
    types: [opened, edited, synchronize]

permissions:
  issues: write
  pull-requests: write
  contents: read

jobs:
  label:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/labeler@<sha>
        if: github.event_name == 'pull_request'
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}

      - uses: actions/github-script@<sha>
        if: github.event_name == 'issues'
        with:
          script: |
            const text = `${context.payload.issue.title}\n${context.payload.issue.body || ""}`.toLowerCase();
            const labels = [];
            if (/\b(خطا|error|bug|crash|broken)\b/.test(text)) labels.push("bug");
            if (/\b(اضافه|افزودن|feature|پیشنهاد)\b/.test(text)) labels.push("enhancement");
            if (/\b(مستند|docs|readme)\b/.test(text)) labels.push("documentation");
            if (/\b(امنیت|security|نشت|injection)\b/.test(text)) labels.push("priority:critical");
            if (labels.length) {
              await github.rest.issues.addLabels({
                ...context.repo,
                issue_number: context.payload.issue.number,
                labels: [...new Set(labels)],
              });
            }
```

`.github/labeler.yml`:

```yaml
"area:api":
  - changed-files:
      - any-glob-to-any-file: ["src/routes/**", "src/services/**"]

"area:db":
  - changed-files:
      - any-glob-to-any-file: ["prisma/**"]

"area:ci":
  - changed-files:
      - any-glob-to-any-file: [".github/**"]

"documentation":
  - changed-files:
      - any-glob-to-any-file: ["**/*.md", "docs/**"]
```

#### مثال ۴: stale

`.github/workflows/stale.yml`:

```yaml
name: Stale

on:
  schedule:
    - cron: "0 3 * * 1"
  workflow_dispatch:

permissions:
  issues: write
  pull-requests: write

jobs:
  stale:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/stale@<sha>
        with:
          days-before-stale: 60
          days-before-close: 14
          stale-issue-label: "status:needs-info"
          stale-issue-message: |
            این Issue ۶۰ روز بی‌فعالیت بوده. اگر هنوز مرتبط است، کامنت بگذار.
            در غیر این صورت ۱۴ روز دیگر بسته می‌شود.
          close-issue-message: "به‌خاطر بی‌فعالیتی بسته شد. اگر لازم شد، باز کن."
          exempt-issue-labels: "priority:critical,priority:high,pinned"
          days-before-pr-stale: -1
```

#### مثال ۵: گزارش هفتگی

`.github/workflows/weekly-report.yml`:

```yaml
name: Weekly report

on:
  schedule:
    - cron: "0 6 * * 1"
  workflow_dispatch:

permissions:
  issues: write
  contents: read

jobs:
  report:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/github-script@<sha>
        with:
          script: |
            const since = new Date(Date.now() - 7 * 86400 * 1000).toISOString();
            const { data: closed } = await github.rest.issues.listForRepo({
              ...context.repo, state: "closed", since, per_page: 100,
            });
            const { data: open } = await github.rest.issues.listForRepo({
              ...context.repo, state: "open", per_page: 100,
            });

            const closedCount = closed.filter(i => !i.pull_request).length;
            const mergedPRs = closed.filter(i => i.pull_request).length;
            const noAssignee = open.filter(i => !i.assignees?.length).length;

            const body = [
              `## گزارش هفتگی — ${new Date().toISOString().slice(0,10)}`,
              "",
              `- Issueهای بسته‌شده: **${closedCount}**`,
              `- PRهای merge‌شده: **${mergedPRs}**`,
              `- Issueهای باز: **${open.length}**`,
              `- بدون assignee: **${noAssignee}**`,
            ].join("\n");

            await github.rest.issues.create({
              ...context.repo,
              title: `گزارش هفتگی — ${new Date().toISOString().slice(0,10)}`,
              body,
              labels: ["report"],
            });
```

#### مثال ۶: Issue خودکار برای CI شکست‌خورده

```yaml
name: Nightly failure

on:
  workflow_run:
    workflows: [CI]
    types: [completed]

permissions:
  issues: write
  actions: read

jobs:
  report:
    if: >
      github.event.workflow_run.conclusion == 'failure' &&
      github.event.workflow_run.head_branch == 'main'
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/github-script@<sha>
        with:
          script: |
            const run = context.payload.workflow_run;

            // جلوگیری از Issue تکراری
            const { data: existing } = await github.rest.search.issuesAndPullRequests({
              q: `repo:${context.repo.owner}/${context.repo.repo} is:issue is:open label:ci "run ${run.id}"`,
            });
            if (existing.total_count > 0) return;

            await github.rest.issues.create({
              ...context.repo,
              title: `CI روی main شکست خورد — ${run.created_at.slice(0,10)}`,
              body: `[اجرا](${run.html_url}) شکست خورد.\n\n\`\`\`\nrun ${run.id}\n\`\`\``,
              labels: ["ci", "bug", "priority:high"],
            });
```

#### مثال ۷: Copilot review خودکار

```yaml
name: Copilot review

on:
  pull_request:
    types: [opened, ready_for_review]

permissions:
  pull-requests: write

jobs:
  request:
    if: github.event.pull_request.draft == false
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/github-script@<sha>
        with:
          script: |
            try {
              await github.rest.pulls.requestReviewers({
                ...context.repo,
                pull_number: context.payload.pull_request.number,
                reviewers: ["copilot-pull-request-reviewer[bot]"],
              });
            } catch (e) {
              core.warning(`درخواست review ناموفق: ${e.message}`);
            }
```

**هشدار:** نام bot و روش فعال‌سازی ممکن است تغییر کند.

#### مثال ۸: استفاده از Copilot Cloud Agent

روی یک Issue با معیار پذیرش مشخص:

```
Assignees → Copilot
```

**مناسب برای:** ارتقای وابستگی، رفع lint، نوشتن تست، رفع باگ با بازتولید مشخص.

**نامناسب برای:** طراحی معماری، تصمیم محصول، بازنویسی بزرگ.

#### مثال ۹: بررسی خودکارسازی‌ها

```bash
gh workflow list
gh run list --limit 20
```

اگر یک workflow مرتب شکست می‌خورد، یا حذفش کن یا fix.

### مقایسه و الگوهای مختلف

#### Actions در برابر Projects workflows

| | Actions | Projects |
|---|---|---|
| قدرت | ✅ | محدود |
| کد | ✅ | ❌ |
| سرعت | ثانیه | فوری |

#### Copilot review خودکار برای همه در برابر انتخابی

| | همه | فقط بزرگ |
|---|---|---|
| پوشش | ✅ | متوسط |
| نویز | زیاد | کم |
| مناسب | تیم کوچک | تیم بزرگ |

### اشکال شایع

#### ۱. حلقه‌ی بی‌پایان

`GITHUB_TOKEN` workflow جدید trigger نمی‌کند — ولی PAT می‌کند. مراقب باش.

#### ۲. برچسب خودکار خیلی وسیع

regex وسیع = همه‌چیز `bug`. `\b` بگذار و تست کن.

#### ۳. stale که Issue مهم را می‌بندد

`exempt-issue-labels` را تنظیم کن.

#### ۴. Issue تکراری برای CI

قبل از ساخت، جست‌وجو کن (مثال ۶).

#### ۵. `pull_request_target` + checkout

فصل ۵۴. در مثال‌های این فصل، checkout نمی‌کنیم — امن است.

#### ۶. Copilot review برای draft

`if: draft == false`.

#### ۷. خودکارسازی که کسی نمی‌داند

در `CONTRIBUTING.md` مستند کن.

### نکات مهم (Tips & Tricks)

- **`copilot-instructions.md`** — کیفیت پیشنهاد را بالا می‌برد.
- **`actions/labeler`** برای برچسب بر اساس مسیر.
- **`actions/stale`** با `exempt-labels`.
- **جست‌وجو قبل از ساخت Issue تکراری.**
- **`pull_request_target` بدون checkout** امن است.
- **خودکارسازی را مستند کن.**

### خلاصه فصل

- Copilot instructions قواعد پروژه را می‌دهد.
- خوش‌آمد، برچسب، stale، گزارش هفتگی، Issue خودکار.
- `pull_request_target` بدون checkout امن است.
- Copilot review برای PRهای non-draft.
- Cloud Agent برای کارهای مشخص و کوچک.
- خودکارسازی را مستند کن.

**سؤالات مروری:**
1. چرا `pull_request_target` بدون checkout امن است؟
2. چطور از Issue تکراری برای CI جلوگیری کنی؟
3. چه کاری برای Cloud Agent مناسب است؟

---

## فصل ۹۸: مستندات، release و تحویل

### مقدمه

این فصل پروژه را تمام می‌کند: مستندات نهایی، اولین release، و تحویل به تیم.

**چرا مهم است؟** پروژه‌ای که کسی نمی‌فهمد چطور استفاده کند، مرده است — حتی اگر کدش عالی باشد.

**کاربرد عملی:** چک‌لیست تحویل.

### مفاهیم اساسی

#### سه مخاطب مستندات

| مخاطب | چه می‌خواهد | کجا |
|---|---|---|
| **کاربر** | چطور استفاده کنم | README، docs/api |
| **مشارکت‌کننده** | چطور کمک کنم | CONTRIBUTING |
| **تیم** | چطور deploy/دیباگ کنم | docs/operations، RUNBOOK |

#### چک‌لیست تحویل

```
□ README کامل (نصب، استفاده، مثال)
□ docs/api.md
□ docs/architecture.md
□ docs/operations.md (deploy، rollback، monitoring)
□ CONTRIBUTING.md
□ SECURITY.md
□ LICENSE
□ CODE_OF_CONDUCT.md
□ CHANGELOG.md (خودکار)
□ اولین release با tag
□ SBOM در release
□ CI سبز
□ امنیت: alerts روشن، ruleset، CODEOWNERS
□ Copilot instructions
□ onboarding در Codespaces تست‌شده
```

### مثال‌های عملی

#### مثال ۱: `docs/api.md`

````markdown
# API

Base URL: `https://api.example.com`

## احراز هویت

همه‌ی endpointها به جز `/health` نیاز به `Authorization: Bearer <token>` دارند.

## خطاها

همه‌ی خطاها یک شکل دارند:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "ایمیل نامعتبر است",
    "details": [{ "field": "email", "issue": "invalid_format" }]
  }
}
```

| کد HTTP | code | معنی |
|---|---|---|
| 400 | `VALIDATION_ERROR` | ورودی نامعتبر |
| 401 | `UNAUTHENTICATED` | توکن نبود/باطل |
| 403 | `FORBIDDEN` | دسترسی ندارد |
| 404 | `NOT_FOUND` | یافت نشد |
| 409 | `CONFLICT` | تعارض (مثلاً ایمیل تکراری) |
| 422 | `BUSINESS_RULE` | قاعده‌ی کسب‌وکار |
| 429 | `RATE_LIMITED` | زیاد درخواست |
| 500 | `INTERNAL` | خطای سرور |

## Endpointها

### `GET /health`

```bash
curl https://api.example.com/health
```

```json
{ "status": "ok", "version": "1.4.0" }
```

### `POST /orders`

**Body:**

```json
{
  "customerId": "cus_123",
  "items": [
    { "sku": "SKU-1", "quantity": 2 }
  ],
  "discountCode": "WELCOME10"
}
```

**پاسخ 201:**

```json
{
  "data": {
    "id": "ord_456",
    "status": "pending",
    "total": 180000,
    "currency": "IRR",
    "createdAt": "2026-10-23T10:00:00Z"
  }
}
```

**خطاها:** `VALIDATION_ERROR`، `NOT_FOUND` (customer)، `BUSINESS_RULE` (کد تخفیف منقضی).
````

#### مثال ۲: `docs/operations.md`

````markdown
# عملیات

## Deploy

### staging
خودکار: هر merge به `main` (بعد از release).

### production
1. tag بزن: `git tag -a v1.4.0 -m "..." && git push origin v1.4.0`
2. workflow `Deploy` اجرا می‌شود
3. در GitHub → Actions → Deploy → **Review deployments** → Approve
4. smoke test خودکار اجرا می‌شود

## Rollback

```bash
# ۱) کدام نسخه؟
gh release list --limit 3

# ۲) deploy نسخه‌ی قبلی
gh workflow run deploy.yml -f environment=production
# یا:
kubectl rollout undo deployment/safaresh -n production

# ۳) تأیید
curl -fsS https://api.example.com/health
```

## Monitoring

| چه چیزی | کجا |
|---|---|
| خطاها | Sentry |
| متریک‌ها | Grafana |
| لاگ‌ها | Loki |
| وضعیت CI | Actions |
| آسیب‌پذیری | Security tab |

## Alertها

| Alert | معنی | اقدام |
|---|---|---|
| `HighErrorRate` | > ۵٪ خطای 5xx | rollback |
| `HighLatency` | p95 > 1s | بررسی DB |
| `DiskFull` | > ۹۰٪ | پاکسازی لاگ |

## RUNBOOK — خطاهای رایج

### خطای 500 در ثبت سفارش
1. لاگ Sentry را ببین
2. اگر `PrismaClientKnownRequestError` → بررسی migration
3. اگر timeout → بررسی connection pool

### CI شکست خورده
1. `gh run view --log-failed`
2. اگر flaky → Issue با label `flaky`
3. اگر واقعی → fix
````

#### مثال ۳: اولین release

```bash
# ۱) مطمئن شو main سبز است
gh run list --branch main --limit 1

# ۲) تگ
git switch main && git pull
git tag -a v1.0.0 -m "نسخه ۱.۰.۰ — انتشار اولیه"
git push origin v1.0.0

# ۳) release
gh release create v1.0.0 \
  --title "v1.0.0 — انتشار اولیه" \
  --generate-notes

# ۴) بررسی
gh release view v1.0.0
```

#### مثال ۴: release notes با دسته‌بندی

`.github/release.yml`:

```yaml
changelog:
  exclude:
    labels: [ignore-for-release, dependencies]
    authors: [dependabot, github-actions]
  categories:
    - title: "⚠️ تغییرات ناسازگار"
      labels: [breaking-change]
    - title: "🚀 ویژگی‌های جدید"
      labels: [enhancement, feature]
    - title: "🐛 رفع باگ"
      labels: [bug, fix]
    - title: "📚 مستندات"
      labels: [documentation]
    - title: "🔧 نگهداری"
      labels: [chore, ci, refactor]
    - title: "سایر"
      labels: ["*"]
```

#### مثال ۵: SBOM در release

```yaml
# در release.yml
- name: SBOM
  uses: anchore/sbom-action@<sha>
  with:
    path: dist
    format: spdx-json
    artifact-name: sbom.spdx.json

- name: الصاق به release
  uses: softprops/action-gh-release@<sha>
  with:
    files: |
      sbom.spdx.json
      dist/**
```

#### مثال ۶: onboarding با Codespaces

`.devcontainer/devcontainer.json`:

```json
{
  "name": "safaresh",
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspaces/safaresh",
  "forwardPorts": [3000, 5432],
  "postCreateCommand": {
    "deps": "npm ci",
    "db": "npm run db:migrate",
    "hooks": "npx husky install || true"
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "prisma.prisma"
      ]
    }
  }
}
```

تست:

```bash
# در Codespace
npm test
npm run dev
curl localhost:3000/health
```

#### مثال ۷: بررسی نهایی

```bash
#!/usr/bin/env bash
set -euo pipefail

REPO="${1:?usage: $0 owner/repo}"
FAIL=0

check() {
  local name="$1" result="$2"
  [ "$result" = "ok" ] && echo "✅ $name" || { echo "❌ $name"; FAIL=1; }
}

# فایل‌های ضروری
for f in README.md LICENSE CONTRIBUTING.md SECURITY.md .gitignore; do
  R=$(gh api "repos/$REPO/contents/$f" -i 2>/dev/null | head -1 | grep -c 200 || true)
  check "$f" "$([ "$R" = "1" ] && echo ok || echo no)"
done

# CI سبز
S=$(gh run list --repo "$REPO" --branch main --limit 1 --json conclusion --jq '.[0].conclusion')
check "CI سبز" "$([ "$S" = "success" ] && echo ok || echo no)"

# امنیت
P=$(gh api "repos/$REPO" --jq '.security_and_analysis.secret_scanning_push_protection.status // "disabled"')
check "Push protection" "$([ "$P" = "enabled" ] && echo ok || echo no)"

W=$(gh api "repos/$REPO/actions/permissions/workflow" --jq '.default_workflow_permissions')
check "Actions = read" "$([ "$W" = "read" ] && echo ok || echo no)"

# release
REL=$(gh release list --repo "$REPO" --limit 1 --json tagName --jq '.[0].tagName // ""')
check "release وجود دارد" "$([ -n "$REL" ] && echo ok || echo no)"

exit "$FAIL"
```

#### مثال ۸: تحویل به تیم

```markdown
## تحویل پروژه‌ی سفارش

### دسترسی‌ها
- مخزن: `myorg/safaresh`
- تیم: `@myorg/core-team` (write)
- production: environment با required reviewer

### شروع
1. Codespace بساز (Code → Codespaces)
2. `npm run dev`
3. `CONTRIBUTING.md` را بخوان

### جریان کار
1. Issue بساز
2. branch از main
3. PR (سایز < ۴۰۰ خط)
4. CI + review
5. Squash merge
6. deploy خودکار به staging

### production
- tag بزن
- approve در Actions
- smoke test خودکار

### در مشکل
- `docs/operations.md` → RUNBOOK
- `docs/architecture.md` → تصمیم‌ها
- تیم: #safaresh در Slack
```

### مقایسه و الگوهای مختلف

#### README در برابر docs

| | README | docs/ |
|---|---|---|
| مخاطب | اولین بازدید | کاربر فعال |
| طول | کوتاه | بلند |
| به‌روزرسانی | هر تغییر مهم | هر تغییر |

#### release دستی در برابر خودکار

| | دستی | خودکار |
|---|---|---|
| کنترل | ✅ | کم |
| سرعت | کند | ✅ |
| خطا | ممکن | کم |

#### مستندات در مخزن در برابر Wiki

| | مخزن | Wiki |
|---|---|---|
| review | ✅ | ❌ |
| جست‌وجو | ✅ | ✅ |
| نسخه‌بندی | ✅ | ✅ |
| توصیه | ✅ رسمی | FAQ |

### اشکال شایع

#### ۱. README کهنه

دستورهای قدیمی بدتر از نبودشان. README را بخشی از PR کن.

#### ۲. بدون RUNBOOK

در بحران، کسی نمی‌داند چه کند. `docs/operations.md`.

#### ۳. release بدون notes

کاربر نمی‌داند چه عوض شد. `--generate-notes` یا `.github/release.yml`.

#### ۴. بدون SBOM

مشتری نمی‌تواند بررسی کند.

#### ۵. Codespaces تست‌نشده

اگر `postCreateCommand` شکست بخورد، onboarding می‌شکند. تست کن.

#### ۶. تحویل بدون آموزش

تیم نمی‌داند جریان کار چیست. یک جلسه + سند.

#### ۷. بدون `CODE_OF_CONDUCT`

برای متن‌باز مهم است. حتی یک فایل ساده.

### نکات مهم (Tips & Tricks)

- **README کوتاه، docs کامل.**
- **RUNBOOK برای بحران** — نه بعد از بحران.
- **`--generate-notes` + `.github/release.yml`.**
- **SBOM در release.**
- **Codespaces را تست کن** قبل از تحویل.
- **چک‌لیست تحویل را خودکار کن.**

### خلاصه فصل

- سه مخاطب: کاربر، مشارکت‌کننده، تیم.
- `docs/operations.md` با RUNBOOK حیاتی است.
- release با notes خودکار و SBOM.
- Codespaces تست‌شده = onboarding صفر.
- چک‌لیست تحویل را اسکریپت کن.
- تحویل = سند + جلسه، نه فقط کد.

**سؤالات مروری:**
1. چرا RUNBOOK قبل از بحران لازم است؟
2. چرا README کهنه بدتر از نبودش است؟
3. چه چیزی در تحویل فراموش می‌شود؟

---

**بعدی:** [`22-Checklist-and-Glossary.md`](22-Checklist-and-Glossary.md)

---

## منابع و مراجع

- GitHub Docs: About repositories — https://docs.github.com/en/repositories
- GitHub Docs: Configuring a release — https://docs.github.com/en/repositories/releasing-projects-on-github
- GitHub Docs: Codespaces prebuilds — https://docs.github.com/en/codespaces/prebuilding-your-codespaces
- Google SRE Book (RUNBOOK) — https://sre.google/sre-book/table-of-contents/
- Keep a Changelog — https://keepachangelog.com/
- Semantic Versioning — https://semver.org/
- `softprops/action-gh-release` — https://github.com/softprops/action-gh-release
