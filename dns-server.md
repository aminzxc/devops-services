### github
```
https://github.com/TechnitiumSoftware/DnsServer/tree/master
```
### docker compose
```yaml
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
# default user `admin`



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


<div dir="rtl" align="right">

<h3><bdi>DNS</bdi> چیست؟</h3>

<p><bdi>DNS</bdi> مخفف <bdi>Domain Name System</bdi> است.</p>

<p><bdi>DNS</bdi> مثل دفترچه تلفن اینترنت و شبکه کار می‌کند. کار اصلی آن تبدیل نام دامنه به <bdi>IP</bdi> است.</p>

<p>مثلاً کاربر در مرورگر می‌زند:</p>

</div>

مثلا کاربر در مرورگر می‌زند:

<div dir="ltr" align="left">

```text
http://stage.30bime.ir
```

</div>

سیستم باید بفهمد این دامنه به چه IP وصل شود:

<div dir="ltr" align="left">

```text
stage.30bime.ir -> 172.24.11.128
```

</div>

پس DNS این کار را انجام می‌دهد:

<div dir="ltr" align="left">

```text
Name -> IP Address
```

</div>

مثال‌های ساده:

<div dir="ltr" align="left">

```text
google.com       -> 142.250.x.x
stage.30bime.ir  -> 172.24.11.128
gitlab.local     -> 172.24.11.223
```

</div>

---

# ۲. DNS Client چیست؟

هر سیستمی که از DNS Server سوال می‌پرسد، DNS Client است.

مثلا:

<div dir="ltr" align="left">

```text
Laptop User
Windows Client
Linux Server
Mobile Phone
Browser
Application
```

</div>

وقتی کاربر یک دامنه را باز می‌کند، سیستم عامل یا برنامه از DNS Server می‌پرسد:

<div dir="rtl" align="right">

```text
IP این دامنه چیست؟
```

</div>

مثلا:

<div dir="ltr" align="left">

```text
Client: stage.30bime.ir چیست؟
DNS Server: 172.24.11.128
```

</div>

---

# ۳. Resolver چیست؟

Resolver بخشی از سیستم عامل یا برنامه است که درخواست DNS را ارسال می‌کند.

مثلا در یک سیستم ویندوزی، وقتی کاربر این آدرس را باز می‌کند:

<div dir="ltr" align="left">

```text
stage.30bime.ir
```

</div>

Windows DNS Resolver از DNS Server تنظیم‌شده روی کارت شبکه سوال می‌پرسد.

اگر روی سیستم کاربر DNS این باشد:

<div dir="ltr" align="left">

```text
172.24.11.10
```

</div>

درخواست به DNS داخلی شرکت ارسال می‌شود.

---

# ۴. Recursive DNS Server چیست؟

Recursive DNS Server سروری است که از طرف کلاینت دنبال جواب می‌گردد.

در سناریوی شما، Technitium برای کلاینت‌ها نقش Recursive DNS دارد.

یعنی اگر خودش جواب را بداند، مستقیم جواب می‌دهد. اگر نداند، از DNSهای بالادستی یا Forwarderها می‌پرسد.

مثال:

<div dir="ltr" align="left">

```text
Client -> Technitium -> Forwarder -> Internet DNS
```

</div>

برای دامنه داخلی:

<div dir="ltr" align="left">

```text
Client asks: stage.30bime.ir
Technitium answers: 172.24.11.128
```

</div>

برای دامنه خارجی:

<div dir="ltr" align="left">

```text
Client asks: google.com
Technitium asks Forwarder
Forwarder answers
Technitium returns answer to Client
```

</div>

---

# ۵. Authoritative DNS Server چیست؟

Authoritative یعنی DNS Server مرجع و مالک اصلی یک Zone است.

وقتی داخل Technitium یک Zone می‌سازی، مثلا:

<div dir="ltr" align="left">

```text
stage.30bime.ir
```

</div>

و داخل آن رکورد می‌گذاری:

<div dir="ltr" align="left">

```text
@ A 172.24.11.128
```

</div>

Technitium برای این Zone می‌شود **Authoritative DNS Server**.

یعنی اگر کسی از Technitium بپرسد:

<div dir="ltr" align="left">

```text
stage.30bime.ir
```

</div>

دیگر Technitium از Forwarder نمی‌پرسد؛ خودش جواب قطعی می‌دهد:

<div dir="ltr" align="left">

```text
172.24.11.128
```

</div>

---

# ۶. تفاوت Recursive و Authoritative

DNS Server می‌تواند هم Recursive باشد، هم Authoritative.

در سناریوی شما Technitium هر دو نقش را دارد.

## برای دامنه داخلی

<div dir="ltr" align="left">

```text
stage.30bime.ir
```

</div>

Technitium نقش Authoritative دارد، چون خودش Zone آن را دارد.

## برای دامنه خارجی

<div dir="ltr" align="left">

```text
google.com
```

</div>

Technitium نقش Recursive دارد، چون خودش مالک google.com نیست و از Forwarderها سوال می‌پرسد.

---

# ۷. Zone چیست؟

Zone یعنی محدوده‌ای از DNS که یک DNS Server مسئول مدیریت آن است.

مثلا وقتی در Technitium می‌سازی:

<div dir="ltr" align="left">

```text
Zone: stage.30bime.ir
```

</div>

یعنی Technitium مسئول جواب دادن برای این نام است:

<div dir="ltr" align="left">

```text
stage.30bime.ir
```

</div>

اگر داخل همین Zone رکورد `api` بسازی:

<div dir="ltr" align="left">

```text
api A 172.24.11.129
```

</div>

نتیجه می‌شود:

<div dir="ltr" align="left">

```text
api.stage.30bime.ir -> 172.24.11.129
```

</div>

---

# ۸. تفاوت Zone و Record

این دو مفهوم خیلی مهم هستند.

## Zone

Zone محدوده مدیریتی است.

مثلا:

<div dir="ltr" align="left">

```text
stage.30bime.ir
```

</div>

## Record

Record اطلاعات داخل Zone است.

مثلا:

<div dir="ltr" align="left">

```text
@ A 172.24.11.128
```

</div>

نتیجه:

<div dir="ltr" align="left">

```text
stage.30bime.ir -> 172.24.11.128
```

</div>

مثال کامل:

<div dir="ltr" align="left">

```text
Zone:
  stage.30bime.ir

