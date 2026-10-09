***

### **DNS 1: What is DNS, its Functions, and an Explanation of the Hosts File - explain** 
***
---


مستند مرجعي شامل يغطي الجزء الأول من سلسلة بروتوكول **DNS (Domain Name System)**، موضحاً المفهوم العام، الوظائف الثلاث الرئيسية (**Name Resolution**, **Resources Locator**, **Service Locator**)، والتشريح التفصيلي لملف الـ **Hosts File** وأولويته داخل نظام التشغيل.

---

## 🌐 1. ما هو بروتوكول DNS؟ (Domain Name System)

### 🔹 التعريف والمفهوم:
هو بروتوكول يعمل في طبقة التطبيقات (**Application Layer - Layer 7**) يُصنف بأنه "دليل الهاتف" (Phonebook) للشبكات. وظيفته الأساسية هي التجميع والربط بين الأسماء النصية المفهومة للبشر (مثل `pdc.test.local` أو `google.com`) والعناوين الرقمية (IPv4 / IPv6) التي تستخدمها الأجهزة للتواصل.

### ⚙️ المنافذ وبروتوكولات النقل (Ports & Protocols):
يعتمد DNS على المنفذ القياسي **Port 53** باستخدام بروتوكولي النقل معاً حسب طبيعة العملية:

* **UDP Port 53 (الافتراضي والأساسي):** يُستخدم في كافة استعلامات الأجهزة العادية (Client Queries) لسرعته العالية وقلة استهلاكه للباندويث.
* **TCP Port 53 (للعمليات الضخمة والأمان):** يُستخدم في حالتين أساسيتين:
  1. عمليات نقل النطاقات بين السيرفرات (**Zone Transfers**).
  2. عندما يتجاوز حجم رد استعلام الـ DNS سعة **512 Bytes** (أو عند استخدام تقنيات التشفير مثل DNSSEC).

---

## 🎯 2. الوظائف الثلاث الأساسية لبروتوكول DNS

يقوم الـ DNS بثلاث وظائف جوهرية تعتمد عليها كافة التطبيقات وأنظمة التشغيل:

### 1️⃣ Name Resolution (تحليل وتطابق الأسماء والعناوين)
هي الوظيفة التقليدية لترجمة الأسماء إلى عناوين والعكس:
* **Forward Lookup (التحليل الأمامي):** تحويل اسم النطاق (FQDN) إلى عنوان IP رقمي.
  * *مثال:* طلب `pdc.test.local` ➡️ يرجع بـ `192.168.1.10` (عبر `A` / `AAAA` Records).
* **Reverse Lookup (التحليل العكسي):** تحويل عنوان الـ IP إلى اسم النطاق المرتبط به.
  * *مثال:* طلب `192.168.1.10` ➡️ يرجع بـ `pdc.test.local` (عبر `PTR` Record).

---

### 2️⃣ Resource Locator (تحديد مواقع الموارد والأنظمة)
توجيه حركة المرور وتحديد الموارد الاستضافية للخدمات المختلفة بناءً على نوع المورد المطلوب داخل الشبكة أو الإنترنت:
* **Mail Servers (تحديد سيرفرات البريد):** استخدام سجلات **MX Records** لتوجيه الرسائل إلى سيرفر الـ Exchange / Mail المعتمد للشركة.
* **Canonical Names & Aliases (الأسماء المستعارة):** استخدام سجلات **CNAME Records** لتوجيه أكثر من اسم موجه لنفس السيرفر الأصلي (مثل توجيه `www.test.local` و `app.test.local` إلى السيرفر الرئيسي `server01.test.local`).
* **Text / Security Metadata (بيانات التوثيق والتحقق):** استخدام سجلات **TXT Records** لتحديد سياسات الأمان وتأكيد ملكية الموارد (مثل تقنيات SPF, DKIM, DMARC للحماية من تزوير الإيميلات).

---

### 3️⃣ Service Locator (تحديد مواقع خدمات الدومين والنظام)
تُعد هذه الوظيفة **العصب المحرك لبيئات Active Directory**؛ حيث لا يبحث الجهاز عن سيرفر باسمه فقط، بل يبحث عن **"من يقدّم خدمة معينة داخل الدومين؟"**:

* **سجلات الـ SRV (Service Records):** هي سجلات متخصصة تدل أجهزة العميل (Win10 / Win11) على الأجهزة التي تقدم خدمات التشغيل والمصادقة.
* **أبرز الخدمات التي يتم تحديد موقعها عبر الـ SRV:**
  * **LDAP (`_ldap`):** للوصول لدليل البيانات والاستعلام عن حسابات المستخدمين والأجهزة.
  * **Kerberos (`_kerberos`):** للوصول لسيرفر المصادقة وتوليد التذاكر (Tickets Logons).
  * **Global Catalog (`_gc`):** للبحث السريع في كائنات الدومين على مستوى الـ Forest.

#### 📝 صيغة سجل الـ SRV Record التركيبية:
```text
_service._proto.name. TTL Class SRV Priority Weight Port Target.
```
* **مثال فعلي داخل AD:**
  `_ldap._tcp.dc._msdcs.test.local 600 IN SRV 0 100 389 pdc.test.local`
  *(يقوم الجهاز بقراءة هذا السجل ليعرف أن الخدمة المطلوبة هي LDAP على بورت 389 وأن السيرفر الموفر لها هو `pdc.test.local`).*

---

## 📄 3. ملف الـ Hosts المحلي (The Hosts File)

### 🔹 المفهوم والتاريخ:
قبل اختراع نظام الـ DNS في عام 1983، كانت شبكة ARPANET تعتمد على ملف نصي محلي يدوي يُدعى `HOSTS.TXT` يتم إدارته مركزياً وتنزل نسخة منه على كل جهاز لمعرفة عناوين الأجهزة الأخرى.

اليوم، ما زال ملف الـ **Hosts** موجوداً داخل كافة نظم التشغيل (Windows, Linux, macOS) كملف نصي محلي يُستخدم لربط العناوين بالأسماء **مباشرة على الجهاز نفسه** دون الحاجة للرجوع للشبكة.

### 📂 مسار الملف داخل أنظمة التشغيل:
* **Windows OS:** `C:\Windows\System32\drivers\etc\hosts`
* **Linux / macOS:** `/etc/hosts`

### ✍️ الصيغة البرمجية لملف الـ Hosts Syntax:
يُكتب الملف بأسلوب بسيط: **[عنوان الـ IP]** متبوعاً بـ **[اسم الجهاز/النطاق]**:
```text
# Example Hosts File Configuration
127.0.0.1       localhost
192.168.1.10    pdc.test.local  pdc
192.168.1.50    app-server.local
```

---

## 🔝 4. أولوية تنفيذ الترجمة داخل نظام التشغيل (Resolution Order)

عندما تكتب اسم سيرفر أو موقع في متصفحك أو موجه الأوامر (مثل `ping app-server`)، يتبع نظام التشغيل (Windows) التسلسل الهرمي التالي بالترتيب الدقيق للبحث عن الـ IP:

```
1. Local Hostname Check (هل الاسم هو اسم الجهاز نفسه؟)
        │
        ▼
2. DNS Client Resolver Cache (فحص الكاش المحلي للأجهزة ipconfig /displaydns)
        │
        ▼
3. Local Hosts File (فحص ملف الـ Hosts المحلي) 👈
        │
        ▼
4. Preferred DNS Server (إرسال استعلام لسيرفر الـ DNS المفصل)
        │
        ▼
5. Alternate Name Resolution (NetBIOS / LLMNR / mDNS - في حال فشل الـ DNS)
```

> 💡 **قاعدة أمنية وإدارية هامة:**
> ملف الـ **Hosts يُنفذ ويُفضل دائماً قبل الذهاب لسيرفر الـ DNS**. لذلك إذا وضعت توجيهاً داخل ملف الـ Hosts، سيتجاهل الجهاز سيرفر الـ DNS تماماً وينفذ ما بداخل الملف.

