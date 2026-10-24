# Chapter 4.3 — `throw`, `throws`, Custom Exceptions and Exception Propagation

[← Previous: try, catch, finally](02-try-catch-finally.md) | [Back to README](../README.md) | [Next: try-with-resources and Best Practices →](04-try-with-resources-and-Best-Practices.md)

---

## What You Will Learn

- The `throw` keyword — throwing an exception explicitly
- The `throws` keyword — declaring exceptions in a method signature
- `throw` vs `throws` (classic comparison)
- Creating custom checked and unchecked exceptions
- Exception propagation through the call stack (checked vs unchecked)
- Exception chaining (wrapping) and re-throwing
- `throws` and method overriding

---

## 1. The `throw` Keyword

### 1.1 Interview Definition

> **The `throw` keyword is used to explicitly throw a single exception object from a method or block of code. It is followed by an instance of `Throwable` or its subclass, such as `throw new IllegalArgumentException("Age cannot be negative")`. When `throw` executes, the normal flow stops immediately and the JVM looks for a matching catch block, first in the current method and then up the call stack. `throw` is typically used for input validation, enforcing business rules, and re-throwing or wrapping exceptions.**

### 1.2 Understand the Concept

- Syntax: `throw new ExceptionType("message");`
- You can throw **only one** exception at a time.
- Statements **after** `throw` in the same block are unreachable (compile error if written directly after).
- If you throw a **checked** exception, the method must catch it or declare it with `throws`.
- You can throw built-in exceptions or your own **custom** exceptions.

### 1.3 Real-World Example

A **referee** (`throw`) blows the whistle and shows a red card when a rule is broken. The game stops at that point, and the decision is passed to the match officials (catch block / caller).

### 1.4 Java Example

```java
public class ThrowDemo {

    static void registerVoter(String name, int age) {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name is required");
        }
        if (age < 18) {
            throw new IllegalArgumentException(name + " is under 18 (age " + age + ")");
        }
        System.out.println(name + " registered successfully");
    }

    public static void main(String[] args) {
        String[][] people = {{"Ravi", "21"}, {"Kiran", "16"}, {"", "30"}};
        for (String[] p : people) {
            try {
                registerVoter(p[0], Integer.parseInt(p[1]));
            } catch (IllegalArgumentException e) {
                System.out.println("Registration failed: " + e.getMessage());
            }
        }
    }
}
```

**Expected Output:**

```text
Ravi registered successfully
Registration failed: Kiran is under 18 (age 16)
Registration failed: Name is required
```

**Explanation:** `registerVoter` validates its inputs and throws `IllegalArgumentException` (unchecked) for invalid data. Since it is unchecked, no `throws` clause is required. The caller decides how to respond.

---

## 2. The `throws` Keyword

### 2.1 Interview Definition

> **The `throws` keyword is used in a method or constructor declaration to declare the exceptions that the method might throw but does not handle itself. It tells the caller that it must handle or further declare those exceptions. Declaring checked exceptions with `throws` is mandatory if they are not caught inside the method; declaring unchecked exceptions is optional and serves only as documentation. A method can declare multiple exceptions separated by commas.**

### 2.2 Understand the Concept

```java
void readFile(String path) throws IOException, ParseException { ... }
```

- `throws` **does not throw** anything — it only **declares** possible exceptions.
- It passes the responsibility of handling to the **caller** ("duck the exception").
- `main` can also declare `throws Exception`; then the JVM's default handler deals with it.

### 2.3 Java Example

```java
import java.io.BufferedReader;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;

public class ThrowsDemo {

    // Declares a checked exception instead of handling it
    static String readFirstLine(String path) throws IOException {
        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            return br.readLine();
        }
    }

    // Declares multiple exceptions; the unchecked one is just documentation
    static int parseAge(String text) throws NumberFormatException, IllegalAccessException {
        if (text.equals("secret")) {
            throw new IllegalAccessException("Access denied");      // checked → must be declared
        }
        return Integer.parseInt(text);
    }

    public static void main(String[] args) {
        try {
            System.out.println(readFirstLine("does-not-exist.txt"));
        } catch (FileNotFoundException e) {                         // more specific first
            System.out.println("File not found - using defaults");
        } catch (IOException e) {
            System.out.println("Other I/O problem");
        }

        try {
            System.out.println("Age: " + parseAge("25"));
            System.out.println("Age: " + parseAge("secret"));
        } catch (IllegalAccessException e) {
            System.out.println("Caught: " + e.getMessage());
        }
    }
}
```

