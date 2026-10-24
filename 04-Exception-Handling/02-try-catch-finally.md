# Chapter 4.2 — `try`, `catch` and `finally`

[← Previous: Exceptions and Hierarchy](01-Exceptions-Errors-and-Hierarchy.md) | [Back to README](../README.md) | [Next: throw, throws, Custom Exceptions →](03-throw-throws-Custom-Exceptions-Propagation.md)

---

## What You Will Learn

- How `try` and `catch` work, step by step
- Multiple catch blocks and their order
- Multi-catch (Java 7)
- Nested try blocks
- The `finally` block — when it runs and when it does not
- `return` inside `try`/`catch`/`finally` — classic tricky questions
- Useful methods of exception objects

---

## 1. `try` and `catch`

### 1.1 Interview Definition

> **The `try` block contains code that might throw an exception, and the `catch` block contains code that handles a specific type of exception. When an exception occurs inside the `try` block, the remaining statements in the `try` block are skipped, and the JVM looks for the first `catch` block whose parameter type matches the thrown exception (the same class or a superclass). If a match is found, that catch block executes and the program continues normally after the try-catch structure. If no catch block matches, the exception propagates to the calling method.**

### 1.2 Understand the Concept

```text
try {
    statement 1;
    statement 2;   ← exception occurs here
    statement 3;   ← SKIPPED
} catch (ExceptionType e) {
    handling code; ← runs if the type matches
}
statement 4;       ← program continues here
```

**Step by step:**
1. Statements in `try` run normally.
2. If no exception occurs, all `catch` blocks are **skipped**.
3. If an exception occurs, the JVM creates an exception object and **jumps** out of `try` immediately.
4. The JVM checks each `catch` from **top to bottom** and runs the **first** one whose type matches (exact class or a parent class).
5. After the catch block finishes, execution continues **after** the whole try-catch.
6. If no catch matches, the exception goes to the **caller** (propagation).

**Rules:**
- A `try` must be followed by at least one `catch` or a `finally` (or be a try-with-resources).
- No code is allowed between the end of `try` and `catch`/`finally`.
- Variables declared inside `try` are **not visible** in `catch` or `finally`.

### 1.3 Real-World Example

A **circus trapeze artist** performs high in the air (try block). A **safety net** is below (catch block). If the artist falls (exception), the net catches them and the show continues. If everything goes well, the net is never used.

### 1.4 Java Example

```java
public class TryCatchDemo {
    public static void main(String[] args) {
        System.out.println("1. Program started");

        try {
            int[] marks = {80, 90, 70};
            System.out.println("2. Before risky line");
            int average = (marks[0] + marks[1] + marks[2]) / 0;      // ArithmeticException
            System.out.println("3. Average = " + average);           // skipped
        } catch (ArithmeticException e) {
            System.out.println("4. Caught: " + e.getMessage());
        }

        System.out.println("5. Program continues normally");

        try {
            System.out.println("6. No exception here");
        } catch (Exception e) {
            System.out.println("Never printed");
        }
        System.out.println("7. Program finished");
    }
}
```

**Expected Output:**

```text
1. Program started
2. Before risky line
4. Caught: / by zero
5. Program continues normally
6. No exception here
7. Program finished
```

**Explanation:** Line 3 is skipped because control jumps to the catch block. After handling, the program continues at line 5 — it does **not** go back into the `try`. In the second `try`, nothing fails, so the `catch` is skipped.

---

## 2. Multiple catch Blocks and Their Order

### 2.1 Interview Definition

> **A single `try` block can be followed by multiple `catch` blocks to handle different exception types differently. The JVM checks the catch blocks in order and executes only the first matching one. Therefore, catch blocks for more specific exceptions (subclasses) must come before catch blocks for more general exceptions (superclasses); if a superclass catch block comes first, the subclass catch block becomes unreachable and the compiler reports an error.**

### 2.2 Java Example

