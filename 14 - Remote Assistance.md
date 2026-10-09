***

دليل شامل ومكثف لإعداد وإدارة خدمة **Remote Assistance (المساعدة عن بُعد)** داخل بيئة الدومين وربطها عبر Group Policy.

---

## 📋 جدول المحتويات (Table of Contents)
- [📌 نبذة عن الخدمة (Overview)](#-نبذة-عن-الخدمة-overview)
- [🛠️ تثبيت الميزة (Add Feature)](#️-تثبيت-الميزة-add-feature)
- [🔧 إعداد سياسة المجموعة (Group Policy Configuration)](#-إعداد-سياسة-المجموعة-group-policy-configuration)
- [🛡️ إعداد جدار الحماية (Firewall Rule)](#️-إعداد-جدار-الحماية-firewall-rule)
- [⚙️ آلية العمل والأوامر (Workflow & Commands)](#️-آلية-العمل-والأوامر-workflow--commands)

---

## 📌 نبذة عن الخدمة (Overview)

خدمة **Remote Assistance (MSRA)** هي أداة مدمجة في نظام تشغيل Windows تتيح لفريق الدعم الفني (**IT Support**) أو مديري النظام الاتصال بأجهزة المستخدمين ورؤية الشاشة والتحكم بها لحل المشاكل التقنية بموافقة المستخدم.

---

## 🛠️ تثبيت الميزة (Add Feature)

لتفعيل الخدمة على خوادم Windows Server:
* افتح **Server Manager** واختر **Add Roles and Features**.
* في قسم **Features**، قم بتحديد ميزة **Remote Assistance** واكتمل التثبيت.

---

## 🔧 إعداد سياسة المجموعة (Group Policy Configuration)

لتطبيق الإعدادات مركزياً على جميع أجهزة الدومين:

1. افتح **Group Policy Management Console** (`gpmc.msc`).
2. انتقل إلى المسار التالي:
   `Computer Configuration` ➡️ `Policies` ➡️ `Administrative Templates` ➡️ `System` ➡️ `Remote Assistance`
3. قم بتفعيل سياسة **Configure Offer Remote Assistance**:
   * اختر **Enabled**.
   * حدد الصلاحيات المطلوبة (مثل: *Allow helpers to remotely control the computer*).
4. أضف المساعدين المصرح لهم (**Helpers**):
   * اضغط على **Show...** في قائمة المساعدين.
   * أضف مجموعات أو مستخدمي الدعم الفني، مثل:
     * `Test\ITsupport` (مجموعة Group أو مستخدم User)
     * `Test\Domain Admins`



---

## 🛡️ إعداد جدار الحماية (Firewall Rule)

لضمان مرور الاتصال دون حظر:
* قم بتفعيل خيار **Allow Remote Assistance in Windows Firewall** سواء عبر GPO أو محلياً للسماح للمنافذ الخاصة بأداة MSRA بالمرور عبر جدار الحماية.

---

## ⚙️ آلية العمل والأوامر (Workflow & Commands)

```text
┌──────────────┐         1. Send Invitation File + Password        ┌──────────────┐
│  Client PC   │ ─────────────────────────────────────────────►│ Server / Admin│
│  (msra.exe)  │ ◄─────────────────────────────────────────────│ (IT Support) │
└──────────────┘           2. Connect & Assist User            └──────────────┘
```

### 🛠️ الطرق والأوامر المستخدمة:

1. **طلب المساعدة من العميل (Client Invitation):**
   * يقوم العميل بفتح الأداة عبر أمر **`msra`** في نافذة Run.
   * يتم إنشاء ملف دعوة (**Remote Assistance File**) مع كلمة سر (**Password**)، وإرساله لدعم IT.

2. **تقديم المساعدة المباشرة من الأدمن (Offer Remote Assistance):**
   * يمكن لمهندس الدعم تقديم المساعدة مباشرة عبر تنفيذ الأمر التالي في Run أو PowerShell/CMD:
     ```cmd
     msra /offerra
     ```
   * يتم إدخال اسم الجهاز أو IP الخاص بالعميل للاتصال به مباشرة وطلب الإذن للتحكم.

***