**Expected Output:**

```text
File not found - using defaults
Age: 25
Caught: Access denied
```

---

## 3. `throw` vs `throws`

| Point | `throw` | `throws` |
|---|---|---|
| Purpose | **Actually throws** an exception object | **Declares** exceptions a method may throw |
| Used | Inside a method body | In the method/constructor signature |
| Followed by | An exception **instance**: `throw new X()` | Exception **class names**: `throws X, Y` |
| Number | One exception at a time | Multiple, comma-separated |
| Effect on flow | Stops execution immediately | No effect on flow — compile-time information for callers |
| Checked exceptions | Throwing one requires catch or `throws` | Required for unhandled checked exceptions |
| Example | `throw new IOException("Disk full");` | `void save() throws IOException` |

### 3.1 Interview Answer

> "`throw` is used inside a method to actually throw an exception object, like `throw new IllegalArgumentException("Invalid age")`. It stops normal execution immediately and can throw only one exception at a time. `throws` is used in the method signature to declare which exceptions the method might throw without handling them, like `void readFile() throws IOException`. It doesn't throw anything itself; it tells the caller to handle or declare those exceptions. For checked exceptions, `throws` is mandatory if we don't catch them; for unchecked exceptions, it's optional. So `throw` is about doing, and `throws` is about declaring."

---

## 4. Custom Exceptions

### 4.1 Interview Definition

> **A custom (user-defined) exception is an exception class created by the programmer to represent an application-specific error condition, such as `InsufficientBalanceException` or `UserNotFoundException`. A checked custom exception is created by extending `Exception`, and an unchecked custom exception is created by extending `RuntimeException`. Custom exceptions usually provide constructors that accept a message and a cause, and may include extra fields with details about the error. They make error handling more meaningful, readable and specific to the business domain.**

### 4.2 Why Create Custom Exceptions?

1. **Meaningful names** — `InsufficientBalanceException` explains the problem better than `Exception("insufficient")`.
2. **Specific handling** — callers can catch exactly the business error they care about.
3. **Extra data** — fields like `requiredAmount`, `availableBalance`, `errorCode`.
4. **Clean API design** — a service layer can expose its own exception types and hide lower-level ones (SQL, I/O).
5. **Global error mapping** — frameworks (e.g., Spring `@ControllerAdvice`) map custom exceptions to HTTP status codes (404 for `UserNotFoundException`).

### 4.3 Checked or Unchecked Custom Exception?

| Make it **checked** (`extends Exception`) when… | Make it **unchecked** (`extends RuntimeException`) when… |
|---|---|
| The caller can reasonably **recover** and you want to **force** them to think about it | It represents a **programming error** or a condition the caller usually cannot fix |
| E.g., `InsufficientBalanceException` where the UI can ask for a smaller amount | E.g., `InvalidConfigurationException`, `ResourceNotFoundException` in web apps |

Modern frameworks (Spring, Hibernate) mostly use **unchecked** exceptions to avoid `throws` clauses spreading through every layer.

### 4.4 Java Example — Banking with Custom Exceptions

```java
public class CustomExceptionDemo {

    // Checked custom exception with extra data
    static class InsufficientBalanceException extends Exception {
        private final double required;
        private final double available;

        InsufficientBalanceException(double required, double available) {
            super(String.format("Need Rs.%.2f but only Rs.%.2f available", required, available));
            this.required = required;
            this.available = available;
        }

        double getShortfall() { return required - available; }
    }

    // Unchecked custom exception
    static class AccountNotFoundException extends RuntimeException {
        AccountNotFoundException(String accountId) {
            super("Account not found: " + accountId);
        }
    }

    static class Bank {
        private final java.util.Map<String, Double> accounts = new java.util.HashMap<>();

        Bank() {
            accounts.put("ACC1", 5000.0);
            accounts.put("ACC2", 300.0);
        }

        void withdraw(String accountId, double amount) throws InsufficientBalanceException {
            Double balance = accounts.get(accountId);
            if (balance == null) {
                throw new AccountNotFoundException(accountId);          // unchecked: no 'throws' needed
            }
            if (amount > balance) {
                throw new InsufficientBalanceException(amount, balance); // checked: declared above
            }
            accounts.put(accountId, balance - amount);
            System.out.println("Withdrew Rs." + amount + " from " + accountId + ", left Rs." + (balance - amount));
        }
    }

    public static void main(String[] args) {
        Bank bank = new Bank();
        String[][] requests = {{"ACC1", "1500"}, {"ACC2", "1000"}, {"ACC9", "100"}};

        for (String[] r : requests) {
            try {
                bank.withdraw(r[0], Double.parseDouble(r[1]));
            } catch (InsufficientBalanceException e) {
                System.out.println("Declined: " + e.getMessage() + " (short by Rs." + e.getShortfall() + ")");
            } catch (AccountNotFoundException e) {
                System.out.println("Error: " + e.getMessage());
            }
        }
    }
}
```

