# Chapter 3.13 — SOLID Principles and Important OOP Design Concepts

[← Previous: Upcasting, Downcasting, Dynamic Dispatch](12-Upcasting-Downcasting-Dynamic-Dispatch.md) | [Back to README](../README.md) | [Next: OOP Interview Question Bank →](14-OOP-Interview-Question-Bank.md)

---

## What You Will Learn

- What SOLID is and why it matters
- Each principle — S, O, L, I, D — with a bad example, a good example and an interview answer
- Coupling and cohesion
- DRY, KISS, YAGNI, Law of Demeter
- Introduction to common design patterns asked of freshers: Singleton, Factory, Builder, Strategy, Observer

---

## 1. SOLID — Overview

### 1.1 Interview Definition

> **SOLID is a set of five object-oriented design principles, popularised by Robert C. Martin, that help create software that is easy to understand, maintain, test and extend. The five principles are: Single Responsibility Principle — a class should have only one reason to change; Open/Closed Principle — classes should be open for extension but closed for modification; Liskov Substitution Principle — objects of a subclass should be replaceable for objects of the parent class without breaking the program; Interface Segregation Principle — clients should not be forced to depend on methods they do not use; and Dependency Inversion Principle — high-level modules should depend on abstractions, not on concrete implementations.**

| Letter | Principle | One-line meaning |
|---|---|---|
| **S** | Single Responsibility | One class → one job → one reason to change |
| **O** | Open/Closed | Add new behaviour by adding code, not editing old code |
| **L** | Liskov Substitution | A child must work wherever its parent works |
| **I** | Interface Segregation | Many small, focused interfaces instead of one fat interface |
| **D** | Dependency Inversion | Depend on interfaces, not concrete classes |

**Why SOLID?** Without it, code becomes **rigid** (hard to change), **fragile** (one change breaks many places), and **hard to test**. SOLID keeps classes small, focused and loosely coupled.

---

## 2. S — Single Responsibility Principle (SRP)

### 2.1 Definition

> **A class should have only one responsibility, meaning only one reason to change. Each class should focus on a single job, and different concerns — such as business logic, data storage, formatting and notifications — should be placed in separate classes.**

### 2.2 Bad vs Good

<!-- skip-check -->
```java
// BAD: one class does four jobs → four reasons to change
class Invoice {
    double calculateTotal() { ... }          // business rules
    void saveToDatabase() { ... }            // persistence
    String generatePdf() { ... }             // presentation
    void emailToCustomer() { ... }           // notification
}
```

If the database changes, the PDF layout changes, or the email provider changes — the same `Invoice` class must be edited every time, and every change risks breaking the other features.

```java
public class SrpDemo {
    record Item(String name, double price, int qty) { }

    static class Invoice {                                     // only business data and rules
        private final java.util.List<Item> items;
        Invoice(java.util.List<Item> items) { this.items = items; }
        double total() {
            return items.stream().mapToDouble(i -> i.price() * i.qty()).sum();
        }
    }

    static class InvoiceRepository {                           // only persistence
        void save(Invoice invoice) { System.out.println("Saved invoice with total " + invoice.total()); }
    }

    static class InvoicePrinter {                              // only formatting
        String format(Invoice invoice) { return String.format("INVOICE TOTAL: Rs.%.2f", invoice.total()); }
    }

    static class EmailService {                                // only notification
        void send(String to, String body) { System.out.println("Email to " + to + " -> " + body); }
    }

    public static void main(String[] args) {
        Invoice invoice = new Invoice(java.util.List.of(new Item("Pen", 10, 5), new Item("Book", 120, 2)));
        new InvoiceRepository().save(invoice);
        String text = new InvoicePrinter().format(invoice);
        new EmailService().send("asha@mail.com", text);
    }
}
```

**Expected Output:**

```text
Saved invoice with total 290.0
Email to asha@mail.com -> INVOICE TOTAL: Rs.290.00
```

Each class now has exactly **one reason to change**. They are also easier to test independently.

### 2.3 Interview Answer

> "The Single Responsibility Principle says a class should have only one reason to change — it should do one job. For example, an `Invoice` class should only hold invoice data and calculate totals. Saving it to a database, generating a PDF and emailing it are separate responsibilities, so they go into `InvoiceRepository`, `InvoicePrinter` and `EmailService`. This makes classes smaller, easier to understand and test, and a change in one area, like switching email providers, doesn't affect the others."

