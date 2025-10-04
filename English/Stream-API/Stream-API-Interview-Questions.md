# أسئلة المقابلات الشخصية - Stream API

## جدول المحتويات
1. [أسئلة أساسية](#أسئلة-أساسية)
2. [Intermediate Operations](#intermediate-operations)
3. [Terminal Operations](#terminal-operations)
4. [Collectors](#collectors)
5. [Parallel Streams](#parallel-streams)
6. [أسئلة متقدمة](#أسئلة-متقدمة)
7. [أسئلة عملية - Coding](#أسئلة-عملية---coding)

---

## أسئلة أساسية

### 1. ما هو Stream API ومتى تم تقديمه؟

**الإجابة:**
Stream API تم تقديمه في **Java 8** وهو يسمح بمعالجة مجموعات البيانات بطريقة وظيفية (Functional Programming).

**الخصائص الرئيسية:**
- **ليس Data Structure**: لا يخزن البيانات
- **Functional**: لا يعدل المصدر
- **Lazy Evaluation**: العمليات الوسيطة لا تُنفذ حتى Terminal Operation
- **Possibly Unbounded**: يمكن أن يكون لانهائي
- **Consumable**: يُستخدم مرة واحدة فقط

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// إنشاء واستخدام Stream
int sum = numbers.stream()
    .filter(n -> n % 2 == 0)
    .mapToInt(Integer::intValue)
    .sum();
```

---

### 2. ما الفرق بين Collection و Stream؟

**الإجابة:**

| المقارنة | Collection | Stream |
|----------|-----------|--------|
| **التخزين** | يخزن العناصر | لا يخزن العناصر |
| **التعديل** | يمكن التعديل | لا يمكن التعديل |
| **الاستهلاك** | يمكن التكرار عدة مرات | مرة واحدة فقط |
| **Iteration** | External (for loop) | Internal (forEach) |
| **Lazy** | لا | نعم |
| **الهدف** | تخزين البيانات | معالجة البيانات |

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);

// Collection - يمكن التكرار عدة مرات
for (int n : list) { }
for (int n : list) { }  // ✅ ممكن

// Stream - مرة واحدة فقط
Stream<Integer> stream = list.stream();
stream.forEach(System.out::println);
// stream.forEach(System.out::println); // ❌ IllegalStateException
```

---

### 3. ما الفرق بين Intermediate و Terminal Operations؟

**الإجابة:**

**Intermediate Operations:**
- تُرجع Stream جديد
- **Lazy**: لا تُنفذ حتى Terminal Operation
- يمكن ربط عدة عمليات وسيطة

```java
Stream<Integer> stream = numbers.stream()
    .filter(n -> n > 2)    // Intermediate
    .map(n -> n * 2);      // Intermediate
// لم يُنفذ أي شيء بعد!
```

**Terminal Operations:**
- تُرجع نتيجة أو void
- **Eager**: تُنفذ فوراً
- تُنهي Stream Pipeline

```java
List<Integer> result = numbers.stream()
    .filter(n -> n > 2)
    .map(n -> n * 2)
    .collect(Collectors.toList()); // Terminal - ينفذ كل شيء
```

**جدول المقارنة:**

| Intermediate | Terminal |
|--------------|----------|
| filter() | forEach() |
| map() | collect() |
| flatMap() | count() |
| distinct() | reduce() |
| sorted() | min(), max() |
| limit(), skip() | findFirst(), findAny() |
| peek() | anyMatch(), allMatch() |

---

### 4. اشرح Lazy Evaluation في Streams

**الإجابة:**
Lazy Evaluation تعني أن العمليات الوسيطة لا تُنفذ حتى يتم استدعاء عملية نهائية.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

Stream<Integer> stream = numbers.stream()
    .filter(n -> {
        System.out.println("Filter: " + n);
        return n > 2;
    })
    .map(n -> {
        System.out.println("Map: " + n);
        return n * 2;
    });

System.out.println("Stream created, nothing executed yet");

// Terminal operation - الآن تُنفذ جميع العمليات
List<Integer> result = stream.collect(Collectors.toList());
```

**Output:**
```
Stream created, nothing executed yet
Filter: 1
Filter: 2
Filter: 3
Map: 3
Filter: 4
Map: 4
Filter: 5
Map: 5
```

**الفوائد:**
- ✅ تحسين الأداء - تُنفذ العمليات فقط عند الحاجة
- ✅ تجنب المعالجة غير الضرورية
- ✅ إمكانية العمل مع Streams لانهائية

---

### 5. هل يمكن إعادة استخدام Stream؟

**الإجابة:**
**لا**، Stream يُستهلك مرة واحدة فقط.

```java
Stream<Integer> stream = numbers.stream();

stream.forEach(System.out::println); // ✅ المرة الأولى

// stream.forEach(System.out::println); // ❌ IllegalStateException
```

**الحل - إنشاء Stream جديد:**
```java
// إنشاء Stream كل مرة
numbers.stream().forEach(System.out::println);
numbers.stream().filter(n -> n > 2).forEach(System.out::println);

// أو استخدام Supplier
Supplier<Stream<Integer>> streamSupplier = () -> numbers.stream();
streamSupplier.get().forEach(System.out::println);
streamSupplier.get().filter(n -> n > 2).forEach(System.out::println);
```

---

## Intermediate Operations

### 6. ما الفرق بين map() و flatMap()؟

**الإجابة:**

**map()**: تحويل 1:1 (كل عنصر → عنصر واحد)
```java
List<String> words = Arrays.asList("Hello", "World");

List<Integer> lengths = words.stream()
    .map(String::length)
    .collect(Collectors.toList());
// النتيجة: [5, 5]
```

**flatMap()**: تحويل 1:N (كل عنصر → Stream من عناصر)
```java
List<String> words = Arrays.asList("Hello", "World");

List<Character> chars = words.stream()
    .flatMap(word -> word.chars().mapToObj(c -> (char) c))
    .collect(Collectors.toList());
// النتيجة: [H, e, l, l, o, W, o, r, l, d]
```

**مثال توضيحي:**
```java
List<List<Integer>> numbers = Arrays.asList(
    Arrays.asList(1, 2, 3),
    Arrays.asList(4, 5, 6),
    Arrays.asList(7, 8, 9)
);

// مع map - List<List<Integer>> يبقى كما هو
List<List<Integer>> mapped = numbers.stream()
    .map(list -> list)
    .collect(Collectors.toList());
// النتيجة: [[1,2,3], [4,5,6], [7,8,9]]

// مع flatMap - تحويل إلى List<Integer> واحدة
List<Integer> flatMapped = numbers.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());
// النتيجة: [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

---

### 7. ما الفرق بين filter() و peek()؟

**الإجابة:**

**filter()**: تصفية العناصر حسب شرط
```java
List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
```

**peek()**: تنفيذ عملية على كل عنصر بدون تعديل Stream (للـ Debugging)
```java
List<Integer> result = numbers.stream()
    .peek(n -> System.out.println("Original: " + n))
    .map(n -> n * 2)
    .peek(n -> System.out.println("After map: " + n))
    .collect(Collectors.toList());
```

**الفرق الرئيسي:**
- filter(): يُغير عدد العناصر
- peek(): لا يُغير عدد العناصر

---

### 8. كيف تعمل sorted() مع Objects؟

**الإجابة:**

```java
class Employee {
    String name;
    int age;
    double salary;
}

List<Employee> employees = getEmployees();

// 1. ترتيب حسب حقل واحد
List<Employee> sorted = employees.stream()
    .sorted(Comparator.comparing(Employee::getSalary))
    .collect(Collectors.toList());

// 2. ترتيب تنازلي
List<Employee> sortedDesc = employees.stream()
    .sorted(Comparator.comparing(Employee::getSalary).reversed())
    .collect(Collectors.toList());

// 3. ترتيب حسب أكثر من حقل
List<Employee> multiSort = employees.stream()
    .sorted(Comparator.comparing(Employee::getDepartment)
                      .thenComparing(Employee::getSalary)
                      .thenComparing(Employee::getName))
    .collect(Collectors.toList());

// 4. ترتيب مع nulls
List<Employee> sortedWithNulls = employees.stream()
    .sorted(Comparator.comparing(Employee::getName,
                                 Comparator.nullsLast(String::compareTo)))
    .collect(Collectors.toList());
```

---

## Terminal Operations

### 9. ما الفرق بين findFirst() و findAny()؟

**الإجابة:**

**findFirst()**: يُرجع أول عنصر (محدد)
```java
Optional<Integer> first = numbers.stream()
    .filter(n -> n > 5)
    .findFirst();  // دائماً نفس العنصر
```

**findAny()**: يُرجع أي عنصر (غير محدد في Parallel Streams)
```java
Optional<Integer> any = numbers.stream()
    .filter(n -> n > 5)
    .findAny();  // قد يكون أي عنصر
```

**الفرق في Parallel Streams:**
```java
List<Integer> numbers = IntStream.rangeClosed(1, 100)
    .boxed()
    .collect(Collectors.toList());

// findFirst - دائماً يُرجع 6 (أول عنصر > 5)
Optional<Integer> first = numbers.parallelStream()
    .filter(n -> n > 5)
    .findFirst();  // Optional[6]

// findAny - قد يُرجع 6, 7, 8, ... أي عنصر
Optional<Integer> any = numbers.parallelStream()
    .filter(n -> n > 5)
    .findAny();  // Optional[?] غير محدد
```

**متى تستخدم كل منهما:**
- **findFirst()**: عندما تهتم بالترتيب
- **findAny()**: عندما لا تهتم بالترتيب (أداء أفضل في Parallel)

---

### 10. اشرح reduce() مع أمثلة

**الإجابة:**
reduce() تدمج العناصر في قيمة واحدة.

**الأشكال:**

#### 1. reduce(BinaryOperator)
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// مجموع الأرقام
Optional<Integer> sum = numbers.stream()
    .reduce((a, b) -> a + b);  // Optional[15]

// أو
Optional<Integer> sum2 = numbers.stream()
    .reduce(Integer::sum);  // Optional[15]
```

#### 2. reduce(identity, BinaryOperator)
```java
// مع قيمة ابتدائية
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);  // 15

int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);  // 120
```

#### 3. reduce(identity, BiFunction, BinaryOperator)
```java
// للاستخدام مع Parallel Streams
int sum = numbers.parallelStream()
    .reduce(
        0,                           // identity
        (a, b) -> a + b,            // accumulator
        (a, b) -> a + b             // combiner
    );
```

**أمثلة عملية:**

```java
// 1. أطول String
List<String> words = Arrays.asList("Java", "Stream", "API");
Optional<String> longest = words.stream()
    .reduce((s1, s2) -> s1.length() > s2.length() ? s1 : s2);
// Optional[Stream]

// 2. دمج Strings
String combined = words.stream()
    .reduce("", (s1, s2) -> s1 + " " + s2);
// " Java Stream API"

// 3. مجموع رواتب الموظفين
double totalSalary = employees.stream()
    .map(Employee::getSalary)
    .reduce(0.0, Double::sum);
```

---

### 11. ما الفرق بين count() و Collectors.counting()؟

**الإجابة:**

**count()**: Terminal operation تُرجع long
```java
long count = numbers.stream()
    .filter(n -> n > 5)
    .count();  // long
```

**Collectors.counting()**: Collector يُستخدم مع collect()
```java
// عد الموظفين في كل قسم
Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.counting()  // Collector
    ));
```

**الاستخدام:**
- **count()**: عندما تريد عد مباشر
- **Collectors.counting()**: عندما تريد عد ضمن grouping أو عمليات أخرى

---

## Collectors

### 12. اشرح groupingBy() مع أمثلة متقدمة

**الإجابة:**

```java
class Employee {
    String name;
    String department;
    double salary;
    int age;
}

List<Employee> employees = getEmployees();

// 1. تجميع بسيط - حسب القسم
Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// 2. عد الموظفين في كل قسم
Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.counting()
    ));

