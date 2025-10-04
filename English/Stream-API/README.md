# Stream API - Comprehensive Guide

## Table of Contents
1. [Introduction](#introduction)
2. [What is Stream API?](#what-is-stream-api)
3. [Creating Streams](#creating-streams)
4. [Intermediate Operations](#intermediate-operations)
5. [Terminal Operations](#terminal-operations)
6. [Collectors](#collectors)
7. [Parallel Streams](#parallel-streams)
8. [Practical Examples](#practical-examples)

---

## Introduction

**Stream API** was introduced in Java 8 and allows processing data in a functional programming style.

### Why Stream API?

```java
// Old Way (Imperative)
List<String> names = Arrays.asList("John", "Jane", "Bob", "Alice", "Charlie");
List<String> filteredNames = new ArrayList<>();
for (String name : names) {
    if (name.length() > 4) {
        filteredNames.add(name);
    }
}
Collections.sort(filteredNames);

// New Way (Declarative with Stream)
List<String> filteredNames = names.stream()
    .filter(name -> name.length() > 4)
    .sorted()
    .collect(Collectors.toList());
```

### Benefits:
- ✅ **Less and clearer code**
- ✅ **Functional Programming**
- ✅ **Lazy Evaluation** (not executed until Terminal Operation)
- ✅ **Easy Parallel Processing**
- ✅ **Doesn't modify original source** (Immutable)

---

## What is Stream API?

**Stream** is a sequence of elements that supports sequential and parallel operations.

### Key Characteristics:

1. **Not a Data Structure**: Stream doesn't store data
2. **Functional**: Doesn't modify the source
3. **Lazy**: Intermediate operations aren't executed until Terminal Operation
4. **Possibly Unbounded**: Can be infinite
5. **Consumable**: Can be used only once

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Create Stream
Stream<Integer> stream = numbers.stream();

// Use Stream
stream.forEach(System.out::println);

// ❌ Error - Stream already consumed
// stream.forEach(System.out::println); // IllegalStateException
```

---

## Creating Streams

### 1. From Collection
```java
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();
```

### 2. From Array
```java
String[] array = {"a", "b", "c"};
Stream<String> stream = Arrays.stream(array);

// Or
Stream<String> stream2 = Stream.of("a", "b", "c");
```

### 3. Stream.of()
```java
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);
```

### 4. Stream.generate()
```java
// Infinite Stream
Stream<Double> randomNumbers = Stream.generate(Math::random);

// Use with limit
Stream.generate(Math::random)
    .limit(5)
    .forEach(System.out::println);
```

### 5. Stream.iterate()
```java
// Numbers from 0 to 9
Stream.iterate(0, n -> n + 1)
    .limit(10)
    .forEach(System.out::println);

// Even numbers
Stream.iterate(0, n -> n + 2)
    .limit(10)
    .forEach(System.out::println); // 0, 2, 4, 6, 8, 10, 12, 14, 16, 18
```

### 6. From String
```java
String str = "Hello";
IntStream chars = str.chars();
chars.forEach(c -> System.out.println((char) c));
```

### 7. From Files
```java
// Read file
Stream<String> lines = Files.lines(Paths.get("file.txt"));
lines.forEach(System.out::println);
```

### 8. IntStream, LongStream, DoubleStream
```java
// IntStream
IntStream.range(1, 10)          // 1 to 9
    .forEach(System.out::println);

IntStream.rangeClosed(1, 10)    // 1 to 10
    .forEach(System.out::println);
```

---

## Intermediate Operations

Intermediate operations return a new Stream and are executed lazily.

### 1. filter()
Filter elements based on a condition.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Even numbers
List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
// Result: [2, 4, 6, 8, 10]

// Numbers greater than 5
List<Integer> greaterThan5 = numbers.stream()
    .filter(n -> n > 5)
    .collect(Collectors.toList());
// Result: [6, 7, 8, 9, 10]
```

**Example with Objects:**
```java
class Employee {
    String name;
    int age;
    double salary;
}

List<Employee> employees = getEmployees();

// Employees with salary > 5000
List<Employee> highSalary = employees.stream()
    .filter(emp -> emp.getSalary() > 5000)
    .collect(Collectors.toList());
```

---

### 2. map()
Transform each element to another form.

```java
List<String> names = Arrays.asList("John", "Jane", "Bob");

// Convert to uppercase
List<String> upperNames = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
// Result: ["JOHN", "JANE", "BOB"]

// Length of each name
List<Integer> nameLengths = names.stream()
    .map(String::length)
    .collect(Collectors.toList());
// Result: [4, 4, 3]
```

**Example with Objects:**
```java
// Extract employee names
List<String> employeeNames = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.toList());

// Double the salary
List<Double> doubleSalaries = employees.stream()
    .map(emp -> emp.getSalary() * 2)
    .collect(Collectors.toList());
```

---

### 3. flatMap()
Transform each element to a Stream then flatten into one stream.

```java
List<List<Integer>> numbers = Arrays.asList(
    Arrays.asList(1, 2, 3),
    Arrays.asList(4, 5, 6),
    Arrays.asList(7, 8, 9)
);

// Flatten all lists into one
List<Integer> flatList = numbers.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());
// Result: [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

**Example - Extract all words:**
```java
List<String> sentences = Arrays.asList(
    "Hello World",
    "Java Stream API",
    "Functional Programming"
);

List<String> words = sentences.stream()
    .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
    .collect(Collectors.toList());
// Result: ["Hello", "World", "Java", "Stream", "API", "Functional", "Programming"]
```

---

### 4. distinct()
Remove duplicates.

```java
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 3, 4, 5, 5);

List<Integer> uniqueNumbers = numbers.stream()
    .distinct()
    .collect(Collectors.toList());
// Result: [1, 2, 3, 4, 5]
```

---

### 5. sorted()
Sort elements.

```java
List<Integer> numbers = Arrays.asList(5, 3, 8, 1, 9, 2);

// Ascending order (Natural Order)
List<Integer> sorted = numbers.stream()
    .sorted()
    .collect(Collectors.toList());
// Result: [1, 2, 3, 5, 8, 9]

// Descending order
List<Integer> sortedDesc = numbers.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());
// Result: [9, 8, 5, 3, 2, 1]
```

**Example with Objects:**
```java
// Sort by salary ascending
List<Employee> sortedBySalary = employees.stream()
    .sorted(Comparator.comparing(Employee::getSalary))
    .collect(Collectors.toList());

// Sort by salary descending
List<Employee> sortedBySalaryDesc = employees.stream()
    .sorted(Comparator.comparing(Employee::getSalary).reversed())
    .collect(Collectors.toList());

// Sort by multiple fields
List<Employee> sorted = employees.stream()
    .sorted(Comparator.comparing(Employee::getDepartment)
                      .thenComparing(Employee::getSalary))
    .collect(Collectors.toList());
```

---

### 6. limit()
Take first n elements.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

List<Integer> first5 = numbers.stream()
    .limit(5)
    .collect(Collectors.toList());
// Result: [1, 2, 3, 4, 5]
```

---

### 7. skip()
Skip first n elements.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

List<Integer> skip5 = numbers.stream()
    .skip(5)
    .collect(Collectors.toList());
// Result: [6, 7, 8, 9, 10]
```

**Pagination example:**
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
Perform an operation on each element without modifying Stream (for Debugging).

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

## Terminal Operations

Terminal operations execute the Stream and return a result.

### 1. forEach()
Perform an operation on each element.

```java
List<String> names = Arrays.asList("John", "Jane", "Bob");

names.stream()
    .forEach(System.out::println);

// With Consumer
names.stream()
    .forEach(name -> {
        System.out.println("Name: " + name);
        System.out.println("Length: " + name.length());
    });
```

---

### 2. collect()
Collect results into a Collection.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// To List
List<Integer> list = numbers.stream()
    .collect(Collectors.toList());

// To Set
Set<Integer> set = numbers.stream()
    .collect(Collectors.toSet());

// To Array
Integer[] array = numbers.stream()
    .toArray(Integer[]::new);
```

---

### 3. count()
Count elements.

```java
long count = numbers.stream()
    .filter(n -> n > 5)
    .count();
```

---

### 4. anyMatch(), allMatch(), noneMatch()
Check conditions.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Any even element?
boolean hasEven = numbers.stream()
    .anyMatch(n -> n % 2 == 0);  // true

// All elements even?
boolean allEven = numbers.stream()
    .allMatch(n -> n % 2 == 0);  // false

// No negative elements?
boolean noNegative = numbers.stream()
    .noneMatch(n -> n < 0);      // true
```

---

### 5. findFirst(), findAny()
Find an element.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// First element
Optional<Integer> first = numbers.stream()
    .findFirst();  // Optional[1]

// Any element (useful in Parallel Streams)
Optional<Integer> any = numbers.stream()
    .findAny();

// With filter
Optional<Integer> firstEven = numbers.stream()
    .filter(n -> n % 2 == 0)
    .findFirst();  // Optional[2]
```

---

### 6. min(), max()
Find smallest and largest element.

```java
List<Integer> numbers = Arrays.asList(5, 3, 8, 1, 9, 2);

Optional<Integer> min = numbers.stream()
    .min(Integer::compareTo);  // Optional[1]

Optional<Integer> max = numbers.stream()
    .max(Integer::compareTo);  // Optional[9]

// With Objects
Optional<Employee> highestPaid = employees.stream()
    .max(Comparator.comparing(Employee::getSalary));
```

---

### 7. reduce()
Combine elements into a single value.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Sum of numbers
Optional<Integer> sum = numbers.stream()
    .reduce((a, b) -> a + b);  // Optional[15]

// Or with initial value
int sum2 = numbers.stream()
    .reduce(0, (a, b) -> a + b);  // 15

// Or using Integer::sum
int sum3 = numbers.stream()
    .reduce(0, Integer::sum);  // 15

// Product of numbers
int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);  // 120
```

**Advanced examples:**
```java
// Longest name
Optional<String> longest = names.stream()
    .reduce((s1, s2) -> s1.length() > s2.length() ? s1 : s2);

// Combine Strings
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
// Convert List to Map
Map<Integer, Employee> map = employees.stream()
    .collect(Collectors.toMap(
        Employee::getId,      // Key
        emp -> emp           // Value
    ));

// With transformation
Map<Integer, String> nameMap = employees.stream()
    .collect(Collectors.toMap(
        Employee::getId,
        Employee::getName
    ));
```

---

### 3. joining()
Join Strings.

```java
List<String> names = Arrays.asList("John", "Jane", "Bob");

String joined = names.stream()
    .collect(Collectors.joining());
// Result: "JohnJaneBob"

String joinedWithComma = names.stream()
    .collect(Collectors.joining(", "));
// Result: "John, Jane, Bob"

String joinedWithPrefix = names.stream()
    .collect(Collectors.joining(", ", "[", "]"));
// Result: "[John, Jane, Bob]"
```

---

### 4. groupingBy()
Group elements.

```java
// Group employees by department
Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// Group by salary level (high/low)
Map<String, List<Employee>> bySalaryLevel = employees.stream()
    .collect(Collectors.groupingBy(emp ->
        emp.getSalary() > 5000 ? "High" : "Low"
    ));

// Count employees in each department
Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.counting()
    ));

// Average salary per department
Map<String, Double> avgSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));
```

---

### 5. partitioningBy()
Partition into two groups (true/false).

```java
// Partition employees by salary
Map<Boolean, List<Employee>> partitioned = employees.stream()
    .collect(Collectors.partitioningBy(emp -> emp.getSalary() > 5000));

