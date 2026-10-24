# Chapter 2.4 — Arrays and Multidimensional Arrays

[← Previous: StringBuilder and StringBuffer](03-StringBuilder-and-StringBuffer.md) | [Back to README](../README.md) | [Next: Wrapper Classes →](05-Wrapper-Classes-Autoboxing-Unboxing.md)

---

## What You Will Learn

- What an array is, how it is stored, and why arrays are objects in Java
- Declaring, creating and initialising arrays; default values
- Traversing arrays; `ArrayIndexOutOfBoundsException`
- The `java.util.Arrays` utility class (`toString`, `sort`, `binarySearch`, `fill`, `copyOf`, `equals`, `asList`)
- Copying arrays: shallow vs deep, `clone()`, `System.arraycopy`
- 2D arrays, matrix operations and **jagged arrays**
- Array vs `ArrayList`

---

## 1. Arrays

### 1.1 Interview Definition

> **An array is a container object that stores a fixed number of elements of the same data type in contiguous memory locations. Each element is accessed using a zero-based index. In Java, arrays are objects created in heap memory; they have a `length` field that gives the number of elements, and the size cannot be changed after creation. Arrays can store primitive values or object references, provide fast O(1) access by index, and the JVM checks every index at runtime, throwing `ArrayIndexOutOfBoundsException` for an invalid index.**

### 1.2 Understand the Concept

**Why do we need arrays?** Without arrays, storing marks of 60 students needs 60 variables (`m1, m2, … m60`). An array stores them under one name: `marks[0] … marks[59]`, and a loop can process all of them.

**Key properties:**
1. **Fixed size** — decided at creation, cannot grow or shrink.
2. **Same type** — all elements have the same declared type (homogeneous).
3. **Indexed** — first index is 0, last is `length − 1`.
4. **Contiguous** — elements are stored next to each other, so the address of `arr[i]` = base address + i × element size. That is why access by index is **O(1)**.
5. **Arrays are objects** — created with `new`, stored in the heap, have a `length` field, extend `Object`, can be `null`, and are passed by reference value.
6. **Default values** — new array elements get defaults: `0`, `0.0`, `false`, `'\u0000'`, or `null`.

**Memory picture:**

```text
int[] marks = {90, 75, 82};

STACK                  HEAP
marks [ref ●]──────►  ┌────┬────┬────┐   length = 3
                      │ 90 │ 75 │ 82 │
                      └────┴────┴────┘
                        [0]  [1]  [2]

String[] names = {"Ravi", "Priya"};

names [ref ●]──────►  ┌──────┬──────┐
                      │  ●   │  ●   │      (array holds references)
                      └──┼───┴──┼───┘
                         ▼      ▼
                      "Ravi"  "Priya"
```

### 1.3 Declaring, Creating and Initialising

```text
int[] a;                       // declaration (preferred style)
int b[];                       // also valid (C-style)
a = new int[5];                // creation with size → {0, 0, 0, 0, 0}
int[] c = {10, 20, 30};        // declaration + initialisation (array literal)
int[] d = new int[]{10, 20};   // anonymous array; useful when passing directly to a method
int[] e = new int[0];          // empty array is valid (length 0)
// int[3] f;                   // INVALID: size is not part of the declaration
// int[] g = new int[3]{1,2,3};// INVALID: cannot give both size and values
```

A negative size, `new int[-1]`, compiles but throws `NegativeArraySizeException` at runtime.

### 1.4 Real-World Example

An array is like a **row of numbered lockers** in a college corridor. All lockers are the same size (same type), the number of lockers is fixed when the corridor is built (fixed size), and each locker has a number starting from 0 (index). To reach locker 37, you walk straight to it — you do not open lockers 0 to 36 first (O(1) access).

### 1.5 Java Example

