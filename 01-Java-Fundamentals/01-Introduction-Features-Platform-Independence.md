# Chapter 1.1 — Java Introduction, Features and Platform Independence

[← Back to README](../README.md) | [Next: JDK, JRE, JVM →](02-JDK-JRE-JVM-Compilation-Bytecode.md)

---

## What You Will Learn

- What Java is and why it became so popular
- The history of Java in a few useful points
- All important features of Java (the "Java buzzwords")
- Platform independence and the "Write Once, Run Anywhere" idea
- How to answer "What is Java?" and "Why is Java platform independent?" in an interview

---

## 1. What is Java?

### 1.1 Interview Definition

> **Java is a high-level, object-oriented, class-based, general-purpose programming language that is designed to be platform independent. Java source code is compiled into an intermediate form called bytecode, and this bytecode is executed by the Java Virtual Machine (JVM). Because a JVM is available for every major operating system, the same compiled Java program can run on Windows, Linux or macOS without changes. Java is also known for being simple, secure, robust, multithreaded and having automatic memory management through garbage collection.**

### 1.2 Understand the Concept

Let us break the definition into small parts:

| Part of definition | Meaning in simple words |
|---|---|
| **High-level** | You write code in English-like words (`if`, `class`, `return`) instead of machine instructions. |
| **Object-oriented** | Programs are built using *objects* that combine data (fields) and behaviour (methods). |
| **Class-based** | Every piece of code lives inside a *class*. A class is a blueprint for objects. |
| **General-purpose** | Java is used for many kinds of software — web back-ends, Android apps, banking systems, big data tools. |
| **Bytecode** | The compiler does not produce Windows or Linux machine code. It produces a neutral `.class` file containing bytecode. |
| **JVM** | A program that reads bytecode and runs it on the current machine. |
| **Platform independent** | Compile once; the same `.class` file runs on any OS that has a JVM. |

**Why was Java created?**

Before Java, popular languages like C and C++ were compiled directly into machine code for one specific operating system and processor. If you compiled a C program on Windows, the resulting `.exe` would not run on Linux. You had to recompile it for every platform, and sometimes even change the code.

Java solved this problem by adding a middle layer — the **JVM**. The compiler converts code into bytecode, and every platform has its own JVM that understands the same bytecode. This idea is called **WORA — Write Once, Run Anywhere**.

### 1.3 A Short History (useful for interviews)

| Year | Event |
|---|---|
| 1991 | James Gosling and team at **Sun Microsystems** start the "Green Project". The language was first called **Oak**. |
| 1995 | Renamed to **Java** and publicly released (Java 1.0 in January 1996). |
| 2004 | Java 5 — generics, enhanced for loop, autoboxing, enums, annotations. |
| 2010 | **Oracle** acquires Sun Microsystems. |
| 2014 | **Java 8** — lambda expressions, Stream API, new Date/Time API. Still very common in companies. |
| 2017 onward | New release every **6 months**. Long-Term Support (LTS) versions: **11 (2018), 17 (2021), 21 (2023), 25 (2025)**. |

> **Interview tip:** If asked "Who developed Java?", answer: *"Java was developed by James Gosling and his team at Sun Microsystems in 1995. Sun was later acquired by Oracle, which maintains Java today."*

### 1.4 Real-World Example

Think of Java bytecode like a **DVD movie**. The same DVD can play on a Sony player, a Samsung player or a laptop's DVD drive. The movie (bytecode) is the same; only the player (JVM) is different for each device. The movie makers do not need to make a separate DVD for each brand of player.

### 1.5 Java Example — Your First Program

```java
public class HelloJava {
    public static void main(String[] args) {
        System.out.println("Hello, Java!");
        System.out.println("This program runs on any OS that has a JVM.");
    }
}
```

### 1.6 Code Explanation

| Line | Explanation |
|---|---|
| `public class HelloJava` | Declares a class named `HelloJava`. Because it is `public`, the file must be named `HelloJava.java`. |
| `public static void main(String[] args)` | The **entry point**. The JVM starts running the program from this method. (Explained in detail in Chapter 1.3.) |
| `System.out.println(...)` | Prints text to the console and moves to the next line. `System` is a class, `out` is a static `PrintStream` object, `println` is a method. |

**How to run it:**

```text
javac HelloJava.java      → creates HelloJava.class (bytecode)
java HelloJava            → JVM loads and runs HelloJava.class
```

