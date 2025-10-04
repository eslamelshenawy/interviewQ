# مبادئ SOLID في Java

## المحتويات
1. [مقدمة](#مقدمة)
2. [Single Responsibility Principle](#1-single-responsibility-principle)
3. [Open/Closed Principle](#2-openclosed-principle)
4. [Liskov Substitution Principle](#3-liskov-substitution-principle)
5. [Interface Segregation Principle](#4-interface-segregation-principle)
6. [Dependency Inversion Principle](#5-dependency-inversion-principle)
7. [ملخص](#ملخص)

---

## مقدمة

**SOLID** هي خمسة مبادئ أساسية للبرمجة الكائنية تساعد في كتابة كود:
- سهل الصيانة
- قابل للتوسع
- سهل الفهم
- قابل لإعادة الاستخدام

---

## 1. Single Responsibility Principle
### مبدأ المسؤولية الوحيدة

**التعريف:**
كل كلاس يجب أن يكون له مسؤولية واحدة فقط، أي سبب واحد فقط للتغيير.

### ❌ مثال خاطئ:

```java
// الكلاس له مسؤوليات متعددة
class User {
    private String name;
    private String email;

    // مسؤولية 1: إدارة بيانات المستخدم
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    // مسؤولية 2: حفظ البيانات في قاعدة البيانات
    public void saveToDatabase() {
        // اتصال بقاعدة البيانات
        System.out.println("Saving user to database...");
    }

    // مسؤولية 3: إرسال البريد الإلكتروني
    public void sendEmail(String message) {
        System.out.println("Sending email to " + email);
    }

    // مسؤولية 4: توليد التقارير
    public void generateReport() {
        System.out.println("Generating report for " + name);
    }
}
```

**المشكلة:**
- صعوبة الصيانة
- تغيير في طريقة الحفظ يتطلب تعديل كلاس User
- تغيير في طريقة إرسال البريد يتطلب تعديل كلاس User

### ✅ مثال صحيح:

```java
// 1. كلاس User - مسؤول فقط عن بيانات المستخدم
class User {
    private String name;
    private String email;

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}

// 2. كلاس UserRepository - مسؤول فقط عن الحفظ
class UserRepository {
    public void save(User user) {
        System.out.println("Saving " + user.getName() + " to database");
        // منطق الحفظ في قاعدة البيانات
    }

    public User findById(int id) {
        // منطق البحث
        return null;
    }
}

// 3. كلاس EmailService - مسؤول فقط عن البريد الإلكتروني
class EmailService {
    public void sendEmail(User user, String message) {
        System.out.println("Sending email to " + user.getEmail());
        System.out.println("Message: " + message);
    }
}

// 4. كلاس ReportGenerator - مسؤول فقط عن التقارير
class ReportGenerator {
    public void generateUserReport(User user) {
        System.out.println("Generating report for " + user.getName());
        // منطق توليد التقرير
    }
}

// الاستخدام
public class SRPExample {
    public static void main(String[] args) {
        User user = new User("Ahmed", "ahmed@example.com");

        UserRepository repository = new UserRepository();
        repository.save(user);

        EmailService emailService = new EmailService();
        emailService.sendEmail(user, "Welcome!");

        ReportGenerator reportGenerator = new ReportGenerator();
        reportGenerator.generateUserReport(user);
    }
}
```

**الفوائد:**
- كل كلاس له مسؤولية واحدة واضحة
- سهولة الاختبار
- سهولة التعديل بدون تأثير على الكلاسات الأخرى

---

## 2. Open/Closed Principle
### مبدأ المفتوح/المغلق

**التعريف:**
الكلاسات يجب أن تكون **مفتوحة للتوسع** (Open for Extension) و**مغلقة للتعديل** (Closed for Modification).

### ❌ مثال خاطئ:

```java
class AreaCalculator {
    public double calculateArea(Object shape) {
        double area = 0;

        if (shape instanceof Circle) {
            Circle circle = (Circle) shape;
            area = Math.PI * circle.getRadius() * circle.getRadius();
        } else if (shape instanceof Rectangle) {
            Rectangle rectangle = (Rectangle) shape;
            area = rectangle.getWidth() * rectangle.getHeight();
        } else if (shape instanceof Triangle) {
            // إضافة شكل جديد تتطلب تعديل هذا الكلاس!
            Triangle triangle = (Triangle) shape;
            area = 0.5 * triangle.getBase() * triangle.getHeight();
        }

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

**المشكلة:**
- كل شكل جديد يتطلب تعديل كلاس AreaCalculator
- خطر كسر الكود الموجود
- انتهاك لمبدأ المفتوح/المغلق

### ✅ مثال صحيح:

```java
// Interface للأشكال
interface Shape {
    double calculateArea();
}

// كل شكل ينفذ طريقته الخاصة
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

// يمكن إضافة أشكال جديدة بدون تعديل هذا الكلاس
class AreaCalculator {
    public double calculateTotalArea(List<Shape> shapes) {
        double totalArea = 0;
        for (Shape shape : shapes) {
            totalArea += shape.calculateArea();
        }
        return totalArea;
    }
}

// الاستخدام
public class OCPExample {
    public static void main(String[] args) {
        List<Shape> shapes = new ArrayList<>();
        shapes.add(new Circle(5));
        shapes.add(new Rectangle(4, 6));
        shapes.add(new Triangle(3, 8));

        AreaCalculator calculator = new AreaCalculator();
        double total = calculator.calculateTotalArea(shapes);
        System.out.println("Total Area: " + total);

        // يمكن إضافة شكل جديد بدون تعديل AreaCalculator
        shapes.add(new Pentagon(5, 3)); // شكل جديد
    }
}

// شكل جديد - لا يتطلب تعديل الكلاسات الموجودة
class Pentagon implements Shape {
    private double side;
    private double apothem;

    public Pentagon(double side, double apothem) {
        this.side = side;
        this.apothem = apothem;
    }

    @Override
    public double calculateArea() {
        return 2.5 * side * apothem;
    }
}
```

**الفوائد:**
- إضافة features جديدة بدون تعديل الكود القديم
- تقليل احتمالية الأخطاء
- كود أكثر مرونة

---

## 3. Liskov Substitution Principle
### مبدأ استبدال ليسكوف

**التعريف:**
الكائنات من الكلاس الفرعي يجب أن تستبدل الكائنات من الكلاس الأساسي بدون كسر وظيفة البرنامج.

### ❌ مثال خاطئ:

```java
class Rectangle {
    protected int width;
    protected int height;

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getArea() {
        return width * height;
    }
}

// Square يخالف سلوك Rectangle
class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width; // مشكلة! يغير الارتفاع أيضاً
    }

    @Override
    public void setHeight(int height) {
        this.width = height;
        this.height = height; // مشكلة! يغير العرض أيضاً
    }
}

// المشكلة تظهر هنا:
public class LSPProblem {
    public static void main(String[] args) {
        Rectangle rect = new Rectangle();
        rect.setWidth(5);
        rect.setHeight(10);
        System.out.println("Area: " + rect.getArea()); // 50 ✓

        // استبدال بـ Square
        Rectangle square = new Square();
        square.setWidth(5);
        square.setHeight(10);
        System.out.println("Area: " + square.getArea()); // 100 ❌
        // المتوقع 50 لكن الناتج 100!
    }
}
```

**المشكلة:**
- Square لا يمكن أن يحل محل Rectangle بشكل صحيح
- سلوك غير متوقع عند الاستبدال

### ✅ مثال صحيح:

```java
// استخدام interface بدلاً من الوراثة
interface Shape {
    double getArea();
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
    public double getArea() {
        return width * height;
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
    public double getArea() {
        return side * side;
    }
}

// الاستخدام
public class LSPSolution {
    public static void printArea(Shape shape) {
        System.out.println("Area: " + shape.getArea());
    }

    public static void main(String[] args) {
        Shape rect = new Rectangle(5, 10);
        Shape square = new Square(5);

        printArea(rect);    // 50 ✓
        printArea(square);  // 25 ✓

        // كل شكل يعمل بشكل صحيح
    }
}
```

**مثال آخر:**

```java
// ❌ خطأ
class Bird {
    public void fly() {
        System.out.println("Flying...");
    }
}

class Penguin extends Bird {
    @Override
    public void fly() {
        throw new UnsupportedOperationException("Penguins can't fly!");
        // انتهاك لمبدأ Liskov!
    }
}

// ✅ صحيح
interface Bird {
    void eat();
}

interface FlyingBird extends Bird {
    void fly();
}

class Sparrow implements FlyingBird {
    public void eat() {
        System.out.println("Eating...");
    }

    public void fly() {
        System.out.println("Flying...");
    }
}

class Penguin implements Bird {
    public void eat() {
        System.out.println("Eating...");
    }

    public void swim() {
        System.out.println("Swimming...");
    }
}
```

**الفوائد:**
- سلوك متوقع عند استخدام الوراثة
- استبدال آمن للكائنات
- كود أكثر موثوقية

---

## 4. Interface Segregation Principle
### مبدأ فصل الواجهات

**التعريف:**
لا تجبر الكلاسات على تنفيذ methods لا تحتاجها. من الأفضل عدة interfaces صغيرة من interface واحد كبير.

### ❌ مثال خاطئ:

```java
// Interface كبير جداً
interface Worker {
    void work();
    void eat();
    void sleep();
    void getPaid();
}

// الإنسان ينفذ كل الـ methods
class Human implements Worker {
    public void work() {
        System.out.println("Human working...");
    }

    public void eat() {
        System.out.println("Human eating...");
    }

    public void sleep() {
        System.out.println("Human sleeping...");
    }

    public void getPaid() {
        System.out.println("Human getting paid...");
    }
}

// الروبوت مجبر على تنفيذ methods لا يحتاجها!
class Robot implements Worker {
    public void work() {
        System.out.println("Robot working...");
    }

    public void eat() {
        // الروبوت لا يأكل!
        throw new UnsupportedOperationException();
    }

    public void sleep() {
        // الروبوت لا ينام!
        throw new UnsupportedOperationException();
    }

    public void getPaid() {
        // الروبوت لا يتقاضى راتب!
        throw new UnsupportedOperationException();
    }
}
```

**المشكلة:**
- الروبوت مجبر على تنفيذ methods لا معنى لها
- كود غير نظيف
- exceptions غير ضرورية

### ✅ مثال صحيح:

```java
// تقسيم الـ interface الكبير إلى interfaces صغيرة
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

interface Sleepable {
    void sleep();
}

interface Payable {
    void getPaid();
}

// الإنسان ينفذ كل الـ interfaces
class Human implements Workable, Eatable, Sleepable, Payable {
    public void work() {
        System.out.println("Human working...");
    }

    public void eat() {
        System.out.println("Human eating...");
    }

    public void sleep() {
        System.out.println("Human sleeping...");
    }

    public void getPaid() {
        System.out.println("Human getting paid...");
    }
}

// الروبوت ينفذ فقط ما يحتاجه
class Robot implements Workable {
    public void work() {
        System.out.println("Robot working 24/7...");
    }
}

// مثال آخر: موظف بدوام جزئي
class PartTimeEmployee implements Workable, Payable {
    public void work() {
        System.out.println("Working part-time...");
    }

    public void getPaid() {
        System.out.println("Getting hourly payment...");
    }
}

// الاستخدام
public class ISPExample {
    public static void main(String[] args) {
        Workable human = new Human();
        Workable robot = new Robot();

        human.work();
        robot.work();

        // كل كلاس ينفذ فقط ما يحتاجه
    }
}
```

**مثال عملي - نظام طباعة:**

```java
// ❌ خطأ
interface MultiFunctionDevice {
    void print();
    void scan();
    void fax();
    void photocopy();
}

class AllInOnePrinter implements MultiFunctionDevice {
    public void print() { System.out.println("Printing..."); }
    public void scan() { System.out.println("Scanning..."); }
    public void fax() { System.out.println("Faxing..."); }
    public void photocopy() { System.out.println("Photocopying..."); }
}

class SimplePrinter implements MultiFunctionDevice {
    public void print() { System.out.println("Printing..."); }

    // مجبر على تنفيذ methods لا يدعمها!
    public void scan() { throw new UnsupportedOperationException(); }
    public void fax() { throw new UnsupportedOperationException(); }
    public void photocopy() { throw new UnsupportedOperationException(); }
}

// ✅ صحيح
interface Printer {
    void print();
}

interface Scanner {
    void scan();
}

interface Fax {
    void fax();
}

interface Photocopier {
    void photocopy();
}

class AllInOnePrinter implements Printer, Scanner, Fax, Photocopier {
    public void print() { System.out.println("Printing..."); }
    public void scan() { System.out.println("Scanning..."); }
    public void fax() { System.out.println("Faxing..."); }
    public void photocopy() { System.out.println("Photocopying..."); }
}

class SimplePrinter implements Printer {
    public void print() { System.out.println("Printing..."); }
}

class ScannerPrinter implements Printer, Scanner {
    public void print() { System.out.println("Printing..."); }
    public void scan() { System.out.println("Scanning..."); }
}
```

**الفوائد:**
- كل كلاس ينفذ فقط ما يحتاجه
- interfaces نظيفة ومتخصصة
- سهولة الصيانة

---

## 5. Dependency Inversion Principle
### مبدأ عكس الاعتمادية

**التعريف:**
- الكلاسات عالية المستوى يجب ألا تعتمد على الكلاسات منخفضة المستوى. كلاهما يجب أن يعتمد على Abstractions.
- Abstractions يجب ألا تعتمد على التفاصيل. التفاصيل يجب أن تعتمد على Abstractions.

### ❌ مثال خاطئ:

```java
// اعتماد مباشر على concrete classes
class MySQLDatabase {
    public void save(String data) {
        System.out.println("Saving to MySQL: " + data);
    }
}

// UserService يعتمد مباشرة على MySQLDatabase
class UserService {
    private MySQLDatabase database;

    public UserService() {
        this.database = new MySQLDatabase(); // tight coupling!
    }

    public void saveUser(String userData) {
        database.save(userData);
    }
}

// المشكلة: لو أردنا التغيير إلى MongoDB؟
class MongoDB {
    public void insert(String data) {
        System.out.println("Inserting to MongoDB: " + data);
    }
}

// سنحتاج لتعديل UserService بالكامل!
```

**المشكلة:**
- ارتباط شديد (Tight Coupling)
- صعوبة تغيير قاعدة البيانات
- صعوبة الاختبار
- انتهاك مبدأ Open/Closed

### ✅ مثال صحيح:

```java
// 1. إنشاء Abstraction (Interface)
interface Database {
    void save(String data);
}

// 2. التفاصيل تعتمد على الـ Abstraction
class MySQLDatabase implements Database {
    @Override
    public void save(String data) {
        System.out.println("Saving to MySQL: " + data);
    }
}

class MongoDB implements Database {
    @Override
    public void save(String data) {
        System.out.println("Saving to MongoDB: " + data);
    }
}

class PostgreSQLDatabase implements Database {
    @Override
    public void save(String data) {
        System.out.println("Saving to PostgreSQL: " + data);
    }
}

// 3. UserService يعتمد على الـ Abstraction
class UserService {
    private Database database;

    // Dependency Injection عبر Constructor
    public UserService(Database database) {
        this.database = database;
    }

    public void saveUser(String userData) {
        database.save(userData);
    }
}

// الاستخدام
public class DIPExample {
    public static void main(String[] args) {
        // يمكن التبديل بسهولة بين قواعد البيانات
        Database mysql = new MySQLDatabase();
        UserService service1 = new UserService(mysql);
        service1.saveUser("Ahmed");

        Database mongo = new MongoDB();
        UserService service2 = new UserService(mongo);
        service2.saveUser("Fatima");

        Database postgres = new PostgreSQLDatabase();
        UserService service3 = new UserService(postgres);
        service3.saveUser("Omar");
    }
}
```

**مثال عملي - نظام إشعارات:**

```java
// ❌ خطأ - اعتماد مباشر
class EmailService {
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

class NotificationManager {
    private EmailService emailService = new EmailService();

    public void notify(String message) {
        emailService.send(message);
    }
}

// ✅ صحيح - استخدام Abstraction
interface MessageService {
    void send(String message);
}

class EmailService implements MessageService {
    @Override
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

class SMSService implements MessageService {
    @Override
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}

class PushNotificationService implements MessageService {
    @Override
    public void send(String message) {
        System.out.println("Push Notification: " + message);
    }
}

class NotificationManager {
    private List<MessageService> services;

    public NotificationManager(List<MessageService> services) {
        this.services = services;
    }

    public void notifyAll(String message) {
        for (MessageService service : services) {
            service.send(message);
        }
    }
}

// الاستخدام
public class NotificationExample {
    public static void main(String[] args) {
        List<MessageService> services = new ArrayList<>();
        services.add(new EmailService());
        services.add(new SMSService());
        services.add(new PushNotificationService());

        NotificationManager manager = new NotificationManager(services);
        manager.notifyAll("Welcome to our app!");
    }
}
```

**أنواع Dependency Injection:**

```java
interface Logger {
    void log(String message);
}

class ConsoleLogger implements Logger {
    public void log(String message) {
        System.out.println("LOG: " + message);
    }
}

class FileLogger implements Logger {
    public void log(String message) {
        System.out.println("Writing to file: " + message);
    }
}

class UserManager {
    private Logger logger;

    // 1. Constructor Injection (الأفضل)
    public UserManager(Logger logger) {
        this.logger = logger;
    }

    // 2. Setter Injection
    public void setLogger(Logger logger) {
        this.logger = logger;
    }

    // 3. Method Injection
    public void createUser(String name, Logger logger) {
        logger.log("Creating user: " + name);
    }

    public void deleteUser(String name) {
        logger.log("Deleting user: " + name);
    }
}

// الاستخدام
public class DIExample {
    public static void main(String[] args) {
        // Constructor Injection
        Logger consoleLogger = new ConsoleLogger();
        UserManager manager1 = new UserManager(consoleLogger);
        manager1.deleteUser("Ahmed");

        // Setter Injection
        UserManager manager2 = new UserManager(consoleLogger);
        manager2.setLogger(new FileLogger());
        manager2.deleteUser("Sara");

        // Method Injection
        manager1.createUser("Ali", new FileLogger());
    }
}
```

**الفوائد:**
- ارتباط ضعيف (Loose Coupling)
- سهولة الاختبار (Mock Objects)
- سهولة تبديل الـ implementations
- كود أكثر مرونة وقابل للتوسع

---

## ملخص

| المبدأ | الاختصار | الوصف | الفائدة الرئيسية |
|--------|---------|-------|------------------|
| **Single Responsibility** | **S** | كلاس واحد = مسؤولية واحدة | سهولة الصيانة |
| **Open/Closed** | **O** | مفتوح للتوسع، مغلق للتعديل | إضافة features بدون كسر الكود |
| **Liskov Substitution** | **L** | الكلاس الفرعي يستبدل الأساسي | استبدال آمن |
| **Interface Segregation** | **I** | interfaces صغيرة ومتخصصة | عدم إجبار الكلاسات على methods غير مفيدة |
| **Dependency Inversion** | **D** | الاعتماد على Abstractions | كود مرن وقابل للاختبار |

### لماذا نستخدم SOLID؟

✅ **كود أسهل في الصيانة**
✅ **كود قابل للتوسع**
✅ **تقليل الأخطاء**
✅ **سهولة الاختبار**
✅ **إعادة استخدام الكود**
✅ **فريق العمل يفهم الكود بسهولة**

### نصائح عملية:

1. **لا تطبق كل المبادئ مرة واحدة** - ابدأ بـ SRP و DIP
2. **لا تبالغ** - SOLID ليس dogma، استخدمه بحكمة
3. **اكتب الكود أولاً ثم refactor** - لا تحاول الكمال من البداية
4. **استخدم Design Patterns** - تطبق مبادئ SOLID بشكل طبيعي
5. **اختبر الكود** - Unit tests تساعدك على تطبيق SOLID

---

**المصادر:**
- Clean Code by Robert C. Martin
- SOLID Principles of Object-Oriented Design

**حظاً موفقاً في تطبيق SOLID! 🚀**
