# Chapter 2.7 — Core String and Array Coding Problems

[← Previous: Varargs and Pass-by-Value](06-Varargs-and-Pass-by-Value.md) | [Back to README](../README.md) | [Next Part: OOP →](../03-OOP/01-OOP-Introduction-Classes-and-Objects.md)

---

## About This Chapter

These are the **most common fresher-level coding questions** on strings and arrays. Interviewers use them to check that you can write clean loops, use `String`/`StringBuilder`/arrays correctly, and explain time and space complexity.

For every problem you get: problem statement → example → approach → dry run → complete Java solution → code explanation → output → complexity → variations/follow-ups.

More advanced problems (two pointers, sliding window, hashing, etc.) are in [Part 11](../11-Coding-Interview-Problems/01-Numbers-and-Basic-Logic.md).

| # | Problem | Level |
|---|---|---|
| 1 | Reverse a string (3 ways) | Beginner |
| 2 | Check palindrome string | Beginner |
| 3 | Check if two strings are anagrams | Beginner–Intermediate |
| 4 | Count vowels, consonants, digits and spaces | Beginner |
| 5 | Frequency of each character | Beginner–Intermediate |
| 6 | First non-repeating character | Intermediate |
| 7 | Remove duplicate characters | Intermediate |
| 8 | Capitalise the first letter of each word | Beginner |
| 9 | Largest and second largest element | Beginner–Intermediate |
| 10 | Reverse an array in place | Beginner |
| 11 | Find duplicate elements in an array | Intermediate |
| 12 | Check whether a string contains only digits | Beginner |

---

## Problem 1 — Reverse a String

**Problem statement:** Given a string, return its reverse.

**Example:** Input `"interview"` → Output `"weivretni"`

**Approaches:**
1. Use `StringBuilder.reverse()` — simplest and most practical.
2. Loop from the last character to the first and build the result.
3. Two pointers on a `char[]` — swap characters from both ends (in-place style, best to show logic).

**Dry run (two pointers) for `"java"`:**

| left | right | array after swap (start: `[j, a, v, a]`) |
|---|---|---|
| 0 | 3 | swap `j` and `a` → `[a, a, v, j]` |
| 1 | 2 | swap `a` and `v` → `[a, v, a, j]` |
| 2 | 1 | left ≥ right → stop |

Result: `"avaj"`.

```java
public class ReverseString {

    static String reverseWithBuilder(String s) {
        return new StringBuilder(s).reverse().toString();
    }

    static String reverseWithLoop(String s) {
        StringBuilder result = new StringBuilder();
        for (int i = s.length() - 1; i >= 0; i--) {
            result.append(s.charAt(i));
        }
        return result.toString();
    }

    static String reverseWithTwoPointers(String s) {
        char[] chars = s.toCharArray();
        int left = 0, right = chars.length - 1;
        while (left < right) {
            char temp = chars[left];
            chars[left] = chars[right];
            chars[right] = temp;
            left++;
            right--;
        }
        return new String(chars);
    }

    public static void main(String[] args) {
        String input = "interview";
        System.out.println("StringBuilder : " + reverseWithBuilder(input));
        System.out.println("Loop          : " + reverseWithLoop(input));
        System.out.println("Two pointers  : " + reverseWithTwoPointers(input));
        System.out.println("Empty string  : [" + reverseWithTwoPointers("") + "]");
    }
}
```

**Expected Output:**

```text
StringBuilder : weivretni
Loop          : weivretni
Two pointers  : weivretni
Empty string  : []
```

**Code explanation:**
- `reverseWithLoop` uses a `StringBuilder` rather than `result = result + c`, which would create a new `String` in every iteration.
- `reverseWithTwoPointers` converts the immutable string into a mutable `char[]`, swaps the ends and moves inward, then creates a new `String` from the array.
- An empty string works naturally because the loop condition is false immediately.

**Complexity:** Time **O(n)** for all three. Space **O(n)** (strings are immutable, so a new string/array is always needed).

