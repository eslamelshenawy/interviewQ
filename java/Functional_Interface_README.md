# Functional Interface في Java

## ما هو الـ Functional Interface؟
الـ Functional Interface هو interface يحتوي على **method واحدة abstract فقط** (SAM - Single Abstract Method). يمكن أن يحتوي على أي عدد من الـ default methods أو static methods، لكن method abstract واحدة فقط.

## الـ Annotation
```java
@FunctionalInterface
```
هذا annotation اختياري لكنه مفيد لأنه:
- يوضح النية أن هذا functional interface
- يجعل الـ compiler يتحقق من وجود method واحدة abstract فقط

## أمثلة على Functional Interfaces المدمجة

### 1. Predicate<T>
يستخدم للتحقق من شرط معين
```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}

// مثال
Predicate<Integer> isEven = n -> n % 2 == 0;
System.out.println(isEven.test(4)); // true
```

### 2. Function<T, R>
يستقبل قيمة من نوع T ويرجع قيمة من نوع R
```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}

// مثال
Function<String, Integer> stringLength = str -> str.length();
System.out.println(stringLength.apply("Hello")); // 5
```

### 3. Consumer<T>
يستقبل قيمة ولا يرجع شيء (void)
```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}

// مثال
Consumer<String> printer = msg -> System.out.println(msg);
printer.accept("Hello World");
```

### 4. Supplier<T>
لا يستقبل شيء لكن يرجع قيمة
```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}

// مثال
Supplier<Double> randomNumber = () -> Math.random();
System.out.println(randomNumber.get());
```

### 5. BiFunction<T, U, R>
يستقبل قيمتين ويرجع قيمة
```java
@FunctionalInterface
public interface BiFunction<T, U, R> {
    R apply(T t, U u);
}

// مثال
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
System.out.println(add.apply(5, 3)); // 8
```

## إنشاء Functional Interface خاص بك

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);

    // يمكن إضافة default methods
    default void printResult(int result) {
        System.out.println("Result: " + result);
    }

    // يمكن إضافة static methods
    static void printInfo() {
        System.out.println("Calculator Interface");
    }
}

// الاستخدام
Calculator addition = (a, b) -> a + b;
Calculator multiplication = (a, b) -> a * b;

System.out.println(addition.calculate(10, 5));      // 15
System.out.println(multiplication.calculate(10, 5)); // 50
```

## Lambda Expressions
الـ Lambda expressions هي طريقة مختصرة لكتابة implementation للـ functional interface:

```java
// بدلاً من Anonymous Inner Class
Runnable r1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running");
    }
};

// نستخدم Lambda
Runnable r2 = () -> System.out.println("Running");

// أمثلة أخرى
Comparator<String> comp = (s1, s2) -> s1.compareTo(s2);
Predicate<String> isEmpty = str -> str.isEmpty();
Function<Integer, String> converter = num -> String.valueOf(num);
```

## Method References
طريقة أكثر اختصاراً للـ lambda عندما تستدعي method موجودة:

```java
// Lambda
Function<String, Integer> f1 = str -> Integer.parseInt(str);

// Method Reference
Function<String, Integer> f2 = Integer::parseInt;

// أنواع Method References:
// 1. Static method reference
Function<String, Integer> parser = Integer::parseInt;

// 2. Instance method reference
String str = "Hello";
Supplier<Integer> length = str::length;

// 3. Constructor reference
Supplier<List<String>> listSupplier = ArrayList::new;
```

## استخدامات عملية

### مع Streams
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// filter يستخدم Predicate
numbers.stream()
    .filter(n -> n % 2 == 0)
    .forEach(System.out::println); // 2, 4

// map يستخدم Function
List<String> strings = numbers.stream()
    .map(n -> "Number: " + n)
    .collect(Collectors.toList());
```

### مع Optional
```java
Optional<String> optional = Optional.of("Hello");

// ifPresent يستخدم Consumer
optional.ifPresent(val -> System.out.println(val));

// map يستخدم Function
Optional<Integer> length = optional.map(String::length);

// filter يستخدم Predicate
optional.filter(s -> s.length() > 3)
        .ifPresent(System.out::println);
```

### في التصميم
```java
public class Calculator {
    public int operate(int a, int b, BiFunction<Integer, Integer, Integer> operation) {
        return operation.apply(a, b);
    }

    public static void main(String[] args) {
        Calculator calc = new Calculator();

        // يمكن تمرير أي عملية
        System.out.println(calc.operate(10, 5, (x, y) -> x + y));  // 15
        System.out.println(calc.operate(10, 5, (x, y) -> x - y));  // 5
        System.out.println(calc.operate(10, 5, (x, y) -> x * y));  // 50
    }
}
```

## الفوائد

1. **كود أنظف وأقصر**: Lambda expressions تجعل الكود أكثر قابلية للقراءة
2. **Functional Programming**: تدعم البرمجة الوظيفية في Java
3. **مرونة أكبر**: يمكن تمرير السلوك كـ parameter
4. **أداء أفضل**: في بعض الحالات، الـ lambda يمكن أن تكون أسرع من Anonymous Inner Classes

## ملاحظات مهمة

- الـ Functional Interface يجب أن يحتوي على method abstract واحدة فقط
- Methods من Object class (equals, hashCode, toString) لا تُحسب
- يمكن أن يحتوي على أي عدد من default و static methods
- الـ @FunctionalInterface annotation اختياري لكنه ممارسة جيدة

## مثال شامل

```java
import java.util.*;
import java.util.function.*;

public class FunctionalInterfaceDemo {
    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
            new Employee("Ahmed", 25, 50000),
            new Employee("Sara", 30, 60000),
            new Employee("Mohamed", 28, 55000),
            new Employee("Fatima", 35, 70000)
        );

        // Predicate - للفلترة
        Predicate<Employee> highSalary = emp -> emp.salary > 55000;

        // Function - للتحويل
        Function<Employee, String> getName = Employee::getName;

        // Consumer - للطباعة
        Consumer<String> printer = System.out::println;

        // استخدامهم مع Stream
        employees.stream()
            .filter(highSalary)
            .map(getName)
            .forEach(printer);
        // Output: Sara, Fatima

        // BiFunction - للمقارنة
        BiFunction<Employee, Employee, Employee> higherSalary =
            (e1, e2) -> e1.salary > e2.salary ? e1 : e2;

        Employee richest = employees.stream()
            .reduce(higherSalary::apply)
            .orElse(null);
        System.out.println("Richest: " + richest.getName()); // Fatima
    }
}

class Employee {
    String name;
    int age;
    double salary;

    Employee(String name, int age, double salary) {
        this.name = name;
        this.age = age;
        this.salary = salary;
    }

    String getName() { return name; }
}
```