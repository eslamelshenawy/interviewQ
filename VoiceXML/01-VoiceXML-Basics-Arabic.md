# VoiceXML - الدليل الشامل للمبتدئين

## المحتويات
1. [ما هو VoiceXML؟](#ما-هو-voicexml)
2. [البنية الأساسية](#البنية-الأساسية)
3. [العناصر الأساسية](#العناصر-الأساسية)
4. [Dialogs - الحوارات](#dialogs---الحوارات)
5. [Forms & Fields](#forms--fields)
6. [Prompts - الرسائل الصوتية](#prompts---الرسائل-الصوتية)
7. [Grammars - قواعد الإدخال](#grammars---قواعد-الإدخال)
8. [Variables - المتغيرات](#variables---المتغيرات)
9. [Control Flow - التحكم بالتدفق](#control-flow---التحكم-بالتدفق)
10. [أمثلة عملية](#أمثلة-عملية)

---

## ما هو VoiceXML؟

### التعريف

**VoiceXML (Voice Extensible Markup Language)** هي لغة قائمة على XML لبناء تطبيقات صوتية تفاعلية (IVR).

### لماذا VoiceXML؟

✅ **معيار عالمي**: W3C Standard
✅ **سهل التعلم**: مشابه لـ HTML
✅ **قوي**: يدعم DTMF والتعرف على الكلام
✅ **متكامل**: يتصل بـ APIs وقواعد البيانات
✅ **متعدد المنصات**: يعمل على معظم أنظمة IVR

### مثال بسيط

```xml
<?xml version="1.0"?>
<vxml version="2.1">
    <form>
        <block>
            <prompt>مرحباً بك في نظام IVR!</prompt>
        </block>
    </form>
</vxml>
```

**ماذا يحدث:**
1. النظام يشغل الملف
2. يجد `<prompt>`
3. يشغل الرسالة الصوتية "مرحباً بك في نظام IVR!"
4. ينتهي

---

## البنية الأساسية

### الهيكل الأساسي لملف VoiceXML

```xml
<?xml version="1.0" encoding="UTF-8"?>
<vxml version="2.1" xmlns="http://www.w3.org/2001/vxml">

    <!-- Document level settings -->
    <property name="timeout" value="5s"/>
    <property name="inputmodes" value="dtmf voice"/>

    <!-- Variables -->
    <var name="applicationName" expr="'Banking IVR'"/>

    <!-- Dialogs (Forms and Menus) -->
    <form id="mainMenu">
        <!-- Content here -->
    </form>

</vxml>
```

### شرح المكونات

**1. XML Declaration:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
```
- يعرّف أن هذا ملف XML
- `encoding="UTF-8"` يدعم العربية والإنجليزية

**2. VXML Root Element:**
```xml
<vxml version="2.1" xmlns="http://www.w3.org/2001/vxml">
```
- العنصر الجذر
- `version="2.1"` إصدار VoiceXML
- `xmlns` Namespace للتحقق

**3. Properties:**
```xml
<property name="timeout" value="5s"/>
```
- إعدادات عامة للتطبيق
- مثال: المدة قبل timeout

---

## العناصر الأساسية

### 1. `<vxml>` - العنصر الجذر

```xml
<vxml version="2.1">
    <!-- All content here -->
</vxml>
```

**الوصف:** العنصر الرئيسي الذي يحتوي على كل شيء

---

### 2. `<form>` - النموذج

```xml
<form id="welcomeForm">
    <block>
        <prompt>مرحباً</prompt>
    </block>
</form>
```

**الوصف:** حاوية للحوارات التفاعلية

**الاستخدام:**
- جمع معلومات من المستخدم
- تنفيذ منطق معقد
- معظم التطبيقات تستخدم forms

---

### 3. `<block>` - كتلة التنفيذ

```xml
<block>
    <prompt>هذه رسالة</prompt>
    <assign name="counter" expr="counter + 1"/>
</block>
```

**الوصف:** ينفذ أكواد بدون انتظار input من المستخدم

**الاستخدام:**
- تشغيل رسائل
- حسابات
- استدعاء APIs

---

### 4. `<field>` - حقل الإدخال

```xml
<field name="accountNumber" type="digits">
    <prompt>أدخل رقم حسابك</prompt>
</field>
```

**الوصف:** يجمع معلومات من المستخدم

**أنواع:**
- `digits`: أرقام فقط
- `boolean`: yes/no
- `date`: تاريخ
- `time`: وقت
- `number`: رقم
- `phone`: رقم هاتف

---

### 5. `<prompt>` - الرسالة الصوتية

```xml
<prompt>
    مرحباً بك في البنك
</prompt>
```

**الوصف:** يشغل رسالة صوتية للمستخدم

**أنواع:**
- TTS (Text-to-Speech)
- Audio files (WAV)
- مزيج من الاثنين

---

### 6. `<menu>` - القائمة

```xml
<menu id="mainMenu">
    <prompt>اضغط 1 للمبيعات، 2 للدعم الفني</prompt>
    <choice dtmf="1" next="#sales">المبيعات</choice>
    <choice dtmf="2" next="#support">الدعم الفني</choice>
</menu>
```

**الوصف:** قائمة خيارات للمستخدم

---

### 7. `<audio>` - ملف صوتي

```xml
<audio src="welcome.wav">
    مرحباً بك
</audio>
```

**الوصف:** يشغل ملف صوتي مسجل مسبقاً

**Fallback:** إذا فشل الملف، يشغل النص (TTS)

---

### 8. `<goto>` - الانتقال

```xml
<goto next="#mainMenu"/>
```

**الوصف:** ينتقل إلى dialog آخر

**أنواع:**
- `next="#formId"`: داخل نفس الملف
- `next="other.vxml"`: ملف آخر
- `expr="dynamicURL"`: URL ديناميكي

---

### 9. `<if>`, `<elseif>`, `<else>` - الشروط

```xml
<if cond="accountType == 'Premium'">
    <prompt>مرحباً عميلنا المميز</prompt>
<elseif cond="accountType == 'Standard'"/>
    <prompt>مرحباً</prompt>
<else/>
    <prompt>مرحباً بك</prompt>
</if>
```

**الوصف:** شروط منطقية

---

### 10. `<assign>` - تعيين قيمة

```xml
<assign name="customerName" expr="'أحمد علي'"/>
<assign name="balance" expr="5000"/>
```

**الوصف:** يعين قيمة لمتغير

---

## Dialogs - الحوارات

في VoiceXML، هناك نوعان من الحوارات:

### 1. Forms

**الاستخدام:** جمع معلومات من المستخدم

```xml
<form id="getAccountInfo">
    <!-- Field 1: Account Number -->
    <field name="accountNumber" type="digits">
        <prompt>أدخل رقم حسابك المكون من 10 أرقام</prompt>

        <filled>
            <if cond="accountNumber.length != 10">
                <clear field="accountNumber"/>
                <prompt>رقم الحساب يجب أن يكون 10 أرقام. حاول مرة أخرى.</prompt>
                <reprompt/>
            </if>
        </filled>
    </field>

    <!-- Field 2: PIN -->
    <field name="pin" type="digits">
        <prompt>أدخل رقمك السري المكون من 4 أرقام</prompt>

        <filled>
            <if cond="pin.length == 4">
                <prompt>شكراً. جاري التحقق...</prompt>
                <goto next="#validateCredentials"/>
            <else/>
                <clear field="pin"/>
                <prompt>الرقم السري يجب أن يكون 4 أرقام</prompt>
                <reprompt/>
            </if>
        </filled>
    </field>
</form>
```

**كيف يعمل Form:**
```
1. VoiceXML يبدأ بأول field (accountNumber)
   ↓
2. يشغل prompt: "أدخل رقم حسابك..."
   ↓
3. ينتظر input من المستخدم
   ↓
4. عند الإدخال، ينفذ <filled>
   ↓
5. يتحقق من الطول (10 أرقام)
   ↓
6. إذا صحيح: ينتقل للـ field التالي (pin)
   إذا خاطئ: يعيد المحاولة
   ↓
7. يكرر نفس العملية للـ PIN
```

---

### 2. Menus

**الاستخدام:** عرض خيارات للمستخدم

```xml
<menu id="mainMenu">
    <prompt>
        القائمة الرئيسية.
        اضغط 1 للاستعلام عن الرصيد.
        اضغط 2 لعرض آخر العمليات.
        اضغط 3 لتحويل الأموال.
        اضغط 0 للتحدث مع موظف.
    </prompt>

    <choice dtmf="1" next="#checkBalance">
        الاستعلام عن الرصيد
    </choice>

    <choice dtmf="2" next="#viewTransactions">
        عرض آخر العمليات
    </choice>

    <choice dtmf="3" next="#transferMoney">
        تحويل الأموال
    </choice>

    <choice dtmf="0" next="#transferToAgent">
        التحدث مع موظف
    </choice>

    <noinput>
        لم أسمع شيئاً. من فضلك اختر من القائمة.
        <reprompt/>
    </noinput>

    <nomatch>
        لم أفهم اختيارك. من فضلك اختر رقماً من 0 إلى 3.
        <reprompt/>
    </nomatch>
</menu>
```

---

## Forms & Fields

### Field Types

#### 1. `digits` - أرقام

```xml
<field name="accountNumber" type="digits">
    <prompt>أدخل رقم حسابك</prompt>

    <!-- Min/Max length -->
    <grammar type="application/x-gsl">
        [0-9]{10}  <!-- Exactly 10 digits -->
    </grammar>
</field>
```

---

#### 2. `boolean` - نعم/لا

```xml
<field name="confirmTransfer" type="boolean">
    <prompt>
        هل تريد تأكيد التحويل؟
        قل نعم أو لا، أو اضغط 1 لنعم، 2 للا.
    </prompt>

    <filled>
        <if cond="confirmTransfer == true">
            <goto next="#executeTransfer"/>
        <else/>
            <goto next="#cancelTransfer"/>
        </if>
    </filled>
</field>
```

---

#### 3. `date` - تاريخ

```xml
<field name="birthDate" type="date">
    <prompt>
        ما هو تاريخ ميلادك؟
        قل السنة ثم الشهر ثم اليوم.
    </prompt>

    <filled>
        <prompt>
            تاريخ ميلادك هو
            <value expr="birthDate"/>
        </prompt>
    </filled>
</field>
```

---

#### 4. Custom Type

```xml
<field name="menuChoice">
    <prompt>اختر خدمة</prompt>

    <grammar type="application/x-gsl">
        [
            (balance رصيد) {<menuChoice "balance">}
            (transfer تحويل) {<menuChoice "transfer">}
            (agent موظف) {<menuChoice "agent">}
        ]
    </grammar>

    <filled>
        <if cond="menuChoice == 'balance'">
            <goto next="#balanceInquiry"/>
        </if>
        <!-- ... -->
    </filled>
</field>
```

---

### Field Properties

```xml
<field name="accountNumber">
    <!-- Timeout: كم ثانية ينتظر input -->
    <property name="timeout" value="5s"/>

    <!-- Input modes: DTMF أو Speech أو كلاهما -->
    <property name="inputmodes" value="dtmf voice"/>

    <!-- Confidence level للـ speech recognition -->
    <property name="confidencelevel" value="0.5"/>

    <!-- Number of speech recognition results -->
    <property name="maxnbest" value="3"/>

    <prompt>أدخل رقم حسابك</prompt>
</field>
```

---

## Prompts - الرسائل الصوتية

### أنواع Prompts

#### 1. Text-to-Speech (TTS)

```xml
<prompt>
    مرحباً بك في البنك الأهلي
</prompt>
```

**المميزات:**
- ✅ سهل: فقط اكتب النص
- ✅ ديناميكي: يمكن تغيير النص برمجياً
- ❌ جودة أقل من التسجيل

---

#### 2. Pre-recorded Audio

```xml
<prompt>
    <audio src="http://ivr.bank.com/audio/welcome.wav">
        مرحباً بك في البنك
    </audio>
</prompt>
```

**المميزات:**
- ✅ جودة عالية
- ✅ صوت احترافي
- ❌ يحتاج تسجيل مسبق
- ❌ صعب التغيير

---

#### 3. Mixed Prompts

```xml
<prompt>
    <!-- Audio file -->
    <audio src="welcome.wav">مرحباً</audio>

    <!-- Dynamic TTS -->
    اسمك هو <value expr="customerName"/>

    <!-- Audio file -->
    <audio src="balance_is.wav">رصيدك هو</audio>

    <!-- Dynamic number -->
    <value expr="accountBalance"/>

    ريال سعودي.
</prompt>
```

---

### SSML - Speech Synthesis Markup Language

تحسين جودة TTS:

```xml
<prompt>
    <speak>
        <!-- Emphasis: تشديد -->
        مرحباً <emphasis level="strong">بك</emphasis> في البنك

        <!-- Break: وقفة -->
        من فضلك انتظر <break time="1s"/> جاري التحميل

        <!-- Say-as: نطق خاص -->
        رقم حسابك هو
        <say-as interpret-as="digits">123456</say-as>

        <!-- Prosody: سرعة ونبرة الصوت -->
        <prosody rate="slow" pitch="high">
            هذا مهم جداً
        </prosody>

        <!-- Audio مع SSML -->
        <audio src="welcome.wav">
            مرحباً بك
        </audio>
    </speak>
</prompt>
```

---

### Prompt Examples

#### مثال: رسالة ترحيب

```xml
<prompt>
    <audio src="welcome.wav">
        مرحباً بك في البنك الأهلي السعودي
    </audio>
    <break time="500ms"/>
    للغة العربية اضغط 1
    <break time="300ms"/>
    For English press 2
</prompt>
```

---

#### مثال: رسالة ديناميكية

```xml
<prompt>
    مرحباً <value expr="customerName"/>
    <break time="500ms"/>

    <!-- Say number as currency -->
    رصيدك الحالي هو
    <say-as interpret-as="currency">
        <value expr="accountBalance"/>
    </say-as>
    ريال سعودي
</prompt>
```

---

#### مثال: قائمة خيارات

```xml
<prompt bargein="true">
    <!-- bargein="true" يسمح للمستخدم بالمقاطعة -->

    القائمة الرئيسية:
    <break time="500ms"/>

    <emphasis>للاستعلام عن الرصيد</emphasis> اضغط 1
    <break time="300ms"/>

    <emphasis>لتحويل الأموال</emphasis> اضغط 2
    <break time="300ms"/>

    <emphasis>للتحدث مع موظف</emphasis> اضغط 0
</prompt>
```

---

## Grammars - قواعد الإدخال

### ما هو Grammar؟

**Grammar** يحدد ما هو الإدخال المقبول من المستخدم.

### أنواع Grammars

#### 1. Built-in Grammars

```xml
<!-- Boolean: yes/no -->
<field name="confirm" type="boolean">
    <prompt>هل أنت متأكد؟</prompt>
</field>

<!-- Digits: numbers only -->
<field name="account" type="digits">
    <prompt>أدخل رقم الحساب</prompt>
</field>

<!-- Date -->
<field name="date" type="date">
    <prompt>ما هو التاريخ؟</prompt>
</field>
```

---

#### 2. DTMF Grammar (GSL)

```xml
<field name="menuChoice">
    <prompt>اضغط 1 أو 2 أو 3</prompt>

    <grammar type="application/x-gsl">
        [
            (1) {<menuChoice "balance">}
            (2) {<menuChoice "transfer">}
            (3) {<menuChoice "agent">}
        ]
    </grammar>
</field>
```

**شرح:**
- `(1)`: المستخدم يضغط 1
- `{<menuChoice "balance">}`: قيمة المتغير تصبح "balance"

---

#### 3. Speech Grammar (GSL)

```xml
<field name="serviceType">
    <prompt>قل: رصيد، تحويل، أو موظف</prompt>

    <grammar type="application/x-gsl">
        [
            (رصيد balance) {<serviceType "balance">}
            (تحويل transfer) {<serviceType "transfer">}
            (موظف agent) {<serviceType "agent">}
        ]
    </grammar>
</field>
```

**يقبل:**
- المستخدم يقول "رصيد" بالعربي
- المستخدم يقول "balance" بالإنجليزي
- كلاهما يعطي نفس القيمة

---

#### 4. SRGS Grammar (XML Format)

```xml
<field name="yesNo">
    <prompt>هل تريد المتابعة؟</prompt>

    <grammar type="application/srgs+xml" mode="voice">
        <rule id="yesNo">
            <one-of>
                <item>نعم</item>
                <item>yes</item>
                <item>أوافق</item>
                <item>موافق</item>
            </one-of>
        </rule>
    </grammar>
</field>
```

---

#### 5. Complex Grammar

```xml
<field name="accountNumber">
    <prompt>أدخل رقم حسابك المكون من 10 أرقام</prompt>

    <grammar type="application/x-gsl">
        <!-- Exactly 10 digits -->
        [
            [0-9]{10}
        ]
    </grammar>

    <!-- Alternative: SRGS -->
    <grammar type="application/srgs+xml">
        <rule id="accountNumber">
            <item repeat="10">
                <one-of>
                    <item>0</item>
                    <item>1</item>
                    <item>2</item>
                    <item>3</item>
                    <item>4</item>
                    <item>5</item>
                    <item>6</item>
                    <item>7</item>
                    <item>8</item>
                    <item>9</item>
                </one-of>
            </item>
        </rule>
    </grammar>
</field>
```

---

### Grammar with Semantic Interpretation

```xml
<field name="service">
    <prompt>ماذا تريد أن تفعل؟</prompt>

    <grammar type="application/x-gsl">
        [
            (
                (check [my] balance)     |
                (استعلام [عن] رصيد)      |
                (رصيد)
            ) {<service "balance">}

            (
                (transfer money)         |
                (تحويل أموال)            |
                (تحويل)
            ) {<service "transfer">}

            (
                ([speak to] [an] agent)  |
                (موظف)                   |
                (التحدث مع موظف)
            ) {<service "agent">}
        ]
    </grammar>

    <filled>
        <if cond="service == 'balance'">
            <goto next="#checkBalance"/>
        <elseif cond="service == 'transfer'"/>
            <goto next="#transferMoney"/>
        <elseif cond="service == 'agent'"/>
            <goto next="#transferToAgent"/>
        </if>
    </filled>
</field>
```

**يقبل أي من:**
- "check my balance"
- "check balance"
- "استعلام عن رصيد"
- "استعلام رصيد"
- "رصيد"

---

## Variables - المتغيرات

### أنواع المتغيرات

#### 1. `<var>` - متغير محلي

```xml
<form id="example">
    <!-- Declare variable -->
    <var name="counter" expr="0"/>
    <var name="customerName" expr="'أحمد'"/>
    <var name="balance" expr="5000"/>

    <block>
        <!-- Use variable -->
        <prompt>
            مرحباً <value expr="customerName"/>
            رصيدك هو <value expr="balance"/>
        </prompt>

        <!-- Modify variable -->
        <assign name="counter" expr="counter + 1"/>
    </block>
</form>
```

**النطاق (Scope):** داخل الـ form فقط

---

#### 2. Application Variables

```xml
<vxml version="2.1" application="app-root.vxml">
    <!-- في app-root.vxml -->
    <var name="application.name" expr="'Banking IVR'"/>
    <var name="application.version" expr="'1.0'"/>
</vxml>

<!-- في ملف آخر -->
<vxml version="2.1">
    <form>
        <block>
            <!-- يمكن الوصول للمتغيرات من أي ملف -->
            <prompt>
                <value expr="application.name"/>
                version <value expr="application.version"/>
            </prompt>
        </block>
    </form>
</vxml>
```

**النطاق:** متاح في كل التطبيق

---

#### 3. Session Variables

```xml
<var name="session.customerId" expr="'123456'"/>
<var name="session.language" expr="'ar'"/>
<var name="session.authenticated" expr="false"/>
```

**النطاق:** طوال جلسة المكالمة

---

#### 4. Field Variables

```xml
<field name="accountNumber">
    <prompt>أدخل رقم حسابك</prompt>

    <filled>
        <!-- accountNumber is the variable -->
        <prompt>
            رقم حسابك هو <value expr="accountNumber"/>
        </prompt>

        <!-- Special properties -->
        <if cond="accountNumber$.confidence &lt; 0.5">
            <prompt>لست متأكداً من الرقم</prompt>
        </if>
    </filled>
</field>
```

**Properties:**
- `accountNumber`: القيمة
- `accountNumber$.confidence`: مستوى الثقة (speech)
- `accountNumber$.utterance`: ما قاله المستخدم
- `accountNumber$.inputmode`: "dtmf" أو "voice"

---

### Variable Operations

```xml
<block>
    <!-- Assignment -->
    <assign name="x" expr="10"/>

    <!-- Arithmetic -->
    <assign name="y" expr="x + 5"/>      <!-- 15 -->
    <assign name="z" expr="x * 2"/>      <!-- 20 -->

    <!-- String concatenation -->
    <assign name="fullName" expr="firstName + ' ' + lastName"/>

    <!-- Comparison -->
    <if cond="balance &gt; 1000">
        <prompt>رصيد كافٍ</prompt>
    </if>

    <!-- Logical operations -->
    <if cond="age &gt;= 18 &amp;&amp; hasAccount == true">
        <prompt>يمكنك المتابعة</prompt>
    </if>
</block>
```

**ملاحظة:** في XML:
- `>` يُكتب `&gt;`
- `<` يُكتب `&lt;`
- `&&` يُكتب `&amp;&amp;`

---

## Control Flow - التحكم بالتدفق

### 1. `<if>`, `<elseif>`, `<else>`

```xml
<block>
    <if cond="customerType == 'VIP'">
        <prompt>مرحباً عميلنا المميز</prompt>
        <assign name="priority" expr="10"/>

    <elseif cond="customerType == 'Premium'"/>
        <prompt>مرحباً عميلنا العزيز</prompt>
        <assign name="priority" expr="5"/>

    <else/>
        <prompt>مرحباً بك</prompt>
        <assign name="priority" expr="1"/>
    </if>
</block>
```

---

### 2. `<goto>`

```xml
<!-- Go to form in same document -->
<goto next="#mainMenu"/>

<!-- Go to another document -->
<goto next="main-menu.vxml"/>

<!-- Go with data -->
<goto next="balance.vxml"
      expr="'balance.vxml?account=' + accountNumber"/>
```

---

### 3. `<submit>`

```xml
<!-- Submit data to server -->
<submit
    next="http://api.bank.com/authenticate"
    method="post"
    namelist="accountNumber pin"
    enctype="application/x-www-form-urlencoded"/>
```

**يرسل:**
```
POST /authenticate
accountNumber=123456&pin=1234
```

---

### 4. `<return>`

```xml
<form id="authenticate">
    <field name="pin">
        <prompt>أدخل الرقم السري</prompt>

        <filled>
            <!-- Validate PIN -->
            <if cond="pin == '1234'">
                <!-- Return success -->
                <return namelist="pin" event="success"/>
            <else/>
                <!-- Return failure -->
                <return event="auth.failed"/>
            </if>
        </filled>
    </field>
</form>
```

---

### 5. `<exit>`

```xml
<block>
    <prompt>شكراً لاستخدامك خدماتنا. مع السلامة</prompt>
    <exit/>
</block>
```

---

## أمثلة عملية

### مثال 1: تطبيق IVR بسيط

```xml
<?xml version="1.0"?>
<vxml version="2.1">

    <!-- Welcome Form -->
    <form id="welcome">
        <block>
            <prompt>
                مرحباً بك في بنك الأهلي
            </prompt>
            <goto next="#mainMenu"/>
        </block>
    </form>

    <!-- Main Menu -->
    <menu id="mainMenu">
        <prompt>
            القائمة الرئيسية.
            اضغط 1 للاستعلام عن الرصيد.
            اضغط 2 للتحدث مع موظف.
        </prompt>

        <choice dtmf="1" next="#checkBalance">
            الاستعلام عن الرصيد
        </choice>

        <choice dtmf="2" next="#transferAgent">
            التحدث مع موظف
        </choice>
    </menu>

    <!-- Check Balance -->
    <form id="checkBalance">
        <block>
            <prompt>
                رصيدك الحالي هو 5000 ريال سعودي
            </prompt>
            <goto next="#goodbye"/>
        </block>
    </form>

    <!-- Transfer to Agent -->
    <form id="transferAgent">
        <block>
            <prompt>
                من فضلك انتظر، جاري تحويلك للموظف
            </prompt>
            <transfer dest="tel:+966112345678"/>
        </block>
    </form>

    <!-- Goodbye -->
    <form id="goodbye">
        <block>
            <prompt>
                شكراً لاستخدامك خدماتنا. مع السلامة
            </prompt>
            <exit/>
        </block>
    </form>

</vxml>
```

---

### مثال 2: جمع معلومات المستخدم

```xml
<?xml version="1.0"?>
<vxml version="2.1">

    <!-- Variables -->
    <var name="accountNumber"/>
    <var name="pin"/>
    <var name="authenticated" expr="false"/>

    <!-- Get Account Number -->
    <form id="getAccount">
        <field name="accountNumber" type="digits">
            <prompt>
                أدخل رقم حسابك المكون من 10 أرقام
            </prompt>

            <filled>
                <if cond="accountNumber.length == 10">
                    <prompt>شكراً</prompt>
                    <goto next="#getPin"/>
                <else/>
                    <clear field="accountNumber"/>
                    <prompt>
                        رقم الحساب يجب أن يكون 10 أرقام.
                        حاول مرة أخرى.
                    </prompt>
                    <reprompt/>
                </if>
            </filled>

            <noinput count="1">
                لم أسمع شيئاً. من فضلك أدخل رقم حسابك.
                <reprompt/>
            </noinput>

            <noinput count="2">
                ما زلت لم أسمع شيئاً.
                دعني أحولك لموظف.
                <goto next="#transferAgent"/>
            </noinput>
        </field>
    </form>

    <!-- Get PIN -->
    <form id="getPin">
        <var name="attempts" expr="0"/>
        <var name="maxAttempts" expr="3"/>

        <field name="pin" type="digits">
            <property name="timeout" value="10s"/>

            <prompt>
                أدخل رقمك السري المكون من 4 أرقام
            </prompt>

            <filled>
                <if cond="pin.length == 4">
                    <!-- Call API to validate -->
                    <data name="authResult"
                          src="'http://api.bank.com/auth?account=' + accountNumber + '&amp;pin=' + pin"
                          method="get"/>

                    <if cond="authResult.valid == true">
                        <!-- Success -->
                        <assign name="authenticated" expr="true"/>
                        <prompt>تم قبول الرقم السري</prompt>
                        <goto next="#mainMenu"/>

                    <else/>
                        <!-- Failed -->
                        <assign name="attempts" expr="attempts + 1"/>

                        <if cond="attempts &lt; maxAttempts">
                            <clear field="pin"/>
                            <prompt>
                                الرقم السري غير صحيح.
                                لديك <value expr="maxAttempts - attempts"/>
                                محاولات متبقية.
                            </prompt>
                            <reprompt/>
                        <else/>
                            <prompt>
                                لقد تجاوزت الحد الأقصى للمحاولات.
                                سيتم تحويلك لموظف.
                            </prompt>
                            <goto next="#transferAgent"/>
                        </if>
                    </if>

                <else/>
                    <clear field="pin"/>
                    <prompt>
                        الرقم السري يجب أن يكون 4 أرقام
                    </prompt>
                    <reprompt/>
                </if>
            </filled>
        </field>
    </form>

    <!-- Main Menu -->
    <form id="mainMenu">
        <field name="choice">
            <prompt>
                القائمة الرئيسية.
                اضغط 1 للرصيد.
                اضغط 2 للعمليات.
                اضغط 0 للموظف.
            </prompt>

            <grammar type="application/x-gsl">
                [(1) {<choice "1">}
                 (2) {<choice "2">}
                 (0) {<choice "0">}]
            </grammar>

            <filled>
                <if cond="choice == '1'">
                    <goto next="#checkBalance"/>
                <elseif cond="choice == '2'"/>
                    <goto next="#viewTransactions"/>
                <elseif cond="choice == '0'"/>
                    <goto next="#transferAgent"/>
                </if>
            </filled>
        </field>
    </form>

</vxml>
```

---

### مثال 3: التكامل مع API

```xml
<?xml version="1.0"?>
<vxml version="2.1">

    <form id="checkBalance">
        <var name="accountNumber" expr="'1234567890'"/>
        <var name="balance"/>

        <block>
            <prompt>
                من فضلك انتظر، جاري الاستعلام عن رصيدك
            </prompt>

            <!-- Call REST API -->
            <data name="apiResponse"
                  src="'https://api.bank.com/accounts/' + accountNumber + '/balance'"
                  method="get"
                  timeout="10s">

                <!-- Headers -->
                <property name="Authorization"
                          value="'Bearer eyJhbGc...'"/>
                <property name="Content-Type"
                          value="'application/json'"/>
            </data>

            <!-- Check if API call succeeded -->
            <if cond="apiResponse != null">
                <!-- Success -->
                <assign name="balance" expr="apiResponse.balance"/>

                <prompt>
                    رصيد حسابك
                    <say-as interpret-as="currency">
                        <value expr="balance"/>
                    </say-as>
                    ريال سعودي
                </prompt>

                <!-- More services? -->
                <prompt>
                    هل تريد خدمة أخرى؟
                    اضغط 1 لنعم، 2 للخروج
                </prompt>

                <field name="moreServices">
                    <grammar type="application/x-gsl">
                        [(1) {<moreServices "yes">}
                         (2) {<moreServices "no">}]
                    </grammar>

                    <filled>
                        <if cond="moreServices == 'yes'">
                            <goto next="#mainMenu"/>
                        <else/>
                            <goto next="#goodbye"/>
                        </if>
                    </filled>
                </field>

            <else/>
                <!-- API call failed -->
                <prompt>
                    عذراً، لا أستطيع الاستعلام عن رصيدك الآن.
                    اضغط 1 لإعادة المحاولة، أو 0 للتحدث مع موظف.
                </prompt>

                <field name="retryChoice">
                    <grammar type="application/x-gsl">
                        [(1) {<retryChoice "retry">}
                         (0) {<retryChoice "agent">}]
                    </grammar>

                    <filled>
                        <if cond="retryChoice == 'retry'">
                            <goto next="#checkBalance"/>
                        <else/>
                            <goto next="#transferAgent"/>
                        </if>
                    </filled>
                </field>
            </if>
        </block>
    </form>

</vxml>
```

---

## الخلاصة

### ما تعلمناه:

✅ **أساسيات VoiceXML**: البنية والعناصر
✅ **Dialogs**: Forms و Menus
✅ **Fields**: أنواع الحقول وكيفية جمع البيانات
✅ **Prompts**: رسائل صوتية (TTS و Audio)
✅ **Grammars**: قواعد إدخال DTMF و Speech
✅ **Variables**: أنواع المتغيرات واستخدامها
✅ **Control Flow**: التحكم بتدفق التطبيق
✅ **أمثلة عملية**: تطبيقات IVR كاملة

### الخطوة التالية:

1. **تطبيق عملي**: بناء IVR بسيط
2. **VoiceXML المتقدم**: Error handling, Events, Subdialogs
3. **التكامل**: APIs, Databases, ICM
4. **التحسين**: Performance, Security

---

**تابع للجزء الثاني: VoiceXML Advanced** 🚀