```java
public class MultipleCatchDemo {
    static void process(String input, int index) {
        try {
            int[] data = {10, 20, 30};
            int number = Integer.parseInt(input);          // may throw NumberFormatException
            int value = data[index];                        // may throw ArrayIndexOutOfBoundsException
            System.out.println("Result: " + value / number); // may throw ArithmeticException
        } catch (NumberFormatException e) {
            System.out.println("Invalid number: " + input);
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Invalid index: " + index);
        } catch (ArithmeticException e) {
            System.out.println("Cannot divide by zero");
        } catch (RuntimeException e) {                      // general handler LAST
            System.out.println("Other runtime problem: " + e);
        }
    }

    public static void main(String[] args) {
        process("5", 1);
        process("abc", 1);
        process("5", 7);
        process("0", 2);
    }
}
```

**Expected Output:**

```text
Result: 4
Invalid number: abc
Invalid index: 7
Cannot divide by zero
```

### 2.3 Wrong Order = Compile Error

<!-- skip-check -->
```java
try {
    int x = Integer.parseInt("abc");
} catch (Exception e) {                    // catches everything
    System.out.println("General");
} catch (NumberFormatException e) {        // COMPILE ERROR: exception NumberFormatException has already been caught
    System.out.println("Specific");
}
```

**Rule: specific → general (child → parent).**

### 2.4 Multi-Catch (Java 7)

When several exceptions need the **same handling**, catch them in one block with `|`:

```java
public class MultiCatchDemo {
    public static void main(String[] args) {
        String[] inputs = {"12", "x", null};
        for (String in : inputs) {
            try {
                int n = Integer.parseInt(in.trim());
                System.out.println("Parsed: " + n);
            } catch (NumberFormatException | NullPointerException e) {
                System.out.println("Bad input (" + e.getClass().getSimpleName() + ")");
            }
        }
    }
}
```

**Expected Output:**

```text
Parsed: 12
Bad input (NumberFormatException)
Bad input (NullPointerException)
```

**Multi-catch rules:**
1. The types must **not be related by inheritance** — `catch (IOException | FileNotFoundException e)` is a compile error (the subclass is redundant).
2. The parameter `e` is **implicitly final** — you cannot assign `e = new ...`.
3. Reduces duplicate code and produces smaller bytecode.

---

## 3. Nested try Blocks

A `try` block can contain another try-catch. If the inner catch cannot handle an exception, it propagates to the outer catch.

```java
public class NestedTryDemo {
    public static void main(String[] args) {
        try {
            System.out.println("Outer try starts");
            try {
                int[] arr = new int[2];
                arr[5] = 10;                                   // AIOOBE
            } catch (ArithmeticException e) {                  // does not match
                System.out.println("Inner catch: arithmetic");
            }
            System.out.println("Outer try continues");         // skipped
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Outer catch handled: " + e.getMessage());
        }
        System.out.println("Done");
    }
}
```

**Expected Output:**

```text
Outer try starts
Outer catch handled: Index 5 out of bounds for length 2
Done
```

The inner `catch` handles only `ArithmeticException`, so the exception moves to the outer `try`'s catch.

---

## 4. The `finally` Block

### 4.1 Interview Definition

> **The `finally` block is an optional block that follows a `try` (and its `catch` blocks) and contains code that always executes, whether an exception is thrown or not, whether it is caught or not, and even if the `try` or `catch` block executes a `return`, `break` or `continue`. It is used for cleanup work such as closing files, database connections and network sockets, or releasing locks. The only situations in which `finally` does not run are when the JVM exits — for example through `System.exit()` — when the JVM crashes or the process is killed, or when the thread running it never finishes the try block (such as an infinite loop or deadlock).**

### 4.2 When Does `finally` Run?

