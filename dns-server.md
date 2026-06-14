<div dir="rtl" align="right">
### docker compose
```
services:
  dns:
    image: technitium/dns-server:15.2.0
    container_name: dns
    hostname: dns
    restart: unless-stopped

    ports:
      - "5380:5380/tcp"
      - "53:53/udp"
      - "53:53/tcp"

    environment:
      DNS_SERVER_DOMAIN: "dns.30bime.local"
      DNS_SERVER_ADMIN_PASSWORD: "1qaz!QAZ"
      DNS_SERVER_RECURSION: "AllowOnlyForPrivateNetworks"
      DNS_SERVER_FORWARDERS: "178.22.122.101,185.51.200.1"
      DNS_SERVER_FORWARDER_PROTOCOL: "Udp"
      DNS_SERVER_LOG_USING_LOCAL_TIME: "true"
      DNS_SERVER_LOG_FOLDER_PATH: "/var/log/technitium/dns"
      DNS_SERVER_LOG_MAX_LOG_FILE_DAYS: "30"
      DNS_SERVER_STATS_MAX_STAT_FILE_DAYS: "30"
      DNS_SERVER_PREFER_IPV6: "false"
      TZ: Asia/Tehran
    volumes:
      - ./config:/etc/dns
      - ./logs:/var/log/technitium/dns
    sysctls:
      - net.ipv4.ip_local_port_range=1024 65535
    networks:
      - dns-net

networks:
  dns-net:
    external: true
```
# راهنمای مفاهیم و اصطلاحات مهم DNS برای شبکه داخلی شرکت

## سناریوی موردنظر

در این سناریو یک DNS Server داخلی برای شرکت راه‌اندازی شده است تا کلاینت‌های داخلی بتوانند دامنه‌های داخلی شرکت را Resolve کنند.

مثال:

```text
stage.30bime.ir  -> 172.24.11.128
```

اگر دامنه‌ای داخل DNS داخلی تعریف نشده بود، DNS Server درخواست را به Forwarderهای خارجی ارسال می‌کند:

```text
178.22.122.101
185.51.200.1
```

معماری کلی:

```text
Company Clients
      |
      | DNS = 172.24.11.10
      v
Technitium DNS Server
      |
      |-- Internal Zone:
      |     stage.30bime.ir -> 172.24.11.128
      |
      |-- Forwarders:
            178.22.122.101
            185.51.200.1
```

---

# 1. DNS چیست؟

DNS مخفف **Domain Name System** است.

DNS مثل دفترچه تلفن اینترنت و شبکه کار می‌کند. کار اصلی آن تبدیل نام دامنه به IP است.

مثلا کاربر در مرورگر می‌زند:

```text
http://stage.30bime.ir
```

سیستم باید بفهمد این دامنه به چه IP وصل شود:

```text
stage.30bime.ir -> 172.24.11.128
```

پس DNS این کار را انجام می‌دهد:

```text
Name -> IP Address
```

مثال‌های ساده:

```text
google.com       -> 142.250.x.x
stage.30bime.ir  -> 172.24.11.128
gitlab.local     -> 172.24.11.223
```

---

# 2. DNS Client چیست؟

هر سیستمی که از DNS Server سوال می‌پرسد، DNS Client است.

مثلا:

```text
Laptop User
Windows Client
Linux Server
Mobile Phone
Browser
Application
```

وقتی کاربر یک دامنه را باز می‌کند، سیستم عامل یا برنامه از DNS Server می‌پرسد:

```text
IP این دامنه چیست؟
```

مثلا:

```text
Client: stage.30bime.ir چیست؟
DNS Server: 172.24.11.128
```

---

# 3. Resolver چیست؟

Resolver بخشی از سیستم عامل یا برنامه است که درخواست DNS را ارسال می‌کند.

مثلا در یک سیستم ویندوزی، وقتی کاربر این آدرس را باز می‌کند:

```text
stage.30bime.ir
```

Windows DNS Resolver از DNS Server تنظیم‌شده روی کارت شبکه سوال می‌پرسد.

اگر روی سیستم کاربر DNS این باشد:

```text
172.24.11.10
```

درخواست به DNS داخلی شرکت ارسال می‌شود.

---

# 4. Recursive DNS Server چیست؟

Recursive DNS Server سروری است که از طرف کلاینت دنبال جواب می‌گردد.

در سناریوی شما، Technitium برای کلاینت‌ها نقش Recursive DNS دارد.

یعنی اگر خودش جواب را بداند، مستقیم جواب می‌دهد. اگر نداند، از DNSهای بالادستی یا Forwarderها می‌پرسد.

مثال:

```text
Client -> Technitium -> Forwarder -> Internet DNS
```

برای دامنه داخلی:

```text
Client asks: stage.30bime.ir
Technitium answers: 172.24.11.128
```

برای دامنه خارجی:

