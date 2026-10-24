# Chapter 3.4 — Inheritance and Types of Inheritance

[← Previous: Encapsulation](03-Encapsulation.md) | [Back to README](../README.md) | [Next: Polymorphism →](05-Polymorphism-Overloading-vs-Overriding.md)

---

## What You Will Learn

- What inheritance is, why it is used, and the IS-A relationship
- The `extends` keyword, superclass and subclass
- What is and is not inherited (private members, constructors, static members)
- Types of inheritance: single, multilevel, hierarchical, multiple, hybrid
- Why Java does not support multiple inheritance of classes (the diamond problem)
- `Object` as the root of all classes
- `final` classes and preventing inheritance; sealed classes (Java 17)
- Advantages, disadvantages, and when **not** to use inheritance

---

## 1. Inheritance

### 1.1 Interview Definition

> **Inheritance is an OOP mechanism in which one class (the subclass, child or derived class) acquires the fields and methods of another class (the superclass, parent or base class). In Java, a class inherits from another class using the `extends` keyword, and it can also inherit abstract behaviour from interfaces using `implements`. Inheritance represents an IS-A relationship — for example, a `Car` IS-A `Vehicle`. It promotes code reusability, allows a subclass to add new members or override inherited methods, and is the foundation for runtime polymorphism. Java supports single, multilevel and hierarchical inheritance with classes, but not multiple inheritance of classes.**

### 1.2 Understand the Concept

```java
class Vehicle {               // superclass / parent / base
    String brand;
    void start() { ... }
}

class Car extends Vehicle {   // subclass / child / derived
    int doors;                // new field
    void openTrunk() { ... }  // new method
}
```

- `Car` automatically gets `brand` and `start()` from `Vehicle` — **no copying of code**.
- `Car` can **add** its own members (`doors`, `openTrunk()`).
- `Car` can **override** (redefine) an inherited method to behave differently.
- A `Car` object contains both parts: the `Vehicle` part and the `Car` part.

**IS-A test:** Inheritance is correct only if "Child **is a** Parent" makes sense.
- `Car` is a `Vehicle` ✅ · `Dog` is an `Animal` ✅ · `Manager` is an `Employee` ✅
- `Car` is an `Engine` ❌ → a car **has an** engine → use **composition** instead ([Chapter 3.11](11-Association-Aggregation-Composition.md)).

**What is inherited?**

| Member | Inherited? | Notes |
|---|---|---|
| `public` and `protected` fields/methods | ✅ Yes | |
| Default (package-private) members | ✅ Only if the subclass is in the **same package** | |
| `private` fields/methods | ❌ Not accessible | They exist inside the object, but the subclass cannot access them directly — use public/protected getters |
| Constructors | ❌ No | Called using `super(...)` |
| Static members | Accessible through the subclass, but **not overridden** (they can be hidden) | They belong to the class |
| Initializer blocks | ❌ No | Each class runs its own |

### 1.3 Real-World Example

Think of a **family**. Children inherit features from parents — surname, eye colour, family business. Children also have their own unique skills (new methods) and may do things differently from their parents (overriding). But children don't automatically get access to their parents' personal diary (private members).

In software: every **smartphone** is a **phone** (can call and send SMS — inherited behaviour) but adds a camera, apps and internet (new behaviour).

### 1.4 Java Example — Single Inheritance

```java
public class InheritanceDemo {

    static class Vehicle {
        protected String brand;
        private int engineNumber = 4521;          // private: not accessible in subclass

        Vehicle(String brand) {
            this.brand = brand;
            System.out.println("Vehicle constructor");
        }

        void start() {
            System.out.println(brand + " vehicle starting...");
        }

        void fuel() {
            System.out.println(brand + " uses petrol");
        }

        int getEngineNumber() {                   // public access to private data
            return engineNumber;
        }
    }

    static class ElectricCar extends Vehicle {
        int batteryKwh;

        ElectricCar(String brand, int batteryKwh) {
            super(brand);                         // initialise the inherited part
            this.batteryKwh = batteryKwh;
            System.out.println("ElectricCar constructor");
        }

        @Override
        void fuel() {                             // overriding: new behaviour
            System.out.println(brand + " uses a " + batteryKwh + " kWh battery");
        }

        void charge() {                           // new method
            System.out.println(brand + " is charging");
        }
    }

    public static void main(String[] args) {
        ElectricCar car = new ElectricCar("Tata Nexon EV", 40);
        car.start();                              // inherited unchanged
        car.fuel();                               // overridden
        car.charge();                             // child's own method
        System.out.println("Engine no. via getter: " + car.getEngineNumber());
        System.out.println("car instanceof Vehicle? " + (car instanceof Vehicle));
        System.out.println("car instanceof Object? " + (car instanceof Object));
        System.out.println("Parent class: " + car.getClass().getSuperclass().getSimpleName());
    }
}
```

