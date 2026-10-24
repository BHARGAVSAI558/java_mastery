# Chapter 3.5 — Polymorphism: Compile-Time and Runtime, Overloading vs Overriding

[← Previous: Inheritance](04-Inheritance-and-Types.md) | [Back to README](../README.md) | [Next: Abstraction and Abstract Classes →](06-Abstraction-and-Abstract-Classes.md)

---

## What You Will Learn

- What polymorphism means and why it is powerful
- Compile-time (static) polymorphism — method overloading
- Runtime (dynamic) polymorphism — method overriding
- **All rules of method overriding** (access, return type, exceptions, static/final/private)
- The `@Override` annotation and covariant return types
- Static vs dynamic binding
- Method hiding (static methods) and why fields are not polymorphic
- The complete **overloading vs overriding** comparison

---

## 1. Polymorphism

### 1.1 Interview Definition

> **Polymorphism means "many forms". It is the ability of a single method name, operator or reference to behave differently depending on the context. In Java, polymorphism is of two types: compile-time (static) polymorphism, achieved through method overloading, where the compiler selects the method to call based on the method signature; and runtime (dynamic) polymorphism, achieved through method overriding, where a parent-class reference points to a child-class object and the JVM decides at runtime which overridden method to execute based on the actual object type. Polymorphism lets us write general code that works with many types and makes programs flexible and extensible.**

### 1.2 Understand the Concept

The word comes from Greek: *poly* (many) + *morph* (form).

| | Compile-time polymorphism | Runtime polymorphism |
|---|---|---|
| Also called | Static polymorphism, early binding, static binding | Dynamic polymorphism, late binding, dynamic method dispatch |
| Achieved by | **Method overloading** (and constructor overloading) | **Method overriding** |
| Decided by | **Compiler**, using the reference type and arguments | **JVM at runtime**, using the actual object type |
| Needs inheritance? | No | Yes (or interface implementation) |
| Speed | Slightly faster (resolved early) | Small runtime cost (virtual method lookup, usually optimised by JIT) |
| Example | `print(int)`, `print(String)` | `Animal a = new Dog(); a.sound();` → Dog's sound |

**Why is runtime polymorphism so useful?**
You can write code against a **general type** (`Shape`, `Payment`, `Animal`) and it automatically works with **any current or future subtype** — without `if-else` chains on types. Adding a new type means adding a new class, not changing existing code (Open/Closed Principle).

### 1.3 Real-World Example

- **A person** behaves differently in different roles: at home as a son, in college as a student, at a shop as a customer — same person, many forms.
- **A "Pay" button** in an app: whether you choose card, UPI or wallet, you press the same button (`pay()`), and the right payment process runs based on the actual payment object.
- **A TV remote's power button**: the same button works on a Sony TV or an LG TV, but each TV implements "power on" in its own way.

---

## 2. Compile-Time Polymorphism — Method Overloading

Overloading was taught in detail in [Chapter 1.8](../01-Java-Fundamentals/08-Methods-Overloading-Recursion.md). Summary:

- Same method name, **different parameter lists** (number, type or order).
- Return type alone is not enough.
- Resolved by the **compiler** → static binding.

```java
public class OverloadingPolymorphism {
    static class Printer {
        void print(int value)     { System.out.println("Printing int: " + value); }
        void print(double value)  { System.out.println("Printing double: " + value); }
        void print(String value)  { System.out.println("Printing String: " + value); }
        void print(String value, int copies) {
            for (int i = 1; i <= copies; i++) {
                System.out.println("Copy " + i + ": " + value);
            }
        }
    }

    public static void main(String[] args) {
        Printer p = new Printer();
        p.print(10);
        p.print(3.5);
        p.print("Report");
        p.print("Invoice", 2);
    }
}
```

**Expected Output:**

```text
Printing int: 10
Printing double: 3.5
Printing String: Report
Copy 1: Invoice
Copy 2: Invoice
```

The same name `print` takes **many forms**, and the compiler chooses the right one from the arguments.

---

## 3. Runtime Polymorphism — Method Overriding

### 3.1 Interview Definition — Method Overriding

> **Method overriding occurs when a subclass provides its own implementation of a method that is already defined in its superclass, using the same method name, the same parameter list and a compatible return type. The overriding method replaces the parent's behaviour for objects of the subclass. When the method is called through a parent-class reference, the JVM uses the actual object's type at runtime to decide which version to execute — this is called dynamic method dispatch or runtime polymorphism. Static, final and private methods cannot be overridden.**

