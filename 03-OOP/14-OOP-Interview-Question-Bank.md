# Chapter 3.14 — OOP Interview Question Bank (with Complete Answers)

[← Previous: SOLID Principles](13-SOLID-Principles-and-OOP-Design.md) | [Back to README](../README.md) | [Next Part: Exception Handling →](../04-Exception-Handling/01-Exceptions-Errors-and-Hierarchy.md)

---

## How to Use This Question Bank

- Questions are grouped: **Fundamentals → Pillars → Classes & Members → Relationships → Design → Output Prediction → Scenarios**.
- Each answer is written so you can **speak it**: definition → explanation → example → key point.
- Detailed teaching for each topic is in Chapters 3.1–3.13 (links are given where useful).
- Cover the answer, try to speak it in 40–60 seconds, then compare.

---

## Section A — OOP Fundamentals

### A1. What is Object-Oriented Programming? Why do we use it?

OOP is a programming approach in which we design software as a set of **objects** that combine **data (state)** and **behaviour (methods)**. Objects are created from **classes**, which act as blueprints. OOP rests on four principles — **encapsulation, inheritance, polymorphism and abstraction**.

We use it because it models real-world problems naturally (a banking app has `Account`, `Customer`, `Transaction` objects), and it makes code **modular** (each class is a separate unit), **reusable** (through inheritance and composition), **secure** (data hidden inside objects), and **maintainable** (changes stay inside a class). Large applications built in Java, like Spring-based back-ends, are organised entirely around these ideas.

### A2. What are the four pillars of OOP? Explain each with one example.

| Pillar | Meaning | Example |
|---|---|---|
| **Encapsulation** | Bundle data and methods; hide data with `private` and expose controlled methods | `BankAccount` with private `balance`, public `deposit()`/`withdraw()` with validation |
| **Inheritance** | A class acquires properties and behaviour of another (`extends`) — IS-A | `Car extends Vehicle` reuses `start()` |
| **Polymorphism** | Same method name, different behaviour — overloading (compile time) and overriding (runtime) | `Shape s = new Circle(); s.area();` runs Circle's `area()` |
| **Abstraction** | Show what an object does, hide how — abstract classes and interfaces | `PaymentMethod` interface with `pay()`; caller doesn't know if it's UPI or card |

### A3. What is the difference between a class and an object?

A **class** is a blueprint — a logical definition of fields, methods and constructors; it does not hold instance data. An **object** is a runtime instance of a class created with `new`, stored in the heap, with its own **state**, **behaviour** from the class, and a unique **identity**. One class can create many objects. Example: `class Student` vs `new Student("Ravi")`.

### A4. What are the characteristics of an object?

**State** — values of its fields at a moment (`name = "Ravi"`, `cgpa = 8.7`). **Behaviour** — what it can do, defined by methods (`study()`, `getCgpa()`). **Identity** — what makes it distinct even when the state is identical; in Java this is its reference (two `new Student("Ravi")` objects are different objects).

### A5. Is Java a pure object-oriented language?

No. Java has 8 **primitive types** that are not objects, and **static** members that can be used without objects. A pure OOP language (like Smalltalk) treats everything as an object. Java is strongly object-oriented, and wrapper classes and autoboxing let primitives behave like objects when needed (e.g., in collections).

### A6. What is the difference between procedural and object-oriented programming?

Procedural programming organises code as **functions** operating on shared data; OOP organises code as **objects** that own their data and behaviour. In procedural code, any function can change global data, which makes large programs fragile. OOP adds data hiding, inheritance and polymorphism, so code is easier to extend and maintain. C is procedural; Java is object-oriented.

---

## Section B — Encapsulation and Abstraction

### B1. What is encapsulation? How is it implemented in Java?

Encapsulation is wrapping data and the methods that operate on it into a single unit (the class) and **restricting direct access** to the data. In Java: make fields `private`, and provide `public` getters, setters or business methods that **validate** changes. Example: a `balance` field can change only through `deposit()` and `withdraw()`, which reject negative amounts and overdrafts. Benefits: data protection, flexibility to change internals, easier debugging, and loose coupling. ([Chapter 3.3](03-Encapsulation.md))

### B2. What is abstraction? How is it different from encapsulation?

Abstraction hides **implementation complexity** and exposes only essential behaviour — **what** an object does. Encapsulation hides **data** and controls access to it — **how** the data is protected.

