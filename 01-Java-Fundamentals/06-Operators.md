# Chapter 1.6 — Operators

[← Previous: Type Casting](05-Type-Casting.md) | [Back to README](../README.md) | [Next: Control Statements and Loops →](07-Control-Statements-and-Loops.md)

---

## What You Will Learn

- All operator categories in Java with examples
- Pre-increment vs post-increment (a favourite interview topic)
- Integer division and the modulus operator with negative numbers
- Short-circuit `&&`/`||` vs non-short-circuit `&`/`|`
- Bitwise and shift operators (`>>` vs `>>>`)
- The ternary operator and `instanceof`
- Operator precedence and associativity
- `==` vs `equals()` (introduction)

---

## 1. Operators — Overview

### 1.1 Interview Definition

> **An operator is a symbol that performs an operation on one, two or three operands and produces a result. Java operators are grouped into arithmetic, unary, assignment, relational (comparison), logical, bitwise, shift, ternary (conditional) and `instanceof` operators. When several operators appear in one expression, operator precedence decides which operation is done first, and associativity decides the order when operators have the same precedence.**

### 1.2 Operator Categories

| Category | Operators | Example |
|---|---|---|
| Arithmetic | `+ - * / %` | `a + b`, `a % b` |
| Unary | `+ - ++ -- !` and `~` | `++i`, `-x`, `!flag` |
| Assignment | `= += -= *= /= %= &= \|= ^= <<= >>= >>>=` | `x += 5` |
| Relational | `== != > < >= <=` | `a > b` |
| Logical | `&& \|\| !` | `a > 0 && b > 0` |
| Bitwise | `& \| ^ ~` | `a & b` |
| Shift | `<< >> >>>` | `a << 2` |
| Ternary | `? :` | `a > b ? a : b` |
| Type comparison | `instanceof` | `obj instanceof String` |

Based on the number of operands:
- **Unary** — one operand (`++i`, `!flag`)
- **Binary** — two operands (`a + b`)
- **Ternary** — three operands (`cond ? x : y`) — Java has only one ternary operator.

---

## 2. Arithmetic Operators

### 2.1 Understand the Concept

| Operator | Meaning | Note |
|---|---|---|
| `+` | Addition (also String concatenation) | `"a" + 1` → `"a1"` |
| `-` | Subtraction | |
| `*` | Multiplication | |
| `/` | Division | `int / int` → **integer division** (decimal part removed) |
| `%` | Modulus (remainder) | The result has the **sign of the left operand** |

**Division by zero:**
- `int / 0` → throws `ArithmeticException: / by zero`.
- `double / 0` → `Infinity` (or `-Infinity`), and `0.0 / 0` → `NaN`. No exception.

### 2.2 Java Example

```java
public class ArithmeticDemo {
    public static void main(String[] args) {
        int a = 17, b = 5;
        System.out.println("a + b = " + (a + b));
        System.out.println("a - b = " + (a - b));
        System.out.println("a * b = " + (a * b));
        System.out.println("a / b = " + (a / b));        // integer division
        System.out.println("a % b = " + (a % b));        // remainder
        System.out.println("17.0 / 5 = " + (17.0 / 5));

        // Modulus with negative numbers: sign follows the LEFT operand
        System.out.println("-17 % 5 = " + (-17 % 5));
        System.out.println("17 % -5 = " + (17 % -5));
        System.out.println("5.5 % 2 = " + (5.5 % 2));    // % works on decimals too

        // Division by zero
        System.out.println("10.0 / 0 = " + (10.0 / 0));
        System.out.println("0.0 / 0 = " + (0.0 / 0));
        try {
            System.out.println(10 / 0);
        } catch (ArithmeticException e) {
            System.out.println("10 / 0 -> " + e);
        }

        // + with Strings is evaluated left to right
        System.out.println(10 + 20 + "Java");
        System.out.println("Java" + 10 + 20);
        System.out.println("Java" + (10 + 20));
    }
}
```

### 2.3 Code Explanation

- `17 / 5` → `3` because both are `int`; the decimal `.4` is discarded.
- `17 % 5` → `2` (17 = 5×3 + 2).
- `-17 % 5` → `-2`; `17 % -5` → `2`. The rule: `a % b` has the same sign as `a`.
- `5.5 % 2` → `1.5`; modulus works with floating-point numbers too.
- Floating-point division by zero does not throw; integer division by zero does.
- `10 + 20 + "Java"` → left to right: `30` first, then `"30Java"`.
- `"Java" + 10 + 20` → `"Java10"` then `"Java1020"`.

