# Chapter 2.3 — StringBuilder and StringBuffer

[← Previous: String Comparison and Methods](02-String-Comparison-and-Methods.md) | [Back to README](../README.md) | [Next: Arrays →](04-Arrays-and-Multidimensional-Arrays.md)

---

## What You Will Learn

- Why we need mutable strings
- `StringBuilder` and `StringBuffer` — what they are and how they work internally
- Capacity and automatic growth
- Important methods: `append`, `insert`, `delete`, `reverse`, `replace`, `setCharAt`
- **String vs StringBuilder vs StringBuffer** — the most asked comparison
- Performance demonstration and how `+` is compiled

---

## 1. The Problem with String in Loops

Because `String` is immutable, every change creates a **new object** and copies all old characters into it:

<!-- skip-check -->
```java
String s = "";
for (int i = 0; i < 5; i++) {
    s = s + i;     // creates "0", "01", "012", "0123", "01234" → 5 new objects
}
```

For *n* iterations, the total characters copied grow like 1 + 2 + 3 + … + n ≈ **n²/2**. For large *n*, this is very slow and creates a lot of garbage. We need a **mutable** sequence of characters that can grow in place. That is what `StringBuilder` and `StringBuffer` provide.

---

## 2. StringBuilder

### 2.1 Interview Definition

> **`StringBuilder` is a mutable sequence of characters in the `java.lang` package, introduced in Java 5. Unlike `String`, its content can be changed after creation using methods such as `append()`, `insert()`, `delete()`, `replace()` and `reverse()`, without creating a new object for every change. Internally it uses a resizable character array that grows automatically when its capacity is exceeded. `StringBuilder` is not synchronized, so it is not thread-safe, but this makes it faster than `StringBuffer`. It is the preferred choice for building or modifying strings in a single thread, such as in loops.**

### 2.2 Understand the Concept

- `StringBuilder` and `StringBuffer` both extend `AbstractStringBuilder`, which holds:
  - a `byte[] value` (since Java 9; `char[]` before) — the character storage
  - an `int count` — how many characters are actually used
- **Capacity** = size of the internal array. **Length** = number of characters currently stored.
- Default capacity is **16**. `new StringBuilder("Java")` has capacity 16 + 4 = **20**.
- When you append beyond the capacity, the array grows to **(old capacity × 2) + 2** (or more if needed), and the old content is copied **once**. This makes appends **amortised O(1)**.
- Most methods return `this`, so calls can be **chained**: `sb.append("a").append(1).append(true)`.

```text
new StringBuilder()          capacity 16, length 0
[_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _]

append("Java")               capacity 16, length 4
[J a v a _ _ _ _ _ _ _ _ _ _ _ _]

append(" Developer!!!")      length 16 → still fits
append("X")                  length 17 > 16 → grow to 16*2+2 = 34, copy once
```

### 2.3 Real-World Example

- `String` is like a **printed page** — to change one word, you print a whole new page.
- `StringBuilder` is like a **whiteboard** — you can write, erase and insert text on the same board. When it is full, you get a bigger board once and continue.
- `StringBuffer` is a whiteboard in a **meeting room with a lock** — only one person can write at a time (synchronized), which is safe but slower.

### 2.4 Java Example — StringBuilder Methods

```java
public class StringBuilderDemo {
    public static void main(String[] args) {
        StringBuilder sb = new StringBuilder();
        System.out.println("Initial length=" + sb.length() + ", capacity=" + sb.capacity());

        sb.append("Java");
        sb.append(' ').append(21).append(" LTS");          // chaining, any type can be appended
        System.out.println("After append : " + sb);

        sb.insert(0, "I love ");                           // insert at index 0
        System.out.println("After insert : " + sb);

        sb.replace(7, 11, "JAVA");                         // replace chars 7..10
        System.out.println("After replace: " + sb);

        sb.delete(11, 14);                                 // delete chars 11..13 (" 21")
        System.out.println("After delete : " + sb);

        sb.deleteCharAt(0);                                // delete the 'I'
        System.out.println("After deleteCharAt: " + sb);

        sb.setCharAt(0, '*');                              // change one character
        System.out.println("After setCharAt : " + sb);

        System.out.println("charAt(3) : " + sb.charAt(3));
        System.out.println("indexOf(\"JAVA\") : " + sb.indexOf("JAVA"));

        StringBuilder word = new StringBuilder("level");
        System.out.println("Palindrome? " + word.toString().equals(word.reverse().toString()));

        StringBuilder sized = new StringBuilder("Java");
        System.out.println("new StringBuilder(\"Java\") capacity = " + sized.capacity());
        sized.append("123456789012345678");                // length 22 > 20 → grows
        System.out.println("After growth: length=" + sized.length() + ", capacity=" + sized.capacity());

        String result = sb.toString();                     // convert back to String
        System.out.println("Final String: " + result);
    }
}
```

