# Chapter 2.2 — String Comparison and Important String Methods

[← Previous: String Basics and Pool](01-String-Basics-Pool-Immutability.md) | [Back to README](../README.md) | [Next: StringBuilder and StringBuffer →](03-StringBuilder-and-StringBuffer.md)

---

## What You Will Learn

- Four ways to compare strings: `==`, `equals()`, `equalsIgnoreCase()`, `compareTo()`
- How `compareTo()` calculates its result
- All important `String` methods, grouped by purpose, with output
- Modern methods: `isBlank()`, `strip()`, `repeat()`, `lines()` (Java 11), text blocks (Java 15)
- `split()` and regex pitfalls
- `null`-safe comparison

---

## 1. String Comparison

### 1.1 Interview Definition

> **Java provides several ways to compare strings. The `==` operator compares references, meaning it checks whether two variables point to the same object in memory. The `equals()` method compares the actual content character by character and is case-sensitive; `equalsIgnoreCase()` compares content while ignoring case. The `compareTo()` method compares two strings lexicographically (dictionary order) based on Unicode values and returns zero if they are equal, a negative number if the first string comes before the second, and a positive number if it comes after. For checking equality of content, we should always use `equals()`, not `==`.**

### 1.2 Understand the Concept

| Method | Compares | Returns | Case-sensitive |
|---|---|---|---|
| `==` | References (same object?) | `boolean` | — |
| `equals()` | Content | `boolean` | Yes |
| `equalsIgnoreCase()` | Content | `boolean` | No |
| `compareTo()` | Lexicographic order | `int` (0, negative, positive) | Yes |
| `compareToIgnoreCase()` | Lexicographic order | `int` | No |
| `contentEquals(CharSequence)` | Content with any `CharSequence` (e.g., `StringBuilder`) | `boolean` | Yes |

**How `compareTo()` works:**
1. Compare characters one by one from the start.
2. At the first position where characters differ, return `char1 - char2` (difference of Unicode values).
3. If all characters match up to the shorter length, return `length1 - length2`.

Examples:
- `"apple".compareTo("banana")` → `'a' - 'b'` = **−1** (apple comes first)
- `"cat".compareTo("car")` → at index 2: `'t' - 'r'` = 116 − 114 = **2**
- `"Java".compareTo("JavaScript")` → prefix matches → 4 − 10 = **−6**
- `"Zebra".compareTo("apple")` → `'Z'(90) - 'a'(97)` = **−7** (uppercase letters come before lowercase in Unicode)

**Never rely on the exact number** — only on the sign (negative/zero/positive). The exact values are implementation details.

### 1.3 Real-World Example

- `==` is asking: "Are you two holding **the same physical book**?"
- `equals()` is asking: "Do your two books have **exactly the same text**?"
- `compareTo()` is asking: "In a **dictionary**, which of your words comes first?"

### 1.4 Java Example

```java
public class StringCompareDemo {
    public static void main(String[] args) {
        String s1 = "Java";
        String s2 = "Java";
        String s3 = new String("Java");
        String s4 = "JAVA";

        System.out.println("s1 == s2              : " + (s1 == s2));
        System.out.println("s1 == s3              : " + (s1 == s3));
        System.out.println("s1.equals(s3)         : " + s1.equals(s3));
        System.out.println("s1.equals(s4)         : " + s1.equals(s4));
        System.out.println("s1.equalsIgnoreCase(s4): " + s1.equalsIgnoreCase(s4));

        System.out.println("apple vs banana : " + "apple".compareTo("banana"));
        System.out.println("cat vs car      : " + "cat".compareTo("car"));
        System.out.println("Java vs JavaScript : " + "Java".compareTo("JavaScript"));
        System.out.println("Zebra vs apple  : " + "Zebra".compareTo("apple"));
        System.out.println("Zebra vs apple (ignore case): " + "Zebra".compareToIgnoreCase("apple"));
        System.out.println("same            : " + "same".compareTo("same"));

        StringBuilder sb = new StringBuilder("Java");
        System.out.println("s1.equals(sb)        : " + s1.equals(sb));
        System.out.println("s1.contentEquals(sb) : " + s1.contentEquals(sb));

        // Null-safe comparison
        String input = null;
        System.out.println("\"yes\".equals(input) : " + "yes".equals(input));      // safe
        System.out.println("Objects.equals       : " + java.util.Objects.equals(input, null));
        try {
            System.out.println(input.equals("yes"));                            // NPE
        } catch (NullPointerException e) {
            System.out.println("input.equals(...) threw NullPointerException");
        }
    }
}
```

