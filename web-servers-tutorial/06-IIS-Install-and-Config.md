# بخش ۰۶: IIS — نصب و پیکربندی

**مدت یادگیری**: ۴ تا ۵ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: بخش ۰۳ (معماری IIS، `HTTP.sys`)، دسترسی Administrator روی ویندوز سرور

---

## فهرست محتوا

- [فصل ۲۳: نصب IIS و سه راه مدیریت](#فصل-۲۳)
- [فصل ۲۴: Site، Application Pool و Binding](#فصل-۲۴)
- [فصل ۲۵: `web.config`، ماژول‌ها و handlerها](#فصل-۲۵)
- [فصل ۲۶: IIS و ASP.NET Core، URL Rewrite و ARR](#فصل-۲۶)

---

## فصل ۲۳: نصب IIS و سه راه مدیریت

### مقدمه

IIS بخشی از ویندوز سرور است، نه یک بسته‌ی جدا. نصبش با یک دستور PowerShell انجام می‌شود، ولی «نصب» و «پیکربندی درست» دو چیزند: بسیاری از نصب‌های پیش‌فرض، `Directory Browsing` و `WebDAV` را روشن می‌گذارند که هر دو سطح حمله‌اند.

### مفاهیم اساسی

**۱) نقش‌ها و ویژگی‌ها**

| ویژگی | نام |
|---|---|
| هسته‌ی وب | `Web-Server` |
| کنسول مدیریت | `Web-Mgmt-Console` |
| ASP.NET (کلاسیک) | `Web-Asp-Net45` |
| ASP.NET Core | Hosting Bundle (جدا) |
| WebSocket | `Web-WebSockets` |
| Redirect | `Web-Http-Redirect` |
| IP Restrictions | `Web-IP-Security` |
| URL Rewrite | افزونه‌ی جدا |
| ARR | افزونه‌ی جدا |
| Logging | `Web-Http-Logging` |
| Compression | `Web-Stat-Compression`، `Web-Dyn-Compression` |

**۲) نصب پایه**

```powershell
Install-WindowsFeature -Name Web-Server -IncludeManagementTools
```

**۳) نصب کامل و امن**

```powershell
$features = @(
    'Web-Server',
    'Web-WebServer',
    'Web-Common-Http',
    'Web-Default-Doc',
    'Web-Http-Errors',
    'Web-Static-Content',
    'Web-Http-Redirect',
    'Web-Health',
    'Web-Http-Logging',
    'Web-Performance',
    'Web-Stat-Compression',
    'Web-Dyn-Compression',
    'Web-Security',
    'Web-Filtering',
    'Web-IP-Security',
    'Web-App-Dev',
    'Web-WebSockets',
    'Web-Mgmt-Console',
    'Web-Scripting-Tools'
)

Install-WindowsFeature -Name $features -IncludeManagementTools
```

**۴) سه راه مدیریت**

| راه | مناسب |
|---|---|
| IIS Manager (GUI) | کار بصری، تنظیم سریع |
| `appcmd.exe` | اسکریپت، CI/CD |
| PowerShell (`WebAdministration`) | خودکارسازی، IaC |

**۵) `appcmd` — پرکاربردترین دستورها**

```cmd
appcmd list site
appcmd list apppool
appcmd list wp
appcmd list config /section:system.webServer/security/requestFiltering
appcmd set config /section:system.webServer/directoryBrowse /enabled:false
appcmd recycle apppool "lab-pool"
```

**۶) PowerShell — `WebAdministration`**

```powershell
Import-Module WebAdministration
Get-Website
Get-WebApplication
Get-IISAppPool
Get-WebBinding
Get-WebConfiguration -PSPath "IIS:\Sites\lab" -Filter "system.webServer/security/requestFiltering"
```

**۷) مسیرهای مهم**

| مورد | مسیر |
|---|---|
| `wwwroot` | `C:\inetpub\wwwroot` |
| لاگ | `C:\inetpub\logs\LogFiles\W3SVC<id>` |
| config سراسری | `C:\Windows\System32\inetsrv\config\applicationHost.config` |
| `web.config` سایت | `<site root>\web.config` |

**۸) `applicationHost.config`**

فایل XML مرکزی IIS. همه‌ی سایت‌ها، poolها، ماژول‌ها. **قبل از هر تغییر، بکاپ بگیر.**

```powershell
Copy-Item C:\Windows\System32\inetsrv\config\applicationHost.config `
  C:\backup\applicationHost.config.$(Get-Date -Format yyyyMMddHHmm)
```

**۹) بررسی سلامت**

```powershell
Get-Service W3SVC, WAS
Get-Website | Select-Object Name, State, PhysicalPath
Invoke-WebRequest http://localhost/ -UseBasicParsing | Select-Object StatusCode
```

**۱۰) `HTTP.sys` و `netsh`**

```cmd
netsh http show servicestate
netsh http show urlacl
netstat -ano | findstr :80
```

### مثال‌های عملی

**Lab Scenario: نصب و سخت‌سازی اولیه**

```powershell
# ۱) نصب
Install-WindowsFeature -Name Web-Server, Web-Mgmt-Console, Web-Http-Logging, `
  Web-Stat-Compression, Web-Dyn-Compression, Web-Filtering, Web-IP-Security, `
  Web-WebSockets, Web-Http-Redirect -IncludeManagementTools

# ۲) بررسی
Get-WindowsFeature Web-Server | Select-Object Name, InstallState
Get-Service W3SVC | Select-Object Status, StartType

# ۳) خاموش کردن ویژگی‌های خطرناک
Set-WebConfigurationProperty -PSPath "IIS:\" `
  -Filter "system.webServer/directoryBrowse" -Name "enabled" -Value $false

# حذف WebDAV
Remove-WindowsFeature Web-DAV-Publishing -ErrorAction SilentlyContinue

# ۴) پنهان کردن نسخه
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/security/requestFiltering" `
  -Name "removeServerHeader" -Value $true

# ۵) حذف X-Powered-By
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/httpProtocol/customHeaders" `
  -Name "." -Value @{name='X-Powered-By'; value=''}

# ۶) لاگ
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.applicationHost/sites/siteDefaults/logFile" `
  -Name "logFormat" -Value "W3C"

# ۷) فایروال
New-NetFirewallRule -DisplayName "IIS HTTP"  -Direction Inbound -Protocol TCP -LocalPort 80  -Action Allow
New-NetFirewallRule -DisplayName "IIS HTTPS" -Direction Inbound -Protocol TCP -LocalPort 443 -Action Allow