### 2.4 Expected Output

```text
a + b = 22
a - b = 12
a * b = 85
a / b = 3
a % b = 2
17.0 / 5 = 3.4
-17 % 5 = -2
17 % -5 = 2
5.5 % 2 = 1.5
10.0 / 0 = Infinity
0.0 / 0 = NaN
10 / 0 -> java.lang.ArithmeticException: / by zero
30Java
Java1020
Java30
```

---

## 3. Unary Operators — Increment and Decrement

### 3.1 Interview Definition

> **The increment (`++`) and decrement (`--`) operators increase or decrease a variable's value by one. In pre-increment (`++x`), the variable is increased first and the new value is used in the expression. In post-increment (`x++`), the current (old) value is used in the expression first, and the variable is increased afterwards. Both forms change the variable by the same amount; they differ only in which value the expression produces.**

### 3.2 Understand the Concept

| Expression | Steps | If `x = 5`, value of expression | `x` afterwards |
|---|---|---|---|
| `++x` | increment, then use | 6 | 6 |
| `x++` | use, then increment | 5 | 6 |
| `--x` | decrement, then use | 4 | 4 |
| `x--` | use, then decrement | 5 | 4 |

Technically, for `x++` Java: (1) saves the old value in a temporary, (2) increments `x`, (3) returns the saved old value.

### 3.3 Real-World Example

- **Pre-increment** is like a shop that **raises the price first, then sells** — the customer pays the new price.
- **Post-increment** is like a shop that **sells at the current price, then raises it** — the customer pays the old price; the next customer pays the new one.

### 3.4 Java Example

```java
public class IncrementDemo {
    public static void main(String[] args) {
        int x = 5;
        int y = x++;      // y gets 5, then x becomes 6
        System.out.println("x = " + x + ", y = " + y);

        int p = 5;
        int q = ++p;      // p becomes 6, then q gets 6
        System.out.println("p = " + p + ", q = " + q);

        int a = 10;
        int result = a++ + ++a;    // 10 + 12
        System.out.println("a = " + a + ", result = " + result);

        int i = 5;
        i = i++;          // famous trick question
        System.out.println("i after i = i++ : " + i);

        int m = 3;
        int n = m-- - --m;         // 3 - 1
        System.out.println("m = " + m + ", n = " + n);

        boolean flag = true;
        System.out.println("!flag = " + !flag);
        int bits = 5;
        System.out.println("~5 = " + ~bits);    // bitwise NOT: -(5+1)
    }
}
```

### 3.5 Code Explanation (Dry Run)

**`int result = a++ + ++a;` with `a = 10`**
1. `a++` → value used = **10**, then `a` becomes 11.
2. `++a` → `a` becomes 12, value used = **12**.
3. `result = 10 + 12 = 22`; final `a = 12`.

**`i = i++;` with `i = 5`**
1. `i++` saves old value 5, increments `i` to 6, returns 5.
2. Assignment `i = 5` overwrites the 6.
3. Final `i = 5`. The increment is "lost". (Never write this in real code.)

**`int n = m-- - --m;` with `m = 3`**
1. `m--` → value used = 3, `m` becomes 2.
2. `--m` → `m` becomes 1, value used = 1.
3. `n = 3 - 1 = 2`; final `m = 1`.

**`~5`** → bitwise NOT flips all bits. For any int, `~x = -(x + 1)`, so `~5 = -6`.

### 3.6 Expected Output

```text
x = 6, y = 5
p = 6, q = 6
a = 12, result = 22
i after i = i++ : 5
m = 1, n = 2
!flag = false
~5 = -6
```

### 3.7 Interview Answer — "Difference between `i++` and `++i`"

> "Both `i++` and `++i` increase the value of `i` by one. The difference is the value the expression returns. `++i` is **pre-increment**: it first increments `i` and then uses the new value. `i++` is **post-increment**: it first uses the current value and then increments. For example, if `i` is 5, `int a = ++i` makes `a` 6 and `i` 6, but `int a = i++` makes `a` 5 and `i` 6. When used as a separate statement like `i++;` in a for loop, both behave the same. A tricky case is `i = i++`, which leaves `i` unchanged because the old value is assigned back."

