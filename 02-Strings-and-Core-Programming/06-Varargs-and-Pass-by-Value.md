# Chapter 2.6 — Varargs and Pass-by-Value

[← Previous: Wrapper Classes](05-Wrapper-Classes-Autoboxing-Unboxing.md) | [Back to README](../README.md) | [Next: Core Coding Problems →](07-Core-Coding-Problems.md)

---

## What You Will Learn

- Variable-length arguments (varargs): syntax, rules, how they work internally
- Varargs and overloading ambiguity
- **Java is always pass-by-value** — with primitives, objects, arrays, strings and wrappers
- Why a method can change an object's content but cannot replace the caller's object
- How to explain this confidently in an interview

---

## 1. Varargs (Variable-Length Arguments)

### 1.1 Interview Definition

> **Varargs, introduced in Java 5, allow a method to accept zero or more arguments of the same type using the syntax `Type... name`. Inside the method, the varargs parameter is treated as an array of that type. The caller can pass individual values separated by commas, an existing array, or no values at all. A method can have only one varargs parameter, and it must be the last parameter in the parameter list.**

### 1.2 Understand the Concept

**The problem before Java 5:** To write a `sum` method for 2, 3 or 10 numbers, you had to either write many overloaded methods or force the caller to create an array: `sum(new int[]{1, 2, 3})`.

**With varargs:** `int sum(int... numbers)` can be called as `sum()`, `sum(5)`, `sum(1, 2, 3)` or `sum(existingArray)`.

**How it works internally:** The compiler converts `int... numbers` into `int[] numbers`, and converts each call `sum(1, 2, 3)` into `sum(new int[]{1, 2, 3})`. So varargs is just **syntactic sugar for arrays**.

**Rules:**
1. Only **one** varargs parameter per method.
2. It must be the **last** parameter: `void log(String level, String... messages)` ✅, `void log(String... messages, String level)` ❌.
3. A call with no arguments passes an **empty array** (length 0), not `null`.
4. You can pass an actual array to a varargs parameter.
5. Well-known varargs methods: `String.format(String, Object...)`, `System.out.printf`, `Arrays.asList(T...)`, `List.of(E...)`, `main(String... args)`.

### 1.3 Real-World Example

A varargs method is like a **shopping bag at a supermarket billing counter**. Whether you put in 0 items, 1 item or 20 items, the cashier processes the bag the same way — item by item.

### 1.4 Java Example

```java
public class VarargsDemo {

    static int sum(int... numbers) {
        System.out.print("(received " + numbers.length + " values) ");
        int total = 0;
        for (int n : numbers) {
            total += n;
        }
        return total;
    }

    // Varargs must be the last parameter
    static void printReport(String title, String... lines) {
        System.out.println("== " + title + " ==");
        for (String line : lines) {
            System.out.println(" - " + line);
        }
    }

    static double average(double first, double... rest) {   // forces at least one argument
        double total = first;
        for (double d : rest) {
            total += d;
        }
        return total / (1 + rest.length);
    }

    public static void main(String[] args) {
        System.out.println(sum());
        System.out.println(sum(5));
        System.out.println(sum(1, 2, 3, 4));
        int[] existing = {10, 20, 30};
        System.out.println(sum(existing));                    // pass an array directly

        printReport("Skills", "Java", "SQL", "Spring Boot");
        printReport("Empty");

        System.out.println("average = " + average(4, 6, 8));
        System.out.println(String.format("%s is %d years old", "Ravi", 21));   // JDK varargs method
    }
}
```

### 1.5 Code Explanation

- `sum()` → `numbers` is an empty array (length 0), so the loop does not run and the result is 0.
- `sum(existing)` — an existing array can be passed directly; no new array is created.
- `printReport("Empty")` — `lines` is an empty array, so only the title prints.
- `average(double first, double... rest)` — a common trick to **require at least one** argument: the first value is a normal parameter.
- `String.format` is itself a varargs method: `format(String format, Object... args)`.

### 1.6 Expected Output

```text
(received 0 values) 0
(received 1 values) 5
(received 4 values) 10
(received 3 values) 60
== Skills ==
 - Java
 - SQL
 - Spring Boot
== Empty ==
average = 6.0
Ravi is 21 years old
```

