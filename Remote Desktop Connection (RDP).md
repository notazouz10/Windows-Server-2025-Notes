***

دليل مكثف لشرح وتعديل إعدادات **التحكم عن بُعد (Remote Desktop Connection)** عبر بروتوكول **RDP** داخل بيئة الدومين.

---

## 📋 جدول المحتويات (Table of Contents)
- [📌 نبذة عن البروتوكول والمنفذ (Overview & Port)](#-نبذة-عن-البروتوكول-والمنفذ-overview--port)
- [⚙️ مخطط الاتصال عبر الشبكة (Architecture Diagram)](#️-مخطط-الاتصال-عبر-الشبكة-architecture-diagram)
- [🔧 إعدادات سيرفر الهدف (PDC Configuration)](#-إعدادات-سيرفر-الهدف-pdc-configuration)
- [💻 كيفية الاتصال من جهاز العميل (Client Connection Steps)](#-كيفية-الاتصال-من-جهاز-العميل-client-connection-steps)

---

## 📌 نبذة عن البروتوكول والمنفذ (Overview & Port)

* **اسم البروتوكول:** Remote Desktop Protocol (**RDP**)
* **رقم المنفذ (Port Number):** **`3389`** (منفذ الاتصال الافتراضي للتحكم بسطح المكتب عن بُعد).
* **المتطلبات الأساسية:**
  * معرفة عنوان الـ IP الخاص بالجهاز المُراد التحكم به.
  * اسم المستخدم وكلمة السر (`Username` & `Password`) لمن يملك صلاحيات الاتصال.

---

## ⚙️ مخطط الاتصال عبر الشبكة (Architecture Diagram)

```text
[ System Admin PC ]                                               [ Target Server / PDC ]
  IP: 192.168.1.31                                                   IP: 192.168.1.2
   (Win10 Client)                                                      (Windows Server)
┌──────────────────┐               ┌──────────┐               ┌──────────────────┐
│  Run: mstsc      │ ─────────────►│  Switch  │ ─────────────►│  Allowed Port:   │
│  User & Pass     │   (LAN Net)   └──────────┘  (Firewall)   │      3389        │
└──────────────────┘                                          └──────────────────┘
```

---

## 🔧 إعدادات سيرفر الهدف (PDC Configuration)

لتجهيز خادم الهدف (مثل **PDC** صاحب الـ IP: `192.168.1.2`) لاستقبال الاتصالات البعيدة، يجب تنفيذ الخطوات التالية:

### 1️⃣ تفعيل خاصية الاتصال عن بُعد (Allow RDP):
* افتح خصائص النظام (**System Properties**) وتوجه إلى تبويب **Remote**.
* قم بتفعيل خيار **Allow remote connections to this computer**.

### 2️⃣ ضبط مصادقة مستوى الشبكة (NLA - Network Level Authentication):
* حدد أو ألغِ خيار:  
  `Allow connections only from computers running Remote Desktop with Network Level Authentication (NLA)` بناءً على متطلبات الأمان والتوافقية في شبكتك.

### 3️⃣ السماح عبر جدار الحماية (Windows Firewall Rule):
* يجب فتح المنفذ في جدار الحماية الخاص بالـ Server:  
  **`Allow RDP in Windows Firewall (Port 3389)`** لضمان عدم حظر طلبات الاتصال القادمة من الـ Switch.

---

## 💻 كيفية الاتصال من جهاز العميل (Client Connection Steps)

من جهاز مدير النظام (`System Admin` / `Win10` بـ IP: `192.168.1.31`):

1. افتح نافذة **Run** بضغط زر `Win + R`.
2. اكتب الأمر التالي واضغط **Enter**:
   ```cmd
   mstsc
   ```
3. ستظهر لك واجهة **Remote Desktop Connection**:
   * أدخل عنوان الـ IP الخاص بالسيرفر: **`192.168.1.2`**.
   * اضغط على **Connect**.
1. ادخل بيانات الاعتماد (`Username` & `Password`) الخاصة بالدومين لتسجيل الدخول إلى سطح المكتب مباشرة.


---
### LAB
***
