# Chapter 4.1 — Exceptions, Errors and the Exception Hierarchy

[← Previous Part: OOP Question Bank](../03-OOP/14-OOP-Interview-Question-Bank.md) | [Back to README](../README.md) | [Next: try, catch, finally →](02-try-catch-finally.md)

---

## What You Will Learn

- What an exception is and what happens when one is not handled
- Why exception handling is needed
- The exception hierarchy: `Throwable`, `Error`, `Exception`, `RuntimeException`
- Exception vs Error
- Checked vs unchecked exceptions — the most asked comparison
- Common built-in exceptions and when they occur
- How to read a stack trace

---

## 1. What is an Exception?

### 1.1 Interview Definition

> **An exception is an unwanted or unexpected event that occurs during the execution of a program and disrupts its normal flow. In Java, an exception is represented by an object of a class that extends `Throwable`. When an error condition occurs, the JVM or the code creates an exception object containing the error's type, message and stack trace, and "throws" it. If the exception is not caught and handled, it propagates up the call stack, and if it reaches the top without being handled, the JVM's default exception handler prints the stack trace and terminates the thread. Exception handling allows the program to detect such problems, respond to them gracefully, and continue or shut down safely.**

### 1.2 Understand the Concept

Examples of exceptional situations:
- Dividing a number by zero.
- Accessing index 10 of an array of size 5.
- Calling a method on a `null` reference.
- Opening a file that does not exist.
- Losing a network connection while downloading.
- Converting `"abc"` into an integer.

**What happens without handling?**

```java
public class UnhandledDemo {
    public static void main(String[] args) {
        System.out.println("Start");
        int result = 10 / 0;                    // ArithmeticException thrown here
        System.out.println("Result: " + result);  // never runs
        System.out.println("End");              // never runs
    }
}
```

**Output (stdout + stderr):**

```text
Start
Exception in thread "main" java.lang.ArithmeticException: / by zero
	at UnhandledDemo.main(UnhandledDemo.java:4)
```

The program **stops abruptly**: "Result" and "End" never print. The **default exception handler** of the JVM printed:
1. The thread name (`main`),
2. The exception class (`java.lang.ArithmeticException`),
3. The message (`/ by zero`),
4. The **stack trace** — where it happened (class, method, file, line).

**Why exception handling?**
1. **Prevent abnormal termination** — the program can recover or fail gracefully.
2. **Separate error-handling code from normal logic** — no checking return codes after every line (C style).
3. **Propagate errors** to the layer that knows how to handle them.
4. **Group and classify errors** using the class hierarchy.
5. **Give meaningful feedback** to users and logs.

### 1.3 Real-World Example

You are driving to college and there is a **road block** (exception). Without a plan, you just stop in the middle of the road (program crash). With a plan (exception handling), you **take a detour** (catch block) and still reach college, or you **call and inform** the college (propagate the problem to someone who can decide).

---

## 2. The Exception Hierarchy

### 2.1 Diagram

```text
                               java.lang.Object
                                      │
                               java.lang.Throwable
                     ┌────────────────┴─────────────────┐
              java.lang.Error                    java.lang.Exception
         (serious, usually unrecoverable)    (conditions programs should handle)
         ├── OutOfMemoryError                ├── IOException  ─────────────────┐ checked
         ├── StackOverflowError              │     ├── FileNotFoundException   │
         ├── VirtualMachineError             │     └── EOFException            │
         ├── NoClassDefFoundError            ├── SQLException                  │
         ├── ExceptionInInitializerError     ├── ClassNotFoundException        │
         └── AssertionError                  ├── InterruptedException          │
                                             ├── CloneNotSupportedException ───┘
              ALL UNCHECKED                  │
                                             └── RuntimeException  ──────────── unchecked
                                                   ├── ArithmeticException
                                                   ├── NullPointerException
                                                   ├── ClassCastException
                                                   ├── IllegalArgumentException
                                                   │     └── NumberFormatException
                                                   ├── IllegalStateException
                                                   ├── IndexOutOfBoundsException
                                                   │     ├── ArrayIndexOutOfBoundsException
                                                   │     └── StringIndexOutOfBoundsException
                                                   ├── ConcurrentModificationException
                                                   └── UnsupportedOperationException
```

### 2.2 Understand the Hierarchy

| Class | Role |
|---|---|
| `Throwable` | Root of everything that can be thrown with `throw` and caught with `catch`. Holds the message, cause and stack trace. |
| `Error` | Serious problems in the **JVM or environment** that applications usually **should not try to catch** (out of memory, stack overflow). |
| `Exception` | Problems that a **program can reasonably handle**. |
| `RuntimeException` | Subclass of `Exception` for **programming bugs** (null, bad index, bad cast) — unchecked. |

