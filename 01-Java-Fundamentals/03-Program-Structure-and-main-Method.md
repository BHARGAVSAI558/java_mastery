# Chapter 1.3 — Java Program Structure and the `main` Method

[← Previous: JDK, JRE, JVM](02-JDK-JRE-JVM-Compilation-Bytecode.md) | [Back to README](../README.md) | [Next: Variables and Data Types →](04-Variables-and-Data-Types.md)

---

## What You Will Learn

- The parts of a Java source file and their correct order
- File naming rules
- Every keyword in `public static void main(String[] args)` and why it is there
- Valid and invalid variations of `main`
- Comments, identifiers, keywords and naming conventions
- Many tricky interview questions about `main`

---

## 1. Structure of a Java Program

### 1.1 Interview Definition

> **A Java source file is a `.java` file that can contain an optional package declaration, followed by optional import statements, followed by one or more type declarations (classes, interfaces, enums or records). Only one top-level type in a file can be `public`, and if a public type exists, the file name must exactly match its name. A standalone Java application starts executing from the `public static void main(String[] args)` method of the class that is passed to the `java` command.**

### 1.2 Understand the Concept

The parts of a source file must appear in this **order**:

```text
1. Package declaration      (optional, at most one, must be the first statement)
2. Import statements        (optional, any number)
3. Type declarations        (class / interface / enum / record — one or more)
     ├── Fields (variables)
     ├── Constructors
     ├── Methods (including main)
     ├── Blocks (static / instance)
     └── Nested types
```

Comments can appear anywhere.

| Section | Example | Purpose |
|---|---|---|
| Package | `package com.college.app;` | Groups related classes, avoids name conflicts |
| Import | `import java.util.Scanner;` | Lets you use classes from other packages by short name |
| Class | `public class Student { }` | The blueprint that holds code |
| Fields | `String name;` | Data of the class |
| Methods | `void study() { }` | Behaviour of the class |
| `main` | `public static void main(String[] args)` | Entry point of the program |

**File naming rules**
1. If the file has a `public` class `Student`, the file **must** be named `Student.java` (case-sensitive).
2. A file can have **at most one** `public` top-level class.
3. A file may have **no** public class; then it can have any name (but that is bad practice).
4. A file can have many non-public classes.

### 1.3 Real-World Example

A Java file is like a **formal letter**:
- *Package* = the sender's address at the top (where this letter belongs).
- *Imports* = "References/Enclosures" (things borrowed from elsewhere).
- *Class* = the body of the letter.
- *main method* = the first line the reader starts from.

### 1.4 Java Example — A Complete Structured Program

<!-- skip-check -->
```java
// 1. Package declaration (optional)
package com.college.demo;

// 2. Import statements (optional)
import java.util.ArrayList;
import java.util.List;

/**
 * 3. Class declaration.
 * This is a documentation (Javadoc) comment.
 */
public class StudentApp {

    // Field (instance variable)
    private String collegeName = "ABC Engineering College";

    // Static field (class variable)
    static int totalStudents = 0;

    // Static block - runs once when the class is loaded
    static {
        System.out.println("StudentApp class loaded");
    }

    // Constructor
    public StudentApp() {
        System.out.println("Object created for " + collegeName);
    }

    // Instance method
    void addStudent(List<String> list, String name) {
        list.add(name);
        totalStudents++;
    }

    // main method - entry point
    public static void main(String[] args) {
        StudentApp app = new StudentApp();
        List<String> students = new ArrayList<>();
        app.addStudent(students, "Ravi");
        app.addStudent(students, "Priya");
        System.out.println("Students: " + students);
        System.out.println("Total: " + totalStudents);
    }
}
```

Because this class is in package `com.college.demo`, it must be saved as `com/college/demo/StudentApp.java`, compiled and run from the folder that *contains* `com`:

```text
javac com/college/demo/StudentApp.java
java com.college.demo.StudentApp
```

### 1.5 Code Explanation

- `package com.college.demo;` must be the **first statement** in the file (only comments may come before it).
- `import java.util.ArrayList;` lets us write `ArrayList` instead of `java.util.ArrayList`.
- `java.lang` is **imported automatically**, so `String` and `System` need no import.
- The static block runs once when the JVM initialises the class — before `main` starts.
- The constructor runs when `new StudentApp()` is executed.
- `addStudent` is an instance method, so it must be called using an object (`app.addStudent(...)`).
- `totalStudents` is `static`, so it can be used directly inside the static `main` method.

### 1.6 Expected Output

