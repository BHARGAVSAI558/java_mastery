# Chapter 1.8 — Methods, Method Overloading and Recursion

[← Previous: Control Statements and Loops](07-Control-Statements-and-Loops.md) | [Back to README](../README.md) | [Next: Command-Line Arguments and Foundations →](09-Command-Line-Arguments-and-Foundations.md)

---

## What You Will Learn

- What a method is, its parts, and why we use methods
- Method signature vs method header
- Static vs instance methods, parameters vs arguments, return types
- How a method call works on the call stack
- **Method overloading** — rules, type promotion during overloading, ambiguity
- **Recursion** — base case, recursive case, stack frames, `StackOverflowError`, recursion vs iteration

---

## 1. Methods

### 1.1 Interview Definition

> **A method is a named block of code inside a class that performs a specific task and runs only when it is called. A method can accept input values through parameters, and it can return a result using the `return` statement; if it returns nothing, its return type is `void`. Methods support code reusability, modularity and readability, because a task written once can be called many times from different places. In Java, every method belongs to a class, and it can be either an instance method, which is called on an object, or a static method, which belongs to the class itself.**

### 1.2 Understand the Concept

**Parts of a method:**

```text
  public    static    int      add   (int a, int b)   throws Exception  {  return a + b;  }
  ──────    ──────    ───      ───    ─────────────   ────────────────     ─────────────
 access    non-access return   method   parameter        exceptions          method body
 modifier  modifier   type     name     list             (optional)
```

| Part | Meaning |
|---|---|
| Access modifier | Who can call it: `public`, `protected`, default, `private` |
| Non-access modifiers | `static`, `final`, `abstract`, `synchronized`, … |
| Return type | Type of value returned (`int`, `String`, `void` …) |
| Method name | Should be a verb in camelCase (`calculateTotal`) |
| Parameter list | Inputs (type + name), can be empty |
| `throws` clause | Checked exceptions the method may throw |
| Body | Statements that do the work |

**Method signature** = **method name + parameter list (number, types and order of parameters)**.
The return type, modifiers, parameter names and `throws` clause are **not** part of the signature. This is important for overloading.

**Parameters vs arguments:**
- **Parameters** are the variables in the method definition: `int add(int a, int b)` → `a`, `b`.
- **Arguments** are the actual values passed in the call: `add(5, 10)` → `5`, `10`.

**Why use methods?**
1. **Reusability** — write once, call many times.
2. **Modularity** — break a big problem into small pieces.
3. **Readability** — `calculateTax(salary)` is easier to understand than 20 lines of maths.
4. **Easier testing and debugging** — test one method at a time.

### 1.3 Static vs Instance Methods

| Point | Static method | Instance method |
|---|---|---|
| Keyword | `static` | no `static` |
| Belongs to | The class | An object |
| Called using | `ClassName.method()` | `object.method()` |
| Can access instance variables directly? | **No** (no `this`) | Yes |
| Can access static variables? | Yes | Yes |
| Can use `this` / `super`? | No | Yes |
| Example | `Math.sqrt(16)`, `Integer.parseInt("5")` | `str.length()`, `list.add(x)` |
| Can be overridden? | No (only hidden) | Yes |

### 1.4 Real-World Example

A method is like a **coffee machine button**:
- The button has a **name** ("Cappuccino").
- You give **inputs** (parameters: sugar level, cup size).
- It **returns** an output (a cup of coffee).
- You can press it again and again — you don't rebuild the machine each time (reusability).
- A "Clean machine" button does a task but gives you nothing back — like a `void` method.

### 1.5 Java Example

```java
public class Calculator {
    String brand = "Casio";                // instance variable
    static int callCount = 0;              // static variable

    // static method: no object needed
    static int add(int a, int b) {
        callCount++;
        return a + b;
    }

    // instance method: needs an object, can use instance variables
    double average(int[] numbers) {
        callCount++;
        int sum = 0;
        for (int n : numbers) {
            sum += n;
        }
        return (double) sum / numbers.length;
    }

    // void method: performs an action, returns nothing
    void printBrand() {
        System.out.println("Brand: " + brand);
    }

    // method returning boolean, with early return
    static boolean isPrime(int n) {
        if (n < 2) {
            return false;
        }
        for (int i = 2; i * i <= n; i++) {
            if (n % i == 0) {
                return false;
            }
        }
        return true;
    }

    public static void main(String[] args) {
        int result = add(10, 20);                    // call static method directly
        System.out.println("Sum: " + result);

        Calculator calc = new Calculator();          // object needed for instance methods
        System.out.println("Average: " + calc.average(new int[]{4, 5, 6, 7}));
        calc.printBrand();

        System.out.println("Is 29 prime? " + isPrime(29));
        System.out.println("Is 15 prime? " + isPrime(15));
        System.out.println("Methods called: " + callCount);
    }
}
```