Record:
  Name: @
  Type: A
  Data: 172.24.11.128
  TTL: 60
```

</div>

---

# ۹. علامت @ در DNS یعنی چه؟

در DNS، علامت `@` یعنی خود نام Zone.

اگر Zone این باشد:

<div dir="ltr" align="left">

```text
stage.30bime.ir
```

</div>

و رکورد این باشد:

<div dir="ltr" align="left">

```text
@ A 172.24.11.128
```

</div>

یعنی:

<div dir="ltr" align="left">

```text
stage.30bime.ir A 172.24.11.128
```

</div>

اگر داخل همین Zone رکورد زیر را بسازی:

<div dir="ltr" align="left">

```text
api A 172.24.11.129
```

</div>

نتیجه می‌شود:

<div dir="ltr" align="left">

```text
api.stage.30bime.ir -> 172.24.11.129
```

</div>

---

# ۱۰. Record Typeهای مهم DNS

## ۱۰.۱. A Record

برای تبدیل دامنه به IPv4 استفاده می‌شود.

مثال:

<div dir="ltr" align="left">

```text
stage.30bime.ir A 172.24.11.128
```

</div>

این پرکاربردترین رکورد برای شبکه داخلی شماست.

---

## ۱۰.۲. AAAA Record

برای تبدیل دامنه به IPv6 استفاده می‌شود.

مثال:

<div dir="ltr" align="left">

```text
example.com AAAA 2001:db8::1
```

</div>

اگر در شبکه شرکت IPv6 نداری، فعلا خیلی درگیر این رکورد نشو.

---

## ۱۰.۳. CNAME Record

CNAME یعنی Alias یا نام مستعار.

مثال:

<div dir="ltr" align="left">

```text
www CNAME stage.30bime.ir
```

</div>

یعنی `www` به جای اینکه خودش IP داشته باشد، به `stage.30bime.ir` اشاره می‌کند.

نکته مهم:

یک Name نباید هم‌زمان CNAME و رکوردهای دیگر مثل A یا MX داشته باشد.

اشتباه:

<div dir="ltr" align="left">

```text
app A 172.24.11.128
app CNAME stage.30bime.ir
```

</div>

درست:

<div dir="ltr" align="left">

```text
app CNAME stage.30bime.ir
```

</div>

یا:

<div dir="ltr" align="left">

```text
app A 172.24.11.128
```

</div>

---

## ۱۰.۴. MX Record

برای ایمیل استفاده می‌شود.

مثال:

<div dir="ltr" align="left">

```text
30bime.ir MX mail.30bime.ir
```

</div>

یعنی ایمیل‌های دامنه `30bime.ir` باید به سرور `mail.30bime.ir` ارسال شوند.

برای Mail Server، Mailcow، Exchange و سرویس‌های ایمیلی، MX بسیار مهم است.

---

## ۱۰.۵. TXT Record

برای ذخیره متن و تنظیمات امنیتی استفاده می‌شود.

کاربردهای رایج:

<div dir="ltr" align="left">

```text
SPF
DKIM
DMARC
Domain Verification
Google Verification
Microsoft 365 Verification
```

</div>

مثال SPF:

<div dir="ltr" align="left">

```text
30bime.ir TXT "v=spf1 mx -all"
```

</div>

---

## ۱۰.۶. NS Record

NS مشخص می‌کند Name Server یک Zone کیست.

مثلا در Technitium ممکن است ببینی:

<div dir="ltr" align="left">

```text
stage.30bime.ir NS dns.30bime.local
```

</div>

یعنی DNS Server مسئول Zone برابر است با:

<div dir="ltr" align="left">

```text
dns.30bime.local
```

</div>

---

## ۱۰.۷. SOA Record

SOA مخفف **Start of Authority** است.

این رکورد اطلاعات مدیریتی Zone را نگه می‌دارد، مثل:

<div dir="ltr" align="left">

```text
Primary Name Server
Responsible Person
Serial Number
Refresh
Retry
Expire
Minimum TTL
```

</div>

تقریبا هر Zone باید یک رکورد SOA داشته باشد.

---

## ۱۰.۸. PTR Record

PTR برای Reverse DNS استفاده می‌شود.

DNS عادی:

<div dir="ltr" align="left">

```text
stage.30bime.ir -> 172.24.11.128
```

</div>

Reverse DNS:

<div dir="ltr" align="left">

```text
172.24.11.128 -> stage.30bime.ir
```

</div>

برای لاگ‌ها، مانیتورینگ، امنیت و Mail Server مهم است.

برای رنج داخلی می‌توان Zone معکوس ساخت، مثلا برای شبکه:

<div dir="ltr" align="left">

```text
172.24.11.0/24
```

</div>

Zone معکوس می‌شود:

<div dir="ltr" align="left">

```text
11.24.172.in-addr.arpa
```

</div>

---

# ۱۱. Forwarder چیست؟

Forwarder یعنی DNS بالادستی.

وقتی Technitium جواب یک دامنه را خودش ندارد، درخواست را به Forwarder می‌فرستد.

در سناریوی شما Forwarderها این‌ها هستند:

<div dir="ltr" align="left">

```text
178.22.122.101
185.51.200.1
```

</div>

رفتار کلی:

<div dir="ltr" align="left">

```text
stage.30bime.ir -> جواب از Zone داخلی
google.com       -> ارسال به Forwarder
docker.com       -> ارسال به Forwarder
```

</div>

مسیر در Technitium معمولا:

<div dir="ltr" align="left">

```text
Settings -> Proxy & Forwarders -> Forwarders
```

</div>

---

# ۱۲. اگر چند Forwarder داشته باشیم، ترتیب استفاده چگونه است؟

در Global Forwarderها، Technitium معمولا به شکل ساده و خطی مثل زیر کار نمی‌کند:

<div dir="rtl" align="right">

```text
اول Forwarder 1
بعد Forwarder 2
بعد Forwarder 3
```

</div>

بلکه ممکن است بر اساس latency، وضعیت پاسخ‌گویی، cache و تنظیمات داخلی، از سریع‌ترین یا مناسب‌ترین Forwarder استفاده کند.

مثال:

<div dir="ltr" align="left">

```text
Forwarders:
178.22.122.101
185.51.200.1
```

</div>

ممکن است گاهی جواب از اولی گرفته شود و گاهی از دومی.

اگر اولویت دقیق و Failover ترتیبی بخواهی، باید از Conditional Forwarder Zone و رکوردهای FWD با Priority استفاده کنی.

اما برای سناریوی فعلی شرکت، ساده‌ترین حالت این است:

<div dir="ltr" align="left">

```text
Global Forwarders:
178.22.122.101
185.51.200.1
```

</div>

---

# ۱۳. Cache چیست؟

Cache یعنی DNS Server جواب‌ها را برای مدتی نگه می‌دارد تا هر بار مجبور نباشد دوباره سوال کند.

مثلا اگر یک کلاینت بپرسد:

<div dir="ltr" align="left">

```text
google.com
```

</div>

Technitium جواب را از Forwarder می‌گیرد و برای مدتی نگه می‌دارد.

دفعه بعد اگر کلاینت دیگری همان دامنه را بپرسد، Technitium سریع‌تر جواب می‌دهد.

مزایا:

<div dir="rtl" align="right">

```text
سرعت بیشتر
مصرف اینترنت کمتر
فشار کمتر روی Forwarder
کاهش latency
```

</div>

---

# ۱۴. TTL چیست؟

TTL مخفف **Time To Live** است.

TTL مشخص می‌کند یک رکورد DNS چه مدت می‌تواند Cache شود.

مثلا:

<div dir="ltr" align="left">

```text
TTL: 60
```

</div>

یعنی جواب این رکورد برای ۶۰ ثانیه Cache شود.

برای رکوردهای داخلی در زمان تست، TTL پایین مناسب است:

<div dir="ltr" align="left">

```text
60 seconds
```

</div>

برای رکوردهای عملیاتی و پایدار، می‌توان مقدار بالاتری گذاشت:

<div dir="ltr" align="left">

```text
300 seconds
900 seconds
3600 seconds
```

</div>

پیشنهاد:

<div dir="ltr" align="left">

```text
TTL 60    = مناسب تست و تغییرات زیاد
TTL 300   = مناسب محیط عملیاتی با تغییرات متوسط
TTL 3600  = مناسب رکوردهای پایدار
```

</div>

---

# ۱۵. Split DNS چیست؟

Split DNS یعنی یک دامنه داخل شبکه شرکت یک جواب بدهد و بیرون شرکت جواب دیگری.

مثلا داخل شرکت:

<div dir="ltr" align="left">

```text
stage.30bime.ir -> 172.24.11.128
```

</div>

ولی بیرون شرکت ممکن است:

<div dir="ltr" align="left">

```text
stage.30bime.ir -> Public IP
```

</div>

یا اصلا Resolve نشود.

این روش در شرکت‌ها بسیار رایج است.

کاربردها:

<div dir="rtl" align="right">

```text
سرویس‌های داخلی
Stage Environment
Dev Environment
GitLab داخلی
Harbor داخلی
Monitoring داخلی
Panelهای مدیریتی
```

</div>

---

# ۱۶. چرا نباید بی‌دلیل Zone کل دامنه اصلی را بسازیم؟

اگر در Technitium این Zone را بسازی:

<div dir="ltr" align="left">

```text
30bime.ir
```

</div>

Technitium فکر می‌کند مسئول کل دامنه `30bime.ir` است.

بعد اگر رکوردهای لازم را داخل آن نسازی، ممکن است کلاینت‌های داخلی نتوانند رکوردهای Public واقعی دامنه را ببینند.

مثلا ممکن است این‌ها خراب شوند:

<div dir="ltr" align="left">

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

</div>

برای همین اگر فقط می‌خواهی این دامنه داخلی شود:

<div dir="ltr" align="left">

```text
stage.30bime.ir
```

</div>

بهتر است فقط Zone همین را بسازی:

<div dir="ltr" align="left">

```text
stage.30bime.ir
```

</div>

نه کل:

<div dir="ltr" align="left">

```text
30bime.ir
```

</div>

مگر اینکه بخواهی کل DNS داخلی دامنه شرکت را کامل مدیریت کنی.

---

# ۱۷. Primary Zone چیست؟

Primary Zone یعنی این DNS Server مالک اصلی Zone است و رکوردها مستقیم روی همین سرور ساخته و ویرایش می‌شوند.

مثال:

<div dir="ltr" align="left">

```text
Zone: stage.30bime.ir
Type: Primary
```

</div>

یعنی Technitium خودش اطلاعات این Zone را نگه می‌دارد.

برای سناریوی شما، `stage.30bime.ir` باید Primary Zone باشد.

---

# ۱۸. Secondary Zone چیست؟

Secondary Zone یک کپی از Zone روی DNS Server دیگر است.

مثلا اگر دو DNS داخلی داشته باشی:

<div dir="ltr" align="left">

```text
DNS1: 172.24.11.10
DNS2: 172.24.11.11
```

</div>

می‌توانی:

<div dir="ltr" align="left">

```text
DNS1 = Primary
DNS2 = Secondary
```

</div>

DNS2 رکوردها را از DNS1 دریافت می‌کند.

این کار برای High Availability و Redundancy مناسب است.

---

# ۱۹. Conditional Forwarder چیست؟

Conditional Forwarder یعنی برای یک دامنه خاص، درخواست‌ها به DNS خاصی فرستاده شوند.

مثلا:

<div dir="ltr" align="left">

```text
corp.local     -> 172.24.11.20
example.local  -> 172.24.11.30
```

</div>

یا:

<div dir="ltr" align="left">

```text
company.internal -> DNS Server داخلی دیگر
```

</div>

کاربرد در شرکت‌های بزرگ:

<div dir="rtl" align="right">

```text
ارتباط بین چند شعبه
ارتباط با Active Directory
ارتباط با DNSهای داخلی دیگر
ارتباط بین چند دیتاسنتر
```

</div>

---

# ۲۰. Wildcard DNS چیست؟

Wildcard یعنی هر زیر دامنه‌ای جواب بگیرد.

مثلا اگر داخل Zone `stage.30bime.ir` این رکورد را بسازی:

<div dir="ltr" align="left">

```text
* A 172.24.11.128
```

</div>

این‌ها همه به همان IP resolve می‌شوند:

<div dir="ltr" align="left">

```text
api.stage.30bime.ir
panel.stage.30bime.ir
test.stage.30bime.ir
anything.stage.30bime.ir
```

</div>

مزیت:

<div dir="rtl" align="right">

```text
برای Traefik/Nginx و سرویس‌های زیاد کاربردی است
نیاز نیست برای هر subdomain رکورد جدا بسازی
```

</div>

عیب:

<div dir="rtl" align="right">

```text
ممکن است خطاها را مخفی کند
هر اسم اشتباهی هم جواب می‌گیرد
```

</div>

برای شروع بهتر است فقط رکوردهای مشخص بسازی.

---

# ۲۱. DNS روی چه پورتی کار می‌کند؟

DNS معمولا روی پورت‌های زیر کار می‌کند:

<div dir="ltr" align="left">

```text
UDP 53
TCP 53
```

</div>

اکثر Queryهای معمولی با UDP انجام می‌شوند.

TCP برای حالت‌هایی مثل موارد زیر استفاده می‌شود:

<div dir="rtl" align="right">

```text
جواب‌های بزرگ
Zone Transfer
DNSSEC
برخی Queryهای خاص
```

</div>

پس روی فایروال باید هر دو را باز کنی:

<div dir="ltr" align="left">

```text
53/udp
53/tcp
```

</div>

پنل Technitium معمولا روی این پورت است:

<div dir="ltr" align="left">

```text
5380/tcp
```

</div>

---

# ۲۲. چرا نباید DNS خارجی را به عنوان DNS دوم روی کلاینت‌ها بگذاریم؟

اشتباه رایج:

<div dir="ltr" align="left">

```text
DNS 1: 172.24.11.10
DNS 2: 178.22.122.101
```

</div>

مشکل اینجاست که سیستم عامل، مخصوصا ویندوز، همیشه تضمین نمی‌کند فقط از DNS اول استفاده کند. ممکن است گاهی مستقیم از DNS دوم بپرسد.

اگر از DNS دوم خارجی بپرسد، رکورد داخلی را نمی‌شناسد:

<div dir="ltr" align="left">

```text
stage.30bime.ir -> 172.24.11.128
```

</div>

پس ممکن است نتیجه این شود:

<div dir="rtl" align="right">

```text
گاهی سایت باز می‌شود
گاهی سایت باز نمی‌شود
گاهی IP اشتباه برمی‌گردد
```

</div>

حالت درست:

<div dir="ltr" align="left">

```text
Client DNS:
DNS 1: 172.24.11.10
DNS 2: 172.24.11.11
```

</div>

و داخل Technitium:

<div dir="ltr" align="left">

```text
Forwarders:
178.22.122.101
185.51.200.1
```

</div>

اگر DNS دوم می‌خواهی، باید یک DNS داخلی دوم با همان رکوردها داشته باشی.

---

# ۲۳. DHCP چه ربطی به DNS دارد؟

DHCP به کلاینت‌ها IP می‌دهد.

علاوه بر IP، DHCP می‌تواند DNS Server را هم به سیستم‌ها بدهد.

به جای اینکه روی تک‌تک سیستم‌ها دستی DNS تنظیم کنی، در DHCP تنظیم می‌کنی:

<div dir="ltr" align="left">

```text
DNS Server = 172.24.11.10
```

</div>

بعد همه سیستم‌ها به صورت خودکار از DNS داخلی شرکت استفاده می‌کنند.

مثال در MikroTik:

<div dir="ltr" align="left">

```mikrotik
/ip dhcp-server network set [find address=172.24.11.0/24] dns-server=172.24.11.10
```

</div>

---

# ۲۴. ترتیب Resolve در سناریوی شما

وقتی کلاینت می‌پرسد:

<div dir="ltr" align="left">

```text
stage.30bime.ir
```

</div>

تقریبا این مراحل طی می‌شود:

<div dir="rtl" align="right">

```text
1. کلاینت Cache خودش را چک می‌کند
2. اگر جواب نداشت، از DNS تنظیم‌شده روی کارت شبکه می‌پرسد
3. Technitium Cache خودش را چک می‌کند
4. Technitium Zoneهای داخلی را چک می‌کند
5. اگر رکورد داخلی پیدا شد، جواب می‌دهد
6. اگر رکورد داخلی نبود، درخواست را به Forwarder می‌فرستد
7. جواب را به کلاینت برمی‌گرداند
```

</div>

برای دامنه داخلی:

<div dir="ltr" align="left">

```text
stage.30bime.ir -> جواب از Zone داخلی Technitium
```

</div>

برای دامنه خارجی:

<div dir="ltr" align="left">

```text
google.com -> جواب از Forwarder
```

</div>

---

# ۲۵. DNS با HTTP فرق دارد

وقتی کاربر می‌زند:

<div dir="ltr" align="left">

```text
http://stage.30bime.ir
```

</div>

دو مرحله جدا اتفاق می‌افتد.

## مرحله اول: DNS

<div dir="ltr" align="left">

```text
stage.30bime.ir -> 172.24.11.128
```

</div>

## مرحله دوم: HTTP

سیستم به این IP وصل می‌شود:

<div dir="ltr" align="left">

```text
172.24.11.128:80
```

</div>

پس اگر DNS درست باشد ولی سایت باز نشود، مشکل ممکن است DNS نباشد.

مشکلات احتمالی:

<div dir="ltr" align="left">

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

</div>

تست‌های مفید:

<div dir="ltr" align="left">

```bash
dig @172.24.11.10 stage.30bime.ir +short
curl -I http://stage.30bime.ir
curl -I -H "Host: stage.30bime.ir" http://172.24.11.128
```

</div>

---

# ۲۶. ابزارهای مهم عیب‌یابی DNS

## ۲۶.۱. dig

بهترین ابزار برای تست DNS در Linux.

تست رکورد داخلی:

<div dir="ltr" align="left">

```bash
dig @172.24.11.10 stage.30bime.ir
```

</div>

خروجی خلاصه:

<div dir="ltr" align="left">

```bash
dig @172.24.11.10 stage.30bime.ir +short
```

</div>

تست دامنه اینترنتی:

<div dir="ltr" align="left">

```bash
dig @172.24.11.10 google.com +short
```

</div>

تست با TCP:

<div dir="ltr" align="left">

```bash
dig @172.24.11.10 google.com +tcp +short
```

</div>

Trace مسیر DNS:

<div dir="ltr" align="left">

```bash
dig +trace stage.30bime.ir
```

</div>

---

## ۲۶.۲. nslookup

روی Windows و Linux قابل استفاده است.

<div dir="ltr" align="left">

```powershell
nslookup stage.30bime.ir 172.24.11.10
nslookup google.com 172.24.11.10
```

</div>

---

## ۲۶.۳. resolvectl

روی Ubuntuهای جدید:

<div dir="ltr" align="left">

```bash
resolvectl status
resolvectl query stage.30bime.ir
```

</div>

پاک کردن Cache:

<div dir="ltr" align="left">

```bash
sudo resolvectl flush-caches
```

</div>

---

## ۲۶.۴. ipconfig در Windows

دیدن تنظیمات DNS:

<div dir="ltr" align="left">

```powershell
ipconfig /all
```

</div>

پاک کردن DNS Cache:

<div dir="ltr" align="left">

```powershell
ipconfig /flushdns
```

</div>

---

## ۲۶.۵. ss برای بررسی پورت 53

<div dir="ltr" align="left">

```bash
sudo ss -lntup | grep ':53'
```

</div>

اگر پورت 53 اشغال بود، ممکن است یکی از این سرویس‌ها فعال باشد:

<div dir="ltr" align="left">

```text
systemd-resolved
dnsmasq
bind9
named
AdGuard Home
Pi-hole
Technitium قبلی
```

</div>

---

# ۲۷. systemd-resolved چیست؟

در Ubuntuهای جدید، سرویس `systemd-resolved` معمولا DNS سیستم را مدیریت می‌کند.

گاهی این سرویس پورت 53 را اشغال می‌کند و اجازه نمی‌دهد Docker DNS Server روی پورت 53 بالا بیاید.

خطای رایج Docker:

<div dir="ltr" align="left">

```text
failed to bind host port 0.0.0.0:53/tcp: address already in use
```

</div>

برای بررسی:

<div dir="ltr" align="left">

```bash
sudo ss -lntup | grep ':53'
```

</div>

اگر `systemd-resolved` پورت 53 را گرفته بود، می‌توان DNS Stub Listener را غیرفعال کرد:

<div dir="ltr" align="left">

```bash
sudo nano /etc/systemd/resolved.conf
```

</div>

تنظیم:

<div dir="ltr" align="left">

```ini
[Resolve]
DNS=178.22.122.101 185.51.200.1
FallbackDNS=8.8.8.8 1.1.1.1
DNSStubListener=no
```

</div>

اعمال:

<div dir="ltr" align="left">

```bash
sudo systemctl restart systemd-resolved
sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

