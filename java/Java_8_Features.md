# دليل شامل لمميزات Java 8

## المحتويات
1. [Lambda Expressions](#lambda-expressions)
2. [Functional Interfaces](#functional-interfaces)
3. [Stream API](#stream-api)
4. [Method References](#method-references)
5. [Optional Class](#optional-class)
6. [Default Methods](#default-methods)
7. [Date/Time API](#datetime-api)
8. [Nashorn JavaScript Engine](#nashorn-javascript-engine)

---

## Lambda Expressions

### ما هي Lambda Expressions؟
Lambda Expressions هي وظائف مجهولة (Anonymous Functions) تسمح بكتابة كود أقصر وأوضح.

### البنية الأساسية
```java
(parameters) -> expression
(parameters) -> { statements; }
```

### أمثلة عملية

#### مثال 1: بدون Lambda
```java
// الطريقة التقليدية
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello World");
    }
};
```

#### مثال 2: مع Lambda
```java
// باستخدام Lambda
Runnable runnable = () -> System.out.println("Hello World");
```

#### مثال 3: Lambda مع Parameters
```java
// الطريقة التقليدية
Comparator<String> comparator = new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return s1.compareTo(s2);
    }
};

// باستخدام Lambda
Comparator<String> comparator = (s1, s2) -> s1.compareTo(s2);
```

#### مثال 4: Lambda مع أكثر من Statement
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

numbers.forEach(n -> {
    int result = n * n;
    System.out.println("Square of " + n + " = " + result);
});
```

---

## Functional Interfaces

### ما هي Functional Interface؟
Interface يحتوي على method واحدة فقط (Single Abstract Method - SAM).

### Built-in Functional Interfaces

#### 1. Predicate&lt;T&gt;
```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```

**مثال:**
```java
Predicate<Integer> isEven = n -> n % 2 == 0;
System.out.println(isEven.test(4)); // true
System.out.println(isEven.test(5)); // false

// استخدام مع List
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
numbers.stream()
       .filter(n -> n % 2 == 0)
       .forEach(System.out::println); // 2, 4, 6
```

#### 2. Function&lt;T, R&gt;
```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

**مثال:**
```java
Function<String, Integer> stringLength = s -> s.length();
System.out.println(stringLength.apply("Hello")); // 5

// تحويل String إلى Integer
Function<String, Integer> converter = Integer::valueOf;
Integer num = converter.apply("123"); // 123
```

#### 3. Consumer&lt;T&gt;
```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

**مثال:**
```java
Consumer<String> printer = s -> System.out.println(s);
printer.accept("Hello World");

// استخدام مع List
List<String> names = Arrays.asList("Ahmed", "Mohamed", "Ali");
names.forEach(name -> System.out.println("Hello " + name));
```

#### 4. Supplier&lt;T&gt;
```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

**مثال:**
```java
Supplier<Double> randomSupplier = () -> Math.random();
System.out.println(randomSupplier.get());

// إنشاء كائن جديد
Supplier<List<String>> listSupplier = ArrayList::new;
List<String> list = listSupplier.get();
```

#### 5. BiFunction&lt;T, U, R&gt;
```java
@FunctionalInterface
public interface BiFunction<T, U, R> {
    R apply(T t, U u);
}
```

**مثال:**
```java
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
System.out.println(add.apply(5, 3)); // 8

BiFunction<String, String, String> concat = (s1, s2) -> s1 + " " + s2;
System.out.println(concat.apply("Hello", "World")); // Hello World
```

### إنشاء Functional Interface مخصص
```java
@FunctionalInterface
public interface Calculator {
    int calculate(int a, int b);
}

// الاستخدام
Calculator addition = (a, b) -> a + b;
Calculator subtraction = (a, b) -> a - b;
Calculator multiplication = (a, b) -> a * b;
Calculator division = (a, b) -> a / b;

System.out.println(addition.calculate(10, 5));       // 15
System.out.println(subtraction.calculate(10, 5));    // 5
System.out.println(multiplication.calculate(10, 5)); // 50
System.out.println(division.calculate(10, 5));       // 2
```

---

## Stream API

### ما هي Stream API؟
Stream API تسمح بمعالجة مجموعات البيانات بطريقة وظيفية (Functional Programming).

### إنشاء Streams

```java
// من Collection
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream1 = list.stream();

// من Array
String[] array = {"a", "b", "c"};
Stream<String> stream2 = Arrays.stream(array);

// باستخدام Stream.of()
Stream<String> stream3 = Stream.of("a", "b", "c");

// Stream لانهائي
Stream<Integer> infiniteStream = Stream.iterate(0, n -> n + 2);
```

### Intermediate Operations

#### 1. filter()
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// الأرقام الزوجية فقط
List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
System.out.println(evenNumbers); // [2, 4, 6, 8, 10]
```

#### 2. map()
```java
List<String> names = Arrays.asList("ahmed", "mohamed", "ali");

// تحويل إلى uppercase
List<String> upperNames = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
System.out.println(upperNames); // [AHMED, MOHAMED, ALI]

// الحصول على أطوال الأسماء
List<Integer> lengths = names.stream()
    .map(String::length)
    .collect(Collectors.toList());
System.out.println(lengths); // [5, 7, 3]
```

#### 3. flatMap()
```java
List<List<Integer>> listOfLists = Arrays.asList(
    Arrays.asList(1, 2, 3),
    Arrays.asList(4, 5, 6),
    Arrays.asList(7, 8, 9)
);

// تحويل List of Lists إلى List واحد
List<Integer> flatList = listOfLists.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());
System.out.println(flatList); // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

#### 4. distinct()
```java
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 3, 4, 5, 5);

List<Integer> uniqueNumbers = numbers.stream()
    .distinct()
    .collect(Collectors.toList());
System.out.println(uniqueNumbers); // [1, 2, 3, 4, 5]
```

#### 5. sorted()
```java
List<String> names = Arrays.asList("Mohamed", "Ahmed", "Ali", "Ziad");

// ترتيب تصاعدي
List<String> sorted = names.stream()
    .sorted()
    .collect(Collectors.toList());
System.out.println(sorted); // [Ahmed, Ali, Mohamed, Ziad]

// ترتيب تنازلي
List<String> reverseSorted = names.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());
System.out.println(reverseSorted); // [Ziad, Mohamed, Ali, Ahmed]
```

#### 6. limit() & skip()
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// أول 5 عناصر
List<Integer> first5 = numbers.stream()
    .limit(5)
    .collect(Collectors.toList());
System.out.println(first5); // [1, 2, 3, 4, 5]

// تخطي أول 5 عناصر
List<Integer> skip5 = numbers.stream()
    .skip(5)
    .collect(Collectors.toList());
System.out.println(skip5); // [6, 7, 8, 9, 10]
```

### Terminal Operations

#### 1. forEach()
```java
List<String> names = Arrays.asList("Ahmed", "Mohamed", "Ali");
names.stream().forEach(System.out::println);
```

#### 2. collect()
```java
List<String> names = Arrays.asList("Ahmed", "Mohamed", "Ali");

// إلى List
List<String> list = names.stream().collect(Collectors.toList());

// إلى Set
Set<String> set = names.stream().collect(Collectors.toSet());

// إلى Map
Map<String, Integer> map = names.stream()
    .collect(Collectors.toMap(
        name -> name,
        name -> name.length()
    ));
System.out.println(map); // {Ahmed=5, Mohamed=7, Ali=3}
```

#### 3. reduce()
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// جمع الأرقام
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);
System.out.println(sum); // 15

// ضرب الأرقام
int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);
System.out.println(product); // 120

// إيجاد الأكبر
Optional<Integer> max = numbers.stream()
    .reduce((a, b) -> a > b ? a : b);
System.out.println(max.get()); // 5
```

#### 4. count()
```java
List<String> names = Arrays.asList("Ahmed", "Mohamed", "Ali");
long count = names.stream().count();
System.out.println(count); // 3
```

#### 5. anyMatch(), allMatch(), noneMatch()
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// هل يوجد رقم زوجي؟
boolean hasEven = numbers.stream().anyMatch(n -> n % 2 == 0);
System.out.println(hasEven); // true

// هل كل الأرقام زوجية؟
boolean allEven = numbers.stream().allMatch(n -> n % 2 == 0);
System.out.println(allEven); // false

// هل لا يوجد رقم أكبر من 10؟
boolean noneGreaterThan10 = numbers.stream().noneMatch(n -> n > 10);
System.out.println(noneGreaterThan10); // true
```

#### 6. findFirst() & findAny()
```java
List<String> names = Arrays.asList("Ahmed", "Mohamed", "Ali");

Optional<String> first = names.stream().findFirst();
System.out.println(first.get()); // Ahmed

Optional<String> any = names.stream().findAny();
System.out.println(any.get()); // Ahmed (في sequential stream)
```

#### 7. min() & max()
```java
List<Integer> numbers = Arrays.asList(5, 2, 8, 1, 9, 3);

Optional<Integer> min = numbers.stream().min(Integer::compareTo);
Optional<Integer> max = numbers.stream().max(Integer::compareTo);

System.out.println(min.get()); // 1
System.out.println(max.get()); // 9
```

### أمثلة متقدمة على Stream

#### مثال 1: معالجة قائمة من الموظفين
```java
class Employee {
    String name;
    int age;
    double salary;
    String department;

    public Employee(String name, int age, double salary, String department) {
        this.name = name;
        this.age = age;
        this.salary = salary;
        this.department = department;
    }

    // Getters
    public String getName() { return name; }
    public int getAge() { return age; }
    public double getSalary() { return salary; }
    public String getDepartment() { return department; }
}

List<Employee> employees = Arrays.asList(
    new Employee("Ahmed", 30, 5000, "IT"),
    new Employee("Mohamed", 25, 4000, "HR"),
    new Employee("Ali", 35, 6000, "IT"),
    new Employee("Sara", 28, 4500, "Finance"),
    new Employee("Fatma", 32, 5500, "IT")
);

// 1. الموظفين في قسم IT فقط
List<Employee> itEmployees = employees.stream()
    .filter(e -> e.getDepartment().equals("IT"))
    .collect(Collectors.toList());

// 2. متوسط رواتب الموظفين
double avgSalary = employees.stream()
    .mapToDouble(Employee::getSalary)
    .average()
    .orElse(0.0);
System.out.println("Average Salary: " + avgSalary);

// 3. الموظفين مرتبين حسب الراتب تنازلياً
List<Employee> sortedBySalary = employees.stream()
    .sorted(Comparator.comparing(Employee::getSalary).reversed())
    .collect(Collectors.toList());

// 4. أعلى راتب
Optional<Employee> highestPaid = employees.stream()
    .max(Comparator.comparing(Employee::getSalary));

// 5. مجموع رواتب قسم IT
double itTotalSalary = employees.stream()
    .filter(e -> e.getDepartment().equals("IT"))
    .mapToDouble(Employee::getSalary)
    .sum();

// 6. تجميع الموظفين حسب القسم
Map<String, List<Employee>> byDepartment = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// 7. عدد الموظفين في كل قسم
Map<String, Long> countByDepartment = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.counting()
    ));

// 8. متوسط الراتب في كل قسم
Map<String, Double> avgSalaryByDepartment = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));
```

#### مثال 2: معالجة النصوص
```java
String text = "Java 8 Stream API is powerful and flexible";

// عدد الكلمات
long wordCount = Arrays.stream(text.split(" ")).count();

// أطوال الكلمات
List<Integer> wordLengths = Arrays.stream(text.split(" "))
    .map(String::length)
    .collect(Collectors.toList());

// الكلمات الفريدة (بدون تكرار)
List<String> uniqueWords = Arrays.stream(text.split(" "))
    .distinct()
    .collect(Collectors.toList());

// الكلمات التي تبدأ بحرف كبير
List<String> capitalWords = Arrays.stream(text.split(" "))
    .filter(word -> Character.isUpperCase(word.charAt(0)))
    .collect(Collectors.toList());
```

### Parallel Streams

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Sequential Stream
long count1 = numbers.stream()
    .filter(n -> n % 2 == 0)
    .count();

// Parallel Stream (أسرع مع البيانات الكبيرة)
long count2 = numbers.parallelStream()
    .filter(n -> n % 2 == 0)
    .count();

// مثال على الفرق في الأداء
List<Integer> largeList = IntStream.rangeClosed(1, 1000000)
    .boxed()
    .collect(Collectors.toList());

// Sequential
long start = System.currentTimeMillis();
long sum1 = largeList.stream()
    .mapToInt(Integer::intValue)
    .sum();
long end = System.currentTimeMillis();
System.out.println("Sequential: " + (end - start) + "ms");

// Parallel
start = System.currentTimeMillis();
long sum2 = largeList.parallelStream()
    .mapToInt(Integer::intValue)
    .sum();
end = System.currentTimeMillis();
System.out.println("Parallel: " + (end - start) + "ms");
```

---

## Method References

### ما هي Method References؟
طريقة مختصرة لكتابة Lambda Expressions عندما تستدعي method موجودة.

### أنواع Method References

#### 1. Static Method Reference
```java
// Lambda
Function<String, Integer> parser1 = s -> Integer.parseInt(s);

// Method Reference
Function<String, Integer> parser2 = Integer::parseInt;

System.out.println(parser2.apply("123")); // 123
```

#### 2. Instance Method Reference (على كائن معين)
```java
String str = "Hello World";

// Lambda
Supplier<String> supplier1 = () -> str.toUpperCase();

// Method Reference
Supplier<String> supplier2 = str::toUpperCase;

System.out.println(supplier2.get()); // HELLO WORLD
```

#### 3. Instance Method Reference (على كائن عشوائي)
```java
List<String> names = Arrays.asList("ahmed", "mohamed", "ali");

// Lambda
names.stream()
     .map(s -> s.toUpperCase())
     .forEach(System.out::println);

// Method Reference
names.stream()
     .map(String::toUpperCase)
     .forEach(System.out::println);
```

#### 4. Constructor Reference
```java
// Lambda
Supplier<List<String>> supplier1 = () -> new ArrayList<>();

// Constructor Reference
Supplier<List<String>> supplier2 = ArrayList::new;

List<String> list = supplier2.get();
```

### أمثلة متقدمة

```java
class Person {
    String name;
    int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() { return name; }
    public int getAge() { return age; }
}

List<Person> people = Arrays.asList(
    new Person("Ahmed", 30),
    new Person("Mohamed", 25),
    new Person("Ali", 35)
);

// استخدام Method Reference مع الترتيب
people.stream()
      .sorted(Comparator.comparing(Person::getAge))
      .map(Person::getName)
      .forEach(System.out::println);

// Constructor Reference
Function<String, Integer> converter = Integer::new;

// Array Constructor Reference
IntFunction<int[]> arrayCreator = int[]::new;
int[] array = arrayCreator.apply(10); // مصفوفة بحجم 10
```

---

## Optional Class

### ما هي Optional؟
Container class لتجنب NullPointerException.

### إنشاء Optional

```java
// Optional فارغ
Optional<String> empty = Optional.empty();

// Optional مع قيمة
Optional<String> optional1 = Optional.of("Hello");

// Optional مع قيمة محتملة null
String nullableString = null;
Optional<String> optional2 = Optional.ofNullable(nullableString);
```

### Methods الأساسية

#### 1. isPresent() & isEmpty()
```java
Optional<String> optional = Optional.of("Hello");

if (optional.isPresent()) {
    System.out.println("Value exists: " + optional.get());
}

// Java 11+
if (optional.isEmpty()) {
    System.out.println("No value");
}
```

#### 2. ifPresent()
```java
Optional<String> optional = Optional.of("Hello");

optional.ifPresent(value -> System.out.println(value));

// أو باستخدام Method Reference
optional.ifPresent(System.out::println);
```

#### 3. orElse() & orElseGet()
```java
Optional<String> optional = Optional.empty();

// orElse - قيمة افتراضية
String value1 = optional.orElse("Default Value");
System.out.println(value1); // Default Value

// orElseGet - استدعاء Supplier
String value2 = optional.orElseGet(() -> "Generated Default");
System.out.println(value2); // Generated Default

// الفرق: orElseGet أفضل في الأداء لأنه لا يُستدعى إلا عند الحاجة
Optional<String> opt = Optional.of("Exists");
String v1 = opt.orElse(expensiveOperation());        // يُستدعى دائماً
String v2 = opt.orElseGet(() -> expensiveOperation()); // لا يُستدعى إذا كانت القيمة موجودة
```

#### 4. orElseThrow()
```java
Optional<String> optional = Optional.empty();

// رمي exception إذا كانت فارغة
try {
    String value = optional.orElseThrow(() ->
        new IllegalArgumentException("Value not found"));
} catch (IllegalArgumentException e) {
    System.out.println(e.getMessage());
}

// Java 10+ - رمي NoSuchElementException
String value = optional.orElseThrow();
```

#### 5. map()
```java
Optional<String> optional = Optional.of("hello");

Optional<String> upper = optional.map(String::toUpperCase);
System.out.println(upper.get()); // HELLO

Optional<Integer> length = optional.map(String::length);
System.out.println(length.get()); // 5
```

#### 6. flatMap()
```java
class Address {
    String city;
    public Optional<String> getCity() {
        return Optional.ofNullable(city);
    }
}

class Person {
    Address address;
    public Optional<Address> getAddress() {
        return Optional.ofNullable(address);
    }
}

Person person = new Person();

// استخدام flatMap لتجنب Optional<Optional<String>>
Optional<String> city = Optional.of(person)
    .flatMap(Person::getAddress)
    .flatMap(Address::getCity);
```

#### 7. filter()
```java
Optional<String> optional = Optional.of("Hello World");

Optional<String> filtered = optional.filter(s -> s.length() > 5);
System.out.println(filtered.isPresent()); // true

Optional<String> filtered2 = optional.filter(s -> s.length() > 20);
System.out.println(filtered2.isPresent()); // false
```

### أمثلة عملية

#### مثال 1: البحث في قائمة
```java
class Product {
    String id;
    String name;
    double price;

    public Product(String id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }

    public String getId() { return id; }
    public String getName() { return name; }
    public double getPrice() { return price; }
}

List<Product> products = Arrays.asList(
    new Product("1", "Laptop", 1000),
    new Product("2", "Mouse", 20),
    new Product("3", "Keyboard", 50)
);

// البحث عن منتج
public Optional<Product> findProductById(String id) {
    return products.stream()
        .filter(p -> p.getId().equals(id))
        .findFirst();
}

// الاستخدام
Optional<Product> product = findProductById("2");

// الطريقة القديمة
if (product.isPresent()) {
    System.out.println(product.get().getName());
}

// الطريقة الأفضل
product.ifPresent(p -> System.out.println(p.getName()));

// مع قيمة افتراضية
String name = product.map(Product::getName).orElse("Product not found");

// مع exception
Product p = product.orElseThrow(() ->
    new ProductNotFoundException("Product with id " + id + " not found"));
```

#### مثال 2: معالجة البيانات الاختيارية
```java
class User {
    String name;
    String email;
    String phone;

    public Optional<String> getEmail() {
        return Optional.ofNullable(email);
    }

    public Optional<String> getPhone() {
        return Optional.ofNullable(phone);
    }
}

User user = new User();
user.name = "Ahmed";
user.email = "ahmed@example.com";
// phone is null

// الحصول على email أو phone
String contact = user.getEmail()
    .orElseGet(() -> user.getPhone().orElse("No contact available"));

// معالجة متسلسلة
user.getEmail()
    .filter(email -> email.contains("@"))
    .map(String::toUpperCase)
    .ifPresent(email -> System.out.println("Valid email: " + email));
```

---

## Default Methods

### ما هي Default Methods؟
Methods لها implementation في Interface (بدلاً من abstract methods فقط).

### لماذا نستخدمها؟
- إضافة methods جديدة لـ Interface دون كسر التوافق مع الكود القديم
- توفير implementation افتراضية يمكن override

### البنية الأساسية

```java
interface Vehicle {
    // Abstract method (كالمعتاد)
    void start();

    // Default method (جديد في Java 8)
    default void stop() {
        System.out.println("Vehicle stopped");
    }

    // Static method في Interface
    static void service() {
        System.out.println("Vehicle serviced");
    }
}

class Car implements Vehicle {
    @Override
    public void start() {
        System.out.println("Car started");
    }

    // يمكن override للـ default method (اختياري)
    @Override
    public void stop() {
        System.out.println("Car stopped with handbrake");
    }
}

class Bike implements Vehicle {
    @Override
    public void start() {
        System.out.println("Bike started");
    }

    // لا نحتاج override للـ default method
    // سيستخدم الـ implementation الافتراضية
}

// الاستخدام
Car car = new Car();
car.start();  // Car started
car.stop();   // Car stopped with handbrake

Bike bike = new Bike();
bike.start(); // Bike started
bike.stop();  // Vehicle stopped (default implementation)

Vehicle.service(); // Vehicle serviced (static method)
```

### أمثلة متقدمة

#### مثال 1: Interface مع عدة Default Methods
```java
interface Payment {
    void processPayment(double amount);

    default void printReceipt(double amount) {
        System.out.println("Receipt:");
        System.out.println("Amount: $" + amount);
        System.out.println("Thank you for your payment!");
    }

    default boolean validateAmount(double amount) {
        return amount > 0;
    }

    default void refund(double amount) {
        if (validateAmount(amount)) {
            System.out.println("Refunding $" + amount);
        } else {
            System.out.println("Invalid refund amount");
        }
    }
}

class CreditCardPayment implements Payment {
    @Override
    public void processPayment(double amount) {
        if (validateAmount(amount)) {
            System.out.println("Processing credit card payment: $" + amount);
            printReceipt(amount);
        }
    }

    // Override default method
    @Override
    public void printReceipt(double amount) {
        System.out.println("=== Credit Card Receipt ===");
        System.out.println("Amount: $" + amount);
        System.out.println("Card ending in: ****1234");
    }
}

class CashPayment implements Payment {
    @Override
    public void processPayment(double amount) {
        if (validateAmount(amount)) {
            System.out.println("Processing cash payment: $" + amount);
            printReceipt(amount); // استخدام default implementation
        }
    }
}
```

#### مثال 2: حل مشكلة Multiple Inheritance
```java
interface A {
    default void print() {
        System.out.println("A");
    }
}

interface B {
    default void print() {
        System.out.println("B");
    }
}

// خطأ: Diamond Problem
// يجب override للـ method
class C implements A, B {
    @Override
    public void print() {
        // يمكن اختيار implementation معينة
        A.super.print(); // استخدام implementation من A
        // أو
        B.super.print(); // استخدام implementation من B
        // أو
        System.out.println("C"); // implementation جديدة
    }
}
```

#### مثال 3: Interface مع Static و Default Methods
```java
interface MathOperations {
    // Abstract method
    double calculate(double a, double b);

    // Default method
    default void displayResult(double a, double b) {
        double result = calculate(a, b);
        System.out.println("Result: " + result);
    }

    // Static method
    static boolean isValid(double number) {
        return !Double.isNaN(number) && !Double.isInfinite(number);
    }

    // Static method
    static void printInfo() {
        System.out.println("Math Operations Interface");
    }
}

class Addition implements MathOperations {
    @Override
    public double calculate(double a, double b) {
        return a + b;
    }
}

class Division implements MathOperations {
    @Override
    public double calculate(double a, double b) {
        if (b == 0) {
            throw new ArithmeticException("Division by zero");
        }
        return a / b;
    }

    @Override
    public void displayResult(double a, double b) {
        if (MathOperations.isValid(a) && MathOperations.isValid(b) && b != 0) {
            System.out.println(a + " / " + b + " = " + calculate(a, b));
        }
    }
}
```

### فوائد Default Methods

```java
// قبل Java 8
interface List<E> {
    boolean add(E e);
    E get(int index);
    int size();
    // ... methods أخرى
}

// بعد Java 8 - إضافة methods جديدة بدون كسر الكود القديم
interface List<E> {
    boolean add(E e);
    E get(int index);
    int size();

    // Default methods جديدة
    default void sort(Comparator<? super E> c) {
        Collections.sort(this, c);
    }

    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, 0);
    }
}

// الكود القديم لا يزال يعمل
// والكود الجديد يمكنه استخدام الـ methods الجديدة
```

---

## Date/Time API

### ما هي Date/Time API الجديدة؟
API جديدة في package `java.time` لتحل محل `java.util.Date` و `java.util.Calendar`.

### المشاكل في النظام القديم
- `Date` و `Calendar` غير thread-safe
- API معقدة وغير واضحة
- الشهور تبدأ من 0 (يناير = 0)
- الخلط بين Date و Time

### Classes الأساسية

#### 1. LocalDate - للتاريخ فقط

```java
import java.time.LocalDate;
import java.time.Month;
import java.time.temporal.ChronoUnit;

// التاريخ الحالي
LocalDate today = LocalDate.now();
System.out.println(today); // 2025-10-04

// تاريخ محدد
LocalDate specificDate = LocalDate.of(2025, 1, 15);
LocalDate specificDate2 = LocalDate.of(2025, Month.JANUARY, 15);

// من String
LocalDate parsed = LocalDate.parse("2025-03-20");

// الحصول على معلومات
int year = today.getYear();        // 2025
Month month = today.getMonth();    // OCTOBER
int day = today.getDayOfMonth();   // 4
int dayOfYear = today.getDayOfYear(); // 277

// عمليات حسابية
LocalDate tomorrow = today.plusDays(1);
LocalDate nextWeek = today.plusWeeks(1);
LocalDate nextMonth = today.plusMonths(1);
LocalDate nextYear = today.plusYears(1);

LocalDate yesterday = today.minusDays(1);
LocalDate lastMonth = today.minusMonths(1);

// المقارنة
LocalDate date1 = LocalDate.of(2025, 1, 1);
LocalDate date2 = LocalDate.of(2025, 12, 31);

boolean isBefore = date1.isBefore(date2);  // true
boolean isAfter = date1.isAfter(date2);    // false
boolean isEqual = date1.isEqual(date2);    // false

// الفرق بين تاريخين
long daysBetween = ChronoUnit.DAYS.between(date1, date2);
long monthsBetween = ChronoUnit.MONTHS.between(date1, date2);
long yearsBetween = ChronoUnit.YEARS.between(date1, date2);
```

#### 2. LocalTime - للوقت فقط

```java
import java.time.LocalTime;

// الوقت الحالي
LocalTime now = LocalTime.now();
System.out.println(now); // 14:30:45.123

// وقت محدد
LocalTime time1 = LocalTime.of(14, 30);           // 14:30
LocalTime time2 = LocalTime.of(14, 30, 45);       // 14:30:45
LocalTime time3 = LocalTime.of(14, 30, 45, 123);  // 14:30:45.000000123

// من String
LocalTime parsed = LocalTime.parse("14:30:45");

// الحصول على معلومات
int hour = now.getHour();      // 14
int minute = now.getMinute();  // 30
int second = now.getSecond();  // 45

// عمليات حسابية
LocalTime later = now.plusHours(2);
LocalTime earlier = now.minusMinutes(30);

// المقارنة
boolean isBefore = time1.isBefore(time2);
```

#### 3. LocalDateTime - للتاريخ والوقت معاً

```java
import java.time.LocalDateTime;

// التاريخ والوقت الحالي
LocalDateTime now = LocalDateTime.now();
System.out.println(now); // 2025-10-04T14:30:45.123

// تاريخ ووقت محدد
LocalDateTime dateTime = LocalDateTime.of(2025, 1, 15, 14, 30, 45);

// من LocalDate و LocalTime
LocalDate date = LocalDate.of(2025, 1, 15);
LocalTime time = LocalTime.of(14, 30);
LocalDateTime combined = LocalDateTime.of(date, time);

// من String
LocalDateTime parsed = LocalDateTime.parse("2025-01-15T14:30:45");

// الحصول على التاريخ أو الوقت
LocalDate extractedDate = now.toLocalDate();
LocalTime extractedTime = now.toLocalTime();

// عمليات حسابية
LocalDateTime future = now.plusDays(5).plusHours(3);
LocalDateTime past = now.minusMonths(2).minusMinutes(30);
```

#### 4. ZonedDateTime - مع المنطقة الزمنية

```java
import java.time.ZonedDateTime;
import java.time.ZoneId;

// الوقت الحالي مع المنطقة الزمنية
ZonedDateTime nowInCairo = ZonedDateTime.now(ZoneId.of("Africa/Cairo"));
System.out.println(nowInCairo); // 2025-10-04T14:30:45.123+02:00[Africa/Cairo]

// وقت محدد مع منطقة زمنية
ZonedDateTime meeting = ZonedDateTime.of(
    LocalDateTime.of(2025, 1, 15, 14, 30),
    ZoneId.of("America/New_York")
);

// التحويل بين المناطق الزمنية
ZonedDateTime cairoTime = ZonedDateTime.now(ZoneId.of("Africa/Cairo"));
ZonedDateTime londonTime = cairoTime.withZoneSameInstant(ZoneId.of("Europe/London"));

System.out.println("Cairo: " + cairoTime);
System.out.println("London: " + londonTime);

// الحصول على جميع المناطق الزمنية
Set<String> allZones = ZoneId.getAvailableZoneIds();
```

#### 5. Period - للفترات الزمنية (بالأيام/الشهور/السنوات)

```java
import java.time.Period;

// إنشاء Period
Period period1 = Period.ofDays(5);
Period period2 = Period.ofWeeks(2);
Period period3 = Period.ofMonths(3);
Period period4 = Period.ofYears(1);
Period period5 = Period.of(1, 6, 15); // سنة و6 شهور و15 يوم

// الفرق بين تاريخين
LocalDate start = LocalDate.of(2025, 1, 1);
LocalDate end = LocalDate.of(2025, 12, 31);
Period between = Period.between(start, end);

System.out.println(between.getYears());   // 0
System.out.println(between.getMonths());  // 11
System.out.println(between.getDays());    // 30

// إضافة Period لتاريخ
LocalDate date = LocalDate.now();
LocalDate futureDate = date.plus(Period.ofMonths(3));
```

#### 6. Duration - للمدد الزمنية (بالساعات/الدقائق/الثواني)

```java
import java.time.Duration;

// إنشاء Duration
Duration duration1 = Duration.ofHours(2);
Duration duration2 = Duration.ofMinutes(30);
Duration duration3 = Duration.ofSeconds(45);
Duration duration4 = Duration.ofMillis(500);

// الفرق بين وقتين
LocalTime start = LocalTime.of(9, 0);
LocalTime end = LocalTime.of(17, 30);
Duration workDay = Duration.between(start, end);

System.out.println(workDay.toHours());   // 8
System.out.println(workDay.toMinutes()); // 510

// الفرق بين تاريخين ووقتين
LocalDateTime start2 = LocalDateTime.of(2025, 1, 1, 9, 0);
LocalDateTime end2 = LocalDateTime.of(2025, 1, 1, 17, 30);
Duration duration = Duration.between(start2, end2);
```

#### 7. Instant - نقطة زمنية (Timestamp)

```java
import java.time.Instant;

// الوقت الحالي (UTC)
Instant now = Instant.now();
System.out.println(now); // 2025-10-04T12:30:45.123Z

// من epoch (عدد الثواني منذ 1970-01-01)
Instant fromEpoch = Instant.ofEpochSecond(1609459200);

// الحصول على epoch
long epochSeconds = now.getEpochSecond();
long epochMillis = now.toEpochMilli();

// عمليات حسابية
Instant later = now.plusSeconds(3600);
Instant earlier = now.minus(Duration.ofHours(2));

// المقارنة
boolean isBefore = now.isBefore(later);
```

### Formatting & Parsing

```java
import java.time.format.DateTimeFormatter;

LocalDateTime dateTime = LocalDateTime.now();

// Formatters جاهزة
String iso = dateTime.format(DateTimeFormatter.ISO_DATE_TIME);
System.out.println(iso); // 2025-10-04T14:30:45.123

// Custom Format
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm:ss");
String formatted = dateTime.format(formatter);
System.out.println(formatted); // 04/10/2025 14:30:45

// Parsing
String dateString = "15/01/2025 14:30:00";
LocalDateTime parsed = LocalDateTime.parse(dateString, formatter);

// أمثلة على Patterns
DateTimeFormatter f1 = DateTimeFormatter.ofPattern("dd-MM-yyyy");
DateTimeFormatter f2 = DateTimeFormatter.ofPattern("MMM dd, yyyy");
DateTimeFormatter f3 = DateTimeFormatter.ofPattern("E, MMM dd yyyy");

LocalDate date = LocalDate.now();
System.out.println(date.format(f1)); // 04-10-2025
System.out.println(date.format(f2)); // Oct 04, 2025
System.out.println(date.format(f3)); // Sat, Oct 04 2025
```

### أمثلة عملية

#### مثال 1: حساب العمر

```java
public class AgeCalculator {
    public static int calculateAge(LocalDate birthDate) {
        return Period.between(birthDate, LocalDate.now()).getYears();
    }

    public static void main(String[] args) {
        LocalDate birthDate = LocalDate.of(1990, 5, 15);
        int age = calculateAge(birthDate);
        System.out.println("Age: " + age + " years");

        Period agePeriod = Period.between(birthDate, LocalDate.now());
        System.out.println("Exact age: " +
            agePeriod.getYears() + " years, " +
            agePeriod.getMonths() + " months, " +
            agePeriod.getDays() + " days");
    }
}
```

#### مثال 2: حساب أيام العمل

```java
public class WorkingDaysCalculator {
    public static long calculateWorkingDays(LocalDate start, LocalDate end) {
        return Stream.iterate(start, date -> date.plusDays(1))
            .limit(ChronoUnit.DAYS.between(start, end) + 1)
            .filter(date -> {
                DayOfWeek day = date.getDayOfWeek();
                return day != DayOfWeek.FRIDAY && day != DayOfWeek.SATURDAY;
            })
            .count();
    }

    public static void main(String[] args) {
        LocalDate start = LocalDate.of(2025, 1, 1);
        LocalDate end = LocalDate.of(2025, 1, 31);

        long workingDays = calculateWorkingDays(start, end);
        System.out.println("Working days: " + workingDays);
    }
}
```

#### مثال 3: Meeting Scheduler

```java
class Meeting {
    private ZonedDateTime startTime;
    private Duration duration;

    public Meeting(ZonedDateTime startTime, Duration duration) {
        this.startTime = startTime;
        this.duration = duration;
    }

    public ZonedDateTime getEndTime() {
        return startTime.plus(duration);
    }

    public boolean overlaps(Meeting other) {
        return this.startTime.isBefore(other.getEndTime()) &&
               this.getEndTime().isAfter(other.startTime);
    }

    public void displayInTimeZone(ZoneId zoneId) {
        ZonedDateTime converted = startTime.withZoneSameInstant(zoneId);
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("MMM dd, yyyy HH:mm z");

        System.out.println("Meeting starts at: " + converted.format(formatter));
        System.out.println("Duration: " + duration.toMinutes() + " minutes");
    }
}

// الاستخدام
Meeting meeting1 = new Meeting(
    ZonedDateTime.of(
        LocalDateTime.of(2025, 1, 15, 14, 0),
        ZoneId.of("America/New_York")
    ),
    Duration.ofHours(1)
);

meeting1.displayInTimeZone(ZoneId.of("Africa/Cairo"));
meeting1.displayInTimeZone(ZoneId.of("Europe/London"));
```

---

## Nashorn JavaScript Engine

### ما هو Nashorn؟
محرك JavaScript مدمج في Java يسمح بتشغيل كود JavaScript من Java.

**ملحوظة:** Nashorn تم إيقافه (deprecated) في Java 11 وتم حذفه في Java 15.

### الاستخدام الأساسي

```java
import javax.script.*;

public class NashornExample {
    public static void main(String[] args) throws Exception {
        // إنشاء ScriptEngine
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("nashorn");

        // تنفيذ JavaScript code
        engine.eval("print('Hello from JavaScript')");

        // تنفيذ JavaScript expression
        Object result = engine.eval("10 + 20");
        System.out.println("Result: " + result); // 30

        // استدعاء JavaScript function
        engine.eval("function add(a, b) { return a + b; }");
        Invocable invocable = (Invocable) engine;
        Object sum = invocable.invokeFunction("add", 5, 3);
        System.out.println("Sum: " + sum); // 8
    }
}
```

### تمرير متغيرات

```java
ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");

// تمرير متغيرات من Java إلى JavaScript
engine.put("name", "Ahmed");
engine.put("age", 30);

engine.eval("print('Name: ' + name + ', Age: ' + age)");

// الحصول على متغيرات من JavaScript
engine.eval("var result = age * 2");
Object result = engine.get("result");
System.out.println("Result from JS: " + result);
```

### استدعاء Java من JavaScript

```java
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }

    public int multiply(int a, int b) {
        return a * b;
    }
}

ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");

// تمرير كائن Java إلى JavaScript
Calculator calc = new Calculator();
engine.put("calculator", calc);

// استدعاء methods من JavaScript
engine.eval("var sum = calculator.add(10, 20)");
engine.eval("var product = calculator.multiply(5, 6)");
engine.eval("print('Sum: ' + sum + ', Product: ' + product)");
```

---

## أمثلة متقدمة ومجمعة

### مثال 1: معالجة بيانات الموظفين

```java
class Employee {
    private String name;
    private int age;
    private String department;
    private double salary;
    private LocalDate hireDate;

    // Constructor, Getters, Setters
    public Employee(String name, int age, String department,
                    double salary, LocalDate hireDate) {
        this.name = name;
        this.age = age;
        this.department = department;
        this.salary = salary;
        this.hireDate = hireDate;
    }

    public String getName() { return name; }
    public int getAge() { return age; }
    public String getDepartment() { return department; }
    public double getSalary() { return salary; }
    public LocalDate getHireDate() { return hireDate; }
}

public class EmployeeAnalyzer {
    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
            new Employee("Ahmed", 30, "IT", 5000, LocalDate.of(2020, 1, 15)),
            new Employee("Mohamed", 25, "HR", 4000, LocalDate.of(2021, 3, 20)),
            new Employee("Ali", 35, "IT", 6000, LocalDate.of(2019, 6, 10)),
            new Employee("Sara", 28, "Finance", 4500, LocalDate.of(2020, 9, 5)),
            new Employee("Fatma", 32, "IT", 5500, LocalDate.of(2018, 2, 28))
        );

        // 1. متوسط الرواتب حسب القسم
        Map<String, Double> avgSalaryByDept = employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.averagingDouble(Employee::getSalary)
            ));
        System.out.println("Average Salary by Department:");
        avgSalaryByDept.forEach((dept, avg) ->
            System.out.println(dept + ": $" + String.format("%.2f", avg)));

        // 2. أعلى 3 رواتب
        List<Employee> top3 = employees.stream()
            .sorted(Comparator.comparing(Employee::getSalary).reversed())
            .limit(3)
            .collect(Collectors.toList());
        System.out.println("\nTop 3 Salaries:");
        top3.forEach(e -> System.out.println(e.getName() + ": $" + e.getSalary()));

        // 3. الموظفين الذين تم توظيفهم في آخر سنتين
        LocalDate twoYearsAgo = LocalDate.now().minusYears(2);
        List<Employee> recentHires = employees.stream()
            .filter(e -> e.getHireDate().isAfter(twoYearsAgo))
            .collect(Collectors.toList());
        System.out.println("\nRecent Hires (last 2 years):");
        recentHires.forEach(e -> System.out.println(e.getName() +
            " - Hired: " + e.getHireDate()));

        // 4. حساب مدة الخدمة لكل موظف
        System.out.println("\nYears of Service:");
        employees.forEach(e -> {
            Period service = Period.between(e.getHireDate(), LocalDate.now());
            System.out.println(e.getName() + ": " +
                service.getYears() + " years, " +
                service.getMonths() + " months");
        });

        // 5. تجميع الموظفين حسب الفئة العمرية
        Map<String, List<Employee>> byAgeGroup = employees.stream()
            .collect(Collectors.groupingBy(e -> {
                int age = e.getAge();
                if (age < 30) return "20-29";
                else if (age < 35) return "30-34";
                else return "35+";
            }));
        System.out.println("\nEmployees by Age Group:");
        byAgeGroup.forEach((group, list) ->
            System.out.println(group + ": " + list.size() + " employees"));

        // 6. البحث عن موظف بأعلى راتب في IT
        Optional<Employee> highestITSalary = employees.stream()
            .filter(e -> e.getDepartment().equals("IT"))
            .max(Comparator.comparing(Employee::getSalary));

        highestITSalary.ifPresent(e ->
            System.out.println("\nHighest IT Salary: " + e.getName() +
                " - $" + e.getSalary()));
    }
}
```

### مثال 2: نظام إدارة الطلبات

```java
class Product {
    private String id;
    private String name;
    private double price;
    private String category;

    public Product(String id, String name, double price, String category) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.category = category;
    }

    public String getId() { return id; }
    public String getName() { return name; }
    public double getPrice() { return price; }
    public String getCategory() { return category; }
}

class OrderItem {
    private Product product;
    private int quantity;

    public OrderItem(Product product, int quantity) {
        this.product = product;
        this.quantity = quantity;
    }

    public Product getProduct() { return product; }
    public int getQuantity() { return quantity; }
    public double getTotal() { return product.getPrice() * quantity; }
}

class Order {
    private String id;
    private LocalDateTime orderDate;
    private List<OrderItem> items;
    private String status;

    public Order(String id, LocalDateTime orderDate,
                 List<OrderItem> items, String status) {
        this.id = id;
        this.orderDate = orderDate;
        this.items = items;
        this.status = status;
    }

    public String getId() { return id; }
    public LocalDateTime getOrderDate() { return orderDate; }
    public List<OrderItem> getItems() { return items; }
    public String getStatus() { return status; }

    public double getTotalAmount() {
        return items.stream()
            .mapToDouble(OrderItem::getTotal)
            .sum();
    }

    public int getTotalItems() {
        return items.stream()
            .mapToInt(OrderItem::getQuantity)
            .sum();
    }
}

public class OrderManagementSystem {
    public static void main(String[] args) {
        // إنشاء منتجات
        List<Product> products = Arrays.asList(
            new Product("P1", "Laptop", 1000, "Electronics"),
            new Product("P2", "Mouse", 20, "Electronics"),
            new Product("P3", "Desk", 200, "Furniture"),
            new Product("P4", "Chair", 150, "Furniture"),
            new Product("P5", "Keyboard", 50, "Electronics")
        );

        // إنشاء طلبات
        List<Order> orders = Arrays.asList(
            new Order("O1", LocalDateTime.of(2025, 1, 15, 10, 30),
                Arrays.asList(
                    new OrderItem(products.get(0), 2),
                    new OrderItem(products.get(1), 3)
                ), "Delivered"),
            new Order("O2", LocalDateTime.of(2025, 2, 20, 14, 15),
                Arrays.asList(
                    new OrderItem(products.get(2), 1),
                    new OrderItem(products.get(3), 2)
                ), "Pending"),
            new Order("O3", LocalDateTime.of(2025, 3, 10, 9, 45),
                Arrays.asList(
                    new OrderItem(products.get(4), 5)
                ), "Delivered")
        );

        // 1. إجمالي المبيعات
        double totalRevenue = orders.stream()
            .mapToDouble(Order::getTotalAmount)
            .sum();
        System.out.println("Total Revenue: $" + totalRevenue);

        // 2. المبيعات حسب الفئة
        Map<String, Double> revenueByCategory = orders.stream()
            .flatMap(order -> order.getItems().stream())
            .collect(Collectors.groupingBy(
                item -> item.getProduct().getCategory(),
                Collectors.summingDouble(OrderItem::getTotal)
            ));
        System.out.println("\nRevenue by Category:");
        revenueByCategory.forEach((cat, rev) ->
            System.out.println(cat + ": $" + rev));

        // 3. المنتجات الأكثر مبيعاً
        Map<Product, Integer> productSales = orders.stream()
            .flatMap(order -> order.getItems().stream())
            .collect(Collectors.groupingBy(
                OrderItem::getProduct,
                Collectors.summingInt(OrderItem::getQuantity)
            ));

        System.out.println("\nTop Selling Products:");
        productSales.entrySet().stream()
            .sorted(Map.Entry.<Product, Integer>comparingByValue().reversed())
            .limit(3)
            .forEach(entry -> System.out.println(
                entry.getKey().getName() + ": " + entry.getValue() + " units"));

        // 4. الطلبات في آخر شهر
        LocalDateTime oneMonthAgo = LocalDateTime.now().minusMonths(1);
        List<Order> recentOrders = orders.stream()
            .filter(o -> o.getOrderDate().isAfter(oneMonthAgo))
            .collect(Collectors.toList());
        System.out.println("\nOrders in last month: " + recentOrders.size());

        // 5. متوسط قيمة الطلب
        double avgOrderValue = orders.stream()
            .mapToDouble(Order::getTotalAmount)
            .average()
            .orElse(0.0);
        System.out.println("Average Order Value: $" +
            String.format("%.2f", avgOrderValue));

        // 6. الطلبات حسب الحالة
        Map<String, Long> ordersByStatus = orders.stream()
            .collect(Collectors.groupingBy(
                Order::getStatus,
                Collectors.counting()
            ));
        System.out.println("\nOrders by Status:");
        ordersByStatus.forEach((status, count) ->
            System.out.println(status + ": " + count));
    }
}
```

---

## الخلاصة

### Java 8 جلبت تحسينات كبيرة:

✅ **Lambda Expressions** - كود أقصر وأوضح
✅ **Stream API** - معالجة البيانات بطريقة وظيفية
✅ **Functional Interfaces** - دعم البرمجة الوظيفية
✅ **Method References** - تبسيط Lambda Expressions
✅ **Optional** - تجنب NullPointerException
✅ **Default Methods** - إضافة methods لـ Interfaces
✅ **Date/Time API** - API أفضل للتواريخ والأوقات

### الفوائد الرئيسية:

🎯 **أداء أفضل** مع Parallel Streams
🎯 **كود أقل** مع Lambda و Method References
🎯 **أمان أكثر** مع Optional
🎯 **سهولة التعامل** مع التواريخ والأوقات
🎯 **مرونة أكبر** مع Default Methods

---

**Java 8 تعتبر نقلة نوعية في تطوير Java وتستخدم بكثرة في المشاريع الحديثة!**

---

**تم إنشاء هذا الدليل كمرجع شامل لمميزات Java 8**