### 3.2 Java Example

```java
import java.util.List;

public class OverridingDemo {

    static class Notification {
        void send(String message) {
            System.out.println("Generic notification: " + message);
        }
    }

    static class EmailNotification extends Notification {
        @Override
        void send(String message) {
            System.out.println("Email sent: " + message);
        }
    }

    static class SmsNotification extends Notification {
        @Override
        void send(String message) {
            System.out.println("SMS sent: " + message.substring(0, Math.min(10, message.length())) + "...");
        }
    }

    static class PushNotification extends Notification {
        // does not override → inherits the parent's send()
    }

    // This method works for ANY Notification type, present or future
    static void notifyUser(Notification n, String msg) {
        n.send(msg);
    }

    public static void main(String[] args) {
        Notification n1 = new EmailNotification();     // parent reference, child object
        Notification n2 = new SmsNotification();
        Notification n3 = new PushNotification();

        n1.send("Your order has shipped");
        n2.send("Your OTP is 482913");
        n3.send("Flash sale today");

        System.out.println("--- Same loop, different behaviour ---");
        List<Notification> channels = List.of(new EmailNotification(), new SmsNotification());
        for (Notification channel : channels) {
            notifyUser(channel, "Payment received");
        }
    }
}
```

### 3.3 Code Explanation

- `Notification n1 = new EmailNotification();` — the **reference type** is `Notification`, the **object type** is `EmailNotification`. This is **upcasting** (automatic).
- `n1.send(...)` — the compiler checks that `Notification` has a `send(String)` method (compile-time check). At runtime, the JVM sees the object is an `EmailNotification` and calls **its** `send()`.
- `PushNotification` does not override `send()`, so the inherited parent version runs.
- `notifyUser()` is written once for the parent type, but behaves differently for each object — that is the real power of polymorphism. If we add a `WhatsAppNotification` class tomorrow, `notifyUser()` needs **no change**.

### 3.4 Expected Output

```text
Email sent: Your order has shipped
SMS sent: Your OTP i...
Generic notification: Flash sale today
--- Same loop, different behaviour ---
Email sent: Payment received
SMS sent: Payment re...
```

### 3.5 Interview Answer — "What is runtime polymorphism?"

> "Runtime polymorphism, also called dynamic method dispatch, is achieved through **method overriding**. A subclass provides its own implementation of a method that already exists in the parent class, with the same signature. When we call that method using a **parent class reference that points to a child class object**, the JVM decides at runtime which version to run, based on the **actual object**, not the reference type. For example, if `Notification n = new EmailNotification()`, then `n.send()` calls the email version. This lets us write general code, like a method that accepts any `Notification`, and it will automatically work with every subclass, even ones added later. It is called runtime polymorphism because the decision is made during execution, not compilation."

---

## 4. Rules of Method Overriding (Very Important)

| # | Rule | Example / Explanation |
|---|---|---|
| 1 | **Same name and same parameter list** (same signature) | Different parameters = overloading, not overriding |
| 2 | **Return type** must be the same, or a **subtype** (covariant return type, Java 5+) for objects; for primitives it must be exactly the same | Parent returns `Animal`, child can return `Dog` |
| 3 | **Access modifier cannot be more restrictive** | `public` → must stay `public`; `protected` → `protected` or `public`; default → default, `protected` or `public` |
| 4 | **Checked exceptions:** the overriding method can throw the **same, fewer, narrower (subclass)** checked exceptions, or none — but **not new or broader** checked exceptions | Parent `throws IOException` → child can throw `FileNotFoundException` or nothing, not `Exception` |
| 5 | **Unchecked exceptions** can be thrown freely | `RuntimeException` and subclasses are not restricted |
| 6 | **`private` methods cannot be overridden** | They are not visible to the child; a same-named method in the child is a new method |
| 7 | **`static` methods cannot be overridden** | A same-signature static method in the child **hides** the parent's method |
| 8 | **`final` methods cannot be overridden** | Compile error |
| 9 | **Constructors cannot be overridden** | They are not inherited |
| 10 | Overriding requires **inheritance** (or implementing an interface) | |
| 11 | Use `@Override` | Compiler checks that you really override something |
| 12 | Can call the parent version using `super.method()` | |

**Why the access rule?** Because of substitution: code that uses a parent reference expects to be able to call the method. If a child made it `private`, a call like `parentRef.method()` would break when the object is the child.

**Why the exception rule?** Code that calls `parentRef.method()` only handles the exceptions declared by the parent. If the child could throw a broader checked exception, that code would face an exception it never prepared for.