</div>

---

# ۲۸. DNS Cache در چند جا وجود دارد

وقتی یک رکورد DNS را تغییر می‌دهی، ممکن است همان لحظه روی کلاینت تغییر دیده نشود.

چون Cache در چند لایه وجود دارد:

<div dir="ltr" align="left">

```text
Browser Cache
Windows DNS Cache
Linux systemd-resolved Cache
Technitium Cache
Forwarder Cache
ISP DNS Cache
```

</div>

برای پاک کردن Cache ویندوز:

<div dir="ltr" align="left">

```powershell
ipconfig /flushdns
```

</div>

برای Ubuntu:

<div dir="ltr" align="left">

```bash
sudo resolvectl flush-caches
```

</div>

در Technitium هم از تب Cache می‌توانی Cache را ببینی یا پاک کنی.

---

# ۲۹. DoH چیست و چرا ممکن است مشکل‌ساز شود؟

DoH مخفف **DNS over HTTPS** است.

بعضی مرورگرها مثل Chrome و Firefox می‌توانند DNS را از طریق HTTPS و مستقل از DNS سیستم بفرستند.

مثلا به جای اینکه از DNS شرکت بپرسند:

<div dir="ltr" align="left">

```text
172.24.11.10
```

</div>

مستقیم از این‌ها می‌پرسند:

<div dir="ltr" align="left">

```text
Cloudflare DoH
Google DoH
```

</div>

در این حالت رکوردهای داخلی شرکت Resolve نمی‌شوند.

مثلا:

<div dir="ltr" align="left">

```text
stage.30bime.ir -> 172.24.11.128
```

</div>

ممکن است در مرورگر Resolve نشود، چون مرورگر از DNS داخلی استفاده نکرده است.

برای شبکه شرکتی بهتر است DoH مرورگرها با Policy کنترل یا غیرفعال شود.

---

# ۳۰. DNSSEC چیست؟

DNSSEC برای اعتبارسنجی جواب‌های DNS است.

هدف DNSSEC این است که مطمئن شویم جواب DNS دستکاری نشده است.

برای DNS داخلی ساده، فعلا ضروری نیست.

ولی برای DNS عمومی و دامنه‌های Public، DNSSEC می‌تواند اهمیت داشته باشد.

---

# ۳۱. Reverse DNS چیست؟

DNS معمولی:

<div dir="ltr" align="left">

```text
stage.30bime.ir -> 172.24.11.128
```

</div>

Reverse DNS:

<div dir="ltr" align="left">

```text
172.24.11.128 -> stage.30bime.ir
```

</div>

Reverse DNS با رکورد PTR انجام می‌شود.