---

## 4. Relational and Logical Operators

### 4.1 Relational Operators

`==`, `!=`, `>`, `<`, `>=`, `<=` compare two values and return a `boolean`.

- For **primitives**, `==` compares values.
- For **objects**, `==` compares **references** (are they the same object?). To compare content, use `.equals()`. (Covered in [Part 2, Chapter 2](../02-Strings-and-Core-Programming/02-String-Comparison-and-Methods.md).)

### 4.2 Logical Operators and Short-Circuiting

| Operator | Name | Result true when | Short-circuit? |
|---|---|---|---|
| `&&` | Logical AND | both are true | **Yes** — if left is false, right is **not evaluated** |
| `\|\|` | Logical OR | at least one is true | **Yes** — if left is true, right is **not evaluated** |
| `!` | Logical NOT | operand is false | — |
| `&` | Boolean AND (non-short-circuit) | both are true | No — both sides always evaluated |
| `\|` | Boolean OR (non-short-circuit) | at least one is true | No |
| `^` | XOR | exactly one is true | No |

### 4.3 Interview Definition — Short-Circuit Evaluation

> **Short-circuit evaluation means that the logical operators `&&` and `||` stop evaluating as soon as the final result is known. With `&&`, if the left operand is false, the right operand is skipped because the result must be false. With `||`, if the left operand is true, the right operand is skipped because the result must be true. This improves performance and is commonly used to prevent errors, such as checking `obj != null && obj.method()`.**

### 4.4 Java Example

```java
public class LogicalDemo {
    static boolean check(String name, boolean value) {
        System.out.println("  evaluating " + name);
        return value;
    }

    public static void main(String[] args) {
        System.out.println("Using && (short-circuit):");
        boolean r1 = check("A", false) && check("B", true);
        System.out.println("  result = " + r1);

        System.out.println("Using & (no short-circuit):");
        boolean r2 = check("A", false) & check("B", true);
        System.out.println("  result = " + r2);

        System.out.println("Using || (short-circuit):");
        boolean r3 = check("A", true) || check("B", false);
        System.out.println("  result = " + r3);

        // Practical use: avoid NullPointerException
        String name = null;
        if (name != null && name.length() > 3) {
            System.out.println("Long name");
        } else {
            System.out.println("Name is null or short - no exception thrown");
        }

        int x = 5;
        boolean r4 = (x > 10) & (++x > 0);   // ++x still runs
        System.out.println("x after & : " + x + ", r4 = " + r4);
        int y = 5;
        boolean r5 = (y > 10) && (++y > 0);  // ++y skipped
        System.out.println("y after && : " + y + ", r5 = " + r5);

        System.out.println("true ^ false = " + (true ^ false));
        System.out.println("true ^ true = " + (true ^ true));
    }
}
```

### 4.5 Expected Output

```text
Using && (short-circuit):
  evaluating A
  result = false
Using & (no short-circuit):
  evaluating A
  evaluating B
  result = false
Using || (short-circuit):
  evaluating A
  result = true
Name is null or short - no exception thrown
x after & : 6, r4 = false
y after && : 5, r5 = false
true ^ false = true
true ^ true = false
```

### 4.6 Explanation

- With `&&`, "B" is never evaluated because "A" is false.
- With `&`, both sides run, even though the result is already known.
- `name != null && name.length() > 3` is safe: if `name` is `null`, the second part is skipped, so no `NullPointerException`. With `&` it would crash.
- `x` changed to 6 with `&` (side effect ran), but `y` stayed 5 with `&&` (side effect skipped).

---

## 5. Bitwise and Shift Operators

### 5.1 Interview Definition

> **Bitwise operators (`&`, `|`, `^`, `~`) work on the individual bits of integer values. Shift operators move bits left or right: `<<` (left shift) multiplies by powers of two, `>>` (signed right shift) divides by powers of two while keeping the sign bit, and `>>>` (unsigned right shift) fills the leftmost bits with zeros, so the result is always non-negative for int and long.**

### 5.2 Truth Table

| a | b | a & b | a \| b | a ^ b |
|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 |
| 0 | 1 | 0 | 1 | 1 |
| 1 | 0 | 0 | 1 | 1 |
| 1 | 1 | 1 | 1 | 0 |

