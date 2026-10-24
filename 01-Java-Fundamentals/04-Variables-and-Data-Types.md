# Chapter 1.4 — Variables and Data Types

[← Previous: Program Structure](03-Program-Structure-and-main-Method.md) | [Back to README](../README.md) | [Next: Type Casting →](05-Type-Casting.md)

---

## What You Will Learn

- What a variable is, how to declare and initialise it
- Types of variables: local, instance (non-static) and static (class) variables
- Default values and scope
- All 8 primitive data types with sizes and ranges
- Reference data types and how they differ from primitives
- Literals, `var` (Java 10), and important tricky points such as `char` arithmetic, overflow and floating-point precision

---

## 1. Variables

### 1.1 Interview Definition

> **A variable is a named memory location used to store a value that can be changed during program execution. In Java, every variable has a data type, which decides the kind of value it can hold, the amount of memory it uses and the operations allowed on it. Java is a statically typed language, so a variable must be declared with a type before it is used. Based on where they are declared, Java has three kinds of variables: local variables, instance variables and static (class) variables.**

### 1.2 Understand the Concept

```text
   int     age   =   21 ;
   │        │         │
 data type  name    value
```

- **Declaration:** `int age;` — tells the compiler the name and type.
- **Initialisation:** `age = 21;` — gives the first value.
- **Both together:** `int age = 21;`

**Statically typed** means the type is fixed at compile time. You cannot later store a `String` in `age`. This helps the compiler catch mistakes early.

### 1.3 Three Types of Variables

| Feature | Local variable | Instance variable | Static (class) variable |
|---|---|---|---|
| Declared | Inside a method, constructor or block | Inside a class, outside methods, **without** `static` | Inside a class, outside methods, **with** `static` |
| Belongs to | The method call | Each **object** | The **class** (shared by all objects) |
| Memory | Stack (in the method's frame) | Heap (inside the object) | Heap (with the class's static data, since Java 8) |
| Created when | Method is called | Object is created (`new`) | Class is loaded |
| Destroyed when | Method finishes | Object is garbage collected | Class is unloaded (usually when the program ends) |
| Default value | **None** — must be initialised before use | Yes (0, false, null…) | Yes (0, false, null…) |
| Access | Only inside its block | Through an object: `obj.name` | Through class name: `ClassName.count` |
| Access modifiers allowed | No (only `final`) | Yes | Yes |

### 1.4 Real-World Example

Think of a **college**:
- **Static variable** = the college name — one value shared by every student.
- **Instance variable** = each student's roll number — every student has their own.
- **Local variable** = a rough calculation a student does on scrap paper during an exam — it exists only during that exam and is thrown away afterwards.

### 1.5 Java Example

```java
public class Student {
    static String collegeName = "ABC College";   // static variable - shared
    String name;                                 // instance variable - per object
    int rollNo;                                  // instance variable - default 0

    Student(String name, int rollNo) {
        this.name = name;
        this.rollNo = rollNo;
    }

    void printDetails() {
        String message = "Student details";      // local variable
        System.out.println(message + ": " + name + ", " + rollNo + ", " + collegeName);
    }

    public static void main(String[] args) {
        Student s1 = new Student("Ravi", 101);
        Student s2 = new Student("Priya", 102);

        s1.printDetails();
        s2.printDetails();

        Student.collegeName = "XYZ Institute";   // change the shared value once
        s1.printDetails();
        s2.printDetails();

        Student s3 = new Student(null, 0);
        System.out.println("Passed null and 0: " + s3.name + ", " + s3.rollNo);
    }
}
```

### 1.6 Code Explanation

- `collegeName` is `static`: there is **only one copy**. Changing it through the class name affects every object.
- `name` and `rollNo` are instance variables: `s1` and `s2` each have their own copy.
- `message` is a local variable: it exists only while `printDetails()` is running.
- `this.name = name;` — the parameter `name` hides the field `name`, so `this.name` refers to the field (covered in OOP chapter 3.2).

### 1.7 Expected Output

