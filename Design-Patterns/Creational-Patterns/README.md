# Creational Design Patterns | الأنماط الإنشائية

## Overview | نظرة عامة

Creational patterns deal with object creation mechanisms, trying to create objects in a manner suitable to the situation.

الأنماط الإنشائية تتعامل مع آليات إنشاء الكائنات، محاولة إنشاء الكائنات بطريقة مناسبة للوضع.

---

## The Three Main Patterns | الأنماط الثلاثة الرئيسية

### 1. [Factory Pattern](./01-Factory-Pattern.md)
**Purpose:** Creates objects without specifying the exact class

**When to use:**
- When you don't know ahead of time what class object you need
- When you want to centralize object creation logic
- When you have multiple similar classes

**Example:** ShapeFactory creates Circle, Rectangle, Triangle

---

### 2. [Builder Pattern](./02-Builder-Pattern.md)
**Purpose:** Constructs complex objects step by step

**When to use:**
- When object creation requires many parameters
- When you want to make objects immutable
- When construction process is complex

**Example:** Building a User with name, age, email, address step by step

---

### 3. [Singleton Pattern](./03-Singleton-Pattern.md)
**Purpose:** Ensures only one instance of a class exists

**When to use:**
- When exactly one instance is needed
- For logging, caching, thread pools
- When creating the object is expensive

**Example:** DatabaseConnection, Configuration, Logger

---

## Quick Comparison | مقارنة سريعة

| Pattern | Purpose | Use Case | Example |
|---------|---------|----------|---------|
| **Factory** | Hide creation logic | Multiple similar objects | ShapeFactory |
| **Builder** | Step-by-step construction | Complex objects | UserBuilder |
| **Singleton** | Single instance | Global access point | Logger |

---

## When to Use Each Pattern | متى تستخدم كل نمط

### Use Factory When | استخدم Factory عندما:
- ✅ Need to create different types of similar objects
- ✅ Want to hide creation complexity
- ✅ Have a family of related objects
- ✅ Decision of which object to create is runtime-based

### Use Builder When | استخدم Builder عندما:
- ✅ Object has many constructor parameters (4+)
- ✅ Need optional parameters
- ✅ Want immutable objects
- ✅ Construction process has multiple steps

### Use Singleton When | استخدم Singleton عندما:
- ✅ Need exactly one instance
- ✅ Instance should be globally accessible
- ✅ Lazy initialization is beneficial
- ⚠️ Be careful: Can make testing difficult!

---

## Code Examples Overview | نظرة عامة على أمثلة الكود

### Factory Pattern
```java
interface Shape { void draw(); }
class Circle implements Shape { }
class Square implements Shape { }

class ShapeFactory {
    public Shape getShape(String type) {
        if (type.equals("CIRCLE")) return new Circle();
        if (type.equals("SQUARE")) return new Square();
        return null;
    }
}
```

### Builder Pattern
```java
class User {
    private String name;
    private int age;

    private User(Builder builder) {
        this.name = builder.name;
        this.age = builder.age;
    }

    static class Builder {
        private String name;
        private int age;

        public Builder name(String name) {
            this.name = name;
            return this;
        }

        public Builder age(int age) {
            this.age = age;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }
}

// Usage
User user = new User.Builder()
    .name("John")
    .age(30)
    .build();
```

### Singleton Pattern
```java
class Logger {
    private static Logger instance;

    private Logger() { }

    public static Logger getInstance() {
        if (instance == null) {
            instance = new Logger();
        }
        return instance;
    }
}
```

---

## Interview Questions | أسئلة المقابلات

### Q1: What are Creational Design Patterns?
**Answer:**
Creational patterns deal with object creation mechanisms. They abstract the instantiation process, making systems independent of how objects are created, composed, and represented. The main patterns are Factory, Builder, and Singleton.

### Q2: Difference between Factory and Builder?
**Answer:**
- **Factory**: Creates complete objects in one step, focuses on *what* to create
- **Builder**: Creates objects step-by-step, focuses on *how* to create
- Factory for simple objects with variations
- Builder for complex objects with many parameters

### Q3: Why is Singleton considered an anti-pattern?
**Answer:**
- Makes unit testing difficult (global state)
- Hides dependencies
- Can cause issues in multi-threaded environments
- Violates Single Responsibility Principle
- Hard to mock in tests
**Better alternatives:** Dependency Injection, use sparingly

### Q4: Can you combine these patterns?
**Answer:**
Yes! Common combinations:
- Singleton Factory: Factory that is a Singleton
- Builder with Factory: Factory returns a Builder
- Prototype with Factory: Factory clones prototypes

