# Lambda Expression في Java

## ما هو الـ Lambda Expression؟
Lambda Expression هو دالة مجهولة (anonymous function) - دالة بدون اسم. تم إضافته في Java 8 لتسهيل كتابة كود أقصر وأوضح، خاصة عند التعامل مع Functional Interfaces.

## الـ Syntax الأساسي

```java
(parameters) -> expression
// أو
(parameters) -> { statements; }
```

### أمثلة على الـ Syntax:
```java
// بدون parameters
() -> System.out.println("Hello")

// parameter واحد (يمكن حذف الأقواس)
x -> x * 2
(x) -> x * 2  // نفس الشيء

// أكثر من parameter
(x, y) -> x + y

// مع body كامل
(x, y) -> {
    int result = x + y;
    return result;
}
```

## القواعد الأساسية

### 1. Type Inference (استنتاج النوع)
```java
// بدلاً من
(String s) -> s.toUpperCase()

// يمكن كتابة
s -> s.toUpperCase()
// الـ compiler يستنتج أن s من نوع String
```

### 2. الأقواس مع Parameters
```java
// بدون parameters - الأقواس مطلوبة
() -> 42

// parameter واحد - الأقواس اختيارية
x -> x * x
(x) -> x * x  // كلاهما صحيح

// أكثر من parameter - الأقواس مطلوبة
(x, y) -> x + y
```

### 3. الـ Body والـ return
```java
// Single expression - لا نحتاج return
x -> x * 2

// Block body - نحتاج return
x -> {
    int result = x * 2;
    return result;
}

// void methods - لا نحتاج return
msg -> System.out.println(msg)
```

## المقارنة مع Anonymous Inner Class

### قبل Java 8 (Anonymous Inner Class):
```java
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running");
    }
};

Comparator<String> comparator = new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return s1.compareTo(s2);
    }
};
```

### مع Lambda Expression:
```java
Runnable runnable = () -> System.out.println("Running");

Comparator<String> comparator = (s1, s2) -> s1.compareTo(s2);
```

## أمثلة عملية

### 1. مع Collections
```java
List<String> names = Arrays.asList("Ahmed", "Sara", "Mohamed", "Fatima");

// Sorting
Collections.sort(names, (s1, s2) -> s1.compareTo(s2));
// أو أبسط
names.sort((s1, s2) -> s1.compareTo(s2));

// forEach
names.forEach(name -> System.out.println(name));

// removeIf
List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
numbers.removeIf(n -> n % 2 == 0); // حذف الأرقام الزوجية
```

### 2. مع Streams
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// filter و map و reduce
int sum = numbers.stream()
    .filter(n -> n % 2 == 0)           // الأرقام الزوجية فقط
    .map(n -> n * n)                    // تربيع كل رقم
    .reduce(0, (a, b) -> a + b);        // جمع النتائج

System.out.println("Sum of squares of even numbers: " + sum);

// أمثلة أخرى
List<String> filtered = names.stream()
    .filter(name -> name.startsWith("A"))
    .map(name -> name.toUpperCase())
    .collect(Collectors.toList());
```

### 3. مع Threads
```java
// Traditional way
Thread t1 = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Thread running");
    }
});

// Lambda way
Thread t2 = new Thread(() -> System.out.println("Thread running"));

// مع ExecutorService
ExecutorService executor = Executors.newFixedThreadPool(2);
executor.submit(() -> {
    System.out.println("Task 1 executing");
});
```

### 4. Event Handlers (في JavaFX/Swing)
```java
// JavaFX
button.setOnAction(event -> System.out.println("Button clicked!"));

// Swing
button.addActionListener(e -> {
    System.out.println("Button clicked!");
    label.setText("Clicked!");
});
```

## Variable Capture (Accessing Variables)

### Local Variables
Lambda يمكنها الوصول للمتغيرات المحلية، لكن يجب أن تكون **final أو effectively final**:

```java
int multiplier = 3;  // effectively final (لا يتغير)
Function<Integer, Integer> multiply = x -> x * multiplier;