```text
Student details: Ravi, 101, ABC College
Student details: Priya, 102, ABC College
Student details: Ravi, 101, XYZ Institute
Student details: Priya, 102, XYZ Institute
Passed null and 0: null, 0
```

After `Student.collegeName` changes, **both** students show the new college, proving there is one shared copy.

### 1.8 Default Values and Local Variable Rule

Instance and static variables get default values automatically:

| Type | Default |
|---|---|
| `byte`, `short`, `int`, `long` | `0` (`0L` for long) |
| `float`, `double` | `0.0` |
| `char` | `'\u0000'` (the null character, invisible) |
| `boolean` | `false` |
| Any reference (`String`, arrays, objects) | `null` |

**Local variables never get default values.** Using one before assigning it is a **compile-time error**:

<!-- skip-check -->
```java
public class LocalRule {
    public static void main(String[] args) {
        int x;
        System.out.println(x);   // compile error: variable x might not have been initialized
    }
}
```

Why does Java do this? A forgotten initialisation is a common bug. For fields, a default is safe because the object is created in a known way. For locals, Java forces you to be explicit.

---

## 2. Data Types

### 2.1 Interview Definition

> **A data type specifies the type of value a variable can store, the size of memory it occupies, the range of values and the operations that can be performed on it. Java has two categories of data types: primitive data types, which are the 8 built-in types (`byte`, `short`, `int`, `long`, `float`, `double`, `char` and `boolean`) that store actual values directly, and reference (non-primitive) data types, such as classes, interfaces, arrays, enums and strings, which store a reference (address) to an object in heap memory.**

### 2.2 The 8 Primitive Data Types

| Type | Size | Range | Default | Example |
|---|---|---|---|---|
| `byte` | 1 byte (8 bits) | −128 to 127 | 0 | `byte b = 100;` |
| `short` | 2 bytes (16 bits) | −32,768 to 32,767 | 0 | `short s = 30000;` |
| `int` | 4 bytes (32 bits) | −2,147,483,648 to 2,147,483,647 (≈ ±2.1 billion) | 0 | `int i = 50000;` |
| `long` | 8 bytes (64 bits) | −2⁶³ to 2⁶³−1 (≈ ±9.2 × 10¹⁸) | 0L | `long l = 9000000000L;` |
| `float` | 4 bytes | ≈ ±3.4 × 10³⁸, about **6–7** decimal digits precision | 0.0f | `float f = 3.14f;` |
| `double` | 8 bytes | ≈ ±1.7 × 10³⁰⁸, about **15–16** decimal digits precision | 0.0d | `double d = 3.14159;` |
| `char` | 2 bytes (16 bits) | 0 to 65,535 (`'\u0000'` to `'\uffff'`), **unsigned** | `'\u0000'` | `char c = 'A';` |
| `boolean` | JVM dependent (logically 1 bit) | `true` or `false` | false | `boolean ok = true;` |

**Groups:**
- **Integer types:** `byte`, `short`, `int`, `long` (whole numbers, signed)
- **Floating-point types:** `float`, `double` (decimal numbers, IEEE 754 standard)
- **Character type:** `char` (single **Unicode** character)
- **Boolean type:** `boolean`

**How to remember the ranges:** an n-bit signed integer goes from **−2ⁿ⁻¹ to 2ⁿ⁻¹ − 1**. For `byte` (8 bits): −2⁷ = −128 to 2⁷−1 = 127. One extra value on the negative side because zero takes one of the positive slots.

### 2.3 Understand Important Points

**1. Why is `char` 2 bytes in Java but 1 byte in C?**
C uses ASCII (0–127, fits in 1 byte). Java uses **Unicode** (UTF-16) so it can represent characters from all world languages — Hindi, Telugu, Chinese, etc. Unicode needs 16 bits for the basic characters.

**2. Default type of literals**
- A whole-number literal like `100` is an **`int`** by default. For a `long`, add `L`: `9000000000L`. (Without `L`, `9000000000` is a compile error — "integer number too large".)
- A decimal literal like `3.14` is a **`double`** by default. For a `float`, add `f`: `3.14f`. (`float f = 3.14;` is a compile error — "possible lossy conversion from double to float".)