**Variations / follow-ups:**
- *"Without using `reverse()` or any library method"* → use approach 2 or 3.
- *"Using recursion"* → `reverse(s.substring(1)) + s.charAt(0)` (O(n²) because of substring copies; mention this).
- *"Reverse only the words' order"* / *"reverse each word"* → see [Chapter 2.3, Q8](03-StringBuilder-and-StringBuffer.md).

---

## Problem 2 — Check Palindrome String

**Problem statement:** A palindrome reads the same forwards and backwards. Check whether a given string is a palindrome. Extended version: ignore case and non-alphanumeric characters.

**Examples:**
- `"madam"` → `true`
- `"Java"` → `false`
- `"A man, a plan, a canal: Panama"` → `true` (extended version)

**Approach (two pointers):** Compare the character at the left end with the character at the right end, then move both inward. If any pair differs, it is not a palindrome. For the extended version, skip characters that are not letters or digits and compare in lower case.

**Dry run for `"racecar"`:**

| left | right | s[left] | s[right] | equal? |
|---|---|---|---|---|
| 0 | 6 | r | r | ✅ |
| 1 | 5 | a | a | ✅ |
| 2 | 4 | c | c | ✅ |
| 3 | 3 | — | — | stop → palindrome |

```java
public class PalindromeCheck {

    static boolean isPalindrome(String s) {
        int left = 0, right = s.length() - 1;
        while (left < right) {
            if (s.charAt(left) != s.charAt(right)) {
                return false;                 // mismatch found → not a palindrome
            }
            left++;
            right--;
        }
        return true;
    }

    static boolean isPalindromeIgnoringSymbols(String s) {
        int left = 0, right = s.length() - 1;
        while (left < right) {
            char l = s.charAt(left);
            char r = s.charAt(right);
            if (!Character.isLetterOrDigit(l)) {
                left++;
            } else if (!Character.isLetterOrDigit(r)) {
                right--;
            } else {
                if (Character.toLowerCase(l) != Character.toLowerCase(r)) {
                    return false;
                }
                left++;
                right--;
            }
        }
        return true;
    }

    public static void main(String[] args) {
        System.out.println("madam   -> " + isPalindrome("madam"));
        System.out.println("racecar -> " + isPalindrome("racecar"));
        System.out.println("Java    -> " + isPalindrome("Java"));
        System.out.println("Madam (case-sensitive) -> " + isPalindrome("Madam"));
        String sentence = "A man, a plan, a canal: Panama";
        System.out.println("Sentence -> " + isPalindromeIgnoringSymbols(sentence));
        System.out.println("Shortcut -> " + new StringBuilder("level").reverse().toString().equals("level"));
    }
}
```

**Expected Output:**

```text
madam   -> true
racecar -> true
Java    -> false
Madam (case-sensitive) -> false
Sentence -> true
Shortcut -> true
```

**Code explanation:**
- The basic version returns `false` immediately at the first mismatch (early exit).
- The extended version moves a pointer past spaces and punctuation using `Character.isLetterOrDigit()` and compares lower-case characters.
- The shortcut using `StringBuilder.reverse()` is fine in practice but creates an extra string; interviewers usually want the two-pointer logic.

**Complexity:** Time **O(n)**, Space **O(1)** for two pointers (O(n) for the reverse shortcut).

**Variations:** Palindrome **number** (reverse the digits — see Part 11), longest palindromic substring (advanced, expand-around-center O(n²)), can the string become a palindrome by removing at most one character.

---

## Problem 3 — Anagram Check

**Problem statement:** Two strings are anagrams if they contain the same characters with the same frequencies, in any order. Check whether two strings are anagrams (ignore case and spaces).

**Examples:**
- `"listen"`, `"silent"` → `true`
- `"Dormitory"`, `"Dirty room"` → `true`
- `"hello"`, `"world"` → `false`

**Approach 1 (sort):** Clean both strings (remove spaces, lower case), sort their characters and compare. O(n log n).

**Approach 2 (count array):** For lowercase English letters, use an `int[26]`. Increase the count for each character of the first string and decrease it for each character of the second. If all counts are zero, they are anagrams. O(n).

