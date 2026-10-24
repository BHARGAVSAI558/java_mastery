# Chapter 6.2 — Bounded Types, Wildcards (extends and super) and Type Erasure

[← Previous: Generic Classes, Methods, Interfaces](01-Generic-Classes-Methods-Interfaces.md) | [Back to README](../README.md) | [Next Part: Multithreading →](../07-Multithreading-and-Concurrency/01-Process-Thread-and-Lifecycle.md)

---

## What You Will Learn

- Bounded type parameters: `<T extends Number>`, multiple bounds
- Invariance of generics — why `List<Integer>` is not a `List<Number>`
- Wildcards: unbounded `<?>`, upper-bounded `<? extends T>`, lower-bounded `<? super T>`
- **PECS — Producer Extends, Consumer Super**
- Type erasure — what happens at compile time and runtime, bridge methods
- Interview questions on all of these

---

## 1. Bounded Type Parameters

### 1.1 Interview Definition

> **A bounded type parameter restricts the types that can be used as type arguments. An upper bound, written `<T extends Number>`, means that `T` must be `Number` or one of its subclasses; this also allows the generic code to call the methods of the bound, such as `doubleValue()`. The keyword `extends` is used for both classes and interfaces in bounds, and multiple bounds can be combined with `&`, as in `<T extends Number & Comparable<T>>`, where a class bound, if present, must come first. Bounded types make generic code both more flexible and safer.**

### 1.2 Understand the Concept

Without a bound, `T` is treated like `Object`, so you can only call `Object` methods (`toString`, `equals`). With `<T extends Number>`, the compiler knows every `T` **is a** `Number`, so `t.doubleValue()` is allowed — and `Box<String>` is rejected.

### 1.3 Java Example

```java
import java.util.List;

public class BoundedTypeDemo {

    // Upper-bounded generic class
    static class Statistics<T extends Number> {
        private final List<T> values;
        Statistics(List<T> values) { this.values = values; }

        double average() {
            double sum = 0;
            for (T v : values) {
                sum += v.doubleValue();            // allowed because T extends Number
            }
            return values.isEmpty() ? 0 : sum / values.size();
        }
    }

    // Bounded generic method: T must be Comparable to itself
    static <T extends Comparable<T>> T findMax(List<T> list) {
        T max = list.get(0);
        for (T item : list) {
            if (item.compareTo(max) > 0) {         // allowed because T extends Comparable<T>
                max = item;
            }
        }
        return max;
    }

    // Multiple bounds: class first, then interfaces, joined with &
    static <T extends Number & Comparable<T>> T clampToMax(T value, T max) {
        return value.compareTo(max) > 0 ? max : value;
    }

    public static void main(String[] args) {
        System.out.println("Average of ints   : " + new Statistics<>(List.of(10, 20, 30)).average());
        System.out.println("Average of doubles: " + new Statistics<>(List.of(1.5, 2.5)).average());
        // new Statistics<>(List.of("a", "b"));  // compile error: String is not a Number

        System.out.println("Max int   : " + findMax(List.of(4, 19, 7)));
        System.out.println("Max string: " + findMax(List.of("pear", "apple", "zebra")));
        System.out.println("Clamp 150 to 100: " + clampToMax(150, 100));
        System.out.println("Clamp 2.5 to 9.0: " + clampToMax(2.5, 9.0));
    }
}
```

**Expected Output:**

```text
Average of ints   : 20.0
Average of doubles: 2.0
Max int   : 19
Max string: zebra
Clamp 150 to 100: 100
Clamp 2.5 to 9.0: 2.5
```

**Explanation:**
- `Statistics<T extends Number>` accepts `Integer`, `Double`, `Long`, `BigDecimal`… but not `String`.
- `<T extends Comparable<T>>` lets `findMax` call `compareTo` on any comparable type — the JDK uses the more flexible form `<T extends Comparable<? super T>>` (explained with PECS below).
- `<T extends Number & Comparable<T>>` requires both; `Integer` and `Double` satisfy both.

---

## 2. Invariance — The Reason Wildcards Exist