**3. Integer overflow**
If a value goes beyond the range, it **wraps around** silently. No exception is thrown.

**4. Floating-point precision**
`float` and `double` store values in binary, so many decimal fractions (like 0.1) cannot be stored exactly. `0.1 + 0.2` is not exactly `0.3`. For money, use **`BigDecimal`**.

**5. `boolean` cannot be converted to or from numbers.** `if (1)` is a compile error in Java (unlike C).

### 2.4 Real-World Example

Data types are like **containers in a kitchen**:
- `byte` = a small spice box, `int` = a regular jar, `long` = a big storage drum.
- If you pour too much into a small box, it spills over (overflow).
- `boolean` is a light switch — only ON or OFF.
- A **reference type** is like a **locker number** written on a card. The card (variable) does not hold your bag; it tells you where the locker (object in the heap) is.

### 2.5 Java Example — Primitives, Ranges, Overflow and Precision

```java
public class DataTypesDemo {
    public static void main(String[] args) {
        byte b = 127;
        short s = 32000;
        int i = 2_000_000_000;          // underscores improve readability (Java 7+)
        long l = 9_000_000_000L;        // L is required: value is beyond int range
        float f = 3.14f;                // f is required: 3.14 is a double by default
        double d = 3.141592653589793;
        char c = 'A';
        boolean isJavaFun = true;

        System.out.println("byte: " + b + ", short: " + s + ", int: " + i + ", long: " + l);
        System.out.println("float: " + f + ", double: " + d);
        System.out.println("char: " + c + ", boolean: " + isJavaFun);

        // Ranges using wrapper class constants
        System.out.println("int range: " + Integer.MIN_VALUE + " to " + Integer.MAX_VALUE);
        System.out.println("byte range: " + Byte.MIN_VALUE + " to " + Byte.MAX_VALUE);

        // Overflow wraps around silently
        int max = Integer.MAX_VALUE;
        System.out.println("MAX_VALUE + 1 = " + (max + 1));
        b++;                             // 127 + 1 wraps to -128
        System.out.println("byte 127 after ++ = " + b);

        // char is a number internally (Unicode value)
        char next = (char) (c + 1);
        System.out.println("Unicode of 'A' = " + (int) c + ", next char = " + next);
        System.out.println("'A' + 1 = " + ('A' + 1));

        // Floating-point precision
        System.out.println("0.1 + 0.2 = " + (0.1 + 0.2));
        System.out.println("float precision: " + 1.123456789f);
        System.out.println("10 / 3 = " + (10 / 3) + ", 10.0 / 3 = " + (10.0 / 3));
    }
}
```

### 2.6 Code Explanation

- `2_000_000_000` — underscores in numeric literals are allowed since **Java 7** and are ignored by the compiler.
- `9_000_000_000L` — without `L`, the compiler treats the literal as an `int` and reports "integer number too large".
- `Integer.MIN_VALUE` / `MAX_VALUE` — constants in the wrapper class give exact ranges.
- `max + 1` — `int` overflow: the bits wrap from the largest positive to the smallest negative number.
- `b++` — the `++` operator on a `byte` does an internal cast back to `byte`, so 127 becomes −128.
- `(char)(c + 1)` — `c + 1` is an `int` (66). Casting back to `char` gives `'B'`.
- `'A' + 1` — inside `println`, the result is an `int`, so it prints `66`, not `B`.
- `0.1 + 0.2` — binary floating point cannot represent 0.1 exactly, so the sum is slightly more than 0.3.
- `1.123456789f` — `float` keeps only about 7 significant digits.
- `10 / 3` — both operands are `int`, so the result is integer division (`3`). With `10.0`, it becomes `double` division.

### 2.7 Expected Output

```text
byte: 127, short: 32000, int: 2000000000, long: 9000000000
float: 3.14, double: 3.141592653589793
char: A, boolean: true
int range: -2147483648 to 2147483647
byte range: -128 to 127
MAX_VALUE + 1 = -2147483648
byte 127 after ++ = -128
Unicode of 'A' = 65, next char = B
'A' + 1 = 66
0.1 + 0.2 = 0.30000000000000004
float precision: 1.1234568
10 / 3 = 3, 10.0 / 3 = 3.3333333333333335
```

