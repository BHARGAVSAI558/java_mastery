# Chapter 4.4 — try-with-resources and Exception Handling Best Practices

[← Previous: throw, throws, Custom Exceptions](03-throw-throws-Custom-Exceptions-Propagation.md) | [Back to README](../README.md) | [Next Part: Collections →](../05-Collections-Framework/01-Collections-Overview-and-Hierarchy.md)

---

## What You Will Learn

- The problem with closing resources in `finally`
- try-with-resources (Java 7) and the `AutoCloseable` interface
- Order of closing, multiple resources, Java 9 enhancement
- Suppressed exceptions
- Writing your own `AutoCloseable` class
- Exception handling best practices (what interviewers expect you to know)
- A final question set for the whole Exception Handling part

---

## 1. The Problem with `finally` for Closing Resources

Before Java 7, resources (files, sockets, database connections) were closed in `finally`:

<!-- skip-check -->
```java
BufferedReader br = null;
try {
    br = new BufferedReader(new FileReader("data.txt"));
    System.out.println(br.readLine());
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (br != null) {
        try {
            br.close();                 // close() can ALSO throw IOException
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

Problems:
1. **Verbose** — nested try-catch inside `finally` and `null` checks.
2. **Easy to forget** — a missed `close()` causes resource leaks (too many open files, connection pool exhausted).
3. **Lost exceptions** — if both `readLine()` and `close()` throw, the exception from `close()` **replaces** the original one, hiding the real problem.

---

## 2. try-with-resources

### 2.1 Interview Definition

> **try-with-resources, introduced in Java 7, is a form of the try statement that declares one or more resources in parentheses after `try`. A resource is any object that implements the `java.lang.AutoCloseable` interface (or its subinterface `java.io.Closeable`). The JVM automatically closes each resource at the end of the try block — whether the block completes normally or throws an exception — in the reverse order of their declaration. If an exception occurs in the try block and another occurs while closing a resource, the closing exception is added to the original as a suppressed exception instead of replacing it. This makes resource handling shorter, safer and free from leaks.**

### 2.2 Syntax

```text
try (ResourceType r1 = new ResourceType(...);
     ResourceType r2 = new ResourceType(...)) {
    // use r1 and r2
} catch (SomeException e) {
    // optional
} finally {
    // optional — runs AFTER the resources are closed
}
```

**Rules:**
1. Resources must implement `AutoCloseable` (`void close() throws Exception`). `Closeable` (I/O) extends it with `void close() throws IOException`.
2. Resources are **implicitly final** inside the block.
3. Closed **automatically** in **reverse order** of declaration (last opened, first closed).
4. Resources are closed **before** `catch` and `finally` blocks run.
5. **Java 9:** you can use an existing **final or effectively final** variable: `try (reader) { ... }`.
6. `catch` and `finally` are optional.

### 2.3 Real-World Example

A **smart car park** that automatically **locks your car** when you walk away. You don't have to remember to lock it (close the resource); it happens automatically even if you leave in a hurry (exception).

### 2.4 Java Example — Reading a File

<!-- skip-check -->
```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class ReadFileDemo {
    public static void main(String[] args) {
        try (BufferedReader br = new BufferedReader(new FileReader("notes.txt"))) {
            String line;
            while ((line = br.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            System.out.println("Could not read file: " + e.getMessage());
        }
        // br is already closed here, even if an exception happened
    }
}
```

Compare with the pre-Java 7 version above: no `finally`, no null check, no nested try-catch.

### 2.5 Java Example — Custom Resources, Close Order and Suppressed Exceptions

```java
public class TryWithResourcesDemo {

    static class Resource implements AutoCloseable {
        private final String name;
        private final boolean failOnClose;

        Resource(String name, boolean failOnClose) {
            this.name = name;
            this.failOnClose = failOnClose;
            System.out.println("Open  " + name);
        }

        void use() { System.out.println("Use   " + name); }

        @Override
        public void close() {
            System.out.println("Close " + name);
            if (failOnClose) {
                throw new IllegalStateException("close failed for " + name);
            }
        }
    }

    public static void main(String[] args) {
        System.out.println("=== Case 1: normal flow, two resources ===");
        try (Resource db = new Resource("DB", false);
             Resource file = new Resource("File", false)) {
            db.use();
            file.use();
        } finally {
            System.out.println("finally (after resources closed)");
        }

        System.out.println("=== Case 2: exception in body + exception in close ===");
        try (Resource socket = new Resource("Socket", true)) {
            socket.use();
            throw new RuntimeException("body failed");
        } catch (RuntimeException e) {
            System.out.println("Caught primary : " + e.getMessage());
            for (Throwable s : e.getSuppressed()) {
                System.out.println("Suppressed     : " + s.getMessage());
            }
        }

        System.out.println("=== Case 3: Java 9 effectively final resource ===");
        Resource cache = new Resource("Cache", false);
        try (cache) {
            cache.use();
        }
    }
}
```

### 2.6 Code Explanation

- `Resource` implements `AutoCloseable`, so it can be used in try-with-resources.
- **Case 1:** resources open in order DB → File and close in **reverse** order File → DB. The `finally` block runs **after** both are closed.
- **Case 2:** the body throws "body failed", then `close()` throws "close failed". The **primary** exception (from the body) is the one propagated; the closing exception is attached as a **suppressed** exception, accessible via `getSuppressed()`. Nothing is lost.
- **Case 3 (Java 9):** an already-created, effectively final variable can be placed directly in the parentheses.

### 2.7 Expected Output

```text
=== Case 1: normal flow, two resources ===
Open  DB
Open  File
Use   DB
Use   File
Close File
Close DB
finally (after resources closed)
=== Case 2: exception in body + exception in close ===
Open  Socket
Use   Socket
Close Socket
Caught primary : body failed
Suppressed     : close failed for Socket
=== Case 3: Java 9 effectively final resource ===
Open  Cache
Use   Cache
Close Cache
```

### 2.8 Interview Answer — "What is try-with-resources?"

> "try-with-resources is a feature from Java 7 that automatically closes resources like files, streams, sockets and database connections. We declare the resources inside parentheses after `try`, and any object that implements `AutoCloseable` can be used. When the try block finishes — normally or with an exception — Java automatically calls `close()` on each resource in the reverse order of creation. This removes the need for a finally block with null checks and nested try-catch, and prevents resource leaks. Another advantage is suppressed exceptions: if the body throws an exception and `close()` also throws one, the original exception is kept and the closing exception is attached as suppressed, so we don't lose the real error. From Java 9, we can also use an existing effectively final variable inside the parentheses."

### 2.9 `AutoCloseable` vs `Closeable`

| | `AutoCloseable` | `Closeable` |
|---|---|---|
| Package | `java.lang` (Java 7) | `java.io` (Java 5) |
| `close()` throws | `Exception` | `IOException` |
| Relationship | Parent | Extends `AutoCloseable` |
| Idempotent requirement | Not required | `close()` should have no effect if called again |
| Used by | `Connection`, `Statement`, `ResultSet`, `Scanner`, `ExecutorService` (Java 19+), streams | `InputStream`, `OutputStream`, `Reader`, `Writer` |

---

## 3. Exception Handling Best Practices

These are the points interviewers expect from a candidate who has written real code.

| # | Practice | Why |
|---|---|---|
| 1 | **Catch specific exceptions**, not `Exception`/`Throwable` | Handle only what you understand; don't hide bugs |
| 2 | **Never swallow exceptions** (empty catch) | Silent failures are the hardest bugs to find |
| 3 | **Use try-with-resources** for anything closeable | No leaks, no lost exceptions |
| 4 | **Throw early, catch late** | Validate inputs immediately; handle where you have enough context to decide |
| 5 | **Preserve the cause** when wrapping: `new X("msg", e)` | Root cause stays visible |
| 6 | **Use meaningful messages** with context (IDs, values) | Faster debugging |
| 7 | **Prefer standard exceptions** (`IllegalArgumentException`, `IllegalStateException`, `UnsupportedOperationException`) when they fit | Familiar to all developers |
| 8 | **Create custom exceptions** for business errors | Clear, domain-specific handling |
| 9 | **Don't use exceptions for normal control flow** | Slow and confusing |
| 10 | **Log once**, at the place where the exception is handled | Avoid duplicate log entries |
| 11 | **Don't return from `finally`** or throw from `finally` | Hides exceptions and results |
| 12 | **Clean up in `finally` only when try-with-resources is not possible** (e.g., unlocking a `Lock`) | Guaranteed cleanup |
| 13 | **Restore the interrupt flag** when catching `InterruptedException` (`Thread.currentThread().interrupt()`) | Don't lose cancellation requests |
| 14 | **Validate to prevent runtime exceptions** (null checks, `Objects.requireNonNull`, bounds checks) | Fix bugs instead of catching them |
| 15 | **Document exceptions** with `@throws` in Javadoc | Callers know what to expect |
| 16 | **Don't expose internal details** (stack traces, SQL) to end users | Security and user experience |

### 3.1 Example — Good vs Bad

<!-- skip-check -->
```java
// BAD
try {
    process(order);
} catch (Exception e) {
    // nothing here
}

// GOOD
try {
    process(order);
} catch (PaymentDeclinedException e) {
    log.warn("Payment declined for order {}: {}", order.getId(), e.getMessage());
    notifyCustomer(order, "Your payment was declined. Please try another card.");
} catch (InventoryException e) {
    log.error("Inventory error for order {}", order.getId(), e);
    throw new OrderProcessingException("Order " + order.getId() + " could not be completed", e);
}
```

### 3.2 Example — Fail Fast with Validation

```java
import java.util.Objects;

public class FailFastDemo {
    static class Transfer {
        private final String from;
        private final String to;
        private final double amount;

        Transfer(String from, String to, double amount) {
            this.from = Objects.requireNonNull(from, "from account must not be null");
            this.to = Objects.requireNonNull(to, "to account must not be null");
            if (amount <= 0) {
                throw new IllegalArgumentException("amount must be positive, got " + amount);
            }
            if (from.equals(to)) {
                throw new IllegalArgumentException("cannot transfer to the same account");
            }
            this.amount = amount;
        }

        @Override public String toString() { return from + " -> " + to + " : " + amount; }
    }

    public static void main(String[] args) {
        Object[][] inputs = {{"A1", "B2", 500.0}, {null, "B2", 100.0}, {"A1", "B2", -5.0}, {"A1", "A1", 10.0}};
        for (Object[] in : inputs) {
            try {
                System.out.println("Created: " + new Transfer((String) in[0], (String) in[1], (Double) in[2]));
            } catch (NullPointerException | IllegalArgumentException e) {
                System.out.println("Rejected: " + e.getMessage());
            }
        }
    }
}
```

**Expected Output:**

```text
Created: A1 -> B2 : 500.0
Rejected: from account must not be null
Rejected: amount must be positive, got -5.0
Rejected: cannot transfer to the same account
```

Invalid objects are rejected **at creation** (fail fast) with clear messages, instead of causing confusing failures later.

---

## 4. Complete Interview Question Set — Exception Handling

### Basic Level

**Q1. What is the difference between `final`, `finally` and `finalize()`?**

`final` is a modifier that prevents re-assignment, overriding or inheritance. `finally` is a block that always runs after try/catch for cleanup. `finalize()` was an `Object` method called by the GC before reclaiming an object — deprecated since Java 9 and for removal since Java 18. They are unrelated despite similar names.

**Q2. Which resources can be used in try-with-resources?**

Any object implementing `AutoCloseable` (or `Closeable`): streams, readers/writers, `Scanner`, JDBC `Connection`/`Statement`/`ResultSet`, sockets, channels, and your own classes that implement `AutoCloseable`.

**Q3. In which order are resources closed?**

In the reverse order of their declaration — the last resource opened is closed first. This matters when one resource depends on another (e.g., close the `ResultSet` before the `Statement` before the `Connection`).

### Intermediate Level

**Q4. What are suppressed exceptions?**

When an exception is thrown in the try block and another exception is thrown while closing a resource, the second one is "suppressed" — attached to the primary exception using `addSuppressed()` — so the primary exception propagates and nothing is lost. You can read them with `getSuppressed()`. You can also add suppressed exceptions manually.

**Q5. Does `finally` run before or after resources are closed in try-with-resources?**

After. The order is: try body → resources closed (reverse order) → `catch` (if an exception occurred) → `finally`.

**Q6. What is the difference between `throw` and `throws`?**

`throw` actually throws one exception object inside a method; `throws` declares in the method signature which exceptions may be thrown so callers can handle them. (See [Chapter 4.3](03-throw-throws-Custom-Exceptions-Propagation.md).)

**Q7. How should you handle `InterruptedException`?**

Either propagate it (declare `throws InterruptedException`), or if you catch it, restore the thread's interrupted status with `Thread.currentThread().interrupt()` and stop the current work. Swallowing it hides a cancellation request from the code above.

### Advanced Level

**Q8. What happens if both the try block and `close()` throw exceptions in the old try-finally style?**

The exception from `close()` in `finally` replaces the original exception, which is lost completely. try-with-resources fixes this by keeping the original as primary and adding the close exception as suppressed.

**Q9. Can we use try-with-resources with a resource that is `null`?**

Yes. If the resource variable is `null`, `close()` is simply not called — no `NullPointerException` from the automatic close.

**Q10. Design question: how would you design exception handling for a REST API?**

- Use **custom unchecked exceptions** for business errors (`ResourceNotFoundException`, `ValidationException`, `ConflictException`).
- Validate input at the boundary (fail fast) and throw meaningful exceptions.
- Wrap low-level exceptions (SQL, I/O) into domain exceptions, preserving the cause.
- Use a **global exception handler** (e.g., Spring `@RestControllerAdvice`) to map exceptions to HTTP status codes (404, 400, 409, 500) and a consistent error JSON (`timestamp`, `status`, `message`, `path`, `errorCode`).
- Log unexpected exceptions with stack traces once; never return stack traces or SQL to clients.

**Q11. What is the output?**

```java
public class FinalQuiz {
    static class R implements AutoCloseable {
        public void close() { System.out.println("close"); }
    }

    static int test() {
        try (R r = new R()) {
            System.out.println("body");
            return 1;
        } finally {
            System.out.println("finally");
        }
    }

    public static void main(String[] args) {
        System.out.println("result " + test());
    }
}
```

**Expected Output:**

```text
body
close
finally
result 1
```

The return value 1 is saved, then the resource is closed, then `finally` runs, then the method returns.

---

## 5. Follow-Up Questions

**F1. Can try-with-resources be used without catch and finally?**

Yes. `try (Resource r = ...) { ... }` alone is valid. If `close()` declares a checked exception, the method must still handle or declare it.

**F2. Is the resource variable accessible in catch or finally?**

No. Its scope is limited to the try block. By the time catch/finally run, it is already closed.

**F3. Why does `Scanner(System.in)` in try-with-resources cause problems later?**

Closing the `Scanner` also closes `System.in`, which cannot be reopened. Later attempts to read from the console fail. Create one console `Scanner` for the whole program and don't close it early.

---

## 6. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Closing resources manually in `finally` in new code | Use try-with-resources. |
| Using a class that doesn't implement `AutoCloseable` | Compile error: incompatible types. |
| Thinking `finally` runs before resources close | Resources close first. |
| Ignoring suppressed exceptions while debugging | Check `getSuppressed()` / logs. |
| Catching `Exception` everywhere | Catch specific types; global handler at the top. |
| Swallowing `InterruptedException` | Restore the interrupt flag. |

---

## 7. Quick Revision — Whole Exception Handling Part

- `Throwable` → `Error` (don't catch) / `Exception` → `RuntimeException` (unchecked).
- Checked: compiler forces catch/declare. Unchecked: programming errors.
- `try` → `catch` (specific first; multi-catch `A | B`) → `finally` (always, except `System.exit`/crash).
- `return` in `finally` = bad (overrides and swallows).
- `throw` = throw object; `throws` = declare.
- Custom: extend `Exception` (checked) or `RuntimeException` (unchecked); keep `cause`.
- Propagation down the call stack; checked needs `throws` at each level.
- **try-with-resources:** `AutoCloseable`, auto close in reverse order, before catch/finally, suppressed exceptions, Java 9 effectively-final resources.
- Best practices: specific catches, no swallowing, fail fast, preserve cause, log once, no exceptions for flow control, restore interrupts.

---

[← Previous: throw, throws, Custom Exceptions](03-throw-throws-Custom-Exceptions-Propagation.md) | [Back to README](../README.md) | [Next Part: Collections →](../05-Collections-Framework/01-Collections-Overview-and-Hierarchy.md)