| Abstraction | Encapsulation |
|---|---|
| Design level | Implementation level |
| Abstract classes, interfaces | `private` fields + methods |
| Hides how something works | Hides internal state |
| `List` interface hides whether it's an array or linked list | `ArrayList` keeps `elementData` private |

They complement each other: abstraction defines a clean public face; encapsulation protects what is behind it.

### B3. How do you achieve 100% abstraction in Java?

Traditionally with an **interface** that contains only abstract methods. Since Java 8, interfaces can include `default` and `static` methods with bodies, so "100%" is not strictly true anymore, but an interface without default methods is still a pure contract. An abstract class gives 0–100% abstraction.

### B4. Can encapsulation exist without getters and setters?

Yes. Encapsulation means controlled access, not specifically getters/setters. Business methods (`deposit`, `changePassword(old, new)`) or immutable objects (no modifiers at all) give **better** encapsulation than generating a setter for every field. A class with public setters that accept any value is technically encapsulated but practically unprotected.

### B5. What is a data class / JavaBean / POJO?

- **POJO** (Plain Old Java Object): a simple class not tied to any framework.
- **JavaBean**: a POJO following conventions — public no-arg constructor, private fields, getters/setters (`getX`, `setX`, `isX`), usually `Serializable`.
- **Record** (Java 16): a concise immutable data class — `record Point(int x, int y) {}` — with auto-generated constructor, accessors, `equals`, `hashCode`, `toString`.

---

## Section C — Inheritance

### C1. What is inheritance? What are its types in Java?

Inheritance lets a class (child) acquire the fields and methods of another class (parent) using `extends`, representing an **IS-A** relationship. It provides code reuse and is the basis of runtime polymorphism. Types supported with classes: **single** (B extends A), **multilevel** (C extends B extends A), **hierarchical** (B and C extend A). **Multiple** and **hybrid** inheritance are supported only through **interfaces**. ([Chapter 3.4](04-Inheritance-and-Types.md))

### C2. Why does Java not support multiple inheritance with classes?

To avoid the **diamond problem**: if class D extended both B and C, and both overrode a method from A, it would be ambiguous which version D inherits. It would also complicate constructor chaining and duplicate state from A. Java allows multiple inheritance of type through interfaces; if two interfaces provide the same default method, the class must override it and can choose with `X.super.method()`.

### C3. What is not inherited by a subclass?

Constructors (they are called with `super()`), private members (they exist in the object but are not accessible), and static members are not overridden (only hidden). Default (package-private) members are not accessible to subclasses in other packages.

### C4. What is the order of constructor execution in multilevel inheritance?

Top to bottom. For `C extends B extends A`, `new C()` calls `C()`, whose first statement calls `B()` (via `super()`), whose first statement calls `A()`. So the **bodies** execute in the order **A → B → C**. Static blocks run once, also parent first, when classes are first initialised.

### C5. What is the difference between IS-A and HAS-A?

IS-A is inheritance (`Manager extends Employee`) — tight coupling, polymorphism built in. HAS-A is association/composition (`Car` has an `Engine` field) — loose coupling, behaviour can change at runtime. Rule: use IS-A only for true, stable "is a kind of" relationships where the child can fully substitute the parent; otherwise prefer HAS-A.

### C6. What is the use of the `super` keyword?

`super(...)` calls the parent constructor (must be the first statement); `super.method()` calls the parent's version of an overridden method; `super.field` accesses a parent field hidden by a child field. It cannot be used in static contexts.

### C7. Can a class extend itself or can two classes extend each other?

No. Both cause a compile error: "cyclic inheritance involving ...".

### C8. Why should we favour composition over inheritance?

Inheritance tightly couples the child to the parent's implementation (fragile base class problem), exposes all inherited methods even when they don't fit (`Stack extends Vector` allows `add(index, e)`), and allows only one parent. Composition holds a reference to another object and delegates to it, giving loose coupling, a controlled API, and the ability to swap behaviour at runtime (Strategy pattern). ([Chapter 3.11](11-Association-Aggregation-Composition.md))

---

## Section D — Polymorphism

### D1. What is polymorphism? What are its types?

Polymorphism means "many forms" — the same method call behaves differently depending on context. **Compile-time polymorphism** is method overloading: same name, different parameters, resolved by the compiler. **Runtime polymorphism** is method overriding: a parent reference calls an overridden method, and the JVM runs the version of the actual object. ([Chapter 3.5](05-Polymorphism-Overloading-vs-Overriding.md))

### D2. Overloading vs overriding — give five differences.

