# راهنمای عملیاتی راه‌اندازی NTP Server روی Ubuntu با Chrony

این راهنما مراحل نصب، پیکربندی، امن‌سازی، عیب‌یابی و نگهداری یک NTP Server داخلی روی Ubuntu را توضیح می‌دهد. در انتها نیز روش تنظیم کلاینت‌های Windows، Linux و به‌صورت اختیاری ESXi آمده است.

## سناریوی این راهنما

| مورد | مقدار نمونه |
|---|---|
| سیستم‌عامل NTP Server | Ubuntu Server |
| نرم‌افزار NTP | Chrony (`chronyd`) |
| IP سرور NTP | `172.24.11.23` |
| شبکه مجاز | `172.24.0.0/16` |
| Time zone محلی | `Asia/Tehran` |
| پورت سرویس | `UDP/123` |
| Upstream اصلی | `ntp.day.ir` |
| Upstream جایگزین | NTP Poolهای Ubuntu |

> مقادیر IP و شبکه را متناسب با زیرساخت خود تغییر دهید. NTP Server باید IP ثابت یا DHCP Reservation داشته باشد.

---

## ۱. مفاهیم مهمی که مدیر سیستم باید بداند

### NTP چه کاری انجام می‌دهد؟

NTP ساعت سیستم‌ها را بر اساس UTC هماهنگ می‌کند. Time zone جزئی از NTP نیست و روی هر سیستم‌عامل جداگانه تنظیم می‌شود. بنابراین ممکن است `chronyc tracking` زمان را به UTC نمایش دهد، اما دستور `date` ساعت تهران را نشان دهد؛ این رفتار کاملاً طبیعی است.

### Chrony چیست؟

Chrony شامل دو بخش اصلی است:

- `chronyd`: سرویس اصلی که ساعت سیستم را Sync می‌کند و می‌تواند به کلاینت‌ها NTP ارائه دهد.
- `chronyc`: ابزار مدیریتی و مشاهده وضعیت `chronyd`.

Chrony گزینه پیشنهادی Ubuntu برای ارائه NTP است و از Ubuntu 25.10 به بعد به‌صورت پیش‌فرض برای Time Synchronization استفاده می‌شود. در نسخه‌های قدیمی‌تر می‌توان آن را با APT نصب کرد.

### Stratum چیست؟

Stratum فاصله منطقی یک NTP Server از منبع مرجع زمان را نشان می‌دهد:

| Stratum | مفهوم |
|---|---|
| 0 | ساعت اتمی، GPS یا مرجع مستقیم؛ معمولاً کلاینت عادی نیست |
| 1 | مستقیماً متصل به Stratum 0 |
| 2 و 3 | NTP Serverهای بالادستی رایج |
| 4 | سرور داخلی که از Stratum 3 زمان گرفته است |
| 5 | کلاینتی که از سرور داخلی Stratum 4 زمان می‌گیرد |

عدد کمتر الزاماً همیشه به معنی کیفیت بهتر نیست؛ پایداری، تأخیر، تعداد منابع مستقل و Reachability نیز مهم‌اند.

### علامت‌های `chronyc sources`

| علامت | معنی |
|---|---|
| `^*` | منبع فعلی و منتخب Chrony |
| `^+` | منبع سالم که با منبع اصلی ترکیب می‌شود |
| `^-` | منبع سالم ولی فعلاً استفاده نمی‌شود |
| `^?` | منبع فعلاً قابل استفاده نیست یا Sample کافی ندارد |
| `^x` | منبع احتمالاً زمان اشتباه ارائه می‌دهد |
| `^~` | منبع ناپایدار یا دارای تغییرات زیاد است |

### Reach چیست؟

ستون `Reach` یک مقدار هشت‌بیتی در مبنای هشت است. پس از Restart معمولاً به‌تدریج رشد می‌کند:

```text
0 → 1 → 3 → 7 → 17 → 37 → 77 → 177 → 377
```

مقدار `377` یعنی هشت درخواست اخیر پاسخ موفق گرفته‌اند. مقدار `0` یعنی هیچ پاسخ معتبری دریافت نشده است.

### Slew و Step

- **Slew:** Chrony ساعت را به‌تدریج اصلاح می‌کند تا Logها و برنامه‌ها با پرش زمانی مواجه نشوند.
- **Step:** در اختلاف‌های بزرگ، ساعت یک‌باره جلو یا عقب می‌رود.

