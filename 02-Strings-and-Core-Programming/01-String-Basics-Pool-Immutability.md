# Chapter 2.1 — String Basics, String Pool and Immutability

[← Previous Part: Fundamentals](../01-Java-Fundamentals/09-Command-Line-Arguments-and-Foundations.md) | [Back to README](../README.md) | [Next: String Comparison and Methods →](02-String-Comparison-and-Methods.md)

---

## What You Will Learn

- What a `String` is in Java and how strings are created
- String literal vs `new String()`
- The **String Constant Pool** (SCP) — where it lives and how it works
- `intern()`
- **Immutability** — what it means, why Java made `String` immutable, and its benefits
- How many objects are created? (a very common interview question)
- Compile-time vs runtime concatenation

---

## 1. What is a String?

### 1.1 Interview Definition

> **In Java, a `String` is an object that represents a sequence of characters. It is defined by the `final` class `java.lang.String`, which implements the `CharSequence`, `Comparable<String>` and `Serializable` interfaces. String objects are immutable, which means their value cannot be changed after creation; any operation that seems to modify a string actually creates a new `String` object. Strings can be created using string literals, which are stored in the String Constant Pool and reused, or using the `new` keyword, which always creates a new object in the heap.**

### 1.2 Understand the Concept

- `String` is a **class**, not a primitive — but Java gives it special support: literals in double quotes, the `+` operator for joining, and the string pool.
- Internally, a string's characters are stored in a private `final` array.
  - Up to Java 8: `private final char[] value` (2 bytes per character).
  - **Java 9+ (Compact Strings):** `private final byte[] value` plus a `coder` field. If all characters are Latin-1, each character uses **1 byte**; otherwise UTF-16 (2 bytes). This saves a lot of memory because most strings are plain English.
- `String` is `final`, so no class can extend it and change its behaviour.

**Ways to create a String:**

| Way | Example | Where the object goes |
|---|---|---|
| String literal | `String s = "Java";` | String Constant Pool (reused if it already exists) |
| `new` keyword | `String s = new String("Java");` | Always a new object in the heap (and the literal `"Java"` in the pool) |
| From char array | `new String(new char[]{'J','a'})` | Heap |
| From other methods | `String.valueOf(10)`, `sb.toString()`, `s.concat("x")` | Heap (new object) |

### 1.3 Real-World Example

A string is like **text printed on a printed certificate**. You cannot erase or change the name on it. If you need a different name, a **new certificate is printed**. The old one stays the same (and may be thrown away later by the garbage collector).

---

## 2. The String Constant Pool (SCP)

### 2.1 Interview Definition

> **The String Constant Pool (also called the String Intern Pool) is a special memory area inside the heap where Java stores string literals. When a string literal is created, the JVM first checks the pool; if an equal string already exists, the existing reference is returned instead of creating a new object. This saves memory by sharing identical strings. Strings created with `new` are not placed in the pool automatically, but they can be added using the `intern()` method. Since Java 7, the String Constant Pool is located in the main heap area, so pooled strings can be garbage collected when no longer referenced.**

### 2.2 Understand the Concept

```java
String s1 = "Java";
String s2 = "Java";
String s3 = new String("Java");
```

```text
          STACK                         HEAP
                                ┌────────────────────────────┐
  s1 [ref]──────────┐           │   String Constant Pool     │
                    ├──────────►│      "Java"                │
  s2 [ref]──────────┘           │                            │
                                └────────────────────────────┘
  s3 [ref]────────────────────►  String object "Java"   (separate heap object)
```

- `s1` creates `"Java"` in the pool.
- `s2` finds `"Java"` already in the pool and **reuses** it → `s1 == s2` is `true`.
- `s3` uses `new`, which **always** creates a new object outside the pool → `s1 == s3` is `false`.

**Why does the pool exist?** Programs use the same strings again and again ("OK", "id", "name", "SELECT"…). Sharing one copy saves a lot of memory. Sharing is only **safe because strings are immutable** — if one reference could change the shared string, every other reference would see the change.

**Where is the pool?**