---



### **DNS 2: Understanding DNS and the Name Resolution Process - explain** 
***
---

مستند مرجعي شامل يغطي الجزء الثاني الممتد من سلسلة بروتوكول **DNS (Domain Name System)**. يوضح المستند بالتفصيل مفاهيم التحليل المحلي، سلطة الخوادم (Authoritative vs Non-Authoritative)، أنواع الاستعلامات (Recursive vs Iterative)، وآليات التوجيه والتفويض داخل الخوادم (**Forwarders**, **Conditional Forwarders**, **Root Hints**).

---

## 💻 1. التحليل المباشر على مستوى الجهاز (Local Scope)

قبل خروج أي طلب شبكي من الجهاز إلى سيرفر الـ DNS، يمر الاستعلام عبر طريقتين محليتين داخل نظام التشغيل:

### 🔹 1. Local Host (المضيف المحلي)
* **المفهوم:** هو الاسم أو الواجهة البرمجية المخصصة للإشارة إلى **الجهاز نفسه** دون الحاجة للخروج كروت الشبكة الفيزيائي.
* **العناوين المخصصة (Loopback Address):**
  * `127.0.0.1` في بروتوكول IPv4.
  * `::1` في بروتوكول IPv6.
* **الوظيفة:** اختبار الاتصال بالخدمات والبرمجيات المضافة على نفس الجهاز المحلي (Local Stack Testing).

### 🔹 2. Hosts File (ملف العناوين المحلي)
* **المفهوم:** ملف نصي استاتيكي محلي موجود داخل نظام التشغيل يُستخدم لربط أسماء النطاقات بعناوين الـ IP يدوياً.
* **المسار:** `C:\Windows\System32\drivers\etc\hosts` (على Windows) أو `/etc/hosts` (على Linux).
* **الأولوية والتأثير:** يتم فحصه بواسطة **DNS Client Resolver** بعد الكاش المحلي وقبل الاتصال بسيرفر الـ DNS. يتجاوز هذا الملف (Overrides) سيرفر الـ DNS بالكامل؛ إذا وجد فيه توجيه للـ IP، يتم الاتصال مباشرة.

---

## 🔐 2. سلطة الخوادم وطبيعة الإجابات (DNS Server Authority)

تُصنف خوادم الـ DNS بناءً على مدى ملكيتها وسلطتها على البيانات الموجودة لديها إلى نوعين:

### 🔹 1. Authoritative DNS Server (الخادم صاحب السلطة / المرجع)
* **المفهوم:** هو السيرفر الذي يمتلك الملفات والمستندات الأصلية (**Zone Files**) الخاصة بالدومين.
* **طبيعة الإجابة:** عند استعلامه عن جهاز داخل نطاقه، يرد بـ **Authoritative Answer** (تظهر فيها علامة `AA Flag` في حزمة الـ DNS)، وتكون إجابته قاطعة ولا تقبل الشك.
* **مثال:** سيرفر الـ Domain Controller الخاص بشركتك `pdc.test.local` هو Authoritative بالنسبة لجميع الأجهزة التابعة للنطاق `test.local`.

### 🔹 2. Non-Authoritative DNS Server (الخادم غير المرجع)
* **المفهوم:** هو سيرفر لا يمتلك ملفات الـ Zone الأصلية للدومين المطلوب، ولكن يقدم إجابات بناءً على:
  1. بيانات مخزنة سابقاً في الـ **Cache** لديه.
  2. استعلام سيرفرات أخرى نيابة عن العميل (عبر Recursion/Forwarding).
* **طبيعة الإجابة:** يرد بـ **Non-Authoritative Answer**، مما يعني أن المعلومة صحيحة ومأخوذة من الكاش ولكنها ليست صادرة من المصدر الأصلي مباشرة.

---

## 🔍 3. أنواع استعلامات الـ DNS (DNS Query Types)

تتم عملية نقل البيانات والاستفسار داخل نظام الـ DNS عبر نوعين رئيسيين من الاستعلامات:

```
[ Client ] ───── (1) Recursive Query ─────> [ Local DNS Server ]
                                                  │
                                          (2) Iterative Queries
                                                  │
                                                  ▼
                                      [ Root / TLD / Auth Servers ]
```

### 1️⃣ Recursive Query (الاستعلام التكراري الشامل)
* **المسار:** بين **العميل (Client)** و**سيرفر الـ DNS المحلي (Local Resolver)**.
* **طريقة العمل:** يطلب العميل من السيرفر إجابة نهائية وقاطعة: *"إما أن تأتيني بالـ IP المطلوب، أو تخبرني بوضوح أن هذا الاسم غير موجود (`NXDOMAIN`)"*. لا يقوم العميل بأي رحلة بحث بنفسه، بل يترك العبء كاملاً على السيرفر.

### 2️⃣ Iterative Query (الاستعلام التكراري المرحلي)
* **المسار:** بين **سيرفر الـ DNS المحلي** و**الخوادم الخارجية** (`Root`, `TLD`, `Authoritative`).
* **طريقة العمل:** يطلب السيرفر المحلي من السيرفرات الأخرى: *"هل تعرف الـ IP لهذا الاسم؟ إذا لم تكن تعرفه، دلني على عنوان السيرفر التالي في الشجرة لأستعلم منه"*. ترد السيرفرات بـ Referral (عنوان الخطوة التالية).

---

## 🔀 4. آليات التوجيه والتحويل داخل الخادم (Forwarding Mechanics)

عندما يستقبل سيرفر الـ DNS المحلي استعلاماً عن دومين **ليس صاحبه (Non-Authoritative)** (مثل طلب موظف الوصول لموقع `google.com`)، يستخدم إحدى الآليات التالية للتوجيه:

### 1️⃣ Forwarder DNS (الموجه العام)
* **المفهوم:** سيرفر DNS خارجي يُحدد داخل إعدادات السيرفر المحلي لتحويل **كافة الاستعلامات الخارجية** إليه (مثل `8.8.8.8` أو DNS الخاص بـ ISP).
* **الأهمية:**
  * **الأمان:** منع خروج جميع أجهزة الشبكة إلى إنترنت للبحث عن الـ DNS؛ فقط سيرفر الـ DNS الداخلي هو من يتصل بالخارج عبر Port 53.
  * **الكاش المركزي:** تجميع نواتج التصفح لكل الموظفين في كاش واحد لتسريع الشبكة.

### 2️⃣ Conditional Forwarder (الموجه الشرطي)
* **المفهوم:** توجيه الاستعلامات بناءً على **اسم دومين محدد بشرط** (Domain-Specific).
* **حالات الاستخدام:**
  * عند ربط شركتك (`companyA.local`) بشركة شريكة (`companyB.local`) عبر VPN. يتم إعداد شرط: *"أي طلب ينتهي بـ `companyB.local` يتم تحويله مباشرة إلى IP سيرفر الشركة الشريكة (`10.20.0.5`)"*.
  * علاقات الثقة بين بيئات الـ Active Directory (**Forest Trusts**).
* **الميزة:** السرعة المباشرة وتجنب إرسال استعلامات شبكات الشركات الخاصة إلى إنترنت.

### 3️⃣ Root Hints (تلميحات الجذر)
* **المفهوم:** قائمة ملفات مدمجة داخل سيرفر الـ DNS تحتوي على العناوين الرقمية لـ **13 مجموعة سيرفرات جذر عالمية** (من `A.ROOT-SERVERS.NET` إلى `M.ROOT-SERVERS.NET`).
* **الوظيفة:** تعتبر **الخيار الأخير (Ultimate Fallback)**. إذا لم يتم إعداد Standard Forwarder، أو في حال توقفه عن الاستجابة، يلجأ السيرفر فوراً إلى Root Hints ليبدأ رحلة البحث المرحلية (Iterative Search) بنفسه بدءاً من الجذر `.`.

---

