# شرح تفصيلي لسياسات المجموعة بناءً على مخطط الـ Group Policy

يلخص هذا المخطط الموضح في الصورة Group+Policy.jpeg المفاهيم الأساسية لكيفية عمل وتطبيق وإدارة الـ Group Policy داخل بيئة الـ Active Directory.

---

## 1. ترتيب وأولوية تطبيق السياسات (LSDOU Hierarchy)
تُطبق السياسات بترتيب تصاعدي من الأعلى لأسفل، والسياسة اللاحقة تلغي السابقة في حال التعارض:
* **L (Local):** السياسات المحلية على الجهاز نفسه، ويمكن فتحها في الـ Run عبر الأمر `gpedit.msc`.
* **S (Site):** السياسات المطبقة على الموقع الجغرافي للشبكة.
* **D (Domain):** السياسات المطبقة على مستوى النطاق بالكامل (مثل النطاق الموضح بالصورة: `Test.local`).
* **O.U (Organizational Unit):** السياسات المطبقة على مستوى الوحدة التنظيمية (القسم)، وهي المربعات الأكثر تخصيصاً والأعلى أولوية في التطبيق الفعلي.

---

## 2. أقسام الـ Group Policy وزمن تطبيقها
تنقسم السياسات إلى شقين رئيسيين بناءً على الهدف وزمن التفعيل:
* **User Configuration:** إعدادات مخصصة لـ "المستخدم"، وتُطبق وتتحدث عند القيام بعمليتي تسجيل الدخول أو الخروج (**Logon / Logoff**).
* **Computer Configuration:** إعدادات مخصصة لـ "الجهاز نفسه"، وتُطبق وتتحدث بمجرد إعادة تشغيل الجهاز (**Restart**).

---

## 3. طريقة الوصول والإنشاء (Management & Configuration)
لإدارة وإنشاء السياسات، نتبع المسار التالي:
1. من خلال سيرفر الويندوز: **Server Manager** ➡️ **Tools** ➡️ **Group Policy Management**.
2. أو نفتحها مباشرة عبر الـ Run بالأمر السريع: `gpmc.msc`.
3. داخل الأداة، نقوم بإنشاء كائن السياسة **Create (GPO)** ثم نربطه بالقسم المستهدف **Link it here**.
4. للتعديل على إعدادات السياسة ومحتواها، نضغط عليها كليك يمين واختيار **Edit**.

---

## 4. مفاهيم التحكم المتقدمة في الـ GPO
يحتوي المخطط على ثلاثة مفاهيم رئيسية للتحكم في كيفية وراثة وتنفيذ السياسات:

* **Block Inheritance:** خاصية تُستخدم لمنع الـ OU من وراثة السياسات القادمة إليها من الأعلى (من مستوى الدومين أو من OUs أعلى منها).
* **Enforced (علامة القفل):** خاصية فرض السياسة إجبارياً؛ إذا تم تفعيلها على مستوى الدومين مثلاً، فإنها ستُطبق رغماً عن وجود خاصية الـ *Block Inheritance* في الأسفل.
* **Link Order (ترتيب الروابط):** عند تطبيق أكثر من سياسة متعارضة على نفس الـ OU (مثل المثال الموضح في قسم `Sales`: السياسة الأولى `Disable CMD` والسياسة الثانية `Enable CMD`). 
  * **القاعدة الذهبية:** `Lowest Number = Highest Priority` (السياسة صاحبة الرقم الأقل في الترتيب هي الفائزة ولها الأولوية القصوى في التطبيق).

---

## 5. أوامر تحديث السياسات (Update Commands)
بشكل افتراضي، تتحدث السياسات تلقائياً كل فترة، ولكن لتطبيق التعديلات الجديدة فوراً على أجهزة المستخدمين، نستخدم الأوامر التالية في الـ Run أو الـ CMD:
* `gpupdate`: يقوم بتحديث السياسات التي جرى عليها تعديل أو المضافة حديثاً فقط.
* `gpupdate /force`: يقوم بإعادة تطبيق وفرض كافة السياسات (القديمة والجديدة) إجبارياً على الجهاز.



---
## Lab 1 : Remove Clock , Remove Properties ,  Link enable 
*** 
---

### 🚀 خطوات التنفيذ (Implementation Steps)

