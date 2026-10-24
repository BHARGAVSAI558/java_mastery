# Chapter 5.6 — Hashing, the equals/hashCode Contract and HashMap Internal Working

[← Previous: Map](05-Map-HashMap-LinkedHashMap-TreeMap-Hashtable.md) | [Back to README](../README.md) | [Next: Iterator and Fail-Fast →](07-Iterator-ListIterator-Fail-Fast.md)

---

## What You Will Learn

- What hashing is and why it gives O(1) lookups
- The equals–hashCode contract (recap) and why it matters for `HashMap`
- `HashMap` internal structure: table, buckets, `Node`
- How `hash()` and the bucket index are calculated
- Step-by-step `put()` and `get()`
- Collisions, linked lists and **treeification** (Java 8)
- Load factor, threshold, resizing and rehashing
- Java 7 vs Java 8 `HashMap`
- The most asked interview question: **"How does HashMap work internally?"**

---

## 1. Hashing

### 1.1 Interview Definition

> **Hashing is a technique of converting a key into an integer value, called a hash code, using a hash function, and then using that value to compute an index in an array where the data is stored. Because the index can be computed directly from the key, insertion and lookup take constant time on average, without searching through all elements. When two different keys produce the same index, it is called a collision, and the data structure must handle it, for example by storing multiple entries in the same bucket.**

### 1.2 Understand the Concept

Without hashing, finding a key in an unsorted list means checking every element — **O(n)**. With hashing:

```text
key "apple" ──hashCode()──► 93029210 ──(index formula)──► bucket 10 ──► look only in bucket 10
```

A good hash function:
1. Is **deterministic** — the same key always gives the same hash.
2. **Spreads keys evenly** across buckets — few collisions.
3. Is **fast** to compute.

### 1.3 Real-World Example

A **college library** puts books on shelves by the **first two digits of the book ID** (hash function). To find book 4521, you go directly to shelf 45 (bucket) and look only at the few books there, instead of searching the entire library.

---

## 2. The equals–hashCode Contract (Recap)

| Rule | Meaning for HashMap |
|---|---|
| `a.equals(b)` ⇒ `a.hashCode() == b.hashCode()` | Equal keys go to the **same bucket**, so lookups find them |
| Same hash code does **not** imply equal | Collisions are allowed; `equals()` resolves them inside the bucket |
| `hashCode()` must not change while the key is in the map | Otherwise the entry becomes unreachable |

If `equals()` is overridden without `hashCode()`, equal keys land in different buckets → duplicates in `HashSet`, `null` from `get()`. (Full explanation: [Chapter 3.10](../03-OOP/10-Object-Class-equals-hashCode-toString.md).)

---

## 3. HashMap Internal Structure

### 3.1 Interview Definition

> **Internally, a `HashMap` is an array of buckets called `table`, where each bucket can hold a chain of nodes. Each node (`Node<K,V>`) stores the key's hash, the key, the value, and a reference to the next node in the same bucket. When a key is inserted, `HashMap` computes a hash from the key's `hashCode()`, spreads its bits, and calculates the bucket index as `(n - 1) & hash`, where `n` is the table length (always a power of two). If the bucket is empty, a new node is placed there; if not, the existing nodes are compared using the hash and `equals()` — a matching key has its value replaced, otherwise the new node is appended. Since Java 8, if a bucket's chain grows beyond 8 nodes and the table has at least 64 buckets, the chain is converted into a red-black tree. When the number of entries exceeds capacity × load factor (0.75 by default), the table is doubled in size and entries are redistributed.**

### 3.2 Diagram

```text
HashMap<String, Integer>   table (length n = 16)
┌─────┐
│  0  │ → null
├─────┤
│  1  │ → [hash|"Aa"|10|next] → [hash|"BB"|20|null]     ← collision: same bucket, linked list
├─────┤
│  2  │ → null
├─────┤
│  3  │ → [hash|"cat"|5|null]
├─────┤
│ ... │
├─────┤
│ 15  │ → TreeNode (red-black tree) if the chain became long (>8) and n >= 64
└─────┘

static class Node<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
}
```

### 3.3 Important Constants

