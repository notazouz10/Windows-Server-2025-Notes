***
### **Security Permissions, Disk Quota, Shadow Copy - explain**  
***


مستند مرجعي نظري يغطي الركائز الثلاث الأساسية لإدارة وحماية وسائط التخزين في بيئات Windows Server: نظام صلاحيات الملفات المتقدم، التحكم في استهلاك المساحات، وآلية استعادة البيانات عبر اللقطات الفورية.

---

## 🛡️ 1. Security Permissions (NTFS / ReFS Permissions)

### 🔹 ما هي صلاحيات الأمان؟
هي الصلاحيات التي يتم تطبيقاها مباشرة على مستوى نظام الملفات (**NTFS / ReFS**) للتحكم في وصول المستخدمين للملفات والمجلدات، سواء تم الوصول إليها **محلياً (Locally)** أو **عبر الشبكة (Network)**.

### ⚙️ مستويات الصلاحيات القياسية (Standard Permissions):

| الصلاحية | Permission | المسموح به للمستخدم |
| :--- | :--- | :--- |
| **قراءة** | **Read** | عرض محتوى الملفات وقراءة الخصائص واستعراض المجلدات. |
| **كتابة** | **Write** | إنشاء ملفات ومجلدات جديدة والتعديل على محتوى الملفات الحالية. |
| **قراءة وتشغيل** | **Read & Execute** | صلاحيات Read + القدرة على تشغيل البرامج والملفات التنفيذية (`.exe`, `.bat`). |
| **عرض محتويات المجلد**| **List Folder Contents**| مخصصة للمجلدات فقط: استعراض أسماء الملفات والمجلدات الفرعية وتنفيذ التصفح. |
| **تعديل** | **Modify** | تشمل Read & Execute + Write + حذف الملفات والمجلدات. |
| **تحكم كامل** | **Full Control** | تشمل Modify + تغيير جدول الصلاحيات والأمان (Security Tab) وتملك الملف (Take Ownership). |

### 🧠 القواعد الحاكمة لصلاحيات الأمان:
1. **التوريث (Inheritance):** بشكل افتراضي، ترث المجلدات الفرعية والملفات نفس الصلاحيات المطبقة على المجلد الأب (Parent Folder).
2. **المنع الصريح (Explicit Deny):** قاعدة أمنية صارمة؛ إذا تم منح مستخدم صلاحية `Allow` وتم حظره بـ `Deny` (سواء بشكل مباشر أو عبر مجموعة)، فإن **الـ Deny يلغي الـ Allow دائماً**.
3. **الصلاحية الفعالة (Effective Permissions):** إذا كان المستخدم عضواً في أكثر من مجموعة أمنية (Security Group)، يكتسب **مجموع الصلاحيات (Cumulative Allow)** الممنوحة لجميع المجموعات، بشرط عدم وجود `Deny`.

---

## 📏 2. حصص التخزين (Disk Quota)

### 🔹 ما هو مفهوم الـ Disk Quota؟
هو نظام إدارة وسيطة يُستخدم لتحديد ومراقبة أقصى مساحة تخزينية مسموح للمستخدم استهلاكها على بارتيشن معين (Volume)، لمنع مستخدم واحد من ملء الهارد بالكامل وتعطيل السيرفر أو باقي الموظفين.

### ⚙️ الأنواع والمستويات الأساسية للـ Quota:

#### 1️⃣ Soft Quota (الحد التحذيري - Limit with Warning)
* **المفهوم:** يرسل النظام تنبيهاً (Event Log / Email) للمستخدم أو الأدمن عند وصول استهلاك المستخدم لنسبة معينة (مثلاً 80% من المساحة المحددة).
* **التأثير:** **لا يمنع** المستخدم من استكمال كتابة البيانات أو حفظ الملفات.

#### 2️⃣ Hard Quota (الحد القاطع - Hard Limit)
* **المفهوم:** يحدد سقفاً خرسانياً للمساحة لا يمكن تجاوزه إطلاقاً (مثلاً 5 GB لكل موظف).
* **التأثير:** مجرد وصول المستخدم للحد الأقصى، يرفض النظام حفظ أي ملفات جديدة ويظهر للمستخدم خطأ `Disk Space Full`.

### 💡 أدوات تطبيق الـ Quota في ويندوز سيرفر:
* **NTFS Volume Quota (الأداة المدمجة):** تطبق بناءً على تملك الملفات (File Ownership) لكل مستخدم على البارتيشن.
* **FSRM (File Server Resource Manager):** الأداة المتقدمة مؤسسياً، وتسمح بتطبيق Quotas على مستويات المجلدات (Folder-based Quota) بغض النظر عن مالك الملف.

---

## 📸 3. النسخ الظلي (Shadow Copies / VSS)

### 🔹 ما هو بروتوكول VSS (Volume Shadow Copy Service)؟
هي ميزة في نظام التشغيل تقوم بأخذ **لقطات فورية (Snapshots)** محددة بزمن لملفات ومجلدات البارتيشن، مما يتيح استعادة النسخ القديمة من الملفات في حالة التلف، التعديل الخاطئ، أو الحذف بالخطأ.

### ⚙️ المبادئ والخصائص التقنية:
* **العملية التفاضلية (Block-Level Differencing):** لا يقوم Shadow Copy بأخذ نسخة كاملة مكررة من الهارد، بل يسجل فقط **التغييرات التي طرأت على البلوكات (Changed Blocks)** منذ آخر لقطة، مما يوفر المساحة التخزينية بشكل هائل.
* **الخدمة الذاتية للمستخدم (Self-Service Recovery):** يمكن للمستخدم العادي استعادة ملفاته المفقودة بنفسه دون الرجوع لإدارة الـ IT، وذلك عبر تبويب **"Previous Versions"** عند ضغط كليك يمين على المجلد أو الملف.
* **العمل دون تعطيل (Live Snapshots):** تستطيع الخدمة أخذ لقطات فورية للملفات حتى أثناء فتحها وقراءتها أو الكتابة عليها من قبل المستخدمين.

> ⚠️ **ملاحظة أمنية هامة:** 
> ميزة Shadow Copies **ليست بديلاً عن النسخ الاحتياطي (Backup)**؛ لأن اللقطات تُحفظ على نفس وحدة التخزين (Volume)، وإذا تعرض الهارد الفيزيائي للتلف الكامل، ستفقد البيانات والـ Shadow Copies معاً.

