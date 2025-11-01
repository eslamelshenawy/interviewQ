# Interface Segregation Principle (ISP) | مبدأ فصل الواجهات

## Definition | التعريف

**English:**
> "Clients should not be forced to depend on interfaces they do not use."

**العربية:**
> "يجب ألا يُجبر العملاء على الاعتماد على واجهات لا يستخدمونها."

**Alternative:** "Many client-specific interfaces are better than one general-purpose interface."

---

## The Problem | المشكلة

When interfaces are too large:
- Classes implement methods they don't need
- Changes to unused methods affect clients
- Tight coupling between unrelated functionality
- Hard to understand and maintain

عندما تكون الواجهات كبيرة جداً:
- الأصناف تطبق طرقاً لا تحتاجها
- التغييرات في طرق غير مستخدمة تؤثر على العملاء
- ترابط قوي بين وظائف غير مرتبطة
- صعوبة في الفهم والصيانة

---

## Bad Example | مثال سيء

```java
// ❌ Violates ISP - Fat interface
interface Worker {
    void work();
    void eat();
    void sleep();
    void takeBreak();
    void attendMeeting();
    void submitTimesheet();
}

// Human worker implements all methods - OK!
class HumanWorker implements Worker {
    public void work() {
        System.out.println("Working...");
    }

    public void eat() {
        System.out.println("Eating lunch...");
    }

    public void sleep() {
        System.out.println("Sleeping...");
    }

    public void takeBreak() {
        System.out.println("Taking a break...");
    }

    public void attendMeeting() {
        System.out.println("In meeting...");
    }

    public void submitTimesheet() {
        System.out.println("Submitting timesheet...");
    }
}

// Robot worker is forced to implement irrelevant methods!
class RobotWorker implements Worker {
    public void work() {
        System.out.println("Working 24/7...");
    }

    // ❌ Robots don't eat!
    public void eat() {
        throw new UnsupportedOperationException("Robots don't eat!");
    }

    // ❌ Robots don't sleep!
    public void sleep() {
        throw new UnsupportedOperationException("Robots don't sleep!");
    }

    // ❌ Robots don't take breaks!
    public void takeBreak() {
        throw new UnsupportedOperationException("Robots don't take breaks!");
    }

    // ❌ Robots don't attend meetings!
    public void attendMeeting() {
        throw new UnsupportedOperationException("Robots don't attend meetings!");
    }

    // ❌ Robots don't submit timesheets!
    public void submitTimesheet() {
        throw new UnsupportedOperationException("Robots don't submit timesheets!");
    }
}

// Client forced to know about all Worker methods
class WorkManager {
    public void manage(Worker worker) {
        worker.work();
        worker.eat();  // This will fail for RobotWorker!
    }
}
```

**Problems | المشاكل:**
- ❌ Robot forced to implement methods it doesn't need
- ❌ Many methods throw UnsupportedOperationException
- ❌ Client can't tell which methods are safe to call
- ❌ Violates ISP

---

## Good Example | مثال جيد

```java
// ✅ Follows ISP - Segregated interfaces

// Core interface - everyone can work
interface Workable {
    void work();
}

// Optional capabilities as separate interfaces
interface Eatable {
    void eat();
}

interface Sleepable {
    void sleep();
}

interface Breakable {
    void takeBreak();
}

interface MeetingAttendee {
    void attendMeeting();
}

interface TimesheetSubmitter {
    void submitTimesheet();
}

// Human implements all relevant interfaces
class HumanWorker implements Workable, Eatable, Sleepable,
                             Breakable, MeetingAttendee, TimesheetSubmitter {
    public void work() {
        System.out.println("Working...");
    }

    public void eat() {
        System.out.println("Eating lunch...");
    }

    public void sleep() {
        System.out.println("Sleeping...");
    }

    public void takeBreak() {
        System.out.println("Taking a break...");
    }

    public void attendMeeting() {
        System.out.println("In meeting...");
    }

    public void submitTimesheet() {
        System.out.println("Submitting timesheet...");
    }
}

// Robot only implements what it needs
class RobotWorker implements Workable {
    public void work() {
        System.out.println("Working 24/7...");
    }
    // No irrelevant methods!
}

// Freelancer has different needs
class FreelanceWorker implements Workable, Breakable {
    public void work() {
        System.out.println("Working remotely...");
    }

    public void takeBreak() {
        System.out.println("Taking a coffee break...");
    }
    // No timesheet, no meetings
}

// Clients use only what they need
class WorkManager {
    public void manageWork(Workable worker) {
        worker.work();  // Safe for all workers!
    }

    public void manageLunch(Eatable worker) {
        worker.eat();  // Only called for eatable workers!
    }

    public void organizeBreak(Breakable worker) {
        worker.takeBreak();  // Only for workers that take breaks!
    }
}
```

