# Java Interview Mastery — Beginner to Advanced

A complete Java interview preparation book, written in simple and professional English.
It is meant for B.Tech CSE students and freshers who want to **understand** Java, **remember** the key technical points, **write** correct code, and **explain** every concept confidently in an interview.

---

## How to Use This Book

### How every chapter is written

Most major concepts follow the same 12-step teaching pattern, so you always know where to look:

| Step | Section | What it gives you |
|---|---|---|
| 1 | **Interview Definition** | A complete definition with the right keywords, ready to speak |
| 2 | **Understand the Concept** | What it is, how it works, why Java has it, what problem it solves |
| 3 | **Real-World Example** | An everyday comparison that makes it easy to remember |
| 4 | **Java Example** | Complete, runnable code |
| 5 | **Code Explanation** | Line-by-line explanation of the important parts |
| 6 | **Expected Output** | The exact output and why it appears |
| 7 | **Interview Answer** | A natural 40–60 second spoken answer |
| 8 | **Interview Questions** | Basic → intermediate → advanced → practical questions |
| 9 | **Complete Answers** | Full answers with reasons, code and output where needed |
| 10 | **Follow-Up Questions** | What the interviewer asks next, with answers |
| 11 | **Common Mistakes** | Misconceptions and confusing points |
| 12 | **Quick Revision** | Key points and comparison tables |

Small topics use only the steps that make sense. Code is not forced where it is not useful.

### About the code

- Every complete program has a `main` method and can be copied into a file with the same name as its public class.
- All programs work on **Java 17 or later**. Features from newer versions are clearly labelled with their version (for example, *Java 16+*, *Java 21*).
- The "Expected Output" blocks were checked by compiling and running the programs with **JDK 21**. Where output can change between runs (threads, timing, hash order), the chapter says so.

To run any example:

```text
javac Main.java
java Main
```

From Java 11 onward you can also run a single file directly: `java Main.java`.

### Suggested study plan (8 weeks)

| Week | Study | Practise |
|---|---|---|
| 1 | Part 1 — Fundamentals | Part 11, Ch 1 (numbers) |
| 2 | Part 2 — Strings and core programming | Part 11, Ch 2–3 (arrays, strings) |
| 3 | Part 3 — OOP (chapters 1–7) | OOP speaking practice daily |
| 4 | Part 3 — OOP (chapters 8–14), Part 4 — Exceptions | Part 11, Ch 10–11 |
| 5 | Part 5 — Collections, Part 6 — Generics | Part 11, Ch 4–7, 9 |
| 6 | Part 7 — Multithreading | Part 11, Ch 13 |
| 7 | Part 8 — Java 8+, Part 9 — JVM, Part 10 — Extra topics | Part 11, Ch 8, 12 |
| 8 | Part 12 — Question bank, Part 13 — Final preparation | Mock interviews |

---

## Table of Contents

### Part 1 — Java Fundamentals
Folder: [`01-Java-Fundamentals`](01-Java-Fundamentals/)

1. [Java Introduction, Features and Platform Independence](01-Java-Fundamentals/01-Introduction-Features-Platform-Independence.md)
2. [JDK, JRE, JVM, Compilation, Execution and Bytecode](01-Java-Fundamentals/02-JDK-JRE-JVM-Compilation-Bytecode.md)
3. [Java Program Structure and the main Method](01-Java-Fundamentals/03-Program-Structure-and-main-Method.md)
4. [Variables and Data Types (Primitive and Reference)](01-Java-Fundamentals/04-Variables-and-Data-Types.md)
5. [Type Casting and Type Promotion](01-Java-Fundamentals/05-Type-Casting.md)
6. [Operators](01-Java-Fundamentals/06-Operators.md)
7. [Control Statements and Loops](01-Java-Fundamentals/07-Control-Statements-and-Loops.md)
8. [Methods, Method Overloading and Recursion](01-Java-Fundamentals/08-Methods-Overloading-Recursion.md)
9. [Command-Line Arguments, Input and Other Foundations](01-Java-Fundamentals/09-Command-Line-Arguments-and-Foundations.md)

> Arrays are taught fully in Part 2, Chapter 4.

### Part 2 — Strings and Core Programming
Folder: [`02-Strings-and-Core-Programming`](02-Strings-and-Core-Programming/)