### 4.1 Java Example — Overriding Rules in Action

```java
import java.io.FileNotFoundException;
import java.io.IOException;

public class OverridingRulesDemo {

    static class Animal {
        protected Animal reproduce() {                       // returns Animal
            return new Animal();
        }

        void readData() throws IOException {
            System.out.println("Animal reads data");
        }

        static void info() {
            System.out.println("Animal static info");
        }

        private void secret() {
            System.out.println("Animal secret");
        }

        void callSecret() {
            secret();                                        // always calls Animal's private method
        }

        public String toString() { return "Animal"; }
    }

    static class Dog extends Animal {
        @Override
        public Dog reproduce() {                             // covariant return + wider access: OK
            return new Dog();
        }

        @Override
        void readData() throws FileNotFoundException {       // narrower checked exception: OK
            System.out.println("Dog reads data");
        }

        static void info() {                                 // HIDING, not overriding
            System.out.println("Dog static info");
        }

        private void secret() {                              // a NEW method, not an override
            System.out.println("Dog secret");
        }

        public String toString() { return "Dog"; }
    }

    public static void main(String[] args) throws IOException {
        Animal a = new Dog();

        System.out.println("reproduce() returns: " + a.reproduce());   // dynamic dispatch
        a.readData();                                                   // dynamic dispatch

        a.info();                // static: decided by REFERENCE type (Animal) → hiding
        Dog.info();
        Animal.info();

        a.callSecret();          // private methods are not overridden
    }
}
```

### 4.2 Code Explanation

- `public Dog reproduce()` overrides `protected Animal reproduce()`: the return type `Dog` is a subtype of `Animal` (**covariant return**), and `public` is **less** restrictive than `protected` — both allowed.
- `readData() throws FileNotFoundException` is allowed because `FileNotFoundException` is a **subclass** of `IOException`. Writing `throws Exception` would be a compile error.
- `a.info()` — static methods are bound at **compile time** using the reference type (`Animal`), so `Animal.info()` runs even though the object is a `Dog`. (Calling static methods through an instance is legal but misleading — IDEs warn about it.)
- `a.callSecret()` — inside `Animal`, `secret()` refers to `Animal`'s private method; `Dog.secret()` is unrelated, so "Animal secret" prints.

### 4.3 Expected Output

```text
reproduce() returns: Dog
Dog reads data
Animal static info
Dog static info
Animal static info
Animal secret
```

### 4.4 Compile Errors to Recognise

<!-- skip-check -->
```java
class Parent {
    public void show() { }
    void process() throws java.io.IOException { }
    final void lock() { }
    int count() { return 1; }
}

class Child extends Parent {
    void show() { }                         // ERROR: attempting to assign weaker access privileges
    void process() throws Exception { }     // ERROR: overridden method does not throw Exception
    void lock() { }                         // ERROR: overridden method is final
    long count() { return 1; }              // ERROR: return type long is not compatible with int
}
```

---

## 5. The `@Override` Annotation

> **`@Override` is an annotation that tells the compiler that a method is intended to override a method in a superclass or implement an interface method. If the method does not actually override anything — because of a typo in the name, a different parameter type or a missing parent method — the compiler reports an error. It does not change behaviour at runtime, but it prevents subtle bugs.**

```java
public class OverrideAnnotationDemo {
    static class Product {
        String name;
        Product(String name) { this.name = name; }

        // Mistake: parameter type is Product, not Object → this OVERLOADS equals, it does not override it
        public boolean equals(Product other) {
            return other != null && name.equals(other.name);
        }
    }

    public static void main(String[] args) {
        Object p1 = new Product("Pen");
        Object p2 = new Product("Pen");
        System.out.println("equals via Object reference: " + p1.equals(p2));
    }
}
```

**Expected Output:**

```text
equals via Object reference: false
```

**Explanation:** `equals(Product)` does not override `Object.equals(Object)`, it overloads it. When called through an `Object` reference, the compiler picks `Object.equals(Object)`, which compares references → `false`. If the method had `@Override`, the compiler would immediately report: *"method does not override or implement a method from a supertype"*. **Always use `@Override`.**

---

## 6. Static Binding vs Dynamic Binding

### 6.1 Interview Definition

> **Binding is the process of connecting a method call to the method body that will be executed. Static (early) binding happens at compile time, and it is used for static, private and final methods, overloaded methods, and field access, because the compiler can determine exactly which code will run. Dynamic (late) binding happens at runtime, and it is used for overridden instance methods: the JVM looks at the actual object type to select the method implementation.**

