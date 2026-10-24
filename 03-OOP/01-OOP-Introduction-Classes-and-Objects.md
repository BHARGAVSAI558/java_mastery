# Chapter 3.1 — OOP Introduction, Classes, Objects, Fields and Methods

[← Previous Part: Core Coding Problems](../02-Strings-and-Core-Programming/07-Core-Coding-Problems.md) | [Back to README](../README.md) | [Next: Constructors, this and super →](02-Constructors-this-and-super.md)

---

## What You Will Learn

- What Object-Oriented Programming is and why it is used
- Procedural vs object-oriented programming
- The four pillars of OOP (overview)
- Class and object — definitions, relationship, memory
- State, behaviour and identity of an object
- Fields (instance and static) and methods (instance and static)
- Ways to create objects in Java
- Anonymous objects, object references and garbage collection eligibility

---

## 1. Object-Oriented Programming (OOP)

### 1.1 Interview Definition

> **Object-Oriented Programming (OOP) is a programming paradigm in which a program is designed as a collection of objects that interact with each other. Each object is an instance of a class and combines data (fields, also called state or attributes) and the operations on that data (methods, also called behaviour) into a single unit. OOP is based on four main principles — encapsulation, inheritance, polymorphism and abstraction — which help make software modular, reusable, easier to maintain and closer to how we think about real-world entities.**

### 1.2 Understand the Concept

**Before OOP — procedural programming (like C):**
A program is a list of functions, and data is passed around between them. As programs grow, any function can change any data, and it becomes hard to track who changed what. Adding new features often means changing many functions.

**OOP approach:**
We model the problem using **real-world entities**. In a banking app, we create `Account`, `Customer` and `Transaction` objects. Each `Account` object holds its own data (`balance`) and the only way to change that data is through its own methods (`deposit()`, `withdraw()`). Data and the code that works on it live together.

| Point | Procedural programming | Object-oriented programming |
|---|---|---|
| Focus | Functions (procedures) | Objects (data + behaviour) |
| Program divided into | Functions | Classes and objects |
| Data access | Data often global/shared, any function can change it | Data hidden inside objects (encapsulation) |
| Reusability | Limited (copying functions) | High (inheritance, composition) |
| Maintenance of large programs | Harder | Easier — changes stay inside a class |
| Real-world modelling | Weak | Strong |
| Examples | C, Pascal | Java, C++, C#, Python |

### 1.3 The Four Pillars of OOP (overview)

| Pillar | One-line meaning | Java feature |
|---|---|---|
| **Encapsulation** | Wrap data and methods together and hide the data | `private` fields + public getters/setters |
| **Inheritance** | A class acquires properties and behaviour of another class | `extends`, `implements` |
| **Polymorphism** | One name, many forms — same method behaves differently | Overloading, overriding |
| **Abstraction** | Show only essential features, hide implementation details | Abstract classes, interfaces |

Each pillar has its own chapter in this part.

**Memory tip — "A PIE":** **A**bstraction, **P**olymorphism, **I**nheritance, **E**ncapsulation.

### 1.4 Advantages of OOP

1. **Modularity** — each class is a separate unit that can be built and tested independently.
2. **Reusability** — use existing classes through inheritance and composition.
3. **Maintainability** — a change inside a class does not break code that uses it (if the public methods stay the same).
4. **Security / data hiding** — private data can be changed only through controlled methods.
5. **Flexibility** — polymorphism lets you write code that works with new types without changes.
6. **Real-world modelling** — easier to design, discuss and understand.

### 1.5 Interview Answer — "What is OOP?"

> "Object-Oriented Programming is a way of designing programs using objects. An object represents a real-world entity and contains both data, called fields or state, and behaviour, called methods. Objects are created from classes, which act like blueprints. OOP is built on four principles: **encapsulation**, which means wrapping data and methods together and hiding the data; **inheritance**, which lets one class reuse another class's properties and behaviour; **polymorphism**, which lets the same method name behave differently; and **abstraction**, which means showing only essential features and hiding implementation details. These principles make code modular, reusable, secure and easier to maintain, which is why Java is designed around OOP."

---

## 2. Class

### 2.1 Interview Definition

