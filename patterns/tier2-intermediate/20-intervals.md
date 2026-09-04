# Pattern 20: Intervals (Merge / Insert / Overlap)

> **Tier**: 2 (Intermediate High-Yield)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Merge Intervals](https://leetcode.com/problems/merge-intervals/), [Insert Interval](https://leetcode.com/problems/insert-interval/), [Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/), [Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/)

---

## 1. Mental Model & Core Concept

Interval problems deal with 1D segments $[start, end]$. The universal prerequisite to solving almost every interval problem is **sorting by start time** (or occasionally by end time).

```text
Overlapping Intervals Detection:
Interval A: [ ------ ]
Interval B:      [ ------ ]       (Overlap exists: B.start <= A.end)
Merged:     [ ------------ ]      (New End = max(A.end, B.end))

Non-Overlapping Intervals:
Interval A: [ ------ ]
Interval C:            [ ------ ] (No overlap: C.start > A.end)
```

Key Invariant:
Once intervals are sorted by start time, an incoming interval `curr` can only overlap with the most recently merged interval `prev` if:
$$curr.start \le prev.end$$

---

## 2. Identification Signals ("When to Use")

- **Consolidating Ranges**: Merging overlapping time spans, busy schedules, or IP address ranges.
- **Insert & Adjust**: Inserting a new interval into an already sorted list.
- **Mutual Exclusions / Maximum Concurrency**: Meeting rooms required, minimum deletions to eliminate overlaps.

---

## 3. Algorithmic Template / Pseudocode

```text
function MERGE_INTERVALS(intervals):
    sort intervals by start time: interval[0]
    merged = [intervals[0]]

    for i from 1 to length(intervals) - 1:
        prev = merged.last()
        curr = intervals[i]

        if curr.start <= prev.end:
            prev.end = max(prev.end, curr.end) // Extend overlap
        else:
            merged.append(curr)                // Disjoint interval

    return merged
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
import heapq
from typing import List

# --- Variant A: Merge Overlapping Intervals ---
def merge(intervals: List[List[int]]) -> List[List[int]]:
    if not intervals:
        return []

    # Sort by start time
    intervals.sort(key=lambda x: x[0])
    merged = [intervals[0]]

    for current in intervals[1:]:
        prev = merged[-1]
        # Overlap condition
        if current[0] <= prev[1]:
            prev[1] = max(prev[1], current[1])
        else:
            merged.append(current)

    return merged

# --- Variant B: Insert Interval (O(N) In-Place Linear Insertion) ---
def insert(intervals: List[List[int]], new_interval: List[int]) -> List[List[int]]:
    res = []
    i = 0
    n = len(intervals)

    # 1. Add all intervals ending before new_interval begins
    while i < n and intervals[i][1] < new_interval[0]:
        res.append(intervals[i])
        i += 1

    # 2. Merge all overlapping intervals into new_interval
    while i < n and intervals[i][0] <= new_interval[1]:
        new_interval[0] = min(new_interval[0], intervals[i][0])
        new_interval[1] = max(new_interval[1], intervals[i][1])
        i += 1
    res.append(new_interval)

    # 3. Add all intervals starting after new_interval ends
    while i < n:
        res.append(intervals[i])
        i += 1

    return res

# --- Variant C: Meeting Rooms II (Minimum Rooms via Min-Heap) ---
def min_meeting_rooms(intervals: List[List[int]]) -> int:
    if not intervals:
        return 0

    intervals.sort(key=lambda x: x[0])
    # Min-heap stores end times of active meetings in occupied rooms
    room_heap = []
    heapq.heappush(room_heap, intervals[0][1])

    for meeting in intervals[1:]:
        # If earliest meeting has finished, free and reuse that room
        if meeting[0] >= room_heap[0]:
            heapq.heappop(room_heap)

        heapq.heappush(room_heap, meeting[1])

    return len(room_heap)
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `merge`:
1. `intervals.sort(key=lambda x: x[0])`:
   * Guarantees that for any indices $j > i$, `intervals[j].start >= intervals[i].start`.
2. `merged = [intervals[0]]`:
   * Seeds output with the earliest starting interval.
3. `if current[0] <= prev[1]: prev[1] = max(prev[1], current[1])`:
   * If current starts before or exactly when previous ends, they overlap. The merged end is `max(prev[1], current[1])` (e.g. $[1, 4]$ and $[2, 3]$ merged is $[1, 4]$).
4. `else: merged.append(current)`:
   * Current interval is disjoint; appends as a new boundary anchor.

### Breakdown of `min_meeting_rooms`:
1. `heapq.heappush(room_heap, intervals[0][1])`:
   * Tracks earliest room freeing time.
2. `if meeting[0] >= room_heap[0]: heapq.heappop(room_heap)`:
   * If current meeting starts after earliest meeting finishes, reuse room by popping old end time.
3. `heapq.heappush(room_heap, meeting[1])`:
   * Assigns room with current meeting's finish time. Heap length represents concurrent rooms required.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(N \log N)$ (Optimal)
- **Proof**:
  - Sorting $N$ intervals takes $O(N \log N)$ time.
  - The linear merge scan inspects each interval exactly once ($O(N)$).
  - Total time: $O(N \log N) + O(N) = O(N \log N)$.
  - Any algorithm determining general interval overlaps on unsorted inputs can be reduced to element uniqueness, meaning $\Omega(N \log N)$ is the theoretical lower bound.
- For `insert`: Input is pre-sorted, enabling linear $O(N)$ execution.

### Space Complexity: $O(N)$ (Optimal)
- Memory is used to store the sorted array and return the merged results, consuming $O(N)$ auxiliary space.

---

## 7. Key Invariants & Common Pitfalls

1. **Overwriting `prev[1]` with `current[1]`**:
   * *Trap*: Writing `prev[1] = current[1]` fails when the previous interval completely subsumes the current one (e.g. $[1, 10]$ and $[2, 5]$ would become $[1, 5]$). Always use `max(prev[1], current[1])`.
2. **Boundary Points Equality**:
   * Intervals that share a boundary point (e.g. $[1, 2]$ and $[2, 3]$) **do overlap** if endpoints are inclusive ($\le$).

---

## 8. Canonical Problem Walkthrough

### Non-overlapping Intervals (LeetCode #435)
* **Problem**: Minimum number of intervals to remove to make the rest non-overlapping.
* **Greedy Strategy**: Sort by **end time**! Always keep the interval that finishes earliest to leave maximum room for future intervals.

```python
def erase_overlap_intervals(intervals: List[List[int]]) -> int:
    intervals.sort(key=lambda x: x[1])  # Sort by end time
    removals = 0
    prev_end = float('-inf')

    for start, end in intervals:
        if start >= prev_end:
            prev_end = end  # Valid, keep interval
        else:
            removals += 1   # Overlaps, greedily remove current

    return removals
```
* **Optimality**: Runs in $O(N \log N)$ time and $O(1)$ auxiliary space.
