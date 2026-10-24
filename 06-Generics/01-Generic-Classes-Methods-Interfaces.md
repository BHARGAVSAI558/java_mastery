# Chapter 6.1 — Generic Classes, Methods and Interfaces

[← Previous Part: Collections Questions](../05-Collections-Framework/09-Collections-Comparisons-and-Interview-Questions.md) | [Back to README](../README.md) | [Next: Bounded Types, Wildcards, Type Erasure →](02-Bounded-Types-Wildcards-Type-Erasure.md)

---

## What You Will Learn

- What generics are and the problems they solve
- Type parameters and naming conventions (`T`, `E`, `K`, `V`, `R`)
- Generic classes (single and multiple type parameters)
- Generic methods (including static generic methods) and type inference
- Generic interfaces and how to implement them
- The diamond operator `<>` (Java 7)
- Raw types and why to avoid them
- Restrictions of generics (preview of type erasure)

---

## 1. Generics

### 1.1 Interview Definition

> **Generics, introduced in Java 5, allow classes, interfaces and methods to be written with type parameters, so the same code can work with different data types while still being checked by the compiler. A type parameter, such as `T` in `class Box<T>`, is a placeholder that is replaced by an actual type, such as `Box<String>`, when the class is used. Generics provide compile-time type safety, eliminate the need for explicit casting, and enable reusable, general-purpose code such as the Collections Framework. At runtime, generic type information is mostly removed by type erasure.**

### 1.2 The Problem Before Generics

```java
import java.util.ArrayList;
import java.util.List;

public class BeforeGenerics {
    @SuppressWarnings({"rawtypes", "unchecked"})
    public static void main(String[] args) {
        List names = new ArrayList();        // raw type: holds Object
        names.add("Ravi");
        names.add("Priya");
        names.add(42);                       // compiler allows anything!

        try {
            for (Object o : names) {
                String s = (String) o;       // manual cast needed
                System.out.println(s.toUpperCase());
            }
        } catch (ClassCastException e) {
            System.out.println("ClassCastException at RUNTIME: Integer is not a String");
        }
    }
}
```

**Expected Output:**

```text
RAVI
PRIYA
ClassCastException at RUNTIME: Integer is not a String
```

**Problems without generics:**
1. **No type safety** — any object can be added; mistakes are found only at runtime.
2. **Manual casting** everywhere — noisy and error-prone.
3. **Bugs appear far from where they were caused** — the wrong `add(42)` fails much later during reading.

**With generics:**

```java
import java.util.ArrayList;
import java.util.List;

public class WithGenerics {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>();
        names.add("Ravi");
        names.add("Priya");
        // names.add(42);                    // COMPILE ERROR: incompatible types: int cannot be converted to String

        for (String s : names) {             // no cast needed
            System.out.println(s.toUpperCase());
        }
    }
}
```

**Expected Output:**

```text
RAVI
PRIYA
```

### 1.3 Benefits of Generics

| Benefit | Explanation |
|---|---|
| **Compile-time type safety** | Wrong types are rejected by the compiler, not discovered at runtime |
| **No explicit casts** | The compiler inserts safe casts automatically |
| **Reusability** | One `Box<T>` class works for `String`, `Integer`, `Student`… |
| **Readability** | `Map<String, List<Order>>` documents exactly what is stored |
| **Generic algorithms** | Write `max(List<T>)` once for any comparable type |

### 1.4 Real-World Example

Generics are like **labelled storage boxes**. A box labelled "Books" accepts only books. You can't accidentally put shoes in it (compile-time check), and when you take something out, you know it is a book (no casting). The box design is the same for every label — only the label (type parameter) changes.

---

## 2. Type Parameter Naming Conventions

| Letter | Meaning | Example |
|---|---|---|
| `T` | Type | `Box<T>` |
| `E` | Element | `List<E>`, `Set<E>` |
| `K` | Key | `Map<K, V>` |
| `V` | Value | `Map<K, V>` |
| `N` | Number | `Calculator<N extends Number>` |
| `R` | Return type | `Function<T, R>` |
| `S`, `U` | 2nd, 3rd types | `BiFunction<T, U, R>` |