### 1.6 Code Explanation

- `add` is `static`, so `main` (also static) calls it directly without an object.
- `average` is an instance method, so we create `calc` and call `calc.average(...)`.
- `(double) sum / numbers.length` — casting before dividing gives a decimal result (22 / 4 = 5.5).
- `printBrand()` uses the instance variable `brand`; this would not be allowed inside a static method.
- `isPrime` uses **early return**: as soon as a divisor is found, it returns `false`. The loop checks up to `√n` (`i * i <= n`) — a common optimisation.
- `callCount` counts calls of `add` and `average` (2 calls).

### 1.7 Expected Output

```text
Sum: 30
Average: 5.5
Brand: Casio
Is 29 prime? true
Is 15 prime? false
Methods called: 2
```

### 1.8 How a Method Call Works — The Call Stack

Every time a method is called, the JVM creates a **stack frame** on the thread's **call stack**. The frame holds the method's parameters, local variables, and the return address. When the method returns, its frame is removed (popped).

```text
main() calls add(10, 20):

  Step 1               Step 2                    Step 3
┌──────────────┐    ┌──────────────────┐     ┌──────────────┐
│   main()     │    │ add: a=10, b=20  │ ◄── │   main()     │
│ args, result │    ├──────────────────┤ top │ result = 30  │
└──────────────┘    │   main()         │     └──────────────┘
                    └──────────────────┘
                     add's frame pushed       add returns 30, frame popped
```

This is why local variables disappear after a method returns, and why too many nested calls cause `StackOverflowError`.

### 1.9 Interview Answer — "What is a method? Static vs instance method?"

> "A method is a block of code inside a class that performs a specific task and runs only when it is called. It can take parameters as input and return a value, or return nothing using `void`. Methods help us reuse code, keep the program modular and make it readable. A method signature is the method name plus its parameter list. There are two main kinds. A **static method** belongs to the class; we call it using the class name, like `Math.max()`, and it cannot directly access instance variables or use `this`. An **instance method** belongs to an object; we call it using an object reference, and it can access both instance and static members. When a method is called, the JVM creates a stack frame for its parameters and local variables, and removes it when the method returns."

---

## 2. Method Overloading

### 2.1 Interview Definition

> **Method overloading is a feature that allows a class to have more than one method with the same name, as long as their parameter lists are different — in the number of parameters, the type of parameters, or the order of parameter types. The compiler decides which overloaded method to call at compile time based on the arguments, so overloading is an example of compile-time (static) polymorphism. Changing only the return type, access modifier or parameter names is not enough to overload a method.**

### 2.2 Understand the Concept

**Valid ways to overload:**

| Rule | Example |
|---|---|
| Different **number** of parameters | `add(int a, int b)` and `add(int a, int b, int c)` |
| Different **types** of parameters | `add(int a, int b)` and `add(double a, double b)` |
| Different **order** of types | `show(int a, String b)` and `show(String b, int a)` |

**Invalid (compile error — "method is already defined"):**
- Only the return type differs: `int sum(int a)` and `double sum(int a)`.
- Only parameter names differ: `sum(int a)` and `sum(int b)`.
- Only modifiers differ: `public void m()` and `static void m()`.

**Why is return type not enough?** Because a call like `sum(5);` might ignore the result. The compiler would have no way to know which method you meant.

**Why use overloading?** It gives one easy-to-remember name for similar operations. `System.out.println()` is overloaded for `int`, `double`, `char`, `String`, `Object`, and more — you don't need `printInt`, `printString`, etc.

### 2.3 Real-World Example

A **"pay" counter in a shop** accepts payment by cash, by card, or by UPI. It is the same action — *pay* — but the input is different. The cashier (compiler) looks at what you hand over and decides how to process it.

### 2.4 Java Example

