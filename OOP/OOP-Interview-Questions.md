# أسئلة مقابلات OOP الشاملة

## المحتويات
1. [المفاهيم الأساسية](#المفاهيم-الأساسية)
2. [Encapsulation](#encapsulation-التغليف)
3. [Inheritance](#inheritance-الوراثة)
4. [Polymorphism](#polymorphism-تعدد-الأشكال)
5. [Abstraction](#abstraction-التجريد)
6. [أسئلة متقدمة](#أسئلة-متقدمة)
7. [أسئلة عملية](#أسئلة-عملية)

---

## المفاهيم الأساسية

### 1. ما هو OOP؟ ولماذا نستخدمه؟

**الإجابة:**

**OOP (Object-Oriented Programming)** هو نموذج برمجي يعتمد على مفهوم الكائنات (Objects) التي تحتوي على بيانات (attributes) وسلوك (methods).

**لماذا نستخدمه:**
- **إعادة استخدام الكود** (Code Reusability)
- **سهولة الصيانة** (Maintainability)
- **التنظيم** (Modularity)
- **الأمان** (Security) عبر التغليف
- **المرونة** (Flexibility) عبر الوراثة وتعدد الأشكال

**مثال بسيط:**
```java
// بدون OOP - Procedural
String[] carNames = {"Toyota", "BMW"};
int[] carSpeeds = {120, 180};

void accelerateCar(int index) {
    carSpeeds[index] += 10;
}

// مع OOP
class Car {
    private String name;
    private int speed;

    public Car(String name) {
        this.name = name;
        this.speed = 0;
    }

    public void accelerate() {
        this.speed += 10;
    }

    public int getSpeed() {
        return speed;
    }
}

// الاستخدام
Car toyota = new Car("Toyota");
Car bmw = new Car("BMW");
toyota.accelerate();
```

---

### 2. ما الفرق بين Class و Object؟

**الإجابة:**

| Class | Object |
|-------|--------|
| Blueprint/Template | Instance من الـ class |
| تعريف منطقي | وجود في الذاكرة |
| لا تستهلك ذاكرة | يستهلك ذاكرة |
| تُكتب مرة واحدة | يمكن إنشاء عدة objects |

**مثال:**
```java
// Class - المخطط
class Student {
    String name;
    int age;

    void study() {
        System.out.println(name + " is studying");
    }
}

// Objects - نماذج من الـ class
public class Main {
    public static void main(String[] args) {
        Student student1 = new Student(); // Object 1
        student1.name = "Ahmed";
        student1.age = 20;

        Student student2 = new Student(); // Object 2
        student2.name = "Fatima";
        student2.age = 22;

        student1.study(); // Ahmed is studying
        student2.study(); // Fatima is studying
    }
}
```

**تشبيه:**
- **Class** = مخطط البيت (Blueprint)
- **Object** = البيت المبني فعلياً

---

### 3. ما هي المبادئ الأربعة للـ OOP؟

**الإجابة:**

#### 1️⃣ **Encapsulation** (التغليف)
إخفاء البيانات الداخلية وتوفير واجهة للوصول إليها.

#### 2️⃣ **Inheritance** (الوراثة)
إنشاء كلاس جديد من كلاس موجود.

#### 3️⃣ **Polymorphism** (تعدد الأشكال)
استخدام نفس الواجهة لأنواع مختلفة.

#### 4️⃣ **Abstraction** (التجريد)
إخفاء التفاصيل المعقدة وإظهار الوظائف الأساسية.

**سنشرح كل واحد بالتفصيل في الأقسام التالية.**

---

## Encapsulation (التغليف)

### 4. ما هو Encapsulation؟ ولماذا نستخدمه؟

**الإجابة:**

Encapsulation هو إخفاء البيانات الداخلية للكلاس ومنع الوصول المباشر لها من الخارج.

**كيف نطبقه:**
- جعل المتغيرات `private`
- توفير `getter` و `setter` methods

**لماذا نستخدمه:**
- **حماية البيانات** من التعديل غير الصحيح
- **التحكم في الوصول** للبيانات
- **سهولة الصيانة**
- **Flexibility** في تغيير التنفيذ الداخلي

**مثال:**

```java
// ❌ بدون Encapsulation
class BankAccount {
    public double balance; // أي أحد يمكنه التعديل!
}

BankAccount account = new BankAccount();
account.balance = -1000; // مشكلة! رصيد سالب

// ✅ مع Encapsulation
class BankAccount {
    private double balance; // مخفي

    // Constructor
    public BankAccount(double initialBalance) {
        if (initialBalance >= 0) {
            this.balance = initialBalance;
        } else {
            this.balance = 0;
        }
    }

    // Getter
    public double getBalance() {
        return balance;
    }

    // Setter مع validation
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        } else {
            System.out.println("Invalid amount");
        }
    }

    public void withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
        } else {
            System.out.println("Insufficient balance or invalid amount");
        }
    }
}

// الاستخدام
BankAccount account = new BankAccount(1000);
// account.balance = -500; // Error: balance is private
account.deposit(500);
account.withdraw(200);
System.out.println(account.getBalance()); // 1300
```

**الفوائد:**
- لا يمكن جعل الرصيد سالب
- كل التعديلات تمر عبر validation
- يمكن تغيير التنفيذ الداخلي بدون تأثير على الكود الخارجي

---

### 5. ما الفرق بين Getter و Setter؟

**الإجابة:**

| Getter | Setter |
|--------|--------|
| يقرأ قيمة المتغير | يعدل قيمة المتغير |
| لا يأخذ parameters | يأخذ parameter واحد |
| يرجع قيمة | void (عادةً) |
| اسمه `getXxx()` | اسمه `setXxx()` |

**مثال:**
```java
class Person {
    private String name;
    private int age;

    // Getter للاسم
    public String getName() {
        return name;
    }

    // Setter للاسم
    public void setName(String name) {
        if (name != null && !name.isEmpty()) {
            this.name = name;
        }
    }

    // Getter للعمر
    public int getAge() {
        return age;
    }

    // Setter للعمر مع validation
    public void setAge(int age) {
        if (age > 0 && age < 150) {
            this.age = age;
        } else {
            System.out.println("Invalid age");
        }
    }
}

// الاستخدام
Person person = new Person();
person.setName("Ahmed");
person.setAge(25);

System.out.println(person.getName()); // Ahmed
System.out.println(person.getAge());  // 25

person.setAge(-5);   // Invalid age
person.setAge(200);  // Invalid age
```

**متى نستخدم:**
- **Getter only**: للقراءة فقط (read-only)
- **Setter only**: للكتابة فقط (write-only) - نادر
- **Both**: للقراءة والكتابة

---

## Inheritance (الوراثة)

### 6. ما هو Inheritance؟ ولماذا نستخدمه؟

**الإجابة:**

Inheritance هو آلية تسمح لكلاس بوراثة الخصائص (attributes) والسلوكيات (methods) من كلاس آخر.

**المصطلحات:**
- **Parent Class** / **Super Class** / **Base Class**
- **Child Class** / **Sub Class** / **Derived Class**

**لماذا نستخدمه:**
- **إعادة استخدام الكود**
- **تجنب التكرار**
- **تنظيم هرمي منطقي**
- **Polymorphism**

**مثال:**

```java
// Parent Class
class Animal {
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
        System.out.println("Some sound");
    }
}

// Child Class 1
class Dog extends Animal {
    private String breed;

    public Dog(String name, int age, String breed) {
        super(name, age); // استدعاء constructor الـ parent
        this.breed = breed;
    }

    // Method خاص بـ Dog
    public void bark() {
        System.out.println(name + " is barking: Woof!");
    }

    // Override parent method
    @Override
    public void makeSound() {
        System.out.println(name + " says: Woof Woof!");
    }
}

// Child Class 2
class Cat extends Animal {
    private String color;

    public Cat(String name, int age, String color) {
        super(name, age);
        this.color = color;
    }

    public void meow() {
        System.out.println(name + " is meowing: Meow!");
    }

    @Override
    public void makeSound() {
        System.out.println(name + " says: Meow Meow!");
    }
}

// الاستخدام
public class Main {
    public static void main(String[] args) {
        Dog dog = new Dog("Max", 3, "German Shepherd");
        dog.eat();       // inherited من Animal
        dog.sleep();     // inherited من Animal
        dog.bark();      // method خاص بـ Dog
        dog.makeSound(); // overridden method

        Cat cat = new Cat("Whiskers", 2, "White");
        cat.eat();       // inherited من Animal
        cat.meow();      // method خاص بـ Cat
        cat.makeSound(); // overridden method
    }
}
```

**الفوائد:**
- كود `eat()` و `sleep()` مكتوب مرة واحدة فقط
- سهولة إضافة حيوانات جديدة
- تنظيم منطقي

---

### 7. ما هي أنواع Inheritance؟

**الإجابة:**

#### 1️⃣ **Single Inheritance**
كلاس يرث من كلاس واحد فقط.

```java
class A { }
class B extends A { }
```

#### 2️⃣ **Multilevel Inheritance**
سلسلة من الوراثة.

```java
class Animal { }
class Mammal extends Animal { }
class Dog extends Mammal { }
```

#### 3️⃣ **Hierarchical Inheritance**
عدة كلاسات ترث من نفس الكلاس.

```java
class Animal { }
class Dog extends Animal { }
class Cat extends Animal { }
class Bird extends Animal { }
```

#### 4️⃣ **Multiple Inheritance** ❌
كلاس يرث من عدة كلاسات (غير مدعوم في Java للـ classes).

```java
// ❌ لا يعمل في Java
class A { }
class B { }
class C extends A, B { } // Error!

// ✅ يعمل مع Interfaces
interface A { }
interface B { }
class C implements A, B { } // OK
```

**لماذا Java لا تدعم Multiple Inheritance للـ classes؟**
لتجنب **Diamond Problem**.

**مثال Diamond Problem:**
```java
// لو كان مدعوماً (مثال نظري)
class A {
    void method() { System.out.println("A"); }
}

class B extends A {
    void method() { System.out.println("B"); }
}

class C extends A {
    void method() { System.out.println("C"); }
}

class D extends B, C { // لو كان مدعوماً
    // أي method() يرث؟ من B أم C؟ ← غموض!
}
```

**الحل في Java:**
استخدام Interfaces مع default methods.

---

### 8. ما الفرق بين `this` و `super`؟

**الإجابة:**

| `this` | `super` |
|--------|---------|
| يشير للكائن الحالي | يشير للـ parent class |
| للوصول لـ members الحالية | للوصول لـ parent members |
| استدعاء constructor الحالي | استدعاء parent constructor |

**مثال:**
```java
class Parent {
    int x = 10;

    Parent() {
        System.out.println("Parent constructor");
    }

    void display() {
        System.out.println("Parent display");
    }
}

class Child extends Parent {
    int x = 20; // متغير بنفس الاسم

    Child() {
        super(); // استدعاء parent constructor
        System.out.println("Child constructor");
    }

    void display() {
        System.out.println("x in Child: " + this.x);      // 20
        System.out.println("x in Parent: " + super.x);    // 10

        super.display(); // استدعاء parent method
        this.show();     // استدعاء child method
    }

    void show() {
        System.out.println("Child show");
    }
}

// الاستخدام
Child child = new Child();
// Output:
// Parent constructor
// Child constructor

child.display();
// x in Child: 20
// x in Parent: 10
// Parent display
// Child show
```

**قواعد `super()`:**
- يجب أن يكون أول سطر في الـ constructor
- إذا لم تكتبه، Java تضيفه تلقائياً

```java
class Child extends Parent {
    Child() {
        // super(); ← Java تضيفها تلقائياً
        System.out.println("Child");
    }
}
```

---

## Polymorphism (تعدد الأشكال)

### 9. ما هو Polymorphism؟ وما أنواعه؟

**الإجابة:**

Polymorphism تعني "أشكال متعددة" - القدرة على استخدام نفس الواجهة لأنواع مختلفة.

**أنواعه:**

#### 1️⃣ **Compile-Time Polymorphism** (Static Polymorphism)
- **Method Overloading**
- يُحدد في وقت الـ compilation

#### 2️⃣ **Runtime Polymorphism** (Dynamic Polymorphism)
- **Method Overriding**
- يُحدد في وقت الـ runtime

---

### 10. ما هو Method Overloading؟

**الإجابة:**

Method Overloading هو وجود عدة methods بنفس الاسم لكن بـ parameters مختلفة.

**شروطه:**
- نفس اسم الـ method
- عدد parameters مختلف، أو
- نوع parameters مختلف، أو
- ترتيب parameters مختلف
- **لا يعتمد على return type**

**مثال:**

```java
class Calculator {
    // Method 1: جمع رقمين int
    public int add(int a, int b) {
        return a + b;
    }

    // Method 2: جمع 3 أرقام int
    public int add(int a, int b, int c) {
        return a + b + c;
    }

    // Method 3: جمع رقمين double
    public double add(double a, double b) {
        return a + b;
    }

    // Method 4: جمع int و double
    public double add(int a, double b) {
        return a + b;
    }

    // Method 5: جمع double و int (ترتيب مختلف)
    public double add(double a, int b) {
        return a + b;
    }
}

// الاستخدام
public class Main {
    public static void main(String[] args) {
        Calculator calc = new Calculator();

        System.out.println(calc.add(5, 3));           // 8 (Method 1)
        System.out.println(calc.add(5, 3, 2));        // 10 (Method 2)
        System.out.println(calc.add(5.5, 3.2));       // 8.7 (Method 3)
        System.out.println(calc.add(5, 3.2));         // 8.2 (Method 4)
        System.out.println(calc.add(5.5, 3));         // 8.5 (Method 5)
    }
}
```

**Constructor Overloading:**
```java
class Student {
    String name;
    int age;
    String course;

    // Constructor 1
    public Student() {
        this.name = "Unknown";
        this.age = 0;
        this.course = "Not enrolled";
    }

    // Constructor 2
    public Student(String name) {
        this.name = name;
        this.age = 0;
        this.course = "Not enrolled";
    }

    // Constructor 3
    public Student(String name, int age) {
        this.name = name;
        this.age = age;
        this.course = "Not enrolled";
    }

    // Constructor 4
    public Student(String name, int age, String course) {
        this.name = name;
        this.age = age;
        this.course = course;
    }
}

// الاستخدام
Student s1 = new Student();
Student s2 = new Student("Ahmed");
Student s3 = new Student("Fatima", 22);
Student s4 = new Student("Ali", 20, "Computer Science");
```

---

### 11. ما هو Method Overriding؟

**الإجابة:**

Method Overriding هو إعادة تعريف method من الـ parent class في الـ child class.

**شروطه:**
- نفس اسم الـ method
- نفس الـ parameters
- نفس return type (أو subtype)
- يجب أن يكون access modifier نفسه أو أوسع
- **لا يمكن override static methods**

**مثال:**

```java
class Animal {
    public void makeSound() {
        System.out.println("Animal makes a sound");
    }

    public void eat() {
        System.out.println("Animal is eating");
    }
}

class Dog extends Animal {
    // Override method
    @Override
    public void makeSound() {
        System.out.println("Dog barks: Woof!");
    }

    // Override method
    @Override
    public void eat() {
        System.out.println("Dog is eating dog food");
    }
}

class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Cat meows: Meow!");
    }
}

// الاستخدام
public class Main {
    public static void main(String[] args) {
        Animal animal1 = new Dog();
        Animal animal2 = new Cat();
        Animal animal3 = new Animal();

        animal1.makeSound(); // Dog barks: Woof!
        animal2.makeSound(); // Cat meows: Meow!
        animal3.makeSound(); // Animal makes a sound
    }
}
```

**الفوائد:**
- سلوك مخصص لكل كلاس
- استخدام polymorphism

---

### 12. ما الفرق بين Overloading و Overriding؟

**الإجابة:**

| Overloading | Overriding |
|-------------|------------|
| نفس الكلاس | كلاسات مختلفة (parent/child) |
| Parameters مختلفة | Parameters نفسها |
| Compile-time | Runtime |
| Static Polymorphism | Dynamic Polymorphism |
| يمكن مع static methods | لا يمكن مع static methods |
| return type لا يهم | return type يجب أن يكون نفسه |

**مثال يجمع الاثنين:**
```java
class Parent {
    // Overloading في نفس الكلاس
    public void display(int a) {
        System.out.println("Int: " + a);
    }

    public void display(String a) {
        System.out.println("String: " + a);
    }

    // Method للـ overriding
    public void show() {
        System.out.println("Parent show");
    }
}

class Child extends Parent {
    // Overriding من parent
    @Override
    public void show() {
        System.out.println("Child show");
    }

    // Overloading في child
    public void show(int a) {
        System.out.println("Child show with int: " + a);
    }
}
```

---

## Abstraction (التجريد)

### 13. ما هو Abstraction؟ وكيف نطبقه؟

**الإجابة:**

Abstraction هو إخفاء التفاصيل المعقدة وإظهار الوظائف الأساسية فقط.

**كيف نطبقه:**
1. **Abstract Classes**
2. **Interfaces**

**مثال من الحياة:**
- **السيارة**: تعرف كيف تقودها (التجريد)، لكن لا تعرف كيف يعمل المحرك (التفاصيل المخفية)
- **الهاتف**: تضغط على أزرار (واجهة بسيطة)، لكن لا تعرف الدوائر الكهربائية الداخلية

**الفرق بين Abstraction و Encapsulation:**
- **Abstraction**: إخفاء **التفاصيل** (How)
- **Encapsulation**: إخفاء **البيانات** (What)

---

### 14. ما هو Abstract Class؟

**الإجابة:**

Abstract Class هو كلاس لا يمكن إنشاء objects منه، يستخدم كـ template للكلاسات الأخرى.

**خصائصه:**
- يحتوي على abstract methods (بدون implementation)
- يمكن أن يحتوي على concrete methods
- يمكن أن يحتوي على constructors
- يمكن أن يحتوي على متغيرات
- يجب أن ترث منه الكلاسات الأخرى

**مثال:**

```java
// Abstract Class
abstract class Shape {
    protected String color;

    // Constructor
    public Shape(String color) {
        this.color = color;
    }

    // Abstract method - بدون implementation
    public abstract double calculateArea();

    // Concrete method - مع implementation
    public void displayColor() {
        System.out.println("Color: " + color);
    }
}

// Concrete Class 1
class Circle extends Shape {
    private double radius;

    public Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }

    // يجب تنفيذ abstract method
    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

// Concrete Class 2
class Rectangle extends Shape {
    private double width;
    private double height;

    public Rectangle(String color, double width, double height) {
        super(color);
        this.width = width;
        this.height = height;
    }

    @Override
    public double calculateArea() {
        return width * height;
    }
}

// الاستخدام
public class Main {
    public static void main(String[] args) {
        // Shape shape = new Shape("Red"); // Error: cannot instantiate

        Shape circle = new Circle("Red", 5);
        circle.displayColor();
        System.out.println("Area: " + circle.calculateArea());

        Shape rectangle = new Rectangle("Blue", 4, 6);
        rectangle.displayColor();
        System.out.println("Area: " + rectangle.calculateArea());
    }
}
```

---

### 15. ما هو Interface؟

**الإجابة:**

Interface هو contract يحدد methods يجب أن تنفذها الكلاسات.

**خصائصه (قبل Java 8):**
- كل الـ methods abstract
- كل المتغيرات `public static final`
- لا يحتوي على constructor
- لا يمكن instantiate منه

**خصائصه (Java 8+):**
- يمكن أن يحتوي على default methods
- يمكن أن يحتوي على static methods

**خصائصه (Java 9+):**
- يمكن أن يحتوي على private methods

**مثال:**

```java
// Interface
interface Drawable {
    // abstract method (implicit)
    void draw();
}

interface Colorable {
    void setColor(String color);
    String getColor();
}

// Class تنفذ interface واحد
class Circle implements Drawable {
    @Override
    public void draw() {
        System.out.println("Drawing Circle");
    }
}

// Class تنفذ عدة interfaces
class Rectangle implements Drawable, Colorable {
    private String color;

    @Override
    public void draw() {
        System.out.println("Drawing Rectangle");
    }

    @Override
    public void setColor(String color) {
        this.color = color;
    }

    @Override
    public String getColor() {
        return color;
    }
}

// الاستخدام
public class Main {
    public static void main(String[] args) {
        Drawable circle = new Circle();
        circle.draw();

        Rectangle rect = new Rectangle();
        rect.draw();
        rect.setColor("Red");
        System.out.println("Color: " + rect.getColor());
    }
}
```

**مثال مع Default Method (Java 8+):**
```java
interface Vehicle {
    // Abstract method
    void start();

    // Default method - لها implementation
    default void stop() {
        System.out.println("Vehicle stopped");
    }

    // Static method
    static void service() {
        System.out.println("Vehicle servicing...");
    }
}

class Car implements Vehicle {
    @Override
    public void start() {
        System.out.println("Car starting...");
    }

    // يمكن override default method
    @Override
    public void stop() {
        System.out.println("Car stopped gracefully");
    }
}

// الاستخدام
Car car = new Car();
car.start();      // Car starting...
car.stop();       // Car stopped gracefully
Vehicle.service(); // Vehicle servicing...
```

---

### 16. ما الفرق بين Abstract Class و Interface؟

**الإجابة:**

| Abstract Class | Interface |
|----------------|-----------|
| `abstract class` | `interface` |
| يمكن أن يحتوي على concrete methods | كل الـ methods abstract (قبل Java 8) |
| يمكن أن يحتوي على constructors | لا يحتوي على constructors |
| يمكن أن يحتوي على أي متغيرات | فقط `public static final` |
| وراثة واحدة فقط | تنفيذ عدة interfaces |
| `extends` | `implements` |
| للعلاقة **"is-a"** | للعلاقة **"can-do"** |

**متى نستخدم Abstract Class:**
- عندما يكون هناك shared behavior بين الكلاسات
- عندما تريد constructor
- عندما تريد متغيرات non-static

**متى نستخدم Interface:**
- عندما تريد contract فقط
- عندما تريد multiple inheritance
- عندما الكلاسات ليس بينها علاقة وراثة

**مثال:**
```java
// Abstract Class - shared behavior
abstract class Animal {
    protected String name;

    public Animal(String name) {
        this.name = name;
    }

    public void sleep() {
        System.out.println(name + " is sleeping");
    }

    public abstract void makeSound();
}

// Interface - contract
interface Swimmable {
    void swim();
}

interface Flyable {
    void fly();
}

// Duck يرث من Animal ويمكنه السباحة والطيران
class Duck extends Animal implements Swimmable, Flyable {
    public Duck(String name) {
        super(name);
    }

    @Override
    public void makeSound() {
        System.out.println("Quack!");
    }

    @Override
    public void swim() {
        System.out.println(name + " is swimming");
    }

    @Override
    public void fly() {
        System.out.println(name + " is flying");
    }
}

// Fish يرث من Animal ويمكنه السباحة فقط
class Fish extends Animal implements Swimmable {
    public Fish(String name) {
        super(name);
    }

    @Override
    public void makeSound() {
        System.out.println("Blub blub");
    }

    @Override
    public void swim() {
        System.out.println(name + " is swimming underwater");
    }
}
```

---

## أسئلة متقدمة

### 17. ما هو Coupling و Cohesion؟

**الإجابة:**

#### **Coupling** (الارتباط)
درجة اعتماد كلاس على كلاسات أخرى.

- **Tight Coupling** (ارتباط شديد) ❌
- **Loose Coupling** (ارتباط ضعيف) ✅

```java
// ❌ Tight Coupling
class Car {
    private Engine engine = new PetrolEngine(); // اعتماد مباشر

    public void start() {
        engine.start();
    }
}

// ✅ Loose Coupling
interface Engine {
    void start();
}

class PetrolEngine implements Engine {
    public void start() {
        System.out.println("Petrol engine starting");
    }
}

class ElectricEngine implements Engine {
    public void start() {
        System.out.println("Electric engine starting");
    }
}

class Car {
    private Engine engine;

    public Car(Engine engine) { // Dependency Injection
        this.engine = engine;
    }

    public void start() {
        engine.start();
    }
}

// الاستخدام
Car petrolCar = new Car(new PetrolEngine());
Car electricCar = new Car(new ElectricEngine());
```

#### **Cohesion** (التماسك)
درجة ارتباط methods و attributes داخل كلاس واحد.

- **High Cohesion** (تماسك عالي) ✅
- **Low Cohesion** (تماسك منخفض) ❌

```java
// ❌ Low Cohesion - كل شيء في كلاس واحد
class User {
    void login() { }
    void logout() { }
    void saveToDatabase() { }
    void sendEmail() { }
    void generateReport() { }
}

// ✅ High Cohesion - كل كلاس له مسؤولية واحدة
class User {
    void login() { }
    void logout() { }
}

class UserRepository {
    void save(User user) { }
}

class EmailService {
    void send(User user) { }
}

class ReportGenerator {
    void generate(User user) { }
}
```

**الهدف:**
- **Low Coupling** (اعتماد قليل بين الكلاسات)
- **High Cohesion** (ارتباط قوي داخل الكلاس)

---

### 18. ما هو Composition vs Aggregation؟

**الإجابة:**

كلاهما نوع من العلاقات بين الكلاسات ("has-a" relationship).

#### **Composition** (تركيب)
علاقة قوية - الجزء لا يمكن أن يوجد بدون الكل.

```java
// Engine لا يمكن أن يوجد بدون Car
class Engine {
    public void start() {
        System.out.println("Engine starting");
    }
}

class Car {
    private Engine engine;

    public Car() {
        this.engine = new Engine(); // Car يملك Engine
    }

    // عند حذف Car، Engine يُحذف أيضاً
}
```

#### **Aggregation** (تجميع)
علاقة ضعيفة - الجزء يمكن أن يوجد بشكل مستقل.

```java
// Student يمكن أن يوجد بدون Department
class Student {
    private String name;

    public Student(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

class Department {
    private List<Student> students;

    public Department() {
        this.students = new ArrayList<>();
    }

    public void addStudent(Student student) {
        students.add(student);
    }

    // عند حذف Department، Students يبقون موجودين
}

// الاستخدام
Student s1 = new Student("Ahmed");
Student s2 = new Student("Fatima");

Department dept = new Department();
dept.addStudent(s1);
dept.addStudent(s2);

// لو حذفنا dept، s1 و s2 ما زالوا موجودين
```

**الفرق:**

| Composition | Aggregation |
|-------------|-------------|
| علاقة قوية | علاقة ضعيفة |
| الجزء لا يوجد بدون الكل | الجزء يوجد بشكل مستقل |
| Car ← Engine | Department ← Students |
| House ← Rooms | University ← Professors |

---

### 19. ما هو Association vs Dependency؟

**الإجابة:**

#### **Association**
علاقة بين كلاسين حيث أحدهما يستخدم أو يملك الآخر.

```java
class Teacher {
    private String name;

    public Teacher(String name) {
        this.name = name;
    }

    public void teach(Student student) {
        System.out.println(name + " is teaching " + student.getName());
    }
}

class Student {
    private String name;

    public Student(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

#### **Dependency**
كلاس يعتمد على كلاس آخر بشكل مؤقت (method parameter أو local variable).

```java
class EmailService {
    public void sendEmail(User user, String message) {
        // EmailService يعتمد على User بشكل مؤقت
        System.out.println("Sending to " + user.getEmail() + ": " + message);
    }
}

class User {
    private String email;

    public User(String email) {
        this.email = email;
    }

    public String getEmail() {
        return email;
    }
}
```

---

### 20. ما هو Final Keyword؟

**الإجابة:**

`final` يمنع التعديل/التغيير.

#### **Final Variable**
قيمتها لا يمكن تغييرها (constant).

```java
final int MAX_SIZE = 100;
// MAX_SIZE = 200; // Error!

class Circle {
    final double PI = 3.14159;

    public double calculateArea(double radius) {
        return PI * radius * radius;
    }
}
```

#### **Final Method**
لا يمكن override في الكلاسات الفرعية.

```java
class Parent {
    final void display() {
        System.out.println("Cannot override this");
    }
}

class Child extends Parent {
    // Error: cannot override
    // void display() { }
}
```

#### **Final Class**
لا يمكن الوراثة منها.

```java
final class Utility {
    public static void print() {
        System.out.println("Utility");
    }
}

// Error: cannot extend
// class MyUtility extends Utility { }
```

**أمثلة من Java:**
- `String` class is final
- `Math` class is final
- `Integer`, `Double` classes are final

---

## أسئلة عملية

### 21. صمم نظام لإدارة المكتبة باستخدام OOP

**الإجابة:**

```java
// Abstract Class للمواد
abstract class LibraryItem {
    protected String id;
    protected String title;
    protected boolean isAvailable;

    public LibraryItem(String id, String title) {
        this.id = id;
        this.title = title;
        this.isAvailable = true;
    }

    public abstract void displayInfo();

    public void checkOut() {
        if (isAvailable) {
            isAvailable = false;
            System.out.println(title + " has been checked out");
        } else {
            System.out.println(title + " is not available");
        }
    }

    public void returnItem() {
        isAvailable = true;
        System.out.println(title + " has been returned");
    }

    public boolean isAvailable() {
        return isAvailable;
    }
}

// Concrete Classes
class Book extends LibraryItem {
    private String author;
    private int pages;

    public Book(String id, String title, String author, int pages) {
        super(id, title);
        this.author = author;
        this.pages = pages;
    }

    @Override
    public void displayInfo() {
        System.out.println("Book: " + title + " by " + author + " (" + pages + " pages)");
    }
}

class DVD extends LibraryItem {
    private String director;
    private int duration;

    public DVD(String id, String title, String director, int duration) {
        super(id, title);
        this.director = director;
        this.duration = duration;
    }

    @Override
    public void displayInfo() {
        System.out.println("DVD: " + title + " directed by " + director + " (" + duration + " min)");
    }
}

class Magazine extends LibraryItem {
    private String publisher;
    private int issueNumber;

    public Magazine(String id, String title, String publisher, int issueNumber) {
        super(id, title);
        this.publisher = publisher;
        this.issueNumber = issueNumber;
    }

    @Override
    public void displayInfo() {
        System.out.println("Magazine: " + title + " - Issue #" + issueNumber + " by " + publisher);
    }
}

// Member Class
class Member {
    private String memberId;
    private String name;
    private List<LibraryItem> borrowedItems;

    public Member(String memberId, String name) {
        this.memberId = memberId;
        this.name = name;
        this.borrowedItems = new ArrayList<>();
    }

    public void borrowItem(LibraryItem item) {
        if (item.isAvailable()) {
            item.checkOut();
            borrowedItems.add(item);
        }
    }

    public void returnItem(LibraryItem item) {
        if (borrowedItems.contains(item)) {
            item.returnItem();
            borrowedItems.remove(item);
        }
    }

    public void displayBorrowedItems() {
        System.out.println(name + "'s borrowed items:");
        for (LibraryItem item : borrowedItems) {
            item.displayInfo();
        }
    }
}

// Library Class
class Library {
    private List<LibraryItem> items;
    private List<Member> members;

    public Library() {
        this.items = new ArrayList<>();
        this.members = new ArrayList<>();
    }

    public void addItem(LibraryItem item) {
        items.add(item);
    }

    public void addMember(Member member) {
        members.add(member);
    }

    public void displayAllItems() {
        System.out.println("Library Items:");
        for (LibraryItem item : items) {
            item.displayInfo();
            System.out.println("Available: " + item.isAvailable());
            System.out.println();
        }
    }
}

// الاستخدام
public class LibrarySystem {
    public static void main(String[] args) {
        Library library = new Library();

        // إضافة مواد
        Book book1 = new Book("B001", "Clean Code", "Robert Martin", 464);
        DVD dvd1 = new DVD("D001", "Inception", "Christopher Nolan", 148);
        Magazine mag1 = new Magazine("M001", "National Geographic", "NG Partners", 150);

        library.addItem(book1);
        library.addItem(dvd1);
        library.addItem(mag1);

        // إضافة أعضاء
        Member member1 = new Member("M001", "Ahmed");
        library.addMember(member1);

        // استعارة
        member1.borrowItem(book1);
        member1.borrowItem(dvd1);

        // عرض المواد المستعارة
        member1.displayBorrowedItems();

        // إرجاع
        member1.returnItem(book1);

        // عرض كل المواد
        library.displayAllItems();
    }
}
```

**المبادئ المطبقة:**
- ✅ **Encapsulation**: البيانات private
- ✅ **Inheritance**: Book, DVD, Magazine ترث من LibraryItem
- ✅ **Polymorphism**: displayInfo() مختلف لكل نوع
- ✅ **Abstraction**: LibraryItem abstract class

---

## ملخص

### المبادئ الأربعة:

1. **Encapsulation** (التغليف)
   - إخفاء البيانات
   - استخدام getters/setters
   - Data protection

2. **Inheritance** (الوراثة)
   - إعادة استخدام الكود
   - علاقة "is-a"
   - Parent/Child relationship

3. **Polymorphism** (تعدد الأشكال)
   - Overloading (compile-time)
   - Overriding (runtime)
   - نفس الواجهة، سلوك مختلف

4. **Abstraction** (التجريد)
   - إخفاء التفاصيل
   - Abstract Classes
   - Interfaces

### نصائح للمقابلات:

✅ **اعط أمثلة عملية**
✅ **اشرح الفوائد**
✅ **قارن بين المفاهيم**
✅ **اكتب كود نظيف**
✅ **اشرح Trade-offs**

---

**حظاً موفقاً في المقابلة! 🚀**
