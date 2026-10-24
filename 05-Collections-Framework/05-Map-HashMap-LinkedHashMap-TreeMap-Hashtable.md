# Chapter 5.5 — Map: HashMap, LinkedHashMap, TreeMap and Hashtable

[← Previous: Queue and Deque](04-Queue-PriorityQueue-Deque.md) | [Back to README](../README.md) | [Next: HashMap Internal Working →](06-Hashing-and-HashMap-Internal-Working.md)

---

## What You Will Learn

- The `Map` interface and its important methods (including Java 8 methods)
- Iterating a map correctly (`entrySet`, `forEach`)
- `HashMap` (overview — internals in the next chapter)
- `LinkedHashMap` — insertion order and access order (LRU cache)
- `TreeMap` — sorted keys and `NavigableMap` methods
- `Hashtable` — legacy, synchronized
- **HashMap vs LinkedHashMap vs TreeMap vs Hashtable**, and **HashMap vs Hashtable**

---

## 1. The `Map` Interface

### 1.1 Interview Definition

> **`Map<K, V>` is an interface in `java.util` that stores data as key–value pairs, where each key is unique and maps to exactly one value; values may be duplicated. A map provides fast lookup of a value by its key. It does not extend `Collection`, but it provides collection views through `keySet()`, `values()` and `entrySet()`. Putting a value with an existing key replaces the old value. The main implementations are `HashMap` (no order, O(1) average), `LinkedHashMap` (insertion or access order), `TreeMap` (sorted by key, O(log n)), `Hashtable` (legacy, synchronized) and `ConcurrentHashMap` (thread-safe).**

### 1.2 Important Methods

| Method | Description |
|---|---|
| `V put(K key, V value)` | Insert or replace; returns the **previous** value (or `null`) |
| `V get(Object key)` | Value for the key, or `null` if absent |
| `V getOrDefault(Object key, V default)` | Value or a default (Java 8) |
| `boolean containsKey(Object key)` / `containsValue(Object v)` | O(1) average / O(n) |
| `V remove(Object key)` | Remove mapping; returns old value |
| `int size()`, `isEmpty()`, `clear()` | Basic operations |
| `Set<K> keySet()`, `Collection<V> values()`, `Set<Map.Entry<K,V>> entrySet()` | Views (backed by the map) |
| `V putIfAbsent(K, V)` | Put only if key absent (Java 8) |
| `V computeIfAbsent(K, Function)` | Compute and store a value if absent — great for grouping (Java 8) |
| `V computeIfPresent(K, BiFunction)` / `compute(K, BiFunction)` | Update with a function (Java 8) |
| `V merge(K, V, BiFunction)` | Combine old and new values — great for counting (Java 8) |
| `void forEach(BiConsumer)` | Iterate pairs (Java 8) |
| `void replaceAll(BiFunction)` | Transform all values (Java 8) |
| `Map.of(...)`, `Map.entry(k, v)`, `Map.copyOf(...)` | Immutable maps (Java 9/10) |

### 1.3 Real-World Example

A **dictionary**: the **word** is the key and its **meaning** is the value. Each word appears once; two words can have the same meaning. You look up meaning by word directly instead of reading the whole book.

### 1.4 Java Example — Map Methods and Iteration

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.TreeMap;

