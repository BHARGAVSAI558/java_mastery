# Chapter 5.1 — Collections Framework Overview and Hierarchy

[← Previous Part: try-with-resources](../04-Exception-Handling/04-try-with-resources-and-Best-Practices.md) | [Back to README](../README.md) | [Next: List →](02-List-ArrayList-LinkedList-Vector-Stack.md)

---

## What You Will Learn

- What the Collections Framework is and why it exists
- Collection vs Collections vs Collection Framework
- The complete hierarchy of interfaces and classes
- Core interfaces: `Iterable`, `Collection`, `List`, `Set`, `Queue`, `Deque`, `Map`
- Common methods of the `Collection` interface
- The `Collections` utility class
- Immutable/unmodifiable collections (`List.of`, `Collections.unmodifiableList`)
- How to choose the right collection

---

## 1. Collections Framework

### 1.1 Interview Definition

> **The Java Collections Framework is a unified architecture in the `java.util` package for storing and manipulating groups of objects. It consists of interfaces, such as `Collection`, `List`, `Set`, `Queue`, `Deque` and `Map`, which define abstract data types; implementation classes, such as `ArrayList`, `LinkedList`, `HashSet`, `TreeSet`, `PriorityQueue`, `ArrayDeque`, `HashMap` and `TreeMap`, which provide concrete data structures; and algorithms, mainly in the `Collections` utility class, for operations such as sorting, searching and shuffling. It provides dynamic sizing, type safety through generics, reusable high-performance data structures, and interoperability through common interfaces.**

### 1.2 Why Do We Need It?

Before Java 1.2, Java had only arrays and a few classes (`Vector`, `Hashtable`, `Stack`) with no common design.

**Problems with arrays:**
1. **Fixed size** — must know the size in advance.
2. **No built-in operations** — insert in the middle, remove, search, sort, check duplicates — you write them yourself.
3. **No standard data structures** — no ready-made linked list, set, queue, map.

**The Collections Framework provides:**
1. **Dynamic size** — collections grow and shrink.
2. **Ready-made data structures** — dynamic arrays, linked lists, hash tables, trees, heaps.
3. **Common interfaces** — write code against `List` and swap `ArrayList` for `LinkedList` without changes.
4. **Algorithms** — `sort`, `binarySearch`, `reverse`, `shuffle`, `max`, `min`, `frequency`.
5. **Type safety** with generics (Java 5) — `List<String>` holds only strings.
6. **Performance** — well-tested, optimised implementations.

### 1.3 Collection vs Collections vs Collection Framework

| Term | What it is |
|---|---|
| **Collection** | The root **interface** (`java.util.Collection<E>`) for lists, sets and queues |
| **Collections** | A **utility class** (`java.util.Collections`) with static methods: `sort`, `reverse`, `unmodifiableList`, `synchronizedList`, `max`, `frequency`… |
| **Collections Framework** | The whole **architecture**: interfaces + implementations + algorithms |

### 1.4 Real-World Example

Think of a **kitchen** with different containers: a **rack of numbered jars** (List — ordered, indexed), a **bowl of unique fruits** where duplicates don't make sense (Set), a **ticket queue** at the counter (Queue), and a **spice box with labels** where each label maps to a spice (Map). Each container is designed for a specific way of storing and finding things.

---

## 2. The Hierarchy

### 2.1 Diagram

```text
                               Iterable<E>
                                    │
                              Collection<E>
          ┌───────────────────┬─────┴──────────────┬───────────────────┐
        List<E>             Set<E>              Queue<E>               │
          │                   │                    │                   │
  ┌───────┼─────────┐    ┌────┴─────┐        ┌─────┴──────┐            │
ArrayList │      Vector HashSet  SortedSet  PriorityQueue  Deque<E>    │
     LinkedList*    │      │        │                      │           │
                  Stack LinkedHashSet NavigableSet    ┌────┴─────┐     │
                                        │          ArrayDeque LinkedList*
                                      TreeSet

   * LinkedList implements BOTH List and Deque

                               Map<K,V>   (NOT a Collection)
                    ┌─────────────┼──────────────┬──────────────┐
                 HashMap      Hashtable       SortedMap     ConcurrentMap
                    │                            │              │
              LinkedHashMap                NavigableMap  ConcurrentHashMap
                                                 │
                                              TreeMap
```

### 2.2 Core Interfaces

