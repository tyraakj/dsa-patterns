# Pattern 16: Graph DFS

> **Layer**: 2 (Intermediate High-Yield)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Number of Provinces](https://leetcode.com/problems/number-of-provinces/), [Clone Graph](https://leetcode.com/problems/clone-graph/), [Is Graph Bipartite?](https://leetcode.com/problems/is-graph-bipartite/), [Course Schedule](https://leetcode.com/problems/course-schedule/), [All Paths From Source to Target](https://leetcode.com/problems/all-paths-from-source-to-target/)

---

## 1. Mental Model & Core Concept

Graph Depth-First Search explores connected components and paths in arbitrary graphs. Unlike trees, general graphs may contain **cycles**, which requires maintaining a `visited` set to prevent infinite recursion loops.

```text
Graph DFS Traversal with Cycle Detection:
State Colors:
  0 = Unvisited (White)
  1 = Visiting / Currently on Call Stack (Gray) ──► Back-edge to 1 implies CYCLE!
  2 = Fully Visited & Closed (Black)
```

Core applications:
1. **Connected Components**: Counting clusters of vertices connected directly or indirectly.
2. **Cycle Detection (Directed Graphs)**: Three-state coloring (visiting vs visited).
3. **Bipartite Testing (2-Coloring)**: Alternating colors across adjacent edges.

---

## 2. Identification Signals ("When to Use")

- **Connected Components**: *"Count number of provinces / connected friend circles"*.
- **Path Exploration**: *"Find all paths from node 0 to node N - 1"*.
- **Cycle Detection**: *"Can all courses be finished given prerequisite constraints"*.
- **Graph Coloring**: *"Determine if graph can be partitioned into two sets where no two adjacent vertices share a set"*.

---

## 3. Algorithmic Template / Pseudocode

```text
function GRAPH_DFS(adj_list):
    visited = empty set
    
    function dfs(node):
        visited.add(node)
        for neighbor in adj_list[node]:
            if neighbor not in visited:
                dfs(neighbor)
                
    components = 0
    for node in all_nodes:
        if node not in visited:
            components += 1
            dfs(node)
            
    return components
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from collections import defaultdict
from typing import List, Dict, Optional

class Node:
    def __init__(self, val=0, neighbors=None):
        self.val = val
        self.neighbors = neighbors if neighbors is not None else []

# --- Variant A: Connected Components (Number of Provinces) ---
def find_circle_num(is_connected: List[List[int]]) -> int:
    n = len(is_connected)
    visited = set()
    provinces = 0

    def dfs(city: int) -> None:
        for neighbor in range(n):
            if is_connected[city][neighbor] == 1 and neighbor not in visited:
                visited.add(neighbor)
                dfs(neighbor)

    for i in range(n):
        if i not in visited:
            provinces += 1
            visited.add(i)
            dfs(i)

    return provinces

# --- Variant B: Clone Graph (DFS with Hash Map Memoization) ---
def clone_graph(node: Optional[Node]) -> Optional[Node]:
    if not node:
        return None

    clones: Dict[Node, Node] = {}

    def dfs(curr: Node) -> Node:
        if curr in clones:
            return clones[curr]

        copy = Node(curr.val)
        clones[curr] = copy
        for neighbor in curr.neighbors:
            copy.neighbors.append(dfs(neighbor))
        return copy

    return dfs(node)

# --- Variant C: Bipartite Graph Verification (2-Coloring) ---
def is_bipartite(graph: List[List[int]]) -> bool:
    # 0 = uncolored, 1 = red, -1 = blue
    color = {}

    def dfs(node: int, c: int) -> bool:
        color[node] = c
        for neighbor in graph[node]:
            if neighbor in color:
                if color[neighbor] == c:
                    return False  # Adjacent nodes share same color
            else:
                if not dfs(neighbor, -c):
                    return False
        return True

    for node in range(len(graph)):
        if node not in color:
            if not dfs(node, 1):
                return False

    return True
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `find_circle_num`:
1. `visited = set()`:
   * Prevents reprocessing cities that belong to an already counted province.
2. `def dfs(city: int) -> None:`:
   * Explores all cities directly or transitively reachable from `city`.
3. `if is_connected[city][neighbor] == 1 and neighbor not in visited:`:
   * Verifies edge existence and unvisited state before recursing.
4. `for i in range(n): if i not in visited: provinces += 1; dfs(i)`:
   * Iterates through all cities. When an unvisited city is discovered, it must belong to a brand-new, isolated component. Increment count and flood all reachable nodes.

### Breakdown of `clone_graph`:
1. `clones = {}`:
   * Maps original nodes to their cloned counterparts to prevent duplicate allocations and cycle deadlocks.
2. `if curr in clones: return clones[curr]`:
   * Returns pre-existing clone when cycles circle back to already visited nodes.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(V + E)$ (Optimal)
- **Proof**:
  - In adjacency list representation, `visited.add(node)` ensures every vertex $V$ is visited at most once.
  - Across all vertices, every outgoing directed edge (or twice for undirected edges) is examined exactly once: $\sum_{u} \text{deg}(u) = 2E$.
  - Total time complexity: $O(V + E)$.
  - For adjacency matrices (`is_connected`), inspecting all matrix cells takes $O(V^2)$ time.
  - Optimal since every edge and vertex must be explored to determine connectivity.

### Space Complexity: $O(V)$ (Optimal)
- `visited` set holds at most $V$ elements.
- Recursion call stack depth is at most $V$ in a linear path graph. Total space $= O(V)$.

---

## 7. Key Invariants & Common Pitfalls

1. **Cycle Infinite Recursion**:
   * Graphs can have cycles. Omitting `visited` set leads immediately to `RecursionError: maximum recursion depth exceeded`.
2. **Disconnected Graphs**:
   * Calling `dfs(0)` alone only visits the component containing node 0. You **must** iterate over all vertices from $0$ to $V-1$ in the outer loop to cover disconnected components.

---

## 8. Canonical Problem Walkthrough

### All Paths From Source to Target (LeetCode #797)
* **Problem**: Directed Acyclic Graph (DAG) with $n$ nodes. Find all paths from `0` to `n - 1`.
* **Backtracking DFS**:
```python
def all_paths_source_target(graph: List[List[int]]) -> List[List[int]]:
    target = len(graph) - 1
    res = []
    path = [0]
    
    def dfs(node: int) -> None:
        if node == target:
            res.append(list(path))
            return
        for neighbor in graph[node]:
            path.append(neighbor)
            dfs(neighbor)
            path.pop()
            
    dfs(0)
    return res
```
* **Optimality**: Operates in $O(2^V \cdot V)$ time and $O(V)$ stack space, which is optimal for returning all possible exponential paths in a DAG.