public class MapMethodsDemo {
    public static void main(String[] args) {
        Map<String, Integer> stock = new HashMap<>();
        System.out.println("put pen    -> previous: " + stock.put("pen", 50));
        System.out.println("put pen    -> previous: " + stock.put("pen", 45));   // replaces
        stock.put("book", 20);
        stock.put("bag", 5);

        System.out.println("get pen         : " + stock.get("pen"));
        System.out.println("get eraser      : " + stock.get("eraser"));
        System.out.println("getOrDefault    : " + stock.getOrDefault("eraser", 0));
        System.out.println("containsKey bag : " + stock.containsKey("bag"));
        System.out.println("containsValue 20: " + stock.containsValue(20));

        stock.putIfAbsent("book", 999);                 // ignored: key exists
        stock.putIfAbsent("eraser", 30);                // added
        stock.merge("pen", 10, Integer::sum);           // 45 + 10
        stock.computeIfPresent("bag", (k, v) -> v - 1); // 5 - 1
        stock.remove("eraser");

        // Iterate in sorted key order (TreeMap copy) using entrySet - the preferred way
        for (Map.Entry<String, Integer> e : new TreeMap<>(stock).entrySet()) {
            System.out.println(e.getKey() + " = " + e.getValue());
        }

        // Word frequency with merge
        String text = "to be or not to be";
        Map<String, Integer> freq = new TreeMap<>();
        for (String w : text.split(" ")) {
            freq.merge(w, 1, Integer::sum);
        }
        System.out.println("Frequency: " + freq);

        // Grouping with computeIfAbsent
        Map<Integer, List<String>> byLength = new TreeMap<>();
        for (String w : List.of("java", "go", "rust", "c", "kotlin", "sql")) {
            byLength.computeIfAbsent(w.length(), k -> new ArrayList<>()).add(w);
        }
        System.out.println("By length: " + byLength);

        byLength.forEach((len, words) -> System.out.println(len + " letters -> " + words.size() + " word(s)"));
    }
}
```

### 1.5 Code Explanation

- `put` returns the previous value: `null` the first time, `50` the second time (when 45 replaces 50).
- `get` returns `null` for a missing key; `getOrDefault` gives a safe default.
- `containsValue` must scan all values → O(n).
- `putIfAbsent` doesn't overwrite existing keys.
- `merge(key, 1, Integer::sum)`: if the key is absent, store 1; otherwise store `old + 1`. The cleanest way to **count**.
- `computeIfAbsent(key, k -> new ArrayList<>()).add(w)`: create the list only when needed, then add — the cleanest way to **group**.
- Iterating with `entrySet()` gives key and value together without an extra `get()` lookup per key.

### 1.6 Expected Output

```text
put pen    -> previous: null
put pen    -> previous: 50
get pen         : 45
get eraser      : null
getOrDefault    : 0
containsKey bag : true
containsValue 20: true
bag = 4
book = 20
pen = 55
Frequency: {be=2, not=1, or=1, to=2}
By length: {1=[c], 2=[go], 3=[sql], 4=[java, rust], 6=[kotlin]}
1 letters -> 1 word(s)
2 letters -> 1 word(s)
3 letters -> 1 word(s)
4 letters -> 2 word(s)
6 letters -> 1 word(s)
```

### 1.7 Ways to Iterate a Map

| Way | Code | Note |
|---|---|---|
| `entrySet()` | `for (Map.Entry<K,V> e : map.entrySet())` | **Best**: key + value, one lookup |
| `forEach` (Java 8) | `map.forEach((k, v) -> ...)` | Concise |
| `keySet()` + `get()` | `for (K k : map.keySet()) map.get(k)` | Extra lookup per key |
| `values()` | `for (V v : map.values())` | Values only |
| Iterator on `entrySet()` | `it.remove()` | Only safe way to remove while iterating (or `removeIf` on views) |
| Streams | `map.entrySet().stream()...` | Filtering, sorting |

---

## 2. HashMap (Overview)

### 2.1 Interview Definition

> **`HashMap` is a hash-table-based implementation of the `Map` interface that stores key–value pairs in an array of buckets, using the key's `hashCode()` to choose a bucket and `equals()` to find the exact key within the bucket. It provides O(1) average time for `get` and `put`, allows one `null` key and multiple `null` values, does not maintain any order, and is not synchronized. Its default initial capacity is 16 and its default load factor is 0.75; when the number of entries exceeds capacity × load factor, it resizes to double the capacity. Since Java 8, a bucket's linked list is converted to a balanced tree when it becomes too long, improving worst-case lookup to O(log n).**

The full internal working — hashing, buckets, collisions, resizing, treeification — is covered in [Chapter 5.6](06-Hashing-and-HashMap-Internal-Working.md).

---

## 3. LinkedHashMap

### 3.1 Interview Definition

> **`LinkedHashMap` is a subclass of `HashMap` that maintains a doubly linked list across all its entries, so it can iterate in a predictable order. By default it keeps insertion order; when created with the `accessOrder` flag set to `true`, it keeps access order, moving an entry to the end each time it is read or updated. By overriding its `removeEldestEntry()` method, it can be turned into a simple LRU (Least Recently Used) cache. It offers the same O(1) average performance as `HashMap` with slightly more memory usage.**

### 3.2 Java Example — Insertion Order and LRU Cache

```java
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;

