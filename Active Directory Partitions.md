

دليل شامل لشرح الأقسام المنطقية لقاعدة بيانات **Active Directory** (`NTDS.dit`)، ونطاق مزامنة كل قسم (**Replication Scope**)، والأدوات المستخدمة في إدارة كل منها.

---

## 📋 جدول المحتويات (Table of Contents)
- [📌 نبذة عن أقسام قاعدة البيانات (Overview)](#-نبذة-عن-أقسام-قاعدة-البيانات-overview)
- [1️⃣ قسم المخطط (Schema Partition)](#1️⃣-قسم-المخطط-schema-partition)
- [2️⃣ قسم التكوين (Configuration Partition)](#2️⃣-قسم-التكوين-configuration-partition)
- [3️⃣ قسم الدومين (Domain Partition)](#3️⃣-قسم-الدومين-domain-partition)
- [4️⃣ قسم التطبيقات (Application Partition)](#4️⃣-قسم-التطبيقات-application-partition)
- [🌐 الفهرس الشامل (Global Catalog)](#-الفهرس-الشامل-global-catalog)
- [📊 جدول مقارنة الأقسام (Summary Table)](#-جدول-مقارنة-الأقسام-summary-table)

---

## 📌 نبذة عن أقسام قاعدة البيانات (Overview)

تتكون قاعدة بيانات Active Directory من أربعة أقسام رئيسية تُسمى **Partitions** أو **Naming Contexts (NC)**. تُقسم هذه البيانات منطقياً لضمان كفاءة المزامنة (**Replication**) بين خوادم الدومين دون استهلاك مفرط للشبكة.

```text
┌────────────────────────────────────────────────────────┐
│            Active Directory Database                   │
│ ┌────────────────────────────────────────────────────┐ │
│ │                  Schema Partition                  │ │ ──► Forest-wide
│ ├────────────────────────────────────────────────────┤ │
│ │               Configuration Partition              │ │ ──► Forest-wide
│ ├────────────────────────────────────────────────────┤ │
│ │                  Domain Partition                  │ │ ──► Domain-wide
│ ├────────────────────────────────────────────────────┤ │
│ │                Application Partition               │ │ ──► Forest / Domain Zone
│ └────────────────────────────────────────────────────┘ │
│                     [ Global Catalog ]                 │
└────────────────────────────────────────────────────────┘
```

---

## 1️⃣ قسم المخطط (Schema Partition)

يحتوي على التعريفات الهيكلية والأصلية لجميع الكائنات والخصائص المسموح بإنشائها داخل الـ Forest.

* **نطاق المزامنة (Scope):** يُنسخ على مستوى الغابة بالكامل (**Forest-wide**).
* **المحتويات:** يعرّف فئات الكائنات (**Object Classes**) مثل Users و Computers، وخصائصها (**Attributes**) مثل Password و Email.
* **طريقة الإظهار والإدارة:**
  1. فتح أداة **Run** وتسجيل المكتبة البرمجية عبر الأمر:
     ```cmd
     regsvr32 schmmgmt.dll
     ```
  2. فتح وحدة التحكم النصية عبر أمر `mmc` في **Run**.
  3. اختيار **Add or Remove Snap-in** وإضافة وحدة **Active Directory Schema**.

---

## 2️⃣ قسم التكوين (Configuration Partition)

يخزن معلومات البنية التحتية والمخطط الفيزيائي للشبكة على مستوى الغابة.

* **نطاق المزامنة (Scope):** يُنسخ على مستوى الغابة بالكامل (**Forest-wide**).
* **المحتويات:** مواقع الشبكة (**Sites**)، شبكات الفرع (**Subnets**)، الخوادم المتاحة (**Servers** مثل `PDC` و `ADC`)، والخدمات المربوطة بالدومين.
* **أداة الإدارة:** يتم التحكم به عبر أداة **Active Directory Sites and Services**.
* **مثال المسار:** `Default-First-Site-Name` ⬅️ `Servers` ⬅️ (`PDC`, `ADC`).

---

## 3️⃣ قسم الدومين (Domain Partition)

يحتوي على كافة البيانات الفعلية اليومية الخاصة بالدومين المحدد.

* **نطاق المزامنة (Scope):** يُنسخ فقط بين خوادم الدومين داخل نفس الدومين (**Domain-wide**).
* **المحتويات:** كائنات المستخدمين (**Users**)، أجهزة الكمبيوتر (**Computers**)، المجموعات (**Groups**)، الوحدات التنظيمية (**OUs**)، والحسابات البرمجية.
* **أداة الإدارة:** يتم التحكم به عبر أداة **Active Directory Users and Computers** (ADUC).

---

## 4️⃣ قسم التطبيقات (Application Partition)

قسم مرن تم إنشاؤه لاستضافة البيانات الخاصة بتطبيقات معينة دون الحاجة لمزامنتها مع بقية بيانات الدومين الأساسية.

* **نطاق المزامنة (Scope):** يمكن تخصيصه ليكون على مستوى الغابة (**Forest Zone**) أو على مستوى الدومين (**Domain Zone**).
* **المحتويات:** بيانات التطبيقات والخدمات المستقلة، وأبرز مثال عليها هي مناطق خادم الـ DNS المدمجة (**Active Directory Integrated Zone**).
* **أداة الإدارة:** يتم التحكم به عبر أداة **DNS Manager** أو أدوات CLI الخاصة بالتطبيقات.

---

## 🌐 الفهرس الشامل (Global Catalog)

الـ **Global Catalog (GC)** هو خادم دومين يمتلك نسخة قراءة فقط (**Read-Only Partial Attribute Set**) من جميع الكائنات الموجودة في كافة الأقسام والدومينات داخل الغابة بالكامل (**Forest**)، مما يسرّع عمليات البحث والمصادقة عبر الدومينات المختلفة.

---

## 📊 جدول مقارنة الأقسام (Summary Table)

| اسم القسم (Partition) | نطاق المزامنة (Replication Scope) | أداة الإدارة الرئيسيّة (Management Tool) | أبرز المحتويات |
| :--- | :--- | :--- | :--- |
| **Schema Partition** | Forest-wide | MMC Snap-in (`schmmgmt.dll`) | القواعد الهيكلية والأنواع والخصائص |
| **Configuration Partition** | Forest-wide | Active Directory Sites and Services | مواقع الشبكة، السيرفرات (`PDC`/`ADC`) والتضاريس |
| **Domain Partition** | Domain-wide | Active Directory Users and Computers | المستخدمين، الأجهزة، المجموعات، و الـ OUs |
| **Application Partition** | Forest / Domain Zone | DNS Manager / Custom Tools | مناطق DNS وسجلات التطبيقات المدمجة |