### 1.5 Code Explanation

- `class ElectricCar extends Vehicle` — `ElectricCar` inherits `brand`, `start()`, `fuel()` and `getEngineNumber()`.
- `super(brand)` — the parent constructor runs first, so the inherited part is initialised before the child part.
- `start()` is used exactly as inherited (**reuse**).
- `fuel()` is **overridden** — `@Override` asks the compiler to confirm we really are overriding a parent method.
- `engineNumber` is `private`: `ElectricCar` cannot write `engineNumber` directly, but it exists inside the object and is reachable through the inherited public getter.
- Every class ultimately extends `Object`, so `car instanceof Object` is true.

### 1.6 Expected Output

```text
Vehicle constructor
ElectricCar constructor
Tata Nexon EV vehicle starting...
Tata Nexon EV uses a 40 kWh battery
Tata Nexon EV is charging
Engine no. via getter: 4521
car instanceof Vehicle? true
car instanceof Object? true
Parent class: Vehicle
```

### 1.7 Interview Answer — "What is inheritance?"

> "Inheritance is a mechanism where one class acquires the properties and behaviour of another class. The class that is inherited from is called the superclass or parent, and the class that inherits is called the subclass or child. In Java we use the `extends` keyword. For example, if `ElectricCar extends Vehicle`, the electric car automatically gets the vehicle's fields and methods like `start()`, and it can add new methods like `charge()` or override existing methods like `fuel()`. Inheritance represents an **IS-A** relationship, it gives us **code reusability** and it is the basis for **runtime polymorphism**. Private members and constructors are not inherited, and the child calls the parent constructor using `super()`. Java supports single, multilevel and hierarchical inheritance with classes, but not multiple inheritance of classes, to avoid the diamond problem — multiple inheritance is supported only through interfaces."

---

## 2. Types of Inheritance

### 2.1 Overview

```text
1. Single            2. Multilevel         3. Hierarchical
   A                    A                        A
   │                    │                    ┌───┼───┐
   B                    B                    B   C   D
                        │
                        C

4. Multiple (NOT with classes)     5. Hybrid (combination; only via interfaces)
   A     B                              A
    \   /                             /   \
      C                              B     C
                                      \   /
                                        D
```

| Type | Description | Supported in Java with classes? |
|---|---|---|
| **Single** | One child extends one parent | ✅ Yes |
| **Multilevel** | A chain: C extends B, B extends A | ✅ Yes |
| **Hierarchical** | Many children extend one parent | ✅ Yes |
| **Multiple** | One child extends two or more parents | ❌ No (✅ with interfaces) |
| **Hybrid** | Combination of two or more types | ❌ With classes only if it needs multiple; ✅ with interfaces |

### 2.2 Java Example — Single, Multilevel and Hierarchical

```java
public class InheritanceTypesDemo {

    // ----- Multilevel: Animal → Mammal → Dog -----
    static class Animal {
        void eat() { System.out.println("Animal eats"); }
    }

    static class Mammal extends Animal {           // single: Mammal extends Animal
        void breathe() { System.out.println("Mammal breathes air"); }
    }

    static class Dog extends Mammal {              // multilevel: Dog → Mammal → Animal
        void bark() { System.out.println("Dog barks"); }
    }

    // ----- Hierarchical: Shape → Circle, Square -----
    static class Shape {
        String name;
        Shape(String name) { this.name = name; }
        double area() { return 0; }
        void print() { System.out.printf("%s area = %.2f%n", name, area()); }
    }

    static class Circle extends Shape {
        double r;
        Circle(double r) { super("Circle"); this.r = r; }
        @Override double area() { return Math.PI * r * r; }
    }

    static class Square extends Shape {
        double side;
        Square(double side) { super("Square"); this.side = side; }
        @Override double area() { return side * side; }
    }

    public static void main(String[] args) {
        System.out.println("--- Multilevel ---");
        Dog dog = new Dog();
        dog.eat();          // from Animal (grandparent)
        dog.breathe();      // from Mammal (parent)
        dog.bark();         // own method

        System.out.println("--- Hierarchical ---");
        Shape[] shapes = { new Circle(2), new Square(3) };
        for (Shape s : shapes) {
            s.print();      // each child provides its own area()
        }
    }
}
```

