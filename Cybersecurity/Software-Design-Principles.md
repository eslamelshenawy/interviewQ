# Software Design Principles | مبادئ تصميم البرمجيات

## English Version

### SOLID Principles

#### 1. Single Responsibility Principle (SRP)
A class should have only one reason to change, meaning it should have only one job or responsibility.

**Example:**
```java
// Bad - Multiple responsibilities
class User {
    void saveToDatabase() { }
    void sendEmail() { }
    void generateReport() { }
}

// Good - Single responsibility
class User {
    // User data only
}
class UserRepository {
    void saveToDatabase(User user) { }
}
class EmailService {
    void sendEmail(User user) { }
}
class ReportGenerator {
    void generateReport(User user) { }
}
```

#### 2. Open/Closed Principle (OCP)
Software entities should be open for extension but closed for modification.

**Example:**
```java
// Bad - Must modify class to add new shapes
class AreaCalculator {
    double calculateArea(Object shape) {
        if (shape instanceof Circle) {
            // calculate circle area
        } else if (shape instanceof Rectangle) {
            // calculate rectangle area
        }
    }
}

// Good - Extend without modifying
interface Shape {
    double calculateArea();
}
class Circle implements Shape {
    public double calculateArea() { /* circle logic */ }
}
class Rectangle implements Shape {
    public double calculateArea() { /* rectangle logic */ }
}
```

#### 3. Liskov Substitution Principle (LSP)
Objects of a superclass should be replaceable with objects of a subclass without breaking the application.

**Example:**
```java
// Bad - Violates LSP
class Bird {
    void fly() { }
}
class Penguin extends Bird {
    @Override
    void fly() {
        throw new UnsupportedOperationException("Penguins can't fly");
    }
}

// Good - Proper inheritance
interface Bird { }
interface FlyingBird extends Bird {
    void fly();
}
class Sparrow implements FlyingBird {
    public void fly() { /* flying logic */ }
}
class Penguin implements Bird {
    // No fly method
}
```

#### 4. Interface Segregation Principle (ISP)
Clients should not be forced to depend on interfaces they don't use.

**Example:**
```java
// Bad - Fat interface
interface Worker {
    void work();
    void eat();
    void sleep();
}

// Good - Segregated interfaces
interface Workable {
    void work();
}
interface Eatable {
    void eat();
}
interface Sleepable {
    void sleep();
}
class Human implements Workable, Eatable, Sleepable { }
class Robot implements Workable { }
```

#### 5. Dependency Inversion Principle (DIP)
High-level modules should not depend on low-level modules. Both should depend on abstractions.

**Example:**
```java
// Bad - High-level depends on low-level
class MySQLDatabase {
    void save() { }
}
class UserService {
    private MySQLDatabase database = new MySQLDatabase();
}

// Good - Depend on abstraction
interface Database {
    void save();
}
class MySQLDatabase implements Database {
    public void save() { }
}
class UserService {
    private Database database;
    public UserService(Database database) {
        this.database = database;
    }
}
```

---

## Design Patterns

### Creational Patterns

#### 1. Factory Pattern
Creates objects without specifying the exact class to create.

**Example:**
```java
interface Shape {
    void draw();
}

class Circle implements Shape {
    public void draw() { System.out.println("Circle"); }
}

class Rectangle implements Shape {
    public void draw() { System.out.println("Rectangle"); }
}

class ShapeFactory {
    public Shape getShape(String shapeType) {
        if (shapeType.equals("CIRCLE")) {
            return new Circle();
        } else if (shapeType.equals("RECTANGLE")) {
            return new Rectangle();
        }
        return null;
    }
}
```

**When to use:**
- When you don't know ahead of time what class object you need
- When all potential classes are in the same subclass hierarchy
- To centralize object creation logic

#### 2. Builder Pattern
Constructs complex objects step by step.

