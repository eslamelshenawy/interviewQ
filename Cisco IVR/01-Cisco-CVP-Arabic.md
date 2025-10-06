# Cisco Customer Voice Portal (CVP) - الدليل الشامل

## المحتويات
1. [مقدمة عن CVP](#مقدمة-عن-cvp)
2. [البنية المعمارية](#البنية-المعمارية)
3. [المكونات الأساسية](#المكونات-الأساسية)
4. [أنواع CVP Deployments](#أنواع-cvp-deployments)
5. [CVP Studio](#cvp-studio)
6. [Call Flow في CVP](#call-flow-في-cvp)
7. [التكامل مع ICM](#التكامل-مع-icm)
8. [VoiceXML](#voicexml)
9. [أفضل الممارسات](#أفضل-الممارسات)
10. [استكشاف الأخطاء](#استكشاف-الأخطاء)

---

## مقدمة عن CVP

### ما هو Cisco Customer Voice Portal (CVP)؟

**Cisco CVP** هو منصة IVR (Interactive Voice Response) متقدمة تمكن المؤسسات من:
- بناء تطبيقات صوتية تفاعلية Self-Service
- توفير خدمة عملاء آلية متطورة
- التكامل مع أنظمة الـ Backend (CRM, Databases, APIs)
- توجيه المكالمات بذكاء إلى الوكلاء المناسبين

### الفوائد الرئيسية

✅ **تقليل التكاليف**: تقليل الحاجة للوكلاء البشريين
✅ **خدمة 24/7**: خدمة عملاء متاحة دائماً
✅ **قابلية التوسع**: يدعم آلاف المكالمات المتزامنة
✅ **التخصيص**: بناء تجارب صوتية مخصصة
✅ **التكامل**: سهولة التكامل مع أنظمة Enterprise

---

## البنية المعمارية

### المكونات الأساسية في نظام CVP

```
┌─────────────────────────────────────────────────────┐
│                    PSTN/SIP                         │
└──────────────────┬──────────────────────────────────┘
                   │
          ┌────────▼────────┐
          │  Cisco CUBE/    │
          │  SIP Gateway    │
          └────────┬────────┘
                   │
          ┌────────▼────────┐
          │  CVP Server     │◄──────┐
          │  (Call Server)  │       │
          └────────┬────────┘       │
                   │                │
          ┌────────▼────────┐       │
          │  VXML Server    │       │
          │  (CVP Studio)   │       │
          └────────┬────────┘       │
                   │                │
          ┌────────▼────────┐       │
          │  ICM/UCCE       │───────┘
          │  (Routing)      │
          └────────┬────────┘
                   │
          ┌────────▼────────┐
          │  Backend APIs   │
          │  (Spring Boot)  │
          └─────────────────┘
```

---

## المكونات الأساسية

### 1. CVP Call Server

**الدور:**
- إدارة جلسات المكالمات (Call Sessions)
- التواصل مع ICM للحصول على قرارات التوجيه
- معالجة SIP Signaling
- إدارة Media Resources

**الوظائف الرئيسية:**
- Call admission control
- SIP call handling
- Session management
- Reporting and logging

### 2. CVP VXML Server

**الدور:**
- تقديم صفحات VoiceXML للمتصل
- معالجة مدخلات المستخدم (DTMF/Speech)
- تنفيذ منطق التطبيق (Application Logic)
- التكامل مع Backend APIs

**القدرات:**
- VoiceXML interpretation
- TTS (Text-to-Speech)
- ASR (Automatic Speech Recognition)
- Audio playback
- Data collection

### 3. CVP Reporting Server

**الدور:**
- جمع بيانات المكالمات
- إنشاء تقارير تفصيلية
- تحليل الأداء
- Historical data storage

### 4. CVP Operations Console

**الدور:**
- واجهة إدارة CVP
- مراقبة النظام (System Monitoring)
- إدارة التطبيقات (Application Management)
- Troubleshooting

---

## أنواع CVP Deployments

### 1. Comprehensive Deployment

```
المتصل → Gateway → CVP → ICM → Agent
                    ↓
              VXML Server
                    ↓
            Backend Systems
```

**الميزات:**
- دعم Self-Service كامل
- Queuing and routing للوكلاء
- تكامل كامل مع ICM/UCCE
- الأكثر مرونة وقوة

**حالات الاستخدام:**
- Contact Centers الكبيرة
- احتياجات IVR معقدة
- تطبيقات متعددة

### 2. Call Studio Deployment

```
المتصل → Gateway → CVP VXML Server → Backend APIs
```

**الميزات:**
- Self-service فقط (بدون توجيه للوكلاء)
- أبسط في التنفيذ
- تكلفة أقل
- لا يتطلب ICM

**حالات الاستخدام:**
- تطبيقات Self-service بسيطة
- Account inquiries
- Bill payment
- Information services

### 3. Standalone Deployment

**الميزات:**
- CVP بدون ICM
- محدود في قدرات التوجيه
- مناسب للتطبيقات الصغيرة

---

## CVP Studio

### ما هو CVP Studio؟

**CVP Studio** هو أداة تطوير رسومية (GUI-based) لبناء تطبيقات IVR دون الحاجة لكتابة VoiceXML يدوياً.

### المكونات الرئيسية

#### 1. Elements (العناصر)

**Call Control Elements:**
- `Start` - نقطة البداية
- `End` - نقطة النهاية
- `Decision` - اتخاذ قرارات
- `Subdialog` - استدعاء dialogs أخرى

**Voice Elements:**
- `Menu` - قوائم صوتية
- `Play` - تشغيل رسائل صوتية
- `Get Digits` - جمع أرقام DTMF
- `Get Speech` - التعرف على الكلام

**Data Elements:**
- `Set` - تعيين متغيرات
- `Database` - استعلامات قواعد البيانات
- `HTTP Request` - استدعاء REST APIs
- `Decision` - منطق شرطي

**Integration Elements:**
- `Send To ICM` - إرسال للـ ICM
- `Web Service` - SOAP/REST calls
- `Java Class` - تنفيذ Java code

#### 2. Variables (المتغيرات)

**أنواع المتغيرات:**

```java
// Session Variables - متاحة طوال الجلسة
session.callerID
session.language
session.accountNumber

// Application Variables - خاصة بالتطبيق
app.maxRetries = 3
app.timeout = 5

// Element Variables - محلية للعنصر
digit_input
menu_choice
```

#### 3. Audio Management

**أنواع الصوتيات:**
- **Pre-recorded Audio**: ملفات WAV محفوظة مسبقاً
- **TTS (Text-to-Speech)**: تحويل نص لصوت
- **Dynamic Audio**: صوتيات مبنية ديناميكياً

**مثال:**
```xml
<!-- Pre-recorded -->
<audio src="welcome.wav"/>

<!-- TTS -->
<audio>Welcome to customer service</audio>

<!-- Dynamic -->
<audio>Your balance is <value expr="session.balance"/> dollars</audio>
```

---

## Call Flow في CVP

### مثال: Call Flow نموذجي

```
1. Customer calls → PSTN/SIP
          ↓
2. Gateway routes to CVP Call Server
          ↓
3. CVP queries ICM for routing decision
          ↓
4. ICM returns VoiceXML URL
          ↓
5. CVP fetches VXML from VXML Server
          ↓
6. Customer interacts with IVR
   - Language selection
   - Authentication
   - Service selection
          ↓
7. Two options:
   a) Self-service completion → End call
   b) Transfer to agent → CVP+ICM routing
```

### مثال تفصيلي: Banking IVR

```
Start
  ↓
Welcome Message
"Welcome to ABC Bank"
  ↓
Language Selection
"Press 1 for English, 2 for Arabic"
  ↓
Main Menu
"Press 1 for Balance, 2 for Transfers, 3 for Agent"
  ↓
┌─────────┬─────────┬─────────┐
│ Balance │Transfer │  Agent  │
└────┬────┴────┬────┴────┬────┘
     ↓         ↓         ↓
  Get PIN   Get PIN   Route to
     ↓         ↓       Agent
  Validate  Validate
     ↓         ↓
  Call API  Call API
     ↓         ↓
  Play      Execute
  Balance   Transfer
     ↓         ↓
   End       End
```

---

## التكامل مع ICM

### لماذا التكامل مع ICM؟

**ICM (Intelligent Contact Management)** يوفر:
- **Intelligent Routing**: توجيه ذكي للمكالمات
- **Skills-based Routing**: توجيه حسب مهارات الوكلاء
- **Queue Management**: إدارة طوابير الانتظار
- **Reporting**: تقارير شاملة

### كيفية التكامل

#### 1. Call Flow مع ICM

```
CVP Call Server
    ↓ (Send routing request)
ICM Peripheral Gateway (PG)
    ↓ (Execute routing script)
ICM Router
    ↓ (Return routing decision)
CVP
    ↓ (Execute VXML or transfer to agent)
```

#### 2. ICM Routing Scripts

**مثال: Script لتوجيه المكالمات**

```
IF (Time of Day = Business Hours) THEN
    IF (Caller Language = Arabic) THEN
        Queue to Arabic_Support_Team
    ELSE
        Queue to English_Support_Team
    END IF
ELSE
    Play "We are closed" message
    Send to Voicemail
END IF
```

#### 3. CVP-ICM Variables

**تمرير البيانات بين CVP و ICM:**

```java
// من CVP إلى ICM
callVariables.accountNumber = "123456"
callVariables.serviceType = "Technical Support"
callVariables.priority = "High"

// من ICM إلى CVP
agent.skillGroup = "Billing"
agent.extension = "1234"
queue.position = "5"
```

---

## VoiceXML

### ما هو VoiceXML؟

**VoiceXML** هو لغة XML لبناء تطبيقات صوتية تفاعلية.

### البنية الأساسية

```xml
<?xml version="1.0"?>
<vxml version="2.1">
    <form id="mainMenu">
        <!-- Play welcome message -->
        <block>
            <prompt>
                Welcome to customer service.
            </prompt>
        </block>

        <!-- Get user input -->
        <field name="menuChoice">
            <prompt>
                Press 1 for account balance.
                Press 2 for bill payment.
                Press 0 for operator.
            </prompt>

            <grammar type="application/x-gsl">
                [
                    (one 1)    {<menuChoice "1">}
                    (two 2)    {<menuChoice "2">}
                    (zero 0)   {<menuChoice "0">}
                ]
            </grammar>

            <filled>
                <if cond="menuChoice == '1'">
                    <goto next="#checkBalance"/>
                <elseif cond="menuChoice == '2'"/>
                    <goto next="#payBill"/>
                <elseif cond="menuChoice == '0'"/>
                    <goto next="#transferAgent"/>
                </if>
            </filled>
        </field>
    </form>

    <form id="checkBalance">
        <!-- Balance inquiry logic -->
        <block>
            <prompt>Your current balance is 1000 dollars.</prompt>
            <goto next="#mainMenu"/>
        </block>
    </form>
</vxml>
```

### العناصر الأساسية

#### 1. `<form>` - النموذج

```xml
<form id="getAccountNumber">
    <field name="accountNum" type="digits">
        <prompt>Please enter your 10 digit account number</prompt>
    </field>
</form>
```

#### 2. `<field>` - حقل الإدخال

```xml
<field name="language">
    <prompt>Press 1 for English, 2 for Arabic</prompt>
    <option dtmf="1" value="en">English</option>
    <option dtmf="2" value="ar">Arabic</option>
</field>
```

#### 3. `<menu>` - القائمة

```xml
<menu id="mainMenu">
    <prompt>
        Main menu.
        Say or press 1 for sales.
        Say or press 2 for support.
    </prompt>
    <choice dtmf="1" next="#sales">Sales</choice>
    <choice dtmf="2" next="#support">Support</choice>
</menu>
```

#### 4. `<block>` - كتلة تنفيذ

```xml
<block>
    <prompt>Please wait while we retrieve your information</prompt>
    <data name="customerData" src="http://api.example.com/customer"/>
</block>
```

#### 5. `<subdialog>` - حوار فرعي

```xml
<subdialog name="auth" src="authentication.vxml">
    <param name="custId" expr="session.customerId"/>
    <filled>
        <if cond="auth.status == 'success'">
            <goto next="#authenticatedMenu"/>
        <else/>
            <goto next="#authFailed"/>
        </if>
    </filled>
</subdialog>
```

---

## أفضل الممارسات

### 1. تصميم User Experience

✅ **اجعل القوائم قصيرة**
- لا تزيد عن 5 خيارات
- اذكر الخيارات الأكثر استخداماً أولاً

✅ **استخدم لغة واضحة**
```
❌ سيء: "للخيارات المتعلقة بحسابك اضغط 1"
✅ جيد: "للاستعلام عن الرصيد اضغط 1"
```

✅ **وفر طريقة للخروج دائماً**
```xml
<field name="choice">
    <prompt>
        Press 1 for balance, 2 for transfer.
        Press star to return to main menu.
        Press 0 for operator.
    </prompt>
</field>
```

### 2. Error Handling

✅ **استخدم Retries محدودة**
```xml
<field name="accountNum">
    <prompt>Enter your account number</prompt>

    <noinput count="1">
        I didn't hear anything. Please enter your account number.
    </noinput>

    <noinput count="2">
        I still didn't hear you. Let me transfer you to an agent.
        <goto next="#transferAgent"/>
    </noinput>

    <nomatch count="1">
        I didn't understand. Please enter digits only.
    </nomatch>
</field>
```

### 3. Performance Optimization

✅ **Cache Audio Files**
- استخدم CDN للملفات الصوتية
- Pre-load الملفات المستخدمة بكثرة

✅ **Optimize API Calls**
```xml
<!-- سيء: Multiple API calls -->
<data name="name" src="api/getName"/>
<data name="balance" src="api/getBalance"/>

<!-- جيد: Single API call -->
<data name="customer" src="api/getCustomerInfo"/>
```

✅ **Use Session Variables**
```xml
<!-- تخزين البيانات مرة واحدة -->
<var name="session.customerId" expr="'123456'"/>
<!-- إعادة استخدامها -->
<value expr="session.customerId"/>
```

### 4. Security

✅ **حماية البيانات الحساسة**
```xml
<!-- Mask sensitive input -->
<field name="pin" type="digits">
    <property name="inputmodes" value="dtmf"/>
    <property name="confidencelevel" value="0.5"/>
    <prompt>Enter your 4 digit PIN</prompt>
    <!-- Don't log PIN -->
</field>
```

✅ **استخدم HTTPS**
```xml
<data name="account" src="https://secure.api.com/account"/>
```

✅ **Timeout Sessions**
```xml
<property name="timeout" value="10s"/>
```

---

## استكشاف الأخطاء

### 1. Common Issues

#### مشكلة: No Audio Playback

**الأسباب المحتملة:**
- مسار الملف الصوتي خاطئ
- تنسيق الصوت غير مدعوم
- مشاكل في الشبكة

**الحل:**
```xml
<!-- تحقق من المسار -->
<audio src="http://cvp-server/audio/welcome.wav">
    <!-- Fallback TTS -->
    Welcome to our service
</audio>
```

#### مشكلة: DTMF Not Recognized

**الأسباب:**
- إعدادات Grammar خاطئة
- مشاكل في Gateway

**الحل:**
```xml
<field name="choice">
    <grammar type="application/x-gsl">
        [
            (1) {<choice "1">}
            (2) {<choice "2">}
        ]
    </grammar>
</field>
```

### 2. Debugging Tools

#### CVP Call Studio Debugger
```
- Breakpoints في Call Flow
- Variable inspection
- Step-by-step execution
```

#### CVP Logs
```bash
# Call Server logs
/cvp/logs/callserver.log

# VXML Server logs
/cvp/logs/vxmlserver.log

# Application logs
/cvp/logs/applications/myapp.log
```

### 3. Log Analysis

**مثال على log entry:**
```
2024-01-15 10:30:45 INFO [CallID: 123456]
Session started from 966501234567
Application: CustomerService
Entry point: MainMenu
```

**ما تبحث عنه:**
- Call ID لتتبع المكالمة
- Error messages
- Session duration
- User inputs
- API response times

---

## مثال شامل: Banking IVR Application

### Call Flow Design

```
Start
  ↓
Welcome + Language Selection
  ↓
Main Menu
  ├─ 1: Check Balance
  │   ├─ Get Account Number
  │   ├─ Authenticate PIN
  │   ├─ Call Balance API
  │   └─ Play Balance
  │
  ├─ 2: Recent Transactions
  │   ├─ Get Account Number
  │   ├─ Authenticate PIN
  │   ├─ Call Transactions API
  │   └─ Play Transactions
  │
  ├─ 3: Transfer Money
  │   ├─ Authenticate
  │   ├─ Get Source Account
  │   ├─ Get Target Account
  │   ├─ Get Amount
  │   ├─ Confirm
  │   ├─ Call Transfer API
  │   └─ Confirmation
  │
  └─ 0: Speak to Agent
      └─ Transfer to ICM
```

### CVP Studio Implementation

سيتم بناء هذا باستخدام:
- **10+ Elements**: Start, Menu, Get Digits, Decision, HTTP Request, Play, End
- **15+ Variables**: language, accountNum, pin, balance, etc.
- **5+ Audio files**: Welcome, Menu, Error, Success, Goodbye
- **3+ API Integrations**: Balance, Transactions, Transfer APIs

---

## الخلاصة

### النقاط الرئيسية

1. **CVP** هو منصة IVR قوية لبناء تطبيقات صوتية
2. **CVP Studio** يسهل التطوير بدون كتابة VoiceXML
3. **التكامل مع ICM** يوفر routing ذكي
4. **VoiceXML** هو اللغة الأساسية للتطبيقات الصوتية
5. **Best Practices** ضرورية لتجربة مستخدم جيدة

### المهارات المطلوبة

- فهم IVR concepts
- CVP Studio development
- VoiceXML programming
- ICM integration
- API integration
- Debugging and troubleshooting

### الخطوات التالية

1. تعلم ICM Scripting
2. فهم Call Routing
3. التكامل مع Spring Boot APIs
4. UCCE Administration
5. Advanced troubleshooting

---

## موارد إضافية

### Documentation
- Cisco CVP Official Documentation
- VoiceXML 2.1 Specification
- CVP Studio User Guide

### Training
- Cisco CVP Administration
- CVP Application Development
- ICM/UCCE Fundamentals

### Community
- Cisco Community Forums
- Stack Overflow - CVP tags
- GitHub - CVP samples