# ۸) تست
Invoke-WebRequest http://localhost/ -UseBasicParsing | Select-Object StatusCode, StatusDescription
```

**بررسی config مؤثر:**

```powershell
# همه‌ی تنظیمات سایت
Get-WebConfiguration -PSPath "IIS:\Sites\Default Web Site" -Filter "system.webServer/*" |
  Select-Object -First 20

# مقدار یک تنظیم
(Get-WebConfigurationProperty -PSPath "IIS:\Sites\Default Web Site" `
  -Filter "system.webServer/security/requestFiltering/requestLimits" `
  -Name "maxAllowedContentLength").Value
```

**بکاپ و بازیابی config:**

```powershell
# بکاپ
& $env:windir\system32\inetsrv\appcmd.exe add backup "before-hardening"

# لیست
& $env:windir\system32\inetsrv\appcmd.exe list backup

# بازیابی
& $env:windir\system32\inetsrv\appcmd.exe restore backup "before-hardening"
```

**`failed request tracing` برای دیباگ:**

```powershell
Install-WindowsFeature Web-Http-Tracing
Set-WebConfigurationProperty -PSPath "IIS:\Sites\lab" `
  -Filter "system.webServer/tracing/traceFailedRequests" -Name "." -Value @{path='C:\inetpub\logs\FailedReqLogs'}

Add-WebConfiguration -PSPath "IIS:\Sites\lab" `
  -Filter "system.webServer/tracing/traceFailedRequests" `
  -Value @{path='*.aspx'; statusCodes='500'}
```

### مقایسه و الگوهای مختلف

**سه راه مدیریت**

| | GUI | `appcmd` | PowerShell |
|---|---|---|---|
| سرعت یادگیری | ✅✅ | ⚠️ | ⚠️ |
| خودکارسازی | ❌ | ✅ | ✅✅ |
| مستندسازی | ❌ | ✅ | ✅ |
| CI/CD | ❌ | ✅ | ✅ |
| مناسب | شروع | اسکریپت | IaC |

**`Web-Server` حداقلی در برابر کامل**

| | حداقلی | کامل |
|---|---|---|
| سطح حمله | ✅ کم | ⚠️ |
| قابلیت | ⚠️ | ✅ |
| توصیه | ✅ شروع | بر اساس نیاز |

**IIS در برابر Nginx در نصب**

| | IIS | Nginx |
|---|---|---|
| نصب | ویژگی ویندوز | بسته |
| config | XML | متن |
| reload | `iisreset`/recycle | `-s reload` |
| نسخه‌بندی config | ⚠️ | ✅ git |

**دیدگاه «IIS فقط با GUI»**: GUI برای شروع خوب است، ولی هر تغییری که باید تکرار شود، باید اسکریپت باشد. `appcmd` و PowerShell، همان کار را مستند و تکرارپذیر می‌کنند.

### اشکال شایع

**۱) `HTTP Error 500.19`**
`web.config` نامعتبر یا ماژول نصب‌نشده. `%windir%\system32\inetsrv\config` و `web.config` را چک کن.

**۲) `HTTP Error 403.14`**
`Directory Browsing` خاموش و `defaultDocument` تنظیم نشده. `index.html` اضافه کن.

**۳) `HTTP Error 404.3`**
MIME type تعریف نشده (مثلاً `.woff2`). `staticContent/mimeMap`.

**۴) `W3SVC` شروع نمی‌شود**
پورت ۸۰ اشغال، یا `applicationHost.config` خراب. `netstat -ano`.

**۵) `appcmd` در PowerShell کار نمی‌کند**
مسیر کامل: `& $env:windir\system32\inetsrv\appcmd.exe`.

**۶) `WebAdministration` لود نمی‌شود**
`Import-Module WebAdministration` و اجرا با Administrator.

**۷) `Directory Browsing` روشن**
افشای ساختار. `Set-WebConfigurationProperty`.

**۸) `WebDAV` روشن**
`PUT`/`DELETE` فعال. حذف ویژگی.

**۹) `X-Powered-By` باقی‌مانده**
`customHeaders` + `removeServerHeader`.

**۱۰) نبود بکاپ `applicationHost.config`**
تغییر اشتباه، کل IIS را می‌خواباند. `appcmd add backup`.

### نکات مهم (Tips & Tricks)

- **`appcmd add backup`** قبل از هر تغییر بزرگ.
- **`Get-WebConfiguration`** مقدار مؤثر را می‌دهد.
- **`removeServerHeader`** برای پنهان کردن نسخه.
- **`Web-Http-Tracing`** برای دیباگ ۵۰۰.
- **`netsh http show servicestate`** برای `HTTP.sys`.
- **`iisreset /noforce`** به‌جای `iisreset` (ملایم‌تر).
- **`Web-Server` حداقلی** شروع کن.

### خلاصه فصل

- IIS با `Install-WindowsFeature` نصب می‌شود.
- سه راه مدیریت: GUI، `appcmd`، PowerShell.
- `applicationHost.config` قلب config است؛ بکاپ بگیر.
- `Directory Browsing` و `WebDAV` را خاموش کن.
- `removeServerHeader` و حذف `X-Powered-By`.
- `failed request tracing` برای دیباگ عمیق.

**سؤال مروری:** چرا `appcmd add backup` قبل از هر تغییر در IIS ضروری است؟

---

## فصل ۲۴: Site، Application Pool و Binding

### مقدمه

سه مفهوم کلیدی IIS: **Site** (چه چیزی سرو شود)، **Application Pool** (با چه هویت و منابعی)، **Binding** (از کدام آدرس). اشتباه در هر کدام، به `503`، `403` یا سرویس اشتباه منجر می‌شود.

### مفاهیم اساسی

**۱) Site**

```
Site "lab"
├── Physical Path: C:\inetpub\lab
├── Application Pool: lab-pool
├── Bindings:
│   ├── http  *:80:lab.example.com
│   └── https *:443:lab.example.com (SNI)
└── Applications:
    ├── /        → C:\inetpub\lab
    └── /api     → C:\inetpub\lab\api
```

**۲) Application Pool**

| تنظیم | معنا |
|---|---|
| `.NET CLR Version` | `v4.0` برای ASP.NET، `No Managed Code` برای ASP.NET Core/PHP |
| `Identity` | `ApplicationPoolIdentity` (پیش‌فرض، امن) |
| `Start Mode` | `AlwaysRunning` برای جلوگیری از cold start |
| `Idle Time-out` | بستن بعد از بی‌کاری (پیش‌فرض ۲۰ دقیقه) |
| `Regular Time Interval` | بازیافت دوره‌ای (پیش‌فرض ۱۷۴۰ دقیقه) |
| `Private Memory Limit` | سقف حافظه |