برای شبکه داخلی می‌توانی بعدا Reverse Zone بسازی:

<div dir="ltr" align="left">

```text
11.24.172.in-addr.arpa
```

</div>

برای Mail Server عمومی، PTR بسیار مهم است و معمولا باید توسط ISP یا دیتاسنتر تنظیم شود.

---

# ۳۲. Recursion Policy چیست؟

Recursion Policy مشخص می‌کند چه کسانی اجازه دارند از DNS Server برای Resolve دامنه‌های خارجی استفاده کنند.

برای DNS داخلی شرکت نباید Recursion برای همه اینترنت باز باشد.

اشتباه خطرناک:

<div dir="ltr" align="left">

```text
Allow recursion for everyone
```

</div>

حالت درست:

<div dir="ltr" align="left">

```text
Allow recursion only for private networks
```

</div>

یا فقط برای شبکه شرکت:

<div dir="ltr" align="left">

```text
172.24.11.0/24
```

</div>

در Technitium می‌توانی این مورد را از Settings کنترل کنی.

---

# ۳۳. Open Resolver چیست؟

Open Resolver یعنی DNS Server شما برای همه اینترنت Recursive Query انجام دهد.

این خطرناک است.

مشکلات Open Resolver:

<div dir="rtl" align="right">

```text
استفاده در DNS Amplification Attack
فشار زیاد روی سرور
ریسک امنیتی
مصرف پهنای باند
قرار گرفتن IP در لیست‌های امنیتی
```