```text
Integer IS-A Number         ✅
Integer[] IS-A Number[]     ✅  (arrays are covariant — checked at runtime)
List<Integer> IS-A List<Number>  ❌  (generics are INVARIANT)
```

<!-- skip-check -->
```java
List<Integer> ints = new ArrayList<>();
List<Number> nums = ints;         // COMPILE ERROR: incompatible types
nums.add(3.14);                   // (if allowed, a Double would enter a List<Integer>!)
```

So a method `double sum(List<Number> list)` **cannot** accept a `List<Integer>`. Wildcards solve this.

---

## 3. Wildcards

### 3.1 Interview Definition

> **A wildcard, written `?`, represents an unknown type in a generic type argument. Java has three kinds: the unbounded wildcard `<?>`, which accepts any type; the upper-bounded wildcard `<? extends T>`, which accepts `T` or any subtype of `T` and is used when a structure is only read from (a producer); and the lower-bounded wildcard `<? super T>`, which accepts `T` or any supertype of `T` and is used when a structure is written to (a consumer). Wildcards make generic method parameters flexible while keeping type safety, and are mainly used in method parameters rather than in class declarations.**

### 3.2 Summary Table

| Wildcard | Accepts | Can read as | Can add | Typical use |
|---|---|---|---|---|
| `List<?>` | Any `List<X>` | `Object` | Only `null` | Size, printing, `clear()` — type-independent operations |
| `List<? extends Number>` | `List<Number>`, `List<Integer>`, `List<Double>`… | `Number` | ❌ Nothing (except `null`) | **Reading / producer** |
| `List<? super Integer>` | `List<Integer>`, `List<Number>`, `List<Object>` | `Object` only | ✅ `Integer` (and subtypes) | **Writing / consumer** |

**Why can't you add to `List<? extends Number>`?** The actual list might be a `List<Double>`; adding an `Integer` would corrupt it. The compiler doesn't know the exact type, so it forbids adding.

**Why can you add `Integer` to `List<? super Integer>`?** Whatever the actual list is (`List<Integer>`, `List<Number>` or `List<Object>`), an `Integer` fits into it. But reading gives only `Object`, because you don't know which supertype it is.

### 3.3 PECS — Producer Extends, Consumer Super

> **PECS (from *Effective Java* by Joshua Bloch): if a parameterised parameter *produces* values that your method reads, use `? extends T`; if it *consumes* values that your method writes into it, use `? super T`. If it does both, use the exact type `T`.**

Example from the JDK: `Collections.copy(List<? super T> dest, List<? extends T> src)` — the source **produces** elements (extends), the destination **consumes** them (super).

### 3.4 Real-World Example

- **`? extends Fruit`** is like a **fruit basket you can only take from**: whether it's a basket of apples or of mangoes, you can always take out "a fruit". But you can't put a banana into it — it might be the apple-only basket.
- **`? super Apple`** is like a **donation box for apples**: it might be an "apples" box, a "fruits" box or an "anything" box — putting an apple in is always fine. But when you take something out, you only know it's "something".

### 3.5 Java Example

```java
import java.util.ArrayList;
import java.util.List;

public class WildcardDemo {

    // Unbounded: works with any list, only uses Object-level operations
    static void printAll(List<?> list) {
        for (Object o : list) {
            System.out.print(o + " ");
        }
        System.out.println("(size " + list.size() + ")");
    }

    // Upper-bounded: PRODUCER — we only read Numbers
    static double sum(List<? extends Number> numbers) {
        double total = 0;
        for (Number n : numbers) {
            total += n.doubleValue();
        }
        // numbers.add(5);        // compile error: cannot add to ? extends
        return total;
    }

    // Lower-bounded: CONSUMER — we only write Integers
    static void addNumbers(List<? super Integer> target, int count) {
        for (int i = 1; i <= count; i++) {
            target.add(i);             // allowed: Integer fits any supertype list
        }
        Object first = target.get(0);  // reading gives only Object
        System.out.println("First element read as Object: " + first);
    }

    // PECS in practice: copy from a producer into a consumer
    static <T> void copy(List<? extends T> source, List<? super T> destination) {
        for (T item : source) {
            destination.add(item);
        }
    }

    public static void main(String[] args) {
        List<Integer> ints = List.of(1, 2, 3);
        List<Double> doubles = List.of(1.5, 2.5);
        List<String> words = List.of("a", "b");

        printAll(ints);
        printAll(words);

        System.out.println("sum(ints)    = " + sum(ints));        // List<Integer> accepted
        System.out.println("sum(doubles) = " + sum(doubles));     // List<Double> accepted

        List<Number> numberList = new ArrayList<>();
        List<Object> objectList = new ArrayList<>();
        addNumbers(numberList, 3);                                 // List<Number> accepted
        addNumbers(objectList, 2);                                 // List<Object> accepted
        System.out.println("numberList = " + numberList + ", objectList = " + objectList);

        List<Object> everything = new ArrayList<>(List.of("start"));
        copy(ints, everything);                                    // T = Integer
        copy(doubles, everything);                                 // T = Double
        System.out.println("everything = " + everything);
    }
}
```

