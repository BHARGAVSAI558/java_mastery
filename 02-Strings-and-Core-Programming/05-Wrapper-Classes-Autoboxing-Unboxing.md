# Chapter 2.5 — Wrapper Classes, Autoboxing and Unboxing

[← Previous: Arrays](04-Arrays-and-Multidimensional-Arrays.md) | [Back to README](../README.md) | [Next: Varargs and Pass-by-Value →](06-Varargs-and-Pass-by-Value.md)

---

## What You Will Learn

- What wrapper classes are and why Java needs them
- The 8 wrapper classes and their hierarchy
- Autoboxing and unboxing — how the compiler does it
- The **Integer cache** (−128 to 127) and the famous `==` trap
- Useful wrapper methods: `parseXxx`, `valueOf`, `xxxValue`, `compare`, `MIN_VALUE/MAX_VALUE`
- `NullPointerException` from unboxing
- Performance cost of boxing

---

## 1. Wrapper Classes

### 1.1 Interview Definition

> **Wrapper classes are classes in the `java.lang` package that wrap (enclose) a primitive value inside an object. Java provides one wrapper class for each of the 8 primitive types: `Byte`, `Short`, `Integer`, `Long`, `Float`, `Double`, `Character` and `Boolean`. Wrapper classes are needed because many Java features work only with objects — such as collections, generics, `null` values and methods that accept `Object`. Wrapper objects are immutable and provide useful utility methods for parsing strings, converting between types and comparing values.**

### 1.2 Understand the Concept

| Primitive | Wrapper class | Size of primitive |
|---|---|---|
| `byte` | `Byte` | 1 byte |
| `short` | `Short` | 2 bytes |
| `int` | **`Integer`** | 4 bytes |
| `long` | `Long` | 8 bytes |
| `float` | `Float` | 4 bytes |
| `double` | `Double` | 8 bytes |
| `char` | **`Character`** | 2 bytes |
| `boolean` | `Boolean` | — |

Only `Integer` and `Character` have names different from the primitive (not just a capital letter).

**Hierarchy:**

```text
                  Object
          ┌─────────┼──────────┐
       Number    Character   Boolean
   ┌──┬──┬─┴─┬───┬────┐
 Byte Short Integer Long Float Double
```

- The six numeric wrappers extend the abstract class **`Number`**, which provides `intValue()`, `longValue()`, `doubleValue()`, etc.
- All wrapper classes are **`final`** and **immutable**.

**Why do we need wrapper classes?**
1. **Collections and generics work only with objects.** `List<int>` is a compile error; you must write `List<Integer>`.
2. **`null` support.** A primitive `int` cannot be `null`; an `Integer` can. Useful for "no value" (e.g., a database column that may be empty).
3. **Utility methods.** `Integer.parseInt()`, `Integer.toBinaryString()`, `Character.isDigit()`, `Integer.MAX_VALUE`.
4. **Methods that take `Object`.** Such as `Object` parameters, reflection, serialization.
5. **Synchronization and multithreading APIs** work on objects.

### 1.3 Real-World Example

A primitive is like a **loose diamond**. A wrapper class is the **diamond in a box with a certificate**. The box lets you store it on a shelf with other boxes (collections), mark a shelf position as empty (`null`), and the certificate gives extra information and services (utility methods). But the box takes extra space and time to open (performance cost).

---

## 2. Autoboxing and Unboxing

### 2.1 Interview Definition

> **Autoboxing is the automatic conversion of a primitive value into an object of its corresponding wrapper class, for example from `int` to `Integer`. Unboxing is the automatic conversion of a wrapper object back into its primitive value, for example from `Integer` to `int`. Both were introduced in Java 5. The compiler performs these conversions by inserting calls such as `Integer.valueOf()` for boxing and `intValue()` for unboxing, which makes code simpler when using collections and generics.**

### 2.2 Understand the Concept

```text
What you write                 What the compiler generates
Integer a = 10;           →    Integer a = Integer.valueOf(10);      // autoboxing
int b = a;                →    int b = a.intValue();                 // unboxing
list.add(5);              →    list.add(Integer.valueOf(5));         // autoboxing
int sum = a + b;          →    int sum = a.intValue() + b;           // unboxing in arithmetic
a++;                      →    a = Integer.valueOf(a.intValue() + 1);// unbox, add, box again
```

