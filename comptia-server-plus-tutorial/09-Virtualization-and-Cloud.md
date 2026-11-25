# مجازی‌سازی و ابر

**مدت یادگیری**: ۷ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فصل ۳۸ تا ۴۱ — HA و ظرفیت
**دامنهٔ آزمون**: Server Administration (۳۰٪)

---

## فهرست محتوا

- [فصل ۴۲: مجازی‌سازی و hypervisor](#ch-42)
- [فصل ۴۳: ماشین مجازی و تخصیص منابع](#ch-43)
- [فصل ۴۴: شبکه و ذخیره‌سازی در محیط مجازی](#ch-44)
- [فصل ۴۵: کانتینر در برابر ماشین مجازی](#ch-45)
- [فصل ۴۶: مدل‌های ابری و مهاجرت](#ch-46)

---

## فصل ۴۲: مجازی‌سازی و hypervisor

### مقدمه

مجازی‌سازی، بزرگ‌ترین تغییر در صنعت سرور در دو دههٔ گذشته است. قبل از آن، هر سرویس یک سرور فیزیکی می‌خواست. بعد از آن، یک سرور فیزیکی می‌تواند ۲۰ سرویس را میزبانی کند.

اما مجازی‌سازی فقط «چند سرور روی یک سرور» نیست. این فصل دربارهٔ **لایهٔ انتزاع** است: hypervisor، ماشین مجازی، و منطق تصمیم‌گیری.

### مفاهیم اساسی

**Hypervisor چیست؟** لایه‌ای که منابع فیزیکی (CPU، RAM، دیسک، شبکه) را بین ماشین‌های مجازی تقسیم می‌کند.

**دو نوع hypervisor:**

| نوع | توضیح | مثال | عملکرد |
|---|---|---|---|
| **Type 1 (bare-metal)** | مستقیم روی سخت‌افزار | ESXi، Hyper-V، KVM، Xen، Proxmox | بالا |
| **Type 2 (hosted)** | روی سیستم‌عامل | VirtualBox، VMware Workstation | پایین‌تر |

**نکتهٔ مهم:** KVM در واقع بخشی از کرنل لینوکس است، پس در دستهٔ Type 1 قرار می‌گیرد — حتی اگر با ابزارهای کاربری مدیریت شود.

**اجزای یک پلتفرم مجازی‌سازی:**

```
┌─────────────────────────────────────────┐
│  [VM1]  [VM2]  [VM3]  [VM4]             │
├─────────────────────────────────────────┤
│  Hypervisor (ESXi / Hyper-V / KVM)      │
├─────────────────────────────────────────┤
│  CPU    RAM    Disk    Network          │
└─────────────────────────────────────────┘
        + Management (vCenter / SCVMM)
        + Storage (SAN / vSAN / local)
        + Network (vSwitch / dvSwitch)
```

**مفاهیم کلیدی:**

- **Host** — سرور فیزیکی که hypervisor روی آن اجراست.
- **Guest** — ماشین مجازی.
- **VM** — فایل‌های روی دیسک که یک کامپیوتر را شبیه‌سازی می‌کنند.
- **vCPU** — پردازندهٔ مجازی.
- **vRAM** — حافظهٔ مجازی.
- **vDisk / VMDK / VHDX** — دیسک مجازی.
- **vSwitch** — سوئیچ مجازی.
- **Snapshot** — وضعیت لحظه‌ای VM.

**چرا مجازی‌سازی؟**

| مزیت | توضیح |
|---|---|
| **کاهش هزینه** | برق، فضا، سخت‌افزار |
| **انعطاف** | ساخت VM در دقیقه |
| **جداسازی** | هر VM مستقل |
| **HA** | VM روی host دیگر بالا می‌آید |
| **Snapshot** | قبل از تغییر، snapshot |
| **مقیاس** | افزودن host به کلاستر |

**معایب و ریسک‌ها:**

| ریسک | توضیح |
|---|---|
| **تمرکز (concentration)** | ۲۰ سرویس روی یک host = SPOF بزرگ |
| **overcommit** | تخصیص بیش از ظرفیت |
| **I/O noisy neighbor** | یک VM دیسک را اشباع می‌کند |
| **پیچیدگی** | لایهٔ اضافه برای عیب‌یابی |
| **وابستگی** | vendor lock-in |

**نکتهٔ حیاتی:** مجازی‌سازی، افزونگی **نمی‌سازد** — فقط آن را راحت‌تر می‌کند. اگر یک host با ۲۰ VM داشته باشی و آن host بمیرد، ۲۰ سرویس می‌خوابند. HA کلاستر لازم است.

**نسبت consolidation:** چند VM روی یک host؟

قاعدهٔ سنتی: ۱۰–۲۰ VM برای هر host فیزیکی. اما این عدد بی‌معناست بدون در نظر گرفتن:

- نوع workload (سبک/سنگین)
- ظرفیت RAM (گلوگاه اصلی)
- IOPS
- نیاز به CPU

**نکته:** **RAM** معمولاً اولین محدودیت است، نه CPU.

**Snapshot — قدرت و خطر:**

Snapshot، وضعیت دیسک را در لحظه ذخیره می‌کند. مزیت: rollback سریع. خطر:

- **رشد snapshot** — دیسک delta می‌تواند بزرگ‌تر از دیسک اصلی شود.
- **افت عملکرد** — نوشتن روی snapshot کندتر است.
- **snapshot ≠ backup** — اگر datastore بسوزد، snapshot هم می‌سوزد.
- **snapshot طولانی** — merge شدن آن ساعت‌ها طول می‌کشد.

**قاعده:** snapshot برای **ساعت‌ها**، نه هفته‌ها. و قبل از هر تغییر بزرگ، یکی بگیر و بعد پاک کن.

### مثال‌های عملی

**Lab Scenario 1 — انتخاب hypervisor**

> **وظیفه:** برای هر مورد، hypervisor پیشنهاد بده.
>
> الف) سازمان ۱۰۰ سرور، بودجهٔ enterprise، تیم VMware-trained
> ب) استارتاپ کوچک، بودجهٔ محدود، تیم لینوکس
> ج) آزمایشگاه روی لپ‌تاپ
>
> **پاسخ:**
>
> | مورد | انتخاب | دلیل |
> |---|---|---|
> | الف | **VMware ESXi + vCenter** | بالغ، پشتیبانی، تیم آموزش‌دیده |
> | ب | **Proxmox VE** یا **KVM** | متن‌باز، بدون لایسنس، لینوکس‌محور |
> | ج | **VirtualBox / VMware Workstation** (Type 2) | ساده، روی OS |
>
> **درس:** انتخاب hypervisor تابع **تیم، بودجه و اکوسیستم** است، نه فقط فنی.

**Lab Scenario 2 — VM کند است**

> **وضعیت:** یک VM روی host با ۳۲ هسته و ۵۱۲GB RAM کند است. VM: ۸ vCPU، ۳۲GB RAM. CPU host ۴۰٪، RAM host ۵۵٪.
>
> **وظیفه:** سه احتمال.
>
> **پاسخ:**
> ۱. **Storage:** datastore اشباع است (`%util` بالا، latency بالا). شایع‌ترین.
> ۲. **Noisy neighbor:** VM دیگری I/O یا شبکه را می‌بلعد.
> ۳. **CPU ready time / steal:** VM منتظر CPU است. در vSphere: `%RDY` بالا (>۵٪ نگران‌کننده).
>
> **درس:** در مجازی‌سازی، «منابع host آزاد» به معنای «VM سالم» نیست.

### مقایسه و الگوهای مختلف

**مقایسهٔ hypervisorها:**

| معیار | ESXi | Hyper-V | KVM/Proxmox | Xen |
|---|---|---|---|---|
| نوع | Type 1 | Type 1 | Type 1 (kernel) | Type 1 |
| هزینه | بالا | متوسط (با ویندوز) | رایگان | رایگان |
| مدیریت | vCenter | SCVMM/WAC | Proxmox UI/CLI | XenCenter |
| بالغ بودن | بسیار بالا | بالا | بالا | بالا |
| مناسب | enterprise | اکوسیستم MS | SME، ابر | ابر (AWS) |

**مقایسهٔ Full virtualization و Paravirtualization:**

| معیار | Full | Para |
|---|---|---|
| تغییر guest OS | لازم نیست | لازم است |
| عملکرد | خوب | بهتر (I/O) |
| سازگاری | همه | محدود |
| امروز | استاندارد | در virtio دیده می‌شود |

**نکته:** امروز «paravirtualized drivers» (virtio) استاندارد است: سریع‌تر از شبیه‌سازی کامل دستگاه.

**مقایسهٔ مجازی‌سازی و کانتینر (اشاره به فصل ۴۵):**

| معیار | VM | Container |
|---|---|---|
| جداسازی | سخت‌افزاری (کامل) | کرنل مشترک |
| زمان راه‌اندازی | دقیقه | ثانیه |
| مصرف | بالا | کم |
| امنیت | بالاتر | کمتر (کرنل مشترک) |
| مناسب | سرویس‌های متفاوت | میکروسرویس |

### اشکال شایع

**۱. فرض اینکه مجازی‌سازی = HA.**
host می‌میرد، همه VMها می‌میرند. HA کلاستر لازم است.

**۲. snapshot به‌جای backup.**
snapshot روی همان datastore است. اگر datastore از دست برود، همه چیز می‌رود.

**۳. snapshot هفته‌ها باقی می‌ماند.**
دیسک delta بزرگ می‌شود، عملکرد می‌افتد، merge ساعت‌ها طول می‌کشد.

**۴. overcommit بیش از حد.**
RAM overcommit ۱.۵ و CPU ۴:۱ در بار اوج، swap و کندی شدید.

**۵. نادیده گرفتن IOPS.**
۳۰ VM روی یک datastore با ۵۰۰ IOPS = فاجعه.

**۶. VM با منابع «بیش از نیاز».**
۸ vCPU برای یک سرویس سبک، باعث CPU ready و هدر می‌شود.

**۷. بی‌توجهی به affinity.**
دو VM که با هم حرف می‌زنند، روی دو host مختلف = ترافیک اضافه.

**۸. نبود مستندات VM.**
نمی‌دانی کدام VM چه نقشی دارد، کدام snapshot دارد، کدام orphaned است.

### نکات مهم (Tips & Tricks)

- **RAM، گلوگاه اول است** — قبل از CPU، حافظه را حساب کن.
- **snapshot را در تقویم پاک کن** — یک یادآور برای هر snapshot.
- **`%RDY` (CPU ready) را پایش کن** — بالای ۵٪ یعنی VM منتظر CPU است.
- **datastore را با IOPS طراحی کن، نه فقط ظرفیت.**
- **برای هر VM، یک نقش** — نه «سرور همه‌کاره».
- **در آزمون:** کلیدواژهٔ «Type 1 hypervisor» → **bare-metal**؛ «snapshot» → **not a backup**.

### خلاصه فصل

- **Type 1** (bare-metal) و **Type 2** (hosted).
- مجازی‌سازی هزینه و انعطاف می‌دهد، اما **تمرکز ریسک** می‌سازد.
- **RAM** معمولاً گلوگاه است، نه CPU.
- **snapshot ≠ backup** و عمر کوتاه دارد.
- IOPS و noisy neighbor را جدی بگیر.
- مجازی‌سازی بدون HA، SPOF بزرگ می‌سازد.

---

## فصل ۴۳: ماشین مجازی و تخصیص منابع

### مقدمه

ساخت یک VM ساده است: چند کلیک. ساخت یک VM **درست** سخت است: چقدر vCPU، چقدر RAM، چه نوع دیسک، چه تنظیماتی.

تخصیص اشتباه، هم عملکرد را می‌کشد هم منابع host را هدر می‌دهد. این فصل دربارهٔ تصمیم‌های تخصیص است.

### مفاهیم اساسی

**vCPU — نکات حیاتی:**

- **vCPU ≠ هستهٔ فیزیکی.** hypervisor زمان‌بندی می‌کند.
- **overcommit** — تخصیص vCPU بیشتر از هستهٔ فیزیکی.
- **CPU ready / %RDY** — زمانی که VM آماده است اما CPU ندارد. بالای ۵٪ = مشکل.
- **co-stop** — در VM با چند vCPU، همهٔ vCPUها باید هم‌زمان زمان‌بندی شوند. VM با ۸ vCPU روی host شلوغ، بیشتر منتظر می‌ماند.

**قاعدهٔ مهم:** **vCPU کمتر، بهتر.** یک VM با ۲ vCPU اغلب سریع‌تر از همان VM با ۸ vCPU روی host شلوغ است.

**توصیهٔ vCPU:**

| نوع workload | vCPU |
|---|---|
| سرویس سبک (DNS، NTP) | ۱ |
| وب سرور معمولی | ۲–۴ |
| دیتابیس متوسط | ۴–۸ |
| دیتابیس سنگین | ۸–۱۶ (با توجه به هستهٔ فیزیکی) |

**نکته:** vCPU را با **هستهٔ فیزیکی** هم‌تراز کن، نه با رشتهٔ منطقی. اگر host ۱۶ هسته با SMT (۳۲ رشته) دارد، VM با ۳۲ vCPU منطقی ممکن است اما کارایی ندارد.

**حافظه — سه مدل تخصیص:**

| مدل | توضیح |
|---|---|
| **Reserved** | حافظهٔ فیزیکی رزرو شده، تضمینی |
| **Limit** | سقف مصرف |
| **Shares** | اولویت در رقابت |

**Overcommit حافظه:**

- **Ballooning** — hypervisor از VM می‌خواهد حافظه آزاد کند (driver داخل VM).
- **Swap** — اگر ballooning کافی نبود، حافظه به دیسک منتقل می‌شود (خیلی کند).
- **Compression** — فشرده‌سازی در RAM (سریع‌تر از swap).

**نکتهٔ حیاتی:** overcommit حافظه با ballooning خوب کار می‌کند، اما **swap در hypervisor فاجعه است**. اگر `swap` در host فعال شد، عملکرد VMها سقوط می‌کند.

**دیسک مجازی — انواع:**

| نوع | توضیح | مزیت | عیب |
|---|---|---|---|
| **Thick eager zeroed** | از ابتدا پر و صفر | عملکرد بالا، امن | زمان ساخت طولانی، فضا |
| **Thick lazy zeroed** | فضا رزرو، صفرنشده | ساخت سریع | عملکرد اولیهٔ کمتر |
| **Thin** | فضا به‌تدریج | صرفه‌جویی | oversubscription، عملکرد |

**نکته:** برای دیتابیس و بار حساس، **thick eager zeroed**. برای VDI و dev، **thin**.

**کنترلر دیسک:**

| کنترلر | مناسب |
|---|---|
| **LSI Logic SAS** | سازگاری بالا |
| **VMware Paravirtual (PVSCSI)** | عملکرد بالا، بار سنگین |
| **SATA** | ساده، کم‌عملکرد |
| **NVMe** | جدیدترین، بالاترین عملکرد |
| **virtio-blk / virtio-scsi** | استاندارد KVM |

**نکته:** PVSCSI/virtio برای دیتابیس و بار I/O سنگین، عملکرد بهتری می‌دهد.

**ابزارهای مهمان (VMware Tools / Integration Services / qemu-guest-agent):**

- **مزیت:** هماهنگی زمان، ballooning، shutdown تمیز، driver بهینه.
- **بدون آن:** عملکرد ضعیف، snapshot ناسازگار، shutdown ناگهانی.

**نکته:** ابزار مهمان را **همیشه** نصب کن. یکی از رایج‌ترین دلایل «VM کند» نبود آن است.

**قالب و کلون:**

| روش | توضیح |
|---|---|
| **Template** | VM آماده، کلون سریع |
| **Full clone** | کپی کامل، مستقل |
| **Linked clone** | دیسک مشترک با parent، سریع اما وابسته |
| **Snapshot clone** | از snapshot |

**نکته:** linked clone سریع است اما اگر parent حذف شود، همهٔ cloneها می‌میرند.

**Cloud-init / Sysprep:** برای پیکربندی اولیه (hostname، IP، SSH key) در کلون.

### مثال‌های عملی

**Lab Scenario 1 — تخصیص منابع برای سه VM**

> **وظیفه:** برای هر VM، vCPU/RAM/disk پیشنهاد بده.
>
> الف) DNS/DHCP
> ب) PostgreSQL با ۵۰۰ تراکنش/ثانیه
> ج) سرور توسعه با IDE
>
> **پاسخ:**
>
> | VM | vCPU | RAM | Disk | نکته |
> |---|---|---|---|---|
> | DNS/DHCP | ۱ | ۲GB | ۴۰GB thin | سبک، uptime مهم |
> | PostgreSQL | ۸ | ۳۲GB | ۵۰۰GB thick | PVSCSI، ابزار مهمان |
> | Dev | ۴ | ۱۶GB | ۱۰۰GB thin | snapshot مجاز |
>
> **درس:** «بیشتر» بهتر نیست؛ **متناسب** بهتر است.