| Constant | Value | Meaning |
|---|---|---|
| `DEFAULT_INITIAL_CAPACITY` | **16** | Initial number of buckets (table created lazily on first `put`) |
| `MAXIMUM_CAPACITY` | 2³⁰ | Upper limit of buckets |
| `DEFAULT_LOAD_FACTOR` | **0.75** | Resize when size > capacity × 0.75 |
| `TREEIFY_THRESHOLD` | **8** | Chain length at which a bucket may become a tree |
| `UNTREEIFY_THRESHOLD` | **6** | Tree converted back to a list when it shrinks to this size (during resize) |
| `MIN_TREEIFY_CAPACITY` | **64** | Minimum table size before treeifying; below this, the table is resized instead |

---

## 4. Step 1 — Computing the Hash and Index

```java
// From java.util.HashMap (Java 8+)
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

// index inside putVal / getNode
int index = (n - 1) & hash;     // n = table.length, always a power of two
```

**Why `h ^ (h >>> 16)`?** The index uses only the **lower bits** of the hash (because `n − 1` is like `0b1111` for n = 16). Many hash codes differ mainly in their **higher bits**. XOR-ing the high 16 bits into the low 16 bits mixes them, so more bits influence the bucket → fewer collisions. This is called **hash spreading** or the **perturbation function**.

**Why `(n − 1) & hash` instead of `hash % n`?** When `n` is a power of two, `hash & (n − 1)` gives the same result as `hash mod n` for non-negative values, but it is a single fast bitwise operation, and it also works for negative hash codes. That is why **capacity is always a power of two** (if you pass 20, it becomes 32).

**`null` key:** `hash(null)` is 0 → always stored in **bucket 0**.

### 4.1 Java Example — Calculating Bucket Indexes Ourselves

```java
public class BucketIndexDemo {
    static int spread(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);   // same as HashMap.hash()
    }

    static int bucketIndex(Object key, int capacity) {
        return (capacity - 1) & spread(key);
    }

    public static void main(String[] args) {
        String[] keys = {"apple", "banana", "cherry", "Aa", "BB", null};
        for (String k : keys) {
            System.out.printf("%-7s hashCode=%-11s index(16)=%-3d index(32)=%d%n",
                    k, (k == null ? "-" : String.valueOf(k.hashCode())), bucketIndex(k, 16), bucketIndex(k, 32));
        }
        System.out.println("tableSizeFor(20) -> " + Integer.highestOneBit(20 - 1) * 2);
    }
}
```

**Expected Output:**

```text
apple   hashCode=93029210    index(16)=1   index(32)=17
banana  hashCode=-1396355227 index(16)=0   index(32)=0
cherry  hashCode=-1361513063 index(16)=1   index(32)=1
Aa      hashCode=2112        index(16)=0   index(32)=0
BB      hashCode=2112        index(16)=0   index(32)=0
null    hashCode=-           index(16)=0   index(32)=0
tableSizeFor(20) -> 32
```

**Explanation:**
- `"Aa"` and `"BB"` have the **same hash code** (2112), so they always share a bucket → a **collision**, resolved by `equals()`.
- `"apple"` and `"cherry"` have **different** hash codes but still land in the same bucket (1) when the capacity is 16 — another kind of collision. When the table grows to 32, `"apple"` moves to bucket 17 while `"cherry"` stays in bucket 1, so the collision disappears.
- Negative hash codes (like `"banana"`) work fine because `&` with `n − 1` always gives a non-negative index.
- `null` always maps to bucket 0.
- A requested capacity of 20 is rounded up to the next power of two, **32**.

---

## 5. Step 2 — How `put(key, value)` Works

```text
put(K key, V value)
1. hash = hash(key)                        (hashCode + spreading; null → 0)
2. if table is null/empty → resize()       (lazy creation, capacity 16)
3. index = (n - 1) & hash
4. if table[index] is empty
       → create new Node and place it there
   else (collision or same key)
       a. if first node has same hash AND (same reference OR equals) → existing key → REPLACE value
       b. else if the bucket is a TreeNode → putTreeVal() (search/insert in red-black tree)
       c. else walk the linked list:
            - if a node with equal key is found → REPLACE value
            - if end reached → append new Node at the TAIL (Java 8)
            - if chain length reaches TREEIFY_THRESHOLD (8) → treeifyBin()
                 (treeify only if table length >= 64, otherwise resize())
5. if a value was replaced → return old value (size unchanged, modCount unchanged)
6. else ++modCount; if (++size > threshold) → resize()
7. return null
```