```java
import java.util.Arrays;

public class ArrayBasics {
    public static void main(String[] args) {
        int[] marks = {78, 92, 65, 88, 71};

        System.out.println("Length: " + marks.length);
        System.out.println("First: " + marks[0] + ", Last: " + marks[marks.length - 1]);

        // Traversal with index (can modify)
        int sum = 0;
        int max = marks[0];
        for (int i = 0; i < marks.length; i++) {
            sum += marks[i];
            if (marks[i] > max) {
                max = marks[i];
            }
        }
        System.out.println("Sum: " + sum + ", Average: " + (double) sum / marks.length + ", Max: " + max);

        // Default values
        double[] prices = new double[3];
        boolean[] flags = new boolean[2];
        String[] names = new String[2];
        System.out.println("Defaults: " + Arrays.toString(prices) + " "
                + Arrays.toString(flags) + " " + Arrays.toString(names));

        // Printing an array directly prints type@hash, not elements
        System.out.println("Direct print looks like: " + marks.getClass().getSimpleName() + "@...");
        System.out.println("Arrays.toString: " + Arrays.toString(marks));

        // Index out of bounds
        try {
            System.out.println(marks[5]);
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Exception: " + e.getMessage());
        }

        // Arrays are objects
        System.out.println("Is an Object? " + (marks instanceof Object));
        System.out.println("Class name: " + marks.getClass().getName());
    }
}
```

### 1.6 Code Explanation

- `marks.length` is a **field** (no brackets). The last index is always `length − 1`.
- Finding the maximum: start with the first element and compare each next element.
- `(double) sum / marks.length` avoids integer division.
- New arrays are filled with default values — `0.0`, `false`, `null`.
- `System.out.println(marks)` would print something like `[I@1b6d3586` (`[I` = int array, then a hash). Use `Arrays.toString()` to print elements.
- Accessing index 5 of a length-5 array throws `ArrayIndexOutOfBoundsException` — Java checks bounds at runtime.
- `getClass().getName()` for `int[]` is `"[I"`; for `String[]` it is `"[Ljava.lang.String;"`.

### 1.7 Expected Output

```text
Length: 5
First: 78, Last: 71
Sum: 394, Average: 78.8, Max: 92
Defaults: [0.0, 0.0, 0.0] [false, false] [null, null]
Direct print looks like: int[]@...
Arrays.toString: [78, 92, 65, 88, 71]
Exception: Index 5 out of bounds for length 5
Is an Object? true
Class name: [I
```

### 1.8 Interview Answer — "What is an array in Java?"

> "An array is an object that stores a fixed number of elements of the same type in contiguous memory, and each element is accessed by a zero-based index. In Java, arrays are created on the heap using `new` or an array initializer, and every array has a `length` field. Once created, the size cannot be changed. Because elements are stored contiguously, accessing any element by index takes constant time. When an array is created, its elements get default values — zero for numbers, false for boolean and null for objects. Java checks the index at runtime, so an invalid index throws `ArrayIndexOutOfBoundsException` instead of corrupting memory. If we need a resizable structure, we use `ArrayList`."

---

## 2. The `java.util.Arrays` Class

```java
import java.util.Arrays;
import java.util.List;

public class ArraysUtilityDemo {
    public static void main(String[] args) {
        int[] nums = {42, 7, 19, 3, 25};

        Arrays.sort(nums);                                     // ascending, in place
        System.out.println("sorted      : " + Arrays.toString(nums));

        int pos = Arrays.binarySearch(nums, 19);               // array MUST be sorted
        System.out.println("binarySearch(19) index: " + pos);
        System.out.println("binarySearch(10): " + Arrays.binarySearch(nums, 10));

        int[] filled = new int[4];
        Arrays.fill(filled, 9);
        System.out.println("fill        : " + Arrays.toString(filled));

        int[] bigger = Arrays.copyOf(nums, 7);                 // pads with 0
        int[] part = Arrays.copyOfRange(nums, 1, 4);           // index 1..3
        System.out.println("copyOf(7)   : " + Arrays.toString(bigger));
        System.out.println("copyOfRange : " + Arrays.toString(part));

        int[] a = {1, 2, 3};
        int[] b = {1, 2, 3};
        System.out.println("a == b          : " + (a == b));
        System.out.println("a.equals(b)     : " + a.equals(b));
        System.out.println("Arrays.equals   : " + Arrays.equals(a, b));

        String[] cities = {"Pune", "Delhi", "Agra"};
        Arrays.sort(cities);
        System.out.println("sorted strings: " + Arrays.toString(cities));

        Integer[] boxed = {5, 1, 4};
        Arrays.sort(boxed, (x, y) -> y - x);                   // descending needs objects
        System.out.println("descending  : " + Arrays.toString(boxed));

        List<String> list = Arrays.asList(cities);             // fixed-size list view
        System.out.println("asList      : " + list);
        try {
            list.add("Goa");
        } catch (UnsupportedOperationException e) {
            System.out.println("asList().add -> UnsupportedOperationException");
        }

        int total = Arrays.stream(nums).sum();                 // Java 8
        System.out.println("stream sum  : " + total);
    }
}
```

