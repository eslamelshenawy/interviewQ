# OOP Interview Questions and Answers

## Table of Contents
1. [The Four Pillars of OOP](#the-four-pillars-of-oop)
2. [Classes and Objects](#classes-and-objects)
3. [Method Overloading vs Overriding](#method-overloading-vs-overriding)
4. [Abstract Classes vs Interfaces](#abstract-classes-vs-interfaces)
5. [Access Modifiers](#access-modifiers)
6. [Static vs Instance](#static-vs-instance)
7. [Constructors](#constructors)
8. [Coupling and Cohesion](#coupling-and-cohesion)
9. [Composition vs Aggregation](#composition-vs-aggregation)
10. [Advanced Questions](#advanced-questions)

---

# The Four Pillars of OOP

## Q1: What are the four pillars of OOP? Explain each with examples.

**Answer:**

### 1. Encapsulation

**Definition**: Bundling data and methods within a single unit (class) and hiding internal details from the outside world.

**Key Benefits**:
- Data hiding and protection
- Controlled access through getters/setters
- Flexibility to change implementation without affecting external code

**Example**:
```java
public class Employee {
    // Private variables - cannot be accessed directly
    private String name;
    private double salary;
    private int age;

    // Constructor
    public Employee(String name, double salary, int age) {
        this.name = name;
        setSalary(salary);  // Using setter for validation
        setAge(age);
    }

    // Getter for name
    public String getName() {
        return name;
    }

    // Setter for name
    public void setName(String name) {
        if (name != null && !name.trim().isEmpty()) {
            this.name = name;
        }
    }

    // Getter for salary
    public double getSalary() {
        return salary;
    }

    // Setter with validation
    public void setSalary(double salary) {
        if (salary >= 0) {
            this.salary = salary;
        } else {
            System.out.println("Invalid salary!");
        }
    }

    // Getter for age
    public int getAge() {
        return age;
    }

    // Setter with validation
    public void setAge(int age) {
        if (age > 0 && age < 150) {
            this.age = age;
        }
    }

    // Method that uses private data
    public void displayInfo() {
        System.out.println("Name: " + name);
        System.out.println("Age: " + age);
        System.out.println("Salary: $" + salary);
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        Employee emp = new Employee("John Doe", 50000, 30);

        // Cannot access directly: emp.salary = -1000; // Compilation error

        // Must use setter which validates the data
        emp.setSalary(-1000);  // Invalid salary! - validation works
        emp.setSalary(60000);  // Valid - updates successfully

        emp.displayInfo();
    }
}
```

### 2. Inheritance

**Definition**: A mechanism where one class acquires properties and behaviors of another class.

**Key Benefits**:
- Code reusability
- Method overriding (runtime polymorphism)
- Hierarchical classification

**Types**:
- Single Inheritance
- Multilevel Inheritance
- Hierarchical Inheritance
- (Multiple Inheritance - not supported in Java with classes, but supported with interfaces)

**Example**:
```java
// Parent class (Superclass)
public class Animal {
    protected String name;
    protected int age;

    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void eat() {
        System.out.println(name + " is eating");
    }

    public void sleep() {
        System.out.println(name + " is sleeping");
    }

    public void makeSound() {
        System.out.println("Animal makes a sound");
    }
}

// Child class (Subclass)
public class Dog extends Animal {
    private String breed;

    public Dog(String name, int age, String breed) {
        super(name, age);  // Call parent constructor
        this.breed = breed;
    }

    // Dog-specific method
    public void bark() {
        System.out.println(name + " barks: Woof! Woof!");
    }

    // Override parent method
    @Override
    public void makeSound() {
        System.out.println(name + " barks loudly!");
    }

    public void displayInfo() {
        System.out.println("Name: " + name);
        System.out.println("Age: " + age);
        System.out.println("Breed: " + breed);
    }
}

public class Cat extends Animal {
    private String furColor;

    public Cat(String name, int age, String furColor) {
        super(name, age);
        this.furColor = furColor;
    }

    public void meow() {
        System.out.println(name + " meows: Meow!");
    }

    @Override
    public void makeSound() {
        System.out.println(name + " meows softly!");
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        Dog dog = new Dog("Buddy", 5, "Golden Retriever");
        dog.eat();         // Inherited from Animal
        dog.sleep();       // Inherited from Animal
        dog.bark();        // Dog-specific method
        dog.makeSound();   // Overridden method

        Cat cat = new Cat("Whiskers", 3, "White");
        cat.eat();         // Inherited from Animal
        cat.meow();        // Cat-specific method
        cat.makeSound();   // Overridden method
    }
}
```

**Multilevel Inheritance Example**:
```java
public class LivingBeing {
    public void breathe() {
        System.out.println("Breathing...");
    }
}

public class Animal extends LivingBeing {
    public void eat() {
        System.out.println("Eating...");
    }
}

public class Dog extends Animal {
    public void bark() {
        System.out.println("Barking...");
    }
}

// Dog inherits from Animal, which inherits from LivingBeing
Dog dog = new Dog();
dog.breathe();  // From LivingBeing
dog.eat();      // From Animal
dog.bark();     // From Dog
```

### 3. Polymorphism

**Definition**: The ability of an object to take many forms. Same method name can behave differently based on context.

**Types**:
1. **Compile-time Polymorphism (Static)**: Method Overloading
2. **Runtime Polymorphism (Dynamic)**: Method Overriding

**Example - Method Overloading (Compile-time)**:
```java
public class Calculator {
    // Method to add two integers
    public int add(int a, int b) {
        return a + b;
    }

    // Method to add three integers
    public int add(int a, int b, int c) {
        return a + b + c;
    }

    // Method to add two doubles
    public double add(double a, double b) {
        return a + b;
    }

    // Method to concatenate strings
    public String add(String a, String b) {
        return a + b;
    }
}

// Usage
Calculator calc = new Calculator();
System.out.println(calc.add(5, 10));           // 15
System.out.println(calc.add(5, 10, 15));       // 30
System.out.println(calc.add(5.5, 10.5));       // 16.0
System.out.println(calc.add("Hello ", "World")); // Hello World
```

**Example - Method Overriding (Runtime)**:
```java
public class Shape {
    public void draw() {
        System.out.println("Drawing a shape");
    }

    public double calculateArea() {
        return 0.0;
    }
}

public class Circle extends Shape {
    private double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public void draw() {
        System.out.println("Drawing a circle");
    }

    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

public class Rectangle extends Shape {
    private double width;
    private double height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public void draw() {
        System.out.println("Drawing a rectangle");
    }

    @Override
    public double calculateArea() {
        return width * height;
    }
}

// Polymorphic behavior
public class Main {
    public static void main(String[] args) {
        Shape[] shapes = new Shape[3];
        shapes[0] = new Circle(5.0);
        shapes[1] = new Rectangle(4.0, 6.0);
        shapes[2] = new Shape();

        for (Shape shape : shapes) {
            shape.draw();  // Different behavior based on actual object
            System.out.println("Area: " + shape.calculateArea());
        }
    }
}
```

**Real-world Polymorphism Example**:
```java
public class PaymentProcessor {
    public void processPayment(Payment payment, double amount) {
        payment.pay(amount);  // Polymorphic call
    }
}

public interface Payment {
    void pay(double amount);
}

public class CreditCardPayment implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Processing credit card payment of $" + amount);
    }
}

public class PayPalPayment implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Processing PayPal payment of $" + amount);
    }
}

public class CashPayment implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Processing cash payment of $" + amount);
    }
}

// Usage
PaymentProcessor processor = new PaymentProcessor();
processor.processPayment(new CreditCardPayment(), 100.0);
processor.processPayment(new PayPalPayment(), 200.0);
processor.processPayment(new CashPayment(), 50.0);
```

### 4. Abstraction

**Definition**: Hiding implementation details and showing only essential features to the user.

**Achieved through**:
- Abstract classes
- Interfaces

**Key Benefits**:
- Reduces complexity
- Hides implementation details
- Provides a clear interface

**Example - Abstract Class**:
```java
public abstract class BankAccount {
    protected String accountNumber;
    protected double balance;

    public BankAccount(String accountNumber, double initialBalance) {
        this.accountNumber = accountNumber;
        this.balance = initialBalance;
    }

    // Abstract methods - must be implemented by subclasses
    public abstract void deposit(double amount);
    public abstract boolean withdraw(double amount);
    public abstract void calculateInterest();

    // Concrete method - shared by all subclasses
    public void displayBalance() {
        System.out.println("Account: " + accountNumber);
        System.out.println("Balance: $" + balance);
    }

    public double getBalance() {
        return balance;
    }
}

public class SavingsAccount extends BankAccount {
    private double interestRate;

    public SavingsAccount(String accountNumber, double initialBalance, double interestRate) {
        super(accountNumber, initialBalance);
        this.interestRate = interestRate;
    }

    @Override
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            System.out.println("Deposited $" + amount);
        }
    }

    @Override
    public boolean withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            System.out.println("Withdrawn $" + amount);
            return true;
        }
        System.out.println("Insufficient funds!");
        return false;
    }

    @Override
    public void calculateInterest() {
        double interest = balance * interestRate / 100;
        balance += interest;
        System.out.println("Interest added: $" + interest);
    }
}

public class CurrentAccount extends BankAccount {
    private double overdraftLimit;

    public CurrentAccount(String accountNumber, double initialBalance, double overdraftLimit) {
        super(accountNumber, initialBalance);
        this.overdraftLimit = overdraftLimit;
    }

    @Override
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            System.out.println("Deposited $" + amount);
        }
    }

    @Override
    public boolean withdraw(double amount) {
        if (amount > 0 && (balance + overdraftLimit) >= amount) {
            balance -= amount;
            System.out.println("Withdrawn $" + amount);
            return true;
        }
        System.out.println("Exceeds overdraft limit!");
        return false;
    }

    @Override
    public void calculateInterest() {
        // No interest for current account
        System.out.println("No interest for current account");
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        BankAccount savings = new SavingsAccount("SA001", 1000, 5.0);
        savings.deposit(500);
        savings.withdraw(200);
        savings.calculateInterest();
        savings.displayBalance();

        BankAccount current = new CurrentAccount("CA001", 500, 1000);
        current.deposit(300);
        current.withdraw(1500);  // Uses overdraft
        current.displayBalance();
    }
}
```

---

# Classes and Objects

## Q2: What is the difference between a class and an object?

**Answer:**

| Class | Object |
|-------|--------|
| Blueprint/template for creating objects | Instance of a class |
| Logical entity | Physical entity |
| Declared once | Can create multiple objects |
| No memory allocated when created | Memory allocated when created |
| Defines properties and methods | Has actual values for properties |

**Example**:
```java
// Class - Blueprint
public class Car {
    // Properties (attributes)
    private String brand;
    private String model;
    private String color;
    private int year;
    private double price;

    // Constructor
    public Car(String brand, String model, String color, int year, double price) {
        this.brand = brand;
        this.model = model;
        this.color = color;
        this.year = year;
        this.price = price;
    }

    // Methods (behaviors)
    public void start() {
        System.out.println(brand + " " + model + " is starting...");
    }

    public void drive() {
        System.out.println("Driving the " + color + " " + brand);
    }

    public void displayInfo() {
        System.out.println("Brand: " + brand);
        System.out.println("Model: " + model);
        System.out.println("Color: " + color);
        System.out.println("Year: " + year);
        System.out.println("Price: $" + price);
    }
}

// Creating Objects - Instances of the Car class
public class Main {
    public static void main(String[] args) {
        // Object 1
        Car car1 = new Car("Toyota", "Camry", "Blue", 2023, 25000);
        car1.start();
        car1.drive();
        car1.displayInfo();

        System.out.println("\n---\n");

        // Object 2 - Different instance with different values
        Car car2 = new Car("Honda", "Civic", "Red", 2024, 23000);
        car2.start();
        car2.drive();
        car2.displayInfo();

        // Object 3
        Car car3 = new Car("Tesla", "Model 3", "White", 2024, 45000);
    }
}
```

## Q3: What is a class? What are its components?

**Answer:**

A class is a blueprint or template that defines the structure and behavior of objects.

**Components of a Class**:

1. **Fields/Attributes**: Variables that hold the state
2. **Methods**: Functions that define behavior
3. **Constructors**: Special methods to initialize objects
4. **Blocks**: Static and instance initialization blocks
5. **Nested Classes**: Classes defined within a class
6. **Access Modifiers**: Control visibility

**Comprehensive Example**:
```java
public class Student {
    // 1. FIELDS/ATTRIBUTES
    // Instance variables
    private String name;
    private int rollNumber;
    private double gpa;

    // Static variable (class variable)
    private static int studentCount = 0;
    private static String universityName = "XYZ University";

    // Constant
    private static final int MAX_GPA = 4;

    // 2. STATIC INITIALIZATION BLOCK
    static {
        System.out.println("Static block executed");
        universityName = "XYZ University";
    }

    // 3. INSTANCE INITIALIZATION BLOCK
    {
        System.out.println("Instance block executed");
        studentCount++;
    }

    // 4. CONSTRUCTORS
    // Default constructor
    public Student() {
        this.name = "Unknown";
        this.rollNumber = 0;
        this.gpa = 0.0;
    }

    // Parameterized constructor
    public Student(String name, int rollNumber, double gpa) {
        this.name = name;
        this.rollNumber = rollNumber;
        setGpa(gpa);  // Using setter for validation
    }

    // Copy constructor
    public Student(Student other) {
        this.name = other.name;
        this.rollNumber = other.rollNumber;
        this.gpa = other.gpa;
    }

    // 5. METHODS
    // Getter methods
    public String getName() {
        return name;
    }

    public int getRollNumber() {
        return rollNumber;
    }

    public double getGpa() {
        return gpa;
    }

    // Setter methods with validation
    public void setName(String name) {
        if (name != null && !name.trim().isEmpty()) {
            this.name = name;
        }
    }

    public void setGpa(double gpa) {
        if (gpa >= 0 && gpa <= MAX_GPA) {
            this.gpa = gpa;
        }
    }

    // Instance method
    public void study(String subject) {
        System.out.println(name + " is studying " + subject);
    }

    // Instance method
    public void displayInfo() {
        System.out.println("Name: " + name);
        System.out.println("Roll Number: " + rollNumber);
        System.out.println("GPA: " + gpa);
        System.out.println("University: " + universityName);
    }

    // Static method
    public static int getStudentCount() {
        return studentCount;
    }

    public static void setUniversityName(String name) {
        universityName = name;
    }

    // 6. NESTED CLASS
    public class Address {
        private String street;
        private String city;

        public Address(String street, String city) {
            this.street = street;
            this.city = city;
        }

        public void displayAddress() {
            System.out.println("Student: " + name);  // Can access outer class members
            System.out.println("Address: " + street + ", " + city);
        }
    }

    // 7. OVERRIDDEN METHODS
    @Override
    public String toString() {
        return "Student{name='" + name + "', rollNumber=" + rollNumber + ", gpa=" + gpa + "}";
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Student student = (Student) obj;
        return rollNumber == student.rollNumber;
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        Student s1 = new Student("Alice", 101, 3.8);
        s1.displayInfo();
        s1.study("Mathematics");

        Student s2 = new Student("Bob", 102, 3.5);

        System.out.println("Total students: " + Student.getStudentCount());
    }
}
```

---

# Method Overloading vs Overriding

## Q4: What is the difference between method overloading and method overriding?

**Answer:**

| Aspect | Method Overloading | Method Overriding |
|--------|-------------------|-------------------|
| Definition | Multiple methods with same name but different parameters | Subclass provides specific implementation of superclass method |
| Polymorphism Type | Compile-time (Static) | Runtime (Dynamic) |
| Class Requirement | Same class | Different classes (inheritance required) |
| Method Signature | Must be different | Must be same |
| Return Type | Can be different | Must be same (or covariant) |
| Access Modifier | Can be any | Must be same or less restrictive |
| Binding | Early binding (compile-time) | Late binding (runtime) |
| Performance | Faster | Slightly slower (due to runtime resolution) |

**Method Overloading Example**:
```java
public class MathOperations {
    // Method 1: Add two integers
    public int add(int a, int b) {
        System.out.println("Adding two integers");
        return a + b;
    }

    // Method 2: Add three integers
    public int add(int a, int b, int c) {
        System.out.println("Adding three integers");
        return a + b + c;
    }

    // Method 3: Add two doubles
    public double add(double a, double b) {
        System.out.println("Adding two doubles");
        return a + b;
    }

    // Method 4: Add two strings (concatenation)
    public String add(String a, String b) {
        System.out.println("Concatenating two strings");
        return a + b;
    }

    // Method 5: Add array of integers
    public int add(int[] numbers) {
        System.out.println("Adding array of integers");
        int sum = 0;
        for (int num : numbers) {
            sum += num;
        }
        return sum;
    }

    // Method 6: Variable arguments
    public int add(int... numbers) {
        System.out.println("Adding variable arguments");
        int sum = 0;
        for (int num : numbers) {
            sum += num;
        }
        return sum;
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        MathOperations math = new MathOperations();

        System.out.println(math.add(5, 10));              // Calls Method 1
        System.out.println(math.add(5, 10, 15));          // Calls Method 2
        System.out.println(math.add(5.5, 10.5));          // Calls Method 3
        System.out.println(math.add("Hello", "World"));   // Calls Method 4
        System.out.println(math.add(new int[]{1,2,3,4})); // Calls Method 5
        System.out.println(math.add(1, 2, 3, 4, 5));      // Calls Method 6
    }
}
```

**Method Overriding Example**:
```java
// Parent class
public class Animal {
    protected String name;

    public Animal(String name) {
        this.name = name;
    }

    // Method to be overridden
    public void makeSound() {
        System.out.println("Animal makes a sound");
    }

    public void eat() {
        System.out.println(name + " is eating");
    }

    public void sleep() {
        System.out.println(name + " is sleeping");
    }
}

// Child class 1
public class Dog extends Animal {
    public Dog(String name) {
        super(name);
    }

    // Overriding makeSound method
    @Override
    public void makeSound() {
        System.out.println(name + " says: Woof! Woof!");
    }

    // Overriding eat method
    @Override
    public void eat() {
        System.out.println(name + " is eating dog food");
    }

    // Dog-specific method (not overriding)
    public void fetch() {
        System.out.println(name + " is fetching the ball");
    }
}

// Child class 2
public class Cat extends Animal {
    public Cat(String name) {
        super(name);
    }

    @Override
    public void makeSound() {
        System.out.println(name + " says: Meow!");
    }

    @Override
    public void eat() {
        System.out.println(name + " is eating cat food");
    }

    public void scratch() {
        System.out.println(name + " is scratching");
    }
}

// Child class 3
public class Cow extends Animal {
    public Cow(String name) {
        super(name);
    }

    @Override
    public void makeSound() {
        System.out.println(name + " says: Moo!");
    }
}

// Usage demonstrating polymorphism
public class Main {
    public static void main(String[] args) {
        // Runtime polymorphism
        Animal animal1 = new Dog("Buddy");
        Animal animal2 = new Cat("Whiskers");
        Animal animal3 = new Cow("Bessie");

        // Same method call, different behaviors
        animal1.makeSound();  // Woof! Woof!
        animal2.makeSound();  // Meow!
        animal3.makeSound();  // Moo!

        animal1.eat();  // eating dog food
        animal2.eat();  // eating cat food
        animal3.eat();  // eating (default implementation from Animal)

        // Array of animals
        Animal[] animals = {
            new Dog("Max"),
            new Cat("Luna"),
            new Cow("Daisy"),
            new Dog("Charlie")
        };

        System.out.println("\nAll animals making sounds:");
        for (Animal animal : animals) {
            animal.makeSound();  // Polymorphic behavior
        }
    }
}
```

**Real-world Example - Payment Processing**:
```java
public class PaymentProcessor {
    // Overloaded methods
    public void processPayment(double amount) {
        System.out.println("Processing payment of $" + amount);
    }

    public void processPayment(double amount, String currency) {
        System.out.println("Processing payment of " + amount + " " + currency);
    }

    public void processPayment(double amount, String currency, String paymentMethod) {
        System.out.println("Processing " + paymentMethod + " payment of " + amount + " " + currency);
    }
}

// Parent class
public class Employee {
    protected String name;
    protected double baseSalary;

    public Employee(String name, double baseSalary) {
        this.name = name;
        this.baseSalary = baseSalary;
    }

    // Method to be overridden
    public double calculateSalary() {
        return baseSalary;
    }

    public void displayInfo() {
        System.out.println("Name: " + name);
        System.out.println("Salary: $" + calculateSalary());
    }
}

public class Manager extends Employee {
    private double bonus;

    public Manager(String name, double baseSalary, double bonus) {
        super(name, baseSalary);
        this.bonus = bonus;
    }

    @Override
    public double calculateSalary() {
        return baseSalary + bonus;
    }
}

public class Developer extends Employee {
    private int projectsCompleted;
    private double projectBonus;

    public Developer(String name, double baseSalary, int projectsCompleted, double projectBonus) {
        super(name, baseSalary);
        this.projectsCompleted = projectsCompleted;
        this.projectBonus = projectBonus;
    }

    @Override
    public double calculateSalary() {
        return baseSalary + (projectsCompleted * projectBonus);
    }
}

// Usage
Employee emp1 = new Employee("John", 50000);
Employee emp2 = new Manager("Sarah", 60000, 10000);
Employee emp3 = new Developer("Mike", 55000, 5, 1000);

emp1.displayInfo();  // Salary: 50000
emp2.displayInfo();  // Salary: 70000 (base + bonus)
emp3.displayInfo();  // Salary: 60000 (base + 5*1000)
```

## Q5: Can we overload the main method? Can we override it?

**Answer:**

**Overloading main method**: YES
**Overriding main method**: NO (main is static, and static methods cannot be overridden)

```java
public class MainOverloadingExample {
    // Standard main method - Entry point
    public static void main(String[] args) {
        System.out.println("Standard main method");

        // Calling overloaded main methods
        main(10);
        main("Hello");
        main(5, 10);
    }

    // Overloaded main method with int parameter
    public static void main(int num) {
        System.out.println("Overloaded main with int: " + num);
    }

    // Overloaded main method with String parameter
    public static void main(String message) {
        System.out.println("Overloaded main with String: " + message);
    }

    // Overloaded main method with two int parameters
    public static void main(int a, int b) {
        System.out.println("Overloaded main with two ints: " + a + ", " + b);
    }
}
```

---

# Abstract Classes vs Interfaces

## Q6: What is the difference between Abstract Class and Interface?

**Answer:**

| Aspect | Abstract Class | Interface |
|--------|---------------|-----------|
| Methods | Can have both abstract and concrete methods | All methods are abstract by default (before Java 8) |
| Variables | Can have any type of variables | Only public static final (constants) |
| Multiple Inheritance | Cannot extend multiple abstract classes | Can implement multiple interfaces |
| Constructor | Can have constructors | Cannot have constructors |
| Access Modifiers | Can have any access modifier | Methods are public by default |
| Keyword | Uses `extends` | Uses `implements` |
| When to Use | When classes share common code | When classes share only method signatures |
| Java 8+ Features | Can have default and static methods | Can have default, static, and private methods |
| Inheritance | Represents "is-a" relationship | Represents "can-do" relationship |

**Abstract Class Example**:
```java
public abstract class Vehicle {
    // Instance variables
    protected String brand;
    protected String model;
    protected int year;

    // Constructor
    public Vehicle(String brand, String model, int year) {
        this.brand = brand;
        this.model = model;
        this.year = year;
    }

    // Abstract methods - must be implemented by subclasses
    public abstract void start();
    public abstract void stop();
    public abstract double calculateFuelEfficiency();

    // Concrete method - shared by all subclasses
    public void displayInfo() {
        System.out.println("Brand: " + brand);
        System.out.println("Model: " + model);
        System.out.println("Year: " + year);
    }

    // Concrete method with logic
    public void honk() {
        System.out.println("Beep! Beep!");
    }

    // Getters
    public String getBrand() {
        return brand;
    }
}

public class Car extends Vehicle {
    private int numberOfDoors;

    public Car(String brand, String model, int year, int numberOfDoors) {
        super(brand, model, year);
        this.numberOfDoors = numberOfDoors;
    }

    @Override
    public void start() {
        System.out.println("Car starting with ignition key");
    }

    @Override
    public void stop() {
        System.out.println("Car stopping with brake pedal");
    }

    @Override
    public double calculateFuelEfficiency() {
        return 15.5;  // km per liter
    }
}

public class Motorcycle extends Vehicle {
    private boolean hasSidecar;

    public Motorcycle(String brand, String model, int year, boolean hasSidecar) {
        super(brand, model, year);
        this.hasSidecar = hasSidecar;
    }

    @Override
    public void start() {
        System.out.println("Motorcycle starting with kick/button");
    }

    @Override
    public void stop() {
        System.out.println("Motorcycle stopping with hand brake");
    }

    @Override
    public double calculateFuelEfficiency() {
        return 35.0;  // km per liter
    }
}
```

**Interface Example**:
```java
// Interface 1
public interface Drivable {
    // Abstract methods (public abstract by default)
    void drive();
    void brake();
    void turn(String direction);

    // Default method (Java 8+)
    default void park() {
        System.out.println("Vehicle is parking");
    }

    // Static method (Java 8+)
    static void displayRules() {
        System.out.println("Follow traffic rules");
    }
}

// Interface 2
public interface Electric {
    void chargeBattery();
    int getBatteryLevel();

    default void displayBatteryStatus() {
        System.out.println("Battery level: " + getBatteryLevel() + "%");
    }
}

// Interface 3
public interface GPS {
    void setDestination(String destination);
    String getCurrentLocation();
    void navigate();
}

// Class implementing multiple interfaces
public class TeslaCar implements Drivable, Electric, GPS {
    private int batteryLevel;
    private String currentLocation;

    public TeslaCar() {
        this.batteryLevel = 100;
        this.currentLocation = "Home";
    }

    // Implementing Drivable interface
    @Override
    public void drive() {
        System.out.println("Tesla is driving silently");
        batteryLevel -= 5;
    }

    @Override
    public void brake() {
        System.out.println("Regenerative braking activated");
        batteryLevel += 1;  // Regenerative braking charges battery
    }

    @Override
    public void turn(String direction) {
        System.out.println("Turning " + direction);
    }

    // Implementing Electric interface
    @Override
    public void chargeBattery() {
        System.out.println("Charging battery...");
        batteryLevel = 100;
    }

    @Override
    public int getBatteryLevel() {
        return batteryLevel;
    }

    // Implementing GPS interface
    @Override
    public void setDestination(String destination) {
        System.out.println("Destination set to: " + destination);
    }

    @Override
    public String getCurrentLocation() {
        return currentLocation;
    }

    @Override
    public void navigate() {
        System.out.println("Navigating from " + currentLocation);
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        TeslaCar tesla = new TeslaCar();

        // Using Drivable methods
        tesla.drive();
        tesla.turn("left");
        tesla.brake();
        tesla.park();  // Default method

        // Using Electric methods
        tesla.displayBatteryStatus();
        if (tesla.getBatteryLevel() < 20) {
            tesla.chargeBattery();
        }

        // Using GPS methods
        tesla.setDestination("Office");
        tesla.navigate();

        // Using static method
        Drivable.displayRules();
    }
}
```

**When to Use Which?**

**Use Abstract Class when**:
```java
// Shared code and state among related classes
public abstract class DatabaseConnection {
    protected String connectionString;
    protected boolean isConnected;

    public DatabaseConnection(String connectionString) {
        this.connectionString = connectionString;
        this.isConnected = false;
    }

    // Common logic
    public void connect() {
        System.out.println("Connecting to: " + connectionString);
        isConnected = true;
    }

    public void disconnect() {
        isConnected = false;
        System.out.println("Disconnected");
    }

    // Abstract - specific to database type
    public abstract void executeQuery(String query);
    public abstract void executeUpdate(String query);
}

public class MySQLConnection extends DatabaseConnection {
    public MySQLConnection(String connectionString) {
        super(connectionString);
    }

    @Override
    public void executeQuery(String query) {
        System.out.println("Executing MySQL query: " + query);
    }

    @Override
    public void executeUpdate(String query) {
        System.out.println("Executing MySQL update: " + query);
    }
}
```

**Use Interface when**:
```java
// Defining capabilities that can be applied to unrelated classes
public interface Sortable {
    void sort();
}

public interface Searchable {
    boolean search(String keyword);
}

// Different classes implementing same interfaces
public class Library implements Sortable, Searchable {
    @Override
    public void sort() {
        System.out.println("Sorting books alphabetically");
    }

    @Override
    public boolean search(String keyword) {
        System.out.println("Searching for: " + keyword);
        return true;
    }
}

public class ProductCatalog implements Sortable, Searchable {
    @Override
    public void sort() {
        System.out.println("Sorting products by price");
    }

    @Override
    public boolean search(String keyword) {
        System.out.println("Searching products: " + keyword);
        return true;
    }
}
```

## Q7: Can an interface extend another interface? Can a class extend an interface?

**Answer:**

**Interface extending Interface**: YES (using `extends`)
**Class extending Interface**: NO (class uses `implements`, not `extends`)

```java
// Interface extending another interface
public interface Animal {
    void eat();
    void sleep();
}

public interface Mammal extends Animal {
    void giveBirth();
    void produceMilk();
}

public interface Flyable {
    void fly();
}

// Interface extending multiple interfaces
public interface Bird extends Animal, Flyable {
    void layEggs();
    void buildNest();
}

// Class implementing the interface
public class Eagle implements Bird {
    @Override
    public void eat() {
        System.out.println("Eagle eats prey");
    }

    @Override
    public void sleep() {
        System.out.println("Eagle sleeps in nest");
    }

    @Override
    public void fly() {
        System.out.println("Eagle flies high");
    }

    @Override
    public void layEggs() {
        System.out.println("Eagle lays eggs");
    }

    @Override
    public void buildNest() {
        System.out.println("Eagle builds nest on cliff");
    }
}

public class Dog implements Mammal {
    @Override
    public void eat() {
        System.out.println("Dog eats food");
    }

    @Override
    public void sleep() {
        System.out.println("Dog sleeps on bed");
    }

    @Override
    public void giveBirth() {
        System.out.println("Dog gives birth to puppies");
    }

    @Override
    public void produceMilk() {
        System.out.println("Dog produces milk");
    }
}
```

---

# Access Modifiers

## Q8: Explain different access modifiers in Java with examples.

**Answer:**

Java has 4 access modifiers:

| Modifier | Class | Package | Subclass | World |
|----------|-------|---------|----------|-------|
| public | Yes | Yes | Yes | Yes |
| protected | Yes | Yes | Yes | No |
| default (no modifier) | Yes | Yes | No | No |
| private | Yes | No | No | No |

**Comprehensive Example**:

```java
// File: AccessModifierDemo.java
package com.example.main;

public class AccessModifierDemo {
    // Public - accessible everywhere
    public String publicField = "Public Field";

    // Protected - accessible in same package and subclasses
    protected String protectedField = "Protected Field";

    // Default (no modifier) - accessible only in same package
    String defaultField = "Default Field";

    // Private - accessible only within this class
    private String privateField = "Private Field";

    // Public method
    public void publicMethod() {
        System.out.println("Public Method");
    }

    // Protected method
    protected void protectedMethod() {
        System.out.println("Protected Method");
    }

    // Default method
    void defaultMethod() {
        System.out.println("Default Method");
    }

    // Private method
    private void privateMethod() {
        System.out.println("Private Method");
    }

    // Public method that can access all members
    public void testAccess() {
        System.out.println(publicField);     // OK
        System.out.println(protectedField);  // OK
        System.out.println(defaultField);    // OK
        System.out.println(privateField);    // OK

        publicMethod();    // OK
        protectedMethod(); // OK
        defaultMethod();   // OK
        privateMethod();   // OK
    }
}

// Same package - different class
package com.example.main;

public class SamePackageClass {
    public void accessTest() {
        AccessModifierDemo obj = new AccessModifierDemo();

        System.out.println(obj.publicField);    // OK
        System.out.println(obj.protectedField); // OK
        System.out.println(obj.defaultField);   // OK
        // System.out.println(obj.privateField); // ERROR - not accessible

        obj.publicMethod();    // OK
        obj.protectedMethod(); // OK
        obj.defaultMethod();   // OK
        // obj.privateMethod(); // ERROR - not accessible
    }
}

// Different package - subclass
package com.example.other;

import com.example.main.AccessModifierDemo;

public class SubclassInDifferentPackage extends AccessModifierDemo {
    public void accessTest() {
        System.out.println(publicField);    // OK
        System.out.println(protectedField); // OK - accessible in subclass
        // System.out.println(defaultField);  // ERROR - not accessible
        // System.out.println(privateField);  // ERROR - not accessible

        publicMethod();    // OK
        protectedMethod(); // OK - accessible in subclass
        // defaultMethod();  // ERROR - not accessible
        // privateMethod();  // ERROR - not accessible
    }
}

// Different package - non-subclass
package com.example.other;

import com.example.main.AccessModifierDemo;

public class DifferentPackageClass {
    public void accessTest() {
        AccessModifierDemo obj = new AccessModifierDemo();

        System.out.println(obj.publicField);    // OK - only public accessible
        // System.out.println(obj.protectedField); // ERROR
        // System.out.println(obj.defaultField);   // ERROR
        // System.out.println(obj.privateField);   // ERROR

        obj.publicMethod();    // OK - only public accessible
        // obj.protectedMethod(); // ERROR
        // obj.defaultMethod();   // ERROR
        // obj.privateMethod();   // ERROR
    }
}
```

**Real-world Example - Banking System**:
```java
public class BankAccount {
    // Private - sensitive data hidden
    private String accountNumber;
    private double balance;
    private String pin;

    // Protected - accessible to subclasses
    protected String accountHolderName;
    protected String accountType;

    // Public - accessible to everyone
    public String bankName;

    // Default - accessible within package
    String branchCode;

    // Public constructor
    public BankAccount(String accountNumber, String accountHolderName, String pin) {
        this.accountNumber = accountNumber;
        this.accountHolderName = accountHolderName;
        this.pin = pin;
        this.balance = 0.0;
        this.bankName = "ABC Bank";
    }

    // Public method - anyone can check balance (with authentication)
    public double getBalance(String enteredPin) {
        if (validatePin(enteredPin)) {
            return balance;
        }
        System.out.println("Invalid PIN");
        return -1;
    }

    // Public method - anyone can deposit
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            System.out.println("Deposited: $" + amount);
        }
    }

    // Public method - withdraw with authentication
    public boolean withdraw(double amount, String enteredPin) {
        if (validatePin(enteredPin) && balance >= amount) {
            balance -= amount;
            System.out.println("Withdrawn: $" + amount);
            return true;
        }
        return false;
    }

    // Private method - internal validation
    private boolean validatePin(String enteredPin) {
        return this.pin.equals(enteredPin);
    }

    // Protected method - accessible to subclasses
    protected void updateAccountType(String newType) {
        this.accountType = newType;
    }

    // Default method - accessible within package
    void setBranchCode(String code) {
        this.branchCode = code;
    }
}

public class SavingsAccount extends BankAccount {
    private double interestRate;

    public SavingsAccount(String accountNumber, String accountHolderName, String pin) {
        super(accountNumber, accountHolderName, pin);
        updateAccountType("Savings");  // Can access protected method
        this.interestRate = 5.0;
    }

    public void addInterest() {
        // Can access protected field
        System.out.println("Adding interest for " + accountHolderName);
        // Cannot access private balance directly
        // Would need to use public methods
    }
}
```

---

# Static vs Instance

## Q9: What is the difference between static and instance variables/methods?

**Answer:**

| Aspect | Static (Class) | Instance (Object) |
|--------|---------------|-------------------|
| Belongs to | Class | Individual object |
| Memory | Single copy shared by all instances | Separate copy for each object |
| Access | ClassName.memberName or object.memberName | object.memberName |
| Initialization | When class is loaded | When object is created |
| Can access | Only static members directly | Both static and instance members |
| this keyword | Cannot use `this` | Can use `this` |
| Use case | Shared data, utility methods | Object-specific data and behavior |

**Comprehensive Example**:
```java
public class Employee {
    // Static variables (class variables) - shared by all instances
    private static int employeeCount = 0;
    private static String companyName = "ABC Corp";
    private static final double TAX_RATE = 0.2;  // Constant

    // Instance variables - unique to each object
    private int employeeId;
    private String name;
    private double salary;
    private String department;

    // Static initialization block - runs once when class is loaded
    static {
        System.out.println("Static block executed - Class loaded");
        companyName = "ABC Corporation";
    }

    // Instance initialization block - runs every time object is created
    {
        System.out.println("Instance block executed - Object created");
        employeeCount++;
        this.employeeId = employeeCount;
    }

    // Constructor
    public Employee(String name, double salary, String department) {
        this.name = name;
        this.salary = salary;
        this.department = department;
    }

    // Instance method - can access both static and instance members
    public void displayInfo() {
        System.out.println("Employee ID: " + employeeId);  // Instance variable
        System.out.println("Name: " + name);               // Instance variable
        System.out.println("Salary: $" + salary);          // Instance variable
        System.out.println("Department: " + department);   // Instance variable
        System.out.println("Company: " + companyName);     // Static variable
        System.out.println("Tax: $" + calculateTax());     // Instance method
    }

    // Instance method
    public double calculateTax() {
        return salary * TAX_RATE;
    }

    // Instance method
    public double getNetSalary() {
        return salary - calculateTax();
    }

    // Static method - can only access static members directly
    public static void displayCompanyInfo() {
        System.out.println("Company Name: " + companyName);
        System.out.println("Total Employees: " + employeeCount);
        System.out.println("Tax Rate: " + (TAX_RATE * 100) + "%");

        // Cannot access instance variables directly
        // System.out.println(name);  // ERROR
        // System.out.println(salary); // ERROR

        // Cannot call instance methods directly
        // calculateTax();  // ERROR
    }

    // Static method
    public static int getEmployeeCount() {
        return employeeCount;
    }

    // Static method
    public static void setCompanyName(String name) {
        companyName = name;
    }

    // Static method - utility method
    public static double calculateAnnualTax(double monthlySalary) {
        return monthlySalary * 12 * TAX_RATE;
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        // Accessing static members without creating objects
        System.out.println("Initial employee count: " + Employee.getEmployeeCount());
        Employee.displayCompanyInfo();

        System.out.println("\n--- Creating Employees ---\n");

        // Creating objects
        Employee emp1 = new Employee("John Doe", 5000, "IT");
        Employee emp2 = new Employee("Jane Smith", 6000, "HR");
        Employee emp3 = new Employee("Bob Johnson", 5500, "Finance");

        // Instance method calls
        emp1.displayInfo();
        System.out.println("Net Salary: $" + emp1.getNetSalary());

        System.out.println();

        emp2.displayInfo();
        System.out.println();

        // Static method call
        System.out.println("Total Employees: " + Employee.getEmployeeCount());
        Employee.displayCompanyInfo();

        // Changing static variable affects all instances
        Employee.setCompanyName("XYZ Corporation");

        System.out.println("\n--- After Company Name Change ---\n");
        emp1.displayInfo();  // Will show new company name
        emp2.displayInfo();  // Will show new company name

        // Static utility method
        double annualTax = Employee.calculateAnnualTax(5000);
        System.out.println("Annual tax for $5000/month: $" + annualTax);
    }
}
```

**Real-world Example - Database Connection Pool**:
```java
public class DatabaseConnectionPool {
    // Static - shared connection pool
    private static DatabaseConnectionPool instance;
    private static final int MAX_CONNECTIONS = 10;
    private static int activeConnections = 0;

    // Instance variable
    private List<Connection> connections;

    // Private constructor - Singleton pattern
    private DatabaseConnectionPool() {
        connections = new ArrayList<>();
        initializePool();
    }

    // Static method to get single instance
    public static DatabaseConnectionPool getInstance() {
        if (instance == null) {
            synchronized (DatabaseConnectionPool.class) {
                if (instance == null) {
                    instance = new DatabaseConnectionPool();
                }
            }
        }
        return instance;
    }

    private void initializePool() {
        for (int i = 0; i < MAX_CONNECTIONS; i++) {
            connections.add(createNewConnection());
        }
    }

    private Connection createNewConnection() {
        activeConnections++;
        return new Connection("Connection-" + activeConnections);
    }

    public Connection getConnection() {
        if (!connections.isEmpty()) {
            return connections.remove(0);
        }
        return null;
    }

    public void releaseConnection(Connection conn) {
        connections.add(conn);
    }

    public static int getActiveConnections() {
        return activeConnections;
    }
}

class Connection {
    private String id;

    public Connection(String id) {
        this.id = id;
    }

    public String getId() {
        return id;
    }
}

// Usage - all parts of application share same pool
DatabaseConnectionPool pool1 = DatabaseConnectionPool.getInstance();
DatabaseConnectionPool pool2 = DatabaseConnectionPool.getInstance();
// pool1 and pool2 refer to the same object
```

**Static Method Examples - Utility Classes**:
```java
public class MathUtils {
    // All static - utility class
    private MathUtils() {
        // Private constructor prevents instantiation
    }

    public static int add(int a, int b) {
        return a + b;
    }

    public static int max(int a, int b) {
        return (a > b) ? a : b;
    }

    public static boolean isPrime(int num) {
        if (num <= 1) return false;
        for (int i = 2; i <= Math.sqrt(num); i++) {
            if (num % i == 0) return false;
        }
        return true;
    }

    public static int factorial(int n) {
        if (n <= 1) return 1;
        return n * factorial(n - 1);
    }
}

// Usage - no need to create objects
int sum = MathUtils.add(5, 10);
int maximum = MathUtils.max(20, 15);
boolean prime = MathUtils.isPrime(17);
```

---

# Constructors

## Q10: What is a constructor? What are the types of constructors?

**Answer:**

A **constructor** is a special method that is called when an object is created. It initializes the object's state.

**Characteristics**:
- Same name as class
- No return type (not even void)
- Automatically called when object is created
- Can be overloaded
- Cannot be static, final, or abstract

**Types of Constructors**:

1. **Default Constructor** (No-argument)
2. **Parameterized Constructor**
3. **Copy Constructor**

**Comprehensive Example**:
```java
public class Student {
    private String name;
    private int rollNumber;
    private double gpa;
    private String email;

    // 1. Default Constructor (No-argument)
    public Student() {
        System.out.println("Default constructor called");
        this.name = "Unknown";
        this.rollNumber = 0;
        this.gpa = 0.0;
        this.email = "not@provided.com";
    }

    // 2. Parameterized Constructor with 2 parameters
    public Student(String name, int rollNumber) {
        System.out.println("2-parameter constructor called");
        this.name = name;
        this.rollNumber = rollNumber;
        this.gpa = 0.0;
        this.email = "not@provided.com";
    }

    // 3. Parameterized Constructor with 3 parameters
    public Student(String name, int rollNumber, double gpa) {
        System.out.println("3-parameter constructor called");
        this.name = name;
        this.rollNumber = rollNumber;
        this.gpa = gpa;
        this.email = "not@provided.com";
    }

    // 4. Parameterized Constructor with all parameters
    public Student(String name, int rollNumber, double gpa, String email) {
        System.out.println("4-parameter constructor called");
        this.name = name;
        this.rollNumber = rollNumber;
        this.gpa = gpa;
        this.email = email;
    }

    // 5. Copy Constructor
    public Student(Student other) {
        System.out.println("Copy constructor called");
        this.name = other.name;
        this.rollNumber = other.rollNumber;
        this.gpa = other.gpa;
        this.email = other.email;
    }

    public void displayInfo() {
        System.out.println("Name: " + name);
        System.out.println("Roll Number: " + rollNumber);
        System.out.println("GPA: " + gpa);
        System.out.println("Email: " + email);
        System.out.println();
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        // Using default constructor
        Student s1 = new Student();
        s1.displayInfo();

        // Using parameterized constructor (2 parameters)
        Student s2 = new Student("Alice", 101);
        s2.displayInfo();

        // Using parameterized constructor (3 parameters)
        Student s3 = new Student("Bob", 102, 3.8);
        s3.displayInfo();

        // Using parameterized constructor (all parameters)
        Student s4 = new Student("Charlie", 103, 3.5, "charlie@email.com");
        s4.displayInfo();

        // Using copy constructor
        Student s5 = new Student(s4);
        s5.displayInfo();
    }
}
```

**Constructor Chaining**:
```java
public class Employee {
    private String name;
    private int id;
    private double salary;
    private String department;

    // Constructor 1 - calls Constructor 2
    public Employee(String name) {
        this(name, 0);  // Calls constructor with 2 parameters
        System.out.println("Constructor 1");
    }

    // Constructor 2 - calls Constructor 3
    public Employee(String name, int id) {
        this(name, id, 0.0);  // Calls constructor with 3 parameters
        System.out.println("Constructor 2");
    }

    // Constructor 3 - calls Constructor 4
    public Employee(String name, int id, double salary) {
        this(name, id, salary, "General");  // Calls constructor with 4 parameters
        System.out.println("Constructor 3");
    }

    // Constructor 4 - main constructor
    public Employee(String name, int id, double salary, String department) {
        this.name = name;
        this.id = id;
        this.salary = salary;
        this.department = department;
        System.out.println("Constructor 4 (Main)");
    }

    public void displayInfo() {
        System.out.println("Name: " + name + ", ID: " + id +
                         ", Salary: $" + salary + ", Dept: " + department);
    }
}

// Usage
Employee emp = new Employee("John");
// Output:
// Constructor 4 (Main)
// Constructor 3
// Constructor 2
// Constructor 1
```

**Constructor with Inheritance**:
```java
public class Person {
    protected String name;
    protected int age;

    // Parent default constructor
    public Person() {
        System.out.println("Person default constructor");
        this.name = "Unknown";
        this.age = 0;
    }

    // Parent parameterized constructor
    public Person(String name, int age) {
        System.out.println("Person parameterized constructor");
        this.name = name;
        this.age = age;
    }
}

public class Student extends Person {
    private int rollNumber;
    private double gpa;

    // Child default constructor
    public Student() {
        super();  // Calls Person()
        System.out.println("Student default constructor");
        this.rollNumber = 0;
        this.gpa = 0.0;
    }

    // Child parameterized constructor
    public Student(String name, int age, int rollNumber, double gpa) {
        super(name, age);  // Calls Person(String, int)
        System.out.println("Student parameterized constructor");
        this.rollNumber = rollNumber;
        this.gpa = gpa;
    }

    public void displayInfo() {
        System.out.println("Name: " + name);
        System.out.println("Age: " + age);
        System.out.println("Roll Number: " + rollNumber);
        System.out.println("GPA: " + gpa);
    }
}

// Usage
Student s1 = new Student();
// Output:
// Person default constructor
// Student default constructor

Student s2 = new Student("Alice", 20, 101, 3.8);
// Output:
// Person parameterized constructor
// Student parameterized constructor
```

---

# Coupling and Cohesion

## Q11: What is the difference between Coupling and Cohesion?

**Answer:**

**Coupling**: The degree of interdependence between software modules/classes.
- **Low coupling** (Loose coupling) is desirable
- Changes in one module should minimally affect others

**Cohesion**: The degree to which elements within a module belong together.
- **High cohesion** is desirable
- All elements in a module should be related and work toward a single purpose

| Aspect | Coupling | Cohesion |
|--------|----------|----------|
| Definition | Dependency between modules | Relationship within a module |
| Desirable level | Low (Loose) | High |
| Focus | Inter-module | Intra-module |
| Goal | Independence | Single responsibility |
| Good practice | Minimize dependencies | Maximize relatedness |

**Types of Coupling** (Best to Worst):
1. No Coupling
2. Data Coupling (best)
3. Stamp Coupling
4. Control Coupling
5. Common Coupling
6. Content Coupling (worst)

**Types of Cohesion** (Best to Worst):
1. Functional Cohesion (best)
2. Sequential Cohesion
3. Communicational Cohesion
4. Procedural Cohesion
5. Temporal Cohesion
6. Logical Cohesion
7. Coincidental Cohesion (worst)

**Example - High Coupling (Bad)**:
```java
// Tightly coupled - BAD DESIGN
public class OrderProcessor {
    // Direct dependency on concrete class
    private MySQLDatabase database;
    private EmailService emailService;
    private PayPalPayment paymentService;

    public OrderProcessor() {
        // Creating dependencies internally - tight coupling
        this.database = new MySQLDatabase();
        this.emailService = new EmailService();
        this.paymentService = new PayPalPayment();
    }

    public void processOrder(Order order) {
        // Directly using specific implementations
        database.connect();
        database.save(order);

        paymentService.processPayment(order.getAmount());

        emailService.sendEmail(order.getCustomerEmail(), "Order Confirmed");
    }
}

// Problem: If you want to change from MySQL to PostgreSQL,
// or from PayPal to Stripe, you need to modify OrderProcessor class
```

**Example - Low Coupling (Good)**:
```java
// Loosely coupled - GOOD DESIGN
// Using interfaces and dependency injection

public interface Database {
    void connect();
    void save(Order order);
}

public interface PaymentService {
    boolean processPayment(double amount);
}

public interface NotificationService {
    void sendNotification(String recipient, String message);
}

public class OrderProcessor {
    // Depend on abstractions, not concrete classes
    private Database database;
    private PaymentService paymentService;
    private NotificationService notificationService;

    // Dependency Injection through constructor
    public OrderProcessor(Database database,
                         PaymentService paymentService,
                         NotificationService notificationService) {
        this.database = database;
        this.paymentService = paymentService;
        this.notificationService = notificationService;
    }

    public void processOrder(Order order) {
        database.connect();
        database.save(order);

        boolean paymentSuccess = paymentService.processPayment(order.getAmount());

        if (paymentSuccess) {
            notificationService.sendNotification(
                order.getCustomerEmail(),
                "Order Confirmed"
            );
        }
    }
}

// Implementations
public class MySQLDatabase implements Database {
    public void connect() { /* MySQL connection */ }
    public void save(Order order) { /* MySQL save */ }
}

public class PostgreSQLDatabase implements Database {
    public void connect() { /* PostgreSQL connection */ }
    public void save(Order order) { /* PostgreSQL save */ }
}

public class StripePayment implements PaymentService {
    public boolean processPayment(double amount) { /* Stripe logic */ return true; }
}

public class PayPalPayment implements PaymentService {
    public boolean processPayment(double amount) { /* PayPal logic */ return true; }
}

public class EmailService implements NotificationService {
    public void sendNotification(String recipient, String message) { /* Email */ }
}

public class SMSService implements NotificationService {
    public void sendNotification(String recipient, String message) { /* SMS */ }
}

// Usage - Easy to switch implementations
Database db = new MySQLDatabase();  // Can easily change to PostgreSQLDatabase
PaymentService payment = new StripePayment();  // Can easily change to PayPalPayment
NotificationService notification = new EmailService();  // Can easily change to SMSService

OrderProcessor processor = new OrderProcessor(db, payment, notification);
```

**Example - Low Cohesion (Bad)**:
```java
// Low cohesion - class doing unrelated things - BAD
public class Utility {
    // Unrelated methods in one class
    public void saveToDatabase(String data) { }
    public void sendEmail(String email) { }
    public int calculateTax(double amount) { return 0; }
    public void printReport() { }
    public boolean validateCreditCard(String cardNumber) { return true; }
    public String formatDate(Date date) { return ""; }
}

// Problem: This class has multiple unrelated responsibilities
```

**Example - High Cohesion (Good)**:
```java
// High cohesion - each class has single, well-defined responsibility - GOOD

public class DatabaseService {
    public void saveToDatabase(String data) { }
    public void updateDatabase(String id, String data) { }
    public String retrieveFromDatabase(String id) { return ""; }
    public void deleteFromDatabase(String id) { }
}

public class EmailService {
    public void sendEmail(String email, String message) { }
    public void sendBulkEmail(List<String> emails, String message) { }
    public boolean validateEmailAddress(String email) { return true; }
}

public class TaxCalculator {
    public double calculateIncomeTax(double income) { return 0; }
    public double calculateSalesTax(double amount) { return 0; }
    public double calculatePropertyTax(double value) { return 0; }
}

public class ReportGenerator {
    public void generateSalesReport() { }
    public void generateInventoryReport() { }
    public void printReport(String reportType) { }
}

public class PaymentValidator {
    public boolean validateCreditCard(String cardNumber) { return true; }
    public boolean validateDebitCard(String cardNumber) { return true; }
    public boolean validateBankAccount(String accountNumber) { return true; }
}

public class DateFormatter {
    public String formatDate(Date date) { return ""; }
    public String formatDateTime(Date date) { return ""; }
    public Date parseDate(String dateString) { return null; }
}
```

**Real-world Example - E-commerce System**:
```java
// GOOD DESIGN: Low Coupling + High Cohesion

// Each class has high cohesion (single responsibility)
public class Product {
    private String id;
    private String name;
    private double price;
    private int quantity;

    // Only product-related methods
    public boolean isAvailable() {
        return quantity > 0;
    }

    public void reduceQuantity(int amount) {
        quantity -= amount;
    }

    // Getters and setters
}

public class ShoppingCart {
    private List<Product> items;

    // Only cart-related methods
    public void addItem(Product product) {
        items.add(product);
    }

    public void removeItem(Product product) {
        items.remove(product);
    }

    public double calculateTotal() {
        return items.stream()
                   .mapToDouble(Product::getPrice)
                   .sum();
    }
}

public class Order {
    private String orderId;
    private ShoppingCart cart;
    private double totalAmount;

    // Only order-related methods
    public void createOrder(ShoppingCart cart) {
        this.cart = cart;
        this.totalAmount = cart.calculateTotal();
    }
}

// Loose coupling through interfaces
public interface PaymentGateway {
    boolean processPayment(double amount);
}

public interface InventoryManager {
    boolean checkAvailability(String productId, int quantity);
    void updateInventory(String productId, int quantity);
}

public class OrderService {
    private PaymentGateway paymentGateway;
    private InventoryManager inventoryManager;

    // Dependency injection - loose coupling
    public OrderService(PaymentGateway paymentGateway,
                       InventoryManager inventoryManager) {
        this.paymentGateway = paymentGateway;
        this.inventoryManager = inventoryManager;
    }

    public boolean placeOrder(Order order) {
        // Process order using injected dependencies
        return paymentGateway.processPayment(order.getTotalAmount());
    }
}
```

---

# Composition vs Aggregation

## Q12: What is the difference between Composition and Aggregation?

**Answer:**

Both are types of "has-a" relationships, but they differ in ownership and lifecycle.

| Aspect | Composition | Aggregation |
|--------|------------|-------------|
| Relationship | Strong "has-a" (part-of) | Weak "has-a" (has-a) |
| Ownership | Strong ownership | Weak ownership |
| Lifecycle | Parts cannot exist without whole | Parts can exist independently |
| Dependency | Child cannot exist independently | Child can exist independently |
| Example | Car has Engine | Department has Teacher |
| UML Diamond | Filled diamond | Empty diamond |

**Composition (Strong Relationship)**:
- Parts are integral to the whole
- If the whole is destroyed, parts are also destroyed
- Parts cannot be shared between wholes

**Example**:
```java
// COMPOSITION - Engine cannot exist without Car
public class Engine {
    private String type;
    private int horsepower;

    public Engine(String type, int horsepower) {
        this.type = type;
        this.horsepower = horsepower;
    }

    public void start() {
        System.out.println(type + " engine starting...");
    }

    public void stop() {
        System.out.println("Engine stopping...");
    }
}

public class Wheel {
    private String type;
    private int size;

    public Wheel(String type, int size) {
        this.type = type;
        this.size = size;
    }
}

public class Car {
    // Composition - Car owns Engine and Wheels
    private Engine engine;  // Engine is part of Car
    private Wheel[] wheels;  // Wheels are parts of Car
    private String model;

    public Car(String model) {
        this.model = model;
        // Engine and Wheels created inside Car
        this.engine = new Engine("V6", 300);  // Engine created with Car
        this.wheels = new Wheel[4];
        for (int i = 0; i < 4; i++) {
            wheels[i] = new Wheel("Alloy", 17);  // Wheels created with Car
        }
    }

    public void start() {
        engine.start();
        System.out.println(model + " is ready to drive");
    }

    // When Car is destroyed, Engine and Wheels are also destroyed
}

// Usage
public class Main {
    public static void main(String[] args) {
        Car car = new Car("Toyota Camry");
        car.start();

        // When car object is garbage collected,
        // engine and wheels are also destroyed
        car = null;  // Car, Engine, and Wheels eligible for garbage collection
    }
}
```

**More Composition Examples**:
```java
// Example 1: House and Rooms (Composition)
public class Room {
    private String type;
    private double area;

    public Room(String type, double area) {
        this.type = type;
        this.area = area;
    }
}

public class House {
    private List<Room> rooms;  // Rooms are part of House

    public House() {
        rooms = new ArrayList<>();
        // Rooms created with House
        rooms.add(new Room("Bedroom", 200));
        rooms.add(new Room("Kitchen", 150));
        rooms.add(new Room("Living Room", 300));
    }

    // If House is destroyed, Rooms are also destroyed
}

// Example 2: Book and Pages (Composition)
public class Page {
    private int pageNumber;
    private String content;

    public Page(int pageNumber, String content) {
        this.pageNumber = pageNumber;
        this.content = content;
    }
}

public class Book {
    private String title;
    private List<Page> pages;  // Pages are part of Book

    public Book(String title, int numberOfPages) {
        this.title = title;
        this.pages = new ArrayList<>();
        for (int i = 1; i <= numberOfPages; i++) {
            pages.add(new Page(i, "Content of page " + i));
        }
    }

    // Pages cannot exist without Book
}

// Example 3: Human and Heart (Composition)
public class Heart {
    private int heartRate;

    public void beat() {
        System.out.println("Heart beating at " + heartRate + " bpm");
    }
}

public class Human {
    private Heart heart;  // Heart is part of Human

    public Human() {
        this.heart = new Heart();  // Heart created with Human
    }

    // Heart cannot exist without Human
}
```

**Aggregation (Weak Relationship)**:
- Parts can exist independently
- If the whole is destroyed, parts can still exist
- Parts can be shared between wholes

**Example**:
```java
// AGGREGATION - Teacher can exist without Department
public class Teacher {
    private String name;
    private String subject;
    private int experience;

    public Teacher(String name, String subject, int experience) {
        this.name = name;
        this.subject = subject;
        this.experience = experience;
    }

    public void teach() {
        System.out.println(name + " is teaching " + subject);
    }

    public String getName() {
        return name;
    }
}

public class Department {
    private String departmentName;
    private List<Teacher> teachers;  // Department has Teachers

    public Department(String departmentName) {
        this.departmentName = departmentName;
        this.teachers = new ArrayList<>();
    }

    // Teachers are passed from outside - not created inside
    public void addTeacher(Teacher teacher) {
        teachers.add(teacher);
        System.out.println(teacher.getName() + " added to " + departmentName);
    }

    public void removeTeacher(Teacher teacher) {
        teachers.remove(teacher);
        System.out.println(teacher.getName() + " removed from " + departmentName);
    }

    public void displayTeachers() {
        System.out.println("Teachers in " + departmentName + ":");
        for (Teacher teacher : teachers) {
            System.out.println("- " + teacher.getName());
        }
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        // Teachers exist independently
        Teacher teacher1 = new Teacher("Dr. Smith", "Mathematics", 10);
        Teacher teacher2 = new Teacher("Prof. Johnson", "Physics", 15);
        Teacher teacher3 = new Teacher("Dr. Williams", "Chemistry", 8);

        // Department created separately
        Department mathDept = new Department("Mathematics Department");
        Department scienceDept = new Department("Science Department");

        // Teachers added to departments
        mathDept.addTeacher(teacher1);
        scienceDept.addTeacher(teacher2);
        scienceDept.addTeacher(teacher3);

        // Same teacher can be in multiple departments
        mathDept.addTeacher(teacher2);  // Teacher shared between departments

        mathDept.displayTeachers();

        // If department is destroyed, teachers still exist
        mathDept = null;

        // Teachers can still function
        teacher1.teach();
        teacher2.teach();
    }
}
```

**More Aggregation Examples**:
```java
// Example 1: Library and Books (Aggregation)
public class Book {
    private String title;
    private String author;

    public Book(String title, String author) {
        this.title = title;
        this.author = author;
    }

    public String getTitle() {
        return title;
    }
}

public class Library {
    private String name;
    private List<Book> books;

    public Library(String name) {
        this.name = name;
        this.books = new ArrayList<>();
    }

    public void addBook(Book book) {
        books.add(book);
    }

    public void removeBook(Book book) {
        books.remove(book);
    }

    // Books exist independently of Library
}

// Usage
Book book1 = new Book("Java Programming", "John Doe");
Book book2 = new Book("Python Basics", "Jane Smith");

Library library1 = new Library("City Library");
Library library2 = new Library("University Library");

library1.addBook(book1);
library2.addBook(book1);  // Same book in multiple libraries

// If library is destroyed, books still exist
library1 = null;
// book1 and book2 still exist

// Example 2: Company and Employees (Aggregation)
public class Employee {
    private String name;
    private String id;

    public Employee(String name, String id) {
        this.name = name;
        this.id = id;
    }
}

public class Company {
    private String companyName;
    private List<Employee> employees;

    public Company(String companyName) {
        this.companyName = companyName;
        this.employees = new ArrayList<>();
    }

    public void hireEmployee(Employee employee) {
        employees.add(employee);
    }

    public void fireEmployee(Employee employee) {
        employees.remove(employee);
    }

    // Employees exist independently of Company
}

// Example 3: Team and Players (Aggregation)
public class Player {
    private String name;
    private int jerseyNumber;

    public Player(String name, int jerseyNumber) {
        this.name = name;
        this.jerseyNumber = jerseyNumber;
    }
}

public class Team {
    private String teamName;
    private List<Player> players;

    public Team(String teamName) {
        this.teamName = teamName;
        this.players = new ArrayList<>();
    }

    public void addPlayer(Player player) {
        players.add(player);
    }

    public void removePlayer(Player player) {
        players.remove(player);
    }

    // Players exist independently - can change teams
}
```

**Comparison Example**:
```java
// Composition vs Aggregation in one system

// COMPOSITION: University and Departments
public class Department {
    private String name;
    // Department is integral part of University
}

public class University {
    private String universityName;
    private List<Department> departments;  // COMPOSITION

    public University(String universityName) {
        this.universityName = universityName;
        this.departments = new ArrayList<>();
        // Departments created with University
        departments.add(new Department("Computer Science"));
        departments.add(new Department("Mathematics"));
    }
    // If University closes, Departments cease to exist
}

// AGGREGATION: Department and Professors
public class Professor {
    private String name;
    // Professor exists independently
}

public class DepartmentWithProfessors {
    private String name;
    private List<Professor> professors;  // AGGREGATION

    public DepartmentWithProfessors(String name) {
        this.name = name;
        this.professors = new ArrayList<>();
    }

    public void addProfessor(Professor professor) {
        professors.add(professor);
    }
    // If Department is dissolved, Professors still exist
}
```

---

# Advanced Questions

## Q13: What is the "is-a" vs "has-a" relationship?

**Answer:**

**"is-a" Relationship** (Inheritance):
- Represents inheritance
- Subclass is a type of superclass
- Uses `extends` keyword

**"has-a" Relationship** (Association):
- Represents composition/aggregation
- Object contains another object
- Uses member variables

**Examples**:
```java
// IS-A Relationship (Inheritance)

// Dog IS-A Animal
public class Animal {
    public void eat() {
        System.out.println("Animal is eating");
    }
}

public class Dog extends Animal {
    public void bark() {
        System.out.println("Dog is barking");
    }
}
// Dog is a type of Animal

// Manager IS-A Employee
public class Employee {
    protected String name;
    protected double salary;
}

public class Manager extends Employee {
    private int teamSize;
}
// Manager is a type of Employee

// HAS-A Relationship (Composition/Aggregation)

// Car HAS-A Engine
public class Engine {
    public void start() {
        System.out.println("Engine started");
    }
}

public class Car {
    private Engine engine;  // HAS-A

    public Car() {
        this.engine = new Engine();
    }
}
// Car has an Engine

// Person HAS-A Address
public class Address {
    private String street;
    private String city;
}

public class Person {
    private String name;
    private Address address;  // HAS-A

    public Person(String name, Address address) {
        this.name = name;
        this.address = address;
    }
}
// Person has an Address
```

## Q14: Can a class be both abstract and final?

**Answer:** NO

**Reason**:
- `final` class cannot be extended
- `abstract` class must be extended to be used
- These are contradictory requirements

```java
// This will cause compilation error
// public abstract final class InvalidClass { }  // ERROR

// Final class - cannot be extended
public final class FinalClass {
    public void method() { }
}

// Abstract class - must be extended
public abstract class AbstractClass {
    public abstract void method();
}
```

## Q15: Can we have private methods in an interface?

**Answer:** YES (from Java 9 onwards)

```java
public interface MyInterface {
    // Public abstract method
    void publicMethod();

    // Default method (Java 8+)
    default void defaultMethod() {
        System.out.println("Default method");
        helperMethod();  // Can call private method
    }

    // Static method (Java 8+)
    static void staticMethod() {
        System.out.println("Static method");
        privateStaticHelper();  // Can call private static method
    }

    // Private method (Java 9+)
    private void helperMethod() {
        System.out.println("Private helper method");
    }

    // Private static method (Java 9+)
    private static void privateStaticHelper() {
        System.out.println("Private static helper");
    }
}
```

## Q16: What is method hiding in Java?

**Answer:** Method hiding occurs when a subclass declares a static method with the same signature as a static method in the superclass.

```java
public class Parent {
    public static void staticMethod() {
        System.out.println("Parent static method");
    }

    public void instanceMethod() {
        System.out.println("Parent instance method");
    }
}

public class Child extends Parent {
    // Method Hiding (static method)
    public static void staticMethod() {
        System.out.println("Child static method");
    }

    // Method Overriding (instance method)
    @Override
    public void instanceMethod() {
        System.out.println("Child instance method");
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        Parent p1 = new Parent();
        Parent p2 = new Child();  // Parent reference, Child object
        Child c = new Child();

        // Static method - Method Hiding (resolved at compile-time)
        p1.staticMethod();  // Parent static method
        p2.staticMethod();  // Parent static method (not Child's!)
        c.staticMethod();   // Child static method

        // Instance method - Method Overriding (resolved at runtime)
        p1.instanceMethod();  // Parent instance method
        p2.instanceMethod();  // Child instance method (polymorphism)
        c.instanceMethod();   // Child instance method
    }
}
```

## Q17: What are the SOLID principles?

**Answer:**

**S** - Single Responsibility Principle
**O** - Open/Closed Principle
**L** - Liskov Substitution Principle
**I** - Interface Segregation Principle
**D** - Dependency Inversion Principle

**Brief examples**:
```java
// S - Single Responsibility Principle
// Each class should have one responsibility

// BAD - Multiple responsibilities
class User {
    public void saveToDatabase() { }
    public void sendEmail() { }
    public void generateReport() { }
}

// GOOD - Separate responsibilities
class User {
    private String name;
    private String email;
}

class UserRepository {
    public void save(User user) { }
}

class EmailService {
    public void sendEmail(User user) { }
}

class ReportGenerator {
    public void generate(User user) { }
}

// O - Open/Closed Principle
// Open for extension, closed for modification

interface Shape {
    double calculateArea();
}

class Circle implements Shape {
    private double radius;
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

class Rectangle implements Shape {
    private double width, height;
    public double calculateArea() {
        return width * height;
    }
}

// L - Liskov Substitution Principle
// Subclasses should be substitutable for their base classes

class Bird {
    public void eat() { }
}

class Sparrow extends Bird {
    public void fly() { }
}

// I - Interface Segregation Principle
// Clients should not be forced to depend on interfaces they don't use

// BAD - Fat interface
interface Worker {
    void work();
    void eat();
    void sleep();
}

// GOOD - Segregated interfaces
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

interface Sleepable {
    void sleep();
}

// D - Dependency Inversion Principle
// Depend on abstractions, not concretions

interface Database {
    void save(String data);
}

class UserService {
    private Database database;  // Depends on abstraction

    public UserService(Database database) {
        this.database = database;
    }
}
```

---

## Summary

This document covers:
1. **Four Pillars of OOP**: Encapsulation, Inheritance, Polymorphism, Abstraction
2. **Classes and Objects**: Blueprint vs instances
3. **Method Overloading vs Overriding**: Compile-time vs runtime polymorphism
4. **Abstract Classes vs Interfaces**: When to use which
5. **Access Modifiers**: public, protected, default, private
6. **Static vs Instance**: Class-level vs object-level
7. **Constructors**: Types and usage
8. **Coupling and Cohesion**: Low coupling and high cohesion
9. **Composition vs Aggregation**: Strong vs weak "has-a"
10. **Advanced Concepts**: SOLID principles, method hiding, etc.

**Key Takeaways for Interviews**:
- Understand the WHY behind each concept, not just the WHAT
- Practice with real-world examples
- Be able to explain trade-offs and design decisions
- Know when to use each concept
- Understand relationships between concepts

Good luck with your interviews!