### 1.7 Varargs and Overloading

- Overload resolution tries **varargs last** (after exact match, widening and boxing).
- Ambiguity example:

<!-- skip-check -->
```java
static void show(int... a)     { }
static void show(Integer... a) { }
show();          // COMPILE ERROR: reference to show is ambiguous
```

Both methods accept zero arguments, and neither is more specific. Avoid overloading varargs methods with similar types.

```java
public class VarargsOverload {
    static void test(int a)       { System.out.println("test(int)"); }
    static void test(int... a)    { System.out.println("test(int...)"); }
    static void test(int a, int b){ System.out.println("test(int, int)"); }

    public static void main(String[] args) {
        test(1);          // exact match wins
        test(1, 2);       // exact match wins
        test(1, 2, 3);    // only varargs fits
        test();           // only varargs fits
    }
}
```

**Expected Output:**

```text
test(int)
test(int, int)
test(int...)
test(int...)
```

### 1.8 Interview Answer — "What are varargs?"

> "Varargs, or variable-length arguments, were added in Java 5. They let a method accept any number of arguments of the same type, including zero, using the three-dot syntax like `int... numbers`. Inside the method, the parameter behaves like an array, so we can use `length` and loop over it. The compiler actually converts the call into an array creation. There are two main rules: a method can have only one varargs parameter, and it must be the last parameter. Common examples are `String.format`, `printf` and `Arrays.asList`. In overloading, the compiler considers varargs methods last, after exact matches, widening and boxing."

---

## 2. Pass-by-Value

### 2.1 Interview Definition

> **Java is strictly pass-by-value. When a method is called, the value of each argument is copied into the method's parameter. For primitive types, the copied value is the actual data, so changes to the parameter inside the method do not affect the caller's variable. For objects, the variable holds a reference, and the value that is copied is that reference. So both the caller's variable and the parameter point to the same object: the method can change the object's internal state, and the caller will see those changes, but if the method assigns a new object to the parameter, the caller's variable is not affected. Java never passes the variable itself, which is what pass-by-reference would mean.**

### 2.2 Understand the Concept

**Pass-by-value vs pass-by-reference (general programming terms):**

| | Pass-by-value | Pass-by-reference |
|---|---|---|
| What is passed | A **copy** of the value | The **variable itself** (an alias) |
| Re-assigning the parameter | Does **not** affect the caller | **Does** affect the caller |
| Languages | Java (always), C, Python-like semantics | C++ (`int&`), C# (`ref`) |

In Java, a **reference variable** contains a value — an address-like reference to an object. That reference value is copied. This is sometimes described as **"pass-by-value of the reference"** or "pass-by-sharing".

**Three cases:**

| Case | What happens | Caller sees change? |
|---|---|---|
| Change a **primitive** parameter | Copy is changed | ❌ No |
| Change an **object's field** through the parameter (`p.name = "X"`) | Same object is changed | ✅ Yes |
| **Re-assign** the parameter to a new object (`p = new Person()`) | Only the copy now points elsewhere | ❌ No |

### 2.3 Real-World Example

You have a **house** (object) and a piece of paper with its **address** (reference). When you call a method, you give the method a **photocopy** of your address paper.

- The method can go to the house using the address and **repaint the walls** (modify the object) — you will see the new paint.
- The method can **write a different address** on its photocopy (re-assign the parameter) — your own paper still has your original address, so nothing changes for you.

### 2.4 Java Example