```text
Client asks: google.com
Technitium asks Forwarder
Forwarder answers
Technitium returns answer to Client
```

---

# 5. Authoritative DNS Server چیست؟

Authoritative یعنی DNS Server مرجع و مالک اصلی یک Zone است.

وقتی داخل Technitium یک Zone می‌سازی، مثلا:

```text
stage.30bime.ir
```

و داخل آن رکورد می‌گذاری:

```text
@ A 172.24.11.128
```

Technitium برای این Zone می‌شود **Authoritative DNS Server**.

یعنی اگر کسی از Technitium بپرسد:

```text
stage.30bime.ir
```

دیگر Technitium از Forwarder نمی‌پرسد؛ خودش جواب قطعی می‌دهد:

```text
172.24.11.128
```

---

# 6. تفاوت Recursive و Authoritative

DNS Server می‌تواند هم Recursive باشد، هم Authoritative.

در سناریوی شما Technitium هر دو نقش را دارد.

## برای دامنه داخلی

```text
stage.30bime.ir
```

Technitium نقش Authoritative دارد، چون خودش Zone آن را دارد.

## برای دامنه خارجی

```text
google.com
```

Technitium نقش Recursive دارد، چون خودش مالک google.com نیست و از Forwarderها سوال می‌پرسد.

---

# 7. Zone چیست؟

Zone یعنی محدوده‌ای از DNS که یک DNS Server مسئول مدیریت آن است.

مثلا وقتی در Technitium می‌سازی:

```text
Zone: stage.30bime.ir
```

یعنی Technitium مسئول جواب دادن برای این نام است:

```text
stage.30bime.ir
```

اگر داخل همین Zone رکورد `api` بسازی:

```text
api A 172.24.11.129
```

نتیجه می‌شود:

```text
api.stage.30bime.ir -> 172.24.11.129
```

---

# 8. تفاوت Zone و Record

این دو مفهوم خیلی مهم هستند.

## Zone

Zone محدوده مدیریتی است.

مثلا:

```text
stage.30bime.ir
```

## Record

Record اطلاعات داخل Zone است.

مثلا:

```text
@ A 172.24.11.128
```

نتیجه:

```text
stage.30bime.ir -> 172.24.11.128
```

مثال کامل:

```text
Zone:
  stage.30bime.ir

Record:
  Name: @
  Type: A
  Data: 172.24.11.128
  TTL: 60
```

---

# 9. علامت @ در DNS یعنی چه؟

در DNS، علامت `@` یعنی خود نام Zone.

اگر Zone این باشد:

```text
stage.30bime.ir
```

و رکورد این باشد:

```text
@ A 172.24.11.128
```

یعنی:

```text
stage.30bime.ir A 172.24.11.128
```

اگر داخل همین Zone رکورد زیر را بسازی:

```text
api A 172.24.11.129
```

نتیجه می‌شود:

```text
api.stage.30bime.ir -> 172.24.11.129
```

---

# 10. Record Typeهای مهم DNS

## 10.1. A Record

برای تبدیل دامنه به IPv4 استفاده می‌شود.

مثال:

```text
stage.30bime.ir A 172.24.11.128
```

این پرکاربردترین رکورد برای شبکه داخلی شماست.

---

## 10.2. AAAA Record

برای تبدیل دامنه به IPv6 استفاده می‌شود.

مثال:

```text
example.com AAAA 2001:db8::1
```

اگر در شبکه شرکت IPv6 نداری، فعلا خیلی درگیر این رکورد نشو.

---

## 10.3. CNAME Record

CNAME یعنی Alias یا نام مستعار.

مثال:

```text
www CNAME stage.30bime.ir
```

یعنی `www` به جای اینکه خودش IP داشته باشد، به `stage.30bime.ir` اشاره می‌کند.

نکته مهم:

یک Name نباید هم‌زمان CNAME و رکوردهای دیگر مثل A یا MX داشته باشد.

اشتباه:

```text
app A 172.24.11.128
app CNAME stage.30bime.ir
```

درست:

```text
app CNAME stage.30bime.ir
```

یا:

```text
app A 172.24.11.128
```

---

## 10.4. MX Record

برای ایمیل استفاده می‌شود.

مثال:

```text
30bime.ir MX mail.30bime.ir
```

یعنی ایمیل‌های دامنه `30bime.ir` باید به سرور `mail.30bime.ir` ارسال شوند.

برای Mail Server، Mailcow، Exchange و سرویس‌های ایمیلی، MX بسیار مهم است.

---

## 10.5. TXT Record

برای ذخیره متن و تنظیمات امنیتی استفاده می‌شود.

کاربردهای رایج:

```text
SPF
DKIM
DMARC
Domain Verification
Google Verification
Microsoft 365 Verification
```

مثال SPF:

```text
30bime.ir TXT "v=spf1 mx -all"
```

