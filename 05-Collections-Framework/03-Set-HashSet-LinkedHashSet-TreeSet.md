# Chapter 5.3 — Set: HashSet, LinkedHashSet and TreeSet

[← Previous: List](02-List-ArrayList-LinkedList-Vector-Stack.md) | [Back to README](../README.md) | [Next: Queue and Deque →](04-Queue-PriorityQueue-Deque.md)

---

## What You Will Learn

- The `Set` interface and how uniqueness is decided
- `HashSet` — internal working (backed by `HashMap`)
- `LinkedHashSet` — insertion order
- `TreeSet` — sorted order, `NavigableSet` methods, Red-Black tree
- **HashSet vs LinkedHashSet vs TreeSet**
- Set operations (union, intersection, difference)
- Using custom objects in sets

---

## 1. The `Set` Interface

### 1.1 Interview Definition

> **`Set` is a collection in `java.util` that does not allow duplicate elements; it models the mathematical concept of a set. Two elements are considered duplicates based on `equals()` (and `hashCode()` for hash-based sets) or, for sorted sets, based on `compareTo()`/`Comparator` returning zero. The `add()` method returns `false` if the element already exists. `Set` does not provide index-based access. Its main implementations are `HashSet` (no order), `LinkedHashSet` (insertion order) and `TreeSet` (sorted order).**

### 1.2 How a Set Decides "Duplicate"

| Implementation | Duplicate check |
|---|---|
| `HashSet`, `LinkedHashSet` | Same `hashCode()` **and** `equals()` returns true |
| `TreeSet` | `compareTo()` / `compare()` returns **0** (`equals()` is **not** used) |

---

## 2. HashSet

### 2.1 Interview Definition

> **`HashSet` is an implementation of the `Set` interface backed by a `HashMap`. Each element added to a `HashSet` is stored as a key in the internal `HashMap`, with a constant dummy object as the value. Because of hashing, `add`, `remove` and `contains` run in O(1) average time. A `HashSet` does not maintain any order of elements, allows one `null` element, and is not synchronized. Correct behaviour with custom objects requires properly overridden `equals()` and `hashCode()` methods.**

### 2.2 Internal Working

```java
// Simplified from the JDK source
public class HashSet<E> {
    private transient HashMap<E, Object> map;
    private static final Object PRESENT = new Object();   // dummy value

    public HashSet() { map = new HashMap<>(); }

    public boolean add(E e) {
        return map.put(e, PRESENT) == null;   // null → key was new → added
    }
    public boolean contains(Object o) { return map.containsKey(o); }
    public boolean remove(Object o)   { return map.remove(o) == PRESENT; }
}
```

- `add(e)` → `map.put(e, PRESENT)`. If the key already existed, `put` returns the old value (`PRESENT`, not `null`) → `add` returns **false** and the set is unchanged.
- So **all HashSet behaviour comes from HashMap**: hashing, buckets, load factor 0.75, default capacity 16, treeification. ([Chapter 5.6](06-Hashing-and-HashMap-Internal-Working.md))

### 2.3 Real-World Example

A `HashSet` is like the **guest list at a wedding entrance**: each name is checked instantly (hashing). The same person cannot enter twice. The list doesn't care about the order people arrived.

### 2.4 Java Example

```java
import java.util.HashSet;
import java.util.Set;

public class HashSetDemo {
    public static void main(String[] args) {
        Set<String> emails = new HashSet<>();
        System.out.println("add a@x.com : " + emails.add("a@x.com"));
        System.out.println("add b@x.com : " + emails.add("b@x.com"));
        System.out.println("add a@x.com : " + emails.add("a@x.com"));   // duplicate → false
        System.out.println("add null    : " + emails.add(null));        // one null allowed
        System.out.println("add null    : " + emails.add(null));
        System.out.println("size        : " + emails.size());
        System.out.println("contains b  : " + emails.contains("b@x.com"));
        emails.remove("b@x.com");
        System.out.println("after remove: size " + emails.size());

        // Remove duplicates from a list in one line
        java.util.List<Integer> withDuplicates = java.util.List.of(4, 1, 4, 2, 1, 3);
        Set<Integer> unique = new HashSet<>(withDuplicates);
        System.out.println("unique count: " + unique.size());
    }
}
```