**Dry run (count array) for `"listen"` vs `"silent"`:**
After processing, each of `l, i, s, t, e, n` is +1 then −1 → all 26 counts are 0 → anagrams.

```java
import java.util.Arrays;

public class AnagramCheck {

    static String clean(String s) {
        return s.replaceAll("\\s", "").toLowerCase();
    }

    static boolean isAnagramSort(String a, String b) {
        char[] x = clean(a).toCharArray();
        char[] y = clean(b).toCharArray();
        if (x.length != y.length) return false;
        Arrays.sort(x);
        Arrays.sort(y);
        return Arrays.equals(x, y);
    }

    static boolean isAnagramCount(String a, String b) {
        a = clean(a);
        b = clean(b);
        if (a.length() != b.length()) return false;
        int[] count = new int[26];
        for (int i = 0; i < a.length(); i++) {
            count[a.charAt(i) - 'a']++;      // 'a' → 0, 'b' → 1, ...
            count[b.charAt(i) - 'a']--;
        }
        for (int c : count) {
            if (c != 0) return false;
        }
        return true;
    }

    public static void main(String[] args) {
        System.out.println("listen/silent (sort)  : " + isAnagramSort("listen", "silent"));
        System.out.println("listen/silent (count) : " + isAnagramCount("listen", "silent"));
        System.out.println("Dormitory/Dirty room  : " + isAnagramCount("Dormitory", "Dirty room"));
        System.out.println("hello/world           : " + isAnagramCount("hello", "world"));
        System.out.println("aab/abb               : " + isAnagramSort("aab", "abb"));
    }
}
```

**Expected Output:**

```text
listen/silent (sort)  : true
listen/silent (count) : true
Dormitory/Dirty room  : true
hello/world           : false
aab/abb               : false
```

**Code explanation:**
- `replaceAll("\\s", "")` removes all whitespace; `toLowerCase()` makes the check case-insensitive.
- A length check first gives a quick `false` for different lengths.
- `c - 'a'` converts a lowercase letter into an index 0–25.
- `"aab"` vs `"abb"` have the same letters but different counts → not anagrams.

**Complexity:** Sort approach — Time **O(n log n)**, Space **O(n)**. Count approach — Time **O(n)**, Space **O(1)** (fixed 26-size array).

**Follow-ups:**
- *"What if the strings contain Unicode or any characters?"* → Use a `HashMap<Character, Integer>` instead of `int[26]`.
- *"Group a list of words into anagram groups"* → Use a map from sorted word → list of words (see Part 11, Hashing).

---

## Problem 4 — Count Vowels, Consonants, Digits and Spaces

**Problem statement:** Count vowels, consonants, digits, spaces and other characters in a string.

**Example:** `"Java 21 is Great!"` → vowels 5, consonants 6, digits 2, spaces 3, others 1.

**Approach:** Loop over each character. Convert it to lower case once. Check: vowel (`"aeiou".indexOf(ch) != -1`) → else letter → consonant; digit; space; otherwise other.

```java
public class CountCharacters {
    public static void main(String[] args) {
        String text = "Java 21 is Great!";
        int vowels = 0, consonants = 0, digits = 0, spaces = 0, others = 0;

        for (char c : text.toLowerCase().toCharArray()) {
            if ("aeiou".indexOf(c) != -1) {
                vowels++;
            } else if (c >= 'a' && c <= 'z') {
                consonants++;
            } else if (Character.isDigit(c)) {
                digits++;
            } else if (c == ' ') {
                spaces++;
            } else {
                others++;
            }
        }
        System.out.println("Vowels     : " + vowels);
        System.out.println("Consonants : " + consonants);
        System.out.println("Digits     : " + digits);
        System.out.println("Spaces     : " + spaces);
        System.out.println("Others     : " + others);
    }
}
```

**Expected Output:**

```text
Vowels     : 5
Consonants : 6
Digits     : 2
Spaces     : 3
Others     : 1
```

**Dry run summary:** `J`(consonant) `a`(vowel) `v`(c) `a`(v) ` `(space) `2`(digit) `1`(digit) ` `(space) `i`(v) `s`(c) ` `(space) `G`(c) `r`(c) `e`(v) `a`(v) `t`(c) `!`(other) → vowels: a, a, i, e, a = 5; consonants: j, v, s, g, r, t = 6.