// 3. متوسط الراتب لكل قسم
Map<String, Double> avgSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));

// 4. مجموع رواتب كل قسم
Map<String, Double> totalSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.summingDouble(Employee::getSalary)
    ));

// 5. أعلى راتب في كل قسم
Map<String, Optional<Employee>> maxSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.maxBy(Comparator.comparing(Employee::getSalary))
    ));

// 6. قائمة أسماء الموظفين في كل قسم
Map<String, List<String>> namesByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.mapping(Employee::getName, Collectors.toList())
    ));

// 7. إحصائيات الرواتب لكل قسم
Map<String, DoubleSummaryStatistics> statsByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.summarizingDouble(Employee::getSalary)
    ));

// 8. تجميع متعدد المستويات - حسب القسم ثم الفئة العمرية
Map<String, Map<String, List<Employee>>> multiLevel = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.groupingBy(emp ->
            emp.getAge() < 30 ? "Junior" :
            emp.getAge() < 40 ? "Mid" : "Senior"
        )
    ));

// 9. تجميع مع TreeMap (مرتب)
Map<String, List<Employee>> sortedMap = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        TreeMap::new,
        Collectors.toList()
    ));
```

---

### 13. ما الفرق بين groupingBy() و partitioningBy()؟

**الإجابة:**

**groupingBy()**: تجميع حسب أي قيمة (يُرجع Map<K, List<V>>)
```java
// تجميع حسب القسم - عدد غير محدود من الأقسام
Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));
// {IT=[...], HR=[...], Finance=[...], ...}
```

**partitioningBy()**: تقسيم إلى مجموعتين فقط (true/false) - (يُرجع Map<Boolean, List<V>>)
```java
// تقسيم حسب الراتب - مجموعتين فقط
Map<Boolean, List<Employee>> partitioned = employees.stream()
    .collect(Collectors.partitioningBy(emp -> emp.getSalary() > 5000));