> **A class is a user-defined blueprint or template from which objects are created. It defines the structure and behaviour that its objects will have, by declaring fields (variables that represent state), methods (functions that represent behaviour), constructors (to initialise new objects), and optionally blocks and nested classes. A class itself is a logical entity: it does not occupy memory for instance data until objects are created from it using the `new` keyword.**

### 2.2 Understand the Concept

```java
class Car {                 // class name
    String brand;           // field (state)
    int speed;              // field (state)

    void accelerate(int increment) {   // method (behaviour)
        speed += increment;
    }
}
```

- A class is a **design**. `Car` describes that every car will have a `brand` and `speed`, and can `accelerate`.
- A class is loaded into memory once (its metadata goes to the **method area/Metaspace**), but **instance fields** take memory only inside each object.
- A class can contain: fields, methods, constructors, initializer blocks (static and instance), nested classes/interfaces/enums/records.

---

## 3. Object

### 3.1 Interview Definition

> **An object is an instance of a class — a real entity created in heap memory at runtime, usually with the `new` keyword. Every object has three characteristics: state, which is the data stored in its fields; behaviour, which is defined by its methods; and identity, which uniquely distinguishes it from other objects, even if their state is the same. Multiple objects can be created from one class, and each object has its own copy of the instance variables.**

### 3.2 Understand the Concept

| Characteristic | Meaning | Car example |
|---|---|---|
| **State** | Values of its fields at a moment | `brand = "Tata", speed = 60` |
| **Behaviour** | What it can do (methods) | `accelerate()`, `brake()` |
| **Identity** | What makes it unique (its memory reference) | Two cars with the same brand and speed are still two different cars |

**Creating an object — what happens in `Car c1 = new Car();`:**

1. `Car c1` — declares a **reference variable** `c1` on the stack (it can point to a `Car`).
2. `new Car()` — the JVM allocates memory in the **heap** for a new `Car` object; all instance fields get **default values** (`null`, `0`).
3. The **constructor** runs and initialises the object.
4. `new` returns the **reference** (address) of the object, which is stored in `c1`.

```text
     STACK                         HEAP
  c1 [ref ●]────────────►  Car object { brand = "Tata", speed = 60 }
  c2 [ref ●]────────────►  Car object { brand = "Kia",  speed = 0  }
  c3 [ref ●]───┐
               └─────────►  (same object as c2 — no new object created by c3 = c2)
```

### 3.3 Class vs Object

| Point | Class | Object |
|---|---|---|
| Meaning | Blueprint / template | Instance of the blueprint |
| Nature | Logical entity | Physical (runtime) entity |
| Memory | No memory for instance data | Memory allocated in the heap |
| Created | Declared once with `class` keyword | Created many times with `new` |
| Example | `class Student` | `new Student("Ravi")` |

### 3.4 Real-World Example

A **class** is the **architect's plan of a house**: it describes rooms, doors and windows, but you cannot live in a plan. An **object** is an **actual house built from that plan**. From one plan, a builder can build 100 houses. Each house (object) has its own paint colour and owner (state), all houses can open doors and switch lights on (behaviour), and each has a unique address (identity).

### 3.5 Java Example

```java
public class CarDemo {

    static class Car {
        // Fields (instance variables) - each Car has its own copy
        String brand;
        String color;
        int speed;

        // Static field - shared by all cars
        static int carsCreated = 0;

        Car(String brand, String color) {      // constructor
            this.brand = brand;
            this.color = color;
            carsCreated++;
        }

        // Instance methods - behaviour
        void accelerate(int increment) {
            speed += increment;
            System.out.println(brand + " accelerates to " + speed + " km/h");
        }

        void brake() {
            speed = 0;
            System.out.println(brand + " stops");
        }

        String describe() {
            return color + " " + brand + " at " + speed + " km/h";
        }

        // Static method - belongs to the class
        static int getCarsCreated() {
            return carsCreated;
        }
    }

    public static void main(String[] args) {
        Car c1 = new Car("Tata", "Red");
        Car c2 = new Car("Kia", "White");

        c1.accelerate(40);
        c1.accelerate(20);
        c2.accelerate(30);
        c2.brake();

        System.out.println("c1: " + c1.describe());
        System.out.println("c2: " + c2.describe());

        Car c3 = c2;                    // c3 refers to the SAME object as c2
        c3.color = "Black";
        System.out.println("c2 after changing c3: " + c2.describe());
        System.out.println("c2 == c3 ? " + (c2 == c3));

        Car c4 = new Car("Kia", "Black");     // same state, different identity
        System.out.println("c2 == c4 ? " + (c2 == c4));

        System.out.println("Cars created: " + Car.getCarsCreated());

        new Car("Honda", "Blue").accelerate(10);   // anonymous object: used once
        System.out.println("Cars created: " + Car.getCarsCreated());
    }
}
```