---

## 3. O — Open/Closed Principle (OCP)

### 3.1 Definition

> **Software entities (classes, modules, functions) should be open for extension but closed for modification. We should be able to add new behaviour by writing new code — usually new classes that implement an interface or extend an abstract class — without changing existing, tested code.**

### 3.2 Bad vs Good

<!-- skip-check -->
```java
// BAD: every new payment type requires editing this method
class PaymentProcessor {
    void pay(String type, double amount) {
        if (type.equals("CARD")) { ... }
        else if (type.equals("UPI")) { ... }
        else if (type.equals("WALLET")) { ... }   // edit again for every new type
    }
}
```

```java
import java.util.List;

public class OcpDemo {
    interface PaymentMethod {
        void pay(double amount);
    }

    static class CardPayment implements PaymentMethod {
        public void pay(double amount) { System.out.println("Paid " + amount + " by card"); }
    }

    static class UpiPayment implements PaymentMethod {
        public void pay(double amount) { System.out.println("Paid " + amount + " by UPI"); }
    }

    // New requirement: add wallet → just ADD a class; PaymentProcessor is untouched
    static class WalletPayment implements PaymentMethod {
        public void pay(double amount) { System.out.println("Paid " + amount + " from wallet"); }
    }

    static class PaymentProcessor {                     // closed for modification
        void process(PaymentMethod method, double amount) {
            method.pay(amount);
        }
    }

    public static void main(String[] args) {
        PaymentProcessor processor = new PaymentProcessor();
        for (PaymentMethod m : List.of(new CardPayment(), new UpiPayment(), new WalletPayment())) {
            processor.process(m, 500);
        }
    }
}
```

**Expected Output:**

```text
Paid 500.0 by card
Paid 500.0 by UPI
Paid 500.0 from wallet
```

OCP is achieved with **abstraction + polymorphism**. The `if-else` chain is replaced by dynamic method dispatch.

### 3.3 Interview Answer

> "The Open/Closed Principle says classes should be open for extension but closed for modification. Instead of editing an existing class with a growing if-else chain every time a new type comes, we define an interface — for example `PaymentMethod` — and each type implements it. To add a wallet payment, I only add a new `WalletPayment` class; the `PaymentProcessor` doesn't change. This protects tested code from new bugs, and it relies on abstraction and runtime polymorphism."

---

## 4. L — Liskov Substitution Principle (LSP)

### 4.1 Definition

> **Objects of a superclass should be replaceable with objects of its subclasses without changing the correctness of the program. A subclass must honour the behaviour and expectations (the contract) of its parent: it should not strengthen preconditions, weaken postconditions, throw unexpected exceptions, or disable inherited behaviour.**

### 4.2 Classic Violation — Rectangle and Square

```java
public class LspViolationDemo {
    static class Rectangle {
        protected int width, height;
        void setWidth(int w)  { width = w; }
        void setHeight(int h) { height = h; }
        int area() { return width * height; }
    }

    static class Square extends Rectangle {           // "a square IS-A rectangle" in maths...
        @Override void setWidth(int w)  { width = w; height = w; }
        @Override void setHeight(int h) { width = h; height = h; }
    }

    static void resize(Rectangle r) {                 // code written for Rectangle
        r.setWidth(5);
        r.setHeight(4);
        System.out.println("Expected area 20, got " + r.area());
    }

    public static void main(String[] args) {
        resize(new Rectangle());
        resize(new Square());                         // substitution breaks expectations
    }
}
```

**Expected Output:**

```text
Expected area 20, got 20
Expected area 20, got 16
```

`Square` cannot be used everywhere a `Rectangle` is expected: code assuming width and height are independent breaks. Mathematically a square is a rectangle, but **behaviourally** it is not a substitute. **Fix:** don't make `Square` extend a mutable `Rectangle`; make both implement a `Shape` interface with `area()`, or make them immutable.

Other common LSP violations:
- A subclass that throws `UnsupportedOperationException` for an inherited method (e.g., a `ReadOnlyList` extending a mutable list — `Arrays.asList().add()` is a known example of this trade-off in the JDK).
- `Ostrich extends Bird` where `Bird` has `fly()`.

