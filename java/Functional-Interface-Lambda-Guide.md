# دليل شامل لـ Functional Interface و Lambda Expressions في Java

## المحتويات
1. [Functional Interface](#functional-interface)
2. [Lambda Expressions](#lambda-expressions)
3. [Built-in Functional Interfaces](#built-in-functional-interfaces)
4. [Method References](#method-references)
5. [Stream API](#stream-api)
6. [أمثلة عملية متقدمة](#أمثلة-عملية-متقدمة)

---

## Functional Interface

### ما هو Functional Interface؟

**التعريف:**
Functional Interface هو interface يحتوي على **method واحدة مجردة فقط** (Single Abstract Method - SAM).

### الخصائص:
- يحتوي على method واحدة abstract فقط
- يمكن أن يحتوي على عدة default methods
- يمكن أن يحتوي على عدة static methods
- يُستخدم مع Lambda Expressions
- يُفضل استخدام `@FunctionalInterface` annotation

### 1. إنشاء Functional Interface بسيط

```java
@FunctionalInterface
public interface Calculator {
    // Method واحدة abstract فقط
    int calculate(int a, int b);

    // يمكن إضافة default methods
    default void printResult(int result) {
        System.out.println("Result: " + result);
    }

    // يمكن إضافة static methods
    static void welcome() {
        System.out.println("Welcome to Calculator");
    }
}

// الاستخدام بالطريقة التقليدية (قبل Java 8)
public class Main {
    public static void main(String[] args) {

        // 1. Anonymous Class (الطريقة القديمة)
        Calculator addition = new Calculator() {
            @Override
            public int calculate(int a, int b) {
                return a + b;
            }
        };

        System.out.println(addition.calculate(5, 3)); // 8

        // 2. Lambda Expression (الطريقة الحديثة)
        Calculator multiplication = (a, b) -> a * b;

        System.out.println(multiplication.calculate(5, 3)); // 15
    }
}
```

### 2. أمثلة على Functional Interfaces

```java
// مثال 1: للطباعة
@FunctionalInterface
public interface Printer {
    void print(String message);
}

// الاستخدام
Printer consolePrinter = message -> System.out.println(message);
consolePrinter.print("Hello World"); // Hello World

Printer upperCasePrinter = message -> System.out.println(message.toUpperCase());
upperCasePrinter.print("hello"); // HELLO

// مثال 2: للتحقق من شرط
@FunctionalInterface
public interface Validator<T> {
    boolean validate(T value);
}

// الاستخدام
Validator<String> emailValidator = email -> email.contains("@");
System.out.println(emailValidator.validate("test@example.com")); // true
System.out.println(emailValidator.validate("invalid")); // false

Validator<Integer> ageValidator = age -> age >= 18;
System.out.println(ageValidator.validate(20)); // true
System.out.println(ageValidator.validate(15)); // false

// مثال 3: للتحويل
@FunctionalInterface
public interface Converter<F, T> {
    T convert(F from);
}

// الاستخدام
Converter<String, Integer> stringToInt = s -> Integer.parseInt(s);
System.out.println(stringToInt.convert("123")); // 123

Converter<Integer, String> intToString = i -> "Number: " + i;
System.out.println(intToString.convert(123)); // Number: 123

Converter<String, String> toUpperCase = String::toUpperCase;
System.out.println(toUpperCase.convert("hello")); // HELLO
```

### 3. Functional Interface مع أكثر من parameter

```java
@FunctionalInterface
public interface TriFunction<T, U, V, R> {
    R apply(T t, U u, V v);
}

// الاستخدام
TriFunction<Integer, Integer, Integer, Integer> sum3Numbers =
    (a, b, c) -> a + b + c;

System.out.println(sum3Numbers.apply(1, 2, 3)); // 6

TriFunction<String, String, String, String> concatenate =
    (s1, s2, s3) -> s1 + " " + s2 + " " + s3;

System.out.println(concatenate.apply("Hello", "from", "Java")); // Hello from Java
```

---

## Lambda Expressions

### ما هي Lambda Expression؟

**التعريف:**
Lambda Expression هي طريقة مختصرة لكتابة anonymous function (دالة بدون اسم).

### الصيغة العامة:

```java
(parameters) -> expression
// أو
(parameters) -> { statements; }
```

### 1. أشكال Lambda Expressions

```java
// 1. بدون parameters
() -> System.out.println("Hello")
() -> 42
() -> { return "Hello"; }

// 2. Parameter واحد (بدون أقواس)
x -> x * 2
x -> System.out.println(x)
name -> "Hello " + name

// 3. Parameter واحد (مع أقواس - اختياري)
(x) -> x * 2

// 4. أكثر من parameter (الأقواس إجبارية)
(a, b) -> a + b
(x, y) -> x * y
(name, age) -> "Name: " + name + ", Age: " + age

// 5. مع block من الأكواد
(a, b) -> {
    int sum = a + b;
    System.out.println("Sum: " + sum);
    return sum;
}

// 6. مع تحديد النوع (اختياري)
(int a, int b) -> a + b
(String s) -> s.length()
```

### 2. أمثلة عملية

```java
public class LambdaExamples {

    public static void main(String[] args) {

        // مثال 1: Runnable
        // الطريقة القديمة
        Runnable oldWay = new Runnable() {
            @Override
            public void run() {
                System.out.println("Running in old way");
            }
        };

        // الطريقة الحديثة بـ Lambda
        Runnable newWay = () -> System.out.println("Running with Lambda");

        new Thread(oldWay).start();
        new Thread(newWay).start();

        // أو مباشرة
        new Thread(() -> {
            System.out.println("Thread 1");
            System.out.println("Doing work...");
        }).start();

        // مثال 2: Comparator
        List<String> names = Arrays.asList("Ahmed", "Ziad", "Omar", "Sara");

        // الطريقة القديمة
        Collections.sort(names, new Comparator<String>() {
            @Override
            public int compare(String s1, String s2) {
                return s1.compareTo(s2);
            }
        });

        // الطريقة الحديثة
        Collections.sort(names, (s1, s2) -> s1.compareTo(s2));

        // أو أبسط
        Collections.sort(names, String::compareTo);

        // مثال 3: ActionListener (في GUI)
        // الطريقة القديمة
        button.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                System.out.println("Button clicked");
            }
        });

        // الطريقة الحديثة
        button.addActionListener(e -> System.out.println("Button clicked"));

        // مثال 4: Custom Functional Interface
        // جمع رقمين
        Calculator add = (a, b) -> a + b;
        Calculator subtract = (a, b) -> a - b;
        Calculator multiply = (a, b) -> a * b;
        Calculator divide = (a, b) -> {
            if (b == 0) {
                throw new ArithmeticException("Cannot divide by zero");
            }
            return a / b;
        };

        System.out.println(add.calculate(10, 5));      // 15
        System.out.println(subtract.calculate(10, 5)); // 5
        System.out.println(multiply.calculate(10, 5)); // 50
        System.out.println(divide.calculate(10, 5));   // 2
    }
}
```

### 3. Lambda مع Collections

```java
public class LambdaWithCollections {

    public static void main(String[] args) {

        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // 1. forEach
        numbers.forEach(n -> System.out.println(n));
        // أو
        numbers.forEach(System.out::println);

        // 2. removeIf
        List<Integer> nums = new ArrayList<>(numbers);
        nums.removeIf(n -> n % 2 == 0); // حذف الأرقام الزوجية
        System.out.println(nums); // [1, 3, 5, 7, 9]

        // 3. replaceAll
        List<String> words = Arrays.asList("hello", "world", "java");
        words.replaceAll(s -> s.toUpperCase());
        System.out.println(words); // [HELLO, WORLD, JAVA]

        // 4. sort
        List<String> names = Arrays.asList("Ahmed", "Ziad", "Omar", "Sara");
        names.sort((s1, s2) -> s1.compareTo(s2));
        System.out.println(names); // [Ahmed, Omar, Sara, Ziad]

        // 5. Map operations
        Map<String, Integer> scores = new HashMap<>();
        scores.put("Ahmed", 85);
        scores.put("Sara", 92);
        scores.put("Omar", 78);

        // forEach على Map
        scores.forEach((name, score) ->
            System.out.println(name + ": " + score));

        // compute
        scores.compute("Ahmed", (name, score) -> score + 10);
        System.out.println(scores.get("Ahmed")); // 95

        // computeIfAbsent
        scores.computeIfAbsent("Ziad", name -> 0);

        // merge
        scores.merge("Sara", 5, (oldValue, newValue) -> oldValue + newValue);
        System.out.println(scores.get("Sara")); // 97
    }
}
```

---

## Built-in Functional Interfaces

Java توفر functional interfaces جاهزة في package `java.util.function`

### 1. Predicate<T> - للتحقق من شرط

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}

// أمثلة
Predicate<Integer> isEven = n -> n % 2 == 0;
Predicate<Integer> isPositive = n -> n > 0;
Predicate<String> isEmpty = s -> s.isEmpty();
Predicate<String> startsWithA = s -> s.startsWith("A");

System.out.println(isEven.test(4));        // true
System.out.println(isPositive.test(-5));   // false
System.out.println(isEmpty.test(""));      // true
System.out.println(startsWithA.test("Ahmed")); // true

// دمج Predicates
Predicate<Integer> isEvenAndPositive = isEven.and(isPositive);
System.out.println(isEvenAndPositive.test(4));  // true
System.out.println(isEvenAndPositive.test(-4)); // false

Predicate<Integer> isEvenOrNegative = isEven.or(n -> n < 0);
System.out.println(isEvenOrNegative.test(3));  // false
System.out.println(isEvenOrNegative.test(-3)); // true

Predicate<Integer> isOdd = isEven.negate();
System.out.println(isOdd.test(5)); // true

// استخدام مع List
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
List<Integer> evenNumbers = numbers.stream()
    .filter(isEven)
    .collect(Collectors.toList());
System.out.println(evenNumbers); // [2, 4, 6, 8, 10]
```

### 2. Function<T, R> - للتحويل

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}

// أمثلة
Function<String, Integer> stringLength = s -> s.length();
Function<Integer, Integer> square = n -> n * n;
Function<String, String> toUpperCase = s -> s.toUpperCase();
Function<Integer, String> intToString = n -> "Number: " + n;

System.out.println(stringLength.apply("Hello"));  // 5
System.out.println(square.apply(5));              // 25
System.out.println(toUpperCase.apply("hello"));   // HELLO
System.out.println(intToString.apply(42));        // Number: 42

// دمج Functions
Function<String, Integer> stringToLength = s -> s.length();
Function<Integer, Integer> doubleValue = n -> n * 2;
Function<String, Integer> lengthThenDouble = stringToLength.andThen(doubleValue);

System.out.println(lengthThenDouble.apply("Hello")); // 10 (5 * 2)

// compose (عكس andThen)
Function<Integer, Integer> addOne = n -> n + 1;
Function<Integer, Integer> multiplyBy2 = n -> n * 2;
Function<Integer, Integer> composed = multiplyBy2.compose(addOne);

System.out.println(composed.apply(3)); // 8 ((3+1)*2)

// استخدام مع Stream
List<String> names = Arrays.asList("ahmed", "sara", "omar");
List<Integer> nameLengths = names.stream()
    .map(stringLength)
    .collect(Collectors.toList());
System.out.println(nameLengths); // [5, 4, 4]
```

### 3. Consumer<T> - للتنفيذ بدون return

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}

// أمثلة
Consumer<String> print = s -> System.out.println(s);
Consumer<String> printUpperCase = s -> System.out.println(s.toUpperCase());
Consumer<Integer> printSquare = n -> System.out.println(n * n);

print.accept("Hello");          // Hello
printUpperCase.accept("hello"); // HELLO
printSquare.accept(5);          // 25

// دمج Consumers
Consumer<String> printAndUpperCase = print.andThen(printUpperCase);
printAndUpperCase.accept("hello");
// Output:
// hello
// HELLO

// استخدام مع List
List<String> names = Arrays.asList("Ahmed", "Sara", "Omar");
names.forEach(print);
// Ahmed
// Sara
// Omar

names.forEach(name -> System.out.println("Hello, " + name));
// Hello, Ahmed
// Hello, Sara
// Hello, Omar
```

### 4. Supplier<T> - للإنتاج بدون input

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}

// أمثلة
Supplier<String> helloSupplier = () -> "Hello World";
Supplier<Double> randomSupplier = () -> Math.random();
Supplier<LocalDateTime> dateSupplier = () -> LocalDateTime.now();
Supplier<Integer> numberSupplier = () -> 42;

System.out.println(helloSupplier.get());   // Hello World
System.out.println(randomSupplier.get());  // رقم عشوائي
System.out.println(dateSupplier.get());    // التاريخ والوقت الحالي
System.out.println(numberSupplier.get());  // 42

// استخدام عملي - Lazy Initialization
public class DatabaseConnection {
    private static Supplier<Connection> connectionSupplier = () -> {
        System.out.println("Creating database connection...");
        return DriverManager.getConnection("jdbc:mysql://localhost/db");
    };

    public static Connection getConnection() {
        return connectionSupplier.get();
    }
}

// استخدام مع Optional
Optional<String> optional = Optional.empty();
String value = optional.orElseGet(() -> "Default Value");
System.out.println(value); // Default Value
```

### 5. BiFunction<T, U, R> - Function بـ parameter مزدوج

```java
@FunctionalInterface
public interface BiFunction<T, U, R> {
    R apply(T t, U u);
}

// أمثلة
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
BiFunction<Integer, Integer, Integer> multiply = (a, b) -> a * b;
BiFunction<String, String, String> concat = (s1, s2) -> s1 + " " + s2;
BiFunction<Integer, Integer, String> format = (a, b) -> a + " + " + b + " = " + (a + b);

System.out.println(add.apply(5, 3));           // 8
System.out.println(multiply.apply(5, 3));      // 15
System.out.println(concat.apply("Hello", "World")); // Hello World
System.out.println(format.apply(5, 3));        // 5 + 3 = 8

// استخدام مع Map
Map<String, Integer> scores = new HashMap<>();
scores.put("Ahmed", 85);

scores.merge("Ahmed", 10, (oldValue, newValue) -> oldValue + newValue);
System.out.println(scores.get("Ahmed")); // 95
```

### 6. BiConsumer<T, U> - Consumer بـ parameter مزدوج

```java
@FunctionalInterface
public interface BiConsumer<T, U> {
    void accept(T t, U u);
}

// أمثلة
BiConsumer<String, Integer> printNameAge = (name, age) ->
    System.out.println(name + " is " + age + " years old");

BiConsumer<String, String> printFullName = (firstName, lastName) ->
    System.out.println("Full Name: " + firstName + " " + lastName);

printNameAge.accept("Ahmed", 25);        // Ahmed is 25 years old
printFullName.accept("Ahmed", "Ali");    // Full Name: Ahmed Ali

// استخدام مع Map
Map<String, Integer> ages = new HashMap<>();
ages.put("Ahmed", 25);
ages.put("Sara", 22);
ages.put("Omar", 30);

ages.forEach((name, age) -> System.out.println(name + ": " + age));
// Ahmed: 25
// Sara: 22
// Omar: 30

ages.forEach(printNameAge);
```

### 7. BiPredicate<T, U> - Predicate بـ parameter مزدوج

```java
@FunctionalInterface
public interface BiPredicate<T, U> {
    boolean test(T t, U u);
}

// أمثلة
BiPredicate<String, Integer> isLengthEqual = (s, len) -> s.length() == len;
BiPredicate<Integer, Integer> isGreater = (a, b) -> a > b;
BiPredicate<String, String> startsWith = (s, prefix) -> s.startsWith(prefix);

System.out.println(isLengthEqual.test("Hello", 5));   // true
System.out.println(isGreater.test(10, 5));            // true
System.out.println(startsWith.test("Ahmed", "Ah"));   // true
```

### 8. UnaryOperator<T> - Function من نفس النوع

```java
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {
    // نفس النوع للـ input والـ output
}

// أمثلة
UnaryOperator<Integer> square = n -> n * n;
UnaryOperator<Integer> increment = n -> n + 1;
UnaryOperator<String> toUpper = s -> s.toUpperCase();

System.out.println(square.apply(5));      // 25
System.out.println(increment.apply(5));   // 6
System.out.println(toUpper.apply("hello")); // HELLO

// استخدام مع List
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
numbers.replaceAll(n -> n * 2);
System.out.println(numbers); // [2, 4, 6, 8, 10]

List<String> words = Arrays.asList("hello", "world");
words.replaceAll(String::toUpperCase);
System.out.println(words); // [HELLO, WORLD]
```

### 9. BinaryOperator<T> - BiFunction من نفس النوع

```java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T, T, T> {
    // نفس النوع للـ inputs والـ output
}

// أمثلة
BinaryOperator<Integer> sum = (a, b) -> a + b;
BinaryOperator<Integer> max = (a, b) -> a > b ? a : b;
BinaryOperator<String> concat = (s1, s2) -> s1 + s2;

System.out.println(sum.apply(5, 3));        // 8
System.out.println(max.apply(5, 3));        // 5
System.out.println(concat.apply("Hello", "World")); // HelloWorld

// استخدام مع Stream reduce
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
int total = numbers.stream().reduce(0, (a, b) -> a + b);
System.out.println(total); // 15

int maximum = numbers.stream().reduce(Integer.MIN_VALUE, (a, b) -> a > b ? a : b);
System.out.println(maximum); // 5
```

### جدول مقارن للـ Built-in Functional Interfaces

| Interface | Method | Parameters | Return | مثال |
|-----------|--------|-----------|--------|------|
| Predicate<T> | test(T t) | 1 | boolean | n -> n > 0 |
| BiPredicate<T,U> | test(T t, U u) | 2 | boolean | (s, len) -> s.length() == len |
| Function<T,R> | apply(T t) | 1 | R | s -> s.length() |
| BiFunction<T,U,R> | apply(T t, U u) | 2 | R | (a,b) -> a + b |
| Consumer<T> | accept(T t) | 1 | void | s -> System.out.println(s) |
| BiConsumer<T,U> | accept(T t, U u) | 2 | void | (k,v) -> System.out.println(k+v) |
| Supplier<T> | get() | 0 | T | () -> "Hello" |
| UnaryOperator<T> | apply(T t) | 1 | T | n -> n * 2 |
| BinaryOperator<T> | apply(T t, T t) | 2 | T | (a,b) -> a + b |

---

## Method References

Method Reference هي طريقة مختصرة للـ Lambda Expression عندما تستدعي method موجودة.

### الأنواع الأربعة:

```java
// 1. Static Method Reference
// Lambda: (args) -> ClassName.staticMethod(args)
// Method Reference: ClassName::staticMethod

Function<String, Integer> parseInt1 = s -> Integer.parseInt(s);
Function<String, Integer> parseInt2 = Integer::parseInt;

System.out.println(parseInt2.apply("123")); // 123

BiFunction<Integer, Integer, Integer> max1 = (a, b) -> Math.max(a, b);
BiFunction<Integer, Integer, Integer> max2 = Math::max;

System.out.println(max2.apply(5, 3)); // 5

// 2. Instance Method Reference (على object محدد)
// Lambda: (args) -> object.instanceMethod(args)
// Method Reference: object::instanceMethod

String str = "Hello World";
Supplier<String> upper1 = () -> str.toUpperCase();
Supplier<String> upper2 = str::toUpperCase;

System.out.println(upper2.get()); // HELLO WORLD

System system = System.out;
Consumer<String> print1 = s -> system.println(s);
Consumer<String> print2 = System.out::println;

print2.accept("Hello"); // Hello

// 3. Instance Method Reference (على parameter)
// Lambda: (obj, args) -> obj.instanceMethod(args)
// Method Reference: ClassName::instanceMethod

Function<String, Integer> length1 = s -> s.length();
Function<String, Integer> length2 = String::length;

System.out.println(length2.apply("Hello")); // 5

BiPredicate<String, String> startsWith1 = (s, prefix) -> s.startsWith(prefix);
BiPredicate<String, String> startsWith2 = String::startsWith;

System.out.println(startsWith2.test("Hello", "He")); // true

// 4. Constructor Reference
// Lambda: (args) -> new ClassName(args)
// Method Reference: ClassName::new

Supplier<List<String>> list1 = () -> new ArrayList<>();
Supplier<List<String>> list2 = ArrayList::new;

Function<Integer, List<String>> listWithSize1 = size -> new ArrayList<>(size);
Function<Integer, List<String>> listWithSize2 = ArrayList::new;

BiFunction<String, Integer, User> createUser1 = (name, age) -> new User(name, age);
BiFunction<String, Integer, User> createUser2 = User::new;

User user = createUser2.apply("Ahmed", 25);
```

### أمثلة عملية على Method References:

```java
public class MethodReferenceExamples {

    public static void main(String[] args) {

        List<String> names = Arrays.asList("ahmed", "sara", "omar", "ziad");

        // 1. طباعة الأسماء
        // Lambda
        names.forEach(name -> System.out.println(name));
        // Method Reference
        names.forEach(System.out::println);

        // 2. تحويل لـ uppercase
        // Lambda
        List<String> upper1 = names.stream()
            .map(s -> s.toUpperCase())
            .collect(Collectors.toList());

        // Method Reference
        List<String> upper2 = names.stream()
            .map(String::toUpperCase)
            .collect(Collectors.toList());

        System.out.println(upper2); // [AHMED, SARA, OMAR, ZIAD]

        // 3. ترتيب الأسماء
        // Lambda
        names.sort((s1, s2) -> s1.compareTo(s2));

        // Method Reference
        names.sort(String::compareTo);

        System.out.println(names); // [ahmed, omar, sara, ziad]

        // 4. تحويل String[] إلى Integer[]
        String[] numberStrings = {"1", "2", "3", "4", "5"};

        // Lambda
        Integer[] numbers1 = Arrays.stream(numberStrings)
            .map(s -> Integer.parseInt(s))
            .toArray(Integer[]::new);

        // Method Reference
        Integer[] numbers2 = Arrays.stream(numberStrings)
            .map(Integer::parseInt)
            .toArray(Integer[]::new);

        System.out.println(Arrays.toString(numbers2)); // [1, 2, 3, 4, 5]

        // 5. إنشاء objects
        List<String> userNames = Arrays.asList("Ahmed", "Sara", "Omar");

        // Lambda
        List<User> users1 = userNames.stream()
            .map(name -> new User(name))
            .collect(Collectors.toList());

        // Method Reference
        List<User> users2 = userNames.stream()
            .map(User::new)
            .collect(Collectors.toList());
    }
}

class User {
    private String name;
    private int age;

    public User(String name) {
        this.name = name;
    }

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // getters/setters
}
```

---

## Stream API

Stream API تستخدم Functional Interfaces و Lambda Expressions بشكل كبير.

### 1. إنشاء Streams

```java
// 1. من Collection
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream1 = list.stream();

// 2. من Array
String[] array = {"a", "b", "c"};
Stream<String> stream2 = Arrays.stream(array);

// 3. من values
Stream<String> stream3 = Stream.of("a", "b", "c");

// 4. Stream فاضي
Stream<String> stream4 = Stream.empty();

// 5. Infinite Stream
Stream<Integer> infiniteStream = Stream.iterate(0, n -> n + 1);
Stream<Double> randomStream = Stream.generate(Math::random);

// 6. من String
IntStream charStream = "Hello".chars();

// 7. Range
IntStream range1 = IntStream.range(1, 5);        // 1,2,3,4
IntStream range2 = IntStream.rangeClosed(1, 5);  // 1,2,3,4,5
```

### 2. Intermediate Operations (تعيد Stream)

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// 1. filter - تصفية
List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
System.out.println(evenNumbers); // [2, 4, 6, 8, 10]

// 2. map - تحويل
List<Integer> squares = numbers.stream()
    .map(n -> n * n)
    .collect(Collectors.toList());
System.out.println(squares); // [1, 4, 9, 16, 25, ...]

// 3. flatMap - تسطيح
List<List<Integer>> listOfLists = Arrays.asList(
    Arrays.asList(1, 2, 3),
    Arrays.asList(4, 5, 6),
    Arrays.asList(7, 8, 9)
);

List<Integer> flattened = listOfLists.stream()
    .flatMap(list -> list.stream())
    .collect(Collectors.toList());
System.out.println(flattened); // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// 4. distinct - إزالة التكرار
List<Integer> withDuplicates = Arrays.asList(1, 2, 2, 3, 3, 3, 4, 4, 5);
List<Integer> unique = withDuplicates.stream()
    .distinct()
    .collect(Collectors.toList());
System.out.println(unique); // [1, 2, 3, 4, 5]

// 5. sorted - ترتيب
List<String> names = Arrays.asList("Ziad", "Ahmed", "Sara", "Omar");
List<String> sorted = names.stream()
    .sorted()
    .collect(Collectors.toList());
System.out.println(sorted); // [Ahmed, Omar, Sara, Ziad]

// ترتيب عكسي
List<String> reverseSorted = names.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());

// ترتيب حسب الطول
List<String> sortedByLength = names.stream()
    .sorted(Comparator.comparing(String::length))
    .collect(Collectors.toList());

// 6. peek - للتنفيذ أثناء المعالجة (للـ debugging)
List<Integer> result = numbers.stream()
    .filter(n -> n % 2 == 0)
    .peek(n -> System.out.println("Filtered: " + n))
    .map(n -> n * 2)
    .peek(n -> System.out.println("Mapped: " + n))
    .collect(Collectors.toList());

// 7. limit - تحديد العدد
List<Integer> first5 = numbers.stream()
    .limit(5)
    .collect(Collectors.toList());
System.out.println(first5); // [1, 2, 3, 4, 5]

// 8. skip - تخطي عناصر
List<Integer> skip5 = numbers.stream()
    .skip(5)
    .collect(Collectors.toList());
System.out.println(skip5); // [6, 7, 8, 9, 10]

// 9. takeWhile (Java 9+) - أخذ عناصر طالما الشرط صحيح
List<Integer> takeWhileResult = numbers.stream()
    .takeWhile(n -> n < 5)
    .collect(Collectors.toList());
System.out.println(takeWhileResult); // [1, 2, 3, 4]

// 10. dropWhile (Java 9+) - حذف عناصر طالما الشرط صحيح
List<Integer> dropWhileResult = numbers.stream()
    .dropWhile(n -> n < 5)
    .collect(Collectors.toList());
System.out.println(dropWhileResult); // [5, 6, 7, 8, 9, 10]
```

### 3. Terminal Operations (تنهي Stream)

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// 1. forEach - تنفيذ على كل عنصر
numbers.stream()
    .forEach(n -> System.out.println(n));

// أو
numbers.forEach(System.out::println);

// 2. collect - جمع النتائج
List<Integer> list = numbers.stream()
    .filter(n -> n > 5)
    .collect(Collectors.toList());

Set<Integer> set = numbers.stream()
    .collect(Collectors.toSet());

String joined = numbers.stream()
    .map(String::valueOf)
    .collect(Collectors.joining(", "));
System.out.println(joined); // 1, 2, 3, 4, 5, 6, 7, 8, 9, 10

// 3. reduce - تقليل لقيمة واحدة
// مجموع الأرقام
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);
System.out.println(sum); // 55

// أو
int sum2 = numbers.stream()
    .reduce(0, Integer::sum);

// ضرب الأرقام
int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);
System.out.println(product); // 3628800

// 4. count - عدد العناصر
long count = numbers.stream()
    .filter(n -> n % 2 == 0)
    .count();
System.out.println(count); // 5

// 5. anyMatch - هل يوجد عنصر يطابق الشرط
boolean hasEven = numbers.stream()
    .anyMatch(n -> n % 2 == 0);
System.out.println(hasEven); // true

// 6. allMatch - هل كل العناصر تطابق الشرط
boolean allPositive = numbers.stream()
    .allMatch(n -> n > 0);
System.out.println(allPositive); // true

// 7. noneMatch - هل لا يوجد عنصر يطابق الشرط
boolean noneNegative = numbers.stream()
    .noneMatch(n -> n < 0);
System.out.println(noneNegative); // true

// 8. findFirst - أول عنصر
Optional<Integer> first = numbers.stream()
    .filter(n -> n > 5)
    .findFirst();
System.out.println(first.get()); // 6

// 9. findAny - أي عنصر (مفيد في Parallel Streams)
Optional<Integer> any = numbers.stream()
    .filter(n -> n > 5)
    .findAny();
System.out.println(any.get()); // 6 (أو أي رقم > 5)

// 10. min/max - أصغر/أكبر قيمة
Optional<Integer> min = numbers.stream()
    .min(Integer::compareTo);
System.out.println(min.get()); // 1

Optional<Integer> max = numbers.stream()
    .max(Integer::compareTo);
System.out.println(max.get()); // 10

// 11. toArray - تحويل لـ array
Integer[] array = numbers.stream()
    .toArray(Integer[]::new);
```

### 4. أمثلة عملية متقدمة

```java
// مثال 1: معالجة قائمة Employees
class Employee {
    private String name;
    private String department;
    private int age;
    private double salary;

    // constructor, getters, setters
}

List<Employee> employees = Arrays.asList(
    new Employee("Ahmed", "IT", 30, 5000),
    new Employee("Sara", "HR", 28, 4500),
    new Employee("Omar", "IT", 35, 6000),
    new Employee("Ziad", "Finance", 32, 5500),
    new Employee("Nour", "IT", 26, 4000)
);

// 1. الموظفين في IT بعمر أكثر من 25
List<Employee> itEmployees = employees.stream()
    .filter(e -> e.getDepartment().equals("IT"))
    .filter(e -> e.getAge() > 25)
    .collect(Collectors.toList());

// 2. متوسط الرواتب في IT
double avgSalary = employees.stream()
    .filter(e -> e.getDepartment().equals("IT"))
    .mapToDouble(Employee::getSalary)
    .average()
    .orElse(0.0);
System.out.println("Average IT Salary: " + avgSalary); // 5000.0

// 3. أعلى راتب
Employee highestPaid = employees.stream()
    .max(Comparator.comparing(Employee::getSalary))
    .orElse(null);
System.out.println("Highest paid: " + highestPaid.getName()); // Omar

// 4. تجميع حسب القسم
Map<String, List<Employee>> byDepartment = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

byDepartment.forEach((dept, emps) -> {
    System.out.println(dept + ": " + emps.size() + " employees");
});

// 5. متوسط الراتب لكل قسم
Map<String, Double> avgSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));

// 6. عدد الموظفين في كل قسم
Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.counting()
    ));

// 7. أسماء الموظفين مفصولة بفاصلة
String names = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.joining(", "));
System.out.println(names); // Ahmed, Sara, Omar, Ziad, Nour

// 8. زيادة رواتب IT بنسبة 10%
employees.stream()
    .filter(e -> e.getDepartment().equals("IT"))
    .forEach(e -> e.setSalary(e.getSalary() * 1.1));

// 9. تقسيم حسب الراتب (أكثر أو أقل من 5000)
Map<Boolean, List<Employee>> partitioned = employees.stream()
    .collect(Collectors.partitioningBy(e -> e.getSalary() >= 5000));

List<Employee> highSalary = partitioned.get(true);
List<Employee> lowSalary = partitioned.get(false);

// 10. إحصائيات الرواتب
DoubleSummaryStatistics stats = employees.stream()
    .mapToDouble(Employee::getSalary)
    .summaryStatistics();

System.out.println("Count: " + stats.getCount());
System.out.println("Sum: " + stats.getSum());
System.out.println("Min: " + stats.getMin());
System.out.println("Max: " + stats.getMax());
System.out.println("Average: " + stats.getAverage());
```

### 5. Parallel Streams

```java
List<Integer> numbers = IntStream.rangeClosed(1, 1000000)
    .boxed()
    .collect(Collectors.toList());

// Sequential Stream
long start1 = System.currentTimeMillis();
long sum1 = numbers.stream()
    .mapToLong(n -> n)
    .sum();
long end1 = System.currentTimeMillis();
System.out.println("Sequential: " + (end1 - start1) + "ms");

// Parallel Stream
long start2 = System.currentTimeMillis();
long sum2 = numbers.parallelStream()
    .mapToLong(n -> n)
    .sum();
long end2 = System.currentTimeMillis();
System.out.println("Parallel: " + (end2 - start2) + "ms");

// تحويل لـ parallel
Stream<Integer> parallelStream = numbers.stream().parallel();

// تحويل لـ sequential
Stream<Integer> sequentialStream = numbers.parallelStream().sequential();
```

---

## أمثلة عملية متقدمة

### 1. Validation Framework

```java
@FunctionalInterface
public interface ValidationRule<T> {
    boolean validate(T value);

    default ValidationRule<T> and(ValidationRule<T> other) {
        return value -> this.validate(value) && other.validate(value);
    }

    default ValidationRule<T> or(ValidationRule<T> other) {
        return value -> this.validate(value) || other.validate(value);
    }
}

public class Validator<T> {
    private List<ValidationRule<T>> rules = new ArrayList<>();

    public Validator<T> addRule(ValidationRule<T> rule) {
        rules.add(rule);
        return this;
    }

    public boolean validate(T value) {
        return rules.stream().allMatch(rule -> rule.validate(value));
    }

    public List<String> getErrors(T value) {
        // implementation
        return new ArrayList<>();
    }
}

// الاستخدام
public class UserValidator {

    public static void main(String[] args) {

        // قواعد التحقق
        ValidationRule<String> notEmpty = s -> s != null && !s.isEmpty();
        ValidationRule<String> minLength = s -> s.length() >= 3;
        ValidationRule<String> maxLength = s -> s.length() <= 50;
        ValidationRule<String> alphanumeric = s -> s.matches("[a-zA-Z0-9]+");
        ValidationRule<String> hasUpperCase = s -> s.matches(".*[A-Z].*");
        ValidationRule<String> hasLowerCase = s -> s.matches(".*[a-z].*");
        ValidationRule<String> hasDigit = s -> s.matches(".*[0-9].*");
        ValidationRule<String> hasSpecialChar = s -> s.matches(".*[@#$%^&+=].*");

        // Username Validator
        Validator<String> usernameValidator = new Validator<String>()
            .addRule(notEmpty)
            .addRule(minLength)
            .addRule(maxLength)
            .addRule(alphanumeric);

        System.out.println(usernameValidator.validate("Ahmed123")); // true
        System.out.println(usernameValidator.validate("ab"));        // false

        // Password Validator
        ValidationRule<String> strongPassword = notEmpty
            .and(minLength)
            .and(hasUpperCase)
            .and(hasLowerCase)
            .and(hasDigit)
            .and(hasSpecialChar);

        Validator<String> passwordValidator = new Validator<String>()
            .addRule(strongPassword);

        System.out.println(passwordValidator.validate("Pass123@"));  // true
        System.out.println(passwordValidator.validate("weak"));      // false

        // Email Validator
        ValidationRule<String> emailFormat = s -> s.matches("^[A-Za-z0-9+_.-]+@(.+)$");

        Validator<String> emailValidator = new Validator<String>()
            .addRule(notEmpty)
            .addRule(emailFormat);

        System.out.println(emailValidator.validate("test@example.com")); // true
        System.out.println(emailValidator.validate("invalid"));           // false
    }
}
```

### 2. Pipeline Pattern

```java
@FunctionalInterface
public interface Pipeline<T> {
    T process(T input);

    default Pipeline<T> andThen(Pipeline<T> next) {
        return input -> next.process(this.process(input));
    }
}

public class TextPipeline {

    public static void main(String[] args) {

        // خطوات المعالجة
        Pipeline<String> trimSpaces = String::trim;
        Pipeline<String> toLowerCase = String::toLowerCase;
        Pipeline<String> removeSpecialChars = s -> s.replaceAll("[^a-zA-Z0-9\\s]", "");
        Pipeline<String> capitalizeFirstLetter = s ->
            s.substring(0, 1).toUpperCase() + s.substring(1);

        // بناء الـ pipeline
        Pipeline<String> textProcessor = trimSpaces
            .andThen(toLowerCase)
            .andThen(removeSpecialChars)
            .andThen(capitalizeFirstLetter);

        String input = "  Hello, WORLD!!!  ";
        String output = textProcessor.process(input);

        System.out.println("Input: '" + input + "'");
        System.out.println("Output: '" + output + "'"); // Hello world

        // مثال آخر - معالجة أرقام
        Pipeline<Integer> addTen = n -> n + 10;
        Pipeline<Integer> multiplyByTwo = n -> n * 2;
        Pipeline<Integer> square = n -> n * n;

        Pipeline<Integer> numberProcessor = addTen
            .andThen(multiplyByTwo)
            .andThen(square);

        System.out.println(numberProcessor.process(5)); // ((5+10)*2)^2 = 900
    }
}
```

### 3. Strategy Pattern مع Lambda

```java
@FunctionalInterface
public interface DiscountStrategy {
    double applyDiscount(double price);
}

public class ShoppingCart {
    private List<Double> prices = new ArrayList<>();
    private DiscountStrategy discountStrategy;

    public void addItem(double price) {
        prices.add(price);
    }

    public void setDiscountStrategy(DiscountStrategy strategy) {
        this.discountStrategy = strategy;
    }

    public double calculateTotal() {
        double total = prices.stream()
            .mapToDouble(Double::doubleValue)
            .sum();

        if (discountStrategy != null) {
            total = discountStrategy.applyDiscount(total);
        }

        return total;
    }
}

public class DiscountExample {

    public static void main(String[] args) {

        // استراتيجيات الخصم
        DiscountStrategy noDiscount = price -> price;
        DiscountStrategy tenPercent = price -> price * 0.9;
        DiscountStrategy twentyPercent = price -> price * 0.8;
        DiscountStrategy fiftyPercent = price -> price * 0.5;

        // خصم تدريجي
        DiscountStrategy tieredDiscount = price -> {
            if (price > 1000) return price * 0.8;      // 20% off
            if (price > 500) return price * 0.9;       // 10% off
            if (price > 100) return price * 0.95;      // 5% off
            return price;
        };

        // خصم العيد
        DiscountStrategy holidayDiscount = price -> {
            LocalDate today = LocalDate.now();
            if (today.getMonthValue() == 12) {
                return price * 0.7; // 30% off في ديسمبر
            }
            return price;
        };

        // استخدام
        ShoppingCart cart = new ShoppingCart();
        cart.addItem(100);
        cart.addItem(200);
        cart.addItem(300);

        cart.setDiscountStrategy(noDiscount);
        System.out.println("No discount: " + cart.calculateTotal()); // 600

        cart.setDiscountStrategy(tenPercent);
        System.out.println("10% off: " + cart.calculateTotal()); // 540

        cart.setDiscountStrategy(tieredDiscount);
        System.out.println("Tiered discount: " + cart.calculateTotal()); // 570
    }
}
```

### 4. Event Handler System

```java
@FunctionalInterface
public interface EventHandler<T> {
    void handle(T event);
}

public class EventBus<T> {
    private Map<String, List<EventHandler<T>>> handlers = new HashMap<>();

    public void subscribe(String eventType, EventHandler<T> handler) {
        handlers.computeIfAbsent(eventType, k -> new ArrayList<>())
                .add(handler);
    }

    public void publish(String eventType, T event) {
        List<EventHandler<T>> eventHandlers = handlers.get(eventType);
        if (eventHandlers != null) {
            eventHandlers.forEach(handler -> handler.handle(event));
        }
    }

    public void unsubscribe(String eventType, EventHandler<T> handler) {
        List<EventHandler<T>> eventHandlers = handlers.get(eventType);
        if (eventHandlers != null) {
            eventHandlers.remove(handler);
        }
    }
}

class UserEvent {
    private String type;
    private String username;
    private LocalDateTime timestamp;

    // constructor, getters, setters
}

public class EventSystemExample {

    public static void main(String[] args) {

        EventBus<UserEvent> eventBus = new EventBus<>();

        // Subscribe to events
        eventBus.subscribe("USER_LOGIN", event -> {
            System.out.println("Logging: User " + event.getUsername() + " logged in");
        });

        eventBus.subscribe("USER_LOGIN", event -> {
            System.out.println("Analytics: Recording login for " + event.getUsername());
        });

        eventBus.subscribe("USER_LOGIN", event -> {
            System.out.println("Email: Sending login notification to " + event.getUsername());
        });

        eventBus.subscribe("USER_LOGOUT", event -> {
            System.out.println("User " + event.getUsername() + " logged out");
        });

        // Publish events
        UserEvent loginEvent = new UserEvent("USER_LOGIN", "ahmed", LocalDateTime.now());
        eventBus.publish("USER_LOGIN", loginEvent);

        System.out.println("---");

        UserEvent logoutEvent = new UserEvent("USER_LOGOUT", "ahmed", LocalDateTime.now());
        eventBus.publish("USER_LOGOUT", logoutEvent);
    }
}
```

### 5. Query Builder Pattern

```java
@FunctionalInterface
public interface Condition<T> {
    boolean test(T item);
}

public class QueryBuilder<T> {
    private List<Condition<T>> conditions = new ArrayList<>();

    public QueryBuilder<T> where(Condition<T> condition) {
        conditions.add(condition);
        return this;
    }

    public QueryBuilder<T> and(Condition<T> condition) {
        return where(condition);
    }

    public QueryBuilder<T> or(Condition<T> condition) {
        if (conditions.isEmpty()) {
            return where(condition);
        }

        Condition<T> combined = item -> {
            Condition<T> lastCondition = conditions.remove(conditions.size() - 1);
            return lastCondition.test(item) || condition.test(item);
        };

        conditions.add(combined);
        return this;
    }

    public List<T> execute(List<T> items) {
        return items.stream()
            .filter(item -> conditions.stream().allMatch(c -> c.test(item)))
            .collect(Collectors.toList());
    }
}

class Product {
    private String name;
    private String category;
    private double price;
    private int stock;

    // constructor, getters, setters
}

public class QueryBuilderExample {

    public static void main(String[] args) {

        List<Product> products = Arrays.asList(
            new Product("Laptop", "Electronics", 1000, 5),
            new Product("Mouse", "Electronics", 25, 50),
            new Product("Desk", "Furniture", 300, 10),
            new Product("Chair", "Furniture", 150, 20),
            new Product("Phone", "Electronics", 800, 15)
        );

        // Query 1: Electronics with price > 100
        List<Product> result1 = new QueryBuilder<Product>()
            .where(p -> p.getCategory().equals("Electronics"))
            .and(p -> p.getPrice() > 100)
            .execute(products);

        System.out.println("Electronics > 100:");
        result1.forEach(p -> System.out.println(p.getName()));

        // Query 2: Low stock items
        List<Product> result2 = new QueryBuilder<Product>()
            .where(p -> p.getStock() < 10)
            .execute(products);

        System.out.println("\nLow stock:");
        result2.forEach(p -> System.out.println(p.getName()));

        // Query 3: Electronics OR Furniture with price < 500
        List<Product> result3 = new QueryBuilder<Product>()
            .where(p -> p.getCategory().equals("Electronics"))
            .or(p -> p.getCategory().equals("Furniture"))
            .and(p -> p.getPrice() < 500)
            .execute(products);

        System.out.println("\nElectronics or Furniture < 500:");
        result3.forEach(p -> System.out.println(p.getName()));
    }
}
```

---

## الخلاصة

### متى تستخدم Functional Interface و Lambda؟

✅ **استخدمها عندما:**
- تحتاج callback function
- تريد تبسيط Anonymous classes
- تعمل مع Collections و Streams
- تريد كتابة كود أقصر وأوضح
- تطبق Functional Programming patterns

❌ **لا تستخدمها عندما:**
- الـ Lambda معقدة جداً (أكثر من 3 أسطر)
- تحتاج state أو fields
- الـ method تُستدعى من أماكن كثيرة (الأفضل method عادية)

### نصائح مهمة:

1. **اجعل Lambda قصيرة ومختصرة**
```java
// ✅ جيد
list.forEach(System.out::println);

// ❌ سيء
list.forEach(item -> {
    // 20 lines of code
});
```

2. **استخدم Method Reference عندما ممكن**
```java
// ✅ أفضل
list.stream().map(String::toUpperCase)

// ❌ مقبول لكن أطول
list.stream().map(s -> s.toUpperCase())
```

3. **اجعل Functional Interfaces واضحة**
```java
// ✅ جيد
@FunctionalInterface
public interface PriceCalculator {
    double calculate(double basePrice, double tax);
}

// ❌ سيء - اسم غير واضح
@FunctionalInterface
public interface Processor {
    void process(Object obj);
}
```

4. **استخدم Built-in Interfaces عندما ممكن**
- بدلاً من إنشاء interface جديد، استخدم `Predicate`, `Function`, `Consumer`, إلخ.

---

## مراجع إضافية

### Functional Interfaces الأكثر استخداماً:

| Interface | Method | استخدام |
|-----------|--------|----------|
| `Runnable` | `run()` | Threads |
| `Callable<V>` | `call()` | Threads بـ return value |
| `Comparator<T>` | `compare(T, T)` | الترتيب |
| `Predicate<T>` | `test(T)` | الشروط |
| `Function<T,R>` | `apply(T)` | التحويل |
| `Consumer<T>` | `accept(T)` | الاستهلاك |
| `Supplier<T>` | `get()` | الإنتاج |

**حظاً موفقاً في تعلم Functional Programming! 🚀**