### 3.6 Code Explanation

- `Car` is a **class** (here written as a static nested class so the whole example fits in one file).
- `brand`, `color`, `speed` are **instance fields** — `c1` and `c2` each have their own values.
- `carsCreated` is a **static field** — one copy shared by all cars; the constructor increases it.
- `c1.accelerate(40)` — calling an instance method on an object: it changes **only `c1`'s** `speed`.
- `Car c3 = c2;` copies the **reference**, not the object. Changing `c3.color` changes the same object `c2` refers to.
- `c2 == c4` is `false` — even with the same state, they are different objects (different **identity**).
- `new Car("Honda", "Blue").accelerate(10);` — an **anonymous object**: it has no reference variable, so after this line it becomes eligible for garbage collection.

### 3.7 Expected Output

```text
Tata accelerates to 40 km/h
Tata accelerates to 60 km/h
Kia accelerates to 30 km/h
Kia stops
c1: Red Tata at 60 km/h
c2: White Kia at 0 km/h
c2 after changing c3: Black Kia at 0 km/h
c2 == c3 ? true
c2 == c4 ? false
Cars created: 3
Honda accelerates to 10 km/h
Cars created: 4
```

### 3.8 Interview Answer — "What is the difference between a class and an object?"

> "A class is a blueprint or template that defines what data and behaviour its objects will have — it declares fields, methods and constructors. It is a logical entity and does not hold instance data by itself. An object is an actual instance of a class created at runtime, usually with the `new` keyword, and it occupies memory in the heap. Every object has its own state, which is the values of its fields; behaviour, which comes from the class's methods; and identity, which makes it unique. For example, `Student` is a class, and `new Student("Ravi")` and `new Student("Priya")` are two objects of that class with different states. One class can have many objects."

---

## 4. Fields and Methods in Detail

### 4.1 Fields (Instance vs Static)

> **Fields are variables declared inside a class but outside any method. Instance fields belong to individual objects — every object gets its own copy, stored inside the object in the heap. Static fields belong to the class — there is only one copy, shared by all objects. Fields get default values if not initialised.**

| | Instance field | Static field |
|---|---|---|
| Declared with | no `static` | `static` |
| Copies | One per object | One per class |
| Access | `object.field` | `ClassName.field` |
| Example | `student.name` | `Student.collegeName` |

### 4.2 Methods (Instance vs Static)

> **Methods define the behaviour of a class. Instance methods operate on a specific object and can access both instance and static members using `this`. Static methods belong to the class, are called using the class name, and can directly access only static members, because they do not have a `this` reference.**

**What can access what?**

| From ↓ / Access → | Instance field/method | Static field/method |
|---|---|---|
| **Instance method** | ✅ Directly | ✅ Directly |
| **Static method** | ❌ Only through an object reference | ✅ Directly |

### 4.3 Getter, Setter and Utility Methods

Common method categories in a class:
- **Accessor (getter):** returns a field value — `getName()`.
- **Mutator (setter):** changes a field value with validation — `setAge(int age)`.
- **Business methods:** real work — `withdraw(amount)`, `calculateGrade()`.
- **`toString()`, `equals()`, `hashCode()`:** inherited from `Object` and usually overridden ([Chapter 3.10](10-Object-Class-equals-hashCode-toString.md)).

---

## 5. Ways to Create Objects in Java