**Expected Output:**

```text
1 2 3 (size 3)
a b (size 2)
sum(ints)    = 6.0
sum(doubles) = 4.0
First element read as Object: 1
First element read as Object: 1
numberList = [1, 2, 3], objectList = [1, 2]
everything = [start, 1, 2, 3, 1.5, 2.5]
```

### 3.6 Code Explanation

- `printAll(List<?>)` accepts lists of any type but treats elements as `Object`.
- `sum(List<? extends Number>)` accepts `List<Integer>` and `List<Double>` — this is impossible with `List<Number>` because of invariance. Inside, reading as `Number` is safe; adding is forbidden.
- `addNumbers(List<? super Integer>)` accepts `List<Integer>`, `List<Number>` and `List<Object>`; adding `Integer`s is safe; reading gives only `Object`.
- `copy` combines both: read from a producer (`extends`), write into a consumer (`super`).

### 3.7 Interview Answer — "Explain `? extends` and `? super`"

> "Generics are invariant, so a `List<Integer>` is not a `List<Number>`. Wildcards give flexibility. `? extends T` is an upper bound — it accepts a list of T or any subtype. I can safely **read** elements as T, but I can't add to it, because I don't know the exact type — it might be a list of Doubles. So I use it when the parameter is a **producer**, like a `sum(List<? extends Number>)` method. `? super T` is a lower bound — it accepts a list of T or any supertype. I can safely **add** T values to it, but when I read, I only get `Object`. So I use it when the parameter is a **consumer**, like a method that adds integers into a list. The rule is PECS: Producer Extends, Consumer Super — for example, `Collections.copy(List<? super T> dest, List<? extends T> src)`. And `<?>` alone means any type when I only need type-independent operations like size or printing."

---

## 4. Type Erasure

### 4.1 Interview Definition

> **Type erasure is the process by which the Java compiler removes generic type information after checking types at compile time. Type parameters are replaced by their bound — `Object` for unbounded parameters, or the first bound for bounded ones — type arguments such as `<String>` are removed, and the compiler inserts casts where needed and generates bridge methods to preserve polymorphism. As a result, at runtime `List<String>` and `List<Integer>` are the same class, `List`. Type erasure was chosen for backward compatibility with pre-Java 5 code and libraries, and it is the reason why we cannot create `new T()`, generic arrays, or use `instanceof` with parameterised types.**

### 4.2 What the Compiler Does

```text
Source code (what you write)                 After erasure (what the JVM sees)
---------------------------------            ---------------------------------
class Box<T> {                               class Box {
    T value;                                     Object value;
    T get() { return value; }                    Object get() { return value; }
}                                            }

class NumBox<T extends Number> {             class NumBox {
    T value;                                     Number value;          // first bound
}                                            }

Box<String> b = new Box<>();                 Box b = new Box();
String s = b.get();                          String s = (String) b.get();   // cast inserted
```

### 4.3 Java Example — Erasure at Runtime

