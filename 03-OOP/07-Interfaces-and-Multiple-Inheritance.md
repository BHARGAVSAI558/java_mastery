# Chapter 3.7 — Interfaces and Multiple Inheritance

[← Previous: Abstraction and Abstract Classes](06-Abstraction-and-Abstract-Classes.md) | [Back to README](../README.md) | [Next: Access Modifiers →](08-Access-Modifiers.md)

---

## What You Will Learn

- What an interface is and why Java needs it
- Interface members: abstract methods, constants, `default`, `static` and `private` methods
- How interfaces evolved: Java 7 → 8 → 9 → 17
- Implementing multiple interfaces (multiple inheritance of type)
- Diamond problem with default methods and how Java resolves it
- Interface inheritance (`interface extends interface`)
- Marker interfaces and functional interfaces
- **Abstract class vs interface** — the complete comparison
- Programming to an interface

---

## 1. Interfaces

### 1.1 Interview Definition

> **An interface in Java is a reference type that defines a contract — a set of methods that a class must implement. It is declared with the `interface` keyword, and a class implements it using the `implements` keyword. By default, interface methods are `public` and `abstract`, and interface variables are `public`, `static` and `final` constants. Since Java 8, interfaces can also contain `default` methods with a body and `static` methods, and since Java 9 they can contain `private` methods. An interface cannot be instantiated and cannot have instance fields or constructors. A class can implement multiple interfaces, which is how Java supports multiple inheritance of type, and interfaces are the main tool for achieving abstraction and loose coupling.**

### 1.2 Understand the Concept

An interface answers the question: **"What can this object do?"** — not "What is it?".

```java
interface Payable {
    double TAX_RATE = 0.18;          // implicitly public static final
    double calculatePay();           // implicitly public abstract
}
```

**Implicit modifiers — the compiler adds them:**

| You write | Compiler treats it as |
|---|---|
| `int MAX = 10;` | `public static final int MAX = 10;` |
| `void pay();` | `public abstract void pay();` |
| `default void log() { }` | `public default void log() { }` |
| `static void util() { }` | `public static void util() { }` |
| `interface Payable` (top-level) | `abstract interface Payable` |

**Evolution of interfaces:**

| Java version | What interfaces can contain |
|---|---|
| Java 1–7 | Only `public abstract` methods and `public static final` constants |
| **Java 8** | + **`default` methods** (with body) and **`static` methods** |
| **Java 9** | + **`private` methods** and `private static` methods (helpers for default/static methods) |
| Java 17 | + **sealed** interfaces (`sealed interface X permits A, B`) |

**Why were default methods added in Java 8?** To add new methods to existing interfaces **without breaking** every class that implements them. For example, `forEach()` was added to `Iterable` and `stream()` to `Collection` as default methods. Without them, every custom `List` implementation in the world would have stopped compiling.

### 1.3 Real-World Example

An interface is like a **job description** or a **contract**. A "Driver" job description says: must be able to `drive()`, `park()`, `followTrafficRules()`. Anyone — a person, a robot, a self-driving system — can take the job if they can do these things. The company (calling code) does not care **who** you are, only that you fulfil the contract.

A **power socket** is another example: any device with the right plug (implements the interface) can be connected, whether it is a laptop charger, a fan or a TV.

### 1.4 Java Example — Interface with All Member Types

```java
public class InterfaceDemo {

    interface PaymentMethod {
        double MAX_LIMIT = 100_000;                       // constant (public static final)

        boolean pay(double amount);                       // abstract method

        default String receipt(double amount) {           // default method (Java 8)
            return format("Paid", amount) + " via " + name();
        }

        String name();

        static boolean isValidAmount(double amount) {     // static method (Java 8)
            return amount > 0 && amount <= MAX_LIMIT;
        }

        private String format(String label, double amount) {   // private method (Java 9)
            return label + " Rs." + String.format("%.2f", amount);
        }
    }

    static class UpiPayment implements PaymentMethod {
        private final String upiId;
        UpiPayment(String upiId) { this.upiId = upiId; }

        @Override
        public boolean pay(double amount) {               // must be public
            System.out.println("UPI request sent to " + upiId);
            return true;
        }

        @Override
        public String name() { return "UPI"; }
    }

    static class CardPayment implements PaymentMethod {
        @Override
        public boolean pay(double amount) {
            System.out.println("Card charged");
            return true;
        }

        @Override
        public String name() { return "Card"; }

        @Override
        public String receipt(double amount) {            // overriding a default method
            return "CARD RECEIPT: " + PaymentMethod.super.receipt(amount);
        }
    }

    static void checkout(PaymentMethod method, double amount) {
        if (!PaymentMethod.isValidAmount(amount)) {       // static method called via interface name
            System.out.println("Invalid amount: " + amount);
            return;
        }
        if (method.pay(amount)) {
            System.out.println(method.receipt(amount));
        }
    }

    public static void main(String[] args) {
        checkout(new UpiPayment("ravi@okbank"), 499.5);
        checkout(new CardPayment(), 1200);
        checkout(new CardPayment(), 250_000);
        System.out.println("Limit: " + PaymentMethod.MAX_LIMIT);
    }
}
```