**۳) `ApplicationPoolIdentity`**

هر pool یک هویت مجازی `IIS APPPOOL\<poolname>` دارد. برای دسترسی به فایل:

```powershell
icacls C:\inetpub\lab /grant "IIS APPPOOL\lab-pool:(OI)(CI)(RX)"
```

**۴) Binding**

| فیلد | مثال |
|---|---|
| Protocol | `http`/`https`/`net.tcp` |
| IP | `*` یا `192.168.56.10` |
| Port | `80`، `443`، `8080` |
| Host name | `lab.example.com` |

**۵) SNI و `SslFlags`**

| مقدار | معنا |
|---|---|
| `0` | بدون SNI (IP-based) |
| `1` | SNI |
| `2` | Central Certificate Store |
| `3` | SNI + Central Cert Store |

**۶) Application (زیربرنامه)**

```powershell
New-WebApplication -Site "lab" -Name "api" -PhysicalPath "C:\inetpub\lab\api" -ApplicationPool "lab-api-pool"
```

`/api` به pool جدا و پوشه‌ی جدا.

**۷) Virtual Directory**

```powershell
New-WebVirtualDirectory -Site "lab" -Name "files" -PhysicalPath "D:\shared\files"
```

پوشه‌ای که در URL ظاهر می‌شود ولی زیر `PhysicalPath` نیست.

**۸) Recycling**

```powershell
# بازیافت دستی
Restart-WebAppPool -Name "lab-pool"

# زمان‌بندی
Set-ItemProperty "IIS:\AppPools\lab-pool" -Name recycling.periodicRestart.time -Value "1.00:00:00"

# بر اساس حافظه
Set-ItemProperty "IIS:\AppPools\lab-pool" -Name recycling.periodicRestart.privateMemory -Value 1048576
```

**۹) `Idle Time-out` و cold start**

```powershell
# جلوگیری از خوابیدن
Set-ItemProperty "IIS:\AppPools\lab-pool" -Name processModel.idleTimeout -Value "00:00:00"
Set-ItemProperty "IIS:\AppPools\lab-pool" -Name startMode -Value "AlwaysRunning"

# preload سایت
Set-ItemProperty "IIS:\Sites\lab" -Name applicationDefaults.preloadEnabled -Value $true
```

**۱۰) `appcmd` معادل‌ها**

```cmd
appcmd add site /name:lab /physicalPath:C:\inetpub\lab /bindings:"http/*:80:lab.example.com"
appcmd add apppool /name:lab-pool
appcmd set app "lab/" /applicationPool:lab-pool
appcmd start site lab
appcmd stop apppool lab-pool
```

### مثال‌های عملی

**Lab Scenario: دو سایت، سه pool**

```powershell
# ۱) پوشه‌ها
New-Item -ItemType Directory -Path C:\inetpub\lab -Force | Out-Null
New-Item -ItemType Directory -Path C:\inetpub\api -Force | Out-Null
Set-Content C:\inetpub\lab\index.html '<h1>Lab</h1>'
Set-Content C:\inetpub\api\index.html '<h1>API</h1>'

# ۲) poolها
New-WebAppPool -Name "lab-pool"
New-WebAppPool -Name "api-pool"
Set-ItemProperty "IIS:\AppPools\lab-pool" -Name managedRuntimeVersion -Value ""
Set-ItemProperty "IIS:\AppPools\api-pool" -Name managedRuntimeVersion -Value ""

# ۳) سایت‌ها
New-Website -Name "lab" -PhysicalPath C:\inetpub\lab `
  -ApplicationPool "lab-pool" -Port 80 -HostHeader "lab.example.com"

New-Website -Name "api" -PhysicalPath C:\inetpub\api `
  -ApplicationPool "api-pool" -Port 80 -HostHeader "api.lab.example.com"

# ۴) مجوز
icacls C:\inetpub\lab /grant "IIS APPPOOL\lab-pool:(OI)(CI)(RX)" | Out-Null
icacls C:\inetpub\api /grant "IIS APPPOOL\api-pool:(OI)(CI)(RX)" | Out-Null

# ۵) بررسی
Get-Website | Format-Table Name, State, PhysicalPath, ApplicationPool -AutoSize
Get-WebBinding
Get-IISAppPool | Format-Table Name, State, ManagedRuntimeVersion -AutoSize

# ۶) تست
Invoke-WebRequest http://lab.example.com/ -UseBasicParsing | Select-Object -ExpandProperty Content
Invoke-WebRequest http://api.lab.example.com/ -UseBasicParsing | Select-Object -ExpandProperty Content
```

**افزودن HTTPS با SNI:**

```powershell
$cert = Get-ChildItem Cert:\LocalMachine\My |
        Where-Object { $_.Subject -like "*lab.example.com*" } |
        Select-Object -First 1

if (-not $cert) { throw "گواهی پیدا نشد" }

New-WebBinding -Name "lab" -Protocol https -Port 443 `
  -HostHeader "lab.example.com" -SslFlags 1

(Get-WebBinding -Name "lab" -Protocol https -HostHeader "lab.example.com").
  AddSslCertificate($cert.Thumbprint, "my")

Get-WebBinding -Name "lab" | Format-Table protocol, bindingInformation, sslFlags
```

**Application زیر `/api`:**

```powershell
New-WebApplication -Site "lab" -Name "v1" `
  -PhysicalPath "C:\inetpub\lab\v1" -ApplicationPool "api-pool"

# بررسی
Get-WebApplication -Site "lab"
Invoke-WebRequest http://lab.example.com/v1/ -UseBasicParsing | Select-Object StatusCode
```

**Virtual Directory:**

```powershell
New-Item -ItemType Directory -Path D:\shared\files -Force | Out-Null
Set-Content D:\shared\files\readme.txt 'shared'

New-WebVirtualDirectory -Site "lab" -Name "files" -PhysicalPath "D:\shared\files"
icacls D:\shared\files /grant "IIS APPPOOL\lab-pool:(OI)(CI)(RX)" | Out-Null

Invoke-WebRequest http://lab.example.com/files/readme.txt -UseBasicParsing |
  Select-Object -ExpandProperty Content
```

**جلوگیری از cold start:**

```powershell
foreach ($p in @("lab-pool","api-pool")) {
    Set-ItemProperty "IIS:\AppPools\$p" -Name processModel.idleTimeout -Value "00:00:00"
    Set-ItemProperty "IIS:\AppPools\$p" -Name startMode -Value "AlwaysRunning"
    Set-ItemProperty "IIS:\AppPools\$p" -Name recycling.periodicRestart.time -Value "00:00:00"
}
```

**محدودیت منابع:**