---

## 3. Exception vs Error

### 3.1 Interview Definition

> **Both `Exception` and `Error` are subclasses of `Throwable`. An `Exception` represents a condition that a program can reasonably anticipate and recover from — such as invalid input, a missing file or a network failure — and it should be handled with try-catch or declared with `throws`. An `Error` represents a serious problem in the JVM or system environment — such as `OutOfMemoryError` or `StackOverflowError` — from which the application typically cannot recover, so it is generally not caught. All errors are unchecked, while exceptions can be checked or unchecked.**

### 3.2 Comparison

| Point | Exception | Error |
|---|---|---|
| Package/class | `java.lang.Exception` | `java.lang.Error` |
| Caused by | Application code or external conditions (input, files, network) | JVM / system resources |
| Recoverable? | Usually yes | Usually no |
| Should we catch it? | Yes, where we can handle it | Generally no |
| Checked/unchecked | Both types exist | Always unchecked |
| Examples | `IOException`, `SQLException`, `NullPointerException` | `OutOfMemoryError`, `StackOverflowError`, `NoClassDefFoundError` |

### 3.3 Real-World Example

- **Exception** = a **flat tyre**: annoying but you can change the tyre and continue.
- **Error** = the **engine catches fire**: you cannot fix it on the road; you stop.

---

## 4. Checked vs Unchecked Exceptions

### 4.1 Interview Definition

> **Checked exceptions are exceptions that the compiler checks at compile time. If a method can throw a checked exception, it must either handle it using try-catch or declare it using the `throws` clause; otherwise the code does not compile. They represent recoverable conditions outside the program's control, such as `IOException`, `SQLException` and `ClassNotFoundException`. All subclasses of `Exception` except `RuntimeException` and its subclasses are checked. Unchecked exceptions are `RuntimeException` and its subclasses, plus all `Error`s. The compiler does not force us to handle or declare them. They usually represent programming mistakes, such as `NullPointerException`, `ArrayIndexOutOfBoundsException` and `IllegalArgumentException`, which should be fixed in the code rather than caught.**

### 4.2 Understand the Concept

**How to tell if an exception is checked:** walk up its class hierarchy:
- reaches `RuntimeException` → **unchecked**
- reaches `Error` → **unchecked**
- reaches `Exception` without passing `RuntimeException` → **checked**

**Why does Java have checked exceptions?** To force the programmer to **think about** failures that are outside the program's control and likely to happen — a file may be missing, a database may be down. The compiler makes sure these cases are not forgotten.

**Why are runtime exceptions unchecked?** Because they can occur almost anywhere (any object access can be null, any division can be by zero). Forcing `try-catch` for them everywhere would make code unreadable. They are usually bugs; the fix is to correct the code (check for null, validate indexes), not to catch them.

### 4.3 Comparison Table

| Point | Checked exception | Unchecked exception |
|---|---|---|
| Checked by compiler? | **Yes** — must handle or declare | **No** |
| Superclass | `Exception` (but not `RuntimeException`) | `RuntimeException` or `Error` |
| When detected | Compile time (requirement) | Only at runtime |
| Typical cause | External conditions: files, network, database | Programming bugs: null, bad index, bad cast, invalid argument |
| Recovery | Often possible (retry, ask user, use default) | Usually fix the code |
| Examples | `IOException`, `FileNotFoundException`, `SQLException`, `ClassNotFoundException`, `InterruptedException` | `NullPointerException`, `ArithmeticException`, `ArrayIndexOutOfBoundsException`, `ClassCastException`, `NumberFormatException`, `IllegalArgumentException` |

> **Important:** "Checked" does **not** mean it happens at compile time. **All exceptions happen at runtime.** "Checked" only means the **compiler checks** that you have handled or declared it.

### 4.4 Real-World Example

- **Checked:** Before a trip, the travel agent **forces** you to sign a form about what to do if your flight is cancelled (compiler forces handling). Cancellation is outside your control but quite possible.
- **Unchecked:** Forgetting your own ticket at home — no form covers that. It is your mistake; the fix is to be careful (fix the bug).

### 4.5 Java Example

