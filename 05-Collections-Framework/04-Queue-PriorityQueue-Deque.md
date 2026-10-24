# Chapter 5.4 — Queue, PriorityQueue and Deque

[← Previous: Set](03-Set-HashSet-LinkedHashSet-TreeSet.md) | [Back to README](../README.md) | [Next: Map →](05-Map-HashMap-LinkedHashMap-TreeMap-Hashtable.md)

---

## What You Will Learn

- The `Queue` interface and its two families of methods (throwing vs returning special values)
- `PriorityQueue` — binary heap, min-heap/max-heap, custom priority
- The `Deque` interface — double-ended queue, used as both queue and stack
- `ArrayDeque` vs `LinkedList` vs `Stack`
- Classic uses: BFS, task scheduling, top-K elements, sliding window maximum (intro)

---

## 1. Queue

### 1.1 Interview Definition

> **`Queue` is a collection interface in `java.util` designed for holding elements before processing, typically in first-in-first-out (FIFO) order. Elements are added at the tail and removed from the head. It provides two forms of each operation: `add()`, `remove()` and `element()` throw an exception when the operation fails, while `offer()`, `poll()` and `peek()` return a special value (`false` or `null`) instead. Common implementations are `LinkedList`, `ArrayDeque` and `PriorityQueue`, which orders elements by priority instead of insertion order.**

### 1.2 Queue Methods

| Operation | Throws exception on failure | Returns special value |
|---|---|---|
| Insert at tail | `add(e)` → `IllegalStateException` if full | `offer(e)` → `false` if full |
| Remove head | `remove()` → `NoSuchElementException` if empty | `poll()` → `null` if empty |
| Examine head | `element()` → `NoSuchElementException` if empty | `peek()` → `null` if empty |

Prefer `offer`/`poll`/`peek` — they handle "empty" and "full" gracefully. (Most queues in `java.util` are unbounded, so `add` and `offer` behave the same; capacity limits matter for `BlockingQueue` implementations in concurrency.)

### 1.3 Real-World Example

A **ticket counter line**: the first person to join is the first to be served (FIFO). New people join at the end (tail); the person at the front (head) is served and leaves.

### 1.4 Java Example

```java
import java.util.ArrayDeque;
import java.util.Queue;

public class QueueDemo {
    public static void main(String[] args) {
        Queue<String> printQueue = new ArrayDeque<>();
        printQueue.offer("Resume.pdf");
        printQueue.offer("Marksheet.pdf");
        printQueue.offer("Photo.png");
        System.out.println("Queue        : " + printQueue);
        System.out.println("peek (head)  : " + printQueue.peek());

        while (!printQueue.isEmpty()) {
            System.out.println("Printing     : " + printQueue.poll());
        }

        System.out.println("poll on empty: " + printQueue.poll());
        System.out.println("peek on empty: " + printQueue.peek());
        try {
            printQueue.remove();
        } catch (java.util.NoSuchElementException e) {
            System.out.println("remove on empty -> NoSuchElementException");
        }
    }
}
```

**Expected Output:**

```text
Queue        : [Resume.pdf, Marksheet.pdf, Photo.png]
peek (head)  : Resume.pdf
Printing     : Resume.pdf
Printing     : Marksheet.pdf
Printing     : Photo.png
poll on empty: null
peek on empty: null
remove on empty -> NoSuchElementException
```

---

## 2. PriorityQueue

### 2.1 Interview Definition

> **`PriorityQueue` is a `Queue` implementation in which elements are ordered by priority rather than by insertion time. By default it is a min-heap: the head is always the smallest element according to natural ordering, or according to a `Comparator` supplied at construction (for example, `Comparator.reverseOrder()` for a max-heap). Internally it uses a binary heap stored in an array, so `offer()` and `poll()` take O(log n) time and `peek()` takes O(1). It does not allow `null` elements, it is not thread-safe, and its iterator does not return elements in sorted order — only repeated `poll()` calls do.**

### 2.2 Internal Working — Binary Heap

A **binary min-heap** is a complete binary tree where each parent is ≤ its children. It is stored in an array:
- children of index `i` → `2i + 1` and `2i + 2`
- parent of index `i` → `(i − 1) / 2`

```text
offer: 40, 10, 30, 5

        5                array: [5, 10, 30, 40]
      /   \
    10     30           smallest is ALWAYS at index 0 (the head)
   /
  40
```

