# Pattern 23: Sorting

> **Layer**: 2 (Intermediate High-Yield)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Sort Colors (Dutch National Flag)](https://leetcode.com/problems/sort-colors/), [Largest Number](https://leetcode.com/problems/largest-number/), [Merge Intervals](https://leetcode.com/problems/merge-intervals/), [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/)

---

## 1. Mental Model & Core Concept

Sorting rearranges elements into a monotonic order. It serves as both a standalone algorithmic technique (e.g. Dutch National Flag 3-way partition) and an essential **preconditioning step** that enables Two Pointers, Binary Search, and Greedy algorithms.

```text
Dutch National Flag 3-Way Partition (Sort 0s, 1s, 2s in-place):
Pointers: low, mid, high
[ 0s region | 1s region | Unexplored | 2s region ]
  0 ... low   low ... mid  mid ... high  high ... n
```

Sorting Paradigms:
1. **Comparison Sort**: Merge Sort, Timsort (Python's `sort()`), Quick Sort $\implies$ $\Omega(N \log N)$ lower bound.
2. **Non-Comparison Sort**: Counting Sort, Bucket Sort, Radix Sort $\implies O(N + K)$ linear time when key ranges are bounded.
3. **Custom Comparator**: Solving non-transitive ordering problems like *Largest Number* ($A + B$ vs $B + A$).

---

## 2. Identification Signals ("When to Use")

- **In-Place Multi-Way Partitioning**: Sorting 3 distinct elements (e.g. 0, 1, 2) in $O(N)$ time with $O(1)$ space.
- **Custom Relative Ordering**: Arranging numbers to form the largest concatenation.
- **Preconditioning**: Grouping duplicates, enabling binary search, or sorting intervals by endpoints.

---

## 3. Algorithmic Template / Pseudocode

```text
function DUTCH_NATIONAL_FLAG(nums):
    low = 0
    mid = 0
    high = length(nums) - 1

    while mid <= high:
        if nums[mid] == 0:
            swap(nums[low], nums[mid])
            low += 1
            mid += 1
        else if nums[mid] == 1:
            mid += 1
        else: // nums[mid] == 2
            swap(nums[mid], nums[high])
            high -= 1
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
import functools
from typing import List

# --- Variant A: Dutch National Flag In-Place 3-Way Partition (Sort Colors) ---
def sort_colors(nums: List[int]) -> None:
    low = 0
    mid = 0
    high = len(nums) - 1

    while mid <= high:
        if nums[mid] == 0:
            nums[low], nums[mid] = nums[mid], nums[low]
            low += 1
            mid += 1
        elif nums[mid] == 1:
            mid += 1
        else:
            nums[mid], nums[high] = nums[high], nums[mid]
            high -= 1  # Note: do NOT increment mid! Value swapped from high is unexplored!

# --- Variant B: Custom Comparator (Largest Number) ---
def largest_number(nums: List[int]) -> str:
    # Custom comparator: compare concatenation order (a+b vs b+a)
    def compare(x: str, y: str) -> int:
        if x + y > y + x:
            return -1  # x should come before y
        elif x + y < y + x:
            return 1   # y should come before x
        else:
            return 0

    str_nums = [str(x) for x in nums]
    str_nums.sort(key=functools.cmp_to_key(compare))

    # Guard against leading zeros (e.g. [0, 0] -> "0")
    return "0" if str_nums[0] == "0" else "".join(str_nums)

# --- Variant C: Bucket Sort (Top K Frequent in O(N)) ---
def top_k_frequent_bucket(nums: List[int], k: int) -> List[int]:
    from collections import Counter
    count = Counter(nums)
    n = len(nums)

    # Buckets where index = frequency, value = list of elements
    buckets: List[List[int]] = [[] for _ in range(n + 1)]
    for num, freq in count.items():
        buckets[freq].append(num)

    res = []
    # Traverse buckets in descending order of frequency
    for freq in range(n, 0, -1):
        for num in buckets[freq]:
            res.append(num)
            if len(res) == k:
                return res

    return res
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `sort_colors` (Dutch National Flag):
1. `low = 0; mid = 0; high = len(nums) - 1`:
   * Invariant: `nums[0...low-1]` are all `0`, `nums[low...mid-1]` are all `1`, and `nums[high+1...N-1]` are all `2`.
2. `if nums[mid] == 0: nums[low], nums[mid] = nums[mid], nums[low]; low += 1; mid += 1`:
   * Swaps `0` into the lower boundary. Because everything before `mid` has already been explored, the element swapped into `mid` from `low` is guaranteed to be a `1`, so both `low` and `mid` advance.
3. `elif nums[mid] == 1: mid += 1`:
   * `1` is already in its correct middle region; simply advance `mid`.
4. `else: nums[mid], nums[high] = nums[high], nums[mid]; high -= 1`:
   * **Crucial Detail**: We swap `2` into the high partition, but we **do not increment `mid`**! The element swapped from `high` into `mid` is completely uninspected (it could be a `0`, `1`, or `2`) and must be evaluated on the next iteration.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity:
- **`sort_colors`**: In each while loop iteration, either `mid` increments or `high` decrements. Total iterations $\le N$. Operations per step $= O(1)$. Total runtime is strictly $O(N)$.
- **`top_k_frequent_bucket`**: Tallying takes $O(N)$, placing in buckets takes $O(U)$ where $U \le N$, and scanning buckets takes $O(N)$. Total time is $O(N)$, beating comparison sort's $\Omega(N \log N)$ lower bound.

### Space Complexity:
- **`sort_colors`**: Swaps in-place using 3 pointer variables $\implies O(1)$ space.
- **`top_k_frequent_bucket`**: Allocates $N + 1$ bucket slots $\implies O(N)$ space.

---

## 7. Key Invariants & Common Pitfalls

1. **Mid Increment on High Swap**:
   * Writing `mid += 1` when swapping with `high` is a classic bug that causes elements swapped from the right to skip evaluation.
2. **String Comparator Concatenation**:
   * Comparing strings lexicographically (`x > y`) is wrong for *Largest Number* (e.g. `"30"` vs `"3"`: `"30"` > `"3"`, but `"330"` > `"303"`). Always compare concatenated candidates: `x + y > y + x`.

---

## 8. Canonical Problem Walkthrough

### H-Index (LeetCode #274)
* **Problem**: Find the maximum $h$ such that the researcher has at least $h$ papers with $\ge h$ citations.
* **Bucket Sort in $O(N)$**:
```python
def h_index(citations: List[int]) -> int:
    n = len(citations)
    # Buckets: count papers with k citations; clamp > n to bucket n
    buckets = [0] * (n + 1)
    for c in citations:
        buckets[min(c, n)] += 1
        
    accumulated = 0
    for h in range(n, -1, -1):
        accumulated += buckets[h]
        if accumulated >= h:
            return h
    return 0
```
* **Optimality**: Runs in $O(N)$ time and $O(N)$ space, outperforming standard $O(N \log N)$ sorting.