## 🔄 5. دورة الحياة الكاملة لطلب الـ DNS (Full Resolution Order)

عندما يطلب العميل الوصول إلى موقع أو سيرفر (مثل `app.partner.com`):

```
[ Step 1: Local Machine Check ]
 ├── Check Local Host / Loopback
 ├── Check Local DNS Client Cache (ipconfig /displaydns)
 └── Check Local Hosts File (C:\Windows\System32\drivers\etc\hosts)
         │
         ▼ (If not found locally)
[ Step 2: Query Preferred DNS Server ] ──► Sends Recursive Query to Local DNS
         │
         ▼
[ Step 3: Local DNS Server Evaluation ]
 ├── 1. Is Local Server Authoritative for 'partner.com'?
 │      └── YES ──► Returns Authoritative Answer (AA).
 │
 ├── 2. Is the record in Local DNS Cache?
 │      └── YES ──► Returns Non-Authoritative Answer.
 │
 ├── 3. Is there a Conditional Forwarder for 'partner.com'?
 │      └── YES ──► Forwards request directly to Specified Server IP.
 │
 ├── 4. Is a Standard Forwarder configured (e.g., 8.8.8.8)?
 │      └── YES ──► Forwards request to External Forwarder.
 │
 └── 5. Fallback to Root Hints (.)
        └── NO Forwarder / Failed ──► Performs Iterative Queries:
               Root Server (.) ──► TLD Server (.com) ──► Authoritative Server
```

---

## 📊 6. Comparison Table (الشامل لكل مكونات الدرس)

| Element / Concept | Category | Core Function / Behavior | Primary Value |
| :--- | :--- | :--- | :--- |
| **Local Host** | Machine Scope | واجهة الاتصال الداخلي المرتد (`127.0.0.1`). | اختار التطبيقات والخدمات المحلية. |
| **Hosts File** | Machine Scope | ملف نصي محلي لربط IPs بأسماء النطاقات يدوياً. | يتجاوز الـ DNS وله أولوية تنفيذ مرتفعة. |
| **Authoritative DNS** | Server Authority | خادم يمتلك ملفات الـ Zone الأصلية للدومين. | يقدم إجابات قاطعة (Authoritative Answers). |
| **Non-Authoritative** | Server Authority | خادم يقدم إجابات بناءً على الكاش أو البحث. | تسريع التصفح وتخفيف الضغط عن السيرفر الأصلي. |
| **Recursive Query** | Query Type | طلب العميل المباشر من السيرفر (إجابة نهائية). | يريح العميل من عناء البحث المرحلي. |
| **Iterative Query** | Query Type | طلب السيرفر من خوادم عالمية (توجيه خطوة بخطوة). | الوصول للعنوان عبر الهيكلية الشجرية. |
| **Forwarder DNS** | Server Forwarding | تحويل كافة الطلبات الخارجية المجهولة لسيرفر عام. | توفير الأمان وتجميع الكاش لجميع العملاء. |
| **Conditional Forwarder**| Server Forwarding | تحويل الطلبات الخارجي بناءً على اسم دومين محدد. | الربط بين الشركات والتنقل في الـ Active Directory. |
| **Root Hints** | Server Fallback | قائمة العناوين الخاصة بسيرفرات الجذر العالمية (13). | الخيار الأخير للبحث في حال عدم وجود Forwarder. |



---
### **DNS 2: Understanding DNS and the Name Resolution Process - LAB**
***
---


دليل عملي معملي يعتمد كلياً على الواجهة الرسومية (**GUI-Only Practical LAB**) لتطبيق وتنفيذ مفاهيم تحليل الأسماء (**Name Resolution Process**)، مناطق البحث الأمامي والعكسي (**FLZ / RLZ**)، وتوجيه الاستعلامات الشامل والشريطي (**Global & Conditional Forwarders**) عبر برنامج **DNS Manager (`dnsmgmt.msc`)**.

---

## 🛠️ 1. مخطط البيئة المعملية والافتراضات (Lab Topology & GUI Setup)

* **Primary DC / DNS Server (`PDC01`):**
  * **IP Address:** `192.168.1.10 /24`
  * **Domain Name:** `test.local`
  * **Tool:** DNS Manager (`dnsmgmt.msc`)
* **Enterprise Client (`WIN-CLIENT01`):**
  * **IP Address:** `192.168.1.50 /24`
  * **Preferred DNS:** `192.168.1.10`
* **Partner Network DNS Server (Simulated):**
  * **Target Network:** `partner.local`
  * **Partner DNS IP:** `10.10.20.5`

---

## 🧪 LAB Task 1: Forward Lookup Zone (FLZ) via GUI

### 🎯 الهدف:
إنشاء **Forward Lookup Zone** وتطعيمها بسجلات مختلفة (**A**, **CNAME**) باستخدام الواجهة الرسومية واختبار تحويل الاسم لـ IP.

### 📝 الخطوات العملية بالواجهة الرسومية:

#### أ) إنشاء Forward Lookup Zone عبر الـ Wizard:
1. اضغط `Win + R` واكتب **`dnsmgmt.msc`** ثم اضغط **Enter** لفتح **DNS Manager**.
2. من القائمة اليسرى، وسّع اسم السيرفر `PDC01`.
3. اضغط كليك يمين على مجلد **Forward Lookup Zones** ⬅️ اختر **New Zone...**.
4. في شاشة الترحيب بـ Wizard، اضغط **Next**.
5. **Zone Type:** اختر **Primary zone** ⬅️ اضغط **Next**.
6. **Zone Name:** اكتب اسم الزون `test.local` ⬅️ اضغط **Next**.
7. **Zone File:** اترك الاسم الافتراضي (`test.local.dns`) ⬅️ اضغط **Next**.
8. **Dynamic Update:** اختر *Do not allow dynamic updates* (أو الخيار المناسب لبيئتك) ⬅️ اضغط **Next** ثم **Finish**.

```
[Forward Lookup Zones] ──(Right Click)──► New Zone...
     ├── Type: Primary Zone
     ├── Name: test.local
     └── File: test.local.dns
```

#### ب) إضافة السجلات (A & CNAME) داخل الزون:
1. اضغط كليك يمين داخل المساحة الفارغة للزون `test.local` ⬅️ اختر **New Host (A or AAAA)...**.
   * **Name:** اكتب `appserver`.
   * **IP address:** اكتب `192.168.1.100`.
   * ضع علامة صح `☑️` على خيار **Create associated pointer (PTR) record** (إذا كانت Reverse Zone موجودة).
   * اضغط **Add Host** ثم **OK**.
2. اضغط كليك يمين مجدداً ⬅️ اختر **New Alias (CNAME)...**.
   * **Alias name:** اكتب `web`.
   * **Fully qualified domain name (FQDN) for target host:** اضغط على **Browse...** واختر `appserver.test.local` (أو اكتبها مباشرة).
   * اضغط **OK**.

---

## 🧪 LAB Task 2: Reverse Lookup Zone (RLZ) & PTR Records via GUI

### 🎯 الهدف:
إنشاء منطقة البحث العكسي للشبكة `192.168.1.0/24` وإنشاء سجلات الـ **PTR** لترجمة عناوين الـ IP إلى FQDN عبر الواجهة الرسومية.

### 📝 الخطوات العملية بالواجهة الرسومية:

#### أ) إنشاء Reverse Lookup Zone:
1. داخل **DNS Manager** (`dnsmgmt.msc`)، اضغط كليك يمين على مجلد **Reverse Lookup Zones** ⬅️ اختر **New Zone...**.
2. اضغط **Next** في شاشة الترحيب.
3. **Zone Type:** اختر **Primary zone** ⬅️ اضغط **Next**.
4. **IPv4 or IPv6:** حدد **IPv4 Reverse Lookup Zone** ⬅️ اضغط **Next**.
5. **Network ID:** اكتب أول 3 خانات من شبكتك: **`192.168.1`** (لاحظ أن السيرفر سيُنشئ تلقائياً اسم المنطقة: `1.168.192.in-addr.arpa`) ⬅️ اضغط **Next**.
6. **Zone File:** اترك الاسم الافتراضي ⬅️ اضغط **Next** ثم **Finish**.