**Example:**
```java
class User {
    private String name;
    private int age;
    private String email;

    private User(UserBuilder builder) {
        this.name = builder.name;
        this.age = builder.age;
        this.email = builder.email;
    }

    static class UserBuilder {
        private String name;
        private int age;
        private String email;

        public UserBuilder setName(String name) {
            this.name = name;
            return this;
        }

        public UserBuilder setAge(int age) {
            this.age = age;
            return this;
        }

        public UserBuilder setEmail(String email) {
            this.email = email;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }
}

// Usage
User user = new User.UserBuilder()
    .setName("Ahmed")
    .setAge(25)
    .setEmail("ahmed@example.com")
    .build();
```

**When to use:**
- When object creation requires many parameters
- When you want to make object immutable
- When construction process is complex

#### 3. Singleton Pattern
Ensures a class has only one instance and provides global access to it.

**Example:**
```java
class DatabaseConnection {
    private static DatabaseConnection instance;

    private DatabaseConnection() { }

    public static DatabaseConnection getInstance() {
        if (instance == null) {
            synchronized (DatabaseConnection.class) {
                if (instance == null) {
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance;
    }
}
```

**When to use:**
- When exactly one instance of a class is needed
- For logging, driver objects, caching, thread pools
- When creating the object is expensive

---

### Structural Patterns

#### 1. Adapter Pattern
Allows incompatible interfaces to work together.

**Example:**
```java
// Existing interface
interface MediaPlayer {
    void play(String filename);
}

// New interface
interface AdvancedMediaPlayer {
    void playVlc(String filename);
    void playMp4(String filename);
}

class VlcPlayer implements AdvancedMediaPlayer {
    public void playVlc(String filename) {
        System.out.println("Playing vlc: " + filename);
    }
    public void playMp4(String filename) { }
}

// Adapter
class MediaAdapter implements MediaPlayer {
    AdvancedMediaPlayer advancedPlayer;

    public MediaAdapter(String audioType) {
        if (audioType.equals("vlc")) {
            advancedPlayer = new VlcPlayer();
        }
    }

    public void play(String filename) {
        advancedPlayer.playVlc(filename);
    }
}
```

**When to use:**
- When you want to use an existing class with an incompatible interface
- When integrating third-party libraries
- When working with legacy code

#### 2. Facade Pattern
Provides a simplified interface to a complex subsystem.

**Example:**
```java
class CPU {
    void freeze() { }
    void jump(long position) { }
    void execute() { }
}

class Memory {
    void load(long position, byte[] data) { }
}

class HardDrive {
    byte[] read(long lba, int size) { return new byte[size]; }
}

// Facade
class ComputerFacade {
    private CPU cpu;
    private Memory memory;
    private HardDrive hardDrive;

    public ComputerFacade() {
        this.cpu = new CPU();
        this.memory = new Memory();
        this.hardDrive = new HardDrive();
    }

    public void start() {
        cpu.freeze();
        memory.load(0, hardDrive.read(0, 1024));
        cpu.jump(0);
        cpu.execute();
    }
}

// Usage
ComputerFacade computer = new ComputerFacade();
computer.start();
```

**When to use:**
- When you want to simplify a complex system
- When you want to layer your subsystems
- When there are many dependencies between clients and implementation classes

#### 3. Decorator Pattern
Adds new functionality to objects dynamically without altering their structure.

**Example:**
```java
interface Coffee {
    double getCost();
    String getDescription();
}

class SimpleCoffee implements Coffee {
    public double getCost() { return 5.0; }
    public String getDescription() { return "Simple Coffee"; }
}

abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;

    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
}

class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }

    public double getCost() {
        return coffee.getCost() + 1.5;
    }

    public String getDescription() {
        return coffee.getDescription() + ", Milk";
    }
}

// Usage
Coffee coffee = new SimpleCoffee();
coffee = new MilkDecorator(coffee);
System.out.println(coffee.getDescription() + " $" + coffee.getCost());
```

**When to use:**
- When you want to add responsibilities to objects dynamically
- When extension by subclassing is impractical
- When you need to add functionality that can be withdrawn