### 1.5 Code Explanation

- `s1 == s3` is `false` because `s3` is a different object, though the content is the same.
- `s1.equals(sb)` is `false` because `String.equals()` returns `true` **only for another `String`** with the same content. To compare with a `StringBuilder`, use `contentEquals()`.
- `"yes".equals(input)` — calling `equals` on the **literal** is safe even when `input` is `null` (it simply returns `false`). This is called the **"Yoda condition"** or *constant-first comparison*.
- `Objects.equals(a, b)` (Java 7) handles `null` on both sides.
- `input.equals("yes")` throws `NullPointerException` because you cannot call a method on `null`.

### 1.6 Expected Output

```text
s1 == s2              : true
s1 == s3              : false
s1.equals(s3)         : true
s1.equals(s4)         : false
s1.equalsIgnoreCase(s4): true
apple vs banana : -1
cat vs car      : 2
Java vs JavaScript : -6
Zebra vs apple  : -7
Zebra vs apple (ignore case): 25
same            : 0
s1.equals(sb)        : false
s1.contentEquals(sb) : true
"yes".equals(input) : false
Objects.equals       : true
input.equals(...) threw NullPointerException
```

`compareToIgnoreCase` treats `'Z'` and `'z'` the same, so `'z'(122) - 'a'(97) = 25`.

### 1.7 Interview Answer — "Difference between `==` and `equals()`"

> "`==` is an operator that compares **references** for objects — it checks whether both variables point to the **same object** in memory. `equals()` is a method that compares the **content**. In the `Object` class, `equals()` also compares references, but `String` overrides it to compare the characters one by one. So if I create two strings with `new String("Java")`, `==` returns false because they are two different objects, but `equals()` returns true because the content is the same. For string literals, `==` may return true because of the string pool, but we should never depend on that. For content comparison we always use `equals()`, and to avoid a NullPointerException we can write the constant first, like `"yes".equals(input)`, or use `Objects.equals()`."

---

## 2. Important String Methods

### 2.1 Methods Grouped by Purpose

| Purpose | Methods |
|---|---|
| Length & access | `length()`, `charAt(i)`, `isEmpty()`, `isBlank()` (11), `toCharArray()`, `chars()` |
| Searching | `indexOf()`, `lastIndexOf()`, `contains()`, `startsWith()`, `endsWith()`, `matches(regex)` |
| Extracting | `substring(begin)`, `substring(begin, end)`, `subSequence()` |
| Changing case | `toUpperCase()`, `toLowerCase()` |
| Trimming | `trim()`, `strip()`, `stripLeading()`, `stripTrailing()` (11) |
| Replacing | `replace(char, char)`, `replace(CharSequence, CharSequence)`, `replaceAll(regex, s)`, `replaceFirst(regex, s)` |
| Splitting & joining | `split(regex)`, `String.join(delim, parts)`, `concat()` |
| Repeating & lines | `repeat(n)` (11), `lines()` (11) |
| Converting | `String.valueOf(x)`, `toCharArray()`, `getBytes()`, `String.format()`, `formatted()` (15) |
| Comparing | `equals()`, `equalsIgnoreCase()`, `compareTo()`, `contentEquals()`, `regionMatches()` |
| Other | `intern()`, `hashCode()`, `indent()` (12), `translateEscapes()` (15) |

### 2.2 Java Example — Core Methods

