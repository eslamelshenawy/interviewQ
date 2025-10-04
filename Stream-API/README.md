# Stream API - دليل شامل

## جدول المحتويات
1. [مقدمة](#مقدمة)
2. [ما هو Stream API؟](#ما-هو-stream-api)
3. [إنشاء Streams](#إنشاء-streams)
4. [العمليات الوسيطة (Intermediate Operations)](#العمليات-الوسيطة-intermediate-operations)
5. [العمليات النهائية (Terminal Operations)](#العمليات-النهائية-terminal-operations)
6. [Collectors](#collectors)
7. [Parallel Streams](#parallel-streams)
8. [أمثلة عملية](#أمثلة-عملية)

---

## مقدمة

**Stream API** تم تقديمه في Java 8 وهو يسمح بمعالجة البيانات بطريقة وظيفية (Functional Programming).

### لماذا Stream API؟

```java
// الطريقة القديمة (Imperative)
List<String> names = Arrays.asList("أحمد", "محمد", "علي", "فاطمة", "زينب");
List<String> filteredNames = new ArrayList<>();
for (String name : names) {
    if (name.length() > 4) {
        filteredNames.add(name);
    }
}
Collections.sort(filteredNames);

// الطريقة الجديدة (Declarative with Stream)
List<String> filteredNames = names.stream()
    .filter(name -> name.length() > 4)
    .sorted()
    .collect(Collectors.toList());
```

### المميزات:
- ✅ **كود أقل وأوضح**
- ✅ **Functional Programming**
- ✅ **Lazy Evaluation** (لا تُنفذ حتى Terminal Operation)
- ✅ **Parallel Processing** سهل
- ✅ **لا تعدل المصدر الأصلي** (Immutable)

---

## ما هو Stream API؟

**Stream** هو تسلسل من العناصر يدعم عمليات متسلسلة ومتوازية.

### الخصائص الرئيسية:

1. **ليس Data Structure**: Stream لا يخزن البيانات
2. **Functional**: لا يعدل المصدر
3. **Lazy**: العمليات الوسيطة لا تُنفذ حتى Terminal Operation
4. **Possibly Unbounded**: يمكن أن يكون لانهائي
5. **Consumable**: يُستخدم مرة واحدة فقط

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// إنشاء Stream
Stream<Integer> stream = numbers.stream();

// استخدام Stream
stream.forEach(System.out::println);

// ❌ خطأ - Stream تم استهلاكه
// stream.forEach(System.out::println); // IllegalStateException
```

---

## إنشاء Streams

### 1. من Collection
```java
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();
```

### 2. من Array
```java
String[] array = {"a", "b", "c"};
Stream<String> stream = Arrays.stream(array);

// أو
Stream<String> stream2 = Stream.of("a", "b", "c");
```

### 3. Stream.of()
```java
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);
```

### 4. Stream.generate()
```java
// Stream لانهائي
Stream<Double> randomNumbers = Stream.generate(Math::random);

// استخدام مع limit
Stream.generate(Math::random)
    .limit(5)
    .forEach(System.out::println);
```

### 5. Stream.iterate()
```java
// أرقام من 0 إلى 9
Stream.iterate(0, n -> n + 1)
    .limit(10)
    .forEach(System.out::println);

// أرقام زوجية
Stream.iterate(0, n -> n + 2)
    .limit(10)
    .forEach(System.out::println); // 0, 2, 4, 6, 8, 10, 12, 14, 16, 18
```

### 6. من String
```java
String str = "Hello";
IntStream chars = str.chars();
chars.forEach(c -> System.out.println((char) c));
```

### 7. من Files
```java
// قراءة ملف
Stream<String> lines = Files.lines(Paths.get("file.txt"));
lines.forEach(System.out::println);
```

### 8. IntStream, LongStream, DoubleStream
```java
// IntStream
IntStream.range(1, 10)          // 1 إلى 9
    .forEach(System.out::println);

IntStream.rangeClosed(1, 10)    // 1 إلى 10
    .forEach(System.out::println);
```

---

## العمليات الوسيطة (Intermediate Operations)

العمليات الوسيطة تُرجع Stream جديد وتُنفذ بشكل Lazy.

### 1. filter()
تصفية العناصر حسب شرط.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// الأرقام الزوجية
List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
// النتيجة: [2, 4, 6, 8, 10]

// الأرقام الأكبر من 5
List<Integer> greaterThan5 = numbers.stream()
    .filter(n -> n > 5)
    .collect(Collectors.toList());
// النتيجة: [6, 7, 8, 9, 10]
```

**مثال مع Objects:**
```java
class Employee {
    String name;
    int age;
    double salary;
}

List<Employee> employees = getEmployees();

// الموظفون الذين راتبهم أكبر من 5000
List<Employee> highSalary = employees.stream()
    .filter(emp -> emp.getSalary() > 5000)
    .collect(Collectors.toList());
```

---

### 2. map()
تحويل كل عنصر إلى شكل آخر.

```java
List<String> names = Arrays.asList("أحمد", "محمد", "علي");

// تحويل إلى أحرف كبيرة
List<String> upperNames = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
// النتيجة: ["أحمد", "محمد", "علي"] بأحرف كبيرة

// طول كل اسم
List<Integer> nameLengths = names.stream()
    .map(String::length)
    .collect(Collectors.toList());
// النتيجة: [4, 4, 3]
```

**مثال مع Objects:**
```java
// استخراج أسماء الموظفين
List<String> employeeNames = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.toList());

// مضاعفة الراتب
List<Double> doubleSalaries = employees.stream()
    .map(emp -> emp.getSalary() * 2)
    .collect(Collectors.toList());
```

---

### 3. flatMap()
تحويل كل عنصر إلى Stream ثم دمجها في stream واحد.

```java
List<List<Integer>> numbers = Arrays.asList(
    Arrays.asList(1, 2, 3),
    Arrays.asList(4, 5, 6),
    Arrays.asList(7, 8, 9)
);

// دمج جميع القوائم في قائمة واحدة
List<Integer> flatList = numbers.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());
// النتيجة: [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

**مثال - استخراج جميع الكلمات:**
```java
List<String> sentences = Arrays.asList(
    "Hello World",
    "Java Stream API",
    "Functional Programming"
);

List<String> words = sentences.stream()
    .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
    .collect(Collectors.toList());
// النتيجة: ["Hello", "World", "Java", "Stream", "API", "Functional", "Programming"]
```

---

### 4. distinct()
إزالة التكرارات.

```java
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 3, 4, 5, 5);

List<Integer> uniqueNumbers = numbers.stream()
    .distinct()
    .collect(Collectors.toList());
// النتيجة: [1, 2, 3, 4, 5]
```

---

### 5. sorted()
ترتيب العناصر.

```java
List<Integer> numbers = Arrays.asList(5, 3, 8, 1, 9, 2);

// ترتيب تصاعدي (Natural Order)
List<Integer> sorted = numbers.stream()
    .sorted()
    .collect(Collectors.toList());
// النتيجة: [1, 2, 3, 5, 8, 9]

// ترتيب تنازلي
List<Integer> sortedDesc = numbers.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());
// النتيجة: [9, 8, 5, 3, 2, 1]
```

**مثال مع Objects:**
```java
// ترتيب حسب الراتب تصاعدي
List<Employee> sortedBySalary = employees.stream()
    .sorted(Comparator.comparing(Employee::getSalary))
    .collect(Collectors.toList());

// ترتيب حسب الراتب تنازلي
List<Employee> sortedBySalaryDesc = employees.stream()
    .sorted(Comparator.comparing(Employee::getSalary).reversed())
    .collect(Collectors.toList());

// ترتيب حسب أكثر من حقل
List<Employee> sorted = employees.stream()
    .sorted(Comparator.comparing(Employee::getDepartment)
                      .thenComparing(Employee::getSalary))
    .collect(Collectors.toList());
```

---

### 6. limit()
أخذ أول n عناصر.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

List<Integer> first5 = numbers.stream()
    .limit(5)
    .collect(Collectors.toList());
// النتيجة: [1, 2, 3, 4, 5]
```

---

### 7. skip()
تخطي أول n عناصر.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

List<Integer> skip5 = numbers.stream()
    .skip(5)
    .collect(Collectors.toList());
// النتيجة: [6, 7, 8, 9, 10]
```

**Pagination مثال:**
```java
int pageNumber = 2;
int pageSize = 10;

List<Employee> page = employees.stream()
    .skip(pageNumber * pageSize)
    .limit(pageSize)
    .collect(Collectors.toList());
```

---

### 8. peek()
تنفيذ عملية على كل عنصر بدون تعديل Stream (للـ Debugging).

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

List<Integer> result = numbers.stream()
    .peek(n -> System.out.println("Original: " + n))
    .map(n -> n * 2)
    .peek(n -> System.out.println("After map: " + n))
    .filter(n -> n > 5)
    .peek(n -> System.out.println("After filter: " + n))
    .collect(Collectors.toList());
```

---

## العمليات النهائية (Terminal Operations)

العمليات النهائية تُنفذ Stream وتُرجع نتيجة.

### 1. forEach()
تنفيذ عملية على كل عنصر.

```java
List<String> names = Arrays.asList("أحمد", "محمد", "علي");

names.stream()
    .forEach(System.out::println);

// مع Consumer
names.stream()
    .forEach(name -> {
        System.out.println("Name: " + name);
        System.out.println("Length: " + name.length());
    });
```

---

### 2. collect()
جمع النتائج في Collection.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// إلى List
List<Integer> list = numbers.stream()
    .collect(Collectors.toList());

// إلى Set
Set<Integer> set = numbers.stream()
    .collect(Collectors.toSet());

// إلى Array
Integer[] array = numbers.stream()
    .toArray(Integer[]::new);
```

---

### 3. count()
عد العناصر.

```java
long count = numbers.stream()
    .filter(n -> n > 5)
    .count();
```

---

### 4. anyMatch(), allMatch(), noneMatch()
فحص الشروط.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// هل يوجد عنصر زوجي؟
boolean hasEven = numbers.stream()
    .anyMatch(n -> n % 2 == 0);  // true

// هل جميع العناصر زوجية؟
boolean allEven = numbers.stream()
    .allMatch(n -> n % 2 == 0);  // false

// هل لا يوجد عنصر سالب؟
boolean noNegative = numbers.stream()
    .noneMatch(n -> n < 0);      // true
```

---

### 5. findFirst(), findAny()
البحث عن عنصر.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// أول عنصر
Optional<Integer> first = numbers.stream()
    .findFirst();  // Optional[1]

// أي عنصر (مفيد في Parallel Streams)
Optional<Integer> any = numbers.stream()
    .findAny();

// مع filter
Optional<Integer> firstEven = numbers.stream()
    .filter(n -> n % 2 == 0)
    .findFirst();  // Optional[2]
```

---

### 6. min(), max()
إيجاد أصغر وأكبر عنصر.

```java
List<Integer> numbers = Arrays.asList(5, 3, 8, 1, 9, 2);

Optional<Integer> min = numbers.stream()
    .min(Integer::compareTo);  // Optional[1]

Optional<Integer> max = numbers.stream()
    .max(Integer::compareTo);  // Optional[9]

// مع Objects
Optional<Employee> highestPaid = employees.stream()
    .max(Comparator.comparing(Employee::getSalary));
```

---

### 7. reduce()
دمج العناصر في قيمة واحدة.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// مجموع الأرقام
Optional<Integer> sum = numbers.stream()
    .reduce((a, b) -> a + b);  // Optional[15]

// أو مع قيمة ابتدائية
int sum2 = numbers.stream()
    .reduce(0, (a, b) -> a + b);  // 15

// أو باستخدام Integer::sum
int sum3 = numbers.stream()
    .reduce(0, Integer::sum);  // 15

// ضرب الأرقام
int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);  // 120
```

**أمثلة متقدمة:**
```java
// أطول اسم
Optional<String> longest = names.stream()
    .reduce((s1, s2) -> s1.length() > s2.length() ? s1 : s2);

// دمج Strings
String combined = names.stream()
    .reduce("", (s1, s2) -> s1 + ", " + s2);
```

---

## Collectors

### 1. toList(), toSet()
```java
List<Integer> list = stream.collect(Collectors.toList());
Set<Integer> set = stream.collect(Collectors.toSet());
```

---

### 2. toMap()
```java
// تحويل List إلى Map
Map<Integer, Employee> map = employees.stream()
    .collect(Collectors.toMap(
        Employee::getId,      // Key
        emp -> emp           // Value
    ));

// مع transformation
Map<Integer, String> nameMap = employees.stream()
    .collect(Collectors.toMap(
        Employee::getId,
        Employee::getName
    ));
```

---

### 3. joining()
دمج Strings.

```java
List<String> names = Arrays.asList("أحمد", "محمد", "علي");

String joined = names.stream()
    .collect(Collectors.joining());
// النتيجة: "أحمدمحمدعلي"

String joinedWithComma = names.stream()
    .collect(Collectors.joining(", "));
// النتيجة: "أحمد, محمد, علي"

String joinedWithPrefix = names.stream()
    .collect(Collectors.joining(", ", "[", "]"));
// النتيجة: "[أحمد, محمد, علي]"
```

---

### 4. groupingBy()
تجميع العناصر.

```java
// تجميع الموظفين حسب القسم
Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// تجميع حسب الراتب (مرتفع/منخفض)
Map<String, List<Employee>> bySalaryLevel = employees.stream()
    .collect(Collectors.groupingBy(emp ->
        emp.getSalary() > 5000 ? "High" : "Low"
    ));

// عد الموظفين في كل قسم
Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.counting()
    ));

// متوسط الراتب لكل قسم
Map<String, Double> avgSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));
```

---

### 5. partitioningBy()
تقسيم إلى مجموعتين (true/false).

```java
// تقسيم الموظفين حسب الراتب
Map<Boolean, List<Employee>> partitioned = employees.stream()
    .collect(Collectors.partitioningBy(emp -> emp.getSalary() > 5000));

List<Employee> highSalary = partitioned.get(true);
List<Employee> lowSalary = partitioned.get(false);
```

---

### 6. summarizingDouble/Int/Long()
إحصائيات شاملة.

```java
DoubleSummaryStatistics stats = employees.stream()
    .collect(Collectors.summarizingDouble(Employee::getSalary));

System.out.println("Count: " + stats.getCount());
System.out.println("Sum: " + stats.getSum());
System.out.println("Min: " + stats.getMin());
System.out.println("Max: " + stats.getMax());
System.out.println("Average: " + stats.getAverage());
```

---

## Parallel Streams

### متى تستخدم Parallel Stream؟
- ✅ مجموعات بيانات كبيرة (> 10,000 عنصر)
- ✅ العمليات مستقلة عن بعضها
- ✅ العمليات تستهلك وقت (CPU-intensive)

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Sequential Stream
long sum1 = numbers.stream()
    .mapToInt(Integer::intValue)
    .sum();

// Parallel Stream
long sum2 = numbers.parallelStream()
    .mapToInt(Integer::intValue)
    .sum();

// تحويل Stream إلى Parallel
long sum3 = numbers.stream()
    .parallel()
    .mapToInt(Integer::intValue)
    .sum();
```

**مثال - معالجة ملايين السجلات:**
```java
List<Employee> employees = getMillionsOfEmployees();

// معالجة متوازية
double avgSalary = employees.parallelStream()
    .mapToDouble(Employee::getSalary)
    .average()
    .orElse(0.0);
```

**تحذير - المشاكل المحتملة:**
```java
// ❌ خطأ - استخدام shared mutable state
List<Integer> result = new ArrayList<>();
numbers.parallelStream()
    .forEach(result::add);  // غير آمن!

// ✅ صحيح
List<Integer> result = numbers.parallelStream()
    .collect(Collectors.toList());
```

---

## أمثلة عملية

### مثال 1: معالجة قائمة موظفين

```java
class Employee {
    private int id;
    private String name;
    private int age;
    private String department;
    private double salary;

    // Constructor, Getters, Setters
}

List<Employee> employees = Arrays.asList(
    new Employee(1, "أحمد", 30, "IT", 6000),
    new Employee(2, "محمد", 25, "HR", 4000),
    new Employee(3, "علي", 35, "IT", 7000),
    new Employee(4, "فاطمة", 28, "Finance", 5000),
    new Employee(5, "زينب", 32, "IT", 6500)
);

// 1. الموظفون في قسم IT براتب أكبر من 6000
List<Employee> itHighSalary = employees.stream()
    .filter(emp -> emp.getDepartment().equals("IT"))
    .filter(emp -> emp.getSalary() > 6000)
    .collect(Collectors.toList());

// 2. متوسط رواتب موظفي IT
double avgSalary = employees.stream()
    .filter(emp -> emp.getDepartment().equals("IT"))
    .mapToDouble(Employee::getSalary)
    .average()
    .orElse(0.0);

// 3. أعلى راتب
double maxSalary = employees.stream()
    .mapToDouble(Employee::getSalary)
    .max()
    .orElse(0.0);

// 4. أسماء جميع الموظفين مفصولة بفاصلة
String names = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.joining(", "));

// 5. عدد الموظفين في كل قسم
Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.counting()
    ));

// 6. أعلى 3 رواتب
List<Double> top3Salaries = employees.stream()
    .map(Employee::getSalary)
    .sorted(Comparator.reverseOrder())
    .limit(3)
    .collect(Collectors.toList());

// 7. الموظف صاحب أعلى راتب
Optional<Employee> highestPaid = employees.stream()
    .max(Comparator.comparing(Employee::getSalary));

// 8. زيادة رواتب موظفي IT بنسبة 10%
List<Employee> updatedEmployees = employees.stream()
    .map(emp -> {
        if (emp.getDepartment().equals("IT")) {
            emp.setSalary(emp.getSalary() * 1.1);
        }
        return emp;
    })
    .collect(Collectors.toList());

// 9. هل يوجد موظف عمره أقل من 25؟
boolean hasYoung = employees.stream()
    .anyMatch(emp -> emp.getAge() < 25);

// 10. مجموع رواتب جميع الموظفين
double totalSalary = employees.stream()
    .mapToDouble(Employee::getSalary)
    .sum();
```

---

### مثال 2: معالجة الطلبات (Orders)

```java
class Order {
    private int id;
    private String customerName;
    private String product;
    private int quantity;
    private double price;
    private String status; // "PENDING", "SHIPPED", "DELIVERED"

    public double getTotalPrice() {
        return quantity * price;
    }
}

List<Order> orders = getOrders();

// 1. الطلبات المعلقة (PENDING)
List<Order> pendingOrders = orders.stream()
    .filter(order -> order.getStatus().equals("PENDING"))
    .collect(Collectors.toList());

// 2. إجمالي المبيعات
double totalSales = orders.stream()
    .mapToDouble(Order::getTotalPrice)
    .sum();

// 3. عدد الطلبات لكل عميل
Map<String, Long> ordersByCustomer = orders.stream()
    .collect(Collectors.groupingBy(
        Order::getCustomerName,
        Collectors.counting()
    ));

// 4. الطلبات مجمعة حسب الحالة
Map<String, List<Order>> ordersByStatus = orders.stream()
    .collect(Collectors.groupingBy(Order::getStatus));

// 5. أكبر طلب (قيمة)
Optional<Order> largestOrder = orders.stream()
    .max(Comparator.comparing(Order::getTotalPrice));

// 6. المنتجات الفريدة
List<String> uniqueProducts = orders.stream()
    .map(Order::getProduct)
    .distinct()
    .collect(Collectors.toList());

// 7. متوسط سعر الطلب
double avgOrderPrice = orders.stream()
    .mapToDouble(Order::getTotalPrice)
    .average()
    .orElse(0.0);

// 8. الطلبات التي قيمتها أكبر من 1000
List<Order> largeOrders = orders.stream()
    .filter(order -> order.getTotalPrice() > 1000)
    .sorted(Comparator.comparing(Order::getTotalPrice).reversed())
    .collect(Collectors.toList());
```

---

### مثال 3: معالجة النصوص

```java
String text = "Java Stream API makes functional programming easy and powerful";

// 1. عدد الكلمات
long wordCount = Arrays.stream(text.split(" "))
    .count();

// 2. الكلمات الفريدة
List<String> uniqueWords = Arrays.stream(text.split(" "))
    .map(String::toLowerCase)
    .distinct()
    .collect(Collectors.toList());

// 3. الكلمات المرتبة أبجدياً
List<String> sortedWords = Arrays.stream(text.split(" "))
    .sorted()
    .collect(Collectors.toList());

// 4. الكلمات التي طولها أكبر من 4
List<String> longWords = Arrays.stream(text.split(" "))
    .filter(word -> word.length() > 4)
    .collect(Collectors.toList());

// 5. طول كل كلمة
Map<String, Integer> wordLengths = Arrays.stream(text.split(" "))
    .collect(Collectors.toMap(
        word -> word,
        String::length,
        (existing, replacement) -> existing
    ));

// 6. تحويل إلى أحرف كبيرة ودمج
String upperText = Arrays.stream(text.split(" "))
    .map(String::toUpperCase)
    .collect(Collectors.joining(" "));
```

---

### مثال 4: معالجة الأرقام

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// 1. مربع كل رقم
List<Integer> squares = numbers.stream()
    .map(n -> n * n)
    .collect(Collectors.toList());

// 2. مجموع الأرقام الزوجية
int sumEven = numbers.stream()
    .filter(n -> n % 2 == 0)
    .mapToInt(Integer::intValue)
    .sum();

// 3. الأرقام الأولية
List<Integer> primes = numbers.stream()
    .filter(n -> isPrime(n))
    .collect(Collectors.toList());

// 4. إحصائيات شاملة
IntSummaryStatistics stats = numbers.stream()
    .mapToInt(Integer::intValue)
    .summaryStatistics();

System.out.println("Count: " + stats.getCount());
System.out.println("Sum: " + stats.getSum());
System.out.println("Min: " + stats.getMin());
System.out.println("Max: " + stats.getMax());
System.out.println("Average: " + stats.getAverage());
```

---

## Best Practices

### 1. استخدم Method References عندما ممكن
```java
// ❌ Less readable
names.stream().forEach(name -> System.out.println(name));

// ✅ Better
names.stream().forEach(System.out::println);
```

### 2. تجنب Side Effects
```java
// ❌ Bad - modifying external state
List<String> result = new ArrayList<>();
names.stream().forEach(name -> result.add(name.toUpperCase()));

// ✅ Good - functional approach
List<String> result = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

### 3. استخدم Parallel Stream بحذر
```java
// ✅ Good for large datasets
largeList.parallelStream()
    .filter(...)
    .collect(Collectors.toList());

// ❌ Overhead for small datasets
smallList.parallelStream()  // Not worth it
    .filter(...)
    .collect(Collectors.toList());
```

### 4. أغلق Streams من Files
```java
try (Stream<String> lines = Files.lines(Paths.get("file.txt"))) {
    lines.forEach(System.out::println);
}
```

### 5. استخدم Optional بشكل صحيح
```java
// ❌ Bad
Optional<String> name = findName();
if (name.isPresent()) {
    System.out.println(name.get());
}

// ✅ Better
findName().ifPresent(System.out::println);

// ✅ With default value
String name = findName().orElse("Unknown");
```

---

## الخلاصة

### Stream API Pipeline:
```
Source → Intermediate Operations → Terminal Operation → Result
```

### العمليات الرئيسية:

**Intermediate (Lazy):**
- filter(), map(), flatMap(), distinct(), sorted(), limit(), skip(), peek()

**Terminal (Eager):**
- forEach(), collect(), count(), anyMatch(), findFirst(), min(), max(), reduce()

### متى تستخدم Stream API:
- ✅ معالجة Collections
- ✅ تصفية وتحويل البيانات
- ✅ عمليات معقدة على البيانات
- ✅ Functional Programming style

### متى لا تستخدم Stream API:
- ❌ عمليات بسيطة جداً (for loop أفضل)
- ❌ تعديل Collection أثناء المعالجة
- ❌ Performance-critical loops (في بعض الحالات)

---

**موارد إضافية:**
- [Java Stream API Documentation](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)
- [Baeldung - Java 8 Streams](https://www.baeldung.com/java-8-streams)
