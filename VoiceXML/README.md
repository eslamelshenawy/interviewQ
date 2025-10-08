# VoiceXML - Complete Learning Resource

## 📚 نظرة عامة

هذا المجلد يحتوي على شرح شامل لـ **VoiceXML** - اللغة المستخدمة في بناء تطبيقات IVR الصوتية.

---

## 📂 محتويات المجلد

### 1. **01-VoiceXML-Basics-Arabic.md**
شرح أساسيات VoiceXML بالتفصيل

**المواضيع:**
- ✅ ما هو VoiceXML؟
- ✅ البنية الأساسية للملف
- ✅ العناصر الأساسية (Elements)
- ✅ Dialogs (Forms & Menus)
- ✅ Fields & Input Types
- ✅ Prompts (الرسائل الصوتية)
- ✅ Grammars (قواعد الإدخال)
- ✅ Variables (المتغيرات)
- ✅ Control Flow (التحكم بالتدفق)
- ✅ أمثلة عملية كاملة

---

### 2. **02-VoiceXML-vs-SOAP-vs-HTTP-Arabic.md**
شرح الفرق بين VoiceXML و SOAP و HTTP

**المواضيع:**
- ✅ ما هو HTTP؟ (بروتوكول النقل)
- ✅ ما هو SOAP؟ (بروتوكول الرسائل)
- ✅ ما هو VoiceXML؟ (لغة برمجة IVR)
- ✅ الفرق بينهم بالتفصيل
- ✅ كيف يعملون معاً
- ✅ أمثلة عملية مجمّعة
- ✅ متى تستخدم كل واحد

---

## 🎯 لمن هذا المجلد؟

هذا المحتوى مثالي لـ:
- **IVR Developers**: مطوري أنظمة IVR
- **VoiceXML Programmers**: مبرمجي VoiceXML
- **Contact Center Engineers**: مهندسي مراكز الاتصال
- **Backend Developers**: المطورين الذين يتكاملون مع IVR
- **المبتدئين**: الذين يريدون تعلم VoiceXML من الصفر

---

## 📖 كيف تستخدم هذا المجلد؟

### مسار التعلم المقترح:

```
اليوم 1-2: أساسيات VoiceXML
├─ اقرأ: 01-VoiceXML-Basics-Arabic.md
├─ تعلم: العناصر الأساسية
├─ تعلم: Forms و Fields
└─ مارس: أمثلة بسيطة

اليوم 3: الفرق بين البروتوكولات
├─ اقرأ: 02-VoiceXML-vs-SOAP-vs-HTTP-Arabic.md
├─ افهم: HTTP Protocol
├─ افهم: SOAP Protocol
└─ افهم: كيف يتكامل VoiceXML معهم

اليوم 4-5: التطبيق العملي
├─ ابنِ: IVR بسيط
├─ اتصل بـ API: استخدام HTTP
├─ جرّب: SOAP Web Service
└─ اختبر: التطبيق كاملاً
```

---

## 💡 ما ستتعلمه

### من الملف الأول (Basics):

```xml
<!-- بعد الانتهاء، ستتمكن من كتابة: -->
<vxml version="2.1">
    <form id="banking">
        <field name="account" type="digits">
            <prompt>أدخل رقم حسابك</prompt>

            <filled>
                <data name="balance"
                      src="'http://api.bank.com/balance?account=' + account"/>

                <prompt>
                    رصيدك هو <value expr="balance.amount"/> ريال
                </prompt>
            </filled>
        </field>
    </form>
</vxml>
```

### من الملف الثاني (Protocols):

```
ستفهم الفرق بين:

HTTP:  الطريق (بروتوكول النقل)
       └─ GET /api/balance → {balance: 5000}

SOAP:  اللغة الرسمية (صيغة الرسائل)
       └─ <soap:Envelope>...</soap:Envelope>

VoiceXML: السكريبت (ماذا نقول للعميل)
       └─ <prompt>رصيدك 5000 ريال</prompt>
```

---

## 🔑 المفاهيم الأساسية

### 1. VoiceXML Elements

```xml
<vxml>       <!-- العنصر الجذر -->
<form>       <!-- النموذج (حوار) -->
<field>      <!-- حقل إدخال -->
<prompt>     <!-- رسالة صوتية -->
<block>      <!-- كتلة تنفيذ -->
<menu>       <!-- قائمة خيارات -->
<goto>       <!-- انتقال -->
<if>         <!-- شرط -->
<data>       <!-- استدعاء API -->
```