| # | Way | Example | Constructor called? |
|---|---|---|---|
| 1 | `new` keyword | `Student s = new Student();` | Yes |
| 2 | Reflection | `Student.class.getDeclaredConstructor().newInstance()` | Yes |
| 3 | `clone()` | `Student s2 = (Student) s1.clone();` | **No** |
| 4 | Deserialization | `(Student) objectInputStream.readObject()` | **No** (for the Serializable class itself) |
| 5 | Factory / builder methods (internally use `new`) | `List.of()`, `Integer.valueOf(5)`, `LocalDate.now()` | Yes (internally) |

```java
public class ObjectCreationWays {

    static class Student implements Cloneable {
        String name;

        Student() {
            this.name = "Default";
            System.out.println("Constructor called");
        }

        @Override
        protected Student clone() throws CloneNotSupportedException {
            return (Student) super.clone();
        }
    }

    public static void main(String[] args) throws Exception {
        System.out.println("1. Using new:");
        Student s1 = new Student();

        System.out.println("2. Using reflection:");
        Student s2 = Student.class.getDeclaredConstructor().newInstance();

        System.out.println("3. Using clone:");
        s1.name = "Ravi";
        Student s3 = s1.clone();               // no constructor call printed
        System.out.println("Clone name: " + s3.name + ", same object? " + (s1 == s3));
    }
}
```

**Expected Output:**

```text
1. Using new:
Constructor called
2. Using reflection:
Constructor called
3. Using clone:
Clone name: Ravi, same object? false
```

`clone()` copies the object's fields directly in memory, so the constructor is **not** executed. The clone is a different object with the same state. (Cloning is discussed in [Chapter 3.10](10-Object-Class-equals-hashCode-toString.md); deserialization in [Part 10](../10-Additional-Java-Topics/05-Serialization-and-Deserialization.md).)

---

## 6. Object References, `null` and Garbage Collection Eligibility

An object becomes **eligible for garbage collection** when no live reference points to it. Common ways:

```java
public class GcEligibility {
    static class Box {
        String label;
        Box(String label) { this.label = label; }
    }

    static void createLocal() {
        Box local = new Box("local");     // eligible after method returns
    }

    public static void main(String[] args) {
        Box a = new Box("A");
        a = null;                          // 1. nullifying the reference → "A" is eligible

        Box b = new Box("B1");
        b = new Box("B2");                 // 2. re-assigning → "B1" is eligible

        createLocal();                     // 3. local object out of scope → eligible

        new Box("anonymous");              // 4. anonymous object → eligible immediately

        Box c = new Box("C");
        Box d = c;
        c = null;                          // "C" is NOT eligible: d still refers to it
        System.out.println("Still reachable through d: " + d.label);
    }
}
```

**Expected Output:**

```text
Still reachable through d: C
```

Eligible does not mean "collected immediately" — the garbage collector decides when to run. Details in [Part 9](../09-JVM-and-Memory/04-Garbage-Collection-and-Memory-Leaks.md).

---

## 7. Interview Questions with Complete Answers

### Basic Level

**Q1. What are the main principles of OOP?**

Encapsulation, inheritance, polymorphism and abstraction. **Encapsulation** binds data and methods together and hides data using private fields. **Inheritance** lets a child class reuse and extend a parent class using `extends`. **Polymorphism** allows one method name to behave differently — through overloading (compile time) and overriding (runtime). **Abstraction** hides implementation details and shows only essential behaviour, using abstract classes and interfaces.

**Q2. What is an object? What are its characteristics?**

An object is an instance of a class created in heap memory at runtime. It has **state** (field values), **behaviour** (methods) and **identity** (a unique reference that distinguishes it from other objects, even with the same state).

**Q3. How much memory does a class take?**

A class definition is loaded once; its metadata (method code, field descriptions, constant pool) is stored in **Metaspace**, and static fields are stored with the class. **Instance fields take no memory until an object is created**; each object then gets its own memory in the heap for its instance fields plus an object header.

### Intermediate Level

**Q4. What is the difference between a reference variable and an object?**

The **object** is the actual data structure in the heap. A **reference variable** is a variable (often on the stack) that holds the address/reference of the object. Many references can point to the same object, and a reference can be `null` (pointing to nothing). In `Car c = new Car();`, `c` is the reference and `new Car()` creates the object.

**Q5. Can we create an object without using `new`?**

