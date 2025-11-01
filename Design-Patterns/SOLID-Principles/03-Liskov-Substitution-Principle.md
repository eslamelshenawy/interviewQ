# Liskov Substitution Principle (LSP) | مبدأ الاستبدال لـ Liskov

## Definition | التعريف

**English:**
> "Objects of a superclass should be replaceable with objects of a subclass without breaking the application or altering its correctness."

**العربية:**
> "يجب أن تكون كائنات الصنف الأساسي قابلة للاستبدال بكائنات الصنف الفرعي دون كسر التطبيق أو تغيير صحته."

**Coined by:** Barbara Liskov in 1987

**Simple Version:**
"If S is a subtype of T, then objects of type T may be replaced with objects of type S without altering the program's behavior."

---

## The Problem | المشكلة

When subclasses don't properly extend their parent class:
- Unexpected behavior at runtime
- Need to check types with `instanceof`
- Violates polymorphism principles
- Breaks abstraction

عندما لا تقوم الأصناف الفرعية بتوسيع الصنف الأساسي بشكل صحيح:
- سلوك غير متوقع أثناء التشغيل
- حاجة لفحص الأنواع باستخدام `instanceof`
- ينتهك مبادئ التعددية
- يكسر التجريد

---

## Bad Example | مثال سيء

```java
// ❌ Violates LSP - Rectangle and Square relationship
class Rectangle {
    protected int width;
    protected int height;

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getWidth() {
        return width;
    }

    public int getHeight() {
        return height;
    }

    public int calculateArea() {
        return width * height;
    }
}

// Square IS-A Rectangle? Seems logical but...
class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width;  // Keep it square!
    }

    @Override
    public void setHeight(int height) {
        this.width = height;   // Keep it square!
        this.height = height;
    }
}

// Usage - This breaks!
class Demo {
    public static void main(String[] args) {
        Rectangle rect = new Rectangle();
        rect.setWidth(5);
        rect.setHeight(10);
        System.out.println("Area: " + rect.calculateArea());  // 50 ✓

        // Substitute with Square - LSP says this should work!
        Rectangle square = new Square();
        square.setWidth(5);
        square.setHeight(10);  // Wait, this changes width too!
        System.out.println("Area: " + square.calculateArea());  // 100 ✗
        // Expected 50, got 100 - BROKEN!
    }

    // This function expects Rectangle behavior
    public static void resizeRectangle(Rectangle rect) {
        rect.setWidth(5);
        rect.setHeight(10);
        assert rect.calculateArea() == 50 : "Area should be 50";
        // This fails for Square! Violates LSP
    }
}
```

**Problems | المشاكل:**
- ❌ Square changes both dimensions when setting one
- ❌ Violates expectations of Rectangle
- ❌ Cannot substitute Square for Rectangle
- ❌ Breaks LSP

---

## Good Example | مثال جيد

```java
// ✅ Follows LSP - Proper abstraction
interface Shape {
    int calculateArea();
    void draw();
}

class Rectangle implements Shape {
    private int width;
    private int height;

    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    @Override
    public int calculateArea() {
        return width * height;
    }

    @Override
    public void draw() {
        System.out.println("Drawing Rectangle: " + width + "x" + height);
    }
}

class Square implements Shape {
    private int side;

    public Square(int side) {
        this.side = side;
    }

    public void setSide(int side) {
        this.side = side;
    }

    @Override
    public int calculateArea() {
        return side * side;
    }

    @Override
    public void draw() {
        System.out.println("Drawing Square: " + side + "x" + side);
    }
}

// Usage - Now both work independently
class Demo {
    public static void main(String[] args) {
        List<Shape> shapes = Arrays.asList(
            new Rectangle(5, 10),
            new Square(5)
        );

        for (Shape shape : shapes) {
            System.out.println("Area: " + shape.calculateArea());
            shape.draw();
        }
        // Both work correctly without unexpected behavior!
    }
}
```

**Benefits | الفوائد:**
- ✅ No unexpected behavior
- ✅ Clear, separate responsibilities
- ✅ Proper polymorphism
- ✅ Follows LSP

---

## Real-World Examples | أمثلة من الواقع

### Example 1: Bird Hierarchy

```java
// ❌ Bad: Violates LSP
class Bird {
    public void fly() {
        System.out.println("Flying...");
    }
}

class Sparrow extends Bird {
    // Sparrows can fly - OK!
}

class Penguin extends Bird {
    @Override
    public void fly() {
        throw new UnsupportedOperationException("Penguins can't fly!");
        // Violates LSP! Can't substitute Penguin for Bird
    }
}

// ✅ Good: Follows LSP
interface Bird {
    void eat();
    void sleep();
}

interface FlyingBird extends Bird {
    void fly();
}

interface SwimmingBird extends Bird {
    void swim();
}

class Sparrow implements FlyingBird {
    public void eat() { System.out.println("Eating seeds"); }
    public void sleep() { System.out.println("Sleeping in nest"); }
    public void fly() { System.out.println("Flying high!"); }
}

class Penguin implements SwimmingBird {
    public void eat() { System.out.println("Eating fish"); }
    public void sleep() { System.out.println("Sleeping on ice"); }
    public void swim() { System.out.println("Swimming fast!"); }
}

// Now we can use them appropriately
class BirdWatcher {
    public void watchFlyingBird(FlyingBird bird) {
        bird.fly();  // Safe! Only flying birds can be passed
    }

    public void watchSwimmingBird(SwimmingBird bird) {
        bird.swim();  // Safe! Only swimming birds can be passed
    }
}
```