| Interface | Ordered? | Duplicates? | Key feature | Main implementations |
|---|---|---|---|---|
| `List` | ✅ insertion order, index-based | ✅ Allowed | Access by index | `ArrayList`, `LinkedList`, `Vector`, `Stack` |
| `Set` | Depends on implementation | ❌ Not allowed | Uniqueness | `HashSet`, `LinkedHashSet`, `TreeSet` |
| `SortedSet`/`NavigableSet` | ✅ sorted | ❌ | Range queries (`floor`, `ceiling`) | `TreeSet` |
| `Queue` | FIFO (usually) or priority | ✅ | Process elements in order | `PriorityQueue`, `LinkedList`, `ArrayDeque` |
| `Deque` | Both ends | ✅ | Stack + queue | `ArrayDeque`, `LinkedList` |
| `Map` | Depends | Keys ❌, values ✅ | Key → value lookup | `HashMap`, `LinkedHashMap`, `TreeMap`, `Hashtable`, `ConcurrentHashMap` |

### 2.3 Why Is `Map` Not a `Collection`?

`Collection` is about **single elements** (`add(E e)`), while `Map` stores **key–value pairs** (`put(K k, V v)`). Their operations don't fit the same interface. But a `Map` provides **collection views**: `keySet()` (a `Set`), `values()` (a `Collection`), `entrySet()` (a `Set<Map.Entry<K,V>>`). `Map` is still part of the Collections **Framework**.

---

## 3. The `Collection` Interface — Common Methods

| Method | Description |
|---|---|
| `boolean add(E e)` | Adds an element; returns `false` if not added (e.g., duplicate in a Set) |
| `boolean addAll(Collection<? extends E> c)` | Adds all elements |
| `boolean remove(Object o)` | Removes one occurrence |
| `boolean removeAll(Collection<?> c)` | Removes all elements present in `c` |
| `boolean retainAll(Collection<?> c)` | Keeps only elements also in `c` (intersection) |
| `boolean removeIf(Predicate<? super E> p)` | Removes matching elements (Java 8) |
| `boolean contains(Object o)` | Membership check (uses `equals()`) |
| `boolean containsAll(Collection<?> c)` | Subset check |
| `int size()` / `boolean isEmpty()` | Size checks |
| `void clear()` | Removes everything |
| `Iterator<E> iterator()` | Iterator for traversal |
| `Object[] toArray()` / `T[] toArray(T[] a)` | Convert to array |
| `Stream<E> stream()` / `parallelStream()` | Stream API (Java 8) |
| `void forEach(Consumer<? super T> action)` | From `Iterable` (Java 8) |

### 3.1 Java Example — Programming to Interfaces

```java
import java.util.ArrayList;
import java.util.Collection;
import java.util.HashSet;
import java.util.LinkedList;
import java.util.List;
import java.util.TreeSet;

public class CollectionBasicsDemo {

    static void describe(String label, Collection<String> c) {
        c.add("Java");
        c.add("Python");
        c.add("Java");             // duplicate
        c.add("C");
        System.out.printf("%-10s size=%d contents=%s contains(C)=%b%n", label, c.size(), c, c.contains("C"));
    }

    public static void main(String[] args) {
        describe("ArrayList", new ArrayList<>());
        describe("LinkedList", new LinkedList<>());
        describe("HashSet", new HashSet<>());
        describe("TreeSet", new TreeSet<>());

        List<Integer> numbers = new ArrayList<>(List.of(5, 12, 7, 20, 3, 12));
        numbers.removeIf(n -> n > 10);                    // Java 8
        System.out.println("After removeIf(>10): " + numbers);

        List<Integer> a = new ArrayList<>(List.of(1, 2, 3, 4));
        List<Integer> b = List.of(3, 4, 5);
        a.retainAll(b);                                   // intersection
        System.out.println("Intersection: " + a);

        String[] arr = numbers.stream().map(String::valueOf).toArray(String[]::new);
        System.out.println("As array length: " + arr.length);
    }
}
```

**Expected Output:**

```text
ArrayList  size=4 contents=[Java, Python, Java, C] contains(C)=true
LinkedList size=4 contents=[Java, Python, Java, C] contains(C)=true
HashSet    size=3 contents=[Java, C, Python] contains(C)=true
TreeSet    size=3 contents=[C, Java, Python] contains(C)=true
After removeIf(>10): [5, 7, 3]
Intersection: [3, 4]
As array length: 3
```

**Explanation:**
- The `describe()` method works with **any** `Collection` — the power of programming to interfaces.
- Lists keep insertion order and allow duplicates (size 4).
- Sets reject the duplicate `"Java"` (size 3). `HashSet` order depends on hash codes (not insertion order); `TreeSet` keeps elements **sorted**.
- `removeIf` and `retainAll` perform bulk operations without manual loops.

> Note: `HashSet` iteration order is not guaranteed by the specification; the order shown is what JDK 21 produces for these strings. Never rely on it in real code.

---