List<Employee> highSalary = partitioned.get(true);
List<Employee> lowSalary = partitioned.get(false);
```

---

### 6. summarizingDouble/Int/Long()
Comprehensive statistics.

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

### When to use Parallel Stream?
- ✅ Large datasets (> 10,000 elements)
- ✅ Independent operations
- ✅ CPU-intensive operations
- ✅ No Shared Mutable State

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

// Convert Stream to Parallel
long sum3 = numbers.stream()
    .parallel()
    .mapToInt(Integer::intValue)
    .sum();
```

**Example - Process millions of records:**
```java
List<Employee> employees = getMillionsOfEmployees();

// Parallel processing
double avgSalary = employees.parallelStream()
    .mapToDouble(Employee::getSalary)
    .average()
    .orElse(0.0);
```

**Warning - Potential issues:**
```java
// ❌ Wrong - using shared mutable state
List<Integer> result = new ArrayList<>();
numbers.parallelStream()
    .forEach(result::add);  // Not thread-safe!

// ✅ Correct
List<Integer> result = numbers.parallelStream()
    .collect(Collectors.toList());
```

---

## Practical Examples

### Example 1: Employee List Processing

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
    new Employee(1, "John", 30, "IT", 6000),
    new Employee(2, "Jane", 25, "HR", 4000),
    new Employee(3, "Bob", 35, "IT", 7000),
    new Employee(4, "Alice", 28, "Finance", 5000),
    new Employee(5, "Charlie", 32, "IT", 6500)
);

