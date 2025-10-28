# Coding Problems & Solutions

## 1. Group Anagrams Together

### المشكلة
Given an array of strings, group the anagrams together. Anagrams are words that contain the same letters but in different order.

**Input:** `["eat","tea","tan","ate","nat","bat"]`
**Output:** `[["bat"],["nat","tan"],["ate","eat","tea"]]`

### الحل الأول: Using Stream API (Java 8+)

```java
import java.util.*;
import java.util.stream.Collectors;

public class GroupAnagramsStream {

    public static List<List<String>> groupAnagrams(String[] strs) {
        // Stream API solution
        return Arrays.stream(strs)
            .collect(Collectors.groupingBy(
                // Key mapper: sort characters to create key
                str -> {
                    char[] chars = str.toCharArray();
                    Arrays.sort(chars);
                    return new String(chars);
                }
            ))
            .values()
            .stream()
            .collect(Collectors.toList());
    }

    // Alternative Stream solution with more clarity
    public static List<List<String>> groupAnagramsStreamAlternative(String[] strs) {
        Map<String, List<String>> anagramMap = Arrays.stream(strs)
            .collect(Collectors.groupingBy(
                str -> sortString(str),
                Collectors.toList()
            ));

        return new ArrayList<>(anagramMap.values());
    }

    // Helper method to sort string characters
    private static String sortString(String str) {
        char[] chars = str.toCharArray();
        Arrays.sort(chars);
        return new String(chars);
    }

    // One-liner version (less readable but compact)
    public static List<List<String>> groupAnagramsOneLiner(String[] strs) {
        return new ArrayList<>(
            Arrays.stream(strs)
                .collect(Collectors.groupingBy(
                    s -> s.chars()
                        .sorted()
                        .collect(StringBuilder::new, StringBuilder::appendCodePoint, StringBuilder::append)
                        .toString()
                ))
                .values()
        );
    }

    // Using frequency count as key (alternative approach)
    public static List<List<String>> groupAnagramsFrequency(String[] strs) {
        return Arrays.stream(strs)
            .collect(Collectors.groupingBy(
                str -> {
                    // Create frequency count key
                    int[] count = new int[26];
                    for (char c : str.toCharArray()) {
                        count[c - 'a']++;
                    }
                    return Arrays.toString(count);
                }
            ))
            .values()
            .stream()
            .collect(Collectors.toList());
    }

    public static void main(String[] args) {
        String[] input = {"eat", "tea", "tan", "ate", "nat", "bat"};

        System.out.println("Stream API Solution:");
        List<List<String>> result = groupAnagrams(input);
        result.forEach(System.out::println);

        System.out.println("\nAlternative Stream Solution:");
        List<List<String>> result2 = groupAnagramsStreamAlternative(input);
        result2.forEach(System.out::println);

        System.out.println("\nFrequency Count Solution:");
        List<List<String>> result3 = groupAnagramsFrequency(input);
        result3.forEach(System.out::println);
    }
}
```

### الحل الثاني: Without Stream API (Traditional Approach)