**When does boxing/unboxing happen?**
- Assigning a primitive to a wrapper variable (or the reverse).
- Passing a primitive to a method expecting a wrapper (or the reverse).
- Adding primitives to collections.
- Arithmetic and comparison (`<`, `>`, `+`) with wrapper objects → unboxing.

### 2.3 Java Example

```java
import java.util.ArrayList;
import java.util.List;

public class BoxingDemo {
    static int square(Integer x) {          // parameter is a wrapper
        return x * x;                        // unboxed for multiplication
    }

    public static void main(String[] args) {
        // Manual boxing (old way) vs autoboxing
        Integer manual = Integer.valueOf(25);
        Integer auto = 25;                   // autoboxing
        int primitive = auto;                // unboxing
        System.out.println("manual = " + manual + ", auto = " + auto + ", primitive = " + primitive);

        // Collections need objects
        List<Integer> marks = new ArrayList<>();
        marks.add(90);                       // autoboxing
        marks.add(75);
        int total = 0;
        for (int m : marks) {                // unboxing in for-each
            total += m;
        }
        System.out.println("Total = " + total);

        System.out.println("square(7) = " + square(7));    // 7 is boxed to pass in

        Integer counter = 0;
        counter++;                           // unbox, add, box again (new object)
        System.out.println("counter = " + counter);

        // Wrapper can be null; primitive cannot
        Integer optionalAge = null;
        System.out.println("optionalAge = " + optionalAge);

        // Conversions
        int parsed = Integer.parseInt("123");          // String → int
        Integer boxed = Integer.valueOf("456");        // String → Integer
        double asDouble = boxed.doubleValue();         // Integer → double
        String text = Integer.toString(789);           // int → String
        System.out.println(parsed + " " + boxed + " " + asDouble + " " + text);

        // Useful constants and methods
        System.out.println("Integer.MAX_VALUE = " + Integer.MAX_VALUE);
        System.out.println("Binary of 10 = " + Integer.toBinaryString(10));
        System.out.println("Hex of 255 = " + Integer.toHexString(255));
        System.out.println("Integer.compare(3, 7) = " + Integer.compare(3, 7));
        System.out.println("Character.isDigit('5') = " + Character.isDigit('5'));
        System.out.println("Character.isLetter('x') = " + Character.isLetter('x'));
        System.out.println("Character.toUpperCase('q') = " + Character.toUpperCase('q'));
        System.out.println("Boolean.parseBoolean(\"TRUE\") = " + Boolean.parseBoolean("TRUE"));
        System.out.println("Boolean.parseBoolean(\"yes\") = " + Boolean.parseBoolean("yes"));
    }
}
```

### 2.4 Code Explanation

- `Integer auto = 25;` — the compiler writes `Integer.valueOf(25)`.
- `marks.add(90)` — the `int` is boxed because `List<Integer>` stores objects.
- `for (int m : marks)` — each `Integer` is unboxed into `m`.
- `counter++` — wrappers are immutable, so this creates a **new** `Integer` object and re-assigns the variable.
- `Integer.parseInt` returns a primitive `int`; `Integer.valueOf` returns an `Integer` object.
- `Integer.compare(3, 7)` returns a negative number (−1) because 3 < 7.
- `Boolean.parseBoolean` returns `true` only for `"true"` (ignoring case); anything else, including `"yes"`, gives `false`.

### 2.5 Expected Output

```text
manual = 25, auto = 25, primitive = 25
Total = 165
square(7) = 49
counter = 1
optionalAge = null
123 456 456.0 789
Integer.MAX_VALUE = 2147483647
Binary of 10 = 1010
Hex of 255 = ff
Integer.compare(3, 7) = -1
Character.isDigit('5') = true
Character.isLetter('x') = true
Character.toUpperCase('q') = Q
Boolean.parseBoolean("TRUE") = true
Boolean.parseBoolean("yes") = false
```

### 2.6 Interview Answer — "What is autoboxing and unboxing?"

> "Autoboxing is the automatic conversion of a primitive into its wrapper object — for example, `int` to `Integer` — and unboxing is the reverse, converting `Integer` back to `int`. Both were introduced in Java 5. The compiler does this for us: for boxing it calls `Integer.valueOf()`, and for unboxing it calls `intValue()`. It is mainly useful with collections and generics, because they can only store objects, so when I write `list.add(5)`, the 5 is autoboxed. There are two things to be careful about. First, unboxing a `null` wrapper throws a `NullPointerException`. Second, comparing two `Integer` objects with `==` compares references, and because of the Integer cache, it works for values between −128 and 127 but fails for larger values — so we should use `equals()`."

