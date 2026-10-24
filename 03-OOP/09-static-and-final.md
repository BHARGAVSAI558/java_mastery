# Chapter 3.9 — `static` and `final`

[← Previous: Access Modifiers](08-Access-Modifiers.md) | [Back to README](../README.md) | [Next: Object Class →](10-Object-Class-equals-hashCode-toString.md)

---

## What You Will Learn

- The `static` keyword: static variables, methods, blocks, nested classes and static imports
- When static members are created and where they live
- Restrictions of static methods; static vs instance
- The `final` keyword: final variables, methods, classes and parameters
- Blank final variables and `static final` constants
- `final` vs immutability
- `final` vs `finally` vs `finalize()`

---

## Part A — The `static` Keyword

### 1.1 Interview Definition

> **The `static` keyword in Java indicates that a member belongs to the class itself rather than to any particular object. A static variable has a single copy shared by all objects of the class; a static method can be called using the class name without creating an object and can directly access only static members; a static block runs once when the class is loaded and is used for static initialisation; and a static nested class is a nested class that does not need an instance of the outer class. Static members are created when the class is loaded, before any object exists.**

### 1.2 Understand the Concept

| Static member | Meaning | Typical use |
|---|---|---|
| **Static variable** (class variable) | One copy per class, shared by all objects | Counters, configuration, constants, caches |
| **Static method** (class method) | Called via `ClassName.method()`; no `this` | Utility methods (`Math.max`), factory methods (`List.of`), `main` |
| **Static block** | Runs once when the class is initialised | Complex static initialisation, loading drivers/config |
| **Static nested class** | Nested class not tied to an outer instance | Builders, helper classes, `Map.Entry` |
| **Static import** | Use static members without the class prefix | `import static java.lang.Math.*;` |

**Memory and life cycle:**
- Static variables are created when the class is **loaded and initialised** — once.
- Since Java 8, static variables are stored in the **heap** (as part of the `Class` object); class metadata is in **Metaspace**.
- They live until the class is unloaded (normally until the program ends).

```text
              Class Student (loaded once)
              ┌───────────────────────────┐
              │ static collegeName = "ABC" │  ◄── ONE shared copy
              │ static count = 3           │
              └───────────────────────────┘
                 ▲          ▲          ▲
    ┌────────────┴┐ ┌───────┴─────┐ ┌──┴──────────┐
    │ s1: name=A  │ │ s2: name=B  │ │ s3: name=C  │   ◄── each object has its own instance fields
    │     roll=1  │ │     roll=2  │ │     roll=3  │
    └─────────────┘ └─────────────┘ └─────────────┘
```

### 1.3 Real-World Example

In a **classroom**, the **whiteboard** is static — one board shared by all students; if the teacher writes on it, everyone sees the change. Each student's **notebook** is an instance variable — everyone has their own. A **wall clock** is like a static utility method — anyone can look at it without owning it.

### 1.4 Java Example

```java
import static java.lang.Math.sqrt;          // static import
import static java.lang.Math.PI;

public class StaticDemo {

    static class Student {
        static String collegeName;           // static variable (shared)
        static int count = 0;                // static counter
        final int rollNo;                    // instance variable (per object)
        String name;

        static {                             // static block: runs once
            collegeName = "ABC Engineering College";
            System.out.println("Static block: Student class initialised");
        }

        Student(String name) {
            this.name = name;
            count++;                         // instance code can access static members
            this.rollNo = count;
        }

        static int getCount() {              // static method
            // System.out.println(name);     // compile error: non-static variable name cannot be referenced
            return count;
        }

        static Student createTopper() {      // static factory method
            Student s = new Student("Topper");
            return s;
        }

        void display() {
            System.out.println(rollNo + ". " + name + " - " + collegeName);
        }
    }

    // Static utility class
    static final class MathUtils {
        private MathUtils() { }              // prevent object creation
        static int square(int x) { return x * x; }
        static boolean isEven(int x) { return x % 2 == 0; }
    }

    public static void main(String[] args) {
        System.out.println("Before any Student is used");
        System.out.println("Count via class name: " + Student.getCount());   // triggers class init

        Student a = new Student("Asha");
        Student b = new Student("Bala");
        Student t = Student.createTopper();
        a.display();
        b.display();
        t.display();

        Student.collegeName = "XYZ Institute";       // change once, visible to all
        a.display();
        b.display();

        System.out.println("Total students: " + Student.count);
        System.out.println("square(7) = " + MathUtils.square(7) + ", isEven(7) = " + MathUtils.isEven(7));
        System.out.printf("sqrt(49) = %.1f, PI = %.4f%n", sqrt(49), PI);   // via static import
    }
}
```

