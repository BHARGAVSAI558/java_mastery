# Chapter 5.8 — Comparable and Comparator

[← Previous: Iterator and Fail-Fast](07-Iterator-ListIterator-Fail-Fast.md) | [Back to README](../README.md) | [Next: Collection Comparisons and Questions →](09-Collections-Comparisons-and-Interview-Questions.md)

---

## What You Will Learn

- Why sorting objects needs rules
- `Comparable` — natural ordering with `compareTo()`
- `Comparator` — custom/external ordering with `compare()`
- Java 8 comparator methods: `comparing`, `thenComparing`, `reversed`, `nullsFirst`, `naturalOrder`
- **Comparable vs Comparator** (very common question)
- Consistency with `equals()` and common bugs (subtraction overflow)

---

## 1. Why Do We Need Them?

`Collections.sort(list)` can sort `Integer` and `String` because Java knows how to compare them. But how should it sort `Student` objects — by roll number, name, CGPA? Java cannot guess. We must provide the comparison rule:
- **Inside the class** → `Comparable` (one natural order).
- **Outside the class** → `Comparator` (any number of orders).

The comparison method returns an `int`:

| Return value | Meaning |
|---|---|
| **Negative** | first object comes **before** the second |
| **Zero** | they are **equal** in ordering |
| **Positive** | first object comes **after** the second |

---

## 2. Comparable

### 2.1 Interview Definition

> **`Comparable<T>` is an interface in the `java.lang` package that defines the natural ordering of a class's objects. A class implements it and overrides the single method `int compareTo(T other)`, which returns a negative number, zero or a positive number when the current object is less than, equal to or greater than the other object. Classes such as `String`, `Integer`, `LocalDate` and `BigDecimal` implement `Comparable`. Methods like `Collections.sort()`, `Arrays.sort()`, `TreeSet` and `TreeMap` use this natural ordering by default. A class can have only one natural ordering.**

### 2.2 Java Example

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.TreeSet;

public class ComparableDemo {

    static class Student implements Comparable<Student> {
        final int rollNo;
        final String name;
        final double cgpa;

        Student(int rollNo, String name, double cgpa) {
            this.rollNo = rollNo;
            this.name = name;
            this.cgpa = cgpa;
        }

        @Override
        public int compareTo(Student other) {
            return Integer.compare(this.rollNo, other.rollNo);   // natural order: by roll number
        }

        @Override
        public String toString() { return rollNo + "-" + name + "(" + cgpa + ")"; }
    }