**Expected Output:**

```text
--- Multilevel ---
Animal eats
Mammal breathes air
Dog barks
--- Hierarchical ---
Circle area = 12.57
Square area = 9.00
```

**Explanation:**
- **Multilevel:** `Dog` gets `eat()` from `Animal` through `Mammal`. A class inherits from all its ancestors.
- **Hierarchical:** `Circle` and `Square` share the common `name` field and `print()` method from `Shape`, but each overrides `area()`. The `print()` method in the parent calls `area()`, and the **child's version runs** — this is runtime polymorphism (next chapter).

---

## 3. Why Java Does Not Support Multiple Inheritance of Classes

### 3.1 Interview Definition

> **Java does not allow a class to extend more than one class in order to avoid the diamond problem — an ambiguity that occurs when a class inherits the same method from two parent classes, and the compiler cannot decide which implementation to use. Avoiding multiple class inheritance keeps the language simpler and avoids problems with duplicate state and complex constructor chains. Java supports multiple inheritance of type and behaviour through interfaces, where any conflict between default methods must be resolved explicitly by the implementing class.**

### 3.2 The Diamond Problem

```text
             class A { void show() }
               /                  \
 class B extends A          class C extends A
   void show() {"B"}          void show() {"C"}
               \                  /
           class D extends B, C   ← not allowed in Java
                  d.show() → "B" or "C"?  AMBIGUOUS
```

If `D` could extend both `B` and `C`, then `new D().show()` would have two possible implementations. C++ allows this and needs special rules (virtual inheritance) to handle it. Java's designers chose **simplicity**: a class can have only **one** direct superclass.

<!-- skip-check -->
```java
class B { void show() { System.out.println("B"); } }
class C { void show() { System.out.println("C"); } }
class D extends B, C { }      // COMPILE ERROR: '{' expected
```

### 3.3 How Java Supports Multiple Inheritance — Interfaces

A class can **implement many interfaces**. Before Java 8, interfaces had only abstract methods, so there was no conflict — the class itself provided the single implementation. Since Java 8, interfaces can have `default` methods; if two interfaces provide the same default method, the class **must override it** and can choose one using `InterfaceName.super.method()`:

```java
public class DiamondWithInterfaces {
    interface Camera {
        default String capture() { return "Photo from Camera"; }
    }

    interface Scanner {
        default String capture() { return "Scan from Scanner"; }
    }

    static class SmartPhone implements Camera, Scanner {
        @Override
        public String capture() {                         // must resolve the conflict
            return Camera.super.capture() + " + " + Scanner.super.capture();
        }
    }

    public static void main(String[] args) {
        System.out.println(new SmartPhone().capture());
    }
}
```

**Expected Output:**

```text
Photo from Camera + Scan from Scanner
```

If `SmartPhone` did not override `capture()`, the code would **not compile** ("inherits unrelated defaults for capture()"). Interfaces are covered fully in [Chapter 3.7](07-Interfaces-and-Multiple-Inheritance.md).

---

## 4. The `Object` Class — Root of Every Class

- Every class in Java directly or indirectly extends **`java.lang.Object`**.
- If you don't write `extends`, the compiler adds `extends Object`.
- So every object has methods like `toString()`, `equals()`, `hashCode()`, `getClass()`, `clone()`, `wait()`, `notify()`, `finalize()` (deprecated).
- Details in [Chapter 3.10](10-Object-Class-equals-hashCode-toString.md).

---

## 5. Preventing Inheritance — `final` and Sealed Classes

### 5.1 `final` class

A class declared `final` **cannot be extended**. Examples: `String`, `Integer`, `Math`, `System`. Reasons: security, immutability, and design guarantees.

<!-- skip-check -->
```java
final class Password { }
class StrongPassword extends Password { }   // COMPILE ERROR: cannot inherit from final Password
```

A `final` **method** cannot be overridden, though the class can be extended.

