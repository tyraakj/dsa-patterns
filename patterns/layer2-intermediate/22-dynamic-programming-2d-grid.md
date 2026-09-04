# Pattern 22: Dynamic Programming - 2D / Grid

> **Layer**: 2 (Intermediate High-Yield)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Unique Paths](https://leetcode.com/problems/unique-paths/), [Unique Paths II](https://leetcode.com/problems/unique-paths-ii/), [Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/), [Edit Distance](https://leetcode.com/problems/edit-distance/), [Minimum Path Sum](https://leetcode.com/problems/minimum-path-sum/)

---

## 1. Mental Model & Core Concept

2D Dynamic Programming models problems where decisions depend on **two independent state dimensions** (e.g. `(row, col)` in a grid, or prefixes of two strings `(text1[:i], text2[:j])`).

```text
Grid Path State Transition:
To arrive at cell (r, c), you can only come from (r - 1, c) [Above] or (r, c - 1) [Left]:

       (r - 1, c) [Above]
           ▼
(r, c - 1) ──► (r, c)
 [Left]

dp[r][c] = dp[r - 1][c] + dp[r][c - 1]
```

Space Optimization Insight:
Because computing `dp[r]` only requires values from the current row and the row immediately above `dp[r - 1]`, a 2D table of size $M \times N$ can almost always be compressed into a **single 1D rolling array** of size $N$, dropping auxiliary space from $O(M \times N)$ down to $O(N)$!

---

## 2. Identification Signals ("When to Use")

- **Grid Navigation**: Minimum path sum, unique paths, dungeon game (moving only down and right).
- **Two-String Alignment / Comparison**: Longest Common Subsequence (LCS), Edit Distance, Interleaving String.
- **2-Parameter State Spaces**: Stock trading with $K$ transactions, buy/sell with cooldown.

---

## 3. Algorithmic Template / Pseudocode

```text
function DP_GRID_PATHS(m, n):
    // 1D space-optimized row array
    row = array of size n filled with 1

    for r from 1 to m - 1:
        new_row = array of size n filled with 1
        for c from 1 to n - 1:
            new_row[c] = new_row[c - 1] + row[c]
        row = new_row

    return row[n - 1]
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import List

# --- Variant A: Unique Paths (Space Optimized O(N)) ---
def unique_paths(m: int, n: int) -> int:
    row = [1] * n

    for _ in range(m - 1):
        new_row = [1] * n
        for c in range(1, n):
            # new_row[c - 1] is Left, row[c] is Above
            new_row[c] = new_row[c - 1] + row[c]
        row = new_row

    return row[-1]

# --- Variant B: Minimum Path Sum in Grid ---
def min_path_sum(grid: List[List[int]]) -> int:
    rows, cols = len(grid), len(grid[0])
    # dp[c] represents minimum cost to reach current row, column c
    dp = [float('inf')] * cols
    dp[0] = 0

    for r in range(rows):
        # Update first column: can only come from above
        dp[0] += grid[r][0]
        for c in range(1, cols):
            # min(from above, from left) + grid value
            dp[c] = min(dp[c], dp[c - 1]) + grid[r][c]

    return int(dp[-1])

# --- Variant C: Longest Common Subsequence (LCS) ---
def longest_common_subsequence(text1: str, text2: str) -> int:
    # Ensure text2 is the shorter string for O(min(M, N)) space
    if len(text1) < len(text2):
        text1, text2 = text2, text1

    m, n = len(text1), len(text2)
    prev = [0] * (n + 1)

    for i in range(1, m + 1):
        curr = [0] * (n + 1)
        for j in range(1, n + 1):
            if text1[i - 1] == text2[j - 1]:
                curr[j] = 1 + prev[j - 1]
            else:
                curr[j] = max(prev[j], curr[j - 1])
        prev = curr

    return prev[n]

# --- Variant D: Edit Distance (Levenshtein) ---
def min_distance(word1: str, word2: str) -> int:
    m, n = len(word1), len(word2)
    # dp[i][j] is min operations to convert word1[:i] to word2[:j]
    dp = [[0] * (n + 1) for _ in range(m + 1)]

    # Base cases: converting to/from empty string
    for i in range(m + 1):
        dp[i][0] = i
    for j in range(n + 1):
        dp[0][j] = j

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if word1[i - 1] == word2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1]
            else:
                dp[i][j] = 1 + min(
                    dp[i - 1][j],      # Deletion
                    dp[i][j - 1],      # Insertion
                    dp[i - 1][j - 1]   # Replacement
                )

    return dp[m][n]
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `longest_common_subsequence`:
1. `prev = [0] * (n + 1)`:
   * Represents the previous row $i - 1$ in the DP matrix. Base case: empty prefix has LCS 0.
2. `for i in range(1, m + 1): curr = [0] * (n + 1)`:
   * Allocates active row for character `text1[i - 1]`.
3. `if text1[i - 1] == text2[j - 1]: curr[j] = 1 + prev[j - 1]`:
   * If current characters match, they extend the longest common subsequence of the prefixes `text1[:i-1]` and `text2[:j-1]`.
4. `else: curr[j] = max(prev[j], curr[j - 1])`:
   * If they mismatch, take the best result between dropping `text1[i-1]` (`prev[j]`) or dropping `text2[j-1]` (`curr[j-1]`).
5. `prev = curr`:
   * Rolls the active row into the previous row buffer in $O(1)$ pointer assignment.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(M \times N)$ (Optimal)
- **Proof**:
  - The DP matrix contains $(M + 1) \times (N + 1)$ states.
  - Computing each cell involves a constant number of operations: 1 character comparison, 1 addition, and 1 `max`/`min` ($O(1)$ work per cell).
  - Total operations: $(M + 1)(N + 1) \implies O(M \times N)$.
  - Any general string alignment algorithm evaluating arbitrary string pairs must consider interactions between characters, matching the information lower bound.

### Space Complexity: $O(\min(M, N))$ (Optimal)
- By compressing the 2D matrix into a 1D rolling array and orienting the shorter string as the columns, memory is reduced from $O(M \times N)$ down to $O(\min(M, N))$, achieving minimal memory footprint.

---

## 7. Key Invariants & Common Pitfalls

1. **Off-by-One in String DP Indexing**:
   * DP table is $(M + 1) \times (N + 1)$ with 1-based indexing, but strings `word1` and `word2` are 0-based. Character at DP step `i` is `word1[i - 1]`.
2. **Base Cases for Edit Distance**:
   * Converting a string of length $i$ to an empty string requires $i$ deletions: `dp[i][0] = i`.
   * Converting an empty string to a string of length $j$ requires $j$ insertions: `dp[0][j] = j`.

---

## 8. Canonical Problem Walkthrough

### Unique Paths II (LeetCode #63 - With Obstacles)
* **Problem**: Grid navigation where cell `obstacleGrid[r][c] == 1` cannot be walked through.
* **Invariant**: If cell is an obstacle, set its paths to 0!

```python
def unique_paths_with_obstacles(obstacle_grid: List[List[int]]) -> int:
    cols = len(obstacle_grid[0])
    dp = [0] * cols
    dp[0] = 1 if obstacle_grid[0][0] == 0 else 0

    for r, row in enumerate(obstacle_grid):
        for c in range(cols):
            if row[c] == 1:
                dp[c] = 0  # Obstacle, no paths possible
            elif c > 0:
                dp[c] += dp[c - 1]

    return dp[-1]
```
* **Optimality**: Operates in $O(M \times N)$ time and $O(N)$ space.
