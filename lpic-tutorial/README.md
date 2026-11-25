# کتابچهٔ جامع LPIC-1 و LPIC-2 از صفر تا آزمون

راهنمای کامل و عملی مدیریت سیستم لینوکس — از مبانی تا آمادگی کامل برای آزمون‌های رسمی LPIC-1 و LPIC-2.

## دربارهٔ این کتابچه

این مجموعه یک مسیر یادگیری کامل و مرحله‌به‌مرحله برای تبدیل‌شدن به یک مدیر سیستم لینوکس حرفه‌ای و آماده‌شدن برای آزمون‌های بین‌المللی **LPIC-1** و **LPIC-2** است. محتوا بر پایهٔ اهداف رسمی آزمون‌ها نوشته شده و همراه با آزمایشگاه‌های عملی، مثال‌های قابل اجرا، تمرین‌ها و چک‌لیست‌های مرور است.

## مخاطب هدف

- **مبتدی:** کسی که با کامپیوتر آشناست اما تجربهٔ جدی لینوکس ندارد.
- **متوسط:** کسی که با خط فرمان کار کرده و می‌خواهد دانش خود را سیستماتیک کند.
- **آماده‌شوندهٔ آزمون:** کسی که می‌خواهد LPIC-1 یا LPIC-2 را با اطمینان بگذراند.

## ساختار کتابچه

```
lpic-tutorial/
├── README.md
├── SUMMARY.md
├── 01-Introduction-and-Certification-Roadmap.md
├── 02-Building-Your-Lab.md
├── 03-System-Architecture.md
├── 04-Installation-and-Package-Management.md
├── 05-GNU-Unix-Commands-Part1.md
├── 06-GNU-Unix-Commands-Part2.md
├── 07-Devices-Filesystems-FHS-Part1.md
├── 08-Devices-Filesystems-FHS-Part2.md
├── 09-Shells-and-Shell-Scripting.md
├── 10-User-Interfaces-and-Desktops.md
├── 11-Administrative-Tasks.md
├── 12-Essential-System-Services.md
├── 13-Networking-Fundamentals.md
├── 14-Security-Essentials.md
├── 15-Capacity-Planning.md
├── 16-Linux-Kernel.md
├── 17-System-Startup.md
├── 18-Filesystem-and-Devices.md
├── 19-Advanced-Storage-RAID-LVM.md
├── 20-Networking-Configuration.md
├── 21-System-Maintenance.md
├── 22-DNS-Server.md
├── 23-HTTP-Services.md
├── 24-File-Sharing.md
├── 25-Network-Client-Management.md
├── 26-Email-Services.md
├── 27-System-Security-LPIC2.md
├── 28-Exam-Preparation-and-Mock-Tests.md
├── 29-CheatSheets.md
└── 30-Glossary-and-Resources.md
```

## نقشهٔ آزمون‌ها

| مدرک | آزمون‌ها | نسخهٔ اهداف | مباحث |
|---|---|---|---|
| **LPIC-1** | 101-500 و 102-500 | 5.0 | Topic 101 تا 110 |
| **LPIC-2** | 201-450 و 202-450 | 4.5 | Topic 200 تا 212 |

### مباحث LPIC-1

1. **Topic 101 — System Architecture**
2. **Topic 102 — Linux Installation and Package Management**
3. **Topic 103 — GNU and Unix Commands**
4. **Topic 104 — Devices, Linux Filesystems, FHS**
5. **Topic 105 — Shells and Shell Scripting**
6. **Topic 106 — User Interfaces and Desktops**
7. **Topic 107 — Administrative Tasks**
8. **Topic 108 — Essential System Services**
9. **Topic 109 — Networking Fundamentals**
10. **Topic 110 — Security**

### مباحث LPIC-2

1. **Topic 200 — Capacity Planning**
2. **Topic 201 — Linux Kernel**
3. **Topic 202 — System Startup**
4. **Topic 203 — Filesystem and Devices**
5. **Topic 204 — Advanced Storage Device Administration**
6. **Topic 205 — Networking Configuration**
7. **Topic 206 — System Maintenance**
8. **Topic 207 — Domain Name Server**
9. **Topic 208 — HTTP Services**
10. **Topic 209 — File Sharing**
11. **Topic 210 — Network Client Management**
12. **Topic 211 — E-Mail Services**
13. **Topic 212 — System Security**

## چطور از این کتابچه استفاده کنیم

1. **آماده‌سازی محیط:** از فایل ۰۲ یک ماشین مجازی بسازید (Debian/Ubuntu + Rocky/RHEL).
2. **مطالعهٔ ترتیبی:** فایل‌ها را به ترتیب بخوانید؛ هر فصل را حتماً در آزمایشگاه تمرین کنید.
3. **تمرین فعال:** دستورها را خودتان اجرا کنید؛ فقط خواندن کافی نیست.
4. **مرور با چیت‌شیت:** فایل ۲۹ برای مرور سریع قبل از آزمون.
5. **آزمون تمرینی:** فایل ۲۸ شامل آزمون‌های شبیه‌سازی‌شده است.

## پیش‌نیازها

- یک کامپیوتر با حداقل ۸ گیگابایت رم (برای اجرای ماشین مجازی).
- VirtualBox یا VMware (رایگان).
- آشنایی مقدماتی با کامپیوتر و خط فرمان.

## کنوانسیون‌های این کتابچه

- **زبان:** فارسی روان + اصطلاحات تخصصی به انگلیسی (در پرانتز در اولین استفاده).
- **نمایش دستورها:** دستورهایی که با `$` شروع می‌شوند برای کاربر عادی و `#` برای کاربر ریشه (root) است.
- **توزیع‌ها:** هر جا تفاوت مهمی بین خانوادهٔ Debian و Red Hat وجود دارد، هر دو پوشش داده می‌شود.
- **آزمایشگاه:** هر فصل بخش عملی دارد که باید در ماشین مجازی اجرا شود.

## وضعیت

- **تعداد فایل:** ۳۰
- **تعداد فصل:** ۱۰۹
- **مدت تخمینی مطالعه:** ۶۰ تا ۸۰ ساعت
- **مرجع:** LPIC-1 v5.0 + LPIC-2 v4.5

---

**شروع مطالعه:** [فایل ۰۱ — مقدمه و نقشهٔ راه مدرک](01-Introduction-and-Certification-Roadmap.md)
