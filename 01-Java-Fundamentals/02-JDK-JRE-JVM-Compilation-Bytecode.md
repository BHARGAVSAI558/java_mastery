# Chapter 1.2 — JDK, JRE, JVM, Compilation, Execution and Bytecode

[← Previous: Introduction](01-Introduction-Features-Platform-Independence.md) | [Back to README](../README.md) | [Next: Program Structure →](03-Program-Structure-and-main-Method.md)

---

## What You Will Learn

- What the JVM, JRE and JDK are, and how they are related
- The full journey of a Java program: from `.java` to output
- What bytecode looks like and how to view it
- The role of the JIT compiler
- The most frequently asked comparison: **JDK vs JRE vs JVM**

---

## 1. JVM — Java Virtual Machine

### 1.1 Interview Definition

> **The JVM (Java Virtual Machine) is an abstract machine that provides the runtime environment in which Java bytecode is executed. It loads `.class` files, verifies the bytecode for safety, executes it using an interpreter and a JIT (Just-In-Time) compiler, and manages memory through garbage collection. The JVM is platform dependent — each operating system has its own JVM implementation — but all JVMs follow the same JVM specification, which is why the same bytecode runs everywhere.**

### 1.2 Understand the Concept

The word "virtual machine" means it is **a computer made of software**. A real CPU understands only its own machine instructions. The JVM is a software "CPU" that understands **bytecode instructions**. When you run `java MyProgram`, the operating system starts the JVM, and the JVM runs your bytecode.

The JVM has three main jobs:

1. **Load** — The *class loader* finds and loads `.class` files into memory.
2. **Verify** — The *bytecode verifier* checks that the bytecode is valid and safe (no illegal memory access, correct types, no stack overflow tricks).
3. **Execute** — The *execution engine* runs the bytecode. It contains:
   - an **interpreter**, which reads and executes bytecode one instruction at a time, and
   - a **JIT compiler**, which compiles frequently used code into native machine code.

It also provides:
- **Memory management** — heap, stack, method area (metaspace)
- **Garbage collection** — automatically removes objects that are no longer used

There are three ways the word "JVM" is used:

| Meaning | Explanation |
|---|---|
| **Specification** | A document by Oracle describing how any JVM must behave. |
| **Implementation** | An actual program following the specification, for example **HotSpot** (the most common, from Oracle/OpenJDK), OpenJ9 (Eclipse), GraalVM. |
| **Runtime instance** | A running JVM process created when you type `java MyClass`. Each `java` command creates a new instance. |

> JVM architecture (class loader subsystem, memory areas, execution engine) is covered in depth in [Part 9](../09-JVM-and-Memory/01-JVM-Architecture-and-Execution.md).

### 1.3 Real-World Example

The JVM is like a **game console emulator** on a PC. A PlayStation game (bytecode) cannot run directly on Windows, but an emulator (JVM) can read the game and run it on Windows. A different emulator for macOS can run the same game on a Mac.

---

## 2. JRE — Java Runtime Environment

### 2.1 Interview Definition

> **The JRE (Java Runtime Environment) is the software package required to run Java applications. It contains the JVM along with the core class libraries (such as `java.lang`, `java.util`, `java.io`) and other supporting files. The JRE can run Java programs, but it does not include development tools like the compiler (`javac`), so it cannot be used to develop or compile Java code.**

### 2.2 Understand the Concept

The JVM alone is not enough to run real programs. Your program uses classes like `String`, `ArrayList`, `System` and `Scanner`. These classes are part of the **Java class library**. The JRE packages the JVM together with these libraries.

```text
JRE = JVM + Core Libraries (java.lang, java.util, java.io, ...) + supporting files
```

**Important modern note:** Since **Java 11**, Oracle no longer provides a separate JRE download. You install the JDK, which contains everything. Developers can create a custom, smaller runtime using the **`jlink`** tool. Interviewers still ask about the JRE as a concept, so you must know it.

### 2.3 Real-World Example

If the JVM is the **engine** of a car, the JRE is the **complete car ready to drive** — engine plus wheels, seats and fuel system. You can drive it (run programs), but you cannot build new car parts with it.

---

## 3. JDK — Java Development Kit

### 3.1 Interview Definition

> **The JDK (Java Development Kit) is a complete software development kit used to develop, compile, debug and run Java applications. It contains the JRE (which includes the JVM and core libraries) plus development tools such as the compiler `javac`, the launcher `java`, the archiver `jar`, the documentation generator `javadoc`, the debugger `jdb`, and the disassembler `javap`. A developer needs the JDK; a user who only runs Java programs needs a runtime.**