```powershell
# سقف حافظه ۱ گیگ
Set-ItemProperty "IIS:\AppPools\lab-pool" -Name recycling.periodicRestart.privateMemory -Value 1048576

# سقف CPU (نیاز به تنظیم در applicationHost.config)
Set-WebConfigurationProperty -PSPath "IIS:\AppPools\lab-pool" `
  -Filter "cpu" -Name "limit" -Value 50000       # ۵۰٪
Set-WebConfigurationProperty -PSPath "IIS:\AppPools\lab-pool" `
  -Filter "cpu" -Name "action" -Value "Throttle"
```

**`appcmd` برای بازرسی:**

```cmd
appcmd list site
appcmd list apppool /state:Started
appcmd list wp
appcmd list app "lab/"
appcmd list vdir "lab/"
```

### مقایسه و الگوهای مختلف

**`ApplicationPoolIdentity` در برابر کاربر سفارشی**

| | `ApplicationPoolIdentity` | کاربر سفارشی |
|---|---|---|
| امنیت | ✅ | ⚠️ |
| مدیریت رمز | ✅ (بدون رمز) | ❌ |
| دسترسی شبکه | ⚠️ (محدود) | ✅ |
| توصیه | ✅ | فقط نیاز شبکه |

**`Idle Time-out` صفر در برابر پیش‌فرض**

| | `0` | `20 min` |
|---|---|---|
| cold start | ✅ ندارد | ❌ دارد |
| حافظه | ❌ همیشه اشغال | ✅ آزاد |
| مناسب | سایت پرترافیک | سایت کم‌ترافیک |

**Site در برابر Application در برابر Virtual Directory**

| | Site | Application | VDir |
|---|---|---|---|
| Binding | ✅ | ❌ | ❌ |
| Pool جدا | ✅ | ✅ | ❌ |
| URL | دامنه | زیرمسیر | زیرمسیر |
| مناسب | دامنه‌ی مستقل | اپ جدا | پوشه‌ی مشترک |

**دیدگاه «یک pool برای همه»**: ساده است، ولی خرابی/نشت حافظه‌ی یک اپ، همه را می‌خواباند. جداسازی، پایه‌ی پایداری است.

### اشکال شایع

**۱) `503 Service Unavailable`**
pool متوقف یا `rapid-fail protection`. `Get-IISAppPool` و Event Log.

**۲) `403.14`**
`defaultDocument` تنظیم نشده. `index.html`.

**۳) `401.3` — Unauthorized**
مجوز NTFS برای `IIS APPPOOL\<pool>`. `icacls`.

**۴) `500.19`**
`web.config` نامعتبر. `appcmd list config`.

**۵) دو سایت روی یک binding**
`socket` تعارض. `Get-WebBinding` و `HostHeader`.

**۶) SNI خاموش**
گواهی اشتباه. `SslFlags 1`.

**۷) `App Pool` با `No Managed Code` برای ASP.NET کلاسیک**
`500.30` یا خطای ماژول. `v4.0`.

**۸) `Idle Time-out` و کندی اول صبح**
cold start. `AlwaysRunning` + `preloadEnabled`.

**۹) بازیافت وسط درخواست**
کاربر قطع می‌شود. `overlapping recycle` (پیش‌فرض روشن).

**۱۰) `PhysicalPath` با مجوز شبکه**
`401.3`. هویت pool باید دسترسی `Read` داشته باشد.

### نکات مهم (Tips & Tricks)

- **`Get-WebBinding`** برای تعارض.
- **`icacls`** با `IIS APPPOOL\`.
- **`AlwaysRunning`** برای جلوگیری از cold start.
- **`SslFlags 1`** برای SNI.
- **`appcmd list wp`** برای worker process.
- **Event Viewer** برای خطای pool.
- **`New-WebApplication`** برای اپ جدا.

### خلاصه فصل

- Site = محتوا + binding + pool.
- Application Pool = هویت و منابع.
- Binding = پروتکل + IP + پورت + hostname.
- `SslFlags 1` برای SNI.
- `ApplicationPoolIdentity` امن‌ترین هویت است.
- `AlwaysRunning` cold start را حذف می‌کند.

**سؤال مروری:** چرا دو سایت با یک `HostHeader` و یک پورت نمی‌توانند همزمان کار کنند؟

---

## فصل ۲۵: `web.config`، ماژول‌ها و handlerها

### مقدمه

`web.config` معادل `.htaccess` است، ولی XML و با ساختار سلسله‌مراتبی (ماشین → سایت → اپ → پوشه). قدرتش در ماژول‌ها و handlerهاست: هر درخواست از یک زنجیره‌ی ماژول عبور می‌کند و handler نهایی، پاسخ را می‌سازد.

### مفاهیم اساسی

**۱) ساختار**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    <!-- تنظیمات IIS -->
  </system.webServer>
  <system.web>          <!-- ASP.NET کلاسیک -->
  </system.web>
  <appSettings>         <!-- اپ -->
  </appSettings>
</configuration>
```

**۲) سطوح config**

```
machine.config
  └── applicationHost.config
        └── web.config (سایت)
              └── web.config (اپ)
                    └── web.config (پوشه)
```

هر سطح، سطح قبلی را override می‌کند (مگر `overrideModeDefault="Deny"`).

**۳) بخش‌های مهم `system.webServer`**

| بخش | کار |
|---|---|
| `defaultDocument` | فایل پیش‌فرض |
| `directoryBrowse` | لیست پوشه |
| `httpErrors` | صفحه‌ی خطا |
| `httpProtocol/customHeaders` | هدرها |
| `security/requestFiltering` | محدودیت درخواست |
| `security/ipSecurity` | محدودیت IP |
| `staticContent` | MIME و کش |
| `rewrite` | URL Rewrite |
| `modules` | ماژول‌ها |
| `handlers` | handlerها |
| `httpCompression` | فشرده‌سازی |
| `webSocket` | WebSocket |
| `aspNetCore` | ASP.NET Core |

**۴) `requestFiltering` — مهم‌ترین بخش امنیتی**

```xml
<security>
  <requestFiltering>
    <requestLimits maxAllowedContentLength="16777216"
                   maxUrl="4096"
                   maxQueryString="2048" />
    <hiddenSegments>
      <add segment=".git" />
      <add segment=".env" />
      <add segment="appsettings.json" />
      <add segment="web.config" />
    </hiddenSegments>
    <fileExtensions>
      <add fileExtension=".config" allowed="false" />
      <add fileExtension=".bak" allowed="false" />
      <add fileExtension=".sql" allowed="false" />
    </fileExtensions>
    <verbs>
      <add verb="TRACE" allowed="false" />
      <add verb="TRACK" allowed="false" />
    </verbs>
  </requestFiltering>
</security>
```

