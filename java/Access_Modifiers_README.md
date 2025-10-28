# Access Modifiers في Java

## ما هي Access Modifiers؟
Access Modifiers (معدلات الوصول) هي كلمات محجوزة تحدد مستوى الوصول (accessibility/visibility) للـ classes، methods، variables، و constructors في Java.

## الأنواع الأربعة لـ Access Modifiers

### 1. private
- **أضيق مستوى وصول**
- يمكن الوصول إليه **داخل نفس الـ class فقط**
- لا يمكن استخدامه مع top-level classes

```java
public class Person {
    private String ssn = "123-45-6789";  // متغير private

    private void showSSN() {  // method private
        System.out.println("SSN: " + ssn);  // يمكن الوصول داخل نفس الـ class
    }

    public void displayInfo() {
        showSSN();  // يمكن استدعاء private method داخل نفس الـ class
    }
}

class Test {
    public static void main(String[] args) {
        Person p = new Person();
        // p.ssn;  // خطأ! لا يمكن الوصول لـ private variable
        // p.showSSN();  // خطأ! لا يمكن الوصول لـ private method
        p.displayInfo();  // يعمل - public method
    }
}
```

### 2. default (Package-Private)
- **بدون keyword** (لا نكتب أي access modifier)
- يمكن الوصول إليه **داخل نفس الـ package فقط**

```java
// في ملف Employee.java في package com.company
package com.company;

class Employee {  // default class
    int id;  // default variable

    void work() {  // default method
        System.out.println("Working...");
    }
}

// في ملف Manager.java في نفس الـ package
package com.company;

class Manager {
    void manage() {
        Employee emp = new Employee();  // يعمل - نفس الـ package
        emp.id = 100;  // يعمل
        emp.work();  // يعمل
    }
}

// في ملف Client.java في package مختلف
package com.client;
import com.company.*;

class Client {
    void useEmployee() {
        // Employee emp = new Employee();  // خطأ! package مختلف
    }
}
```

### 3. protected
- يمكن الوصول إليه من:
  - **نفس الـ class**
  - **نفس الـ package**
  - **الـ subclasses (حتى لو في package مختلف)**

```java
// في package com.animals
package com.animals;

public class Animal {
    protected String name = "Animal";

    protected void makeSound() {
        System.out.println("Some sound");
    }
}

// في نفس الـ package
package com.animals;

class Zoo {
    void showAnimal() {
        Animal animal = new Animal();
        System.out.println(animal.name);  // يعمل - نفس الـ package
        animal.makeSound();  // يعمل
    }
}

// في package مختلف - subclass
package com.pets;
import com.animals.Animal;

public class Dog extends Animal {
    void bark() {
        System.out.println(name);  // يعمل - inherited protected member
        makeSound();  // يعمل - inherited protected method

        // لكن انتبه:
        Animal otherAnimal = new Animal();
        // System.out.println(otherAnimal.name);  // خطأ! ليس through inheritance
        // otherAnimal.makeSound();  // خطأ!
    }
}

// في package مختلف - ليس subclass
package com.pets;
import com.animals.Animal;

class PetShop {
    void sellAnimal() {
        Animal animal = new Animal();
        // System.out.println(animal.name);  // خطأ! package مختلف وليس subclass
        // animal.makeSound();  // خطأ!
    }
}
```

### 4. public
- **أوسع مستوى وصول**
- يمكن الوصول إليه **من أي مكان**

```java
// في أي package
public class Calculator {
    public int value;

    public void calculate() {
        System.out.println("Calculating...");
    }
}

// من أي مكان آخر
class AnyClass {
    void useCalculator() {
        Calculator calc = new Calculator();
        calc.value = 10;  // يعمل من أي مكان
        calc.calculate();  // يعمل من أي مكان
    }
}
```

## جدول المقارنة الشامل

| Access Modifier | Same Class | Same Package | Subclass (Different Package) | Different Package |
|----------------|------------|--------------|------------------------------|-------------------|
| **private**    | ✅ Yes     | ❌ No        | ❌ No                        | ❌ No            |
| **default**    | ✅ Yes     | ✅ Yes       | ❌ No                        | ❌ No            |
| **protected**  | ✅ Yes     | ✅ Yes       | ✅ Yes (through inheritance) | ❌ No            |
| **public**     | ✅ Yes     | ✅ Yes       | ✅ Yes                       | ✅ Yes           |

## الفرق بين protected و default