### 3.2 Understand the Concept

```text
JDK = JRE + Development Tools
    = (JVM + Libraries) + (javac, java, jar, javadoc, javap, jdb, jshell, jlink, ...)
```

Common JDK tools:

| Tool | Purpose |
|---|---|
| `javac` | Compiles `.java` source into `.class` bytecode |
| `java` | Starts the JVM and runs a class |
| `jar` | Packages many `.class` files into one `.jar` archive |
| `javadoc` | Generates HTML documentation from `/** ... */` comments |
| `javap` | Disassembles `.class` files (shows bytecode) |
| `jdb` | Command-line debugger |
| `jshell` | Interactive Java shell (Java 9+) — type Java code and see results immediately |
| `jlink` | Builds a custom runtime image (Java 9+) |
| `jconsole`, `jvisualvm`, `jcmd` | Monitoring and diagnostics tools |

Popular JDK distributions: **Oracle JDK, OpenJDK, Eclipse Temurin (Adoptium), Amazon Corretto, Azul Zulu, Microsoft Build of OpenJDK**. They are all based on OpenJDK source code.

### 3.3 Real-World Example

The JDK is a **car factory with a showroom**. It has the complete car (JRE) plus all the machines and tools (compiler, debugger) needed to build and test new cars.

---

## 4. JDK vs JRE vs JVM — The Big Comparison

### 4.1 Diagram

```text
┌───────────────────────────────────────────────────────┐
│                         JDK                           │
│  Development tools: javac, jar, javadoc, javap, jdb   │
│  ┌─────────────────────────────────────────────────┐  │
│  │                      JRE                        │  │
│  │  Class libraries: java.lang, java.util, ...     │  │
│  │  ┌───────────────────────────────────────────┐  │  │
│  │  │                   JVM                     │  │  │
│  │  │  Class Loader | Memory Areas | Execution  │  │  │
│  │  │  Engine (Interpreter + JIT) | GC          │  │  │
│  │  └───────────────────────────────────────────┘  │  │
│  └─────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────┘
```

### 4.2 Comparison Table

| Point | JVM | JRE | JDK |
|---|---|---|---|
| Full form | Java Virtual Machine | Java Runtime Environment | Java Development Kit |
| What it is | Abstract machine that executes bytecode | Environment to run Java programs | Kit to develop and run Java programs |
| Contains | Class loader, memory areas, execution engine, GC | JVM + core libraries | JRE + development tools |
| Can compile code? | No | No | **Yes** (`javac`) |
| Can run code? | Yes (executes bytecode) | Yes | Yes |
| Who needs it? | Part of every runtime | End users (historically) | Developers |
| Platform dependent? | Yes | Yes | Yes |

### 4.3 Interview Answer — "Explain JDK, JRE and JVM"

> "JVM, JRE and JDK are three layers, one inside the other. The **JVM** is the Java Virtual Machine — it is the engine that actually runs bytecode. It loads class files, verifies them, executes them using an interpreter and JIT compiler, and manages memory with garbage collection. The **JRE**, Java Runtime Environment, is the JVM plus the core class libraries like `java.lang` and `java.util`. It is everything needed to *run* a Java program. The **JDK**, Java Development Kit, is the JRE plus development tools like `javac`, `jar`, `javadoc` and the debugger. It is what developers need to *write and compile* programs. So, JDK contains JRE, and JRE contains JVM. All three are platform dependent, but the bytecode they work with is platform independent. Since Java 11, a separate JRE is not distributed — we install the JDK."

---

## 5. Compilation and Execution — The Journey of a Java Program

### 5.1 Interview Definition

> **Java program execution happens in two phases. In the compilation phase, the `javac` compiler checks the source code for syntax and type errors and converts the `.java` file into platform-independent bytecode stored in a `.class` file. In the execution phase, the `java` command starts the JVM; the class loader loads the class, the bytecode verifier checks it, and the execution engine runs it using the interpreter and the JIT compiler, which converts frequently executed bytecode into native machine code.**

### 5.2 Step-by-Step Process

```text
 ┌───────────────┐   javac    ┌───────────────┐   java     ┌──────────────────────────────┐
 │ Welcome.java  │──────────► │ Welcome.class │──────────► │             JVM              │
 │ (source code) │  compile   │  (bytecode)   │   run      │ 1. Class Loader loads class  │
 └───────────────┘            └───────────────┘            │ 2. Bytecode Verifier checks  │
                                                           │ 3. Interpreter executes      │
                                                           │ 4. JIT compiles hot code     │
                                                           │ 5. Native code runs on CPU   │
                                                           └──────────────────────────────┘
```

