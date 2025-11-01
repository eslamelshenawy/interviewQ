# Decorator Pattern - نمط المزخرف

## ما هو Decorator Pattern / What is Decorator Pattern

### العربية
**Decorator Pattern** هو نمط تصميم هيكلي يسمح بإضافة سلوكيات (Behaviors) جديدة للكائنات بشكل ديناميكي دون تعديل الكود الأصلي. يلف (Wraps) الكائن الأصلي بكائنات decorator.

**الفكرة الأساسية:** مثل تزيين الكيك - تضيف طبقات جديدة فوق الطبقة الأساسية.

### English
**Decorator Pattern** is a structural design pattern that allows adding new behaviors to objects dynamically without modifying the original code. It wraps the original object with decorator objects.

**Core Idea:** Like decorating a cake - you add new layers on top of the base layer.

---

## المشكلة والحل / Problem & Solution

### المشكلة
```javascript
// ❌ استخدام الوراثة يخلق انفجار في عدد الأصناف
class Coffee { cost() { return 5; } }
class CoffeeWithMilk extends Coffee { cost() { return 6; } }
class CoffeeWithSugar extends Coffee { cost() { return 5.5; } }
class CoffeeWithMilkAndSugar extends Coffee { cost() { return 6.5; } }
// ماذا لو أردنا إضافة Chocolate؟ المزيد من الأصناف!
```

### الحل
```javascript
// ✅ Decorator Pattern
class Coffee {
  cost() { return 5; }
  description() { return 'Coffee'; }
}

class MilkDecorator {
  constructor(coffee) {
    this.coffee = coffee;
  }

  cost() { return this.coffee.cost() + 1; }
  description() { return this.coffee.description() + ', Milk'; }
}

class SugarDecorator {
  constructor(coffee) {
    this.coffee = coffee;
  }

  cost() { return this.coffee.cost() + 0.5; }
  description() { return this.coffee.description() + ', Sugar'; }
}

// الاستخدام - مرونة كاملة!
let coffee = new Coffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);
console.log(coffee.description()); // Coffee, Milk, Sugar
console.log(coffee.cost()); // 6.5
```

---

## أمثلة عملية / Practical Examples

### مثال 1: Text Formatting
```javascript
class Text {
  constructor(content) {
    this.content = content;
  }

  render() {
    return this.content;
  }
}

class BoldDecorator {
  constructor(text) {
    this.text = text;
  }

  render() {
    return `<strong>${this.text.render()}</strong>`;
  }
}

class ItalicDecorator {
  constructor(text) {
    this.text = text;
  }

  render() {
    return `<em>${this.text.render()}</em>`;
  }
}

class UnderlineDecorator {
  constructor(text) {
    this.text = text;
  }

  render() {
    return `<u>${this.text.render()}</u>`;
  }
}

// الاستخدام
let text = new Text('Hello World');
text = new BoldDecorator(text);
text = new ItalicDecorator(text);
text = new UnderlineDecorator(text);

console.log(text.render());
// <u><em><strong>Hello World</strong></em></u>
```

### مثال 2: Logger Decorator
```javascript
class DataService {
  getData() {
    return { data: [1, 2, 3] };
  }
}

class LoggerDecorator {
  constructor(service) {
    this.service = service;
  }

  getData() {
    console.log('[LOG] Getting data...');
    const data = this.service.getData();
    console.log('[LOG] Data retrieved:', data);
    return data;
  }
}

class CacheDecorator {
  constructor(service) {
    this.service = service;
    this.cache = null;
  }

  getData() {
    if (this.cache) {
      console.log('[CACHE] Returning cached data');
      return this.cache;
    }

    const data = this.service.getData();
    this.cache = data;
    return data;
  }
}

class ValidationDecorator {
  constructor(service) {
    this.service = service;
  }

  getData() {
    const data = this.service.getData();
    if (!data || !data.data || !Array.isArray(data.data)) {
      throw new Error('Invalid data format');
    }
    return data;
  }
}

// الاستخدام - يمكن تركيب decorators بأي ترتيب
let service = new DataService();
service = new ValidationDecorator(service);
service = new CacheDecorator(service);
service = new LoggerDecorator(service);

service.getData();
```

### مثال 3: Pizza Decorator
```javascript
class Pizza {
  cost() { return 10; }
  description() { return 'Basic Pizza'; }
}

class CheeseDecorator {
  constructor(pizza) {
    this.pizza = pizza;
  }

  cost() { return this.pizza.cost() + 2; }
  description() { return this.pizza.description() + ' + Cheese'; }
}

class PepperoniDecorator {
  constructor(pizza) {
    this.pizza = pizza;
  }

  cost() { return this.pizza.cost() + 3; }
  description() { return this.pizza.description() + ' + Pepperoni'; }
}

class MushroomDecorator {
  constructor(pizza) {
    this.pizza = pizza;
  }

  cost() { return this.pizza.cost() + 1.5; }
  description() { return this.pizza.description() + ' + Mushroom'; }
}

// الاستخدام
let myPizza = new Pizza();
myPizza = new CheeseDecorator(myPizza);
myPizza = new PepperoniDecorator(myPizza);
myPizza = new MushroomDecorator(myPizza);

console.log(myPizza.description()); // Basic Pizza + Cheese + Pepperoni + Mushroom
console.log(`Cost: $${myPizza.cost()}`); // Cost: $16.5
```

---

## المزايا والعيوب

### المزايا
- ✅ إضافة وظائف دون تعديل الكود الأصلي
- ✅ مرونة أكبر من الوراثة
- ✅ Single Responsibility - كل decorator مسؤول عن وظيفة واحدة
- ✅ يمكن تركيب decorators بأي ترتيب
- ✅ Open/Closed Principle

### العيوب
- ❌ عدد كبير من الكائنات الصغيرة
- ❌ صعوبة الفهم للمبتدئين
- ❌ صعوبة إزالة decorator محدد من المنتصف

---

## متى تستخدم

استخدم Decorator عندما:
- ✅ تريد إضافة وظائف لكائنات بشكل ديناميكي
- ✅ لا يمكن استخدام الوراثة (final classes)
- ✅ الوراثة ستؤدي لعدد كبير من الأصناف
- ✅ تريد تركيب وظائف بطرق مختلفة

---

## أسئلة المقابلات

**1. ما الفرق بين Decorator و Inheritance؟**
- **Decorator**: ديناميكي، runtime، مرن
- **Inheritance**: ستاتيكي، compile-time، جامد

**2. ما الفرق بين Decorator و Proxy؟**
- **Decorator**: يضيف وظائف جديدة
- **Proxy**: يتحكم في الوصول

**3. اذكر أمثلة Decorator في JavaScript؟**
- React Higher-Order Components (HOC)
- Express.js middleware
- Python decorators (@decorator)