### الفرق الأساسي:
- **default**: يمكن الوصول من **نفس الـ package فقط**
- **protected**: يمكن الوصول من **نفس الـ package + subclasses في أي package**

### مثال توضيحي للفرق:

```java
// Package: com.base
package com.base;

public class BaseClass {
    int defaultVar = 10;           // default
    protected int protectedVar = 20;  // protected

    void defaultMethod() {          // default
        System.out.println("Default method");
    }

    protected void protectedMethod() { // protected
        System.out.println("Protected method");
    }
}

// Package: com.base (نفس الـ package)
package com.base;

class SamePackageClass {
    void test() {
        BaseClass base = new BaseClass();
        System.out.println(base.defaultVar);     // ✅ يعمل
        System.out.println(base.protectedVar);   // ✅ يعمل
        base.defaultMethod();                    // ✅ يعمل
        base.protectedMethod();                  // ✅ يعمل
    }
}

// Package: com.derived (package مختلف - Subclass)
package com.derived;
import com.base.BaseClass;

class DerivedClass extends BaseClass {
    void test() {
        // Through inheritance:
        System.out.println(defaultVar);      // ❌ خطأ - default لا يعمل
        System.out.println(protectedVar);    // ✅ يعمل - protected يعمل
        defaultMethod();                     // ❌ خطأ - default لا يعمل
        protectedMethod();                   // ✅ يعمل - protected يعمل

        // Through object reference:
        BaseClass base = new BaseClass();
        // System.out.println(base.defaultVar);   // ❌ خطأ
        // System.out.println(base.protectedVar); // ❌ خطأ - ليس through inheritance
    }
}

// Package: com.other (package مختلف - ليس Subclass)
package com.other;
import com.base.BaseClass;

class OtherClass {
    void test() {
        BaseClass base = new BaseClass();
        // System.out.println(base.defaultVar);     // ❌ خطأ
        // System.out.println(base.protectedVar);   // ❌ خطأ
        // base.defaultMethod();                    // ❌ خطأ
        // base.protectedMethod();                  // ❌ خطأ
    }
}
```

## قواعد مهمة

### 1. Classes والـ Access Modifiers
```java
public class PublicClass { }      // ✅ يمكن
class DefaultClass { }            // ✅ يمكن (default)
// private class PrivateClass { }  // ❌ خطأ - top-level class
// protected class ProtectedClass { } // ❌ خطأ - top-level class

// لكن nested classes يمكن:
public class Outer {
    private class PrivateInner { }     // ✅ يمكن
    protected class ProtectedInner { } // ✅ يمكن
    public class PublicInner { }       // ✅ يمكن
    class DefaultInner { }              // ✅ يمكن
}
```

### 2. Constructors
```java
public class Person {
    private Person() { }      // Singleton pattern
    protected Person(int id) { }  // للـ subclasses
    Person(String name) { }   // package access
    public Person(String name, int age) { }  // public access
}
```

### 3. Interfaces
```java
public interface MyInterface {
    // جميع الـ methods في interface هي public abstract by default
    void method1();  // public abstract

    // Variables في interface هي public static final by default
    int CONSTANT = 100;  // public static final

    // من Java 8+
    default void defaultMethod() { }  // public
    static void staticMethod() { }    // public

    // من Java 9+
    private void privateMethod() { }  // private helper methods
}
```

## Pseudo Code - أمثلة Find the Output

### مثال 1: Access Modifiers مختلفة
```java
package com.test;

public class A {
    private int a = 1;
    int b = 2;              // default
    protected int c = 3;
    public int d = 4;

    private void methodA() { System.out.println("A"); }
    void methodB() { System.out.println("B"); }
    protected void methodC() { System.out.println("C"); }
    public void methodD() { System.out.println("D"); }
}

class B {
    public static void main(String[] args) {
        A obj = new A();
        // System.out.println(obj.a);  // خطأ - private
        System.out.println(obj.b);     // 2 - same package
        System.out.println(obj.c);     // 3 - same package
        System.out.println(obj.d);     // 4 - public

        // obj.methodA();  // خطأ - private
        obj.methodB();     // B - same package
        obj.methodC();     // C - same package
        obj.methodD();     // D - public
    }
}

// Output:
// 2
// 3
// 4
// B
// C
// D
```

