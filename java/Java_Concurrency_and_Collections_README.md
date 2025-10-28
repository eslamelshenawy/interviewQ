# Java Concurrency, Keywords, and Collections

## 1. Different Ways to Create a Thread

### الطريقة الأولى: Extending Thread Class
```java
class MyThread extends Thread {
    @Override
    public void run() {
        for (int i = 1; i <= 5; i++) {
            System.out.println(Thread.currentThread().getName() + " - Count: " + i);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

public class ThreadExample1 {
    public static void main(String[] args) {
        MyThread thread1 = new MyThread();
        MyThread thread2 = new MyThread();

        thread1.setName("Thread-A");
        thread2.setName("Thread-B");

        thread1.start();
        thread2.start();
    }
}
```

### الطريقة الثانية: Implementing Runnable Interface
```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        for (int i = 1; i <= 5; i++) {
            System.out.println(Thread.currentThread().getName() + " - Count: " + i);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

public class ThreadExample2 {
    public static void main(String[] args) {
        MyRunnable runnable = new MyRunnable();

        Thread thread1 = new Thread(runnable, "Thread-A");
        Thread thread2 = new Thread(runnable, "Thread-B");

        thread1.start();
        thread2.start();
    }
}
```

### الطريقة الثالثة: Anonymous Inner Class
```java
public class ThreadExample3 {
    public static void main(String[] args) {
        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("Thread using anonymous class");
            }
        });

        thread1.start();
    }
}
```

### الطريقة الرابعة: Lambda Expression (Java 8+)
```java
public class ThreadExample4 {
    public static void main(String[] args) {
        // Lambda with Thread
        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                System.out.println("Lambda Thread: " + i);
            }
        });

        thread1.start();

        // أو مباشرة
        new Thread(() -> System.out.println("Direct Lambda")).start();
    }
}
```

### الطريقة الخامسة: Using Executor Framework
```java
import java.util.concurrent.*;

public class ThreadExample5 {
    public static void main(String[] args) {
        // Fixed Thread Pool
        ExecutorService executor = Executors.newFixedThreadPool(3);

        for (int i = 0; i < 5; i++) {
            final int taskId = i;
            executor.submit(() -> {
                System.out.println("Task " + taskId + " executed by " +
                                 Thread.currentThread().getName());
            });
        }

        executor.shutdown();

        // Single Thread Executor
        ExecutorService singleExecutor = Executors.newSingleThreadExecutor();
        singleExecutor.submit(() -> System.out.println("Single thread task"));
        singleExecutor.shutdown();

        // Cached Thread Pool
        ExecutorService cachedExecutor = Executors.newCachedThreadPool();
        cachedExecutor.submit(() -> System.out.println("Cached thread task"));
        cachedExecutor.shutdown();

        // Scheduled Executor
        ScheduledExecutorService scheduledExecutor = Executors.newScheduledThreadPool(2);
        scheduledExecutor.schedule(() -> System.out.println("Delayed task"),
                                  2, TimeUnit.SECONDS);
        scheduledExecutor.scheduleAtFixedRate(() -> System.out.println("Periodic task"),
                                             0, 1, TimeUnit.SECONDS);
    }
}
```

### الطريقة السادسة: Using Callable and Future
```java
import java.util.concurrent.*;

class MyCallable implements Callable<Integer> {
    private final int number;

    public MyCallable(int number) {
        this.number = number;
    }

    @Override
    public Integer call() throws Exception {
        int sum = 0;
        for (int i = 1; i <= number; i++) {
            sum += i;
        }
        Thread.sleep(1000);
        return sum;
    }
}

public class ThreadExample6 {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newSingleThreadExecutor();

        Future<Integer> future = executor.submit(new MyCallable(10));

        System.out.println("Doing other work...");

        // Get the result (blocks if not ready)
        Integer result = future.get();
        System.out.println("Sum = " + result);

        executor.shutdown();
    }
}
```

