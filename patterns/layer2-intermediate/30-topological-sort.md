# Pattern 30: Topological Sort

> **Layer**: 2 (Intermediate High-Yield)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Course Schedule](https://leetcode.com/problems/course-schedule/), [Course Schedule II](https://leetcode.com/problems/course-schedule-ii/), [Alien Dictionary](https://leetcode.com/problems/alien-dictionary/), [Minimum Height Trees](https://leetcode.com/problems/minimum-height-trees/)

---

## 1. Mental Model & Core Concept

A Topological Sort linearly orders the vertices of a **Directed Acyclic Graph (DAG)** such that for every directed edge $u \to v$, vertex $u$ appears before vertex $v$ in the ordering.

```text
Dependency Resolution (Course Prerequisites):
(Course 0) ──► (Course 1) ──► (Course 3)
                   ▲
(Course 2) ────────┘

In-degree: count of incoming edges
- Course 0: in-degree 0 (Ready to take!)
- Course 2: in-degree 0 (Ready to take!)
- Course 1: in-degree 2 (Blocked until 0 and 2 complete)

Kahn's Algorithm:
1. Enqueue all vertices with in-degree == 0.
2. Pop vertex, append to order, decrement in-degree of all neighbors.
3. If neighbor in-degree reaches 0, enqueue it!
```

If the number of processed nodes at termination is less than $V$, the graph contains a **cycle** (impossible to order)!

---

## 2. Identification Signals ("When to Use")

- **Dependency / Prerequisite Resolution**: Build systems, course schedules, task compilation orders.
- **Cycle Detection in Directed Graphs**: If topological sort cannot process all $V$ nodes, a directed cycle exists.
- **Lexicographical Language Ordering**: Deducing alphabet order from a sorted dictionary of alien words.

---

## 3. Algorithmic Template / Pseudocode

```text
function KAHN_TOPOLOGICAL_SORT(num_nodes, edges):
    adj = list of empty lists of size num_nodes
    in_degree = array of size num_nodes filled with 0

    for u, v in edges: // u -> v
        adj[u].append(v)
        in_degree[v] += 1

    queue = deque([node for node in range(num_nodes) if in_degree[node] == 0])
    order = []

    while queue is not empty:
        curr = queue.pop_front()
        order.append(curr)

        for neighbor in adj[curr]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.push_back(neighbor)

    return order if length(order) == num_nodes else [] // Empty if cycle detected
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from collections import deque, defaultdict
from typing import List

# --- Variant A: Course Schedule II (Return Valid Order or Empty if Cycle) ---
def find_order(num_courses: int, prerequisites: List[List[int]]) -> List[int]:
    adj = defaultdict(list)
    in_degree = [0] * num_courses

    # [course, prereq] means prereq -> course
    for course, prereq in prerequisites:
        adj[prereq].append(course)
        in_degree[course] += 1

    # Start with courses that have no prerequisites
    queue = deque([i for i in range(num_courses) if in_degree[i] == 0])
    order = []

    while queue:
        curr = queue.popleft()
        order.append(curr)

        for next_course in adj[curr]:
            in_degree[next_course] -= 1
            if in_degree[next_course] == 0:
                queue.append(next_course)

    return order if len(order) == num_courses else []

# --- Variant B: Cycle Detection via DFS 3-Coloring ---
def can_finish(num_courses: int, prerequisites: List[List[int]]) -> bool:
    adj = defaultdict(list)
    for course, prereq in prerequisites:
        adj[prereq].append(course)

    # 0 = unvisited, 1 = visiting (on current path), 2 = visited & safe
    state = [0] * num_courses

    def has_cycle(node: int) -> bool:
        if state[node] == 1:
            return True   # Back-edge detected! Cycle exists!
        if state[node] == 2:
            return False  # Already confirmed acyclic

        state[node] = 1   # Mark visiting
        for neighbor in adj[node]:
            if has_cycle(neighbor):
                return True
        state[node] = 2   # Mark processed
        return False

    for i in range(num_courses):
        if state[i] == 0:
            if has_cycle(i):
                return False

    return True
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `find_order` (Kahn's Algorithm):
1. `in_degree = [0] * num_courses`:
   * Tracks unresolved dependencies for each course.
2. `adj[prereq].append(course); in_degree[course] += 1`:
   * Constructs directed adjacency list from prereq to course.
3. `queue = deque([i for i in range(num_courses) if in_degree[i] == 0])`:
   * Collects all starting nodes with zero blocking dependencies.
4. `in_degree[next_course] -= 1`:
   * Simulates completing course `curr` by satisfying one prerequisite of `next_course`.
5. `if in_degree[next_course] == 0: queue.append(next_course)`:
   * When all prerequisites of `next_course` are fulfilled, it is unlocked and enters the queue.
6. `return order if len(order) == num_courses else []`:
   * If a cycle exists (e.g. $A \to B \to A$), their in-degrees never reach 0, preventing them from entering `order`. If `len(order) < num_courses`, a deadlock cycle exists.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(V + E)$ (Optimal)
- **Proof**:
  - Graph construction inspects all $E$ edges once $\implies O(E)$.
  - In-degree 0 initialization inspects all $V$ vertices $\implies O(V)$.
  - Every vertex enters and leaves the queue at most once.
  - As each vertex is dequeued, its outgoing edges are traversed: $\sum_{u} \text{deg}(u) = E$.
  - Total runtime: $O(V + E)$.
  - Any topological ordering must inspect every node and edge to confirm dependencies, matching the $\Omega(V + E)$ lower bound.

### Space Complexity: $O(V + E)$ (Optimal)
- Adjacency list requires $O(V + E)$ memory; `in_degree` array and queue require $O(V)$ auxiliary space.

---

## 7. Key Invariants & Common Pitfalls

1. **Reversed Edge Direction**:
   * *Trap*: The problem statement says `[a, b]` means to take course `a` you must have taken `b`. The directed edge points **from `b` to `a`** (`b -> a`), NOT `a -> b`. Reversing this breaks the dependency graph.
2. **Disconnected Components**:
   * Always seed the queue with *all* nodes with in-degree 0, not just node 0.

---

## 8. Canonical Problem Walkthrough

### Minimum Height Trees (LeetCode #310)
* **Problem**: Find the roots of trees that have the minimum possible height.
* **Topological Trimming (Leaf Pruning)**: Trim leaves (degree == 1) layer by layer until $\le 2$ centroid nodes remain!

```python
def find_min_height_trees(n: int, edges: List[List[int]]) -> List[int]:
    if n <= 2:
        return [i for i in range(n)]
        
    adj = defaultdict(set)
    for u, v in edges:
        adj[u].add(v)
        adj[v].add(u)
        
    leaves = deque([i for i in range(n) if len(adj[i]) == 1])
    remaining_nodes = n
    
    while remaining_nodes > 2:
        leaf_count = len(leaves)
        remaining_nodes -= leaf_count
        for _ in range(leaf_count):
            leaf = leaves.popleft()
            neighbor = adj[leaf].pop()
            adj[neighbor].remove(leaf)
            if len(adj[neighbor]) == 1:
                leaves.append(neighbor)
                
    return list(leaves)
```
* **Optimality**: Operates in $O(V)$ time and $O(V)$ space.