```java
import java.io.FileReader;
import java.io.IOException;

public class CheckedVsUncheckedDemo {

    // Checked: must declare with throws (or handle inside)
    static void readConfig(String path) throws IOException {
        try (FileReader reader = new FileReader(path)) {     // FileReader throws FileNotFoundException
            System.out.println("First char: " + (char) reader.read());
        }
    }

    // Unchecked: compiler does not force handling
    static int divide(int a, int b) {
        return a / b;                                        // may throw ArithmeticException
    }

    public static void main(String[] args) {
        // Checked exception: compiler forces this try-catch (or main must declare throws)
        try {
            readConfig("missing-config.txt");
        } catch (IOException e) {
            System.out.println("Checked exception handled: " + e.getClass().getSimpleName());
        }

        // Unchecked exceptions: handled here only for demonstration
        try {
            System.out.println(divide(10, 0));
        } catch (ArithmeticException e) {
            System.out.println("Unchecked: " + e.getClass().getSimpleName() + " - " + e.getMessage());
        }

        String text = null;
        try {
            System.out.println(text.length());
        } catch (NullPointerException e) {
            System.out.println("Unchecked: NullPointerException");
        }

        System.out.println("Is IOException checked? " + !RuntimeException.class.isAssignableFrom(IOException.class));
        System.out.println("Is NumberFormatException a RuntimeException? "
                + RuntimeException.class.isAssignableFrom(NumberFormatException.class));
    }
}
```

### 4.6 Code Explanation

- `new FileReader(path)` declares `throws FileNotFoundException` (a subclass of `IOException`, checked). If `readConfig` neither caught it nor declared `throws IOException`, the code would **not compile**: *"unreported exception FileNotFoundException; must be caught or declared to be thrown"*.
- `divide(10, 0)` compiles without any `try` or `throws` because `ArithmeticException` is unchecked.
- `text.length()` on `null` throws `NullPointerException`, also unchecked.
- `isAssignableFrom` checks the class hierarchy programmatically: `IOException` is not a `RuntimeException` → checked.

### 4.7 Expected Output

```text
Checked exception handled: FileNotFoundException
Unchecked: ArithmeticException - / by zero
Unchecked: NullPointerException
Is IOException checked? true
Is NumberFormatException a RuntimeException? true
```

### 4.8 Interview Answer — "Checked vs unchecked exceptions"

> "Checked exceptions are checked by the compiler. If my code calls a method that can throw a checked exception, I must either catch it with try-catch or declare it with `throws`, otherwise the code won't compile. They extend `Exception` but not `RuntimeException`, and they represent recoverable situations outside the program's control, like `IOException` when a file is missing or `SQLException` when the database fails. Unchecked exceptions extend `RuntimeException`, and errors are also unchecked. The compiler doesn't force handling them. They usually indicate programming bugs — like `NullPointerException`, `ArrayIndexOutOfBoundsException` or `IllegalArgumentException` — and the right fix is usually to correct the code, for example by validating input or checking for null. One important point: both types occur at runtime; 'checked' only means the compiler checks that we handled them."

---

## 5. Common Built-in Exceptions

| Exception | Type | When it occurs | Example |
|---|---|---|---|
| `ArithmeticException` | Unchecked | Integer division by zero | `10 / 0` |
| `NullPointerException` | Unchecked | Using a `null` reference | `String s = null; s.length();` |
| `ArrayIndexOutOfBoundsException` | Unchecked | Invalid array index | `arr[arr.length]` |
| `StringIndexOutOfBoundsException` | Unchecked | Invalid string index | `"abc".charAt(5)` |
| `ClassCastException` | Unchecked | Invalid downcast | `(Integer) (Object) "x"` |
| `NumberFormatException` | Unchecked | Invalid number text | `Integer.parseInt("12a")` |
| `IllegalArgumentException` | Unchecked | Method receives an invalid argument | `Thread.sleep(-1)` |
| `IllegalStateException` | Unchecked | Method called at the wrong time | `iterator.remove()` twice; `Scanner` used after `close()` |
| `ConcurrentModificationException` | Unchecked | Collection modified while iterating | removing in a for-each loop |
| `UnsupportedOperationException` | Unchecked | Operation not supported | `List.of(1).add(2)` |
| `InputMismatchException` | Unchecked | Scanner input of wrong type | `nextInt()` on "abc" |
| `IOException` | **Checked** | I/O failure | reading a file |
| `FileNotFoundException` | **Checked** | File does not exist / cannot be opened | `new FileReader("x.txt")` |
| `SQLException` | **Checked** | Database access error | JDBC calls |
| `ClassNotFoundException` | **Checked** | Class not found by name at runtime | `Class.forName("com.x.Driver")` |
| `InterruptedException` | **Checked** | A waiting/sleeping thread is interrupted | `Thread.sleep(1000)` |
| `CloneNotSupportedException` | **Checked** | `clone()` on a non-`Cloneable` class | `super.clone()` |
| `OutOfMemoryError` | Error | Heap (or metaspace) is full | creating huge arrays endlessly |
| `StackOverflowError` | Error | Call stack too deep | infinite recursion |
| `NoClassDefFoundError` | Error | Class present at compile time missing at runtime | missing JAR |

