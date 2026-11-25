# فایل ۲۹ — چیت‌شیت‌ها

**مدت یادگیری**: ۳ ساعت
**سطح سختی**: همه
**پیش‌نیاز**: تمام فایل‌های ۰۱–۲۸

---

## فهرست محتوا

- [فصل ۱۰۳: چیت‌شیت دستورات LPIC-1](#فصل-۱۰۳-چیتشیت-دستورات-lpic-1)
- [فصل ۱۰۴: چیت‌شیت دستورات LPIC-2](#فصل-۱۰۴-چیتشیت-دستورات-lpic-2)
- [فصل ۱۰۵: چیت‌شیت پیکربندی سرویس‌ها](#فصل-۱۰۵-چیتشیت-پیکربندی-سرویسها)
- [فصل ۱۰۶: نکات طلایی مرور سریع](#فصل-۱۰۶-نکات-طلایی-مرور-سریع)

---

## فصل ۱۰۳: چیت‌شیت دستورات LPIC-1

### مقدمه

این فصل مرجع سریع دستورات کلیدی LPIC-1 برای مرور قبل از آزمون است.

### مفاهیم اساسی

**مدیریت فایل و ناوبری:**

```bash
pwd; cd; ls -la; cp -a; mv; rm -rf; mkdir -p; touch
find / -name "*.conf"; locate; which; whereis; type
ln -s target link; ln target hardlink
```

**متن و پردازش:**

```bash
cat -n; head -5; tail -f; less; grep -r; sort -n; uniq -c
wc -l; cut -d: -f1; tr; sed 's/a/b/g'; awk '{print $1}'
grep -E; grep -A3; grep -v; grep -o
```

**جریان‌ها:**

```bash
cmd > file; cmd >> file; cmd 2> err; cmd 2>&1; cmd | cmd2; tee
xargs; $(cmd); <(cmd)
```

**فرآیندها:**

```bash
ps aux; top; htop; pstree; pgrep; kill -15; kill -9; killall
nice -n 19; renice -n 10; jobs; bg; fg; nohup
```

**کاربران:**

```bash
useradd -m; adduser; usermod -aG; userdel -r; passwd
groupadd; gpasswd -a; id; groups; chage -l
vipw; vigr
```

**مجوز و مالکیت:**

```bash
chmod 755; chmod u+x; chown user:group; chgrp
umask; find / -perm -4000; setfacl; getfacl
```

**دیسک و فایل‌سیستم:**

```bash
lsblk; fdisk -l; parted; mkfs.ext4; fsck; mount; umount
df -h; du -sh; blkid; tune2fs; dumpe2fs; e2label
```

**بوت و systemd:**

```bash
systemctl start/stop/restart/enable/status
systemctl get-default; set-default; isolate
journalctl -b; -u service; -f; -p err
systemd-analyze blame
```

**شبکه:**

```bash
ip a; ip r; ip link; ss -tlnp; ping; traceroute; mtr
dig; nslookup; nmcli; netplan apply; resolvectl
```

**پکیج:**

```bash
# Debian
apt update/install/remove/purge/autoremove; dpkg -i/-L/-S
# RHEL
dnf install/remove/update/history; rpm -ivh/-ql/-qf
```

### مثال‌های عملی

**ترکیب‌های پرکاربرد:**

```bash
# ۵ پروسه پر مصرف CPU
ps aux | sort -k3 -rn | head -5

# پاک‌سازی لاگ‌های قدیمی
find /var/log -name "*.log" -mtime +30 -delete

# جستجوی فایل حاوی متن
grep -rl "TODO" /project

# بکاپ فشرده
tar -czf backup.tar.gz /home/user

# شمارش خطوط همهٔ فایل‌های .log
find /var/log -name "*.log" | xargs wc -l
```

### مقایسه و الگوهای مختلف

**کلیدهای میانبر:**

| کلید | کار |
|---|---|
| `Tab` | تکمیل |
| `Ctrl+R` | جستجوی تاریخچه |
| `Ctrl+C` | توقف |
| `Ctrl+Z` | توقف موقت |
| `Ctrl+L` | پاک صفحه |
| `Ctrl+A`/`E` | ابتدا/انتهای خط |
| `Ctrl+U`/`W` | پاک خط/کلمه |
| `!!` | تکرار آخرین |

**تفاوت Debian/RHEL:**

| کار | Debian | RHEL |
|---|---|---|
| نصب | `apt install` | `dnf install` |
| فایروال | `ufw` | `firewalld` |
| لاگ auth | `auth.log` | `secure` |

### اشکال شایع

- **اشکال: فراموش کردن گزینه‌ها.** `ls -la` نه `ls -l -a`.
- **اشکال: اشتباه مسیرها.** از Tab استفاده کنید.
- **اشکال: `rm -rf` بی‌احتیاط.** خطرناک.
- **اشکال: اشتباه Debian/RHEL.** به توزیع دقت کنید.

### نکات مهم (Tips & Tricks)

- **این چیت‌شیت را چاپ کنید و کنار دست داشته باشید.**
- **دستورها را در VM تمرین کنید.**
- **`man` بهترین مرجع است.**
- **تفاوت Debian/RHEL را حفظ کنید.**
- **کلیدهای میانبر کار را سریع می‌کنند.**
- **`history` و `Ctrl+R` برای تکرار.**
- **`alias` برای میانبر.**
- **`--help` برای یادآوری سریع.**
- **`whatis` توضیح یک‌خطی.**
- **`tldr` مثال‌های عملی.**

### خلاصه فصل

این چیت‌شیت دستورات کلیدی LPIC-1 را در دسته‌های فایل، متن، فرآیند، کاربر، دیسک، بوت، شبکه و پکیج جمع کرد. برای مرور سریع قبل از آزمون استفاده کنید.

---

## فصل ۱۰۴: چیت‌شیت دستورات LPIC-2

### مقدمه

مرجع سریع دستورات پیشرفتهٔ LPIC-2.

### مفاهیم اساسی

**کرنل:**

```bash
uname -r; lsmod; modinfo; modprobe; sysctl -w; dmesg -T
depmod -a; /etc/modules-load.d/; /etc/sysctl.d/
```

**ذخیره‌سازی:**

```bash
# RAID
mdadm --create --level=1; --detail; --fail; --add; cat /proc/mdstat
# LVM
pvcreate; vgcreate; lvcreate; lvextend; vgextend; resize2fs
lvs; vgs; pvs; lvdisplay; pvmove
```

**فایل‌سیستم پیشرفته:**

```bash
tune2fs -m 1; resize2fs; dumpe2fs; xfs_growfs; smartctl -H
xfs_repair; xfs_info; e2fsck
```

**بوت و بازیابی:**

```bash
systemctl rescue; emergency; systemd-analyze blame
grub-install; update-grub; bootctl; efibootmgr
init=/bin/bash; mount -o remount,rw /
```

**پایش:**

```bash
sar -u; -r; -d; -n DEV; iostat -x; vmstat; free -h
mpstat -P ALL; pidstat; iotop
```

**شبکه پیشرفته:**

```bash
ip route add; sysctl net.ipv4.ip_forward; iptables -t nat
tcpdump -i eth0; mtr; ss -tunap
bonding; vlan; bridge
```

### مثال‌های عملی

**سناریوهای پرکاربرد:**

```bash
# بزرگ کردن LV آنلاین
sudo lvextend -L +5G /dev/vg/lv && sudo resize2fs /dev/vg/lv

# ساخت RAID 1
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# بکاپ incremental
rsync -av --delete /src/ /backup/

# ساخت سرویس systemd
sudo systemctl daemon-reload && sudo systemctl enable --now myapp

# NAT
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

### مقایسه و الگوهای مختلف

**دستورهای کلیدی:**

| موضوع | دستور |
|---|---|
| RAID | `mdadm` |
| LVM | `pv/vg/lv*` |
| کرنل | `modprobe`, `sysctl` |
| پایش | `sar`, `iostat` |
| شبکه | `ip`, `iptables` |
| بکاپ | `rsync`, `tar` |

**ترتیب LVM:**

```
pvcreate → vgcreate → lvcreate → mkfs → mount
```

### اشکال شایع

- **اشکال: ترتیب LVM.** PV→VG→LV.
- **اشکال: فراموش کردن resize.** بعد از lvextend.
- **اشکال: XFS کوچک‌شدن.** ممکن نیست.
- **اشکال: mdadm.conf فراموش.** بعد از reboot.

### نکات مهم (Tips & Tricks)

- **LVM و RAID را عملی تمرین کنید.**
- **`sar` برای تحلیل روند.**
- **`iptables-save` برای پایداری.**
- **`mdadm.conf` و `fstab` را ذخیره کنید.**
- **snapshot برای بکاپ سازگار.**
- **`dmesg` منبع عیب‌یابی سخت‌افزار.**
- **`rescue.target` برای بازیابی.**
- **`tune2fs -m 1` فضای ریشه.**
- **`smartctl` سلامت دیسک.**
- **`mtr` تشخیص نقطهٔ قطع.**

### خلاصه فصل

این چیت‌شیت دستورات پیشرفتهٔ کرنل، ذخیره‌سازی، فایل‌سیستم، بوت، پایش و شبکه را جمع کرد. برای مرور سریع LPIC-2 استفاده کنید.

---

## فصل ۱۰۵: چیت‌شیت پیکربندی سرویس‌ها

### مقدمه

مرجع سریع مسیرهای پیکربندی سرویس‌های اصلی.

### مفاهیم اساسی

**مسیرهای پیکربندی:**

| سرویس | مسیر پیکربندی |
|---|---|
| Apache | `/etc/apache2/` یا `/etc/httpd/` |
| Nginx | `/etc/nginx/nginx.conf` |
| BIND | `/etc/bind/named.conf` |
| Samba | `/etc/samba/smb.conf` |
| NFS | `/etc/exports` |
| DHCP | `/etc/dhcp/dhcpd.conf` |
| Postfix | `/etc/postfix/main.cf` |
| Dovecot | `/etc/dovecot/dovecot.conf` |
| Squid | `/etc/squid/squid.conf` |
| OpenVPN | `/etc/openvpn/server.conf` |
| SSH | `/etc/ssh/sshd_config` |
| LDAP | `/etc/ldap/slapd.d/` |
| PAM | `/etc/pam.d/` |

**مسیرهای لاگ:**

| سرویس | لاگ |
|---|---|
| Apache | `/var/log/apache2/` |
| Nginx | `/var/log/nginx/` |
| Mail | `/var/log/mail.log` |
| System | `journalctl` |
| Auth | `/var/log/auth.log` |

### مثال‌های عملی

**بررسی پیکربندی:**

```bash
apache2ctl configtest     # Apache
nginx -t                  # Nginx
named-checkconf           # BIND
named-checkzone           # zone
testparm                  # Samba
postfix check             # Postfix
doveconf -n               # Dovecot
squid -k parse            # Squid
sshd -t                   # SSH
dhcpd -t                  # DHCP
```

**ری‌استارت سرویس:**

```bash
sudo systemctl reload nginx     # بازخوانی (بدون قطع)
sudo systemctl restart nginx    # ری‌استارت کامل
```

### مقایسه و الگوهای مختلف

**پورت‌های استاندارد:**

| سرویس | پورت |
|---|---|
| HTTP | 80 |
| HTTPS | 443 |
| DNS | 53 |
| SMTP | 25 |
| Submission | 587 |
| IMAP | 143/993 |
| POP3 | 110/995 |
| NFS | 2049 |
| SSH | 22 |
| DHCP | 67/68 |
| LDAP | 389/636 |
| SMB | 139/445 |
| OpenVPN | 1194 |

**SELinux booleans:**

| Boolean | کار |
|---|---|
| `httpd_can_network_connect` | Apache به شبکه |
| `samba_export_all_rw` | Samba نوشتن |
| `named_write_master_zones` | BIND نوشتن |

### اشکال شایع

- **اشکال: فراموش کردن بررسی پیکربندی.** سرویس بالا نمی‌آید.
- **اشکال: اشتباه مسیر.** بسته به توزیع متفاوت است.
- **اشکال: نبود SELinux boolean.** سرویس مسدود.
- **اشکال: فایروال.** پورت بسته.

### نکات مهم (Tips & Tricks)

- **همیشه بعد از تغییر، پیکربندی را بررسی کنید.**
- **لاگ‌ها اولین جای عیب‌یابی‌اند.**
- **پورت‌ها را حفظ کنید.**
- **`reload` بهتر از `restart` است.**
- **SELinux booleans را برای سرویس‌ها بررسی کنید.**
- **فایروال پورت لازم را باز کنید.**
- **`systemctl status` وضعیت.**
- **`journalctl -u` لاگ سرویس.**
- **`ss -tlnp` پورت‌های باز.**
- **مسیرها را حفظ کنید.**

### خلاصه فصل

این چیت‌شیت مسیرهای پیکربندی، لاگ و پورت سرویس‌های اصلی را جمع کرد. با دستورهای بررسی (configtest, nginx -t, ...) پیکربندی را اعتبارسنجی کنید. پورت‌ها و SELinux booleans را حفظ کنید.

---

## فصل ۱۰۶: نکات طلایی مرور سریع

### مقدمه

آخرین نکات کلیدی برای مرور سریع قبل از آزمون.

### مفاهیم اساسی

**۱۵ نکتهٔ طلایی:**

1. **تفاوت Debian/RHEL** را یک صفحه یادداشت کنید.
2. **ترتیب LVM:** PV → VG → LV.
3. **RAID بکاپ نیست.**
4. **XFS قابل کوچک‌شدن نیست.**
5. **RAID 1 = آینه، RAID 0 = سرعت، RAID 5 = parity.**
6. **مجوز:** 644 فایل، 755 اسکریپت، 600 کلید.
7. **systemctl:** start/stop/enable/status/daemon-reload.
8. **`2>&1`** برای ادغام خطا و خروجی.
9. **`journalctl -b`** برای لاگ بوت.
10. **همیشه بکاپ و تست بازیابی.**
11. **`resize2fs` بعد از `lvextend`.**
12. **`mdadm.conf` را ذخیره کنید.**
13. **`restorecon` برای SELinux.**
14. **`_netdev` در fstab برای NFS.**
15. **TLS 1.2+ برای HTTPS.**

### مثال‌های عملی

**جدول تفاوت‌های سریع:**

| کار | Debian | RHEL |
|---|---|---|
| نصب | `apt install` | `dnf install` |
| فعال ماژول | `a2enmod` | ویرایش مستقیم |
| فایروال | `ufw` | `firewalld` |
| MAC | AppArmor | SELinux |
| لاگ auth | `/var/log/auth.log` | `/var/log/secure` |
| پکیج | `.deb` | `.rpm` |

**چک‌لیست نهایی:**

```
☑ همهٔ Topics مطالعه شده
☑ آزمایشگاه عملی هر Topic
☑ آزمون‌های تمرینی (۸۰٪+)
☑ تفاوت Debian/RHEL
☑ دستورهای کلیدی
☑ مسیرهای پیکربندی
☑ پورت‌ها
☑ چیت‌شیت مرور
```

### مقایسه و الگوهای مختلف

**اولویت‌های مرور:**

| اولویت | موضوع |
|---|---|
| بالا | LVM, RAID, systemd, شبکه |
| متوسط | سرویس‌ها (DNS/Web/Mail) |
| پایین | مباحث تاریخی |

**نقاط ضعف رایج:**

| Topic | اشتباه |
|---|---|
| LVM | ترتیب |
| RAID | بکاپ |
| systemd | daemon-reload |
| شبکه | Debian/RHEL |

### اشکال شایع

- **اشکال: مرور سطحی.** عمق داشته باشید.
- **اشکال: نادیده گرفتن تفاوت توزیع‌ها.**
- **اشکال: نبود تمرین عملی.**
- **اشکال: عجله در آزمون.**

### نکات مهم (Tips & Tricks)

- **یک شب قبل، فقط چیت‌شیت‌ها را مرور کنید.**
- **خواب کافی.**
- **سؤال را کامل بخوانید.**
- **سخت‌ها را علامت بزنید.**
- **نفس عمیق.**
- **آب بنوشید.**
- **لباس راحت.**
- **صبحانهٔ سبک.**
- **زودتر حاضر شوید.**
- **نمرهٔ ۵۰۰ کافی است.**
- **حدس منطقی بزنید.**
- **بازبینی در پایان.**

### خلاصه فصل

۱۵ نکتهٔ طلایی و جدول تفاوت‌های Debian/RHEL برای مرور نهایی. اولویت‌های مرور: LVM/RAID/systemd/شبکه. تمرین عملی، خواب کافی و آرامش در آزمون کلید موفقیت است.

---

## منابع و مراجع

- man pages (مرجع نهایی)
- LPI Exam Objectives: https://www.lpi.org
- این کتابچه، فایل‌های ۰۱–۲۸
- crontab.guru: https://crontab.guru
- explainshell.com: https://explainshell.com

**بعدی:** [فایل ۳۰ — واژه‌نامه و منابع](30-Glossary-and-Resources.md)
