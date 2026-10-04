***
How to Join `Domain` as `Client`

`Preparing to join the domain`
* Installing the `operating system` on the device
* change date , time and time zone . 
* IP configurations manual and disable IPV6
* Change Computer Name
* Check `Client` show `Server` > open `cmd` > Ping `ip server`
* Check `DNS` Server 
---
`Join Machine on Domain`
* from `Settings` > `System` > `About` > `Change Name`
* `Win+R` > `sysdm.cpl`
* Choose `Domain` > Write `Domain Name` 
* Any `User` inside Active Directory He can join the domain
* Write `username` and `Password`
* Sign in `local machine` Write in `username` `.\namePC`
* pass user Mohamed Nabil `P@$Sw0rd`

***
After Machine Join  `Domain`
***
* Create Computer account in Computers Container in `AD` 
* Domain Users group , member of `local Users group`    `TEST\Domain Users`
* Domain Admins group , member of `local administrators group`  `TEST\Domain Admins`

*** 
## خطوات انضمام جهاز إلى النطاق (How to Join a Computer to a Domain)

لدخول جهاز كمبيوتر داخل النطاق ليصبح تحت إدارة الـ Domain Controller، يجب تنفيذ خطوتين بالترتيب: ضبط الـ DNS أولاً (حتى يرى الجهاز السيرفر)، ثم إتمام عملية الـ Join.

---

### الخطوة 1: ضبط إعدادات الـ DNS (خطوة إجبارية)
بدون هذه الخطوة، لن يتمكن الجهاز من العثور على الـ Domain Controller في الشبكة.
1. افتح نافذة إعدادات الشبكة بالضغط على `Win + R` واكتب الأمر السريع: `ncpa.cpl`.
2. اضغط كليك يمين على كارت الشبكة الفعّال واختر **Properties**.
3. اضغط مرتين على **Internet Protocol Version 4 (TCP/IPv4)**.
4. في خانة **Preferred DNS server**، اكتب **عنوان الـ IP الخاص بالـ Domain Controller** (مثال: `192.168.1.10`).
5. اضغط **OK** لحفظ الإعدادات.

---

### الخطوة 2: الانضمام للنطاق (Joining the Domain)
1. افتح نافذة خصائص النظام بالضغط على `Win + R` واكتب الأمر: `sysdm.cpl`.
2. من تبويب **Computer Name**، اضغط على زر **Change**.
3. في نافذة التعديل، قم بتغيير الاختيار من *Workgroup* إلى **Domain**.
4. اكتب اسم النطاق الخاص بك (مثال: `TEST.local`) ثم اضغط **OK**.
5. ستظهر لك نافذة تطلب صلاحيات؛ أدخل اسم المستخدم وكلمة المرور لحساب يمتلك صلاحية ضم الأجهزة (مثل الـ **Domain Admin**).
6. ستظهر لك رسالة النجاح: *"Welcome to the TEST.local domain"*.
7. اضغط **OK**، ثم وافق على إعادة تشغيل الجهاز (**Restart**) لتفعيل الانضمام.

---

> 🔒 **معلومة أمنية (Cybersecurity Tip):** افتراضياً في بيئة الـ Active Directory، يسمح النظام لأي مستخدم عادي (**Domain User**) بضم حتى **10 أجهزة** إلى النطاق بدون صلاحيات أدمن (تُعرف خاصية بـ `MachineAccountQuota`). في البيئات المحمية (Hardened Environments)، يتم تعديل هذه القيمة إلى `0` لمنع المهاجمين من إدخال أجهزة خبيثة داخل النطاق بسهولة.

---

# إدارة الأمان في Active Directory: عزل صلاحيات المسؤول المحلي (Local Admin Isolation)