**Complexity:** Time **O(n)**, Space **O(n)** for the lower-case copy (can be O(1) by calling `Character.toLowerCase` per character).

**Common mistake:** counting every non-vowel as a consonant — digits, spaces and symbols are not consonants.

---

## Problem 5 — Frequency of Each Character

**Problem statement:** Print how many times each character appears in a string, in the order of first appearance.

**Example:** `"programming"` → `p=1, r=2, o=1, g=2, a=1, m=2, i=1, n=1`

**Approach:** Use a `LinkedHashMap<Character, Integer>` — it keeps **insertion order**. For each character, increase its count using `getOrDefault` or `merge`.

```java
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.TreeMap;
import java.util.function.Function;
import java.util.stream.Collectors;

public class CharFrequency {
    public static void main(String[] args) {
        String s = "programming";

        // Approach 1: LinkedHashMap (keeps first-appearance order)
        Map<Character, Integer> freq = new LinkedHashMap<>();
        for (char c : s.toCharArray()) {
            freq.put(c, freq.getOrDefault(c, 0) + 1);
        }
        System.out.println("LinkedHashMap : " + freq);

        // Approach 2: int array for lowercase letters (alphabetical order)
        int[] counts = new int[26];
        for (char c : s.toCharArray()) {
            counts[c - 'a']++;
        }
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 26; i++) {
            if (counts[i] > 0) {
                sb.append((char) ('a' + i)).append('=').append(counts[i]).append(' ');
            }
        }
        System.out.println("Array         : " + sb.toString().trim());

        // Approach 3: Java 8 streams (sorted by key using TreeMap)
        Map<Character, Long> streamFreq = s.chars()
                .mapToObj(c -> (char) c)
                .collect(Collectors.groupingBy(Function.identity(), TreeMap::new, Collectors.counting()));
        System.out.println("Streams       : " + streamFreq);
    }
}
```

**Expected Output:**

```text
LinkedHashMap : {p=1, r=2, o=1, g=2, a=1, m=2, i=1, n=1}
Array         : a=1 g=2 i=1 m=2 n=1 o=1 p=1 r=2
Streams       : {a=1, g=2, i=1, m=2, n=1, o=1, p=1, r=2}
```

**Code explanation:**
- `getOrDefault(c, 0) + 1` returns the current count (or 0 if the character is new) and adds 1. An equivalent shorter form is `freq.merge(c, 1, Integer::sum)`.
- A `HashMap` would work too, but its iteration order is not guaranteed.
- The array approach is fastest for lowercase English letters.
- `groupingBy(identity, TreeMap::new, counting())` groups equal characters, keeps keys sorted, and counts each group.

**Complexity:** Time **O(n)** (O(n log k) with `TreeMap`), Space **O(k)** where k = number of distinct characters.

**Follow-ups:** most frequent character; characters that appear more than once; word frequency in a sentence (split by spaces and use the same map logic).

---

## Problem 6 — First Non-Repeating Character

**Problem statement:** Find the first character in a string that appears exactly once.

**Examples:** `"swiss"` → `'w'`; `"aabbcc"` → none.

**Approach (two passes):**
1. First pass: count every character (array or `LinkedHashMap`).
2. Second pass: go through the **string** again (to keep the original order) and return the first character whose count is 1.

**Dry run for `"swiss"`:** counts: s=3, w=1, i=1. Second pass: `s`(3) → skip, `w`(1) → **answer `w`**.

```java
public class FirstNonRepeating {

    static char firstUnique(String s) {
        int[] count = new int[256];               // covers extended ASCII
        for (char c : s.toCharArray()) {
            count[c]++;
        }
        for (char c : s.toCharArray()) {
            if (count[c] == 1) {
                return c;
            }
        }
        return '_';                               // marker meaning "not found"
    }

    public static void main(String[] args) {
        System.out.println("swiss     -> " + firstUnique("swiss"));
        System.out.println("interview -> " + firstUnique("interview"));
        System.out.println("aabbcc    -> " + firstUnique("aabbcc"));
    }
}
```

