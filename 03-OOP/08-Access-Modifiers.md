# Chapter 3.8 — Access Modifiers

[← Previous: Interfaces](07-Interfaces-and-Multiple-Inheritance.md) | [Back to README](../README.md) | [Next: static and final →](09-static-and-final.md)

---

## What You Will Learn

- The four access levels: `private`, default (package-private), `protected`, `public`
- What each modifier can be applied to (classes, methods, fields, constructors)
- The tricky rules of `protected` across packages
- Access modifiers and overriding
- Access modifiers vs non-access modifiers
- How to choose the right access level

---

## 1. Access Modifiers

### 1.1 Interview Definition

> **Access modifiers in Java are keywords that control the visibility, or accessibility, of classes, constructors, methods and fields from other classes. Java has four access levels: `private`, which allows access only within the same class; default (package-private, when no modifier is written), which allows access only within the same package; `protected`, which allows access within the same package and in subclasses in other packages through inheritance; and `public`, which allows access from everywhere. Access modifiers are the main tool for implementing encapsulation and data hiding.**

### 1.2 Understand the Concept

**Visibility table (most restrictive → least restrictive):**

| Modifier | Same class | Same package (other class) | Subclass in another package | Any class in another package |
|---|---|---|---|---|
| `private` | ✅ | ❌ | ❌ | ❌ |
| default (no keyword) | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ (through inheritance only) | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

**Memory tip:** **P**rivate < **D**efault < **P**rotected < **P**ublic → "**P**lease **D**on't **P**ass **P**apers" (visibility grows left to right).

**Where can each modifier be used?**

| Applied to | `private` | default | `protected` | `public` |
|---|---|---|---|---|
| Top-level class / interface | ❌ | ✅ | ❌ | ✅ |
| Nested (inner) class | ✅ | ✅ | ✅ | ✅ |
| Constructor | ✅ | ✅ | ✅ | ✅ |
| Method | ✅ | ✅ | ✅ | ✅ |
| Field | ✅ | ✅ | ✅ | ✅ |
| Local variable | ❌ (no access modifiers at all; only `final`) | | | |

**Why can't a top-level class be `private` or `protected`?**
- `private` would mean "visible only inside itself" — no other code could ever use it, which is useless.
- `protected` relates to subclasses and packages; for a top-level class, it would behave the same as default. Java allows only `public` or default for top-level types.

### 1.3 Real-World Example

Think of a **house**:
- `private` — your **personal diary**: only you (the same class).
- default — the **family room**: everyone living in the house (same package).
- `protected` — **family heirlooms**: family members and your children even if they move to another city (subclasses in other packages).
- `public` — the **front porch**: anyone walking by.

### 1.4 Java Example — Across Packages

This example needs multiple files in different packages, so it is shown as a project structure.

```text
src/
├── com/bank/core/Account.java
├── com/bank/core/Auditor.java          (same package)
└── com/bank/app/SavingsAccount.java    (different package, subclass)
    com/bank/app/MainApp.java           (different package, not a subclass)
```

**`com/bank/core/Account.java`**

<!-- skip-check -->
```java
package com.bank.core;

public class Account {
    private double balance = 1000;          // only Account
    String branchCode = "BR01";             // default: package com.bank.core only
    protected String accountType = "BASIC"; // package + subclasses
    public String holderName = "Ravi";      // everyone

    private void audit()        { System.out.println("private audit"); }
    void internalReport()       { System.out.println("default internalReport"); }
    protected void applyInterest() { balance += balance * 0.04; }
    public double getBalance()  { return balance; }
}
```

**`com/bank/core/Auditor.java`** — same package

<!-- skip-check -->
```java
package com.bank.core;

public class Auditor {
    void check(Account acc) {
        // acc.balance;            ❌ private
        System.out.println(acc.branchCode);    // ✅ default, same package
        System.out.println(acc.accountType);   // ✅ protected, same package
        System.out.println(acc.holderName);    // ✅ public
        acc.internalReport();                  // ✅
        acc.applyInterest();                   // ✅ protected works in same package
    }
}
```

**`com/bank/app/SavingsAccount.java`** — different package, subclass

<!-- skip-check -->
```java
package com.bank.app;

import com.bank.core.Account;

public class SavingsAccount extends Account {
    void upgrade() {
        accountType = "SAVINGS";       // ✅ protected, through inheritance (this.accountType)
        applyInterest();               // ✅ protected, through inheritance
        // branchCode = "X";           ❌ default: different package
        // audit();                    ❌ private

        Account other = new Account();
        // other.accountType = "X";    ❌ protected NOT accessible through a parent-type reference
        SavingsAccount mine = new SavingsAccount();
        mine.accountType = "GOLD";     // ✅ allowed through a SavingsAccount reference
    }
}
```

