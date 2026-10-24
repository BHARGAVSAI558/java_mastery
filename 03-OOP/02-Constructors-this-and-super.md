# Chapter 3.2 — Constructors, Constructor Overloading, `this` and `super`

[← Previous: Classes and Objects](01-OOP-Introduction-Classes-and-Objects.md) | [Back to README](../README.md) | [Next: Encapsulation →](03-Encapsulation.md)

---

## What You Will Learn

- What a constructor is, its rules, and how it differs from a method
- Default, no-argument and parameterised constructors
- Constructor overloading and constructor chaining with `this()`
- The `this` keyword — all six uses
- The `super` keyword — calling parent constructors, methods and fields
- Order of execution when objects are created in an inheritance chain
- Private constructors, copy constructors and tricky interview questions

---

## 1. Constructors

### 1.1 Interview Definition

> **A constructor is a special block of code that is called automatically when an object is created using the `new` keyword, and it is used to initialise the new object's state. A constructor has the same name as the class and has no return type, not even `void`. If a class does not declare any constructor, the Java compiler automatically provides a default no-argument constructor. Constructors can be overloaded, can call other constructors of the same class using `this()`, and always call a parent class constructor using `super()`, either explicitly or implicitly as their first statement.**

### 1.2 Understand the Concept

**Why do we need constructors?**
Without a constructor, every new object would start with default values (`null`, `0`), and we would have to set each field one by one after creation — easy to forget and error-prone. A constructor makes sure an object is **valid from the moment it is created**.

**Rules of constructors:**
1. The name must be **exactly the same as the class name**.
2. **No return type** — not even `void`. (If you write `void Student()`, it becomes a normal method, not a constructor!)
3. Called **automatically** once per object, by `new`.
4. Can have **access modifiers** (`public`, `protected`, default, `private`).
5. **Cannot** be `static`, `final`, `abstract` or `synchronized`.
6. Can be **overloaded**, but **cannot be overridden** or **inherited**.
7. The first statement is always either `this(...)` or `super(...)`. If you write neither, the compiler inserts `super();`.

### 1.3 Types of Constructors

| Type | Description | Example |
|---|---|---|
| **Default constructor** | Added by the **compiler** only when the class has **no** constructor at all. No parameters; calls `super()`. | (invisible) `Student() { super(); }` |
| **No-argument constructor** | Written by the programmer with no parameters | `Student() { name = "Unknown"; }` |
| **Parameterised constructor** | Takes parameters to initialise fields | `Student(String name, int age) { ... }` |
| **Copy constructor** (concept) | Takes an object of the same class and copies its state | `Student(Student other) { ... }` |
| **Private constructor** | Prevents object creation from outside the class | Singletons, utility classes |

> **Important:** As soon as you write **any** constructor, the compiler **no longer adds the default constructor**. Then `new Student()` fails to compile unless you write a no-argument constructor yourself.

### 1.4 Constructor vs Method

| Point | Constructor | Method |
|---|---|---|
| Purpose | Initialise a new object | Perform an operation |
| Name | Same as class | Any valid name |
| Return type | None | Must have one (`void` or a type) |
| Called | Automatically by `new` | Explicitly by name |
| Times called per object | Once | Any number of times |
| Inherited | No | Yes (non-private) |
| Can be overridden | No | Yes |
| Can be `static`/`final`/`abstract` | No | Yes |
| Default provided by compiler | Yes (if none written) | No |

### 1.5 Real-World Example

A constructor is like the **admission process at a college**. When a new student joins (object created), the office fills the admission form: name, roll number, branch (initialise fields). It happens **once**, automatically, at the moment of joining. Different admission types (regular, lateral entry, management quota) are like **overloaded constructors** — different inputs, same purpose.

### 1.6 Java Example — Types and Overloading

