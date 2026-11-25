# فایل ۲۷ — امنیت سیستم (LPIC-2)

**مدت یادگیری**: ۸ ساعت
**سطح سختی**: پیشرفته
**پیش‌نیاز**: فایل ۱۴ (امنیت پایه)، فایل ۲۰ (شبکه)

---

## فهرست محتوا

- [فصل ۹۳: پیکربندی روتر](#فصل-۹۳-پیکربندی-روتر)
- [فصل ۹۴: مدیریت سرورهای FTP](#فصل-۹۴-مدیریت-سرورهای-ftp)
- [فصل ۹۵: Secure Shell (SSH)](#فصل-۹۵-secure-shell-ssh)
- [فصل ۹۶: وظایف امنیتی](#فصل-۹۶-وظایف-امنیتی)
- [فصل ۹۷: OpenVPN](#فصل-۹۷-openvpn)

---

## فصل ۹۳: پیکربندی روتر

### مقدمه

یک سیستم لینوکسی می‌تواند به‌عنوان **روتر (router)** بین شبکه‌ها عمل کند. تصور کنید می‌خواهید یک سرور لینوکسی را به‌عنوان gateway بین اینترنت و شبکهٔ داخلی پیکربندی کنید.

در این فصل پیکربندی آن را یاد می‌گیرید.

### مفاهیم اساسی

**روتر چیست؟** دستگاهی که بسته‌ها را بین شبکه‌ها هدایت می‌کند.

**مفاهیم کلیدی:**

- **IP Forwarding:** عبور بسته بین رابط‌ها.
- **NAT:** ترجمهٔ آدرس.
- **Static Routing:** مسیرهای دستی.
- **Policy Routing:** مسیریابی بر اساس سیاست.
- **Firewall:** فیلتر بسته‌ها.

### مثال‌های عملی

**فعال‌سازی forwarding:**

```bash
# موقت
sudo sysctl -w net.ipv4.ip_forward=1

# دائمی
echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/99-router.conf
sudo sysctl -p
```

**NAT با iptables:**

```bash
# MASQUERADE
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# FORWARD
sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT

# ذخیره
sudo iptables-save > /etc/iptables/rules.v4
```

**مسیر استاتیک:**

```bash
sudo ip route add 10.10.0.0/16 via 192.168.1.254

# دائمی در netplan/nmcli
```

**Port Forwarding:**

```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 \
  -j DNAT --to-destination 192.168.1.20:80
```

### مقایسه و الگوهای مختلف

**ابزارهای فایروال/NAT:**

| ابزار | مزیت |
|---|---|
| `iptables` | کلاسیک، کامل |
| `nftables` | مدرن، سریع |
| `firewalld` | ساده، zone |

**انواع NAT:**

| نوع | کاربرد |
|---|---|
| SNAT | خروج (source) |
| DNAT | ورود/port forward |
| MASQUERADE | IP پویا |

**sysctlهای مهم:**

| پارامتر | کار |
|---|---|
| `net.ipv4.ip_forward` | routing |
| `net.ipv4.conf.all.rp_filter` | reverse path |
| `net.ipv4.conf.all.accept_redirects` | redirects |

### اشکال شایع

**اشکال ۱: فراموش کردن forwarding.**
NAT کار نمی‌کند. راه‌حل: فعال کنید.

**اشکال ۲: rules ذخیره‌نشده.**
بعد از reboot پاک می‌شود. راه‌حل: `iptables-save`.

**اشکال ۳: FORWARD پیش‌فرض DROP.**
بسته‌ها عبور نمی‌کنند. راه‌حل: قانون FORWARD.

**اشکال ۴: اشتباه interface.**
NAT روی رابط اشتباه. راه‌حل: بررسی.

**اشکال ۵: rp_filter سخت‌گیر.**
بسته‌ها drop می‌شوند. راه‌حل: تنظیم.

**اشکال ۶: نبود MASQUERADE.**
بسته‌ها برنمی‌گردند. راه‌حل: MASQUERADE.

### نکات مهم (Tips & Tricks)

- **هم forwarding و هم MASQUERADE لازم است.**
- **`iptables-save`/`iptables-restore` برای پایداری.**
- **`iptables -L -v -n` قوانین و شمارنده‌ها را نشان می‌دهد.**
- **`nftables` را برای آینده در نظر بگیرید.**
- **`conntrack` اتصالات را ردیابی می‌کند.**
- **برای firewall پیچیده، `firewalld` راحت‌تر است.**
- **`sysctl -p` تنظیمات را اعمال می‌کند.**
- **`ip route` برای بررسی مسیرها.**
- **`traceroute` برای تست.**
- **`net.ipv4.conf.all.rp_filter=0` برای multi-homing.**
- **`iptables -t nat -L -n -v` قوانین NAT.**
- **`nft list ruleset` قوانین nftables.**

### خلاصه فصل

لینوکس می‌تواند روتر باشد. IP forwarding و NAT (MASQUERADE) پایهٔ مسیریابی‌اند. مسیرهای استاتیک با `ip route` و port forwarding با DNAT انجام می‌شود. قوانین را ذخیره کنید و به ترتیب FORWARD دقت کنید. `nftables` جایگزین مدرن است.

---

## فصل ۹۴: مدیریت سرورهای FTP

### مقدمه

**FTP (File Transfer Protocol)** پروتکل انتقال فایل است. اگرچه SSH جایگزین امن‌تری است، FTP همچنان استفاده می‌شود.

تصور کنید باید فایل‌ها را با یک سیستم قدیمی مبادله کنید که فقط FTP پشتیبانی می‌کند.

### مفاهیم اساسی

**حالت‌های FTP:**

- **Active:** سرور به کلاینت وصل می‌شود.
- **Passive:** کلاینت به سرور وصل می‌شود (مناسب NAT).

**انواع:**

| نوع | امنیت |
|---|---|
| **FTP** | ناامن (متن‌رو) |
| **FTPS** | با TLS |
| **SFTP** | از طریق SSH (امن) |

**سرورها:** vsftpd, proftpd, pure-ftpd.

**پورت‌ها:** ۲۱ (کنترل)، ۲۰ (داده active).

### مثال‌های عملی

**نصب vsftpd:**

```bash
sudo apt install vsftpd
sudo systemctl enable --now vsftpd
```

**پیکربندی (`/etc/vsftpd.conf`):**

```
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
chroot_local_user=YES
allow_writeable_chroot=YES
pasv_enable=YES
pasv_min_port=40000
pasv_max_port=40100
ssl_enable=YES
rsa_cert_file=/etc/ssl/certs/vsftpd.pem
rsa_private_key_file=/etc/ssl/private/vsftpd.key
```

**کلاینت:**

```bash
ftp server
sftp user@server        # SFTP (از SSH)
lftp server             # کلاینت پیشرفته
lftp -u user,pass server
```

**کلاینت خط فرمان:**

```bash
ftp> open server
ftp> user alice
ftp> ls
ftp> get file.txt
ftp> put file.txt
ftp> bye
```

### مقایسه و الگوهای مختلف

**FTP در برابر SFTP:**

| ویژگی | FTP | SFTP |
|---|---|---|
| رمزنگاری | ندارد | دارد |
| پورت | 21 | 22 |
| فایروال | پیچیده | ساده |
| امنیت | ضعیف | قوی |

**حالت‌های انتقال:**

| حالت | توضیح |
|---|---|
| Active | سرور→کلاینت |
| Passive | کلاینت→سرور |

**سرورهای FTP:**

| سرور | ویژگی |
|---|---|
| **vsftpd** | امن، سریع |
| **proftpd** | قابل تنظیم |
| **pure-ftpd** | ساده |

### اشکال شایع

**اشکال ۱: FTP بدون TLS.**
رمزها متن‌رو. راه‌حل: FTPS یا SFTP.

**اشکال ۲: نبود پورت‌های passive.**
فایروال بسته. راه‌حل: باز کردن range.

**اشکال ۳: chroot بدون allow_writeable.**
خطا. راه‌حل: `allow_writeable_chroot=YES`.

**اشکال ۴: anonymous فعال.**
دسترسی ناخواسته. راه‌حل: `anonymous_enable=NO`.

**اشکال ۵: SELinux.**
vsftpd نمی‌تواند بنویسد. راه‌حل: `setsebool`.

**اشکال ۶: اشتباه مجوز.**
کاربر نمی‌تواند آپلود کند. راه‌حل: مجوز.

### نکات مهم (Tips & Tricks)

- **همیشه SFTP یا FTPS را ترجیح دهید.**
- **`pasv_min_port`/`pasv_max_port` را در فایروال باز کنید.**
- **`chroot_local_user` امنیت را بالا می‌برد.**
- **`anonymous_enable=NO` مگر نیاز باشد.**
- **`fail2ban` برای محافظت از brute-force.**
- **SFTP از همان SSH استفاده می‌کند (ساده‌تر).**
- **`user_sub_token`/`local_root` برای پوشهٔ هر کاربر.**
- **`/etc/ftpusers` کاربران ممنوع.**
- **`lftp` کلاینت قدرتمند.**
- **`ssl_enable=YES` برای FTPS.**
- **`tlsv1` فقط برای سازگاری قدیمی.**
- **`pasv_address` برای NAT.**

### خلاصه فصل

FTP پروتکل انتقال فایل است اما ناامن. FTPS (با TLS) و SFTP (از SSH) امن‌ترند. vsftpd محبوب‌ترین سرور است. حالت passive برای NAT لازم است و پورت‌های آن باید باز باشند. `chroot_local_user` امنیت را بالا می‌برد. SFTP را ترجیح دهید.

---

## فصل ۹۵: Secure Shell (SSH)

### مقدمه

**SSH** پروتکل امن برای دسترسی از راه دور است. امروزه استاندارد مدیریت سرورها. تصور کنید هر روز با SSH به سرورها وصل می‌شوید — امنیت آن حیاتی است.

در این فصل پیکربندی امن آن را یاد می‌گیرید.

### مفاهیم اساسی

**اجزا:**

- **sshd:** daemon سرور.
- **ssh:** کلاینت.
- **ssh-keygen:** تولید کلید.
- **ssh-agent:** مدیریت کلید.

**احراز هویت:**

- رمز عبور.
- کلید عمومی/خصوصی (امن‌تر).

**فایل‌های مهم:**

- `/etc/ssh/sshd_config` — پیکربندی سرور.
- `~/.ssh/authorized_keys` — کلیدهای مجاز.
- `~/.ssh/config` — پیکربندی کلاینت.
- `~/.ssh/known_hosts` — میزبان‌های شناخته‌شده.

### مثال‌های عملی

**تولید کلید:**

```bash
ssh-keygen -t ed25519 -C "comment"
# کلیدها: ~/.ssh/id_ed25519 و id_ed25519.pub

# کپی کلید به سرور
ssh-copy-id user@server

# یا دستی
cat ~/.ssh/id_ed25519.pub | ssh user@server "cat >> ~/.ssh/authorized_keys"
```

**پیکربندی امن (`/etc/ssh/sshd_config`):**

```
Port 22
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AllowUsers alice bob
MaxAuthTries 3
X11Forwarding no
ClientAliveInterval 300
ClientAliveCountMax 2
LoginGraceTime 30
```

**پیکربندی کلاینت (`~/.ssh/config`):**

```
Host server1
    HostName 192.168.1.10
    User alice
    Port 22
    IdentityFile ~/.ssh/id_ed25519

Host *
    ServerAliveInterval 60
```

**Tunnel:**

```bash
# Local forward
ssh -L 8080:localhost:80 user@server

# Remote forward
ssh -R 9090:localhost:3000 user@server

# Dynamic (SOCKS)
ssh -D 1080 user@server

# در config
# LocalForward 8080 localhost:80
```

**SSH agent:**

```bash
eval $(ssh-agent)
ssh-add ~/.ssh/id_ed25519
ssh-add -l
```

### مقایسه و الگوهای مختلف

**انواع کلید:**

| الگوریتم | امنیت | طول |
|---|---|---|
| RSA | خوب | 2048+ |
| ECDSA | خوب | 256/384 |
| Ed25519 | عالی (توصیه) | 256 |

**روش‌های احراز:**

| روش | امنیت |
|---|---|
| کلید عمومی | بالا |
| رمز + کلید | بالا |
| فقط رمز | پایین |

**کاربردها:**

| کاربرد | دستور |
|---|---|
| اجرای دستور | `ssh user@host cmd` |
| کپی فایل | `scp` |
| همگام‌سازی | `rsync -e ssh` |
| mount | `sshfs` |
| Tunnel | `-L`/`-R`/`-D` |

### اشکال شایع

**اشکال ۱: PermitRootLogin yes.**
خطر امنیتی. راه‌حل: `no`.

**اشکال ۲: PasswordAuthentication yes.**
brute-force. راه‌حل: `no` (فقط کلید).

**اشکال ۳: مجوز کلید اشتباه.**
`authorized_keys` باید ۶۰۰ باشد. راه‌حل: chmod.

**اشکال ۴: fail2ban نصب‌نشده.**
حمله بی‌پایان. راه‌حل: نصب.

**اشکال ۵: پورت پیش‌فرض ۲۲.**
اسکن زیاد. راه‌حل: پورت دیگر.

**اشکال ۶: نبود timeout.**
اتصال‌های باز می‌مانند. راه‌حل: `ClientAliveInterval`.

### نکات مهم (Tips & Tricks)

- **`PermitRootLogin no` و `PasswordAuthentication no`.**
- **Ed25519 امن‌ترین کلید است.**
- **`~/.ssh` باید ۷۰۰ و `authorized_keys` ۶۰۰.**
- **`fail2ban` را برای SSH نصب کنید.**
- **`ssh -v` برای دیباگ.**
- **`ssh-agent` برای جلوگیری از تایپ مکرر رمز کلید.**
- **`Match` در sshd_config برای قوانین شرطی.**
- **`AllowUsers`/`DenyUsers` دسترسی را محدود می‌کند.**
- **`ssh-copy-id` کلید را کپی می‌کند.**
- **`sshfs` پوشهٔ راه دور را mount می‌کند.**
- **`ProxyJump` برای عبور از bastion.**
- **`sshd -t` پیکربندی را بررسی می‌کند.**

### خلاصه فصل

SSH پروتکل امن دسترسی از راه دور است. احراز با کلید (Ed25519) امن‌تر از رمز است. `PermitRootLogin no` و `PasswordAuthentication no` توصیه می‌شود. Tunnel (L/R/D) کاربردهای پیشرفته می‌دهد. مجوزهای `~/.ssh` را رعایت کنید و `fail2ban` نصب کنید. `ssh-agent` و `~/.ssh/config` کار را راحت می‌کنند.

---

## فصل ۹۶: وظایف امنیتی

### مقدمه

این فصل ابزارها و وظایف امنیتی پیشرفته را پوشش می‌دهد: پایش یکپارچگی، اسکن، SELinux و حسابرسی.

تصور کنید می‌خواهید بدانید کسی فایل‌های سیستمی را تغییر داده یا نه — AIDE این را نشان می‌دهد.

### مفاهیم اساسی

**ابزارها:**

- **SELinux/AppArmor:** کنترل دسترسی اجباری (MAC).
- **AIDE/Tripwire:** پایش یکپارچگی فایل.
- **Nmap:** اسکن پورت.
- **fail2ban:** جلوگیری از brute-force.
- **auditd:** حسابرسی.

**SELinux modeها:**

| Mode | توضیح |
|---|---|
| `enforcing` | اعمال قوانین |
| `permissive` | فقط لاگ |
| `disabled` | غیرفعال |

### مثال‌های عملی

**SELinux:**

```bash
# وضعیت
getenforce
sestatus

# تغییر موقت
sudo setenforce 0    # permissive
sudo setenforce 1    # enforcing

# دائمی (/etc/selinux/config)

# context فایل
ls -Z file
sudo restorecon -Rv /var/www
sudo semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"
sudo restorecon -Rv /web

# boolean
sudo getsebool -a | grep httpd
sudo setsebool -P httpd_can_network_connect on
```

**AppArmor (Debian):**

```bash
sudo aa-status
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx
sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.nginx
```

**پایش یکپارچگی:**

```bash
# AIDE
sudo apt install aide
sudo aideinit
sudo aide --check
sudo aide --update

# Tripwire
sudo tripwire --init
sudo tripwire --check
```

**fail2ban:**

```bash
sudo apt install fail2ban
sudo systemctl enable --now fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

**حسابرسی:**

```bash
sudo apt install auditd
sudo systemctl enable --now auditd
sudo ausearch -m USER_LOGIN
sudo aureport --summary
sudo auditctl -w /etc/passwd -p wa -k passwd_changes
```

### مقایسه و الگوهای مختلف

**MAC systems:**

| سیستم | توزیع |
|---|---|
| SELinux | RHEL/Rocky |
| AppArmor | Debian/Ubuntu |

**ابزارهای امنیتی:**

| ابزار | کار |
|---|---|
| `fail2ban` | جلوگیری brute-force |
| `rkhunter` | تشخیص rootkit |
| `lynis` | حسابرسی امنیت |
| `nmap` | اسکن |
| `clamav` | آنتی‌ویروس |
| `chkrootkit` | تشخیص rootkit |

**SELinux در برابر AppArmor:**

| ویژگی | SELinux | AppArmor |
|---|---|---|
| مدل | برچسب | مسیر |
| پیچیدگی | بالا | کم |
| انعطاف | بالا | متوسط |

### اشکال شایع

**اشکال ۱: غیرفعال کردن SELinux برای حل مشکل.**
سطح امنیت پایین می‌آید. راه‌حل: `permissive` + رفع.

**اشکال ۲: نبود پایش یکپارچگی.**
تغییرات مخرب دیده نمی‌شود. راه‌حل: AIDE.

**اشکال ۳: نبود حسابرسی.**
رویدادهای امنیتی ثبت نمی‌شوند. راه‌حل: auditd.

**اشکال ۴: نصب‌نکردن fail2ban.**
حمله بی‌پایان. راه‌حل: نصب.

**اشکال ۵: context اشتباه.**
سرویس کار نمی‌کند. راه‌حل: `restorecon`.

**اشکال ۶: boolean نادرست.**
سرویس مسدود. راه‌حل: `setsebool`.

### نکات مهم (Tips & Tricks)

- **SELinux را `permissive` کنید (نه `disabled`) برای دیباگ.**
- **`restorecon` context را اصلاح می‌کند.**
- **`fail2ban` را برای SSH و وب نصب کنید.**
- **AIDE را دوره‌ای اجرا کنید (cron).**
- **`auditd` برای انطباق و حسابرسی.**
- **`lynis` ابزار عالی حسابرسی است.**
- **`ausearch`/`aureport` برای تحلیل لاگ audit.**
- **`semanage` برای قوانین SELinux.**
- **`aa-status` وضعیت AppArmor.**
- **`rkhunter`/`chkrootkit` برای rootkit.**
- **`nmap -sV` نسخه‌ها را تشخیص می‌دهد.**
- **`clamav` برای اسکن ویروس.**

### خلاصه فصل

وظایف امنیتی شامل SELinux/AppArmor (MAC)، پایش یکپارچگی (AIDE/Tripwire)، fail2ban، auditd و ابزارهای اسکن است. SELinux را `permissive` کنید (نه `disabled`). `restorecon` و `semanage` context را مدیریت می‌کنند. `fail2ban`، `rkhunter` و `lynis` ابزارهای کلیدی‌اند. `auditd` برای حسابرسی.

---

## فصل ۹۷: OpenVPN

### مقدمه

**VPN (Virtual Private Network)** تونل امن بین شبکه‌ها می‌سازد. **OpenVPN** پیاده‌سازی متن‌باز و محبوب است. تصور کنید می‌خواهید از خانه به شبکهٔ شرکت دسترسی امن داشته باشید.

در این فصل راه‌اندازی آن را یاد می‌گیرید.

### مفاهیم اساسی

**مفاهیم کلیدی:**

- **Tunnel:** کانال رمزنگاری‌شده.
- **CA:** مرجع صدور گواهی.
- **Client/Server:** دو نقش.
- **TUN/TAP:** رابط‌های مجازی.

**مدل‌ها:**

- **Road-warrior:** کلاینت به سرور.
- **Site-to-site:** شبکه به شبکه.

**پورت پیش‌فرض:** ۱۱۹۴.

### مثال‌های عملی

**نصب:**

```bash
sudo apt install openvpn easy-rsa
```

**ساخت PKI:**

```bash
make-cadir ~/openvpn-ca
cd ~/openvpn-ca
source vars

./clean-all
./build-ca
./build-key-server server
./build-key client1
./build-dh
openvpn --genkey --secret keys/ta.key
```

**پیکربندی سرور (`/etc/openvpn/server.conf`):**

```
port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh2048.pem
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
push "redirect-gateway def1"
push "dhcp-option DNS 8.8.8.8"
keepalive 10 120
tls-auth ta.key 0
cipher AES-256-CBC
user nobody
group nogroup
persist-key
persist-tun
status openvpn-status.log
verb 3
```

**راه‌اندازی:**

```bash
sudo systemctl enable --now openvpn@server
sudo systemctl status openvpn@server

# IP forwarding
echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/99-vpn.conf
sudo sysctl -p

# NAT
sudo iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
```

**کلاینت:**

```bash
sudo openvpn --config client.ovpn
```

### مقایسه و الگوهای مختلف

**پروتکل‌های VPN:**

| پروتکل | ویژگی |
|---|---|
| **OpenVPN** | بالغ، انعطاف |
| **WireGuard** | مدرن، سریع |
| **IPSec** | استاندارد، پیچیده |
| **L2TP** | ساده‌تر |

**OpenVPN در برابر WireGuard:**

| ویژگی | OpenVPN | WireGuard |
|---|---|---|
| سرعت | خوب | عالی |
| سادگی | پیچیده | ساده |
| بلوغ | بالا | جدید |
| کد | زیاد | کم |

**TCP در برابر UDP:**

| ویژگی | UDP | TCP |
|---|---|---|
| سرعت | سریع‌تر | کندتر |
| اطمینان | از tunnel | از TCP |
| توصیه | ✅ | در فایروال سخت |

### اشکال شایع

**اشکال ۱: CA اشتباه.**
کلاینت وصل نمی‌شود. راه‌حل: CA درست.

**اشکال ۲: فایروال پورت ۱۱۹۴.**
VPN کار نمی‌کند. راه‌حل: باز کردن.

**اشکال ۳: نبود forwarding.**
ترافیک عبور نمی‌کند. راه‌حل: فعال کنید.

**اشکال ۴: زمان اشتباه.**
گواهی معتبر نیست. راه‌حل: NTP.

**اشکال ۵: نبود NAT.**
کلاینت اینترنت ندارد. راه‌حل: MASQUERADE.

**اشکال ۶: tls-auth اشتباه.**
اتصال رد می‌شود. راه‌حل: کلید درست.

### نکات مهم (Tips & Tricks)

- **`redirect-gateway` کل ترافیک را از VPN می‌فرستد.**
- **`tls-auth` امنیت را بالا می‌برد.**
- **برای کارایی، UDP را ترجیح دهید.**
- **WireGuard را برای راه‌حل مدرن‌تر در نظر بگیرید.**
- **`push "dhcp-option DNS"` DNS را تنظیم می‌کند.**
- **زمان سیستم باید دقیق باشد (گواهی).**
- **`openvpn --status` وضعیت اتصال.**
- **`verb 3` برای دیباگ.**
- **`easy-rsa` برای PKI.**
- **`--genkey` برای کلید ta.**
- **`persist-key`/`persist-tun` پایدار.**
- **`client-config-dir` برای کلاینت خاص.**

### خلاصه فصل

VPN تونل امن می‌سازد. OpenVPN با PKI (CA، گواهی سرور/کلاینت، DH) کار می‌کند. `server.conf` تونل و redirect-gateway را تنظیم می‌کند. `tls-auth` امنیت را بالا می‌برد. WireGuard جایگزین مدرن‌تر است. پورت و forwarding و NAT را فراموش نکنید.

---

## منابع و مراجع

- man pages: `man sshd_config`, `man iptables`, `man vsftpd.conf`, `man openvpn`, `man semanage`
- OpenSSH: https://www.openssh.com/manual.html
- SELinux: https://selinuxproject.org
- OpenVPN HOWTO: https://openvpn.net/community-resources/
- WireGuard: https://www.wireguard.com

**بعدی:** [فایل ۲۸ — آماده‌سازی آزمون و آزمون‌های تمرینی](28-Exam-Preparation-and-Mock-Tests.md)
