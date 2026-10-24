# Chapter 5.2 — List: ArrayList, LinkedList, Vector and Stack

[← Previous: Collections Overview](01-Collections-Overview-and-Hierarchy.md) | [Back to README](../README.md) | [Next: Set →](03-Set-HashSet-LinkedHashSet-TreeSet.md)

---

## What You Will Learn

- The `List` interface and its important methods
- `ArrayList` — internal working (dynamic array, capacity, growth)
- `LinkedList` — internal working (doubly linked list), List + Deque
- **ArrayList vs LinkedList** (most asked)
- `Vector` and **ArrayList vs Vector**
- `Stack` and why `ArrayDeque` is preferred
- Common pitfalls: `remove(int)` vs `remove(Object)`, removing while iterating

---

## 1. The `List` Interface

### 1.1 Interview Definition

> **`List` is an ordered collection (also called a sequence) in the `java.util` package that extends `Collection`. It maintains the insertion order of elements, allows duplicate elements and `null` values (depending on the implementation), and provides positional access using zero-based indexes through methods such as `get(int)`, `set(int, E)`, `add(int, E)` and `remove(int)`. Its main implementations are `ArrayList`, `LinkedList`, `Vector` and `Stack`.**

### 1.2 Important List Methods (beyond `Collection`)

| Method | Description |
|---|---|
| `E get(int index)` | Element at index |
| `E set(int index, E e)` | Replace element, returns old value |
| `void add(int index, E e)` | Insert at index (shifts elements right) |
| `E remove(int index)` | Remove at index (shifts left), returns removed element |
| `boolean remove(Object o)` | Remove first occurrence of the object |
| `int indexOf(Object o)` / `lastIndexOf(Object o)` | Search (uses `equals`) |
| `List<E> subList(int from, int to)` | View of a portion (`to` exclusive) |
| `ListIterator<E> listIterator()` | Bidirectional iterator |
| `void sort(Comparator<? super E> c)` | Sort in place (Java 8) |
| `void replaceAll(UnaryOperator<E> op)` | Transform every element (Java 8) |
| `static List.of(...)` / `List.copyOf(...)` | Immutable lists (Java 9/10) |

---

## 2. ArrayList

### 2.1 Interview Definition

> **`ArrayList` is a resizable-array implementation of the `List` interface. Internally, it stores elements in an `Object[]` array called `elementData`. It provides fast O(1) random access by index and fast amortised O(1) insertion at the end, but inserting or deleting in the middle is O(n) because elements must be shifted. When the internal array is full, `ArrayList` creates a new array about 1.5 times larger and copies the existing elements into it. It allows duplicates and `null`, maintains insertion order, and is not synchronized.**

### 2.2 Internal Working

```text
ArrayList<String> list = new ArrayList<>();      elementData = {} (empty, capacity 0)
list.add("A");                                   capacity becomes 10 (DEFAULT_CAPACITY), size 1
...
after 10 adds:  [A B C D E F G H I J]            size 10, capacity 10 (full)
list.add("K"):  newCapacity = 10 + (10 >> 1) = 15
                copy old array into new array of 15, then add "K"
                [A B C D E F G H I J K _ _ _ _]   size 11, capacity 15
next growth:    15 → 22 → 33 → 49 → ...
```

**Key facts:**
1. **Default capacity = 10**, but the array is created **lazily** on the first `add()` (since Java 7/8, `new ArrayList<>()` starts with a shared empty array to save memory).
2. **Growth formula:** `newCapacity = oldCapacity + (oldCapacity >> 1)` → **1.5×**.
3. Growth uses `Arrays.copyOf()` (which uses `System.arraycopy`) — O(n), but rare, so `add()` at the end is **amortised O(1)**.
4. `add(index, e)` and `remove(index)` shift elements using `System.arraycopy` → O(n).
5. **Size** = number of elements; **capacity** = length of the internal array. You can call `ensureCapacity(n)` to pre-allocate, or `new ArrayList<>(n)`; `trimToSize()` releases extra space.
6. Removing elements does **not** shrink the capacity automatically.

### 2.3 Real-World Example

An `ArrayList` is like a **row of seats in a cinema** that can be **rebuilt bigger** when full. Finding seat 7 is instant (index access). But if someone wants to sit in seat 3 when it is occupied, everyone from seat 3 onwards must **shift one seat right** (insert in middle = slow).

