# Chapter 3.11 — Association, Aggregation and Composition (IS-A and HAS-A)

[← Previous: Object Class](10-Object-Class-equals-hashCode-toString.md) | [Back to README](../README.md) | [Next: Upcasting, Downcasting, Dynamic Dispatch →](12-Upcasting-Downcasting-Dynamic-Dispatch.md)

---

## What You Will Learn

- IS-A (inheritance) vs HAS-A (association) relationships
- Association — one-to-one, one-to-many, many-to-many; unidirectional vs bidirectional
- Aggregation — weak HAS-A (parts can exist independently)
- Composition — strong HAS-A (parts cannot exist without the whole)
- How to tell them apart in code and in interviews
- **Composition over inheritance** with a practical example

---

## 1. IS-A vs HAS-A

### 1.1 Interview Definition

> **An IS-A relationship represents inheritance, where one class is a specialised type of another class — for example, a `Car` IS-A `Vehicle` — and it is implemented using `extends` or `implements`. A HAS-A relationship represents association, where one class contains or uses an object of another class as a field — for example, a `Car` HAS-A `Engine`. HAS-A relationships are further divided into aggregation, a weak relationship in which the contained object can exist independently, and composition, a strong relationship in which the contained object's life cycle depends on the container.**

### 1.2 Overview Diagram

```text
                    Relationships between classes
                    ┌────────────┴─────────────┐
               IS-A (Inheritance)         HAS-A (Association)
               extends / implements       field reference
               Car IS-A Vehicle           ┌────────┴─────────┐
                                     Aggregation          Composition
                                     (weak HAS-A)         (strong HAS-A)
                                     Department ◇── Teacher   House ◆── Room
                                     part can exist alone     part dies with whole
```

UML notation: **hollow diamond ◇** = aggregation, **filled diamond ◆** = composition, plain line = association, hollow triangle arrow = inheritance.

| | IS-A | HAS-A |
|---|---|---|
| Meaning | "is a kind of" | "has a" / "uses a" |
| Java | `extends`, `implements` | Instance field of another class's type |
| Coupling | Tight (compile-time, fixed) | Loose (can change at runtime) |
| Reuse type | White-box (child sees parent internals) | Black-box (uses only public methods) |
| Polymorphism | Built in | Through interfaces |
| Example | `Dog extends Animal` | `Dog` has a `Collar` |

---

## 2. Association

### 2.1 Interview Definition

> **Association is a general relationship between two separate classes in which objects of one class are connected to or use objects of another class. Both objects have their own independent life cycles, and neither owns the other. Association can be one-to-one, one-to-many, many-to-one or many-to-many, and it can be unidirectional (only one class knows about the other) or bidirectional (both classes know each other). Aggregation and composition are special, stronger forms of association.**

### 2.2 Understand the Concept

- **Teacher ↔ Student**: a teacher teaches many students, a student learns from many teachers (**many-to-many**). Neither owns the other; both exist independently.
- **Doctor → Patient**: a doctor treats patients (**one-to-many**), but patients exist without that doctor.
- **Person → Passport**: one person has one passport (**one-to-one**).

**Direction:**
- **Unidirectional:** `Order` knows its `Customer`, but `Customer` doesn't keep a list of orders.
- **Bidirectional:** `Teacher` has a list of `Student`s and each `Student` has a list of `Teacher`s.

### 2.3 Java Example — Association