```java
public class OverloadDemo {

    static int add(int a, int b) {
        System.out.print("add(int, int) -> ");
        return a + b;
    }

    static int add(int a, int b, int c) {
        System.out.print("add(int, int, int) -> ");
        return a + b + c;
    }

    static double add(double a, double b) {
        System.out.print("add(double, double) -> ");
        return a + b;
    }

    static String add(String a, String b) {
        System.out.print("add(String, String) -> ");
        return a + b;
    }

    static void show(int a, String b) {
        System.out.println("show(int, String): " + a + ", " + b);
    }

    static void show(String b, int a) {
        System.out.println("show(String, int): " + b + ", " + a);
    }

    public static void main(String[] args) {
        System.out.println(add(2, 3));
        System.out.println(add(2, 3, 4));
        System.out.println(add(2.5, 3.5));
        System.out.println(add("Hello, ", "Java"));
        show(1, "one");
        show("two", 2);
    }
}
```

### 2.5 Expected Output

```text
add(int, int) -> 5
add(int, int, int) -> 9
add(double, double) -> 6.0
add(String, String) -> Hello, Java
show(int, String): 1, one
show(String, int): two, 2
```

The compiler matches each call to the method whose parameters fit the arguments best.

### 2.6 Overloading with Type Promotion

If there is no exact match, the compiler tries, **in this order**:
1. **Exact match**
2. **Widening** primitive conversion (`int → long → float → double`)
3. **Autoboxing** (`int → Integer`)
4. **Varargs** (`int...`)

```java
public class OverloadPromotion {
    static void test(long x)    { System.out.println("long version: " + x); }
    static void test(Integer x) { System.out.println("Integer version: " + x); }
    static void test(int... x)  { System.out.println("varargs version, length " + x.length); }
    static void test(double x)  { System.out.println("double version: " + x); }

    static void print(Object o) { System.out.println("Object version"); }
    static void print(String s) { System.out.println("String version"); }

    public static void main(String[] args) {
        int i = 10;
        test(i);          // no test(int); widening to long wins over boxing and varargs
        test(5.5f);       // float → double (widening)
        test();           // only varargs accepts zero arguments
        print("Hi");      // most specific type wins
        print(null);      // null fits both; String is more specific than Object
        print(42);        // int → boxed Integer → Object
    }
}
```

**Expected Output:**

```text
long version: 10
double version: 5.5
varargs version, length 0
String version
String version
Object version
```

**Explanation:**
- `test(10)` — there is no `test(int)`. **Widening** (`int → long`) has higher priority than **boxing** (`int → Integer`), and both beat varargs.
- `test(5.5f)` — `float` widens to `double`.
- `print(null)` — `null` can be a `String` or an `Object`. Java chooses the **most specific** type, `String` (because `String` is a subclass of `Object`).
- If we had `print(String)` and `print(StringBuilder)`, `print(null)` would be **ambiguous** → compile error, because neither is more specific.

### 2.7 Interview Answer — "What is method overloading?"

> "Method overloading means having multiple methods with the **same name in the same class** but with **different parameter lists** — different number of parameters, different types, or a different order of types. The compiler decides which method to call at **compile time** by looking at the arguments, so overloading is **compile-time polymorphism** or static binding. We cannot overload by changing only the return type, because the compiler cannot decide based on return type alone. A common example is `System.out.println`, which is overloaded for int, String, double and so on. If there is no exact match, the compiler tries widening first, then autoboxing, then varargs. We can also overload static methods, constructors, and even the main method."

---

## 3. Recursion

### 3.1 Interview Definition

> **Recursion is a programming technique in which a method calls itself to solve a problem by breaking it into smaller sub-problems of the same type. Every recursive method must have a base case, which stops the recursion and returns a result directly, and a recursive case, which calls the method again with a smaller or simpler input that moves towards the base case. Each recursive call creates a new stack frame, so without a correct base case the calls never end and the program fails with a `StackOverflowError`.**

### 3.2 Understand the Concept

To write any recursive method, answer two questions:
1. **Base case:** What is the smallest input I can answer directly? (e.g., `factorial(0) = 1`)
2. **Recursive case:** How can I express the answer using the same problem on a smaller input? (e.g., `factorial(n) = n × factorial(n−1)`)

Then **trust** that the smaller call works (this is called the "leap of faith").

**How the stack works for `factorial(4)`:**

```text
Calls going down (pushing frames)        Returns coming up (popping frames)
factorial(4) = 4 * factorial(3)          factorial(0) returns 1
  factorial(3) = 3 * factorial(2)        factorial(1) returns 1 * 1 = 1
    factorial(2) = 2 * factorial(1)      factorial(2) returns 2 * 1 = 2
      factorial(1) = 1 * factorial(0)    factorial(3) returns 3 * 2 = 6
        factorial(0) = 1  ← base case    factorial(4) returns 4 * 6 = 24
```