### 2.5 Code Explanation

- A new empty `StringBuilder` has length 0 and capacity 16.
- `append` accepts any type (`char`, `int`, `boolean`, `Object` …) and converts it to text.
- After `insert(0, "I love ")`, the content is `"I love Java 21 LTS"`.
  Indexes: `I`=0, ` `=1, `l`=2 … ` `=6, `J`=7, `a`=8, `v`=9, `a`=10, ` `=11, `2`=12, `1`=13, ` `=14, `L`=15 …
- `replace(7, 11, "JAVA")` replaces indexes 7–10 (`"Java"`) — end index exclusive, like `substring`.
- `delete(11, 14)` removes `" 21"` (indexes 11, 12, 13).
- `reverse()` **modifies the builder itself** and returns it. For the palindrome check, we take `toString()` **before** reversing so we have the original text to compare.
- `new StringBuilder("Java")` → capacity 16 + 4 = 20. Appending 18 more characters makes length 22 > 20, so capacity becomes 20 × 2 + 2 = 42.
- `toString()` creates a new `String` from the current content.

### 2.6 Expected Output

```text
Initial length=0, capacity=16
After append : Java 21 LTS
After insert : I love Java 21 LTS
After replace: I love JAVA 21 LTS
After delete : I love JAVA LTS
After deleteCharAt:  love JAVA LTS
After setCharAt : *love JAVA LTS
charAt(3) : v
indexOf("JAVA") : 6
Palindrome? true
new StringBuilder("Java") capacity = 20
After growth: length=22, capacity=42
Final String: *love JAVA LTS
```

---

## 3. StringBuffer

### 3.1 Interview Definition

> **`StringBuffer` is a mutable, thread-safe sequence of characters in the `java.lang` package, available since Java 1.0. It provides the same operations as `StringBuilder` — `append()`, `insert()`, `delete()`, `reverse()` and so on — but all its public methods are `synchronized`, so only one thread can modify a `StringBuffer` object at a time. This makes it safe for use by multiple threads but slower than `StringBuilder` because of the locking overhead.**

### 3.2 Understand the Concept

- `StringBuffer` came first (Java 1.0). Most string building happens in a single thread, so the synchronization was unnecessary overhead.
- **Java 5** added `StringBuilder` as a **drop-in, unsynchronized replacement** with the same API.
- Today, `StringBuffer` is rarely needed. Even when multiple threads are involved, it is usually better to build strings in each thread separately, or use other concurrency tools.

### 3.3 Java Example — Thread Safety Difference

<!-- output-varies -->
```java
public class BufferVsBuilderThreads {

    // Runs the same appending task on two threads and returns the final length
    static int appendFromTwoThreads(Appendable target) throws InterruptedException {
        Runnable task = () -> {
            for (int i = 0; i < 100_000; i++) {
                try {
                    target.append('x');
                } catch (Exception e) {
                    // StringBuilder may even throw ArrayIndexOutOfBoundsException
                    // when two threads resize it at the same time; we ignore it here
                }
            }
        };
        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        return ((CharSequence) target).length();
    }

    public static void main(String[] args) throws InterruptedException {
        System.out.println("StringBuffer length : " + appendFromTwoThreads(new StringBuffer()));
        System.out.println("StringBuilder length: " + appendFromTwoThreads(new StringBuilder()));
    }
}
```

**Possible Output** (the second number changes from run to run):

```text
StringBuffer length : 200000
StringBuilder length: 196841
```

**Explanation:** Two threads append 100,000 characters each to the same object (both classes implement the `Appendable` interface, so one method can test both). `StringBuffer`'s synchronized `append` ensures that every append completes fully before another starts, so the length is always **200,000**. `StringBuilder` has no locking, so the two threads can read and update the internal `count` at the same time; some appends are lost (the result is usually less than 200,000 and changes on every run), and occasionally an `ArrayIndexOutOfBoundsException` is thrown during resizing. This is a **race condition** (see [Part 7](../07-Multithreading-and-Concurrency/04-Synchronization-Race-Conditions-Locks.md)).

---

## 4. String vs StringBuilder vs StringBuffer

### 4.1 Comparison Table