## 4. The `Collections` Utility Class

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class CollectionsUtilityDemo {
    public static void main(String[] args) {
        List<Integer> marks = new ArrayList<>(List.of(72, 95, 60, 88, 95, 41));

        Collections.sort(marks);
        System.out.println("sort        : " + marks);
        System.out.println("binarySearch(88): index " + Collections.binarySearch(marks, 88));
        Collections.reverse(marks);
        System.out.println("reverse     : " + marks);
        System.out.println("max / min   : " + Collections.max(marks) + " / " + Collections.min(marks));
        System.out.println("frequency(95): " + Collections.frequency(marks, 95));
        Collections.swap(marks, 0, marks.size() - 1);
        System.out.println("swap ends   : " + marks);
        marks.sort(Collections.reverseOrder());
        System.out.println("reverseOrder: " + marks);

        List<String> readOnly = Collections.unmodifiableList(new ArrayList<>(List.of("a", "b")));
        try {
            readOnly.add("c");
        } catch (UnsupportedOperationException e) {
            System.out.println("unmodifiableList.add -> UnsupportedOperationException");
        }

        List<String> empty = Collections.emptyList();
        List<String> single = Collections.singletonList("only");
        List<String> copies = Collections.nCopies(3, "hi");
        System.out.println("empty=" + empty + ", single=" + single + ", nCopies=" + copies);

        List<Integer> syncList = Collections.synchronizedList(new ArrayList<>());   // thread-safe wrapper
        syncList.add(1);
        System.out.println("synchronizedList: " + syncList);
    }
}
```

**Expected Output:**

```text
sort        : [41, 60, 72, 88, 95, 95]
binarySearch(88): index 3
reverse     : [95, 95, 88, 72, 60, 41]
max / min   : 95 / 41
frequency(95): 2
swap ends   : [41, 95, 88, 72, 60, 95]
reverseOrder: [95, 95, 88, 72, 60, 41]
unmodifiableList.add -> UnsupportedOperationException
empty=[], single=[only], nCopies=[hi, hi, hi]
synchronizedList: [1]
```

---

## 5. Unmodifiable vs Immutable Collections

| Way | Example | Can the underlying data change? | `null` allowed? |
|---|---|---|---|
| `Collections.unmodifiableList(list)` | read-only **view** | ✅ Yes — changes to the original list show through the view | ✅ |
| `List.of(...)`, `Set.of`, `Map.of` (Java 9) | truly **immutable** | ❌ No | ❌ (`NullPointerException`) |
| `List.copyOf(c)` (Java 10) | immutable **copy** | ❌ No | ❌ |
| `stream().toList()` (Java 16) | unmodifiable list | ❌ | ✅ |
| `Arrays.asList(arr)` | **fixed-size** view of array | `set()` ✅, `add/remove` ❌ | ✅ |

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class UnmodifiableDemo {
    public static void main(String[] args) {
        List<String> original = new ArrayList<>(List.of("x", "y"));
        List<String> view = Collections.unmodifiableList(original);
        List<String> copy = List.copyOf(original);

        original.add("z");                          // modify the original
        System.out.println("view sees change: " + view);
        System.out.println("copy unchanged  : " + copy);

        try {
            List.of("a", null);
        } catch (NullPointerException e) {
            System.out.println("List.of does not allow null");
        }
    }
}
```

**Expected Output:**

```text
view sees change: [x, y, z]
copy unchanged  : [x, y]
List.of does not allow null
```

---

## 6. Choosing the Right Collection

```text
Need key → value lookup?
 ├── Yes → Map
 │     ├── fastest, order not needed ............ HashMap
 │     ├── keep insertion (or access) order ..... LinkedHashMap
 │     ├── sorted keys / range queries .......... TreeMap
 │     └── many threads ......................... ConcurrentHashMap
 └── No
      ├── Need uniqueness? → Set
      │     ├── fastest ............................ HashSet
      │     ├── insertion order .................... LinkedHashSet
      │     └── sorted ............................. TreeSet
      ├── Need FIFO / priority / stack?
      │     ├── priority order ..................... PriorityQueue
      │     └── queue or stack (both ends) ......... ArrayDeque
      └── Ordered list with index access / duplicates → List
            ├── mostly reads, index access ......... ArrayList (default choice)
            └── frequent add/remove at ends via iterator .. LinkedList (rarely better in practice)
```

---

## 7. Interview Questions with Complete Answers

### Basic Level

**Q1. What is the Collections Framework?**

A unified architecture in `java.util` for storing and processing groups of objects. It includes interfaces (`List`, `Set`, `Queue`, `Map`), implementations (`ArrayList`, `HashSet`, `HashMap`…) and algorithms (`Collections.sort`, `binarySearch`…). It gives dynamic size, ready-made data structures, type safety with generics and better performance than writing everything yourself.

**Q2. What is the difference between `Collection` and `Collections`?**

`Collection` is the root interface of lists, sets and queues. `Collections` is a final utility class with static helper methods like `sort()`, `reverse()`, `shuffle()`, `max()`, `unmodifiableList()` and `synchronizedList()`.