**Expected Output:**

```text
swiss     -> w
interview -> n
aabbcc    -> _
```

**Code explanation:**
- Using the `char` itself as an array index works because a `char` is a number (its code).
- For `"interview"` (`i n t e r v i e w`), the counts are i=2, n=1, t=1, e=2, r=1, v=1, w=1. Scanning the string in order: `i` has count 2 (skip), `n` has count 1 → answer **`n`**.
- We scan the **string** (not the count array) in the second pass, so the answer respects the original order.
- For `"aabbcc"` every count is 2, so the method returns the marker `'_'`.

**Complexity:** Time **O(n)** (two passes), Space **O(1)** (fixed 256-size array). For full Unicode use a `LinkedHashMap<Character, Integer>`.

**Follow-up:** *"Do it in one pass"* — use a `LinkedHashMap` for counts and then read the first entry with value 1 (iteration over the map is O(k)).

---

## Problem 7 — Remove Duplicate Characters

**Problem statement:** Remove duplicate characters from a string, keeping the first occurrence of each character.

**Example:** `"programming"` → `"progamin"`

**Approach:** Use a `LinkedHashSet<Character>` (keeps insertion order, ignores duplicates), or a `boolean[256]` "seen" array with a `StringBuilder`.

```java
import java.util.LinkedHashSet;
import java.util.Set;

public class RemoveDuplicateChars {

    static String removeDuplicatesSeen(String s) {
        boolean[] seen = new boolean[256];
        StringBuilder sb = new StringBuilder();
        for (char c : s.toCharArray()) {
            if (!seen[c]) {
                seen[c] = true;
                sb.append(c);
            }
        }
        return sb.toString();
    }

    static String removeDuplicatesSet(String s) {
        Set<Character> set = new LinkedHashSet<>();
        for (char c : s.toCharArray()) {
            set.add(c);                       // duplicates are ignored by a Set
        }
        StringBuilder sb = new StringBuilder();
        for (char c : set) {
            sb.append(c);
        }
        return sb.toString();
    }

    public static void main(String[] args) {
        System.out.println(removeDuplicatesSeen("programming"));
        System.out.println(removeDuplicatesSet("programming"));
        System.out.println(removeDuplicatesSeen("aaaa"));
    }
}
```

**Expected Output:**

```text
progamin
progamin
a
```

**Dry run (`"programming"`):** p(new) r(new) o(new) g(new) r(seen) a(new) m(new) m(seen) i(new) n(new) g(seen) → `p r o g a m i n`.

**Complexity:** Time **O(n)**, Space **O(n)** for the result.

**Variations:** remove characters that appear more than once *entirely* (`"programming"` → `"poain"`), remove adjacent duplicates (`"aabbca"` → `"abca"`).

---

## Problem 8 — Capitalise the First Letter of Each Word

**Problem statement:** Convert a sentence so that each word starts with a capital letter and the rest are lower case.

**Example:** `"java iS a POPULAR language"` → `"Java Is A Popular Language"`

```java
public class CapitalizeWords {
    static String capitalize(String sentence) {
        String[] words = sentence.trim().split("\\s+");
        StringBuilder result = new StringBuilder();
        for (String word : words) {
            result.append(Character.toUpperCase(word.charAt(0)))
                  .append(word.substring(1).toLowerCase())
                  .append(' ');
        }
        return result.toString().trim();
    }

    public static void main(String[] args) {
        System.out.println(capitalize("java iS a POPULAR language"));
        System.out.println(capitalize("   hello    world  "));
    }
}
```

**Expected Output:**

```text
Java Is A Popular Language
Hello World
```

**Code explanation:** `trim()` removes outer spaces and `split("\\s+")` splits on one or more spaces, so extra spaces do not create empty words. Each word becomes upper-case first character + lower-case remainder. The final `trim()` removes the last extra space.

**Complexity:** Time **O(n)**, Space **O(n)**.

---

## Problem 9 — Largest and Second Largest Element

**Problem statement:** Find the largest and second largest **distinct** elements in an integer array in a single pass.

