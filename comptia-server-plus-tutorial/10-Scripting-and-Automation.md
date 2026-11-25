# اسکریپت‌نویسی و خودکارسازی

**مدت یادگیری**: ۶ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فصل ۲۳ تا ۳۷ — کار با ویندوز سرور و لینوکس
**دامنهٔ آزمون**: Server Administration (۳۰٪) — بخش Scripting basics

---

## فهرست محتوا

- [فصل ۴۷: چرا مدیر سرور باید اسکریپت بنویسد](#ch-47)
- [فصل ۴۸: PowerShell برای مدیریت ویندوز](#ch-48)
- [فصل ۴۹: Bash برای مدیریت لینوکس](#ch-49)
- [فصل ۵۰: مدیریت از راه دور و خودکارسازی](#ch-50)

---

## فصل ۴۷: چرا مدیر سرور باید اسکریپت بنویسد

### مقدمه

در آزمون Server+، بخش «Scripting basics» فقط «متغیر، حلقه، شرط» می‌خواهد — نه برنامه‌نویسی حرفه‌ای. اما دلیل وجود این بخش مهم‌تر از محتوای آن است: **کار تکراری، منبع خطاست.**

این فصل دربارهٔ منطق خودکارسازی است، نه سینتکس. سینتکس در دو فصل بعد می‌آید.

### مفاهیم اساسی

**سه سطح خودکارسازی:**

| سطح | توضیح | مثال |
|---|---|---|
| **دستی** | هر بار از صفر | SSH بزن، دستور بزن |
| **اسکریپت** | یک فایل، چند دستور | `backup.sh` |
| **مدیریت پیکربندی** | وضعیت نهایی را تعریف می‌کنی | Ansible، Puppet |

**تفاوت اسکریپت و Configuration Management:**

| معیار | اسکریپت | Config Mgmt |
|---|---|---|
| رویکرد | «چه کارهایی بکن» (imperative) | «چه وضعیتی باشد» (declarative) |
| تکرارپذیری | متوسط | بالا |
| idempotent | معمولاً نه | بله |
| مقیاس | ۱–۱۰ سرور | ۱۰+ سرور |
| مثال | Bash، PowerShell | Ansible، Puppet، Chef |

**Idempotent یعنی چه؟** یعنی اجرای دوباره، همان نتیجه را بدهد. `mkdir /data` اگر پوشه باشد خطا می‌دهد (غیر idempotent)؛ `mkdir -p /data` خطا نمی‌دهد (idempotent).

**این مفهوم، قلب خودکارسازی است:** اسکریپتی که دوبار اجرا شود و سیستم را خراب کند، خودکارسازی نیست؛ یک بمب ساعتی است.

**چهار دلیل برای اسکریپت نوشتن:**

۱. **کاهش خطای انسانی** — انسان در ساعت ۳ بامداد اشتباه می‌کند؛ اسکریپت نه (اگر درست نوشته شده باشد).
۲. **سرعت** — ۵۰ سرور در ۲ دقیقه، نه ۲ روز.
۳. **مستندسازی زنده** — اسکریپت، خودش مستند است (اگر خوانا باشد).
۴. **تکرارپذیری** — همان نتیجه، هر بار.

**سه دلیل برای اسکریپت ننوشتن (یا با احتیاط نوشتن):**

۱. **اسکریپت بد، بدتر از دستی است** — یک `rm -rf` اشتباه، ۵۰ سرور را پاک می‌کند.
۲. **هزینهٔ نگهداشت** — اسکریپت باید نگهداری شود، وگرنه می‌پوسد.
۳. **پیچیدگی پنهان** — اسکریپتی که هیچ‌کس نمی‌فهمد، یک SPOF است.

**قواعد اسکریپت خوب:**

| قاعده | توضیح |
|---|---|
| **خوانا** | نام متغیر معنادار، کامنت |
| **امن** | بررسی ورودی، quoted variables |
| **Idempotent** | اجرای دوباره بی‌خطر |
| **Fail-safe** | در خطا متوقف شود، نه ادامه |
| **Loggable** | چه کاری انجام داد |
| **Reversible** | اگر ممکن، rollback |

**نکتهٔ حیاتی:** اسکریپت باید **قبل از اجرا** قابل خواندن باشد. اگر نمی‌توانی در ۳۰ ثانیه بفهمی چه می‌کند، برای تولید مناسب نیست.

### مثال‌های عملی

**Lab Scenario 1 — تصمیم خودکارسازی**

> **وظیفه:** برای هر کار، بگو خودکار کن یا نه.
>
> الف) ساخت کاربر برای کارمند جدید
> ب) تغییر تنظیمات فایروال در بحران
> ج) بررسی فضای دیسک ۵۰ سرور
> د) بازیابی دیتابیس از backup
>
> **پاسخ:**
>
> | کار | تصمیم | دلیل |
> |---|---|---|
> | الف | ✅ خودکار | تکراری، استاندارد، قابل تست |
> | ب | ❌ دستی | بحرانی، نیاز به قضاوت، ریسک بالا |
> | ج | ✅ خودکار | تکراری، بی‌خطر، پایش |
> | د | ⚠️ نیمه‌خودکار | اسکریپت آماده، اجرای تأییدشده |
>
> **درس:** خودکارسازی برای کارهای **تکراری و استاندارد**؛ نه برای تصمیم‌های بحرانی.

**Lab Scenario 2 — اسکریپت خطرناک**

