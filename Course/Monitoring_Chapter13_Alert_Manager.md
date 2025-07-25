# آموزش پرومتئوس - قسمت سیزدهم هشدارها و Alertmanager 

در این آموزش، به صورت عمیق و جامع به موضوع **هشدارها (Alerting)** در پرومتئوس و نحوه استفاده از **Alertmanager** می‌پردازیم.
این راهنما از مفاهیم پایه شروع می‌کند، به مثال‌های عملی می‌رسد و در نهایت به بررسی عمیق Alertmanager و نحوه کار با آن می‌پردازد. 
هدف این است که تمام جنبه‌های مرتبط با هشدارها، از فلسفه و مفاهیم اولیه تا پیاده‌سازی و مدیریت، به زبانی ساده و قابل فهم توضیح داده شود.

---

## 1. فلسفه هشدارها در پرومتئوس

هشدارها در پرومتئوس یکی از اجزای کلیدی برای نظارت (Monitoring) و اطمینان از پایداری و سلامت سیستم‌ها هستند.
شاید اصلا مانیتورینگ برای این بوجود آمد تا اطلاعات مورد نظر برای بحث هشدار هارو بتواند تولید کند. یعنی اینقدر مسئله مهمی است.
و در واقع تمام کننده هدف مانیتورینگ میباشد. هشدار ها بنابر تفسیر و فیلتر های تعریف شده توسط تیم ما میتوانند معنا و مفهوم متفاوت داشته باشند.
برخی هشدار ها پیش گیری کننده هستند ، برخی اطلاع دهنده ، برخی برای اقدامات اورژانسی هستند و ....
یک از فلسفه های اصلی هشدارها این است که به تیم‌های عملیاتی اجازه دهند **قبل** از وقوع مشکلات بزرگ، از مسائل احتمالی آگاه شوند و اقدامات لازم را انجام دهند.

### هشدارها به ما کمک می‌کنند تا:

 - **مشکلات را زود تشخیص دهیم**:
     -  به جای اینکه منتظر خرابی سیستم باشیم، می‌توانیم با تنظیم آستانه‌های مشخص، مشکلات را در مراحل اولیه شناسایی کنیم.
 - **اولویت‌بندی کنیم**:
     - هشدارها بر اساس شدت (Severity) اولویت‌بندی می‌شوند تا تیم‌ها بدانند کدام مسائل فوریت بیشتری دارند.
 - **اتوماسیون را بهبود دهیم**:
     - هشدارها می‌توانند به سیستم‌های خودکار متصل شوند تا اقدامات اصلاحی یا پیشگیری کننده به صورت خودکار انجام شوند.

### مثال جذاب:
فرض کنید شما یک اپلیکیشن فروش آنلاین دارید.اگر نرخ خطای درخواست‌ها (Error Rate) به بیش از 5% برسد، می‌خواهید فوراً مطلع شوید تا تیم توسعه سریعاً مشکل را بررسی کند.
یا اگر سرور شما بیش از 80% از ظرفیت CPU را استفاده کند، هشداری دریافت کنید تا قبل از کند شدن سیستم، سرور جدیدی اضافه کنید.
این هشدارها مثل یک نگهبان هوشیار عمل می‌کنند که قبل از به صدا درآمدن زنگ خطر، شما را بیدار می‌کند!

---

## 2. مفاهیم پایه در هشدارها

### 2.1 هشدار چیست؟
هشدار (Alert) یک اعلان است که وقتی یک شرط خاص در داده‌های جمع‌آوری‌شده توسط پرومتئوس(یا هر سیستم مانیتورینگ دیگر) برقرار شود، فعال (Firing) می‌شود. این شرط‌ها در قالب **Prometheus Rules** تعریف می‌شوند.

### 2.2 اجزای اصلی هشدارها
 - بخش **Prometheus Rules**: قوانینی که شرایط هشدار را تعریف می‌کنند (مثلاً اگر CPU Usage بیش از 80% شد).
 - بخش **Alertmanager**: ابزاری که هشدارهای تولیدشده توسط پرومتئوس را مدیریت کرده و به مقصدهای مختلف (مانند Slack، Email، PagerDuty) ارسال می‌کند.
 - بخش **شدت (Severity)**: نشان‌دهنده میزان اهمیت هشدار است (مثل Critical، Warning، Info).
 - بخش **وضعیت‌های هشدار**:
   1. **مرحله Pending**: هشدار فعال شده اما هنوز به مدت زمان مشخصی (Duration) نرسیده تا به حالت Firing برود.
   2. **مرحله Firing**: هشدار به طور کامل فعال شده و به Alertmanager ارسال شده است.
   3. **مرحله Resolved**: هشدار برطرف شده و دیگر فعال نیست.