| Point | `String` | `StringBuilder` | `StringBuffer` |
|---|---|---|---|
| Mutability | **Immutable** | Mutable | Mutable |
| Thread-safe | Yes (because immutable) | **No** | Yes (synchronized methods) |
| Performance for many changes | Slowest (new object each change) | **Fastest** | Slower than StringBuilder (locking) |
| Introduced | Java 1.0 | **Java 5** | Java 1.0 |
| Storage | String pool (literals) / heap | Heap | Heap |
| `equals()` | Compares content | **Not overridden** (compares references) | **Not overridden** |
| `hashCode()` | Content-based, cached | From `Object` | From `Object` |
| Implements `Comparable` | Yes | Yes (Java 11+) | Yes (Java 11+) |
| `+` operator | Supported | Not supported (use `append`) | Not supported |
| Use when | Value does not change; keys, constants | Building strings in a single thread (loops) | Building strings shared by multiple threads (legacy) |

### 4.2 Performance Demonstration

<!-- output-varies -->
```java
public class PerformanceTest {
    public static void main(String[] args) {
        int n = 50_000;

        long start = System.nanoTime();
        String s = "";
        for (int i = 0; i < n; i++) {
            s += "a";
        }
        long stringTime = System.nanoTime() - start;

        start = System.nanoTime();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < n; i++) {
            sb.append("a");
        }
        long builderTime = System.nanoTime() - start;

        start = System.nanoTime();
        StringBuffer sbf = new StringBuffer();
        for (int i = 0; i < n; i++) {
            sbf.append("a");
        }
        long bufferTime = System.nanoTime() - start;

        System.out.println("String        : " + stringTime / 1_000_000 + " ms");
        System.out.println("StringBuilder : " + builderTime / 1_000_000 + " ms");
        System.out.println("StringBuffer  : " + bufferTime / 1_000_000 + " ms");
    }
}
```

**Typical Output** (numbers depend on the machine):

```text
String        : 412 ms
StringBuilder : 1 ms
StringBuffer  : 2 ms
```

The `String` version is hundreds of times slower because it copies the whole string on every iteration. `StringBuffer` is slightly slower than `StringBuilder` due to synchronization (the JIT can remove uncontended locks, so the difference is often small).

### 4.3 How is `+` Compiled?

- For **compile-time constants** (`"a" + "b"`), the compiler joins them at compile time.
- For **runtime values** in a single expression (`name + " is " + age`):
  - Up to Java 8: compiled into `new StringBuilder().append(name).append(" is ").append(age).toString()`.
  - **Java 9+**: compiled into an `invokedynamic` call to `StringConcatFactory`, which chooses an efficient strategy at runtime.
- In both cases, a **single expression** with `+` is efficient. The problem is `+` **inside a loop**, because each iteration starts a new builder and creates a new `String`.

### 4.4 Interview Answer — "String vs StringBuilder vs StringBuffer"

> "All three represent sequences of characters, but they differ in mutability, thread safety and performance. **String** is immutable — every modification creates a new object — so it is thread-safe and can be stored in the string pool, but it is slow when we modify it many times. **StringBuilder** and **StringBuffer** are mutable — they change the same object using an internal resizable array — so they are much faster for repeated changes like appending in a loop. The difference between them is thread safety: **StringBuffer** has synchronized methods, so it is thread-safe but slower. **StringBuilder** was added in Java 5 without synchronization, so it is faster but not thread-safe. In practice, I use String for fixed values, StringBuilder for building strings in a single thread, and StringBuffer only if the same object must be modified by multiple threads. Also, StringBuilder and StringBuffer do not override `equals()`, so to compare content I convert them with `toString()` or use `compareTo()`."

---

## 5. Interview Questions with Complete Answers

### Basic Level

**Q1. Why is `StringBuilder` faster than `String` for concatenation in loops?**

`String` is immutable, so each `+` creates a new object and copies all existing characters — about O(n²) total work for n appends. `StringBuilder` keeps one internal array and appends in place; it only copies when the array must grow, and it doubles the capacity each time, so total work is about O(n).

**Q2. What is the default capacity of `StringBuilder`?**

16 characters. With a String argument, capacity is 16 + the string's length. With an `int` argument, `new StringBuilder(100)`, capacity is exactly 100. Setting an initial capacity when you know the approximate final size avoids repeated resizing.

**Q3. How do you convert a `StringBuilder` to a `String` and vice versa?**

`String s = sb.toString();` and `StringBuilder sb = new StringBuilder(s);`. `String.valueOf(sb)` also works.

### Intermediate Level

**Q4. What is the output of `new StringBuilder("abc").equals(new StringBuilder("abc"))`?**

`false`. `StringBuilder` does not override `equals()`, so it uses `Object.equals()`, which compares references. Compare content with `sb1.toString().equals(sb2.toString())`, or with `sb1.compareTo(sb2) == 0` (Java 11+), or `sb1.toString().contentEquals(sb2)`.

