# Chapter 5.9 — Collection Comparisons and Interview Questions

[← Previous: Comparable and Comparator](08-Comparable-and-Comparator.md) | [Back to README](../README.md) | [Next Part: Generics →](../06-Generics/01-Generic-Classes-Methods-Interfaces.md)

---

## What You Will Find Here

1. All the important **comparison tables** in one place, for fast revision.
2. **Scenario questions** — "Which collection would you use for …?"
3. **Mixed interview questions** with complete answers, including coding questions.

For detailed teaching, follow the chapter links.

---

## Part A — Master Comparison Tables

### A1. List vs Set vs Queue vs Map

| Feature | List | Set | Queue | Map |
|---|---|---|---|---|
| Duplicates | ✅ | ❌ | ✅ | Keys ❌, values ✅ |
| Order | Insertion (index) | Depends (none / insertion / sorted) | FIFO / priority | Depends |
| Index access | ✅ `get(i)` | ❌ | ❌ | By key |
| `null` | ✅ (most) | One (Hash/LinkedHash) | Usually ❌ (ArrayDeque, PriorityQueue) | One null key (HashMap) |
| Extends `Collection` | ✅ | ✅ | ✅ | ❌ |
| Main classes | ArrayList, LinkedList | HashSet, LinkedHashSet, TreeSet | ArrayDeque, PriorityQueue, LinkedList | HashMap, LinkedHashMap, TreeMap |

### A2. ArrayList vs LinkedList vs Vector

| Feature | ArrayList | LinkedList | Vector |
|---|---|---|---|
| Structure | Dynamic array | Doubly linked list | Dynamic array |
| get(i) | O(1) | O(n) | O(1) |
| Insert/remove at front | O(n) | O(1) | O(n) |
| Insert at end | Amortised O(1) | O(1) | Amortised O(1) |
| Growth | 1.5× | — (per node) | 2× |
| Synchronized | ❌ | ❌ | ✅ |
| Implements Deque | ❌ | ✅ | ❌ |
| Memory | Low | High | Low |
| Status | Default | Special cases | Legacy |

([Chapter 5.2](02-List-ArrayList-LinkedList-Vector-Stack.md))

### A3. HashSet vs LinkedHashSet vs TreeSet

| Feature | HashSet | LinkedHashSet | TreeSet |
|---|---|---|---|
| Backed by | HashMap | LinkedHashMap | TreeMap (Red-Black tree) |
| Order | None | Insertion | Sorted |
| Time | O(1) | O(1) | O(log n) |
| `null` | One | One | ❌ |
| Duplicate check | hashCode + equals | hashCode + equals | compareTo / compare |

([Chapter 5.3](03-Set-HashSet-LinkedHashSet-TreeSet.md))

### A4. HashMap vs LinkedHashMap vs TreeMap vs Hashtable vs ConcurrentHashMap

| Feature | HashMap | LinkedHashMap | TreeMap | Hashtable | ConcurrentHashMap |
|---|---|---|---|---|---|
| Order | None | Insertion/access | Sorted keys | None | None |
| Time | O(1) | O(1) | O(log n) | O(1) | O(1) |
| null key / values | 1 / ✅ | 1 / ✅ | ❌ / ✅ | ❌ / ❌ | ❌ / ❌ |
| Thread-safe | ❌ | ❌ | ❌ | ✅ (one lock) | ✅ (per-bin locking + CAS) |
| Iterator | Fail-fast | Fail-fast | Fail-fast | Fail-fast (+ Enumeration) | Weakly consistent |
| Status | Default | Order/LRU | Sorted | Legacy | Concurrency |

([Chapter 5.5](05-Map-HashMap-LinkedHashMap-TreeMap-Hashtable.md), [5.6](06-Hashing-and-HashMap-Internal-Working.md))

### A5. Iterator vs ListIterator vs Enumeration

| Feature | Iterator | ListIterator | Enumeration |
|---|---|---|---|
| Applies to | All collections | Lists | Legacy (Vector, Hashtable) |
| Direction | Forward | Both | Forward |
| Remove | ✅ | ✅ | ❌ |
| Add / set | ❌ | ✅ | ❌ |
| Fail-fast | ✅ | ✅ | ❌ |