#### أولاً: إنشاء الـ GPO وتسميتها
1. افتح أداة **Group Policy Management**.
2. اضغط كليك يمين على الـ **OU** الأولى المستهدفة ⬅️ اختر **Create a GPO in this domain, and Link it here...**.
3. قم بتسمية الكائن الجديد: `🔒 User-Restrictions-Policy`.
4. اضغط كليك يمين على السياسة المنشأة ⬅️ اختر **Edit** لفتح المحرر وضبط الإعدادات.

#### ثانياً: ضبط السياسات المطلوبة (Policy Configurations)

| السياسة (Policy) | المسار داخل المحرر (Path) | الحالة (Setting) |
| :--- | :--- | :--- |
| **إخفاء الساعة** | `User Configuration` > `Administrative Templates` > `Start Menu and Taskbar` > **Remove Clock from the system notification area** | **Enabled** |
| **تعطيل Properties** | `User Configuration` > `Administrative Templates` > `Desktop` > **Remove Properties from the Computer icon context menu** | **Enabled** |

---

### 📝 💡 ملاحظات هامة وقواعد التطبيق (Crucial Lab Notes)

* **إعادة استخدام السياسات الجاهزة (Link an Existing GPO):** إذا قمت بإنشاء وضبط هذه السياسة لمجموعة معينة، وأردت تطبيقها على مجموعة (OU) ثانية أو ثالثة، **لا تحتاج لإعادة الخطوات من الصفر**. 
  * **الطريقة:** اضغط كليك يمين على المجموعة الجديدة ⬅️ اختر **Link an Existing GPO...** ⬅️ اختر السياسة الجاهزة `User-Restrictions-Policy` لتُطبق عليها فوراً وبشكل مباشر.
* **حالة الرابط (Link Enabled):** عند ربط أي سياسة (سواء كانت جديدة أو تم إعادة استخدامها من مجموعة أخرى)، تأكد دائماً أن خيار **Link Enabled** مفعل (تظهر علامة الصح ✔️ بجانبه)؛ فبدون تفعيل هذا الخيار، ستظل السياسة مربوطة بالـ OU ولكنها معطلة ولن تؤثر على المستخدمين.
* **ترتيب الروابط والأولوية (Link Order):** في حال تعارضت أكثر من سياسة على نفس الـ OU، فإن السياسة صاحبة الرقم الأقل (`Lowest Number`) هي التي تمتلك الأولوية القصوى (`Highest Priority`) وتُلغي ما سواها.
* **طبيعة التأثير والزمن:** الإعدادات المطبقة هنا تقع تحت قسم `User Configuration`؛ لذلك لن تظهر التغييرات فوراً إلا بعد أن يقوم المستخدم بعمل تسجيل خروج (**Logoff**) ثم إعادة الدخول للنظام.

---

### 🔍 التحقق والاختبار (Verification & Testing)

لتحديث وفرض السياسات الجديدة فوراً على جهاز العميل دون انتظار التحديث التلقائي للويندوز، افتح موجّه الأوامر (**CMD**) ونفّذ:

bash
`gpupdate /force`


---
## Lab 2 : Disable External Storage (USB) , Exclude user from policy 
***
***

### 🚀 خطوات التنفيذ (Implementation Steps)

#### أولاً: إنشاء الـ GPO وحظر الـ USB
1. افتح أداة **Group Policy Management** (`gpmc.msc`).
2. اضغط كليك يمين على الـ **OU** المستهدفة ⬅️ اختر **Create a GPO in this domain, and Link it here...**.
3. قم بتسمية الكائن الجديد: `🛑 Block-USB-Storage-Policy`.
4. اضغط كليك يمين على السياسة الجديدة ⬅️ اختر **Edit**.
5. توجه إلى المسار التالي:
   * **المسار:** `User Configuration` ⬅️ `Administrative Templates` ⬅️ `System` ⬅️ `Removable Storage Access`
6. ابحث عن السياسة: **All Removable Storage classes: Deny all access**.
7. اضغط عليها مرتين، واختر **Enabled**، ثم اضغط **OK**.

#### ثانياً: استثناء مستخدم معين (Exclude User)
1. من الواجهة الرئيسية لأداة `gpmc.msc` اضغط على اسم السياسة نفسها `Block-USB-Storage-Policy` من القائمة اليسرى.
2. من القائمة اليمنى، توجه إلى تبويب **Delegation**.
3. اضغط على زر **Advanced** في الأسفل.
4. اضغط **Add** واكتب اسم المستخدم المراد استثناؤه ثم اضغط **OK**.
5. اضغط على اسم المستخدم، وانزل لأسفل في مربع الصلاحيات (Permissions).
6. ابحث عن صلاحية **Apply group policy** وضَع علامة صح تحت عمود **Deny** (رفض).
7. اضغط **Apply** ثم **OK** ووافق على رسالة التحذير.