**Q5. How does the capacity grow?**

When an append needs more space than the current capacity, the new capacity is **(old capacity × 2) + 2**. If that is still not enough for the required length, the required length is used instead. The old characters are copied into the new array. You can reserve space with `ensureCapacity(n)` and release unused space with `trimToSize()`.

**Q6. Can you use `StringBuilder` as a `HashMap` key?**

Technically yes, but it is a **bad idea**. It does not override `equals()`/`hashCode()`, so two builders with the same content are different keys, and lookups using a new builder fail. Also, since it is mutable, changing it after insertion causes confusion. Use `String` keys.

**Q7. Why was `StringBuilder` introduced when `StringBuffer` already existed?**

Because most string building is done within a single thread, and `StringBuffer`'s synchronization added unnecessary locking cost. `StringBuilder` (Java 5) offers the same API without synchronization, giving better performance in the common case.

### Advanced Level

**Q8. Reverse each word of a sentence using `StringBuilder`.**

```java
public class ReverseWords {
    public static void main(String[] args) {
        String sentence = "Java is fun";
        StringBuilder result = new StringBuilder();
        for (String word : sentence.split(" ")) {
            result.append(new StringBuilder(word).reverse()).append(' ');
        }
        System.out.println(result.toString().trim());

        // Reverse the order of words
        String[] words = sentence.split(" ");
        StringBuilder reversedOrder = new StringBuilder();
        for (int i = words.length - 1; i >= 0; i--) {
            reversedOrder.append(words[i]);
            if (i > 0) reversedOrder.append(' ');
        }
        System.out.println(reversedOrder);
    }
}
```

**Expected Output:**

```text
avaJ si nuf
fun is Java
```

**Q9. Is `StringBuffer` completely thread-safe for all operations?**

Each **individual method** is synchronized, so a single `append` is atomic. But a **sequence** of calls is not atomic. For example, `if (buf.length() > 0) buf.deleteCharAt(0);` can fail if another thread empties the buffer between the two calls. For compound operations, you still need external synchronization (`synchronized (buf) { ... }`).

**Q10. What does `sb.append(sb)` do?**

It appends the current content of the builder to itself — the content is doubled. `new StringBuilder("ab").append(...)` with itself gives `"abab"`. The implementation handles this safely.

---

## 6. Follow-Up Questions

**F1. Does `reverse()` handle surrogate pairs (like emojis)?**

Yes. `StringBuilder.reverse()` treats surrogate pairs as single units, so emojis and other supplementary characters are not broken during reversal.

**F2. Is `sb.toString()` creating a new object every time?**

Yes. Each call creates a new `String` with a copy of the current characters, so the returned string is not affected by later changes to the builder.

**F3. Should I always use `StringBuilder` instead of `+`?**

No. For a single expression like `"Hello " + name + "!"`, `+` is clear and efficiently compiled. Use `StringBuilder` when building a string across **multiple statements**, especially in **loops**. For joining with a delimiter, `String.join()`, `StringJoiner` or `Collectors.joining()` are even clearer.

---

## 7. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Using `equals()` on StringBuilders | Compares references; use `toString()` or `compareTo()`. |
| `String s = sb;` | Compile error; use `sb.toString()`. |
| Comparing `sb.reverse()` to `sb.toString()` after reversing | `reverse()` changes the builder itself. Save the original first. |
| Using `StringBuffer` by default | Use `StringBuilder` unless shared between threads. |
| `s += x` in large loops | Use `StringBuilder`. |
| Thinking `StringBuffer` makes compound operations atomic | Only single method calls are synchronized. |

---

## 8. Quick Revision

- `String` → immutable; `StringBuilder` → mutable, not synchronized, fastest (Java 5); `StringBuffer` → mutable, synchronized, thread-safe (Java 1.0).
- Default capacity 16; with string: 16 + length; growth: **old × 2 + 2**.
- Methods: `append`, `insert`, `delete`, `deleteCharAt`, `replace`, `reverse`, `setCharAt`, `charAt`, `indexOf`, `length`, `capacity`, `toString`.
- `StringBuilder`/`StringBuffer` do **not** override `equals()`/`hashCode()`.
- `+` in one expression is fine (compiled to builder / `invokedynamic`); avoid `+` in loops.
- Choose: fixed text → `String`; single-thread building → `StringBuilder`; shared multi-thread building → `StringBuffer`.

---

[← Previous: String Comparison and Methods](02-String-Comparison-and-Methods.md) | [Back to README](../README.md) | [Next: Arrays →](04-Arrays-and-Multidimensional-Arrays.md)
