# Single Responsibility Principle (SRP) | مبدأ المسؤولية الواحدة

## Definition | التعريف

**English:**
> "A class should have only one reason to change, meaning it should have only one job or responsibility."

**العربية:**
> "يجب أن يكون للصنف سبب واحد فقط للتغيير، أي أن يكون له مهمة أو مسؤولية واحدة فقط."

---

## The Problem | المشكلة

When a class has multiple responsibilities, changes to one responsibility can affect the others, leading to:
- High coupling between unrelated functionalities
- Difficult testing
- Harder maintenance
- Code that's difficult to understand

عندما يكون للصنف مسؤوليات متعددة، التغييرات في مسؤولية واحدة يمكن أن تؤثر على الأخرى، مما يؤدي إلى:
- ترابط عالي بين وظائف غير مرتبطة
- صعوبة الاختبار
- صيانة أصعب
- كود صعب الفهم

---

## Bad Example | مثال سيء

```java
// ❌ Violates SRP - Multiple responsibilities
class User {
    private String name;
    private String email;

    // User data management
    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    // Database operations (Responsibility #1)
    public void saveToDatabase() {
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost/db");
        PreparedStatement stmt = conn.prepareStatement("INSERT INTO users VALUES (?, ?)");
        stmt.setString(1, name);
        stmt.setString(2, email);
        stmt.execute();
    }

    // Email operations (Responsibility #2)
    public void sendWelcomeEmail() {
        SMTP smtp = new SMTP("smtp.gmail.com");
        smtp.send(email, "Welcome!", "Thank you for joining us!");
    }

    // Report generation (Responsibility #3)
    public void generateReport() {
        PDFGenerator pdf = new PDFGenerator();
        pdf.addText("User Report: " + name);
        pdf.save("report.pdf");
    }

    // Validation (Responsibility #4)
    public boolean validateEmail() {
        return email.contains("@") && email.contains(".");
    }

    // Getters and setters
    public String getName() { return name; }
    public String getEmail() { return email; }
}
```

**Problems with this design | المشاكل في هذا التصميم:**
- ❌ User class has 4 different reasons to change
- ❌ Cannot test email validation without database
- ❌ Cannot reuse email service for other entities
- ❌ Hard to maintain and extend
- ❌ Tightly coupled to external services

---

## Good Example | مثال جيد

```java
// ✅ Follows SRP - Single responsibility per class

// 1. User entity - Only manages user data
class User {
    private String name;
    private String email;

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    public String getName() { return name; }
    public String getEmail() { return email; }

    public void setName(String name) { this.name = name; }
    public void setEmail(String email) { this.email = email; }
}

// 2. UserRepository - Only handles database operations
class UserRepository {
    private Connection connection;

    public UserRepository(Connection connection) {
        this.connection = connection;
    }

    public void save(User user) {
        try {
            PreparedStatement stmt = connection.prepareStatement(
                "INSERT INTO users (name, email) VALUES (?, ?)"
            );
            stmt.setString(1, user.getName());
            stmt.setString(2, user.getEmail());
            stmt.execute();
        } catch (SQLException e) {
            throw new DatabaseException("Failed to save user", e);
        }
    }

    public User findByEmail(String email) {
        // Implementation for finding user
        return null;
    }
}

// 3. EmailService - Only handles email operations
class EmailService {
    private SMTP smtpClient;

    public EmailService(SMTP smtpClient) {
        this.smtpClient = smtpClient;
    }

    public void sendWelcomeEmail(User user) {
        String subject = "Welcome!";
        String body = "Thank you for joining us, " + user.getName() + "!";
        smtpClient.send(user.getEmail(), subject, body);
    }

    public void sendPasswordResetEmail(User user, String resetLink) {
        String subject = "Password Reset";
        String body = "Click here to reset: " + resetLink;
        smtpClient.send(user.getEmail(), subject, body);
    }
}

// 4. EmailValidator - Only handles email validation
class EmailValidator {
    private static final String EMAIL_PATTERN =
        "^[A-Za-z0-9+_.-]+@(.+)\\.[A-Za-z]{2,}$";

    public boolean isValid(String email) {
        if (email == null || email.trim().isEmpty()) {
            return false;
        }
        return email.matches(EMAIL_PATTERN);
    }
}

// 5. UserReportGenerator - Only handles report generation
class UserReportGenerator {
    private PDFGenerator pdfGenerator;

    public UserReportGenerator(PDFGenerator pdfGenerator) {
        this.pdfGenerator = pdfGenerator;
    }

    public void generateUserReport(User user, String filePath) {
        pdfGenerator.startDocument();
        pdfGenerator.addTitle("User Report");
        pdfGenerator.addText("Name: " + user.getName());
        pdfGenerator.addText("Email: " + user.getEmail());
        pdfGenerator.save(filePath);
    }
}

// Usage - Coordinating class
class UserService {
    private UserRepository repository;
    private EmailService emailService;
    private EmailValidator validator;

    public UserService(UserRepository repository,
                       EmailService emailService,
                       EmailValidator validator) {
        this.repository = repository;
        this.emailService = emailService;
        this.validator = validator;
    }

    public void registerUser(String name, String email) {
        // Validate
        if (!validator.isValid(email)) {
            throw new IllegalArgumentException("Invalid email format");
        }

        // Create user
        User user = new User(name, email);

        // Save to database
        repository.save(user);

        // Send welcome email
        emailService.sendWelcomeEmail(user);
    }
}
```