### الطريقة السابعة: CompletableFuture (Java 8+)
```java
import java.util.concurrent.CompletableFuture;

public class ThreadExample7 {
    public static void main(String[] args) {
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "Hello from CompletableFuture";
        });

        future.thenAccept(result -> System.out.println("Result: " + result));

        // Chain multiple operations
        CompletableFuture<Integer> chainedFuture =
            CompletableFuture.supplyAsync(() -> 10)
                .thenApply(x -> x * 2)
                .thenApply(x -> x + 5);

        chainedFuture.thenAccept(System.out::println);

        // Wait for completion
        CompletableFuture.allOf(future, chainedFuture).join();
    }
}
```

## 2. Difference between Runnable and Callable

### Runnable Interface
```java
@FunctionalInterface
public interface Runnable {
    void run();
}
```

### Callable Interface
```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

### الفروقات الأساسية

| الخاصية | Runnable | Callable |
|---------|----------|----------|
| **Return Type** | void - لا يرجع قيمة | Generic type V - يرجع قيمة |
| **Exceptions** | لا يمكن throw checked exceptions | يمكن throw Exception |
| **Method Name** | run() | call() |
| **متى ظهر** | Java 1.0 | Java 5 |
| **الاستخدام مع** | Thread class مباشرة | ExecutorService فقط |
| **Future** | لا يدعم | يدعم Future للحصول على النتيجة |

### أمثلة المقارنة

```java
import java.util.concurrent.*;

public class RunnableVsCallable {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newSingleThreadExecutor();

        // Runnable - لا يرجع قيمة
        Runnable runnable = () -> {
            System.out.println("Runnable executing");
            // return 10;  // خطأ! Runnable لا يرجع قيمة
        };

        executor.submit(runnable);

        // Callable - يرجع قيمة
        Callable<String> callable = () -> {
            System.out.println("Callable executing");
            Thread.sleep(1000);
            return "Result from Callable";
        };

        Future<String> future = executor.submit(callable);
        String result = future.get(); // Block and get result
        System.out.println(result);

        // Callable with Exception
        Callable<Integer> callableWithException = () -> {
            if (Math.random() > 0.5) {
                throw new Exception("Random error!");
            }
            return 42;
        };

        try {
            Future<Integer> future2 = executor.submit(callableWithException);
            Integer result2 = future2.get();
            System.out.println("Result: " + result2);
        } catch (ExecutionException e) {
            System.out.println("Exception occurred: " + e.getCause());
        }

        executor.shutdown();
    }
}
```

## 3. Transient Keyword

### ما هو transient؟
الـ `transient` keyword يُستخدم مع variables لمنعها من الـ serialization. عندما object يتم serialize، الـ transient fields لا يتم حفظها.

### متى نستخدم transient؟
1. **Sensitive Data**: passwords, credit card numbers
2. **Derived/Calculated Fields**: قيم يمكن حسابها من fields أخرى
3. **Non-Serializable Objects**: objects لا تدعم serialization
4. **Temporary/Cache Data**: بيانات مؤقتة

### أمثلة عملية

```java
import java.io.*;

class User implements Serializable {
    private static final long serialVersionUID = 1L;

    private String username;
    private transient String password;  // لن يتم حفظه
    private transient int age;  // calculated field
    private String email;
    private transient Logger logger;  // non-serializable

    // transient مع static - غير ضروري لأن static لا يتم serialize أصلاً
    private static transient String companyName = "TechCorp";

    public User(String username, String password, String email) {
        this.username = username;
        this.password = password;
        this.email = email;
        this.age = calculateAge();
        this.logger = Logger.getLogger(User.class.getName());
    }

    private int calculateAge() {
        // Calculate from some other data
        return 25;
    }

    @Override
    public String toString() {
        return "User{username='" + username + "', password='" + password +
               "', email='" + email + "', age=" + age + "}";
    }
}