### 1.5 Code Explanation

- `MAX_LIMIT` is a constant — accessible as `PaymentMethod.MAX_LIMIT`; implementing classes cannot change it.
- `pay()` and `name()` are abstract. `UpiPayment` and `CardPayment` **must** implement them, and the implementations **must be `public`** (because interface methods are implicitly public, and you cannot reduce visibility).
- `receipt()` is a **default method** — `UpiPayment` gets it for free; `CardPayment` overrides it and calls the original using `PaymentMethod.super.receipt(amount)`.
- `isValidAmount()` is a **static method** — called using the interface name only. Static interface methods are **not inherited** by implementing classes (`UpiPayment.isValidAmount()` is a compile error).
- `format()` is a **private method** — a helper used only inside the interface.
- `checkout()` depends only on the **interface**, not on concrete classes — this is "programming to an interface".

### 1.6 Expected Output

```text
UPI request sent to ravi@okbank
Paid Rs.499.50 via UPI
Card charged
CARD RECEIPT: Paid Rs.1200.00 via Card
Invalid amount: 250000.0
Limit: 100000.0
```

### 1.7 Interview Answer — "What is an interface?"

> "An interface is a contract that defines what a class must do, without saying how. We declare it with the `interface` keyword, and a class implements it using `implements`. By default, interface methods are public and abstract, and variables are public static final constants. From Java 8, interfaces can also have **default methods** with a body — mainly so new methods can be added to existing interfaces without breaking old implementations — and **static methods**. From Java 9, they can have **private methods** to share code between default methods. An interface cannot be instantiated and cannot have constructors or instance variables. A class can implement **multiple interfaces**, which is how Java supports multiple inheritance. Interfaces give us abstraction and loose coupling: for example, my code can depend on a `PaymentMethod` interface and work with UPI, card or wallet implementations without changes."

---

## 2. Multiple Inheritance Using Interfaces

### 2.1 Interview Definition

> **Java does not allow a class to extend more than one class, but it allows a class to implement any number of interfaces, and an interface can extend multiple interfaces. This provides multiple inheritance of type (and, since Java 8, of behaviour through default methods). If two interfaces provide default methods with the same signature, the implementing class must override the method to resolve the conflict, and it can call a specific interface's version using `InterfaceName.super.methodName()`.**

### 2.2 Java Example

```java
public class MultipleInheritanceDemo {

    interface Flyable {
        void fly();
        default String describe() { return "I can fly"; }
    }

    interface Swimmable {
        void swim();
        default String describe() { return "I can swim"; }
    }

    // Interface extending multiple interfaces
    interface Amphibious extends Flyable, Swimmable {
        @Override
        default String describe() {                       // must resolve the conflict here too
            return Flyable.super.describe() + " and " + Swimmable.super.describe();
        }
    }

    static class Animal {
        String name;
        Animal(String name) { this.name = name; }
        public String describe() { return "I am an animal"; }   // must be public (see note below)
    }

    // Class: extends one class AND implements many interfaces
    static class Duck extends Animal implements Flyable, Swimmable {
        Duck() { super("Duck"); }
        @Override public void fly()  { System.out.println(name + " flies low"); }
        @Override public void swim() { System.out.println(name + " paddles"); }
        // describe() comes from Animal: class method WINS over interface defaults (rule 1)
    }

    static class SeaPlane implements Amphibious {
        @Override public void fly()  { System.out.println("SeaPlane flies"); }
        @Override public void swim() { System.out.println("SeaPlane floats"); }
    }

    public static void main(String[] args) {
        Duck duck = new Duck();
        duck.fly();
        duck.swim();
        System.out.println("Duck: " + duck.describe());

        Flyable f = duck;            // a Duck can be treated as Flyable
        Swimmable s = duck;          // ... or as Swimmable
        Animal a = duck;             // ... or as Animal
        System.out.println(f instanceof Swimmable);

        SeaPlane plane = new SeaPlane();
        plane.fly();
        System.out.println("SeaPlane: " + plane.describe());
    }
}
```