public class LinkedHashMapDemo {

    static class LruCache<K, V> extends LinkedHashMap<K, V> {
        private final int capacity;

        LruCache(int capacity) {
            super(16, 0.75f, true);               // accessOrder = true
            this.capacity = capacity;
        }

        @Override
        protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
            return size() > capacity;             // evict least recently used when over capacity
        }
    }

    public static void main(String[] args) {
        Map<String, Integer> insertion = new LinkedHashMap<>();
        insertion.put("banana", 3);
        insertion.put("apple", 5);
        insertion.put("cherry", 7);
        insertion.put("apple", 6);                // updating does NOT change insertion position
        System.out.println("Insertion order: " + insertion);

        Map<String, Integer> hash = new HashMap<>(insertion);
        System.out.println("HashMap (no guaranteed order) has same size: " + (hash.size() == 3));

        LruCache<Integer, String> cache = new LruCache<>(3);
        cache.put(1, "page-1");
        cache.put(2, "page-2");
        cache.put(3, "page-3");
        cache.get(1);                             // 1 becomes most recently used
        cache.put(4, "page-4");                   // evicts least recently used → 2
        System.out.println("LRU cache keys : " + cache.keySet());
    }
}
```

**Expected Output:**

```text
Insertion order: {banana=3, apple=6, cherry=7}
HashMap (no guaranteed order) has same size: true
LRU cache keys : [3, 1, 4]
```

**Explanation:**
- Insertion order is kept; re-putting "apple" updates the value but keeps its original position.
- In access-order mode, each `get`/`put` moves the entry to the end. After accessing key 1, the order is 2, 3, 1. Adding 4 exceeds capacity 3, so `removeEldestEntry` removes the eldest (key 2). Final: 3, 1, 4.
- The LRU cache is a **very common interview question** — this is the shortest correct Java implementation.

---

## 4. TreeMap

### 4.1 Interview Definition

> **`TreeMap` is a `NavigableMap` implementation that stores entries sorted by key, using the keys' natural ordering or a `Comparator` provided at construction. It is implemented as a Red-Black tree, so `get`, `put`, `remove` and `containsKey` take O(log n) time. It does not allow `null` keys with natural ordering (but allows `null` values), uses `compareTo()`/`compare()` to identify equal keys, and provides navigation methods such as `firstKey()`, `lastKey()`, `floorKey()`, `ceilingKey()`, `headMap()`, `tailMap()`, `subMap()` and `descendingMap()`.**

### 4.2 Java Example

```java
import java.util.Map;
import java.util.NavigableMap;
import java.util.TreeMap;