```java
import java.util.Arrays;

public class PassByValueDemo {
    static class Person {
        String name;
        Person(String name) { this.name = name; }
    }

    static void changePrimitive(int x) {
        x = 100;
        System.out.println("  inside changePrimitive: x = " + x);
    }

    static void changeObjectState(Person p) {
        p.name = "Changed";                 // modifies the shared object
    }

    static void reassignObject(Person p) {
        p = new Person("New Person");       // only the local copy points to a new object
        p.name = "Modified New Person";
    }

    static void changeArray(int[] arr) {
        arr[0] = 999;                       // modifies the shared array
        arr = new int[]{1, 1, 1};           // re-assignment: caller unaffected
    }

    static void changeString(String s) {
        s = s + " World";                   // creates a new String; caller unaffected
    }

    static void changeWrapper(Integer n) {
        n = n + 1;                          // new Integer object; caller unaffected
    }

    static void swap(Person a, Person b) {  // does NOT work in Java
        Person temp = a;
        a = b;
        b = temp;
    }

    public static void main(String[] args) {
        int number = 10;
        changePrimitive(number);
        System.out.println("1. primitive after call: " + number);

        Person person = new Person("Ravi");
        changeObjectState(person);
        System.out.println("2. object after state change: " + person.name);

        reassignObject(person);
        System.out.println("3. object after reassign: " + person.name);

        int[] data = {1, 2, 3};
        changeArray(data);
        System.out.println("4. array after call: " + Arrays.toString(data));

        String text = "Hello";
        changeString(text);
        System.out.println("5. string after call: " + text);

        Integer count = 5;
        changeWrapper(count);
        System.out.println("6. Integer after call: " + count);

        Person p1 = new Person("A");
        Person p2 = new Person("B");
        swap(p1, p2);
        System.out.println("7. after swap: p1 = " + p1.name + ", p2 = " + p2.name);
    }
}
```

### 2.5 Code Explanation (with memory pictures)

**Case 2 — `changeObjectState(person)`**

```text
main:   person ──┐
                 ├──► Person{name="Ravi"} → becomes "Changed"
method: p ───────┘     (both references point to ONE object)
```
The method follows its copy of the reference to the same object and changes its field. The caller sees `"Changed"`.

**Case 3 — `reassignObject(person)`**

```text
main:   person ────► Person{name="Changed"}       (unchanged)
method: p ─────────► Person{name="Modified New Person"}   (new object, lost after return)
```
`p = new Person(...)` changes only the method's copy. The caller's `person` still points to the original object.

**Case 4 — arrays** behave like objects: `arr[0] = 999` changes the shared array; `arr = new int[]{...}` affects only the local copy.

**Cases 5 and 6 — `String` and `Integer`** are immutable. `s = s + " World"` and `n = n + 1` create **new objects** and re-assign the local parameter, so the caller does not see any change. (This is why some people *wrongly* think strings are "passed by value and objects by reference" — in fact, everything is passed by value, and strings simply cannot be modified.)

**Case 7 — `swap`** only swaps the method's local copies of the references. This is the **classic proof** that Java is not pass-by-reference: in a pass-by-reference language like C++ (with `Person&`), the swap would work.

### 2.6 Expected Output

```text
  inside changePrimitive: x = 100
1. primitive after call: 10
2. object after state change: Changed
3. object after reassign: Changed
4. array after call: [999, 2, 3]
5. string after call: Hello
6. Integer after call: 5
7. after swap: p1 = A, p2 = B
```

### 2.7 Interview Answer — "Is Java pass-by-value or pass-by-reference?"

> "Java is **always pass-by-value**. When we call a method, a copy of each argument's value is passed. For primitives, the copy is the actual value, so changing the parameter inside the method does not affect the original variable. For objects, the variable holds a reference, so the method gets a **copy of the reference**. Because both references point to the same object, the method can modify the object's fields, and the caller will see those changes. But if the method assigns a new object to the parameter, only its local copy changes, and the caller's variable still points to the old object. The best proof is a swap method: swapping two object parameters inside a method does not swap them in the caller. So the correct statement is: Java passes object references by value."

---

## 3. Interview Questions with Complete Answers

### Basic Level

**Q1. Can a method have two varargs parameters?**

No. Only one varargs parameter is allowed, and it must be the last parameter. Otherwise, the compiler could not decide where one variable list ends and the next begins.

**Q2. What happens if you call a varargs method without arguments?**

The method receives an **empty array** with length 0. It does not receive `null`. (It receives `null` only if you explicitly pass `null`.)

**Q3. Is `main(String... args)` valid?**

Yes. `String... args` is equivalent to `String[] args`, so the JVM accepts it as the entry point.

### Intermediate Level

**Q4. If Java is pass-by-value, how can a method modify an object passed to it?**