**Benefits of this design | فوائد هذا التصميم:**
- ✅ Each class has a single, well-defined responsibility
- ✅ Easy to test each component independently
- ✅ Can reuse EmailService for other entities
- ✅ Easy to maintain and extend
- ✅ Loosely coupled components

---

## Real-World Examples | أمثلة من الواقع

### Example 1: E-Commerce System

```java
// ❌ Bad: Order class doing everything
class Order {
    void calculateTotal() { }
    void saveToDatabase() { }
    void sendConfirmationEmail() { }
    void generateInvoice() { }
    void processPayment() { }
}

// ✅ Good: Separated responsibilities
class Order {
    // Only order data
}
class OrderCalculator {
    Money calculateTotal(Order order) { }
}
class OrderRepository {
    void save(Order order) { }
}
class OrderNotificationService {
    void sendConfirmation(Order order) { }
}
class InvoiceGenerator {
    Invoice generate(Order order) { }
}
class PaymentProcessor {
    void process(Order order, PaymentMethod method) { }
}
```

### Example 2: Employee Management

```java
// ❌ Bad: Employee class with multiple concerns
class Employee {
    void calculatePay() { }
    void saveEmployee() { }
    void reportHours() { }
    void exportData() { }
}

// ✅ Good: Separated by responsibility
class Employee {
    // Employee data only
}
class PayrollCalculator {
    Money calculatePay(Employee employee) { }
}
class EmployeeRepository {
    void save(Employee employee) { }
}
class TimeTracker {
    void recordHours(Employee employee, int hours) { }
}
class EmployeeReporter {
    Report generate(Employee employee) { }
}
```

---

## How to Identify SRP Violations | كيفية تحديد انتهاكات SRP

### Red Flags | العلامات التحذيرية

**English:**
1. **Class name contains "And", "Or", "Manager"** - Suggests multiple responsibilities
2. **Many imports** - Class depends on too many things
3. **Large class** - More than 200-300 lines often indicates multiple responsibilities
4. **Multiple reasons to change** - Ask "Why would this class change?"
5. **Hard to name the class** - Unclear responsibility
6. **Difficult to test** - Too many dependencies or scenarios

**العربية:**
1. **اسم الصنف يحتوي على "و"، "أو"، "مدير"** - يشير إلى مسؤوليات متعددة
2. **استيرادات كثيرة** - الصنف يعتمد على أشياء كثيرة جداً
3. **صنف كبير** - أكثر من 200-300 سطر غالباً يشير إلى مسؤوليات متعددة
4. **أسباب متعددة للتغيير** - اسأل "لماذا قد يتغير هذا الصنف؟"
5. **صعوبة تسمية الصنف** - مسؤولية غير واضحة
6. **صعوبة الاختبار** - اعتماديات أو سيناريوهات كثيرة جداً

### Questions to Ask | أسئلة لطرحها

```
1. Does this class have more than one reason to change?
2. Can I describe what this class does in one sentence without using "and"?
3. Are there methods that operate on different data?
4. Would changes in one part affect other parts?
5. Can I split this class into smaller, focused classes?
```

---

## Benefits of SRP | فوائد مبدأ المسؤولية الواحدة

### English

1. **Easier Testing**
   - Smaller, focused classes are easier to unit test
   - Fewer dependencies to mock
   - Clear test scenarios

2. **Better Organization**
   - Clear separation of concerns
   - Easier to navigate codebase
   - Intuitive project structure