([Chapter 5.7](07-Iterator-ListIterator-Fail-Fast.md))

### A6. Comparable vs Comparator

| Feature | Comparable | Comparator |
|---|---|---|
| Package | java.lang | java.util |
| Method | compareTo(o) | compare(a, b) |
| Location | Inside the class | Outside (lambda/class) |
| Orders | One natural order | Many |

([Chapter 5.8](08-Comparable-and-Comparator.md))

### A7. Array vs ArrayList

| Feature | Array | ArrayList |
|---|---|---|
| Size | Fixed | Dynamic |
| Primitives | ✅ | ❌ (wrappers) |
| Generics | ❌ | ✅ |
| Length | `arr.length` | `list.size()` |
| Performance | Faster, less memory | Small overhead |
| Multi-dimensional | ✅ | Lists of lists |

### A8. Collections.synchronizedMap vs ConcurrentHashMap

| Feature | `Collections.synchronizedMap(map)` | `ConcurrentHashMap` |
|---|---|---|
| Locking | One lock for the whole map (every method) | Fine-grained: CAS for empty bins, `synchronized` on individual bins |
| Read performance | Blocks other threads | Reads are mostly lock-free |
| Iteration | Must manually `synchronized(map) { ... }`, else CME possible | Weakly consistent; safe without locking |
| `null` keys/values | Depends on the wrapped map (HashMap allows) | ❌ Not allowed |
| Atomic compound ops | No (except via external locking) | ✅ `putIfAbsent`, `computeIfAbsent`, `merge` are atomic |
| Scalability | Poor under contention | High |

---

## Part B — Scenario Questions ("Which collection would you use?")

| # | Scenario | Best choice | Why |
|---|---|---|---|
| 1 | Store students' names; read by index often | `ArrayList` | O(1) index access |
| 2 | Unique email IDs, order not important | `HashSet` | O(1) uniqueness checks |
| 3 | Unique tags in the order the user added them | `LinkedHashSet` | Uniqueness + insertion order |
| 4 | Leaderboard sorted by score; need top/bottom quickly | `TreeSet`/`TreeMap` (or `PriorityQueue` for top-k only) | Sorted, `first()/last()` |
| 5 | Count frequency of words | `HashMap<String, Integer>` with `merge` | O(1) updates |
| 6 | Word frequency printed alphabetically | `TreeMap<String, Integer>` | Sorted keys |
| 7 | Cache that evicts least recently used | `LinkedHashMap` (accessOrder + `removeEldestEntry`) | LRU behaviour built in |
| 8 | Task scheduler: highest priority first | `PriorityQueue` with comparator | Heap ordering |
| 9 | Undo/redo, browser back button | `ArrayDeque` as stack | Fast LIFO |
| 10 | BFS traversal | `ArrayDeque` as queue | Fast FIFO |
| 11 | Map shared by many threads (e.g., session store) | `ConcurrentHashMap` | Thread-safe, scalable |
| 12 | List of event listeners, rarely changed, read very often by threads | `CopyOnWriteArrayList` | Lock-free reads, safe iteration |
| 13 | Producer–consumer between threads | `ArrayBlockingQueue` / `LinkedBlockingQueue` | Blocking `put`/`take` |
| 14 | Map with enum keys (`Day → schedule`) | `EnumMap` | Compact, fast, ordered by enum |
| 15 | Find tax slab for an income (range lookup) | `TreeMap.floorEntry()` | Navigable keys |
| 16 | Fixed configuration that must never change | `Map.of(...)` / `List.of(...)` | Immutable |

---

## Part C — Interview Questions with Complete Answers

### Basic Level

**C1. What is the difference between `Collection` and `Collections`?**

`Collection` is the root interface for `List`, `Set` and `Queue`. `Collections` is a utility class with static methods such as `sort`, `reverse`, `shuffle`, `max`, `min`, `frequency`, `unmodifiableList` and `synchronizedList`.

**C2. Which collections allow `null`?**

`ArrayList`, `LinkedList`, `Vector` (any number of nulls); `HashSet`, `LinkedHashSet` (one null); `HashMap`, `LinkedHashMap` (one null key, many null values); `TreeMap` (null values but no null key with natural ordering). Not allowed: `TreeSet` (natural ordering), `Hashtable`, `ConcurrentHashMap`, `ArrayDeque`, `PriorityQueue`, `List.of/Set.of/Map.of`.