| Overloading | Overriding |
|---|---|
| Same name, **different parameters** | Same name, **same parameters** |
| Compile-time polymorphism | Runtime polymorphism |
| Same class (or subclass) | Parent–child classes |
| Return type can differ freely | Same or covariant return type |
| Any access modifier | Same or wider access |
| `static/private/final` can be overloaded | `static/private/final` cannot be overridden |

### D3. List the rules of method overriding.

Same name and parameters; return type same or a subtype (covariant); access same or less restrictive; cannot throw new or broader **checked** exceptions (unchecked are fine); `final`, `static` and `private` methods cannot be overridden; constructors cannot be overridden; inheritance (or interface implementation) is required; use `@Override` to get compiler checking.

### D4. Can we override static methods? What is method hiding?

No. If a child declares a static method with the same signature as the parent's static method, it **hides** it. Which version runs depends on the **reference type** at compile time: `Parent p = new Child(); p.staticMethod();` runs `Parent.staticMethod()`. Overriding (instance methods) depends on the object type.

### D5. What is dynamic method dispatch?

The runtime mechanism that selects which overridden method to execute based on the actual object type when the call is made through a parent reference. The compiler only checks the method exists in the reference type; the JVM looks up the implementation in the object's class (method table). ([Chapter 3.12](12-Upcasting-Downcasting-Dynamic-Dispatch.md))

### D6. Can we achieve runtime polymorphism with data members?

No. Fields are resolved by the reference type at compile time. Only overridden instance methods are polymorphic. That's another reason to keep fields private and access them through methods.

### D7. What is a covariant return type?

An overriding method can return a **subclass** of the return type declared in the parent method (Java 5+). Example: `Animal reproduce()` in parent, `Dog reproduce()` in child. It removes the need for casting by callers that use the child type.

### D8. Can an overriding method throw a checked exception that the parent doesn't declare?

No — compile error. It may throw the same checked exceptions, narrower ones (subclasses), none at all, or any unchecked exceptions. Reason: code using the parent reference only prepared for the parent's declared exceptions.

---

## Section E — Abstract Classes and Interfaces

### E1. What is an abstract class? Can it have a constructor?

A class declared `abstract` that cannot be instantiated; it may contain abstract methods (no body) and concrete methods, fields, constructors and static members. **Yes**, it can have constructors — they initialise its fields and run when a concrete subclass object is created via `super()`. ([Chapter 3.6](06-Abstraction-and-Abstract-Classes.md))

### E2. What is an interface? What can it contain in modern Java?

A contract that classes implement. It can contain `public abstract` methods (default), `public static final` constants, **default** and **static** methods (Java 8), **private** methods (Java 9), and can be **sealed** (Java 17). No constructors, no instance fields. ([Chapter 3.7](07-Interfaces-and-Multiple-Inheritance.md))

### E3. Abstract class vs interface — when do you use which?

| Abstract class | Interface |
|---|---|
| Single inheritance | Multiple implementation |
| Can have state and constructors | No state, no constructors |
| Any access modifiers | Public API (+ private helpers) |
| IS-A for closely related classes | CAN-DO capability, even unrelated classes |

Use an **abstract class** for a family of related classes sharing code and state (e.g., `Vehicle`, Template Method). Use an **interface** for capabilities and contracts (`Comparable`, `Payable`), for multiple inheritance, and to program to abstractions (dependency injection). Often both: `List` → `AbstractList` → `ArrayList`.

### E4. Why were default methods introduced?

To add new methods to existing interfaces without breaking existing implementations — e.g., `Iterable.forEach()`, `Collection.stream()`, `List.sort()` in Java 8. Without default methods, every class implementing these interfaces would have failed to compile.

### E5. What happens if a class implements two interfaces with the same default method?

Compile error unless the class overrides the method. Inside the override it can call `InterfaceA.super.method()` and/or `InterfaceB.super.method()`. If a superclass already provides that method, the class version wins automatically ("class wins" rule).

### E6. Can an abstract class have no abstract methods? Can an interface have no methods?

Yes to both. An abstract class with no abstract methods simply prevents instantiation. An interface with no methods is a **marker interface** (`Serializable`, `Cloneable`, `RandomAccess`).

### E7. Why can't abstract methods be `private`, `static` or `final`?

They must be overridden by subclasses. `private` methods are invisible to subclasses, `static` methods can't be overridden (only hidden), and `final` methods can't be overridden at all. Each combination contradicts `abstract`.

---

## Section F — Constructors, `this`, `static`, `final`, Access