```
[Reverse Lookup Zones] ──(Right Click)──► New Zone...
     ├── Type: Primary Zone
     ├── Scope: IPv4 Reverse Lookup
     └── Network ID: 192.168.1
```

#### ب) إضافة سجل Pointer (`PTR Record`):
1. افتح الزون العكسية الجديدة `1.168.192.in-addr.arpa`.
2. اضغط كليك يمين في مساحة السجلات ⬅️ اختر **New Pointer (PTR)...**.
3. **Host IP number:** اكتب الجزء الأخير من الـ IP (مثلاً: `100` للجهاز `192.168.1.100`).
4. **Host name:** اضغط **Browse...** واختر الجهاز `appserver.test.local`.
5. اضغط **OK**.

---

## 🧪 LAB Task 3: Global Forwarders Configuration via GUI

### 🎯 الهدف:
توجيه جميع استعلامات الإنترنت العامة المجهولة (`*`) من السيرفر المحلي إلى Public DNS (مثل `8.8.8.8` أو `1.1.1.1`) عبر شاشة الخصائص.

### 📝 الخطوات العملية بالواجهة الرسومية:

1. داخل **DNS Manager**، اضغط كليك يمين على **اسم السيرفر الرئيسي** (`PDC01`) في أعلى الشجرة ⬅️ اختر **Properties**.
2. انتقل إلى تبويب **Forwarders**.
3. اضغط على زر **Edit...**.
4. في القائمة، انقر لإضافة عناوين الـ IP التالية:
   * `8.8.8.8` (Google DNS)
   * `1.1.1.1` (Cloudflare DNS)
5. انتظر ثوانٍ معدودة حتى يقوم النظام بالتحقق (**OK / Validated**).
6. اضغط **OK** ثم **Apply**.

```
[Server Properties (PDC01)]
   └── [Forwarders Tab]
         └── [Edit...] ──► Add IPs: 8.8.8.8 , 1.1.1.1 ──► Validate ──► OK
```

---

## 🧪 LAB Task 4: Conditional Forwarders via GUI

### 🎯 الهدف:
إنشاء قاعدة توجيه شرطية لتوجيه طلبات نطاق الشركة الشريكة `partner.local` حصراً إلى سيرفرهم الخاص `10.10.20.5`.

### 📝 الخطوات العملية بالواجهة الرسومية:

1. داخل **DNS Manager**، ابحث في الشجرة اليسرى عن مجلد **Conditional Forwarders**.
2. اضغط كليك يمين عليه ⬅️ اختر **New Conditional Forwarder...**.
3. **DNS Domain:** اكتب اسم دومين الشركة الشريكة: `partner.local`.
4. **IP addresses of the master servers:** اضغط مرتين في جدول الـ IP واكتب: **`10.10.20.5`**.
5. انتظر تحكم السيرفر من الاتصال وتأكيد الـ Validation.
6. (اختياري) اختر فترة الـ Timeout بالثواني إذا لزم الأمر.
7. اضغط **OK**.

```
[Conditional Forwarders] ──(Right Click)──► New Conditional Forwarder...
     ├── Domain Name: partner.local
     └── Target Master Server IP: 10.10.20.5
```

---

## 🧪 LAB Task 5: Testing & Inspecting Resolution Mechanics (GUI + CMD Verification)

### 🎯 الهدف:
معاينة وعرض جدول الـ Cache، الـ Root Hints، واختبار آليات التحليل عبر أدوات الواجهة الرسومية وأمر `nslookup`.

### 📝 الخطوات العملية:

#### أ) عرض وفحص الـ Root Hints والـ Cache من الواجهة الرسومية:
1. **الاطلاع على Root Hints:**
   * اضغط كليك يمين على السيرفر `PDC01` ⬅️ **Properties** ⬅️ افتح تبويب **Root Hints**.
   * ستشاهد قائمة السيرفرات الجذرية الـ 13 العالمية (`a.root-servers.net` إلى `m.root-servers.net`).
2. **تفريغ معروضات الكاش (Clear Server Cache) عبر GUI:**
   * اضغط كليك يمين على اسم السيرفر `PDC01` ⬅️ اختر **Clear Cache**.
3. **إظهار البيانات المخزنة موقتاً (Advanced View):**
   * من القائمة العلوية لبرنامج DNS Manager اختر **View** ⬅️ تفعل خيار **Advanced**.
   * سيظهر لك مجلد جديد تحت السيرفر باسم **Cached Lookups** يعرض كافة النطاقات التي قام السيرفر بتخزينها مؤقتاً أثناء التصفح!

#### ب) اختبار تسلسل التحليل (End-to-End Test):

من جهاز العميل (`WIN-CLIENT01`) أو موجه الأوامر:

```cmd
:: 1. Testing FLZ (Internal Name Resolution)
nslookup appserver.test.local

:: 2. Testing RLZ (Reverse IP-to-Name Resolution)
nslookup 192.168.1.100

:: 3. Testing Conditional Forwarder (Partner Domain)
nslookup portal.partner.local

:: 4. Testing Global Forwarder (External Internet Query)
nslookup [www.google.com](https://www.google.com)
```

---

## 📊 GUI Troubleshooting & Verification Matrix

| Event / Goal                      | GUI Tool      | Menu Path / Click Sequence                                                     |
| :-------------------------------- | :------------ | :----------------------------------------------------------------------------- |
| **تفريغ سيرفر الكاش**             | `dnsmgmt.msc` | Right-click Server ➔ **Clear Cache**.                                          |
| **عرض المجلدات المتقدمة والكاش**  | `dnsmgmt.msc` | Top Menu: **View** ➔ Check **Advanced**.                                       |
| **إضافة IP للـ Global Forwarder** | `dnsmgmt.msc` | Right-click Server ➔ **Properties** ➔ **Forwarders** tab ➔ **Edit**.           |
| **تحديث سجل الـ PTR تلقائياً**    | `dnsmgmt.msc` | Inside A Record Properties ➔ Check **Update associated pointer (PTR) record**. |
| **فحص السيرفرات الجذرية**         | `dnsmgmt.msc` | Right-click Server ➔ **Properties** ➔ **Root Hints** tab.                      |


## ---
### **DNS 3: DNS Zones and Records: Types, Functions, and Configuration - explain**
***
---

مستند مرجعي شامل يغطي الجزء الثالث الممتد من سلسلة بروتوكول **DNS (Domain Name System)**. يوضح المستند بالتفصيل أنواع مناطق الـ DNS وتخزينها (Primary, Secondary, Stub, AD-Integrated)، آليات ونقل المناطق وتأمينها (**AXFR, IXFR, DNS Notify**)، وتفكيك هندسي لكافة أنواع السجلات (**SOA, NS, A, AAAA, CNAME, MX, SRV, PTR, TXT**).

---

## 🏗️ 1. أنواع مناطق الـ DNS وطرق تخزينها (DNS Zone Types & Storage Modes)

تُقسم المناطق (Zones) داخل سيرفر الـ DNS بناءً على صلاحيات التعديل ونمط التخزين والتكرار (Replication):

```
                               ┌───────────────────────────┐
                               │     DNS Server Zones      │
                               └─────────────┬─────────────┘
                                             │
      ┌──────────────────────────────┬───────┴──────────────────────┬──────────────────────────────┐
      ▼                              ▼                              ▼                              ▼
┌───────────┐                  ┌───────────┐                  ┌───────────┐                  ┌───────────┐
│  Primary  │                  │ Secondary │                  │ Stub Zone │                  │ AD-Integ. │
│ (Read/Write)                 │ (Read-Only)                  │(NS/SOA/A) │                  │(Multi-Mast)
└───────────┘                  └───────────┘                  └───────────┘                  └───────────┘
```