### Example 2: Account Types

```java
// ❌ Bad: Violates LSP
class Account {
    protected double balance;

    public void withdraw(double amount) {
        if (balance >= amount) {
            balance -= amount;
        }
    }

    public void deposit(double amount) {
        balance += amount;
    }
}

class SavingsAccount extends Account {
    @Override
    public void withdraw(double amount) {
        throw new UnsupportedOperationException("Cannot withdraw from savings!");
        // Violates LSP!
    }
}

// ✅ Good: Follows LSP
interface Account {
    void deposit(double amount);
    double getBalance();
}

interface WithdrawableAccount extends Account {
    void withdraw(double amount);
}

class CheckingAccount implements WithdrawableAccount {
    private double balance;

    public void deposit(double amount) {
        balance += amount;
    }

    public void withdraw(double amount) {
        if (balance >= amount) {
            balance -= amount;
        }
    }

    public double getBalance() {
        return balance;
    }
}

class FixedDepositAccount implements Account {
    private double balance;
    private LocalDate maturityDate;

    public void deposit(double amount) {
        balance += amount;
    }

    public double getBalance() {
        return balance;
    }

    // No withdraw method - that's OK!
    // Only withdrawable at maturity
}
```

### Example 3: File Storage

```java
// ❌ Bad: Violates LSP
class FileStorage {
    public void save(String filename, byte[] data) {
        // Save to disk
    }

    public byte[] load(String filename) {
        // Load from disk
        return new byte[0];
    }

    public void delete(String filename) {
        // Delete from disk
    }
}

class ReadOnlyFileStorage extends FileStorage {
    @Override
    public void save(String filename, byte[] data) {
        throw new UnsupportedOperationException("Read-only storage!");
        // Violates LSP!
    }

    @Override
    public void delete(String filename) {
        throw new UnsupportedOperationException("Read-only storage!");
        // Violates LSP!
    }
}

// ✅ Good: Follows LSP
interface ReadableStorage {
    byte[] load(String filename);
    boolean exists(String filename);
}

interface WritableStorage extends ReadableStorage {
    void save(String filename, byte[] data);
    void delete(String filename);
}

class DiskStorage implements WritableStorage {
    public byte[] load(String filename) {
        // Load from disk
        return new byte[0];
    }

    public void save(String filename, byte[] data) {
        // Save to disk
    }

    public void delete(String filename) {
        // Delete from disk
    }

    public boolean exists(String filename) {
        return new File(filename).exists();
    }
}

class CDROMStorage implements ReadableStorage {
    public byte[] load(String filename) {
        // Load from CD
        return new byte[0];
    }

    public boolean exists(String filename) {
        // Check CD
        return false;
    }

    // No save/delete - CD-ROM is read-only by nature!
}
```

---

## LSP Rules | قواعد LSP

### Formal Rules | القواعد الرسمية

1. **Preconditions cannot be strengthened**
   - Subclass can't require more than parent

```java
// ❌ Bad: Strengthening precondition
class User {
    public void setAge(int age) {
        if (age < 0) throw new IllegalArgumentException();
    }
}

class AdminUser extends User {
    @Override
    public void setAge(int age) {
        // Strengthened: requires age >= 18
        if (age < 18) throw new IllegalArgumentException();
    }
}
```

2. **Postconditions cannot be weakened**
   - Subclass must provide at least what parent promises

```java
// ❌ Bad: Weakening postcondition
class Calculator {
    // Promises to return positive result
    public int add(int a, int b) {
        return Math.abs(a + b);
    }
}

class BrokenCalculator extends Calculator {
    @Override
    public int add(int a, int b) {
        // Weakened: might return negative
        return a + b;
    }
}
```

3. **Invariants must be preserved**
   - Subclass must maintain parent's invariants

```java
// ❌ Bad: Breaking invariant
class BankAccount {
    protected double balance;

    // Invariant: balance should never be negative
    public void withdraw(double amount) {
        if (balance >= amount) {
            balance -= amount;
        }
    }
}

class OverdraftAccount extends BankAccount {
    @Override
    public void withdraw(double amount) {
        balance -= amount;  // Breaks invariant! Can go negative
    }
}
```

4. **History constraint**
   - Subclass shouldn't allow state changes that parent doesn't allow

---

## How to Identify LSP Violations | كيفية تحديد انتهاكات LSP

### Red Flags | العلامات التحذيرية