    public static void main(String[] args) {
        List<Student> students = new ArrayList<>(List.of(
                new Student(103, "Priya", 9.1),
                new Student(101, "Ravi", 8.2),
                new Student(104, "Asha", 8.2),
                new Student(102, "Kiran", 7.5)));

        Collections.sort(students);                              // uses compareTo
        System.out.println("Sorted by roll: " + students);

        TreeSet<Student> tree = new TreeSet<>(students);         // TreeSet also uses compareTo
        System.out.println("Lowest roll in TreeSet: " + tree.first());

        System.out.println("\"apple\".compareTo(\"banana\") = " + "apple".compareTo("banana"));
        System.out.println("Integer 10 vs 5 = " + Integer.valueOf(10).compareTo(5));
    }
}
```

**Expected Output:**

```text
Sorted by roll: [101-Ravi(8.2), 102-Kiran(7.5), 103-Priya(9.1), 104-Asha(8.2)]
Lowest roll in TreeSet: 101-Ravi(8.2)
"apple".compareTo("banana") = -1
Integer 10 vs 5 = 1
```

**Explanation:**
- `Student` implements `Comparable<Student>` and defines its natural order by `rollNo`.
- `Integer.compare(a, b)` returns −1, 0 or 1 safely (no overflow).
- `Collections.sort()` and `TreeSet` both use `compareTo` automatically.
- Without implementing `Comparable`, `Collections.sort(students)` would not compile (the method requires `T extends Comparable<? super T>`), and `new TreeSet<>().add(student)` would throw `ClassCastException` at runtime.

### 2.3 Real-World Example

**Natural ordering** is like how **roll numbers** are naturally used to arrange students in an attendance register — it's the default, built-in order for that kind of object.

---

## 3. Comparator

### 3.1 Interview Definition

> **`Comparator<T>` is a functional interface in the `java.util` package used to define a custom ordering for objects, separately from the class itself. It has one abstract method, `int compare(T o1, T o2)`, which returns a negative number, zero or a positive number. Because it is external to the class, we can create many comparators for the same class — for example by name, by salary, or by date — and we can sort classes that we cannot modify or that do not implement `Comparable`. Since Java 8, comparators can be created with lambdas and combined using methods such as `Comparator.comparing()`, `thenComparing()`, `reversed()`, `naturalOrder()`, `reverseOrder()` and `nullsFirst()`.**

### 3.2 Java Example — Old Style and Java 8 Style

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class ComparatorDemo {

    record Employee(String name, String dept, double salary, Integer age) { }

    // Old style: a separate class
    static class SalaryComparator implements Comparator<Employee> {
        @Override
        public int compare(Employee a, Employee b) {
            return Double.compare(a.salary(), b.salary());
        }
    }

    public static void main(String[] args) {
        List<Employee> staff = new ArrayList<>(List.of(
                new Employee("Ravi", "IT", 65000, 28),
                new Employee("Asha", "HR", 52000, 35),
                new Employee("Kiran", "IT", 65000, 24),
                new Employee("Dev", "Sales", 48000, null),
                new Employee("Bala", "HR", 70000, 41)));

        Collections.sort(staff, new SalaryComparator());
        System.out.println("By salary (class)      : " + names(staff));

        // Anonymous class (pre-Java 8)
        staff.sort(new Comparator<Employee>() {
            @Override
            public int compare(Employee a, Employee b) { return a.name().compareTo(b.name()); }
        });
        System.out.println("By name (anonymous)    : " + names(staff));

        // Lambda
        staff.sort((a, b) -> Double.compare(b.salary(), a.salary()));
        System.out.println("By salary desc (lambda): " + names(staff));

        // Java 8 comparator methods
        staff.sort(Comparator.comparing(Employee::dept)
                .thenComparing(Employee::salary, Comparator.reverseOrder())
                .thenComparing(Employee::name));
        System.out.println("Dept, salary desc, name: " + names(staff));

        staff.sort(Comparator.comparing(Employee::age, Comparator.nullsLast(Comparator.naturalOrder())));
        System.out.println("By age (nulls last)    : " + names(staff));

        staff.sort(Comparator.comparingDouble(Employee::salary).reversed());
        System.out.println("comparingDouble rev    : " + names(staff));

        String[] words = {"kiwi", "Banana", "apple", "Cherry"};
        Arrays.sort(words);
        System.out.println("Strings natural        : " + Arrays.toString(words));
        Arrays.sort(words, String.CASE_INSENSITIVE_ORDER);
        System.out.println("Case-insensitive       : " + Arrays.toString(words));
        Arrays.sort(words, Comparator.comparingInt(String::length).thenComparing(Comparator.reverseOrder()));
        System.out.println("By length, then Z-A    : " + Arrays.toString(words));
    }

    static List<String> names(List<Employee> list) {
        return list.stream().map(Employee::name).toList();
    }
}
```

### 3.3 Code Explanation

- **Separate class** (`SalaryComparator`) — reusable but verbose.
- **Anonymous class** — inline, but still verbose.
- **Lambda** — `(a, b) -> Double.compare(b.salary(), a.salary())` sorts descending by swapping `a` and `b`.
- `Comparator.comparing(Employee::dept)` — builds a comparator from a key extractor.
- `.thenComparing(Employee::salary, Comparator.reverseOrder())` — secondary key, descending.
- `.thenComparing(Employee::name)` — tie-breaker.
- `nullsLast(naturalOrder())` — handles `null` ages without `NullPointerException`.
- `comparingDouble(...)` avoids boxing for primitive keys; `.reversed()` flips the whole order.
- `String` natural order puts **uppercase before lowercase** (Unicode); `CASE_INSENSITIVE_ORDER` ignores case.

### 3.4 Expected Output

```text
By salary (class)      : [Dev, Asha, Ravi, Kiran, Bala]
By name (anonymous)    : [Asha, Bala, Dev, Kiran, Ravi]
By salary desc (lambda): [Bala, Kiran, Ravi, Asha, Dev]
Dept, salary desc, name: [Bala, Asha, Kiran, Ravi, Dev]
By age (nulls last)    : [Kiran, Ravi, Asha, Bala, Dev]
comparingDouble rev    : [Bala, Kiran, Ravi, Asha, Dev]
Strings natural        : [Banana, Cherry, apple, kiwi]
Case-insensitive       : [apple, Banana, Cherry, kiwi]
By length, then Z-A    : [kiwi, apple, Cherry, Banana]
```

Notice: `List.sort` is **stable** (TimSort) — employees with equal salary keep their previous relative order. That is why after sorting by name, "Kiran" stays before "Ravi" in the salary-descending sort (both 65000).

---

## 4. Comparable vs Comparator

### 4.1 Comparison Table