### 1.7 Expected Output

```text
Hello, Java!
This program runs on any OS that has a JVM.
```

Each `println` prints its text and then a new line, so the two messages appear on separate lines.

### 1.8 Where is Java used?

| Area | Examples |
|---|---|
| Enterprise / back-end | Banking systems, insurance, e-commerce back-ends (Spring Boot) |
| Android | Android apps (Java and Kotlin, both run on Java-like bytecode) |
| Big data | Hadoop, Kafka, Elasticsearch are written in Java |
| Web servers | Tomcat, Jetty |
| Desktop tools | IntelliJ IDEA, Eclipse |
| Cloud / microservices | Spring Boot, Quarkus, Micronaut |

---

## 2. Features of Java

### 2.1 Interview Definition

> **The main features of Java are: simple, object-oriented, platform independent, portable, secure, robust, multithreaded, architecture neutral, interpreted and compiled (with JIT), high performance, distributed and dynamic. These features come mainly from the JVM, bytecode, automatic memory management, strong type checking and a rich standard library.**

### 2.2 Understand Each Feature

#### 1. Simple
Java's syntax is based on C/C++, so it feels familiar, but Java removed confusing features:
- No **explicit pointers** (you cannot do pointer arithmetic like `*(p+1)`)
- No **operator overloading** (except `+` for strings, which is built in)
- No **multiple inheritance of classes** (avoids the "diamond problem")
- No manual memory freeing (`free()`/`delete`) — the **garbage collector** handles it

#### 2. Object-Oriented
Everything in Java is organised around **classes and objects**. Java supports the four OOP pillars: **encapsulation, inheritance, polymorphism and abstraction**.

> Note: Java is *not 100% pure* object-oriented because it has **primitive types** (`int`, `char`, `boolean`, etc.) which are not objects, and it has `static` members that can be used without objects.

#### 3. Platform Independent
Compiled bytecode runs on any OS with a JVM. This is the most famous Java feature (explained fully in Section 3).

#### 4. Portable
Java programs behave the same way on every platform. For example, an `int` is **always 32 bits** in Java, on every machine. In C, the size of `int` can depend on the compiler and platform. Fixed sizes make Java code portable.

#### 5. Secure
- No explicit pointers, so a program cannot directly access random memory addresses.
- Bytecode is checked by the **bytecode verifier** before running.
- Programs run inside the JVM, which acts like a protective layer between the program and the operating system.
- Strong access control (`private`, `protected`, etc.).

#### 6. Robust (strong and reliable)
- **Strong type checking** at compile time catches many errors early.
- **Exception handling** lets programs handle errors without crashing.
- **Garbage collection** prevents most memory leaks and dangling pointer bugs.
- Array bounds are checked at runtime (`ArrayIndexOutOfBoundsException` instead of corrupting memory).

#### 7. Multithreaded
Java has built-in support for threads (`Thread`, `Runnable`, `ExecutorService`). One program can perform many tasks at the same time, for example downloading a file while updating the UI.

#### 8. Architecture Neutral
Bytecode does not depend on processor architecture (x86, ARM). Primitive sizes are fixed. So the compiled file is not tied to a particular CPU.

#### 9. Compiled and Interpreted
Java uses **both**:
1. `javac` **compiles** source code into bytecode.
2. The JVM **interprets** bytecode, and the **JIT (Just-In-Time) compiler** converts frequently used bytecode into native machine code for speed.

#### 10. High Performance
Java is slower than C for some tasks, but much faster than purely interpreted languages because of the **JIT compiler**, which optimises "hot" code at runtime.

#### 11. Distributed
Java has strong networking support (`java.net`, HTTP client, RMI) which makes it easy to build applications that run across many computers.

#### 12. Dynamic
Java loads classes **when they are needed** at runtime (dynamic class loading). It also supports **reflection**, which lets a program inspect classes at runtime.

### 2.3 Real-World Example

Think of Java as a **well-designed car for new drivers**:
- *Simple*: no complicated manual gear (no pointers).
- *Robust*: airbags and ABS (exception handling, garbage collection).
- *Secure*: central locking (bytecode verifier, no direct memory access).
- *Portable*: runs on any road (any OS with a JVM).

### 2.4 Java Example — Robustness in Action

This example shows two robust features: array bound checking and exception handling.

