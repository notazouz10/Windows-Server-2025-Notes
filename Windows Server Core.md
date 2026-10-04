***

دليل شامل ومكثف لشرح وإدارة **Windows Server Core** والتحكم به مباشرة أو عن بُعد.

---

## 📋 جدول المحتويات (Table of Contents)
- [📌 نبذة عن النظام (Overview)](#-نبذة-عن-النظام-overview)
- [💻 وسائل التحكم المباشر (Direct Management)](#-وسائل-التحكم-المباشر-direct-management)
- [⚙️ الإعدادات الأساسية عبر SConfig (Basic Setup)](#️-الإعدادات-الأساسية-عبر-sconfig-basic-setup)
- [🌐 وسائل التحكم عن بُعد (Remote Management)](#-وسائل-التحكم-عن-بُعد-remote-management)
- [🛠️ تثبيت الخدمات والأوامر الشهيرة (PowerShell Commands)](#️-تثبيت-الخدمات-والأوامر-الشهيرة-powershell-commands)

---

## 📌 نبذة عن النظام (Overview)

نظام **Windows Server Core** هو خيار تثبيت لنظام التشغيل Windows Server يعمل ببيئة الأوامر النصية **CLI (Command Line Interface)** فقط وبدون واجهة رسومية (**No GUI ❌**).

### 💡 الفوائد والمميزات:
* **توفير استهلاك الموارد:** استهلاك أقل بكثير في الـ RAM والـ CPU والتخزين.
* **زيادة الأمان:** تقليل مساحة الهجوم (**Reduced Attack Surface**) لعدم وجود متصفح أو مكونات GUI.
* **تحديثات وصيانة أقل:** يتطلب إعادة تشغيل وتحديثات دورية أقل مقارنة بإصدار GUI الكامل.

---

## 💻 وسائل التحكم المباشر (Direct Management)

يمكن إدارة خادم Server Core مباشرة من الشاشة المحلية عبر ثلاث أدوات رئيسية:

1. **`CMD` (Command Prompt / MS-DOS):** شاشة الأوامر النصية الافتراضية عند بدء التشغيل.
2. **`PowerShell` (`PS`):** بيئة الأوامر المتقدمة، ويمكن الانتقال إليها بكتابة `powershell` أو `Start-PowerShell`.
3. **`SConfig` (Server Configuration Tool):** أداة قائمة نصية مدمجة لإدارة إعدادات السيرفر الأساسية بسهولة.

---

## ⚙️ الإعدادات الأساسية عبر SConfig (Basic Setup)

عند كتابة الأمر `sconfig` في الشاشة النصية، تظهر قائمة خيارات لإعداد السيرفر:

### 1️⃣ الانضمام للدومين (Domain Join):
* اختر **Option 1** (Domain/Workgroup).
* أدخل اسم الدومين: `Test.local`.
* أدخل اسم مستخدم الأدمن وكلمة السر: `Test\administrator` / `P@ssw0rd`.

### 2️⃣ ضبط إعدادات الشبكة (Network Settings):
* اختر **Option 8** (Network Settings).
* تحويل العنوان من تلقائي (`DHCP` / `APIPA`) إلى ثابت (`Static IP`):
  * **IP Address:** `192.168.1.5`
  * **Subnet Mask:** `255.255.255.0`
  * **Default Gateway:** `192.168.1.1`
  * **Preferred DNS:** `192.168.1.2`

### 3️⃣ الخروج لبيئة الأوامر (Exit to CLI):
* اختر **Option 15** (أو `16` / `19` في إصدارات Server 2022 و 2025) للخروج إلى شاشة CMD/PowerShell.

---

## 🌐 وسائل التحكم عن بُعد (Remote Management)

نظراً لعدم وجود GUI على Server Core، يتم إدارته في البيئة العملية عن بُعد عبر خادم آخر يحتوي على GUI (مثل `PDC`) أو عبر أجهزة الأدمن:

```text
[ PDC / Admin Server (GUI) ]                        [ Windows Server Core (192.168.1.5) ]
┌──────────────────────────┐                        ┌──────────────────────────────────┐
│  Server Manager          │ ─────────────────────► │  CLI Only (No GUI)               │
│  MMC (.msc Consoles)     │     Add Server BY      │  Services:                       │
│  Windows Admin Center    │     IP / Hostname      │  - DHCP Role                     │
└──────────────────────────┘                        └──────────────────────────────────┘
```

### 🛠️ طرق الإدارة البعيدة:
1. **Server Manager (GUI):**
   * افتح Server Manager على سيرفر الـ GUI.
   * انتقل إلى **All Servers** ➡️ اضغط كليك يمين واختر **Add Servers**.
   * ابحث عن السيرفر بواسطة **IP** (`192.168.1.5`) أو الاسم (`Core.test.local`) وقم بإضافته.
2. **MMC (Microsoft Management Console):**
   * فتح أدوات الإدارة البعيدة وتوصيلها بـ Server Core مثل:
     * **DHCP Console**
     * **DNS Console**
     * **Hyper-V Manager**
3. **Windows Admin Center (WAC):**
   * منصة الإدارة الحديثة المستندة للويب لإدارة خوادم Core بالكامل بأسلوب غرافيكي متطور.

---

## 🛠️ تثبيت الخدمات والأوامر الشهيرة (PowerShell Commands)

يمكن تثبيت الأدوار والخدمات (مثل دور **DHCP** لتطبيق سيناريو **DHCP Failover**) على Server Core باستخدام أوامر PowerShell المباشرة:

### 1. عرض حالة الخدمات:
```powershell
Get-Service
```

### 2. تثبيت دور DHCP مع أدوات الإدارة وإعادة التشغيل:
```powershell
Install-WindowsFeature -Name DHCP -IncludeManagementTools -Restart
```

### 3. ربط الخادم بسيناريو DHCP Failover:
* بعد تثبيت دور DHCP على Server Core (`192.168.1.5`)، يتم الانتقال إلى سيرفر الـ GUI (`PDC`) وإضافة Server Core إلى كونسول **DHCP MMC** لإعداد الـ **DHCP Failover** بين السيرفرين.