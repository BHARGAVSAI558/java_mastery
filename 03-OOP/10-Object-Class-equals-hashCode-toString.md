# Chapter 3.10 — The Object Class: `equals()`, `hashCode()`, `toString()` and More

[← Previous: static and final](09-static-and-final.md) | [Back to README](../README.md) | [Next: Association, Aggregation, Composition →](11-Association-Aggregation-Composition.md)

---

## What You Will Learn

- The `Object` class and all its methods
- `toString()` — default behaviour and how to override it
- `equals()` — default behaviour, the equals contract, and a correct implementation
- `hashCode()` — what it is, the equals–hashCode contract, and what breaks if you ignore it
- `getClass()` vs `instanceof` in `equals()`
- `clone()` — shallow vs deep copy and why copy constructors are often better
- `finalize()` (deprecated), `wait()`/`notify()` (preview)

---

## 1. The `Object` Class

### 1.1 Interview Definition

> **`java.lang.Object` is the root class of the Java class hierarchy. Every class in Java directly or indirectly extends `Object`, and if a class does not explicitly extend another class, the compiler makes it extend `Object`. Therefore every object, including arrays, inherits the methods of `Object`, such as `toString()`, `equals()`, `hashCode()`, `getClass()`, `clone()`, `finalize()`, and the thread communication methods `wait()`, `notify()` and `notifyAll()`. Classes commonly override `toString()`, `equals()` and `hashCode()` to provide meaningful behaviour.**

### 1.2 Methods of the `Object` Class

| Method | Purpose | Usually overridden? |
|---|---|---|
| `public String toString()` | Text representation of the object | ✅ Yes |
| `public boolean equals(Object obj)` | Logical equality | ✅ Yes (for value objects) |
| `public int hashCode()` | Integer hash code used by hash-based collections | ✅ Yes (whenever `equals` is) |
| `public final Class<?> getClass()` | Runtime class of the object | ❌ (final) |
| `protected Object clone()` | Creates a copy (field-by-field) | Sometimes |
| `protected void finalize()` | Called by GC before collection — **deprecated** | ❌ Don't use |
| `public final void wait()`, `wait(long)`, `wait(long, int)` | Thread waits until notified | ❌ (final) |
| `public final void notify()` / `notifyAll()` | Wake waiting thread(s) | ❌ (final) |

`wait/notify` belong to `Object` (not `Thread`) because **every object can be a lock (monitor)** — see [Part 7, Chapter 6](../07-Multithreading-and-Concurrency/06-wait-notify-Producer-Consumer.md).

---

## 2. `toString()`

### 2.1 Interview Definition

> **The `toString()` method returns a string representation of an object. The default implementation in `Object` returns the class name followed by `@` and the hexadecimal form of the object's hash code, such as `Student@1b6d3586`, which is not meaningful. Classes override `toString()` to return a readable description of the object's state. It is called automatically when an object is printed with `System.out.println()` or concatenated with a string.**

### 2.2 Java Example

```java
public class ToStringDemo {

    static class WithoutToString {
        String name = "Ravi";
    }

    static class Student {
        private final String name;
        private final int rollNo;
        private final double cgpa;

        Student(String name, int rollNo, double cgpa) {
            this.name = name;
            this.rollNo = rollNo;
            this.cgpa = cgpa;
        }

        @Override
        public String toString() {
            return "Student{name='" + name + "', rollNo=" + rollNo + ", cgpa=" + cgpa + "}";
        }
    }

    public static void main(String[] args) {
        WithoutToString w = new WithoutToString();
        String defaultText = w.toString();
        System.out.println("Default starts with class name? "
                + defaultText.startsWith("ToStringDemo$WithoutToString@"));
        String hashPart = defaultText.substring(defaultText.indexOf('@') + 1);
        System.out.println("Part after @ equals hex hashCode? " + hashPart.equals(Integer.toHexString(w.hashCode())));

        Student s = new Student("Ravi", 101, 8.7);
        System.out.println(s);                         // println calls toString() automatically
        String text = "Topper: " + s;                  // concatenation calls toString()
        System.out.println(text);

        Student nothing = null;
        System.out.println("Printing null: " + nothing);   // String.valueOf handles null
    }
}
```

**Expected Output:**