### 3.3 Real-World Example

**Russian nesting dolls (Matryoshka):** To find the smallest doll, you open a doll, and inside is a smaller doll of the same kind. You repeat the same action until you reach a doll that cannot be opened (the base case). Then you put them back together in reverse order (returning up the stack).

### 3.4 Java Example

```java
public class RecursionDemo {

    // Factorial: n! = n * (n-1)!
    static long factorial(int n) {
        if (n <= 1) {                 // base case
            return 1;
        }
        return n * factorial(n - 1);  // recursive case
    }

    // Fibonacci: fib(n) = fib(n-1) + fib(n-2)
    static int fibonacci(int n) {
        if (n <= 1) {
            return n;                 // fib(0) = 0, fib(1) = 1
        }
        return fibonacci(n - 1) + fibonacci(n - 2);
    }

    // Sum of digits
    static int sumOfDigits(int n) {
        if (n == 0) {
            return 0;
        }
        return n % 10 + sumOfDigits(n / 10);
    }

    // Reverse a string
    static String reverse(String s) {
        if (s.isEmpty()) {
            return s;
        }
        return reverse(s.substring(1)) + s.charAt(0);
    }

    // Power: a^b using fast exponentiation (O(log b))
    static long power(long a, int b) {
        if (b == 0) {
            return 1;
        }
        long half = power(a, b / 2);
        return (b % 2 == 0) ? half * half : half * half * a;
    }

    // Print 1..n (call before print) and n..1 (print before call)
    static void printAscending(int n) {
        if (n == 0) return;
        printAscending(n - 1);
        System.out.print(n + " ");
    }

    static void printDescending(int n) {
        if (n == 0) return;
        System.out.print(n + " ");
        printDescending(n - 1);
    }

    public static void main(String[] args) {
        System.out.println("5! = " + factorial(5));
        System.out.println("20! = " + factorial(20));
        System.out.print("Fibonacci series: ");
        for (int i = 0; i < 10; i++) {
            System.out.print(fibonacci(i) + " ");
        }
        System.out.println();
        System.out.println("Sum of digits of 9875 = " + sumOfDigits(9875));
        System.out.println("Reverse of 'java' = " + reverse("java"));
        System.out.println("2^10 = " + power(2, 10));
        System.out.print("Ascending: ");
        printAscending(5);
        System.out.println();
        System.out.print("Descending: ");
        printDescending(5);
        System.out.println();
    }
}
```

### 3.5 Code Explanation

- **factorial** — base case `n <= 1` returns 1; otherwise multiplies `n` by the factorial of `n − 1`. It returns `long` because 20! (2,432,902,008,176,640,000) does not fit in `int`. (21! overflows even `long` — use `BigInteger`.)
- **fibonacci** — two recursive calls per step. Simple but slow: **O(2ⁿ)**, because the same values are calculated many times. (Fixed with memoization or a loop — see Q8.)
- **sumOfDigits(9875)** → `5 + sumOfDigits(987)` → `5 + 7 + sumOfDigits(98)` → … → 29.
- **reverse("java")** → `reverse("ava") + 'j'` → `reverse("va") + 'a' + 'j'` → … → `"avaj"`.
- **power** — calculates `a^(b/2)` once and squares it, so it needs only about log₂(b) calls.
- **printAscending vs printDescending** — the *position* of the print relative to the recursive call decides the order. Printing **after** the call prints on the way back up (ascending); printing **before** prints on the way down (descending).

### 3.6 Expected Output

```text
5! = 120
20! = 2432902008176640000
Fibonacci series: 0 1 1 2 3 5 8 13 21 34 
Sum of digits of 9875 = 29
Reverse of 'java' = avaj
2^10 = 1024
Ascending: 1 2 3 4 5 
Descending: 5 4 3 2 1 
```

### 3.7 StackOverflowError

```java
public class StackOverflowDemo {
    static int depth = 0;

    static void infinite() {
        depth++;
        infinite();              // no base case
    }

    public static void main(String[] args) {
        try {
            infinite();
        } catch (StackOverflowError e) {
            System.out.println("StackOverflowError caught. Depth reached > 1000: " + (depth > 1000));
        }
    }
}
```

**Expected Output:**

