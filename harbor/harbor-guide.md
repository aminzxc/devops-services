<div dir="rtl" lang="fa">

<h1 dir="rtl">راهنمای کاربردی <bdi dir="ltr">Harbor</bdi> برای مهندس <bdi dir="ltr">DevOps</bdi></h1>

<p dir="rtl">مبنای راهنما: مستندات <bdi dir="ltr">Harbor 2.14</bdi>؛ تاریخ بررسی: ۲۰۲۶/۰۹/۱۶. نام و جای بعضی گزینه‌ها با نسخه و سطح دسترسی فرق می‌کند. این نسخه، مبنای توضیح است و الزاماً آخرین نسخه نیست.</p>

<h2 dir="rtl">۱. <bdi dir="ltr">Harbor</bdi> چیست و کجا به کار می‌آید؟</h2>

<p dir="rtl"><strong><bdi dir="ltr">Harbor</bdi> یک رجیستری متن‌باز برای نگهداری و توزیع <bdi dir="ltr">Image</bdi> کانتینر و <bdi dir="ltr">OCI Artifact</bdi> است.</strong> کنترل دسترسی، اسکن آسیب‌پذیری، مدیریت پروژه و <bdi dir="ltr">Replication</bdi> را به رجیستری اضافه می‌کند. در <bdi dir="ltr">DevOps</bdi>، محل تحویل خروجی <bdi dir="ltr">Build</bdi> به سیستم استقرار است؛ خودش ابزار <bdi dir="ltr">Build</bdi> یا اجرای کانتینر نیست. <a href="https://goharbor.io/">معرفی رسمی</a></p>

<p dir="rtl">نمونهٔ مسیر یک <bdi dir="ltr">Image</bdi>:</p>

<pre dir="ltr" style="text-align: left;"><code class="language-text">harbor.example.com/payments/api:1.4.2
</code></pre>

<table dir="rtl">
<tr><th dir="rtl" align="right">بخش</th><th dir="rtl" align="right">معنی</th></tr>
<tr><td dir="rtl" align="right"><code dir="ltr">harbor.example.com</code></td><td dir="rtl" align="right">آدرس <bdi dir="ltr">Registry</bdi></td></tr>
<tr><td dir="rtl" align="right"><code dir="ltr">payments</code></td><td dir="rtl" align="right"><bdi dir="ltr">Project</bdi>؛ مرز دسترسی و سیاست‌ها</td></tr>
<tr><td dir="rtl" align="right"><code dir="ltr">api</code></td><td dir="rtl" align="right"><bdi dir="ltr">Repository</bdi>؛ مجموعهٔ نسخه‌های یک برنامه</td></tr>
<tr><td dir="rtl" align="right"><code dir="ltr">1.4.2</code></td><td dir="rtl" align="right"><bdi dir="ltr">Tag</bdi>؛ نام نسخه</td></tr>
<tr><td dir="rtl" align="right"><code dir="ltr">sha256:...</code></td><td dir="rtl" align="right"><bdi dir="ltr">Digest</bdi>؛ شناسهٔ محتوایی برای اشارهٔ دقیق به <bdi dir="ltr">Artifact</bdi></td></tr>
</table>

<p dir="rtl">گردش کار پیشنهادی: <bdi dir="ltr">CI</bdi> ایمیج را می‌سازد، با تگ <bdi dir="ltr">Commit SHA</bdi> به <bdi dir="ltr">Harbor</bdi> می‌فرستد، نتیجهٔ اسکن و سیاست‌ها بررسی می‌شود، سپس <bdi dir="ltr">CD</bdi> همان <bdi dir="ltr">Digest</bdi> تأییدشده را مستقر می‌کند. برای ارتقای نسخه از <bdi dir="ltr">Stage</bdi> به <bdi dir="ltr">Prod</bdi>، همان <bdi dir="ltr">Artifact</bdi> را منتقل کنید؛ دوباره <bdi dir="ltr">Build</bdi> نکنید.</p>

<h2 dir="rtl">۲. قابلیت‌هایی که در کار روزانه مهم‌اند</h2>

<table dir="rtl">
<tr><th dir="rtl" align="right">قابلیت</th><th dir="rtl" align="right">کاربرد عملی</th></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Private Project</bdi> و <bdi dir="ltr">RBAC</bdi></td><td dir="rtl" align="right">جداسازی تیم‌ها و محدودکردن <bdi dir="ltr">Push/Pull</bdi></td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Vulnerability Scan</bdi></td><td dir="rtl" align="right">شناسایی <bdi dir="ltr">CVE</bdi> در ایمیج با اسکنرهایی مثل <bdi dir="ltr">Trivy</bdi></td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Robot Account</bdi></td><td dir="rtl" align="right">حساب مخصوص <bdi dir="ltr">CI/CD</bdi> و <bdi dir="ltr">Pull</bdi> کلاستر</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Replication</bdi></td><td dir="rtl" align="right">انتقال <bdi dir="ltr">Artifact</bdi> میان رجیستری‌ها طبق <bdi dir="ltr">Rule</bdi></td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Proxy Cache</bdi></td><td dir="rtl" align="right">کش ایمیج <bdi dir="ltr">upstream</bdi> برای کاهش دانلود تکراری</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Retention</bdi>، <bdi dir="ltr">Quota</bdi> و <bdi dir="ltr">GC</bdi></td><td dir="rtl" align="right">کنترل رشد فضای ذخیره‌سازی</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Immutability</bdi></td><td dir="rtl" align="right">جلوگیری از تغییر یا حذف تگ‌های انتخاب‌شده</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Webhook</bdi> و <bdi dir="ltr">API</bdi></td><td dir="rtl" align="right">اتصال رویدادها و عملیات <bdi dir="ltr">Harbor</bdi> به اتوماسیون</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">SBOM</bdi></td><td dir="rtl" align="right">فهرست اجزای نرم‌افزاری <bdi dir="ltr">Artifact</bdi> برای بررسی وابستگی‌ها</td></tr>
</table>

