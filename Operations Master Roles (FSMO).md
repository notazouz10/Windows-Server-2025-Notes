

دليل شامل لشرح أدوار العمليات الرئيسية **FSMO (Flexible Single Master Operations)** الخمسة في **Active Directory**، وأدوات الإدارة الخاصة بكل دور، بالإضافة إلى كيفية معرفة الخادم الحامل للأدوار، وآلية نقلها (**Transfer**) أو انتزاعها (**Seize**).

---

## 📋 جدول المحتويات (Table of Contents)
- [📌 نبذة عن أدوار FSMO (Overview)](#-نبذة-عن-أدوار-fsmo-overview)
- [🏢 1. أدوار على مستوى الدومين (Domain-Wide Roles)](#-1-أدوار-على-مستوى-الدومين-domain-wide-roles)
- [🌲 2. أدوار على مستوى الغابة (Forest-Wide Roles)](#-2-أدوار-على-مستوى-الغابة-forest-wide-roles)
- [🔍 الاستعلام عن أطراف الأدوار (Query FSMO Roles)](#-الاستعلام-عن-أطراف-الأدوار-query-fsmo-roles)
- [🔄 طرق نقل وسحب الأدوار (Transfer vs. Seize)](#-طرق-نقل-وسحب-الأدوار-transfer-vs-seize)
- [📊 جدول ملخص الأدوار وأدوات الإدارة (Summary Table)](#-جدول-ملخص-الأدوار-وأدوات-الإدارة-summary-table)

---

## 📌 نبذة عن أدوار FSMO (Overview)

تُعرف بـ **Flexible Single Master Operations (FSMO)**، وهي 5 أدوار خاصة وحرجة في بيئة **Active Directory** لا يمكن تنفيذها إلا بواسطة خادم دومين واحد (**Domain Controller**) في نفس الوقت لمنع التعارضات وإدارة التغييرات الحساسة.

---

## 🏢 1. أدوار على مستوى الدومين (Domain-Wide Roles)

توجد هذه الأدوار الثلاثة بشكل مستقل داخل كل دومين في الـ Forest، وتُدار جميعها عبر أداة **Active Directory Users and Computers**:

1. **PDC Emulator:**
   * المسؤول عن معالجة تغييرات كلمات السر فوراً، ومزامنة الوقت بين الأجهزة (Time Synchronization)، وتوافقية الأنظمة القديمة.
2. **RID Master (Relative Identifier Master):**
   * كفيل بتوزيع مجمعات الـ RID (RID Pools) على جميع الـ DCs لإنشاء كائنات جديدة (Users, Groups, Computers) بأرقام SID فريدة.
3. **Infrastructure Master:**
   * مسؤول عن تحديث الإشارات المرجعية والحسابات وتأكيد المجموعات بين الدومينات المختلفة.

---

## 🌲 2. أدوار على مستوى الغابة (Forest-Wide Roles)

توجد نسخة واحدة فقط من هذين الدورين على مستوى الغابة بالكامل (**Forest**):

4. **Domain Naming Master:**
   * مسؤول عن إضافة أو إزالة الدومينات والمجالات الفرعية داخل الـ Forest.
   * **أداة الإدارة:** يُدار عبر أداة **Active Directory Domains and Trusts**.

5. **Schema Master:**
   * مسؤول عن إجراء التعديلات على بنية وقواعد قاعدة بيانات الـ Active Directory (الخصائص والكائنات).
   * **أداة الإدارة:** يُدار عبر أداة **Active Directory Schema**.
   * **خطوات إظهار الأدوات:**
     1. فتح نافذة **Run** وتشغيل الأمر لتسجيل الملف البرمجي:
        ```cmd
        regsvr32 schmmgmt.dll
        ```
     2. فتح **Run** وكتابة `mmc` لفتح وحدة التحكم النصية.
     3. اختيار **File** ⬅️ **Add or Remove Snap-in** ⬅️ إضافة **Active Directory Schema**.

---

## 🔍 الاستعلام عن أطراف الأدوار (Query FSMO Roles)

لمعرفة الخادم الذي يحمل أدوار FSMO الخمسة حالياً، يُمكن تشغيل الأمر التالي في موجه الأوامر (CMD):

```cmd
netdom query fsmo
```

---

## 🔄 طرق نقل وسحب الأدوار (Transfer vs. Seize)

```text
                           [ FSMO Roles Operations ]
                                       │
            ┌──────────────────────────┴──────────────────────────┐
            ▼                                                     ▼
 1️⃣ Transfer (Graceful)                                2️⃣ Seize (Forced)
 [ PDC Online ] ─── Transfer Roles ──► [ ADC ]        [ PDC Down / Dead ] ─── Seize Roles ──► [ ADC ]
  (نقل طبيعي للأدوار قبل تنزيل رتبة PDC)              (استيلاء بالقوة باستخدام ntdsutil)
```

### 1️⃣ Transfer (النقل المنظم):
* **الحالة:** يُستخدم عندما يكون السيرفر الرئيسي (`PDC`) يعمل بشكل طبيعي ويتواصل مع باقي السيرفرات (`PDC Online`).
* **الاستخدام:** عند رغبة المسؤول في عمل تنزيل رتبة للسيرفر (`Demote`) أو إجراء صيانة دورية.
* **النتيجة:** يتم نقل الأدوار بسلاسة إلى الخادم البديل (`ADC`) دون فقدان بيانات.

### 2️⃣ Seize (الانتزاع بالقوة):
* **الحالة:** يُستخدم فقط في حالة تلف أو سقوط السيرفر الرئيسي تماماً وعدم إتاحة الخادم للخدمة (`PDC Down / Dead`).
* **الوسيلة:** يتم تنفيذ العملية عبر أداة الأوامر **`ntdsutil`** لاستيلاء خادم `ADC` على الأدوار بالقوة.
* **خطوات الأمر:**
  ```cmd
  ntdsutil
  roles
  connections
  connect to server ADC
  quit
  seize [role-name]
  ```

---

## 📊 جدول ملخص الأدوار وأدوات الإدارة (Summary Table)

| اسم الدور (FSMO Role) | النطاق (Scope) | أداة الإدارة (Management Snap-in) |
| :--- | :--- | :--- |
| **PDC Emulator** | Domain-wide | Active Directory Users and Computers |
| **RID Master** | Domain-wide | Active Directory Users and Computers |
| **Infrastructure Master** | Domain-wide | Active Directory Users and Computers |
| **Domain Naming Master** | Forest-wide | Active Directory Domains and Trusts |
| **Schema Master** | Forest-wide | Active Directory Schema (`regsvr32 schmmgmt.dll`) |