> **وضعیت:** اسکریپت زیر را در تولید پیدا می‌کنی:
>
> ```bash
> cd /var/log
> rm -rf *
> ```
>
> **وظیفه:** سه مشکل را نام ببر.
>
> **پاسخ:**
> ۱. **بدون بررسی متغیر:** اگر `cd` شکست بخورد (مثلاً پوشه نباشد)، `rm -rf *` در پوشهٔ فعلی (که ممکن است `/` باشد) اجرا می‌شود.
> ۲. **بدون `set -e`:** خطا متوقف نمی‌کند.
> ۳. **بدون فیلتر:** همه چیز را پاک می‌کند، از جمله لاگ‌هایی که ممکن است لازم باشند.
>
> **نسخهٔ بهتر:**
> ```bash
> #!/usr/bin/env bash
> set -euo pipefail
> LOG_DIR=/var/log
> [ -d "$LOG_DIR" ] || { echo "not found"; exit 1; }
> find "$LOG_DIR" -type f -name "*.log" -mtime +30 -delete
> ```
>
> **درس:** `cd` بدون بررسی + `rm -rf *` = کلاسیک‌ترین فاجعهٔ اسکریپت.

### مقایسه و الگوهای مختلف

**مقایسهٔ زبان‌های اسکریپت در سرور:**

| زبان | پلتفرم | مناسب |
|---|---|---|
| **Bash** | لینوکس | عملیات سیستم، فایل |
| **PowerShell** | ویندوز/چندسکویی | مدیریت ویندوز، شیءمحور |
| **Python** | هر دو | منطق پیچیده، API |
| **Perl** | لینوکس | متن (میراثی) |

**توصیه:** برای عملیات سیستم، Bash/PowerShell. برای منطق پیچیده یا API، Python.

**مقایسهٔ imperative و declarative:**

| معیار | Imperative (Bash) | Declarative (Ansible) |
|---|---|---|
| خوانایی | متوسط | بالا |
| idempotent | دستی | خودکار |
| کنترل | کامل | محدود به ماژول |
| مناسب | کار خاص | پیکربندی استاندارد |

**مقایسهٔ Cron و systemd timer:**

| معیار | Cron | systemd timer |
|---|---|---|
| سادگی | بالا | متوسط |
| لاگ | ایمیل/فایل | journald |
| وابستگی | ❌ | ✅ |
| دقت | دقیقه | ثانیه |
| توصیه | ساده | مدرن |

### اشکال شایع

**۱. نبود `set -euo pipefail` در Bash.**
خطا نادیده گرفته می‌شود و اسکریپت به کار خطرناک ادامه می‌دهد.

**۲. متغیر بدون quote.**
`rm -rf $DIR` اگر `DIR` خالی باشد → `rm -rf` (بدون آرگومان، خطا) اما `rm -rf $DIR/*` با `DIR` خالی → `rm -rf /*`.

**۳. اسکریپت بدون تست.**
در تولید تست می‌شود — با دادهٔ واقعی.

**۴. رمز hardcoded.**
در Git می‌رود و برای همیشه می‌ماند.

**۵. اسکریپت بدون لاگ.**
نمی‌دانی چه شد.

**۶. اسکریپت بدون idempotency.**
اجرای دوم، خرابی می‌سازد.

**۷. اسکریپت خیلی بزرگ.**
۵۰۰ خط Bash بدون تابع = غیرقابل نگهداشت.

**۸. نبود rollback.**
تغییر اعمال شد و راه برگشت نیست.

### نکات مهم (Tips & Tricks)

- **همیشه `#!/usr/bin/env bash`** نه `#!/bin/bash` (سازگاری بیشتر).
- **`set -euo pipefail`** در خط اول (بعد از shebang).
- **`shellcheck`** برای بررسی اسکریپت Bash — رایگان و عالی.
- **`PSScriptAnalyzer`** برای PowerShell.
- **در تست، `echo` قبل از دستور خطرناک بگذار** (dry run).
- **در آزمون:** کلیدواژهٔ «idempotent» → **safe to run multiple times**.

### خلاصه فصل

- سه سطح: **دستی، اسکریپت، Config Management**.
- **Idempotent** = اجرای دوباره بی‌خطر.
- خودکارسازی برای کار **تکراری و استاندارد**، نه تصمیم بحرانی.
- قواعد: خوانا، امن، fail-safe، loggable، reversible.
- `set -euo pipefail` و `shellcheck` حداقل‌ها هستند.
- اسکریپت بد، بدتر از دستی است.

---

## فصل ۴۸: PowerShell برای مدیریت ویندوز

### مقدمه

PowerShell یک پوستهٔ متنی نیست؛ یک **زبان شیءمحور** است. تفاوت بنیادی با Bash: در Bash با **متن** کار می‌کنی، در PowerShell با **شیء**.

این تفاوت، همه چیز را تغییر می‌دهد: خروجی `Get-Service` یک شیء است با propertyهای `Name`، `Status`، `DisplayName` — نه یک خط متن.

### مفاهیم اساسی

**ساختار دستور (Verb-Noun):**

```powershell
Get-Service        # Get = فعل، Service = اسم
Set-Service
New-Item
Remove-Item
Restart-Service
```

**افعال رایج:** Get، Set، New، Remove، Start، Stop، Restart، Test، Import، Export، Invoke.

**مفاهیم پایه:**

| مفهوم | توضیح | مثال |
|---|---|---|
| **Cmdlet** | دستور داخلی | `Get-Process` |
| **Pipeline** | انتقال شیء | `Get-Service \| Where-Object ...` |
| **Variable** | `$name` | `$servers = @("srv1","srv2")` |
| **Array** | `@()` | `$list = @(1,2,3)` |
| **Hashtable** | `@{}` | `@{Name="srv1"; IP="10.0.0.1"}` |
| **Object** | `[PSCustomObject]` | ساختار سفارشی |

