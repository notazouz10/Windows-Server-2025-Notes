***


دليل عملي معملي مخصص للواجهة الرسومية (**GUI-Only Practical LAB Guide**) يغطي كيفية تثبيت وإدارة دور **Print and Document Services**، إضافة وتأمين طابعات الشبكة، تفعيل عزل التعريفات (**Driver Isolation**)، ونشر الطابعات تلقائياً لكافة المستخدمين عبر **Group Policy (GPO)** باستخدام أداة **Print Management Console (`printmanagement.msc`)**.

---

## 🛠️ 1. مخطط البيئة المعملية (Lab Setup Topology)

* **Print Server / DC (`DC01`):**
  * **IP Address:** `192.168.1.10 /24`
  * **OS:** Windows Server (Domain Controller & Print Server)
  * **Management Tool:** Print Management Console (`printmanagement.msc`)
* **Network Printer (Simulated):**
  * **IP Address:** `192.168.1.200 /24`
  * **Port:** Standard TCP/IP Port (Port 9100)
* **Enterprise Client (`WIN-CLIENT01`):**
  * **IP Address:** `192.168.1.50 /24`
  * **OS:** Windows 10/11 Enterprise (Joined to Domain)

---

## 🧪 LAB Task 1: Installing Print and Document Services Role via GUI

### 🎯 الهدف:
تثبيت دور **Print and Document Services** على السيرفر الرئيسي باستخدام **Server Manager**.

### 📝 الخطوات العملية:

1. افتح برنامج **Server Manager** على السيرفر `DC01`.
2. اضغط على **Manage** في الزاوية العلوية ⬅️ اختر **Add Roles and Features**.
3. في شاشة الترحيب (Before you begin)، اضغط **Next**.
4. **Installation Type:** اختر **Role-based or feature-based installation** ⬅️ اضغط **Next**.
5. **Server Selection:** تأكد من تحديد السيرفر `DC01.corp.local` ⬅️ اضغط **Next**.
6. **Server Roles:** ضع علامة صح `☑️` على **Print and Document Services**.
   * ستظهر نافذة تُطالب بإضافة الميزات الملحقة (Add Features)، اضغط **Add Features** ⬅️ ثم اضغط **Next**.
7. **Features:** اضغط **Next** بدون تعديل.
8. **Print and Document Services:** اضغط **Next**.
9. **Role Services:**
   * تأكد من تحديد خدمة **Print Server** أساسياً.
   * (اختياري) حدد **Internet Printing** في حال الحاجة للطباعة عبر المتصفح.
10. اضغط **Next** ⬅️ ثم اضغط **Install**.
11. بعد اكتمال التثبيت، اضغط **Close**.

```
[Server Manager] ──► Add Roles and Features
     ├── Installation Type: Role-based
     ├── Server Roles: ☑️ Print and Document Services
     └── Role Services: ☑️ Print Server ──► [Install]
```

---

## 🧪 LAB Task 2: Adding & Sharing Network Printers in Print Management

### 🎯 الهدف:
إضافة طابعة شبكية عبر IP وربط تعريفاتها ومشاركتها مع جميع الأجهزة في الشبكة.

### 📝 الخطوات العملية:

1. افتح أداة الإدارة: اضغط `Win + R` واكتب **`printmanagement.msc`** ثم اضغط **Enter**.
2. من القائمة اليسرى، وسّع مجلد **Print Servers** ⬅️ وسّع اسم السيرفر الخاص بك `DC01`.
3. اضغط كليك يمين على مجلد **Printers** ⬅️ اختر **Add Printer...**.
4. في **Network Printer Installation Wizard**:
   * اختر الخيار الثاني: **Add a TCP/IP or Web Services printer by IP address or hostname**.
   * اضغط **Next**.
5. **Printer Conditions:**
   * **Type of Device:** اختر *TCP/IP Device*.
   * **Printer name or IP address:** اكتب عنوان IP الطابعة (`192.168.1.200`).
   * ضع علامة صح `☑️` على *Auto detect the printer driver to use*.
   * اضغط **Next**.
6. **Printer Driver Selection:**
   * اختر **Install a new driver** واضغط **Next**.
   * حدد الشركة المصنعة والطراز، أو اضغط **Have Disk...** لتحديد مكان تعريف الطابعة (.inf).
7. **Printer Name and Sharing Settings:**
   * **Printer Name:** اكتب اسم الطابعة المرجعي (مثل: `HP-LaserJet-Floor1`).
   * **Share Name:** اكتب اسم المشاركة الشبكي (مثل: `Public-Printer`).
   * ضع علامة صح `☑️` على **Share this printer**.
8. اضغط **Next** لتأكيد الإعدادات، ثم اضغط **Finish**.

```
[Print Management Console]
   └── Print Servers ──► DC01 ──► Printers ──(Right Click)──► Add Printer...
         ├── Connection: TCP/IP Device (192.168.1.200)
         ├── Driver: Install / Select Driver
         └── Sharing: ☑️ Share this printer (Share Name: Public-Printer)
```

---

## 🧪 LAB Task 3: Driver Isolation & Advanced Printer Properties

### 🎯 الهدف:
تأمين السيرفر ومنع سقوط خدمة الـ Spooler عند تعطل التعريفات عبر **Driver Isolation**.

### 📝 الخطوات العملية:

#### أ) تفعيل عزل التعريفات (Driver Isolation):
1. داخل **Print Management Console** (`printmanagement.msc`).
2. وسّع اسم السيرفر `DC01` ⬅️ اضغط على مجلد **Drivers**.
3. ابحث عن التعريف الخاص بالطابعة في القائمة الوسطى.
4. اضغط كليك يمين على التعريف ⬅️ اختر **Set Driver Isolation**.
5. اختر إما:
   * **Shared:** لجميع الطابعات التي تستخدم نفس التعريف في بيئة واحدة.
   * **Isolated:** (الأكثر أماناً) لتشغيل التعريف في عملية خاصة مستقلة تماماً منفصلة عن السيرفر الرئيسي.

#### ب) ضبط أوقات العمل والأولويات (Priority & Availability):
1. اضغط كليك يمين على الطابعة `HP-LaserJet-Floor1` تحت مجلد **Printers** ⬅️ اختر **Properties**.
2. افتح تبويب **Advanced**:
   * **Always available:** للطباعة في أي وقت.
   * **Available from:** لتحديد ساعات عمل مخصصة (مثلاً من 8:00 AM إلى 5:00 PM).
   * **Priority:** حدد أولوية الطابعة من `1` (أقل) إلى `99` (أعلى أولوية للمدراء).
   * **Spool print documents:** تأكد من تحديد *Start printing immediately*.
3. اضغط **Apply** ثم **OK**.

---

## 🧪 LAB Task 4: Deploying Printers Automatically via GPO

### 🎯 الهدف:
نشر الطابعة تلقائياً على أجهزة ومستخدمين الدومين دون الذهاب لإضافتها يدوياً.

### 📝 الخطوات العملية:

1. داخل أداة **Print Management Console** (`printmanagement.msc`).
2. اضغط كليك يمين على الطابعة `HP-LaserJet-Floor1` ⬅️ اختر **Deploy with Group Policy...**.
3. في نافذة Deploy with Group Policy:
   * اضغط على زر **Browse...**.
   * اختر سياسة الـ GPO المطلوبة (مثل: `Default Domain Policy` أو سياسة قسم معين) واضغط **OK**.
4. حدد طريقة النشر تحت خيار **Deployment type**:
   * ضع علامة صح `☑️` على **The users to which this GPO applies (per user)**.
   * ضع علامة صح `☑️` على **The computers to which this GPO applies (per machine)**.
5. اضغط على زر **Add** لإدراج السياسة في الجدول أسفله.
6. اضغط **Apply** ثم **OK**.

```
[Print Management] ──► Right Click Printer ──► Deploy with Group Policy...
      ├── Select GPO ──► (e.g. Default Domain Policy)
      ├── Options: ☑️ Per User | ☑️ Per Machine
      └── [Add] ──► [Apply] ──► [OK]
```

#### 🔍 اختبار الاستلام على جهاز العميل (`WIN-CLIENT01`):
1. افتح موجه الأوامر **CMD** واكتب أمر تحديث السياسات:
   ```cmd
   gpupdate /force
   ```
2. افتح **Control Panel** ⬅️ **Devices and Printers** (أو Settings ➔ Bluetooth & devices ➔ Printers & scanners).
3. ستشاهد ظهور الطابعة `HP-LaserJet-Floor1 on DC01` تلقائياً ومستعدة للطباعة!

---

## 🧪 LAB Task 5: Managing Print Queues & Clearing Stuck Jobs via GUI

### 🎯 الهدف:
إدارة طوابير الطباعة المعلقة وإلغاء المستندات المتوقفة عن طريق الواجهة الرسومية دون إيقاف الخدمة يدوياً.

### 📝 الخطوات العملية:

1. داخل **Print Management Console**، اضغط كليك يمين على الطابعة ⬅️ اختر **Open Printer Queue...**.
2. للتحكم في ملف معين:
   * اضغط كليك يمين على الملف المعلق ⬅️ اختر **Pause** لإيقافه مؤقتاً، أو **Restart** لإعادة تحفيزه، أو **Cancel** لإلغائه.
3. لتفريغ الطابور بالكامل:
   * من القائمة العلوية للنافذة اختر **Printer** ⬅️ اضغط **Cancel All Documents**.

#### 💡 طريقة إعادة تشغيل الـ Print Spooler بالواجهة الرسومية:
إذا تعلقت طابعة بالكامل ولم تستجب لإلغاء الأمر:
1. اضغط `Win + R` واكتب **`services.msc`**.
2. ابحث عن خدمة **Print Spooler**.
3. اضغط عليها كليك يمين ⬅️ اختر **Restart**.

---

## 📊 GUI Navigation & Action Matrix

| Requirement / Goal | GUI Tool | Click Sequence |
| :--- | :--- | :--- |
| **تثبيت دور الطباعة** | `Server Manager` | Manage ➔ Add Roles ➔ **Print and Document Services**. |
| **إضافة طابعة شبكية** | `printmanagement.msc` | Print Servers ➔ ServerName ➔ Printers ➔ **Add Printer...**. |
| **تفعيل عزل التعريفات** | `printmanagement.msc` | Print Servers ➔ ServerName ➔ Drivers ➔ Right-Click Driver ➔ **Set Driver Isolation** ➔ Isolated. |
| **نشر الطابعة للعملاء** | `printmanagement.msc` | Right-Click Printer ➔ **Deploy with Group Policy...** ➔ Select GPO ➔ Add. |
| **مسح الأوامر المعلقة** | `printmanagement.msc` | Right-Click Printer ➔ **Open Printer Queue...** ➔ Printer Menu ➔ **Cancel All Documents**. |