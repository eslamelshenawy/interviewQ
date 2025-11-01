# Observer Pattern - نمط المراقب

## ما هو Observer Pattern / What is Observer Pattern

### العربية
**Observer Pattern** هو نمط تصميم سلوكي يُعرف أيضاً بـ Pub/Sub (Publish-Subscribe). يعرّف علاقة واحد-إلى-كثير (One-to-Many) بين الكائنات، بحيث عندما يتغير كائن واحد (Subject)، يتم إشعار جميع الكائنات المعتمدة عليه (Observers) تلقائياً.

**الفكرة الأساسية:** مثل الاشتراك في YouTube - عندما ينشر المستخدم فيديو، كل المشتركين يتلقون إشعار.

### English
**Observer Pattern** (also known as Pub/Sub) is a behavioral design pattern that defines a one-to-many dependency between objects, so that when one object (Subject) changes state, all its dependents (Observers) are notified automatically.

**Core Idea:** Like YouTube subscription - when a user publishes a video, all subscribers get notified.

---

## المشكلة والحل / Problem & Solution

### المشكلة
```javascript
// ❌ ارتباط قوي بين الكائنات
class Store {
  updatePrice(price) {
    this.price = price;

    // يجب تحديث كل شيء يدوياً
    ui.updatePrice(price);
    analytics.trackPriceChange(price);
    email.sendNotification(price);
    // صعب الصيانة!
  }
}
```

### الحل
```javascript
// ✅ Observer Pattern
class Subject {
  constructor() {
    this.observers = [];
  }

  subscribe(observer) {
    this.observers.push(observer);
  }

  unsubscribe(observer) {
    this.observers = this.observers.filter(obs => obs !== observer);
  }

  notify(data) {
    this.observers.forEach(observer => observer.update(data));
  }
}

class Store extends Subject {
  updatePrice(price) {
    this.price = price;
    this.notify(price); // إشعار جميع المشتركين
  }
}

// Observers
class UIObserver {
  update(price) {
    console.log(`UI updated: $${price}`);
  }
}

class AnalyticsObserver {
  update(price) {
    console.log(`Analytics tracked: $${price}`);
  }
}

class EmailObserver {
  update(price) {
    console.log(`Email sent: Price changed to $${price}`);
  }
}

// الاستخدام
const store = new Store();
store.subscribe(new UIObserver());
store.subscribe(new AnalyticsObserver());
store.subscribe(new EmailObserver());

store.updatePrice(100); // جميع المشتركين يتلقون الإشعار
```

---

## أمثلة عملية / Practical Examples

### مثال 1: News Publisher
```javascript
class NewsAgency {
  constructor() {
    this.subscribers = [];
    this.news = null;
  }

  subscribe(subscriber) {
    this.subscribers.push(subscriber);
  }

  unsubscribe(subscriber) {
    this.subscribers = this.subscribers.filter(sub => sub !== subscriber);
  }

  publishNews(news) {
    this.news = news;
    this.notifySubscribers();
  }

  notifySubscribers() {
    this.subscribers.forEach(sub => sub.notify(this.news));
  }
}

class NewsChannel {
  constructor(name) {
    this.name = name;
  }

  notify(news) {
    console.log(`${this.name} received news: ${news}`);
  }
}

// الاستخدام
const agency = new NewsAgency();
const cnn = new NewsChannel('CNN');
const bbc = new NewsChannel('BBC');
const aljazeera = new NewsChannel('Al Jazeera');

agency.subscribe(cnn);
agency.subscribe(bbc);
agency.subscribe(aljazeera);

agency.publishNews('Breaking: Major event happened!');
```

### مثال 2: Stock Market
```javascript
class Stock {
  constructor(symbol) {
    this.symbol = symbol;
    this.price = 0;
    this.investors = [];
  }

  attach(investor) {
    this.investors.push(investor);
  }

  detach(investor) {
    this.investors = this.investors.filter(inv => inv !== investor);
  }

  setPrice(price) {
    this.price = price;
    this.notify();
  }

  notify() {
    this.investors.forEach(investor => {
      investor.update(this);
    });
  }
}

class Investor {
  constructor(name) {
    this.name = name;
  }

  update(stock) {
    console.log(`${this.name} notified: ${stock.symbol} is now $${stock.price}`);
  }
}

// الاستخدام
const apple = new Stock('AAPL');

const investor1 = new Investor('Ahmed');
const investor2 = new Investor('Sara');

apple.attach(investor1);
apple.attach(investor2);

apple.setPrice(150); // كلا المستثمرين يتلقون إشعار
apple.setPrice(155);
```