// 1. IT employees with salary > 6000
List<Employee> itHighSalary = employees.stream()
    .filter(emp -> emp.getDepartment().equals("IT"))
    .filter(emp -> emp.getSalary() > 6000)
    .collect(Collectors.toList());

// 2. Average salary of IT employees
double avgSalary = employees.stream()
    .filter(emp -> emp.getDepartment().equals("IT"))
    .mapToDouble(Employee::getSalary)
    .average()
    .orElse(0.0);

// 3. Highest salary
double maxSalary = employees.stream()
    .mapToDouble(Employee::getSalary)
    .max()
    .orElse(0.0);

// 4. All employee names separated by comma
String names = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.joining(", "));

// 5. Count employees in each department
Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.counting()
    ));

// 6. Top 3 salaries
List<Double> top3Salaries = employees.stream()
    .map(Employee::getSalary)
    .sorted(Comparator.reverseOrder())
    .limit(3)
    .collect(Collectors.toList());

// 7. Employee with highest salary
Optional<Employee> highestPaid = employees.stream()
    .max(Comparator.comparing(Employee::getSalary));

// 8. Increase IT employees' salaries by 10%
List<Employee> updatedEmployees = employees.stream()
    .map(emp -> {
        if (emp.getDepartment().equals("IT")) {
            emp.setSalary(emp.getSalary() * 1.1);
        }
        return emp;
    })
    .collect(Collectors.toList());