public class TreeMapDemo {
    public static void main(String[] args) {
        // Grade boundaries: minimum marks → grade
        NavigableMap<Integer, String> grades = new TreeMap<>();
        grades.put(90, "A");
        grades.put(75, "B");
        grades.put(60, "C");
        grades.put(40, "D");
        grades.put(0, "F");
        System.out.println("Sorted map   : " + grades);

        for (int marks : new int[]{95, 75, 68, 12}) {
            System.out.println(marks + " marks -> grade " + grades.floorEntry(marks).getValue());
        }

        System.out.println("firstKey/lastKey : " + grades.firstKey() + " / " + grades.lastKey());
        System.out.println("ceilingKey(70)   : " + grades.ceilingKey(70));
        System.out.println("higherKey(90)    : " + grades.higherKey(90));
        System.out.println("headMap(60)      : " + grades.headMap(60));        // keys < 60
        System.out.println("tailMap(60)      : " + grades.tailMap(60));        // keys >= 60
        System.out.println("descendingMap    : " + grades.descendingMap());

        Map<String, Integer> population = new TreeMap<>(String.CASE_INSENSITIVE_ORDER);
        population.put("delhi", 32);
        population.put("Delhi", 33);                 // same key under this comparator → replaces
        System.out.println("Case-insensitive : " + population);
    }
}
```

**Expected Output:**

```text
Sorted map   : {0=F, 40=D, 60=C, 75=B, 90=A}
95 marks -> grade A
75 marks -> grade B
68 marks -> grade C
12 marks -> grade F
firstKey/lastKey : 0 / 90
ceilingKey(70)   : 75
higherKey(90)    : null
headMap(60)      : {0=F, 40=D}
tailMap(60)      : {60=C, 75=B, 90=A}
descendingMap    : {90=A, 75=B, 60=C, 40=D, 0=F}
Case-insensitive : {delhi=33}
```

**Explanation:** `floorEntry(marks)` finds the greatest boundary ≤ marks — a neat way to implement range lookups (grades, tax slabs, shipping charges) without `if-else` chains. With a case-insensitive comparator, "Delhi" is treated as the same key as "delhi", so the value is replaced but the original key text is kept.

---

## 5. Hashtable

### 5.1 Interview Definition

> **`Hashtable` is a legacy (Java 1.0) hash-table implementation of `Map` in which all public methods are synchronized, making it thread-safe but slow because a single lock protects the whole table. It does not allow `null` keys or `null` values, has a default capacity of 11 and load factor 0.75, and grows to `2 × capacity + 1`. It supports the legacy `Enumeration` interface. In modern code, `HashMap` is used for single-threaded access and `ConcurrentHashMap` is used for concurrent access.**

### 5.2 HashMap vs Hashtable

| Point | HashMap | Hashtable |
|---|---|---|
| Introduced | Java 1.2 | Java 1.0 (legacy) |
| Synchronized | No | Yes (every method) |
| Thread-safe | No | Yes (but coarse-grained locking) |
| Performance | Faster | Slower |
| `null` keys/values | One `null` key, many `null` values | **No** `null` key or value (`NullPointerException`) |
| Default capacity / growth | 16 / 2× | 11 / 2× + 1 |
| Iteration | Fail-fast `Iterator` | `Iterator` (fail-fast) and `Enumeration` (not fail-fast) |
| Superclass | `AbstractMap` | `Dictionary` (obsolete) |
| Treeification (Java 8) | Yes | No |
| Modern replacement for threads | `ConcurrentHashMap` | — |

```java
import java.util.HashMap;
import java.util.Hashtable;
import java.util.Map;