**Key comparison inside a bucket:** `p.hash == hash && (p.key == key || (key != null && key.equals(p.key)))`. Checking the `int` hash first is cheap and avoids most `equals()` calls.

## 6. Step 3 — How `get(key)` Works

```text
get(Object key)
1. hash = hash(key); index = (n - 1) & hash
2. first = table[index]; if null → return null
3. if first matches (hash and equals) → return first.value
4. if first is a TreeNode → search the red-black tree (O(log k))
5. else walk the list comparing hash + equals → return value when found
6. return null if not found
```

---

## 7. Collisions and Treeification (Java 8)

### 7.1 Understand the Concept

- A **collision** happens when two keys map to the same bucket — either because their hash codes are equal (`"Aa"`, `"BB"`) or because different hash codes give the same index after masking.
- Java uses **separate chaining**: the bucket holds a linked list of nodes.
- In the worst case (bad `hashCode()` or an attack with many colliding keys), a bucket's list becomes long and lookups degrade to **O(n)**.
- **Java 8 improvement:** when a bucket's list reaches **8** nodes (and the table has at least **64** buckets), it is converted into a **red-black tree** (`TreeNode`), so lookups in that bucket become **O(log n)**. If the table is smaller than 64, HashMap **resizes** instead (spreading the nodes out is usually enough).
- When a tree bin shrinks to **6** or fewer nodes (during a resize split), it is converted back to a list.
- Tree bins order nodes by hash, and if keys implement `Comparable`, by `compareTo()` — so keys that are `Comparable` benefit most.

### 7.2 Java Example — A Badly Hashed Key

```java
import java.util.HashMap;
import java.util.Map;

public class CollisionDemo {
    static final class BadKey implements Comparable<BadKey> {
        final int id;
        BadKey(int id) { this.id = id; }
        @Override public int hashCode() { return 42; }                 // every key collides!
        @Override public boolean equals(Object o) { return o instanceof BadKey b && b.id == id; }
        @Override public int compareTo(BadKey o) { return Integer.compare(id, o.id); }
    }

    public static void main(String[] args) {
        Map<BadKey, String> map = new HashMap<>();
        for (int i = 0; i < 1000; i++) {
            map.put(new BadKey(i), "value-" + i);
        }
        System.out.println("size = " + map.size());
        System.out.println("get(777) = " + map.get(new BadKey(777)));
        System.out.println("All 1000 keys share one bucket; Java 8 treeifies it, so lookups stay O(log n).");
    }
}
```

**Expected Output:**

```text
size = 1000
get(777) = value-777
All 1000 keys share one bucket; Java 8 treeifies it, so lookups stay O(log n).
```

Correctness is preserved (thanks to `equals()`), but performance depends on a good `hashCode()`. Because `BadKey` is `Comparable`, the tree bin can order keys efficiently.

---

## 8. Load Factor, Threshold, Resizing and Rehashing

### 8.1 Interview Definition

> **The load factor is a measure of how full the `HashMap` is allowed to get before its capacity is increased. The threshold is capacity × load factor; with the defaults (16 × 0.75), the threshold is 12. When the number of entries exceeds the threshold, the `HashMap` resizes: it creates a new table with double the capacity and redistributes all existing entries into the new buckets. In Java 8, each entry either stays at its original index or moves to "original index + old capacity", determined by a single bit of its hash, so the hash does not need to be recomputed.**

### 8.2 Understand the Concept

| Capacity | Threshold (× 0.75) | Resize happens when size becomes |
|---|---|---|
| 16 | 12 | 13 |
| 32 | 24 | 25 |
| 64 | 48 | 49 |

**Why 0.75?** It is a trade-off between **time** and **space**: a lower load factor means fewer collisions but more empty buckets (memory waste); a higher one saves memory but increases collisions. 0.75 gives good performance in practice (statistically, with a good hash, bucket lengths stay very short).

**Java 8 resize split:** with capacity doubling from `n` to `2n`, the new index uses one extra bit of the hash. If `(hash & oldCap) == 0`, the entry stays at index `i` (**lo list**); otherwise it moves to `i + oldCap` (**hi list**). The relative order of nodes is preserved.

