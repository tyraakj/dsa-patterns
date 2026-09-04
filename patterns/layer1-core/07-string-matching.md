# Pattern 07: String Matching

> **Layer**: 1 (Core Foundation)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Find the Index of the First Occurrence in a String](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/), [Repeated DNA Sequences](https://leetcode.com/problems/repeated-dna-sequences/), [Longest Happy Prefix](https://leetcode.com/problems/longest-happy-prefix/)

---

## 1. Mental Model & Core Concept

String Matching searches for occurrences of a pattern $P$ of length $M$ inside a haystack $T$ of length $N$. Naive comparison takes $O(N \cdot M)$ time. Advanced algorithms achieve linear $O(N + M)$ time by eliminating redundant character comparisons.

```text
Rolling Hash (Rabin-Karp):
Window:   [ a , b , c ]  ──► Hash_1 = (a*B^2 + b*B^1 + c*B^0) mod M
Slide 1:    [ b , c , d ]  ──► Hash_2 = ((Hash_1 - a*B^2) * B + d) mod M

In $O(1)$ arithmetic, subtract the leaving character's high-order weight
and add the incoming character's low-order weight.
```

Primary paradigms:
1. **Rabin-Karp (Rolling Hash)**: Maps fixed-size windows to an integer hash that updates in $O(1)$ time.
2. **KMP (Knuth-Morris-Pratt)**: Uses a Longest Prefix-Suffix (LPS) table to skip character comparisons upon mismatch.

---

## 2. Identification Signals ("When to Use")

- **Substrings of Fixed Length**: *"Find all 10-letter-long sequences that occur more than once"* $\implies$ Rabin-Karp or bitmask rolling hash.
- **Pattern Search**: *"Find the first occurrence of needle in haystack"* $\implies$ KMP or Rabin-Karp.
- **Prefix Equals Suffix**: *"Find the longest prefix that is also a suffix"* $\implies$ KMP LPS array construction.

---

## 3. Algorithmic Template / Pseudocode

```text
function BUILD_LPS(pattern):
    m = length(pattern)
    lps = array of size m initialized with 0
    prev_lps = 0
    i = 1
    
    while i < m:
        if pattern[i] == pattern[prev_lps]:
            prev_lps += 1
            lps[i] = prev_lps
            i += 1
        else:
            if prev_lps != 0:
                prev_lps = lps[prev_lps - 1]
            else:
                lps[i] = 0
                i += 1
    return lps
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import List

# --- Variant A: KMP (Knuth-Morris-Pratt) Pattern Search ---
def str_str(haystack: str, needle: str) -> int:
    if not needle:
        return 0
    
    n, m = len(haystack), len(needle)
    if m > n:
        return -1

    # Step 1: Precompute LPS (Longest Prefix Suffix) table
    lps = [0] * m
    prev_lps, i = 0, 1
    while i < m:
        if needle[i] == needle[prev_lps]:
            prev_lps += 1
            lps[i] = prev_lps
            i += 1
        else:
            if prev_lps != 0:
                prev_lps = lps[prev_lps - 1]
            else:
                lps[i] = 0
                i += 1

    # Step 2: Traverse haystack with pattern fallback
    h_idx = n_idx = 0
    while h_idx < n:
        if haystack[h_idx] == needle[n_idx]:
            h_idx += 1
            n_idx += 1
        else:
            if n_idx != 0:
                n_idx = lps[n_idx - 1]  # Fallback to previous longest prefix
            else:
                h_idx += 1

        if n_idx == m:
            return h_idx - m  # Match found at this starting index

    return -1

# --- Variant B: Rabin-Karp Rolling Hash (Repeated DNA Sequences) ---
def find_repeated_dna_sequences(s: str) -> List[str]:
    n = len(s)
    if n <= 10:
        return []

    # Map nucleotides to 2-bit integers
    to_int = {'A': 0, 'C': 1, 'G': 2, 'T': 3}
    seen = set()
    repeated = set()

    # Calculate initial hash for first 10-letter window
    bitmask = (1 << 20) - 1  # 20 bits mask for 10 characters (2 bits each)
    curr_hash = 0
    for i in range(10):
        curr_hash = (curr_hash << 2) | to_int[s[i]]
    seen.add(curr_hash)

    # Slide 10-letter window in O(1) time
    for i in range(10, n):
        # Shift left by 2 bits, add new 2 bits, mask out overflow beyond 20 bits
        curr_hash = ((curr_hash << 2) & bitmask) | to_int[s[i]]
        if curr_hash in seen:
            repeated.add(s[i - 9 : i + 1])
        else:
            seen.add(curr_hash)

    return list(repeated)
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of KMP in `str_str`:
1. `lps = [0] * m`:
   * Allocates Longest Proper Prefix that is also a Suffix lookup array.
2. `if needle[i] == needle[prev_lps]: prev_lps += 1; lps[i] = prev_lps; i += 1`:
   * Extends the matched prefix-suffix sequence by 1 character.
3. `else: if prev_lps != 0: prev_lps = lps[prev_lps - 1]`:
   * When mismatch occurs, falls back to the previous known prefix suffix without resetting all the way to 0.
4. `if haystack[h_idx] == needle[n_idx]: h_idx += 1; n_idx += 1`:
   * Increments matching pointers synchronously.
5. `else: if n_idx != 0: n_idx = lps[n_idx - 1]`:
   * **The Core KMP Optimization**: Never backtracks the `h_idx` in haystack! Only `n_idx` falls back according to the precomputed LPS table.
6. `if n_idx == m: return h_idx - m`:
   * Full needle matched; returns starting index.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(N + M)$ (Optimal)
- **Proof**:
  - Building the LPS array takes at most $2M$ steps because `i` increments monotonically and `prev_lps` decreases by at most $M$ throughout.
  - Scanning the haystack: `h_idx` strictly increments $N$ times and never moves backward. `n_idx` can only fall back as many times as it advanced.
  - Total comparisons $\le 2N + 2M \implies O(N + M)$.
- **Comparison to Naive**: Naive search checks $M$ characters for each of the $N - M + 1$ positions, resulting in $O(N \cdot M)$ worst-case time (e.g. `haystack = "AAAAAAAAB", needle = "AAAB"`).

### Space Complexity: $O(M)$ (Optimal)
- The LPS table stores $M$ integers, consuming $O(M)$ auxiliary memory.
- In Rabin-Karp bitmasking for DNA sequences, hashing 10 chars into a 20-bit integer enables $O(1)$ window transitions and compact integer set lookups.

---

## 7. Key Invariants & Common Pitfalls

1. **Haystack Pointer Monotonicity**:
   * In KMP, `h_idx` **must never decrement**. Backtracking `h_idx` destroys the $O(N)$ runtime guarantee.
2. **Rolling Hash Collisions & Modulo Overflow**:
   * When using Rabin-Karp with large alphabets, choose a large prime modulus ($10^9 + 7$) or perform double hashing to prevent spurious collision hits.

---

## 8. Canonical Problem Walkthrough

### Repeated DNA Sequences (LeetCode #187)
* **Problem**: Identify all 10-letter-long sequences in DNA string `s` that occur more than once.
* **Key Invariant**: Each nucleotide is 1 of 4 characters (`A, C, G, T`), encodable in 2 bits. A 10-letter sequence fits into $10 \times 2 = 20$ bits (a single 32-bit integer).
* **Optimality**: Rolling hash bitmask updates in $O(1)$ per position, achieving total $O(N)$ time with $O(N)$ memory for the integer set.
