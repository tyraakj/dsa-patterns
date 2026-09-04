# Pattern 14: Recursion / Divide and Conquer

> **Tier**: 1 (Core Foundation)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Pow(x, n)](https://leetcode.com/problems/powx-n/), [Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/), [Sort List](https://leetcode.com/problems/sort-list/), [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/)

---

## 1. Mental Model & Core Concept

Divide and Conquer breaks a problem into **non-overlapping** independent subproblems, recursively solves each subproblem, and merges the sub-results into the final solution:

```text
Divide and Conquer Structure:
                 Problem of Size N
                  /             \
      Subproblem (N/2)        Subproblem (N/2)
         /        \              /        \
       (N/4)     (N/4)         (N/4)     (N/4)
         └─────────┴──────────────┴─────────┘
                       MERGE PHASE
```

Master Theorem Intuition:
- **Fast Exponentiation**: $x^n = (x^{n/2})^2 \implies O(\log N)$ steps instead of $O(N)$ multiplications.
- **Merge Sort**: Divide in half in $O(1)$, sort recursively, merge two sorted halves in $O(N) \implies O(N \log N)$ total time.
- **Quickselect**: Partition array around pivot; recurse into **only one** half $\implies O(N)$ average time.

---

## 2. Identification Signals ("When to Use")

- **Subproblem Halving**: Splitting problems symmetrically into halves ($N/2$).
- **Merging Sorted Partitions**: Sorting linked lists in $O(N \log N)$ time with $O(1)$ auxiliary space.
- **Order Statistics**: Finding $K$-th largest/smallest element in unsorted arrays without full sorting.
- **Binary Power Calculations**: Calculating $x^n$ for massive powers $n \le 2^{31} - 1$.

---

## 3. Algorithmic Template / Pseudocode

```text
function DIVIDE_AND_CONQUER(problem):
    if problem is simple base case:
        return solve_base_case(problem)
        
    subproblem1, subproblem2 = split(problem)
    
    result1 = DIVIDE_AND_CONQUER(subproblem1)
    result2 = DIVIDE_AND_CONQUER(subproblem2)
    
    return merge(result1, result2)
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import Optional, List

class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

# --- Variant A: Fast Binary Exponentiation (Pow(x, n)) ---
def my_pow(x: float, n: int) -> float:
    def binary_pow(base: float, exp: int) -> float:
        if exp == 0:
            return 1.0
        half = binary_pow(base, exp // 2)
        if exp % 2 == 0:
            return half * half
        else:
            return half * half * base

    if n < 0:
        x = 1.0 / x
        n = -n

    return binary_pow(x, n)

# --- Variant B: Merge Sort on Linked List (Sort List in O(N log N)) ---
def sort_list(head: Optional[ListNode]) -> Optional[ListNode]:
    if not head or not head.next:
        return head

    # Step 1: Split list into two halves using Fast & Slow pointers
    prev = None
    slow, fast = head, head
    while fast and fast.next:
        prev = slow
        slow = slow.next
        fast = fast.next.next

    prev.next = None  # Sever connection to create two separate lists

    # Step 2: Recursively sort both halves
    left = sort_list(head)
    right = sort_list(slow)

    # Step 3: Merge the two sorted linked lists
    dummy = ListNode(0)
    curr = dummy
    while left and right:
        if left.val <= right.val:
            curr.next = left
            left = left.next
        else:
            curr.next = right
            right = right.next
        curr = curr.next

    curr.next = left if left else right
    return dummy.next

# --- Variant C: Quickselect for Kth Largest Element (O(N) Average) ---
def find_kth_largest(nums: List[int], k: int) -> int:
    import random
    # Convert kth largest to target index in ascending order
    target_idx = len(nums) - k

    def quickselect(left: int, right: int) -> int:
        pivot_idx = random.randint(left, right)
        pivot = nums[pivot_idx]
        nums[pivot_idx], nums[right] = nums[right], nums[pivot_idx]

        p = left
        for i in range(left, right):
            if nums[i] <= pivot:
                nums[p], nums[i] = nums[i], nums[p]
                p += 1

        nums[p], nums[right] = nums[right], nums[p]

        if p == target_idx:
            return nums[p]
        elif p < target_idx:
            return quickselect(p + 1, right)
        else:
            return quickselect(left, p - 1)

    return quickselect(0, len(nums) - 1)
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `my_pow`:
1. `if exp == 0: return 1.0`:
   * Base case: any number raised to power 0 is 1.0.
2. `half = binary_pow(base, exp // 2)`:
   * **Divide and Conquer Halving**: Computes $base^{\lfloor exp/2 \rfloor}$ recursively *only once*. Reusing `half` avoids evaluating the duplicate branch.
3. `if exp % 2 == 0: return half * half else: return half * half * base`:
   * If even power, squares the half result. If odd power, multiplies by an additional factor of `base`.
4. `if n < 0: x = 1.0 / x; n = -n`:
   * Handles negative exponents symmetrically: $x^{-n} = (1/x)^n$.

### Breakdown of `sort_list`:
1. `while fast and fast.next: prev = slow; slow = slow.next; fast = fast.next.next`:
   * Locates midpoint of linked list in $O(N)$ time.
2. `prev.next = None`:
   * Cuts the linked list into two independent components `head` and `slow`.
3. `left = sort_list(head); right = sort_list(slow)`:
   * Recursively sorts each half of size $N/2$.
4. `while left and right: ... curr.next = left if left else right`:
   * Merges two sorted lists in linear time $O(N)$.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity:
- **`my_pow`**: $T(N) = T(N/2) + O(1)$. By the Master Theorem, $T(N) = O(\log N)$. This is optimal because computing $x^N$ via repeated multiplication takes $O(N)$ steps.
- **`sort_list`**: $T(N) = 2T(N/2) + O(N)$. By the Master Theorem (Case 2), $T(N) = O(N \log N)$, which matches the proven $\Omega(N \log N)$ lower bound for comparison-based sorting.
- **`find_kth_largest`**: $T(N) = T(N/2) + O(N)$ on average. Recurrence resolves to $N + \frac{N}{2} + \frac{N}{4} + \dots \le 2N \implies O(N)$ average time.

### Space Complexity:
- **`my_pow`**: $O(\log N)$ stack frames during recursive halving.
- **`sort_list`**: $O(\log N)$ recursion stack depth with $O(1)$ heap memory (merging nodes in-place).

---

## 7. Key Invariants & Common Pitfalls

1. **Severing Linked Lists**:
   * *Trap*: Forgetting `prev.next = None` leaves the left half connected to the right half, causing infinite recursion.
2. **Branch Duplication**:
   * *Trap*: Writing `return binary_pow(base, exp // 2) * binary_pow(base, exp // 2)` executes two identical recursive calls, destroying the $O(\log N)$ speedup and degrading to $O(N)$! Always save the result in a local variable `half`.

---

## 8. Canonical Problem Walkthrough

### Merge Two Sorted Lists (LeetCode #21)
* **Problem**: Merge two sorted linked lists into one sorted list.
* **Recursive Formulation**:
```python
def merge_two_lists(l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
    if not l1:
        return l2
    if not l2:
        return l1
    
    if l1.val < l2.val:
        l1.next = merge_two_lists(l1.next, l2)
        return l1
    else:
        l2.next = merge_two_lists(l1, l2.next)
        return l2
```
* **Optimality**: Operates in $O(N + M)$ time and $O(N + M)$ stack space.