```java
import java.util.ArrayList;
import java.util.List;

public class AssociationDemo {

    static class Student {
        private final String name;
        private final List<Teacher> teachers = new ArrayList<>();
        Student(String name) { this.name = name; }
        void addTeacher(Teacher t) { teachers.add(t); }
        String getName() { return name; }
        List<Teacher> getTeachers() { return teachers; }
    }

    static class Teacher {
        private final String name;
        private final List<Student> students = new ArrayList<>();
        Teacher(String name) { this.name = name; }

        void teach(Student s) {                 // bidirectional link
            students.add(s);
            s.addTeacher(this);
        }
        String getName() { return name; }
        List<Student> getStudents() { return students; }
    }

    public static void main(String[] args) {
        Teacher math = new Teacher("Mrs. Rao");
        Teacher physics = new Teacher("Mr. Khan");
        Student ravi = new Student("Ravi");
        Student priya = new Student("Priya");

        math.teach(ravi);
        math.teach(priya);
        physics.teach(ravi);

        System.out.print(math.getName() + " teaches: ");
        math.getStudents().forEach(s -> System.out.print(s.getName() + " "));
        System.out.println();
        System.out.print(ravi.getName() + " learns from: ");
        ravi.getTeachers().forEach(t -> System.out.print(t.getName() + " "));
        System.out.println();

        math = null;                             // teacher object gone from our reference
        System.out.println("Ravi still exists: " + ravi.getName());
    }
}
```

**Expected Output:**

```text
Mrs. Rao teaches: Ravi Priya 
Ravi learns from: Mrs. Rao Mr. Khan 
Ravi still exists: Ravi
```

**Explanation:** Teachers and students are created **independently** and linked later (many-to-many, bidirectional). Removing our reference to a teacher does not destroy the students — their life cycles are independent.

---

## 3. Aggregation (Weak HAS-A)

### 3.1 Interview Definition

> **Aggregation is a special form of association that represents a whole–part (HAS-A) relationship in which the part can exist independently of the whole. The container object holds references to part objects, but it does not control their life cycle — the parts are usually created outside and passed in, and they can be shared with other containers. If the container is destroyed, the parts continue to exist. For example, a `Department` has `Teacher`s, but a teacher can exist without the department or move to another one.**

### 3.2 Real-World Example

A **cricket team** has **players**. If the team is dissolved, the players still exist and can join other teams. A **playlist** has **songs**; deleting a playlist does not delete the songs from your library.

### 3.3 Java Example — Aggregation

```java
import java.util.ArrayList;
import java.util.List;

public class AggregationDemo {

    static class Player {
        final String name;
        Player(String name) { this.name = name; }
    }

    static class Team {
        private final String teamName;
        private final List<Player> players = new ArrayList<>();

        Team(String teamName) { this.teamName = teamName; }

        void addPlayer(Player p) {               // part is created OUTSIDE and passed in
            players.add(p);
        }

        void show() {
            System.out.print(teamName + ": ");
            players.forEach(p -> System.out.print(p.name + " "));
            System.out.println();
        }
    }

    public static void main(String[] args) {
        Player virat = new Player("Virat");      // players exist independently
        Player rohit = new Player("Rohit");

        Team india = new Team("India");
        india.addPlayer(virat);
        india.addPlayer(rohit);

        Team rcb = new Team("RCB");
        rcb.addPlayer(virat);                    // same player shared by another team

        india.show();
        rcb.show();

        india = null;                            // team destroyed
        System.out.println("Player still exists after team removed: " + virat.name);
    }
}
```

**Expected Output:**

```text
India: Virat Rohit 
RCB: Virat 
Player still exists after team removed: Virat
```

**Key signs of aggregation in code:**
- Parts are **passed in** (constructor/setter/add method), not created inside.
- Parts can be **shared** between containers.
- Parts **outlive** the container.

---

## 4. Composition (Strong HAS-A)

### 4.1 Interview Definition

> **Composition is a strong form of association that represents a whole–part relationship in which the part cannot exist independently of the whole. The container object owns its parts: it usually creates them itself, does not share them, and when the container is destroyed, its parts are destroyed with it. For example, a `House` is composed of `Room`s — a room has no meaning without its house. Composition provides strong encapsulation and is generally preferred over inheritance for code reuse.**

### 4.2 Real-World Example

A **human body** and its **heart**: the heart cannot live as part of another body (in this model), and its life ends with the body. A **book** and its **pages**; an **order** and its **order lines**.