### Q5: Real-world examples?
**Answer:**
- **Factory**: JDBC DriverManager, Spring BeanFactory
- **Builder**: StringBuilder, Lombok @Builder, HTTP clients
- **Singleton**: Runtime, Spring Beans (default scope)

---

## Best Practices | أفضل الممارسات

### Factory Pattern
- ✅ Use when you have a family of related objects
- ✅ Return interfaces, not concrete classes
- ✅ Consider Abstract Factory for multiple factories
- ❌ Don't use for simple object creation

### Builder Pattern
- ✅ Use for objects with 4+ parameters
- ✅ Make original class immutable
- ✅ Use method chaining for fluency
- ✅ Validate in build() method
- ❌ Don't use for simple objects

### Singleton Pattern
- ✅ Use thread-safe implementation
- ✅ Consider lazy initialization
- ✅ Make constructor private
- ⚠️ Use sparingly - consider alternatives
- ❌ Don't use for everything

---

## Common Mistakes | الأخطاء الشائعة

### Factory Mistakes
```java
// ❌ Bad: Returning concrete class
public Circle createCircle() { }

// ✅ Good: Returning interface
public Shape createShape() { }
```

### Builder Mistakes
```java
// ❌ Bad: Not validating
public User build() {
    return new User(this);  // No validation!
}

// ✅ Good: Validate before building
public User build() {
    if (name == null) throw new IllegalStateException();
    return new User(this);
}
```

### Singleton Mistakes
```java
// ❌ Bad: Not thread-safe
public static Singleton getInstance() {
    if (instance == null) {
        instance = new Singleton();  // Race condition!
    }
    return instance;
}

// ✅ Good: Thread-safe (Double-check locking)
public static Singleton getInstance() {
    if (instance == null) {
        synchronized (Singleton.class) {
            if (instance == null) {
                instance = new Singleton();
            }
        }
    }
    return instance;
}
```

---

## Comparison with Other Pattern Categories | المقارنة مع فئات الأنماط الأخرى

| Category | Focus | Example |
|----------|-------|---------|
| **Creational** | Object creation | Factory, Builder, Singleton |
| **Structural** | Object composition | Adapter, Decorator, Facade |
| **Behavioral** | Object interaction | Strategy, Observer, Command |

---

## Related Patterns | الأنماط المرتبطة

### Factory + Strategy
```java
class PaymentFactory {
    public PaymentStrategy createPayment(String type) {
        // Returns different payment strategies
    }
}
```

### Builder + Prototype
```java
class UserBuilder {
    public Builder from(User prototype) {
        // Clone and modify
    }
}
```

### Singleton + Factory
```java
class DatabaseFactory {
    private static DatabaseFactory instance;

    public static DatabaseFactory getInstance() { }

    public Database createDatabase(String type) { }
}
```

---

## Practice Exercises | تمارين عملية

### Exercise 1: Pizza Builder
Create a Pizza class with Builder pattern:
- Toppings (cheese, pepperoni, mushrooms)
- Size (small, medium, large)
- Crust type (thin, thick, stuffed)

### Exercise 2: Vehicle Factory
Create a factory that produces:
- Car, Bike, Truck
- Each with different properties
- Use interface and polymorphism

### Exercise 3: Configuration Singleton
Create a thread-safe Configuration singleton:
- Loads from file once
- Provides global access
- Implements lazy initialization

---

## Resources | المصادر

### Books
- **Design Patterns: Elements of Reusable Object-Oriented Software** (Gang of Four)
- **Head First Design Patterns**
- **Effective Java** by Joshua Bloch (Builder pattern)

### Online
- [Refactoring Guru - Creational Patterns](https://refactoring.guru/design-patterns/creational-patterns)
- [Source Making - Creational Patterns](https://sourcemaking.com/design-patterns/creational-patterns)

---

## Summary | الملخص

**English:**
Creational patterns solve problems related to object creation. Factory abstracts object creation, Builder constructs complex objects step by step, and Singleton ensures single instance. Choose the right pattern based on your needs: Factory for variations, Builder for complexity, Singleton for single instance.

**العربية:**
الأنماط الإنشائية تحل مشاكل تتعلق بإنشاء الكائنات. Factory يجرد إنشاء الكائنات، Builder يبني كائنات معقدة خطوة بخطوة، و Singleton يضمن نسخة واحدة. اختر النمط المناسب بناءً على احتياجاتك: Factory للتنويعات، Builder للتعقيد، Singleton للنسخة الواحدة.

---

**Next:** Explore each pattern in detail:
1. [Factory Pattern →](./01-Factory-Pattern.md)
2. [Builder Pattern →](./02-Builder-Pattern.md)
3. [Singleton Pattern →](./03-Singleton-Pattern.md)