```text
Default starts with class name? true
Part after @ equals hex hashCode? true
Student{name='Ravi', rollNo=101, cgpa=8.7}
Topper: Student{name='Ravi', rollNo=101, cgpa=8.7}
Printing null: null
```

**Explanation:**
- The default `toString()` is `getClass().getName() + "@" + Integer.toHexString(hashCode())`. For a nested class the name includes `Outer$Inner`. The hash part changes every run, so the program checks the format instead of printing it.
- `System.out.println(obj)` calls `String.valueOf(obj)`, which calls `obj.toString()` — or prints `"null"` for a null reference (no exception).
- **Why override `toString()`?** Logging, debugging and error messages become readable. IDEs can generate it; records generate it automatically.

---

## 3. `equals()`

### 3.1 Interview Definition

> **The `equals()` method checks whether two objects are logically equal. The default implementation in `Object` compares references, exactly like `==`, so it returns true only if both references point to the same object. Classes such as `String`, wrapper classes and collections override `equals()` to compare content. When we override `equals()`, we must follow its contract: it must be reflexive, symmetric, transitive, consistent, and return false when compared with `null`. Whenever `equals()` is overridden, `hashCode()` must also be overridden.**

### 3.2 The `equals()` Contract

For any non-null references `x`, `y`, `z`:

| Property | Rule | Meaning |
|---|---|---|
| **Reflexive** | `x.equals(x)` is `true` | An object equals itself |
| **Symmetric** | `x.equals(y)` ⇔ `y.equals(x)` | Order doesn't matter |
| **Transitive** | `x.equals(y)` and `y.equals(z)` ⇒ `x.equals(z)` | Equality chains |
| **Consistent** | Repeated calls give the same result if objects don't change | No randomness |
| **Non-null** | `x.equals(null)` is `false` | Never true for null, never throws |

### 3.3 How to Write a Correct `equals()` — Step by Step

1. **Same reference?** `if (this == o) return true;` (fast path, satisfies reflexive)
2. **Null or different class?** `if (o == null || getClass() != o.getClass()) return false;`
3. **Cast** `o` to your type.
4. **Compare significant fields**: primitives with `==` (for `double`/`float` use `Double.compare`), objects with `Objects.equals(a, b)` (null-safe).

---

## 4. `hashCode()`

### 4.1 Interview Definition

> **The `hashCode()` method returns an integer hash value for an object. It is used by hash-based collections such as `HashMap`, `HashSet` and `Hashtable` to decide which bucket an object should be stored in, which makes lookups fast. The contract between `equals()` and `hashCode()` states that if two objects are equal according to `equals()`, they must return the same hash code; however, two objects with the same hash code are not necessarily equal (this is a collision). The hash code must remain consistent as long as the fields used in `equals()` do not change. If `equals()` is overridden without overriding `hashCode()`, hash-based collections will not work correctly.**

### 4.2 The equals–hashCode Contract

| Rule | Required? |
|---|---|
| If `a.equals(b)` is true → `a.hashCode() == b.hashCode()` | ✅ **Must** |
| If `a.hashCode() == b.hashCode()` → `a.equals(b)` | ❌ Not required (collisions are allowed) |
| If `a.equals(b)` is false → hash codes should *preferably* differ | Recommended (better performance) |
| `hashCode()` is consistent while the object's equality fields don't change | ✅ Must |

### 4.3 Why Does the Contract Matter? (How HashMap uses them)

```text
map.put(key, value):
  1. h = key.hashCode()         → choose bucket index from h
  2. search that bucket using key.equals(existingKey)
       found → replace value
       not found → add new entry

map.get(key):
  1. h = key.hashCode()         → go to that bucket ONLY
  2. compare with equals() inside that bucket
```

If two **equal** objects had **different** hash codes, `get()` would look in the **wrong bucket** and never find the entry, and a `HashSet` would store "duplicates". Details of HashMap internals: [Part 5, Chapter 6](../05-Collections-Framework/06-Hashing-and-HashMap-Internal-Working.md).

### 4.4 Real-World Example

Think of a **library** with shelves labelled by the **first letter of the book title** (hash code → shelf/bucket). To find "Java Basics", you go straight to shelf **J** (hash code), then compare titles one by one on that shelf (equals). If someone put a copy of "Java Basics" on shelf **K** (wrong hash code for an equal object), you would never find it, even though it is in the library.

### 4.5 Java Example — Broken vs Correct