**Pipeline — قلب PowerShell:**

```powershell
# همهٔ سرویس‌های متوقف
Get-Service | Where-Object { $_.Status -eq 'Stopped' }

# مرتب‌سازی و انتخاب
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5 Name, CPU

# تبدیل به جدول
Get-Service | Format-Table Name, Status -AutoSize

# خروجی JSON
Get-Service | Select-Object Name, Status | ConvertTo-Json
```

**متغیر و نوع:**

```powershell
$name = "srv01"
$count = 10
$list = @("srv01","srv02","srv03")
$map = @{ "srv01" = "10.0.0.1"; "srv02" = "10.0.0.2" }

# دسترسی
$list[0]         # srv01
$map["srv01"]    # 10.0.0.1
$list.Count      # 3
```

**شرط:**

```powershell
if ($count -gt 5) {
    Write-Output "بیشتر از ۵"
} elseif ($count -eq 5) {
    Write-Output "مساوی ۵"
} else {
    Write-Output "کمتر از ۵"
}
```

**عملگرهای مقایسه:**

| عملگر | معنی |
|---|---|
| `-eq` / `-ne` | مساوی / نامساوی |
| `-gt` / `-lt` | بزرگ‌تر / کوچک‌تر |
| `-ge` / `-le` | بزرگ‌تر یا مساوی / کوچک‌تر یا مساوی |
| `-like` / `-notlike` | الگو (`*`) |
| `-match` / `-notmatch` | regex |
| `-contains` / `-in` | عضویت |
| `-and` / `-or` / `-not` | منطقی |

**حلقه‌ها:**

```powershell
# for
for ($i = 1; $i -le 5; $i++) {
    Write-Output "شماره $i"
}

# foreach
$servers = @("srv01","srv02","srv03")
foreach ($s in $servers) {
    Write-Output "بررسی $s"
}

# while
$i = 0
while ($i -lt 3) {
    Write-Output $i
    $i++
}

# ForEach-Object (در pipeline)
$servers | ForEach-Object { Write-Output "بررسی $_" }
```

**توابع:**

```powershell
function Get-DiskReport {
    param(
        [Parameter(Mandatory=$true)]
        [string]$ComputerName,

        [int]$ThresholdGB = 20
    )

    $disks = Get-CimInstance -ClassName Win32_LogicalDisk `
        -ComputerName $ComputerName `
        -Filter "DriveType=3"

    foreach ($d in $disks) {
        $freeGB = [math]::Round($d.FreeSpace / 1GB, 2)
        [PSCustomObject]@{
            Computer = $ComputerName
            Drive    = $d.DeviceID
            FreeGB   = $freeGB
            Low      = ($freeGB -lt $ThresholdGB)
        }
    }
}
```

**خطا و try/catch:**

```powershell
try {
    Get-Service -Name "NoSuchService" -ErrorAction Stop
}
catch {
    Write-Error "خطا: $($_.Exception.Message)"
}
finally {
    Write-Output "پایان"
}
```

**نکتهٔ مهم:** `-ErrorAction Stop` لازم است، وگرنه خطا catch نمی‌شود.

**خروجی و ورودی:**

```powershell
# CSV
Get-Service | Export-Csv -Path "C:\temp\services.csv" -NoTypeInformation
$data = Import-Csv -Path "C:\temp\services.csv"

# JSON
Get-Process | ConvertTo-Json -Depth 3 | Out-File "proc.json"

# لاگ
Start-Transcript -Path "C:\temp\script.log"
# ... دستورها ...
Stop-Transcript
```

**اسکریپت کامل — گزارش فضای دیسک:**

```powershell
<#
.SYNOPSIS
    گزارش فضای دیسک سرورها و ارسال هشدار
.DESCRIPTION
    برای هر سرور، فضای آزاد را چک می‌کند و اگر کمتر از آستانه بود، هشدار می‌دهد.
.EXAMPLE
    .\DiskReport.ps1 -Servers "srv01","srv02" -ThresholdGB 20
#>

[CmdletBinding()]
param(
    [string[]]$Servers = @($env:COMPUTERNAME),
    [int]$ThresholdGB = 20
)

$results = @()

foreach ($srv in $Servers) {
    try {
        $disks = Get-CimInstance -ClassName Win32_LogicalDisk `
            -ComputerName $srv -ErrorAction Stop |
            Where-Object { $_.DriveType -eq 3 }

        foreach ($d in $disks) {
            $freeGB = [math]::Round($d.FreeSpace / 1GB, 2)
            $totalGB = [math]::Round($d.Size / 1GB, 2)
            $pct = if ($totalGB -gt 0) { [math]::Round(($freeGB / $totalGB) * 100, 1) } else { 0 }

            $results += [PSCustomObject]@{
                Server  = $srv
                Drive   = $d.DeviceID
                TotalGB = $totalGB
                FreeGB  = $freeGB
                FreePct = $pct
                Status  = if ($freeGB -lt $ThresholdGB) { "WARNING" } else { "OK" }
            }
        }
    }
    catch {
        $results += [PSCustomObject]@{
            Server  = $srv
            Drive   = "N/A"
            TotalGB = 0
            FreeGB  = 0
            FreePct = 0
            Status  = "UNREACHABLE"
        }
    }
}

$results | Format-Table -AutoSize

$warn = $results | Where-Object { $_.Status -eq "WARNING" }
if ($warn.Count -gt 0) {
    Write-Warning "$($warn.Count) درایو زیر آستانه است"
    $warn | Export-Csv -Path "$env:TEMP\disk_warnings.csv" -NoTypeInformation
}