// {true=[...], false=[...]}

List<Employee> highSalary = partitioned.get(true);
List<Employee> lowSalary = partitioned.get(false);
```

**متى تستخدم كل منهما:**
- **groupingBy()**: عندما يكون لديك أكثر من مجموعتين
- **partitioningBy()**: عندما تريد تقسيم ثنائي (نعم/لا)

**أمثلة إضافية:**

```java
// partitioningBy مع Downstream collector
Map<Boolean, Long> countPartitioned = employees.stream()
    .collect(Collectors.partitioningBy(
        emp -> emp.getSalary() > 5000,
        Collectors.counting()
    ));
// {true=15, false=25}

// partitioningBy متعدد المستويات
Map<Boolean, Map<String, List<Employee>>> multiPartition = employees.stream()
    .collect(Collectors.partitioningBy(
        emp -> emp.getSalary() > 5000,
        Collectors.groupingBy(Employee::getDepartment)
    ));
```

---

### 14. اشرح joining() Collector

**الإجابة:**

```java
List<String> names = Arrays.asList("أحمد", "محمد", "علي", "فاطمة");

// 1. دمج بسيط (بدون فاصل)
String joined = names.stream()
    .collect(Collectors.joining());
// "أحمدمحمدعليفاطمة"

// 2. دمج مع فاصل
String withComma = names.stream()
    .collect(Collectors.joining(", "));