Because the value that is copied is a **reference** to the object. The method's parameter and the caller's variable hold the same reference, so they point to the same object. Changing the object's fields through either reference changes that single object. What the method cannot do is make the caller's variable point to a different object.

**Q5. Why doesn't changing a `String` parameter affect the caller?**

`String` is immutable. Any "change" like `s = s + "x"` creates a new `String` object and assigns it to the local parameter. The caller's variable still refers to the original object, which was never modified.

**Q6. How do you swap two integers using a method in Java?**

You cannot swap two primitive variables directly through a method. Workarounds:
- Pass an **array** and swap its elements: `swap(int[] arr, int i, int j)`.
- Use a **wrapper/holder object** with fields and swap the fields.
- Return the swapped values (e.g., as an array or a record) and re-assign in the caller.

```java
import java.util.Arrays;

public class SwapWorkaround {
    static void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }

    public static void main(String[] args) {
        int[] values = {5, 9};
        swap(values, 0, 1);
        System.out.println(Arrays.toString(values));
    }
}
```

**Expected Output:**

```text
[9, 5]
```

### Advanced Level

**Q7. What is the difference between `void m(int[] a)` and `void m(int... a)` for the caller?**

Inside the method, both are arrays. For the caller, `m(int[] a)` requires an array (`m(new int[]{1,2})`), while `m(int... a)` also accepts a comma-separated list (`m(1, 2)`) or nothing (`m()`). You cannot declare both in the same class because they have the same signature after compilation.

**Q8. What is heap pollution with generic varargs, and what is `@SafeVarargs`?**

A generic varargs parameter like `T... items` creates an array of a non-reifiable type (`T[]` becomes `Object[]` at runtime due to type erasure). If code stores the wrong type into that array, a `ClassCastException` can happen later — this is called **heap pollution**, and the compiler gives an "unchecked generic array creation" warning. If the method only reads from the array and does not expose it, you can mark it with **`@SafeVarargs`** (allowed on `static`, `final`, and `private` methods, and constructors) to suppress the warning. `List.of(E...)` and `Arrays.asList(T...)` use `@SafeVarargs`.

**Q9. Can the caller's object become eligible for garbage collection because of something a method does?**

No. The method only has a copy of the reference. Setting the parameter to `null` inside the method (`p = null`) does not affect the caller's variable, so the object stays reachable as long as the caller holds its reference.

---

## 4. Follow-Up Questions

**F1. Is passing a large object to a method expensive?**

No. Only the reference (typically 4 or 8 bytes) is copied, not the object. That is why Java passes large objects cheaply.

**F2. How would you protect an object from being modified by a method?**

Pass an **immutable** object, pass a **copy** (defensive copy), or pass an unmodifiable view (for collections, `Collections.unmodifiableList(list)` or `List.copyOf(list)`).

**F3. Does `final` on a parameter make the object immutable?**

No. `final Person p` only prevents re-assigning `p` inside the method (`p = new Person()` is a compile error). The object's fields can still be changed: `p.name = "X"` is allowed.

---

## 5. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| "Objects are passed by reference in Java." | References are passed **by value**. |
| Expecting a swap method to swap caller variables | It only swaps local copies. |
| Thinking `String` is special in passing | It is passed the same way; it is just immutable. |
| Placing varargs before another parameter | Varargs must be last. |
| Overloading `m(int...)` and `m(Integer...)` | Ambiguous calls like `m()`. |
| Thinking `final` parameter = immutable object | It only prevents re-assignment. |

---

## 6. Quick Revision

- Varargs (Java 5): `Type... name` → array inside method; 0 or more args; **one per method, must be last**; no args → empty array.
- Overload resolution checks varargs **last**.
- **Java is always pass-by-value.**
  - Primitive → value copied → caller unchanged.
  - Object → reference copied → object state changes are visible; re-assignment is not.
  - String/wrappers → immutable → caller never sees a change.
- `swap(a, b)` on references does not work → proof of pass-by-value.

---

[← Previous: Wrapper Classes](05-Wrapper-Classes-Autoboxing-Unboxing.md) | [Back to README](../README.md) | [Next: Core Coding Problems →](07-Core-Coding-Problems.md)
