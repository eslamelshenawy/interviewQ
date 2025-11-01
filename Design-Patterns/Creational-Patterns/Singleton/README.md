# Singleton Pattern - نمط المفرد

## المحتويات
1. [ما هو Singleton Pattern](#ما-هو-singleton-pattern)
2. [المشكلة](#المشكلة)
3. [الحل](#الحل)
4. [طرق التنفيذ](#طرق-التنفيذ)
5. [أمثلة عملية](#أمثلة-عملية)
6. [المزايا والعيوب](#المزايا-والعيوب)
7. [متى تستخدم](#متى-تستخدم)
8. [أسئلة المقابلات](#أسئلة-المقابلات)

---

## ما هو Singleton Pattern / What is Singleton Pattern

### العربية

**Singleton Pattern** هو نمط تصميم إنشائي (Creational Design Pattern) يضمن أن الصنف (Class) له نسخة واحدة فقط (Single Instance) في التطبيق بأكمله، ويوفر نقطة وصول عامة (Global Access Point) لهذه النسخة.

**الفكرة الأساسية:** بدلاً من إنشاء كائن جديد في كل مرة، نستخدم نفس النسخة دائماً.

### English

**Singleton Pattern** is a creational design pattern that ensures a class has only one instance throughout the application and provides a global access point to that instance.

**Core Idea:** Instead of creating a new object each time, we always use the same instance.

---

## المشكلة / The Problem

### العربية

تخيل أنك تريد:
- إدارة اتصال واحد بقاعدة البيانات
- Logger واحد للتطبيق
- Configuration manager واحد

```javascript
// ❌ مشكلة: إنشاء نسخ متعددة
const db1 = new Database();
const db2 = new Database();
const db3 = new Database();

// الآن لديك 3 اتصالات مختلفة!
// هذا يستهلك الموارد ويسبب مشاكل
```

**المشاكل:**
- استهلاك موارد غير ضروري
- عدم تزامن البيانات
- صعوبة إدارة الحالة المشتركة
- مشاكل في التطبيقات متعددة الخيوط (Multi-threaded)

### English

Imagine you want:
- One database connection manager
- One logger for the application
- One configuration manager

```javascript
// ❌ Problem: Creating multiple instances
// (Same code as above - code is universal)
```

**Problems:**
- Unnecessary resource consumption
- Data synchronization issues
- Difficult to manage shared state
- Issues in multi-threaded applications

---

## الحل / The Solution

### العربية

استخدم Singleton Pattern لضمان نسخة واحدة:

```javascript
// ✅ الحل: Singleton Pattern
class Database {
  constructor() {
    if (Database.instance) {
      return Database.instance;
    }

    this.connection = this.connect();
    Database.instance = this;
  }

  connect() {
    console.log('Creating new database connection');
    return { connected: true };
  }

  query(sql) {
    console.log(`Executing: ${sql}`);
  }
}

// الاستخدام
const db1 = new Database();
const db2 = new Database();
const db3 = new Database();

console.log(db1 === db2); // true
console.log(db2 === db3); // true
// جميعها نفس النسخة!
```

### English

Use Singleton Pattern to ensure one instance:

```javascript
// ✅ Solution: Singleton Pattern
// (Same code as above - code is universal)
```

---

## طرق التنفيذ / Implementation Methods

### 1. Basic Singleton (JavaScript)

```javascript
class Singleton {
  constructor() {
    if (Singleton.instance) {
      return Singleton.instance;
    }

    this.data = [];
    Singleton.instance = this;
  }

  addData(item) {
    this.data.push(item);
  }

  getData() {
    return this.data;
  }
}

const s1 = new Singleton();
s1.addData('First');

const s2 = new Singleton();
s2.addData('Second');

console.log(s1.getData()); // ['First', 'Second']
console.log(s1 === s2);    // true
```

### 2. Lazy Initialization Singleton

```javascript
class LazyS Singleton {
  static instance = null;

  constructor() {
    if (LazySingleton.instance) {
      return LazySingleton.instance;
    }

    this.createdAt = new Date();
    LazySingleton.instance = this;
  }

  static getInstance() {
    if (!LazySingleton.instance) {
      LazySingleton.instance = new LazySingleton();
    }
    return LazySingleton.instance;
  }
}

// الاستخدام
const instance = LazySingleton.getInstance();
```

### 3. Module Pattern Singleton (IIFE)

```javascript
const Singleton = (function() {
  let instance;

  function createInstance() {
    const object = {
      data: [],
      addData(item) {
        this.data.push(item);
      },
      getData() {
        return this.data;
      }
    };
    return object;
  }

  return {
    getInstance() {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();

// الاستخدام
const s1 = Singleton.getInstance();
const s2 = Singleton.getInstance();

console.log(s1 === s2); // true
```

### 4. ES6 Module Singleton

```javascript
// singleton.js
class ConfigManager {
  constructor() {
    this.config = {};
  }

  set(key, value) {
    this.config[key] = value;
  }

  get(key) {
    return this.config[key];
  }
}

// إنشاء نسخة واحدة وتصديرها
const instance = new ConfigManager();
export default instance;

// app.js
import config from './singleton.js';

config.set('apiUrl', 'https://api.example.com');
console.log(config.get('apiUrl'));
```

### 5. Frozen Singleton (Immutable)

```javascript
class FrozenSingleton {
  constructor() {
    if (FrozenSingleton.instance) {
      return FrozenSingleton.instance;
    }

    this.config = {
      apiUrl: 'https://api.example.com',
      timeout: 5000
    };

    // تجميد الكائن
    Object.freeze(this);
    FrozenSingleton.instance = this;
  }
}

const s1 = new FrozenSingleton();
s1.config = {}; // لن يعمل - الكائن مُجمد
```

### 6. TypeScript Singleton

```typescript
class Singleton {
  private static instance: Singleton;
  private constructor() {
    // Private constructor لمنع new من الخارج
  }

  public static getInstance(): Singleton {
    if (!Singleton.instance) {
      Singleton.instance = new Singleton();
    }
    return Singleton.instance;
  }

  public doSomething(): void {
    console.log('Doing something...');
  }
}

// الاستخدام
const instance = Singleton.getInstance();
// const wrong = new Singleton(); // Error: Constructor is private
```

---

## أمثلة عملية / Practical Examples

### مثال 1: Logger Singleton

```javascript
// العربية: نظام تسجيل موحد

class Logger {
  constructor() {
    if (Logger.instance) {
      return Logger.instance;
    }

    this.logs = [];
    this.logLevel = 'info';
    Logger.instance = this;
  }

  setLogLevel(level) {
    this.logLevel = level;
  }

  log(message, level = 'info') {
    const levels = ['debug', 'info', 'warn', 'error'];
    const currentLevelIndex = levels.indexOf(this.logLevel);
    const messageLevelIndex = levels.indexOf(level);

    if (messageLevelIndex >= currentLevelIndex) {
      const timestamp = new Date().toISOString();
      const logEntry = {
        timestamp,
        level,
        message
      };

      this.logs.push(logEntry);
      console.log(`[${timestamp}] [${level.toUpperCase()}] ${message}`);
    }
  }

  debug(message) {
    this.log(message, 'debug');
  }

  info(message) {
    this.log(message, 'info');
  }

  warn(message) {
    this.log(message, 'warn');
  }

  error(message) {
    this.log(message, 'error');
  }

  getLogs() {
    return this.logs;
  }

  clearLogs() {
    this.logs = [];
  }
}

// الاستخدام
const logger1 = new Logger();
logger1.info('Application started');

const logger2 = new Logger();
logger2.warn('Low memory warning');

console.log(logger1 === logger2); // true
console.log(logger1.getLogs());   // Both logs
```

### مثال 2: Database Connection Pool

```javascript
// العربية: إدارة اتصالات قاعدة البيانات

class DatabasePool {
  constructor() {
    if (DatabasePool.instance) {
      return DatabasePool.instance;
    }

    this.connections = [];
    this.maxConnections = 10;
    this.activeConnections = 0;
    this.init();

    DatabasePool.instance = this;
  }

  init() {
    console.log('Initializing database connection pool...');
  }

  getConnection() {
    if (this.activeConnections < this.maxConnections) {
      const connection = {
        id: ++this.activeConnections,
        query: (sql) => {
          console.log(`[Connection ${this.activeConnections}] Executing: ${sql}`);
        },
        release: () => {
          this.activeConnections--;
          console.log(`Connection released. Active: ${this.activeConnections}`);
        }
      };

      this.connections.push(connection);
      return connection;
    } else {
      throw new Error('No available connections');
    }
  }

  getStats() {
    return {
      total: this.maxConnections,
      active: this.activeConnections,
      available: this.maxConnections - this.activeConnections
    };
  }
}

// الاستخدام
const pool1 = new DatabasePool();
const pool2 = new DatabasePool();

console.log(pool1 === pool2); // true

const conn1 = pool1.getConnection();
conn1.query('SELECT * FROM users');

const conn2 = pool2.getConnection();
conn2.query('SELECT * FROM posts');

console.log(pool1.getStats());
```

### مثال 3: Application Config

```javascript
// العربية: إدارة إعدادات التطبيق

class AppConfig {
  constructor() {
    if (AppConfig.instance) {
      return AppConfig.instance;
    }

    this.config = this.loadConfig();
    AppConfig.instance = this;
  }

  loadConfig() {
    // في التطبيق الحقيقي، يتم تحميل من ملف أو بيئة
    return {
      app: {
        name: 'My Application',
        version: '1.0.0',
        env: process.env.NODE_ENV || 'development'
      },
      database: {
        host: process.env.DB_HOST || 'localhost',
        port: process.env.DB_PORT || 5432,
        name: process.env.DB_NAME || 'myapp'
      },
      api: {
        baseUrl: process.env.API_URL || 'https://api.example.com',
        timeout: 5000,
        retries: 3
      },
      features: {
        darkMode: true,
        analytics: true,
        notifications: false
      }
    };
  }

  get(path) {
    const keys = path.split('.');
    let value = this.config;

    for (const key of keys) {
      value = value[key];
      if (value === undefined) {
        return null;
      }
    }

    return value;
  }

  set(path, newValue) {
    const keys = path.split('.');
    const lastKey = keys.pop();
    let obj = this.config;

    for (const key of keys) {
      if (!(key in obj)) {
        obj[key] = {};
      }
      obj = obj[key];
    }

    obj[lastKey] = newValue;
  }

  getAll() {
    return { ...this.config };
  }
}

// الاستخدام
const config1 = new AppConfig();
console.log(config1.get('app.name')); // My Application

const config2 = new AppConfig();
config2.set('features.darkMode', false);

console.log(config1.get('features.darkMode')); // false (نفس النسخة)
console.log(config1 === config2); // true
```

### مثال 4: Cache Manager

```javascript
// العربية: إدارة التخزين المؤقت

class CacheManager {
  constructor() {
    if (CacheManager.instance) {
      return CacheManager.instance;
    }

    this.cache = new Map();
    this.defaultTTL = 60000; // 60 seconds
    CacheManager.instance = this;
  }

  set(key, value, ttl = this.defaultTTL) {
    const expiresAt = Date.now() + ttl;
    this.cache.set(key, {
      value,
      expiresAt
    });
  }

  get(key) {
    const item = this.cache.get(key);

    if (!item) {
      return null;
    }

    if (Date.now() > item.expiresAt) {
      this.cache.delete(key);
      return null;
    }

    return item.value;
  }

  has(key) {
    return this.get(key) !== null;
  }

  delete(key) {
    return this.cache.delete(key);
  }

  clear() {
    this.cache.clear();
  }

  size() {
    // تنظيف العناصر المنتهية
    this.cleanup();
    return this.cache.size;
  }

  cleanup() {
    const now = Date.now();
    for (const [key, item] of this.cache.entries()) {
      if (now > item.expiresAt) {
        this.cache.delete(key);
      }
    }
  }

  startAutoCleanup(interval = 60000) {
    setInterval(() => {
      this.cleanup();
      console.log(`Cache cleaned. Size: ${this.cache.size}`);
    }, interval);
  }
}

// الاستخدام
const cache1 = new CacheManager();
cache1.set('user:1', { name: 'Ahmed', email: 'ahmed@example.com' });

const cache2 = new CacheManager();
console.log(cache2.get('user:1')); // { name: 'Ahmed', ... }

console.log(cache1 === cache2); // true
```

### مثال 5: Event Bus (Pub/Sub)

```javascript
// العربية: ناقل الأحداث

class EventBus {
  constructor() {
    if (EventBus.instance) {
      return EventBus.instance;
    }

    this.events = {};
    EventBus.instance = this;
  }

  on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
  }

  off(event, callback) {
    if (!this.events[event]) return;

    this.events[event] = this.events[event].filter(cb => cb !== callback);
  }

  emit(event, data) {
    if (!this.events[event]) return;

    this.events[event].forEach(callback => {
      callback(data);
    });
  }

  once(event, callback) {
    const wrapper = (data) => {
      callback(data);
      this.off(event, wrapper);
    };
    this.on(event, wrapper);
  }

  clear() {
    this.events = {};
  }
}

// الاستخدام
const bus1 = new EventBus();
bus1.on('user:login', (user) => {
  console.log(`User logged in: ${user.name}`);
});

const bus2 = new EventBus();
bus2.emit('user:login', { name: 'Ahmed' });

console.log(bus1 === bus2); // true
```

### مثال 6: State Manager (Redux-like)

```javascript
// العربية: إدارة الحالة

class StateManager {
  constructor() {
    if (StateManager.instance) {
      return StateManager.instance;
    }

    this.state = {};
    this.listeners = [];
    StateManager.instance = this;
  }

  getState() {
    return { ...this.state };
  }

  setState(newState) {
    this.state = {
      ...this.state,
      ...newState
    };

    this.notify();
  }

  subscribe(listener) {
    this.listeners.push(listener);

    // إرجاع دالة لإلغاء الاشتراك
    return () => {
      this.listeners = this.listeners.filter(l => l !== listener);
    };
  }

  notify() {
    this.listeners.forEach(listener => {
      listener(this.state);
    });
  }
}

// الاستخدام
const store1 = new StateManager();
const unsubscribe = store1.subscribe((state) => {
  console.log('State changed:', state);
});

const store2 = new StateManager();
store2.setState({ user: { name: 'Ahmed' } });

console.log(store1 === store2); // true
console.log(store1.getState()); // { user: { name: 'Ahmed' } }
```

---

## المزايا والعيوب / Advantages & Disadvantages

### المزايا / Advantages

#### العربية
- ✅ **نسخة واحدة**: ضمان نسخة واحدة فقط
- ✅ **Global Access**: وصول عام من أي مكان
- ✅ **Lazy Initialization**: إنشاء النسخة عند الحاجة فقط
- ✅ **توفير الموارد**: عدم إنشاء نسخ متعددة
- ✅ **حالة مشتركة**: مشاركة البيانات بين أجزاء التطبيق
- ✅ **سهولة الصيانة**: نقطة واحدة للتحكم

#### English
- ✅ **Single Instance**: Ensures only one instance
- ✅ **Global Access**: Access from anywhere
- ✅ **Lazy Initialization**: Create instance only when needed
- ✅ **Resource Saving**: No multiple instances
- ✅ **Shared State**: Share data across application
- ✅ **Easy Maintenance**: Single point of control

### العيوب / Disadvantages

#### العربية
- ❌ **Global State**: حالة عامة يصعب تتبعها
- ❌ **Testing**: صعوبة الاختبار (Hard to Mock)
- ❌ **Tight Coupling**: ارتباط قوي بين الأجزاء
- ❌ **Violates SRP**: ينتهك مبدأ المسؤولية الواحدة
- ❌ **Concurrency**: مشاكل في البيئات متعددة الخيوط
- ❌ **Hidden Dependencies**: اعتماديات مخفية

#### English
- ❌ **Global State**: Hard to track global state
- ❌ **Testing**: Difficult to test (hard to mock)
- ❌ **Tight Coupling**: Strong coupling between parts
- ❌ **Violates SRP**: Violates Single Responsibility Principle
- ❌ **Concurrency**: Issues in multi-threaded environments
- ❌ **Hidden Dependencies**: Hidden dependencies

---

## متى تستخدم / When to Use

### العربية

استخدم Singleton Pattern عندما:

- ✅ تحتاج نسخة واحدة فقط (Database connection, Logger, Config)
- ✅ تريد نقطة وصول عامة للموارد المشتركة
- ✅ تريد Lazy initialization
- ✅ تحتاج مشاركة حالة بين أجزاء التطبيق
- ✅ تريد التحكم في الوصول للموارد

**لا تستخدم Singleton عندما:**
- ❌ يمكن استخدام Dependency Injection
- ❌ التطبيق بسيط
- ❌ تحتاج testability عالية
- ❌ تعمل في بيئة متعددة الخيوط معقدة

### English

Use Singleton Pattern when:

- ✅ You need only one instance (Database, Logger, Config)
- ✅ You want global access point to shared resource
- ✅ You want lazy initialization
- ✅ You need to share state across application
- ✅ You want to control resource access

**Don't use Singleton when:**
- ❌ You can use Dependency Injection
- ❌ Application is simple
- ❌ You need high testability
- ❌ Working in complex multi-threaded environment

---

## أسئلة المقابلات / Interview Questions

### أسئلة أساسية / Basic Questions

**1. ما هو Singleton Pattern؟**

**الإجابة:**
نمط تصميم يضمن أن الصنف له نسخة واحدة فقط في التطبيق ويوفر نقطة وصول عامة لها.

**2. كيف تنفذ Singleton في JavaScript؟**

**الإجابة:**
```javascript
class Singleton {
  constructor() {
    if (Singleton.instance) {
      return Singleton.instance;
    }
    Singleton.instance = this;
  }
}
```

**3. ما الفرق بين Singleton و Static Class؟**

**الإجابة:**
- **Singleton**: يمكن أن يرث من أصناف أخرى، Lazy initialization
- **Static Class**: لا يمكن الوراثة، يُحمل مباشرة

### أسئلة متقدمة / Advanced Questions

**4. ما هي مشاكل Singleton في Multi-threaded Environment؟**

**الإجابة:**
في البيئات متعددة الخيوط، قد يتم إنشاء أكثر من نسخة إذا لم يتم استخدام Synchronization.

**5. كيف تختبر كود يستخدم Singleton؟**

**الإجابة:**
```javascript
// مشكلة الاختبار
test('test 1', () => {
  const singleton = new Singleton();
  singleton.data = 'test1';
});

test('test 2', () => {
  const singleton = new Singleton();
  // singleton.data لا يزال 'test1' من الاختبار السابق!
});

// الحل: إعادة تعيين Singleton بين الاختبارات
afterEach(() => {
  Singleton.instance = null;
});
```

**6. ما هي بدائل Singleton Pattern؟**

**الإجابة:**
- **Dependency Injection**: حقن التبعيات
- **Module Pattern**: استخدام ES6 modules
- **Factory Pattern**: لإنشاء وإدارة النسخ

**7. هل Singleton يعتبر Anti-pattern؟**

**الإجابة:**
البعض يعتبره Anti-pattern لأنه:
- ينتهك Single Responsibility Principle
- يصعب الاختبار
- يخلق Global State
- لكنه مفيد في حالات محددة

**8. ما هي أمثلة Singleton في JavaScript APIs؟**

**الإجابة:**
- `window` object في المتصفح
- `Math` object
- `JSON` object
- `console` object

---

## الخلاصة / Summary

### العربية

**Singleton Pattern** يضمن نسخة واحدة فقط:
- مفيد للموارد المشتركة (Database, Logger, Config)
- يوفر Global Access Point
- يدعم Lazy Initialization
- لكن يجب استخدامه بحذر

**البدائل الحديثة:**
- Dependency Injection
- ES6 Modules
- Context API (React)
- State Management Libraries

### English

**Singleton Pattern** ensures single instance:
- Useful for shared resources (Database, Logger, Config)
- Provides global access point
- Supports lazy initialization
- But use with caution

**Modern Alternatives:**
- Dependency Injection
- ES6 Modules
- Context API (React)
- State Management Libraries
