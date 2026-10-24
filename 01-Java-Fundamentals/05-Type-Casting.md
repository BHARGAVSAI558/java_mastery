# Chapter 1.5 — Type Casting and Type Promotion

[← Previous: Variables and Data Types](04-Variables-and-Data-Types.md) | [Back to README](../README.md) | [Next: Operators →](06-Operators.md)

---

## What You Will Learn

- What type casting is and why it is needed
- Widening (implicit) casting and narrowing (explicit) casting
- What happens to data during narrowing (truncation and wrap-around)
- Type promotion in expressions (why `byte + byte` is an `int`)
- Compound assignment operators and hidden casts
- A short introduction to reference casting (upcasting and downcasting)

---

## 1. Type Casting

### 1.1 Interview Definition

> **Type casting is the process of converting a value from one data type to another. Java supports two kinds of primitive type casting: widening casting (implicit), where a smaller type is automatically converted to a larger compatible type without any data loss, such as `int` to `long` or `int` to `double`; and narrowing casting (explicit), where a larger type is converted to a smaller type using a cast operator like `(int)`, which may cause loss of data or precision. Java also supports casting between reference types in an inheritance hierarchy, called upcasting and downcasting.**

### 1.2 Understand the Concept

The widening order of primitive types is:

```text
byte → short → int → long → float → double
              ↑
            char
```

- Moving **right** (small → large) is **widening** — automatic and safe.
- Moving **left** (large → small) is **narrowing** — you must write a cast, and data may be lost.
- `boolean` **cannot** be cast to or from any other primitive type.
- `char` widens to `int` (and beyond). `byte`/`short` do **not** widen to `char` automatically, because `char` is unsigned and they can be negative.

**Why does Java force an explicit cast for narrowing?**
Because data could be lost. The cast is your way of telling the compiler, *"I know the risk, do it anyway."*

**Interesting point:** `long` (8 bytes) → `float` (4 bytes) is still *widening*, because `float` has a much larger **range** (it can store up to ~3.4 × 10³⁸). However, it can lose **precision** for very large `long` values.

### 1.3 Real-World Example

- **Widening** = pouring water from a small glass into a big bucket. Everything fits; nothing spills.
- **Narrowing** = pouring water from a big bucket into a small glass. You must do it deliberately, and some water may spill (data loss).

---

## 2. Widening (Implicit) Casting

### 2.1 Java Example

```java
public class WideningDemo {
    public static void main(String[] args) {
        int marks = 95;
        long bigMarks = marks;        // int → long
        float fMarks = marks;         // int → float
        double dMarks = marks;        // int → double

        char grade = 'A';
        int gradeCode = grade;        // char → int (Unicode value)

        long population = 1_400_000_000L;
        float popFloat = population;  // long → float (allowed, range is bigger)

        System.out.println("int to long: " + bigMarks);
        System.out.println("int to float: " + fMarks);
        System.out.println("int to double: " + dMarks);
        System.out.println("char to int: " + gradeCode);
        System.out.println("long to float: " + popFloat);

        long precise = 123_456_789_123L;
        float lossy = precise;        // widening, but precision is lost
        System.out.println("long 123456789123 as float: " + lossy);
    }
}
```

### 2.2 Code Explanation

- No cast operator is needed in any line, because each conversion is widening.
- `int → float/double` prints with `.0` because the value is now a floating-point number.
- `char → int` gives the Unicode code of `'A'` (65).
- `long → float` is allowed but the last line shows precision loss: `float` keeps only about 7 significant digits.

### 2.3 Expected Output

```text
int to long: 95
int to float: 95.0
int to double: 95.0
char to int: 65
long to float: 1.4E9
long 123456789123 as float: 1.2345679E11
```

`1.4E9` is scientific notation for 1.4 × 10⁹. The last value is printed as `1.2345679E11`. The float actually stores 123,456,790,528 — not the original 123,456,789,123 — which shows the precision loss.

---

## 3. Narrowing (Explicit) Casting

### 3.1 What happens to the data?

| Conversion | What happens |
|---|---|
| `double`/`float` → `int`/`long` | The decimal part is **truncated** (cut off, not rounded). `9.99` → `9`, `-9.99` → `-9`. |
| Larger integer → smaller integer (`int` → `byte`) | Only the **lowest bits** are kept. The value **wraps around** if out of range. |
| `double` → `float` | Precision reduced; very large values become `Infinity`. |
| `double` too large for `int` | Becomes `Integer.MAX_VALUE` (or `MIN_VALUE`); `NaN` becomes `0`. |

