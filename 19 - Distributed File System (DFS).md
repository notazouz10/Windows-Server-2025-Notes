

دليل شامل لشرح وإدارة خدمة **DFS** بنوعيها (**DFS Namespaces** و **DFS Replication**) وتكاملها مع **Active Directory Sites and Services** لتوحيد مجلدات المشاركة ومزامنتها عبر السيرفرات والفروع.

---

## 📋 جدول المحتويات (Table of Contents)
- [📌 نبذة عن الخدمة (Overview)](#-نبذة-عن-الخدمة-overview)
- [1️⃣ مساحات الأسماء (DFS Namespaces)](#1️⃣-مساحات-الأسماء-dfs-namespaces)
- [2️⃣ النسخ التكراري (DFS Replication)](#2️⃣-النسخ-التكراري-dfs-replication)
- [🛡️ خصائص وميزات إضافية (ABE & Site Awareness)](#️-خصائص-وميزات-إضافية-abe--site-awareness)

---

## 📌 نبذة عن الخدمة (Overview)

خدمة **Distributed File System (DFS)** هي دور فرعي (**Role Service**) يتبع دور **File Server** على نظام Windows Server. تتيح تجميع المجلدات المشاركة (**Shared Folders**) الموزعة على سيرفرات مختلفة وإبرازها للمستخدمين ضمن مسار شجري موحد، مع مزامنة البيانات بين تلك السيرفرات لضمان التوافر العالي (**High Availability**).

---

## 1️⃣ مساحات الأسماء (DFS Namespaces)

تُستخدم **DFS Namespaces** لإخفاء التعقيد المادي للشبكة وتوفير مسار وهمي موحد للوصول للملفات بدلاً من حفظ أسماء السيرفرات متعددة.

```text
                           [ Domain: Test.local ]
                        Unified DFS Path (Run / Explorer):
                         \\Test.local\FileServer
                                    │
         ┌──────────────────────────┼──────────────────────────┐
         ▼                          ▼                          ▼
    [ Folder: HR ]           [ Folder: HRD1 ]           [ Folder: HRS2 ]
    Target Server:           Target Server:             Target Server:
    \\PDC\HR                 \\FSRV1\HRD1               \\FSRV2\HRS2
    (IP: 192.168.1.2)        (Server 1)                 (Server 2)
```

### ⚙️ آلية الإعداد والربط:
1. **تثبيت الميزة:** إضافة خدمة `DFS Namespaces` كـ Role Service تحت `File and Storage Services` على السيرفر (سواء GUI أو Core).
2. **إنشاء Namespace:** إقامة مسار رئيسي على مستوى الدومين باسم:
   ```cmd
   \\Test.local\FileServer
   ```
3. **إضافة المجلدات والمستهدفات (Folders & Targets):**
   * إضافة المجلد `HR` وتوجيهه إلى المسار الفعلي: `\\PDC\HR` (الموجود على سيرفر PDC صاحب IP: `192.168.1.2`).
   * إضافة المجلد `HRD1` وتوجيهه إلى المسار الفعلي: `\\FSRV1\HRD1` على السيرفر الأول (`FSRV1`).
   * إضافة المجلد `HRS2` وتوجيهه إلى المسار الفعلي: `\\FSRV2\HRS2` على السيرفر الثاني (`FSRV2`).
4. **تجربة الوصول (Client View):** عندما يفتح المستخدم أمر **Run** ويكتب `\\Test.local\FileServer` يظهر له المجلد الموحد محتوياً على `[HR]`، `[HRD1]`، و `[HRS2]` دون الحاجة لمعرفة IP أو اسم السيرفر الفعلي المخزن عليه الملفات.

---

## 2️⃣ النسخ التكراري (DFS Replication)

تسمح ميزة **DFS Replication (DFSR)** بمزامنة المجلدات والملفات تلقائياً بين أكثر من خادم (مثل `FSRV1` و `FSRV2` و `PDC`) لضمان عدم انقطاع الخدمة وتبادل التحديثات فوراً.

* **مزامنة المجلدات المشتركة:** يتم ربط المجلد `HRD1` على `FSRV1` بالمجلد `HRS2` على `FSRV2` في زمرة مزامنة واحدة (**Replication Group**).
* **توزيع البيانات:** في حال تعديل أي ملف في سيرفر `FSRV1` يتم نسخه تكرارياً إلى `FSRV2` والعكس تلقائياً.

---

## 🛡️ خصائص وميزات إضافية (ABE & Site Awareness)

### 🔒 1. Access-Based Enumeration (ABE):
* عند تفعيل ميزة **ABE** على الـ Namespace، لن يظهر للمستخدم إلا المجلدات والملفات التي يملك عليها صلاحيات وصول قراءة أو كتابة (**Permissions**) فقط، بينما تُخفى بقية المجلدات غير المصرح له بها لزيادة الأمان.

### 🌍 2. الربط مع Active Directory Sites and Services (Site Awareness):
تقوم خدمة DFS بتوجيه المستخدم تلقائياً لأقرب خادم ملفات بناءً على الموقع الجغرافي للشبكة (**AD Site**) لتقليل استهلاك الباندويث:

```text
[ Active Directory Sites and Services ]
 ├── Default-First-Site-Name  ---> (Contains: PDC , SRV1)
 └── Alex (Branch Site)        ---> (Contains: SRV2)
```

* **المستخدمون في المقر الرئيسي (Default Site):** يتم توجيههم تلقائياً للسيرفرات المحلية القريبة منهم (`PDC` أو `SRV1`).
* **المستخدمون في فرع الإسكندرية (Alex Site):** يتم توجيههم تلقائياً للتحميل المباشر من السيرفر المحلي بفرعهم (`SRV2`).