<p dir="rtl"><bdi dir="ltr">Harbor</bdi> برای <bdi dir="ltr">OCI</bdi> مناسب است؛ برای مخزن عمومی <bdi dir="ltr">Maven</bdi> یا <bdi dir="ltr">npm</bdi>، قابلیت موردنیاز را در ابزارهای مدیریت بسته بررسی کنید. <a href="https://goharbor.io/docs/2.14.0/administration/">مدیریت <bdi dir="ltr">Harbor</bdi></a></p>

<h2 dir="rtl">۳. اجزای اصلی معماری</h2>

<table dir="rtl">
<tr><th dir="rtl" align="right">جزء</th><th dir="rtl" align="right">مسئولیت</th></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Portal</bdi></td><td dir="rtl" align="right">رابط وب</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Core</bdi></td><td dir="rtl" align="right"><bdi dir="ltr">API</bdi>، احراز هویت و منطق مدیریت</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Registry</bdi></td><td dir="rtl" align="right">سرویس <bdi dir="ltr">Push/Pull</bdi> و دسترسی به <bdi dir="ltr">Blob</bdi> و <bdi dir="ltr">Manifest</bdi></td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Jobservice</bdi></td><td dir="rtl" align="right">اجرای کارهای پس‌زمینه مثل <bdi dir="ltr">Replication</bdi></td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">PostgreSQL</bdi></td><td dir="rtl" align="right">متادیتا، کاربران، پروژه‌ها و تنظیمات</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Redis</bdi></td><td dir="rtl" align="right">کش و داده‌های موردنیاز صف/کارهای سرویس‌ها</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Trivy Adapter</bdi></td><td dir="rtl" align="right">اتصال <bdi dir="ltr">Harbor</bdi> به اسکنر <bdi dir="ltr">Trivy</bdi></td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Proxy / Ingress</bdi></td><td dir="rtl" align="right">ورودی <bdi dir="ltr">HTTP(S)</bdi> و هدایت درخواست‌ها</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Filesystem / Object Storage</bdi></td><td dir="rtl" align="right">نگهداری محتوای <bdi dir="ltr">Artifact</bdi>ها</td></tr>
</table>

<p dir="rtl"><strong>دیتابیس با محل ذخیرهٔ لایه‌های ایمیج متفاوت است؛ بکاپ یکی به‌تنهایی کافی نیست.</strong> برای <bdi dir="ltr">HA</bdi> به چند <bdi dir="ltr">Replica</bdi>، توزیع روی <bdi dir="ltr">Node</bdi>ها، <bdi dir="ltr">PostgreSQL</bdi> و <bdi dir="ltr">Redis</bdi> در دسترس، <bdi dir="ltr">Storage</bdi> مشترک یا <bdi dir="ltr">Object Storage</bdi> و ورودی پایدار نیاز دارید؛ نصب روی <bdi dir="ltr">Kubernetes</bdi> به‌تنهایی <bdi dir="ltr">HA</bdi> ایجاد نمی‌کند. <a href="https://goharbor.io/docs/2.14.0/install-config/harbor-ha-helm/">معماری <bdi dir="ltr">HA</bdi></a></p>

<h2 dir="rtl">۴. نصب و <bdi dir="ltr">Config</bdi> اولیه</h2>

<h3 dir="rtl">انتخاب روش</h3>

<ul dir="rtl">
<li dir="rtl"><strong><bdi dir="ltr">VM + Docker Compose</bdi>:</strong> مناسب آزمایشگاه و استقرار سادهٔ تک‌سرور؛ خرابی سرور باعث قطع سرویس می‌شود.</li>
<li dir="rtl"><strong><bdi dir="ltr">Kubernetes + Helm</bdi>:</strong> مناسب مدیریت با <bdi dir="ltr">GitOps</bdi> و طراحی <bdi dir="ltr">HA</bdi>؛ نیازمند تنظیم مستقل <bdi dir="ltr">Storage</bdi> و وابستگی‌هاست.</li>
</ul>

<p dir="rtl">پیش از نصب: <bdi dir="ltr">DNS</bdi>، گواهی <bdi dir="ltr">TLS</bdi> معتبر، فضای پایدار، دسترسی شبکه و نسخه‌های سازگار پیش‌نیازها را آماده کنید. گواهی باید نام دامنه را در <bdi dir="ltr">SAN</bdi> داشته باشد؛ <bdi dir="ltr">CA</bdi> خصوصی باید برای <bdi dir="ltr">Runner</bdi> و <bdi dir="ltr">Runtime</bdi> نودهای کلاستر مورد اعتماد باشد.</p>

<h3 dir="rtl">نمونهٔ نصب روی <bdi dir="ltr">VM</bdi></h3>

<p dir="rtl"><bdi dir="ltr">Installer</bdi> نسخهٔ انتخابی را از <a href="https://github.com/goharbor/harbor/releases"><bdi dir="ltr">Release</bdi>های رسمی</a> دریافت و بررسی کنید. داخل پوشهٔ استخراج‌شده:</p>

<pre dir="ltr" style="text-align: left;"><code class="language-bash">cp harbor.yml.tmpl harbor.yml
</code></pre>

<p dir="rtl"><strong>فقط این فیلدها را در <bdi dir="ltr">Template</bdi> همان نسخه ویرایش کنید؛ این قطعه جایگزین کل فایل نیست.</strong> مقدارهای <code dir="ltr">CHANGE_ME</code> و مسیر گواهی نمونه‌اند.</p>

<pre dir="ltr" style="text-align: left;"><code class="language-yaml">hostname: harbor.example.com
https:
  port: 443
  certificate: /opt/harbor/certs/fullchain.pem
  private_key: /opt/harbor/certs/privkey.pem
harbor_admin_password: &quot;CHANGE_ME_STRONG_ADMIN_PASSWORD&quot;
database:
  password: &quot;CHANGE_ME_STRONG_DB_PASSWORD&quot;