## 📌 نظرة عامة (Overview)
في الشبكات المؤسسية المعتمدة على **Active Directory**، يمثل إعطاء المستخدمين صلاحيات إدارية واسعة خطراً أمنياً كبيراً. لتطبيق **مبدأ الصلاحيات الأقل (Principle of Least Privilege)**، نقوم بعزل الصلاحيات الإدارية بحيث يمتلك المستخدم صلاحية مدير (Admin) على جهازه الشخصي فقط لتأدية مهامه، بينما يظل مستخدماً عادياً بلا أي صلاحيات مرتفعة بمجرد انتقاله لأي جهاز آخر داخل الدومين.

هذا التوثيق يمثل إثبات مفهوم عملي (**Proof of Concept**) لكيفية عزل وتحديد نطاق الصلاحيات المحلية على أجهزة الدومين.

---

## 🧪 التجربة العملية وإثبات المفهوم (Hands-On Lab)

تم إجراء تجربة عملية للتحقق من سلوك عزل الصلاحيات بين جهازين (Workstation A و Workstation B) منضمين إلى الدومين، وكانت النتائج كالتالي:

### ❌ المرحلة الأولى: محاولة رفع الصلاحيات غير المصرح بها
* **الإجراء:** تم تسجيل الدخول إلى **Workstation A** باستخدام حساب مستخدم دومين عادي (Standard User)، ومحاولة إضافته يدوياً إلى مجموعة الـ `Administrators` المحلية.
* **النتيجة:** **تم رفض الوصول (Access Denied).** طلب النظام صلاحيات مسؤول شبكة أو أدمن محلي، مما يثبت أن المستخدم العادي لا يمكنه التلاعب بالمجموعات الأمنية للجهاز.

### 🔑 المرحلة الثانية: رفع الصلاحيات محلياً بواسطة المسؤول
* **الإجراء:** تم تسجيل الدخول إلى **Workstation A** بحساب يمتلك صلاحيات إدارة (Administrator)، وتمت إضافة مستخدم الدومين العادي بنجاح إلى مجموعة الـ `Administrators` المحلية الخاصة بهذا الجهاز فقط.
* **النتيجة:** **نجاح العملية.** عند إعادة تسجيل الدخول باليوزر العادي، أصبح يمتلك كامل صلاحيات الإدارة (Local Admin) **حصرياً** على هذا الجهاز.

### 🔒 المرحلة الثالثة: اختبار وتأكيد عزل الصلاحيات
* **الإجراء:** تم الانتقال إلى **Workstation B** (جهاز آخر داخل نفس الدومين)، وتسجيل الدخول بنفس حساب مستخدم الدومين.
* **النتيجة:** **تأكيد العزل الأمنى.** تم تجريد المستخدم من أي صلاحيات إدارية فوراً، وتعامَل معه النظام كمستقبل عادي تماماً (Standard User)، مما يثبت نجاح عزل الصلاحية داخل حدود الجهاز الأول.

---

## 🧠 التحليل التقني: لماذا يحدث هذا؟

السر يكمن في مكان تخزين الصلاحية:
1. عند إضافة يوزر الدومين إلى مجموعة الـ `Administrators` على جهاز معين، فإن هذا التعديل يُحفظ داخل قاعدة بيانات الـ **SAM (Security Account Manager)** المحلية الخاصة بهذا الجهاز فقط.
2. قاعدة بيانات الـ **Active Directory** المركزية على الـ Domain Controller لا تتأثر بهذا التعديل، ويبقى حساب المستخدم مصنفاً كـ `Standard User` على مستوى الشبكة.
3. وبالتالي، فإن رمز الوصول الإداري (Administrative Access Token) يكون **مرتبطاً بالجهاز (Machine-Bound)** ولا ينتقل مع اليوزر إلى أجهزة أخرى.

---

![[Screenshot (50).png]]
## 🛠️ طرق التنفيذ داخل البيئات المؤسسية

### الطريقة الأولى: يدوياً (للأجهزة المعزولة أو المعامل)
تتم من خلال إدارة الجهاز محلياً بعد انضمامه للدومين:
```powershell
# تشغيل الأمر بصلاحيات مسؤول على الجهاز المستهدف
Add-LocalGroupMember -Group "Administrators" -Member "DOMAIN\Domain_User"

