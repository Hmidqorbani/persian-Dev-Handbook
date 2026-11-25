# سرویس‌های شبکه

**مدت یادگیری**: ۷ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فصل ۲۳ تا ۲۷ — نصب و پیکربندی اولیهٔ سیستم‌عامل
**دامنهٔ آزمون**: Server Administration (۳۰٪)

---

## فهرست محتوا

- [فصل ۲۸: آدرس‌دهی IP و subnetting برای مدیر سرور](#ch-28)
- [فصل ۲۹: DNS در سرور](#ch-29)
- [فصل ۳۰: DHCP و مدیریت آدرس](#ch-30)
- [فصل ۳۱: VLAN، سوئیچینگ و جداسازی شبکه](#ch-31)
- [فصل ۳۲: NTP و سرویس‌های پایهٔ زیرساخت](#ch-32)

---

## فصل ۲۸: آدرس‌دهی IP و subnetting برای مدیر سرور

### مقدمه

هر سرور یک آدرس دارد، و اگر آن آدرس اشتباه باشد، هیچ چیز کار نمی‌کند. اما subnetting فقط «تعیین IP» نیست؛ ابزار **طراحی** است: تعیین اینکه چه چیزی با چه چیزی صحبت کند، کجا broadcast برود، و کجا جداسازی امنیتی اعمال شود.

این فصل، محاسبات لازم برای مدیر سرور را می‌سازد: CIDR، subnet mask، شبکه/broadcast، تعداد هاست، و تقسیم زیرشبکه.

### مفاهیم اساسی

**IPv4 — ساختار:**

```
192.168.10.25
│   │   │  │
└───┴───┴──┴── ۴ اکتت (هر کدام ۰–۲۵۵، ۸ بیت)
```

مجموع: ۳۲ بیت.

**CIDR notation:** `192.168.10.25/24` — عدد بعد از `/` تعداد بیت‌های **شبکه** است.

**Subnet mask:**

| CIDR | Mask | تعداد آدرس | هاست قابل استفاده |
|---|---|---|---|
| /30 | 255.255.255.252 | ۴ | **۲** |
| /29 | 255.255.255.248 | ۸ | **۶** |
| /28 | 255.255.255.240 | ۱۶ | **۱۴** |
| /27 | 255.255.255.224 | ۳۲ | **۳۰** |
| /26 | 255.255.255.192 | ۶۴ | **۶۲** |
| /25 | 255.255.255.128 | ۱۲۸ | **۱۲۶** |
| **/24** | 255.255.255.0 | ۲۵۶ | **۲۵۴** |
| /23 | 255.255.254.0 | ۵۱۲ | ۵۱۰ |
| /22 | 255.255.252.0 | ۱۰۲۴ | ۱۰۲۲ |
| /16 | 255.255.0.0 | ۶۵٬۵۳۶ | ۶۵٬۵۳۴ |
| /8 | 255.0.0.0 | ۱۶٬۷۷۷٬۲۱۶ | ۱۶٬۷۷۷٬۲۱۴ |

**فرمول‌ها:**

```
تعداد آدرس کل = ۲^(۳۲ − CIDR)
هاست قابل استفاده = ۲^(۳۲ − CIDR) − ۲
```

**چرا منهای ۲؟** چون اولین آدرس **network address** و آخرین **broadcast address** است.

**مثال کامل:** `10.20.30.0/26`

```
/26 → ۲^(۳۲−۲۶) = ۲^۶ = ۶۴ آدرس
هاست: ۶۴ − ۲ = ۶۲

Network:   10.20.30.0
اولین هاست: 10.20.30.1
آخرین هاست: 10.20.30.62
Broadcast: 10.20.30.63
Next subnet: 10.20.30.64
```

**محاسبهٔ سریع بلوک:** `block size = 256 − octet mask`

مثال: /26 → mask octet = 192 → block = 256 − 192 = **64**. پس subnetها با گام ۶۴ هستند: ۰، ۶۴، ۱۲۸، ۱۹۲.

**کلاس‌های IP (تاریخی، اما در آزمون می‌آید):**

| کلاس | محدودهٔ اولین اکتت | Mask پیش‌فرض | کاربرد |
|---|---|---|---|
| A | ۱–۱۲۶ | /8 | شبکه‌های بسیار بزرگ |
| B | ۱۲۸–۱۹۱ | /16 | متوسط |
| C | ۱۹۲–۲۲۳ | /24 | کوچک |
| D | ۲۲۴–۲۳۹ | — | multicast |
| E | ۲۴۰–۲۵۵ | — | آزمایشی |

**آدرس‌های خصوصی (RFC 1918):**

| محدوده | CIDR | تعداد آدرس |
|---|---|---|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | ۱۶.۷ میلیون |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | ۱ میلیون |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | ۶۵٬۵۳۶ |

**آدرس‌های ویژه:**

| محدوده | کاربرد |
|---|---|
| 127.0.0.0/8 | loopback |
| 169.254.0.0/16 | link-local (APIPA) |
| 0.0.0.0 | «هر رابط» یا «نامشخص» |
| 255.255.255.255 | broadcast محدود |
| 224.0.0.0/4 | multicast |
| 100.64.0.0/10 | CGNAT (RFC 6598) |
| 192.0.2.0/24، 198.51.100.0/24، 203.0.113.0/24 | مستندات (RFC 5737) |

**نکتهٔ مهم:** آدرس‌های `192.0.2.x`، `198.51.100.x` و `203.0.113.x` برای **مستندات و مثال** رزرو شده‌اند. در این کتابچه از همین‌ها استفاده می‌کنم تا هیچ آدرس واقعی در مثال نیاید.

**IPv6 — خلاصهٔ لازم:**

- ۱۲۸ بیت، نمایش هگزادسیمال: `2001:db8::1/64`
- **بدون broadcast**، اما multicast دارد.
- **SLAAC** — پیکربندی خودکار بدون DHCP.
- **Link-local** — `fe80::/10`، همیشه موجود.
- **Loopback** — `::1`
- آدرس مستندات: `2001:db8::/32`

**نکته:** Server+ عمق زیادی در IPv6 نمی‌خواهد، اما باید بدانی که وجود دارد و سرورها معمولاً dual-stack هستند.

**Default Gateway و routing:**

- اگر مقصد در همان subnet باشد → ARP مستقیم.
- اگر خارج باشد → به gateway.
- **تلهٔ رایج:** gateway باید در همان subnet سرور باشد، وگرنه مسیر کار نمی‌کند.

**مثال تله:** سرور `10.0.10.50/24` با gateway `10.0.20.1`. غلط است، چون `10.0.20.1` خارج از `10.0.10.0/24` است.

### مثال‌های عملی

**Lab Scenario 1 — تقسیم یک شبکه**

> **وضعیت:** شبکهٔ `10.50.0.0/24` داری. باید ۴ زیرشبکه بسازی که هر کدام حداقل ۵۰ هاست بدهد.
>
> **وظیفه:** CIDR و محدوده‌ها.
>
> **پاسخ:**
> - نیاز: ۵۰ هاست → ۲^n − ۲ ≥ ۵۰ → n = ۶ → ۶۴ آدرس → **/26**
> - ۴ زیرشبکهٔ /26 در یک /24:
>
> | # | Network | محدودهٔ هاست | Broadcast |
> |---|---|---|---|
> | ۱ | 10.50.0.0/26 | .1 – .62 | .63 |
> | ۲ | 10.50.0.64/26 | .65 – .126 | .127 |
> | ۳ | 10.50.0.128/26 | .129 – .190 | .191 |
> | ۴ | 10.50.0.192/26 | .193 – .254 | .255 |
>
> **درس:** `/24` را با `/26` به چهار قسمت تقسیم کردیم و هر کدام ۶۲ هاست داد.

**Lab Scenario 2 — عیب‌یابی آدرس‌دهی**

> **وضعیت:** سرور جدید `10.0.5.200/28` گرفته. gateway `10.0.5.193`. کاربران به سرور دسترسی ندارند.
>
> **وظیفه:** مشکل چیست؟
>
> **پاسخ:**
> - /28 → block = ۲۵۶ − ۲۴۰ = ۱۶
> - subnetها: .0، .16، .32، ... .192، .208
> - `10.0.5.200` در subnet `10.0.5.192/28` است: محدودهٔ .193–.206، broadcast .207
> - **IP .200 معتبر است** ✓
> - **gateway .193** — این آدرس network نیست (network .192 است)، پس معتبر است ✓
>
> **پس مشکل آدرس نیست.** باید چیز دیگری چک شود: VLAN، فایروال، ARP، یا route.
>
> **درس:** قبل از عیب‌یابی شبکه، **درستی آدرس‌دهی** را تأیید کن — اما اگر درست بود، وقت را تلف نکن.

### مقایسه و الگوهای مختلف

**سه رویکرد به آدرس‌دهی:**

| رویکرد | توضیح | مزیت | عیب |
|---|---|---|---|
| **Flat** | یک subnet بزرگ | ساده | broadcast زیاد، امنیت کم |
| **Subnet به تفکیک نقش** | هر نقش یک subnet | جداسازی، امنیت | نیاز به routing |
| **Subnet به تفکیک طبقه/سایت** | جغرافیایی | مقیاس | پیچیدگی |

**توصیه:** حداقل تفکیک: مدیریت، سرورها، کاربران، مهمان، DMZ، ذخیره‌سازی.

**مقایسهٔ اندازهٔ subnet:**

| اندازه | مناسب | نکته |
|---|---|---|
| /24 | عمومی، رایج | ۲۵۴ هاست، broadcast معقول |
| /25–/26 | بخش‌های کوچک | کاهش broadcast |
| /27–/28 | نقطه‌به‌نقطه، سرورهای خاص | مدیریت دقیق |
| /30 | لینک point-to-point | فقط ۲ هاست |
| /31 | point-to-point (RFC 3021) | ۲ هاست بدون network/broadcast |
| /32 | loopback، host route | یک آدرس |

**مقایسهٔ IPv4 و IPv6:**

| معیار | IPv4 | IPv6 |
|---|---|---|
| طول آدرس | ۳۲ بیت | ۱۲۸ بیت |
| نمایش | dotted decimal | hex با `:` |
| Broadcast | ✅ | ❌ (multicast) |
| NAT | رایج | معمولاً نه |
| پیکربندی خودکار | DHCP | SLAAC + DHCPv6 |
| Header | متغیر | ساده‌تر، ثابت |

### اشکال شایع

**۱. اشتباه در محاسبهٔ broadcast.**
broadcast = آخرین آدرس subnet، نه آدرس شبکهٔ بعدی.

**۲. گذاشتن gateway خارج از subnet.**
سرور نمی‌تواند route کند. همیشه gateway باید در همان subnet باشد.

**۳. استفاده از آدرس network یا broadcast برای هاست.**
در /24، آدرس .0 و .255 قابل استفاده نیستند (در /31 استثنا).

**۴. subnet mask ناهماهنگ.**
سرور /24 و gateway /25 → بعضی مقاصد «محلی» دیده می‌شوند که نیستند.

**۵. IP تکراری.**
باعث قطعی متناوب و ARP conflict. همیشه IP management داشته باش.

**۶. فراموش کردن رزرو آدرس‌های زیرساختی.**
DNS، gateway، DHCP، BMC — همه باید در محدودهٔ ثابت و مستند باشند.

**۷. بی‌توجهی به APIPA.**
اگر سرور `169.254.x.x` گرفت، یعنی DHCP نرسیده. این یک علامت است، نه یک آدرس.

### نکات مهم (Tips & Tricks)

- **جدول subnet را حفظ کن** (/24 تا /30). در آزمون وقت محاسبه نیست.
- **`block = 256 − mask octet`** — سریع‌ترین روش محاسبه.
- **برای هر VLAN، یک subnet.** نگه داشتن نقشه در CMDB/NetBox.
- **آدرس‌های زیرساختی را در ابتدای رنج بگذار** (مثلاً .1–.20 برای gateway، DNS، BMC).
- **در آزمون:** اگر سؤال «how many usable hosts in /27» داد → **۳۰** (نه ۳۲).
- **برای مثال، از 192.0.2.x استفاده کن** — آدرس مستندات است.

### خلاصه فصل

- CIDR = تعداد بیت شبکه؛ هاست = ۲^(۳۲−CIDR) − ۲.
- `block size = 256 − mask octet`.
- آدرس‌های خصوصی: 10/8، 172.16/12، 192.168/16.
- gateway باید در همان subnet باشد.
- تفکیک subnet = جداسازی امنیتی + کاهش broadcast.
- /31 برای point-to-point (۲ هاست).

---

## فصل ۲۹: DNS در سرور

### مقدمه

DNS «دفترچه تلفن اینترنت» است — اما در سرور، چیزی بسیار بیشتر: ستون فقرات Active Directory، سرویس‌های داخلی، load balancing و امنیت.

بیشتر خرابی‌های «سرویس کار نمی‌کند» به DNS برمی‌گردد. این فصل، DNS را از دید مدیر سرور توضیح می‌دهد.

### مفاهیم اساسی

**سلسله‌مراتب DNS:**

```
. (root)
└── com
    └── example
        └── www.example.com
```

**انواع سرور DNS:**

| نوع | توضیح |
|---|---|
| **Recursive resolver** | پاسخ کامل برای کلاینت پیدا می‌کند (مثلاً 8.8.8.8) |
| **Authoritative** | پاسخ نهایی برای یک zone را دارد |
| **Forwarder** | درخواست را به سرور دیگر می‌فرستد |
| **Caching-only** | فقط cache می‌کند |

**انواع رکورد:**

| رکورد | کاربرد | مثال |
|---|---|---|
| **A** | نام → IPv4 | `web01 A 10.0.10.21` |
| **AAAA** | نام → IPv6 | `web01 AAAA 2001:db8::21` |
| **CNAME** | نام → نام دیگر | `www CNAME web01` |
| **MX** | سرور ایمیل | `example.com MX 10 mail.example.com` |
| **TXT** | متن (SPF، DKIM، تأیید) | `v=spf1 ...` |
| **NS** | سرور نام zone | `example.com NS ns1` |
| **SOA** | اطلاعات zone | serial، refresh، retry |
| **PTR** | IP → نام (reverse) | `21.10.0.10.in-addr.arpa PTR web01` |
| **SRV** | سرویس خاص | `_ldap._tcp...` |
| **CAA** | مجوز CA برای گواهی | `example.com CAA 0 issue "letsencrypt.org"` |

**Zone و delegation:**

- **Forward zone** — نام به IP.
- **Reverse zone** — IP به نام (`in-addr.arpa`).
- **Primary** — قابل نوشتن.
- **Secondary** — کپی، از primary انتقال می‌گیرد (AXFR/IXFR).
- **Stub** — فقط NS records.
- **Conditional forwarder** — برای نام‌های خاص به سرور خاص.

**نکتهٔ حیاتی برای Active Directory:** AD به DNS وابسته است. رکوردهای `_msdcs`، `_sites`، `_tcp`، `_udp` برای پیدا کردن DC لازم‌اند. اگر DNS اشتباه باشد، join دامنه شکست می‌خورد.

**TTL:** زمان زنده بودن cache. TTL کوتاه = تغییر سریع، اما query بیشتر. TTL بلند = بار کمتر، اما تغییر کند.

**پیکربندی BIND (لینوکس) — نمونه:**

```bash
# /etc/bind/named.conf.local
zone "corp.example.com" {
    type master;
    file "/etc/bind/db.corp.example.com";
};

# /etc/bind/db.corp.example.com
$TTL 3600
@   IN  SOA ns1.corp.example.com. admin.corp.example.com. (
        2026111201 ; serial (YYYYMMDDnn)
        3600       ; refresh
        900        ; retry
        604800     ; expire
        86400 )    ; minimum
@       IN  NS  ns1.corp.example.com.
ns1     IN  A   10.0.10.5
web01   IN  A   10.0.10.21
db01    IN  A   10.0.10.31
www     IN  CNAME web01
```

**نکتهٔ مهم:** **serial** باید با هر تغییر افزایش یابد، وگرنه secondaryها تغییر را نمی‌بینند. فرمت رایج: `YYYYMMDDnn`.

**پیکربندی ویندوز (PowerShell):**

```powershell
Install-WindowsFeature -Name DNS -IncludeManagementTools

Add-DnsServerPrimaryZone -Name "corp.example.com" -ZoneFile "corp.example.com.dns"

Add-DnsServerResourceRecordA -Name "web01" -ZoneName "corp.example.com" `
    -IPv4Address "10.0.10.21"

Add-DnsServerResourceRecordCName -Name "www" -HostNameAlias "web01.corp.example.com" `
    -ZoneName "corp.example.com"

Get-DnsServerResourceRecord -ZoneName "corp.example.com"
```

**ابزارهای عیب‌یابی:**

| ابزار | کاربرد |
|---|---|
| `nslookup` | ساده، همه‌جا |
| `dig` | دقیق، لینوکس |
| `host` | ساده |
| `Resolve-DnsName` | PowerShell |
| `dnscmd` | مدیریت ویندوز |

**نمونه:**

```bash
dig @10.0.10.5 web01.corp.example.com A +short
dig @10.0.10.5 corp.example.com AXFR    # تست zone transfer
```

**امنیت DNS:**

- **Open resolver** — سروری که برای همه پاسخ می‌دهد = سوءاستفاده در DDoS amplification. باید محدود شود.
- **DNSSEC** — امضای رکوردها، جلوگیری از جعل.
- **Response Rate Limiting (RRL)** — محدودیت نرخ پاسخ.
- **Split-horizon** — پاسخ متفاوت برای داخلی و خارجی.
- **Zone transfer restriction** — فقط به secondaryهای مجاز.

### مثال‌های عملی

**Lab Scenario 1 — راه‌اندازی DNS داخلی**

> **وظیفه:** یک zone برای `corp.example.com` با سه رکورد A و یک CNAME بساز (BIND).
>
> **پاسخ:** (فایل بالا) — و تست:
>
> ```bash
> named-checkzone corp.example.com /etc/bind/db.corp.example.com
> systemctl restart bind9
> dig @localhost web01.corp.example.com +short
> # خروجی: 10.0.10.21
> ```
>
> **درس:** `named-checkzone` قبل از restart، از خطای سینتکس جلوگیری می‌کند.

**Lab Scenario 2 — سرویس داخلی resolve نمی‌شود**

> **وضعیت:** کاربران `intranet.corp.example.com` را باز نمی‌کنند، اما `ping 10.0.10.50` کار می‌کند.
>
> **وظیفه:** عیب‌یابی.
>
> **پاسخ:**
> ۱. **از سرور DNS بپرس:** `dig @10.0.10.5 intranet.corp.example.com` — آیا رکورد وجود دارد؟
> ۲. **اگر نه:** رکورد اضافه نشده یا serial افزایش نیافته.
> ۳. **اگر بله، از کلاینت بپرس:** `nslookup intranet.corp.example.com` — کدام DNS پاسخ می‌دهد؟
> ۴. **ممکن است کلاینت DNS دیگری داشته باشد** (مثلاً از DHCP).
> ۵. **cache:** TTL قدیمی، پاسخ منفی cache شده. `ipconfig /flushdns` یا `systemd-resolve --flush-caches`.
>
> **درس:** همیشه از منبع شروع کن (سرور DNS)، بعد به کلاینت برو.

### مقایسه و الگوهای مختلف

**BIND در برابر Windows DNS در برابر dnsmasq:**

| معیار | BIND | Windows DNS | dnsmasq |
|---|---|---|---|
| پلتفرم | لینوکس/یونیکس | ویندوز سرور | لینوکس/روتر |
| AD integration | نیاز به تنظیم | بومی | ❌ |
| پیچیدگی | بالا | متوسط | کم |
| مناسب برای | DNS عمومی/enterprise | AD | شعبه، lab |
| DNSSEC | ✅ | ✅ | محدود |

**مقایسهٔ recursive و forwarder:**

| معیار | Recursive | Forwarder |
|---|---|---|
| بار | بالا (query به root) | کم |
| کنترل | کامل | وابسته به upstream |
| cache | مؤثر | مؤثر |
| مناسب برای | ISP، سازمان بزرگ | سازمان متوسط |

**مقایسهٔ split-horizon:**

| سناریو | پاسخ داخلی | پاسخ خارجی |
|---|---|---|
| سرویس داخلی | IP خصوصی | هیچ (یا IP عمومی) |
| وب‌سایت | IP داخلی | IP عمومی |
| مزیت | امنیت، عملکرد | — |

### اشکال شایع

**۱. open resolver.**
سرور DNS برای همهٔ اینترنت پاسخ می‌دهد → سوءاستفاده در amplification attack.

**۲. فراموش کردن افزایش serial.**
secondary تغییر را نمی‌بیند و پاسخ قدیمی می‌دهد.

**۳. zone transfer آزاد.**
هر کسی می‌تواند کل zone را دانلود کند (نقشهٔ شبکه). محدود به secondaryهای مجاز.

**۴. TTL خیلی بلند.**
تغییر IP، ساعت‌ها طول می‌کشد تا اعمال شود.

**۵. CNAME برای apex.**
`example.com CNAME ...` غیرمجاز است (باید A/AAAA باشد).

**۶. عدم هماهنگی forward/reverse.**
PTR اشتباه، در لاگ و ایمیل مشکل می‌سازد.

**۷. DNS روی DC بدون backup.**
اگر DC/DNS بخوابد و secondary نباشد، کل دامنه می‌خوابد.

**۸. استفاده از DNS عمومی برای سرورهای داخلی.**
سرور داخلی باید DNS داخلی داشته باشد، وگرنه نام‌های داخلی resolve نمی‌شوند.

### نکات مهم (Tips & Tricks)

- **دو سرور DNS** حداقل. یکی primary، یکی secondary.
- **`dig +trace`** مسیر resolve را نشان می‌دهد — عالی برای عیب‌یابی.
- **`named-checkconf` و `named-checkzone`** قبل از restart.
- **در AD، همیشه `_msdcs` را چک کن** اگر join مشکل دارد.
- **TTL را برای رکوردهای در حال تغییر، موقتاً کم کن** (مثلاً ۳۰۰) و بعد از تغییر برگردان.
- **در آزمون:** کلیدواژهٔ «name resolution fails» → **DNS**؛ «cannot join domain» → **DNS first**.

### خلاصه فصل

- DNS سلسله‌مراتبی است؛ zone، رکورد، TTL.
- رکوردها: **A، AAAA، CNAME، MX، TXT، NS، SOA، PTR، SRV**.
- **serial** را با هر تغییر افزایش بده.
- **open resolver** و **zone transfer آزاد** ریسک امنیتی‌اند.
- AD به DNS وابسته است؛ `_msdcs` حیاتی است.
- حداقل دو سرور DNS.

---

## فصل ۳۰: DHCP و مدیریت آدرس

### مقدمه

DHCP آدرس IP را خودکار توزیع می‌کند. در شبکهٔ ۵۰۰ کلاینتی، تخصیص دستی غیرممکن است. اما DHCP فقط «دادن IP» نیست: gateway، DNS، NTP، domain، boot server هم می‌دهد.

این فصل دربارهٔ DORA، scope، reservation، relay و failover است.

### مفاهیم اساسی

**فرآیند DORA:**

```
۱. Discover  کلاینت: «کسی DHCP هست؟» (broadcast)
۲. Offer     سرور: «من میتوانم این IP را بدهم»
۳. Request   کلاینت: «آن را میخواهم»
۴. Ack       سرور: «بفرما، تا این مدت مال توست»
```

**انواع تخصیص:**

| نوع | توضیح | کاربرد |
|---|---|---|
| **Dynamic** | از pool، با lease | کلاینت‌های معمولی |
| **Automatic** | از pool، دائمی | ایستگاه‌های ثابت |
| **Manual / Reservation** | IP مشخص به MAC مشخص | چاپگر، سرور، دوربین |

**نکتهٔ مهم:** برای **سرورها**، همیشه IP static (روی خود سرور) یا reservation. DHCP خالص برای سرور، ریسک است.

**Scope — اجزا:**

- **Range** — محدودهٔ آدرس.
- **Subnet mask** — mask شبکه.
- **Lease duration** — مدت اجاره.
- **Options** — gateway (3)، DNS (6)، domain (15)، NTP (42)، TFTP (66)، bootfile (67).
- **Exclusions** — آدرس‌هایی که توزیع نشوند (مثلاً برای تجهیزات ثابت).
- **Reservations** — IP به MAC.

**Lease duration — انتخاب:**

| محیط | Lease |
|---|---|
| شبکهٔ اداری پایدار | ۸ روز |
| Wi-Fi مهمان | ۲–۸ ساعت |
| آزمایشگاه | ۱ روز |
| شبکهٔ با دستگاه زیاد و متغیر | ۴–۱۲ ساعت |

**نکته:** lease کوتاه = IP بیشتر آزاد می‌شود، اما DHCP traffic بیشتر. lease بلند = ترافیک کمتر، اما pool سریع پر می‌شود.

**DHCP Relay (IP Helper):** DHCP Discover یک broadcast است و از روتر عبور نمی‌کند. برای شبکه‌های VLAN-بندی‌شده:

```
[کلاینت VLAN 10] --broadcast--> [روتر] --unicast به DHCP--> [سرور DHCP VLAN 99]
```

روتر با `ip helper-address 10.0.99.10` تنظیم می‌شود.

**گزینهٔ ۸۲ (Option 82):** روتر اطلاعاتی دربارهٔ پورت و VLAN اضافه می‌کند. برای شناسایی محل کلاینت مفید است.

**DHCP Failover (ویندوز):**

| حالت | توضیح |
|---|---|
| **Load balance** | هر دو سرور فعال، درصد تقسیم |
| **Hot standby** | یکی فعال، یکی standby |

**در لینوکس:** `dhcpd` با `failover peer` یا `kea` با HA.

**پیکربندی ISC DHCP (نمونه):**

```bash
# /etc/dhcp/dhcpd.conf
option domain-name "corp.example.com";
option domain-name-servers 10.0.10.5, 10.0.10.6;
option routers 10.0.10.1;
default-lease-time 28800;
max-lease-time 86400;

subnet 10.0.10.0 netmask 255.255.255.0 {
    range 10.0.10.100 10.0.10.200;
    option ntp-servers 10.0.10.5;
}

host printer01 {
    hardware ethernet aa:bb:cc:dd:ee:ff;
    fixed-address 10.0.10.50;
}
```

**پیکربندی ویندوز (PowerShell):**

```powershell
Install-WindowsFeature -Name DHCP -IncludeManagementTools

Add-DhcpServerv4Scope -Name "LAN" -StartRange 10.0.10.100 `
    -EndRange 10.0.10.200 -SubnetMask 255.255.255.0 -State Active

Set-DhcpServerv4OptionValue -DnsServer 10.0.10.5,10.0.10.6 `
    -Router 10.0.10.1 -DnsDomain "corp.example.com"

Add-DhcpServerv4Reservation -ScopeId 10.0.10.0 `
    -IPAddress 10.0.10.50 -ClientId "AA-BB-CC-DD-EE-FF" -Name "printer01"

Get-DhcpServerv4Lease -ScopeId 10.0.10.0
```

**IP Address Management (IPAM):** ابزار ثبت و مدیریت آدرس‌ها. بدون IPAM، IP تکراری و آدرس‌های «گم‌شده» رایج می‌شود.

**امنیت DHCP:**

- **Rogue DHCP server** — سرور جعلی که gateway/DNS اشتباه می‌دهد (حملهٔ MITM).
- **DHCP snooping** — روی سوئیچ، پورت‌های مجاز DHCP را مشخص می‌کند.
- **Port security** — محدودیت MAC در هر پورت.

### مثال‌های عملی

**Lab Scenario 1 — طراحی scope برای سه VLAN**

> **وضعیت:** سه VLAN: 10 (کاربران، ۱۵۰)، 20 (مهمان، ۱۰۰)، 30 (سرورها، ۳۰). سرور DHCP در VLAN 99.
>
> **وظیفه:** scopeها و relay.
>
> **پاسخ:**
>
> | VLAN | Subnet | Range | Lease |
> |---|---|---|---|
> | 10 | 10.0.10.0/24 | .100–.220 | ۸ روز |
> | 20 | 10.0.20.0/25 | .10–.120 | ۴ ساعت |
> | 30 | 10.0.30.0/27 | — (static) | — |
>
> - VLAN 30: سرورها static، DHCP نه.
> - روتر: `ip helper-address 10.0.99.10` روی interfaceهای VLAN 10 و 20.
> - مهمان: lease کوتاه + DNS محدود + ACL به شبکهٔ داخلی.
>
> **درس:** هر VLAN یک scope؛ مهمان، lease کوتاه و جداسازی.

**Lab Scenario 2 — کلاینت IP نمی‌گیرد**

> **وضعیت:** کلاینت `169.254.10.5` گرفته.
>
> **وظیفه:** عیب‌یابی.
>
> **پاسخ:**
> ۱. **آیا کابل/لینک هست؟** LED، `ip link`.
> ۲. **آیا DHCP در همان VLAN است؟** اگر VLAN دیگر است، relay تنظیم شده؟
> ۳. **آیا scope تمام شده؟** (`Get-DhcpServerv4Statistics` — آمار in-use)
> ۴. **آیا سرور DHCP روشن است؟** سرویس، سرویس‌های وابسته.
> ۵. **آیا DHCP snooping پورت را بسته؟**
> ۶. **آیا MAC filter یا reservation اشتباه است؟**
>
> **درس:** `169.254.x.x` یعنی «DHCP نرسید» — از لایهٔ ۱ شروع کن.

### مقایسه و الگوهای مختلف

**DHCP در برابر static در برابر reservation:**

| روش | مدیریت | مناسب |
|---|---|---|
| **Static** | دستی روی هر دستگاه | سرور، تجهیزات شبکه |
| **Reservation** | مرکزی در DHCP | چاپگر، دوربین، AP |
| **Dynamic** | خودکار | کلاینت کاربر |

**مقایسهٔ ISC DHCP و Kea:**

| معیار | ISC DHCP | Kea |
|---|---|---|
| وضعیت | maintenance (EOL برنامه‌ریزی‌شده) | فعال |
| پایگاه داده | فایل/OMAPI | SQL (MySQL/PostgreSQL) |
| HA | failover قدیمی | HA built-in |
| API | محدود | REST |
| توصیه | مهاجرت تدریجی | ✅ جدید |

**نکته:** ISC DHCP در حال بازنشستگی است؛ برای نصب جدید، **Kea** انتخاب بهتری است.

**مقایسهٔ DHCPv4 و DHCPv6:**

| معیار | DHCPv4 | DHCPv6 |
|---|---|---|
| Broadcast | ✅ | ❌ (multicast) |
| SLAAC | ❌ | ✅ (جایگزین) |
| Options | کامل | متفاوت |
| کاربرد | همه‌جا | مکمل SLAAC |

### اشکال شایع

**۱. Rogue DHCP.**
یک روتر خانگی به شبکه وصل می‌شود و IP اشتباه می‌دهد. راه‌حل: DHCP snooping.

**۲. Scope تمام‌شده.**
pool کوچک، lease بلند، دستگاه زیاد. پایش استفاده ضروری است.

**۳. relay فراموش‌شده.**
VLAN جدید ساخته شده اما `ip helper-address` تنظیم نشده.

**۴. exclusion نکردن آدرس‌های ثابت.**
DHCP همان IP سرور را به کلاینت می‌دهد → conflict.

**۵. lease خیلی بلند در شبکهٔ متغیر.**
pool پر می‌شود با دستگاه‌هایی که رفته‌اند.

**۶. DHCP روی همان سرور DC بدون افزونگی.**
اگر سرور بخوابد، شبکه IP نمی‌گیرد و کاربران نمی‌توانند وارد دامنه شوند.

**۷. بی‌توجهی به Option 66/67 برای PXE.**
نصب شبکه‌ای کار نمی‌کند.

### نکات مهم (Tips & Tricks)

- **آدرس‌های زیرساختی را exclusion کن** (.1–.50).
- **reservation برای هر دستگاه ثابت** — حتی اگر static هم بگذاری.
- **IPAM داشته باش** — NetBox، phpIPAM، یا حتی یک Excel به‌روز.
- **در محیط مجازی، MAC را قبل از reservation بگیر** (گاهی MAC خودکار عوض می‌شود).
- **بعد از تغییر scope، `Release/Renew` روی کلاینت تست کن.**
- **در آزمون:** کلیدواژهٔ «169.254» → **APIPA / DHCP unavailable**؛ «different subnet» → **relay/helper**.

### خلاصه فصل

- DORA: **Discover، Offer، Request، Ack**.
- سه نوع تخصیص: **dynamic، automatic، manual/reservation**.
- **Relay/helper** برای عبور از روتر لازم است.
- lease کوتاه برای مهمان، بلند برای اداری.
- **DHCP snooping** جلوی rogue server را می‌گیرد.
- سرورها static یا reservation، نه dynamic.

---

## فصل ۳۱: VLAN، سوئیچینگ و جداسازی شبکه

### مقدمه

VLAN یک شبکهٔ فیزیکی را به چند شبکهٔ منطقی تقسیم می‌کند — بدون کابل‌کشی جدید. این ساده‌ترین و مؤثرترین ابزار جداسازی امنیتی و کاهش broadcast است.

این فصل دربارهٔ VLAN، trunk، tagging و طراحی جداسازی است.

### مفاهیم اساسی

**VLAN چیست؟** یک دامنهٔ broadcast منطقی. دستگاه‌های یک VLAN، مثل اینکه به یک سوئیچ جدا وصل‌اند، با هم صحبت می‌کنند.

**انواع پورت سوئیچ:**

| نوع | توضیح | ترافیک |
|---|---|---|
| **Access** | یک VLAN، برای دستگاه نهایی | untagged |
| **Trunk** | چند VLAN، بین سوئیچ‌ها | **tagged** (802.1Q) |
| **Hybrid** | ترکیب | هر دو |

**802.1Q tagging:** یک تگ ۴ بایتی به فریم اضافه می‌شود که VLAN ID (۱۲ بیت، ۱–۴۰۹۴) را نشان می‌دهد.

```
[فریم اصلی] → [تگ 802.1Q: VLAN 20] → [فریم trunk]
```

**Native VLAN:** VLANای که روی trunk **بدون تگ** عبور می‌کند. پیش‌فرض VLAN 1 است.

> ⚠️ **هشدار امنیتی:** native VLAN باید از VLAN 1 تغییر کند. حملات **VLAN hopping** از native VLAN و double tagging سوءاستفاده می‌کنند.

**VLAN IDهای مهم:**

| VLAN | کاربرد |
|---|---|
| 1 | default (استفاده نکن) |
| 2–1001 | normal range |
| 1002–1005 | legacy (Token Ring، FDDI) |
| 1006–4094 | extended range |

**Inter-VLAN routing:** VLANها جدا هستند؛ برای ارتباط، روتر یا سوئیچ لایهٔ ۳ لازم است.

- **Router-on-a-stick** — یک رابط فیزیکی با subinterface برای هر VLAN.
- **SVI (Switch Virtual Interface)** — سوئیچ لایهٔ ۳ با interface مجازی برای هر VLAN. سریع‌تر.

**طراحی VLAN — الگوی پیشنهادی:**

| VLAN | کاربرد | مثال subnet |
|---|---|---|
| 10 | مدیریت (سرورها، سوئیچ‌ها) | 10.0.10.0/24 |
| 20 | کاربران | 10.0.20.0/23 |
| 30 | مهمان | 10.0.30.0/24 |
| 40 | DMZ | 10.0.40.0/27 |
| 50 | ذخیره‌سازی (iSCSI) | 10.0.50.0/24 |
| 60 | VoIP | 10.0.60.0/24 |
| 99 | OOB/BMC | 10.0.99.0/24 |

**نکتهٔ مهم:** VLAN جداسازی **logical** است، نه فیزیکی. اگر دو VLAN از یک سوئیچ عبور کنند و سوئیچ compromise شود، جداسازی می‌شکند. برای جداسازی قوی، **فیزیکی** یا **firewall** لازم است.

**Private VLAN:** تقسیم یک VLAN به sub-VLAN (isolated، community، promiscuous). برای محیط‌های multi-tenant.

**STP (Spanning Tree Protocol):** جلوگیری از loop در شبکهٔ سوئیچی. نسخه‌های RSTP (سریع‌تر)، MSTP (چند نمونه).

**نکته:** loop در لایهٔ ۲ = broadcast storm = شبکهٔ کامل از کار می‌افتد. STP ضروری است، اما **BPDU Guard** و **Root Guard** هم برای امنیت.

**Link aggregation (LACP):** چند لینک فیزیکی به یک لینک منطقی. افزونگی + پهنای باند.

**امنیت لایهٔ ۲:**

| حمله | توضیح | دفاع |
|---|---|---|
| **VLAN hopping** | عبور از VLAN | native VLAN غیر ۱، غیرفعال کردن DTP |
| **MAC flooding** | پر کردن CAM table | port security |
| **ARP spoofing** | جعل ARP | Dynamic ARP Inspection |
| **DHCP starvation** | پر کردن pool | DHCP snooping + rate limit |
| **STP attack** | تصاحب root bridge | BPDU Guard، Root Guard |

### مثال‌های عملی

**Lab Scenario 1 — طراحی VLAN برای یک شرکت**

> **وضعیت:** ۱۲۰ کاربر، ۱۵ سرور، ۱۰ دوربین IP، مهمان Wi-Fi. یک سوئیچ لایهٔ ۳.
>
> **وظیفه:** VLANها و routing.
>
> **پاسخ:**
>
> | VLAN | نام | Subnet | توضیح |
> |---|---|---|---|
> | 10 | MGMT | 10.0.10.0/26 | مدیریت سوئیچ، BMC |
> | 20 | USERS | 10.0.20.0/25 | کاربران |
> | 30 | SERVERS | 10.0.30.0/26 | سرورها |
> | 40 | CCTV | 10.0.40.0/27 | دوربین‌ها (بدون اینترنت) |
> | 50 | GUEST | 10.0.50.0/25 | مهمان (فقط اینترنت) |
>
> - Inter-VLAN: SVI روی سوئیچ لایهٔ ۳.
> - ACL: GUEST → فقط اینترنت؛ CCTV → فقط NVR.
> - trunk بین سوئیچ‌ها با native VLAN 999.
>
> **درس:** جداسازی بر اساس **نقش و سطح اعتماد**، نه بر اساس طبقه.

**Lab Scenario 2 — VLAN کار نمی‌کند**

> **وضعیت:** کلاینت در VLAN 20، IP گرفته اما نمی‌تواند به gateway پینگ کند.
>
> **وظیفه:** عیب‌یابی.
>
> **پاسخ:**
> ۱. **آیا پورت access در VLAN درست است؟** `show vlan brief` / `switchport access vlan 20`
> ۲. **آیا trunk، VLAN 20 را مجاز کرده؟** `switchport trunk allowed vlan`
> ۳. **آیا SVI برای VLAN 20 بالاست؟** `show ip interface brief`
> ۴. **آیا native VLAN mismatch روی trunk؟** (CDP/LLDP نشان می‌دهد)
> ۵. **آیا ACL جلوی ICMP را گرفته؟**
>
> **درس:** ۹۰٪ مشکلات VLAN: access port، allowed list، یا SVI down.

### مقایسه و الگوهای مختلف

**Router-on-a-stick در برابر SVI:**

| معیار | Router-on-a-stick | SVI (L3 switch) |
|---|---|---|
| تجهیز | روتر + سوئیچ L2 | سوئیچ L3 |
| عملکرد | محدود به پهنای باند روتر | بالاتر (ASIC) |
| پیچیدگی | کم | متوسط |
| مناسب برای | شبکهٔ کوچک | سازمانی |

**مقایسهٔ VLAN و subnet:**

| معیار | VLAN | Subnet |
|---|---|---|
| لایه | ۲ | ۳ |
| دامنهٔ broadcast | تعریف می‌کند | تعریف می‌کند |
| ارتباط | نیاز به routing | نیاز به routing |
| نسبت | معمولاً ۱:۱ | معمولاً ۱:۱ |

**نکته:** در طراحی خوب، **یک VLAN = یک subnet**. استثناها وجود دارد (مثلاً چند subnet در یک VLAN برای مهاجرت)، اما قاعدهٔ کلی ۱:۱ است.

**مقایسهٔ trunk و access:**

| معیار | Access | Trunk |
|---|---|---|
| تعداد VLAN | ۱ | چند |
| تگ | ندارد | دارد |
| دستگاه | کلاینت، سرور | سوئیچ، hypervisor، AP |
| امنیت | ساده | نیاز به محدود کردن allowed VLAN |

### اشکال شایع

**۱. نگه داشتن native VLAN روی ۱.**
ریسک VLAN hopping. همیشه تغییر بده.

**۲. trunk با allowed VLAN = all.**
هر VLAN روی هر trunk = سطح حملهٔ بیشتر. فقط لازم‌ها را مجاز کن.

**۳. VLAN بدون subnet.**
گیج‌کننده. یک VLAN = یک subnet.

**۴. فراموش کردن routing.**
VLANها ساخته شده اما کسی نمی‌تواند به دیگری برسد. SVI یا روتر لازم است.

**۵. VLAN 1 برای مدیریت.**
VLAN 1 هرگز برای مدیریت استفاده نشود.

**۶. بی‌توجهی به STP.**
یک loop، کل شبکه را می‌خواباند.

**۷. VLAN برای جداسازی امنیتی قوی.**
VLAN جداسازی logical است. برای DMZ یا multi-tenant، firewall فیزیکی لازم است.

### نکات مهم (Tips & Tricks)

- **VLAN ID را مستند کن** — در NetBox یا حتی یک جدول.
- **نام‌گذاری معنادار:** `VLAN10-MGMT` نه `VLAN10`.
- **native VLAN را ۹۹۹ کن** و روی هیچ access port استفاده نکن.
- **`show interfaces trunk`** بهترین دستور عیب‌یابی trunk است.
- **BPDU Guard روی access port** — جلوی اتصال سوئیچ غیرمجاز.
- **در آزمون:** کلیدواژهٔ «devices in different VLANs cannot communicate» → **routing/SVI needed**.

### خلاصه فصل

- VLAN = دامنهٔ broadcast منطقی.
- **Access** (untagged) در برابر **Trunk** (tagged 802.1Q).
- **Native VLAN** را از ۱ تغییر بده (ضد VLAN hopping).
- Inter-VLAN routing با **SVI** یا **router-on-a-stick**.
- یک VLAN = یک subnet.
- VLAN جداسازی logical است، نه فیزیکی.

---

## فصل ۳۲: NTP و سرویس‌های پایهٔ زیرساخت

### مقدمه

NTP، بی‌سر‌و‌صداترین سرویس زیرساخت است — تا روزی که همه چیز خراب شود. Kerberos شکست می‌خورد، گواهی‌ها معتبر نیستند، لاگ‌ها بی‌ترتیب می‌شوند و کسی نمی‌فهمد چرا.

این فصل NTP و چند سرویس پایهٔ دیگر (syslog، SNMP، LDAP) را پوشش می‌دهد که «پایه» هستند: بدون آن‌ها، بقیهٔ سرویس‌ها قابل مدیریت نیستند.

### مفاهیم اساسی

**NTP — چرا حیاتی است؟**

| سرویس | وابستگی به زمان |
|---|---|
| **Kerberos (AD)** | حداکثر ۵ دقیقه اختلاف |
| **TLS/SSL** | اعتبار گواهی |
| **Log correlation** | ترتیب رخداد |
| **Backup/schedule** | زمان اجرا |
| **Database** | تراکنش توزیع‌شده |
| **Replication** | ترتیب تغییرات |

**سلسله‌مراتب NTP (stratum):**

```
Stratum 0: ساعت اتمی، GPS
Stratum 1: سرور متصل مستقیم به stratum 0
Stratum 2: سرور متصل به stratum 1
...
Stratum 15: حداکثر
Stratum 16: unsynchronized
```

**نکته:** سرورهای داخلی معمولاً stratum 2 یا 3 هستند. برای سازمان، یک یا دو سرور NTP داخلی که به منبع خارجی sync می‌شوند، کافی است.

**پروتکل‌ها:**

- **NTP** (UDP 123) — استاندارد، دقت میلی‌ثانیه.
- **SNTP** — ساده‌شده، دقت کمتر.
- **PTP (IEEE 1588)** — دقت میکروثانیه/نانوثانیه، برای مالی و صنعتی.

**پیکربندی chrony (لینوکس مدرن):**

```bash
# /etc/chrony.conf
pool 2.pool.ntp.org iburst
# یا سرور داخلی
server ntp1.corp.example.com iburst prefer
server ntp2.corp.example.com iburst

# اجازه به کلاینت‌های داخلی (اگر سرور NTP هستی)
allow 10.0.0.0/16

# بررسی
chronyc sources -v
chronyc tracking
chronyc clients
```

**پیکربندی ویندوز:**

```powershell
# به‌عنوان کلاینت
w32tm /config /manualpeerlist:"ntp1.corp.example.com,0x8 ntp2.corp.example.com,0x8" `
    /syncfromflags:manual /reliable:no /update
Restart-Service w32time

# به‌عنوان سرور NTP برای دامنه
w32tm /config /reliable:yes /update

# بررسی
w32tm /query /status
w32tm /query /peers
w32tm /stripchart /computer:ntp1.corp.example.com
```

**نکتهٔ AD:** سرور PDC Emulator باید به منبع خارجی sync شود؛ بقیهٔ DCها به PDC. این سلسله‌مراتب، زمان دامنه را یکنواخت می‌کند.

**Syslog — جمع‌آوری لاگ:**

| سطح | معنی |
|---|---|
| 0 | emerg |
| 1 | alert |
| 2 | crit |
| 3 | err |
| 4 | warning |
| 5 | notice |
| 6 | info |
| 7 | debug |

**پیکربندی rsyslog:**

```bash
# /etc/rsyslog.conf (کلاینت)
*.* @@10.0.10.60:514    # TCP
# یا
*.* @10.0.10.60:514     # UDP

# سرور: فعال کردن دریافت
module(load="imudp")
input(type="imudp" port="514")
module(load="imtcp")
input(type="imtcp" port="514")
```

**نکته:** TCP مطمئن‌تر است (بدون از دست رفتن)، UDP سبک‌تر. برای امنیت، **TLS** (rsyslog با GnuTLS) یا ارسال به SIEM.

**SNMP — پایش:**

| نسخه | امنیت |
|---|---|
| v1 | community string ساده (امنیت صفر) |
| v2c | community string (بهتر، اما بدون رمزنگاری) |
| **v3** | احراز هویت + رمزنگاری |

**نکتهٔ امنیتی:** SNMP v1/v2c را در تولید استفاده نکن. community پیش‌فرض (`public`، `private`) را عوض کن.

**OID و MIB:** SNMP با OID (مثل `1.3.6.1.2.1.1.3.0` = uptime) داده می‌دهد. MIB توضیح می‌دهد هر OID چیست.

**LDAP/AD — دایرکتوری:**

- **LDAP** (پورت 389، LDAPS 636) — پروتکل دسترسی.
- **AD** — پیاده‌سازی مایکروسافت با Kerberos.
- **OpenLDAP** — پیاده‌سازی متن‌باز.

**سرویس‌های پایهٔ دیگر:**

| سرویس | پورت | کاربرد |
|---|---|---|
| **NTP** | 123/UDP | زمان |
| **Syslog** | 514 | لاگ |
| **SNMP** | 161/UDP، 162 (trap) | پایش |
| **LDAP** | 389، 636 | دایرکتوری |
| **Kerberos** | 88 | احراز هویت |
| **RADIUS** | 1812/1813 | احراز هویت شبکه |
| **TACACS+** | 49 | مدیریت تجهیزات |

### مثال‌های عملی

**Lab Scenario 1 — راه‌اندازی NTP داخلی**

> **وظیفه:** یک سرور NTP داخلی برای ۵۰ سرور و ۲۰۰ کلاینت.
>
> **پاسخ:**
> ۱. **سرور NTP:** chrony با دو منبع خارجی (`pool`)، `allow 10.0.0.0/16`.
> ۲. **فایروال:** UDP 123 از شبکهٔ داخلی به سرور NTP.
> ۳. **کلاینت‌ها:** `server ntp.corp.example.com iburst`.
> ۴. **DC:** PDC Emulator به سرور NTP داخلی.
> ۵. **پایش:** هشدار اگر `chronyc tracking` نشان دهد offset > ۱۰۰ms.
>
> **درس:** یک منبع زمان مرکزی، همه چیز را هماهنگ می‌کند.

**Lab Scenario 2 — لاگ‌ها زمان اشتباه دارند**

> **وضعیت:** در SIEM، لاگ سرورها با ۲ ساعت اختلاف زمانی می‌آید.
>
> **وظیفه:** علت و راه‌حل.
>
> **پاسخ:**
> - **علت ۱: timezone.** سرور روی UTC، SIEM روی محلی. راه‌حل: همه UTC یا همه یک timezone.
> - **علت ۲: NTP کار نمی‌کند.** `chronyc tracking` نشان می‌دهد `Not synchronised`.
> - **علت ۳: فایروال UDP 123 را بسته.** بررسی با `chronyc sources`.
> - **راه‌حل:** NTP را درست کن، timezone را استاندارد کن، لاگ را با UTC ذخیره کن و در نمایش تبدیل کن.
>
> **درس:** در SIEM، زمان استاندارد = UTC. نمایش، محلی.

### مقایسه و الگوهای مختلف

**NTP در برابر PTP:**

| معیار | NTP | PTP |
|---|---|---|
| دقت | میلی‌ثانیه | میکرو/نانوثانیه |
| نیاز به سخت‌افزار | کم | timestamp سخت‌افزاری |
| پیچیدگی | کم | بالا |
| کاربرد | عمومی | مالی، صنعتی، مخابرات |

**chrony در برابر ntpd:**

| معیار | chrony | ntpd |
|---|---|---|
| sync اولیه | سریع | کند |
| محیط متغیر | بهتر | ضعیف‌تر |
| مصرف منابع | کم | بیشتر |
| توصیه | ✅ | میراثی |

**مقایسهٔ syslog متمرکز و محلی:**

| معیار | متمرکز | محلی |
|---|---|---|
| حفظ لاگ در خرابی سرور | ✅ | ❌ |
| correlation | ✅ | ❌ |
| حجم | زیاد | کم |
| توصیه | ✅ تولیدی | فقط آزمایش |

### اشکال شایع

**۱. NTP تنظیم‌نشده روی DC.**
Kerberos شکست می‌خورد و کاربران نمی‌توانند وارد شوند.

**۲. timezone اشتباه.**
لاگ‌ها گمراه‌کننده می‌شوند. بهترین کار: **UTC روی همه سرورها**.

**۳. syslog محلی بدون ارسال.**
سرور می‌سوزد و لاگ‌ها هم می‌سوزند — دقیقاً وقتی که لازم‌اند.

**۴. SNMP v2c با community پیش‌فرض.**
`public` = دسترسی خواندن به همه چیز.

**۵. UDP 123 بسته.**
NTP بی‌صدا شکست می‌خورد. همیشه `chronyc sources` را چک کن.

**۶. سرور NTP داخلی بدون منبع خارجی.**
زمان داخلی هماهنگ است اما غلط.

**۷. بی‌توجه به clock drift در VM.**
VMها می‌توانند drift داشته باشند؛ در hypervisor، time sync را فعال کن.

### نکات مهم (Tips & Tricks)

- **همه چیز UTC.** تبدیل به محلی، فقط در نمایش.
- **`chronyc tracking`** اولین دستور عیب‌یابی زمان است.
- **دو منبع NTP** حداقل — یکی ممکن است از دسترس خارج شود.
- **syslog را به SIEM بفرست**، حتی اگر SIEM ساده باشد.
- **SNMP v3 با auth+priv** برای هر چیز تولیدی.
- **در آزمون:** کلیدواژهٔ «Kerberos authentication fails» → **time skew / NTP**.

### خلاصه فصل

- NTP برای **Kerberos، TLS، لاگ، backup** حیاتی است.
- **stratum** سلسله‌مراتب زمان است.
- **chrony** جایگزین مدرن ntpd است.
- **syslog متمرکز** برای حفظ و correlation لازم است.
- **SNMP v3** برای امنیت؛ v2c فقط در شبکهٔ بسته.
- همه سرورها **UTC**.

---

**بعدی:** [فایل ۰۷: مدیریت و پایش سرور](07-Server-Administration-and-Monitoring.md)

---

## منابع و مراجع

- اهداف رسمی Server+ (V5)، دامنهٔ ۲: Server Administration
- RFC 1918 — آدرس‌های خصوصی
- RFC 5737 — آدرس‌های مستندات
- RFC 3021 — /31 برای point-to-point
- RFC 2131 — DHCP
- RFC 1035 — DNS
- RFC 5905 — NTPv4
- IEEE 802.1Q — VLAN tagging
- ISC Kea DHCP: <https://www.isc.org/kea/>
- chrony documentation: <https://chrony-project.org/>
- <https://www.comptia.org/en-us/certifications/server/>