data_volume: /data
</code></pre>

<p dir="rtl">برای نمونهٔ <bdi dir="ltr">HTTPS</bdi> مستقیم، بخش <code dir="ltr">http</code> را در <bdi dir="ltr">Template</bdi> کامنت کنید. سپس:</p>

<pre dir="ltr" style="text-align: left;"><code class="language-bash">sudo ./install.sh --with-trivy
docker compose ps
</code></pre>

<p dir="rtl">در <code dir="ltr">https://harbor.example.com</code> وارد شوید و پروژهٔ خصوصی <code dir="ltr">payments</code> بسازید. بدون <code dir="ltr">--with-trivy</code>، <bdi dir="ltr">Installer</bdi> به‌صورت پیش‌فرض <bdi dir="ltr">Trivy</bdi> را نصب نمی‌کند. <a href="https://goharbor.io/docs/2.14.0/install-config/run-installer-script/">نصب رسمی</a></p>

<table dir="rtl">
<tr><th dir="rtl" align="right">تنظیم زیرساختی</th><th dir="rtl" align="right">نکته</th></tr>
<tr><td dir="rtl" align="right"><code dir="ltr">hostname</code></td><td dir="rtl" align="right">دامنهٔ قابل دسترسی کاربران و <bdi dir="ltr">Runner</bdi>ها</td></tr>
<tr><td dir="rtl" align="right"><code dir="ltr">external_url</code></td><td dir="rtl" align="right">آدرس عمومی هنگام استفاده از <bdi dir="ltr">Reverse Proxy</bdi></td></tr>
<tr><td dir="rtl" align="right"><code dir="ltr">data_volume</code> / <code dir="ltr">storage_service</code></td><td dir="rtl" align="right">دیسک پایدار یا <bdi dir="ltr">Backend</bdi> ذخیره‌سازی</td></tr>
<tr><td dir="rtl" align="right"><code dir="ltr">harbor_admin_password</code></td><td dir="rtl" align="right">فقط رمز اولیه؛ تغییر فایل بعد از نصب رمز را عوض نمی‌کند</td></tr>
<tr><td dir="rtl" align="right"><code dir="ltr">external_database</code> / <code dir="ltr">external_redis</code></td><td dir="rtl" align="right">اتصال سرویس‌های بیرونی</td></tr>
<tr><td dir="rtl" align="right"><code dir="ltr">proxy</code> / <code dir="ltr">no_proxy</code></td><td dir="rtl" align="right">دسترسی خروجی و استثناهای داخلی</td></tr>
<tr><td dir="rtl" align="right"><code dir="ltr">metric</code> / <code dir="ltr">log</code></td><td dir="rtl" align="right">متریک و ثبت لاگ</td></tr>
<tr><td dir="rtl" align="right"><code dir="ltr">internal_tls</code></td><td dir="rtl" align="right">رمزنگاری ارتباط داخلی اجزا، مستقل از <bdi dir="ltr">TLS</bdi> ورودی</td></tr>
</table>

<p dir="rtl">تغییر فایل به‌تنهایی تنظیمات سرویس در حال اجرا را اعمال نمی‌کند؛ روند <bdi dir="ltr">Reconfigure</bdi> نسخهٔ نصب‌شده را دنبال کنید. فایل‌های دارای رمز را وارد <bdi dir="ltr">Git</bdi> نکنید. <a href="https://goharbor.io/docs/2.14.0/install-config/configure-yml-file/">تنظیمات <bdi dir="ltr">harbor.yml</bdi></a></p>

<h3 dir="rtl">مسیر نصب روی <bdi dir="ltr">Kubernetes</bdi></h3>

<pre dir="ltr" style="text-align: left;"><code class="language-bash">helm repo add harbor https://helm.goharbor.io
helm repo update
helm search repo harbor/harbor --versions
# CHART_VERSION را با نسخهٔ سازگار انتخاب‌شده تنظیم کنید.
helm show values harbor/harbor --version &quot;$CHART_VERSION&quot; &gt; values.yaml
# پس از ویرایش values.yaml:
helm upgrade --install harbor harbor/harbor \
  --namespace harbor --create-namespace \
  --version &quot;$CHART_VERSION&quot; -f values.yaml
</code></pre>

<p dir="rtl">فیلدهای مهم: <code dir="ltr">externalURL</code>، <code dir="ltr">expose.ingress.hosts.core</code>، <code dir="ltr">expose.tls</code>، <code dir="ltr">persistence</code>، <code dir="ltr">database</code>، <code dir="ltr">redis</code> و <code dir="ltr">trivy.enabled</code>. نسخهٔ <bdi dir="ltr">Chart</bdi> را <bdi dir="ltr">Pin</bdi> کنید؛ شمارهٔ <bdi dir="ltr">Chart</bdi> لزوماً با نسخهٔ <bdi dir="ltr">Harbor</bdi> یکی نیست. پیش از اجرا، <bdi dir="ltr">TLS Secret</bdi>، <bdi dir="ltr">StorageClass</bdi> و رمزها را مطابق <bdi dir="ltr">Chart</bdi> آماده کنید. <a href="https://github.com/goharbor/harbor-helm"><bdi dir="ltr">Chart</bdi> رسمی</a></p>

<h2 dir="rtl">۵. بخش <bdi dir="ltr">Settings</bdi> و <bdi dir="ltr">Administration</bdi></h2>

<p dir="rtl">سه سطح تنظیم را جدا بدانید: <strong>زیرساخت در <bdi dir="ltr">harbor.yml</bdi> یا <bdi dir="ltr">Helm values</bdi>؛ تنظیمات سراسری در <bdi dir="ltr">Administration</bdi>؛ سیاست هر پروژه داخل همان <bdi dir="ltr">Project</bdi>.</strong></p>

<h3 dir="rtl">تنظیمات سراسری</h3>

