# Pattern 36: Simulation / Array Rearrangement

> **Layer**: 3 (Advanced & Specialized)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Spiral Matrix](https://leetcode.com/problems/spiral-matrix/), [Rotate Image](https://leetcode.com/problems/rotate-image/), [Game of Life](https://leetcode.com/problems/game-of-life/), [Diagonal Traverse](https://leetcode.com/problems/diagonal-traverse/)

---

## 1. Mental Model & Core Concept

Simulation faithfully executes a defined physical or mechanical rule-set step-by-step. The principal algorithmic challenge is managing **state transitions and boundary boundaries without allocating auxiliary memory**.

```text
Spiral Matrix Boundary Squeeze:
top     ──► [ 1 , 2 , 3 ]
            [ 4 , 5 , 6 ]
bottom  ──► [ 7 , 8 , 9 ]
              ▲       ▲
             left   right

1. Traverse Left -> Right across top;   top += 1
2. Traverse Top -> Bottom along right;  right -= 1
3. Traverse Right -> Left across bottom; bottom -= 1
4. Traverse Bottom -> Top along left;   left += 1
```

In-Place State Encoding Trick:
In cellular simulations like *Game of Life*, cells transition simultaneously ($old \to new$). Instead of allocating a 2nd grid, encode both old and new states in the bits of each integer:
- `00`: Dead $\to$ Dead (`0`)
- `01`: Live $\to$ Dead (`1`)
- `10`: Dead $\to$ Live (`2`)
- `11`: Live $\to$ Live (`3`)
Decode old state using `val & 1`; decode new state using `val >> 1`!

---

## 2. Identification Signals ("When to Use")

- **Geometric Traversals**: Spiral matrices, zigzag/diagonal sweeps.
- **In-Place Matrix Manipulations**: Rotating a grid 90 degrees clockwise without extra memory.
- **Cellular Automata**: Game of Life simultaneous state updates.

---

## 3. Algorithmic Template / Pseudocode

```text
function SPIRAL_ORDER(matrix):
    top = 0, bottom = rows - 1
    left = 0, right = cols - 1
    result = []

    while top <= bottom and left <= right:
        for c from left to right: result.append(matrix[top][c])
        top += 1
        for r from top to bottom: result.append(matrix[r][right])
        right -= 1

        if top <= bottom:
            for c from right down to left: result.append(matrix[bottom][c])
            bottom -= 1
        if left <= right:
            for r from bottom down to top: result.append(matrix[r][left])
            left += 1

    return result
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import List

# --- Variant A: Spiral Matrix (Boundary Squeeze) ---
def spiral_order(matrix: List[List[int]]) -> List[int]:
    res = []
    top, bottom = 0, len(matrix) - 1
    left, right = 0, len(matrix[0]) - 1

    while top <= bottom and left <= right:
        # Traverse right
        for c in range(left, right + 1):
            res.append(matrix[top][c])
        top += 1

        # Traverse down
        for r in range(top, bottom + 1):
            res.append(matrix[r][right])
        right -= 1

        # Guard against single row/col over-traversal
        if top <= bottom:
            for c in range(right, left - 1, -1):
                res.append(matrix[bottom][c])
            bottom -= 1

        if left <= right:
            for r in range(bottom, top - 1, -1):
                res.append(matrix[r][left])
            left += 1

    return res

# --- Variant B: Rotate Image 90 Degrees Clockwise In-Place ---
def rotate(matrix: List[List[int]]) -> None:
    n = len(matrix)
    # 1. Transpose matrix (swap across main diagonal)
    for r in range(n):
        for c in range(r + 1, n):
            matrix[r][c], matrix[c][r] = matrix[c][r], matrix[r][c]

    # 2. Reverse each row horizontally
    for r in range(n):
        matrix[r].reverse()

# --- Variant C: Game of Life (2-Bit In-Place State Encoding) ---
def game_of_life(board: List[List[int]]) -> None:
    rows, cols = len(board), len(board[0])

    for r in range(rows):
        for c in range(cols):
            # Count live neighbors using lowest bit (board[nr][nc] & 1)
            live_neighbors = 0
            for dr in (-1, 0, 1):
                for dc in (-1, 0, 1):
                    if dr == 0 and dc == 0:
                        continue
                    nr, nc = r + dr, c + dc
                    if 0 <= nr < rows and 0 <= nc < cols:
                        live_neighbors += board[nr][nc] & 1

            # Rule application:
            if board[r][c] == 1 and (live_neighbors in (2, 3)):
                board[r][c] = 3  # Was 1, will be 1 (0b11)
            elif board[r][c] == 0 and live_neighbors == 3:
                board[r][c] = 2  # Was 0, will be 1 (0b10)

    # Shift right to finalize new states
    for r in range(rows):
        for c in range(cols):
            board[r][c] >>= 1
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `spiral_order`:
1. `top, bottom = 0, len(matrix) - 1; left, right = 0, len(matrix[0]) - 1`:
   * Four directional boundary walls that constrict inward.
2. `if top <= bottom: ... if left <= right:`:
   * **The Non-Square Guard**: When handling non-square matrices (e.g. $1 \times 4$ or $3 \times 1$), `top` and `right` modify boundaries during the first half of the loop. These guard checks prevent re-traversing the same row or column in reverse.

### Breakdown of `rotate`:
1. `matrix[r][c], matrix[c][r] = matrix[c][r], matrix[r][c]`:
   * Transposes $(r, c) \to (c, r)$.
2. `matrix[r].reverse()`:
   * Reversing each row transforms $(c, r) \to (c, N - 1 - r)$, achieving a mathematically pure 90-degree clockwise rotation in-place without any trigonometry.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(R \times C)$ (Optimal)
- In `spiral_order`, every cell is entered into `res` exactly once.
- In `rotate`, transpose touches $\frac{N(N-1)}{2}$ pairs and reverse touches $\frac{N^2}{2}$ items $\implies O(N^2)$.
- In `game_of_life`, each cell checks 8 neighbors $\implies 8 \cdot (R \times C) \implies O(R \times C)$.
- Optimal because every element must be read and moved.

### Space Complexity: $O(1)$ (Optimal)
- In `rotate` and `game_of_life`, execution modifies grid bits strictly in-place $\implies O(1)$ space.

---

## 7. Key Invariants & Common Pitfalls

1. **Non-Square Boundary Guards**:
   * Forgetting `if top <= bottom:` before the backward bottom sweep leads to duplicate outputs on matrices with an odd number of rows.
2. **Reading Old State While Updating**:
   * In cellular automata, updating a neighbor directly corrupts subsequent neighbor evaluations. Bitmasking (`& 1`) preserves historical state during simulation.

---

## 8. Canonical Problem Walkthrough

### Diagonal Traverse (LeetCode #498)
* **Problem**: Return all elements of an $M \times N$ matrix in diagonal zigzag order.
* **Invariant**: Elements on the same diagonal share the exact same sum: `r + c = k`. Group elements by `r + c`, and reverse every other diagonal!
* **Optimality**: Operates in $O(M \times N)$ time and $O(M \times N)$ space.
