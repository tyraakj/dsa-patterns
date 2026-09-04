# Pattern 01: Hash Map / Hash Set

> **Layer**: 1 (Core Foundation)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Two Sum](https://leetcode.com/problems/two-sum/), [Group Anagrams](https://leetcode.com/problems/group-anagrams/), [Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/), [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)

---

## 1. Mental Model & Core Concept

A Hash Map (dictionary) or Hash Set trades space for time, converting an $O(N)$ linear search into an average $O(1)$ amortized lookup via hashing. 

```text
Linear Scan with Memory:
Element x  ───► Check if (Target - x) in Lookup Table?
                  ├── Yes ──► Match Found! Return indices / answer
                  └── No  ──► Store x in Lookup Table for future queries
```

The core intuition is **memoization of past state**: instead of looking forward or re-scanning past elements with nested loops ($O(N^2)$), store seen values, frequencies, or indices so that each subsequent element queries past history in $O(1)$ time.

---

## 2. Identification Signals ("When to Use")

Look for these cues in problem descriptions:
- **Pair or Complement Matching**: *"Find two numbers that add up to target"* $\implies$ query `target - num` in a hash map.
- **Frequency Counting**: *"Find the most frequent / top K elements / anagram check"* $\implies$ tally occurrences via `collections.Counter` or `collections.defaultdict(int)`.
- **Categorization / Grouping**: *"Group words by common anagram signature / coordinate grouping"* $\implies$ use a tuple of character counts or sorted string as a hashable dictionary key.
- **Deduplication & Uniqueness**: *"Check for duplicates / longest consecutive streak"* $\implies$ query in a Hash Set in $O(1)$ time.
- **Prefix Sum Intersections**: *"Count continuous subarrays summing to K"* $\implies$ hash map storing running prefix sums and their frequency counts.

---

## 3. Algorithmic Template / Pseudocode

```text
function HASH_MAP_COMPLEMENT_SEARCH(nums, target):
    lookup = empty hash map (stores value -> index)
    
    for i from 0 to length(nums) - 1:
        complement = target - nums[i]
        if complement exists in lookup:
            return [lookup[complement], i]
        lookup[nums[i]] = i
        
    return []  // No valid pair found
```

```text
function GROUPING_BY_SIGNATURE(items):
    groups = hash map mapping (signature -> list of items)
    
    for each item in items:
        sig = compute_canonical_signature(item)  // e.g., sorted string or char tuple
        groups[sig].append(item)
        
    return list of all values in groups
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from collections import defaultdict, Counter
from typing import List, Dict, Tuple, Optional

# --- Variant A: Complement Lookup (Two Sum) ---
def two_sum(nums: List[int], target: int) -> List[int]:
    seen: Dict[int, int] = {}  # num -> index
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return []

# --- Variant B: Grouping by Canonical Key (Group Anagrams) ---
def group_anagrams(strs: List[str]) -> List[List[str]]:
    groups: Dict[Tuple[int, ...], List[str]] = defaultdict(list)
    for s in strs:
        # Array of 26 character frequencies as a hashable tuple
        count = [0] * 26
        for char in s:
            count[ord(char) - ord('a')] += 1
        groups[tuple(count)].append(s)
    return list(groups.values())

# --- Variant C: O(1) Set Sequence Expansion (Longest Consecutive Sequence) ---
def longest_consecutive(nums: List[int]) -> int:
    num_set = set(nums)
    max_streak = 0

    for num in num_set:
        # Only start counting if 'num' is the start of a streak
        if num - 1 not in num_set:
            current_num = num
            current_streak = 1
            while current_num + 1 in num_set:
                current_num += 1
                current_streak += 1
            max_streak = max(max_streak, current_streak)

    return max_streak
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `two_sum`:
1. `seen: Dict[int, int] = {}`:
   * Allocates an empty hash table mapping array values to their corresponding index in `nums`.
2. `for i, num in enumerate(nums):`:
   * Executes a single linear scan from index $0$ to $N-1$, providing index $i$ and element $num$.
3. `complement = target - num`:
   * Calculates the exact partner value required to reach `target`.
4. `if complement in seen:`:
   * Queries the hash table in $O(1)$ amortized time.
   * **Invariant**: `seen` contains only elements *prior* to index $i$. This mechanically prevents pairing an element with itself.
5. `return [seen[complement], i]`:
   * Returns the earlier index and current index $i$ immediately upon first discovery.
6. `seen[num] = i`:
   * If complement was not found, records current number and index for subsequent elements to query.
7. `return []`:
   * Fallback for when no valid pair exists.

### Breakdown of `longest_consecutive`:
1. `num_set = set(nums)`:
   * Deduplicates array and places elements in a hash set for $O(1)$ membership testing.
2. `for num in num_set:`:
   * Iterates over unique numbers.
3. `if num - 1 not in num_set:`:
   * **Crucial Pruning Check**: We only initiate streak expansion if `num` is the **absolute beginning** of a sequence. If `num - 1` exists, `num` is part of an ongoing sequence and will be evaluated when its predecessor is visited. This eliminates redundant $O(N^2)$ checks!
4. `while current_num + 1 in num_set: current_num += 1; current_streak += 1`:
   * Expands forward sequentially as long as consecutive neighbors exist in the set.
5. `max_streak = max(max_streak, current_streak)`:
   * Updates global maximum sequence length.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(N)$ (Optimal)
- **Theoretical Lower Bound**: In an unsorted, arbitrary array, determining if two numbers add up to a target requires inspecting each element at least once. Any algorithm must have a time complexity of at least $\Omega(N)$.
- **Proof of Upper Bound**:
  - `two_sum`: Traverses $N$ elements. Each step does one subtraction, one hash map lookup ($O(1)$ amortized), and one hash map write ($O(1)$ amortized). Total operations: $C \cdot N \implies O(N)$.
  - `longest_consecutive`: Even with the inner `while` loop, each number is visited at most twice: once during the outer `for` loop and once during sequence expansion. Total operations $\le 2N \implies O(N)$.
- **Comparison to Brute Force**: A double nested loop comparing all pairs takes $\frac{N(N-1)}{2} = O(N^2)$ time. Hash Map drops this to the theoretical minimum $O(N)$.

### Space Complexity: $O(N)$ (Optimal)
- To achieve $O(1)$ lookup without mutating or sorting the input array, we must store previously seen elements in memory. Storing $N$ keys requires $O(N)$ auxiliary space.
- Any algorithm running in $o(N)$ auxiliary space on unsorted inputs must re-scan earlier elements, degrading runtime to $O(N \log N)$ (sorting) or $O(N^2)$ (brute force).

---

## 7. Key Invariants & Common Pitfalls

1. **Using the Same Element Twice**:
   * *Trap*: If you pre-populate the entire hash map before checking, `seen[complement]` might return the current index $i$ (e.g. `nums = [3, 2, 4], target = 6` might match `3 + 3` using index 0 twice).
   * *Rule*: Single-pass checking: always check `complement in seen` **before** adding `seen[num] = i`.
2. **Unhashable Mutable Types**:
   * *Trap*: In Python, lists cannot be dictionary keys (`TypeError: unhashable type: 'list'`).
   * *Rule*: Convert lists to tuples (e.g., `tuple(count)` for anagram character frequencies).
3. **Worst-Case Collisions**:
   * While average lookup is $O(1)$, worst-case under malicious hash collisions is $O(N)$. Python 3 uses randomized SipHash to prevent predictable collision attacks.

---

## 8. Canonical Problem Walkthrough

### Subarray Sum Equals K (LeetCode #560)
* **Problem**: Find total number of continuous subarrays whose sum equals `k`.
* **State Invariant**: At index $j$, if `curr_sum - k` was seen $M$ times in prior prefix sums, exactly $M$ subarrays ending at $j$ sum to $k$.

```python
def subarray_sum(nums: List[int], k: int) -> int:
    count = 0
    curr_sum = 0
    prefix_counts = defaultdict(int)
    prefix_counts[0] = 1  # Base case: empty prefix has sum 0
    
    for num in nums:
        curr_sum += num
        count += prefix_counts[curr_sum - k]
        prefix_counts[curr_sum] += 1
        
    return count
```
* **Optimality**: Runs in $O(N)$ time and $O(N)$ space, compared to $O(N^2)$ naive prefix comparison.
