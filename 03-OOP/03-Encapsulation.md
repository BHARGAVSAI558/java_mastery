# Chapter 3.3 — Encapsulation

[← Previous: Constructors, this and super](02-Constructors-this-and-super.md) | [Back to README](../README.md) | [Next: Inheritance →](04-Inheritance-and-Types.md)

---

## What You Will Learn

- What encapsulation is and the problem it solves
- How to implement encapsulation in Java (private fields + controlled access)
- Getters and setters with validation, read-only and write-only properties
- Encapsulation vs data hiding vs abstraction
- Encapsulation of mutable fields (defensive copying)
- JavaBeans naming conventions
- Interview questions and follow-ups

---

## 1. Encapsulation

### 1.1 Interview Definition

> **Encapsulation is the OOP principle of wrapping data (fields) and the code that operates on that data (methods) together into a single unit, the class, and restricting direct access to the data from outside. In Java, encapsulation is achieved by declaring fields as `private` and providing `public` methods, such as getters and setters, to read or modify them in a controlled way. This protects the object's internal state from invalid changes, allows validation, hides implementation details, and lets the class change its internal design without affecting the code that uses it.**

### 1.2 Understand the Concept

**The problem without encapsulation:**

```java
class BankAccount {
    public double balance;      // anyone can change it
}

BankAccount acc = new BankAccount();
acc.balance = -50000;          // invalid state! No one stopped this.
```

Any class can put the object into an **invalid state**. There is no place to add rules, logging or security checks.

**The solution — encapsulation:**
1. Make fields **`private`** → only code inside the class can access them directly.
2. Provide **`public` methods** that control access:
   - getters to read (`getBalance()`),
   - setters or business methods to modify, with **validation** (`deposit(amount)`, `withdraw(amount)`).
3. Outside code talks to the object only through these methods.

```text
           ┌───────────────── BankAccount ─────────────────┐
           │  private double balance;   ◄── hidden data     │
 outside   │                                               │
 code ───► │  public void deposit(amount)  ─┐ validation    │
           │  public void withdraw(amount) ─┼─ rules        │
           │  public double getBalance()   ─┘ controlled    │
           └───────────────────────────────────────────────┘
```

**Benefits of encapsulation:**

| Benefit | Explanation |
|---|---|
| **Data protection / validation** | Invalid values (negative balance, age = −5) are rejected. |
| **Control** | Make fields read-only (getter only), write-only (setter only), or fully hidden. |
| **Flexibility / maintainability** | Internal representation can change (e.g., store balance in paise as `long`) without changing outside code. |
| **Easier debugging** | All changes go through a few methods, so you can log or set breakpoints in one place. |
| **Security** | Sensitive data (password, PIN) is never exposed directly. |
| **Loose coupling** | Other classes depend on methods (the contract), not on internal fields. |

### 1.3 Real-World Example

An **ATM machine** is a perfect example of encapsulation. The cash and the account database are locked inside (private data). You cannot open the machine and take money. You can only use the **screen and keypad** (public methods): enter a PIN, withdraw, check balance. The machine **validates** every request — wrong PIN, insufficient balance, daily limit — before changing anything.

Another simple example: a **medicine capsule** wraps different ingredients inside one shell. You take the capsule as one unit without touching the ingredients.

### 1.4 Java Example — Bank Account

