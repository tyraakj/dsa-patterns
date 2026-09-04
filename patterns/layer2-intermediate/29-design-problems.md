# Pattern 29: Design Problems

> **Layer**: 2 (Intermediate High-Yield)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [LRU Cache](https://leetcode.com/problems/lru-cache/), [LFU Cache](https://leetcode.com/problems/lfu-cache/), [Design Twitter](https://leetcode.com/problems/design-twitter/), [Insert Delete GetRandom O(1)](https://leetcode.com/problems/insert-delete-getrandom-o1/)

---

## 1. Mental Model & Core Concept

Design problems combine multiple foundational data structures to meet strict $O(1)$ constant-time constraints across all interface methods.

```text
LRU Cache Architecture:
Hash Map (O(1) Key Lookup) + Doubly Linked List (O(1) Node Eviction & Insertion)

[Head Sentinel] <──► [Node A] <──► [Node B] <──► [Tail Sentinel]
  (Least Recently Used)                             (Most Recently Used)

1. Access Node:  Splice out of current position and move adjacent to Tail.
2. Evict Oldest: Pop node adjacent to Head and delete key from Hash Map.
```

Data Structure Pairing Matrix:
- **$O(1)$ Lookup + $O(1)$ Removal/Reordering**: Hash Map + Doubly Linked List (LRU Cache).
- **$O(1)$ Insert + $O(1)$ Delete + $O(1)$ Random Choice**: Hash Map + Dynamic Array (swap-to-back eviction trick).
- **$O(1)$ Feed Aggregation**: Hash Map + Min-Heap of K latest posts per followed user.

---

## 2. Identification Signals ("When to Use")

- **Explicit Class Interface**: Implement `get`, `put`, `insert`, `remove`, `getRandom` in $O(1)$ average time.
- **Cache Invalidation**: Evicting Least Recently Used (LRU) or Least Frequently Used (LFU) entries upon reaching capacity.
- **Random Selection with Uniform Probability**: Random element from a dynamic set in $O(1)$ time.

---

## 3. Algorithmic Template / Pseudocode

```text
class LRU_CACHE(capacity):
    map = hash table mapping (key -> Node)
    head, tail = sentinel nodes connected to each other

    function get(key):
        if key in map:
            node = map[key]
            remove_node(node)
            insert_to_tail(node) // Mark most recently used
            return node.val
        return -1
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import Dict, Optional

# --- Variant A: LRU Cache (Hash Map + Doubly Linked List) ---
class DNode:
    def __init__(self, key=0, val=0):
        self.key = key
        self.val = val
        self.prev: Optional['DNode'] = None
        self.next: Optional['DNode'] = None

class LRUCache:
    def __init__(self, capacity: int):
        self.cap = capacity
        self.cache: Dict[int, DNode] = {}
        # Sentinels
        self.head = DNode()
        self.tail = DNode()
        self.head.next = self.tail
        self.tail.prev = self.head

    def _remove(self, node: DNode) -> None:
        p, n = node.prev, node.next
        p.next = n
        n.prev = p

    def _insert_tail(self, node: DNode) -> None:
        p = self.tail.prev
        p.next = node
        node.prev = p
        node.next = self.tail
        self.tail.prev = node

    def get(self, key: int) -> int:
        if key in self.cache:
            node = self.cache[key]
            self._remove(node)
            self._insert_tail(node)  # Move to MRU position
            return node.val
        return -1

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self._remove(self.cache[key])

        new_node = DNode(key, value)
        self.cache[key] = new_node
        self._insert_tail(new_node)

        if len(self.cache) > self.cap:
            # Evict LRU (node next to head)
            lru = self.head.next
            self._remove(lru)
            del self.cache[lru.key]

# --- Variant B: Insert Delete GetRandom O(1) ---
class RandomizedSet:
    def __init__(self):
        self.val_to_idx = {}  # value -> array index
        self.arr = []

    def insert(self, val: int) -> bool:
        if val in self.val_to_idx:
            return False
        self.val_to_idx[val] = len(self.arr)
        self.arr.append(val)
        return True

    def remove(self, val: int) -> bool:
        if val not in self.val_to_idx:
            return False
        # Swap target element with last element in array for O(1) deletion!
        idx_to_remove = self.val_to_idx[val]
        last_val = self.arr[-1]

        self.arr[idx_to_remove] = last_val
        self.val_to_idx[last_val] = idx_to_remove

        self.arr.pop()
        del self.val_to_idx[val]
        return True

    def get_random(self) -> int:
        import random
        return random.choice(self.arr)
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `LRUCache`:
1. `self.head.next = self.tail; self.tail.prev = self.head`:
   * Sentinel nodes avoid edge-case branching for empty lists or head/tail pointer mutations.
2. `self._remove(node); self._insert_tail(node)`:
   * Detaches node from its current position and reattaches it directly in front of `tail` in $O(1)$ constant pointer manipulations.
3. `lru = self.head.next; self._remove(lru); del self.cache[lru.key]`:
   * When capacity is exceeded, evicts the least recently used node (immediately after `head`) from both the doubly linked list and the hash map in $O(1)$ time.

### Breakdown of `RandomizedSet`:
1. `idx_to_remove = self.val_to_idx[val]; last_val = self.arr[-1]`:
   * Deleting from an arbitrary array index takes $O(N)$ because elements must shift.
2. `self.arr[idx_to_remove] = last_val; self.val_to_idx[last_val] = idx_to_remove`:
   * **Swap-to-Back Trick**: Overwrites the target element with the last element in $O(1)$, updates the hash map pointer for `last_val`, and pops the back of the array in $O(1)$ time!

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(1)$ for All Operations (Optimal)
- In `LRUCache`: Hash map lookups and deletions take $O(1)$ average time. Doubly linked list removals and insertions modify exactly 4 pointers ($O(1)$ steps). Total runtime for `get` and `put` is strictly $O(1)$.
- In `RandomizedSet`: Array index lookup, swap-to-back, and `random.choice` on contiguous arrays execute in $O(1)$ constant time.
- Matches the theoretical minimum bound for individual cache interactions.

### Space Complexity: $O(\text{Capacity})$ (Optimal)
- Stores at most `capacity` entries in both the hash map and the doubly linked list $\implies O(\text{capacity})$ space.

---

## 7. Key Invariants & Common Pitfalls

1. **Storing Keys in Linked List Nodes**:
   * DNode must store **both** `key` and `val`. When evicting `head.next`, you must know its `key` to delete it from `self.cache`!
2. **Deleting Last Element in `RandomizedSet`**:
   * If the element being removed is already the last element in `self.arr`, updating `self.val_to_idx[last_val]` before deleting handles the edge case seamlessly.

---

## 8. Canonical Problem Walkthrough

### LFU Cache (LeetCode #460)
* **Problem**: Evict Least Frequently Used; if frequencies tie, evict Least Recently Used.
* **Architecture**:
  - `key_to_val_freq = {key: (val, freq)}`
  - `freq_to_nodes = {freq: OrderedDict()}`
  - `min_freq` integer tracker.
* **Optimality**: Operates in $O(1)$ time per `get` and `put` using dual maps.