```java
public class ConstructorDemo {

    static class Student {
        String name;
        int age;
        String branch;

        // No-argument constructor
        Student() {
            this.name = "Unknown";
            this.age = 18;
            this.branch = "Not assigned";
            System.out.println("No-arg constructor called");
        }

        // Parameterised constructor
        Student(String name, int age) {
            this.name = name;
            this.age = age;
            this.branch = "CSE";
            System.out.println("2-arg constructor called");
        }

        // Overloaded with 3 parameters
        Student(String name, int age, String branch) {
            this.name = name;
            this.age = age;
            this.branch = branch;
            System.out.println("3-arg constructor called");
        }

        // Copy constructor
        Student(Student other) {
            this.name = other.name;
            this.age = other.age;
            this.branch = other.branch;
            System.out.println("Copy constructor called");
        }

        void display() {
            System.out.println("  " + name + ", " + age + ", " + branch);
        }
    }

    static class Laptop {
        String model;
        // no constructor written → compiler adds a default constructor
    }

    public static void main(String[] args) {
        Student s1 = new Student();
        s1.display();

        Student s2 = new Student("Ravi", 20);
        s2.display();

        Student s3 = new Student("Priya", 21, "ECE");
        s3.display();

        Student s4 = new Student(s3);
        s4.name = "Priya Copy";
        s4.display();
        s3.display();                          // original unchanged

        Laptop laptop = new Laptop();          // default constructor
        System.out.println("Laptop model (default value): " + laptop.model);
    }
}
```

### 1.7 Code Explanation

- `new Student()` → calls the no-arg constructor, which sets meaningful defaults.
- `new Student("Ravi", 20)` → the compiler picks the 2-parameter constructor (overloading by number of parameters).
- `new Student(s3)` → copy constructor copies field values; `s4` is a **separate object**, so changing `s4.name` does not change `s3`.
- `Laptop` has no constructor, so the compiler adds `Laptop() { super(); }`; the field `model` keeps its default `null`.
- `this.name = name;` → the parameter `name` hides the field `name`; `this.name` refers to the field of the current object.

### 1.8 Expected Output

```text
No-arg constructor called
  Unknown, 18, Not assigned
2-arg constructor called
  Ravi, 20, CSE
3-arg constructor called
  Priya, 21, ECE
Copy constructor called
  Priya Copy, 21, ECE
  Priya, 21, ECE
Laptop model (default value): null
```

### 1.9 Interview Answer — "What is a constructor?"

> "A constructor is a special member of a class that is automatically called when we create an object using `new`. Its job is to initialise the object. It has the same name as the class and no return type, not even void. If we don't write any constructor, the compiler adds a default no-argument constructor that simply calls the parent constructor. But if we write even one constructor, the default one is not added. Constructors can be overloaded — we can have a no-argument constructor and parameterised constructors with different parameters — but they are not inherited and cannot be overridden. A constructor can call another constructor of the same class using `this()`, or the parent class constructor using `super()`, and that call must be the first statement."

---

## 2. Constructor Chaining with `this()`

### 2.1 Interview Definition

> **Constructor chaining is the process of calling one constructor from another constructor. Within the same class, it is done using `this(arguments)`; to call a parent class constructor, it is done using `super(arguments)`. The `this()` or `super()` call must be the first statement in the constructor, and a constructor cannot contain both. Constructor chaining avoids duplicate initialisation code by keeping the main logic in one constructor.**

### 2.2 Java Example

```java
public class ChainingDemo {

    static class Account {
        String holder;
        double balance;
        String type;

        Account() {
            this("Guest");                       // calls Account(String)
            System.out.println("Account() finished");
        }

        Account(String holder) {
            this(holder, 0.0);                   // calls Account(String, double)
            System.out.println("Account(String) finished");
        }

        Account(String holder, double balance) {
            this(holder, balance, "SAVINGS");    // calls the main constructor
            System.out.println("Account(String, double) finished");
        }

        Account(String holder, double balance, String type) {   // main constructor
            if (balance < 0) {
                throw new IllegalArgumentException("Balance cannot be negative");
            }
            this.holder = holder;
            this.balance = balance;
            this.type = type;
            System.out.println("Main constructor finished");
        }
    }

    public static void main(String[] args) {
        Account a = new Account();
        System.out.println(a.holder + " | " + a.balance + " | " + a.type);
    }
}
```