### 2.3 Expected Output

```text
Duck flies low
Duck paddles
Duck: I am an animal
true
SeaPlane flies
SeaPlane: I can fly and I can swim
```

### 2.4 Rules for Resolving Default Method Conflicts

When a class inherits the same method from multiple places, Java uses three rules:

1. **Class wins.** A method from a superclass (or the class itself) always has priority over interface default methods. (That is why `Duck.describe()` uses `Animal.describe()`.)
   Note: the superclass method must be `public`. If `Animal.describe()` had default (package-private) access, the compiler would report *"describe() in Animal cannot implement describe() in Flyable; attempting to assign weaker access privileges; was public"* — because the inherited method now acts as the implementation of a public interface method.
2. **More specific interface wins.** If interface `B extends A` and both provide the default, `B`'s version is used.
3. **Otherwise, the class must override** the method explicitly and can choose with `X.super.method()`. If it does not, it is a compile error.

This is how Java solves the diamond problem for interfaces: the ambiguity is never silent — the programmer must decide.

---

## 3. Marker Interfaces and Functional Interfaces

### 3.1 Marker Interface

> **A marker interface is an interface with no methods or constants. It is used to "mark" a class so that the JVM or a framework treats objects of that class in a special way.**

| Marker interface | Meaning |
|---|---|
| `java.io.Serializable` | Objects of this class can be serialized |
| `java.lang.Cloneable` | `Object.clone()` is allowed (else `CloneNotSupportedException`) |
| `java.util.RandomAccess` | List supports fast index access (used by algorithms to choose a strategy) |

Today, **annotations** are often used for this purpose instead (e.g., `@Entity`), but marker interfaces have an advantage: they define a **type**, so the compiler can check it (`void save(Serializable obj)`).

### 3.2 Functional Interface (Java 8)

> **A functional interface is an interface with exactly one abstract method (it can still have any number of default and static methods). It can be implemented using a lambda expression or a method reference. The optional `@FunctionalInterface` annotation makes the compiler verify that the interface has exactly one abstract method.**

```java
public class FunctionalInterfaceDemo {

    @FunctionalInterface
    interface Calculator {
        int compute(int a, int b);                         // exactly one abstract method
        default Calculator andThenDouble() {
            return (a, b) -> compute(a, b) * 2;
        }
    }

    public static void main(String[] args) {
        Calculator add = (a, b) -> a + b;                  // lambda implements the interface
        Calculator multiply = (a, b) -> a * b;
        Calculator max = Math::max;                        // method reference

        System.out.println("add: " + add.compute(4, 5));
        System.out.println("multiply: " + multiply.compute(4, 5));
        System.out.println("max: " + max.compute(4, 5));
        System.out.println("add then double: " + add.andThenDouble().compute(4, 5));

        Runnable task = () -> System.out.println("Runnable is a functional interface too");
        task.run();
    }
}
```

**Expected Output:**

```text
add: 9
multiply: 20
max: 5
add then double: 18
Runnable is a functional interface too
```

Built-in functional interfaces (`Runnable`, `Comparator`, `Predicate`, `Function`, `Supplier`, `Consumer`) are covered in [Part 8](../08-Java-8-and-Modern-Java/01-Lambda-Expressions-and-Functional-Interfaces.md).

---

## 4. Abstract Class vs Interface

### 4.1 Comparison Table

| Point | Abstract class | Interface |
|---|---|---|
| Keyword | `abstract class` | `interface` |
| Inherit using | `extends` | `implements` (class), `extends` (interface) |
| Multiple inheritance | A class can extend **only one** | A class can implement **many** |
| Methods | Abstract and concrete (any access) | Abstract; `default`, `static` (Java 8); `private` (Java 9) |
| Method access | Any (`public`, `protected`, default, `private`) | `public` (except `private` helpers) |
| Variables | Any kind: instance, static, final, non-final | Only `public static final` constants |
| Constructors | **Yes** | **No** |
| State (instance fields) | Yes | No |
| Instantiation | Not allowed | Not allowed |
| Abstraction | Partial (0–100%) | Full contract (plus default behaviour) |
| Speed | Slightly faster method calls in old JVMs (not significant today) | — |
| Relationship | **IS-A** (closely related classes) | **CAN-DO** / capability (can be unrelated classes) |
| When added to an existing hierarchy | Forces a single parent | Can be added to any class |
| Example | `abstract class Vehicle` | `interface Comparable`, `Runnable`, `Serializable` |