```java
import java.util.*;

public class GroupAnagramsTraditional {

    // Solution 1: Using sorted string as key
    public static List<List<String>> groupAnagrams(String[] strs) {
        // Edge case
        if (strs == null || strs.length == 0) {
            return new ArrayList<>();
        }

        // Map to store anagram groups
        Map<String, List<String>> anagramMap = new HashMap<>();

        // Process each string
        for (String str : strs) {
            // Sort characters to create key
            char[] chars = str.toCharArray();
            Arrays.sort(chars);
            String key = new String(chars);

            // Add to corresponding group
            if (!anagramMap.containsKey(key)) {
                anagramMap.put(key, new ArrayList<>());
            }
            anagramMap.get(key).add(str);
        }

        // Convert map values to result list
        return new ArrayList<>(anagramMap.values());
    }

    // Solution 2: Using character frequency as key
    public static List<List<String>> groupAnagramsFrequency(String[] strs) {
        if (strs == null || strs.length == 0) {
            return new ArrayList<>();
        }

        Map<String, List<String>> anagramMap = new HashMap<>();

        for (String str : strs) {
            // Create frequency count array
            int[] count = new int[26]; // for lowercase letters only

            for (char c : str.toCharArray()) {
                count[c - 'a']++;
            }

            // Create key from frequency array
            StringBuilder keyBuilder = new StringBuilder();
            for (int i = 0; i < 26; i++) {
                if (count[i] > 0) {
                    keyBuilder.append((char)('a' + i)).append(count[i]);
                }
            }
            String key = keyBuilder.toString();

            // Add to map
            if (!anagramMap.containsKey(key)) {
                anagramMap.put(key, new ArrayList<>());
            }
            anagramMap.get(key).add(str);
        }

        return new ArrayList<>(anagramMap.values());
    }

    // Solution 3: Using prime numbers (mathematical approach)
    public static List<List<String>> groupAnagramsPrime(String[] strs) {
        if (strs == null || strs.length == 0) {
            return new ArrayList<>();
        }

        // Prime numbers for each letter
        int[] primes = {2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41,
                       43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97, 101};

        Map<Long, List<String>> anagramMap = new HashMap<>();

        for (String str : strs) {
            long key = 1;

            // Calculate product of primes
            for (char c : str.toCharArray()) {
                key *= primes[c - 'a'];
            }

            // Add to map
            if (!anagramMap.containsKey(key)) {
                anagramMap.put(key, new ArrayList<>());
            }
            anagramMap.get(key).add(str);
        }

        return new ArrayList<>(anagramMap.values());
    }

    // Solution 4: Optimized with getOrDefault
    public static List<List<String>> groupAnagramsOptimized(String[] strs) {
        if (strs == null || strs.length == 0) {
            return new ArrayList<>();
        }

        Map<String, List<String>> anagramMap = new HashMap<>();

        for (String str : strs) {
            // Sort characters
            char[] chars = str.toCharArray();
            Arrays.sort(chars);
            String key = String.valueOf(chars);

            // Use getOrDefault for cleaner code
            List<String> list = anagramMap.getOrDefault(key, new ArrayList<>());
            list.add(str);
            anagramMap.put(key, list);
        }

        return new ArrayList<>(anagramMap.values());
    }

    // Solution 5: Manual implementation without using HashMap methods
    public static List<List<String>> groupAnagramsManual(String[] strs) {
        List<List<String>> result = new ArrayList<>();
        boolean[] used = new boolean[strs.length];

        for (int i = 0; i < strs.length; i++) {
            if (used[i]) continue;

            List<String> currentGroup = new ArrayList<>();
            currentGroup.add(strs[i]);
            used[i] = true;

            for (int j = i + 1; j < strs.length; j++) {
                if (!used[j] && areAnagrams(strs[i], strs[j])) {
                    currentGroup.add(strs[j]);
                    used[j] = true;
                }
            }

            result.add(currentGroup);
        }

        return result;
    }

    // Helper method to check if two strings are anagrams
    private static boolean areAnagrams(String s1, String s2) {
        if (s1.length() != s2.length()) {
            return false;
        }

        int[] count = new int[26];

        for (char c : s1.toCharArray()) {
            count[c - 'a']++;
        }

        for (char c : s2.toCharArray()) {
            count[c - 'a']--;
            if (count[c - 'a'] < 0) {
                return false;
            }
        }

        return true;
    }

    public static void main(String[] args) {
        String[] input = {"eat", "tea", "tan", "ate", "nat", "bat"};

        System.out.println("Traditional Solution (Sorted Key):");
        List<List<String>> result1 = groupAnagrams(input);
        for (List<String> group : result1) {
            System.out.println(group);
        }

        System.out.println("\nFrequency Count Solution:");
        List<List<String>> result2 = groupAnagramsFrequency(input);
        for (List<String> group : result2) {
            System.out.println(group);
        }

        System.out.println("\nPrime Number Solution:");
        List<List<String>> result3 = groupAnagramsPrime(input);
        for (List<String> group : result3) {
            System.out.println(group);
        }

        System.out.println("\nOptimized Solution:");
        List<List<String>> result4 = groupAnagramsOptimized(input);
        for (List<String> group : result4) {
            System.out.println(group);
        }

        System.out.println("\nManual Solution:");
        List<List<String>> result5 = groupAnagramsManual(input);
        for (List<String> group : result5) {
            System.out.println(group);
        }
    }
}
```

### Time & Space Complexity Analysis

