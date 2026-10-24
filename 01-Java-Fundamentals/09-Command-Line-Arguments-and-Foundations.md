# Chapter 1.9 — Command-Line Arguments, Input and Other Foundations

[← Previous: Methods, Overloading and Recursion](08-Methods-Overloading-Recursion.md) | [Back to README](../README.md) | [Next Part: Strings →](../02-Strings-and-Core-Programming/01-String-Basics-Pool-Immutability.md)

---

## What You Will Learn

- Command-line arguments: passing, reading and converting them
- Reading user input with `Scanner` and `BufferedReader`
- Formatted output with `printf` and `String.format`
- Variable scope and blocks (including static and instance initializer blocks)
- Useful `Math` class methods and constants with `final`
- Part 1 summary and readiness checklist

---

## 1. Command-Line Arguments

### 1.1 Interview Definition

> **Command-line arguments are values passed to a Java program when it is started from the command line, written after the class name in the `java` command. The JVM collects these values, stores them as strings in the `String[] args` array, and passes the array to the `main` method. Arguments are separated by spaces, can be grouped using double quotes, and must be converted to other types (for example with `Integer.parseInt`) if numeric values are needed. If no arguments are passed, `args` is an empty array, not null.**

### 1.2 Understand the Concept

```text
java  SumArgs   10   20   "Ravi Kumar"
      ───────   ──   ──   ────────────
     class name args[0] args[1] args[2]
```

- Every argument arrives as a **`String`**, even `10`.
- `args.length` tells how many arguments were passed.
- Quotes group words with spaces into one argument.
- The class name itself is **not** part of `args` (unlike C, where `argv[0]` is the program name).

**Why use them?** To give input to a program **without changing code** — for example, a file name to process, a port number for a server, or a "debug" flag. Scripts and schedulers often run Java programs this way.

### 1.3 Real-World Example

When you order at a food counter, you say: "**Burger, large, no onion**". The cook (the program) is the same every time; your order details (arguments) change its behaviour.

### 1.4 Java Example

<!-- skip-check -->
```java
public class SumArgs {
    public static void main(String[] args) {
        if (args.length == 0) {
            System.out.println("Usage: java SumArgs <num1> <num2> ...");
            return;
        }

        int sum = 0;
        for (String arg : args) {
            try {
                sum += Integer.parseInt(arg);         // String → int
            } catch (NumberFormatException e) {
                System.out.println("Skipping invalid number: " + arg);
            }
        }
        System.out.println("Arguments received: " + args.length);
        System.out.println("Sum = " + sum);
    }
}
```

**Run:**

```text
javac SumArgs.java
java SumArgs 10 20 abc 30
```

**Expected Output:**

```text
Skipping invalid number: abc
Arguments received: 4
Sum = 60
```

Running `java SumArgs` with no arguments prints the usage message.

### 1.5 Code Explanation

- `args.length == 0` checks for missing input — a good habit to avoid `ArrayIndexOutOfBoundsException`.
- `Integer.parseInt(arg)` converts text to `int`. For `"abc"` it throws `NumberFormatException`, which we catch so the program continues.
- In an IDE (IntelliJ/Eclipse), you set arguments in **Run Configuration → Program arguments**.

### 1.6 Interview Answer

> "Command-line arguments are values we pass to a Java program when running it, after the class name, like `java App input.txt 8080`. The JVM puts them into the `String[] args` parameter of the main method. All arguments are received as strings, so if we need numbers we convert them with methods like `Integer.parseInt`. Arguments are separated by spaces, and we use double quotes to pass a value containing spaces. If no arguments are given, `args` is an empty array with length zero, so we should check `args.length` before accessing elements. They are useful for giving configuration or input without changing the code."

---

## 2. Reading User Input

### 2.1 Interview Definition

> **Java reads keyboard input from `System.in`, which is a byte stream. The `Scanner` class (in `java.util`) wraps an input source and provides convenient methods such as `nextInt()`, `nextDouble()`, `next()` and `nextLine()` to read and parse tokens. `BufferedReader` (in `java.io`) wraps an `InputStreamReader` and reads text line by line using `readLine()`; it is faster for large input but returns only strings, which must be parsed manually.**

### 2.2 Scanner Example

<!-- skip-check -->
```java
import java.util.Scanner;

public class ScannerDemo {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        System.out.print("Enter your age: ");
        int age = sc.nextInt();

        sc.nextLine();                          // consume the leftover newline!

        System.out.print("Enter your full name: ");
        String name = sc.nextLine();

        System.out.print("Enter your CGPA: ");
        double cgpa = sc.nextDouble();

        System.out.println("Name: " + name + ", Age: " + age + ", CGPA: " + cgpa);
        sc.close();
    }
}
```

