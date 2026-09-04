# Pattern 15: Backtracking / Subsets / Combinations / Permutations

> **Layer**: 1 (Core Foundation)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Subsets](https://leetcode.com/problems/subsets/), [Subsets II](https://leetcode.com/problems/subsets-ii/), [Permutations](https://leetcode.com/problems/permutations/), [Combination Sum](https://leetcode.com/problems/combination-sum/), [Word Search](https://leetcode.com/problems/word-search/), [N-Queens](https://leetcode.com/problems/n-queens/)

---

## 1. Mental Model & Core Concept

Backtracking systematically searches an **exhaustive decision tree** for solutions. It builds candidates incrementally and abandons ("backtracks") a candidate path as soon as it determines the path cannot satisfy the problem constraints.

```text
Decision Tree for Subsets [1, 2]:
                    [ ]
                 /       \
          Include 1      Exclude 1
            [1]             [ ]
           /   \           /   \
        Inc 2  Exc 2    Inc 2  Exc 2
       [1, 2]   [1]      [2]    [ ]
```

The canonical 3-step state invariant:
1. **Choose**: Append candidate to current path.
2. **Explore**: Recurse down the decision tree.
3. **Un-choose (Backtrack)**: Pop candidate from path to restore prior state for neighboring branches.

---

## 2. Identification Signals ("When to Use")

- **Exhaustive Generation**: Generate all subsets ($2^N$), all permutations ($N!$), or all combinations ($\binom{N}{K}$).
- **Small Input Constraints**: $N \le 20$ indicates an exponential $O(2^N)$ algorithm; $N \le 10$ indicates a factorial $O(N!)$ algorithm.
- **Constraint Satisfaction**: Sudoku Solver, N-Queens, Word Search on a grid.
- **Duplicate Elements**: Requires sorting first to prune identical sibling decision branches.

---

## 3. Algorithmic Template / Pseudocode

```text
function BACKTRACK(state, start_index, path, result):
    if GOAL_REACHED(state):
        result.append(copy_of(path))
        return

    for i from start_index to total_candidates - 1:
        if not IS_VALID(candidate[i]):
            continue // Prune invalid branch

        path.push(candidate[i])              // 1. Choose
        BACKTRACK(state, i + 1, path, result) // 2. Explore
        path.pop()                           // 3. Un-choose
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import List

# --- Variant A: Subsets (Power Set 2^N) ---
def subsets(nums: List[int]) -> List[List[int]]:
    res = []
    path = []

    def backtrack(start: int) -> None:
        # Every node in the decision tree is a valid subset
        res.append(list(path))

        for i in range(start, len(nums)):
            path.append(nums[i])      # Choose
            backtrack(i + 1)          # Explore
            path.pop()                # Un-choose

    backtrack(0)
    return res

# --- Variant B: Subsets II (With Duplicates) ---
def subsets_with_dup(nums: List[int]) -> List[List[int]]:
    nums.sort()  # Sorting brings identical values together
    res = []
    path = []

    def backtrack(start: int) -> None:
        res.append(list(path))

        for i in range(start, len(nums)):
            # Prune duplicate siblings at the same depth
            if i > start and nums[i] == nums[i - 1]:
                continue

            path.append(nums[i])
            backtrack(i + 1)
            path.pop()

    backtrack(0)
    return res

# --- Variant C: Permutations (N! Orders) ---
def permute(nums: List[int]) -> List[List[int]]:
    res = []
    path = []
    used = [False] * len(nums)

    def backtrack() -> None:
        if len(path) == len(nums):
            res.append(list(path))
            return

        for i in range(len(nums)):
            if not used[i]:
                used[i] = True
                path.append(nums[i])
                backtrack()
                path.pop()
                used[i] = False

    backtrack()
    return res

# --- Variant D: Combination Sum (Reuse Allowed) ---
def combination_sum(candidates: List[int], target: int) -> List[List[int]]:
    res = []
    path = []

    def backtrack(start: int, remain: int) -> None:
        if remain == 0:
            res.append(list(path))
            return
        if remain < 0:
            return  # Prune branch exceeding target

        for i in range(start, len(candidates)):
            path.append(candidates[i])
            # Reuse allowed -> stay at index i instead of i + 1
            backtrack(i, remain - candidates[i])
            path.pop()

    backtrack(0, target)
    return res
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `subsets`:
1. `res.append(list(path))`:
   * **Crucial Snapshot**: Appends `list(path)` (a shallow copy) rather than `path`. If you append `path` directly, you store a reference to the mutable list, which becomes empty once backtracking completes!
2. `for i in range(start, len(nums)):`:
   * `start` index prevents re-using previous elements or generating duplicate permutations.
3. `path.append(nums[i]); backtrack(i + 1); path.pop()`:
   * **State Transition**: Adds `nums[i]`, traverses deeper with next index `i + 1`, and then removes `nums[i]` to restore the state for the next candidate.

### Breakdown of Duplicate Pruning (`subsets_with_dup`):
1. `nums.sort()`:
   * Groups duplicates together.
2. `if i > start and nums[i] == nums[i - 1]: continue`:
   * **The Pruning Invariant**: `i > start` ensures this is a *sibling* decision at the same recursion depth, not a parent-to-child descent. If the previous sibling used the same value, exploring this branch would create identical duplicate subsets.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity:
- **`subsets`**: A set of $N$ elements has exactly $2^N$ subsets. For each subset, copying the path takes up to $O(N)$ time. Total operations: $O(N \cdot 2^N)$. This is optimal because every subset must be generated and returned.
- **`permute`**: $N!$ distinct permutations exist. Copying each takes $O(N)$. Total runtime: $O(N \cdot N!)$.
- **`combination_sum`**: $O(K \cdot 2^T)$ where $T = \text{target} / \min(\text{candidates})$.

### Space Complexity: $O(N)$ (Optimal)
- Auxiliary space is bounded by the recursion stack depth and the current `path` length, which is at most $N$.
- Zero extra memory beyond the recursion stack and final output storage.

---

## 7. Key Invariants & Common Pitfalls

1. **Appending References Instead of Copies**:
   * Writing `res.append(path)` stores references. When `path.pop()` empties the list, `res` will contain only empty lists `[[], [], []]`. Always write `res.append(list(path))` or `res.append(path[:])`.
2. **Permutations vs Subsets**:
   * Subsets and combinations use a `start` index to only pick elements to the right.
   * Permutations always loop from `0` to `N - 1` and use a `used` boolean array (or bitmask) to avoid re-picking the same index.

---

## 8. Canonical Problem Walkthrough

### Word Search (LeetCode #79)
* **Problem**: Determine if a word exists in a 2D grid of characters using adjacent cells without reusing cells.
* **In-Place Grid Marking**: Mark visited cell with `'#'` and restore original character on backtrack!

```python
def exist(board: List[List[str]], word: str) -> bool:
    rows, cols = len(board), len(board[0])
    
    def backtrack(r: int, c: int, idx: int) -> bool:
        if idx == len(word):
            return True
        if r < 0 or r >= rows or c < 0 or c >= cols or board[r][c] != word[idx]:
            return False
            
        temp = board[r][c]
        board[r][c] = '#'  # Choose: mark visited
        
        # Explore 4 cardinal directions
        found = (backtrack(r + 1, c, idx + 1) or
                 backtrack(r - 1, c, idx + 1) or
                 backtrack(r, c + 1, idx + 1) or
                 backtrack(r, c - 1, idx + 1))
                 
        board[r][c] = temp  # Un-choose: restore original character
        return found

    for r in range(rows):
        for c in range(cols):
            if backtrack(r, c, 0):
                return True
    return False
```
* **Optimality**: Operates in $O(R \cdot C \cdot 3^L)$ time (where $L$ is word length) with $O(L)$ stack space.