### 5.2 Sealed classes (Java 17)

A **sealed** class allows inheritance **only by specific, listed subclasses**. This gives control in between "anyone can extend" and "no one can extend" (`final`).

```java
public class SealedDemo {

    sealed interface Payment permits Card, Upi, Cash { }

    static final class Card implements Payment { }          // final: cannot be extended further
    static final class Upi implements Payment { }
    static non-sealed class Cash implements Payment { }     // non-sealed: open for extension again

    static String describe(Payment p) {
        return switch (p) {                                 // exhaustive: no default needed
            case Card c -> "Paid by card";
            case Upi u  -> "Paid by UPI";
            case Cash c -> "Paid by cash";
        };
    }

    public static void main(String[] args) {
        System.out.println(describe(new Upi()));
        System.out.println(describe(new Card()));
    }
}
```

**Expected Output:**

```text
Paid by UPI
Paid by card
```

Each permitted subclass must be `final`, `sealed` or `non-sealed`. Because the compiler knows **all** possible subtypes, a `switch` over a sealed type can be **exhaustive** without `default` (Java 21 pattern matching).

---

## 6. Advantages and Disadvantages of Inheritance

| Advantages | Disadvantages |
|---|---|
| Code reusability — write common code once in the parent | **Tight coupling** — child depends on parent's implementation |
| Easier maintenance of shared logic | Changes in the parent can break children ("fragile base class") |
| Enables **method overriding** and **runtime polymorphism** | Deep hierarchies are hard to understand |
| Natural modelling of IS-A relationships | Child inherits **everything**, even methods it should not have |
| Extensibility — add new subclasses without changing existing code | Only one parent class allowed in Java |

### 6.1 When NOT to use inheritance — prefer composition

**"Favour composition over inheritance"** (from the book *Design Patterns* and *Effective Java*).

A famous mistake: `class Stack extends Vector` in the JDK. Because `Stack` IS-A `Vector`, a stack also inherits `add(index, element)` and `remove(index)`, which let you insert or delete in the middle — breaking the rules of a stack.

Use inheritance only when:
1. There is a true, permanent **IS-A** relationship.
2. The child can be used **anywhere** the parent is used without surprising behaviour (**Liskov Substitution Principle**, [Chapter 3.13](13-SOLID-Principles-and-OOP-Design.md)).
3. The parent class was **designed** for extension.

Otherwise, use **composition** (HAS-A): hold a reference to another object and delegate work to it.

---

## 7. Interview Questions with Complete Answers

### Basic Level

**Q1. What is the syntax of inheritance in Java?**

`class Child extends Parent { }` for classes, and `class MyClass implements InterfaceA, InterfaceB { }` for interfaces. An interface can extend other interfaces: `interface C extends A, B { }`. A class can extend one class and implement many interfaces at the same time: `class D extends Parent implements A, B { }`.

**Q2. Which types of inheritance does Java support?**

With classes: **single**, **multilevel** and **hierarchical**. **Multiple** and **hybrid** inheritance are supported only through **interfaces**.

**Q3. Are private members inherited?**

