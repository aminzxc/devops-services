<div dir="rtl">

# راهنمای کاربردی Harbor برای مهندس DevOps

مبنای راهنما: مستندات Harbor 2.14؛ تاریخ بررسی: ۲۰۲۶/۰۹/۱۶. نام و جای بعضی گزینه‌ها با نسخه و سطح دسترسی فرق می‌کند. این نسخه، مبنای توضیح است و الزاماً آخرین نسخه نیست.

## ۱. Harbor چیست و کجا به کار می‌آید؟

**Harbor یک رجیستری متن‌باز برای نگهداری و توزیع Image کانتینر و OCI Artifact است.** کنترل دسترسی، اسکن آسیب‌پذیری، مدیریت پروژه و Replication را به رجیستری اضافه می‌کند. در DevOps، محل تحویل خروجی Build به سیستم استقرار است؛ خودش ابزار Build یا اجرای کانتینر نیست. [معرفی رسمی](https://goharbor.io/)

نمونهٔ مسیر یک Image:

```text
harbor.example.com/payments/api:1.4.2
```

| بخش | معنی |
|---|---|
| `harbor.example.com` | آدرس Registry |
| `payments` | Project؛ مرز دسترسی و سیاست‌ها |
| `api` | Repository؛ مجموعهٔ نسخه‌های یک برنامه |
| `1.4.2` | Tag؛ نام نسخه |
| `sha256:...` | Digest؛ شناسهٔ محتوایی برای اشارهٔ دقیق به Artifact |

گردش کار پیشنهادی: CI ایمیج را می‌سازد، با تگ Commit SHA به Harbor می‌فرستد، نتیجهٔ اسکن و سیاست‌ها بررسی می‌شود، سپس CD همان Digest تأییدشده را مستقر می‌کند. برای ارتقای نسخه از Stage به Prod، همان Artifact را منتقل کنید؛ دوباره Build نکنید.

## ۲. قابلیت‌هایی که در کار روزانه مهم‌اند

| قابلیت | کاربرد عملی |
|---|---|
| Private Project و RBAC | جداسازی تیم‌ها و محدودکردن Push/Pull |
| Vulnerability Scan | شناسایی CVE در ایمیج با اسکنرهایی مثل Trivy |
| Robot Account | حساب مخصوص CI/CD و Pull کلاستر |
| Replication | انتقال Artifact میان رجیستری‌ها طبق Rule |
| Proxy Cache | کش ایمیج upstream برای کاهش دانلود تکراری |
| Retention، Quota و GC | کنترل رشد فضای ذخیره‌سازی |
| Immutability | جلوگیری از تغییر یا حذف تگ‌های انتخاب‌شده |
| Webhook و API | اتصال رویدادها و عملیات Harbor به اتوماسیون |
| SBOM | فهرست اجزای نرم‌افزاری Artifact برای بررسی وابستگی‌ها |

Harbor برای OCI مناسب است؛ برای مخزن عمومی Maven یا npm، قابلیت موردنیاز را در ابزارهای مدیریت بسته بررسی کنید. [مدیریت Harbor](https://goharbor.io/docs/2.14.0/administration/)

## ۳. اجزای اصلی معماری

| جزء | مسئولیت |
|---|---|
| Portal | رابط وب |
| Core | API، احراز هویت و منطق مدیریت |
| Registry | سرویس Push/Pull و دسترسی به Blob و Manifest |
| Jobservice | اجرای کارهای پس‌زمینه مثل Replication |
| PostgreSQL | متادیتا، کاربران، پروژه‌ها و تنظیمات |
| Redis | کش و داده‌های موردنیاز صف/کارهای سرویس‌ها |
| Trivy Adapter | اتصال Harbor به اسکنر Trivy |
| Proxy / Ingress | ورودی HTTP(S) و هدایت درخواست‌ها |
| Filesystem / Object Storage | نگهداری محتوای Artifactها |

**دیتابیس با محل ذخیرهٔ لایه‌های ایمیج متفاوت است؛ بکاپ یکی به‌تنهایی کافی نیست.** برای HA به چند Replica، توزیع روی Nodeها، PostgreSQL و Redis در دسترس، Storage مشترک یا Object Storage و ورودی پایدار نیاز دارید؛ نصب روی Kubernetes به‌تنهایی HA ایجاد نمی‌کند. [معماری HA](https://goharbor.io/docs/2.14.0/install-config/harbor-ha-helm/)

## ۴. نصب و Config اولیه

### انتخاب روش

- **VM + Docker Compose:** مناسب آزمایشگاه و استقرار سادهٔ تک‌سرور؛ خرابی سرور باعث قطع سرویس می‌شود.
- **Kubernetes + Helm:** مناسب مدیریت با GitOps و طراحی HA؛ نیازمند تنظیم مستقل Storage و وابستگی‌هاست.

پیش از نصب: DNS، گواهی TLS معتبر، فضای پایدار، دسترسی شبکه و نسخه‌های سازگار پیش‌نیازها را آماده کنید. گواهی باید نام دامنه را در SAN داشته باشد؛ CA خصوصی باید برای Runner و Runtime نودهای کلاستر مورد اعتماد باشد.

### نمونهٔ نصب روی VM

Installer نسخهٔ انتخابی را از [Releaseهای رسمی](https://github.com/goharbor/harbor/releases) دریافت و بررسی کنید. داخل پوشهٔ استخراج‌شده:

```bash
cp harbor.yml.tmpl harbor.yml
```

**فقط این فیلدها را در Template همان نسخه ویرایش کنید؛ این قطعه جایگزین کل فایل نیست.** مقدارهای `CHANGE_ME` و مسیر گواهی نمونه‌اند.

```yaml
hostname: harbor.example.com
https:
  port: 443
  certificate: /opt/harbor/certs/fullchain.pem
  private_key: /opt/harbor/certs/privkey.pem
harbor_admin_password: "CHANGE_ME_STRONG_ADMIN_PASSWORD"
database:
  password: "CHANGE_ME_STRONG_DB_PASSWORD"
data_volume: /data
```

برای نمونهٔ HTTPS مستقیم، بخش `http` را در Template کامنت کنید. سپس:

```bash
sudo ./install.sh --with-trivy
docker compose ps
```

در `https://harbor.example.com` وارد شوید و پروژهٔ خصوصی `payments` بسازید. بدون `--with-trivy`، Installer به‌صورت پیش‌فرض Trivy را نصب نمی‌کند. [نصب رسمی](https://goharbor.io/docs/2.14.0/install-config/run-installer-script/)

| تنظیم زیرساختی | نکته |
|---|---|
| `hostname` | دامنهٔ قابل دسترسی کاربران و Runnerها |
| `external_url` | آدرس عمومی هنگام استفاده از Reverse Proxy |
| `data_volume` / `storage_service` | دیسک پایدار یا Backend ذخیره‌سازی |
| `harbor_admin_password` | فقط رمز اولیه؛ تغییر فایل بعد از نصب رمز را عوض نمی‌کند |
| `external_database` / `external_redis` | اتصال سرویس‌های بیرونی |
| `proxy` / `no_proxy` | دسترسی خروجی و استثناهای داخلی |
| `metric` / `log` | متریک و ثبت لاگ |
| `internal_tls` | رمزنگاری ارتباط داخلی اجزا، مستقل از TLS ورودی |

تغییر فایل به‌تنهایی تنظیمات سرویس در حال اجرا را اعمال نمی‌کند؛ روند Reconfigure نسخهٔ نصب‌شده را دنبال کنید. فایل‌های دارای رمز را وارد Git نکنید. [تنظیمات harbor.yml](https://goharbor.io/docs/2.14.0/install-config/configure-yml-file/)

### مسیر نصب روی Kubernetes

```bash
helm repo add harbor https://helm.goharbor.io
helm repo update
helm search repo harbor/harbor --versions
# CHART_VERSION را با نسخهٔ سازگار انتخاب‌شده تنظیم کنید.
helm show values harbor/harbor --version "$CHART_VERSION" > values.yaml
# پس از ویرایش values.yaml:
helm upgrade --install harbor harbor/harbor \
  --namespace harbor --create-namespace \
  --version "$CHART_VERSION" -f values.yaml
```

فیلدهای مهم: `externalURL`، `expose.ingress.hosts.core`، `expose.tls`، `persistence`، `database`، `redis` و `trivy.enabled`. نسخهٔ Chart را Pin کنید؛ شمارهٔ Chart لزوماً با نسخهٔ Harbor یکی نیست. پیش از اجرا، TLS Secret، StorageClass و رمزها را مطابق Chart آماده کنید. [Chart رسمی](https://github.com/goharbor/harbor-helm)

## ۵. بخش Settings و Administration

سه سطح تنظیم را جدا بدانید: **زیرساخت در harbor.yml یا Helm values؛ تنظیمات سراسری در Administration؛ سیاست هر پروژه داخل همان Project.**

### تنظیمات سراسری

| بخش یا گزینه | کارکرد و نکته |
|---|---|
| Configuration → Authentication | انتخاب Database، LDAP/AD یا OIDC |
| Configuration → System Settings | تنظیم رفتار عمومی رجیستری |
| Project Creation | اجازهٔ ساخت پروژه؛ در سازمان معمولاً Admin Only |
| Read Only | Pull مجاز می‌ماند؛ Push و حذف متوقف می‌شوند |
| Retain image last pull time on scanning | مانع تغییر زمان آخرین Pull توسط اسکن؛ مهم برای Retention مبتنی بر Pull |
| Banner | نمایش اطلاعیهٔ عمومی در رابط وب |

[تنظیمات عمومی](https://goharbor.io/docs/2.14.0/administration/general-settings/)

**Authentication:** برای LDAP، آدرس سرور، Base DN، حساب جست‌وجو و فیلتر کاربران؛ برای OIDC، آدرس Provider، Client ID/Secret و Redirect URI را تنظیم کنید. پیش از ساخت کاربران محلی دربارهٔ روش ورود تصمیم بگیرید؛ ساخت آن‌ها می‌تواند تغییر Auth Mode را محدود کند. [روش‌های احراز هویت](https://goharbor.io/docs/2.14.0/administration/configure-authentication/)

برای کاربر انسانی OIDC، ورود Docker معمولاً با **CLI Secret** پروفایل Harbor انجام می‌شود، نه رمز SSO؛ برای Pipeline از Robot استفاده کنید. [OIDC و CLI](https://goharbor.io/docs/2.14.0/administration/configure-authentication/oidc-auth/)

### سایر بخش‌های مدیریتی

| بخش | کاربرد |
|---|---|
| Users / Groups | مدیریت هویت‌ها و دسترسی مدیریتی |
| Registries | تعریف Endpoint مقصد یا upstream و اطلاعات اتصال |
| Replications | Rule انتقال با فیلتر، جهت و Trigger |
| Interrogation Services / Scanners | ثبت اسکنر و انتخاب اسکنر پیش‌فرض |
| Robot Accounts | حساب سیستمی برای اتوماسیون چند پروژه |
| Project Quotas | سقف فضای پروژه‌ها |
| Clean Up | Garbage Collection و پاک‌سازی لاگ‌ها |
| Audit Logs | پیگیری عملیات کاربران |
| Job Service Dashboard | مشاهدهٔ وضعیت و خطای کارهای پس‌زمینه |
| Security Hub | نمای تجمیعی وضعیت امنیتی |

این‌ها الزاماً تب‌های یک صفحهٔ Settings نیستند؛ بخش‌های مجاور در منوی مدیریت‌اند. [مرجع Administration](https://goharbor.io/docs/2.14.0/administration/)

### تنظیمات داخل Project

| بخش | کاربرد عملی |
|---|---|
| Configuration → Public | اجازهٔ Pull عمومی؛ برای خروجی خصوصی خاموش بماند |
| Automatically scan images on push | اسکن خودکار پس از Push |
| Prevent vulnerable images from running | مسدودکردن Pull براساس آستانهٔ شدت آسیب‌پذیری |
| CVE Allowlist | استثنا برای CVE مشخص؛ با دلیل و تاریخ انقضا مدیریت شود |
| Members | نقش‌هایی مثل Project Admin، Maintainer، Developer، Guest و Limited Guest |
| Robot Accounts | حساب محدود به پروژه |
| Tag Retention | انتخاب نسخه‌هایی که باید باقی بمانند |
| Tag Immutability | محافظت از تگ‌های Release در برابر تغییر/حذف |
| Webhooks | اعلان رویدادهایی مثل Push و پایان Scan |

**گزینهٔ جلوگیری از اجرای ایمیج آسیب‌پذیر، عملاً Pull را کنترل می‌کند؛ کانتینر در حال اجرا یا ایمیج Cacheشده روی Node را متوقف نمی‌کند.** برای کنترل استقرار در Kubernetes، Admission Policy هم لازم است. اسکن نیز تضمین نبود همهٔ آسیب‌پذیری‌ها نیست. [تنظیمات پروژه](https://goharbor.io/docs/2.14.0/working-with-projects/project-configuration/)

## ۶. اتصال CI/CD و Kubernetes

### Robot و Push

دو Robot جدا بسازید: CI با مجوز **Pull + Push**؛ کلاستر با **Pull**. Secret را در Secret Manager نگهداری و دوره‌ای Rotate کنید. Robot به UI وارد نمی‌شود؛ Push Permission باید همراه Pull باشد. [Robot Accounts](https://goharbor.io/docs/2.14.0/working-with-projects/project-configuration/create-robot-accounts/)

نمونهٔ Pipeline؛ متغیرها از CI تزریق می‌شوند و پروژه از قبل وجود دارد:

```bash
set -eu
: "${HARBOR_USER:?}" "${HARBOR_TOKEN:?}" "${CI_COMMIT_SHA:?}"
printf '%s' "$HARBOR_TOKEN" | docker login harbor.example.com \
  --username "$HARBOR_USER" --password-stdin
IMAGE="harbor.example.com/payments/api:${CI_COMMIT_SHA}"
docker build -t "$IMAGE" .
docker push "$IMAGE"
```

Push موفق به معنی Scan موفق نیست. Pipeline باید پایان اسکن را از API/رویداد بررسی کند و پیش از Deploy، آستانهٔ امنیتی را اعمال کند. از نمایش Secret با `set -x` جلوگیری کنید.

### Pull در Kubernetes

یک Secret نوع `kubernetes.io/dockerconfigjson` با حساب Pull-only و نام `harbor-pull` در **همان Namespace برنامه** ایجاد کنید؛ ترجیحاً با مدیریت Secret موجود سازمان. سپس در Pod Template:

```yaml
spec:
  template:
    spec:
      imagePullSecrets:
        - name: harbor-pull
      containers:
        - name: api
          image: harbor.example.com/payments/api:1.4.2
```

تگ نمونه باید قبلاً Push شده باشد؛ در Production ترجیحاً از Digest واقعی استفاده کنید. `imagePullSecrets` مشکل اعتبار گواهی را حل نمی‌کند؛ CA باید در Runtime نودها نیز معتبر باشد. [راهنمای Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

## ۷. نگهداری، کارایی و خطاهای رایج

- **Retention با GC فرق دارد:** Retention تعیین می‌کند چه نسخه‌هایی بمانند؛ GC، Blobهای بدون ارجاع را حذف می‌کند. ابتدا Dry Run بگیرید و نسخه‌های لازم برای Rollback را حفظ کنید. [Retention](https://goharbor.io/docs/2.14.0/working-with-projects/working-with-images/create-tag-retention-rules/)
- **حذف Tag الزاماً فضا آزاد نمی‌کند:** ممکن است لایه هنوز مرجع داشته باشد؛ آزادسازی واقعی وابسته به GC است. GC جدید می‌تواند هم‌زمان با استفاده از Harbor اجرا شود و برای دادهٔ تازه پنجرهٔ محافظتی دارد. [GC](https://goharbor.io/docs/2.14.0/administration/garbage-collection/)
- **تگ Release را Immutable کنید:** مثلاً الگوی `v*`؛ برای انتشار جدید تگ جدید بسازید. اثر آن بر حذف و Retention را در Ruleها لحاظ کنید. [Immutability](https://goharbor.io/docs/2.14.0/working-with-projects/working-with-images/create-tag-immutability-rules/)
- **Proxy Cache برای Pull تکراری مناسب است:** ابتدا Endpoint، سپس پروژهٔ Proxy Cache بسازید. Push مستقیم به آن مجاز نیست؛ اولین دریافت ایمیج کش‌نشده به upstream نیاز دارد. [Proxy Cache](https://goharbor.io/docs/2.14.0/administration/configure-proxy-cache/)
- **Replication انتقال سیاست‌محور است:** برای توزیع بین سایت‌ها مفید است؛ با کش در زمان Pull تفاوت دارد و جای بکاپ کامل تنظیمات و دیتابیس را نمی‌گیرد. [Replication](https://goharbor.io/docs/2.14.0/administration/configuring-replication/)

پیشنهاد عملیاتی: مصرف Storage، تأخیر/خطای Push و Pull، صف Jobها، وضعیت Scanner و انقضای TLS را مانیتور کنید. برای کارایی، اول شبکه و I/O ذخیره‌سازی را اندازه بگیرید؛ افزایش Worker بدون سنجش می‌تواند فشار را بیشتر کند. بکاپ هماهنگ دیتابیس، Blobها، تنظیمات و Secretها بگیرید و Restore را آزمایش کنید. قبل از Upgrade، مسیر ارتقای پشتیبانی‌شده و Migration دیتابیس را بررسی کنید.

| علامت | بررسی اولیه |
|---|---|
| `unauthorized` / `denied` | Robot منقضی، مجوز پروژه، نام حساب، Read Only یا سیاست امنیتی |
| `x509: certificate signed by unknown authority` | CA در Runner/Runtime و زنجیرهٔ گواهی |
| `ImagePullBackOff` | رویداد Pod، نام/Namespace Secret، DNS، Tag و دسترسی شبکه |
| خطای `413` یا Timeout در Push | محدودیت اندازه/Timeout در Ingress یا Reverse Proxy |
| Push ناموفق با فضای ظاهراً خالی | Quota پروژه، فضای واقعی Backend، Immutability |
| Scan در حالت Pending / Error | Scanner، صف Job، دسترسی به دیتابیس آسیب‌پذیری و Proxy |
| فضای دیسک پس از حذف کم نشد | GC، Blobهای مشترک، Artifact بدون Tag و پنجرهٔ محافظتی |

## ۸. سؤالات مصاحبه با پاسخ کوتاه

**۱. Harbor چه تفاوتی با یک Registry ساده دارد؟**  
علاوه بر Push/Pull، مدیریت پروژه، RBAC، اسکن، سیاست نگهداری، Replication و رابط مدیریتی می‌دهد.

**۲. Project با Repository چه فرقی دارد؟**  
Project مرز دسترسی و سیاست‌هاست؛ Repository داخل آن نسخه‌های یک Artifact را گروه‌بندی می‌کند.

**۳. Tag با Digest چه فرقی دارد؟**  
Tag نام قابل‌تغییر است مگر Immutable شود؛ Digest به محتوای دقیق اشاره می‌کند و برای استقرار تکرارپذیر مناسب‌تر است.

**۴. چرا در CI از حساب Admin استفاده نمی‌کنیم؟**  
Robot با کمترین مجوز، دامنهٔ اثر افشای Secret را محدود می‌کند و مستقل از حساب افراد است.

**۵. Scan on Push برای امن‌بودن Deploy کافی است؟**  
خیر؛ اسکن غیرهم‌زمان است. باید نتیجه و تازگی آن، آستانهٔ CVE و سیاست Admission بررسی شود.

**۶. آیا Harbor کانتینر آسیب‌پذیر در حال اجرا را متوقف می‌کند؟**  
خیر؛ سیاست رجیستری جلوی Pull را می‌گیرد. کنترل Runtime و Admission وظیفهٔ ابزارهای دیگر است.

**۷. Retention، Immutability و GC چه تفاوتی دارند؟**  
اولی انتخاب نسخه‌های ماندگار، دومی جلوگیری از تغییر/حذف تگ‌های مشخص و سومی آزادسازی Blobهای بدون مرجع است.

**۸. چرا با حذف ایمیج فضا آزاد نشد؟**  
ممکن است لایه‌ها مشترک باشند یا هنوز GC اجرا نشده باشد؛ گزارش Dry Run و مراجع باقی‌مانده را بررسی می‌کنم.

**۹. Replication با Proxy Cache چه تفاوتی دارد؟**  
Replication انتقال طبق Rule است؛ Proxy Cache هنگام Pull محتوا را از upstream دریافت و کش می‌کند.

**۱۰. برای HA چه چیزهایی لازم است؟**  
Replicaهای توزیع‌شده، ورودی پایدار، Storage قابل اشتراک و PostgreSQL/Redis در دسترس؛ وابستگی‌ها هم باید HA باشند.

**۱۱. از چه چیزهایی بکاپ می‌گیرید؟**  
دیتابیس، Blob Storage، تنظیمات، گواهی‌ها و Secretها به‌شکل سازگار؛ سپس بازیابی عملی را تست می‌کنم.

**۱۲. اگر Kubernetes نتواند Pull کند از کجا شروع می‌کنید؟**  
از Events در `kubectl describe pod`؛ سپس Secret همان Namespace، مجوز Robot، وجود Image، DNS و اعتماد TLS را بررسی می‌کنم.

**۱۳. برای محیط بدون اینترنت چه می‌کنید؟**  
Installer و ایمیج‌های لازم، Artifactهای برنامه و دیتابیس آسیب‌پذیری اسکنر را از مسیر کنترل‌شده وارد و به‌روزرسانی می‌کنم؛ کش خالی کافی نیست.

**۱۴. امضای Image با اسکن چه تفاوتی دارد؟**  
امضا برای بررسی اصالت و تمامیت است؛ اسکن برای کشف آسیب‌پذیری شناخته‌شده. هیچ‌کدام جای دیگری را نمی‌گیرد و امضا باید هنگام مصرف Verify شود.

**۱۵. اگر Harbor از دسترس خارج شود چه اتفاقی می‌افتد؟**  
کانتینرهای در حال اجرا معمولاً ادامه می‌دهند؛ Build/Push و استقرارهای نیازمند Pull مختل می‌شوند. اثر Cache به وضعیت Node و Pull Policy بستگی دارد.

**۱۶. برای جلوگیری از تغییر ناخواستهٔ Release چه می‌کنید؟**  
تگ یکتا، Immutability، مجوز محدود و Deploy با Digest؛ نسخه‌های Rollback را نیز در سیاست نگهداری حفظ می‌کنم.

## ۹. چک‌لیست شروع کار

- [ ] HTTPS و DNS معتبر؛ اعتماد CA در Runner و Nodeها
- [ ] پروژهٔ خصوصی و انتخاب زودهنگام روش Authentication
- [ ] Robot مجزای CI و کلاستر با کمترین مجوز
- [ ] اسکن خودکار و کنترل نتیجه پیش از Deploy
- [ ] تگ یکتا، Immutable Release و استقرار با Digest
- [ ] Quota، Retention و GC با بررسی Dry Run
- [ ] مانیتورینگ، بکاپ و آزمایش Restore

تمرین پیشنهادی: پروژهٔ آزمایشی بسازید، یک Image را با Robot Push کنید، اسکن را ببینید، با Kubernetes Pull کنید و در پایان رفتار Immutability و GC Dry Run را آزمایش کنید.

</div>