### F1. What is a constructor? How is it different from a method?

A special block that initialises a new object; same name as the class; no return type; called automatically by `new`; not inherited; cannot be `static`, `final` or `abstract`. A method performs operations, has a return type, is called explicitly, can be inherited and overridden. ([Chapter 3.2](02-Constructors-this-and-super.md))

### F2. When does Java provide a default constructor?

Only when the class declares **no** constructor. It is a no-arg constructor that calls `super()`. If you write any constructor, the default is not added, so `new X()` fails unless you write a no-arg constructor.

### F3. What is constructor chaining?

Calling one constructor from another — `this(...)` within the same class, `super(...)` to the parent. It must be the first statement, and a constructor can't have both. It avoids duplicate initialisation code by funnelling all constructors into one main constructor.

### F4. What is the `this` keyword used for?

To refer to the current object: distinguish fields from parameters (`this.name = name`), call another constructor (`this(...)`), pass the current object (`register(this)`), and return it for method chaining (`return this`). Not available in static contexts.

### F5. Explain `static` with variables, methods and blocks.

Static members belong to the class: a static variable has one shared copy; a static method is called on the class and cannot use instance members or `this`; a static block runs once when the class is initialised. Uses: counters, constants, utility methods, factory methods, `main`. ([Chapter 3.9](09-static-and-final.md))

### F6. Explain `final` with variables, methods and classes.

A final variable is assigned once (a final reference cannot point elsewhere, but the object may still change); a final method cannot be overridden; a final class cannot be extended (`String`). `static final` creates constants.

### F7. Explain the four access modifiers.

`private` — same class only. Default — same package. `protected` — same package plus subclasses in other packages (through inheritance). `public` — everywhere. Top-level classes can only be `public` or default. Best practice: start with `private`. ([Chapter 3.8](08-Access-Modifiers.md))

### F8. Why is the `main` method static and public?

`public` so the JVM, outside the class, can call it; `static` so it can be called without creating an object, since no object exists at startup.

---

## Section G — Object Class and Equality

### G1. Why must `hashCode()` be overridden when `equals()` is overridden?

Contract: equal objects must have equal hash codes. Hash collections first use `hashCode()` to find a bucket, then `equals()` within it. If only `equals()` is overridden, equal objects usually land in different buckets — `HashSet` keeps duplicates and `HashMap.get()` returns `null`. Use the same fields in both, e.g., `Objects.hash(id, name)`. ([Chapter 3.10](10-Object-Class-equals-hashCode-toString.md))

### G2. What is the `equals()` contract?

Reflexive (`x.equals(x)`), symmetric (`x.equals(y)` ⇔ `y.equals(x)`), transitive, consistent (same result if unchanged), and `x.equals(null)` is false.

### G3. Shallow copy vs deep copy?

A shallow copy copies field values — primitives are copied, but references point to the **same** inner objects (default `Object.clone()`). A deep copy also copies the referenced objects, so the copy is fully independent. Prefer copy constructors or immutable objects over `clone()`.

### G4. What methods does the `Object` class have?

`toString()`, `equals()`, `hashCode()`, `getClass()`, `clone()`, `finalize()` (deprecated), `wait()` (3 overloads), `notify()`, `notifyAll()`.

---

## Section H — Relationships and Design

### H1. Association vs aggregation vs composition?

Association: a general link between independent objects (Teacher–Student). Aggregation: weak whole–part; parts exist independently and can be shared (Team–Player). Composition: strong whole–part; the container owns the parts, which die with it (House–Room). In code, aggregated parts are passed in; composed parts are created inside. ([Chapter 3.11](11-Association-Aggregation-Composition.md))

### H2. Explain SOLID briefly.

Single Responsibility (one reason to change), Open/Closed (extend without modifying), Liskov Substitution (subtypes must be substitutable), Interface Segregation (small, focused interfaces), Dependency Inversion (depend on abstractions; inject implementations). ([Chapter 3.13](13-SOLID-Principles-and-OOP-Design.md))

### H3. What is tight coupling vs loose coupling?

Tight coupling: a class depends directly on another concrete class's details (`new MySqlDatabase()` inside a service) — changes ripple, testing is hard. Loose coupling: classes depend on interfaces and receive implementations from outside — easy to change and test. Achieved through abstraction, encapsulation and dependency injection.

### H4. What is an immutable class? How do you create one?