### 5.1 Java Example — Producing Common Exceptions Safely

```java
import java.util.ArrayList;
import java.util.List;

public class CommonExceptionsDemo {
    interface Action { void run() throws Exception; }

    static void attempt(String label, Action action) {
        try {
            action.run();
            System.out.println(label + " -> no exception");
        } catch (Exception e) {
            System.out.println(label + " -> " + e.getClass().getSimpleName());
        }
    }

    public static void main(String[] args) {
        attempt("10 / 0", () -> System.out.println(10 / 0));
        attempt("null.length()", () -> { String s = null; s.length(); });
        attempt("arr[3] of size 3", () -> { int[] a = new int[3]; a[3] = 1; });
        attempt("\"abc\".charAt(5)", () -> "abc".charAt(5));
        attempt("(Integer) \"x\"", () -> { Object o = "x"; Integer i = (Integer) o; });
        attempt("parseInt(\"12a\")", () -> Integer.parseInt("12a"));
        attempt("List.of(1).add(2)", () -> List.of(1).add(2));
        attempt("remove in for-each", () -> {
            List<Integer> list = new ArrayList<>(List.of(1, 2, 3));
            for (Integer n : list) {
                if (n == 1) list.remove(n);
            }
        });
        attempt("Class.forName(\"com.x.Missing\")", () -> Class.forName("com.x.Missing"));
        attempt("valid code", () -> Integer.parseInt("42"));
    }
}
```

**Expected Output:**

```text
10 / 0 -> ArithmeticException
null.length() -> NullPointerException
arr[3] of size 3 -> ArrayIndexOutOfBoundsException
"abc".charAt(5) -> StringIndexOutOfBoundsException
(Integer) "x" -> ClassCastException
parseInt("12a") -> NumberFormatException
List.of(1).add(2) -> UnsupportedOperationException
remove in for-each -> ConcurrentModificationException
Class.forName("com.x.Missing") -> ClassNotFoundException
valid code -> no exception
```

---

## 6. Reading a Stack Trace

```text
Exception in thread "main" java.lang.NullPointerException: Cannot invoke "String.length()" because "name" is null
	at com.app.UserService.validate(UserService.java:25)
	at com.app.UserService.register(UserService.java:12)
	at com.app.Main.main(Main.java:8)
Caused by: ...   (if the exception wraps another)
```

How to read it:
1. **First line:** exception type and message — *what* happened.
2. **First `at` line:** *where* it was thrown (`UserService.java` line 25, inside `validate`).
3. **Lines below:** the **call chain** — who called whom (`main` → `register` → `validate`). Read from top (most recent) to bottom (earliest).
4. **`Caused by:`** — the original (root) exception when exceptions are wrapped/chained. Look at the **last** "Caused by" for the root cause.

**Helpful NullPointerExceptions (Java 14+, default since 15):** the message tells you **exactly which variable was null**, e.g., *Cannot invoke "String.length()" because "name" is null*.

---

## 7. Interview Questions with Complete Answers

### Basic Level

**Q1. What is an exception?**

An exception is an event that disrupts the normal flow of a program during execution, represented by an object of a `Throwable` subclass. It carries information about the error (type, message, stack trace). If it isn't handled, the program (thread) terminates and the JVM prints the stack trace.

**Q2. What is the root class of the exception hierarchy?**

`java.lang.Throwable`. Its two direct subclasses are `Exception` and `Error`. Only objects of `Throwable` or its subclasses can be thrown and caught.

**Q3. Give examples of checked and unchecked exceptions.**

Checked: `IOException`, `FileNotFoundException`, `SQLException`, `ClassNotFoundException`, `InterruptedException`. Unchecked: `NullPointerException`, `ArithmeticException`, `ArrayIndexOutOfBoundsException`, `ClassCastException`, `NumberFormatException`, `IllegalArgumentException`.

### Intermediate Level

**Q4. What is the difference between `Exception` and `Error`?**

`Exception` represents conditions an application can reasonably handle (bad input, missing file). `Error` represents serious JVM/system problems (out of memory, stack overflow) that the application usually cannot recover from, so it should not catch them. All errors are unchecked.

**Q5. Is `NumberFormatException` checked or unchecked?**

Unchecked. Its hierarchy is `NumberFormatException → IllegalArgumentException → RuntimeException`. So the compiler does not force handling `Integer.parseInt()`, although handling it is wise when parsing user input.