---

## 3. The Integer Cache and the `==` Trap

### 3.1 Interview Definition

> **The Integer cache is an optimisation in which `Integer.valueOf()` returns pre-created, shared `Integer` objects for values from −128 to 127 instead of creating new objects. Since autoboxing uses `Integer.valueOf()`, two boxed integers within this range refer to the same object, so `==` returns true; outside this range, new objects are created and `==` returns false. Similar caches exist for `Byte`, `Short`, `Long` (−128 to 127), `Character` (0 to 127) and `Boolean` (`TRUE` and `FALSE`), but not for `Float` or `Double`.**

### 3.2 Java Example

```java
public class IntegerCacheDemo {
    public static void main(String[] args) {
        Integer a = 127;
        Integer b = 127;
        System.out.println("127 == 127 : " + (a == b));

        Integer c = 128;
        Integer d = 128;
        System.out.println("128 == 128 : " + (c == d));
        System.out.println("128 equals : " + c.equals(d));

        Integer e = new Integer(100);        // deprecated constructor: always a new object
        Integer f = 100;
        System.out.println("new Integer(100) == 100 boxed : " + (e == f));

        Integer g = 1000;
        int h = 1000;
        System.out.println("Integer 1000 == int 1000 : " + (g == h));   // unboxing → value comparison

        Long l1 = 127L;
        Long l2 = 127L;
        System.out.println("Long 127 == Long 127 : " + (l1 == l2));

        Double d1 = 1.0;
        Double d2 = 1.0;
        System.out.println("Double 1.0 == Double 1.0 : " + (d1 == d2));  // no cache for Double

        Long big = 10L;
        Integer ten = 10;
        System.out.println("Long(10).equals(Integer(10)) : " + big.equals(ten));   // different types
    }
}
```

### 3.3 Code Explanation

- `a` and `b` (127) come from the cache → **same object** → `true`.
- `c` and `d` (128) are outside the cache → **two objects** → `==` is `false`, `equals` is `true`.
- `new Integer(100)` always creates a new object (this constructor is **deprecated since Java 9** and marked for removal). It is not the cached object → `false`.
- `g == h` — one side is a primitive, so `g` is **unboxed** and the values are compared → `true`.
- `Long` also caches −128..127. `Double` and `Float` have **no cache**, so even `1.0 == 1.0` (boxed) is `false`.
- `Long.equals(Integer)` returns `false` because `equals` first checks the **type**.

### 3.4 Expected Output

```text
127 == 127 : true
128 == 128 : false
128 equals : true
new Integer(100) == 100 boxed : false
Integer 1000 == int 1000 : true
Long 127 == Long 127 : true
Double 1.0 == Double 1.0 : false
Long(10).equals(Integer(10)) : false
```

### 3.5 Cache Summary

| Wrapper | Cached values |
|---|---|
| `Byte` | All values (−128 to 127) |
| `Short`, `Integer`, `Long` | −128 to 127 |
| `Character` | 0 to 127 |
| `Boolean` | `TRUE`, `FALSE` (only two objects exist) |
| `Float`, `Double` | No cache |

The upper limit of the `Integer` cache can be increased with the JVM option `-XX:AutoBoxCacheMax=<size>` (or `-Djava.lang.Integer.IntegerCache.high`). The lower limit is always −128.

**Rule:** **Never use `==` to compare wrapper objects. Always use `equals()`** (or unbox explicitly).

---

## 4. NullPointerException from Unboxing

```java
import java.util.HashMap;
import java.util.Map;

public class UnboxingNPE {
    public static void main(String[] args) {
        Map<String, Integer> stock = new HashMap<>();
        stock.put("pen", 50);

        Integer pencils = stock.get("pencil");          // key missing → null
        System.out.println("pencils = " + pencils);

        try {
            int count = stock.get("pencil");            // unboxing null!
            System.out.println(count);
        } catch (NullPointerException e) {
            System.out.println("NullPointerException while unboxing null");
        }

        int safe = stock.getOrDefault("pencil", 0);     // safe alternative
        System.out.println("safe = " + safe);

        Boolean flag = null;
        try {
            if (flag) {                                 // unboxing null Boolean
                System.out.println("never");
            }
        } catch (NullPointerException e) {
            System.out.println("NullPointerException in if(Boolean null)");
        }

        // Ternary trap: mixing Integer and int unboxes the Integer
        Integer maybe = null;
        try {
            Integer result = true ? maybe : 0;          // type of ?: is int → unboxes maybe
            System.out.println(result);
        } catch (NullPointerException e) {
            System.out.println("NullPointerException in ternary with Integer and int");
        }
    }
}
```

