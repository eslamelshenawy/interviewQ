# SOLID Principles in Java

## Table of Contents
1. [Introduction](#introduction)
2. [Single Responsibility Principle (SRP)](#single-responsibility-principle-srp)
3. [Open/Closed Principle (OCP)](#openclosed-principle-ocp)
4. [Liskov Substitution Principle (LSP)](#liskov-substitution-principle-lsp)
5. [Interface Segregation Principle (ISP)](#interface-segregation-principle-isp)
6. [Dependency Inversion Principle (DIP)](#dependency-inversion-principle-dip)

---

## Introduction

SOLID is an acronym for five design principles intended to make software designs more understandable, flexible, and maintainable. These principles were introduced by Robert C. Martin (Uncle Bob) and are fundamental to object-oriented programming and design.

**Why SOLID Principles Matter:**
- Reduce code complexity
- Improve code readability and maintainability
- Enable easier testing and debugging
- Facilitate code reusability
- Support scalable architecture

---

## Single Responsibility Principle (SRP)

### Definition
**"A class should have one, and only one, reason to change."**

This principle states that a class should have only one job or responsibility. If a class has multiple responsibilities, it becomes coupled, making it harder to change without affecting other parts of the system.

### Wrong Example (Before)

```java
// Violates SRP - This class has multiple responsibilities
public class Employee {
    private String name;
    private String email;
    private double salary;

    public Employee(String name, String email, double salary) {
        this.name = name;
        this.email = email;
        this.salary = salary;
    }

    // Responsibility 1: Employee data management
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    // Responsibility 2: Salary calculation
    public double calculateTax() {
        return salary * 0.2;
    }

    public double calculateBonus() {
        return salary * 0.1;
    }

    // Responsibility 3: Database operations
    public void saveToDatabase() {
        // Database connection and save logic
        System.out.println("Saving employee to database...");
    }

    // Responsibility 4: Email notifications
    public void sendWelcomeEmail() {
        // Email sending logic
        System.out.println("Sending welcome email to " + email);
    }

    // Responsibility 5: Report generation
    public String generateEmployeeReport() {
        return "Employee Report: " + name + ", Salary: " + salary;
    }
}
```

**Problems:**
- Too many reasons to change (tax rules, database schema, email service, report format)
- Difficult to test individual functionalities
- High coupling between unrelated operations
- Violates separation of concerns

### Correct Example (After)

```java
// Employee class - Only manages employee data
public class Employee {
    private String name;
    private String email;
    private double salary;

    public Employee(String name, String email, double salary) {
        this.name = name;
        this.email = email;
        this.salary = salary;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public double getSalary() {
        return salary;
    }
}

// SalaryCalculator - Handles salary-related calculations
public class SalaryCalculator {
    public double calculateTax(Employee employee) {
        return employee.getSalary() * 0.2;
    }

    public double calculateBonus(Employee employee) {
        return employee.getSalary() * 0.1;
    }

    public double calculateNetSalary(Employee employee) {
        return employee.getSalary() - calculateTax(employee) + calculateBonus(employee);
    }
}

// EmployeeRepository - Handles database operations
public class EmployeeRepository {
    public void save(Employee employee) {
        // Database connection and save logic
        System.out.println("Saving employee to database: " + employee.getName());
    }

    public Employee findById(int id) {
        // Fetch employee from database
        return null;
    }

    public void update(Employee employee) {
        // Update employee in database
        System.out.println("Updating employee: " + employee.getName());
    }
}

// EmailService - Handles email notifications
public class EmailService {
    public void sendWelcomeEmail(Employee employee) {
        // Email sending logic
        System.out.println("Sending welcome email to " + employee.getEmail());
    }

    public void sendPayslip(Employee employee) {
        System.out.println("Sending payslip to " + employee.getEmail());
    }
}

// EmployeeReportGenerator - Handles report generation
public class EmployeeReportGenerator {
    public String generateReport(Employee employee) {
        return "Employee Report:\n" +
               "Name: " + employee.getName() + "\n" +
               "Email: " + employee.getEmail() + "\n" +
               "Salary: " + employee.getSalary();
    }

    public String generateSalarySlip(Employee employee, SalaryCalculator calculator) {
        return "Salary Slip:\n" +
               "Gross Salary: " + employee.getSalary() + "\n" +
               "Tax: " + calculator.calculateTax(employee) + "\n" +
               "Bonus: " + calculator.calculateBonus(employee) + "\n" +
               "Net Salary: " + calculator.calculateNetSalary(employee);
    }
}
```

### Real-World Use Cases

1. **E-commerce System**: Separate classes for Order, PaymentProcessor, InventoryManager, and EmailNotifier
2. **Banking Application**: Different classes for Account, TransactionProcessor, InterestCalculator, and StatementGenerator
3. **User Management**: Separate UserProfile, UserAuthentication, UserAuthorization, and UserNotification classes

### Benefits

- **Easier Maintenance**: Changes to one responsibility don't affect others
- **Better Testing**: Each class can be tested independently
- **Improved Readability**: Smaller, focused classes are easier to understand
- **Reduced Coupling**: Classes have fewer dependencies
- **Reusability**: Focused classes are easier to reuse in different contexts

---

## Open/Closed Principle (OCP)

### Definition
**"Software entities (classes, modules, functions) should be open for extension but closed for modification."**

This means you should be able to add new functionality without changing existing code. This is typically achieved through abstraction and polymorphism.

### Wrong Example (Before)

```java
// Violates OCP - Requires modification to add new payment methods
public class PaymentProcessor {
    public void processPayment(String paymentType, double amount) {
        if (paymentType.equals("CREDIT_CARD")) {
            System.out.println("Processing credit card payment: $" + amount);
            // Credit card processing logic
            validateCreditCard();
            chargeCreditCard(amount);
        } else if (paymentType.equals("PAYPAL")) {
            System.out.println("Processing PayPal payment: $" + amount);
            // PayPal processing logic
            authenticatePayPal();
            transferPayPal(amount);
        } else if (paymentType.equals("BITCOIN")) {
            System.out.println("Processing Bitcoin payment: $" + amount);
            // Bitcoin processing logic
            verifyBitcoinWallet();
            transferBitcoin(amount);
        }
        // Adding new payment method requires modifying this class
    }

    private void validateCreditCard() { /* ... */ }
    private void chargeCreditCard(double amount) { /* ... */ }
    private void authenticatePayPal() { /* ... */ }
    private void transferPayPal(double amount) { /* ... */ }
    private void verifyBitcoinWallet() { /* ... */ }
    private void transferBitcoin(double amount) { /* ... */ }
}
```

**Problems:**
- Must modify existing code to add new payment methods
- Risk of breaking existing functionality
- Violates OCP - not closed for modification
- Difficult to test each payment method independently

### Correct Example (After)

```java
// Payment interface - Abstraction for all payment methods
public interface PaymentMethod {
    void processPayment(double amount);
    boolean validatePayment();
}

// Credit Card payment implementation
public class CreditCardPayment implements PaymentMethod {
    private String cardNumber;
    private String cvv;
    private String expiryDate;

    public CreditCardPayment(String cardNumber, String cvv, String expiryDate) {
        this.cardNumber = cardNumber;
        this.cvv = cvv;
        this.expiryDate = expiryDate;
    }

    @Override
    public boolean validatePayment() {
        // Validate credit card details
        System.out.println("Validating credit card...");
        return cardNumber != null && cvv != null && expiryDate != null;
    }

    @Override
    public void processPayment(double amount) {
        if (validatePayment()) {
            System.out.println("Processing credit card payment: $" + amount);
            // Credit card specific processing logic
        }
    }
}

// PayPal payment implementation
public class PayPalPayment implements PaymentMethod {
    private String email;
    private String password;

    public PayPalPayment(String email, String password) {
        this.email = email;
        this.password = password;
    }

    @Override
    public boolean validatePayment() {
        System.out.println("Authenticating PayPal account...");
        return email != null && password != null;
    }

    @Override
    public void processPayment(double amount) {
        if (validatePayment()) {
            System.out.println("Processing PayPal payment: $" + amount);
            // PayPal specific processing logic
        }
    }
}

// Bitcoin payment implementation
public class BitcoinPayment implements PaymentMethod {
    private String walletAddress;
    private String privateKey;

    public BitcoinPayment(String walletAddress, String privateKey) {
        this.walletAddress = walletAddress;
        this.privateKey = privateKey;
    }

    @Override
    public boolean validatePayment() {
        System.out.println("Verifying Bitcoin wallet...");
        return walletAddress != null && privateKey != null;
    }

    @Override
    public void processPayment(double amount) {
        if (validatePayment()) {
            System.out.println("Processing Bitcoin payment: $" + amount);
            // Bitcoin specific processing logic
        }
    }
}

// New payment method can be added without modifying existing code
public class GooglePayPayment implements PaymentMethod {
    private String accountId;

    public GooglePayPayment(String accountId) {
        this.accountId = accountId;
    }

    @Override
    public boolean validatePayment() {
        System.out.println("Validating Google Pay account...");
        return accountId != null;
    }

    @Override
    public void processPayment(double amount) {
        if (validatePayment()) {
            System.out.println("Processing Google Pay payment: $" + amount);
            // Google Pay specific processing logic
        }
    }
}

// Payment processor - Open for extension, closed for modification
public class PaymentProcessor {
    public void process(PaymentMethod paymentMethod, double amount) {
        paymentMethod.processPayment(amount);
    }
}

// Usage example
public class PaymentDemo {
    public static void main(String[] args) {
        PaymentProcessor processor = new PaymentProcessor();

        PaymentMethod creditCard = new CreditCardPayment("1234-5678-9012-3456", "123", "12/25");
        processor.process(creditCard, 100.0);

        PaymentMethod paypal = new PayPalPayment("user@example.com", "password");
        processor.process(paypal, 200.0);

        PaymentMethod bitcoin = new BitcoinPayment("1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa", "privatekey");
        processor.process(bitcoin, 300.0);

        // New payment method - no modification to existing code needed
        PaymentMethod googlePay = new GooglePayPayment("google-account-123");
        processor.process(googlePay, 150.0);
    }
}
```

### Real-World Use Cases

1. **Notification System**: Email, SMS, Push notifications - add new channels without modifying core logic
2. **Shipping Calculators**: Different shipping methods (Standard, Express, Overnight) as strategies
3. **Report Generators**: PDF, Excel, CSV formats without changing report generation logic
4. **Authentication Providers**: OAuth, LDAP, Database authentication

### Benefits

- **Extensibility**: Add new features without modifying existing code
- **Stability**: Existing code remains unchanged, reducing risk of bugs
- **Maintainability**: Changes are isolated to new classes
- **Testability**: New functionality can be tested independently
- **Scalability**: Easy to add new behaviors as requirements grow

---

## Liskov Substitution Principle (LSP)

### Definition
**"Objects of a superclass should be replaceable with objects of its subclasses without breaking the application."**

This principle states that derived classes must be substitutable for their base classes. A subclass should extend the capability of the parent class, not replace or reduce it.

### Wrong Example (Before)

```java
// Violates LSP - Not all birds can fly
public class Bird {
    private String name;

    public Bird(String name) {
        this.name = name;
    }

    public void fly() {
        System.out.println(name + " is flying");
    }

    public void eat() {
        System.out.println(name + " is eating");
    }
}

public class Sparrow extends Bird {
    public Sparrow() {
        super("Sparrow");
    }

    @Override
    public void fly() {
        System.out.println("Sparrow is flying high!");
    }
}

// Problem: Penguin cannot fly, but inherits fly() method
public class Penguin extends Bird {
    public Penguin() {
        super("Penguin");
    }

    @Override
    public void fly() {
        // Penguins can't fly - this violates LSP
        throw new UnsupportedOperationException("Penguins cannot fly!");
    }
}

// Problem: Ostrich cannot fly either
public class Ostrich extends Bird {
    public Ostrich() {
        super("Ostrich");
    }

    @Override
    public void fly() {
        // Ostriches can't fly
        throw new UnsupportedOperationException("Ostriches cannot fly!");
    }
}

// This code will break when we substitute Bird with Penguin or Ostrich
public class BirdWatcher {
    public void watchBird(Bird bird) {
        bird.fly(); // Will throw exception for Penguin and Ostrich
        bird.eat();
    }

    public static void main(String[] args) {
        BirdWatcher watcher = new BirdWatcher();

        Bird sparrow = new Sparrow();
        watcher.watchBird(sparrow); // Works fine

        Bird penguin = new Penguin();
        watcher.watchBird(penguin); // Throws exception - violates LSP!
    }
}
```

**Problems:**
- Substituting Bird with Penguin breaks the application
- Subclasses throw exceptions for inherited behavior
- Client code needs to check the specific type before calling methods
- Violates the contract established by the base class

### Correct Example (After)

```java
// Base class for all birds
public abstract class Bird {
    protected String name;

    public Bird(String name) {
        this.name = name;
    }

    public void eat() {
        System.out.println(name + " is eating");
    }

    public void sleep() {
        System.out.println(name + " is sleeping");
    }

    public abstract void move();
}

// Interface for flying capability
public interface Flyable {
    void fly();
    int getMaxAltitude();
}

// Interface for swimming capability
public interface Swimmable {
    void swim();
    int getMaxDepth();
}

// Sparrow can fly
public class Sparrow extends Bird implements Flyable {
    public Sparrow() {
        super("Sparrow");
    }

    @Override
    public void move() {
        fly();
    }

    @Override
    public void fly() {
        System.out.println("Sparrow is flying high!");
    }

    @Override
    public int getMaxAltitude() {
        return 3000; // meters
    }
}

// Eagle can fly
public class Eagle extends Bird implements Flyable {
    public Eagle() {
        super("Eagle");
    }

    @Override
    public void move() {
        fly();
    }

    @Override
    public void fly() {
        System.out.println("Eagle is soaring!");
    }

    @Override
    public int getMaxAltitude() {
        return 10000; // meters
    }
}

// Penguin can swim but cannot fly
public class Penguin extends Bird implements Swimmable {
    public Penguin() {
        super("Penguin");
    }

    @Override
    public void move() {
        swim();
    }

    @Override
    public void swim() {
        System.out.println("Penguin is swimming!");
    }

    @Override
    public int getMaxDepth() {
        return 250; // meters
    }
}

// Ostrich can run but cannot fly
public class Ostrich extends Bird {
    public Ostrich() {
        super("Ostrich");
    }

    @Override
    public void move() {
        run();
    }

    public void run() {
        System.out.println("Ostrich is running fast!");
    }
}

// Duck can both fly and swim
public class Duck extends Bird implements Flyable, Swimmable {
    public Duck() {
        super("Duck");
    }

    @Override
    public void move() {
        fly(); // or swim() depending on context
    }

    @Override
    public void fly() {
        System.out.println("Duck is flying!");
    }

    @Override
    public int getMaxAltitude() {
        return 5000;
    }

    @Override
    public void swim() {
        System.out.println("Duck is swimming!");
    }

    @Override
    public int getMaxDepth() {
        return 10;
    }
}

// Client code respects LSP
public class BirdSanctuary {
    // Works with all birds
    public void feedBird(Bird bird) {
        bird.eat();
    }

    // Works only with flying birds
    public void flyingShow(Flyable flyingBird) {
        flyingBird.fly();
        System.out.println("Max altitude: " + flyingBird.getMaxAltitude() + " meters");
    }

    // Works only with swimming birds
    public void swimmingShow(Swimmable swimmingBird) {
        swimmingBird.swim();
        System.out.println("Max depth: " + swimmingBird.getMaxDepth() + " meters");
    }

    public static void main(String[] args) {
        BirdSanctuary sanctuary = new BirdSanctuary();

        Bird sparrow = new Sparrow();
        Bird penguin = new Penguin();
        Bird ostrich = new Ostrich();
        Bird duck = new Duck();

        // All birds can be fed - LSP satisfied
        sanctuary.feedBird(sparrow);
        sanctuary.feedBird(penguin);
        sanctuary.feedBird(ostrich);
        sanctuary.feedBird(duck);

        // Only flying birds participate in flying show
        sanctuary.flyingShow((Flyable) sparrow);
        sanctuary.flyingShow((Flyable) duck);

        // Only swimming birds participate in swimming show
        sanctuary.swimmingShow((Swimmable) penguin);
        sanctuary.swimmingShow((Swimmable) duck);

        // Each bird moves in its own way
        sparrow.move();
        penguin.move();
        ostrich.move();
        duck.move();
    }
}
```

### Real-World Use Cases

1. **Shape Hierarchy**: Rectangle and Square problem - Square should not extend Rectangle if it violates area calculation contracts
2. **Database Connections**: All database implementations should properly implement connection interface methods
3. **Collection Framework**: List, Set, Map implementations all properly substitute their interfaces
4. **User Types**: Admin, Regular User, Guest - each should properly implement User interface without breaking contracts

### Benefits

- **Reliability**: Subclasses can be used interchangeably without unexpected behavior
- **Maintainability**: Changes to subclasses don't break client code
- **Reusability**: Client code works with base types without knowing specific implementations
- **Polymorphism**: Enables true polymorphic behavior
- **Contract Compliance**: Ensures subclasses honor the contracts of their parent classes

---

## Interface Segregation Principle (ISP)

### Definition
**"Clients should not be forced to depend on interfaces they do not use."**

This principle states that many client-specific interfaces are better than one general-purpose interface. No client should be forced to implement methods it doesn't use.

### Wrong Example (Before)

```java
// Fat interface - Violates ISP
public interface Worker {
    void work();
    void eat();
    void sleep();
    void attendMeeting();
    void submitTimesheet();
    void takePaidLeave();
    void receiveHealthBenefits();
}

// Full-time employee can implement all methods
public class FullTimeEmployee implements Worker {
    private String name;

    public FullTimeEmployee(String name) {
        this.name = name;
    }

    @Override
    public void work() {
        System.out.println(name + " is working");
    }

    @Override
    public void eat() {
        System.out.println(name + " is eating");
    }

    @Override
    public void sleep() {
        System.out.println(name + " is sleeping");
    }

    @Override
    public void attendMeeting() {
        System.out.println(name + " is attending meeting");
    }

    @Override
    public void submitTimesheet() {
        System.out.println(name + " is submitting timesheet");
    }

    @Override
    public void takePaidLeave() {
        System.out.println(name + " is taking paid leave");
    }

    @Override
    public void receiveHealthBenefits() {
        System.out.println(name + " is receiving health benefits");
    }
}

// Robot worker - forced to implement irrelevant methods
public class Robot implements Worker {
    private String id;

    public Robot(String id) {
        this.id = id;
    }

    @Override
    public void work() {
        System.out.println("Robot " + id + " is working 24/7");
    }

    @Override
    public void eat() {
        // Robots don't eat - forced to implement
        throw new UnsupportedOperationException("Robots don't eat");
    }

    @Override
    public void sleep() {
        // Robots don't sleep - forced to implement
        throw new UnsupportedOperationException("Robots don't sleep");
    }

    @Override
    public void attendMeeting() {
        // Robots don't attend meetings - forced to implement
        throw new UnsupportedOperationException("Robots don't attend meetings");
    }

    @Override
    public void submitTimesheet() {
        // Robots don't submit timesheets - forced to implement
        throw new UnsupportedOperationException("Robots don't submit timesheets");
    }

    @Override
    public void takePaidLeave() {
        // Robots don't take leave - forced to implement
        throw new UnsupportedOperationException("Robots don't take leave");
    }

    @Override
    public void receiveHealthBenefits() {
        // Robots don't need health benefits - forced to implement
        throw new UnsupportedOperationException("Robots don't need health benefits");
    }
}

// Contractor - forced to implement methods they don't need
public class Contractor implements Worker {
    private String name;

    public Contractor(String name) {
        this.name = name;
    }

    @Override
    public void work() {
        System.out.println(name + " is working on contract");
    }

    @Override
    public void eat() {
        System.out.println(name + " is eating");
    }

    @Override
    public void sleep() {
        System.out.println(name + " is sleeping");
    }

    @Override
    public void attendMeeting() {
        System.out.println(name + " is attending client meeting");
    }

    @Override
    public void submitTimesheet() {
        System.out.println(name + " is submitting timesheet");
    }

    @Override
    public void takePaidLeave() {
        // Contractors don't get paid leave - forced to implement
        throw new UnsupportedOperationException("Contractors don't get paid leave");
    }

    @Override
    public void receiveHealthBenefits() {
        // Contractors don't get company health benefits - forced to implement
        throw new UnsupportedOperationException("Contractors don't get health benefits");
    }
}
```

**Problems:**
- Fat interface forces classes to implement methods they don't need
- Many methods throw UnsupportedOperationException
- Difficult to maintain and understand
- Changes to interface affect all implementing classes

### Correct Example (After)

```java
// Segregated interfaces - Each interface has a specific purpose

// Core work interface
public interface Workable {
    void work();
}

// Human needs interface
public interface Eatable {
    void eat();
}

public interface Sleepable {
    void sleep();
}

// Employee-specific interfaces
public interface MeetingAttendee {
    void attendMeeting();
}

public interface TimesheetSubmitter {
    void submitTimesheet();
}

// Full-time employee benefits
public interface LeaveEligible {
    void takePaidLeave();
    int getRemainingLeaveDays();
}

public interface BenefitsEligible {
    void receiveHealthBenefits();
    void enrollInPensionPlan();
}

// Full-time employee implements all relevant interfaces
public class FullTimeEmployee implements Workable, Eatable, Sleepable,
                                         MeetingAttendee, TimesheetSubmitter,
                                         LeaveEligible, BenefitsEligible {
    private String name;
    private int leaveDays = 20;

    public FullTimeEmployee(String name) {
        this.name = name;
    }

    @Override
    public void work() {
        System.out.println(name + " is working");
    }

    @Override
    public void eat() {
        System.out.println(name + " is eating lunch");
    }

    @Override
    public void sleep() {
        System.out.println(name + " is sleeping");
    }

    @Override
    public void attendMeeting() {
        System.out.println(name + " is attending meeting");
    }

    @Override
    public void submitTimesheet() {
        System.out.println(name + " is submitting timesheet");
    }

    @Override
    public void takePaidLeave() {
        if (leaveDays > 0) {
            leaveDays--;
            System.out.println(name + " is taking paid leave. Remaining: " + leaveDays);
        }
    }

    @Override
    public int getRemainingLeaveDays() {
        return leaveDays;
    }

    @Override
    public void receiveHealthBenefits() {
        System.out.println(name + " is receiving health benefits");
    }

    @Override
    public void enrollInPensionPlan() {
        System.out.println(name + " is enrolled in pension plan");
    }
}

// Robot only implements what it needs
public class Robot implements Workable {
    private String id;

    public Robot(String id) {
        this.id = id;
    }

    @Override
    public void work() {
        System.out.println("Robot " + id + " is working 24/7");
    }

    // No need to implement eat, sleep, meetings, etc.
}

// Contractor implements only relevant interfaces
public class Contractor implements Workable, Eatable, Sleepable,
                                   MeetingAttendee, TimesheetSubmitter {
    private String name;

    public Contractor(String name) {
        this.name = name;
    }

    @Override
    public void work() {
        System.out.println(name + " is working on contract");
    }

    @Override
    public void eat() {
        System.out.println(name + " is eating");
    }

    @Override
    public void sleep() {
        System.out.println(name + " is sleeping");
    }

    @Override
    public void attendMeeting() {
        System.out.println(name + " is attending client meeting");
    }

    @Override
    public void submitTimesheet() {
        System.out.println(name + " is submitting billable hours");
    }

    // No leave or benefits - doesn't implement those interfaces
}

// Intern - limited responsibilities
public class Intern implements Workable, Eatable, Sleepable, MeetingAttendee {
    private String name;

    public Intern(String name) {
        this.name = name;
    }

    @Override
    public void work() {
        System.out.println(name + " is learning and working");
    }

    @Override
    public void eat() {
        System.out.println(name + " is eating");
    }

    @Override
    public void sleep() {
        System.out.println(name + " is sleeping");
    }

    @Override
    public void attendMeeting() {
        System.out.println(name + " is observing the meeting");
    }

    // No timesheet submission or benefits
}

// Client code works with specific interfaces
public class WorkforceManager {
    public void assignWork(Workable worker) {
        worker.work();
    }

    public void scheduleBreak(Eatable worker) {
        worker.eat();
    }

    public void organizeMeeting(List<MeetingAttendee> attendees) {
        for (MeetingAttendee attendee : attendees) {
            attendee.attendMeeting();
        }
    }

    public void processTimesheets(List<TimesheetSubmitter> submitters) {
        for (TimesheetSubmitter submitter : submitters) {
            submitter.submitTimesheet();
        }
    }

    public void processLeaveRequests(LeaveEligible employee) {
        System.out.println("Processing leave request...");
        employee.takePaidLeave();
        System.out.println("Remaining leave days: " + employee.getRemainingLeaveDays());
    }

    public static void main(String[] args) {
        WorkforceManager manager = new WorkforceManager();

        FullTimeEmployee john = new FullTimeEmployee("John");
        Contractor jane = new Contractor("Jane");
        Robot robot = new Robot("R2D2");
        Intern mike = new Intern("Mike");

        // All can work
        manager.assignWork(john);
        manager.assignWork(jane);
        manager.assignWork(robot);
        manager.assignWork(mike);

        // Only humans need breaks
        manager.scheduleBreak(john);
        manager.scheduleBreak(jane);

        // Meeting with humans only
        List<MeetingAttendee> meeting = Arrays.asList(john, jane, mike);
        manager.organizeMeeting(meeting);

        // Only employees and contractors submit timesheets
        List<TimesheetSubmitter> timesheetSubmitters = Arrays.asList(john, jane);
        manager.processTimesheets(timesheetSubmitters);

        // Only full-time employees have leave
        manager.processLeaveRequests(john);
    }
}
```

### Real-World Use Cases

1. **Printer Interfaces**: Separate interfaces for Print, Scan, Fax instead of one MultiFunctionDevice
2. **Database Operations**: Separate interfaces for Readable, Writable, Deletable instead of one CRUD interface
3. **UI Components**: Separate Clickable, Draggable, Resizable interfaces instead of one Interactive interface
4. **Payment Systems**: Separate interfaces for Refundable, Subscribable, OneTimePayable

### Benefits

- **Flexibility**: Classes implement only what they need
- **Maintainability**: Changes to one interface don't affect unrelated classes
- **Clarity**: Smaller interfaces are easier to understand and implement
- **Testability**: Easier to mock and test specific behaviors
- **Reduced Coupling**: Classes don't depend on methods they don't use

---

## Dependency Inversion Principle (DIP)

### Definition
**"High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions."**

This principle states that we should depend on abstractions (interfaces/abstract classes) rather than concrete implementations.

### Wrong Example (Before)

```java
// Low-level module - MySQL Database
public class MySQLDatabase {
    public void connect() {
        System.out.println("Connecting to MySQL database...");
    }

    public void executeQuery(String query) {
        System.out.println("Executing MySQL query: " + query);
    }

    public void disconnect() {
        System.out.println("Disconnecting from MySQL database...");
    }
}

// High-level module directly depends on low-level module
public class UserService {
    private MySQLDatabase database; // Direct dependency on concrete class

    public UserService() {
        this.database = new MySQLDatabase(); // Tight coupling
    }

    public void getUser(int userId) {
        database.connect();
        database.executeQuery("SELECT * FROM users WHERE id = " + userId);
        database.disconnect();
    }

    public void saveUser(String username) {
        database.connect();
        database.executeQuery("INSERT INTO users (username) VALUES ('" + username + "')");
        database.disconnect();
    }
}

// Another high-level module with same problem
public class ProductService {
    private MySQLDatabase database; // Direct dependency on concrete class

    public ProductService() {
        this.database = new MySQLDatabase();
    }

    public void getProduct(int productId) {
        database.connect();
        database.executeQuery("SELECT * FROM products WHERE id = " + productId);
        database.disconnect();
    }
}

// Client code
public class Application {
    public static void main(String[] args) {
        UserService userService = new UserService();
        userService.getUser(1);

        // Problem: If we want to switch to PostgreSQL or MongoDB,
        // we need to modify UserService and ProductService classes
    }
}
```

**Problems:**
- High-level modules (UserService, ProductService) tightly coupled to low-level module (MySQLDatabase)
- Cannot easily switch to different database implementations
- Difficult to test (cannot mock the database)
- Violates Open/Closed Principle
- Changes to MySQLDatabase affect all dependent classes

### Correct Example (After)

```java
// Abstraction - Database interface
public interface Database {
    void connect();
    void executeQuery(String query);
    void disconnect();
    List<Map<String, Object>> fetchResults();
}

// Low-level module - MySQL implementation
public class MySQLDatabase implements Database {
    @Override
    public void connect() {
        System.out.println("Connecting to MySQL database...");
    }

    @Override
    public void executeQuery(String query) {
        System.out.println("Executing MySQL query: " + query);
    }

    @Override
    public void disconnect() {
        System.out.println("Disconnecting from MySQL database...");
    }

    @Override
    public List<Map<String, Object>> fetchResults() {
        System.out.println("Fetching MySQL results...");
        return new ArrayList<>();
    }
}

// Low-level module - PostgreSQL implementation
public class PostgreSQLDatabase implements Database {
    @Override
    public void connect() {
        System.out.println("Connecting to PostgreSQL database...");
    }

    @Override
    public void executeQuery(String query) {
        System.out.println("Executing PostgreSQL query: " + query);
    }

    @Override
    public void disconnect() {
        System.out.println("Disconnecting from PostgreSQL database...");
    }

    @Override
    public List<Map<String, Object>> fetchResults() {
        System.out.println("Fetching PostgreSQL results...");
        return new ArrayList<>();
    }
}

// Low-level module - MongoDB implementation
public class MongoDB implements Database {
    @Override
    public void connect() {
        System.out.println("Connecting to MongoDB...");
    }

    @Override
    public void executeQuery(String query) {
        System.out.println("Executing MongoDB query: " + query);
    }

    @Override
    public void disconnect() {
        System.out.println("Disconnecting from MongoDB...");
    }

    @Override
    public List<Map<String, Object>> fetchResults() {
        System.out.println("Fetching MongoDB documents...");
        return new ArrayList<>();
    }
}

// High-level module - depends on abstraction
public class UserService {
    private Database database; // Depends on abstraction, not concrete class

    // Dependency injection through constructor
    public UserService(Database database) {
        this.database = database;
    }

    public void getUser(int userId) {
        database.connect();
        database.executeQuery("SELECT * FROM users WHERE id = " + userId);
        List<Map<String, Object>> results = database.fetchResults();
        database.disconnect();
        System.out.println("User retrieved successfully");
    }

    public void saveUser(String username) {
        database.connect();
        database.executeQuery("INSERT INTO users (username) VALUES ('" + username + "')");
        database.disconnect();
        System.out.println("User saved successfully");
    }

    // Setter injection (alternative to constructor injection)
    public void setDatabase(Database database) {
        this.database = database;
    }
}

// High-level module - depends on abstraction
public class ProductService {
    private Database database;

    public ProductService(Database database) {
        this.database = database;
    }

    public void getProduct(int productId) {
        database.connect();
        database.executeQuery("SELECT * FROM products WHERE id = " + productId);
        List<Map<String, Object>> results = database.fetchResults();
        database.disconnect();
        System.out.println("Product retrieved successfully");
    }

    public void saveProduct(String productName, double price) {
        database.connect();
        database.executeQuery("INSERT INTO products (name, price) VALUES ('" +
                            productName + "', " + price + ")");
        database.disconnect();
        System.out.println("Product saved successfully");
    }
}

// Abstraction for notification service
public interface NotificationService {
    void sendNotification(String message, String recipient);
}

// Low-level module - Email notification
public class EmailNotification implements NotificationService {
    @Override
    public void sendNotification(String message, String recipient) {
        System.out.println("Sending email to " + recipient + ": " + message);
    }
}

// Low-level module - SMS notification
public class SMSNotification implements NotificationService {
    @Override
    public void sendNotification(String message, String recipient) {
        System.out.println("Sending SMS to " + recipient + ": " + message);
    }
}

// Low-level module - Push notification
public class PushNotification implements NotificationService {
    @Override
    public void sendNotification(String message, String recipient) {
        System.out.println("Sending push notification to " + recipient + ": " + message);
    }
}

// High-level module using multiple dependencies
public class OrderService {
    private Database database;
    private NotificationService notificationService;

    // Constructor injection with multiple dependencies
    public OrderService(Database database, NotificationService notificationService) {
        this.database = database;
        this.notificationService = notificationService;
    }

    public void placeOrder(int userId, int productId) {
        database.connect();
        database.executeQuery("INSERT INTO orders (user_id, product_id) VALUES (" +
                            userId + ", " + productId + ")");
        database.disconnect();

        notificationService.sendNotification(
            "Your order has been placed successfully!",
            "user" + userId + "@example.com"
        );

        System.out.println("Order placed successfully");
    }
}

// Application configuration (Dependency Injection Container simulation)
public class ApplicationConfig {
    public static Database createDatabase(String type) {
        switch (type) {
            case "mysql":
                return new MySQLDatabase();
            case "postgresql":
                return new PostgreSQLDatabase();
            case "mongodb":
                return new MongoDB();
            default:
                return new MySQLDatabase();
        }
    }

    public static NotificationService createNotificationService(String type) {
        switch (type) {
            case "email":
                return new EmailNotification();
            case "sms":
                return new SMSNotification();
            case "push":
                return new PushNotification();
            default:
                return new EmailNotification();
        }
    }
}

// Client code
public class Application {
    public static void main(String[] args) {
        // Configuration - easily switch implementations
        Database database = ApplicationConfig.createDatabase("mysql");
        NotificationService notificationService = ApplicationConfig.createNotificationService("email");

        // Dependency injection
        UserService userService = new UserService(database);
        ProductService productService = new ProductService(database);
        OrderService orderService = new OrderService(database, notificationService);

        // Use services
        userService.saveUser("john_doe");
        userService.getUser(1);

        productService.saveProduct("Laptop", 999.99);
        productService.getProduct(1);

        orderService.placeOrder(1, 1);

        System.out.println("\n--- Switching to PostgreSQL and SMS ---\n");

        // Easy to switch implementations without modifying service classes
        Database postgresDB = ApplicationConfig.createDatabase("postgresql");
        NotificationService smsService = ApplicationConfig.createNotificationService("sms");

        UserService userService2 = new UserService(postgresDB);
        OrderService orderService2 = new OrderService(postgresDB, smsService);

        userService2.getUser(2);
        orderService2.placeOrder(2, 2);
    }
}

// Example with testing - easy to mock
class MockDatabase implements Database {
    @Override
    public void connect() {
        System.out.println("Mock: Connected");
    }

    @Override
    public void executeQuery(String query) {
        System.out.println("Mock: Query executed - " + query);
    }

    @Override
    public void disconnect() {
        System.out.println("Mock: Disconnected");
    }

    @Override
    public List<Map<String, Object>> fetchResults() {
        return Arrays.asList(
            Map.of("id", 1, "username", "test_user")
        );
    }
}

// Unit test example
public class UserServiceTest {
    public static void testUserService() {
        // Easy to test with mock
        Database mockDatabase = new MockDatabase();
        UserService userService = new UserService(mockDatabase);

        System.out.println("\n--- Testing UserService ---");
        userService.getUser(1);
    }

    public static void main(String[] args) {
        testUserService();
    }
}
```

### Real-World Use Cases

1. **Payment Gateway Integration**: Depend on PaymentGateway interface, not specific providers (Stripe, PayPal)
2. **Logging Systems**: Depend on Logger interface, not specific implementations (FileLogger, DatabaseLogger)
3. **Authentication Services**: Depend on AuthProvider interface, not specific providers (OAuth, LDAP, JWT)
4. **Cache Systems**: Depend on Cache interface, not specific implementations (Redis, Memcached)
5. **Email Services**: Depend on EmailService interface, not specific providers (SendGrid, AWS SES)

### Benefits

- **Flexibility**: Easy to switch implementations without modifying high-level code
- **Testability**: Easy to create mocks and stubs for testing
- **Maintainability**: Changes to low-level modules don't affect high-level modules
- **Reusability**: High-level modules can work with any implementation of the abstraction
- **Decoupling**: Reduces dependencies between modules
- **Extensibility**: New implementations can be added without modifying existing code

---

## Summary and Interview Tips

### Quick Reference

| Principle | Key Question | Goal |
|-----------|-------------|------|
| **SRP** | Does this class have more than one reason to change? | One class = One responsibility |
| **OCP** | Can I add new features without modifying existing code? | Open for extension, closed for modification |
| **LSP** | Can I substitute a parent class with a child class? | Subclasses should be substitutable |
| **ISP** | Am I forced to implement methods I don't need? | Many specific interfaces > One general interface |
| **DIP** | Am I depending on concrete classes or abstractions? | Depend on abstractions, not concretions |

### Common Interview Questions

1. **What are SOLID principles?**
   - Five design principles for object-oriented programming
   - Make software more maintainable, flexible, and scalable
   - Introduced by Robert C. Martin (Uncle Bob)

2. **Why are SOLID principles important?**
   - Reduce code complexity
   - Improve code quality and maintainability
   - Enable easier testing
   - Support agile development and refactoring
   - Create more robust and scalable systems

3. **How do you apply SOLID in real projects?**
   - Start with SRP when designing classes
   - Use interfaces and abstract classes for OCP and DIP
   - Design inheritance hierarchies carefully for LSP
   - Keep interfaces focused for ISP
   - Use dependency injection for DIP

4. **What's the difference between OCP and LSP?**
   - OCP: About extending functionality without modification
   - LSP: About proper inheritance and substitutability

5. **How do DIP and ISP work together?**
   - DIP promotes depending on abstractions
   - ISP ensures those abstractions are focused and specific
   - Together they create flexible, decoupled designs

### Red Flags (Code Smells)

- **SRP Violation**: Classes with "and" in their name (UserAndOrderManager)
- **OCP Violation**: Multiple if-else or switch statements for type checking
- **LSP Violation**: Throwing UnsupportedOperationException in subclass
- **ISP Violation**: Empty or default implementations in interface methods
- **DIP Violation**: Using "new" keyword to instantiate dependencies directly

### Best Practices

1. **Start Simple**: Don't over-engineer - apply principles when complexity demands it
2. **Refactor Gradually**: Apply SOLID principles during refactoring, not always upfront
3. **Use Design Patterns**: Many patterns implement SOLID principles (Strategy, Factory, Observer)
4. **Write Tests**: SOLID principles make code more testable
5. **Code Reviews**: Check for SOLID violations during reviews
6. **Balance**: Don't sacrifice simplicity for strict adherence to principles

### Interview Success Tips

1. **Provide Examples**: Always back principles with concrete code examples
2. **Explain Trade-offs**: Discuss when to apply and when to skip certain principles
3. **Show Evolution**: Demonstrate refactoring from violation to compliance
4. **Connect to Real Projects**: Relate principles to actual project experiences
5. **Discuss Benefits**: Explain how SOLID improved code quality in your projects

---

## Conclusion

SOLID principles are foundational to writing clean, maintainable, and scalable object-oriented code. While they provide excellent guidelines, remember that they are principles, not laws. Apply them judiciously based on your project's needs and complexity.

**Key Takeaways:**
- **SRP**: Keep classes focused on a single responsibility
- **OCP**: Design for extension without modification
- **LSP**: Ensure proper inheritance relationships
- **ISP**: Create focused, client-specific interfaces
- **DIP**: Depend on abstractions, not concrete implementations

Mastering these principles will make you a better software developer and help you create more robust, flexible, and maintainable software systems.

---

**Remember**: The goal is not to follow these principles blindly, but to understand them deeply and apply them wisely to create better software.