---

### 📝 💡 ملاحظات هامة (Crucial Lab Notes)
* **قاعدة الأمان الذهبية:** الرفض الصريح يلغي أي سماح (`Explicit Deny Overrides Allow`). وضع صلاحية Deny على Apply Group Policy تجعل النظام يتخطى تطبيق هذه السياسة على هذا الحساب تحديداً.

---

### 🔍 طرق الفحص والاختبار التقني (Verification & Testing)

بعد فتح موجّه الأوامر (**CMD**) على جهاز العميل وتحديث السياسات عبر الأمر:
bash
`gpupdate /force` 



---
## Lab 3 : Control Panel, Task Manager, Block Inheritance, Enforced, Link Order 
***
***

### 🚀 خطوات التنفيذ (Implementation Steps)

#### أولاً: حظر الـ Control Panel والـ Task Manager
1. افتح أداة **Group Policy Management** واذهب إلى الـ **Parent OU** (`Management`).
2. اضغط كليك يمين ⬅️ اختر **Create a GPO in this domain, and Link it here...**.
3. قم بتسميتها: `🚫 Core-Restrictions-Policy` ثم اضغط كليك يمين عليها واختر **Edit**.
4. قم بضبط السياسات التالية من قسم **User Configuration**:

| السياسة (Policy) | المسار داخل المحرر (Path) | الحالة (Setting) |
| :--- | :--- | :--- |
| **حظر لوحة التحكم** | `User Configuration` > `Administrative Templates` > `Control Panel` > **Prohibit access to Control Panel and PC settings** | **Enabled** |
| **حظر Task Manager** | `User Configuration` > `Administrative Templates` > `System` > `Ctrl+Alt+Del Options` > **Remove Task Manager** | **Enabled** |

---

#### ثانياً: تطبيق قفل الوراثة والفرض (Block Inheritance & Enforced)

1. **تفعيل Block Inheritance (منع الوراثة):**
   * افتراضياً، الـ Child OU (`Sales`) ترث السياسات من الـ Parent OU.
   * لمنع الـ `Sales` من تطبيق حظر الـ Control Panel والـ Task Manager عليها: اضغط كليك يمين على الـ OU المسماة `Sales` واختر **Block Inheritance**.
   * *النتيجة:* ستظهر علامة تعجب زرقاء (`i`) على مجلد الـ OU، مما يعني حجب السياسات القادمة من الأعلى.

2. **تفعيل Enforced (الفرض الإجباري):**
   * إذا أردت كمدير نظام فرض الحظر على الجميع رغماً عن قفل الوراثة: اذهب إلى الـ Parent OU واضغط كليك يمين على رابط السياسة `Core-Restrictions-Policy` واختر **Enforced**.
   * *النتيجة:* ستظهر علامة **القفل** بجانب السياسة، وسيتم كسر قفل الوراثة في الـ `Sales OU` وتطبيق الحظر عليها إجبارياً.

---

#### ثالثاً: التحكم في أولويات التطبيق (Link Order)
في حال وجود سياسات متعارضة تم ربطها معاً على نفس الـ OU (مثل المثال الموضح بالصورة: سياسة تقوم بعمل `Disable CMD` وسياسة أخرى تقوم بعمل `Enable CMD` على قسم الـ `Sales`):
1. اضغط على الـ **OU** المستهدفة من القائمة اليسرى.
2. من القائمة اليمنى، توجه إلى تبويب **Linked Group Policy Objects**.
3. ستظهر لك قائمة السياسات المربوطة وبجانبها عمود **Link Order**.
4. استخدم الأسهم الموجهة لأعلى ولأسفل (تحت خيار *Link Order*) لتعديل الترتيب.

---

### 📝 💡 ملاحظات هامة وقواعد التطبيق (Crucial Lab Notes)

* **القاعدة الذهبية للـ Link Order:** 
  > `Lowest Number = Highest Priority`
  * السياسة المكتوب بجانبها رقم **1** في الـ Link Order هي التي تمتلك الأولوية القصوى والفوز في التطبيق في حال حدوث تعارض مع سياسة تحمل الرقم **2**.
* **مفهوم الـ Enforced:** خيار الـ Enforce يمتلك سلطة مطلقة تكسر قاعدة الـ الوراثة وسلسلة الـ LSDOU؛ حيث يجبر الحاويات الأدنى (Child OUs) على تنفيذ السياسة حتى لو قامت بعمل *Block Inheritance*.

