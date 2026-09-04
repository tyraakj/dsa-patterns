# Pattern 12: Tree BFS (Level Order)

> **Layer**: 1 (Core Foundation)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/), [Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/), [Binary Tree Zigzag Level Order Traversal](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/), [Populating Next Right Pointers in Each Node](https://leetcode.com/problems/populating-next-right-pointers-in-each-node/)

---

## 1. Mental Model & Core Concept

Breadth-First Search (BFS) processes a tree **horizontally, layer by layer**, using a FIFO queue. All nodes at depth $D$ are visited before any node at depth $D + 1$.

```text
Level-Order Queue Mechanism:
Level 0: [ 1 ]
  Pop 1, Push Left (2) and Right (3)
Level 1: [ 2 , 3 ]
  Pop 2, Push 4, 5
  Pop 3, Push 6, 7
Level 2: [ 4 , 5 , 6 , 7 ]

Snapshot Trick:
Take snapshot of queue size: level_size = len(queue)
Loop exactly level_size times to process all nodes of that depth simultaneously!
```

Core applications:
1. **Level Grouping**: Collecting values grouped into separate sub-arrays per depth.
2. **Boundary / Horizon Views**: Right side view (last element of each level) or left side view (first element).
3. **Shortest Path in Trees**: First time a node meeting a condition is encountered in BFS, it is guaranteed to be at the minimum depth.

---

## 2. Identification Signals ("When to Use")

- **"Level by level"**: Grouping nodes by vertical depth.
- **Side Views**: Looking at the tree from the left or right horizon.
- **Zigzag / Alternating Traversal**: Reversing alternate levels.
- **Connecting Horizontal Siblings**: Setting `node.next` to point to its adjacent neighbor on the same depth.

---

## 3. Algorithmic Template / Pseudocode

```text
function TREE_BFS_LEVEL_ORDER(root):
    if root is null: return []
    queue = deque([root])
    result = []
    
    while queue is not empty:
        level_size = length(queue)
        current_level = []
        
        for i from 0 to level_size - 1:
            node = queue.pop_front()
            current_level.append(node.val)
            
            if node.left is not null: queue.push_back(node.left)
            if node.right is not null: queue.push_back(node.right)
            
        result.append(current_level)
        
    return result
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from collections import deque
from typing import Optional, List

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

# --- Variant A: Binary Tree Level Order Traversal ---
def level_order(root: Optional[TreeNode]) -> List[List[int]]:
    if not root:
        return []

    res = []
    queue = deque([root])

    while queue:
        level_size = len(queue)
        current_level = []

        for _ in range(level_size):
            node = queue.popleft()
            current_level.append(node.val)

            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)

        res.append(current_level)

    return res

# --- Variant B: Binary Tree Right Side View ---
def right_side_view(root: Optional[TreeNode]) -> List[int]:
    if not root:
        return []

    right_view = []
    queue = deque([root])

    while queue:
        level_size = len(queue)
        for i in range(level_size):
            node = queue.popleft()
            # If it is the last node in the current level, it is visible from the right
            if i == level_size - 1:
                right_view.append(node.val)

            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)

    return right_view

# --- Variant C: Zigzag Level Order Traversal ---
def zigzag_level_order(root: Optional[TreeNode]) -> List[List[int]]:
    if not root:
        return []

    res = []
    queue = deque([root])
    left_to_right = True

    while queue:
        level_size = len(queue)
        current_level = deque()

        for _ in range(level_size):
            node = queue.popleft()
            if left_to_right:
                current_level.append(node.val)
            else:
                current_level.appendleft(node.val)

            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)

        res.append(list(current_level))
        left_to_right = not left_to_right

    return res
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `level_order`:
1. `if not root: return []`:
   * Guards against empty tree input.
2. `queue = deque([root])`:
   * Seeds FIFO queue with the root node.
3. `while queue:`:
   * Continues until all tree levels are exhausted.
4. `level_size = len(queue)`:
   * **The Level-Boundary Snapshot**: Freezes the count of nodes currently residing at this exact depth. Even as children are added to the back of the queue, the inner loop processes only this level.
5. `node = queue.popleft()`:
   * Pops next node in $O(1)$ time.
6. `if node.left: queue.append(node.left); if node.right: queue.append(node.right)`:
   * Enqueues child nodes for the next level.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(V)$ (Optimal)
- **Proof**:
  - Each of the $V$ vertices enters the queue exactly once and is dequeued exactly once.
  - Enqueue and dequeue take $O(1)$ operations with `collections.deque`.
  - Visiting every node is required to construct the traversal, matching the $\Omega(V)$ lower bound.
  - Total time is strictly $O(V)$.

### Space Complexity: $O(W)$ (Optimal)
- Space is bounded by the maximum width $W$ of the tree (the maximum number of nodes at any single level).
- In a full binary tree, the leaf level contains $W = \lceil V / 2 \rceil$ nodes $\implies O(V)$ space.
- In a skewed tree, $W = 1 \implies O(1)$ space.
- Queue memory is bounded by $O(W)$, which is mathematically optimal for breadth-first layers.

---

## 7. Key Invariants & Common Pitfalls

1. **Snapshot Freezing**:
   * *Trap*: Using `for node in queue` directly without taking `level_size = len(queue)` causes the loop to iterate indefinitely as child nodes are appended.
2. **List as Queue**:
   * *Trap*: Using `queue.pop(0)` on a Python list degrades performance from $O(V)$ to $O(V^2)$ due to array shifting. Always use `collections.deque`.

---

## 8. Canonical Problem Walkthrough

### Minimum Depth of Binary Tree (LeetCode #111)
* **Problem**: Find the minimum depth (shortest path from root to nearest leaf).
* **BFS Superiority over DFS**: DFS must traverse all paths in the tree (worst-case $O(V)$ even if a leaf is at depth 1). BFS terminates immediately on the very first leaf encountered!

```python
def min_depth(root: Optional[TreeNode]) -> int:
    if not root:
        return 0
    queue = deque([(root, 1)])
    
    while queue:
        node, depth = queue.popleft()
        # First leaf encountered is guaranteed to be at minimum depth!
        if not node.left and not node.right:
            return depth
        if node.left:
            queue.append((node.left, depth + 1))
        if node.right:
            queue.append((node.right, depth + 1))
            
    return 0
```
* **Optimality**: Can terminate in $O(1)$ if the root has a leaf child, saving unnecessary traversals.