**English:**
1. **Throwing exceptions in overridden methods** that parent doesn't throw
2. **Using `instanceof` checks** before calling methods
3. **Empty or no-op implementations** of parent methods
4. **Changing method signatures** or return types inappropriately
5. **Requiring type casting** after polymorphic calls
6. **Documentation says "doesn't work for..."**
7. **Methods that do nothing** or throw UnsupportedOperationException

**العربية:**
1. **رمي استثناءات في طرق محررة** لا يرميها الأب
2. **استخدام فحوصات `instanceof`** قبل استدعاء الطرق
3. **تطبيقات فارغة** لطرق الأب
4. **تغيير توقيعات الطرق** بشكل غير مناسب
5. **الحاجة لتحويل الأنواع** بعد استدعاءات متعددة
6. **التوثيق يقول "لا يعمل لـ..."**
7. **طرق لا تفعل شيئاً** أو ترمي UnsupportedOperationException

---

## Interview Questions | أسئلة المقابلات

### Q1: What is the Liskov Substitution Principle?
**Answer:**
LSP states that objects of a superclass should be replaceable with objects of a subclass without breaking the application. In other words, if class B is a subtype of class A, we should be able to use B wherever A is expected without any unexpected behavior.

### Q2: Can you explain the Rectangle-Square problem?
**Answer:**
The classic LSP violation: mathematically, a square IS-A rectangle, but in OOP, making Square extend Rectangle violates LSP because:
- Rectangle allows independent width/height changes
- Square must keep width==height
- Substituting Square for Rectangle breaks expected behavior
- Solution: Make both implement a common Shape interface

### Q3: How is LSP different from polymorphism?
**Answer:**
- Polymorphism is the ability to treat objects of different types uniformly
- LSP is about doing so *correctly* without breaking expectations
- You can have polymorphism without LSP, but it leads to bugs
- LSP ensures polymorphism works as intended

### Q4: What are the consequences of violating LSP?
**Answer:**
- Need to use instanceof checks before method calls
- Runtime exceptions and unexpected behavior
- Breaks the benefit of polymorphism
- Leads to fragile code that's hard to maintain
- Forces clients to know about specific subtypes

### Q5: How do you fix LSP violations?
**Answer:**
- Use composition instead of inheritance
- Create proper abstractions (interfaces)
- Refactor hierarchy to follow IS-A relationship correctly
- Apply "Tell, Don't Ask" principle
- Use design patterns like Strategy or State

---

## LSP and Design Patterns | LSP وأنماط التصميم

### Strategy Pattern
```java
// Follows LSP - all strategies are substitutable
interface SortStrategy {
    void sort(int[] array);
}

class QuickSort implements SortStrategy { }
class MergeSort implements SortStrategy { }
// Any strategy can be substituted
```

### Template Method Pattern
```java
// Follows LSP - all implementations follow template
abstract class DataProcessor {
    public final void process() {
        readData();
        processData();
        writeData();
    }

    protected abstract void readData();
    protected abstract void processData();
    protected abstract void writeData();
}
```

---

## Best Practices | أفضل الممارسات

### English

1. **Design by Contract**
   - Clearly define preconditions, postconditions, and invariants
   - Document expected behavior
   - Don't weaken contracts in subclasses

2. **Favor Composition Over Inheritance**
   - Inheritance creates tight coupling
   - Composition is more flexible
   - Use interfaces for contracts

3. **Use Interfaces for Contracts**
   - Define what, not how
   - Multiple implementations stay independent
   - Easier to maintain LSP

4. **Apply "Tell, Don't Ask"**
   - Don't check types before calling methods
   - Let polymorphism work
   - Trust the abstraction

5. **Test Substitutability**
   - Write tests using base types
   - Run same tests on all subtypes
   - Verify behavior matches expectations

### العربية

1. **التصميم بالعقد**
   - حدد بوضوح الشروط المسبقة واللاحقة والثوابت
   - وثق السلوك المتوقع
   - لا تضعف العقود في الأصناف الفرعية

2. **فضل التركيب على الوراثة**
   - الوراثة تخلق ترابطاً قوياً
   - التركيب أكثر مرونة
   - استخدم الواجهات للعقود

---

## Conclusion | الخلاصة

**English:**
The Liskov Substitution Principle is fundamental to proper inheritance and polymorphism. It ensures that subclasses can be used wherever their parent classes are expected without causing unexpected behavior. Follow LSP by designing proper abstractions, using composition when appropriate, and ensuring subclasses truly extend rather than alter parent behavior.

**العربية:**
مبدأ الاستبدال لـ Liskov أساسي للوراثة والتعددية الصحيحة. يضمن أن الأصناف الفرعية يمكن استخدامها حيثما تُتوقع الأصناف الأساسية دون التسبب في سلوك غير متوقع. اتبع LSP من خلال تصميم تجريدات مناسبة، واستخدام التركيب عند الاقتضاء، والتأكد من أن الأصناف الفرعية تمتد فعلاً بدلاً من تغيير سلوك الأب.

---

**Previous:** [← Open/Closed Principle](./02-Open-Closed-Principle.md) | **Next:** [Interface Segregation Principle →](./04-Interface-Segregation-Principle.md)