- **offer(e):** put at the end, then **sift up** (swap with parent while smaller) → O(log n).
- **poll():** take the root, move the last element to the root, then **sift down** (swap with the smaller child) → O(log n).
- **peek():** read index 0 → O(1).
- Printing the queue (`toString`/iterator) shows the **array order**, which is **not fully sorted**.

### 2.3 Real-World Example

A **hospital emergency room**: patients are treated by **severity**, not by arrival time. A heart-attack patient who arrives later is seen before someone with a minor cut.

### 2.4 Java Example

```java
import java.util.Comparator;
import java.util.PriorityQueue;

public class PriorityQueueDemo {
    record Patient(String name, int severity) { }       // higher severity = more urgent

    public static void main(String[] args) {
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();
        for (int n : new int[]{40, 10, 30, 5, 20}) {
            minHeap.offer(n);
        }
        System.out.println("Internal order (not sorted!): " + minHeap);
        System.out.println("peek (smallest): " + minHeap.peek());
        System.out.print("poll order: ");
        while (!minHeap.isEmpty()) {
            System.out.print(minHeap.poll() + " ");
        }
        System.out.println();

        PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
        maxHeap.addAll(java.util.List.of(40, 10, 30, 5, 20));
        System.out.println("max-heap peek : " + maxHeap.peek());

        PriorityQueue<Patient> er = new PriorityQueue<>(
                Comparator.comparingInt(Patient::severity).reversed()
                          .thenComparing(Patient::name));
        er.offer(new Patient("Cut finger", 2));
        er.offer(new Patient("Heart attack", 10));
        er.offer(new Patient("Fracture", 6));
        er.offer(new Patient("Burn", 6));
        System.out.print("Treatment order: ");
        while (!er.isEmpty()) {
            System.out.print(er.poll().name() + " | ");
        }
        System.out.println();

        // Top-3 largest using a min-heap of size 3
        int[] data = {7, 2, 19, 4, 15, 11, 3};
        PriorityQueue<Integer> top3 = new PriorityQueue<>();
        for (int x : data) {
            top3.offer(x);
            if (top3.size() > 3) top3.poll();          // drop the smallest
        }
        System.out.println("Top 3 largest (min-heap contents): " + new java.util.TreeSet<>(top3));
    }
}
```

### 2.5 Code Explanation

- Printing `minHeap` shows the heap's array layout `[5, 10, 30, 40, 20]` — valid heap, **not** sorted. Only `poll()` returns elements in sorted order.
- `Comparator.reverseOrder()` turns it into a **max-heap**.
- For `Patient`, the comparator orders by severity **descending**, then by name to break ties ("Burn" before "Fracture" at severity 6).
- **Top-K pattern:** keep a min-heap of size K; whenever it exceeds K, remove the smallest. What remains are the K largest. Time O(n log K) — better than sorting O(n log n) for small K.

### 2.6 Expected Output

```text
Internal order (not sorted!): [5, 10, 30, 40, 20]
peek (smallest): 5
poll order: 5 10 20 30 40 
max-heap peek : 40
Treatment order: Heart attack | Burn | Fracture | Cut finger | 
Top 3 largest (min-heap contents): [11, 15, 19]
```

### 2.7 Interview Answer — "How does PriorityQueue work?"

> "PriorityQueue is a queue where elements come out based on priority instead of insertion order. By default, it's a min-heap — the smallest element according to natural ordering is always at the head. We can pass a Comparator, for example `Comparator.reverseOrder()` to make it a max-heap, or a custom comparator for objects. Internally it uses a binary heap stored in an array: adding an element puts it at the end and moves it up, and removing takes the root and moves the last element down, so both `offer` and `poll` are O(log n), and `peek` is O(1). It doesn't allow null, isn't thread-safe, and its iterator or `toString` does not give sorted order — only repeated `poll` does. A common use is finding the top K elements or scheduling tasks by priority."

---

## 3. Deque (Double-Ended Queue)

### 3.1 Interview Definition

> **`Deque` (pronounced "deck") is an interface that extends `Queue` and supports inserting, removing and examining elements at both ends — the head (first) and the tail (last). Because of this, a `Deque` can be used both as a FIFO queue and as a LIFO stack. Its main implementations are `ArrayDeque`, a resizable circular array that is faster than `Stack` and usually faster than `LinkedList`, and `LinkedList`. `ArrayDeque` does not allow `null` elements and is not thread-safe.**

### 3.2 Deque Methods