---

## 10.6. NS Record

NS مشخص می‌کند Name Server یک Zone کیست.

مثلا در Technitium ممکن است ببینی:

```text
stage.30bime.ir NS dns.30bime.local
```

یعنی DNS Server مسئول Zone برابر است با:

```text
dns.30bime.local
```

---

## 10.7. SOA Record

SOA مخفف **Start of Authority** است.

این رکورد اطلاعات مدیریتی Zone را نگه می‌دارد، مثل:

```text
Primary Name Server
Responsible Person
Serial Number
Refresh
Retry
Expire
Minimum TTL
```

تقریبا هر Zone باید یک رکورد SOA داشته باشد.

---

## 10.8. PTR Record

PTR برای Reverse DNS استفاده می‌شود.

DNS عادی:

```text
stage.30bime.ir -> 172.24.11.128
```

Reverse DNS:

```text
172.24.11.128 -> stage.30bime.ir
```

برای لاگ‌ها، مانیتورینگ، امنیت و Mail Server مهم است.

برای رنج داخلی می‌توان Zone معکوس ساخت، مثلا برای شبکه:

```text
172.24.11.0/24
```

Zone معکوس می‌شود:

```text
11.24.172.in-addr.arpa
```

---

# 11. Forwarder چیست؟

Forwarder یعنی DNS بالادستی.

وقتی Technitium جواب یک دامنه را خودش ندارد، درخواست را به Forwarder می‌فرستد.

در سناریوی شما Forwarderها این‌ها هستند:

```text
178.22.122.101
185.51.200.1
```

رفتار کلی:

```text
stage.30bime.ir -> جواب از Zone داخلی
google.com       -> ارسال به Forwarder
docker.com       -> ارسال به Forwarder
```

مسیر در Technitium معمولا:

```text
Settings -> Proxy & Forwarders -> Forwarders
```

---

# 12. اگر چند Forwarder داشته باشیم، ترتیب استفاده چگونه است؟

در Global Forwarderها، Technitium معمولا به شکل ساده و خطی مثل زیر کار نمی‌کند:

```text
اول Forwarder 1
بعد Forwarder 2
بعد Forwarder 3
```

بلکه ممکن است بر اساس latency، وضعیت پاسخ‌گویی، cache و تنظیمات داخلی، از سریع‌ترین یا مناسب‌ترین Forwarder استفاده کند.

مثال:

```text
Forwarders:
178.22.122.101
185.51.200.1
```

ممکن است گاهی جواب از اولی گرفته شود و گاهی از دومی.

اگر اولویت دقیق و Failover ترتیبی بخواهی، باید از Conditional Forwarder Zone و رکوردهای FWD با Priority استفاده کنی.

اما برای سناریوی فعلی شرکت، ساده‌ترین حالت این است:

```text
Global Forwarders:
178.22.122.101
185.51.200.1
```

---

# 13. Cache چیست؟

Cache یعنی DNS Server جواب‌ها را برای مدتی نگه می‌دارد تا هر بار مجبور نباشد دوباره سوال کند.

مثلا اگر یک کلاینت بپرسد:

```text
google.com
```

Technitium جواب را از Forwarder می‌گیرد و برای مدتی نگه می‌دارد.

دفعه بعد اگر کلاینت دیگری همان دامنه را بپرسد، Technitium سریع‌تر جواب می‌دهد.

مزایا:

```text
سرعت بیشتر
مصرف اینترنت کمتر
فشار کمتر روی Forwarder
کاهش latency
```

---

# 14. TTL چیست؟

TTL مخفف **Time To Live** است.

TTL مشخص می‌کند یک رکورد DNS چه مدت می‌تواند Cache شود.

مثلا:

```text
TTL: 60
```

یعنی جواب این رکورد برای ۶۰ ثانیه Cache شود.

برای رکوردهای داخلی در زمان تست، TTL پایین مناسب است:

```text
60 seconds
```

برای رکوردهای عملیاتی و پایدار، می‌توان مقدار بالاتری گذاشت:

```text
300 seconds
900 seconds
3600 seconds
```

پیشنهاد:

```text
TTL 60    = مناسب تست و تغییرات زیاد
TTL 300   = مناسب محیط عملیاتی با تغییرات متوسط
TTL 3600  = مناسب رکوردهای پایدار
```

---

# 15. Split DNS چیست؟

Split DNS یعنی یک دامنه داخل شبکه شرکت یک جواب بدهد و بیرون شرکت جواب دیگری.

مثلا داخل شرکت:

```text
stage.30bime.ir -> 172.24.11.128
```

ولی بیرون شرکت ممکن است:

```text
stage.30bime.ir -> Public IP
```

یا اصلا Resolve نشود.

این روش در شرکت‌ها بسیار رایج است.

کاربردها:

```text
سرویس‌های داخلی
Stage Environment
Dev Environment
GitLab داخلی
Harbor داخلی
Monitoring داخلی
Panelهای مدیریتی
```

---

# 16. چرا نباید بی‌دلیل Zone کل دامنه اصلی را بسازیم؟

اگر در Technitium این Zone را بسازی:

```text
30bime.ir
```

Technitium فکر می‌کند مسئول کل دامنه `30bime.ir` است.

بعد اگر رکوردهای لازم را داخل آن نسازی، ممکن است کلاینت‌های داخلی نتوانند رکوردهای Public واقعی دامنه را ببینند.

مثلا ممکن است این‌ها خراب شوند:

```text
www.30bime.ir
mail.30bime.ir
api.30bime.ir
MX Records
TXT Records
SPF
DKIM
DMARC
```

برای همین اگر فقط می‌خواهی این دامنه داخلی شود:

```text
stage.30bime.ir
```

بهتر است فقط Zone همین را بسازی:

```text
stage.30bime.ir
```

نه کل:

```text
30bime.ir
```

مگر اینکه بخواهی کل DNS داخلی دامنه شرکت را کامل مدیریت کنی.

---

# 17. Primary Zone چیست؟

Primary Zone یعنی این DNS Server مالک اصلی Zone است و رکوردها مستقیم روی همین سرور ساخته و ویرایش می‌شوند.

مثال:

```text
Zone: stage.30bime.ir
Type: Primary
```

یعنی Technitium خودش اطلاعات این Zone را نگه می‌دارد.

برای سناریوی شما، `stage.30bime.ir` باید Primary Zone باشد.

---

# 18. Secondary Zone چیست؟

Secondary Zone یک کپی از Zone روی DNS Server دیگر است.

مثلا اگر دو DNS داخلی داشته باشی:

```text
DNS1: 172.24.11.10
DNS2: 172.24.11.11
```

می‌توانی:

```text
DNS1 = Primary
DNS2 = Secondary
```

DNS2 رکوردها را از DNS1 دریافت می‌کند.

این کار برای High Availability و Redundancy مناسب است.

---

# 19. Conditional Forwarder چیست؟

Conditional Forwarder یعنی برای یک دامنه خاص، درخواست‌ها به DNS خاصی فرستاده شوند.

مثلا:

```text
corp.local     -> 172.24.11.20
example.local  -> 172.24.11.30
```

یا:

```text
company.internal -> DNS Server داخلی دیگر
```

کاربرد در شرکت‌های بزرگ:

```text
ارتباط بین چند شعبه
ارتباط با Active Directory
ارتباط با DNSهای داخلی دیگر
ارتباط بین چند دیتاسنتر
```

---

# 20. Wildcard DNS چیست؟

Wildcard یعنی هر زیر دامنه‌ای جواب بگیرد.

مثلا اگر داخل Zone `stage.30bime.ir` این رکورد را بسازی:

```text
* A 172.24.11.128
```

این‌ها همه به همان IP resolve می‌شوند:

```text
api.stage.30bime.ir
panel.stage.30bime.ir
test.stage.30bime.ir
anything.stage.30bime.ir
```

مزیت:

```text
برای Traefik/Nginx و سرویس‌های زیاد کاربردی است
نیاز نیست برای هر subdomain رکورد جدا بسازی
```

عیب:

```text
ممکن است خطاها را مخفی کند
هر اسم اشتباهی هم جواب می‌گیرد
```

برای شروع بهتر است فقط رکوردهای مشخص بسازی.

---

# 21. DNS روی چه پورتی کار می‌کند؟

DNS معمولا روی پورت‌های زیر کار می‌کند:

```text
UDP 53
TCP 53
```

اکثر Queryهای معمولی با UDP انجام می‌شوند.

TCP برای حالت‌هایی مثل موارد زیر استفاده می‌شود:

```text
جواب‌های بزرگ
Zone Transfer
DNSSEC
برخی Queryهای خاص
```

پس روی فایروال باید هر دو را باز کنی:

```text
53/udp
53/tcp
```

پنل Technitium معمولا روی این پورت است:

```text
5380/tcp
```

---

# 22. چرا نباید DNS خارجی را به عنوان DNS دوم روی کلاینت‌ها بگذاریم؟

اشتباه رایج:

```text
DNS 1: 172.24.11.10
DNS 2: 178.22.122.101
```

مشکل اینجاست که سیستم عامل، مخصوصا ویندوز، همیشه تضمین نمی‌کند فقط از DNS اول استفاده کند. ممکن است گاهی مستقیم از DNS دوم بپرسد.

اگر از DNS دوم خارجی بپرسد، رکورد داخلی را نمی‌شناسد:

```text
stage.30bime.ir -> 172.24.11.128
```

پس ممکن است نتیجه این شود:

```text
گاهی سایت باز می‌شود
گاهی سایت باز نمی‌شود
گاهی IP اشتباه برمی‌گردد
```

حالت درست:

```text
Client DNS:
DNS 1: 172.24.11.10
DNS 2: 172.24.11.11
```

و داخل Technitium:

```text
Forwarders:
178.22.122.101
185.51.200.1
```

اگر DNS دوم می‌خواهی، باید یک DNS داخلی دوم با همان رکوردها داشته باشی.

---

# 23. DHCP چه ربطی به DNS دارد؟

DHCP به کلاینت‌ها IP می‌دهد.

علاوه بر IP، DHCP می‌تواند DNS Server را هم به سیستم‌ها بدهد.

به جای اینکه روی تک‌تک سیستم‌ها دستی DNS تنظیم کنی، در DHCP تنظیم می‌کنی:

```text
DNS Server = 172.24.11.10
```

بعد همه سیستم‌ها به صورت خودکار از DNS داخلی شرکت استفاده می‌کنند.

مثال در MikroTik:

```mikrotik
/ip dhcp-server network set [find address=172.24.11.0/24] dns-server=172.24.11.10
```

---

# 24. ترتیب Resolve در سناریوی شما

وقتی کلاینت می‌پرسد:

```text
stage.30bime.ir
```

تقریبا این مراحل طی می‌شود:

```text
1. کلاینت Cache خودش را چک می‌کند
2. اگر جواب نداشت، از DNS تنظیم‌شده روی کارت شبکه می‌پرسد
3. Technitium Cache خودش را چک می‌کند
4. Technitium Zoneهای داخلی را چک می‌کند
5. اگر رکورد داخلی پیدا شد، جواب می‌دهد
6. اگر رکورد داخلی نبود، درخواست را به Forwarder می‌فرستد
7. جواب را به کلاینت برمی‌گرداند
```

برای دامنه داخلی:

```text
stage.30bime.ir -> جواب از Zone داخلی Technitium
```

برای دامنه خارجی:

```text
google.com -> جواب از Forwarder
```

---

# 25. DNS با HTTP فرق دارد

وقتی کاربر می‌زند:

```text
http://stage.30bime.ir
```

دو مرحله جدا اتفاق می‌افتد.

## مرحله اول: DNS

```text
stage.30bime.ir -> 172.24.11.128
```

## مرحله دوم: HTTP

سیستم به این IP وصل می‌شود:

```text
172.24.11.128:80
```

پس اگر DNS درست باشد ولی سایت باز نشود، مشکل ممکن است DNS نباشد.

مشکلات احتمالی:

```text
Nginx
Apache
Traefik
Firewall
Port 80/443
Host Rule
Application
Docker Network
Reverse Proxy
```

تست‌های مفید:

```bash
dig @172.24.11.10 stage.30bime.ir +short
curl -I http://stage.30bime.ir
curl -I -H "Host: stage.30bime.ir" http://172.24.11.128
```

---

# 26. ابزارهای مهم عیب‌یابی DNS

## 26.1. dig

بهترین ابزار برای تست DNS در Linux.

تست رکورد داخلی:

```bash
dig @172.24.11.10 stage.30bime.ir
```

خروجی خلاصه:

```bash
dig @172.24.11.10 stage.30bime.ir +short
```

تست دامنه اینترنتی:

```bash
dig @172.24.11.10 google.com +short
```

تست با TCP:

```bash
dig @172.24.11.10 google.com +tcp +short
```

Trace مسیر DNS:

```bash
dig +trace stage.30bime.ir
```

---

## 26.2. nslookup

روی Windows و Linux قابل استفاده است.

```powershell
nslookup stage.30bime.ir 172.24.11.10
nslookup google.com 172.24.11.10
```

---

## 26.3. resolvectl

روی Ubuntuهای جدید:

```bash
resolvectl status
resolvectl query stage.30bime.ir
```

پاک کردن Cache:

```bash
sudo resolvectl flush-caches
```

---

## 26.4. ipconfig در Windows

دیدن تنظیمات DNS:

```powershell
ipconfig /all
```

پاک کردن DNS Cache:

```powershell
ipconfig /flushdns
```

---

## 26.5. ss برای بررسی پورت 53

```bash
sudo ss -lntup | grep ':53'
```

اگر پورت 53 اشغال بود، ممکن است یکی از این سرویس‌ها فعال باشد:

```text
systemd-resolved
dnsmasq
bind9
named
AdGuard Home
Pi-hole
Technitium قبلی
```

---

# 27. systemd-resolved چیست؟

در Ubuntuهای جدید، سرویس `systemd-resolved` معمولا DNS سیستم را مدیریت می‌کند.

گاهی این سرویس پورت 53 را اشغال می‌کند و اجازه نمی‌دهد Docker DNS Server روی پورت 53 بالا بیاید.

خطای رایج Docker:

```text
failed to bind host port 0.0.0.0:53/tcp: address already in use
```

برای بررسی:

```bash
sudo ss -lntup | grep ':53'
```

اگر `systemd-resolved` پورت 53 را گرفته بود، می‌توان DNS Stub Listener را غیرفعال کرد:

```bash
sudo nano /etc/systemd/resolved.conf
```

تنظیم:

```ini
[Resolve]
DNS=178.22.122.101 185.51.200.1
FallbackDNS=8.8.8.8 1.1.1.1
DNSStubListener=no
```

اعمال:

```bash
sudo systemctl restart systemd-resolved
sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

---

# 28. DNS Cache در چند جا وجود دارد

وقتی یک رکورد DNS را تغییر می‌دهی، ممکن است همان لحظه روی کلاینت تغییر دیده نشود.

چون Cache در چند لایه وجود دارد:

```text
Browser Cache
Windows DNS Cache
Linux systemd-resolved Cache
Technitium Cache
Forwarder Cache
ISP DNS Cache
```

برای پاک کردن Cache ویندوز:

```powershell
ipconfig /flushdns
```

برای Ubuntu:

```bash
sudo resolvectl flush-caches
```

در Technitium هم از تب Cache می‌توانی Cache را ببینی یا پاک کنی.

---

# 29. DoH چیست و چرا ممکن است مشکل‌ساز شود؟

DoH مخفف **DNS over HTTPS** است.

بعضی مرورگرها مثل Chrome و Firefox می‌توانند DNS را از طریق HTTPS و مستقل از DNS سیستم بفرستند.

مثلا به جای اینکه از DNS شرکت بپرسند:

```text
172.24.11.10
```

مستقیم از این‌ها می‌پرسند:

```text
Cloudflare DoH
Google DoH
```

در این حالت رکوردهای داخلی شرکت Resolve نمی‌شوند.

مثلا:

```text
stage.30bime.ir -> 172.24.11.128
```

ممکن است در مرورگر Resolve نشود، چون مرورگر از DNS داخلی استفاده نکرده است.

برای شبکه شرکتی بهتر است DoH مرورگرها با Policy کنترل یا غیرفعال شود.

---

# 30. DNSSEC چیست؟

DNSSEC برای اعتبارسنجی جواب‌های DNS است.

هدف DNSSEC این است که مطمئن شویم جواب DNS دستکاری نشده است.

برای DNS داخلی ساده، فعلا ضروری نیست.

ولی برای DNS عمومی و دامنه‌های Public، DNSSEC می‌تواند اهمیت داشته باشد.

---

# 31. Reverse DNS چیست؟

DNS معمولی:

```text
stage.30bime.ir -> 172.24.11.128
```

Reverse DNS:

```text
172.24.11.128 -> stage.30bime.ir
```

Reverse DNS با رکورد PTR انجام می‌شود.

برای شبکه داخلی می‌توانی بعدا Reverse Zone بسازی:

```text
11.24.172.in-addr.arpa
```

برای Mail Server عمومی، PTR بسیار مهم است و معمولا باید توسط ISP یا دیتاسنتر تنظیم شود.

---

# 32. Recursion Policy چیست؟

Recursion Policy مشخص می‌کند چه کسانی اجازه دارند از DNS Server برای Resolve دامنه‌های خارجی استفاده کنند.

برای DNS داخلی شرکت نباید Recursion برای همه اینترنت باز باشد.

اشتباه خطرناک:

```text
Allow recursion for everyone
```

حالت درست:

```text
Allow recursion only for private networks
```

یا فقط برای شبکه شرکت:

```text
172.24.11.0/24
```

در Technitium می‌توانی این مورد را از Settings کنترل کنی.

---

# 33. Open Resolver چیست؟

Open Resolver یعنی DNS Server شما برای همه اینترنت Recursive Query انجام دهد.

این خطرناک است.

مشکلات Open Resolver:

```text
استفاده در DNS Amplification Attack
فشار زیاد روی سرور
ریسک امنیتی
مصرف پهنای باند
قرار گرفتن IP در لیست‌های امنیتی
```

پس DNS داخلی شرکت باید فقط از شبکه داخلی جواب Recursive بدهد.

---

# 34. DNS و Firewall

برای اینکه کلاینت‌ها بتوانند از DNS Server استفاده کنند، باید پورت‌های زیر از شبکه داخلی باز باشد:

```text
53/udp
53/tcp
```

برای پنل Technitium:

```text
5380/tcp
```

پیشنهاد امنیتی:

```text
پورت 53 فقط برای شبکه داخلی باز باشد
پورت 5380 فقط برای مدیران شبکه باز باشد
```

مثال UFW:

```bash
sudo ufw allow from 172.24.11.0/24 to any port 53 proto udp
sudo ufw allow from 172.24.11.0/24 to any port 53 proto tcp
sudo ufw allow from 172.24.11.0/24 to any port 5380 proto tcp
```

---

# 35. DNS و Docker

وقتی DNS Server را با Docker اجرا می‌کنی، باید پورت 53 روی Host آزاد باشد.

بررسی پورت:

```bash
sudo ss -lntup | grep ':53'
```

اگر پورت اشغال باشد، Docker خطا می‌دهد:

```text
address already in use
```

در Docker Compose بهتر است پورت‌ها را روی IP داخلی Bind کنی، نه روی همه IPها.

مثال بهتر:

```yaml
ports:
  - "172.24.11.10:53:53/udp"
  - "172.24.11.10:53:53/tcp"
  - "172.24.11.10:5380:5380/tcp"