**Expected Output:**

```text
add a@x.com : true
add b@x.com : true
add a@x.com : false
add null    : true
add null    : false
size        : 3
contains b  : true
after remove: size 2
unique count: 4
```

---

## 3. LinkedHashSet

### 3.1 Interview Definition

> **`LinkedHashSet` is a subclass of `HashSet` that maintains the insertion order of elements. Internally it is backed by a `LinkedHashMap`, which keeps a doubly linked list running through all entries in addition to the hash table. It offers the same O(1) average performance as `HashSet` for add, remove and contains, with slightly higher memory usage, and iteration returns elements in the order in which they were first inserted. Re-inserting an existing element does not change its position.**

### 3.2 Java Example

```java
import java.util.LinkedHashSet;
import java.util.Set;

public class LinkedHashSetDemo {
    public static void main(String[] args) {
        Set<String> visited = new LinkedHashSet<>();
        visited.add("Home");
        visited.add("Products");
        visited.add("Cart");
        visited.add("Home");                 // duplicate: ignored, position unchanged
        visited.add("Checkout");
        System.out.println("Pages in first-visit order: " + visited);

        String sentence = "the cat and the dog and the bird";
        Set<String> uniqueWords = new LinkedHashSet<>(java.util.Arrays.asList(sentence.split(" ")));
        System.out.println("Unique words in order: " + uniqueWords);
    }
}
```

**Expected Output:**

```text
Pages in first-visit order: [Home, Products, Cart, Checkout]
Unique words in order: [the, cat, and, dog, bird]
```

---

## 4. TreeSet

### 4.1 Interview Definition

> **`TreeSet` is a `NavigableSet` implementation that stores elements in sorted order, either by their natural ordering (`Comparable`) or by a `Comparator` supplied at construction. Internally it is backed by a `TreeMap`, which is a Red-Black tree — a self-balancing binary search tree — so `add`, `remove` and `contains` take O(log n) time. It does not allow `null` elements (natural ordering throws `NullPointerException`), uses `compareTo()`/`compare()` instead of `equals()` to detect duplicates, and provides navigation methods such as `first()`, `last()`, `floor()`, `ceiling()`, `higher()`, `lower()`, `headSet()`, `tailSet()` and `subSet()`.**

### 4.2 Understand the Concept

- **Red-Black tree:** a binary search tree that keeps itself approximately balanced using node colours and rotations, so its height stays O(log n).
- Elements must be **comparable**: either the class implements `Comparable`, or you pass a `Comparator`. Otherwise adding the first non-comparable element throws `ClassCastException`.
- **Duplicates are decided by comparison**: if `compare(a, b) == 0`, `b` is treated as a duplicate of `a`, even if `a.equals(b)` is false.

### 4.3 Java Example

