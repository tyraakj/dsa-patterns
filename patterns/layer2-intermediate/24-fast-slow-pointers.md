# Pattern 24: Fast & Slow Pointers (Linked List)

> **Layer**: 2 (Intermediate High-Yield)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/), [Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/), [Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/), [Happy Number](https://leetcode.com/problems/happy-number/), [Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/)

---

## 1. Mental Model & Core Concept

Also known as **Floyd's Tortoise and Hare Algorithm**, this pattern moves two pointers through a sequence or state space at different speeds:
- `slow` advances 1 step at a time ($v = 1$).
- `fast` advances 2 steps at a time ($v = 2$).

```text
Cycle Detection:
[ Head ] ──► ... ──► [ Cycle Entrance ] ──► [ Loop ... ]
                          ▲                        │
                          └────────────────────────┘
If a cycle exists:
Relative speed = 2 - 1 = 1 step per iteration.
Fast reduces the gap to slow by 1 in each step, guaranteeing collision!
```

Mathematical Proof for Cycle Entrance:
- Distance from head to cycle entrance $= L$.
- Distance from entrance to collision point $= K$.
- Cycle length $= C$.
- When slow and fast collide, reset `slow` to `head` and keep `fast` at collision point. Move **both at speed 1**. They will meet precisely at the **cycle entrance** after $L$ steps!

---

## 2. Identification Signals ("When to Use")

- **Cycle Detection**: Determine if a linked list or implicit function chain has a loop.
- **Cycle Entrance**: Locate the exact node where the cycle begins.
- **Midpoint of Linked List**: Find the middle node in a single pass without counting length.
- **Implicit Linked Lists**: Number sequences like *Happy Number* or arrays representing next-pointer transitions (*Find the Duplicate Number*).

---

## 3. Algorithmic Template / Pseudocode

```text
function FLOYD_CYCLE_DETECTION(head):
    slow = head
    fast = head

    while fast is not null and fast.next is not null:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return true // Cycle detected!

    return false
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import Optional

class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

# --- Variant A: Linked List Cycle (Boolean Check) ---
def has_cycle(head: Optional[ListNode]) -> bool:
    slow = head
    fast = head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True

    return False

# --- Variant B: Linked List Cycle II (Find Entrance) ---
def detect_cycle(head: Optional[ListNode]) -> Optional[ListNode]:
    slow = head
    fast = head

    # Phase 1: Detect intersection point inside loop
    has_loop = False
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            has_loop = True
            break

    if not has_loop:
        return None

    # Phase 2: Locate entrance by moving both at speed 1
    slow = head
    while slow != fast:
        slow = slow.next
        fast = fast.next

    return slow

# --- Variant C: Middle of the Linked List ---
def middle_node(head: Optional[ListNode]) -> Optional[ListNode]:
    slow = head
    fast = head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    return slow

# --- Variant D: Find Duplicate Number (Array as Linked List) ---
def find_duplicate(nums: list[int]) -> int:
    # Array indices represent node addresses; nums[i] represents pointer to next node
    slow = nums[0]
    fast = nums[0]

    # Phase 1: Find collision point
    while True:
        slow = nums[slow]
        fast = nums[nums[fast]]
        if slow == fast:
            break

    # Phase 2: Find cycle entrance (the duplicate value)
    slow = nums[0]
    while slow != fast:
        slow = nums[slow]
        fast = nums[fast]

    return slow
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `detect_cycle`:
1. `while fast and fast.next: slow = slow.next; fast = fast.next.next`:
   * `fast` advances twice as fast as `slow`. If `fast` encounters a null pointer, the list is acyclic.
2. `if slow == fast: has_loop = True; break`:
   * The pointers have met inside the loop.
3. `slow = head`:
   * Resets `slow` to the origin while leaving `fast` at the collision point.
4. `while slow != fast: slow = slow.next; fast = fast.next`:
   * Both move at speed 1. Mathematical derivation proves they will collide after traversing exactly $L$ steps at the entry node.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(N)$ (Optimal)
- **Proof**:
  - Before entering the cycle, `slow` travels $L$ steps.
  - Once inside the cycle of length $C$, the distance between `fast` and `slow` decreases by 1 each step. They collide in at most $C$ steps.
  - Total steps $\le L + C = N \implies O(N)$.
  - Any cycle detection algorithm must traverse nodes up to the cycle, matching $\Omega(N)$ lower bound.

### Space Complexity: $O(1)$ (Optimal)
- Only two pointer variables (`slow`, `fast`) are stored.
- Eliminates the need for an $O(N)$ hash set of visited nodes, achieving the theoretical minimum space bound.

---

## 7. Key Invariants & Common Pitfalls

1. **Loop Condition Guard**:
   * *Trap*: Checking only `while fast:` without `while fast and fast.next:` triggers `AttributeError: 'NoneType' object has no attribute 'next'` when accessing `fast.next.next`.
2. **Cycle Entrance Reset**:
   * Resetting both pointers to `head` is useless. Only reset `slow = head` while keeping `fast` at the collision point.

---

## 8. Canonical Problem Walkthrough

### Happy Number (LeetCode #202)
* **Problem**: Sum squares of digits repeatedly. If it reaches 1, it is happy; otherwise it loops endlessly.
* **Reduction to Cycle Detection**: Treat digit sum calculation as `f(x) = next_node`.

```python
def is_happy(n: int) -> bool:
    def get_next(num: int) -> int:
        total = 0
        while num > 0:
            digit = num % 10
            total += digit * digit
            num //= 10
        return total

    slow = n
    fast = get_next(n)
    while fast != 1 and slow != fast:
        slow = get_next(slow)
        fast = get_next(get_next(fast))

    return fast == 1
```
* **Optimality**: Runs in $O(\log N)$ time and $O(1)$ space.