---

### 2. Input Types

```xml
<field name="account" type="digits">   <!-- أرقام -->
<field name="confirm" type="boolean">  <!-- نعم/لا -->
<field name="date" type="date">        <!-- تاريخ -->
<field name="choice">                  <!-- مخصص -->
```

---

### 3. Prompts

```xml
<!-- TTS (Text-to-Speech) -->
<prompt>مرحباً بك</prompt>

<!-- Audio File -->
<prompt>
    <audio src="welcome.wav">مرحباً</audio>
</prompt>

<!-- Mixed -->
<prompt>
    <audio src="hello.wav">مرحباً</audio>
    اسمك <value expr="name"/>
</prompt>
```

---

### 4. Grammars

```xml
<!-- DTMF (أزرار الهاتف) -->
<grammar type="application/x-gsl">
    [(1) {<choice "balance">}
     (2) {<choice "transfer">}]
</grammar>

<!-- Speech (التعرف على الكلام) -->
<grammar type="application/x-gsl">
    [(رصيد balance) {<choice "balance">}]
</grammar>
```

---

## 🛠️ الأدوات المطلوبة

### للتطوير:
- **CVP Studio**: أداة Cisco لتطوير IVR
- **Text Editor**: VS Code, Notepad++
- **VXML Validator**: للتحقق من صحة الكود

### للاختبار:
- **CVP Platform**: لتشغيل التطبيقات
- **SIP Phone/Softphone**: للاتصال واختبار IVR
- **Postman**: لاختبار APIs

---

## 📊 أمثلة عملية

### مثال 1: IVR بسيط

```xml
<?xml version="1.0"?>
<vxml version="2.1">
    <form id="hello">
        <block>
            <prompt>مرحباً بك في نظام IVR!</prompt>
            <exit/>
        </block>
    </form>
</vxml>
```

**ماذا يفعل:**
- يشغل رسالة ترحيبية
- ينهي المكالمة

---

### مثال 2: قائمة بسيطة

```xml
<?xml version="1.0"?>
<vxml version="2.1">
    <menu>
        <prompt>
            اضغط 1 للمبيعات
            اضغط 2 للدعم الفني
        </prompt>

        <choice dtmf="1" next="sales.vxml"/>
        <choice dtmf="2" next="support.vxml"/>
    </menu>
</vxml>
```

---

### مثال 3: جمع بيانات

```xml
<?xml version="1.0"?>
<vxml version="2.1">
    <form>
        <field name="name">
            <prompt>ما اسمك؟</prompt>
            <filled>
                <prompt>
                    مرحباً <value expr="name"/>
                </prompt>
            </filled>
        </field>
    </form>
</vxml>
```

---

### مثال 4: استدعاء API

```xml
<?xml version="1.0"?>
<vxml version="2.1">
    <form>
        <block>
            <!-- Call REST API -->
            <data name="weather"
                  src="http://api.weather.com/riyadh"
                  method="get"/>

            <prompt>
                الطقس اليوم <value expr="weather.temp"/> درجة
            </prompt>
        </block>
    </form>
</vxml>
```

---

## 🔗 التكامل مع Backend

### VoiceXML → REST API (HTTP + JSON)

```xml
<data name="result"
      src="http://api.bank.com/balance"
      method="get">
    <property name="Content-Type" value="'application/json'"/>
</data>

<!-- Response: {"balance": 5000} -->
<prompt>
    رصيدك <value expr="result.balance"/> ريال
</prompt>
```

---

### VoiceXML → SOAP Web Service

```xml
<data name="soap"
      src="http://api.bank.com/soap"
      method="post"
      enctype="text/xml">

    <property name="body" value="'
        &lt;soap:Envelope&gt;
            &lt;soap:Body&gt;
                &lt;getBalance/&gt;
            &lt;/soap:Body&gt;
        &lt;/soap:Envelope&gt;
    '"/>
</data>
```

---

## 🎓 مستويات التعلم

### Level 1: مبتدئ (Beginner)
```
✅ فهم أساسيات VoiceXML
✅ كتابة IVR بسيط
✅ استخدام Forms و Fields
✅ تشغيل رسائل صوتية
```

### Level 2: متوسط (Intermediate)
```
✅ استخدام Grammars
✅ Error handling
✅ استدعاء APIs
✅ التعامل مع Variables
```