```java
public class RobustDemo {
    public static void main(String[] args) {
        int[] marks = {90, 85, 70};

        try {
            System.out.println("First mark: " + marks[0]);
            System.out.println("Fourth mark: " + marks[3]);   // index 3 does not exist
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Error caught: " + e.getMessage());
        }

        System.out.println("Program continues normally.");
    }
}
```

### 2.5 Code Explanation

- `int[] marks = {90, 85, 70};` creates an array with 3 elements at indexes 0, 1 and 2.
- `marks[3]` tries to read index 3. In C this could silently read garbage memory. **Java checks the index at runtime** and throws `ArrayIndexOutOfBoundsException`.
- The `catch` block handles the error, so the program does not crash.
- The last line runs normally, proving the program recovered.

### 2.6 Expected Output

```text
First mark: 90
Error caught: Index 3 out of bounds for length 3
Program continues normally.
```

The first line prints successfully. The second `println` never prints because the exception happens while evaluating `marks[3]`. Control jumps to `catch`, which prints the exception message. Then the program continues.

### 2.7 Interview Answer — "What are the features of Java?"

> "Java has many features, but the most important ones are: it is **simple**, because it removed complex features like pointers and multiple inheritance of classes. It is **object-oriented**, so code is organised into classes and objects. It is **platform independent**, because the compiler generates bytecode which runs on any JVM. It is **secure**, because there are no explicit pointers and the bytecode verifier checks code before running. It is **robust** because of strong type checking, exception handling and automatic garbage collection. It is **multithreaded**, so we can run multiple tasks at the same time. And it gives **good performance** because of the JIT compiler, which converts frequently used bytecode into native code."

---

## 3. Platform Independence

### 3.1 Interview Definition

> **Platform independence means that a Java program compiled on one operating system can run on any other operating system without recompilation. This is possible because the Java compiler converts source code into platform-neutral bytecode instead of machine-specific code. Each platform has its own JVM, which reads the same bytecode and converts it into the native machine instructions of that platform. So Java programs are platform independent, but the JVM itself is platform dependent.**

### 3.2 Understand the Concept

First, what is a **platform**? A platform is the combination of **operating system + processor architecture** (for example Windows on x86-64, Linux on ARM).

**How C works (platform dependent):**

```text
program.c ──(C compiler for Windows)──► program.exe  → runs only on Windows
program.c ──(C compiler for Linux)────► program      → runs only on Linux
```

You must compile separately for each platform.

**How Java works (platform independent):**

```text
                         ┌──► JVM for Windows ──► Windows machine code
Hello.java ──javac──► Hello.class ──► JVM for Linux ────► Linux machine code
              (bytecode)  └──► JVM for macOS ──► macOS machine code
```

You compile **once**. The same `Hello.class` file goes to every platform. Only the JVM changes.

**The key point interviewers check:**

| Item | Platform independent? |
|---|---|
| Java source code (`.java`) | Yes |
| Bytecode (`.class`) | **Yes** — this is what makes Java platform independent |
| JVM | **No** — each OS has its own JVM implementation |
| JDK / JRE installers | No — you download a different installer for each OS |

### 3.3 Real-World Example

Think of bytecode as a **speech written in English**, and the JVM as a **live interpreter** at a conference. The same speech is given in Japan, Germany and Brazil. In each country, a local interpreter translates it into Japanese, German or Portuguese. The speaker writes the speech only once. The interpreters (JVMs) are different for each country.

### 3.4 Java Example — Same Program, Different Platforms

This program prints details of the platform it runs on. Compile it once and run the same `.class` file on Windows and Linux — it works on both, but prints different OS names.

<!-- output-varies -->
```java
public class PlatformInfo {
    public static void main(String[] args) {
        System.out.println("OS Name     : " + System.getProperty("os.name"));
        System.out.println("Architecture: " + System.getProperty("os.arch"));
        System.out.println("Java Version: " + System.getProperty("java.version"));
    }
}
```

### 3.5 Code Explanation

- `System.getProperty("os.name")` asks the JVM which operating system it is running on.
- `os.arch` gives the processor architecture (for example `amd64`, `aarch64`).
- `java.version` gives the version of Java that is running the program.
- The bytecode is identical everywhere. The *JVM* fills in these values for its platform.

### 3.6 Expected Output

On Windows (example):

```text
OS Name     : Windows 11
Architecture: amd64
Java Version: 21.0.4
```