### 5.3 Java Example

```java
public class BitwiseDemo {
    public static void main(String[] args) {
        int a = 12;   // 1100
        int b = 10;   // 1010
        System.out.println("a & b = " + (a & b));   // 1000 = 8
        System.out.println("a | b = " + (a | b));   // 1110 = 14
        System.out.println("a ^ b = " + (a ^ b));   // 0110 = 6
        System.out.println("~a = " + (~a));         // -(12+1) = -13

        System.out.println("5 << 2 = " + (5 << 2));       // 5 * 4
        System.out.println("20 >> 2 = " + (20 >> 2));    // 20 / 4
        System.out.println("-20 >> 2 = " + (-20 >> 2));   // sign kept
        System.out.println("-20 >>> 28 = " + (-20 >>> 28)); // zeros fill from left

        // Practical tricks
        int n = 7;
        System.out.println(n + " is " + ((n & 1) == 0 ? "even" : "odd"));
        int p = 16;
        System.out.println(p + " is power of two? " + (p > 0 && (p & (p - 1)) == 0));

        // Swap without a temp variable using XOR
        int x = 3, y = 9;
        x = x ^ y;
        y = x ^ y;
        x = x ^ y;
        System.out.println("After XOR swap: x = " + x + ", y = " + y);

        System.out.println("Binary of 12: " + Integer.toBinaryString(12));
    }
}
```

### 5.4 Code Explanation

- `12 & 10`: `1100 & 1010 = 1000` = 8.
- `~12 = -13` because of two's complement (`~x = -x - 1`).
- `5 << 2` shifts bits two places left: `101` → `10100` = 20 (multiply by 2²).
- `-20 >> 2` = −5: the sign bit (1) is copied into the empty left positions.
- `-20 >>> 28`: zeros fill from the left. −20 in 32-bit is `1111…1110 1100`; shifting 28 places leaves `1111` = 15.
- `(n & 1) == 0` checks the last bit — 0 means even.
- `p & (p - 1)` clears the lowest set bit; it is zero only for powers of two.
- XOR swap: works because `a ^ a = 0` and `a ^ 0 = a`.

### 5.5 Expected Output

```text
a & b = 8
a | b = 14
a ^ b = 6
~a = -13
5 << 2 = 20
20 >> 2 = 5
-20 >> 2 = -5
-20 >>> 28 = 15
7 is odd
16 is power of two? true
After XOR swap: x = 9, y = 3
Binary of 12: 1100
```

---

## 6. Ternary Operator and `instanceof`

### 6.1 Ternary Operator

> **The ternary (conditional) operator `condition ? valueIfTrue : valueIfFalse` evaluates a boolean condition and returns one of two values. It is a compact replacement for a simple if-else that produces a value.**

```java
public class TernaryDemo {
    public static void main(String[] args) {
        int a = 15, b = 25, c = 20;
        int max = (a > b) ? a : b;
        System.out.println("Max of a and b = " + max);

        int largest = (a > b) ? (a > c ? a : c) : (b > c ? b : c);   // nested
        System.out.println("Largest of three = " + largest);

        int marks = 72;
        String grade = marks >= 90 ? "A" : marks >= 75 ? "B" : marks >= 60 ? "C" : "D";
        System.out.println("Grade = " + grade);

        Object obj = "Hello";
        if (obj instanceof String) {
            String s = (String) obj;
            System.out.println("Length using classic instanceof: " + s.length());
        }
        if (obj instanceof String str && str.length() > 3) {     // Java 16+ pattern matching
            System.out.println("Pattern matching: " + str.toUpperCase());
        }
        Object nothing = null;
        System.out.println("null instanceof String = " + (nothing instanceof String));
    }
}
```

**Expected Output:**

```text
Max of a and b = 25
Largest of three = 25
Grade = C
Length using classic instanceof: 5
Pattern matching: HELLO
null instanceof String = false
```

**Notes:**
- Use nested ternaries only when they stay readable.
- `instanceof` returns `false` for `null` — it never throws.
- **Pattern matching for `instanceof` (Java 16)** checks the type and creates a typed variable (`str`) in one step, removing the explicit cast.

---

## 7. Operator Precedence and Associativity

### 7.1 Table (highest to lowest)