| | First (head) — throws | First — special value | Last (tail) — throws | Last — special value |
|---|---|---|---|---|
| Insert | `addFirst(e)` | `offerFirst(e)` | `addLast(e)` | `offerLast(e)` |
| Remove | `removeFirst()` | `pollFirst()` | `removeLast()` | `pollLast()` |
| Examine | `getFirst()` | `peekFirst()` | `getLast()` | `peekLast()` |

**As a stack:** `push(e)` = `addFirst`, `pop()` = `removeFirst`, `peek()` = `peekFirst`.
**As a queue:** `offer(e)` = `offerLast`, `poll()` = `pollFirst`, `peek()` = `peekFirst`.

### 3.3 Java Example

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class DequeDemo {
    public static void main(String[] args) {
        Deque<Integer> deque = new ArrayDeque<>();
        deque.offerLast(2);
        deque.offerLast(3);
        deque.offerFirst(1);
        deque.offerLast(4);
        System.out.println("Deque          : " + deque);
        System.out.println("peekFirst/Last : " + deque.peekFirst() + " / " + deque.peekLast());
        System.out.println("pollFirst      : " + deque.pollFirst());
        System.out.println("pollLast       : " + deque.pollLast());
        System.out.println("Remaining      : " + deque);

        // As a stack: browser back button
        Deque<String> history = new ArrayDeque<>();
        history.push("google.com");
        history.push("github.com");
        history.push("stackoverflow.com");
        System.out.println("Back from      : " + history.pop() + " -> now at " + history.peek());

        // Palindrome check using both ends
        String word = "racecar";
        Deque<Character> chars = new ArrayDeque<>();
        for (char c : word.toCharArray()) chars.offerLast(c);
        boolean palindrome = true;
        while (chars.size() > 1) {
            if (!chars.pollFirst().equals(chars.pollLast())) {
                palindrome = false;
                break;
            }
        }
        System.out.println(word + " palindrome? " + palindrome);
    }
}
```

**Expected Output:**

```text
Deque          : [1, 2, 3, 4]
peekFirst/Last : 1 / 4
pollFirst      : 1
pollLast       : 4
Remaining      : [2, 3]
Back from      : stackoverflow.com -> now at github.com
racecar palindrome? true
```

### 3.4 ArrayDeque Internal Working

- A **circular array** with `head` and `tail` indexes. Adding at the front moves `head` backwards (wrapping around); adding at the back moves `tail` forwards.
- When full, the array grows (doubles for small sizes, +50% for large) and elements are copied.
- All end operations are **amortised O(1)** with excellent cache locality.

### 3.5 ArrayDeque vs LinkedList vs Stack

| Point | `ArrayDeque` | `LinkedList` | `Stack` |
|---|---|---|---|
| Structure | Circular resizable array | Doubly linked list | Array (extends `Vector`) |
| Use as stack | ✅ Best | ✅ | ✅ Legacy |
| Use as queue | ✅ Best | ✅ | ❌ |
| `null` elements | ❌ Not allowed | ✅ Allowed | ✅ Allowed |
| Synchronized | No | No | Yes (slow) |
| Index access | No | Yes (O(n)) | Yes (breaks stack semantics) |
| Memory/speed | Low memory, fastest | More memory per element | Locking overhead |

> The Javadoc of `ArrayDeque` says it "is likely to be faster than `Stack` when used as a stack, and faster than `LinkedList` when used as a queue."

---

## 4. A Classic Queue Use — Breadth-First Search

```java
import java.util.ArrayDeque;
import java.util.HashSet;
import java.util.List;
import java.util.Map;
import java.util.Queue;
import java.util.Set;