Notice: underscores are not printed; overflow wraps silently; `0.1 + 0.2` shows the precision issue; the float is rounded to about 7 digits.

### 2.8 Interview Answer — "What are the data types in Java?"

> "Java has two categories of data types: primitive and reference. There are **eight primitive types**: `byte` (1 byte), `short` (2 bytes), `int` (4 bytes) and `long` (8 bytes) for whole numbers; `float` (4 bytes) and `double` (8 bytes) for decimal numbers; `char`, which is 2 bytes because Java uses Unicode; and `boolean`, which stores true or false. Primitive variables store the actual value directly and have fixed sizes on every platform, which makes Java portable. **Reference types** include classes, interfaces, arrays, enums and `String`. A reference variable does not store the object itself — it stores a reference to an object that lives in heap memory, and its default value is `null`. By default, a whole-number literal is `int` and a decimal literal is `double`, which is why we add `L` for long and `f` for float."

---

## 3. Reference Data Types

### 3.1 Interview Definition

> **Reference (non-primitive) data types are types created from classes, interfaces, arrays, enums and records. A reference variable stores the memory reference of an object rather than the object's data itself. The actual object is created in heap memory using `new` (or literals for strings and arrays). The default value of a reference variable is `null`, which means it does not refer to any object. Reference types can have methods, can be `null`, and two references can point to the same object.**

### 3.2 Primitive vs Reference

| Point | Primitive | Reference |
|---|---|---|
| What it stores | The actual value | A reference (address) to an object |
| Examples | `int`, `double`, `char`, `boolean` | `String`, `int[]`, `Student`, `List<Integer>` |
| Memory of value | In the variable itself (stack for locals, inside the object for fields) | Object always in the **heap** |
| Default value (as field) | 0 / 0.0 / false / `'\u0000'` | `null` |
| Can be `null`? | No | Yes |
| Has methods? | No | Yes (`s.length()`) |
| `==` compares | Values | References (whether both point to the same object) |
| Size | Fixed (by type) | Reference size fixed by JVM; object size varies |
| Defined by | The language | Classes (Java library or programmer) |

### 3.3 Java Example — Copying Primitives vs References

```java
public class ReferenceDemo {
    static class Box {
        int value;
    }

    public static void main(String[] args) {
        // Primitive copy: independent values
        int a = 10;
        int b = a;
        b = 20;
        System.out.println("a = " + a + ", b = " + b);

        // Reference copy: both variables point to the same object
        Box box1 = new Box();
        box1.value = 10;
        Box box2 = box1;
        box2.value = 20;
        System.out.println("box1.value = " + box1.value + ", box2.value = " + box2.value);
        System.out.println("box1 == box2 ? " + (box1 == box2));

        Box box3 = new Box();
        box3.value = 20;
        System.out.println("box1 == box3 ? " + (box1 == box3));

        Box nothing = null;
        System.out.println("nothing is null? " + (nothing == null));
    }
}
```

### 3.4 Code Explanation and Memory Picture

```text
STACK                         HEAP
a    [10]
b    [20]
box1 [ref ●]──────┐
box2 [ref ●]──────┴──►  Box object { value = 20 }
box3 [ref ●]─────────►  Box object { value = 20 }
nothing [null]
```

- `int b = a;` copies the **value** 10. Changing `b` does not affect `a`.
- `Box box2 = box1;` copies the **reference**. Both variables point to the **same** object, so changing `box2.value` also changes what `box1` sees.
- `box1 == box3` is `false` because they are **different objects**, even though their values are equal. `==` on references compares identity, not content.

### 3.5 Expected Output

```text
a = 10, b = 20
box1.value = 20, box2.value = 20
box1 == box2 ? true
box1 == box3 ? false
nothing is null? true
```

---

## 4. `var` — Local Variable Type Inference (Java 10)

### 4.1 Interview Definition