| Point | Comparable | Comparator |
|---|---|---|
| Package | `java.lang` | `java.util` |
| Method | `int compareTo(T o)` | `int compare(T o1, T o2)` |
| Implemented by | The class itself | A separate class / lambda / method reference |
| Number of orderings | **One** (natural ordering) | **Many** |
| Modifies the class? | Yes | No |
| Use when | There is one obvious default order (roll no., ID, date) | Different orders needed, or class can't be modified (third-party, `final`) |
| Used by | `Collections.sort(list)`, `Arrays.sort(arr)`, `TreeSet`, `TreeMap` by default | `Collections.sort(list, c)`, `list.sort(c)`, `new TreeSet<>(c)`, `stream.sorted(c)` |
| Functional interface | Technically single method, rarely used as lambda | Yes — commonly a lambda |
| Java 8 helper methods | — | `comparing`, `thenComparing`, `reversed`, `nullsFirst/Last`, `naturalOrder`, `reverseOrder` |

### 4.2 Interview Answer

> "Both are used to sort objects. **Comparable** is in `java.lang`, and the class itself implements it by overriding `compareTo()`. It defines the **natural ordering**, and a class can have only one — for example, `Student` sorted by roll number. `Collections.sort`, `TreeSet` and `TreeMap` use it by default. **Comparator** is in `java.util`; it's a separate object with a `compare(a, b)` method, so we can define **many different orderings** without changing the class — by name, by marks, by date — and we can sort classes we can't modify. Since Java 8, Comparator is usually written as a lambda or built with `Comparator.comparing()`, and combined with `thenComparing()` and `reversed()`. Both methods return negative, zero or positive. I use Comparable for the default order and Comparator for any additional or custom ordering."

---

## 5. Important Rules and Pitfalls

### 5.1 Don't Compare Using Subtraction

```java
public class SubtractionBug {
    public static void main(String[] args) {
        int a = Integer.MIN_VALUE;   // -2147483648
        int b = 1;
        int wrong = a - b;           // overflows to a large POSITIVE number
        int right = Integer.compare(a, b);
        System.out.println("a - b              = " + wrong + "  (says a > b: WRONG)");
        System.out.println("Integer.compare(a,b) = " + right + "  (says a < b: correct)");
    }
}
```

**Expected Output:**

```text
a - b              = 2147483647  (says a > b: WRONG)
Integer.compare(a,b) = -1  (says a < b: correct)
```

Use `Integer.compare`, `Long.compare`, `Double.compare` or `Comparator.comparingInt`, never `return a - b;` (safe only if you are certain values are small). For `double`, subtraction also breaks with fractions (`(int)(0.5 - 0.3)` is 0).

### 5.2 Contract of compareTo / compare

1. **Sign symmetry:** `sgn(compare(a, b)) == -sgn(compare(b, a))`.
2. **Transitivity:** if a > b and b > c, then a > c.
3. **Consistency:** if `compare(a, b) == 0`, then `a` and `b` compare the same way with every other element.
4. **Strongly recommended:** consistent with `equals()` — `compare(a, b) == 0` exactly when `a.equals(b)`. Otherwise `TreeSet`/`TreeMap` (which use compare) and `HashSet`/`HashMap` (which use equals) disagree about duplicates.

Breaking the contract can make `Collections.sort()` throw `IllegalArgumentException: Comparison method violates its general contract!`.

### 5.3 Sorting Methods Summary

| Code | Order used |
|---|---|
| `Collections.sort(list)` / `list.sort(null)` | Natural (`Comparable`) |
| `Collections.sort(list, cmp)` / `list.sort(cmp)` | Comparator |
| `Arrays.sort(arr)` | Natural |
| `Arrays.sort(arr, cmp)` | Comparator (object arrays only) |
| `new TreeSet<>(cmp)` / `new TreeMap<>(cmp)` | Comparator |
| `stream.sorted()` / `stream.sorted(cmp)` | Natural / Comparator |
| `Collections.reverseOrder()` / `Comparator.reverseOrder()` | Reverse natural |
| `Collections.reverseOrder(cmp)` / `cmp.reversed()` | Reverse comparator |

---

## 6. Interview Questions with Complete Answers

### Basic Level

**Q1. What does `compareTo()` return?**

A negative integer if this object is less than the argument, zero if they are equal in order, and a positive integer if it is greater. Only the sign matters, not the exact value.

**Q2. Can a class implement both Comparable and use Comparators?**

Yes. `Comparable` gives the default ordering; any number of `Comparator`s can provide additional orderings. For example, `String` implements `Comparable` (case-sensitive), and `String.CASE_INSENSITIVE_ORDER` is a `Comparator`.

**Q3. How do you sort a list in descending order?**

