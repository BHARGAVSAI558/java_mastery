# Chapter 3.6 — Abstraction and Abstract Classes

[← Previous: Polymorphism](05-Polymorphism-Overloading-vs-Overriding.md) | [Back to README](../README.md) | [Next: Interfaces →](07-Interfaces-and-Multiple-Inheritance.md)

---

## What You Will Learn

- What abstraction is and how it differs from encapsulation
- Two ways to achieve abstraction in Java: abstract classes and interfaces
- Abstract classes and abstract methods — all rules
- Constructors, fields and concrete methods in abstract classes
- The **Template Method** pattern (a classic use of abstract classes)
- When to use an abstract class

---

## 1. Abstraction

### 1.1 Interview Definition

> **Abstraction is the OOP principle of hiding the internal implementation details and showing only the essential features or functionality to the user. It focuses on what an object does rather than how it does it. In Java, abstraction is achieved using abstract classes, which provide partial abstraction (0 to 100%) because they can contain both abstract and concrete methods, and interfaces, which define a pure contract of behaviour that implementing classes must provide. Abstraction reduces complexity, separates the "what" from the "how", and allows implementations to change without affecting the code that uses them.**

### 1.2 Understand the Concept

Every time you use something without knowing how it works inside, you are using abstraction:
- You call `list.add(x)` without knowing how `ArrayList` resizes its internal array.
- You call `Collections.sort(list)` without knowing it uses TimSort.
- You call `connection.executeQuery(sql)` without knowing the network protocol of MySQL or PostgreSQL.

**Levels of abstraction in Java:**

| Tool | Abstraction level | Contains |
|---|---|---|
| Concrete class | 0% (everything implemented) | Only concrete methods |
| **Abstract class** | **0–100% (partial)** | Abstract + concrete methods, fields, constructors |
| **Interface** | Close to 100% (pure contract), but Java 8+ allows `default`/`static` methods | Abstract methods, constants, default/static/private methods |

### 1.3 Abstraction vs Encapsulation

| Point | Abstraction | Encapsulation |
|---|---|---|
| Hides | **Implementation complexity** (how) | **Data** (internal state) |
| Focus | What the object does | How data is protected |
| Level | Design level | Implementation level |
| Achieved by | Abstract classes, interfaces | `private` fields + public methods |
| Question it answers | "What should this do?" | "Who can access/change this data?" |
| Example | `Vehicle` with abstract `start()`; caller doesn't know petrol vs electric | `BankAccount` with private `balance` |

They work together: abstraction decides the **public face**; encapsulation **protects the internals** behind that face.

### 1.4 Real-World Example

When you **drive a car**, you use the steering wheel, accelerator and brake (the essential interface). You don't need to know how fuel injection, gear ratios or ABS work internally (hidden implementation). A petrol car and an electric car hide completely different engines behind the same pedals — that is abstraction.

Another example: an **ATM** shows "Withdraw", "Balance", "Transfer". The bank's complex server communication is hidden.

---

## 2. Abstract Classes

### 2.1 Interview Definition

> **An abstract class is a class declared with the `abstract` keyword that cannot be instantiated directly. It is meant to be extended by subclasses. An abstract class can contain abstract methods — methods declared without a body that subclasses must implement — as well as concrete methods with full implementations, instance and static fields, constructors and static blocks. A concrete (non-abstract) subclass must implement all inherited abstract methods; otherwise it must also be declared abstract. Abstract classes are used to provide a common base with shared code and state for closely related classes, while forcing each subclass to supply its own specific behaviour.**

### 2.2 Rules of Abstract Classes and Methods

**Abstract class rules:**
1. Declared with `abstract`: `abstract class Shape { }`.
2. **Cannot be instantiated**: `new Shape()` → compile error.
3. **Can have constructors** — they run when a subclass object is created (via `super()`).
4. Can have **abstract and concrete** methods, **instance and static** fields, `final` methods, static methods, and `main`.
5. Can have **zero** abstract methods (still cannot be instantiated — used to prevent direct object creation).
6. If a class has **at least one** abstract method, the class **must** be declared abstract.
7. A subclass must implement **all** abstract methods, or itself be declared `abstract`.
8. **Cannot be `final`** — `final` prevents extension, abstract requires it (`abstract final class` → compile error).
9. A class can extend only **one** abstract class (single inheritance).