---

### 🔍 طرق الفحص والاختبار التقني (Verification & Testing)

بعد تنفيذ أمر التحديث الإجباري على جهاز العميل:
bash
gpupdate




---
## Lab 4 : Password Policy and lockout policy
***
---


### 🚀 خطوات التنفيذ (Implementation Steps)

1. افتح أداة **Group Policy Management** (`gpmc.msc`).
2. انتقل إلى `Domains` ⬅️ `[Your Domain Name]` ⬅️ **Default Domain Policy**.
3. اضغط كليك يمين عليها واختر **Edit**.
4. انتقل إلى المسار التالي:
   `Computer Configuration` > `Policies` > `Windows Settings` > `Security Settings` > `Account Policies`

#### أ. ضبط سياسة كلمة المرور (Password Policy)
| الإعداد (Setting) | القيمة المقترحة | الوصف |
| :--- | :--- | :--- |
| **Enforce password history** | 24 passwords | حفظ آخر 24 كلمة مرور لمنع التكرار. |
| **Maximum password age** | 42 days | إجبار المستخدم على تغيير كلمته بعد 42 يوماً. |
| **Minimum password length** | 12 characters | طول كلمة المرور (يُفضل 12 فأكثر). |
| **Complexity requirements** | Enabled | فرض مزيج (حروف كبيرة، صغيرة، أرقام، رموز). |

#### ب. ضبط سياسة حظر الحساب (Account Lockout Policy)
| الإعداد (Setting) | القيمة المقترحة | الوصف |
| :--- | :--- | :--- |
| **Account lockout threshold** | 5 attempts | حظر الحساب بعد 5 محاولات خاطئة. |
| **Account lockout duration** | 30 minutes | مدة الحظر قبل أن يفتح الحساب تلقائياً. |
| **Reset lockout counter after** | 30 minutes | الوقت المطلوب لإعادة عدّ المحاولات الخاطئة. |

---

### 📝 💡 ملاحظات هامة (Crucial Lab Notes)

* **نطاق التطبيق (Domain-Wide):** بخلاف اللابات السابقة، سياسات `Account Policies` لا يمكن تطبيقها على مستوى الـ OU (وحدات تنظيمية)، بل تُطبق على مستوى الدومين بالكامل. 
* **سياسات الحبيبات الدقيقة (FGPP - Fine-Grained Password Policies):** إذا أردت تطبيق سياسة كلمات مرور مختلفة لمجموعة معينة (مثلاً: المدراء بكلمات مرور أطول، والموظفون بكلمات مرور أقصر)، لن تنفعك الـ GPO العادية. يجب استخدام ميزة **FGPP** عبر أداة `ADAC` (Active Directory Administrative Center).
* **الحذر من الـ Lockout:** وضع "مدة حظر" طويلة جداً (مثل 99999 دقيقة) قد يسبب إغلاق الحسابات للأبد ويجعل المستخدمين يتصلون بمركز الدعم باستمرار لفك الحظر (Lockout Denial of Service).

---

### 🔍 طرق الفحص والاختبار التقني (Verification & Testing)

بعد تحديث السياسات عبر `gpupdate /force` في الـ CMD، يمكنك التحقق من القيم الحالية المطبقة على الجهاز عبر الأمر التالي:

bash
`net accounts`




---
## Lab 5 : Scripts - Add local admin user and IT-group as a local administrator 
***
---


## 📂 1. Scripts Configuration (ملفات السكريبتات)

### A. Logon Script (User-based Application Startup)
* **اسم الملف:** `open-calc.bat`
* **الغرض:** تشغيل الآلة الحاسبة تلقائياً فور تسجيل دخول المستخدم.
* **الكود:**
```cmd
@echo off
start calc.exe
```

### B. Startup Script (Local Admin Creation)
* **اسم الملف:** `create-local-admin.bat`
* **الغرض:** إنشاء حساب مدير محلي جديد وتفعيله على الجهاز.
* **الكود:**
```cmd
@echo off
net user itadmin P@$$w0rd /add
net localgroup administrators itadmin /add
```

### C. Startup Script (Domain Group to Local Admins)
* **اسم الملف:** `add-itgroup-to-admins.bat`
* **الغرض:** إضافة مجموعة الـ IT الخاصة بالدومين إلى الـ Administrators المحليين للجهاز.
* **الكود:**
```cmd
@echo off
net localgroup administrators IT-Group /add
```

---

## 🚀 2. GPO Implementation Steps (خطوات التنفيذ)