---

### Behavioral Patterns

#### 1. Strategy Pattern
Defines a family of algorithms and makes them interchangeable.

**Example:**
```java
interface PaymentStrategy {
    void pay(int amount);
}

class CreditCardStrategy implements PaymentStrategy {
    private String cardNumber;

    public CreditCardStrategy(String cardNumber) {
        this.cardNumber = cardNumber;
    }

    public void pay(int amount) {
        System.out.println("Paid " + amount + " using Credit Card");
    }
}

class PayPalStrategy implements PaymentStrategy {
    private String email;

    public PayPalStrategy(String email) {
        this.email = email;
    }

    public void pay(int amount) {
        System.out.println("Paid " + amount + " using PayPal");
    }
}

class ShoppingCart {
    private PaymentStrategy paymentStrategy;

    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }

    public void checkout(int amount) {
        paymentStrategy.pay(amount);
    }
}

// Usage
ShoppingCart cart = new ShoppingCart();
cart.setPaymentStrategy(new CreditCardStrategy("1234-5678-9012-3456"));
cart.checkout(100);
```

**When to use:**
- When you have multiple algorithms for a specific task
- When you want to avoid conditional statements
- When you need to switch between different algorithms at runtime

#### 2. Observer Pattern
Defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified.

**Example:**
```java
import java.util.ArrayList;
import java.util.List;

interface Observer {
    void update(String message);
}

interface Subject {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyObservers();
}

class NewsAgency implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private String news;

    public void attach(Observer observer) {
        observers.add(observer);
    }

    public void detach(Observer observer) {
        observers.remove(observer);
    }

    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(news);
        }
    }

    public void setNews(String news) {
        this.news = news;
        notifyObservers();
    }
}

class NewsChannel implements Observer {
    private String name;

    public NewsChannel(String name) {
        this.name = name;
    }

    public void update(String news) {
        System.out.println(name + " received news: " + news);
    }
}

// Usage
NewsAgency agency = new NewsAgency();
NewsChannel channel1 = new NewsChannel("CNN");
NewsChannel channel2 = new NewsChannel("BBC");

agency.attach(channel1);
agency.attach(channel2);
agency.setNews("Breaking News!");
```

**When to use:**
- When a change to one object requires changing others
- When an object should notify other objects without knowing who they are
- Event handling systems

#### 3. Command Pattern
Encapsulates a request as an object, allowing parameterization of clients with different requests.

**Example:**
```java
interface Command {
    void execute();
    void undo();
}

class Light {
    public void turnOn() {
        System.out.println("Light is ON");
    }

    public void turnOff() {
        System.out.println("Light is OFF");
    }
}

class LightOnCommand implements Command {
    private Light light;

    public LightOnCommand(Light light) {
        this.light = light;
    }

    public void execute() {
        light.turnOn();
    }

    public void undo() {
        light.turnOff();
    }
}

class RemoteControl {
    private Command command;

    public void setCommand(Command command) {
        this.command = command;
    }

    public void pressButton() {
        command.execute();
    }

    public void pressUndo() {
        command.undo();
    }
}

// Usage
Light light = new Light();
Command lightOn = new LightOnCommand(light);
RemoteControl remote = new RemoteControl();
remote.setCommand(lightOn);
remote.pressButton();
remote.pressUndo();
```

**When to use:**
- When you need to parameterize objects with operations
- When you need to queue operations, schedule their execution, or execute them remotely
- When you need to support undo/redo functionality

---

## Interview Questions

### SOLID Principles Questions

1. **What is the Single Responsibility Principle and why is it important?**
   - Each class should have only one reason to change
   - Makes code more maintainable and testable
   - Reduces coupling between components

2. **Explain the difference between Open/Closed and Liskov Substitution principles**
   - OCP: Code should be open for extension but closed for modification
   - LSP: Subtypes must be substitutable for their base types without breaking functionality