Single uppercase letters make type parameters easy to distinguish from real class names.

---

## 3. Generic Classes

### 3.1 Interview Definition

> **A generic class is a class declared with one or more type parameters in angle brackets after its name, such as `class Box<T>` or `class Pair<K, V>`. The type parameters can be used as the types of fields, method parameters and return types inside the class. When creating an object, the actual type arguments are supplied, such as `new Box<String>()` or with the diamond operator `new Box<>()`, and the compiler ensures that only values of that type are used with that object.**

### 3.2 Java Example

```java
public class GenericClassDemo {

    // Single type parameter
    static class Box<T> {
        private T content;

        void put(T item) { this.content = item; }
        T get() { return content; }
        boolean isEmpty() { return content == null; }
        @Override public String toString() { return "Box[" + content + "]"; }
    }

    // Multiple type parameters
    static class Pair<K, V> {
        private final K key;
        private final V value;

        Pair(K key, V value) { this.key = key; this.value = value; }
        K getKey() { return key; }
        V getValue() { return value; }
        @Override public String toString() { return "(" + key + ", " + value + ")"; }
    }

    // Generic class with a generic field type used in logic
    static class Result<T> {
        private final T data;
        private final String error;

        private Result(T data, String error) { this.data = data; this.error = error; }
        static <T> Result<T> success(T data) { return new Result<>(data, null); }
        static <T> Result<T> failure(String error) { return new Result<>(null, error); }
        boolean isSuccess() { return error == null; }
        T getData() { return data; }
        String getError() { return error; }
    }

    static Result<Integer> parseAge(String text) {
        try {
            int age = Integer.parseInt(text);
            return age >= 0 ? Result.success(age) : Result.failure("Age cannot be negative");
        } catch (NumberFormatException e) {
            return Result.failure("Not a number: " + text);
        }
    }

    public static void main(String[] args) {
        Box<String> stringBox = new Box<>();          // diamond operator: type inferred
        stringBox.put("Java");
        String s = stringBox.get();                   // no cast
        System.out.println(stringBox + " length " + s.length());

        Box<Integer> intBox = new Box<>();
        intBox.put(21);
        int doubled = intBox.get() * 2;               // auto-unboxing, no cast
        System.out.println(intBox + " doubled " + doubled);
        // intBox.put("text");                        // compile error

        Pair<String, Integer> score = new Pair<>("Ravi", 92);
        Pair<Integer, Pair<String, Boolean>> nested = new Pair<>(1, new Pair<>("active", true));
        System.out.println(score + " key=" + score.getKey() + " value=" + score.getValue());
        System.out.println("nested: " + nested);

        for (String input : new String[]{"25", "-3", "abc"}) {
            Result<Integer> r = parseAge(input);
            System.out.println(input + " -> " + (r.isSuccess() ? "age " + r.getData() : "error: " + r.getError()));
        }
    }
}
```

### 3.3 Code Explanation

- `Box<T>`: `T` is a placeholder. `Box<String>` makes `put(String)` and `String get()`; `Box<Integer>` makes `put(Integer)` and `Integer get()`.
- The **diamond operator** `new Box<>()` (Java 7) lets the compiler infer the type argument from the variable's declared type.
- `Pair<K, V>` has two type parameters; type arguments can themselves be generic (`Pair<String, Boolean>`).
- Type arguments must be **reference types** — `Box<int>` is illegal; use `Box<Integer>` (autoboxing handles conversion).
- `Result<T>` is a practical generic class that wraps either data or an error. Its static factory methods `success` and `failure` are **generic methods** (they declare their own `<T>`).

### 3.4 Expected Output

```text
Box[Java] length 4
Box[21] doubled 42
(Ravi, 92) key=Ravi value=92
nested: (1, (active, true))
25 -> age 25
-3 -> error: Age cannot be negative
abc -> error: Not a number: abc
```

---

## 4. Generic Methods

### 4.1 Interview Definition