### 4.2 When to Use Which

**Choose an abstract class when:**
- Classes are **closely related** and share **state** (fields) and **code**.
- You need **constructors**, `protected` or `private` members.
- You want to provide a **template** (Template Method pattern).

**Choose an interface when:**
- You want to define a **capability** that **unrelated** classes can have (`Comparable` for `String`, `Integer`, `Employee`).
- You need **multiple inheritance** of type.
- You want maximum **flexibility and loose coupling** (dependency injection, testing with mocks).
- You are designing an **API** — interfaces are easier to evolve with default methods.

**Both together (common in the JDK):** `List` (interface) → `AbstractList` (abstract class with shared code) → `ArrayList` (concrete). This gives the flexibility of an interface plus the code reuse of an abstract class.

### 4.3 Interview Answer — "Abstract class vs interface"

> "Both are used for abstraction and neither can be instantiated, but they differ in several ways. An abstract class can have instance variables, constructors, and both abstract and concrete methods with any access modifier, but a class can extend only one abstract class. An interface defines a contract: its variables are always public static final constants, it has no constructors, and its methods are public — abstract by default, with default and static methods from Java 8 and private methods from Java 9. A class can implement many interfaces, so interfaces support multiple inheritance. I use an abstract class when related classes share common state and code — like a `Vehicle` base class — and an interface when I want to define a capability that even unrelated classes can have, like `Comparable` or `Payable`. Often both are combined, like `List`, `AbstractList` and `ArrayList` in the Collections framework."

---

## 5. Programming to an Interface

```java
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

public class ProgramToInterface {

    interface NotificationSender {
        void send(String to, String msg);
    }

    static class EmailSender implements NotificationSender {
        public void send(String to, String msg) { System.out.println("Email to " + to + ": " + msg); }
    }

    static class SmsSender implements NotificationSender {
        public void send(String to, String msg) { System.out.println("SMS to " + to + ": " + msg); }
    }

    static class OrderService {
        private final NotificationSender sender;            // depends on the interface

        OrderService(NotificationSender sender) {            // implementation injected from outside
            this.sender = sender;
        }

        void placeOrder(String customer) {
            System.out.println("Order placed for " + customer);
            sender.send(customer, "Your order is confirmed");
        }
    }

    public static void main(String[] args) {
        new OrderService(new EmailSender()).placeOrder("asha@mail.com");
        new OrderService(new SmsSender()).placeOrder("+91-98xxxxxx10");

        List<String> names = new ArrayList<>();              // declare with the interface type
        names.add("Ravi");
        names = new LinkedList<>(names);                     // switch implementation easily
        System.out.println(names + " using " + names.getClass().getSimpleName());
    }
}
```

**Expected Output:**

```text
Order placed for asha@mail.com
Email to asha@mail.com: Your order is confirmed
Order placed for +91-98xxxxxx10
SMS to +91-98xxxxxx10: Your order is confirmed
[Ravi] using LinkedList
```

`OrderService` never mentions `EmailSender` or `SmsSender`. We can add `WhatsAppSender` without touching `OrderService`. This is **loose coupling** and the basis of **Dependency Injection** in frameworks like Spring. Declaring `List<String> names` instead of `ArrayList<String> names` follows the same idea.

---

## 6. Interview Questions with Complete Answers

### Basic Level

**Q1. Can we create an object of an interface?**

No. But we can create a reference of an interface type pointing to an implementing object (`Runnable r = new MyTask();`), or create an anonymous class / lambda that implements it (`Runnable r = () -> {...};`).

**Q2. Why are interface variables `public static final`?**

- `public` — the interface is a public contract, so its constants should be visible.
- `static` — an interface has no instances, so a variable must belong to the interface itself.
- `final` — many classes implement the interface; if one could change the value, it would affect all. Constants keep the contract stable.

**Q3. Can an interface have a constructor?**

No. Constructors initialise instance fields, and interfaces cannot have instance fields. Also, interfaces cannot be instantiated.

### Intermediate Level

**Q4. What are default methods and why were they introduced?**

Default methods (Java 8) are interface methods with a body, marked with `default`. Implementing classes inherit them and can override them. They were introduced for **backward compatibility** — to add new methods (like `forEach`, `stream`, `removeIf`, `sort`) to existing interfaces (`Iterable`, `Collection`, `List`) without breaking millions of existing implementations.