public class BfsDemo {
    public static void main(String[] args) {
        Map<String, List<String>> friends = Map.of(
                "Asha", List.of("Bala", "Chitra"),
                "Bala", List.of("Dev"),
                "Chitra", List.of("Dev", "Esha"),
                "Dev", List.of("Farhan"),
                "Esha", List.of(),
                "Farhan", List.of());

        Queue<String> queue = new ArrayDeque<>();
        Set<String> visited = new HashSet<>();
        queue.offer("Asha");
        visited.add("Asha");

        System.out.print("BFS order: ");
        while (!queue.isEmpty()) {
            String person = queue.poll();
            System.out.print(person + " ");
            for (String f : friends.get(person)) {
                if (visited.add(f)) {          // add() returns false if already visited
                    queue.offer(f);
                }
            }
        }
        System.out.println();
    }
}
```

**Expected Output:**

```text
BFS order: Asha Bala Chitra Dev Esha Farhan 
```

BFS visits people level by level (Asha → her direct friends → their friends), which is exactly FIFO behaviour.

---

## 5. Interview Questions with Complete Answers

### Basic Level

**Q1. What is the difference between `offer()` and `add()`, `poll()` and `remove()`, `peek()` and `element()`?**

They do the same job, but `add/remove/element` **throw exceptions** on failure (full/empty queue), while `offer/poll/peek` **return special values** (`false`/`null`). Prefer the second set for normal checks.

**Q2. Is PriorityQueue sorted?**

Not internally. It only guarantees that the **head** is the smallest (or highest priority). Iterating or printing shows heap order. To get sorted output, `poll()` repeatedly.

**Q3. Which class should you use for a stack in modern Java?**

`ArrayDeque` through the `Deque` interface (`push`, `pop`, `peek`). `Stack` is legacy, synchronized, and extends `Vector`, exposing non-stack operations.

### Intermediate Level

**Q4. How do you create a max-heap in Java?**

`new PriorityQueue<>(Comparator.reverseOrder())`, or `new PriorityQueue<>((a, b) -> b - a)` (be careful: subtraction can overflow for large values; prefer `Integer.compare(b, a)` or `reverseOrder()`).

**Q5. Why doesn't `ArrayDeque` (or `PriorityQueue`) allow `null`?**

Because `poll()` and `peek()` return `null` to mean "empty". If `null` elements were allowed, you could not tell an empty deque from a `null` element.

**Q6. What is the time complexity of PriorityQueue operations?**

`offer`/`add`: O(log n). `poll`/`remove()`: O(log n). `peek`: O(1). `remove(Object)` and `contains`: O(n) (linear search in the array). Building from a collection: O(n).

### Advanced Level

**Q7. How would you find the k-th largest element in an array efficiently?**

Use a **min-heap of size k**: offer each element; if size exceeds k, poll. At the end, `peek()` is the k-th largest. Time O(n log k), space O(k). (Quickselect gives average O(n).)

**Q8. Is `LinkedList` or `ArrayDeque` better for a queue?**

`ArrayDeque` — it uses a contiguous circular array (better cache locality, less memory, no node allocation per element). `LinkedList` is useful only if you also need `null` elements or `List` operations.

**Q9. What is a `BlockingQueue`?**

A thread-safe queue (in `java.util.concurrent`) whose `put()` waits when the queue is full and `take()` waits when it is empty. Implementations: `ArrayBlockingQueue`, `LinkedBlockingQueue`, `PriorityBlockingQueue`. It is the standard tool for producer–consumer problems. See [Part 7](../07-Multithreading-and-Concurrency/06-wait-notify-Producer-Consumer.md).

---

## 6. Follow-Up Questions

**F1. Can a PriorityQueue contain duplicate elements?**

Yes. Equal-priority elements are allowed; their relative order among themselves is not guaranteed (it is not a stable ordering). Add a tie-breaker in the comparator if order matters.

**F2. How do you iterate a Deque in reverse?**

`deque.descendingIterator()`.

**F3. What is a circular queue and does Java have one?**

A fixed-size queue where the tail wraps around to the beginning of the array. `ArrayDeque` uses this idea internally (but resizes when full). `ArrayBlockingQueue` is a bounded circular buffer.

---

## 7. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Printing a PriorityQueue to see sorted order | Only `poll()` gives sorted order. |
| Adding `null` to ArrayDeque/PriorityQueue | `NullPointerException`. |
| Using `Stack` in new code | Use `ArrayDeque`. |
| Using `remove()` without checking emptiness | Use `poll()` or check `isEmpty()`. |
| `(a, b) -> b - a` comparator with huge values | Overflow; use `Integer.compare(b, a)`. |

---

## 8. Quick Revision

- `Queue`: FIFO; `offer/poll/peek` (safe) vs `add/remove/element` (throw).
- `PriorityQueue`: binary heap in array, min-heap by default, `Comparator.reverseOrder()` for max-heap; offer/poll O(log n), peek O(1); no null; iteration not sorted.
- `Deque`: both ends; stack (`push/pop/peek`) and queue (`offer/poll/peek`).
- `ArrayDeque`: circular array, fastest stack/queue, no null, not thread-safe.
- Patterns: BFS (queue), undo/back/brackets (stack), top-K (heap), palindrome (deque).

---

[← Previous: Set](03-Set-HashSet-LinkedHashSet-TreeSet.md) | [Back to README](../README.md) | [Next: Map →](05-Map-HashMap-LinkedHashMap-TreeMap-Hashtable.md)
