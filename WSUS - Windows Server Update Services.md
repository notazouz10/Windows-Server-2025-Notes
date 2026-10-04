
***
## 📋 جدول المحتويات (Table of Contents)
- [📌 نبذة عن الخدمة (Overview)](#-نبذة-عن-الخدمة-overview)
- [💡 الأهمية والفوائد (Key Benefits)](#-الأهمية-والفوائد-key-benefits)
- [⚙️ آلية العمل (Architecture & Workflow)](#️-آلية-العمل-architecture--workflow)
- [🛠️ المكونات ومتطلبات البيئة (Requirements & Components)](#️-المكونات-ومتطلبات-البيئة-requirements--components)
- [🔧 إعداد سياسة المجموعة (GPO Configuration)](#-إعداد-سياسة-المجموعة-gpo-configuration)
- [🛡️ إعداد جدار الحماية (Firewall Inbound Rule)](#️-إعداد-جدار-الحماية-firewall-inbound-rule)
- [🚀 أفضل الممارسات (Best Practices)](#-أفضل-الممارسات-best-practices)
- [💻 أوامر سريعة للتحكم والصيانة (Useful Commands)](#-أوامر-سريعة-للتحكم-والصيانة-useful-commands)

---

## 📌 نبذة عن الخدمة (Overview)

**WSUS** (اختصار لـ *Windows Server Update Services*) هي دور محلي (**Server Role**) مدمج في نظام تشغيل **Windows Server**، يُتيح لمديري الأنظمة (*System Administrators*) إدارة وتوزيع التحديثات والتصحيحات الأمنية (**Patches & Updates**) لجميع أجهزة الكمبيوتر والسيرفرات داخل الشبكة المؤسسية من نقطة تحكم مركزية واحدة.

---

## 💡 الأهمية والفوائد (Key Benefits)

* **🚀 توفير استهلاك سعة الإنترنت (Bandwidth Optimization):**
  بدلاً من سحب التحديثات بشكل مستقل لكل جهاز من سيرفرات Microsoft، يقوم سيرفر WSUS بتنزيل التحديث **مرة واحدة فقط** وتوزيعه محلياً عبر الشبكة الداخلية (LAN).
* **🎯 التحكم الكامل والموافقة (Centralized Approval):**
  إمكانية اختبار التحديثات والموافقة عليها (`Approve`) أو رفضها (`Decline`) قبل تعميمها، لتجنب حدوث أي تعارض مع برمجيات الشركة.
* **📊 المراقبة والتقارير (Reporting & Auditing):**
  تقديم تقارير تفصيلية تبيّن حالة الأجهزة المحدثة، الثغرات المعالجة، والأجهزة التي تفشل في التحديث أو تحتاج إلى إعادة تشغيل.
* **📅 التوزيع المجدول والتدرج (Targeted Deployment):**
  تقسيم الأجهزة إلى مجموعات مستهدفة (*Computer Groups*) مثل: `Test Group` ⬅️ `Production Group`.

---

## ⚙️ آلية العمل (Architecture & Workflow)

```text
[ Microsoft Update Services ] 
             │ (Internet Download - Single Source)
             ▼
┌──────────────────────────┐
│       WSUS Server        │ ◄─── (Admin Approvals & GPO Policies)
└────────────┬─────────────┘
             │ (Internal LAN - HTTP:8530 / HTTPS:8531)
     ┌───────┼───────┐
     ▼       ▼       ▼
  [PC-01] [PC-02] [Server-01]
```

1. **المزامنة (Synchronization):** يتصل سيرفر WSUS بـ Microsoft Update لجلب عناوين وملفات التحديثات الجديدة.
2. **المراجعة والموافقة (Approval):** يراجع مسؤول النظام التحديثات ويوافق على التحديثات المناسبة للمجموعات المستهدفة.
3. **التوزيع (Distribution):** تتصل الأجهزة العميلة بالسيرفر المحلي عبر الشبكة الداخلية لتنزيل التحديثات المعتمدة وتثبيتها.

---

## 🛠️ المكونات ومتطلبات البيئة (Requirements & Components)

| المكون | الوصف / المتطلب |
| :--- | :--- |
| **WSUS Role** | الدور الأساسي المثبت على سيرفر Windows Server. |
| **IIS (Web Server)** | خادم الويب المسؤول عن نقل ملفات التحديث للأجهزة عبر HTTP (8530) / HTTPS (8531). |
| **Database** | **WID** (Windows Internal Database) للشبكات الصغيرة، أو **SQL Server** للبيئات الضخمة. |
| **Group Policy (GPO)** | توجيه الأجهزة عبر Active Directory Domain للاتصال بالسيرفر المحلي بدلاً من الإنترنت. |

---

## 🔧 إعداد سياسة المجموعة (GPO Configuration)

لتوجيه أجهزة الدومين للتحديث من سيرفر WSUS المحلي:

1. افتح **Group Policy Management Console** (`gpmc.msc`).
2. أنشئ GPO جديدة واربطها بـ OU الأجهزة المطلوب تحديثها.
3. انتقل إلى المسار التالي:
   `Computer Configuration` ➡️ `Policies` ➡️ `Administrative Templates` ➡️ `Windows Components` ➡️ `Windows Update`
4. قم بضبط إعدادات **Specify intranet Microsoft update service location**:
   * **Set the intranet update service for detecting updates:** `http://<WSUS-Server-IP-or-FQDN>:8530`
   * **Set the intranet statistics server:** `http://<WSUS-Server-IP-or-FQDN>:8530`
5. قم بتفعيل إعداد **Configure Automatic Updates** واختر آلية التحميل والتثبيت المناسبة.

---

## 🛡️ إعداد جدار الحماية (Firewall Inbound Rule)

للسماح للأجهزة العميلة بالاتصال بسيرفر WSUS عبر الشبكة، يجب فتح المنفذ الخاص بالخدمة في جدار الحماية (Windows Defender Firewall / GPO Firewall Rule):

1. افتح **Windows Defender Firewall with Advanced Security** (أو عبر GPO من مسار `Windows Settings` ➡️ `Security Settings` ➡️ `Windows Defender Firewall`).
2. اضغط على **Inbound Rules** ثم اختر **New Rule...** لتفتتح معالج (**New Inbound Rule Wizard**).
3. اختر نوع القاعدة **Port** ثم اضغط **Next**.
4. حدد البروتوكول والمنفذ (كما بالمخطط):
   * **Protocol:** اختر **TCP**.
   * **Local Ports:** اختر **Specific local ports** وأدخل المنفذ **`8530`** (أو `8531` في حال استخدام HTTPS).
5. حدد الإجراء **Allow the connection** ثم اضغط **Next**.
6. اختر البروفايل (Domain, Private, Public) ثم اسم القاعدة (مثال: `WSUS Inbound Port 8530`) واضغط **Finish**.

---

## 🚀 أفضل الممارسات (Best Practices)

> [!WARNING]
> **تنبيه:** لا تقم بنشر التحديثات مباشرة على بيئة الإنتاج (Production) دون اختبارها أولاً.

* **مجموعة الاختبار (Staging Group):** تطبيق التحديثات دائماً على مجموعة أجهزة تجريبية لمدة 7-10 أيام قبل نشرها لباقي الشركة.
* **التنظيف الدوري (Cleanup Wizard):** حظر وتخليص السيرفر من التحديثات القديمة والمستبدلة (`Superseded Updates`) لتوفير المساحة وتسرّع الأداء.
* **تخصيص المساحة (Storage Allocation):** توفير مساحة لا تقل عن **300GB - 500GB** لمجلد `WSUSContent`.

---

## 💻 أوامر سريعة للتحكم والصيانة (Useful Commands)

### 1. إجبار جهاز العميل (Client) على الاتصال بـ WSUS فوراً (CMD):
```cmd
gpupdate /force
wuauclt /detectnow /reportnow
```

### 2. إعادة تشغيل خدمة Windows Update على أجهزة العملاء (PowerShell):
```powershell
Restart-Service -Name wuauserv
```

### 3. تشغيل أداة التنظيف التلقائي على سيرفر WSUS (PowerShell):
```powershell
Get-WsusServer | Invoke-WsusServerCleanup -DeclineExpiredUpdates -DeclineSupersededUpdates -CleanupObsoleteComputers -CleanupUnneededFiles
```

---
Command Fix WSUS in Windows Client 
***

```
regedit
HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate

<powershell>
Test-NetConnection wsus.<name-server>.local -Port 8530

<command line>
net stop wuauserv
net start wuauserv

usoclient StartScan

usoclient StartInteractiveScan

```