</div>

پس DNS داخلی شرکت باید فقط از شبکه داخلی جواب Recursive بدهد.

---

# ۳۴. DNS و Firewall

برای اینکه کلاینت‌ها بتوانند از DNS Server استفاده کنند، باید پورت‌های زیر از شبکه داخلی باز باشد:

<div dir="ltr" align="left">

```text
53/udp
53/tcp
```

</div>

برای پنل Technitium:

<div dir="ltr" align="left">

```text
5380/tcp
```

</div>

پیشنهاد امنیتی:

<div dir="rtl" align="right">

```text
پورت 53 فقط برای شبکه داخلی باز باشد
پورت 5380 فقط برای مدیران شبکه باز باشد
```

</div>

مثال UFW:

<div dir="ltr" align="left">

```bash
sudo ufw allow from 172.24.11.0/24 to any port 53 proto udp
sudo ufw allow from 172.24.11.0/24 to any port 53 proto tcp
sudo ufw allow from 172.24.11.0/24 to any port 5380 proto tcp
```

</div>

---

# ۳۵. DNS و Docker

وقتی DNS Server را با Docker اجرا می‌کنی، باید پورت 53 روی Host آزاد باشد.

بررسی پورت:

<div dir="ltr" align="left">

```bash
sudo ss -lntup | grep ':53'
```

