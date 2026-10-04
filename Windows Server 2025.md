Shortcut RUN**
* open card network    `ncpa.cpl`
* open to change computer name  `sysdm.cpl`

**Windows Server initial Configurations - LAB** 
`Steps `
* change date , time and time zone . 
* IP configurations manual and disable IPV6   `The server IP address should preferably be entered manually.`
* Change Computer Name  

**Add (ADDS Role)** and configure Role - شرح و تطبيق عملي لكيفية انشاء دومين 

س / أمتي اقول علي جهاز Domain Controller ؟
* Windows Server is installed on it.
* Added to the roll called ADDS
*  configuration ADDS
---
س / الفرق ما بين Role , Features ؟ 

* Role وظيفة بيقوم بيها الويندوز سيرفر للنيتورك كلها 
* Features هي **الإمكانيات أو الوظائف** التي يوفرها منتج أو نظام أو تطبيق

---
**How to Install ADDS** 

`steps`
* **open Server Manager** > Dashboard 
* choose **`Add roles and features`**
* **`Before You Begin`** Click `Next`
* **`Installation Type`** > choose `Role-based or feature-based installation` > `Next`
* **`Server Selection`** choose `Server` > `Next`
* **`Server Roles`** choose `Active Directory Domain Services` > after choose will open automatically `Add Roles and Features Wizard` > click `Add Features` > click `Next`
* **`Features`** Click `Next`
* **`AD DS`** > Click `Next`
* `Confirmation` > Click `install`
* `Results` after Features installation is Done
* restart windows server
*  open `Server Manager` > Click to `Manage`  choose > `Promote this server to a domain controller`
****
**`Active Directory Domain Services Configuration Wizard`**
* **`Deployment Configurations`** > choose `Add a new forest` > `Root domain name: company.local` > Click `Next`
* **`Domain Controller Options`** > `Forest functional level` : Windows Server 2025 and `Domain functional level`: Windows Server 2025 > create password DSRM > click `Next` 
* **`DNS Options`** > Click `Next`
* `Additional Options`  > Click `Next`
* **`Paths`** > Click `Next`
* `Review Options` > Click `Next`
* `Prerequisites Check` > checks passed successfully > Click `Install` 
****
**`Forest functional level`** > The minimum version of Windows Server that **Forest** accepts as a domain controller

### NetBIOS Domain Name in Active Directory

هو اسم تعريفي قصير (مثل `TEST`) يتم إنشاؤه تلقائياً أثناء إعداد الـ Domain Controller، وله أهميتان رئيسيتان:

1. **صيغة تسجيل الدخول القديمة (Legacy Login):** يُستعمل لتمكين المستخدمين من تسجيل الدخول بالصيغة المختصرة `DOMAIN\username` (مثال: `TEST\Administrator`).
2. **التوافقية (Backward Compatibility):** يضمن قدرة التطبيقات والأنظمة القديمة (Legacy APIs) التي لا تدعم الـ DNS على التعرف على النطاق والاتصال به.

> 📌 **خلاصة:** رغم أن الشبكات الحديثة تعتمد كلياً على الـ DNS لترجمة الأسماء، إلا أن Windows Server لا يزال يفرضه كمتطلب أساسي لضمان التوافقية وإتاحة صيغة الدخول المختصرة.

---
## Active Directory Core Components