| Java version | Location |
|---|---|
| Java 6 and before | **PermGen** (fixed size, could cause `OutOfMemoryError: PermGen space`) |
| **Java 7 and later** | **Main heap** (can grow and pooled strings can be garbage collected) |

(Java 8 replaced PermGen with Metaspace, but the string pool remains in the heap.)

### 2.3 Real-World Example

The string pool is like a **library**. If a book (string) already exists, the library gives you the same copy to read instead of buying a new one. `new String()` is like **buying your own personal copy** even though the library has one.

### 2.4 Java Example

```java
public class StringPoolDemo {
    public static void main(String[] args) {
        String s1 = "Java";
        String s2 = "Java";
        String s3 = new String("Java");
        String s4 = s3.intern();          // returns the pooled "Java"

        System.out.println("s1 == s2      : " + (s1 == s2));
        System.out.println("s1 == s3      : " + (s1 == s3));
        System.out.println("s1.equals(s3) : " + s1.equals(s3));
        System.out.println("s1 == s4      : " + (s1 == s4));

        // Compile-time constant concatenation is done by the compiler
        String s5 = "Ja" + "va";
        System.out.println("s1 == \"Ja\" + \"va\"   : " + (s1 == s5));

        // Runtime concatenation creates a new object in the heap
        String part = "Ja";
        String s6 = part + "va";
        System.out.println("s1 == part + \"va\" : " + (s1 == s6));
        System.out.println("s1 == s6.intern()  : " + (s1 == s6.intern()));

        // final variable makes it a compile-time constant again
        final String finalPart = "Ja";
        String s7 = finalPart + "va";
        System.out.println("s1 == finalPart + \"va\" : " + (s1 == s7));
    }
}
```

### 2.5 Code Explanation

- `s1 == s2` → both refer to the same pooled object → `true`.
- `s1 == s3` → `s3` is a separate heap object → `false`; but `equals` compares content → `true`.
- `s3.intern()` → looks up `"Java"` in the pool and returns that reference → `s1 == s4` is `true`.
- `"Ja" + "va"` → both are literals, so the **compiler** joins them into `"Java"` at compile time. The bytecode contains just `"Java"`, which is pooled.
- `part + "va"` → `part` is a variable, so concatenation happens at **runtime** and creates a new `String` in the heap (not pooled).
- `finalPart + "va"` → `finalPart` is a `final` variable initialised with a literal, so it is a **compile-time constant**; the compiler folds it to `"Java"`.

### 2.6 Expected Output

```text
s1 == s2      : true
s1 == s3      : false
s1.equals(s3) : true
s1 == s4      : true
s1 == "Ja" + "va"   : true
s1 == part + "va" : false
s1 == s6.intern()  : true
s1 == finalPart + "va" : true
```

### 2.7 Interview Answer — "What is the String Constant Pool?"

> "The String Constant Pool is a special area in the heap where Java stores string literals. When we write `String s = "Java"`, the JVM checks the pool first. If `"Java"` already exists, it returns the same reference; otherwise it creates the string in the pool. So two literals with the same value share one object, and `==` returns true for them. When we use `new String("Java")`, a new object is always created in the heap outside the pool, so `==` returns false, although `equals()` returns true. We can move a string into the pool, or get the pooled copy, using `intern()`. The pool saves memory, and it is safe only because strings are immutable. Since Java 7, the pool is in the main heap instead of PermGen."

---

## 3. How Many Objects Are Created?

This is one of the most frequently asked string questions.

| Code | Objects created | Explanation |
|---|---|---|
| `String s = "Java";` (first time) | **1** | One in the pool |
| `String s = "Java";` (pool already has "Java") | **0** | Reuses pooled object |
| `String s = new String("Java");` (first time) | **2** | One in the pool for the literal, one in the heap from `new` |
| `String s = new String("Java");` (pool already has "Java") | **1** | Only the heap object |
| `String a = "Hello" + "World";` | **1** | Compiler folds to `"HelloWorld"` (pool) |
| `String a = "Hello"; String b = a + "World";` | **3** (in total: "Hello", "World" in pool + 1 result in heap) | Runtime concatenation creates the result in the heap |

