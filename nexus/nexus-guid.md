<div dir="rtl">

# راهنمای کاربردی Nexus Repository برای مهندس DevOps

محدوده: Nexus Repository 3، نصب Self-hosted. نام و محل بعضی منوها و قابلیت‌ها به نسخه، Edition و سطح دسترسی بستگی دارد. مثال‌ها آموزشی‌اند و روی زیرساخت شما اجرا نشده‌اند.

## ۱. Nexus چیست و کجای DevOps قرار می‌گیرد؟

Nexus مخزن مرکزی **Artifact** است؛ یعنی خروجی قابل انتشار مانند Docker Image، فایل JAR، پکیج npm یا Python و فایل‌های عمومی. Git کد منبع را نگه می‌دارد؛ Nexus بسته‌ها و خروجی‌های Build را مدیریت می‌کند.

کاربردهای اصلی:

- **افزایش سرعت Build:** کش‌کردن Dependencyهای خارجی و کاهش دانلود تکراری.
- **انتشار داخلی:** نگهداری بسته‌های خصوصی و خروجی Pipelineها.
- **کنترل دسترسی:** تعیین اینکه چه کسی دانلود، انتشار یا حذف کند.
- **انتشار قابل ردیابی:** نگهداری نسخه مشخص برای Deploy و Rollback؛ یک Artifact را بساز و همان را بین محیط‌ها منتقل کن.