**Q3. What is the difference between List, Set and Map?**

`List` is an ordered collection that allows duplicates and index access. `Set` does not allow duplicates; ordering depends on the implementation. `Map` stores key–value pairs with unique keys; it is not a subtype of `Collection`.

### Intermediate Level

**Q4. Why doesn't `Map` extend `Collection`?**

Because a map stores pairs, not single elements; methods like `add(E)` don't make sense for it. It provides collection views instead: `keySet()`, `values()`, `entrySet()`.

**Q5. What is the difference between arrays and collections?**

| Arrays | Collections |
|---|---|
| Fixed size | Dynamic size |
| Primitives and objects | Objects only (wrappers for primitives) |
| `length` field, few operations | Rich API (`add`, `remove`, `contains`, `sort`) |
| Covariant, runtime type checks | Generics, compile-time type safety |
| Slightly faster, less memory | Small overhead |

**Q6. What is the difference between `Collections.unmodifiableList()` and `List.of()`?**

`unmodifiableList()` returns a **read-only view** of an existing list — you can't modify it through the view, but changes to the original list are visible. `List.of()` creates a truly **immutable** list that nobody can change, and it rejects `null` elements.

### Advanced Level

**Q7. Which collections are legacy?**

`Vector`, `Stack`, `Hashtable`, `Properties` and the `Enumeration` interface existed before Java 1.2. They were retrofitted into the framework. They are synchronized (slow) and generally replaced by `ArrayList`, `ArrayDeque`, `HashMap`/`ConcurrentHashMap` and `Iterator`.

**Q8. What are the time complexities of common collections?**

| Collection | get/access | add | remove | contains/search |
|---|---|---|---|---|
| `ArrayList` | O(1) by index | O(1) amortised at end, O(n) middle | O(n) | O(n) |
| `LinkedList` | O(n) | O(1) at ends | O(1) at ends / with iterator, O(n) search | O(n) |
| `HashSet`/`HashMap` | O(1) average | O(1) average | O(1) average | O(1) average (O(log n) worst in treeified buckets) |
| `LinkedHashSet`/`LinkedHashMap` | O(1) average | O(1) | O(1) | O(1) |
| `TreeSet`/`TreeMap` | O(log n) | O(log n) | O(log n) | O(log n) |
| `PriorityQueue` | peek O(1) | O(log n) | poll O(log n), remove(Object) O(n) | O(n) |
| `ArrayDeque` | ends O(1) | O(1) amortised | O(1) at ends | O(n) |

---

## 8. Follow-Up Questions

**F1. Why do collections store only objects?**

Because they are implemented with generics, and generics work only with reference types (type erasure to `Object`). Primitives are stored via autoboxing into wrapper objects (`int` → `Integer`). For large primitive data, arrays or primitive streams are more efficient.

**F2. Which collection would you use to store unique usernames in registration order?**

`LinkedHashSet<String>` — uniqueness plus insertion order with O(1) operations.

**F3. Is `ArrayList` thread-safe?**

No. Use `Collections.synchronizedList()`, `CopyOnWriteArrayList` (read-heavy), or external synchronization. See [Part 7](../07-Multithreading-and-Concurrency/08-CompletableFuture-and-Concurrent-Collections.md).

---

## 9. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Confusing `Collection` and `Collections` | Interface vs utility class. |
| Saying `Map` is a `Collection` | It isn't; it's part of the framework. |
| Relying on `HashSet`/`HashMap` iteration order | Not guaranteed; use Linked/Tree versions. |
| Modifying `List.of()` / `Arrays.asList()` lists | `UnsupportedOperationException`. |
| Declaring variables as concrete types | Prefer `List<String> list = new ArrayList<>();`. |

---

## 10. Quick Revision

- Framework = interfaces + implementations + algorithms (`java.util`).
- `Collection` (interface) vs `Collections` (utility class).
- `Iterable` → `Collection` → `List` / `Set` / `Queue` (→ `Deque`); `Map` separate.
- List: ordered, duplicates, index. Set: unique. Queue: FIFO/priority. Deque: both ends. Map: unique keys.
- `Collections`: sort, binarySearch, reverse, shuffle, max, min, frequency, swap, unmodifiableX, synchronizedX, emptyX, nCopies.
- `unmodifiableList` = view; `List.of` = immutable, no nulls; `Arrays.asList` = fixed-size.
- Default choices: `ArrayList`, `HashMap`, `HashSet`, `ArrayDeque`.

---

[← Previous Part: try-with-resources](../04-Exception-Handling/04-try-with-resources-and-Best-Practices.md) | [Back to README](../README.md) | [Next: List →](02-List-ArrayList-LinkedList-Vector-Stack.md)