**Lab Scenario 2 — host با RAM پر**

> **وضعیت:** host با ۲۵۶GB RAM، مجموع vRAM تخصیص‌یافته ۳۲۰GB (overcommit ۱.۲۵). هشدار: host swap فعال شده.
>
> **وظیفه:** تحلیل و اقدام.
>
> **پاسخ:**
> - **overcommit ۱.۲۵** به‌خودی‌خود بد نیست، اما **swap فعال شدن** یعنی فشار واقعی.
> - **اقدام ۱:** بررسی ballooning — آیا ابزار مهمان نصب است؟ بدون آن، ballooning کار نمی‌کند.
> - **اقدام ۲:** پیدا کردن VM با مصرف بالا (`esxtop` → `%ACTV`، `%SWPWT`).
> - **اقدام ۳:** کاهش vRAM VMهای کم‌مصرف (اغلب تخصیص بیش از نیاز).
> - **اقدام ۴:** افزودن host به کلاستر.
>
> **درس:** overcommit با ابزار مهمان کار می‌کند؛ بدون آن، فاجعه.

### مقایسه و الگوهای مختلف

**مقایسهٔ thick و thin:**

| معیار | Thick | Thin |
|---|---|---|
| مصرف اولیه | کامل | حداقلی |
| عملکرد | بالاتر | کمی کمتر |
| oversubscription | ❌ | ✅ |
| مناسب | دیتابیس، تولید حساس | VDI، dev، آرشیو |