```java
public class StringMethodsDemo {
    public static void main(String[] args) {
        String s = "Java Interview Preparation";

        // Length and access
        System.out.println("length()          : " + s.length());
        System.out.println("charAt(5)         : " + s.charAt(5));
        System.out.println("isEmpty()         : " + "".isEmpty());

        // Searching
        System.out.println("indexOf('a')      : " + s.indexOf('a'));
        System.out.println("lastIndexOf('a')  : " + s.lastIndexOf('a'));
        System.out.println("indexOf(\"view\")   : " + s.indexOf("view"));
        System.out.println("indexOf('a', 2)   : " + s.indexOf('a', 2));
        System.out.println("indexOf(\"xyz\")    : " + s.indexOf("xyz"));
        System.out.println("contains(\"Inter\") : " + s.contains("Inter"));
        System.out.println("startsWith(\"Java\"): " + s.startsWith("Java"));
        System.out.println("endsWith(\"tion\")  : " + s.endsWith("tion"));

        // Extracting: substring(begin, end) → end index is EXCLUSIVE
        System.out.println("substring(5)      : " + s.substring(5));
        System.out.println("substring(5, 14)  : " + s.substring(5, 14));

        // Case
        System.out.println("toUpperCase()     : " + s.toUpperCase());
        System.out.println("toLowerCase()     : " + s.toLowerCase());

        // Replacing
        System.out.println("replace('a','@')  : " + s.replace('a', '@'));
        System.out.println("replace(\"Java\",\"Kotlin\"): " + s.replace("Java", "Kotlin"));
        System.out.println("replaceAll(\"[aeiou]\",\"*\"): " + s.replaceAll("[aeiou]", "*"));

        // Splitting and joining
        String[] words = s.split(" ");
        System.out.println("split(\" \").length : " + words.length);
        System.out.println("String.join       : " + String.join("-", words));

        // Converting
        char[] chars = "abc".toCharArray();
        System.out.println("toCharArray()[1]  : " + chars[1]);
        System.out.println("String.valueOf(42): " + (String.valueOf(42) + 1));
        System.out.println("valueOf(chars)    : " + String.valueOf(chars));
    }
}
```

### 2.3 Expected Output

```text
length()          : 26
charAt(5)         : I
isEmpty()         : true
indexOf('a')      : 1
lastIndexOf('a')  : 21
indexOf("view")   : 10
indexOf('a', 2)   : 3
indexOf("xyz")    : -1
contains("Inter") : true
startsWith("Java"): true
endsWith("tion")  : true
substring(5)      : Interview Preparation
substring(5, 14)  : Interview
toUpperCase()     : JAVA INTERVIEW PREPARATION
toLowerCase()     : java interview preparation
replace('a','@')  : J@v@ Interview Prep@r@tion
replace("Java","Kotlin"): Kotlin Interview Preparation
replaceAll("[aeiou]","*"): J*v* Int*rv**w Pr*p*r*t**n
split(" ").length : 3
String.join       : Java-Interview-Preparation
toCharArray()[1]  : b
String.valueOf(42): 421
valueOf(chars)    : abc
```

### 2.4 Code Explanation

- Indexes start at **0**. `"Java Interview Preparation"`: J=0, a=1, v=2, a=3, space=4, I=5 …
- `indexOf` returns **−1** when not found.
- `indexOf('a', 2)` starts searching from index 2, finding the `'a'` at 3.
- `substring(5, 14)` → characters from index 5 **up to but not including** 14 → length = 14 − 5 = 9 → `"Interview"`.
- `replace` replaces **all** occurrences (it is not regex-based). `replaceAll` uses a **regular expression**: `[aeiou]` means "any one vowel" (lowercase only, so `I` stays).
- `String.valueOf(42) + 1` → `"42" + 1` → `"421"` (string concatenation).
- `charAt()` with an invalid index throws `StringIndexOutOfBoundsException`.

### 2.5 Java 11+ Methods and Text Blocks

```java
public class ModernStringMethods {
    public static void main(String[] args) {
        String spaces = "   ";
        System.out.println("isEmpty() : " + spaces.isEmpty());     // false: length is 3
        System.out.println("isBlank() : " + spaces.isBlank());     // Java 11: only whitespace → true

        String padded = "\u2003 Hello \u2003";                      // \u2003 is a Unicode "em space"
        System.out.println("original length : " + padded.length());
        System.out.println("trim() length   : " + padded.trim().length());    // trim removes only chars <= ' '
        System.out.println("strip() length  : " + padded.strip().length());   // Java 11: Unicode-aware
        System.out.println("strip() : [" + padded.strip() + "]");

        System.out.println("repeat  : " + "ab".repeat(3));          // Java 11
        System.out.println("lines   : " + "one\ntwo\nthree".lines().count());   // Java 11

        String json = """
                {
                  "name": "Ravi",
                  "skill": "Java"
                }
                """;                                                // Java 15 text block
        System.out.print(json);

        String msg = "Hello %s, you scored %d".formatted("Priya", 95);   // Java 15
        System.out.println(msg);
    }
}
```

**Expected Output:**

```text
isEmpty() : false
isBlank() : true
original length : 9
trim() length   : 9
strip() length  : 5
strip() : [Hello]
repeat  : ababab
lines   : 3
{
  "name": "Ravi",
  "skill": "Java"
}
Hello Priya, you scored 95
```