### 4.5 Code Explanation

- `InsufficientBalanceException extends Exception` → **checked**: `withdraw` must declare it, and callers must handle it. It stores extra data (`required`, `available`) and offers a helper (`getShortfall()`).
- `super(message)` passes the message to `Throwable`, so `getMessage()` returns it.
- `AccountNotFoundException extends RuntimeException` → **unchecked**: thrown without `throws`.
- The caller handles each business error differently.

### 4.6 Expected Output

```text
Withdrew Rs.1500.0 from ACC1, left Rs.3500.0
Declined: Need Rs.1000.00 but only Rs.300.00 available (short by Rs.700.0)
Error: Account not found: ACC9
```

### 4.7 Recommended Constructors for a Custom Exception

<!-- skip-check -->
```java
public class PaymentFailedException extends Exception {
    public PaymentFailedException() { super(); }
    public PaymentFailedException(String message) { super(message); }
    public PaymentFailedException(String message, Throwable cause) { super(message, cause); }   // for chaining
    public PaymentFailedException(Throwable cause) { super(cause); }
}
```

The `(message, cause)` constructor is the most important — it allows **exception chaining** so the original error is not lost.

### 4.8 Interview Answer — "How do you create a custom exception?"

> "To create a custom exception, I create a class that extends `Exception` if I want it to be checked, or `RuntimeException` if I want it unchecked. I add constructors that call `super(message)` and `super(message, cause)` so the message and the original cause are preserved. I can also add fields for extra details. For example, in a banking project, I created `InsufficientBalanceException extends Exception` with the required and available amounts, so the caller is forced to handle it and can show a meaningful message. For errors the caller usually can't recover from, like `AccountNotFoundException`, I extend `RuntimeException`. Custom exceptions make the code more readable and let callers handle specific business errors separately."

---

## 5. Exception Propagation

### 5.1 Interview Definition

> **Exception propagation is the process by which an exception that is not handled in the method where it occurs is passed to the calling method, and then to its caller, moving down the call stack until a matching catch block is found. If no method handles it, it reaches the JVM's default exception handler, which prints the stack trace and terminates the thread. Unchecked exceptions propagate automatically; checked exceptions propagate only if each method in the chain declares them with `throws`, otherwise the code does not compile.**

### 5.2 Understand the Concept

```text
Call stack (top = most recent)            Exception travels DOWN the stack
┌──────────────┐
│  method3()   │  ← exception thrown here, not caught
├──────────────┤        │
│  method2()   │  ← not caught, propagates  ▼
├──────────────┤        │
│  method1()   │  ← catch found → handled here
├──────────────┤
│  main()      │  ← continues normally after method1() returns
└──────────────┘
```

Each method that doesn't handle the exception is **terminated** (its stack frame is removed, after its `finally` blocks run).

### 5.3 Java Example

```java
public class PropagationDemo {

    static void method3() {
        System.out.println("method3: about to fail");
        int[] arr = new int[2];
        arr[3] = 7;                                 // unchecked exception thrown here
        System.out.println("method3: never printed");
    }

    static void method2() {
        try {
            method3();
        } finally {
            System.out.println("method2: finally runs while exception passes through");
        }
        System.out.println("method2: never printed");
    }

    static void method1() {
        try {
            method2();
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("method1: handled -> " + e.getMessage());
        }
        System.out.println("method1: continues after handling");
    }

    // Checked exceptions need 'throws' at every level that doesn't handle them
    static void readData() throws java.io.IOException {
        throw new java.io.IOException("disk not ready");
    }

    static void loadReport() throws java.io.IOException {   // must declare to propagate
        readData();
    }

    public static void main(String[] args) {
        method1();
        System.out.println("main: continues");

        try {
            loadReport();
        } catch (java.io.IOException e) {
            System.out.println("main: handled checked exception -> " + e.getMessage());
        }
    }
}
```

**Expected Output:**

