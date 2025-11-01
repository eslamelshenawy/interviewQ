# Open/Closed Principle (OCP) | مبدأ المفتوح/المغلق

## Definition | التعريف

**English:**
> "Software entities (classes, modules, functions) should be open for extension but closed for modification."

**العربية:**
> "الكيانات البرمجية (الأصناف، الوحدات، الدوال) يجب أن تكون مفتوحة للتوسيع ولكن مغلقة للتعديل."

**Coined by:** Bertrand Meyer in 1988

---

## The Problem | المشكلة

When you need to add new functionality:
- Modifying existing code risks breaking working features
- Every change requires retesting everything
- Hard to maintain backward compatibility
- Increases coupling between components

عندما تحتاج إلى إضافة وظائف جديدة:
- تعديل الكود الموجود يخاطر بكسر الميزات العاملة
- كل تغيير يتطلب إعادة اختبار كل شيء
- صعوبة الحفاظ على التوافق
- يزيد الترابط بين المكونات

---

## Bad Example | مثال سيء

```java
// ❌ Violates OCP - Must modify code to add new shapes
class AreaCalculator {
    public double calculateArea(Object shape) {
        double area = 0;

        if (shape instanceof Circle) {
            Circle circle = (Circle) shape;
            area = Math.PI * circle.getRadius() * circle.getRadius();
        }
        else if (shape instanceof Rectangle) {
            Rectangle rect = (Rectangle) shape;
            area = rect.getWidth() * rect.getHeight();
        }
        else if (shape instanceof Triangle) {
            Triangle triangle = (Triangle) shape;
            area = 0.5 * triangle.getBase() * triangle.getHeight();
        }
        // Every new shape requires modifying this method!

        return area;
    }
}

class Circle {
    private double radius;
    public Circle(double radius) { this.radius = radius; }
    public double getRadius() { return radius; }
}

class Rectangle {
    private double width, height;
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    public double getWidth() { return width; }
    public double getHeight() { return height; }
}
```

**Problems | المشاكل:**
- ❌ Adding a new shape requires modifying AreaCalculator
- ❌ Risk of breaking existing calculations
- ❌ Violates Open/Closed Principle
- ❌ High coupling with specific shape classes
- ❌ Not scalable

---

## Good Example | مثال جيد

```java
// ✅ Follows OCP - Can add new shapes without modifying existing code

// Abstract interface that all shapes must implement
interface Shape {
    double calculateArea();
}

// Each shape knows how to calculate its own area
class Circle implements Shape {
    private double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

class Rectangle implements Shape {
    private double width;
    private double height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public double calculateArea() {
        return width * height;
    }
}

class Triangle implements Shape {
    private double base;
    private double height;

    public Triangle(double base, double height) {
        this.base = base;
        this.height = height;
    }

    @Override
    public double calculateArea() {
        return 0.5 * base * height;
    }
}

// NEW: Adding a new shape WITHOUT modifying existing code
class Pentagon implements Shape {
    private double side;

    public Pentagon(double side) {
        this.side = side;
    }

    @Override
    public double calculateArea() {
        return (5 * side * side) / (4 * Math.tan(Math.PI / 5));
    }
}

// Calculator doesn't need to know about specific shapes
class AreaCalculator {
    public double calculateTotalArea(List<Shape> shapes) {
        double totalArea = 0;
        for (Shape shape : shapes) {
            totalArea += shape.calculateArea();
        }
        return totalArea;
    }

    // Can also calculate individual shape without modification
    public double calculateArea(Shape shape) {
        return shape.calculateArea();
    }
}

// Usage
class Demo {
    public static void main(String[] args) {
        List<Shape> shapes = Arrays.asList(
            new Circle(5),
            new Rectangle(4, 6),
            new Triangle(3, 4),
            new Pentagon(5)  // New shape added seamlessly!
        );

        AreaCalculator calculator = new AreaCalculator();
        double total = calculator.calculateTotalArea(shapes);
        System.out.println("Total Area: " + total);
    }
}
```

**Benefits | الفوائد:**
- ✅ Adding new shapes requires no changes to existing code
- ✅ No risk of breaking existing functionality
- ✅ Follows Open/Closed Principle
- ✅ Low coupling through abstraction
- ✅ Highly scalable

---

## Real-World Examples | أمثلة من الواقع

### Example 1: Payment Processing

```java
// ❌ Bad: Must modify code for each payment method
class PaymentProcessor {
    public void processPayment(String type, double amount) {
        if (type.equals("CREDIT_CARD")) {
            // Process credit card
        } else if (type.equals("PAYPAL")) {
            // Process PayPal
        } else if (type.equals("BITCOIN")) {
            // Process Bitcoin
        }
        // Adding new method requires modifying this class!
    }
}

// ✅ Good: Open for extension
interface PaymentMethod {
    void processPayment(double amount);
}

class CreditCardPayment implements PaymentMethod {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing $" + amount + " via Credit Card");
    }
}

class PayPalPayment implements PaymentMethod {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing $" + amount + " via PayPal");
    }
}

class BitcoinPayment implements PaymentMethod {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing $" + amount + " via Bitcoin");
    }
}

// New payment method - no modification needed!
class ApplePayPayment implements PaymentMethod {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing $" + amount + " via Apple Pay");
    }
}

class PaymentProcessor {
    public void process(PaymentMethod method, double amount) {
        method.processPayment(amount);
    }
}
```