---

## 📊 4. Storage Management Comparison (مقارنة سريعة)

| المفهوم | المفهوم بالإنجليزية | الهدف الأمني / الإداري الرئيسية | مستوى التطبيق |
| :--- | :--- | :--- | :--- |
| **صلاحيات الأمان** | **NTFS Permissions** | التحكم في من يستطيع القراءة، التعديل، والحذف. | الملفات والمجلدات (File/Folder) |
| **حصص التخزين** | **Disk Quota** | منع استنزاف المساحة وتوزيع السعة بالعدل. | البارتيشن أو المجلد (Volume/Folder) |
| **النسخ الظلي** | **Shadow Copies (VSS)**| حماية البيانات من الحذف والتعديل الخاطئ والاستعادة السريعة. | البارتيشن بالكامل (Volume Level) |



---
### Security Permissions - Lab 
***
---


## 🛡️ Task 1: Basic NTFS Permissions Assignment

### 🎯 الهدف:
تطبيق الصلاحيات القياسية على المجلدات بناءً على الدور الوظيفي لكل مجموعة عبر تبويب الأمان (**Properties ➡️ Security Tab**).

### 🔹 خطوات التنفيذ:
1. **مجلد `D:\CompanyData\Public`:**
   * منح مجموعة `SG_AllEmployees`: صلاحية **Read & Execute** و **Write**.
   * منح مجموعة `Domain Admins`: صلاحية **Full Control**.

2. **مجلد `D:\CompanyData\Finance`:**
   * منح مجموعة `SG_Finance`: صلاحية **Modify** (لتسمح بقراءة، تعديل، وإنشاء وإنهاء الملفات).
   * إزالة حساب `Users` العادي من الوصول المباشر.

---

## ⛓️ Task 2: Managing NTFS Inheritance (كسر وتعطيل التوريث)

### 🎯 الهدف:
منع المجلدات الحساسة مثل `HR_Confidential` من وراثة صلاحيات المجلد الأب (`CompanyData`) لضمان الخصوصية.

### 🔹 الخطوات التطبيقية (GUI):
1. كليك يمين على مجلد `HR_Confidential` ➡️ اختيار **Properties**.
2. الانتقال لتبويب **Security** ➡️ الضغط على **Advanced**.
3. الضغط على زر **Disable Inheritance** (تعطيل التوريث).
4. عند ظهور نافذة الخيارات المتاحة:
   * **Convert inherited permissions into explicit permissions on this object:** (تحويل الصلاحيات الموروثة إلى صريحة للتعديل عليها).
   * **Remove all inherited permissions:** (حذف جميع الصلاحيات الموروثة والبدء بجدول أمان فارغ).
5. **الخيّار المنفذ:** اختيار **Convert** ثم إزالة كافة المجموعات وغير الإبقاء على `Domain Admins` و `SG_HR` فقط.
6. منح مجموعة `SG_HR` صلاحية **Modify**.

---
`Rename = Delete` > لو جربت تنشئ ملف بعد لما قمت ب الغاء خيار الحذف علي مجموعة معينة مينفعش تغير اسمه ف نفس الفايل شير > انشئ الملف علي الديسكتوب بالأسم اللي تحبه وبعدها قم بنقله اللي الفايل شير 

- عشان اخلي يوزر معين يبقي مسموحله حذف الملفات لازم اخرجه برا الجروب الغير مسموح ليه. 

***


---
### Disk Quota - LAB
***
---

# Lab Manual: Windows Server Disk Quota Management (NTFS )

تطبيق عملي شامل للتحكم في استهلاك وسائط التخزين وإدارة حصص المستخدمين (**Disk Quota Management**) على بيئة **Windows Server (PDC VM)**، يغطي تهيئة أقراص NTFS، تثبيت وتفعيل دور FSRM، تطبيق الـ Hard & Soft Quotas، واختبار سلوك النظام عند تجاوز السعة.


---

## 📏 Task 1: Built-in NTFS Volume Quota Configuration

### 🎯 الهدف:
تفعيل تحديد المساحات المدمج على مستوى القرص (`D:`) بالكامل بناءً على مالك الملف (File Ownership).

### 🔹 خطوات التنفيذ (GUI):
1. فتح **This PC** ➡️ كليك يمين على القرص `New Volume (D:)` ➡️ اختيار **Properties**.
2. الانتقال إلى تبويب **Quota** ➡️ الضغط على **Show Quota Settings**.
3. تفعيل الخيارات التالية:
   * 🗹 **Enable quota management** (تفعيل إدارة الحصص).
   * 🗹 **Deny disk space to users exceeding quota limit** (حظر حفظ الملفات عند تجاوز الحد).
4. تحديد السعة الافتراضية للمستخدمين الجدد:
   * **Limit disk space to:** `5 GB`
   * **Set warning level to:** `4 GB`
5. تفعيل تسجيل الأحداث (Event Logging):
   * 🗹 Log event when a user exceeds their quota limit.
   * 🗹 Log event when a user exceeds their warning level.
6. الضغط على **Apply** ➡️ ثم **OK** لتطبيق السياسة على القرص.

### 🔹 تخصيص مساحة استثنائية لمستخدم معين (Quota Entries):
* من نفس النافذة ➡️ الضغط على **Quota Entries...**.
* الضغط على **Quota ➡️ New Quota Entry**.
* اختيار الحساب `HRUser01` وتخصيص حد استثنائي: **Limit:** `10 GB` / **Warning:** `8 GB`.

---



---
### Shadow Copy - LAB
***
---

# Lab Manual: Windows Server Volume Shadow Copies (VSS) & Self-Service Recovery

تطبيق عملي شامل لتفعيل وإدارة ميزة النسخ الظلي (**Volume Shadow Copy Service - VSS**) على بيئة **Windows Server (PDC VM)**، يغطي إعداد اللقطات الفورية للقرص، ضبط الجدولة والمساحة التخزينية، محاكاة فقدان وتلف البيانات، واختبار الاستعادة الذاتية عبر ميزة **Previous Versions**.

---