```text
method3: about to fail
method2: finally runs while exception passes through
method1: handled -> Index 3 out of bounds for length 2
method1: continues after handling
main: continues
main: handled checked exception -> disk not ready
```

**Explanation:**
- `method3` throws `ArrayIndexOutOfBoundsException` and does not catch it → `method3` ends.
- `method2` doesn't catch it either, but its `finally` runs as the exception passes through → `method2` ends.
- `method1` catches it, handles it, and continues normally. `main` is unaffected.
- For the **checked** `IOException`, both `readData` and `loadReport` must declare `throws IOException`; if `loadReport` didn't, the code wouldn't compile.

### 5.4 Interview Answer — "What is exception propagation?"

> "Exception propagation means that when an exception is thrown and not caught in the current method, it is passed to the method that called it, and so on down the call stack until some method catches it. Every method on the way is terminated, but their finally blocks still run. If no method catches it, the JVM's default handler prints the stack trace and ends the thread. Unchecked exceptions propagate automatically. Checked exceptions also propagate, but every method in the chain that doesn't handle them must declare them using `throws`, otherwise the compiler gives an error. Propagation is useful because the method that detects an error is often not the right place to handle it — for example, a DAO method throws, and the service or controller layer decides what to show the user."

---

## 6. Exception Chaining and Re-throwing

### 6.1 Interview Definition

> **Exception chaining (wrapping) is the technique of catching one exception and throwing a different, usually higher-level, exception while attaching the original exception as its cause. This is done using constructors like `new ServiceException("message", cause)` or the `initCause()` method. The original exception remains available through `getCause()` and appears in the stack trace as "Caused by". It allows each layer of an application to expose meaningful exceptions without losing the root cause.**

### 6.2 Java Example

```java
import java.sql.SQLException;

public class ChainingDemo {

    static class DataAccessException extends RuntimeException {
        DataAccessException(String message, Throwable cause) { super(message, cause); }
    }

    static class UserServiceException extends Exception {
        UserServiceException(String message, Throwable cause) { super(message, cause); }
    }

    // Low level: database
    static void queryUser(int id) {
        try {
            throw new SQLException("Connection refused: db-server:3306");   // simulated DB failure
        } catch (SQLException e) {
            throw new DataAccessException("Could not load user " + id, e);   // wrap and re-throw
        }
    }

    // Service level
    static void getProfile(int id) throws UserServiceException {
        try {
            queryUser(id);
        } catch (DataAccessException e) {
            throw new UserServiceException("Profile temporarily unavailable", e);
        }
    }

    public static void main(String[] args) {
        try {
            getProfile(42);
        } catch (UserServiceException e) {
            System.out.println("Shown to user : " + e.getMessage());
            Throwable cause = e.getCause();
            System.out.println("Caused by     : " + cause.getClass().getSimpleName() + " - " + cause.getMessage());
            Throwable root = e;
            while (root.getCause() != null) {
                root = root.getCause();
            }
            System.out.println("Root cause    : " + root.getClass().getSimpleName() + " - " + root.getMessage());
        }
    }
}
```

**Expected Output:**

```text
Shown to user : Profile temporarily unavailable
Caused by     : DataAccessException - Could not load user 42
Root cause    : SQLException - Connection refused: db-server:3306
```

**Explanation:** Each layer translates the exception into its own vocabulary but keeps the original as the **cause**. The user sees a friendly message; developers can still find the root cause (`SQLException`) in logs. **Never lose the cause:** `throw new ServiceException("failed")` without `e` throws away the most important debugging information.

### 6.3 Re-throwing the Same Exception

<!-- skip-check -->
```java
try {
    process();
} catch (IOException e) {
    log.error("Processing failed", e);    // log, then
    throw e;                              // re-throw the same exception to the caller
}
```

Java 7 added **precise re-throw**: if you catch `Exception e` and re-throw it, the compiler knows only the checked exceptions actually thrown by the `try` block can escape, so the method only needs to declare those.

---

## 7. `throws` and Method Overriding

| Parent method declares | Child (overriding) method may declare |
|---|---|
| Nothing | No checked exceptions (unchecked are fine) |
| `throws IOException` | `throws IOException`, a subclass like `FileNotFoundException`, or nothing — **not** `Exception` |
| `throws Exception` | Any checked exception, or nothing |

Rule: **the overriding method cannot throw new or broader checked exceptions**, because callers using the parent type only prepared for the parent's exceptions.

---

## 8. Interview Questions with Complete Answers

### Basic Level

**Q1. Can we throw an object that is not a `Throwable`?**