### 1. Active Directory Database (`ntds.dit`)
الملف الفيزيائي الفعلي الذي يحتوي على كافة كائنات وبيانات النطاق (Domain).
* **المسار الافتراضي:** `C:\Windows\NTDS\`
* **المحتوى:** بيانات المستخدمين، الحسابات، الأجهزة، والمجموعات، بالإضافة إلى كلمات المرور المشفرة (Password Hashes).

### 2. NTDS (Active Directory Domain Services)
هو النظام والخدمة الكاملة المسؤولين عن إدارة ملف قاعدة البيانات `ntds.dit` ومعالجة العمليات الخاصة به.
* **ملفات السجلات (Transaction Logs - `edb.log`):** تُكتب التعديلات فيها أولاً في الذاكرة لضمان سلامة البيانات قبل ترحيلها لقاعدة البيانات.
* **الوظيفة:** معالجة استعلامات LDAP، طلبات التحقق من الهوية (Authentication)، وإدارة المزامنة (Replication) بين الـ Domain Controllers.

### 3. SYSVOL (System Volume)
مجلد مشترك (Shared Folder) موجود على كل Domain Controller يُستخدم لتخزين الملفات الفعليه التي يجب مزامنتها عبر الشبكة.
* **المسار الافتراضي:** `C:\Windows\SYSVOL\`
* **المحتوى:** سكريبتات تسجيل الدخول والخروج (Logon/Logoff Scripts) وملفات وقوالب سياسات المجموعة (Group Policy Objects - GPOs).
* **آلية المزامنة:** يتم نسخ محتوياته بين السيرفرات باستخدام بروتوكول **DFSR**.

---

### 📊 ملخص مقارنة سريع

| وجه المقارنة | `ntds.dit` (قاعدة البيانات) | SYSVOL (المجلد المشترك) |
| :--- | :--- | :--- |
| **نوع البيانات** | قاعدة بيانات مهيكلة (Objects) | ملفات ومجلدات (File System) |
| **المحتوى الأساسي** | المستخدمين، الهاشات، المجموعات | ملفات الـ GPOs، سكريبتات الـ Logon |
| **بروتوكول الوصول** | LDAP | SMB |
| **آلية المزامنة** | RPC over IP | DFSR |

---
## الفرق بين Windows Server Standard و Datacenter

الاختلاف الرئيسي بين النسختين لا يكمن في الخصائص الأساسية للنظام، بل في **حقوق الافتراضية (Virtualization Rights)** وبعض الميزات المتقدمة الخاصة بمراكز البيانات الضخمة.

### 1. بيئة الافتراضية (Virtualization)
* **Standard Edition:** تمنحك الحق في تشغيل **جهازين افتراضيين فقط (2 VMs)** أو حاويتين (Hyper-V containers) لكل ترخيص. إذا أردت تفعيل المزيد من الـ VMs، ستحتاج لشراء تراخيص إضافية لنفس السيرفر.
* **Datacenter Edition:** تمنحك الحق في تشغيل **عدد غير محدود من الأجهزة الافتراضية (Unlimited VMs)** على نفس السيرفر الفيزيائي المرخص.

### 2. ميزات التخزين والشبكات المتقدمة (المدعومة في Datacenter فقط)
تنفرد نسخة **Datacenter** بميزات متطورة لبناء بنية تحتية سحابية ومحمية:
* **Storage Spaces Direct (S2D):** تجميع الأقراص الصلبة الداخلية للسيرفرات المختلفة لإنشاء وحدات تخزين مشتركة وعالية التوفر (Highly Available) بتكلفة منخفضة.
* **Software-Defined Networking (SDN):** إدارة وتكوين الشبكات الفيزيائية والافتراضية مركزيًا عبر برمجيات.
* **Storage Replica:** ميزة لنسخ البيانات احتياطيًا وبشكل متماثل بين السيرفرات للحماية من الكوارث (Disaster Recovery).

### 3. الحماية والأمان المتقدم (Shielded VMs)
* **Datacenter Edition:** تدعم خاصية **Shielded Virtual Machines**، وهي تقنية تقوم بتشفير الأجهزة الافتراضية لحمايتها من الوصول غير المصرح به، حتى من قِبل مدير السيرفر الفيزيائي نفسه (Hyper-V host administrator).

---
### 📊 جدول مقارنة سريع

| وجه المقارنة                    | Windows Server Standard                           | Windows Server Datacenter                        |
| :------------------------------ | :------------------------------------------------ | :----------------------------------------------- |
| **عدد الـ VMs المسموحة**        | 2 فقط (لكل ترخيص)                                 | عدد غير محدود (Unlimited)                        |
| **الاستخدام المثالي**           | البيئات الصغيرة والمتوسطة ذات الافتراضية المحدودة | البيئات الضخمة، مراكز البيانات، والسحب الـ Cloud |
| **Storage Spaces Direct**       | ❌ غير مدعومة                                      | مدعومة                                           |
| **Software-Defined Networking** | ❌ غير مدعومة                                      | مدعومة                                           |
| **Shielded VMs**                | ❌ غير مدعومة                                      | مدعومة                                           |

---
## الفرق بين Domain و Domain Controller (DC)

باختصار شديد: الـ **Domain** هو "الحدود أو المدينة الفرضية"، بينما الـ **Domain Controller** هو "مبنى إدارة هذه المدينة".

### 1. الـ Domain (النطاق)
* **المفهوم:** هو عبارة عن **حدود منطقية (Logical Boundary)** تجمع تحتها كل عناصر الشبكة.
* **الوظيفة:** يربط الأجهزة، المستخدمين، المجموعات، والطابعات معاً تحت اسم إدارة واحد (مثل `TEST.local`) لتسهيل إدارتهم مركزياً بدلاً من إدارة كل جهاز بشكل منفصل.
* **طبيعة وجوده:** هو مفهوم تنظيمي (وليس سيرفر أو جهاز حقيقي).

### 2. الـ Domain Controller - DC (متحكم النطاق)
* **المفهوم:** هو **السيرفر الفيزيائي أو الافتراضي (Server)** الفعلي الذي يُمثّل قلب النطاق.
* **الوظيفة:** هو المسؤول عن تشغيل خدمة Active Directory، ويحتوي على قاعدة البيانات (`ntds.dit`). يقوم بالتحقق من هويات المستخدمين (Authentication) والسماح لهم بالوصول للموارد بناءً على الصلاحيات (Authorization).
* **طبيعة وجوده:** هو خادم (Server) حقيقي يعمل بنظام Windows Server وتمت ترقيته ليلعب هذا الدور.

---

### 📊 ملخص سريع

* **الـ Domain:** هو الشبكة أو النطاق التنظيمي نفسه (المكان).  `logical expression`
* **الـ Domain Controller:** هو السيرفر (الكمبيوتر) المحرك والمدير لهذا النطاق (المدير).  `physical expression`