// 9. Any employee under 25?
boolean hasYoung = employees.stream()
    .anyMatch(emp -> emp.getAge() < 25);

// 10. Total salaries of all employees
double totalSalary = employees.stream()
    .mapToDouble(Employee::getSalary)
    .sum();
```

---

### Example 2: Order Processing

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

// 1. Pending orders
List<Order> pendingOrders = orders.stream()
    .filter(order -> order.getStatus().equals("PENDING"))
    .collect(Collectors.toList());

// 2. Total sales
double totalSales = orders.stream()
    .mapToDouble(Order::getTotalPrice)
    .sum();

// 3. Order count per customer
Map<String, Long> ordersByCustomer = orders.stream()
    .collect(Collectors.groupingBy(
        Order::getCustomerName,
        Collectors.counting()
    ));

// 4. Orders grouped by status
Map<String, List<Order>> ordersByStatus = orders.stream()
    .collect(Collectors.groupingBy(Order::getStatus));

// 5. Largest order (by value)
Optional<Order> largestOrder = orders.stream()
    .max(Comparator.comparing(Order::getTotalPrice));

// 6. Unique products
List<String> uniqueProducts = orders.stream()
    .map(Order::getProduct)
    .distinct()
    .collect(Collectors.toList());

// 7. Average order price
double avgOrderPrice = orders.stream()
    .mapToDouble(Order::getTotalPrice)
    .average()
    .orElse(0.0);

// 8. Orders with value > 1000
List<Order> largeOrders = orders.stream()
    .filter(order -> order.getTotalPrice() > 1000)
    .sorted(Comparator.comparing(Order::getTotalPrice).reversed())
    .collect(Collectors.toList());
```