On Linux, using the **same** `PlatformInfo.class` file (example):

```text
OS Name     : Linux
Architecture: amd64
Java Version: 21.0.4
```

The output differs because each JVM reports its own platform, which proves one compiled file is running on different systems.

### 3.7 Interview Answer — "Why is Java platform independent?"

> "Java is platform independent because the Java compiler does not convert source code directly into machine code. Instead, it converts it into **bytecode**, which is stored in a `.class` file. Bytecode is not specific to any operating system or processor. Every platform has its own **JVM**, and the JVM converts this bytecode into the native machine code of that platform at runtime. So I can compile a program on Windows and run the same `.class` file on Linux or macOS. That is why Java follows the principle **Write Once, Run Anywhere**. One important point is that the bytecode is platform independent, but the **JVM itself is platform dependent**, because each operating system needs its own JVM."

---

## 4. Interview Questions with Complete Answers

### Basic Level

**Q1. What is Java?**

Java is a high-level, object-oriented, platform-independent programming language developed by James Gosling at Sun Microsystems in 1995 and now owned by Oracle. Java source code is compiled into bytecode, which runs on the JVM. Because JVMs exist for all major operating systems, the same program runs everywhere — "Write Once, Run Anywhere". Java is widely used for back-end web applications, Android apps, enterprise systems and big data tools.

**Q2. Who developed Java and what was its original name?**

Java was developed by **James Gosling** and his team (Mike Sheridan, Patrick Naughton) at **Sun Microsystems**. The project started in 1991, and the language was originally named **Oak**, after an oak tree outside Gosling's office. It was renamed **Java** in 1995 because "Oak" was already a trademark. Oracle acquired Sun in 2010.

**Q3. What does WORA mean?**

WORA stands for **Write Once, Run Anywhere**. It means a Java program is written and compiled once, and the resulting bytecode can run on any device or OS that has a JVM, without changing or recompiling the code.

**Q4. Is Java a compiled or an interpreted language?**

Java is **both**. First, the `javac` compiler compiles source code into bytecode. Then, at runtime, the JVM executes the bytecode using an **interpreter**, and the **JIT compiler** compiles frequently executed parts ("hot spots") into native machine code. So Java uses a combination of compilation and interpretation to get both portability and good performance.

### Intermediate Level

**Q5. Is Java 100% object-oriented?**

No. Java is **not purely object-oriented** for two main reasons:
1. It has **8 primitive data types** (`byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`) which are not objects.
2. It has **static** methods and variables, which can be used without creating any object.

However, Java provides **wrapper classes** (`Integer`, `Double`, etc.) to treat primitives as objects when needed. Languages like Smalltalk are considered purely object-oriented.

**Q6. Why does Java not support pointers?**

Java removed explicit pointers for **security and simplicity**:
- Pointers allow direct memory access, which can be misused to read or corrupt memory.
- Pointer arithmetic causes bugs such as dangling pointers and buffer overflows.
- Without pointers, the JVM can safely manage memory and run the garbage collector.

Java still uses **references** internally (a reference variable holds the address of an object), but you cannot do arithmetic on them or access arbitrary memory.

**Q7. What is the difference between platform independent and portable?**

- **Platform independent** means the compiled bytecode runs on any OS with a JVM.
- **Portable** means the program **behaves the same way** on all platforms — for example, `int` is always 32 bits and `char` is always 16 bits, so results do not change from machine to machine.

Platform independence is about *where* it runs; portability is about *how consistently* it runs.

**Q8. Why is the JVM platform dependent if Java is platform independent?**

The JVM is the program that actually talks to the operating system and CPU. Windows, Linux and macOS have different system calls, file systems and machine instructions, so the JVM must be written separately for each. This is the trade-off: Oracle and other vendors write a JVM for each platform once, so that millions of Java programs do not need to be rewritten.

### Advanced Level

**Q9. Why is Java considered secure?**

1. **No explicit pointers** — programs cannot access random memory.
2. **Bytecode verifier** — checks `.class` files for illegal code before execution (for example, stack overflows or type violations).
3. **Class loader** — separates classes loaded from different sources, so untrusted code cannot replace core Java classes.
4. **JVM sandbox** — the program runs inside the JVM, not directly on the OS.
5. **Strong typing and access modifiers** — prevent illegal access to data.
6. **Automatic memory management** — no use-after-free bugs.

