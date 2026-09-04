# Pattern 02: Sliding Window

> **Layer**: 1 (Core Foundation)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/), [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/), [Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/), [Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/)

---

## 1. Mental Model & Core Concept

A Sliding Window maintains an active subsegment `[left, right]` across a sequence (array or string). Instead of recalculating properties of every possible subarray from scratch in $O(N^2)$, it incrementally adds elements at the `right` boundary and removes elements from the `left` boundary in amortized $O(N)$ time.

```text
       left              right
         ▼                 ▼
Array: [ a , b , c , d , e , f , g ]
         └────── window ───┘
Expand:            right moves right ──► (incorporate new item)
Shrink:  left moves right ──►            (evict invalid items until condition restored)
```

There are two primary categories:
1. **Fixed-Size Window**: Length $K$ is invariant. Shift both `left` and `right` in lockstep after reaching size $K$.
2. **Variable-Size Dynamic Window**: Find the longest/shortest subarray satisfying a condition. Expand `right` greedily; shrink `left` as needed.

---

## 2. Identification Signals ("When to Use")

Watch for these constraints and phrasing:
- **Contiguous Subarrays / Substrings**: The problem demands contiguous elements (not subsequences).
- **Extremes with Conditions**: *"Find the maximum length of a substring with at most $K$ distinct characters"*, *"Find minimum length subarray with sum $\ge S$"*.
- **Running Aggregates**: Running sums, character frequency windows, anagram matches in a string.
- **Complexity Goal**: Brute force is $O(N^2)$; optimal sliding window runs in $O(N)$ time with $O(1)$ or $O(K)$ space.

---

## 3. Algorithmic Template / Pseudocode

### Template A: Dynamic Window (Find Longest Valid Window)
```text
function LONGEST_VALID_WINDOW(array):
    left = 0
    max_len = 0
    state = empty tracker (e.g. hash map or counter)

    for right from 0 to length(array) - 1:
        add array[right] to state
        
        // Shrink window if invariant is violated
        while window state is INVALID:
            remove array[left] from state
            left += 1
            
        // At this point, window [left, right] is valid
        max_len = max(max_len, right - left + 1)

    return max_len
```

