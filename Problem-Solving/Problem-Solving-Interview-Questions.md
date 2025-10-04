# أسئلة مقابلات Problem Solving

## المحتويات
1. [Arrays & Strings](#arrays--strings)
2. [Linked Lists](#linked-lists)
3. [Stacks & Queues](#stacks--queues)
4. [Trees & Binary Search Trees](#trees--binary-search-trees)
5. [Sorting & Searching](#sorting--searching)
6. [Recursion & Dynamic Programming](#recursion--dynamic-programming)
7. [Hash Tables](#hash-tables)
8. [Two Pointers & Sliding Window](#two-pointers--sliding-window)
9. [Complexity Analysis](#complexity-analysis)

---

## Arrays & Strings

### 1. اعكس String

**السؤال:** اكتب function لعكس string.

**مثال:**
```
Input: "hello"
Output: "olleh"
```

**الحل:**

```java
// الطريقة 1: باستخدام StringBuilder
public static String reverseString1(String str) {
    return new StringBuilder(str).reverse().toString();
}

// الطريقة 2: باستخدام Two Pointers
public static String reverseString2(String str) {
    char[] chars = str.toCharArray();
    int left = 0;
    int right = chars.length - 1;

    while (left < right) {
        // Swap
        char temp = chars[left];
        chars[left] = chars[right];
        chars[right] = temp;

        left++;
        right--;
    }

    return new String(chars);
}

// الطريقة 3: باستخدام Recursion
public static String reverseString3(String str) {
    if (str.isEmpty()) {
        return str;
    }
    return reverseString3(str.substring(1)) + str.charAt(0);
}

// اختبار
public static void main(String[] args) {
    System.out.println(reverseString1("hello")); // olleh
    System.out.println(reverseString2("world")); // dlrow
    System.out.println(reverseString3("java"));  // avaj
}
```

**Complexity:**
- Time: O(n)
- Space: O(n)

---

### 2. تحقق من Palindrome

**السؤال:** تحقق إذا كان string هو palindrome (يُقرأ بنفس الطريقة من اليسار واليمين).

**مثال:**
```
Input: "racecar"
Output: true

Input: "hello"
Output: false
```

**الحل:**

```java
public static boolean isPalindrome(String str) {
    // تنظيف String من المسافات وجعلها lowercase
    str = str.toLowerCase().replaceAll("[^a-z0-9]", "");

    int left = 0;
    int right = str.length() - 1;

    while (left < right) {
        if (str.charAt(left) != str.charAt(right)) {
            return false;
        }
        left++;
        right--;
    }

    return true;
}

// اختبار
public static void main(String[] args) {
    System.out.println(isPalindrome("racecar"));        // true
    System.out.println(isPalindrome("A man a plan a canal Panama")); // true
    System.out.println(isPalindrome("hello"));          // false
}
```

**Complexity:**
- Time: O(n)
- Space: O(1)

---

### 3. إيجاد أول حرف غير مكرر

**السؤال:** أوجد أول حرف غير مكرر في string.

**مثال:**
```
Input: "leetcode"
Output: 'l'

Input: "loveleetcode"
Output: 'v'
```

**الحل:**

```java
public static char firstNonRepeatingChar(String str) {
    // حساب frequency لكل حرف
    Map<Character, Integer> frequency = new HashMap<>();

    for (char c : str.toCharArray()) {
        frequency.put(c, frequency.getOrDefault(c, 0) + 1);
    }

    // إيجاد أول حرف frequency له 1
    for (char c : str.toCharArray()) {
        if (frequency.get(c) == 1) {
            return c;
        }
    }

    return '_'; // لا يوجد
}

// اختبار
public static void main(String[] args) {
    System.out.println(firstNonRepeatingChar("leetcode"));     // l
    System.out.println(firstNonRepeatingChar("loveleetcode")); // v
    System.out.println(firstNonRepeatingChar("aabb"));         // _
}
```

**Complexity:**
- Time: O(n)
- Space: O(n)

---

### 4. إيجاد أكبر رقمين في Array

**السؤال:** أوجد أكبر رقمين في array.

**مثال:**
```
Input: [12, 35, 1, 10, 34, 1]
Output: [35, 34]
```

**الحل:**

```java
public static int[] findTwoLargest(int[] arr) {
    if (arr.length < 2) {
        return new int[]{};
    }

    int first = Integer.MIN_VALUE;
    int second = Integer.MIN_VALUE;

    for (int num : arr) {
        if (num > first) {
            second = first;
            first = num;
        } else if (num > second && num != first) {
            second = num;
        }
    }

    return new int[]{first, second};
}

// اختبار
public static void main(String[] args) {
    int[] arr = {12, 35, 1, 10, 34, 1};
    int[] result = findTwoLargest(arr);
    System.out.println(Arrays.toString(result)); // [35, 34]
}
```

**Complexity:**
- Time: O(n)
- Space: O(1)

---

### 5. نقل كل الأصفار للنهاية

**السؤال:** انقل كل الأصفار لنهاية الarray مع الحفاظ على ترتيب الأرقام الأخرى.

**مثال:**
```
Input: [0, 1, 0, 3, 12]
Output: [1, 3, 12, 0, 0]
```

**الحل:**

```java
public static void moveZeros(int[] arr) {
    int nonZeroIndex = 0;

    // نقل كل الأرقام غير الصفرية للبداية
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] != 0) {
            arr[nonZeroIndex] = arr[i];
            nonZeroIndex++;
        }
    }

    // ملء الباقي بالأصفار
    while (nonZeroIndex < arr.length) {
        arr[nonZeroIndex] = 0;
        nonZeroIndex++;
    }
}

// اختبار
public static void main(String[] args) {
    int[] arr = {0, 1, 0, 3, 12};
    moveZeros(arr);
    System.out.println(Arrays.toString(arr)); // [1, 3, 12, 0, 0]
}
```

**Complexity:**
- Time: O(n)
- Space: O(1)

---

### 6. إيجاد Missing Number

**السؤال:** لديك array من 0 إلى n، رقم واحد ناقص. أوجده.

**مثال:**
```
Input: [3, 0, 1]
Output: 2

Input: [0, 1]
Output: 2
```

**الحل:**

```java
// الطريقة 1: باستخدام مجموع الأرقام
public static int missingNumber1(int[] arr) {
    int n = arr.length;
    int expectedSum = n * (n + 1) / 2;
    int actualSum = 0;

    for (int num : arr) {
        actualSum += num;
    }

    return expectedSum - actualSum;
}

// الطريقة 2: باستخدام XOR
public static int missingNumber2(int[] arr) {
    int n = arr.length;
    int xor = 0;

    // XOR كل الأرقام من 0 إلى n
    for (int i = 0; i <= n; i++) {
        xor ^= i;
    }

    // XOR مع أرقام الarray
    for (int num : arr) {
        xor ^= num;
    }

    return xor;
}

// اختبار
public static void main(String[] args) {
    System.out.println(missingNumber1(new int[]{3, 0, 1})); // 2
    System.out.println(missingNumber2(new int[]{0, 1}));    // 2
}
```

**Complexity:**
- Time: O(n)
- Space: O(1)

---

## Linked Lists

### 7. اعكس Linked List

**السؤال:** اعكس linked list.

**مثال:**
```
Input: 1 -> 2 -> 3 -> 4 -> 5
Output: 5 -> 4 -> 3 -> 2 -> 1
```

**الحل:**

```java
class Node {
    int data;
    Node next;

    Node(int data) {
        this.data = data;
        this.next = null;
    }
}

// الطريقة 1: Iterative
public static Node reverseList(Node head) {
    Node prev = null;
    Node current = head;
    Node next = null;

    while (current != null) {
        next = current.next;  // حفظ التالي
        current.next = prev;  // عكس الاتجاه
        prev = current;       // التقدم
        current = next;
    }

    return prev; // prev الآن هو الرأس الجديد
}

// الطريقة 2: Recursive
public static Node reverseListRecursive(Node head) {
    if (head == null || head.next == null) {
        return head;
    }

    Node newHead = reverseListRecursive(head.next);
    head.next.next = head;
    head.next = null;

    return newHead;
}

// Helper method للطباعة
public static void printList(Node head) {
    Node current = head;
    while (current != null) {
        System.out.print(current.data + " -> ");
        current = current.next;
    }
    System.out.println("null");
}

// اختبار
public static void main(String[] args) {
    Node head = new Node(1);
    head.next = new Node(2);
    head.next.next = new Node(3);
    head.next.next.next = new Node(4);
    head.next.next.next.next = new Node(5);

    System.out.println("Original:");
    printList(head);

    head = reverseList(head);
    System.out.println("Reversed:");
    printList(head);
}
```

**Complexity:**
- Time: O(n)
- Space: O(1) للـ iterative، O(n) للـ recursive

---

### 8. اكتشف دورة في Linked List

**السؤال:** تحقق إذا كان هناك دورة (cycle) في linked list.

**الحل:**

```java
public static boolean hasCycle(Node head) {
    if (head == null) {
        return false;
    }

    Node slow = head;
    Node fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;           // خطوة واحدة
        fast = fast.next.next;      // خطوتان

        if (slow == fast) {
            return true; // يوجد دورة
        }
    }

    return false;
}

// اختبار
public static void main(String[] args) {
    Node head = new Node(1);
    head.next = new Node(2);
    head.next.next = new Node(3);
    head.next.next.next = new Node(4);
    head.next.next.next.next = head.next; // دورة

    System.out.println(hasCycle(head)); // true
}
```

**Complexity:**
- Time: O(n)
- Space: O(1)

---

### 9. أوجد النقطة الوسطى في Linked List

**السؤال:** أوجد العنصر الأوسط في linked list.

**الحل:**

```java
public static Node findMiddle(Node head) {
    if (head == null) {
        return null;
    }

    Node slow = head;
    Node fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }

    return slow; // slow الآن في المنتصف
}

// اختبار
public static void main(String[] args) {
    Node head = new Node(1);
    head.next = new Node(2);
    head.next.next = new Node(3);
    head.next.next.next = new Node(4);
    head.next.next.next.next = new Node(5);

    Node middle = findMiddle(head);
    System.out.println("Middle: " + middle.data); // 3
}
```

**Complexity:**
- Time: O(n)
- Space: O(1)

---

## Stacks & Queues

### 10. تحقق من Balanced Parentheses

**السؤال:** تحقق إذا كانت الأقواس متوازنة.

**مثال:**
```
Input: "{[()]}"
Output: true

Input: "{[(])}"
Output: false
```

**الحل:**

```java
public static boolean isBalanced(String str) {
    Stack<Character> stack = new Stack<>();

    for (char c : str.toCharArray()) {
        // إذا قوس فاتح، أضفه للـ stack
        if (c == '(' || c == '[' || c == '{') {
            stack.push(c);
        }
        // إذا قوس مغلق
        else if (c == ')' || c == ']' || c == '}') {
            if (stack.isEmpty()) {
                return false;
            }

            char top = stack.pop();

            if ((c == ')' && top != '(') ||
                (c == ']' && top != '[') ||
                (c == '}' && top != '{')) {
                return false;
            }
        }
    }

    return stack.isEmpty();
}

// اختبار
public static void main(String[] args) {
    System.out.println(isBalanced("{[()]}")); // true
    System.out.println(isBalanced("{[(])}")); // false
    System.out.println(isBalanced("()")); // true
    System.out.println(isBalanced("(]")); // false
}
```

**Complexity:**
- Time: O(n)
- Space: O(n)

---

### 11. نفذ Queue باستخدام Stacks

**السؤال:** نفذ queue باستخدام اثنين من stacks.

**الحل:**

```java
class QueueUsingStacks {
    private Stack<Integer> stack1; // للـ enqueue
    private Stack<Integer> stack2; // للـ dequeue

    public QueueUsingStacks() {
        stack1 = new Stack<>();
        stack2 = new Stack<>();
    }

    // إضافة عنصر
    public void enqueue(int value) {
        stack1.push(value);
    }

    // حذف عنصر
    public int dequeue() {
        if (stack2.isEmpty()) {
            // نقل كل العناصر من stack1 إلى stack2
            while (!stack1.isEmpty()) {
                stack2.push(stack1.pop());
            }
        }

        if (stack2.isEmpty()) {
            throw new RuntimeException("Queue is empty");
        }

        return stack2.pop();
    }

    // قراءة العنصر الأول بدون حذف
    public int peek() {
        if (stack2.isEmpty()) {
            while (!stack1.isEmpty()) {
                stack2.push(stack1.pop());
            }
        }

        if (stack2.isEmpty()) {
            throw new RuntimeException("Queue is empty");
        }

        return stack2.peek();
    }

    public boolean isEmpty() {
        return stack1.isEmpty() && stack2.isEmpty();
    }
}

// اختبار
public static void main(String[] args) {
    QueueUsingStacks queue = new QueueUsingStacks();
    queue.enqueue(1);
    queue.enqueue(2);
    queue.enqueue(3);

    System.out.println(queue.dequeue()); // 1
    System.out.println(queue.peek());    // 2
    System.out.println(queue.dequeue()); // 2
}
```

**Complexity:**
- enqueue: O(1)
- dequeue: O(1) amortized

---

## Trees & Binary Search Trees

### 12. ارتفاع Binary Tree

**السؤال:** أوجد ارتفاع binary tree.

**الحل:**

```java
class TreeNode {
    int data;
    TreeNode left;
    TreeNode right;

    TreeNode(int data) {
        this.data = data;
        this.left = null;
        this.right = null;
    }
}

public static int height(TreeNode root) {
    if (root == null) {
        return 0;
    }

    int leftHeight = height(root.left);
    int rightHeight = height(root.right);

    return Math.max(leftHeight, rightHeight) + 1;
}

// اختبار
public static void main(String[] args) {
    TreeNode root = new TreeNode(1);
    root.left = new TreeNode(2);
    root.right = new TreeNode(3);
    root.left.left = new TreeNode(4);
    root.left.right = new TreeNode(5);

    System.out.println("Height: " + height(root)); // 3
}
```

**Complexity:**
- Time: O(n)
- Space: O(h) حيث h هو الارتفاع

---

### 13. Binary Tree Traversals

**السؤال:** نفذ الأنواع الثلاثة من tree traversals.

**الحل:**

```java
// 1. In-Order Traversal (Left -> Root -> Right)
public static void inOrder(TreeNode root) {
    if (root != null) {
        inOrder(root.left);
        System.out.print(root.data + " ");
        inOrder(root.right);
    }
}

// 2. Pre-Order Traversal (Root -> Left -> Right)
public static void preOrder(TreeNode root) {
    if (root != null) {
        System.out.print(root.data + " ");
        preOrder(root.left);
        preOrder(root.right);
    }
}

// 3. Post-Order Traversal (Left -> Right -> Root)
public static void postOrder(TreeNode root) {
    if (root != null) {
        postOrder(root.left);
        postOrder(root.right);
        System.out.print(root.data + " ");
    }
}

// 4. Level-Order Traversal (BFS)
public static void levelOrder(TreeNode root) {
    if (root == null) return;

    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);

    while (!queue.isEmpty()) {
        TreeNode current = queue.poll();
        System.out.print(current.data + " ");

        if (current.left != null) {
            queue.offer(current.left);
        }
        if (current.right != null) {
            queue.offer(current.right);
        }
    }
}

// اختبار
public static void main(String[] args) {
    TreeNode root = new TreeNode(1);
    root.left = new TreeNode(2);
    root.right = new TreeNode(3);
    root.left.left = new TreeNode(4);
    root.left.right = new TreeNode(5);

    System.out.print("In-Order: ");
    inOrder(root);    // 4 2 5 1 3

    System.out.print("\nPre-Order: ");
    preOrder(root);   // 1 2 4 5 3

    System.out.print("\nPost-Order: ");
    postOrder(root);  // 4 5 2 3 1

    System.out.print("\nLevel-Order: ");
    levelOrder(root); // 1 2 3 4 5
}
```

---

### 14. تحقق من BST صحيح

**السؤال:** تحقق إذا كان binary tree هو binary search tree صحيح.

**الحل:**

```java
public static boolean isBST(TreeNode root) {
    return isBSTHelper(root, Long.MIN_VALUE, Long.MAX_VALUE);
}

private static boolean isBSTHelper(TreeNode node, long min, long max) {
    if (node == null) {
        return true;
    }

    // يجب أن يكون القيمة بين min و max
    if (node.data <= min || node.data >= max) {
        return false;
    }

    // تحقق من الـ subtrees
    return isBSTHelper(node.left, min, node.data) &&
           isBSTHelper(node.right, node.data, max);
}

// اختبار
public static void main(String[] args) {
    TreeNode root = new TreeNode(5);
    root.left = new TreeNode(3);
    root.right = new TreeNode(7);
    root.left.left = new TreeNode(2);
    root.left.right = new TreeNode(4);

    System.out.println(isBST(root)); // true
}
```

**Complexity:**
- Time: O(n)
- Space: O(h)

---

## Sorting & Searching

### 15. Binary Search

**السؤال:** نفذ binary search.

**الحل:**

```java
// Iterative
public static int binarySearch(int[] arr, int target) {
    int left = 0;
    int right = arr.length - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;

        if (arr[mid] == target) {
            return mid;
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return -1; // لم يُعثر عليه
}

// Recursive
public static int binarySearchRecursive(int[] arr, int target, int left, int right) {
    if (left > right) {
        return -1;
    }

    int mid = left + (right - left) / 2;

    if (arr[mid] == target) {
        return mid;
    } else if (arr[mid] < target) {
        return binarySearchRecursive(arr, target, mid + 1, right);
    } else {
        return binarySearchRecursive(arr, target, left, mid - 1);
    }
}

// اختبار
public static void main(String[] args) {
    int[] arr = {1, 3, 5, 7, 9, 11, 13};
    System.out.println(binarySearch(arr, 7));  // 3
    System.out.println(binarySearch(arr, 6));  // -1
}
```

**Complexity:**
- Time: O(log n)
- Space: O(1) للـ iterative، O(log n) للـ recursive

---

### 16. Bubble Sort

**السؤال:** نفذ bubble sort.

**الحل:**

```java
public static void bubbleSort(int[] arr) {
    int n = arr.length;

    for (int i = 0; i < n - 1; i++) {
        boolean swapped = false;

        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                // Swap
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
                swapped = true;
            }
        }

        // إذا لم يحدث swap، الarray مرتب
        if (!swapped) {
            break;
        }
    }
}

// اختبار
public static void main(String[] args) {
    int[] arr = {64, 34, 25, 12, 22, 11, 90};
    bubbleSort(arr);
    System.out.println(Arrays.toString(arr));
}
```

**Complexity:**
- Time: O(n²)
- Space: O(1)

---

### 17. Merge Sort

**السؤال:** نفذ merge sort.

**الحل:**

```java
public static void mergeSort(int[] arr) {
    if (arr.length < 2) {
        return;
    }

    int mid = arr.length / 2;
    int[] left = Arrays.copyOfRange(arr, 0, mid);
    int[] right = Arrays.copyOfRange(arr, mid, arr.length);

    mergeSort(left);
    mergeSort(right);
    merge(arr, left, right);
}

private static void merge(int[] arr, int[] left, int[] right) {
    int i = 0, j = 0, k = 0;

    while (i < left.length && j < right.length) {
        if (left[i] <= right[j]) {
            arr[k++] = left[i++];
        } else {
            arr[k++] = right[j++];
        }
    }

    while (i < left.length) {
        arr[k++] = left[i++];
    }

    while (j < right.length) {
        arr[k++] = right[j++];
    }
}

// اختبار
public static void main(String[] args) {
    int[] arr = {64, 34, 25, 12, 22, 11, 90};
    mergeSort(arr);
    System.out.println(Arrays.toString(arr));
}
```

**Complexity:**
- Time: O(n log n)
- Space: O(n)

---

## Recursion & Dynamic Programming

### 18. Fibonacci

**السؤال:** احسب الرقم الفيبوناتشي رقم n.

**الحل:**

```java
// 1. Recursive (بطيء)
public static int fibRecursive(int n) {
    if (n <= 1) {
        return n;
    }
    return fibRecursive(n - 1) + fibRecursive(n - 2);
}

// 2. Dynamic Programming - Memoization (أسرع)
public static int fibMemo(int n, Map<Integer, Integer> memo) {
    if (n <= 1) {
        return n;
    }

    if (memo.containsKey(n)) {
        return memo.get(n);
    }

    int result = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
    memo.put(n, result);
    return result;
}

// 3. Dynamic Programming - Tabulation (الأفضل)
public static int fibDP(int n) {
    if (n <= 1) {
        return n;
    }

    int[] dp = new int[n + 1];
    dp[0] = 0;
    dp[1] = 1;

    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }

    return dp[n];
}

// 4. Optimized Space
public static int fibOptimized(int n) {
    if (n <= 1) {
        return n;
    }

    int prev = 0, curr = 1;

    for (int i = 2; i <= n; i++) {
        int next = prev + curr;
        prev = curr;
        curr = next;
    }

    return curr;
}

// اختبار
public static void main(String[] args) {
    System.out.println(fibRecursive(10));           // 55
    System.out.println(fibMemo(10, new HashMap<>())); // 55
    System.out.println(fibDP(10));                  // 55
    System.out.println(fibOptimized(10));           // 55
}
```

**Complexity:**
- Recursive: Time O(2^n), Space O(n)
- Memoization: Time O(n), Space O(n)
- Tabulation: Time O(n), Space O(n)
- Optimized: Time O(n), Space O(1)

---

### 19. Factorial

**السؤال:** احسب factorial لرقم.

**الحل:**

```java
// Recursive
public static int factorialRecursive(int n) {
    if (n <= 1) {
        return 1;
    }
    return n * factorialRecursive(n - 1);
}

// Iterative
public static int factorialIterative(int n) {
    int result = 1;
    for (int i = 2; i <= n; i++) {
        result *= i;
    }
    return result;
}

// اختبار
public static void main(String[] args) {
    System.out.println(factorialRecursive(5)); // 120
    System.out.println(factorialIterative(5)); // 120
}
```

---

### 20. أبراج هانوي (Tower of Hanoi)

**السؤال:** حل مشكلة أبراج هانوي.

**الحل:**

```java
public static void towerOfHanoi(int n, char from, char to, char aux) {
    if (n == 1) {
        System.out.println("Move disk 1 from " + from + " to " + to);
        return;
    }

    // نقل n-1 أقراص من from إلى aux باستخدام to
    towerOfHanoi(n - 1, from, aux, to);

    // نقل القرص الأكبر من from إلى to
    System.out.println("Move disk " + n + " from " + from + " to " + to);

    // نقل n-1 أقراص من aux إلى to باستخدام from
    towerOfHanoi(n - 1, aux, to, from);
}

// اختبار
public static void main(String[] args) {
    towerOfHanoi(3, 'A', 'C', 'B');
}
```

**Complexity:**
- Time: O(2^n)
- Space: O(n)

---

## Hash Tables

### 21. Two Sum

**السؤال:** أوجد رقمين في array مجموعهما يساوي target.

**مثال:**
```
Input: [2, 7, 11, 15], target = 9
Output: [0, 1] (لأن 2 + 7 = 9)
```

**الحل:**

```java
public static int[] twoSum(int[] arr, int target) {
    Map<Integer, Integer> map = new HashMap<>();

    for (int i = 0; i < arr.length; i++) {
        int complement = target - arr[i];

        if (map.containsKey(complement)) {
            return new int[]{map.get(complement), i};
        }

        map.put(arr[i], i);
    }

    return new int[]{};
}

// اختبار
public static void main(String[] args) {
    int[] arr = {2, 7, 11, 15};
    int[] result = twoSum(arr, 9);
    System.out.println(Arrays.toString(result)); // [0, 1]
}
```

**Complexity:**
- Time: O(n)
- Space: O(n)

---

### 22. أوجد العناصر المكررة

**السؤال:** أوجد كل العناصر المكررة في array.

**الحل:**

```java
public static List<Integer> findDuplicates(int[] arr) {
    Map<Integer, Integer> frequency = new HashMap<>();
    List<Integer> duplicates = new ArrayList<>();

    for (int num : arr) {
        frequency.put(num, frequency.getOrDefault(num, 0) + 1);
    }

    for (Map.Entry<Integer, Integer> entry : frequency.entrySet()) {
        if (entry.getValue() > 1) {
            duplicates.add(entry.getKey());
        }
    }

    return duplicates;
}

// اختبار
public static void main(String[] args) {
    int[] arr = {1, 2, 3, 2, 4, 5, 1, 6};
    System.out.println(findDuplicates(arr)); // [1, 2]
}
```

**Complexity:**
- Time: O(n)
- Space: O(n)

---

## Two Pointers & Sliding Window

### 23. Remove Duplicates from Sorted Array

**السؤال:** احذف العناصر المكررة من sorted array.

**الحل:**

```java
public static int removeDuplicates(int[] arr) {
    if (arr.length == 0) {
        return 0;
    }

    int uniqueIndex = 0;

    for (int i = 1; i < arr.length; i++) {
        if (arr[i] != arr[uniqueIndex]) {
            uniqueIndex++;
            arr[uniqueIndex] = arr[i];
        }
    }

    return uniqueIndex + 1;
}

// اختبار
public static void main(String[] args) {
    int[] arr = {1, 1, 2, 2, 3, 4, 4, 5};
    int newLength = removeDuplicates(arr);
    System.out.println("New length: " + newLength);
    System.out.println(Arrays.toString(Arrays.copyOf(arr, newLength)));
}
```

**Complexity:**
- Time: O(n)
- Space: O(1)

---

### 24. Maximum Sum Subarray of Size K

**السؤال:** أوجد أكبر مجموع لـ subarray بحجم k.

**مثال:**
```
Input: [2, 1, 5, 1, 3, 2], k = 3
Output: 9 (من [5, 1, 3])
```

**الحل:**

```java
public static int maxSumSubarray(int[] arr, int k) {
    if (arr.length < k) {
        return -1;
    }

    // حساب مجموع الـ window الأول
    int windowSum = 0;
    for (int i = 0; i < k; i++) {
        windowSum += arr[i];
    }

    int maxSum = windowSum;

    // تحريك الـ window
    for (int i = k; i < arr.length; i++) {
        windowSum = windowSum - arr[i - k] + arr[i];
        maxSum = Math.max(maxSum, windowSum);
    }

    return maxSum;
}

// اختبار
public static void main(String[] args) {
    int[] arr = {2, 1, 5, 1, 3, 2};
    System.out.println(maxSumSubarray(arr, 3)); // 9
}
```

**Complexity:**
- Time: O(n)
- Space: O(1)

---

## Complexity Analysis

### 25. ما هو Big O Notation؟

**الإجابة:**

Big O Notation يصف كيف يزداد وقت التنفيذ أو المساحة المستخدمة مع زيادة حجم الـ input.

**الترتيب من الأسرع للأبطأ:**

```
O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2^n) < O(n!)
```

**أمثلة:**

```java
// O(1) - Constant Time
public static int getFirst(int[] arr) {
    return arr[0]; // عملية واحدة بغض النظر عن حجم الarray
}

// O(n) - Linear Time
public static int sum(int[] arr) {
    int total = 0;
    for (int num : arr) { // نمر على كل العناصر
        total += num;
    }
    return total;
}

// O(n²) - Quadratic Time
public static void printPairs(int[] arr) {
    for (int i = 0; i < arr.length; i++) {      // n iterations
        for (int j = 0; j < arr.length; j++) {  // n iterations
            System.out.println(arr[i] + ", " + arr[j]);
        }
    }
}

// O(log n) - Logarithmic Time
public static int binarySearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}

// O(n log n) - Linearithmic Time
// Merge Sort, Quick Sort (average case)

// O(2^n) - Exponential Time
public static int fibonacci(int n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}
```

**قواعد تبسيط Big O:**

1. **احذف الثوابت**
   ```
   O(2n) → O(n)
   O(500) → O(1)
   ```

2. **احذف الدرجات الأصغر**
   ```
   O(n² + n) → O(n²)
   O(n + log n) → O(n)
   ```

3. **اضرب الـ loops المتداخلة**
   ```
   for (i...) {
     for (j...) { } // O(n²)
   }
   ```

4. **اجمع الـ loops المتتالية**
   ```
   for (i...) { }  // O(n)
   for (j...) { }  // O(n)
   // Total: O(n + n) = O(n)
   ```

---

## نصائح للمقابلات

### استراتيجية حل المشكلة:

1. **افهم المشكلة**
   - اقرأ السؤال بعناية
   - اسأل أسئلة توضيحية
   - تحقق من الـ edge cases

2. **خطط للحل**
   - فكر في أكثر من approach
   - ابدأ بحل بسيط (brute force)
   - ثم حسّنه

3. **اكتب الكود**
   - تكلم بصوت عالٍ
   - اكتب كود نظيف
   - استخدم أسماء متغيرات واضحة

4. **اختبر الحل**
   - اختبر بـ edge cases
   - تحقق من الـ complexity

5. **حسّن الحل**
   - ناقش البدائل
   - حلل الـ trade-offs

### أسئلة توضيحية مهمة:

- ما حجم الـ input؟
- هل الـ array مرتب؟
- هل يوجد duplicates؟
- ماذا أفعل في حالة input فارغ؟
- هل أحتاج لتعديل الـ input الأصلي أم إنشاء واحد جديد؟

---

**حظاً موفقاً في المقابلة! 🚀**
