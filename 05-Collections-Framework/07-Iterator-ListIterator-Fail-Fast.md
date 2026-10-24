# Chapter 5.7 — Iterator, ListIterator and Fail-Fast Behaviour

[← Previous: HashMap Internal Working](06-Hashing-and-HashMap-Internal-Working.md) | [Back to README](../README.md) | [Next: Comparable and Comparator →](08-Comparable-and-Comparator.md)

---

## What You Will Learn

- The `Iterator` interface and how the for-each loop uses it
- Removing elements safely while iterating
- **Fail-fast vs fail-safe** iterators and `ConcurrentModificationException`
- How `modCount` works internally (and a famous trick question)
- `ListIterator` — bidirectional traversal, `set`, `add`, indexes
- `Enumeration` (legacy) vs `Iterator`

---

## 1. Iterator

### 1.1 Interview Definition

> **`Iterator` is an interface in `java.util` that provides a uniform way to traverse the elements of any collection one by one, without exposing the collection's internal structure. It has three main methods: `hasNext()`, which checks whether more elements exist; `next()`, which returns the next element; and `remove()`, which removes the last element returned by `next()` from the underlying collection. Every `Collection` provides an iterator through the `iterator()` method, and the enhanced for loop internally uses an iterator. `Iterator.remove()` is the safe way to remove elements from a collection during traversal.**

### 1.2 Understand the Concept

```text
list = [A, B, C]

cursor starts BEFORE the first element:
   ↓
 | A | B | C |
hasNext() → true,  next() → A   (cursor moves after A)
hasNext() → true,  next() → B
hasNext() → true,  next() → C
hasNext() → false  → stop
```

**Rules of `remove()`:**
1. It removes the element **last returned by `next()`**.
2. It can be called **only once per `next()`**.
3. Calling it before `next()` or twice in a row → `IllegalStateException`.
4. Some collections don't support it (`List.of(...)`) → `UnsupportedOperationException`.

**The for-each loop is compiled into an iterator loop:**

<!-- skip-check -->
```java
for (Iterator<String> it = list.iterator(); it.hasNext(); ) {
    String s = it.next();
    // loop body
}
```

That is why changing the list inside a for-each loop affects a **hidden iterator**.

### 1.3 Real-World Example

An iterator is like a **TV remote's "next channel" button**. You don't need to know how the TV stores channels; you ask "is there another channel?" (`hasNext`) and "show the next one" (`next`). If you don't like the current channel, you can delete it from your favourites right there (`remove`).

### 1.4 Java Example

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class IteratorDemo {
    public static void main(String[] args) {
        List<Integer> marks = new ArrayList<>(List.of(45, 82, 33, 91, 28, 76));

        Iterator<Integer> it = marks.iterator();
        while (it.hasNext()) {
            int m = it.next();
            if (m < 40) {
                it.remove();                    // safe removal of the current element
            }
        }
        System.out.println("After removing fails: " + marks);

        marks.removeIf(m -> m > 90);            // Java 8 alternative (uses an iterator internally)
        System.out.println("After removeIf(>90) : " + marks);

        Iterator<Integer> it2 = marks.iterator();
        try {
            it2.remove();                       // remove() before next()
        } catch (IllegalStateException e) {
            System.out.println("remove() before next() -> IllegalStateException");
        }
    }
}
```

**Expected Output:**

```text
After removing fails: [45, 82, 91, 76]
After removeIf(>90) : [45, 82, 76]
remove() before next() -> IllegalStateException
```

---

## 2. Fail-Fast Iterators and `ConcurrentModificationException`

### 2.1 Interview Definition

> **A fail-fast iterator immediately throws a `ConcurrentModificationException` if the underlying collection is structurally modified — elements added or removed — after the iterator was created, by any means other than the iterator's own `remove()` or `add()` methods. It works by storing the collection's modification counter (`modCount`) in a field called `expectedModCount` when the iterator is created, and comparing the two on each call to `next()`. Iterators of `ArrayList`, `LinkedList`, `HashMap`, `HashSet` and most `java.util` collections are fail-fast. Fail-fast behaviour is a best-effort bug-detection mechanism, not a guarantee, and it is not a thread-safety feature.**

### 2.2 How It Works Internally

```text
ArrayList has: int modCount;      // incremented on every structural change (add/remove/clear)