| Situation | Does `finally` run? |
|---|---|
| No exception in `try` | ✅ Yes |
| Exception thrown and caught | ✅ Yes (after `catch`) |
| Exception thrown and **not** caught | ✅ Yes (before the exception propagates) |
| `return` inside `try` or `catch` | ✅ Yes (before the method actually returns) |
| `break`/`continue` inside `try` in a loop | ✅ Yes |
| `System.exit()` in `try`/`catch` | ❌ No |
| JVM crash / power failure / `kill -9` | ❌ No |
| Infinite loop or deadlock in `try` | ❌ Never reached |
| Daemon thread when JVM exits | ❌ May not run |

### 4.3 Real-World Example

When you **leave a hotel room**, you always **hand back the key card** — whether your trip went perfectly, or there was a problem with the bill, or you are leaving early. Returning the key is the `finally` block: cleanup that must happen in all cases.

### 4.4 Java Example

```java
public class FinallyDemo {

    static void scenario(int caseNo) {
        System.out.println("--- Case " + caseNo + " ---");
        try {
            System.out.println("try: opening resource");
            if (caseNo == 2) {
                int x = 5 / 0;                                // caught exception
            }
            if (caseNo == 3) {
                throw new IllegalStateException("not handled here");   // uncaught by this method
            }
            System.out.println("try: work completed");
        } catch (ArithmeticException e) {
            System.out.println("catch: " + e.getMessage());
        } finally {
            System.out.println("finally: closing resource");
        }
        System.out.println("after try-catch-finally");
    }

    public static void main(String[] args) {
        scenario(1);
        scenario(2);
        try {
            scenario(3);
        } catch (IllegalStateException e) {
            System.out.println("main caught: " + e.getMessage());
        }
    }
}
```

**Expected Output:**

```text
--- Case 1 ---
try: opening resource
try: work completed
finally: closing resource
after try-catch-finally
--- Case 2 ---
try: opening resource
catch: / by zero
finally: closing resource
after try-catch-finally
--- Case 3 ---
try: opening resource
finally: closing resource
main caught: not handled here
```

**Explanation:**
- **Case 1:** no exception → `finally` runs → continue.
- **Case 2:** exception caught → `catch` → `finally` → continue.
- **Case 3:** the `IllegalStateException` is not caught in `scenario`, but `finally` **still runs** before the exception leaves the method. "after try-catch-finally" does not print because the method exits with the exception; `main` catches it.

### 4.5 `finally` and `return` — Tricky Cases

```java
public class FinallyReturnDemo {

    static int returnFromTry() {
        try {
            return 1;
        } finally {
            System.out.println("finally runs even after return in try");
        }
    }

    @SuppressWarnings("finally")
    static int finallyOverridesReturn() {
        try {
            return 1;
        } finally {
            return 2;                    // overrides the try's return value (bad practice!)
        }
    }

    static int modifyPrimitiveInFinally() {
        int x = 10;
        try {
            return x;                    // value 10 is saved for return
        } finally {
            x = 20;                      // changing x does NOT change the saved return value
        }
    }

    static StringBuilder modifyObjectInFinally() {
        StringBuilder sb = new StringBuilder("try");
        try {
            return sb;                   // reference saved
        } finally {
            sb.append("+finally");       // object is modified → caller sees it
        }
    }

    @SuppressWarnings("finally")
    static int finallySwallowsException() {
        try {
            throw new RuntimeException("lost!");
        } finally {
            return 99;                   // the exception is silently discarded!
        }
    }

    public static void main(String[] args) {
        System.out.println("returnFromTry = " + returnFromTry());
        System.out.println("finallyOverridesReturn = " + finallyOverridesReturn());
        System.out.println("modifyPrimitiveInFinally = " + modifyPrimitiveInFinally());
        System.out.println("modifyObjectInFinally = " + modifyObjectInFinally());
        System.out.println("finallySwallowsException = " + finallySwallowsException());
    }
}
```

**Expected Output:**

```text
finally runs even after return in try
returnFromTry = 1
finallyOverridesReturn = 2
modifyPrimitiveInFinally = 10
modifyObjectInFinally = try+finally
finallySwallowsException = 99
```

