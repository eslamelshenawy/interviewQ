# Object-Oriented Programming (OOP)

## Table of Contents
1. [Introduction](#introduction)
2. [What is OOP?](#what-is-oop)
3. [Why OOP?](#why-oop)
4. [The Four Pillars of OOP](#the-four-pillars-of-oop)
5. [Key Concepts](#key-concepts)
6. [Benefits of OOP](#benefits-of-oop)
7. [Popular OOP Languages](#popular-oop-languages)
8. [Getting Started](#getting-started)

---

## Introduction

Object-Oriented Programming (OOP) is a programming paradigm that organizes software design around data (objects) rather than functions and logic. It's one of the most popular programming paradigms used in modern software development.

## What is OOP?

Object-Oriented Programming is a methodology or paradigm to design a program using classes and objects. It simplifies software development and maintenance by providing several key concepts:

- **Object**: An instance of a class that contains data (attributes) and methods (behaviors)
- **Class**: A blueprint or template for creating objects
- **Method**: A function defined within a class that describes the behaviors of an object
- **Attribute**: A variable defined within a class that represents the state of an object

### Simple Example

```java
// Class definition
public class Car {
    // Attributes (data)
    private String brand;
    private String color;
    private int speed;

    // Constructor
    public Car(String brand, String color) {
        this.brand = brand;
        this.color = color;
        this.speed = 0;
    }

    // Methods (behavior)
    public void accelerate(int increment) {
        speed += increment;
        System.out.println(brand + " is now moving at " + speed + " km/h");
    }

    public void brake() {
        speed = 0;
        System.out.println(brand + " has stopped");
    }
}

// Creating objects
public class Main {
    public static void main(String[] args) {
        Car myCar = new Car("Toyota", "Red");
        myCar.accelerate(50);
        myCar.brake();
    }
}
```

## Why OOP?

OOP offers several advantages over procedural programming:

1. **Modularity**: Code is organized into separate classes, making it easier to manage
2. **Reusability**: Classes can be reused across different programs
3. **Maintainability**: Changes to one part of the code don't affect other parts
4. **Scalability**: Easy to extend and scale applications
5. **Security**: Data hiding through encapsulation protects data from external access
6. **Flexibility**: Polymorphism allows objects to be treated in multiple ways
7. **Real-world Modeling**: Maps closely to real-world entities and relationships

## The Four Pillars of OOP

### 1. Encapsulation

**Definition**: Bundling data (attributes) and methods that operate on that data within a single unit (class), and restricting direct access to some components.

**Key Points**:
- Data hiding using access modifiers (private, protected, public)
- Provides controlled access through getter and setter methods
- Protects object integrity

**Example**:
```java
public class BankAccount {
    private double balance;  // Private - cannot be accessed directly

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    public double getBalance() {
        return balance;
    }
}
```

### 2. Inheritance

**Definition**: A mechanism where a new class (child/subclass) inherits properties and behaviors from an existing class (parent/superclass).

**Key Points**:
- Promotes code reusability
- Establishes a "is-a" relationship
- Supports hierarchical classification

**Example**:
```java
public class Animal {
    public void eat() {
        System.out.println("This animal eats food");
    }
}

public class Dog extends Animal {
    public void bark() {
        System.out.println("The dog barks");
    }
}
```

### 3. Polymorphism

**Definition**: The ability of objects to take many forms. The same method can behave differently based on the object that invokes it.

**Types**:
- **Compile-time Polymorphism**: Method Overloading
- **Runtime Polymorphism**: Method Overriding

**Example**:
```java
public class Shape {
    public void draw() {
        System.out.println("Drawing a shape");
    }
}

public class Circle extends Shape {
    @Override
    public void draw() {
        System.out.println("Drawing a circle");
    }
}

public class Square extends Shape {
    @Override
    public void draw() {
        System.out.println("Drawing a square");
    }
}

// Polymorphic behavior
Shape shape1 = new Circle();
Shape shape2 = new Square();
shape1.draw();  // Outputs: Drawing a circle
shape2.draw();  // Outputs: Drawing a square
```

### 4. Abstraction

**Definition**: Hiding complex implementation details and showing only essential features of an object.

**Key Points**:
- Reduces complexity
- Achieved through abstract classes and interfaces
- Focuses on "what" an object does rather than "how"

**Example**:
```java
public abstract class Vehicle {
    abstract void start();  // Abstract method - no implementation

    public void stop() {    // Concrete method
        System.out.println("Vehicle stopped");
    }
}

public class Car extends Vehicle {
    @Override
    void start() {
        System.out.println("Car starts with a key");
    }
}

public class Bike extends Vehicle {
    @Override
    void start() {
        System.out.println("Bike starts with a kick");
    }
}
```

## Key Concepts

### Classes and Objects

- **Class**: A blueprint for creating objects
- **Object**: An instance of a class with actual values

### Constructors

Special methods called when an object is created to initialize the object's state.

```java
public class Person {
    private String name;
    private int age;

    // Constructor
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

### Access Modifiers

Control the visibility of class members:
- **public**: Accessible from anywhere
- **private**: Accessible only within the same class
- **protected**: Accessible within the same package and subclasses
- **default**: Accessible within the same package

### Static vs Instance Members

- **Instance members**: Belong to individual objects
- **Static members**: Belong to the class itself, shared across all instances

### Interfaces

Contracts that define methods a class must implement:

```java
public interface Drawable {
    void draw();
}

public class Circle implements Drawable {
    @Override
    public void draw() {
        System.out.println("Drawing circle");
    }
}
```

## Benefits of OOP

1. **Better Code Organization**: Logical grouping of related data and functions
2. **Code Reusability**: Through inheritance and composition
3. **Easy Maintenance**: Modular structure makes updates easier
4. **Data Security**: Encapsulation protects data from unauthorized access
5. **Flexibility**: Polymorphism allows for flexible code
6. **Problem Solving**: Maps real-world problems to code structures
7. **Team Collaboration**: Multiple developers can work on different classes simultaneously

## Popular OOP Languages

- **Java**: Purely object-oriented (except primitives)
- **C++**: Supports both OOP and procedural programming
- **Python**: Multi-paradigm with strong OOP support
- **C#**: Strongly typed OOP language
- **JavaScript**: Prototype-based OOP
- **Ruby**: Purely object-oriented
- **PHP**: Supports OOP
- **Swift**: Modern OOP language for iOS development

## Getting Started

To master OOP, follow these steps:

1. **Understand the Basics**: Learn the four pillars thoroughly
2. **Practice with Simple Examples**: Start with basic classes and objects
3. **Study Design Patterns**: Learn common OOP design patterns
4. **Build Projects**: Apply OOP concepts in real-world projects
5. **Review Code**: Read and analyze OOP code written by others
6. **Refactor**: Practice converting procedural code to OOP

### Recommended Learning Path

1. Classes and Objects
2. Encapsulation and Access Modifiers
3. Constructors and Methods
4. Inheritance
5. Polymorphism (Overloading and Overriding)
6. Abstraction (Abstract Classes and Interfaces)
7. Advanced Concepts (Composition, Aggregation, Coupling, Cohesion)
8. Design Patterns
9. SOLID Principles

---

## Additional Resources

For comprehensive interview questions and detailed examples, refer to:
- [OOP-Interview-Questions.md](./OOP-Interview-Questions.md)

## Practice Tips

1. **Code Daily**: Practice writing classes and objects every day
2. **Solve Problems**: Use platforms like LeetCode, HackerRank with OOP approach
3. **Design Systems**: Try to design systems using OOP principles
4. **Review SOLID Principles**: Understand and apply SOLID principles
5. **Study Real Code**: Analyze open-source projects to see OOP in action

---

**Remember**: OOP is not just about syntax; it's about thinking in terms of objects, their responsibilities, and their interactions. The goal is to write code that is maintainable, scalable, and mirrors real-world relationships.

**Happy Learning!**