**Expected Output:**

```text
sorted      : [3, 7, 19, 25, 42]
binarySearch(19) index: 2
binarySearch(10): -3
fill        : [9, 9, 9, 9]
copyOf(7)   : [3, 7, 19, 25, 42, 0, 0]
copyOfRange : [7, 19, 25]
a == b          : false
a.equals(b)     : false
Arrays.equals   : true
sorted strings: [Agra, Delhi, Pune]
descending  : [5, 4, 1]
asList      : [Agra, Delhi, Pune]
asList().add -> UnsupportedOperationException
stream sum  : 96
```

**Explanation:**
- `Arrays.sort(int[])` uses **Dual-Pivot Quicksort** for primitives (O(n log n) average). For objects, it uses **TimSort** (stable, O(n log n)).
- `binarySearch` returns the index if found. If not found, it returns **−(insertion point) − 1**. 10 would be inserted at index 2 → returns −3.
- `a.equals(b)` is `false` because arrays inherit `equals()` from `Object` (reference comparison). Use `Arrays.equals()` for content (and `Arrays.deepEquals()` for nested arrays).
- Sorting with a custom comparator (descending) requires an **object array** like `Integer[]`, not `int[]`.
- `Arrays.asList()` returns a **fixed-size** list backed by the array: `set()` works, but `add()`/`remove()` throw `UnsupportedOperationException`.

---

## 3. Copying Arrays

### 3.1 Interview Definition

> **Assigning one array variable to another (`b = a`) copies only the reference, so both variables point to the same array. To create a real copy, Java provides `clone()`, `Arrays.copyOf()`, `Arrays.copyOfRange()` and `System.arraycopy()`. These create a shallow copy: for primitive arrays, values are copied; for object arrays, only the references are copied, so both arrays share the same element objects. A deep copy creates new copies of the element objects as well.**

### 3.2 Java Example

```java
import java.util.Arrays;

public class ArrayCopyDemo {
    static class Student {
        String name;
        Student(String name) { this.name = name; }
        public String toString() { return name; }
    }

    public static void main(String[] args) {
        int[] original = {1, 2, 3};

        int[] sameRef = original;             // reference copy
        sameRef[0] = 100;
        System.out.println("After sameRef change, original = " + Arrays.toString(original));

        int[] cloned = original.clone();      // new array with copied values
        cloned[1] = 200;
        System.out.println("After clone change, original = " + Arrays.toString(original)
                + ", cloned = " + Arrays.toString(cloned));

        int[] target = new int[5];
        System.arraycopy(original, 0, target, 1, 3);   // src, srcPos, dest, destPos, length
        System.out.println("System.arraycopy: " + Arrays.toString(target));

        // Shallow copy of object array
        Student[] classA = {new Student("Ravi"), new Student("Priya")};
        Student[] shallow = classA.clone();
        shallow[0].name = "Changed";                    // same Student object!
        System.out.println("Shallow copy: classA = " + Arrays.toString(classA));

        // Deep copy: create new Student objects
        Student[] deep = new Student[classA.length];
        for (int i = 0; i < classA.length; i++) {
            deep[i] = new Student(classA[i].name);
        }
        deep[1].name = "Deep";
        System.out.println("Deep copy: classA = " + Arrays.toString(classA) + ", deep = " + Arrays.toString(deep));
    }
}
```

