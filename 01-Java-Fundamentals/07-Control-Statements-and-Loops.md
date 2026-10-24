# Chapter 1.7 — Control Statements and Loops

[← Previous: Operators](06-Operators.md) | [Back to README](../README.md) | [Next: Methods, Overloading and Recursion →](08-Methods-Overloading-Recursion.md)

---

## What You Will Learn

- Decision-making statements: `if`, `if-else`, `else-if` ladder, nested `if`
- `switch` statement, fall-through, and the modern **switch expression** (Java 14)
- Loops: `for`, `while`, `do-while`, enhanced `for-each`
- Jump statements: `break`, `continue`, `return`, and **labelled** break/continue
- Choosing the right loop, infinite loops, and classic pattern programs

---

## 1. Control Statements — Overview

### 1.1 Interview Definition

> **Control statements decide the order in which statements in a program are executed. By default, Java runs statements one after another from top to bottom. Control statements change this flow. Java has three types: decision-making (selection) statements such as `if`, `if-else` and `switch`, which choose a path based on a condition; looping (iteration) statements such as `for`, `while`, `do-while` and the enhanced for loop, which repeat a block of code; and jump statements such as `break`, `continue` and `return`, which transfer control to another part of the program.**

| Category | Statements |
|---|---|
| Decision making | `if`, `if-else`, `else-if` ladder, nested `if`, `switch` |
| Looping | `for`, `while`, `do-while`, enhanced `for` (for-each) |
| Jump | `break`, `continue`, `return` |

### 1.2 Real-World Example

Your daily routine is a program:
- **Decision:** *If* it is raining, take an umbrella; *else* wear sunglasses.
- **Loop:** *Repeat* attending lectures *for* 6 periods.
- **Jump:** If the teacher is absent, *skip* (continue) that period; if there is a fire alarm, *leave* (break) immediately.

---

## 2. Decision-Making Statements

### 2.1 `if`, `if-else`, `else-if` Ladder, Nested `if`

```java
public class IfDemo {
    public static void main(String[] args) {
        int marks = 82;

        // simple if
        if (marks >= 40) {
            System.out.println("Result: Pass");
        }

        // if-else
        if (marks % 2 == 0) {
            System.out.println(marks + " is even");
        } else {
            System.out.println(marks + " is odd");
        }

        // else-if ladder: first true condition wins, the rest are skipped
        if (marks >= 90) {
            System.out.println("Grade: A");
        } else if (marks >= 75) {
            System.out.println("Grade: B");
        } else if (marks >= 60) {
            System.out.println("Grade: C");
        } else {
            System.out.println("Grade: D");
        }

        // nested if
        int age = 20;
        boolean hasId = true;
        if (age >= 18) {
            if (hasId) {
                System.out.println("Allowed to vote");
            } else {
                System.out.println("Bring your ID");
            }
        } else {
            System.out.println("Too young to vote");
        }
    }
}
```

**Expected Output:**

```text
Result: Pass
82 is even
Grade: B
Allowed to vote
```

**Explanation:**
- In the ladder, `82 >= 90` is false, `82 >= 75` is true → prints "Grade: B" and **skips** the remaining checks. The order of conditions matters — always check the most restrictive first.
- The condition in `if` must be a `boolean`. `if (marks)` is a compile error in Java.
- Braces `{}` are optional for a single statement, but **always use them** to avoid bugs like the "dangling else".

### 2.2 The Dangling-Else Trap

<!-- skip-check -->
```java
int a = 5, b = -1;
if (a > 0)
    if (b > 0)
        System.out.println("both positive");
else
    System.out.println("a is not positive");   // misleading indentation!
```

This prints **"a is not positive"** even though `a` is positive. An `else` always attaches to the **nearest** unmatched `if` (here, `if (b > 0)`), regardless of indentation. Braces fix this.

---

## 3. The `switch` Statement

### 3.1 Interview Definition

> **The `switch` statement selects one block of code to execute from many options, based on the value of a single expression. The expression is compared with each `case` label, and execution starts from the matching case. In the traditional switch, execution continues into the following cases ("fall-through") until a `break` is found. A `default` case runs when no case matches. Switch works with `byte`, `short`, `char`, `int`, their wrapper classes, `String` (Java 7+) and enums; it does not work with `long`, `float`, `double` or `boolean`. Since Java 14, switch can also be used as an expression with arrow (`->`) labels, which do not fall through.**