**۵) `httpErrors`**

```xml
<httpErrors errorMode="DetailedLocalOnly" existingResponse="Replace">
  <remove statusCode="404" />
  <error statusCode="404" path="/404.html" responseMode="File" />
  <remove statusCode="500" />
  <error statusCode="500" path="/500.html" responseMode="File" />
</httpErrors>
```

| `errorMode` | رفتار |
|---|---|
| `DetailedLocalOnly` | جزئیات برای local، ساده برای بیرون |
| `Detailed` | جزئیات برای همه (❌ تولید) |
| `Custom` | صفحه‌ی سفارشی |

**۶) `staticContent`**

```xml
<staticContent>
  <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="7.00:00:00" />
  <remove fileExtension=".woff2" />
  <mimeMap fileExtension=".woff2" mimeType="font/woff2" />
  <remove fileExtension=".webp" />
  <mimeMap fileExtension=".webp" mimeType="image/webp" />
</staticContent>
```

**۷) `modules` و `handlers`**

```xml
<handlers>
  <add name="StaticFile" path="*" verb="*" modules="StaticFileModule"
       resourceType="File" requireAccess="Read" />
</handlers>

<modules runAllManagedModulesForAllRequests="false">
  <remove name="WebDAVModule" />
</modules>
```

`runAllManagedModulesForAllRequests="true"` = هر درخواست (حتی استاتیک) از ماژول‌های .NET عبور می‌کند. **کند و ناامن.**

**۸) `httpCompression`**

```xml
<httpCompression directory="%SystemDrive%\inetpub\temp\IIS Temporary Compressed Files">
  <scheme name="gzip" dll="%Windir%\system32\inetsrv\gzip.dll" staticCompressionLevel="9" />
  <dynamicTypes>
    <add mimeType="text/*" enabled="true" />
    <add mimeType="application/json" enabled="true" />
    <add mimeType="application/javascript" enabled="true" />
    <add mimeType="*/*" enabled="false" />
  </dynamicTypes>
  <staticTypes>
    <add mimeType="text/*" enabled="true" />
    <add mimeType="application/javascript" enabled="true" />
    <add mimeType="image/svg+xml" enabled="true" />
    <add mimeType="*/*" enabled="false" />
  </staticTypes>
</httpCompression>
```

**۹) `ipSecurity`**

```xml
<security>
  <ipSecurity allowUnlisted="false">
    <add ipAddress="192.168.56.0" subnetMask="255.255.255.0" allowed="true" />
    <add ipAddress="203.0.113.0" subnetMask="255.255.255.0" allowed="false" />
  </ipSecurity>
</security>
```

**۱۰) `location` در `web.config`**

```xml
<location path="admin">
  <system.webServer>
    <security>
      <ipSecurity allowUnlisted="false">
        <add ipAddress="192.168.56.0" subnetMask="255.255.255.0" allowed="true" />
      </ipSecurity>
    </security>
  </system.webServer>
</location>
```

### مثال‌های عملی

**Lab Scenario: `web.config` کامل و امن**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    <!-- ۱) فایل پیش‌فرض -->
    <defaultDocument enabled="true">
      <files>
        <clear />
        <add value="index.html" />
        <add value="default.htm" />
      </files>
    </defaultDocument>

    <!-- ۲) لیست پوشه خاموش -->
    <directoryBrowse enabled="false" />

    <!-- ۳) هدرهای امنیتی -->
    <httpProtocol>
      <customHeaders>
        <remove name="X-Powered-By" />
        <remove name="X-AspNet-Version" />
        <add name="X-Content-Type-Options" value="nosniff" />
        <add name="X-Frame-Options" value="SAMEORIGIN" />
        <add name="Referrer-Policy" value="strict-origin-when-cross-origin" />
        <add name="Permissions-Policy" value="geolocation=(), microphone=(), camera=()" />
        <add name="Strict-Transport-Security" value="max-age=31536000; includeSubDomains" />
      </customHeaders>
    </httpProtocol>

    <!-- ۴) محدودیت درخواست -->
    <security>
      <requestFiltering removeServerHeader="true">
        <requestLimits maxAllowedContentLength="16777216"
                       maxUrl="4096"
                       maxQueryString="2048" />
        <hiddenSegments>
          <add segment=".git" />
          <add segment=".env" />
          <add segment="appsettings.json" />
          <add segment="bin" />
        </hiddenSegments>
        <fileExtensions>
          <add fileExtension=".config" allowed="false" />
          <add fileExtension=".bak" allowed="false" />
          <add fileExtension=".sql" allowed="false" />
          <add fileExtension=".log" allowed="false" />
        </fileExtensions>
        <verbs>
          <add verb="TRACE" allowed="false" />
          <add verb="TRACK" allowed="false" />
          <add verb="DEBUG" allowed="false" />
        </verbs>
      </requestFiltering>
    </security>

    <!-- ۵) خطاها -->
    <httpErrors errorMode="DetailedLocalOnly" existingResponse="Replace">
      <remove statusCode="404" />
      <error statusCode="404" path="/404.html" responseMode="File" />
      <remove statusCode="500" />
      <error statusCode="500" path="/500.html" responseMode="File" />
    </httpErrors>

    <!-- ۶) MIME و کش -->
    <staticContent>
      <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="7.00:00:00" />
      <remove fileExtension=".woff2" />
      <mimeMap fileExtension=".woff2" mimeType="font/woff2" />
      <remove fileExtension=".webp" />
      <mimeMap fileExtension=".webp" mimeType="image/webp" />
      <remove fileExtension=".json" />
      <mimeMap fileExtension=".json" mimeType="application/json" />
    </staticContent>

    <!-- ۷) حذف ماژول‌های خطرناک -->
    <modules runAllManagedModulesForAllRequests="false">
      <remove name="WebDAVModule" />
    </modules>

    <!-- ۸) فشرده‌سازی -->
    <urlCompression doStaticCompression="true" doDynamicCompression="true" />

    <!-- ۹) WebSocket -->
    <webSocket enabled="true" pingInterval="00:00:30" />
  </system.webServer>
</configuration>
```

```powershell
# اعمال و تست
iisreset /noforce

# بررسی هدرها
curl.exe -sSI http://lab.example.com/ | Select-String "X-|Strict|Server"

# بررسی فایل مخفی
curl.exe -sS -o NUL -w "%{http_code}`n" http://lab.example.com/.env
# 404