**Expected Output:**

```text
pencils = null
NullPointerException while unboxing null
safe = 0
NullPointerException in if(Boolean null)
NullPointerException in ternary with Integer and int
```

**Explanation:**
- `map.get()` returns `null` for a missing key. Assigning it to an `int` calls `null.intValue()` → NPE.
- Use `getOrDefault(key, 0)` or check for `null` first.
- `if (flag)` with a `null` `Boolean` throws NPE because it must be unboxed to `boolean`.
- **Ternary trap:** when one branch is `Integer` and the other is `int`, the whole expression's type is `int`, so the `Integer` branch is unboxed — and `null` causes NPE, even though the result is assigned to an `Integer`.

---

## 5. Performance Cost of Boxing

<!-- output-varies -->
```java
public class BoxingPerformance {
    public static void main(String[] args) {
        long start = System.nanoTime();
        Long boxedSum = 0L;                   // wrapper!
        for (int i = 0; i < 10_000_000; i++) {
            boxedSum += i;                    // unbox, add, box a NEW Long each time
        }
        long boxedTime = System.nanoTime() - start;

        start = System.nanoTime();
        long primitiveSum = 0L;               // primitive
        for (int i = 0; i < 10_000_000; i++) {
            primitiveSum += i;
        }
        long primitiveTime = System.nanoTime() - start;

        System.out.println("Same result? " + (boxedSum == primitiveSum));
        System.out.println("Boxed     : " + boxedTime / 1_000_000 + " ms");
        System.out.println("Primitive : " + primitiveTime / 1_000_000 + " ms");
    }
}
```

**Typical Output:**

```text
Same result? true
Boxed     : 45 ms
Primitive : 4 ms
```

Using `Long` as an accumulator creates millions of temporary objects. **Use primitives for calculations; use wrappers only when an object is required.** For large numeric data in streams, use `IntStream`/`LongStream` instead of `Stream<Integer>`.

---

## 6. Interview Questions with Complete Answers

### Basic Level

**Q1. Why do we need wrapper classes?**

Because many parts of Java work only with objects: collections (`ArrayList<Integer>`), generics, methods that take `Object`, and features that need `null`. Wrapper classes also provide utility methods like `Integer.parseInt()` and constants like `Integer.MAX_VALUE`.

**Q2. What is the difference between `int` and `Integer`?**

| `int` | `Integer` |
|---|---|
| Primitive type | Wrapper class (object) |
| Stores the value directly | Stores a reference to an object |
| Cannot be `null` (default 0) | Can be `null` (default null as a field) |
| Faster, less memory (4 bytes) | Slower, more memory (object header ~16 bytes) |
| Cannot be used in collections/generics | Can be used in collections/generics |
| No methods | Has methods (`compareTo`, `equals`, `toString`) |
| `==` compares values | `==` compares references |

**Q3. What is the difference between `parseInt()` and `valueOf()`?**

`Integer.parseInt("10")` returns a primitive `int`. `Integer.valueOf("10")` returns an `Integer` object (and uses the cache for −128..127). Both throw `NumberFormatException` for invalid input.

### Intermediate Level

**Q4. Why does `Integer a = 127, b = 127; a == b` give true, but with 128 gives false?**

Autoboxing uses `Integer.valueOf()`, which returns cached objects for values from −128 to 127. So both variables point to the same cached object for 127. For 128, `valueOf()` creates a new object each time, so `==` (reference comparison) is false. Always use `equals()` for wrapper comparison.

**Q5. Can unboxing cause an exception?**

Yes, a `NullPointerException`, if the wrapper reference is `null`. This often happens with `map.get()` for a missing key, `Boolean` fields that were never set, or ternary expressions mixing wrapper and primitive types.

**Q6. Are wrapper classes immutable? Why?**

Yes. Their value field is `private final`, and there are no setters. Immutability makes caching safe (shared cached objects cannot be changed by anyone), makes them thread-safe, and makes them safe as `HashMap` keys. That is why `counter++` on an `Integer` creates a new object.