### 3.2 Understand the Concept

**Rules of the classic switch:**
1. Allowed types: `byte`, `short`, `char`, `int`, `Byte`, `Short`, `Character`, `Integer`, `String` (Java 7), `enum`. (Java 21 adds pattern matching for any reference type.)
2. Not allowed: `long`, `float`, `double`, `boolean`.
3. Case labels must be **compile-time constants** and **unique**.
4. Without `break`, execution **falls through** to the next case.
5. `default` is optional and can be placed anywhere (but put it last for readability).
6. Switching on a `null` `String` or wrapper throws `NullPointerException` (unless Java 21 `case null` is used).

**Why is switch useful?** For many fixed values (menu options, days, commands), switch is more readable than a long `else-if` ladder, and the compiler can optimise it into a jump table.

### 3.3 Java Example — Classic Switch and Fall-Through

```java
public class SwitchDemo {
    public static void main(String[] args) {
        int day = 3;
        switch (day) {
            case 1:
                System.out.println("Monday");
                break;
            case 2:
                System.out.println("Tuesday");
                break;
            case 3:
                System.out.println("Wednesday");
                break;
            default:
                System.out.println("Other day");
        }

        // Fall-through: no break after case 2
        int option = 2;
        switch (option) {
            case 1:
                System.out.println("Option 1");
            case 2:
                System.out.println("Option 2");
            case 3:
                System.out.println("Option 3");
                break;
            case 4:
                System.out.println("Option 4");
        }

        // Grouping cases intentionally using fall-through
        char grade = 'B';
        switch (grade) {
            case 'A':
            case 'B':
                System.out.println("Excellent or good");
                break;
            case 'C':
                System.out.println("Average");
                break;
            default:
                System.out.println("Needs improvement");
        }

        // switch on String (Java 7+), uses equals() internally
        String command = "stop";
        switch (command) {
            case "start":
                System.out.println("Starting");
                break;
            case "stop":
                System.out.println("Stopping");
                break;
            default:
                System.out.println("Unknown command");
        }
    }
}
```

**Expected Output:**

```text
Wednesday
Option 2
Option 3
Excellent or good
Stopping
```

**Explanation:**
- `option = 2` matches `case 2`, prints "Option 2", then **falls through** to `case 3` (no `break`), prints "Option 3", and stops at its `break`.
- Grouping `case 'A': case 'B':` uses fall-through intentionally to share code.
- A `String` switch uses `hashCode()` and `equals()` internally, so it is case-sensitive.

### 3.4 Switch Expressions (Java 14+)

```java
public class SwitchExpressionDemo {
    enum Day { MONDAY, TUESDAY, SATURDAY, SUNDAY }

    public static void main(String[] args) {
        int day = 6;
        String type = switch (day) {
            case 1, 2, 3, 4, 5 -> "Weekday";       // multiple labels, no fall-through
            case 6, 7 -> "Weekend";
            default -> "Invalid";
        };
        System.out.println(day + " is a " + type);

        Day today = Day.SUNDAY;
        int workHours = switch (today) {
            case MONDAY, TUESDAY -> 8;
            case SATURDAY -> {
                System.out.println("Half day");
                yield 4;                              // yield returns a value from a block
            }
            case SUNDAY -> 0;
        };                                            // no default needed: all enum values covered
        System.out.println("Work hours on " + today + ": " + workHours);

        // Java 21: pattern matching in switch
        Object value = 42;
        String description = switch (value) {
            case Integer i when i > 40 -> "Large integer " + i;
            case Integer i -> "Small integer " + i;
            case String s -> "String of length " + s.length();
            default -> "Something else";
        };
        System.out.println(description);
    }
}
```

**Expected Output:**

```text
6 is a Weekend
Work hours on SUNDAY: 0
Large integer 42
```

**Key points of switch expressions:**
- **Arrow labels (`->`) never fall through**, so no `break` is needed.
- Multiple labels in one case: `case 1, 2, 3 ->`.
- The switch **returns a value**, so it must be **exhaustive** (cover every possible value, usually with `default`). For enums, covering all constants is enough.
- `yield` returns a value from a block `{ }`.
- **Java 21** adds **pattern matching for switch**: `case Integer i when i > 40 ->` checks the type and an extra condition (a *guard*).

### 3.5 Interview Answer — "Explain the switch statement"