> **A generic method is a method that declares its own type parameters, written in angle brackets before the return type, such as `public static <T> void printAll(List<T> items)`. The type parameter is local to that method and can be used in its parameters, return type and body. Generic methods can be static or instance methods and can exist in non-generic classes. The compiler usually infers the type argument from the method arguments or the assignment target, but it can also be given explicitly, as in `Util.<String>printAll(list)`.**

### 4.2 Java Example

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class GenericMethodDemo {

    // Generic method in a non-generic class
    static <T> void printArray(T[] items) {
        for (T item : items) {
            System.out.print(item + " ");
        }
        System.out.println();
    }

    // Generic return type
    static <T> T firstOrDefault(List<T> list, T defaultValue) {
        return list.isEmpty() ? defaultValue : list.get(0);
    }

    // Two type parameters
    static <K, V> String describe(K key, V value) {
        return key.getClass().getSimpleName() + "=" + key + " -> " + value.getClass().getSimpleName() + "=" + value;
    }

    // Generic swap for any array type
    static <T> void swap(T[] arr, int i, int j) {
        T temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }

    // Generic factory method
    @SafeVarargs
    static <T> List<T> listOf(T... elements) {
        return new ArrayList<>(Arrays.asList(elements));
    }

    public static void main(String[] args) {
        Integer[] numbers = {1, 2, 3};
        String[] words = {"Java", "is", "fun"};
        printArray(numbers);                                   // T inferred as Integer
        printArray(words);                                     // T inferred as String

        System.out.println(firstOrDefault(List.of(), "none"));
        System.out.println(firstOrDefault(List.of(10, 20), 0));

        System.out.println(describe("age", 21));
        System.out.println(describe(101, 8.5));

        swap(words, 0, 2);
        System.out.println(Arrays.toString(words));

        List<String> skills = listOf("Java", "SQL");
        skills.add("Spring");
        System.out.println(skills);

        List<Double> explicit = GenericMethodDemo.<Double>listOf(1.5, 2.5);   // explicit type argument
        System.out.println(explicit);
    }
}
```

### 4.3 Expected Output

```text
1 2 3 
Java is fun 
none
10
String=age -> Integer=21
Integer=101 -> Double=8.5
[fun, is, Java]
[Java, SQL, Spring]
[1.5, 2.5]
```

### 4.4 Code Explanation

- `<T>` before `void` declares the type parameter for **this method only**.
- **Type inference:** for `printArray(numbers)`, the compiler sees `Integer[]` and infers `T = Integer`.
- `firstOrDefault(List.of(), "none")` infers `T = String` from the default value.
- `<K, V>` allows two independent types.
- `@SafeVarargs` suppresses the "possible heap pollution" warning for generic varargs; it is allowed on `static`, `final`, `private` methods and constructors.
- Explicit type arguments (`GenericMethodDemo.<Double>listOf(...)`) are rarely needed but can resolve ambiguous inference.

### 4.5 Generic Class vs Generic Method

| Point | Generic class | Generic method |
|---|---|---|
| Declaration | `class Box<T>` | `<T> void m(T t)` |
| Scope of `T` | Whole class (all instance members) | Only that method |
| Type chosen | When the object is created | At each call |
| Static use | Class type parameters **cannot** be used in static members | Static generic methods are common (`Collections.sort`, `List.of`) |

---

## 5. Generic Interfaces

### 5.1 Interview Definition

> **A generic interface is an interface declared with type parameters, such as `interface Repository<T, ID>` or the JDK's `Comparable<T>`, `Comparator<T>`, `Function<T, R>` and `List<E>`. A class implementing a generic interface can either fix the type arguments, such as `class StudentRepository implements Repository<Student, Integer>`, or remain generic itself, such as `class InMemoryRepository<T, ID> implements Repository<T, ID>`.**

### 5.2 Java Example — A Generic Repository

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;
import java.util.function.Function;

public class GenericInterfaceDemo {

    interface Repository<T, ID> {
        void save(T entity);
        Optional<T> findById(ID id);
        List<T> findAll();
    }

    // Implementation that stays generic
    static class InMemoryRepository<T, ID> implements Repository<T, ID> {
        private final Map<ID, T> store = new HashMap<>();
        private final Function<T, ID> idExtractor;

        InMemoryRepository(Function<T, ID> idExtractor) { this.idExtractor = idExtractor; }

        @Override public void save(T entity) { store.put(idExtractor.apply(entity), entity); }
        @Override public Optional<T> findById(ID id) { return Optional.ofNullable(store.get(id)); }
        @Override public List<T> findAll() { return new ArrayList<>(store.values()); }
    }

    record Student(int roll, String name) { }
    record Product(String sku, double price) { }

    // Implementation that fixes the types
    static class StudentRepository extends InMemoryRepository<Student, Integer> {
        StudentRepository() { super(Student::roll); }
    }

    public static void main(String[] args) {
        StudentRepository students = new StudentRepository();
        students.save(new Student(1, "Ravi"));
        students.save(new Student(2, "Priya"));
        System.out.println("Find 2: " + students.findById(2).map(Student::name).orElse("not found"));
        System.out.println("Find 9: " + students.findById(9).map(Student::name).orElse("not found"));

        Repository<Product, String> products = new InMemoryRepository<>(Product::sku);
        products.save(new Product("PEN-01", 10.0));
        System.out.println("Product: " + products.findById("PEN-01").orElseThrow());
        System.out.println("All students count: " + students.findAll().size());
    }
}
```