### الخطوة 1: تجهيز وحفظ ملفات السكريبتات (.bat)
1. افتح أداة الـ Notepad على جهاز الـ Domain Controller.
2. اكتب الأكواد الثلاثة السابقة، واحفظ كل ملف بالامتداد الخاص به وهو `.bat` (تأكد من اختيار Save as type: All Files).
3. 💡 **ملاحظة إملائية حاسمة:** تأكد من كتابة اسم المجموعة المحلية باللغة الإنجليزية وبصيغة الجمع تماماً كما هي في نظام التشغيل (`administrators` بوجود حرف الـ **s** في النهاية) لتجنب فشل السكريبت في التعرف على المجموعة.
4. ⚠️ **تنبيه أمني حرج جداً:** كتابة كلمة المرور بشكل صريح مثل `P@$$w0rd` داخل ملفات الـ Batch المخزنة في مجلد الـ `SYSVOL` تمثل ثغرة أمنية لأن أي مستخدم عادي بالدومين يستطيع قراءة الملف ومعرفة الباسورد. في البيئات الحقيقية، يُنصح بشدة باستخدام **Microsoft LAPS** لإدارة حسابات المسؤولين المحليين وتجنب هجمات تصعيد الصلاحيات (Privilege Escalation).

### الخطوة 2: ربط وتطبيق سكريبت الآلة الحاسبة كـ (User Logon)
1. افتح أداة **Group Policy Management** (`gpmc.msc`).
2. اضغط كليك يمين على الـ **OU الخاصة بالمستخدمين** ⬅️ اختر **Create a GPO in this domain, and Link it here...**.
3. سمّها `💻 Logon-Calc-GPO` ثم اضغط كليك يمين عليها واختر **Edit**.
4. توجه إلى المسار التالي:
   `User Configuration` ➡️ `Policies` ➡️ `Windows Settings` ➡️ `Scripts (Logon/Logoff)`
5. اضغط مرتين على **Logon** ⬅️ اضغط على زر **Show Files** ⬅️ ضع ملف `open-calc.bat` داخل المجلد الذي فتح لك.
6. اغلق مجلد التطبيق، ثم من نافذة الـ GPO اضغط **Add** واختَر الملف نفسه ثم اضغط **OK**.
* 💡 **ملاحظة سياق التشغيل:** ينجح هذا السكريبت كـ **Logon Script** لأنه مرتبط بواجهة المستخدم الرسومية (GUI) ويعمل بنجاح تحت صلاحيات المستخدم العادي فور تسجيل دخوله لسطح المكتب.

### الخطوة 3: ربط وتطبيق سكريبتات المدير المحلي ومجموعة الـ IT كـ (Computer Startup)
1. في أداة `gpmc.msc` اضغط كليك يمين على الـ **OU الخاصة بأجهزة الكمبيوتر** ⬅️ اختر **Create a GPO in this domain, and Link it here...**.
2. سمّها `⚙️ Startup-Admins-GPO` ثم اضغط كليك يمين عليها واختر **Edit**.
3. توجه إلى المسار التالي:
   `Computer Configuration` ➡️ `Policies` ➡️ `Windows Settings` ➡️ `Scripts (Startup/Shutdown)`
4. اضغط مرتين على **Startup** ⬅️ اضغط على زر **Show Files** ⬅️ ضع ملفي `create-local-admin.bat` و `add-itgroup-to-admins.bat` داخل المجلد الذي فتح لك تلقائياً.
5. اغلق المجلد، ثم من نافذة الـ GPO اضغط **Add** وأضف الملفين واحداً تلو الآخر، ثم اضغط **OK**.
* 💡 **ملاحظة الصلاحيات وسياق التشغيل:** سكريبتات تعديل الصلاحيات وإنشاء الحسابات المعتمدة على أوامر (`net user` / `net localgroup`) **ستفشل تماماً** إذا تم وضعها كـ *Logon Script* مع مستخدم عادي لأنه لا يمتلك صلاحيات الكتابة والتعديل على النظام؛ لذلك تم تطبيقها هنا كـ **Startup Script** لأنها تُنفذ أثناء إقلاع الجهاز وقبل دخول أي مستخدم، مما يجعلها تعمل بأعلى صلاحية داخل النظام وهي صلاحية الـ (`SYSTEM`).

---

## 🔍 3. Verification & Testing (طرق الفحص والاختبار التقني)