### 4.3 Interview Answer

> "The Liskov Substitution Principle says that if `B` is a subclass of `A`, we should be able to use a `B` object anywhere an `A` is expected, without the program behaving incorrectly. The classic example is `Square extends Rectangle`: if code sets width 5 and height 4 and expects area 20, a square gives 16, because setting one side changes both. So even though a square 'is a' rectangle mathematically, it is not a valid behavioural subtype. LSP tells us inheritance should be based on behaviour, not just on real-world classification. A violation is often visible when subclasses throw `UnsupportedOperationException` or need `instanceof` checks."

---

## 5. I — Interface Segregation Principle (ISP)

### 5.1 Definition

> **Clients should not be forced to depend on interfaces they do not use. Instead of one large, general-purpose interface, create several smaller, role-specific interfaces, so that each class implements only the methods that are relevant to it.**

### 5.2 Bad vs Good

<!-- skip-check -->
```java
// BAD: fat interface
interface Worker {
    void work();
    void eat();
    void attendMeeting();
}

class Robot implements Worker {
    public void work() { ... }
    public void eat() { throw new UnsupportedOperationException(); }   // forced, meaningless
    public void attendMeeting() { throw new UnsupportedOperationException(); }
}
```

```java
public class IspDemo {
    interface Workable { void work(); }
    interface Eatable { void eat(); }
    interface MeetingAttendee { void attendMeeting(); }

    static class Employee implements Workable, Eatable, MeetingAttendee {
        public void work() { System.out.println("Employee works"); }
        public void eat() { System.out.println("Employee eats lunch"); }
        public void attendMeeting() { System.out.println("Employee attends stand-up"); }
    }

    static class Robot implements Workable {           // only what it actually does
        public void work() { System.out.println("Robot assembles parts"); }
    }

    public static void main(String[] args) {
        Workable[] workers = { new Employee(), new Robot() };
        for (Workable w : workers) w.work();
        new Employee().attendMeeting();
    }
}
```

**Expected Output:**

```text
Employee works
Robot assembles parts
Employee attends stand-up
```

### 5.3 Interview Answer

> "Interface Segregation says no class should be forced to implement methods it doesn't need. If we have one big `Worker` interface with `work`, `eat` and `attendMeeting`, a `Robot` must implement `eat` with an empty body or an exception, which is wrong. So we split it into smaller interfaces — `Workable`, `Eatable`, `MeetingAttendee` — and each class implements only what applies. Java itself follows this: `Comparable`, `Runnable`, `AutoCloseable` are tiny, focused interfaces."

---

## 6. D — Dependency Inversion Principle (DIP)

### 6.1 Definition

> **High-level modules (business logic) should not depend on low-level modules (implementation details such as databases, email or file systems); both should depend on abstractions. Abstractions should not depend on details; details should depend on abstractions. In practice, a class should depend on an interface and receive the concrete implementation from outside, usually through its constructor (dependency injection).**

### 6.2 Bad vs Good

<!-- skip-check -->
```java
// BAD: high-level class creates and depends on a concrete low-level class
class OrderService {
    private final MySqlDatabase db = new MySqlDatabase();   // tightly coupled
    void placeOrder(Order o) { db.insert(o); }
}
```

Problems: cannot switch to PostgreSQL or MongoDB without editing `OrderService`; cannot unit-test without a real MySQL database.

```java
import java.util.ArrayList;
import java.util.List;

public class DipDemo {
    interface OrderRepository {                          // abstraction
        void save(String order);
        int count();
    }

    static class MySqlOrderRepository implements OrderRepository {    // detail
        private int saved;
        public void save(String order) { saved++; System.out.println("INSERT INTO orders: " + order); }
        public int count() { return saved; }
    }

    static class InMemoryOrderRepository implements OrderRepository { // detail (great for tests)
        private final List<String> orders = new ArrayList<>();
        public void save(String order) { orders.add(order); }
        public int count() { return orders.size(); }
    }

    static class OrderService {                          // high-level module
        private final OrderRepository repository;        // depends on the abstraction

        OrderService(OrderRepository repository) {       // constructor injection
            this.repository = repository;
        }

        void placeOrder(String order) {
            if (order.isBlank()) throw new IllegalArgumentException("Empty order");
            repository.save(order);
        }
    }

    public static void main(String[] args) {
        OrderService production = new OrderService(new MySqlOrderRepository());
        production.placeOrder("2 x Java Book");

        InMemoryOrderRepository fakeRepo = new InMemoryOrderRepository();
        OrderService underTest = new OrderService(fakeRepo);
        underTest.placeOrder("1 x Pen");
        underTest.placeOrder("3 x Notebook");
        System.out.println("Test check - orders saved: " + fakeRepo.count());
    }
}
```