3. **Improved Maintainability**
   - Changes are localized
   - Less risk of breaking unrelated functionality
   - Easier to understand and modify

4. **Enhanced Reusability**
   - Focused classes can be reused in different contexts
   - No unnecessary baggage

5. **Reduced Coupling**
   - Classes depend on fewer things
   - More flexible architecture

6. **Better Collaboration**
   - Different team members can work on different responsibilities
   - Less merge conflicts

### العربية

1. **اختبار أسهل**
   - الأصناف الصغيرة المركزة أسهل في اختبار الوحدات
   - اعتماديات أقل للمحاكاة
   - سيناريوهات اختبار واضحة

2. **تنظيم أفضل**
   - فصل واضح للاهتمامات
   - سهولة التنقل في قاعدة الكود
   - بنية مشروع بديهية

3. **قابلية صيانة محسنة**
   - التغييرات محلية
   - خطر أقل لكسر وظائف غير مرتبطة
   - أسهل للفهم والتعديل

---

## Common Misconceptions | المفاهيم الخاطئة الشائعة

### Misconception #1 | المفهوم الخاطئ #1
**"One method per class"** ❌
- SRP doesn't mean one method
- It means one cohesive responsibility
- Multiple methods can serve the same responsibility

### Misconception #2 | المفهوم الخاطئ #2
**"Create a class for everything"** ❌
- Don't over-engineer
- Balance between SRP and pragmatism
- Small utilities don't need separate classes always

### Misconception #3 | المفهوم الخاطئ #3
**"Never have dependencies"** ❌
- Classes will have dependencies
- SRP is about responsibility, not isolation
- Use dependency injection properly

---

## Interview Questions | أسئلة المقابلات

### Q1: What is the Single Responsibility Principle?
**Answer:**
SRP states that a class should have only one reason to change, meaning it should have only one responsibility or job. This makes the code more maintainable, testable, and easier to understand.

### Q2: How do you identify if a class violates SRP?
**Answer:**
- Ask "How many reasons does this class have to change?"
- Look for multiple concerns (e.g., business logic + persistence)
- Check if the class is difficult to name clearly
- See if it has many dependencies or imports

### Q3: What are the benefits of following SRP?
**Answer:**
- Easier testing and maintenance
- Better code organization
- Reduced coupling
- Improved reusability
- Clearer code intent

### Q4: Can you give an example of SRP violation?
**Answer:**
A User class that handles user data, database operations, email sending, and report generation violates SRP. It should be split into User (data), UserRepository (persistence), EmailService (notifications), and ReportGenerator (reporting).

### Q5: How does SRP relate to other SOLID principles?
**Answer:**
- SRP is the foundation for other SOLID principles
- Helps achieve Open/Closed by making changes localized
- Enables Dependency Inversion by creating focused abstractions
- Works with Interface Segregation to keep interfaces focused

---

## Best Practices | أفضل الممارسات

### English

1. **Name Classes Clearly**
   - Use nouns that describe the single responsibility
   - Avoid generic names like "Manager", "Handler", "Util"

2. **Keep Classes Small**
   - Aim for 100-200 lines of code
   - If larger, consider splitting

3. **Use Composition**
   - Combine focused classes to build complex behavior
   - Prefer composition over inheritance

4. **Follow Cohesion**
   - Methods should operate on the same data
   - All methods should serve the same purpose

5. **Apply Common Sense**
   - Don't over-engineer simple scenarios
   - Balance SRP with practicality

### العربية

1. **سمي الأصناف بوضوح**
   - استخدم أسماء توضح المسؤولية الواحدة
   - تجنب الأسماء العامة

2. **اجعل الأصناف صغيرة**
   - استهدف 100-200 سطر من الكود
   - إذا كانت أكبر، فكر في التقسيم

3. **استخدم التركيب**
   - اجمع الأصناف المركزة لبناء سلوك معقد
   - فضل التركيب على الوراثة

---

## Conclusion | الخلاصة

**English:**
The Single Responsibility Principle is the foundation of clean code. By ensuring each class has one clear purpose, we create code that is easier to test, maintain, and extend. Remember: a class should have only one reason to change.

**العربية:**
مبدأ المسؤولية الواحدة هو أساس الكود النظيف. من خلال التأكد من أن كل صنف له غرض واحد واضح، نخلق كوداً أسهل للاختبار والصيانة والتوسيع. تذكر: يجب أن يكون للصنف سبب واحد فقط للتغيير.

---

**Next:** [Open/Closed Principle →](./02-Open-Closed-Principle.md)