**Phase 1 — Compile time (`javac Welcome.java`)**
1. The compiler reads the source file.
2. It checks **syntax** (missing semicolons, brackets) and **types** (assigning a `String` to an `int`).
3. If there are errors, it prints them and **no `.class` file** is produced.
4. If successful, it creates one `.class` file **for each class** in the source file (not per file).

**Phase 2 — Runtime (`java Welcome`)**
1. **Class loading:** The JVM's class loader finds `Welcome.class` and loads it into memory.
2. **Linking:** The **verifier** checks the bytecode is safe. Memory is prepared for static variables (default values). Symbolic references are resolved.
3. **Initialization:** Static variables are given their actual values and `static` blocks run.
4. **Execution:** The JVM calls `main()`. The **interpreter** executes bytecode. The JVM counts how often methods and loops run; when a method becomes "hot", the **JIT compiler** compiles it to native code and caches it, so later calls run much faster.
5. **Garbage collection** runs in the background to free unused objects.
6. When `main` finishes (and no non-daemon threads remain), the JVM shuts down.

### 5.3 Real-World Example

Think of **publishing a book in many countries**:
- The author writes the book (source code).
- An editor checks grammar and spelling and creates a final master copy in a standard format (compiler → bytecode).
- In each country, a local reader reads the master copy aloud in the local language (JVM interpreter). If a chapter is read very often, the reader writes a translated copy of that chapter and uses it next time instead of translating again (JIT compiler).

### 5.4 Java Example — Compile and Run

```java
public class Welcome {
    static int counter = 10;          // static variable, initialised during class initialization

    static {
        System.out.println("Static block: class Welcome is being initialized");
    }

    public static void main(String[] args) {
        System.out.println("main() started");
        Helper helper = new Helper();
        System.out.println("Counter value: " + counter);
        System.out.println("Square of 7 is " + helper.square(7));
    }
}

class Helper {
    int square(int n) {
        return n * n;
    }
}
```

Commands:

```text
javac Welcome.java     → creates Welcome.class and Helper.class
java Welcome           → runs the program
```

### 5.5 Code Explanation

- The file contains **two classes**, so `javac` produces **two** `.class` files: `Welcome.class` and `Helper.class`. This proves `.class` files are created per class, not per source file.
- `static int counter = 10;` and the `static { ... }` block run during the **initialization** step, which happens **before** `main()` is called.
- `new Helper()` causes the JVM to load `Helper.class` only at this moment — this is **lazy (dynamic) class loading**.
- `helper.square(7)` calls an instance method and returns 49.

### 5.6 Expected Output

```text
Static block: class Welcome is being initialized
main() started
Counter value: 10
Square of 7 is 49
```

The static block prints first because the class is initialized before `main()` runs. After that, `main()` prints its lines in order.

### 5.7 Interview Answer — "What happens when you compile and run a Java program?"

> "When I run `javac`, the compiler checks my source code for syntax and type errors and converts it into bytecode, which is stored in a `.class` file — one file for every class. Then, when I run the `java` command, the JVM starts. The class loader loads the main class, the bytecode verifier checks that the code is valid and safe, static variables are initialized and static blocks are executed. Then the JVM calls the `main` method. The execution engine runs the bytecode using an interpreter, and the JIT compiler converts frequently used methods into native machine code to make them faster. While the program runs, the garbage collector frees memory used by objects that are no longer needed."

---

## 6. Bytecode

### 6.1 Interview Definition

> **Bytecode is the intermediate, platform-independent instruction set generated by the Java compiler and stored in `.class` files. It is not machine code for any real processor; it is designed to be executed by the JVM. Each instruction has a one-byte opcode (for example `iload`, `iadd`, `invokevirtual`), which is why it is called bytecode. Bytecode is what makes Java "Write Once, Run Anywhere".**

### 6.2 Viewing Real Bytecode

You can view bytecode with the JDK tool `javap -c`.

```java
public class AddDemo {
    public static void main(String[] args) {
        int a = 5;
        int b = 7;
        int sum = a + b;
        System.out.println(sum);
    }
}
```

**Expected Output:**

```text
12
```

Now run `javap -c AddDemo` after compiling. The important part of the result looks like this:

```text
public static void main(java.lang.String[]);
  Code:
     0: iconst_5          // push constant 5 on the operand stack
     1: istore_1          // store it in local variable 1 (a)
     2: bipush        7   // push 7
     4: istore_2          // store in local variable 2 (b)
     5: iload_1           // load a
     6: iload_2           // load b
     7: iadd              // add the top two values
     8: istore_3          // store result in local variable 3 (sum)
     9: getstatic     #7  // get System.out
    12: iload_3           // load sum
    13: invokevirtual #13 // call println(int)
    16: return
```

### 6.3 Explanation

- The JVM is a **stack-based machine**. Values are pushed onto an *operand stack*, and instructions like `iadd` pop values, compute, and push the result.
- `i` at the start of an instruction means **int** (`iload`, `iadd`, `istore`). There are similar instructions for long (`l`), float (`f`), double (`d`) and references (`a`).
- Local variable 0 is `args`; then `a`, `b`, `sum` are slots 1, 2, 3.
- `invokevirtual` calls an instance method (`println`).

You **do not** need to memorise bytecode instructions for interviews. You need to know: what bytecode is, that it is stack-based, platform independent, and viewable with `javap`.

---

## 7. JIT Compiler (Introduction)

### 7.1 Interview Definition

> **The JIT (Just-In-Time) compiler is a part of the JVM's execution engine that improves performance by compiling frequently executed bytecode ("hot spots") into native machine code at runtime. The compiled native code is cached and reused, so the JVM does not have to interpret the same bytecode again and again.**

### 7.2 Understand the Concept

- The **interpreter** starts fast but is slow for repeated code, because it translates each instruction every time.
- The **JIT** is slower to start (compiling takes time) but produces very fast code.
- The JVM uses both: it **starts with the interpreter**, counts how often each method or loop is executed, and when a threshold is crossed it **JIT-compiles** that code.
- HotSpot JVM has two JIT compilers — **C1** (fast compile, light optimisation) and **C2** (slower compile, heavy optimisation). Using both is called **tiered compilation**.
- JIT optimisations include **method inlining**, **dead code elimination**, **loop unrolling** and **escape analysis**.

More details are in [Part 9, Chapter 1](../09-JVM-and-Memory/01-JVM-Architecture-and-Execution.md).

---

## 8. Interview Questions with Complete Answers

### Basic Level

**Q1. What is the difference between JDK, JRE and JVM?**

The **JVM** executes bytecode and manages memory. The **JRE** is the JVM plus the core class libraries — it is what you need to run Java programs. The **JDK** is the JRE plus development tools like `javac`, `jar`, `javadoc` and `jdb` — it is what you need to develop Java programs. JDK ⊃ JRE ⊃ JVM. All three are platform dependent.

**Q2. What is a `.class` file?**

A `.class` file is the output of the Java compiler. It contains **bytecode** for exactly one class (or interface, enum, record), along with a *constant pool* (names, literals), information about fields and methods, and metadata such as the Java version it was compiled for. It starts with the "magic number" `0xCAFEBABE`, which identifies it as a Java class file.

**Q3. If a `.java` file has three classes, how many `.class` files are generated?**

Three. The compiler creates one `.class` file per class, regardless of how many source files there are. Nested and anonymous classes also get their own files, for example `Outer$Inner.class` and `Outer$1.class`.

**Q4. Can we run a Java program without the JDK?**

Yes, you need only a Java **runtime** (the JRE, or a runtime image created with `jlink`). But to *compile* code you need the JDK because only it contains `javac`. Since Java 11, Oracle distributes only the JDK, so most people install the JDK anyway.

### Intermediate Level

**Q5. What is the role of the bytecode verifier?**

The bytecode verifier is part of the linking step of class loading. Before any code runs, it checks that:
- the bytecode follows the JVM rules and is well-formed,
- there are no illegal type conversions (for example treating an `int` as an object reference),
- the operand stack will not overflow or underflow,
- methods are called with the correct argument types,
- access rules (private/protected) are respected.

This protects the JVM from corrupted or malicious `.class` files.

**Q6. Is the JVM an interpreter or a compiler?**

The JVM uses **both**. Its execution engine has an interpreter, which executes bytecode instruction by instruction, and a JIT compiler, which compiles hot code into native machine code. So the JVM is best described as a runtime environment that interprets and compiles bytecode.

**Q7. What is the difference between `javac` and `java` commands?**

| `javac` | `java` |
|---|---|
| Compiler | Launcher (starts the JVM) |
| Input: `.java` source file (with extension: `javac Test.java`) | Input: class name (without extension: `java Test`) |
| Output: `.class` bytecode | Output: program execution |
| Reports compile-time errors | Reports runtime errors/exceptions |

