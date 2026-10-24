# Chapter 3.12 — Upcasting, Downcasting and Dynamic Method Dispatch

[← Previous: Association, Aggregation, Composition](11-Association-Aggregation-Composition.md) | [Back to README](../README.md) | [Next: SOLID Principles →](13-SOLID-Principles-and-OOP-Design.md)

---

## What You Will Learn

- Reference type vs object type — the key idea behind casting
- Upcasting — what it is, why it is safe and why we use it
- Downcasting — when it is needed, `ClassCastException`, and `instanceof`
- Pattern matching for `instanceof` (Java 16) and switch (Java 21)
- Dynamic method dispatch — how the JVM chooses the method at runtime
- What the compiler checks vs what the JVM decides

---

## 1. Reference Type vs Object Type

```java
Animal a = new Dog();
//  ^             ^
//  reference     object (actual) type
//  type
```

| | Reference (declared/static) type | Object (actual/runtime) type |
|---|---|---|
| Decided | At compile time | At runtime (`new Dog()`) |
| Controls | **Which methods/fields you can access** (compiler check) | **Which overridden method actually runs** |
| Can change? | Fixed for the variable | The variable can later point to other objects |

**Golden rule:**
> The **compiler** looks at the **reference type** to check that a call is allowed.
> The **JVM** looks at the **object type** to decide which overridden method runs.

---

## 2. Upcasting

### 2.1 Interview Definition

> **Upcasting is the conversion of a subclass reference into a superclass (or interface) reference, such as `Animal a = new Dog();`. It is done automatically (implicitly) by the compiler and is always safe, because every subclass object is also an object of its superclass. After upcasting, only the members declared in the superclass type can be accessed through that reference, but overridden methods still execute the subclass implementation. Upcasting is the basis of runtime polymorphism and lets us write general code that works with many subtypes.**

### 2.2 Understand the Concept

- It is **safe** because of IS-A: a `Dog` IS-A `Animal`, so it has everything an `Animal` has.
- Nothing about the **object** changes — only the **view** through the reference becomes narrower.
- Why do it? To write **general, reusable code**: `void feed(Animal a)` accepts dogs, cats and any future animal; `List<Animal>` can hold all of them.

### 2.3 Real-World Example

A **Dog** at a **vet clinic's general check-in desk** is treated as "an animal": the desk records its weight and age (common `Animal` features). The dog can still bark — but the check-in form has no "bark" field. The dog has not changed; the **form (reference type)** only shows general animal details.

---

## 3. Downcasting

### 3.1 Interview Definition

> **Downcasting is the conversion of a superclass reference back into a subclass reference, such as `Dog d = (Dog) a;`. It must be done explicitly using a cast operator, because it is not always safe: it succeeds only if the actual object is an instance of the target subclass (or its subclass). If it is not, the JVM throws a `ClassCastException` at runtime. Downcasting is needed to access members that exist only in the subclass, and it should be guarded with `instanceof`, or with pattern matching for `instanceof` in Java 16 and later.**

### 3.2 Understand the Concept

```text
            Upcasting (implicit, always safe)
     Dog ─────────────────────────────────────► Animal
     Dog ◄───────────────────────────────────── Animal
            Downcasting (explicit, may fail)
```

| Scenario | Compiles? | Runs? |
|---|---|---|
| `Animal a = new Dog(); Dog d = (Dog) a;` | ✅ | ✅ object really is a Dog |
| `Animal a = new Cat(); Dog d = (Dog) a;` | ✅ | ❌ `ClassCastException` |
| `Animal a = new Animal(); Dog d = (Dog) a;` | ✅ | ❌ `ClassCastException` |
| `String s = "x"; Integer i = (Integer) s;` | ❌ inconvertible types (unrelated classes) | — |
| `Object o = "x"; Integer i = (Integer) o;` | ✅ | ❌ `ClassCastException` |

The compiler allows a cast only if it **could** succeed (the types are related). The JVM checks whether it **actually** succeeds.