```java
import java.util.Comparator;
import java.util.TreeSet;

public class TreeSetDemo {
    public static void main(String[] args) {
        TreeSet<Integer> scores = new TreeSet<>();
        scores.add(72);
        scores.add(95);
        scores.add(60);
        scores.add(88);
        scores.add(72);                                   // duplicate ignored
        System.out.println("Sorted     : " + scores);

        System.out.println("first/last : " + scores.first() + " / " + scores.last());
        System.out.println("floor(80)  : " + scores.floor(80));     // greatest <= 80
        System.out.println("ceiling(80): " + scores.ceiling(80));   // smallest >= 80
        System.out.println("higher(88) : " + scores.higher(88));    // strictly greater
        System.out.println("lower(60)  : " + scores.lower(60));     // strictly smaller → null
        System.out.println("headSet(80): " + scores.headSet(80));   // < 80
        System.out.println("tailSet(80): " + scores.tailSet(80));   // >= 80
        System.out.println("subSet(60,90): " + scores.subSet(60, 90));
        System.out.println("descending : " + scores.descendingSet());
        System.out.println("pollFirst  : " + scores.pollFirst() + " → " + scores);

        // Custom order: by length, then alphabetically
        TreeSet<String> names = new TreeSet<>(Comparator.comparing(String::length).thenComparing(Comparator.naturalOrder()));
        names.add("Ravi");
        names.add("Al");
        names.add("Priya");
        names.add("Anu");
        names.add("Bob");
        System.out.println("By length  : " + names);

        // Comparator that treats different strings as "equal"
        TreeSet<String> caseInsensitive = new TreeSet<>(String.CASE_INSENSITIVE_ORDER);
        caseInsensitive.add("java");
        caseInsensitive.add("JAVA");                      // compare == 0 → treated as duplicate
        System.out.println("Case-insensitive: " + caseInsensitive);

        try {
            new TreeSet<String>().add(null);
        } catch (NullPointerException e) {
            System.out.println("TreeSet does not accept null");
        }
    }
}
```

### 4.4 Expected Output

```text
Sorted     : [60, 72, 88, 95]
first/last : 60 / 95
floor(80)  : 72
ceiling(80): 88
higher(88) : 95
lower(60)  : null
headSet(80): [60, 72]
tailSet(80): [88, 95]
subSet(60,90): [60, 72, 88]
descending : [95, 88, 72, 60]
pollFirst  : 60 → [72, 88, 95]
By length  : [Al, Anu, Bob, Ravi, Priya]
Case-insensitive: [java]
TreeSet does not accept null
```

### 4.5 Code Explanation

- Elements are always iterated in **ascending** order.
- `floor`/`ceiling` include equality; `lower`/`higher` are strict. They return `null` when no such element exists.
- `headSet(80)` excludes 80; `tailSet(80)` includes 80; `subSet(60, 90)` = [60, 90).
- `pollFirst()` removes and returns the smallest element.
- A custom `Comparator` defines both **order** and **uniqueness**.
- `CASE_INSENSITIVE_ORDER` treats "java" and "JAVA" as equal, so only the first is kept.

---

## 5. HashSet vs LinkedHashSet vs TreeSet

| Point | HashSet | LinkedHashSet | TreeSet |
|---|---|---|---|
| Internal structure | `HashMap` (hash table) | `LinkedHashMap` (hash table + doubly linked list) | `TreeMap` (Red-Black tree) |
| Order | None (unpredictable) | **Insertion order** | **Sorted** (natural or comparator) |
| add / remove / contains | O(1) average | O(1) average | O(log n) |
| `null` | One `null` allowed | One `null` allowed | **Not allowed** (natural ordering) |
| Duplicate check | `hashCode()` + `equals()` | `hashCode()` + `equals()` | `compareTo()` / `compare()` |
| Memory | Least | More (links) | More (tree nodes) |
| Extra methods | — | — | `first`, `last`, `floor`, `ceiling`, `headSet`, `tailSet`, `subSet`, `descendingSet`, `pollFirst/Last` |
| Use when | Fast uniqueness, order not needed | Uniqueness + preserve insertion order | Uniqueness + sorted order / range queries |

### 5.1 Interview Answer

> "All three implement `Set`, so none allows duplicates, but they differ in ordering and performance. **HashSet** is backed by a HashMap, gives O(1) average add, remove and contains, but no order, and allows one null. **LinkedHashSet** extends HashSet and is backed by a LinkedHashMap, so it keeps insertion order with almost the same O(1) performance and a little extra memory. **TreeSet** is backed by a TreeMap, which is a Red-Black tree, so it keeps elements sorted, operations are O(log n), it doesn't allow null, and it uses `compareTo` or a Comparator to detect duplicates instead of `equals`. It also offers navigation methods like `floor`, `ceiling` and `headSet`. I choose HashSet by default, LinkedHashSet when order of insertion matters, and TreeSet when I need sorted data or range queries."

---

## 6. Set Operations