**مقایسهٔ full clone و linked clone:**

| معیار | Full | Linked |
|---|---|---|
| زمان ساخت | دقیقه | ثانیه |
| فضا | کامل | حداقلی |
| وابستگی | مستقل | به parent |
| مناسب | تولید | تست، VDI موقت |

**مقایسهٔ overcommit در CPU و RAM:**

| معیار | CPU overcommit | RAM overcommit |
|---|---|---|
| مکانیزم | زمان‌بندی | ballooning/swap |
| ریسک | CPU ready | swap (بدتر) |
| نسبت امن | ۲:۱ تا ۴:۱ | ۱.۱:۱ تا ۱.۳:۱ |

**نکته:** CPU overcommit بالاتر امن است چون زمان‌بندی خوب کار می‌کند. RAM overcommit خطرناک‌تر است چون swap فاجعه است.

### اشکال شایع

**۱. vCPU زیاد برای VM سبک.**
co-stop و CPU ready. VM کندتر می‌شود.

**۲. نصب نکردن ابزار مهمان.**
ballooning، زمان، shutdown، driver — همه ضعیف.

**۳. thin برای دیتابیس پرترافیک.**
عملکرد پایین‌تر و ریسک oversubscription.

**۴. snapshot هفته‌ها.**
رشد delta، افت عملکرد.