لتطبيق السياسات فوراً، نفذ الأمر التالي في الـ CMD على جهاز العميل:
```bash
gpupdate /force
```
بعدها قم بعمل **إعادة تشغيل لجهاز العميل (Restart)** لتطبيق سكريبتات الـ Startup أولاً ثم سجل دخولك:

1. **فحص الآلة الحاسبة:** بمجرد تسجيل الدخول بحساب المستخدم العادي، يجب أن تفتح الآلة الحاسبة تلقائياً على الشاشة فوراً.
2. **فحص الـ Administrators المحليين:** افتح الـ CMD كمسؤول (Run as administrator) على جهاز العميل، ونفذ الأمر التالي:
   ```cmd
   net localgroup administrators
   ```
   * **النتيجة المتوقعة لنجاح المعمل:** يجب أن تجد الحساب المحلي الجديد `itadmin` ومجموعة الدومين `IT-Group` مدرجين بالكامل داخل هذه المجموعة في قائمة المخرجات.





---
## Lab 6 : Windows Firewall policies - (Allow ICMP)  allow ping using GPO 
***
---


## 🎯 Objectives
تكوين سياسة جدار حماية موحدة (Firewall Policy) عبر الـ GPO للسماح ببروتوكول **ICMPv4 (Echo Request)** والمعروف بأمر الـ **Ping**، لتمكين مسؤولي الشبكة والدعم الفني من فحص اتصال الأجهزة وحل مشكلات الشبكة مركزياً دون الحاجة للمرور على كل جهاز وتعطيل الفايروال يدوياً.

---

## ⚙️ 1. Firewall Rule Specifications (تفاصيل قاعدة الجدار الناري)

لتفعيل الـ Ping بشكل آمن وصحيح، سيتم إنشاء قاعدة واردة (**Inbound Rule**) بالخصائص التالية:
* **Rule Type:** Custom Rule (قاعدة مخصصة).
* **Protocol Type:** ICMPv4.
* **ICMP Settings:** Echo Request (طلب الصدى).
* **Action:** Allow the connection (السماح بالاتصال).
* **Profiles:** Domain, Private (ويُفضل استثناء Public للأمان).

---

## 🚀 2. GPO Implementation Steps (خطوات التنفيذ والملاحظات)

### الخطوة 1: إنشاء الـ GPO وفتح محرر السياسات
1. افتح أداة **Group Policy Management** (`gpmc.msc`).
2. اذهب إلى الـ **OU الخاصة بأجهزة الكمبيوتر** المستهدفة ⬅️ اضغط كليك يمين واختر **Create a GPO in this domain, and Link it here...**.
3. قم بتسمية السياسة الجديدة: `🛡️ Firewall-Allow-Ping-GPO` ثم اضغط كليك يمين عليها واختر **Edit**.

### الخطوة 2: بناء قاعدة الـ ICMP داخل الجدار الناري
1. داخل المحرر، توجه إلى المسار التالي:
   `Computer Configuration` ➡️ `Policies` ➡️ `Windows Settings` ➡️ `Security Settings` ➡️ `Windows Defender Firewall with Advanced Security` ➡️ `Windows Defender Firewall...`
2. اضغط كليك يمين على **Inbound Rules** (القواعد الواردة) من القائمة اليسرى ⬅️ اختر **New Rule...**.
3. في نافذة المعالج (Wizard)، اضبط الإعدادات كالتالي:
   * **Rule Type:** اختر **Custom** ثم اضغط Next.
   * **Program:** اختر **All programs** ثم اضغط Next.
   * **Protocol and Ports:** 
     * غير خيار Protocol type إلى **ICMPv4**.
     * اضغط على زر **Customize** الخاص بـ ICMP settings ⬅️ اختر **Specific ICMP types** ⬅️ ضع علامة صح على **Echo Request** ثم اضغط OK، وبعدها Next.
   * **Scope:** اتركها على **Any IP address** للـ Local والـ Remote (أو حدد شبكة الـ IT فقط لزيادة الأمان)، ثم اضغط Next.
   * **Action:** اختر **Allow the connection** ثم اضغط Next.
   * **Profile:** ضع علامة صح على **Domain** و **Private** (💡 *يُفضل إلغاء Public لحماية الجهاز في الشبكات الخارجية*)، ثم اضغط Next.
   * **Name:** سمّ القاعدة باسم واضح مثل: `🔒 Allow-ICMPv4-In (Ping)` ثم اضغط **Finish**.

---

## 📝 💡 ملاحظات هامة وقواعد التطبيق (Crucial Lab Notes)