### Template B: Dynamic Window (Find Shortest Valid Window)
```text
function SHORTEST_VALID_WINDOW(array, target):
    left = 0
    min_len = INFINITY
    state = empty tracker

    for right from 0 to length(array) - 1:
        add array[right] to state
        
        // While condition is satisfied, record and shrink to find smaller valid window
        while window state is VALID:
            min_len = min(min_len, right - left + 1)
            remove array[left] from state
            left += 1

    return min_len if min_len != INFINITY else 0
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from collections import defaultdict
from typing import List

# --- Variant 1: Longest Substring Without Repeating Characters ---
def length_of_longest_substring(s: str) -> int:
    char_index_map = {}  # char -> last seen index
    left = 0
    max_length = 0

    for right, char in enumerate(s):
        # If character is already in window, jump 'left' past its previous occurrence
        if char in char_index_map and char_index_map[char] >= left:
            left = char_index_map[char] + 1
            
        char_index_map[char] = right
        max_length = max(max_length, right - left + 1)

    return max_length

# --- Variant 2: Minimum Size Subarray Sum ---
def min_sub_array_len(target: int, nums: List[int]) -> int:
    left = 0
    curr_sum = 0
    min_len = float('inf')

    for right in range(len(nums)):
        curr_sum += nums[right]

        # Invariant: shrink as long as current window meets or exceeds target
        while curr_sum >= target:
            min_len = min(min_len, right - left + 1)
            curr_sum -= nums[left]
            left += 1

    return int(min_len) if min_len != float('inf') else 0

# --- Variant 3: Fixed Size Window (Max Sum Subarray of Size K) ---
def max_sub_array_of_size_k(k: int, nums: List[int]) -> int:
    if len(nums) < k:
        return 0
    
    window_sum = sum(nums[:k])
    max_sum = window_sum

    for right in range(k, len(nums)):
        window_sum += nums[right] - nums[right - k]  # Add incoming, subtract outgoing
        max_sum = max(max_sum, window_sum)

    return max_sum
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `length_of_longest_substring`:
1. `char_index_map = {}`:
   * Stores the most recent index where each character appeared.
2. `left = 0; max_length = 0`:
   * `left` represents the starting boundary of the non-repeating window; `max_length` records the running maximum.
3. `for right, char in enumerate(s):`:
   * Right pointer monotonically advances across the string from index $0$ to $N-1$.
4. `if char in char_index_map and char_index_map[char] >= left:`:
   * **Crucial Jump Invariant**: If `char` was seen before, but its previous position was *before* `left` (`char_index_map[char] < left`), it is already outside the current window and can be ignored!
   * If its position is $\ge left$, we have a duplicate inside the window, so we jump `left` directly to `char_index_map[char] + 1` in $O(1)$ without an inner while loop.
5. `char_index_map[char] = right`:
   * Updates the character's last seen position to current index `right`.
6. `max_length = max(max_length, right - left + 1)`:
   * Window size is always `right - left + 1`. We update the maximum valid window seen so far.
7. `return max_length`:
   * Returns the length of the longest valid substring found.

### Breakdown of `min_sub_array_len`:
1. `left = 0; curr_sum = 0; min_len = float('inf')`:
   * Initializes running sum and sets minimum length to infinity.
2. `for right in range(len(nums)): curr_sum += nums[right]`:
   * Expands the window by adding `nums[right]`.
3. `while curr_sum >= target:`:
   * As long as the window meets or exceeds `target`, it is a valid candidate.
4. `min_len = min(min_len, right - left + 1)`:
   * Records candidate minimum window length.
5. `curr_sum -= nums[left]; left += 1`:
   * Shrinks the window from the left to test if a strictly smaller valid window exists.
6. `return int(min_len) if min_len != float('inf') else 0`:
   * Returns minimum length found, or 0 if no valid subarray exists.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(N)$ (Optimal)
- **Theoretical Lower Bound**: To find a subarray or substring with an optimal property across an arbitrary sequence of length $N$, any algorithm must inspect every element at least once ($\Omega(N)$ lower bound).
- **Proof of Amortized $O(N)$**:
  - Even though `min_sub_array_len` has a `while` loop nested inside a `for` loop, analyze pointer movements:
    - The `right` pointer increments exactly $N$ times.
    - The `left` pointer only increments, never resets backwards, and cannot exceed `right`. Therefore, `left` increments at most $N$ times throughout the entire execution.
    - Total operations: $N \text{ (right moves)} + N \text{ (left moves)} = 2N \text{ operations} \implies O(N)$.
- **Comparison to Brute Force**: Brute force evaluates all $\frac{N(N+1)}{2}$ subarrays, computing their sums in $O(N^2)$ or $O(N^3)$ time. Sliding window achieves an exponential speedup to $O(N)$.

### Space Complexity: $O(1)$ to $O(\min(N, U))$ (Optimal)
- In `min_sub_array_len` and `max_sub_array_of_size_k`, state is tracked with scalar numbers (`curr_sum`, `left`, `right`), requiring strictly $O(1)$ auxiliary memory.
- In `length_of_longest_substring`, the map stores at most $U$ unique characters (where $U \le 26$ for lowercase, or $U \le 128$ for ASCII, or $U \le N$ for arbitrary Unicode), which is $O(1)$ space for fixed character sets.

---

## 7. Key Invariants & Common Pitfalls

1. **Monotonicity Requirement**:
   * Sliding window sum techniques *require non-negative numbers*. If the array contains negative numbers, expanding `right` does not monotonically increase the sum, and shrinking `left` does not monotonically decrease it. For negative numbers, use Pattern 05 (Prefix Sum + Hash Map).
2. **Stale Indices in Hash Map**:
   * *Trap*: Without `char_index_map[char] >= left`, jumping `left` to an old index from a previous window can cause `left` to erroneously move *backwards*, corrupting the window.

---

## 8. Canonical Problem Walkthrough

### Longest Repeating Character Replacement (LeetCode #424)
* **Problem**: Replace at most $k$ characters in string `s` to produce the longest uniform substring.
* **Window Invariant**: In any window of length $L = \text{right} - \text{left} + 1$, valid if:
  $$L - \text{max\_frequency} \le k$$

```python
def character_replacement(s: str, k: int) -> int:
    count = defaultdict(int)
    left = 0
    max_freq = 0
    max_len = 0
    
    for right in range(len(s)):
        count[s[right]] += 1
        max_freq = max(max_freq, count[s[right]])
        
        # If window size - max frequency > k, it's invalid -> shrink
        while (right - left + 1) - max_freq > k:
            count[s[left]] -= 1
            left += 1
            
        max_len = max(max_len, right - left + 1)
        
    return max_len
```
* **Optimality**: Operates in $O(N)$ time with $O(26) = O(1)$ space.