### Level 3: متقدم (Advanced)
```
✅ تكامل معقد مع Backend
✅ Authentication flows
✅ Multi-language support
✅ Performance optimization
```

---

## 📋 Checklist - هل أنت جاهز؟

بعد دراسة هذا المجلد، يجب أن تكون قادراً على:

**Basics:**
- [ ] شرح ما هو VoiceXML
- [ ] كتابة ملف VoiceXML صحيح
- [ ] استخدام العناصر الأساسية
- [ ] بناء Forms و Menus

**Prompts & Audio:**
- [ ] تشغيل رسائل صوتية (TTS)
- [ ] استخدام ملفات صوتية (Audio files)
- [ ] دمج TTS و Audio
- [ ] استخدام SSML

**Input & Grammars:**
- [ ] جمع مدخلات DTMF
- [ ] استخدام Speech recognition
- [ ] كتابة Grammars
- [ ] التحقق من الإدخال

**Integration:**
- [ ] استدعاء REST APIs
- [ ] استدعاء SOAP Web Services
- [ ] معالجة Responses
- [ ] Error handling

**Protocols:**
- [ ] فهم HTTP
- [ ] فهم SOAP
- [ ] الفرق بين HTTP و SOAP
- [ ] كيف يستخدم VoiceXML كليهما

---

## 🚀 مشاريع للتطبيق

### Project 1: Simple IVR
```
Build: Welcome message IVR
Features:
- Play welcome
- Language selection
- Main menu
- Transfer to agent

Time: 2 hours
```

---

### Project 2: Information Lookup
```
Build: Weather inquiry IVR
Features:
- Get city name
- Call weather API
- Play weather info
- Repeat or exit

Time: 4 hours
```

---

### Project 3: Banking IVR
```
Build: Bank account IVR
Features:
- Authentication (PIN)
- Check balance (API call)
- Recent transactions
- Transfer money
- Agent transfer

Time: 8 hours
```

---

## 📚 موارد إضافية

### Documentation:
- [VoiceXML 2.1 Specification (W3C)](https://www.w3.org/TR/voicexml21/)
- [Cisco CVP Documentation](https://www.cisco.com/c/en/us/support/customer-collaboration/unified-customer-voice-portal/tsd-products-support-series-home.html)

### Tutorials:
- W3C VoiceXML Tutorial
- Cisco CVP Studio Guides

### Tools:
- CVP Studio (Cisco)
- Voxeo Prophecy
- BeVocal Cafe

---

## 💼 فرص العمل

**المهارات المكتسبة تساعدك في:**

✅ **IVR Developer** (₹6-12 LPA / $50k-$80k)
✅ **VoiceXML Programmer** (₹8-15 LPA / $60k-$90k)
✅ **Contact Center Developer** (₹10-18 LPA / $70k-$100k)
✅ **Cisco CVP Specialist** (₹15-25 LPA / $90k-$120k)

---

## 🎯 الخطوات التالية

بعد إتمام هذا المجلد:

1. **تطبيق عملي**: ابنِ 3 مشاريع IVR
2. **التكامل**: تعلم Spring Boot للـ Backend
3. **CVP Studio**: احترف أداة التطوير
4. **ICM Integration**: تعلم توجيه المكالمات
5. **Production**: نشر IVR على بيئة حقيقية

---

## 📊 إحصائيات المحتوى

| المحتوى | العدد |
|---------|-------|
| **ملفات** | 3 (README + 2 guides) |
| **صفحات** | 150+ |
| **أمثلة برمجية** | 50+ |
| **رسومات توضيحية** | 20+ |
| **وقت الدراسة** | 10-15 ساعة |

---

## ✅ Quick Start

**ابدأ الآن في 3 خطوات:**

```
1. اقرأ: 01-VoiceXML-Basics-Arabic.md
   └─ افهم الأساسيات

2. اقرأ: 02-VoiceXML-vs-SOAP-vs-HTTP-Arabic.md
   └─ افهم التكامل

3. مارس: ابنِ IVR بسيط
   └─ طبّق ما تعلمته
```

---

**Happy Learning! 🎓**

تذكر: أفضل طريقة للتعلم هي التطبيق العملي!

---

**Quick Links:**
- [البدء بالأساسيات →](01-VoiceXML-Basics-Arabic.md)
- [فهم البروتوكولات →](02-VoiceXML-vs-SOAP-vs-HTTP-Arabic.md)

---

Last Updated: January 2025
Version: 1.0
