# Pattern 18: Matrix / Grid Traversal

> **Tier**: 2 (Intermediate High-Yield)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Number of Islands](https://leetcode.com/problems/number-of-islands/), [Max Area of Island](https://leetcode.com/problems/max-area-of-island/), [Surrounded Regions](https://leetcode.com/problems/surrounded-regions/), [Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/)

---

## 1. Mental Model & Core Concept

A 2D Grid is fundamentally an implicit graph where each cell `(r, c)` is a vertex, and its adjacent neighbors (4-directionally or 8-directionally) are connected by edges.

```text
Grid as Implicit Graph:
                 (r-1, c)   [Up]
                    ▲
(r, c-1) [Left] ◄─ (r, c) ─► (r, c+1) [Right]
                    ▼
                 (r+1, c)   [Down]

Boundary Guard Invariant:
0 <= r < ROWS and 0 <= c < COLS
```

Primary Techniques:
1. **Flood Fill (In-Place Mutation)**: Overwrite visited cells (e.g. change `'1'` to `'0'`) to eliminate the need for a separate $O(R \times C)$ `visited` set.
2. **Reverse Boundary Traversal**: For problems like *Pacific Atlantic Water Flow* or *Surrounded Regions*, start DFS/BFS from the outer perimeter inward rather than exploring every interior cell outward.

---

## 2. Identification Signals ("When to Use")

- **Connected Landmasses**: *"Count number of islands"*, *"Find maximum area of an island"*.
- **Boundary Enclosure**: *"Capture all regions surrounded by 'X'"*.
- **Water Flow / Terrain**: *"Cells from which water can flow to both oceans"*.

---

## 3. Algorithmic Template / Pseudocode

```text
function GRID_DFS(grid):
    rows, cols = length(grid), length(grid[0])
    
    function dfs(r, c):
        if r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] != TARGET:
            return
            
        grid[r][c] = VISITED_MARKER // In-place sink
        
        dfs(r + 1, c)
        dfs(r - 1, c)
        dfs(r, c + 1)
        dfs(r, c - 1)
        
    for r from 0 to rows - 1:
        for c from 0 to cols - 1:
            if grid[r][c] == TARGET:
                dfs(r, c)
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import List

# --- Variant A: Flood Fill (Number of Islands) ---
def num_islands(grid: List[List[str]]) -> int:
    if not grid:
        return 0

    rows, cols = len(grid), len(grid[0])
    count = 0

    def dfs(r: int, c: int) -> None:
        if r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] != '1':
            return

        grid[r][c] = '0'  # In-place sink island cell to prevent re-visit

        dfs(r + 1, c)
        dfs(r - 1, c)
        dfs(r, c + 1)
        dfs(r, c - 1)

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1':
                count += 1
                dfs(r, c)

    return count

# --- Variant B: Perimeter / Boundary DFS (Surrounded Regions) ---
def solve_surrounded_regions(board: List[List[str]]) -> None:
    if not board:
        return

    rows, cols = len(board), len(board[0])

    def dfs(r: int, c: int) -> None:
        if r < 0 or r >= rows or c < 0 or c >= cols or board[r][c] != 'O':
            return
        board[r][c] = 'T'  # Mark temporarily as connected to boundary
        dfs(r + 1, c)
        dfs(r - 1, c)
        dfs(r, c + 1)
        dfs(r, c - 1)

    # 1. Traverse all 4 borders
    for r in range(rows):
        dfs(r, 0)
        dfs(r, cols - 1)
    for c in range(cols):
        dfs(0, c)
        dfs(rows - 1, c)

    # 2. Capture surrounded 'O's and restore 'T's
    for r in range(rows):
        for c in range(cols):
            if board[r][c] == 'O':
                board[r][c] = 'X'  # Captured!
            elif board[r][c] == 'T':
                board[r][c] = 'O'  # Saved boundary-connected cell
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `num_islands`:
1. `if r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] != '1': return`:
   * Combines out-of-bounds checks and island criteria into a single defensive guard clause.
2. `grid[r][c] = '0'`:
   * **In-Place Sinking**: Overwriting the cell with `'0'` marks it as visited without allocating an $O(R \times C)$ set.
3. `dfs(r + 1, c); dfs(r - 1, c); dfs(r, c + 1); dfs(r, c - 1)`:
   * Expands in 4 cardinal directions to sink the entire contiguous landmass.
4. `for r in range(rows): for c in range(cols): if grid[r][c] == '1': count += 1; dfs(r, c)`:
   * Scans every cell. Finding a `'1'` guarantees a distinct island because previous DFS calls sunk all connected parts of prior islands.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(R \times C)$ (Optimal)
- **Proof**:
  - The outer nested loops visit each of the $R \times C$ cells once.
  - Each cell is sunk from `'1'` to `'0'` at most once during DFS.
  - No cell is visited by DFS more than 4 times (from its 4 neighbors).
  - Total operations $\le 5 \cdot (R \times C) \implies O(R \times C)$.
  - Optimal because every cell must be inspected to distinguish land from water.

### Space Complexity: $O(R \times C)$ (Optimal)
- In the worst case (the entire grid is land), the recursion call stack can reach $R \times C$ frames.
- By sinking cells in-place, auxiliary heap space is $O(1)$.

---

## 7. Key Invariants & Common Pitfalls

1. **Boundary Guard Order**:
   * *Critical*: In Python, write `0 <= r < rows and 0 <= c < cols` **before** indexing `grid[r][c]`. Reversing the order causes `IndexError`.
2. **Reverse Boundary Traversal Advantage**:
   * Trying to determine if an interior region is connected to a boundary requires complex tracking. Starting from the boundary and flooding inward guarantees only truly surrounded cells remain untouched.

---

## 8. Canonical Problem Walkthrough

### Pacific Atlantic Water Flow (LeetCode #417)
* **Problem**: Find grid coordinates where water can flow to both Pacific (top/left) and Atlantic (bottom/right).
* **Reverse Flow Insight**: Start from ocean boundaries and flow **uphill** (`next_height >= curr_height`). Coordinates reached by both ocean searches form the intersection!

```python
def pacific_atlantic(heights: List[List[int]]) -> List[List[int]]:
    rows, cols = len(heights), len(heights[0])
    pac = set()
    atl = set()

    def dfs(r: int, c: int, visited: set, prev_h: int) -> None:
        if (r < 0 or r >= rows or c < 0 or c >= cols or 
            (r, c) in visited or heights[r][c] < prev_h):
            return
        visited.add((r, c))
        for dr, dc in [(1, 0), (-1, 0), (0, 1), (0, -1)]:
            dfs(r + dr, c + dc, visited, heights[r][c])

    for c in range(cols):
        dfs(0, c, pac, heights[0][c])
        dfs(rows - 1, c, atl, heights[rows - 1][c])
    for r in range(rows):
        dfs(r, 0, pac, heights[r][0])
        dfs(r, cols - 1, atl, heights[r][cols - 1])

    return [[r, c] for r, c in pac & atl]
```
* **Optimality**: Runs in $O(R \times C)$ time and $O(R \times C)$ space.