| Solution | Time Complexity | Space Complexity | Notes |
|----------|----------------|------------------|-------|
| Sorted Key | O(n * k log k) | O(n * k) | n = array length, k = max string length |
| Frequency Count | O(n * k) | O(n * k) | Most efficient for time |
| Prime Numbers | O(n * k) | O(n * k) | Risk of overflow for long strings |
| Manual (Brute Force) | O(n² * k) | O(n) | Least efficient |

## 2. Sum of Two Numbers Using Lambda Expression

### Multiple Approaches with Lambda

```java
import java.util.function.*;
import java.util.Arrays;
import java.util.List;
import java.util.stream.IntStream;

public class SumWithLambda {

    // 1. Basic Lambda with BiFunction
    public static void basicLambdaSum() {
        // BiFunction takes two arguments and returns a result
        BiFunction<Integer, Integer, Integer> sum = (a, b) -> a + b;

        int result = sum.apply(10, 20);
        System.out.println("Sum using BiFunction: " + result); // 30
    }

    // 2. Custom Functional Interface
    @FunctionalInterface
    interface Calculator {
        int calculate(int a, int b);
    }

    public static void customInterfaceSum() {
        // Lambda expression for sum
        Calculator sum = (a, b) -> a + b;

        // Lambda expression for other operations
        Calculator subtract = (a, b) -> a - b;
        Calculator multiply = (a, b) -> a * b;
        Calculator divide = (a, b) -> {
            if (b == 0) {
                throw new ArithmeticException("Division by zero");
            }
            return a / b;
        };

        System.out.println("Sum: " + sum.calculate(15, 25)); // 40
        System.out.println("Subtract: " + subtract.calculate(30, 10)); // 20
        System.out.println("Multiply: " + multiply.calculate(5, 6)); // 30
        System.out.println("Divide: " + divide.calculate(20, 4)); // 5
    }

    // 3. IntBinaryOperator (specialized for int)
    public static void intBinaryOperatorSum() {
        IntBinaryOperator sum = (a, b) -> a + b;

        int result = sum.applyAsInt(100, 200);
        System.out.println("Sum using IntBinaryOperator: " + result); // 300

        // Can also use method reference
        IntBinaryOperator sumRef = Integer::sum;
        System.out.println("Sum using method reference: " + sumRef.applyAsInt(50, 75)); // 125
    }

    // 4. BinaryOperator (generic version)
    public static void binaryOperatorSum() {
        BinaryOperator<Integer> sum = (a, b) -> a + b;

        Integer result = sum.apply(33, 67);
        System.out.println("Sum using BinaryOperator: " + result); // 100

        // With Double
        BinaryOperator<Double> sumDouble = (a, b) -> a + b;
        Double doubleResult = sumDouble.apply(10.5, 20.3);
        System.out.println("Sum of doubles: " + doubleResult); // 30.8
    }

    // 5. Stream API for sum of array/list
    public static void streamSum() {
        // Sum of array elements
        int[] numbers = {1, 2, 3, 4, 5};
        int sum = Arrays.stream(numbers)
                        .reduce(0, (a, b) -> a + b);
        System.out.println("Sum of array: " + sum); // 15

        // Sum of list elements
        List<Integer> list = Arrays.asList(10, 20, 30, 40, 50);
        int listSum = list.stream()
                         .reduce(0, (a, b) -> a + b);
        System.out.println("Sum of list: " + listSum); // 150

        // Using method reference
        int sumWithRef = list.stream()
                            .reduce(0, Integer::sum);
        System.out.println("Sum with method reference: " + sumWithRef); // 150

        // Using mapToInt and sum()
        int sumWithMapToInt = list.stream()
                                  .mapToInt(Integer::intValue)
                                  .sum();
        System.out.println("Sum with mapToInt: " + sumWithMapToInt); // 150
    }

    // 6. Currying approach
    public static void curryingSum() {
        // Function that returns another function
        Function<Integer, Function<Integer, Integer>> curriedSum =
            a -> b -> a + b;

        // Partial application
        Function<Integer, Integer> add10 = curriedSum.apply(10);

        System.out.println("10 + 5 = " + add10.apply(5)); // 15
        System.out.println("10 + 20 = " + add10.apply(20)); // 30

        // Direct application
        int result = curriedSum.apply(25).apply(35);
        System.out.println("25 + 35 = " + result); // 60
    }

    // 7. Variable arguments sum
    public static void varArgsSum() {
        // Function to sum variable number of arguments
        Function<int[], Integer> sumAll = arr ->
            Arrays.stream(arr).reduce(0, (a, b) -> a + b);

        int result1 = sumAll.apply(new int[]{1, 2, 3, 4, 5});
        System.out.println("Sum of [1,2,3,4,5]: " + result1); // 15

        int result2 = sumAll.apply(new int[]{10, 20, 30});
        System.out.println("Sum of [10,20,30]: " + result2); // 60
    }

    // 8. Supplier for constant sum
    public static void supplierSum() {
        int x = 100;
        int y = 200;

        // Supplier that returns the sum
        Supplier<Integer> sumSupplier = () -> x + y;

        System.out.println("Sum from Supplier: " + sumSupplier.get()); // 300
    }

    // 9. Consumer to print sum
    public static void consumerSum() {
        BiConsumer<Integer, Integer> printSum =
            (a, b) -> System.out.println("Sum of " + a + " and " + b + " = " + (a + b));

        printSum.accept(45, 55); // Sum of 45 and 55 = 100
    }

    // 10. Recursive sum using lambda (advanced)
    static class Recursive {
        static Function<Integer, Function<Integer, Integer>> recursiveSum;

        static {
            recursiveSum = a -> b -> {
                if (b == 0) return a;
                return recursiveSum.apply(a + 1).apply(b - 1);
            };
        }
    }

    public static void recursiveSum() {
        int result = Recursive.recursiveSum.apply(5).apply(3);
        System.out.println("Recursive sum of 5 and 3: " + result); // 8
    }

    // 11. Generic sum method
    public static <T extends Number> double genericSum(T a, T b,
                                                       BiFunction<T, T, Double> sumFunction) {
        return sumFunction.apply(a, b);
    }

    public static void testGenericSum() {
        BiFunction<Integer, Integer, Double> intSum =
            (a, b) -> (double)(a + b);

        BiFunction<Double, Double, Double> doubleSum =
            (a, b) -> a + b;

        System.out.println("Generic int sum: " + genericSum(10, 20, intSum)); // 30.0
        System.out.println("Generic double sum: " + genericSum(10.5, 20.3, doubleSum)); // 30.8
    }

    // 12. Parallel sum for large arrays
    public static void parallelSum() {
        int[] largeArray = IntStream.rangeClosed(1, 1000000).toArray();

        long startTime = System.currentTimeMillis();
        int sequentialSum = Arrays.stream(largeArray)
                                  .reduce(0, (a, b) -> a + b);
        long sequentialTime = System.currentTimeMillis() - startTime;

        startTime = System.currentTimeMillis();
        int parallelSum = Arrays.stream(largeArray)
                               .parallel()
                               .reduce(0, (a, b) -> a + b);
        long parallelTime = System.currentTimeMillis() - startTime;

        System.out.println("Sequential sum: " + sequentialSum + " (Time: " + sequentialTime + "ms)");
        System.out.println("Parallel sum: " + parallelSum + " (Time: " + parallelTime + "ms)");
    }

    public static void main(String[] args) {
        System.out.println("=== Basic Lambda Sum ===");
        basicLambdaSum();

        System.out.println("\n=== Custom Interface Sum ===");
        customInterfaceSum();

        System.out.println("\n=== IntBinaryOperator Sum ===");
        intBinaryOperatorSum();

        System.out.println("\n=== BinaryOperator Sum ===");
        binaryOperatorSum();

        System.out.println("\n=== Stream Sum ===");
        streamSum();

        System.out.println("\n=== Currying Sum ===");
        curryingSum();

        System.out.println("\n=== Variable Arguments Sum ===");
        varArgsSum();

        System.out.println("\n=== Supplier Sum ===");
        supplierSum();

        System.out.println("\n=== Consumer Sum ===");
        consumerSum();

        System.out.println("\n=== Recursive Sum ===");
        recursiveSum();

        System.out.println("\n=== Generic Sum ===");
        testGenericSum();

        System.out.println("\n=== Parallel Sum ===");
        parallelSum();
    }
}
```