public class HashMapVsHashtable {
    public static void main(String[] args) {
        Map<String, String> hashMap = new HashMap<>();
        hashMap.put(null, "null key ok");
        hashMap.put("k", null);
        System.out.println("HashMap: " + hashMap.get(null) + ", value of k = " + hashMap.get("k"));

        Map<String, String> table = new Hashtable<>();
        try {
            table.put(null, "x");
        } catch (NullPointerException e) {
            System.out.println("Hashtable: null key -> NullPointerException");
        }
        try {
            table.put("k", null);
        } catch (NullPointerException e) {
            System.out.println("Hashtable: null value -> NullPointerException");
        }
    }
}
```

**Expected Output:**

```text
HashMap: null key ok, value of k = null
Hashtable: null key -> NullPointerException
Hashtable: null value -> NullPointerException
```

**Why does `HashMap` allow a `null` key?** It handles `null` specially: `hash(null)` is 0, so the `null` key is stored in bucket 0. `Hashtable` calls `key.hashCode()` directly, which fails for `null`. **Why does `ConcurrentHashMap` forbid nulls?** In concurrent code, `get(key) == null` would be ambiguous — "absent" or "mapped to null" — and you can't safely check with `containsKey()` in between because another thread may change the map.

---

## 6. Comparison of Map Implementations

| Point | HashMap | LinkedHashMap | TreeMap | Hashtable | ConcurrentHashMap |
|---|---|---|---|---|---|
| Order | None | Insertion (or access) | Sorted by key | None | None |
| get/put | O(1) avg | O(1) avg | O(log n) | O(1) avg | O(1) avg |
| `null` key | 1 allowed | 1 allowed | ❌ (natural order) | ❌ | ❌ |
| `null` values | ✅ | ✅ | ✅ | ❌ | ❌ |
| Thread-safe | ❌ | ❌ | ❌ | ✅ (global lock) | ✅ (fine-grained) |
| Structure | Array of buckets (list/tree) | HashMap + doubly linked list | Red-Black tree | Array of buckets | Array of buckets + CAS/sync per bin |
| Use when | Default choice | Predictable order, LRU cache | Sorted keys, range queries | Legacy code only | Concurrent access |

### 6.1 Interview Answer — "HashMap vs TreeMap vs LinkedHashMap"

> "All three implement Map. **HashMap** stores entries in hash buckets, gives O(1) average get and put, allows one null key, and has no ordering. **LinkedHashMap** extends HashMap and adds a doubly linked list, so it remembers insertion order — or access order, which makes it useful for an LRU cache — with the same O(1) performance and a bit more memory. **TreeMap** is a Red-Black tree that keeps keys sorted, so operations are O(log n); it doesn't allow null keys with natural ordering, and it offers navigation methods like `floorKey`, `ceilingKey`, `headMap` and `tailMap`. I use HashMap by default, LinkedHashMap when I need predictable order, and TreeMap when I need sorted keys or range lookups."

---

## 7. Interview Questions with Complete Answers

### Basic Level

**Q1. Can a Map contain duplicate keys? Duplicate values?**

No duplicate keys — putting an existing key replaces its value (and `put` returns the old value). Duplicate values are allowed.

**Q2. What does `map.get(key)` return if the key is missing?**

`null`. But `null` could also be a stored value in `HashMap`. Use `containsKey()` to distinguish, or `getOrDefault()` for a default.

**Q3. What is `Map.Entry`?**

A nested interface representing one key–value pair. `entrySet()` returns a set of these. Methods: `getKey()`, `getValue()`, `setValue()` (for mutable maps), and static comparators `Map.Entry.comparingByKey()`/`comparingByValue()`.

### Intermediate Level

**Q4. How do you sort a HashMap by values?**

```java
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.stream.Collectors;