دستور `makestep 1 3` اجازه می‌دهد در سه Update اول، اختلاف بیشتر از یک ثانیه یک‌باره اصلاح شود. اجرای دستی `chronyc makestep` در محیط عملیاتی ممکن است باعث پرش ساعت شود؛ قبل از آن اثر روی Database، Kerberos، لاگ‌ها و Jobهای زمان‌بندی‌شده را در نظر بگیرید.

---

## ۲. پیش‌نیازها

### بررسی شبکه و IP

```bash
ip -br address
ip route
hostname -I
```

NTP Server باید IP ثابت داشته باشد. اگر Ubuntu از DHCP استفاده می‌کند، در DHCP Server برای MAC آن یک Reservation ایجاد کنید یا IP را با Netplan استاتیک کنید.

### بررسی DNS

```bash
resolvectl status
getent ahostsv4 ntp.day.ir
getent ahostsv4 ntp.ubuntu.com
```

### قوانین Firewall

برای ارتباط NTP، مسیر زیر لازم است:

```text
Client → NTP Server: UDP/123
NTP Server → Upstream NTP: UDP/123
```

TCP/123 برای NTP معمولی استفاده نمی‌شود.

---

## ۳. نصب Chrony

```bash
sudo apt update
sudo apt install chrony -y
sudo systemctl enable --now chrony.service
```

بررسی سرویس:

```bash
systemctl status chrony.service --no-pager -l
systemctl is-enabled chrony.service
systemctl is-active chrony.service
```

در صورت نصب Chrony، نباید یک Time Daemon دیگر مانند `systemd-timesyncd` یا `ntpd` هم‌زمان ساعت را کنترل کند. وضعیت را بررسی کنید:

```bash
systemctl status systemd-timesyncd.service --no-pager
dpkg -l | grep -E 'chrony|ntpsec|^ii[[:space:]]+ntp[[:space:]]'
```

نصب Chrony روی Ubuntu معمولاً تعارض با `systemd-timesyncd` را مدیریت می‌کند، اما پس از Upgradeهای قدیمی بهتر است وضعیت دستی بررسی شود.

---

## ۴. پشتیبان‌گیری از تنظیمات فعلی

قبل از هر تغییر:

```bash
sudo cp -a /etc/chrony/chrony.conf \
  "/etc/chrony/chrony.conf.$(date +%F-%H%M%S).bak"
```

فایل‌های مرتبط را مشاهده کنید:

```bash
ls -la /etc/chrony
ls -la /etc/chrony/conf.d
ls -la /etc/chrony/sources.d
```

در نسخه‌های جدید Ubuntu ممکن است Upstreamها در `/etc/chrony/sources.d/ubuntu-ntp-pools.sources` تعریف شده باشند. از تعریف بی‌دلیل منابع تکراری خودداری کنید.

---

## ۵. پیکربندی NTP Server

فایل اصلی را باز کنید:

```bash
sudo nano /etc/chrony/chrony.conf
```

نمونه پیکربندی مناسب:

```conf
# فایل‌های تنظیمات تکمیلی
confdir /etc/chrony/conf.d

# Upstream اصلی
server ntp.day.ir iburst prefer

# منابع دریافت‌شده از DHCP یا فایل‌های sources.d
sourcedir /run/chrony-dhcp
sourcedir /etc/chrony/sources.d

# فایل کلیدها و Drift
keyfile /etc/chrony/chrony.keys
driftfile /var/lib/chrony/chrony.drift
ntsdumpdir /var/lib/chrony

# هماهنگ‌سازی RTC و اصلاح اختلاف بزرگ در شروع سرویس
rtcsync
makestep 1 3

# جلوگیری از پذیرش Estimateهای بسیار نامطمئن
maxupdateskew 100.0

# اطلاعات Leap Second
leapsectz right/UTC

# فقط شبکه‌های داخلی مجاز
allow 172.24.0.0/16

# Logging
log tracking measurements statistics
logdir /var/log/chrony
```

### نکات مهم Syntax

این خط اشتباه است و باعث Fail شدن سرویس می‌شود:

```text
ntp.day.ir
```

فرم صحیح:

```text
server ntp.day.ir iburst prefer
```

یا برای یک Pool:

```text
pool ntp.ubuntu.com iburst maxsources 4
```

هر Directive مانند `logdir` را فقط یک مرتبه تعریف کنید تا فایل قابل نگهداری و عیب‌یابی باشد.

### انتخاب محدوده `allow`

اگر فقط VLAN فعلی مجاز است، بهتر است به‌جای `/16` از محدوده محدودتر استفاده شود:

```conf
allow 172.24.11.0/24
```

از موارد زیر در شبکه عملیاتی متصل به اینترنت استفاده نکنید:

```conf
allow 0.0.0.0/0
allow all
```

NTP Server باز می‌تواند در حملات Amplification مورد سوءاستفاده قرار گیرد.

---

## ۶. بررسی Syntax و Restart

قبل از Restart، پیکربندی ادغام‌شده را بررسی کنید:

```bash
sudo chronyd -p
echo $?
```

Exit Code صفر یعنی Syntax معتبر است. بررسی کوتاه‌تر:

```bash
sudo chronyd -p >/dev/null && echo "Chrony configuration: OK"
```

سپس:

```bash
sudo systemctl restart chrony.service
systemctl status chrony.service --no-pager -l
```

اگر سرویس Fail شد:

```bash
journalctl -u chrony.service -n 100 --no-pager
journalctl -xeu chrony.service --no-pager
```

---

## ۷. تنظیم Time zone تهران

```bash
sudo timedatectl set-timezone Asia/Tehran
timedatectl status
date
```

خروجی مورد انتظار:

```text
Time zone: Asia/Tehran (+0330)
```

`chronyc tracking` همچنان `Ref time (UTC)` نشان می‌دهد؛ این طبیعی است. NTP همیشه زمان UTC را منتقل می‌کند و Time zone فقط نحوه نمایش محلی ساعت است.

---

## ۸. بازکردن Firewall

اگر UFW فعال است:

```bash
sudo ufw allow from 172.24.0.0/16 to any port 123 proto udp
sudo ufw status numbered
```

اگر فقط یک VLAN مجاز است:

```bash
sudo ufw allow from 172.24.11.0/24 to any port 123 proto udp
```

در Firewall مرکزی نیز `UDP/123` بین کلاینت‌ها و `172.24.11.23` و همچنین بین NTP Server و Upstreamها مجاز باشد.

بررسی Listen شدن سرویس:

```bash
sudo ss -lunp | grep ':123'
```

خروجی مطلوب:

```text
0.0.0.0:123 ... chronyd
```

پورت `127.0.0.1:323/UDP` مربوط به رابط مدیریتی محلی `chronyc` است و طبیعی است.

---

## ۹. بررسی سلامت NTP Server

برای Online کردن Sourceها و جمع‌آوری سریع Sample:

```bash
sudo chronyc online
sudo chronyc burst 4/4
```

بعد از مدتی کوتاه:

```bash
chronyc sources -v
chronyc tracking
chronyc activity
chronyc sourcestats -v
```

وضعیت سالم:

```text
^* upstream.example
Leap status     : Normal
Stratum         : 2 تا 5
```

برای انتظار خودکار تا Sync شدن:

```bash
chronyc waitsync 30 0.5
```

این نمونه حداکثر حدود پنج دقیقه منتظر می‌ماند تا Correction از نیم ثانیه کمتر شود.

در صورت نیاز و پس از ارزیابی اثر پرش زمانی:

```bash
sudo chronyc makestep
```

مشاهده کلاینت‌هایی که از سرور درخواست زمان داشته‌اند:

```bash
chronyc clients
```

---

## ۱۰. عیب‌یابی Upstreamها

### همه Sourceها `^?` هستند

بلافاصله بعد از Restart می‌تواند طبیعی باشد. یک تا چند Poll صبر کنید و Reach را بررسی کنید. اگر `Reach` از صفر افزایش پیدا می‌کند، پاسخ دریافت می‌شود و Chrony در حال جمع‌آوری Sample است.

اگر پس از چند دقیقه همچنان `Reach=0` بود:

```bash
getent ahostsv4 ntp.day.ir
chronyc activity
journalctl -u chrony.service -n 100 --no-pager
```

ترافیک UDP/123 را مشاهده کنید:

```bash
sudo tcpdump -ni any udp port 123
```

در ترمینال دیگر:

```bash
sudo chronyc burst 4/4
```

- فقط Packet خروجی: پاسخ توسط Firewall، Router یا ISP مسدود شده است.
- Packet رفت‌وبرگشت وجود دارد ولی Source نامعتبر است: کیفیت Source، Offset و خروجی `chronyc ntpdata` را بررسی کنید.

### منبع جایگزین

اگر Upstream اصلی در دسترس نبود، می‌توان از Pool رسمی Ubuntu استفاده کرد:

```conf
pool ntp.ubuntu.com iburst maxsources 4
```

داشتن چند منبع مستقل برای تشخیص یک منبع اشتباه بهتر از اتکا به یک Upstream است.

### سرویس Start نمی‌شود

معمولاً یکی از این موارد علت است:

- نوشتن نام Host بدون `server` یا `pool`
- Directive اشتباه یا آرگومان ناقص
- فایل نامعتبر داخل `conf.d` یا `sources.d`
- دسترسی اشتباه فایل‌ها
- اشغال پورت 123 توسط سرویس NTP دیگر

بررسی‌ها:

```bash
sudo chronyd -p
sudo ss -lunp | grep ':123'
sudo systemctl status chrony.service --no-pager -l
sudo journalctl -xeu chrony.service --no-pager
```

---

## ۱۱. حالت شبکه بدون اینترنت

اگر Upstream در دسترس نیست ولی لازم است همه سیستم‌های داخلی حداقل ساعت یکسانی داشته باشند، می‌توان اضافه کرد:

```conf
local stratum 10
```

سپس:

```bash
sudo chronyd -p >/dev/null && sudo systemctl restart chrony.service
```

> هشدار: این گزینه سرور را به‌عنوان یک منبع محلی معرفی می‌کند، اما ساعت آن الزاماً با زمان واقعی جهان دقیق نیست. برای محیط‌های دارای Kerberos، Certificate، Database و Audit بهتر است Upstream معتبر یا منبع GPS وجود داشته باشد.

---

## ۱۲. تنظیم کلاینت Windows مستقل یا Workgroup

### آزمایش دسترسی قبل از تغییر

PowerShell را با `Run as Administrator` باز کنید:

```powershell
w32tm /stripchart /computer:172.24.11.23 /dataonly /samples:5
```

اگر Sample دریافت می‌شود، ارتباط `UDP/123` برقرار است. عددی مثل `+1129s` یعنی ساعت‌ها اختلاف زیادی دارند؛ پس از Sync باید به چند میلی‌ثانیه کاهش یابد.

### ثبت NTP Server

```powershell
Set-Service -Name W32Time -StartupType Automatic
Start-Service W32Time

w32tm /config /manualpeerlist:"172.24.11.23,0x8" /syncfromflags:manual /reliable:no /update

Restart-Service W32Time
w32tm /resync /rediscover
```

`0x8` باعث می‌شود Windows در NTP Client Mode با Chrony ارتباط برقرار کند.

### بررسی نتیجه

```powershell
w32tm /query /source
w32tm /query /peers
w32tm /query /status
w32tm /query /configuration | findstr /i "NtpServer Type"
Get-Date
```

نتیجه مطلوب:

```text
Source: 172.24.11.23
Leap Indicator: 0 (no warning)
Type: NTP
NtpServer: 172.24.11.23,0x8
```

اگر NTP Server داخلی Stratum 4 باشد، Windows معمولاً Stratum 5 می‌شود.

### نکته درباره Windows Settings

صفحه `Settings > Time & language > Date & time` ممکن است همچنان `time.windows.com` یا اطلاعات Cache‌شده نشان دهد. معیار اصلی تشخیص Source واقعی این دستور است:

```powershell
w32tm /query /source
```

### خطاهای رایج Windows

اگر Source همچنان این بود:

```text
Free-running System Clock
```

یا خطای زیر دریافت شد:

```text
The computer did not resync because no time data was available
```

موارد زیر را بررسی کنید:

```powershell
Get-Service W32Time
w32tm /query /configuration
w32tm /stripchart /computer:172.24.11.23 /dataonly /samples:5
Restart-Service W32Time
w32tm /resync /force
```

همچنین Event Viewer را بررسی کنید:

```text
Event Viewer
  └─ Applications and Services Logs
      └─ Microsoft
          └─ Windows
              └─ Time-Service
                  └─ Operational
```

---

## ۱۳. تنظیم Windows در محیط Active Directory

در Domain، کلاینت‌ها معمولاً نباید مستقیماً به NTP عمومی یا Ubuntu متصل شوند. ساختار استاندارد:

```text
External/Internal NTP
        ↓
PDC Emulator
        ↓
سایر Domain Controllerها
        ↓
Domain Memberها
```

NTP داخلی را روی Domain Controller دارای نقش PDC Emulator تنظیم کنید. ابتدا PDC را پیدا کنید:

```powershell
netdom query fsmo
```

روی PDC Emulator:

```powershell
w32tm /config /manualpeerlist:"172.24.11.23,0x8" /syncfromflags:manual /reliable:yes /update
Restart-Service W32Time
w32tm /resync /rediscover
```

روی Domain Memberها:

```powershell
w32tm /config /syncfromflags:domhier /update
Restart-Service W32Time
w32tm /resync /rediscover
w32tm /query /source
```

Group Policy می‌تواند تنظیم دستی NTP را Override کند. در صورت برگشت تنظیمات، مسیر زیر را بررسی کنید:

```text
Computer Configuration
  └─ Administrative Templates
      └─ System
          └─ Windows Time Service
              └─ Time Providers
```

---

## ۱۴. تنظیم کلاینت Linux با Chrony

### نصب

```bash
sudo apt update
sudo apt install chrony -y
```

### پیکربندی

از فایل اصلی نسخه پشتیبان بگیرید:

```bash
sudo cp -a /etc/chrony/chrony.conf \
  "/etc/chrony/chrony.conf.$(date +%F-%H%M%S).bak"
sudo nano /etc/chrony/chrony.conf
```

Upstreamهای عمومی را در صورت نیاز Comment و NTP داخلی را اضافه کنید:

```conf
server 172.24.11.23 iburst prefer
```

اگر دو NTP Server داخلی دارید:

```conf
server 172.24.11.23 iburst prefer
server 172.24.11.24 iburst
```

سپس:

```bash
sudo chronyd -p >/dev/null && echo "Chrony configuration: OK"
sudo systemctl enable --now chrony.service
sudo systemctl restart chrony.service
```

### بررسی

```bash
chronyc sources -v
chronyc tracking
timedatectl status
```

نتیجه مطلوب:

```text
^* 172.24.11.23
Leap status : Normal
```

برای دریافت سریع Sample:

```bash
sudo chronyc online
sudo chronyc burst 4/4
```

برای Step فوری فقط در صورت مجازبودن پرش زمانی:

```bash
sudo chronyc makestep
```

Time zone کلاینت را جداگانه تنظیم کنید:

```bash
sudo timedatectl set-timezone Asia/Tehran
```

---

## ۱۵. کلاینت Linux با systemd-timesyncd

برای کلاینت‌های ساده می‌توان به‌جای Chrony از `systemd-timesyncd` استفاده کرد، اما یک سیستم نباید هم‌زمان Chrony و timesyncd فعال داشته باشد.

فایل زیر را ویرایش کنید:

```bash
sudo nano /etc/systemd/timesyncd.conf
```

```ini
[Time]
NTP=172.24.11.23
FallbackNTP=ntp.ubuntu.com
```

سپس:

```bash
sudo systemctl enable --now systemd-timesyncd.service
sudo systemctl restart systemd-timesyncd.service
timedatectl timesync-status
timedatectl status
```

برای سرورهای حساس، Chrony قابلیت‌های تشخیصی و کنترلی بیشتری ارائه می‌دهد و معمولاً انتخاب مناسب‌تری است.

---

## ۱۶. تنظیم ESXi به‌عنوان کلاینت NTP

در رابط ESXi:

```text
Host
  └─ Manage یا Configure
      └─ System
          └─ Time & date / Time Configuration
```

تنظیم کنید:

- NTP Server: `172.24.11.23`
- سرویس NTP: `Start`
- Startup Policy: `Start and stop with host`

اگر VMware Tools روی VMها گزینه `Synchronize guest time with host` دارد، از وجود دو منبع زمان متعارض جلوگیری کنید. در محیط Domain معمولاً Windows باید از Domain Hierarchy زمان بگیرد و ESXi نیز خودش NTP صحیح داشته باشد.

---

## ۱۷. نگهداری و مانیتورینگ عملیاتی

حداقل موارد زیر را مانیتور کنید:

| شاخص | وضعیت مطلوب |
|---|---|
| وضعیت سرویس | `chrony.service = active` |
| Source منتخب | حداقل یک `^*` |
| Leap status | `Normal` |
| Reach | در حالت پایدار نزدیک `377` |
| Offset | متناسب با SLA؛ معمولاً چند ms تا ده‌ها ms |
| Stratum | عدد معتبر و غیرصفر |
| UDP/123 | در حال Listen و از شبکه داخلی قابل دسترس |
| فضای Log | `/var/log/chrony` پر نشود |

دستورات مناسب Health Check:

```bash
systemctl is-active chrony.service
chronyc tracking
chronyc sources -v
chronyc activity
chronyc clients
ss -lunp | grep ':123'
```