```

به جای:

```yaml
ports:
  - "53:53/udp"
  - "53:53/tcp"
  - "5380:5380/tcp"
```

---

# 36. مثال Docker Compose برای Technitium

نمونه Compose عملیاتی:

```yaml
services:
  technitium-dns:
    image: technitium/dns-server:15.2.0
    container_name: technitium-dns
    hostname: technitium-dns
    restart: unless-stopped

    ports:
      - "${DNS_SERVER_IP}:5380:5380/tcp"
      - "${DNS_SERVER_IP}:53:53/udp"
      - "${DNS_SERVER_IP}:53:53/tcp"

    environment:
      DNS_SERVER_DOMAIN: "${DNS_SERVER_DOMAIN}"
      DNS_SERVER_ADMIN_PASSWORD_FILE: "/run/secrets/admin_password"
      DNS_SERVER_RECURSION: "AllowOnlyForPrivateNetworks"
      DNS_SERVER_FORWARDERS: "178.22.122.101,185.51.200.1"
      DNS_SERVER_FORWARDER_PROTOCOL: "Udp"
      DNS_SERVER_LOG_USING_LOCAL_TIME: "true"
      DNS_SERVER_LOG_FOLDER_PATH: "/var/log/technitium/dns"
      DNS_SERVER_LOG_MAX_LOG_FILE_DAYS: "30"
      DNS_SERVER_STATS_MAX_STAT_FILE_DAYS: "30"
      DNS_SERVER_PREFER_IPV6: "false"
      TZ: "${TZ}"

    volumes:
      - ./config:/etc/dns
      - ./logs:/var/log/technitium/dns

    secrets:
      - admin_password

secrets:
  admin_password:
    file: ./secrets/admin_password.txt
