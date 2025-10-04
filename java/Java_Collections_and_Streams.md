# دليل شامل لـ Java Collections و Streams

## المحتويات
1. [Collection Framework Overview](#collection-framework-overview)
2. [List Interface](#list-interface)
3. [Set Interface](#set-interface)
4. [Queue Interface](#queue-interface)
5. [Map Interface](#map-interface)
6. [Stream API المتقدمة](#stream-api-المتقدمة)
7. [Collectors](#collectors)
8. [أمثلة عملية متقدمة](#أمثلة-عملية-متقدمة)

---

## Collection Framework Overview

### التسلسل الهرمي للـ Collections

```
Collection (Interface)
│
├── List (Interface)
│   ├── ArrayList
│   ├── LinkedList
│   ├── Vector
│   └── Stack
│
├── Set (Interface)
│   ├── HashSet
│   ├── LinkedHashSet
│   └── TreeSet (SortedSet)
│
└── Queue (Interface)
    ├── PriorityQueue
    ├── ArrayDeque
    └── LinkedList

Map (Interface) - منفصلة عن Collection
├── HashMap
├── LinkedHashMap
├── TreeMap (SortedMap)
├── Hashtable
└── ConcurrentHashMap
```

### الفروقات الأساسية

| Feature | List | Set | Queue | Map |
|---------|------|-----|-------|-----|
| **التكرار** | ✅ يسمح | ❌ لا يسمح | ✅ يسمح | ❌ Keys فريدة |
| **الترتيب** | ✅ مرتب | ❌ غير مرتب* | ✅ FIFO/Priority | ❌ غير مرتب* |
| **Null** | ✅ يسمح | ✅ null واحد فقط | ✅ يسمح* | ✅ يسمح* |
| **Index** | ✅ موجود | ❌ غير موجود | ❌ غير موجود | ❌ غير موجود |

*يعتمد على Implementation

---

## List Interface

### ArrayList

```java
import java.util.*;

// إنشاء ArrayList
List<String> list = new ArrayList<>();
ArrayList<String> arrayList = new ArrayList<>();

// إنشاء بحجم أولي
ArrayList<String> list2 = new ArrayList<>(100);

// إنشاء من Collection أخرى
List<String> list3 = new ArrayList<>(Arrays.asList("A", "B", "C"));

// إضافة عناصر
list.add("Ahmed");
list.add("Mohamed");
list.add("Ali");
list.add(1, "Sara"); // إضافة في index محدد

// الحصول على عنصر
String name = list.get(0); // Ahmed

// تعديل عنصر
list.set(0, "Abdullah");

// حذف عنصر
list.remove(0);              // بالـ index
list.remove("Mohamed");      // بالـ value
list.clear();                // حذف الكل

// البحث
boolean exists = list.contains("Ali");
int index = list.indexOf("Ali");        // -1 إذا لم يوجد
int lastIndex = list.lastIndexOf("Ali");

// الحجم
int size = list.size();
boolean isEmpty = list.isEmpty();

// التحويل إلى Array
String[] array = list.toArray(new String[0]);
Object[] objArray = list.toArray();
```

### LinkedList

```java
import java.util.*;

LinkedList<String> linkedList = new LinkedList<>();

// Methods خاصة بـ LinkedList
linkedList.addFirst("First");
linkedList.addLast("Last");
linkedList.add("Middle");

String first = linkedList.getFirst();
String last = linkedList.getLast();

linkedList.removeFirst();
linkedList.removeLast();

// استخدام كـ Stack
linkedList.push("Top");
String top = linkedList.pop();

// استخدام كـ Queue
linkedList.offer("End");
String head = linkedList.poll();
String peek = linkedList.peek(); // بدون حذف
```

### Vector و Stack

```java
// Vector - thread-safe (بطيء)
Vector<String> vector = new Vector<>();
vector.add("A");
vector.add("B");

// Stack - LIFO
Stack<String> stack = new Stack<>();
stack.push("A");
stack.push("B");
stack.push("C");

String top = stack.pop();    // C
String peek = stack.peek();  // B
int position = stack.search("A"); // 2 (من الأعلى)
boolean empty = stack.empty();
```

### مقارنة List Implementations

```java
// ArrayList vs LinkedList
public class ListComparison {
    public static void main(String[] args) {
        int n = 100000;

        // ArrayList - سريع في get/set
        List<Integer> arrayList = new ArrayList<>();
        long start = System.currentTimeMillis();
        for (int i = 0; i < n; i++) {
            arrayList.add(i);
        }
        long end = System.currentTimeMillis();
        System.out.println("ArrayList add: " + (end - start) + "ms");

        start = System.currentTimeMillis();
        for (int i = 0; i < n; i++) {
            arrayList.get(i);
        }
        end = System.currentTimeMillis();
        System.out.println("ArrayList get: " + (end - start) + "ms");

        // LinkedList - سريع في add/remove من البداية
        List<Integer> linkedList = new LinkedList<>();
        start = System.currentTimeMillis();
        for (int i = 0; i < n; i++) {
            linkedList.add(0, i); // إضافة في البداية
        }
        end = System.currentTimeMillis();
        System.out.println("LinkedList add at start: " + (end - start) + "ms");
    }
}
```

### List Operations

```java
List<Integer> numbers = new ArrayList<>(Arrays.asList(5, 2, 8, 1, 9, 3, 7, 4, 6));

// الترتيب
Collections.sort(numbers);                           // تصاعدي
Collections.sort(numbers, Collections.reverseOrder()); // تنازلي
Collections.sort(numbers, Comparator.reverseOrder()); // تنازلي

// البحث الثنائي (Binary Search) - يجب أن تكون مرتبة
Collections.sort(numbers);
int index = Collections.binarySearch(numbers, 7);

// القلب
Collections.reverse(numbers);

// الخلط
Collections.shuffle(numbers);

// التدوير
Collections.rotate(numbers, 2); // تدوير يمين بمقدار 2

// الاستبدال
Collections.replaceAll(numbers, 5, 50); // استبدال 5 بـ 50

// النسخ
List<Integer> copy = new ArrayList<>(numbers);
List<Integer> copy2 = new ArrayList<>();
Collections.copy(copy2, numbers); // يجب أن يكون copy2 بنفس الحجم

// الملء
Collections.fill(numbers, 0); // ملء بـ 0

// Min/Max
int min = Collections.min(numbers);
int max = Collections.max(numbers);

// التكرار
int frequency = Collections.frequency(numbers, 5);

// Disjoint (لا توجد عناصر مشتركة)
boolean disjoint = Collections.disjoint(
    Arrays.asList(1, 2, 3),
    Arrays.asList(4, 5, 6)
); // true
```

---

## Set Interface

### HashSet

```java
import java.util.*;

// إنشاء HashSet
Set<String> set = new HashSet<>();

// إضافة عناصر
set.add("Ahmed");
set.add("Mohamed");
set.add("Ali");
set.add("Ahmed"); // لن تضاف (مكررة)

System.out.println(set.size()); // 3

// البحث
boolean exists = set.contains("Ali"); // true

// الحذف
set.remove("Mohamed");

// التكرار
for (String name : set) {
    System.out.println(name);
}

// العمليات على المجموعات
Set<Integer> set1 = new HashSet<>(Arrays.asList(1, 2, 3, 4, 5));
Set<Integer> set2 = new HashSet<>(Arrays.asList(4, 5, 6, 7, 8));

// Union (الاتحاد)
Set<Integer> union = new HashSet<>(set1);
union.addAll(set2);
System.out.println("Union: " + union); // [1, 2, 3, 4, 5, 6, 7, 8]

// Intersection (التقاطع)
Set<Integer> intersection = new HashSet<>(set1);
intersection.retainAll(set2);
System.out.println("Intersection: " + intersection); // [4, 5]

// Difference (الفرق)
Set<Integer> difference = new HashSet<>(set1);
difference.removeAll(set2);
System.out.println("Difference: " + difference); // [1, 2, 3]

// Symmetric Difference (الفرق المتماثل)
Set<Integer> symDiff = new HashSet<>(set1);
symDiff.addAll(set2);
Set<Integer> temp = new HashSet<>(set1);
temp.retainAll(set2);
symDiff.removeAll(temp);
System.out.println("Symmetric Difference: " + symDiff); // [1, 2, 3, 6, 7, 8]
```

### LinkedHashSet

```java
// يحافظ على ترتيب الإدخال
Set<String> linkedSet = new LinkedHashSet<>();
linkedSet.add("C");
linkedSet.add("A");
linkedSet.add("B");

System.out.println(linkedSet); // [C, A, B] - بنفس الترتيب
```

### TreeSet

```java
// مرتب تلقائياً (Sorted)
Set<Integer> treeSet = new TreeSet<>();
treeSet.add(5);
treeSet.add(2);
treeSet.add(8);
treeSet.add(1);

System.out.println(treeSet); // [1, 2, 5, 8]

// TreeSet مع Comparator
Set<String> nameSet = new TreeSet<>(Comparator.reverseOrder());
nameSet.add("Ahmed");
nameSet.add("Ziad");
nameSet.add("Mohamed");
System.out.println(nameSet); // [Ziad, Mohamed, Ahmed]

// SortedSet methods
TreeSet<Integer> numbers = new TreeSet<>(Arrays.asList(1, 3, 5, 7, 9));

int first = numbers.first();  // 1
int last = numbers.last();    // 9

SortedSet<Integer> headSet = numbers.headSet(5);  // [1, 3] - أقل من 5
SortedSet<Integer> tailSet = numbers.tailSet(5);  // [5, 7, 9] - >= 5
SortedSet<Integer> subSet = numbers.subSet(3, 7); // [3, 5] - من 3 إلى 7

// NavigableSet methods (Java 6+)
Integer lower = numbers.lower(5);    // 3 - أقل عنصر < 5
Integer floor = numbers.floor(5);    // 5 - أقل عنصر <= 5
Integer ceiling = numbers.ceiling(5); // 5 - أكبر عنصر >= 5
Integer higher = numbers.higher(5);   // 7 - أكبر عنصر > 5

Integer pollFirst = numbers.pollFirst();  // حذف وإرجاع الأول
Integer pollLast = numbers.pollLast();    // حذف وإرجاع الأخير

NavigableSet<Integer> descendingSet = numbers.descendingSet(); // عكس الترتيب
```

### مقارنة Set Implementations

| Feature | HashSet | LinkedHashSet | TreeSet |
|---------|---------|---------------|---------|
| **الترتيب** | ❌ عشوائي | ✅ ترتيب الإدخال | ✅ مرتب طبيعياً |
| **Performance** | O(1) | O(1) | O(log n) |
| **Null** | ✅ واحد فقط | ✅ واحد فقط | ❌ لا يسمح |
| **الاستخدام** | أسرع | ترتيب + سرعة | ترتيب تلقائي |

---

## Queue Interface

### PriorityQueue

```java
import java.util.*;

// أقل عنصر له الأولوية (Min Heap)
Queue<Integer> minHeap = new PriorityQueue<>();
minHeap.offer(5);
minHeap.offer(2);
minHeap.offer(8);
minHeap.offer(1);

System.out.println(minHeap.poll()); // 1
System.out.println(minHeap.poll()); // 2

// أكبر عنصر له الأولوية (Max Heap)
Queue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.offer(5);
maxHeap.offer(2);
maxHeap.offer(8);
maxHeap.offer(1);

System.out.println(maxHeap.poll()); // 8
System.out.println(maxHeap.poll()); // 5

// PriorityQueue مع Objects
class Task {
    String name;
    int priority;

    public Task(String name, int priority) {
        this.name = name;
        this.priority = priority;
    }

    public String getName() { return name; }
    public int getPriority() { return priority; }
}

Queue<Task> taskQueue = new PriorityQueue<>(
    Comparator.comparingInt(Task::getPriority)
);

taskQueue.offer(new Task("Low Priority", 3));
taskQueue.offer(new Task("High Priority", 1));
taskQueue.offer(new Task("Medium Priority", 2));

while (!taskQueue.isEmpty()) {
    Task task = taskQueue.poll();
    System.out.println(task.getName() + " - Priority: " + task.getPriority());
}
// Output:
// High Priority - Priority: 1
// Medium Priority - Priority: 2
// Low Priority - Priority: 3
```

### ArrayDeque

```java
// Deque = Double-Ended Queue
Deque<String> deque = new ArrayDeque<>();

// إضافة من البداية
deque.addFirst("First");
deque.offerFirst("New First");

// إضافة من النهاية
deque.addLast("Last");
deque.offerLast("New Last");

// الحصول بدون حذف
String first = deque.peekFirst();
String last = deque.peekLast();

// الحصول مع الحذف
String polledFirst = deque.pollFirst();
String polledLast = deque.pollLast();

// استخدام كـ Stack (أفضل من Stack)
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);
stack.push(2);
stack.push(3);
System.out.println(stack.pop()); // 3

// استخدام كـ Queue
Deque<Integer> queue = new ArrayDeque<>();
queue.offer(1);
queue.offer(2);
queue.offer(3);
System.out.println(queue.poll()); // 1
```

### Queue Methods Summary

```java
Queue<String> queue = new LinkedList<>();

// إضافة
queue.add("A");      // يرمي exception إذا فشل
queue.offer("B");    // يرجع false إذا فشل

// الحصول بدون حذف
String peek1 = queue.element(); // يرمي exception إذا فارغة
String peek2 = queue.peek();    // يرجع null إذا فارغة

// الحصول مع حذف
String poll1 = queue.remove();  // يرمي exception إذا فارغة
String poll2 = queue.poll();    // يرجع null إذا فارغة
```

---

## Map Interface

### HashMap

```java
import java.util.*;

// إنشاء HashMap
Map<String, Integer> map = new HashMap<>();

// إضافة عناصر
map.put("Ahmed", 30);
map.put("Mohamed", 25);
map.put("Ali", 35);
map.put("Sara", 28);

// الحصول على قيمة
Integer age = map.get("Ahmed"); // 30
Integer age2 = map.get("Ziad");  // null

// getOrDefault
Integer age3 = map.getOrDefault("Ziad", 0); // 0

// البحث
boolean hasKey = map.containsKey("Ali");     // true
boolean hasValue = map.containsValue(30);    // true

// الحذف
map.remove("Mohamed");
map.remove("Ali", 35);  // حذف فقط إذا كانت القيمة = 35

// الحجم
int size = map.size();
boolean isEmpty = map.isEmpty();

// putIfAbsent - إضافة فقط إذا لم يكن موجود
map.putIfAbsent("Ziad", 40);
map.putIfAbsent("Ahmed", 50); // لن يتم التحديث (Ahmed موجود)

// replace
map.replace("Ahmed", 31);
map.replace("Sara", 28, 29); // تحديث فقط إذا كانت القيمة الحالية = 28

// compute methods
map.compute("Ahmed", (key, value) -> value + 1);
map.computeIfPresent("Mohamed", (key, value) -> value + 1);
map.computeIfAbsent("Fatma", key -> 26);

// merge
map.merge("Ahmed", 5, (oldValue, newValue) -> oldValue + newValue);

// التكرار
// 1. entrySet
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}

// 2. keySet
for (String key : map.keySet()) {
    System.out.println(key + ": " + map.get(key));
}

// 3. values
for (Integer value : map.values()) {
    System.out.println(value);
}

// 4. forEach (Java 8)
map.forEach((key, value) ->
    System.out.println(key + ": " + value)
);
```

### LinkedHashMap

```java
// يحافظ على ترتيب الإدخال
Map<String, Integer> linkedMap = new LinkedHashMap<>();
linkedMap.put("C", 3);
linkedMap.put("A", 1);
linkedMap.put("B", 2);

linkedMap.forEach((k, v) -> System.out.println(k + ": " + v));
// Output: C: 3, A: 1, B: 2

// LRU Cache باستخدام LinkedHashMap
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    public LRUCache(int capacity) {
        super(capacity, 0.75f, true); // true = access-order
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}

LRUCache<String, String> cache = new LRUCache<>(3);
cache.put("1", "One");
cache.put("2", "Two");
cache.put("3", "Three");
cache.get("1"); // Access
cache.put("4", "Four"); // "2" سيتم حذفه (الأقدم)
System.out.println(cache); // {3=Three, 1=One, 4=Four}
```

### TreeMap

```java
// مرتب حسب الـ Keys
Map<String, Integer> treeMap = new TreeMap<>();
treeMap.put("Ziad", 40);
treeMap.put("Ahmed", 30);
treeMap.put("Mohamed", 25);

treeMap.forEach((k, v) -> System.out.println(k + ": " + v));
// Output: Ahmed: 30, Mohamed: 25, Ziad: 40 (مرتب أبجدياً)

// مع Comparator
Map<String, Integer> reverseMap = new TreeMap<>(Comparator.reverseOrder());
reverseMap.put("A", 1);
reverseMap.put("C", 3);
reverseMap.put("B", 2);
System.out.println(reverseMap); // {C=3, B=2, A=1}

// NavigableMap methods
TreeMap<Integer, String> numbers = new TreeMap<>();
numbers.put(1, "One");
numbers.put(3, "Three");
numbers.put(5, "Five");
numbers.put(7, "Seven");

Map.Entry<Integer, String> firstEntry = numbers.firstEntry();  // 1=One
Map.Entry<Integer, String> lastEntry = numbers.lastEntry();    // 7=Seven

Integer lowerKey = numbers.lowerKey(5);   // 3
Integer floorKey = numbers.floorKey(5);   // 5
Integer ceilingKey = numbers.ceilingKey(5); // 5
Integer higherKey = numbers.higherKey(5);   // 7

NavigableMap<Integer, String> headMap = numbers.headMap(5, true); // <= 5
NavigableMap<Integer, String> tailMap = numbers.tailMap(5, false); // > 5
NavigableMap<Integer, String> subMap = numbers.subMap(3, true, 7, false); // [3, 7)
```

### Hashtable vs HashMap vs ConcurrentHashMap

```java
// Hashtable - thread-safe (قديم وبطيء)
Hashtable<String, Integer> hashtable = new Hashtable<>();
hashtable.put("A", 1);
// hashtable.put(null, 1); // ❌ لا يسمح بـ null

// HashMap - ليس thread-safe (سريع)
HashMap<String, Integer> hashMap = new HashMap<>();
hashMap.put("A", 1);
hashMap.put(null, 2);  // ✅ يسمح بـ null key واحد
hashMap.put("B", null); // ✅ يسمح بـ null values

// ConcurrentHashMap - thread-safe (أسرع من Hashtable)
ConcurrentHashMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();
concurrentMap.put("A", 1);
// concurrentMap.put(null, 1); // ❌ لا يسمح بـ null

// استخدام ConcurrentHashMap
concurrentMap.putIfAbsent("B", 2);
concurrentMap.computeIfPresent("A", (k, v) -> v + 1);
concurrentMap.merge("A", 5, Integer::sum);
```

---

## Stream API المتقدمة

### إنشاء Streams

```java
import java.util.stream.*;

// من Collection
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream1 = list.stream();
Stream<String> parallelStream = list.parallelStream();

// من Array
String[] array = {"a", "b", "c"};
Stream<String> stream2 = Arrays.stream(array);
Stream<String> stream3 = Arrays.stream(array, 0, 2); // من index 0 إلى 2

// Stream.of()
Stream<String> stream4 = Stream.of("a", "b", "c");
Stream<Integer> stream5 = Stream.of(1, 2, 3, 4, 5);

// Stream.builder()
Stream<String> stream6 = Stream.<String>builder()
    .add("a")
    .add("b")
    .add("c")
    .build();

// Stream.iterate() - لانهائي
Stream<Integer> infiniteStream = Stream.iterate(0, n -> n + 2);
Stream<Integer> limitedStream = Stream.iterate(0, n -> n + 2)
    .limit(10); // [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

// Java 9+ - iterate مع predicate
Stream<Integer> stream7 = Stream.iterate(0, n -> n < 20, n -> n + 2);

// Stream.generate() - لانهائي
Stream<Double> randomStream = Stream.generate(Math::random)
    .limit(5);

Stream<String> constantStream = Stream.generate(() -> "Hello")
    .limit(3);

// IntStream, LongStream, DoubleStream
IntStream intStream = IntStream.range(1, 10);        // 1 إلى 9
IntStream intStream2 = IntStream.rangeClosed(1, 10); // 1 إلى 10
IntStream intStream3 = IntStream.of(1, 2, 3, 4, 5);

LongStream longStream = LongStream.range(1L, 100L);
DoubleStream doubleStream = DoubleStream.of(1.1, 2.2, 3.3);

// من String
IntStream chars = "Hello".chars();
Stream<String> lines = "Line1\nLine2\nLine3".lines();

// من File
try (Stream<String> lines = Files.lines(Paths.get("file.txt"))) {
    lines.forEach(System.out::println);
} catch (IOException e) {
    e.printStackTrace();
}

// من Pattern
Stream<String> words = Pattern.compile("\\s+")
    .splitAsStream("Hello World Java Stream");

// Empty Stream
Stream<String> emptyStream = Stream.empty();
```

### Intermediate Operations المتقدمة

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// filter
List<Integer> even = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

// map
List<Integer> squared = numbers.stream()
    .map(n -> n * n)
    .collect(Collectors.toList());

// mapToInt, mapToLong, mapToDouble
int sum = numbers.stream()
    .mapToInt(Integer::intValue)
    .sum();

double average = numbers.stream()
    .mapToDouble(Integer::doubleValue)
    .average()
    .orElse(0.0);

// flatMap
List<List<Integer>> listOfLists = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4),
    Arrays.asList(5, 6)
);

List<Integer> flattened = listOfLists.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());

// flatMap مع String
List<String> sentences = Arrays.asList(
    "Hello World",
    "Java Stream API",
    "Functional Programming"
);

List<String> words = sentences.stream()
    .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
    .collect(Collectors.toList());

// distinct
List<Integer> unique = Arrays.asList(1, 2, 2, 3, 3, 3, 4, 4, 4, 4).stream()
    .distinct()
    .collect(Collectors.toList());

// sorted
List<String> names = Arrays.asList("Ziad", "Ahmed", "Mohamed", "Ali");

List<String> sortedAsc = names.stream()
    .sorted()
    .collect(Collectors.toList());

List<String> sortedDesc = names.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());

List<String> sortedByLength = names.stream()
    .sorted(Comparator.comparingInt(String::length))
    .collect(Collectors.toList());

// peek - للـ debugging
List<Integer> result = numbers.stream()
    .peek(n -> System.out.println("Before filter: " + n))
    .filter(n -> n % 2 == 0)
    .peek(n -> System.out.println("After filter: " + n))
    .map(n -> n * n)
    .peek(n -> System.out.println("After map: " + n))
    .collect(Collectors.toList());

// limit & skip
List<Integer> first5 = numbers.stream()
    .limit(5)
    .collect(Collectors.toList()); // [1, 2, 3, 4, 5]

List<Integer> skip5 = numbers.stream()
    .skip(5)
    .collect(Collectors.toList()); // [6, 7, 8, 9, 10]

// Pagination
int page = 2;
int pageSize = 3;
List<Integer> pageData = numbers.stream()
    .skip((page - 1) * pageSize)
    .limit(pageSize)
    .collect(Collectors.toList());

// takeWhile & dropWhile (Java 9+)
List<Integer> takeWhile = numbers.stream()
    .takeWhile(n -> n < 5)
    .collect(Collectors.toList()); // [1, 2, 3, 4]

List<Integer> dropWhile = numbers.stream()
    .dropWhile(n -> n < 5)
    .collect(Collectors.toList()); // [5, 6, 7, 8, 9, 10]
```

### Terminal Operations المتقدمة

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// forEach & forEachOrdered
numbers.stream().forEach(System.out::println);
numbers.parallelStream().forEachOrdered(System.out::println); // ترتيب محفوظ

// collect
List<Integer> list = numbers.stream().collect(Collectors.toList());
Set<Integer> set = numbers.stream().collect(Collectors.toSet());

// toArray
Integer[] array = numbers.stream().toArray(Integer[]::new);
Object[] objArray = numbers.stream().toArray();

// reduce
// 1. مجموع الأرقام
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);

int sum2 = numbers.stream()
    .reduce(0, Integer::sum);

// 2. ضرب الأرقام
int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);

// 3. إيجاد الأكبر
Optional<Integer> max = numbers.stream()
    .reduce((a, b) -> a > b ? a : b);

Optional<Integer> max2 = numbers.stream()
    .reduce(Integer::max);

// 4. دمج Strings
String joined = Arrays.asList("a", "b", "c").stream()
    .reduce("", (a, b) -> a + b);

// count
long count = numbers.stream()
    .filter(n -> n % 2 == 0)
    .count();

// min & max
Optional<Integer> min = numbers.stream().min(Integer::compareTo);
Optional<Integer> max3 = numbers.stream().max(Integer::compareTo);

// findFirst & findAny
Optional<Integer> first = numbers.stream()
    .filter(n -> n > 5)
    .findFirst(); // 6

Optional<Integer> any = numbers.stream()
    .filter(n -> n > 5)
    .findAny(); // 6 (أو أي رقم آخر في parallel)

// anyMatch, allMatch, noneMatch
boolean hasEven = numbers.stream()
    .anyMatch(n -> n % 2 == 0); // true

boolean allPositive = numbers.stream()
    .allMatch(n -> n > 0); // true

boolean noneNegative = numbers.stream()
    .noneMatch(n -> n < 0); // true

// summaryStatistics
IntSummaryStatistics stats = numbers.stream()
    .mapToInt(Integer::intValue)
    .summaryStatistics();

System.out.println("Count: " + stats.getCount());
System.out.println("Sum: " + stats.getSum());
System.out.println("Min: " + stats.getMin());
System.out.println("Max: " + stats.getMax());
System.out.println("Average: " + stats.getAverage());
```

### Primitive Streams

```java
// IntStream
IntStream intStream = IntStream.range(1, 11); // 1 to 10

int sum = intStream.sum();
OptionalDouble avg = IntStream.range(1, 11).average();
OptionalInt max = IntStream.range(1, 11).max();
OptionalInt min = IntStream.range(1, 11).min();

// تحويل إلى Stream<Integer>
Stream<Integer> boxed = IntStream.range(1, 11).boxed();

// LongStream
long factorial = LongStream.rangeClosed(1, 10)
    .reduce(1, (a, b) -> a * b);

// DoubleStream
double[] values = {1.5, 2.5, 3.5, 4.5};
double average = DoubleStream.of(values).average().orElse(0.0);

// mapToObj
List<String> strings = IntStream.range(1, 6)
    .mapToObj(i -> "Number: " + i)
    .collect(Collectors.toList());
```

---

## Collectors

### Basic Collectors

```java
List<String> names = Arrays.asList("Ahmed", "Mohamed", "Ali", "Sara", "Fatma");

// toList
List<String> list = names.stream()
    .collect(Collectors.toList());

// toSet
Set<String> set = names.stream()
    .collect(Collectors.toSet());

// toCollection
LinkedList<String> linkedList = names.stream()
    .collect(Collectors.toCollection(LinkedList::new));

TreeSet<String> treeSet = names.stream()
    .collect(Collectors.toCollection(TreeSet::new));

// toMap
Map<String, Integer> nameLength = names.stream()
    .collect(Collectors.toMap(
        name -> name,           // key
        name -> name.length()   // value
    ));

// toMap مع duplicate keys
List<String> duplicates = Arrays.asList("A", "B", "A", "C", "B");
Map<String, Integer> countMap = duplicates.stream()
    .collect(Collectors.toMap(
        s -> s,
        s -> 1,
        (existing, replacement) -> existing + replacement
    ));

// toMap مع TreeMap
Map<String, Integer> sortedMap = names.stream()
    .collect(Collectors.toMap(
        name -> name,
        name -> name.length(),
        (v1, v2) -> v1,
        TreeMap::new
    ));
```

### Grouping

```java
class Person {
    String name;
    int age;
    String city;
    double salary;

    public Person(String name, int age, String city, double salary) {
        this.name = name;
        this.age = age;
        this.city = city;
        this.salary = salary;
    }

    public String getName() { return name; }
    public int getAge() { return age; }
    public String getCity() { return city; }
    public double getSalary() { return salary; }
}

List<Person> people = Arrays.asList(
    new Person("Ahmed", 30, "Cairo", 5000),
    new Person("Mohamed", 25, "Alex", 4000),
    new Person("Ali", 35, "Cairo", 6000),
    new Person("Sara", 28, "Alex", 4500),
    new Person("Fatma", 32, "Cairo", 5500)
);

// groupingBy - تجميع بسيط
Map<String, List<Person>> byCity = people.stream()
    .collect(Collectors.groupingBy(Person::getCity));

System.out.println(byCity);
// {Cairo=[Ahmed, Ali, Fatma], Alex=[Mohamed, Sara]}

// groupingBy مع counting
Map<String, Long> countByCity = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity,
        Collectors.counting()
    ));

// groupingBy مع summingDouble
Map<String, Double> totalSalaryByCity = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity,
        Collectors.summingDouble(Person::getSalary)
    ));

// groupingBy مع averagingDouble
Map<String, Double> avgSalaryByCity = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity,
        Collectors.averagingDouble(Person::getSalary)
    ));

// groupingBy مع mapping
Map<String, List<String>> namesByCity = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity,
        Collectors.mapping(Person::getName, Collectors.toList())
    ));

// groupingBy مع maxBy
Map<String, Optional<Person>> oldestByCity = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity,
        Collectors.maxBy(Comparator.comparingInt(Person::getAge))
    ));

// Multi-level grouping
Map<String, Map<String, List<Person>>> byCityAndAgeGroup = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity,
        Collectors.groupingBy(p -> {
            if (p.getAge() < 30) return "20-29";
            else if (p.getAge() < 35) return "30-34";
            else return "35+";
        })
    ));

// groupingBy مع TreeMap (مرتب)
Map<String, List<Person>> sortedByCity = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity,
        TreeMap::new,
        Collectors.toList()
    ));
```

### Partitioning

```java
// partitioningBy - تقسيم إلى true/false
Map<Boolean, List<Person>> partitionedByAge = people.stream()
    .collect(Collectors.partitioningBy(p -> p.getAge() >= 30));

System.out.println("Age >= 30: " + partitionedByAge.get(true));
System.out.println("Age < 30: " + partitionedByAge.get(false));

// partitioningBy مع downstream collector
Map<Boolean, Long> countByAge = people.stream()
    .collect(Collectors.partitioningBy(
        p -> p.getAge() >= 30,
        Collectors.counting()
    ));

Map<Boolean, Double> avgSalaryByAge = people.stream()
    .collect(Collectors.partitioningBy(
        p -> p.getAge() >= 30,
        Collectors.averagingDouble(Person::getSalary)
    ));
```

### Joining

```java
List<String> names = Arrays.asList("Ahmed", "Mohamed", "Ali", "Sara");

// joining - دمج strings
String joined = names.stream()
    .collect(Collectors.joining());
System.out.println(joined); // AhmedMohamedAliSara

// joining مع delimiter
String joinedWithComma = names.stream()
    .collect(Collectors.joining(", "));
System.out.println(joinedWithComma); // Ahmed, Mohamed, Ali, Sara

// joining مع prefix و suffix
String formatted = names.stream()
    .collect(Collectors.joining(", ", "[", "]"));
System.out.println(formatted); // [Ahmed, Mohamed, Ali, Sara]

// مثال: إنشاء SQL IN clause
String sqlIn = names.stream()
    .map(name -> "'" + name + "'")
    .collect(Collectors.joining(", ", "WHERE name IN (", ")"));
System.out.println(sqlIn);
// WHERE name IN ('Ahmed', 'Mohamed', 'Ali', 'Sara')
```

### Summarizing

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// summarizingInt
IntSummaryStatistics stats = numbers.stream()
    .collect(Collectors.summarizingInt(Integer::intValue));

System.out.println("Count: " + stats.getCount());
System.out.println("Sum: " + stats.getSum());
System.out.println("Min: " + stats.getMin());
System.out.println("Max: " + stats.getMax());
System.out.println("Average: " + stats.getAverage());

// summarizingDouble
DoubleSummaryStatistics salaryStats = people.stream()
    .collect(Collectors.summarizingDouble(Person::getSalary));

// summarizingLong
LongSummaryStatistics longStats = numbers.stream()
    .collect(Collectors.summarizingLong(Integer::longValue));
```

### Custom Collectors

```java
// Collector مخصص لحساب المتوسط
class AverageCollector {
    public static Collector<Integer, int[], Double> toAverage() {
        return Collector.of(
            () -> new int[2],  // supplier: [sum, count]
            (acc, value) -> {  // accumulator
                acc[0] += value;
                acc[1]++;
            },
            (acc1, acc2) -> {  // combiner
                acc1[0] += acc2[0];
                acc1[1] += acc2[1];
                return acc1;
            },
            acc -> acc[1] == 0 ? 0.0 : (double) acc[0] / acc[1] // finisher
        );
    }
}

double average = numbers.stream()
    .collect(AverageCollector.toAverage());
```

---

## أمثلة عملية متقدمة

### مثال 1: نظام إدارة الطلاب

```java
class Student {
    String id;
    String name;
    int age;
    String major;
    double gpa;
    List<String> courses;

    public Student(String id, String name, int age, String major,
                   double gpa, List<String> courses) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.major = major;
        this.gpa = gpa;
        this.courses = courses;
    }

    public String getId() { return id; }
    public String getName() { return name; }
    public int getAge() { return age; }
    public String getMajor() { return major; }
    public double getGpa() { return gpa; }
    public List<String> getCourses() { return courses; }
}

public class StudentManagement {
    public static void main(String[] args) {
        List<Student> students = Arrays.asList(
            new Student("S1", "Ahmed", 20, "CS", 3.8,
                Arrays.asList("Math", "Physics", "Programming")),
            new Student("S2", "Mohamed", 21, "IT", 3.5,
                Arrays.asList("Database", "Networks", "Security")),
            new Student("S3", "Ali", 19, "CS", 3.9,
                Arrays.asList("Math", "AI", "Programming")),
            new Student("S4", "Sara", 20, "IT", 3.7,
                Arrays.asList("Web Dev", "Database", "Mobile")),
            new Student("S5", "Fatma", 22, "CS", 3.6,
                Arrays.asList("AI", "ML", "Data Science"))
        );

        // 1. أعلى 3 طلاب حسب GPA
        System.out.println("Top 3 Students by GPA:");
        students.stream()
            .sorted(Comparator.comparingDouble(Student::getGpa).reversed())
            .limit(3)
            .forEach(s -> System.out.println(s.getName() + ": " + s.getGpa()));

        // 2. متوسط GPA حسب التخصص
        System.out.println("\nAverage GPA by Major:");
        students.stream()
            .collect(Collectors.groupingBy(
                Student::getMajor,
                Collectors.averagingDouble(Student::getGpa)
            ))
            .forEach((major, avg) ->
                System.out.println(major + ": " + String.format("%.2f", avg)));

        // 3. عدد الطلاب في كل تخصص
        System.out.println("\nStudent Count by Major:");
        students.stream()
            .collect(Collectors.groupingBy(
                Student::getMajor,
                Collectors.counting()
            ))
            .forEach((major, count) ->
                System.out.println(major + ": " + count));

        // 4. جميع الكورسات الفريدة
        System.out.println("\nAll Unique Courses:");
        Set<String> allCourses = students.stream()
            .flatMap(s -> s.getCourses().stream())
            .collect(Collectors.toSet());
        System.out.println(allCourses);

        // 5. الطلاب الذين يدرسون "Programming"
        System.out.println("\nStudents taking Programming:");
        students.stream()
            .filter(s -> s.getCourses().contains("Programming"))
            .map(Student::getName)
            .forEach(System.out::println);

        // 6. تجميع الطلاب حسب الفئة العمرية
        System.out.println("\nStudents by Age Group:");
        students.stream()
            .collect(Collectors.groupingBy(
                s -> s.getAge() < 20 ? "Teens" : s.getAge() < 22 ? "Early 20s" : "22+",
                Collectors.mapping(Student::getName, Collectors.toList())
            ))
            .forEach((group, names) ->
                System.out.println(group + ": " + names));

        // 7. الطالب صاحب أعلى GPA في كل تخصص
        System.out.println("\nTop Student by Major:");
        students.stream()
            .collect(Collectors.groupingBy(
                Student::getMajor,
                Collectors.maxBy(Comparator.comparingDouble(Student::getGpa))
            ))
            .forEach((major, student) ->
                student.ifPresent(s ->
                    System.out.println(major + ": " + s.getName() + " (" + s.getGpa() + ")")));

        // 8. الطلاب ذوو GPA أكبر من 3.7
        System.out.println("\nHonor Students (GPA > 3.7):");
        List<String> honorStudents = students.stream()
            .filter(s -> s.getGpa() > 3.7)
            .map(Student::getName)
            .collect(Collectors.toList());
        System.out.println(honorStudents);

        // 9. إحصائيات GPA
        System.out.println("\nGPA Statistics:");
        DoubleSummaryStatistics gpaStats = students.stream()
            .collect(Collectors.summarizingDouble(Student::getGpa));
        System.out.println("Count: " + gpaStats.getCount());
        System.out.println("Average: " + String.format("%.2f", gpaStats.getAverage()));
        System.out.println("Min: " + gpaStats.getMin());
        System.out.println("Max: " + gpaStats.getMax());

        // 10. Map من ID إلى اسم الطالب
        Map<String, String> idToName = students.stream()
            .collect(Collectors.toMap(
                Student::getId,
                Student::getName
            ));
        System.out.println("\nID to Name Map: " + idToName);
    }
}
```

### مثال 2: تحليل معاملات مالية

```java
class Transaction {
    String id;
    String type;  // "DEPOSIT" or "WITHDRAWAL"
    double amount;
    LocalDateTime timestamp;
    String accountId;

    public Transaction(String id, String type, double amount,
                      LocalDateTime timestamp, String accountId) {
        this.id = id;
        this.type = type;
        this.amount = amount;
        this.timestamp = timestamp;
        this.accountId = accountId;
    }

    public String getId() { return id; }
    public String getType() { return type; }
    public double getAmount() { return amount; }
    public LocalDateTime getTimestamp() { return timestamp; }
    public String getAccountId() { return accountId; }
}

public class TransactionAnalyzer {
    public static void main(String[] args) {
        List<Transaction> transactions = Arrays.asList(
            new Transaction("T1", "DEPOSIT", 1000,
                LocalDateTime.of(2025, 1, 15, 10, 0), "ACC1"),
            new Transaction("T2", "WITHDRAWAL", 200,
                LocalDateTime.of(2025, 1, 16, 14, 30), "ACC1"),
            new Transaction("T3", "DEPOSIT", 500,
                LocalDateTime.of(2025, 1, 17, 9, 15), "ACC2"),
            new Transaction("T4", "WITHDRAWAL", 150,
                LocalDateTime.of(2025, 1, 18, 16, 45), "ACC1"),
            new Transaction("T5", "DEPOSIT", 2000,
                LocalDateTime.of(2025, 1, 19, 11, 20), "ACC2"),
            new Transaction("T6", "WITHDRAWAL", 300,
                LocalDateTime.of(2025, 1, 20, 13, 10), "ACC2")
        );

        // 1. مجموع المبالغ حسب النوع
        System.out.println("Total by Type:");
        transactions.stream()
            .collect(Collectors.groupingBy(
                Transaction::getType,
                Collectors.summingDouble(Transaction::getAmount)
            ))
            .forEach((type, total) ->
                System.out.println(type + ": $" + total));

        // 2. عدد المعاملات لكل حساب
        System.out.println("\nTransaction Count by Account:");
        transactions.stream()
            .collect(Collectors.groupingBy(
                Transaction::getAccountId,
                Collectors.counting()
            ))
            .forEach((account, count) ->
                System.out.println(account + ": " + count));

        // 3. حساب الرصيد لكل حساب
        System.out.println("\nBalance by Account:");
        Map<String, Double> balanceByAccount = transactions.stream()
            .collect(Collectors.groupingBy(
                Transaction::getAccountId,
                Collectors.summingDouble(t ->
                    t.getType().equals("DEPOSIT") ? t.getAmount() : -t.getAmount()
                )
            ));
        balanceByAccount.forEach((account, balance) ->
            System.out.println(account + ": $" + balance));

        // 4. أكبر معاملة
        System.out.println("\nLargest Transaction:");
        transactions.stream()
            .max(Comparator.comparingDouble(Transaction::getAmount))
            .ifPresent(t -> System.out.println(
                "ID: " + t.getId() + ", Amount: $" + t.getAmount()));

        // 5. المعاملات في آخر 3 أيام
        LocalDateTime threeDaysAgo = LocalDateTime.now().minusDays(3);
        System.out.println("\nRecent Transactions (last 3 days):");
        long recentCount = transactions.stream()
            .filter(t -> t.getTimestamp().isAfter(threeDaysAgo))
            .count();
        System.out.println("Count: " + recentCount);

        // 6. متوسط قيمة المعاملة حسب النوع
        System.out.println("\nAverage Amount by Type:");
        transactions.stream()
            .collect(Collectors.groupingBy(
                Transaction::getType,
                Collectors.averagingDouble(Transaction::getAmount)
            ))
            .forEach((type, avg) ->
                System.out.println(type + ": $" + String.format("%.2f", avg)));

        // 7. تجميع المعاملات حسب الحساب والنوع
        System.out.println("\nTransactions by Account and Type:");
        Map<String, Map<String, List<Transaction>>> grouped = transactions.stream()
            .collect(Collectors.groupingBy(
                Transaction::getAccountId,
                Collectors.groupingBy(Transaction::getType)
            ));

        grouped.forEach((account, typeMap) -> {
            System.out.println(account + ":");
            typeMap.forEach((type, trans) ->
                System.out.println("  " + type + ": " + trans.size() + " transactions"));
        });

        // 8. إجمالي الإيداعات والسحوبات
        System.out.println("\nTotal Deposits and Withdrawals:");
        Map<Boolean, Double> depositVsWithdrawal = transactions.stream()
            .collect(Collectors.partitioningBy(
                t -> t.getType().equals("DEPOSIT"),
                Collectors.summingDouble(Transaction::getAmount)
            ));
        System.out.println("Deposits: $" + depositVsWithdrawal.get(true));
        System.out.println("Withdrawals: $" + depositVsWithdrawal.get(false));

        // 9. أحدث معاملة لكل حساب
        System.out.println("\nMost Recent Transaction by Account:");
        transactions.stream()
            .collect(Collectors.groupingBy(
                Transaction::getAccountId,
                Collectors.maxBy(Comparator.comparing(Transaction::getTimestamp))
            ))
            .forEach((account, transaction) ->
                transaction.ifPresent(t -> System.out.println(
                    account + ": " + t.getId() + " at " + t.getTimestamp())));

        // 10. تقرير يومي
        System.out.println("\nDaily Report:");
        transactions.stream()
            .collect(Collectors.groupingBy(
                t -> t.getTimestamp().toLocalDate(),
                TreeMap::new,
                Collectors.summarizingDouble(Transaction::getAmount)
            ))
            .forEach((date, stats) -> System.out.println(
                date + ": Count=" + stats.getCount() +
                ", Total=$" + stats.getSum() +
                ", Avg=$" + String.format("%.2f", stats.getAverage())));
    }
}
```

### مثال 3: معالجة ملفات Log

```java
class LogEntry {
    LocalDateTime timestamp;
    String level;  // INFO, WARNING, ERROR
    String message;
    String source;

    public LogEntry(LocalDateTime timestamp, String level,
                    String message, String source) {
        this.timestamp = timestamp;
        this.level = level;
        this.message = message;
        this.source = source;
    }

    public LocalDateTime getTimestamp() { return timestamp; }
    public String getLevel() { return level; }
    public String getMessage() { return message; }
    public String getSource() { return source; }
}

public class LogAnalyzer {
    public static void main(String[] args) {
        List<LogEntry> logs = Arrays.asList(
            new LogEntry(LocalDateTime.of(2025, 1, 15, 10, 0),
                "INFO", "Application started", "Main"),
            new LogEntry(LocalDateTime.of(2025, 1, 15, 10, 5),
                "WARNING", "High memory usage", "Monitor"),
            new LogEntry(LocalDateTime.of(2025, 1, 15, 10, 10),
                "ERROR", "Database connection failed", "DB"),
            new LogEntry(LocalDateTime.of(2025, 1, 15, 10, 15),
                "INFO", "User logged in", "Auth"),
            new LogEntry(LocalDateTime.of(2025, 1, 15, 10, 20),
                "ERROR", "File not found", "FileSystem"),
            new LogEntry(LocalDateTime.of(2025, 1, 15, 10, 25),
                "WARNING", "API timeout", "Network")
        );

        // 1. عدد الـ logs حسب المستوى
        System.out.println("Log Count by Level:");
        logs.stream()
            .collect(Collectors.groupingBy(
                LogEntry::getLevel,
                Collectors.counting()
            ))
            .forEach((level, count) ->
                System.out.println(level + ": " + count));

        // 2. جميع الأخطاء
        System.out.println("\nAll Errors:");
        logs.stream()
            .filter(log -> log.getLevel().equals("ERROR"))
            .forEach(log -> System.out.println(
                log.getTimestamp() + " - " + log.getMessage()));

        // 3. المصادر التي أنتجت errors
        System.out.println("\nSources with Errors:");
        Set<String> errorSources = logs.stream()
            .filter(log -> log.getLevel().equals("ERROR"))
            .map(LogEntry::getSource)
            .collect(Collectors.toSet());
        System.out.println(errorSources);

        // 4. Logs حسب الساعة
        System.out.println("\nLogs by Hour:");
        logs.stream()
            .collect(Collectors.groupingBy(
                log -> log.getTimestamp().getHour(),
                Collectors.counting()
            ))
            .forEach((hour, count) ->
                System.out.println("Hour " + hour + ": " + count));

        // 5. أول وآخر log لكل مستوى
        System.out.println("\nFirst and Last Log by Level:");
        logs.stream()
            .collect(Collectors.groupingBy(LogEntry::getLevel))
            .forEach((level, levelLogs) -> {
                LogEntry first = levelLogs.stream()
                    .min(Comparator.comparing(LogEntry::getTimestamp))
                    .orElse(null);
                LogEntry last = levelLogs.stream()
                    .max(Comparator.comparing(LogEntry::getTimestamp))
                    .orElse(null);
                System.out.println(level + ":");
                if (first != null)
                    System.out.println("  First: " + first.getTimestamp());
                if (last != null)
                    System.out.println("  Last: " + last.getTimestamp());
            });

        // 6. البحث في الرسائل
        System.out.println("\nLogs containing 'failed':");
        logs.stream()
            .filter(log -> log.getMessage().toLowerCase().contains("failed"))
            .forEach(log -> System.out.println(
                log.getLevel() + " - " + log.getMessage()));

        // 7. تجميع حسب Source والمستوى
        System.out.println("\nLogs by Source and Level:");
        Map<String, Map<String, Long>> sourceLevel = logs.stream()
            .collect(Collectors.groupingBy(
                LogEntry::getSource,
                Collectors.groupingBy(
                    LogEntry::getLevel,
                    Collectors.counting()
                )
            ));
        sourceLevel.forEach((source, levels) -> {
            System.out.println(source + ":");
            levels.forEach((level, count) ->
                System.out.println("  " + level + ": " + count));
        });
    }
}
```

### مثال 4: Stream Performance - Sequential vs Parallel

```java
public class StreamPerformance {
    public static void main(String[] args) {
        // إنشاء قائمة كبيرة
        List<Integer> largeList = IntStream.rangeClosed(1, 10_000_000)
            .boxed()
            .collect(Collectors.toList());

        // Sequential Stream
        long start = System.currentTimeMillis();
        long sum1 = largeList.stream()
            .mapToLong(Integer::longValue)
            .sum();
        long end = System.currentTimeMillis();
        System.out.println("Sequential: " + (end - start) + "ms, Sum: " + sum1);

        // Parallel Stream
        start = System.currentTimeMillis();
        long sum2 = largeList.parallelStream()
            .mapToLong(Integer::longValue)
            .sum();
        end = System.currentTimeMillis();
        System.out.println("Parallel: " + (end - start) + "ms, Sum: " + sum2);

        // مثال على عملية معقدة
        start = System.currentTimeMillis();
        double avg1 = largeList.stream()
            .filter(n -> n % 2 == 0)
            .mapToDouble(n -> Math.sqrt(n))
            .average()
            .orElse(0.0);
        end = System.currentTimeMillis();
        System.out.println("\nSequential Complex: " + (end - start) + "ms");

        start = System.currentTimeMillis();
        double avg2 = largeList.parallelStream()
            .filter(n -> n % 2 == 0)
            .mapToDouble(n -> Math.sqrt(n))
            .average()
            .orElse(0.0);
        end = System.currentTimeMillis();
        System.out.println("Parallel Complex: " + (end - start) + "ms");

        // تحذير: Parallel ليس دائماً أسرع
        List<String> strings = Arrays.asList("a", "b", "c", "d", "e");

        // مع قائمة صغيرة، sequential أسرع
        start = System.nanoTime();
        strings.stream().map(String::toUpperCase).collect(Collectors.toList());
        long sequential = System.nanoTime() - start;

        start = System.nanoTime();
        strings.parallelStream().map(String::toUpperCase).collect(Collectors.toList());
        long parallel = System.nanoTime() - start;

        System.out.println("\nSmall List (nanoseconds):");
        System.out.println("Sequential: " + sequential);
        System.out.println("Parallel: " + parallel);
    }
}
```

---

## نصائح وأفضل الممارسات

### 1. اختيار Collection المناسبة

```java
// ✅ ArrayList - للوصول السريع بالـ index
List<String> list = new ArrayList<>();

// ✅ LinkedList - للإضافة/الحذف من البداية/النهاية
List<String> linkedList = new LinkedList<>();

// ✅ HashSet - للبحث السريع بدون تكرار
Set<String> set = new HashSet<>();

// ✅ TreeSet - للترتيب التلقائي
Set<String> sortedSet = new TreeSet<>();

// ✅ HashMap - للبحث السريع بـ key-value
Map<String, String> map = new HashMap<>();

// ✅ TreeMap - للـ keys مرتبة
Map<String, String> sortedMap = new TreeMap<>();
```

### 2. Stream Best Practices

```java
// ❌ لا تعيد استخدام Stream
Stream<String> stream = list.stream();
stream.forEach(System.out::println);
// stream.count(); // خطأ: stream already used

// ✅ استخدم Stream جديد
list.stream().forEach(System.out::println);
list.stream().count();

// ❌ لا تعدل Collection أثناء Stream
list.stream().forEach(s -> list.add("new")); // ConcurrentModificationException

// ✅ اجمع النتائج أولاً
List<String> toAdd = list.stream()
    .map(s -> s + "_new")
    .collect(Collectors.toList());
list.addAll(toAdd);

// ❌ لا تستخدم parallel للعمليات البسيطة
List<Integer> small = Arrays.asList(1, 2, 3);
small.parallelStream().forEach(System.out::println); // overhead كبير

// ✅ استخدم parallel مع البيانات الكبيرة فقط
List<Integer> large = IntStream.range(1, 1_000_000).boxed().collect(Collectors.toList());
large.parallelStream().filter(n -> n % 2 == 0).count();
```

### 3. Performance Tips

```java
// ✅ استخدم primitive streams عندما يمكن
IntStream.range(1, 1000).sum();  // أسرع
Stream.iterate(1, n -> n + 1).limit(1000).mapToInt(Integer::intValue).sum(); // أبطأ

// ✅ تجنب boxing/unboxing غير الضروري
int sum1 = numbers.stream().mapToInt(Integer::intValue).sum(); // ✅
int sum2 = numbers.stream().reduce(0, Integer::sum); // ✅ أبطأ قليلاً

// ✅ استخدم method references عندما يمكن
list.stream().map(String::toUpperCase); // ✅ أوضح
list.stream().map(s -> s.toUpperCase()); // ✅ نفس الشيء لكن أطول
```

---

## الخلاصة

### Collections Framework

✅ **List** - مرتبة، تسمح بالتكرار، لها index
✅ **Set** - فريدة، لا تكرار، بدون index
✅ **Queue** - FIFO/Priority، للمعالجة المتسلسلة
✅ **Map** - Key-Value pairs، keys فريدة

### Stream API

✅ **Declarative** - تصف ماذا تريد، ليس كيف
✅ **Lazy** - لا تنفذ إلا عند terminal operation
✅ **Parallelizable** - سهل تحويلها لـ parallel
✅ **Composable** - يمكن دمج operations

### متى تستخدم ماذا؟

| الحالة | الحل |
|--------|------|
| ترتيب + وصول سريع | ArrayList |
| إضافة/حذف من البداية | LinkedList |
| عدم تكرار + بحث سريع | HashSet |
| عدم تكرار + ترتيب | TreeSet |
| Key-Value + بحث سريع | HashMap |
| Key-Value + ترتيب | TreeMap |
| معالجة بيانات | Stream API |
| تجميع وتحليل | Collectors |

---

**تم إنشاء هذا الدليل كمرجع شامل لـ Java Collections و Streams**