Yes — using reflection (`Constructor.newInstance()`), `clone()`, deserialization, or factory methods (which use `new` internally, like `Integer.valueOf()` and `List.of()`). Also, string literals and array initializers (`"abc"`, `{1, 2}`) create objects without writing `new`.

**Q6. Why can't a static method access instance variables directly?**

A static method belongs to the class and can be called without any object. Instance variables exist only inside objects. If no object exists, there is no instance variable to access, and even if many objects exist, the static method would not know which object's variable you mean. So static methods have no `this` reference. They can access instance members only through an explicit object reference.

**Q7. Is Java a pure object-oriented language?**

No. A pure OOP language treats everything as an object. Java has primitive types (`int`, `boolean`, …) that are not objects, and static members that can be used without objects. However, Java is strongly object-oriented, and wrapper classes allow primitives to be treated as objects when needed.

### Advanced Level

**Q8. What is the difference between object-oriented and object-based languages?**

An **object-based** language supports objects and encapsulation but not all OOP features, especially **inheritance** and polymorphism through inheritance (for example, early JavaScript and VBScript). An **object-oriented** language supports all OOP principles including inheritance and runtime polymorphism (Java, C++, C#).

**Q9. What happens in memory when `Student s = new Student("Ravi");` is executed?**

1. If not already loaded, the `Student` class is loaded, linked and initialised (static blocks run).
2. Memory for the new object is allocated in the heap (usually in the young generation's Eden space), sized for its fields plus an object header.
3. Fields get default values (`null`, `0`, `false`).
4. Field initialisers and instance initializer blocks run, then the constructor body runs (after the parent constructor via `super()`).
5. The reference to the object is returned and stored in `s`, which lives in the current method's stack frame.

**Q10. What is an anonymous object? When is it useful?**

An object created without storing its reference in a variable, for example `new Calculator().add(2, 3);`. It is useful when the object is needed only once. After that statement, it is immediately eligible for garbage collection.

---

## 8. Follow-Up Questions

**F1. If two objects have the same state, are they equal?**

With `==`, no — they are different objects. With `equals()`, it depends on whether the class overrides `equals()` to compare state. By default (from `Object`), `equals()` behaves like `==`.

**F2. Can a class be empty? Can an object of an empty class be created?**

Yes, `class Empty {}` is valid. The compiler adds a default constructor, so `new Empty()` works. The object still has an object header (identity hash code, lock information, class pointer).

**F3. What is the default value of an object reference field?**

`null`. Calling a method on it throws `NullPointerException`.

**F4. Give a real project example of classes and objects.**

In a library management system: classes `Book`, `Member`, `Loan`, `Library`. Objects: each actual book (`new Book("Clean Code", "Robert Martin")`), each registered member, each loan record. `Library` manages lists of books and members and has methods like `issueBook(member, book)`.

---

## 9. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| "A class is an object." | A class is a blueprint; objects are instances of it. |
| Thinking `Car c2 = c1;` creates a new object | It copies the reference only. |
| Accessing instance fields from `main` directly | `main` is static; use an object. |
| "Objects are stored in the stack." | Objects are in the heap; references (locals) are in the stack. |
| Saying "Java is 100% OOP" | Not pure — primitives and static members. |
| Calling a method on a `null` reference | `NullPointerException`. |

---

## 10. Quick Revision

- OOP = programs built from **objects** (state + behaviour); pillars: **Abstraction, Polymorphism, Inheritance, Encapsulation (A PIE)**.
- **Class** = blueprint (logical); **object** = instance in heap (physical), created with `new`.
- Object = **state + behaviour + identity**.
- `new` → allocate heap memory → defaults → initialisers/constructor → return reference.
- Instance fields: per object. Static fields: one per class.
- Static methods cannot use instance members directly (no `this`).
- Ways to create objects: `new`, reflection, `clone()` (no constructor), deserialization (no constructor), factory methods.
- Object eligible for GC when unreachable (null, reassigned, out of scope, anonymous).

---

[← Previous Part: Core Coding Problems](../02-Strings-and-Core-Programming/07-Core-Coding-Problems.md) | [Back to README](../README.md) | [Next: Constructors, this and super →](02-Constructors-this-and-super.md)
