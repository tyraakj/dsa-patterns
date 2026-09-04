# Pattern 19: Heap / Priority Queue

> **Tier**: 2 (Intermediate High-Yield)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/), [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/), [Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/), [Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)

---

## 1. Mental Model & Core Concept

A Binary Heap is a complete binary tree that maintains the **Heap Invariant**: each parent node is smaller than or equal to its children (Min-Heap) or greater than or equal to its children (Max-Heap).

```text
Min-Heap Property:
            10
          /    \
        15      30
       /  \    /
      40  50  100

Root is ALWAYS the minimum!
- Get Minimum: O(1)
- Push / Pop:  O(log K)
```

Golden Rules for Top $K$:
- **To find the Top $K$ Largest elements**: Maintain a **Min-Heap of size $K$**. The root holds the $K$-th largest element; smaller elements are evicted.
- **To find the Top $K$ Smallest elements**: Maintain a **Max-Heap of size $K$**.
- In Python, `heapq` is a Min-Heap by default. To simulate a Max-Heap, invert values (`-val`).

---

## 2. Identification Signals ("When to Use")

- **Top $K$ Elements**: *"Find the K most frequent words / largest numbers"*.
- **Dynamic Extrema**: Continuous streams where elements arrive dynamically and you repeatedly query min/max.
- **Merging Multiple Ordered Sequences**: Merging $K$ sorted arrays/lists in $O(N \log K)$ time.
- **Cost Optimization**: Minimum cost to connect sticks, interval scheduling.

---

## 3. Algorithmic Template / Pseudocode

```text
function TOP_K_LARGEST(nums, k):
    min_heap = empty min-heap of capacity k

    for num in nums:
        heapq.push(min_heap, num)
        if min_heap.size() > k:
            heapq.pop(min_heap) // Evict smallest; leaves k largest!

    return min_heap.peek() // The k-th largest element
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
import heapq
from collections import Counter
from typing import List

# --- Variant A: Kth Largest Element via Min-Heap of Size K ---
def find_kth_largest(nums: List[int], k: int) -> int:
    min_heap = []
    for num in nums:
        heapq.heappush(min_heap, num)
        if len(min_heap) > k:
            heapq.heappop(min_heap)
    return min_heap[0]

# --- Variant B: Top K Frequent Elements ---
def top_k_frequent(nums: List[int], k: int) -> List[int]:
    count = Counter(nums)
    # Heap stores tuples: (frequency, number)
    min_heap = []

    for num, freq in count.items():
        heapq.heappush(min_heap, (freq, num))
        if len(min_heap) > k:
            heapq.heappop(min_heap)

    return [num for freq, num in min_heap]

# --- Variant C: Continuous Median Streaming (Dual Heaps) ---
class MedianFinder:
    def __init__(self):
        # small: max-heap (invert values) storing smaller half of numbers
        self.small = []
        # large: min-heap storing larger half of numbers
        self.large = []

    def add_num(self, num: int) -> None:
        # Default push to small (max-heap)
        heapq.heappush(self.small, -num)

        # Invariant 1: every element in small <= every element in large
        if self.small and self.large and (-self.small[0] > self.large[0]):
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)

        # Invariant 2: sizes must be balanced (|len(small) - len(large)| <= 1)
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
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `find_kth_largest`:
1. `min_heap = []`:
   * Allocates list backing the binary heap.
2. `heapq.heappush(min_heap, num)`:
   * Inserts element in $O(\log K)$ time, bubbling it up to preserve min-heap order.
3. `if len(min_heap) > k: heapq.heappop(min_heap)`:
   * **The Pruning Invariant**: When heap size reaches $K + 1$, the smallest element is evicted in $O(\log K)$. What remains inside the heap are strictly the $K$ largest elements seen so far!
4. `return min_heap[0]`:
   * The root of this size-$K$ min-heap is the smallest of the $K$ largest elements $\implies$ the $K$-th largest!

### Breakdown of `MedianFinder`:
1. `self.small = []; self.large = []`:
   * Divides data stream into two equal halves.
2. `heapq.heappush(self.small, -num)`:
   * Inverts sign to simulate Max-Heap in Python's native Min-Heap `heapq`.
3. `if -self.small[0] > self.large[0]: ...`:
   * Rebalances partitions so all elements in the lower half are $\le$ all elements in the upper half.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(N \log K)$ (Optimal)
- **Proof**:
  - Processing $N$ elements with a heap restricted to size $K$.
  - Each insertion and deletion takes at most $\log_2(K)$ operations.
  - Total time: $N \cdot \log K \implies O(N \log K)$.
  - **Comparison**: Full sorting takes $O(N \log N)$. When $K \ll N$ (e.g. top 10 out of 10,000,000 items), $O(N \log K)$ is essentially linear, outperforming full sorts.

### Space Complexity: $O(K)$ (Optimal)
- The heap size never exceeds $K + 1$ elements. Auxiliary memory is strictly bounded by $O(K)$.

---

## 7. Key Invariants & Common Pitfalls

1. **Python Max-Heap Sign Inversion**:
   * When popping from a max-heap where values were negated, remember to negate back: `val = -heapq.heappop(self.small)`.
2. **Tuple Comparison in Heaps**:
   * When storing `(priority, object)`, if priorities tie, Python compares the second tuple element. If the second element is not comparable (e.g. a `ListNode`), Python raises `TypeError: '<' not supported between instances`. Store a unique tie-breaking integer index: `(priority, index, object)`.

---

## 8. Canonical Problem Walkthrough

### K Closest Points to Origin (LeetCode #973)
* **Problem**: Find the $K$ closest points to $(0, 0)$ in 2D space.
* **Max-Heap of Size K**: Evict points with the largest Euclidean distance!

```python
def k_closest(points: List[List[int]], k: int) -> List[List[int]]:
    # Max-heap storing (-distance, x, y)
    max_heap = []
    for x, y in points:
        dist = x*x + y*y
        heapq.heappush(max_heap, (-dist, x, y))
        if len(max_heap) > k:
            heapq.heappop(max_heap)
            
    return [[x, y] for dist, x, y in max_heap]
```
* **Optimality**: Operates in $O(N \log K)$ time and $O(K)$ space.