# بررسی Directory Browsing
curl.exe -sS -o NUL -w "%{http_code}`n" http://lab.example.com/empty/
# 403
```

**`web.config` برای API با CORS:**

```xml
<system.webServer>
  <httpProtocol>
    <customHeaders>
      <add name="Access-Control-Allow-Origin" value="https://app.example.com" />
      <add name="Access-Control-Allow-Methods" value="GET, POST, PUT, DELETE, OPTIONS" />
      <add name="Access-Control-Allow-Headers" value="Content-Type, Authorization" />
      <add name="Access-Control-Max-Age" value="86400" />
    </customHeaders>
  </httpProtocol>
</system.webServer>
```

> `OPTIONS` باید توسط اپ یا ماژول مدیریت شود؛ IIS خودش پاسخ نمی‌دهد.

**محدودیت IP برای `/admin`:**

```xml
<location path="admin">
  <system.webServer>
    <security>
      <ipSecurity allowUnlisted="false">
        <add ipAddress="192.168.56.0" subnetMask="255.255.255.0" allowed="true" />
      </ipSecurity>
    </security>
  </system.webServer>
</location>
```

```powershell
# فعال‌سازی ماژول
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/security/ipSecurity" -Name "allowUnlisted" -Value $false
```

**بازرسی config مؤثر:**

```powershell
# مقدار یک بخش
Get-WebConfiguration -PSPath "IIS:\Sites\lab" -Filter "system.webServer/security/requestFiltering"

# همه‌ی هدرها
Get-WebConfigurationProperty -PSPath "IIS:\Sites\lab" `
  -Filter "system.webServer/httpProtocol/customHeaders" -Name "collection"
```

**`appcmd` برای config:**

```cmd
appcmd list config "lab" /section:system.webServer/security/requestFiltering
appcmd set config "lab" /section:system.webServer/directoryBrowse /enabled:false
appcmd set config "lab" /section:system.webServer/httpErrors /errorMode:DetailedLocalOnly
```

### مقایسه و الگوهای مختلف

**`web.config` در برابر `.htaccess`**

| | `web.config` | `.htaccess` |
|---|---|---|
| فرمت | XML | متن |
| سرعت | ✅ (cache) | ❌ |
| ساختار | سلسله‌مراتبی | ارث‌بری |
| ماژول | ✅ | ✅ |
| محدودیت | ⚠️ (`overrideMode`) | ✅ |

**`runAllManagedModulesForAllRequests`**

| | `true` | `false` |
|---|---|---|
| سرعت استاتیک | ❌ | ✅ |
| سازگاری | ✅ | ⚠️ |
| توصیه | ❌ | ✅ |

**`errorMode`**

| | `Detailed` | `Custom` | `DetailedLocalOnly` |
|---|---|---|---|
| امنیت | ❌ | ✅ | ✅ |
| دیباگ | ✅ | ❌ | ✅ محلی |
| تولید | ❌ | ✅ | ✅ |

**دیدگاه «`web.config` فقط برای ASP.NET»**: نه. IIS از آن برای همه‌چیز (MIME، امنیت، rewrite) استفاده می‌کند.

### اشکال شایع

**۱) `500.19` با `config Error`**
بخش قفل‌شده (`overrideModeDefault="Deny"`). در `applicationHost.config` باز کن یا از `appcmd` استفاده کن.

**۲) `runAllManagedModulesForAllRequests="true"`**
کندی. `false`.

**۳) `httpErrors` با `Detailed`**
افشای stack trace. `DetailedLocalOnly`.

**۴) `requestFiltering` بدون `removeServerHeader`**
نسخه لو می‌رود.

**۵) `.woff2` با `404.3`**
MIME تعریف نشده. `mimeMap`.

**۶) `ipSecurity` بدون نصب ماژول**
`500.19`. `Install-WindowsFeature Web-IP-Security`.

**۷) `WebDAVModule` باقی‌مانده**
`PUT`/`DELETE` فعال. `remove`.

**۸) `clientCache` برای HTML**
کش طولانی صفحه‌ی داینامیک. `max-age=0` برای HTML.

**۹) `customHeaders` تکراری**
دو `X-Frame-Options` → مرورگر رد می‌کند. `remove` اول.

**۱۰) `location path` با `/` ابتدایی**
`path="/admin"` یا `path="admin"`؟ هر دو کار می‌کنند، ولی یکدست باش.

### نکات مهم (Tips & Tricks)

- **`removeServerHeader`**.
- **`runAllManagedModulesForAllRequests="false"`**.
- **`httpErrors errorMode="DetailedLocalOnly"`**.
- **`hiddenSegments`** برای `.git` و `.env`.
- **`mimeMap`** برای فرمت‌های جدید.
- **`appcmd list config`** برای بازرسی.
- **`web.config` را در git نگه دار**.

### خلاصه فصل

- `web.config` XML و سلسله‌مراتبی است.
- `requestFiltering` مهم‌ترین بخش امنیتی.
- `httpErrors` با `DetailedLocalOnly`.
- `runAllManagedModulesForAllRequests="false"`.
- `mimeMap` برای فرمت‌های جدید.
- `location` برای تنظیم per-path.

**سؤال مروری:** چرا `runAllManagedModulesForAllRequests="true"` هم کند است و هم سطح حمله را زیاد می‌کند؟

---

## فصل ۲۶: IIS و ASP.NET Core، URL Rewrite و ARR

### مقدمه

IIS در دنیای مدرن، دو نقش اصلی دارد: میزبان ASP.NET Core (با `ANCM`) و reverse proxy (با ARR). این فصل، هر دو را با پیکربندی کامل می‌گوید.

### مفاهیم اساسی

**۱) `ANCM` — ASP.NET Core Module**

```
کلاینت → HTTP.sys → IIS (ANCM) → Kestrel (:5000 یا named pipe) → اپ
```

`ANCM` یک reverse proxy سبک است که درون IIS اجرا می‌شود.

**۲) نصب Hosting Bundle**

```
https://dotnet.microsoft.com/download/dotnet/8.0 → ASP.NET Core Hosting Bundle
```

بعد از نصب: `iisreset`.

```powershell
# بررسی
Get-WebGlobalModule | Where-Object { $_.Name -like "AspNetCoreModule*" }
```

**۳) `web.config` برای ASP.NET Core**

```xml
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*"
             modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="dotnet"
                  arguments=".\Lab.Api.dll"
                  stdoutLogEnabled="false"
                  stdoutLogFile=".\logs\stdout"
                  hostingModel="inprocess" />
    </system.webServer>
  </location>
</configuration>
```

| `hostingModel` | معنا |
|---|---|
| `inprocess` | اپ در همان پروسه‌ی `w3wp` (سریع‌تر) |
| `outofprocess` | `Kestrel` جدا + proxy |