exit $(if ($warn.Count -gt 0) { 1 } else { 0 })
```

**نکته:** `exit code` برای استفاده در scheduled task و monitoring مهم است.

### مثال‌های عملی

**Lab Scenario 1 — پیدا کردن سرویس‌های متوقف**

> **وظیفه:** یک خط PowerShell بنویس که سرویس‌های Automatic را که متوقف‌اند نشان دهد.
>
> **پاسخ:**
>
> ```powershell
> Get-Service |
>     Where-Object { $_.StartType -eq 'Automatic' -and $_.Status -eq 'Stopped' } |
>     Select-Object Name, DisplayName, StartType |
>     Format-Table -AutoSize
> ```
>
> **درس:** PowerShell با pipeline، جایگزین حلقه‌های طولانی است.

**Lab Scenario 2 — اسکریپت برای ۱۰۰ سرور**

> **وظیفه:** یک سرویس را روی ۱۰۰ سرور ری‌استارت کن.
>
> **پاسخ:**
>
> ```powershell
> $servers = Get-Content "C:\temp\servers.txt"
> $results = foreach ($s in $servers) {
>     try {
>         Invoke-Command -ComputerName $s -ScriptBlock {
>             Restart-Service -Name "Spooler" -ErrorAction Stop
>             (Get-Service Spooler).Status
>         } -ErrorAction Stop
>     }
>     catch {
>         "ERROR: $($_.Exception.Message)"
>     }
> }
> $results | Out-File "C:\temp\restart_log.txt"
> ```
>
> **درس:** `Invoke-Command` + `try/catch` = مدیریت از راه دور مقاوم.

### مقایسه و الگوهای مختلف

**مقایسهٔ PowerShell و Bash:**

| معیار | PowerShell | Bash |
|---|---|---|
| نوع داده | شیء | متن |
| Pipeline | شیء | stream متن |
| پلتفرم | ویندوز/لینوکس | لینوکس |
| یادگیری | متوسط | ساده شروع، سخت تسلط |
| مناسب | ویندوز، ساخت‌یافته | عملیات سیستم |

**مقایسهٔ CIM و WMI:**

| معیار | WMI (`Get-WmiObject`) | CIM (`Get-CimInstance`) |
|---|---|---|
| پروتکل | DCOM | WS-Man |
| امنیت | ضعیف‌تر | بهتر |
| وضعیت | منسوخ | ✅ توصیه‌شده |
| راه دور | نیاز به تنظیم | آسان‌تر |

**نکته:** `Get-WmiObject` منسوخ شده؛ `Get-CimInstance` استفاده کن.

**مقایسهٔ `-ErrorAction`:**

| مقدار | رفتار |
|---|---|
| `Continue` | خطا نشان، ادامه (پیش‌فرض) |
| `Stop` | خطا، توقف (برای try/catch) |
| `SilentlyContinue` | نادیده |
| `Inquire` | پرسش |

### اشکال شایع

**۱. استفاده از `Get-WmiObject`.**
منسوخ است. `Get-CimInstance`.

**۲. نبود `-ErrorAction Stop` در try/catch.**
catch اجرا نمی‌شود.

**۳. مقایسه با `==`.**
در PowerShell `-eq` است. `==` خطا می‌دهد.

**۴. رشته بدون quote در مسیر.**
مسیر با فاصله، شکسته می‌شود.

**۵. `Write-Host` به‌جای `Write-Output`.**
`Write-Host` به pipeline نمی‌رود. برای خروجی داده، `Write-Output`.

**۶. اجرای اسکریپت بدون `ExecutionPolicy`.**
`Set-ExecutionPolicy RemoteSigned` (یا `-Scope Process`).

**۷. بی‌توجهی به `$Error`.**
بعد از اسکریپت، `$Error[0]` را چک کن.

**۸. حلقه به‌جای pipeline.**
کندتر و طولانی‌تر.

### نکات مهم (Tips & Tricks)

- **`Get-Help <cmdlet> -Examples`** — بهترین راه یادگیری.
- **`Get-Command -Verb Get`** — همهٔ دستورهای Get.
- **`Get-Member`** — چه propertyهایی دارد.
- **`-WhatIf`** — نمایش بدون اجرا (برای دستورهای تغییردهنده).
- **`-Confirm`** — تأیید قبل از اجرا.
- **`PSScriptAnalyzer`** — بررسی کیفیت.
- **در آزمون:** کلیدواژهٔ «Verb-Noun» → **PowerShell**؛ «object pipeline» → **PowerShell**.

### خلاصه فصل

- PowerShell = **شیءمحور**، Verb-Noun، pipeline.
- متغیر `$x`، آرایه `@()`، هشت‌جدولی `@{}`.
- مقایسه با `-eq`، `-gt`، `-like`، `-match`.
- حلقه: `for`، `foreach`، `while`، `ForEach-Object`.
- `try/catch` با `-ErrorAction Stop`.
- `Get-CimInstance` جای `Get-WmiObject`.

---

## فصل ۴۹: Bash برای مدیریت لینوکس

### مقدمه

Bash زبان مادری لینوکس است. در آزمون Server+، Bash در سطح «متغیر، حلقه، شرط» لازم است — اما در کار واقعی، هر روز با آن سر و کار داری.

این فصل، Bash کاربردی برای مدیر سرور را می‌سازد: از `set -euo pipefail` تا تابع و trap.

### مفاهیم اساسی

**ساختار اسکریپت:**

```bash
#!/usr/bin/env bash
set -euo pipefail

