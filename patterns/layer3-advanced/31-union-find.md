# Pattern 31: Union-Find (Disjoint Set)

> **Layer**: 3 (Advanced & Specialized)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Number of Connected Components in an Undirected Graph](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/), [Redundant Connection](https://leetcode.com/problems/redundant-connection/), [Accounts Merge](https://leetcode.com/problems/accounts-merge/), [Number of Islands II](https://leetcode.com/problems/number-of-islands-ii/)

---

## 1. Mental Model & Core Concept

A Disjoint Set Union (DSU) maintains a collection of disjoint (non-overlapping) sets. It efficiently answers two primary queries:
1. **`find(x)`**: Which set/representative does element $x$ belong to?
2. **`union(x, y)`**: Merge the sets containing $x$ and $y$.

```text
DSU with Path Compression & Union by Rank:
Before Find(4):           After Find(4) (Path Compression):
      1                                1
     /                               / | \
    2                               2  3  4
   /                            (Every node points directly to root!)
  3
 /
4

Union by Rank:
Always attach the shallower tree beneath the root of the deeper tree!
```

Two Optimizations for Near-$O(1)$ Performance:
- **Path Compression**: During `find(x)`, point every visited node directly to the set representative root.
- **Union by Rank / Size**: Attach the smaller tree beneath the root of the larger tree to keep tree depth flat.

With both optimizations, operations run in **$O(\alpha(N))$** time, where $\alpha(N)$ is the Inverse Ackermann function ($\alpha(N) < 5$ for any $N \le 10^{80}$, effectively constant time $O(1)$).

---

## 2. Identification Signals ("When to Use")

- **Dynamic Connectivity**: Edges added one by one; check if two elements are connected dynamically.
- **Cycle Detection in Undirected Graphs**: If `find(u) == find(v)` before merging edge $(u, v)$, this edge forms a cycle!
- **Kruskal's Minimum Spanning Tree (MST)**: Greedy edge addition checking for cycles.

---

## 3. Algorithmic Template / Pseudocode

```text
class UnionFind:
    parent = [0, 1, 2, ..., n - 1]
    rank = [0] * n

    function find(i):
        if parent[i] != i:
            parent[i] = find(parent[i]) // Path compression
        return parent[i]

    function union(i, j):
        root_i = find(i)
        root_j = find(j)
        if root_i == root_j:
            return false // Already connected! (Cycle detected)

        // Union by rank
        if rank[root_i] < rank[root_j]:
            parent[root_i] = root_j
        elif rank[root_i] > rank[root_j]:
            parent[root_j] = root_i
        else:
            parent[root_j] = root_i
            rank[root_i] += 1
        return true
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import List

# --- Universal Production-Grade DSU Implementation ---
class UnionFind:
    def __init__(self, size: int):
        self.parent = list(range(size))
        self.rank = [1] * size
        self.components = size

    def find(self, x: int) -> int:
        if self.parent[x] != x:
            # Path compression
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]

    def union(self, x: int, y: int) -> bool:
        root_x = self.find(x)
        root_y = self.find(y)

        if root_x == root_y:
            return False  # Same set, union is redundant

        # Union by rank: attach smaller tree under larger
        if self.rank[root_x] < self.rank[root_y]:
            self.parent[root_x] = root_y
        elif self.rank[root_x] > self.rank[root_y]:
            self.parent[root_y] = root_x
        else:
            self.parent[root_y] = root_x
            self.rank[root_x] += 1

        self.components -= 1
        return True

# --- Variant A: Redundant Connection (Cycle Detection) ---
def find_redundant_connection(edges: List[List[int]]) -> List[int]:
    n = len(edges)
    uf = UnionFind(n + 1)

    for u, v in edges:
        # If u and v are already connected, this edge creates a cycle!
        if not uf.union(u, v):
            return [u, v]

    return []

# --- Variant B: Number of Connected Components ---
def count_components(n: int, edges: List[List[int]]) -> int:
    uf = UnionFind(n)
    for u, v in edges:
        uf.union(u, v)
    return uf.components
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `find` (Path Compression):
1. `if self.parent[x] != x: self.parent[x] = self.find(self.parent[x])`:
   * Recursively finds the absolute root of the set, and on the unwind phase, assigns `self.parent[x]` directly to the root. Future calls on $x$ or any of its descendants resolve in $O(1)$!
2. `return self.parent[x]`:
   * Returns the set representative.

### Breakdown of `union` (Union by Rank):
1. `root_x = self.find(x); root_y = self.find(y)`:
   * Finds representatives for both elements.
2. `if root_x == root_y: return False`:
   * **The Cycle Invariant**: If two nodes already share a common root, an alternative path already connects them. Adding edge $(x, y)$ forms a cycle!
3. `if self.rank[root_x] < self.rank[root_y]: self.parent[root_x] = root_y`:
   * Attaches the root with smaller depth beneath the root with greater depth, preventing tree height from growing.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(M \cdot \alpha(N))$ (Optimal)
- **Tarjan's Bound**: Combining Path Compression and Union by Rank yields an amortized time of $O(\alpha(N))$ per operation, where $\alpha$ is the Inverse Ackermann function.
- For all practical universe-scale values ($N < 10^{80}$), $\alpha(N) \le 4$.
- Across $M$ edge operations, total runtime is $O(M)$, matching the information-theoretic lower bound for dynamic connectivity.

### Space Complexity: $O(N)$ (Optimal)
- Stores `parent` and `rank` arrays of size $N \implies O(N)$ auxiliary memory.

---

## 7. Key Invariants & Common Pitfalls

1. **Union by Value vs Union by Root**:
   * *Critical Bug*: Setting `self.parent[x] = y` instead of `self.parent[root_x] = root_y` breaks set connectivity for all other elements in $x$'s set! Always attach roots to roots.
2. **Missing Path Compression**:
   * Without path compression, trees can degenerate into linked lists of depth $N$, causing find operations to degrade to $O(N)$.

---

## 8. Canonical Problem Walkthrough

### Accounts Merge (LeetCode #721)
* **Problem**: Merge user accounts that share at least one common email address.
* **DSU Mapping**: Map each email to a unique integer ID; union emails belonging to the same account.

```python
from collections import defaultdict

def accounts_merge(accounts: List[List[str]]) -> List[List[str]]:
    uf = UnionFind(len(accounts))
    email_to_acc = {}

    for i, acc in enumerate(accounts):
        for email in acc[1:]:
            if email in email_to_acc:
                uf.union(i, email_to_acc[email])
            else:
                email_to_acc[email] = i

    groups = defaultdict(list)
    for email, acc_idx in email_to_acc.items():
        leader = uf.find(acc_idx)
        groups[leader].append(email)

    return [[accounts[leader][0]] + sorted(emails) for leader, emails in groups.items()]
```
* **Optimality**: Operates in $O(N \cdot K \log K)$ time (due to sorting output) and $O(N \cdot K)$ space.