**۵. linked clone در تولید.**
وابستگی به parent، ریسک.

**۶. نادیده گرفتن NUMA.**
VM با ۳۲ vCPU روی host دو‌سوکتی، ممکن است بین NUMA node پخش شود و کند شود.

**۷. تخصیص RAM بیشتر از نیاز.**
هدر منابع، کاهش consolidation ratio.

**۸. کلون بدون cloud-init/sysprep.**
نام تکراری، IP تکراری، SID تکراری.

### نکات مهم (Tips & Tricks)

- **vCPU را کم شروع کن، بعد اضافه کن.** برعکسش سخت است.
- **ابزار مهمان را همیشه نصب کن.**
- **`%RDY` > ۵٪** = مشکل CPU ready.
- **`%SWPWT` > ۰** = host در حال swap؛ بحرانی.
- **برای دیتابیس: thick + PVSCSI/virtio.**
- **در آزمون:** کلیدواژهٔ «VM slow, host has free resources» → **CPU ready / storage latency / noisy neighbor**.

### خلاصه فصل

- **vCPU کمتر بهتر**؛ overcommit امن است اما نه بی‌حد.
- RAM: **reserved/limit/shares**؛ ballooning نیاز به ابزار مهمان دارد.
- دیسک: **thick** برای حساس، **thin** برای انعطاف.
- کنترلر: **PVSCSI/virtio** برای بار سنگین.
- **ابزار مهمان الزامی** است.
- clone بدون cloud-init/sysprep = مشکل.

---

## فصل ۴۴: شبکه و ذخیره‌سازی در محیط مجازی

### مقدمه

در مجازی‌سازی، شبکه و ذخیره‌سازی «مجازی» می‌شوند — اما فیزیکی باقی می‌مانند. یک vSwitch روی NIC فیزیکی کار می‌کند؛ یک datastore روی SAN فیزیکی.

این فصل دربارهٔ لایهٔ انتزاع شبکه و ذخیره‌سازی است: vSwitch، port group، VLAN، datastore، VMFS، vSAN.

### مفاهیم اساسی

**شبکهٔ مجازی — اجزا:**

```
[VM] → [vNIC] → [Port Group] → [vSwitch] → [pNIC/Uplink] → [سوئیچ فیزیکی]
```

| جزء | توضیح |
|---|---|
| **vNIC** | کارت شبکهٔ VM |
| **Port Group** | گروه پورت با تنظیمات مشترک (VLAN، security) |
| **vSwitch** | سوئیچ مجازی داخل hypervisor |
| **Uplink / pNIC** | NIC فیزیکی host |
| **dvSwitch** (توزیع‌شده) | سوئیچ مجازی در سطح کلاستر |

**انواع vSwitch:**

| نوع | توضیح |
|---|---|
| **Standard vSwitch** | هر host جدا |
| **Distributed vSwitch** | مدیریت متمرکز در کلاستر |

**VLAN در مجازی‌سازی — سه روش:**

| روش | توضیح |
|---|---|
| **EST (External Switch Tagging)** | VLAN روی سوئیچ فیزیکی، port group بدون تگ |
| **VST (Virtual Switch Tagging)** | VLAN روی port group، trunk به سوئیچ |
| **VGT (Virtual Guest Tagging)** | VLAN داخل VM (802.1Q در guest) |

**نکتهٔ مهم:** VST رایج‌ترین است: uplink به‌صورت trunk، و هر port group یک VLAN ID دارد.

**Network segmentation در مجازی‌سازی:**

- **Management network** — مدیریت host.
- **VM network** — ترافیک ماشین‌ها.
- **Storage network** — iSCSI/NFS.
- **vMotion/Live Migration** — انتقال VM.
- **Fault tolerance / replication**.

**نکتهٔ حیاتی:** این شبکه‌ها را **جدا** کن (VLAN یا NIC فیزیکی جدا). اگر vMotion و iSCSI روی یک شبکه باشند، migration می‌تواند storage را اشباع کند.

**ذخیره‌سازی مجازی — انواع datastore:**

| نوع | پروتکل | مناسب |
|---|---|---|
| **VMFS** | بلوک (SAN) | VMware سنتی |
| **NFS** | فایل | ساده، انعطاف |
| **vSAN** | توزیع‌شده | HCI |
| **RDM** | بلوک مستقیم | دیتابیس cluster |
| **VVOL** | بلوک مجازی | سیاست‌محور |
| **CSV** (Hyper-V) | بلوک | کلاستر ویندوز |
| **SMB3** (Hyper-V) | فایل | ویندوز |

**VMFS در برابر NFS:**

| معیار | VMFS | NFS |
|---|---|---|
| نوع | بلوک | فایل |
| مدیریت | نیاز به LUN | ساده |
| عملکرد | بالا | بالا (با 10GbE) |
| snapshot | بومی | در NFS datastore |
| مناسب | enterprise | ساده، SME |

**vSAN (HCI):**

- دیسک محلی هر host در یک pool مشترک.
- نیاز به حداقل ۳ host.
- **Storage Policy** — تعداد replica، FTT (Failures To Tolerate).
- **مزیت:** بدون SAN، مقیاس افقی.
- **عیب:** وابسته به شبکه، پیچیدگی troubleshooting.

**نکتهٔ مهم:** در vSAN با FTT=1، هر VM دو replica دارد → **۵۰٪ ظرفیت مصرف**. با FTT=2، سه replica → ۳۳٪ کارایی.

**Thin/thick در datastore:**

- **VMFS thin** — دیسک VM به‌تدریج.
- **VMFS thick** — رزرو کامل.
- **NFS thin** — همیشه thin.