**`com/bank/app/MainApp.java`** — different package, not a subclass

<!-- skip-check -->
```java
package com.bank.app;

import com.bank.core.Account;

public class MainApp {
    public static void main(String[] args) {
        Account acc = new Account();
        System.out.println(acc.holderName);    // ✅ public
        System.out.println(acc.getBalance());  // ✅ public
        // acc.accountType;                    ❌ protected (not a subclass)
        // acc.branchCode;                     ❌ default (different package)
    }
}
```

### 1.5 Code Explanation

- **private** members are visible only inside `Account`, even to subclasses.
- **default** members are visible to every class in `com.bank.core` but not to `com.bank.app`, **even to a subclass**.
- **protected** members are visible to the whole package **and** to subclasses in other packages — but in another package, the subclass can access them only **through inheritance**: on `this`, or on references of its own type (or its subtypes), **not** on a reference of the parent type (`other.accountType` is not allowed). This rule prevents a subclass from tampering with protected state of unrelated objects.
- **public** members are visible everywhere.

### 1.6 Single-File Runnable Example

```java
public class AccessDemo {

    static class Parent {
        private String secret = "private data";
        String packageInfo = "default data";
        protected String familyInfo = "protected data";
        public String publicInfo = "public data";

        private void privateMethod() { System.out.println("Parent private method"); }

        public void show() {
            privateMethod();                          // private: accessible inside the class
            System.out.println("Inside Parent: " + secret);
        }
    }

    static class Child extends Parent {
        void display() {
            // System.out.println(secret);            // compile error: secret has private access
            System.out.println("Child sees: " + packageInfo + ", " + familyInfo + ", " + publicInfo);
        }
    }

    public static void main(String[] args) {
        Child c = new Child();
        c.show();
        c.display();
        // Note: AccessDemo is the top-level class enclosing Parent, so it COULD read c.secret
        // through a Parent reference. Nested classes share private access with their enclosing class.
        Parent p = c;
        System.out.println("Enclosing class reads private: " + p.secret);
    }
}
```

**Expected Output:**

```text
Parent private method
Inside Parent: private data
Child sees: default data, protected data, public data
Enclosing class reads private: private data
```

**Explanation:** `Child` cannot use `secret` directly (compile error), but the inherited public method `show()` can, because it is code inside `Parent`. The last line shows a special rule: a top-level class and all its nested classes can access each other's private members (they are in the same "top-level unit"). This is why single-file examples can sometimes access private members that another top-level class could not.

### 1.7 Interview Answer — "Explain access modifiers in Java"

> "Access modifiers control where a class, method, constructor or field can be accessed from. Java has four levels. **private** is the most restrictive — accessible only inside the same class; we use it for fields to achieve encapsulation. **Default**, when we don't write any modifier, means package-private — accessible only by classes in the same package. **protected** is accessible in the same package and also in subclasses in other packages, but only through inheritance. **public** is accessible from everywhere. A top-level class can only be public or default. A good practice is to give the most restrictive access possible: fields private, helper methods private, and only the methods that form the class's API public. Access modifiers also matter for overriding — an overriding method cannot reduce visibility."

---

## 2. Access Modifiers and Overriding

When overriding, the child's method must have **the same or wider** access:

| Parent method | Allowed in child |
|---|---|
| `public` | `public` only |
| `protected` | `protected`, `public` |
| default | default, `protected`, `public` |
| `private` | Not inherited → not overriding at all (any modifier, it is a new method) |

**Reason:** **Liskov substitution** — code using a parent reference must be able to call the method on any child object.

---

## 3. Access Modifiers vs Non-Access Modifiers

| Access modifiers (visibility) | Non-access modifiers (behaviour) |
|---|---|
| `public`, `protected`, default, `private` | `static`, `final`, `abstract`, `synchronized`, `volatile`, `transient`, `native`, `strictfp`, `sealed`, `non-sealed`, `default` (interface methods) |
| Decide **who** can access | Decide **how** the member behaves |

A member can have **one** access modifier and **several** non-access modifiers: `public static final int MAX = 10;`.

---

## 4. Choosing the Right Access Level — Best Practices