### 2.4 Java Example

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class ArrayListDemo {
    public static void main(String[] args) {
        List<String> cities = new ArrayList<>();
        cities.add("Delhi");
        cities.add("Mumbai");
        cities.add("Chennai");
        cities.add("Mumbai");                          // duplicates allowed
        cities.add(null);                              // null allowed
        System.out.println("List           : " + cities);

        cities.add(1, "Pune");                         // insert at index 1 (shifts right)
        System.out.println("After add(1,..): " + cities);

        System.out.println("get(2)         : " + cities.get(2));
        System.out.println("indexOf Mumbai : " + cities.indexOf("Mumbai"));
        System.out.println("lastIndexOf    : " + cities.lastIndexOf("Mumbai"));

        String old = cities.set(0, "New Delhi");       // replace
        System.out.println("set(0) replaced: " + old);

        cities.remove(null);                           // remove(Object)
        cities.remove(3);                              // remove(int index)
        System.out.println("After removes  : " + cities);

        cities.replaceAll(String::toUpperCase);        // Java 8
        Collections.sort(cities);
        System.out.println("Upper + sorted : " + cities);

        List<String> part = cities.subList(1, 3);      // view [1, 3)
        System.out.println("subList(1,3)   : " + part);

        List<Integer> numbers = new ArrayList<>(List.of(10, 20, 30));
        numbers.remove(1);                             // removes INDEX 1 → 20
        numbers.remove(Integer.valueOf(10));           // removes VALUE 10
        System.out.println("numbers        : " + numbers);
    }
}
```

### 2.5 Code Explanation

- Insertion order is preserved; duplicates and `null` are allowed.
- `add(1, "Pune")` shifts "Mumbai", "Chennai", … one position to the right.
- `indexOf`/`lastIndexOf` use `equals()` to search linearly — O(n).
- `set()` returns the element it replaced.
- `remove(null)` calls `remove(Object)`; `remove(3)` removes the element at **index 3** — at that moment the list is `[New Delhi, Pune, Mumbai, Chennai, Mumbai]`, so "Chennai" is removed.
- `subList` returns a **view** — changes to it affect the original list.
- **Trap:** for `List<Integer>`, `remove(1)` removes the **index** 1 (int matches `remove(int)` exactly); to remove the value, pass an `Integer` object.

### 2.6 Expected Output

```text
List           : [Delhi, Mumbai, Chennai, Mumbai, null]
After add(1,..): [Delhi, Pune, Mumbai, Chennai, Mumbai, null]
get(2)         : Mumbai
indexOf Mumbai : 2
lastIndexOf    : 4
set(0) replaced: Delhi
After removes  : [New Delhi, Pune, Mumbai, Mumbai]
Upper + sorted : [MUMBAI, MUMBAI, NEW DELHI, PUNE]
subList(1,3)   : [MUMBAI, NEW DELHI]
numbers        : [30]
```

### 2.7 Interview Answer — "How does ArrayList work internally?"

> "ArrayList is a resizable array. Internally it keeps an `Object[]` called `elementData` and an int `size`. When we create it with the default constructor, it starts with an empty array, and on the first add it allocates a capacity of 10. When the array becomes full, it grows to about 1.5 times the old capacity — the formula is old plus old shifted right by one — and copies the elements to the new array using `Arrays.copyOf`. Because growth happens rarely, adding at the end is amortised O(1). Getting by index is O(1) because it's a direct array access. But adding or removing in the middle is O(n), because elements have to be shifted with `System.arraycopy`. ArrayList allows duplicates and null, keeps insertion order, and is not thread-safe."

---

## 3. LinkedList

### 3.1 Interview Definition

> **`LinkedList` is a doubly linked list implementation of both the `List` and `Deque` interfaces. Each element is stored in a separate node that holds the element and references to the previous and next nodes, and the list keeps references to the first and last nodes. Adding or removing at the beginning or end is O(1), and inserting or removing through an iterator at a known position is O(1), but accessing an element by index is O(n) because the list must be traversed. It allows duplicates and `null`, maintains insertion order, and is not synchronized.**

### 3.2 Internal Working

```text
            first                                          last
              │                                              │
              ▼                                              ▼
null ◄── [prev|"A"|next] ◄──► [prev|"B"|next] ◄──► [prev|"C"|next] ──► null

Node<E> { E item; Node<E> next; Node<E> prev; }
```

- `addFirst/addLast/removeFirst/removeLast` → just update a few links → **O(1)**.
- `get(i)` → walks from the **nearer end** (from `first` if `i < size/2`, else from `last`) → **O(n)**.
- `add(i, e)` → O(n) to **find** position i, then O(1) to link.
- Each node stores two extra references → **more memory** than `ArrayList`, and nodes are scattered in memory → **poor CPU cache usage**.

### 3.3 Real-World Example

A `LinkedList` is like a **train**: each coach is connected to the coach before and after it. Adding a coach at the engine end or at the back is easy. But to reach coach 7, you must walk through coaches 1 to 6.

### 3.4 Java Example

```java
import java.util.LinkedList;

