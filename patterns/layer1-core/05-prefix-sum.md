# Pattern 05: Prefix Sum

> **Layer**: 1 (Core Foundation)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Range Sum Query - Immutable](https://leetcode.com/problems/range-sum-query-immutable/), [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/), [Contiguous Array](https://leetcode.com/problems/contiguous-array/), [Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/), [Continuous Subarray Sum](https://leetcode.com/problems/continuous-subarray-sum/)

---

## 1. Mental Model & Core Concept

A Prefix Sum precomputes cumulative running totals so that any continuous range sum query over interval `[i, j]` can be evaluated in $O(1)$ constant time rather than scanning all elements in $O(N)$.

```text
Original:    [  3 ,  1 ,  4 ,  1 ,  5  ]
Prefix:   [ 0,  3 ,  4 ,  8 ,  9 , 14  ]
Index:      0   1    2    3    4    5

Sum between index 1 and 3 (elements 1, 4, 1 = 6):
Formula: Prefix[4] - Prefix[1] = 9 - 3 = 6
```

Crucial algebraic transformation:
$$\text{Sum}(i \dots j) = \text{Prefix}[j + 1] - \text{Prefix}[i]$$
If we require $\text{Sum}(i \dots j) = K$, then:
$$\text{Prefix}[i] = \text{Prefix}[j + 1] - K$$
This connects Prefix Sum directly to Hash Maps: query how many times the value $(\text{curr\_sum} - K)$ occurred in previous prefixes.

---

## 2. Identification Signals ("When to Use")

- **Repeated Range Queries**: Multiple queries asking for sums in sub-intervals $[L, R]$.
- **Subarrays with Sum Constraints (with Negatives)**: Problems where negative numbers prevent sliding windows from maintaining monotonicity.
- **Equal Number of 0s and 1s**: Transform `0` into `-1`, and find subarrays whose sum equals `0`.
- **Modulo Properties**: Subarray sums that are multiples of $K$ ($\text{prefix}_j \equiv \text{prefix}_i \pmod K$).
- **Prefix and Suffix Products**: Multiplying all elements except self in $O(N)$ time without division.

---

## 3. Algorithmic Template / Pseudocode

```text
function BUILD_PREFIX_SUM(nums):
    n = length(nums)
    prefix = array of size (n + 1) filled with 0
    
    for i from 0 to n - 1:
        prefix[i + 1] = prefix[i] + nums[i]
        
    return prefix

function QUERY_RANGE(prefix, left, right):
    return prefix[right + 1] - prefix[left]
```

```text
function SUBARRAY_SUM_HASH_MAP(nums, k):
    count = 0
    curr_sum = 0
    seen = hash map initialized with {0: 1} // prefix sum -> frequency
    
    for num in nums:
        curr_sum += num
        if (curr_sum - k) in seen:
            count += seen[curr_sum - k]
        seen[curr_sum] += 1
        
    return count
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from collections import defaultdict
from typing import List

# --- Variant A: Immutable Range Sum Query (1D) ---
class NumArray:
    def __init__(self, nums: List[int]):
        self.prefix = [0] * (len(nums) + 1)
        for i, val in enumerate(nums):
            self.prefix[i + 1] = self.prefix[i] + val

    def sum_range(self, left: int, right: int) -> int:
        return self.prefix[right + 1] - self.prefix[left]

# --- Variant B: Contiguous Array (Equal 0s and 1s) ---
def find_max_length(nums: List[int]) -> int:
    prefix_indices = {0: -1}  # running_sum -> earliest index
    curr_sum = 0
    max_len = 0

    for i, num in enumerate(nums):
        # Treat 0 as -1, 1 as +1
        curr_sum += 1 if num == 1 else -1

        if curr_sum in prefix_indices:
            max_len = max(max_len, i - prefix_indices[curr_sum])
        else:
            # Only record first occurrence to maximize subarray length
            prefix_indices[curr_sum] = i

    return max_len

# --- Variant C: Product of Array Except Self ---
def product_except_self(nums: List[int]) -> List[int]:
    n = len(nums)
    res = [1] * n

    # Forward prefix products
    prefix = 1
    for i in range(n):
        res[i] = prefix
        prefix *= nums[i]

    # Backward suffix products
    suffix = 1
    for i in range(n - 1, -1, -1):
        res[i] *= suffix
        suffix *= nums[i]

    return res

# --- Variant D: Continuous Subarray Sum (Modulo K) ---
def check_subarray_sum(nums: List[int], k: int) -> bool:
    remainder_map = {0: -1}  # remainder -> earliest index
    curr_sum = 0
    
    for i, num in enumerate(nums):
        curr_sum += num
        rem = curr_sum % k
        
        if rem in remainder_map:
            if i - remainder_map[rem] >= 2:
                return True
        else:
            remainder_map[rem] = i
            
    return False
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `NumArray` (Prefix Range Query):
1. `self.prefix = [0] * (len(nums) + 1)`:
   * Allocates an array of size $N + 1$ with index `0` initialized to `0`.
2. `for i, val in enumerate(nums): self.prefix[i + 1] = self.prefix[i] + val`:
   * Each entry `self.prefix[i + 1]` stores the cumulative sum of `nums[0...i]`.
3. `return self.prefix[right + 1] - self.prefix[left]`:
   * Evaluates the range sum `nums[left...right]` in exactly 1 subtraction ($O(1)$ time).
   * Notice that when `left = 0`, `self.prefix[left] == self.prefix[0] == 0`, returning `self.prefix[right + 1]` without needing special `if-else` branches.

### Breakdown of `check_subarray_sum`:
1. `remainder_map = {0: -1}`:
   * Maps modulo remainder to earliest index seen. Initialized with `{0: -1}` to handle cases where a subarray starting at index 0 is directly divisible by $k$ (length $= i - (-1) = i + 1$).
2. `curr_sum += num`:
   * Computes running prefix sum without allocating an array.
3. `rem = curr_sum % k`:
   * Computes remainder modulo $k$.
   * **Modulo Equivalence Invariant**: If two prefixes have the exact same remainder modulo $k$, their difference is an exact multiple of $k$.
4. `if rem in remainder_map:`:
   * Checks if this remainder has occurred previously in $O(1)$ average time.
5. `if i - remainder_map[rem] >= 2: return True`:
   * Verifies the subarray length condition ($\ge 2$).
6. `else: remainder_map[rem] = i`:
   * **Greedy Invariant**: Never overwrites an existing remainder key! Preserving the earliest seen index maximizes the subarray length for future elements.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(N)$ (Optimal)
- **Theoretical Lower Bound**: Any algorithm determining whether an unsorted array contains a subarray satisfying an arbitrary sum property must inspect every element at least once ($\Omega(N)$ lower bound).
- **Proof of Upper Bound**:
  - Precomputing `NumArray` takes $N$ additions $\implies O(N)$. Each query takes $O(1)$ constant time.
  - `check_subarray_sum`: Iterates through $N$ items. Each iteration performs 1 addition, 1 modulo operation, 1 hash lookup, and at most 1 hash insertion. Total runtime is $O(N)$.
- **Comparison to Brute Force**: Evaluating all subarrays directly requires $O(N^2)$ time. Prefix Sum reduces queries to $O(1)$ and subarray matching to $O(N)$.

### Space Complexity: $O(N)$ or $O(\min(N, K))$ (Optimal)
- In `product_except_self`, prefix and suffix products are stored directly inside the output array `res`, achieving $O(1)$ auxiliary space beyond the return container.
- In `check_subarray_sum`, remainders modulo $K$ are bounded by the range $[0, K - 1]$. By the Pigeonhole Principle, the hash map contains at most $\min(N, K)$ entries, ensuring optimal space usage.

---

## 7. Key Invariants & Common Pitfalls

1. **The Size $N+1$ Prefix Array**:
   * *Trap*: Creating a prefix array of size $N$ forces a conditional check `if left == 0` for every query.
   * *Rule*: Always allocate size $N+1$ with index `0` equal to `0`.
2. **Missing `{0: -1}` Base Case**:
   * *Trap*: Forgetting `{0: -1}` in hash map prefix problems will cause valid subarrays beginning at index `0` to be ignored.
3. **Overwriting Earliest Indices**:
   * *Trap*: Doing `remainder_map[rem] = i` on every encounter resets the recorded start position to the newest index, shrinking the distance $i - \text{prev}$ and failing the length $\ge 2$ check.

---

## 8. Canonical Problem Walkthrough

### Contiguous Array (LeetCode #525)
* **Problem**: Find the maximum length of a contiguous subarray with an equal number of `0`s and `1`s.
* **Reduction**: Map `0` $\to -1$ and `1` $\to +1$. An equal count of 0s and 1s corresponds to a subarray sum of `0`!
* **Optimality**: Operates in $O(N)$ time with $O(N)$ space.