```java
import java.util.ArrayList;
import java.util.List;

public class TypeErasureDemo {
    static class Box<T> {
        T value;
        Box(T value) { this.value = value; }
    }

    @SuppressWarnings("unchecked")
    public static void main(String[] args) throws Exception {
        List<String> strings = new ArrayList<>();
        List<Integer> integers = new ArrayList<>();
        System.out.println("Same runtime class? " + (strings.getClass() == integers.getClass()));
        System.out.println("Runtime class name: " + strings.getClass().getName());

        // The field type after erasure is Object
        System.out.println("Box.value field type: " + Box.class.getDeclaredField("value").getType().getSimpleName());

        // instanceof with type arguments is not allowed, wildcard is fine
        Object obj = strings;
        System.out.println("obj instanceof List<?> : " + (obj instanceof List<?>));
        // obj instanceof List<String>   → compile error: illegal generic type for instanceof

        // Heap pollution through a raw type: erasure means no runtime check at add time
        List raw = strings;
        raw.add(99);                                     // compiles with a warning, runs fine!
        System.out.println("strings contains: " + strings);
        try {
            String first = strings.get(0);               // compiler-inserted cast fails HERE
        } catch (ClassCastException e) {
            System.out.println("ClassCastException when reading, not when adding");
        }
    }
}
```

**Expected Output:**

```text
Same runtime class? true
Runtime class name: java.util.ArrayList
Box.value field type: Object
obj instanceof List<?> : true
strings contains: [99]
ClassCastException when reading, not when adding
```

**Explanation:**
- Both lists are plain `ArrayList` at runtime — the `<String>`/`<Integer>` information is gone.
- The field `T value` became `Object value`.
- Through a raw reference, an `Integer` entered a `List<String>`. Nothing checked it at `add()` time (no type info at runtime). The failure appears later, at the **cast the compiler inserted** when reading. This is **heap pollution**.

### 4.4 Bridge Methods

When a class implements a generic interface with a concrete type, erasure would break overriding. The compiler generates a **bridge method**:

<!-- skip-check -->
```java
class Student implements Comparable<Student> {
    public int compareTo(Student other) { ... }        // what you write

    // generated by the compiler (synthetic bridge method):
    // public int compareTo(Object other) { return compareTo((Student) other); }
}
```

The erased interface method is `compareTo(Object)`; the bridge forwards calls to your `compareTo(Student)`, so polymorphism works.

### 4.5 Why Type Erasure? Consequences

| Reason | Consequence |
|---|---|
| **Backward compatibility**: Java 5 generic code had to run on old JVMs and interoperate with old non-generic libraries (migration compatibility) | No generic type information at runtime for objects |
| Single class per generic type (no code duplication like C++ templates) | `List<String>.class` doesn't exist |
| | Can't `new T()`, `new T[]`, `instanceof List<String>` |
| | Can't overload `m(List<String>)` and `m(List<Integer>)` — same erasure |
| | Primitives can't be type arguments (erasure to `Object`) |

**Note:** Some generic information **is** kept in class files as metadata (signatures of fields, method parameters, superclass types) and can be read via reflection (`getGenericType()`, `getGenericSuperclass()`). Frameworks like Jackson and Spring use this (e.g., the "super type token" `new TypeReference<List<User>>() {}`). What is erased is the type argument of **objects** at runtime.

---

## 5. Interview Questions with Complete Answers

### Basic Level

**Q1. What is a bounded type parameter?**

A type parameter with a restriction, such as `<T extends Number>`, meaning only `Number` or its subclasses are allowed. It lets generic code call the bound's methods (`doubleValue()`). Multiple bounds: `<T extends Number & Comparable<T>>` (class first).

**Q2. What is a wildcard?**

`?` in a type argument, meaning "unknown type". Forms: `<?>` (any), `<? extends T>` (T or subtypes), `<? super T>` (T or supertypes).

**Q3. Is `List<Integer>` a subtype of `List<Number>`?**

No — generics are invariant. But `List<Integer>` is a subtype of `List<? extends Number>`.

### Intermediate Level

**Q4. What is PECS?**

"Producer Extends, Consumer Super". If a generic parameter supplies values you read, declare it `? extends T`. If it receives values you write, declare it `? super T`. Example: `Collections.copy(List<? super T> dest, List<? extends T> src)` and `Collections.max(Collection<? extends T> coll)`.

**Q5. Why can't we add elements to a `List<? extends Number>`?**