**Benefits | الفوائد:**
- ✅ Each class implements only what it needs
- ✅ No UnsupportedOperationException
- ✅ Clear, focused interfaces
- ✅ Clients depend only on what they use
- ✅ Follows ISP

---

## Real-World Examples | أمثلة من الواقع

### Example 1: Document Management

```java
// ❌ Bad: Fat interface
interface Document {
    void open();
    void save();
    void print();
    void fax();
    void email();
    void scan();
}

class PDFDocument implements Document {
    public void open() { /* OK */ }
    public void save() { /* OK */ }
    public void print() { /* OK */ }
    public void fax() { throw new UnsupportedOperationException(); }
    public void email() { /* OK */ }
    public void scan() { throw new UnsupportedOperationException(); }
}

// ✅ Good: Segregated interfaces
interface Openable {
    void open();
}

interface Savable {
    void save();
}

interface Printable {
    void print();
}

interface Faxable {
    void fax();
}

interface Emailable {
    void email();
}

interface Scannable {
    void scan();
}

class PDFDocument implements Openable, Savable, Printable, Emailable {
    public void open() { System.out.println("Opening PDF..."); }
    public void save() { System.out.println("Saving PDF..."); }
    public void print() { System.out.println("Printing PDF..."); }
    public void email() { System.out.println("Emailing PDF..."); }
}

class ScannedDocument implements Savable, Printable, Scannable {
    public void save() { System.out.println("Saving scan..."); }
    public void print() { System.out.println("Printing scan..."); }
    public void scan() { System.out.println("Scanning..."); }
}

// Clients can be specific
class Printer {
    public void print(Printable doc) {
        doc.print();
    }
}

class EmailService {
    public void send(Emailable doc) {
        doc.email();
    }
}
```

### Example 2: Database Operations

```java
// ❌ Bad: All-in-one interface
interface Repository {
    void save();
    void update();
    void delete();
    void findById(int id);
    void findAll();
    void search(String query);
    void backup();
    void restore();
}

// Read-only repository forced to implement write methods!
class ReadOnlyRepository implements Repository {
    public void findById(int id) { /* OK */ }
    public void findAll() { /* OK */ }
    public void search(String query) { /* OK */ }

    // All these throw exceptions!
    public void save() { throw new UnsupportedOperationException(); }
    public void update() { throw new UnsupportedOperationException(); }
    public void delete() { throw new UnsupportedOperationException(); }
    public void backup() { throw new UnsupportedOperationException(); }
    public void restore() { throw new UnsupportedOperationException(); }
}

// ✅ Good: Segregated repository interfaces
interface ReadableRepository {
    Object findById(int id);
    List<Object> findAll();
}

interface SearchableRepository extends ReadableRepository {
    List<Object> search(String query);
}

interface WritableRepository {
    void save(Object entity);
    void update(Object entity);
    void delete(int id);
}

interface BackupableRepository {
    void backup();
    void restore();
}

// Now each repository implements what it needs
class UserReadRepository implements SearchableRepository {
    public Object findById(int id) { return new User(); }
    public List<Object> findAll() { return new ArrayList<>(); }
    public List<Object> search(String query) { return new ArrayList<>(); }
}

class UserWriteRepository implements WritableRepository {
    public void save(Object entity) { /* Save logic */ }
    public void update(Object entity) { /* Update logic */ }
    public void delete(int id) { /* Delete logic */ }
}

class AdminRepository implements SearchableRepository,
                                  WritableRepository,
                                  BackupableRepository {
    // Implements all interfaces - admins have full access
    public Object findById(int id) { return null; }
    public List<Object> findAll() { return null; }
    public List<Object> search(String query) { return null; }
    public void save(Object entity) { }
    public void update(Object entity) { }
    public void delete(int id) { }
    public void backup() { }
    public void restore() { }
}
```