```java
public class EncapsulationDemo {

    static class BankAccount {
        private final String accountNumber;     // read-only after creation
        private String holderName;
        private double balance;
        private int pin;                         // write-only (no getter)
        private int transactionCount;

        BankAccount(String accountNumber, String holderName, int pin) {
            this.accountNumber = accountNumber;
            setHolderName(holderName);           // reuse validation
            setPin(pin);
        }

        // Getter (read access)
        public String getAccountNumber() { return accountNumber; }
        public String getHolderName()    { return holderName; }
        public double getBalance()       { return balance; }
        public int getTransactionCount() { return transactionCount; }

        // Setter with validation
        public void setHolderName(String holderName) {
            if (holderName == null || holderName.isBlank()) {
                throw new IllegalArgumentException("Name cannot be empty");
            }
            this.holderName = holderName.trim();
        }

        // Write-only property: no getPin() method
        public void setPin(int pin) {
            if (pin < 1000 || pin > 9999) {
                throw new IllegalArgumentException("PIN must be 4 digits");
            }
            this.pin = pin;
        }

        // Business methods instead of a plain setBalance()
        public void deposit(double amount) {
            if (amount <= 0) {
                System.out.println("Deposit rejected: amount must be positive");
                return;
            }
            balance += amount;
            transactionCount++;
            System.out.println("Deposited " + amount + ", balance = " + balance);
        }

        public void withdraw(double amount, int enteredPin) {
            if (enteredPin != pin) {
                System.out.println("Withdrawal rejected: wrong PIN");
                return;
            }
            if (amount > balance) {
                System.out.println("Withdrawal rejected: insufficient balance");
                return;
            }
            balance -= amount;
            transactionCount++;
            System.out.println("Withdrew " + amount + ", balance = " + balance);
        }
    }

    public static void main(String[] args) {
        BankAccount acc = new BankAccount("ACC1001", "  Meera  ", 4321);

        acc.deposit(5000);
        acc.deposit(-200);              // validation stops invalid change
        acc.withdraw(1000, 1111);       // wrong PIN
        acc.withdraw(9000, 4321);       // insufficient balance
        acc.withdraw(1500, 4321);       // valid

        System.out.println("Holder: [" + acc.getHolderName() + "]");
        System.out.println("Account: " + acc.getAccountNumber() + ", Balance: " + acc.getBalance()
                + ", Transactions: " + acc.getTransactionCount());

        // acc.balance = 1_000_000;     // compile error: balance has private access
        // acc.getPin();                // compile error: no such method

        try {
            acc.setPin(12);
        } catch (IllegalArgumentException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

### 1.5 Code Explanation

- All fields are `private`. Outside code cannot do `acc.balance = ...` — this is checked at **compile time**.
- `accountNumber` is `private final` with only a getter → **read-only**.
- `pin` has only a setter → **write-only**; the PIN can be changed but never read from outside.
- There is **no `setBalance()`**. Balance can change only through `deposit()` and `withdraw()`, which apply business rules. This is better encapsulation than blindly generating a setter for every field.
- The constructor calls `setHolderName()` so the same validation (and trimming) applies when creating the object.
- `transactionCount` is updated internally only — outside code can read it but never fake it.

### 1.6 Expected Output

```text
Deposited 5000.0, balance = 5000.0
Deposit rejected: amount must be positive
Withdrawal rejected: wrong PIN
Withdrawal rejected: insufficient balance
Withdrew 1500.0, balance = 3500.0
Holder: [Meera]
Account: ACC1001, Balance: 3500.0, Transactions: 2
Error: PIN must be 4 digits
```

### 1.7 Interview Answer — "What is encapsulation?"

> "Encapsulation means wrapping the data and the methods that work on that data into a single unit, which is the class, and restricting direct access to the data. In Java we do this by making the fields `private` and providing `public` methods like getters and setters to access them. The main advantage is control: inside the setter or business methods, we can validate values, so the object can never be put into an invalid state. For example, in a `BankAccount` class, `balance` is private, and it can be changed only through `deposit` and `withdraw`, which check that the amount is positive and that there is enough balance. Encapsulation also lets us change the internal implementation without affecting other classes, it improves security and maintainability, and it supports loose coupling. A fully encapsulated class is one in which all fields are private."

---

## 2. Getters and Setters — JavaBeans Conventions

| Field type | Getter | Setter |
|---|---|---|
| `String name` | `getName()` | `setName(String name)` |
| `int age` | `getAge()` | `setAge(int age)` |
| `boolean active` | **`isActive()`** (or `getActive()`) | `setActive(boolean active)` |

A **JavaBean** is a class that follows these conventions:
1. A **public no-argument constructor**.
2. **Private fields** with public getters/setters following the naming rules.
3. Usually implements **`Serializable`**.

Frameworks such as Spring, Hibernate, Jackson (JSON) and JSP rely on these conventions to read and write object properties automatically using reflection.

**Good practice — don't generate setters blindly.** Ask: "Should outside code be allowed to change this field? With what rules?" Many fields should be set only in the constructor (immutable) or changed only through meaningful methods (`deposit`, `changeAddress`, `promote`).

---

## 3. Encapsulating Mutable Fields — Defensive Copies

Making a field `private` is **not enough** if a getter returns a reference to a mutable object (like a `List`, an array or a `Date`). The caller could modify the internal object directly.

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class DefensiveCopyDemo {

    static class LeakyCourse {
        private final List<String> students = new ArrayList<>();
        void enroll(String name) { students.add(name); }
        List<String> getStudents() { return students; }          // leaks internal list!
    }

    static class SafeCourse {
        private final List<String> students = new ArrayList<>();

        SafeCourse(List<String> initial) {
            this.students.addAll(initial);                         // copy IN: don't keep caller's list
        }

        void enroll(String name) {
            if (students.size() >= 3) {
                System.out.println("Course full, cannot enroll " + name);
                return;
            }
            students.add(name);
        }

        List<String> getStudents() {
            return Collections.unmodifiableList(students);         // read-only view OUT
        }
    }

    public static void main(String[] args) {
        LeakyCourse leaky = new LeakyCourse();
        leaky.enroll("Ravi");
        leaky.getStudents().add("Hacker");                         // bypasses enroll() rules
        System.out.println("Leaky course: " + leaky.getStudents());

        List<String> initial = new ArrayList<>(List.of("Asha", "Bala"));
        SafeCourse safe = new SafeCourse(initial);
        initial.add("Outsider");                                   // caller's list changes
        safe.enroll("Chitra");
        safe.enroll("Dev");                                        // rule enforced
        System.out.println("Safe course: " + safe.getStudents());

        try {
            safe.getStudents().add("Hacker");
        } catch (UnsupportedOperationException e) {
            System.out.println("Cannot modify safe course list from outside");
        }
    }
}
```