A class whose objects cannot change after creation (`String`, `Integer`, `LocalDate`). Steps: make the class `final`; all fields `private final`; no setters; initialise everything in the constructor; defensively copy mutable inputs and outputs; or simply use a `record` (with care for mutable components). Benefits: thread safety, safe sharing, safe hash keys. ([Part 10, Chapter 8](../10-Additional-Java-Topics/08-Immutable-Classes-and-Records.md))

### H5. How would you implement a Singleton? What can break it?

Use the holder idiom or an enum (`enum Config { INSTANCE; }`). It can be broken by **reflection** (calling the private constructor via `setAccessible`), **serialization** (deserialization creates a new object unless `readResolve()` returns the instance), **cloning**, and multiple class loaders. An enum singleton is immune to reflection and serialization issues.

---

## Section I — Output Prediction (Practise Speaking the Reason)

### I1. What is the output?

```java
public class Q_I1 {
    static class A { void print() { System.out.println("A"); } }
    static class B extends A { @Override void print() { System.out.println("B"); } }

    public static void main(String[] args) {
        A obj = new B();
        obj.print();
        A obj2 = new A();
        obj2.print();
    }
}
```

**Expected Output:**

```text
B
A
```

`obj` refers to a `B` object, so dynamic dispatch runs `B.print()`. `obj2` is an actual `A` object.

### I2. What is the output?

```java
public class Q_I2 {
    static class Parent {
        Parent() { System.out.println("Parent()"); }
        Parent(String s) { System.out.println("Parent(" + s + ")"); }
    }
    static class Child extends Parent {
        Child() { this("x"); System.out.println("Child()"); }
        Child(String s) { super(s); System.out.println("Child(" + s + ")"); }
    }

    public static void main(String[] args) {
        new Child();
    }
}
```

**Expected Output:**

```text
Parent(x)
Child(x)
Child()
```

`Child()` calls `this("x")` → `Child(String)` calls `super("x")` → `Parent(String)` prints first; then `Child(x)`; then control returns to finish `Child()`. `Parent()` is never called.

### I3. What is the output?

```java
public class Q_I3 {
    static class Base {
        static void greet() { System.out.println("Base static"); }
        void hello() { System.out.println("Base instance"); }
    }
    static class Derived extends Base {
        static void greet() { System.out.println("Derived static"); }
        @Override void hello() { System.out.println("Derived instance"); }
    }

    public static void main(String[] args) {
        Base b = new Derived();
        b.greet();
        b.hello();
    }
}
```

**Expected Output:**

```text
Base static
Derived instance
```

Static methods are hidden, so the reference type (`Base`) decides; instance methods are overridden, so the object type (`Derived`) decides.

### I4. What is the output?

```java
public class Q_I4 {
    static class Counter {
        static int shared = 0;
        int own = 0;
        Counter() { shared++; own++; }
    }

    public static void main(String[] args) {
        Counter c1 = new Counter();
        Counter c2 = new Counter();
        Counter c3 = new Counter();
        System.out.println(c1.own + " " + c3.own + " " + Counter.shared);
    }
}
```

**Expected Output:**

```text
1 1 3
```

Each object has its own `own` (always 1), while `shared` is one copy incremented three times.

### I5. What is the output?

```java
public class Q_I5 {
    static class Shape {
        String name = "Shape";
        String getName() { return name; }
    }
    static class Circle extends Shape {
        String name = "Circle";
        @Override String getName() { return name; }
    }

    public static void main(String[] args) {
        Shape s = new Circle();
        System.out.println(s.name + " " + s.getName());
    }
}
```

**Expected Output:**

```text
Shape Circle
```

Field access uses the reference type (`Shape.name`); the overridden method uses the object type and returns `Circle.name`.

### I6. Will this compile? If yes, what is the output?

```java
public class Q_I6 {
    interface Greeter { default String greet() { return "Hello from Greeter"; } }
    static class Person { public String greet() { return "Hello from Person"; } }
    static class Student extends Person implements Greeter { }

    public static void main(String[] args) {
        System.out.println(new Student().greet());
    }
}
```

**Expected Output:**

```text
Hello from Person
```

It compiles. The "class wins" rule: a method inherited from a superclass takes priority over an interface default method. (`Person.greet()` must be `public`, otherwise it would be a compile error for weaker access.)

### I7. What is the output?

```java
public class Q_I7 {
    static class Animal {
        Animal() { describe(); }
        void describe() { System.out.println("Animal"); }
    }
    static class Dog extends Animal {
        private String breed = "Labrador";
        Dog() { describe(); }
        @Override void describe() { System.out.println("Dog: " + breed); }
    }

    public static void main(String[] args) {
        new Dog();
    }
}
```