### Example 3: Vehicle Interface

```java
// ❌ Bad: All vehicles must implement all methods
interface Vehicle {
    void drive();
    void fly();
    void sail();
}

class Car implements Vehicle {
    public void drive() { System.out.println("Driving..."); }
    public void fly() { throw new UnsupportedOperationException(); }
    public void sail() { throw new UnsupportedOperationException(); }
}

// ✅ Good: Segregated by capability
interface Drivable {
    void drive();
}

interface Flyable {
    void fly();
}

interface Sailable {
    void sail();
}

class Car implements Drivable {
    public void drive() {
        System.out.println("Driving on road...");
    }
}

class Airplane implements Flyable {
    public void fly() {
        System.out.println("Flying in sky...");
    }
}

class Boat implements Sailable {
    public void sail() {
        System.out.println("Sailing on water...");
    }
}

class AmphibiousVehicle implements Drivable, Sailable {
    public void drive() {
        System.out.println("Driving on land...");
    }

    public void sail() {
        System.out.println("Sailing on water...");
    }
}

// Specific handlers
class RoadManager {
    public void manage(Drivable vehicle) {
        vehicle.drive();
    }
}

class AirTrafficControl {
    public void manage(Flyable aircraft) {
        aircraft.fly();
    }
}
```

---

## Benefits of ISP | فوائد مبدأ فصل الواجهات

### English

1. **Reduced Coupling**
   - Clients depend only on methods they use
   - Changes don't affect unrelated clients
   - More modular architecture

2. **Easier Maintenance**
   - Smaller, focused interfaces
   - Clear responsibilities
   - Less ripple effect from changes

3. **Better Testability**
   - Easier to mock small interfaces
   - Test only relevant functionality
   - Clear test boundaries

4. **Improved Flexibility**
   - Easy to add new implementations
   - Can combine interfaces as needed
   - No forced unused dependencies

5. **Clearer API**
   - Intent is obvious from interface name
   - No confusion about which methods to use
   - Self-documenting code

### العربية

1. **تقليل الترابط**
   - العملاء يعتمدون فقط على الطرق التي يستخدمونها
   - التغييرات لا تؤثر على عملاء غير مرتبطين
   - معمارية أكثر نمطية

2. **صيانة أسهل**
   - واجهات صغيرة ومركزة
   - مسؤوليات واضحة
   - تأثير تموجي أقل من التغييرات

---

## How to Apply ISP | كيفية تطبيق ISP

### Step-by-Step Guide | دليل خطوة بخطوة

1. **Identify Fat Interfaces**
   - Look for interfaces with many methods
   - Check for UnsupportedOperationException
   - Find empty method implementations

2. **Group Related Methods**
   - Methods that operate on same data
   - Methods used together
   - Methods serving same purpose

3. **Create Focused Interfaces**
   - One interface per responsibility
   - Use meaningful names
   - Keep interfaces small (2-5 methods)

4. **Use Interface Inheritance**
   - Combine interfaces when needed
   - Build hierarchies carefully
   - Don't create deep hierarchies

5. **Apply to Existing Code**
   - Refactor gradually
   - Maintain backward compatibility
   - Use adapter pattern if needed

---

## Interview Questions | أسئلة المقابلات

### Q1: What is the Interface Segregation Principle?
**Answer:**
ISP states that clients should not be forced to depend on interfaces they don't use. It's better to have many small, focused interfaces than one large, general-purpose interface. This reduces coupling and makes code more maintainable.

### Q2: How does ISP differ from SRP?
**Answer:**
- SRP is about classes having one responsibility
- ISP is about interfaces not forcing unused dependencies
- SRP applies to classes; ISP applies to interfaces
- Both aim to reduce coupling and improve maintainability
- They complement each other