```java
import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;
import java.util.Objects;
import java.util.Set;

public class EqualsHashCodeDemo {

    // Only equals() overridden - BROKEN for hash collections
    static class BrokenEmployee {
        final int id;
        final String name;
        BrokenEmployee(int id, String name) { this.id = id; this.name = name; }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            BrokenEmployee other = (BrokenEmployee) o;
            return id == other.id && Objects.equals(name, other.name);
        }
        // hashCode() NOT overridden → uses Object's identity-based hash code
    }

    // Both equals() and hashCode() overridden - CORRECT
    static class Employee {
        private final int id;
        private final String name;
        Employee(int id, String name) { this.id = id; this.name = name; }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;                                // 1. same reference
            if (o == null || getClass() != o.getClass()) return false; // 2. null / different class
            Employee other = (Employee) o;                             // 3. cast
            return id == other.id && Objects.equals(name, other.name); // 4. compare fields
        }

        @Override
        public int hashCode() {
            return Objects.hash(id, name);                             // same fields as equals()
        }

        @Override
        public String toString() { return "Employee(" + id + ", " + name + ")"; }
    }

    public static void main(String[] args) {
        BrokenEmployee b1 = new BrokenEmployee(1, "Asha");
        BrokenEmployee b2 = new BrokenEmployee(1, "Asha");
        System.out.println("Broken: b1.equals(b2) = " + b1.equals(b2));
        Set<BrokenEmployee> brokenSet = new HashSet<>();
        brokenSet.add(b1);
        brokenSet.add(b2);
        System.out.println("Broken: set size (expected 1) = " + brokenSet.size());
        System.out.println("Broken: set contains new equal object? "
                + brokenSet.contains(new BrokenEmployee(1, "Asha")));

        Employee e1 = new Employee(1, "Asha");
        Employee e2 = new Employee(1, "Asha");
        Employee e3 = new Employee(2, "Bala");
        System.out.println("Correct: e1 == e2 ? " + (e1 == e2));
        System.out.println("Correct: e1.equals(e2) ? " + e1.equals(e2));
        System.out.println("Correct: same hashCode ? " + (e1.hashCode() == e2.hashCode()));

        Set<Employee> set = new HashSet<>();
        set.add(e1);
        set.add(e2);                                                  // duplicate → ignored
        set.add(e3);
        System.out.println("Correct: set size = " + set.size());

        Map<Employee, String> projects = new HashMap<>();
        projects.put(e1, "Payments");
        System.out.println("Correct: lookup with equal key = " + projects.get(new Employee(1, "Asha")));

        System.out.println("equals(null) = " + e1.equals(null));
    }
}
```

### 4.6 Code Explanation

- `BrokenEmployee` overrides only `equals()`. `b1.equals(b2)` is `true`, but their hash codes come from `Object` and are (almost always) **different**, so `HashSet` puts them in different buckets → **two "equal" objects stored**, and `contains()` fails.
- `Employee` overrides both, using the **same fields** (`id`, `name`). Equal objects → equal hash codes → `HashSet` detects the duplicate and `HashMap.get()` finds the entry with a new but equal key.
- `Objects.equals(a, b)` is null-safe; `Objects.hash(...)` builds a hash from several fields (internally `31 * result + fieldHash`).
- `e1.equals(null)` returns `false` (contract), without throwing.

### 4.7 Expected Output

```text
Broken: b1.equals(b2) = true
Broken: set size (expected 1) = 2
Broken: set contains new equal object? false
Correct: e1 == e2 ? false
Correct: e1.equals(e2) ? true
Correct: same hashCode ? true
Correct: set size = 2
Correct: lookup with equal key = Payments
equals(null) = false
```

(The "Broken" results depend on identity hash codes, which are different for different objects in practice; that is the real-world behaviour you will see.)

### 4.8 Interview Answer — "Why must we override hashCode() when we override equals()?"

> "Because of the contract between them: if two objects are equal according to `equals()`, they must return the same hash code. Hash-based collections like `HashMap` and `HashSet` first use `hashCode()` to find the bucket, and only then use `equals()` inside that bucket. If I override only `equals()`, two logically equal objects will usually have different hash codes from the `Object` class, so they go into different buckets. Then a `HashSet` will store duplicates, and `HashMap.get()` with an equal key will return null. So whenever I override `equals()`, I also override `hashCode()` using the same fields — for example with `Objects.hash(id, name)`. The reverse is not required: two objects with the same hash code don't have to be equal; that's just a collision."