> "The switch statement is used to choose one of many code blocks based on the value of an expression. The value is compared with each case label, and when it matches, execution starts from that case. In the traditional switch we must use `break`; otherwise execution falls through into the next case. The `default` case runs if no case matches. Switch supports `int`, `char`, `byte`, `short`, their wrappers, `String` from Java 7, and enums, but not `long`, `float`, `double` or `boolean`. From Java 14, we have switch expressions with the arrow syntax, which do not fall through, can return a value, and allow multiple labels like `case 1, 2, 3`. Java 21 added pattern matching in switch, so we can switch on types."

### 3.6 `if-else` vs `switch`

| Point | `if-else` | `switch` |
|---|---|---|
| Condition type | Any boolean expression (ranges, `&&`, method calls) | Equality against constant values |
| Data types | Any | int-family, char, String, enum (+ patterns in Java 21) |
| Readability for many fixed values | Lower | Higher |
| Performance for many cases | Checks conditions one by one | Can use jump table / hash lookup |
| Fall-through | No | Yes (classic), No (arrow form) |

---

## 4. Loops

### 4.1 Interview Definition

> **A loop is a control statement that executes a block of code repeatedly as long as a condition remains true. Java provides four loops: the `for` loop, used when the number of iterations is known; the `while` loop, an entry-controlled loop used when the number of iterations is not known in advance; the `do-while` loop, an exit-controlled loop that executes the body at least once; and the enhanced `for` (for-each) loop, used to traverse arrays and collections without using an index.**

### 4.2 Understand Each Loop

```text
for (initialisation; condition; update) {   // 1. init once
    body                                    // 2. check condition → 3. body → 4. update → back to 2
}

while (condition) {                         // condition checked BEFORE each iteration
    body                                    // (entry-controlled, may run 0 times)
}

do {
    body                                    // body runs FIRST
} while (condition);                        // condition checked AFTER (exit-controlled, runs ≥ 1 time)
                                            // note the semicolon!

for (Type item : arrayOrCollection) {       // for-each: no index, read-only traversal
    body
}
```

| Loop | When to use | Minimum executions | Type |
|---|---|---|---|
| `for` | Known number of iterations (counting) | 0 | Entry-controlled |
| `while` | Unknown iterations; depends on a condition (reading input until "quit") | 0 | Entry-controlled |
| `do-while` | Body must run at least once (menus, input validation) | **1** | Exit-controlled |
| for-each | Visiting every element of an array/collection | 0 | Entry-controlled |

### 4.3 Real-World Example

- **for** — Doing exactly 20 push-ups. You know the count.
- **while** — Eating until you are full. You don't know how many bites.
- **do-while** — Tasting the soup and then deciding whether to add salt. You taste at least once.
- **for-each** — Checking every student's ID card in a queue, one by one.

### 4.4 Java Example — All Four Loops

```java
import java.util.List;

public class LoopsDemo {
    public static void main(String[] args) {
        // for loop: multiplication table
        System.out.print("Table of 5: ");
        for (int i = 1; i <= 5; i++) {
            System.out.print(5 * i + " ");
        }
        System.out.println();

        // while loop: sum of digits (number of iterations unknown)
        int number = 4721;
        int sum = 0;
        while (number > 0) {
            sum += number % 10;    // take last digit
            number /= 10;          // remove last digit
        }
        System.out.println("Sum of digits of 4721 = " + sum);

        // do-while: runs at least once even though the condition is false
        int count = 10;
        do {
            System.out.println("do-while body ran with count = " + count);
            count++;
        } while (count < 5);

        // while with the same false condition runs zero times
        int count2 = 10;
        while (count2 < 5) {
            System.out.println("This never prints");
        }

        // for-each over an array and a List
        int[] scores = {70, 85, 90};
        int total = 0;
        for (int s : scores) {
            total += s;
        }
        System.out.println("Total score = " + total);

        List<String> languages = List.of("Java", "Python", "Go");
        for (String lang : languages) {
            System.out.print(lang + " ");
        }
        System.out.println();

        // for loop with multiple variables
        for (int i = 0, j = 10; i < j; i += 3, j -= 3) {
            System.out.println("i = " + i + ", j = " + j);
        }
    }
}
```

### 4.5 Code Explanation

- **for:** `i` starts at 1; the condition `i <= 5` is checked; the body prints; then `i++`. Repeats until `i` is 6.
- **while (sum of digits) dry run for 4721:**