**How `int` → `byte` wrap-around works:** a `byte` stores 8 bits. Java keeps the lowest 8 bits of the `int`.
- `130` in binary is `1000 0010`. As a signed byte, the leftmost 1 means negative: value = 130 − 256 = **−126**.
- Quick formula for `byte`: if the value is outside −128..127, result = value **mod 256**, adjusted into the range.

### 3.2 Java Example

```java
public class NarrowingDemo {
    public static void main(String[] args) {
        double price = 99.99;
        int rupees = (int) price;             // truncation, not rounding
        System.out.println("(int) 99.99 = " + rupees);
        System.out.println("(int) -99.99 = " + (int) -99.99);

        System.out.println("Math.round(99.99) = " + Math.round(price));   // proper rounding

        int big = 130;
        byte small = (byte) big;              // wraps around
        System.out.println("(byte) 130 = " + small);

        int value = 300;
        byte b = (byte) value;                // 300 - 256 = 44
        System.out.println("(byte) 300 = " + b);

        int code = 66;
        char letter = (char) code;            // int → char
        System.out.println("(char) 66 = " + letter);

        long hugeLong = 5_000_000_000L;
        int cut = (int) hugeLong;             // keeps lowest 32 bits
        System.out.println("(int) 5000000000L = " + cut);

        double tooBig = 1e20;
        System.out.println("(int) 1e20 = " + (int) tooBig);   // saturates to MAX_VALUE

        double average = 7 / 2;               // int division happens first
        double correct = (double) 7 / 2;      // cast first, then divide
        System.out.println("7 / 2 stored in double = " + average);
        System.out.println("(double) 7 / 2 = " + correct);
    }
}
```

### 3.3 Code Explanation

- `(int) price` cuts off `.99`. It does **not** round. For rounding, use `Math.round()` (returns `long` for `double`).
- `(byte) 130` keeps the low 8 bits (`1000 0010`) → −126.
- `(byte) 300` → 300 in binary is `1 0010 1100`; the low 8 bits are `0010 1100` = 44.
- `(int) 5_000_000_000L` keeps the low 32 bits → 705,032,704.
- `(int) 1e20` — when a floating-point value is too large for `int`, Java **saturates** it to `Integer.MAX_VALUE` (this is different from integer-to-integer narrowing, which wraps).
- `7 / 2` is evaluated as `int` division (= 3) **before** being stored into a `double`, so you get `3.0`. Casting one operand first forces `double` division.

### 3.4 Expected Output

```text
(int) 99.99 = 99
(int) -99.99 = -99
Math.round(99.99) = 100
(byte) 130 = -126
(byte) 300 = 44
(char) 66 = B
(int) 5000000000L = 705032704
(int) 1e20 = 2147483647
7 / 2 stored in double = 3.0
(double) 7 / 2 = 3.5
```

### 3.5 Interview Answer — "What is type casting? Explain widening and narrowing."

> "Type casting means converting a value from one data type to another. In Java there are two kinds for primitives. **Widening casting** is when we convert a smaller type to a larger type — like `int` to `long` or `int` to `double`. It is done automatically by the compiler because there is no risk of losing data. The order is byte, short, int, long, float, double, and char can widen to int. **Narrowing casting** is the opposite — converting a larger type to a smaller one, like `double` to `int`. We must write it explicitly using the cast operator, for example `(int) 9.99`, because data can be lost. When converting decimal to integer, the decimal part is truncated, and when converting a large integer to a smaller type like byte, the value can wrap around. Boolean cannot be cast to any other type. For objects, we also have upcasting and downcasting in inheritance."

---

## 4. Type Promotion in Expressions

### 4.1 Interview Definition

> **Type promotion is the automatic conversion that Java performs when evaluating an arithmetic expression. Before a binary operation, `byte`, `short` and `char` operands are always promoted to `int`. Then, if either operand is `long`, `float` or `double`, the other operand is promoted to the larger of these types. The result of the expression has that promoted type, which is why `byte + byte` produces an `int`.**

### 4.2 The Rules

1. `byte`, `short`, `char` → promoted to **`int`** in any arithmetic expression.
2. If one operand is `long` → the result is `long`.
3. If one operand is `float` → the result is `float`.
4. If one operand is `double` → the result is `double`.

(Highest type wins: `double > float > long > int`.)

### 4.3 Java Example

<!-- skip-check -->
```java
public class PromotionError {
    public static void main(String[] args) {
        byte a = 10;
        byte b = 20;
        byte c = a + b;      // COMPILE ERROR: incompatible types: possible lossy conversion from int to byte
    }
}
```

Fix it with a cast, or use an `int`:

```java
public class PromotionDemo {
    public static void main(String[] args) {
        byte a = 10;
        byte b = 20;
        byte c = (byte) (a + b);     // explicit cast needed
        int sum = a + b;             // or store in an int
        System.out.println("c = " + c + ", sum = " + sum);

        char ch = 'a';
        System.out.println("ch + 1 = " + (ch + 1));          // int result
        System.out.println("(char)(ch + 1) = " + (char) (ch + 1));

        int i = 5;
        long l = 10L;
        float f = 2.5f;
        double d = 1.5;
        var result1 = i + l;        // long
        var result2 = l + f;        // float
        var result3 = f + d;        // double
        System.out.println(result1 + " " + result2 + " " + result3);

        final byte x = 10, y = 20;
        byte z = x + y;             // OK! compile-time constants that fit in byte
        System.out.println("z = " + z);
    }
}
```

**Expected Output:**

```text
c = 30, sum = 30
ch + 1 = 98
(char)(ch + 1) = b
15 12.5 4.0
z = 30
```

### 4.4 Code Explanation

- `a + b` — both `byte`s become `int` before adding, so the result is an `int`. Assigning it to `byte` needs a cast.
- `ch + 1` — `char` promoted to `int` (97 + 1 = 98).
- `i + l` → `long` (15), `l + f` → `float` (12.5), `f + d` → `double` (4.0).
- **Special case:** `final byte x = 10, y = 20; byte z = x + y;` compiles, because `x` and `y` are **compile-time constants**. The compiler computes `30` at compile time and knows it fits in a `byte`. Without `final`, it is an error.

---

## 5. Compound Assignment Operators Have a Hidden Cast

```java
public class CompoundCast {
    public static void main(String[] args) {
        byte b = 10;
        // b = b + 5;       // compile error: int cannot be stored in byte
        b += 5;             // OK: same as b = (byte)(b + 5)
        System.out.println("b = " + b);

        byte max = 127;
        max += 1;           // hidden cast causes wrap-around
        System.out.println("max = " + max);

        int i = 10;
        i *= 2.5;           // same as i = (int)(i * 2.5)
        System.out.println("i = " + i);

        short s = 1;
        s++;                // ++ also has a hidden cast
        System.out.println("s = " + s);
    }
}
```

**Expected Output:**

```text
b = 15
max = -128
i = 25
s = 2
```

`+=`, `-=`, `*=`, `/=`, `%=`, `++` and `--` automatically cast the result back to the type of the left-hand variable. This is convenient, but it can **hide overflow and truncation**. `i *= 2.5` gives `25` (not `25.0`) because the result is cast back to `int`.

---

## 6. Reference Type Casting (Introduction)

For objects, casting works inside an inheritance hierarchy:

- **Upcasting:** child → parent reference. Automatic and always safe. `Animal a = new Dog();`
- **Downcasting:** parent → child reference. Needs an explicit cast and can fail at runtime with `ClassCastException`. `Dog d = (Dog) a;`
- Use `instanceof` to check before downcasting (Java 16+ pattern matching: `if (a instanceof Dog d) { ... }`).
- Casting between unrelated classes (`String` → `Integer`) is a **compile-time error**.

This is taught in detail in [Part 3, Chapter 12](../03-OOP/12-Upcasting-Downcasting-Dynamic-Dispatch.md).

**Converting strings to numbers is NOT casting.** `(int) "123"` is a compile error. Use parsing methods:

```java
public class ParseDemo {
    public static void main(String[] args) {
        String text = "123";
        int number = Integer.parseInt(text);
        double decimal = Double.parseDouble("45.67");
        String back = String.valueOf(number + 1);
        System.out.println(number + 1);
        System.out.println(decimal * 2);
        System.out.println(back + "!");
    }
}
```

**Expected Output:**

```text
124
91.34
124!
```

`Integer.parseInt("12a")` would throw `NumberFormatException`.

---

## 7. Interview Questions with Complete Answers

### Basic Level

**Q1. What is the difference between implicit and explicit casting?**

| Implicit (widening) | Explicit (narrowing) |
|---|---|
| Smaller → larger type | Larger → smaller type |
| Done automatically by compiler | Programmer writes `(type)` |
| No data loss (except precision in `long`→`float`/`double`) | Possible data loss |
| `int x = 10; double d = x;` | `double d = 9.7; int x = (int) d;` |

**Q2. Does `(int) 9.99` give 9 or 10?**

It gives **9**. Casting from floating-point to integer **truncates** toward zero; it does not round. `(int) -9.99` gives `-9`. To round, use `Math.round(9.99)`, which returns `10`.

**Q3. Can we cast `boolean` to `int`?**

No. `boolean` is not compatible with any numeric type in Java. `(int) true` is a compile error. If you need it, write `int x = flag ? 1 : 0;`.

### Intermediate Level