<table dir="rtl">
<tr><th dir="rtl" align="right">بخش یا گزینه</th><th dir="rtl" align="right">کارکرد و نکته</th></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Configuration</bdi> → <bdi dir="ltr">Authentication</bdi></td><td dir="rtl" align="right">انتخاب <bdi dir="ltr">Database</bdi>، <bdi dir="ltr">LDAP/AD</bdi> یا <bdi dir="ltr">OIDC</bdi></td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Configuration</bdi> → <bdi dir="ltr">System Settings</bdi></td><td dir="rtl" align="right">تنظیم رفتار عمومی رجیستری</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Project Creation</bdi></td><td dir="rtl" align="right">اجازهٔ ساخت پروژه؛ در سازمان معمولاً <bdi dir="ltr">Admin Only</bdi></td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Read Only</bdi></td><td dir="rtl" align="right"><bdi dir="ltr">Pull</bdi> مجاز می‌ماند؛ <bdi dir="ltr">Push</bdi> و حذف متوقف می‌شوند</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Retain image last pull time on scanning</bdi></td><td dir="rtl" align="right">مانع تغییر زمان آخرین <bdi dir="ltr">Pull</bdi> توسط اسکن؛ مهم برای <bdi dir="ltr">Retention</bdi> مبتنی بر <bdi dir="ltr">Pull</bdi></td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Banner</bdi></td><td dir="rtl" align="right">نمایش اطلاعیهٔ عمومی در رابط وب</td></tr>
</table>

<p dir="rtl"><a href="https://goharbor.io/docs/2.14.0/administration/general-settings/">تنظیمات عمومی</a></p>

<p dir="rtl"><strong><bdi dir="ltr">Authentication</bdi>:</strong> برای <bdi dir="ltr">LDAP</bdi>، آدرس سرور، <bdi dir="ltr">Base DN</bdi>، حساب جست‌وجو و فیلتر کاربران؛ برای <bdi dir="ltr">OIDC</bdi>، آدرس <bdi dir="ltr">Provider</bdi>، <bdi dir="ltr">Client ID/Secret</bdi> و <bdi dir="ltr">Redirect URI</bdi> را تنظیم کنید. پیش از ساخت کاربران محلی دربارهٔ روش ورود تصمیم بگیرید؛ ساخت آن‌ها می‌تواند تغییر <bdi dir="ltr">Auth Mode</bdi> را محدود کند. <a href="https://goharbor.io/docs/2.14.0/administration/configure-authentication/">روش‌های احراز هویت</a></p>

<p dir="rtl">برای کاربر انسانی <bdi dir="ltr">OIDC</bdi>، ورود <bdi dir="ltr">Docker</bdi> معمولاً با <strong><bdi dir="ltr">CLI Secret</bdi></strong> پروفایل <bdi dir="ltr">Harbor</bdi> انجام می‌شود، نه رمز <bdi dir="ltr">SSO</bdi>؛ برای <bdi dir="ltr">Pipeline</bdi> از <bdi dir="ltr">Robot</bdi> استفاده کنید. <a href="https://goharbor.io/docs/2.14.0/administration/configure-authentication/oidc-auth/"><bdi dir="ltr">OIDC</bdi> و <bdi dir="ltr">CLI</bdi></a></p>

<h3 dir="rtl">سایر بخش‌های مدیریتی</h3>

<table dir="rtl">
<tr><th dir="rtl" align="right">بخش</th><th dir="rtl" align="right">کاربرد</th></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Users / Groups</bdi></td><td dir="rtl" align="right">مدیریت هویت‌ها و دسترسی مدیریتی</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Registries</bdi></td><td dir="rtl" align="right">تعریف <bdi dir="ltr">Endpoint</bdi> مقصد یا <bdi dir="ltr">upstream</bdi> و اطلاعات اتصال</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Replications</bdi></td><td dir="rtl" align="right"><bdi dir="ltr">Rule</bdi> انتقال با فیلتر، جهت و <bdi dir="ltr">Trigger</bdi></td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Interrogation Services / Scanners</bdi></td><td dir="rtl" align="right">ثبت اسکنر و انتخاب اسکنر پیش‌فرض</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Robot Accounts</bdi></td><td dir="rtl" align="right">حساب سیستمی برای اتوماسیون چند پروژه</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Project Quotas</bdi></td><td dir="rtl" align="right">سقف فضای پروژه‌ها</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Clean Up</bdi></td><td dir="rtl" align="right"><bdi dir="ltr">Garbage Collection</bdi> و پاک‌سازی لاگ‌ها</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Audit Logs</bdi></td><td dir="rtl" align="right">پیگیری عملیات کاربران</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Job Service Dashboard</bdi></td><td dir="rtl" align="right">مشاهدهٔ وضعیت و خطای کارهای پس‌زمینه</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Security Hub</bdi></td><td dir="rtl" align="right">نمای تجمیعی وضعیت امنیتی</td></tr>
</table>

<p dir="rtl">این‌ها الزاماً تب‌های یک صفحهٔ <bdi dir="ltr">Settings</bdi> نیستند؛ بخش‌های مجاور در منوی مدیریت‌اند. <a href="https://goharbor.io/docs/2.14.0/administration/">مرجع <bdi dir="ltr">Administration</bdi></a></p>

<h3 dir="rtl">تنظیمات داخل <bdi dir="ltr">Project</bdi></h3>