Sample run (user input after the prompts):

```text
Enter your age: 21
Enter your full name: Ravi Kumar
Enter your CGPA: 8.7
Name: Ravi Kumar, Age: 21, CGPA: 8.7
```

### 2.3 The Famous `nextInt()` + `nextLine()` Problem

`nextInt()` reads only the number `21`, leaving the **newline character** (`\n`) from pressing Enter in the input buffer. The next `nextLine()` immediately reads that empty remainder and returns `""`, so the name is skipped.
**Fix:** call an extra `sc.nextLine()` after `nextInt()` (as above), or read every line with `nextLine()` and parse it: `int age = Integer.parseInt(sc.nextLine().trim());`.

| Method | Reads |
|---|---|
| `next()` | One token (stops at whitespace) |
| `nextLine()` | The rest of the current line (including spaces) |
| `nextInt()`, `nextDouble()`, `nextBoolean()` | One token, parsed to the type |
| `hasNextInt()`, `hasNext()` | Checks whether the next token is available/valid |

### 2.4 BufferedReader Example

<!-- skip-check -->
```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class BufferedReaderDemo {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        System.out.print("Enter two numbers separated by space: ");
        String line = br.readLine();                 // reads whole line as String
        String[] parts = line.trim().split("\\s+");  // split on one or more spaces
        int a = Integer.parseInt(parts[0]);
        int b = Integer.parseInt(parts[1]);
        System.out.println("Sum = " + (a + b));
    }
}
```

Sample run:

```text
Enter two numbers separated by space: 12 30
Sum = 42
```

### 2.5 Scanner vs BufferedReader

| Point | `Scanner` | `BufferedReader` |
|---|---|---|
| Package | `java.util` (Java 5) | `java.io` (Java 1.1) |
| Parsing | Built-in (`nextInt`, `nextDouble`) | Manual (`Integer.parseInt`) |
| Speed | Slower (uses regex for parsing) | **Faster** (large 8 KB buffer, no parsing) |
| Buffer size | 1 KB | 8 KB |
| Exceptions | Unchecked (`InputMismatchException`) | Checked `IOException` must be handled |
| Thread-safe | No | Yes (synchronized) |
| Best for | Simple programs, beginners | Competitive programming, large input files |

---

## 3. Formatted Output — `printf` and `String.format`

```java
public class FormatDemo {
    public static void main(String[] args) {
        String name = "Priya";
        int rollNo = 7;
        double percentage = 87.4567;

        System.out.printf("Name: %s, Roll: %d%n", name, rollNo);
        System.out.printf("Percentage: %.2f%%%n", percentage);   // %% prints a % sign
        System.out.printf("|%10s|%-10s|%n", "right", "left");   // width and alignment
        System.out.printf("|%05d|%n", 42);                       // zero padding
        System.out.printf("%,d%n", 1234567);                     // thousands separator
        String report = String.format("%s scored %.1f", name, percentage);
        System.out.println(report);
        System.out.println(String.format("Hex of 255 = %x, char = %c, bool = %b", 255, 'J', true));
    }
}
```

**Expected Output:**

```text
Name: Priya, Roll: 7
Percentage: 87.46%
|     right|left      |
|00042|
1,234,567
Priya scored 87.5
Hex of 255 = ff, char = J, bool = true
```

| Specifier | Meaning |
|---|---|
| `%d` | integer |
| `%f`, `%.2f` | decimal (2 places) — rounds |
| `%s` | string (any object via `toString()`) |
| `%c` | character |
| `%b` | boolean |
| `%x` | hexadecimal |
| `%n` | platform-independent new line |
| `%10s` / `%-10s` | width 10, right / left aligned |
| `%05d` | pad with zeros to width 5 |

---

## 4. Scope and Blocks

### 4.1 Interview Definition

> **The scope of a variable is the region of the program where the variable can be accessed. In Java, scope is decided by blocks, which are groups of statements inside curly braces. A local variable is visible from its declaration to the end of the block in which it is declared; instance and static variables are visible throughout the class (subject to access modifiers). Java does not allow a local variable to be redeclared with the same name inside a nested block within its scope.**

### 4.2 Java Example — Scope, Static Blocks and Instance Initializer Blocks