**Tip:** If you know you'll store about N entries, create the map with capacity `N / 0.75 + 1` (or use `HashMap.newHashMap(N)` in **Java 19+**) to avoid repeated resizing.

### 8.3 Java Example — Watching the Resize Split Rule

```java
public class ResizeSplitDemo {
    public static void main(String[] args) {
        int oldCap = 16;
        int[] hashes = {5, 21, 37, 53};                 // all map to index 5 when capacity is 16
        for (int h : hashes) {
            int oldIndex = h & (oldCap - 1);
            int newIndex = h & (2 * oldCap - 1);
            String rule = (h & oldCap) == 0 ? "stays (lo)" : "moves to old+16 (hi)";
            System.out.println("hash " + h + ": index " + oldIndex + " -> " + newIndex + "  " + rule);
        }
    }
}
```

**Expected Output:**

```text
hash 5: index 5 -> 5  stays (lo)
hash 21: index 5 -> 21  moves to old+16 (hi)
hash 37: index 5 -> 5  stays (lo)
hash 53: index 5 -> 21  moves to old+16 (hi)
```

All four keys shared bucket 5 in a 16-slot table. After doubling to 32, half stay at 5 and half move to 21 — collisions are reduced without recalculating `hashCode()`.

---

## 9. Java 7 vs Java 8 HashMap

| Point | Java 7 | Java 8+ |
|---|---|---|
| Bucket structure | Linked list only | Linked list → **red-black tree** when long (≥ 8, table ≥ 64) |
| Worst-case lookup | O(n) | **O(log n)** |
| Insertion into list | At the **head** | At the **tail** |
| Resize with concurrent misuse | Could create an **infinite loop** (circular list) when used by multiple threads | Tail insertion avoids the loop (but HashMap is still **not thread-safe**) |
| Hash function | Several shifts and XORs | Simpler `h ^ (h >>> 16)` |
| Rehash on resize | Recompute index for each entry | Split using `hash & oldCap` (lo/hi lists) |
| Node class | `Entry` | `Node` and `TreeNode` |

---

## 10. Interview Answer — "How does HashMap work internally?"

> "HashMap stores data in an array of buckets called the table, and each bucket holds nodes containing the hash, key, value and a next pointer. When I call `put(key, value)`, HashMap first computes the key's hash by taking `hashCode()` and XOR-ing the high 16 bits into the low 16 bits, to spread the bits better. Then it finds the bucket index using `(n - 1) & hash`, where n is the table size, which is always a power of two. If the bucket is empty, the new node is stored there. If not, it compares the existing nodes using the hash and `equals()`: if it finds the same key, it replaces the value; otherwise it adds the new node at the end of the list. That situation is a collision. Since Java 8, if a bucket gets more than 8 nodes and the table has at least 64 buckets, the list is converted into a red-black tree, so worst-case lookup becomes O(log n) instead of O(n). `get()` follows the same steps — compute hash, go to the bucket, compare with equals — so on average both operations are O(1). The default capacity is 16 and the load factor is 0.75, so when the size crosses 12, the table doubles and entries are redistributed. A null key is allowed and always goes to bucket 0. And that's why keys must correctly override both `equals()` and `hashCode()`, and should ideally be immutable."

---

## 11. Interview Questions with Complete Answers

### Basic Level

**Q1. What is the default capacity and load factor of HashMap?**

Capacity 16 and load factor 0.75, so the first resize happens when the map holds more than 12 entries. The table itself is created lazily on the first `put()`.

**Q2. Where is the `null` key stored?**

In bucket 0, because `hash(null)` returns 0. Only one `null` key can exist; putting another replaces its value.

**Q3. What is a collision? How does HashMap handle it?**

A collision happens when two keys map to the same bucket index. HashMap handles it with **separate chaining**: the bucket stores a linked list of nodes (and a red-black tree if the list becomes long). `equals()` identifies the correct key inside the bucket.

### Intermediate Level

**Q4. Why is the HashMap capacity always a power of two?**

So that the bucket index can be computed with the fast bitwise expression `(n − 1) & hash` instead of the slower modulus, and so that resizing can split each bucket into two using one bit (`hash & oldCap`). Any requested capacity is rounded up to the next power of two.

**Q5. What happens when two keys have the same hashCode but are not equal?**