<table dir="rtl">
<tr><th dir="rtl" align="right">بخش</th><th dir="rtl" align="right">کاربرد عملی</th></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Configuration</bdi> → <bdi dir="ltr">Public</bdi></td><td dir="rtl" align="right">اجازهٔ <bdi dir="ltr">Pull</bdi> عمومی؛ برای خروجی خصوصی خاموش بماند</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Automatically scan images on push</bdi></td><td dir="rtl" align="right">اسکن خودکار پس از <bdi dir="ltr">Push</bdi></td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Prevent vulnerable images from running</bdi></td><td dir="rtl" align="right">مسدودکردن <bdi dir="ltr">Pull</bdi> براساس آستانهٔ شدت آسیب‌پذیری</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">CVE Allowlist</bdi></td><td dir="rtl" align="right">استثنا برای <bdi dir="ltr">CVE</bdi> مشخص؛ با دلیل و تاریخ انقضا مدیریت شود</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Members</bdi></td><td dir="rtl" align="right">نقش‌هایی مثل <bdi dir="ltr">Project Admin</bdi>، <bdi dir="ltr">Maintainer</bdi>، <bdi dir="ltr">Developer</bdi>، <bdi dir="ltr">Guest</bdi> و <bdi dir="ltr">Limited Guest</bdi></td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Robot Accounts</bdi></td><td dir="rtl" align="right">حساب محدود به پروژه</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Tag Retention</bdi></td><td dir="rtl" align="right">انتخاب نسخه‌هایی که باید باقی بمانند</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Tag Immutability</bdi></td><td dir="rtl" align="right">محافظت از تگ‌های <bdi dir="ltr">Release</bdi> در برابر تغییر/حذف</td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Webhooks</bdi></td><td dir="rtl" align="right">اعلان رویدادهایی مثل <bdi dir="ltr">Push</bdi> و پایان <bdi dir="ltr">Scan</bdi></td></tr>
</table>

<p dir="rtl"><strong>گزینهٔ جلوگیری از اجرای ایمیج آسیب‌پذیر، عملاً <bdi dir="ltr">Pull</bdi> را کنترل می‌کند؛ کانتینر در حال اجرا یا ایمیج <bdi dir="ltr">Cache</bdi>شده روی <bdi dir="ltr">Node</bdi> را متوقف نمی‌کند.</strong> برای کنترل استقرار در <bdi dir="ltr">Kubernetes</bdi>، <bdi dir="ltr">Admission Policy</bdi> هم لازم است. اسکن نیز تضمین نبود همهٔ آسیب‌پذیری‌ها نیست. <a href="https://goharbor.io/docs/2.14.0/working-with-projects/project-configuration/">تنظیمات پروژه</a></p>

<h2 dir="rtl">۶. اتصال <bdi dir="ltr">CI/CD</bdi> و <bdi dir="ltr">Kubernetes</bdi></h2>

<h3 dir="rtl"><bdi dir="ltr">Robot</bdi> و <bdi dir="ltr">Push</bdi></h3>

<p dir="rtl">دو <bdi dir="ltr">Robot</bdi> جدا بسازید: <bdi dir="ltr">CI</bdi> با مجوز <strong><bdi dir="ltr">Pull + Push</bdi></strong>؛ کلاستر با <strong><bdi dir="ltr">Pull</bdi></strong>. <bdi dir="ltr">Secret</bdi> را در <bdi dir="ltr">Secret Manager</bdi> نگهداری و دوره‌ای <bdi dir="ltr">Rotate</bdi> کنید. <bdi dir="ltr">Robot</bdi> به <bdi dir="ltr">UI</bdi> وارد نمی‌شود؛ <bdi dir="ltr">Push Permission</bdi> باید همراه <bdi dir="ltr">Pull</bdi> باشد. <a href="https://goharbor.io/docs/2.14.0/working-with-projects/project-configuration/create-robot-accounts/"><bdi dir="ltr">Robot Accounts</bdi></a></p>

<p dir="rtl">نمونهٔ <bdi dir="ltr">Pipeline</bdi>؛ متغیرها از <bdi dir="ltr">CI</bdi> تزریق می‌شوند و پروژه از قبل وجود دارد:</p>

<pre dir="ltr" style="text-align: left;"><code class="language-bash">set -eu
: &quot;${HARBOR_USER:?}&quot; &quot;${HARBOR_TOKEN:?}&quot; &quot;${CI_COMMIT_SHA:?}&quot;
printf &#x27;%s&#x27; &quot;$HARBOR_TOKEN&quot; | docker login harbor.example.com \
  --username &quot;$HARBOR_USER&quot; --password-stdin
IMAGE=&quot;harbor.example.com/payments/api:${CI_COMMIT_SHA}&quot;
docker build -t &quot;$IMAGE&quot; .
docker push &quot;$IMAGE&quot;
</code></pre>

<p dir="rtl"><bdi dir="ltr">Push</bdi> موفق به معنی <bdi dir="ltr">Scan</bdi> موفق نیست. <bdi dir="ltr">Pipeline</bdi> باید پایان اسکن را از <bdi dir="ltr">API</bdi>/رویداد بررسی کند و پیش از <bdi dir="ltr">Deploy</bdi>، آستانهٔ امنیتی را اعمال کند. از نمایش <bdi dir="ltr">Secret</bdi> با <code dir="ltr">set -x</code> جلوگیری کنید.</p>

<h3 dir="rtl"><bdi dir="ltr">Pull</bdi> در <bdi dir="ltr">Kubernetes</bdi></h3>

<p dir="rtl">یک <bdi dir="ltr">Secret</bdi> نوع <code dir="ltr">kubernetes.io/dockerconfigjson</code> با حساب <bdi dir="ltr">Pull-only</bdi> و نام <code dir="ltr">harbor-pull</code> در <strong>همان <bdi dir="ltr">Namespace</bdi> برنامه</strong> ایجاد کنید؛ ترجیحاً با مدیریت <bdi dir="ltr">Secret</bdi> موجود سازمان. سپس در <bdi dir="ltr">Pod Template</bdi>:</p>

<pre dir="ltr" style="text-align: left;"><code class="language-yaml">spec:
  template:
    spec:
      imagePullSecrets:
        - name: harbor-pull
      containers:
        - name: api
          image: harbor.example.com/payments/api:1.4.2
</code></pre>