1. [String Basics, String Pool and Immutability](02-Strings-and-Core-Programming/01-String-Basics-Pool-Immutability.md)
2. [String Comparison and Important String Methods](02-Strings-and-Core-Programming/02-String-Comparison-and-Methods.md)
3. [StringBuilder and StringBuffer](02-Strings-and-Core-Programming/03-StringBuilder-and-StringBuffer.md)
4. [Arrays and Multidimensional Arrays](02-Strings-and-Core-Programming/04-Arrays-and-Multidimensional-Arrays.md)
5. [Wrapper Classes, Autoboxing and Unboxing](02-Strings-and-Core-Programming/05-Wrapper-Classes-Autoboxing-Unboxing.md)
6. [Varargs and Pass-by-Value](02-Strings-and-Core-Programming/06-Varargs-and-Pass-by-Value.md)
7. [Core String and Array Coding Problems](02-Strings-and-Core-Programming/07-Core-Coding-Problems.md)

### Part 3 — Complete Object-Oriented Programming
Folder: [`03-OOP`](03-OOP/)

1. [OOP Introduction, Classes, Objects, Fields and Methods](03-OOP/01-OOP-Introduction-Classes-and-Objects.md)
2. [Constructors, Constructor Overloading, this and super](03-OOP/02-Constructors-this-and-super.md)
3. [Encapsulation](03-OOP/03-Encapsulation.md)
4. [Inheritance and Types of Inheritance](03-OOP/04-Inheritance-and-Types.md)
5. [Polymorphism — Overloading vs Overriding](03-OOP/05-Polymorphism-Overloading-vs-Overriding.md)
6. [Abstraction and Abstract Classes](03-OOP/06-Abstraction-and-Abstract-Classes.md)
7. [Interfaces and Multiple Inheritance](03-OOP/07-Interfaces-and-Multiple-Inheritance.md)
8. [Access Modifiers](03-OOP/08-Access-Modifiers.md)
9. [static and final](03-OOP/09-static-and-final.md)
10. [Object Class — equals(), hashCode(), toString()](03-OOP/10-Object-Class-equals-hashCode-toString.md)
11. [Association, Aggregation, Composition — IS-A and HAS-A](03-OOP/11-Association-Aggregation-Composition.md)
12. [Upcasting, Downcasting and Dynamic Method Dispatch](03-OOP/12-Upcasting-Downcasting-Dynamic-Dispatch.md)
13. [SOLID Principles and OOP Design Concepts](03-OOP/13-SOLID-Principles-and-OOP-Design.md)
14. [OOP Interview Question Bank](03-OOP/14-OOP-Interview-Question-Bank.md)

### Part 4 — Exception Handling
Folder: [`04-Exception-Handling`](04-Exception-Handling/)

1. [Exceptions, Errors and the Exception Hierarchy](04-Exception-Handling/01-Exceptions-Errors-and-Hierarchy.md)
2. [try, catch and finally](04-Exception-Handling/02-try-catch-finally.md)
3. [throw, throws, Custom Exceptions and Exception Propagation](04-Exception-Handling/03-throw-throws-Custom-Exceptions-Propagation.md)
4. [try-with-resources and Best Practices](04-Exception-Handling/04-try-with-resources-and-Best-Practices.md)

### Part 5 — Collections Framework
Folder: [`05-Collections-Framework`](05-Collections-Framework/)

1. [Collections Overview and Hierarchy](05-Collections-Framework/01-Collections-Overview-and-Hierarchy.md)
2. [List — ArrayList, LinkedList, Vector, Stack](05-Collections-Framework/02-List-ArrayList-LinkedList-Vector-Stack.md)
3. [Set — HashSet, LinkedHashSet, TreeSet](05-Collections-Framework/03-Set-HashSet-LinkedHashSet-TreeSet.md)
4. [Queue, PriorityQueue and Deque](05-Collections-Framework/04-Queue-PriorityQueue-Deque.md)
5. [Map — HashMap, LinkedHashMap, TreeMap, Hashtable](05-Collections-Framework/05-Map-HashMap-LinkedHashMap-TreeMap-Hashtable.md)
6. [Hashing, equals/hashCode Contract and HashMap Internal Working](05-Collections-Framework/06-Hashing-and-HashMap-Internal-Working.md)
7. [Iterator, ListIterator and Fail-Fast Behaviour](05-Collections-Framework/07-Iterator-ListIterator-Fail-Fast.md)
8. [Comparable and Comparator](05-Collections-Framework/08-Comparable-and-Comparator.md)
9. [Collection Comparisons and Interview Questions](05-Collections-Framework/09-Collections-Comparisons-and-Interview-Questions.md)

