# دليل أسئلة مقابلات Java الشامل

## المحتويات
1. [أساسيات Java](#أساسيات-java)
2. [البرمجة الكائنية (OOP)](#البرمجة-الكائنية-oop)
3. [Collections Framework](#collections-framework)
4. [Multithreading](#multithreading)
5. [Exception Handling](#exception-handling)
6. [Java 8+ Features](#java-8-features)
7. [Memory Management](#memory-management)

---

## أساسيات Java

### 1. ما الفرق بين JDK و JRE و JVM؟

**الإجابة:**

- **JVM (Java Virtual Machine):**
  - محرك تنفيذ البرنامج الذي يقوم بتشغيل bytecode
  - يوفر بيئة Runtime للتطبيقات
  - مسؤول عن Garbage Collection وإدارة الذاكرة

- **JRE (Java Runtime Environment):**
  - يحتوي على JVM + المكتبات القياسية
  - كل ما تحتاجه لتشغيل تطبيق Java
  - لا يحتوي على أدوات التطوير (compiler)

- **JDK (Java Development Kit):**
  - يحتوي على JRE + أدوات التطوير (javac, debugger, etc.)
  - ما تحتاجه لتطوير تطبيقات Java
  - النسخة الكاملة للمطورين

```
JDK = JRE + Development Tools
JRE = JVM + Library Classes
```

---

### 2. ما الفرق بين `==` و `equals()`؟

**الإجابة:**

**`==` Operator:**
- يقارن **المراجع (References)** للكائنات
- للأنواع البدائية (int, char, etc.) يقارن القيم
- لا يمكن تخصيصه (override)

**`equals()` Method:**
- يقارن **محتوى** الكائنات
- يمكن تخصيصه في الكلاسات الخاصة بك
- موجود في كل الكائنات (ورث من Object)

**مثال عملي:**
```java
String s1 = new String("Hello");
String s2 = new String("Hello");
String s3 = s1;

System.out.println(s1 == s2);      // false (مراجع مختلفة)
System.out.println(s1.equals(s2)); // true (نفس المحتوى)
System.out.println(s1 == s3);      // true (نفس المرجع)
```

---

### 3. ما الفرق بين String و StringBuilder و StringBuffer؟

**الإجابة:**

| الخاصية | String | StringBuilder | StringBuffer |
|---------|--------|--------------|--------------|
| Mutability | Immutable (غير قابل للتغيير) | Mutable | Mutable |
| Thread Safety | Thread-safe | Not thread-safe | Thread-safe |
| الأداء | بطيء في التعديلات المتكررة | سريع جداً | أبطأ من StringBuilder |
| الاستخدام | نصوص ثابتة | single-threaded | multi-threaded |

**مثال عملي:**
```java
// String - creates new object each time
String str = "Hello";
str = str + " World"; // كائن جديد تماماً

// StringBuilder - modifies same object (أسرع)
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World"); // نفس الكائن

// StringBuffer - thread-safe version
StringBuffer sbf = new StringBuffer("Hello");
sbf.append(" World"); // synchronized methods
```

**متى تستخدم كل واحد:**
- **String:** للنصوص الثابتة التي لا تتغير
- **StringBuilder:** للتعديلات المتكررة في single thread
- **StringBuffer:** للتعديلات المتكررة في multi-threaded environment

---

## البرمجة الكائنية (OOP)

### 4. اشرح المبادئ الأربعة للـ OOP

**الإجابة:**

#### 1. Encapsulation (التغليف)
إخفاء تفاصيل التنفيذ الداخلية وإظهار ما هو ضروري فقط

```java
public class BankAccount {
    private double balance; // مخفي عن الخارج

    // الوصول المحكوم
    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
}
```

**الفوائد:**
- حماية البيانات من التعديل غير الصحيح
- سهولة الصيانة
- Flexibility في تغيير التنفيذ الداخلي

---

#### 2. Inheritance (الوراثة)
إنشاء كلاس جديد من كلاس موجود

```java
// Parent class
public class Animal {
    protected String name;

    public void eat() {
        System.out.println("Animal is eating");
    }
}

// Child class
public class Dog extends Animal {
    public void bark() {
        System.out.println("Dog is barking");
    }

    @Override
    public void eat() {
        System.out.println("Dog is eating");
    }
}
```

**الفوائد:**
- إعادة استخدام الكود (Code Reusability)
- تنظيم هرمي للكلاسات
- Polymorphism

---

#### 3. Polymorphism (تعدد الأشكال)
القدرة على استخدام نفس الواجهة لأنواع مختلفة

**أنواعه:**
- **Compile-time (Method Overloading):**
```java
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }

    public double add(double a, double b) {
        return a + b;
    }

    public int add(int a, int b, int c) {
        return a + b + c;
    }
}
```

- **Runtime (Method Overriding):**
```java
Animal animal = new Dog(); // reference من نوع Animal
animal.eat(); // سينفذ eat() من Dog
```

---

#### 4. Abstraction (التجريد)
إخفاء التفاصيل المعقدة وإظهار الوظائف الأساسية فقط

```java
// Abstract class
public abstract class Shape {
    abstract double calculateArea();

    public void display() {
        System.out.println("Area: " + calculateArea());
    }
}

// Interface
public interface Drawable {
    void draw();
}

// Implementation
public class Circle extends Shape implements Drawable {
    private double radius;

    @Override
    double calculateArea() {
        return Math.PI * radius * radius;
    }

    @Override
    public void draw() {
        System.out.println("Drawing circle");
    }
}
```

---

### 5. ما الفرق بين Abstract Class و Interface؟

**الإجابة:**

| الخاصية | Abstract Class | Interface |
|---------|---------------|-----------|
| Multiple Inheritance | لا تدعم | تدعم |
| Constructor | يمكن أن يحتوي | لا يمكن |
| Variables | أي نوع من المتغيرات | public static final فقط |
| Methods | abstract و concrete | abstract (قبل Java 8) |
| Access Modifiers | أي modifier | public فقط (قبل Java 9) |
| متى تستخدم | "is-a" relationship | "can-do" relationship |

**مثال عملي:**
```java
// Abstract Class - عندما يكون هناك shared behavior
public abstract class Vehicle {
    protected String brand;

    // Constructor
    public Vehicle(String brand) {
        this.brand = brand;
    }

    // Concrete method
    public void start() {
        System.out.println("Vehicle starting...");
    }

    // Abstract method
    public abstract void accelerate();
}

// Interface - عندما تريد contract فقط
public interface Flyable {
    void fly();
    void land();
}

// استخدام الاثنين
public class Airplane extends Vehicle implements Flyable {
    public Airplane(String brand) {
        super(brand);
    }

    @Override
    public void accelerate() {
        System.out.println("Airplane accelerating");
    }

    @Override
    public void fly() {
        System.out.println("Airplane flying");
    }

    @Override
    public void land() {
        System.out.println("Airplane landing");
    }
}
```

---

## Collections Framework

### 6. اشرح الفرق بين ArrayList و LinkedList

**الإجابة:**

| الخاصية | ArrayList | LinkedList |
|---------|-----------|-----------|
| التنفيذ الداخلي | Dynamic Array | Doubly Linked List |
| الوصول للعناصر | O(1) - سريع جداً | O(n) - بطيء |
| الإضافة/الحذف | O(n) - بطيء (في الوسط) | O(1) - سريع |
| Memory Overhead | أقل | أكثر (pointers) |
| الاستخدام المثالي | كثرة القراءة | كثرة الإضافة/الحذف |

**مثال عملي:**
```java
// ArrayList - للقراءة المتكررة
List<String> arrayList = new ArrayList<>();
arrayList.add("A");
arrayList.add("B");
arrayList.add("C");
String element = arrayList.get(1); // O(1) - سريع

// LinkedList - للإضافة/الحذف المتكرر
List<String> linkedList = new LinkedList<>();
linkedList.add("A");
linkedList.add(0, "B"); // O(1) في البداية
linkedList.remove(0);    // O(1) في البداية

// اختبار الأداء
// ArrayList: أفضل للوصول العشوائي
for (int i = 0; i < arrayList.size(); i++) {
    System.out.println(arrayList.get(i)); // سريع
}

// LinkedList: أفضل للإضافة في البداية/النهاية
((LinkedList<String>) linkedList).addFirst("Start");
((LinkedList<String>) linkedList).addLast("End");
```

**متى تستخدم:**
- **ArrayList:** عندما تحتاج وصول سريع بـ index
- **LinkedList:** عندما تحتاج إضافة/حذف متكرر في البداية/النهاية

---

### 7. ما الفرق بين HashMap و HashTable و ConcurrentHashMap؟

**الإجابة:**

| الخاصية | HashMap | HashTable | ConcurrentHashMap |
|---------|---------|-----------|-------------------|
| Thread Safety | لا | نعم | نعم |
| Null Keys/Values | يسمح | لا يسمح | لا يسمح |
| الأداء | الأسرع | بطيء | سريع في multi-threading |
| Synchronization | لا | Full lock | Segment locking |
| Java Version | 1.2+ | 1.0+ (Legacy) | 1.5+ |

**شرح تفصيلي:**

```java
// HashMap - single threaded
Map<String, Integer> hashMap = new HashMap<>();
hashMap.put("One", 1);
hashMap.put(null, 0);     // يسمح بـ null key
hashMap.put("Two", null); // يسمح بـ null value

// HashTable - thread-safe (قديم)
Map<String, Integer> hashTable = new Hashtable<>();
hashTable.put("One", 1);
// hashTable.put(null, 0); // سيرمي NullPointerException

// ConcurrentHashMap - modern thread-safe
Map<String, Integer> concurrentMap = new ConcurrentHashMap<>();
concurrentMap.put("One", 1);
// concurrentMap.put(null, 0); // سيرمي NullPointerException

// اختبار Thread Safety
// HashMap - مشكلة في multi-threading
Map<String, Integer> map1 = new HashMap<>();
Runnable task1 = () -> {
    for (int i = 0; i < 1000; i++) {
        map1.put("Key" + i, i); // قد يسبب مشاكل
    }
};

// ConcurrentHashMap - آمن في multi-threading
Map<String, Integer> map2 = new ConcurrentHashMap<>();
Runnable task2 = () -> {
    for (int i = 0; i < 1000; i++) {
        map2.put("Key" + i, i); // آمن تماماً
    }
};
```

**الفرق في Locking:**
- **HashTable:** يقفل الـ Map بالكامل
- **ConcurrentHashMap:** يقفل segments فقط (أسرع)

---

### 8. ما الفرق بين Set و List؟

**الإجابة:**

| الخاصية | List | Set |
|---------|------|-----|
| التكرار | يسمح بالعناصر المكررة | لا يسمح بالتكرار |
| الترتيب | يحفظ ترتيب الإدخال | قد يحفظ أو لا (حسب النوع) |
| Index Access | يدعم | لا يدعم |
| Null Elements | يسمح بعدة null | يسمح بـ null واحد (في HashSet) |

**أنواع Set:**

```java
// HashSet - لا يحفظ الترتيب
Set<String> hashSet = new HashSet<>();
hashSet.add("B");
hashSet.add("A");
hashSet.add("C");
hashSet.add("A"); // لن يضاف (مكرر)
System.out.println(hashSet); // [A, B, C] أو ترتيب عشوائي

// LinkedHashSet - يحفظ ترتيب الإدخال
Set<String> linkedHashSet = new LinkedHashSet<>();
linkedHashSet.add("B");
linkedHashSet.add("A");
linkedHashSet.add("C");
System.out.println(linkedHashSet); // [B, A, C]

// TreeSet - ترتيب طبيعي (sorted)
Set<String> treeSet = new TreeSet<>();
treeSet.add("B");
treeSet.add("A");
treeSet.add("C");
System.out.println(treeSet); // [A, B, C]

// أنواع List
List<String> arrayList = new ArrayList<>();
arrayList.add("A");
arrayList.add("A"); // يسمح بالتكرار
arrayList.add("B");
System.out.println(arrayList.get(0)); // الوصول بـ index
```

---

## Multithreading

### 9. ما الفرق بين Process و Thread؟

**الإجابة:**

| الخاصية | Process | Thread |
|---------|---------|--------|
| التعريف | برنامج مستقل يعمل | وحدة تنفيذ داخل process |
| Memory | لكل process ذاكرة منفصلة | Threads يشاركون نفس الذاكرة |
| Communication | IPC (معقد) | سهل (shared memory) |
| Creation Cost | مكلف | رخيص |
| Context Switching | بطيء | سريع |

**مثال عملي:**

```java
// إنشاء Thread - الطريقة الأولى (Extending Thread)
class MyThread extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + ": " + i);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

// إنشاء Thread - الطريقة الثانية (Implementing Runnable)
class MyRunnable implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + ": " + i);
        }
    }
}

// الاستخدام
public class ThreadExample {
    public static void main(String[] args) {
        // الطريقة الأولى
        MyThread thread1 = new MyThread();
        thread1.start();

        // الطريقة الثانية (مفضلة)
        Thread thread2 = new Thread(new MyRunnable());
        thread2.start();

        // Lambda (Java 8+)
        Thread thread3 = new Thread(() -> {
            System.out.println("Lambda thread running");
        });
        thread3.start();
    }
}
```

**لماذا Runnable أفضل من Thread:**
- Java لا تدعم multiple inheritance
- أكثر مرونة
- يمكن استخدامه مع Thread Pools

---

### 10. اشرح Synchronization في Java

**الإجابة:**

Synchronization يضمن أن thread واحد فقط يصل للـ resource في وقت معين.

**أنواع Synchronization:**

```java
// 1. Synchronized Method
class Counter {
    private int count = 0;

    // كل الـ method محمي
    public synchronized void increment() {
        count++;
    }

    public synchronized int getCount() {
        return count;
    }
}

// 2. Synchronized Block
class Counter2 {
    private int count = 0;
    private Object lock = new Object();

    public void increment() {
        // جزء معين فقط محمي
        synchronized(lock) {
            count++;
        }
    }
}

// 3. Static Synchronization
class Counter3 {
    private static int count = 0;

    // يقفل الـ class نفسه
    public static synchronized void increment() {
        count++;
    }
}

// مثال على مشكلة بدون Synchronization
class UnsafeCounter {
    private int count = 0;

    public void increment() {
        count++; // عملية غير atomic!
        // 1. قراءة count
        // 2. إضافة 1
        // 3. كتابة النتيجة
        // قد يحدث race condition بين هذه الخطوات
    }
}

// الاستخدام
public class SyncExample {
    public static void main(String[] args) throws InterruptedException {
        Counter counter = new Counter();

        // إنشاء 1000 thread يزيدون العداد
        Thread[] threads = new Thread[1000];
        for (int i = 0; i < 1000; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 100; j++) {
                    counter.increment();
                }
            });
            threads[i].start();
        }

        // انتظار كل الـ threads
        for (Thread t : threads) {
            t.join();
        }

        System.out.println("Count: " + counter.getCount()); // 100000
    }
}
```

**مشاكل Synchronization:**
- **Deadlock:** threads يقفلون بعضهم
- **Performance:** قد يبطئ البرنامج
- **Starvation:** thread لا يحصل على الـ lock أبداً

---

## Exception Handling

### 11. ما الفرق بين Checked و Unchecked Exceptions؟

**الإجابة:**

**Checked Exceptions:**
- يجب التعامل معها compile-time
- ترث من `Exception` (ما عدا RuntimeException)
- أمثلة: IOException, SQLException, ClassNotFoundException

**Unchecked Exceptions:**
- لا يجب التعامل معها compile-time
- ترث من `RuntimeException`
- أمثلة: NullPointerException, ArrayIndexOutOfBoundsException

```java
// Checked Exception - يجب try-catch أو throws
public void readFile() throws IOException {
    FileReader file = new FileReader("test.txt");
    // أو
    try {
        FileReader file2 = new FileReader("test.txt");
    } catch (IOException e) {
        e.printStackTrace();
    }
}

// Unchecked Exception - اختياري
public void divideNumbers(int a, int b) {
    int result = a / b; // قد يرمي ArithmeticException
    // لا يجبرك Compiler على try-catch
}

// Hierarchy
/*
Throwable
├── Error (unchecked - system errors)
│   ├── OutOfMemoryError
│   └── StackOverflowError
└── Exception
    ├── RuntimeException (unchecked)
    │   ├── NullPointerException
    │   ├── ArrayIndexOutOfBoundsException
    │   └── ArithmeticException
    └── Checked Exceptions
        ├── IOException
        ├── SQLException
        └── ClassNotFoundException
*/
```

---

### 12. اشرح try-catch-finally و try-with-resources

**الإجابة:**

**try-catch-finally:**
```java
public void traditionalTryCatch() {
    FileReader reader = null;
    try {
        reader = new FileReader("file.txt");
        // قراءة الملف
        int data = reader.read();

    } catch (IOException e) {
        System.out.println("Error: " + e.getMessage());

    } finally {
        // ينفذ دائماً (حتى لو حدث exception)
        try {
            if (reader != null) {
                reader.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

**try-with-resources (Java 7+):**
```java
// أسهل وأنظف - يغلق الـ resources تلقائياً
public void tryWithResources() {
    try (FileReader reader = new FileReader("file.txt");
         BufferedReader br = new BufferedReader(reader)) {

        String line = br.readLine();
        System.out.println(line);

    } catch (IOException e) {
        e.printStackTrace();
    }
    // reader و br يتم إغلاقهم تلقائياً
}

// Multiple catch blocks
public void multipleCatch() {
    try {
        // code that may throw exceptions

    } catch (FileNotFoundException e) {
        // معالجة محددة
        System.out.println("File not found");

    } catch (IOException e) {
        // معالجة أعم
        System.out.println("IO Error");

    } catch (Exception e) {
        // معالجة عامة (يجب أن تكون الأخيرة)
        System.out.println("General error");
    }
}

// Multi-catch (Java 7+)
public void multiCatch() {
    try {
        // code
    } catch (IOException | SQLException e) {
        // نفس المعالجة للنوعين
        e.printStackTrace();
    }
}
```

**Custom Exception:**
```java
// إنشاء exception خاص
public class InsufficientBalanceException extends Exception {
    public InsufficientBalanceException(String message) {
        super(message);
    }
}

// الاستخدام
public class BankAccount {
    private double balance;

    public void withdraw(double amount) throws InsufficientBalanceException {
        if (amount > balance) {
            throw new InsufficientBalanceException(
                "Balance: " + balance + ", Requested: " + amount
            );
        }
        balance -= amount;
    }
}
```

---

## Java 8+ Features

### 13. اشرح Lambda Expressions

**الإجابة:**

Lambda expressions تتيح لك كتابة كود أقصر لـ functional interfaces.

**قبل Lambda (Java 7):**
```java
// Anonymous class
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running");
    }
};

// Comparator
List<String> names = Arrays.asList("John", "Alice", "Bob");
Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return s1.compareTo(s2);
    }
});
```

**بعد Lambda (Java 8+):**
```java
// Runnable
Runnable runnable = () -> System.out.println("Running");

// Comparator
List<String> names = Arrays.asList("John", "Alice", "Bob");
Collections.sort(names, (s1, s2) -> s1.compareTo(s2));
// أو أبسط:
names.sort(String::compareTo);
```

**أمثلة متنوعة:**
```java
// بدون parameters
Runnable r = () -> System.out.println("Hello");

// parameter واحد (الأقواس اختيارية)
Consumer<String> print = s -> System.out.println(s);
Consumer<String> print2 = (s) -> System.out.println(s);

// عدة parameters
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;

// مع block
BiFunction<Integer, Integer, Integer> multiply = (a, b) -> {
    int result = a * b;
    System.out.println("Result: " + result);
    return result;
};

// Functional Interfaces
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}

Calculator addition = (a, b) -> a + b;
Calculator subtraction = (a, b) -> a - b;

System.out.println(addition.calculate(5, 3));    // 8
System.out.println(subtraction.calculate(5, 3)); // 2
```

---

### 14. اشرح Stream API

**الإجابة:**

Stream API تتيح معالجة Collections بطريقة declarative وفعالة.

**العمليات الأساسية:**

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Filter - تصفية العناصر
List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
// [2, 4, 6, 8, 10]

// Map - تحويل العناصر
List<Integer> squares = numbers.stream()
    .map(n -> n * n)
    .collect(Collectors.toList());
// [1, 4, 9, 16, 25, ...]

// Reduce - دمج العناصر
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);
// أو: .reduce(0, Integer::sum);
// 55

// Sorted - ترتيب
List<Integer> sorted = numbers.stream()
    .sorted((a, b) -> b - a) // ترتيب عكسي
    .collect(Collectors.toList());

// Distinct - إزالة المكرر
List<Integer> unique = Arrays.asList(1, 2, 2, 3, 3, 4).stream()
    .distinct()
    .collect(Collectors.toList());
// [1, 2, 3, 4]

// Limit & Skip
List<Integer> limited = numbers.stream()
    .skip(2)   // تخطي أول 2
    .limit(3)  // خذ 3 فقط
    .collect(Collectors.toList());
// [3, 4, 5]
```

**أمثلة عملية:**

```java
class Employee {
    String name;
    int age;
    double salary;
    String department;

    // constructor, getters, setters
}

List<Employee> employees = Arrays.asList(
    new Employee("John", 25, 50000, "IT"),
    new Employee("Alice", 30, 60000, "HR"),
    new Employee("Bob", 35, 70000, "IT"),
    new Employee("Charlie", 28, 55000, "Finance")
);

// 1. إيجاد متوسط رواتب قسم IT
double avgITSalary = employees.stream()
    .filter(e -> e.getDepartment().equals("IT"))
    .mapToDouble(Employee::getSalary)
    .average()
    .orElse(0.0);

// 2. جمع الأسماء في String
String names = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.joining(", "));
// "John, Alice, Bob, Charlie"

// 3. تجميع حسب القسم
Map<String, List<Employee>> byDepartment = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// 4. إيجاد أعلى راتب
Optional<Employee> highestPaid = employees.stream()
    .max(Comparator.comparing(Employee::getSalary));

// 5. هل كل الموظفين فوق 20 سنة؟
boolean allAdults = employees.stream()
    .allMatch(e -> e.getAge() > 20);

// 6. هل يوجد موظف في Finance؟
boolean hasFinance = employees.stream()
    .anyMatch(e -> e.getDepartment().equals("Finance"));

// 7. حساب إحصائيات
IntSummaryStatistics ageStats = employees.stream()
    .mapToInt(Employee::getAge)
    .summaryStatistics();
System.out.println("Average age: " + ageStats.getAverage());
System.out.println("Max age: " + ageStats.getMax());
```

**Intermediate vs Terminal Operations:**

```java
// Intermediate (lazy - لا تنفذ حتى terminal operation)
// filter, map, sorted, distinct, limit, skip

// Terminal (eager - تنفذ الـ stream)
// collect, forEach, count, reduce, anyMatch, allMatch, findFirst
```

---

### 15. ما هو Optional؟ ولماذا نستخدمه؟

**الإجابة:**

Optional هو container قد يحتوي على قيمة أو لا (null-safe).

**المشكلة التي يحلها:**
```java
// الطريقة القديمة - معرض لـ NullPointerException
public String getUserCity(User user) {
    if (user != null) {
        Address address = user.getAddress();
        if (address != null) {
            City city = address.getCity();
            if (city != null) {
                return city.getName();
            }
        }
    }
    return "Unknown";
}
```

**باستخدام Optional:**
```java
// أنظف وأكثر أماناً
public Optional<String> getUserCity(User user) {
    return Optional.ofNullable(user)
        .map(User::getAddress)
        .map(Address::getCity)
        .map(City::getName);
}

// الاستخدام
String city = getUserCity(user).orElse("Unknown");
```

**العمليات الأساسية:**

```java
// 1. إنشاء Optional
Optional<String> empty = Optional.empty();
Optional<String> nonEmpty = Optional.of("Hello"); // يرمي exception إذا null
Optional<String> nullable = Optional.ofNullable(null); // آمن مع null

// 2. فحص وجود قيمة
if (optional.isPresent()) {
    String value = optional.get();
}

// 3. ifPresent - تنفيذ action إذا موجود
optional.ifPresent(value -> System.out.println(value));

// 4. orElse - قيمة افتراضية
String result = optional.orElse("default");

// 5. orElseGet - قيمة افتراضية (lazy)
String result2 = optional.orElseGet(() -> computeDefault());

// 6. orElseThrow - رمي exception
String result3 = optional.orElseThrow(() -> new IllegalArgumentException());

// 7. map - تحويل القيمة
Optional<Integer> length = optional.map(String::length);

// 8. flatMap - للـ nested Optionals
Optional<String> upper = optional.flatMap(s -> Optional.of(s.toUpperCase()));

// 9. filter - تصفية
Optional<String> filtered = optional.filter(s -> s.length() > 5);
```

**أمثلة عملية:**

```java
// مثال: البحث عن user
public Optional<User> findUserById(Long id) {
    // بدلاً من return null
    return Optional.ofNullable(database.get(id));
}

// الاستخدام
findUserById(1L)
    .map(User::getEmail)
    .ifPresent(email -> sendEmail(email));

// أو
String email = findUserById(1L)
    .map(User::getEmail)
    .orElse("no-reply@example.com");

// مثال: سلسلة من العمليات
Optional<String> result = findUserById(1L)
    .filter(user -> user.getAge() > 18)
    .map(User::getName)
    .map(String::toUpperCase);
```

**متى لا تستخدم Optional:**
- في Collections (استخدم empty collection)
- كـ class fields
- كـ method parameters
- في serialization

---

## Memory Management

### 16. اشرح Garbage Collection في Java

**الإجابة:**

Garbage Collection (GC) هي عملية أوتوماتيكية لتنظيف الذاكرة من الكائنات غير المستخدمة.

**أقسام Memory في JVM:**

```
Heap Memory:
├── Young Generation
│   ├── Eden Space
│   └── Survivor Spaces (S0, S1)
└── Old Generation (Tenured)

Non-Heap Memory:
├── Method Area (Metaspace in Java 8+)
└── Stack Memory (لكل thread)
```

**كيف يعمل GC:**

```java
// 1. Object creation
public void createObjects() {
    String s1 = new String("Hello");  // يُنشأ في Eden
    String s2 = new String("World");

    // s1 و s2 eligible for GC عند نهاية الـ method
}

// 2. Object reachability
String globalString; // reference في heap

public void methodScope() {
    String local = "temp"; // stack reference
    globalString = "permanent"; // heap reference

    // local eligible for GC عند نهاية الـ method
    // globalString يبقى حتى يصبح null
}

// 3. Breaking references
public void breakReference() {
    MyObject obj1 = new MyObject();
    MyObject obj2 = new MyObject();

    obj1.setNext(obj2);
    obj2.setNext(obj1); // circular reference

    obj1 = null;
    obj2 = null;
    // الآن كلاهما eligible for GC
    // (Java GC يكتشف circular references)
}
```

**أنواع GC:**

```java
// 1. Minor GC - ينظف Young Generation
// سريع، يحدث كثيراً

// 2. Major GC - ينظف Old Generation
// بطيء، يحدث نادراً

// 3. Full GC - ينظف كل الـ heap
// الأبطأ

// استدعاء GC يدوياً (غير مضمون)
System.gc(); // suggestion فقط
Runtime.getRuntime().gc();
```

**finalize() Method:**

```java
// يُنفذ قبل حذف الكائن (deprecated في Java 9+)
class MyClass {
    @Override
    protected void finalize() throws Throwable {
        try {
            // cleanup resources
            System.out.println("Finalizing object");
        } finally {
            super.finalize();
        }
    }
}

// البديل الأفضل: try-with-resources أو AutoCloseable
class MyResource implements AutoCloseable {
    @Override
    public void close() {
        // cleanup
        System.out.println("Closing resource");
    }
}

try (MyResource resource = new MyResource()) {
    // use resource
} // close() يُستدعى تلقائياً
```

---

### 17. ما الفرق بين Stack و Heap Memory؟

**الإجابة:**

| الخاصية | Stack | Heap |
|---------|-------|------|
| التخزين | Local variables, method calls | Objects, instance variables |
| الحجم | صغير (MB) | كبير (GB) |
| السرعة | سريع جداً | أبطأ |
| الـ Lifecycle | مع الـ method | حتى GC |
| Thread | لكل thread واحد منفصل | مشترك بين كل الـ threads |
| Exception | StackOverflowError | OutOfMemoryError |

**مثال توضيحي:**

```java
public class MemoryExample {

    // Class variable - في Heap
    static int classVar = 10;

    // Instance variable - في Heap
    int instanceVar = 20;

    public void method(int param) { // param في Stack
        // Local variables - في Stack
        int localVar = 30;
        String localString = "test"; // reference في Stack

        // الـ Object نفسه - في Heap
        MyObject obj = new MyObject();

        /*
        Stack:
        ├── param = value
        ├── localVar = 30
        ├── localString = reference
        └── obj = reference

        Heap:
        ├── "test" String object
        ├── MyObject instance
        │   └── instanceVar = 20
        └── classVar = 10
        */
    }

    public void recursiveMethod(int n) {
        if (n == 0) return;

        int local = n; // كل استدعاء يضيف لـ stack
        recursiveMethod(n - 1);

        // StackOverflowError إذا n كبير جداً
    }
}

// مثال عملي
public class StackHeapDemo {
    public static void main(String[] args) {
        int x = 10;           // Stack
        Integer y = new Integer(20); // y في Stack, object في Heap

        Person p1 = new Person("John"); // p1 في Stack, Person في Heap
        Person p2 = p1;      // p2 في Stack, نفس الـ object

        p2.setName("Alice");
        System.out.println(p1.getName()); // "Alice"
        // لأن p1 و p2 يشيران لنفس الـ object في Heap
    }
}

class Person {
    private String name; // في Heap مع الـ object

    public Person(String name) {
        this.name = name;
    }

    // getters/setters
}
```

**Memory Errors:**

```java
// 1. StackOverflowError
public void infiniteRecursion() {
    infiniteRecursion(); // لا يتوقف
}

// 2. OutOfMemoryError: Heap space
public void memoryLeak() {
    List<byte[]> list = new ArrayList<>();
    while (true) {
        list.add(new byte[1024 * 1024]); // 1 MB
    }
}

// 3. OutOfMemoryError: Metaspace
// يحدث عند تحميل classes كثيرة جداً
```

---

## أسئلة إضافية متقدمة

### 18. اشرح Functional Interfaces

**الإجابة:**

Functional Interface هو interface يحتوي على abstract method واحد فقط.

**Built-in Functional Interfaces:**

```java
// 1. Predicate<T> - يأخذ T ويرجع boolean
Predicate<Integer> isEven = n -> n % 2 == 0;
System.out.println(isEven.test(4)); // true

Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();

// 2. Function<T, R> - يأخذ T ويرجع R
Function<String, Integer> length = String::length;
System.out.println(length.apply("Hello")); // 5

Function<Integer, Integer> square = n -> n * n;
Function<Integer, Integer> addOne = n -> n + 1;
Function<Integer, Integer> combined = square.andThen(addOne);
System.out.println(combined.apply(3)); // 10 (3^2 + 1)

// 3. Consumer<T> - يأخذ T ولا يرجع شيء
Consumer<String> print = System.out::println;
print.accept("Hello");

Consumer<String> c1 = s -> System.out.print(s);
Consumer<String> c2 = s -> System.out.println("!");
Consumer<String> combined2 = c1.andThen(c2);
combined2.accept("Hello"); // Hello!

// 4. Supplier<T> - لا يأخذ شيء ويرجع T
Supplier<Double> random = Math::random;
System.out.println(random.get());

Supplier<String> supplier = () -> "Hello World";

// 5. BiFunction<T, U, R> - يأخذ T و U ويرجع R
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
System.out.println(add.apply(5, 3)); // 8

// 6. BiConsumer<T, U> - يأخذ T و U
BiConsumer<String, Integer> printWithLength = (s, n) -> {
    System.out.println(s + " has length " + n);
};
printWithLength.accept("Hello", 5);

// 7. BiPredicate<T, U> - يأخذ T و U ويرجع boolean
BiPredicate<String, Integer> lengthEquals = (s, n) -> s.length() == n;
System.out.println(lengthEquals.test("Hello", 5)); // true
```

**Custom Functional Interface:**

```java
@FunctionalInterface
interface MathOperation {
    int operate(int a, int b);

    // يمكن أن يحتوي على:
    // 1. default methods
    default int addTen(int x) {
        return x + 10;
    }

    // 2. static methods
    static int multiply(int a, int b) {
        return a * b;
    }
}

// الاستخدام
MathOperation addition = (a, b) -> a + b;
MathOperation subtraction = (a, b) -> a - b;

System.out.println(addition.operate(5, 3));    // 8
System.out.println(subtraction.operate(5, 3)); // 2
System.out.println(addition.addTen(5));        // 15
System.out.println(MathOperation.multiply(5, 3)); // 15
```

---

### 19. ما هو Method Reference؟

**الإجابة:**

Method Reference هي اختصار لـ lambda expression التي تستدعي method موجود.

**أنواع Method References:**

```java
// 1. Static Method Reference
// ClassName::staticMethod

List<String> numbers = Arrays.asList("1", "2", "3");

// Lambda
numbers.forEach(s -> System.out.println(s));

// Method reference
numbers.forEach(System.out::println);

// مثال آخر
Function<String, Integer> parser1 = s -> Integer.parseInt(s);
Function<String, Integer> parser2 = Integer::parseInt;

// 2. Instance Method Reference (object specific)
// object::instanceMethod

String str = "Hello";
Supplier<Integer> length1 = () -> str.length();
Supplier<Integer> length2 = str::length;

// 3. Instance Method Reference (arbitrary object)
// ClassName::instanceMethod

List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// Lambda
names.sort((s1, s2) -> s1.compareToIgnoreCase(s2));

// Method reference
names.sort(String::compareToIgnoreCase);

// 4. Constructor Reference
// ClassName::new

// Lambda
Supplier<List<String>> list1 = () -> new ArrayList<>();

// Constructor reference
Supplier<List<String>> list2 = ArrayList::new;

// مع parameters
Function<String, Integer> converter1 = s -> new Integer(s);
Function<String, Integer> converter2 = Integer::new;

// Array constructor
Function<Integer, String[]> arrayCreator = String[]::new;
String[] array = arrayCreator.apply(10); // array بطول 10
```

**أمثلة عملية:**

```java
class Person {
    private String name;
    private int age;

    public Person() {}

    public Person(String name) {
        this.name = name;
    }

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public static int compareByAge(Person p1, Person p2) {
        return Integer.compare(p1.age, p2.age);
    }

    public int compareByName(Person other) {
        return this.name.compareTo(other.name);
    }

    // getters/setters
}

// الاستخدام
List<Person> people = Arrays.asList(
    new Person("John", 25),
    new Person("Alice", 30),
    new Person("Bob", 20)
);

// 1. Static method reference
people.sort(Person::compareByAge);

// 2. Instance method reference
Person first = people.get(0);
Comparator<Person> comparator = first::compareByName;

// 3. Constructor reference
Supplier<Person> personSupplier = Person::new;
Person p = personSupplier.get();

Function<String, Person> personFactory = Person::new;
Person p2 = personFactory.apply("John");

BiFunction<String, Integer, Person> personFactory2 = Person::new;
Person p3 = personFactory2.apply("Alice", 30);

// 4. في Stream API
List<String> names = Arrays.asList("john", "alice", "bob");

List<String> upperNames = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());

names.stream()
    .filter(String::isEmpty)
    .forEach(System.out::println);
```

---

### 20. اشرح Immutability في Java

**الإجابة:**

Immutable object هو كائن لا يمكن تغيير حالته بعد إنشائه.

**فوائد Immutability:**
- Thread-safe بدون synchronization
- آمن في HashMap keys
- سهل الفهم والصيانة
- لا يحتاج defensive copying

**أمثلة Built-in Immutable Classes:**
```java
// String
String s = "Hello";
s.concat(" World"); // يُنشئ String جديد
System.out.println(s); // "Hello" (لم يتغير)

// Wrapper Classes
Integer i = 10;
i = i + 5; // ينشئ Integer جديد

// Java 8+ Classes
LocalDate date = LocalDate.now();
LocalDate tomorrow = date.plusDays(1); // جديد
// date لم يتغير
```

**إنشاء Immutable Class:**

```java
// قواعد Immutable Class:
// 1. الـ class نفسه final
// 2. كل الـ fields private final
// 3. لا setters
// 4. defensive copy للـ mutable objects
// 5. لا تسمح بـ subclasses تغير البيانات

public final class ImmutablePerson {
    private final String name;
    private final int age;
    private final List<String> hobbies;
    private final Address address; // mutable object

    public ImmutablePerson(String name, int age,
                          List<String> hobbies, Address address) {
        this.name = name;
        this.age = age;

        // Defensive copy للـ list
        this.hobbies = new ArrayList<>(hobbies);

        // Defensive copy للـ mutable object
        this.address = new Address(address);
    }

    // Getters only (no setters)
    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    // Return defensive copy
    public List<String> getHobbies() {
        return new ArrayList<>(hobbies);
    }

    public Address getAddress() {
        return new Address(address);
    }

    // للتعديل: إنشاء instance جديد
    public ImmutablePerson withName(String newName) {
        return new ImmutablePerson(newName, age, hobbies, address);
    }

    public ImmutablePerson withAge(int newAge) {
        return new ImmutablePerson(name, newAge, hobbies, address);
    }
}

// الاستخدام
ImmutablePerson person = new ImmutablePerson(
    "John", 25,
    Arrays.asList("Reading", "Gaming"),
    new Address("123 Main St")
);

// لا يمكن تغييره مباشرة
// person.setName("Alice"); // لا يوجد setter

// للتعديل: إنشاء instance جديد
ImmutablePerson modified = person.withName("Alice");
// person لم يتغير
```

**مثال: Immutable Date Range**

```java
public final class DateRange {
    private final LocalDate start;
    private final LocalDate end;

    public DateRange(LocalDate start, LocalDate end) {
        if (start.isAfter(end)) {
            throw new IllegalArgumentException("Start must be before end");
        }
        // LocalDate already immutable
        this.start = start;
        this.end = end;
    }

    public LocalDate getStart() {
        return start; // آمن (LocalDate immutable)
    }

    public LocalDate getEnd() {
        return end;
    }

    public boolean contains(LocalDate date) {
        return !date.isBefore(start) && !date.isAfter(end);
    }

    public DateRange extendBy(int days) {
        return new DateRange(start, end.plusDays(days));
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof DateRange)) return false;
        DateRange that = (DateRange) o;
        return start.equals(that.start) && end.equals(that.end);
    }

    @Override
    public int hashCode() {
        return Objects.hash(start, end);
    }
}
```

---

## نصائح للمقابلات

### استراتيجية الإجابة:
1. **افهم السؤال جيداً** - اسأل إذا كان هناك غموض
2. **ابدأ بالأساسيات** - ثم انتقل للتفاصيل
3. **أعط أمثلة عملية** - كود يوضح النقطة
4. **اذكر Use Cases** - متى تستخدم كل approach
5. **تكلم عن Trade-offs** - المزايا والعيوب

### أسئلة شائعة إضافية:
- SOLID Principles
- Design Patterns (Singleton, Factory, Observer, etc.)
- Serialization vs Deserialization
- Reflection API
- Annotations
- Generics
- Concurrency Utilities (ExecutorService, CountDownLatch, etc.)

---

**حظاً موفقاً في المقابلة! 🚀**

**ملاحظة:** هذا الدليل يغطي المواضيع الأساسية. تأكد من فهم كل مفهوم عملياً بكتابة الكود وتجربته.