---

### Example 3: Text Processing

```java
String text = "Java Stream API makes functional programming easy and powerful";

// 1. Word count
long wordCount = Arrays.stream(text.split(" "))
    .count();

// 2. Unique words
List<String> uniqueWords = Arrays.stream(text.split(" "))
    .map(String::toLowerCase)
    .distinct()
    .collect(Collectors.toList());

// 3. Words sorted alphabetically
List<String> sortedWords = Arrays.stream(text.split(" "))
    .sorted()
    .collect(Collectors.toList());

// 4. Words with length > 4
List<String> longWords = Arrays.stream(text.split(" "))
    .filter(word -> word.length() > 4)
    .collect(Collectors.toList());

// 5. Length of each word
Map<String, Integer> wordLengths = Arrays.stream(text.split(" "))
    .collect(Collectors.toMap(
        word -> word,
        String::length,
        (existing, replacement) -> existing
    ));

// 6. Convert to uppercase and join
String upperText = Arrays.stream(text.split(" "))
    .map(String::toUpperCase)
    .collect(Collectors.joining(" "));
```

---

### Example 4: Number Processing

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// 1. Square of each number
List<Integer> squares = numbers.stream()
    .map(n -> n * n)
    .collect(Collectors.toList());

// 2. Sum of even numbers
int sumEven = numbers.stream()
    .filter(n -> n % 2 == 0)
    .mapToInt(Integer::intValue)
    .sum();

// 3. Prime numbers
List<Integer> primes = numbers.stream()
    .filter(n -> isPrime(n))
    .collect(Collectors.toList());

// 4. Comprehensive statistics
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

### 1. Use Method References when possible
```java
// ❌ Less readable
names.stream().forEach(name -> System.out.println(name));

// ✅ Better
names.stream().forEach(System.out::println);
```

### 2. Avoid Side Effects
```java
// ❌ Bad - modifying external state
List<String> result = new ArrayList<>();
names.stream().forEach(name -> result.add(name.toUpperCase()));

// ✅ Good - functional approach
List<String> result = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

### 3. Use Parallel Stream carefully
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

### 4. Close Streams from Files
```java
try (Stream<String> lines = Files.lines(Paths.get("file.txt"))) {
    lines.forEach(System.out::println);
}
```

### 5. Use Optional correctly
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

## Summary

### Stream API Pipeline:
```
Source → Intermediate Operations → Terminal Operation → Result
```

### Main Operations:

**Intermediate (Lazy):**
- filter(), map(), flatMap(), distinct(), sorted(), limit(), skip(), peek()

**Terminal (Eager):**
- forEach(), collect(), count(), anyMatch(), findFirst(), min(), max(), reduce()

### When to use Stream API:
- ✅ Processing Collections
- ✅ Filtering and transforming data
- ✅ Complex data operations
- ✅ Functional Programming style

### When NOT to use Stream API:
- ❌ Very simple operations (for loop is better)
- ❌ Modifying Collection during processing
- ❌ Performance-critical loops (in some cases)

---

**Additional Resources:**
- [Java Stream API Documentation](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)
- [Baeldung - Java 8 Streams](https://www.baeldung.com/java-8-streams)