**Expected Output:**

```text
After sameRef change, original = [100, 2, 3]
After clone change, original = [100, 2, 3], cloned = [100, 200, 3]
System.arraycopy: [0, 100, 2, 3, 0]
Shallow copy: classA = [Changed, Priya]
Deep copy: classA = [Changed, Priya], deep = [Changed, Deep]
```

**Explanation:**
- `sameRef = original` → both names refer to one array; changing one changes "both".
- `clone()` creates a new array; changes to `cloned` do not affect `original`.
- `System.arraycopy` is a fast native method used internally by `ArrayList` and `Arrays.copyOf`.
- Cloning `Student[]` copies **references**: `shallow[0]` and `classA[0]` are the same `Student` object.
- The deep copy creates new `Student` objects, so changes do not affect the original.

---

## 4. Multidimensional Arrays

### 4.1 Interview Definition

> **A multidimensional array in Java is an array of arrays. A two-dimensional array such as `int[][] matrix = new int[3][4]` is an array of 3 references, where each reference points to a separate one-dimensional array of 4 integers. Because each row is an independent array, Java also supports jagged (ragged) arrays, in which different rows can have different lengths. Elements are accessed using one index per dimension, for example `matrix[row][column]`.**

### 4.2 Understand the Concept

```text
int[][] m = new int[3][4];

m ──► ┌───┐
      │ ● │──► [0, 0, 0, 0]     m[0]
      ├───┤
      │ ● │──► [0, 0, 0, 0]     m[1]
      ├───┤
      │ ● │──► [0, 0, 0, 0]     m[2]
      └───┘
m.length = 3 (rows), m[0].length = 4 (columns)
```

- `m.length` → number of rows. `m[i].length` → number of columns in row `i`.
- Only the first dimension is required at creation: `new int[3][]` creates 3 rows set to `null`; you create each row later (jagged array).
- `new int[][4]` is **invalid**.

### 4.3 Real-World Example

A 2D array is like a **cinema hall seating chart**: rows (A, B, C…) and seat numbers within each row. A jagged array is like a **theatre with curved rows**, where the front rows have fewer seats than the back rows.

### 4.4 Java Example — Matrix Operations and Jagged Arrays

```java
import java.util.Arrays;

public class MatrixDemo {
    static void print(int[][] m) {
        for (int[] row : m) {
            for (int value : row) {
                System.out.print(value + "\t");
            }
            System.out.println();
        }
    }

    public static void main(String[] args) {
        int[][] a = {
            {1, 2, 3},
            {4, 5, 6}
        };
        int[][] b = {
            {7, 8},
            {9, 10},
            {11, 12}
        };

        System.out.println("Rows of a = " + a.length + ", Columns of a = " + a[0].length);

        // Transpose of a (2x3 → 3x2)
        int[][] t = new int[a[0].length][a.length];
        for (int i = 0; i < a.length; i++) {
            for (int j = 0; j < a[0].length; j++) {
                t[j][i] = a[i][j];
            }
        }
        System.out.println("Transpose of a:");
        print(t);

        // Multiplication: (2x3) * (3x2) = (2x2)
        int[][] product = new int[a.length][b[0].length];
        for (int i = 0; i < a.length; i++) {
            for (int j = 0; j < b[0].length; j++) {
                for (int k = 0; k < b.length; k++) {
                    product[i][j] += a[i][k] * b[k][j];
                }
            }
        }
        System.out.println("a x b:");
        print(product);

        // Jagged array: Pascal's triangle
        int[][] pascal = new int[5][];
        for (int i = 0; i < pascal.length; i++) {
            pascal[i] = new int[i + 1];
            pascal[i][0] = pascal[i][i] = 1;
            for (int j = 1; j < i; j++) {
                pascal[i][j] = pascal[i - 1][j - 1] + pascal[i - 1][j];
            }
        }
        System.out.println("Pascal's triangle (jagged):");
        for (int[] row : pascal) {
            System.out.println(Arrays.toString(row));
        }

        System.out.println("deepToString: " + Arrays.deepToString(a));
    }
}
```