---

## 5. `getClass()` vs `instanceof` in `equals()`

| | `getClass() != o.getClass()` | `!(o instanceof Employee)` |
|---|---|---|
| Subclass object equal to parent object? | Never | Possible |
| Symmetry with subclasses | ✅ Safe | ⚠️ Can break symmetry if a subclass adds fields and overrides `equals` |
| Works with proxies (e.g., Hibernate) | ❌ Proxy class differs | ✅ |
| Recommended when | Class may be subclassed with extra fields | Class is `final`, or equality is defined at the parent level |

**Symmetry problem with `instanceof`:** if `Point.equals` uses `instanceof Point` and `ColorPoint extends Point` adds a colour and compares it, then `point.equals(colorPoint)` is `true` but `colorPoint.equals(point)` is `false` → **symmetry broken**. The safest approaches: use `getClass()`, or make the class `final`, or prefer composition.

Java 16 pattern matching makes the `instanceof` style shorter:

<!-- skip-check -->
```java
@Override
public boolean equals(Object o) {
    return o instanceof Employee e && id == e.id && Objects.equals(name, e.name);
}
```

**Records** (Java 16) generate correct `equals()`, `hashCode()` and `toString()` automatically:

```java
import java.util.HashSet;
import java.util.Set;

public class RecordEqualsDemo {
    record Point(int x, int y) { }

    public static void main(String[] args) {
        Point p1 = new Point(2, 3);
        Point p2 = new Point(2, 3);
        System.out.println(p1);
        System.out.println("equals: " + p1.equals(p2) + ", same hash: " + (p1.hashCode() == p2.hashCode()));
        Set<Point> points = new HashSet<>(Set.of(p1));
        System.out.println("contains equal point: " + points.contains(p2));
    }
}
```

**Expected Output:**

```text
Point[x=2, y=3]
equals: true, same hash: true
contains equal point: true
```

---

## 6. Mutable Keys — A Hidden Danger

```java
import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

public class MutableKeyDemo {
    static class Box {
        int size;
        Box(int size) { this.size = size; }
        @Override public boolean equals(Object o) {
            return o instanceof Box b && size == b.size;
        }
        @Override public int hashCode() { return Objects.hash(size); }
    }

    public static void main(String[] args) {
        Set<Box> boxes = new HashSet<>();
        Box box = new Box(10);
        boxes.add(box);
        box.size = 20;                                  // changes hashCode while inside the set!
        System.out.println("contains(box)? " + boxes.contains(box));
        System.out.println("size = " + boxes.size());
    }
}
```

**Expected Output:**

```text
contains(box)? false
size = 1
```

After changing `size`, the object's hash code changes, so the set looks in a **different bucket** and cannot find the object that is clearly inside it. **Rule:** fields used in `hashCode()`/`equals()` should not change while the object is a key in a hash collection — prefer **immutable keys** (like `String`, `Integer`, records).

---

## 7. `clone()` — Shallow vs Deep Copy

### 7.1 Interview Definition

> **`clone()` is a protected method of `Object` that creates and returns a field-by-field copy of an object without calling its constructor. To use it, a class must implement the marker interface `Cloneable`; otherwise `Object.clone()` throws `CloneNotSupportedException`. The default `clone()` performs a shallow copy: primitive fields are copied by value, but reference fields are copied as references, so the original and the clone share the same inner objects. A deep copy also copies the referenced objects, and must be implemented manually.**

### 7.2 Java Example

