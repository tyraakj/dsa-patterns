# Pattern 03: Two Pointers

> **Layer**: 1 (Core Foundation)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Two Sum II - Input Array Is Sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/), [Container With Most Water](https://leetcode.com/problems/container-with-most-water/), [3Sum](https://leetcode.com/problems/3sum/), [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)

---

## 1. Mental Model & Core Concept

Two Pointers uses two index variables to traverse a sequence concurrently. Rather than checking all $O(N^2)$ pairs, it relies on monotonicity (sorted order) or structural bounds to prune an entire row or column of the search matrix with every single pointer step.

```text
Opposite Direction (Inward Converging):
left                                    right
  ▼                                       ▼
[ 1 , 3 , 5 , 8 , 11 , 15 , 18 , 22 , 30 ]
  ───►                                 ◄───
Condition Check:
  If sum < target: left += 1  (need a larger value)
  If sum > target: right -= 1 (need a smaller value)
  If sum == target: match found!
```

Primary configurations:
1. **Opposite Ends (Converging)**: Used for sorted pair sums, palindromes, container boundaries.
2. **Same Direction (Reader / Writer)**: Used for in-place array compaction (e.g. remove duplicates, move zeroes).
3. **Outward Expansion**: Used for finding longest palindromic substrings centered at index `i`.

---

## 2. Identification Signals ("When to Use")

- **Input is Sorted**: Pair search in sorted arrays without extra space.
- **Opposite Boundary Constraints**: Squeezing inward to optimize area, volume, or bounds (e.g. Container With Most Water).
- **In-Place Mutation with $O(1)$ Space**: Removing duplicates or moving zeros in-place.
- **Triplets / Quadruplets**: Fixing one element and running Two Pointers on the remaining subarray to solve 3Sum in $O(N^2)$ instead of $O(N^3)$.

---

## 3. Algorithmic Template / Pseudocode

```text
function CONVERGING_TWO_POINTERS(sorted_arr, target):
    left = 0
    right = length(sorted_arr) - 1

    while left < right:
        current_val = evaluate(sorted_arr[left], sorted_arr[right])
        
        if current_val == target:
            return [left, right]
        else if current_val < target:
            left += 1   // Monotonic increase
        else:
            right -= 1  // Monotonic decrease

    return []
```

```text
function FAST_SLOW_READER_WRITER(arr):
    slow = 0  // Writer index

    for fast from 0 to length(arr) - 1: // Reader index
        if arr[fast] satisfies keep_condition:
            arr[slow] = arr[fast]
            slow += 1

    return slow // New effective length
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import List

# --- Variant A: Two Sum II (Sorted Array) ---
def two_sum_sorted(numbers: List[int], target: int) -> List[int]:
    left, right = 0, len(numbers) - 1

    while left < right:
        curr_sum = numbers[left] + numbers[right]
        if curr_sum == target:
            return [left + 1, right + 1]  # 1-indexed return
        elif curr_sum < target:
            left += 1
        else:
            right -= 1

    return []

# --- Variant B: Container With Most Water (Greedy Boundary Shrink) ---
def max_area(height: List[int]) -> int:
    left, right = 0, len(height) - 1
    max_water = 0

    while left < right:
        width = right - left
        h = min(height[left], height[right])
        max_water = max(max_water, width * h)

        # Always advance the pointer pointing to the shorter wall
        if height[left] < height[right]:
            left += 1
        else:
            right -= 1

    return max_water

# --- Variant C: 3Sum with Duplicate Pruning ---
def three_sum(nums: List[int]) -> List[List[int]]:
    nums.sort()
    res = []
    n = len(nums)

    for i in range(n - 2):
        if i > 0 and nums[i] == nums[i - 1]:
            continue
        if nums[i] + nums[i + 1] + nums[i + 2] > 0:
            break

        left, right = i + 1, n - 1
        while left < right:
            total = nums[i] + nums[left] + nums[right]
            if total < 0:
                left += 1
            elif total > 0:
                right -= 1
            else:
                res.append([nums[i], nums[left], nums[right]])
                while left < right and nums[left] == nums[left + 1]:
                    left += 1
                while left < right and nums[right] == nums[right - 1]:
                    right -= 1
                left += 1
                right -= 1

    return res
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `two_sum_sorted`:
1. `left, right = 0, len(numbers) - 1`:
   * Places `left` at the minimum value and `right` at the maximum value of the sorted array.
2. `while left < right:`:
   * Continues inward until the pointers meet. Distinct indices required, so strictly `<`.
3. `curr_sum = numbers[left] + numbers[right]`:
   * Evaluates the pair sum of the current boundary candidates.
4. `if curr_sum == target: return [left + 1, right + 1]`:
   * Returns 1-based indices immediately upon finding the exact match.
5. `elif curr_sum < target: left += 1`:
   * **Monotonic Elimination**: Because the array is sorted, every pair formed by `left` with any index $< right$ will have sum $< numbers[left] + numbers[right] < target$. Therefore, `left` cannot pair with *any* remaining candidate to form `target`. We safely eliminate index `left` forever!
6. `else: right -= 1`:
   * Symmetrically, `numbers[right]` is too large to pair with any element $\ge left$. We eliminate `right` forever.

### Breakdown of `max_area` (Container With Most Water):
1. `left, right = 0, len(height) - 1`:
   * Starts with the widest possible container (width $= N - 1$).
2. `width = right - left; h = min(height[left], height[right]); max_water = max(max_water, width * h)`:
   * Calculates volume bounded by the shorter wall.
3. `if height[left] < height[right]: left += 1 else: right -= 1`:
   * **The Invariant**: If `height[left] < height[right]`, `height[left]` is the bottleneck. Moving `right` inward reduces width while the height bottleneck cannot exceed `height[left]`. Hence, any container pairing `left` with any interior `right'` will have strictly less area than the current container. We can safely discard `left`.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(N)$ (Optimal)
- **Proof of Upper Bound**:
  - `left` begins at $0$ and only increments.
  - `right` begins at $N - 1$ and only decrements.
  - In every single iteration of the while loop, `left` increases by $1$ or `right` decreases by $1$.
  - Therefore, the loop executes at most $N - 1$ times.
  - Each iteration involves $O(1)$ scalar arithmetic and comparison.
  - Total runtime $= O(N)$.
- **Comparison to Brute Force**: Brute force pair search checks all $\frac{N(N-1)}{2}$ pairs, taking $O(N^2)$ time. Two Pointers eliminates an entire row or column of pairs in $O(1)$ steps.

### Space Complexity: $O(1)$ (Optimal)
- Only uses two integer index variables (`left`, `right`) and a running accumulator.
- Zero auxiliary heap or stack allocations $\implies O(1)$ space.

---

## 7. Key Invariants & Common Pitfalls

1. **Loop Bound: `<` vs `<=`**:
   * For pair searching between two separate elements, using `left <= right` allows matching an element with itself, leading to invalid answers.
2. **Duplicate Skipping in 3Sum**:
   * *Trap*: Generating identical triplets.
   * *Rule*: Advance pointers *past* duplicate values using `while left < right and nums[left] == nums[left + 1]: left += 1` only *after* recording the valid triplet.

---

## 8. Canonical Problem Walkthrough

### Trapping Rain Water (LeetCode #42)
* **Problem**: Compute total rainwater trapped after raining on an elevation map.
* **Core Insight**: Water trapped at index $i$ is $\max(0, \min(\text{max\_left}, \text{max\_right}) - \text{height}[i])$.
* **Two Pointer Invariant**: If `max_left <= max_right`, water at `left` is strictly bounded by `max_left` regardless of any heights between `left` and `right`.

```python
def trap(height: List[int]) -> int:
    if not height:
        return 0
    left, right = 0, len(height) - 1
    max_l, max_r = height[left], height[right]
    water = 0
    
    while left < right:
        if max_l <= max_r:
            left += 1
            max_l = max(max_l, height[left])
            water += max_l - height[left]
        else:
            right -= 1
            max_r = max(max_r, height[right])
            water += max_r - height[right]
            
    return water
```
* **Optimality**: $O(N)$ time and $O(1)$ space, improving upon the $O(N)$ space required by 2-pass DP prefix/suffix arrays.