```

نمونه `.env`:

```env
DNS_SERVER_IP=172.24.11.10
DNS_SERVER_DOMAIN=dns.30bime.local
TZ=Asia/Tehran
```

---

# 37. DNS_SERVER_DOMAIN در Technitium چیست؟

این env:

```env
DNS_SERVER_DOMAIN=dns.30bime.local
```

برای تعیین نام خود DNS Server است.

یعنی Technitium خودش را با این نام می‌شناسد:

```text
dns.30bime.local
```

این مقدار ربط مستقیم به رکورد زیر ندارد:

```text
stage.30bime.ir -> 172.24.11.128
```

رکوردهای دامنه باید جداگانه داخل Zone ساخته شوند.

پیشنهاد برای شبکه داخلی:

```env
DNS_SERVER_DOMAIN=dns.30bime.local
```

---

# 38. ساخت Zone برای stage.30bime.ir در Technitium

داخل پنل Technitium:

```text
Zones -> Add Zone
```

مقادیر:

```text
Zone: stage.30bime.ir
Type: Primary Zone
```

بعد داخل Zone:

```text
Add Record
```

مقادیر رکورد:

```text
Name: @
Type: A
TTL: 60
IPv4 Address: 172.24.11.128
```

نتیجه:

```text
stage.30bime.ir -> 172.24.11.128
```

---

# 39. تست نهایی DNS

## تست رکورد داخلی

```bash
dig @172.24.11.10 stage.30bime.ir +short
```

خروجی مورد انتظار:

```text
172.24.11.128
```

## تست دامنه اینترنتی

```bash
dig @172.24.11.10 google.com +short
```

باید IP برگرداند.

## تست TCP DNS

```bash
dig @172.24.11.10 google.com +tcp +short
```

## تست از Windows

```powershell
nslookup stage.30bime.ir 172.24.11.10
nslookup google.com 172.24.11.10
```

## تست HTTP

```bash
curl -I http://stage.30bime.ir
```

یا:

```bash
curl -I -H "Host: stage.30bime.ir" http://172.24.11.128
```

---

# 40. چک‌لیست عملیاتی DNS شرکت

برای محیط شرکت، این موارد مهم هستند:

```text
1. DNS Server داخلی IP ثابت داشته باشد
2. کلاینت‌ها فقط DNS داخلی بگیرند
3. DNS خارجی فقط داخل Technitium به عنوان Forwarder باشد
4. روی کلاینت‌ها DNS دوم خارجی تنظیم نشود
5. برای رکوردهای داخلی Zone جدا بساز
6. بی‌دلیل کل Zone دامنه اصلی را داخلی نکن
7. TTL در زمان تست پایین باشد
8. بعد از تغییر رکورد، Cache را در نظر بگیر
9. پورت‌های 53/udp و 53/tcp باز باشند
10. پنل Technitium فقط از شبکه داخلی قابل دسترسی باشد
11. Recursion فقط برای شبکه داخلی فعال باشد
12. از config و volumeهای Technitium بکاپ بگیر
13. اگر DNS دوم می‌خواهی، DNS داخلی دوم راه‌اندازی کن
14. DoH مرورگرها را در شبکه شرکتی کنترل کن
15. لاگ‌ها و Queryها را در Technitium بررسی کن
```

---

# 41. خلاصه اصطلاحات مهم

| اصطلاح                | معنی                                     |
| --------------------- | ---------------------------------------- |
| DNS                   | سیستم تبدیل نام دامنه به IP              |
| Resolver              | بخشی که درخواست DNS ارسال می‌کند         |
| Recursive DNS         | DNSی که از طرف کلاینت دنبال جواب می‌گردد |
| Authoritative DNS     | DNS مرجع و مالک یک Zone                  |
| Zone                  | محدوده‌ای از DNS که مدیریت می‌شود        |
| Record                | اطلاعات داخل Zone                        |
| A Record              | تبدیل دامنه به IPv4                      |
| AAAA Record           | تبدیل دامنه به IPv6                      |
| CNAME                 | نام مستعار                               |
| MX                    | رکورد ایمیل                              |
| TXT                   | رکورد متنی برای SPF/DKIM/Verification    |
| NS                    | مشخص‌کننده Name Server                   |
| SOA                   | رکورد مدیریتی Zone                       |
| PTR                   | رکورد Reverse DNS                        |
| Forwarder             | DNS بالادستی                             |
| Cache                 | ذخیره موقت جواب DNS                      |
| TTL                   | مدت اعتبار Cache                         |
| Split DNS             | جواب متفاوت داخل و خارج شرکت             |
| Conditional Forwarder | Forward کردن دامنه خاص به DNS خاص        |
| Wildcard              | جواب دادن به همه Subdomainها             |
| DoH                   | DNS over HTTPS                           |
| DNSSEC                | اعتبارسنجی امنیتی DNS                    |
| Open Resolver         | DNS باز برای همه اینترنت، ناامن          |

---

# 42. معماری پیشنهادی نهایی

```text
Clients
  |
  | DNS Server = 172.24.11.10
  v
Technitium DNS Server
  |
  |-- Internal Authoritative Zone:
  |      stage.30bime.ir
  |      @ A 172.24.11.128
  |
  |-- Recursive Resolver:
  |      Forwarders:
  |        178.22.122.101
  |        185.51.200.1
```

نتیجه:

```text
دامنه داخلی؟       جواب از Technitium
دامنه غیر داخلی؟   ارسال به Forwarderها
```

---

# 43. دستورهای کاربردی سریع

## بررسی پورت DNS

```bash
sudo ss -lntup | grep ':53'
```

## تست رکورد داخلی

```bash
dig @172.24.11.10 stage.30bime.ir +short
```

## تست Forwarder

```bash
dig @172.24.11.10 google.com +short
```

## پاک کردن Cache در Windows

```powershell
ipconfig /flushdns
```

## پاک کردن Cache در Ubuntu

```bash
sudo resolvectl flush-caches
```

## دیدن DNS Client در Windows

```powershell
ipconfig /all
```

## دیدن DNS در Ubuntu

```bash
resolvectl status
```

## تست سایت با دامنه

```bash
curl -I http://stage.30bime.ir
```

## تست سایت با IP و Host Header

```bash
curl -I -H "Host: stage.30bime.ir" http://172.24.11.128
```

---

# 44. نتیجه‌گیری

برای شبکه شرکت، لازم نیست همه جزئیات سطح ISP یا Public DNS را بلد باشی. اما این مفاهیم برای مدیریت DNS داخلی ضروری هستند:

```text
A Record
Zone
Forwarder
Recursive DNS
Authoritative DNS
TTL
Cache
NS
SOA
CNAME
DHCP DNS
Split DNS
Firewall
DoH
```

با همین مفاهیم می‌توانی اکثر نیازهای DNS داخلی شرکت را مدیریت کنی، از جمله:

```text
Resolve کردن دامنه‌های داخلی
Forward کردن دامنه‌های خارجی
عیب‌یابی DNS
مدیریت رکوردها
کنترل دسترسی کلاینت‌ها
تنظیم DNS از طریق DHCP
راه‌اندازی DNS دوم برای Redundancy
```
</div>