```java
import java.util.ArrayList;
import java.util.List;

public class CloneDemo {

    static class Address {
        String city;
        Address(String city) { this.city = city; }
    }

    static class Person implements Cloneable {
        String name;
        Address address;
        List<String> hobbies = new ArrayList<>();

        Person(String name, Address address) {
            this.name = name;
            this.address = address;
        }

        // Shallow copy using Object.clone()
        @Override
        public Person clone() {
            try {
                return (Person) super.clone();
            } catch (CloneNotSupportedException e) {
                throw new AssertionError(e);         // cannot happen: we implement Cloneable
            }
        }

        // Deep copy (manual)
        Person deepCopy() {
            Person copy = new Person(name, new Address(address.city));
            copy.hobbies = new ArrayList<>(hobbies);
            return copy;
        }

        // Copy constructor (often preferred over clone)
        Person(Person other) {
            this(other.name, new Address(other.address.city));
            this.hobbies = new ArrayList<>(other.hobbies);
        }
    }

    public static void main(String[] args) {
        Person original = new Person("Ravi", new Address("Hyderabad"));
        original.hobbies.add("Chess");

        Person shallow = original.clone();
        shallow.name = "Shallow";                     // String field reassigned → independent
        shallow.address.city = "Pune";                // shared Address object!
        shallow.hobbies.add("Cricket");               // shared List!
        System.out.println("After shallow clone change -> original city: "
                + original.address.city + ", hobbies: " + original.hobbies + ", name: " + original.name);

        Person deep = original.deepCopy();
        deep.address.city = "Chennai";
        deep.hobbies.add("Music");
        System.out.println("After deep copy change -> original city: "
                + original.address.city + ", hobbies: " + original.hobbies);

        Person copied = new Person(original);
        copied.address.city = "Delhi";
        System.out.println("After copy-constructor change -> original city: " + original.address.city);
        System.out.println("Clone is a different object? " + (shallow != original));
    }
}
```

**Expected Output:**

```text
After shallow clone change -> original city: Pune, hobbies: [Chess, Cricket], name: Ravi
After deep copy change -> original city: Pune, hobbies: [Chess, Cricket]
After copy-constructor change -> original city: Pune
Clone is a different object? true
```

**Explanation:**
- **Shallow copy:** `super.clone()` copies the `address` and `hobbies` **references**. Changing the clone's address city or hobbies changes the original's too, because they share those objects. Re-assigning `shallow.name` does not affect the original because it changes only the clone's reference.
- **Deep copy:** creates a new `Address` and a new list, so changes to `deep` do not affect `original` (the city stays "Pune", from the earlier shallow change).
- **Copy constructor:** same idea as deep copy, without `Cloneable` or exceptions.

### 7.3 Why `clone()` Is Considered Problematic

1. `Cloneable` has no `clone()` method — a strange design (marker interface changes `Object.clone()` behaviour).
2. Constructors are not called, so invariants set in constructors can be skipped.
3. It does a shallow copy by default — easy to forget deep copying.
4. It does not work well with `final` fields that need new values.
5. Checked `CloneNotSupportedException` must be handled.

**Preferred alternatives:** copy constructors (`new ArrayList<>(list)`), static factory methods (`List.copyOf()`), or immutable objects that don't need copying.

---

## 8. `finalize()` — Deprecated

`finalize()` was called by the garbage collector before reclaiming an object, intended for cleanup. Problems: no guarantee it ever runs, unpredictable timing, performance cost, and it can resurrect objects. It is **deprecated since Java 9** and **deprecated for removal since Java 18**. Use **try-with-resources** (`AutoCloseable`) or `java.lang.ref.Cleaner` instead.

---

## 9. Interview Questions with Complete Answers

### Basic Level

**Q1. What is the default implementation of `equals()`?**

`Object.equals()` returns `this == obj` — it compares **references**, so it is true only if both refer to the same object. Classes like `String`, `Integer`, `ArrayList` and `HashMap` override it to compare content.

**Q2. What does the default `toString()` return?**

`getClass().getName() + "@" + Integer.toHexString(hashCode())`, for example `com.app.Student@7a81197d`.

**Q3. What is the difference between `==` and `equals()`?**

`==` compares references for objects (and values for primitives). `equals()` is a method that, when overridden, compares logical content. `new String("a") == new String("a")` is false; `.equals()` is true.

### Intermediate Level

**Q4. What happens if we override `hashCode()` but not `equals()`?**

Equal-looking objects get the same bucket, but `Object.equals()` still compares references, so they are treated as different. A `HashSet` would still store both "duplicates" (in the same bucket), and `map.get(newEqualKey)` would return `null`. Both must be overridden together.

**Q5. Can two unequal objects have the same hash code?**

Yes. That is a **collision**, and it is allowed. For example, `"Aa".hashCode() == "BB".hashCode()` (both 2112). Hash collections handle collisions by storing multiple entries in a bucket (linked list, or a balanced tree when a bucket becomes large) and using `equals()` to distinguish them.

**Q6. Which fields should be used in `equals()` and `hashCode()`?**