**Q7. Which is preferred: `new Integer(5)` or `Integer.valueOf(5)`?**

`Integer.valueOf(5)` (or simply autoboxing). It can return a cached object and avoid creating new ones. The constructor always creates a new object and has been **deprecated since Java 9** (marked for removal in Java 16+).

### Advanced Level

**Q8. What will this print, and why?**

```java
import java.util.ArrayList;
import java.util.List;

public class RemoveTrap {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>(List.of(10, 20, 30, 1));
        list.remove(1);                       // remove(int index)
        System.out.println(list);
        list.remove(Integer.valueOf(1));      // remove(Object o)
        System.out.println(list);
    }
}
```

**Expected Output:**

```text
[10, 30, 1]
[10, 30]
```

`List` has two `remove` methods: `remove(int index)` and `remove(Object o)`. For `remove(1)`, the exact match `remove(int)` wins over boxing (overload resolution prefers no boxing), so it removes the element at **index 1** (20). To remove the value 1, pass an `Integer` object.

**Q9. How does overloading interact with boxing?**

The compiler resolves overloads in phases: (1) exact match or widening without boxing, (2) with boxing/unboxing, (3) with varargs. So `m(long)` is chosen over `m(Integer)` for an `int` argument. This preserves backward compatibility with pre-Java-5 code.

**Q10. What is the size of an `Integer` object compared to an `int`?**

An `int` is 4 bytes. An `Integer` object on a 64-bit JVM with compressed pointers has a 12-byte header plus the 4-byte value = **16 bytes**, plus a 4-byte reference to it. So a boxed integer costs about 5 times more memory, and also adds GC pressure. This is why `int[]` is much more memory-efficient than `ArrayList<Integer>`.

---

## 7. Follow-Up Questions

**F1. How do you convert `char` digits to `int`?**

`Character.getNumericValue('7')` → 7, or `'7' - '0'` → 7 (because digits are consecutive in Unicode). `(int) '7'` gives 55 (its Unicode value), not 7.

**F2. What does `Integer.valueOf("08")` and `Integer.parseInt("1_000")` give?**

`Integer.valueOf("08")` → 8 (leading zeros are fine in parsing; only *literals* with leading 0 are octal). `Integer.parseInt("1_000")` → `NumberFormatException` (underscores are allowed only in source code literals, not in parsed strings).

**F3. How do you compare two `Double` values safely?**

Use `Double.compare(a, b)` for ordering, or `a.equals(b)`. For calculated values, compare with a tolerance: `Math.abs(a - b) < 1e-9`. Note: `Double.NaN.equals(Double.NaN)` is `true`, but `Double.NaN == Double.NaN` (primitive) is `false`.

---

## 8. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Comparing `Integer`s with `==` | Use `equals()`; `==` works only by chance within −128..127. |
| `int x = map.get(key)` | NPE if key missing; use `getOrDefault`. |
| Using `Long sum = 0L` in loops | Creates many objects; use `long`. |
| `list.remove(1)` to remove value 1 | Removes index 1; use `remove(Integer.valueOf(1))`. |
| `new Integer(5)` | Deprecated; use `Integer.valueOf(5)`. |
| Expecting `Boolean.parseBoolean("yes")` to be true | Only `"true"` (any case) is true. |
| `Long.equals(Integer)` with same number | false (different types). |

---

## 9. Quick Revision

- 8 wrappers: `Byte Short Integer Long Float Double Character Boolean`; numeric ones extend `Number`; all `final` and immutable.
- Needed for collections/generics, `null`, utility methods.
- **Autoboxing:** primitive → wrapper (`Integer.valueOf`). **Unboxing:** wrapper → primitive (`intValue`). Java 5.
- **Cache:** −128..127 for Byte/Short/Integer/Long, 0..127 Character, Boolean TRUE/FALSE; none for Float/Double.
- `==` on wrappers → references; use `equals()`.
- Unboxing `null` → `NullPointerException` (watch `map.get`, `Boolean` in `if`, ternary).
- `parseXxx` → primitive; `valueOf` → wrapper.
- Boxing is slow in loops; prefer primitives.

---

[← Previous: Arrays](04-Arrays-and-Multidimensional-Arrays.md) | [Back to README](../README.md) | [Next: Varargs and Pass-by-Value →](06-Varargs-and-Pass-by-Value.md)