| Iteration | number % 10 | sum | number / 10 |
|---|---|---|---|
| 1 | 1 | 1 | 472 |
| 2 | 2 | 3 | 47 |
| 3 | 7 | 10 | 4 |
| 4 | 4 | 14 | 0 → loop stops |

- **do-while:** body runs once with `count = 10`, then `10 < 5` is false → stops. A `while` loop with the same condition runs **zero** times.
- **for-each:** `s` takes each value of the array in order. You cannot get the index directly.
- **Multiple variables:** both `i` and `j` are declared and updated in one `for` header.

### 4.6 Expected Output

```text
Table of 5: 5 10 15 20 25 
Sum of digits of 4721 = 14
do-while body ran with count = 10
Total score = 245
Java Python Go 
i = 0, j = 10
i = 3, j = 7
```

### 4.7 Interview Answer — "Difference between while and do-while"

> "Both loops repeat code while a condition is true. The difference is **when** the condition is checked. `while` is an **entry-controlled** loop — it checks the condition first, so if the condition is false initially, the body does not run even once. `do-while` is an **exit-controlled** loop — it runs the body first and then checks the condition, so the body always runs **at least once**. `do-while` is useful for menu-driven programs or input validation, where we must show the menu or read input at least once. Also, a `do-while` statement ends with a semicolon after the condition."

### 4.8 For-each Limitations

The enhanced for loop is clean, but:
1. **No index** — you cannot know the position directly.
2. **Cannot modify an array element through the loop variable** — `s = 0` changes only the local copy.
3. **Cannot remove from a collection** while iterating — this throws `ConcurrentModificationException`. Use an `Iterator` with `iterator.remove()` or `removeIf()`.
4. Only **forward** traversal.

```java
public class ForEachLimit {
    public static void main(String[] args) {
        int[] arr = {1, 2, 3};
        for (int x : arr) {
            x = x * 10;              // changes only the copy
        }
        System.out.println("After for-each: " + arr[0] + " " + arr[1] + " " + arr[2]);

        for (int i = 0; i < arr.length; i++) {
            arr[i] = arr[i] * 10;    // changes the actual array
        }
        System.out.println("After index loop: " + arr[0] + " " + arr[1] + " " + arr[2]);
    }
}
```

**Expected Output:**

```text
After for-each: 1 2 3
After index loop: 10 20 30
```

---

## 5. Jump Statements: `break`, `continue`, `return`, Labels

### 5.1 Interview Definition

> **Jump statements transfer control to another part of the program. `break` immediately terminates the nearest enclosing loop or switch. `continue` skips the remaining statements of the current iteration and moves to the next iteration of the loop. `return` exits from the current method, optionally returning a value. Java also supports labelled `break` and `continue`, which allow control to exit or continue an outer loop from inside a nested loop.**

### 5.2 Java Example

```java
public class JumpDemo {
    static int findFirstNegative(int[] arr) {
        for (int i = 0; i < arr.length; i++) {
            if (arr[i] < 0) {
                return i;               // exits the whole method
            }
        }
        return -1;
    }

    public static void main(String[] args) {
        // break: stop at the first multiple of 7
        for (int i = 1; i <= 20; i++) {
            if (i % 7 == 0) {
                System.out.println("First multiple of 7: " + i);
                break;
            }
        }

        // continue: skip odd numbers
        System.out.print("Even numbers: ");
        for (int i = 1; i <= 10; i++) {
            if (i % 2 != 0) {
                continue;
            }
            System.out.print(i + " ");
        }
        System.out.println();

        // labelled break: exit BOTH loops
        outer:
        for (int i = 1; i <= 3; i++) {
            for (int j = 1; j <= 3; j++) {
                if (i * j == 4) {
                    System.out.println("Found i*j == 4 at i=" + i + ", j=" + j);
                    break outer;
                }
            }
        }

        // labelled continue: skip to the next iteration of the OUTER loop
        rows:
        for (int i = 1; i <= 3; i++) {
            for (int j = 1; j <= 3; j++) {
                if (j == 2) {
                    continue rows;
                }
                System.out.println("i=" + i + ", j=" + j);
            }
        }

        System.out.println("Index of first negative: " + findFirstNegative(new int[]{4, 9, -2, 7}));
    }
}
```

### 5.3 Expected Output

```text
First multiple of 7: 7
Even numbers: 2 4 6 8 10 
Found i*j == 4 at i=2, j=2
i=1, j=1
i=2, j=1
i=3, j=1
Index of first negative: 2
```