// هذا خطأ:
int counter = 0;
Runnable r = () -> {
    // counter++;  // Compilation error! counter must be final
};

// الحل: استخدام array أو wrapper
int[] counter = {0};
Runnable r2 = () -> {
    counter[0]++;  // يعمل!
};
```

### Instance Variables
يمكن الوصول إليها وتعديلها:
```java
public class Calculator {
    private int value = 0;

    public void process(List<Integer> numbers) {
        numbers.forEach(n -> {
            value += n;  // يمكن تعديل instance variables
        });
    }
}
```

## Method References
عندما Lambda Expression يستدعي method واحدة فقط، يمكن استخدام Method Reference:

```java
// Lambda
list.forEach(item -> System.out.println(item));

// Method Reference
list.forEach(System.out::println);

// أنواع Method References:
// 1. Static method
Function<String, Integer> f1 = s -> Integer.parseInt(s);
Function<String, Integer> f2 = Integer::parseInt;

// 2. Instance method on particular object
String str = "Hello";
Supplier<Integer> s1 = () -> str.length();
Supplier<Integer> s2 = str::length;

// 3. Instance method on arbitrary object
Comparator<String> c1 = (s1, s2) -> s1.compareToIgnoreCase(s2);
Comparator<String> c2 = String::compareToIgnoreCase;

// 4. Constructor
Supplier<List<String>> supplier1 = () -> new ArrayList<>();
Supplier<List<String>> supplier2 = ArrayList::new;
```

## أمثلة متقدمة

### 1. Custom Functional Interface
```java
@FunctionalInterface
interface MathOperation {
    int calculate(int a, int b);
}

public class Calculator {
    public static void main(String[] args) {
        MathOperation add = (a, b) -> a + b;
        MathOperation subtract = (a, b) -> a - b;
        MathOperation multiply = (a, b) -> a * b;
        MathOperation divide = (a, b) -> {
            if (b == 0) {
                throw new ArithmeticException("Division by zero");
            }
            return a / b;
        };

        System.out.println(operate(10, 5, add));       // 15
        System.out.println(operate(10, 5, subtract));  // 5
        System.out.println(operate(10, 5, multiply));  // 50
        System.out.println(operate(10, 5, divide));    // 2
    }

    static int operate(int a, int b, MathOperation operation) {
        return operation.calculate(a, b);
    }
}
```

### 2. Builder Pattern مع Lambda
```java
public class EmailBuilder {
    private String to;
    private String subject;
    private String body;

    public EmailBuilder with(Consumer<EmailBuilder> builderFunction) {
        builderFunction.accept(this);
        return this;
    }

    public EmailBuilder to(String to) {
        this.to = to;
        return this;
    }

    public EmailBuilder subject(String subject) {
        this.subject = subject;
        return this;
    }

    public EmailBuilder body(String body) {
        this.body = body;
        return this;
    }

    public void send() {
        System.out.println("Sending email to: " + to);
    }

    public static void main(String[] args) {
        new EmailBuilder()
            .with(email -> {
                email.to("user@example.com");
                email.subject("Important");
                email.body("Hello World");
            })
            .send();
    }
}
```

### 3. Exception Handling في Lambda
```java
// Lambda لا يمكنها throw checked exceptions مباشرة
// الحل: wrapper method

@FunctionalInterface
interface CheckedFunction<T, R> {
    R apply(T t) throws Exception;
}