**Expected Output:**

```text
Main constructor finished
Account(String, double) finished
Account(String) finished
Account() finished
Guest | 0.0 | SAVINGS
```

**Explanation:** Each constructor immediately calls the next one with `this(...)`, so the calls go **down** the chain first. The **main constructor** finishes first, and then control returns **up** the chain, printing messages in reverse order. All validation and assignment logic lives in **one place** (the main constructor), which avoids duplicate code.

**Rules:**
- `this(...)` must be the **first statement**.
- You **cannot** use both `this(...)` and `super(...)` in the same constructor (both want to be first). The constructor at the end of the `this()` chain calls `super()`.
- **Recursive constructor invocation** (`A() { this(); }` or A calls B and B calls A) is a **compile error**.

---

## 3. The `this` Keyword

### 3.1 Interview Definition

> **`this` is a reference variable that refers to the current object — the object whose method or constructor is being executed. It is mainly used to distinguish instance variables from local variables or parameters with the same name, to call another constructor of the same class using `this()`, to pass the current object as an argument to a method or constructor, and to return the current object from a method (as in method chaining). `this` cannot be used inside a static context.**

### 3.2 Six Uses of `this`

| # | Use | Example |
|---|---|---|
| 1 | Refer to the current object's **field** when hidden by a parameter | `this.name = name;` |
| 2 | Call the current object's **method** (optional, implicit) | `this.display();` |
| 3 | Call another **constructor** of the same class | `this(name, 0);` |
| 4 | **Pass** the current object as an argument | `printer.print(this);` |
| 5 | **Return** the current object (method chaining / builder) | `return this;` |
| 6 | Pass to another constructor / register the object | `new Engine(this);` |

### 3.3 Java Example

```java
public class ThisKeywordDemo {

    static class Pizza {
        private String size;
        private final java.util.List<String> toppings = new java.util.ArrayList<>();

        Pizza(String size) {
            this.size = size;                 // 1. field vs parameter
        }

        Pizza addTopping(String topping) {
            this.toppings.add(topping);
            return this;                      // 5. return current object → chaining
        }

        void order(Kitchen kitchen) {
            kitchen.prepare(this);            // 4. pass current object
        }

        void showSummary() {
            System.out.println(size + " pizza with " + toppings);
        }

        void printIdentity() {
            System.out.println("this == pizza reference? " + (this == ThisKeywordDemo.lastPizza));
        }
    }

    static class Kitchen {
        void prepare(Pizza pizza) {
            System.out.print("Kitchen preparing: ");
            pizza.showSummary();
        }
    }

    static Pizza lastPizza;

    public static void main(String[] args) {
        Pizza p = new Pizza("Large")
                .addTopping("Cheese")
                .addTopping("Corn")
                .addTopping("Olives");          // method chaining using return this
        lastPizza = p;
        p.order(new Kitchen());
        p.printIdentity();
    }
}
```

**Expected Output:**

```text
Kitchen preparing: Large pizza with [Cheese, Corn, Olives]
this == pizza reference? true
```

**Explanation:**
- `this.size = size;` — without `this`, the statement `size = size;` would assign the parameter to itself and the field would stay `null`. This is called **variable shadowing**.
- `return this;` lets us chain calls: `new Pizza(...).addTopping(...).addTopping(...)`. This style is used in `StringBuilder` and the **Builder pattern**.
- `kitchen.prepare(this)` passes the current pizza object to another class.
- `this` inside an instance method is exactly the reference used to call the method (`p`), which the identity check proves.

### 3.4 What happens without `this` (common bug)

```java
public class ShadowingBug {
    static class Employee {
        String name;
        Employee(String name) {
            name = name;          // BUG: assigns parameter to itself
        }
    }

    public static void main(String[] args) {
        Employee e = new Employee("Asha");
        System.out.println("Name: " + e.name);
    }
}
```

**Expected Output:**

```text
Name: null
```

---

## 4. The `super` Keyword

### 4.1 Interview Definition