---

### 1️⃣ Primary Zone (المنطقة الأساسية - Standard Primary)
* **الصلاحية:** نسخة **قابلة للقراءة والتعديل (Read/Write)**.
* **مكان التخزين:** يُحفظ زون الـ DNS في ملف نصي عادي بصيغة `.dns` داخل المسار:
  `C:\Windows\System32\dns\`
* **طريقة العمل:** أي إضافة أو تعديل على السجلات يتم مباشرتها على هذا السيرفر حصراً.
* **العيب:** تُشكل نقطة فشل واحدة (**Single Point of Failure**) في حال توقف السيرفر عن العمل؛ حيث لا يمكن إجراء تعديلات جديدة من سيرفرات أخرى.

---

### 2️⃣ Secondary Zone (المنطقة الثانوية)
* **الصلاحية:** نسخة **للقراءة فقط (Read-Only)**.
* **طريقة الإنشاء:** يتم جلب البيانات والسجلات فيها عبر عملية نقل المنطقة (**Zone Transfer**) من السيرفر الأساسي (Primary).
* **الأهمية والوظيفة:**
  1. **موازنة الأحمال (Load Balancing):** توزيع طلبات الاستعلام بين السيرفر الأساسي والثانوي.
  2. **تحمل الأخطاء (Fault Tolerance):** ضمان استمرار الإجابة على استعلامات العملاء في حال انهيار السيرفر الأساسي.

---

### 3️⃣ Stub Zone (منطقة العينات / الجذور المستعرضة)
* **المفهوم:** منطقة خفيفة جداً لا تحتوي على كافة سجلات الدومين، بل تحتوي حصراً على السجلات الكفيلة بالتعرف على السيرفرات المرجعية لدومين آخر:
  * سجل **`SOA`** (Start of Authority).
  * سجلات **`NS`** (Name Servers).
  * سجلات **`A` (Glue Records)** الخاصة بالـ Name Servers.
* **حالات الاستخدام (Use Cases):**
  * **تسريع التحليل بين الشركات (Forest Trusts / Mergers):** بدلاً من نقل زون كامل بحجم ضخم بين شركتين عبر VPN، يتم إنشاء Stub Zone ليعرف سيرفرك السيرفر المباشر المسؤول عن الدومين الآخر والتواصل معه فوراً.
  * تحديث قائمة الـ Name Servers تلقائياً عند تغييرها في الدومين التابع دون الحاجة لتحديث يدوي.

---

### 4️⃣ Active Directory-Integrated Zone (المنطقة المدمجة مع الدومين)
* **المفهوم:** النمط المعياري والأكثر أماناً في بيئات **Active Directory Domain Services (AD DS)**.
* **مكان التخزين:** لا تُخزن في ملفات `.dns` نصية، بل تُحفظ كجزء من قاعدة بيانات الـ AD نفسها (`NTDS.dit`) داخل **Application Directory Partitions**.
* **الميزات التشغيلية:**
  * **Multi-Master Replication:** يمكنك الكتابة والتعديل من أي Domain Controller في الشبكة ويتم مزامنة التغييرات تلقائياً عبر آليات **AD Replication**.
  * **الأمان العالي (Secure Dynamic Updates):** يمنع أي جهاز غريب من تسجيل اسمه برمجياً في الـ DNS إلا بعد المصادقة عبر **Kerberos / Active Directory**.
  * **تشفير المزامنة:** يتم نقل التحديثات مشفرة تلقائياً عبر قنوات الـ AD الآمنة دون الحاجة لفتح أرقام الموانئ التقليدية للنقل غير المباشر.

---

## 🔄 2. آليات نقل المناطق وآليات المزامنة (Zone Transfer Mechanics)

عملية نقل المناطق هي الآلية التي يتم بها نسخ سجلات الـ DNS من سيرفر أساسي (Master) إلى سيرفر ثانوي (Secondary).

### 1️⃣ AXFR (Full Zone Transfer)
* **المفهوم:** نقل **شامل وكامل** لكافة قاعدة البيانات والـ Records داخل الزون.
* **متى يُستخدم؟**
  * عند إنشاء سيرفر ثانوي (Secondary Zone) لأول مرة.
  * عند حدوث الفشل في مزامنة الفروقات أو إعادة تشغيل الخدمة بشكل كامل.
* **البروتوكول والموائم:** يتم عبر بروتوكول **TCP Port 53** لضمان وصول كامل البيانات بدون مفقودات.

### 2️⃣ IXFR (Incremental Zone Transfer)
* **المفهوم:** نقل **الفروقات والتعديلات فقط** التي حدثت منذ آخر مزامنة.
* **طريقة العمل:** يربط السيرفر التعديلات بـ **SOA Serial Number**؛ إذا كان الرقم في السيرفر الأساسي أعلى من الثانوي، يُطلب نقل التغيرات المستحدثة فقط.
* **الميزة:** توفير استهلاك الباندويث (Bandwidth) وطاقة المعالجة بشكل هائل.

### 3️⃣ DNS Notify (التنبيه التلقائي)
* **المفهوم:** آلية إخطار فورية (Push Mechanism) ينبه فيها السيرفر الأساسي (Primary) السيرفرات الثانوية بوجود تحديث جديد.
* **خطوات العملية:**
  1. يقوم الأدمن بتعديل أو إضافة سجل في الـ Primary Server.
  2. يرتفع رقم الـ **Serial Number** داخل سجل الـ `SOA`.
  3. يرسل الـ Primary حزمة **DNS Notify** للسيرفرات الثانوية المحددة.
  4. تبدأ السيرفرات الثانوية فاعلية طلب **IXFR** فوراً لجلب التعديل.

---

### 🛡️ تأمين حركات نقل المناطق (Zone Transfer Hardening)

> ⚠️ **تحذير أمني (Security Vulnerability):**
> ترك خيار نقل المناطق مفتوحاً لأي سيرفر (**To Any Server**) يتيح للمخترقين تنفيذ هجوم **DNS Zone Transfer Reconnaissance** باستخدام أدوات مثل `dig axfr` واستخراج خريطة الشبكة الداخلية بالكامل!

#### 🔒 أفضل الممارسات لتأمين الـ Zone Transfer:
1. **Never Allow "To Any Server":** تعطيل النقل العام مطلقاً.
2. **Restrict to Explicit IPs:** السماح بالنقل حصراً لسيرفرات الـ NS المسجلة أو عناوين IP محددة ومعروفة للسيرفرات الثانوية (`Only to servers listed on the Name Servers tab` أو `Only to the following IP addresses`).

---

## 📑 3. تفكيك أنواع سجلات الـ DNS (DNS Record Types Deep Dive)

```
        ┌────────────────────────────────────────────────────────┐
        │                 Common DNS Record Types                │
        └───────────────────────────┬────────────────────────────┘
                                    │
     ┌──────────┬──────────┬────────┴─┬──────────┬──────────┬──────────┐
     ▼          ▼          ▼          ▼          ▼          ▼          ▼
  ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐
  │ SOA │    │ NS  │    │A/AAAA   │ CNAME│    │ MX  │    │ SRV │    │ TXT │
  └─────┘    └─────┘    └─────┘    └─────┘    └─────┘    └─────┘    └─────┘
