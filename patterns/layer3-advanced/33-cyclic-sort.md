# Pattern 33: Cyclic Sort

> **Layer**: 3 (Advanced & Specialized)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Missing Number](https://leetcode.com/problems/missing-number/), [Find All Numbers Disappeared in an Array](https://leetcode.com/problems/find-all-numbers-disappeared-in-an-array/), [Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/), [First Missing Positive](https://leetcode.com/problems/first-missing-positive/)

---

## 1. Mental Model & Core Concept

Cyclic Sort operates on arrays containing numbers within a known range $[1, N]$ or $[0, N]$. Instead of using an external hash set to track presence, it **uses the input array itself as its own hash table**:

```text
The Cyclic Invariant:
For an array containing numbers from 1 to N:
Every element value 'x' belongs at index 'x - 1'!

Index:    0    1    2    3
Array:  [ 3 ,  4 , -1 ,  1 ]

Step 1: Value 3 belongs at index 2. Swap 3 with -1:
        [ -1 , 4 ,  3 ,  1 ]
Step 2: -1 is out of range [1, 4]. Skip.
Step 3: Value 4 belongs at index 3. Swap 4 with 1:
        [ -1 , 1 ,  3 ,  4 ]
Step 4: Value 1 belongs at index 0. Swap 1 with -1:
        [  1 , -1 , 3 ,  4 ]

Result: At index 1, value is -1 != 2. Thus 2 is the First Missing Positive!
```

---

## 2. Identification Signals ("When to Use")

- **Numbers in Range $[1, N]$ or $[0, N]$**: Finding missing numbers, duplicate numbers, or corrupted elements.
- **Strict Constraints**: Demands **$O(N)$ runtime** and **$O(1)$ auxiliary space**.
- **First Missing Positive**: The hardest variant; ignore non-positive numbers and values $> N$.

---

## 3. Algorithmic Template / Pseudocode

```text
function CYCLIC_SORT(nums):
    i = 0
    n = length(nums)

    while i < n:
        correct_idx = nums[i] - 1 // Or nums[i] for [0, n - 1]
        
        // If element is in valid range and not already at its correct position
        if 1 <= nums[i] <= n and nums[i] != nums[correct_idx]:
            swap(nums[i], nums[correct_idx])
        else:
            i += 1 // Advance only when current position is settled

    // Scan for mismatched index
    for i from 0 to n - 1:
        if nums[i] != i + 1:
            return i + 1 // Missing element!
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import List

# --- Variant A: First Missing Positive (Hard) ---
def first_missing_positive(nums: List[int]) -> int:
    n = len(nums)
    i = 0

    while i < n:
        correct_idx = nums[i] - 1
        # Place nums[i] at index nums[i] - 1 if in range [1, n] and not a duplicate
        if 1 <= nums[i] <= n and nums[i] != nums[correct_idx]:
            nums[i], nums[correct_idx] = nums[correct_idx], nums[i]
        else:
            i += 1

    # First index where value does not match expected (i + 1)
    for i in range(n):
        if nums[i] != i + 1:
            return i + 1

    return n + 1

# --- Variant B: Find All Disappeared Numbers in Array ---
def find_disappeared_numbers(nums: List[int]) -> List[int]:
    n = len(nums)
    i = 0

    while i < n:
        correct_idx = nums[i] - 1
        if nums[i] != nums[correct_idx]:
            nums[i], nums[correct_idx] = nums[correct_idx], nums[i]
        else:
            i += 1

    return [i + 1 for i in range(n) if nums[i] != i + 1]
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `first_missing_positive`:
1. `while i < n:`:
   * Drives the cyclic placement phase.
2. `correct_idx = nums[i] - 1`:
   * The destination index where value `nums[i]` must reside.
3. `if 1 <= nums[i] <= n and nums[i] != nums[correct_idx]:`:
   * Validates that `nums[i]` is within range $[1, n]$.
   * **Duplicate Prevention Invariant**: `nums[i] != nums[correct_idx]` ensures we do not attempt to swap if the destination index already contains that exact value, avoiding infinite swap loops!
4. `nums[i], nums[correct_idx] = nums[correct_idx], nums[i]`:
   * Places `nums[i]` into its home slot in $O(1)$. Notice we do *not* increment $i$, because the incoming element swapped into index $i$ must now be placed into its own home slot.
5. `else: i += 1`:
   * Only advance $i$ when the element at $i$ is out of range or already at its home slot.
6. `for i in range(n): if nums[i] != i + 1: return i + 1`:
   * The first slot containing an incongruent number exposes the smallest missing positive integer.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(N)$ (Optimal)
- **Proof**:
  - Each swap places at least **one** element into its final, correct destination index $nums[i] - 1$.
  - Once an element is at its correct index, it is never swapped again.
  - Therefore, at most $N$ swaps can occur across the entire while loop.
  - The pointer $i$ advances at most $N$ times.
  - Total operations $\le 2N \implies O(N)$ runtime.
  - Optimal because verifying presence in an unsorted array requires $\Omega(N)$ operations.

### Space Complexity: $O(1)$ (Optimal)
- Swapping occurs entirely in-place within the provided array without allocating auxiliary sets or hash tables $\implies O(1)$ space.

---

## 7. Key Invariants & Common Pitfalls

1. **Python Tuple Swap Bug**:
   * *Trap*: In Python, `nums[i], nums[nums[i] - 1] = nums[nums[i] - 1], nums[i]` evaluates `nums[i]` on the left first! Mutating `nums[i]` changes the index for the second assignment.
   * *Rule*: Always pre-calculate `correct_idx = nums[i] - 1` before swapping: `nums[i], nums[correct_idx] = nums[correct_idx], nums[i]`.
2. **Infinite Loops on Duplicates**:
   * If array contains duplicates (e.g. `[1, 1]`), swapping `nums[i]` with `nums[correct_idx]` without checking `nums[i] != nums[correct_idx]` causes an infinite loop.

---

## 8. Canonical Problem Walkthrough

### Set Mismatch (LeetCode #645)
* **Problem**: Array of numbers from 1 to $n$ has one duplicate and one missing number. Return `[duplicate, missing]`.

```python
def find_error_nums(nums: List[int]) -> List[int]:
    i = 0
    while i < len(nums):
        c_idx = nums[i] - 1
        if nums[i] != nums[c_idx]:
            nums[i], nums[c_idx] = nums[c_idx], nums[i]
        else:
            i += 1
            
    for i, num in enumerate(nums):
        if num != i + 1:
            return [num, i + 1]
    return []
```
* **Optimality**: Operates in $O(N)$ time and $O(1)$ space.