1. **Fields:** almost always `private`. Expose through methods.
2. **Constants:** `public static final` if part of the API; otherwise `private static final`.
3. **Methods:** `public` only for the class's intended API. Helpers → `private`.
4. **protected:** use only when you design a class for extension and want subclasses to customise specific steps (e.g., Template Method hooks). Remember it also opens access to the whole package.
5. **default:** good for classes and methods that are internal to a package (package-level encapsulation).
6. **Constructors:** `private` for singletons and utility classes; `protected` for abstract base classes.
7. Rule: **"Start with private; widen only when needed."** It is easy to make something more visible later, but hard to make it less visible once others depend on it.

---

## 5. Interview Questions with Complete Answers

### Basic Level

**Q1. What is the default access modifier in Java?**

When no modifier is written, the member has **package-private** (default) access: visible to all classes in the same package, and invisible outside it, including to subclasses in other packages. Note: interface members are an exception — interface methods are `public` by default.

**Q2. Can a top-level class be `private` or `protected`?**

No. Top-level classes can be only `public` or default. Nested classes can use all four modifiers.

**Q3. Which is more restrictive: `protected` or default?**

**Default** is more restrictive. Both allow access within the same package, but `protected` additionally allows access from subclasses in other packages.

### Intermediate Level

**Q4. Can a subclass in another package access a protected member through a parent-class object?**

No. In another package, a subclass can access a protected member only through inheritance — using `this`/`super`, or a reference of the subclass type (or its subclasses). `new Parent().protectedField` from another package is a compile error.

**Q5. Can we declare a constructor `private`? What happens then?**

Yes. No class outside can create objects using `new`. Also, no class can **extend** it from outside, because a subclass constructor must call `super()`, which is private. This is used in singletons, utility classes and static factory methods.

**Q6. Can a private method be overridden? Can a protected method become private in a subclass?**

A private method cannot be overridden because it is not visible to the subclass. A protected method cannot become private (or default) in a subclass because overriding cannot reduce visibility — compile error.

### Advanced Level

**Q7. Does `private` give real security?**

It gives **compile-time encapsulation**, not absolute security. Reflection with `setAccessible(true)` can access private members of classes on the classpath. Since **Java 9 modules**, strong encapsulation blocks reflective access into packages that a module does not `open`, which is a stronger protection. Sensitive data should still not rely only on access modifiers.

**Q8. What is module-level access (Java 9)?**

Java 9 introduced the **Java Platform Module System**. Even a `public` class is accessible from another module only if its package is **exported** in `module-info.java` (`exports com.bank.api;`). So in modular applications, `public` means "public to modules that can read this package", adding a level above packages.

**Q9. Why do nested classes access each other's private members?**

Java treats a top-level class and all its nested classes as one unit of encapsulation. Since Java 11, this is implemented using **nest-based access control** (JEP 181), where the JVM knows the classes are "nestmates". Before Java 11, the compiler generated hidden synthetic accessor methods.

---

## 6. Follow-Up Questions

**F1. Which access modifier did you use for fields in your project and why?**

"I kept all entity fields `private` and exposed them through getters, with setters only where updates made sense and with validation. Service-layer helper methods were `private`, and only the service interface methods were `public`."

**F2. Are interface methods public by default even without writing `public`?**

Yes. Abstract, default and static interface methods are implicitly `public`. Only `private` methods (Java 9+) are non-public.

**F3. Is `protected` useful for fields?**

Usually not recommended. Protected fields expose internal state to all subclasses and the entire package, making it hard to change the parent later. Prefer `private` fields with `protected` methods if subclasses need controlled access.

---

## 7. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| "default means public." | Default = package-private. |
| "protected is only for subclasses." | Also open to the whole package. |
| Accessing protected member via parent reference in another package | Not allowed. |
| `private class TopLevel` | Not allowed for top-level types. |
| Reducing visibility in overriding | Compile error. |
| Making everything `public` | Breaks encapsulation; start with `private`. |

---

## 8. Quick Revision

| Modifier | Class | Package | Subclass (other pkg) | World |
|---|---|---|---|---|
| `private` | ✅ | ❌ | ❌ | ❌ |
| default | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ (inheritance only) | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

- Top-level class: `public` or default only.
- Local variables: no access modifiers.
- Overriding: same or wider access.
- Private constructor → no outside instantiation, no outside subclassing.
- Java 9 modules add `exports` on top of `public`.
- Best practice: **start private, widen only when needed.**

---

[← Previous: Interfaces](07-Interfaces-and-Multiple-Inheritance.md) | [Back to README](../README.md) | [Next: static and final →](09-static-and-final.md)