**Q10. Why is Java slower than C/C++? Is it still slow?**

Historically, Java was slower because bytecode was purely interpreted and garbage collection added pauses. Today, the **JIT compiler** compiles hot code to optimised native code, and it can even optimise using runtime information (for example, inlining methods that are called often). Modern Java is close to C++ performance for many server workloads. Java still has some overhead: JVM startup time, memory for the JVM itself, and GC pauses.

### Practical / Frequently Asked

**Q11. Can a `.class` file compiled with Java 21 run on Java 8?**

No. Bytecode is **backward compatible**, not forward compatible. A newer JVM can run older bytecode, but an older JVM cannot run bytecode compiled for a newer version. Java 8 would throw `UnsupportedClassVersionError`. To target an older version, compile with `javac --release 8 MyClass.java`.

**Q12. What are LTS versions of Java and why do companies use them?**

LTS means **Long-Term Support**. Oracle and other vendors provide updates and security fixes for LTS versions for many years. Current LTS versions are **8, 11, 17, 21 and 25**. Companies prefer LTS releases because they are stable and supported for a long time, while non-LTS versions are supported only for about six months.

---

## 5. Follow-Up Questions

**F1. You said bytecode is platform independent. What exactly is bytecode?**

Bytecode is a set of compact instructions for the JVM, stored in `.class` files. Each instruction is one byte long (that is where the name comes from), followed by optional operands. For example, `iadd` adds two integers and `invokevirtual` calls a method. You can see bytecode using the `javap -c ClassName` command. (More detail in Chapter 1.2.)

**F2. If the JVM converts bytecode to machine code, why not compile directly to machine code?**

Direct compilation would make the output platform specific, so we would lose WORA. Bytecode also lets the JVM do runtime work such as **security verification**, **garbage collection**, and **JIT optimisation** based on how the program actually runs.

**F3. Name a feature removed from C++ in Java and why.**

**Multiple inheritance of classes** was removed to avoid the **diamond problem** — when a class inherits the same method from two parents, it is unclear which one to use. Java allows multiple inheritance only through **interfaces**, where conflicts must be resolved explicitly.

**F4. Is Java pass-by-value or pass-by-reference?**

Java is **always pass-by-value**. For objects, the value that is passed is a copy of the reference. This is explained in Part 2, Chapter 6.

---

## 6. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| "The JVM is platform independent." | The **bytecode** is platform independent. The JVM is platform **dependent**. |
| "Java is purely object-oriented." | It is not, because of primitive types and static members. |
| "Java is only interpreted." | Java is compiled to bytecode, then interpreted **and** JIT-compiled. |
| "Java has no pointers at all." | Java has **references**; it has no *explicit* pointers or pointer arithmetic. |
| "Java and JavaScript are related." | They are completely different languages. The name similarity was marketing. |
| Saying only one-word features ("simple, secure…") | Always give a **reason** for each feature. |

---

## 7. Quick Revision

**Key points**
- Java: high-level, object-oriented, platform-independent, developed by James Gosling (Sun, 1995), now Oracle.
- Source `.java` → `javac` → bytecode `.class` → JVM → machine code.
- **WORA**: Write Once, Run Anywhere.
- **Bytecode is platform independent; the JVM is platform dependent.**
- Features: Simple, OOP, Platform independent, Portable, Secure, Robust, Multithreaded, Architecture neutral, Compiled + Interpreted, High performance (JIT), Distributed, Dynamic.
- Not 100% OOP: primitives and static members.
- LTS versions: 8, 11, 17, 21, 25.

**Memory trick for features — "SOP PR SM HD"**
**S**imple, **O**bject-oriented, **P**latform independent, **P**ortable, **R**obust, **S**ecure, **M**ultithreaded, **H**igh performance, **D**ynamic/Distributed.

| Question | One-line answer to start with |
|---|---|
| What is Java? | A high-level, object-oriented, platform-independent language that runs on the JVM. |
| Why platform independent? | Because it compiles to bytecode, which any platform's JVM can run. |
| Is JVM platform independent? | No, each OS has its own JVM. |
| Compiled or interpreted? | Both — compiled to bytecode, then interpreted and JIT-compiled. |
| Why no pointers? | For security, simplicity and safe garbage collection. |

---

[← Back to README](../README.md) | [Next: JDK, JRE, JVM →](02-JDK-JRE-JVM-Compilation-Bytecode.md)