# بدنهٔ اسکریپت
echo "شروع"
```

**`set -euo pipefail` — سه محافظ:**

| گزینه | اثر |
|---|---|
| `-e` | توقف در اولین خطا |
| `-u` | خطا در استفاده از متغیر تعریف‌نشده |
| `-o pipefail` | خطا در pipeline هم گرفته می‌شود |

**نکتهٔ حیاتی:** بدون `-o pipefail`، `cmd1 | cmd2` فقط کد خروج `cmd2` را برمی‌گرداند. اگر `cmd1` شکست بخورد و `cmd2` موفق شود، خطا دیده نمی‌شود.

**متغیر:**

```bash
NAME="srv01"
COUNT=10
readonly CONFIG="/etc/myapp/config.yml"

echo "$NAME"          # استفاده با $
echo "${NAME}_backup" # آکولاد برای جداسازی
```

**نکتهٔ طلایی:** همیشه `"$VAR"` با quote. بدون quote، فاصله و کاراکتر خاص مشکل می‌سازد.

**متغیرهای ویژه:**

| متغیر | معنی |
|---|---|
| `$0` | نام اسکریپت |
| `$1`..`$9` | آرگومان‌ها |
| `$#` | تعداد آرگومان |
| `$@` | همهٔ آرگومان‌ها |
| `$?` | کد خروج آخرین دستور |
| `$$` | PID اسکریپت |
| `$!` | PID آخرین process پس‌زمینه |

**شرط:**

```bash
if [ "$COUNT" -gt 5 ]; then
    echo "بیشتر از ۵"
elif [ "$COUNT" -eq 5 ]; then
    echo "مساوی ۵"
else
    echo "کمتر از ۵"
fi
```

**عملگرهای تست:**

| عملگر | معنی |
|---|---|
| `-eq` / `-ne` | عددی مساوی/نامساوی |
| `-gt` / `-lt` / `-ge` / `-le` | عددی |
| `-z` / `-n` | رشتهٔ خالی/غیرخالی |
| `=` / `!=` | رشته |
| `-f` / `-d` / `-e` | فایل/دایرکتوری/وجود |
| `-r` / `-w` / `-x` | خواندنی/نوشتنی/اجرایی |

**نکته:** `[[ ]]` (bash) بهتر از `[ ]` (POSIX) است: پشتیبانی از regex و بدون مشکل word splitting.

```bash
if [[ "$FILE" == *.log ]]; then
    echo "فایل لاگ"
fi

if [[ "$IP" =~ ^10\.[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
    echo "IP خصوصی"
fi
```

**حلقه:**

```bash
# for روی لیست
for srv in srv01 srv02 srv03; do
    echo "بررسی $srv"
done

# for روی فایل
while IFS= read -r line; do
    echo "$line"
done < servers.txt

# for عددی (C-style)
for ((i=1; i<=5; i++)); do
    echo "$i"
done

# while
i=0
while [[ $i -lt 3 ]]; do
    echo "$i"
    ((i++))
done
```

> ⚠️ **تلهٔ کلاسیک:** `((i++))` وقتی `i=0` باشد، مقدار صفر برمی‌گرداند و در `set -e` اسکریپت را می‌کشد. راه‌حل: `i=$((i+1))` یا `((++i))`.

**تابع:**

```bash
log() {
    local level="$1"
    shift
    echo "[$(date -Iseconds)] [$level] $*" >&2
}

check_disk() {
    local path="${1:-/}"
    local threshold="${2:-80}"
    local usage
    usage=$(df -h "$path" | awk 'NR==2 {gsub("%","",$5); print $5}')

    if [[ "$usage" -ge "$threshold" ]]; then
        log WARN "دیسک $path با $usage% پر است"
        return 1
    fi
    return 0
}
```

**نکته:** `local` در تابع، متغیر را محلی می‌کند. بدون آن، متغیر سراسری است.

**Trap — پاک‌سازی در خروج:**

```bash
TMP_DIR=$(mktemp -d)
cleanup() {
    rm -rf "$TMP_DIR"
}
trap cleanup EXIT

# ... کار با TMP_DIR ...
```

**نکتهٔ حیاتی:** `trap ... EXIT` حتی در خطا هم اجرا می‌شود. برای پاک‌سازی فایل موقت ضروری است.

**اسکریپت کامل — بررسی سلامت سرور:**

```bash
#!/usr/bin/env bash
set -euo pipefail

LOG_FILE="/var/log/healthcheck.log"
THRESHOLD_DISK=85
THRESHOLD_MEM=90

log() {
    echo "[$(date -Iseconds)] $*" | tee -a "$LOG_FILE"
}

fail=0

# ۱. فضای دیسک
while read -r usage mount; do
    usage=${usage%\%}
    if (( usage >= THRESHOLD_DISK )); then
        log "WARN: دیسک $mount با $usage% پر است"
        fail=1
    fi
done < <(df -P | awk 'NR>1 {print $5, $6}')

# ۲. حافظه
mem_used=$(free | awk '/^Mem:/ {printf "%d", $3/$2*100}')
if (( mem_used >= THRESHOLD_MEM )); then
    log "WARN: حافظه با $mem_used% مصرف"
    fail=1
fi

# ۳. سرویس‌های حیاتی
for svc in sshd cron; do
    if ! systemctl is-active --quiet "$svc"; then
        log "CRIT: سرویس $svc فعال نیست"
        fail=1
    fi
done

# ۴. load average
load1=$(awk '{print int($1)}' /proc/loadavg)
cores=$(nproc)
if (( load1 > cores * 2 )); then
    log "WARN: load $load1 روی $cores هسته"
    fail=1
fi

if (( fail == 0 )); then
    log "OK: همه چیز سالم است"
else
    log "FAIL: مشکلات بالا را بررسی کن"
fi

exit "$fail"
```

