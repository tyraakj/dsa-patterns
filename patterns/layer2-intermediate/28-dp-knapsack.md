# Pattern 28: Dynamic Programming - Knapsack Style

> **Layer**: 2 (Intermediate High-Yield)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/), [Target Sum](https://leetcode.com/problems/target-sum/), [Coin Change II](https://leetcode.com/problems/coin-change-ii/), [Ones and Zeroes](https://leetcode.com/problems/ones-and-zeroes/)

---

## 1. Mental Model & Core Concept

Knapsack problems optimize item selection under a **finite capacity constraint $W$**.

```text
0/1 Knapsack vs Unbounded Knapsack Invariant:

0/1 Knapsack (Each item used AT MOST ONCE):
  Traverse capacity BACKWARDS (from Target down to Item Value):
  for num in nums:
      for w from Target down to num:
          dp[w] = dp[w] or dp[w - num]
  (Moving backwards ensures item 'num' is only added ONCE per subproblem!)

Unbounded Knapsack (Items can be REUSED infinitely):
  Traverse capacity FORWARDS (from Item Value up to Target):
  for num in nums:
      for w from num up to Target:
          dp[w] += dp[w - num]
```

State Definition:
- Let $dp[w]$ be the boolean reachability, count of ways, or maximum value achievable with total weight $w$.

---

## 2. Identification Signals ("When to Use")

- **Subset Partitioning**: Can array be partitioned into two subsets with equal sum ($\text{target} = \text{sum} / 2$).
- **Combinations Summing to Target (Zero-One)**: Number of ways to assign signs (+ / -) to reach target.
- **Unbounded Combinations**: Counting ways to make change using unlimited coins (*Coin Change II*).

---

## 3. Algorithmic Template / Pseudocode

```text
function ZERO_ONE_KNAPSACK_SUBSET_SUM(nums, target):
    dp = array of size (target + 1) filled with false
    dp[0] = true // Base case: 0 sum is always achievable

    for num in nums:
        for w from target down to num: // CRITICAL: Backwards traversal!
            dp[w] = dp[w] or dp[w - num]

    return dp[target]
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import List

# --- Variant A: Partition Equal Subset Sum (0/1 Knapsack Boolean) ---
def can_partition(nums: List[int]) -> bool:
    total = sum(nums)
    if total % 2 != 0:
        return False

    target = total // 2
    dp = [False] * (target + 1)
    dp[0] = True

    for num in nums:
        # Loop backwards so we only use each number once!
        for w in range(target, num - 1, -1):
            if dp[w - num]:
                dp[w] = True

    return dp[target]

# --- Variant B: Coin Change II (Unbounded Knapsack Combinations) ---
def change(amount: int, coins: List[int]) -> int:
    dp = [0] * (amount + 1)
    dp[0] = 1  # 1 way to make amount 0

    # Outer loop on coins guarantees combinations (no duplicate permutations like [1, 2] and [2, 1])
    for coin in coins:
        for w in range(coin, amount + 1):
            dp[w] += dp[w - coin]

    return dp[amount]

# --- Variant C: Target Sum (+/- Assignment to reach Target) ---
def find_target_sum_ways(nums: List[int], target: int) -> int:
    # Mathematical transformation:
    # Let P = positive subset, N = negative subset
    # sum(P) - sum(N) = target
    # sum(P) + sum(N) = total
    # 2 * sum(P) = target + total => sum(P) = (target + total) // 2
    total = sum(nums)
    if (total + target) % 2 != 0 or total < abs(target):
        return 0

    subset_target = (total + target) // 2
    dp = [0] * (subset_target + 1)
    dp[0] = 1

    for num in nums:
        for w in range(subset_target, num - 1, -1):
            dp[w] += dp[w - num]

    return dp[subset_target]
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `can_partition`:
1. `if total % 2 != 0: return False`:
   * An odd sum cannot be divided into two equal integer halves.
2. `dp = [False] * (target + 1); dp[0] = True`:
   * `dp[w]` is True if sum $w$ can be formed by some subset. Base case $\text{dp}[0] = \text{True}$ (empty set).
3. `for num in nums: for w in range(target, num - 1, -1):`:
   * **The Backwards Traversal Invariant**: When computing $\text{dp}[w]$ using $\text{dp}[w - num]$, we require that $\text{dp}[w - num]$ was computed *in the previous outer iteration* (i.e. without using the current `num`). If we traversed forward, `num` could be added multiple times to its own previous results, turning 0/1 Knapsack into Unbounded Knapsack!

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(N \times W)$ Pseudo-Polynomial (Optimal)
- **Proof**:
  - Outer loop runs $N$ times (for each item).
  - Inner loop runs $W = \text{target}$ times.
  - Total operations $= N \cdot W \implies O(N \times W)$.
  - Subset Sum is NP-Complete; pseudo-polynomial DP running in $O(N \times W)$ is the fastest known exact algorithm.

### Space Complexity: $O(W)$ (Optimal)
- A 2D table would take $O(N \times W)$ space. By traversing backwards, the entire 2D table is collapsed into a single 1D array of size $W + 1 \implies O(W)$ auxiliary memory.

---

## 7. Key Invariants & Common Pitfalls

1. **Permutations vs Combinations in Unbounded Knapsack**:
   * Outer loop over coins $\implies$ **Combinations** (order does not matter: $[1, 2]$ counted once).
   * Outer loop over target amount $\implies$ **Permutations** (order matters: $[1, 2]$ and $[2, 1]$ counted separately, like Combination Sum IV).
2. **Backwards vs Forwards Direction**:
   * 0/1 Knapsack: **Must loop backwards**.
   * Unbounded Knapsack: **Must loop forwards**.

---

## 8. Canonical Problem Walkthrough

### Ones and Zeroes (LeetCode #474 - 2D Knapsack)
* **Problem**: Maximum subset of binary strings with at most $m$ zeros and $n$ ones.
* **Two-Constraint Knapsack**: Maintain 2D matrix backwards in both dimensions!

```python
def find_max_form(strs: List[str], m: int, n: int) -> int:
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for s in strs:
        zeros = s.count('0')
        ones = len(s) - zeros
        for z in range(m, zeros - 1, -1):
            for o in range(n, ones - 1, -1):
                dp[z][o] = max(dp[z][o], 1 + dp[z - zeros][o - ones])
    return dp[m][n]
```
* **Optimality**: Operates in $O(S \times M \times N)$ time and $O(M \times N)$ space.
