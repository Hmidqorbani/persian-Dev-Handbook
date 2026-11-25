# بخش ۱۸: Pages، Codespaces و سایر قابلیت‌ها

**مدت یادگیری**: ۳ تا ۴ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۱۰ (Actions)

---

## فهرست محتوا

- [فصل ۸۰: GitHub Pages — میزبانی سایت از مخزن](#فصل-۸۰)
- [فصل ۸۱: Codespaces — محیط توسعه‌ی ابری](#فصل-۸۱)
- [فصل ۸۲: devcontainer — محیط قابل تکرار](#فصل-۸۲)
- [فصل ۸۳: Discussions، Wiki، Gist، Sponsors](#فصل-۸۳)

---

## فصل ۸۰: GitHub Pages — میزبانی سایت از مخزن

### مقدمه

GitHub Pages یک میزبان استاتیک رایگان است: HTML، CSS، JS، و سایت‌های تولیدشده (مثل Vite، Next.js static export). برای مستندات، وبلاگ، نمونه‌کار، و صفحه‌ی معرفی پروژه عالی است.

**چرا مهم است؟** مستندات در همان‌جایی که کد است، به‌روز می‌ماند. و رایگان است، با HTTPS و دامنه‌ی سفارشی.

**کاربرد عملی:** یک سایت مستندات یا نمونه‌کار، بدون هیچ هزینه‌ای.

### مفاهیم اساسی

#### انواع source

| Source | کِی |
|---|---|
| **Deploy from a branch** | فایل‌های آماده در branch (مثلاً `gh-pages` یا `docs/`) |
| **GitHub Actions** | build در CI و deploy (توصیه‌شده) |

#### محدودیت‌ها

| محدودیت | مقدار |
|---|---|
| حجم مخزن (توصیه) | ۱GB |
| حجم سایت منتشرشده | ۱GB |
| build در ساعت | ۱۰ بار (soft limit) |
| bandwidth | ۱۰۰GB/ماه (soft) |
| سایت خصوصی | ❌ (فقط با Enterprise) |

#### URLها

| نوع | URL |
|---|---|
| User/Org site | `username.github.io` |
| Project site | `username.github.io/repo-name` |
| دامنه‌ی سفارشی | `example.com` |

#### محدودیت‌های فنی

- فقط **static** — بدون سرور، بدون PHP، بدون دیتابیس
- Jekyll به‌طور پیش‌فرض پردازش می‌شود (می‌توان غیرفعال کرد)
- `.nojekyll` برای فایل‌های شروع‌شده با `_`

### مثال‌های عملی

#### مثال ۱: سریع‌ترین راه — از branch

```
Settings → Pages → Source: Deploy from a branch
  Branch: main
  Folder: /docs
→ Save
```

بعد از چند دقیقه: `https://username.github.io/repo/`.

#### مثال ۲: از Actions (توصیه‌شده) — Vite

`.github/workflows/pages.yml`:

```yaml
name: Deploy to Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>

      - uses: actions/setup-node@<sha>
        with:
          node-version: "22"
          cache: npm

      - run: npm ci
      - run: npm run build

      - uses: actions/configure-pages@<sha>

      - uses: actions/upload-pages-artifact@<sha>
        with:
          path: dist

  deploy:
    needs: build
    runs-on: ubuntu-24.04
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@<sha>
```

**نکته‌ی مهم:** `permissions` باید `pages: write` و `id-token: write` داشته باشد.

#### مثال ۳: Next.js (static export)

`next.config.js`:

```javascript
/** @type {import('next').NextConfig} */
export default {
  output: "export",
  basePath: process.env.NODE_ENV === "production" ? "/repo-name" : "",
  images: { unoptimized: true },
  trailingSlash: true,
};
```

workflow:

```yaml
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-pages-artifact@<sha>
        with:
          path: out
```

**توجه:** Next.js با `output: "export"` فقط بخش استاتیک را می‌دهد. Server Components با داده‌ی dynamic، API routes و Server Actions کار نمی‌کنند.

#### مثال ۴: Vite — تنظیم base

`vite.config.js`:

```javascript
export default defineConfig({
  base: process.env.GITHUB_ACTIONS ? "/repo-name/" : "/",
});
```

بدون `base`، assetها 404 می‌دهند (چون از `/repo-name/` سرو می‌شوند).

#### مثال ۵: Jekyll

```yaml
name: Deploy Jekyll

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
      - uses: ruby/setup-ruby@<sha>
        with:
          bundler-cache: true
      - run: bundle exec jekyll build
        env:
          JEKYLL_ENV: production
      - uses: actions/upload-pages-artifact@<sha>
        with:
          path: _site

  deploy:
    needs: build
    runs-on: ubuntu-24.04
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@<sha>
```

#### مثال ۶: `.nojekyll`

اگر فایل یا پوشه‌ای با `_` شروع می‌شود (مثل `_next/`، `_astro/`)، Jekyll آن را نادیده می‌گیرد. راه‌حل:

```bash
touch public/.nojekyll
# یا در build:
- run: touch dist/.nojekyll
```

#### مثال ۷: دامنه‌ی سفارشی

```
Settings → Pages → Custom domain: example.com
```

سپس در DNS:

```
# apex domain
A     @    185.199.108.153
A     @    185.199.109.153
A     @    185.199.110.153
A     @    185.199.111.153

# www
CNAME www  username.github.io
```

**نکته:** `CNAME` فایل در مخزن (اگر از branch deploy می‌کنی) هم لازم است. با Actions، GitHub خودش مدیریت می‌کند.

`Enforce HTTPS` را روشن کن (بعد از صدور گواهی، چند دقیقه تا چند ساعت).

#### مثال ۸: سایت کاربر (`username.github.io`)

مخزنی به نام `username.github.io` بساز. سایت در ریشه سرو می‌شود:

```
https://username.github.io/
```

این مخزن می‌تواند هم‌زمان میزبان چند پروژه هم باشد (هر پروژه در زیرمسیر).

#### مثال ۹: PR preview (با Cloudflare Pages یا Netlify)

GitHub Pages خودش preview ندارد. راه‌حل‌ها:

1. **Cloudflare Pages** — preview برای هر PR، رایگان
2. **Netlify** — همان
3. **workflow که به زیرمسیر deploy می‌کند:**

```yaml
on:
  pull_request:

jobs:
  preview:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
      - run: npm ci && npm run build -- --base=/preview/pr-${{ github.event.pull_request.number }}/
      - uses: peaceiris/actions-gh-pages@<sha>
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
          destination_dir: preview/pr-${{ github.event.pull_request.number }}
```

#### مثال ۱۰: بررسی مشکلات

```bash
# وضعیت
gh api repos/OWNER/REPO/pages

# buildهای اخیر
gh api repos/OWNER/REPO/pages/builds --jq '.[] | "\(.status) \(.created_at)"'

# اجرای workflow
gh run list --workflow pages.yml
```

### مقایسه و الگوهای مختلف

#### Pages در برابر Vercel/Netlify/Cloudflare

| | Pages | Vercel | Netlify | Cloudflare Pages |
|---|---|---|---|---|
| رایگان | ✅ | ✅ | ✅ | ✅ |
| preview PR | ❌ | ✅ | ✅ | ✅ |
| SSR | ❌ | ✅ | ✅ | ✅ (Workers) |
| دامنه‌ی سفارشی | ✅ | ✅ | ✅ | ✅ |
| یکپارچگی GitHub | ✅ بومی | ✅ | ✅ | ✅ |
| مناسب | مستندات، نمونه‌کار | اپ کامل | اپ کامل | اپ کامل |

#### Branch deploy در برابر Actions deploy

| | Branch | Actions |
|---|---|---|
| سادگی | ✅ | متوسط |
| build سفارشی | ❌ | ✅ |
| Jekyll خودکار | ✅ | دستی |
| توصیه | سایت ساده | پروژه‌ی واقعی |

### اشکال شایع

#### ۱. 404 برای assetها

```
GET https://user.github.io/repo/assets/index.js 404
```

`base` در Vite/Next تنظیم نشده. باید `/repo-name/` باشد.

#### ۲. سایت سفید است

- `.nojekyll` نبوده و `_next/` نادیده گرفته شده
- `base` اشتباه
- مسیر `path` در `upload-pages-artifact` اشتباه (باید پوشه‌ی build باشد، نه ریشه)

#### ۳. `permissions` ناقص

```
Error: Resource not accessible by integration
```

`pages: write` و `id-token: write` لازم است.

#### ۴. `environment` تنظیم نشده در job deploy

```
Error: Environment 'github-pages' not found
```

باید `environment: github-pages` باشد.

#### ۵. دامنه‌ی سفارشی کار نمی‌کند

- DNS propagation (تا ۲۴ ساعت)
- `CNAME` اشتباه
- `Enforce HTTPS` هنوز گواهی نگرفته

#### ۶. سایت خصوصی می‌خواهم

Pages فقط عمومی است (به‌جز Enterprise). برای خصوصی: Cloudflare Access، Netlify با password، یا self-host.

#### ۷. build timeout

Pages build (branch mode) محدودیت زمانی دارد. برای build سنگین، از Actions استفاده کن.

#### ۸. Jekyll سایت من را می‌شکند

اگر فایل `.md` با front matter (`---`) داری که نباید پردازش شود، `.nojekyll` بگذار یا `_config.yml` را تنظیم کن.

### نکات مهم (Tips & Tricks)

- **Actions deploy** بهتر از branch deploy است — کنترل کامل.
- **`base`** را برای project site تنظیم کن.
- **`.nojekyll`** برای فریم‌ورک‌های مدرن.
- **`concurrency: group: pages`** — جلوگیری از deploy هم‌زمان.
- **`Enforce HTTPS`** را روشن کن.
- **برای preview PR، Cloudflare Pages** بهتر است.

### خلاصه فصل

- Pages میزبان استاتیک رایگان با HTTPS.
- دو حالت: branch deploy و Actions deploy (توصیه‌شده).
- `base` را برای project site تنظیم کن.
- `.nojekyll` برای فریم‌ورک‌های مدرن.
- فقط static — بدون سرور.
- برای SSR/preview، Vercel/Netlify/Cloudflare.

**سؤالات مروری:**
1. چرا assetها 404 می‌دهند؟
2. چه زمانی `.nojekyll` لازم است؟
3. چرا Pages برای اپ SSR مناسب نیست؟

---

## فصل ۸۱: Codespaces — محیط توسعه‌ی ابری

### مقدمه

Codespaces یک محیط توسعه‌ی کامل در ابر است: VS Code در مرورگر (یا متصل به ادیتور محلی) با ترمینال، extensionها، و محیط آماده. با یک کلیک، بدون نصب چیزی.

**چرا مهم است؟** onboarding از روزها به دقیقه می‌رسد. «روی سیستم من کار می‌کند» حذف می‌شود. و می‌توانی از هر دستگاهی کار کنی.

**کاربرد عملی:** برای تیم‌های جدید، مشارکت‌کننده‌های متن‌باز، و کار از دستگاه ضعیف.

### مفاهیم اساسی

#### انواع machine

| Type | vCPU | RAM | Storage |
|---|---|---|---|
| 2-core | ۲ | ۸GB | ۳۲GB |
| 4-core | ۴ | ۱۶GB | ۳۲GB |
| 8-core | ۸ | ۳۲GB | ۶۴GB |
| 16-core | ۱۶ | ۶۴GB | ۱۲۸GB |
| 32-core | ۳۲ | ۱۲۸GB | ۱۲۸GB |

#### سهمیه (Free)

| | مقدار |
|---|---|
| Core hours | ۱۲۰/ماه (2-core) |
| Storage | ۱۵GB-month |
| idle timeout | ۳۰ دقیقه (قابل تنظیم) |

#### چطور کار می‌کند

```
مخزن → Codespace → container (devcontainer) → VS Code (web یا desktop)
```

Codespace یک VM است که در آن container اجرا می‌شود. container از `.devcontainer/devcontainer.json` ساخته می‌شود (اگر باشد) یا از image پیش‌فرض.

#### حالت‌های دسترسی

| حالت | چطور |
|---|---|
| **مرورگر** | github.com/codespaces → Open |
| **VS Code Desktop** | `code --remote` یا از UI |
| **JetBrains** | Gateway |
| **CLI** | `gh codespace ssh` |

### مثال‌های عملی

#### مثال ۱: ساخت Codespace

```
مخزن → Code (سبز) → Codespaces → Create codespace on main
```

یا:

```bash
gh codespace create --repo myorg/backend --branch main --machine basicLinux32gb
gh codespace list
gh codespace code            # باز کردن در VS Code
gh codespace ssh             # SSH
gh codespace stop
gh codespace delete
```

#### مثال ۲: تنظیمات

```
Settings → Codespaces
  Default region: Europe West
  Default machine type: 4-core
  Idle timeout: 30 minutes
  Dotfiles: https://github.com/ali/dotfiles
```

**Dotfiles** — یک مخزن با `.bashrc`، `.gitconfig`، ... که در هر Codespace اعمال می‌شود.

#### مثال ۳: `devcontainer.json` حداقلی

`.devcontainer/devcontainer.json`:

```json
{
  "name": "backend",
  "image": "mcr.microsoft.com/devcontainers/javascript-node:22",
  "features": {
    "ghcr.io/devcontainers/features/docker-in-docker:2": {},
    "ghcr.io/devcontainers/features/github-cli:1": {}
  },
  "forwardPorts": [3000, 5432],
  "postCreateCommand": "npm ci",
  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "bradlc.vscode-tailwindcss"
      ],
      "settings": {
        "editor.formatOnSave": true,
        "editor.defaultFormatter": "esbenp.prettier-vscode"
      }
    }
  }
}
```

#### مثال ۴: با دیتابیس (docker-compose)

`.devcontainer/docker-compose.yml`:

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - ../..:/workspaces:cached
    command: sleep infinity
    environment:
      DATABASE_URL: postgres://postgres:postgres@db:5432/app

  db:
    image: postgres:17
    restart: unless-stopped
    environment:
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: app
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
```

`.devcontainer/devcontainer.json`:

```json
{
  "name": "full-stack",
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspaces/${localWorkspaceFolderBasename}",
  "forwardPorts": [3000, 5432],
  "postCreateCommand": "npm ci && npm run db:migrate"
}
```

#### مثال ۵: با secret

```json
{
  "secrets": {
    "NPM_TOKEN": {
      "description": "توکن npm برای پکیج‌های خصوصی"
    }
  },
  "containerEnv": {
    "NODE_ENV": "development"
  }
}
```

**نکته:** secret در Codespaces جدا از Actions secrets است. در **Settings → Codespaces → Secrets** تعریف می‌شود.

#### مثال ۶: prebuild (سرعت)

```
Settings → Codespaces → Prebuild configuration
  Set up prebuild
  Configuration: .devcontainer/devcontainer.json
  Trigger: On push to main
  Region: Europe West
```

Prebuild یک image آماده می‌سازد. Codespace جدید در ۱۰-۲۰ ثانیه بالا می‌آید (به‌جای ۲-۵ دقیقه).

**هزینه:** prebuild هم core-hour مصرف می‌کند (ولی کمتر از build هر بار).

#### مثال ۷: چند devcontainer

```
.devcontainer/
├── devcontainer.json          ← پیش‌فرض
├── backend/
│   └── devcontainer.json
└── frontend/
    └── devcontainer.json
```

استفاده: **Code → Codespaces → ... → New with options → Dev container configuration**.

#### مثال ۸: پورت‌ها

```json
{
  "forwardPorts": [3000, 5173, 5432],
  "portsAttributes": {
    "3000": { "label": "API", "onAutoForward": "notify" },
    "5173": { "label": "Frontend", "onAutoForward": "openBrowser" },
    "5432": { "label": "Postgres", "onAutoForward": "silent" }
  }
}
```

پورت‌های forwarded در یک URL عمومی (اختیاری) یا خصوصی قابل دسترسی‌اند.

**هشدار:** پورت عمومی = هر کسی با لینک می‌تواند دسترسی داشته باشد. برای دیتابیس، `silent` و خصوصی.

#### مثال ۹: `gh codespace` برای اسکریپت

```bash
# لیست
gh codespace list

# ساخت و اتصال
gh codespace create -R myorg/backend -b feat/x -m basicLinux32gb

# SSH و اجرای دستور
gh codespace ssh -c <name> -- "npm test"

# کپی فایل
gh codespace cp ./file.txt remote:/workspaces/backend/

# پورت
gh codespace ports -c <name>

# logs
gh codespace logs -c <name>

# حذف همه
gh codespace delete --all --force
```

#### مثال ۱۰: محدودیت‌ها

| محدودیت | توضیح |
|---|---|
| زمان ساخت | ۳۰ ثانیه تا ۵ دقیقه (بدون prebuild) |
| idle timeout | ۳۰ دقیقه (پیش‌فرض) |
| توقف بعد از | ۳۰ روز بی‌استفاده (حذف) |
| GPU | محدود |
| دسترسی شبکه | عمومی (نه شبکه‌ی شرکت) |

### مقایسه و الگوهای مختلف

#### Codespaces در برابر Gitpod در برابر local

| | Codespaces | Gitpod | Local |
|---|---|---|---|
| یکپارچگی GitHub | ✅ بومی | ✅ | — |
| رایگان | ✅ (۱۲۰ ساعت) | ✅ محدود | — |
| سرعت شروع | ✅ (با prebuild) | ✅ | فوری |
| منابع | محدود | محدود | ✅ کامل |
| آفلاین | ❌ | ❌ | ✅ |

#### Codespaces در برابر devcontainer محلی

| | Codespaces | devcontainer محلی |
|---|---|---|
| نیاز به Docker | ❌ | ✅ |
| منابع | ابر | لپ‌تاپ |
| هزینه | سهمیه | صفر |
| یکسان بودن محیط | ✅ | ✅ |

**نکته:** `devcontainer.json` هم در Codespaces و هم در VS Code محلی کار می‌کند — همان فایل، دو محیط.

#### machine type

| | 2-core | 4-core | 8-core |
|---|---|---|---|
| تست | ✅ | ✅ | ✅ |
| build سنگین | کند | ✅ | ✅ |
| core-hour مصرف | ۱× | ۲× | ۴× |

**توصیه:** ۲-core پیش‌فرض، ۴-core برای build.

### اشکال شایع

#### ۱. Codespace کند بالا می‌آید

بدون prebuild، هر بار از صفر. prebuild راه‌اندازی کن.

#### ۲. سهمیه تمام شد

```
You have no Codespaces quota remaining
```

idle timeout را کم کن، machine را کوچک کن، یا Codespaceهای بی‌استفاده را حذف کن.

#### ۳. `postCreateCommand` شکست می‌خورد

لاگ را ببین:

```bash
gh codespace logs -c <name>
```

معمولاً: secret نبودن، نسخه‌ی Node اشتباه، شبکه.

#### ۴. پورت باز نمی‌شود

- `forwardPorts` تنظیم نشده
- برنامه روی `0.0.0.0` گوش نمی‌دهد (باید `--host 0.0.0.0`)
- پورت در firewall

#### ۵. secret در devcontainer

```json
✗ "containerEnv": { "API_KEY": "sk-xxx" }
```

در مخزن commit می‌شود. از `secrets` استفاده کن.

#### ۶. Codespace با image اشتباه

اگر `image` را تنظیم نکنی، image پیش‌فرض (universal) استفاده می‌شود که بزرگ است و کند بالا می‌آید.

#### ۷. devcontainer محلی کار نمی‌کند

نیاز به Docker Desktop + extension "Dev Containers".

#### ۸. فراموش کردن حذف

Codespaceهای قدیمی storage مصرف می‌کنند. ماهی یک بار `gh codespace delete --all` یا حذف دستی.

### نکات مهم (Tips & Tricks)

- **prebuild** — تفاوت بین ۳۰ ثانیه و ۵ دقیقه.
- **`devcontainer.json`** را در مخزن بگذار — onboarding صفر.
- **idle timeout را کم کن** (۱۵ دقیقه) — سهمیه.
- **`gh codespace ssh`** برای اسکریپت.
- **`forwardPorts` با `silent`** برای دیتابیس.
- **Codespaceهای قدیمی را حذف کن.**

### خلاصه فصل

- Codespaces محیط توسعه‌ی ابری با VS Code.
- `devcontainer.json` محیط را تعریف می‌کند (هم Codespaces هم محلی).
- prebuild سرعت شروع را چند برابر می‌کند.
- سهمیه: ۱۲۰ core-hour/ماه (Free).
- پورت عمومی = خطر؛ دیتابیس را خصوصی نگه دار.
- Codespaceهای بی‌استفاده را حذف کن.

**سؤالات مروری:**
1. چرا `devcontainer.json` در مخزن مهم است؟
2. prebuild چه چیزی را سریع می‌کند؟
3. چرا پورت عمومی برای دیتابیس خطرناک است؟

---

## فصل ۸۲: devcontainer — محیط قابل تکرار

### مقدمه

`devcontainer.json` یک استاندارد باز است که محیط توسعه را به‌صورت کد تعریف می‌کند. همان فایل در VS Code، Codespaces، JetBrains، و CLI کار می‌کند.

**چرا مهم است؟** «روی سیستم من کار می‌کند» با یک فایل حل می‌شود. نسخه‌ی Node، ابزارها، extensionها، پورت‌ها — همه تعریف‌شده.

**کاربرد عملی:** onboarding از «۲ روز نصب» به «یک کلیک».

### مفاهیم اساسی

#### ساختار

```
.devcontainer/
├── devcontainer.json     ← اجباری
├── Dockerfile            ← اختیاری
└── docker-compose.yml    ← اختیاری
```

#### کلیدهای مهم

| کلید | کار |
|---|---|
| `image` | image آماده |
| `build.dockerfile` | Dockerfile سفارشی |
| `dockerComposeFile` | compose |
| `features` | قابلیت اضافه (Node، Docker، gh، ...) |
| `forwardPorts` | پورت‌ها |
| `postCreateCommand` | بعد از ساخت |
| `postStartCommand` | هر بار شروع |
| `postAttachCommand` | بعد از اتصال |
| `containerEnv` | متغیر محیطی |
| `mounts` | mount |
| `customizations.vscode.extensions` | extensionها |
| `customizations.vscode.settings` | تنظیمات |
| `remoteUser` | کاربر |

#### Features

Features قطعه‌های آماده‌اند که روی image اضافه می‌شوند:

```json
"features": {
  "ghcr.io/devcontainers/features/node:1": { "version": "22" },
  "ghcr.io/devcontainers/features/python:1": { "version": "3.12" },
  "ghcr.io/devcontainers/features/docker-in-docker:2": {},
  "ghcr.io/devcontainers/features/github-cli:1": {},
  "ghcr.io/devcontainers/features/aws-cli:1": {}
}
```

**مزیت:** بدون Dockerfile سفارشی، محیط دلخواه.

### مثال‌های عملی

#### مثال ۱: Node.js ساده

```json
{
  "name": "Node.js",
  "image": "mcr.microsoft.com/devcontainers/javascript-node:22",
  "postCreateCommand": "npm ci",
  "forwardPorts": [3000],
  "customizations": {
    "vscode": {
      "extensions": ["dbaeumer.vscode-eslint", "esbenp.prettier-vscode"]
    }
  }
}
```

#### مثال ۲: Python با uv

```json
{
  "name": "Python",
  "image": "mcr.microsoft.com/devcontainers/python:3.12",
  "features": {
    "ghcr.io/astral-sh/uv:1": {}
  },
  "postCreateCommand": "uv sync",
  "customizations": {
    "vscode": {
      "extensions": ["charliermarsh.ruff", "ms-python.python"]
    }
  }
}
```

#### مثال ۳: چند زبانه (Node + Python + Postgres)

```json
{
  "name": "fullstack",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu-24.04",
  "features": {
    "ghcr.io/devcontainers/features/node:1": { "version": "22" },
    "ghcr.io/devcontainers/features/python:1": { "version": "3.12" },
    "ghcr.io/devcontainers/features/docker-in-docker:2": {},
    "ghcr.io/devcontainers/features/postgresql:1": { "version": "17" }
  },
  "postCreateCommand": "npm ci && uv sync",
  "forwardPorts": [3000, 5432]
}
```

#### مثال ۴: Dockerfile سفارشی

`.devcontainer/Dockerfile`:

```dockerfile
FROM mcr.microsoft.com/devcontainers/base:ubuntu-24.04

# ابزارهای سیستمی
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    postgresql-client \
    jq \
    ripgrep \
    && rm -rf /var/lib/apt/lists/*

# کاربر غیر root
ARG USERNAME=vscode
USER $USERNAME

# ابزارهای زبان
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/home/$USERNAME/.bun/bin:$PATH"
```

```json
{
  "build": {
    "dockerfile": "Dockerfile",
    "args": { "USERNAME": "vscode" }
  }
}
```

#### مثال ۵: `postCreateCommand` چندتایی

```json
{
  "postCreateCommand": {
    "deps": "npm ci",
    "db": "npm run db:migrate",
    "hooks": "npx lefthook install",
    "env": "cp .env.example .env"
  }
}
```

هر کلید یک نام است — در لاگ جدا نشان داده می‌شود.

#### مثال ۶: mount برای cache

```json
{
  "mounts": [
    "source=${localWorkspaceFolder}/.cache,target=/home/node/.cache,type=bind",
    "source=node-modules,target=${containerWorkspaceFolder}/node_modules,type=volume"
  ]
}
```

**`node_modules` به‌عنوان volume** — سریع‌تر از bind mount (که در macOS/Windows کند است).

#### مثال ۷: secret

```json
{
  "secrets": {
    "NPM_TOKEN": { "description": "توکن npm" },
    "DATABASE_URL": { "description": "اتصال دیتابیس محلی" }
  }
}
```

Codespaces از **Settings → Codespaces → Secrets** می‌خواند. VS Code محلی از prompt.

#### مثال ۸: کاربر غیر root

```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "remoteUser": "vscode",
  "containerUser": "vscode"
}
```

**مهم:** اگر با root کار کنی، فایل‌های ساخته‌شده مال root می‌شوند و روی host مشکل می‌سازند.

#### مثال ۹: `waitFor`

```json
{
  "waitFor": "postCreateCommand"
}
```

یا `"onCreateCommand"` — ترتیب اجرا را کنترل می‌کند.

#### مثال ۱۰: تست devcontainer

```bash
# نصب CLI
npm install -g @devcontainers/cli

# ساخت
devcontainer up --workspace-folder .

# اجرای دستور
devcontainer exec --workspace-folder . npm test

# build image
devcontainer build --workspace-folder . --image-name my-devcontainer
```

**مزیت:** تست در CI که devcontainer سالم است.

#### مثال ۱۱: devcontainer در CI

```yaml
jobs:
  devcontainer:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@<sha>
      - run: npm install -g @devcontainers/cli
      - run: devcontainer up --workspace-folder .
      - run: devcontainer exec --workspace-folder . npm test
```

### مقایسه و الگوهای مختلف

#### `image` در برابر `Dockerfile` در برابر `compose`

| | image | Dockerfile | compose |
|---|---|---|---|
| سادگی | ✅ | متوسط | پیچیده |
| سفارشی‌سازی | features | ✅ کامل | ✅ کامل |
| چند سرویس | ❌ | ❌ | ✅ |
| مناسب | اکثر | ابزار سیستمی | دیتابیس + اپ |

#### `postCreateCommand` در برابر `postStartCommand`

| | postCreate | postStart |
|---|---|---|
| کِی | اولین ساخت | هر بار شروع |
| مناسب | `npm ci`، migrate | `npm run dev` |

#### features در برابر Dockerfile

| | features | Dockerfile |
|---|---|---|
| سرعت | ✅ | متوسط |
| کنترل | محدود | ✅ |
| کش | ✅ | ✅ |
| مناسب | ابزار استاندارد | نیاز خاص |

### اشکال شایع

#### ۱. `postCreateCommand` که هر بار اجرا می‌شود

اگر سنگین است، از `onCreateCommand` استفاده کن (فقط یک بار، در build).

#### ۲. `node_modules` روی bind mount

در macOS/Windows کند است. از volume استفاده کن (مثال ۶).

#### ۳. کاربر root

فایل‌ها مال root می‌شوند. `remoteUser` بگذار.

#### ۴. secret در `containerEnv`

commit می‌شود. از `secrets`.

#### ۵. پورت روی `localhost` bind شده

برنامه باید روی `0.0.0.0` گوش بدهد، وگرنه از بیرون container قابل دسترسی نیست:

```javascript
app.listen(3000, "0.0.0.0");
```

#### ۶. image بزرگ

`base:ubuntu` + features، حجیم است. اگر ممکن است، image آماده‌ی زبان استفاده کن.

#### ۷. devcontainer بدون تست

اگر شکست بخورد، تازه در Codespace می‌فهمی. در CI تست کن (مثال ۱۱).

#### ۸. `mounts` با مسیر اشتباه

```
✗ source=./cache
✓ source=${localWorkspaceFolder}/.cache
```

### نکات مهم (Tips & Tricks)

- **features** برای ابزار استاندارد — سریع‌تر از Dockerfile.
- **`node_modules` volume** — سرعت در macOS/Windows.
- **`remoteUser`** غیر root.
- **`postCreateCommand`** برای setup یک‌باره.
- **در CI تست کن** — `devcontainer up`.
- **prebuild در Codespaces** برای سرعت.

### خلاصه فصل

- `devcontainer.json` محیط را به‌صورت کد تعریف می‌کند.
- همان فایل در VS Code، Codespaces، JetBrains کار می‌کند.
- `features` ابزار اضافه می‌کند بدون Dockerfile.
- `postCreateCommand` برای setup؛ `postStartCommand` برای هر بار.
- `node_modules` را volume کن.
- در CI تست کن.

**سؤالات مروری:**
1. چرا `devcontainer.json` در مخزن مهم است؟
2. تفاوت `postCreateCommand` و `postStartCommand` چیست؟
3. چرا `node_modules` را باید volume کرد؟

---

## فصل ۸۳: Discussions، Wiki، Gist، Sponsors

### مقدمه

چهار قابلیت جانبی که هر کدام یک نیاز مشخص را برطرف می‌کنند. اگر بدانی کدام برای چه، از Issue برای همه‌چیز استفاده نمی‌کنی.

**چرا مهم است؟** Issueهای پر از سؤال، پروژه را فلج می‌کنند. Discussions برای گفت‌وگو، Wiki برای مستندات سریع، Gist برای قطعه‌کد.

**کاربرد عملی:** این فصل نقشه‌ی «کدام ابزار برای کدام نیاز» است.

### مفاهیم اساسی

#### جدول تصمیم

| نیاز | ابزار |
|---|---|
| باگ قابل بستن | Issue |
| پرسش «چطور X کنم؟» | Discussions (Q&A) |
| ایده‌ی بلندمدت | Discussions (Ideas) |
| اعلام نسخه | Discussions (Announcements) یا Release |
| مستندات سریع بدون PR | Wiki |
| مستندات با review | فایل `.md` در مخزن |
| قطعه‌کد اشتراکی | Gist |
| حمایت مالی | Sponsors |

#### Discussions

| دسته | کاربرد |
|---|---|
| 📣 Announcements | اعلان‌ها |
| 💡 Ideas | پیشنهاد ویژگی |
| 🙏 Q&A | پرسش و پاسخ (با «best answer») |
| 🗣 General | گفت‌وگوی آزاد |
| 🐛 Show and tell | نمایش کار |

مزیت: قابل جست‌وجو، best answer، بدون بستن.

#### Wiki

- مخزن Git جدا (`repo.wiki.git`)
- قابل clone و ویرایش محلی
- بدون PR review
- مناسب: مستندات سریع، FAQ

#### Gist

- فایل‌های تک یا چندتایی
- public یا secret (نه private — هر کسی با لینک می‌بیند)
- قابل embed
- نسخه‌بندی (revision)

#### Sponsors

- دکمه‌ی Sponsor در مخزن
- GitHub Sponsors، Patreon، Open Collective، Ko-fi
- نیاز به `FUNDING.yml`

### مثال‌های عملی

#### مثال ۱: فعال‌سازی Discussions

```bash
gh api -X PATCH repos/OWNER/REPO -F has_discussions=true
```

یا **Settings → General → Features → Discussions**.

#### مثال ۲: دسته‌های Discussions

```
Discussions → Categories → New category

Name: Q&A
Format: Question / answer
Description: پرسش درباره‌ی استفاده
```

#### مثال ۳: قالب Discussion

`.github/DISCUSSION_TEMPLATE/q-and-a.yml`:

```yaml
title: "[Q&A] "
labels: ["question"]
body:
  - type: markdown
    attributes:
      value: |
        قبل از پرسیدن، در Discussions جست‌وجو کن.

  - type: textarea
    id: question
    attributes:
      label: سؤال
    validations:
      required: true

  - type: textarea
    id: tried
    attributes:
      label: چه چیزی امتحان کردی؟
    validations:
      required: true

  - type: input
    id: version
    attributes:
      label: نسخه
    validations:
      required: true
```

#### مثال ۴: تبدیل Issue به Discussion

```
Issue → ... → Convert to discussion
```

و برعکس: `Discussion → ... → Create issue`.

#### مثال ۵: Wiki

```bash
# clone ویکی
git clone https://github.com/OWNER/REPO.wiki.git
cd REPO.wiki

# صفحه بساز
echo "# راهنمای نصب" > Installation.md
git add . && git commit -m "docs: راهنمای نصب"
git push
```

**توجه:** Wiki تاریخچه‌ی جدا دارد و در PR review نمی‌آید.

#### مثال ۶: Wiki یا فایل در مخزن؟

| | Wiki | فایل `.md` |
|---|---|---|
| review | ❌ | ✅ |
| جست‌وجو در کد | ❌ | ✅ |
| دسترسی مشارکت‌کننده | محدود | ✅ |
| سرعت ویرایش | ✅ | متوسط |
| مناسب | FAQ، یادداشت | مستندات رسمی |

**توصیه:** مستندات رسمی در `docs/`، FAQ سریع در Wiki (یا Discussions).

#### مثال ۷: Gist

```bash
# ساخت
gh gist create script.sh --desc "اسکریپت پشتیبان‌گیری" --public
gh gist create file1.txt file2.txt --desc "چند فایل" --secret

# لیست
gh gist list

# clone
git clone https://gist.github.com/<id>.git

# ویرایش
gh gist edit <id>

# حذف
gh gist delete <id>
```

**نکته:** `secret` gist **خصوصی نیست** — هر کسی با لینک می‌بیند. برای کد حساس استفاده نکن.

#### مثال ۸: embed Gist

```html
<script src="https://gist.github.com/USER/ID.js"></script>
```

#### مثال ۹: Sponsors

`.github/FUNDING.yml`:

```yaml
github: [ali]
patreon: ali_dev
open_collective: my-project
ko_fi: alidev
tidelift: npm/my-package
custom: ["https://example.com/donate"]
```

بعد از commit، دکمه‌ی **Sponsor** ظاهر می‌شود.

#### مثال ۱۰: `gh` برای Discussions

`gh` دستور بومی برای Discussions ندارد، ولی با `gh api` می‌شود:

```bash
# لیست
gh api graphql -f query='
query($owner: String!, $repo: String!) {
  repository(owner: $owner, name: $repo) {
    discussions(first: 10) {
      nodes { number title category { name } }
    }
  }
}' -f owner=myorg -f repo=backend
```

#### مثال ۱۱: لینک دادن بین Issue و Discussion

```markdown
این سؤال در #discussion-12 پاسخ داده شد.
```

GitHub خودکار لینک می‌دهد.

### مقایسه و الگوهای مختلف

#### Discussions در برابر Issue در برابر Slack

| | Discussions | Issue | Slack |
|---|---|---|---|
| قابل جست‌وجو (عمومی) | ✅ | ✅ | ❌ |
| best answer | ✅ | ❌ | ❌ |
| قابل بستن | ❌ | ✅ | ❌ |
| در Projects | محدود | ✅ | ❌ |
| مناسب | پرسش، ایده | کار | گفت‌وگوی فوری |

#### Wiki در برابر Pages در برابر `docs/`

| | Wiki | Pages | docs/ |
|---|---|---|---|
| review | ❌ | ✅ (با PR) | ✅ |
| جست‌وجو | ✅ | ✅ | ✅ |
| ساختار | ساده | کامل | ساده |
| مناسب | FAQ | سایت مستندات | مستندات فنی |

#### Gist در برابر repo

| | Gist | Repo |
|---|---|---|
| نسخه‌بندی | ✅ | ✅ |
| issue/PR | ❌ | ✅ |
| fork | ✅ | ✅ |
| مناسب | قطعه‌کد | پروژه |

### اشکال شایع

#### ۱. Issue برای همه‌چیز

«چطور نصب کنم؟» در Issue → Issueهای تکراری، پروژه شلوغ. Discussions.

#### ۲. Discussions بدون دسته

همه در General → پیدا کردن سخت. دسته‌بندی کن.

#### ۳. Wiki برای مستندات رسمی

بدون review، مستندات کهنه می‌شود. برای رسمی، `docs/` با PR.

#### ۴. `secret` gist برای کد حساس

```
✗ gist با API key
```

هر کسی با لینک می‌بیند. Gist حتی secret، **خصوصی نیست**.

#### ۵. Wiki که fork نمی‌شود

Wiki در fork کپی نمی‌شود. مشارکت‌کننده نمی‌تواند PR بدهد.

#### ۶. FUNDING.yml بدون حساب فعال

دکمه ظاهر می‌شود ولی لینک کار نمی‌کند. حساب Sponsors را فعال کن.

#### ۷. Discussions بدون `best answer`

اگر format را Question نگذاری، امکان best answer نیست.

### نکات مهم (Tips & Tricks)

- **Discussions را در `config.yml` Issue template لینک بده** — کاربران را هدایت کن.
- **Q&A format** برای best answer.
- **`docs/` برای رسمی، Wiki برای FAQ.**
- **Gist secret = عمومی با لینک.**
- **FUNDING.yml** — یک خط، دکمه‌ی Sponsor.
- **Issue را به Discussion تبدیل کن** وقتی گفت‌وگو می‌شود.

### خلاصه فصل

- Discussions برای گفت‌وگو (Q&A، Ideas، Announcements).
- Wiki برای مستندات سریع بدون review.
- Gist برای قطعه‌کد؛ secret ≠ خصوصی.
- Sponsors با `FUNDING.yml`.
- مستندات رسمی در `docs/` با PR.
- Issue را به Discussion تبدیل کن وقتی کار نیست.

**سؤالات مروری:**
1. چه زمانی Issue و چه زمانی Discussion؟
2. چرا Gist secret خصوصی نیست؟
3. چرا Wiki برای مستندات رسمی مناسب نیست؟

---

**بعدی:** [`19-Advanced-Git.md`](19-Advanced-Git.md)

---

## منابع و مراجع

- GitHub Docs: About GitHub Pages — https://docs.github.com/en/pages
- GitHub Docs: Custom domains for Pages — https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site
- GitHub Docs: Codespaces — https://docs.github.com/en/codespaces
- Development Containers spec — https://containers.dev/
- devcontainer features — https://containers.dev/features
- GitHub Docs: Discussions — https://docs.github.com/en/discussions
- GitHub Docs: About wikis — https://docs.github.com/en/communities/documenting-your-project-with-wikis
- GitHub Docs: Gists — https://docs.github.com/en/get-started/writing-on-github/editing-and-sharing-content-with-gists
- GitHub Sponsors — https://docs.github.com/en/sponsors
