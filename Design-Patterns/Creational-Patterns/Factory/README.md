# Factory Pattern - نمط المصنع

## المحتويات
1. [ما هو Factory Pattern](#ما-هو-factory-pattern)
2. [المشكلة](#المشكلة)
3. [الحل](#الحل)
4. [أنواع Factory Pattern](#أنواع-factory-pattern)
5. [أمثلة عملية](#أمثلة-عملية)
6. [المزايا والعيوب](#المزايا-والعيوب)
7. [متى تستخدم](#متى-تستخدم)
8. [أسئلة المقابلات](#أسئلة-المقابلات)

---

## ما هو Factory Pattern / What is Factory Pattern

### العربية

**Factory Pattern** هو نمط تصميم إنشائي (Creational Design Pattern) يوفر واجهة (Interface) لإنشاء الكائنات (Objects) دون تحديد الصنف (Class) المحدد للكائن الذي سيتم إنشاؤه.

**الفكرة الأساسية:** بدلاً من استدعاء `new` مباشرة، نستخدم دالة أو صنف يتولى إنشاء الكائنات.

### English

**Factory Pattern** is a creational design pattern that provides an interface for creating objects without specifying the exact class of the object that will be created.

**Core Idea:** Instead of calling `new` directly, we use a function or class that handles object creation.

---

## المشكلة / The Problem

### العربية

تخيل أنك تبني تطبيق لإدارة النقل:

```javascript
// ❌ مشكلة: الكود مرتبط بأصناف محددة
class TransportApp {
  createTransport(type) {
    if (type === 'truck') {
      return new Truck();
    } else if (type === 'ship') {
      return new Ship();
    } else if (type === 'plane') {
      return new Plane();
    }
  }
}
```

**المشاكل:**
- الكود مرتبط بشدة (Tight Coupling) بالأصناف المحددة
- إضافة نوع جديد يتطلب تعديل الكود الموجود
- صعوبة الاختبار (Testing)
- انتهاك مبدأ Open/Closed Principle

### English

Imagine you're building a transport management application:

```javascript
// ❌ Problem: Code is tightly coupled to specific classes
class TransportApp {
  createTransport(type) {
    if (type === 'truck') {
      return new Truck();
    } else if (type === 'ship') {
      return new Ship();
    } else if (type === 'plane') {
      return new Plane();
    }
  }
}
```

**Problems:**
- Code is tightly coupled to specific classes
- Adding a new type requires modifying existing code
- Difficult to test
- Violates Open/Closed Principle

---

## الحل / The Solution

### العربية

استخدم Factory Pattern لعزل منطق الإنشاء:

```javascript
// ✅ الحل: Factory Pattern
class Transport {
  deliver() {
    throw new Error('Method must be implemented');
  }
}

class Truck extends Transport {
  deliver() {
    return 'Delivering by land in a truck';
  }
}

class Ship extends Transport {
  deliver() {
    return 'Delivering by sea in a ship';
  }
}

class Plane extends Transport {
  deliver() {
    return 'Delivering by air in a plane';
  }
}

// Factory
class TransportFactory {
  static createTransport(type) {
    switch(type) {
      case 'truck':
        return new Truck();
      case 'ship':
        return new Ship();
      case 'plane':
        return new Plane();
      default:
        throw new Error(`Unknown transport type: ${type}`);
    }
  }
}

// الاستخدام
const truck = TransportFactory.createTransport('truck');
console.log(truck.deliver()); // Delivering by land in a truck

const ship = TransportFactory.createTransport('ship');
console.log(ship.deliver()); // Delivering by sea in a ship
```

### English

Use Factory Pattern to isolate creation logic:

```javascript
// ✅ Solution: Factory Pattern
// (Same code as above - code is universal)
```

---

## أنواع Factory Pattern / Types of Factory Pattern

### 1. Simple Factory (Factory Method)

```javascript
class LoggerFactory {
  static createLogger(type) {
    if (type === 'console') {
      return new ConsoleLogger();
    } else if (type === 'file') {
      return new FileLogger();
    } else if (type === 'database') {
      return new DatabaseLogger();
    }
    throw new Error(`Unknown logger type: ${type}`);
  }
}

// الاستخدام
const logger = LoggerFactory.createLogger('console');
logger.log('Hello World');
```

### 2. Factory Method Pattern

```javascript
// Abstract Creator
class Dialog {
  render() {
    const button = this.createButton();
    button.onClick();
    button.render();
  }

  // Factory Method
  createButton() {
    throw new Error('Must be implemented by subclass');
  }
}

// Concrete Creators
class WindowsDialog extends Dialog {
  createButton() {
    return new WindowsButton();
  }
}

class WebDialog extends Dialog {
  createButton() {
    return new HTMLButton();
  }
}

// Products
class Button {
  render() { }
  onClick() { }
}

class WindowsButton extends Button {
  render() {
    console.log('Rendering Windows button');
  }
  onClick() {
    console.log('Windows button clicked');
  }
}

class HTMLButton extends Button {
  render() {
    console.log('Rendering HTML button');
  }
  onClick() {
    console.log('HTML button clicked');
  }
}

// الاستخدام
const dialog = new WindowsDialog();
dialog.render();
```

### 3. Abstract Factory Pattern

```javascript
// Abstract Factory
class UIFactory {
  createButton() { }
  createCheckbox() { }
}

// Concrete Factory 1
class WindowsFactory extends UIFactory {
  createButton() {
    return new WindowsButton();
  }
  createCheckbox() {
    return new WindowsCheckbox();
  }
}

// Concrete Factory 2
class MacFactory extends UIFactory {
  createButton() {
    return new MacButton();
  }
  createCheckbox() {
    return new MacCheckbox();
  }
}

// Products
class WindowsButton {
  render() { return 'Windows Button'; }
}

class WindowsCheckbox {
  render() { return 'Windows Checkbox'; }
}

class MacButton {
  render() { return 'Mac Button'; }
}

class MacCheckbox {
  render() { return 'Mac Checkbox'; }
}

// الاستخدام
function renderUI(factory) {
  const button = factory.createButton();
  const checkbox = factory.createCheckbox();

  console.log(button.render());
  console.log(checkbox.render());
}

const os = 'windows';
const factory = os === 'windows' ? new WindowsFactory() : new MacFactory();
renderUI(factory);
```

---

## أمثلة عملية / Practical Examples

### مثال 1: Database Connection Factory

```javascript
// العربية: مصنع اتصالات قواعد البيانات

class DatabaseConnection {
  connect() { }
  query(sql) { }
  close() { }
}

class MySQLConnection extends DatabaseConnection {
  connect() {
    console.log('Connecting to MySQL database');
    return this;
  }

  query(sql) {
    console.log(`MySQL Query: ${sql}`);
    return [];
  }

  close() {
    console.log('MySQL connection closed');
  }
}

class PostgreSQLConnection extends DatabaseConnection {
  connect() {
    console.log('Connecting to PostgreSQL database');
    return this;
  }

  query(sql) {
    console.log(`PostgreSQL Query: ${sql}`);
    return [];
  }

  close() {
    console.log('PostgreSQL connection closed');
  }
}

class MongoDBConnection extends DatabaseConnection {
  connect() {
    console.log('Connecting to MongoDB database');
    return this;
  }

  query(sql) {
    console.log(`MongoDB Query: ${sql}`);
    return [];
  }

  close() {
    console.log('MongoDB connection closed');
  }
}

class DatabaseFactory {
  static createConnection(type, config) {
    switch(type) {
      case 'mysql':
        return new MySQLConnection(config);
      case 'postgresql':
        return new PostgreSQLConnection(config);
      case 'mongodb':
        return new MongoDBConnection(config);
      default:
        throw new Error(`Unsupported database type: ${type}`);
    }
  }
}

// الاستخدام
const dbType = process.env.DB_TYPE || 'mysql';
const db = DatabaseFactory.createConnection(dbType, {
  host: 'localhost',
  port: 3306
});

db.connect();
db.query('SELECT * FROM users');
db.close();
```

### مثال 2: Payment Method Factory

```javascript
// العربية: مصنع طرق الدفع

class PaymentMethod {
  processPayment(amount) { }
}

class CreditCardPayment extends PaymentMethod {
  constructor(cardNumber, cvv) {
    super();
    this.cardNumber = cardNumber;
    this.cvv = cvv;
  }

  processPayment(amount) {
    console.log(`Processing $${amount} via Credit Card ${this.cardNumber}`);
    // معالجة الدفع
    return { success: true, transactionId: 'CC-12345' };
  }
}

class PayPalPayment extends PaymentMethod {
  constructor(email) {
    super();
    this.email = email;
  }

  processPayment(amount) {
    console.log(`Processing $${amount} via PayPal (${this.email})`);
    // معالجة الدفع
    return { success: true, transactionId: 'PP-67890' };
  }
}

class CryptoPayment extends PaymentMethod {
  constructor(walletAddress) {
    super();
    this.walletAddress = walletAddress;
  }

  processPayment(amount) {
    console.log(`Processing $${amount} via Crypto to ${this.walletAddress}`);
    // معالجة الدفع
    return { success: true, transactionId: 'BTC-11111' };
  }
}

class PaymentFactory {
  static createPaymentMethod(type, details) {
    switch(type) {
      case 'creditcard':
        return new CreditCardPayment(details.cardNumber, details.cvv);
      case 'paypal':
        return new PayPalPayment(details.email);
      case 'crypto':
        return new CryptoPayment(details.walletAddress);
      default:
        throw new Error(`Unknown payment method: ${type}`);
    }
  }
}

// الاستخدام
const paymentType = 'creditcard';
const payment = PaymentFactory.createPaymentMethod(paymentType, {
  cardNumber: '**** **** **** 1234',
  cvv: '123'
});

const result = payment.processPayment(100);
console.log(result);
```

### مثال 3: Notification Factory

```javascript
// العربية: مصنع الإشعارات

class Notification {
  send(message, recipient) { }
}

class EmailNotification extends Notification {
  send(message, recipient) {
    console.log(`📧 Sending Email to ${recipient}`);
    console.log(`Subject: Notification`);
    console.log(`Body: ${message}`);
  }
}

class SMSNotification extends Notification {
  send(message, recipient) {
    console.log(`📱 Sending SMS to ${recipient}`);
    console.log(`Message: ${message}`);
  }
}

class PushNotification extends Notification {
  send(message, recipient) {
    console.log(`🔔 Sending Push Notification to ${recipient}`);
    console.log(`Message: ${message}`);
  }
}

class SlackNotification extends Notification {
  send(message, recipient) {
    console.log(`💬 Sending Slack message to ${recipient}`);
    console.log(`Message: ${message}`);
  }
}

class NotificationFactory {
  static createNotification(type) {
    switch(type) {
      case 'email':
        return new EmailNotification();
      case 'sms':
        return new SMSNotification();
      case 'push':
        return new PushNotification();
      case 'slack':
        return new SlackNotification();
      default:
        throw new Error(`Unknown notification type: ${type}`);
    }
  }
}

// الاستخدام
const preferences = {
  email: true,
  sms: true,
  push: false
};

Object.keys(preferences).forEach(type => {
  if (preferences[type]) {
    const notification = NotificationFactory.createNotification(type);
    notification.send('Your order has been shipped!', 'user@example.com');
  }
});
```

### مثال 4: Document Factory (TypeScript)

```typescript
// العربية: مصنع المستندات

interface Document {
  open(): void;
  save(): void;
  close(): void;
}

class PDFDocument implements Document {
  private fileName: string;

  constructor(fileName: string) {
    this.fileName = fileName;
  }

  open(): void {
    console.log(`Opening PDF: ${this.fileName}`);
  }

  save(): void {
    console.log(`Saving PDF: ${this.fileName}`);
  }

  close(): void {
    console.log(`Closing PDF: ${this.fileName}`);
  }
}

class WordDocument implements Document {
  private fileName: string;

  constructor(fileName: string) {
    this.fileName = fileName;
  }

  open(): void {
    console.log(`Opening Word document: ${this.fileName}`);
  }

  save(): void {
    console.log(`Saving Word document: ${this.fileName}`);
  }

  close(): void {
    console.log(`Closing Word document: ${this.fileName}`);
  }
}

class ExcelDocument implements Document {
  private fileName: string;

  constructor(fileName: string) {
    this.fileName = fileName;
  }

  open(): void {
    console.log(`Opening Excel spreadsheet: ${this.fileName}`);
  }

  save(): void {
    console.log(`Saving Excel spreadsheet: ${this.fileName}`);
  }

  close(): void {
    console.log(`Closing Excel spreadsheet: ${this.fileName}`);
  }
}

class DocumentFactory {
  static createDocument(fileName: string): Document {
    const extension = fileName.split('.').pop()?.toLowerCase();

    switch(extension) {
      case 'pdf':
        return new PDFDocument(fileName);
      case 'doc':
      case 'docx':
        return new WordDocument(fileName);
      case 'xls':
      case 'xlsx':
        return new ExcelDocument(fileName);
      default:
        throw new Error(`Unsupported file type: ${extension}`);
    }
  }
}

// الاستخدام
const doc1 = DocumentFactory.createDocument('report.pdf');
doc1.open();
doc1.save();

const doc2 = DocumentFactory.createDocument('letter.docx');
doc2.open();
doc2.save();
```

---

## المزايا والعيوب / Advantages & Disadvantages

### المزايا / Advantages

#### العربية
- ✅ **فصل المسؤوليات**: فصل منطق الإنشاء عن الاستخدام
- ✅ **سهولة الصيانة**: إضافة أنواع جديدة بدون تعديل الكود الموجود
- ✅ **إعادة الاستخدام**: منطق الإنشاء في مكان واحد
- ✅ **سهولة الاختبار**: يمكن استبدال الـ Factory بسهولة في الاختبارات
- ✅ **مرونة**: تغيير الكائنات المُنشأة بدون تعديل العملاء (Clients)
- ✅ **Single Responsibility Principle**: كل صنف له مسؤولية واحدة
- ✅ **Open/Closed Principle**: مفتوح للتوسع، مغلق للتعديل

#### English
- ✅ **Separation of Concerns**: Separates creation logic from usage
- ✅ **Easy Maintenance**: Add new types without modifying existing code
- ✅ **Reusability**: Creation logic in one place
- ✅ **Easy Testing**: Factory can be easily replaced in tests
- ✅ **Flexibility**: Change created objects without modifying clients
- ✅ **Single Responsibility Principle**: Each class has one responsibility
- ✅ **Open/Closed Principle**: Open for extension, closed for modification

### العيوب / Disadvantages

#### العربية
- ❌ **تعقيد إضافي**: إضافة أصناف جديدة للمشروع
- ❌ **Over-engineering**: قد يكون مبالغاً فيه للتطبيقات البسيطة
- ❌ **صعوبة التتبع**: قد يكون من الصعب تتبع أي صنف سيُنشأ

#### English
- ❌ **Additional Complexity**: Adding new classes to the project
- ❌ **Over-engineering**: May be overkill for simple applications
- ❌ **Difficult Tracking**: May be hard to track which class will be created

---

## متى تستخدم / When to Use

### العربية

استخدم Factory Pattern عندما:

- ✅ لا تعرف مسبقاً نوع الكائن الذي ستحتاجه
- ✅ تريد توفير مكتبة من الأصناف وإظهار واجهتها فقط
- ✅ تريد إعادة استخدام كائنات موجودة بدلاً من إنشاء جديدة
- ✅ تريد تطبيق مبادئ SOLID
- ✅ الكود يعتمد على أنواع مختلفة من الكائنات المرتبطة
- ✅ تريد تسهيل الاختبار باستخدام Mock Objects

### English

Use Factory Pattern when:

- ✅ You don't know beforehand the type of object you'll need
- ✅ You want to provide a library of classes and expose only their interface
- ✅ You want to reuse existing objects instead of creating new ones
- ✅ You want to apply SOLID principles
- ✅ Code depends on different types of related objects
- ✅ You want to facilitate testing using mock objects

---

## أسئلة المقابلات / Interview Questions

### أسئلة أساسية / Basic Questions

**1. ما هو Factory Pattern؟ / What is Factory Pattern?**

**الإجابة / Answer:**
- **العربية**: نمط تصميم إنشائي يوفر واجهة لإنشاء الكائنات دون تحديد الصنف المحدد
- **English**: A creational design pattern that provides an interface for creating objects without specifying the exact class

**2. ما الفرق بين Factory Method و Abstract Factory؟**

**الإجابة:**
- **Factory Method**: ينشئ كائن واحد، يستخدم الوراثة (Inheritance)
- **Abstract Factory**: ينشئ عائلة من الكائنات المرتبطة، يستخدم التكوين (Composition)

**3. لماذا نستخدم Factory بدلاً من `new`؟**

**الإجابة:**
- فصل منطق الإنشاء عن الاستخدام
- سهولة التعديل والصيانة
- تطبيق مبادئ SOLID
- سهولة الاختبار

### أسئلة متقدمة / Advanced Questions

**4. كيف يساعد Factory Pattern في تطبيق SOLID Principles؟**

**الإجابة:**
- **Single Responsibility**: فصل مسؤولية الإنشاء
- **Open/Closed**: إضافة أنواع جديدة بدون تعديل الكود
- **Dependency Inversion**: الاعتماد على Abstractions

**5. ما الفرق بين Simple Factory و Factory Method Pattern؟**

**الإجابة:**
- **Simple Factory**: دالة/صنف بسيط يحتوي على if/switch لإنشاء الكائنات (ليس نمط رسمي من GoF)
- **Factory Method**: نمط رسمي يستخدم الوراثة والـ Polymorphism

**6. كيف تختبر كود يستخدم Factory Pattern؟**

**الإجابة:**
```javascript
// Mock Factory للاختبار
class MockTransportFactory {
  static createTransport(type) {
    return {
      deliver: jest.fn().mockReturnValue('Mock delivery')
    };
  }
}

// في الاختبار
test('should create transport', () => {
  const transport = MockTransportFactory.createTransport('truck');
  expect(transport.deliver()).toBe('Mock delivery');
});
```

**7. متى لا يجب استخدام Factory Pattern؟**

**الإجابة:**
- التطبيق بسيط جداً
- عدد محدود من الأصناف لن يتغير
- Over-engineering بدون حاجة

**8. ما هي أمثلة Factory Pattern في المكتبات الشهيرة؟**

**الإجابة:**
- `React.createElement()`
- `document.createElement()`
- `jQuery.ajax()`
- `mongoose.model()`
- `express.Router()`

---

## الخلاصة / Summary

### العربية

**Factory Pattern** هو نمط أساسي في البرمجة الكائنية:
- يفصل منطق الإنشاء عن الاستخدام
- يسهل الصيانة والتوسع
- يطبق مبادئ SOLID
- يحسن قابلية الاختبار

**متى تستخدمه:**
- أنواع متعددة من الكائنات المرتبطة
- عدم معرفة نوع الكائن مسبقاً
- تطبيق يحتاج مرونة وقابلية توسع

### English

**Factory Pattern** is a fundamental pattern in OOP:
- Separates creation logic from usage
- Facilitates maintenance and extension
- Applies SOLID principles
- Improves testability

**When to use it:**
- Multiple types of related objects
- Unknown object type beforehand
- Application needs flexibility and extensibility