// "أحمد, محمد, علي, فاطمة"

// 3. دمج مع Prefix و Suffix
String withBrackets = names.stream()
    .collect(Collectors.joining(", ", "[", "]"));
// "[أحمد, محمد, علي, فاطمة]"

// 4. دمج بعد تحويل
String upperJoined = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.joining(" | "));
// "أحمد | محمد | علي | فاطمة" (بأحرف كبيرة)
```

**أمثلة عملية:**

```java
// 1. أسماء الموظفين كـ String واحد
String employeeNames = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.joining(", "));

// 2. SQL IN clause
String ids = employees.stream()
    .map(emp -> String.valueOf(emp.getId()))
    .collect(Collectors.joining(", ", "SELECT * FROM employees WHERE id IN (", ")"));
// "SELECT * FROM employees WHERE id IN (1, 2, 3, 4, 5)"

// 3. CSV format
String csv = employees.stream()
    .map(emp -> emp.getId() + "," + emp.getName() + "," + emp.getSalary())
    .collect(Collectors.joining("\n"));
```

---

## Parallel Streams

### 15. متى تستخدم Parallel Streams؟

**الإجابة:**

**متى تستخدم:**
- ✅ مجموعات بيانات كبيرة (> 10,000 عنصر)
- ✅ العمليات مستقلة عن بعضها
- ✅ العمليات تستهلك وقت (CPU-intensive)
- ✅ لا توجد Shared Mutable State

**متى لا تستخدم:**
- ❌ مجموعات صغيرة (Overhead أكبر من الفائدة)
- ❌ عمليات سريعة جداً
- ❌ عمليات تعتمد على الترتيب
- ❌ استخدام Shared Mutable State

**مثال - متى يكون مفيد:**
```java
// مفيد - معالجة ملايين الأرقام
List<Integer> largeList = IntStream.rangeClosed(1, 10_000_000)
    .boxed()
    .collect(Collectors.toList());

// Parallel أسرع هنا
long sum = largeList.parallelStream()
    .filter(n -> isPrime(n))  // عملية CPU-intensive
    .mapToLong(Integer::longValue)
    .sum();
