# Pattern 06: String Manipulation & Parsing

> **Layer**: 1 (Core Foundation)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Valid Palindrome](https://leetcode.com/problems/valid-palindrome/), [Reverse Words in a String](https://leetcode.com/problems/reverse-words-in-a-string/), [String to Integer (atoi)](https://leetcode.com/problems/string-to-integer-atoi/), [Longest Common Prefix](https://leetcode.com/problems/longest-common-prefix/)

---

## 1. Mental Model & Core Concept

String manipulation problems require inspecting, sanitizing, tokenizing, or transforming character sequences without incurring accidental quadratic time penalties due to string immutability.

```text
State-Machine Parsing:
Input String: "   -42 with words"
                ▲
Scan:
 1. Whitespace State ──► Skip leading spaces
 2. Sign State       ──► Record '+' or '-'
 3. Digit State      ──► Accumulate: num = num * 10 + digit (clamp at 32-bit boundary)
 4. Termination      ──► Stop on first non-digit character
```

Key principles:
- **String Immutability in Python**: Concatenating strings inside a loop with `s += char` generates a new string object each time, turning an $O(N)$ loop into $O(N^2)$. Always accumulate in a list (`chars.append(c)`) and join at the end (`"".join(chars)`).
- **Two Pointers for In-Place Reversal**: Skip non-alphanumeric noise on the fly without creating intermediate filtered copies.

---

## 2. Identification Signals ("When to Use")

- **Sanitization & Palindrome Verification**: Ignoring whitespace, punctuation, and casing.
- **Lexical Scanning & Parsing**: Parsing integers from text, mathematical expressions, or structured path strings (`/a/./b/../../c/`).
- **Word Tokenization & Rearrangement**: Reversing word orders while preserving intra-word characters and trimming variable whitespace.

---

## 3. Algorithmic Template / Pseudocode

```text
function STATE_MACHINE_PARSER(s):
    index = 0
    n = length(s)
    
    // Step 1: Skip leading whitespace
    while index < n and s[index] is whitespace:
        index += 1
        
    // Step 2: Handle sign
    sign = +1
    if index < n and (s[index] == '+' or s[index] == '-'):
        if s[index] == '-': sign = -1
        index += 1
        
    // Step 3: Accumulate numerical digits
    result = 0
    while index < n and s[index] is digit:
        digit = s[index] - '0'
        result = result * 10 + digit
        // Handle 32-bit overflow/underflow clamping here
        index += 1
        
    return sign * result
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import List

# --- Variant A: Two-Pointer String Inspection (Valid Palindrome) ---
def is_palindrome(s: str) -> bool:
    left, right = 0, len(s) - 1

    while left < right:
        # Advance left past non-alphanumeric characters
        while left < right and not s[left].isalnum():
            left += 1
        # Advance right past non-alphanumeric characters
        while left < right and not s[right].isalnum():
            right -= 1

        if s[left].lower() != s[right].lower():
            return False

        left += 1
        right -= 1

    return True

# --- Variant B: Word Tokenization & Reversal ---
def reverse_words(s: str) -> str:
    # Python split() handles arbitrary contiguous whitespace automatically
    words = s.split()
    return " ".join(reversed(words))

# --- Variant C: Deterministic Finite Automaton (String to Integer atoi) ---
def my_atoi(s: str) -> int:
    i = 0
    n = len(s)
    sign = 1
    result = 0
    INT_MAX = 2**31 - 1
    INT_MIN = -2**31

    # 1. Discard leading whitespace
    while i < n and s[i] == ' ':
        i += 1

    # 2. Check sign
    if i < n and (s[i] == '+' or s[i] == '-'):
        if s[i] == '-':
            sign = -1
        i += 1

    # 3. Parse digits with overflow clamping
    while i < n and s[i].isdigit():
        digit = int(s[i])

        # Clamp against 32-bit signed boundaries
        if result > (INT_MAX - digit) // 10:
            return INT_MAX if sign == 1 else INT_MIN

        result = result * 10 + digit
        i += 1

    return sign * result
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `is_palindrome`:
1. `left, right = 0, len(s) - 1`:
   * Positions pointers at opposite string boundaries.
2. `while left < right and not s[left].isalnum(): left += 1`:
   * Filters out whitespace, commas, and punctuation on-the-fly without allocating a sanitized copy of the string.
3. `while left < right and not s[right].isalnum(): right -= 1`:
   * Symmetric filtration for the right pointer.
4. `if s[left].lower() != s[right].lower(): return False`:
   * Converts characters to lowercase and performs equality check. If mismatched, terminates early.
5. `left += 1; right -= 1`:
   * Converges inward once the current pair is validated.
6. `return True`:
   * String is confirmed palindromic.

### Breakdown of `my_atoi`:
1. `INT_MAX = 2**31 - 1; INT_MIN = -2**31`:
   * Constants defining 32-bit signed integer limits.
2. `while i < n and s[i] == ' ': i += 1`:
   * Safely skips leading spaces up to the end of string.
3. `if i < n and (s[i] == '+' or s[i] == '-'):`:
   * Reads optional sign character.
4. `if result > (INT_MAX - digit) // 10:`:
   * **Overflow Prevention Invariant**: Evaluates whether `result * 10 + digit > INT_MAX` before actually performing the multiplication. If it would overflow, clamps immediately to `INT_MAX` or `INT_MIN`.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(N)$ (Optimal)
- **Lower Bound**: To parse an arbitrary string of length $N$, every character up to the termination condition must be inspected at least once ($\Omega(N)$ lower bound).
- **Execution**: Both pointers in `is_palindrome` move monotonically inward, visiting each character at most once ($N$ steps). In `my_atoi`, index $i$ increments monotonically from $0$ to at most $N$. Total runtime is strictly $O(N)$.

### Space Complexity: $O(1)$ (Optimal)
- `is_palindrome` operates in-place using two integer indices, achieving $O(1)$ auxiliary space. (Naive solutions creating `filtered = [c for c in s if c.isalnum()]` consume $O(N)$ memory).
- `my_atoi` uses scalar accumulator variables, consuming $O(1)$ auxiliary space.

---

## 7. Key Invariants & Common Pitfalls

1. **Inner While Loop Bounds**:
   * When skipping whitespace or non-alphanumeric characters with inner loops (`while s[left]...`), you **must** include `while left < right` in the inner condition to prevent `IndexError: string index out of range`.
2. **Accidental $O(N^2)$ String Concatenation**:
   * In Python, `res += s[i]` allocates a brand-new string of length $|res| + 1$ and copies all characters over. Over $N$ steps, total work is $\sum_{k=1}^N k = \frac{N(N+1)}{2} = O(N^2)$. Always append to a list and use `"".join()`.

---

## 8. Canonical Problem Walkthrough

### Longest Common Prefix (LeetCode #14)
* **Problem**: Find the longest common prefix string amongst an array of strings.
* **Vertical Scanning Template**:
```python
def longest_common_prefix(strs: List[str]) -> str:
    if not strs:
        return ""
    
    # Iterate through characters of the first string
    for col in range(len(strs[0])):
        char = strs[0][col]
        for row in range(1, len(strs)):
            # If current string ends or character differs, return prefix up to col
            if col == len(strs[row]) or strs[row][col] != char:
                return strs[0][:col]
                
    return strs[0]
```
* **Optimality**: Operates in $O(S)$ time (where $S$ is the sum of characters across all strings) and $O(1)$ space.