```text
StudentApp class loaded
Object created for ABC Engineering College
Students: [Ravi, Priya]
Total: 2
```

The static block prints first (class loading), then the constructor message, then `main`'s output. `ArrayList.toString()` prints elements inside square brackets.

---

## 2. The `main` Method

### 2.1 Interview Definition

> **The `main` method is the entry point of a standalone Java application. Its standard signature is `public static void main(String[] args)`. It is `public` so the JVM can access it from outside the class, `static` so the JVM can call it without creating an object, `void` because it does not return any value to the JVM, named `main` because the JVM looks for exactly this name, and it takes a `String[]` parameter to receive command-line arguments.**

### 2.2 Understand Every Keyword

| Part | Why it is needed | What happens if it is changed |
|---|---|---|
| `public` | The JVM is outside your class. It must be able to access the method. | If `private`/default/`protected`: compiles fine, but at runtime: *"Error: Main method not found in class..."* |
| `static` | The JVM calls `main` **before any object exists**. A static method belongs to the class, so no object is required. | If non-static: compiles, but runtime error (in Java 21+, see the note below). |
| `void` | `main` returns nothing to the JVM. To return an exit code, use `System.exit(code)`. | If `int main`: compiles, but runtime error "main method must return a value of type void". |
| `main` | The JVM is hard-coded to look for this exact name. | `Main` or `mian` is just a normal method; JVM reports main not found. |
| `String[] args` | Receives **command-line arguments** as strings. | Other parameter types make it a normal overloaded method. |

**Valid variations** (all accepted by the JVM):

```text
public static void main(String[] args)
public static void main(String args[])        // C-style array declaration
public static void main(String... args)       // varargs (Java 5+)
static public void main(String[] args)        // modifier order does not matter
public static final void main(String[] args)  // final is allowed
public static synchronized void main(String[] args)
public static void main(String[] myInputs)    // parameter name can be anything
```

**Invalid as an entry point** (they compile, but `java` refuses to start them — it prints an error such as *"Main method not found"* or *"Main method is not static"*):

```text
public void main(String[] args)          // not static (before Java 21 rules)
static void main(String[] args)          // not public (before Java 21 rules)
public static void main(String args)     // not an array
public static int main(String[] args)    // not void
```

> **Java 21+ note — "Unnamed classes and instance main methods" (JEP 445, preview in 21; finalized as "compact source files and instance main methods" in Java 25).** Newer Java versions relax these rules for beginners: `void main() { ... }` can work without `public`, `static` or parameters. Interviewers almost always expect the classic signature, so use `public static void main(String[] args)` in interviews, and mention this only as an extra point.

### 2.3 Real-World Example

The `main` method is like the **main entrance door of a shopping mall**. Security (the JVM) always enters through the main door with a known name and location. It must be *public* (anyone can enter), *static* (the door exists even before any shop opens), and it does not *return* anything to security.

### 2.4 Java Example — Overloading `main`

```java
public class MainOverload {

    // The JVM calls only this one
    public static void main(String[] args) {
        System.out.println("Standard main called");
        main(5);                     // we can call other overloaded versions ourselves
        main("Java", "Interview");
    }

    public static void main(int number) {
        System.out.println("main(int) called with " + number);
    }

    public static void main(String first, String second) {
        System.out.println("main(String, String) called with " + first + " and " + second);
    }
}
```

### 2.5 Code Explanation

- `main` can be **overloaded** like any other method — same name, different parameters.
- The JVM **always** starts with the version that takes `String[]`.
- The other versions are normal methods; they run only when we call them.

### 2.6 Expected Output

```text
Standard main called
main(int) called with 5
main(String, String) called with Java and Interview
```

### 2.7 Java Example — Using `args`

<!-- skip-check -->
```java
public class Greet {
    public static void main(String[] args) {
        System.out.println("Number of arguments: " + args.length);
        for (int i = 0; i < args.length; i++) {
            System.out.println("args[" + i + "] = " + args[i]);
        }
    }
}
```

Run with: `java Greet Ravi 21 CSE`

```text
Number of arguments: 3
args[0] = Ravi
args[1] = 21
args[2] = CSE
```

If run without arguments (`java Greet`), the output is `Number of arguments: 0`. Note that `args` is **never null** when the JVM calls `main` — it is an empty array. Command-line arguments are covered fully in [Chapter 1.9](09-Command-Line-Arguments-and-Foundations.md).

### 2.8 Interview Answer — "Explain `public static void main(String[] args)`"