### 1.5 Code Explanation

- The static block runs **once**, the first time `Student` is actually used (`Student.getCount()`), not at program start. That is why "Before any Student is used" prints first.
- `count` is shared: every constructor call increases the same variable, and each student uses it to get a unique roll number.
- `getCount()` is static: it cannot access `name` (an instance variable) because there is no object (`this`) in a static method.
- Changing `Student.collegeName` once updates it for **all** objects.
- `MathUtils` is a **utility class**: `final`, private constructor, only static methods (like `java.lang.Math`, `java.util.Collections`).
- `import static java.lang.Math.sqrt;` lets us write `sqrt(49)` instead of `Math.sqrt(49)`.

### 1.6 Expected Output

```text
Before any Student is used
Static block: Student class initialised
Count via class name: 0
1. Asha - ABC Engineering College
2. Bala - ABC Engineering College
3. Topper - ABC Engineering College
1. Asha - XYZ Institute
2. Bala - XYZ Institute
Total students: 3
square(7) = 49, isEven(7) = false
sqrt(49) = 7.0, PI = 3.1416
```

### 1.7 Restrictions of Static Methods

A static method:
1. **Cannot** access instance variables or call instance methods **directly** (only via an object reference).
2. **Cannot** use `this` or `super`.
3. **Cannot be overridden** — a same-signature static method in a subclass **hides** it.
4. **Cannot be abstract** (in a class).
5. **Can** be overloaded.

### 1.8 Static vs Instance — Comparison

| Point | Static | Instance |
|---|---|---|
| Belongs to | Class | Object |
| Copies of variable | One | One per object |
| Created | When class is loaded | When object is created |
| Access | `ClassName.member` | `object.member` |
| `this`/`super` available | No | Yes |
| Can access instance members directly | No | Yes |
| Overriding (methods) | No (hiding) | Yes |
| Memory | With the class (heap `Class` object, Java 8+) | Inside each object (heap) |

### 1.9 Interview Answer — "What is the static keyword?"

> "The `static` keyword means the member belongs to the class, not to individual objects. A **static variable** has only one copy shared by all objects — for example, a counter of how many objects were created, or a common college name. A **static method** can be called using the class name without creating an object, like `Math.sqrt()`; because there is no object, it cannot access instance variables directly or use `this` and `super`. A **static block** runs once when the class is loaded and is used to initialise static data. We also have **static nested classes** and **static imports**. Static members are created when the class is loaded, before any object exists. Static methods cannot be overridden; if a subclass defines the same static method, it only hides the parent's method. The `main` method is static so the JVM can call it without creating an object."

---

## Part B — The `final` Keyword

### 2.1 Interview Definition

> **The `final` keyword in Java is used to restrict change. A `final` variable can be assigned only once, so its value (or, for a reference, the object it points to) cannot be changed after initialisation. A `final` method cannot be overridden by subclasses. A `final` class cannot be extended. `final` can also be applied to method parameters and local variables. Combined with `static`, it is used to declare constants. Note that a final reference variable does not make the object immutable — the object's internal state can still change.**

### 2.2 Understand the Concept

| Applied to | Effect | Example |
|---|---|---|
| **Local variable** | Assigned once | `final int x = 10;` |
| **Instance variable** | Assigned once — at declaration, in an instance block, or in **every** constructor | `private final String id;` |
| **Static variable** | Assigned once — at declaration or in a static block → **constant** | `static final double PI = 3.14159;` |
| **Parameter** | Cannot be re-assigned inside the method | `void m(final int x)` |
| **Method** | Cannot be overridden | `final void calculate()` |
| **Class** | Cannot be extended | `final class String` |

