***
## **`info 1`:**
ال virtual switches فى برنامج ال VMware workstation فى منها 3 أنواع:

- الأول هوا ال bridge وفيه لما بتربط ال VM ب VMnet0 اللى هوا ال bridge وعلشان ال VM تطلع انترنت لازم تكون واخدة IP فى النتورك الحقيقية أو dynamic IP من خلال ال DSL modem بتاع الشبكة الحقيقة. كمثال، عندى النتورك 192.168.100.0/24 والراوتر واخد 192.168.100.1. وبالتالى ال VM لازم تاخد IP فى النتورك دى، وال gateway يكون 192.168.100.1..

- التانى هوا ال NAT أو ال VMnet8.. وعندى على اللاب بتاعى ف رينج ال 192.168.113.0/24.. وبالتالى ال VM لازم تكون واخدة IP ف الرينج دا وتكون واخدة gateway قيمته 192.168.113.2. ودا لأن ال VMware workstation بيعمل راوتر وهمى بال IP دا.. لو اديت ال VM اديتها IP ف ال network ID الحقيقة بتاعة الشبكة بتاعتك مش هتطلع انترنت ولا توصل للشبكة الحقيقة لكن كل ال VMs اللى متوصلة على نفس ال VMnet هتوصل لبعض طالما واخدين ايبيهات ف نفس النتورك..

- التالت وهوا ال host only أو ال VMnet1 وفيه ال VM مش هتوصل للشبكة الحقيقية لكن ال VMS زى ما قلنا هتوصل لبعضها طالما ف نفس النتورك وطالما متوصلة بنفس ال VMnet..

مع الوضع ف الاعتبار ان من حقك تعمل create up to 19 virtual switch لكن واحد بس اللى يكون NAT واللى هوا VMnet 8 by default..

أى VMnet ممكن تشغل عليه DHCP service طالما مفعل ال use local DHCP service to distribute IP addresses to VMs وبالتالى ال VM ممكن تخليها تاخد dynamic IP بدل ال static .. وممكن تعديل ال range /
ن خلال DHCP settings..

كمان لو مفعلتش ال option الخاص ب connect a host virtual adapter to this host، ساعتها الكارت دا مش هيظهر تحت ال network connections.. ودا بحتاجه لما أكون موصلة virtual switches كتير ومش عايزة زحمة تحت ال network connections..

حبيت تغير قيمة ال IP لل virtual router على مستوى كارت ال NAT من خلال ال NAT settings..

كارت ال bridge ال default بيكون مربوط ب automatic بمعنى انه هيشوف انهى كارت physical عندك ع اللاب بتاعك ويربطه بيه، لكن ساعات دا بيعمل مشاكل معايا وبالتالى بحتاج أعمل hard code للكارت بإيدى سواء ال wifi أو ال ethernet أو حتى ال loopback..

لو عملت restore default كل حاجة بترجع ع ال default وبيظهر بس ال VMnet0 و ال VMnet 1 وال VMnet8، والباقى بيتمسح.. وكمان ال IP على مستوى كروت ال VMnet1 و VMnet8 بيتغيروا..  



# **`Programs Important:`** 
* https://networklookout.com
* https://www.solarwinds.com