No. `throw` requires an instance of `Throwable` or its subclasses. `throw new String("error")` is a compile error.

**Q2. Can we write statements after `throw`?**

Not directly after it in the same block — they are unreachable and cause a compile error. Code can follow in other branches (`if/else`) or after the enclosing block if the throw is conditional.

**Q3. Is `throws` mandatory for unchecked exceptions?**

No. It is optional and only documents the possibility. For checked exceptions that are not caught, it is mandatory.

### Intermediate Level

**Q4. Should a custom exception be checked or unchecked?**

If the caller can reasonably recover and should be forced to handle it (insufficient balance, invalid user input in a desktop app), checked. If it signals a programming error, or the caller usually can't do anything except report it (resource not found in a REST API, invalid state), unchecked. Many modern codebases prefer unchecked exceptions with global handlers.

**Q5. What happens if `main` declares `throws Exception` and an exception occurs?**

The exception propagates out of `main` to the JVM. The default handler prints the stack trace to `System.err` and the program terminates with a non-zero exit code. `finally` blocks on the way still run.

**Q6. What is exception chaining and why is it important?**

Wrapping a caught exception inside a new exception as its cause (`new X("msg", e)`). It lets higher layers throw meaningful, abstraction-appropriate exceptions while preserving the root cause for debugging (visible as "Caused by" in stack traces and through `getCause()`).

### Advanced Level

**Q7. Why do checked exceptions not work well with lambdas/streams?**

Functional interfaces like `Function` or `Consumer` don't declare checked exceptions, so a lambda calling a method that throws `IOException` won't compile inside `stream().map(...)`. Common solutions: catch inside the lambda and wrap in an unchecked exception (e.g., `UncheckedIOException`), or write a helper functional interface that allows exceptions.

**Q8. What is the difference between `throw e` and `throw new RuntimeException(e)` inside a catch?**

`throw e` re-throws the **same** exception object with its original stack trace; the method must declare it if it's checked. `throw new RuntimeException(e)` **wraps** it in a new unchecked exception, adding a new stack trace from this point and keeping the original as the cause — useful when converting checked to unchecked.

**Q9. Can a constructor throw an exception? Can it declare `throws`?**

Yes to both. If the constructor throws, the object is not created and the `new` expression does not complete. Subclass constructors must handle or declare checked exceptions thrown by the superclass constructor they call (they cannot catch around `super()`, so they must declare them).

---

## 9. Follow-Up Questions

**F1. Should you include sensitive data in exception messages?**

No. Messages often end up in logs or API responses. Avoid passwords, card numbers, tokens and personal data. Include IDs and context needed for debugging.

**F2. What is `UncheckedIOException`?**

A JDK unchecked exception (Java 8) that wraps an `IOException`. It's used by APIs like `Files.lines()` and `BufferedReader.lines()` streams, and by programmers who need to throw I/O failures from lambdas.

**F3. How does Spring handle custom exceptions?**

With `@ExceptionHandler` methods or a global `@ControllerAdvice`/`@RestControllerAdvice` class that maps exception types to HTTP responses — e.g., `UserNotFoundException` → 404, `ValidationException` → 400.

---

## 10. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| `throws new IOException()` | `throws` declares classes; `throw` throws an object. |
| Wrapping without the cause: `new X("msg")` | Pass the cause: `new X("msg", e)`. |
| Making every custom exception checked | Choose based on recoverability. |
| Catching and re-throwing without adding value | Either handle it, wrap it with context, or let it propagate. |
| Overriding method declaring a broader checked exception | Compile error. |
| Logging and re-throwing at every layer | Produces duplicate logs; log once where handled. |

---

## 11. Quick Revision

- `throw` → throws **one object** now: `throw new X("msg");`
- `throws` → **declares** classes in the signature: `void m() throws A, B`.
- Checked → must catch or declare; unchecked → optional.
- Custom exception: `extends Exception` (checked) or `extends RuntimeException` (unchecked); constructors with `message` and `cause`; extra fields allowed.
- **Propagation:** unhandled exceptions move down the call stack; `finally` blocks run on the way; checked ones need `throws` at each level.
- **Chaining:** `new HighLevelException("msg", cause)`; `getCause()`; "Caused by" in traces.
- Overriding can't add new/broader checked exceptions.

---

[← Previous: try, catch, finally](02-try-catch-finally.md) | [Back to README](../README.md) | [Next: try-with-resources and Best Practices →](04-try-with-resources-and-Best-Practices.md)