### 4.3 Java Example — Composition

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class CompositionDemo {

    static class Engine {
        private final int horsePower;
        private boolean running;
        Engine(int horsePower) { this.horsePower = horsePower; }
        void start() { running = true; System.out.println("  Engine (" + horsePower + " HP) started"); }
        void stop()  { running = false; System.out.println("  Engine stopped"); }
    }

    static class Car {
        private final String model;
        private final Engine engine;                         // composed part
        private final List<String> wheels;                   // composed parts

        Car(String model, int hp) {
            this.model = model;
            this.engine = new Engine(hp);                    // created INSIDE, owned by Car
            List<String> w = new ArrayList<>();
            for (int i = 1; i <= 4; i++) w.add("Wheel-" + i);
            this.wheels = Collections.unmodifiableList(w);
        }

        void drive() {
            System.out.println(model + " driving:");
            engine.start();                                  // delegation to the part
            System.out.println("  Rolling on " + wheels.size() + " wheels");
            engine.stop();
        }
        // No getEngine() that exposes the engine for sharing → strong ownership
    }

    public static void main(String[] args) {
        Car car = new Car("Swift", 89);
        car.drive();
        car = null;      // Car and its Engine become unreachable together
        System.out.println("Car removed; its engine has no independent use");
    }
}
```

**Expected Output:**

```text
Swift driving:
  Engine (89 HP) started
  Rolling on 4 wheels
  Engine stopped
Car removed; its engine has no independent use
```

**Key signs of composition in code:**
- The part is **created inside** the container (often in the constructor) and stored in a `private final` field.
- The part is **not shared** and usually not exposed (or exposed only as a copy/read-only view).
- When the container becomes unreachable, the part becomes unreachable too.
- The container **delegates** work to the part (`engine.start()`).

> Java has garbage collection, so "destroyed together" means "both become unreachable and are collected together" — there is no explicit delete as in C++.

---

## 5. Association vs Aggregation vs Composition

| Point | Association | Aggregation | Composition |
|---|---|---|---|
| Relationship | Uses-a / knows-a | Weak HAS-A (whole–part) | Strong HAS-A (whole–part, owns) |
| Ownership | None | Container does **not** own the part | Container **owns** the part |
| Life cycle | Independent | Part can exist without the whole | Part dies with the whole |
| Sharing parts | Yes | Yes (part can belong to many) | No (part belongs to one whole) |
| Part created | Anywhere | Outside, passed in | Inside the container |
| UML | Line | Hollow diamond ◇ | Filled diamond ◆ |
| Example | Teacher–Student, Doctor–Patient | Team–Player, Department–Teacher, Playlist–Song | House–Room, Car–Engine, Order–OrderLine |
| Coupling strength | Weakest | Medium | Strongest |

### 5.1 Interview Answer

> "Association, aggregation and composition are all ways one class relates to another through object references — the HAS-A relationship. **Association** is the general term: two objects are connected, like a teacher and a student, and both have independent life cycles. **Aggregation** is a weak whole–part relationship: the container has parts, but the parts can exist independently and can even be shared — for example, a team has players, and if the team is dissolved the players still exist. **Composition** is a strong whole–part relationship: the container owns the parts, usually creates them itself, and the parts cannot exist without it — like a house and its rooms, or a car and its engine in my design. In code, in aggregation the part is passed in from outside, while in composition the part is created inside and kept private. All three are different from IS-A, which is inheritance."

---

## 6. Composition Over Inheritance

### 6.1 Interview Definition

> **"Favour composition over inheritance" is a design principle that recommends reusing behaviour by holding a reference to another object (HAS-A) and delegating work to it, instead of extending a class (IS-A). Composition gives loose coupling, lets behaviour change at runtime, avoids the fragile base class problem and the single-inheritance limit, and exposes only the methods we choose. Inheritance should be used only for genuine, stable IS-A relationships.**

### 6.2 Java Example — Inheritance Problem and Composition Fix

```java
import java.util.ArrayList;
import java.util.List;