public class LinkedListDemo {
    public static void main(String[] args) {
        LinkedList<String> tasks = new LinkedList<>();
        tasks.add("Write code");
        tasks.addFirst("Plan");                 // O(1) at the head
        tasks.addLast("Test");                  // O(1) at the tail
        tasks.add(2, "Review");                 // O(n) to reach index 2
        System.out.println("Tasks       : " + tasks);

        System.out.println("getFirst    : " + tasks.getFirst());
        System.out.println("getLast     : " + tasks.getLast());
        System.out.println("get(2)      : " + tasks.get(2));

        // Used as a Queue (FIFO)
        tasks.offer("Deploy");
        System.out.println("poll (queue): " + tasks.poll());

        // Used as a Stack (LIFO)
        tasks.push("Hotfix");
        System.out.println("pop (stack) : " + tasks.pop());

        tasks.removeLast();
        System.out.println("Final       : " + tasks);
    }
}
```

**Expected Output:**

```text
Tasks       : [Plan, Write code, Review, Test]
getFirst    : Plan
getLast     : Test
get(2)      : Review
poll (queue): Plan
pop (stack) : Hotfix
Final       : [Write code, Review, Test]
```

**Explanation:** `LinkedList` can act as a list, a queue (`offer`/`poll` from the ends) and a stack (`push`/`pop` at the head) because it implements `Deque`.

---

## 4. ArrayList vs LinkedList

### 4.1 Comparison Table

| Point | ArrayList | LinkedList |
|---|---|---|
| Internal structure | Dynamic array (`Object[]`) | Doubly linked list (nodes) |
| `get(index)` | **O(1)** | O(n) |
| Add at end | Amortised O(1) | O(1) |
| Add/remove at beginning | O(n) (shift) | **O(1)** |
| Add/remove in middle | O(n) shift | O(n) to find + O(1) to link |
| Remove via iterator at current position | O(n) (shift) | **O(1)** |
| Memory | Less (just array + spare capacity) | More (2 extra references per node) |
| Cache friendliness | High (contiguous) | Low (scattered nodes) |
| Implements | `List`, `RandomAccess` | `List`, `Deque` |
| Use as queue/stack | Not ideal | Possible (but `ArrayDeque` is better) |
| Best for | Most cases: reading, iterating, adding at end | Frequent insertions/removals at the ends or through an iterator |

### 4.2 Interview Answer

> "Both implement `List`, keep insertion order and allow duplicates and null, but their internal structures differ. ArrayList uses a dynamic array, so getting an element by index is O(1), adding at the end is amortised O(1), but inserting or removing in the middle or at the start is O(n) because elements are shifted. LinkedList is a doubly linked list, so adding or removing at the beginning or end is O(1), but `get(index)` is O(n) because it must traverse nodes. LinkedList also uses more memory because every node stores previous and next references, and it's less cache-friendly. In practice, ArrayList is the default choice for most use cases. I'd consider LinkedList only for frequent insertions and deletions at the ends, but for queue and stack operations, ArrayDeque is usually even better."

### 4.3 Performance Demonstration

<!-- output-varies -->
```java
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

public class ListPerformance {
    static long time(Runnable r) {
        long start = System.nanoTime();
        r.run();
        return (System.nanoTime() - start) / 1_000_000;
    }

    public static void main(String[] args) {
        int n = 50_000;
        List<Integer> array = new ArrayList<>();
        List<Integer> linked = new LinkedList<>();

        System.out.println("Insert at index 0 (" + n + " times):");
        System.out.println("  ArrayList : " + time(() -> { for (int i = 0; i < n; i++) array.add(0, i); }) + " ms");
        System.out.println("  LinkedList: " + time(() -> { for (int i = 0; i < n; i++) linked.add(0, i); }) + " ms");

        System.out.println("Random get(i) (" + n + " times):");
        System.out.println("  ArrayList : " + time(() -> { long s = 0; for (int i = 0; i < n; i++) s += array.get(i); }) + " ms");
        System.out.println("  LinkedList: " + time(() -> { long s = 0; for (int i = 0; i < n; i++) s += linked.get(i); }) + " ms");
    }
}
```

**Typical Output:**

```text
Insert at index 0 (50000 times):
  ArrayList : 180 ms
  LinkedList: 3 ms