**Example:** `{12, 35, 1, 10, 34, 35}` → largest 35, second largest 34.

**Approach:** Keep two variables, `first` and `second`, starting at `Integer.MIN_VALUE`. For each number:
- If it is greater than `first`: move `first` to `second`, and set `first` = number.
- Else if it is greater than `second` **and not equal to `first`**: set `second` = number.

**Dry run:**

| num | first | second |
|---|---|---|
| start | MIN | MIN |
| 12 | 12 | MIN |
| 35 | 35 | 12 |
| 1 | 35 | 12 |
| 10 | 35 | 12 |
| 34 | 35 | 34 |
| 35 | 35 | 34 (equal to first → ignored) |

```java
public class SecondLargest {
    static void findTopTwo(int[] arr) {
        if (arr.length < 2) {
            System.out.println("Need at least two elements");
            return;
        }
        int first = Integer.MIN_VALUE, second = Integer.MIN_VALUE;
        for (int num : arr) {
            if (num > first) {
                second = first;
                first = num;
            } else if (num > second && num != first) {
                second = num;
            }
        }
        if (second == Integer.MIN_VALUE) {
            System.out.println("Largest = " + first + ", no distinct second largest");
        } else {
            System.out.println("Largest = " + first + ", Second largest = " + second);
        }
    }

    public static void main(String[] args) {
        findTopTwo(new int[]{12, 35, 1, 10, 34, 35});
        findTopTwo(new int[]{5, 5, 5});
        findTopTwo(new int[]{-3, -1, -7});
    }
}
```

**Expected Output:**

```text
Largest = 35, Second largest = 34
Largest = 5, no distinct second largest
Largest = -1, Second largest = -3
```

**Complexity:** Time **O(n)** (one pass), Space **O(1)**. Sorting would be O(n log n) — mention that the single-pass solution is better.

**Follow-ups:** k-th largest element (use a min-heap `PriorityQueue` of size k → O(n log k)); handle the case when the array actually contains `Integer.MIN_VALUE` (use `Integer` objects with `null`, or a boolean flag).

---

## Problem 10 — Reverse an Array In Place

**Problem statement:** Reverse an array without using another array.

**Example:** `{1, 2, 3, 4, 5}` → `{5, 4, 3, 2, 1}`

```java
import java.util.Arrays;

public class ReverseArray {
    static void reverse(int[] arr) {
        int left = 0, right = arr.length - 1;
        while (left < right) {
            int temp = arr[left];
            arr[left] = arr[right];
            arr[right] = temp;
            left++;
            right--;
        }
    }

    public static void main(String[] args) {
        int[] a = {1, 2, 3, 4, 5};
        reverse(a);
        System.out.println(Arrays.toString(a));
        int[] b = {10, 20};
        reverse(b);
        System.out.println(Arrays.toString(b));
    }
}
```

**Expected Output:**

```text
[5, 4, 3, 2, 1]
[20, 10]
```

**Explanation:** The method changes the caller's array because the array reference is passed by value and both refer to the same array object ([Chapter 2.6](06-Varargs-and-Pass-by-Value.md)). The loop runs only `n/2` times.

**Complexity:** Time **O(n)**, Space **O(1)**.

**Follow-up:** reverse a `List` → `Collections.reverse(list)`.

---

## Problem 11 — Find Duplicate Elements in an Array

**Problem statement:** Print each element that appears more than once.

**Example:** `{4, 2, 7, 2, 9, 4, 4}` → `[4, 2]` (in order of first repetition found)

**Approaches:**
1. Nested loops — O(n²), no extra space (mention, but do not use for large data).
2. `HashSet` — if `add()` returns `false`, the element was already seen → O(n).
3. Sort first, then compare neighbours — O(n log n), O(1) extra space.

