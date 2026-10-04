

دليل شامل لشرح وإدارة خادم الدومين الإضافي **(Additional Domain Controller - ADC)** وتكامله مع الخادم الرئيسي **(PDC)**، مع توضيح آلية مزامنة قاعدة البيانات وإدارة أدوار **FSMO** الخمسة (Transfer & Seize).

---

## 📋 جدول المحتويات (Table of Contents)
- [📌 نبذة عن الخدمة (Overview)](#-نبذة-عن-الخدمة-overview)
- [⚙️ بنية الشبكة والمزامنة (Architecture & Replication)](#️-بنية-الشبكة-والمزامنة-architecture--replication)
- [👑 أدوار FSMO الخمسة (FSMO Master Roles)](#-أدوار-fsmo-الخمسة-fsmo-master-roles)
- [🔄 إدارة نقل ونزع الأدوار (Transfer vs. Seize)](#-إدارة-نقل-ونزع-الأدوار-transfer-vs-seize)

---

## 📌 نبذة عن الخدمة (Overview)

خادم الدومين الإضافي **Additional Domain Controller (ADC)** هو خادم إضافي يتم ترقيته داخل نفس الدومين ليعمل إلى جانب خادم الدومين الرئيسي **Primary Domain Controller (PDC)**.

### 💡 الأهداف والمميزات الرئيسية:
* **التوافر العالي (High Availability & Fault Tolerance):** ضمان استمرار عمل الشبكة وتسجيل دخول المستخدمين في حال تعطل الخادم الرئيسي.
* **توزيع الأحمال (Load Balancing):** تخفيف الضغط عن الـ PDC وتوزيع عمليات المصادقة (Authentication).
* **المزامنة الثنائية (Multi-Master Replication):** مزامنة التغييرات تلقائياً في قاعدة بيانات Active Directory بين السيرفرين.

---

## ⚙️ بنية الشبكة والمزامنة (Architecture & Replication)

```text
                           [ Domain: Test.local ]
                                     │
           ┌─────────────────────────┴─────────────────────────┐
           ▼                                                   ▼
┌───────────────────────────┐                         ┌───────────────────────────┐
│ PDC (Primary Domain Ctrl) │ ◄──── Replication ────► │  ADC (Additional DC)      │
│ IP: 192.168.1.2           │   (Active Directory     │  IP: 192.168.1.3          │
│ Role: AD DS               │        Database)        │  Role: AD DS              │
│ - Users, Groups, OUs      │                         │  - Replicated Objects     │
│ - HR, Fin Departments     │                         │  - HR, Fin Departments    │
└───────────────────────────┘                         └───────────────────────────┘
```

### 📝 آلية المزامنة (AD DB Replication):
* تتم مزامنة قاعدة بيانات **Active Directory Database** بين الـ **PDC** والـ **ADC** في الاتجاهين.
* تشمل المزامنة كافة الكائنات: **Users**, **Groups**, **Organizational Units (OUs)**، والمجلدات الخاصة بالمباني أو الأقسام مثل (**HR**, **Fin**).

---

## 👑 أدوار FSMO الخمسة (FSMO Master Roles)

تُعرف بـ **FSMO (Flexible Single Master Operation)**، وهي 5 أدوار خاصة في Active Directory لا يمكن أن تتواجد إلا على خادم واحد فقط في أي وقت لضمان عدم التعارض:

| نطاق الدور (Scope) | اسم الدور (FSMO Role) | الوظيفة الأساسية |
| :--- | :--- | :--- |
| **Forest-wide** | **Schema Master** | التحكم في تعديلات وتطوير بنية قاعدة بيانات الـ Active Directory. |
| **Forest-wide** | **Domain Naming Master** | المسؤول عن إضافة أو إزالة الدومينات داخل الـ Forest. |
| **Domain-wide** | **RID Master** | توزيع معرفات الأمان الفرعية (RIDs) للكائنات الجديدة (Users, Groups). |
| **Domain-wide** | **PDC Emulator** | معالجة تغييرات كلمات السر، مزامنة الوقت (Time Sync)، وتوافقية الأنظمة القديمة. |
| **Domain-wide** | **Infrastructure Master** | مزامنة الإشارات المرجعية بين الكائنات في الدومينات المختلفة. |

---

## 🔄 إدارة نقل ونزع الأدوار (Transfer vs. Seize)

تتواجد أدوار FSMO الخمسة افتراضياً على الخادم الرئيسي (**PDC**)، ولكن يمكن نقلها أو سحبها لصالح **ADC** عند الحاجة عبر إحدى الطريقتين:

```text
                          [ FSMO Roles Operations ]
                                     │
           ┌─────────────────────────┴─────────────────────────┐
           ▼                                                   ▼
 1️⃣ Transfer (Graceful)                               2️⃣ Seize (Forced)
 [ PDC Online ] ─── Transfer Roles ───► [ ADC ]       [ PDC Down / Dead ] ─── Seize Roles ───► [ ADC ]
 (نقل طبيعي للأدوار بدون مشاكل)                        (استيلاء بالقوة عند تلف السيرفر الرئيسي)
```

### 1️⃣ Transfer (النقل المباشر والسلس):
* **الحالة:** يُستخدم عندما يكون السيرفر الرئيسي (**PDC**) يعمل بشكل طبيعي (**PDC Online**).
* **الإجراء:** يتم نقل الأدوار بشكل منظم وآمن من `PDC` إلى `ADC` عبر أدوات GUI أو PowerShell أو `ntdsutil`.
* **الاستخدام:** حالات الصيانة المخططة للسيرفر الرئيسي.

### 2️⃣ Seize (الاستيلاء/السحب بالقوة):
* **الحالة:** يُستخدم فقط عند تعطل السيرفر الرئيسي تماماً أو تلفه بشكل لا يمكن إصلاحه (**PDC Down / Dead**).
* **الإجراء:** يتم انتزاع الأدوار بالقوة وإجبار الخادم التابع **ADC** على أخذ جميع أدوار FSMO الخمسة ليصبح هو الـ Domain Controller الرئيسي.
* **⚠️ تنبيه هام:** بعد عملية الـ **Seize**، لا يجوز إعادة توصيل السيرفر القديم (PDC) بالشبكة مرة أخرى أبداً دون إعادة تهيئته بالكامل لتجنب التعارض في الشبكة.