```java
import java.util.HashSet;
import java.util.Set;
import java.util.TreeSet;

public class SetOperationsDemo {
    public static void main(String[] args) {
        Set<String> javaDevs = Set.of("Asha", "Bala", "Chitra", "Dev");
        Set<String> pythonDevs = Set.of("Chitra", "Dev", "Esha");

        Set<String> union = new TreeSet<>(javaDevs);
        union.addAll(pythonDevs);
        System.out.println("Union        : " + union);

        Set<String> intersection = new TreeSet<>(javaDevs);
        intersection.retainAll(pythonDevs);
        System.out.println("Intersection : " + intersection);

        Set<String> onlyJava = new TreeSet<>(javaDevs);
        onlyJava.removeAll(pythonDevs);
        System.out.println("Java only    : " + onlyJava);

        Set<String> symmetric = new TreeSet<>(union);
        symmetric.removeAll(intersection);
        System.out.println("Exactly one  : " + symmetric);

        System.out.println("Is subset?   : " + javaDevs.containsAll(Set.of("Asha", "Dev")));
    }
}
```

**Expected Output:**

```text
Union        : [Asha, Bala, Chitra, Dev, Esha]
Intersection : [Chitra, Dev]
Java only    : [Asha, Bala]
Exactly one  : [Asha, Bala, Esha]
Is subset?   : true
```

| Operation | Method |
|---|---|
| Union | `addAll` |
| Intersection | `retainAll` |
| Difference | `removeAll` |
| Subset | `containsAll` |

(`TreeSet` copies were used so the output is sorted and predictable.)

---

## 7. Custom Objects in Sets

```java
import java.util.HashSet;
import java.util.Objects;
import java.util.Set;
import java.util.TreeSet;

public class CustomObjectSetDemo {
    static class Student implements Comparable<Student> {
        final int roll;
        final String name;
        Student(int roll, String name) { this.roll = roll; this.name = name; }

        @Override public boolean equals(Object o) {
            return o instanceof Student s && roll == s.roll;
        }
        @Override public int hashCode() { return Objects.hash(roll); }
        @Override public int compareTo(Student other) { return Integer.compare(roll, other.roll); }
        @Override public String toString() { return roll + ":" + name; }
    }

    static class NoEqualsStudent {
        final int roll;
        NoEqualsStudent(int roll) { this.roll = roll; }
    }

    public static void main(String[] args) {
        Set<Student> hashSet = new HashSet<>();
        hashSet.add(new Student(2, "Ravi"));
        hashSet.add(new Student(2, "Ravi Kumar"));   // same roll → duplicate
        hashSet.add(new Student(1, "Asha"));
        System.out.println("HashSet size: " + hashSet.size());

        Set<Student> treeSet = new TreeSet<>(hashSet);
        System.out.println("TreeSet     : " + treeSet);

        Set<NoEqualsStudent> broken = new HashSet<>();
        broken.add(new NoEqualsStudent(5));
        broken.add(new NoEqualsStudent(5));
        System.out.println("Without equals/hashCode size: " + broken.size());
    }
}
```

**Expected Output:**

```text
HashSet size: 2
TreeSet     : [1:Asha, 2:Ravi]
Without equals/hashCode size: 2
```

- With `equals`/`hashCode` based on `roll`, two students with roll 2 are duplicates.
- Without them, two "identical" objects are different (identity comparison) → both are stored.
- `TreeSet` needs `Comparable` (or a `Comparator`); otherwise `ClassCastException` on `add`.

---

## 8. Interview Questions with Complete Answers

### Basic Level

**Q1. How does HashSet ensure there are no duplicates?**

`HashSet` stores each element as a key in an internal `HashMap`. `add(e)` calls `map.put(e, PRESENT)`. A `HashMap` key is unique by `hashCode()` + `equals()`, so if an equal key exists, `put` returns the old value and `add` returns `false` without adding.

**Q2. Can a HashSet contain null? A TreeSet?**