### Example 2: Notification System

```java
// ✅ Good: Notification system following OCP
interface NotificationSender {
    void send(String message, String recipient);
}

class EmailNotification implements NotificationSender {
    @Override
    public void send(String message, String recipient) {
        System.out.println("Sending email to " + recipient + ": " + message);
    }
}

class SMSNotification implements NotificationSender {
    @Override
    public void send(String message, String recipient) {
        System.out.println("Sending SMS to " + recipient + ": " + message);
    }
}

class PushNotification implements NotificationSender {
    @Override
    public void send(String message, String recipient) {
        System.out.println("Sending push to " + recipient + ": " + message);
    }
}

// Easy to add new notification types
class SlackNotification implements NotificationSender {
    @Override
    public void send(String message, String recipient) {
        System.out.println("Sending Slack message to " + recipient + ": " + message);
    }
}

class NotificationService {
    private List<NotificationSender> senders = new ArrayList<>();

    public void addSender(NotificationSender sender) {
        senders.add(sender);
    }

    public void notifyAll(String message, String recipient) {
        for (NotificationSender sender : senders) {
            sender.send(message, recipient);
        }
    }
}
```

### Example 3: Discount Calculation

```java
// ✅ Good: Discount system following OCP
interface DiscountStrategy {
    double applyDiscount(double price);
}

class NoDiscount implements DiscountStrategy {
    @Override
    public double applyDiscount(double price) {
        return price;
    }
}

class PercentageDiscount implements DiscountStrategy {
    private double percentage;

    public PercentageDiscount(double percentage) {
        this.percentage = percentage;
    }

    @Override
    public double applyDiscount(double price) {
        return price * (1 - percentage / 100);
    }
}

class FixedAmountDiscount implements DiscountStrategy {
    private double amount;

    public FixedAmountDiscount(double amount) {
        this.amount = amount;
    }

    @Override
    public double applyDiscount(double price) {
        return Math.max(0, price - amount);
    }
}

// New: Seasonal discount
class SeasonalDiscount implements DiscountStrategy {
    @Override
    public double applyDiscount(double price) {
        LocalDate now = LocalDate.now();
        if (now.getMonthValue() == 12) {  // December
            return price * 0.8;  // 20% off
        }
        return price;
    }
}

class ShoppingCart {
    private DiscountStrategy discountStrategy;

    public void setDiscountStrategy(DiscountStrategy strategy) {
        this.discountStrategy = strategy;
    }

    public double calculateTotal(double price) {
        return discountStrategy.applyDiscount(price);
    }
}
```

---

## Techniques for Achieving OCP | تقنيات لتحقيق OCP

### 1. Abstraction (Interfaces/Abstract Classes)
Use interfaces or abstract classes to define contracts that can be extended.

```java
interface DataExporter {
    void export(Data data);
}

class PDFExporter implements DataExporter {
    public void export(Data data) { /* PDF logic */ }
}

class ExcelExporter implements DataExporter {
    public void export(Data data) { /* Excel logic */ }
}
```

### 2. Inheritance
Extend base classes to add new functionality.

```java
abstract class Report {
    abstract void generate();

    public void print() {
        // Common printing logic
    }
}

class SalesReport extends Report {
    void generate() { /* Sales logic */ }
}

class InventoryReport extends Report {
    void generate() { /* Inventory logic */ }
}
```

### 3. Composition
Use composition to combine behaviors.

```java
class Logger {
    private List<LogHandler> handlers;

    public void log(String message) {
        for (LogHandler handler : handlers) {
            handler.handle(message);
        }
    }
}
```

### 4. Strategy Pattern
Encapsulate algorithms that can be swapped.

```java
interface SortingStrategy {
    void sort(int[] array);
}

class QuickSort implements SortingStrategy { }
class MergeSort implements SortingStrategy { }
```

---

## Benefits of OCP | فوائد مبدأ المفتوح/المغلق

### English

1. **Reduced Risk**
   - Existing code remains untouched
   - Less chance of introducing bugs
   - Existing tests remain valid

2. **Easier Maintenance**
   - New features added through extension
   - Clear separation of concerns
   - Minimal impact on existing code

3. **Better Testability**
   - New features can be tested independently
   - No need to retest existing functionality
   - Easier to mock and stub

4. **Improved Flexibility**
   - Easy to add new behaviors
   - Can plug in different implementations
   - Runtime flexibility

5. **Scalability**
   - System grows without becoming brittle
   - New features don't complicate existing code
   - Better long-term maintainability