public static <T, R> Function<T, R> wrap(CheckedFunction<T, R> checkedFunction) {
    return t -> {
        try {
            return checkedFunction.apply(t);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    };
}

// الاستخدام
List<String> paths = Arrays.asList("file1.txt", "file2.txt");
paths.stream()
    .map(wrap(path -> Files.readAllLines(Paths.get(path))))
    .forEach(System.out::println);
```

### 4. Chaining Lambdas
```java
Function<Integer, Integer> multiplyBy2 = x -> x * 2;
Function<Integer, Integer> add10 = x -> x + 10;

// Chaining
Function<Integer, Integer> multiplyThenAdd = multiplyBy2.andThen(add10);
System.out.println(multiplyThenAdd.apply(5));  // (5 * 2) + 10 = 20

// Compose (عكس andThen)
Function<Integer, Integer> addThenMultiply = multiplyBy2.compose(add10);
System.out.println(addThenMultiply.apply(5));  // (5 + 10) * 2 = 30

// Predicate chaining
Predicate<Integer> isEven = n -> n % 2 == 0;
Predicate<Integer> isPositive = n -> n > 0;

Predicate<Integer> isEvenAndPositive = isEven.and(isPositive);
Predicate<Integer> isEvenOrPositive = isEven.or(isPositive);
Predicate<Integer> isOdd = isEven.negate();
```

## الفروقات مع Anonymous Inner Class

| الخاصية | Lambda Expression | Anonymous Inner Class |
|---------|------------------|---------------------|
| الـ Syntax | مختصر وواضح | طويل ومعقد |
| الـ Performance | أسرع (invokedynamic) | أبطأ (class جديد) |
| this keyword | يشير للـ enclosing class | يشير للـ anonymous class |
| Variable capture | final/effectively final فقط | نفس الشيء |
| الاستخدام | Functional Interface فقط | أي interface أو class |

```java
public class ThisExample {
    private String message = "Outer";

    public void testThis() {
        // Anonymous Inner Class
        Runnable r1 = new Runnable() {
            private String message = "Inner";

            @Override
            public void run() {
                System.out.println(this.message); // "Inner"
            }
        };

        // Lambda
        Runnable r2 = () -> {
            System.out.println(this.message); // "Outer"
        };
    }
}
```

## Best Practices

### 1. Keep it Simple
```java
// جيد
list.forEach(item -> System.out.println(item));

// أفضل (Method Reference)
list.forEach(System.out::println);

// سيء - معقد جداً
list.forEach(item -> {
    String formatted = String.format("Item: %s", item);
    System.out.println(formatted.toUpperCase());
    // ... more code
});
// الأفضل: extract to method
```

### 2. Use Type Inference
```java
// غير ضروري
Comparator<String> comp = (String s1, String s2) -> s1.compareTo(s2);

// أفضل
Comparator<String> comp = (s1, s2) -> s1.compareTo(s2);
```

### 3. Avoid Side Effects
```java
// سيء - side effect
int[] sum = {0};
list.forEach(n -> sum[0] += n);

// جيد - functional approach
int sum = list.stream()
    .reduce(0, Integer::sum);
```

### 4. Extract Complex Lambdas
```java
// سيء - lambda معقدة
employees.stream()
    .filter(e -> {
        return e.getAge() > 25 &&
               e.getSalary() > 50000 &&
               e.getDepartment().equals("IT") &&
               e.getYearsOfExperience() > 3;
    })
    .collect(Collectors.toList());

// جيد - extracted to method
employees.stream()
    .filter(this::isEligibleForPromotion)
    .collect(Collectors.toList());

private boolean isEligibleForPromotion(Employee e) {
    return e.getAge() > 25 &&
           e.getSalary() > 50000 &&
           e.getDepartment().equals("IT") &&
           e.getYearsOfExperience() > 3;
}
```

## الخلاصة

Lambda Expressions جعلت Java أكثر قوة ومرونة:
- كود أقصر وأوضح
- دعم البرمجة الوظيفية
- تكامل ممتاز مع Stream API
- أداء أفضل من Anonymous Inner Classes
- تسهيل العمل مع Collections و Threading

الاستخدام الصحيح للـ Lambda يجعل الكود أكثر قابلية للقراءة والصيانة!