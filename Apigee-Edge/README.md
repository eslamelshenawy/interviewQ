# Apigee Edge - دليل شامل

## المحتويات
1. [ما هو Apigee Edge](#ما-هو-apigee-edge)
2. [المفاهيم الأساسية](#المفاهيم-الأساسية)
3. [المكونات الرئيسية](#المكونات-الرئيسية)
4. [الميزات الرئيسية](#الميزات-الرئيسية)
5. [API Proxies](#api-proxies)
6. [Policies](#policies)
7. [حالات الاستخدام](#حالات-الاستخدام)
8. [أمثلة عملية](#أمثلة-عملية)
9. [أسئلة المقابلات](#أسئلة-المقابلات)

---

## ما هو Apigee Edge

**Apigee Edge** هو منصة API Management من Google Cloud تساعد في تصميم، تأمين، نشر، مراقبة، وتحليل الـ APIs. يوفر Apigee طبقة وسيطة بين الـ Backend Services والـ Client Applications.

### المزايا الرئيسية
- **API Gateway**: توجيه وإدارة الطلبات
- **Security**: تأمين APIs بطرق متعددة (OAuth, JWT, API Keys)
- **Analytics**: تحليلات مفصلة عن استخدام APIs
- **Developer Portal**: بوابة للمطورين لاستكشاف APIs
- **Monetization**: إمكانية تحقيق الدخل من APIs
- **Mediation**: تحويل الطلبات والاستجابات

---

## المفاهيم الأساسية

### 1. API Proxy
واجهة تمثل الـ Backend Service، حيث يتم استقبال الطلبات من العملاء وتوجيهها إلى الـ Backend.

### 2. Target Endpoint
الـ Backend Service الفعلي الذي يتم استدعاؤه من خلال API Proxy.

### 3. Proxy Endpoint
النقطة التي يستقبل فيها Apigee الطلبات من العملاء.

### 4. Policies
قواعد وإجراءات تطبق على الطلبات والاستجابات (مثل Authentication، Rate Limiting، Transformation).

### 5. Products
مجموعة من API Proxies يتم تجميعها معًا لتوفيرها للمطورين.

### 6. Developer Apps
التطبيقات التي يقوم المطورون بإنشائها للوصول إلى APIs.

### 7. Environment
بيئات مختلفة (Development, Testing, Production) لنشر API Proxies.

---

## المكونات الرئيسية

### 1. Management Server
- إدارة وتهيئة API Proxies
- إدارة المستخدمين والصلاحيات
- نشر APIs على البيئات المختلفة

### 2. Runtime (Message Processor)
- معالجة الطلبات والاستجابات
- تطبيق الـ Policies
- توجيه الطلبات إلى Backend

### 3. Analytics
- جمع البيانات عن استخدام APIs
- تقارير مفصلة عن الأداء
- مراقبة الأخطاء والمشاكل

### 4. Developer Portal
- واجهة للمطورين
- وثائق APIs (API Documentation)
- إدارة API Keys والتطبيقات

---

## الميزات الرئيسية

### 1. Traffic Management
```yaml
Rate Limiting:
  - Spike Arrest: منع الهجمات المفاجئة
  - Quota: تحديد عدد الطلبات في فترة زمنية
  - Concurrent Rate Limit: تحديد عدد الطلبات المتزامنة
```

### 2. Security Features
- **API Keys**: مفاتيح للتحقق من الوصول
- **OAuth 2.0**: توكينات للتفويض
- **JWT**: التحقق من JSON Web Tokens
- **SAML**: دعم Security Assertion Markup Language
- **IP Whitelisting/Blacklisting**: التحكم بالوصول حسب IP

### 3. Mediation & Transformation
- تحويل XML إلى JSON والعكس
- تعديل Headers
- تغيير هيكل البيانات (Payload Transformation)
- JavaScript و Python policies لمعالجة مخصصة

### 4. Caching
- Response Caching لتحسين الأداء
- تقليل الحمل على Backend Services
- Cache Management Policies

### 5. Analytics & Monitoring
- Real-time Analytics
- Custom Reports
- Error Analysis
- Performance Metrics
- API Traffic Patterns

---

## API Proxies

### هيكل API Proxy

```
API Proxy
├── Proxy Endpoint (ProxyEndpoint)
│   ├── PreFlow
│   ├── Conditional Flows
│   └── PostFlow
├── Target Endpoint (TargetEndpoint)
│   ├── PreFlow
│   ├── Conditional Flows
│   └── PostFlow
└── Resources
    ├── JavaScript
    ├── Python
    ├── Java
    └── XSL
```

### مراحل معالجة الطلب (Request Flow)

```
Client Request
    ↓
Proxy Endpoint PreFlow
    ↓
Proxy Endpoint Conditional Flows
    ↓
Proxy Endpoint PostFlow
    ↓
Target Endpoint PreFlow
    ↓
Target Endpoint Conditional Flows
    ↓
Backend Service
    ↓
Target Endpoint PostFlow
    ↓
Proxy Endpoint Response PostFlow
    ↓
Client Response
```

---

## Policies

### 1. Security Policies

#### Verify API Key
```xml
<VerifyAPIKey name="VAK-VerifyKey">
    <APIKey ref="request.queryparam.apikey"/>
</VerifyAPIKey>
```

#### OAuth v2.0
```xml
<OAuthV2 name="OAuth-GenerateAccessToken">
    <Operation>GenerateAccessToken</Operation>
    <ExpiresIn>3600000</ExpiresIn>
    <GenerateResponse enabled="true"/>
</OAuthV2>
```

#### Verify JWT
```xml
<VerifyJWT name="JWT-Verify">
    <Algorithm>RS256</Algorithm>
    <Source>request.header.authorization</Source>
    <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
    <PublicKey>
        <Value ref="public.key"/>
    </PublicKey>
</VerifyJWT>
```

### 2. Traffic Management Policies

#### Spike Arrest
```xml
<SpikeArrest name="SA-10pm">
    <Rate>10pm</Rate>
    <Identifier ref="client_id"/>
</SpikeArrest>
```

#### Quota
```xml
<Quota name="Q-QuotaPolicy">
    <Allow count="1000" countRef="verifyapikey.VAK-VerifyKey.apiproduct.developer.quota.limit"/>
    <Interval>1</Interval>
    <TimeUnit>month</TimeUnit>
    <Identifier ref="request.queryparam.apikey"/>
</Quota>
```

### 3. Mediation Policies

#### Assign Message (تعديل Request/Response)
```xml
<AssignMessage name="AM-SetHeaders">
    <Set>
        <Headers>
            <Header name="Content-Type">application/json</Header>
            <Header name="X-Custom-Header">CustomValue</Header>
        </Headers>
    </Set>
</AssignMessage>
```

#### JSON to XML
```xml
<JSONToXML name="JSON-to-XML">
    <Source>request</Source>
    <OutputVariable>xmlRequest</OutputVariable>
</JSONToXML>
```

#### Extract Variables
```xml
<ExtractVariables name="EV-ExtractUserID">
    <Source>request</Source>
    <JSONPayload>
        <Variable name="userId">
            <JSONPath>$.user.id</JSONPath>
        </Variable>
    </JSONPayload>
</ExtractVariables>
```

### 4. Extension Policies

#### JavaScript
```xml
<Javascript name="JS-CustomLogic" timeLimit="200">
    <ResourceURL>jsc://custom-logic.js</ResourceURL>
</Javascript>
```

مثال على JavaScript Policy:
```javascript
// custom-logic.js
var userId = context.getVariable("request.queryparam.userId");
var timestamp = Date.now();

context.setVariable("custom.timestamp", timestamp);
context.setVariable("custom.userId", userId);

// تعديل Response
var response = JSON.parse(context.getVariable("response.content"));
response.metadata = {
    timestamp: timestamp,
    userId: userId
};
context.setVariable("response.content", JSON.stringify(response));
```

### 5. Cache Policies

#### Response Cache
```xml
<ResponseCache name="RC-CacheResponse">
    <CacheKey>
        <Prefix>userCache</Prefix>
        <KeyFragment ref="request.queryparam.userId"/>
    </CacheKey>
    <ExpirySettings>
        <TimeoutInSec>600</TimeoutInSec>
    </ExpirySettings>
</ResponseCache>
```

---

## حالات الاستخدام

### 1. API Gateway للـ Microservices
استخدام Apigee كنقطة دخول واحدة لجميع Microservices.

### 2. Legacy System Modernization
ربط Legacy Systems القديمة مع تطبيقات حديثة من خلال APIs.

### 3. Mobile Backend
توفير APIs محسنة لتطبيقات الموبايل مع Rate Limiting وSecurity.

### 4. Third-Party Integration
توفير APIs للشركاء الخارجيين مع تحكم في الوصول والاستخدام.

### 5. API Monetization
تحقيق الدخل من APIs من خلال نماذج الاشتراك والدفع.

### 6. Data Transformation
تحويل البيانات بين صيغ مختلفة (XML ↔ JSON).

---

## أمثلة عملية

### مثال 1: إنشاء API Proxy بسيط

#### Proxy Endpoint Configuration
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ProxyEndpoint name="default">
    <Description>Default Proxy Endpoint</Description>
    <HTTPProxyConnection>
        <BasePath>/api/v1/users</BasePath>
        <VirtualHost>secure</VirtualHost>
    </HTTPProxyConnection>

    <PreFlow name="PreFlow">
        <Request>
            <Step>
                <Name>VA-VerifyAPIKey</Name>
            </Step>
            <Step>
                <Name>SA-SpikeArrest</Name>
            </Step>
        </Request>
    </PreFlow>

    <Flows>
        <Flow name="GetUser">
            <Condition>(proxy.pathsuffix MatchesPath "/{userId}") and (request.verb = "GET")</Condition>
            <Request>
                <Step>
                    <Name>RC-ResponseCache</Name>
                </Step>
            </Request>
        </Flow>

        <Flow name="CreateUser">
            <Condition>(proxy.pathsuffix MatchesPath "/") and (request.verb = "POST")</Condition>
            <Request>
                <Step>
                    <Name>AM-ValidateRequest</Name>
                </Step>
            </Request>
        </Flow>
    </Flows>

    <RouteRule name="default">
        <TargetEndpoint>default</TargetEndpoint>
    </RouteRule>
</ProxyEndpoint>
```

#### Target Endpoint Configuration
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<TargetEndpoint name="default">
    <Description>Default Target Endpoint</Description>

    <PreFlow name="PreFlow">
        <Request>
            <Step>
                <Name>AM-AddBackendHeaders</Name>
            </Step>
        </Request>
    </PreFlow>

    <PostFlow name="PostFlow">
        <Response>
            <Step>
                <Name>AM-CleanupResponse</Name>
            </Step>
        </Response>
    </PostFlow>

    <HTTPTargetConnection>
        <URL>https://backend-service.example.com</URL>
        <LoadBalancer>
            <Server name="server1"/>
            <Server name="server2"/>
        </LoadBalancer>
    </HTTPTargetConnection>
</TargetEndpoint>
```

### مثال 2: OAuth 2.0 Implementation

#### Policy للحصول على Access Token
```xml
<OAuthV2 name="OAuth-GenerateAccessToken">
    <Operation>GenerateAccessToken</Operation>
    <ExpiresIn>1800000</ExpiresIn> <!-- 30 minutes -->
    <SupportedGrantTypes>
        <GrantType>client_credentials</GrantType>
        <GrantType>password</GrantType>
    </SupportedGrantTypes>
    <GenerateResponse enabled="true"/>
    <Tokens/>
</OAuthV2>
```

#### Policy للتحقق من Access Token
```xml
<OAuthV2 name="OAuth-VerifyAccessToken">
    <Operation>VerifyAccessToken</Operation>
    <Scope>read write</Scope>
</OAuthV2>
```

### مثال 3: Rate Limiting مع Quota

```xml
<!-- Spike Arrest: 100 requests per minute -->
<SpikeArrest name="SA-RateLimit">
    <Rate>100pm</Rate>
    <Identifier ref="request.header.apikey"/>
</SpikeArrest>

<!-- Quota: 10000 requests per month -->
<Quota name="Q-MonthlyQuota">
    <Allow count="10000"/>
    <Interval>1</Interval>
    <TimeUnit>month</TimeUnit>
    <Identifier ref="request.header.apikey"/>
    <Distributed>true</Distributed>
    <Synchronous>true</Synchronous>
</Quota>
```

### مثال 4: Request/Response Transformation

#### تحويل XML Request إلى JSON
```xml
<XMLToJSON name="XML-to-JSON">
    <Source>request</Source>
    <OutputVariable>jsonRequest</OutputVariable>
    <Options>
        <NullValue>NULL</NullValue>
        <NamespaceBlockName>#namespaces</NamespaceBlockName>
        <DefaultNamespaceNodeName>$default</DefaultNamespaceNodeName>
        <NamespaceSeparator>:</NamespaceSeparator>
        <TextNodeName>value</TextNodeName>
        <AttributeBlockName>#attrs</AttributeBlockName>
        <AttributePrefix>@</AttributePrefix>
    </Options>
</XMLToJSON>

<AssignMessage name="AM-AssignJSON">
    <Set>
        <Payload contentType="application/json" variablePrefix="#" variableSuffix="#">
            {jsonRequest}
        </Payload>
    </Set>
</AssignMessage>
```

### مثال 5: Error Handling

```xml
<FaultRules>
    <FaultRule name="InvalidAPIKey">
        <Step>
            <Name>AM-InvalidAPIKeyMessage</Name>
        </Step>
        <Condition>(fault.name Matches "InvalidApiKey")</Condition>
    </FaultRule>

    <FaultRule name="QuotaExceeded">
        <Step>
            <Name>AM-QuotaExceededMessage</Name>
        </Step>
        <Condition>(fault.name = "QuotaViolation")</Condition>
    </FaultRule>
</FaultRules>
```

AssignMessage للأخطاء:
```xml
<AssignMessage name="AM-InvalidAPIKeyMessage">
    <Set>
        <StatusCode>401</StatusCode>
        <Payload contentType="application/json">
        {
            "error": {
                "code": "INVALID_API_KEY",
                "message": "Invalid API Key provided",
                "timestamp": "{system.timestamp}"
            }
        }
        </Payload>
    </Set>
</AssignMessage>
```

---

## أسئلة المقابلات

### أسئلة أساسية

**1. ما هو Apigee Edge ولماذا نستخدمه؟**
- **الإجابة**: Apigee Edge هو API Management Platform يوفر:
  - API Gateway للتوجيه والأمان
  - Traffic Management (Rate Limiting, Quota)
  - Analytics وMonitoring
  - Developer Portal
  - Monetization

**2. ما الفرق بين Proxy Endpoint و Target Endpoint؟**
- **الإجابة**:
  - **Proxy Endpoint**: النقطة التي يستقبل فيها Apigee الطلبات من العملاء
  - **Target Endpoint**: Backend Service الفعلي الذي يتم استدعاؤه

**3. ما هي الـ Policies في Apigee؟**
- **الإجابة**: Policies هي قواعد وإجراءات تطبق على الطلبات والاستجابات مثل:
  - Security (OAuth, JWT, API Keys)
  - Traffic Management (Spike Arrest, Quota)
  - Mediation (JSON-XML Conversion)
  - Caching

**4. ما الفرق بين Spike Arrest و Quota؟**
- **الإجابة**:
  - **Spike Arrest**: يحمي من الطلبات المفاجئة (short-term)، مثل 10 طلبات في الدقيقة
  - **Quota**: يحدد عدد الطلبات في فترة زمنية طويلة (long-term)، مثل 10000 طلب في الشهر

**5. كيف يعمل Request Flow في Apigee؟**
- **الإجابة**:
```
Client → ProxyEndpoint PreFlow → Conditional Flows → PostFlow
→ TargetEndpoint PreFlow → Backend → TargetEndpoint PostFlow
→ ProxyEndpoint PostFlow → Client
```

### أسئلة متقدمة

**6. كيف تنفذ OAuth 2.0 في Apigee؟**
- **الإجابة**:
  - استخدام OAuthV2 Policy
  - Operations: GenerateAccessToken, VerifyAccessToken, RefreshAccessToken
  - دعم Grant Types مختلفة (client_credentials, password, authorization_code)

**7. كيف تحسن أداء APIs باستخدام Apigee؟**
- **الإجابة**:
  - **Response Cache**: تخزين الاستجابات مؤقتاً
  - **Edge Microgateway**: لتقليل Latency
  - **Load Balancing**: توزيع الحمل على Backend Servers
  - **Compression**: ضغط البيانات

**8. ما هي Shared Flows وكيف تستخدمها؟**
- **الإجابة**: Shared Flows هي مجموعة من Policies يمكن إعادة استخدامها عبر API Proxies مختلفة:
  - تقلل التكرار
  - سهلة الصيانة
  - مثال: Security checks, Logging, Error handling

**9. كيف تتعامل مع الأخطاء في Apigee؟**
- **الإجابة**:
  - استخدام FaultRules
  - RaiseFault Policy لإنشاء أخطاء مخصصة
  - AssignMessage لتخصيص رسائل الأخطاء
  - DefaultFaultRule للأخطاء غير المعالجة

**10. ما الفرق بين API Product و API Proxy؟**
- **الإجابة**:
  - **API Proxy**: التطبيق الفعلي الذي يعالج الطلبات
  - **API Product**: مجموعة من API Proxies يتم تجميعها للمطورين مع Quota وAccess Control

**11. كيف تنفذ Service Callout في Apigee؟**
- **الإجابة**: استخدام ServiceCallout Policy لاستدعاء خدمة خارجية:
```xml
<ServiceCallout name="SC-AuthService">
    <Request>
        <Set>
            <Verb>POST</Verb>
            <Payload contentType="application/json">
                {"userId": "{userId}"}
            </Payload>
        </Set>
    </Request>
    <Response>authResponse</Response>
    <HTTPTargetConnection>
        <URL>https://auth-service.example.com/validate</URL>
    </HTTPTargetConnection>
</ServiceCallout>
```

**12. ما هي Key Value Maps (KVM) ومتى تستخدمها؟**
- **الإجابة**: KVM هي مخزن key-value لحفظ بيانات حساسة:
  - API Keys
  - Credentials
  - Configuration values
  - يمكن تشفيرها
  - متاحة على مستوى Organization, Environment, أو API Proxy

**13. كيف تنفذ Logging في Apigee؟**
- **الإجابة**:
  - MessageLogging Policy للتسجيل في Syslog أو File
  - ExtractVariables لاستخراج البيانات
  - JavaScript Policy للتسجيل المخصص

**14. ما هو API Monetization في Apigee؟**
- **الإجابة**: خاصية تتيح تحقيق الدخل من APIs:
  - Rate Plans (Fixed, Variable, Revenue Share)
  - Billing وPayment Integration
  - Usage Tracking
  - Developer Portal Integration

**15. كيف تتعامل مع CORS في Apigee؟**
- **الإجابة**: استخدام AssignMessage Policy لإضافة CORS Headers:
```xml
<AssignMessage name="AM-AddCORS">
    <Set>
        <Headers>
            <Header name="Access-Control-Allow-Origin">*</Header>
            <Header name="Access-Control-Allow-Methods">GET, POST, PUT, DELETE</Header>
            <Header name="Access-Control-Allow-Headers">Content-Type, Authorization</Header>
        </Headers>
    </Set>
</AssignMessage>
```

### أسئلة عن Best Practices

**16. ما هي Best Practices لتصميم API Proxies؟**
- **الإجابة**:
  - استخدام Shared Flows للكود المشترك
  - تطبيق Security في ProxyEndpoint PreFlow
  - استخدام Response Cache للبيانات المتكررة
  - تنفيذ Error Handling شامل
  - استخدام Flow Variables بحكمة
  - تجنب JavaScript Policies الثقيلة

**17. كيف تؤمن APIs في Apigee؟**
- **الإجابة**:
  - API Keys للتحقق الأساسي
  - OAuth 2.0 للتفويض
  - JWT للتوكينات
  - IP Whitelisting/Blacklisting
  - TLS/SSL للاتصالات الآمنة
  - Regular Expression Protection من الهجمات

**18. كيف تراقب أداء APIs في Apigee؟**
- **الإجابة**:
  - استخدام Analytics Dashboard
  - Custom Reports
  - Alerts وNotifications
  - API Monitoring في Edge UI
  - Integration مع Stackdriver (Google Cloud)

**19. ما هي Environment Variables ومتى تستخدمها؟**
- **الإجابة**: متغيرات تختلف بين البيئات:
  - Backend URLs مختلفة (Dev, Test, Prod)
  - API Keys وCredentials
  - Timeouts وConfigurations
  - تسهل Deployment عبر البيئات

**20. كيف تنفذ Versioning في Apigee؟**
- **الإجابة**:
  - استخدام BasePath مع Version: `/api/v1/users`, `/api/v2/users`
  - استخدام Header للإصدار: `Accept: application/vnd.api+json; version=2`
  - Conditional Flows بناءً على الإصدار
  - Maintain multiple API Proxies للإصدارات المختلفة

---

## الخلاصة

Apigee Edge هو منصة قوية لإدارة APIs توفر:
- ✅ API Gateway مع توجيه وتحويل متقدم
- ✅ Security شامل (OAuth, JWT, API Keys)
- ✅ Traffic Management (Rate Limiting, Quota)
- ✅ Analytics وMonitoring تفصيلي
- ✅ Developer Portal للمطورين الخارجيين
- ✅ Monetization لتحقيق الدخل من APIs
- ✅ Integration سهل مع Google Cloud

**متى تستخدم Apigee:**
- عندما تحتاج API Management كامل
- للمشاريع الكبيرة مع متطلبات معقدة
- عند الحاجة لـ Analytics متقدم
- للمشاريع على Google Cloud Platform
- عند الحاجة لـ API Monetization

**البدائل:**
- Kong (Open Source)
- AWS API Gateway
- Azure API Management
- MuleSoft Anypoint Platform