**Interview answer for `new String("Java")`:**
> "It creates two objects if `"Java"` is not already in the pool — one in the String Constant Pool because of the literal, and one in the heap because of `new`. The reference variable points to the heap object. If `"Java"` is already in the pool, only one new object is created in the heap."

---

## 4. String Immutability

### 4.1 Interview Definition

> **String immutability means that once a `String` object is created, its value cannot be changed. Methods such as `concat()`, `toUpperCase()`, `replace()` and `substring()` do not modify the original string; they return a new `String` object with the changed value. The `String` class achieves this by being declared `final`, by storing its characters in a `private final` array, and by never providing any method that modifies that array. Java made strings immutable for security, thread safety, the string pool, hashcode caching and safe use as keys in hash-based collections.**

### 4.2 Understand the Concept

```java
String s = "Java";
s.concat(" Rocks");        // creates "Java Rocks" but nobody keeps it
System.out.println(s);      // still "Java"
s = s.concat(" Rocks");    // now s points to the NEW object
```

```text
Before:  s ──► "Java"

After s.concat(" Rocks") without assignment:
         s ──► "Java"
               "Java Rocks"   ← created but unreferenced → eligible for GC

After s = s.concat(" Rocks"):
               "Java"          ← unchanged (still in the pool)
         s ──► "Java Rocks"
```

**The variable can change; the object cannot.** Re-assigning `s` makes it point to a different object; the old object is not modified. (`final String s` would prevent re-assignment of the variable — that is a different concept.)

### 4.3 Why is String Immutable? (5 reasons — memorise these)

1. **String Constant Pool** — Many references share one pooled object. If strings were mutable, changing one would silently change all the others. Immutability makes sharing safe.
2. **Security** — Strings are used for file paths, URLs, database usernames, class names for class loading, and network hosts. If a string could be changed after a security check, an attacker could pass the check with a safe value and then change it.
3. **Thread safety** — Immutable objects can be shared between threads without synchronization, because no thread can change them.
4. **Hashcode caching** — `String` caches its `hashCode()` in a private field the first time it is calculated. Since the value never changes, the hash never changes. This makes strings fast and reliable as **`HashMap` keys**.
5. **Class loading** — Class names are passed as strings to class loaders. Immutability ensures the correct class is loaded.

### 4.4 Real-World Example

A **bank cheque** after it is signed: you cannot edit the amount. If you need a new amount, you write a new cheque. This protects everyone who relies on that cheque (security), many people can look at it at the same time safely (thread safety), and its identity (cheque number = hashcode) never changes.

### 4.5 Java Example

```java
import java.util.HashMap;
import java.util.Map;

public class ImmutabilityDemo {
    public static void main(String[] args) {
        String original = "hello";
        String upper = original.toUpperCase();
        String replaced = original.replace('l', 'L');
        String joined = original.concat(" world");

        System.out.println("original : " + original);
        System.out.println("upper    : " + upper);
        System.out.println("replaced : " + replaced);
        System.out.println("joined   : " + joined);
        System.out.println("same object after toUpperCase? " + (original == upper));

        // Reassignment changes the reference, not the object
        String a = "Java";
        String b = a;
        a = a + " 21";
        System.out.println("a = " + a + ", b = " + b);

        // Safe as HashMap key because its hashCode never changes
        Map<String, Integer> marks = new HashMap<>();
        String key = "Ravi";
        marks.put(key, 90);
        key = key + "Kumar";             // creates a new String; map key is untouched
        System.out.println("Lookup 'Ravi' : " + marks.get("Ravi"));

        // When nothing changes, methods may return the same object
        String noChange = original.replace('z', 'y');
        System.out.println("replace with no match returns same object? " + (noChange == original));
    }
}
```

### 4.6 Code Explanation

- Every "modifying" method returns a **new** string; `original` is still `"hello"`.
- `b = a` copies the reference. `a = a + " 21"` creates a new object and points `a` to it; `b` still points to `"Java"`.
- Changing the local variable `key` cannot affect the key stored inside the map, because the stored `String` object itself can never change.
- When a method has nothing to change (no `'z'` found), the JDK implementation returns `this` (the same object) — a small optimisation that is safe only because strings are immutable.