### العربية

1. **تقليل المخاطر**
   - الكود الموجود يبقى دون تغيير
   - فرصة أقل لإدخال أخطاء
   - الاختبارات الموجودة تبقى صالحة

2. **صيانة أسهل**
   - الميزات الجديدة تضاف عبر التوسيع
   - فصل واضح للاهتمامات
   - تأثير ضئيل على الكود الموجود

---

## Common Mistakes | الأخطاء الشائعة

### Mistake #1: Over-abstraction
```java
// ❌ Too much abstraction for simple case
interface NumberAdder {
    int add(int a, int b);
}
// Sometimes a simple method is fine!
```

### Mistake #2: Premature Optimization
Don't create extensibility for features that might never be needed.

### Mistake #3: Forgetting Closed Part
```java
// ❌ Still allowing modification
class Shape {
    public double calculateArea() {
        // Base implementation that subclasses override completely
        return 0;  // This invites modification!
    }
}

// ✅ Better: Force implementation
abstract class Shape {
    public abstract double calculateArea();
}
```

---

## Interview Questions | أسئلة المقابلات

### Q1: What is the Open/Closed Principle?
**Answer:**
OCP states that software entities should be open for extension but closed for modification. This means you should be able to add new functionality without changing existing code. It's typically achieved through abstraction (interfaces/abstract classes) and polymorphism.

### Q2: How do you implement OCP?
**Answer:**
- Use interfaces or abstract classes to define contracts
- Implement concrete classes that extend the abstraction
- Use dependency injection to inject implementations
- Apply design patterns like Strategy, Template Method, or Decorator

### Q3: What's the difference between OCP and SRP?
**Answer:**
- SRP focuses on a class having one responsibility
- OCP focuses on extending functionality without modification
- SRP is about organization; OCP is about evolution
- Both work together: SRP makes OCP easier to achieve

### Q4: Can you give an example where OCP is violated?
**Answer:**
Using if-else or switch statements to handle different types is a common violation. For example, a payment processor that checks payment type strings and has different logic for each. Instead, use polymorphism with a PaymentMethod interface.

### Q5: What are the downsides of OCP?
**Answer:**
- Can lead to over-engineering if applied too early
- May create unnecessary abstraction layers
- Can make code harder to understand initially
- Requires good design foresight

---

## OCP in Design Patterns | OCP في أنماط التصميم

### Strategy Pattern
```java
// Open for new strategies, closed for modification
interface CompressionStrategy {
    byte[] compress(byte[] data);
}
```

### Decorator Pattern
```java
// Open for new decorators, closed for modification
interface Component {
    void operation();
}
class ConcreteComponent implements Component { }
class Decorator implements Component {
    private Component component;
}
```

### Factory Pattern
```java
// Open for new product types, closed for modification
interface Product { }
class Factory {
    public Product create(String type) {
        // Uses registry or reflection to be extensible
    }
}
```

---

## Best Practices | أفضل الممارسات

### English

1. **Use Abstraction Wisely**
   - Not everything needs to be extensible
   - Apply OCP where change is expected
   - Keep it simple for stable code

2. **Prefer Composition Over Inheritance**
   - More flexible extension mechanism
   - Avoids inheritance hierarchies
   - Easier to test

3. **Design for Change**
   - Identify variation points in your code
   - Create abstractions around them
   - Use dependency injection

4. **Apply YAGNI**
   - "You Aren't Gonna Need It"
   - Don't create extensibility you don't need yet
   - Refactor to OCP when patterns emerge

5. **Use Plugin Architecture**
   - Design systems that can load new functionality
   - Use service providers or registries
   - Enable runtime extensibility

### العربية

1. **استخدم التجريد بحكمة**
   - ليس كل شيء يحتاج أن يكون قابلاً للتوسيع
   - طبق OCP حيث التغيير متوقع
   - ابقه بسيطاً للكود المستقر

2. **فضل التركيب على الوراثة**
   - آلية توسيع أكثر مرونة
   - يتجنب التسلسلات الهرمية
   - أسهل للاختبار

---

## Conclusion | الخلاصة

**English:**
The Open/Closed Principle is crucial for building maintainable, scalable systems. By making your code open for extension but closed for modification, you reduce the risk of introducing bugs while adding new features. Use abstraction, polymorphism, and design patterns to achieve OCP effectively.

**العربية:**
مبدأ المفتوح/المغلق حاسم لبناء أنظمة قابلة للصيانة والتوسع. من خلال جعل الكود مفتوحاً للتوسيع ولكن مغلقاً للتعديل، تقلل من خطر إدخال الأخطاء عند إضافة ميزات جديدة. استخدم التجريد والتعددية وأنماط التصميم لتحقيق OCP بفعالية.

---

**Previous:** [← Single Responsibility Principle](./01-Single-Responsibility-Principle.md) | **Next:** [Liskov Substitution Principle →](./03-Liskov-Substitution-Principle.md)
