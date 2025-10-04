# Problem-Solving Interview Questions

## Table of Contents
1. [Arrays and Strings](#arrays-and-strings)
2. [Linked Lists](#linked-lists)
3. [Stacks and Queues](#stacks-and-queues)
4. [Trees](#trees)
5. [Sorting Algorithms](#sorting-algorithms)
6. [Searching Algorithms](#searching-algorithms)
7. [Recursion and Dynamic Programming](#recursion-and-dynamic-programming)
8. [Hash Tables](#hash-tables)
9. [Two Pointers and Sliding Window](#two-pointers-and-sliding-window)
10. [Big O Notation](#big-o-notation)

---

## Arrays and Strings

### 1. Reverse a String

**Problem**: Given a string, reverse it.

**Solution**:
```java
public class StringReversal {
    // Method 1: Using StringBuilder
    public static String reverseString(String str) {
        if (str == null || str.length() <= 1) {
            return str;
        }
        return new StringBuilder(str).reverse().toString();
    }

    // Method 2: Using character array
    public static String reverseStringManual(String str) {
        if (str == null || str.length() <= 1) {
            return str;
        }

        char[] chars = str.toCharArray();
        int left = 0;
        int right = chars.length - 1;

        while (left < right) {
            char temp = chars[left];
            chars[left] = chars[right];
            chars[right] = temp;
            left++;
            right--;
        }

        return new String(chars);
    }

    public static void main(String[] args) {
        String test = "Hello World";
        System.out.println("Original: " + test);
        System.out.println("Reversed: " + reverseString(test));
    }
}
```

**Explanation**:
- Use two pointers approach, one at the start and one at the end
- Swap characters and move pointers toward the center
- Time Complexity: O(n), Space Complexity: O(n) for the char array

---

### 2. Check if String is Palindrome

**Problem**: Determine if a string reads the same forward and backward.

**Solution**:
```java
public class PalindromeChecker {
    public static boolean isPalindrome(String str) {
        if (str == null || str.length() <= 1) {
            return true;
        }

        // Convert to lowercase and remove non-alphanumeric characters
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

    public static void main(String[] args) {
        System.out.println(isPalindrome("A man, a plan, a canal: Panama")); // true
        System.out.println(isPalindrome("race a car")); // false
        System.out.println(isPalindrome("racecar")); // true
    }
}
```

**Explanation**:
- Clean the string by removing special characters and converting to lowercase
- Use two pointers to compare characters from both ends
- Time Complexity: O(n), Space Complexity: O(n) for cleaned string

---

### 3. Two Sum Problem

**Problem**: Find two numbers in an array that add up to a target value.

**Solution**:
```java
import java.util.HashMap;
import java.util.Arrays;

public class TwoSum {
    public static int[] twoSum(int[] nums, int target) {
        HashMap<Integer, Integer> map = new HashMap<>();

        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];

            if (map.containsKey(complement)) {
                return new int[] { map.get(complement), i };
            }

            map.put(nums[i], i);
        }

        return new int[] {}; // No solution found
    }

    public static void main(String[] args) {
        int[] nums = {2, 7, 11, 15};
        int target = 9;
        int[] result = twoSum(nums, target);
        System.out.println("Indices: " + Arrays.toString(result)); // [0, 1]
    }
}
```

**Explanation**:
- Use a HashMap to store visited numbers and their indices
- For each number, check if its complement (target - number) exists in the map
- Time Complexity: O(n), Space Complexity: O(n)

---

### 4. Find Missing Number

**Problem**: Given an array containing n distinct numbers from 0 to n, find the missing number.

**Solution**:
```java
public class MissingNumber {
    // Method 1: Using sum formula
    public static int findMissingNumber(int[] nums) {
        int n = nums.length;
        int expectedSum = n * (n + 1) / 2;
        int actualSum = 0;

        for (int num : nums) {
            actualSum += num;
        }

        return expectedSum - actualSum;
    }

    // Method 2: Using XOR
    public static int findMissingNumberXOR(int[] nums) {
        int result = nums.length;

        for (int i = 0; i < nums.length; i++) {
            result ^= i ^ nums[i];
        }

        return result;
    }

    public static void main(String[] args) {
        int[] nums = {3, 0, 1};
        System.out.println("Missing number: " + findMissingNumber(nums)); // 2
        System.out.println("Missing number (XOR): " + findMissingNumberXOR(nums)); // 2
    }
}
```

**Explanation**:
- **Sum Method**: Calculate expected sum and subtract actual sum
- **XOR Method**: XOR all indices and values; missing number remains
- Time Complexity: O(n), Space Complexity: O(1)

---

## Linked Lists

### 1. Reverse a Linked List

**Problem**: Reverse a singly linked list.

**Solution**:
```java
class ListNode {
    int val;
    ListNode next;

    ListNode(int val) {
        this.val = val;
        this.next = null;
    }
}

public class LinkedListReversal {
    // Iterative approach
    public static ListNode reverseList(ListNode head) {
        ListNode prev = null;
        ListNode current = head;

        while (current != null) {
            ListNode nextNode = current.next;
            current.next = prev;
            prev = current;
            current = nextNode;
        }

        return prev;
    }

    // Recursive approach
    public static ListNode reverseListRecursive(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }

        ListNode newHead = reverseListRecursive(head.next);
        head.next.next = head;
        head.next = null;

        return newHead;
    }

    public static void printList(ListNode head) {
        while (head != null) {
            System.out.print(head.val + " -> ");
            head = head.next;
        }
        System.out.println("null");
    }

    public static void main(String[] args) {
        ListNode head = new ListNode(1);
        head.next = new ListNode(2);
        head.next.next = new ListNode(3);
        head.next.next.next = new ListNode(4);

        System.out.println("Original list:");
        printList(head);

        head = reverseList(head);
        System.out.println("Reversed list:");
        printList(head);
    }
}
```

**Explanation**:
- **Iterative**: Use three pointers (prev, current, next) to reverse links
- **Recursive**: Reverse rest of list, then fix current node
- Time Complexity: O(n), Space Complexity: O(1) iterative, O(n) recursive

---

### 2. Detect Cycle in Linked List

**Problem**: Determine if a linked list has a cycle.

**Solution**:
```java
public class CycleDetection {
    public static boolean hasCycle(ListNode head) {
        if (head == null || head.next == null) {
            return false;
        }

        ListNode slow = head;
        ListNode fast = head;

        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;

            if (slow == fast) {
                return true;
            }
        }

        return false;
    }

    public static void main(String[] args) {
        ListNode head = new ListNode(3);
        head.next = new ListNode(2);
        head.next.next = new ListNode(0);
        head.next.next.next = new ListNode(-4);
        head.next.next.next.next = head.next; // Create cycle

        System.out.println("Has cycle: " + hasCycle(head)); // true
    }
}
```

**Explanation**:
- Floyd's Cycle Detection Algorithm (Tortoise and Hare)
- Slow pointer moves 1 step, fast pointer moves 2 steps
- If they meet, there's a cycle
- Time Complexity: O(n), Space Complexity: O(1)

---

### 3. Find Middle Node

**Problem**: Find the middle node of a linked list.

**Solution**:
```java
public class MiddleNode {
    public static ListNode findMiddle(ListNode head) {
        if (head == null) {
            return null;
        }

        ListNode slow = head;
        ListNode fast = head;

        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }

        return slow;
    }

    public static void main(String[] args) {
        ListNode head = new ListNode(1);
        head.next = new ListNode(2);
        head.next.next = new ListNode(3);
        head.next.next.next = new ListNode(4);
        head.next.next.next.next = new ListNode(5);

        ListNode middle = findMiddle(head);
        System.out.println("Middle node value: " + middle.val); // 3
    }
}
```

**Explanation**:
- Use slow and fast pointers
- When fast reaches the end, slow is at the middle
- Time Complexity: O(n), Space Complexity: O(1)

---

## Stacks and Queues

### 1. Balanced Parentheses

**Problem**: Check if parentheses in a string are balanced.

**Solution**:
```java
import java.util.Stack;
import java.util.HashMap;

public class BalancedParentheses {
    public static boolean isBalanced(String s) {
        Stack<Character> stack = new Stack<>();
        HashMap<Character, Character> matchingPairs = new HashMap<>();
        matchingPairs.put(')', '(');
        matchingPairs.put('}', '{');
        matchingPairs.put(']', '[');

        for (char c : s.toCharArray()) {
            if (c == '(' || c == '{' || c == '[') {
                stack.push(c);
            } else if (c == ')' || c == '}' || c == ']') {
                if (stack.isEmpty() || stack.pop() != matchingPairs.get(c)) {
                    return false;
                }
            }
        }

        return stack.isEmpty();
    }

    public static void main(String[] args) {
        System.out.println(isBalanced("()[]{}"));     // true
        System.out.println(isBalanced("([)]"));       // false
        System.out.println(isBalanced("{[()]}"));     // true
        System.out.println(isBalanced("(("));         // false
    }
}
```

**Explanation**:
- Use a stack to track opening brackets
- When encountering a closing bracket, check if it matches the top of stack
- Stack should be empty at the end for balanced parentheses
- Time Complexity: O(n), Space Complexity: O(n)

---

### 2. Implement Queue Using Stacks

**Problem**: Implement a queue using two stacks.

**Solution**:
```java
import java.util.Stack;

public class QueueUsingStacks {
    private Stack<Integer> stack1; // For enqueue
    private Stack<Integer> stack2; // For dequeue

    public QueueUsingStacks() {
        stack1 = new Stack<>();
        stack2 = new Stack<>();
    }

    public void enqueue(int x) {
        stack1.push(x);
    }

    public int dequeue() {
        if (stack2.isEmpty()) {
            while (!stack1.isEmpty()) {
                stack2.push(stack1.pop());
            }
        }

        if (stack2.isEmpty()) {
            throw new RuntimeException("Queue is empty");
        }

        return stack2.pop();
    }

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

    public static void main(String[] args) {
        QueueUsingStacks queue = new QueueUsingStacks();
        queue.enqueue(1);
        queue.enqueue(2);
        queue.enqueue(3);

        System.out.println(queue.dequeue()); // 1
        System.out.println(queue.peek());    // 2
        System.out.println(queue.dequeue()); // 2
    }
}
```

**Explanation**:
- Use two stacks: one for incoming elements, one for outgoing
- Transfer elements from stack1 to stack2 only when needed
- Amortized time complexity for dequeue: O(1)
- Time Complexity: Enqueue O(1), Dequeue O(1) amortized

---

## Trees

### 1. Tree Height (Depth)

**Problem**: Find the maximum depth of a binary tree.

**Solution**:
```java
class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;

    TreeNode(int val) {
        this.val = val;
    }
}

public class TreeHeight {
    public static int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }

        int leftDepth = maxDepth(root.left);
        int rightDepth = maxDepth(root.right);

        return Math.max(leftDepth, rightDepth) + 1;
    }

    public static void main(String[] args) {
        TreeNode root = new TreeNode(3);
        root.left = new TreeNode(9);
        root.right = new TreeNode(20);
        root.right.left = new TreeNode(15);
        root.right.right = new TreeNode(7);

        System.out.println("Tree height: " + maxDepth(root)); // 3
    }
}
```

**Explanation**:
- Recursively calculate depth of left and right subtrees
- Return the maximum depth plus 1 for the current node
- Time Complexity: O(n), Space Complexity: O(h) where h is height

---

### 2. Tree Traversals

**Problem**: Implement in-order, pre-order, and post-order traversals.

**Solution**:
```java
import java.util.ArrayList;
import java.util.List;

public class TreeTraversals {
    // In-order: Left -> Root -> Right
    public static void inorderTraversal(TreeNode root, List<Integer> result) {
        if (root == null) {
            return;
        }

        inorderTraversal(root.left, result);
        result.add(root.val);
        inorderTraversal(root.right, result);
    }

    // Pre-order: Root -> Left -> Right
    public static void preorderTraversal(TreeNode root, List<Integer> result) {
        if (root == null) {
            return;
        }

        result.add(root.val);
        preorderTraversal(root.left, result);
        preorderTraversal(root.right, result);
    }

    // Post-order: Left -> Right -> Root
    public static void postorderTraversal(TreeNode root, List<Integer> result) {
        if (root == null) {
            return;
        }

        postorderTraversal(root.left, result);
        postorderTraversal(root.right, result);
        result.add(root.val);
    }

    public static void main(String[] args) {
        TreeNode root = new TreeNode(1);
        root.left = new TreeNode(2);
        root.right = new TreeNode(3);
        root.left.left = new TreeNode(4);
        root.left.right = new TreeNode(5);

        List<Integer> inorder = new ArrayList<>();
        inorderTraversal(root, inorder);
        System.out.println("In-order: " + inorder); // [4, 2, 5, 1, 3]

        List<Integer> preorder = new ArrayList<>();
        preorderTraversal(root, preorder);
        System.out.println("Pre-order: " + preorder); // [1, 2, 4, 5, 3]

        List<Integer> postorder = new ArrayList<>();
        postorderTraversal(root, postorder);
        System.out.println("Post-order: " + postorder); // [4, 5, 2, 3, 1]
    }
}
```

**Explanation**:
- **In-order**: Left subtree, root, right subtree (gives sorted order for BST)
- **Pre-order**: Root, left subtree, right subtree (useful for copying tree)
- **Post-order**: Left subtree, right subtree, root (useful for deletion)
- Time Complexity: O(n) for all, Space Complexity: O(h) for recursion stack

---

### 3. Validate Binary Search Tree

**Problem**: Check if a binary tree is a valid BST.

**Solution**:
```java
public class ValidateBST {
    public static boolean isValidBST(TreeNode root) {
        return validate(root, null, null);
    }

    private static boolean validate(TreeNode node, Integer min, Integer max) {
        if (node == null) {
            return true;
        }

        if ((min != null && node.val <= min) || (max != null && node.val >= max)) {
            return false;
        }

        return validate(node.left, min, node.val) &&
               validate(node.right, node.val, max);
    }

    public static void main(String[] args) {
        // Valid BST
        TreeNode root1 = new TreeNode(2);
        root1.left = new TreeNode(1);
        root1.right = new TreeNode(3);
        System.out.println("Is valid BST: " + isValidBST(root1)); // true

        // Invalid BST
        TreeNode root2 = new TreeNode(5);
        root2.left = new TreeNode(1);
        root2.right = new TreeNode(4);
        root2.right.left = new TreeNode(3);
        root2.right.right = new TreeNode(6);
        System.out.println("Is valid BST: " + isValidBST(root2)); // false
    }
}
```

**Explanation**:
- For each node, ensure its value is within the valid range
- Left subtree values must be less than node value
- Right subtree values must be greater than node value
- Time Complexity: O(n), Space Complexity: O(h)

---

## Sorting Algorithms

### 1. Bubble Sort

**Problem**: Sort an array using bubble sort.

**Solution**:
```java
import java.util.Arrays;

public class BubbleSort {
    public static void bubbleSort(int[] arr) {
        int n = arr.length;
        boolean swapped;

        for (int i = 0; i < n - 1; i++) {
            swapped = false;

            for (int j = 0; j < n - i - 1; j++) {
                if (arr[j] > arr[j + 1]) {
                    // Swap
                    int temp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = temp;
                    swapped = true;
                }
            }

            // If no swaps occurred, array is sorted
            if (!swapped) {
                break;
            }
        }
    }

    public static void main(String[] args) {
        int[] arr = {64, 34, 25, 12, 22, 11, 90};
        System.out.println("Original array: " + Arrays.toString(arr));
        bubbleSort(arr);
        System.out.println("Sorted array: " + Arrays.toString(arr));
    }
}
```

**Explanation**:
- Compare adjacent elements and swap if they're in wrong order
- Repeat until no more swaps are needed
- Time Complexity: O(n²) worst/average, O(n) best case
- Space Complexity: O(1)

---

### 2. Merge Sort

**Problem**: Sort an array using merge sort.

**Solution**:
```java
import java.util.Arrays;

public class MergeSort {
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

    public static void main(String[] args) {
        int[] arr = {38, 27, 43, 3, 9, 82, 10};
        System.out.println("Original array: " + Arrays.toString(arr));
        mergeSort(arr);
        System.out.println("Sorted array: " + Arrays.toString(arr));
    }
}
```

**Explanation**:
- Divide array into two halves recursively
- Sort each half and merge them back together
- Time Complexity: O(n log n) for all cases
- Space Complexity: O(n)

---

### 3. Quick Sort

**Problem**: Sort an array using quick sort.

**Solution**:
```java
import java.util.Arrays;

public class QuickSort {
    public static void quickSort(int[] arr, int low, int high) {
        if (low < high) {
            int pivotIndex = partition(arr, low, high);

            quickSort(arr, low, pivotIndex - 1);
            quickSort(arr, pivotIndex + 1, high);
        }
    }

    private static int partition(int[] arr, int low, int high) {
        int pivot = arr[high];
        int i = low - 1;

        for (int j = low; j < high; j++) {
            if (arr[j] < pivot) {
                i++;
                // Swap arr[i] and arr[j]
                int temp = arr[i];
                arr[i] = arr[j];
                arr[j] = temp;
            }
        }

        // Swap arr[i+1] and arr[high] (pivot)
        int temp = arr[i + 1];
        arr[i + 1] = arr[high];
        arr[high] = temp;

        return i + 1;
    }

    public static void main(String[] args) {
        int[] arr = {10, 7, 8, 9, 1, 5};
        System.out.println("Original array: " + Arrays.toString(arr));
        quickSort(arr, 0, arr.length - 1);
        System.out.println("Sorted array: " + Arrays.toString(arr));
    }
}
```

**Explanation**:
- Select a pivot element and partition array around it
- Recursively sort elements before and after partition
- Time Complexity: O(n log n) average, O(n²) worst case
- Space Complexity: O(log n) for recursion stack

---

## Searching Algorithms

### 1. Binary Search

**Problem**: Find the position of a target value in a sorted array.

**Solution**:
```java
public class BinarySearch {
    // Iterative approach
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

        return -1; // Target not found
    }

    // Recursive approach
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

    public static void main(String[] args) {
        int[] arr = {2, 3, 4, 10, 40, 50, 60};
        int target = 10;

        int result = binarySearch(arr, target);
        if (result == -1) {
            System.out.println("Element not found");
        } else {
            System.out.println("Element found at index: " + result);
        }

        int resultRecursive = binarySearchRecursive(arr, target, 0, arr.length - 1);
        System.out.println("Element found at index (recursive): " + resultRecursive);
    }
}
```

**Explanation**:
- Only works on sorted arrays
- Compare target with middle element
- Eliminate half of the search space in each iteration
- Time Complexity: O(log n), Space Complexity: O(1) iterative, O(log n) recursive

---

## Recursion and Dynamic Programming

### 1. Fibonacci Sequence

**Problem**: Calculate the nth Fibonacci number.

**Solution**:
```java
import java.util.HashMap;

public class Fibonacci {
    // Method 1: Simple recursion (inefficient)
    public static int fibRecursive(int n) {
        if (n <= 1) {
            return n;
        }
        return fibRecursive(n - 1) + fibRecursive(n - 2);
    }

    // Method 2: Dynamic Programming with memoization
    private static HashMap<Integer, Integer> memo = new HashMap<>();

    public static int fibMemo(int n) {
        if (n <= 1) {
            return n;
        }

        if (memo.containsKey(n)) {
            return memo.get(n);
        }

        int result = fibMemo(n - 1) + fibMemo(n - 2);
        memo.put(n, result);
        return result;
    }

    // Method 3: Dynamic Programming with tabulation
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

    // Method 4: Space-optimized approach
    public static int fibOptimized(int n) {
        if (n <= 1) {
            return n;
        }

        int prev2 = 0;
        int prev1 = 1;
        int current = 0;

        for (int i = 2; i <= n; i++) {
            current = prev1 + prev2;
            prev2 = prev1;
            prev1 = current;
        }

        return current;
    }

    public static void main(String[] args) {
        int n = 10;
        System.out.println("Fibonacci(" + n + ") recursive: " + fibRecursive(n));
        System.out.println("Fibonacci(" + n + ") memoization: " + fibMemo(n));
        System.out.println("Fibonacci(" + n + ") DP: " + fibDP(n));
        System.out.println("Fibonacci(" + n + ") optimized: " + fibOptimized(n));
    }
}
```

**Explanation**:
- **Recursive**: Simple but exponential time O(2^n)
- **Memoization**: Top-down DP, O(n) time, O(n) space
- **Tabulation**: Bottom-up DP, O(n) time, O(n) space
- **Optimized**: Only keep last two values, O(n) time, O(1) space

---

### 2. Factorial

**Problem**: Calculate factorial of a number.

**Solution**:
```java
public class Factorial {
    // Recursive approach
    public static long factorialRecursive(int n) {
        if (n <= 1) {
            return 1;
        }
        return n * factorialRecursive(n - 1);
    }

    // Iterative approach
    public static long factorialIterative(int n) {
        long result = 1;
        for (int i = 2; i <= n; i++) {
            result *= i;
        }
        return result;
    }

    // Tail recursive approach
    public static long factorialTailRecursive(int n, long accumulator) {
        if (n <= 1) {
            return accumulator;
        }
        return factorialTailRecursive(n - 1, n * accumulator);
    }

    public static void main(String[] args) {
        int n = 5;
        System.out.println("Factorial(" + n + ") recursive: " + factorialRecursive(n));
        System.out.println("Factorial(" + n + ") iterative: " + factorialIterative(n));
        System.out.println("Factorial(" + n + ") tail recursive: " +
                          factorialTailRecursive(n, 1));
    }
}
```

**Explanation**:
- **Recursive**: n! = n * (n-1)!
- **Iterative**: Multiply all numbers from 1 to n
- **Tail Recursive**: Uses accumulator for optimization
- Time Complexity: O(n), Space Complexity: O(n) recursive, O(1) iterative

---

### 3. Climbing Stairs

**Problem**: You're climbing stairs with n steps. You can climb 1 or 2 steps at a time. How many distinct ways can you climb to the top?

**Solution**:
```java
public class ClimbingStairs {
    // Dynamic Programming approach
    public static int climbStairs(int n) {
        if (n <= 2) {
            return n;
        }

        int[] dp = new int[n + 1];
        dp[1] = 1;
        dp[2] = 2;

        for (int i = 3; i <= n; i++) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }

        return dp[n];
    }

    // Space-optimized approach
    public static int climbStairsOptimized(int n) {
        if (n <= 2) {
            return n;
        }

        int prev2 = 1;
        int prev1 = 2;
        int current = 0;

        for (int i = 3; i <= n; i++) {
            current = prev1 + prev2;
            prev2 = prev1;
            prev1 = current;
        }

        return current;
    }

    public static void main(String[] args) {
        int n = 5;
        System.out.println("Ways to climb " + n + " stairs: " + climbStairs(n));
        System.out.println("Ways (optimized): " + climbStairsOptimized(n));
    }
}
```

**Explanation**:
- Similar to Fibonacci sequence
- To reach step n, you can come from step n-1 or n-2
- dp[i] = dp[i-1] + dp[i-2]
- Time Complexity: O(n), Space Complexity: O(1) optimized

---

## Hash Tables

### 1. First Non-Repeating Character

**Problem**: Find the first non-repeating character in a string.

**Solution**:
```java
import java.util.HashMap;
import java.util.LinkedHashMap;

public class FirstNonRepeatingChar {
    public static char firstUniqChar(String s) {
        HashMap<Character, Integer> charCount = new HashMap<>();

        // Count occurrences
        for (char c : s.toCharArray()) {
            charCount.put(c, charCount.getOrDefault(c, 0) + 1);
        }

        // Find first character with count 1
        for (char c : s.toCharArray()) {
            if (charCount.get(c) == 1) {
                return c;
            }
        }

        return '\0'; // No unique character found
    }

    public static void main(String[] args) {
        String s = "leetcode";
        char result = firstUniqChar(s);
        if (result != '\0') {
            System.out.println("First unique character: " + result); // 'l'
        } else {
            System.out.println("No unique character found");
        }
    }
}
```

**Explanation**:
- Use HashMap to count character frequencies
- Iterate through string again to find first character with count 1
- Time Complexity: O(n), Space Complexity: O(k) where k is alphabet size

---

### 2. Group Anagrams

**Problem**: Group strings that are anagrams of each other.

**Solution**:
```java
import java.util.*;

public class GroupAnagrams {
    public static List<List<String>> groupAnagrams(String[] strs) {
        HashMap<String, List<String>> map = new HashMap<>();

        for (String str : strs) {
            char[] chars = str.toCharArray();
            Arrays.sort(chars);
            String sorted = new String(chars);

            if (!map.containsKey(sorted)) {
                map.put(sorted, new ArrayList<>());
            }
            map.get(sorted).add(str);
        }

        return new ArrayList<>(map.values());
    }

    public static void main(String[] args) {
        String[] strs = {"eat", "tea", "tan", "ate", "nat", "bat"};
        List<List<String>> result = groupAnagrams(strs);

        System.out.println("Grouped anagrams:");
        for (List<String> group : result) {
            System.out.println(group);
        }
        // Output: [[eat, tea, ate], [tan, nat], [bat]]
    }
}
```

**Explanation**:
- Sort each string to create a key
- Use HashMap where key is sorted string and value is list of anagrams
- Time Complexity: O(n * k log k) where n is number of strings, k is max length
- Space Complexity: O(n * k)

---

### 3. Two Sum (Hash Table Approach)

**Problem**: Find indices of two numbers that add up to target (covered earlier with detailed explanation).

**Solution**:
```java
import java.util.HashMap;

public class TwoSumHashTable {
    public static int[] twoSum(int[] nums, int target) {
        HashMap<Integer, Integer> map = new HashMap<>();

        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];

            if (map.containsKey(complement)) {
                return new int[] { map.get(complement), i };
            }

            map.put(nums[i], i);
        }

        return new int[] {};
    }

    public static void main(String[] args) {
        int[] nums = {2, 7, 11, 15};
        int target = 9;
        int[] result = twoSum(nums, target);
        System.out.println("Indices: [" + result[0] + ", " + result[1] + "]");
    }
}
```

**Explanation**:
- Hash table provides O(1) lookup time
- Store each number and its index as we iterate
- Check if complement exists in map
- Time Complexity: O(n), Space Complexity: O(n)

---

## Two Pointers and Sliding Window

### 1. Two Pointers - Container with Most Water

**Problem**: Find two lines that together with x-axis form a container that holds the most water.

**Solution**:
```java
public class ContainerWithMostWater {
    public static int maxArea(int[] height) {
        int left = 0;
        int right = height.length - 1;
        int maxArea = 0;

        while (left < right) {
            int width = right - left;
            int currentHeight = Math.min(height[left], height[right]);
            int currentArea = width * currentHeight;

            maxArea = Math.max(maxArea, currentArea);

            // Move the pointer pointing to shorter line
            if (height[left] < height[right]) {
                left++;
            } else {
                right--;
            }
        }

        return maxArea;
    }

    public static void main(String[] args) {
        int[] height = {1, 8, 6, 2, 5, 4, 8, 3, 7};
        System.out.println("Max area: " + maxArea(height)); // 49
    }
}
```

**Explanation**:
- Start with widest container (pointers at both ends)
- Move the pointer at the shorter line inward
- Track maximum area encountered
- Time Complexity: O(n), Space Complexity: O(1)

---

### 2. Sliding Window - Maximum Sum Subarray

**Problem**: Find maximum sum of a subarray of size k.

**Solution**:
```java
public class MaxSumSubarray {
    public static int maxSumSubarray(int[] arr, int k) {
        if (arr.length < k) {
            return -1;
        }

        // Calculate sum of first window
        int windowSum = 0;
        for (int i = 0; i < k; i++) {
            windowSum += arr[i];
        }

        int maxSum = windowSum;

        // Slide the window
        for (int i = k; i < arr.length; i++) {
            windowSum = windowSum - arr[i - k] + arr[i];
            maxSum = Math.max(maxSum, windowSum);
        }

        return maxSum;
    }

    public static void main(String[] args) {
        int[] arr = {2, 1, 5, 1, 3, 2};
        int k = 3;
        System.out.println("Max sum of subarray size " + k + ": " +
                          maxSumSubarray(arr, k)); // 9 (5+1+3)
    }
}
```

**Explanation**:
- Calculate sum of first k elements
- Slide window by removing leftmost and adding rightmost element
- Track maximum sum seen
- Time Complexity: O(n), Space Complexity: O(1)

---

### 3. Sliding Window - Longest Substring Without Repeating Characters

**Problem**: Find the length of longest substring without repeating characters.

**Solution**:
```java
import java.util.HashSet;

public class LongestSubstringWithoutRepeating {
    public static int lengthOfLongestSubstring(String s) {
        HashSet<Character> charSet = new HashSet<>();
        int left = 0;
        int maxLength = 0;

        for (int right = 0; right < s.length(); right++) {
            while (charSet.contains(s.charAt(right))) {
                charSet.remove(s.charAt(left));
                left++;
            }

            charSet.add(s.charAt(right));
            maxLength = Math.max(maxLength, right - left + 1);
        }

        return maxLength;
    }

    public static void main(String[] args) {
        String s = "abcabcbb";
        System.out.println("Longest substring length: " +
                          lengthOfLongestSubstring(s)); // 3 (abc)

        s = "pwwkew";
        System.out.println("Longest substring length: " +
                          lengthOfLongestSubstring(s)); // 3 (wke)
    }
}
```

**Explanation**:
- Use sliding window with HashSet to track characters
- Expand window by moving right pointer
- Shrink window from left when duplicate is found
- Time Complexity: O(n), Space Complexity: O(min(n, m)) where m is charset size

---

## Big O Notation

### Understanding Time and Space Complexity

Big O notation describes the upper bound of an algorithm's growth rate. It helps us understand how an algorithm scales with input size.

### Common Time Complexities

#### O(1) - Constant Time
```java
public class ConstantTime {
    // Accessing array element by index
    public static int getElement(int[] arr, int index) {
        return arr[index]; // O(1)
    }

    // Hash table lookup
    public static String getValue(HashMap<String, String> map, String key) {
        return map.get(key); // O(1) average case
    }
}
```

**Examples**:
- Array access by index
- Hash table insertion/lookup (average case)
- Stack push/pop
- Getting first element of linked list

---

#### O(log n) - Logarithmic Time
```java
public class LogarithmicTime {
    // Binary search
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
}
```

**Examples**:
- Binary search
- Balanced binary search tree operations
- Finding power of a number (using divide and conquer)

---

#### O(n) - Linear Time
```java
public class LinearTime {
    // Linear search
    public static int linearSearch(int[] arr, int target) {
        for (int i = 0; i < arr.length; i++) {
            if (arr[i] == target) {
                return i;
            }
        }
        return -1;
    }

    // Sum of array elements
    public static int sum(int[] arr) {
        int total = 0;
        for (int num : arr) {
            total += num;
        }
        return total;
    }
}
```

**Examples**:
- Linear search
- Traversing an array or linked list
- Finding min/max in unsorted array

---

#### O(n log n) - Linearithmic Time
```java
public class LinearithmicTime {
    // Merge sort
    public static void mergeSort(int[] arr, int left, int right) {
        if (left < right) {
            int mid = left + (right - left) / 2;

            mergeSort(arr, left, mid);
            mergeSort(arr, mid + 1, right);
            merge(arr, left, mid, right);
        }
    }

    private static void merge(int[] arr, int left, int mid, int right) {
        // Merge implementation (O(n) operation)
        // ... code omitted for brevity
    }
}
```

**Examples**:
- Merge sort
- Quick sort (average case)
- Heap sort
- Efficient sorting algorithms

---

#### O(n²) - Quadratic Time
```java
public class QuadraticTime {
    // Bubble sort
    public static void bubbleSort(int[] arr) {
        for (int i = 0; i < arr.length - 1; i++) {
            for (int j = 0; j < arr.length - i - 1; j++) {
                if (arr[j] > arr[j + 1]) {
                    int temp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = temp;
                }
            }
        }
    }

    // Finding all pairs in array
    public static void printAllPairs(int[] arr) {
        for (int i = 0; i < arr.length; i++) {
            for (int j = i + 1; j < arr.length; j++) {
                System.out.println(arr[i] + ", " + arr[j]);
            }
        }
    }
}
```

**Examples**:
- Bubble sort, selection sort, insertion sort
- Nested loops iterating through array
- Comparing all pairs of elements

---

#### O(2^n) - Exponential Time
```java
public class ExponentialTime {
    // Naive recursive Fibonacci
    public static int fibonacci(int n) {
        if (n <= 1) {
            return n;
        }
        return fibonacci(n - 1) + fibonacci(n - 2);
    }

    // Generate all subsets of a set
    public static List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        backtrack(nums, 0, new ArrayList<>(), result);
        return result;
    }

    private static void backtrack(int[] nums, int start,
                                  List<Integer> current,
                                  List<List<Integer>> result) {
        result.add(new ArrayList<>(current));
        for (int i = start; i < nums.length; i++) {
            current.add(nums[i]);
            backtrack(nums, i + 1, current, result);
            current.remove(current.size() - 1);
        }
    }
}
```

**Examples**:
- Recursive Fibonacci (without memoization)
- Generating all subsets
- Solving Tower of Hanoi

---

### Space Complexity

Space complexity measures the memory used by an algorithm relative to input size.

```java
public class SpaceComplexity {
    // O(1) - Constant space
    public static void swapElements(int[] arr, int i, int j) {
        int temp = arr[i]; // Only uses a few variables
        arr[i] = arr[j];
        arr[j] = temp;
    }

    // O(n) - Linear space
    public static int[] copyArray(int[] arr) {
        int[] copy = new int[arr.length]; // New array of size n
        for (int i = 0; i < arr.length; i++) {
            copy[i] = arr[i];
        }
        return copy;
    }

    // O(n) - Recursive call stack
    public static int factorial(int n) {
        if (n <= 1) return 1;
        return n * factorial(n - 1); // n recursive calls on stack
    }
}
```

---

### Comparison Table

| Complexity | Name | Example |
|------------|------|---------|
| O(1) | Constant | Array access, hash table lookup |
| O(log n) | Logarithmic | Binary search |
| O(n) | Linear | Linear search, array traversal |
| O(n log n) | Linearithmic | Merge sort, quick sort |
| O(n²) | Quadratic | Bubble sort, nested loops |
| O(2^n) | Exponential | Recursive Fibonacci |
| O(n!) | Factorial | Generating permutations |

---

### Best, Average, and Worst Case

```java
public class ComplexityAnalysis {
    // Example: Quick Sort
    // Best case: O(n log n) - pivot divides array evenly
    // Average case: O(n log n)
    // Worst case: O(n²) - pivot is always smallest/largest element

    // Example: Linear Search
    public static int linearSearch(int[] arr, int target) {
        for (int i = 0; i < arr.length; i++) {
            if (arr[i] == target) {
                return i;
            }
        }
        return -1;
    }
    // Best case: O(1) - element is at first position
    // Average case: O(n/2) = O(n)
    // Worst case: O(n) - element is at last position or not present
}
```

---

### Rules for Calculating Big O

1. **Drop Constants**: O(2n) → O(n)
2. **Drop Non-Dominant Terms**: O(n² + n) → O(n²)
3. **Different Inputs Use Different Variables**: O(a + b) for two arrays
4. **Multiplication for Nested Operations**: O(a * b) for nested loops

```java
public class BigORules {
    // O(n + m), not O(n)
    public static void processTwoArrays(int[] arr1, int[] arr2) {
        for (int num : arr1) {
            System.out.println(num);
        }
        for (int num : arr2) {
            System.out.println(num);
        }
    }

    // O(n * m), not O(n²)
    public static void nestedLoops(int[] arr1, int[] arr2) {
        for (int num1 : arr1) {
            for (int num2 : arr2) {
                System.out.println(num1 + ", " + num2);
            }
        }
    }
}
```

---

## Conclusion

This documentation covers fundamental problem-solving patterns and algorithms essential for technical interviews. Practice implementing these solutions, understand their complexities, and learn to recognize patterns in new problems.

### Key Takeaways:

1. **Master the basics**: Arrays, strings, linked lists, and hash tables
2. **Understand complexity**: Always analyze time and space complexity
3. **Recognize patterns**: Two pointers, sliding window, recursion, DP
4. **Practice regularly**: Implement solutions from scratch
5. **Optimize**: Start with brute force, then optimize

### Next Steps:

- Solve variations of these problems
- Participate in coding challenges
- Study advanced topics (graphs, tries, advanced DP)
- Practice explaining your solutions
- Time yourself to simulate interview conditions

Good luck with your problem-solving journey!