**Expected Output:**

```text
Leaky course: [Ravi, Hacker]
Course full, cannot enroll Dev
Safe course: [Asha, Bala, Chitra]
Cannot modify safe course list from outside
```

**Explanation:**
- `LeakyCourse.getStudents()` returns the **actual internal list**. Outside code adds `"Hacker"` without going through `enroll()` — encapsulation is broken even though the field is `private`.
- `SafeCourse` copies the incoming list in the constructor (**copy in**), so later changes to `initial` don't affect it, and returns an **unmodifiable view** (**copy/protect out**). The size rule in `enroll()` can no longer be bypassed.
- Alternatives for getters: `List.copyOf(students)` (Java 10, immutable copy) or `new ArrayList<>(students)` (a mutable copy the caller can change freely without affecting the object).

---

## 4. Encapsulation vs Data Hiding vs Abstraction

| Point | Encapsulation | Data hiding | Abstraction |
|---|---|---|---|
| Meaning | Bundling data + methods in one unit and controlling access | Restricting direct access to data (making it `private`) | Showing only essential features, hiding *how* it works |
| Focus | **How** data is protected and organised | **Who** can access data | **What** an object does |
| Level | Implementation level | Part of encapsulation | Design level |
| Achieved by | Classes, `private` fields, getters/setters | Access modifiers (`private`) | Abstract classes, interfaces |
| Example | `BankAccount` with private `balance` and `deposit()` | `private double balance;` | `Payment` interface with `pay()` — caller doesn't know if it is UPI or card |

**Simple way to explain:**
- **Abstraction** hides **complexity** (implementation details) — "You only need to know *what* the button does."
- **Encapsulation** hides **data** and protects it — "You cannot touch the wires inside the machine."
- Data hiding is the *goal*; encapsulation is the *mechanism* that achieves it (plus bundling).

---

## 5. Interview Questions with Complete Answers

### Basic Level

**Q1. How do you achieve encapsulation in Java?**

1. Declare all fields `private`.
2. Provide `public` getter methods to read values where needed.
3. Provide `public` setter or business methods to modify values, with validation.
4. Optionally make fields `final` and omit setters for read-only data.

**Q2. What is a fully encapsulated class?**

A class in which **all data members are private** and accessed only through methods. Example: a `Student` class with private `name`, `rollNo`, `marks` and public getters/setters.

**Q3. What is the advantage of a setter over a public field?**

A setter is a **method**, so it can validate input (reject negative age), transform it (trim spaces), trigger other actions (log changes, update a timestamp), or enforce security. A public field accepts any value silently. Setters also let you change the internal representation later without breaking callers.

### Intermediate Level

**Q4. Can encapsulation be achieved without getters and setters?**