```

**مثال - متى لا يكون مفيد:**
```java
// غير مفيد - قائمة صغيرة
List<Integer> smallList = Arrays.asList(1, 2, 3, 4, 5);

// Sequential أفضل هنا
int sum = smallList.stream()  // لا تستخدم parallel
    .mapToInt(Integer::intValue)
    .sum();
```

---

### 16. ما هي المشاكل المحتملة مع Parallel Streams؟

**الإجابة:**

#### 1. Shared Mutable State
```java
// ❌ خطأ - غير Thread-safe
List<Integer> result = new ArrayList<>();
numbers.parallelStream()
    .forEach(result::add);  // Race condition!

// ✅ صحيح - استخدام Collector
List<Integer> result = numbers.parallelStream()
    .collect(Collectors.toList());
```

#### 2. Order Dependency
```java
// ❌ Parallel قد يُغير الترتيب
List<Integer> result = numbers.parallelStream()
    .collect(Collectors.toList());  // ترتيب غير مضمون

// ✅ إذا كنت تحتاج الترتيب
List<Integer> result = numbers.parallelStream()
    .forEachOrdered(System.out::println);  // يحافظ على الترتيب
```

#### 3. Overhead
```java
// ❌ Overhead أكبر من الفائدة
List<Integer> small = Arrays.asList(1, 2, 3, 4, 5);
small.parallelStream().map(n -> n * 2).collect(Collectors.toList());

// ✅ Sequential أفضل للقوائم الصغيرة
small.stream().map(n -> n * 2).collect(Collectors.toList());
```

#### 4. Wrong Reduce Identity
```java
// ❌ خطأ - identity خاطئ
int result = numbers.parallelStream()
    .reduce(1, (a, b) -> a + b);  // نتيجة خاطئة!

// ✅ صحيح
int result = numbers.parallelStream()
    .reduce(0, (a, b) -> a + b);
```

---

## أسئلة متقدمة

### 17. ما هو Short-Circuiting؟

**الإجابة:**
Short-Circuiting هو إيقاف معالجة Stream بمجرد الحصول على النتيجة المطلوبة.

**عمليات Short-Circuiting:**
- anyMatch(), allMatch(), noneMatch()
- findFirst(), findAny()
- limit()

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// limit() - يتوقف بعد 5 عناصر
List<Integer> first5 = numbers.stream()
    .peek(n -> System.out.println("Processing: " + n))
    .limit(5)
    .collect(Collectors.toList());
// Output: Processing 1, 2, 3, 4, 5 فقط

// findFirst() - يتوقف بعد إيجاد أول عنصر
Optional<Integer> first = numbers.stream()
    .peek(n -> System.out.println("Processing: " + n))
    .filter(n -> n > 5)
    .findFirst();
// Output: Processing 1, 2, 3, 4, 5, 6 فقط

// anyMatch() - يتوقف بمجرد إيجاد match
boolean hasEven = numbers.stream()
    .peek(n -> System.out.println("Processing: " + n))
    .anyMatch(n -> n % 2 == 0);
// Output: Processing 1, 2 فقط
```

**الفوائد:**
- ✅ تحسين الأداء
- ✅ تقليل العمليات
- ✅ يعمل مع Infinite Streams

---

### 18. كيف تتعامل مع Infinite Streams؟

**الإجابة:**

```java
// 1. Stream.generate() - لانهائي
Stream<Double> randomNumbers = Stream.generate(Math::random);

// ❌ لا تفعل هذا - لن ينتهي!
// randomNumbers.forEach(System.out::println);

// ✅ استخدم limit()
Stream.generate(Math::random)
    .limit(10)
    .forEach(System.out::println);

// 2. Stream.iterate() - لانهائي
Stream<Integer> numbers = Stream.iterate(0, n -> n + 1);

// ✅ مع limit
Stream.iterate(0, n -> n + 1)
    .limit(100)
    .forEach(System.out::println);

// Java 9+ مع takeWhile
Stream.iterate(0, n -> n + 1)
    .takeWhile(n -> n < 100)
    .forEach(System.out::println);

// 3. مثال - Fibonacci
Stream.iterate(new int[]{0, 1}, arr -> new int[]{arr[1], arr[0] + arr[1]})
    .limit(10)
    .map(arr -> arr[0])
    .forEach(System.out::println);
// Output: 0, 1, 1, 2, 3, 5, 8, 13, 21, 34
```