**Expected Output:**

```text
INSERT INTO orders: 2 x Java Book
Test check - orders saved: 2
```

`OrderService` doesn't know or care which database is used. The same class works in production and in tests. This is exactly what **Spring's dependency injection** does for you (`@Autowired` / constructor injection).

**DIP vs DI vs IoC:**
- **DIP** — the *principle*: depend on abstractions.
- **Dependency Injection (DI)** — the *technique*: supply dependencies from outside (constructor/setter).
- **Inversion of Control (IoC)** — the *broader idea*: a framework (like Spring) creates and wires objects instead of your code.

### 6.3 Interview Answer

> "The Dependency Inversion Principle says high-level business classes should not depend directly on low-level classes like a specific database or email provider; both should depend on abstractions, usually interfaces. For example, `OrderService` should depend on an `OrderRepository` interface, and the actual MySQL or in-memory implementation is passed in through the constructor. This is dependency injection, and frameworks like Spring do it automatically. The benefits are loose coupling, the ability to switch implementations without changing business code, and easy unit testing with fake implementations."

---

## 7. Other Important Design Concepts

### 7.1 Coupling and Cohesion

| Concept | Meaning | Goal |
|---|---|---|
| **Coupling** | How much one class depends on the internals of another | **Low (loose) coupling** |
| **Cohesion** | How closely the responsibilities inside one class are related | **High cohesion** |

- **Low coupling:** classes interact through small interfaces; a change in one rarely forces changes in others (achieved by encapsulation, interfaces, DIP).
- **High cohesion:** each class does one well-defined job; its methods and fields belong together (achieved by SRP).
- **Goal: "High cohesion, low coupling."**

### 7.2 DRY, KISS, YAGNI, Law of Demeter

| Principle | Meaning | Example |
|---|---|---|
| **DRY** — Don't Repeat Yourself | Every piece of knowledge should exist in one place | Put validation in one method, not copied in five places |
| **KISS** — Keep It Simple, Stupid | Prefer the simplest solution that works | Don't build a plugin framework for two options |
| **YAGNI** — You Aren't Gonna Need It | Don't build features until they are actually needed | Don't add multi-currency support "just in case" |
| **Law of Demeter** ("talk only to friends") | A method should call methods only on itself, its fields, its parameters, and objects it creates — avoid long chains | Avoid `order.getCustomer().getAddress().getCity()`; prefer `order.getShippingCity()` |

---

## 8. Common Design Patterns (Fresher Level)

| Pattern | Category | Problem it solves | JDK / framework example |
|---|---|---|---|
| **Singleton** | Creational | Exactly one instance of a class | `Runtime.getRuntime()`, Spring beans (default scope) |
| **Factory** | Creational | Create objects without exposing creation logic / concrete classes | `Calendar.getInstance()`, `List.of()` |
| **Builder** | Creational | Build objects with many optional parameters readably | `StringBuilder`, `HttpRequest.newBuilder()` |
| **Strategy** | Behavioural | Swap algorithms at runtime | `Comparator` passed to `sort()` |
| **Observer** | Behavioural | Notify many listeners when something changes | Event listeners, `PropertyChangeListener` |
| **Decorator** | Structural | Add behaviour by wrapping objects | `BufferedReader(new FileReader(...))` |
| **Adapter** | Structural | Make incompatible interfaces work together | `Arrays.asList()`, `InputStreamReader` |
| **Template Method** | Behavioural | Fixed algorithm skeleton, varying steps | `AbstractList`, `HttpServlet` ([Chapter 3.6](06-Abstraction-and-Abstract-Classes.md)) |