**Blank final variable:** a `final` field that is not initialised at declaration. It **must** be initialised in **every constructor** (or an instance initializer). A **blank static final** must be initialised in a **static block**.

### 2.3 Real-World Example

- A **final variable** is like your **date of birth** on a certificate — set once, never changed.
- A **final method** is like a **company policy** that every branch must follow exactly; branches (subclasses) cannot change it.
- A **final class** is like a **sealed product** — no one can build a modified version of it.

### 2.4 Java Example

```java
import java.util.ArrayList;
import java.util.List;

public class FinalDemo {

    static class Employee {
        private final String employeeId;                       // blank final: set in constructor
        private final List<String> skills = new ArrayList<>(); // final reference to a mutable list
        static final String COMPANY = "TechCorp";              // constant
        static final int MAX_LEAVES;                           // blank static final

        static {
            MAX_LEAVES = 24;                                   // initialised in static block
        }

        Employee(String employeeId) {
            this.employeeId = employeeId;                      // must be assigned in every constructor
        }

        final String getEmployeeId() {                         // final method: cannot be overridden
            return employeeId;
        }

        void addSkill(String skill) {
            skills.add(skill);                                 // OK: changing the object's state
            // skills = new ArrayList<>();                     // compile error: cannot assign a final variable
        }

        List<String> getSkills() { return skills; }
    }

    static class Manager extends Employee {
        Manager(String id) { super(id); }
        // String getEmployeeId() { return "X"; }              // compile error: cannot override final method
    }

    static int doubleIt(final int value) {
        // value = value * 2;                                  // compile error: final parameter
        return value * 2;
    }

    public static void main(String[] args) {
        final int localConstant = 100;
        // localConstant = 200;                                // compile error

        Employee e = new Employee("E101");
        e.addSkill("Java");
        e.addSkill("SQL");
        System.out.println(e.getEmployeeId() + " skills: " + e.getSkills());

        Manager m = new Manager("M201");
        System.out.println("Manager id: " + m.getEmployeeId());

        System.out.println("Company: " + Employee.COMPANY + ", Max leaves: " + Employee.MAX_LEAVES);
        System.out.println("doubleIt(21) = " + doubleIt(21) + ", localConstant = " + localConstant);

        final int[] numbers = {1, 2, 3};
        numbers[0] = 99;                                       // allowed: array content changes
        System.out.println("final array content changed: " + numbers[0]);
    }
}
```

### 2.5 Code Explanation

- `employeeId` is a **blank final**: it gets its value in the constructor and can never change. This is the standard way to create read-only fields.
- `skills` is `final`, but the **list object itself is mutable**: `skills.add()` works; only re-assigning `skills` is blocked. **final reference ≠ immutable object.**
- `COMPANY` is a constant (`static final`), named in UPPER_SNAKE_CASE.
- `MAX_LEAVES` is a blank static final, initialised in a static block.
- `getEmployeeId()` is `final`: `Manager` inherits it but cannot override it.
- A `final` array reference cannot point to another array, but its elements can be changed.

### 2.6 Expected Output

```text
E101 skills: [Java, SQL]
Manager id: M201
Company: TechCorp, Max leaves: 24
doubleIt(21) = 42, localConstant = 100
final array content changed: 99
```

### 2.7 Why Use `final`?

1. **Safety** — prevent accidental re-assignment of important values.
2. **Immutability** — `final` fields are a key part of immutable classes ([Part 10](../10-Additional-Java-Topics/08-Immutable-Classes-and-Records.md)).
3. **Thread safety** — the Java Memory Model guarantees that properly constructed objects' `final` fields are visible to all threads without synchronization.
4. **Design** — `final` methods/classes protect behaviour that must not change (`String`, security checks).
5. **Lambdas and inner classes** — local variables used inside them must be `final` or **effectively final** (never re-assigned).
6. **Compile-time constants** — `static final` primitives/strings initialised with constant expressions are inlined by the compiler.