> **`super` is a reference variable used inside a subclass to refer to its immediate parent class. It is used to call the parent class constructor using `super(arguments)`, which must be the first statement of the subclass constructor; to call a parent class method that has been overridden in the subclass, using `super.methodName()`; and to access a parent class field that is hidden by a field with the same name in the subclass, using `super.fieldName`. If a subclass constructor does not explicitly call `super()` or `this()`, the compiler automatically inserts a call to the parent's no-argument constructor.**

### 4.2 Three Uses of `super`

| Use | Example | Purpose |
|---|---|---|
| `super(...)` | `super(name, salary);` | Call parent constructor to initialise inherited fields |
| `super.method()` | `super.display();` | Reuse the parent's version of an overridden method |
| `super.field` | `super.name` | Access a parent field hidden by a child field with the same name |

### 4.3 Java Example

```java
public class SuperKeywordDemo {

    static class Employee {
        String name;
        double salary;
        String type = "Employee";

        Employee(String name, double salary) {
            this.name = name;
            this.salary = salary;
            System.out.println("Employee constructor: " + name);
        }

        double calculateBonus() {
            return salary * 0.10;
        }

        void display() {
            System.out.println("Name: " + name + ", Salary: " + salary);
        }
    }

    static class Manager extends Employee {
        double teamBonus;
        String type = "Manager";              // hides parent's 'type' field

        Manager(String name, double salary, double teamBonus) {
            super(name, salary);              // 1. call parent constructor (must be first)
            this.teamBonus = teamBonus;
            System.out.println("Manager constructor: " + name);
        }

        @Override
        double calculateBonus() {
            return super.calculateBonus() + teamBonus;   // 2. reuse parent logic
        }

        @Override
        void display() {
            super.display();                  // 2. call parent version first
            System.out.println("Team bonus: " + teamBonus);
        }

        void showTypes() {
            System.out.println("type = " + type + ", super.type = " + super.type);   // 3. hidden field
        }
    }

    public static void main(String[] args) {
        Manager m = new Manager("Kiran", 100000, 5000);
        m.display();
        System.out.println("Total bonus: " + m.calculateBonus());
        m.showTypes();
    }
}
```

**Expected Output:**

```text
Employee constructor: Kiran
Manager constructor: Kiran
Name: Kiran, Salary: 100000.0
Team bonus: 5000.0
Total bonus: 15000.0
type = Manager, super.type = Employee
```

**Explanation:**
- `super(name, salary)` runs the `Employee` constructor **first** to initialise the inherited fields; then the rest of the `Manager` constructor runs.
- `super.calculateBonus()` returns 10% of 100000 = 10000; adding 5000 gives 15000. This avoids copying the parent's formula.
- `super.display()` prints the common details, then the child adds its own line.
- Fields are **not** polymorphic: `type` in `Manager` **hides** `Employee.type`. `super.type` accesses the parent's field. (Field hiding is confusing and should be avoided in real code.)

### 4.4 The Implicit `super()` and a Common Compile Error

<!-- skip-check -->
```java
class Parent {
    Parent(String msg) { }          // only a parameterised constructor
}

class Child extends Parent {
    Child() {                       // COMPILE ERROR:
        // compiler inserts super(); but Parent() does not exist
    }
}
```

**Error:** `constructor Parent in class Parent cannot be applied to given types; required: String, found: no arguments`.
**Fix:** call `super("something")` explicitly, or add a no-argument constructor to `Parent`.

---

## 5. Order of Execution When Creating an Object

```java
public class InitOrderDemo {

    static class Parent {
        static { System.out.println("1. Parent static block"); }
        { System.out.println("3. Parent instance block"); }
        Parent() { System.out.println("4. Parent constructor"); }
    }

    static class Child extends Parent {
        static { System.out.println("2. Child static block"); }
        { System.out.println("5. Child instance block"); }
        Child() {
            super();                      // implicit even if not written
            System.out.println("6. Child constructor");
        }
    }

    public static void main(String[] args) {
        System.out.println("--- First object ---");
        new Child();
        System.out.println("--- Second object ---");
        new Child();
    }
}
```

**Expected Output:**