**نکته:** datastore بالای ۸۵٪ = هشدار؛ بالای ۹۰٪ = مشکل جدی (snapshot نمی‌تواند رشد کند).

**Storage I/O Control (SIOC) / QoS:** محدود کردن IOPS هر VM برای جلوگیری از noisy neighbor.

**Multipath در مجازی‌سازی:** هر host باید چند مسیر به LUN داشته باشد (فصل ۲۱). **PSP** (Path Selection Policy):

- **Fixed** — یک مسیر فعال.
- **MRU** (Most Recently Used).
- **Round Robin** — چرخشی (پیش‌فرض مدرن).

### مثال‌های عملی

**Lab Scenario 1 — طراحی شبکهٔ host**

> **وضعیت:** host با ۶ NIC فیزیکی 10GbE. باید مدیریت، VM، storage و vMotion را جدا کنی.
>
> **وظیفه:** تخصیص.
>
> **پاسخ:**
>
> | NIC | استفاده | VLAN |
> |---|---|---|
> | vmnic0+1 | Management + vMotion | 10 |
> | vmnic2+3 | VM traffic | 20, 30 |
> | vmnic4+5 | iSCSI (MPIO) | 50 |
>
> - هر گروه با active/standby یا LACP.
> - iSCSI با jumbo frame و VLAN جدا.
> - vMotion در VLAN مدیریت (یا جدا).
>
> **درس:** جداسازی ترافیک، پایهٔ عملکرد و امنیت است.

**Lab Scenario 2 — datastore پر شد**

> **وضعیت:** datastore ۲TB با ۹۲٪ استفاده. VMها کار می‌کنند اما هشدار.
>
> **وظیفه:** اقدام.
>
> **پاسخ:**
> ۱. **پیدا کردن مقصر:** snapshotهای قدیمی (`Get-Snapshot`)، دیسک‌های orphaned، ISOهای فراموش‌شده.
> ۲. **پاک‌سازی:** حذف snapshotهای قدیمی (با احتیاط — merge زمان می‌برد).
> ۳. **انتقال:** Storage vMotion یک VM به datastore دیگر.
> ۴. **گسترش:** افزودن LUN یا گسترش datastore.
> ۵. **پیشگیری:** هشدار در ۷۵٪، بازبینی ماهانه.
>
> **درس:** snapshotها بزرگ‌ترین مصرف‌کنندهٔ پنهان datastore هستند.

### مقایسه و الگوهای مختلف

**مقایسهٔ Standard و Distributed vSwitch:**

| معیار | Standard | Distributed |
|---|---|---|
| مدیریت | هر host جدا | متمرکز |
| مهاجرت VM | تنظیمات از دست می‌رود | حفظ می‌شود |
| هزینه | رایگان | لایسنس |
| مناسب | کوچک | کلاستر |

**مقایسهٔ SAN، NAS و vSAN در مجازی‌سازی:**

| معیار | SAN (VMFS) | NAS (NFS) | vSAN |
|---|---|---|---|
| هزینهٔ ورودی | بالا | متوسط | متوسط |
| عملکرد | بالا | بالا | بالا |
| مقیاس | vertical | متوسط | horizontal |
| پیچیدگی | متوسط | کم | بالا |
| مناسب | enterprise | SME | HCI |

**مقایسهٔ RDM و VMDK:**

| معیار | RDM | VMDK |
|---|---|---|
| snapshot | محدود | ✅ |
| cluster دیتابیس | ✅ | ❌ (معمولاً) |
| انعطاف | کم | بالا |
| مناسب | MSCS، Oracle RAC | عمومی |

### اشکال شایع

**۱. همهٔ ترافیک روی یک vSwitch.**
storage و vMotion و VM روی یک uplink = گلوگاه.

**۲. VLAN نادرست روی port group.**
VM در VLAN اشتباه، دسترسی ندارد.

**۳. jumbo frame فقط در یک طرف.**
MTU mismatch = عملکرد بد.

**۴. datastore بالای ۹۰٪.**
snapshot نمی‌تواند رشد کند، VM متوقف می‌شود.

**۵. snapshot به‌جای backup.**
datastore می‌سوزد، snapshot هم.

**۶. نادیده گرفتن IOPS در طراحی datastore.**
ظرفیت کافی، IOPS ناکافی.

**۷. بدون multipath.**
یک مسیر قطع = datastore از دسترس خارج.

**۸. vMotion روی شبکهٔ ۱GbE.**
migration کند، سرویس تحت فشار.

### نکات مهم (Tips & Tricks)

- **چهار شبکهٔ مجزا:** management، VM، storage، vMotion.
- **jumbo frame را همه‌جا یکسان کن.**
- **datastore را زیر ۸۰٪ نگه دار.**
- **snapshot را در تقویم پاک کن.**
- **Round Robin برای multipath** (پیش‌فرض مدرن).
- **در آزمون:** کلیدواژهٔ «VLAN tagging in virtual» → **VST**؛ «datastore full» → **snapshot/orphaned disks**.

### خلاصه فصل

- شبکهٔ مجازی: **vNIC → port group → vSwitch → uplink**.
- VLAN: **EST، VST، VGT** — VST رایج‌ترین.
- جداسازی ترافیک: **management، VM، storage، vMotion**.
- datastore: **VMFS، NFS، vSAN**.
- **vSAN FTT=1 = ۵۰٪ ظرفیت**.
- datastore بالای ۸۵٪ هشدار، بالای ۹۰٪ بحران.

---

## فصل ۴۵: کانتینر در برابر ماشین مجازی

### مقدمه

کانتینر، «مجازی‌سازی سبک» است — اما نه مجازی‌سازی. تفاوت بنیادی: VM یک سیستم‌عامل کامل دارد؛ کانتینر **کرنل را با بقیه شریک است**.

این تفاوت، همهٔ مزایا و معایب را می‌سازد. این فصل دربارهٔ انتخاب درست بین این دو است.

### مفاهیم اساسی

**معماری:**

```
VM:                              Container:
┌─────────┐ ┌─────────┐          ┌─────┐ ┌─────┐ ┌─────┐
│ App A   │ │ App B   │          │App A│ │App B│ │App C│
│ Bins/Lib│ │ Bins/Lib│          │Libs │ │Libs │ │Libs │
│ Guest OS│ │ Guest OS│          └─────┘ └─────┘ └─────┘
├─────────┴─┴─────────┤          ├─────────────────────┤
│     Hypervisor      │          │   Container Runtime │
├─────────────────────┤          ├─────────────────────┤
│      Host OS        │          │      Host OS        │
└─────────────────────┘          └─────────────────────┘
```