public class CompositionOverInheritance {

    // BAD: inheritance exposes all ArrayList methods, breaking stack rules
    static class InheritedStack<E> extends ArrayList<E> {
        void push(E e) { add(e); }
        E pop() { return remove(size() - 1); }
    }

    // GOOD: composition exposes only stack operations
    static class ComposedStack<E> {
        private final List<E> items = new ArrayList<>();     // HAS-A list
        void push(E e) { items.add(e); }
        E pop() {
            if (items.isEmpty()) throw new IllegalStateException("Stack is empty");
            return items.remove(items.size() - 1);
        }
        E peek() { return items.get(items.size() - 1); }
        boolean isEmpty() { return items.isEmpty(); }
        int size() { return items.size(); }
    }

    // Behaviour that can change at runtime through composition (Strategy idea)
    interface DiscountPolicy { double apply(double amount); }

    static class Checkout {
        private DiscountPolicy policy;                       // composed behaviour
        Checkout(DiscountPolicy policy) { this.policy = policy; }
        void setPolicy(DiscountPolicy policy) { this.policy = policy; }
        double total(double amount) { return policy.apply(amount); }
    }

    public static void main(String[] args) {
        InheritedStack<String> bad = new InheritedStack<>();
        bad.push("A");
        bad.push("B");
        bad.add(0, "HACK");                                  // inserting at the bottom of a "stack"!
        System.out.println("Inherited stack allows: " + bad);

        ComposedStack<String> good = new ComposedStack<>();
        good.push("A");
        good.push("B");
        // good.add(0, "HACK");                              // compile error: no such method
        System.out.println("Composed stack pop: " + good.pop() + ", size now " + good.size());

        Checkout checkout = new Checkout(amount -> amount);              // no discount
        System.out.println("Normal day: " + checkout.total(1000));
        checkout.setPolicy(amount -> amount * 0.8);                      // swap behaviour at runtime
        System.out.println("Sale day: " + checkout.total(1000));
    }
}
```

**Expected Output:**

```text
Inherited stack allows: [HACK, A, B]
Composed stack pop: B, size now 1
Normal day: 1000.0
Sale day: 800.0
```

**Explanation:**
- `InheritedStack` IS-A `ArrayList`, so it inherits `add(index, e)`, `remove(index)`, `set()`, etc. — users can break stack behaviour. This is the same design mistake as `java.util.Stack extends Vector`.
- `ComposedStack` HAS-A `List` and exposes only `push`, `pop`, `peek`. The internal list can even be changed to `ArrayDeque` later without affecting users.
- `Checkout` HAS-A `DiscountPolicy`; the policy can be **replaced at runtime** — impossible with inheritance, where behaviour is fixed when you choose the parent class.

### 6.3 Inheritance vs Composition

| Point | Inheritance (IS-A) | Composition (HAS-A) |
|---|---|---|
| Coupling | Tight | Loose |
| Behaviour change | Fixed at compile time | Can change at runtime |
| Exposes | All public/protected parent methods | Only chosen methods |
| Encapsulation | Weaker (child depends on parent internals) | Stronger |
| Multiple sources of reuse | One parent only | Many components |
| Testing | Harder (need real parent) | Easier (mock the component) |
| Use when | True IS-A, parent designed for extension | Code reuse, flexible behaviour |

---

## 7. Interview Questions with Complete Answers

### Basic Level

**Q1. What is a HAS-A relationship? Give an example.**

A HAS-A relationship means one class contains a reference to another class as a field. Example: `class Library { private List<Book> books; }` — a library has books. It is implemented through association, aggregation or composition, not through `extends`.

**Q2. What is the difference between aggregation and composition?**

Both are whole–part HAS-A relationships. In **aggregation**, the part can exist independently and may be shared (Team–Player). In **composition**, the part belongs to exactly one whole and its life ends with the whole (House–Room). In code, aggregated parts are usually passed in from outside; composed parts are usually created inside the container.

### Intermediate Level

**Q3. Why is composition preferred over inheritance?**

Composition gives looser coupling, stronger encapsulation, the ability to change behaviour at runtime, and freedom from Java's single-inheritance limit. It also avoids the fragile base class problem and avoids inheriting methods that don't make sense (like `Stack` inheriting `Vector.add(index, e)`). Inheritance is still right for genuine IS-A relationships where the subclass can fully substitute the parent.

**Q4. How do you identify the relationship type from a requirement?**

Ask:
1. "Is X a kind of Y?" → **IS-A** (inheritance).
2. "Does X have/use Y?" → **HAS-A**.
3. "Can Y exist without X, or be shared?" → yes → **aggregation**; no → **composition**.
4. If it's not whole–part at all, just a link (teacher–student) → **association**.

**Q5. Is a `Car`–`Engine` relationship aggregation or composition?**

It depends on the design. If the engine is created inside the car and never reused, it is **composition**. In a car-repair system where engines are tracked separately and moved between cars, it is **aggregation**. Interviewers want to see that you reason from **life cycle and ownership**, not from the class names.

### Advanced Level

**Q6. How does composition support the Strategy and Decorator patterns?**

- **Strategy:** a class holds a reference to an interface (`DiscountPolicy`, `Comparator`) and delegates behaviour to it; you can swap implementations at runtime.
- **Decorator:** an object wraps another object of the same interface and adds behaviour before/after delegating (e.g., `new BufferedReader(new FileReader(file))` in Java I/O).
Both rely on HAS-A rather than IS-A for reuse.

**Q7. How is composition represented in databases/JPA?**

With **cascade** and **orphan removal**: `@OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true) List<OrderLine> lines;` — saving/deleting the `Order` saves/deletes its lines, and lines cannot exist without an order. Aggregation would not cascade deletes.

---

## 8. Follow-Up Questions

**F1. Can a class have both IS-A and HAS-A relationships?**

Yes, and it is very common: `class Car extends Vehicle { private Engine engine; }` — a car IS-A vehicle and HAS-A engine.

**F2. Is dependency (uses-a) the same as association?**

Dependency is weaker: a class **uses** another temporarily, for example as a method parameter or local variable (`void print(Printer p)`), without storing it as a field. Association stores a long-term reference as a field.

**F3. In composition, should getters return the internal part?**

Avoid returning the internal mutable part directly, because outside code could then share or modify it, weakening ownership. Return a copy, an unmodifiable view, or only the data needed.

---

## 9. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Using inheritance just to reuse code | Use composition unless there is a true IS-A. |
| Deciding aggregation vs composition by names only | Decide by ownership and life cycle. |
| Exposing composed parts through getters | Weakens composition; return copies/views. |
| Saying "association = composition" | Composition is the strongest special case of association. |
| Thinking Java deletes composed parts explicitly | GC collects them when unreachable. |

---

## 10. Quick Revision

- **IS-A** = inheritance (`extends`/`implements`), tight coupling.
- **HAS-A** = field reference: association → aggregation (weak) → composition (strong).
- **Association:** independent objects linked (Teacher–Student); 1-1, 1-many, many-many; uni/bidirectional.
- **Aggregation ◇:** part exists independently, can be shared, passed in from outside (Team–Player).
- **Composition ◆:** part owned, created inside, not shared, dies with whole (House–Room).
- **Favour composition over inheritance**: loose coupling, runtime flexibility, only chosen methods exposed (`Stack extends Vector` mistake).

---

[← Previous: Object Class](10-Object-Class-equals-hashCode-toString.md) | [Back to README](../README.md) | [Next: Upcasting, Downcasting, Dynamic Dispatch →](12-Upcasting-Downcasting-Dynamic-Dispatch.md)