> "`main` is the entry point of a Java application — the JVM starts execution from this method. It is **public** so that the JVM, which is outside the class, can access it. It is **static** so the JVM can call it without creating an object of the class, because at startup no object exists yet. It is **void** because it does not return a value to the JVM; if we need to return a status code, we use `System.exit()`. The name must be exactly **main**, because the JVM looks for that name. And `String[] args` receives command-line arguments as an array of strings. We can also write it as `String... args`, change the parameter name, or change the order of `public` and `static`. We can overload `main`, but the JVM always calls the version with the `String` array."

---

## 3. Comments, Identifiers, Keywords and Naming Conventions

### 3.1 Comments

```text
// Single-line comment

/* Multi-line comment
   spanning several lines */

/**
 * Documentation (Javadoc) comment.
 * @param name the student's name
 * @return a greeting
 */
```

- Comments are ignored by the compiler.
- Javadoc comments are read by the `javadoc` tool to produce HTML documentation.

### 3.2 Identifiers

An **identifier** is the name of a class, method, variable, package or label.

Rules:
1. Can contain letters, digits, `_` and `$`.
2. **Cannot start with a digit** (`1name` is invalid).
3. Cannot be a **keyword** (`int`, `class`).
4. Case-sensitive (`total` and `Total` are different).
5. No length limit.
6. `_` alone is not allowed as an identifier since Java 9 (it became a keyword; Java 22 uses it for unnamed variables).

| Identifier | Valid? | Reason |
|---|---|---|
| `studentName` | ✅ | |
| `_count` | ✅ | |
| `$price` | ✅ | (legal, but avoid `$` — it is used by generated code) |
| `2ndValue` | ❌ | starts with a digit |
| `total-marks` | ❌ | `-` not allowed |
| `class` | ❌ | keyword |
| `Class` | ✅ | different case from the keyword (but confusing) |

### 3.3 Keywords

Java has around **50 reserved keywords** (plus some contextual ones). Important groups:

| Group | Keywords |
|---|---|
| Primitive types | `byte short int long float double char boolean` |
| Control flow | `if else switch case default for while do break continue return` |
| Class related | `class interface enum extends implements new this super instanceof` |
| Access modifiers | `public protected private` |
| Other modifiers | `static final abstract synchronized volatile transient native strictfp` |
| Exceptions | `try catch finally throw throws` |
| Package | `package import` |
| Others | `void assert` |
| Reserved but unused | `goto const` |

**Literals, not keywords:** `true`, `false`, `null`.
**Contextual keywords** (keywords only in certain places): `var` (Java 10), `record` (16), `sealed`, `permits`, `non-sealed` (17), `yield` (14), `module`, `exports`, `requires` (9).

### 3.4 Naming Conventions (Interviewers notice these)

| Element | Convention | Example |
|---|---|---|
| Class / interface / enum / record | PascalCase (UpperCamelCase), nouns | `StudentRecord`, `Runnable` |
| Method | camelCase, verbs | `calculateTotal()`, `getName()` |
| Variable | camelCase | `totalMarks`, `firstName` |
| Constant (`static final`) | UPPER_SNAKE_CASE | `MAX_SIZE`, `PI` |
| Package | all lowercase, reverse domain | `com.google.common` |
| Generic type parameter | single capital letter | `T`, `E`, `K`, `V` |

---

## 4. Interview Questions with Complete Answers

### Basic Level

**Q1. Why is the `main` method `static`?**

Because the JVM must call `main` **before any object of the class exists**. Static methods belong to the class itself, so the JVM can call `ClassName.main(args)` directly. If `main` were an instance method, the JVM would first need to create an object, and it would not know which constructor to call or what arguments to pass.

**Q2. Can we write a Java program without a `main` method?**

- A **class** can be compiled without `main` — many classes (helpers, models) have no `main`.
- But to **run** a class as an application with `java ClassName`, it needs a `main` method.
- Before Java 7, a trick using a `static` block could print output without `main`. **Since Java 7, the JVM checks for `main` first**, so this trick no longer works: you get *"Error: Main method not found in class"*.
- Frameworks such as servlets or JUnit tests do not need your own `main` because the framework has its own entry point.

**Q3. Can we overload the `main` method?**

Yes. `main` can be overloaded with different parameter lists. However, the JVM always calls only `public static void main(String[] args)`. Other versions run only if called from code.

**Q4. Can we have more than one public class in a single file?**

No. A source file can have **at most one public top-level class**, and the file name must match it. You can have many non-public classes in the same file. (Nested classes can be public, because they are not top-level.)

### Intermediate Level

**Q5. Can we override the `main` method?**