### Part 6 — Generics
Folder: [`06-Generics`](06-Generics/)

1. [Generic Classes, Methods and Interfaces](06-Generics/01-Generic-Classes-Methods-Interfaces.md)
2. [Bounded Types, Wildcards and Type Erasure](06-Generics/02-Bounded-Types-Wildcards-Type-Erasure.md)

### Part 7 — Multithreading and Concurrency
Folder: [`07-Multithreading-and-Concurrency`](07-Multithreading-and-Concurrency/)

1. [Process vs Thread and Thread Lifecycle](07-Multithreading-and-Concurrency/01-Process-Thread-and-Lifecycle.md)
2. [Creating Threads — Thread, Runnable, Callable](07-Multithreading-and-Concurrency/02-Creating-Threads-Runnable-Callable.md)
3. [Thread Methods — sleep, join, interrupt, yield, daemon](07-Multithreading-and-Concurrency/03-Thread-Methods-sleep-join-interrupt.md)
4. [Race Conditions, Synchronization and Locks](07-Multithreading-and-Concurrency/04-Synchronization-Race-Conditions-Locks.md)
5. [Deadlock, volatile and Atomic Classes](07-Multithreading-and-Concurrency/05-Deadlock-volatile-Atomic.md)
6. [wait, notify, notifyAll and Producer–Consumer](07-Multithreading-and-Concurrency/06-wait-notify-Producer-Consumer.md)
7. [ExecutorService, Thread Pools and Future](07-Multithreading-and-Concurrency/07-ExecutorService-Thread-Pools-Future.md)
8. [CompletableFuture and Concurrent Collections](07-Multithreading-and-Concurrency/08-CompletableFuture-and-Concurrent-Collections.md)

### Part 8 — Java 8 and Modern Java
Folder: [`08-Java-8-and-Modern-Java`](08-Java-8-and-Modern-Java/)

1. [Lambda Expressions and Functional Interfaces](08-Java-8-and-Modern-Java/01-Lambda-Expressions-and-Functional-Interfaces.md)
2. [Method References, Default and Static Interface Methods](08-Java-8-and-Modern-Java/02-Method-References-Default-Static-Methods.md)
3. [Stream API — Intermediate and Terminal Operations](08-Java-8-and-Modern-Java/03-Stream-API.md)
4. [Collectors and Advanced Stream Operations](08-Java-8-and-Modern-Java/04-Collectors-and-Advanced-Streams.md)
5. [Optional](08-Java-8-and-Modern-Java/05-Optional.md)
6. [Date and Time API](08-Java-8-and-Modern-Java/06-Date-and-Time-API.md)
7. [Modern Java Features — Java 9 to Java 21](08-Java-8-and-Modern-Java/07-Modern-Java-Features-9-to-21.md)

### Part 9 — JVM and Memory
Folder: [`09-JVM-and-Memory`](09-JVM-and-Memory/)

1. [JVM Architecture, Execution Process and JIT Compiler](09-JVM-and-Memory/01-JVM-Architecture-and-Execution.md)
2. [Class Loading and Class Loaders](09-JVM-and-Memory/02-Class-Loading-and-Class-Loaders.md)
3. [Runtime Memory Areas — Heap, Stack, Metaspace and Object Creation](09-JVM-and-Memory/03-Runtime-Memory-Areas.md)
4. [Garbage Collection and Memory Leaks](09-JVM-and-Memory/04-Garbage-Collection-and-Memory-Leaks.md)

### Part 10 — Additional Important Java Topics
Folder: [`10-Additional-Java-Topics`](10-Additional-Java-Topics/)

1. [Packages, Imports and Access Control](10-Additional-Java-Topics/01-Packages-Imports-Access-Control.md)
2. [Enums](10-Additional-Java-Topics/02-Enums.md)
3. [Annotations](10-Additional-Java-Topics/03-Annotations.md)
4. [Reflection](10-Additional-Java-Topics/04-Reflection.md)
5. [Serialization and Deserialization](10-Additional-Java-Topics/05-Serialization-and-Deserialization.md)
6. [I/O, NIO Basics and File Handling](10-Additional-Java-Topics/06-IO-NIO-and-File-Handling.md)
7. [Nested Classes and Inner Classes](10-Additional-Java-Topics/07-Nested-and-Inner-Classes.md)
8. [Immutable Classes and Records](10-Additional-Java-Topics/08-Immutable-Classes-and-Records.md)

### Part 11 — Java Coding Interview Problems
Folder: [`11-Coding-Interview-Problems`](11-Coding-Interview-Problems/)