3. **How does Dependency Inversion help in testing?**
   - Allows easy mocking and stubbing of dependencies
   - Facilitates unit testing by injecting test doubles
   - Reduces coupling between modules

### Design Patterns Questions

4. **When would you use Factory pattern over Builder pattern?**
   - Factory: Simple object creation, few parameters
   - Builder: Complex objects, many optional parameters, step-by-step construction

5. **What are the disadvantages of Singleton pattern?**
   - Difficult to test (global state)
   - Can hide dependencies
   - Threading issues if not implemented correctly
   - Violates Single Responsibility Principle

6. **Explain the difference between Adapter and Facade patterns**
   - Adapter: Converts one interface to another
   - Facade: Simplifies a complex interface
   - Adapter wraps one class, Facade wraps multiple classes

7. **How does Strategy pattern differ from State pattern?**
   - Strategy: Client chooses which algorithm to use
   - State: Object changes behavior based on internal state
   - Both allow changing behavior at runtime

8. **What problems does the Observer pattern solve?**
   - Loose coupling between objects
   - One-to-many relationships
   - Event handling systems
   - Real-time updates to multiple objects

---

## النسخة العربية

### مبادئ SOLID

#### 1. مبدأ المسؤولية الواحدة (Single Responsibility Principle)
يجب أن يكون للصنف سبب واحد فقط للتغيير، أي أن يكون له مسؤولية واحدة فقط.

**مثال:**
```java
// سيء - مسؤوليات متعددة
class User {
    void saveToDatabase() { }
    void sendEmail() { }
    void generateReport() { }
}

// جيد - مسؤولية واحدة
class User {
    // بيانات المستخدم فقط
}
class UserRepository {
    void saveToDatabase(User user) { }
}
class EmailService {
    void sendEmail(User user) { }
}
class ReportGenerator {
    void generateReport(User user) { }
}
```

#### 2. مبدأ المفتوح/المغلق (Open/Closed Principle)
يجب أن تكون الكيانات البرمجية مفتوحة للتوسيع ولكن مغلقة للتعديل.

#### 3. مبدأ الاستبدال لـ Liskov (Liskov Substitution Principle)
يجب أن تكون كائنات الصنف الفرعي قابلة للاستبدال بكائنات الصنف الأساسي دون كسر التطبيق.

#### 4. مبدأ فصل الواجهات (Interface Segregation Principle)
يجب ألا يُجبر العميل على الاعتماد على واجهات لا يستخدمها.

#### 5. مبدأ عكس الاعتمادية (Dependency Inversion Principle)
الوحدات عالية المستوى يجب ألا تعتمد على الوحدات منخفضة المستوى. كلاهما يجب أن يعتمد على التجريدات.

---

### أنماط التصميم

#### الأنماط الإنشائية (Creational Patterns)

**1. نمط المصنع (Factory Pattern)**
- ينشئ الكائنات دون تحديد الصنف الدقيق للإنشاء
- يستخدم عندما لا تعرف مسبقاً نوع الكائن المطلوب

**2. نمط البناء (Builder Pattern)**
- يبني الكائنات المعقدة خطوة بخطوة
- مفيد عندما يحتاج الكائن إلى العديد من المعاملات

**3. نمط المفرد (Singleton Pattern)**
- يضمن وجود نسخة واحدة فقط من الصنف
- يوفر نقطة وصول عالمية لها

#### الأنماط الهيكلية (Structural Patterns)

**1. نمط المحول (Adapter Pattern)**
- يسمح للواجهات غير المتوافقة بالعمل معاً
- يستخدم عند دمج مكتبات خارجية

**2. نمط الواجهة (Facade Pattern)**
- يوفر واجهة مبسطة لنظام معقد
- يخفي تعقيد النظام الفرعي

**3. نمط المزخرف (Decorator Pattern)**
- يضيف وظائف جديدة للكائنات ديناميكياً
- دون تغيير بنيتها

#### الأنماط السلوكية (Behavioral Patterns)

