# Strategy Pattern - نمط الاستراتيجية

## ما هو Strategy Pattern / What is Strategy Pattern

### العربية
**Strategy Pattern** هو نمط تصميم سلوكي يسمح بتعريف عائلة من الخوارزميات (Algorithms)، ووضع كل واحدة في صنف منفصل، وجعلها قابلة للتبديل (Interchangeable). يسمح للخوارزمية بالتغيير بشكل مستقل عن العملاء الذين يستخدمونها.

**الفكرة الأساسية:** بدلاً من if/else كثيرة، كل استراتيجية في صنف منفصل.

### English
**Strategy Pattern** is a behavioral design pattern that defines a family of algorithms, puts each one in a separate class, and makes them interchangeable. The algorithm can vary independently from clients that use it.

**Core Idea:** Instead of many if/else statements, each strategy in a separate class.

---

## المشكلة والحل / Problem & Solution

### المشكلة
```javascript
// ❌ كود مليء بـ if/else
class PaymentProcessor {
  processPayment(amount, method) {
    if (method === 'creditcard') {
      // معالجة بطاقة ائتمان
    } else if (method === 'paypal') {
      // معالجة PayPal
    } else if (method === 'crypto') {
      // معالجة Cryptocurrency
    }
    // صعب الصيانة والتوسع!
  }
}
```

### الحل
```javascript
// ✅ Strategy Pattern
// استراتيجيات منفصلة
class CreditCardStrategy {
  pay(amount) {
    console.log(`Paying ${amount} via Credit Card`);
  }
}

class PayPalStrategy {
  pay(amount) {
    console.log(`Paying ${amount} via PayPal`);
  }
}

class CryptoStrategy {
  pay(amount) {
    console.log(`Paying ${amount} via Cryptocurrency`);
  }
}

// Context
class PaymentProcessor {
  constructor(strategy) {
    this.strategy = strategy;
  }

  setStrategy(strategy) {
    this.strategy = strategy;
  }

  processPayment(amount) {
    this.strategy.pay(amount);
  }
}

// الاستخدام
const processor = new PaymentProcessor(new CreditCardStrategy());
processor.processPayment(100);

processor.setStrategy(new PayPalStrategy());
processor.processPayment(200);
```

---

## أمثلة عملية / Practical Examples

### مثال 1: Sorting Strategy
```javascript
class BubbleSortStrategy {
  sort(data) {
    console.log('Sorting using Bubble Sort');
    return data.sort((a, b) => a - b);
  }
}

class QuickSortStrategy {
  sort(data) {
    console.log('Sorting using Quick Sort');
    // تنفيذ Quick Sort
    return data.sort((a, b) => a - b);
  }
}

class MergeSortStrategy {
  sort(data) {
    console.log('Sorting using Merge Sort');
    // تنفيذ Merge Sort
    return data.sort((a, b) => a - b);
  }
}

class Sorter {
  constructor(strategy) {
    this.strategy = strategy;
  }

  sort(data) {
    return this.strategy.sort(data);
  }
}

// الاستخدام
const data = [5, 2, 8, 1, 9];
const sorter = new Sorter(new QuickSortStrategy());
sorter.sort(data);
```

### مثال 2: Validation Strategy
```javascript
class EmailValidationStrategy {
  validate(value) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(value);
  }
}

class PhoneValidationStrategy {
  validate(value) {
    const phoneRegex = /^\d{10}$/;
    return phoneRegex.test(value);
  }
}

class PasswordValidationStrategy {
  validate(value) {
    return value.length >= 8 && /[A-Z]/.test(value) && /[0-9]/.test(value);
  }
}

class Validator {
  constructor(strategy) {
    this.strategy = strategy;
  }

  validate(value) {
    return this.strategy.validate(value);
  }
}

// الاستخدام
const emailValidator = new Validator(new EmailValidationStrategy());
console.log(emailValidator.validate('test@example.com')); // true

const passwordValidator = new Validator(new PasswordValidationStrategy());
console.log(passwordValidator.validate('Pass123')); // true
```

### مثال 3: Compression Strategy
```javascript
class ZipCompressionStrategy {
  compress(files) {
    console.log('Compressing files using ZIP');
    return `${files.join(',')}.zip`;
  }
}

class RarCompressionStrategy {
  compress(files) {
    console.log('Compressing files using RAR');
    return `${files.join(',')}.rar`;
  }
}

class TarCompressionStrategy {
  compress(files) {
    console.log('Compressing files using TAR');
    return `${files.join(',')}.tar.gz`;
  }
}

class FileCompressor {
  constructor(strategy) {
    this.strategy = strategy;
  }

  compress(files) {
    return this.strategy.compress(files);
  }
}

// الاستخدام
const files = ['file1.txt', 'file2.txt'];
const compressor = new FileCompressor(new ZipCompressionStrategy());
compressor.compress(files);
```

### مثال 4: Shipping Strategy
```javascript
class StandardShippingStrategy {
  calculate(weight, distance) {
    return weight * 0.5 + distance * 0.1;
  }

  getDeliveryTime() {
    return '5-7 days';
  }
}

class ExpressShippingStrategy {
  calculate(weight, distance) {
    return weight * 1 + distance * 0.5;
  }

  getDeliveryTime() {
    return '1-2 days';
  }
}

class OvernightShippingStrategy {
  calculate(weight, distance) {
    return weight * 2 + distance * 1;
  }

  getDeliveryTime() {
    return 'Next day';
  }
}

class ShippingCalculator {
  constructor(strategy) {
    this.strategy = strategy;
  }

  calculate(weight, distance) {
    const cost = this.strategy.calculate(weight, distance);
    const time = this.strategy.getDeliveryTime();
    return { cost, deliveryTime: time };
  }
}

// الاستخدام
const shipping = new ShippingCalculator(new ExpressShippingStrategy());
console.log(shipping.calculate(5, 100)); // { cost: 55, deliveryTime: '1-2 days' }
```

---

## المزايا والعيوب

### المزايا
- ✅ Open/Closed Principle - إضافة استراتيجيات جديدة بسهولة
- ✅ تبديل الخوارزميات في Runtime
- ✅ عزل تفاصيل التنفيذ
- ✅ إزالة if/else المعقدة
- ✅ سهولة الاختبار

### العيوب
- ❌ زيادة عدد الأصناف
- ❌ العميل يجب أن يعرف الفرق بين الاستراتيجيات
- ❌ Over-engineering للحالات البسيطة

---

## متى تستخدم

استخدم Strategy عندما:
- ✅ عدة خوارزميات متشابهة
- ✅ if/else كثيرة لاختيار الخوارزمية
- ✅ تريد تبديل الخوارزمية في Runtime
- ✅ تريد عزل منطق الخوارزمية

---

## أسئلة المقابلات

**1. ما الفرق بين Strategy و State Pattern؟**
- **Strategy**: يركز على الخوارزمية، يختار العميل
- **State**: يركز على الحالة، تتغير تلقائياً

**2. ما الفرق بين Strategy و Factory؟**
- **Strategy**: يغير السلوك في runtime
- **Factory**: ينشئ كائنات

**3. اذكر أمثلة واقعية؟**
- Array.prototype.sort() callback
- React Router navigation strategies
- Express.js middleware