> **`var`, introduced in Java 10, lets the compiler infer the type of a local variable from the value used to initialise it. The variable is still statically typed — the type is decided at compile time and cannot change later. `var` can be used only for local variables with an initialiser, enhanced for-loop variables, and lambda parameters (Java 11); it cannot be used for fields, method parameters or return types.**

```java
import java.util.ArrayList;

public class VarDemo {
    public static void main(String[] args) {
        var count = 10;                        // inferred as int
        var name = "Java";                     // inferred as String
        var list = new ArrayList<String>();    // inferred as ArrayList<String>
        list.add("Spring");
        list.add("Hibernate");

        for (var item : list) {
            System.out.println(name + " -> " + item);
        }
        System.out.println("count + 5 = " + (count + 5));
        // count = "ten";   // compile error: incompatible types, count is an int
    }
}
```

**Expected Output:**

```text
Java -> Spring
Java -> Hibernate
count + 5 = 15
```

**Rules:**
- `var x;` — invalid (no initialiser).
- `var x = null;` — invalid (type cannot be inferred).
- `var` is **not** a keyword; it is a *reserved type name*, so you can still have a variable named `var` (but do not).
- `var` is **not** like JavaScript `var` — the type is fixed.

---

## 5. Interview Questions with Complete Answers

### Basic Level

**Q1. What is the difference between local, instance and static variables?**

A **local variable** is declared inside a method or block, lives on the stack only while that method runs, and must be initialised before use. An **instance variable** is declared in a class without `static`; each object gets its own copy, stored in the heap, with default values. A **static variable** is declared with `static`; only one copy exists for the whole class, shared by all objects, and it is created when the class is loaded. Example: in a `Student` class, `rollNo` is an instance variable, `collegeName` is a static variable, and a temporary `total` inside a method is a local variable.

**Q2. What is the size of `char` in Java and why?**

`char` is **2 bytes (16 bits)** because Java uses **Unicode (UTF-16)** to support characters from all languages, not only English. Its range is 0 to 65,535, and it is unsigned. Characters outside this basic range (like many emojis) are stored as two `char` values called a *surrogate pair*.

**Q3. What is the default value of local variables?**

Local variables have **no default value**. The compiler gives an error ("variable might not have been initialized") if you try to read one before assigning a value. Only instance and static variables (and array elements) receive default values.

**Q4. Why do we write `L` after long values and `f` after float values?**

Because by default, integer literals are `int` and decimal literals are `double`. Without `L`, a number larger than the `int` range cannot even be written as a literal. Without `f`, assigning a `double` literal to a `float` is a narrowing conversion that could lose precision, so the compiler refuses it.

### Intermediate Level

**Q5. What happens when an `int` overflows?**

It **wraps around** without any error. `Integer.MAX_VALUE + 1` becomes `Integer.MIN_VALUE` (−2147483648). This happens because Java uses **two's complement** binary representation; adding one to the largest positive value flips the sign bit. To detect overflow, use `Math.addExact(a, b)`, which throws `ArithmeticException`, or use `long` / `BigInteger` for larger values.

```java
public class OverflowCheck {
    public static void main(String[] args) {
        try {
            int result = Math.addExact(Integer.MAX_VALUE, 1);
            System.out.println(result);
        } catch (ArithmeticException e) {
            System.out.println("Caught: " + e.getMessage());
        }
    }
}
```

**Expected Output:**

```text
Caught: integer overflow
```

**Q6. Why is `0.1 + 0.2` not equal to `0.3`?**

`float` and `double` use **binary (base-2) floating-point** representation (IEEE 754). Just as 1/3 cannot be written exactly in decimal (0.333…), 0.1 cannot be written exactly in binary. So small rounding errors appear. `0.1 + 0.2` gives `0.30000000000000004`. That is why we never compare doubles with `==` directly — we compare with a small tolerance (`Math.abs(a - b) < 1e-9`) — and why we use **`BigDecimal`** for financial calculations.

**Q7. What is the size of `boolean`?**

The Java specification does not define an exact size. Logically it needs one bit. In practice, the HotSpot JVM uses **1 byte** for a `boolean` in arrays and typically 4 bytes (an `int` slot) for local `boolean` variables during execution. The correct interview answer: *"The size is JVM-dependent; it represents one bit of information."*