**تفاوت‌های کلیدی:**

| معیار | VM | Container |
|---|---|---|
| جداسازی | سخت‌افزاری (کامل) | فرآیندی (namespace/cgroup) |
| کرنل | مستقل | مشترک با host |
| زمان راه‌اندازی | ۳۰ ثانیه–چند دقیقه | <۱ ثانیه |
| حجم | GB | MB |
| تراکم | ده‌ها | صدها |
| امنیت | بالاتر | کمتر (کرنل مشترک) |
| OS متفاوت | ✅ (ویندوز روی لینوکس) | ❌ (کرنل مشترک) |
| State | stateful راحت | stateless ترجیح |

**مفاهیم کانتینر:**

- **Image** — قالب فقط‌خواندنی (لایه‌ها).
- **Container** — نمونهٔ در حال اجرا از image.
- **Registry** — مخزن image (Docker Hub، Harbor، ECR).
- **Dockerfile** — دستور ساخت image.
- **Volume** — ذخیره‌سازی پایدار.
- **Namespace** — جداسازی (PID، network، mount، user).
- **cgroup** — محدودیت منابع (CPU، RAM).

**نکتهٔ حیاتی:** کانتینر **جداسازی امنیتی کامل نیست**. اگر کرنل آسیب‌پذیر باشد، کانتینر می‌تواند از آن خارج شود (container escape). برای multi-tenant، VM امن‌تر است.

**ارکستراسیون:**

| ابزار | توضیح |
|---|---|
| **Kubernetes** | استاندارد صنعت |
| **Docker Swarm** | ساده‌تر، کم‌کاربرد |
| **Nomad** | سبک، چندمنظوره |
| **OpenShift** | Kubernetes سازمانی |

**مفاهیم Kubernetes (سطح بالا):**

- **Pod** — کوچک‌ترین واحد (یک یا چند container).
- **Deployment** — مدیریت replica.
- **Service** — آدرس پایدار برای podها.
- **Ingress** — ورود HTTP.
- **ConfigMap/Secret** — پیکربندی.
- **PersistentVolume** — ذخیره‌سازی پایدار.
- **Namespace** — جداسازی منطقی.

**نکتهٔ مهم:** Kubernetes یک **لایهٔ اضافه** است. برای ۳ کانتینر، Docker Compose کافی است. برای ۳۰۰، Kubernetes.

**VM در برابر کانتینر — چه زمانی کدام؟**

| سناریو | انتخاب |
|---|---|
| اپلیکیشن legacy (Windows، .NET Framework) | VM |
| دیتابیس با I/O سنگین | VM (یا bare-metal) |
| میکروسرویس stateless | Container |
| CI/CD runner | Container |
| Multi-tenant با نیاز امنیتی بالا | VM |
| Batch job کوتاه | Container |
| سرویس با نیاز به کرنل خاص | VM |
| توسعه و تست سریع | Container |

**نکته:** بسیاری از سازمان‌ها **هر دو** را دارند: VM برای legacy و دیتابیس، container برای اپلیکیشن جدید.

**VM روی Kubernetes (و برعکس):**

- **KubeVirt** — اجرای VM در Kubernetes.
- **Kubernetes در VM** — رایج (managed Kubernetes).
- **VM با container runtime** — Docker در VM.

### مثال‌های عملی

**Lab Scenario 1 — انتخاب برای سه workload**

> **وظیفه:** VM یا container؟
>
> الف) یک اپلیکیشن PHP با ۲۰۰ کاربر
> ب) SQL Server 2016
> ج) ۵۰ میکروسرویس با deploy روزانه
>
> **پاسخ:**
>
> | مورد | انتخاب | دلیل |
> |---|---|---|
> | الف | **VM** یا container | اگر stateless است، container؛ اگر session محلی دارد، VM |
> | ب | **VM** | دیتابیس stateful، I/O سنگین، لایسنس |
> | ج | **Container + Kubernetes** | مقیاس، deploy مکرر، stateless |
>
> **درس:** «مدرن» به معنای «container برای همه چیز» نیست.

**Lab Scenario 2 — کانتینر امن نیست**

> **وضعیت:** یک کانتینر با `--privileged` اجرا می‌شود و به `/var/run/docker.sock` دسترسی دارد.
>
> **وظیفه:** ریسک را توضیح بده.
>
> **پاسخ:**
> - `--privileged` یعنی کانتینر تقریباً همهٔ قابلیت‌های host را دارد.
> - دسترسی به `docker.sock` یعنی می‌تواند کانتینر جدید (با mount روت host) بسازد → **فرار کامل از کانتینر**.
> - **اقدام:** حذف `--privileged`، حذف mount سوکت، اجرای non-root، read-only root filesystem، seccomp/AppArmor.
>
> **درس:** کانتینر با تنظیمات اشتباه، جداسازی ندارد.

### مقایسه و الگوهای مختلف

**مقایسهٔ Docker و Podman:**

| معیار | Docker | Podman |
|---|---|---|
| daemon | ✅ (dockerd) | ❌ (daemonless) |
| rootless | محدود | ✅ |
| سازگاری | استاندارد | سازگار با Docker CLI |
| مناسب | عمومی | امنیت بالاتر |

**مقایسهٔ Kubernetes و Docker Compose:**

| معیار | Compose | Kubernetes |
|---|---|---|
| پیچیدگی | کم | بالا |
| مقیاس | یک host | کلاستر |
| self-healing | ❌ | ✅ |
| مناسب | dev، کوچک | تولید بزرگ |

**مقایسهٔ VM و container از دید امنیت:**

| لایه | VM | Container |
|---|---|---|
| جداسازی کرنل | ✅ | ❌ |
| attack surface | hypervisor | کرنل + runtime |
| escape | نادر | ممکن |
| توصیه multi-tenant | ✅ | با sandbox (gVisor، Kata) |

**نکته:** **gVisor** و **Kata Containers** جداسازی قوی‌تری برای کانتینر می‌دهند (شبیه VM سبک).

### اشکال شایع

**۱. container برای دیتابیس تولیدی بدون برنامه.**
stateful، backup، persistence — همه سخت‌تر.

**۲. `--privileged` برای «حل کردن مشکل».**
باز کردن درِ امنیت.