</div>

اگر پورت اشغال باشد، Docker خطا می‌دهد:

<div dir="ltr" align="left">

```text
address already in use
```

</div>

در Docker Compose بهتر است پورت‌ها را روی IP داخلی Bind کنی، نه روی همه IPها.

مثال بهتر:

<div dir="ltr" align="left">

```yaml
ports:
  - "172.24.11.10:53:53/udp"
  - "172.24.11.10:53:53/tcp"
  - "172.24.11.10:5380:5380/tcp"
```

</div>

به جای:

<div dir="ltr" align="left">

```yaml
ports:
  - "53:53/udp"
  - "53:53/tcp"
  - "5380:5380/tcp"
```

</div>

---

# ۳۶. مثال Docker Compose برای Technitium

نمونه Compose عملیاتی:

<div dir="ltr" align="left">

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

</div>

نمونه `.env`:

<div dir="ltr" align="left">

```env
DNS_SERVER_IP=172.24.11.10
DNS_SERVER_DOMAIN=dns.30bime.local
TZ=Asia/Tehran
```

</div>

---

# ۳۷. DNS_SERVER_DOMAIN در Technitium چیست؟

این env:

<div dir="ltr" align="left">

```env
DNS_SERVER_DOMAIN=dns.30bime.local
```

</div>