### Q3: What are signs of ISP violation?
**Answer:**
- Methods throwing UnsupportedOperationException
- Empty method implementations
- Interfaces with many unrelated methods
- Clients implementing interfaces they partially use
- Need to check implementation type before calling methods

### Q4: Give an example of ISP in Java
**Answer:**
Java's Collection API follows ISP:
- List, Set, Map are separate interfaces
- Not one monolithic Collection interface with all methods
- Implementations choose relevant interfaces
- Clients depend on specific collection types

### Q5: How do you refactor code to follow ISP?
**Answer:**
1. Identify fat interfaces
2. Group related methods into cohesive interfaces
3. Create multiple focused interfaces
4. Update implementations to use new interfaces
5. Use interface inheritance to combine when needed

---

## ISP and Design Patterns | ISP وأنماط التصميم

### Adapter Pattern
```java
// Use ISP to create focused target interfaces
interface MediaPlayer {
    void play(String filename);
}

interface VolumeControl {
    void setVolume(int level);
}

// Adapter can implement only needed interfaces
```

### Facade Pattern
```java
// Facade provides ISP-compliant simple interfaces
interface OrderFacade {
    void placeOrder();
}

interface PaymentFacade {
    void processPayment();
}
// Instead of one huge facade interface
```

---

## Best Practices | أفضل الممارسات

### English

1. **Keep Interfaces Small**
   - Aim for 1-5 methods per interface
   - Each interface should have one clear purpose
   - Don't add methods "just in case"

2. **Use Role Interfaces**
   - Define interfaces based on client needs
   - Name interfaces by capability (Printable, Readable)
   - Think about how clients will use them

3. **Prefer Composition**
   - Combine multiple interfaces
   - More flexible than single large interface
   - Easier to extend

4. **Don't Over-Segregate**
   - Balance between ISP and pragmatism
   - Don't create interface for every method
   - Group truly related methods

5. **Use Interface Inheritance Wisely**
   - Extend interfaces when there's true relationship
   - Avoid deep hierarchies
   - Keep it simple

### العربية

1. **اجعل الواجهات صغيرة**
   - استهدف 1-5 طرق لكل واجهة
   - كل واجهة يجب أن يكون لها غرض واحد واضح
   - لا تضف طرقاً "للاحتياط فقط"

2. **استخدم واجهات الأدوار**
   - عرّف الواجهات بناءً على احتياجات العميل
   - سمِ الواجهات حسب القدرة
   - فكر في كيفية استخدام العملاء لها

---

## Common Mistakes | الأخطاء الشائعة

### Mistake #1: Over-Segregation
```java
// ❌ Too granular
interface Savable {
    void save();
}

interface Updatable {
    void update();
}
// If always used together, combine them!
```

### Mistake #2: Wrong Grouping
```java
// ❌ Unrelated methods together
interface UserOperations {
    void login();
    void generateReport();  // Unrelated!
}
```

### Mistake #3: God Interface
```java
// ❌ One interface does everything
interface ApplicationService {
    // 50+ methods
}
```

---

## Conclusion | الخلاصة

**English:**
The Interface Segregation Principle keeps interfaces focused and prevents clients from depending on methods they don't use. By creating small, role-based interfaces, we achieve loose coupling, better maintainability, and clearer code intent. Remember: many small interfaces are better than one large interface.

**العربية:**
مبدأ فصل الواجهات يحافظ على تركيز الواجهات ويمنع العملاء من الاعتماد على طرق لا يستخدمونها. من خلال إنشاء واجهات صغيرة قائمة على الأدوار، نحقق ترابطاً ضعيفاً وقابلية صيانة أفضل ونية كود أوضح. تذكر: واجهات صغيرة كثيرة أفضل من واجهة كبيرة واحدة.

---

**Previous:** [← Liskov Substitution Principle](./03-Liskov-Substitution-Principle.md) | **Next:** [Dependency Inversion Principle →](./05-Dependency-Inversion-Principle.md)