```java
import java.util.Arrays;
import java.util.HashSet;
import java.util.LinkedHashSet;
import java.util.Set;

public class FindDuplicates {
    public static void main(String[] args) {
        int[] arr = {4, 2, 7, 2, 9, 4, 4};

        // Approach 2: HashSet
        Set<Integer> seen = new HashSet<>();
        Set<Integer> duplicates = new LinkedHashSet<>();
        for (int num : arr) {
            if (!seen.add(num)) {          // add() returns false if already present
                duplicates.add(num);
            }
        }
        System.out.println("Duplicates (HashSet): " + duplicates);

        // Approach 3: sort and compare neighbours
        int[] copy = arr.clone();
        Arrays.sort(copy);                 // [2, 2, 4, 4, 4, 7, 9]
        System.out.print("Duplicates (sorting): ");
        for (int i = 1; i < copy.length; i++) {
            if (copy[i] == copy[i - 1] && (i == 1 || copy[i] != copy[i - 2])) {
                System.out.print(copy[i] + " ");
            }
        }
        System.out.println();
    }
}
```

**Expected Output:**

```text
Duplicates (HashSet): [2, 4]
Duplicates (sorting): 2 4 
```

**Code explanation:**
- `Set.add()` returns `false` when the element already exists — a neat trick to detect duplicates in one line.
- Using a `LinkedHashSet` for `duplicates` prints each duplicate only once, in the order it was first detected as a duplicate: 2 is detected at index 3, then 4 at index 5.
- In the sorting approach, the extra condition `copy[i] != copy[i - 2]` prevents printing 4 twice (it appears three times).

**Complexity:** HashSet — Time **O(n)**, Space **O(n)**. Sorting — Time **O(n log n)**, Space **O(1)** extra (excluding the copy).

**Follow-ups:** find the *count* of each duplicate (use a `HashMap`); array contains numbers 1..n with one duplicate — use Floyd's cycle detection or a sum formula (see Part 11).

---

## Problem 12 — Check Whether a String Contains Only Digits

**Problem statement:** Return `true` if the string is non-empty and every character is a digit.

```java
public class OnlyDigits {
    static boolean isNumericLoop(String s) {
        if (s == null || s.isEmpty()) return false;
        for (char c : s.toCharArray()) {
            if (c < '0' || c > '9') return false;
        }
        return true;
    }

    static boolean isNumericRegex(String s) {
        return s != null && s.matches("\\d+");     // one or more digits
    }

    static boolean isNumericStream(String s) {
        return s != null && !s.isEmpty() && s.chars().allMatch(Character::isDigit);
    }

    public static void main(String[] args) {
        String[] inputs = {"12345", "12a45", "", "007"};
        for (String in : inputs) {
            System.out.println("\"" + in + "\" -> " + isNumericLoop(in) + " " + isNumericRegex(in) + " " + isNumericStream(in));
        }
    }
}
```

**Expected Output:**

```text
"12345" -> true true true
"12a45" -> false false false
"" -> false false false
"007" -> true true true
```

**Complexity:** Time **O(n)**, Space **O(1)**.

**Note:** `Character.isDigit` also accepts digits from other scripts (like Devanagari ४), while `c >= '0' && c <= '9'` accepts only ASCII digits. Using `Integer.parseInt` inside `try/catch` is another approach but fails for very long numbers (overflow) and accepts a leading `+`/`-`.

---

## Quick Revision — Patterns from This Chapter

| Pattern | Used in |
|---|---|
| **Two pointers** (left/right moving inward) | Reverse string/array, palindrome |
| **Counting with an array** (`int[26]` / `int[256]`) | Anagram, frequency, first unique |
| **Map counting** (`getOrDefault`, `merge`) | Frequency with any characters |
| **Set for uniqueness** (`add()` returns false) | Duplicates, remove duplicate chars |
| **Single-pass tracking variables** | Largest/second largest |
| **StringBuilder for building results** | Every string-building problem |

**Always tell the interviewer:**
1. Your approach in one sentence before coding.
2. Edge cases: empty string/array, single element, all duplicates, negative numbers, case sensitivity, `null`.
3. Time and space complexity at the end.

---

[← Previous: Varargs and Pass-by-Value](06-Varargs-and-Pass-by-Value.md) | [Back to README](../README.md) | [Next Part: OOP →](../03-OOP/01-OOP-Introduction-Classes-and-Objects.md)