The same set of **significant fields** that define logical identity (e.g., `id`, or `id + name`). Never use a field in `hashCode()` that is not used in `equals()`. Avoid mutable fields if the object will be used as a key. Do not include derived or cached values.

**Q7. What is a good `hashCode()` implementation?**

Combine the hash codes of the significant fields with a multiplier (31): `result = 31 * result + fieldHash`. `Objects.hash(a, b, c)` does this for you (slightly slower because it creates an array). For a single `int` id, `Integer.hashCode(id)` or `id` itself is fine. A constant `return 1;` satisfies the contract but turns a `HashMap` into a slow list — **O(n)** lookups.

### Advanced Level

**Q8. Why is 31 used in hash code calculations?**

31 is an odd prime, which helps spread values evenly and reduces collisions. Multiplying by 31 can be optimised by the JVM as `(i << 5) - i` (a shift and a subtraction). It is also the number used by `String.hashCode()`.

**Q9. Is `hashCode()` the memory address of an object?**

Not necessarily. The default (identity) hash code is generated by the JVM and stored in the object header; in HotSpot it is typically a random-like number generated on first call, not the actual address (objects move during GC, but the hash code must not change). `System.identityHashCode(obj)` returns this value even if `hashCode()` is overridden.

**Q10. How do you write `equals()` for a class with a `double` field?**

Use `Double.compare(a, b) == 0` instead of `a == b`. This treats `NaN` as equal to itself and distinguishes `0.0` and `-0.0` consistently with `Double.equals()`, keeping the contract (reflexivity for NaN). In `hashCode()`, use `Double.hashCode(value)`.

**Q11. Why does `clone()` not call the constructor?**

`Object.clone()` is a native method that allocates memory and copies the object's bytes (fields) directly. It bypasses constructors, which is one reason it is considered risky — any initialisation logic or validation in constructors is skipped.

---

## 10. Follow-Up Questions

**F1. Can `equals()` be overloaded instead of overridden? What happens?**

Writing `public boolean equals(Employee e)` **overloads** it. Collections call `equals(Object)`, so your method is never used by `HashSet`, `ArrayList.contains()`, etc. Always use the signature `equals(Object o)` with `@Override`.

**F2. If `equals()` is true, must `compareTo()` return 0?**

It is **strongly recommended** ("consistent with equals"), but not required. If inconsistent, sorted collections like `TreeSet` (which use `compareTo`) and hash collections (which use `equals`) will disagree about duplicates. `BigDecimal` is a famous example: `new BigDecimal("1.0").equals(new BigDecimal("1.00"))` is false but `compareTo` returns 0.

**F3. Why does `String` cache its hash code?**

Strings are immutable, so the hash never changes. Computing it requires looping over all characters; caching after the first call makes repeated `HashMap` lookups with the same string key very fast.

---

## 11. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Overriding `equals()` without `hashCode()` | Hash collections break. |
| `equals(MyClass o)` signature | Overloads, not overrides; use `equals(Object o)`. |
| Using `==` for `String`/object fields inside `equals()` | Use `Objects.equals()`. |
| Different fields in `equals()` and `hashCode()` | Violates the contract. |
| Mutating a key inside a `HashSet`/`HashMap` | Object becomes unreachable by lookup. |
| Thinking `clone()` deep copies | Default is shallow. |
| Using `finalize()` for cleanup | Deprecated; use try-with-resources. |

---

## 12. Quick Revision

- `Object` = root of all classes; methods: `toString, equals, hashCode, getClass, clone, finalize, wait, notify, notifyAll`.
- Default `toString()` → `ClassName@hexHash`; override for readable output.
- Default `equals()` → `==`. Contract: **reflexive, symmetric, transitive, consistent, non-null**.
- equals template: same ref → null/class check → cast → compare fields (`Objects.equals`).
- **equals true ⇒ same hashCode** (must). Same hashCode ⇏ equals (collision).
- Override both, using the same fields: `Objects.hash(...)`.
- Hash collections: `hashCode()` → bucket, `equals()` → match.
- Mutable keys break lookups. Records auto-generate all three methods.
- `clone()` → needs `Cloneable`, shallow copy, no constructor call; prefer copy constructors.
- `finalize()` → deprecated.

---

[← Previous: static and final](09-static-and-final.md) | [Back to README](../README.md) | [Next: Association, Aggregation, Composition →](11-Association-Aggregation-Composition.md)