Random get(i) (50000 times):
  ArrayList : 1 ms
  LinkedList: 1250 ms
```

The numbers show the trade-off: front insertion favours `LinkedList`; index access strongly favours `ArrayList`.

---

## 5. Vector

### 5.1 Interview Definition

> **`Vector` is a legacy class (from Java 1.0) that implements a growable array, similar to `ArrayList`, but all its public methods are synchronized, which makes it thread-safe but slower. By default, it doubles its capacity when it is full. It was retrofitted to implement `List` in Java 1.2. In modern code, `ArrayList` is preferred for single-threaded use, and `Collections.synchronizedList()` or `CopyOnWriteArrayList` is preferred when thread safety is required.**

### 5.2 ArrayList vs Vector

| Point | ArrayList | Vector |
|---|---|---|
| Introduced | Java 1.2 | Java 1.0 (legacy) |
| Synchronized | No | Yes (every method) |
| Thread-safe | No | Yes (for single operations) |
| Performance | Faster | Slower (locking) |
| Growth | 1.5× | 2× (or by `capacityIncrement` if set) |
| Default capacity | 10 (lazy) | 10 (eager) |
| Traversal | `Iterator`, `ListIterator` | Also legacy `Enumeration` |
| Modern alternative for threads | `CopyOnWriteArrayList`, `Collections.synchronizedList` | — |

---

## 6. Stack

### 6.1 Interview Definition

> **`Stack` is a legacy class that extends `Vector` and represents a last-in-first-out (LIFO) stack of objects. It provides `push()` to add an element on top, `pop()` to remove and return the top element, `peek()` to view the top element without removing it, `empty()` to check whether it is empty, and `search()` to find an element's position from the top. Because it extends `Vector`, it is synchronized and also exposes index-based methods that break stack semantics, so the Java documentation recommends using the `Deque` interface, typically `ArrayDeque`, for stack operations.**

### 6.2 Java Example — Stack vs ArrayDeque

```java
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.Stack;

public class StackDemo {
    static boolean isBalanced(String expr) {
        Deque<Character> stack = new ArrayDeque<>();           // recommended stack
        for (char c : expr.toCharArray()) {
            if (c == '(' || c == '[' || c == '{') {
                stack.push(c);
            } else if (c == ')' || c == ']' || c == '}') {
                if (stack.isEmpty()) return false;
                char open = stack.pop();
                if ((c == ')' && open != '(') || (c == ']' && open != '[') || (c == '}' && open != '{')) {
                    return false;
                }
            }
        }
        return stack.isEmpty();
    }

