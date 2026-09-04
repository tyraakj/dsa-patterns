# Pattern 39: Game Theory / Minimax

> **Layer**: 3 (Advanced & Specialized)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Nim Game](https://leetcode.com/problems/nim-game/), [Stone Game](https://leetcode.com/problems/stone-game/), [Predict the Winner](https://leetcode.com/problems/predict-the-winner/), [Cat and Mouse](https://leetcode.com/problems/cat-and-mouse/)

---

## 1. Mental Model & Core Concept

Minimax solves **two-player, zero-sum games with perfect information**. Both players play optimally:
- The **Maximizer** seeks the highest score.
- The **Minimizer** seeks to minimize the Maximizer's score (or maximize their own advantage).

```text
Game State Recurrence:
score(state) = max(
    choice_1_gain - score(resulting_state_1),
    choice_2_gain - score(resulting_state_2)
)
Subtracting the opponent's optimal future score implements minimax
naturally through a single symmetric perspective!
```

Fundamental Theorems:
1. **Nim-Sum (Bouton's Theorem)**: In impartial removal games, a position is a winning position if and only if the bitwise XOR sum of pile sizes is non-zero ($\bigoplus \text{piles} \neq 0$).
2. **First-Player Parity Advantage**: In problems like *Stone Game*, the first player can always choose all even or all odd piles, guaranteeing victory if sum is odd.

---

## 2. Identification Signals ("When to Use")

- **Alternating Player Turns**: "Alice and Bob take turns...", "Predict the winner".
- **Optimal Play**: Both players make moves that maximize their own probability of winning.
- **Score Differentials**: Seeking whether Player 1 can achieve score $\ge$ Player 2.

---

## 3. Algorithmic Template / Pseudocode

```text
function MINIMAX_SCORE_DIFF(state, memo):
    if state is terminal:
        return 0
    if state in memo:
        return memo[state]

    best_diff = -INFINITY
    for each valid move:
        gain = score_from(move)
        opponent_best = MINIMAX_SCORE_DIFF(next_state, memo)
        best_diff = max(best_diff, gain - opponent_best)

    memo[state] = best_diff
    return best_diff
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import List, Dict, Tuple

# --- Variant A: Predict the Winner / Stone Game ---
def predict_the_winner(nums: List[int]) -> bool:
    memo: Dict[Tuple[int, int], int] = {}

    def max_diff(left: int, right: int) -> int:
        if left == right:
            return nums[left]
        if (left, right) in memo:
            return memo[(left, right)]

        # Pick left pile OR pick right pile, minus opponent's optimal differential
        pick_left = nums[left] - max_diff(left + 1, right)
        pick_right = nums[right] - max_diff(left, right - 1)

        memo[(left, right)] = max(pick_left, pick_right)
        return memo[(left, right)]

    # Player 1 wins if their net point differential >= 0
    return max_diff(0, len(nums) - 1) >= 0

# --- Variant B: Nim Game (O(1) Mathematical Deduction) ---
def can_win_nim(n: int) -> bool:
    # Any multiple of 4 is a forced loss; any non-multiple of 4 is a forced win!
    return n % 4 != 0

# --- Variant C: Can I Win (Bitmask DP with Minimax) ---
def can_i_win(max_choosable_integer: int, desired_total: int) -> bool:
    if desired_total <= 0:
        return True
    if (max_choosable_integer * (max_choosable_integer + 1)) // 2 < desired_total:
        return False

    memo: Dict[int, bool] = {}

    def dfs(used_mask: int, remaining: int) -> bool:
        if used_mask in memo:
            return memo[used_mask]

        for i in range(1, max_choosable_integer + 1):
            bit = 1 << i
            if not (used_mask & bit):
                # If picking i directly wins OR forces opponent into a losing state
                if i >= remaining or not dfs(used_mask | bit, remaining - i):
                    memo[used_mask] = True
                    return True

        memo[used_mask] = False
        return False

    return dfs(0, desired_total)
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `predict_the_winner`:
1. `pick_left = nums[left] - max_diff(left + 1, right)`:
   * **The Zero-Sum Subtraction**: By picking `nums[left]`, the player gains points, but the opponent will subsequently play optimally over the subproblem `[left + 1, right]`. Subtracting the opponent's best differential accurately models their optimal resistance.
2. `memo[(left, right)] = max(pick_left, pick_right)`:
   * Memoizes results for the sub-interval $[left, right]$ to prevent exponential $O(2^N)$ recalculation.

### Breakdown of `can_i_win`:
1. `bit = 1 << i; if not (used_mask & bit):`:
   * Uses an integer bitmask to track which numbers from $1$ to $max\_choosable$ have already been claimed.
2. `if i >= remaining or not dfs(used_mask | bit, remaining - i):`:
   * If current pick reaches target, win immediately. Otherwise, if the resulting state causes the opponent's search to return `False`, this pick guarantees victory.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity:
- **`predict_the_winner`**: There are $\frac{N(N+1)}{2}$ sub-intervals $[left, right]$. Each state evaluates 2 choices in $O(1)$ time. Total runtime: $O(N^2)$, reduced from $O(2^N)$ naive recursion.
- **`can_win_nim`**: Modulo arithmetic executes in single-cycle $O(1)$ time.
- **`can_i_win`**: State space size is $2^M$ where $M = \text{max\_choosable} \le 20$. Total states: $2^{20} \approx 10^6 \implies O(M \cdot 2^M)$.

### Space Complexity:
- `predict_the_winner`: $O(N^2)$ memoization dictionary.
- `can_i_win`: $O(2^M)$ memoization table.

---

## 7. Key Invariants & Common Pitfalls

1. **Symmetric Perspective Inversion**:
   * Instead of alternating between `maximize()` and `minimize()` functions, use `my_score - opponent_score(next_state)`. This halves code complexity while preserving mathematical rigor.
2. **Impossible Desired Total Guard**:
   * If the sum of all available numbers $\frac{M(M+1)}{2} < \text{desired}$, neither player can win; return `False` immediately before launching DP.

---

## 8. Canonical Problem Walkthrough

### Stone Game (LeetCode #877)
* **Problem**: Piles of stones with odd total sum. First player picks from either end.
* **Mathematical Invariant**: The first player can calculate the sum of all odd-indexed piles vs even-indexed piles. One of these sums is strictly greater. By picking the first pile, Player 1 can force the game so they collect **all** piles of the winning parity!
* **Solution**:
```python
def stone_game(piles: List[int]) -> bool:
    return True  # First player is mathematically guaranteed to win!
```
* **Optimality**: Operates in $O(1)$ time and $O(1)$ space.
