# Pattern 04: Binary Search

> **Tier**: 1 (Core Foundation)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Binary Search](https://leetcode.com/problems/binary-search/), [Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/), [Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/), [Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/)

---

## 1. Mental Model & Core Concept

Binary Search eliminates half of the search space in each step by testing a monotonic property at the midpoint. It works not only on sorted arrays, but on **any function $f(x)$ whose boolean evaluation is monotonic** (e.g. $[F, F, F, T, T, T]$).

```text
Monotonic Feasibility Function: check(x)
x:         1    2    3    4    5    6    7    8    9
check(x): [F,   F,   F,   F,   T,   T,   T,   T,   T]
                               ▲
            Boundary: Find the first True value!
```

Two major applications:
1. **Discrete Index Search**: Locating a target element in a sorted or rotated array.
2. **Binary Search on Answer (Optimization)**: Searching for the minimum speed, maximum capacity, or boundary value within a continuous range `[low, high]`.

---

## 2. Identification Signals ("When to Use")

- **Explicit Logarithmic Constraint**: *"Solve in $O(\log N)$ runtime"*.
- **Sorted or Rotated Array**: Direct index lookups in ordered sequences.
- **Min-Max / Max-Min Optimization**: *"Find the minimum speed to finish in $H$ hours"*, *"Find the maximum minimum distance between cows"*, *"Split array largest sum"*.
- **Monotonic Predicate**: If condition is true for $X$, it is guaranteed to be true for all values $> X$ (or $< X$).

---

## 3. Algorithmic Template / Pseudocode

### Template A: Standard Exact Match
```text
function BINARY_SEARCH_EXACT(nums, target):
    low = 0
    high = length(nums) - 1

    while low <= high:
        mid = low + (high - low) // 2
        if nums[mid] == target:
            return mid
        else if nums[mid] < target:
            low = mid + 1
        else:
            high = mid - 1

    return -1
```

### Template B: Binary Search on Answer (First True)
```text
function BINARY_SEARCH_FIRST_TRUE(low, high, condition):
    ans = -1
    while low <= high:
        mid = low + (high - low) // 2
        if condition(mid) is TRUE:
            ans = mid         // Record candidate
            high = mid - 1    // Try to find an even smaller valid answer
        else:
            low = mid + 1     // Condition not met, increase value

    return ans
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
import math
from typing import List, Callable

# --- Variant 1: Search in Rotated Sorted Array ---
def search_rotated(nums: List[int], target: int) -> int:
    left, right = 0, len(nums) - 1

    while left <= right:
        mid = left + (right - left) // 2
        if nums[mid] == target:
            return mid

        # Check if left half [left...mid] is sorted
        if nums[left] <= nums[mid]:
            if nums[left] <= target < nums[mid]:
                right = mid - 1  # Target is in sorted left half
            else:
                left = mid + 1   # Target is in right half
        # Otherwise, right half [mid...right] must be sorted
        else:
            if nums[mid] < target <= nums[right]:
                left = mid + 1   # Target is in sorted right half
            else:
                right = mid - 1  # Target is in left half

    return -1

# --- Variant 2: Binary Search on Answer (Koko Eating Bananas) ---
def min_eating_speed(piles: List[int], h: int) -> int:
    def can_finish(speed: int) -> bool:
        hours = sum(math.ceil(p / speed) for p in piles)
        return hours <= h

    left, right = 1, max(piles)
    res = right

    while left <= right:
        mid = left + (right - left) // 2
        if can_finish(mid):
            res = mid
            right = mid - 1  # Feasible, try to find smaller speed
        else:
            left = mid + 1   # Infeasible, need faster speed

    return res

# --- Variant 3: Find Minimum in Rotated Sorted Array ---
def find_min(nums: List[int]) -> int:
    left, right = 0, len(nums) - 1
    while left < right:
        mid = left + (right - left) // 2
        if nums[mid] > nums[right]:
            left = mid + 1
        else:
            right = mid
    return nums[left]
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `search_rotated`:
1. `left, right = 0, len(nums) - 1`:
   * Establishes initial search interval covering the entire array.
2. `while left <= right:`:
   * Continues as long as the search interval contains at least one candidate.
3. `mid = left + (right - left) // 2`:
   * Midpoint computed defensively to prevent integer overflow.
4. `if nums[mid] == target: return mid`:
   * Immediate match check.
5. `if nums[left] <= nums[mid]:`:
   * **Rotated Invariant**: At least one half of the rotated array is **always** normally sorted. If `nums[left] <= nums[mid]`, the left half `[left, mid]` is strictly monotonic.
6. `if nums[left] <= target < nums[mid]: right = mid - 1 else: left = mid + 1`:
   * Because the left half is sorted, we can decisively test whether `target` falls within `[nums[left], nums[mid])`. If yes, discard right half (`right = mid - 1`); otherwise discard left half (`left = mid + 1`).
7. `else: if nums[mid] < target <= nums[right]: left = mid + 1 else: right = mid - 1`:
   * Symmetrically, if the left half is not sorted, the right half `[mid, right]` is guaranteed to be sorted.

### Breakdown of `min_eating_speed`:
1. `def can_finish(speed: int) -> bool:`:
   * Evaluates if Koko can eat all bananas within $h$ hours at the given integer `speed`.
2. `left, right = 1, max(piles)`:
   * `left = 1` is the absolute slowest speed possible (1 banana/hr).
   * `right = max(piles)` is the guaranteed upper bound (eating the largest pile in 1 hour).
3. `if can_finish(mid): res = mid; right = mid - 1`:
   * If speed `mid` is feasible, record `res = mid` and shrink `right` to check if a lower valid integer speed exists.
4. `else: left = mid + 1`:
   * If `mid` is too slow, eliminate all speeds $\le mid$.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(\log N)$ or $O(N \log M)$ (Optimal)
- **Search Space Reduction**: Let $S_0 = N$ be the initial size of the search space. At each step:
  $$S_{k} = \frac{S_{k-1}}{2} = \frac{N}{2^k}$$
  The algorithm terminates when $S_k \le 1$, meaning:
  $$\frac{N}{2^k} \le 1 \implies 2^k \ge N \implies k = \lceil \log_2 N \rceil$$
- **Comparison-Based Lower Bound**: By information theory, identifying 1 target out of $N$ elements requires $\log_2(N)$ bits of information. A comparison provides at most 1 bit of information (left or right). Therefore, $\Omega(\log N)$ is the theoretical minimum bound for any comparison-based search.
- For `min_eating_speed`: Testing `can_finish` takes $O(N)$ time. Range of speeds is $M = \max(piles)$. Total runtime is $O(N \log M)$, which is optimal since every pile must be inspected to evaluate feasibility.

### Space Complexity: $O(1)$ (Optimal)
- State is tracked exclusively via pointer indices (`left`, `right`, `mid`).
- Iterative binary search performs zero recursion and zero dynamic memory allocations, achieving $O(1)$ space.

---

## 7. Key Invariants & Common Pitfalls

1. **Integer Overflow Prevention**:
   * Always write `mid = left + (right - left) // 2` rather than `(left + right) // 2`.
2. **Infinite Loops (`left < right` vs `left <= right`)**:
   * If using `left = mid`, integer division rounds down, causing an infinite loop when `right - left == 1`.
   * Standard convention: use `while left <= right` with `left = mid + 1` and `right = mid - 1`.
3. **Boundaries for Monotonic Search**:
   * Set `left` to the smallest mathematically legal value (e.g. `1`, not `0` to avoid division by zero) and `right` to the guaranteed feasible maximum.

---

## 8. Canonical Problem Walkthrough

### Find Minimum in Rotated Sorted Array (LeetCode #153)
* **Problem**: Find the minimum element in a rotated sorted array in $O(\log N)$.
* **Key Invariant**: Compare `nums[mid]` with `nums[right]`:
  - If `nums[mid] > nums[right]`: Inflection point must be to the right of `mid` $\implies left = mid + 1$.
  - If `nums[mid] <= nums[right]`: Minimum is at `mid` or to the left of `mid` $\implies right = mid$.

```python
def find_min(nums: List[int]) -> int:
    left, right = 0, len(nums) - 1
    while left < right:
        mid = left + (right - left) // 2
        if nums[mid] > nums[right]:
            left = mid + 1
        else:
            right = mid
    return nums[left]
```
* **Optimality**: Operates in $O(\log N)$ time and $O(1)$ space, compared to an $O(N)$ linear scan.