    public static void main(String[] args) {
        Stack<String> legacy = new Stack<>();
        legacy.push("A");
        legacy.push("B");
        legacy.push("C");
        System.out.println("Stack        : " + legacy);
        System.out.println("peek         : " + legacy.peek());
        System.out.println("pop          : " + legacy.pop());
        System.out.println("search(\"A\")  : " + legacy.search("A"));   // 1-based position from top
        legacy.add(0, "HACK");                                       // allowed because Stack extends Vector!
        System.out.println("After add(0) : " + legacy);

        System.out.println("{[()]} balanced? " + isBalanced("{[()]}"));
        System.out.println("([)]   balanced? " + isBalanced("([)]"));
        System.out.println("((     balanced? " + isBalanced("(("));
    }
}
```

**Expected Output:**

```text
Stack        : [A, B, C]
peek         : C
pop          : C
search("A")  : 2
After add(0) : [HACK, A, B]
{[()]} balanced? true
([)]   balanced? false
((     balanced? false
```

**Explanation:**
- `search("A")` returns 2: position counted from the **top**, starting at 1 (top is "B" = 1, "A" = 2).
- `add(0, "HACK")` shows the design flaw: a `Stack` lets you insert at the bottom because it inherits `Vector`'s methods (see "composition over inheritance", [Chapter 3.11](../03-OOP/11-Association-Aggregation-Composition.md)).
- The bracket checker uses `ArrayDeque` as the stack — the recommended approach: faster, not synchronized, and exposes only deque operations.

---

## 7. Interview Questions with Complete Answers

### Basic Level

**Q1. What is the default capacity of ArrayList and how does it grow?**

Default capacity is 10, allocated lazily on the first `add()`. When full, it grows to `old + old/2` (1.5×), copying elements into a new array.

**Q2. Can ArrayList store primitives?**

Not directly. It stores objects, so primitives are autoboxed (`list.add(5)` → `Integer.valueOf(5)`). For large primitive data, an `int[]` is more memory-efficient.

**Q3. What is the difference between `size()` and capacity?**

`size()` is the number of elements actually stored. Capacity is the length of the internal array (how many elements can be stored before resizing). Capacity ≥ size. There is no public method to read capacity.

### Intermediate Level

**Q4. Why is `get(index)` O(1) in ArrayList but O(n) in LinkedList?**

ArrayList stores elements in a contiguous array, so the address of index i is computed directly. LinkedList nodes are connected by references; to reach index i, it must follow links from the head or tail one by one (it starts from whichever end is closer, but that is still O(n)).

**Q5. When would you use LinkedList over ArrayList?**

When the main operations are adding/removing at the **ends**, or removing many elements **while iterating** using a `ListIterator`. In practice, `ArrayDeque` is better for queue/stack use, and `ArrayList` usually wins for general use because of cache efficiency. So LinkedList is rarely the best choice today.

**Q6. How do you make an ArrayList thread-safe?**

`Collections.synchronizedList(new ArrayList<>())` (and synchronize manually during iteration), or `CopyOnWriteArrayList` for read-heavy workloads, or guard access with your own locks. `Vector` works but is legacy.

**Q7. What happens when you remove elements from an ArrayList in a for-each loop?**

It throws `ConcurrentModificationException` (fail-fast iterator). Use `iterator.remove()`, `removeIf()`, or iterate with an index backwards. See [Chapter 5.7](07-Iterator-ListIterator-Fail-Fast.md).

### Advanced Level

**Q8. What is the `RandomAccess` interface?**

A marker interface implemented by `ArrayList` and `Vector` (not `LinkedList`) to signal fast index access. Generic algorithms like `Collections.binarySearch()` check `list instanceof RandomAccess` to choose between index-based and iterator-based strategies.

**Q9. How does `ArrayList.remove(int index)` work internally?**

It checks the index, reads the element to return, calculates `numMoved = size - index - 1`, shifts the elements after the index one position left with `System.arraycopy`, sets the last slot to `null` (so the GC can collect the object — avoiding a memory leak), decrements `size`, and increments `modCount`.

**Q10. Is `subList()` a copy?**

No, it is a **view** backed by the original list. Changes through the sublist affect the original, and structural changes to the original (adding/removing) made **outside** the sublist make the sublist invalid (`ConcurrentModificationException` on next use). To get an independent copy: `new ArrayList<>(list.subList(a, b))`.

---

## 8. Follow-Up Questions

**F1. How to remove duplicates from an ArrayList while keeping order?**

`new ArrayList<>(new LinkedHashSet<>(list))`, or `list.stream().distinct().toList()`.

**F2. How to convert an array to an ArrayList?**

`new ArrayList<>(Arrays.asList(arr))` (for object arrays) or `new ArrayList<>(List.of(arr))`. For `int[]`: `Arrays.stream(arr).boxed().collect(Collectors.toList())`.

**F3. Why does `ArrayList` set the removed slot to `null`?**

To remove the array's reference to the object so the garbage collector can reclaim it. Otherwise, the internal array would keep "obsolete references" alive — a classic memory leak.

---

## 9. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| `list.remove(1)` on `List<Integer>` to remove value 1 | Removes index 1; use `remove(Integer.valueOf(1))`. |
| Choosing LinkedList "because inserts are O(1)" | Finding the position is O(n); ArrayList is usually faster. |
| Using `Stack`/`Vector` in new code | Use `ArrayDeque`/`ArrayList`. |
| Modifying a list inside for-each | `ConcurrentModificationException`. |
| Calling `get(i)` in a loop on LinkedList | O(n²); use an iterator/for-each. |
| Treating `subList` as an independent copy | It's a view. |

---

## 10. Quick Revision

- `List`: ordered, duplicates, index access.
- **ArrayList:** `Object[]`, default 10 (lazy), grows 1.5×, get O(1), add end amortised O(1), middle O(n), not synchronized, `RandomAccess`.
- **LinkedList:** doubly linked nodes, List + Deque, ends O(1), get O(n), more memory.
- **Vector:** synchronized, grows 2×, legacy.
- **Stack:** extends Vector, LIFO (`push/pop/peek/search`), legacy → use `ArrayDeque`.
- `remove(int)` vs `remove(Object)` trap; for-each removal → CME.

---

[← Previous: Collections Overview](01-Collections-Overview-and-Hierarchy.md) | [Back to README](../README.md) | [Next: Set →](03-Set-HashSet-LinkedHashSet-TreeSet.md)