### 5.4 Explanation

- `break` exits only the **innermost** loop. `break outer;` exits the loop labelled `outer`.
- With `continue rows;`, when `j == 2`, the inner loop stops and the **outer** loop moves to its next `i`. So only `j=1` prints for each row.
- `return i;` ends the method immediately — the remaining loop iterations never run.
- Java has `goto` as a reserved word but **does not support it**. Labelled break/continue are the safe alternative.

---

## 6. Infinite Loops

```text
for (;;) { ... }            // all three parts are optional
while (true) { ... }
do { ... } while (true);
```

Infinite loops are used in servers, game loops and menu programs, and must contain a `break` or `return` to stop. A loop like `for (int i = 0; i < 10; i--)` is an *accidental* infinite loop (until overflow).

**Unreachable code:** `while (false) { ... }` is a **compile error** ("unreachable statement"), but `if (false) { ... }` is allowed (it is used for conditional compilation). Also, a statement directly after `while (true) { }` with no `break` is unreachable → compile error.

---

## 7. Pattern Programs (Very Common in Fresher Interviews)

```java
public class Patterns {
    public static void main(String[] args) {
        int n = 4;

        System.out.println("Right triangle:");
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= i; j++) {
                System.out.print("* ");
            }
            System.out.println();
        }

        System.out.println("Pyramid:");
        for (int i = 1; i <= n; i++) {
            for (int s = 1; s <= n - i; s++) {
                System.out.print(" ");
            }
            for (int j = 1; j <= i; j++) {
                System.out.print("* ");
            }
            System.out.println();
        }

        System.out.println("Number triangle:");
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= i; j++) {
                System.out.print(j);
            }
            System.out.println();
        }

        System.out.println("Floyd's triangle:");
        int num = 1;
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= i; j++) {
                System.out.print(num++ + " ");
            }
            System.out.println();
        }
    }
}
```

**Expected Output:**

```text
Right triangle:
* 
* * 
* * * 
* * * * 
Pyramid:
   * 
  * * 
 * * * 
* * * * 
Number triangle:
1
12
123
1234
Floyd's triangle:
1 
2 3 
4 5 6 
7 8 9 10 
```

**How to think about any pattern:**
1. The **outer loop** controls the **rows** (`i` from 1 to n).
2. The **inner loop(s)** control what is printed in each row (columns), usually depending on `i`.
3. For a pyramid, first print `n - i` spaces, then `i` stars.
4. After each row, print a new line.

---

## 8. Interview Questions with Complete Answers

### Basic Level

**Q1. What is the difference between `break` and `continue`?**

`break` **terminates** the loop completely and moves control to the statement after the loop. `continue` **skips only the current iteration** and moves to the next iteration (for a `for` loop, it jumps to the update part). Example: in a loop from 1 to 5, `if (i == 3) break;` prints 1 2, but `if (i == 3) continue;` prints 1 2 4 5. `break` is also used in `switch`; `continue` cannot be used in a `switch` unless the switch is inside a loop.

**Q2. Which data types are not allowed in a switch?**

`long`, `float`, `double` and `boolean` are not allowed in a classic switch. `long` is excluded because switch labels are compiled to `int`-based jump tables; `float`/`double` are excluded because floating-point equality is unreliable; `boolean` is excluded because `if-else` is enough for two values.

**Q3. When would you use a `do-while` loop?**

When the loop body must run **at least once** before checking the condition. Typical examples are a menu-driven program (show the menu, read the choice, repeat until "exit") and input validation (ask for input, repeat while the input is invalid).

### Intermediate Level

**Q4. What is fall-through in a switch statement?**

When a case matches and there is no `break` at the end of its statements, execution continues into the next case's statements, even if that case does not match. This is fall-through. It is sometimes useful for grouping cases, but forgetting a `break` is a common bug. The arrow form (`case X ->`) introduced in Java 14 does not fall through.

**Q5. What is the difference between a switch statement and a switch expression?**

| Switch statement | Switch expression (Java 14+) |
|---|---|
| Executes statements; does not produce a value | Produces a value: `String s = switch (x) { ... };` |
| Uses `case X:` with `break` | Usually uses `case X ->` (no fall-through) |
| Need not be exhaustive | **Must be exhaustive** (cover all values or have `default`) |
| Returns from block not possible | `yield` returns a value from a block |

**Q6. Can we use `String` in switch? How does it work internally?**