---

### 19. ما هو Optional وكيف يُستخدم مع Streams؟

**الإجابة:**

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// 1. findFirst() يُرجع Optional
Optional<Integer> first = numbers.stream()
    .filter(n -> n > 10)
    .findFirst();

// استخدام Optional
if (first.isPresent()) {
    System.out.println(first.get());
}

// ✅ أفضل - استخدام ifPresent
first.ifPresent(System.out::println);

// ✅ مع قيمة افتراضية
int value = first.orElse(0);

// ✅ مع Exception
int value2 = first.orElseThrow(() -> new NoSuchElementException("Not found"));

// 2. max() و min() يُرجعان Optional
Optional<Integer> max = numbers.stream()
    .max(Integer::compareTo);

max.ifPresent(m -> System.out.println("Max: " + m));

// 3. reduce() يُرجع Optional
Optional<Integer> sum = numbers.stream()
    .reduce((a, b) -> a + b);

int total = sum.orElse(0);
```

**Optional Methods:**
```java
Optional<String> opt = Optional.of("Hello");

// map()
Optional<Integer> length = opt.map(String::length);

// flatMap()
Optional<String> upper = opt.flatMap(s -> Optional.of(s.toUpperCase()));

// filter()
Optional<String> filtered = opt.filter(s -> s.length() > 3);

// or() - Java 9+
Optional<String> or = opt.or(() -> Optional.of("Default"));
```

---

### 20. كيف تتعامل مع Exceptions في Streams؟

**الإجابة:**

**المشكلة:**
```java
// ❌ لا يمكن رمي Checked Exception من Lambda
List<String> urls = Arrays.asList("url1", "url2", "url3");
urls.stream()
    .map(url -> new URL(url))  // Compile Error! MalformedURLException
    .collect(Collectors.toList());
```

**الحلول:**

#### 1. Wrapper Method
```java
public static URL createURL(String url) {
    try {
        return new URL(url);
    } catch (MalformedURLException e) {
        throw new RuntimeException(e);
    }
}

// الاستخدام
List<URL> urls = urlStrings.stream()
    .map(MyClass::createURL)
    .collect(Collectors.toList());
```

#### 2. Try-Catch داخل Lambda
```java
List<URL> urls = urlStrings.stream()
    .map(url -> {
        try {
            return new URL(url);
        } catch (MalformedURLException e) {
            throw new RuntimeException(e);
        }
    })
    .collect(Collectors.toList());
```

#### 3. استخدام Optional
```java
public static Optional<URL> tryCreateURL(String url) {
    try {
        return Optional.of(new URL(url));
    } catch (MalformedURLException e) {
        return Optional.empty();
    }
}

List<URL> urls = urlStrings.stream()
    .map(MyClass::tryCreateURL)
    .filter(Optional::isPresent)
    .map(Optional::get)
    .collect(Collectors.toList());

// أو مع flatMap
List<URL> urls2 = urlStrings.stream()
    .map(MyClass::tryCreateURL)
    .flatMap(Optional::stream)  // Java 9+
    .collect(Collectors.toList());
```

---

## أسئلة عملية - Coding

### 21. أوجد ثاني أعلى راتب من قائمة موظفين

**الإجابة:**

```java
class Employee {
    String name;
    double salary;
}

List<Employee> employees = getEmployees();

// الحل
Optional<Double> secondHighest = employees.stream()
    .map(Employee::getSalary)
    .distinct()
    .sorted(Comparator.reverseOrder())
    .skip(1)
    .findFirst();

secondHighest.ifPresent(salary ->
    System.out.println("Second highest salary: " + salary)
);
```

---

### 22. أوجد الموظف الأعلى راتباً في كل قسم

**الإجابة:**

```java
Map<String, Optional<Employee>> highestByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.maxBy(Comparator.comparing(Employee::getSalary))
    ));

// طباعة النتائج
highestByDept.forEach((dept, empOpt) ->
    empOpt.ifPresent(emp ->
        System.out.println(dept + ": " + emp.getName() + " - " + emp.getSalary())
    )
);
```

---

### 23. أوجد الكلمات المكررة في نص

**الإجابة:**

```java
String text = "hello world hello java world";

Map<String, Long> wordCount = Arrays.stream(text.split(" "))
    .collect(Collectors.groupingBy(
        word -> word,
        Collectors.counting()
    ));