```

### 1️⃣ `SOA` Record (Start of Authority)
السجل القيادي والأول داخل أي Zone، ويحتوي على بيانات الإدارة والتحكم في الزون:
* **Primary Name Server:** اسم السيرفر المرجع الرئيسي.
* **Responsible Person:** إيميل مسؤول النظام (تُكتب `.` بدلاً من `@` مثل: `admin.test.local`).
* **Serial Number:** رقم إصداري يتغير مع كل تعديل (يتبع عادة صيغة التاريخ: `YYYYMMDDNN`).
* **Refresh Interval:** المدة التي ينتظرها السيرفر الثانوي قبل طلب تحديث البيانات.
* **Retry Interval:** وقت إعادة المحاولة في حال فشل الاتصال بالـ Primary.
* **Expire Limit:** المدة التي يظل فيها السيرفر الثانوي يعطي إجابات قبل اعتبار البيانات ملغاة وغير صالحة عند انقطاع الـ Primary.
* **Minimum TTL:** وقت الحياة المؤقت الافتراضي للسجلات السالبة (Negative Caching).

---

### 2️⃣ `NS` Record (Name Server)
* **الوظيفة:** يحدد الخوادم ذات السلطة (**Authoritative Servers**) المسؤولة عن تقديم الإجابات الخاصة بهذا النطاق.

### 3️⃣ `A` & `AAAA` Records
* **`A` Record (Address):** ربط اسم الجهاز بـ **IPv4** (مثال: `host1.test.local -> 192.168.1.50`).
* **`AAAA` Record (Quad-A):** ربط اسم الجهاز بـ **IPv6** (مثال: `host1.test.local -> fe80::1`).

### 4️⃣ `CNAME` Record (Canonical Name / Alias)
* **الوظيفة:** ربط اسم مستعار باسم حقيقي آخر (Host to Host Mapping).
* *مثال:* توجيه `www.test.local` و `ftp.test.local` ليشيرا كلاهما إلى الاسم الحقيقي للسيرفر `webserver01.test.local`.

---

### 5️⃣ `MX` Record (Mail Exchanger)
* **الوظيفة:** توجيه البريد الإلكتروني الوارد للدومين إلى سيرفرات الـ Mail Servers الخاصة بالشركة.
* **مفهوم الأولوية (Preference / Priority Value):**
  * يتم إسناد رقم أولوية لكل سيرفر بريد (الرقم الأقل يعني الأولوية الأعلى).
  * *مثال:* 
    * `mail1.company.com` (Priority: 10) 👈 المستهدف الأول.
    * `mail2.company.com` (Priority: 20) 👈 سيرفر احتياطي في حال تداعي الأول.

---

### 6️⃣ `SRV` Record (Service Location Record)
* **الوظيفة:** تمكين الأجهزة والعملاء من اكتشاف مواقع خدمات معينة على الشبكة (البروتوكول، المنفذ، والسيرفر) دون معرفة الـ IP المباشر.
* **صيغة السجل القياسية:** `_Service._Protocol.Domain`
* **أهميته الكبرى في الـ Active Directory:**
  تعتمد الأجهزة للانضمام وتسجيل الدخول للدومين على سجلات الـ SRV للوصول لخدمات الـ **LDAP** والـ **Kerberos**:
  `_ldap._tcp.dc._msdcs.test.local` (Port 389)
  `_kerberos._tcp.dc._msdcs.test.local` (Port 88)

---

### 7️⃣ `PTR` Record (Pointer Record)
* **الوظيفة:** يعمل داخل منطقة البحث العكسي (**Reverse Lookup Zone**) لترجمة عنوان الـ IP إلى اسم الجهاز (FQDN).

---

### 8️⃣ `TXT` Record (Text Record)
* **الوظيفة:** تخزين نصوص وصفية قابلة للقراءة آلياً، وتُستخدم أساساً في آليات حماية وتأمين النطاقات والبريد:
  * **SPF (Sender Policy Framework):** تحديد الـ IPs المسموح لها بإرسال إيميلات باسم الدومين لمنع الـ Spoofing.
  * **DKIM (DomainKeys Identified Mail):** إرفاق مفتاح توقيع رقمي للتحقق من سلامة رسائل البريد.
  * **DMARC:** تحديد سياسة التعامل مع الإيميلات التي تفشل في اختبارات SPF و DKIM.

---



### **DNS 4: DNS Zone types and DNS Records - LAB** 
***
---

دليل عملي معملي مخصص للواجهة الرسومية (**GUI-Only Practical LAB**) يغطي كيفية إنشاء وإدارة أنواع مناطق الـ DNS المختلفة (**Primary, AD-Integrated, Secondary, Stub**)، تأمين نقل المناطق ضد هجمات الاستطلاع (**AXFR Hardening**)، وإنشاء وإدارة كافة السجلات المتقدمة (**SOA, SRV, MX, TXT/SPF**) عبر برنامج **DNS Manager (`dnsmgmt.msc`)**.

---

## 🛠️ 1. مخطط البيئة المعملية (Lab Setup Topology)

* **Primary DC / DNS Master (`DC01`):**
  * **IP Address:** `192.168.1.10 /24`
  * **Domain Name:** `corp.local`
  * **OS:** Windows Server (Domain Controller / Active Directory)
* **Secondary DNS Server (`DNS02`):**
  * **IP Address:** `192.168.1.11 /24`
  * **OS:** Windows Server (Member Server)
* **Partner Network DNS (`DNS-PARTNER`):**
  * **Target Network:** `partner.local`
  * **IP Address:** `10.10.20.5`

---

## 🧪 LAB Task 1: Creating Zone Types via GUI Wizard

### 🎯 الهدف:
التعرف على كيفية إنشاء وفهم الفروق العملية بين أنواع الـ Zones المختلفة باستخدام **New Zone Wizard**.

### 📝 الخطوات العملية بالواجهة الرسومية:

#### أ) إنشاء Standard Primary Zone (تخزين في ملف نصي):
1. افتح **DNS Manager**: اضغط `Win + R` واكتب **`dnsmgmt.msc`**.
2. وسّع اسم السيرفر `DC01` ⬅️ اضغط كليك يمين على مجلد **Forward Lookup Zones** ⬅️ اختر **New Zone...**.
3. اضغط **Next** في شاشة الترحيب.
4. **Zone Type:** اختر **Primary zone** (قم بإزالة علامة الصح `☐` عن خيار *Store the zone in Active Directory* مؤقتاً) ⬅️ اضغط **Next**.
5. **Zone Name:** اكتب `standalone.local` ⬅️ اضغط **Next**.
6. **Zone File:** اتركه افتراضياً (`standalone.local.dns`) ⬅️ اضغط **Next** ثم **Finish**.

```
[Forward Lookup Zones] ──(Right Click)──► New Zone...
     ├── Type: Primary zone (Uncheck AD-Integrated)
     ├── Name: standalone.local
     └── File: standalone.local.dns  ──► (Stored in C:\Windows\System32\dns)
