# Pattern 27: Bit Manipulation

> **Layer**: 2 (Intermediate High-Yield)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Single Number](https://leetcode.com/problems/single-number/), [Number of 1 Bits (Hamming Weight)](https://leetcode.com/problems/number-of-1-bits/), [Counting Bits](https://leetcode.com/problems/counting-bits/), [Reverse Bits](https://leetcode.com/problems/reverse-bits/), [Missing Number](https://leetcode.com/problems/missing-number/)

---

## 1. Mental Model & Core Concept

Bit Manipulation treats binary words directly as sets or arithmetic flags. It executes in single-cycle CPU instructions, unlocking maximum speed and zero memory overhead.

```text
The Master Bitwise Properties:
1. XOR Cancellation:     x ^ x = 0     and     x ^ 0 = x
   (Identical values annihilate each other; order does not matter!)
2. Brian Kernighan Trick: n & (n - 1)
   (Clears the lowest set bit in exactly one operation!)
   Example: 12 (1100) & 11 (1011) = 8 (1000)
3. Isolate Lowest Bit:    n & (-n)
4. Bitmasking Set:        (1 << i) represents element i in a set.
```

---

## 2. Identification Signals ("When to Use")

- **Finding Unique Non-Duplicate**: Every element appears twice except one $\implies$ XOR all numbers.
- **Tallying Set Bits**: Hamming weight, population count.
- **Powers of Two**: Checking if $N$ is a power of two ($N > 0 \text{ and } N \ \& \ (N - 1) == 0$).
- **Subsets via Bitmask**: Generating $2^N$ combinations using numbers from $0$ to $2^N - 1$.

---

## 3. Algorithmic Template / Pseudocode

```text
function BRIAN_KERNIGHAN_COUNT_BITS(n):
    count = 0
    while n > 0:
        n = n & (n - 1) // Drops lowest set bit
        count += 1
    return count
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import List

# --- Variant A: Single Number (XOR Cancellation) ---
def single_number(nums: List[int]) -> int:
    res = 0
    for num in nums:
        res ^= num  # Duplicates cancel to 0: a ^ b ^ a = b
    return res

# --- Variant B: Number of 1 Bits (Hamming Weight) ---
def hamming_weight(n: int) -> int:
    count = 0
    while n:
        n &= (n - 1)  # Clears least significant set bit
        count += 1
    return count

# --- Variant C: Counting Bits from 0 to N in O(N) ---
def count_bits(n: int) -> List[int]:
    # dp[i] = dp[i >> 1] + (i & 1)
    dp = [0] * (n + 1)
    for i in range(1, n + 1):
        dp[i] = dp[i >> 1] + (i & 1)
    return dp

# --- Variant D: Missing Number in Range [0, n] ---
def missing_number(nums: List[int]) -> int:
    res = len(nums)
    for i, num in enumerate(nums):
        res ^= i ^ num  # XOR index with value
    return res
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `single_number`:
1. `res = 0`:
   * Neutral identity for XOR ($x \oplus 0 = x$).
2. `for num in nums: res ^= num`:
   * XOR is commutative ($a \oplus b = b \oplus a$) and associative. Pairs of duplicates $x \oplus x$ evaluate to $0$, leaving only the solitary unique number.

### Breakdown of `hamming_weight` (Brian Kernighan's Algorithm):
1. `while n: n &= (n - 1); count += 1`:
   * Subtracting 1 flips all bits up to and including the lowest set bit. Bitwise ANDing $n$ with $n - 1$ resets that lowest 1-bit to 0.
   * The loop runs in **$O(K)$ iterations** where $K$ is the number of 1-bits, rather than $O(32)$ scanning all positions!

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(1)$ to $O(N)$ (Optimal)
- In `single_number`: Linear scan over $N$ integers with $O(1)$ bitwise operations $\implies O(N)$ time.
- In `hamming_weight`: Bounded by at most 32 operations for 32-bit integers $\implies O(1)$ time.
- In `count_bits`: Computes each number's bit count in $O(1)$ using the bit shift DP transition $\implies O(N)$ total time.

### Space Complexity: $O(1)$ (Optimal)
- Operates strictly in CPU registers without auxiliary heap memory.

---

## 7. Key Invariants & Common Pitfalls

1. **Python Infinite Precision Integers**:
   * *Trap*: Python integers do not overflow at 32 bits (they grow arbitrarily large). Bitwise NOT `~x` yields $-(x + 1)$ rather than unsigned complement.
   * *Fix*: To simulate 32-bit unsigned behavior in Python, mask with `0xFFFFFFFF`: `n = n & 0xFFFFFFFF`.
2. **Operator Precedence**:
   * *Trap*: Comparison operators (`==`, `!=`) have higher precedence than bitwise operators in Python!
   * *Wrong*: `if n & 1 == 0:` (evaluates as `n & (1 == 0) = n & False = 0`).
   * *Correct*: Always wrap bitwise operations in parentheses: `if (n & 1) == 0:`.

---

## 8. Canonical Problem Walkthrough

### Reverse Bits (LeetCode #190)
* **Problem**: Reverse the bits of a given 32-bit unsigned integer.

```python
def reverse_bits(n: int) -> int:
    res = 0
    for _ in range(32):
        res = (res << 1) | (n & 1)
        n >>= 1
    return res
```
* **Optimality**: Operates in exactly 32 steps $\implies O(1)$ time and $O(1)$ space.