| | Static binding | Dynamic binding |
|---|---|---|
| When | Compile time | Runtime |
| Based on | Reference type | Actual object type |
| Used for | `static`, `private`, `final` methods, overloading, fields, constructors | Overridden instance methods |
| Bytecode | `invokestatic`, `invokespecial` | `invokevirtual`, `invokeinterface` |

### 6.2 Fields Are NOT Polymorphic

```java
public class FieldsNotPolymorphic {
    static class Parent {
        String name = "Parent field";
        String getName() { return name; }
    }

    static class Child extends Parent {
        String name = "Child field";               // hides Parent.name
        @Override
        String getName() { return name; }
    }

    public static void main(String[] args) {
        Parent p = new Child();
        System.out.println("p.name      = " + p.name);        // field: reference type decides
        System.out.println("p.getName() = " + p.getName());   // method: object type decides
        System.out.println("((Child) p).name = " + ((Child) p).name);
    }
}
```

**Expected Output:**

```text
p.name      = Parent field
p.getName() = Child field
((Child) p).name = Child field
```

Fields are resolved by the **reference type** (static binding); methods are resolved by the **object type** (dynamic binding). Keep fields `private` and access them through methods to avoid confusion.

---

## 7. Overloading vs Overriding — Complete Comparison

| Point | Method overloading | Method overriding |
|---|---|---|
| Definition | Same method name, **different parameters** | Same method name, **same parameters** in parent and child |
| Polymorphism type | Compile-time (static) | Runtime (dynamic) |
| Binding | Static (early) | Dynamic (late) |
| Where | Same class (or parent + child) | Parent and child classes (or interface + class) |
| Inheritance required | No | Yes |
| Parameters | Must differ | Must be the same |
| Return type | Can be anything | Same or covariant subtype |
| Access modifier | Any | Same or less restrictive |
| Checked exceptions | Any | Same, narrower or none |
| `static` methods | Can be overloaded | Cannot be overridden (hidden) |
| `private` methods | Can be overloaded | Cannot be overridden |
| `final` methods | Can be overloaded | Cannot be overridden |
| `@Override` | Not applicable | Recommended |
| Purpose | Same action, different inputs (readability) | Change/extend inherited behaviour |
| Example | `add(int, int)` and `add(double, double)` | `Animal.sound()` and `Dog.sound()` |

### 7.1 Interview Answer — "Difference between overloading and overriding"

> "Overloading means having multiple methods with the **same name but different parameter lists**, usually in the same class. It is resolved by the compiler, so it is compile-time polymorphism. The return type and access modifier can be anything, and static, private and final methods can be overloaded. Overriding means a **subclass redefines a parent method with the same signature**. It is resolved at runtime based on the actual object, so it is runtime polymorphism. Overriding has strict rules: the return type must be the same or a subtype, the access cannot be more restrictive, it cannot throw broader checked exceptions, and static, private and final methods cannot be overridden. For example, `print(int)` and `print(String)` are overloaded, while `Dog.sound()` overriding `Animal.sound()` is overriding."

---

## 8. Interview Questions with Complete Answers

### Basic Level

**Q1. Can we override a static method?**

No. Static methods belong to the class and are bound at compile time based on the reference type. If a subclass declares a static method with the same signature, it **hides** the parent's method (method hiding). Calling it through a parent reference still runs the parent version. Also, you cannot override a static method with an instance method or vice versa — that is a compile error.

**Q2. Can we override a private method?**

No. Private methods are not visible to subclasses, so they cannot be overridden. A method with the same signature in the subclass is a completely new, unrelated method. Calls inside the parent class to its private method always run the parent's version.

**Q3. Can we override the `main` method?**

No, because `main` is static. A subclass can declare its own `main` (hiding), and you can run either class separately.

### Intermediate Level

**Q4. What is a covariant return type?**

From Java 5, an overriding method can return a **subtype** of the parent method's return type. Example: `Animal.reproduce()` returns `Animal`, and `Dog.reproduce()` returns `Dog`. This avoids casting at the call site when you have a `Dog` reference. It applies only to reference types, not primitives (`int` cannot become `long`).

**Q5. Can an overriding method throw an exception if the parent method does not?**

It can throw **unchecked** exceptions (`RuntimeException` and its subclasses) freely. It **cannot** throw any **checked** exception that the parent method does not declare.

**Q6. Can we change the access modifier while overriding?**

