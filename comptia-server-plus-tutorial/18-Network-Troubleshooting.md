# عیب‌یابی شبکه

**مدت یادگیری**: ۷ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فصل ۸۴ تا ۸۸ — عیب‌یابی سیستم‌عامل
**دامنهٔ آزمون**: Troubleshooting (۲۸٪)

---

## فهرست محتوا

- [فصل ۸۹: عیب‌یابی لایه‌ای شبکه](#ch-89)
- [فصل ۹۰: تأخیر، افت بسته و نوسان](#ch-90)
- [فصل ۹۱: پیکربندی نادرست — DNS، DHCP، VLAN، routing](#ch-91)
- [فصل ۹۲: ابزارهای تحلیل شبکه](#ch-92)
- [فصل ۹۳: نفوذ شبکه و پاسخ به رخداد](#ch-93)

---

## فصل ۸۹: عیب‌یابی لایه‌ای شبکه

### مقدمه

شبکه، لایه‌ای است. و خرابی در هر لایه، علامت متفاوتی دارد. اگر لایه را اشتباه تشخیص بدهی، ابزار اشتباه استفاده می‌کنی و وقت تلف می‌شود.

این فصل، مدل OSI را به **رویهٔ عملی** تبدیل می‌کند.

### مفاهیم اساسی

**مدل OSI — هفت لایه:**

| # | لایه | مثال | علامت خرابی |
|---|---|---|---|
| ۷ | Application | HTTP، DNS | خطای اپلیکیشن |
| ۶ | Presentation | TLS، encoding | خطای گواهی |
| ۵ | Session | session management | قطع session |
| ۴ | Transport | TCP، UDP | connection refused/timeout |
| ۳ | Network | IP، ICMP | no route، unreachable |
| ۲ | Data Link | Ethernet، ARP، VLAN | no link، VLAN mismatch |
| ۱ | Physical | کابل، برق | no link light |

**نکتهٔ حیاتی:** در آزمون Server+، معمولاً مدل **TCP/IP (چهار لایه)** کافی است: **Link → Internet → Transport → Application**.

**رویکردهای عیب‌یابی:**

| رویکرد | شروع | مناسب |
|---|---|---|
| **Bottom-up** | لایه ۱ (کابل) | خرابی فیزیکی مشکوک |
| **Top-down** | لایه ۷ (اپ) | مشکل اپلیکیشن |
| **Divide and conquer** | لایه ۳ (ping) | نامشخص |

**توصیهٔ عملی:** از **ping** شروع کن. اگر جواب داد، لایه ۱–۳ سالم است و مشکل بالاتر است. اگر نداد، پایین برو.

**تست لایه به لایه:**

```
لایه ۱: LED کارت، کابل، سوئیچ
لایه ۲: ARP، MAC، VLAN
لایه ۳: ping gateway، ping 8.8.8.8، route
لایه ۴: telnet/nc به پورت، firewall
لایه ۷: curl، dig، مرورگر
```

**نکتهٔ حیاتی:** **ping فقط لایه ۳ است.** اگر ping جواب می‌دهد، یعنی IP کار می‌کند — نه اینکه سرویس کار می‌کند، نه اینکه DNS کار می‌کند، نه اینکه فایروال اجازه می‌دهد.

**ماتریس تشخیص:**

| ping IP | ping نام | nc پورت | نتیجه |
|---|---|---|---|
| ❌ | ❌ | ❌ | لایه ۱–۳: کابل، IP، route |
| ✅ | ❌ | ❌ | **DNS** |
| ✅ | ✅ | ❌ | **فایروال/سرویس** |
| ✅ | ✅ | ✅ | لایه ۷: اپلیکیشن |

**این ماتریس، ارزش حفظ کردن دارد.**

**علامت‌های کلیدی:**

| علامت | معنی |
|---|---|
| **no link light** | لایه ۱: کابل، پورت، NIC |
| **link light، no ping** | لایه ۲/۳: VLAN، IP، route |
| **ping gateway، no internet** | route یا ISP |
| **ping IP، no name** | DNS |
| **connection refused** | سرویس گوش نمی‌دهد |
| **connection timeout** | فایروال یا مسیر |
| **intermittent** | کابل، duplex، wireless |

**نکته:** **connection refused** با **timeout** فرق دارد:
- `refused` = بسته رسید، اما سرویس روی آن پورت نیست (RST).
- `timeout` = بسته رسید یا نرسید، پاسخی نیامد (فایروال drop).

این تفاوت، مسیر عیب‌یابی را تعیین می‌کند.

**ARP و لایه ۲:**

```bash
ip neigh
arp -a
```

**نکته:** اگر ARP برای gateway خالی است، لایه ۲ مشکل دارد (VLAN، کابل، سوئیچ).

**VLAN — تست:**

- روی سوئیچ: `show vlan brief`، `show interfaces trunk`
- روی سرور: `ip -d link` (آیا VLAN tag دارد؟)

**نکته:** اگر سرور در VLAN 10 و پورت سوئیچ در VLAN 20 باشد، هیچ چیز کار نمی‌کند — اما LED سبز است. **link light، VLAN را تضمین نمی‌کند.**

**Route — تست:**

```bash
ip route
ip route get 8.8.8.8
traceroute 8.8.8.8
```

**نکته:** اگر default gateway تنظیم نشده، فقط شبکهٔ محلی در دسترس است.

### مثال‌های عملی

**Lab Scenario 1 — ماتریس تشخیص**

> **وضعیت:** کاربر می‌گوید «اینترنت قطع است». `ping 8.8.8.8` جواب می‌دهد، اما `ping google.com` نه.
>
> **وظیفه:** لایه و علت.
>
> **پاسخ:**
> - ping IP ✅، ping نام ❌ → **DNS**
> - **بررسی:** `nslookup google.com`، `/etc/resolv.conf`، `ipconfig /all`
> - **علت‌ها:** DNS سرور اشتباه، DNS down، فایروال UDP 53، cache خراب.
>
> **درس:** «اینترنت قطع» می‌تواند فقط DNS باشد.

**Lab Scenario 2 — link هست، هیچ چیز کار نمی‌کند**

> **وضعیت:** سرور جدید نصب شد. LED NIC سبز، اما ping gateway جواب نمی‌دهد.
>
> **وظیفه:** سه احتمال لایه ۲/۳.
>
> **پاسخ:**
> ۱. **VLAN mismatch:** پورت سوئیچ در VLAN اشتباه.
> ۲. **IP/subnet اشتباه:** IP خارج از subnet gateway.
> ۳. **Duplex/speed mismatch:** نادر امروز، اما ممکن.
>
> **درس:** LED سبز فقط لایه ۱ را تأیید می‌کند.

### مقایسه و الگوهای مختلف

**مقایسهٔ Bottom-up و Top-down:**

| معیار | Bottom-up | Top-down |
|---|---|---|
| شروع | کابل | اپلیکیشن |
| سرعت (خرابی فیزیکی) | سریع | کند |
| سرعت (خرابی اپ) | کند | سریع |
| توصیه | مشکوک به فیزیکی | مشکوک به اپ |

**مقایسهٔ ping و nc:**

| معیار | ping (ICMP) | nc (TCP) |
|---|---|---|
| لایه | ۳ | ۴ |
| چه می‌سنجد | دسترس‌پذیری IP | سرویس روی پورت |
| فایروال | ممکن است block شود | دقیق‌تر |

**نکته:** بسیاری از فایروال‌ها ICMP را block می‌کنند. **ping شکست، همیشه یعنی قطعی نیست.**

**مقایسهٔ traceroute و mtr:**

| معیار | traceroute | mtr |
|---|---|---|
| تک‌بار | ✅ | ✅ |
| پیوسته | ❌ | ✅ |
| loss در هر hop | ❌ | ✅ |
| مناسب | مسیر | کیفیت |

### اشکال شایع

**۱. فرض «ping = سالم».**
ping فقط لایه ۳ است.

**۲. اشتباه refused و timeout.**
مسیر عیب‌یابی متفاوت.

**۳. نادیده گرفتن DNS.**
«اینترنت قطع» اغلب DNS است.

**۴. VLAN را چک نکردن.**
link سبز، VLAN اشتباه.

**۵. تست از سرور به‌جای کلاینت.**
مسیر متفاوت است.

**۶. ping بلاک‌شده.**
فایروال ICMP را می‌بندد.

**۷. بدون تست دوطرفه.**
فایروال یک‌طرفه.

**۸. نادیده گرفتن MTU.**
پکت بزرگ می‌شکند، کوچک نه.

### نکات مهم (Tips & Tricks)

- **ماتریس ping IP / ping نام / nc پورت** را حفظ کن.
- **`nc -zv host port`** سریع‌ترین تست پورت.
- **`ip route get`** مسیر واقعی را نشان می‌دهد.
- **`mtr`** بهتر از traceroute.
- **دو طرف** را تست کن (سرور → کلاینت، کلاینت → سرور).
- **در آزمون:** کلیدواژهٔ «ping IP works, name fails» → **DNS**.

### خلاصه فصل

- لایه‌ها: **Link، Internet، Transport، Application**.
- **ping = لایه ۳**، نه بیشتر.
- ماتریس: **IP / نام / پورت** → لایهٔ مشکل.
- **refused** = سرویس نیست؛ **timeout** = فایروال/مسیر.
- **link light** فقط لایه ۱.
- **DNS** شایع‌ترین علت «اینترنت قطع».

---

## فصل ۹۰: تأخیر، افت بسته و نوسان

### مقدمه

«شبکه کند است» سه مشکل متفاوت است: **تأخیر (latency)**، **افت بسته (loss)**، و **نوسان (jitter)**. هر کدام علت و درمان متفاوتی دارند.

این فصل آن‌ها را جدا می‌کند.

### مفاهیم اساسی

**سه متریک:**

| متریک | تعریف | اثر |
|---|---|---|
| **Latency** | تأخیر رفت‌وبرگشت (RTT) | کندی تعاملی |
| **Packet loss** | درصد بستهٔ گم‌شده | retransmit، کندی، قطعی |
| **Jitter** | تغییر latency | کیفیت VoIP، ویدیو |

**نکتهٔ حیاتی:** این سه، **مستقل** هستند. شبکه می‌تواند latency کم و loss بالا داشته باشد، یا برعکس.

**محدوده‌های قابل قبول:**

| متریک | LAN | WAN | اینترنت |
|---|---|---|---|
| **Latency** | <۱ms | <۲۰ms | <۱۰۰ms |
| **Loss** | ۰٪ | <۰.۱٪ | <۱٪ |
| **Jitter** | <۱ms | <۵ms | <۳۰ms |

**نکته:** برای VoIP، jitter >۳۰ms محسوس است. برای دیتابیس، latency >۵ms بین سرور و storage مشکوک است.

**علل latency:**

| علت | توضیح |
|---|---|
| **فاصلهٔ فیزیکی** | سرعت نور، غیرقابل رفع |
| **تعداد hop** | هر روتر، تأخیر |
| **صف (queuing)** | ازدحام |
| **پردازش** | فایروال، proxy، DPI |
| **retransmit** | loss باعث تأخیر می‌شود |
| **serialization** | لینک کند |

**نکته:** **فاصله** غیرقابل رفع است. تهران به اروپا ≈ ۸۰–۱۲۰ms. هیچ tuning این را حل نمی‌کند.

**علل packet loss:**

| علت | علامت |
|---|---|
| **کابل معیوب** | loss در یک لینک |
| **duplex mismatch** | loss + collision |
| **ازدحام** | loss در ساعت اوج |
| **buffer کوچک** | loss با burst |
| **خطای NIC** | CRC error |
| **wireless** | loss متغیر |
| **فایروال/IDS** | drop عمدی |

**نکتهٔ حیاتی:** **duplex mismatch** کلاسیک است: یک طرف full، طرف دیگر half → collision و loss شدید. امروز نادر است (autoneg)، اما با تنظیم دستی رخ می‌دهد.

**تشخیص duplex:**

```bash
ethtool eth0
# Speed: 1000Mb/s
# Duplex: Full
# Link detected: yes
```

**نکته:** اگر یک طرف `Full` و طرف دیگر `Half` باشد، loss و کندی شدید.

**علل jitter:**

- ازدحام متغیر.
- مسیرهای متفاوت (load balancing).
- bufferbloat.
- wireless.

**نکته:** **bufferbloat** — بافر بزرگ در روتر، تأخیر را در بار بالا افزایش می‌دهد. راه‌حل: AQM (fq_codel، CAKE).

**ابزارهای اندازه‌گیری:**

| ابزار | چه می‌سنجد |
|---|---|
| **ping** | RTT و loss |
| **mtr** | loss در هر hop |
| **iperf3** | throughput، jitter |
| **`ss -ti`** | retransmit، RTT |
| **`sar -n DEV`** | throughput تاریخی |
| **Wireshark** | تحلیل دقیق |

**نکتهٔ مهم:** **ping ممکن است در فایروال deprioritize شود.** اگر ping loss دارد اما سرویس خوب است، ping را ملاک نگیر.

**iperf3 — نمونه:**

```bash
# سرور
iperf3 -s

# کلاینت (TCP)
iperf3 -c 10.0.10.21 -t 30

# UDP با jitter
iperf3 -c 10.0.10.21 -u -b 100M -t 30
# خروجی: Jitter، Lost/Total
```

**نکته:** برای تست شبکه، **UDP** jitter و loss را بهتر نشان می‌دهد؛ **TCP** throughput واقعی.

**retransmit — تشخیص:**

```bash
ss -ti
# retrans، rtt
netstat -s | grep -i retrans
```

**نکته:** retransmit بالا (>۱٪) یعنی loss. TCP آن را پنهان می‌کند اما با هزینهٔ latency.

**MTU و fragmentation:**

```bash
# تست MTU
ping -M do -s 1472 8.8.8.8
# 1472 + 28 = 1500
```

**نکته:** اگر پکت بزرگ می‌شکند و کوچک نه، **MTU mismatch** است. رایج در VPN، iSCSI، tunnel.

**علامت MTU:** ping کوچک کار می‌کند، SSH/HTTP بزرگ گیر می‌کند (در handshake).

### مثال‌های عملی

**Lab Scenario 1 — loss در یک لینک**

> **وظیفه:** `mtr` نشان می‌دهد loss از hop ۳ به بعد ۱۰٪ است، اما hop ۱ و ۲ صفر.
>
> **پاسخ:**
> - **loss از hop ۳** = مشکل در لینک بین hop ۲ و ۳، یا خود hop ۳.
> - **توجه:** loss در hop میانی می‌تواند **ICMP deprioritization** باشد، نه loss واقعی.
> - **تأیید:** loss در **hop آخر** مهم است؛ loss میانی با ادامهٔ صفر، اغلب توهم است.
>
> **درس:** در mtr، loss **end-to-end** مهم است، نه loss یک hop.

**Lab Scenario 2 — سرور کند، شبکه سالم**

> **وضعیت:** انتقال فایل از سرور ۱۰MB/s است، لینک 10GbE.
>
> **وظیفه:** سه احتمال.
>
> **پاسخ:**
> ۱. **دیسک:** منبع یا مقصد کند (HDD).
> ۲. **پروتکل:** SMB بدون tuning، یا تک‌رشته.
> ۳. **retransmit/MTU:** `ss -ti`، تست MTU.
>
> **درس:** «شبکه کند» اغلب شبکه نیست.

### مقایسه و الگوهای مختلف

**مقایسهٔ latency و throughput:**

| معیار | Latency | Throughput |
|---|---|---|
| چه می‌سنجد | تأخیر | حجم |
| اثر بر | تعاملی | انتقال بزرگ |
| ابزار | ping | iperf |

**نکته:** **پهنای باند بالا، latency را کم نمی‌کند.** یک لینک ۱۰۰Gbps با latency ۱۰۰ms، برای دیتابیس بد است.

**مقایسهٔ TCP و UDP در تست:**

| معیار | TCP | UDP |
|---|---|---|
| loss | پنهان (retransmit) | آشکار |
| throughput | واقعی | خام |
| jitter | غیرمستقیم | مستقیم |
| مناسب | کار واقعی | کیفیت لینک |

**مقایسهٔ ping و mtr:**

| معیار | ping | mtr |
|---|---|---|
| مسیر | ❌ | ✅ |
| پیوسته | با `-c` | ✅ |
| loss per hop | ❌ | ✅ |
| مناسب | سریع | تحلیل |

### اشکال شایع

**۱. تفسیر loss میانی به‌عنوان loss واقعی.**
ICMP deprioritization.

**۲. ملاک گرفتن ping.**
ping ممکن است کند شود.

**۳. فرض «پهنای باند = سرعت».**
latency هم مهم است.

**۴. نادیده گرفتن duplex.**
loss و collision.

**۵. MTU mismatch.**
پکت بزرگ گیر می‌کند.

**۶. تست با یک رشته.**
throughput تک‌رشته، ظرفیت کل را نشان نمی‌دهد.

**۷. تست در ساعت کم‌بار.**
ازدحام دیده نمی‌شود.

**۸. بدون baseline.**
نمی‌دانی «عادی» چیست.

### نکات مهم (Tips & Tricks)

- **سه متریک را جدا کن:** latency، loss، jitter.
- **`mtr`** برای loss per hop.
- **loss در hop آخر** مهم است.
- **`iperf3 -u`** برای jitter.
- **`ethtool`** برای duplex.
- **در آزمون:** کلیدواژهٔ «packet loss with collisions» → **duplex mismatch**.

### خلاصه فصل

- سه متریک: **latency، loss، jitter** — مستقل.
- **فاصله** غیرقابل رفع است.
- **duplex mismatch** → loss و collision.
- **loss میانی** ممکن است توهم باشد.
- **MTU mismatch** → پکت بزرگ گیر می‌کند.
- **پهنای باند ≠ سرعت**.

---

## فصل ۹۱: پیکربندی نادرست — DNS، DHCP، VLAN، routing

### مقدمه

بیشتر خرابی‌های شبکه، خرابی سخت‌افزار نیست؛ **پیکربندی نادرست** است. و این‌ها با یک اشتباه کوچک، ساعت‌ها وقت می‌گیرند.

این فصل، پنج اشتباه رایج را با علامت و رفعشان می‌آورد.

### مفاهیم اساسی

**۱. DNS — شایع‌ترین**

| علامت | علت |
|---|---|
| **ping IP ✅، ping نام ❌** | DNS |
| **resolve کند** | DNS سرور دور، timeout |
| **resolve اشتباه** | رکورد قدیمی، cache |
| **فقط بعضی نام‌ها** | split-horizon، forwarder |
| **join دامنه شکست** | DNS اشتباه |

**بررسی:**

```bash
cat /etc/resolv.conf
resolvectl status
nslookup name
dig @10.0.10.5 name
```

```powershell
Get-DnsClientServerAddress
Resolve-DnsName name
ipconfig /all
```

**نکتهٔ حیاتی:** **DNS سرور باید در همان شبکه یا قابل دسترس باشد.** اگر سرور DNS روی `10.0.10.5` است و سرور در `10.0.20.0/24` بدون route، resolve شکست می‌خورد.

**۲. DHCP — دومین**

| علامت | علت |
|---|---|
| **IP = 169.254.x.x** | DHCP نرسید (APIPA) |
| **IP از subnet اشتباه** | rogue DHCP، relay |
| **IP تکراری** | conflict، reservation اشتباه |
| **IP می‌گیرد، gateway ندارد** | option 3 |
| **IP می‌گیرد، DNS ندارد** | option 6 |

**بررسی:**

```bash
ip a
dhclient -v
journalctl -u dhcpcd
```

```powershell
ipconfig /all
ipconfig /release
ipconfig /renew
```

**نکته:** `169.254.x.x` (APIPA) یعنی «DHCP نرسید». از لایهٔ ۱ شروع کن، بعد relay.

**۳. VLAN — سومین**

| علامت | علت |
|---|---|
| **link سبز، no ping** | access port در VLAN اشتباه |
| **فقط بعضی VLANها** | trunk allowed list |
| **بعد از جابه‌جایی پورت** | VLAN پیش‌فرض |
| **native VLAN mismatch** | دو طرف trunk |

**بررسی (سوئیچ):**

```
show vlan brief
show interfaces status
show interfaces trunk
show interfaces switchport
```

**نکته:** در trunk، **allowed VLAN list** را چک کن. اگر VLAN 20 در لیست نباشد، ترافیک عبور نمی‌کند حتی اگر در هر دو طرف تعریف شده باشد.

**۴. Routing — چهارمین**

| علامت | علت |
|---|---|
| **فقط شبکهٔ محلی** | default gateway نبود |
| **بعضی مقاصد** | route اشتباه |
| **loop** | route حلقه‌ای |
| **asymmetric** | مسیر برگشت متفاوت |

**بررسی:**

```bash
ip route
ip route get 8.8.8.8
traceroute 8.8.8.8
```

**نکتهٔ حیاتی:** **asymmetric routing** — بسته از یک مسیر می‌رود و از مسیر دیگر برمی‌گردد. فایروال stateful، بستهٔ برگشت را drop می‌کند. علامت: اتصال برقرار نمی‌شود، اما ping یک‌طرفه کار می‌کند.

**۵. Firewall — پنجمین**

| علامت | علت |
|---|---|
| **timeout** | drop |
| **refused** | reject (RST) |
| **بعضی پورت‌ها** | rule ناقص |
| **یک‌طرفه** | rule جهت‌دار |

**بررسی:**

```bash
# لینوکس
nft list ruleset
iptables -L -n -v
ufw status verbose
firewall-cmd --list-all

# ویندوز
Get-NetFirewallRule | Where-Object {$_.Enabled -eq 'True'}
Get-NetFirewallProfile
```

**نکته:** در ویندوز، **پروفایل** (Domain/Private/Public) مهم است. یک rule در Private فعال و در Public غیرفعال می‌تواند باعث رفتار متفاوت شود.

**جدول تشخیص ترکیبی:**

| علامت | اولین بررسی |
|---|---|
| **169.254.x.x** | DHCP، relay، کابل |
| **ping gateway ❌** | VLAN، IP/subnet، کابل |
| **ping gateway ✅، ping 8.8.8.8 ❌** | default route، ISP |
| **ping 8.8.8.8 ✅، ping name ❌** | DNS |
| **ping ✅، nc پورت ❌** | فایروال، سرویس |
| **nc ✅، curl ❌** | اپلیکیشن، TLS، proxy |
| **intermittent** | کابل، duplex، wireless، ARP |

**نکتهٔ حیاتی:** این جدول، **ترتیب** را می‌دهد. از بالا به پایین برو.

### مثال‌های عملی

**Lab Scenario 1 — سرور جدید به شبکه وصل نمی‌شود**

> **وظیفه:** ترتیب بررسی.
>
> **پاسخ:**
> ۱. **لایه ۱:** LED، کابل، پورت سوئیچ.
> ۲. **IP:** `ip a` — آیا IP گرفته؟ (169.254 = DHCP نرسید)
> ۳. **VLAN:** پورت سوئیچ در VLAN درست؟
> ۴. **Subnet:** IP و mask با gateway هم‌خوان؟
> ۵. **Route:** `ip route` — default gateway؟
> ۶. **ping gateway** → اگر نه، لایه ۲/۳.
>
> **درس:** از پایین به بالا، بدون پرش.

**Lab Scenario 2 — بعد از تغییر VLAN، سرویس قطع**

> **وضعیت:** تیم شبکه VLAN سرور را عوض کرد. سرور IP دارد اما کاربران دسترسی ندارند.
>
> **وظیفه:** سه احتمال.
>
> **پاسخ:**
> ۱. **فایروال:** rule بین VLAN قدیم و جدید وجود ندارد.
> ۲. **DNS:** رکورد A به IP قدیمی اشاره می‌کند.
> ۳. **ACL روی سوئیچ:** محدودیت جدید.
>
> **درس:** تغییر VLAN = تغییر مسیر و سیاست. فقط IP کافی نیست.

### مقایسه و الگوهای مختلف

**مقایسهٔ reject و drop:**

| معیار | Reject | Drop |
|---|---|---|
| پاسخ | RST/ICMP | هیچ |
| علامت کلاینت | connection refused | timeout |
| تشخیص | آسان | سخت |
| امنیت | کمتر (نشت اطلاعات) | بیشتر |

**مقایسهٔ static و dynamic routing:**

| معیار | Static | Dynamic |
|---|---|---|
| پیکربندی | دستی | پروتکل (OSPF، BGP) |
| مقیاس | کوچک | بزرگ |
| خطا | فراموشی | misconfiguration |
| مناسب | سرور | روتر |

**مقایسهٔ APIPA و static:**

| معیار | APIPA | Static |
|---|---|---|
| محدوده | 169.254/16 | دلخواه |
| معنی | DHCP نرسید | تنظیم دستی |
| عمل | تشخیص | — |

### اشکال شایع

**۱. پرش از لایه ۱.**
VLAN و DNS را چک می‌کنی، کابل قطع است.

**۲. نادیده گرفتن subnet.**
IP خارج از subnet gateway.

**۳. trunk allowed list.**
VLAN تعریف شده اما مجاز نیست.

**۴. asymmetric routing.**
ping یک‌طرفه، اتصال نه.

**۵. firewall profile در ویندوز.**
رفتار متفاوت بر اساس شبکه.

**۶. DNS cache.**
تغییر اعمال شده، cache قدیمی.

**۷. rogue DHCP.**
IP از منبع اشتباه.

**۸. MTU در VPN.**
پکت بزرگ گیر می‌کند.

### نکات مهم (Tips & Tricks)

- **جدول تشخیص** را حفظ کن.
- **از پایین به بالا** برو.
- **`ip route get`** مسیر واقعی.
- **`nft list ruleset`** قبل از حدس.
- **cache را flush کن** (`ipconfig /flushdns`، `resolvectl flush-caches`).
- **در آزمون:** کلیدواژهٔ «169.254» → **DHCP/APIPA**؛ «ping IP ok, name fails» → **DNS**.

### خلاصه فصل

- پنج اشتباه: **DNS، DHCP، VLAN، routing، firewall**.
- **169.254** = DHCP نرسید.
- **trunk allowed list** را چک کن.
- **asymmetric routing** = اتصال نه، ping یک‌طرفه.
- **firewall profile** در ویندوز.
- جدول تشخیص، ترتیب را می‌دهد.

---

## فصل ۹۲: ابزارهای تحلیل شبکه

### مقدمه

در فصل ۷۵ ابزارها را کلی دیدیم. این فصل، ابزارهای شبکه را **عمیق‌تر** و با مثال واقعی می‌آورد.

### مفاهیم اساسی

**دسته‌بندی:**

| دسته | ابزار | کاربرد |
|---|---|---|
| **اتصال** | ping، mtr، traceroute | دسترس‌پذیری، مسیر |
| **پورت** | nc، telnet، nmap | سرویس |
| **DNS** | dig، nslookup، host | resolve |
| **capture** | tcpdump، Wireshark | تحلیل بسته |
| **throughput** | iperf3 | پهنای باند |
| **پیکربندی** | ip، ss، ethtool | وضعیت |
| **لاگ** | journalctl، firewall log | رخداد |

**ping — گزینه‌های مفید:**

```bash
ping -c 4 host           # ۴ بسته
ping -i 0.2 host         # فاصله ۰.۲ ثانیه
ping -s 1472 -M do host  # تست MTU
ping -W 1 host           # timeout یک ثانیه
ping -f host             # flood (فقط root)
```

**نکته:** `-M do` یعنی «fragment نکن» — برای تست MTU ضروری است.

**mtr — نمونه:**

```bash
mtr -rwc 100 8.8.8.8
# -r گزارش، -w عرض، -c تعداد
```

**خروجی:**

```
HOST: srv01          Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- 10.0.10.1     0.0%   100    0.5   0.6   0.4   1.2   0.1
  2.|-- 10.0.0.1      0.0%   100    2.1   2.3   1.8   4.0   0.4
  3.|-- 203.0.113.9   1.0%   100   12.0  13.5  11.0  30.0   3.2
```

**نکته:** loss در hop ۳ که در hop ۴ ادامه ندارد، احتمالاً ICMP deprioritization است.

**nc (netcat) — تست پورت:**

```bash
nc -zv 10.0.10.21 443       # تست پورت
nc -zv 10.0.10.21 1-1000    # محدوده
nc -l 8080                  # گوش دادن (تست)
```

**نکته:** `-z` فقط اسکن، `-v` verbose. سریع‌ترین تست پورت.

**nmap — کشف:**

```bash
nmap -sn 10.0.10.0/24         # ping scan
nmap -p 22,80,443 10.0.10.21  # پورت مشخص
nmap -sV -p- 10.0.10.21       # نسخه و همه پورت
```

> ⚠️ **مرز قانونی:** اسکن شبکه فقط روی محدوده‌ای که **مجاز** هستی. اسکن شبکهٔ دیگران، تخلف است.

**dig — نمونه:**

```bash
dig example.com A
dig @10.0.10.5 example.com A +short
dig example.com MX
dig -x 10.0.10.21              # reverse
dig example.com AXFR           # zone transfer (اگر مجاز)
dig +trace example.com         # مسیر کامل
```

**نکته:** `+short` خروجی تمیز؛ `+trace` مسیر resolve از root.

**tcpdump — capture:**

```bash
tcpdump -i eth0 -nn
tcpdump -i eth0 -nn port 443
tcpdump -i eth0 -nn host 10.0.10.21
tcpdump -i eth0 -nn -w capture.pcap
tcpdump -i eth0 -nn -A port 80   # ASCII
```

**فیلترهای مفید:**

| فیلتر | معنی |
|---|---|
| `port 443` | پورت |
| `host 10.0.10.21` | میزبان |
| `net 10.0.10.0/24` | شبکه |
| `tcp[tcpflags] & tcp-syn != 0` | SYN |
| `icmp` | ICMP |

**نکته:** `-w` برای ذخیره و تحلیل در Wireshark. `-c 100` برای محدود کردن.

**Wireshark — فیلترهای نمایش:**

| فیلتر | معنی |
|---|---|
| `ip.addr == 10.0.10.21` | آدرس |
| `tcp.port == 443` | پورت |
| `http.request` | درخواست HTTP |
| `tcp.analysis.retransmission` | retransmit |
| `dns` | DNS |
| `tls.handshake.type == 1` | Client Hello |

**نکته:** فیلتر capture (`tcpdump`) و display (`Wireshark`) متفاوت‌اند. اول capture را محدود کن، بعد display.

**iperf3 — نمونه:**

```bash
iperf3 -s                          # سرور
iperf3 -c host -t 30               # TCP
iperf3 -c host -u -b 500M -t 30    # UDP
iperf3 -c host -P 4                # ۴ رشته موازی
```

**نکته:** `-P 4` برای تست ظرفیت کل (تک‌رشته ممکن است محدود باشد).

**ss — نمونه:**

```bash
ss -tulpn              # پورتهای گوشدهنده
ss -tan                # همه اتصالها
ss -ti                 # با جزئیات TCP (rtt، retrans)
ss -s                  # خلاصه
```

**ethtool:**

```bash
ethtool eth0           # speed، duplex، link
ethtool -S eth0        # آمار (errors، drops)
ethtool -i eth0        # driver، firmware
```

**نکته:** `ethtool -S` برای دیدن **CRC error** و **drop** — نشانهٔ کابل یا duplex.

### مثال‌های عملی

**Lab Scenario 1 — اتصال برقرار نمی‌شود**

> **وظیفه:** با ابزارها، گام‌به‌گام.
>
> **پاسخ:**
> ۱. `ping -c 4 10.0.10.21` — دسترس‌پذیری
> ۲. `mtr -rwc 20 10.0.10.21` — مسیر و loss
> ۳. `nc -zv 10.0.10.21 443` — پورت
> ۴. `ss -tulpn | grep 443` روی سرور — سرویس گوش می‌دهد؟
> ۵. `tcpdump -i eth0 -nn host 10.0.10.21 and port 443` — بسته می‌رسد؟
> ۶. لاگ فایروال — drop؟
>
> **درس:** هر ابزار، یک لایه را تأیید یا رد می‌کند.

**Lab Scenario 2 — retransmit بالا**

> **وظیفه:** تشخیص.
>
> **پاسخ:**
> ۱. `ss -ti` — `retrans` و `rtt`.
> ۲. `ethtool -S eth0` — CRC error، drop؟
> ۳. `mtr` — loss در کدام hop؟
> ۴. `tcpdump` — retransmit الگو؟
> ۵. **علت‌ها:** کابل، duplex، ازدحام، MTU.
>
> **درس:** retransmit، علامت loss است — علت را جدا پیدا کن.

### مقایسه و الگوهای مختلف

**مقایسهٔ tcpdump و Wireshark:**

| معیار | tcpdump | Wireshark |
|---|---|---|
| رابط | CLI | GUI |
| مناسب | سرور، اسکریپت | تحلیل |
| حجم | کم | زیاد |
| توصیه | capture | analyze |

**مقایسهٔ nc و nmap:**

| معیار | nc | nmap |
|---|---|---|
| هدف | تست یک پورت | کشف چند پورت |
| سرعت | سریع | متوسط |
| مناسب | عیب‌یابی | audit |

**مقایسهٔ dig و nslookup:**

| معیار | dig | nslookup |
|---|---|---|
| جزئیات | کامل | محدود |
| اسکریپت | ✅ | متوسط |
| مناسب | عیب‌یابی | سریع |

### اشکال شایع

**۱. capture بدون فیلتر.**
فایل گیگابایتی.

**۲. تحلیل capture بدون context.**
بسته‌ها بدون سناریو، بی‌معنا.

**۳. اسکن بدون مجوز.**
تخلف قانونی.

**۴. اتکا به یک ابزار.**
هر ابزار، دید محدود.

**۵. نادیده گرفتن `ethtool -S`.**
CRC error، drop.

**۶. تست با یک رشته.**
throughput کم نشان داده می‌شود.

**۷. فراموش کردن `-nn`.**
resolve کند و گمراه‌کننده.

**۸. capture روی interface اشتباه.**
هیچ بسته‌ای دیده نمی‌شود.

### نکات مهم (Tips & Tricks)

- **`nc -zv`** سریع‌ترین تست پورت.
- **`mtr -rwc`** برای گزارش.
- **`tcpdump -nn`** بدون resolve.
- **`ss -ti`** برای retransmit.
- **`ethtool -S`** برای خطای NIC.
- **در آزمون:** کلیدواژهٔ «capture traffic» → **tcpdump/Wireshark**.

### خلاصه فصل

- ابزارها: **ping، mtr، nc، nmap، dig، tcpdump، Wireshark، iperf3، ss، ethtool**.
- **`-nn`** در tcpdump برای سرعت.
- **`ss -ti`** retransmit و RTT.
- **`ethtool -S`** خطای NIC.
- **capture با فیلتر**، بعد تحلیل.
- هر ابزار، یک لایه.

---

## فصل ۹۳: نفوذ شبکه و پاسخ به رخداد

### مقدمه

گاهی «مشکل شبکه» یک خرابی نیست؛ یک **حمله** است. و پاسخ به آن، رویهٔ متفاوتی می‌خواهد: نه رفع، بلکه **مهار، حفظ شواهد، و ریشه‌یابی**.

این فصل دربارهٔ تشخیص و پاسخ است.

> ⚠️ **مرز قانونی:** این فصل برای **دفاع** نوشته شده. تست نفوذ، رصد ترافیک دیگران، و دسترسی به سیستم بدون مجوز، در بسیاری از کشورها تخلف است. هر اقدام تهاجمی فقط با **مجوز کتبی** و در محدودهٔ مشخص.

### مفاهیم اساسی

**علائم نفوذ شبکه:**

| علامت | احتمال |
|---|---|
| **ترافیک خروجی زیاد به IP ناشناس** | exfiltration، C2 |
| **اتصال به پورت غیرمعمول** | backdoor، miner |
| **پورت باز ناشناس** | backdoor |
| **اسکن پورت از داخل** | آلودگی، lateral movement |
| **DNS query به دامنهٔ ناشناس** | DNS tunneling |
| **حجم بالای auth failure** | brute force |
| **ترافیک در ساعت غیرعادی** | automated |
| **گواهی TLS ناشناس** | MITM |

**نکتهٔ حیاتی:** **ترافیک خروجی** مهم‌تر از ورودی است. بیشتر نشت‌ها با اتصال **از داخل به بیرون** رخ می‌دهد.

**چهار فاز پاسخ به رخداد (NIST SP 800-61):**

```
۱. Preparation
۲. Detection & Analysis
۳. Containment, Eradication & Recovery
۴. Post-Incident Activity
```

**نکته:** Server+ سطح «پاسخ اولیه» را می‌سنجد، نه تحقیق کامل فارنزیک.

**مراحل عملی — پاسخ اولیه:**

```
۱. تأیید (آیا واقعاً رخداد است؟)
۲. مهار (isolate، نه خاموشی)
۳. حفظ شواهد (لاگ، memory، دیسک)
۴. اطلاعرسانی (تیم امنیت، مدیریت)
۵. ریشهیابی (چطور وارد شد؟)
۶. پاکسازی (malware، حساب)
۷. بازیابی (از backup تمیز)
۸. درسآموخته
```

**نکتهٔ حیاتی:** **خاموش نکن.** خاموش کردن، شواهد حافظه (RAM) را نابود می‌کند. به‌جای آن، **شبکه را ایزوله کن**.

**ایزوله کردن — روش‌ها:**

| روش | اثر | شواهد |
|---|---|---|
| **قطع کابل** | کامل | حفظ |
| **غیرفعال کردن پورت سوئیچ** | کامل | حفظ |
| **VLAN قرنطینه** | شبکهٔ محدود | حفظ + امکان تحلیل |
| **خاموش کردن** | کامل | **از دست رفتن RAM** |

**نکته:** VLAN قرنطینه بهترین است — سیستم روشن می‌ماند، تحلیل ممکن است، اما به شبکهٔ تولیدی دسترسی ندارد.

**حفظ شواهد — ترتیب:**

```
۱. RAM (فرارترین) — memory dump
۲. اتصالهای شبکه — ss، netstat
۳. فرآیندها — ps
۴. لاگها — copy
۵. دیسک — image
```

**نکتهٔ حیاتی:** ترتیب **فراریت** (volatility order) مهم است. RAM اول، چون با خاموشی از بین می‌رود.

**زنجیرهٔ custody:** هر شاهد، با تاریخ، فرد، امضا (فصل ۷۲).

**تشخیص — منابع:**

| منبع | چه می‌دهد |
|---|---|
| **Firewall log** | اتصال‌های block/allow |
| **IDS/IPS** | الگوی حمله |
| **SIEM** | همبستگی |
| **NetFlow** | جریان ترافیک |
| **DNS log** | query مشکوک |
| **Proxy log** | دسترسی وب |
| **EDR** | رفتار endpoint |
| **Auth log** | ورود |

**نکته:** **NetFlow** (یا sFlow/IPFIX) جریان ترافیک را بدون capture کامل ثبت می‌کند — برای تشخیص exfiltration عالی است.

**الگوهای حمله رایج:**

| حمله | علامت |
|---|---|
| **Port scan** | اتصال به پورت‌های متعدد از یک منبع |
| **Brute force** | auth failure مکرر |
| **DDoS** | ترافیک ورودی انبوه |
| **MITM** | ARP spoofing، گواهی ناشناس |
| **DNS tunneling** | query طولانی/مکرر به یک دامنه |
| **Data exfiltration** | خروجی زیاد به IP جدید |
| **Lateral movement** | SMB/RDP بین سرورها |

**نکته:** **lateral movement** در شبکهٔ تخت (flat) راحت است. Segmentation (فصل ۳۱) آن را سخت می‌کند.

**ARP spoofing — تشخیص:**

```bash
ip neigh
arp -a
# MAC تکراری برای دو IP = مشکوک
```

**نکته:** ابزار: `arpwatch`. دفاع: Dynamic ARP Inspection روی سوئیچ.

**پورت اسکن از داخل — تشخیص:**

```bash
ss -tan state syn-sent
# اتصالهای نیمهباز زیاد
```

**نکته:** یک سرور که به صدها IP روی پورت ۴۴۵ تلاش می‌کند، احتمالاً آلوده است (worm، ransomware).

**پس از مهار — پاکسازی:**

| مرحله | اقدام |
|---|---|
| **حساب‌ها** | تغییر رمز، حذف حساب ناشناس |
| **malware** | حذف با EDR، یا rebuild |
| **پیکربندی** | بازگرداندن از baseline |
| **آسیب‌پذیری** | patch |
| **بازگشت** | از backup **قبل از** آلودگی |

**نکتهٔ حیاتی:** **backup قبل از آلودگی** — اگر backup بعد از آلودگی گرفته شده، malware هم برمی‌گردد.

**گزارش به نهاد ناظر:**

| مقرره | بازه |
|---|---|
| **GDPR** | ۷۲ ساعت |
| **HIPAA** | ۶۰ روز |
| **PCI DSS** | فوری (به acquirer) |
| **NIS2** | ۲۴ ساعت (هشدار اولیه) |

> ⚠️ **مرز قانونی:** الزامات گزارش‌دهی، تابع مقررات کشور و صنعت است. با واحد حقوقی مشورت کن.

### مثال‌های عملی

**Lab Scenario 1 — ترافیک خروجی مشکوک**

> **وضعیت:** مانیتورینگ نشان می‌دهد سرور هر شب ساعت ۳، ۲۰GB به یک IP در کشور دیگر می‌فرستد.
>
> **وظیفه:** مراحل.
>
> **پاسخ:**
> ۱. **تأیید:** `ss -tnp`، NetFlow، firewall log — کدام پروسه؟
> ۲. **مهار:** VLAN قرنطینه (نه خاموشی).
> ۳. **شواهد:** memory dump، `ps`، `ss`، لاگ.
> ۴. **اطلاع:** تیم امنیت، مدیریت.
> ۵. **ریشه:** چطور وارد شد؟ (وب، SSH، حساب لو رفته)
> ۶. **پاکسازی:** rebuild یا حذف malware.
> ۷. **بازیابی:** از backup قبل از آلودگی.
>
> **درس:** exfiltration، اغلب در ساعت کم‌ترافیک رخ می‌دهد.

**Lab Scenario 2 — ransomware روی یک سرور**

> **وظیفه:** پنج اقدام فوری.
>
> **پاسخ:**
> ۱. **ایزولهٔ شبکه** (VLAN قرنطینه یا قطع پورت).
> ۲. **غیرفعال کردن حساب‌های درگیر.**
> ۳. **قطع دسترسی به share** از سایر سرورها.
> ۴. **ارزیابی دامنه:** کدام سرورها آلوده‌اند؟
> ۵. **اطلاع فوری** به تیم امنیت و مدیریت.
>
> **نکته:** **خاموش نکن** — کلید رمزنگاری ممکن است در RAM باشد.
>
> **درس:** سرعت مهار، گسترش را محدود می‌کند.

### مقایسه و الگوهای مختلف

**مقایسهٔ IDS و IPS:**

| معیار | IDS | IPS |
|---|---|---|
| رفتار | تشخیص و هشدار | تشخیص و مسدود |
| محل | out-of-band | inline |
| ریسک | کم | false positive = قطع سرویس |

**مقایسهٔ NetFlow و packet capture:**

| معیار | NetFlow | Capture |
|---|---|---|
| حجم | کم | زیاد |
| جزئیات | متادیتا | محتوا |
| مناسب | روند، exfiltration | تحلیل دقیق |

**مقایسهٔ ایزوله و خاموشی:**

| معیار | ایزوله | خاموشی |
|---|---|---|
| شواهد RAM | حفظ | از دست |
| تحلیل زنده | ✅ | ❌ |
| گسترش | متوقف | متوقف |
| توصیه | ✅ | فقط اگر ضروری |

### اشکال شایع

**۱. خاموش کردن فوری.**
شواهد RAM از دست می‌رود.

**۲. پاک کردن malware قبل از تحلیل.**
ریشه پیدا نمی‌شود.

**۳. بازیابی از backup آلوده.**
malware برمی‌گردد.

**۴. بدون گزارش به مدیریت.**
مهار ناقص، گسترش.

**۵. تغییر رمز بدون حذف مکانیزم persistence.**
مهاجم دوباره وارد می‌شود.

**۶. نادیده گرفتن ترافیک خروجی.**
فقط ورودی بررسی می‌شود.

**۷. بدون segmentation.**
lateral movement راحت.

**۸. بدون زنجیرهٔ custody.**
شواهد بی‌اعتبار.

### نکات مهم (Tips & Tricks)

- **ایزوله کن، خاموش نکن.**
- **ترتیب فراریت:** RAM → network → process → log → disk.
- **NetFlow** برای exfiltration.
- **VLAN قرنطینه** بهترین مهار.
- **backup قبل از آلودگی** برای بازیابی.
- **در آزمون:** کلیدواژهٔ «suspected compromise» → **isolate from network, preserve evidence**.

### خلاصه فصل

- علائم: **ترافیک خروجی، پورت ناشناس، auth failure، DNS مشکوک**.
- پاسخ: **تأیید → مهار → شواهد → اطلاع → ریشه → پاکسازی → بازیابی**.
- **ایزوله، نه خاموشی** (شواهد RAM).
- **ترتیب فراریت** شواهد.
- **backup قبل از آلودگی**.
- Segmentation، lateral movement را سخت می‌کند.

---

**بعدی:** [فایل ۱۹: عیب‌یابی بازیابی و تحلیل ریشه‌ای](19-Recovery-Troubleshooting-and-RCA.md)

---

## منابع و مراجع

- اهداف رسمی Server+ (V5)، دامنهٔ ۴: Troubleshooting
- NIST SP 800-61 Rev.2 — Computer Security Incident Handling Guide
- NIST SP 800-41 Rev.1 — Guidelines on Firewalls and Firewall Policy
- RFC 792 — ICMP
- `man tcpdump`، `man ss`، `man ethtool`، `man mtr`
- Wireshark Display Filter Reference
- MITRE ATT&CK — Lateral Movement، Exfiltration
- <https://www.comptia.org/en-us/certifications/server/>