### مثال‌های عملی

**Lab Scenario 1 — پشتیبان‌گیری با rotation**

> **وظیفه:** اسکریپتی بنویس که یک پوشه را روزانه backup کند و ۷ نسخهٔ آخر را نگه دارد.
>
> **پاسخ:**
>
> ```bash
> #!/usr/bin/env bash
> set -euo pipefail
>
> SRC="/srv/data"
> DEST="/backup/daily"
> KEEP=7
> STAMP=$(date +%Y%m%d-%H%M%S)
>
> [[ -d "$SRC" ]] || { echo "منبع نیست: $SRC" >&2; exit 1; }
> mkdir -p "$DEST"
>
> tar -czf "$DEST/data-$STAMP.tar.gz" -C "$(dirname "$SRC")" "$(basename "$SRC")"
>
> # نگه داشتن ۷ نسخهٔ آخر
> ls -1t "$DEST"/data-*.tar.gz | tail -n +$((KEEP+1)) | xargs -r rm -f
>
> echo "backup ok: $DEST/data-$STAMP.tar.gz"
> ```
>
> **درس:** `tail -n +$((KEEP+1))` نسخه‌های قدیمی را انتخاب می‌کند — الگوی استاندارد rotation.

**Lab Scenario 2 — اسکریپت روی ۵۰ سرور**

> **وظیفه:** یک اسکریپت را روی ۵۰ سرور اجرا کن.
>
> **پاسخ:**
>
> ```bash
> #!/usr/bin/env bash
> set -euo pipefail
>
> while IFS= read -r host; do
>     [[ -z "$host" ]] && continue
>     echo "=== $host ==="
>     ssh -o ConnectTimeout=5 -o BatchMode=yes "$host" 'bash -s' < ./remote_check.sh \
>         || echo "FAILED: $host"
> done < servers.txt
> ```
>
> **درس:** `ssh host 'bash -s' < script.sh` اسکریپت را بدون کپی اجرا می‌کند. `BatchMode=yes` از توقف برای رمز جلوگیری می‌کند.

### مقایسه و الگوهای مختلف

**مقایسهٔ `[ ]` و `[[ ]]`:**

| معیار | `[ ]` | `[[ ]]` |
|---|---|---|
| POSIX | ✅ | ❌ (bash) |
| Regex | ❌ | ✅ |
| word splitting | مشکل‌ساز | امن |
| `&&` / `\|\|` داخلی | ❌ | ✅ |
| توصیه | اسکریپت portable | Bash |

**مقایسهٔ `for` و `while read`:**

| معیار | `for` روی فایل | `while read` |
|---|---|---|
| فاصله در خط | می‌شکند | حفظ می‌شود |
| خط خالی | رد می‌شود | پردازش |
| توصیه | لیست ساده | فایل |

**مقایسهٔ Cron و systemd timer (تکمیلی):**

| معیار | Cron | Timer |
|---|---|---|
| تعریف | `crontab -e` | `.timer` + `.service` |
| لاگ | `/var/log/syslog` | `journalctl -u` |
| missed run | اجرا نمی‌شود | `Persistent=true` |
| مناسب | ساده | تولیدی |

### اشکال شایع

**۱. نبود `set -euo pipefail`.**
خطا نادیده گرفته می‌شود.

**۲. `rm -rf $DIR/*` با `DIR` خالی.**
`rm -rf /*`. همیشه `"${DIR:?}"` یا بررسی.

**۳. `((i++))` با `i=0` زیر `set -e`.**
اسکریپت می‌میرد. راه‌حل: `i=$((i+1))`.

**۴. `local x=$(cmd)`**
کد خروج `cmd` بلعیده می‌شود. راه‌حل:
```bash
local x
x=$(cmd)
```

**۵. `cd` بدون بررسی.**
اگر شکست بخورد، دستورهای بعدی در پوشهٔ اشتباه اجرا می‌شوند.

**۶. مقایسهٔ رشته با `-eq`.**
`[ "$a" -eq "$b" ]` روی رشته، خطا یا نتیجهٔ غلط.

**۷. پارس کردن `ls`.**
از `find` یا glob استفاده کن.

**۸. اسکریپت بدون `trap`.**
فایل موقت باقی می‌ماند.

### نکات مهم (Tips & Tricks)

- **`shellcheck script.sh`** — قبل از هر اجرا.
- **`bash -n script.sh`** — بررسی سینتکس بدون اجرا.
- **`bash -x script.sh`** — trace اجرا (برای دیباگ).
- **`set -x` موقت** در بخش مشکل‌دار.
- **`"${VAR:?message}"`** — خطا اگر خالی باشد.
- **در آزمون:** کلیدواژهٔ «script exits on first error» → **`set -e`**.

### خلاصه فصل

- `set -euo pipefail` = سه محافظ.
- متغیر با `"$VAR"` quote کن.
- `[[ ]]` بهتر از `[ ]` در Bash.
- `while IFS= read -r line` برای خواندن فایل.
- `trap ... EXIT` برای پاک‌سازی.
- `shellcheck` قبل از اجرا.

---

## فصل ۵۰: مدیریت از راه دور و خودکارسازی

### مقدمه

تا اینجا اسکریپت‌ها را روی یک سرور نوشتیم. اما مدیر سرور، ده‌ها یا صدها سرور دارد. مدیریت از راه دور، تفاوت بین «یک سرور» و «یک زیرساخت» است.

این فصل دربارهٔ پروتکل‌ها، ابزارها و الگوهای خودکارسازی است.

