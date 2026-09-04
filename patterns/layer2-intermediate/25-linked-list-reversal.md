# Pattern 25: Linked List Reversal / Manipulation

> **Layer**: 2 (Intermediate High-Yield)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/), [Reverse Linked List II](https://leetcode.com/problems/reverse-linked-list-ii/), [Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/), [Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/)

---

## 1. Mental Model & Core Concept

Linked List Reversal modifies pointer directions in-place without creating new nodes or altering data values. 

```text
Iterative 3-Pointer Reversal:
prev      curr      next_temp
  ▼         ▼           ▼
[None]    [ 1 ] ──►   [ 2 ] ──► [ 3 ]

Step 1: next_temp = curr.next (Save forward connection)
Step 2: curr.next = prev      (Reverse pointer)
Step 3: prev = curr           (Advance prev)
Step 4: curr = next_temp      (Advance curr)
```

Essential Design Patterns:
1. **Sentinel / Dummy Node**: Placing a `dummy = ListNode(0, head)` eliminates edge cases when reversing or deleting the head node.
2. **Subsegment Reversal**: Isolating a window `[left, right]` and reattaching boundaries cleanly.

---

## 2. Identification Signals ("When to Use")

- **In-Place Reversal**: Reversing an entire list, or a range between position $m$ and $n$.
- **Chunk Reversal**: Reversing nodes in groups of $k$.
- **Palindrome Verification**: Finding midpoint, reversing second half in-place, and comparing with first half in $O(1)$ space.

---

## 3. Algorithmic Template / Pseudocode

```text
function REVERSE_LIST(head):
    prev = null
    curr = head
    
    while curr is not null:
        next_temp = curr.next
        curr.next = prev
        prev = curr
        curr = next_temp
        
    return prev // New head of reversed list
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import Optional

class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

# --- Variant A: Standard In-Place Reversal ---
def reverse_list(head: Optional[ListNode]) -> Optional[ListNode]:
    prev = None
    curr = head

    while curr:
        next_temp = curr.next
        curr.next = prev
        prev = curr
        curr = next_temp

    return prev

# --- Variant B: Reverse Linked List II (Subsegment [left, right]) ---
def reverse_between(head: Optional[ListNode], left: int, right: int) -> Optional[ListNode]:
    if not head or left == right:
        return head

    dummy = ListNode(0, head)
    prev = dummy

    # 1. Advance prev to node immediately before subsegment
    for _ in range(left - 1):
        prev = prev.next

    # 2. Reverse subsegment in-place
    curr = prev.next
    for _ in range(right - left):
        temp = curr.next
        curr.next = temp.next
        temp.next = prev.next
        prev.next = temp

    return dummy.next

# --- Variant C: Reverse Nodes in k-Group ---
def reverse_k_group(head: Optional[ListNode], k: int) -> Optional[ListNode]:
    dummy = ListNode(0, head)
    group_prev = dummy

    def get_kth(curr: Optional[ListNode], k_steps: int) -> Optional[ListNode]:
        while curr and k_steps > 0:
            curr = curr.next
            k_steps -= 1
        return curr

    while True:
        kth = get_kth(group_prev, k)
        if not kth:
            break  # Fewer than k nodes remaining, preserve order

        group_next = kth.next

        # Reverse this k-group
        prev, curr = kth.next, group_prev.next
        while curr != group_next:
            tmp = curr.next
            curr.next = prev
            prev = curr
            curr = tmp

        # Re-link reversed group with surroundings
        tmp = group_prev.next
        group_prev.next = kth
        group_prev = tmp

    return dummy.next
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `reverse_list`:
1. `prev = None; curr = head`:
   * `prev` tracks the tail of the newly reversed portion; `curr` tracks the head of the remaining portion.
2. `next_temp = curr.next`:
   * **Saves Forward Reference**: You *must* store `curr.next` before mutating it, otherwise the rest of the list becomes unreachable in memory.
3. `curr.next = prev`:
   * Flips pointer direction backward.
4. `prev = curr; curr = next_temp`:
   * Shifts both pointers forward by one node.
5. `return prev`:
   * When `curr` is None, `prev` sits on the final node (the new head).

### Breakdown of `reverse_between`:
1. `dummy = ListNode(0, head)`:
   * Sentinel node ensures `dummy.next` always points to the correct head, even if `left == 1`.
2. `temp = curr.next; curr.next = temp.next; temp.next = prev.next; prev.next = temp`:
   * Moves `temp` to the front of the subsegment in a single step without breaking continuity.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(N)$ (Optimal)
- **Proof**:
  - Each node in `reverse_list` is visited once. At each node, exactly 4 pointer assignments take place ($O(1)$ operations).
  - In `reverse_k_group`, each node is checked once for length and reversed once $\implies \le 2N$ pointer adjustments.
  - Total time: $O(N)$. Optimal because all pointers must be inverted.

### Space Complexity: $O(1)$ (Optimal)
- Pointer mutations occur strictly in-place.
- No new nodes, recursion stack frames, or auxiliary arrays are created $\implies O(1)$ space.

---

## 7. Key Invariants & Common Pitfalls

1. **Lost Head Reference**:
   * Modifying `head.next` before saving forward pointers drops the entire list from memory, causing silent garbage collection.
2. **Dummy Sentinel Requirement**:
   * Any problem modifying the beginning of a list (`left = 1` or deleting head) should use `dummy = ListNode(0, head)` to avoid special-case branching.

---

## 8. Canonical Problem Walkthrough

### Palindrome Linked List (LeetCode #234)
* **Problem**: Determine if a singly linked list is a palindrome in $O(N)$ time and $O(1)$ space.
* **Algorithm**:
  1. Find middle with Fast & Slow pointers.
  2. Reverse second half of list in-place.
  3. Compare first half and reversed second half node-by-node.

```python
def is_palindrome(head: Optional[ListNode]) -> bool:
    # 1. Find midpoint
    slow, fast = head, head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    # 2. Reverse second half
    prev = None
    curr = slow
    while curr:
        nxt = curr.next
        curr.next = prev
        prev = curr
        curr = nxt

    # 3. Compare two halves
    left, right = head, prev
    while right:
        if left.val != right.val:
            return False
        left = left.next
        right = right.next

    return True
```
* **Optimality**: Operates in $O(N)$ time and $O(1)$ auxiliary space.
