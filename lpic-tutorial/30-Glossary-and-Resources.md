# فایل ۳۰ — واژه‌نامه و منابع

**مدت یادگیری**: ۳ ساعت
**سطح سختی**: همه
**پیش‌نیاز**: تمام فایل‌های ۰۱–۲۹

---

## فهرست محتوا

- [فصل ۱۰۷: واژه‌نامهٔ فارسی–انگلیسی](#فصل-۱۰۷-واژهنامهٔ-فارسیانگلیسی)
- [فصل ۱۰۸: منابع رسمی، کتاب‌ها و مستندات](#فصل-۱۰۸-منابع-رسمی-کتابها-و-مستندات)
- [فصل ۱۰۹: مسیر ادامهٔ یادگیری](#فصل-۱۰۹-مسیر-ادامهٔ-یادگیری)

---

## فصل ۱۰۷: واژه‌نامهٔ فارسی–انگلیسی

### مقدمه

واژه‌نامهٔ اصطلاحات کلیدی لینوکس و LPIC به‌صورت الفبایی (بر اساس انگلیسی). تسلط بر اصطلاحات انگلیسی برای آزمون ضروری است.

### مفاهیم اساسی

| انگلیسی | فارسی |
|---|---|
| **ACL** | لیست کنترل دسترسی |
| **AppArmor** | چارچوب کنترل دسترسی (Ubuntu) |
| **ARP** | پروتکل تفکیک آدرس |
| **BIND** | سرور DNS |
| **Block** | بلوک (واحد دیسک) |
| **Bootloader** | بارگذارندهٔ بوت |
| **Bonding** | ترکیب رابط‌های شبکه |
| **Bridge** | پل (اتصال لایه ۲) |
| **CIDR** | نشانه‌گذاری آدرس شبکه |
| **CIFS** | سیستم فایل اینترنتی مشترک |
| **Container** | کانتینر |
| **cron** | زمان‌بند تکرارشونده |
| **CUPS** | سیستم چاپ یونیکس |
| **Daemon** | سرویس پس‌زمینه |
| **DHCP** | پروتکل پیکربندی پویا میزبان |
| **DMARC** | احراز ایمیل دامنه |
| **DNS** | سیستم نام دامنه |
| **DNSSEC** | DNS امن |
| **Dovecot** | سرور IMAP/POP3 |
| **ext4** | فایل‌سیستم استاندارد لینوکس |
| **firewalld** | فایروال RHEL |
| **fstab** | جدول فایل‌سیستم |
| **GPG** | ابزار رمزنگاری |
| **GRUB** | بارگذارندهٔ بوت استاندارد |
| **Host** | میزبان |
| **IMAP** | پروتکل دسترسی به ایمیل |
| **inode** | گرهٔ اطلاعات فایل |
| **IP forwarding** | عبوردهی IP |
| **journald** | لاگ systemd |
| **Kernel** | کرنل |
| **LDAP** | پروتکل دایرکتوری سبک |
| **Lease** | اجارهٔ DHCP |
| **Load average** | میانگین بار |
| **Locale** | تنظیمات منطقه‌ای |
| **LVM** | مدیر حجم منطقی |
| **MAC** | کنترل دسترسی اجباری |
| **mdadm** | ابزار RAID نرم‌افزاری |
| **MTA** | عامل انتقال ایمیل |
| **MUA** | کلاینت ایمیل |
| **NAT** | ترجمهٔ آدرس شبکه |
| **NFS** | سیستم فایل شبکه |
| **Nice** | اولویت فرآیند |
| **NTP** | پروتکل زمان شبکه |
| **OpenVPN** | پیاده‌سازی VPN |
| **PAM** | ماژول‌های احراز هویت |
| **Partition** | پارتیشن |
| **POP3** | پروتکل پست الکترونیک |
| **Postfix** | سرور ایمیل |
| **PV/VG/LV** | لایه‌های LVM |
| **RAID** | آرایهٔ افزونه دیسک |
| **Route** | مسیر |
| **Samba** | پیاده‌سازی SMB |
| **SELinux** | لینوکس امنیتی (RHEL) |
| **SMB** | پروتکل بلوک پیام سرور |
| **SMTP** | پروتکل انتقال ایمیل |
| **Snapshot** | تصویر لحظه‌ای |
| **SPF** | چارچوب سیاست فرستنده |
| **Squid** | پروکسی کش |
| **SSH** | پوستهٔ امن |
| **Subnet** | زیرشبکه |
| **swap** | حافظهٔ مجازی |
| **sysctl** | پارامترهای کرنل |
| **systemd** | مدیر سیستم |
| **Target** | هدف (معادل runlevel) |
| **UID/GID** | شناسهٔ کاربر/گروه |
| **Unit** | واحد systemd |
| **VLAN** | شبکهٔ مجازی |
| **VPN** | شبکهٔ خصوصی مجازی |
| **XFS** | فایل‌سیستم با کارایی بالا |
| **zone** | منطقهٔ DNS |

### مثال‌های عملی

**اصطلاحات پرکاربرد در آزمون:**

```text
RAID, LVM, systemd, SELinux, AppArmor, fstab, GRUB,
NAT, DHCP, DNS, zone, MTA, IMAP, inode, symlink,
mdadm, PV/VG/LV, bonding, VLAN, PAM, LDAP
```

### مقایسه و الگوهای مختلف

**معادل‌های توزیعی:**

| مفهوم | Debian | RHEL |
|---|---|---|
| فایروال | ufw | firewalld |
| MAC | AppArmor | SELinux |
| لاگ auth | auth.log | secure |
| مخزن | sources.list | yum.repos.d |
| پکیج | .deb | .rpm |

### اشکال شایع

- **اشکال: ندانستن اصطلاحات انگلیسی.** آزمون انگلیسی است؛ واژه‌نامه را مرور کنید.
- **اشکال: گیج شدن معادل‌ها.** ufw/firewalld، AppArmor/SELinux.
- **اشکال: فراموش کردن اختصارها.** PV/VG/LV، MTA/MDA/MUA.

### نکات مهم (Tips & Tricks)

- **واژه‌نامه را قبل از آزمون مرور کنید.**
- **اصطلاحات را در context یاد بگیرید.**
- **flashcard بسازید.**
- **معادل‌های توزیعی را حفظ کنید.**
- **اختصارها را بسط دهید.**
- **از mnemonics استفاده کنید.**
- **واژه‌نامه را چاپ کنید.**
- **با اصطلاحات در متن تمرین کنید.**
- **Anki برای flashcard.**
- **گروه‌بندی موضوعی.**

### خلاصه فصل

واژه‌نامهٔ الفبایی اصطلاحات کلیدی لینوکس/LPIC. تسلط بر اصطلاحات انگلیسی برای آزمون ضروری است. معادل‌های توزیعی (ufw/firewalld، AppArmor/SELinux) را به خاطر بسپارید.

---

## فصل ۱۰۸: منابع رسمی، کتاب‌ها و مستندات

### مقدمه

این فصل منابع معتبر برای مطالعهٔ بیشتر و مرجع.

### مفاهیم اساسی

**منابع رسمی LPI:**

| منبع | آدرس |
|---|---|
| LPI | https://www.lpi.org |
| LPIC-1 Objectives | https://www.lpi.org/our-certifications/lpic-1-overview |
| LPIC-2 Objectives | https://www.lpi.org/our-certifications/lpic-2-overview |
| Pearson VUE | https://www.pearsonvue.com |

**مستندات توزیع‌ها:**

| توزیع | مستندات |
|---|---|
| Ubuntu | https://ubuntu.com/server/docs |
| Debian | https://www.debian.org/doc/ |
| RHEL | https://access.redhat.com/documentation/ |
| Rocky | https://docs.rockylinux.org |
| Arch | https://wiki.archlinux.org |

**مستندات اجزا:**

| جزء | آدرس |
|---|---|
| Kernel | https://www.kernel.org/doc/ |
| systemd | https://www.freedesktop.org/wiki/Software/systemd/ |
| BIND | https://bind9.readthedocs.io |
| Nginx | https://nginx.org/en/docs/ |
| Apache | https://httpd.apache.org/docs/ |
| Postfix | https://www.postfix.org/documentation.html |
| OpenLDAP | https://www.openldap.org/doc/ |
| Samba | https://www.samba.org/samba/docs/ |
| OpenVPN | https://openvpn.net/community-resources/ |

### مثال‌های عملی

**کتاب‌های پیشنهادی:**

```text
- LPIC-1 Study Guide (Christine Bresnahan, Richard Blum)
- LPIC-2 Study Guide (Christine Bresnahan, Richard Blum)
- Linux Bible (Christopher Negus)
- The Linux Command Line (William Shotts)
- How Linux Works (Brian Ward)
- UNIX and Linux System Administration Handbook
- Linux Administration Handbook
```

**منابع آنلاین:**

```text
- tldp.org (The Linux Documentation Project)
- kernelnewbies.org
- wiki.archlinux.org (مستندات عالی)
- digitalocean.com/community (آموزش‌ها)
- explainshell.com (توضیح دستورها)
- crontab.guru (cron syntax)
- regex101.com (regex)
- linuxjourney.com
```

**ابزارهای کمک‌آموزشی:**

| ابزار | کار |
|---|---|
| `man` | راهنمای رسمی |
| `tldr` | مثال‌های عملی |
| `explainshell` | توضیح دستور |
| `shellcheck` | تحلیل اسکریپت |

### مقایسه و الگوهای مختلف

**منابع رسمی در برابر غیررسمی:**

| نوع | مزیت | معایب |
|---|---|---|
| رسمی | دقیق، به‌روز | خشک |
| غیررسمی | ساده، مثال‌محور | کیفیت متغیر |

**توصیه:** هر دو را ترکیب کنید.

**انواع منابع:**

| نوع | مناسب |
|---|---|
| کتاب | ساختاریافته |
| ویدیو | بصری |
| مستندات | دقیق |
| انجمن | پرسش سریع |

### اشکال شایع

- **اشکال: اتکا به یک منبع.** ترکیب کنید.
- **اشکال: نادیده گرفتن man pages.** بهترین مرجع.
- **اشکال: مطالب قدیمی.** نسخه را چک کنید.
- **اشکال: منابع نامعتبر.** اعتبار را بررسی کنید.

### نکات مهم (Tips & Tricks)

- **اهداف رسمی LPI را دانلود کنید.**
- **man pages را دوست خود بدانید.**
- **wiki.archlinux.org منبع عالی است.**
- **مستندات رسمی هر سرویس را بخوانید.**
- **از منابع به‌روز استفاده کنید.**
- **`tldr` مثال‌های سریع می‌دهد.**
- **`explainshell.com` دستورها را توضیح می‌دهد.**
- **کتاب‌های Study Guide جامع‌اند.**
- **انجمن‌ها برای پرسش.**
- **ویدیو برای مفاهیم سخت.**
- **مستندات نسخه را چک کنید.**
- **بوکمارک کنید.**

### خلاصه فصل

منابع رسمی LPI، مستندات توزیع‌ها و اجزا، و کتاب‌های پیشنهادی جمع‌آوری شد. ترکیب منابع رسمی و غیررسمی بهترین نتیجه را می‌دهد. man pages مرجع نهایی است. ابزارهایی مثل `tldr` و `explainshell` کار را راحت می‌کنند.

---

## فصل ۱۰۹: مسیر ادامهٔ یادگیری

### مقدمه

تبریک! شما کل مباحث LPIC-1 و LPIC-2 را پوشش دادید. در این فصل مسیرهای بعدی را می‌بینید.

### مفاهیم اساسی

**مسیرهای بعد از LPIC-2:**

| مسیر | توضیح |
|---|---|
| **LPIC-3** | تخصص (مجازی‌سازی، امنیت، Enterprise) |
| **RHCE** | خودکارسازی Red Hat (Ansible) |
| **DevOps** | CI/CD، Docker، Kubernetes |
| **Cloud** | AWS/Azure/GCP |
| **تخصص امنیت** | CEH, CISSP |
| **تخصص شبکه** | CCNA, CCNP |

**مهارت‌های مکمل:**

- Ansible (خودکارسازی).
- Docker/Kubernetes (کانتینر).
- Terraform (زیرساخت به‌عنوان کد).
- Python/Bash (اسکریپت‌نویسی).
- Git (نسخه‌بندی).
- CI/CD (Jenkins, GitLab).

### مثال‌های عملی

**نقشهٔ راه پیشنهادی:**

```
۱. LPIC-1 (تکمیل شده ✓)
۲. LPIC-2 (تکمیل شده ✓)
۳. یک تخصص:
   الف) LPIC-3 (مدیریت سازمانی)
   ب) RHCE (خودکارسازی)
   ج) Kubernetes (CKA/CKAD)
۴. مهارت مکمل (Ansible, Python)
۵. تجربهٔ عملی (پروژه، شغل)
```

**مثال پروژهٔ عملی:**

```bash
# یک هوم‌لب بسازید:
# - Proxmox/KVM برای مجازی‌سازی
# - LVM/RAID برای ذخیره‌سازی
# - Docker برای سرویس‌ها
# - Nginx برای reverse proxy
# - Prometheus/Grafana برای پایش
# - Ansible برای خودکارسازی
```

### مقایسه و الگوهای مختلف

**مسیرهای شغلی:**

| مسیر | مدرک پیشنهادی |
|---|---|
| SysAdmin | LPIC-2 → RHCE |
| DevOps | LPIC-2 → CKA + Ansible |
| Cloud | LPIC-2 → AWS/Azure |
| Security | LPIC-2 → LPIC-3 Security |
| Network | LPIC-2 → CCNA |
| SRE | LPIC-2 → CKA + Prometheus |

**LPIC-3 specializations:**

| تخصص | تمرکز |
|---|---|
| Mixed Environment | Samba، LDAP، ادغام |
| Security | SELinux، VPN، امنیت |
| Virtualization | KVM، کانتینر |
| High Availability | خوشه، failover |

### اشکال شایع

- **اشکال: توقف بعد از LPIC-2.** یادگیری ادامه دارد.
- **اشکال: مدرک بدون تجربه.** پروژهٔ واقعی بسازید.
- **اشکال: پراکندگی.** یک مسیر را انتخاب کنید.
- **اشکال: نادیده گرفتن خودکارسازی.** Ansible ضروری است.
- **اشکال: نبود Git.** نسخه‌بندی مهم است.
- **اشکال: توقف یادگیری.** فناوری تغییر می‌کند.

### نکات مهم (Tips & Tricks)

- **یک پروژهٔ واقعی (سرور، هوم‌لب) بسازید.**
- **مهارت خودکارسازی (Ansible) را یاد بگیرید.**
- **در انجمن‌ها فعال باشید.**
- **مستندات بنویسید (تجربهٔ خود را).**
- **یادگیری مستمر داشته باشید.**
- **مدرک + تجربه = موفقیت.**
- **Git و CI/CD را یاد بگیرید.**
- **کانتینر (Docker/K8s) ضروری است.**
- **ابری (Cloud) را یاد بگیرید.**
- **یک تخصص انتخاب کنید.**
- **پروژهٔ open source مشارکت کنید.**
- **وبلاگ بنویسید.**

### خلاصه فصل

بعد از LPIC-2، مسیرها شامل LPIC-3، RHCE، DevOps، Cloud و امنیت است. مهارت‌های مکمل (Ansible، Docker، Python) را یاد بگیرید. یک مسیر را انتخاب کنید و با پروژهٔ عملی تجربه بسازید. مدرک + تجربه = موفقیت.

---

## منابع و مراجع

- Linux Professional Institute: https://www.lpi.org
- The Linux Documentation Project: https://tldp.org
- Arch Wiki: https://wiki.archlinux.org
- Kernel Newbies: https://kernelnewbies.org
- Ansible Documentation: https://docs.ansible.com
- Kubernetes Documentation: https://kubernetes.io/docs/
- Docker Documentation: https://docs.docker.com

**پایان کتابچه** — تبریک! شما کل مسیر LPIC-1 و LPIC-2 را پوشش دادید. این مجموعهٔ ۳۰ فایلی راهنمای کاملی برای تبدیل‌شدن به یک مدیر سیستم لینوکس حرفه‌ای و آماده‌شدن برای آزمون‌های بین‌المللی بود. موفق باشید! 🐧