```text
StackOverflowError caught. Depth reached > 1000: true
```

Each call adds a frame; the stack has a fixed size (default around 512 KB–1 MB per thread, configurable with `-Xss`). When it is full, the JVM throws `StackOverflowError`. It is an **`Error`**, not an `Exception` — catching it is shown here only for learning; real programs should fix the recursion instead.

### 3.8 Recursion vs Iteration

| Point | Recursion | Iteration (loops) |
|---|---|---|
| How it repeats | Method calls itself | `for` / `while` loop |
| Termination | Base case | Loop condition becomes false |
| Memory | Uses a stack frame per call → O(depth) extra memory | Usually O(1) extra memory |
| Speed | Slower (call overhead) | Faster |
| Risk | `StackOverflowError` for deep recursion | Infinite loop if condition wrong |
| Code clarity | Very clean for trees, graphs, divide-and-conquer, backtracking | Clean for simple repetition |
| Examples | Tree traversal, merge sort, quick sort, permutations, Tower of Hanoi | Sum of array, printing tables |

**Note:** Java does **not** perform *tail-call optimisation*, so even tail-recursive methods use a new stack frame for each call.

### 3.9 Interview Answer — "What is recursion?"

> "Recursion is when a method calls itself to solve a problem by breaking it into smaller versions of the same problem. Every recursive method needs two parts: a **base case**, which is the simplest input that can be answered directly and stops the recursion, and a **recursive case**, which calls the same method with a smaller input that moves towards the base case. For example, factorial of n is n multiplied by factorial of n minus one, and the base case is factorial of 0 or 1 equals 1. Each call creates a new stack frame, so recursion uses more memory than a loop, and if the base case is missing or never reached, the stack fills up and we get a `StackOverflowError`. Recursion is very useful for problems like tree traversal, merge sort and backtracking, where the problem is naturally recursive."

---

## 4. Interview Questions with Complete Answers

### Basic Level

**Q1. Can we overload a method by changing only its return type?**

No. The method signature consists of the name and parameter list only, so two methods with the same name and parameters but different return types have the same signature, which is a compile error ("method is already defined"). The compiler cannot decide which one to call, especially when the return value is ignored, like `calculate(5);`.

**Q2. Can a static method call an instance method directly?**

No. A static method has no `this` reference because it is not called on any object. To call an instance method from a static method, you must first create an object: `new MyClass().instanceMethod();`. But an instance method can call static methods directly.

**Q3. What is the difference between `return` and `System.exit()`?**

`return` exits only the **current method** and gives control back to the caller. `System.exit(status)` **terminates the whole JVM** (after running shutdown hooks). `finally` blocks run after `return`, but **not** after `System.exit()`.

### Intermediate Level

**Q4. Can we overload static methods? Can we overload `main`?**

Yes to both. Static methods can be overloaded like any method — same name, different parameters. `main` can also be overloaded, but the JVM always starts with `public static void main(String[] args)`; other versions run only if called explicitly.

**Q5. What is the difference between method overloading and method overriding?**

| Overloading | Overriding |
|---|---|
| Same name, **different parameters** | Same name, **same parameters** (same signature) |
| Usually in the **same class** (can also be in a subclass) | In **parent and child** classes |
| Compile-time polymorphism (static binding) | Runtime polymorphism (dynamic binding) |
| Return type can be anything | Return type must be same or a subtype (covariant) |
| `static`, `private`, `final` methods **can** be overloaded | `static`, `private`, `final` methods **cannot** be overridden |
| Inheritance not needed | Inheritance required |

(Full detail in [Part 3, Chapter 5](../03-OOP/05-Polymorphism-Overloading-vs-Overriding.md).)

**Q6. Which method is called: `m(long)` or `m(Integer)` for `m(5)`?**

`m(long)`. When there is no exact `m(int)`, the compiler prefers **widening** over **autoboxing** (and both over varargs). This rule was kept for backward compatibility: code written before Java 5 (when autoboxing was introduced) should keep calling the same method.

**Q7. What is a recursive method's time and space complexity for factorial?**

Factorial makes `n` calls, each doing constant work → **O(n) time**. Each call stays on the stack until the base case is reached → **O(n) space**. An iterative loop has O(n) time but **O(1)** space.

### Advanced Level

**Q8. Why is recursive Fibonacci slow, and how do you make it fast?**