**۴) `inprocess` در برابر `outofprocess`**

| | `inprocess` | `outofprocess` |
|---|---|---|
| کارایی | ✅ بالاتر | ⚠️ |
| دیباگ | ⚠️ | ✅ |
| پشتیبانی از `Kestrel` features | ⚠️ | ✅ |
| توصیه | ✅ تولید | توسعه/نیاز خاص |

**۵) URL Rewrite — نصب**

```
https://www.iis.net/downloads/microsoft/url-rewrite
```

```powershell
# بررسی نصب
Get-WebGlobalModule | Where-Object { $_.Name -eq "RewriteModule" }
```

**۶) قواعد رایج**

```xml
<rewrite>
  <rules>
    <!-- HTTPS -->
    <rule name="Force HTTPS" stopProcessing="true">
      <match url="(.*)" />
      <conditions>
        <add input="{HTTPS}" pattern="off" />
      </conditions>
      <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" redirectType="Permanent" />
    </rule>

    <!-- حذف www -->
    <rule name="Remove WWW" stopProcessing="true">
      <match url="(.*)" />
      <conditions>
        <add input="{HTTP_HOST}" pattern="^www\.(.*)$" />
      </conditions>
      <action type="Redirect" url="https://{C:1}/{R:1}" redirectType="Permanent" />
    </rule>

    <!-- SPA fallback -->
    <rule name="SPA" stopProcessing="true">
      <match url=".*" />
      <conditions logicalGrouping="MatchAll">
        <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
        <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
      </conditions>
      <action type="Rewrite" url="/index.html" />
    </rule>
  </rules>
</rewrite>
```

**۷) ARR — Application Request Routing**

افزونه‌ای که IIS را به reverse proxy تبدیل می‌کند.

```
https://www.iis.net/downloads/microsoft/application-request-routing
```

```powershell
# فعال‌سازی proxy
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/proxy" -Name "enabled" -Value $true

# health check
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/proxy" -Name "healthCheckInterval" -Value "00:00:30"
```

**۸) Reverse proxy با ARR**

```xml
<rewrite>
  <rules>
    <rule name="ReverseProxy" stopProcessing="true">
      <match url="(.*)" />
      <action type="Rewrite" url="http://127.0.0.1:3000/{R:1}" />
      <serverVariables>
        <set name="HTTP_X_FORWARDED_PROTO" value="{HTTPS}" />
        <set name="HTTP_X_FORWARDED_FOR" value="{REMOTE_ADDR}" />
      </serverVariables>
    </rule>
  </rules>
</rewrite>
```

```powershell
# اجازه‌ی تنظیم server variable
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/rewrite/allowedServerVariables" `
  -Name "." -Value @{name='HTTP_X_FORWARDED_PROTO'}
```

**۹) Load balancing با ARR**

```powershell
# سرور farm
New-WebFarm -Name "lab-farm" -Servers @("10.0.0.11","10.0.0.12")
Add-WebFarmServer -Name "lab-farm" -ServerName "10.0.0.13"
Get-WebFarm
```

```xml
<rewrite>
  <rules>
    <rule name="LB" stopProcessing="true">
      <match url="(.*)" />
      <action type="Rewrite" url="http://lab-farm/{R:1}" />
    </rule>
  </rules>
</rewrite>
```

**۱۰) `ANCM` و `stdout` log**

```xml
<aspNetCore processPath="dotnet" arguments=".\Lab.Api.dll"
            stdoutLogEnabled="true" stdoutLogFile=".\logs\stdout"
            hostingModel="inprocess" />
```

```powershell
# پوشه‌ی لاگ و مجوز
New-Item -ItemType Directory -Path C:\inetpub\lab\logs -Force
icacls C:\inetpub\lab\logs /grant "IIS APPPOOL\lab-pool:(OI)(CI)(M)"
```

### مثال‌های عملی

**Lab Scenario: ASP.NET Core روی IIS با HTTPS**

```powershell
# ۱) نصب Hosting Bundle (دستی از سایت مایکروسافت)
# سپس:
iisreset /noforce
Get-WebGlobalModule | Where-Object Name -like "AspNetCore*"

# ۲) publish
dotnet publish -c Release -o C:\inetpub\lab-api

# ۳) pool بدون managed runtime
New-WebAppPool -Name "lab-api-pool"
Set-ItemProperty "IIS:\AppPools\lab-api-pool" -Name managedRuntimeVersion -Value ""
Set-ItemProperty "IIS:\AppPools\lab-api-pool" -Name processModel.idleTimeout -Value "00:00:00"
Set-ItemProperty "IIS:\AppPools\lab-api-pool" -Name startMode -Value "AlwaysRunning"

# ۴) سایت
New-Website -Name "lab-api" -PhysicalPath C:\inetpub\lab-api `
  -ApplicationPool "lab-api-pool" -Port 8080 -HostHeader "api.lab.example.com"

# ۵) مجوز
icacls C:\inetpub\lab-api /grant "IIS APPPOOL\lab-api-pool:(OI)(CI)(RX)" | Out-Null

# ۶) تست
Invoke-WebRequest http://api.lab.example.com:8080/health -UseBasicParsing |
  Select-Object StatusCode, Content
```

**`web.config` تولیدی ASP.NET Core:**

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*"
             modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="dotnet"
                  arguments=".\Lab.Api.dll"
                  stdoutLogEnabled="false"
                  stdoutLogFile=".\logs\stdout"
                  hostingModel="inprocess">
        <environmentVariables>
          <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Production" />
          <environmentVariable name="ASPNETCORE_FORWARDEDHEADERS_ENABLED" value="true" />
        </environmentVariables>
      </aspNetCore>
      <security>
        <requestFiltering removeServerHeader="true">
          <requestLimits maxAllowedContentLength="10485760" />
        </requestFiltering>
      </security>
      <httpProtocol>
        <customHeaders>
          <remove name="X-Powered-By" />
          <add name="X-Content-Type-Options" value="nosniff" />
          <add name="X-Frame-Options" value="SAMEORIGIN" />
        </customHeaders>
      </httpProtocol>
    </system.webServer>
  </location>
</configuration>
```

**HTTPS با SNI:**

```powershell
$cert = Get-ChildItem Cert:\LocalMachine\My | Where-Object { $_.Subject -like "*api.lab.example.com*" }
New-WebBinding -Name "lab-api" -Protocol https -Port 443 -HostHeader "api.lab.example.com" -SslFlags 1
(Get-WebBinding -Name "lab-api" -Protocol https -HostHeader "api.lab.example.com").
  AddSslCertificate($cert.Thumbprint, "my")

