# Pattern 37: Two Heaps

> **Layer**: 3 (Advanced & Specialized)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/), [Sliding Window Median](https://leetcode.com/problems/sliding-window-median/), [IPO](https://leetcode.com/problems/ipo/)

---

## 1. Mental Model & Core Concept

The Two Heaps pattern maintains a dynamic stream partitioned into two balanced halves to enable **instantaneous $O(1)$ access to the median**:
- A **Max-Heap (`small`)** stores the smaller half of numbers.
- A **Min-Heap (`large`)** stores the larger half of numbers.

```text
Dual-Heap Balance Architecture:
Smaller Half: [ 1 , 2 , 3 ] (Max-Heap: peek is 3)
Larger Half:  [ 4 , 5 , 6 ] (Min-Heap: peek is 4)

Invariants:
1. Max(small) <= Min(large)
2. Size Balance: len(small) == len(large)  OR  len(small) == len(large) + 1

Median = small.peek()  (if odd)
       = (small.peek() + large.peek()) / 2.0  (if even)
```

---

## 2. Identification Signals ("When to Use")

- **Continuous Median Tracking**: Dynamically arriving numbers with constant median requests.
- **Percentile / Horizon Tracking**: Splitting elements into two halves where extrema of both halves interact (e.g. maximizing capital in *IPO*).

---

## 3. Algorithmic Template / Pseudocode

```text
class TwoHeaps:
    small = MaxHeap()
    large = MinHeap()

    function add(num):
        small.push(num)
        // Invariant 1: all small <= all large
        if small.max() > large.min():
            large.push(small.pop())
        // Invariant 2: size balance
        if small.size() > large.size() + 1:
            large.push(small.pop())
        elif large.size() > small.size():
            small.push(large.pop())

    function get_median():
        if small.size() > large.size(): return small.max()
        return (small.max() + large.min()) / 2.0
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
import heapq
from typing import List

# --- Variant A: Find Median from Data Stream ---
class MedianFinder:
    def __init__(self):
        self.small = []  # Max-heap (negated numbers)
        self.large = []  # Min-heap

    def add_num(self, num: int) -> None:
        # Step 1: Add to small (max-heap)
        heapq.heappush(self.small, -num)

        # Step 2: Ensure all small <= all large
        if self.small and self.large and (-self.small[0] > self.large[0]):
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)

        # Step 3: Rebalance sizes so len(small) is either equal or 1 larger
        if len(self.small) > len(self.large) + 1:
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)
        elif len(self.large) > len(self.small):
            val = heapq.heappop(self.large)
            heapq.heappush(self.small, -val)

    def find_median(self) -> float:
        if len(self.small) > len(self.large):
            return float(-self.small[0])
        return (-self.small[0] + self.large[0]) / 2.0

# --- Variant B: IPO (Maximize Capital with Two Heaps) ---
def find_maximized_capital(k: int, w: int, profits: List[int], capital: List[int]) -> int:
    # min_capital_heap stores (capital, profit) of projects we can't afford yet
    min_capital_heap = [(c, p) for c, p in zip(capital, profits)]
    heapq.heapify(min_capital_heap)

    # max_profit_heap stores profits of projects we CAN afford right now
    max_profit_heap = []

    for _ in range(k):
        # Move all projects that we can now afford into max_profit_heap
        while min_capital_heap and min_capital_heap[0][0] <= w:
            _, p = heapq.heappop(min_capital_heap)
            heapq.heappush(max_profit_heap, -p)

        if not max_profit_heap:
            break  # Cannot afford any more projects

        # Greedily pick the project with the highest profit
        w += -heapq.heappop(max_profit_heap)

    return w
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `add_num`:
1. `heapq.heappush(self.small, -num)`:
   * Pushes negated value into `small` to implement a max-heap in Python.
2. `if self.small and self.large and (-self.small[0] > self.large[0]):`:
   * Checks if the largest element in the lower half exceeds the smallest element in the upper half. If violated, transfers the element to `large`.
3. `if len(self.small) > len(self.large) + 1:`:
   * Enforces the size parity invariant. The lower half `small` is allowed to hold at most 1 extra element (which serves as the median for odd counts).

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(\log N)$ Insert, $O(1)$ Query (Optimal)
- **Proof**:
  - `add_num`: Performs a constant number of heap push and pop operations on trees of size $\le N/2 \implies O(\log N)$ time.
  - `find_median`: Peeks at the root of one or both heaps in $O(1)$ constant time.
  - Sorting on each insertion takes $O(N \log N)$ or $O(N)$ with insertion sort; Two Heaps cuts insertion to logarithmic $O(\log N)$.

### Space Complexity: $O(N)$ (Optimal)
- Both heaps combined store all $N$ stream numbers $\implies O(N)$ memory.

---

## 7. Key Invariants & Common Pitfalls

1. **Sign Reversal Maintenance**:
   * Negating integers when pushing to `self.small` and negating back when popping is essential. A common bug is accidentally pushing a positive integer into `small`.
2. **Rebalance Check Order**:
   * Always verify value boundary ordering (`max(small) <= min(large)`) **before** checking size balance.

---

## 8. Canonical Problem Walkthrough

### Sliding Window Median (LeetCode #480)
* **Problem**: Return median of every sliding window of size $k$.
* **Two Heaps + Lazy Deletion**: Use a hash map `delayed = {val: count}` to lazily delete numbers that have slid out of the window only when they appear at the top of the heaps!
* **Optimality**: Operates in $O(N \log K)$ time and $O(K)$ space.
