# عیب‌یابی سیستم‌عامل و نرم‌افزار

**مدت یادگیری**: ۷ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فصل ۷۸ تا ۸۳ — عیب‌یابی سخت‌افزار
**دامنهٔ آزمون**: Troubleshooting (۲۸٪)

---

## فهرست محتوا

- [فصل ۸۴: شکست بوت](#ch-84)
- [فصل ۸۵: BSOD، kernel panic و توقف سیستم](#ch-85)
- [فصل ۸۶: افت عملکرد سیستم‌عامل](#ch-86)
- [فصل ۸۷: شکست وصله و به‌روزرسانی](#ch-87)
- [فصل ۸۸: خرابی سرویس و اپلیکیشن](#ch-88)

---

## فصل ۸۴: شکست بوت

### مقدمه

بوت، زنجیره‌ای از مراحل است. اگر یکی بشکند، سیستم بالا نمی‌آید — و **محل شکست**، تشخیص را تعیین می‌کند.

این فصل زنجیره را طی می‌کند: از POST تا login.

### مفاهیم اساسی

**زنجیرهٔ بوت:**

```
[POST] → [BIOS/UEFI] → [boot device] → [bootloader] → [kernel] → [init/systemd] → [login]
```

**هر مرحله، علامت متفاوتی دارد:**

| توقف در | علامت |
|---|---|
| **POST** | بوق، LED، صفحهٔ سیاه |
| **BIOS/UEFI** | «no boot device» |
| **bootloader** | GRUB rescue، «Bootmgr is missing» |
| **kernel** | panic، صفحهٔ سیاه بعد از GRUB |
| **init** | «emergency mode»، «failed to mount» |
| **login** | سرویس‌ها بالا نمی‌آیند |

**نکتهٔ حیاتی:** **کدام مرحله** را تشخیص بده — این نیمی از کار است.

**مراحل بوت لینوکس:**

```
۱. BIOS/UEFI → POST
۲. انتخاب boot device
۳. MBR/GPT → GRUB stage 1
۴. GRUB stage 2 → menu
۵. kernel + initramfs
۶. initramfs: mount root
۷. systemd (PID 1)
۸. targetها (multi-user، graphical)
۹. login
```

**مراحل بوت ویندوز:**

```
۱. BIOS/UEFI → POST
۲. Windows Boot Manager (bootmgr)
۳. winload.exe / winload.efi
۴. kernel (ntoskrnl.exe) + hal
۵. drivers
۶. session manager (smss.exe)
۷. winlogon → login
```

**علل رایج شکست بوت:**

| علت | علامت | رفع |
|---|---|---|
| **boot order اشتباه** | «no boot device» | BIOS |
| **دیسک boot خراب** | «no boot device» | تعویض/بازیابی |
| **MBR/GPT خراب** | GRUB rescue | بازسازی |
| **bootloader خراب** | «Bootmgr missing» | repair |
| **فایل سیستمی خراب** | kernel panic | rescue mode |
| **fstab اشتباه** | emergency mode | اصلاح fstab |
| **درایور مشکل‌دار** | BSOD در بوت | safe mode |
| **آپدیت نیمه‌کاره** | بوت نمی‌شود | rollback |

**نکتهٔ حیاتی:** **آخرین تغییر** را به یاد بیاور. آپدیت، درایور جدید، تغییر پارتیشن — رایج‌ترین علل.

**حالت‌های بازیابی:**

| پلتفرم | حالت |
|---|---|
| **ویندوز** | Safe Mode، WinRE، Startup Repair |
| **لینوکس** | rescue mode، emergency mode، single-user |
| **هر دو** | boot از ISO/USB زنده |

**نکته:** **بوت از رسانهٔ زنده** قدرتمندترین ابزار است — چون سیستم‌عامل خراب را دور می‌زند.

**GRUB rescue:**

```
grub rescue> ls
grub rescue> ls (hd0,gpt2)/
grub rescue> set root=(hd0,gpt2)
grub rescue> set prefix=(hd0,gpt2)/boot/grub
grub rescue> insmod normal
grub rescue> normal
```

**نکته:** بعد از بوت موفق، `update-grub` و `grub-install` را اجرا کن تا دائمی شود.

**بازسازی MBR/GPT:**

```bash
# ببین کدام دیسک
lsblk

# بازسازی GRUB (Debian/Ubuntu)
grub-install /dev/sda
update-grub

# RHEL
grub2-install /dev/sda
grub2-mkconfig -o /boot/grub2/grub.cfg
```

**Windows — Startup Repair:**

```
۱. بوت از ISO ویندوز
۲. Repair your computer
۳. Troubleshoot → Advanced options
۴. Startup Repair
یا:
   bootrec /fixmbr
   bootrec /fixboot
   bootrec /rebuildbcd
```

**نکته:** `bootrec /fixboot` در UEFI ممکن است «Access denied» بدهد. راه‌حل: `bcdboot C:\Windows /s S: /f UEFI`.

**emergency mode در لینوکس:**

علت رایج: **fstab** به دیسکی اشاره می‌کند که mount نمی‌شود.

```
۱. رمز root را وارد کن
۲. journalctl -xb | grep -i fail
۳. cat /etc/fstab
۴. اصلاح یا comment کردن خط مشکل‌دار
۵. mount -a
۶. systemctl reboot
```

**نکتهٔ حیاتی:** در fstab، از `nofail` استفاده کن برای دیسک‌های غیرحیاتی:

```
UUID=xxxx  /data  ext4  defaults,nofail  0 2
```

بدون `nofail`، اگر دیسک نباشد، سیستم در emergency mode می‌ماند.

**initramfs — مشکل رایج:**

اگر kernel پیام «unable to mount root fs» می‌دهد:

- **درایور storage** در initramfs نیست (بعد از تغییر کنترلر).
- **UUID عوض شده** (بعد از clone).
- **LVM/dm-crypt** پیکربندی نشده.

**رفع:**

```bash
# از live CD، chroot
mount /dev/sda2 /mnt
mount --bind /dev /mnt/dev
mount --bind /proc /mnt/proc
mount --bind /sys /mnt/sys
chroot /mnt
update-initramfs -u -k all
# یا
dracut --force
```

**نکته:** این سناریو بعد از **مهاجرت P2V** یا **تعویض کنترلر** بسیار رایج است.

### مثال‌های عملی

**Lab Scenario 1 — «no boot device»**

> **وضعیت:** سرور روشن می‌شود، POST موفق، اما «no boot device found».
>
> **وظیفه:** ترتیب بررسی.
>
> **پاسخ:**
> ۱. **boot order:** آیا دیسک/کنترلر در لیست است؟
> ۲. **تشخیص دیسک:** آیا در BIOS/UEFI دیده می‌شود؟
> ۳. **RAID:** آیا آرایه optimal است؟ (اگر degraded و boot دیسک خراب، بوت نمی‌شود)
> ۴. **کابل/backplane:** آیا دیسک برق دارد؟
> ۵. **boot sector:** آیا MBR/GPT سالم است؟ (با live CD چک کن)
> ۶. **UEFI vs Legacy:** آیا حالت بوت با نصب هم‌خوان است؟
>
> **درس:** «no boot device» می‌تواند از boot order تا دیسک مرده باشد.

**Lab Scenario 2 — بعد از آپدیت، بوت نمی‌شود**

> **وضعیت:** آپدیت kernel نصب شد، ری‌استارت، سیستم بالا نمی‌آید.
>
> **وظیفه:** اقدام.
>
> **پاسخ:**
> ۱. **GRUB:** در منوی GRUB، kernel قبلی را انتخاب کن (`Advanced options`).
> ۲. اگر بوت شد: kernel جدید مشکل دارد (درایور، initramfs).
> ۳. **رفع:** `update-initramfs -u`، یا حذف kernel جدید.
> ۴. **پیشگیری:** `installonly_limit` را بالا ببر تا چند kernel نگه داری.
>
> **درس:** همیشه یک kernel قبلی در GRUB نگه دار.

### مقایسه و الگوهای مختلف

**مقایسهٔ MBR و GPT در بوت:**

| معیار | MBR | GPT |
|---|---|---|
| بوت | BIOS/Legacy | UEFI |
| حداکثر دیسک | ۲TB | نامحدود |
| پارتیشن | ۴ primary | ۱۲۸ |
| backup | ❌ | ✅ |

**مقایسهٔ Safe Mode و WinRE:**

| معیار | Safe Mode | WinRE |
|---|---|---|
| هدف | بوت با درایور حداقلی | تعمیر |
| دسترسی | سیستم زنده | محیط بازیابی |
| مناسب | درایور مشکل‌دار | boot خراب |

**مقایسهٔ rescue و emergency mode:**

| معیار | rescue | emergency |
|---|---|---|
| mount | root read-write | root read-only |
| سرویس | حداقلی | تقریباً هیچ |
| مناسب | تعمیر | تشخیص |

### اشکال شایع

**۱. بوت از دیسک اشتباه.**
USB یا دیسک دوم در boot order.

**۲. UEFI در برابر Legacy.**
نصب UEFI، بوت Legacy.

**۳. fstab بدون nofail.**
emergency mode با دیسک غایب.

**۴. initramfs بدون درایور storage.**
«unable to mount root».

**۵. حذف همهٔ kernelهای قدیمی.**
راه بازگشت نیست.

**۶. bootrec /fixboot در UEFI.**
«Access denied».

**۷. تعویض کنترلر بدون آپدیت initramfs.**
بوت نمی‌شود.

**۸. بدون رسانهٔ بازیابی.**
وقتی لازم است، نداری.

### نکات مهم (Tips & Tricks)

- **کدام مرحله؟** اول تشخیص بده.
- **رسانهٔ زنده** همیشه در دسترس.
- **kernel قبلی** در GRUB نگه دار.
- **`nofail`** در fstab برای دیسک غیرحیاتی.
- **`update-initramfs`** بعد از تغییر storage.
- **در آزمون:** کلیدواژهٔ «unable to mount root fs» → **initramfs/storage driver**.

### خلاصه فصل

- زنجیرهٔ بوت: **POST → BIOS → boot device → bootloader → kernel → init → login**.
- **محل توقف** = نیمی از تشخیص.
- **GRUB rescue** و **Startup Repair** ابزارهای اصلی.
- **fstab بدون nofail** = emergency mode.
- **initramfs** بعد از تغییر storage لازم است.
- همیشه **kernel/نسخهٔ قبلی** نگه دار.

---

## فصل ۸۵: BSOD، kernel panic و توقف سیستم

### مقدمه

BSOD (Blue Screen of Death) و kernel panic، «توقف کنترل‌شده» هستند: سیستم‌عامل ترجیح می‌دهد متوقف شود تا داده را خراب کند.

این توقف، **پیام** دارد — و پیام، کلید تشخیص است.

### مفاهیم اساسی

**BSOD — ساختار:**

```
STOP CODE: 0x0000007B (INACCESSIBLE_BOOT_DEVICE)
│
├── کد اصلی (کلید جست‌وجو)
├── پارامترها (۱–۴)
└── نام نمادین (در ویندوز جدید)
```

**نکتهٔ حیاتی:** **کد توقف** را ثبت کن. `0x0000007B` با `0x0000001A` کاملاً متفاوت است.

**کدهای رایج BSOD:**

| کد | نام | علت رایج |
|---|---|---|
| **0x0000007B** | INACCESSIBLE_BOOT_DEVICE | درایور storage، تغییر کنترلر |
| **0x0000001A** | MEMORY_MANAGEMENT | حافظه |
| **0x00000050** | PAGE_FAULT_IN_NONPAGED_AREA | حافظه، درایور |
| **0x000000D1** | DRIVER_IRQL_NOT_LESS_OR_EQUAL | درایور |
| **0x0000003B** | SYSTEM_SERVICE_EXCEPTION | درایور، گرافیک |
| **0x0000007E** | SYSTEM_THREAD_EXCEPTION_NOT_HANDLED | درایور |
| **0x000000EF** | CRITICAL_PROCESS_DIED | فایل سیستمی |
| **0x00000024** | NTFS_FILE_SYSTEM | دیسک، فایل‌سیستم |
| **0x0000009F** | DRIVER_POWER_STATE_FAILURE | درایور، power |
| **0xC000021A** | STATUS_SYSTEM_PROCESS_TERMINATED | فایل سیستمی |

**نکته:** در ویندوز سرور، BSOD پیش‌فرض **ری‌استارت خودکار** می‌کند و پیام دیده نمی‌شود. برای دیدن: `System Properties → Advanced → Startup and Recovery → Automatically restart` را بردار.

**Kernel panic — ساختار:**

```
Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
│
├── نوع panic
└── پیام
```

**پیام‌های رایج:**

| پیام | علت |
|---|---|
| **Unable to mount root fs** | initramfs، UUID، درایور |
| **Out of memory** | RAM، leak |
| **BUG: soft lockup** | CPU hang، درایور |
| **general protection fault** | درایور، سخت‌افزار |
| **Machine Check Exception** | سخت‌افزار (CPU، RAM، PCIe) |
| **Kernel panic — attempted to kill init** | فایل سیستمی خراب |

**نکتهٔ حیاتی:** **Machine Check Exception (MCE)** یعنی سخت‌افزار مشکل دارد — نه نرم‌افزار. باید BMC log و dmesg را چک کنی.

**ثبت panic — kdump:**

```bash
# نصب
dnf install kexec-tools

# فعال
systemctl enable --now kdump

# تنظیم حافظه
# /etc/kdump.conf
# crashkernel=256M

# بررسی
cat /proc/cmdline | grep crashkernel
ls /var/crash/
```

**نکته:** kdump یک kernel دوم را در حافظهٔ رزروشده بالا می‌آورد و از آن برای ذخیرهٔ memory dump استفاده می‌کند. بدون آن، بعد از panic چیزی نداری.

**ویندوز — memory dump:**

| نوع | محتوا | محل |
|---|---|---|
| **Small (minidump)** | حداقلی | `C:\Windows\Minidump` |
| **Kernel** | kernel memory | `C:\Windows\MEMORY.DMP` |
| **Complete** | کل RAM | `C:\Windows\MEMORY.DMP` |

**ابزار تحلیل:** **WinDbg** با `!analyze -v`.

**نکته:** minidump برای بیشتر موارد کافی است و حجم کمی دارد.

**علل رایج توقف:**

| دسته | علت |
|---|---|
| **حافظه** | ماژول معیوب، ناسازگاری |
| **درایور** | نسخهٔ ناسازگار، باگ |
| **دیسک** | bad sector، کابل، کنترلر |
| **برق** | ناپایداری |
| **حرارت** | throttle، خاموشی |
| **نرم‌افزار** | آنتی‌ویروس، درایور سطح پایین |
| **firmware** | باگ BIOS/کنترلر |

**نکته:** اگر BSOD **بعد از نصب یک نرم‌افزار** شروع شد، همان نرم‌افزار (معمولاً درایور آن) مقصر است.

**Safe Mode — چرا کار می‌کند؟**

Safe Mode فقط درایورهای ضروری را بار می‌کند. اگر در Safe Mode مشکل نیست، **درایور شخص ثالث** مقصر است.

**نکته:** این یک **آزمون تفکیک** عالی است: Safe Mode سالم = درایور.

### مثال‌های عملی

**Lab Scenario 1 — BSOD بعد از تعویض مادربرد**

> **وضعیت:** مادربرد تعویض شد. ویندوز با `0x0000007B INACCESSIBLE_BOOT_DEVICE` بوت نمی‌شود.
>
> **وظیفه:** علت و رفع.
>
> **پاسخ:**
> - **علت:** کنترلر storage جدید، درایور قدیمی را ندارد (یا حالت AHCI/RAID عوض شده).
> - **رفع:**
>   ۱. در BIOS، حالت storage را به همان قبلی برگردان (AHCI/RAID).
>   ۲. یا در WinRE، درایور storage را load کن (`pnputil /add-driver`).
>   ۳. یا `DISM /Image:C:\ /Add-Driver`.
>
> **درس:** تغییر سخت‌افزار storage، درایور می‌خواهد.

**Lab Scenario 2 — panic تصادفی**

> **وضعیت:** سرور لینوکس هر چند روز یک بار panic می‌کند. پیام: `Machine Check Exception`.
>
> **وظیفه:** تحلیل.
>
> **پاسخ:**
> - **MCE = سخت‌افزار.** CPU، RAM، PCIe، یا حرارت.
> - **بررسی:** `mcelog`، `dmesg | grep -i mce`، BMC log (correctable/uncorrectable).
> - **اقدام:** Memtest86+، بررسی دما، بررسی firmware، تعویض قطعهٔ مشکوک.
> - **نکته:** MCE را با باگ نرم‌افزاری اشتباه نگیر.
>
> **درس:** MCE یعنی سخت‌افزار، نه OS.

### مقایسه و الگوهای مختلف

**مقایسهٔ BSOD و panic:**

| معیار | BSOD | panic |
|---|---|---|
| پلتفرم | ویندوز | لینوکس/یونیکس |
| کد | STOP code | پیام متنی |
| dump | MEMORY.DMP | kdump/vmcore |
| ابزار | WinDbg | crash، drgn |

**مقایسهٔ minidump و full dump:**

| معیار | Minidump | Full |
|---|---|---|
| حجم | کوچک | بزرگ (اندازهٔ RAM) |
| جزئیات | کافی برای بیشتر | کامل |
| مناسب | تولیدی | تحلیل عمیق |

**مقایسهٔ Safe Mode و Last Known Good:**

| معیار | Safe Mode | Last Known Good |
|---|---|---|
| درایور | حداقلی | قبلی |
| مناسب | درایور جدید خراب | تنظیمات خراب |

### اشکال شایع

**۱. نادیده گرفتن کد STOP.**
بدون کد، تشخیص حدس است.

**۲. ری‌استارت خودکار.**
پیام دیده نمی‌شود.

**۳. بدون kdump/dump.**
بعد از panic، چیزی نداری.

**۴. فرض نرم‌افزاری برای MCE.**
MCE سخت‌افزاری است.

**۵. نصب درایور beta در تولید.**
BSOD.

**۶. تحلیل dump بدون نمادها.**
WinDbg بدون symbol، بی‌فایده است.

**۷. تعویض RAM بدون تست.**
ممکن است مشکل در جای دیگر باشد.

**۸. نادیده گرفتن الگو.**
BSOD در ساعت خاص = backup یا scheduled task.

### نکات مهم (Tips & Tricks)

- **کد STOP را ثبت کن.**
- **ری‌استارت خودکار را غیرفعال کن** تا پیام را ببینی.
- **kdump** را فعال کن.
- **WinDbg با `!analyze -v`**.
- **Safe Mode** برای تفکیک درایور.
- **در آزمون:** کلیدواژهٔ «0x0000007B» → **storage driver/boot device**.

### خلاصه فصل

- BSOD/panic = توقف کنترل‌شده با پیام.
- **کد STOP** کلید تشخیص است.
- **MCE** = سخت‌افزار.
- **kdump** و **memory dump** برای تحلیل.
- **Safe Mode** درایور شخص ثالث را تفکیک می‌کند.
- ری‌استارت خودکار را غیرفعال کن تا پیام ببینی.

---

## فصل ۸۶: افت عملکرد سیستم‌عامل

### مقدمه

«سرور کند است» شایع‌ترین شکایت است و مبهم‌ترین. این فصل، آن را به **چهار منبع قابل اندازه‌گیری** تجزیه می‌کند.

### مفاهیم اساسی

**چهار منبع — بازبینی فصل ۳۴:**

| منبع | متریک | آستانه |
|---|---|---|
| **CPU** | utilization، load، run queue | >۸۰٪ پایدار |
| **RAM** | available، swap، page fault | <۱۰٪ available |
| **Disk** | latency، queue، IOPS | >۲۰ms (HDD) |
| **Network** | throughput، loss، retransmit | >۱٪ loss |

**نکتهٔ حیاتی:** «کندی» را **قابل اندازه‌گیری** کن. «کند است» → «باز کردن فایل ۴۵ ثانیه طول می‌کشد، قبلاً ۳ ثانیه بود».

**علل رایج — لینوکس:**

| علامت | علت |
|---|---|
| **load بالا، CPU پایین** | I/O wait |
| **swap فعال** | کمبود RAM |
| **`%util` بالا، `await` بالا** | دیسک اشباع |
| **`%sy` بالا** | syscall، درایور |
| **`%st` بالا (VM)** | steal time، host شلوغ |
| **proces D state** | uninterruptible sleep (I/O) |

**علل رایج — ویندوز:**

| علامت | علت |
|---|---|
| **`% Privileged Time` بالا** | درایور |
| **`Pages/sec` بالا** | کمبود RAM |
| **`Avg. Disk Queue Length` بالا** | دیسک اشباع |
| **`Context Switches/sec` بالا** | overhead |
| **`% Processor Time` بالا با process کم** | درایور/kerne |

**نکتهٔ مهم:** در VM، **steal time** (`%st`) را چک کن. اگر بالا باشد، مشکل در **host** است، نه VM.

**روش عیب‌یابی — گام‌به‌گام:**

```
۱. تعریف مشکل (قابل اندازهگیری)
۲. baseline (قبلاً چطور بود؟)
۳. نمای کلی (vmstat/perfmon)
۴. منبع گلوگاه
۵. پروسهٔ مقصر (top/iotop/Resource Monitor)
۶. علت (پیکربندی؟ داده؟ باگ؟)
۷. رفع و تأیید
```

**نکتهٔ حیاتی:** **baseline** بدون آن، نمی‌دانی «کند» یعنی چه. عدد فعلی بدون مرجع، بی‌معناست.

**علل نرم‌افزاری رایج:**

| علت | علامت |
|---|---|
| **آنتی‌ویروس scan** | I/O و CPU بالا در ساعت خاص |
| **backup در حال اجرا** | I/O بالا |
| **ایندکس‌گذاری** | I/O و CPU |
| **حافظهٔ leak** | RAM تدریجاً پر می‌شود |
| **کوئری بدون ایندکس** | دیتابیس کند |
| **log level = debug** | I/O و دیسک |
| **کلاستر/ replication lag** | I/O و شبکه |
| **اسکریپت بی‌بازده** | CPU |

**نکته:** **الگوی زمانی** را ببین. اگر کندی هر شب ساعت ۲ است، backup است. اگر هر ساعت است، scheduled task.

**حافظهٔ leak — تشخیص:**

```
RAM مصرفی یک پروسه، با گذشت زمان رشد میکند و آزاد نمیشود
```

**ابزار:** `ps aux --sort=-%mem`، `pmap`، `valgrind` (توسعه)، Performance Monitor (ویندوز).

**نکته:** restart موقت، راه‌حل است نه رفع. باید به vendor گزارش شود.

**I/O wait — تشخیص:**

```bash
iostat -xz 1
# await بالا، %util بالا
iotop -o
# کدام پروسه؟
```

**نکته:** اگر `%iowait` بالا و `%util` پایین است، ممکن است مشکل در **شبکهٔ ذخیره‌سازی** (NFS/iSCSI) باشد، نه دیسک محلی.

**Network — تشخیص:**

```bash
sar -n DEV 1
ss -s
ip -s link
```

**علل:** duplex mismatch، MTU، کابل، سوئیچ اشباع، retransmission.

### مثال‌های عملی

**Lab Scenario 1 — سرور هر شب کند می‌شود**

> **وضعیت:** کاربران می‌گویند هر شب ساعت ۲–۳ سرور خیلی کند است.
>
> **وظیفه:** علت.
>
> **پاسخ:**
> ۱. **الگوی زمانی = scheduled task.**
> ۲. **بررسی:** crontab، Task Scheduler، backup schedule، آنتی‌ویروس.
> ۳. **ابزار:** `sar -f /var/log/sa/saXX` (تاریخی)، Performance Monitor.
> ۴. **رفع:** انتقال backup به ساعت کم‌بار، محدود کردن I/O، یا افزایش پنجره.
>
> **درس:** الگوی زمانی، مقصر را لو می‌دهد.

**Lab Scenario 2 — VM کند، host سالم**

> **وضعیت:** VM کند است. داخل VM: CPU ۳۰٪، RAM ۵۰٪، disk latency ۵ms. host: CPU ۸۵٪.
>
> **وظیفه:** تحلیل.
>
> **پاسخ:**
> - داخل VM همه چیز نرمال است، اما host شلوغ.
> - **`%st` (steal time)** را چک کن — اگر بالا، VM منتظر CPU است.
> - **علت:** overcommit CPU در host، یا VM دیگر.
> - **رفع:** کاهش overcommit، افزودن host، یا محدود کردن VM پرترافیک.
>
> **درس:** در VM، همیشه از host هم نگاه کن.

### مقایسه و الگوهای مختلف

**مقایسهٔ top و vmstat:**

| معیار | top | vmstat |
|---|---|---|
| نمای کلی | ✅ | ✅ |
| I/O wait | ✅ | ✅ |
| تاریخی | ❌ | ✅ (با sar) |
| مناسب | لحظه‌ای | روند |

**مقایسهٔ iostat و iotop:**

| معیار | iostat | iotop |
|---|---|---|
| سطح | دیسک | پروسه |
| مناسب | کدام دیسک | کدام پروسه |

**مقایسهٔ کمبود RAM و leak:**

| معیار | کمبود | Leak |
|---|---|---|
| الگو | ثابت | رشد تدریجی |
| رفع | افزودن RAM | restart + گزارش باگ |

### اشکال شایع

**۱. بدون baseline.**
«کند» تعریف‌نشده.

**۲. فقط CPU.**
گلوگاه دیسک دیده نمی‌شود.

**۳. نادیده گرفتن الگوی زمانی.**
scheduled task پیدا نمی‌شود.

**۴. تفسیر `free` در لینوکس.**
cache طبیعی است.

**۵. نادیده گرفتن steal time.**
مشکل host.

**۶. restart به‌جای رفع leak.**
برمی‌گردد.

**۷. اندازه‌گیری در بازهٔ کوتاه.**
الگوی روزانه دیده نمی‌شود.

**۸. نادیده گرفتن آنتی‌ویروس.**
scan می‌تواند I/O را ببلعد.

### نکات مهم (Tips & Tricks)

- **مشکل را قابل اندازه‌گیری کن.**
- **baseline** بساز.
- **الگوی زمانی** را ببین.
- **`%st`** در VM.
- **`iotop -o`** برای پروسهٔ I/O.
- **در آزمون:** کلیدواژهٔ «slow at same time daily» → **scheduled task/backup**.

### خلاصه فصل

- چهار منبع: **CPU، RAM، Disk، Network**.
- **Baseline** بدون آن، «کند» بی‌معناست.
- **الگوی زمانی** مقصر را لو می‌دهد.
- **steal time** در VM حیاتی است.
- **Leak** با restart رفع نمی‌شود.
- از **کلی به جزئی** برو.

---

## فصل ۸۷: شکست وصله و به‌روزرسانی

### مقدمه

وصله، برای امنیت لازم است — اما خودش می‌تواند منبع خرابی باشد. این فصل دربارهٔ **وقتی وصله شکست می‌خورد**.

### مفاهیم اساسی

**انواع شکست وصله:**

| نوع | علامت |
|---|---|
| **دانلود شکست** | خطای شبکه/فضا |
| **نصب شکست** | خطای مجوز/وابستگی |
| **نصب شد، سیستم بوت نمی‌شود** | درایور، kernel |
| **نصب شد، سرویس خراب** | تغییر رفتار |
| **نصب شد، عملکرد افت** | باگ |
| **نصب نیمه‌کاره** | وضعیت ناسازگار |

**نکتهٔ حیاتی:** **نصب نیمه‌کاره** بدترین حالت است — سیستم در وضعیت نامعلوم. باید با ابزار repair یا rollback حل شود.

**علل رایج شکست (ویندوز):**

| کد خطا | علت |
|---|---|
| **0x80070005** | دسترسی رد شد |
| **0x80070070** | فضای دیسک |
| **0x80073712** | فایل سیستمی خراب |
| **0x8024402C** | مشکل proxy/WSUS |
| **0x800F0922** | پارتیشن system reserved کوچک |
| **0x800F081F** | فایل منبع گم‌شده |

**نکته:** `0x800F0922` یکی از رایج‌ترین‌هاست — پارتیشن **System Reserved** یا **EFI** فضای کافی ندارد. باید بزرگ‌تر شود.

**ابزار تعمیر (ویندوز):**

```powershell
# بررسی فایلهای سیستمی
sfc /scannow

# تعمیر تصویر
DISM /Online /Cleanup-Image /CheckHealth
DISM /Online /Cleanup-Image /ScanHealth
DISM /Online /Cleanup-Image /RestoreHealth

# پاک کردن cache آپدیت
net stop wuauserv
net stop bits
Remove-Item -Recurse -Force C:\Windows\SoftwareDistribution\*
net start wuauserv
net start bits

# حذف آپدیت
wusa /uninstall /kb:5000000
# یا
DISM /Online /Remove-Package /PackageName:Package_for_KB5000000
```

**نکته:** `DISM /RestoreHealth` نیاز به منبع سالم دارد (اینترنت یا ISO). بدون آن، ممکن است شکست بخورد.

**علل رایج شکست (لینوکس):**

| خطا | علت |
|---|---|
| **`dpkg was interrupted`** | نصب نیمه‌کاره |
| **`unmet dependencies`** | وابستگی |
| **`No space left`** | فضای `/boot` یا `/` |
| **`GPG error`** | کلید منقضی |
| **`held packages`** | بستهٔ نگه‌داشته‌شده |
| **`file conflicts`** | بستهٔ third-party |

**رفع:**

```bash
# نصب نیمهکاره
dpkg --configure -a
apt --fix-broken install

# فضای /boot
df -h /boot
apt autoremove --purge

# کلید GPG
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys <KEY>

# RHEL
dnf clean all
rpm --rebuilddb
```

**نکتهٔ حیاتی:** **`/boot` پر شدن** یکی از رایج‌ترین مشکلات لینوکس است. kernelهای قدیمی جمع می‌شوند و آپدیت بعدی شکست می‌خورد.

**رفع:**

```bash
# Ubuntu
apt autoremove --purge

# RHEL
dnf remove --oldinstallonly --setopt installonly_limit=2 kernel
```

**Rollback:**

| پلتفرم | روش |
|---|---|
| **ویندوز** | `wusa /uninstall`، System Restore |
| **لینوکس** | `dnf downgrade`، `apt install pkg=version` |
| **هر دو** | snapshot قبل از patch |
| **firmware** | نسخهٔ قبلی (اگر ممکن) |

**نکته:** بعضی وصله‌ها uninstall نمی‌شوند (مخصوصاً rollupهای بزرگ). برای آن‌ها، **snapshot** تنها rollback است.

**پس از شکست — رویه:**

```
۱. توقف استقرار (اگر مرحلهای بود)
۲. تشخیص (لاگ، کد خطا)
۳. Rollback فوری
۴. ریشهیابی (چرا شکست؟)
۵. Exclusion (این وصله را مسدود کن)
۶. گزارش به vendor
۷. مستندسازی
```

**نکته:** **Exclusion** مهم است. اگر وصله را مسدود نکنی، بار بعد هم شکست می‌خورد.

**پیشگیری:**

| اقدام | دلیل |
|---|---|
| **pilot** | کشف زودهنگام |
| **snapshot** | rollback |
| **فضای کافی** | جلوگیری از شکست |
| **تست در staging** | کشف ناسازگاری |
| **پنجرهٔ تغییر** | زمان برای رفع |
| **backup** | آخرین دفاع |

### مثال‌های عملی

**Lab Scenario 1 — آپدیت ویندوز شکست می‌خورد**

> **وضعیت:** آپدیت با `0x800F0922` شکست می‌خورد.
>
> **وظیفه:** علت و رفع.
>
> **پاسخ:**
> - **علت:** پارتیشن System Reserved (یا EFI) فضای کافی ندارد (معمولاً <۵۰۰MB).
> - **رفع:**
>   ۱. بررسی فضا: `Get-Partition`
>   ۲. اگر EFI کوچک است: گسترش با `diskpart` (نیاز به فضای مجاور)
>   ۳. یا حذف فایل‌های قدیمی log در پارتیشن
>   ۴. `DISM /RestoreHealth` و تلاش مجدد
>
> **درس:** کد خطا، مسیر را نشان می‌دهد.

**Lab Scenario 2 — آپدیت لینوکس شکست**

> **وضعیت:** `apt upgrade` با `No space left on device` شکست می‌خورد.
>
> **وظیفه:** اقدام.
>
> **پاسخ:**
> ۱. **بررسی:** `df -h` — کدام پارتیشن؟
> ۲. **اغلب `/boot`:** kernelهای قدیمی.
> ۳. **رفع:** `apt autoremove --purge`، حذف kernelهای قدیمی.
> ۴. **اگر `/` پر است:** `du -xh / --max-depth=1 | sort -h`، پاک‌سازی cache و log.
> ۵. **تکرار آپدیت.**
>
> **درس:** `/boot` جدا و با فضای کافی طراحی کن (۱GB).

### مقایسه و الگوهای مختلف

**مقایسهٔ SFC و DISM:**

| معیار | SFC | DISM |
|---|---|---|
| دامنه | فایل‌های سیستمی | تصویر ویندوز |
| منبع | cache محلی | اینترنت/ISO |
| قدرت | متوسط | بالا |
| ترتیب | اول SFC | بعد DISM |

**نکته:** اگر SFC شکست خورد، DISM. اگر DISM شکست خورد، منبع ISO.

**مقایسهٔ rollback و snapshot:**

| معیار | Rollback | Snapshot |
|---|---|---|
| سرعت | متوسط | سریع |
| دامنه | وصله | کل سیستم |
| وابستگی | vendor | hypervisor |
| مناسب | وصلهٔ منفرد | تغییر بزرگ |

**مقایسهٔ apt و dnf در rollback:**

| معیار | apt | dnf |
|---|---|---|
| نسخهٔ قبلی | `apt install pkg=v` | `dnf downgrade pkg` |
| تاریخ | `history` | `history` |
| undo | `apt history` | `dnf history undo` |

### اشکال شایع

**۱. آپدیت بدون فضای کافی.**
شکست یا نصب نیمه‌کاره.

**۲. بدون snapshot.**
rollback ممکن نیست.

**۳. آپدیت همه سرورها هم‌زمان.**
اگر باگ داشته باشد، همه خراب.

**۴. نادیده گرفتن `/boot`.**
kernelهای قدیمی، فضا را پر می‌کنند.

**۵. حذف SoftwareDistribution بدون توقف سرویس.**
فایل قفل است.

**۶. SFC قبل از DISM.**
ترتیب اشتباه (اول SFC، بعد DISM).

**۷. بدون exclusion بعد از شکست.**
تکرار شکست.

**۸. آپدیت در ساعت اوج.**
قطعی در بدترین زمان.

### نکات مهم (Tips & Tricks)

- **کد خطا** را جست‌وجو کن — راه‌حل معمولاً مستند است.
- **snapshot** قبل از patch بزرگ.
- **`/boot` ۱GB** و `autoremove` دورهای.
- **pilot** قبل از گسترش.
- **exclusion** بعد از شکست.
- **در آزمون:** کلیدواژهٔ «update fails, disk space» → **insufficient space on system partition**.

### خلاصه فصل

- شکست وصله: **دانلود، نصب، بوت، سرویس، عملکرد**.
- **کد خطا** مسیر را نشان می‌دهد.
- **SFC → DISM → ISO** ترتیب تعمیر.
- **`/boot` پر** = شکست رایج لینوکس.
- **snapshot** تنها rollback برای rollup.
- **exclusion** بعد از شکست.

---

## فصل ۸۸: خرابی سرویس و اپلیکیشن

### مقدمه

سرویس‌ها، لایه‌ای هستند که کاربر واقعاً می‌بیند. و خرابی‌شان می‌تواند از پنج لایهٔ متفاوت بیاید: پیکربندی، مجوز، وابستگی، منابع، یا کد.

این فصل، رویکرد نظام‌مند به خرابی سرویس است.

### مفاهیم اساسی

**دسته‌بندی خرابی سرویس:**

| دسته | علامت | مثال |
|---|---|---|
| **شروع نمی‌شود** | `failed` در systemctl | پورت اشغال، config غلط |
| **شروع می‌شود و می‌میرد** | restart loop | وابستگی، مجوز |
| **بالاست اما پاسخ نمی‌دهد** | timeout | deadlock، منابع |
| **پاسخ می‌دهد اما خطا** | ۵xx | کد، دیتابیس |
| **کند است** | latency بالا | منابع، کوئری |

**نکتهٔ حیاتی:** این پنج دسته، **مسیر عیب‌یابی متفاوتی** دارند. اول دسته را تعیین کن.

**ترتیب بررسی — «پنج لایه»:**

```
۱. آیا پروسه اجراست؟      → systemctl status / ps
۲. آیا پورت گوش میدهد؟   → ss -tulpn
۳. آیا به وابستگی وصل است؟ → دیتابیس، cache، API
۴. آیا مجوز دارد؟         → فایل، شبکه، SELinux
۵. آیا منابع دارد؟        → CPU، RAM، fd، connection
```

**نکته:** این ترتیب، از **سطحی به عمیق** است. هر لایه، ابزار خودش را دارد.

**علل رایج — شروع نمی‌شود:**

| علت | بررسی |
|---|---|
| **پورت اشغال** | `ss -tulpn \| grep :PORT` |
| **config غلط** | `nginx -t`، `sshd -t` |
| **فایل گم‌شده** | `ls -l` روی مسیر config |
| **مجوز** | `namei -l /path` |
| **کاربر وجود ندارد** | `id myapp` |
| **وابستگی** | `systemctl list-dependencies` |
| **SELinux** | `ausearch -m avc -ts recent` |

**نکتهٔ حیاتی:** **SELinux** یکی از پنهان‌ترین علل است. سرویس بالا نمی‌آید، لاگ systemd پیام کلی می‌دهد، اما `audit.log` می‌گوید `denied`.

**علل رایج — پاسخ نمی‌دهد:**

| علت | بررسی |
|---|---|
| **Deadlock** | stack trace، `gdb` |
| **Thread pool تمام** | متریک‌های اپ |
| **Connection pool تمام** | متریک دیتابیس |
| **file descriptor تمام** | `ulimit -n`، `lsof \| wc -l` |
| **حافظه** | OOM killer در `dmesg` |
| **DNS** | resolve نام داخلی |
| **فایروال** | پورت بین سرویس‌ها |

**نکته:** **file descriptor** محدودیت پیش‌فرض (۱۰۲۴) برای سرورهای پرترافیک کم است. `ulimit -n 65536` در unit file.

**OOM Killer — تشخیص:**

```bash
dmesg | grep -i "killed process"
journalctl -k | grep -i oom
```

**نکته:** OOM killer پروسهٔ با بیشترین مصرف را می‌کشد — که ممکن است سرویس اصلی نباشد. اگر سرویس شما مرتب می‌میرد، OOM را چک کن.

**Log — بهترین دوست:**

```bash
journalctl -xeu myapp --since "1 hour ago"
journalctl -u myapp -f
tail -f /var/log/myapp/error.log
```

**ویندوز:**

```powershell
Get-EventLog -LogName Application -Source "MyApp" -Newest 50
Get-WinEvent -FilterHashtable @{LogName='Application'; Level=2} -MaxEvents 50
```

**نکته:** در ویندوز، **Event ID** و **Source** را فیلتر کن — نه کل لاگ.

**وابستگی‌ها — منبع خرابی زنجیره‌ای:**

```
[اپلیکیشن] → [دیتابیس] → [DNS] → [شبکه]
```

**نکته:** اگر اپلیکیشن خطا می‌دهد، **وابستگی‌ها** را قبل از کد چک کن. ۵۰٪ خرابی‌های اپلیکیشن، از وابستگی است.

**Health check و readiness:**

| نوع | سؤال |
|---|---|
| **Liveness** | آیا پروسه زنده است؟ |
| **Readiness** | آیا آمادهٔ دریافت ترافیک است؟ |
| **Startup** | آیا شروع شده؟ |

**نکته:** اگر health check فقط TCP باشد، سرویسی که اپلیکیشنش خطا می‌دهد سالم دیده می‌شود (فصل ۴۰).

### مثال‌های عملی

**Lab Scenario 1 — سرویس شروع نمی‌شود**

> **وضعیت:** `systemctl start myapp` → `failed`.
>
> **وظیفه:** پنج قدم.
>
> **پاسخ:**
> ۱. **`systemctl status myapp -l`** — پیام و exit code.
> ۲. **`journalctl -xeu myapp`** — لاگ کامل.
> ۳. **config:** `myapp --test-config` یا معادل.
> ۴. **پورت:** `ss -tulpn | grep :8080`.
> ۵. **SELinux:** `ausearch -m avc -ts today`.
>
> **درس:** `journalctl -xeu` بیشتر موارد را حل می‌کند.

**Lab Scenario 2 — سرویس هر ۵ دقیقه restart می‌شود**

> **وضعیت:** uptime سرویس هیچ‌وقت بیش از ۵ دقیقه نیست.
>
> **وظیفه:** علت.
>
> **پاسخ:**
> ۱. **`systemctl show myapp -p Restart`** — آیا `Restart=always` است؟
> ۲. **علت خروج:** `journalctl -u myapp | grep -i "exited\|killed"`.
> ۳. **OOM:** `dmesg | grep -i oom`.
> ۴. **Health check بیرونی:** آیا monitoring آن را restart می‌کند؟
> ۵. **Watchdog:** آیا `WatchdogSec` تنظیم شده؟
>
> **درس:** restart loop، معمولاً یک علت بیرونی دارد.

### مقایسه و الگوهای مختلف

**مقایسهٔ liveness و readiness:**

| معیار | Liveness | Readiness |
|---|---|---|
| شکست → | restart | حذف از LB |
| سؤال | زنده؟ | آماده؟ |
| مناسب | deadlock | startup |

**مقایسهٔ journalctl و فایل لاگ:**

| معیار | journalctl | فایل |
|---|---|---|
| ساختار | ✅ | متن |
| فیلتر | ✅ | grep |
| چرخش | خودکار | logrotate |
| مناسب | سرویس systemd | اپلیکیشن |

**مقایسهٔ ulimit و systemd limit:**

| معیار | ulimit | systemd |
|---|---|---|
| دامنه | session | سرویس |
| پایداری | موقت | دائمی |
| مناسب | تست | تولید |

### اشکال شایع

**۱. نادیده گرفتن SELinux.**
سرویس بالا نمی‌آید، دلیل پنهان.

**۲. فرض «کد خراب است».**
۵۰٪ موارد، وابستگی یا پیکربندی.

**۳. بدون health check عمیق.**
سرویس معیوب، ترافیک می‌گیرد.

**۴. ulimit پیش‌فرض.**
fd تمام می‌شود در بار بالا.

**۵. OOM بدون بررسی.**
سرویس می‌میرد و کسی نمی‌داند چرا.

**۶. لاگ بدون چرخش.**
دیسک پر می‌شود، سرویس می‌خوابد.

**۷. restart loop بدون ریشه‌یابی.**
هر ۵ دقیقه restart، به‌جای رفع.

**۸. تغییر config بدون تست.**
سرویس بعد از restart بالا نمی‌آید.

### نکات مهم (Tips & Tricks)

- **دسته را تعیین کن** (شروع نمی‌شود / می‌میرد / پاسخ نمی‌دهد).
- **`journalctl -xeu`** اولین دستور.
- **SELinux** را چک کن (`ausearch`).
- **OOM** را در `dmesg` ببین.
- **config را قبل از restart تست کن** (`nginx -t`).
- **در آزمون:** کلیدواژهٔ «service fails to start after reboot» → **not enabled**؛ «port in use» → **conflicting service**.

### خلاصه فصل

- پنج دسته: **شروع نمی‌شود، می‌میرد، پاسخ نمی‌دهد، خطا، کند**.
- پنج لایه: **پروسه، پورت، وابستگی، مجوز، منابع**.
- **SELinux** و **OOM** علل پنهان‌اند.
- **`journalctl -xeu`** بهترین ابزار.
- **health check عمیق** لازم است.
- **وابستگی‌ها** را قبل از کد چک کن.

---

**بعدی:** [فایل ۱۸: عیب‌یابی شبکه](18-Network-Troubleshooting.md)

---

## منابع و مراجع

- اهداف رسمی Server+ (V5)، دامنهٔ ۴: Troubleshooting
- Microsoft Learn — Windows Recovery Environment، WinDbg، DISM
- `man systemctl`، `man journalctl`، `man dmesg`
- Red Hat — Troubleshooting Guide
- Ubuntu — Recovery Mode documentation
- kdump: <https://documentation.suse.com/sles/html/SLES-all/cha-kdump.html>
- <https://www.comptia.org/en-us/certifications/server/>