### 3.3 Java Example — Upcasting, Downcasting and `instanceof`

```java
import java.util.List;

public class CastingDemo {

    static class Animal {
        String name;
        Animal(String name) { this.name = name; }
        void sound() { System.out.println(name + " makes a sound"); }
        void eat()   { System.out.println(name + " eats"); }
    }

    static class Dog extends Animal {
        Dog(String name) { super(name); }
        @Override void sound() { System.out.println(name + " barks"); }
        void fetch() { System.out.println(name + " fetches the ball"); }   // Dog-only method
    }

    static class Cat extends Animal {
        Cat(String name) { super(name); }
        @Override void sound() { System.out.println(name + " meows"); }
        void climb() { System.out.println(name + " climbs the tree"); }    // Cat-only method
    }

    public static void main(String[] args) {
        // Upcasting: implicit
        Animal a1 = new Dog("Bruno");
        Animal a2 = new Cat("Kitty");

        a1.sound();          // Dog's overridden version (dynamic dispatch)
        a1.eat();            // inherited
        // a1.fetch();       // compile error: cannot find symbol (Animal has no fetch())

        // Downcasting: explicit
        Dog d = (Dog) a1;    // safe: the object really is a Dog
        d.fetch();

        // Unsafe downcast
        try {
            Dog wrong = (Dog) a2;      // a2 is a Cat
            wrong.fetch();
        } catch (ClassCastException e) {
            System.out.println("ClassCastException: Cat cannot be cast to Dog");
        }

        // Safe downcasting with instanceof
        List<Animal> zoo = List.of(new Dog("Rex"), new Cat("Tom"), new Animal("Generic"));
        for (Animal animal : zoo) {
            animal.sound();
            if (animal instanceof Dog) {
                Dog dog = (Dog) animal;               // classic style: check, then cast
                dog.fetch();
            } else if (animal instanceof Cat cat) {   // Java 16 pattern matching: check + cast in one step
                cat.climb();
            }
        }

        System.out.println("a1 instanceof Animal: " + (a1 instanceof Animal));
        System.out.println("a1 instanceof Dog: " + (a1 instanceof Dog));
        System.out.println("a1 instanceof Cat: " + (a1 instanceof Cat));
    }
}
```

### 3.4 Code Explanation

- `Animal a1 = new Dog("Bruno");` — upcasting is automatic. `a1.sound()` runs `Dog.sound()` because the object is a `Dog`.
- `a1.fetch()` would not compile: the compiler sees only the `Animal` type, which has no `fetch()`.
- `(Dog) a1` works because the object is a `Dog`. `(Dog) a2` compiles (Cat and Dog are both Animals, so it *might* work), but fails at runtime because the object is a `Cat`.
- `instanceof` checks the **object type** at runtime, so we downcast only when it is safe.
- `animal instanceof Cat cat` — **pattern matching (Java 16)** tests the type and declares a typed variable `cat` in one step; no separate cast is needed.
- `new Animal("Generic")` is neither a Dog nor a Cat, so only `sound()` runs.

### 3.5 Expected Output

```text
Bruno barks
Bruno eats
Bruno fetches the ball
ClassCastException: Cat cannot be cast to Dog
Rex barks
Rex fetches the ball
Tom meows
Tom climbs the tree
Generic makes a sound
a1 instanceof Animal: true
a1 instanceof Dog: true
a1 instanceof Cat: false
```

### 3.6 Interview Answer — "Upcasting vs downcasting"

> "Upcasting means assigning a child class object to a parent class reference, like `Animal a = new Dog()`. It happens automatically and is always safe, because a dog is an animal. Through the parent reference, I can call only the methods declared in the parent, but if those methods are overridden, the child's version runs — that is runtime polymorphism. Downcasting is the opposite: converting the parent reference back to the child type, like `Dog d = (Dog) a`. It must be explicit, and it is risky: if the object is not actually a `Dog`, the JVM throws a `ClassCastException`. So before downcasting we check with `instanceof`, or in Java 16 and later we use pattern matching like `if (a instanceof Dog d)`, which checks and casts in one step. We usually need downcasting only to call child-specific methods; frequent downcasting is often a sign that the design should use polymorphism instead."