### 4.5 Code Explanation

- **Transpose:** element at `(i, j)` moves to `(j, i)`. The result has columns and rows swapped.
- **Matrix multiplication:** `product[i][j]` = sum of `a[i][k] × b[k][j]` for all `k`. Requires columns of `a` = rows of `b`. Time complexity **O(n³)** for square matrices.
  - `product[0][0]` = 1×7 + 2×9 + 3×11 = 7 + 18 + 33 = **58**.
- **Jagged array:** `new int[5][]` creates 5 empty row slots. Row `i` is created with `i + 1` columns. Each inner element is the sum of the two elements above it.
- `Arrays.deepToString()` prints nested arrays (plain `toString` would print row references).

### 4.6 Expected Output

```text
Rows of a = 2, Columns of a = 3
Transpose of a:
1	4	
2	5	
3	6	
a x b:
58	64	
139	154	
Pascal's triangle (jagged):
[1]
[1, 1]
[1, 2, 1]
[1, 3, 3, 1]
[1, 4, 6, 4, 1]
deepToString: [[1, 2, 3], [4, 5, 6]]
```

(Values are separated by tab characters.)

---

## 5. Array vs ArrayList

| Point | Array | `ArrayList` |
|---|---|---|
| Size | Fixed | Dynamic (grows automatically) |
| Elements | Primitives **and** objects | Objects only (uses wrappers for primitives) |
| Length | `arr.length` (field) | `list.size()` (method) |
| Generics | No (but typed) | Yes: `ArrayList<String>` |
| Performance | Slightly faster, less memory | Small overhead (boxing, resizing) |
| Multidimensional | Yes (`int[][]`) | Use `List<List<Integer>>` |
| Methods | Few (use `Arrays` utility) | Many (`add`, `remove`, `contains`, …) |
| Type checking | Runtime `ArrayStoreException` possible | Compile-time with generics |

---

## 6. Interview Questions with Complete Answers

### Basic Level

**Q1. Why do array indexes start at 0?**

Because the index is an **offset** from the start of the array. The address of `arr[i]` = base address + i × element size. The first element is at offset 0. This makes address calculation simple and fast. Java follows the C tradition.

**Q2. Can we change the size of an array after creating it?**

No. Arrays are fixed-size. To "resize", create a new bigger array and copy the elements (`Arrays.copyOf(arr, newSize)`). This is exactly what `ArrayList` does internally.

**Q3. What is the default value of array elements?**

`0` for `byte/short/int/long`, `0.0` for `float/double`, `'\u0000'` for `char`, `false` for `boolean`, and `null` for object references. This applies even when the array is a local variable.

### Intermediate Level

**Q4. What is the difference between `length`, `length()` and `size()`?**

`length` is a final field of arrays. `length()` is a method of `String` (and `CharSequence`). `size()` is a method of collections like `ArrayList`, `HashSet`, `HashMap`.

**Q5. How do you compare two arrays for equality?**

`==` and `arr1.equals(arr2)` both compare references. Use `Arrays.equals(arr1, arr2)` for one-dimensional content comparison and `Arrays.deepEquals()` for multidimensional arrays.

**Q6. What is a jagged array?**