```java
public class ScopeAndBlocks {
    static int staticCounter;
    int instanceValue;

    static {
        staticCounter = 100;
        System.out.println("1. Static block runs once, when the class is loaded");
    }

    {
        instanceValue = 5;
        System.out.println("3. Instance initializer block runs before the constructor body");
    }

    ScopeAndBlocks() {
        System.out.println("4. Constructor runs, instanceValue = " + instanceValue);
    }

    public static void main(String[] args) {
        System.out.println("2. main starts, staticCounter = " + staticCounter);
        new ScopeAndBlocks();

        int x = 10;                       // scope: rest of main
        {
            int y = 20;                   // scope: only this block
            System.out.println("Inside block: x = " + x + ", y = " + y);
            // int x = 5;                 // compile error: x is already defined in main
        }
        // System.out.println(y);         // compile error: y cannot be found here

        for (int i = 0; i < 2; i++) {
            int square = i * i;           // new 'square' each iteration
            System.out.println("i = " + i + ", square = " + square);
        }
        // System.out.println(i);         // compile error: i exists only inside the loop
        new ScopeAndBlocks();
    }
}
```

**Expected Output:**

```text
1. Static block runs once, when the class is loaded
2. main starts, staticCounter = 100
3. Instance initializer block runs before the constructor body
4. Constructor runs, instanceValue = 5
Inside block: x = 10, y = 20
i = 0, square = 0
i = 1, square = 1
3. Instance initializer block runs before the constructor body
4. Constructor runs, instanceValue = 5
```

### 4.3 Explanation

- The **static block** runs **once** when the class is loaded — before `main`.
- The **instance initializer block** `{ ... }` runs **every time an object is created**, before the constructor body. (It is copied by the compiler into the start of every constructor, after the `super()` call.)
- Local variables like `y` and `i` disappear at the end of their block.
- Java does **not** allow **shadowing a local variable** with another local variable of the same name in a nested block (unlike C). However, a local variable **can** shadow a field — then you use `this.field` to access the field.

| Block | Runs | How many times | Typical use |
|---|---|---|---|
| Static block `static { }` | When the class is loaded | Once per class | Initialise static data, load drivers/config |
| Instance block `{ }` | When an object is created, before constructor body | Once per object | Code shared by all constructors |
| Constructor | When an object is created | Once per object | Initialise the object |

---

## 5. The `Math` Class and Constants

```java
public class MathDemo {
    static final double GST_RATE = 0.18;     // constant: static final, UPPER_SNAKE_CASE

    public static void main(String[] args) {
        System.out.println("max(7, 3) = " + Math.max(7, 3));
        System.out.println("min(7, 3) = " + Math.min(7, 3));
        System.out.println("abs(-9) = " + Math.abs(-9));
        System.out.println("pow(2, 8) = " + Math.pow(2, 8));
        System.out.println("sqrt(144) = " + Math.sqrt(144));
        System.out.println("cbrt(27) = " + Math.cbrt(27));
        System.out.println("ceil(4.1) = " + Math.ceil(4.1));
        System.out.println("floor(4.9) = " + Math.floor(4.9));
        System.out.println("round(4.5) = " + Math.round(4.5));
        System.out.println("round(-4.5) = " + Math.round(-4.5));
        System.out.println("PI = " + Math.PI);
        System.out.println("floorMod(-7, 3) = " + Math.floorMod(-7, 3));
        System.out.println("Price with GST: " + (1000 * (1 + GST_RATE)));
        int dice = (int) (Math.random() * 6) + 1;   // random 1..6
        System.out.println("Dice in range? " + (dice >= 1 && dice <= 6));
    }
}
```

**Expected Output:**

```text
max(7, 3) = 7
min(7, 3) = 3
abs(-9) = 9
pow(2, 8) = 256.0
sqrt(144) = 12.0
cbrt(27) = 3.0
ceil(4.1) = 5.0
floor(4.9) = 4.0
round(4.5) = 5
round(-4.5) = -4
PI = 3.141592653589793
floorMod(-7, 3) = 2
Price with GST: 1180.0
Dice in range? true
```

**Notes:**
- `Math.pow` and `Math.sqrt` return `double`.
- `Math.round(x)` = `floor(x + 0.5)`, so `round(-4.5)` is **−4**, not −5.
- `Math.random()` returns a `double` in [0.0, 1.0). For more control, use `java.util.Random` or `ThreadLocalRandom`.
- `Math` is a `final` class with a `private` constructor — you cannot create a `Math` object; all its methods are `static`.
- A **constant** is declared `static final`, so there is one copy and it cannot change.

---

## 6. Interview Questions with Complete Answers

**Q1. What is the data type of command-line arguments? How do you pass a number?**