* **لماذا قسم الـ Computer؟** إعدادات جدار الحماية (Firewall) هي إعدادات أمنية تؤثر على كينونة الجهاز نفسه بغض النظر عن المستخدم الذي يسجل دخوله؛ لذلك يتم تطبيقها إجبارياً في قسم `Computer Configuration`.
* **الاعتبارات الأمنية (Security Risk):** بروتوكول ICMP (الـ Ping) أداة ممتازة لعمل الـ Troubleshooting، ولكن تركها مفتوحة بالكامل في الشبكات العامة (Public) يسهل على المهاجمين عمل مسح للشبكة والتعرف على الأجهزة النشطة (Network Reconnaissance / OSINT)؛ لذلك قمنا بقصر القاعدة على بروفايل الـ Domain والـ Private فقط كإجراء وقائي.

---

## 🔍 3. Verification & Testing (طرق الفحص والاختبار التقني)

طبق السياسات فوراً عن طريق فتح الـ CMD كمسؤول على جهاز العميل وكتابة الأمر:
```bash
gpupdate /force
```

### 1. الفحص من جهاز آخر (Black-box Testing)
* قبل تطبيق السياسة: عند عمل `ping [Client_IP]` من جهاز الـ Domain Controller، سيعطي `Request timed out` (لأن الفايروال يحظر الحزم).
* بعد تطبيق السياسة: بمجرد تحديث الـ GPO، سيبدأ العميل في الرد فوراً وتظهر رسالة الـ `Reply from...`.

### 2. الفحص من داخل جدار حماية العميل (GUI Verification)
* افتح أداة الفايروال المتقدمة على جهاز العميل (`wf.msc`).
* اذهب إلى **Inbound Rules** وابحث عن القاعدة التي أنشأتها `Allow-ICMPv4-In (Ping)`.
* **مؤشر النجاح:** يجب أن تجد القاعدة تظهر باللون الأخضر (مفعلة) ومصحوبة بأيقونة **قفل صغير**، مما يعني أنها مفرودة مركزياً عبر الـ GPO ولا يمكن للمستخدم المحلي تعديلها أو تعطيلها.






---
## LAB 7: Hide Control panel Items, Disable Run, CMD, Shortcut URL, Backup GPO
***
---


## 🎯 Objectives
تطبيق مجموعة من السياسات الأمنية المتقدمة لتقييد واجهة المستخدم ومنع تشغيل الأدوات الحساسة (إخفاء عناصر من الـ Control Panel، تعطيل نافذة Run، حظر الـ CMD، ومنع تشغيل اختصارات روابط الإنترنت الضارة .url)، بالإضافة إلى تعلم كيفية تأمين السياسات وأخذ نسخة احتياطية كاملة منها (GPO Backup) لاستعادتها في حالات الطوارئ.

---

## ⚙️ 1. Policies & Administrative Template Paths

تجد أدناه المسارات الدقيقة داخل محرر السياسات (Group Policy Management Editor) لكل متطلب:

* **Hide Control Panel Items (إخفاء عناصر محددة من لوحة التحكم):**
  * **المسار:** `User Configuration` ➡️ `Administrative Templates` ➡️ `Control Panel`
  * **السياسة:** `Hide specified Control Panel items` ➡️ اجعلها **Enabled** ➡️ اضغط على **Show** وأضف أسماء العناصر البرمجية (Canonical Names) المراد حظرها (مثال: `Microsoft.Mouse` أو `Microsoft.NetworkAndSharingCenter`).

* **Disable Run (تعطيل أمر التشغيل السريع):**
  * **المسار:** `User Configuration` ➡️ `Administrative Templates` ➡️ `Start Menu and Taskbar`
  * **السياسة:** `Remove Run menu from Start Menu` ➡️ اجعلها **Enabled** (هذا الإجراء يعطل اختصار `Win + R` تماماً).

* **Disable CMD (منع الوصول لموجه الأوامر):**
  * **المسار:** `User Configuration` ➡️ `Administrative Templates` ➡️ `System`
  * **السياسة:** `Prevent access to the command prompt` ➡️ اجعلها **Enabled** ➡️ (💡 اختياري: اجعل خيار *Disable the command prompt script processing* بقيمة **No** إذا كنت تريد السماح بتشغيل سكريبتات الـ Logon/Startup الخلفية مع منع واجهة المستخدم).