They are **not accessible** in the subclass. Technically, a subclass object contains the private fields of its parent (memory is allocated for them, and the parent's methods use them), but the subclass code cannot access them by name. The JLS says private members are "not inherited". Access them through public/protected getters or methods.

**Q4. Are constructors inherited?**

No. A constructor's name must match its class, so it cannot belong to the subclass. The subclass constructor calls the parent constructor using `super(...)` (implicitly `super()` if not written).

### Intermediate Level

**Q5. Why doesn't Java support multiple inheritance through classes?**

To avoid the **diamond problem** — if a class extended two classes that both define the same method, it would be ambiguous which one to inherit. It would also complicate constructor chaining and create duplicate copies of a common ancestor's fields. Java keeps one direct superclass for simplicity and safety, and allows multiple inheritance of type via interfaces.

**Q6. Can a subclass access a parent's protected member from a different package?**

Yes, but only through **inheritance** — that is, inside the subclass, using `this` or a reference of the subclass type. It cannot access the protected member through a reference of the parent type (`new Parent().protectedField` from another package is not allowed). See [Chapter 3.8](08-Access-Modifiers.md).

**Q7. Can static methods be inherited?**

Static methods are accessible through the subclass name (`Child.staticMethod()`), so in that sense they are inherited. But they **cannot be overridden**. If a subclass declares a static method with the same signature, it **hides** the parent's method. The version called depends on the **reference type** at compile time, not the object type.

**Q8. What is the difference between IS-A and HAS-A?**

IS-A is **inheritance** (`Car extends Vehicle` → a car is a vehicle). HAS-A is **composition/aggregation** (`Car` has an `Engine` field → a car has an engine). IS-A gives tight coupling and polymorphism; HAS-A gives flexibility and loose coupling. Prefer HAS-A unless there is a genuine IS-A relationship.

### Advanced Level

**Q9. What is the fragile base class problem?**

When a parent class is changed (for example, a method's internal implementation now calls another overridable method), subclasses that relied on the old behaviour can break, even though they were not modified. Example: a subclass of `HashSet` that counts additions by overriding both `add()` and `addAll()` gets double counts because `HashSet.addAll()` internally calls `add()`. This is why inheritance across code you don't control is risky, and why composition is often safer.

```java
import java.util.Collection;
import java.util.HashSet;
import java.util.List;

public class FragileBaseClass {
    static class CountingSet<E> extends HashSet<E> {
        int addCount = 0;

        @Override
        public boolean add(E e) {
            addCount++;
            return super.add(e);
        }

        @Override
        public boolean addAll(Collection<? extends E> c) {
            addCount += c.size();
            return super.addAll(c);        // internally calls add() for each element!
        }
    }

    public static void main(String[] args) {
        CountingSet<String> set = new CountingSet<>();
        set.addAll(List.of("a", "b", "c"));
        System.out.println("Expected 3, got " + set.addCount);
    }
}
```

**Expected Output:**

```text
Expected 3, got 6
```

**Q10. What is the maximum depth of inheritance? Is deep inheritance good?**

There is no fixed limit in Java, but deep hierarchies (more than 3–4 levels) are hard to understand and maintain, because a method's behaviour may be defined anywhere up the chain. Prefer shallow hierarchies and composition.

**Q11. Can a class extend itself, or can two classes extend each other?**

No. `class A extends A` and cyclic inheritance (`A extends B`, `B extends A`) cause a compile error: "cyclic inheritance involving A".

---

## 8. Follow-Up Questions

**F1. If a parent and child both have a field with the same name, which is used?**

Field access depends on the **reference type**, not the object type (fields are not polymorphic). `Parent p = new Child(); p.name` uses `Parent.name`. Inside the child, `super.name` accesses the parent's field. Avoid this situation ("field hiding").

**F2. Can we reduce the visibility of an inherited method when overriding?**

No. An overriding method cannot have **weaker** access. A `public` method must stay `public`; a `protected` method can become `public` but not private/default. See the overriding rules in [Chapter 3.5](05-Polymorphism-Overloading-vs-Overriding.md).

**F3. Does a subclass object contain a separate parent object?**

No. Only **one object** is created. That object contains fields from all classes in the hierarchy. `super` is not a separate object — it is a way to refer to the parent-class part of the same object.

---

## 9. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| `class C extends A, B` | Only one superclass allowed. |
| Using inheritance only for code reuse when no IS-A exists | Use composition. |
| Thinking private members don't exist in the child object | They exist but are not accessible. |
| Forgetting `super(...)` when the parent has no no-arg constructor | Compile error. |
| Thinking static methods can be overridden | They are hidden, not overridden. |
| Thinking creating a child object creates a parent object too | Only one object is created. |

---

## 10. Quick Revision

- Inheritance = child acquires parent's members; `extends` (class), `implements` (interface); **IS-A**.
- Inherited: public, protected (and default in the same package). Not accessible: private. Not inherited: constructors.
- Types: single ✅, multilevel ✅, hierarchical ✅, multiple ❌ (classes) / ✅ (interfaces), hybrid via interfaces.
- No multiple class inheritance → avoid **diamond problem**.
- Interface default-method conflict → override and choose with `X.super.method()`.
- Every class extends `Object`.
- `final` class → no subclass; `final` method → no override; **sealed** (Java 17) → only permitted subclasses.
- Prefer **composition over inheritance**; beware fragile base class and `Stack extends Vector`.

---

[← Previous: Encapsulation](03-Encapsulation.md) | [Back to README](../README.md) | [Next: Polymorphism →](05-Polymorphism-Overloading-vs-Overriding.md)
