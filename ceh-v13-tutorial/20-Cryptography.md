# رمزنگاری (Cryptography)

**مدت یادگیری**: ۴ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۲ (شبکه) و ۰۴ (مفاهیم امنیت)

---

## فهرست محتوا

- [فصل ۱: مقدمه رمزنگاری](#فصل-۱-مقدمه-رمزنگاری)
- [فصل ۲: رمزنگاری متقارن و نامتقارن](#فصل-۲-رمزنگاری-متقارن-و-نامتقارن)
- [فصل ۳: هش و امضای دیجیتال](#فصل-۳-هش-و-امضای-دیجیتال)
- [فصل ۴: PKI، SSL/TLS و گواهی](#فصل-۴-pki-ssltls-و-گواهی)
- [فصل ۵: حملات رمزنگاری و پساکوانتومی](#فصل-۵-حملات-رمزنگاری-و-پساکوانتومی)
- [فصل ۶: آزمایشگاه و اقدامات متقابل](#فصل-۶-آزمایشگاه-و-اقدامات-متقابل)

---

## فصل ۱: مقدمه رمزنگاری

### مقدمه

**رمزنگاری (Cryptography)** علم تبدیل اطلاعات خوانا به شکل غیرخوانا برای محافظت از آن است. رمزنگاری ستون **محرمانگی** و **یکپارچگی** است.

> **تشبیه:** رمزنگاری مثل گذاشتن نامه در گاوصندوق است؛ فقط دارنده کلید می‌تواند بازش کند.

### مفاهیم اساسی

**اصطلاحات:**
- **Plaintext:** متن اصلی.
- **Ciphertext:** متن رمزشده.
- **Cipher:** الگوریتم رمزنگاری.
- **Key:** کلید رمزنگاری.
- **Encryption / Decryption:** رمزگذاری / رمزگشایی.

**هدف رمزنگاری:** محرمانگی، یکپارچگی، احراز هویت، انکارناپذیری.

**دو خانواده اصلی:** متقارن (Symmetric) و نامتقارن (Asymmetric).

### نکات مهم (Tips & Tricks)

- **Kerckhoffs's Principle:** امنیت باید فقط به **کلید** وابسته باشد، نه پنهان بودن الگوریتم.
- رمزنگاری **جایگزین** سایر کنترل‌ها نیست؛ مکمل است.
- الگوریتم ضعیف یا مدیریت کلید ضعیف = شکست.

### خلاصه فصل

- رمزنگاری: تبدیل متن خوانا به غیرخوانا.
- اجزا: Plaintext، Ciphertext، Cipher، Key.
- دو خانواده: متقارن و نامتقارن.

---

## فصل ۲: رمزنگاری متقارن و نامتقارن

### مقدمه

دو رویکرد اصلی رمزنگاری با مزایا و معایب متفاوت.

### مفاهیم اساسی

**متقارن (Symmetric):** یک کلید برای رمز و رمزگشایی.
- **مزیت:** سریع.
- **عیب:** توزیع کلید چالش‌برانگیز.
- الگوریتم‌ها: **AES**، DES، 3DES، Blowfish، RC4.

**نامتقارن (Asymmetric):** دو کلید (عمومی و خصوصی).
- **مزیت:** حل مشکل توزیع کلید.
- **عیب:** کند.
- الگوریتم‌ها: **RSA**، ECC، Diffie-Hellman، DSA.

**مقایسه:**

| ویژگی | متقارن | نامتقارن |
|-------|--------|----------|
| تعداد کلید | ۱ | ۲ |
| سرعت | سریع | کند |
| کاربرد | حجم بالا | تبادل کلید، امضا |
| مثال | AES-256 | RSA-2048 |

**ترکیب در عمل:** TLS از نامتقارن برای تبادل کلید و از متقارن برای داده استفاده می‌کند (Hybrid).

### مثال عملی

```bash
# === متقارن: AES با openssl ===
# رمزگذاری
openssl enc -aes-256-cbc -salt -in secret.txt -out secret.enc -k MyPassword
# رمزگشایی
openssl enc -d -aes-256-cbc -in secret.enc -out secret.txt -k MyPassword

# === نامتقارن: RSA ===
# تولید کلید خصوصی
openssl genrsa -out private.pem 2048
# استخراج کلید عمومی
openssl rsa -in private.pem -pubout -out public.pem
# رمزگذاری با کلید عمومی
openssl rsautl -encrypt -inkey public.pem -pubin -in secret.txt -out secret.rsa
# رمزگشایی با کلید خصوصی
openssl rsautl -decrypt -inkey private.pem -in secret.rsa -out secret.txt
```

### اشکال شایع

- **مشکل:** اشتباه گرفتن کلید عمومی و خصوصی → عمومی برای رمزگذاری/تأیید، خصوصی برای رمزگشایی/امضا.
- **مشکل:** استفاده از DES/RC4 → منسوخ؛ از AES استفاده کنید.
- **مشکل:** RSA کوچک (1024) → حداقل 2048 بیت.

### نکات مهم (Tips & Tricks)

- **AES-256** استاندارد فعلی متقارن است.
- **RSA-2048+** یا **ECC** برای نامتقارن.
- **Diffie-Hellman** برای تبادل امن کلید.
- **Hybrid Encryption** در عمل رایج است.

### خلاصه فصل

- متقارن: یک کلید، سریع، مشکل توزیع.
- نامتقارن: دو کلید، کند، حل توزیع.
- TLS ترکیبی از هر دو است.

---

## فصل ۳: هش و امضای دیجیتال

### مقدمه

**هش** یک «اثر انگشت» دیجیتال است و **امضای دیجیتال** اثبات اصالت.

### مفاهیم اساسی

**توابع هش (Hash):** ورودی با هر طول → خروجی با طول ثابت.
- **ویژگی‌ها:** یک‌طرفه (غیرقابل بازگشت)، حساس به تغییر (Avalanche)، بدون برخورد.
- **الگوریتم‌ها:** MD5 (شکسته)، SHA-1 (ضعیف)، **SHA-256/SHA-3** (امن)، bcrypt/Argon2 (پسورد).

**HMAC:** هش + کلید مخفی (برای یکپارچگی پیام).

**امضای دیجیتال:** هش پیام + رمزگذاری با کلید خصوصی.
- تضمین: اصالت، یکپارچگی، انکارناپذیری.

### مثال عملی

```bash
# === هش ===
echo -n "hello" | md5sum       # ضعیف (فقط برای بررسی)
echo -n "hello" | sha256sum    # امن

# تغییر یک کاراکتر → هش کاملاً عوض می‌شود
echo -n "hello" | sha256sum
echo -n "hellp" | sha256sum    # کاملاً متفاوت

# === HMAC ===
echo -n "message" | openssl dgst -sha256 -hmac "secretkey"

# === امضای دیجیتال ===
# امضا
openssl dgst -sha256 -sign private.pem -out signature.bin document.txt
# تأیید
openssl dgst -sha256 -verify public.pem -signature signature.bin document.txt

# === هش پسورد (امن) ===
# با bcrypt در Python:
# import bcrypt; bcrypt.hashpw(password, bcrypt.gensalt())
```

### اشکال شایع

- **مشکل:** استفاده از MD5/SHA-1 برای امنیت → شکسته‌اند؛ از SHA-256/3.
- **مشکل:** ذخیره پسورد با هش ساده → از bcrypt/Argon2 با Salt استفاده کنید.
- **مشکل:** مقایسه هش با `==` → از مقایسه زمان‌ثابت استفاده کنید (Timing Attack).

### نکات مهم (Tips & Tricks)

- **هش ≠ رمزنگاری:** هش یک‌طرفه است، رمزنگاری برگشت‌پذیر.
- **Salt** جلوی Rainbow Table را می‌گیرد.
- برای پسورد از **bcrypt**، **scrypt** یا **Argon2** استفاده کنید.
- **MD5 و SHA-1** فقط برای بررسی یکپارچگی غیرامنیتی.

### خلاصه فصل

- هش: اثر انگشت، یک‌طرفه، حساس به تغییر.
- HMAC: هش با کلید.
- امضای دیجیتال: اصالت + یکپارچگی + انکارناپذیری.

---

## فصل ۴: PKI، SSL/TLS و گواهی

### مقدمه

**PKI (Public Key Infrastructure)** چارچوب اعتماد برای کلیدهای عمومی است و **TLS** امنیت وب را تأمین می‌کند.

### مفاهیم اساسی

**اجزای PKI:**
- **CA (Certificate Authority):** مرجع صدور گواهی.
- **Certificate:** گواهی دیجیتال (هویت + کلید عمومی + امضای CA).
- **Registration Authority (RA):** تأیید هویت.
- **CRL / OCSP:** بررسی ابطال گواهی.

**TLS Handshake (ساده‌شده):**
```
Client → Server: ClientHello
Server → Client: ServerHello + Certificate
Client: تأیید گواهی با CA
Client ↔ Server: تبادل کلید (Diffie-Hellman)
Client ↔ Server: ارتباط رمز (متقارن)
```

**انواع گواهی:** DV (دامنه)، OV (سازمان)، EV (تأیید گسترده)، Wildcard، SAN، Self-signed.

### مثال عملی

```bash
# === مشاهده گواهی یک سایت ===
openssl s_client -connect example.com:443 -servername example.com
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -text

# === ساخت گواهی Self-signed ===
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes

# === بررسی زنجیره گواهی ===
openssl verify -CAfile ca.pem cert.pem

# === بررسی TLS یک سرور ===
nmap --script ssl-enum-ciphers -p 443 example.com
# یا: testssl.sh example.com
```

### اشکال شایع

- **مشکل:** گواهی Self-signed هشدار می‌دهد → برای محیط داخلی است، نه عمومی.
- **مشکل:** گواهی منقضی شده → به‌روزرسانی خودکار.
- **مشکل:** زنجیره ناقص → گواهی‌های Intermediate را اضافه کنید.

### نکات مهم (Tips & Tricks)

- **CA** پایه اعتماد است؛ اگر CA آلوده شود، همه گواهی‌ها بی‌اعتبار می‌شوند.
- **Certificate Pinning** جلوی MITM با گواهی جعلی را می‌گیرد.
- **Let's Encrypt** گواهی رایگان و خودکار ارائه می‌دهد.
- **TLS 1.3** سریع‌تر و امن‌تر از 1.2 است.

### خلاصه فصل

- PKI: CA، Certificate، CRL/OCSP.
- TLS Handshake: تبادل کلید + ارتباط رمز.
- Certificate Pinning دفاع مؤثر است.

---

## فصل ۵: حملات رمزنگاری و پساکوانتومی

### مقدمه

رمزنگاری هم هدف حمله است — از ضعف الگوریتم تا خطای پیاده‌سازی.

### مفاهیم اساسی

**انواع حملات رمزنگاری:**
- **Brute-force:** امتحان همه کلیدها.
- **Known/Chosen Plaintext:** استفاده از متن معلوم.
- **Downgrade Attack:** تنزل به الگوریتم ضعیف.
- **Birthday Attack:** یافتن برخورد هش.
- **Man-in-the-Middle:** رهگیری و جایگزینی کلید.
- **Padding Oracle:** سوءاستفاده از خطای Padding.
- **Side-Channel:** تحلیل زمان/مصرف برق.
- **Weak Key:** کلید ضعیف یا پیش‌فرض.
- **Implementation Flaws:** خطای پیاده‌سازی (مثل Heartbleed).

**رمزنگاری پساکوانتومی:** کامپیوترهای کوانتومی RSA و ECC را می‌شکنند (الگوریتم Shor). الگوریتم‌های مقاوم: Lattice-based، CRYSTALS-Kyber، Dilithium.

**نقش AI:** تحلیل الگوهای رمزنگاری ضعیف، کمک به cryptanalysis.

### مثال عملی

```bash
# === تحلیل الگوریتم ضعیف ===
nmap --script ssl-enum-ciphers -p 443 example.com
# اگر RC4 یا DES دید → آسیب‌پذیر

# === حمله به WEP (مثال) ===
# در فایل ۱۷ دیدیم

# === حملات هش ===
hashcat -m 0 hashes.txt rockyou.txt   # MD5 شکسته

# === Heartbleed (نمونه تاریخی) ===
nmap --script ssl-heartbleed -p 443 target
```

### اشکال شایع

- **مشکل:** استفاده از الگوریتم منسوخ → به AES/SHA-256 ارتقا دهید.
- **مشکل:** کلید پیش‌فرض → کلید تصادفی و مدیریت‌شده.
- **مشکل:** بی‌توجهی به پساکوانتومی → برنامه‌ریزی مهاجرت (Crypto Agility).

### نکات مهم (Tips & Tricks)

- **Downgrade Attack** با غیرفعال‌سازی الگوریتم‌های قدیمی خنثی می‌شود.
- **Padding Oracle** نیاز به پیام خطای متفاوت دارد؛ خطای یکسان بدهید.
- **Post-Quantum** روند آینده است؛ آماده مهاجرت باشید.
- **Crypto Agility:** سیستم باید بدون بازطراحی، الگوریتم را عوض کند.

### خلاصه فصل

- حملات: Brute-force، Downgrade، MITM، Padding Oracle، Side-Channel.
- الگوریتم‌های منسوخ خطرناک‌اند.
- پساکوانتومی چالش آینده است.

---

## فصل ۶: آزمایشگاه و اقدامات متقابل

### مقدمه

یک آزمایشگاه عملی رمزنگاری و تحلیل TLS.

### مثال عملی — آزمایشگاه

```bash
# === ۱. رمزگذاری و رمزگشایی ===
echo "Top Secret Data" > secret.txt
openssl enc -aes-256-cbc -salt -pbkdf2 -in secret.txt -out secret.enc -k StrongPass123
openssl enc -d -aes-256-cbc -pbkdf2 -in secret.enc -out decrypted.txt -k StrongPass123
diff secret.txt decrypted.txt   # باید یکسان باشد

# === ۲. هش و بررسی یکپارچگی ===
sha256sum secret.txt > secret.sha256
# تغییر فایل و بررسی
echo "tampered" >> secret.txt
sha256sum -c secret.sha256   # خطا → فایل تغییر کرده!

# === ۳. امضای دیجیتال ===
openssl genrsa -out priv.pem 2048
openssl rsa -in priv.pem -pubout -out pub.pem
openssl dgst -sha256 -sign priv.pem -out doc.sig secret.txt
openssl dgst -sha256 -verify pub.pem -signature doc.sig secret.txt

# === ۴. تحلیل TLS یک سرور ===
openssl s_client -connect example.com:443 -tls1_3 </dev/null 2>/dev/null | head -20

# === ۵. ابزار testssl.sh ===
git clone https://github.com/drwetter/testssl.sh
./testssl.sh/testssl.sh example.com
```

### اشکال شایع

- **مشکل:** `openssl enc` رمز اشتباه می‌دهد → مطمئن شوید PBKDF2 و الگوریتم یکسان است.
- **مشکل:** گواهی Self-signed در مرورگر هشدار می‌دهد → طبیعی است.

### نکات مهم (Tips & Tricks)

- **PBKDF2** برای مشتق‌سازی کلید از رمز عبور.
- **SHA-256** برای بررسی یکپارچگی.
- **testssl.sh** ابزار جامع تحلیل TLS.
- الگوریتم‌های منسوخ را غیرفعال کنید.

### اقدامات متقابل (Countermeasures)

| تهدید | راهکار |
|-------|--------|
| Brute-force | کلید بلند (AES-256، RSA-2048+) |
| Downgrade | غیرفعال‌سازی پروتکل/الگوریتم قدیمی |
| MITM | Certificate Pinning، HSTS |
| Weak Hash | SHA-256/3، bcrypt/Argon2 |
| Padding Oracle | پیام خطای یکسان، AEAD |
| Side-Channel | پیاده‌سازی زمان‌ثابت |
| Quantum | مهاجرت به پساکوانتومی، Crypto Agility |

### خلاصه فصل

- آزمایشگاه: رمزگذاری، هش، امضا، تحلیل TLS.
- OpenSSL و testssl.sh ابزارهای اصلی.
- دفاع: کلید قوی، الگوریتم مدرن، Pinning.

---

## منابع و مراجع

- **NIST Cryptographic Standards:** https://csrc.nist.gov/
- **OpenSSL:** https://www.openssl.org/
- **Let's Encrypt:** https://letsencrypt.org/
- **testssl.sh:** https://github.com/drwetter/testssl.sh
- **NIST Post-Quantum:** https://csrc.nist.gov/projects/post-quantum-cryptography
- **کتاب:** "Applied Cryptography" — Bruce Schneier
- **OWASP Cryptographic Storage:** https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html