```

---

#### ب) إنشاء Active Directory-Integrated Zone (تخزين داخل قاعدة البيانات):
1. داخل **DNS Manager**، اضغط كليك يمين على مجلد **Forward Lookup Zones** ⬅️ اختر **New Zone...**.
2. **Zone Type:** اختر **Primary zone** وثبّت علامة الصح `☑️` على خيار:
   **`Store the zone in Active Directory (available only if domain controller is a domain controller)`**.
3. **Active Directory Zone Replication Scope:** اختر **To all DNS servers running on domain controllers in this domain** ⬅️ اضغط **Next**.
4. **Zone Name:** اكتب `adcorp.local` ⬅️ اضغط **Next**.
5. **Dynamic Update:** اختر **Allow only secure dynamic updates (recommended for Active Directory)** ⬅️ اضغط **Next** ثم **Finish**.

---

#### ج) إنشاء Secondary Zone على السيرفر الثانوي (`DNS02`):
1. افتح **DNS Manager** على السيرفر الثانوي `DNS02`.
2. اضغط كليك يمين على مجلد **Forward Lookup Zones** ⬅️ اختر **New Zone...**.
3. **Zone Type:** اختر **Secondary zone** ⬅️ اضغط **Next**.
4. **Zone Name:** اكتب اسم الزون المراد نسخها: `corp.local` ⬅️ اضغط **Next**.
5. **Master DNS Servers:** اكتب IP السيرفر الرئيسي `192.168.1.10` ⬅️ اضغط **Enter** وانتظر ظهور العلامة الخضراء `✔️` ⬅️ اضغط **Next** ثم **Finish**.

---

#### د) إنشاء Stub Zone للربط مع شركة شريكة:
1. على السيرفر `DC01` داخل **DNS Manager**، اضغط كليك يمين على **Forward Lookup Zones** ⬅️ اختر **New Zone...**.
2. **Zone Type:** اختر **Stub zone** ⬅️ اضغط **Next**.
3. **Zone Name:** اكتب `partner.local` ⬅️ اضغط **Next**.
4. **Master DNS Servers:** اكتب IP سيرفر الشركة الشريكة `10.10.20.5` ⬅️ اضغط **Next** ثم **Finish**.
5. افتح الزون المستحدثة `partner.local` لتشاهد اقتصار البيانات المنقولة على سجلات `SOA`, `NS`, وسجلات הـ `A` الخاصة بالسيرفر المرجعي فقط!

---

## 🧪 LAB Task 2: Securing Zone Transfers (AXFR Hardening) via GUI

### 🎯 الهدف:
حماية السيرفر الأساسي من هجمات استطلاع الخريطة الداخلية (**DNS Reconnaissance**) عبر تقييد صلاحية نقل الزون من شاشة الخصائص.

### 📝 الخطوات العملية بالواجهة الرسومية:

1. داخل **DNS Manager** على `DC01`، اضغط كليك يمين على الزون `corp.local` ⬅️ اختر **Properties**.
2. افتح تبويب **Zone Transfers**:
   * ضع علامة صح `☑️` على خيار **Allow zone transfers**.
   * حدد الخيار الثاني: **Only to servers listed on the Name Servers tab** (أو حدد *Only to the following servers* وأضف الـ IP المباشر `192.168.1.11`).
3. اضغط على تبويب **Name Servers**:
   * تأكد من إضافة السيرفر الثانوي `DNS02` برقم الـ IP الخاص به في القائمة.
4. اضغط **Apply** ثم **OK**.

```
[corp.local Properties]
   └── [Zone Transfers Tab]
         ├── ☑️ Allow zone transfers
         └── 🔘 Only to servers listed on the Name Servers tab
```

---

## 🧪 LAB Task 3: Configuring Advanced Records via GUI

### 🎯 الهدف:
إدارة وتعديل السجلات المتقدمة (**SOA, SRV, MX, TXT/SPF**) باستخدام القوائم والنوافذ الرسومية.

### 📝 الخطوات العملية بالواجهة الرسومية:

#### 1️⃣ تعديل إعدادات الـ `SOA` Record (Start of Authority):
1. اضغط كليك يمين على الزون `corp.local` ⬅️ اختر **Properties**.
2. افتح تبويب **Start of Authority (SOA)**:
   * **Serial number:** اضغط على زر **Increment** لرفع الرقم التسلسلي يدوياً وإجبار الثانوية على المزامنة.
   * **Primary server:** حدد اسم السيرفر الرئيسي `dc01.corp.local.`.
   * **Responsible person:** حدد إيميل المسؤول (مثل `admin.corp.local.`).
   * عدّل قيم الـ **Refresh interval**, **Retry interval**, و **Expires after** حسب الحاجة.
3. اضغط **Apply**.

---

#### 2️⃣ إنشاء سجل بريد `MX` مع الأولوية (Mail Exchanger):
1. افتح الزون `corp.local` ⬅️ اضغط كليك يمين في المساحة الفارغة ⬅️ اختر **New Mail Exchanger (MX)...**.
2. **Host or child domain:** اتركها فارغة (أو اكتب `@` للشجرة الرئيسية).
3. **Fully qualified domain name (FQDN) of mail server:** اكتب `mail1.corp.local`.
4. **Mail server priority:** اكتب الرقم **`10`** (الأولوية العالية) ⬅️ اضغط **OK**.
5. كرري الخطوة لسيرفر الاحتياط:
   * FQDN: `mail2.corp.local`
   * Mail server priority: **`20`** (أولوية أقل) ⬅️ اضغط **OK**.

---

#### 3️⃣ إنشاء سجل اكتشاف الخدمات `SRV` (Service Location):
1. داخل الزون `corp.local`، اضغط كليك يمين في المساحة الفارغة ⬅️ اختر **Other New Records...**.
2. من القائمة، اختر **Service Location (SRV)** ⬅️ اضغط **Create Record...**.
3. قم بتعبئة الحقول التالية:
   * **Service:** اكتب `_ldap`
   * **Protocol:** اختر `_tcp`
   * **Port number:** اكتب `389`
   * **Host offering this service:** اكتب `dc01.corp.local`
   * **Priority:** `0` | **Weight:** `100`
4. اضغط **OK** ثم **Done**.

---

#### 4️⃣ إنشاء سجل الأمان `TXT` لمنع تزوير البريد (SPF Record):
1. اضغط كليك يمين في المساحة الفارغة داخل الزون `corp.local` ⬅️ اختر **Other New Records...**.
2. اختر نوع السجل **Text (TXT)** ⬅️ اضغط **Create Record...**.
3. **Record name:** اتركه فارغاً ليعبر عن الزون الرئيسي.
4. **Text:** اكتب نص سياسة الـ SPF المعيارية:
   `v=spf1 ip4:192.168.1.10 ip4:192.168.1.50 -all`
5. اضغط **OK** ثم **Done**.

```
[Resource Record Types Window]
   ├── Select: Service Location (SRV) ──► (For LDAP / Kerberos discovery)
   └── Select: Text (TXT)             ──► (For SPF / Email Security Policy)
```

---

## 📊 GUI Verification & Troubleshooting Guide

| Issue / Goal | GUI Navigation Sequence | Verification Step |
| :--- | :--- | :--- |
| **التحقق من سجلات الـ SRV للـ AD** | `dnsmgmt.msc` ➔ `corp.local` ➔ `_msdcs` folder | تأكد من وجود مجلدات `_tcp` و `_udp` الممتلئة بسجلات الـ SRV. |
| **إجبار المزامنة إلى السيرفر الثانوي** | Right-click Zone ➔ **Properties** ➔ **SOA** tab | اضغط **Increment** لرفع الـ Serial Number ثم انقر **Apply**. |
| **تغيير نوع الزون لـ AD-Integrated** | Right-click Zone ➔ **Properties** ➔ **General** tab | اضغط **Change...** بجانب Type واستعر خيار **Store in AD**. |
| **معاينة سجلات ה- Stub Zone** | Open `partner.local` Zone | تحقق من وجود سجلات `SOA`, `NS`, و `A` الخاصة بالسيرفرات فقط. |



***
 **DNS 5: DNS Zone transfer and offline restore DNS Zone- LAB**
***
***




دليل عملي معملي مخصص للواجهة الرسومية (**GUI-Only Practical LAB**) يغطي إدارة وتأمين نقل المناطق (**Zone Transfers**)، أخذ النسخ الاحتياطية، وتطبيق خطة التعافي من الكوارث (**Offline Restore**) لاستعادة مناطق الـ DNS المحذوفة عبر **DNS Manager Wizard** ومتصفح الملفات.

---

## 🛠️ 1. الأدوات المستخدمة في الواجهة الرسومية (GUI Tools)

* **DNS Manager:** (`dnsmgmt.msc`) لإدارة السجلات والـ Zones والـ Forwarders.
* **Services Console:** (`services.msc`) لإيقاف وتشغيل خدمة الـ DNS Server.
* **Windows File Explorer:** (`explorer.exe`) للتعامل مع ملفات النسخ الاحتياطية `.dns`.

---

## 🧪 LAB Task 1: Managing Zone Transfers & DNS Notify via GUI

### 🎯 الهدف:
تفعيل المزامنة بين السيرفر الرئيسي والسيرفر الثانوي وتأمينها، وإرسال تنبيهات فورية عند التعديل عبر واجهة **DNS Manager**.

### 📝 الخطوات العملية:

#### أ) تهيئة الـ Zone Transfer والتنبيهات (على السيرفر الرئيسي `DC01`):
1. افتح **DNS Manager** عبر الخيار: `Start` ⬅️ `Administrative Tools` ⬅️ `DNS` (أو اضغط `Win + R` واكتب `dnsmgmt.msc`).
2. وسّع السيرفر ⬅️ مجلد **Forward Lookup Zones**.
3. اضغط كليك يمين على الزون الخاص بك `corp.local` ⬅️ اختر **Properties**.
4. افتح تبويب **Zone Transfers**:
   * ضع علامة صح `☑️` على خيار **Allow zone transfers**.
   * حدد خيار **Only to the following servers** لتقييد النقل أمنياً.
   * اضغط على زر **Edit...** وأضف عنوان IP الخاص بالسيرفر الثانوي (`192.168.1.11`) ثم اضغط **OK**.
5. اضغط على زر **Notify...** أسفل التبويب نفسه:
   * ضع علامة صح `☑️` على خيار **Automatically notify**.
   * اختر **The following servers** وأضف IP السيرفر الثانوي `192.168.1.11`.
   * اضغط **OK** ثم **Apply**.

```
[corp.local Properties]
   └── [Zone Transfers Tab]
         ├── ☑️ Allow zone transfers ──► (Only to the following servers)
         └── [Notify...] ──────────────► ☑️ Automatically notify (192.168.1.11)
