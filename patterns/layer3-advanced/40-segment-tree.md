# Pattern 40: Segment Tree

> **Layer**: 3 (Advanced & Specialized)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Range Sum Query - Mutable](https://leetcode.com/problems/range-sum-query-mutable/), [My Calendar III](https://leetcode.com/problems/my-calendar-iii/), [Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/)

---

## 1. Mental Model & Core Concept

A Segment Tree is a binary tree where each node stores an aggregate property (sum, min, max, gcd) over a continuous interval $[L, R]$.

```text
Segment Tree Hierarchy over Array of size 4:
                     [0, 3] (Sum = 16)
                    /      \
             [0, 1] (Sum = 7)  [2, 3] (Sum = 9)
             /    \            /    \
         [0, 0]  [1, 1]    [2, 2]  [3, 3]
          (3)     (4)       (2)     (7)

Array:   [ 3   ,   4   ,     2   ,   7   ]
```

Why Segment Trees Outperform Standard Arrays:
- Standard Array: $O(1)$ Point Update, but $O(N)$ Range Query.
- Prefix Sum: $O(1)$ Range Query, but $O(N)$ Point Update.
- **Segment Tree: $O(\log N)$ Point Update AND $O(\log N)$ Range Query!**

Array Storage Representation:
A segment tree over $N$ elements is conventionally stored in a single 1D array of size $4N$:
- Left child of index $i$: $2i + 1$
- Right child of index $i$: $2i + 2$

---

## 2. Identification Signals ("When to Use")

- **Dynamic Range Queries with Updates**: Repeated queries for range sums, range minimums, or range maximums interspersed with point or range value updates.
- **Range Booking / Overlap Counting**: Detecting maximum simultaneous overlapping intervals dynamically.
- **Inversion Counting**: Counting elements to the right that are smaller than self.

---

## 3. Algorithmic Template / Pseudocode

```text
function BUILD(node, start, end):
    if start == end:
        tree[node] = nums[start]
        return
    mid = (start + end) // 2
    BUILD(2*node + 1, start, mid)
    BUILD(2*node + 2, mid + 1, end)
    tree[node] = tree[2*node + 1] + tree[2*node + 2]

function UPDATE(node, start, end, idx, val):
    if start == end:
        tree[node] = val
        return
    mid = (start + end) // 2
    if idx <= mid:
        UPDATE(2*node + 1, start, mid, idx, val)
    else:
        UPDATE(2*node + 2, mid + 1, end, idx, val)
    tree[node] = tree[2*node + 1] + tree[2*node + 2]

function QUERY(node, start, end, ql, qr):
    if ql <= start and end <= qr:
        return tree[node] // Total overlap
    if end < ql or start > qr:
        return 0          // No overlap
    mid = (start + end) // 2
    return QUERY(2*node + 1, start, mid, ql, qr) + QUERY(2*node + 2, mid + 1, end, ql, qr)
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import List

# --- Universal Array-Backed Segment Tree (Range Sum Query - Mutable) ---
class NumArray:
    def __init__(self, nums: List[int]):
        self.n = len(nums)
        self.nums = list(nums)
        # Allocate 4 * N array for the tree representation
        self.tree = [0] * (4 * self.n)
        if self.n > 0:
            self._build(0, 0, self.n - 1)

    def _build(self, node: int, start: int, end: int) -> None:
        if start == end:
            self.tree[node] = self.nums[start]
            return
        mid = start + (end - start) // 2
        left_child = 2 * node + 1
        right_child = 2 * node + 2

        self._build(left_child, start, mid)
        self._build(right_child, mid + 1, end)
        self.tree[node] = self.tree[left_child] + self.tree[right_child]

    def update(self, index: int, val: int) -> None:
        self._update(0, 0, self.n - 1, index, val)

    def _update(self, node: int, start: int, end: int, idx: int, val: int) -> None:
        if start == end:
            self.nums[idx] = val
            self.tree[node] = val
            return

        mid = start + (end - start) // 2
        left_child = 2 * node + 1
        right_child = 2 * node + 2

        if idx <= mid:
            self._update(left_child, start, mid, idx, val)
        else:
            self._update(right_child, mid + 1, end, idx, val)

        self.tree[node] = self.tree[left_child] + self.tree[right_child]

    def sum_range(self, left: int, right: int) -> int:
        return self._query(0, 0, self.n - 1, left, right)

    def _query(self, node: int, start: int, end: int, ql: int, qr: int) -> int:
        # Case 1: Complete overlap
        if ql <= start and end <= qr:
            return self.tree[node]

        # Case 2: Disjoint (no overlap)
        if end < ql or start > qr:
            return 0

        # Case 3: Partial overlap
        mid = start + (end - start) // 2
        left_sum = self._query(2 * node + 1, start, mid, ql, qr)
        right_sum = self._query(2 * node + 2, mid + 1, end, ql, qr)

        return left_sum + right_sum
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `_build`:
1. `if start == end: self.tree[node] = self.nums[start]; return`:
   * Base case: leaf node represents a single array element.
2. `mid = start + (end - start) // 2`:
   * Divides interval symmetrically into two halves.
3. `self.tree[node] = self.tree[left_child] + self.tree[right_child]`:
   * Bottom-up aggregation: parent stores the sum of both child subtrees.

### Breakdown of `_query`:
1. `if ql <= start and end <= qr: return self.tree[node]`:
   * If the current node's interval is fully enclosed by the query window, return the precomputed sum in $O(1)$.
2. `if end < ql or start > qr: return 0`:
   * If current segment is completely outside the query bounds, return identity 0.
3. `return left_sum + right_sum`:
   * If partially overlapping, splits into sub-queries and aggregates child responses.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(\log N)$ Query, $O(\log N)$ Update (Optimal)
- **Proof of Query Bound**:
  - At each level of the tree, at most 4 nodes are visited (at most 2 nodes are partially covered, while fully covered nodes return immediately and disjoint nodes prune).
  - The height of the segment tree is $\lceil \log_2 N \rceil$.
  - Therefore, at most $4 \lceil \log_2 N \rceil$ nodes are visited during any range query $\implies O(\log N)$.
- **Proof of Update Bound**: Descends a single path from root to leaf $\implies$ exactly $\lceil \log_2 N \rceil$ nodes visited $\implies O(\log N)$.
- **Build Time**: Sum of nodes in full binary tree $= 2N - 1 \implies O(N)$ linear build time.

### Space Complexity: $O(N)$ (Optimal)
- An array of size $4N$ is sufficient to store all internal and leaf nodes in a 1-based or 0-based heap-style segment tree $\implies O(N)$ memory.

---

## 7. Key Invariants & Common Pitfalls

1. **Why Size $4N$ is Required**:
   * If $N$ is not a power of 2, rounding up to the next power of 2 can double the size, and leaf child pointers extend one level deeper. $4N$ mathematically guarantees no `IndexError` occurs.
2. **Lazy Propagation**:
   * For range updates (updating all elements in $[L, R]$ with $+val$), naive segment trees take $O(N)$. Lazy propagation defers updates to child nodes until queried, restoring range updates to $O(\log N)$.

---

## 8. Canonical Problem Walkthrough

### My Calendar III (LeetCode #732)
* **Problem**: Find the maximum $k$-booking (maximum number of concurrent overlapping events).
* **Dynamic Segment Tree / Coordinate Compression**: Maintain max overlap count with lazy propagation over continuous timestamps.
* **Optimality**: Operates in $O(N \log C)$ time where $C$ is the coordinate span.