**Abstract method rules:**
1. Declared with `abstract` and **no body** — ends with a semicolon: `abstract double area();`
2. Can exist only inside an abstract class (or interface).
3. **Cannot be** `private` (subclass couldn't implement it), `static` (static methods can't be overridden), `final` (can't be overridden) or `synchronized`/`native`/`strictfp` combined with `abstract`.
4. Can be `public`, `protected` or default (package-private).

### 2.3 Real-World Example

A **"Shape" in geometry** is an abstract idea. Nobody can draw "a shape" — you draw a circle, a rectangle, a triangle. But all shapes have a colour and an area, and all can describe themselves. So:
- `Shape` = abstract class (common colour field, common `describe()` method).
- `area()` = abstract method (each shape calculates it differently).
- `Circle`, `Rectangle` = concrete subclasses.

### 2.4 Java Example

```java
public class AbstractClassDemo {

    static abstract class Shape {
        private final String color;              // fields allowed
        private static int shapesCreated = 0;    // static fields allowed

        Shape(String color) {                    // constructor allowed
            this.color = color;
            shapesCreated++;
            System.out.println("Shape constructor (" + color + ")");
        }

        abstract double area();                  // no body: each subclass must implement
        abstract double perimeter();

        // Concrete method: shared by all shapes, uses abstract methods
        void describe() {
            System.out.printf("%s %s: area=%.2f, perimeter=%.2f%n",
                    color, getClass().getSimpleName(), area(), perimeter());
        }

        static int getShapesCreated() {
            return shapesCreated;
        }
    }

    static class Circle extends Shape {
        private final double radius;

        Circle(String color, double radius) {
            super(color);                        // calls the abstract class constructor
            this.radius = radius;
        }

        @Override double area()      { return Math.PI * radius * radius; }
        @Override double perimeter() { return 2 * Math.PI * radius; }
    }

    static class Rectangle extends Shape {
        private final double length, width;

        Rectangle(String color, double length, double width) {
            super(color);
            this.length = length;
            this.width = width;
        }

        @Override double area()      { return length * width; }
        @Override double perimeter() { return 2 * (length + width); }
    }

    // Partially implemented: still abstract because perimeter() is missing
    static abstract class Polygon extends Shape {
        Polygon(String color) { super(color); }
        @Override double area() { return 0; }
    }

    public static void main(String[] args) {
        // Shape s = new Shape("Red");     // compile error: Shape is abstract; cannot be instantiated

        Shape[] shapes = {
            new Circle("Red", 1.5),
            new Rectangle("Blue", 4, 2.5)
        };

        for (Shape s : shapes) {
            s.describe();                        // concrete method + polymorphic abstract calls
        }
        System.out.println("Shapes created: " + Shape.getShapesCreated());
    }
}
```

### 2.5 Code Explanation

- `Shape` is abstract: it holds common state (`color`), a counter, a constructor and a shared `describe()` method.
- `area()` and `perimeter()` are abstract — `Shape` **forces** every concrete shape to implement them.
- `describe()` is concrete but **calls the abstract methods**. At runtime, the subclass versions run (polymorphism).
- `new Shape("Red")` is not allowed, but a `Shape` reference can point to a `Circle` or `Rectangle` object.
- The `Shape` constructor runs for every subclass object via `super(color)` — proof that abstract classes have working constructors.
- `Polygon` implements only `area()`, so it must still be declared `abstract`.

### 2.6 Expected Output

```text
Shape constructor (Red)
Shape constructor (Blue)
Red Circle: area=7.07, perimeter=9.42
Blue Rectangle: area=10.00, perimeter=13.00
Shapes created: 2
```

### 2.7 Interview Answer — "What is an abstract class?"

> "An abstract class is a class declared with the `abstract` keyword, and it cannot be instantiated directly. It is used as a base class for related subclasses. It can contain **abstract methods**, which have no body and must be implemented by subclasses, and also **concrete methods**, fields, constructors and static members. So it provides **partial abstraction**. For example, a `Shape` abstract class can store the colour and provide a common `describe()` method, while declaring `area()` as abstract, because each shape calculates area differently. Even though we cannot create a `Shape` object, its constructor runs when a `Circle` is created, to initialise the common fields. If a subclass doesn't implement all the abstract methods, it must also be declared abstract. An abstract class cannot be final, and abstract methods cannot be private, static or final."

### 2.8 Interview Answer — "What is abstraction?"

> "Abstraction means hiding the internal implementation and showing only the essential functionality. The user knows **what** an object does, but not **how** it does it. For example, when we call `list.sort()`, we don't need to know the sorting algorithm used inside. In Java, we achieve abstraction using **abstract classes** and **interfaces**. Abstract classes give partial abstraction because they can have both abstract and concrete methods, while interfaces define a contract that implementing classes must follow. Abstraction reduces complexity, improves maintainability, and lets us change the implementation without affecting the code that uses it — for example, switching from a MySQL to a PostgreSQL implementation behind the same interface."

---

## 3. Template Method Pattern — A Classic Use of Abstract Classes

### 3.1 Concept

> **The Template Method pattern defines the skeleton of an algorithm in a method of an abstract class, called the template method, and lets subclasses provide the implementation of specific steps by overriding abstract methods. The template method is usually declared `final` so that subclasses cannot change the order of steps.**

### 3.2 Java Example

```java
public class TemplateMethodDemo {

    static abstract class DataExporter {

        // Template method: fixed sequence of steps
        public final void export() {
            connect();
            String data = fetchData();
            String formatted = format(data);       // varies by subclass
            write(formatted);
            if (needsCompression()) {              // optional step (hook)
                System.out.println("  Compressing output");
            }
            disconnect();
        }

        private void connect()    { System.out.println("  Connecting to database"); }
        private String fetchData(){ return "id=1,name=Ravi"; }
        private void disconnect() { System.out.println("  Disconnected"); }

        protected abstract String format(String raw);      // must implement
        protected abstract void write(String content);     // must implement

        protected boolean needsCompression() {              // hook: optional override
            return false;
        }
    }

    static class CsvExporter extends DataExporter {
        @Override protected String format(String raw) { return raw.replace("=", ",").replace(",name", "\nname"); }
        @Override protected void write(String content) { System.out.println("  Writing CSV:\n" + content); }
    }

    static class JsonExporter extends DataExporter {
        @Override protected String format(String raw) { return "{\"id\":1,\"name\":\"Ravi\"}"; }
        @Override protected void write(String content) { System.out.println("  Writing JSON: " + content); }
        @Override protected boolean needsCompression() { return true; }
    }

    public static void main(String[] args) {
        System.out.println("CSV export:");
        new CsvExporter().export();
        System.out.println("JSON export:");
        new JsonExporter().export();
    }
}
```

**Expected Output:**

```text
CSV export:
  Connecting to database
  Writing CSV:
id,1
name,Ravi
  Disconnected
JSON export:
  Connecting to database
  Writing JSON: {"id":1,"name":"Ravi"}
  Compressing output
  Disconnected
```

**Explanation:**
- `export()` is `final`: the order connect → fetch → format → write → (compress) → disconnect cannot be changed by subclasses.
- Common steps (`connect`, `fetchData`, `disconnect`) are written **once** in the abstract class.
- `format()` and `write()` are abstract → each exporter must provide them.
- `needsCompression()` is a **hook** — a concrete method with a default that subclasses may override.
- This is exactly how frameworks work: `HttpServlet.service()` calls `doGet()`/`doPost()`; `AbstractList` implements most of `List` using your `get()` and `size()`.

---

## 4. When to Use an Abstract Class

Use an **abstract class** when:
1. Several **closely related** classes share **common state (fields)** and **common code**.
2. You need **constructors** to initialise common fields.
3. You want **non-public** members (`protected` helpers, `private` fields).
4. You want to define a **template** of steps (Template Method pattern).
5. There is a clear **IS-A** relationship.

Use an **interface** when:
1. Unrelated classes should share a **capability** (`Comparable`, `Runnable`, `Serializable`).
2. You need **multiple inheritance** of type.
3. You want to define a **contract** without state.

The full **abstract class vs interface** comparison is in [Chapter 3.7](07-Interfaces-and-Multiple-Inheritance.md).

---

## 5. Interview Questions with Complete Answers

### Basic Level

**Q1. Can we create an object of an abstract class?**

No, `new AbstractClass()` is a compile error. But we can create a reference of the abstract type pointing to a concrete subclass object: `Shape s = new Circle(...)`. We can also create an **anonymous subclass** on the spot: `Shape s = new Shape("Red") { double area() { return 0; } double perimeter() { return 0; } };` — this creates an object of an unnamed subclass, not of `Shape` itself.

**Q2. Can an abstract class have a constructor? Why?**

Yes. The constructor initialises the fields defined in the abstract class. It runs whenever a concrete subclass object is created, through the `super(...)` call in the subclass constructor.

**Q3. Is it compulsory for an abstract class to have an abstract method?**

No. An abstract class can have zero abstract methods. Declaring it abstract simply prevents direct instantiation. Example: `HttpServlet` is abstract but provides default implementations for all its `doXxx` methods.

### Intermediate Level

**Q4. Why can't an abstract method be `private`, `static` or `final`?**

An abstract method has no body and exists only to be overridden by subclasses.
- `private` → not visible to subclasses, so it could never be implemented.
- `static` → static methods are not overridden (only hidden), so no subclass could provide the implementation polymorphically.
- `final` → final methods cannot be overridden, which contradicts the purpose of abstract.
All three combinations cause compile errors.

**Q5. Can an abstract class be `final`?**

No. `final` means "cannot be extended", and `abstract` means "must be extended to be useful". Together they make no sense, so `abstract final class` is a compile error.

**Q6. Can an abstract class have a `main` method and be run?**

Yes. `main` is static, so it can be called without an object. You can run an abstract class that has a `main` method; you just cannot create an instance of it inside `main`.

**Q7. What happens if a subclass does not implement all abstract methods?**

Compile error: *"Circle is not abstract and does not override abstract method perimeter() in Shape"*. The fix is to implement the missing methods or declare the subclass `abstract`.

### Advanced Level

**Q8. Can an abstract class implement an interface without implementing its methods?**

Yes. An abstract class can implement an interface and leave some or all of its methods unimplemented. The first **concrete** subclass must implement the remaining methods. This is common in the JDK: `AbstractList implements List` provides many methods, leaving `get()` and `size()` abstract.

**Q9. Can we call an abstract method from a constructor of the abstract class?**

It compiles, and the subclass's implementation runs (dynamic dispatch). But it is dangerous because the subclass's fields are not yet initialised when the parent constructor runs (they will have default values). See the example in [Chapter 3.2, Q11](02-Constructors-this-and-super.md).

**Q10. Give real JDK examples of abstract classes.**

`java.io.InputStream`/`OutputStream`/`Reader`/`Writer` (abstract `read()`/`write()`), `java.util.AbstractList`, `AbstractMap`, `java.lang.Number` (abstract `intValue()`, `doubleValue()` …), `javax.servlet.http.HttpServlet`, `java.util.TimerTask`, `java.lang.Enum` (abstract base of all enums).

---

## 6. Follow-Up Questions

**F1. Is an abstract class 100% abstraction?**

No — it can be anywhere from 0% to 100%. If all its methods are abstract and it has no state, it is close to an interface, but then an interface is usually the better choice.

**F2. Can an abstract class extend a concrete class?**

Yes. `abstract class B extends A` is fine even if `A` is concrete. `B` can also re-declare an inherited concrete method as `abstract` to force its subclasses to implement it.

**F3. Can abstract methods have parameters and return types?**

Yes, just like normal methods. They have everything except the body: `abstract double calculate(double rate, int years);`.

**F4. How does abstraction help in testing?**

Code that depends on an abstraction (an abstract class or interface) can be given a **fake/mock implementation** in tests. For example, a `PaymentService` that depends on an abstract `PaymentGateway` can be tested with a dummy gateway that doesn't contact a real bank.

---

## 7. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| "Abstract classes cannot have constructors." | They can; constructors run via `super()`. |
| "Abstract classes must have abstract methods." | Zero abstract methods is allowed. |
| `abstract void show() { }` | An abstract method cannot have a body. |
| `abstract private` / `abstract static` / `abstract final` | All compile errors. |
| `new Shape()` for an abstract `Shape` | Compile error; use a subclass or anonymous class. |
| Thinking abstraction and encapsulation are the same | Abstraction hides complexity; encapsulation hides data. |

---

## 8. Quick Revision

- **Abstraction** = show **what**, hide **how**; achieved by abstract classes (partial) and interfaces (contract).
- **Abstract class**: `abstract` keyword, cannot instantiate, can have constructors, fields, concrete + abstract methods, static members; can have 0 abstract methods; cannot be `final`.
- **Abstract method**: no body; not `private/static/final`; concrete subclass must implement all.
- Abstract class constructor runs when a subclass object is created.
- **Template Method pattern**: `final` template method + abstract steps + hooks.
- Use abstract class for related classes with shared state/code; interface for capabilities/contracts.

---

[← Previous: Polymorphism](05-Polymorphism-Overloading-vs-Overriding.md) | [Back to README](../README.md) | [Next: Interfaces →](07-Interfaces-and-Multiple-Inheritance.md)