Plain recursive Fibonacci recalculates the same sub-problems many times. `fib(5)` calls `fib(3)` twice and `fib(2)` three times. The number of calls grows like 2ⁿ → **O(2ⁿ) time**. Fixes:
1. **Memoization** — store results in an array/map and reuse them → O(n) time.
2. **Iteration** — keep the last two values → O(n) time, O(1) space.

```java
public class FibFast {
    static long[] memo = new long[100];

    static long fibMemo(int n) {
        if (n <= 1) return n;
        if (memo[n] != 0) return memo[n];          // already calculated
        memo[n] = fibMemo(n - 1) + fibMemo(n - 2);
        return memo[n];
    }

    static long fibIterative(int n) {
        if (n <= 1) return n;
        long prev = 0, curr = 1;
        for (int i = 2; i <= n; i++) {
            long next = prev + curr;
            prev = curr;
            curr = next;
        }
        return curr;
    }

    public static void main(String[] args) {
        System.out.println("fib(50) memo      = " + fibMemo(50));
        System.out.println("fib(50) iterative = " + fibIterative(50));
    }
}
```

**Expected Output:**

```text
fib(50) memo      = 12586269025
fib(50) iterative = 12586269025
```

The plain recursive version would make billions of calls for `fib(50)`; both versions above finish instantly.

**Q9. Can recursion always be replaced by iteration?**

Yes, in theory every recursive algorithm can be written iteratively, often by using an explicit `Stack` data structure to simulate the call stack. For simple recursion (factorial, sum), a loop is easy. For tree/graph algorithms, the recursive version is usually cleaner, but an iterative version with an explicit stack avoids `StackOverflowError` for very deep inputs.

**Q10. What is the difference between direct and indirect recursion, and head vs tail recursion?**

- **Direct recursion:** method `A` calls `A`.
- **Indirect recursion:** `A` calls `B`, and `B` calls `A`.
- **Tail recursion:** the recursive call is the **last** operation (nothing left to do after it returns), for example `printDescending`.
- **Head recursion:** the recursive call happens **first**, and work is done after it returns, for example `printAscending`.

Java does not optimise tail recursion, so both use stack frames.

---

## 5. Follow-Up Questions

**F1. Can we declare a method inside another method in Java?**

No, Java does not allow nested methods. But you can declare a **local class** or use a **lambda** inside a method to get similar behaviour.

**F2. What is a "method header" vs "method signature"?**

The **signature** is the name + parameter types (used for overloading and overriding). The **header** (or declaration) is the full first line: modifiers, return type, name, parameters and `throws` clause.

**F3. What is the maximum recursion depth in Java?**

There is no fixed number. It depends on the stack size (`-Xss`, e.g. `-Xss2m`) and how much each frame uses (more local variables = bigger frames). Typically a few thousand to tens of thousands of calls.

**F4. Is it possible to return multiple values from a method?**

Not directly. You can return an array, a collection, a custom class, or (Java 16+) a **record**, for example `record MinMax(int min, int max) {}`.

---

## 6. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Overloading by return type only | Not allowed. |
| Calling an instance method from `main` without an object | Compile error: "non-static method cannot be referenced from a static context". |
| Missing base case in recursion | `StackOverflowError`. |
| Base case that is never reached (`factorial(-1)` with `n == 0` check) | Use `n <= 1` or validate input. |
| Using `int` for factorial | Overflows at 13!. Use `long` (up to 20!) or `BigInteger`. |
| Thinking recursion is always better | Loops are usually faster and use less memory. |
| Thinking `print(null)` always works with overloads | Can be ambiguous if two unrelated reference types match. |

---

## 7. Quick Revision

- Method = named reusable block; signature = **name + parameter types** (not return type).
- Static method → class-level, no `this`; instance method → needs an object.
- Each call → new **stack frame**; popped on return.
- **Overloading** = same name, different parameters (number/type/order); **compile-time polymorphism**.
- Overload resolution order: **exact → widening → boxing → varargs**; most specific type wins.
- **Recursion** = method calls itself; needs **base case** + **recursive case**.
- Recursion uses O(depth) stack memory; too deep → `StackOverflowError` (an Error).
- Recursive Fibonacci O(2ⁿ) → memoization/iteration O(n).
- Java has **no tail-call optimisation**.

---

[← Previous: Control Statements and Loops](07-Control-Statements-and-Loops.md) | [Back to README](../README.md) | [Next: Command-Line Arguments and Foundations →](09-Command-Line-Arguments-and-Foundations.md)