Because the actual list might be `List<Integer>`, `List<Double>`, or another subtype. The compiler can't guarantee that what you add matches the real type, so it forbids adding anything except `null`.

**Q6. What is the difference between `<T extends Number>` and `<? extends Number>`?**

`<T extends Number>` declares a **named type parameter** (on a class or method) that can be used multiple times to relate types — e.g., `<T extends Number> T max(T a, T b)` ensures the return type equals the argument type. `<? extends Number>` is a **wildcard** used in a parameter or variable type when you don't need to refer to the type again. Wildcards can also have `super` bounds; type parameters cannot.

### Advanced Level

**Q7. What is type erasure, and why did Java choose it?**

The compiler removes generic type arguments after type-checking, replaces type parameters with their bounds (or `Object`), inserts casts and bridge methods. Java chose it so that generic code would be **binary-compatible** with existing pre-Java 5 code and JVMs — old libraries using raw `List` could work with new generic code without recompilation.

**Q8. Why can't we create a generic array like `new T[10]`?**

Arrays need their exact component type at runtime (to throw `ArrayStoreException`), but `T` is erased. The JVM cannot create a `T[]`. Workarounds: use a `List<T>`, pass a `Class<T>` and use `Array.newInstance(clazz, n)`, or cast `(T[]) new Object[n]` carefully inside the class (as `ArrayList` does internally with `Object[]`).

**Q9. Why does `Collections.max` have the signature `<T extends Object & Comparable<? super T>> T max(Collection<? extends T> coll)`?**

- `Comparable<? super T>` allows types whose comparison is defined in a superclass (e.g., `java.sql.Timestamp` or a subclass inheriting `compareTo` from its parent).
- `Collection<? extends T>` (producer) accepts collections of subtypes.
- `T extends Object & ...` keeps the **erased** return type as `Object`, preserving binary compatibility with the pre-generics signature.

**Q10. Can you overload `void print(List<String> l)` and `void print(List<Integer> l)`?**

No. Both erase to `print(List)`, which gives the compile error "name clash: both methods have same erasure".

---

## 6. Follow-Up Questions

**F1. Where would you use `<?>` in real code?**

Utility methods that don't care about the element type: `void printSize(Collection<?> c)`, `boolean isEmpty(Map<?, ?> m)`, `Class<?> type = obj.getClass()`.

**F2. How do frameworks get the generic type at runtime if it's erased?**

They read generic signatures stored in class metadata through reflection — e.g., an anonymous subclass `new TypeReference<List<User>>() {}` preserves `List<User>` in its superclass signature, which Jackson reads with `getGenericSuperclass()`.

**F3. What are reifiable types?**

Types whose full information is available at runtime: primitives, non-generic types, raw types, and unbounded wildcard types like `List<?>`. Only reifiable types can be used in `instanceof` and as array component types.

---

## 7. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Using `List<Number>` as a parameter to accept `List<Integer>` | Use `List<? extends Number>`. |
| Trying to add to `? extends` collections | Not allowed; use `? super` for writing. |
| `<T super Number>` | `super` is only for wildcards, not type parameters. |
| Thinking generics exist at runtime | Erased; only compile-time checks. |
| Using raw types with generics | Causes heap pollution. |
| Overloading methods differing only in type arguments | Name clash after erasure. |

---

## 8. Quick Revision

- Bounded type: `<T extends Number>`; multiple bounds `<T extends A & B & C>` (class first).
- Generics are **invariant**: `List<Integer>` ≠ `List<Number>`.
- `<?>` any (read Object, no add) · `<? extends T>` read T, no add (producer) · `<? super T>` add T, read Object (consumer).
- **PECS**: Producer Extends, Consumer Super (`Collections.copy`).
- **Type erasure**: parameters → bound/`Object`, casts inserted, bridge methods; for backward compatibility.
- Consequences: no `new T()`, no generic arrays, no `instanceof List<String>`, no primitive type args, erasure name clashes, heap pollution via raw types.

---

[← Previous: Generic Classes, Methods, Interfaces](01-Generic-Classes-Methods-Interfaces.md) | [Back to README](../README.md) | [Next Part: Multithreading →](../07-Multithreading-and-Concurrency/01-Process-Thread-and-Lifecycle.md)