iterator created     → expectedModCount = modCount        (say 4)
list.remove(x)       → modCount = 5   (change NOT made through the iterator)
it.next()            → checkForComodification(): 5 != 4 → throw ConcurrentModificationException

it.remove()          → removes the element AND sets expectedModCount = modCount → no exception
```

- **Structural modification** = something that changes the size (add, remove, clear). `set(index, value)` is **not** structural → no exception.
- Even though the name says "concurrent", CME happens most often in **single-threaded** code: modifying a list in a for-each loop.

### 2.3 Java Example — The Classic Mistake and Three Fixes

```java
import java.util.ArrayList;
import java.util.ConcurrentModificationException;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.Map;

public class FailFastDemo {
    public static void main(String[] args) {
        // WRONG: modifying the list inside for-each
        List<String> names = new ArrayList<>(List.of("Asha", "Bala", "Chitra", "Dev"));
        try {
            for (String n : names) {
                if (n.startsWith("B")) {
                    names.remove(n);            // structural change outside the iterator
                }
            }
        } catch (ConcurrentModificationException e) {
            System.out.println("for-each + list.remove -> ConcurrentModificationException");
        }
        System.out.println("List after the exception: " + names);

        // FIX 1: Iterator.remove()
        List<String> list1 = new ArrayList<>(List.of("Asha", "Bala", "Chitra", "Dev"));
        Iterator<String> it = list1.iterator();
        while (it.hasNext()) {
            if (it.next().startsWith("B")) it.remove();
        }
        System.out.println("Fix 1 iterator.remove : " + list1);

        // FIX 2: removeIf
        List<String> list2 = new ArrayList<>(List.of("Asha", "Bala", "Chitra", "Dev"));
        list2.removeIf(n -> n.startsWith("B"));
        System.out.println("Fix 2 removeIf        : " + list2);

        // FIX 3: loop backwards by index
        List<Integer> nums = new ArrayList<>(List.of(1, 2, 3, 4, 5, 6));
        for (int i = nums.size() - 1; i >= 0; i--) {
            if (nums.get(i) % 2 == 0) nums.remove(i);
        }
        System.out.println("Fix 3 backward loop   : " + nums);

        // set() is not a structural modification → no exception
        List<String> letters = new ArrayList<>(List.of("a", "b", "c"));
        for (int i = 0; i < letters.size(); i++) letters.set(i, letters.get(i).toUpperCase());
        System.out.println("After set()           : " + letters);

        // Maps: remove entries safely through a view
        Map<String, Integer> stock = new HashMap<>(Map.of("pen", 0, "book", 4, "bag", 0));
        stock.entrySet().removeIf(e -> e.getValue() == 0);
        System.out.println("Map after removeIf    : " + stock);
    }
}
```

**Expected Output:**

```text
for-each + list.remove -> ConcurrentModificationException
List after the exception: [Asha, Chitra, Dev]
Fix 1 iterator.remove : [Asha, Chitra, Dev]
Fix 2 removeIf        : [Asha, Chitra, Dev]
Fix 3 backward loop   : [1, 3, 5]
After set()           : [A, B, C]
Map after removeIf    : {book=4}
```

### 2.4 Code Explanation

- In the wrong version, `names.remove("Bala")` **does** remove "Bala" and increments `modCount`. On the **next** iteration, the hidden iterator's `next()` detects `modCount != expectedModCount` and throws CME. That is why the list printed after the exception no longer contains "Bala" — but the loop could not continue to check the remaining elements.
- **Fix 1:** `it.remove()` removes through the iterator and keeps `expectedModCount` in sync.
- **Fix 2:** `removeIf` is the shortest and clearest solution (Java 8).
- **Fix 3:** looping **backwards** with indexes is safe because removing index `i` only shifts elements **after** `i`, which have already been visited. (Looping forwards with `remove(i)` would skip the element that shifts into position `i`.)
- `set()` doesn't change the size → not structural → no CME.
- For maps, use `entrySet().removeIf(...)`, `keySet().removeIf(...)`, or `values().removeIf(...)`, or an explicit iterator.

### 2.5 The Famous Trick: Removing the Second-Last Element

```java
import java.util.ArrayList;
import java.util.List;