### 8.1 Java Example — Singleton (thread-safe), Factory, Builder, Observer

```java
import java.util.ArrayList;
import java.util.List;

public class DesignPatternsDemo {

    // ---------- Singleton: thread-safe, lazy (Bill Pugh / holder idiom) ----------
    static class Logger {
        private Logger() { }
        private static class Holder {                 // loaded only when getInstance() is first called
            private static final Logger INSTANCE = new Logger();
        }
        static Logger getInstance() { return Holder.INSTANCE; }
        void log(String msg) { System.out.println("[LOG] " + msg); }
    }

    // ---------- Factory ----------
    interface Notification { void send(String msg); }
    static class EmailNotification implements Notification {
        public void send(String msg) { System.out.println("Email: " + msg); }
    }
    static class SmsNotification implements Notification {
        public void send(String msg) { System.out.println("SMS: " + msg); }
    }
    static class NotificationFactory {
        static Notification create(String channel) {
            return switch (channel.toLowerCase()) {
                case "email" -> new EmailNotification();
                case "sms" -> new SmsNotification();
                default -> throw new IllegalArgumentException("Unknown channel: " + channel);
            };
        }
    }

    // ---------- Builder ----------
    static class Pizza {
        private final String size;          // required
        private final boolean cheese;       // optional
        private final boolean olives;       // optional
        private final String crust;         // optional

        private Pizza(Builder b) {
            this.size = b.size; this.cheese = b.cheese; this.olives = b.olives; this.crust = b.crust;
        }
        @Override public String toString() {
            return size + " pizza, crust=" + crust + ", cheese=" + cheese + ", olives=" + olives;
        }

        static class Builder {
            private final String size;
            private boolean cheese;
            private boolean olives;
            private String crust = "regular";
            Builder(String size) { this.size = size; }
            Builder cheese()            { this.cheese = true; return this; }
            Builder olives()            { this.olives = true; return this; }
            Builder crust(String crust) { this.crust = crust; return this; }
            Pizza build()               { return new Pizza(this); }
        }
    }

    // ---------- Observer ----------
    interface StockListener { void onPriceChange(String stock, double price); }
    static class StockTicker {
        private final List<StockListener> listeners = new ArrayList<>();
        void subscribe(StockListener l) { listeners.add(l); }
        void updatePrice(String stock, double price) {
            for (StockListener l : listeners) l.onPriceChange(stock, price);
        }
    }

    public static void main(String[] args) {
        Logger.getInstance().log("App started");
        System.out.println("Same logger? " + (Logger.getInstance() == Logger.getInstance()));

        NotificationFactory.create("sms").send("OTP 1234");
        NotificationFactory.create("email").send("Welcome!");

        Pizza pizza = new Pizza.Builder("Large").cheese().crust("thin").build();
        System.out.println(pizza);

        StockTicker ticker = new StockTicker();
        ticker.subscribe((s, p) -> System.out.println("Mobile app: " + s + " = " + p));
        ticker.subscribe((s, p) -> { if (p > 1500) System.out.println("Alert service: " + s + " crossed 1500"); });
        ticker.updatePrice("INFY", 1520.5);
    }
}
```

**Expected Output:**

```text
[LOG] App started
Same logger? true
SMS: OTP 1234
Email: Welcome!
Large pizza, crust=thin, cheese=true, olives=false
Mobile app: INFY = 1520.5
Alert service: INFY crossed 1500
```

**Short explanations:**
- **Singleton (holder idiom):** the instance is created lazily when `Holder` is first loaded; class loading is thread-safe in the JVM, so no `synchronized` is needed. (Other approaches: eager static field, double-checked locking with `volatile`, or an `enum` singleton — the most robust against reflection and serialization.)
- **Factory:** callers ask for a "sms" notification without knowing the concrete class — supports OCP and DIP.
- **Builder:** readable creation of objects with many optional fields; avoids "telescoping constructors" (`Pizza(size)`, `Pizza(size, cheese)`, `Pizza(size, cheese, olives)` …) and allows immutable objects.
- **Observer:** the ticker notifies all subscribers; new subscribers can be added without changing the ticker.

---

## 9. Interview Questions with Complete Answers