| Level | Operators | Associativity |
|---|---|---|
| 1 | Postfix `x++ x--` | Left to right |
| 2 | Unary `++x --x + - ! ~` and cast `(type)` | **Right to left** |
| 3 | Multiplicative `* / %` | Left to right |
| 4 | Additive `+ -` | Left to right |
| 5 | Shift `<< >> >>>` | Left to right |
| 6 | Relational `< > <= >= instanceof` | Left to right |
| 7 | Equality `== !=` | Left to right |
| 8 | Bitwise AND `&` | Left to right |
| 9 | Bitwise XOR `^` | Left to right |
| 10 | Bitwise OR `\|` | Left to right |
| 11 | Logical AND `&&` | Left to right |
| 12 | Logical OR `\|\|` | Left to right |
| 13 | Ternary `? :` | **Right to left** |
| 14 | Assignment `= += -=` … | **Right to left** |

**Memory tip:** "**U**nary **M**ultiply **A**dd **S**hift **R**elate **E**qual **B**it **L**ogic **T**ernary **A**ssign" → *"Ugly Monkeys Always Steal Ripe Every Banana Late Tonight Anyway."*

### 7.2 Java Example

```java
public class PrecedenceDemo {
    public static void main(String[] args) {
        int result1 = 10 + 5 * 2;            // * first
        int result2 = (10 + 5) * 2;          // brackets first
        int result3 = 100 / 10 * 2;          // left to right: (100/10)*2
        int result4 = 10 - 4 - 3;            // left to right: (10-4)-3
        int a, b, c;
        a = b = c = 7;                       // right to left
        boolean result5 = 5 > 3 && 2 > 4 || 1 < 2;   // && before ||
        System.out.println(result1 + " " + result2 + " " + result3 + " " + result4);
        System.out.println(a + " " + b + " " + c + " " + result5);
    }
}
```

**Expected Output:**

```text
20 30 20 3
7 7 7 true
```

- `100 / 10 * 2` → `/` and `*` have equal precedence and are left-associative → `(100/10)*2 = 20` (not `100/20 = 5`).
- `a = b = c = 7` → assignment is right-associative: `c = 7`, then `b = 7`, then `a = 7`.
- `5 > 3 && 2 > 4 || 1 < 2` → `(true && false) || true` → `true`.

> **Best practice:** In real code, use **brackets** to make your intention clear instead of relying on precedence rules.

---

## 8. Interview Questions with Complete Answers

### Basic Level

**Q1. What is the difference between `/` and `%`?**

`/` gives the **quotient** and `%` gives the **remainder**. For integers, `17 / 5 = 3` and `17 % 5 = 2`. They are often used together: `%` is used to check even/odd (`n % 2 == 0`), get the last digit (`n % 10`), and wrap indexes in circular arrays.

**Q2. What is the difference between `=` and `==`?**

`=` is the **assignment** operator — it stores a value in a variable. `==` is the **equality** operator — it compares two values and returns a boolean. Writing `if (x = 5)` is a compile error in Java (because `x = 5` is an `int`, not a `boolean`), which prevents a common C bug.

**Q3. What is the difference between `&&` and `&`?**

Both perform logical AND on booleans. `&&` is **short-circuit**: if the left side is false, the right side is not evaluated. `&` **always evaluates both sides**. `&` can also be used as a **bitwise** AND on integers. In conditions we use `&&` for performance and safety, for example `obj != null && obj.isValid()`.

### Intermediate Level

**Q4. What is the output of `int i = 5; i = i++ + ++i;`?**

`i++` gives 5 (i becomes 6), `++i` makes i 7 and gives 7. So `i = 5 + 7 = 12`. The final value of `i` is **12**.

**Q5. What is the result of `-7 % 3` and `7 % -3`?**

`-7 % 3 = -1` and `7 % -3 = 1`. In Java, the result of `%` always takes the **sign of the dividend** (left operand). If you need an always-positive result (for example, for circular indexes), use `Math.floorMod(-7, 3)`, which returns `2`.

**Q6. What is the difference between `>>` and `>>>`?**

Both shift bits to the right. `>>` is the **signed** right shift: it fills the empty left bits with the **sign bit**, so negative numbers stay negative (`-8 >> 1 = -4`). `>>>` is the **unsigned** right shift: it always fills with **zeros**, so the result is non-negative for `int` and `long` (`-8 >>> 1 = 2147483644`). There is no `<<<` operator because left shift always fills with zeros anyway.