**Q5. What is the difference between default and static methods in an interface?**

A default method is an **instance** method: it is inherited by implementing classes, can be overridden, and is called on an object (`obj.method()`). A static method belongs to the **interface only**: it is not inherited, cannot be overridden, and is called with the interface name (`MyInterface.method()`). Static methods are used for utility/helper/factory methods (`Comparator.comparing()`, `List.of()`).

**Q6. Can an interface extend a class?**

No. An interface can extend only other interfaces (and it can extend **multiple** interfaces). However, every interface implicitly has the public methods of `Object` (like `equals`, `toString`) available, because every implementing object is an `Object`.

**Q7. Can an interface method be `protected`, `final` or `synchronized`?**

No. Interface methods are `public` (or `private` for helpers). They cannot be `protected` or `final` (final would prevent implementation). `synchronized` is not allowed on default methods because locking is an implementation concern of the class.

### Advanced Level

**Q8. If a class inherits the same default method from two interfaces, what happens?**

The class gets a compile error ("class X inherits unrelated defaults for m() from types A and B") unless it overrides the method. Inside the override, it can call `A.super.m()` or `B.super.m()` (or both). If the method also exists in a superclass, the superclass version wins automatically (class-wins rule).

**Q9. Can a default method override a method from `Object` like `equals` or `toString`?**

No. A default method cannot override `equals()`, `hashCode()` or `toString()` from `Object` — it is a compile error. Because of the "class wins" rule, the `Object` version would always be used anyway, so such a default would be useless and confusing.

**Q10. What is a functional interface? Is `Comparator` a functional interface even though it has many methods?**

A functional interface has exactly **one abstract method**. `Comparator` has many default and static methods, and it declares `equals(Object)`, but `equals` is a public `Object` method and **does not count**. Its single abstract method is `compare(T, T)`, so it is a functional interface.

**Q11. Can we have a `main` method in an interface?**

Yes, since Java 8 (static methods are allowed). The interface can be run with `java InterfaceName`.

---

## 7. Follow-Up Questions

**F1. If interfaces have default methods now, what is the difference from abstract classes?**

Interfaces still cannot have **state** (instance fields) or **constructors**, and their methods are public. Abstract classes can hold state, have constructors, and use any access level. Also, a class can implement many interfaces but extend only one class.

**F2. Is it possible to implement an interface partially?**

Only in an **abstract class**. A concrete class must implement all abstract methods.

**F3. What is the "constant interface anti-pattern"?**

Using an interface only to hold constants and implementing it to access them without a prefix. It pollutes the implementing class's API with constants. Better: use a `final` class with a private constructor, or an `enum`, and `import static`.

**F4. Why must an implementing method be `public`?**

Interface methods are implicitly `public`. Overriding rules do not allow reducing visibility, so the implementation must also be `public`. Writing `void pay()` without `public` gives "attempting to assign weaker access privileges".

---

## 8. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Forgetting `public` on implementing methods | Compile error (weaker access). |
| Calling a static interface method through an object or implementing class | Use `InterfaceName.method()`. |
| Expecting interface variables to be changeable | They are constants. |
| Thinking interfaces can't have method bodies | `default`, `static`, `private` methods have bodies. |
| Not resolving duplicate default methods | Compile error; override and choose. |
| Saying "Java supports multiple inheritance" without context | Only through interfaces, not classes. |

---

## 9. Quick Revision

- Interface = contract; `implements`; cannot instantiate; no constructors; no instance fields.
- Methods: `public abstract` by default; `default` + `static` (Java 8); `private` (Java 9); sealed (Java 17).
- Variables: `public static final` constants.
- Static interface methods are **not inherited**; call via interface name.
- Multiple inheritance: class implements many interfaces; interface extends many interfaces.
- Conflict rules: **class wins → more specific interface wins → must override** (`X.super.m()`).
- Marker interfaces: `Serializable`, `Cloneable`, `RandomAccess`.
- Functional interface: one abstract method; `@FunctionalInterface`; lambdas.
- Abstract class = IS-A + shared state/code; interface = CAN-DO capability + multiple inheritance.
- Program to an interface → loose coupling, easy to swap implementations.

---

[← Previous: Abstraction and Abstract Classes](06-Abstraction-and-Abstract-Classes.md) | [Back to README](../README.md) | [Next: Access Modifiers →](08-Access-Modifiers.md)