**Q1. What does SOLID stand for? Explain briefly.**

S — Single Responsibility: one class, one reason to change. O — Open/Closed: extend behaviour by adding code, not modifying existing code. L — Liskov Substitution: subclasses must be usable wherever the parent is expected. I — Interface Segregation: prefer small, specific interfaces. D — Dependency Inversion: depend on abstractions and inject implementations. Together they produce loosely coupled, maintainable and testable code.

**Q2. Which OOP features support the Open/Closed Principle?**

Abstraction (interfaces/abstract classes) and runtime polymorphism (overriding). Code depends on an abstraction; new behaviour comes as new implementations. Design patterns like Strategy, Factory and Template Method are common ways to apply OCP.

**Q3. How do you identify an LSP violation?**

Signs: a subclass overrides a method to throw `UnsupportedOperationException` or to do nothing; callers need `instanceof` checks to treat a subclass specially; a subclass requires stricter input or gives weaker guarantees than the parent; tests written for the parent fail for the child.

**Q4. What is the difference between Dependency Inversion and Dependency Injection?**

Dependency Inversion is a **design principle** — high-level code should depend on abstractions. Dependency Injection is a **technique** to achieve it — providing the concrete implementation from outside, via the constructor, a setter or a framework, instead of creating it inside with `new`.

**Q5. How do you make a Singleton thread-safe?**

Options: (1) **eager initialisation** — `private static final Singleton INSTANCE = new Singleton();` (thread-safe by class loading); (2) **holder class idiom** — lazy and thread-safe; (3) **double-checked locking** — with a `volatile` instance field and a `synchronized` block; (4) **enum singleton** — `enum Singleton { INSTANCE; }`, which is thread-safe and also safe against reflection and serialization attacks (recommended by *Effective Java*).

**Q6. What is the difference between Factory and Builder?**

A **Factory** decides **which** object/class to create (often returns an interface type) in one step. A **Builder** focuses on **how** to construct one complex object step by step, especially with many optional parameters. Factory hides the concrete type; Builder improves readability and supports immutability.

**Q7. Explain high cohesion and low coupling with an example.**

High cohesion: a `PasswordHasher` class contains only hashing-related methods. Low coupling: `UserService` depends on a `PasswordHasher` interface, not on a BCrypt-specific class, so the hashing algorithm can be changed without touching `UserService`.

**Q8. Which SOLID principle does `Stack extends Vector` violate?**

Mainly **Liskov Substitution** (a stack should not support inserting at an arbitrary index, so it is not a proper behavioural subtype of a vector, nor should a stack be used as a vector) and it reflects misuse of inheritance instead of composition. It also weakens encapsulation of stack behaviour. `Deque`/`ArrayDeque` is the recommended replacement.

---

## 10. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Thinking SRP means "one method per class" | One **responsibility / reason to change**, not one method. |
| Applying OCP by making everything abstract upfront | Abstract where change is likely (YAGNI). |
| Using real-world "is-a" to justify inheritance | LSP requires behavioural substitutability. |
| Huge "service" interfaces implemented with empty methods | Split interfaces (ISP). |
| `new` concrete dependencies inside business classes | Inject interfaces (DIP). |
| Non-thread-safe lazy singletons | Use holder idiom, eager init or enum. |

---

## 11. Quick Revision

- **S**RP: one reason to change · **O**CP: extend, don't modify · **L**SP: subclass substitutable · **I**SP: small interfaces · **D**IP: depend on abstractions + DI.
- **High cohesion, low coupling.** DRY, KISS, YAGNI, Law of Demeter.
- Patterns: Singleton (holder/enum), Factory (which object), Builder (how to build), Strategy (swap algorithm), Observer (notify listeners), Decorator (wrap), Adapter (convert interface), Template Method (skeleton + steps).
- Examples in JDK: `Comparator` (Strategy), `BufferedReader` (Decorator), `Runtime` (Singleton), `List.of` (Factory), `StringBuilder` (Builder).

---

[← Previous: Upcasting, Downcasting, Dynamic Dispatch](12-Upcasting-Downcasting-Dynamic-Dispatch.md) | [Back to README](../README.md) | [Next: OOP Interview Question Bank →](14-OOP-Interview-Question-Bank.md)