// الكلمات المكررة فقط
List<String> duplicates = wordCount.entrySet().stream()
    .filter(entry -> entry.getValue() > 1)
    .map(Map.Entry::getKey)
    .collect(Collectors.toList());

System.out.println(duplicates); // [hello, world]
```

---

### 24. حوّل List<List<Integer>> إلى List<Integer> مع إزالة التكرارات

**الإجابة:**

```java
List<List<Integer>> lists = Arrays.asList(
    Arrays.asList(1, 2, 3),
    Arrays.asList(2, 3, 4),
    Arrays.asList(3, 4, 5)
);

List<Integer> flattened = lists.stream()
    .flatMap(List::stream)
    .distinct()
    .sorted()
    .collect(Collectors.toList());

System.out.println(flattened); // [1, 2, 3, 4, 5]
```

---

### 25. أوجد أطول 3 كلمات في نص

**الإجابة:**

```java
String text = "Java Stream API makes functional programming easy and powerful";

List<String> longest3 = Arrays.stream(text.split(" "))
    .sorted(Comparator.comparing(String::length).reversed())
    .limit(3)
    .collect(Collectors.toList());

System.out.println(longest3); // [programming, functional, powerful]
```

---

### 26. احسب متوسط عمر الموظفين في كل قسم

**الإجابة:**

```java
Map<String, Double> avgAgeByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingInt(Employee::getAge)
    ));

avgAgeByDept.forEach((dept, avg) ->
    System.out.println(dept + ": " + String.format("%.2f", avg))
);
```

---

### 27. أوجد جميع الأرقام الزوجية وضاعفها واجمعها

**الإجابة:**

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

int sum = numbers.stream()
    .filter(n -> n % 2 == 0)
    .map(n -> n * 2)
    .mapToInt(Integer::intValue)
    .sum();

System.out.println(sum); // 60 (4+8+12+16+20)
```

---

### 28. حوّل Map<String, List<Employee>> إلى Map<String, List<String>> (أسماء فقط)

**الإجابة:**

```java
Map<String, List<Employee>> empByDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

Map<String, List<String>> namesByDept = empByDept.entrySet().stream()
    .collect(Collectors.toMap(
        Map.Entry::getKey,
        entry -> entry.getValue().stream()
                      .map(Employee::getName)
                      .collect(Collectors.toList())
    ));

// أو مباشرة
Map<String, List<String>> namesByDept2 = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.mapping(Employee::getName, Collectors.toList())
    ));
```

---

### 29. أوجد الموظفين الذين رواتبهم أعلى من متوسط رواتب قسمهم

**الإجابة:**

```java
// 1. احسب متوسط الراتب لكل قسم
Map<String, Double> avgSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));

// 2. صفي الموظفين
List<Employee> aboveAvg = employees.stream()
    .filter(emp -> emp.getSalary() > avgSalaryByDept.get(emp.getDepartment()))
    .collect(Collectors.toList());
```

---

### 30. أوجد أول 5 أرقام في سلسلة Fibonacci

**الإجابة:**

```java
List<Integer> fibonacci = Stream.iterate(new int[]{0, 1},
        arr -> new int[]{arr[1], arr[0] + arr[1]})
    .limit(5)
    .map(arr -> arr[0])
    .collect(Collectors.toList());

System.out.println(fibonacci); // [0, 1, 1, 2, 3]
```

---

## خاتمة

### أسئلة سريعة للمراجعة:

1. **ما الفرق بين map() و flatMap()?**
   - map: 1:1, flatMap: 1:N

2. **هل Stream يُعدل المصدر؟**
   - لا، Stream لا يُعدل المصدر

3. **متى تُنفذ Intermediate Operations؟**
   - عند استدعاء Terminal Operation (Lazy)

4. **هل يمكن إعادة استخدام Stream؟**
   - لا، Stream يُستخدم مرة واحدة فقط

5. **متى تستخدم Parallel Stream؟**
   - مجموعات كبيرة + عمليات CPU-intensive

6. **ما الفرق بين findFirst() و findAny()?**
   - findFirst: محدد, findAny: غير محدد في Parallel

---

**موارد إضافية:**
- [Java Stream API Documentation](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)
- [Baeldung - Java 8 Streams](https://www.baeldung.com/java-8-streams)
- [Oracle Java Tutorials](https://docs.oracle.com/javase/tutorial/collections/streams/)