**۳. image بدون tag مشخص.**
`latest` می‌تواند فردا چیز دیگری باشد. همیشه tag دقیق.

**۴. image بزرگ.**
لایه‌های اضافه، زمان pull طولانی. multi-stage build.

**۵. اجرای container با root.**
اگر escape رخ دهد، root host است.

**۶. بی‌توجهی به محدودیت منابع.**
یک container می‌تواند کل host را ببلعد. `--memory`، `--cpus`.

**۷. Kubernetes برای ۳ کانتینر.**
پیچیدگی بی‌دلیل.

**۸. فرض اینکه container = امن.**
جداسازی کمتر از VM است.

### نکات مهم (Tips & Tricks)

- **image را با tag دقیق (digest) پین کن.**
- **multi-stage build** برای image کوچک.
- **non-root user** در Dockerfile.
- **`--read-only`** برای root filesystem.
- **محدودیت منابع** همیشه.
- **در آزمون:** کلیدواژهٔ «shares kernel» → **container**؛ «separate OS» → **VM**.

### خلاصه فصل

- کانتینر **کرنل مشترک** دارد؛ VM مستقل.
- کانتینر: سریع، سبک، **جداسازی کمتر**.
- VM: سنگین‌تر، **جداسازی بیشتر**.
- **Kubernetes** برای مقیاس؛ Compose برای کوچک.
- `--privileged` و docker.sock = ریسک فرار.
- بسیاری از سازمان‌ها **هر دو** را دارند.

---

## فصل ۴۶: مدل‌های ابری و مهاجرت

### مقدمه

ابر، «کامپیوتر کسی دیگر» است — اما با قرارداد، API و مدل هزینه‌ای متفاوت. برای مدیر سرور، ابر یعنی تصمیم: چه چیزی بماند، چه چیزی برود، و چطور.

این فصل مدل‌های سرویس، استقرار و مهاجرت را پوشش می‌دهد.

### مفاهیم اساسی

**مدل‌های سرویس:**

| مدل | چه چیزی می‌دهی | چه چیزی می‌گیری | مثال |
|---|---|---|---|
| **IaaS** | VM، شبکه، storage | OS و بالاتر | AWS EC2، Azure VM |
| **PaaS** | پلتفرم اجرا | کد و داده | App Service، Heroku |
| **SaaS** | نرم‌افزار کامل | فقط استفاده | Microsoft 365، Salesforce |
| **FaaS/Serverless** | تابع | کد | Lambda، Functions |

**مدل مسئولیت مشترک (Shared Responsibility):**

```
IaaS:  شما → OS, runtime, app, data
       provider → physical, hypervisor

PaaS:  شما → app, data
       provider → OS, runtime, hypervisor

SaaS:  شما → data, access
       provider → همه چیز دیگر
```

**نکتهٔ حیاتی:** در ابر، **داده همیشه مسئولیت شماست**. provider از سخت‌افزار محافظت می‌کند، نه از پیکربندی اشتباه شما.

**مدل‌های استقرار:**

| مدل | توضیح |
|---|---|
| **Public** | provider مشترک |
| **Private** | اختصاصی سازمان |
| **Hybrid** | ترکیب |
| **Multi-cloud** | چند provider |

**مزایای ابر:**

- **Elasticity** — مقیاس سریع.
- **Pay-as-you-go** — هزینهٔ متغیر.
- **Global reach** — منطقهٔ جغرافیایی.
- **Managed services** — کاهش بار عملیاتی.
- **DR** — سایت دوم ارزان‌تر.

**معایب و ریسک‌ها:**

| ریسک | توضیح |
|---|---|
| **هزینهٔ غیرقابل پیش‌بینی** | egress، storage، API call |
| **vendor lock-in** | سرویس‌های اختصاصی |
| **latency** | فاصلهٔ شبکه |
| **انطباق** | محل داده، مقررات |
| **مهارت** | تیم باید یاد بگیرد |
| **امنیت پیکربندی** | bucket عمومی، IAM اشتباه |

**نکتهٔ مهم:** بزرگ‌ترین ریسک ابر، **پیکربندی اشتباه** است، نه خود provider. بیشتر نشت‌های دادهٔ ابری از bucket عمومی و IAM بیش‌از‌حد باز آمده‌اند.

**مدل‌های هزینه:**

| مدل | توضیح | مناسب |
|---|---|---|
| **On-demand** | پرداخت ساعتی/ثانیه‌ای | بار متغیر |
| **Reserved** | تعهد ۱–۳ ساله، تخفیف | بار پایدار |
| **Spot** | ظرفیت مازاد، تخفیف زیاد، قابل قطع | Batch، CI |
| **Savings plan** | تعهد مصرف | انعطاف |

**نکته:** برای بار پایدار، reserved ۳۰–۶۰٪ ارزان‌تر است. برای batch، spot.

**مهاجرت به ابر — استراتژی‌های ۶R:**

| استراتژی | توضیح | تلاش |
|---|---|---|
| **Rehost** (lift & shift) | همان‌طور منتقل کن | کم |
| **Replatform** | کمی بهینه (مثلاً managed DB) | متوسط |
| **Repurchase** | جایگزین با SaaS | متوسط |
| **Refactor** | بازنویسی برای ابر | زیاد |
| **Retire** | حذف (دیگر لازم نیست) | کم |
| **Retain** | نگه دار on-prem | — |

**نکتهٔ حیاتی:** **Retire** را جدی بگیر. بسیاری از سازمان‌ها ۱۰–۲۰٪ اپلیکیشن‌هایی دارند که کسی استفاده نمی‌کند. اول حذف کن، بعد مهاجرت.

**مهاجرت داده به ابر:**

| روش | مناسب برای |
|---|---|
| **Over the network** (VPN/Direct Connect) | حجم متوسط، زمان‌بندی |
| **Physical transfer** (Snowball، Data Box) | حجم بسیار زیاد |
| **Replication** (database) | دیتابیس با downtime کم |
| **Hybrid** | ترکیب |

**نکتهٔ محاسبه:** ۱۰TB روی لینک ۱Gbps:
```
۱۰ × ۱۰۲۴ × ۸ ÷ ۱۰۰۰ = ۸۱٬۹۲۰ ثانیه ≈ ۲۲.۸ ساعت (تئوری)
در عمل با overhead: ۳۰–۴۰ ساعت
```
اگر لینک ۱۰۰Mbps باشد: ~۱۰ روز. **انتقال فیزیکی منطقی‌تر است.**