### 2.3 کی هشدار تولید می‌کند؟
شاید به اشتباه این تصور شود که alert manager هشدار هارا تولید میکند ولی در واقع alert manager فقط مدیریت بعد از تولید هشدار را به عهده دارد و پرومتئوس مسئول تولید هشدارها بر اساس قوانین تعریف‌شده (Prometheus Rules) است.این قوانین در قالب **Custom Resource Definitions (CRDs)** یا فایل‌های YAML تعریف می‌شوند و پرومتئوس داده‌های متریک را بررسی می‌کند تا ببیند آیا شرایط هشدار برقرار است یا خیر.

---

## 3. سیستم‌های مدیریت هشدار

مدیریت هشدارها در پرومتئوس توسط **Alertmanager** انجام می‌شود. 
این ابزار وظیفه دریافت هشدارها از پرومتئوس و مدیریت آن‌ها (مانند گروه‌بندی، سرکوب، ارسال به مقصدهای مختلف) را بر عهده دارد.

- <img width="100%" alt="Prometheus-and-Alertmanager-Architecture" src="https://github.com/user-attachments/assets/3a4cb6c4-c404-4711-9e82-55d0c14ae7db"/>


### 3.1 وظایف Alertmanager
**گروه‌بندی (Grouping)**:

- چی هست؟ هشدارهای مرتبط رو توی یه پیام جمع می‌کنه تا تعداد اعلان‌ها کم بشه و از شلوغی جلوگیری کنه.
- مثال ساده: فرض کن 10 سرور تو یه دقیقه CPUشون بالا میره.
- به جای 10 پیام جدا، Alertmanager یه پیام می‌فرسته که می‌گه: "CPU سرورهای 1    تا 10 بالاست!"
- چرا مهمه؟ این کار باعث می‌شه تیم شما غرق پیام‌های تکراری نشه.

**سرکوب (Inhibition)**:

- چی هست؟ اگر یه هشدار مهم‌تر فعال باشه، هشدارهای کم اهمیت‌تر رو خاموش می‌کنه تا تمرکز روی مشکل اصلی باشه.
- مثال ساده: اگه سرور کامل خاموش بشه (هشدار بحرانی)، Alertmanager هشدارهای "CPU بالاست" (کم اهمیت‌تر) رو سرکوب می‌کنه چون مشکل بزرگ‌تره.
- چرا مهمه؟ کمک می‌کنه فقط روی مسائل حیاتی تمرکز کنید.

**سکوت (Silencing)**:

- چی هست؟ می‌تونی هشدارها رو برای یه مدت مشخص خاموش کنی، مثلاً وقتی داری سیستم رو تعمیر یا به‌روزرسانی می‌کنی.
- مثال ساده: موقع آپدیت سرور، هشدارهای "سرور در دسترس نیست" رو برای 2 ساعت خاموش می‌کنی تا تیم اذیت نشه.
- چرا مهمه؟ از دریافت اعلان‌های غیرضروری در زمان‌های خاص جلوگیری می‌کنه.

**ارسال اعلان‌ها**:

- چی هست؟ Alertmanager هشدارها رو به جاهایی مثل Slack، ایمیل یا PagerDuty می‌فرسته تا تیم بتونه سریع واکنش نشون بده.
- مثال ساده: اگه دیسک سرور پر بشه، Alertmanager یه پیام به کانال Slack تیم می‌فرسته: "دیسک سرور X پر شده!"
- چرا مهمه؟ باعث می‌شه هشدارها به سرعت به دست افراد مناسب برسه.
    
> این 4 وظیفه با هم کاری می‌کنن که سیستم هشداردهی شما منظم، مفید و بدون شلوغی باشه!


### 3.2 معماری Alertmanager
سرویس Alertmanager معمولاً به صورت یک جزء جداگانه در کنار پرومتئوس نصب می‌شود. 
در یک خوشه Kubernetes، می‌توان آن را از طریق Helm Chart (مانند kube-prometheus-stack) نصب کرد.
پرومتئوس هشدارها را به Alertmanager ارسال می‌کند و Alertmanager بر اساس تنظیماتش (مانند فایل `alertmanager.yml`) تصمیم می‌گیرد که چه اقدامی انجام دهد.

<img width="100%" alt="Prometheus-and-Alertmanager-Architecture" src="https://github.com/user-attachments/assets/7717a13b-719f-4d8d-afb2-c6e3b49adb43" />


---

## 4. راه‌های مدیریت هشدارها