```text
--- First object ---
1. Parent static block
2. Child static block
3. Parent instance block
4. Parent constructor
5. Child instance block
6. Child constructor
--- Second object ---
3. Parent instance block
4. Parent constructor
5. Child instance block
6. Child constructor
```

**Rules to remember:**
1. **Static blocks** run **once**, when the class is first used — **parent first**, then child.
2. For **every object**: parent's instance initialisers + parent constructor body, then child's instance initialisers + child constructor body.
3. Instance initializer blocks run **before** the constructor body of the same class (the compiler places them right after the `super()` call).

---

## 6. Private Constructors

A `private` constructor prevents other classes from creating objects with `new`. Uses:
1. **Singleton pattern** — only one instance allowed.
2. **Utility classes** — only static methods (`Math`, `Collections`, `Arrays`).
3. **Factory methods** — control object creation (`LocalDate.of(...)`).

```java
public class SingletonDemo {

    static class AppConfig {
        private static final AppConfig INSTANCE = new AppConfig();   // created once
        private int requestCount;

        private AppConfig() {                 // no one outside can call new AppConfig()
            System.out.println("AppConfig created");
        }

        static AppConfig getInstance() {
            return INSTANCE;
        }

        void recordRequest() {
            requestCount++;
        }

        int getRequestCount() {
            return requestCount;
        }
    }

    public static void main(String[] args) {
        AppConfig c1 = AppConfig.getInstance();
        AppConfig c2 = AppConfig.getInstance();
        c1.recordRequest();
        c2.recordRequest();
        System.out.println("Same instance? " + (c1 == c2));
        System.out.println("Requests: " + c1.getRequestCount());
    }
}
```

**Expected Output:**

```text
AppConfig created
Same instance? true
Requests: 2
```