public class TransientExample {
    public static void main(String[] args) {
        User user = new User("john_doe", "secret123", "john@email.com");
        System.out.println("Before Serialization: " + user);

        // Serialize
        try (ObjectOutputStream oos = new ObjectOutputStream(
                new FileOutputStream("user.ser"))) {
            oos.writeObject(user);
        } catch (IOException e) {
            e.printStackTrace();
        }

        // Deserialize
        try (ObjectInputStream ois = new ObjectInputStream(
                new FileInputStream("user.ser"))) {
            User deserializedUser = (User) ois.readObject();
            System.out.println("After Deserialization: " + deserializedUser);
            // password سيكون null لأنه transient
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}

// Output:
// Before Serialization: User{username='john_doe', password='secret123', email='john@email.com', age=25}
// After Deserialization: User{username='john_doe', password='null', email='john@email.com', age=0}
```

### Custom Serialization مع transient
```java
class Account implements Serializable {
    private String accountNumber;
    private transient String pin;  // transient but we'll handle it

    // Custom serialization
    private void writeObject(ObjectOutputStream oos) throws IOException {
        oos.defaultWriteObject();
        // Encrypt PIN before saving
        String encryptedPin = encrypt(pin);
        oos.writeObject(encryptedPin);
    }

    private void readObject(ObjectInputStream ois)
            throws IOException, ClassNotFoundException {
        ois.defaultReadObject();
        // Decrypt PIN after reading
        String encryptedPin = (String) ois.readObject();
        this.pin = decrypt(encryptedPin);
    }

    private String encrypt(String data) {
        // Simple encryption (use proper encryption in real code)
        return Base64.getEncoder().encodeToString(data.getBytes());
    }

    private String decrypt(String data) {
        return new String(Base64.getDecoder().decode(data));
    }
}
```

## 4. Volatile Keyword

### ما هو volatile؟
الـ `volatile` keyword يضمن أن قيمة المتغير تُقرأ من الـ main memory وليس من cache الخاص بكل thread.

### متى نستخدم volatile؟
1. **Flags/Status Variables**: متغيرات تتحكم في thread execution
2. **One Writer, Multiple Readers**: متغير يُكتب من thread واحد ويُقرأ من threads متعددة
3. **Double-Checked Locking**: في Singleton pattern

### كيف يعمل volatile؟
```java
public class VolatileExample {
    private static volatile boolean flag = false;

    public static void main(String[] args) throws InterruptedException {
        // Thread 1 - Writer
        Thread writer = new Thread(() -> {
            System.out.println("Writer thread started");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            flag = true;  // Write to main memory
            System.out.println("Flag set to true");
        });

        // Thread 2 - Reader
        Thread reader = new Thread(() -> {
            System.out.println("Reader thread started");
            while (!flag) {  // Read from main memory
                // بدون volatile، قد لا يرى التغيير أبداً
            }
            System.out.println("Flag detected as true!");
        });

        reader.start();
        writer.start();

        reader.join();
        writer.join();
    }
}
```

### Singleton with volatile (Double-Checked Locking)
```java
public class Singleton {
    // volatile يضمن أن instance مرئي لجميع threads
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {  // First check (no locking)
            synchronized (Singleton.class) {
                if (instance == null) {  // Second check (with locking)
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

### volatile vs synchronized

| الخاصية | volatile | synchronized |
|---------|----------|--------------|
| **Visibility** | ✅ يضمن visibility | ✅ يضمن visibility |
| **Atomicity** | ❌ لا يضمن atomicity | ✅ يضمن atomicity |
| **Performance** | أسرع | أبطأ (locking overhead) |
| **Blocking** | Non-blocking | Blocking |
| **الاستخدام** | Simple flags | Complex operations |

```java
public class VolatileVsSynchronized {
    private volatile int volatileCounter = 0;
    private int synchronizedCounter = 0;

    // NOT thread-safe even with volatile!
    public void incrementVolatile() {
        volatileCounter++;  // Not atomic (read-modify-write)
    }

    // Thread-safe
    public synchronized void incrementSynchronized() {
        synchronizedCounter++;
    }

    // Better solution for counters
    private AtomicInteger atomicCounter = new AtomicInteger(0);

    public void incrementAtomic() {
        atomicCounter.incrementAndGet();  // Atomic operation
    }
}
```

## 5. Internal Working of HashMap

### قبل Java 8
- **Structure**: Array of LinkedList (buckets)
- **Collision Resolution**: Chaining with LinkedList
- **Time Complexity**: O(1) average, O(n) worst case

### بعد Java 8
- **Structure**: Array of LinkedList/TreeNode
- **Treeification**: LinkedList يتحول لـ Red-Black Tree عند threshold
- **Time Complexity**: O(1) average, O(log n) worst case

### كيف يعمل HashMap داخلياً

```java
// Simplified HashMap Structure
class MyHashMap<K, V> {
    // Default values
    static final int DEFAULT_CAPACITY = 16;
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    static final int TREEIFY_THRESHOLD = 8;  // Java 8+
    static final int UNTREEIFY_THRESHOLD = 6;  // Java 8+

    // Internal Node class
    static class Node<K, V> {
        final int hash;
        final K key;
        V value;
        Node<K, V> next;

        Node(int hash, K key, V value, Node<K, V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }

    // The table (array of buckets)
    Node<K, V>[] table;
    int size;
    int threshold;
    final float loadFactor;

    // Put operation
    public V put(K key, V value) {
        return putVal(hash(key), key, value);
    }

    // Hash function
    static final int hash(Object key) {
        int h;
        // XOR higher bits with lower bits for better distribution
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

    // Determine bucket index
    static int indexFor(int hash, int length) {
        return hash & (length - 1);  // Same as hash % length when length is power of 2
    }

    final V putVal(int hash, K key, V value) {
        Node<K, V>[] tab;
        Node<K, V> p;
        int n, i;

        // Initialize or resize if necessary
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;

        // Calculate index
        if ((p = tab[i = (n - 1) & hash]) == null) {
            // No collision - create new node
            tab[i] = new Node<>(hash, key, value, null);
        } else {
            // Collision handling
            Node<K, V> e;
            K k;

            // Check if key already exists
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k)))) {
                e = p;  // Key exists, update value
            } else if (p instanceof TreeNode) {  // Java 8+
                // Tree node handling (Red-Black Tree)
                e = ((TreeNode<K, V>)p).putTreeVal(this, tab, hash, key, value);
            } else {
                // LinkedList traversal
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = new Node<>(hash, key, value, null);
                        // Treeify if threshold reached (Java 8+)
                        if (binCount >= TREEIFY_THRESHOLD - 1)
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }

            // Update existing value
            if (e != null) {
                V oldValue = e.value;
                e.value = value;
                return oldValue;
            }
        }

        // Resize if needed
        if (++size > threshold)
            resize();

        return null;
    }

    // Get operation
    public V get(Object key) {
        Node<K, V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }

    final Node<K, V> getNode(int hash, Object key) {
        Node<K, V>[] tab;
        Node<K, V> first, e;
        int n;
        K k;

        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {

            // Check first node
            if (first.hash == hash &&
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;

            // Check rest of the chain
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)  // Java 8+
                    return ((TreeNode<K, V>)first).getTreeNode(hash, key);

                // LinkedList traversal
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
}
```

### التحسينات في Java 8

```java
public class Java8HashMapImprovements {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();

        // 1. Treeification
        // عندما bucket واحد يحتوي على 8+ عناصر، يتحول من LinkedList لـ Red-Black Tree
        // هذا يحسن worst-case من O(n) لـ O(log n)

        // 2. putIfAbsent
        map.putIfAbsent("key", 10);  // يضع القيمة فقط إذا key غير موجود

        // 3. compute methods
        map.compute("key", (k, v) -> v == null ? 1 : v + 1);
        map.computeIfAbsent("newKey", k -> k.length());
        map.computeIfPresent("key", (k, v) -> v * 2);

        // 4. merge
        map.merge("key", 5, (oldVal, newVal) -> oldVal + newVal);

        // 5. forEach
        map.forEach((k, v) -> System.out.println(k + ": " + v));

        // 6. replaceAll
        map.replaceAll((k, v) -> v * 2);

        // 7. getOrDefault
        Integer value = map.getOrDefault("missingKey", 0);
    }
}
```

## 6. Internal Working of HashSet

### الحقيقة المفاجئة
**HashSet internally uses HashMap!**

```java
public class HashSet<E> extends AbstractSet<E> implements Set<E> {
    private transient HashMap<E, Object> map;

    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();

    public HashSet() {
        map = new HashMap<>();
    }

    public HashSet(int initialCapacity) {
        map = new HashMap<>(initialCapacity);
    }

    // Add element
    public boolean add(E e) {
        // Key = element, Value = PRESENT (dummy object)
        return map.put(e, PRESENT) == null;
    }

    // Remove element
    public boolean remove(Object o) {
        return map.remove(o) == PRESENT;
    }

    // Contains check
    public boolean contains(Object o) {
        return map.containsKey(o);
    }

    // Size
    public int size() {
        return map.size();
    }

    // Clear
    public void clear() {
        map.clear();
    }

    // Iterator
    public Iterator<E> iterator() {
        return map.keySet().iterator();
    }
}
```

### كيف يعمل HashSet

1. **Storage**: كل عنصر في HashSet يُخزن كـ key في HashMap داخلي
2. **Value**: جميع keys لها نفس القيمة (PRESENT object)
3. **Uniqueness**: HashMap يضمن عدم تكرار keys
4. **Performance**: نفس performance الـ HashMap - O(1) average

### مثال عملي

```java
public class HashSetExample {
    public static void main(String[] args) {
        Set<String> set = new HashSet<>();

        // Adding elements
        System.out.println(set.add("Apple"));   // true (added)
        System.out.println(set.add("Banana"));  // true (added)
        System.out.println(set.add("Apple"));   // false (duplicate)

        // Behind the scenes:
        // HashMap: {"Apple" -> PRESENT, "Banana" -> PRESENT}

        // Custom objects in HashSet
        Set<Person> personSet = new HashSet<>();

        Person p1 = new Person("John", 25);
        Person p2 = new Person("John", 25);

        personSet.add(p1);
        personSet.add(p2);  // Will add if equals() and hashCode() not overridden

        System.out.println("Set size: " + personSet.size());
    }
}

class Person {
    String name;
    int age;

    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // Must override for proper HashSet behavior
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && name.equals(person.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

### LinkedHashSet vs TreeSet

```java
public class SetComparison {
    public static void main(String[] args) {
        // HashSet - No order
        Set<Integer> hashSet = new HashSet<>();
        hashSet.add(5); hashSet.add(1); hashSet.add(3);
        System.out.println("HashSet: " + hashSet);  // [1, 3, 5] or any order

        // LinkedHashSet - Insertion order
        Set<Integer> linkedHashSet = new LinkedHashSet<>();
        linkedHashSet.add(5); linkedHashSet.add(1); linkedHashSet.add(3);
        System.out.println("LinkedHashSet: " + linkedHashSet);  // [5, 1, 3]

        // TreeSet - Sorted order
        Set<Integer> treeSet = new TreeSet<>();
        treeSet.add(5); treeSet.add(1); treeSet.add(3);
        System.out.println("TreeSet: " + treeSet);  // [1, 3, 5]
    }
}
```

## خلاصة Performance

| Operation | HashMap/HashSet | After Java 8 (worst case) |
|-----------|----------------|---------------------------|
| put/add | O(1) average | O(log n) with treeification |
| get/contains | O(1) average | O(log n) with treeification |
| remove | O(1) average | O(log n) with treeification |
| Collision handling | LinkedList O(n) | Red-Black Tree O(log n) |

## Important Notes

1. **HashMap null handling**: يسمح بـ null key واحد وعدة null values
2. **HashSet null handling**: يسمح بـ null element واحد
3. **Thread Safety**: كلاهما NOT thread-safe. استخدم ConcurrentHashMap أو Collections.synchronizedSet()
4. **Initial Capacity**: 16 by default
5. **Load Factor**: 0.75 by default (resize when 75% full)
6. **Resize**: يتضاعف الحجم (capacity * 2)