public class SecondLastTrick {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>(List.of("A", "B", "C"));
        for (String s : list) {
            if (s.equals("B")) {
                list.remove(s);                 // removing the SECOND-LAST element
            }
        }
        System.out.println("No exception! List = " + list);
    }
}
```

**Expected Output:**

```text
No exception! List = [A, C]
```

**Why no exception?** `ArrayList`'s iterator checks for modification in `next()`, but `hasNext()` only checks `cursor != size`. After visiting "B", the cursor is 2. Removing "B" makes the size 2. Now `hasNext()` returns `false` (2 == 2), so the loop ends **without calling `next()`** — the check never happens, and "C" is never visited. This shows that fail-fast behaviour is **best-effort**, not guaranteed, and the code is still wrong (it silently skipped "C").

### 2.6 Fail-Fast vs Fail-Safe

> **Fail-safe (more accurately, weakly consistent or snapshot) iterators do not throw `ConcurrentModificationException` when the collection is modified during iteration. They either work on a copy (snapshot) of the data, like `CopyOnWriteArrayList`, or traverse the live structure with weakly consistent guarantees, like `ConcurrentHashMap`. They may not reflect modifications made after the iterator was created.**

| Point | Fail-fast | Fail-safe / weakly consistent |
|---|---|---|
| On concurrent structural modification | Throws `ConcurrentModificationException` | No exception |
| Works on | Original collection | Snapshot copy (`CopyOnWriteArrayList`) or live structure with weak guarantees (`ConcurrentHashMap`) |
| Sees changes made during iteration? | N/A (fails) | May or may not (snapshot: never) |
| Memory | No extra | Snapshot copies cost memory |
| Examples | `ArrayList`, `LinkedList`, `HashMap`, `HashSet`, `TreeMap` iterators | `CopyOnWriteArrayList`, `CopyOnWriteArraySet`, `ConcurrentHashMap`, `ConcurrentLinkedQueue` |
| Package | `java.util` | `java.util.concurrent` |

```java
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.CopyOnWriteArrayList;

public class FailSafeDemo {
    public static void main(String[] args) {
        List<String> cow = new CopyOnWriteArrayList<>(List.of("A", "B", "C"));
        for (String s : cow) {                     // iterates over a SNAPSHOT
            if (s.equals("A")) cow.add("D");       // no exception
        }
        System.out.println("CopyOnWriteArrayList: " + cow);

        Map<String, Integer> chm = new ConcurrentHashMap<>(Map.of("x", 1, "y", 2));
        for (String key : chm.keySet()) {
            chm.remove("y");                       // no exception (weakly consistent)
        }
        System.out.println("ConcurrentHashMap size: " + chm.size());
    }
}
```

**Expected Output:**

```text
CopyOnWriteArrayList: [A, B, C, D]
ConcurrentHashMap size: 1
```

The `CopyOnWriteArrayList` loop iterated over the original 3 elements; "D" was added to a new internal array, which the current iterator doesn't see.

### 2.7 Interview Answer — "Fail-fast vs fail-safe iterators"

> "A fail-fast iterator throws `ConcurrentModificationException` if the collection is structurally modified after the iterator is created, except through the iterator's own `remove` method. It works using a `modCount` counter: the iterator stores the expected count and checks it on every `next()` call. ArrayList, HashMap and most `java.util` collections have fail-fast iterators. A common example is removing from a list inside a for-each loop — the fix is to use `iterator.remove()` or `removeIf()`. Fail-safe iterators, found in `java.util.concurrent` classes, don't throw this exception: `CopyOnWriteArrayList` iterates over a snapshot copy, and `ConcurrentHashMap` uses weakly consistent iterators that may or may not show changes. One more point: fail-fast is best-effort, not guaranteed, so it shouldn't be used for program logic — only for detecting bugs."

---

## 3. ListIterator

### 3.1 Interview Definition

> **`ListIterator` is a sub-interface of `Iterator` available only for `List` implementations. In addition to forward traversal, it supports backward traversal using `hasPrevious()` and `previous()`, gives the position of the cursor through `nextIndex()` and `previousIndex()`, and can modify the list during iteration with `set()` to replace the last returned element and `add()` to insert a new element at the cursor position. It can be obtained using `list.listIterator()` or `list.listIterator(index)` to start from a specific position.**

### 3.2 Java Example

```java
import java.util.ArrayList;
import java.util.List;
import java.util.ListIterator;