**1. نمط الاستراتيجية (Strategy Pattern)**
- يعرّف عائلة من الخوارزميات ويجعلها قابلة للتبديل
- يستخدم لتجنب العبارات الشرطية

**2. نمط المراقب (Observer Pattern)**
- يعرّف علاقة واحد لمتعدد بين الكائنات
- عند تغيير كائن، يتم إخطار جميع التابعين له

**3. نمط الأمر (Command Pattern)**
- يغلف الطلب ككائن
- يسمح بالتراجع عن العمليات (Undo/Redo)

---

## أسئلة المقابلات الشائعة

### أسئلة عن SOLID

1. **ما هو مبدأ المسؤولية الواحدة ولماذا هو مهم؟**
   - كل صنف يجب أن يكون له سبب واحد للتغيير
   - يجعل الكود أكثر قابلية للصيانة والاختبار
   - يقلل من الترابط بين المكونات

2. **اشرح الفرق بين مبدأ المفتوح/المغلق ومبدأ Liskov**
   - المفتوح/المغلق: الكود مفتوح للتوسيع مغلق للتعديل
   - Liskov: الأصناف الفرعية يجب أن تكون قابلة للاستبدال بالأصناف الأساسية

3. **كيف يساعد مبدأ عكس الاعتمادية في الاختبار؟**
   - يسمح بسهولة استخدام Mock و Stub
   - يسهل اختبار الوحدات
   - يقلل الترابط بين الوحدات

### أسئلة عن أنماط التصميم

4. **متى تستخدم نمط Factory بدلاً من Builder؟**
   - Factory: إنشاء بسيط، معاملات قليلة
   - Builder: كائنات معقدة، معاملات اختيارية كثيرة

5. **ما هي عيوب نمط Singleton؟**
   - صعب الاختبار (حالة عالمية)
   - يمكن أن يخفي الاعتماديات
   - مشاكل في بيئة متعددة الخيوط
   - ينتهك مبدأ المسؤولية الواحدة

6. **اشرح الفرق بين نمط Adapter و Facade**
   - Adapter: يحول واجهة إلى أخرى
   - Facade: يبسط واجهة معقدة
   - Adapter يغلف صنف واحد، Facade يغلف أصناف متعددة

7. **ما الفرق بين نمط Strategy و State؟**
   - Strategy: العميل يختار الخوارزمية
   - State: الكائن يغير سلوكه بناءً على حالته الداخلية
   - كلاهما يسمح بتغيير السلوك في وقت التشغيل

8. **ما المشاكل التي يحلها نمط Observer؟**
   - ترابط ضعيف بين الكائنات
   - علاقات واحد لمتعدد
   - أنظمة معالجة الأحداث
   - تحديثات فورية لعدة كائنات

---

## Best Practices | أفضل الممارسات

### English
- Always follow SOLID principles in your design
- Choose the right pattern for the problem
- Don't overuse patterns - keep it simple
- Consider maintainability and testability
- Document your design decisions
- Use composition over inheritance when possible
- Keep interfaces small and focused

### العربية
- اتبع دائماً مبادئ SOLID في تصميمك
- اختر النمط المناسب للمشكلة
- لا تفرط في استخدام الأنماط - ابقها بسيطة
- فكر في قابلية الصيانة والاختبار
- وثق قرارات التصميم الخاصة بك
- استخدم التركيب بدلاً من الوراثة عندما يكون ممكناً
- اجعل الواجهات صغيرة ومركزة

---

## Resources | المصادر

- **Books:**
  - Design Patterns: Elements of Reusable Object-Oriented Software (Gang of Four)
  - Clean Code by Robert C. Martin
  - Head First Design Patterns

- **Online:**
  - refactoring.guru
  - sourcemaking.com
  - GitHub repositories with pattern implementations

---

## Contributing | المساهمة

Feel free to contribute by adding more patterns, examples, or translations!

لا تتردد في المساهمة بإضافة المزيد من الأنماط أو الأمثلة أو الترجمات!