## 🖥️ 1. Lab Environment Setup

* **Server Name:** `PDC.TEST.LOCAL`
* **Target Volume:** `New Volume (D:)`
* **Target Shared Path for Testing:** `\\pdc\hr` (المجلد المحلي `D:\HR`)
* **Test File:** `HR_Employees_2026.docx`

---

## 📸 Task 1: Enabling & Configuring Shadow Copies (GUI)

### 🎯 الهدف:
تفعيل ميزة VSS على القرص `D:` وتحديد سقف المساحة التخزينية المخصصة لحفظ اللقطات التفاضلية.

### 🔹 خطوات التنفيذ:
1. فتح **This PC** ➡️ كليك يمين على القرص `New Volume (D:)` ➡️ اختيار **Configure Shadow Copies...** (أو من تبويب **Shadow Copies** في Properties).
2. تحديد القرص `D:` من القائمة ➡️ الضغط على **Enable**.
3. عند ظهور الرسالة التحذيرية الخاصة بالجدولة الافتراضية، اضغط **Yes** لإنشاء أول لقطة فورية (Initial Snapshot).
4. الضغط على **Settings...** لتخصيص الخيارات:
   * **Storage area:** اختيار القرص المحدد لحفظ اللقطات (`D:\`).
   * **Maximum size:** اختيار **Use limit** وتحديد مساحة `5000 MB` (5 GB) لحفظ التغيرات.
   * **Schedule:** الضغط على **Schedule...** للتحكم في مواعيد أخذ اللقطات تلقائياً (مثلاً: مرتين يومياً في الساعة 7:00 AM والساعة 12:00 PM طوال أيام العمل).
5. الضغط على **OK** ➡️ ثم **Apply**.

---

## 🧪 Task 2: Data Loss Simulation & Snapshot Creation

### 🔹 Step 1: Initial Baseline Snapshot
1. إنشاء ملف نصي داخل المجلد المشترك: `D:\HR\HR_Employees_2026.docx` يحتوي على النص الأصلي: `"Initial HR Data - Version 1.0"`.
2. من نافذة **Shadow Copies** على القرص `D:` ➡️ الضغط على **Create Now** لأخذ لقطة يدوية فورية لبيانات الهارد في هذه اللحظة.

### 🔹 Step 2: Simulating File Corruption & Deletion
1. **سيناريو التعديل الخاطئ:** فتح الملف `HR_Employees_2026.docx` ومسح محتواه وكتابة `"Corrupted Data"` ثم الحفظ والتقفيل.
2. **سيناريو الحذف النهائي (Permanent Delete):** إنشاء ملف آخر باسم `HR_Salaries.xlsx` ثم حذفه بـ `Shift + Delete` لمسحه نهائياً دون المرور على Recycle Bin.

---

## 🔄 Task 3: Self-Service Data Recovery via Previous Versions

### 🎯 الهدف:
استعادة الملفات المحذوفة أو التالفة بنجاح دون الحاجة لعمل Full Backup Restore أو التوقف عن العمل.

### 🔬 Scenario A: Restoring Modified File Content
1. التوجه إلى المجلد `D:\HR` (محلياً أو عبر الشبكة `\\pdc\hr`).
2. كليك يمين على الملف التالف `HR_Employees_2026.docx` ➡️ اختيار **Properties**.
3. الانتقال إلى تبويب **Previous Versions**.
4. يظهر بالنظام إصدار سابق تم التقاطه قبل عملية التعديل الخاطئ.
5. الخيارات المتاحة:
   * **Open:** لفتح الإصدار القديم واستعراضه وقراءته أولاً للتأكد.
   * **Copy...:** لاستخراج النسخة القديمة وحفظها في مكان آخر دون المساس بالملف الحالي.
   * **Restore:** لاستبدال الملف الحالي مباشرة بالنسخة السليمة.
6. اختيار **Restore** ➡️ التأكيد بنعم.
7. **النتيجة:** عودة المحتوى الأصلي للملف بنجاح (`"Version 1.0"`).

### 🔬 Scenario B: Recovering Permanently Deleted Files
1. كليك يمين على الفولدر الرئيسي `HR` نفسه ➡️ اختيار **Properties** ➡️ تبويب **Previous Versions**.
2. تحديد اللقطة الزمانية التي تم أخذها قبل عملية الحذف ➡️ الضغط على **Open**.
3. يفتح النظام نافذة استعراض المجلد كما كان في الماضي؛ نجد داخلها الملف المحذوف `HR_Salaries.xlsx`.
4. نسخ الملف المحذوف وسحبه إلى المجلد الحالي (Drag & Drop).
5. **النتيجة:** استعادة الملف المحذوف بنجاح 100%.

---


---
### File Server Resource Manager - explain 
***
---

# Enterprise File Management: File Server Resource Manager (FSRM)

مستند مرجعي شامل يغطي أداة **File Server Resource Manager (FSRM)** في بيئات **Windows Server**، والتي تُعد الركيزة الأساسية لإدارة مراكز البيانات وسيرفرات الملفات (File Servers) وتحقيق الأمان، المراقبة، وتنظيم المساحات التخزينية.

---

## 🛡️ 1. ما هي أداة FSRM؟

**File Server Resource Manager (FSRM)** هي خدمة دورية (**Role Service**) مدمجة تحت مظلة **File and Storage Services** في Windows Server. 

تُستخدم FSRM لتأمين وتنظيم خوادم الملفات عبر تمكين مسؤولي النظام (System Administrators) من:
* التحكم الدقيق في السعات التخزينية للمجلدات.
* منع حفظ أنواع معينة من الملفات الضارة أو غير المصرح بها.
* أتمتة تصنيف البيانات (File Classification).
* توليد تقارير شاملة عن استهلاك القرص الصلب.

---

## ⚙️ 2. الوظائف والميزات الأساسية (Core Features)

تتكون أداة FSRM من خمسة المحاور الرئيسية التالية:

### 1️⃣ Quota Management (إدارة حصص التخزين المتقدمة)
تختلف FSRM عن الـ Quotas العادية بأنها تطبق على مستوى **المجلدات (Folder-Level)** وليس فقط البارتيشن بالكامل.
* **Hard Quota:** يمنع المستخدم من تجاوز المساحة المحددة نهائياً ويظهر خطأ `Disk Space Full`.
* **Soft Quota:** يسمح للتطبيقات والمستخدمين بتجاوز المساحة مع إرسال تنبيهات بريدية أو تسجيل حدث في الـ Event Log.
* **Auto Apply Quota:** ميزة فائقة الأهمية تسمح بتطبيق الـ Quota تلقائياً على **أي مجلد فرعي جديد (Subfolder)** يتم إنشاؤه مستقبلاً داخل مجلد معين (مثل مجلدات الموظفين `D:\Users\*`).

### 2️⃣ File Screening Management (حظر وفلترة الملفات)
تسمح بمنع المستخدمين من رفع أو حفظ أنواع معينة من الملفات على خوادم الشركة بناءً على الامتداد (File Extension).
* **Active Screening (حظر فعلي):** يمنع المستخدم تماماً من حفظ الملفات المحددة (مثلاً منع حفظ مقاطع الفيديو `.mp4` أو الملفات التنفيذية `.exe`).
* **Passive Screening (مراقبة وتسجيل):** يسمح للمستخدم بحفظ الملف ولكنه يرسل إشعاراً للأدمن أو يسجل الحدث للمتابعة.
* **File Groups:** تجميع الامتدادات في مجموعات معيارية (مثل: Audio/Video Files, Executable Files, أو امتدادات الفدية Ransomware Extensions).
* **File Screen Exceptions:** استثناء مجلد فرعي معين من سياسة الحظر العامة.

### 3️⃣ Storage Reports Management (تقارير التخزين)
توليد تقارير دورية أو فورية تساعد في تحليل سلوك استهلاك الهارد وسرعة امتلاء المساحات.
* **أبرز التقارير المتاحة:**
  * **Duplicate Files:** كشف الملفات المكررة على الخادم لهدر المساحة.
  * **Large Files:** عرض أكبر الملفات حيزاً.
  * **Least Recently Accessed Files:** الملفات المهجورة التي لم يتم فتحها منذ فترة طويلة.
  * **File Screen Audit:** تقرير بالمستخدمين الذين حاولوا رفع ملفات محظورة.

### 4️⃣ File Classification Infrastructure (FCI)
آلية ذكية لتصنيف الملفات تلقائياً بناءً على محتواها أو خصائصها (Metadata).
* **المفهوم:** يمكن إنشاء قواعد تفحص محتوى المستندات؛ إذا احتوى المستند على أرقام بطاقات إشارات أو كلمات مثل `"Confidential"`، تضع الأداة وسماً (Tag) تلقائياً على الملف بأنه **"عالي السرية"**.
* **الفائدة:** تسهيل ربط الملفات مع سياسات منع تسريب البيانات (**DLP**) وحمايتها.

### 5️⃣ File Management Tasks (إدارة المهام التلقائية)
تنفيذ إجراءات وأوامر تلقائية بناءً على تصنيف الملفات (FCI) أو عمرها الزمني.
* **أمثلة للتطبيق:**
  * نقل الملفات التي لم تُفتح منذ عامين تلقائياً إلى سيرفر أرشيف بطيء (Archive Server).
  * تشفير الملفات المصنفة كـ "سرية" فور إنشائها.
  * حذف الملفات المؤقتة تلقائياً بعد فترة زمنية محددة.

---

## ⚔️ 3. مقارنة: FSRM Quota vs. NTFS Volume Quota

| وجه المقارنة | NTFS Volume Quotas (المدمجة) | FSRM Quotas (المتقدمة) |
| :--- | :--- | :--- |
| **مستوى التطبيق** | البارتيشن بالكامل (Volume Level). | مجلدات محددة (Folder Level) أو البارتيشن. |
| **طبيعة الحساب** | تعتمد على مالك الملف (File Ownership). | تعتمد على المساحة الإجمالية للمجلد بغض النظر عن المالك. |
| **التطبيق التلقائي** | لا تدعم التطبيق على الفولدرات الجديدة. | تدعم **Auto-apply** على المجلدات الفرعية المستحدثة. |
| **التنبيهات** | تنبيهات بسيطة في الـ Event Log. | تنبيهات بريد إلكتروني (SMTP)، أوامر Scripts، وتقارير. |

---

## 🛡️ 4. استخدام FSRM في الحماية من هجمات الفدية (Ransomware Protection)

تُعد FSRM أحد خطوط الدفاع الأولى ضد الفدية عبر تكتيك بسيط ومباشر:
1. إنشاء **File Group** يحتوي على كافة امتدادات التشفير المعروفة لخنافس الفدية (مثل `.crypto`, `.locky`, `.enc`).
2. إنشاء **Active File Screen** يمنع كتابة أو تعديل أي ملف بهذه الامتدادات على المجلدات المشتركة.
3. ربط سياسة الحظر بإجراء تلقائي (**Command Executed**) يقوم بـ:
   * عزل حساب المستخدم فوراً وإيقاف جلسة الـ SMB الخاصة به لمنع استكمال تشفير بقية سيرفر الملفات.

---

## 📊 5. Summary Cheatsheet (ملخص الأداة)

| الميزة | Purpose / الفائدة الأساسية |
| :--- | :--- |
| **Quota Management** | تحديد سقف تخزيني للمجلدات لمنع ملء الهارد (Hard/Soft). |
| **File Screening** | منع رفع أنواع ملفات غير مرغوبة (`.exe`, `.mp3`) وحماية من الفدية. |
| **Storage Reports** | كشف الملفات المكررة والضخمة والمهجورة لتوفير المساحات. |
| **Classification (FCI)**| وسم وتصنيف البيانات تلقائياً حسب درجة السريّة والمحتوى. |
| **File Management Tasks**| أتمتة الأرشفة، الحذف، والتشفير للملفات بناءً على قواعد سابقة. |




### ---
### File Server Resource Manager - LAB
***
---


تطبيق عملي شامل لإدارة وتأمين خوادم الملفات باستخدام أداة **File Server Resource Manager (FSRM)** على بيئة **Windows Server (PDC VM)**، يغطي أتمتة حصص التخزين للمجلدات المستحدثة، حظر رفع الملفات التنفيذية والوقاية من هجمات الفدية، وتوليد تقارير تحليل المساحات.

---

## 🖥️ 1. Lab Environment Setup

* **Domain Controller / File Server:** `PDC.TEST.LOCAL`
* **Target Storage Volume:** `New Volume (D:)`
* **Directories Structure for Lab:**
  * `D:\DepartmentShares\Users` (مخصص لاختبار الـ Auto-apply Quota للموظفين)
  * `D:\DepartmentShares\PublicDocs` (مخصص لاختبار حظر أنواع الملفات - File Screening)
* **Testing User Account:** `HRUser01`

---

## 🛠️ Task 1: Installing FSRM Role & Verification

### 🔹 Step 1: Install FSRM via PowerShell
```powershell
# تثبيت دور FSRM وأدوات الإدارة التابعة له
Install-WindowsFeature -Name FS-Resource-Manager -IncludeManagementTools
```

### 🔹 Step 2: Open FSRM Management Console
* من قائمة **Tools** داخل **Server Manager** ➡️ اختيار **File Server Resource Manager**.

---

## 📏 Task 2: Configuring Auto-Apply Quotas on User Directories

### 🎯 الهدف:
تطبيق سياسة حصة تخزينية (Quota) تلقائية على أي مجلد فرعي جديد يتم إنشاؤه مستقبلاً داخل المجلد الرئيسي `D:\DepartmentShares\Users` دون تدخل يدوي من الأدمن.

### 🔹 خطوات التنفيذ:
1. داخل FSRM ➡️ الانتقال إلى `Quota Management` ➡️ `Quota Templates`.
2. كليك يمين ➡️ **Create Quota Template...**:
   * **Template Name:** `Enterprise User 2GB Hard Quota`
   * **Space Limit:** `2 GB`
   * **Quota Type:** `Hard Quota`
   * **Notification Thresholds:** إضافة تنبيه عند الوصول لـ 85% (Event Log Notification).
3. الانتقال إلى `Quota Management` ➡️ `Quotas`.
4. كليك يمين ➡️ **Create Quota...**:
   * **Quota Path:** `D:\DepartmentShares\Users`
   * **Selection Mode:** اختيار 🔘 **Auto apply template and create quotas on existing and new subfolders**.
   * **Template:** اختيار `Enterprise User 2GB Hard Quota`.
5. الضغط على **Create**.

---

## 🛡️ Task 3: File Screening & Anti-Ransomware Enforcement

### 🎯 الهدف:
حظر رفع الملفات التنفيذية (`.exe`, `.bat`) وامتدادات التشفير الخاصة بهجمات الفدية (Ransomware) على مجلد المستندات العامة `PublicDocs`.

### 🔹 Step 1: Defining Custom File Groups (Ransomware Extensions)
1. داخل FSRM ➡️ الانتقال إلى `File Screening Management` ➡️ `File Groups`.
2. كليك يمين ➡️ **Create File Group...**:
   * **File Group Name:** `Anti-Ransomware Group`
   * **Files to include:** إضافة الامتدادات المشبوهة مثل:
     `*.crypto`, `*.locky`, `*.cerber`, `*.encrypted`, `*.wannacry`
3. الضغط على **OK**.

### 🔹 Step 2: Creating Active File Screen
1. الانتقال إلى `File Screening Management` ➡️ `File Screens`.
2. كليك يمين ➡️ **Create File Screen...**:
   * **File Screen Path:** `D:\DepartmentShares\PublicDocs`
   * **Screening Type:** 🔘 **Active Screening** (منع المستخدم من حفظ الملفات).
   * **File Groups:** تحديد `Executable Files` + `Anti-Ransomware Group`.
3. الضغط على **Create**.

---

## 📊 Task 4: Generating Storage Diagnostic Reports

### 🎯 الهدف:
إنشاء تقرير فوري لكشف الملفات المكررة والملفات الضخمة التي تستهلك سعة القرص `D:`.

### 🔹 خطوات التنفيذ:
1. داخل FSRM ➡️ الانتقال إلى `Storage Reports Management`.
2. كليك يمين ➡️ **Generate Reports Now...**.
3. من قائمة **Report data to include**، حدد الخيارات التالية:
   * 🗹 **Duplicate Files** (الملفات المكررة).
   * 🗹 **Large Files** (الملفات الضخمة - ضبط الحد الأدنى لـ 100 MB).
4. في تبويب **Scope** ➡️ الضغط على **Add** واختيار القرص `D:\`.
5. اختيار **Format:** `HTML` ➡️ الضغط على **OK** واختيار **Wait for reports to be generated and then display them**.
6. **النتيجة:** يفتح متصفح النت تلقائياً بتقرير تفصيلي يوضح أماكن الملفات المكررة وحجمها لتوفير المساحة.

---

## 🧪 Task 5: Testing, Verification & CLI Commands

### 🔬 Test 1: Testing Auto-Apply Quota
1. إنشاء مجلد فرعي جديد باسم: `D:\DepartmentShares\Users\HRUser01`.
2. الانتقال لـ FSRM ➡️ `Quotas` ➡️ **Refresh**.
3. **النتيجة:** يظهر مجلد `HRUser01` تلقائياً كـ Quota مستقلة بحجم `2 GB Hard Quota`.

### 🔬 Test 2: Testing Active File Screen (Block Execution)
1. محاولة نسخ ملف باسم `installer.exe` أو `test.crypto` داخل `\\pdc\PublicDocs`.
2. **النتيجة المتوقعة:**
   * يفشل نظام الملفات في حفظ الملف ويظهر خطأ:
     **"You need permission to perform this action / Access Denied"**.

---



---
### Home Folder and Roaming Profile - explain 
***
---

# Enterprise User Data & Profile Management: Home Folder vs. Roaming Profile

مستند مرجعي نظري يغطي المبادئ الأساسية لتقنيات إدارة بيانات وملفات تعريف المستخدمين (**User Profiles & Personal Data**) في بيئات **Active Directory Domain Services**، مع توضيح الفروق الجوهرية بين المجلد الشخصي (**Home Folder**) وملف التعريف المتجول (**Roaming Profile**).

---

## 📁 1. المجلد الشخصي (Home Folder)

### 🔹 المفهوم والتعريف:
هو مجلد تخزين مركزي خاص بكل مستخدم يُستضاف على سيرفر الملفات (**File Server**)، ويتم ربطه تلقائياً كقرص شبكي (Mapped Drive مثل `:H`) عند تسجيل دخول المستخدم لأي جهاز داخل الدومين.

### ⚙️ آلية العمل والخصائص:
* **التخزين المركزي للمستندات:** مخصص فقط لحفظ ملفات ومستندات العمل الخاصة بالمستخدم (`Documents`, `Projects`) وليس لإعدادات ويندوز.
* **الربط التلقائي:** يتم تهيئته عبر خصائص حساب المستخدم في Active Directory (`ADUC`) من تبويب **Profile** باستخدام المتغير الذكي `%username%` في مسار الـ UNC:
  `\\FileServer\HomeFolders$\%username%`
* **الوصول حسب الطلب (On-Demand Access):** البيانات تظل على السيرفر، ويقوم الجهاز بقراءتها أو تعديلها عبر الشبكة مباشرة دون الحاجة لتنزيل المجلد بالكامل على الهارد المحلي للجهاز.

### 👍 الإيجابيات:
1. **حماية البيانات من الضياع:** في حال تلف جهاز العميل (Client Machine)، تبقى ملفات الموظف آمنة على السيرفر المركزي.
2. **تسهيل النسخ الاحتياطي (Centralized Backup):** يستطيع مسؤول النظام عمل Backup لكافة مجلدات الموظفين من مكان واحد دون المرور على أجهزتهم.
3. **سرعة تسجيل الدخول:** لا يؤثر على وقت الدخول للنظام لأنه لا ينقل ملفات أثناء الـ Logon.

### 👎 السلبيات:
* لا يحفظ إعدادات سطح المكتب، خلفية الشاشة، تفضيلات التطبيقات، أو الملفات الموجودة على الـ Desktop المحلي.

---

## 👤 2. ملف التعريف المتجول (Roaming User Profile)

### 🔹 المفهوم والتعريف:
هو ملف تعريف مستخدم بالكامل (**User Profile Folder**) يُستضاف على السيرفر، ويتضمن كافة ملفات وإعدادات وتفضيلات الشخص (`Desktop`, `Documents`, `AppData`, `Downloads`, بالإضافة لملف السجل الشخصي `NTUSER.DAT`).

### ⚙️ آلية العمل (Login/Logout Synchronization):
عندما ينتقل الموظف للعمل من جهاز كمبيوتر إلى آخر داخل الشركة:
1. **عند تسجيل الدخول (Logon):** يقوم النظام بسحب ونسخ ملف التعريف الخاص بالمستخدم بالكامل من السيرفر وتنزيله على الجهاز المحلي (`C:\Users\%username%`).
2. **أثناء العمل (Session):** يتعامل المستخدم مع النظام كأنه جهازه الشخصي تماماً (نفس الخلفية، نفس الأيقونات، نفس إعدادات البرامج).
3. **عند تسجيل الخروج (Logoff):** يقوم النظام بمقارنة التغيرات ورفع (Sync/Upload) كافة الملفات والإعدادات الجديدة من الجهاز المحلي إلى سيرفر الملفات.

### 👍 الإيجابيات:
1. **تجربة مستخدم موحدة (Seamless User Experience):** تضمن للموظف بيئة عمل مطابقة تماماً بغض النظر عن الجهاز الذي يجلس عليه داخل المؤسسة.
2. **مرونة الحركة (Mobility):** مثالية للشركات التي تعتمد نظام "المكاتب المشتركة" (Hot-Desking).

### 👎 السلبيات:
1. **بطء تسجيل الدخول والخروج (Slow Logon/Logoff Times):** إذا أصبح حجم بروفايل المستخدم ضخماً (مثلاً يحتوي على ملفات كبيرة في الـ Desktop أو Downloads)، يستغرق النظام وقتاً طويلاً جداً لنقل البيانات عبر الشبكة أثناء الدخول والخروج.
2. **استهلاك الباندويث (Network Traffic):** يولد ضغطاً عالياً على شبكة LAN أثناء الساعات الأولى للعمل (Morning Logon Storm).

---

## ⚔️ 3. مقارنة شاملة (Home Folder vs. Roaming Profile)

| وجه المقارنة | Home Folder (المجلد الشخصي) | Roaming Profile (البروفايل المتجول) |
| :--- | :--- | :--- |
| **طبيعة المحتوى** | مستندات وملفات العمل المباشرة فقط. | ملف التعريف بالكامل (Desktop, AppData, Settings, Registry). |
| **طريقة الوصول** | ربط كقرص شبكي محلي (`Drive Map` مثل `H:`). | تنزيل ومزامنة البروفايل بالكامل على الجهاز. |
| **تنسيق البيئة والأنظمة**| لا يحفظ إعدادات الشكل أو البرامج. | يحفظ كافة الإعدادات، الخلفيات، والتفضيلات الشخصية. |
| **التأثير على سرعة الدخول**| **سريع جداً** (لا يوجد نقل بيانات عند الدخول). | **قد يكون بطيئاً جداً** حسب حجم البروفايل ورابط الشبكة. |
| **استهلاك التخزين المحلي**| لا يستهلك مساحة من الهارد المحلي لجهاز العميل. | يستهلك مساحة محلية أثناء وقت الجلسة (Session). |
| **مسار الإعداد في ADUC** | Profile Tab ➡️ Home folder ➡️ Connect. | Profile Tab ➡️ Profile path. |

---

## 💡 4. الحل المعماري الحديث (Folder Redirection + Local Profiles)

نظراً لمشكلة البطيء الشديد في **Roaming Profiles**، تعتمد الشركات الحديثة معياراً مختلفاً يجمع بين الميزتين:

* **Folder Redirection (إعادة توجيه المجلدات):** استخدام سياسات GPO لتحويل مسارات مجلدات المستندات وسطح المكتب (`Desktop`, `Documents`) مباشرة إلى سيرفر الملفات عبر مسار UNC، مع الإبقاء على ملف التعريف محلياً (**Local Profile**).
* **الفائدة:** الحصول على سرعة الـ Local Profile في تسجيل الدخول، مع الحفاظ على حماية البيانات وأمانها على السيرفر المركزي مثل الـ Roaming Profile دون بطء الشبكة.

---

## 📊 5. Summary Cheatsheet (ملخص المفاهيم)

| التقنية                | الاستخدام الأنسب (Best Use Case)                                                                    |
| :--------------------- | :-------------------------------------------------------------------------------------------------- |
| **Home Folder**        | توفير مساحة تخزين شخصية ومحمية لكل موظف على السيرفر لحفظ التقارير والعمل.                           |
| **Roaming Profile**    | الموظفون المتجولون بين أجهزة متعددة والذين يحتاجون لنفس سطح المكتب والإعدادات دائماً.               |
| **Folder Redirection** | المعيار المؤسسي الأفضل لحفظ بيانات سطح المكتب والـ Documents على السيرفر مع الحفاظ على سرعة الدخول. |




---
### Home Folder and Roaming Profile - LAB
***
---

تطبيق عملي شامل لإعداد وتأمين المجلدات الشخصية (**Home Folders**) على بيئة **Windows Server (PDC VM)**، يغطي مشاركة المجلدات المخفية (Hidden SMB Shares)، ضبط صلاحيات NTFS المتقدمة لمنع تداخل ملفات الموظفين، الأتمتة عبر متغير `%username%` في Active Directory، واختبار الربط التلقائي كقرص شبكي (`H:`).

---

## 🖥️ 1. Lab Environment Setup

* **Domain Controller / File Server:** `PDC.TEST.LOCAL`
* **Local Storage Path:** `D:\HomeFolders`
* **Shared UNC Path (Hidden Share):** `\\pdc\HomeFolders$`
* **Target Mapped Drive Letter:** `H:`
* **Test Users:** `HRUser01`, `FinUser01`

---

## 🛡️ Task 1: Creating Parent Storage & Security Permissions Setup

السر في تأمين الـ Home Folders هو ضبط صلاحيات المجلد الأب (`D:\HomeFolders`) بشكل صحيح حتى يستطيع النظام إنشاء فولدر لكل مستخدم بملكية وصلاحيات منعزلة تماماً.

### 🔹 Step 1: Create Parent Directory & Hidden SMB Share
1. إنشاء مجلد محلي على الهارد: `D:\HomeFolders`.
2. كليك يمين ➡️ **Properties** ➡️ تبويب **Sharing** ➡️ **Advanced Sharing**.
3. تفعيل 🗹 **Share this folder**.
4. **Share name:** تغيير الاسم إلى `HomeFolders$` (علامة `$` تُخفي المشاركة من التصفح العادي في الشبكة).
5. الضغط على **Permissions**:
   * منح `Authenticated Users` صلاحية **Full Control** (على مستوى الـ Share فقط، وسنقيد الوصول عبر NTFS).
6. الضغط على **Apply** ➡️ **OK**.

### 🔹 Step 2: Advanced NTFS Permissions Configuration
1. الانتقال إلى تبويب **Security** ➡️ الضغط على **Advanced**.
2. الضغط على **Disable Inheritance** ➡️ اختيار **Convert inherited permissions into explicit permissions**.
3. تعديل جدول الصلاحيات ليطابق الآتي:

| Principal / Group | Permission | Applies To | Purpose |
| :--- | :--- | :--- | :--- |
| **Domain Admins** | **Full Control** | This folder, subfolders, and files | إدارة السيرفر الكاملة |
| **CREATOR OWNER** | **Full Control** | Subfolders and files only | منح الموظف ملكية مجلده الخاص فقط |
| **Authenticated Users** | **Read & Execute / Create Folders** | This folder only | السماح للنظام بإنشاء مجلد المستخدم |

---



---
### Roaming Folder - LAB
***
---


# Lab Manual: Active Directory Home Folder Provisioning & Secure Automated Mapping

تطبيق عملي شامل لإعداد وتأمين المجلدات الشخصية (**Home Folders**) على بيئة **Windows Server (PDC VM)**، يغطي مشاركة المجلدات المخفية (Hidden SMB Shares)، ضبط صلاحيات NTFS المتقدمة لمنع تداخل ملفات الموظفين، الأتمتة عبر متغير `%username%` في Active Directory، واختبار الربط التلقائي كقرص شبكي (`H:`).

---

## 🖥️ 1. Lab Environment Setup

* **Domain Controller / File Server:** `PDC.TEST.LOCAL`
* **Local Storage Path:** `D:\HomeFolders`
* **Shared UNC Path (Hidden Share):** `\\pdc\HomeFolders$`
* **Target Mapped Drive Letter:** `H:`
* **Test Users:** `HRUser01`, `FinUser01`

---

## 🛡️ Task 1: Creating Parent Storage & Security Permissions Setup

السر في تأمين الـ Home Folders هو ضبط صلاحيات المجلد الأب (`D:\HomeFolders`) بشكل صحيح حتى يستطيع النظام إنشاء فولدر لكل مستخدم بملكية وصلاحيات منعزلة تماماً.

### 🔹 Step 1: Create Parent Directory & Hidden SMB Share
1. إنشاء مجلد محلي على الهارد: `D:\HomeFolders`.
2. كليك يمين ➡️ **Properties** ➡️ تبويب **Sharing** ➡️ **Advanced Sharing**.
3. تفعيل 🗹 **Share this folder**.
4. **Share name:** تغيير الاسم إلى `HomeFolders$` (علامة `$` تُخفي المشاركة من التصفح العادي في الشبكة).
5. الضغط على **Permissions**:
   * منح `Authenticated Users` صلاحية **Full Control** (على مستوى الـ Share فقط، وسنقيد الوصول عبر NTFS).
6. الضغط على **Apply** ➡️ **OK**.

### 🔹 Step 2: Advanced NTFS Permissions Configuration
1. الانتقال إلى تبويب **Security** ➡️ الضغط على **Advanced**.
2. الضغط على **Disable Inheritance** ➡️ اختيار **Convert inherited permissions into explicit permissions**.
3. تعديل جدول الصلاحيات ليطابق الآتي:

| Principal / Group | Permission | Applies To | Purpose |
| :--- | :--- | :--- | :--- |
| **Domain Admins** | **Full Control** | This folder, subfolders, and files | إدارة السيرفر الكاملة |
| **CREATOR OWNER** | **Full Control** | Subfolders and files only | منح الموظف ملكية مجلده الخاص فقط |
| **Authenticated Users** | **Read & Execute / Create Folders** | This folder only | السماح للنظام بإنشاء مجلد المستخدم |

---



---
### File Server Audit Log Monitoring - LAB
***
---


# Lab Manual: Windows Server File Server Audit Log Monitoring & Security Event Analysis

تطبيق عملي شامل لإعداد وتفعيل منظومة المراقبة والتدقيق (**File System Auditing**) على خوادم الملفات في بيئة **Windows Server (PDC VM)**، يغطي تفعيل سياسات المراقبة المتقدمة عبر GPO، ضبط قوائم التحكم بالمراقبة (SACL)، محاكاة حوادث الاختراق والتعديل والحذف، وتحليل سجلات الأحداث الأمنية (Security Event Viewer & PowerShell).

---

## 🖥️ 1. Lab Environment Setup

* **Domain Controller / File Server:** `PDC.TEST.LOCAL`
* **Target Storage Directory:** `D:\SensitiveData`
* **Test File:** `Confidential_Salaries_2026.xlsx`
* **Audit Target Group:** `Everyone` or `Authenticated Users`
* **Test Users:** `HRUser01`, `FinUser01`

---

## ⚙️ Task 1: Enabling Advanced Audit Policies via GPO

### 🎯 الهدف:
تفعيل سياسة مراقبة نظام الملفات المتقدمة (**Audit File System**) على مستوى الخادم لضمان تسجيل كافة محاولات الوصول الناجحة والفاشلة.

### 🔹 خطوات التنفيذ (Group Policy Management):
1. فتح أداة **Group Policy Management** (`gpmc.msc`).
2. إنشاء/تعديل GPO مخصص ومربوط بـ Domain Controllers أو خادم الملفات باسم: `File Server Audit Policy`.
3. الانتقال إلى المسار التالي:
   `Computer Configuration` ➡️ `Policies` ➡️ `Windows Settings` ➡️ `Security Settings` ➡️ `Advanced Audit Policy Configuration` ➡️ `Audit Policies` ➡️ `Object Access`
4. كليك يمين على **Audit File System** ➡️ **Properties**:
   * 🗹 **Configure the following audit events**
   * 🗹 **Success** (تسجيل العمليات الناجحة)
   * 🗹 **Failure** (تسجيل محاولات الوصول المرفوضة)
5. تطبيق السياسة على السيرفر فوراً عبر موجه الأوامر:
   ```cmd
   gpupdate /force
   ```

---

## 🛡️ Task 2: Configuring System Access Control Lists (SACL)

لتسجيل الأحداث على مجلد معين، لا يكفي تفعيل GPO فقط، بل يجب تحديد المجلد والملفات المراد مراقبتها عبر الـ **SACL (System Access Control List)**.

### 🔹 خطوات ضبط الـ SACL على المجلد (D:\SensitiveData):
1. كليك يمين على مجلد `D:\SensitiveData` ➡️ **Properties** ➡️ تبويب **Security**.
2. الضغط على **Advanced** ➡️ الانتقال إلى تبويب **Auditing**.
3. الضغط على **Add** لتحديد قواعد المراقبة:
   * **Principal:** اختيار `Everyone` (أو `Authenticated Users`).
   * **Type:** `All` (لمراقبة Success & Failure).
   * **Applies to:** `This folder, subfolders and files`.
4. الضغط على **Show advanced permissions** وتحديد العمليات الحرجة:
   * 🗹 **Full control** (أو تحديد: Write data, Delete, Delete subfolders and files, Change permissions).
5. الضغط على **OK** ➡️ **Apply** ➡️ **OK**.

---

## 🧪 Task 3: Incident Simulation & Event Generation

### 🔬 Scenario A: Unauthorized File Access & Read
* تسجيل الدخول بحساب `FinUser01` وقراءة الملف `D:\SensitiveData\Confidential_Salaries_2026.xlsx`.

### 🔬 Scenario B: File Content Modification
* التعديل على محتوى الملف وحفظ التغيرات بواسطة `FinUser01`.

### 🔬 Scenario C: File Permanent Deletion
* حذف ملف حساس داخل المجلد عبر `Shift + Delete`.

### 🔬 Scenario D: Security Permissions Alteration
* محاولة تعديل صلاحيات المجلد (Change Permissions) لحظر بقية المستخدمين.

---

## 🔍 Task 4: Security Event Viewer Analysis & Critical Event IDs

المرحلة الحاسمة هي تحليل سجلات الأمان داخل **Event Viewer ➡️ Windows Logs ➡️ Security**.

### 📊 أهم رموز الأحداث (Key Event IDs) الخاصة بمراقبة الملفات:

| Event ID | Event Name / Meaning | Technical Context & Triggers |
| :---: | :--- | :--- |
| **`4656`** | **Handle Requested** | تم طلب فتح مقبض الوصول للملف (فحص بداية محاولة الفتح). |
| **`4663`** | **Object Access Executed** | **الأهم مطلقاً:** ينفذ عند القراءة، الكتابة، أو الحذف الفعلي. |
| **`4660`** | **Object Deleted** | تأكيد عملية حذف الملف أو المجلد نهائياً. |
| **`4670`** | **Permissions Changed** | تم تغيير جدول الصلاحيات (NTFS ACLs) الخاص بالملف. |

### 🔬 قراءة تفاصيل Event ID 4663:
عند فتح الحدث `4663` داخل الـ Event Viewer، تجد المعلومات الجنائية التالية:
* **Account Name:** اسم المستخدم الذي قام بالعملية (مثلاً: `FinUser01`).
* **Object Name:** المسار الكامل للملف (`D:\SensitiveData\Confidential_Salaries_2026.xlsx`).
* **Process Name:** البرنامج المستخدم في العملية (مثلاً: `C:\Windows\explorer.exe` أو `powershell.exe`).
* **Accesses / Access Mask:** نوع العملية الدقيقة:
  * `WriteData (or AddFile)` ➡️ تعني تعديل/كتابة.
  * `DELETE` ➡️ تعني مسح الملف.
  * `READ_CONTROL` / `ReadData` ➡️ تعني قراءة واستعراض.

---