Invoke-WebRequest https://api.lab.example.com/health -UseBasicParsing
```

**ARR به‌عنوان reverse proxy برای Node.js:**

```powershell
# ۱) نصب ARR و URL Rewrite (دستی)
# ۲) فعال‌سازی proxy
Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST" `
  -Filter "system.webServer/proxy" -Name "enabled" -Value $true

# ۳) سایت با rewrite
New-Website -Name "lab-proxy" -PhysicalPath C:\inetpub\empty `
  -ApplicationPool "DefaultAppPool" -Port 8080 -HostHeader "app.lab.example.com"
```

```xml
<!-- C:\inetpub\empty\web.config -->
<configuration>
  <system.webServer>
    <rewrite>
      <rules>
        <rule name="Proxy" stopProcessing="true">
          <match url="(.*)" />
          <action type="Rewrite" url="http://127.0.0.1:3000/{R:1}" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
```

```powershell
# ۴) تست
Invoke-WebRequest http://app.lab.example.com:8080/ -UseBasicParsing
```

**دیباگ `500.30`:**

```powershell
# لاگ stdout
Get-Content C:\inetpub\lab-api\logs\stdout*.log -Tail 50

# اجرای مستقیم
cd C:\inetpub\lab-api
dotnet .\Lab.Api.dll
# خطا را می‌بینی
```

**دیباگ `502.5`:**

```powershell
# ANCM نصب است؟
Get-WebGlobalModule | Where-Object Name -like "AspNetCore*"

# dotnet در PATH سرویس؟
& "C:\Program Files\dotnet\dotnet.exe" --info
```

### مقایسه و الگوهای مختلف

**IIS + ASP.NET Core در برابر Nginx + Kestrel**

| | IIS + ANCM | Nginx + Kestrel |
|---|---|---|
| سیستم‌عامل | ویندوز | لینوکس |
| کارایی | ✅ | ✅ |
| مدیریت | ✅ GUI | ⚠️ |
| TLS | ✅ | ✅ |
| هزینه | 💰 | ۰ |

**`inprocess` در برابر `outofprocess`**

| | `inprocess` | `outofprocess` |
|---|---|---|
| کارایی | ✅✅ | ⚠️ |
| `Kestrel` config | ⚠️ بی‌اثر | ✅ |
| دیباگ | ⚠️ | ✅ |
| توصیه | ✅ | موارد خاص |

**ARR در برابر Nginx به‌عنوان proxy**

| | ARR | Nginx |
|---|---|---|
| کارایی | ⚠️ | ✅ |
| config | XML | متن |
| load balancing | ✅ | ✅ |
| مناسب | ویندوز | ✅ عمومی |

**دیدگاه «IIS + Kestrel کند است»**: با `inprocess`، تفاوت ناچیز است. مشکل معمولاً از `outofprocess` یا config اشتباه می‌آید.

### اشکال شایع

**۱) `500.30`**
اپ بالا نمی‌آید. `stdout` log یا اجرای مستقیم `dotnet`.

**۲) `500.31`**
`ANCM` نسخه‌ی `dotnet` را پیدا نمی‌کند. Hosting Bundle نصب/به‌روز نیست.

**۳) `502.5`**
پروسه‌ی اپ کرش. Event Viewer + `stdout`.

**۴) `500.19` در `web.config` ASP.NET Core**
بخش `aspNetCore` قفل است. `appcmd unlock config`.

**۵) `404` برای همه‌ی مسیرها**
`handlers` تعریف نشده. `aspNetCore` handler.

**۶) HTTPS و redirect loop**
`ASPNETCORE_FORWARDEDHEADERS_ENABLED` تنظیم نشده و اپ `X-Forwarded-Proto` را نمی‌بیند.

**۷) `ARR` و WebSocket**
`webSocket enabled="true"` + `proxy` فعال. timeout.

**۸) `Rewrite` و query string**
`{R:1}` مسیر است، query جدا. `appendQueryString="true"`.

**۹) `serverVariables` بدون allowed**
`500.50`. `allowedServerVariables`.

**۱۰) `inprocess` و `Kestrel` config**
بی‌اثر. `outofprocess` یا تنظیم در IIS.

### نکات مهم (Tips & Tricks)

- **Hosting Bundle** قبل از هر چیز.
- **`inprocess`** برای تولید.
- **`stdoutLogEnabled`** فقط برای دیباگ.
- **`ASPNETCORE_FORWARDEDHEADERS_ENABLED`** پشت proxy.
- **`allowedServerVariables`** برای ARR.
- **`appcmd unlock config`** برای بخش قفل.
- **`dotnet .\App.dll`** برای دیباگ مستقیم.

### خلاصه فصل

- `ANCM` پل IIS و ASP.NET Core است.
- `inprocess` سریع‌تر، `outofprocess` انعطاف‌پذیرتر.
- URL Rewrite برای redirect و SPA.
- ARR برای reverse proxy و load balancing.
- `500.30`/`502.5` با `stdout` و اجرای مستقیم دیباگ می‌شوند.
- `X-Forwarded-*` برای اپ پشت proxy.

**سؤال مروری:** تفاوت `500.30` و `502.5` چیست و هر کدام را چطور دیباگ می‌کنی؟

---

## منابع و مراجع

**مستندات رسمی**
- [IIS Documentation](https://learn.microsoft.com/en-us/iis/)
- [IIS Administration with appcmd](https://learn.microsoft.com/en-us/iis/get-started/getting-started-with-iis/getting-started-with-appcmdexe)
- [ASP.NET Core Module (ANCM)](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/aspnet-core-module)
- [URL Rewrite Module](https://learn.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-url-rewrite-module-20)
- [Application Request Routing (ARR)](https://learn.microsoft.com/en-us/iis/extensions/planning-for-arr/application-request-routing-version-3)
- [web.config reference](https://learn.microsoft.com/en-us/iis/configuration/system.webserver/)

**ابزار**
- [IIS Manager](https://learn.microsoft.com/en-us/iis/get-started/getting-started-with-iis/getting-started-with-the-iis-manager)
- [Failed Request Tracing](https://learn.microsoft.com/en-us/iis/troubleshoot/using-failed-request-tracing/troubleshooting-failed-requests-using-tracing-in-iis)
- [Jexus Manager](https://github.com/jexuswebserver/JexusManager) — مدیریت IIS روی ویندوز کلاینت

---

**بعدی:** [`07-Static-and-Dynamic-Content.md`](07-Static-and-Dynamic-Content.md) — MIME، PHP-FPM، Python/Node و خطاهای ۴۰۳/۵۰۰