<p dir="rtl">تگ نمونه باید قبلاً <bdi dir="ltr">Push</bdi> شده باشد؛ در <bdi dir="ltr">Production</bdi> ترجیحاً از <bdi dir="ltr">Digest</bdi> واقعی استفاده کنید. <code dir="ltr">imagePullSecrets</code> مشکل اعتبار گواهی را حل نمی‌کند؛ <bdi dir="ltr">CA</bdi> باید در <bdi dir="ltr">Runtime</bdi> نودها نیز معتبر باشد. <a href="https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/">راهنمای <bdi dir="ltr">Kubernetes</bdi></a></p>

<h2 dir="rtl">۷. نگهداری، کارایی و خطاهای رایج</h2>

<ul dir="rtl">
<li dir="rtl"><strong><bdi dir="ltr">Retention</bdi> با <bdi dir="ltr">GC</bdi> فرق دارد:</strong> <bdi dir="ltr">Retention</bdi> تعیین می‌کند چه نسخه‌هایی بمانند؛ <bdi dir="ltr">GC</bdi>، <bdi dir="ltr">Blob</bdi>های بدون ارجاع را حذف می‌کند. ابتدا <bdi dir="ltr">Dry Run</bdi> بگیرید و نسخه‌های لازم برای <bdi dir="ltr">Rollback</bdi> را حفظ کنید. <a href="https://goharbor.io/docs/2.14.0/working-with-projects/working-with-images/create-tag-retention-rules/"><bdi dir="ltr">Retention</bdi></a></li>
<li dir="rtl"><strong>حذف <bdi dir="ltr">Tag</bdi> الزاماً فضا آزاد نمی‌کند:</strong> ممکن است لایه هنوز مرجع داشته باشد؛ آزادسازی واقعی وابسته به <bdi dir="ltr">GC</bdi> است. <bdi dir="ltr">GC</bdi> جدید می‌تواند هم‌زمان با استفاده از <bdi dir="ltr">Harbor</bdi> اجرا شود و برای دادهٔ تازه پنجرهٔ محافظتی دارد. <a href="https://goharbor.io/docs/2.14.0/administration/garbage-collection/"><bdi dir="ltr">GC</bdi></a></li>
<li dir="rtl"><strong>تگ <bdi dir="ltr">Release</bdi> را <bdi dir="ltr">Immutable</bdi> کنید:</strong> مثلاً الگوی <code dir="ltr">v*</code>؛ برای انتشار جدید تگ جدید بسازید. اثر آن بر حذف و <bdi dir="ltr">Retention</bdi> را در <bdi dir="ltr">Rule</bdi>ها لحاظ کنید. <a href="https://goharbor.io/docs/2.14.0/working-with-projects/working-with-images/create-tag-immutability-rules/"><bdi dir="ltr">Immutability</bdi></a></li>
<li dir="rtl"><strong><bdi dir="ltr">Proxy Cache</bdi> برای <bdi dir="ltr">Pull</bdi> تکراری مناسب است:</strong> ابتدا <bdi dir="ltr">Endpoint</bdi>، سپس پروژهٔ <bdi dir="ltr">Proxy Cache</bdi> بسازید. <bdi dir="ltr">Push</bdi> مستقیم به آن مجاز نیست؛ اولین دریافت ایمیج کش‌نشده به <bdi dir="ltr">upstream</bdi> نیاز دارد. <a href="https://goharbor.io/docs/2.14.0/administration/configure-proxy-cache/"><bdi dir="ltr">Proxy Cache</bdi></a></li>
<li dir="rtl"><strong><bdi dir="ltr">Replication</bdi> انتقال سیاست‌محور است:</strong> برای توزیع بین سایت‌ها مفید است؛ با کش در زمان <bdi dir="ltr">Pull</bdi> تفاوت دارد و جای بکاپ کامل تنظیمات و دیتابیس را نمی‌گیرد. <a href="https://goharbor.io/docs/2.14.0/administration/configuring-replication/"><bdi dir="ltr">Replication</bdi></a></li>
</ul>

<p dir="rtl">پیشنهاد عملیاتی: مصرف <bdi dir="ltr">Storage</bdi>، تأخیر/خطای <bdi dir="ltr">Push</bdi> و <bdi dir="ltr">Pull</bdi>، صف <bdi dir="ltr">Job</bdi>ها، وضعیت <bdi dir="ltr">Scanner</bdi> و انقضای <bdi dir="ltr">TLS</bdi> را مانیتور کنید. برای کارایی، اول شبکه و <bdi dir="ltr">I/O</bdi> ذخیره‌سازی را اندازه بگیرید؛ افزایش <bdi dir="ltr">Worker</bdi> بدون سنجش می‌تواند فشار را بیشتر کند. بکاپ هماهنگ دیتابیس، <bdi dir="ltr">Blob</bdi>ها، تنظیمات و <bdi dir="ltr">Secret</bdi>ها بگیرید و <bdi dir="ltr">Restore</bdi> را آزمایش کنید. قبل از <bdi dir="ltr">Upgrade</bdi>، مسیر ارتقای پشتیبانی‌شده و <bdi dir="ltr">Migration</bdi> دیتابیس را بررسی کنید.</p>

