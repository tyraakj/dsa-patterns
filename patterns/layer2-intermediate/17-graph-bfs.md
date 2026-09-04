# Pattern 17: Graph BFS

> **Layer**: 2 (Intermediate High-Yield)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Word Ladder](https://leetcode.com/problems/word-ladder/), [Rotting Oranges](https://leetcode.com/problems/rotting-oranges/), [Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/), [Open the Lock](https://leetcode.com/problems/open-the-lock/)

---

## 1. Mental Model & Core Concept

Graph Breadth-First Search (BFS) explores all neighbors of the current frontier before moving to the next level of depth. 

```text
Shortest Path Invariant:
In any graph where all edges have identical (unweighted) cost = 1,
the FIRST time BFS reaches target T, the path length is GUARANTEED
to be the strictly shortest path possible!
```

Two major extensions:
1. **Single-Source Shortest Path**: Finding the minimum operations / steps to transform state $A$ into state $B$ (e.g. Word Ladder, Open the Lock).
2. **Multi-Source BFS**: Seeding the initial queue with multiple starting points simultaneously (e.g. all rotten oranges, all ocean boundaries) to simulate simultaneous wave propagation.

---

## 2. Identification Signals ("When to Use")

- **"Shortest path in unweighted graph"**: Minimum hops, minimum edge count.
- **"Minimum number of mutations / turns"**: Transforming a start string into a target string where each step changes 1 character.
- **Simultaneous Wave Spread**: Multiple infection/propagation points expanding at 1 unit per minute.

---

## 3. Algorithmic Template / Pseudocode

```text
function GRAPH_BFS_SHORTEST_PATH(start_node, target_node, adj_list):
    queue = deque([(start_node, 0)]) // (node, distance)
    visited = set([start_node])
    
    while queue is not empty:
        curr, dist = queue.pop_front()
        if curr == target_node:
            return dist
            
        for neighbor in adj_list[curr]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.push_back((neighbor, dist + 1))
                
    return -1 // Target unreachable
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from collections import deque
from typing import List, Set

# --- Variant A: Multi-Source BFS (Rotting Oranges) ---
def oranges_rotting(grid: List[List[int]]) -> int:
    rows, cols = len(grid), len(grid[0])
    queue = deque()
    fresh_count = 0

    # Step 1: Collect all initial rotten oranges (sources) and count fresh ones
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 2:
                queue.append((r, c))
            elif grid[r][c] == 1:
                fresh_count += 1

    if fresh_count == 0:
        return 0

    minutes = 0
    directions = [(1, 0), (-1, 0), (0, 1), (0, -1)]

    # Step 2: Multi-source BFS wave propagation
    while queue and fresh_count > 0:
        minutes += 1
        for _ in range(len(queue)):
            r, c = queue.popleft()

            for dr, dc in directions:
                nr, nc = r + dr, c + dc
                if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] == 1:
                    grid[nr][nc] = 2  # Rot the fresh orange
                    fresh_count -= 1
                    queue.append((nr, nc))

    return minutes if fresh_count == 0 else -1

# --- Variant B: State Space Transformation (Word Ladder) ---
def ladder_length(begin_word: str, end_word: str, word_list: List[str]) -> int:
    word_set = set(word_list)
    if end_word not in word_set:
        return 0

    queue = deque([(begin_word, 1)])
    visited = {begin_word}

    while queue:
        word, length = queue.popleft()
        if word == end_word:
            return length

        # Generate all 1-letter mutation neighbors
        for i in range(len(word)):
            for char_code in range(ord('a'), ord('z') + 1):
                char = chr(char_code)
                if char != word[i]:
                    next_word = word[:i] + char + word[i + 1:]
                    if next_word in word_set and next_word not in visited:
                        visited.add(next_word)
                        queue.append((next_word, length + 1))

    return 0
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `oranges_rotting`:
1. `queue = deque(); fresh_count = 0`:
   * Queue holds coordinates of all infected cells; `fresh_count` tracks targets remaining.
2. `if grid[r][c] == 2: queue.append((r, c))`:
   * **Multi-Source Seeding**: All initially rotten oranges are added to the queue *before* the while loop starts. This guarantees they expand synchronously minute by minute.
3. `while queue and fresh_count > 0:`:
   * Loop continues as long as there are rotten oranges and fresh oranges remain.
4. `for _ in range(len(queue)):`:
   * Processes all oranges that rotted in the current minute as one single time step.
5. `grid[nr][nc] = 2; fresh_count -= 1`:
   * In-place mutation serves as the `visited` set, eliminating extra memory allocations.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(V + E)$ (Optimal)
- In `oranges_rotting`: $V = R \times C$. Each cell is added to the queue at most once and has at most 4 edges ($E \le 4V$). Total operations: $O(R \times C)$.
- In `Word Ladder`: $N$ is number of words, $L$ is word length. Generating mutations takes $26 \times L \times L = O(L^2)$ per word. Searching across the dictionary takes $O(N \cdot L^2)$.
- Any unweighted shortest path search must explore candidate paths level-by-level, making BFS optimal.

### Space Complexity: $O(V)$ (Optimal)
- The queue and visited set hold at most $V$ vertices in the worst case $\implies O(V)$ space.

---

## 7. Key Invariants & Common Pitfalls

1. **Mark Visited Upon Enqueueing, NOT Dequeueing**:
   * *Critical Trap*: If you only add a node to `visited` when it is popped (`popleft()`), adjacent nodes will repeatedly enqueue the same neighbor multiple times, causing exponential queue explosions and Time Limit Exceeded (TLE).
   * *Rule*: Always mark `visited.add(neighbor)` **immediately** when adding to the queue.

---

## 8. Canonical Problem Walkthrough

### Open the Lock (LeetCode #752)
* **Problem**: 4 wheels with digits 0-9. Find minimum turns from `"0000"` to `target` avoiding `deadends`.
* **State Space**: 10,000 combinations. Each has 8 neighbors (turning each of 4 wheels forward or backward).

```python
def open_lock(deadends: List[str], target: str) -> int:
    dead = set(deadends)
    if "0000" in dead:
        return -1
    queue = deque([("0000", 0)])
    visited = {"0000"}
    
    while queue:
        combo, steps = queue.popleft()
        if combo == target:
            return steps
        for i in range(4):
            digit = int(combo[i])
            for delta in (-1, 1):
                next_d = (digit + delta) % 10
                neighbor = combo[:i] + str(next_d) + combo[i+1:]
                if neighbor not in dead and neighbor not in visited:
                    visited.add(neighbor)
                    queue.append((neighbor, steps + 1))
    return -1
```
* **Optimality**: Operates in $O(10^4 \cdot 8) = O(1)$ bounded constant time.