**Q4. Why does `byte c = a + b;` give a compile error when `a` and `b` are bytes?**

Because of **type promotion**: in arithmetic, `byte`, `short` and `char` are automatically promoted to `int`. So `a + b` is an `int`, and storing an `int` into a `byte` is narrowing, which needs an explicit cast. The fix is `byte c = (byte)(a + b);` or `int c = a + b;`. The exception is when `a` and `b` are `final` compile-time constants whose sum fits in a byte.

**Q5. What is the output of `(byte) 200`?**

`-56`. A byte holds −128 to 127. 200 in binary is `1100 1000`. As a signed 8-bit value this is 200 − 256 = **−56**.

**Q6. Why does `b += 5` compile but `b = b + 5` does not (for a `byte b`)?**

Compound assignment operators include an **implicit cast** to the left-hand type. The Java Language Specification defines `b += 5` as `b = (byte)(b + 5)`. So it compiles, but be aware it can silently overflow.

**Q7. Is `long` to `float` widening or narrowing? It is 8 bytes to 4 bytes.**

It is **widening**. Widening is decided by **range**, not byte size. A `float` can represent values up to about 3.4 × 10³⁸, much larger than `long`'s maximum (~9.2 × 10¹⁸). However, `float` has only ~7 digits of precision, so large `long` values lose precision.

### Advanced Level

**Q8. What does `(int) Double.NaN` and `(int) 1e20` give?**

`(int) Double.NaN` gives **0**. `(int) 1e20` gives **`Integer.MAX_VALUE`** (2147483647). When a floating-point value is converted to an integer type and is out of range, Java **saturates** it to the minimum or maximum value. This is different from integer-to-integer narrowing, which wraps around.

**Q9. What is the output?**

```java
public class TrickyCast {
    public static void main(String[] args) {
        int x = 'A' + 'B';
        char c = 'A' + 1;
        System.out.println(x);
        System.out.println(c);
        System.out.println('A' + 'B' + "C");
        System.out.println("C" + 'A' + 'B');
    }
}
```

**Expected Output:**

```text
131
B
131C
CAB
```

- `'A' + 'B'` = 65 + 66 = 131 (promoted to `int`).
- `char c = 'A' + 1;` compiles because `'A' + 1` is a **constant expression** (66) that fits in `char`.
- `'A' + 'B' + "C"` → evaluated left to right: 131 first, then `"131" + "C"`.
- `"C" + 'A' + 'B'` → once a `String` appears on the left, `+` means concatenation: `"CA"` then `"CAB"`.

---

## 8. Follow-Up Questions

**F1. How do you safely convert a `long` to `int` and detect overflow?**

Use `Math.toIntExact(longValue)`. It throws `ArithmeticException` if the value does not fit in an `int`, instead of silently wrapping.

**F2. What is the difference between casting and parsing?**

Casting converts between compatible types (numbers to numbers, or related classes). **Parsing** converts text to a number: `Integer.parseInt("42")`. You cannot cast a `String` to an `int`.

**F3. Which method gives rounding, and what type does it return?**

`Math.round(double)` returns `long`; `Math.round(float)` returns `int`. `Math.floor` and `Math.ceil` return `double`. For decimal places in money, use `BigDecimal.setScale(2, RoundingMode.HALF_UP)`.

---

## 9. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Thinking `(int) 9.9` rounds to 10 | It truncates to 9. |
| `double avg = sum / count;` with int variables | Integer division happens first. Write `(double) sum / count`. |
| Forgetting `byte + byte = int` | All arithmetic on byte/short/char produces at least `int`. |
| Writing `(int) "123"` | Use `Integer.parseInt("123")`. |
| Ignoring overflow in `+=` | Compound assignment casts silently. |
| Assuming narrowing of floating values wraps like integers | Floating → integer **saturates**; integer → smaller integer **wraps**. |

---

## 10. Quick Revision

- **Widening (implicit):** `byte → short → int → long → float → double`; `char → int`. Safe.
- **Narrowing (explicit):** needs `(type)`; may truncate or wrap.
- `boolean` cannot be cast.
- Floating → integer: **truncates** decimals; out-of-range **saturates**; NaN → 0.
- Integer → smaller integer: keeps low bits → **wraps**. `(byte)130 = -126`, `(byte)300 = 44`.
- **Promotion:** byte/short/char → int in expressions; highest type wins.
- Compound operators (`+=`, `++`) contain a hidden cast.
- Strings → numbers = **parsing**, not casting.
- Reference casting: upcasting automatic, downcasting explicit + `instanceof`.

---

[← Previous: Variables and Data Types](04-Variables-and-Data-Types.md) | [Back to README](../README.md) | [Next: Operators →](06-Operators.md)