```

#### ب) طلب المزامنة يدوياً من السيرفر الثانوي (`DNS02`):
1. افتح **DNS Manager** على السيرفر الثانوي `DNS02`.
2. وسّع مجلد **Forward Lookup Zones** واضغط كليك يمين على الزون الثانوية `corp.local`.
3. اختر أحد الخيارين حسب نوع المزامنة المطلوب:
   * **Transfer from Master:** لإجراء نقل تزايدي (**IXFR**).
   * **Reload from Master:** لإعادة سحب الزون بالكامل وإجبار النقل الشامل (**AXFR**).

---

## 🧪 LAB Task 2: Manual Backup / Exporting Zone Files via GUI

### 🎯 الهدف:
أخذ نسخة احتياطية من ملفات الـ Zone النصية باستخدام متصفح الملفات (Windows File Explorer).

### 📝 الخطوات العملية:

1. افتح **File Explorer** وتوجه إلى المسار التالي الذي تُحفظ فيه ملفات الـ DNS افتراضياً:
   `C:\Windows\System32\dns\`
2. ابحث عن الملف الذي يحمل اسم الزون الخاص بك: `corp.local.dns`.
3. اضغط كليك يمين على الملف ⬅️ اختر **Copy**.
4. انشئ مجلد جديد باسم `backup` داخل نفس المسار أو في أي مكان آمن، وقم بعمل **Paste** للملف مع إعادة تسميته إلى:
   `corp.local.dns.bk`

---

## 🧪 LAB Task 3: Disaster Recovery - Offline Restore via GUI Wizard

### 🎯 سيناريو الكارثة:
قام أحد المستخدمين بحذف الزون `corp.local` بالخطأ من واجهة الـ DNS Manager، وتوقفت الخدمة. المطلوب: **إعادة بناء الزون باستخدام خيار "Use Existing File" في الـ Wizard**.

### 📝 الخطوات العملية للاستعادة (Step-by-Step GUI Wizard):

#### الخطوة 1: إيقاف خدمة الـ DNS
1. افتح نافذة الخدمات: `Win + R` ⬅️ اكتب `services.msc`.
2. ابحث عن خدمة **DNS Server**.
3. اضغط عليها كليك يمين ⬅️ اختر **Stop**.

#### الخطوة 2: تجهيز ملف النسخة الاحتياطية
1. افتح **File Explorer** وانتقل لمجلد النسخ الاحتياطية `C:\Windows\System32\dns\backup\`.
2. انسخ ملف `corp.local.dns.bk`.
3. انقله مباشرة إلى المجلد الرئيسي `C:\Windows\System32\dns\` وقم بتغيير اسمه ليصبح بالضبط:
   `corp.local.dns`

#### الخطوة 3: إعادة تشغيل خدمة الـ DNS
1. اعد إلى نافذة `services.msc`.
2. اضغط كليك يمين على **DNS Server** ⬅️ اختر **Start**.

#### الخطوة 4: تحميل الزون عبر New Zone Wizard
1. افتح برنامج **DNS Manager** (`dnsmgmt.msc`).
2. اضغط كليك يمين على مجلد **Forward Lookup Zones** ⬅️ اختر **New Zone...**.
3. في شاشة الترحيب بـ Wizard، اضغط **Next**.
4. **Zone Type:** اختر **Primary zone** (تأكد من إزالة العلامة عن Store in Active Directory مؤقتاً) ⬅️ اضغط **Next**.
5. **Zone Name:** اكتب اسم الزون المفقود بدقة: `corp.local` ⬅️ اضغط **Next**.
6. **Zone File (النقطة الأهم):**
   * حدد الخيار الثاني: **`Use this existing file`**.
   * تأكد أن اسم الملف الظاهر هو `corp.local.dns` (وهو الملف الذي وضعناه في الخطوة 2).
   * اضغط **Next**.
7. **Dynamic Update:** اختر سياسة التحديث المناسبة (مثل *Do not allow dynamic updates*) ⬅️ اضغط **Next**.
8. اضغط **Finish**.

```
[New Zone Wizard]
   ├── 1. Zone Type: Primary zone
   ├── 2. Zone Name: corp.local
   ├── 3. Zone File: ☑️ Use this existing file (corp.local.dns)
   └── 4. Finish ──► (All records restored automatically)
```

#### 🔍 التحقق:
افتح الزون `corp.local` داخل **DNS Manager**، ستلاحظ عودة جميع السجلات السابقة فوراً وبشكل كلي دون الحاجة لإعادة إدخال أي سجل يدوياً!

---

## 🧪 LAB Task 4: Converting Restored Zone to AD-Integrated via GUI

### 🎯 الهدف:
بعد استعادة الزون كـ Standard Primary Zone من الملف النصي، تحويلها إلى **Active Directory-Integrated Zone** لتتم مزامنتها تلقائياً عبر قاعدة بيانات الـ AD.

### 📝 الخطوات العملية:

1. داخل **DNS Manager**، اضغط كليك يمين على الزون المستعادة `corp.local` ⬅️ اختر **Properties**.
2. في تبويب **General**، ابحث عن حقل **Type: Primary**.
3. اضغط على زر **Change...** المجاور له.
4. ضع علامة صح `☑️` على الخيار:
   **`Store the zone in Active Directory (available only if domain controller is a domain controller)`**.
5. اضغط **OK**.
6. ستظهر لك نافذة تأكيد تسألك عن نطاق المزامنة (Replication Scope):
   * اختر **To all DNS servers running on domain controllers in this domain**.
7. اضغط **OK** ثم **Apply**.

---

## 📊 GUI Navigation & Action Matrix

| Task / Goal | GUI Tool | Menu Path / Click Sequence |
| :--- | :--- | :--- |
| **تفعيل Zone Transfers** | `dnsmgmt.msc` | Right-click Zone ➔ Properties ➔ Zone Transfers tab ➔ Check "Allow zone transfers". |
| **إيقاف/تشغيل الخدمة** | `services.msc` | Right-click "DNS Server" ➔ Stop / Start. |
| **نسخ ملف البيانات** | `explorer.exe` | Open `C:\Windows\System32\dns\` ➔ Copy/Paste `.dns` file. |
| **تحميل الزون أوفلاين** | `dnsmgmt.msc` | Forward Lookup Zones ➔ New Zone ➔ Primary ➔ **Use this existing file**. |
| **تحويل إلى AD-Integrated** | `dnsmgmt.msc` | Right-click Zone ➔ Properties ➔ General tab ➔ Type: Change ➔ Check "Store in AD". |