<table dir="rtl">
<tr><th dir="rtl" align="right">علامت</th><th dir="rtl" align="right">بررسی اولیه</th></tr>
<tr><td dir="rtl" align="right"><code dir="ltr">unauthorized</code> / <code dir="ltr">denied</code></td><td dir="rtl" align="right"><bdi dir="ltr">Robot</bdi> منقضی، مجوز پروژه، نام حساب، <bdi dir="ltr">Read Only</bdi> یا سیاست امنیتی</td></tr>
<tr><td dir="rtl" align="right"><code dir="ltr">x509: certificate signed by unknown authority</code></td><td dir="rtl" align="right"><bdi dir="ltr">CA</bdi> در <bdi dir="ltr">Runner/Runtime</bdi> و زنجیرهٔ گواهی</td></tr>
<tr><td dir="rtl" align="right"><code dir="ltr">ImagePullBackOff</code></td><td dir="rtl" align="right">رویداد <bdi dir="ltr">Pod</bdi>، نام/<bdi dir="ltr">Namespace Secret</bdi>، <bdi dir="ltr">DNS</bdi>، <bdi dir="ltr">Tag</bdi> و دسترسی شبکه</td></tr>
<tr><td dir="rtl" align="right">خطای <code dir="ltr">413</code> یا <bdi dir="ltr">Timeout</bdi> در <bdi dir="ltr">Push</bdi></td><td dir="rtl" align="right">محدودیت اندازه/<bdi dir="ltr">Timeout</bdi> در <bdi dir="ltr">Ingress</bdi> یا <bdi dir="ltr">Reverse Proxy</bdi></td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Push</bdi> ناموفق با فضای ظاهراً خالی</td><td dir="rtl" align="right"><bdi dir="ltr">Quota</bdi> پروژه، فضای واقعی <bdi dir="ltr">Backend</bdi>، <bdi dir="ltr">Immutability</bdi></td></tr>
<tr><td dir="rtl" align="right"><bdi dir="ltr">Scan</bdi> در حالت <bdi dir="ltr">Pending / Error</bdi></td><td dir="rtl" align="right"><bdi dir="ltr">Scanner</bdi>، صف <bdi dir="ltr">Job</bdi>، دسترسی به دیتابیس آسیب‌پذیری و <bdi dir="ltr">Proxy</bdi></td></tr>
<tr><td dir="rtl" align="right">فضای دیسک پس از حذف کم نشد</td><td dir="rtl" align="right"><bdi dir="ltr">GC</bdi>، <bdi dir="ltr">Blob</bdi>های مشترک، <bdi dir="ltr">Artifact</bdi> بدون <bdi dir="ltr">Tag</bdi> و پنجرهٔ محافظتی</td></tr>
</table>

<h2 dir="rtl">۸. سؤالات مصاحبه با پاسخ کوتاه</h2>

<p dir="rtl"><strong>۱. <bdi dir="ltr">Harbor</bdi> چه تفاوتی با یک <bdi dir="ltr">Registry</bdi> ساده دارد؟</strong><br>
علاوه بر <bdi dir="ltr">Push/Pull</bdi>، مدیریت پروژه، <bdi dir="ltr">RBAC</bdi>، اسکن، سیاست نگهداری، <bdi dir="ltr">Replication</bdi> و رابط مدیریتی می‌دهد.</p>

<p dir="rtl"><strong>۲. <bdi dir="ltr">Project</bdi> با <bdi dir="ltr">Repository</bdi> چه فرقی دارد؟</strong><br>
<bdi dir="ltr">Project</bdi> مرز دسترسی و سیاست‌هاست؛ <bdi dir="ltr">Repository</bdi> داخل آن نسخه‌های یک <bdi dir="ltr">Artifact</bdi> را گروه‌بندی می‌کند.</p>

<p dir="rtl"><strong>۳. <bdi dir="ltr">Tag</bdi> با <bdi dir="ltr">Digest</bdi> چه فرقی دارد؟</strong><br>
<bdi dir="ltr">Tag</bdi> نام قابل‌تغییر است مگر <bdi dir="ltr">Immutable</bdi> شود؛ <bdi dir="ltr">Digest</bdi> به محتوای دقیق اشاره می‌کند و برای استقرار تکرارپذیر مناسب‌تر است.</p>

<p dir="rtl"><strong>۴. چرا در <bdi dir="ltr">CI</bdi> از حساب <bdi dir="ltr">Admin</bdi> استفاده نمی‌کنیم؟</strong><br>
<bdi dir="ltr">Robot</bdi> با کمترین مجوز، دامنهٔ اثر افشای <bdi dir="ltr">Secret</bdi> را محدود می‌کند و مستقل از حساب افراد است.</p>

<p dir="rtl"><strong>۵. <bdi dir="ltr">Scan on Push</bdi> برای امن‌بودن <bdi dir="ltr">Deploy</bdi> کافی است؟</strong><br>
خیر؛ اسکن غیرهم‌زمان است. باید نتیجه و تازگی آن، آستانهٔ <bdi dir="ltr">CVE</bdi> و سیاست <bdi dir="ltr">Admission</bdi> بررسی شود.</p>

<p dir="rtl"><strong>۶. آیا <bdi dir="ltr">Harbor</bdi> کانتینر آسیب‌پذیر در حال اجرا را متوقف می‌کند؟</strong><br>
خیر؛ سیاست رجیستری جلوی <bdi dir="ltr">Pull</bdi> را می‌گیرد. کنترل <bdi dir="ltr">Runtime</bdi> و <bdi dir="ltr">Admission</bdi> وظیفهٔ ابزارهای دیگر است.</p>

<p dir="rtl"><strong>۷. <bdi dir="ltr">Retention</bdi>، <bdi dir="ltr">Immutability</bdi> و <bdi dir="ltr">GC</bdi> چه تفاوتی دارند؟</strong><br>
اولی انتخاب نسخه‌های ماندگار، دومی جلوگیری از تغییر/حذف تگ‌های مشخص و سومی آزادسازی <bdi dir="ltr">Blob</bdi>های بدون مرجع است.</p>

<p dir="rtl"><strong>۸. چرا با حذف ایمیج فضا آزاد نشد؟</strong><br>
ممکن است لایه‌ها مشترک باشند یا هنوز <bdi dir="ltr">GC</bdi> اجرا نشده باشد؛ گزارش <bdi dir="ltr">Dry Run</bdi> و مراجع باقی‌مانده را بررسی می‌کنم.</p>

