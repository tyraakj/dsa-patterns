# Pattern 38: K-way Merge

> **Layer**: 3 (Advanced & Specialized)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/), [Find K Pairs with Smallest Sums](https://leetcode.com/problems/find-k-pairs-with-smallest-sums/), [Kth Smallest Element in a Sorted Matrix](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/), [Smallest Range Covering Elements from K Lists](https://leetcode.com/problems/smallest-range-covering-elements-from-k-lists/)

---

## 1. Mental Model & Core Concept

When merging $K$ sorted streams or searching for the $K$-th smallest combination across sorted arrays, maintaining a **Min-Heap of size $K$** allows finding the next global minimum in $O(\log K)$ time.

```text
K-Way Merge with Min-Heap:
Stream 1: [ 1 , 4 , 5 ]
Stream 2: [ 1 , 3 , 4 ]
Stream 3: [ 2 , 6 ]

Min-Heap stores (val, list_index, element_index):
Initial Heap: [ (1, Stream1), (1, Stream2), (2, Stream3) ]

1. Pop minimum (1 from Stream 1)
2. Push next element from Stream 1 (4) into Heap!
Heap maintains exactly K active elements at all times!
```

---

## 2. Identification Signals ("When to Use")

- **$K$ Sorted Sequences**: Merging sorted linked lists, sorted row matrices.
- **Pairs with Smallest Sums**: Finding $K$ smallest pairs $(u, v)$ from two sorted arrays.
- **Smallest Bounding Range**: Finding the shortest interval covering at least one element from each of $K$ lists.

---

## 3. Algorithmic Template / Pseudocode

```text
function K_WAY_MERGE(lists):
    min_heap = empty heap
    
    // Seed heap with head of each non-empty list
    for i from 0 to k - 1:
        if lists[i] is not empty:
            min_heap.push((lists[i].val, i, lists[i].head))
            
    dummy = ListNode(0)
    curr = dummy
    
    while min_heap is not empty:
        val, i, node = min_heap.pop()
        curr.next = node
        curr = curr.next
        
        if node.next is not null:
            min_heap.push((node.next.val, i, node.next))
            
    return dummy.next
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
import heapq
from typing import List, Optional

class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

# --- Variant A: Merge K Sorted Lists (Optimal Heap) ---
def merge_k_lists(lists: List[Optional[ListNode]]) -> Optional[ListNode]:
    # Heap stores: (val, unique_id, node)
    min_heap = []

    for i, node in enumerate(lists):
        if node:
            # i serves as unique tie-breaker to avoid comparing ListNode instances directly
            heapq.heappush(min_heap, (node.val, i, node))

    dummy = ListNode(0)
    curr = dummy

    while min_heap:
        val, i, node = heapq.heappop(min_heap)
        curr.next = node
        curr = curr.next

        if node.next:
            heapq.heappush(min_heap, (node.next.val, i, node.next))

    return dummy.next

# --- Variant B: Kth Smallest Element in a Sorted Matrix ---
def kth_smallest_matrix(matrix: List[List[int]], k: int) -> int:
    n = len(matrix)
    # Min-heap stores: (val, row, col)
    min_heap = []

    # Seed heap with first element of each row
    for r in range(min(n, k)):
        heapq.heappush(min_heap, (matrix[r][0], r, 0))

    val = 0
    for _ in range(k):
        val, r, c = heapq.heappop(min_heap)
        if c + 1 < n:
            heapq.heappush(min_heap, (matrix[r][c + 1], r, c + 1))

    return val

# --- Variant C: Find K Pairs with Smallest Sums ---
def k_smallest_pairs(nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
    if not nums1 or not nums2:
        return []

    res = []
    # Heap stores: (sum, i, j)
    min_heap = []

    # Seed heap with first column of pairs
    for i in range(min(len(nums1), k)):
        heapq.heappush(min_heap, (nums1[i] + nums2[0], i, 0))

    while min_heap and len(res) < k:
        curr_sum, i, j = heapq.heappop(min_heap)
        res.append([nums1[i], nums2[j]])

        if j + 1 < len(nums2):
            heapq.heappush(min_heap, (nums1[i] + nums2[j + 1], i, j + 1))

    return res
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `merge_k_lists`:
1. `heapq.heappush(min_heap, (node.val, i, node))`:
   * Seeds heap with the head of all $K$ lists.
   * **The Tie-Breaker Invariant**: `i` guarantees that if two nodes share identical values (`node.val == node2.val`), Python compares integer `i` instead of attempting `node < node2`, which would raise a `TypeError`.
2. `val, i, node = heapq.heappop(min_heap)`:
   * Extracts the global minimum across all active stream heads in $O(\log K)$ time.
3. `if node.next: heapq.heappush(min_heap, (node.next.val, i, node.next))`:
   * Refills the heap with the next candidate from the *same* list that produced the popped minimum.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(N \log K)$ (Optimal)
- **Proof**:
  - Let $N$ be the total number of elements across all $K$ lists.
  - The heap holds at most $K$ elements at any moment.
  - Each of the $N$ nodes is pushed and popped from the heap exactly once.
  - Each heap operation takes $O(\log K)$.
  - Total time: $N \cdot \log K \implies O(N \log K)$.
  - Comparison: Merging lists sequentially takes $O(N \cdot K)$ time. K-Way Merge reduces this to $O(N \log K)$.

### Space Complexity: $O(K)$ (Optimal)
- Heap stores at most 1 active node per list $\implies O(K)$ auxiliary memory.

---

## 7. Key Invariants & Common Pitfalls

1. **Python Object Comparison Crash**:
   * *Trap*: Pushing `(node.val, node)` raises `TypeError: '<' not supported between instances of 'ListNode'` whenever values tie. Always include index: `(node.val, i, node)`.
2. **K Pairs Initialization**:
   * For `k_smallest_pairs`, you only need to push at most `k` initial pairs (`min(len(nums1), k)`), ensuring the heap never exceeds size $K$.

---

## 8. Canonical Problem Walkthrough

### Smallest Range Covering Elements from K Lists (LeetCode #632 - Hard)
* **Problem**: Find the smallest interval $[a, b]$ that includes at least one number from each of the $k$ lists.
* **Heap + Sliding Range Tracker**:
  1. Store `(val, list_idx, elem_idx)` in min-heap; maintain running `current_max`.
  2. Range candidate is `[min_heap[0].val, current_max]`.
  3. Pop minimum and advance its list to shrink range!

```python
def smallest_range(nums: List[List[int]]) -> List[int]:
    min_heap = []
    curr_max = float('-inf')
    
    for i in range(len(nums)):
        heapq.heappush(min_heap, (nums[i][0], i, 0))
        curr_max = max(curr_max, nums[i][0])
        
    best_range = [min_heap[0][0], curr_max]
    
    while True:
        min_val, list_idx, elem_idx = heapq.heappop(min_heap)
        
        if curr_max - min_val < best_range[1] - best_range[0]:
            best_range = [min_val, curr_max]
            
        if elem_idx + 1 == len(nums[list_idx]):
            break  # One list exhausted, cannot cover all k lists anymore
            
        next_val = nums[list_idx][elem_idx + 1]
        heapq.heappush(min_heap, (next_val, list_idx, elem_idx + 1))
        curr_max = max(curr_max, next_val)
        
    return best_range
```
* **Optimality**: Operates in $O(N \log K)$ time and $O(K)$ space.