مدیریت موثر هشدارها نیازمند رعایت چند اصل است:
1. **وضوح در تعریف هشدارها**: هشدارها باید دقیق و مرتبط با مشکلات واقعی باشند و حتی باید اسامی یا معانی با مسمایی را برای آن انتخاب کرد.
2. **اولویت‌بندی**: هشدارهای بحرانی (Critical) باید از هشدارهای اطلاعاتی (Info) متمایز شوند.
3. **کاهش نویز**: از ایجاد هشدارهای غیرضروری که باعث خستگی تیم می‌شوند (Alert Fatigue) خودداری کنید.
4. **اتوماسیون**: هشدارها را به ابزارهای خودکار متصل کنید تا اقدامات اصلاحی سریع‌تر انجام شوند.
5. **مستندسازی**: برای هر هشدار، یک Runbook (راهنمای عملیاتی) تهیه کنید تا تیم‌ها بدانند چگونه به آن واکنش نشان دهند.

---

## 5. عمیق شدن در Alertmanager

این این مرحله صفر تا صد نصب و پیکربندی **Prometheus Alertmanager** را با تمرکز بر روش دانلود با استفاده از **wget** و همچنین معرفی روش‌های دیگر نصب (مانند استفاده از Docker) شرح میدهیم.
هدف این است که شما بتوانید Alertmanager را به‌طور کامل نصب و پیکربندی کنید و هشدارهای Prometheus را مدیریت کنید.

---

## پیش‌نیازها
- **سیستم‌عامل**: لینوکس (این راهنما بر اساس اوبونتو/Debian یا CentOS نوشته شده است، اما قابل تعمیم به سایر توزیع‌ها است).
- **دسترسی root یا sudo**: برای اجرای دستورات نصب و مدیریت فایل‌ها.
- **پکیج Prometheus نصب‌شده**: فرض می‌شود که Prometheus از قبل روی سیستم نصب و پیکربندی شده است.
- **ابزارهای مورد نیاز**:
  - `پکیج wget` یا `curl` برای دانلود فایل‌ها.
  - `پکیج tar` برای استخراج آرشیوها.
  - `پکیج systemctl` برای مدیریت سرویس‌ها (در صورت استفاده از systemd).

---

## مرحله ۱: ایجاد کاربر و دایرکتوری‌های مورد نیاز

در نصب Alertmanager بهتر است با یک کاربر غیرروت اجرا شود تا امنیت سیستم افزایش یابد.
همچنین، دایرکتوری‌های مشخصی برای ذخیره فایل‌های باینری، پیکربندی و داده‌ها نیاز است.

### ایجاد کاربر
 یک کاربر سیستمی برای Alertmanager ایجاد کنید:
   ```bash
   sudo useradd --no-create-home --shell /bin/false alertmanager
   ```

 بررسی کنید که کاربر ایجاد شده است:
   ```bash
   id alertmanager
   cat /etc/group
   ```

### ایجاد دایرکتوری‌ها
 دایرکتوری‌های مورد نیاز برای ذخیره فایل‌های پیکربندی و داده‌ها را بسازید:
   ```bash
   sudo mkdir -p /etc/alertmanager
   sudo mkdir -p /var/lib/alertmanager
   ```

 مالکیت دایرکتوری‌ها را به کاربر `alertmanager` اختصاص دهید:
   ```bash
   sudo chown alertmanager:alertmanager /etc/alertmanager
   sudo chown alertmanager:alertmanager /var/lib/alertmanager
   ```

---

## مرحله ۲: دانلود و نصب Alertmanager

پکیج Alertmanager را می‌توان از طریق روش‌های مختلفی نصب کرد. در اینجا ابتدا روش دانلود با **wget** توضیح داده می‌شود و سپس روش‌های جایگزین معرفی می‌شوند.