برای تعیین نام خود DNS Server است.

یعنی Technitium خودش را با این نام می‌شناسد:

<div dir="ltr" align="left">

```text
dns.30bime.local
```

</div>

این مقدار ربط مستقیم به رکورد زیر ندارد:

<div dir="ltr" align="left">

```text
stage.30bime.ir -> 172.24.11.128
```

</div>

رکوردهای دامنه باید جداگانه داخل Zone ساخته شوند.

پیشنهاد برای شبکه داخلی:

<div dir="ltr" align="left">

```env
DNS_SERVER_DOMAIN=dns.30bime.local
```

</div>

---

# ۳۸. ساخت Zone برای stage.30bime.ir در Technitium

داخل پنل Technitium:

<div dir="ltr" align="left">

```text
Zones -> Add Zone
```

</div>

مقادیر:

<div dir="ltr" align="left">

```text
Zone: stage.30bime.ir
Type: Primary Zone
```

</div>

بعد داخل Zone:

<div dir="ltr" align="left">

```text
Add Record
```

</div>

مقادیر رکورد:

<div dir="ltr" align="left">

```text
Name: @
Type: A
TTL: 60
IPv4 Address: 172.24.11.128
```

</div>

نتیجه:

<div dir="ltr" align="left">

```text
stage.30bime.ir -> 172.24.11.128
```

</div>

---

# ۳۹. تست نهایی DNS

## تست رکورد داخلی

<div dir="ltr" align="left">

```bash
dig @172.24.11.10 stage.30bime.ir +short
```

</div>

خروجی مورد انتظار:

<div dir="ltr" align="left">

```text
172.24.11.128
```

</div>

## تست دامنه اینترنتی

<div dir="ltr" align="left">

```bash
dig @172.24.11.10 google.com +short
```

</div>

باید IP برگرداند.

## تست TCP DNS

<div dir="ltr" align="left">

```bash
dig @172.24.11.10 google.com +tcp +short
```

</div>

## تست از Windows

<div dir="ltr" align="left">

```powershell
nslookup stage.30bime.ir 172.24.11.10
nslookup google.com 172.24.11.10
```

</div>

## تست HTTP

<div dir="ltr" align="left">

```bash
curl -I http://stage.30bime.ir
```

</div>

یا:

<div dir="ltr" align="left">

```bash
curl -I -H "Host: stage.30bime.ir" http://172.24.11.128
```

</div>

---

# ۴۰. چک‌لیست عملیاتی DNS شرکت

برای محیط شرکت، این موارد مهم هستند:

<div dir="rtl" align="right">

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

</div>

---

# ۴۱. خلاصه اصطلاحات مهم

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

# ۴۲. معماری پیشنهادی نهایی

<div dir="ltr" align="left">

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

</div>

نتیجه:

<div dir="rtl" align="right">

```text
دامنه داخلی؟       جواب از Technitium
دامنه غیر داخلی؟   ارسال به Forwarderها
```

</div>

---

# ۴۳. دستورهای کاربردی سریع

## بررسی پورت DNS

<div dir="ltr" align="left">

```bash
sudo ss -lntup | grep ':53'
```

</div>

## تست رکورد داخلی

<div dir="ltr" align="left">

```bash
dig @172.24.11.10 stage.30bime.ir +short
```

</div>

## تست Forwarder

<div dir="ltr" align="left">

```bash
dig @172.24.11.10 google.com +short
```

</div>

## پاک کردن Cache در Windows

<div dir="ltr" align="left">

```powershell
ipconfig /flushdns
```

</div>

## پاک کردن Cache در Ubuntu

<div dir="ltr" align="left">

```bash
sudo resolvectl flush-caches
```

</div>

## دیدن DNS Client در Windows

<div dir="ltr" align="left">

```powershell
ipconfig /all
```

</div>

## دیدن DNS در Ubuntu

<div dir="ltr" align="left">

```bash
resolvectl status
```

</div>

## تست سایت با دامنه

<div dir="ltr" align="left">

```bash
curl -I http://stage.30bime.ir
```

</div>

## تست سایت با IP و Host Header

<div dir="ltr" align="left">

```bash
curl -I -H "Host: stage.30bime.ir" http://172.24.11.128
```

</div>

---

# ۴۴. نتیجه‌گیری

برای شبکه شرکت، لازم نیست همه جزئیات سطح ISP یا Public DNS را بلد باشی. اما این مفاهیم برای مدیریت DNS داخلی ضروری هستند:

<div dir="ltr" align="left">

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

</div>

با همین مفاهیم می‌توانی اکثر نیازهای DNS داخلی شرکت را مدیریت کنی، از جمله:

<div dir="rtl" align="right">

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