### مفاهیم اساسی

**پروتکل‌های مدیریت از راه دور:**

| پروتکل | پورت | پلتفرم | توضیح |
|---|---|---|---|
| **SSH** | 22 | لینوکس/چندسکویی | پوستهٔ امن |
| **RDP** | 3389 | ویندوز | دسکتاپ گرافیکی |
| **WinRM** | 5985/5986 | ویندوز | PowerShell Remoting |
| **WMI/CIM** | 135 + dynamic | ویندوز | مدیریت شیء |
| **SNMP** | 161 | تجهیزات | پایش |
| **IPMI/Redfish** | 623/443 | BMC | سخت‌افزار |

**نکتهٔ امنیتی:** RDP و WinRM هرگز مستقیم روی اینترنت نباشند. VPN، bastion host، یا jump server.

**SSH — مبانی:**

```bash
# اتصال
ssh user@10.0.10.21

# با کلید
ssh -i ~/.ssh/id_ed25519 user@10.0.10.21

# اجرای دستور
ssh user@host 'uptime; df -h'

# تونل
ssh -L 8080:localhost:80 user@host      # local forward
ssh -R 9000:localhost:9000 user@host    # remote forward

# کپی فایل
scp file.txt user@host:/tmp/
rsync -avz --progress /data/ user@host:/backup/data/
```

**SSH key — راه‌اندازی:**

```bash
# ساخت کلید (Ed25519 توصیه‌شده)
ssh-keygen -t ed25519 -C "admin@corp" -f ~/.ssh/id_ed25519

# کپی به سرور
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@host

# تست
ssh -i ~/.ssh/id_ed25519 user@host 'echo ok'
```

**SSH hardening (`/etc/ssh/sshd_config`):**

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
ClientAliveInterval 300
AllowUsers admin deploy
```

**نکتهٔ حیاتی:** قبل از restart، مطمئن شو کلید کار می‌کند — وگرنه از سرور قفل می‌شوی.

**PowerShell Remoting:**

```powershell
# فعال‌سازی (روی سرور مقصد)
Enable-PSRemoting -Force

# تست
Test-WSMan -ComputerName srv01

# اجرای دستور
Invoke-Command -ComputerName srv01 -ScriptBlock { Get-Service }

# session پایدار
$s = New-PSSession -ComputerName srv01
Invoke-Command -Session $s -ScriptBlock { hostname }
Enter-PSSession -Session $s
Remove-PSSession $s
```

**نکته:** `Invoke-Command` برای چند سرور موازی اجرا می‌کند (تا ۳۲ پیش‌فرض).

**Ansible — اصول:**

```yaml
# inventory.ini
[webservers]
web01 ansible_host=10.0.10.21
web02 ansible_host=10.0.10.22

[all:vars]
ansible_user=deploy
ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

```yaml
# playbook.yml
---
- name: پیکربندی وب سرورها
  hosts: webservers
  become: true
  tasks:
    - name: نصب nginx
      package:
        name: nginx
        state: present

    - name: کپی config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        owner: root
        mode: '0644'
      notify: restart nginx

    - name: اطمینان از فعال بودن
      service:
        name: nginx
        state: started
        enabled: true

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```

**اجرا:**

```bash
ansible-playbook -i inventory.ini playbook.yml --check   # dry run
ansible-playbook -i inventory.ini playbook.yml
```

**نکتهٔ مهم:** `--check` (dry run) قبل از اجرای واقعی. و `notify` + `handler` یعنی restart فقط وقتی config عوض شد.

**Idempotency در Ansible:** اگر playbook را دوبار اجرا کنی، بار دوم هیچ تغییری نمی‌دهد (`changed=0`).

**الگوهای خودکارسازی:**

| الگو | توضیح |
|---|---|
| **Push** | سرور مدیریت دستور می‌فرستد (Ansible) |
| **Pull** | سرور خودش config می‌گیرد (Puppet/Chef agent) |
| **Agentless** | بدون نصب agent (Ansible با SSH) |
| **Agent-based** | agent روی هر سرور |

**مقایسه:**

| معیار | Push/Agentless | Pull/Agent |
|---|---|---|
| نصب | فقط روی کنترلر | روی همه |
| مقیاس | متوسط–بالا | بسیار بالا |
| زمان‌بندی | لحظه‌ای | دوره‌ای |
| مثال | Ansible | Puppet، Chef |

**Scheduled task / Cron:**

```bash
# crontab -e
# دقیقه ساعت روز ماه روزهفته دستور
0 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
*/5 * * * * /usr/local/bin/check.sh
```

**systemd timer (مدرن):**

```ini
# /etc/systemd/system/backup.timer
[Unit]
Description=Daily backup

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
systemctl enable --now backup.timer
systemctl list-timers
```

**نکته:** `Persistent=true` یعنی اگر سیستم خاموش بود، در بوت بعدی اجرا می‌شود. Cron این را ندارد.

### مثال‌های عملی

**Lab Scenario 1 — اجرای موازی روی ۵۰ سرور**

> **وظیفه:** یک اسکریپت را روی ۵۰ سرور هم‌زمان اجرا کن.
>
> **پاسخ:**
>
> ```bash
> #!/usr/bin/env bash
> set -uo pipefail   # -e را برمی‌داریم تا یک خطا کل حلقه را نکشد
>
> run_on_host() {
>     local host="$1"
>     local out
>     if out=$(ssh -o ConnectTimeout=5 -o BatchMode=yes "$host" 'bash -s' < ./check.sh 2>&1); then
>         printf '%-20s OK\n' "$host"
>     else
>         printf '%-20s FAIL: %s\n' "$host" "$out"
>     fi
> }
> export -f run_on_host
>
> # اجرای موازی با حداکثر ۱۰ هم‌زمان
> xargs -P 10 -I{} bash -c 'run_on_host "$@"' _ {} < servers.txt
> ```
>
> **درس:** `xargs -P` موازی‌سازی ساده و بدون وابستگی است.