**Explanation:**
1. `return 1` in `try`: the value 1 is **saved**, `finally` runs (prints), then the method returns 1.
2. `return 2` in `finally` **replaces** the saved value. Compilers/IDEs warn about this.
3. For a primitive, the **value** 10 is saved before `finally`; changing `x` later doesn't affect it.
4. For an object, the **reference** is saved; changes to the object's content in `finally` are visible to the caller.
5. A `return` in `finally` **swallows** the exception from `try` — the caller never knows it happened. **Never return from `finally`.**

### 4.6 Interview Answer — "What is the finally block?"

> "The finally block comes after try or catch, and it contains code that always runs — whether an exception occurs or not, whether it is caught or not, and even if the try or catch block has a return statement. We use it for cleanup, like closing files, database connections or releasing locks. The main cases where finally does not run are when we call `System.exit()`, when the JVM crashes or the process is killed, or if the try block never finishes, like in an infinite loop. One tricky point is that if both try and finally have return statements, the finally's return wins and can even hide an exception, so we should never return from finally. In modern Java, we usually use try-with-resources instead of finally for closing resources."

---

## 5. Useful Methods of Exception Objects

```java
public class ExceptionMethodsDemo {
    public static void main(String[] args) {
        try {
            Object o = "text";
            Integer i = (Integer) o;
        } catch (ClassCastException e) {
            System.out.println("getMessage()        : " + e.getMessage().startsWith("class java.lang.String cannot be cast"));
            System.out.println("toString() starts   : " + e.toString().startsWith("java.lang.ClassCastException: "));
            System.out.println("getClass()          : " + e.getClass().getName());
            System.out.println("getCause()          : " + e.getCause());
            StackTraceElement top = e.getStackTrace()[0];
            System.out.println("thrown in method    : " + top.getMethodName());
            // e.printStackTrace();   // prints full trace to System.err
        }
    }
}
```

**Expected Output:**

```text
getMessage()        : true
toString() starts   : true
getClass()          : java.lang.ClassCastException
getCause()          : null
thrown in method    : main
```

| Method | Returns |
|---|---|
| `getMessage()` | Detail message (can be `null`) |
| `toString()` | `ClassName: message` |
| `printStackTrace()` | Prints type, message and full stack trace to `System.err` |
| `getStackTrace()` | Array of `StackTraceElement` (class, method, file, line) |
| `getCause()` | The wrapped (cause) exception, or `null` |
| `getSuppressed()` | Exceptions suppressed during try-with-resources |
| `getLocalizedMessage()` | Locale-specific message (same as `getMessage()` by default) |

In real projects, use a **logger** (`log.error("Payment failed", e)`) instead of `printStackTrace()`.

---

## 6. Interview Questions with Complete Answers

### Basic Level

**Q1. Can we have a `try` block without `catch`?**

Yes, if it has a `finally` block (`try { } finally { }`), or if it is a try-with-resources (`try (Resource r = ...) { }`). A plain `try { }` alone is a compile error.

**Q2. Can we write code between `try` and `catch`?**

No. The `catch` (or `finally`) must immediately follow the closing brace of `try`. Any statement in between is a compile error.

**Q3. What happens after a catch block executes?**

Execution continues with the first statement **after** the entire try-catch-finally structure (after `finally` runs, if present). It never goes back into the `try` block.

### Intermediate Level

**Q4. Why must the catch block for a subclass come before the superclass?**

Because the JVM chooses the first matching catch. A superclass catch would catch subclass exceptions too, so a later subclass catch could never execute. The compiler detects this unreachable code and reports an error.

**Q5. Does `finally` run if there is a `return` in `try`?**

Yes. The return value is evaluated and saved, then `finally` runs, then the method returns. If `finally` itself has a `return`, it overrides the earlier value.

**Q6. When does `finally` not execute?**

When `System.exit()` is called before it, when the JVM crashes or the process is killed, when the thread is stuck forever (infinite loop/deadlock) inside `try`, or for daemon threads when the JVM shuts down.