(Note: in this single-file example, `main` is in the outer class, which *can* access the nested class's private constructor. From a different top-level class, `new AppConfig()` would be a compile error.)

---

## 7. Interview Questions with Complete Answers

### Basic Level

**Q1. Does a constructor return a value?**

A constructor has **no return type** in its declaration and does not use `return value;`. Technically, the `new` expression returns the reference of the newly created object, but that is done by `new`, not by the constructor. A constructor can use an empty `return;` to exit early.

**Q2. What happens if we write a return type for a constructor?**

It is no longer a constructor — it becomes an ordinary method that happens to have the same name as the class. The code compiles (with a style warning in some IDEs), but that method is not called automatically when the object is created.

**Q3. When does the compiler provide a default constructor?**

Only when the class has **no constructor at all**. The default constructor has no parameters, has the same access modifier as the class, and only calls `super()`. If you write any constructor, no default constructor is added.

### Intermediate Level

**Q4. Can a constructor be private? Why would you do that?**

Yes. A private constructor prevents creating objects from outside the class. It is used for the **Singleton pattern**, for **utility classes** containing only static methods (to stop anyone instantiating them), and for classes that create objects through **static factory methods**.

**Q5. Can constructors be inherited or overridden?**

No. Constructors are not members that a subclass inherits, because a constructor's name must match its own class name. Since they are not inherited, they cannot be overridden. A subclass can only **call** the parent's constructor using `super(...)`.

**Q6. Why must `this()` or `super()` be the first statement in a constructor?**

To guarantee that the object is initialised **from the top of the hierarchy downwards**. The parent part of the object must be fully initialised before the child constructor uses it. If code could run before `super()`, it might access parent fields that are not yet initialised. Because both `this()` and `super()` must be first, a constructor cannot have both.

> (Java 22+ previews "statements before super()" — JEP 447/482 — allowing simple statements that do not use `this` before the `super(...)` call. For interviews, state the classic rule.)

**Q7. What is the difference between `this` and `super`?**

| `this` | `super` |
|---|---|
| Refers to the **current object** | Refers to the **parent class part** of the current object |
| `this()` calls another constructor of the **same class** | `super()` calls a constructor of the **parent class** |
| `this.field` accesses the current class's field | `super.field` accesses the parent's (hidden) field |
| `this.method()` calls the current (possibly overridden) method | `super.method()` calls the parent's version |
| Can be passed as an argument or returned | Cannot be used alone as a value (`return super;` is invalid) |

Both cannot be used in a static context.

### Advanced Level

**Q8. Can we call a constructor explicitly like a method?**

No. You cannot write `obj.Student()`. A constructor is invoked only through `new`, or from another constructor using `this(...)` / `super(...)`.

**Q9. Can an abstract class have a constructor? Can an interface?**

An **abstract class can** have constructors. Even though you cannot create an object of an abstract class directly, its constructor runs when a concrete subclass object is created (through `super()`), to initialise the abstract class's fields. An **interface cannot** have constructors because it cannot have instance fields to initialise.

**Q10. What happens if a constructor throws an exception?**

The object creation fails; `new` does not return a reference, so the variable is not assigned, and the partially created object becomes eligible for garbage collection. Constructors can declare checked exceptions with `throws`, and the caller must handle them.

**Q11. What is the output?**

```java
public class ConstructorTrick {
    static class A {
        A() {
            System.out.println("A()");
            show();                       // calls the overridden version!
        }
        void show() { System.out.println("A.show"); }
    }

    static class B extends A {
        int value = 10;
        B() {
            System.out.println("B(), value = " + value);
        }
        @Override
        void show() { System.out.println("B.show, value = " + value); }
    }

    public static void main(String[] args) {
        new B();
    }
}
```

**Expected Output:**

```text
A()
B.show, value = 0
B(), value = 10
```

**Explanation:** `new B()` first runs `A()`. Inside `A()`, `show()` is called — because of **dynamic dispatch**, the overridden `B.show()` runs. But `B`'s field initialiser (`value = 10`) has **not run yet** (it runs after `super()` returns), so `value` is still its default **0**. Lesson: **never call overridable methods from a constructor.**

---

## 8. Follow-Up Questions

**F1. What is the access modifier of the default constructor?**

The same as the class: a `public` class gets a `public` default constructor; a package-private class gets a package-private one.

**F2. Is a copy constructor built into Java like in C++?**

No. Java does not generate copy constructors automatically. You write one yourself if needed. Java's alternative is `clone()`, but copy constructors are generally preferred because they are simpler and type-safe.

**F3. Can a constructor be `final` or `static`?**

No. `final` is meaningless because constructors cannot be overridden anyway. `static` is impossible because a constructor works on a specific new object.

**F4. How many times is a parent constructor called when a grandchild object is created?**

Exactly once per level: for `class A`, `class B extends A`, `class C extends B`, creating `new C()` calls `A()` → `B()` → `C()` constructor bodies, each once, in that order (top-down).

---

## 9. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Writing `void` before a constructor | It becomes a method. |
| Expecting `new X()` to work after adding a parameterised constructor | Default constructor is no longer added. |
| `name = name;` in constructor | Use `this.name = name;`. |
| Code before `super()`/`this()` | Compile error (classic rule). |
| Using `this` in a static method | Compile error. |
| Calling overridable methods in constructors | Subclass fields may not be initialised yet. |
| Thinking `super()` is optional in behaviour | It is always called — implicitly if not written. |

---

## 10. Quick Revision

- Constructor: same name as class, **no return type**, called by `new`, initialises object.
- Default constructor added **only if no constructor exists**.
- Types: default, no-arg, parameterised, copy (manual), private.
- Overloaded ✅; inherited ❌; overridden ❌; `static/final/abstract` ❌.
- First statement: `this(...)` or `super(...)` (implicit `super()` if none); never both.
- `this` = current object: fields, methods, `this()`, pass/return current object.
- `super` = parent: `super()`, `super.method()`, `super.field`.
- Order: parent static → child static (once); per object: parent init+constructor → child init+constructor.
- Private constructor → singleton, utility classes, factories.
- Don't call overridable methods in constructors.

---

[← Previous: Classes and Objects](01-OOP-Introduction-Classes-and-Objects.md) | [Back to README](../README.md) | [Next: Encapsulation →](03-Encapsulation.md)