<p dir="rtl"><strong>۹. <bdi dir="ltr">Replication</bdi> با <bdi dir="ltr">Proxy Cache</bdi> چه تفاوتی دارد؟</strong><br>
<bdi dir="ltr">Replication</bdi> انتقال طبق <bdi dir="ltr">Rule</bdi> است؛ <bdi dir="ltr">Proxy Cache</bdi> هنگام <bdi dir="ltr">Pull</bdi> محتوا را از <bdi dir="ltr">upstream</bdi> دریافت و کش می‌کند.</p>

<p dir="rtl"><strong>۱۰. برای <bdi dir="ltr">HA</bdi> چه چیزهایی لازم است؟</strong><br>
<bdi dir="ltr">Replica</bdi>های توزیع‌شده، ورودی پایدار، <bdi dir="ltr">Storage</bdi> قابل اشتراک و <bdi dir="ltr">PostgreSQL/Redis</bdi> در دسترس؛ وابستگی‌ها هم باید <bdi dir="ltr">HA</bdi> باشند.</p>

<p dir="rtl"><strong>۱۱. از چه چیزهایی بکاپ می‌گیرید؟</strong><br>
دیتابیس، <bdi dir="ltr">Blob Storage</bdi>، تنظیمات، گواهی‌ها و <bdi dir="ltr">Secret</bdi>ها به‌شکل سازگار؛ سپس بازیابی عملی را تست می‌کنم.</p>

<p dir="rtl"><strong>۱۲. اگر <bdi dir="ltr">Kubernetes</bdi> نتواند <bdi dir="ltr">Pull</bdi> کند از کجا شروع می‌کنید؟</strong><br>
از <bdi dir="ltr">Events</bdi> در <code dir="ltr">kubectl describe pod</code>؛ سپس <bdi dir="ltr">Secret</bdi> همان <bdi dir="ltr">Namespace</bdi>، مجوز <bdi dir="ltr">Robot</bdi>، وجود <bdi dir="ltr">Image</bdi>، <bdi dir="ltr">DNS</bdi> و اعتماد <bdi dir="ltr">TLS</bdi> را بررسی می‌کنم.</p>

<p dir="rtl"><strong>۱۳. برای محیط بدون اینترنت چه می‌کنید؟</strong><br>
<bdi dir="ltr">Installer</bdi> و ایمیج‌های لازم، <bdi dir="ltr">Artifact</bdi>های برنامه و دیتابیس آسیب‌پذیری اسکنر را از مسیر کنترل‌شده وارد و به‌روزرسانی می‌کنم؛ کش خالی کافی نیست.</p>

<p dir="rtl"><strong>۱۴. امضای <bdi dir="ltr">Image</bdi> با اسکن چه تفاوتی دارد؟</strong><br>
امضا برای بررسی اصالت و تمامیت است؛ اسکن برای کشف آسیب‌پذیری شناخته‌شده. هیچ‌کدام جای دیگری را نمی‌گیرد و امضا باید هنگام مصرف <bdi dir="ltr">Verify</bdi> شود.</p>

<p dir="rtl"><strong>۱۵. اگر <bdi dir="ltr">Harbor</bdi> از دسترس خارج شود چه اتفاقی می‌افتد؟</strong><br>
کانتینرهای در حال اجرا معمولاً ادامه می‌دهند؛ <bdi dir="ltr">Build/Push</bdi> و استقرارهای نیازمند <bdi dir="ltr">Pull</bdi> مختل می‌شوند. اثر <bdi dir="ltr">Cache</bdi> به وضعیت <bdi dir="ltr">Node</bdi> و <bdi dir="ltr">Pull Policy</bdi> بستگی دارد.</p>

<p dir="rtl"><strong>۱۶. برای جلوگیری از تغییر ناخواستهٔ <bdi dir="ltr">Release</bdi> چه می‌کنید؟</strong><br>
تگ یکتا، <bdi dir="ltr">Immutability</bdi>، مجوز محدود و <bdi dir="ltr">Deploy</bdi> با <bdi dir="ltr">Digest</bdi>؛ نسخه‌های <bdi dir="ltr">Rollback</bdi> را نیز در سیاست نگهداری حفظ می‌کنم.</p>

<h2 dir="rtl">۹. چک‌لیست شروع کار</h2>

<ul dir="rtl">
<li dir="rtl">☐ <bdi dir="ltr">HTTPS</bdi> و <bdi dir="ltr">DNS</bdi> معتبر؛ اعتماد <bdi dir="ltr">CA</bdi> در <bdi dir="ltr">Runner</bdi> و <bdi dir="ltr">Node</bdi>ها</li>
<li dir="rtl">☐ پروژهٔ خصوصی و انتخاب زودهنگام روش <bdi dir="ltr">Authentication</bdi></li>
<li dir="rtl">☐ <bdi dir="ltr">Robot</bdi> مجزای <bdi dir="ltr">CI</bdi> و کلاستر با کمترین مجوز</li>
<li dir="rtl">☐ اسکن خودکار و کنترل نتیجه پیش از <bdi dir="ltr">Deploy</bdi></li>
<li dir="rtl">☐ تگ یکتا، <bdi dir="ltr">Immutable Release</bdi> و استقرار با <bdi dir="ltr">Digest</bdi></li>
<li dir="rtl">☐ <bdi dir="ltr">Quota</bdi>، <bdi dir="ltr">Retention</bdi> و <bdi dir="ltr">GC</bdi> با بررسی <bdi dir="ltr">Dry Run</bdi></li>
<li dir="rtl">☐ مانیتورینگ، بکاپ و آزمایش <bdi dir="ltr">Restore</bdi></li>
</ul>

<p dir="rtl">تمرین پیشنهادی: پروژهٔ آزمایشی بسازید، یک <bdi dir="ltr">Image</bdi> را با <bdi dir="ltr">Robot Push</bdi> کنید، اسکن را ببینید، با <bdi dir="ltr">Kubernetes Pull</bdi> کنید و در پایان رفتار <bdi dir="ltr">Immutability</bdi> و <bdi dir="ltr">GC Dry Run</bdi> را آزمایش کنید.</p>

</div>