### 2.8 Interview Answer — "What is the final keyword?"

> "The `final` keyword is used to prevent change. When applied to a **variable**, it can be assigned only once — for a primitive, the value can't change, and for a reference, it can't point to another object, although the object itself can still be modified. A final field that is not initialised at declaration is called a blank final, and it must be initialised in every constructor. When applied to a **method**, the method cannot be overridden. When applied to a **class**, the class cannot be extended — `String` and wrapper classes are examples. Combining `static` and `final` creates constants, like `static final double PI`. We use final for safety, for immutable classes, and because variables used inside lambdas must be final or effectively final."

---

## 3. `final` vs `finally` vs `finalize()`

| | `final` | `finally` | `finalize()` |
|---|---|---|---|
| What it is | Keyword (modifier) | Block used with `try` | Method of `Object` |
| Purpose | Restrict change: constant variables, no overriding, no inheritance | Code that always runs after `try`/`catch` — cleanup | Called by the GC before reclaiming an object (cleanup) |
| Applied to | Variables, methods, classes | Exception handling | Objects (overridden in a class) |
| Status | Widely used | Widely used | **Deprecated since Java 9**, deprecated *for removal* since Java 18; unpredictable — use try-with-resources or `Cleaner` instead |

```java
public class FinalFinallyFinalize {
    public static void main(String[] args) {
        final int limit = 3;                       // final
        try {
            System.out.println("In try, limit = " + limit);
            int x = 10 / 0;
        } catch (ArithmeticException e) {
            System.out.println("Caught: " + e.getMessage());
        } finally {
            System.out.println("finally always runs");   // finally
        }
        // finalize(): do not use — deprecated
    }
}
```

**Expected Output:**

```text
In try, limit = 3
Caught: / by zero
finally always runs
```

---

## 4. Interview Questions with Complete Answers

### Basic Level

**Q1. Why is the `main` method static?**

So that the JVM can call it without creating an object. At program start, no object of the class exists, and the JVM would not know which constructor or arguments to use.

**Q2. Can we access a static variable using an object reference?**

Yes, `obj.staticVar` compiles, but it is misleading — the compiler converts it to `ClassName.staticVar`. It even works if `obj` is `null` (no `NullPointerException`), because the object is not used. Always use the class name.

```java
public class StaticViaNull {
    static int counter = 42;
    public static void main(String[] args) {
        StaticViaNull ref = null;
        System.out.println(ref.counter);       // works: resolved by type, not object
    }
}
```

**Expected Output:**

```text
42
```

**Q3. Can a final variable be initialised later?**

Yes, if it is a **blank final**. An instance blank final must be assigned exactly once in every constructor (or an instance initializer block). A static blank final must be assigned in a static block. A local final variable can be declared first and assigned once later.

### Intermediate Level

**Q4. Can we make a constructor `static` or `final`?**

No. Constructors initialise objects, so `static` makes no sense. They are never inherited or overridden, so `final` makes no sense either. Both are compile errors.

**Q5. Can a static method be overridden? What is method hiding?**

No. If a subclass defines a static method with the same signature as a static method in the parent, the child's method **hides** the parent's. Which one runs depends on the **reference type** at compile time, not the object type. With overriding (instance methods), the object type decides.

**Q6. What is the difference between `static final` and just `final`?**

`final` alone: each object has its own copy, assigned once per object (it can differ between objects — e.g., `employeeId`). `static final`: one copy for the whole class, assigned once — a true constant (e.g., `MAX_SIZE`).

**Q7. Does `final` make an object immutable?**

No. `final` on a reference prevents re-assigning the variable. The object it refers to can still change (`finalList.add(x)` works). To make an object immutable, its class must have private final fields, no setters, defensive copies of mutable fields, and usually be final.

**Q8. When are static blocks executed? Can a class have multiple static blocks?**

Static blocks execute **once**, when the class is **initialised** — which happens on first active use: creating an object, calling a static method, accessing a non-constant static field, or using reflection like `Class.forName()`. A class can have multiple static blocks; they run in the order they appear.