Yes, since Java 7. The compiler converts it into a switch on `str.hashCode()`, and then uses `str.equals()` to confirm the exact match (because different strings can share a hash code). It is case-sensitive, and a `null` string throws `NullPointerException`.

**Q7. How do you break out of nested loops?**

Use a **labelled break**: put a label like `outer:` before the outer loop and write `break outer;` inside the inner loop. Alternatives are using a boolean flag checked by the outer loop, or moving the loops into a method and using `return`.

### Advanced Level

**Q8. Is `for(;;)` valid? What does it do?**

Yes. All three parts of a `for` header are optional. With no condition, the condition is treated as `true`, so it is an **infinite loop**. It is equivalent to `while (true)`. It must contain a `break`, `return` or exception to end.

**Q9. Why does `while(false) { }` give a compile error but `if(false) { }` does not?**

The Java Language Specification has special "unreachable code" rules. For `while (false)`, the body can never execute, so it is reported as an **unreachable statement** error. The `if` statement is deliberately excluded from this rule so that developers can use `if (DEBUG) { ... }` with a constant flag for conditional code.

**Q10. What is the output?**

```java
public class LoopTrick {
    public static void main(String[] args) {
        int i = 0;
        for (System.out.print("A"); i < 2; System.out.print("C")) {
            System.out.print("B");
            i++;
        }
        System.out.println();
        int x = 0;
        while (x++ < 3);
        System.out.println("x = " + x);
    }
}
```

**Expected Output:**

```text
ABCBC
x = 4
```

- The `for` initialisation can be any statement: it prints "A" once. Then: condition true → body "B" → update "C" → condition true → "B" → "C" → condition false.
- `while (x++ < 3);` has an **empty body** (the semicolon). Comparisons: 0<3 (x→1), 1<3 (x→2), 2<3 (x→3), 3<3 false (x→4). So `x` is 4.

---

## 9. Follow-Up Questions

**F1. Which loop is fastest?**

For normal code, all loops compile to similar bytecode, so performance is about the same. Choose the loop that makes the code most readable. For collections, the for-each loop uses an `Iterator`; for `ArrayList`, an index-based loop and for-each are similar, but for `LinkedList`, index-based access (`list.get(i)`) is slow (O(n) per call), so for-each is much better.

**F2. Can you declare a variable inside a loop? Is it created again every time?**

Yes. The variable's scope is limited to the loop body. Logically, it is a new variable each iteration, but the compiler reuses the same stack slot, so there is no performance cost. Declaring variables in the smallest possible scope is a good practice.

**F3. What happens if you forget the update in a `for` loop?**

If the condition never becomes false, the loop runs forever (infinite loop). For example, `for (int i = 0; i < 5; ) { }` never ends.

---

## 10. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Missing `break` in classic switch | Causes fall-through. Use arrow syntax in modern Java. |
| `while (x < 5);` with a semicolon | Creates an empty loop body — often infinite. |
| Forgetting `;` after `do-while` | Compile error: `} while (cond);` |
| Wrong order in else-if ladder (`>= 60` before `>= 90`) | Earlier condition catches everything. Put the most specific first. |
| Modifying a collection inside for-each | `ConcurrentModificationException`. Use `Iterator.remove()` or `removeIf()`. |
| Thinking `continue` works in `switch` alone | Only inside a loop. |
| Switching on `long`/`double` | Not allowed. |

---

## 11. Quick Revision

- **Decision:** `if`, `if-else`, ladder (first true wins), nested; `else` binds to nearest `if`.
- **switch:** int-family, char, String (7), enum; no long/float/double/boolean; fall-through without `break`.
- **Switch expression (14):** `->`, no fall-through, multiple labels, returns value, exhaustive, `yield`.
- **Pattern switch (21):** `case Integer i when i > 0 ->`.
- **Loops:** `for` (known count), `while` (entry-controlled, 0+ times), `do-while` (exit-controlled, 1+ times), for-each (no index, read-only, no removal).
- **Jumps:** `break` (exit loop), `continue` (next iteration), `return` (exit method), labels for outer loops. No `goto`.
- `for(;;)` = infinite loop. `while(false)` = compile error.
- Patterns: outer loop = rows, inner loop = columns.

---

[← Previous: Operators](06-Operators.md) | [Back to README](../README.md) | [Next: Methods, Overloading and Recursion →](08-Methods-Overloading-Recursion.md)