**Cloud migration — مراحل:**

```
۱. Assessment (کشف، وابستگی، TCO)
۲. Planning (6R، ترتیب، rollback)
۳. Landing zone (شبکه، IAM، logging)
۴. Pilot (یک اپلیکیشن کم‌ریسک)
۵. Migration waves
۶. Optimization (rightsizing، cost)
۷. Operations (monitoring، FinOps)
```

**نکته:** **Landing zone** قبل از مهاجرت. بدون شبکه، IAM و logging درست، مهاجرت به یک آشفتگی امنیتی می‌رسد.

### مثال‌های عملی

**Lab Scenario 1 — تصمیم مهاجرت**

> **وظیفه:** برای هر workload، استراتژی 6R را انتخاب کن.
>
> الف) یک اپلیکیشن .NET قدیمی که کسی استفاده نمی‌کند
> ب) یک وب‌سایت PHP با ترافیک متغیر
> ج) یک دیتابیس Oracle حیاتی با latency حساس
> د) ایمیل سازمانی
>
> **پاسخ:**
>
> | مورد | استراتژی | دلیل |
> |---|---|---|
> | الف | **Retire** | حذف، هزینه صفر |
> | ب | **Rehost** یا **Replatform** | سریع، مقیاس‌پذیر |
> | ج | **Retain** (یا Rehost با bare-metal) | latency و لایسنس |
> | د | **Repurchase** (Microsoft 365) | SaaS بالغ |
>
> **درس:** مهاجرت، «همه چیز به ابر» نیست؛ انتخاب است.

**Lab Scenario 2 — هزینهٔ ابر غیرمنتظره**

> **وضعیت:** صورتحساب ماهانه از ۵٬۰۰۰ به ۱۸٬۰۰۰ دلار رسیده.
>
> **وظیفه:** پنج علت رایج.
>
> **پاسخ:**
> ۱. **Egress** — انتقال داده به بیرون (CDN، کاربران).
> ۲. **Storage** — snapshotهای قدیمی، دیسک‌های orphaned.
> ۳. **Idle resources** — VMهای خاموش اما تخصیص‌یافته (EIP، دیسک).
> ۴. **Over-provisioning** — instance بزرگ‌تر از نیاز.
> ۵. **Logging/monitoring** — نگهداشت لاگ بی‌پایان.
>
> **اقدام:** tagging برای مالکیت، budget alert، rightsizing ماهانه، lifecycle policy برای storage.
>
> **درس:** در ابر، هزینه یک متریک فنی است.

### مقایسه و الگوهای مختلف

**مقایسهٔ IaaS، PaaS، SaaS:**

| معیار | IaaS | PaaS | SaaS |
|---|---|---|---|
| کنترل | بالا | متوسط | کم |
| بار عملیاتی | بالا | متوسط | کم |
| انعطاف | بالا | متوسط | کم |
| lock-in | کم | متوسط | بالا |
| مناسب | legacy، کنترل کامل | اپ جدید | نرم‌افزار استاندارد |

**مقایسهٔ on-prem و cloud:**

| معیار | on-prem | cloud |
|---|---|---|
| هزینهٔ اولیه | بالا (CAPEX) | کم (OPEX) |
| هزینهٔ بلندمدت | پایین‌تر برای بار پایدار | متغیر |
| انعطاف | کم | بالا |
| کنترل | کامل | محدود |
| امنیت | مسئولیت کامل شما | shared |
| مناسب | بار پایدار، داده حساس | بار متغیر، رشد سریع |

**مقایسهٔ Reserved و Spot:**

| معیار | Reserved | Spot |
|---|---|---|
| تخفیف | ۳۰–۶۰٪ | ۶۰–۹۰٪ |
| قطعی | ندارد | ممکن (۲ دقیقه اخطار) |
| مناسب | بار پایدار | batch، CI، stateless |

### اشکال شایع

**۱. lift & shift بدون بهینه‌سازی.**
هزینهٔ ابر بیشتر از on-prem می‌شود.

**۲. بدون landing zone.**
IAM باز، بدون logging، شبکهٔ تخت.

**۳. bucket عمومی.**
بزرگ‌ترین علت نشت دادهٔ ابری.

**۴. بدون tagging.**
نمی‌دانی هزینه از کجاست.

**۵. نادیده گرفتن egress.**
انتقال داده به بیرون، گران است.

**۶. VM خاموش اما هزینه‌دار.**
دیسک و IP همچنان پول می‌گیرند.

**۷. فرض اینکه provider backup می‌گیرد.**
مسئولیت داده با شماست.

**۸. مهاجرت بدون rollback.**
اگر شکست بخورد، برگشت سخت است.

### نکات مهم (Tips & Tricks)

- **اول Retire، بعد Migrate.**
- **Landing zone قبل از هر چیز.**
- **Tagging از روز اول** — مالک، محیط، پروژه.
- **Budget alert** در ۵۰٪، ۸۰٪، ۱۰۰٪.
- **Lifecycle policy** برای storage و snapshot.
- **در آزمون:** کلیدواژهٔ «shared responsibility» → **customer always owns data**؛ «unpredictable cost» → **egress/over-provisioning**.

### خلاصه فصل

- مدل‌ها: **IaaS، PaaS، SaaS، FaaS**.
- **Shared responsibility**: داده همیشه با شماست.
- استراتژی‌ها: **6R** (Rehost، Replatform، Repurchase، Refactor، Retire، Retain).
- **Retire** را جدی بگیر.
- **Landing zone** قبل از مهاجرت.
- هزینه در ابر یک **متریک فنی** است.

---

**بعدی:** [فایل ۱۰: اسکریپت‌نویسی و خودکارسازی](10-Scripting-and-Automation.md)

---

## منابع و مراجع

- اهداف رسمی Server+ (V5)، دامنهٔ ۲: Server Administration (Virtualization)
- VMware — vSphere Documentation، vSAN Design Guide
- Microsoft Learn — Hyper-V، Failover Clustering، Azure Architecture Center
- Proxmox VE Documentation: <https://pve.proxmox.com/pve-docs/>
- Kubernetes Documentation: <https://kubernetes.io/docs/>
- NIST SP 800-125 — Security of Full Virtualization
- NIST SP 800-190 — Application Container Security Guide
- NIST SP 800-145 — Definition of Cloud Computing
- AWS/Azure Well-Architected Framework
- <https://www.comptia.org/en-us/certifications/server/>