1. [Numbers and Basic Logic](11-Coding-Interview-Problems/01-Numbers-and-Basic-Logic.md)
2. [Array Problems](11-Coding-Interview-Problems/02-Array-Problems.md)
3. [String Problems](11-Coding-Interview-Problems/03-String-Problems.md)
4. [Searching and Sorting](11-Coding-Interview-Problems/04-Searching-and-Sorting.md)
5. [Hashing Problems](11-Coding-Interview-Problems/05-Hashing-Problems.md)
6. [Two Pointers](11-Coding-Interview-Problems/06-Two-Pointers.md)
7. [Sliding Window](11-Coding-Interview-Problems/07-Sliding-Window.md)
8. [Recursion and Backtracking](11-Coding-Interview-Problems/08-Recursion-and-Backtracking.md)
9. [Collections Problems](11-Coding-Interview-Problems/09-Collections-Problems.md)
10. [OOP Design Coding Problems](11-Coding-Interview-Problems/10-OOP-Design-Coding-Problems.md)
11. [Exception Handling Problems](11-Coding-Interview-Problems/11-Exception-Handling-Problems.md)
12. [Java 8 Stream Problems](11-Coding-Interview-Problems/12-Java-8-Stream-Problems.md)
13. [Basic Multithreading Problems](11-Coding-Interview-Problems/13-Multithreading-Problems.md)

### Part 12 — Complete Java Interview Question Bank
Folder: [`12-Interview-Question-Bank`](12-Interview-Question-Bank/)

1. [Basic Java Questions](12-Interview-Question-Bank/01-Basic-Java-Questions.md)
2. [Core Java Questions](12-Interview-Question-Bank/02-Core-Java-Questions.md)
3. [OOP Questions (Comparisons and Scenarios)](12-Interview-Question-Bank/03-OOP-Questions.md)
4. [String Questions](12-Interview-Question-Bank/04-String-Questions.md)
5. [Collections Questions](12-Interview-Question-Bank/05-Collections-Questions.md)
6. [Exception Handling Questions](12-Interview-Question-Bank/06-Exception-Questions.md)
7. [Multithreading Questions](12-Interview-Question-Bank/07-Multithreading-Questions.md)
8. [Java 8 Questions](12-Interview-Question-Bank/08-Java-8-Questions.md)
9. [JVM Questions](12-Interview-Question-Bank/09-JVM-Questions.md)
10. [Coding Questions](12-Interview-Question-Bank/10-Coding-Questions.md)
11. [Scenario-Based Questions](12-Interview-Question-Bank/11-Scenario-Based-Questions.md)
12. [Tricky and Output-Prediction Questions](12-Interview-Question-Bank/12-Tricky-Questions.md)
13. [Frequently Asked Follow-Up Questions](12-Interview-Question-Bank/13-Follow-Up-Questions.md)

### Part 13 — Final Interview Preparation
Folder: [`13-Final-Interview-Preparation`](13-Final-Interview-Preparation/)

1. [Important Definitions for Revision](13-Final-Interview-Preparation/01-Important-Definitions.md)
2. [Most Important Comparisons](13-Final-Interview-Preparation/02-Most-Important-Comparisons.md)
3. [Common Interview Mistakes](13-Final-Interview-Preparation/03-Common-Interview-Mistakes.md)
4. [OOP Speaking Practice](13-Final-Interview-Preparation/04-OOP-Speaking-Practice.md)
5. [Mock Interviews with Complete Answers and Follow-Ups](13-Final-Interview-Preparation/05-Mock-Interviews.md)
6. [Final Revision Checklist and Coding Problems to Practise](13-Final-Interview-Preparation/06-Final-Checklist-and-Practice-List.md)

---

## Interview Speaking Tips (read before every interview)

1. **Start with the definition, then explain, then give an example.** "X is … It works by … For example …"
2. **Use the keywords** the interviewer is waiting for (for example: *bytecode*, *JVM*, *heap*, *immutable*, *hash bucket*), but always explain them in your own simple words.
3. **Do not memorise word by word.** Remember the key points of each answer and speak them naturally.
4. **If you do not know, say what you do know.** "I have not used X directly, but I know it is used for …"
5. **Mention a real use.** "In my project I used a `HashMap` to …" makes your answer stronger.
6. **Keep answers to around 40–60 seconds** for major concepts. Stop, and let the interviewer ask follow-ups.
#   j a v a _ m a s t e r y  
 