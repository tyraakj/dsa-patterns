# Pattern 35: Sliding Window Maximum / Minimum

> **Layer**: 3 (Advanced & Specialized)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/), [Constrained Subsequence Sum](https://leetcode.com/problems/constrained-subsequence-sum/), [Shortest Subarray with Sum at Least K](https://leetcode.com/problems/shortest-subarray-with-sum-at-least-k/)

---

## 1. Mental Model & Core Concept

While a regular sliding window tracks scalar aggregates (like sum or frequency counts), finding the **Maximum or Minimum element over a moving window of size $K$** cannot be done in $O(1)$ with a simple variable because evicting the maximum requires finding the second-largest element.

```text
Monotonic Deque (Double-Ended Queue) Invariant:
Elements stored in the deque are STRICTLY MONOTONICALLY DECREASING.
Front of the deque ALWAYS holds the MAXIMUM of the current window!

Array:  [ 1 , 3 , -1 , -3 , 5 , 3 , 6 , 7 ],  k = 3

Window [1, 3, -1]:
- Push 1:  deque = [1]
- Push 3:  3 > 1! Pop 1 from back! (1 can NEVER be max while 3 is alive!)
           deque = [3]
- Push -1: -1 < 3. Append to back.
           deque = [3, -1]
           Max is deque.front() = 3!
```

Two Invariant Maintenance Rules:
1. **Evict Outdated Elements from Front**: If `deque[0] < i - k + 1`, the element has slid out of the window $\implies$ pop from left.
2. **Evict Inferior Elements from Back**: When adding `nums[i]`, pop all elements from the right of the deque that are $\le nums[i]$ (they will never be the maximum again).

---

## 2. Identification Signals ("When to Use")

- **Extremes Over Moving Window**: Finding max/min in every window of size $K$ in $O(N)$ time (avoiding $O(N \log K)$ heaps).
- **DP Optimization**: Transition equations of the form:
  $$\text{dp}[i] = \text{nums}[i] + \max_{i - k \le j < i} \text{dp}[j]$$

---

## 3. Algorithmic Template / Pseudocode

```text
function SLIDING_WINDOW_MAXIMUM(nums, k):
    deque = empty double-ended queue // stores INDICES
    result = []

    for i from 0 to length(nums) - 1:
        // 1. Remove indices outside current window
        if deque is not empty and deque.front() < i - k + 1:
            deque.pop_front()

        // 2. Maintain decreasing monotonic order
        while deque is not empty and nums[i] >= nums[deque.back()]:
            deque.pop_back()

        deque.push_back(i)

        // 3. Record max once first window of size k is formed
        if i >= k - 1:
            result.append(nums[deque.front()])

    return result
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from collections import deque
from typing import List

# --- Variant A: Sliding Window Maximum (Hard - O(N) Optimal) ---
def max_sliding_window(nums: List[int], k: int) -> List[int]:
    res = []
    # Monotonically decreasing deque storing indices
    d = deque()

    for i, n in enumerate(nums):
        # 1. Pop smaller elements from back of deque
        while d and nums[d[-1]] < n:
            d.pop()

        d.append(i)

        # 2. Pop elements that have fallen outside the left window boundary
        if d[0] < i - k + 1:
            d.popleft()

        # 3. Add to result once the window has expanded to size k
        if i >= k - 1:
            res.append(nums[d[0]])

    return res

# --- Variant B: Constrained Subsequence Sum (DP + Monotonic Deque) ---
def constrained_subset_sum(nums: List[int], k: int) -> int:
    # dp[i] = nums[i] + max(0, max(dp[i-k...i-1]))
    dp = list(nums)
    d = deque()  # stores indices with decreasing dp values

    for i in range(len(nums)):
        # Remove elements older than distance k
        if d and d[0] < i - k:
            d.popleft()

        if d:
            dp[i] = max(dp[i], nums[i] + dp[d[0]])

        # Maintain decreasing order in deque
        while d and dp[i] >= dp[d[-1]]:
            d.pop()

        if dp[i] > 0:
            d.append(i)

    return max(dp)
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `max_sliding_window`:
1. `d = deque()`:
   * Double-ended queue storing **indices** (to enable window expiration checks).
2. `while d and nums[d[-1]] < n: d.pop()`:
   * **Monotonic Maintenance**: If incoming element $n$ is strictly greater than the element at the back of the deque, the older smaller element will *never* be the maximum of any current or future window. We pop it from the back in $O(1)$.
3. `d.append(i)`:
   * Appends current index. The deque is guaranteed to be in descending order of value.
4. `if d[0] < i - k + 1: d.popleft()`:
   * **Window Expiration**: If the front element's index is out of the valid range $[i - k + 1, i]$, evict it from the front in $O(1)$.
5. `if i >= k - 1: res.append(nums[d[0]])`:
   * The front of the deque is guaranteed to hold the index of the maximum element in the window.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(N)$ (Optimal)
- **Proof**:
  - Each element index $i \in [0, N-1]$ is pushed into the deque **at most once**.
  - Each element index is popped from the deque (either from front or back) **at most once**.
  - Total push operations $= N$; total pop operations $\le N$.
  - Total deque mutations $\le 2N$.
  - Amortized cost per array element is $O(1)$, yielding strict $O(N)$ overall time.
  - **Comparison**: A standard Max-Heap takes $O(N \log K)$ or $O(N \log N)$ with lazy eviction. Monotonic Deque achieves true linear $O(N)$.

### Space Complexity: $O(K)$ (Optimal)
- At any point, the deque contains at most $K$ indices (the current window span) $\implies O(K)$ auxiliary space.

---

## 7. Key Invariants & Common Pitfalls

1. **Storing Indices vs Values**:
   * Storing raw values prevents expiration checking (`d[0] < i - k + 1`). Always store indices.
2. **Strictly Less vs Less-or-Equal**:
   * Using `nums[d[-1]] < n` retains duplicates in the deque, ensuring correct count tracking when duplicate maximums exist.

---

## 8. Canonical Problem Walkthrough

### Shortest Subarray with Sum at Least K (LeetCode #862 - Hard)
* **Problem**: Shortest subarray with sum $\ge K$ (contains negative numbers!).
* **Monotonic Deque on Prefix Sums**:
  1. If `prefix[i] - prefix[d[0]] >= k`, record length and pop front (since future $i'$ will only produce longer subarrays).
  2. If `prefix[i] <= prefix[d[-1]]`, pop back (a smaller prefix at a later index is strictly superior).
* **Optimality**: Solves an otherwise quadratic problem with negative numbers in $O(N)$ time and $O(N)$ space.