They are always `String`s, stored in `String[] args`. To use a number, parse it: `int n = Integer.parseInt(args[0]);`. If the text is not a valid number, `NumberFormatException` is thrown.

**Q2. What happens if you access `args[0]` without passing arguments?**

`args` is an empty array, so `args[0]` throws `ArrayIndexOutOfBoundsException: Index 0 out of bounds for length 0`. Always check `args.length` first.

**Q3. Why does `nextLine()` get skipped after `nextInt()`?**

`nextInt()` reads only the numeric token and leaves the newline character in the buffer. The next `nextLine()` reads the rest of that line, which is empty, and returns immediately. Fix it by calling an extra `nextLine()` after `nextInt()` or by reading whole lines and parsing them.

**Q4. Which is faster, `Scanner` or `BufferedReader`? Why?**

`BufferedReader` is faster. It has a bigger buffer (8 KB vs 1 KB) and it only reads characters/lines without parsing. `Scanner` uses regular expressions to split and parse tokens, which adds overhead. For large inputs (like competitive programming), `BufferedReader` is preferred.

**Q5. What is the difference between a static block and an instance initializer block?**

A static block runs **once** when the class is loaded and can access only static members. An instance initializer block runs **every time an object is created**, before the constructor body, and can access both instance and static members. Static blocks are used for one-time setup like loading configuration; instance blocks are used for code common to all constructors.

**Q6. Can we have multiple static blocks? In which order do they run?**

Yes. Multiple static blocks are allowed, and they execute in the **order they appear** in the class (top to bottom), together with static variable initialisers in textual order.

**Q7. Why does `Math.round(-4.5)` return −4?**

`Math.round` is defined as `floor(x + 0.5)`. For −4.5: −4.5 + 0.5 = −4.0, and `floor(−4.0) = −4`. So halves are rounded **towards positive infinity**. For "round half away from zero" or banker's rounding, use `BigDecimal` with a `RoundingMode`.

**Q8. Can you create an object of the `Math` class?**

No. `Math` has a **private constructor**, so it cannot be instantiated, and it is `final`, so it cannot be extended. It is a utility class — all methods and constants are `static`.

---

## 7. Follow-Up Questions

**F1. How do you pass an argument that contains spaces?**

Wrap it in double quotes: `java App "New Delhi"`. Then `args[0]` is `New Delhi`.

**F2. Should you close a `Scanner` that wraps `System.in`?**

Closing the `Scanner` also closes `System.in`, and it cannot be reopened in the same program. In small programs, close it once at the very end. In larger programs, create one `Scanner` for `System.in` and reuse it everywhere.

**F3. What is the difference between `System.out.println` and `System.err.println`?**

`System.out` is standard output; `System.err` is standard error. They are separate streams, so their output can be redirected separately (`java App > out.txt 2> err.txt`), and they may appear in a mixed order on the console because they are flushed independently.

---

## 8. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Expecting `args[0]` to be the class name | `args` contains only the arguments. |
| Using `args[0]` without checking length | Throws `ArrayIndexOutOfBoundsException`. |
| Forgetting to consume the newline after `nextInt()` | Next `nextLine()` returns empty. |
| `Math.pow(2, 3)` expected to be `int` | Returns `double` (8.0). |
| Declaring a constant without `static` | Each object would get its own copy; use `static final`. |
| Redeclaring a local variable in an inner block | Compile error in Java. |

---

## 9. Part 1 Summary — Are You Ready?

You should now be able to answer these without looking:

- [ ] What is Java? Why is it platform independent? Is the JVM platform independent?
- [ ] JDK vs JRE vs JVM, with a diagram.
- [ ] What happens when you compile and run a Java program?
- [ ] Explain every word of `public static void main(String[] args)`.
- [ ] 8 primitive types with sizes; why `char` is 2 bytes.
- [ ] Local vs instance vs static variables; default values.
- [ ] Widening vs narrowing; why `byte + byte` is `int`.
- [ ] `i++` vs `++i`; `&&` vs `&`; `>>` vs `>>>`.
- [ ] `switch` rules, fall-through and switch expressions.
- [ ] `while` vs `do-while`; `break` vs `continue`; labelled break.
- [ ] Method overloading rules and resolution order.
- [ ] Recursion: base case, stack frames, `StackOverflowError`.
- [ ] Command-line arguments and `Scanner` vs `BufferedReader`.

---

[← Previous: Methods, Overloading and Recursion](08-Methods-Overloading-Recursion.md) | [Back to README](../README.md) | [Next Part: Strings →](../02-Strings-and-Core-Programming/01-String-Basics-Pool-Immutability.md)
