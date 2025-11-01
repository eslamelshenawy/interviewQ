# Builder Pattern - نمط البناء

## المحتويات
1. [ما هو Builder Pattern](#ما-هو-builder-pattern)
2. [المشكلة](#المشكلة)
3. [الحل](#الحل)
4. [مكونات النمط](#مكونات-النمط)
5. [أمثلة عملية](#أمثلة-عملية)
6. [المزايا والعيوب](#المزايا-والعيوب)
7. [متى تستخدم](#متى-تستخدم)
8. [أسئلة المقابلات](#أسئلة-المقابلات)

---

## ما هو Builder Pattern / What is Builder Pattern

### العربية

**Builder Pattern** هو نمط تصميم إنشائي (Creational Design Pattern) يسمح ببناء كائنات معقدة خطوة بخطوة (Step by Step). يفصل بناء الكائن المعقد عن تمثيله، بحيث يمكن استخدام نفس عملية البناء لإنشاء تمثيلات مختلفة.

**الفكرة الأساسية:** بدلاً من استخدام Constructor معقد بعشرات المعاملات، نبني الكائن خطوة بخطوة باستخدام methods واضحة.

### English

**Builder Pattern** is a creational design pattern that allows you to construct complex objects step by step. It separates the construction of a complex object from its representation, allowing the same construction process to create different representations.

**Core Idea:** Instead of using a complex constructor with dozens of parameters, we build the object step by step using clear methods.

---

## المشكلة / The Problem

### العربية

تخيل أنك تبني نظام لإنشاء منازل:

```javascript
// ❌ مشكلة: Constructor معقد
class House {
  constructor(
    windows,
    doors,
    rooms,
    hasGarage,
    hasSwimmingPool,
    hasGarden,
    hasRoof,
    hasBasement,
    wallColor,
    roofColor
  ) {
    this.windows = windows;
    this.doors = doors;
    this.rooms = rooms;
    this.hasGarage = hasGarage;
    this.hasSwimmingPool = hasSwimmingPool;
    this.hasGarden = hasGarden;
    this.hasRoof = hasRoof;
    this.hasBasement = hasBasement;
    this.wallColor = wallColor;
    this.roofColor = roofColor;
  }
}

// الاستخدام صعب ومربك
const house = new House(10, 2, 4, true, true, false, true, false, 'white', 'red');
// ما هو ترتيب المعاملات؟ 🤔
```

**المشاكل:**
- Constructor معقد مع معاملات كثيرة
- صعوبة تذكر ترتيب المعاملات
- صعوبة إنشاء كائنات بخيارات مختلفة
- كود غير قابل للقراءة
- معاملات اختيارية كثيرة

### English

Imagine you're building a house creation system:

```javascript
// ❌ Problem: Complex constructor
// (Same code as above - code is universal)
```

**Problems:**
- Complex constructor with many parameters
- Hard to remember parameter order
- Difficult to create objects with different options
- Unreadable code
- Many optional parameters

---

## الحل / The Solution

### العربية

استخدم Builder Pattern لبناء الكائن خطوة بخطوة:

```javascript
// ✅ الحل: Builder Pattern
class House {
  constructor() {
    this.windows = 0;
    this.doors = 0;
    this.rooms = 0;
    this.hasGarage = false;
    this.hasSwimmingPool = false;
    this.hasGarden = false;
    this.hasRoof = true;
    this.hasBasement = false;
    this.wallColor = 'white';
    this.roofColor = 'gray';
  }
}

class HouseBuilder {
  constructor() {
    this.house = new House();
  }

  setWindows(count) {
    this.house.windows = count;
    return this; // للسماح بـ Method Chaining
  }

  setDoors(count) {
    this.house.doors = count;
    return this;
  }

  setRooms(count) {
    this.house.rooms = count;
    return this;
  }

  addGarage() {
    this.house.hasGarage = true;
    return this;
  }

  addSwimmingPool() {
    this.house.hasSwimmingPool = true;
    return this;
  }

  addGarden() {
    this.house.hasGarden = true;
    return this;
  }

  setWallColor(color) {
    this.house.wallColor = color;
    return this;
  }

  setRoofColor(color) {
    this.house.roofColor = color;
    return this;
  }

  build() {
    return this.house;
  }
}

// الاستخدام واضح وسهل القراءة
const house = new HouseBuilder()
  .setWindows(10)
  .setDoors(2)
  .setRooms(4)
  .addGarage()
  .addSwimmingPool()
  .setWallColor('blue')
  .setRoofColor('red')
  .build();

console.log(house);
```

### English

Use Builder Pattern to build the object step by step:

```javascript
// ✅ Solution: Builder Pattern
// (Same code as above - code is universal)
```

---

## مكونات النمط / Pattern Components

### 1. Product (المنتج)

الكائن المعقد الذي يتم بناؤه.

```javascript
class House {
  // الكائن النهائي
}
```

### 2. Builder (البناء)

يوفر methods لبناء أجزاء الكائن.

```javascript
class HouseBuilder {
  setWindows(count) { }
  setDoors(count) { }
  build() { }
}
```

### 3. Director (المدير) - اختياري

يستخدم Builder لبناء كائنات بخطوات محددة.

```javascript
class HouseDirector {
  constructor(builder) {
    this.builder = builder;
  }

  buildMinimalHouse() {
    return this.builder
      .setWindows(2)
      .setDoors(1)
      .setRooms(1)
      .build();
  }

  buildLuxuryHouse() {
    return this.builder
      .setWindows(20)
      .setDoors(5)
      .setRooms(10)
      .addGarage()
      .addSwimmingPool()
      .addGarden()
      .build();
  }
}

// الاستخدام
const builder = new HouseBuilder();
const director = new HouseDirector(builder);

const minimalHouse = director.buildMinimalHouse();
const luxuryHouse = director.buildLuxuryHouse();
```

---

## أمثلة عملية / Practical Examples

### مثال 1: Query Builder (SQL)

```javascript
// العربية: بناء استعلامات SQL

class SQLQuery {
  constructor() {
    this.selectFields = [];
    this.table = '';
    this.whereConditions = [];
    this.orderByFields = [];
    this.limitValue = null;
  }

  toString() {
    let query = `SELECT ${this.selectFields.join(', ')} FROM ${this.table}`;

    if (this.whereConditions.length > 0) {
      query += ` WHERE ${this.whereConditions.join(' AND ')}`;
    }

    if (this.orderByFields.length > 0) {
      query += ` ORDER BY ${this.orderByFields.join(', ')}`;
    }

    if (this.limitValue !== null) {
      query += ` LIMIT ${this.limitValue}`;
    }

    return query;
  }
}

class QueryBuilder {
  constructor() {
    this.query = new SQLQuery();
  }

  select(...fields) {
    this.query.selectFields.push(...fields);
    return this;
  }

  from(table) {
    this.query.table = table;
    return this;
  }

  where(condition) {
    this.query.whereConditions.push(condition);
    return this;
  }

  orderBy(...fields) {
    this.query.orderByFields.push(...fields);
    return this;
  }

  limit(value) {
    this.query.limitValue = value;
    return this;
  }

  build() {
    return this.query.toString();
  }
}

// الاستخدام
const query = new QueryBuilder()
  .select('id', 'name', 'email')
  .from('users')
  .where('age > 18')
  .where('active = true')
  .orderBy('created_at DESC')
  .limit(10)
  .build();

console.log(query);
// SELECT id, name, email FROM users WHERE age > 18 AND active = true ORDER BY created_at DESC LIMIT 10
```

### مثال 2: HTTP Request Builder

```javascript
// العربية: بناء طلبات HTTP

class HttpRequest {
  constructor() {
    this.method = 'GET';
    this.url = '';
    this.headers = {};
    this.params = {};
    this.body = null;
    this.timeout = 5000;
  }

  async execute() {
    const queryString = new URLSearchParams(this.params).toString();
    const fullUrl = queryString ? `${this.url}?${queryString}` : this.url;

    const options = {
      method: this.method,
      headers: this.headers,
      body: this.body ? JSON.stringify(this.body) : undefined
    };

    const response = await fetch(fullUrl, options);
    return response.json();
  }
}

class HttpRequestBuilder {
  constructor() {
    this.request = new HttpRequest();
  }

  setMethod(method) {
    this.request.method = method;
    return this;
  }

  setUrl(url) {
    this.request.url = url;
    return this;
  }

  addHeader(key, value) {
    this.request.headers[key] = value;
    return this;
  }

  addParam(key, value) {
    this.request.params[key] = value;
    return this;
  }

  setBody(body) {
    this.request.body = body;
    return this;
  }

  setTimeout(timeout) {
    this.request.timeout = timeout;
    return this;
  }

  build() {
    return this.request;
  }
}

// الاستخدام
const request = new HttpRequestBuilder()
  .setMethod('POST')
  .setUrl('https://api.example.com/users')
  .addHeader('Content-Type', 'application/json')
  .addHeader('Authorization', 'Bearer token123')
  .addParam('page', '1')
  .addParam('limit', '10')
  .setBody({
    name: 'Ahmed',
    email: 'ahmed@example.com'
  })
  .setTimeout(10000)
  .build();

// تنفيذ الطلب
request.execute().then(data => console.log(data));
```

### مثال 3: Email Builder

```javascript
// العربية: بناء رسائل البريد الإلكتروني

class Email {
  constructor() {
    this.to = [];
    this.cc = [];
    this.bcc = [];
    this.subject = '';
    this.body = '';
    this.attachments = [];
    this.priority = 'normal';
    this.isHtml = false;
  }

  send() {
    console.log('Sending email...');
    console.log('To:', this.to.join(', '));
    console.log('Subject:', this.subject);
    console.log('Body:', this.body);
    console.log('Attachments:', this.attachments.length);
  }
}

class EmailBuilder {
  constructor() {
    this.email = new Email();
  }

  to(email) {
    this.email.to.push(email);
    return this;
  }

  cc(email) {
    this.email.cc.push(email);
    return this;
  }

  bcc(email) {
    this.email.bcc.push(email);
    return this;
  }

  subject(subject) {
    this.email.subject = subject;
    return this;
  }

  body(body) {
    this.email.body = body;
    return this;
  }

  attach(file) {
    this.email.attachments.push(file);
    return this;
  }

  priority(level) {
    this.email.priority = level;
    return this;
  }

  asHtml() {
    this.email.isHtml = true;
    return this;
  }

  build() {
    return this.email;
  }
}

// الاستخدام
const email = new EmailBuilder()
  .to('user1@example.com')
  .to('user2@example.com')
  .cc('manager@example.com')
  .subject('Monthly Report')
  .body('<h1>Report</h1><p>Here is your monthly report...</p>')
  .asHtml()
  .attach('report.pdf')
  .attach('chart.png')
  .priority('high')
  .build();

email.send();
```

### مثال 4: Pizza Builder

```javascript
// العربية: بناء البيتزا

class Pizza {
  constructor() {
    this.size = 'medium';
    this.crust = 'regular';
    this.cheese = 'mozzarella';
    this.toppings = [];
    this.sauce = 'tomato';
    this.extraCheese = false;
  }

  describe() {
    console.log(`🍕 ${this.size} ${this.crust} crust pizza`);
    console.log(`   Cheese: ${this.cheese}${this.extraCheese ? ' (extra)' : ''}`);
    console.log(`   Sauce: ${this.sauce}`);
    console.log(`   Toppings: ${this.toppings.join(', ') || 'none'}`);
  }

  calculatePrice() {
    let price = 10; // base price

    if (this.size === 'large') price += 5;
    if (this.size === 'extra-large') price += 8;
    if (this.crust === 'stuffed') price += 3;
    if (this.extraCheese) price += 2;

    price += this.toppings.length * 1.5;

    return price;
  }
}

class PizzaBuilder {
  constructor() {
    this.pizza = new Pizza();
  }

  size(size) {
    this.pizza.size = size;
    return this;
  }

  crust(type) {
    this.pizza.crust = type;
    return this;
  }

  cheese(type) {
    this.pizza.cheese = type;
    return this;
  }

  addTopping(topping) {
    this.pizza.toppings.push(topping);
    return this;
  }

  sauce(type) {
    this.pizza.sauce = type;
    return this;
  }

  extraCheese() {
    this.pizza.extraCheese = true;
    return this;
  }

  build() {
    return this.pizza;
  }
}

// الاستخدام
const myPizza = new PizzaBuilder()
  .size('large')
  .crust('thin')
  .cheese('mozzarella')
  .sauce('tomato')
  .addTopping('pepperoni')
  .addTopping('mushrooms')
  .addTopping('olives')
  .extraCheese()
  .build();

myPizza.describe();
console.log(`Price: $${myPizza.calculatePrice()}`);
```

### مثال 5: User Builder (TypeScript)

```typescript
// العربية: بناء المستخدم

interface Address {
  street: string;
  city: string;
  country: string;
  zipCode: string;
}

interface User {
  id?: string;
  firstName: string;
  lastName: string;
  email: string;
  phone?: string;
  age?: number;
  address?: Address;
  role: 'admin' | 'user' | 'guest';
  isActive: boolean;
  createdAt: Date;
}

class UserBuilder {
  private user: Partial<User>;

  constructor() {
    this.user = {
      role: 'user',
      isActive: true,
      createdAt: new Date()
    };
  }

  setFirstName(firstName: string): this {
    this.user.firstName = firstName;
    return this;
  }

  setLastName(lastName: string): this {
    this.user.lastName = lastName;
    return this;
  }

  setEmail(email: string): this {
    this.user.email = email;
    return this;
  }

  setPhone(phone: string): this {
    this.user.phone = phone;
    return this;
  }

  setAge(age: number): this {
    this.user.age = age;
    return this;
  }

  setAddress(address: Address): this {
    this.user.address = address;
    return this;
  }

  setRole(role: 'admin' | 'user' | 'guest'): this {
    this.user.role = role;
    return this;
  }

  setActive(isActive: boolean): this {
    this.user.isActive = isActive;
    return this;
  }

  build(): User {
    if (!this.user.firstName || !this.user.lastName || !this.user.email) {
      throw new Error('First name, last name, and email are required');
    }

    this.user.id = Math.random().toString(36).substr(2, 9);

    return this.user as User;
  }
}

// الاستخدام
const user = new UserBuilder()
  .setFirstName('Ahmed')
  .setLastName('Ali')
  .setEmail('ahmed@example.com')
  .setPhone('+1234567890')
  .setAge(30)
  .setAddress({
    street: '123 Main St',
    city: 'Cairo',
    country: 'Egypt',
    zipCode: '12345'
  })
  .setRole('admin')
  .build();

console.log(user);
```

### مثال 6: Form Builder (React-like)

```javascript
// العربية: بناء النماذج

class FormField {
  constructor(name, type) {
    this.name = name;
    this.type = type;
    this.label = '';
    this.placeholder = '';
    this.required = false;
    this.defaultValue = '';
    this.validation = null;
  }
}

class Form {
  constructor() {
    this.fields = [];
    this.submitUrl = '';
    this.method = 'POST';
    this.onSubmit = null;
  }

  render() {
    console.log(`Form: ${this.method} ${this.submitUrl}`);
    this.fields.forEach(field => {
      console.log(`  - ${field.label} (${field.type})${field.required ? ' *' : ''}`);
    });
  }
}

class FormBuilder {
  constructor() {
    this.form = new Form();
  }

  addField(name, type) {
    const field = new FormField(name, type);
    this.currentField = field;
    this.form.fields.push(field);
    return this;
  }

  label(label) {
    if (this.currentField) {
      this.currentField.label = label;
    }
    return this;
  }

  placeholder(placeholder) {
    if (this.currentField) {
      this.currentField.placeholder = placeholder;
    }
    return this;
  }

  required() {
    if (this.currentField) {
      this.currentField.required = true;
    }
    return this;
  }

  defaultValue(value) {
    if (this.currentField) {
      this.currentField.defaultValue = value;
    }
    return this;
  }

  validate(fn) {
    if (this.currentField) {
      this.currentField.validation = fn;
    }
    return this;
  }

  submitTo(url) {
    this.form.submitUrl = url;
    return this;
  }

  method(method) {
    this.form.method = method;
    return this;
  }

  onSubmit(handler) {
    this.form.onSubmit = handler;
    return this;
  }

  build() {
    return this.form;
  }
}

// الاستخدام
const contactForm = new FormBuilder()
  .addField('name', 'text')
    .label('Full Name')
    .placeholder('Enter your name')
    .required()
  .addField('email', 'email')
    .label('Email Address')
    .placeholder('your@email.com')
    .required()
    .validate(email => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email))
  .addField('message', 'textarea')
    .label('Message')
    .placeholder('Type your message...')
    .required()
  .submitTo('/api/contact')
  .method('POST')
  .onSubmit(data => console.log('Submitting:', data))
  .build();

contactForm.render();
```

---

## المزايا والعيوب / Advantages & Disadvantages

### المزايا / Advantages

#### العربية
- ✅ **قابلية القراءة**: كود واضح وسهل الفهم
- ✅ **المرونة**: إنشاء كائنات بخيارات مختلفة
- ✅ **Method Chaining**: بناء الكائن بطريقة Fluent
- ✅ **Immutability**: يمكن جعل الكائن النهائي Immutable
- ✅ **Validation**: التحقق من البيانات قبل إنشاء الكائن
- ✅ **Default Values**: قيم افتراضية واضحة
- ✅ **Single Responsibility**: فصل منطق البناء عن منطق العمل

#### English
- ✅ **Readability**: Clear and easy to understand code
- ✅ **Flexibility**: Create objects with different options
- ✅ **Method Chaining**: Build object in fluent way
- ✅ **Immutability**: Can make final object immutable
- ✅ **Validation**: Validate data before creating object
- ✅ **Default Values**: Clear default values
- ✅ **Single Responsibility**: Separate construction from business logic

### العيوب / Disadvantages

#### العربية
- ❌ **كود إضافي**: إضافة صنف Builder
- ❌ **Over-engineering**: قد يكون مبالغاً فيه للكائنات البسيطة
- ❌ **Memory**: كائن Builder إضافي في الذاكرة

#### English
- ❌ **Extra Code**: Adding Builder class
- ❌ **Over-engineering**: May be overkill for simple objects
- ❌ **Memory**: Additional Builder object in memory

---

## متى تستخدم / When to Use

### العربية

استخدم Builder Pattern عندما:

- ✅ الكائن له معاملات كثيرة (أكثر من 4-5)
- ✅ معظم المعاملات اختيارية
- ✅ تريد Method Chaining
- ✅ بناء الكائن يحتاج خطوات متعددة
- ✅ تريد Immutable objects
- ✅ تريد Validation قبل إنشاء الكائن
- ✅ نفس عملية البناء لتمثيلات مختلفة

### English

Use Builder Pattern when:

- ✅ Object has many parameters (more than 4-5)
- ✅ Most parameters are optional
- ✅ You want method chaining
- ✅ Object construction needs multiple steps
- ✅ You want immutable objects
- ✅ You want validation before creating object
- ✅ Same construction process for different representations

---

## أسئلة المقابلات / Interview Questions

### أسئلة أساسية / Basic Questions

**1. ما هو Builder Pattern؟ / What is Builder Pattern?**

**الإجابة / Answer:**
- **العربية**: نمط تصميم إنشائي يسمح ببناء كائنات معقدة خطوة بخطوة
- **English**: A creational design pattern that allows building complex objects step by step

**2. ما الفرق بين Builder Pattern و Factory Pattern؟**

**الإجابة:**
- **Factory**: إنشاء كائن بخطوة واحدة
- **Builder**: إنشاء كائن خطوة بخطوة مع خيارات متعددة

**3. ما هو Method Chaining؟**

**الإجابة:**
```javascript
// Method Chaining: كل method ترجع this
builder
  .setName('Ahmed')
  .setAge(30)
  .build();
```

**4. لماذا نستخدم Builder بدلاً من Constructor طويل؟**

**الإجابة:**
- سهولة القراءة
- وضوح المعاملات
- معاملات اختيارية
- Validation

### أسئلة متقدمة / Advanced Questions

**5. كيف تنفذ Immutable Builder Pattern؟**

**الإجابة:**
```javascript
class ImmutableUser {
  constructor(builder) {
    this.name = builder.name;
    this.email = builder.email;
    Object.freeze(this); // جعل الكائن Immutable
  }

  static get Builder() {
    return class {
      setName(name) {
        this.name = name;
        return this;
      }

      setEmail(email) {
        this.email = email;
        return this;
      }

      build() {
        return new ImmutableUser(this);
      }
    };
  }
}

// الاستخدام
const user = new ImmutableUser.Builder()
  .setName('Ahmed')
  .setEmail('ahmed@example.com')
  .build();

user.name = 'Ali'; // لن يعمل - الكائن Immutable
```

**6. ما هو Director في Builder Pattern؟**

**الإجابة:**
Director يوفر طرق جاهزة لبناء كائنات شائعة:

```javascript
class HouseDirector {
  constructor(builder) {
    this.builder = builder;
  }

  buildStandardHouse() {
    return this.builder
      .setWindows(4)
      .setDoors(1)
      .setRooms(2)
      .build();
  }
}
```

**7. كيف تضيف Validation في Builder؟**

**الإجابة:**
```javascript
class UserBuilder {
  build() {
    if (!this.email || !this.email.includes('@')) {
      throw new Error('Invalid email');
    }
    if (!this.age || this.age < 18) {
      throw new Error('User must be 18+');
    }
    return new User(this);
  }
}
```

**8. ما هي أمثلة Builder Pattern في المكتبات الشهيرة؟**

**الإجابة:**
- `StringBuilder` في Java
- `jQuery` methods chaining
- `Lodash` chain
- `Mongoose` query builder
- `Knex.js` query builder
- `Axios` config builder

---

## الخلاصة / Summary

### العربية

**Builder Pattern** يحل مشكلة Constructors المعقدة:
- يبني الكائنات خطوة بخطوة
- يسمح بـ Method Chaining
- يحسن قابلية القراءة
- يوفر Validation وDefault Values

**متى تستخدمه:**
- كائنات بمعاملات كثيرة
- معاملات اختيارية
- Immutable objects
- Fluent Interface

### English

**Builder Pattern** solves complex constructor problem:
- Builds objects step by step
- Allows method chaining
- Improves readability
- Provides validation and default values

**When to use it:**
- Objects with many parameters
- Optional parameters
- Immutable objects
- Fluent interface
