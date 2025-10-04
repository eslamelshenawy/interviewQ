# Web Services vs REST API - الدليل الشامل

## جدول المحتويات
1. [المفاهيم الأساسية](#المفاهيم-الأساسية)
2. [شجرة التفرعات والأنواع](#شجرة-التفرعات-والأنواع)
3. [SOAP Web Services](#soap-web-services)
4. [REST API](#rest-api)
5. [المقارنة الشاملة](#المقارنة-الشاملة)
6. [GraphQL](#graphql)
7. [gRPC](#grpc)
8. [أمثلة عملية](#أمثلة-عملية)
9. [متى تستخدم أيهما](#متى-تستخدم-أيهما)

---

## المفاهيم الأساسية

### ما هو Web Service؟

**Web Service** هو أي خدمة برمجية تعمل عبر الإنترنت وتسمح للتطبيقات بالتواصل مع بعضها.

```
التطبيق A  ←→  Web Service  ←→  التطبيق B
(موقع ويب)   (عبر الإنترنت)   (قاعدة بيانات)
```

**الفكرة الأساسية**:
- تطبيق يطلب بيانات أو خدمة
- Web Service يعالج الطلب
- يرد بالنتيجة

**مثال بسيط**:
- تطبيق الطقس على هاتفك → يطلب من Web Service
- Web Service → يحصل على بيانات الطقس
- يرد لتطبيقك → تظهر لك درجة الحرارة

---

### API vs Web Service

```
API (Application Programming Interface)
│
├── Web API (عبر الإنترنت)
│   │
│   ├── Web Services (بروتوكولات محددة)
│   │   ├── SOAP
│   │   ├── XML-RPC
│   │   └── WSDL-based
│   │
│   └── Web APIs (حديثة ومرنة)
│       ├── REST
│       ├── GraphQL
│       └── gRPC
│
└── Library API (داخل نفس التطبيق)
```

**الفرق المبسط**:
- **API**: أي واجهة برمجية (عامة)
- **Web Service**: API تعمل عبر الويب (خاصة)
- **كل Web Service هو API**
- **ليس كل API هو Web Service**

---

## شجرة التفرعات والأنواع

### الشجرة الكاملة

```
تقنيات التواصل بين التطبيقات
│
├── 1. Web Services (التقليدية)
│   │
│   ├── SOAP (Simple Object Access Protocol)
│   │   ├── SOAP 1.1
│   │   ├── SOAP 1.2
│   │   ├── WS-Security
│   │   ├── WS-ReliableMessaging
│   │   └── WS-Transaction
│   │
│   ├── XML-RPC
│   │   └── استدعاء إجراءات بعيدة عبر XML
│   │
│   ├── UDDI
│   │   └── دليل للبحث عن Web Services
│   │
│   └── WSDL
│       └── وصف Web Service بصيغة XML
│
├── 2. REST (Representational State Transfer)
│   │
│   ├── RESTful API (يطبق كل مبادئ REST)
│   │   ├── Stateless
│   │   ├── Client-Server
│   │   ├── Cacheable
│   │   ├── Layered System
│   │   ├── Uniform Interface
│   │   └── Code on Demand (اختياري)
│   │
│   ├── REST-like API (لا يطبق كل المبادئ)
│   │   └── معظم APIs الحديثة
│   │
│   └── HATEOAS
│       └── Hypermedia as the Engine of Application State
│
├── 3. GraphQL
│   │
│   ├── Queries (قراءة البيانات)
│   ├── Mutations (تعديل البيانات)
│   ├── Subscriptions (تحديثات فورية)
│   └── Schema Definition Language
│
├── 4. gRPC (Google Remote Procedure Call)
│   │
│   ├── Unary RPC (طلب-رد واحد)
│   ├── Server Streaming (سيل من السيرفر)
│   ├── Client Streaming (سيل من العميل)
│   ├── Bidirectional Streaming (سيل ثنائي)
│   └── Protocol Buffers (protobuf)
│
├── 5. WebSocket
│   │
│   ├── Full-Duplex Communication
│   ├── Real-time Updates
│   └── Persistent Connection
│
├── 6. Server-Sent Events (SSE)
│   │
│   └── One-way Real-time من السيرفر للعميل
│
└── 7. Webhooks
    │
    └── Event-driven HTTP Callbacks
```

---

## SOAP Web Services

### ما هو SOAP؟

**SOAP** = Simple Object Access Protocol
- بروتوكول معقد للتواصل
- يعتمد على XML بالكامل
- ظهر في أواخر التسعينات
- مازال مستخدم في الأنظمة القديمة والمصرفية

### البنية الأساسية

```xml
SOAP Message Structure
│
├── Envelope (المظروف) - العنصر الجذر
│   │
│   ├── Header (الرأس) - اختياري
│   │   ├── Security (الأمان)
│   │   ├── Transaction (المعاملات)
│   │   └── Routing (التوجيه)
│   │
│   └── Body (الجسم) - إجباري
│       ├── Request (الطلب)
│       ├── Response (الرد)
│       └── Fault (الخطأ)
```

### مثال SOAP كامل

#### الطلب (Request)

```xml
POST /WebServices/CustomerService.asmx HTTP/1.1
Host: www.example.com
Content-Type: text/xml; charset=utf-8
Content-Length: xxx
SOAPAction: "http://tempuri.org/GetCustomerDetails"

<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope
    xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema">

    <soap:Header>
        <Authentication xmlns="http://tempuri.org/">
            <Username>admin</Username>
            <Password>pass123</Password>
        </Authentication>
    </soap:Header>

    <soap:Body>
        <GetCustomerDetails xmlns="http://tempuri.org/">
            <CustomerId>12345</CustomerId>
        </GetCustomerDetails>
    </soap:Body>

</soap:Envelope>
```

#### الرد (Response)

```xml
<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope
    xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">

    <soap:Body>
        <GetCustomerDetailsResponse xmlns="http://tempuri.org/">
            <GetCustomerDetailsResult>
                <Customer>
                    <CustomerId>12345</CustomerId>
                    <Name>أحمد محمد</Name>
                    <Email>ahmed@example.com</Email>
                    <Phone>+966501234567</Phone>
                    <Balance>5000.00</Balance>
                    <Status>Active</Status>
                </Customer>
            </GetCustomerDetailsResult>
        </GetCustomerDetailsResponse>
    </soap:Body>

</soap:Envelope>
```

#### الخطأ (Fault)

```xml
<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <soap:Fault>
            <faultcode>soap:Client</faultcode>
            <faultstring>Customer Not Found</faultstring>
            <detail>
                <ErrorCode>404</ErrorCode>
                <Message>العميل برقم 12345 غير موجود</Message>
            </detail>
        </soap:Fault>
    </soap:Body>
</soap:Envelope>
```

---

### WSDL (Web Services Description Language)

**WSDL** = ملف XML يصف Web Service

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions name="CustomerService"
    targetNamespace="http://tempuri.org/"
    xmlns="http://schemas.xmlsoap.org/wsdl/"
    xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/">

    <!-- التعريفات (Types) -->
    <types>
        <xsd:schema targetNamespace="http://tempuri.org/">
            <xsd:element name="GetCustomerDetails">
                <xsd:complexType>
                    <xsd:sequence>
                        <xsd:element name="CustomerId" type="xsd:int"/>
                    </xsd:sequence>
                </xsd:complexType>
            </xsd:element>
        </xsd:schema>
    </types>

    <!-- الرسائل (Messages) -->
    <message name="GetCustomerDetailsRequest">
        <part name="parameters" element="tns:GetCustomerDetails"/>
    </message>

    <message name="GetCustomerDetailsResponse">
        <part name="parameters" element="tns:Customer"/>
    </message>

    <!-- العمليات (Port Type) -->
    <portType name="CustomerServicePortType">
        <operation name="GetCustomerDetails">
            <input message="tns:GetCustomerDetailsRequest"/>
            <output message="tns:GetCustomerDetailsResponse"/>
        </operation>
    </portType>

    <!-- الربط (Binding) -->
    <binding name="CustomerServiceBinding" type="tns:CustomerServicePortType">
        <soap:binding style="document"
            transport="http://schemas.xmlsoap.org/soap/http"/>
        <operation name="GetCustomerDetails">
            <soap:operation
                soapAction="http://tempuri.org/GetCustomerDetails"/>
            <input>
                <soap:body use="literal"/>
            </input>
            <output>
                <soap:body use="literal"/>
            </output>
        </operation>
    </binding>

    <!-- الخدمة (Service) -->
    <service name="CustomerService">
        <port name="CustomerServicePort"
            binding="tns:CustomerServiceBinding">
            <soap:address
                location="http://www.example.com/CustomerService"/>
        </port>
    </service>

</definitions>
```

---

### مميزات SOAP

✅ **المميزات**:
1. **الأمان المدمج** (WS-Security)
   - تشفير
   - توقيع رقمي
   - مصادقة

2. **المعاملات** (ACID Transactions)
   - دعم معاملات معقدة
   - Rollback تلقائي

3. **الموثوقية** (WS-ReliableMessaging)
   - ضمان وصول الرسائل
   - إعادة إرسال تلقائية

4. **التوحيد الصارم**
   - معايير واضحة
   - WSDL للتوثيق

5. **دعم بروتوكولات متعددة**
   - HTTP, HTTPS
   - SMTP
   - TCP, JMS

❌ **العيوب**:
1. **معقد جداً**
   - صعب الفهم
   - صعب التطوير

2. **بطيء**
   - XML كبير الحجم
   - معالجة بطيئة

3. **XML فقط**
   - لا يدعم JSON
   - ثقيل

4. **صعوبة الاختبار**
   - يحتاج أدوات خاصة (SoapUI)

---

## REST API

### ما هو REST؟

**REST** = Representational State Transfer
- أسلوب معماري (ليس بروتوكول)
- يستخدم HTTP بطريقة بسيطة ومباشرة
- اقترحه Roy Fielding سنة 2000
- السائد اليوم في معظم التطبيقات

### المبادئ الستة لـ REST

```
REST Principles
│
├── 1. Client-Server Architecture
│   └── فصل العميل عن السيرفر
│
├── 2. Stateless
│   └── كل طلب مستقل (لا يحفظ حالة)
│
├── 3. Cacheable
│   └── يمكن حفظ الردود مؤقتاً
│
├── 4. Uniform Interface
│   └── واجهة موحدة للتعامل مع الموارد
│
├── 5. Layered System
│   └── نظام طبقات (proxy, load balancer)
│
└── 6. Code on Demand (اختياري)
    └── إرسال كود قابل للتنفيذ (JavaScript)
```

---

### HTTP Methods (الأفعال)

```
HTTP Methods في REST
│
├── GET - قراءة (Read)
│   ├── آمن (Safe) - لا يعدل البيانات
│   ├── Idempotent - نفس النتيجة دائماً
│   └── مثال: GET /api/users/123
│
├── POST - إنشاء (Create)
│   ├── غير آمن - يعدل البيانات
│   ├── Non-Idempotent - كل طلب يُنشئ مورد جديد
│   └── مثال: POST /api/users
│
├── PUT - تحديث كامل (Update/Replace)
│   ├── غير آمن
│   ├── Idempotent - نفس النتيجة
│   └── مثال: PUT /api/users/123
│
├── PATCH - تحديث جزئي (Partial Update)
│   ├── غير آمن
│   ├── Non-Idempotent عادةً
│   └── مثال: PATCH /api/users/123
│
├── DELETE - حذف (Delete)
│   ├── غير آمن
│   ├── Idempotent
│   └── مثال: DELETE /api/users/123
│
├── HEAD - معلومات الرأس فقط
│   └── مثل GET لكن بدون Body
│
├── OPTIONS - الأفعال المتاحة
│   └── OPTIONS /api/users
│
└── TRACE - تتبع الطلب (نادر)
```

---

### HTTP Status Codes

```
HTTP Status Codes
│
├── 1xx - معلوماتي (Informational)
│   ├── 100 Continue
│   └── 101 Switching Protocols
│
├── 2xx - نجاح (Success)
│   ├── 200 OK - نجح الطلب
│   ├── 201 Created - تم الإنشاء
│   ├── 202 Accepted - تم القبول (معالجة لاحقة)
│   ├── 204 No Content - نجح بدون محتوى
│   └── 206 Partial Content - محتوى جزئي
│
├── 3xx - إعادة توجيه (Redirection)
│   ├── 301 Moved Permanently - نقل دائم
│   ├── 302 Found - نقل مؤقت
│   ├── 304 Not Modified - لم يتغير (cache)
│   └── 307 Temporary Redirect
│
├── 4xx - خطأ العميل (Client Error)
│   ├── 400 Bad Request - طلب خاطئ
│   ├── 401 Unauthorized - غير مصرح
│   ├── 403 Forbidden - ممنوع
│   ├── 404 Not Found - غير موجود
│   ├── 405 Method Not Allowed - الفعل غير مسموح
│   ├── 409 Conflict - تعارض
│   ├── 422 Unprocessable Entity - بيانات خاطئة
│   └── 429 Too Many Requests - طلبات كثيرة
│
└── 5xx - خطأ السيرفر (Server Error)
    ├── 500 Internal Server Error - خطأ داخلي
    ├── 501 Not Implemented - غير مطبق
    ├── 502 Bad Gateway - بوابة سيئة
    ├── 503 Service Unavailable - الخدمة غير متاحة
    └── 504 Gateway Timeout - انتهت مهلة البوابة
```

---

### أمثلة REST API كاملة

#### مثال 1: الحصول على مستخدم (GET)

**الطلب**:
```http
GET /api/v1/users/12345 HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**الرد**:
```http
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: max-age=3600
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"

{
  "id": 12345,
  "username": "ahmed_mohamed",
  "email": "ahmed@example.com",
  "firstName": "أحمد",
  "lastName": "محمد",
  "avatar": "https://cdn.example.com/avatars/12345.jpg",
  "createdAt": "2024-01-15T10:30:00Z",
  "updatedAt": "2024-03-20T14:22:00Z",
  "role": "customer",
  "status": "active",
  "_links": {
    "self": "/api/v1/users/12345",
    "orders": "/api/v1/users/12345/orders",
    "profile": "/api/v1/users/12345/profile"
  }
}
```

---

#### مثال 2: إنشاء مستخدم (POST)

**الطلب**:
```http
POST /api/v1/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

{
  "username": "sara_ali",
  "email": "sara@example.com",
  "password": "SecurePass123!",
  "firstName": "سارة",
  "lastName": "علي",
  "phone": "+966501234567"
}
```

**الرد**:
```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/v1/users/12346

{
  "id": 12346,
  "username": "sara_ali",
  "email": "sara@example.com",
  "firstName": "سارة",
  "lastName": "علي",
  "phone": "+966501234567",
  "createdAt": "2025-01-04T15:30:00Z",
  "status": "active",
  "message": "تم إنشاء الحساب بنجاح"
}
```

---

#### مثال 3: تحديث مستخدم (PUT)

**الطلب**:
```http
PUT /api/v1/users/12345 HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

{
  "username": "ahmed_mohamed",
  "email": "ahmed.new@example.com",
  "firstName": "أحمد",
  "lastName": "محمد علي",
  "phone": "+966509876543",
  "avatar": "https://cdn.example.com/avatars/new-avatar.jpg"
}
```

**الرد**:
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 12345,
  "username": "ahmed_mohamed",
  "email": "ahmed.new@example.com",
  "firstName": "أحمد",
  "lastName": "محمد علي",
  "phone": "+966509876543",
  "avatar": "https://cdn.example.com/avatars/new-avatar.jpg",
  "updatedAt": "2025-01-04T16:00:00Z",
  "message": "تم تحديث البيانات بنجاح"
}
```

---

#### مثال 4: تحديث جزئي (PATCH)

**الطلب**:
```http
PATCH /api/v1/users/12345 HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

{
  "phone": "+966501111111"
}
```

**الرد**:
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 12345,
  "phone": "+966501111111",
  "updatedAt": "2025-01-04T16:15:00Z",
  "message": "تم تحديث رقم الهاتف"
}
```

---

#### مثال 5: حذف مستخدم (DELETE)

**الطلب**:
```http
DELETE /api/v1/users/12345 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**الرد**:
```http
HTTP/1.1 204 No Content
```

أو:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "message": "تم حذف الحساب بنجاح",
  "deletedAt": "2025-01-04T16:30:00Z"
}
```

---

#### مثال 6: قائمة مع Pagination

**الطلب**:
```http
GET /api/v1/users?page=2&limit=20&sort=createdAt&order=desc&status=active HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**الرد**:
```http
HTTP/1.1 200 OK
Content-Type: application/json
X-Total-Count: 150
X-Page: 2
X-Per-Page: 20
Link: </api/v1/users?page=1&limit=20>; rel="first",
      </api/v1/users?page=1&limit=20>; rel="prev",
      </api/v1/users?page=3&limit=20>; rel="next",
      </api/v1/users?page=8&limit=20>; rel="last"

{
  "data": [
    {
      "id": 12345,
      "username": "ahmed_mohamed",
      "email": "ahmed@example.com",
      "status": "active"
    },
    {
      "id": 12346,
      "username": "sara_ali",
      "email": "sara@example.com",
      "status": "active"
    }
    // ... 18 more users
  ],
  "pagination": {
    "total": 150,
    "count": 20,
    "per_page": 20,
    "current_page": 2,
    "total_pages": 8,
    "links": {
      "first": "/api/v1/users?page=1&limit=20",
      "prev": "/api/v1/users?page=1&limit=20",
      "next": "/api/v1/users?page=3&limit=20",
      "last": "/api/v1/users?page=8&limit=20"
    }
  }
}
```

---

#### مثال 7: معالجة الأخطاء

**الطلب** (بيانات خاطئة):
```http
POST /api/v1/users HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "username": "ab",
  "email": "not-an-email",
  "password": "123"
}
```

**الرد**:
```http
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/json

{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "فشل التحقق من البيانات",
    "details": [
      {
        "field": "username",
        "message": "اسم المستخدم يجب أن يكون 3 أحرف على الأقل",
        "code": "MIN_LENGTH"
      },
      {
        "field": "email",
        "message": "البريد الإلكتروني غير صحيح",
        "code": "INVALID_EMAIL"
      },
      {
        "field": "password",
        "message": "كلمة المرور ضعيفة جداً",
        "code": "WEAK_PASSWORD"
      }
    ],
    "timestamp": "2025-01-04T16:45:00Z",
    "path": "/api/v1/users"
  }
}
```

---

### تصميم REST API - Best Practices

#### 1. تسمية الموارد (Resources)

```
✅ جيد:
GET    /api/users              - قائمة المستخدمين
GET    /api/users/123          - مستخدم محدد
POST   /api/users              - إنشاء مستخدم
PUT    /api/users/123          - تحديث مستخدم
DELETE /api/users/123          - حذف مستخدم

GET    /api/users/123/orders   - طلبات المستخدم
GET    /api/orders/456         - طلب محدد

❌ سيء:
GET    /api/getUsers
POST   /api/createUser
GET    /api/user_list
DELETE /api/deleteUserById?id=123
```

---

#### 2. التفرعات (Nested Resources)

```
✅ جيد (حتى مستويين):
/api/users/123/orders
/api/users/123/orders/456
/api/users/123/addresses

❌ سيء (تداخل عميق):
/api/users/123/orders/456/items/789/reviews
(استخدم: /api/order-items/789/reviews)
```

---

#### 3. الفلترة والترتيب والبحث

```
✅ أمثلة جيدة:

الفلترة:
GET /api/users?status=active
GET /api/users?role=admin&status=active

الترتيب:
GET /api/users?sort=createdAt&order=desc
GET /api/users?sort=-createdAt (- للترتيب التنازلي)

البحث:
GET /api/users?search=ahmed
GET /api/users?q=ahmed

الحقول المحددة:
GET /api/users?fields=id,username,email

Pagination:
GET /api/users?page=2&limit=20
GET /api/users?offset=20&limit=20
```

---

#### 4. Versioning (الإصدارات)

```
الطريقة 1: في الـ URL (الأكثر شيوعاً)
✅ /api/v1/users
✅ /api/v2/users

الطريقة 2: في الـ Header
✅ Accept: application/vnd.example.v1+json

الطريقة 3: Query Parameter
❌ /api/users?version=1 (غير مفضل)
```

---

#### 5. معالجة الأخطاء - Error Handling

```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "المستخدم غير موجود",
    "details": "لا يوجد مستخدم بالمعرف 12345",
    "timestamp": "2025-01-04T17:00:00Z",
    "path": "/api/v1/users/12345",
    "requestId": "abc-123-def-456"
  }
}
```

---

### مميزات REST

✅ **المميزات**:
1. **بسيط وسهل**
   - سهل الفهم
   - سهل التطوير

2. **خفيف وسريع**
   - JSON صغير الحجم
   - أداء عالي

3. **مرن**
   - يدعم JSON, XML, HTML, Plain Text
   - لا يفرض بروتوكول معين

4. **قابل للتخزين المؤقت (Cacheable)**
   - تحسين الأداء
   - تقليل الحمل

5. **سهل الاختبار**
   - Postman, Insomnia
   - cURL, HTTPie
   - أدوات بسيطة

6. **واسع الانتشار**
   - مدعوم في كل اللغات
   - مكتبات كثيرة

❌ **العيوب**:
1. **Over-fetching / Under-fetching**
   - قد تحصل على بيانات أكثر من اللازم
   - أو أقل من المطلوب

2. **عدد كبير من الطلبات**
   - لجلب بيانات مترابطة
   - N+1 problem

3. **لا يوجد معيار صارم**
   - كل API مختلف
   - صعوبة التوحيد

4. **Stateless**
   - صعوبة في بعض الحالات
   - حاجة لإدارة الحالة من العميل

---

## GraphQL

### ما هو GraphQL؟

**GraphQL** = Graph Query Language
- لغة استعلام للـ APIs
- طورتها Facebook سنة 2012
- بديل لـ REST
- تحل مشاكل Over-fetching و Under-fetching

### الفكرة الأساسية

```
REST:
العميل: أعطني المستخدم
السيرفر: تفضل كل بيانات المستخدم (100 حقل)
العميل: أنا أريد الاسم فقط! 😔

GraphQL:
العميل: أعطني المستخدم (الاسم والبريد فقط)
السيرفر: تفضل الاسم والبريد 😊
```

---

### مثال GraphQL

#### Schema (المخطط)

```graphql
type User {
  id: ID!
  username: String!
  email: String!
  firstName: String
  lastName: String
  posts: [Post!]!
  createdAt: DateTime!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  comments: [Comment!]!
  createdAt: DateTime!
}

type Comment {
  id: ID!
  text: String!
  author: User!
  createdAt: DateTime!
}

type Query {
  user(id: ID!): User
  users(limit: Int, offset: Int): [User!]!
  post(id: ID!): Post
  posts: [Post!]!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
  createPost(input: CreatePostInput!): Post!
}

type Subscription {
  postCreated: Post!
  commentAdded(postId: ID!): Comment!
}
```

---

#### Query (الاستعلام)

**الطلب**:
```graphql
query {
  user(id: "12345") {
    id
    username
    email
    posts {
      id
      title
      comments {
        id
        text
        author {
          username
        }
      }
    }
  }
}
```

**الرد**:
```json
{
  "data": {
    "user": {
      "id": "12345",
      "username": "ahmed_mohamed",
      "email": "ahmed@example.com",
      "posts": [
        {
          "id": "1",
          "title": "مقال عن GraphQL",
          "comments": [
            {
              "id": "10",
              "text": "مقال رائع!",
              "author": {
                "username": "sara_ali"
              }
            }
          ]
        }
      ]
    }
  }
}
```

---

#### Mutation (التعديل)

**الطلب**:
```graphql
mutation {
  createUser(input: {
    username: "sara_ali"
    email: "sara@example.com"
    password: "SecurePass123!"
    firstName: "سارة"
    lastName: "علي"
  }) {
    id
    username
    email
    createdAt
  }
}
```

**الرد**:
```json
{
  "data": {
    "createUser": {
      "id": "12346",
      "username": "sara_ali",
      "email": "sara@example.com",
      "createdAt": "2025-01-04T17:30:00Z"
    }
  }
}
```

---

#### Subscription (الاشتراك)

**الطلب**:
```graphql
subscription {
  postCreated {
    id
    title
    author {
      username
    }
  }
}
```

**الرد** (في الوقت الفعلي):
```json
{
  "data": {
    "postCreated": {
      "id": "999",
      "title": "مقال جديد للتو",
      "author": {
        "username": "omar_khaled"
      }
    }
  }
}
```

---

### مميزات GraphQL

✅ **المميزات**:
1. **طلب واحد فقط**
   - بيانات مترابطة في طلب واحد
   - لا N+1 problem

2. **البيانات المطلوبة فقط**
   - لا Over-fetching
   - لا Under-fetching

3. **Strongly Typed**
   - Schema واضح
   - Type safety

4. **تحديثات فورية**
   - Subscriptions
   - Real-time

5. **Introspection**
   - استكشاف API تلقائياً
   - توثيق ذاتي

❌ **العيوب**:
1. **معقد للبداية**
   - منحنى تعلم أعلى

2. **صعوبة الـ Caching**
   - لا يستفيد من HTTP cache

3. **Over-querying**
   - استعلامات معقدة قد تبطئ السيرفر

4. **File Upload معقد**

---

## gRPC

### ما هو gRPC؟

**gRPC** = Google Remote Procedure Call
- تقنية من Google
- تستخدم Protocol Buffers (protobuf)
- سريعة جداً
- مناسبة للـ Microservices

### Protocol Buffers

```protobuf
syntax = "proto3";

package user;

service UserService {
  rpc GetUser (GetUserRequest) returns (User);
  rpc CreateUser (CreateUserRequest) returns (User);
  rpc ListUsers (ListUsersRequest) returns (ListUsersResponse);
  rpc UpdateUser (UpdateUserRequest) returns (User);
  rpc DeleteUser (DeleteUserRequest) returns (DeleteUserResponse);
}

message User {
  int32 id = 1;
  string username = 2;
  string email = 3;
  string first_name = 4;
  string last_name = 5;
  int64 created_at = 6;
}

message GetUserRequest {
  int32 id = 1;
}

message CreateUserRequest {
  string username = 1;
  string email = 2;
  string password = 3;
  string first_name = 4;
  string last_name = 5;
}

message ListUsersRequest {
  int32 page = 1;
  int32 limit = 2;
}

message ListUsersResponse {
  repeated User users = 1;
  int32 total = 2;
}

message UpdateUserRequest {
  int32 id = 1;
  string email = 2;
  string first_name = 3;
  string last_name = 4;
}

message DeleteUserRequest {
  int32 id = 1;
}

message DeleteUserResponse {
  bool success = 1;
}
```

---

### أنواع gRPC

```
gRPC Communication Patterns
│
├── 1. Unary RPC (طلب-رد واحد)
│   Client → Request → Server
│   Client ← Response ← Server
│
├── 2. Server Streaming (سيل من السيرفر)
│   Client → Request → Server
│   Client ← Response 1 ← Server
│   Client ← Response 2 ← Server
│   Client ← Response N ← Server
│
├── 3. Client Streaming (سيل من العميل)
│   Client → Request 1 → Server
│   Client → Request 2 → Server
│   Client → Request N → Server
│   Client ← Response ← Server
│
└── 4. Bidirectional Streaming (سيل ثنائي)
    Client ↔ Request/Response ↔ Server
    (تبادل مستمر في الاتجاهين)
```

---

### مميزات gRPC

✅ **المميزات**:
1. **سرعة فائقة**
   - Binary Protocol (protobuf)
   - أسرع من JSON بكثير

2. **حجم صغير**
   - البيانات مضغوطة

3. **Streaming**
   - دعم 4 أنواع من الـ Streaming

4. **Strongly Typed**
   - Type safety
   - Code generation

5. **HTTP/2**
   - Multiplexing
   - Server Push

❌ **العيوب**:
1. **غير مفهوم للبشر**
   - Binary format
   - صعب Debug

2. **محدود في المتصفحات**
   - يحتاج gRPC-Web

3. **أقل شيوعاً**
   - أدوات أقل

---

## المقارنة الشاملة

### جدول المقارنة الكامل

| المعيار | SOAP | REST | GraphQL | gRPC |
|---------|------|------|---------|------|
| **البروتوكول** | SOAP | HTTP | HTTP | HTTP/2 |
| **صيغة البيانات** | XML | JSON, XML, etc | JSON | Protobuf (Binary) |
| **الأفعال** | POST فقط | GET, POST, PUT, DELETE | Query, Mutation | RPC Methods |
| **السرعة** | بطيء | سريع | متوسط | سريع جداً |
| **حجم الرسالة** | كبير | متوسط | متوسط | صغير جداً |
| **سهولة التعلم** | صعب | سهل | متوسط | صعب |
| **التوثيق** | WSDL | Swagger/OpenAPI | Schema | Protobuf |
| **الأمان** | WS-Security | HTTPS, OAuth | HTTPS, OAuth | TLS, Auth |
| **Caching** | صعب | سهل | صعب | صعب |
| **Browser Support** | نعم | نعم | نعم | محدود |
| **Streaming** | لا | لا (SSE منفصل) | نعم (Subscriptions) | نعم (4 أنواع) |
| **Type Safety** | نعم | لا | نعم | نعم |
| **الاستخدام** | Legacy Systems | معظم APIs | Facebook, GitHub | Google, Microservices |

---

### مقارنة الأداء

```
حجم الرسالة (نفس البيانات):

SOAP (XML):     1200 bytes  ████████████
REST (JSON):     500 bytes  █████
GraphQL (JSON):  450 bytes  ████▌
gRPC (Protobuf): 150 bytes  █▌

السرعة (معالجة 1000 طلب):

SOAP:     850ms  ████████▌
REST:     320ms  ███▏
GraphQL:  380ms  ███▊
gRPC:     120ms  █▏
```

---

### مقارنة الأمثلة

#### نفس الطلب في كل التقنيات

**الهدف**: الحصول على اسم وبريد المستخدم 12345

---

**SOAP**:
```xml
<!-- الطلب -->
<?xml version="1.0"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <GetUser xmlns="http://tempuri.org/">
      <UserId>12345</UserId>
      <Fields>
        <Field>Name</Field>
        <Field>Email</Field>
      </Fields>
    </GetUser>
  </soap:Body>
</soap:Envelope>

<!-- الرد -->
<?xml version="1.0"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <GetUserResponse>
      <User>
        <Name>أحمد محمد</Name>
        <Email>ahmed@example.com</Email>
      </User>
    </GetUserResponse>
  </soap:Body>
</soap:Envelope>

الحجم: ~450 bytes
```

---

**REST**:
```http
<!-- الطلب -->
GET /api/users/12345?fields=name,email HTTP/1.1
Host: api.example.com

<!-- الرد -->
{
  "name": "أحمد محمد",
  "email": "ahmed@example.com"
}

الحجم: ~80 bytes
```

---

**GraphQL**:
```graphql
# الطلب
query {
  user(id: "12345") {
    name
    email
  }
}

# الرد
{
  "data": {
    "user": {
      "name": "أحمد محمد",
      "email": "ahmed@example.com"
    }
  }
}

الحجم: ~100 bytes
```

---

**gRPC**:
```protobuf
// الطلب (Binary)
GetUserRequest { id: 12345 }

// الرد (Binary)
User {
  name: "أحمد محمد"
  email: "ahmed@example.com"
}

الحجم: ~35 bytes (Binary)
```

---

## متى تستخدم أيهما؟

### شجرة القرار

```
                    [ابدأ هنا]
                         │
                         ↓
          ⬦ هل تتعامل مع نظام قديم/مصرفي؟
            ↙                              ↘
          نعم                              لا
           ↓                                ↓
        [SOAP]                    ⬦ هل تحتاج أداء عالي جداً؟
      (مضطر)                       ↙                    ↘
                                 نعم                    لا
                                  ↓                      ↓
                               [gRPC]          ⬦ هل العميل متصفح ويب؟
                          (Microservices)       ↙                ↘
                                             نعم                  لا
                                              ↓                    ↓
                                    ⬦ بيانات معقدة        [gRPC]
                                    ومترابطة؟          (Internal)
                                    ↙        ↘
                                  نعم        لا
                                   ↓          ↓
                              [GraphQL]    [REST]
                            (Facebook)   (الافتراضي)
```

---

### الحالات التفصيلية

#### استخدم SOAP إذا:
```
✅ تتعامل مع:
   - أنظمة بنوك قديمة
   - حكومات
   - شركات كبيرة قديمة
   - أنظمة تتطلب WS-Security

❌ لا تستخدمه في:
   - تطبيقات جديدة
   - موبايل
   - تطبيقات ويب حديثة
```

---

#### استخدم REST إذا:
```
✅ مناسب لـ:
   - معظم التطبيقات (الافتراضي)
   - تطبيقات ويب
   - تطبيقات موبايل
   - Public APIs
   - CRUD بسيط

✅ أمثلة:
   - متجر إلكتروني
   - مدونة
   - نظام إدارة
   - API عامة

❌ لا تستخدمه إذا:
   - Over-fetching مشكلة كبيرة
   - تحتاج Real-time
   - علاقات معقدة بين البيانات
```

---

#### استخدم GraphQL إذا:
```
✅ مناسب لـ:
   - بيانات معقدة ومترابطة
   - تطبيقات موبايل (توفير Data)
   - عملاء متعددين بحاجات مختلفة
   - Real-time (Subscriptions)

✅ أمثلة:
   - Facebook, Instagram
   - GitHub
   - Shopify
   - تطبيق اجتماعي

❌ لا تستخدمه إذا:
   - البيانات بسيطة
   - لا خبرة في GraphQL
   - Caching مهم جداً
```

---

#### استخدم gRPC إذا:
```
✅ مناسب لـ:
   - Microservices (Internal)
   - أداء عالي جداً مطلوب
   - Real-time Streaming
   - اتصالات Server-to-Server

✅ أمثلة:
   - Google (YouTube, Maps)
   - Netflix
   - Square
   - Dropbox

❌ لا تستخدمه إذا:
   - العميل متصفح ويب
   - تحتاج Human-readable
   - Public API
```

---

## أمثلة عملية حسب المشروع

### مشروع 1: متجر إلكتروني

```
التوصية: REST API

السبب:
✅ CRUD بسيط (منتجات، طلبات، عملاء)
✅ سهل التطوير
✅ الفريق يعرفه
✅ أدوات كثيرة
✅ التخزين المؤقت مهم

Endpoints:
GET    /api/products
GET    /api/products/123
POST   /api/orders
GET    /api/users/profile
```

---

### مشروع 2: تطبيق تواصل اجتماعي

```
التوصية: GraphQL + WebSocket

السبب:
✅ بيانات معقدة (Users, Posts, Comments, Likes)
✅ كل عميل يحتاج بيانات مختلفة
✅ Real-time (notifications, messages)
✅ تقليل عدد الطلبات

مثال:
query {
  feed {
    posts {
      author { name, avatar }
      content
      likes { count }
      comments(limit: 3) {
        author { name }
        text
      }
    }
  }
}
```

---

### مشروع 3: نظام Microservices داخلي

```
التوصية: gRPC

السبب:
✅ اتصالات Server-to-Server
✅ سرعة عالية جداً
✅ Type safety
✅ Streaming

Services:
UserService  ←gRPC→  OrderService
                ↕
            PaymentService
                ↕
          NotificationService
```

---

### مشروع 4: تكامل مع بنك

```
التوصية: SOAP (مضطر)

السبب:
❗ البنك يستخدم SOAP فقط
❗ لا خيار آخر
✅ WS-Security مدمج

WSDL:
https://bank.example.com/services/payment?wsdl

Operations:
- TransferMoney
- CheckBalance
- GetTransactionHistory
```

---

## الخلاصة النهائية

### الترتيب من حيث الشيوعاء في 2025

```
1. REST API          ████████████████████ 80%
   (الأكثر استخداماً)

2. GraphQL           ████████ 12%
   (في نمو)

3. gRPC              ████ 6%
   (Microservices)

4. SOAP              ██ 2%
   (Legacy فقط)
```

---

### نصائح أخيرة

#### للمبتدئين:
```
1. ابدأ بـ REST
2. تعلم HTTP جيداً
3. استخدم Postman
4. اقرأ عن RESTful Best Practices
```

#### للمتقدمين:
```
1. جرّب GraphQL في مشروع جانبي
2. اقرأ عن gRPC للـ Microservices
3. افهم متى تستخدم أي تقنية
4. لا تستخدم تقنية لأنها "رائجة"
```

---

### الجدول المختصر

| التقنية | الاستخدام | الشيوعاء |
|---------|-----------|---------|
| **REST** | معظم التطبيقات | ⭐⭐⭐⭐⭐ |
| **GraphQL** | بيانات معقدة | ⭐⭐⭐ |
| **gRPC** | Microservices | ⭐⭐ |
| **SOAP** | Legacy فقط | ⭐ |

---

**في الشك، استخدم REST!**

معظم المشاريع لا تحتاج GraphQL أو gRPC. ابدأ بسيط مع REST، وإذا واجهت مشاكل محددة، فكر في البدائل.

---

**تم إنشاء هذا الدليل كمرجع شامل للمقارنة بين Web Services و REST API والتقنيات الحديثة**
