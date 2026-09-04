# Pattern 13: Dynamic Programming - 1D

> **Tier**: 1 (Core Foundation)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Climbing Stairs](https://leetcode.com/problems/climbing-stairs/), [House Robber](https://leetcode.com/problems/house-robber/), [House Robber II](https://leetcode.com/problems/house-robber-ii/), [Decode Ways](https://leetcode.com/problems/decode-ways/), [Coin Change](https://leetcode.com/problems/coin-change/), [Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/)

---

## 1. Mental Model & Core Concept

Dynamic Programming (DP) solves complex problems by breaking them down into **overlapping subproblems** that exhibit **optimal substructure**. In 1D DP, the state at index $i$ depends only on a fixed window of earlier states:

```text
Fibonacci / Climbing Stairs State Transition:
dp[i] = dp[i - 1] + dp[i - 2]

Instead of allocating a full array dp of size N:
prev2    prev1    curr
  ▼        ▼        ▼
[ 1   ,    2   ,   (3) ]
           ──►     ──► Shift pointers forward: prev2 = prev1, prev1 = curr
```

Core Evolution:
1. **Recursion with Memoization (Top-Down)**: Start from target $N$, cache results in a hash table or array.
2. **Tabulation (Bottom-Up)**: Build from base cases up to $N$.
3. **Space Optimization ($O(1)$ Memory)**: If $dp[i]$ only depends on the last $K$ states, discard older states and maintain only $K$ scalar variables.

---

## 2. Identification Signals ("When to Use")

- **Extremes or Counts Over Sequences**: *"Find the maximum profit"*, *"Find minimum cost to climb stairs"*, *"Count distinct ways to decode"*.
- **Subproblem Overlap**: Solving choice at step $i$ re-evaluates the same subproblems repeatedly.
- **No Future Influence**: Making an optimal choice at step $i$ does not alter or invalidate options available at step $i+1$ (Markov property).

---

## 3. Algorithmic Template / Pseudocode

```text
function DP_1D_OPTIMIZED(nums):
    if length(nums) == 0: return 0
    if length(nums) == 1: return nums[0]
    
    prev2 = base_case_0
    prev1 = base_case_1
    
    for i from 2 to length(nums) - 1:
        current = transition_function(prev1, prev2, nums[i])
        prev2 = prev1
        prev1 = current
        
    return prev1
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import List

# --- Variant A: House Robber (Non-Adjacent Maximum) ---
def rob(nums: List[int]) -> int:
    # State: rob1 = max loot up to house i - 2, rob2 = max loot up to house i - 1
    rob1, rob2 = 0, 0

    for n in nums:
        # Decision at house n: rob this house (n + rob1) OR skip it (rob2)
        new_rob = max(n + rob1, rob2)
        rob1 = rob2
        rob2 = new_rob

    return rob2

# --- Variant B: Coin Change (Unbounded 1D Knapsack Minimum) ---
def coin_change(coins: List[int], amount: int) -> int:
    # dp[i] represents minimum coins needed to make amount i
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0  # Base case: 0 coins needed for amount 0

    for a in range(1, amount + 1):
        for c in coins:
            if a - c >= 0:
                dp[a] = min(dp[a], 1 + dp[a - c])

    return dp[amount] if dp[amount] != float('inf') else -1

# --- Variant C: Longest Increasing Subsequence (Patience Sorting) ---
def length_of_lis(nums: List[int]) -> int:
    import bisect
    tails = []  # tails[i] stores the smallest tail of all increasing subsequences of length i + 1

    for x in nums:
        idx = bisect.bisect_left(tails, x)
        if idx == len(tails):
            tails.append(x)
        else:
            tails[idx] = x

    return len(tails)
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `rob`:
1. `rob1, rob2 = 0, 0`:
   * `rob1` represents $\text{dp}[i-2]$ and `rob2` represents $\text{dp}[i-1]$. Both initialize to 0 for virtual houses prior to index 0.
2. `for n in nums:`:
   * Linearly processes each house loot value.
3. `new_rob = max(n + rob1, rob2)`:
   * **Recurrence Relation**: At house $i$, we have two mutually exclusive choices:
     - Rob house $i$: gain $n$ plus the best outcome from 2 houses prior (`rob1`).
     - Skip house $i$: retain the best outcome up to the adjacent house (`rob2`).
4. `rob1 = rob2; rob2 = new_rob`:
   * Shifts the two-element sliding window forward by 1 step in $O(1)$ time.
5. `return rob2`:
   * Returns maximum loot after evaluating all houses.

### Breakdown of `coin_change`:
1. `dp = [float('inf')] * (amount + 1); dp[0] = 0`:
   * Initializes DP table with $\infty$. Base case $\text{dp}[0] = 0$.
2. `for a in range(1, amount + 1):`:
   * Iterates through every sub-amount from 1 up to target `amount`.
3. `for c in coins: if a - c >= 0: dp[a] = min(dp[a], 1 + dp[a - c])`:
   * Tries every coin denomination. If valid, takes 1 coin plus optimal solution for remaining amount `a - c`.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity:
- **`rob`**: Traverses $N$ houses with $O(1)$ scalar arithmetic per step $\implies O(N)$ time. Optimal since every house must be inspected.
- **`coin_change`**: $O(A \cdot C)$ where $A = \text{amount}$ and $C = \text{len}(coins)$.
- **`length_of_lis`**: For each of the $N$ numbers, `bisect_left` performs a binary search over `tails` ($\le N$) taking $O(\log N)$ time. Total runtime is $O(N \log N)$, beating the naive $O(N^2)$ DP formulation.

### Space Complexity:
- **`rob`**: Space-optimized from $O(N)$ down to $O(1)$ auxiliary memory by tracking only the previous 2 states.
- **`length_of_lis`**: `tails` array grows to at most $N$ elements $\implies O(N)$ space.

---

## 7. Key Invariants & Common Pitfalls

1. **Circular Array Variants (House Robber II)**:
   * If houses are arranged in a circle, house $0$ and house $N-1$ are neighbors.
   * *Invariant Solution*: Solve two linear subproblems: `rob(nums[1:])` and `rob(nums[:-1])` and take the maximum: `max(rob(nums[1:]), rob(nums[:-1]))`.
2. **Unreachable State Sentinel**:
   * Use `float('inf')` (or `amount + 1`) to mark unreachable states in min-cost problems. Guard against comparing with invalid states.

---

## 8. Canonical Problem Walkthrough

### Climbing Stairs (LeetCode #70)
* **Problem**: Each time you can climb 1 or 2 steps. How many distinct ways to reach the top ($n$)?
* **Recurrence**: $\text{ways}(n) = \text{ways}(n-1) + \text{ways}(n-2)$.

```python
def climb_stairs(n: int) -> int:
    if n <= 2:
        return n
    prev2, prev1 = 1, 2
    for _ in range(3, n + 1):
        prev2, prev1 = prev1, prev2 + prev1
    return prev1
```
* **Optimality**: Operates in $O(N)$ time and $O(1)$ space.