**C3. What is the default capacity of `ArrayList`, `HashMap`, `Hashtable` and `Vector`?**

`ArrayList` 10 (lazy), `HashMap` 16 (load factor 0.75), `Hashtable` 11 (load factor 0.75), `Vector` 10.

### Intermediate Level

**C4. How do you convert between arrays and lists?**

Array → List: `Arrays.asList(arr)` (fixed-size view), `new ArrayList<>(Arrays.asList(arr))` (resizable), `List.of(arr)` (immutable). List → Array: `list.toArray(new String[0])` or `list.toArray(String[]::new)` (Java 11).

**C5. How do you make a collection read-only?**

`Collections.unmodifiableList/Set/Map(c)` (a read-only view), or `List.copyOf(c)` / `Set.copyOf` / `Map.copyOf` (immutable copies, Java 10), or `List.of(...)` for literals.

**C6. How do you sort a Map by keys and by values?**

By keys: `new TreeMap<>(map)`. By values: stream the entries, sort with `Map.Entry.comparingByValue()`, and collect into a `LinkedHashMap` (example in [Chapter 5.5, Q4](05-Map-HashMap-LinkedHashMap-TreeMap-Hashtable.md)).

**C7. What is the difference between `remove()` methods in `List` and `Set`?**

`List` has `remove(int index)` and `remove(Object o)`; with `List<Integer>`, `remove(1)` removes **index** 1. `Set` has only `remove(Object o)`.

**C8. What is the difference between `HashMap` and `HashSet`?**

`HashMap` stores key–value pairs (`put`, `get`); `HashSet` stores only elements (`add`, `contains`). Internally, `HashSet` uses a `HashMap` where each element is a key and the value is a dummy constant. Both use hashing, have O(1) average operations and allow one null (key/element).

### Advanced Level

**C9. How does `ConcurrentHashMap` achieve thread safety without locking the whole map?**

In Java 8+, it uses an array of bins like `HashMap`. Inserting into an empty bin uses a lock-free **CAS** (compare-and-swap). Inserting into a non-empty bin locks **only that bin's first node** with `synchronized`. Reads are mostly lock-free using `volatile` reads. Resizing is done cooperatively by multiple threads. Size is tracked with striped counters (`LongAdder`-like). (Java 7 used 16 lock "segments".) It forbids nulls to avoid ambiguity in concurrent `get()` results. More in [Part 7](../07-Multithreading-and-Concurrency/08-CompletableFuture-and-Concurrent-Collections.md).

**C10. Why should you size collections in advance when possible?**

To avoid repeated resizing (array copying for `ArrayList`, rehashing for `HashMap`). If you know you'll add ~10,000 elements, `new ArrayList<>(10_000)` or `HashMap.newHashMap(10_000)` (Java 19+) / `new HashMap<>((int) (10_000 / 0.75f) + 1)` saves time and garbage.

**C11. What is the difference between `Iterator.remove()` and `Collection.remove()` during iteration?**

`Iterator.remove()` removes the current element and updates the iterator's expected modification count, so iteration continues safely. `Collection.remove()` changes `modCount` without informing the iterator, so the next `next()` throws `ConcurrentModificationException`.

**C12. What are `WeakHashMap` and `IdentityHashMap` used for?**

`WeakHashMap` holds keys weakly; when a key has no other strong references, its entry is removed by GC — useful for caches of metadata tied to object lifetimes. `IdentityHashMap` uses `==` instead of `equals()` for keys — used for tracking object identity (e.g., visited objects during deep copy/serialization).

---

## Part D — Coding Questions

### D1. Count the frequency of each element and print the most frequent one

```java
import java.util.HashMap;
import java.util.Map;

public class MostFrequent {
    public static void main(String[] args) {
        int[] arr = {3, 7, 3, 1, 7, 3, 9, 7, 7};
        Map<Integer, Integer> freq = new HashMap<>();
        for (int x : arr) {
            freq.merge(x, 1, Integer::sum);
        }

        int bestKey = -1, bestCount = 0;
        for (Map.Entry<Integer, Integer> e : freq.entrySet()) {
            if (e.getValue() > bestCount) {
                bestKey = e.getKey();
                bestCount = e.getValue();
            }
        }
        System.out.println("Frequencies: " + new java.util.TreeMap<>(freq));
        System.out.println("Most frequent: " + bestKey + " (" + bestCount + " times)");
    }
}
```