### 4.7 Expected Output

```text
original : hello
upper    : HELLO
replaced : heLLo
joined   : hello world
same object after toUpperCase? false
a = Java 21, b = Java
Lookup 'Ravi' : 90
replace with no match returns same object? true
```

### 4.8 Interview Answer — "Why is String immutable in Java?"

> "String is immutable, which means once a String object is created, its content cannot be changed. Any method like `concat`, `replace` or `toUpperCase` returns a new String object. Java made it immutable for several reasons. First, the **String Constant Pool**: many references share the same literal, so if one could change it, all of them would be affected. Second, **security**: strings hold sensitive values like file paths, URLs, usernames and class names, and they must not be changed after validation. Third, **thread safety**: immutable strings can be shared across threads without synchronization. Fourth, **hashcode caching**: the hash code is calculated once and cached, which makes String a fast and safe key in `HashMap`. The class is made immutable by being `final`, keeping its value in a `private final` array, and not exposing any method that changes it."

### 4.9 Disadvantage of Immutability

Repeated modification creates many temporary objects:

<!-- skip-check -->
```java
String result = "";
for (int i = 0; i < 10000; i++) {
    result = result + i;      // creates a new String every iteration → slow, O(n²) copying
}
```

Use **`StringBuilder`** for repeated changes ([next chapter but one](03-StringBuilder-and-StringBuffer.md)).

---

## 5. Interview Questions with Complete Answers

### Basic Level

**Q1. Is `String` a primitive type or a class?**

`String` is a **class** in the `java.lang` package, so it is a reference type. It has special language support (literals, `+` operator, string pool) which makes it feel like a primitive.

**Q2. What is the difference between `String s = "abc"` and `String s = new String("abc")`?**

The literal version checks the String Constant Pool and reuses an existing `"abc"` if present, or creates it there. The `new` version always creates a **new object in the heap** (and also ensures `"abc"` exists in the pool because of the literal argument). So two literals compare `==` true, but a literal and a `new` string compare `==` false. Using `new String("abc")` is almost never needed and wastes memory.

**Q3. What does immutable mean?**

An immutable object's state cannot change after it is created. For `String`, its characters can never be modified. Operations return new objects.

### Intermediate Level

**Q4. What is `intern()`?**

`intern()` returns the canonical (pooled) representation of a string. If the pool already contains an equal string (by `equals()`), it returns the pooled reference; otherwise it adds this string to the pool and returns its reference. After interning, two equal strings can be compared using `==`. It is used to save memory when the program has many duplicate strings created at runtime, but overusing it can slow things down.

**Q5. Why is `String` declared `final`?**

So that no subclass can override its methods and break immutability. If `String` could be extended, someone could create a `MutableString extends String` and pass it wherever a `String` is expected — for example, into a security check — and then change its value. `final` guarantees every `String` behaves exactly as specified.

**Q6. Is `String` thread-safe?**

Yes. Because it is immutable, multiple threads can read the same `String` object without any synchronization. No thread can ever see it in a half-changed state. (Note: a *variable* holding a string can still be re-assigned by different threads — that is a separate issue.)

**Q7. Why is `String` a good key for `HashMap`?**

Because it is immutable and caches its hash code. A key's hash code must not change while it is in a map; otherwise the map would look in the wrong bucket and fail to find it. `String` also correctly overrides `equals()` and `hashCode()`.

### Advanced Level

**Q8. How many objects are created by `String s = new String("Hello");`?**

Two objects if `"Hello"` is not already in the pool (one in the pool for the literal, one in the heap from `new`); one object if the pool already contains `"Hello"`. The variable `s` refers to the heap object.

**Q9. What is the output and why?**

```java
public class PoolTrick {
    public static void main(String[] args) {
        String a = "Hello";
        String b = "Hel";
        String c = b + "lo";
        String d = "Hel" + "lo";
        System.out.println(a == c);
        System.out.println(a == d);
        System.out.println(a == c.intern());
        System.out.println(a.equals(c));
    }
}
```

**Expected Output:**

```text
false
true
true
true
```