### مثال 2: Inheritance و Access
```java
package com.parent;

public class Parent {
    private String private_var = "private";
    String default_var = "default";
    protected String protected_var = "protected";
    public String public_var = "public";

    private void privateMethod() {
        System.out.println("Parent private");
    }

    void defaultMethod() {
        System.out.println("Parent default");
    }

    protected void protectedMethod() {
        System.out.println("Parent protected");
    }

    public void publicMethod() {
        System.out.println("Parent public");
    }
}

package com.child;
import com.parent.Parent;

public class Child extends Parent {
    public void testAccess() {
        // System.out.println(private_var);  // خطأ - private
        // System.out.println(default_var);  // خطأ - different package
        System.out.println(protected_var);   // "protected" - inherited
        System.out.println(public_var);      // "public"

        // privateMethod();   // خطأ - private
        // defaultMethod();   // خطأ - different package
        protectedMethod();    // "Parent protected"
        publicMethod();       // "Parent public"
    }

    public static void main(String[] args) {
        Child child = new Child();
        child.testAccess();

        Parent parent = new Parent();
        // System.out.println(parent.protected_var); // خطأ - not through inheritance
        System.out.println(parent.public_var);       // "public"
    }
}

// Output:
// protected
// public
// Parent protected
// Parent public
// public
```

### مثال 3: Method Overriding و Access Modifiers
```java
class Parent {
    protected void show() {
        System.out.println("Parent show");
    }
}

class Child extends Parent {
    // @Override
    // private void show() { }  // خطأ - can't reduce visibility

    @Override
    protected void show() {     // ✅ same access
        System.out.println("Child protected show");
    }

    // أو
    // @Override
    // public void show() {     // ✅ wider access
    //     System.out.println("Child public show");
    // }
}

public class Test {
    public static void main(String[] args) {
        Parent p = new Child();
        p.show();  // "Child protected show"
    }
}

// Output: Child protected show
```

### مثال 4: Static Members
```java
public class StaticTest {
    private static int privateStatic = 1;
    static int defaultStatic = 2;
    protected static int protectedStatic = 3;
    public static int publicStatic = 4;

    public static void main(String[] args) {
        System.out.println(privateStatic);    // 1 - same class
        System.out.println(defaultStatic);    // 2
        System.out.println(protectedStatic);  // 3
        System.out.println(publicStatic);     // 4
    }
}

class AnotherClass {
    public static void main(String[] args) {
        // System.out.println(StaticTest.privateStatic);  // خطأ
        System.out.println(StaticTest.defaultStatic);     // 2 - same package
        System.out.println(StaticTest.protectedStatic);   // 3 - same package
        System.out.println(StaticTest.publicStatic);      // 4
    }
}

// Output from AnotherClass:
// 2
// 3
// 4
```

### مثال 5: Constructor Access
```java
public class ConstructorTest {
    private ConstructorTest(int x) {
        System.out.println("Private: " + x);
    }

    ConstructorTest(String s) {
        System.out.println("Default: " + s);
    }

    protected ConstructorTest(double d) {
        System.out.println("Protected: " + d);
    }

    public ConstructorTest() {
        this(10);  // يمكن استدعاء private constructor من نفس الـ class
        System.out.println("Public");
    }

    public static void main(String[] args) {
        new ConstructorTest();
    }
}

// Output:
// Private: 10
// Public
```

## Best Practices

### 1. Principle of Least Privilege
```java
public class BankAccount {
    private double balance;  // private - أضيق مستوى ممكن

    public double getBalance() {  // public getter
        return balance;
    }

    protected void audit() {  // protected - للـ subclasses فقط
        System.out.println("Auditing...");
    }

    void logTransaction() {  // default - package-level utility
        System.out.println("Logging...");
    }
}
```

### 2. Encapsulation
```java
public class Employee {
    private int id;          // hide implementation
    private String name;
    private double salary;

    // Public interface
    public int getId() { return id; }
    public String getName() { return name; }

    // Protected for subclasses
    protected void calculateBonus() { }

    // Package-private helper
    void validateData() { }
}
```

### 3. Framework Design
```java
public abstract class Framework {
    // Public API
    public final void execute() {
        init();
        doWork();
        cleanup();
    }

    // Protected hooks for subclasses
    protected abstract void doWork();

    // Private/default implementation details
    private void init() { }
    private void cleanup() { }
}
```

## الخلاصة

- **private**: الأكثر تقييداً - نفس الـ class فقط
- **default**: نفس الـ package فقط
- **protected**: نفس الـ package + subclasses
- **public**: يمكن الوصول من أي مكان

استخدم أضيق مستوى وصول ممكن (Principle of Least Privilege) لتحسين الـ encapsulation والأمان في الكود!