---

## 4. Dynamic Method Dispatch

### 4.1 Interview Definition

> **Dynamic method dispatch is the mechanism by which a call to an overridden method is resolved at runtime rather than at compile time. When an overridden method is called through a superclass reference, the JVM determines the actual class of the object the reference points to and executes that class's version of the method. It is how Java implements runtime polymorphism, and it applies only to overridable instance methods — not to static, private or final methods, or to fields.**

### 4.2 How It Works Step by Step

For `Animal a = new Dog(); a.sound();`:

1. **Compile time:** the compiler checks that `Animal` (reference type) has an accessible `sound()` method. It records the call as "invoke `Animal.sound()` virtually" (`invokevirtual`).
2. **Runtime:** the JVM looks at the object `a` points to → it is a `Dog`.
3. The JVM looks up `sound()` in the `Dog` class's method table. `Dog` overrides it → `Dog.sound()` is executed.
4. If `Dog` had not overridden it, the lookup would continue to the parent (`Animal.sound()`).

```text
 a ──► [ Dog object ] ──► class Dog's method table
                              sound() → Dog.sound()     ◄── found here, executed
                              eat()   → Animal.eat()    (inherited entry)
```

### 4.3 Java Example — Dispatch Through a Hierarchy

```java
public class DynamicDispatchDemo {

    static class Employee {
        String name;
        double baseSalary;
        Employee(String name, double baseSalary) { this.name = name; this.baseSalary = baseSalary; }

        double calculateSalary() { return baseSalary; }

        final void printPayslip() {                                   // calls an overridable method
            System.out.printf("%-8s %-10s Rs.%,.0f%n", name, getClass().getSimpleName(), calculateSalary());
        }
    }

    static class Manager extends Employee {
        Manager(String name, double base) { super(name, base); }
        @Override double calculateSalary() { return baseSalary * 1.30; }   // +30% allowance
    }

    static class SeniorManager extends Manager {
        SeniorManager(String name, double base) { super(name, base); }
        @Override double calculateSalary() { return super.calculateSalary() + 20_000; }
    }

    static class Intern extends Employee {
        Intern(String name) { super(name, 15_000); }
        // no override → inherits Employee.calculateSalary()
    }

    public static void main(String[] args) {
        Employee[] staff = {
            new Employee("Asha", 50_000),
            new Manager("Bala", 80_000),
            new SeniorManager("Chitra", 100_000),
            new Intern("Dev")
        };

        double total = 0;
        for (Employee e : staff) {           // same call, different methods at runtime
            e.printPayslip();
            total += e.calculateSalary();
        }
        System.out.printf("Total payroll: Rs.%,.0f%n", total);
    }
}
```

**Expected Output:**

```text
Asha     Employee   Rs.50,000
Bala     Manager    Rs.104,000
Chitra   SeniorManager Rs.150,000
Dev      Intern     Rs.15,000
Total payroll: Rs.319,000
```

**Explanation:**
- The loop variable is always `Employee`, but `calculateSalary()` runs the version of the **actual object**: Employee → 50,000; Manager → 80,000 × 1.3 = 104,000; SeniorManager → (100,000 × 1.3) + 20,000 = 150,000; Intern → inherited version → 15,000.
- `printPayslip()` is `final` in `Employee`, yet inside it, `calculateSalary()` is dispatched dynamically — the parent's code calls the child's method.
- `SeniorManager` uses `super.calculateSalary()` to build on `Manager`'s logic (dispatch goes one level up explicitly).
- Adding a new employee type requires **no change** to the payroll loop.

### 4.4 What Is NOT Dynamically Dispatched