### Practical Examples with Lambda Sum

```java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;

public class PracticalLambdaSum {

    // 1. Calculator Application
    public static class CalculatorApp {
        private Map<String, BinaryOperator<Double>> operations = new HashMap<>();

        public CalculatorApp() {
            operations.put("+", (a, b) -> a + b);
            operations.put("-", (a, b) -> a - b);
            operations.put("*", (a, b) -> a * b);
            operations.put("/", (a, b) -> b != 0 ? a / b : Double.NaN);
            operations.put("^", (a, b) -> Math.pow(a, b));
        }

        public double calculate(double a, String operator, double b) {
            return operations.getOrDefault(operator, (x, y) -> Double.NaN).apply(a, b);
        }

        public static void demo() {
            CalculatorApp calc = new CalculatorApp();
            System.out.println("5 + 3 = " + calc.calculate(5, "+", 3)); // 8.0
            System.out.println("10 - 4 = " + calc.calculate(10, "-", 4)); // 6.0
            System.out.println("6 * 7 = " + calc.calculate(6, "*", 7)); // 42.0
            System.out.println("2 ^ 3 = " + calc.calculate(2, "^", 3)); // 8.0
        }
    }

    // 2. Matrix Operations
    public static class MatrixOperations {
        public static int[][] addMatrices(int[][] matrix1, int[][] matrix2) {
            BinaryOperator<Integer> sum = (a, b) -> a + b;

            int rows = matrix1.length;
            int cols = matrix1[0].length;
            int[][] result = new int[rows][cols];

            IntStream.range(0, rows).forEach(i ->
                IntStream.range(0, cols).forEach(j ->
                    result[i][j] = sum.apply(matrix1[i][j], matrix2[i][j])
                )
            );

            return result;
        }

        public static void demo() {
            int[][] matrix1 = {{1, 2}, {3, 4}};
            int[][] matrix2 = {{5, 6}, {7, 8}};

            int[][] result = addMatrices(matrix1, matrix2);

            System.out.println("Matrix Addition Result:");
            Arrays.stream(result)
                  .map(Arrays::toString)
                  .forEach(System.out::println);
            // [6, 8]
            // [10, 12]
        }
    }

    // 3. Banking Application
    public static class BankAccount {
        private double balance;
        private List<Transaction> transactions = new ArrayList<>();

        class Transaction {
            String type;
            double amount;

            Transaction(String type, double amount) {
                this.type = type;
                this.amount = amount;
            }
        }

        public BankAccount(double initialBalance) {
            this.balance = initialBalance;
        }

        public void performTransaction(String type, double amount,
                                      BiFunction<Double, Double, Double> operation) {
            balance = operation.apply(balance, amount);
            transactions.add(new Transaction(type, amount));
        }

        public void deposit(double amount) {
            performTransaction("DEPOSIT", amount, (bal, amt) -> bal + amt);
        }

        public void withdraw(double amount) {
            performTransaction("WITHDRAW", amount, (bal, amt) -> bal - amt);
        }

        public double calculateTotalDeposits() {
            return transactions.stream()
                .filter(t -> t.type.equals("DEPOSIT"))
                .map(t -> t.amount)
                .reduce(0.0, (a, b) -> a + b);
        }

        public static void demo() {
            BankAccount account = new BankAccount(1000);

            account.deposit(500);
            account.withdraw(200);
            account.deposit(300);

            System.out.println("Current Balance: " + account.balance); // 1600
            System.out.println("Total Deposits: " + account.calculateTotalDeposits()); // 800
        }
    }

    public static void main(String[] args) {
        System.out.println("=== Calculator Demo ===");
        CalculatorApp.demo();

        System.out.println("\n=== Matrix Operations Demo ===");
        MatrixOperations.demo();

        System.out.println("\n=== Banking Demo ===");
        BankAccount.demo();
    }
}
```

## Summary

### Key Points for Anagram Grouping:
1. **Stream API**: أكثر إيجازاً وقابلية للقراءة
2. **Traditional**: أفضل للفهم والتحكم الكامل
3. **Sorted Key**: الطريقة الأكثر شيوعاً
4. **Frequency Count**: أسرع من Sorting
5. **Prime Numbers**: طريقة رياضية مبتكرة

### Key Points for Lambda Sum:
1. **BiFunction<Integer, Integer, Integer>**: للأغراض العامة
2. **IntBinaryOperator**: أفضل للـ primitive int
3. **Stream.reduce()**: لجمع collections
4. **Method References**: Integer::sum أنظف من (a, b) -> a + b
5. **Currying**: للـ partial application