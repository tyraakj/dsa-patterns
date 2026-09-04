# Pattern 09: Monotonic Stack / Queue

> **Layer**: 1 (Core Foundation)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Daily Temperatures](https://leetcode.com/problems/daily-temperatures/), [Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/), [Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/), [Online Stock Span](https://leetcode.com/problems/online-stock-span/)

---

## 1. Mental Model & Core Concept

A Monotonic Stack maintains elements in strictly **increasing** or **decreasing** order. When a new incoming element violates this monotonicity, elements are popped from the stack until order is restored. The act of popping identifies the **Next Greater Element (NGE)** or **Next Smaller Element (NSE)** in amortized $O(N)$ time.

```text
Monotonically Decreasing Stack:
Array:    [ 73 , 74 , 75 , 71 , 69 , 72 ]
Push 73 ──► [ 73 ]
Push 74 ──► 74 > 73! Pop 73! (Next greater element for 73 is 74!)
            [ 74 ]
Push 75 ──► 75 > 74! Pop 74! (Next greater element for 74 is 75!)
            [ 75 ]
Push 71 ──► [ 75, 71 ]
Push 69 ──► [ 75, 71, 69 ]
Push 72 ──► 72 > 69! Pop 69! (Next greater for 69 is 72)
            72 > 71! Pop 71! (Next greater for 71 is 72)
            [ 75, 72 ]
```

Rule of Thumb:
- **Next Greater Element**: Use a **monotonically decreasing stack** (pop when incoming element is greater).
- **Next Smaller Element**: Use a **monotonically increasing stack** (pop when incoming element is smaller).

---

## 2. Identification Signals ("When to Use")

- **"Next Greater" or "Next Warmer"**: Finding the nearest element to the right (or left) that is strictly larger.
- **Span Problems**: Stock span, visibility span of buildings.
- **Histogram / Matrix Area Optimization**: Largest rectangle in a histogram, maximal rectangle in a binary matrix.
- **Subarray Extremes**: Subarrays bounded by local minimums or maximums.

---

## 3. Algorithmic Template / Pseudocode

```text
function MONOTONIC_NEXT_GREATER(nums):
    n = length(nums)
    res = array of size n filled with 0 (or -1)
    stack = empty stack // stores INDICES

    for i from 0 to n - 1:
        // While incoming nums[i] is greater than top of stack, top has found its NGE!
        while stack is not empty and nums[i] > nums[stack.top()]:
            prev_index = stack.pop()
            res[prev_index] = nums[i] (or i - prev_index for distance)

        stack.push(i)

    return res
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import List

# --- Variant A: Daily Temperatures (Next Warmer Day Distance) ---
def daily_temperatures(temperatures: List[int]) -> List[int]:
    n = len(temperatures)
    res = [0] * n
    stack = []  # stores indices

    for i, temp in enumerate(temperatures):
        while stack and temp > temperatures[stack[-1]]:
            prev_index = stack.pop()
            res[prev_index] = i - prev_index
        stack.append(i)

    return res

# --- Variant B: Next Greater Element I ---
def next_greater_element(nums1: List[int], nums2: List[int]) -> List[int]:
    nge_map = {}
    stack = []

    for num in nums2:
        while stack and num > stack[-1]:
            nge_map[stack.pop()] = num
        stack.append(num)

    return [nge_map.get(x, -1) for x in nums1]

# --- Variant C: Largest Rectangle in Histogram ---
def largest_rectangle_area(heights: List[int]) -> int:
    stack = []  # stores indices of monotonically increasing heights
    max_area = 0
    # Append dummy 0 height to force popping all remaining bars at termination
    extended_heights = heights + [0]

    for i, h in enumerate(extended_heights):
        while stack and h < extended_heights[stack[-1]]:
            popped_idx = stack.pop()
            height = extended_heights[popped_idx]
            # Width extends from element after current stack top to i
            width = i if not stack else (i - stack[-1] - 1)
            max_area = max(max_area, height * width)
        stack.append(i)

    return max_area
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `daily_temperatures`:
1. `res = [0] * n`:
   * Allocates answer array initialized to 0 (default for days with no warmer temperature).
2. `stack = []`:
   * Monotonically decreasing stack storing **indices** (not raw values) so distance $i - \text{prev}$ can be computed.
3. `for i, temp in enumerate(temperatures):`:
   * Scans temperatures from left to right.
4. `while stack and temp > temperatures[stack[-1]]:`:
   * Condition: if current temperature `temp` is warmer than the temperature at the top index of the stack, the invariant is broken.
5. `prev_index = stack.pop(); res[prev_index] = i - prev_index`:
   * The current day $i$ is the *first* warmer day for `prev_index`! We pop it and record the wait time.
6. `stack.append(i)`:
   * Pushes current day $i$ onto the stack, preserving the monotonic decreasing order.

### Breakdown of `largest_rectangle_area`:
1. `extended_heights = heights + [0]`:
   * Appending a sentinel `0` ensures that all bars still on the stack when the loop reaches the end are flushed out and calculated.
2. `width = i if not stack else (i - stack[-1] - 1)`:
   * **Boundary Invariant**: Because bars on the stack are monotonically increasing, the popped bar was taller than all bars between `stack[-1]` and `i`. Its valid rectangle spans exactly from `stack[-1] + 1` to `i - 1`.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(N)$ (Optimal)
- **Proof**:
  - Each index $i \in [0, N-1]$ is pushed onto the stack **at most once**.
  - Each index is popped from the stack **at most once**.
  - Total push operations $= N$; total pop operations $\le N$.
  - All comparisons inside the `while` loop occur strictly alongside a pop operation.
  - Total runtime $\le 2N$ operations $\implies O(N)$.
- **Comparison to Brute Force**: A naive nested loop scanning to the right for each element takes $O(N^2)$ time. Monotonic Stack avoids re-scanning previously rejected elements.

### Space Complexity: $O(N)$ (Optimal)
- The stack holds at most $N$ elements in the worst case (monotonically decreasing inputs), consuming $O(N)$ auxiliary memory.

---

## 7. Key Invariants & Common Pitfalls

1. **Storing Values vs Indices**:
   * Storing indices is strictly more powerful than storing raw values because it gives you both the value (`nums[stack[-1]]`) and the index/distance.
2. **Strictly Greater vs Greater-or-Equal**:
   * For Next Greater: pop when `incoming > top`.
   * For Next Greater or Equal: pop when `incoming >= top`.
3. **Sentinel Values**:
   * For histogram problems, adding a `0` sentinel to the end cleanly empties the stack, preventing boilerplate flush code after the main loop.

---

## 8. Canonical Problem Walkthrough

### Online Stock Span (LeetCode #901)
* **Problem**: Design an algorithm that collects daily price quotes and returns the span of that stock's price for today (consecutive days price was $\le$ today's price).
* **Stack State**: Store pairs `(price, span)`. Pop smaller prices and merge their spans.

```python
class StockSpanner:
    def __init__(self):
        self.stack = []  # stores (price, span)

    def next(self, price: int) -> int:
        span = 1
        while self.stack and self.stack[-1][0] <= price:
            span += self.stack.pop()[1]
        self.stack.append((price, span))
        return span
```
* **Optimality**: Each price is pushed and popped at most once, yielding amortized $O(1)$ time per `next` call with $O(N)$ space.