| Member | Resolved by | Reason |
|---|---|---|
| Overridden instance methods | **Object type (runtime)** | Dynamic dispatch |
| Static methods | Reference type (compile time) | Belong to the class; hidden, not overridden |
| Private methods | Compile time | Not inherited/overridden |
| Final methods | Compile time (can be) | Cannot be overridden, so only one version |
| Fields | Reference type (compile time) | Fields are not polymorphic |
| Overloaded method selection | Compile time (declared argument types) | Overload resolution |

### 4.5 Interview Answer — "What is dynamic method dispatch?"

> "Dynamic method dispatch is the process of deciding at runtime which overridden method to call. When a parent class reference points to a child class object and we call an overridden method, the compiler only checks that the method exists in the parent type. At runtime, the JVM checks the actual object type and calls that class's version. For example, if `Employee e = new Manager()`, then `e.calculateSalary()` runs the Manager's version. This is how Java achieves runtime polymorphism. It works only for overridden instance methods — static, private and final methods, and fields, are resolved at compile time based on the reference type."

---

## 5. Pattern Matching (Java 16 and Java 21)

```java
public class PatternMatchingDemo {

    sealed interface Shape permits Circle, Rectangle, Square { }
    record Circle(double radius) implements Shape { }
    record Rectangle(double length, double width) implements Shape { }
    record Square(double side) implements Shape { }

    // Java 21: switch with type patterns and record patterns
    static double area(Shape shape) {
        return switch (shape) {
            case Circle c -> Math.PI * c.radius() * c.radius();
            case Rectangle(double l, double w) -> l * w;           // record pattern: deconstruct
            case Square s when s.side() == 0 -> 0;                 // guarded pattern
            case Square s -> s.side() * s.side();
        };
    }

    // Java 16: pattern matching for instanceof
    static String describe(Object obj) {
        if (obj instanceof String s && !s.isEmpty()) {
            return "String of length " + s.length();
        } else if (obj instanceof Integer i) {
            return "Integer, doubled = " + (i * 2);
        }
        return "Other: " + obj;
    }

    public static void main(String[] args) {
        System.out.printf("%.2f%n", area(new Circle(1)));
        System.out.printf("%.2f%n", area(new Rectangle(2, 5)));
        System.out.printf("%.2f%n", area(new Square(3)));
        System.out.println(describe("Java"));
        System.out.println(describe(21));
        System.out.println(describe(3.5));
    }
}
```

**Expected Output:**

```text
3.14
10.00
9.00
String of length 4
Integer, doubled = 42
Other: 3.5
```

**Explanation:** Pattern matching combines the type check and the cast safely. With a **sealed** interface, the compiler knows all subtypes, so the `switch` is exhaustive without `default`. Record patterns (`Rectangle(double l, double w)`) extract components directly (Java 21).

> Even with pattern matching, prefer **polymorphism** (an `area()` method in each shape) when you own the classes. Pattern matching is most useful for data-oriented code or types you cannot modify.

---

## 6. Interview Questions with Complete Answers

### Basic Level

**Q1. Is upcasting implicit or explicit?**

Implicit. The compiler performs it automatically because a subclass object is always an instance of its superclass. You can write the cast explicitly (`(Animal) dog`), but it is unnecessary.

**Q2. When does `ClassCastException` occur?**

At runtime, when you downcast a reference to a class that the actual object is not an instance of — for example, casting an object that is a `Cat` to `Dog`, or an `Object` holding a `String` to `Integer`.

**Q3. What does `instanceof` return for `null`?**

`false`. `null instanceof AnyType` is always false and never throws an exception, which makes `instanceof` a safe check before casting.

### Intermediate Level

**Q4. After upcasting, can we call child-specific methods?**

Not through the parent reference. The compiler allows only methods declared in the reference type. You must downcast first (after checking with `instanceof`) to call child-only methods.

**Q5. Why does `(Dog) a2` compile when `a2` holds a `Cat`?**

