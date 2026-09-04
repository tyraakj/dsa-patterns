# Pattern 10: Queue / Deque

> **Tier**: 1 (Core Foundation)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks/), [Moving Average from Data Stream](https://leetcode.com/problems/moving-average-from-data-stream/), [Design Circular Deque](https://leetcode.com/problems/design-circular-deque/), [Number of Recent Calls](https://leetcode.com/problems/number-of-recent-calls/)

---

## 1. Mental Model & Core Concept

A Queue maintains a strict **First-In, First-Out (FIFO)** discipline: elements are appended at the rear and evicted from the front. A Double-Ended Queue (Deque) permits $O(1)$ push and pop operations at both the head and the tail.

```text
FIFO Queue Mechanism:
Enqueue: Append to Right ──► [ 1 , 2 , 3 , 4 ] ──► Dequeue: Pop from Left (Oldest First)
```

Critical Python Implementation Detail:
- Never use a Python standard list `list` as a FIFO queue with `list.pop(0)`. In Python, `list.pop(0)` takes $O(N)$ time because all subsequent memory elements must shift left by one word.
- **Always use `collections.deque`**, which is implemented as a doubly linked list of fixed-size blocks, ensuring true $O(1)$ amortized `append()` and `popleft()`.

---

## 2. Identification Signals ("When to Use")

- **Sequential Buffering**: Processing streams in the exact temporal order they arrived.
- **Time-Window Eviction**: Evicting timestamps older than a fixed interval (e.g., requests within the last 3000 ms).
- **Moving Averages**: Computing sliding statistics over the last $K$ stream items.
- **Level-Order Traversals (Tree/Graph BFS)**: Foundational FIFO queue driver.

---

## 3. Algorithmic Template / Pseudocode

```text
function TIME_WINDOW_QUEUE():
    queue = empty FIFO queue
    
    function ADD_EVENT(timestamp, window_size):
        queue.push(timestamp)
        while queue is not empty and queue.front() < timestamp - window_size:
            queue.pop_front()
        return queue.size()
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from collections import deque
from typing import List

# --- Variant A: Rate Limiter / Window Buffer (Number of Recent Calls) ---
class RecentCounter:
    def __init__(self):
        self.queue = deque()

    def ping(self, t: int) -> int:
        self.queue.append(t)
        # Evict all timestamps older than t - 3000
        while self.queue and self.queue[0] < t - 3000:
            self.queue.popleft()
        return len(self.queue)

# --- Variant B: Implement Queue Using Two Stacks ---
class MyQueue:
    def __init__(self):
        self.in_stack = []   # Accepts pushes
        self.out_stack = []  # Serves pops

    def push(self, x: int) -> None:
        self.in_stack.append(x)

    def pop(self) -> int:
        self._shift_elements()
        return self.out_stack.pop()

    def peek(self) -> int:
        self._shift_elements()
        return self.out_stack[-1]

    def empty(self) -> bool:
        return not self.in_stack and not self.out_stack

    def _shift_elements(self) -> None:
        # Only transfer elements if out_stack is empty to maintain FIFO order
        if not self.out_stack:
            while self.in_stack:
                self.out_stack.append(self.in_stack.pop())

# --- Variant C: Circular Deque Array Implementation ---
class MyCircularDeque:
    def __init__(self, k: int):
        self.capacity = k + 1  # 1 extra slot to distinguish full from empty
        self.arr = [0] * self.capacity
        self.front = 0
        self.rear = 0

    def insert_front(self, value: int) -> bool:
        if self.is_full():
            return False
        self.front = (self.front - 1 + self.capacity) % self.capacity
        self.arr[self.front] = value
        return True

    def insert_last(self, value: int) -> bool:
        if self.is_full():
            return False
        self.arr[self.rear] = value
        self.rear = (self.rear + 1) % self.capacity
        return True

    def delete_front(self) -> bool:
        if self.is_empty():
            return False
        self.front = (self.front + 1) % self.capacity
        return True

    def delete_last(self) -> bool:
        if self.is_empty():
            return False
        self.rear = (self.rear - 1 + self.capacity) % self.capacity
        return True

    def get_front(self) -> int:
        return -1 if self.is_empty() else self.arr[self.front]

    def get_rear(self) -> int:
        return -1 if self.is_empty() else self.arr[(self.rear - 1 + self.capacity) % self.capacity]

    def is_empty(self) -> bool:
        return self.front == self.rear

    def is_full(self) -> bool:
        return (self.rear + 1) % self.capacity == self.front
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `RecentCounter`:
1. `self.queue = deque()`:
   * Instantiates a double-ended queue supporting $O(1)$ operations at both ends.
2. `self.queue.append(t)`:
   * Adds incoming timestamp to rear of queue.
3. `while self.queue and self.queue[0] < t - 3000: self.queue.popleft()`:
   * Evicts outdated timestamps from the front of the queue in $O(1)$ time per eviction.
4. `return len(self.queue)`:
   * Returns count of valid events remaining in the 3000 ms sliding window.

### Breakdown of `MyQueue` (Amortized FIFO via 2 Stacks):
1. `self.in_stack = []; self.out_stack = []`:
   * `in_stack` buffers new arrivals; `out_stack` serves reads/deletions.
2. `self._shift_elements()`:
   * **The Invariant**: Elements are only poured from `in_stack` into `out_stack` when `out_stack` is completely empty. Pouring reverses the LIFO order into FIFO order. Leaving them in `out_stack` until depleted guarantees elements are popped in exact arrival order.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(1)$ Amortized per Operation (Optimal)
- In `RecentCounter`: Each timestamp is enqueued once and dequeued at most once. Over $N$ ping calls, total queue modifications $\le 2N$. Amortized time per ping is $O(1)$.
- In `MyQueue`: Each item is pushed to `in_stack` once, popped to `out_stack` once, and popped from `out_stack` once. Total operations per element $= 3 \implies O(1)$ amortized.

### Space Complexity: $O(K)$ or $O(N)$ (Optimal)
- Memory is strictly bounded by the maximum number of active elements in the window or queue. In `RecentCounter`, space is bounded by the max number of calls in 3000 ms ($O(K)$), which is optimal.

---

## 7. Key Invariants & Common Pitfalls

1. **`list.pop(0)` Penalty**:
   * Using `list.pop(0)` on an array of size $N$ requires shifting all $N-1$ elements in memory, causing total runtime to degrade from $O(N)$ to $O(N^2)$. Always use `collections.deque.popleft()`.
2. **Circular Buffer Full vs Empty Condition**:
   * In a circular array buffer of size $K$, `front == rear` is ambiguous (could mean empty or full).
   * *Solution*: Allocate size $K + 1$. Empty is `front == rear`; Full is `(rear + 1) % capacity == front`.

---

## 8. Canonical Problem Walkthrough

### Moving Average from Data Stream (LeetCode #346)
* **Problem**: Calculate moving average of all integers in the sliding window of size `size`.
* **Queue Invariant**: Maintain sum incrementally by subtracting the popped item and adding the new item.

```python
class MovingAverage:
    def __init__(self, size: int):
        self.size = size
        self.queue = deque()
        self.window_sum = 0

    def next(self, val: int) -> float:
        if len(self.queue) == self.size:
            self.window_sum -= self.queue.popleft()
        self.queue.append(val)
        self.window_sum += val
        return self.window_sum / len(self.queue)
```
* **Optimality**: Operates in $O(1)$ time per `next` call and $O(K)$ space.