* **Disable Shortcut URL (حظر تشغيل اختصارات الروابط الخارجية):**
  * لحظر تشغيل ملفات الروابط الاختصارية (`.url`) التي تستخدم في هجمات التصيد والاختراق، نستخدم سياسات تقييد البرمجيات:
  * **المسار:** `User Configuration` ➡️ `Policies` ➡️ `Windows Settings` ➡️ `Security Settings` ➡️ `Software Restriction Policies`
  * **طريقة التفعيل:** كليك يمين على *Software Restriction Policies* واختر **New Software Restriction Policies** ➡️ اذهب إلى مجلد **Additional Rules** ➡️ كليك يمين واختر **New Path Rule**.
  * **الإعداد:** في خانة Path اكتب `*.url` واجعل الـ Security Level بقيمة **Disallowed**.

---

## 🚀 2. GPO Implementation Steps (خطوات التنفيذ)

### أولاً: إنشاء الـ GPO وتطبيق القيود
1. افتح أداة **Group Policy Management** عبر الأمر `gpmc.msc`.
2. اضغط كليك يمين على الـ **OU المستهدفة للمستخدمين** ⬅️ اختر **Create a GPO in this domain, and Link it here...**.
3. قم بتسميتها باسم واضح: `🔒 Advanced-User-Hardening-Policy` ثم اضغط كليك يمين عليها واختر **Edit**.
4. قم بالدخول وتطبيق السياسات الأربعة المذكورة في جدول المسارات بالأعلى بدقة، ثم اغلق المحرر.

### ثانياً: عمل نسخة احتياطية للسياسة (Backup GPO)
لحماية مجهودك وضمان إمكانية استعادة السياسة في أي وقت:
1. من الواجهة الرئيسية لأداة `gpmc.msc` توجه إلى مجلد **Group Policy Objects** (حيث تتواجد جميع السياسات غير المرتبطة).
2. ابحث عن السياسة التي أنشأتها `Advanced-User-Hardening-Policy`.
3. اضغط عليها كليك يمين واختر **Back Up...**.
4. في النافذة التي تظهر لك:
   * **Location:** اختر المجلد الذي تريد حفظ النسخة الاحتياطية بداخله على الخادم (مثال: `C:\GPO_Backups`).
   * **Description:** اكتب وصفاً مختصراً للنسخة (مثال: `Backup for Lab 7 - Hardening Rules 2026`).
5. اضغط على زر **Back Up**، وانتظر حتى تظهر رسالة النجاح `Succeeded` ثم اضغط **OK**.

---

## 📝 💡 Crucial Lab Notes (ملاحظات المعمل)

* **الأهمية السيبرانية لحظر الـ URL Shortcuts:** ملفات الامتداد `.url` أصبحت تُستغل بشكل كبير جداً من قِبل المخترقين لتمرير برمجيات خبيثة وتحميل ملفات ضارة بمجرد ضغط المستخدم عليها؛ حظرها عبر الـ *Software Restriction Policies* يمثل خط دفاع ممتاز داخل الدومين.
* **تأثير تعطيل الـ Run والـ CMD:** هذه القيود تمنع المستخدم العادي من الوصول للأدوات المتقدمة للنظام، مما يقلل من فرص العبث بالإعدادات أو محاولة تشغيل أوامر برمجية غير مصرح بها محلياً.

---

## 🔍 3. Verification & Testing (طرق الفحص والاختبار)

لتطبيق السياسات فوراً، افتح الـ CMD على جهاز العميل ونفذ الأمر:
```bash
gpupdate /force
```
*قم بعمل تسجيل خروج (Logoff) ثم دخول مرة أخرى لتفعيل قيود واجهة المستخدم:*

1. **اختبار الـ Run:** اضغط على زر `Windows + R` في لوحة المفاتيح؛ يجب أن تظهر لك رسالة خطأ تفيد بأن العملية تم إلغاؤها بسبب القيود المفروضة على الكمبيوتر.
2. **اختبار الـ CMD:** جرب فتح موجه الأوامر؛ ستفتح النافذة وتظهر لك رسالة فورية: `The command prompt has been disabled by your administrator` وتغلق تلقائياً عند الضغط على أي زر.
3. **اختبار الـ Control Panel:** افتح لوحة التحكم وتأكد من اختفاء العناصر التي قمت بتحديدها (مثل الفأرة أو مركز الشبكة والمشاركة).
4. **اختبار الـ Shortcut URL:** جرب إنشاء ملف اختصار إنترنت (Internet Shortcut) ينتهي بامتداد `.url` واضغط عليه؛ يجب أن يمنع النظام تشغيله وتظهر رسالة تفيد بأن البرمجية محظورة بوضع السياسة الأمنية.