The compiler only knows the reference type is `Animal`, and an `Animal` reference *could* point to a `Dog`, so the cast is possible in principle. Only the JVM knows the actual object at runtime, and it throws `ClassCastException`. If the types are completely unrelated (e.g., `String` to `Integer`), the compiler rejects the cast immediately.

**Q6. What is the difference between `instanceof` and `getClass()`?**

`obj instanceof Animal` is true for `Animal` **and all its subclasses**. `obj.getClass() == Animal.class` is true only if the object's exact class is `Animal`. `instanceof` also handles `null` (false), while `getClass()` on `null` throws `NullPointerException`.

### Advanced Level

**Q7. What is the output?**

```java
public class DispatchTrick {
    static class A {
        String name = "A-field";
        static String type() { return "A-static"; }
        String who() { return "A-method"; }
    }

    static class B extends A {
        String name = "B-field";
        static String type() { return "B-static"; }
        @Override String who() { return "B-method"; }
    }

    public static void main(String[] args) {
        A ref = new B();
        System.out.println(ref.name);
        System.out.println(ref.type());
        System.out.println(ref.who());
        System.out.println(((B) ref).name);
    }
}
```

**Expected Output:**

```text
A-field
A-static
B-method
B-field
```

Field access and static methods use the **reference type** (`A`). The overridden instance method uses the **object type** (`B`). After casting to `B`, the field access uses `B`'s field.

**Q8. Is downcasting a bad practice?**

Not always, but frequent `instanceof` + downcast chains are a **code smell**: they usually mean behaviour that should be in an overridden method is written outside the class hierarchy (violating the Open/Closed Principle). Legitimate uses: implementing `equals(Object)`, working with heterogeneous data (JSON, events), and pattern matching over sealed types.

**Q9. How do generics reduce the need for downcasting?**

Before generics, collections stored `Object`, so every `get()` needed a downcast: `String s = (String) list.get(0);`, with a risk of `ClassCastException`. With generics (`List<String>`), the compiler knows the element type and inserts the cast safely, and wrong types are rejected at compile time.

---

## 7. Follow-Up Questions

**F1. Does upcasting create a new object?**

No. It only changes the type of the reference. The same object is referenced with a different "view". `dog == (Animal) dog` is `true`.

**F2. Can you upcast to an interface type?**

Yes. `Runnable r = new MyTask();` or `List<String> list = new ArrayList<>();` — assigning to an interface the class implements is also upcasting.

**F3. Does dynamic dispatch make Java slow?**

Very little. The JVM uses method tables, and the JIT compiler optimises virtual calls using inline caches and, when it sees only one implementation in practice, devirtualises and inlines the call.

---

## 8. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Calling child-only methods via a parent reference | Compile error; downcast after checking. |
| Downcasting without `instanceof` | Risk of `ClassCastException`. |
| Expecting fields or static methods to be dispatched dynamically | They use the reference type. |
| Thinking upcasting changes the object | Only the reference view changes. |
| Using long `instanceof` chains instead of polymorphism | Put behaviour in overridden methods. |

---

## 9. Quick Revision

- **Reference type** → what you can call (compiler). **Object type** → which override runs (JVM).
- **Upcasting:** child → parent, implicit, always safe, enables polymorphism.
- **Downcasting:** parent → child, explicit `(Child)`, can throw `ClassCastException`; guard with `instanceof`.
- Unrelated class casts → compile error.
- `null instanceof X` → false.
- **Pattern matching:** `if (o instanceof Dog d)` (Java 16); `switch` with type/record patterns and guards (Java 21).
- **Dynamic method dispatch:** runtime selection of overridden instance methods; not for static/private/final methods or fields.

---

[← Previous: Association, Aggregation, Composition](11-Association-Aggregation-Composition.md) | [Back to README](../README.md) | [Next: SOLID Principles →](13-SOLID-Principles-and-OOP-Design.md)
