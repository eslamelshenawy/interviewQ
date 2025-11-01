# Adapter Pattern - نمط المحول

## المحتويات
1. [ما هو Adapter Pattern](#ما-هو-adapter-pattern)
2. [المشكلة](#المشكلة)
3. [الحل](#الحل)
4. [أنواع Adapter](#أنواع-adapter)
5. [أمثلة عملية](#أمثلة-عملية)
6. [المزايا والعيوب](#المزايا-والعيوب)
7. [متى تستخدم](#متى-تستخدم)
8. [أسئلة المقابلات](#أسئلة-المقابلات)

---

## ما هو Adapter Pattern / What is Adapter Pattern

### العربية

**Adapter Pattern** هو نمط تصميم هيكلي (Structural Design Pattern) يسمح لكائنات ذات واجهات غير متوافقة (Incompatible Interfaces) بالعمل معاً. يعمل كجسر (Bridge) بين واجهتين مختلفتين.

**الفكرة الأساسية:** مثل محول الكهرباء الذي يربط قابس أمريكي بمقبس أوروبي، Adapter يربط بين واجهات البرمجيات غير المتوافقة.

### English

**Adapter Pattern** is a structural design pattern that allows objects with incompatible interfaces to work together. It acts as a bridge between two different interfaces.

**Core Idea:** Like an electrical adapter that connects an American plug to a European socket, Adapter connects incompatible software interfaces.

---

## المشكلة / The Problem

### العربية

تخيل أن لديك نظام قديم (Legacy System) وتريد دمجه مع نظام جديد:

```javascript
// ❌ مشكلة: واجهات غير متوافقة

// النظام القديم
class OldPaymentSystem {
  makePayment(amount, accountNumber) {
    console.log(`Old system: Paying ${amount} from account ${accountNumber}`);
    return { status: 'success', id: 123 };
  }
}

// النظام الجديد يتوقع واجهة مختلفة
class NewPaymentProcessor {
  process(paymentData) {
    // يتوقع: { amount, cardNumber, cvv }
    // ولكن النظام القديم يستخدم accountNumber
  }
}

// لا يمكنهما العمل معاً مباشرة!
```

**المشاكل:**
- واجهات غير متوافقة
- لا يمكن تعديل الكود القديم
- تكرار الكود للتعامل مع الحالتين

### English

Imagine you have a legacy system and want to integrate it with a new system:

```javascript
// ❌ Problem: Incompatible interfaces
// (Same code as above - code is universal)
```

**Problems:**
- Incompatible interfaces
- Cannot modify legacy code
- Code duplication to handle both cases

---

## الحل / The Solution

### العربية

استخدم Adapter لتحويل الواجهة:

```javascript
// ✅ الحل: Adapter Pattern

// النظام القديم
class OldPaymentSystem {
  makePayment(amount, accountNumber) {
    console.log(`Old system: Paying ${amount} from account ${accountNumber}`);
    return { status: 'success', id: 123 };
  }
}

// الواجهة المطلوبة
interface PaymentProcessor {
  processPayment(paymentData) { }
}

// Adapter يحول واجهة النظام القديم للواجهة الجديدة
class PaymentAdapter {
  constructor(oldSystem) {
    this.oldSystem = oldSystem;
  }

  processPayment(paymentData) {
    // تحويل البيانات من الشكل الجديد للقديم
    const { amount, accountNumber } = paymentData;
    return this.oldSystem.makePayment(amount, accountNumber);
  }
}

// الاستخدام
const oldSystem = new OldPaymentSystem();
const adapter = new PaymentAdapter(oldSystem);

// الآن يمكن استخدامه مع الواجهة الجديدة
adapter.processPayment({
  amount: 100,
  accountNumber: '123456'
});
```

### English

Use Adapter to convert the interface:

```javascript
// ✅ Solution: Adapter Pattern
// (Same code as above - code is universal)
```

---

## أنواع Adapter / Types of Adapter

### 1. Object Adapter (التركيب - Composition)

```javascript
// يستخدم Composition
class ObjectAdapter {
  constructor(adaptee) {
    this.adaptee = adaptee;
  }

  request() {
    return this.adaptee.specificRequest();
  }
}
```

### 2. Class Adapter (الوراثة - Inheritance)

```javascript
// يستخدم Inheritance
class ClassAdapter extends Adaptee {
  request() {
    return this.specificRequest();
  }
}
```

---

## أمثلة عملية / Practical Examples

### مثال 1: XML to JSON Adapter

```javascript
// العربية: محول من XML إلى JSON

// خدمة قديمة ترجع XML
class XMLDataService {
  getXMLData() {
    return `
      <user>
        <name>Ahmed</name>
        <email>ahmed@example.com</email>
        <age>30</age>
      </user>
    `;
  }
}

// التطبيق الجديد يتوقع JSON
class JSONAdapter {
  constructor(xmlService) {
    this.xmlService = xmlService;
  }

  getJSONData() {
    const xmlData = this.xmlService.getXMLData();

    // تحويل XML إلى JSON (مبسط)
    const parser = new DOMParser();
    const xmlDoc = parser.parseFromString(xmlData, 'text/xml');

    const user = {
      name: xmlDoc.getElementsByTagName('name')[0].textContent,
      email: xmlDoc.getElementsByTagName('email')[0].textContent,
      age: parseInt(xmlDoc.getElementsByTagName('age')[0].textContent)
    };

    return user;
  }
}

// الاستخدام
const xmlService = new XMLDataService();
const adapter = new JSONAdapter(xmlService);

const userData = adapter.getJSONData();
console.log(userData); // { name: 'Ahmed', email: '...', age: 30 }
```

### مثال 2: Third-Party Library Adapter

```javascript
// العربية: محول لمكتبة خارجية

// مكتبة خارجية للدفع (مثل Stripe)
class StripeAPI {
  createCharge(token, amount, currency) {
    console.log(`Stripe: Charging ${amount} ${currency} with token ${token}`);
    return { id: 'ch_123', status: 'succeeded' };
  }
}

// مكتبة أخرى (مثل PayPal)
class PayPalAPI {
  pay(details) {
    console.log(`PayPal: Processing payment for ${details.total}`);
    return { transactionId: 'pp_456', state: 'approved' };
  }
}

// واجهة موحدة للتطبيق
interface PaymentGateway {
  charge(paymentInfo) { }
}

// Adapter لـ Stripe
class StripeAdapter {
  constructor() {
    this.stripe = new StripeAPI();
  }

  charge(paymentInfo) {
    const result = this.stripe.createCharge(
      paymentInfo.token,
      paymentInfo.amount,
      paymentInfo.currency || 'USD'
    );

    return {
      success: result.status === 'succeeded',
      transactionId: result.id
    };
  }
}

// Adapter لـ PayPal
class PayPalAdapter {
  constructor() {
    this.paypal = new PayPalAPI();
  }

  charge(paymentInfo) {
    const result = this.paypal.pay({
      total: paymentInfo.amount,
      currency: paymentInfo.currency || 'USD'
    });

    return {
      success: result.state === 'approved',
      transactionId: result.transactionId
    };
  }
}

// الاستخدام
function processPayment(gateway, paymentInfo) {
  const result = gateway.charge(paymentInfo);
  console.log('Payment result:', result);
}

const paymentInfo = {
  amount: 100,
  currency: 'USD',
  token: 'tok_123'
};

// يمكن استخدام أي gateway بنفس الواجهة
processPayment(new StripeAdapter(), paymentInfo);
processPayment(new PayPalAdapter(), paymentInfo);
```

### مثال 3: Database Adapter

```javascript
// العربية: محول قواعد البيانات

// قاعدة بيانات MongoDB
class MongoDB {
  connect(uri) {
    console.log(`Connecting to MongoDB: ${uri}`);
  }

  find(collection, query) {
    console.log(`MongoDB find in ${collection}:`, query);
    return [{ _id: '123', name: 'Ahmed' }];
  }

  insert(collection, document) {
    console.log(`MongoDB insert into ${collection}:`, document);
    return { _id: '123' };
  }
}

// قاعدة بيانات MySQL
class MySQL {
  connect(config) {
    console.log(`Connecting to MySQL: ${config.host}`);
  }

  select(table, conditions) {
    console.log(`MySQL SELECT FROM ${table} WHERE`, conditions);
    return [{ id: 1, name: 'Ahmed' }];
  }

  insert(table, data) {
    console.log(`MySQL INSERT INTO ${table}`, data);
    return { insertId: 1 };
  }
}

// واجهة موحدة
class DatabaseAdapter {
  connect() { }
  find(collection, query) { }
  insert(collection, data) { }
}

// Adapter لـ MongoDB
class MongoAdapter extends DatabaseAdapter {
  constructor() {
    super();
    this.db = new MongoDB();
  }

  connect(config) {
    this.db.connect(config.uri);
  }

  find(collection, query) {
    return this.db.find(collection, query);
  }

  insert(collection, data) {
    return this.db.insert(collection, data);
  }
}

// Adapter لـ MySQL
class MySQLAdapter extends DatabaseAdapter {
  constructor() {
    super();
    this.db = new MySQL();
  }

  connect(config) {
    this.db.connect(config);
  }

  find(collection, query) {
    return this.db.select(collection, query);
  }

  insert(collection, data) {
    return this.db.insert(collection, data);
  }
}

// الاستخدام
function useDatabase(adapter) {
  adapter.connect({ uri: 'localhost', host: 'localhost' });
  const users = adapter.find('users', { active: true });
  console.log(users);
}

useDatabase(new MongoAdapter());
useDatabase(new MySQLAdapter());
```

### مثال 4: Logger Adapter

```javascript
// العربية: محول أنظمة التسجيل

// مكتبة تسجيل قديمة
class LegacyLogger {
  logMessage(message, level) {
    const timestamp = new Date().toISOString();
    console.log(`[${timestamp}] [${level}] ${message}`);
  }
}

// مكتبة حديثة
class ModernLogger {
  log(data) {
    console.log(JSON.stringify({
      timestamp: data.timestamp,
      level: data.level,
      message: data.message,
      metadata: data.metadata
    }));
  }
}

// واجهة موحدة
class Logger {
  info(message, metadata = {}) { }
  warn(message, metadata = {}) { }
  error(message, metadata = {}) { }
}

// Adapter للنظام القديم
class LegacyLoggerAdapter extends Logger {
  constructor() {
    super();
    this.logger = new LegacyLogger();
  }

  info(message, metadata) {
    this.logger.logMessage(message, 'INFO');
  }

  warn(message, metadata) {
    this.logger.logMessage(message, 'WARN');
  }

  error(message, metadata) {
    this.logger.logMessage(message, 'ERROR');
  }
}

// Adapter للنظام الحديث
class ModernLoggerAdapter extends Logger {
  constructor() {
    super();
    this.logger = new ModernLogger();
  }

  info(message, metadata) {
    this.logger.log({
      timestamp: Date.now(),
      level: 'info',
      message,
      metadata
    });
  }

  warn(message, metadata) {
    this.logger.log({
      timestamp: Date.now(),
      level: 'warn',
      message,
      metadata
    });
  }

  error(message, metadata) {
    this.logger.log({
      timestamp: Date.now(),
      level: 'error',
      message,
      metadata
    });
  }
}

// الاستخدام
function logSomething(logger) {
  logger.info('Application started');
  logger.warn('Low memory', { available: '100MB' });
  logger.error('Database connection failed');
}

logSomething(new LegacyLoggerAdapter());
logSomething(new ModernLoggerAdapter());
```

### مثال 5: API Response Adapter

```javascript
// العربية: محول استجابات API

// API قديم
class OldAPI {
  async getUserData(userId) {
    return {
      user_id: userId,
      user_name: 'Ahmed Ali',
      user_email: 'ahmed@example.com',
      user_age: 30,
      created_date: '2020-01-01'
    };
  }
}

// API جديد
class NewAPI {
  async fetchUser(id) {
    return {
      id: id,
      profile: {
        fullName: 'Sara Mohamed',
        contact: {
          email: 'sara@example.com'
        },
        age: 25
      },
      metadata: {
        createdAt: '2021-01-01T00:00:00Z'
      }
    };
  }
}

// النموذج الموحد الذي يريده التطبيق
interface UserData {
  id: string;
  name: string;
  email: string;
  age: number;
  createdAt: string;
}

// Adapter للـ API القديم
class OldAPIAdapter {
  constructor() {
    this.api = new OldAPI();
  }

  async getUser(userId) {
    const data = await this.api.getUserData(userId);

    return {
      id: data.user_id,
      name: data.user_name,
      email: data.user_email,
      age: data.user_age,
      createdAt: data.created_date
    };
  }
}

// Adapter للـ API الجديد
class NewAPIAdapter {
  constructor() {
    this.api = new NewAPI();
  }

  async getUser(userId) {
    const data = await this.api.fetchUser(userId);

    return {
      id: data.id,
      name: data.profile.fullName,
      email: data.profile.contact.email,
      age: data.profile.age,
      createdAt: data.metadata.createdAt
    };
  }
}

// الاستخدام
async function displayUser(adapter, userId) {
  const user = await adapter.getUser(userId);
  console.log(`
    Name: ${user.name}
    Email: ${user.email}
    Age: ${user.age}
  `);
}

displayUser(new OldAPIAdapter(), '123');
displayUser(new NewAPIAdapter(), '456');
```

---

## المزايا والعيوب / Advantages & Disadvantages

### المزايا / Advantages

#### العربية
- ✅ **Single Responsibility**: فصل تحويل الواجهة عن منطق العمل
- ✅ **Open/Closed**: إضافة adapters جديدة بدون تعديل الكود الموجود
- ✅ **إعادة الاستخدام**: استخدام كود قديم مع أنظمة جديدة
- ✅ **المرونة**: تبديل التنفيذات بسهولة
- ✅ **Integration**: دمج مكتبات وأنظمة مختلفة

#### English
- ✅ **Single Responsibility**: Separate interface conversion from business logic
- ✅ **Open/Closed**: Add new adapters without modifying existing code
- ✅ **Reusability**: Use legacy code with new systems
- ✅ **Flexibility**: Easily swap implementations
- ✅ **Integration**: Integrate different libraries and systems

### العيوب / Disadvantages

#### العربية
- ❌ **تعقيد**: إضافة طبقة إضافية من التجريد
- ❌ **أداء**: Overhead بسيط في الأداء
- ❌ **صيانة**: المزيد من الأصناف للصيانة

#### English
- ❌ **Complexity**: Adding extra abstraction layer
- ❌ **Performance**: Slight performance overhead
- ❌ **Maintenance**: More classes to maintain

---

## متى تستخدم / When to Use

### العربية

استخدم Adapter Pattern عندما:

- ✅ تريد استخدام صنف موجود لكن واجهته غير متوافقة
- ✅ تدمج مكتبات أو APIs خارجية
- ✅ تعمل مع Legacy code
- ✅ تريد واجهة موحدة لتنفيذات مختلفة
- ✅ لا يمكنك تعديل الكود المصدري للصنف القديم

### English

Use Adapter Pattern when:

- ✅ You want to use existing class but interface is incompatible
- ✅ Integrating external libraries or APIs
- ✅ Working with legacy code
- ✅ You want unified interface for different implementations
- ✅ You cannot modify source code of legacy class

---

## أسئلة المقابلات / Interview Questions

### أسئلة أساسية / Basic Questions

**1. ما هو Adapter Pattern؟**

**الإجابة:**
نمط هيكلي يسمح لكائنات ذات واجهات غير متوافقة بالعمل معاً من خلال توفير واجهة وسيطة.

**2. ما الفرق بين Object Adapter و Class Adapter؟**

**الإجابة:**
- **Object Adapter**: يستخدم Composition (has-a)
- **Class Adapter**: يستخدم Inheritance (is-a)

**3. ما الفرق بين Adapter و Decorator؟**

**الإجابة:**
- **Adapter**: يغير الواجهة
- **Decorator**: يضيف وظائف جديدة بدون تغيير الواجهة

**4. متى تستخدم Adapter Pattern؟**

**الإجابة:**
عند دمج أنظمة ذات واجهات غير متوافقة، خاصة مع Legacy code أو Third-party libraries.

### أسئلة متقدمة / Advanced Questions

**5. كيف يساعد Adapter في تطبيق SOLID Principles؟**

**الإجابة:**
- **Single Responsibility**: فصل مسؤولية التحويل
- **Open/Closed**: إضافة adapters جديدة بدون تعديل
- **Dependency Inversion**: الاعتماد على Abstractions

**6. ما هي عيوب استخدام Adapter Pattern؟**

**الإجابة:**
- زيادة التعقيد
- طبقة إضافية قد تؤثر على الأداء
- المزيد من الأصناف للصيانة

**7. ما الفرق بين Adapter و Bridge Pattern؟**

**الإجابة:**
- **Adapter**: يُستخدم مع كود موجود (post-design)
- **Bridge**: يُصمم مسبقاً لفصل Abstraction عن Implementation (pre-design)

**8. اذكر أمثلة واقعية لـ Adapter Pattern؟**

**الإجابة:**
- محولات الكهرباء
- تحويل XML إلى JSON
- Wrappers لمكتبات خارجية
- Database adapters (PDO في PHP)

---

## الخلاصة / Summary

### العربية

**Adapter Pattern** يربط الواجهات غير المتوافقة:
- يسمح بإعادة استخدام الكود القديم
- يسهل دمج المكتبات الخارجية
- يوفر واجهة موحدة
- يطبق مبادئ SOLID

**متى تستخدمه:**
- Legacy code integration
- Third-party libraries
- واجهات غير متوافقة
- توحيد الواجهات

### English

**Adapter Pattern** connects incompatible interfaces:
- Allows legacy code reuse
- Facilitates external library integration
- Provides unified interface
- Applies SOLID principles

**When to use it:**
- Legacy code integration
- Third-party libraries
- Incompatible interfaces
- Interface unification