**Q7. Can we catch multiple exceptions in one catch block?**

Yes, since Java 7: `catch (IOException | SQLException e)`. The types must not be subclasses of each other, and `e` is implicitly final.

### Advanced Level

**Q8. What happens if an exception is thrown inside a `catch` or `finally` block?**

If thrown in `catch`, the new exception propagates (after `finally` runs). If thrown in `finally`, it **replaces** any exception that was propagating from `try`/`catch` — the original exception is lost. This is one reason try-with-resources is better: it keeps the original exception and attaches closing exceptions as **suppressed** exceptions.

**Q9. What is the output?**

```java
public class TrickyFinally {
    static String test() {
        String s = "A";
        try {
            s = s + "B";
            throw new RuntimeException();
        } catch (RuntimeException e) {
            s = s + "C";
            return s;
        } finally {
            s = s + "D";
            System.out.println("finally sees: " + s);
        }
    }

    public static void main(String[] args) {
        System.out.println("returned: " + test());
    }
}
```

**Expected Output:**

```text
finally sees: ABCD
returned: ABC
```

The catch block's `return s` saves the reference to `"ABC"`. `finally` creates a **new** String `"ABCD"` and assigns it to the local variable `s`, but the saved return value still refers to `"ABC"` (strings are immutable).

**Q10. Is it good practice to catch `Exception` or `Throwable`?**

Generally no. Catching `Exception` hides specific problems and also catches programming bugs (like `NullPointerException`) you would rather fix. Catching `Throwable` also catches `Error`s like `OutOfMemoryError`. Catch the most specific exceptions you can actually handle. A broad catch is acceptable only at the top level (a request handler or `main`) to log and return a generic error.

---

## 7. Follow-Up Questions

**F1. Can `finally` be used without `catch` — why would you?**

Yes: `try { ... } finally { cleanup(); }`. You use it when the current method cannot handle the exception but must still clean up (release a lock) before letting the exception propagate. Example: `lock.lock(); try { ... } finally { lock.unlock(); }`.

**F2. Is an empty catch block acceptable?**

Almost never. It silently "swallows" the exception, hiding failures and making bugs very hard to find. At minimum, log the exception. If ignoring is truly intentional, add a comment explaining why (e.g., `// ignore: optional cleanup`).

**F3. How is `catch (Exception e)` different from multi-catch?**

`catch (Exception e)` catches every subclass of `Exception`, including ones you didn't expect. Multi-catch catches only the listed types, so it's precise while still avoiding duplicate code.

---

## 8. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Superclass catch before subclass catch | Compile error (unreachable catch). |
| `return` inside `finally` | Overrides results and swallows exceptions. |
| Empty catch blocks | Hide errors; at least log. |
| Expecting execution to resume inside `try` after catch | It resumes after the whole structure. |
| Using a variable declared in `try` inside `catch` | Out of scope; declare it before `try`. |
| `catch (IOException \| FileNotFoundException e)` | Related types not allowed in multi-catch. |

---

## 9. Quick Revision

- `try` = risky code; `catch` = handle matching type; first matching catch wins.
- After an exception, rest of `try` is skipped; execution resumes **after** the structure.
- Multiple catches: **specific → general**.
- Multi-catch (Java 7): `catch (A | B e)`; unrelated types; `e` final.
- Nested try: unhandled inner exceptions go to the outer catch.
- `finally` always runs (except `System.exit`, crash, kill, never-ending `try`).
- `return` in try → value saved → finally runs → return. `return` in finally overrides and swallows exceptions — avoid.
- Exception methods: `getMessage`, `toString`, `printStackTrace`, `getStackTrace`, `getCause`, `getSuppressed`.

---

[← Previous: Exceptions and Hierarchy](01-Exceptions-Errors-and-Hierarchy.md) | [Back to README](../README.md) | [Next: throw, throws, Custom Exceptions →](03-throw-throws-Custom-Exceptions-Propagation.md)
