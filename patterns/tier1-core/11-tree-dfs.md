# Pattern 11: Tree DFS (Preorder / Inorder / Postorder)

> **Tier**: 1 (Core Foundation)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/), [Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/), [Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/), [Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/)

---

## 1. Mental Model & Core Concept

Tree Depth-First Search (DFS) leverages recursion (or an explicit stack) to explore branches to their leaf nodes before backtracking. Trees are acyclic and hierarchical, meaning:
- Every subtree is itself a complete, independent binary tree.
- Subproblem results from children can be combined bottom-up to resolve the parent's state.

```text
Traversal Orders:
       1
      / \
     2   3
    / \
   4   5

Preorder  (Root, Left, Right) : [1, 2, 4, 5, 3]  (Top-down serialization, cloning)
Inorder   (Left, Root, Right) : [4, 2, 5, 1, 3]  (Produces sorted values in a BST)
Postorder (Left, Right, Root) : [4, 5, 2, 3, 1]  (Bottom-up aggregation, heights, deletions)
```

Core Insight:
- **Top-Down (Preorder)**: Pass parameters down the call stack (e.g. current path sum, parent value).
- **Bottom-Up (Postorder)**: Compute sub-results from leaves first (e.g. height, diameter, balance status) and return them up to the root.

---

## 2. Identification Signals ("When to Use")

- **Tree Properties**: Maximum depth, diameter, symmetric tree, balanced binary tree.
- **Path Verification**: Path sum equals target, longest consecutive sequence in tree.
- **Subtree Aggregation**: Maximum path sum, lowest common ancestor (LCA), subtree with identical structure.
- **Inorder Sortedness**: Validating a Binary Search Tree (BST) or finding $k$-th smallest element.

---

## 3. Algorithmic Template / Pseudocode

```text
function TREE_DFS_BOTTOM_UP(node):
    if node is null:
        return base_case_value (e.g. 0, true, null)
        
    left_result = TREE_DFS_BOTTOM_UP(node.left)
    right_result = TREE_DFS_BOTTOM_UP(node.right)
    
    // Process current node with children's returned results
    update_global_state(left_result, right_result, node.val)
    
    return aggregate_for_parent(left_result, right_result, node.val)
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import Optional

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

# --- Variant A: Bottom-Up Tree Diameter ---
def diameter_of_binary_tree(root: Optional[TreeNode]) -> int:
    max_diameter = 0

    def get_height(node: Optional[TreeNode]) -> int:
        nonlocal max_diameter
        if not node:
            return 0

        left_h = get_height(node.left)
        right_h = get_height(node.right)

        # Diameter at current node is sum of left and right branch heights
        max_diameter = max(max_diameter, left_h + right_h)

        # Return height of this subtree to parent
        return 1 + max(left_h, right_h)

    get_height(root)
    return max_diameter

# --- Variant B: Lowest Common Ancestor (LCA) ---
def lowest_common_ancestor(root: TreeNode, p: TreeNode, q: TreeNode) -> Optional[TreeNode]:
    # Base case: empty node or found one of the targets
    if not root or root == p or root == q:
        return root

    left = lowest_common_ancestor(root.left, p, q)
    right = lowest_common_ancestor(root.right, p, q)

    # If both left and right return non-null, root is the LCA
    if left and right:
        return root
    # Otherwise propagate the non-null child up
    return left if left else right

# --- Variant C: Binary Tree Maximum Path Sum ---
def max_path_sum(root: Optional[TreeNode]) -> int:
    max_sum = float('-inf')

    def get_gain(node: Optional[TreeNode]) -> int:
        nonlocal max_sum
        if not node:
            return 0

        # Ignore paths with negative sums by clamping to 0
        left_gain = max(get_gain(node.left), 0)
        right_gain = max(get_gain(node.right), 0)

        # Price of new path where current node is the highest point (turning point)
        current_path_sum = node.val + left_gain + right_gain
        max_sum = max(max_sum, current_path_sum)

        # Return maximum contribution if parent continues through this node
        return node.val + max(left_gain, right_gain)

    get_gain(root)
    return int(max_sum)
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `diameter_of_binary_tree`:
1. `max_diameter = 0`:
   * Global accumulator tracking the maximum path length (number of edges).
2. `if not node: return 0`:
   * Base case: null node has a height of 0.
3. `left_h = get_height(node.left); right_h = get_height(node.right)`:
   * Postorder recursion: computes heights of both subtrees first.
4. `max_diameter = max(max_diameter, left_h + right_h)`:
   * **The Turning Point**: A path between two nodes in different subtrees passes through `node`. Its length is `left_h + right_h`.
5. `return 1 + max(left_h, right_h)`:
   * The height returned to the caller can only extend along **one** branch (either left or right).

### Breakdown of `lowest_common_ancestor`:
1. `if not root or root == p or root == q: return root`:
   * If root is null, return null. If root matches target node `p` or `q`, return root (found target).
2. `left = lowest_common_ancestor(root.left, p, q); right = lowest_common_ancestor(root.right, p, q)`:
   * Recursively searches left and right subtrees.
3. `if left and right: return root`:
   * If one target is found in the left subtree and the other in the right subtree, `root` is their Lowest Common Ancestor.
4. `return left if left else right`:
   * If only one side found a target, bubble that result up to the parent.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(V)$ (Optimal)
- **Proof**:
  - In an acyclic binary tree with $V$ vertices and $E = V - 1$ edges, DFS visits each vertex exactly once.
  - At each vertex, the work performed is $O(1)$ constant time (comparisons, additions, assignments).
  - Total operations $= C \cdot V \implies O(V)$.
  - Any algorithm determining tree diameter or LCA must inspect tree nodes in the worst case, matching the $\Omega(V)$ lower bound.

### Space Complexity: $O(H)$ (Optimal)
- Space is determined by recursion call stack depth, which equals the height of the tree $H$.
- For balanced trees: $H = \lceil \log_2 V \rceil \implies O(\log V)$ space.
- For skewed/degenerate trees (linked list structure): $H = V \implies O(V)$ space.
- No auxiliary heap memory is allocated.

---

## 7. Key Invariants & Common Pitfalls

1. **Path Sum vs Branch Return**:
   * *Trap*: When computing maximum path sum, you can combine left and right gains at the local root (`node.val + left_gain + right_gain`), but you **cannot** return both to the parent! A valid path cannot bifurcate; you can only return `node.val + max(left_gain, right_gain)` to the caller.
2. **Negative Subtree Values**:
   * In path sum problems, subtrees with negative total sums should be clamped to 0 (`max(gain, 0)`), which corresponds to choosing not to include that subtree.

---

## 8. Canonical Problem Walkthrough

### Maximum Depth of Binary Tree (LeetCode #104)
* **Problem**: Find the length of the longest path from root to leaf.
* **Recurrence**: $\text{depth}(node) = 1 + \max(\text{depth}(node.left), \text{depth}(node.right))$.

```python
def max_depth(root: Optional[TreeNode]) -> int:
    if not root:
        return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))
```
* **Optimality**: Operates in $O(V)$ time with $O(H)$ stack space.
