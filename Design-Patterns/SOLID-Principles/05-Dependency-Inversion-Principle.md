# Dependency Inversion Principle (DIP) | مبدأ عكس الاعتمادية

## Definition | التعريف

**English:**
> "High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions."

**العربية:**
> "الوحدات عالية المستوى يجب ألا تعتمد على الوحدات منخفضة المستوى. كلاهما يجب أن يعتمد على التجريدات. التجريدات يجب ألا تعتمد على التفاصيل. التفاصيل يجب أن تعتمد على التجريدات."

**Simple Version:** "Depend on interfaces, not implementations."

---

## The Problem | المشكلة

When high-level code depends on low-level details:
- Tight coupling between modules
- Hard to test (can't mock dependencies)
- Difficult to change implementations
- Cannot reuse high-level logic
- Violates Open/Closed Principle

عندما يعتمد الكود عالي المستوى على التفاصيل منخفضة المستوى:
- ترابط قوي بين الوحدات
- صعوبة الاختبار (لا يمكن محاكاة الاعتماديات)
- صعوبة تغيير التطبيقات
- لا يمكن إعادة استخدام المنطق عالي المستوى
- ينتهك مبدأ المفتوح/المغلق

---

## Bad Example | مثال سيء

```java
// ❌ Violates DIP - High-level depends on low-level

// Low-level module
class MySQLDatabase {
    public void connect() {
        System.out.println("Connecting to MySQL...");
    }

    public void save(String data) {
        System.out.println("Saving to MySQL: " + data);
    }

    public String load(int id) {
        return "Data from MySQL";
    }
}

// High-level module depends directly on low-level MySQL
class UserService {
    private MySQLDatabase database;  // ❌ Tight coupling!

    public UserService() {
        this.database = new MySQLDatabase();  // ❌ Creates concrete class!
    }

    public void saveUser(String userData) {
        database.save(userData);
    }

    public String getUser(int id) {
        return database.load(id);
    }
}

// Problems:
// 1. Can't switch to PostgreSQL without modifying UserService
// 2. Can't test UserService without actual MySQL
// 3. UserService tightly coupled to MySQL
// 4. Hard to reuse UserService with different database
```

**Problems | المشاكل:**
- ❌ UserService (high-level) depends on MySQLDatabase (low-level)
- ❌ Cannot swap database implementation
- ❌ Cannot test without real database
- ❌ Tight coupling
- ❌ Violates DIP

---

## Good Example | مثال جيد

```java
// ✅ Follows DIP - Both depend on abstraction

// Abstraction (interface)
interface Database {
    void connect();
    void save(String data);
    String load(int id);
}

// Low-level modules implement the abstraction
class MySQLDatabase implements Database {
    @Override
    public void connect() {
        System.out.println("Connecting to MySQL...");
    }

    @Override
    public void save(String data) {
        System.out.println("Saving to MySQL: " + data);
    }

    @Override
    public String load(int id) {
        return "Data from MySQL: " + id;
    }
}

class PostgreSQLDatabase implements Database {
    @Override
    public void connect() {
        System.out.println("Connecting to PostgreSQL...");
    }

    @Override
    public void save(String data) {
        System.out.println("Saving to PostgreSQL: " + data);
    }

    @Override
    public String load(int id) {
        return "Data from PostgreSQL: " + id;
    }
}

class MongoDatabase implements Database {
    @Override
    public void connect() {
        System.out.println("Connecting to MongoDB...");
    }

    @Override
    public void save(String data) {
        System.out.println("Saving to MongoDB: " + data);
    }

    @Override
    public String load(int id) {
        return "Data from MongoDB: " + id;
    }
}

// High-level module depends on abstraction
class UserService {
    private Database database;  // ✅ Depends on interface!

    // Dependency Injection via constructor
    public UserService(Database database) {
        this.database = database;  // ✅ Injected!
    }

    public void saveUser(String userData) {
        database.save(userData);
    }

    public String getUser(int id) {
        return database.load(id);
    }
}

// Usage - Can easily swap implementations
class Application {
    public static void main(String[] args) {
        // Use MySQL
        Database mysqlDb = new MySQLDatabase();
        UserService userService1 = new UserService(mysqlDb);
        userService1.saveUser("John Doe");

        // Switch to PostgreSQL - no changes to UserService!
        Database postgresDb = new PostgreSQLDatabase();
        UserService userService2 = new UserService(postgresDb);
        userService2.saveUser("Jane Smith");

        // Or MongoDB
        Database mongoDb = new MongoDatabase();
        UserService userService3 = new UserService(mongoDb);
        userService3.saveUser("Bob Johnson");
    }
}

// Easy to test with mock
class MockDatabase implements Database {
    public void connect() { }
    public void save(String data) {
        // Mock implementation for testing
    }
    public String load(int id) {
        return "Mock data";
    }
}

class UserServiceTest {
    public void testSaveUser() {
        Database mockDb = new MockDatabase();
        UserService service = new UserService(mockDb);
        service.saveUser("Test User");
        // Easy to test!
    }
}
```

**Benefits | الفوائد:**
- ✅ UserService depends on Database interface
- ✅ Can easily swap database implementations
- ✅ Easy to test with mocks
- ✅ Loose coupling
- ✅ Follows DIP

---

## Dependency Injection (DI) | حقن الاعتمادية

DIP is often implemented using Dependency Injection:

### 1. Constructor Injection (Best)
```java
class UserService {
    private Database database;

    public UserService(Database database) {  // ✅ Best practice
        this.database = database;
    }
}
```

### 2. Setter Injection
```java
class UserService {
    private Database database;

    public void setDatabase(Database database) {
        this.database = database;
    }
}
```

### 3. Interface Injection
```java
interface DatabaseInjector {
    void injectDatabase(Database database);
}

class UserService implements DatabaseInjector {
    private Database database;

    @Override
    public void injectDatabase(Database database) {
        this.database = database;
    }
}
```

---

## Real-World Examples | أمثلة من الواقع

### Example 1: Email Service

```java
// ❌ Bad: Direct dependency on SMTP
class NotificationService {
    private SMTPEmailSender emailSender;  // Tight coupling!

    public NotificationService() {
        this.emailSender = new SMTPEmailSender();
    }

    public void notify(String message) {
        emailSender.sendEmail(message);
    }
}

// ✅ Good: Depend on abstraction
interface EmailSender {
    void send(String recipient, String message);
}

class SMTPEmailSender implements EmailSender {
    @Override
    public void send(String recipient, String message) {
        System.out.println("Sending via SMTP to " + recipient);
    }
}

class SendGridEmailSender implements EmailSender {
    @Override
    public void send(String recipient, String message) {
        System.out.println("Sending via SendGrid to " + recipient);
    }
}

class MailgunEmailSender implements EmailSender {
    @Override
    public void send(String recipient, String message) {
        System.out.println("Sending via Mailgun to " + recipient);
    }
}

class NotificationService {
    private EmailSender emailSender;  // Depends on interface

    public NotificationService(EmailSender emailSender) {
        this.emailSender = emailSender;
    }

    public void notify(String recipient, String message) {
        emailSender.send(recipient, message);
    }
}

// Easy to switch providers
NotificationService smtp = new NotificationService(new SMTPEmailSender());
NotificationService sendgrid = new NotificationService(new SendGridEmailSender());
```

### Example 2: Payment Processing

```java
// ✅ Good: Payment system following DIP
interface PaymentGateway {
    boolean processPayment(double amount, String cardNumber);
    void refund(String transactionId);
}

class StripeGateway implements PaymentGateway {
    @Override
    public boolean processPayment(double amount, String cardNumber) {
        System.out.println("Processing $" + amount + " via Stripe");
        return true;
    }

    @Override
    public void refund(String transactionId) {
        System.out.println("Refunding via Stripe: " + transactionId);
    }
}

class PayPalGateway implements PaymentGateway {
    @Override
    public boolean processPayment(double amount, String cardNumber) {
        System.out.println("Processing $" + amount + " via PayPal");
        return true;
    }

    @Override
    public void refund(String transactionId) {
        System.out.println("Refunding via PayPal: " + transactionId);
    }
}

class OrderService {
    private PaymentGateway paymentGateway;

    public OrderService(PaymentGateway gateway) {
        this.paymentGateway = gateway;
    }

    public void checkout(double amount, String cardNumber) {
        if (paymentGateway.processPayment(amount, cardNumber)) {
            System.out.println("Order completed!");
        }
    }
}

// Can switch payment providers easily
OrderService stripeOrders = new OrderService(new StripeGateway());
OrderService paypalOrders = new OrderService(new PayPalGateway());
```

### Example 3: Logging System

```java
// ✅ Good: Logging following DIP
interface Logger {
    void log(String message);
    void error(String message);
}

class FileLogger implements Logger {
    @Override
    public void log(String message) {
        System.out.println("Logging to file: " + message);
    }

    @Override
    public void error(String message) {
        System.out.println("Error to file: " + message);
    }
}

class ConsoleLogger implements Logger {
    @Override
    public void log(String message) {
        System.out.println("Console: " + message);
    }

    @Override
    public void error(String message) {
        System.err.println("Console Error: " + message);
    }
}

class CloudLogger implements Logger {
    @Override
    public void log(String message) {
        System.out.println("Logging to cloud: " + message);
    }

    @Override
    public void error(String message) {
        System.out.println("Error to cloud: " + message);
    }
}

class Application {
    private Logger logger;

    public Application(Logger logger) {
        this.logger = logger;
    }

    public void run() {
        logger.log("Application started");
        try {
            // Business logic
        } catch (Exception e) {
            logger.error("Application error: " + e.getMessage());
        }
    }
}

// Can combine multiple loggers
class CompositeLogger implements Logger {
    private List<Logger> loggers;

    public CompositeLogger(Logger... loggers) {
        this.loggers = Arrays.asList(loggers);
    }

    @Override
    public void log(String message) {
        for (Logger logger : loggers) {
            logger.log(message);
        }
    }

    @Override
    public void error(String message) {
        for (Logger logger : loggers) {
            logger.error(message);
        }
    }
}

// Log to multiple destinations
Logger multiLogger = new CompositeLogger(
    new FileLogger(),
    new ConsoleLogger(),
    new CloudLogger()
);
Application app = new Application(multiLogger);
```

---

## Benefits of DIP | فوائد مبدأ عكس الاعتمادية

### English

1. **Loose Coupling**
   - Modules independent of implementations
   - Easy to change dependencies
   - Better separation of concerns

2. **Enhanced Testability**
   - Easy to inject mocks and stubs
   - Can test without real dependencies
   - Faster unit tests

3. **Flexibility**
   - Swap implementations at runtime
   - Support multiple implementations
   - Easy to extend

4. **Reusability**
   - High-level logic can be reused
   - Not tied to specific implementations
   - Portable across projects

5. **Maintainability**
   - Changes localized to implementations
   - No ripple effects
   - Easier to understand

### العربية

1. **ترابط ضعيف**
   - الوحدات مستقلة عن التطبيقات
   - سهولة تغيير الاعتماديات
   - فصل أفضل للاهتمامات

2. **قابلية اختبار محسنة**
   - سهولة حقن المحاكيات
   - يمكن الاختبار بدون اعتماديات حقيقية
   - اختبارات وحدات أسرع

---

## DIP in Frameworks | DIP في أطر العمل

### Spring Framework
```java
@Service
class UserService {
    private final UserRepository repository;

    @Autowired  // Dependency Injection
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}

interface UserRepository extends JpaRepository<User, Long> {
    // Spring provides implementation
}
```

### ASP.NET Core
```csharp
public class UserService
{
    private readonly IUserRepository _repository;

    public UserService(IUserRepository repository)  // DI
    {
        _repository = repository;
    }
}

services.AddScoped<IUserRepository, SqlUserRepository>();
```

---

## Interview Questions | أسئلة المقابلات

### Q1: What is the Dependency Inversion Principle?
**Answer:**
DIP states that high-level modules should not depend on low-level modules; both should depend on abstractions. This means depend on interfaces or abstract classes, not concrete implementations. It enables loose coupling, easier testing, and flexibility to change implementations.

### Q2: What's the difference between DIP and Dependency Injection?
**Answer:**
- DIP is a principle: "depend on abstractions"
- Dependency Injection (DI) is a technique to implement DIP
- DIP is the "what" (the goal)
- DI is the "how" (the implementation)
- You can follow DIP without DI, but DI makes it easier

### Q3: How does DIP help with testing?
**Answer:**
DIP makes testing easier by:
- Allowing injection of mock/stub implementations
- Not requiring real databases, APIs, or external services
- Enabling fast, isolated unit tests
- Making tests independent of infrastructure

### Q4: What are the three types of Dependency Injection?
**Answer:**
1. Constructor Injection (best) - dependencies passed through constructor
2. Setter Injection - dependencies set through setter methods
3. Interface Injection - interface defines injection method

### Q5: How does DIP relate to other SOLID principles?
**Answer:**
- Works with OCP: Abstractions allow extension without modification
- Supports LSP: Implementations must be substitutable
- Complements ISP: Depend on focused interfaces
- Enables SRP: Classes focus on business logic, not dependency creation

---

## Common Mistakes | الأخطاء الشائعة

### Mistake #1: Depending on Concrete Classes
```java
// ❌ Wrong
class UserService {
    private MySQLDatabase db = new MySQLDatabase();
}

// ✅ Right
class UserService {
    private Database db;
    public UserService(Database db) {
        this.db = db;
    }
}
```

### Mistake #2: Creating Dependencies Inside Class
```java
// ❌ Wrong
class OrderService {
    private PaymentGateway gateway = new StripeGateway();
}

// ✅ Right
class OrderService {
    private PaymentGateway gateway;
    public OrderService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

### Mistake #3: Abstractions Depending on Details
```java
// ❌ Wrong
interface UserRepository {
    MySQLConnection getConnection();  // Leaks implementation!
}

// ✅ Right
interface UserRepository {
    User findById(Long id);  // Abstract!
}
```

---

## Best Practices | أفضل الممارسات

### English

1. **Prefer Constructor Injection**
   - Makes dependencies explicit
   - Enables immutability
   - Easier to test

2. **Depend on Interfaces**
   - Not abstract classes (unless needed)
   - Keep interfaces focused (ISP)
   - Use meaningful names

3. **Use Dependency Injection Containers**
   - Spring, Guice, etc.
   - Automate dependency management
   - Centralize configuration

4. **Don't Overuse**
   - Not every class needs DI
   - Simple classes can create dependencies
   - Balance pragmatism and principles

5. **Keep Abstractions Stable**
   - Don't change interfaces frequently
   - Design interfaces carefully
   - Consider future needs

### العربية

1. **فضل حقن المُنشئ**
   - يجعل الاعتماديات واضحة
   - يمكّن عدم التغيير
   - أسهل للاختبار

2. **اعتمد على الواجهات**
   - وليس الأصناف المجردة (إلا عند الحاجة)
   - اجعل الواجهات مركزة
   - استخدم أسماء ذات معنى

---

## Conclusion | الخلاصة

**English:**
The Dependency Inversion Principle is fundamental to creating flexible, testable, and maintainable systems. By depending on abstractions rather than concrete implementations, we achieve loose coupling and gain the ability to easily swap implementations. Use Dependency Injection to implement DIP effectively, and leverage DI containers in your frameworks.

**العربية:**
مبدأ عكس الاعتمادية أساسي لإنشاء أنظمة مرنة وقابلة للاختبار والصيانة. من خلال الاعتماد على التجريدات بدلاً من التطبيقات المحددة، نحقق ترابطاً ضعيفاً ونكتسب القدرة على تبديل التطبيقات بسهولة. استخدم حقن الاعتمادية لتطبيق DIP بفعالية، واستفد من حاويات DI في أطر العمل الخاصة بك.

---

**Previous:** [← Interface Segregation Principle](./04-Interface-Segregation-Principle.md) | **Home:** [SOLID Principles](./README.md)