A multidimensional array in which rows have different lengths. It is possible because a 2D array in Java is an array of separate 1D arrays. Example: `int[][] j = new int[3][]; j[0] = new int[2]; j[1] = new int[5];`. It saves memory when rows naturally have different sizes (like Pascal's triangle).

**Q7. What is `ArrayStoreException`?**

Arrays are **covariant**: a `String[]` can be assigned to an `Object[]` variable. But the actual array still only accepts `String`s. Storing another type throws `ArrayStoreException` at runtime:

```java
public class ArrayStoreDemo {
    public static void main(String[] args) {
        Object[] objects = new String[2];   // allowed: arrays are covariant
        objects[0] = "OK";
        try {
            objects[1] = 10;                 // Integer into a String[]
        } catch (ArrayStoreException e) {
            System.out.println("ArrayStoreException: " + e.getMessage());
        }
    }
}
```

**Expected Output:**

```text
ArrayStoreException: java.lang.Integer
```

Generics avoid this problem by checking types at compile time (`List<String>` is not a `List<Object>`).

### Advanced Level

**Q8. Is `int[]` a subclass of `Object[]`?**

No. `int[]` is a subclass of `Object`, but **not** of `Object[]`, because `int` is not an object. `String[]` *is* assignable to `Object[]` (covariance), because `String` is an `Object`. All arrays implement `Cloneable` and `Serializable`.

**Q9. What is the time complexity of common array operations?**

| Operation | Complexity |
|---|---|
| Access by index | O(1) |
| Search (unsorted) | O(n) |
| Search (sorted, binary search) | O(log n) |
| Insert/delete at end (if space) | O(1) |
| Insert/delete in middle | O(n) — elements must shift |
| Sort (`Arrays.sort`) | O(n log n) |

**Q10. How do you return multiple values or an array from a method, and what is an anonymous array?**

A method can return an array: `int[] getMinMax(int[] a) { return new int[]{min, max}; }`. `new int[]{min, max}` is an **anonymous array** — created and used without a variable name, often passed directly to a method like `sum(new int[]{1, 2, 3})`.

---

## 7. Follow-Up Questions

**F1. How do you convert an array to a list and a list to an array?**

Array → list: `Arrays.asList(arr)` (fixed-size) or `new ArrayList<>(Arrays.asList(arr))` (resizable) or `List.of(arr)` (immutable, Java 9). List → array: `list.toArray(new String[0])`. For `int[]`, use streams: `Arrays.stream(intArr).boxed().toList()`.

**F2. Why does `Arrays.asList(new int[]{1,2,3})` give a list of size 1?**

Because generics do not work with primitives. The whole `int[]` is treated as **one object**, so the result is `List<int[]>` with one element. Use `Integer[]` or `Arrays.stream(arr).boxed().toList()`.

**F3. What is the maximum size of an array?**

The index is an `int`, so the theoretical maximum is `Integer.MAX_VALUE` (about 2.1 billion). In practice, JVMs allow slightly less (around `Integer.MAX_VALUE − 8`), and you need enough heap memory.

---

## 8. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Loop `i <= arr.length` | Use `i < arr.length` (last index is `length − 1`). |
| Printing an array with `println(arr)` | Use `Arrays.toString(arr)` / `deepToString`. |
| `arr1.equals(arr2)` for content | Use `Arrays.equals`. |
| `b = a` thinking it copies | Copies only the reference; use `clone()` / `copyOf`. |
| `binarySearch` on unsorted array | Result is undefined; sort first. |
| `Arrays.asList(...).add()` | Fixed-size; throws `UnsupportedOperationException`. |
| `new int[][3]` | First dimension is mandatory: `new int[3][]`. |

---

## 9. Quick Revision

- Array = fixed-size, same-type, indexed (0-based), contiguous, **object** in heap, `length` field.
- Defaults: 0 / 0.0 / false / `'\u0000'` / null.
- Invalid index → `ArrayIndexOutOfBoundsException`; negative size → `NegativeArraySizeException`.
- `Arrays`: `toString`, `deepToString`, `sort` (dual-pivot quicksort for primitives, TimSort for objects), `binarySearch` (−(ins)−1), `fill`, `copyOf`, `copyOfRange`, `equals`, `deepEquals`, `asList`, `stream`.
- Copy: `=` (reference), `clone`/`copyOf`/`arraycopy` (shallow), manual loop (deep).
- 2D = array of arrays; jagged arrays allowed; `m.length` rows, `m[i].length` columns.
- Arrays are covariant → `ArrayStoreException`.

---

[← Previous: StringBuilder and StringBuffer](03-StringBuilder-and-StringBuffer.md) | [Back to README](../README.md) | [Next: Wrapper Classes →](05-Wrapper-Classes-Autoboxing-Unboxing.md)