**Q7. Why does `10 + 20 + "Java"` give `30Java` but `"Java" + 10 + 20` give `Java1020`?**

The `+` operator is evaluated **left to right**. In the first case, `10 + 20` are both numbers, so it adds them to 30, then concatenates with `"Java"`. In the second case, the first operation is `"Java" + 10`, which produces a `String`, so every following `+` is concatenation.

### Advanced Level

**Q8. Can integer division by zero and floating-point division by zero behave differently?**

Yes. Integer division by zero throws `ArithmeticException: / by zero` because integers have no way to represent infinity. Floating-point arithmetic follows IEEE 754, which has special values: `5.0 / 0` gives `Infinity`, `-5.0 / 0` gives `-Infinity`, and `0.0 / 0` gives `NaN` (Not a Number). Also note `NaN == NaN` is `false`; use `Double.isNaN()`.

**Q9. How can you check if a number is even using bitwise operators, and why is it faster?**

`(n & 1) == 0` means even. The last bit of an even number is always 0. Bitwise AND is a single CPU instruction, but in modern Java the JIT usually optimises `n % 2` too, so the real benefit is small. Note: `n % 2 == 1` fails for negative odd numbers (because `-3 % 2 = -1`), while `(n & 1) == 1` works correctly for negatives.

**Q10. What does `x += 1` do differently from `x = x + 1` for a `short x`?**

`x += 1` includes an implicit cast: it is equivalent to `x = (short)(x + 1)`, so it compiles. `x = x + 1` does not compile, because `x + 1` is promoted to `int`, and assigning `int` to `short` requires an explicit cast.

---

## 9. Follow-Up Questions

**F1. Is `i++` thread-safe?**

No. `i++` is three steps: read, add, write. Two threads can read the same value and both write the same result, losing an update. Use `AtomicInteger.incrementAndGet()` or synchronization. (See [Part 7](../07-Multithreading-and-Concurrency/04-Synchronization-Race-Conditions-Locks.md).)

**F2. Which is evaluated first in `a + b * c`?**

`b * c`, because `*` has higher precedence than `+`. But **operands** are always evaluated left to right in Java: `a` is evaluated first (its value is fetched), then `b`, then `c`, then the multiplication, then the addition. This matters when operands have side effects, like method calls.

**F3. Does Java support operator overloading?**

No, not for user-defined classes. The only built-in "overloaded" operator is `+`, which works for both numeric addition and `String` concatenation. This keeps code simple and readable.

---

## 10. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Thinking `i = i++` increments `i` | It leaves `i` unchanged. |
| Using `&` instead of `&&` in null checks | `&` evaluates both sides → possible `NullPointerException`. |
| Expecting `%` to always be positive | Sign follows the left operand; use `Math.floorMod`. |
| `7 / 2` expected as `3.5` | Integer division → `3`. Use `7.0 / 2`. |
| Comparing strings with `==` | Compares references. Use `equals()`. |
| `x == NaN` checks | Always false. Use `Double.isNaN(x)`. |
| Relying on precedence | Use brackets for clarity. |

---

## 11. Quick Revision

- Categories: arithmetic, unary, assignment, relational, logical, bitwise, shift, ternary, `instanceof`.
- Integer division truncates; `%` takes sign of the left operand.
- int `/0` → exception; double `/0` → Infinity/NaN.
- `++x` → increment then use; `x++` → use then increment. `i = i++` → unchanged.
- `&&`/`||` short-circuit; `&`/`|` evaluate both sides.
- `<<` × 2ⁿ; `>>` ÷ 2ⁿ keeping sign; `>>>` fills zeros.
- `~x = -(x + 1)`.
- Ternary: `cond ? a : b`; right-associative.
- `instanceof` returns false for null; Java 16 pattern matching `obj instanceof String s`.
- Precedence: unary > `* / %` > `+ -` > shift > relational > equality > `&` > `^` > `|` > `&&` > `||` > `?:` > `=`.

---

[← Previous: Type Casting](05-Type-Casting.md) | [Back to README](../README.md) | [Next: Control Statements and Loops →](07-Control-Statements-and-Loops.md)