`list.sort(Comparator.reverseOrder())` for natural ordering, or `list.sort(myComparator.reversed())`, or `Collections.sort(list, Collections.reverseOrder())`.

### Intermediate Level

**Q4. When would you choose Comparator over Comparable?**

When you need more than one ordering (by name, by salary), when the class is from a library you can't modify, when the class's natural ordering isn't what you need in a particular place, or when you want to keep sorting logic separate from the model class.

**Q5. How do you sort by multiple fields?**

Chain comparators: `Comparator.comparing(Employee::dept).thenComparing(Employee::salary, Comparator.reverseOrder()).thenComparing(Employee::name)`. The first comparator decides; ties are broken by the next.

**Q6. Is `Collections.sort()` stable? Which algorithm does it use?**

Yes, it is stable — equal elements keep their relative order. For objects it uses **TimSort** (a hybrid of merge sort and insertion sort), O(n log n) worst case and very fast on partially sorted data. `Arrays.sort()` for primitives uses dual-pivot quicksort (not stable, which doesn't matter for primitives).

### Advanced Level

**Q7. What happens if `compareTo` is inconsistent with `equals` in a TreeSet?**

`TreeSet` uses only `compareTo`. If two different objects compare as 0, the second is treated as a duplicate and not added, even though `equals` says they're different. Example: a `TreeSet` of employees sorted only by salary would keep only one employee per salary. Fix by adding tie-breakers so that `compare` returns 0 only for truly equal objects.

**Q8. How do you handle `null` values when sorting?**

Wrap the comparator: `Comparator.nullsFirst(Comparator.naturalOrder())` or `nullsLast(...)`. For a nullable field: `Comparator.comparing(Employee::age, Comparator.nullsLast(Comparator.naturalOrder()))`. A plain comparator would throw `NullPointerException`.

**Q9. Write a comparator to sort strings by frequency of a character, then alphabetically.**

```java
import java.util.Arrays;
import java.util.Comparator;

public class FrequencyComparator {
    static long countA(String s) { return s.chars().filter(c -> c == 'a').count(); }

    public static void main(String[] args) {
        String[] words = {"banana", "apple", "avocado", "cherry", "papaya"};
        Arrays.sort(words, Comparator.comparingLong(FrequencyComparator::countA).reversed()
                                     .thenComparing(Comparator.naturalOrder()));
        System.out.println(Arrays.toString(words));
    }
}
```

**Expected Output:**

```text
[banana, papaya, avocado, apple, cherry]
```

"banana" and "papaya" have three 'a's (alphabetical tie-break puts banana first), "avocado" has two, "apple" one, "cherry" zero.

---

## 7. Follow-Up Questions

**F1. Is `Comparator` a functional interface even though it has many methods?**

Yes. It has exactly one abstract method, `compare(T, T)`. `equals(Object)` is also declared, but it is a public method of `Object` and does not count. All the others are default or static methods.

**F2. Can records implement Comparable?**

Yes: `record Point(int x, int y) implements Comparable<Point> { public int compareTo(Point o) { ... } }`.

**F3. Why does `Collections.sort(list)` not compile for a class without Comparable?**

Its signature is `<T extends Comparable<? super T>> void sort(List<T> list)`. The generic bound requires the element type to be `Comparable`, so the compiler rejects it.

---

## 8. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| `return a.age - b.age;` | Can overflow; use `Integer.compare`. |
| Comparator returning only 0 or 1 | Must return negative for "less than". |
| Inconsistent comparator in TreeSet/TreeMap | Elements silently dropped as duplicates. |
| Forgetting tie-breakers | Unstable business ordering; duplicates in tree collections. |
| Sorting `int[]` with a comparator | Comparators work on objects only; use `Integer[]` or streams. |
| Ignoring nulls | Use `nullsFirst`/`nullsLast`. |

---

## 9. Quick Revision

- **Comparable** (`java.lang`): `compareTo(T)`, inside the class, one natural order; used by `sort(list)`, `TreeSet`, `TreeMap`.
- **Comparator** (`java.util`): `compare(T, T)`, external, many orders, lambdas.
- Return: negative / zero / positive.
- Java 8: `comparing`, `comparingInt/Long/Double`, `thenComparing`, `reversed`, `naturalOrder`, `reverseOrder`, `nullsFirst/Last`.
- Never use subtraction; use `Integer.compare`.
- Keep comparison consistent with `equals` for sorted sets/maps.
- `Collections.sort`/`List.sort` = TimSort, stable.

---

[← Previous: Iterator and Fail-Fast](07-Iterator-ListIterator-Fail-Fast.md) | [Back to README](../README.md) | [Next: Collection Comparisons and Questions →](09-Collections-Comparisons-and-Interview-Questions.md)
