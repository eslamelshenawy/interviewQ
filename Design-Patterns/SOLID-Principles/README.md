# SOLID Principles | مبادئ SOLID

## Overview | نظرة عامة

SOLID is an acronym for five design principles intended to make software designs more understandable, flexible, and maintainable.

SOLID هي اختصار لخمسة مبادئ تصميم تهدف إلى جعل تصاميم البرمجيات أكثر قابلية للفهم والمرونة والصيانة.

---

## The Five Principles | المبادئ الخمسة

| Principle | Description | التوصيف |
|-----------|-------------|---------|
| **S**ingle Responsibility | A class should have one reason to change | الصنف يجب أن يكون له سبب واحد للتغيير |
| **O**pen/Closed | Open for extension, closed for modification | مفتوح للتوسيع، مغلق للتعديل |
| **L**iskov Substitution | Subtypes must be substitutable for base types | الأصناف الفرعية يجب أن تكون قابلة للاستبدال |
| **I**nterface Segregation | Don't force clients to depend on unused methods | لا تجبر العملاء على الاعتماد على طرق غير مستخدمة |
| **D**ependency Inversion | Depend on abstractions, not concretions | اعتمد على التجريدات، وليس التطبيقات |

---

## Quick Navigation | التنقل السريع

1. [Single Responsibility Principle (SRP)](./01-Single-Responsibility-Principle.md)
2. [Open/Closed Principle (OCP)](./02-Open-Closed-Principle.md)
3. [Liskov Substitution Principle (LSP)](./03-Liskov-Substitution-Principle.md)
4. [Interface Segregation Principle (ISP)](./04-Interface-Segregation-Principle.md)
5. [Dependency Inversion Principle (DIP)](./05-Dependency-Inversion-Principle.md)

---

## Why SOLID? | لماذا SOLID؟

### Benefits | الفوائد

**English:**
- ✅ **Maintainability**: Easier to modify and extend code
- ✅ **Testability**: Easier to write unit tests
- ✅ **Flexibility**: Adapt to changing requirements
- ✅ **Reusability**: Components can be reused
- ✅ **Reduced Complexity**: Simpler, more focused classes
- ✅ **Team Collaboration**: Better code organization

**العربية:**
- ✅ **قابلية الصيانة**: سهولة تعديل وتوسيع الكود
- ✅ **قابلية الاختبار**: سهولة كتابة اختبارات الوحدات
- ✅ **المرونة**: التكيف مع المتطلبات المتغيرة
- ✅ **إعادة الاستخدام**: يمكن إعادة استخدام المكونات
- ✅ **تقليل التعقيد**: أصناف أبسط وأكثر تركيزاً
- ✅ **التعاون الجماعي**: تنظيم أفضل للكود

---

## Common Violations | الانتهاكات الشائعة

### 🚫 Code Smells that Violate SOLID

**English:**
- **God Classes**: Classes that do too much (violates SRP)
- **Shotgun Surgery**: One change requires many modifications (violates OCP)
- **Tight Coupling**: Classes depend on concrete implementations (violates DIP)
- **Fat Interfaces**: Interfaces with too many methods (violates ISP)
- **Type Checking**: Using instanceof or type switches (violates LSP & OCP)

**العربية:**
- **الأصناف الإلهية**: أصناف تقوم بالكثير من المهام (ينتهك SRP)
- **جراحة البندقية**: تغيير واحد يتطلب تعديلات كثيرة (ينتهك OCP)
- **الترابط الشديد**: الأصناف تعتمد على تطبيقات محددة (ينتهك DIP)
- **الواجهات الضخمة**: واجهات بطرق كثيرة جداً (ينتهك ISP)
- **فحص الأنواع**: استخدام instanceof أو مفاتيح الأنواع (ينتهك LSP و OCP)

---

## Practical Example | مثال عملي

Let's see how SOLID principles work together in a real scenario:

دعنا نرى كيف تعمل مبادئ SOLID معاً في سيناريو حقيقي:

```java
// ❌ BAD: Violates multiple SOLID principles
class UserManager {
    public void createUser(String name, String email) {
        // Validate email
        if (!email.contains("@")) {
            throw new IllegalArgumentException("Invalid email");
        }

        // Save to database
        Connection conn = DriverManager.getConnection("jdbc:mysql://localhost/db");
        PreparedStatement stmt = conn.prepareStatement("INSERT INTO users VALUES (?, ?)");
        stmt.setString(1, name);
        stmt.setString(2, email);
        stmt.execute();

        // Send welcome email
        SMTP smtp = new SMTP("smtp.gmail.com");
        smtp.send(email, "Welcome!", "Welcome to our platform");

        // Log activity
        FileWriter fw = new FileWriter("log.txt", true);
        fw.write("User created: " + name);
        fw.close();
    }
}
```

