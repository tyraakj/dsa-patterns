# Pattern 32: Trie (Prefix Tree)

> **Layer**: 3 (Advanced & Specialized)  
> **Original Curriculum**: [Automedon/ultimate-leetcode-patterns](https://github.com/Automedon/ultimate-leetcode-patterns)  
> **Key Problems**: [Implement Trie (Prefix Tree)](https://leetcode.com/problems/implement-trie-prefix-tree/), [Design Add and Search Words Data Structure](https://leetcode.com/problems/design-add-and-search-words-data-structure/), [Word Search II](https://leetcode.com/problems/word-search-ii/), [Maximum XOR of Two Numbers in an Array](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/)

---

## 1. Mental Model & Core Concept

A Trie is an $M$-ary tree where edges (or child links) represent characters in an alphabet. Nodes sharing a common prefix branch off a common ancestral path in memory.

```text
Trie Structure for ["app", "apple", "apply"]:
             (root)
               │ 'a'
              ( )
               │ 'p'
              ( )
               │ 'p'
             (is_end: True) ──► "app"
            /     \
      'l'  /       \ 'l'
         ( )       ( )
          │ 'e'     │ 'y'
     (is_end: T) (is_end: T)
       "apple"     "apply"
```

Core Advantages over Hash Sets:
- Checking if any word starts with a prefix takes **$O(L)$** time (where $L$ is prefix length), independent of the number of words $N$ in the dictionary!
- Enables wildcards (e.g. `b.d` matches `bad`, `bed`, `bid`) and bitwise binary tries for Maximum XOR.

---

## 2. Identification Signals ("When to Use")

- **Prefix Lookups**: Autocomplete, `startsWith(prefix)`.
- **Wildcard / Regular Expression Dictionary Matching**: Searching words with `.` wildcard characters.
- **Grid Word Searches**: Validating whether a grid traversal path is a prefix of any dictionary word (*Word Search II*).
- **Bitwise XOR Optimization**: Finding maximum XOR between numbers in an array in $O(32 \cdot N)$.

---

## 3. Algorithmic Template / Pseudocode

```text
class TrieNode:
    children = hash map or array of size 26
    is_word = false

class Trie:
    root = TrieNode()

    function insert(word):
        curr = root
        for char in word:
            if char not in curr.children:
                curr.children[char] = TrieNode()
            curr = curr.children[char]
        curr.is_word = true

    function startsWith(prefix):
        curr = root
        for char in prefix:
            if char not in curr.children:
                return false
            curr = curr.children[char]
        return true
```

---

## 4. Python Semi-Code / Idiomatic Implementation

```python
from typing import List, Dict

# --- Variant A: Standard Trie Implementation ---
class TrieNode:
    def __init__(self):
        self.children: Dict[str, 'TrieNode'] = {}
        self.is_word = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        curr = self.root
        for char in word:
            if char not in curr.children:
                curr.children[char] = TrieNode()
            curr = curr.children[char]
        curr.is_word = True

    def search(self, word: str) -> bool:
        curr = self.root
        for char in word:
            if char not in curr.children:
                return False
            curr = curr.children[char]
        return curr.is_word

    def starts_with(self, prefix: str) -> bool:
        curr = self.root
        for char in prefix:
            if char not in curr.children:
                return False
            curr = curr.children[char]
        return True

# --- Variant B: Trie + Backtracking (Word Search II) ---
def find_words(board: List[List[str]], words: List[str]) -> List[str]:
    root = TrieNode()
    # Build trie with full word stored at leaf for instant retrieval
    for w in words:
        curr = root
        for c in w:
            if c not in curr.children:
                curr.children[c] = TrieNode()
            curr = curr.children[c]
        curr.word = w  # Store terminal word

    rows, cols = len(board), len(board[0])
    res = []

    def dfs(r: int, c: int, parent_node: TrieNode) -> None:
        char = board[r][c]
        if char not in parent_node.children:
            return

        curr_node = parent_node.children[char]
        if hasattr(curr_node, 'word') and curr_node.word:
            res.append(curr_node.word)
            curr_node.word = None  # Deduplicate: avoid re-adding same word

        board[r][c] = '#'  # Mark visited
        for dr, dc in [(1, 0), (-1, 0), (0, 1), (0, -1)]:
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols and board[nr][nc] != '#':
                dfs(nr, nc, curr_node)
        board[r][c] = char  # Backtrack

    for r in range(rows):
        for c in range(cols):
            dfs(r, c, root)

    return res
```

---

## 5. Line-by-Line Code Breakdown

### Breakdown of `insert`:
1. `curr = self.root`:
   * Begins traversal at the root.
2. `if char not in curr.children: curr.children[char] = TrieNode()`:
   * Dynamically allocates child node only when branching for a new prefix.
3. `curr.is_word = True`:
   * Flags that a complete dictionary word terminates at this specific node (e.g. distinguishing `"app"` from `"apple"`).

### Breakdown of `Word Search II`:
1. `curr_node.word = None`:
   * Eliminates the need for a separate `set()` to deduplicate words found in the grid.
2. `if char not in parent_node.children: return`:
   * **Massive Pruning Invariant**: If the prefix constructed so far does not exist anywhere in the dictionary, the search aborts immediately, pruning entire exponential exploration branches!

---

## 6. Mathematical Proof of Time & Space Optimality

### Time Complexity: $O(L)$ per Word Operation (Optimal)
- Inserting, searching, or checking prefixes takes strictly $L$ steps, where $L$ is the length of the string.
- This is completely independent of the total number of words $N$ in the trie.
- Lower bound: In any model, every character of word $L$ must be evaluated, making $O(L)$ optimal.

### Space Complexity: $O(\Sigma \cdot L \cdot N)$ (Optimal)
- In the worst case with no common prefixes, storing $N$ words of length $L$ over an alphabet of size $\Sigma$ requires $O(\Sigma \cdot L \cdot N)$ space. In practice, shared prefixes drastically compress memory.

---

## 7. Key Invariants & Common Pitfalls

1. **Distinguishing Word End vs Prefix**:
   * A node may exist in the trie because of `"apple"`, but `search("app")` must return `False` unless `curr.is_word` is set to `True`.
2. **Trie Pruning in Backtracking**:
   * For optimal performance in *Word Search II*, prune empty leaf nodes when backtracking to prevent re-searching dead branches.

---

## 8. Canonical Problem Walkthrough

### Maximum XOR of Two Numbers in an Array (LeetCode #421)
* **Problem**: Find the maximum result of `nums[i] ^ nums[j]`.
* **Binary Bitwise Trie**: Insert 32-bit binary representations of all numbers. For each number, greedily navigate down the **opposite bit** branch ($1 \to 0$ or $0 \to 1$) to maximize the XOR output!
* **Optimality**: Operates in $O(32 \cdot N) = O(N)$ time and $O(32 \cdot N) = O(N)$ space, outperforming $O(N^2)$ brute force pairs.