`c` is built at runtime from a non-final variable → new heap object → `false`. `d` is a compile-time constant → folded to `"Hello"` → same pooled object → `true`. `c.intern()` returns the pooled `"Hello"` → `true`. `equals` compares characters → `true`.

**Q10. Why are passwords stored in `char[]` instead of `String`?**

A `String` is immutable, so you **cannot erase** it from memory after use; it stays in the heap (and possibly the pool) until garbage collection, and could appear in a memory dump. A `char[]` can be **overwritten** with zeros immediately after use (`Arrays.fill(password, '\0')`). Also, printing a `char[]` by mistake shows an address-like value, not the password. That is why `Console.readPassword()` and Swing's `JPasswordField.getPassword()` return `char[]`.

**Q11. What are Compact Strings (Java 9)?**

Before Java 9, every `String` stored its characters in a `char[]`, using 2 bytes per character. Since most strings contain only Latin-1 characters, half the memory was wasted. Java 9 changed the internal storage to a `byte[]` with a `coder` flag: Latin-1 strings use 1 byte per character, and other strings use UTF-16. This reduces heap usage significantly without changing the public API.

---

## 6. Follow-Up Questions

**F1. If strings are immutable, how does `s = s + "x"` work?**

The variable `s` is re-assigned to a **new** `String` object that contains the joined value. The original object is not modified. The compiler implements `+` using `StringBuilder` (before Java 9) or `StringConcatFactory` via `invokedynamic` (Java 9+).

**F2. Can we make our own class immutable like `String`?**

Yes: make the class `final`, make all fields `private final`, provide no setters, initialise everything in the constructor, and make **defensive copies** of mutable fields (in the constructor and getters). See [Part 10, Chapter 8](../10-Additional-Java-Topics/08-Immutable-Classes-and-Records.md).

**F3. Can reflection break String immutability?**

Historically, reflection could change the private `value` array. Since **Java 9+ modules** and stronger encapsulation (fully enforced in Java 16+/17), reflective access to `java.lang.String` internals is blocked by default and throws `InaccessibleObjectException`. So in modern Java, strings are effectively protected.

**F4. Are strings in the pool ever garbage collected?**

Yes, since Java 7 (pool in the heap). If no references to a pooled string remain, it can be collected. String literals referenced by loaded classes stay alive as long as those classes are loaded.

---

## 7. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Comparing strings with `==` | Compares references. Use `equals()`. |
| Thinking `s.concat("x")` changes `s` | It returns a new string; you must assign it. |
| Thinking `final String` = immutable String | `final` stops re-assigning the variable; immutability is about the object. |
| Saying the pool is in PermGen | Since Java 7 it is in the heap. |
| Saying `new String("a")` always creates 2 objects | Only if `"a"` is not already pooled. |
| Concatenating in loops with `+` | Use `StringBuilder`. |

---

## 8. Quick Revision

- `String` = `final` class in `java.lang`, implements `CharSequence`, `Comparable`, `Serializable`.
- Internally `private final byte[] value` (Java 9+ compact strings), `char[]` before.
- Literal → **String Constant Pool** (reused). `new` → always new heap object.
- Pool location: PermGen (≤ Java 6) → **heap (Java 7+)**.
- `intern()` → returns pooled copy.
- Compile-time constants (`"a" + "b"`, `final` variables) are folded and pooled; runtime concatenation is not.
- **Immutable** because: pool sharing, security, thread safety, hashcode caching, class loading.
- `new String("x")` → 2 objects (if "x" not pooled) else 1.
- Passwords → `char[]`, not `String`.

| Expression | `==` result vs `"Java"` literal |
|---|---|
| `"Java"` | true |
| `new String("Java")` | false |
| `new String("Java").intern()` | true |
| `"Ja" + "va"` | true |
| `var + "va"` (non-final var) | false |
| `finalVar + "va"` | true |

---

[← Previous Part: Fundamentals](../01-Java-Fundamentals/09-Command-Line-Arguments-and-Foundations.md) | [Back to README](../README.md) | [Next: String Comparison and Methods →](02-String-Comparison-and-Methods.md)