```java
// ✅ GOOD: Follows SOLID principles

// Single Responsibility - Each class has one job
interface EmailValidator {
    boolean isValid(String email);
}

interface UserRepository {
    void save(User user);
}

interface EmailService {
    void sendWelcomeEmail(User user);
}

interface Logger {
    void log(String message);
}

// Dependency Inversion - Depend on abstractions
class UserService {
    private final EmailValidator validator;
    private final UserRepository repository;
    private final EmailService emailService;
    private final Logger logger;

    // Dependencies injected through constructor
    public UserService(EmailValidator validator,
                       UserRepository repository,
                       EmailService emailService,
                       Logger logger) {
        this.validator = validator;
        this.repository = repository;
        this.emailService = emailService;
        this.logger = logger;
    }

    public void createUser(String name, String email) {
        if (!validator.isValid(email)) {
            throw new IllegalArgumentException("Invalid email");
        }

        User user = new User(name, email);
        repository.save(user);
        emailService.sendWelcomeEmail(user);
        logger.log("User created: " + name);
    }
}

// Open/Closed - Can extend without modifying
class RegexEmailValidator implements EmailValidator {
    public boolean isValid(String email) {
        return email.matches("^[A-Za-z0-9+_.-]+@(.+)$");
    }
}

// Liskov Substitution - Can substitute implementations
class MySQLUserRepository implements UserRepository {
    public void save(User user) {
        // MySQL implementation
    }
}

class MongoUserRepository implements UserRepository {
    public void save(User user) {
        // MongoDB implementation
    }
}
```

---

## Interview Questions | أسئلة المقابلات

### Common Questions | أسئلة شائعة

1. **What does SOLID stand for?**
   - Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion

2. **Why is SOLID important?**
   - Makes code more maintainable, testable, and flexible to changes

3. **Can you give an example of violating SRP?**
   - A class that handles both business logic and database operations

4. **How does DIP help with testing?**
   - Allows easy mocking and stubbing of dependencies

5. **What's the difference between OCP and LSP?**
   - OCP is about extending without modifying; LSP is about proper inheritance

---

## Comparison | المقارنة

| Principle | Focus | Benefit | مع التركيز على | الفائدة |
|-----------|-------|---------|-----------------|---------|
| SRP | Class responsibility | Easier maintenance | مسؤولية الصنف | صيانة أسهل |
| OCP | Extension vs modification | Less bugs | التوسع مقابل التعديل | أخطاء أقل |
| LSP | Inheritance correctness | Reliable polymorphism | صحة الوراثة | تعددية موثوقة |
| ISP | Interface size | No unused dependencies | حجم الواجهة | لا اعتماديات غير مستخدمة |
| DIP | Dependency direction | Loose coupling | اتجاه الاعتماد | ترابط ضعيف |

---

## Learning Order | ترتيب التعلم

### Recommended Path | المسار الموصى به

1. **Start with SRP** - Easiest to understand and apply
2. **Then OCP** - Builds on SRP concepts
3. **Study DIP** - Critical for modern architectures
4. **Learn ISP** - Related to SRP but for interfaces
5. **Master LSP** - Most complex, requires OOP understanding

### المسار الموصى به بالعربية

1. **ابدأ بـ SRP** - الأسهل للفهم والتطبيق
2. **ثم OCP** - يبني على مفاهيم SRP
3. **ادرس DIP** - حاسم للمعماريات الحديثة
4. **تعلم ISP** - مرتبط بـ SRP ولكن للواجهات
5. **أتقن LSP** - الأكثر تعقيداً، يتطلب فهم OOP

---

## Real-World Applications | التطبيقات الواقعية

### Where SOLID is Used | أين تستخدم SOLID

**Frameworks & Libraries:**
- Spring Framework (heavy use of DIP with Dependency Injection)
- Android Development (interface-based architecture)
- .NET Core (follows all SOLID principles)
- React (component isolation follows SRP)

**Design Patterns:**
- Strategy Pattern → Uses OCP and DIP
- Observer Pattern → Uses DIP and ISP
- Factory Pattern → Uses OCP and DIP
- Decorator Pattern → Uses OCP

---

## Common Mistakes | الأخطاء الشائعة

### English
1. **Over-engineering**: Applying SOLID everywhere unnecessarily
2. **Premature Optimization**: Using SOLID before understanding requirements
3. **Ignoring Context**: Not all code needs strict SOLID adherence
4. **Misunderstanding LSP**: Thinking it's just about inheritance
5. **Creating Too Many Interfaces**: Violating simplicity

### العربية
1. **الإفراط في الهندسة**: تطبيق SOLID في كل مكان دون داعٍ
2. **التحسين المبكر**: استخدام SOLID قبل فهم المتطلبات
3. **تجاهل السياق**: ليس كل الكود يحتاج إلى التزام صارم بـ SOLID
4. **سوء فهم LSP**: الاعتقاد بأنه يتعلق فقط بالوراثة
5. **إنشاء واجهات كثيرة جداً**: انتهاك البساطة

---

## Resources | المصادر

### Books
- **Clean Code** by Robert C. Martin (Uncle Bob)
- **Agile Software Development: Principles, Patterns, and Practices** by Robert C. Martin
- **Clean Architecture** by Robert C. Martin

### Articles
- [SOLID Principles by Uncle Bob](https://blog.cleancoder.com/uncle-bob/2020/10/18/Solid-Relevance.html)
- [SOLID Principles Explained](https://stackify.com/solid-design-principles/)

---

## Next Steps | الخطوات التالية

After understanding SOLID, move to:
1. **Design Patterns** - Apply SOLID in practice
2. **Clean Architecture** - System-level SOLID
3. **Test-Driven Development** - SOLID makes testing easier

بعد فهم SOLID، انتقل إلى:
1. **أنماط التصميم** - تطبيق SOLID عملياً
2. **المعمارية النظيفة** - SOLID على مستوى النظام
3. **التطوير الموجه بالاختبار** - SOLID يجعل الاختبار أسهل

---

**Remember: SOLID is a guideline, not a strict rule. Use your judgment!**

**تذكر: SOLID دليل إرشادي، وليس قاعدة صارمة. استخدم حكمك!**