**Expected Output:**

```text
Frequencies: {1=1, 3=3, 7=4, 9=1}
Most frequent: 7 (4 times)
```

**Approach:** count with `merge` (O(n)), then one pass over the entries to find the maximum (O(k)). **Time O(n), space O(k)**.

### D2. Find the first non-repeating element in a list

```java
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

public class FirstUniqueElement {
    public static void main(String[] args) {
        List<String> orders = List.of("tea", "coffee", "tea", "juice", "coffee", "water");
        Map<String, Integer> counts = new LinkedHashMap<>();       // keeps first-seen order
        for (String o : orders) counts.merge(o, 1, Integer::sum);

        String firstUnique = null;
        for (Map.Entry<String, Integer> e : counts.entrySet()) {
            if (e.getValue() == 1) { firstUnique = e.getKey(); break; }
        }
        System.out.println("First non-repeating: " + firstUnique);
    }
}
```

**Expected Output:**

```text
First non-repeating: juice
```

`LinkedHashMap` keeps keys in first-appearance order, so the first entry with count 1 is the answer. **O(n) time, O(k) space.**

### D3. Group employees by department

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.TreeMap;

public class GroupByDepartment {
    record Employee(String name, String dept) { }

    public static void main(String[] args) {
        List<Employee> staff = List.of(new Employee("Ravi", "IT"), new Employee("Asha", "HR"),
                new Employee("Kiran", "IT"), new Employee("Dev", "Sales"), new Employee("Bala", "HR"));

        Map<String, List<String>> byDept = new TreeMap<>();
        for (Employee e : staff) {
            byDept.computeIfAbsent(e.dept(), d -> new ArrayList<>()).add(e.name());
        }
        System.out.println(byDept);
    }
}
```

**Expected Output:**

```text
{HR=[Asha, Bala], IT=[Ravi, Kiran], Sales=[Dev]}
```

`computeIfAbsent` creates the list the first time a department appears. (With streams: `Collectors.groupingBy` — [Part 8](../08-Java-8-and-Modern-Java/04-Collectors-and-Advanced-Streams.md).)

### D4. Check if two lists have common elements and find them

```java
import java.util.HashSet;
import java.util.List;
import java.util.Set;
import java.util.TreeSet;

public class CommonElements {
    public static void main(String[] args) {
        List<Integer> a = List.of(5, 1, 9, 3, 7);
        List<Integer> b = List.of(8, 3, 6, 5, 2);

        Set<Integer> setA = new HashSet<>(a);
        Set<Integer> common = new TreeSet<>();
        for (int x : b) {
            if (setA.contains(x)) common.add(x);
        }
        System.out.println("Common: " + common);
        System.out.println("Disjoint? " + java.util.Collections.disjoint(a, b));
    }
}
```

**Expected Output:**

```text
Common: [3, 5]
Disjoint? false
```

Using a `HashSet` makes each lookup O(1), so the total is **O(n + m)** instead of O(n × m) with nested loops.

---

## Part E — Rapid Revision

- Default choices: **ArrayList, HashMap, HashSet, ArrayDeque**.
- Order needed → Linked versions; sorted → Tree versions; priority → PriorityQueue.
- Threads → ConcurrentHashMap, CopyOnWriteArrayList, BlockingQueue.
- Legacy → Vector, Stack, Hashtable, Enumeration.
- Fail-fast iterators throw CME; use `iterator.remove()`/`removeIf()`.
- `equals()` + `hashCode()` for hash collections; `compareTo()`/`Comparator` for tree collections.
- Counting → `merge`; grouping → `computeIfAbsent`; LRU → `LinkedHashMap`.

---

[← Previous: Comparable and Comparator](08-Comparable-and-Comparator.md) | [Back to README](../README.md) | [Next Part: Generics →](../06-Generics/01-Generic-Classes-Methods-Interfaces.md)