From Java 11, `java Test.java` can compile in memory and run a single source file directly.

**Q8. What is `jshell`?**

`jshell` is an interactive tool (REPL — Read, Evaluate, Print, Loop) added in **Java 9**. You can type Java statements and expressions and see results immediately without writing a class or a `main` method. It is useful for learning and quickly testing small pieces of code.

### Advanced Level

**Q9. What is `UnsupportedClassVersionError`?**

It is thrown when you try to run a `.class` file on a JVM that is **older** than the Java version used to compile it. Each class file has a version number (Java 8 = 52, Java 11 = 55, Java 17 = 61, Java 21 = 65). An older JVM refuses to run a newer version. Fix it by running on a newer JVM or compiling with `javac --release <older-version>`.

**Q10. Why does Java use both an interpreter and a JIT compiler instead of only one?**

- If the JVM **only interpreted**, programs would start quickly but run slowly.
- If it **only compiled** everything to native code before running, startup would be very slow, and the compiler would not know which code is actually important.
- Using both gives **fast startup** (interpreter) and **high peak performance** (JIT for hot code). The JIT can also optimise based on actual runtime behaviour, which a normal ahead-of-time compiler cannot.

**Q11. What is the difference between the JVM specification and HotSpot?**

The **JVM specification** is a document that defines how a JVM must behave (class file format, bytecode instructions, memory areas). **HotSpot** is a concrete implementation of that specification, developed by Sun/Oracle and included in OpenJDK. Other implementations include **Eclipse OpenJ9** and **GraalVM**. Any correct implementation can run the same bytecode.

---

## 9. Follow-Up Questions

**F1. You said the JVM is platform dependent. Is the JDK also platform dependent?**

Yes. The JDK contains the JVM and native tools, so you download a different JDK for Windows, Linux and macOS. Only the `.class` files are platform independent.

**F2. Where are the core libraries stored in the JDK?**

Before Java 9, they were in a single file called `rt.jar`. Since Java 9 (the **Java Platform Module System**), they are stored as modules in a file called `lib/modules`, and the libraries are split into modules like `java.base`, `java.sql` and `java.desktop`.

**F3. What does `java -version` vs `javac -version` show?**

`java -version` shows the version of the runtime that will run your programs. `javac -version` shows the compiler version. If they differ (because of PATH settings), you may compile with one version and run with another, which can cause `UnsupportedClassVersionError`.

**F4. What is AOT compilation?**

AOT means **Ahead-Of-Time** compilation — compiling Java into native machine code before running it. **GraalVM Native Image** does this to create fast-starting native executables (useful for microservices and serverless). The trade-off is that you lose some dynamic features and runtime JIT optimisations.

---

## 10. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| "JVM contains the JDK." | It is the reverse: **JDK ⊃ JRE ⊃ JVM**. |
| Running `java Welcome.class` | Use the class name: `java Welcome`. |
| "One `.java` file creates one `.class` file." | One `.class` file is created **per class**. |
| "Bytecode is machine code." | Bytecode is for the JVM, not for any real CPU. |
| "The JIT compiles everything at startup." | The JIT compiles only **hot** code at runtime. |
| "JRE is still downloaded separately." | Since Java 11, Oracle ships only the JDK; custom runtimes are built with `jlink`. |

---

## 11. Quick Revision

- **JVM** = executes bytecode (class loader + verifier + interpreter + JIT + GC). Platform dependent.
- **JRE** = JVM + libraries → *run* programs.
- **JDK** = JRE + tools (`javac`, `jar`, `javadoc`, `javap`, `jdb`, `jshell`) → *develop* programs.
- `.java` → `javac` → `.class` (bytecode, one per class) → `java` → JVM → native code.
- Runtime steps: **Load → Link (verify, prepare, resolve) → Initialize → Execute**.
- **Bytecode**: one-byte opcodes, stack-based, viewable with `javap -c`, starts with `0xCAFEBABE`.
- **JIT**: compiles hot code to native code; C1 + C2 = tiered compilation.
- Class file versions: Java 8 = 52, 11 = 55, 17 = 61, 21 = 65.

| Remember as | |
|---|---|
| JVM | **Runs** code |
| JRE | **Run** environment |
| JDK | **Develop** + run |

---

[← Previous: Introduction](01-Introduction-Features-Platform-Independence.md) | [Back to README](../README.md) | [Next: Program Structure →](03-Program-Structure-and-main-Method.md)