**Lab Scenario 2 — Ansible برای پیکربندی یکسان**

> **وظیفه:** اطمینان حاصل کن که روی ۲۰ سرور، NTP و SSH مطابق استاندارد است.
>
> **پاسخ:**
>
> ```yaml
> ---
> - name: Baseline سرورها
>   hosts: all
>   become: true
>   tasks:
>     - name: chrony نصب
>       package: { name: chrony, state: present }
>
>     - name: config NTP
>       copy:
>         dest: /etc/chrony.conf
>         content: |
>           server ntp1.corp.example.com iburst
>           server ntp2.corp.example.com iburst
>       notify: restart chrony
>
>     - name: SSH hardening
>       lineinfile:
>         path: /etc/ssh/sshd_config
>         regexp: '^#?PermitRootLogin'
>         line: 'PermitRootLogin no'
>       notify: restart sshd
>
>     - name: سرویس‌ها فعال
>       service: { name: "{{ item }}", state: started, enabled: true }
>       loop: [chronyd, sshd]
>
>   handlers:
>     - name: restart chrony
>       service: { name: chronyd, state: restarted }
>     - name: restart sshd
>       service: { name: sshd, state: restarted }
> ```
>
> **درس:** `lineinfile` با `regexp` — idempotent، حتی اگر خط موجود باشد.

### مقایسه و الگوهای مختلف

**مقایسهٔ SSH و RDP:**

| معیار | SSH | RDP |
|---|---|---|
| نوع | متنی | گرافیکی |
| پهنای باند | کم | زیاد |
| امنیت | بالا (با کلید) | نیاز به NLA |
| خودکارسازی | ✅ | محدود |
| مناسب | لینوکس، اسکریپت | ویندوز، GUI |

**مقایسهٔ ابزارهای Config Management:**

| ابزار | زبان | مدل | مناسب |
|---|---|---|---|
| **Ansible** | YAML | push/agentless | اکثر موارد |
| **Puppet** | DSL | pull/agent | enterprise بزرگ |
| **Chef** | Ruby | pull/agent | DevOps بالغ |
| **SaltStack** | YAML/Python | push/pull | مقیاس بالا |

**مقایسهٔ Cron و Timer (تکمیلی):**

| معیار | Cron | systemd timer |
|---|---|---|
| granularity | دقیقه | ثانیه |
| missed run | ❌ | `Persistent=true` |
| dependency | ❌ | ✅ |
| لاگ | syslog | journald |

### اشکال شایع

**۱. RDP باز روی اینترنت.**
هدف اول brute force و ransomware. همیشه VPN/bastion.

**۲. SSH با رمز به‌جای کلید.**
کلید امن‌تر و قابل خودکارسازی است.

**۳. `PasswordAuthentication yes` بعد از راه‌اندازی کلید.**
کلید را بی‌فایده می‌کند. غیرفعال کن.

**۴. اجرای Ansible بدون `--check`.**
تغییر ناخواسته روی ۵۰ سرور.

**۵. playbook غیر idempotent.**
`shell:` به‌جای ماژول idempotent. بار دوم هم `changed`.

**۶. نبود bastion host.**
هر سرور مستقیماً در دسترس است.

**۷. اسکریپت خودکار بدون لاگ.**
نمی‌دانی چه شد.

**۸. بدون rate limit در اجرای موازی.**
۵۰ اتصال هم‌زمان، شبکه یا سرور مقصد را می‌خواباند.

### نکات مهم (Tips & Tricks)

- **Bastion host** برای همهٔ دسترسی‌های مدیریتی.
- **SSH key با passphrase** + `ssh-agent`.
- **`~/.ssh/config`** برای alias و تنظیمات:

```
Host srv01
    HostName 10.0.10.21
    User deploy
    IdentityFile ~/.ssh/id_ed25519
```

- **`ansible-playbook --check --diff`** قبل از اجرا.
- **`xargs -P`** برای موازی‌سازی ساده.
- **در آزمون:** کلیدواژهٔ «agentless» → **Ansible**؛ «PowerShell remoting» → **WinRM**.

### خلاصه فصل

- پروتکل‌ها: **SSH، RDP، WinRM، WMI، SNMP، Redfish**.
- **SSH key** به‌جای رمز؛ `PasswordAuthentication no`.
- **PowerShell Remoting** با `Invoke-Command`.
- **Ansible**: YAML، push، agentless، idempotent.
- **`--check`** قبل از اجرا.
- **Bastion host** برای دسترسی مدیریتی.

---

**بعدی:** [فایل ۱۱: امنیت فیزیکی و امنیت داده](11-Physical-and-Data-Security.md)

---

## منابع و مراجع

- اهداف رسمی Server+ (V5)، دامنهٔ ۲: Server Administration (Scripting)
- Microsoft Learn — PowerShell documentation، about_Operators، about_Try_Catch_Finally
- GNU Bash Reference Manual: <https://www.gnu.org/software/bash/manual/>
- ShellCheck: <https://www.shellcheck.net/>
- Ansible Documentation: <https://docs.ansible.com/>
- OpenSSH manual pages
- `man 5 crontab`، `man systemd.timer`
- Google Shell Style Guide
- <https://www.comptia.org/en-us/certifications/server/>