### مثال 3: Event Bus
```javascript
class EventBus {
  constructor() {
    this.events = {};
  }

  on(eventName, callback) {
    if (!this.events[eventName]) {
      this.events[eventName] = [];
    }
    this.events[eventName].push(callback);
  }

  off(eventName, callback) {
    if (!this.events[eventName]) return;

    this.events[eventName] = this.events[eventName].filter(
      cb => cb !== callback
    );
  }

  emit(eventName, data) {
    if (!this.events[eventName]) return;

    this.events[eventName].forEach(callback => {
      callback(data);
    });
  }

  once(eventName, callback) {
    const wrapper = (data) => {
      callback(data);
      this.off(eventName, wrapper);
    };
    this.on(eventName, wrapper);
  }
}

// الاستخدام
const eventBus = new EventBus();

eventBus.on('user:login', (user) => {
  console.log(`User logged in: ${user.name}`);
});

eventBus.on('user:login', (user) => {
  console.log(`Send welcome email to ${user.email}`);
});

eventBus.emit('user:login', { name: 'Ahmed', email: 'ahmed@example.com' });
```

### مثال 4: Weather Station
```javascript
class WeatherStation {
  constructor() {
    this.temperature = 0;
    this.humidity = 0;
    this.pressure = 0;
    this.observers = [];
  }

  registerObserver(observer) {
    this.observers.push(observer);
  }

  removeObserver(observer) {
    this.observers = this.observers.filter(obs => obs !== observer);
  }

  notifyObservers() {
    this.observers.forEach(observer => {
      observer.update(this.temperature, this.humidity, this.pressure);
    });
  }

  setMeasurements(temperature, humidity, pressure) {
    this.temperature = temperature;
    this.humidity = humidity;
    this.pressure = pressure;
    this.notifyObservers();
  }
}

class PhoneDisplay {
  update(temp, humidity, pressure) {
    console.log(`📱 Phone: Temp ${temp}°C, Humidity ${humidity}%`);
  }
}

class TVDisplay {
  update(temp, humidity, pressure) {
    console.log(`📺 TV: Temperature ${temp}°C, Pressure ${pressure} hPa`);
  }
}

class WebDisplay {
  update(temp, humidity, pressure) {
    console.log(`🌐 Web: ${temp}°C, ${humidity}%, ${pressure} hPa`);
  }
}

// الاستخدام
const station = new WeatherStation();

station.registerObserver(new PhoneDisplay());
station.registerObserver(new TVDisplay());
station.registerObserver(new WebDisplay());

station.setMeasurements(25, 65, 1013);
station.setMeasurements(28, 70, 1015);
```

---

## المزايا والعيوب

### المزايا
- ✅ Open/Closed Principle - إضافة observers جدد بدون تعديل subject
- ✅ Loose Coupling بين Subject و Observers
- ✅ يدعم Broadcast Communication
- ✅ Dynamic relationships في runtime

### العيوب
- ❌ Observers يتلقون إشعارات بترتيب عشوائي
- ❌ Memory leaks إذا لم يتم unsubscribe
- ❌ صعوبة التتبع والتصحيح

---

## متى تستخدم

استخدم Observer عندما:
- ✅ تغيير في كائن يتطلب تغيير كائنات أخرى
- ✅ عدد الكائنات المتأثرة غير معروف مسبقاً
- ✅ Event-driven architecture
- ✅ Real-time updates

---

## أسئلة المقابلات

**1. ما الفرق بين Observer و Pub/Sub؟**
- **Observer**: Subject يعرف Observers مباشرة
- **Pub/Sub**: Event Bus وسيط، لا يعرف بعضهم

**2. كيف تتجنب Memory Leaks؟**
استخدم unsubscribe دائماً عند عدم الحاجة

**3. اذكر أمثلة واقعية؟**
- Event listeners في DOM
- React state management
- RxJS Observables
- Redux subscriptions