public class ListIteratorDemo {
    public static void main(String[] args) {
        List<String> steps = new ArrayList<>(List.of("design", "code", "deploy"));

        ListIterator<String> it = steps.listIterator();
        while (it.hasNext()) {
            String step = it.next();
            if (step.equals("code")) {
                it.add("test");                     // inserted after "code"
            }
            it.set(step.toUpperCase());             // WRONG ORDER: set() right after add()
        }
        System.out.println(steps);
    }
}
```

**Output** (the first element is processed; the program fails while processing "code"):

```text
Exception in thread "main" java.lang.IllegalStateException
```

**Explanation:** `set()` replaces the element **last returned by `next()` or `previous()`**, and it is **not allowed after `add()` or `remove()`** until `next()`/`previous()` is called again. Calling `set()` right after `add()` throws `IllegalStateException`. The correct version calls `set()` **before** `add()`:

```java
import java.util.ArrayList;
import java.util.List;
import java.util.ListIterator;

public class ListIteratorFixed {
    public static void main(String[] args) {
        List<String> steps = new ArrayList<>(List.of("design", "code", "deploy"));

        ListIterator<String> it = steps.listIterator();
        while (it.hasNext()) {
            System.out.print("index " + it.nextIndex() + " ");
            String step = it.next();
            it.set(step.toUpperCase());             // replace current element
            if (step.equals("code")) {
                it.add("TEST");                     // insert after current element
            }
        }
        System.out.println();
        System.out.println("Forward result : " + steps);

        System.out.print("Backward       : ");
        while (it.hasPrevious()) {                  // iterator is now at the end
            System.out.print(it.previous() + " ");
        }
        System.out.println();

        ListIterator<String> fromMiddle = steps.listIterator(2);
        System.out.println("Start at index 2: " + fromMiddle.next());
    }
}
```

**Expected Output:**

```text
index 0 index 1 index 3 
Forward result : [DESIGN, CODE, TEST, DEPLOY]
Backward       : DEPLOY TEST CODE DESIGN 
Start at index 2: TEST
```

**Explanation:**
- `nextIndex()` shows the cursor position; after `add("TEST")` at index 2, the next index jumps to 3.
- `set()` replaces the current element; `add()` inserts at the cursor (after the current element), and the new element is **not** visited by the ongoing forward traversal.
- After the forward loop, the same iterator can walk **backwards** with `hasPrevious()`/`previous()`.
- `listIterator(2)` starts with the cursor before index 2.

### 3.3 Iterator vs ListIterator

| Point | Iterator | ListIterator |
|---|---|---|
| Available for | All `Collection`s | Only `List`s |
| Direction | Forward only | Forward and backward |
| Methods | `hasNext`, `next`, `remove` | + `hasPrevious`, `previous`, `nextIndex`, `previousIndex`, `set`, `add` |
| Modify element | No | `set()` |
| Add element | No | `add()` |
| Get index | No | Yes |
| Start position | Beginning | Any index (`listIterator(i)`) |

---

## 4. Enumeration (Legacy)

`Enumeration` is the pre-Java 1.2 way to traverse `Vector` and `Hashtable`.

| Point | Enumeration | Iterator |
|---|---|---|
| Introduced | Java 1.0 | Java 1.2 |
| Methods | `hasMoreElements()`, `nextElement()` | `hasNext()`, `next()`, `remove()` |
| Remove support | No | Yes |
| Fail-fast | No (for `Vector.elements()`) | Yes |
| Applies to | Legacy classes (`Vector`, `Hashtable`) | All collections |
| Conversion | `enumeration.asIterator()` (Java 9) | `Collections.enumeration(collection)` |

---

## 5. Interview Questions with Complete Answers

### Basic Level

**Q1. What is the difference between Iterator and for-each loop?**

The for-each loop is syntactic sugar that internally uses an `Iterator` (for collections) — it is shorter and cleaner, but it doesn't expose the iterator, so you cannot call `remove()`. Use an explicit `Iterator` when you need to remove elements during traversal; otherwise for-each is preferred.

**Q2. Why do we get `ConcurrentModificationException` in a single-threaded program?**

Because the collection was structurally modified (added/removed) directly while a fail-fast iterator was in use — typically `list.remove(x)` inside a for-each loop. The iterator detects that `modCount` changed unexpectedly and throws the exception on the next `next()` call.

**Q3. How do you remove elements from a list while iterating?**

Use `Iterator.remove()`, `removeIf()` (Java 8), a backwards index loop, or collect items to remove in a separate list and call `removeAll()` after the loop.

### Intermediate Level

**Q4. What is `modCount`?**

A field in `AbstractList` (and in `HashMap`) that counts structural modifications. Iterators save its value as `expectedModCount` and compare on each `next()`. A mismatch means someone modified the collection outside the iterator → `ConcurrentModificationException`.

**Q5. Is `ConcurrentModificationException` guaranteed to be thrown?**

No. Fail-fast behaviour is best-effort. For example, removing the **second-last** element of an `ArrayList` inside a for-each loop ends the loop without an exception, because `hasNext()` returns false before `next()` can check `modCount`. Never rely on CME for correctness.

**Q6. What are fail-safe iterators? Give examples.**

Iterators that don't throw CME when the collection is modified during iteration: `CopyOnWriteArrayList`/`CopyOnWriteArraySet` (snapshot iterators) and `ConcurrentHashMap`, `ConcurrentLinkedQueue`, `ConcurrentSkipListMap` (weakly consistent iterators).

### Advanced Level

**Q7. Why can't `ListIterator.set()` be called after `add()`?**

`set()` replaces the element last returned by `next()` or `previous()`. After `add()` (or `remove()`), there is no valid "last returned element" anymore — the iterator's internal `lastRet` is reset to −1 — so calling `set()` throws `IllegalStateException` until `next()` or `previous()` is called again.

**Q8. How does a `CopyOnWriteArrayList` iterator avoid CME?**

Every write (`add`, `set`, `remove`) creates a **new copy** of the internal array. An iterator holds a reference to the array that existed when it was created, so it never sees concurrent changes. This makes reads lock-free and very fast but writes expensive — ideal for read-mostly data like listener lists.

**Q9. Can you modify a `HashMap` while iterating over `keySet()`?**

Not directly with `map.remove(key)` or `map.put(newKey, ...)` (structural) — that causes CME. Use the iterator's `remove()` on the key set or entry set, or `removeIf()` on the view. Updating the value of an existing key (`map.put(existingKey, newValue)` or `entry.setValue()`) is not structural and is allowed.

---

## 6. Follow-Up Questions

**F1. What is the `Spliterator`?**

A Java 8 iterator designed for **parallel processing**: it can split a source into parts (`trySplit()`) so streams can process them on multiple threads. Every `Collection` provides one through `spliterator()`.

**F2. Does `Iterator.forEachRemaining()` exist?**

Yes (Java 8): `it.forEachRemaining(action)` performs the action on all remaining elements.

**F3. Why does a forwards `for (int i...)` loop with `list.remove(i)` give wrong results?**

After removing index `i`, the next element shifts into index `i`, but the loop increments `i` and skips it. Loop backwards, or decrement `i` after removal, or use `removeIf`.

---

## 7. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| `list.remove()` inside for-each | CME; use iterator/removeIf. |
| Calling `it.remove()` twice per `next()` | `IllegalStateException`. |
| Thinking CME only happens with threads | Usually single-threaded misuse. |
| Relying on CME being thrown | Best-effort only. |
| Calling `ListIterator.set()` after `add()` | `IllegalStateException`. |
| Forward index loop with removal | Skips elements. |

---

## 8. Quick Revision

- `Iterator`: `hasNext`, `next`, `remove` (once per `next`); for-each uses it internally.
- **Fail-fast**: `modCount` vs `expectedModCount` → CME on structural change outside the iterator; best-effort (second-last trick).
- Safe removal: `it.remove()`, `removeIf()`, backward loop, `entrySet().removeIf()` for maps.
- **Fail-safe / weakly consistent**: `CopyOnWriteArrayList` (snapshot), `ConcurrentHashMap` — no CME.
- `ListIterator`: both directions, `nextIndex/previousIndex`, `set` (not after `add/remove`), `add`; lists only.
- `Enumeration`: legacy, no remove.

---

[← Previous: HashMap Internal Working](06-Hashing-and-HashMap-Internal-Working.md) | [Back to README](../README.md) | [Next: Comparable and Comparator →](08-Comparable-and-Comparator.md)