Only to a **less restrictive** (wider) one. `protected` can become `public`. `public` cannot become `protected`, default or `private`. Interface methods are implicitly `public`, so implementing methods must be `public`.

**Q7. Is runtime polymorphism possible with data members?**

No. Runtime polymorphism applies only to **overridden instance methods**. Fields are resolved by the reference type at compile time. If parent and child have a field with the same name, the parent reference sees the parent's field.

### Advanced Level

**Q8. How does the JVM implement dynamic method dispatch?**

Each class has a **method table** (often called a *vtable*) containing pointers to the implementations of its instance methods. A subclass's table starts as a copy of its parent's table, with entries replaced for overridden methods. When `invokevirtual` executes, the JVM finds the object's actual class, looks up the method in that class's table, and calls the implementation found there. The JIT often optimises this with **inline caching** and devirtualisation when only one implementation is used.

**Q9. Which method is called?**

```java
public class OverloadOverrideMix {
    static class Animal {
        void eat(Object food) { System.out.println("Animal eats Object"); }
        void eat(String food) { System.out.println("Animal eats String"); }
    }

    static class Cow extends Animal {
        @Override
        void eat(Object food) { System.out.println("Cow eats Object"); }
        @Override
        void eat(String food) { System.out.println("Cow eats String"); }
    }

    public static void main(String[] args) {
        Animal a = new Cow();
        Object grass = "grass";
        a.eat(grass);              // overload chosen at compile time by declared type (Object)
        a.eat("hay");              // overload: String
    }
}
```

**Expected Output:**

```text
Cow eats Object
Cow eats String
```

**Explanation:** This combines both kinds of polymorphism. **Step 1 (compile time):** the compiler selects the **overload** using the **declared types** of the arguments — `grass` is declared as `Object`, so `eat(Object)` is chosen even though the actual value is a `String`. **Step 2 (runtime):** the JVM selects the **override** based on the actual object (`Cow`). So overloading uses static types; overriding uses the runtime object type.

**Q10. Can a parent reference call a method that exists only in the child?**

No. The compiler checks method calls against the **reference type**. `Animal a = new Dog(); a.bark();` fails to compile if `Animal` has no `bark()`. You must downcast first: `((Dog) a).bark();` — see [Chapter 3.12](12-Upcasting-Downcasting-Dynamic-Dispatch.md).

---

## 9. Follow-Up Questions

**F1. Is overloading "real" polymorphism?**

Some authors say only overriding is true polymorphism (subtype polymorphism). But in Java interviews, overloading is accepted as **compile-time (ad hoc) polymorphism**. Mention both and say runtime polymorphism is the more powerful one.

**F2. What is the benefit of `@Override` if the code works without it?**

It turns silent bugs into compile errors: misspelled names (`tostring()`), wrong parameter types (`equals(Product)`), or a parent method that was later renamed or removed.

**F3. Can we override a method and make it `final`?**

Yes. A child can override a method and declare it `final`, so that grandchildren cannot override it further.

**F4. Can we override `synchronized`/`strictfp`/`native` methods?**

Yes. These modifiers are not part of the method signature. An overriding method can add or remove `synchronized`.

---

## 10. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Thinking static methods can be overridden | They are hidden; binding uses reference type. |
| Overloading `equals(MyClass)` instead of overriding `equals(Object)` | Always `equals(Object)` with `@Override`. |
| Narrowing access in the child | Compile error. |
| Declaring broader checked exceptions | Compile error. |
| Expecting fields to behave polymorphically | Fields use the reference type. |
| Calling a child-only method through parent reference | Compile error; downcast after `instanceof`. |
| Saying "overloading is runtime polymorphism" | Overloading = compile time; overriding = runtime. |

---

## 11. Quick Revision

- Polymorphism = "many forms": **compile-time (overloading)** and **runtime (overriding)**.
- Runtime polymorphism: `Parent ref = new Child(); ref.method()` → child's version (object type decides).
- Overriding rules: same signature; same/covariant return; access same or wider; checked exceptions same/narrower/none; not `static`/`private`/`final`/constructors; use `@Override`.
- Static binding: static/private/final methods, overloads, fields. Dynamic binding: overridden instance methods.
- Overload selection uses **declared** argument types; override selection uses the **runtime** object.
- Fields are not polymorphic.

---

[← Previous: Inheritance](04-Inheritance-and-Types.md) | [Back to README](../README.md) | [Next: Abstraction and Abstract Classes →](06-Abstraction-and-Abstract-Classes.md)
