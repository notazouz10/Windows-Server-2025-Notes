***

دليل شامل لشرح وإدارة **أدوات إدارة السيرفر عن بُعد (RSAT)** وتفويض الصلاحيات (**Delegation of Control**) لمهندسي الدعم الفني من أجهزتهم الشخصية.

---

## 📋 جدول المحتويات (Table of Contents)
- [📌 نبذة عن الخدمة والفكرة العامة (Overview & Concept)](#-نبذة-عن-الخدمة-والفكرة-العامة-overview--concept)
- [⚙️ مخطط البيئة وتفويض الصلاحيات (Architecture & Delegation)](#️-مخطط-البيئة-وتفويض-الصلاحيات-architecture--delegation)
- [🛠️ طرق تثبيت RSAT على نظام التشغيل (Installation Methods)](#-طرق-تثبيت-rsat-على-نظام-التشغيل-installation-methods)

---

## 📌 نبذة عن الخدمة والفكرة العامة (Overview & Concept)

أدوات **RSAT** هي حزمة من أدوات الإدارة المدمجة التي تسمح لمسؤولي النظام ومهندسي الدعم الفني (**IT Support**) بإدارة خوادم **Windows Server** وخادمي **Active Directory** من أجهزة العميل الخاص بهم (مثل Windows 10 أو Windows 11) دون الحاجة لتسجيل الدخول المباشر أو فتح جلسات RDP على سيرفر الـ **PDC**.

---

## ⚙️ مخطط البيئة وتفويض الصلاحيات (Architecture & Delegation)

```text
       [ Target Server / Domain Controller ]
                  Domain: Test.local
                   ┌──────────────┐
                   │   AD (PDC)   │
                   └──────┬───────┘
                          │
            ┌─────────────┼─────────────┐
            ▼             ▼             ▼
         [ HR ]        [ Fin ]       [ IT ]   (OUs)
            │
      Delegate Control (Right-click OU)
      ├── m.zohdy  ---> Reset Password
      └── a.ali    ---> Reset Password
            │
            ├──────────────────────────┐
            ▼                          ▼
     [ Client PC: IT01 ]        [ Client PC: IT02 ]
     OS: Windows 10             OS: Windows 11
     User: m.zohdy              User: a.ali
     Tool: RSAT (ADUC)          Tool: RSAT (ADUC)
```

### 🔐 آلية تفويض الصلاحيات (Delegation of Control):
1. **الهدف:** منح موظف الـ IT صلاحيات محددة فقط (مثل إعادة تعيين كلمة السر `Reset Password`) على وحدة تنظيمية معينة (مثل `HR OU`) دون إعطائه صلاحيات أدمن كاملة على السيرفر.
2. **الخطوات:**
   * من كونسول Active Directory على السيرفر (**PDC** داخل دومين `Test.local`):
   * اضغط كليك يمين على الـ **OU** المطلوب (مثل `HR`) واختر **Delegate Control...**.
   * قم بإضافة حساب المستخدم (مثل `m.zohdy` أو `a.ali`) وتحديد المهمة المطلوبة (مثال: `Reset Password`).
3. **النتيجة:** يستطيع الموظف فتح أداة **Active Directory Users and Computers** من جهازه الشخصي (`IT01` أو `IT02`) عبر **RSAT** وتطبيق التغييرات المسموح له بها فقط.

---

## 🛠️ طرق تثبيت RSAT على نظام التشغيل (Installation Methods)

تختلف طريقة تثبيت وتفعيل حزمة **RSAT** حسب إصدار نظام التشغيل على جهاز العميل:

### 1️⃣ على نظام Windows 10:
* تتطلب تنزيل حزمة التثبيت المستقلة الخاصة بـ **RSAT for Windows 10** من موقع Microsoft الرسمي وتثبيتها يدوياً.

### 2️⃣ على نظام Windows 11 (مدمجة - Built-in):
تكون الأدوات مدمجة داخل النظام كميزات اختيارية (**Optional Features**):

1. افتح الإعدادات (**Settings**).
2. انتقل إلى **System** ➡️ **Optional features**.
3. اضغط على **View features**.
4. ابحث عن **RSAT** واختر الأدوات المطلوبة (مثل: `RSAT: Active Directory Domain Services and Lightweight Directory Services Tools` أو `AD Users and Computers`) ثم اضغط **Install**.