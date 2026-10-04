

دليل شامل لإعداد وإدارة لوحة التحكم الموحدة **Windows Admin Center (WAC)** على Windows Server، وتفعيل إدارة السيرفرات المركزية عبر المتصفح باستخدام بروتوكولات **WinRM** و **HTTPS**.

---

## 📋 جدول المحتويات (Table of Contents)
- [📌 نبذة عن الخدمة (Overview)](#-نبذة-عن-الخدمة-overview)
- [🏗️ بنية الشبكة والأجهزة (Network Architecture)](#️-بنية-الشبكة-والأجهزة-network-architecture)
- [⚙️ التثبيت وطريقة الوصول (Installation & Access)](#️-التثبيت-وطريقة-الوصول-installation--access)
- [🔌 المنافذ والبروتوكولات المطلوبة (Ports & Protocols)](#-المنافذ-والبروتوكولات-المطلوبة-ports--protocols)
- [🛡️ إعداد WinRM وسياسات المجموعة (Group Policy Setup)](#️-إعداد-winrm-وسياسات-المجموعة-group-policy-setup)

---

## 📌 نبذة عن الخدمة (Overview)

**Windows Admin Center (WAC)** هي أداة إدارة رسمية قائمة على المتصفح (Browser-based) خفيفة ومحليّة (Locally Deployed) تُستخدم لإدارة خوادم Windows Server (سواء كانت GUI أو Core) والأجهزة دون الحاجة للاتصال بالسحابة أو الاعتماد التام على أدوات MMC القديمة.

---

## 🏗️ بنية الشبكة والأجهزة (Network Architecture)

```text
                           [ Active Directory Domain Controller ]
                                 PDC (Test.local)
                                 IP: 192.168.1.2
                                        │
                                        │ ([https://WAC.test.local](https://WAC.test.local))
                                        ▼
                           ┌─────────────────────────┐
                           │   WAC Gateway Server    │
                           │   IP: 192.168.1.10      │
                           └────────────┬────────────┘
                                        │
               ┌────────────────────────┴────────────────────────┐
               ▼                                                 ▼
     [ Server Core Node ]                              [ Web Server Node ]
      Core (Server Core)                                webserv (IIS Role)
       IP: 192.168.1.5                                   IP: 192.168.1.8
```

### 🖥️ مكونات البيئة (Servers Setup):
* **PDC (Domain Controller):** خادم الدومين الرئيسي لبيئة `Test.local` كصاحب العنوان `192.168.1.2`.
* **WAC Gateway Server:** السيرفر المخصص لاستضافة WAC بملف `WAC.msi` مع دور `IIS` على IP: `192.168.1.10`.
* **Core Server:** خادم بدون واجهة رسمية (Windows Server Core) على IP: `192.168.1.5`.
* **Webserv:** خادم ويب يعمل عليه دور `IIS` على IP: `192.168.1.8`.

---

## ⚙️ التثبيت وطريقة الوصول (Installation & Access)

### 1️⃣ خطوات التثبيت:
1. إضافة دور خادم الويب (`Add Role: IIS`) على خادم الإدارة (`192.168.1.10`).
2. تشغيل ملف التثبيت الخاص بـ WAC:
   ```cmd
   WAC.msi
   ```
3. ضبط شهادة الأمان (SSL Certificate) وتحديد منفذ الاتصال بالبوابة (`443`).

### 2️⃣ الوصول وإضافة السيرفرات:
1. فتح المتصفح والانتقال إلى رابط البوابة:
   ```text
   [https://WAC.test.local](https://WAC.test.local)  (أو  [https://192.168.1.10](https://192.168.1.10))
   ```
2. تسجبل الدخول عبر خيار **Connect as** باستخدام صلاحيات حساب المسؤول (Domain Admin Username & Password).
3. استخدام خيار **Add Servers** لإضافة السيرفرات المستهدفة (`PDC`, `Core`, `webserv`) للوحة التحكم الموحدة.

---

## 🔌 المنافذ والبروتوكولات المطلوبة (Ports & Protocols)

تتطلب WAC فتح المنافذ التالية على الجدار الناري (Firewall) لضمان التواصل بين المتصفح وسيرفر WAC والسيرفرات المدارة:

| Service / Protocol | Port Number | Description |
| :--- | :---: | :--- |
| **HTTPS** | **`443`** | الوصول للوحة تحكم WAC عبر متصفح الويب بشكل مشفر |
| **WinRM (HTTP)** | **`5985`** | إدارة السيرفرات عن بُعد عبر Windows Remote Management (غير مشفر) |
| **WinRM (HTTPS)** | **`5986`** | إدارة السيرفرات عن بُعد عبر WinRM المشفر بـ SSL/TLS |

---

## 🛡️ إعداد WinRM وسياسات المجموعة (Group Policy Setup)

تعتمد بوابة WAC كلياً على خدمة **WinRM (Windows Remote Management)** للاتصال بالخوادم المدارة وإدارتها.

### 📝 الخطوات عبر Group Policy (GPO):
1. **تمكين WinRM:** تفعيل خدمة WinRM على كافة السيرفرات في الدومين تلقائياً.
2. **استثناءات الجدار الناري (Firewall Rules):** إنشـاء قواعد سماح (**Allow Rules**) للمنافذ:
   * **`5985`** (WinRM HTTP)
   * **`5986`** (WinRM HTTPS)
3. تطبيق الـ Policy على أجهزة Domain Controllers والسيرفرات المدارة (`PDC`, `Core`, `webserv`) لتسهيل عملية الإدارة المركزية بدون مشاكل اتصال.