**Q6. Can we catch an `Error`?**

Technically yes — `catch (OutOfMemoryError e)` compiles because `Error` is a `Throwable`. But it is usually a bad idea: after an error, the JVM may be in an unstable state. Some frameworks catch `Throwable` at the very top level only to log the problem before shutting down. `StackOverflowError` is sometimes caught in tests or tools that run untrusted recursive code.

**Q7. What is the difference between `ClassNotFoundException` and `NoClassDefFoundError`?**

| `ClassNotFoundException` | `NoClassDefFoundError` |
|---|---|
| **Checked exception** | **Error** (unchecked) |
| Thrown when code explicitly loads a class **by name** at runtime and it isn't found: `Class.forName()`, `ClassLoader.loadClass()` | Thrown when a class that was **present at compile time** is missing at runtime when the JVM tries to load it implicitly (e.g., `new MyClass()`), or its static initialisation failed earlier |
| Typical cause: wrong class name, missing JDBC driver JAR in `Class.forName("com.mysql.cj.jdbc.Driver")` | Typical cause: missing JAR on the runtime classpath, or a previous `ExceptionInInitializerError` |

### Advanced Level

**Q8. Why are `RuntimeException`s not checked by the compiler?**

They usually represent programming errors that can happen almost anywhere (null references, invalid indexes, invalid casts). If they were checked, nearly every method would need `try-catch` or `throws`, making code unreadable without real benefit. The recommended response is to prevent them with validation and correct code.

**Q9. Is `InterruptedException` checked? Why does `Thread.sleep()` force us to handle it?**

Yes, it is checked. Any blocking operation (`sleep`, `wait`, `join`) can be interrupted by another thread calling `interrupt()`. Java forces the caller to decide what to do — typically stop the work or restore the interrupt flag with `Thread.currentThread().interrupt()`. See [Part 7](../07-Multithreading-and-Concurrency/03-Thread-Methods-sleep-join-interrupt.md).

**Q10. What is `ExceptionInInitializerError`?**

An `Error` thrown when an exception occurs inside a **static initializer** (static block or static field initialisation). The class fails to initialise. Any later attempt to use that class throws `NoClassDefFoundError: Could not initialize class X`.

---

## 8. Follow-Up Questions

**F1. Does every exception extend `Exception`?**

No. Every throwable extends `Throwable`. `Error`s do not extend `Exception`. So `catch (Exception e)` does **not** catch `OutOfMemoryError` or `StackOverflowError`; `catch (Throwable t)` would.

**F2. Can you create your own checked and unchecked exceptions?**

Yes: extend `Exception` for a checked custom exception, or `RuntimeException` for an unchecked one. See [Chapter 4.3](03-throw-throws-Custom-Exceptions-Propagation.md).

**F3. Do exceptions affect performance?**

Throwing is relatively expensive because the JVM captures the stack trace (`fillInStackTrace`). That's fine for exceptional situations but wrong for normal control flow (don't use exceptions to end loops or check conditions). Try-catch blocks themselves have almost no cost when no exception is thrown.

---

## 9. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| "Checked exceptions happen at compile time." | All exceptions happen at runtime; checked ones are verified by the compiler. |
| "Errors and exceptions are the same." | Errors are serious JVM problems; usually don't catch them. |
| Catching `NullPointerException` instead of fixing the bug | Validate and fix the code. |
| "`catch (Exception e)` catches everything." | It does not catch `Error`s. |
| Using exceptions for normal flow | Expensive and unclear; use conditions. |

---

## 10. Quick Revision

- Exception = event disrupting normal flow; an object of a `Throwable` subclass.
- Unhandled → propagates → default handler prints stack trace → thread ends.
- Hierarchy: `Throwable` → `Error` (unchecked, don't catch) and `Exception` → `RuntimeException` (unchecked).
- **Checked** = `Exception` but not `RuntimeException` → handle or declare (compiler enforced). IOException, SQLException, ClassNotFoundException, InterruptedException.
- **Unchecked** = `RuntimeException` + `Error`. NPE, AIOOBE, ClassCast, NumberFormat, Arithmetic, IllegalArgument, IllegalState, CME, UnsupportedOperation.
- All exceptions occur at runtime.
- `ClassNotFoundException` (checked, explicit load by name) vs `NoClassDefFoundError` (error, class missing at runtime).
- Read stack traces top-down; root cause in the last "Caused by".

---

[← Previous Part: OOP Question Bank](../03-OOP/14-OOP-Interview-Question-Bank.md) | [Back to README](../README.md) | [Next: try, catch, finally →](02-try-catch-finally.md)