They go to the same bucket. HashMap stores both as separate nodes (in a list or tree) and uses `equals()` to distinguish them. Both can be retrieved correctly. Performance decreases only if many keys collide.

**Q6. What happens when two keys are equal (equals returns true)?**

They must have the same hash code (contract), so they map to the same bucket. HashMap finds the existing node and **replaces its value**; the key object stored remains the original one. `put` returns the old value.

**Q7. What is treeification and when does it happen?**

Converting a bucket's linked list into a red-black tree. It happens when a bucket's chain reaches 8 nodes (TREEIFY_THRESHOLD) **and** the table capacity is at least 64 (MIN_TREEIFY_CAPACITY). Otherwise the table is resized. It improves worst-case lookups from O(n) to O(log n).

### Advanced Level

**Q8. Why is HashMap not thread-safe? What can go wrong?**

Its operations are not atomic: two threads putting at the same time can overwrite each other's nodes (lost updates), size can become inconsistent, and in Java 7 concurrent resizing could create a circular linked list causing an infinite loop in `get()`. Use `ConcurrentHashMap` for concurrent access.

**Q9. Why does `hash()` XOR the higher 16 bits?**

The index uses only the low bits (`n − 1` masks them). Hash codes that differ only in high bits (common for some objects, like floats or similar strings) would collide. Mixing high bits into low bits spreads entries more evenly at almost no cost.

**Q10. What is the time complexity of HashMap operations?**

Average O(1) for `put`, `get`, `remove`, `containsKey`. Worst case O(log n) in Java 8+ (tree bins), O(n) in Java 7 (long lists). `containsValue` is O(n). Iteration is O(capacity + size). Resizing is O(n) but happens rarely, so `put` is amortised O(1).

**Q11. If a key's hashCode changes after insertion, what happens?**

The entry remains in the bucket computed from the **old** hash. `get()` with the same key object computes the **new** hash and searches a different bucket, so it returns `null` (unless by chance the index is the same). The entry is effectively lost until the key changes back, causing memory leaks and bugs.

---

## 12. Follow-Up Questions

**F1. Can we use a custom class as a key without overriding equals and hashCode?**

Yes, but then key equality is **identity**: only the exact same object can retrieve the value. A new object with the same data won't be found.

**F2. What is the difference between capacity and size?**

Capacity is the number of buckets (table length); size is the number of key–value mappings stored.

**F3. How does HashSet relate to this?**

`HashSet` is backed by a `HashMap` (elements are keys, value is a dummy `PRESENT` object), so everything in this chapter applies to `HashSet` as well.

**F4. Is ConcurrentHashMap's internal structure similar?**

Yes, it also uses an array of bins with lists/trees, but it uses CAS operations and synchronizes on individual bins (Java 8+) instead of locking the whole map, and it doesn't allow `null` keys or values. (Java 7 used segments.)

---

## 13. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| "HashMap uses `hashCode() % capacity`." | It uses `(n − 1) & hash` after spreading. |
| "Treeification happens at 8 entries in the map." | At 8 nodes **in one bucket**, and only if capacity ≥ 64. |
| "Resize happens at size 16." | At size > threshold (12 for capacity 16). |
| "Equal hash codes mean equal keys." | Collisions are allowed; `equals()` decides. |
| Using mutable keys | Lookups break after mutation. |
| Using HashMap from multiple threads | Use ConcurrentHashMap. |

---

## 14. Quick Revision

- Hashing: key → hashCode → spread → index → bucket → O(1) average.
- `hash = h ^ (h >>> 16)`; `index = (n − 1) & hash`; `null` → bucket 0.
- Node: `hash, key, value, next`; bucket = list or red-black tree.
- put: same key → replace value; new key → append at tail; size > threshold → resize ×2.
- Defaults: capacity **16**, load factor **0.75**, threshold **12**; capacity always power of 2.
- Treeify at **8** (if capacity ≥ **64**), untreeify at **6**.
- Resize split: `(hash & oldCap) == 0` → same index, else index + oldCap.
- Java 8: trees, tail insertion, simpler hash. Not thread-safe → `ConcurrentHashMap`.

---

[← Previous: Map](05-Map-HashMap-LinkedHashMap-TreeMap-Hashtable.md) | [Back to README](../README.md) | [Next: Iterator and Fail-Fast →](07-Iterator-ListIterator-Fail-Fast.md)