**Expected Output:**

```text
Dog: null
Dog: Labrador
```

During `Animal()`, the overridden `Dog.describe()` runs, but `breed` is not yet initialised (field initialisers run after `super()` returns) → `null`. Then `Dog()` runs and `breed` is set. Lesson: don't call overridable methods from constructors.

---

## Section J — Scenario-Based Questions

### J1. Design a simple parking lot using OOP. Which classes and principles would you use?

**Classes:** `ParkingLot` (has many `ParkingFloor`s — composition), `ParkingFloor` (has many `ParkingSpot`s), abstract `ParkingSpot` with subclasses `CompactSpot`, `LargeSpot`, `BikeSpot` (inheritance), abstract `Vehicle` with `Car`, `Bike`, `Truck`, `Ticket` (vehicle, spot, entry time), and a `FeeCalculator` interface with strategies such as `HourlyFee`, `FlatFee` (Strategy pattern, OCP).

**Principles:** encapsulation (spot availability changes only via `park()`/`unpark()`), polymorphism (`spot.canFit(vehicle)` overridden per spot type), SRP (fee calculation separate from parking logic), DIP (`ParkingLot` depends on the `FeeCalculator` interface), and a Singleton or a single injected instance for `ParkingLot` if there is only one lot.

### J2. Your `HashSet<Employee>` contains duplicate employees. What is wrong and how do you fix it?

`Employee` probably overrides `equals()` without `hashCode()` (or uses different fields in each, or uses mutable fields that changed after insertion). Fix: override both using the same immutable identity fields (e.g., `employeeId`), and don't modify those fields while the object is in the set. Or make `Employee` a record.

### J3. A new payment type must be added, and the `PaymentService` has a long if-else on the type string. How would you refactor?

Introduce a `PaymentMethod` interface with `pay(amount)`, create one class per type, and let `PaymentService` call `method.pay(amount)`. Use a factory or a `Map<String, PaymentMethod>` to choose the implementation. New types then require only a new class — applying the **Open/Closed Principle** with **runtime polymorphism**.

### J4. You need to log every method call of a service without changing the service class. What OOP technique helps?

Use the **Decorator** (or Proxy) pattern: create `LoggingOrderService implements OrderService` that holds the real `OrderService` (composition) and adds logging before/after delegating. Frameworks like Spring AOP use dynamic proxies for exactly this.

### J5. A teammate wants `Square extends Rectangle`. What do you say?

Explain the **Liskov Substitution** issue: code that sets width and height independently on a `Rectangle` breaks for a `Square`. Suggest a common `Shape` interface with `area()` and separate `Rectangle` and `Square` classes, or immutable shapes created with their dimensions.

### J6. How would you explain polymorphism to a non-technical person?

"Think of a universal remote's 'play' button. Whether it's connected to a TV, a music system or a projector, you press the same button, and each device plays in its own way. The button is the method; each device is a different object responding differently."

---

## Section K — Rapid-Fire (One or Two Lines Each)

| Question | Answer |
|---|---|
| Can we instantiate an abstract class? | No; only through a subclass or anonymous class. |
| Can an interface have a constructor? | No. |
| Can a constructor be private? | Yes — singletons, utility classes. |
| Can a constructor be final/static/abstract? | No. |
| Can we overload `main`? | Yes; JVM calls only `main(String[])`. |
| Can we override `main`? | No; it's static (hiding only). |
| Can we override a private method? | No. |
| Can a final method be overloaded? | Yes. |
| Default value of an object reference field? | `null`. |
| Root class of all classes? | `java.lang.Object`. |
| Keyword to inherit a class? | `extends`; interface → `implements`. |
| How many classes can a class extend? | One. |
| How many interfaces can a class implement? | Any number. |
| Is `super()` always called? | Yes — implicitly if not written (unless `this()` is used). |
| `instanceof` on null? | `false`. |
| Are fields polymorphic? | No. |
| Marker interface examples? | `Serializable`, `Cloneable`, `RandomAccess`. |
| Functional interface? | Exactly one abstract method; usable with lambdas. |
| Is `String` final? | Yes. |
| What is `this`? | Reference to the current object. |

---

[← Previous: SOLID Principles](13-SOLID-Principles-and-OOP-Design.md) | [Back to README](../README.md) | [Next Part: Exception Handling →](../04-Exception-Handling/01-Exceptions-Errors-and-Hierarchy.md)