**Explanation:**
- `isEmpty()` → true only when length is 0. `isBlank()` → true when empty **or** only whitespace.
- `trim()` removes characters with code ≤ 32 (`' '`) only. It does **not** remove Unicode whitespace like `\u2003`, so `trim()` removes nothing here and the length stays 9 (the first and last characters are em spaces). `strip()` (Java 11) uses `Character.isWhitespace()` and removes all of them, leaving `"Hello"` with length 5.
- **Text blocks** (`"""`, Java 15) let you write multi-line strings without `\n` and escape characters. Indentation common to all lines is removed automatically.

### 2.6 `split()` Pitfalls

```java
import java.util.Arrays;

public class SplitDemo {
    public static void main(String[] args) {
        System.out.println(Arrays.toString("a,b,,c,,".split(",")));        // trailing empties removed
        System.out.println(Arrays.toString("a,b,,c,,".split(",", -1)));    // keep all
        System.out.println(Arrays.toString("1.2.3".split(".")));           // '.' is regex "any char"!
        System.out.println(Arrays.toString("1.2.3".split("\\.")));         // escape the dot
        System.out.println(Arrays.toString("a|b|c".split("\\|")));         // escape the pipe
        System.out.println(Arrays.toString("Java   is  fun".split("\\s+"))); // one or more spaces
        System.out.println(Arrays.toString("key=value=x".split("=", 2)));  // limit to 2 parts
    }
}
```

**Expected Output:**

```text
[a, b, , c]
[a, b, , c, , ]
[]
[1, 2, 3]
[a, b, c]
[Java, is, fun]
[key, value=x]
```

- `split` takes a **regular expression**. Special regex characters (`. | * + ? ( ) [ ] { } \ ^ $`) must be escaped with `\\`.
- `"1.2.3".split(".")` returns an **empty array**: every character matches `.`, so all pieces are empty, and trailing empty strings are removed.
- By default, **trailing empty strings are removed**. Pass a negative limit (`-1`) to keep them.

---

## 3. Interview Questions with Complete Answers

### Basic Level

**Q1. What is the difference between `length` and `length()`?**

`length` (no brackets) is a **field** of arrays: `arr.length`. `length()` is a **method** of `String` (and other `CharSequence` types): `str.length()`. Collections use `size()`. Mixing them up is a compile error.

**Q2. What does `substring(2, 5)` return for `"Programming"`?**

`"ogr"`. The begin index (2) is inclusive and the end index (5) is exclusive, so it returns characters at indexes 2, 3 and 4. The result length is always `end − begin` = 3.

**Q3. What is the difference between `isEmpty()` and `isBlank()`?**

`isEmpty()` returns `true` only when the length is 0. `isBlank()` (Java 11) returns `true` when the string is empty or contains only whitespace characters. `"  ".isEmpty()` is `false`, but `"  ".isBlank()` is `true`.

### Intermediate Level

**Q4. What is the difference between `replace()` and `replaceAll()`?**

Both replace **all** occurrences. `replace()` treats its arguments as **plain text** (literal characters/sequences). `replaceAll()` treats the first argument as a **regular expression**. So `"a.b.c".replace(".", "-")` gives `"a-b-c"`, but `"a.b.c".replaceAll(".", "-")` gives `"-----"` because `.` matches any character. `replaceFirst()` replaces only the first regex match.

**Q5. What is the difference between `trim()` and `strip()`?**

`trim()` (old method) removes leading and trailing characters whose code is ≤ `' '` (space and ASCII control characters). `strip()` (Java 11) removes all Unicode whitespace as defined by `Character.isWhitespace()`, such as the em space `\u2003`. Prefer `strip()` in modern code. `stripLeading()` and `stripTrailing()` remove from one side only.

**Q6. How does `compareTo()` work? What does it return?**

It compares two strings character by character using Unicode values. At the first difference, it returns the difference of the two characters. If one string is a prefix of the other, it returns the difference in lengths. It returns 0 for equal strings, a negative value if the calling string is lexicographically smaller, and a positive value if it is larger. It is used for sorting strings (`String` implements `Comparable<String>`).

**Q7. Why should we write `"constant".equals(variable)`?**

