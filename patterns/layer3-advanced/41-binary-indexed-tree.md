# Pattern 41: Binary Indexed Tree (Fenwick Tree)

> **Layer**: 3 (Advanced & Specialized)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Range Sum Query - Mutable](https://leetcode.com/problems/range-sum-query-mutable/), [Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/), [Create Sorted Array through Instructions](https://leetcode.com/problems/create-sorted-array-through-instructions/)

---

## 1. Mental Model & Core Concept

A Binary Indexed Tree (BIT), or Fenwick Tree, is an exceptionally compact data structure that supports **Prefix Sum Queries** and **Point Updates** in $O(\log N)$ time, using **only an array of size $N + 1$** (a 75% memory savings compared to Segment Trees!).

```text
The Magic of Lowbit:
lowbit(i) = i & (-i)
Isolates the lowest set bit in the binary representation of i.

Examples:
6 (0110) & (-6) = 2 (0010)
8 (1000) & (-8) = 8 (1000)

Tree Navigation via Lowbit:
- To QUERY Prefix Sum up to i:  i -= i & (-i) (Strip lowest bit, jump to parent)
- To UPDATE Point at i:         i += i & (-i) (Add lowest bit, propagate to ancestors)
```

Comparison: Segment Tree vs Fenwick Tree
- **Segment Tree**: More versatile (can handle arbitrary functions, range updates with lazy propagation), but requires $4N$ memory and complex pointer/recursion logic.
- **Fenwick Tree**: Shorter (15 lines of code), extremely cache-friendly (flat $N$ array), executes in single-cycle bit shifts with no recursion overhead.

---

## 2. Identification Signals ("When to Use")

- **Point Update with Prefix Sum**: Repeated queries of $\sum_{k=1}^i \text{nums}[k]$ alongside updates to individual elements.
- **Dynamic Frequency Counts**: Counting smaller elements seen so far (*Count of Smaller Numbers After Self*).
- **Inversion Counting**: Counting pairs $(i, j)$ where $i < j$ and $nums[i] > nums[j]$.

---

## 3. Algorithmic Template / Pseudocode

```text
class FenwickTree(size):
    tree = array of size (size + 1) filled with 0 // 1-INDEXED!

    function update(i, delta):
        while i <= size:
            tree[i] += delta
            i += i & (-i) // Move to next responsible ancestor

    function query(i):
        sum = 0
        while i > 0:
            sum += tree[i]
            i -= i & (-i) // Strip lowest set bit
        return sum

    function query_range(left, right):
        return query(right) - query(left - 1)
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import List

# --- Universal Fenwick Tree (Binary Indexed Tree) ---
class FenwickTree:
    def __init__(self, size: int):
        self.size = size
        # 1-indexed internal array
        self.tree = [0] * (size + 1)

    def update(self, i: int, delta: int) -> None:
        """Adds delta to index i (1-indexed)."""
        while i <= self.size:
            self.tree[i] += delta
            i += i & (-i)  # Add lowest set bit

    def query(self, i: int) -> int:
        """Returns prefix sum from 1 to i (inclusive)."""
        total = 0
        while i > 0:
            total += self.tree[i]
            i -= i & (-i)  # Strip lowest set bit
        return total

    def query_range(self, left: int, right: int) -> int:
        """Returns sum of elements between [left, right] (1-indexed)."""
        return self.query(right) - self.query(left - 1)

# --- Variant A: Range Sum Query - Mutable via BIT ---
class NumArrayBIT:
    def __init__(self, nums: List[int]):
        self.nums = list(nums)
        self.bit = FenwickTree(len(nums))
        for i, val in enumerate(nums):
            self.bit.update(i + 1, val)

    def update(self, index: int, val: int) -> None:
        delta = val - self.nums[index]
        self.nums[index] = val
        self.bit.update(index + 1, delta)

    def sum_range(self, left: int, right: int) -> int:
        return self.bit.query_range(left + 1, right + 1)

# --- Variant B: Count of Smaller Numbers After Self ---
def count_smaller(nums: List[int]) -> List[int]:
    # Coordinate compression: map arbitrary numbers to ranks [1, len(unique)]
    ranks = {val: rank + 1 for rank, val in enumerate(sorted(set(nums)))}
    bit = FenwickTree(len(ranks))
    res = []

    # Traverse from right to left
    for num in reversed(nums):
        rank = ranks[num]
        # Query count of elements with strictly smaller rank
        res.append(bit.query(rank - 1))
        # Insert current element into BIT frequency table
        bit.update(rank, 1)

    return res[::-1]
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `update`:
1. `while i <= self.size:`:
   * Traverses upwards along the ancestor chain in the implicit binary index hierarchy.
2. `self.tree[i] += delta`:
   * Increments the aggregate value stored in node `i`.
3. `i += i & (-i)`:
   * **The Upward Jump**: Adding the lowest set bit navigates to the parent node responsible for the larger containing interval in $O(\log N)$ steps.

### Breakdown of `query`:
1. `while i > 0:`:
   * Traverses downwards through disjoint sub-interval totals.
2. `total += self.tree[i]`:
   * Accumulates precomputed interval sum.
3. `i -= i & (-i)`:
   * **The Downward Jump**: Stripping the lowest set bit jumps over the current interval to the predecessor interval.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(\log N)$ Query & Update (Optimal)
- **Proof**:
  - Every non-zero integer $i \le N$ has at most $\lfloor \log_2 N \rfloor + 1$ set bits in binary.
  - `query` strips one set bit per step $\implies$ executes in $\le \log_2 N$ steps.
  - `update` adds the lowest bit, doubling the lowest power of 2 until exceeding $N \implies \le \log_2 N$ steps.
  - Both operations are strictly bounded by $O(\log N)$ single-cycle bitwise operations.

### Space Complexity: $O(N)$ (Optimal)
- The tree array requires exactly $N + 1$ integer slots.
- Unlike Segment Trees ($4N$ space), Fenwick Trees require only $1N$ space, matching the size of the original data.

---

## 7. Key Invariants & Common Pitfalls

1. **Strictly 1-Indexed**:
   * *Critical*: Fenwick Trees **cannot use index 0**! If $i = 0$, `i & (-i) = 0`, causing `i += 0` or `i -= 0`, resulting in an immediate infinite loop. Always offset indices to 1-based (`i + 1`).
2. **Delta Updates vs Value Replacements**:
   * Calling `update(i, val)` replaces the value with $+val$. When modifying an existing element, you must pass `delta = new_val - old_val`.

---

## 8. Canonical Problem Walkthrough

### Create Sorted Array through Instructions (LeetCode #1649 - Hard)
* **Problem**: Insert elements into array; cost is $\min(\text{count smaller}, \text{count greater}) \pmod{10^9 + 7}$.
* **Fenwick Tree Frequency Table**:
  1. `cost = min(bit.query(val - 1), bit.query(MAX) - bit.query(val))`.
  2. `bit.update(val, 1)`.
* **Optimality**: Operates in $O(N \log(\max V))$ time and $O(\max V)$ space.