**Expected Output:**

```text
Find 2: Priya
Find 9: not found
Product: Product[sku=PEN-01, price=10.0]
All students count: 2
```

**Explanation:** One generic `Repository<T, ID>` interface and one generic implementation serve **students and products** with full type safety. `StudentRepository` fixes `T = Student` and `ID = Integer`. This is exactly how Spring Data's `JpaRepository<T, ID>` works.

---

## 6. Raw Types

### 6.1 Interview Definition

> **A raw type is a generic class or interface used without type arguments, such as `List` instead of `List<String>`. Raw types exist only for backward compatibility with code written before Java 5. Using them turns off generic type checking: the compiler gives "unchecked" warnings, any object can be inserted, and `ClassCastException`s can occur at runtime. Raw types should not be used in new code; if the type is unknown, use the unbounded wildcard `List<?>` instead.**

| Declaration | Meaning |
|---|---|
| `List<String>` | List of strings — type-safe |
| `List<Object>` | List that can hold any objects, **type-checked** |
| `List<?>` | List of some unknown type — read as `Object`, cannot add (except `null`) — safe |
| `List` (raw) | Generics disabled — unsafe, legacy only |

---

## 7. Restrictions of Generics

| Restriction | Example (invalid) | Reason / workaround |
|---|---|---|
| No primitives as type arguments | `List<int>` | Use wrappers: `List<Integer>` |
| Cannot create instances of `T` | `new T()` | Type erased at runtime; pass a `Supplier<T>` or `Class<T>` |
| Cannot create generic arrays | `new T[10]`, `new List<String>[5]` | Use `List<T>` or `(T[]) new Object[n]` with care |
| No `instanceof` with type arguments | `obj instanceof List<String>` | Use `obj instanceof List<?>` |
| Static fields cannot use class type parameter | `static T value;` | `T` belongs to instances |
| Cannot overload by erasure | `m(List<String>)` and `m(List<Integer>)` | Both erase to `m(List)` |
| Generic classes cannot extend `Throwable` | `class MyEx<T> extends Exception` | Catch blocks need runtime types |
| No `.class` of parameterised type | `List<String>.class` | Only `List.class` exists |

Most of these come from **type erasure** — explained in the [next chapter](02-Bounded-Types-Wildcards-Type-Erasure.md).

---

## 8. Interview Answer — "What are generics and why do we use them?"

> "Generics let us write classes, interfaces and methods with type parameters, so the same code works with different types while the compiler checks the types. For example, `List<String>` only accepts strings — if I try to add an integer, I get a compile error instead of a ClassCastException at runtime. They were added in Java 5. The main benefits are compile-time type safety, no need for explicit casts, and reusable code — the whole Collections Framework is built on generics. We can create generic classes like `Box<T>` or `Pair<K, V>`, generic methods like `static <T> T first(List<T> list)`, and generic interfaces like `Comparable<T>` or a `Repository<T, ID>`. Type arguments must be reference types, so we use `Integer` instead of `int`. At runtime, the type information is mostly removed by type erasure, which is why we can't do things like `new T()`."

