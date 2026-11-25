# فایل ۱۰ — رابط‌های کاربری و دسکتاپ

**مدت یادگیری**: ۴ ساعت
**سطح سختی**: متوسط
**پیش‌نیاز**: فایل ۰۹ (شل)

---

## فهرست محتوا

- [فصل ۳۸: نصب و پیکربندی X11](#فصل-۳۸-نصب-و-پیکربندی-x11)
- [فصل ۳۹: دسکتاپ‌های گرافیکی](#فصل-۳۹-دسکتاپهای-گرافیکی)
- [فصل ۴۰: دسترس‌پذیری](#فصل-۴۰-دسترس‌پذیری)

---

## فصل ۳۸: نصب و پیکربندی X11

### مقدمه

**X Window System** (که معمولاً **X11** نامیده می‌شود) سیستم پایهٔ گرافیکی در لینوکس است. حتی اگر سرور شما فقط متنی باشد، دانستن X11 برای آزمون LPIC لازم است.

تصور کنید می‌خواهید یک برنامهٔ گرافیکی را از سرور روی کامپیوتر خود اجرا کنید. این کار با X11 و X11 forwarding انجام می‌شود. در این فصل اصول آن را یاد می‌گیرید.

### مفاهیم اساسی

**معماری X11:** مدل **client-server**:

- **X Server:** روی ماشینی که نمایشگر دارد اجرا می‌شود.
- **X Client:** برنامه‌های گرافیکی که به سرور وصل می‌شوند.

**نکتهٔ مهم:** برخلاف نامش، X Server روی ماشین کاربر (نمایشگر) اجرا می‌شود، نه سرور. این برعکس انتظار است.

**مفاهیم کلیدی:**

- **Display:** صفحهٔ X، به‌صورت `:0` (اولین نمایشگر).
- **DISPLAY:** متغیری که کلاینت را به سرور وصل می‌کند (مثل `:0` یا `host:0`).
- **Xorg:** پیاده‌سازی استاندارد X Server.
- **X11 forwarding:** ارسال خروجی گرافیکی از راه دور.

**فایل پیکربندی:** `/etc/X11/xorg.conf` (اغلب امروز خودکار است).

**مقایسه با Wayland:**

| ویژگی | X11 | Wayland |
|---|---|---|
| سن | قدیمی (۱۹۸۴) | مدرن |
| معماری | client-server شبکه‌ای | محلی |
| امنیت | ضعیف‌تر | بهتر |
| پایداری | بسیار | در حال بلوغ |
| ابزار | `xrandr`, `xinput` | `wlr-randr` |

### مثال‌های عملی

**نصب X:**

```bash
# Debian/Ubuntu
sudo apt install xorg xinit x11-apps

# RHEL/Rocky
sudo dnf install xorg-x11-server-Xorg xorg-x11-apps

# تست
xeyes            # یک برنامهٔ سادهٔ X
xclock
xcalc
```

**اجرای برنامهٔ X از راه دور:**

```bash
# روی سرور (کلاینت) DISPLAY را تنظیم کنید
export DISPLAY=192.168.1.100:0

# یا از طریق SSH با X11 forwarding
ssh -X user@server
# سپس روی سرور:
xeyes

# با اعتماد کامل (کمتر امن)
ssh -Y user@server
```

**اطلاعات X:**

```bash
echo $DISPLAY
xdpyinfo           # اطلاعات نمایشگر
xrandr             # تنظیمات صفحه
xrandr --listmonitors
xrandr --output HDMI-1 --mode 1920x1080
```

**مدیریت دسترسی:**

```bash
# دیدن
xhost

# اجازه دادن به یک میزبان
xhost +192.168.1.100

# اجازه به همه (ناامن!)
xhost +

# لغو
xhost -192.168.1.100
xhost -
```

**راه‌اندازی X:**

```bash
# از ترمینال
startx

# یا با xinit
xinit /usr/bin/xterm -- :1
```

### مقایسه و الگوهای مختلف

**نمایشگر و متغیر DISPLAY:**

| مقدار | معنا |
|---|---|
| `:0` | نمایشگر اول محلی |
| `:1` | نمایشگر دوم محلی |
| `host:0` | نمایشگر روی میزبان دیگر |
| `host:0.0` | صفحهٔ اول نمایشگر |

**روش‌های X11 forwarding:**

| روش | امنیت | کاربرد |
|---|---|---|
| `ssh -X` | بالا | توصیه‌شده |
| `ssh -Y` | کم | برنامه‌های خاص |
| `xhost` | کم | شبکهٔ محلی |
| VNC | متوسط | دسکتاپ کامل |

**ابزارهای X:**

| ابزار | کار |
|---|---|
| `xdpyinfo` | اطلاعات نمایشگر |
| `xrandr` | تنظیمات صفحه |
| `xinput` | دستگاه‌های ورودی |
| `xset` | تنظیمات X |
| `xwininfo` | اطلاعات پنجره |
| `xev` | رویدادهای ورودی |

### اشکال شایع

**اشکال ۱: نبود `DISPLAY`.**
برنامه‌های گرافیکی اجرا نمی‌شوند. راه‌حل: `export DISPLAY=:0`.

**اشکال ۲: `xorg.conf` دستی اشتباه.**
سیستم گرافیکی بالا نمی‌آید؛ حذفش کنید تا خودکار شود. راه‌حل: حذف فایل.

**اشکال ۳: مشکل X11 forwarding.**
در سرور `X11Forwarding yes` در sshd_config لازم است. راه‌حل: فعال کنید.

**اشکال ۴: درایور گرافیکی نادرست.**
صفحه سیاه یا رزولوشن اشتباه. راه‌حل: درایور درست.

**اشکال ۵: `xhost +` بدون محدودیت.**
امنیت به خطر می‌افتد. راه‌حل: از `ssh -X` استفاده کنید.

**اشکال ۶: X server روی ماشین اشتباه.**
گیج شدن با مدل client-server. راه‌حل: X server روی نمایشگر است.

### نکات مهم (Tips & Tricks)

- **`xrandr` برای تنظیم رزولوشن بدون ویرایش فایل.**
- **`xdpyinfo` اطلاعات دقیق نمایشگر.**
- **`startx` X را از ترمینال اجرا می‌کند.**
- **برای forwarding، `ssh -X` (امن) یا `ssh -Y` (اعتماد کامل).**
- **`xhost` کنترل دسترسی را مدیریت می‌کند (احتیاط امنیتی).**
- **`Xephyr` یک X server تودرتو برای تست.**
- **`Xvfb` X server مجازی (بدون نمایشگر).**
- **`xauth` مدیریت کوکی‌های احراز.**
- **در VM، نصب Guest Additions رزولوشن را درست می‌کند.**
- **Wayland جایگزین مدرن است اما X11 همچنان برای سازگاری.**
- **`xdpyinfo | grep dimensions` ابعاد نمایشگر.**
- **`xset q` تنظیمات فعلی X.**

### خلاصه فصل

X11 سیستم پایهٔ گرافیکی لینوکس با معماری client-server است. X Server روی ماشین نمایشگر و X Client برنامه‌های گرافیکی هستند. `DISPLAY` آن‌ها را وصل می‌کند. Xorg پیاده‌سازی استاندارد است و `xorg.conf` پیکربندی (اغلب خودکار) آن. `xrandr`, `xdpyinfo`, `xhost` ابزارهای اصلی‌اند. SSH X11 forwarding (`ssh -X`) راه امن اجرای برنامه‌های گرافیکی از راه دور است. Wayland جایگزین مدرن‌تر است.

---

## فصل ۳۹: دسکتاپ‌های گرافیکی

### مقدمه

روی X11 (یا Wayland)، یک **محیط دسکتاپ (Desktop Environment)** یا **مدیر پنجره (Window Manager)** اجرا می‌شود که رابط کاربری کامل می‌سازد.

تصور کنید می‌خواهید یک VM سبک برای تمرین گرافیکی بسازید — انتخاب دسکتاپ درست بر کارایی اثر می‌گذارد. این فصل انواع دسکتاپ و انتخاب درست را پوشش می‌دهد.

### مفاهیم اساسی

**تفکیک مفاهیم:**

- **Display Manager:** صفحهٔ ورود (login) — مثل GDM, LightDM, SDDM.
- **Desktop Environment:** مجموعهٔ کامل — مثل GNOME, KDE, XFCE.
- **Window Manager:** فقط مدیریت پنجره‌ها — مثل i3, Openbox.

**دسکتاپ‌های محبوب:**

| دسکتاپ | ویژگی | منابع | مناسب |
|---|---|---|---|
| **GNOME** | مدرن، ساده | سنگین | دسکتاپ مدرن |
| **KDE Plasma** | قابل تنظیم | متوسط | کاربر قدرتمند |
| **XFCE** | سبک | کم | VM، سیستم قدیمی |
| **LXDE/LXQt** | بسیار سبک | خیلی کم | سیستم خیلی قدیمی |
| **MATE** | سنتی | متوسط | کاربر ویندوزی |
| **Cinnamon** | شبیه ویندوز | متوسط | کاربر ویندوزی |

**مدیران پنجره (بدون دسکتاپ کامل):**

| WM | ویژگی |
|---|---|
| **i3** | کاشی‌کاری (tiling) |
| **Openbox** | سبک، شناور |
| **Awesome** | کاشی‌کاری، قابل تنظیم |
| **Xfwm4** | WM پیش‌فرض XFCE |

### مثال‌های عملی

**نصب دسکتاپ:**

```bash
# XFCE (سبک)
sudo apt install xfce4 xfce4-goodies

# GNOME
sudo apt install gnome

# KDE
sudo apt install kde-plasma-desktop

# LXQt (بسیار سبک)
sudo apt install lxqt

# شروع
startxfce4    # یا انتخاب از display manager
```

**مدیریت Display Manager:**

```bash
# وضعیت
systemctl status gdm3
systemctl status lightdm
systemctl status sddm

# تغییر پیش‌فرض
sudo dpkg-reconfigure lightdm

# فعال/غیرفعال
sudo systemctl disable gdm3
sudo systemctl set-default multi-user.target   # بوت متنی

# برگشت به گرافیکی
sudo systemctl set-default graphical.target
```

**تنظیمات GNOME (خط فرمان):**

```bash
# تنظیمات کلی
gsettings list-schemas
gsettings get org.gnome.desktop.interface gtk-theme
gsettings set org.gnome.desktop.interface gtk-theme 'Adwaita-dark'

# پس‌زمینه
gsettings set org.gnome.desktop.background picture-uri 'file:///path/img.jpg'

# اندازهٔ فونت
gsettings set org.gnome.desktop.interface text-scaling-factor 1.2
```

### مقایسه و الگوهای مختلف

**انتخاب دسکتاپ:**

| سناریو | پیشنهاد |
|---|---|
| سرور با GUI کم | XFCE, LXQt |
| دسکتاپ مدرن | GNOME, KDE |
| سیستم قدیمی | LXDE |
| کاربر ویندوزی | Cinnamon, MATE |
| حرفه‌ای | KDE, GNOME |

**توصیه برای LPIC:** با XFCE یا LXQt آشنا باشید (سبک، برای VM مناسب).

**Display Managerها:**

| DM | دسکتاپ | ویژگی |
|---|---|---|
| GDM | GNOME | مدرن |
| LightDM | چندتایی | سبک |
| SDDM | KDE | Qt-based |
| LXDM | LXDE | بسیار سبک |

**دسکتاپ کامل در برابر WM:**

| ویژگی | Desktop Environment | Window Manager |
|---|---|---|
| اجزا | کامل (پنل، فایل‌منیجر) | فقط پنجره |
| منابع | زیاد | کم |
| مناسب | کاربر عادی | حرفه‌ای |

### اشکال شایع

**اشکال ۱: نصب چند دسکتاپ و تداخل.**
گاهی منوی ورود شلوغ می‌شود. راه‌حل: یکی را انتخاب کنید.

**اشکال ۲: منابع ناکافی.**
GNOME روی VM با ۲ گیگ رم کند است. راه‌حل: XFCE/LXQt.

**اشکال ۳: display manager غیرفعال.**
سیستم به حالت متنی بوت می‌شود. راه‌حل: `systemctl enable gdm3`.

**اشکال ۴: اشتباه در `set-default`.**
ممکن است به گرافیکی برنگردید. راه‌حل: `graphical.target`.

**اشکال ۵: درایور گرافیکی نبودن.**
صفحه سیاه. راه‌حل: درایور نصب کنید.

**اشکال ۶: زبان اشتباه صفحه‌کلید در DM.**
تایپ سخت می‌شود. راه‌حل: layout درست.

### نکات مهم (Tips & Tricks)

- **برای VM سبک، XFCE یا LXQt نصب کنید.**
- **`systemctl set-default graphical.target` بوت گرافیکی را فعال می‌کند.**
- **`sudo systemctl restart gdm3` برای restart display manager.**
- **محیط دسکتاپ را می‌توان در صفحهٔ ورود انتخاب کرد.**
- **برای سرور تولیدی، GUI نصب نکنید (امنیت و منابع).**
- **`gsettings` راه اسکریپت‌پذیر تنظیمات است.**
- **`startx` بدون display manager هم کار می‌کند.**
- **`~/.xinitrc` برنامهٔ startup X را تعیین می‌کند.**
- **`xrdp` دسترسی دسکتاپ از راه دور (RDP).**
- **`VNC` جایگزین سبک‌تر.**
- **`lxappearance` ظاهر LXDE/LXQt را تنظیم می‌کند.**
- **`xfce4-settings-manager` تنظیمات XFCE.**

### خلاصه فصل

دسکتاپ گرافیکی از سه لایه ساخته می‌شود: Display Manager (ورود)، Desktop Environment (مجموعه) و Window Manager (پنجره‌ها). دسکتاپ‌های محبوب: GNOME، KDE، XFCE، LXQt. برای VM سبک XFCE/LXQt توصیه می‌شود. `systemctl set-default graphical.target` بوت گرافیکی را فعال می‌کند. `gsettings` تنظیمات GNOME را اسکریپت‌پذیر می‌کند.

---

## فصل ۴۰: دسترس‌پذیری

### مقدمه

**دسترس‌پذیری (Accessibility)** یعنی سیستم‌عامل برای همه قابل استفاده باشد — از جمله افراد با ناتوانی‌های بینایی، شنوایی یا حرکتی. LPIC به این موضوع اهمیت می‌دهد.

تصور کنید کاربری با ناتوانی بینایی نیاز به صفحه‌خوان دارد یا کاربری با مشکل حرکتی به کیبورد روی صفحه نیاز دارد. این فصل ابزارهای دسترس‌پذیری را معرفی می‌کند.

### مفاهیم اساسی

**ابزارهای دسترس‌پذیری:**

| ابزار | کاربرد | ناتوانی |
|---|---|---|
| **Orca** | صفحه‌خوان (screen reader) | بینایی |
| **BRLTTY** | پشتیبانی از نمایشگر بریل | بینایی |
| **Onboard** | کیبورد روی صفحه | حرکتی |
| **Dasher** | ورودی جایگزین | حرکتی |
| **Magnifier** | بزرگ‌نمایی | بینایی |
| **High Contrast** | کنتراست بالا | بینایی |
| **Sticky Keys** | کلیدهای چسبنده | حرکتی |
| **Slow Keys** | کلیدهای کند | حرکتی |

**محل تنظیمات:** اکثر دسکتاپ‌ها بخش «Accessibility» یا «Universal Access» دارند.

**دسته‌بندی ناتوانی‌ها:**

| ناتوانی | ابزارها |
|---|---|
| بینایی | Orca, Magnifier, High Contrast, BRLTTY |
| شنوایی | زیرنویس، اعلان بصری |
| حرکتی | Onboard, Dasher, Sticky Keys |
| شناختی | رابط ساده |

### مثال‌های عملی

**اجرای Orca:**

```bash
# نصب
sudo apt install orca

# اجرا
orca

# از خط فرمان با گزینه
orca -s          # تنظیمات
orca --help
```

**پیکربندی تنظیمات:**

```bash
# GNOME
gsettings set org.gnome.desktop.a11y always-show-universal-access-status true

# تنظیمات بزرگ‌نمایی
gsettings set org.gnome.desktop.a11y.magnifier mag-factor 2.0

# صفحه‌خوان
gsettings set org.gnome.desktop.a11y.applications screen-reader-enabled true

# کیبورد روی صفحه
gsettings set org.gnome.desktop.a11y.applications screen-keyboard-enabled true
```

**تنظیمات سطح سیستم:**

```bash
# فونت بزرگ‌تر
gsettings set org.gnome.desktop.interface text-scaling-factor 1.5

# کنتراست بالا
gsettings set org.gnome.desktop.interface gtk-theme 'HighContrast'
gsettings set org.gnome.desktop.interface icon-theme 'HighContrast'

# Sticky Keys
gsettings set org.gnome.desktop.a11y.keyboard stickykeys-enable true
```

**BRLTTY:**

```bash
# نصب
sudo apt install brltty

# وضعیت
brltty -b
systemctl status brltty

# پیکربندی
sudo dpkg-reconfigure brltty
```

### مقایسه و الگوهای مختلف

**انواع ناتوانی و ابزار:**

| ناتوانی | ابزار | نوع |
|---|---|---|
| بینایی کامل | Orca | صفحه‌خوان |
| بینایی جزئی | Magnifier | بزرگ‌نما |
| بینایی | BRLTTY | بریل |
| حرکتی | Onboard | کیبورد صفحه |
| حرکتی | Dasher | ورودی پیش‌بین |
| شناختی | رابط ساده | — |

**سطح دسترس‌پذیری:**

| سطح | ابزار |
|---|---|
| کاربر | تنظیمات دسکتاپ |
| سیستم | gsettings, dconf |
| برنامه | APIهای AT-SPI |

**فناوری کمکی (Assistive Technology):**

| فناوری | کار |
|---|---|
| AT-SPI | API ارتباط با برنامه‌ها |
| a11y | دسترس‌پذیری |
| D-Bus | ارتباط بین اجزا |

### اشکال شایع

**اشکال ۱: نصب نشدن ابزار.**
بعضی ابزارها باید دستی نصب شوند. راه‌حل: نصب پکیج.

**اشکال ۲: نبود پشتیبانی دسکتاپ.**
همهٔ ابزارها در همهٔ دسکتاپ‌ها کار نمی‌کنند. راه‌حل: دسکتاپ سازگار.

**اشکال ۳: فراموش کردن تنظیمات بریل.**
BRLTTY نیاز به پیکربندی جداگانه دارد. راه‌حل: `dpkg-reconfigure brltty`.

**اشکال ۴: نادیده گرفتن در محیط متنی.**
بعضی ابزارها گرافیکی‌اند. راه‌حل: ابزار خط فرمان.

**اشکال ۵: Orca بدون برنامهٔ سازگار.**
بعضی برنامه‌ها با Orca کار نمی‌کنند. راه‌حل: برنامه‌های سازگار.

**اشکال ۶: تداخل میانبر.**
Orca میانبرهایی دارد که ممکن است تداخل کنند. راه‌حل: پیکربندی.

### نکات مهم (Tips & Tricks)

- **Orca با `Super+Alt+S` فعال/غیرفعال می‌شود (در GNOME).**
- **`gsettings` راه اسکریپت‌پذیر تنظیمات است.**
- **Sticky Keys برای کاربران با مشکل حرکتی مفید است.**
- **تنظیمات دسترس‌پذیری را می‌توان در سطح کاربر ذخیره کرد.**
- **برای سرور، دسترس‌پذیری گرافیکی کاربرد کمتری دارد.**
- **`dconf-editor` رابط گرافیکی تنظیمات.**
- **`orca -u` راهنمای Orca.**
- **Magnifier برای بزرگ‌نمایی موقت.**
- **High Contrast برای خوانایی بهتر.**
- **زیرنویس در برنامه‌های چندرسانه‌ای.**
- **`AT-SPI` لایهٔ ارتباط است.**
- **برای تست، دسترس‌پذیری را فعال و بررسی کنید.**

### خلاصه فصل

دسترس‌پذیری استفادهٔ سیستم برای همه را ممکن می‌کند. ابزارهای اصلی: Orca (صفحه‌خوان)، BRLTTY (بریل)، Onboard (کیبورد صفحه)، Magnifier. تنظیمات در بخش Accessibility دسکتاپ یا با `gsettings` انجام می‌شود. ابزار مناسب برای هر نوع ناتوانی متفاوت است. `gsettings` راه اسکریپت‌پذیر و `dconf-editor` راه گرافیکی است.

---

## منابع و مراجع

- man pages: `man Xorg`, `man xrandr`, `man startx`, `man orca`
- X.Org Foundation: https://www.x.org
- GNOME Accessibility: https://help.gnome.org/users/gnome-help/stable/a11y.html
- Orca Screen Reader: https://wiki.gnome.org/Projects/Orca
- Wayland: https://wayland.freedesktop.org

**بعدی:** [فایل ۱۱ — وظایف مدیریتی](11-Administrative-Tasks.md)