public class SortByValue {
    public static void main(String[] args) {
        Map<String, Integer> marks = Map.of("Ravi", 72, "Asha", 95, "Dev", 60, "Bala", 88);
        Map<String, Integer> sorted = marks.entrySet().stream()
                .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
                .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue,
                        (a, b) -> a, LinkedHashMap::new));      // LinkedHashMap keeps the sorted order
        System.out.println(sorted);
    }
}
```

**Expected Output:**

```text
{Asha=95, Bala=88, Ravi=72, Dev=60}
```

A `HashMap` itself can't be sorted; we stream its entries, sort them, and collect into a `LinkedHashMap` to keep that order. To sort by **keys**, simply use `new TreeMap<>(map)`.

**Q5. What are the different ways to iterate over a Map and which is best?**

`entrySet()` (best — key and value together), `forEach((k, v) -> ...)`, `keySet()` with `get()` (extra lookups), `values()` (values only), and `Iterator` over `entrySet()` (when removing). Prefer `entrySet()` or `forEach`.

**Q6. How do you implement an LRU cache in Java?**

Extend `LinkedHashMap` with `accessOrder = true` in the constructor and override `removeEldestEntry()` to return `size() > capacity`. Each access moves the entry to the end; the eldest (least recently used) is removed automatically after an insertion exceeds capacity. In an interview, also be ready to implement it manually with a `HashMap` + doubly linked list for O(1) operations.

### Advanced Level

**Q7. Why is `TreeMap` O(log n) and `HashMap` O(1)?**

`HashMap` computes the bucket index directly from the hash code (array access), and buckets are short on average. `TreeMap` must walk down a balanced binary search tree, comparing keys at each level; the height is O(log n).

**Q8. Can we use a mutable object as a HashMap key?**

You can, but you shouldn't change the fields used in `hashCode()`/`equals()` while it is a key. Otherwise the entry stays in the bucket for its old hash and becomes unreachable. Use immutable keys (`String`, `Integer`, records, enums).

**Q9. What is `IdentityHashMap`, `WeakHashMap` and `EnumMap`?**

- `IdentityHashMap` compares keys with `==` instead of `equals()` (used in serialization/graph algorithms).
- `WeakHashMap` holds keys with weak references, so entries are removed when keys are no longer referenced elsewhere (useful for caches/metadata).
- `EnumMap` is a compact, array-based map for enum keys — very fast, keeps enum declaration order.

---

## 8. Follow-Up Questions

**F1. What does `Map.of()` do with duplicate keys or null?**

Throws `IllegalArgumentException` for duplicate keys and `NullPointerException` for null keys or values. The result is immutable with unspecified iteration order. `Map.of` supports up to 10 pairs; use `Map.ofEntries(Map.entry(k, v), ...)` for more.

**F2. Is the `keySet()` a copy?**

No, it is a **view**: removing a key from `keySet()` removes the mapping from the map. Adding to it is not supported.

**F3. How do you make a HashMap thread-safe?**

Prefer `ConcurrentHashMap`. Alternatively `Collections.synchronizedMap(map)` (single lock, and you must synchronize manually while iterating). `Hashtable` is legacy.

---

## 9. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Expecting HashMap to keep insertion order | Use LinkedHashMap. |
| `int count = map.get(key)` for a missing key | NPE from unboxing; use `getOrDefault`/`merge`. |
| Iterating `keySet()` and calling `get()` | Use `entrySet()`. |
| Removing entries in a for-each over the map | CME; use iterator `remove()` or `entrySet().removeIf()`. |
| `null` key in TreeMap/Hashtable/ConcurrentHashMap | `NullPointerException`. |
| Using Hashtable in new code | Use HashMap / ConcurrentHashMap. |

---

## 10. Quick Revision

- `Map<K,V>`: unique keys, duplicate values OK, not a `Collection`; views `keySet/values/entrySet`.
- Java 8 helpers: `getOrDefault`, `putIfAbsent`, `computeIfAbsent` (grouping), `merge` (counting), `forEach`, `replaceAll`.
- **HashMap**: buckets, O(1), one null key, no order, cap 16, LF 0.75, not synchronized.
- **LinkedHashMap**: + doubly linked list; insertion/access order; LRU via `removeEldestEntry`.
- **TreeMap**: Red-Black tree, sorted keys, O(log n), no null key; `floorKey/ceilingKey/headMap/tailMap/subMap/descendingMap`.
- **Hashtable**: legacy, synchronized, no nulls, cap 11 → `2n+1`.
- Concurrent use → **ConcurrentHashMap** (no nulls).

---

[← Previous: Queue and Deque](04-Queue-PriorityQueue-Deque.md) | [Back to README](../README.md) | [Next: HashMap Internal Working →](06-Hashing-and-HashMap-Internal-Working.md)