**Q8. Is `String` a primitive data type?**

No. `String` is a **class** in `java.lang`, so it is a reference type. It feels like a primitive because of special language support: string literals (`"abc"`), the `+` concatenation operator and the string pool. But `String` objects live in the heap, can be `null`, and have methods.

### Advanced Level

**Q9. Why does Java use fixed sizes for primitive types?**

For **portability**. In C, the size of `int` can be 2 or 4 bytes depending on the compiler and machine, so the same program may give different results on different systems. Java fixes sizes in the language specification, so an `int` is always 32 bits everywhere and arithmetic behaves identically on every JVM.

**Q10. What is the difference between `char c = 65;` and `char c = 'A';`?**

Both store the same value — the Unicode code 65, which is `'A'`. Java allows assigning an `int` **constant** within the `char` range (0–65535) to a `char` without a cast. Printing `c` shows `A` in both cases. However, assigning an int **variable** needs a cast: `int x = 65; char c = (char) x;`.

**Q11. Where are local, instance and static variables stored in memory?**

- **Local variables** (both primitive values and reference variables) are in the method's **stack frame**.
- **Instance variables** are stored inside the object, in the **heap**.
- **Static variables** are stored with the class's data. Since Java 8, static variables are kept in the **heap** (as part of the `Class` object), while class metadata is in **Metaspace**. (Before Java 8, they were in PermGen.)

---

## 6. Follow-Up Questions

**F1. How would you store a value larger than `long`?**

Use `java.math.BigInteger`, which can store integers of any size (limited only by memory). For precise decimals, use `BigDecimal`. These are classes, so you use methods instead of operators: `a.add(b)`, `a.multiply(b)`.

**F2. Can a `char` hold a negative value?**

No. `char` is the only **unsigned** primitive type in Java. Its range is 0 to 65,535. Casting −1 to `char` gives 65,535 (`'\uffff'`).

**F3. What is `'\u0000'`?**

It is the Unicode escape for the null character (code 0), and it is the default value of a `char` field. It is not the same as `null`, which is only for references.

**F4. Should I use `float` or `double`?**

Use **`double`** by default — it has much better precision (15–16 digits vs 6–7), and it is the default for decimal literals and most `Math` methods. Use `float` only when memory is very limited (for example, very large arrays in graphics).

---

## 7. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| `float f = 3.14;` | Compile error; write `3.14f`. |
| `long big = 9000000000;` | Compile error; write `9000000000L`. |
| Expecting `int` overflow to throw an exception | It wraps silently; use `Math.addExact` to detect it. |
| Comparing doubles with `==` | Use a tolerance or `BigDecimal`. |
| Thinking `char` is 1 byte | It is 2 bytes (Unicode). |
| Thinking `String` is primitive | It is a class (reference type). |
| Using a local variable without initialising | Compile error. |
| `if (1)` like in C | Compile error; Java needs a `boolean`. |
| Using `float` for money | Use `BigDecimal`. |

---

## 8. Quick Revision

- Variable = named memory location; Java is **statically typed**.
- **Local** (stack, no default) · **Instance** (heap, per object, default) · **Static** (one per class, shared, default).
- 8 primitives: `byte(1) short(2) int(4) long(8) float(4) double(8) char(2) boolean(JVM-dependent)`.
- Integer literal default = `int`; decimal literal default = `double`.
- Signed range: −2ⁿ⁻¹ to 2ⁿ⁻¹−1. `char` is unsigned 0–65535.
- Overflow wraps silently. `0.1 + 0.2 != 0.3`. Money → `BigDecimal`.
- Reference types store references; default `null`; `==` compares references.
- `var` (Java 10): local inference only, still statically typed.

**Memory trick for sizes — "1-2-4-8, 4-8, 2, ?":** byte-short-int-long, float-double, char, boolean.

---

[← Previous: Program Structure](03-Program-Structure-and-main-Method.md) | [Back to README](../README.md) | [Next: Type Casting →](05-Type-Casting.md)