`HashSet` and `LinkedHashSet` allow one `null`. `TreeSet` with natural ordering does not — comparing `null` throws `NullPointerException`. A `TreeSet` with a custom null-friendly comparator (`Comparator.nullsFirst(...)`) can accept `null`.

**Q3. Does Set allow index-based access?**

No. There is no `get(int)` method. Iterate it, or convert it to a `List` if index access is needed.

### Intermediate Level

**Q4. Why does TreeSet not use `equals()`?**

`TreeSet` is a binary search tree; to place and find elements it compares them with `compareTo()`/`compare()`. If the comparison returns 0, the element is considered already present. Therefore the comparator should be **consistent with equals**; otherwise the set may behave strangely (e.g., case-insensitive comparator merges "java" and "JAVA").

**Q5. What happens if you add an object to a HashSet and then change the fields used in `hashCode()`?**

The object stays in the bucket for its old hash code. `contains()` and `remove()` compute the new hash code and look in a different bucket, so they fail — the object becomes "lost" inside the set (a memory leak risk). Use immutable fields for hashing.

**Q6. What is the time complexity of TreeSet operations, and why?**

O(log n) for `add`, `remove`, `contains`, `floor`, `ceiling` etc., because the Red-Black tree is kept balanced, so its height is O(log n).

### Advanced Level

**Q7. How does LinkedHashSet maintain insertion order?**

It is backed by a `LinkedHashMap`, whose entries contain extra `before` and `after` pointers forming a doubly linked list in insertion order. Iteration follows this list instead of the bucket array.

**Q8. How would you find the first repeated element in an array using a Set?**

Iterate the array and call `set.add(x)`; the first time `add` returns `false`, `x` is the first repeated element. O(n) time, O(n) space.

**Q9. How do you create a thread-safe Set?**

`Collections.synchronizedSet(new HashSet<>())`, `ConcurrentHashMap.newKeySet()` (high concurrency), `CopyOnWriteArraySet` (small, read-heavy), or `ConcurrentSkipListSet` (sorted, concurrent).

---

## 9. Follow-Up Questions

**F1. What does `Set.of()` do with duplicates?**

`Set.of("a", "a")` throws `IllegalArgumentException` (duplicate element) at creation. It also rejects `null`, and its iteration order is unspecified (it may change between runs).

**F2. What is `EnumSet`?**

A highly efficient `Set` implementation for enum types, internally using bit vectors. Example: `EnumSet.of(Day.SAT, Day.SUN)`. Much faster and smaller than `HashSet` for enums.

**F3. Which Set would you use for a leaderboard of unique scores, always sorted?**

`TreeSet<Integer>` (or `TreeSet` with `Comparator.reverseOrder()` for highest first), using `first()`/`last()` and `headSet()` for queries.

---

## 10. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Expecting HashSet to keep insertion order | Use LinkedHashSet. |
| Custom objects without `equals`/`hashCode` in HashSet | Duplicates are stored. |
| Adding non-comparable objects to TreeSet | `ClassCastException`. |
| Adding `null` to TreeSet | `NullPointerException`. |
| Comparator inconsistent with `equals` | Unexpected "duplicates" removed. |
| Mutating elements inside a HashSet | Elements become unfindable. |

---

## 11. Quick Revision

- `Set` = no duplicates; no index; `add()` returns false for duplicates.
- **HashSet**: backed by HashMap (`PRESENT` dummy value), O(1), no order, one null.
- **LinkedHashSet**: backed by LinkedHashMap, insertion order, O(1).
- **TreeSet**: backed by TreeMap (Red-Black tree), sorted, O(log n), no null, uses `compareTo`; navigation: `first, last, floor, ceiling, higher, lower, headSet, tailSet, subSet, descendingSet, pollFirst`.
- Set operations: `addAll` (union), `retainAll` (intersection), `removeAll` (difference), `containsAll` (subset).

---

[← Previous: List](02-List-ArrayList-LinkedList-Vector-Stack.md) | [Back to README](../README.md) | [Next: Queue and Deque →](04-Queue-PriorityQueue-Deque.md)