### Advanced Level

**Q9. Will a static block run if we access a `static final` constant?**

Not if the constant is a **compile-time constant** (primitive or `String` initialised with a constant expression). The compiler copies (inlines) the value into the using class, so the class is not even initialised.

```java
public class ConstantInlining {
    static class Config {
        static final int TIMEOUT = 30;                      // compile-time constant
        static final String NAME = "App";                   // compile-time constant
        static final Integer BOXED = 5;                     // NOT a compile-time constant
        static { System.out.println("Config static block"); }
    }

    public static void main(String[] args) {
        System.out.println("TIMEOUT = " + Config.TIMEOUT);  // no class initialisation
        System.out.println("NAME = " + Config.NAME);        // no class initialisation
        System.out.println("BOXED = " + Config.BOXED);      // triggers static block
    }
}
```

**Expected Output:**

```text
TIMEOUT = 30
NAME = App
Config static block
BOXED = 5
```

**Q10. What is "effectively final"?**

A local variable that is **not declared `final` but is never re-assigned** after initialisation (Java 8). Such variables can be used inside lambdas and anonymous/local classes. If you re-assign it anywhere, using it in a lambda is a compile error: "local variables referenced from a lambda expression must be final or effectively final".

**Q11. Why are static variables a risk in multithreaded or large applications?**

They are shared global state: every thread and every object can change them, which leads to race conditions unless synchronized, and makes code harder to test (state leaks between tests). Static collections used as caches can also cause **memory leaks** because they live as long as the class is loaded. Prefer instance state and dependency injection; use static mainly for constants and stateless utilities.

**Q12. What is a static nested class vs an inner class?**

A **static nested class** does not need an instance of the outer class and cannot access the outer class's instance members directly (`Outer.Nested n = new Outer.Nested();`). An **inner (non-static) class** is tied to an instance of the outer class and can access all of its members (`Outer.Inner i = outer.new Inner();`). See [Part 10, Nested Classes](../10-Additional-Java-Topics/07-Nested-and-Inner-Classes.md).

---

## 5. Follow-Up Questions

**F1. Can an abstract method be static or final?**

No for both. Abstract methods must be overridden; static methods can't be overridden and final methods can't be overridden.

**F2. Can we declare a local variable `static`?**

No. Local variables live only during a method call; `static` means "belongs to the class". `static int x` inside a method is a compile error.

**F3. Is `static` thread-safe?**

No. `static` only means shared. Shared mutable data requires synchronization, atomic classes or concurrent collections. `static final` immutable values (like `String` constants) are safe.

**F4. Why is `String` final?**

To guarantee immutability and security: no subclass can change its behaviour, so strings used as class names, file paths, and map keys can be trusted. It also allows the string pool and hashcode caching to work safely.

---

## 6. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Accessing instance fields from a static method | Use an object reference. |
| Using `this` in a static context | Compile error. |
| Thinking `final List` cannot be modified | The list's contents can change; only re-assignment is blocked. |
| Forgetting to initialise a blank final in one constructor | Compile error. |
| Thinking static methods are overridden | They are hidden. |
| Using mutable static fields as global variables | Causes thread-safety and testing issues. |
| Confusing `final`, `finally` and `finalize()` | Three unrelated things. |

---

## 7. Quick Revision

**static**
- Belongs to the class; one copy; created at class loading.
- Variable (shared), method (no object, no `this`), block (runs once), nested class, static import.
- Static methods: can't use instance members directly, can't be overridden (hidden), can be overloaded.
- Static block runs on first active use; compile-time constants don't trigger it.

**final**
- Variable → assign once (blank final in every constructor / static block).
- Method → no overriding. Class → no inheritance.
- `static final` → constant (UPPER_SNAKE_CASE).
- final reference ≠ immutable object.
- Lambdas need final or effectively final locals.

**final vs finally vs finalize** → modifier vs cleanup block vs deprecated GC callback.

---

[← Previous: Access Modifiers](08-Access-Modifiers.md) | [Back to README](../README.md) | [Next: Object Class →](10-Object-Class-equals-hashCode-toString.md)