Because if `variable` is `null`, `variable.equals("constant")` throws `NullPointerException`, while `"constant".equals(null)` simply returns `false`. Alternatively, use `Objects.equals(a, b)`.

### Advanced Level

**Q8. How does `String.hashCode()` work?**

It uses the formula `s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]`, computed with `int` arithmetic (it can overflow). The result is cached in a private field after the first call, so later calls are instant. 31 is used because it is an odd prime, and `31 * i` can be optimised by the JVM as `(i << 5) - i`. For example, `"Ab".hashCode()` = 65 × 31 + 98 = 2113.

**Q9. Can two different strings have the same hash code?**

Yes — this is a **hash collision**. For example, `"Aa"` and `"BB"` both have hash code 2112 (65×31+97 = 2112 and 66×31+66 = 2112). That is why `HashMap` uses `equals()` after matching the hash code.

```java
public class HashCollision {
    public static void main(String[] args) {
        System.out.println("Aa".hashCode() + " " + "BB".hashCode());
        System.out.println("Aa".equals("BB"));
    }
}
```

**Expected Output:**

```text
2112 2112
false
```

**Q10. How do you count occurrences of a character in a string?**

```java
public class CountChar {
    public static void main(String[] args) {
        String text = "programming";
        char target = 'g';

        int count = 0;
        for (char c : text.toCharArray()) {
            if (c == target) count++;
        }
        System.out.println("Loop count  : " + count);

        long streamCount = text.chars().filter(c -> c == target).count();   // Java 8
        System.out.println("Stream count: " + streamCount);

        int replaceTrick = text.length() - text.replace(String.valueOf(target), "").length();
        System.out.println("Replace trick: " + replaceTrick);
    }
}
```

**Expected Output:**

```text
Loop count  : 2
Stream count: 2
Replace trick: 2
```

---

## 4. Follow-Up Questions

**F1. Is `switch` on strings case-sensitive?**

Yes, because it uses `equals()` internally. To ignore case, switch on `str.toLowerCase()`.

**F2. What does `toUpperCase()` depend on?**

It depends on the default **locale**. In Turkish locale, `"title".toUpperCase()` gives `"TİTLE"` (with a dotted İ). For locale-independent code (like keys or protocol names), use `toUpperCase(Locale.ROOT)`.

**F3. What happens with `"abc".substring(3)` and `"abc".substring(4)`?**

`substring(3)` returns an **empty string** `""` (begin index equal to length is allowed). `substring(4)` throws `StringIndexOutOfBoundsException`.

**F4. Does `substring` share the original char array?**

In **Java 6**, `substring` shared the original array (which could cause memory leaks when a small substring kept a huge string alive). Since **Java 7u6**, `substring` creates a **new array copy**, so no leak happens.

---

## 5. Common Mistakes

| Mistake | Correct understanding |
|---|---|
| Using `==` for content | Use `equals()`. |
| `str.length` | It is `str.length()`; arrays use `arr.length`. |
| `substring(begin, end)` end inclusive | End is **exclusive**. |
| `split(".")` or `split("|")` | Escape regex characters: `split("\\.")`, `split("\\|")`. |
| Forgetting to assign `s.toUpperCase()` | Strings are immutable; assign the result. |
| Relying on exact `compareTo` numbers | Only the sign is guaranteed meaningful. |
| `input.equals("x")` when input can be null | Use `"x".equals(input)`. |

---

## 6. Quick Revision

- `==` → reference; `equals()` → content; `equalsIgnoreCase()`; `compareTo()` → lexicographic int.
- `compareTo` = first char difference, else length difference. Uppercase < lowercase.
- `"x".equals(var)` or `Objects.equals()` for null safety.
- `indexOf` returns −1 if not found; `substring(b, e)` → `e` exclusive.
- `replace` = literal; `replaceAll` = regex.
- `trim` (≤ ' ') vs `strip` (Unicode, Java 11); `isEmpty` vs `isBlank` (11).
- Java 11: `repeat`, `lines`, `strip`, `isBlank`. Java 15: text blocks `"""`, `formatted`.
- `split` uses regex, removes trailing empties (use limit −1 to keep).
- `hashCode` = Σ s[i]·31^(n−1−i), cached; `"Aa"` and `"BB"` collide.

---

[← Previous: String Basics and Pool](01-String-Basics-Pool-Immutability.md) | [Back to README](../README.md) | [Next: StringBuilder and StringBuffer →](03-StringBuilder-and-StringBuffer.md)
