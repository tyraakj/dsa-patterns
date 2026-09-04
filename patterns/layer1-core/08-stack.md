# Pattern 08: Stack

> **Layer**: 1 (Core Foundation)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/), [Min Stack](https://leetcode.com/problems/min-stack/), [Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/), [Daily Temperatures](https://leetcode.com/problems/daily-temperatures/)

---

## 1. Mental Model & Core Concept

A Stack enforces a **Last-In, First-Out (LIFO)** access discipline. It is the fundamental data structure for processing **nested, hierarchical, or reversible relationships**.

```text
Stack LIFO Mechanism:
Push '('  ──► [ '(' ]
Push '{'  ──► [ '(', '{' ]
Match '}' ──► Pop '{' (Most recently opened must be closed first!)
Push '['  ──► [ '(', '[' ]
```

Core applications:
1. **Bracket Matching / Syntax Validation**: The most recently opened delimiter must be the first closed.
2. **Expression Evaluation & AST Traversal**: Maintaining operator precedence and operands in Polish/Reverse Polish notations.
3. **State History / Undo Operations**: Tracking past states to support backtracking or constant-time queries (e.g. Min Stack).

---

## 2. Identification Signals ("When to Use")

- **Nested Structures**: Parentheses, brackets, XML/HTML tags, nested directory paths (`/a/./b/../../c/`).
- **Reverse Polish Notation (RPN)**: Evaluating postfix mathematical expressions.
- **Immediate Reversal**: Cancelling adjacent matching elements (e.g., remove all adjacent duplicates in string).
- **Retrieving Min/Max in $O(1)$**: Pairing elements with running extrema.

---

## 3. Algorithmic Template / Pseudocode

```text
function VALID_PARENTHESES(s):
    stack = empty stack
    mapping = {')': '(', '}': '{', ']': '['}
    
    for char in s:
        if char is closing bracket in mapping:
            if stack is empty or stack.pop() != mapping[char]:
                return false
        else:
            stack.push(char)
            
    return stack is empty
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import List

# --- Variant A: Valid Parentheses ---
def is_valid(s: str) -> bool:
    stack = []
    bracket_map = {')': '(', '}': '{', ']': '['}

    for char in s:
        if char in bracket_map:
            top_element = stack.pop() if stack else '#'
            if bracket_map[char] != top_element:
                return False
        else:
            stack.append(char)

    return len(stack) == 0

# --- Variant B: Min Stack with O(1) Minimum Retrieval ---
class MinStack:
    def __init__(self):
        # Stack stores tuples: (current_val, running_minimum)
        self.stack = []

    def push(self, val: int) -> None:
        if not self.stack:
            self.stack.append((val, val))
        else:
            curr_min = min(val, self.stack[-1][1])
            self.stack.append((val, curr_min))

    def pop(self) -> None:
        if self.stack:
            self.stack.pop()

    def top(self) -> int:
        return self.stack[-1][0] if self.stack else -1

    def get_min(self) -> int:
        return self.stack[-1][1] if self.stack else -1

# --- Variant C: Evaluate Reverse Polish Notation (RPN) ---
def eval_rpn(tokens: List[str]) -> int:
    stack = []
    for token in tokens:
        if token in {"+", "-", "*", "/"}:
            b = stack.pop()
            a = stack.pop()
            if token == "+":
                stack.append(a + b)
            elif token == "-":
                stack.append(a - b)
            elif token == "*":
                stack.append(a * b)
            elif token == "/":
                # Integer division truncating toward zero
                stack.append(int(a / b))
        else:
            stack.append(int(token))
            
    return stack[0]
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `is_valid`:
1. `bracket_map = {')': '(', '}': '{', ']': '['}`:
   * Maps each closing bracket to its expected matching opening bracket.
2. `for char in s:`:
   * Scans string characters from left to right.
3. `if char in bracket_map:`:
   * Checks if `char` is a closing bracket.
4. `top_element = stack.pop() if stack else '#'`:
   * Pops top of stack. If stack is empty when a closing bracket appears, uses dummy character `'#'` to trigger immediate failure.
5. `if bracket_map[char] != top_element: return False`:
   * Validates delimiter symmetry.
6. `else: stack.append(char)`:
   * If character is an opening bracket, pushes onto stack.
7. `return len(stack) == 0`:
   * Ensures no unclosed opening brackets linger after traversal (e.g. `"((("`).

### Breakdown of `MinStack`:
1. `self.stack.append((val, curr_min))`:
   * Pairs each value with the minimum element in the stack up to that point. When values are popped, the previous minimum automatically becomes active again without needing recomputation.

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(N)$ (Optimal)
- **Proof**:
  - In `is_valid` and `eval_rpn`, every element is pushed onto the stack exactly once and popped at most once.
  - Total stack push and pop operations $\le 2N$.
  - Each push and pop on a Python dynamic array (`list`) is amortized $O(1)$.
  - Total time complexity is strictly $O(N)$.
- **Optimality**: Inspecting every bracket or token is mandatory to verify syntax, matching the theoretical $\Omega(N)$ lower bound.

### Space Complexity: $O(N)$ (Optimal)
- In the worst case (e.g., all opening brackets `"((((("`), the stack stores all $N$ elements, consuming $O(N)$ memory. This is optimal for arbitrary nesting depth.

---

## 7. Key Invariants & Common Pitfalls

1. **Stack Emptiness Underflow**:
   * Calling `stack.pop()` on an empty stack raises `IndexError: pop from empty list`. Always guard with `if stack:` or handle with a fallback value.
2. **Integer Division Truncation in RPN**:
   * *Trap*: `a // b` in Python floors toward $-\infty$ (e.g. `6 // -132 = -1`), whereas LeetCode demands truncation toward zero (`int(6 / -132) = 0`). Use `int(a / b)`.
3. **Operand Order in Non-Commutative Operations**:
   * When evaluating subtraction or division, the first popped element is the right operand `b`, and the second popped element is the left operand `a`. Swapping them (`b - a` instead of `a - b`) yields erroneous results.

---

## 8. Canonical Problem Walkthrough

### Simplify Path (LeetCode #71)
* **Problem**: Convert an absolute Unix file path into its canonical normalized form.
* **Stack Logic**:
```python
def simplify_path(path: str) -> str:
    stack = []
    # Split path by '/' to eliminate duplicate slashes
    for part in path.split("/"):
        if part == "..":
            if stack:
                stack.pop()
        elif part and part != ".":
            stack.append(part)
            
    return "/" + "/".join(stack)
```
* **Optimality**: Operates in $O(N)$ time and $O(N)$ space.