---

## 9. Interview Questions with Complete Answers

### Basic Level

**Q1. What is the diamond operator?**

`<>` used on the right side of an object creation, e.g., `Map<String, List<Integer>> m = new HashMap<>();`. Introduced in Java 7, it lets the compiler infer the type arguments from the declaration, reducing repetition. From Java 9, it also works with anonymous classes (when the inferred type is denotable).

**Q2. Can we use primitive types with generics?**

No. Type arguments must be reference types because generics are implemented with `Object` after erasure. Use wrapper classes (`List<Integer>`); autoboxing converts automatically. For performance-critical primitive work, use arrays or `IntStream`.

**Q3. What does `<T>` before the return type mean?**

It declares a **generic method** with its own type parameter `T`. Without it, `T` would have to be a class-level type parameter or a real class name.

### Intermediate Level

**Q4. What is the difference between `List<Object>` and `List<?>`?**

`List<Object>` is a list whose element type is exactly `Object` — you can add anything, but you **cannot** pass a `List<String>` to a parameter of type `List<Object>` (generics are invariant). `List<?>` means "a list of some unknown type" — any `List<X>` can be passed, you can read elements as `Object`, but you cannot add (except `null`).

**Q5. Why is `List<String>` not a subtype of `List<Object>`?**

Because it would break type safety. If it were allowed, you could do `List<Object> objs = stringList; objs.add(42);` — putting an `Integer` into a list of strings. Generics are **invariant** to prevent this. (Arrays are covariant and check at runtime with `ArrayStoreException` instead.)

**Q6. Can a generic class have a static method that uses the class's type parameter?**

No. Class type parameters belong to instances (each object can have a different `T`), so static members cannot use them. A static method can declare **its own** type parameter: `static <U> Box<U> of(U value)`.

### Advanced Level

**Q7. How do you create an instance of `T` inside a generic class?**

You can't write `new T()` because of type erasure. Pass a factory: a `Supplier<T>` (`new Factory<>(Student::new)`) or a `Class<T>` token and use reflection (`clazz.getDeclaredConstructor().newInstance()`).

**Q8. Can a generic method be overridden?**

Yes. The overriding method must have the same signature after erasure and compatible type parameters. Overloads that differ only in generic type arguments (`process(List<String>)` vs `process(List<Integer>)`) are not allowed because their erasures clash.

**Q9. What is heap pollution?**

When a variable of a parameterised type refers to an object that is not of that type — usually caused by raw types, unchecked casts or generic varargs. The compiler warns ("unchecked"), and a `ClassCastException` can occur later at an unexpected place.

---

## 10. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Using raw types (`List list = new ArrayList();`) | Always specify type arguments. |
| `List<int>` | Use `List<Integer>`. |
| Expecting `List<String>` to be assignable to `List<Object>` | Generics are invariant; use wildcards. |
| `new T()` or `new T[]` | Not allowed; use factories. |
| Using class `T` in static methods | Declare a method-level type parameter. |

---

## 11. Quick Revision

- Generics (Java 5): type parameters → compile-time type safety, no casts, reuse.
- Conventions: T, E, K, V, N, R, S/U.
- Generic class `class Box<T>`; generic method `<T> T m(...)`; generic interface `interface Repo<T, ID>`.
- Diamond `<>` (Java 7). Type inference for methods.
- Raw types = legacy, unsafe. `List<?>` for unknown types.
- Restrictions: no primitives, no `new T()`, no generic arrays, no `instanceof List<String>`, no static `T`, erasure clashes.

---

[← Previous Part: Collections Questions](../05-Collections-Framework/09-Collections-Comparisons-and-Interview-Questions.md) | [Back to README](../README.md) | [Next: Bounded Types, Wildcards, Type Erasure →](02-Bounded-Types-Wildcards-Type-Erasure.md)
