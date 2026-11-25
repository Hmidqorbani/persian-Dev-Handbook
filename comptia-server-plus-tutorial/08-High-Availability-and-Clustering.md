# دسترس‌پذیری بالا و کلاسترینگ

**مدت یادگیری**: ۶ ساعت
**سطح سختی**: متوسط تا پیشرفته
**پیش‌نیاز**: فصل ۳۳ تا ۳۷ — مدیریت و پایش سرور
**دامنهٔ آزمون**: Server Administration (۳۰٪)

---

## فهرست محتوا

- [فصل ۳۸: دسترس‌پذیری، SLA و نقطهٔ شکست واحد](#ch-38)
- [فصل ۳۹: کلاسترینگ و failover](#ch-39)
- [فصل ۴۰: متعادل‌سازی بار](#ch-40)
- [فصل ۴۱: آزمون failover و ظرفیت](#ch-41)

---

## فصل ۳۸: دسترس‌پذیری، SLA و نقطهٔ شکست واحد

### مقدمه

«سرور نباید بخوابد» یک آرزو است، نه یک طرح. طرح این است: **چقدر دسترس‌پذیری می‌خواهی، چقدر می‌ارزد، و کدام اجزا باید افزونه باشند.**

این فصل، زبان مشترک دسترس‌پذیری را می‌سازد: nines، SLA، SLO، SPOF، MTTR و MTBF. بدون این زبان، نمی‌توانی دربارهٔ افزونگی تصمیم بگیری.

### مفاهیم اساسی

**دسترس‌پذیری با «نُه» (nines):**

| دسترس‌پذیری | downtime سالانه | downtime ماهانه | downtime هفتگی |
|---|---|---|---|
| ۹۰٪ (one nine) | ۳۶.۵ روز | ۳ روز | ۱۶.۸ ساعت |
| ۹۹٪ (two nines) | ۳.۶۵ روز | ۷.۲ ساعت | ۱.۶۸ ساعت |
| ۹۹.۹٪ (three nines) | ۸.۷۶ ساعت | ۴۳.۸ دقیقه | ۱۰.۱ دقیقه |
| ۹۹.۹۵٪ | ۴.۳۸ ساعت | ۲۱.۹ دقیقه | ۵.۰۴ دقیقه |
| ۹۹.۹۹٪ (four nines) | ۵۲.۶ دقیقه | ۴.۳۸ دقیقه | ۶۰.۵ ثانیه |
| ۹۹.۹۹۹٪ (five nines) | ۵.۲۶ دقیقه | ۲۶.۳ ثانیه | ۶.۰۵ ثانیه |
| ۹۹.۹۹۹۹٪ | ۳۱.۵ ثانیه | ۲.۶ ثانیه | ۰.۶ ثانیه |

**فرمول:** `دسترس‌پذیری = MTBF ÷ (MTBF + MTTR) × ۱۰۰`

که در آن:
- **MTBF** (Mean Time Between Failures) — میانگین زمان بین خرابی‌ها.
- **MTTR** (Mean Time To Repair) — میانگین زمان تعمیر.

**نکتهٔ کلیدی:** دو راه برای افزایش دسترس‌پذیری وجود دارد: **افزایش MTBF** (قطعات بهتر، نگهداشت) یا **کاهش MTTR** (افزونگی، بازیابی سریع). دومی معمولاً ارزان‌تر و مؤثرتر است.

**مثال:** MTBF = ۱۰٬۰۰۰ ساعت، MTTR = ۱۰ ساعت:
```
۱۰٬۰۰۰ ÷ ۱۰٬۰۱۰ = ۹۹.۹٪
```
اگر MTTR را به ۱ ساعت کاهش دهی:
```
۱۰٬۰۰۰ ÷ ۱۰٬۰۰۱ = ۹۹.۹۹٪
```
یک صفر اضافه، فقط با سریع‌تر کردن بازیابی.

**SLA، SLO، SLI:**

| اصطلاح | معنی | مثال |
|---|---|---|
| **SLI** (Indicator) | اندازه‌گیری واقعی | ۹۹.۹۲٪ uptime ماه گذشته |
| **SLO** (Objective) | هدف داخلی | ۹۹.۹٪ در ماه |
| **SLA** (Agreement) | تعهد قراردادی با جریمه | ۹۹.۵٪، وگرنه اعتبار |

**نکتهٔ مهم:** SLO باید **سخت‌گیرانه‌تر** از SLA باشد. اگر SLA ۹۹.۵٪ است، SLO را ۹۹.۹٪ بگذار تا حاشیه داشته باشی.

**SPOF (Single Point of Failure):** جزئی که خرابی‌اش کل سیستم را می‌خواباند.

**فهرست SPOFهای رایج:**

| لایه | SPOF | افزونگی |
|---|---|---|
| برق | یک PSU | PSU دوتایی + مسیر A/B |
| شبکه | یک سوئیچ | دو سوئیچ + teaming |
| ذخیره‌سازی | یک کنترلر | دو کنترلر |
| سرور | یک سرور | کلاستر/LB |
| دیتابیس | یک نمونه | replica/cluster |
| DNS | یک سرور | دو سرور |
| اینترنت | یک لینک | دو ISP |
| اتاق | یک دیتاسنتر | سایت دوم |
| انسان | یک متخصص | مستندسازی + آموزش |

**نکتهٔ حیاتی:** افزونگی در یک لایه، SPOF لایهٔ دیگر را جبران نمی‌کند. سرور با دو PSU که به **یک** PDU وصل است، هنوز SPOF دارد.

**انواع افزونگی:**

| نوع | توضیح | بازیابی |
|---|---|---|
| **Active/Active** | هر دو فعال، بار تقسیم | فوری |
| **Active/Passive** | یکی فعال، یکی standby | چند ثانیه تا دقیقه |
| **N+1** | یک واحد اضافه | فوری |
| **N+M** | M واحد اضافه | فوری |
| **2N** | دو برابر ظرفیت | فوری |

**مفاهیم مرتبط:**

- **Failover** — انتقال خودکار به نسخهٔ پشتیبان.
- **Failback** — بازگشت به نسخهٔ اصلی پس از رفع مشکل.
- **Split-brain** — دو نود هر دو فکر می‌کنند فعال‌اند (خطر خرابی داده).
- **Quorum** — حداقل رأی برای تصمیم‌گیری در کلاستر.
- **Fencing/STONITH** — «Shoot The Other Node In The Head» — خاموش کردن نود مشکل‌دار قبل از failover.

**نکتهٔ مهم:** بدون fencing، split-brain می‌تواند دو نود را هم‌زمان به یک دیسک بنویسد و داده را نابود کند. **fencing، بخشی از HA است، نه لوکس.**

### مثال‌های عملی

**Lab Scenario 1 — محاسبهٔ دسترس‌پذیری زنجیره**

> **وضعیت:** یک سرویس از سه جزء تشکیل شده: سرور (۹۹.۹٪)، شبکه (۹۹.۹۹٪)، برق (۹۹.۹۵٪). دسترس‌پذیری کل؟
>
> **پاسخ:**
> - اجزای **سری** (اگر یکی بخوابد، همه می‌خوابد): دسترس‌پذیری‌ها **ضرب** می‌شوند.
> ```
> ۰.۹۹۹ × ۰.۹۹۹۹ × ۰.۹۹۹۵ = ۰.۹۹۸۴ ≈ ۹۹.۸۴٪
> ```
> - یعنی **حدود ۱۴ ساعت downtime در سال** — کمتر از ضعیف‌ترین جزء (۹۹.۹٪ = ۸.۷۶ ساعت).
>
> **درس:** زنجیره، همیشه از ضعیف‌ترین حلقه ضعیف‌تر است.

**Lab Scenario 2 — افزونگی، دسترس‌پذیری را چقدر بالا می‌برد؟**

> **وضعیت:** همان سه جزء، اما سرور را دو نمونهٔ موازی (active/active) می‌کنی. هر سرور ۹۹.۹٪.
>
> **وظیفه:** دسترس‌پذیری سرور؟
>
> **پاسخ:**
> - اجزای **موازی**: احتمال اینکه **هر دو** بخوابند = (۱ − ۰.۹۹۹)² = ۰.۰۰۰۰۰۱
> - دسترس‌پذیری = ۱ − ۰.۰۰۰۰۰۱ = **۹۹.۹۹۹۹٪**
> - اما کل زنجیره: ۰.۹۹۹۹۹۹ × ۰.۹۹۹۹ × ۰.۹۹۹۵ ≈ **۹۹.۸۹٪**
>
> **درس:** افزونگی سرور، ضعف شبکه و برق را جبران نمی‌کند. باید هر لایه را جدا حل کنی.

### مقایسه و الگوهای مختلف

**مقایسهٔ Active/Active و Active/Passive:**

| معیار | Active/Active | Active/Passive |
|---|---|---|
| استفاده از منابع | ۱۰۰٪ | ۵۰٪ (standby بیکار) |
| زمان failover | صفر | ثانیه تا دقیقه |
| پیچیدگی | بالا (state sharing) | متوسط |
| مناسب | وب، stateless | دیتابیس، stateful |

**مقایسهٔ HA و DR:**

| معیار | HA | DR |
|---|---|---|
| هدف | جلوگیری از قطعی | بازیابی پس از فاجعه |
| محدوده | یک جزء/سرویس | کل سایت |
| زمان | ثانیه | دقیقه تا ساعت |
| مثال | کلاستر، LB | سایت دوم، backup offsite |

**نکته:** HA و DR مکمل هستند، نه جایگزین. HA برای خرابی دیسک است؛ DR برای آتش‌سوزی دیتاسنتر.

**مقایسهٔ رویکردهای کاهش MTTR:**

| رویکرد | اثر | هزینه |
|---|---|---|
| **افزونگی (HA)** | MTTR نزدیک صفر | بالا |
| **Runbook** | MTTR کم | کم |
| **پایش خوب** | تشخیص سریع | متوسط |
| **قطعه یدکی** | تعمیر سریع | کم |
| **قرارداد پشتیبانی ۴ ساعته** | تعمیر سریع | متوسط |

**توصیه:** قبل از خرید HA گران، runbook و پایش را درست کن. بسیاری از سازمان‌ها با MTTR بهتر، به همان دسترس‌پذیری می‌رسند با هزینهٔ کمتر.

### اشکال شایع

**۱. SPOF پنهان.**
دو PSU به یک PDU، دو سوئیچ به یک برق، دو لینک از یک ISP.

**۲. HA بدون fencing.**
split-brain و خرابی داده.

**۳. HA بدون تست.**
کلاستری که هرگز failover نشده، کلاستر نیست؛ یک فرض است.

**۴. افزونگی در لایهٔ اشتباه.**
RAID 10 روی سروری که یک PSU دارد. دیسک محافظت شده، برق نه.

**۵. SLA بلندپروازانه بدون بودجه.**
۹۹.۹۹٪ با یک سرور و یک لینک، غیرممکن است.

**۶. فراموش کردن لایهٔ انسانی.**
یک متخصص که همه چیز را می‌داند و مستند نکرده = SPOF.

**۷. Active/Passive بدون تست standby.**
standby بعد از شش ماه، آپدیت نشده و کار نمی‌کند.

### نکات مهم (Tips & Tricks)

- **جدول nines را حفظ کن** — ۹۹.۹٪ = ۸.۷۶ ساعت در سال.
- **SPOF را روی کاغذ پیدا کن:** هر جزء را بکش، ببین چه می‌شود.
- **MTTR را اندازه بگیر، نه حدس بزن.** از رخدادهای گذشته.
- **fencing را جدی بگیر** — STONITH، IPMI reset، یا network fencing.
- **standby را ماهانه تست کن** — آپدیت و failover آزمایشی.
- **در آزمون:** کلیدواژهٔ «single point of failure» → **any component whose failure stops the service**.

### خلاصه فصل

- دسترس‌پذیری با **nines**: ۹۹.۹٪ = ۸.۷۶ ساعت/سال.
- `دسترس‌پذیری = MTBF ÷ (MTBF + MTTR)`.
- **SLI < SLO < SLA** (سخت‌گیرانه‌تر به ترتیب).
- **SPOF** در هر لایه باید شناسایی و حذف شود.
- **fencing** برای جلوگیری از split-brain الزامی است.
- HA و DR مکمل‌اند، نه جایگزین.

---

## فصل ۳۹: کلاسترینگ و failover

### مقدمه

کلاستر، دو یا چند سرور است که مثل یک سرویس رفتار می‌کنند. اما «کلاستر» یک کلمهٔ واحد برای چند مفهوم متفاوت است: failover cluster، load-balancing cluster، storage cluster، database cluster.

این فصل دربارهٔ **failover clustering** است: وقتی یک نود می‌میرد، نود دیگر کار را برمی‌دارد.

### مفاهیم اساسی

**انواع کلاستر:**

| نوع | هدف | مثال |
|---|---|---|
| **Failover (HA)** | تداوم سرویس | Windows Failover Cluster، Pacemaker |
| **Load balancing** | توزیع بار | NLB، HAProxy، LVS |
| **Storage** | ذخیره‌سازی مشترک | Ceph، GlusterFS |
| **Database** | دسترس‌پذیری داده | AlwaysOn، Galera، Patroni |
| **Compute/HPC** | پردازش موازی | Beowulf، Slurm |

**اجزای یک failover cluster:**

```
[Node 1] ──┐
           ├── [Heartbeat] ── [Shared Storage / Replication]
[Node 2] ──┘         │
                     └── [Quorum / Witness]
```

- **Node** — سرور عضو کلاستر.
- **Heartbeat** — پیام «زنده‌ام» بین نودها.
- **Quorum** — مکانیزم تصمیم‌گیری وقتی ارتباط مشکل دارد.
- **Shared storage** — دیسک مشترک (SAN) یا replication.
- **Virtual IP / Service** — آدرسی که با failover جابه‌جا می‌شود.
- **Resource group** — مجموعهٔ منابعی که با هم جابه‌جا می‌شوند.

**Quorum — چرا حیاتی است؟**

اگر ارتباط بین نودها قطع شود، هر نود باید تصمیم بگیرد: «آیا نود دیگر مرده یا فقط شبکه قطع است؟»

| مدل | توضیح |
|---|---|
| **Node majority** | رأی اکثریت نودها |
| **Node + disk witness** | اکثریت + یک دیسک شاهد |
| **Node + file share witness** | اکثریت + یک share |
| **Cloud witness** | اکثریت + یک blob ابری |
| **No majority (disk only)** | برای کلاستر ۲ نودی با دیسک مشترک |

**نکتهٔ مهم:** کلاستر **۲ نودی** با node majority مشکل دارد (۱ در برابر ۱ = بن‌بست). راه‌حل: **witness** (disk/file/cloud) که رأی سوم می‌دهد.

**Fencing / STONITH:**

قبل از failover، باید مطمئن شوی نود قدیمی **واقعاً** متوقف است. روش‌ها:

- **Power fencing** — خاموش کردن با IPMI/PDU.
- **Network fencing** — قطع شبکه.
- **Storage fencing** — قطع دسترسی به دیسک.
- **SBD (STONITH Block Device)** — دیسک مشترک برای هماهنگی.

**بدون fencing:** نود قدیمی ممکن است هنوز به دیسک بنویسد و نود جدید هم بنویسد → خرابی داده.

**سناریوهای failover:**

| سناریو | رفتار |
|---|---|
| **Node failure** | نود دیگر منابع را برمی‌دارد |
| **Network failure** | quorum تصمیم می‌گیرد |
| **Storage failure** | بستگی به افزونگی storage |
| **Service crash** | resource monitor ری‌استارت می‌کند |
| **Planned maintenance** | live migration یا drain |

**Windows Failover Cluster — مراحل:**

```powershell
# نصب feature
Install-WindowsFeature -Name Failover-Clustering -IncludeManagementTools

# تست پیش‌نیازها
Test-Cluster -Node "node1","node2"

# ساخت کلاستر
New-Cluster -Name "CL01" -Node "node1","node2" -StaticAddress "10.0.10.100"

# تنظیم quorum
Set-ClusterQuorum -NodeAndFileShareWitness "\\fs01\witness"

# افزودن نقش
Add-ClusterFileServerRole -Name "FS01" -Storage "Cluster Disk 1" -StaticAddress "10.0.10.101"

# تست failover
Move-ClusterGroup -Name "FS01" -Node "node2"
```

**Pacemaker/Corosync (لینوکس) — نمونهٔ ساده:**

```bash
# نصب
dnf install pacemaker corosync pcs

# احراز هویت
pcs host auth node1 node2 -u hacluster -p <password>

# ساخت کلاستر
pcs cluster setup mycluster node1 node2 --start

# فعال‌سازی
pcs cluster enable --all

# منابع: VIP و سرویس
pcs resource create VIP ocf:heartbeat:IPaddr2 ip=10.0.10.100 cidr_netmask=24
pcs resource create WebSite systemd:nginx

# گروه‌بندی (با هم جابه‌جا شوند)
pcs resource group add WebGroup VIP WebSite

# fencing
pcs stonith create fence1 fence_ipmilan \
    pcmk_host_list="node1" ip=10.0.99.11 login=admin passwd=<pass> lanplus=1

# وضعیت
pcs status
```

**نکتهٔ مهم:** `pcs resource group add` باعث می‌شود VIP و سرویس با هم جابه‌جا شوند. بدون گروه، ممکن است VIP روی یک نود و سرویس روی نود دیگر باشد.

**AlwaysOn Availability Group (SQL Server):**

- **AG** — گروهی از دیتابیس‌ها که به‌طور خودکار failover می‌شوند.
- **Listener** — یک نام/IP مجازی که کلاینت به آن وصل می‌شود.
- **Sync commit** — بدون از دست رفتن داده، اما latency بیشتر.
- **Async commit** — با احتمال از دست رفتن داده، اما سریع.

**نکته:** برای DR، معمولاً async؛ برای HA، sync. ترکیب هر دو (یکی sync محلی، یکی async دور) رایج است.

**PostgreSQL HA:**

| ابزار | کار |
|---|---|
| **Patroni** | مدیریت خودکار failover |
| **pg_auto_failover** | ساده‌تر |
| **repmgr** | ابزار قدیمی‌تر |
| **Pgpool-II** | LB + failover |

**نکته:** PostgreSQL streaming replication + Patroni + etcd/Consul = یک HA مدرن.

### مثال‌های عملی

**Lab Scenario 1 — طراحی کلاستر دو نودی**

> **وضعیت:** دو سرور فیزیکی، یک SAN، نیاز به HA برای فایل سرور.
>
> **وظیفه:** طرح کلاستر.
>
> **پاسخ:**
> ۱. **نودها:** node1، node2 — هر کدام ۲ NIC (heartbeat جدا).
> ۲. **شبکهٔ heartbeat:** VLAN جدا یا کابل مستقیم.
> ۳. **Shared storage:** LUN از SAN، متصل به هر دو نود با MPIO.
> ۴. **Quorum:** **file share witness** (چون ۲ نود داریم و بدون witness بن‌بست می‌شود).
> ۵. **Fencing:** IPMI برای هر نود.
> ۶. **Resource:** Cluster Disk + File Share Role + Client Access Point (VIP).
> ۷. **تست:** failover دستی، قطع کابل، خاموشی نود.
>
> **درس:** در ۲ نود، witness الزامی است؛ در غیر این صورت split-brain.

**Lab Scenario 2 — failover رخ داد اما کلاینت‌ها وصل نشدند**

> **وضعیت:** کلاستر فایل failover شد، سرویس روی node2 بالاست، اما کاربران به share دسترسی ندارند.
>
> **وظیفه:** عیب‌یابی.
>
> **پاسخ:**
> ۱. **VIP:** آیا VIP روی node2 بالا آمده؟ `Get-ClusterResource` یا `ip a`.
> ۲. **DNS:** آیا رکورد DNS به VIP اشاره می‌کند؟ (اگر A record باشد، درست است)
> ۳. **ARP cache:** کلاینت‌ها MAC قدیمی را cache کرده‌اند. `arp -d` یا انتظار.
> ۴. **Share permissions:** آیا share روی node2 با همان مجوزهاست؟
> ۵. **SMB signing/encryption:** تنظیمات یکسان است؟
>
> **درس:** failover موفق، یعنی **سرویس از دید کلاینت** کار کند — نه فقط resource بالاتر بیاید.

### مقایسه و الگوهای مختلف

**مقایسهٔ ابزارهای HA:**

| ابزار | پلتفرم | مناسب |
|---|---|---|
| **Windows Failover Cluster** | ویندوز | فایل، SQL، Hyper-V |
| **Pacemaker/Corosync** | لینوکس | سرویس‌های عمومی |
| **Keepalived** | لینوکس | VIP ساده با VRRP |
| **Patroni** | PostgreSQL | دیتابیس |
| **Kubernetes** | container | میکروسرویس |

**مقایسهٔ Keepalived و Pacemaker:**

| معیار | Keepalived | Pacemaker |
|---|---|---|
| پیچیدگی | کم | بالا |
| VRRP | ✅ | با resource agent |
| مدیریت منبع چندگانه | محدود | ✅ |
| مناسب | VIP ساده | کلاستر کامل |

**مقایسهٔ sync و async replication:**

| معیار | Sync | Async |
|---|---|---|
| از دست رفتن داده | صفر | ممکن |
| Latency | بالا (وابسته به فاصله) | کم |
| مناسب | HA محلی | DR دور |

**مقایسهٔ shared storage و replicated storage:**

| معیار | Shared (SAN) | Replicated (vSAN/Ceph) |
|---|---|---|
| هزینه | بالا | متوسط |
| پیچیدگی | متوسط | بالا |
| SPOF | SAN | ندارد (توزیع‌شده) |
| مناسب | سنتی | مدرن |

### اشکال شایع

**۱. کلاستر بدون fencing.**
split-brain و خرابی داده.

**۲. کلاستر ۲ نودی بدون witness.**
بن‌بست quorum.

**۳. heartbeat روی همان شبکهٔ داده.**
اگر شبکهٔ داده شلوغ یا قطع شود، heartbeat از دست می‌رود و failover الکی رخ می‌دهد.

**۴. failover بدون تست.**
کلاستری که تست نشده، در بحران شکست می‌خورد.

**۵. VIP بدون به‌روزرسانی DNS.**
کلاینت‌ها به آدرس قدیمی می‌روند.

**۶. standby آپدیت‌نشده.**
نود passive شش ماه patch نشده و در failover، سرویس بالا نمی‌آید.

**۷. فراموش کردن لایهٔ اپلیکیشن.**
کلاستر فایل کار می‌کند، اما اپلیکیشن session روی local disk دارد و از دست می‌رود.

### نکات مهم (Tips & Tricks)

- **heartbeat را جدا کن** — VLAN اختصاصی یا کابل مستقیم.
- **fencing را قبل از HA تست کن** — اگر fencing کار نکند، failover خطرناک است.
- **witness برای ۲ نود الزامی است.**
- **standby را ماهانه patch کن** — مثل production.
- **failover را در ساعات کاری تست کن** (با اطلاع)، نه فقط شب.
- **در آزمون:** کلیدواژهٔ «split-brain» → **quorum/fencing**؛ «2-node cluster» → **witness needed**.

### خلاصه فصل

- انواع کلاستر: **failover، load balancing، storage، database**.
- **Quorum** تصمیم می‌گیرد؛ در ۲ نود، **witness** لازم است.
- **Fencing/STONITH** جلوی split-brain را می‌گیرد.
- **Sync** بدون از دست رفتن داده، **async** برای DR دور.
- failover موفق = **سرویس از دید کلاینت** کار کند.
- کلاستر تست‌نشده، کلاستر نیست.

---

## فصل ۴۰: متعادل‌سازی بار

### مقدمه

کلاستر failover، «یکی بمیرد، دیگری بگیرد» است. متعادل‌سازی بار (load balancing) چیز دیگری است: **همه کار کنند، بار تقسیم شود.**

این فصل دربارهٔ الگوریتم‌ها، لایه‌های LB (L4/L7)، health check و sticky session است.

### مفاهیم اساسی

**Load balancer چه می‌کند؟**

```
[کلاینتها] → [VIP] → [Load Balancer] → ┬─ [Server 1]
                                        ├─ [Server 2]
                                        └─ [Server 3]
```

**لایه‌های LB:**

| لایه | توضیح | می‌بیند | مثال |
|---|---|---|---|
| **L4 (Transport)** | بر اساس IP/پورت | TCP/UDP | LVS، NLB، F5 LTM |
| **L7 (Application)** | بر اساس URL/header/cookie | HTTP | Nginx، HAProxy، ALB |

**L4 سریع‌تر است** (بدون باز کردن packet). **L7 انعطاف‌پذیرتر است** (مسیریابی بر اساس مسیر، هدر، cookie).

**الگوریتم‌های توزیع:**

| الگوریتم | توضیح | مناسب |
|---|---|---|
| **Round Robin** | نوبتی | سرورهای هم‌ظرفیت |
| **Weighted Round Robin** | نوبتی با وزن | سرورهای ناهم‌ظرفیت |
| **Least Connections** | کمترین اتصال فعال | اتصال‌های طولانی |
| **Least Response Time** | سریع‌ترین پاسخ | بار متغیر |
| **IP Hash** | بر اساس IP کلاینت | sticky بدون cookie |
| **Random** | تصادفی | ساده |
| **Consistent Hash** | hash پایدار | cache، sharding |

**نکتهٔ مهم:** «Round Robin» ساده‌ترین است اما **ظرفیت سرورها را در نظر نمی‌گیرد**. اگر یکی از سه سرور نصف ظرفیت داشته باشد، بار مساوی، آن را می‌خواباند.

**Health check:**

| نوع | توضیح |
|---|---|
| **TCP check** | آیا پورت باز است؟ |
| **HTTP check** | آیا URL وضعیت ۲۰۰ برمی‌گرداند؟ |
| **Custom check** | اسکریپت/endpoint اختصاصی |
| **Passive check** | بر اساس خطاهای واقعی |

**نکتهٔ حیاتی:** health check باید **عمیق** باشد. اگر فقط پورت چک شود، سروری که اپلیکیشنش خطا می‌دهد سالم دیده می‌شود و ترافیک می‌گیرد.

**مثال endpoint سلامت:**

```
GET /health
200 OK  {"status":"ok","db":"ok","cache":"ok"}
```

**Sticky session (session persistence):**

اگر اپلیکیشن session را روی local disk نگه می‌دارد، کاربر باید همیشه به همان سرور برود.

| روش | توضیح |
|---|---|
| **Cookie-based** | LB یک cookie می‌گذارد |
| **Source IP** | بر اساس IP کلاینت |
| **SSL session ID** | بر اساس session TLS |

**نکتهٔ مهم:** sticky session یک **بوی بد معماری** است. راه بهتر: session در Redis/DB مشترک، یا stateless با JWT. sticky فقط وقتی که نمی‌توان تغییر داد.

**SSL termination:**

| حالت | توضیح |
|---|---|
| **SSL termination** | LB رمزگشایی می‌کند، به backend HTTP |
| **SSL passthrough** | LB فقط عبور می‌دهد، backend رمزگشایی |
| **SSL bridging** | LB رمزگشایی و دوباره رمزگذاری |

**نکته:** termination ساده‌تر است (certificate در یک جا)، اما ترافیک داخلی بدون رمز است. برای امنیت بالاتر، bridging.

**Nginx به‌عنوان LB:**

```nginx
upstream backend {
    least_conn;
    server 10.0.10.21:8080 weight=3 max_fails=2 fail_timeout=30s;
    server 10.0.10.22:8080 weight=1;
    server 10.0.10.23:8080 backup;
    keepalive 32;
}

server {
    listen 443 ssl http2;
    server_name app.example.com;

    ssl_certificate     /etc/ssl/app.crt;
    ssl_certificate_key /etc/ssl/app.key;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 5s;
        proxy_read_timeout 60s;
    }

    location /health {
        access_log off;
        return 200 "ok\n";
    }
}
```

**HAProxy — نمونه:**

```
global
    maxconn 50000

defaults
    mode http
    timeout connect 5s
    timeout client  30s
    timeout server  30s

frontend web
    bind *:443 ssl crt /etc/ssl/app.pem
    default_backend app_servers

backend app_servers
    balance leastconn
    option httpchk GET /health
    http-check expect status 200
    server app1 10.0.10.21:8080 check weight 3
    server app2 10.0.10.22:8080 check weight 1
    server app3 10.0.10.23:8080 check backup
```

**نکتهٔ حیاتی:** `option httpchk` + `http-check expect status 200` = health check واقعی. بدون آن، فقط TCP چک می‌شود.

**Global Server Load Balancing (GSLB):** توزیع بین **سایت‌ها** (DNS-based، anycast، یا appliance). برای DR و geo-distribution.

### مثال‌های عملی

**Lab Scenario 1 — طراحی LB برای یک اپلیکیشن**

> **وضعیت:** سه سرور وب، اپلیکیشن session روی local disk، ۵۰۰۰ کاربر هم‌زمان. یکی از سرورها قوی‌تر است (۲ برابر).
>
> **وظیفه:** پیکربندی.
>
> **پاسخ:**
> ۱. **الگوریتم:** `least_conn` با وزن (سرور قوی weight=2، بقیه 1).
> ۲. **Health check:** `GET /health` با انتظار ۲۰۰.
> ۳. **Sticky:** چون session روی local disk است، cookie-based sticky لازم است — **اما** بهتر است session به Redis منتقل شود.
> ۴. **TLS:** termination روی LB، با HSTS.
> ۵. **Timeouts:** connect ۵s، read ۶۰s.
>
> **درس:** sticky، بدهی فنی است؛ راه‌حل بلندمدت، stateless یا session مشترک.

**Lab Scenario 2 — کاربران به‌طور تصادفی logout می‌شوند**

> **وضعیت:** بعد از اضافه شدن LB، کاربران هر چند دقیقه یک بار logout می‌شوند.
>
> **وظیفه:** علت؟
>
> **پاسخ:**
> - **علت:** session روی local disk هر سرور است، و LB بین سرورها می‌چرخد. کاربر به سرور دیگری می‌رود و session ندارد.
> - **راه‌حل فوری:** sticky session (cookie-based).
> - **راه‌حل درست:** session در Redis/Memcached مشترک، یا JWT stateless.
>
> **درس:** LB بدون در نظر گرفتن state، سرویس را می‌شکند.

### مقایسه و الگوهای مختلف

**مقایسهٔ L4 و L7:**

| معیار | L4 | L7 |
|---|---|---|
| سرعت | بالا | کمتر |
| آگاهی از محتوا | ❌ | ✅ |
| مسیریابی بر اساس URL | ❌ | ✅ |
| TLS termination | pass-through | ✅ |
| مناسب | TCP عمومی | HTTP/API |

**مقایسهٔ LB سخت‌افزاری و نرم‌افزاری:**

| معیار | سخت‌افزاری (F5) | نرم‌افزاری (Nginx/HAProxy) |
|---|---|---|
| عملکرد | بسیار بالا | بالا |
| هزینه | بسیار بالا | کم |
| انعطاف | متوسط | بالا |
| پشتیبانی | vendor | جامعه/تجاری |
| مناسب | enterprise بزرگ | اکثر موارد |

**مقایسهٔ الگوریتم‌ها:**

| الگوریتم | مزیت | عیب |
|---|---|---|
| Round Robin | ساده | ظرفیت را نمی‌بیند |
| Least Conn | تطبیق با بار | نیاز به state |
| IP Hash | sticky بدون cookie | توزیع نامتوازن |
| Weighted | ظرفیت متفاوت | نیاز به تنظیم |

### اشکال شایع

**۱. health check سطحی.**
سروری که اپلیکیشنش خطا می‌دهد، ترافیک می‌گیرد.

**۲. LB خودش SPOF.**
یک LB = نقطهٔ شکست. راه‌حل: LB دوتایی با VIP (keepalived/VRRP).

**۳. بدون sticky و بدون session مشترک.**
کاربران logout می‌شوند.

**۴. timeout پیش‌فرض.**
اتصال‌های طولانی قطع می‌شوند.

**۵. X-Forwarded-For را ست نکردن.**
اپلیکیشن IP واقعی کاربر را نمی‌بیند (لاگ، rate limit، geo).

**۶. وزن‌دهی نادرست.**
سرور ضعیف، بار مساوی می‌گیرد.

**۷. فراموش کردن WebSocket.**
پروکسی باید `Upgrade` و `Connection` را پاس بدهد، وگرنه WebSocket کار نمی‌کند.

### نکات مهم (Tips & Tricks)

- **LB را دوتایی کن** (active/passive با VRRP) — وگرنه SPOF جدید ساخته‌ای.
- **health check عمیق** — شامل دیتابیس و cache.
- **`X-Forwarded-For` و `X-Forwarded-Proto`** را همیشه ست کن.
- **برای WebSocket:** `proxy_set_header Upgrade $http_upgrade; proxy_set_header Connection "upgrade";`
- **session را از local disk بیرون ببر** — Redis، DB، یا JWT.
- **در آزمون:** کلیدواژهٔ «sessions lost after adding LB» → **sticky sessions / shared session store**.

### خلاصه فصل

- **L4** سریع، **L7** انعطاف‌پذیر.
- الگوریتم‌ها: round robin، least conn، weighted، IP hash.
- **health check عمیق** ضروری است.
- **sticky session** بدهی فنی است؛ راه‌حل، state مشترک.
- LB خودش باید افزونه باشد.
- `X-Forwarded-*` را ست کن.

---

## فصل ۴۱: آزمون failover و ظرفیت

### مقدمه

یک کلاستر تست‌نشده، فقط یک **فرض گران‌قیمت** است. و یک طرح ظرفیت تست‌نشده، فقط یک **حدس** است.

این فصل دربارهٔ **اثبات** است: چطور failover را تست کنی بدون اینکه سرویس تولیدی را بشکنی، و چطور بفهمی چقدر ظرفیت داری.

### مفاهیم اساسی

**چرا تست failover؟**

- کشف SPOFهای پنهان.
- اندازه‌گیری MTTR واقعی.
- آموزش تیم.
- اعتبارسنجی runbook.
- کشف تنظیمات فراموش‌شده (standby آپدیت‌نشده).

**انواع تست:**

| نوع | توضیح | ریسک |
|---|---|---|
| **Tabletop** | بحث روی کاغذ | صفر |
| **Non-disruptive** | تست بدون قطع (مثلاً بررسی config) | صفر |
| **Planned failover** | failover دستی در پنجرهٔ تغییر | کم |
| **Unplanned failover** | قطع واقعی (کابل، خاموشی) | متوسط |
| **Chaos engineering** | خرابی تصادفی در تولید | بالا |

**نکته:** از tabletop شروع کن، بعد planned، بعد unplanned در محیط آزمایشی. **هرگز unplanned را اول در تولید انجام نده.**

**چک‌لیست تست failover:**

```
□ آیا VIP منتقل شد؟
□ آیا سرویس از دید کلاینت کار می‌کند؟ (نه فقط process)
□ چقدر طول کشید؟ (MTTR)
□ آیا داده‌ای از دست رفت؟
□ آیا DNS درست است؟
□ آیا monitoring هشدار داد؟
□ آیا runbook کافی بود؟
□ آیا rollback کار می‌کند؟
```

**نکتهٔ حیاتی:** معیار موفقیت، **بالا آمدن resource** نیست؛ **کار کردن سرویس از دید کاربر** است.

**ظرفیت (Capacity) — مفاهیم:**

| مفهوم | توضیح |
|---|---|
| **Baseline** | مصرف عادی |
| **Peak** | حداکثر مصرف |
| **Headroom** | ظرفیت آزاد |
| **N+1 capacity** | تحمل خرابی یک نود |
| **Scaling factor** | ضریب رشد |

**قاعدهٔ ظرفیت HA:** در کلاستر N+1، هر نود باید بتواند بار **کل کلاستر منهای یک نود** را تحمل کند.

**مثال:** ۳ نود، هر کدام ۶۰٪ بار. اگر یکی بمیرد، دو نود باید ۹۰٪ بار را تحمل کنند. اگر ظرفیت هر نود ۱۰۰٪ باشد، ۹۰٪ ممکن است اما بدون headroom. **پس ۳ نود با ۶۰٪، برای N+1 کافی نیست** (بهتر: ۵۰٪ یا کمتر).

**فرمول:** برای N+1 با N نود، بار هر نود باید ≤ `(N−1)/N` باشد.

| تعداد نود | حداکثر بار هر نود برای N+1 |
|---|---|
| ۲ | ۵۰٪ |
| ۳ | ۶۶.۷٪ |
| ۴ | ۷۵٪ |
| ۵ | ۸۰٪ |

**نکته:** برای N+2، فرمول `(N−2)/N` است.

**آزمون بار (Load Testing):**

| نوع | هدف |
|---|---|
| **Smoke test** | آیا کار می‌کند؟ (بار کم) |
| **Load test** | رفتار در بار مورد انتظار |
| **Stress test** | نقطهٔ شکست |
| **Soak test** | بار متوسط برای مدت طولانی (نشت حافظه) |
| **Spike test** | افزایش ناگهانی بار |

**ابزارها:** Apache JMeter، k6، Locust، Gatling، wrk، ab.

**نکتهٔ مهم:** آزمون بار را در **محیط شبیه تولید** انجام بده. تست روی یک VM با ۲ هسته، عدد واقعی نمی‌دهد.

**متریک‌های آزمون بار:**

| متریک | معنی |
|---|---|
| **Throughput** | درخواست در ثانیه |
| **Latency p50/p95/p99** | صدک‌ها (مهم‌تر از میانگین) |
| **Error rate** | درصد خطا |
| **Saturation** | نقطه‌ای که منابع اشباع می‌شوند |

**نکتهٔ حیاتی:** **میانگین گمراه‌کننده است.** اگر p99 = ۵ ثانیه باشد، ۱٪ کاربران تجربهٔ بدی دارند — که در ۱۰۰٬۰۰۰ کاربر یعنی ۱۰۰۰ نفر.

**ظرفیت‌سنجی — رویکرد:**

```
۱. اندازه‌گیری baseline (۳۰ روز)
۲. تعیین peak (ساعت اوج، روز اوج)
۳. محاسبهٔ رشد (ماهانه/سالانه)
۴. تعیین headroom (۲۰–۳۰٪)
۵. تعیین N+1 (بار هر نود)
۶. پیش‌بینی تاریخ اشباع
```

**مثال:** رشد ۵٪ ماهانه، ظرفیت فعلی ۷۰٪:
```
ماه تا ۱۰۰٪ = log(100/70) ÷ log(1.05) ≈ ۷.۳ ماه
```
یعنی حدود ۷ ماه تا اشباع — اگر رشد ادامه یابد.

### مثال‌های عملی

**Lab Scenario 1 — برنامهٔ تست failover**

> **وظیفه:** یک برنامهٔ تست سه‌ماهه برای کلاستر فایل بنویس.
>
> **پاسخ:**
>
> | ماه | تست | محیط | معیار موفقیت |
> |---|---|---|---|
> | ۱ | Tabletop + بررسی config | — | runbook تأیید شد |
> | ۱ | Planned failover دستی | تولید (پنجرهٔ تغییر) | <۶۰ ثانیه، بدون خطا |
> | ۲ | قطع heartbeat | آزمایشی | quorum درست، بدون split-brain |
> | ۲ | خاموشی ناگهانی نود | آزمایشی | failover خودکار، بدون از دست رفتن داده |
> | ۳ | قطع برق یک PDU | تولید (با اطلاع) | بدون قطعی سرویس |
> | ۳ | آزمون بازگشت (failback) | تولید | بازگشت تمیز |
>
> **درس:** تست، یک رویداد نیست؛ یک برنامه است.

**Lab Scenario 2 — محاسبهٔ ظرفیت N+1**

> **وضعیت:** ۴ نود وب، هر کدام ظرفیت ۱۰۰۰ درخواست/ثانیه. بار فعلی ۲۸۰۰ req/s. آیا N+1 کافی است؟
>
> **پاسخ:**
> - حداکثر بار هر نود برای N+1: (۴−۱)/۴ = **۷۵٪** → ۷۵۰ req/s
> - بار فعلی هر نود: ۲۸۰۰ ÷ ۴ = **۷۰۰ req/s** ✓ (زیر ۷۵۰)
> - **اما:** اگر یکی بمیرد، ۳ نود باید ۲۸۰۰ را تحمل کنند = ۹۳۳ req/s هر نود → **۹۳٪** — بالاتر از ۷۵٪ و نزدیک به سقف.
> - **نتیجه:** N+1 فنی ممکن است، اما headroom کم است. با رشد ۱۰٪، باید نود پنجم اضافه شود.
>
> **درس:** N+1 را با **بار پس از خرابی** حساب کن، نه بار فعلی.

### مقایسه و الگوهای مختلف

**مقایسهٔ ابزارهای load test:**

| ابزار | زبان | مناسب |
|---|---|---|
| **JMeter** | GUI/XML | جامع، پیچیده |
| **k6** | JavaScript | مدرن، CI-friendly |
| **Locust** | Python | قابل برنامه‌نویسی |
| **wrk/ab** | CLI | سریع، ساده |
| **Gatling** | Scala | گزارش خوب |

**مقایسهٔ رویکردهای ظرفیت:**

| رویکرد | توضیح | مزیت | عیب |
|---|---|---|---|
| **Reactive** | وقتی پر شد، اضافه کن | ارزان | ریسک قطعی |
| **Predictive** | بر اساس رشد | برنامه‌ریزی | نیاز به داده |
| **Reserved** | همیشه ۳۰٪ آزاد | امن | هزینه |

**مقایسهٔ تست در تولید و آزمایشی:**

| معیار | تولید | آزمایشی |
|---|---|---|
| واقع‌گرایی | بالا | کم |
| ریسک | بالا | کم |
| هزینه | بالا (پنجرهٔ تغییر) | کم |
| توصیه | فقط با chaos engineering بالغ | شروع |

### اشکال شایع

**۱. تست failover فقط روی کاغذ.**
در بحران، تیم نمی‌داند چه کند.

**۲. سنجش موفقیت با «resource بالا آمد».**
سرویس از دید کلاینت کار نمی‌کند.

**۳. تست بدون اندازه‌گیری زمان.**
MTTR واقعی معلوم نمی‌شود.

**۴. N+1 با بار فعلی.**
باید با بار **پس از خرابی** حساب شود.

**۵. load test در محیط غیرواقعی.**
عددها بی‌معنا هستند.

**۶. نگاه به میانگین latency.**
p99 پنهان می‌شود.

**۷. فراموش کردن failback.**
بازگشت، خودش یک عملیات پرخطر است.

**۸. تست در ساعت اوج.**
قطعی تولیدی در بدترین زمان.

### نکات مهم (Tips & Tricks)

- **هر تست، یک runbook و یک rollback.**
- **زمان را اندازه بگیر** — کرنومتر، نه حس.
- **p95/p99 را گزارش کن**، نه میانگین.
- **failback را هم تست کن** — نصف تست‌ها فراموش می‌کنند.
- **بعد از هر تست، postmortem** — چه چیزی یاد گرفتیم.
- **در آزمون:** کلیدواژهٔ «N+1 capacity» → **each node must handle total load of N−1 nodes**.

### خلاصه فصل

- تست: **tabletop → planned → unplanned → chaos**.
- معیار موفقیت = **کار کردن سرویس از دید کلاینت**.
- N+1: بار هر نود ≤ `(N−1)/N`.
- ظرفیت: baseline → peak → growth → headroom.
- **p99** مهم‌تر از میانگین.
- failback را هم تست کن.

---

**بعدی:** [فایل ۰۹: مجازی‌سازی و ابر](09-Virtualization-and-Cloud.md)

---

## منابع و مراجع

- اهداف رسمی Server+ (V5)، دامنهٔ ۲: Server Administration (High availability)
- Microsoft Learn — Failover Clustering، quorum، NLB
- Pacemaker documentation: <https://clusterlabs.org/>
- Keepalived: <https://www.keepalived.org/>
- HAProxy Configuration Manual: <https://docs.haproxy.org/>
- Nginx — HTTP Load Balancing
- Google SRE Book — فصل‌های SLO و Load Balancing
- Patroni: <https://patroni.readthedocs.io/>
- <https://www.comptia.org/en-us/certifications/server/>