### روش ۱: دانلود و نصب با استفاده از wget
 **دانلود آخرین نسخه Alertmanager**:
   - به صفحه انتشارات رسمی Prometheus در GitHub مراجعه کنید تا آخرین نسخه را بیابید: [https://github.com/prometheus/alertmanager/releases](https://github.com/prometheus/alertmanager/releases).
   - فرض کنید آخرین نسخه 0.28.1 است (این نسخه را با نسخه فعلی جایگزین کنید).
   - دستور زیر را برای دانلود اجرا کنید:
     ```bash
     sudo su
     cd /tmp
     wget https://github.com/prometheus/alertmanager/releases/download/v0.28.1/alertmanager-0.28.1.linux-amd64.tar.gz
     ```

 **استخراج فایل دانلودشده**:
   ```bash
   tar -xvzf alertmanager-0.28.1.linux-amd64.tar.gz
   ```

 **انتقال فایل‌های باینری**:
   فایل‌های اجرایی `alertmanager` و `amtool` را به مسیر مناسب منتقل کنید:
   ```bash
   mv alertmanager-0.28.1.linux-amd64/alertmanager /usr/bin/
   mv alertmanager-0.28.1.linux-amd64/amtool /usr/bin/
   ```

 **اعطای دسترسی مناسب به فایل‌ها**:
   ```bash
   sudo chown alertmanager:alertmanager /usr/bin/alertmanager
   sudo chown alertmanager:alertmanager /usr/bin/amtool
   sudo chmod +x /usr/bin/alertmanager
   sudo chmod +x /usr/bin/amtool
   ```

 **بررسی نصب**:
   برای اطمینان از نصب صحیح، نسخه Alertmanager را بررسی کنید:
   ```bash
   alertmanager --version
   ```

### روش ۲: نصب با استفاده از Docker
اگر ترجیح می‌دهید از Docker استفاده کنید:
1. تصویر رسمی Alertmanager را از Docker Hub بکشید:
   ```bash
   docker pull prom/alertmanager:latest
   ```

2. پکیج Alertmanager را با استفاده از Docker اجرا کنید:
   ```bash
   docker run --name alertmanager -p 9093:9093 \
     -v /etc/alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml \
     prom/alertmanager:latest
   ```

   **توجه**: فایل پیکربندی `alertmanager.yml` باید از قبل در مسیر `/etc/alertmanager/` ایجاد شده باشد.

### روش ۳: نصب با استفاده از بسته‌های مدیریت بسته
برخی توزیع‌های لینوکس ممکن است Alertmanager را در مخازن خود داشته باشند. برای مثال، در اوبونتو:
```bash
sudo apt update
sudo apt install prometheus-alertmanager
```
**توجه**: بسته‌های مدیریت بسته ممکن است نسخه‌های قدیمی‌تر را ارائه دهند، بنابراین استفاده از روش wget یا Docker برای دریافت آخرین نسخه توصیه می‌شود.

---

## مرحله ۳: پیکربندی Alertmanager

پکیج Alertmanager از یک فایل پیکربندی به نام `alertmanager.yml` استفاده می‌کند.
این فایل شامل تنظیمات مربوط به نحوه مدیریت و ارسال هشدارها (مانند ایمیل، Slack، PagerDuty و غیره) است.

 **ایجاد فایل پیکربندی**:
   فایل پیکربندی را در مسیر `/etc/alertmanager/` ایجاد کنید:
   ```bash
   sudo nano /etc/alertmanager/alertmanager.yml
   ```

 **نمونه پیکربندی پایه**:
   محتوای زیر را در فایل `alertmanager.yml` قرار دهید:
   ```yaml
   global:
     # تنظیمات عمومی، مانند سرور SMTP برای ارسال ایمیل
     smtp_smarthost: 'smtp.example.com:587'
     smtp_from: 'alertmanager@example.com'
     smtp_auth_username: 'your_username'
     smtp_auth_password: 'your_password'

   route:
     # مسیر پیش‌فرض برای هشدارها
     receiver: 'team-email'
     group_by: ['alertname', 'cluster', 'service']
     group_wait: 30s
     group_interval: 5m
     repeat_interval: 4h

   receivers:
     - name: 'team-email'
       email_configs:
         - to: 'team@example.com'
           send_resolved: true

     - name: 'slack-notifications'
       slack_configs:
         - api_url: 'https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX'
           channel: '#alerts'
           send_resolved: true
   ```

فایل تنظیمات Alertmanager (معمولاً `alertmanager.yml`) شامل بخش‌های زیر است:

1. **قسمت Global**: 
  - تنظیمات کلی مثل URL وب‌هوک Slack یا تنظیمات SMTP برای ایمیل.
2. **قسمت Route**:
  - تعریف مسیرهای هدایت هشدارها (مثلاً کدام هشدار به کدام مقصد ارسال شود).
3. **قسمت Receivers**:
  - تعریف مقصدهای اعلان (مثل Slack، PagerDuty).
4. **قسمت Inhibit Rules**:
  - قوانینی برای سرکوب هشدارهای کم‌اهمیت.
5. **قسمت group_by**:
  - هشدارها را بر اساس برچسب‌هایی مثل `alertname` و `namespace` گروه‌بندی می‌کند.
  - مثال: اگه 5 سرور تو namespace "test" مشکل CPU داشته باشن، یه پیام می‌فرسته: "مشکل CPU در namespace test".
6. **قسمت group_wait**:
  - مدت زمانی که منتظر می‌ماند تا هشدارهای مرتبط جمع شوند.
  - مثال: اگه CPU سرور بالا بره، 30 ثانیه صبر می‌کنه تا ببینه هشدارهای دیگه‌ای هم هست، بعد یه پیام می‌فرسته.
7. **قسمت group_interval**:
  - فاصله بین ارسال گروه‌های جدید هشدار.
  - مثال: اگه گروهی از هشدارها هر 5 دقیقه یه بار جمع بشن، هر 5 دقیقه یه پیام جدید می‌فرسته.
8. **قسمت repeat_interval**:
  - فاصله زمانی برای تکرار هشدار.
  - مثال: اگه مشکل CPU حل نشه، هر 4 ساعت یه بار دوباره هشدار می‌فرسته.
9. **قسمت slack_configs**:
  - تنظیمات مربوط به ارسال اعلان به Slack.
  - مثال: تنظیم می‌کنه که هشدارها به کانال Slack "#alerts" برن با متن "دیسک پر شده!".

---
 **اعمال مالکیت و دسترسی**:
   ```bash
   sudo chown alertmanager:alertmanager /etc/alertmanager/alertmanager.yml
   sudo chmod 640 /etc/alertmanager/alertmanager.yml
   ```

 **بررسی صحت فایل پیکربندی**: از ابزار `amtool` برای بررسی صحت فایل پیکربندی استفاده کنید.
   ```bash
   amtool check-config /etc/alertmanager/alertmanager.yml
   ```

---

## مرحله ۴: اتصال Alertmanager به Prometheus

برای اینکه Prometheus هشدارها را به Alertmanager ارسال کند، باید فایل پیکربندی Prometheus (`prometheus.yml`) را ویرایش کنید.

فایل پیکربندی Prometheus را باز کنید (معمولاً در `/etc/prometheus/prometheus.yml`):
   ```bash
   sudo nano /etc/prometheus/prometheus.yml
   ```

بخش `alerting` را به فایل اضافه کنید:
   ```yaml
   alerting:
     alertmanagers:
       - static_configs:
           - targets:
               - localhost:9093
   rule_files:
     - "alert.rules.yml"
   ```

یک فایل به نام `alert.rules.yml` در مسیر `/etc/prometheus/` ایجاد کنید:
   ```bash
   sudo nano /etc/prometheus/alert.rules.yml
   ```

نمونه محتوای فایل:
   ```yaml
   groups:
   - name: example
     rules:
     - alert: HighDiskUsage
       expr: 100 * (node_filesystem_size_bytes - node_filesystem_free_bytes) / node_filesystem_size_bytes > 95
       for: 5m
       labels:
         severity: critical
       annotations:
         summary: "High Disk Usage on {{ $labels.instance }}"
         description: "{{ $labels.instance }} has disk usage above 95%."
   ```

**بررسی صحت فایل prometheus.yml**:
   ```bash
   promtool check config /etc/prometheus/prometheus.yml
   ```

برای اعمال تغییرات، سرویس Prometheus را ری‌استارت کنید:
   ```bash
   sudo systemctl restart prometheus
   ```

---

## مرحله ۵: ایجاد سرویس Systemd برای Alertmanager

برای اجرای Alertmanager به‌صورت یک سرویس دائمی در سیستم، یک فایل سرویس Systemd ایجاد کنید.

1. **ایجاد فایل سرویس**:
   ```bash
   sudo nano /etc/systemd/system/alertmanager.service
   ```

2. **محتوای فایل سرویس**:
   محتوای زیر را در فایل قرار دهید:
   ```ini
   [Unit]
   Description=Prometheus Alertmanager Service
   After=network.target

   [Service]
   User=alertmanager
   Group=alertmanager
   Type=simple
   ExecStart=/usr/bin/alertmanager \
     --config.file=/etc/alertmanager/alertmanager.yml \
     --storage.path=/var/lib/alertmanager

   [Install]
   WantedBy=multi-user.target
   ```

**اعمال تغییرات و راه‌اندازی سرویس**:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable alertmanager
   sudo systemctl start alertmanager
   ```

**بررسی وضعیت سرویس**:
   ```bash
   sudo systemctl status alertmanager
   ```

**دسترسی به رابط کاربری Alertmanager**:
   مرورگر خود را باز کنید و به آدرس `http://<server-ip>:9093` بروید. باید رابط کاربری وب Alertmanager را مشاهده کنید.

---

## مرحله ۶: تست و عیب‌یابی

**تست ارسال هشدار**:
   - یک قانون هشدار در فایل `alert.rules.yml` تعریف کنید که به‌راحتی فعال شود (مثلاً بررسی استفاده از CPU).
   - بررسی کنید که هشدار به گیرنده‌های تعریف‌شده (مانند ایمیل یا Slack) ارسال می‌شود.

**بررسی لاگ‌ها**:
   لاگ‌های Alertmanager را برای عیب‌یابی بررسی کنید:
   ```bash
   journalctl -u alertmanager.service
   ```

**تست با amtool**:
   از ابزار `amtool` برای ارسال یک هشدار آزمایشی استفاده کنید:
   ```bash
   amtool alert add alertname=TestAlert severity=critical
   ```

---

## روش‌های جایگزین دانلود و نصب

### ۱. استفاده از curl به‌جای wget
به‌جای `wget` می‌توانید از `curl` استفاده کنید:
```bash
curl -LO https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.28.1.linux-amd64.tar.gz
```

### ۲. استفاده از مخازن بسته
در برخی توزیع‌ها، می‌توانید از مدیر بسته استفاده کنید، اما ممکن است نسخه قدیمی‌تر نصب شود:
```bash
sudo apt install prometheus-alertmanager
```

### ۳. استفاده از Helm در Kubernetes
اگر از Kubernetes استفاده می‌کنید، می‌توانید Alertmanager را با Helm نصب کنید:
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install alertmanager prometheus-community/alertmanager
```

---

## نکات نهایی نصب Alert Manager
- **به‌روزرسانی Alertmanager**: برای به‌روزرسانی، نسخه جدید را دانلود کنید و فایل‌های باینری را جایگزین کنید. سپس سرویس را ری‌استارت کنید.
- **امنیت**: برای افزایش امنیت، فایروال را تنظیم کنید تا فقط پورت 9093 برای دسترسی‌های مجاز باز باشد.
- **یکپارچه‌سازی با Grafana**: می‌توانید Alertmanager را با Grafana ادغام کنید تا هشدارها را در داشبوردهای Grafana مشاهده کنید.


---


### 6.2 مفهوم Matchers در Alertmanager
باید در نظر گرفت که ** Matchers** در Alertmanager برای فیلتر کردن و هدایت هشدارها استفاده می‌شوند.
در واقع بررسی میکند هشداری که بوجود آمده(بر اساس alert_rules.yml ) لیبلش چیست؟ تطابقش چگونه است تا ارسال شود به مسیر مدنظر.

**مفهوم Route (مسیر)**
مثل یک "نقشه راه" برای هشدارها عمل می‌کنه. مشخص می‌کنه که هر هشدار کجا بره و به کی تحویل داده بشه.
مثلاً می‌تونی بگی: «اگر هشداری با برچسب severity=critical اومد، برو به کانال Slack و به تیم DevOps خبر بده.

سه نوع Matcher داریم:
 - **مدل match**: تطبیق دقیق با یک مقدار (مثلاً `severity=critical`).
 - **مدل match_re**: تطبیق با استفاده از Regular Expression (مثلاً `severity=~critical|warning`).
 - **مدل matchers**: ترکیبی از match و match_re برای نسخه‌های جدیدتر Alertmanager.

#### مثال استفاده از Matchers:
```yaml
route:
  receiver: 'slack-critical'
  match:
    severity: critical
  match_re:
    namespace: test.*
```

**توضیحات:**

 - `سازو کار match`: هشدارهایی که دقیقاً `severity=critical` دارند به این مسیر هدایت می‌شوند.
 - `سازو کار match_re`: هشدارهایی که نام فضای کاری (namespace) آن‌ها با `test` شروع می‌شود.

---

## 7. حالات Pending و Firing

 - **مرحله Pending**: وقتی شرط یک هشدار برقرار می‌شود، ابتدا به حالت Pending می‌رود. در این حالت، پرومتئوس منتظر می‌ماند تا شرط به مدت زمان مشخصی (تعریف‌شده در `for`) برقرار بماند.
 - **مرحله Firing**: اگر شرط برای مدت زمان مشخص برقرار بماند، هشدار به حالت Firing می‌رود و به Alertmanager ارسال می‌شود.

**چرا این دو حالت؟**
این مکانیزم از ارسال هشدارهای لحظه‌ای و ناپایدار (مثلاً به دلیل نوسانات موقتی) جلوگیری می‌کند.
برای مثال، اگر CPU Usage برای چند ثانیه به 80% برسد، ممکن است نخواهید هشداری دریافت کنید، اما اگر این وضعیت 30 ثانیه ادامه پیدا کند، هشدار فعال می‌شود.

---

## 8. مثال عملی: تنظیم هشدار برای آسیب‌پذیری‌ها

فرض کنید از **Trivy Operator** برای اسکن آسیب‌پذیری‌های امنیتی در خوشه Kubernetes استفاده می‌کنید. 
می‌خواهید اگر یک آسیب‌پذیری بحرانی (Critical Vulnerability) در فضای کاری `test` شناسایی شد، هشداری دریافت کنید.

### 8.1 نصب Trivy Operator
```bash
helm repo add aqua https://aquasecurity.github.io/helm-charts/
helm repo update
helm upgrade --install trivy-operator aqua/trivy-operator \
  --namespace trivy-system --create-namespace \
  --set trivy.ignoreUnfixed=true \
  --set serviceMonitor.enabled=true
```

### 8.2 تعریف PrometheusRule برای Trivy
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: trivy-alerts
  namespace: monitoring
  labels:
    prometheus: example
spec:
  groups:
  - name: trivy
    rules:
    - alert: TrivyNewCriticalVulnerability
      expr: trivy_vulnerability_critical_count{namespace="test"} > 0
      for: 30s
      labels:
        severity: critical
      annotations:
        summary: "New critical vulnerability detected in test namespace"
        description: "A critical vulnerability was found in {{ $labels.namespace }} namespace. Check the Trivy vulnerability report for details."
        runbook_url: "https://example.com/runbook/trivy-critical"
```

### 8.3 تنظیم Alertmanager برای ارسال به Slack
```yaml
global:
  slack_api_url: 'https://hooks.slack.com/services/xxx/yyy/zzz'

route:
  receiver: 'slack-notifications'
  group_by: ['alertname', 'namespace']
  match:
    severity: critical

receivers:
- name: 'slack-notifications'
  slack_configs:
  - channel: '#alerts'
    send_resolved: true
    text: "<!channel> {{ .CommonAnnotations.summary }}: {{ .CommonAnnotations.description }}"
```

### 8.4 اعمال تنظیمات
 فایل PrometheusRule را اعمال کنید:
```bash
kubectl apply -f trivy-alerts.yaml -n monitoring
```
 فایل تنظیمات Alertmanager را به‌روزرسانی کنید:
```bash
kubectl edit configmap alertmanager-prometheus-alertmanager -n monitoring
```

---

## 9. بررسی پنل Alertmanager

برای دسترسی به Alertmanager، می‌توانید سرویس آن را Port Forward کنید:
```bash
kubectl port-forward svc/prom-prometheus-stack-alertmanager 9093:9093 -n monitoring
```

سپس در مرورگر به آدرس `http://localhost:9093` بروید. در پنل Alertmanager:
 **قسمت Alerts**: لیستی از هشدارهای فعال (Pending یا Firing) را نشان می‌دهد.
 **قسمت Silences**: می‌توانید هشدارها را برای مدت زمان مشخصی غیرفعال کنید.
 **قسمت Status**: وضعیت Alertmanager و تنظیمات آن را نمایش می‌دهد.

### مثال عملی در پنل:
فرض کنید هشدار `TrivyNewCriticalVulnerability` فعال شده است:
1. در بخش Alerts، هشدار را با برچسب `severity=critical` می‌بینید.
2. می‌توانید روی آن کلیک کنید و جزئیات (مانند Annotations و Runbook URL) را مشاهده کنید.
3. اگر بخواهید هشدار را موقتاً غیرفعال کنید، از بخش Silences استفاده کنید.

---

## 10. نکات پیشرفته و بهترین روش‌ها

 **گروه‌بندی هشدارها**: از `group_by` برای کاهش تعداد اعلان‌ها استفاده کنید.
  مثلاً هشدارهای یک Namespace را در یک اعلان ترکیب کنید.
 **ایجاد Runbook**: برای هر هشدار، یک Runbook با دستورالعمل‌های دقیق برای رفع مشکل ایجاد کنید.
 **تست هشدارها**: قبل از استفاده در محیط Production، هشدارها را در محیط تست بررسی کنید.
 **مانیتورینگ Alertmanager**: از متریک‌های خود Alertmanager (مثل `alertmanager_alerts`) برای نظارت بر سلامت آن استفاده کنید.

---

## 11. جمع‌بندی

هشدارها در پرومتئوس و Alertmanager ابزارهای قدرتمندی برای نظارت و واکنش به مشکلات سیستم هستند. 
با تعریف قوانین دقیق، تنظیم Alertmanager برای ارسال اعلان‌ها به مقصدهای مناسب، و استفاده از بهترین روش‌ها، می‌توانید یک سیستم نظارتی قوی و کارآمد ایجاد کنید.
در این آموزش، از مفاهیم پایه تا پیاده‌سازی عملی و کار با پنل Alertmanager را پوشش دادیم. حالا شما آماده‌اید تا هشدارهای خود را در محیط واقعی پیاده‌سازی کنید!


---
## مثال ساده: سرور در دسترس است یا خیر؟
سرویس Prometheus متریکی به نام up دارد که وضعیت یک سرویس را نشان می‌دهد:

> - وقتی up{job="my_service"} == 1 یعنی سرویس در دسترس است.
> - وقتی up{job="my_service"} == 0 یعنی سرویس از دسترس خارج است.
> - ما می‌خواهیم اگر سرور Down شد، یک هشدار دریافت کنیم.


### نوشتن فایل Alerting Rule
فایل‌های قوانین هشدار معمولاً در یک فایل YAML جداگانه (مثلاً alert_rules.yml) نوشته می‌شوند و به Prometheus معرفی می‌شوند.
بیایید یک قانون ساده بنویسیم ، فایل alert_rules.yml:
```yaml
groups:
- name: example-alerts
  rules:
  - alert: ServiceDown
    expr: up{job="my_service"} == 0
    for: 5m
    labels:
      severity: critical
      team: devops
    annotations:
      summary: "Service {{ $labels.instance }} is down"
      description: "{{ $labels.instance }} has been down for more than 5 minutes."
```
### توضیحات:

- قسمت groups: قوانین هشدار در گروه‌ها سازمان‌دهی می‌شوند. اینجا یک گروه به نام example-alerts تعریف کردیم.
- قسمتalert: نام هشدار (مثلاً ServiceDown) که در Alertmanager نمایش داده می‌شود.
- قسمتexpr: عبارتی که Prometheus بررسی می‌کند. اینجا چک می‌کنیم که اگر متریک up برای سرویس my_service صفر باشد، یعنی سرویس خاموش است.
- قسمتfor: مدت زمانی که شرط باید برقرار باشد تا هشدار فعال شود (اینجا ۵ دقیقه).
- قسمتlabels: برچسب‌هایی برای دسته‌بندی هشدار (مثلاً severity: critical برای نشان دادن اهمیت).
- قسمتannotations: اطلاعات اضافی برای توضیح هشدار:
- قسمتsummary: خلاصه‌ای کوتاه از مشکل.
- قسمتdescription: توضیح مفصل‌تر که می‌تواند از متغیرهایی مثل {{ $labels.instance }} استفاده کند تا نام نمونه (instance) را نمایش دهد.


### پیکربندی Prometheus
برای استفاده از این قانون، باید فایل alert_rules.yml را به Prometheus معرفی کنید. در فایل پیکربندی Prometheus (مثلاً prometheus.yml)، بخش rule_files را به این صورت تنظیم کنید:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'my_service'
    static_configs:
      - targets: ['localhost:8080']  # آدرس سرویسی که می‌خواهید مانیتور کنید

rule_files:
  - 'alert_rules.yml'  # مسیر فایل قوانین هشدار
```

- بعد از این کار، Prometheus را ری‌استارت کنید تا قوانین بارگذاری شوند:
```bash
sudo systemctl restart prometheus
```

### پیکربندی Alertmanager برای ارسال پیام
سرویس Alertmanager مسئول مدیریت هشدارها و ارسال آن‌ها به کانال‌های مختلف (مثل ایمیل، Slack، Telegram و غیره) است. 
بیایید یک پیکربندی ساده برای Alertmanager بنویسیم که هشدارها را به یک کانال Slack ارسال کند.

فایل alertmanager.yml:
```yaml
global:
  slack_api_url: 'https://hooks.slack.com/services/XXXX/YYYY/ZZZZ'  # وب‌هوک Slack خود را وارد کنید

route:
  receiver: 'slack-notifications'
  group_by: ['alertname', 'instance']

receivers:
- name: 'slack-notifications'
  slack_configs:
  - channel: '#alerts'
    send_resolved: true
    title: "{{ .CommonAnnotations.summary }}"
    text: "{{ .CommonAnnotations.description }}"
```

### راه‌اندازی Alertmanager
فایل alertmanager.yml را در مسیر مناسب (مثلاً /etc/alertmanager/) ذخیره کنید.
سرویس Alertmanager را ری‌استارت کنید:
```bash
sudo systemctl restart alertmanager
```
- مطمئن شوید که Prometheus به Alertmanager متصل است. در فایل prometheus.yml، بخش alerting را اضافه کنید:
```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']  # آدرس Alertmanager
```


### تست و بررسی
برای تست، می‌توانید سرویسی که مانیتور می‌کنید (مثلاً localhost:8080) را خاموش کنید تا متریک up به 0 تغییر کند. بعد از ۵ دقیقه (طبق تنظیم for: 5m)، یک هشدار به alert manager ارسال میشود.