Nexus جای CI/CD یا اسکنر کامل امنیتی را نمی‌گیرد؛ این ابزارها را به هم متصل می‌کند. قابلیت‌های تحلیل و مسدودسازی امنیتی به محصول و مجوز فعال وابسته‌اند. [معرفی رسمی محصول](https://www.sonatype.com/products/sonatype-nexus-repository)

## ۲. سه نوع Repository که باید بلد باشی

| نوع | عملکرد | مثال |
|---|---|---|
| Hosted | نگهداری بسته‌هایی که خودت منتشر می‌کنی | `npm-internal` |
| Proxy | دریافت از مخزن خارجی و کش‌کردن درخواست‌ها | `npm-proxy` برای npmjs |
| Group | ارائه چند مخزن هم‌فرمت با یک آدرس دانلود | `npm-all` شامل دو مورد بالا |

**الگوی پیشنهادی:** دانلود از Group، انتشار مستقیم در Hosted. بعضی نسخه‌ها/فرمت‌ها قابلیت انتشار از طریق Group دارند؛ این راهنما به آن وابسته نیست. Proxy معمولاً فقط محتوای درخواست‌شده را کش می‌کند؛ یک Mirror کامل یا تضمین کارکرد آفلاین نیست. [انواع Repository](https://help.sonatype.com/en/repository-types.html)

## ۳. راه‌اندازی اولیه با Docker Compose

فایل `compose.yaml`:

```yaml
services:
  nexus:
    image: sonatype/nexus3:${NEXUS_VERSION:?Set a tested image tag}
    restart: unless-stopped
    ports:
      - "127.0.0.1:8081:8081"
    volumes:
      - nexus-data:/nexus-data
    stop_grace_period: 2m

volumes:
  nexus-data:
```

در `.env`، متغیر `NEXUS_VERSION` را برابر یک **Tag واقعی، پشتیبانی‌شده و تست‌شده** از Image رسمی قرار بده؛ برای Production از `latest` استفاده نکن.

```bash
docker compose up -d
docker compose logs -f nexus
# پس از آماده‌شدن سرویس؛ خروج از نمایش لاگ با Ctrl+C
docker compose exec nexus cat /nexus-data/admin.password
```

رابط وب روی `http://localhost:8081` است. با `admin` و رمز اولیه وارد شو، رمز را تغییر بده و در صورت نداشتن نیاز عمومی، Anonymous Access را خاموش کن. در نصب روی سرور، این Bind فقط محلی است؛ از SSH Tunnel یا Reverse Proxy استفاده کن.

داده‌ها در `/nexus-data` ماندگارند؛ در Bind Mount دسترسی UID مربوط به Image، معمولاً `200`، را رعایت کن. حذف Volume داده‌ها را از بین می‌برد. [Image رسمی و راه‌اندازی](https://github.com/sonatype/docker-nexus3)

## ۴. بخش‌های مهم Settings / Administration

این جدول نقشه کاربردی منوهای رایج است؛ نبودن گزینه می‌تواند به نسخه، مجوز یا Role مربوط باشد.

| بخش | گزینه | کاربرد |
|---|---|---|
| Repository | Repositories | ساخت و ویرایش مخزن، مشاهده URL و وضعیت |
| Repository | Blob Stores | محل ذخیره باینری‌ها؛ جدا از مفهوم مخزن |
| Repository | Cleanup Policies | تعیین شرایط حذف محتوای قدیمی |
| Repository | Routing Rules | محدودکردن مسیرهای قابل دریافت از مخزن خارجی |
| Security | Users | حساب‌های انسانی و حساب‌های CI |
| Security | Roles | مجموعه مجوزهای یک نقش |
| Security | Privileges | مجوزهای خواندن، انتشار، حذف یا مدیریت |
| Security | Realms | فعال‌سازی روش‌های احراز هویت |
| Security | Anonymous Access | تعیین دسترسی بدون ورود |
| Security | LDAP / SSO | اتصال هویت سازمانی، در صورت پشتیبانی |
| Security | Content Selectors | محدودکردن دسترسی به بخشی از محتوا |
| System | Tasks | زمان‌بندی Cleanup و کارهای نگهداری |
| System | HTTP | تنظیم ارتباط خروجی و پراکسی سازمانی |
| System | Email | تنظیم SMTP |
| System | Capabilities | قابلیت‌های تکمیلی مانند Base URL |
| System | API | مرجع REST API برای اتوماسیون |
| Support | Logs / System Information | عیب‌یابی، اطلاعات سیستم و بسته پشتیبانی |
| IQ Server | اتصال IQ | یکپارچه‌سازی با Lifecycle / Firewall |

مرجع ساختار منوها: [Nexus Repository Administration](https://help.sonatype.com/en/nexus-repository-administration.html). جزئیات دسترسی کاربران: [Access Control](https://help.sonatype.com/en/access-control.html).

## ۵. ساخت و تنظیم یک مخزن

از `Settings → Repository → Repositories → Create repository` شروع کن. **Format** مثل `npm` را متناسب با Package Manager انتخاب کن. [مدیریت مخزن](https://help.sonatype.com/en/repository-management.html)

| تنظیم | معنی و پیشنهاد |
|---|---|
| Name | نام روشن و پایدار؛ مانند `npm-internal` |
| Online | روشن باشد تا Client به مخزن دسترسی داشته باشد |
| Blob Store | محل ذخیره؛ برای Hostedهای ارزشمند، تفکیک عملیاتی مفید است |
| Remote Storage | آدرس مخزن بالادستی در Proxy |
| Member Repositories | اعضای Group؛ ترتیب را آگاهانه انتخاب کن |
| Deployment Policy | در Hosted انتشار مجدد را کنترل می‌کند؛ برای Releaseها در صورت پشتیبانی `Disable redeploy` |
| Strict Content Type Validation | اعتبارسنجی نوع فایل؛ معمولاً روشن بماند |
| Cache / Negative Cache | مدت اعتبار کش، از جمله نتیجه «پیدا نشد»؛ علت احتمالی 404 پس از انتشار |
| Cleanup Policies | سیاست حذف متصل به همین مخزن |

برای Maven، Release و Snapshot را جدا کن: اولی نسخه نهایی و دومی نسخه در حال توسعه است. گزینه‌های موجود به فرمت و نوع مخزن وابسته‌اند. [تنظیمات مخزن](https://help.sonatype.com/en/configurable-repository-fields.html)

## ۶. مثال عملی اتصال CI/CD

برای پروژه npm این سه مخزن را بساز:

| نام | نوع | تنظیم اصلی |
|---|---|---|
| `npm-internal` | npm hosted | انتشار بسته داخلی |
| `npm-proxy` | npm proxy | Remote Storage برابر `https://registry.npmjs.org` |
| `npm-all` | npm group | اعضا: internal و سپس proxy |

پس از تنظیم HTTPS و احراز هویت Client، الگوی فرمان‌ها:

```bash
# دریافت Dependencyها
npm ci --registry=https://nexus.example.com/repository/npm-all/

# انتشار خروجی؛ پس از Build و Test پروژه
npm publish --registry=https://nexus.example.com/repository/npm-internal/
```

این دامنه نمونه است. Credential را در Secret Store ابزار CI قرار بده؛ روش Token/Login باید با نسخه Nexus و npm سازگار باشد. ترتیب اعضای Group به‌تنهایی دفاع کامل در برابر Dependency Confusion نیست؛ برای نام‌های داخلی، Scope و Routing Rule مناسب تعریف کن.

برای Docker، مخزن `docker (hosted)` بساز، روش دسترسی Registry مانند Connector پشت HTTPS را تنظیم و `Docker Bearer Token Realm` را فعال کن. در روش Connector، پورت Registry از پورت UI مستقل است و باید در شبکه کانتینر/Reverse Proxy هم قابل دسترسی باشد. فرض کن `registry.example.com` به Hosted متصل شده است:

```bash
# NEXUS_USER و NEXUS_PASSWORD از Secretهای CI تزریق می‌شوند
printf '%s' "$NEXUS_PASSWORD" | docker login registry.example.com \
  --username "$NEXUS_USER" --password-stdin
docker tag myapp:1.0.0 registry.example.com/team/myapp:1.0.0
docker push registry.example.com/team/myapp:1.0.0
```

صرف اجرای Compose بالا، Docker Registry را آماده نمی‌کند. [Docker Registry](https://help.sonatype.com/en/docker-registry.html) و [Docker Authentication](https://help.sonatype.com/en/docker-authentication.html)

## ۷. امنیت و عملیات روزمره

- **حداقل دسترسی:** حساب دانلود فقط `read/browse`؛ حساب انتشار فقط مجوزهای لازم همان Hosted، معمولاً `add/edit/read/browse`. مجوز `delete` و Admin را به CI نده. [کنترل دسترسی](https://help.sonatype.com/en/access-control.html)
- **HTTPS:** Nexus را پشت Reverse Proxy با گواهی معتبر قرار بده؛ محدودیت حجم Upload و Timeout را متناسب با Artifactها تنظیم کن. پنل مدیریت را به شبکه موردنیاز محدود کن.
- **منابع:** RAM کانتینر فقط Heap نیست؛ Direct Memory، حافظه JVM و سیستم‌عامل را هم حساب کن. اندازه‌گذاری را با بار واقعی و الزامات نسخه انجام بده؛ برای کاهش مصرف، Limit دلخواه و بسیار کم نگذار.
- **مانیتورینگ:** فضای دیسک و نرخ رشد، latency دانلود/آپلود، خطاهای 5xx، مصرف حافظه و GC، اتصال Upstream و نتیجه Tasks را پایش کن.
- **نسخه‌گذاری:** از نسخه‌های یکتا و Lockfile استفاده کن؛ برای Imageهای Deployشده Digest را ثبت کن. نسخه‌های لازم برای Rollback را از حذف محافظت کن.

### Cleanup و Backup

Cleanup ابتدا محتوا را به‌صورت منطقی حذف می‌کند؛ برای آزادشدن فضا، Task مربوط به `Admin - Compact blob store` را زمان‌بندی کن. ابتدا دامنه حذف را بررسی کن. در Docker، پاک‌سازی Manifestها و Layerهای بدون استفاده مراحل مخصوص دارد؛ Imageهای موردنیاز Deploy را قربانی سیاست حذف نکن. [Cleanup Policies](https://help.sonatype.com/en/cleanup-policies.html)

Backup باید **Database، Blob Storeها، تنظیمات و کلیدهای لازم** را به‌صورت سازگار پوشش دهد. کپی زنده و ساده Volume تضمین سازگاری نیست. برای نصب کوچک، Backup در زمان توقف کامل سرویس ساده‌تر است؛ برای Database خارجی، روش رسمی همان Database را نیز رعایت کن. بازیابی را دوره‌ای در محیط جدا آزمایش کن. [Backup and Restore](https://help.sonatype.com/en/backup-and-restore.html)

پیش از Upgrade، مسیر ارتقا و تغییرات Database/Java را بررسی و Backup معتبر تهیه کن. زیادکردن Replicaهای یک نصب تک‌گره‌ای با Volume مشترک، معادل HA پشتیبانی‌شده نیست.

## ۸. عیب‌یابی سریع

این جدول نقطه شروع بررسی است، نه تشخیص قطعی:

| نشانه | بررسی اولیه |
|---|---|
| 401 | Credential، Token و Realm |
| 403 | Role و Privilege روی مخزن مقصد |
| 404 | URL، وجود نسخه، اعضای Group و Negative Cache |
| انتشار ناموفق | مقصد Hosted، Deployment Policy و تکراری‌بودن نسخه |
| 413 | محدودیت اندازه درخواست در Reverse Proxy |
| 502 / 504 | آماده‌بودن Nexus، Timeout، حافظه و شبکه |
| Proxy unavailable | DNS، TLS، دسترسی اینترنت و Auto-block |
| دیسک پس از Cleanup خالی نشده | اجرای Compact و مراحل پاک‌سازی مخصوص فرمت |

برای بررسی اولیه کانتینر: `docker compose logs --tail=200 nexus`.

## ۹. چک‌لیست آمادگی Production

- [ ] نسخه Image ثابت، پشتیبانی‌شده و آزمایش‌شده است.
- [ ] Storage ماندگار، هشدار فضای دیسک و ظرفیت کافی داریم.
- [ ] HTTPS، حساب اختصاصی CI و دسترسی حداقلی تنظیم شده‌اند.
- [ ] انتشار، دانلود و Deploy یک نسخه آزمایشی موفق بوده‌اند.
- [ ] Cleanup نسخه‌های لازم برای Rollback را حذف نمی‌کند.
- [ ] Backup و Restore واقعی آزمایش شده‌اند.
- [ ] محدودیت‌های Edition و نیاز به HA/SSO/قابلیت‌های تجاری بررسی شده‌اند.

نسخه رایگان از 3.77.0 با نام **Community Edition** عرضه می‌شود و محدودیت مصرف دارد؛ راهنماهای قدیمی OSS را بدون بررسی به نسخه فعلی تعمیم نده. [توضیح رسمی Edition](https://github.com/sonatype/docker-nexus3#announcing-nexus-repository-community-edition)

## ۱۰. سؤال‌های مصاحبه با پاسخ کوتاه

### مفاهیم پایه

**۱. چرا در کنار Git و Jenkins یا GitLab CI به Nexus نیاز داریم؟**

Git کد را نسخه‌بندی می‌کند، CI خروجی را می‌سازد و تست می‌کند، Nexus بسته‌ها و خروجی‌های قابل انتشار را نگه می‌دارد. با Nexus، تیم‌ها یک منبع مشترک برای Dependency و Artifact دارند.

**۲. تفاوت Hosted، Proxy و Group چیست؟**

Hosted برای انتشار بسته داخلی، Proxy برای دریافت و کش مخزن خارجی و Group برای تجمیع آدرس دانلود مخزن‌های هم‌فرمت است. الگوی معمول: دانلود از Group و انتشار در Hosted.

**۳. تفاوت Repository و Blob Store چیست؟**

Repository واحد منطقی ارائه محتوا با فرمت و سیاست دسترسی است؛ Blob Store محل ذخیره بایت‌های فایل‌هاست. چند Repository می‌توانند از یک Blob Store استفاده کنند؛ Database اطلاعات و ارتباطات محتوا را نگه می‌دارد. [Blob Stores](https://help.sonatype.com/en/blob-stores.html)

**۴. آیا با Proxy دیگر به اینترنت نیاز نداریم؟**

خیر؛ بسته‌ها و Metadata موردنیاز باید قبلاً کش شده باشند. محتوای جدید یا بررسی تازگی ممکن است به Upstream نیاز داشته باشد؛ کارکرد آفلاین را باید برای Build واقعی آزمایش کرد.

**۵. تفاوت Release و Snapshot چیست؟**
در Maven، Release نسخه نهایی با هویت ثابت است؛ Snapshot نسخه در حال توسعه است و می‌تواند تغییر کند. جداسازی آن‌ها اعمال سیاست انتشار و نگهداری متفاوت را ساده می‌کند.

**۶. چرا انتشار مجدد یک Release را محدود می‌کنیم؟**

اگر محتوای یک نسخه عوض شود، دو Deploy با شماره نسخه یکسان ممکن است خروجی متفاوتی داشته باشند. انتشار نسخه جدید و نگهداری خروجی قبلی، ردیابی و Rollback را قابل اعتمادتر می‌کند.

مرجع مفاهیم مخزن و سیاست انتشار: [Repository Types](https://help.sonatype.com/en/repository-types.html) و [Configurable Repository Fields](https://help.sonatype.com/en/configurable-repository-fields.html).

### امنیت و اتصال Pipeline

**۷. چه دسترسی‌ای به حساب CI می‌دهی؟**

فقط مجوزهای لازم روی مخزن هدف؛ برای دانلود `read/browse` و برای انتشار مجوزهای لازم مانند `add/edit`. حساب CI نباید Admin باشد؛ Credential در Secret Store نگهداری و دوره‌ای تعویض می‌شود.

**۸. تفاوت Role، Privilege و Realm چیست؟**

Privilege یک اجازه مشخص است؛ Role چند اجازه را برای انتساب به کاربر جمع می‌کند؛ Realm سازوکار احراز هویت یا یکپارچه‌سازی امنیتی را فعال می‌کند. Realm به‌تنهایی مجوز انتشار نمی‌دهد. [Access Control](https://help.sonatype.com/en/access-control.html)

**۹. ترتیب مناسب استفاده از Nexus در Pipeline چیست؟**

Dependencyها از Nexus دریافت می‌شوند؛ سپس Build و Test انجام می‌شود و خروجی با نسخه یکتا در Hosted منتشر می‌شود. محیط‌های بعدی همان Artifact تأییدشده را دریافت می‌کنند؛ برای هر محیط دوباره Build نمی‌کنیم.

**۱۰. چرا Docker Login موفق است ولی Push شکست می‌خورد؟**

Login فقط احراز هویت را نشان می‌دهد. مقصد Registry و Hostedبودن آن، مجوز انتشار، Deployment Policy، Tag و خطای Reverse Proxy را بررسی می‌کنم. مثلاً 403 بیشتر به دسترسی و 413 به اندازه Upload مربوط است. [Docker Authentication](https://help.sonatype.com/en/docker-authentication.html)

### سناریوهای عملی و سؤال‌های سطح بالاتر

**۱۱. بسته در Upstream منتشر شده، اما Nexus هنوز 404 می‌دهد؛ چه می‌کنی؟**

ابتدا نام، نسخه و URL را تأیید می‌کنم؛ سپس اعضای Group، Routing Rule و وضعیت Proxy را می‌بینم. Negative Cache ممکن است پاسخ قبلی «پیدا نشد» را نگه داشته باشد؛ پس از تأیید علت، کش مربوط را با روش پشتیبانی‌شده بی‌اعتبار می‌کنم یا منتظر انقضای آن می‌مانم.

**۱۲. Cleanup اجرا شده ولی دیسک خالی نشده؛ علت چیست؟**

حذف منطقی لزوماً بایت‌ها را فوراً حذف نمی‌کند. نتیجه Task و سپس اجرای `Admin - Compact blob store` را بررسی می‌کنم؛ در Docker، Manifestها و Layerهای بدون استفاده هم مراحل پاک‌سازی مخصوص دارند. [Cleanup Policies](https://help.sonatype.com/en/cleanup-policies.html)

**۱۳. دیسک Nexus تقریباً پر شده؛ اقدام فوری چیست؟**

رشد مصرف را پیدا و در صورت نیاز نوشتن غیرضروری را محدود می‌کنم؛ ظرفیت را افزایش می‌دهم یا Cleanup تأییدشده و Compact را اجرا می‌کنم. فایل‌های Blob Store را دستی حذف نمی‌کنم، چون هماهنگی با Database و سلامت محتوا به خطر می‌افتد.

**۱۴. آیا Backup از Blob Store کافی است؟**

خیر؛ Database، Blobها، تنظیمات و کلیدهای لازم باید مجموعه‌ای سازگار باشند. موفقیت Backup با Restore آزمایشی، دانلود Artifact و بررسی دسترسی‌ها سنجیده می‌شود. RPO یعنی میزان قابل‌قبول از دست رفتن داده و RTO یعنی زمان قابل‌قبول بازیابی سرویس. [Backup and Restore](https://help.sonatype.com/en/backup-and-restore.html)

**۱۵. Nexus کند شده؛ از کجا شروع می‌کنی؟**

اول مشخص می‌کنم کندی مربوط به UI، دانلود کش‌شده، دانلود از Upstream یا Upload است. سپس latency دیسک و شبکه، حافظه و GC، Database، خطاها و Tasks هم‌زمان را بررسی می‌کنم؛ افزایش RAM بدون یافتن گلوگاه، پاسخ کافی نیست.

**۱۶. چطور Upgrade و Rollback را برنامه‌ریزی می‌کنی؟**

نسخه و Database فعلی، مسیر ارتقای مجاز و تغییرات ناسازگار را بررسی می‌کنم؛ Backup و Restore را تست می‌کنم و ارتقا را ابتدا در محیط آزمایشی انجام می‌دهم. اگر Schema تغییر کند، اجرای Image قدیمی روی داده جدید الزاماً امن نیست؛ بازگشت باید داده و نرم‌افزار سازگار را بازیابی کند. [Upgrade Nexus Repository](https://help.sonatype.com/en/upgrade-nexus-repository.html)

**نکته مصاحبه:** در سؤال سناریویی، پاسخ را با «چه شواهدی جمع می‌کنم، چه فرضیه‌ای را بررسی می‌کنم و چطور موفقیت اصلاح را می‌سنجم» پیش ببر. تجربه واقعی خودت را مثال بزن و قابلیت‌های وابسته به نسخه یا Edition را قطعی فرض نکن.