برای Zabbix یا سیستم مانیتورینگ، روی موارد زیر Alert بسازید:

- سرویس Chrony متوقف شده است.
- `Leap status` برای چند Poll متوالی `Not synchronised` است.
- هیچ Source با `^*` وجود ندارد.
- Offset از آستانه سازمان بیشتر است.
- همه Sourceها `Reach=0` دارند.
- NTP Server داخلی از کلاینت تست پاسخ نمی‌دهد.

### افزونگی

برای محیط عملیاتی بهتر است حداقل دو NTP Server داخلی با IP ثابت وجود داشته باشد و کلاینت‌ها هر دو را بشناسند. خود NTP Serverها نیز بهتر است چند Upstream مستقل داشته باشند.

### امنیت

- `allow` را فقط به Subnetهای لازم محدود کنید.
- UDP/123 را از اینترنت به سرور داخلی باز نکنید.
- سرور را Patch نگه دارید.
- دسترسی SSH و `sudo` را محدود کنید.
- تغییرات فایل پیکربندی را Version Control یا Backup کنید.
- Logها و تغییرات غیرمنتظره Stratum/Source را مانیتور کنید.

### مجازی‌سازی

اگر NTP Server یک VM است:

- ESXi باید NTP صحیح داشته باشد.
- IP سرور NTP ثابت باشد.
- از Snapshot قدیمی به‌عنوان روش Backup دائمی استفاده نشود.
- پس از Restore یا Resume، وضعیت `chronyc tracking` بررسی شود.
- از رقابت VMware Tools Time Sync با Chrony جلوگیری شود.

---

## ۱۸. Runbook سریع مدیر سیستم

### بررسی روزانه

```bash
systemctl is-active chrony.service
chronyc sources -v
chronyc tracking
```

### اگر Source انتخاب نشده بود

```bash
sudo chronyc online
sudo chronyc burst 4/4
chronyc sources -v
journalctl -u chrony.service -n 100 --no-pager
```

### اگر سرویس Fail بود

```bash
sudo chronyd -p
sudo ss -lunp | grep ':123'
sudo journalctl -xeu chrony.service --no-pager
```

### بررسی از Windows

```powershell
w32tm /stripchart /computer:172.24.11.23 /dataonly /samples:5
w32tm /query /source
w32tm /query /status
```

### بررسی از Linux

```bash
chronyc sources -v
chronyc tracking
```

---

## ۱۹. چک‌لیست نهایی

- [ ] NTP Server دارای IP ثابت یا DHCP Reservation است.
- [ ] Time zone روی `Asia/Tehran` تنظیم شده است.
- [ ] `chrony.service` فعال و Enabled است.
- [ ] حداقل یک Upstream با `^*` انتخاب شده است.
- [ ] `Leap status` برابر `Normal` است.
- [ ] `UDP/123` روی سرور Listen می‌شود.
- [ ] `allow` فقط شبکه‌های لازم را پوشش می‌دهد.
- [ ] Firewall ورودی و خروجی NTP بررسی شده است.
- [ ] کلاینت Windows در `w32tm /query /source` منبع صحیح دارد.
- [ ] کلاینت Linux در `chronyc sources` منبع داخلی را با `^*` نشان می‌دهد.
- [ ] ESXi و Hypervisor نیز ساعت صحیح دارند.
- [ ] مانیتورینگ و Alert برای Service، Offset، Leap و Source ایجاد شده است.
- [ ] حداقل یک تست پس از Reboot سرور انجام شده است.
- [ ] برای محیط حساس، NTP Server دوم در نظر گرفته شده است.

---

## منابع رسمی

- [Ubuntu Server: ارائه NTP با Chrony](https://ubuntu.com/server/docs/how-to/networking/serve-ntp-with-chrony/)
- [Ubuntu Server: همگام‌سازی زمان با Chrony](https://ubuntu.com/server/docs/how-to/networking/chrony-client/)
- [Ubuntu Server: آشنایی با Time Synchronization](https://ubuntu.com/server/docs/explanation/networking/about-time-synchronisation/)
- [Ubuntu Server: استفاده از systemd-timesyncd](https://ubuntu.com/server/docs/how-to/networking/timedatectl-and-timesyncd/)
- [Chrony Documentation](https://chrony-project.org/documentation.html)
- [Microsoft Windows Time Service Tools](https://learn.microsoft.com/windows-server/networking/windows-time-service/windows-time-service-tools-and-settings)

