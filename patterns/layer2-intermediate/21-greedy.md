# Pattern 21: Greedy

> **Layer**: 2 (Intermediate High-Yield)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Jump Game](https://leetcode.com/problems/jump-game/), [Jump Game II](https://leetcode.com/problems/jump-game-ii/), [Gas Station](https://leetcode.com/problems/gas-station/), [Hand of Straights](https://leetcode.com/problems/hand-of-straights/)

---

## 1. Mental Model & Core Concept

A Greedy Algorithm makes the **locally optimal choice** at each stage with the guarantee that these local choices lead directly to a **globally optimal solution**.

```text
Greedy Choice Property:
At each state, select the option that looks best RIGHT NOW.
Never reconsider or backtrack on past decisions!

Jump Game Reachability Invariant:
Index:   0    1    2    3    4
Array:  [2 ,  3 ,  1 ,  1 ,  4]
Reach:   2    4    4    4    8
           ▲
At each index i <= max_reach:
max_reach = max(max_reach, i + nums[i])
```

Greedy works when a problem exhibits:
1. **Greedy Choice Property**: A global optimum can be arrived at by selecting a local optimum without looking ahead or backwards.
2. **Optimal Substructure**: An optimal solution to the problem contains optimal solutions to subproblems.

---

## 2. Identification Signals ("When to Use")

- **Reachability / Milestones**: Can you reach the end index? Minimum jumps required.
- **Circuitous Loops**: Gas station circular trip starting position.
- **Sorted Partitions**: Assigning cookies to children, task scheduling.
- **Warning**: If making a local choice restricts or ruins subsequent decisions unpredictably, the problem is **Dynamic Programming**, not Greedy.

---

## 3. Algorithmic Template / Pseudocode

```text
function GREEDY_REACH(nums):
    max_reach = 0
    
    for i from 0 to length(nums) - 1:
        if i > max_reach:
            return false // Stalled: current position is unreachable
        max_reach = max(max_reach, i + nums[i])
        
    return true
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import List

# --- Variant A: Jump Game (Can Reach End) ---
def can_jump(nums: List[int]) -> bool:
    max_reach = 0
    for i, jump in enumerate(nums):
        if i > max_reach:
            return False
        max_reach = max(max_reach, i + jump)
        if max_reach >= len(nums) - 1:
            return True
    return True

# --- Variant B: Jump Game II (Minimum Jumps Required) ---
def jump(nums: List[int]) -> int:
    jumps = 0
    current_end = 0
    farthest = 0

    # Note: loop ends at len(nums) - 1 because we don't jump from the last index
    for i in range(len(nums) - 1):
        farthest = max(farthest, i + nums[i])

        # When we reach the end of the current jump horizon, we MUST take a jump
        if i == current_end:
            jumps += 1
            current_end = farthest

    return jumps

# --- Variant C: Gas Station Circuit ---
def can_complete_circuit(gas: List[int], cost: List[int]) -> int:
    # If total gas is less than total cost, circuit is strictly impossible
    if sum(gas) < sum(cost):
        return -1

    total_tank = 0
    start_station = 0

    for i in range(len(gas)):
        total_tank += gas[i] - cost[i]
        # If tank becomes negative, starting at or before station i is impossible
        if total_tank < 0:
            start_station = i + 1
            total_tank = 0

    return start_station
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `can_jump`:
1. `max_reach = 0`:
   * Tracks the furthest index reachable from the start.
2. `if i > max_reach: return False`:
   * If current index $i$ exceeds `max_reach`, we have encountered an unbridgeable gap (trapped by zeros).
3. `max_reach = max(max_reach, i + jump)`:
   * Greedily extends the frontier to the furthest point reachable from $i$.
4. `if max_reach >= len(nums) - 1: return True`:
   * Early exit as soon as the last index is encompassed.

### Breakdown of `can_complete_circuit`:
1. `if sum(gas) < sum(cost): return -1`:
   * Conservation of energy: if total gas generated along the entire loop is less than fuel consumed, no starting point can complete the circuit.
2. `if total_tank < 0: start_station = i + 1; total_tank = 0`:
   * **The Elimination Invariant**: If you run out of fuel between `start_station` and `i`, then **no station between `start_station` and `i` could have worked either**, because you reached all intermediate stations with $\ge 0$ surplus. We can greedily reset our candidate start to $i + 1$.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(N)$ (Optimal)
- **Proof**:
  - In `can_jump`, `jump`, and `can_complete_circuit`, the array is traversed in a single linear pass.
  - Each element undergoes $O(1)$ scalar comparisons and additions.
  - Total operations $= C \cdot N \implies O(N)$.
  - Any reachability check must inspect array elements, matching the $\Omega(N)$ lower bound.
  - **Comparison**: A DP approach to Jump Game takes $O(N^2)$ time. Greedy drops it to $O(N)$ by eliminating backtracking.

### Space Complexity: $O(1)$ (Optimal)
- Memory is strictly limited to primitive accumulator integers (`max_reach`, `start_station`, `jumps`), requiring zero heap memory.

---

## 7. Key Invariants & Common Pitfalls

1. **Loop Termination in Jump Game II**:
   * *Trap*: Running the loop up to `len(nums)`. If you do that, you trigger an extraneous jump upon arriving at the destination! Always loop up to `len(nums) - 1`.
2. **Greedy Fallacy**:
   * *Trap*: Assuming greedy works without proving the local choice invariant. For 0/1 Knapsack, picking items with the highest value-to-weight ratio greedily fails; DP is required.

---

## 8. Canonical Problem Walkthrough

### Hand of Straights (LeetCode #846)
* **Problem**: Rearrange cards into groups of size `groupSize` where cards are consecutive integers.
* **Greedy Strategy**: Always start forming a group from the **smallest available card** currently in hand!

```python
from collections import Counter

def is_n_straight_hand(hand: List[int], group_size: int) -> bool:
    if len(hand) % group_size != 0:
        return False
    count = Counter(hand)
    for card in sorted(count):
        if count[card] > 0:
            needed = count[card]
            for next_card in range(card, card + group_size):
                if count[next_card] < needed:
                    return False
                count[next_card] -= needed
    return True
```
* **Optimality**: Runs in $O(N \log N)$ time (for sorting keys) and $O(N)$ space.
