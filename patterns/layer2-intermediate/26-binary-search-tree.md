# Pattern 26: Binary Search Tree (BST)

> **Layer**: 2 (Intermediate High-Yield)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/), [Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/), [Lowest Common Ancestor of a BST](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/), [Delete Node in a BST](https://leetcode.com/problems/delete-node-in-a-bst/)

---

## 1. Mental Model & Core Concept

A Binary Search Tree (BST) is a binary tree where every node satisfies the **BST Ordering Invariant**:
- Every node in the **left subtree** has a value strictly less than `node.val`.
- Every node in the **right subtree** has a value strictly greater than `node.val`.

```text
The Fundamental BST Invariant:
An INORDER traversal (Left, Root, Right) of a valid BST
produces values in STRICTLY ASCENDING ORDER!

           4
         /   \
        2     6      ──► Inorder Traversal: [1, 2, 3, 4, 5, 6, 7]
       / \   / \         (Strictly Monotonically Increasing)
      1   3 5   7
```

Searching in a BST is analogous to binary search on an array: if target is smaller than `root.val`, discard the entire right subtree; if target is larger, discard the entire left subtree.

---

## 2. Identification Signals ("When to Use")

- **BST Property Exploitation**: Finding $K$-th smallest/largest element, LCA in BST.
- **BST Validation**: Proving a tree is a valid BST (checking against $(min\_val, max\_val)$ intervals).
- **Mutations**: Insertion, deletion, or finding the Inorder Successor/Predecessor in $O(H)$ time.

---

## 3. Algorithmic Template / Pseudocode

```text
function VALIDATE_BST(node, low_bound, high_bound):
    if node is null:
        return true

    if not (low_bound < node.val < high_bound):
        return false // Invariant violated

    return VALIDATE_BST(node.left, low_bound, node.val) and
           VALIDATE_BST(node.right, node.val, high_bound)
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

# --- Variant A: Validate Binary Search Tree ---
def is_valid_bst(root: Optional[TreeNode]) -> bool:
    def validate(node: Optional[TreeNode], low: float, high: float) -> bool:
        if not node:
            return True
        if not (low < node.val < high):
            return False
        # Left subtree bounded by (low, node.val); Right subtree bounded by (node.val, high)
        return validate(node.left, low, node.val) and validate(node.right, node.val, high)

    return validate(root, float('-inf'), float('inf'))

# --- Variant B: Kth Smallest Element (Iterative Inorder) ---
def kth_smallest(root: Optional[TreeNode], k: int) -> int:
    stack = []
    curr = root

    while curr or stack:
        # Traverse to leftmost leaf
        while curr:
            stack.append(curr)
            curr = curr.left

        curr = stack.pop()
        k -= 1
        if k == 0:
            return curr.val

        curr = curr.right

    return -1

# --- Variant C: Lowest Common Ancestor in BST ---
def lowest_common_ancestor_bst(root: TreeNode, p: TreeNode, q: TreeNode) -> TreeNode:
    curr = root
    while curr:
        if p.val < curr.val and q.val < curr.val:
            curr = curr.left   # Both nodes in left subtree
        elif p.val > curr.val and q.val > curr.val:
            curr = curr.right  # Both nodes in right subtree
        else:
            return curr        # Split point: curr is the LCA!
    return root
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `is_valid_bst`:
1. `def validate(node, low, high):`:
   * Passes permissible interval ranges $(low, high)$ downward.
2. `if not (low < node.val < high): return False`:
   * Validates that `node.val` is strictly inside the valid interval. This catches subtle bugs where an element is valid relative to its parent, but violates an ancestor's bound (e.g. root is 5, right child is 8, right child's left is 4).
3. `return validate(node.left, low, node.val) and validate(node.right, node.val, high)`:
   * Left subtree narrows the upper bound to `node.val`; right subtree narrows the lower bound to `node.val`.

### Breakdown of `lowest_common_ancestor_bst`:
1. `if p.val < curr.val and q.val < curr.val: curr = curr.left`:
   * Because of BST ordering, if both targets are strictly smaller than `curr.val`, their common ancestor must be in the left subtree.
2. `elif p.val > curr.val and q.val > curr.val: curr = curr.right`:
   * Symmetrically, if both are larger, navigate right.
3. `else: return curr`:
   * **The Split Invariant**: If one target is on the left and the other is on the right (or one equals `curr`), `curr` is the earliest divergence point $\implies$ the LCA!

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(H)$ (Optimal)
- In `lowest_common_ancestor_bst`, each decision step descends one level, inspecting exactly 1 node per level $\implies O(H)$ operations where $H$ is the tree height.
  - Balanced BST: $H = \lceil \log_2 N \rceil \implies O(\log N)$.
  - Skewed BST: $H = N \implies O(N)$.
- For `kth_smallest`: Stops after visiting $k$ elements $\implies O(H + k)$ time.
- Optimal because at least the path to the node must be traversed.

### Space Complexity: $O(H)$ (Optimal)
- In recursive validation and iterative inorder traversal, memory is bounded by the recursion depth or stack size $\implies O(H)$ space.

---

## 7. Key Invariants & Common Pitfalls

1. **Local Parent Check is Insufficient**:
   * *Trap*: Only checking `node.left.val < node.val` and `node.right.val > node.val` allows invalid trees like `[5, 4, 6, null, null, 3, 7]` (where 3 is in 5's right subtree!). Always enforce global range bounds $(low, high)$.
2. **Strict Inequality**:
   * Standard BST definition requires strictly distinct elements (`low < node.val < high`).

---

## 8. Canonical Problem Walkthrough

### Delete Node in a BST (LeetCode #450)
* **Problem**: Delete a node with key while maintaining the BST invariant.
* **3 Cases**:
  1. Leaf node: simply remove it.
  2. One child: replace node with its child.
  3. Two children: replace node with its **Inorder Successor** (smallest node in right subtree), then delete the successor.

```python
def delete_node(root: Optional[TreeNode], key: int) -> Optional[TreeNode]:
    if not root:
        return None
    if key < root.val:
        root.left = delete_node(root.left, key)
    elif key > root.val:
        root.right = delete_node(root.right, key)
    else:
        if not root.left:
            return root.right
        elif not root.right:
            return root.left
        # Node has two children: find inorder successor (min in right subtree)
        successor = root.right
        while successor.left:
            successor = successor.left
        root.val = successor.val
        root.right = delete_node(root.right, successor.val)
    return root
```
* **Optimality**: Operates in $O(H)$ time and $O(H)$ stack space.