No. `main` is `static`, and **static methods cannot be overridden** — they are resolved at compile time based on the class name. If a subclass declares the same static `main`, it **hides** the parent's `main` (method hiding), it does not override it.

**Q6. What happens if `main` is declared `private`?**

The program compiles successfully, because `private static void main(String[] args)` is a legal method. But when you run it with the classic launcher rules, the JVM cannot find a public `main` and prints:
`Error: Main method not found in class X, please define the main method as: public static void main(String[] args)`.
(From Java 25, the launch protocol is more relaxed and a non-private `main` can be chosen; but a `private` one is still not used as an entry point.)

**Q7. Can `main` be declared `final`, `synchronized` or `strictfp`?**

Yes. These modifiers are allowed and do not change how the JVM finds `main`. `final` means subclasses cannot hide it, `synchronized` means it locks the class object while running. None of them are commonly used.

**Q8. Can the `main` method throw an exception?**

Yes. `public static void main(String[] args) throws Exception` is valid. If an exception escapes `main`, the JVM's default exception handler prints the stack trace, and the program terminates with a non-zero exit code.

**Q9. What is the default value of `args` if no arguments are passed?**

`args` is an **empty array** of length 0, not `null`. So `args.length` is 0, and accessing `args[0]` throws `ArrayIndexOutOfBoundsException`.

### Advanced Level

**Q10. Is it necessary that the file name and class name are the same?**

Only if the class is `public`. If the class has default (package-private) access, the file name can be anything. For example, `class Test` saved in `Abc.java` compiles to `Test.class` and is run with `java Test`. However, matching names is the standard practice.

**Q11. What is the order in which things execute when a class with `main` is run?**

1. Class loading.
2. Static variables get default values, then their initial values, and **static blocks run in the order they appear**.
3. `main` is called.
4. Inside `main`, when an object is created: instance variables are initialised and **instance initializer blocks** run, then the **constructor body** runs.

**Q12. Can `main` be in an interface, enum or record?**

Yes. Since Java 8, interfaces can have static methods, so an interface can have `public static void main(String[] args)` and can be run. Enums and records can also have a `main` method.

---

## 5. Follow-Up Questions

**F1. How do you return an exit code from a Java program if `main` is void?**

Use `System.exit(status)`. By convention, `0` means success and any non-zero value means an error. Scripts and build tools check this exit code.

**F2. What is the difference between `System.out.println` and `System.out.print`?**

`println` prints the value and then a line separator (new line). `print` prints the value without moving to a new line. `printf` prints using a format string, for example `System.out.printf("%.2f%n", 3.14159)` prints `3.14`.

**F3. What is `System.out` exactly?**

`System` is a final class in `java.lang`. `out` is a `public static final PrintStream` field of `System`, connected to standard output (the console). `println` is an instance method of `PrintStream`. Similarly, `System.err` is for error output and `System.in` is an `InputStream` for keyboard input.

**F4. Why is `java.lang` imported automatically?**

Because it contains the classes almost every program needs: `Object`, `String`, `System`, `Math`, wrapper classes (`Integer`, etc.), `Thread`, and the basic exceptions. Importing it automatically saves effort.

---

## 6. Common Mistakes

| Mistake | Result / Correct way |
|---|---|
| File name `student.java` with `public class Student` | Compile error. Names are case-sensitive. |
| Writing `String args[]` vs `String[] args` and thinking one is wrong | Both are valid. |
| Thinking `main` cannot be overloaded | It can; the JVM just ignores the overloads. |
| Thinking `args` is `null` when no arguments are passed | It is an empty array. |
| Putting `import` before `package` | Compile error; `package` must be first. |
| Expecting the static-block trick to run without `main` | Does not work since Java 7. |

---

## 7. Quick Revision

- File order: **package → imports → types**.
- Only **one public top-level class** per file; file name must match it.
- `public` (JVM access) · `static` (no object needed) · `void` (no return) · `main` (fixed name) · `String[] args` (command-line arguments).
- Valid: `String args[]`, `String... args`, `static public`, `final`, `synchronized`.
- `main` can be **overloaded**, cannot be **overridden** (static → hiding).
- No arguments → `args.length == 0` (not null).
- Exit code → `System.exit(n)`.
- `java.lang` is imported automatically.
- Naming: `ClassName`, `methodName`, `variableName`, `CONSTANT_NAME`, `package.name`.

---

[← Previous: JDK, JRE, JVM](02-JDK-JRE-JVM-Compilation-Bytecode.md) | [Back to README](../README.md) | [Next: Variables and Data Types →](04-Variables-and-Data-Types.md)