Yes. Getters and setters are just a common way. Encapsulation means controlled access — you can provide **meaningful methods** instead (`deposit()`, `withdraw()`, `changePassword(old, new)`), or no modification methods at all (immutable classes). In fact, adding a getter and setter for every field without rules gives very weak encapsulation.

**Q5. Is a class with private fields and public getters/setters for every field truly encapsulated?**

Technically yes, but poorly. If every setter accepts any value without validation, the fields are effectively public. Also, if getters return mutable internal objects (lists, arrays, dates), callers can modify internal state directly. Good encapsulation involves validation, avoiding unnecessary setters, and defensive copies.

**Q6. How does encapsulation help maintainability?**

Outside code depends only on the public methods. If you later change how data is stored — for example, storing `fullName` as separate `firstName` and `lastName`, or storing money in paise as a `long` instead of a `double` — you change only the class internals. The getters keep returning the same values, so no other class needs to change.

### Advanced Level

**Q7. Can reflection break encapsulation?**

Yes, `Field.setAccessible(true)` lets reflection read and modify private fields. However, since **Java 9 modules** (and strongly enforced since Java 16/17), reflection cannot access private members of classes in other modules unless the module explicitly `opens` the package. A `SecurityManager` could also restrict it in older versions. Encapsulation is a design-level protection, not an absolute security mechanism.

**Q8. How is encapsulation related to immutability?**

An immutable class is the **strongest form of encapsulation**: all fields are `private final`, there are no setters, mutable fields are defensively copied, and the class is often `final`. Its state can never change after construction, so it is automatically thread-safe and safe to share. Examples: `String`, `Integer`, `LocalDate`, records.

**Q9. What is the "Tell, Don't Ask" principle?**

Instead of asking an object for its data and making decisions outside (`if (acc.getBalance() >= amt) acc.setBalance(acc.getBalance() - amt);`), **tell** the object to do the work (`acc.withdraw(amt)`), and let it apply its own rules. This keeps logic and data together, which is the spirit of encapsulation.

---

## 6. Follow-Up Questions

**F1. Why is `boolean` getter named `isActive()`?**

That is the JavaBeans convention for boolean properties; it reads naturally (`if (user.isActive())`). Frameworks recognise both `isActive()` and `getActive()` for `boolean` (only `getActive()` for the wrapper `Boolean`).

**F2. Should a getter ever return `null`?**

Avoid it where possible because it forces null checks everywhere. Return an empty collection instead of `null` for lists, or use `Optional<T>` for values that may be absent (see [Part 8, Optional](../08-Java-8-and-Modern-Java/05-Optional.md)).

**F3. Where have you used encapsulation in a project?**

Sample answer: "In my student management project, the `Student` entity had private fields like `email` and `cgpa`. The `setCgpa` method validated that the value was between 0 and 10, and the email setter checked the format. The list of enrolled courses was returned as an unmodifiable list so other classes couldn't add courses without calling `enrollCourse()`, which checked prerequisites."

---

## 7. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| "Encapsulation = getters and setters." | It is about controlled access and protecting state; getters/setters are one tool. |
| Generating setters for every field | Only allow changes that make sense, with validation. |
| Returning internal mutable lists/arrays | Return unmodifiable views or copies. |
| Storing caller-provided mutable objects directly | Copy them in the constructor/setter. |
| Confusing encapsulation with abstraction | Encapsulation hides data; abstraction hides implementation complexity. |
| Making fields `protected` "for convenience" | It exposes them to all subclasses and the package; prefer `private`. |

---

## 8. Quick Revision

- **Encapsulation** = bundle data + methods in a class + restrict direct access.
- Implement: `private` fields → `public` getters/setters/business methods with **validation**.
- Read-only: getter only (+ `final`). Write-only: setter only.
- Benefits: data protection, validation, flexibility, maintainability, security, loose coupling.
- Mutable fields: **copy in, copy (or unmodifiable view) out**.
- JavaBean: public no-arg constructor, private fields, getX/setX/isX, usually Serializable.
- Encapsulation (protect data) vs Abstraction (hide complexity) vs Data hiding (restrict access).
- Real-world: ATM, capsule.

---

[← Previous: Constructors, this and super](02-Constructors-this-and-super.md) | [Back to README](../README.md) | [Next: Inheritance →](04-Inheritance-and-Types.md)
