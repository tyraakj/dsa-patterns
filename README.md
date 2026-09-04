# Ultimate LeetCode Patterns: Study Guide & Algorithmic Reference

> **Attribution & Source Material**:  
> This study guide, classification framework, and curated problem set is based directly on the acclaimed [Ultimate LeetCode Patterns](https://github.com/Automedon/ultimate-leetcode-patterns) curriculum created by **Vasily Melnik** ([@Automedon](https://github.com/Automedon)).  
> All 41 patterns are organized in descending order of real-world technical interview frequency (ROI), featuring 820 accessible, non-premium LeetCode problems with comprehensive notes, algorithmic templates, and production-grade Python implementations.

---

## 🎯 Master Triage: Problem Keyword Cheat Sheet

When solving problems under time pressure in technical interviews, match problem signals and keywords to the optimal pattern:

| Problem Signal / Clue | Pattern Name | Layer | Optimal Time | Space | Primary File |
| :--- | :--- | :---: | :---: | :---: | :--- |
| Contiguous subarray, fixed or at-most-k size, substring window | **Sliding Window** | 1 | $O(N)$ | $O(1)$ / $O(K)$ | [02-sliding-window.md](patterns/layer1-core/02-sliding-window.md) |
| Sorted array, two sum, palindromes, opposite ends moving inward | **Two Pointers** | 1 | $O(N)$ | $O(1)$ | [03-two-pointers.md](patterns/layer1-core/03-two-pointers.md) |
| Fast lookup, frequency count, anagram grouping, pair difference | **Hash Map / Set** | 1 | $O(N)$ | $O(N)$ | [01-hash-map-set.md](patterns/layer1-core/01-hash-map-set.md) |
| Sorted array/matrix, $O(\log N)$ constraint, "min of max" answer range | **Binary Search** | 1 | $O(\log N)$ | $O(1)$ | [04-binary-search.md](patterns/layer1-core/04-binary-search.md) |
| Range sum queries, subarray sum equals $K$, running parity checks | **Prefix Sum** | 1 | $O(N)$ | $O(N)$ / $O(1)$ | [05-prefix-sum.md](patterns/layer1-core/05-prefix-sum.md) |
| Next greater/smaller element, histogram area, stock span | **Monotonic Stack** | 1 | $O(N)$ | $O(N)$ | [09-monotonic-stack-queue.md](patterns/layer1-core/09-monotonic-stack-queue.md) |
| Balanced parentheses, nested structures, reverse Polish notation | **Stack** | 1 | $O(N)$ | $O(N)$ | [08-stack.md](patterns/layer1-core/08-stack.md) |
| Tree path sums, diameter, tree validation, maximum depth | **Tree DFS** | 1 | $O(V)$ | $O(H)$ | [11-tree-dfs.md](patterns/layer1-core/11-tree-dfs.md) |
| Level-by-level traversal, shortest path in tree, right side view | **Tree BFS** | 1 | $O(V)$ | $O(W)$ | [12-tree-bfs.md](patterns/layer1-core/12-tree-bfs.md) |
| Combinations, permutations, subsets, exhaustive search with pruning | **Backtracking** | 1 | $O(2^N)$ / $O(N!)$ | $O(N)$ | [15-backtracking.md](patterns/layer1-core/15-backtracking.md) |
| Optimal substructure, overlapping subproblems, max/min profit | **1D Dynamic Programming** | 1 | $O(N)$ | $O(N)$ / $O(1)$ | [13-dynamic-programming-1d.md](patterns/layer1-core/13-dynamic-programming-1d.md) |
| Connected components, cycle detection, bipartite graphs | **Graph DFS** | 2 | $O(V + E)$ | $O(V)$ | [16-graph-dfs.md](patterns/layer2-intermediate/16-graph-dfs.md) |
| Shortest path in unweighted graph, multi-source rotten oranges | **Graph BFS** | 2 | $O(V + E)$ | $O(V)$ | [17-graph-bfs.md](patterns/layer2-intermediate/17-graph-bfs.md) |
| Number of islands, pacman/flood fill, boundary matrix traversal | **Matrix / Grid** | 2 | $O(R \times C)$ | $O(R \times C)$ | [18-matrix-grid-traversal.md](patterns/layer2-intermediate/18-matrix-grid-traversal.md) |
| Top $K$ elements, running median, merging sorted streams | **Heap / Priority Queue** | 2 | $O(N \log K)$ | $O(K)$ | [19-heap-priority-queue.md](patterns/layer2-intermediate/19-heap-priority-queue.md) |
| Overlapping ranges, meeting rooms, interval insertions/merges | **Intervals** | 2 | $O(N \log N)$ | $O(N)$ / $O(1)$ | [20-intervals.md](patterns/layer2-intermediate/20-intervals.md) |
| Local optimal choice leads to global optimal, jump game | **Greedy** | 2 | $O(N)$ | $O(1)$ | [21-greedy.md](patterns/layer2-intermediate/21-greedy.md) |
| 2D grid paths, edit distance, longest common subsequence | **2D Dynamic Programming** | 2 | $O(M \times N)$ | $O(M \times N)$ / $O(N)$ | [22-dynamic-programming-2d-grid.md](patterns/layer2-intermediate/22-dynamic-programming-2d-grid.md) |
| Cycle detection in linked list, middle node, happy number | **Fast & Slow Pointers** | 2 | $O(N)$ | $O(1)$ | [24-fast-slow-pointers.md](patterns/layer2-intermediate/24-fast-slow-pointers.md) |
| Reverse linked list in-place, reverse in $K$-groups, reorder list | **Linked List Reversal** | 2 | $O(N)$ | $O(1)$ | [25-linked-list-reversal.md](patterns/layer2-intermediate/25-linked-list-reversal.md) |
| Inorder traversal sorted, BST search, lowest common ancestor in BST | **Binary Search Tree** | 2 | $O(H)$ | $O(H)$ | [26-binary-search-tree.md](patterns/layer2-intermediate/26-binary-search-tree.md) |
| Bitwise XOR cancellation, subset bitmask, power of two | **Bit Manipulation** | 2 | $O(1)$ / $O(N)$ | $O(1)$ | [27-bit-manipulation.md](patterns/layer2-intermediate/27-bit-manipulation.md) |
| Subset sum, coin change, item selection under weight capacity | **DP Knapsack** | 2 | $O(N \times W)$ | $O(W)$ | [28-dp-knapsack.md](patterns/layer2-intermediate/28-dp-knapsack.md) |
| LRU cache, LFU cache, design data structures with $O(1)$ constraints | **Design Problems** | 2 | $O(1)$ ops | $O(N)$ | [29-design-problems.md](patterns/layer2-intermediate/29-design-problems.md) |
| Course prerequisites, build order, DAG dependency resolution | **Topological Sort** | 2 | $O(V + E)$ | $O(V + E)$ | [30-topological-sort.md](patterns/layer2-intermediate/30-topological-sort.md) |
| Dynamic connectivity, redundancy detection in graph edges | **Union-Find (DSU)** | 3 | $O(N \cdot \alpha(N))$ | $O(N)$ | [31-union-find.md](patterns/layer3-advanced/31-union-find.md) |
| Prefix search, autocomplete, dictionary word search | **Trie** | 3 | $O(L)$ per op | $O(\Sigma \cdot L \cdot N)$ | [32-trie.md](patterns/layer3-advanced/32-trie.md) |
| Numbers in range $[1, N]$, find duplicate or missing numbers | **Cyclic Sort** | 3 | $O(N)$ | $O(1)$ | [33-cyclic-sort.md](patterns/layer3-advanced/33-cyclic-sort.md) |
| Continuous sliding window max/min over moving frames | **Sliding Window Max/Min** | 3 | $O(N)$ | $O(K)$ | [35-sliding-window-max-min.md](patterns/layer3-advanced/35-sliding-window-max-min.md) |
| Dynamic continuous median tracking | **Two Heaps** | 3 | $O(\log N)$ | $O(N)$ | [37-two-heaps.md](patterns/layer3-advanced/37-two-heaps.md) |
| Range sum/min/max with dynamic point/range updates | **Segment Tree** | 3 | $O(\log N)$ | $O(N)$ | [40-segment-tree.md](patterns/layer3-advanced/40-segment-tree.md) |
| Prefix sums with fast point updates | **Binary Indexed Tree** | 3 | $O(\log N)$ | $O(N)$ | [41-binary-indexed-tree.md](patterns/layer3-advanced/41-binary-indexed-tree.md) |

---

## 📚 Complete Curriculum Index (41 Patterns)

### 🥇 Layer 1: Core Foundation (Patterns 01–15)
*Mastery here unlocks ~80% of standard software engineering interview loops.*

| # | Pattern Name | Focus Areas & Concepts | Guide Link |
| :-: | :--- | :--- | :--- |
| **01** | **Hash Map / Hash Set** | Complements, frequency counting, grouping, coordinate hashing | [01-hash-map-set.md](patterns/layer1-core/01-hash-map-set.md) |
| **02** | **Sliding Window** | Contiguous subarrays, dynamic expand/shrink loops | [02-sliding-window.md](patterns/layer1-core/02-sliding-window.md) |
| **03** | **Two Pointers** | Inward converging, sorted pairs, partition markers | [03-two-pointers.md](patterns/layer1-core/03-two-pointers.md) |
| **04** | **Binary Search** | Search space pruning, monotonic predicates, answer ranges | [04-binary-search.md](patterns/layer1-core/04-binary-search.md) |
| **05** | **Prefix Sum** | Constant-time range queries, hash-indexed prefix differences | [05-prefix-sum.md](patterns/layer1-core/05-prefix-sum.md) |
| **06** | **String Manipulation & Parsing** | Token parsing, two-pointer reversal, state-machine scanning | [06-string-manipulation-parsing.md](patterns/layer1-core/06-string-manipulation-parsing.md) |
| **07** | **String Matching** | Rolling hash (Rabin-Karp), KMP prefix arrays, rolling window | [07-string-matching.md](patterns/layer1-core/07-string-matching.md) |
| **08** | **Stack** | LIFO ordering, syntax validation, nested block evaluation | [08-stack.md](patterns/layer1-core/08-stack.md) |
| **09** | **Monotonic Stack / Queue** | Next Greater/Smaller Element, boundary expansion, histogram area | [09-monotonic-stack-queue.md](patterns/layer1-core/09-monotonic-stack-queue.md) |
| **10** | **Queue / Deque** | FIFO buffers, sliding frame queues, circular buffers | [10-queue-deque.md](patterns/layer1-core/10-queue-deque.md) |
| **11** | **Tree DFS** | Preorder, Inorder, Postorder, bottom-up tree diameter | [11-tree-dfs.md](patterns/layer1-core/11-tree-dfs.md) |
| **12** | **Tree BFS** | Level-order snapshots, zigzag, boundary views | [12-tree-bfs.md](patterns/layer1-core/12-tree-bfs.md) |
| **13** | **Dynamic Programming (1D)** | Recurrence state relations, subproblem memoization, $O(1)$ space | [13-dynamic-programming-1d.md](patterns/layer1-core/13-dynamic-programming-1d.md) |
| **14** | **Recursion / Divide & Conquer** | Binary subproblem splitting, merge paradigms, exponentiation | [14-recursion-divide-conquer.md](patterns/layer1-core/14-recursion-divide-conquer.md) |
| **15** | **Backtracking** | State trees, constraint checks, decision pruning | [15-backtracking.md](patterns/layer1-core/15-backtracking.md) |

---

### 🥈 Layer 2: Intermediate High-Yield (Patterns 16–30)
*Advanced graphs, 2D dynamic programming, heaps, intervals, and data structure design.*

| # | Pattern Name | Focus Areas & Concepts | Guide Link |
| :-: | :--- | :--- | :--- |
| **16** | **Graph DFS** | Connected components, cycle detection, bipartite testing | [16-graph-dfs.md](patterns/layer2-intermediate/16-graph-dfs.md) |
| **17** | **Graph BFS** | Unweighted shortest paths, multi-source wave propagation | [17-graph-bfs.md](patterns/layer2-intermediate/17-graph-bfs.md) |
| **18** | **Matrix / Grid Traversal** | Directional arrays, in-bounds validation, flood-fill | [18-matrix-grid-traversal.md](patterns/layer2-intermediate/18-matrix-grid-traversal.md) |
| **19** | **Heap / Priority Queue** | Min/Max heap state, Top $K$ elements, dynamic order tracking | [19-heap-priority-queue.md](patterns/layer2-intermediate/19-heap-priority-queue.md) |
| **20** | **Intervals** | Sorting endpoints, interval merging, intersection logic | [20-intervals.md](patterns/layer2-intermediate/20-intervals.md) |
| **21** | **Greedy** | Local optima choice, sorting preconditions, jump limits | [21-greedy.md](patterns/layer2-intermediate/21-greedy.md) |
| **22** | **Dynamic Programming (2D / Grid)** | Grid path transitions, LCS, Edit Distance, matrix DP | [22-dynamic-programming-2d-grid.md](patterns/layer2-intermediate/22-dynamic-programming-2d-grid.md) |
| **23** | **Sorting** | Custom comparators, Dutch National Flag, bucket/radix sort | [23-sorting.md](patterns/layer2-intermediate/23-sorting.md) |
| **24** | **Fast & Slow Pointers** | Floyd's cycle detection, linked list midpoint, circular lists | [24-fast-slow-pointers.md](patterns/layer2-intermediate/24-fast-slow-pointers.md) |
| **25** | **Linked List Reversal** | In-place pointer mutation, $K$-group reversal, sentinel dummy nodes | [25-linked-list-reversal.md](patterns/layer2-intermediate/25-linked-list-reversal.md) |
| **26** | **Binary Search Tree (BST)** | BST ordering invariant, validate BST, Inorder successor | [26-binary-search-tree.md](patterns/layer2-intermediate/26-binary-search-tree.md) |
| **27** | **Bit Manipulation** | Bit masking, XOR properties, clearing low-bit `n & (n - 1)` | [27-bit-manipulation.md](patterns/layer2-intermediate/27-bit-manipulation.md) |
| **28** | **DP Knapsack Style** | 0/1 Knapsack, Unbounded Knapsack, Subset Partition | [28-dp-knapsack.md](patterns/layer2-intermediate/28-dp-knapsack.md) |
| **29** | **Design Problems** | LRU/LFU cache, Doubly Linked List + Hash Map combinations | [29-design-problems.md](patterns/layer2-intermediate/29-design-problems.md) |
| **30** | **Topological Sort** | Kahn's Algorithm (in-degree array + queue), DFS postorder | [30-topological-sort.md](patterns/layer2-intermediate/30-topological-sort.md) |

---

### 🥉 Layer 3: Advanced & Specialized (Patterns 31–41)
*Specialized data structures and competitive algorithmic paradigms.*

| # | Pattern Name | Focus Areas & Concepts | Guide Link |
| :-: | :--- | :--- | :--- |
| **31** | **Union-Find (Disjoint Set)** | Path compression, union-by-rank, dynamic connectivity | [31-union-find.md](patterns/layer3-advanced/31-union-find.md) |
| **32** | **Trie (Prefix Tree)** | Character matrix nodes, prefix lookups, wildcard search | [32-trie.md](patterns/layer3-advanced/32-trie.md) |
| **33** | **Cyclic Sort** | Index placement $[0, N-1]$ or $[1, N]$ without auxiliary space | [33-cyclic-sort.md](patterns/layer3-advanced/33-cyclic-sort.md) |
| **34** | **Math / Number Theory** | Euclidean GCD, Sieve of Eratosthenes, fast modular power | [34-math-number-theory.md](patterns/layer3-advanced/34-math-number-theory.md) |
| **35** | **Sliding Window Max / Min** | Monotonic double-ended queue, extreme value window frames | [35-sliding-window-max-min.md](patterns/layer3-advanced/35-sliding-window-max-min.md) |
| **36** | **Simulation / Rearrangement** | Matrix spirals, cellular automata (Game of Life), state flags | [36-simulation-array-rearrangement.md](patterns/layer3-advanced/36-simulation-array-rearrangement.md) |
| **37** | **Two Heaps** | Dual min/max heaps balancing median streaming | [37-two-heaps.md](patterns/layer3-advanced/37-two-heaps.md) |
| **38** | **K-way Merge** | Min-heap tracking $K$ sorted streams concurrently | [38-k-way-merge.md](patterns/layer3-advanced/38-k-way-merge.md) |
| **39** | **Game Theory / Minimax** | Zero-sum adversarial games, state transitions, memoization | [39-game-theory-minimax.md](patterns/layer3-advanced/39-game-theory-minimax.md) |
| **40** | **Segment Tree** | Binary tree segment intervals, $O(\log N)$ range queries and updates | [40-segment-tree.md](patterns/layer3-advanced/40-segment-tree.md) |
| **41** | **Binary Indexed Tree (Fenwick)** | Lowbit bitwise navigation, running prefix sums, point updates | [41-binary-indexed-tree.md](patterns/layer3-advanced/41-binary-indexed-tree.md) |

---

## ⚡ 5-Step Interview Execution Protocol

1. **Clarify Inputs & Edge Cases (First 3–5 Minutes)**:
   * Ask about duplicate elements, negative numbers, empty arrays, null pointers, and sortedness.
   * State expected input bounds ($N \le 10^5 \implies O(N)$ or $O(N \log N)$; $N \le 20 \implies O(2^N)$ backtracking).
2. **State the Brute Force Solution Loudly**:
   * Mention the baseline $O(N^2)$ or $O(2^N)$ solution to establish correctness before diving into optimization.
3. **Pattern Match & Announce the State Invariant**:
   * *"Since the array is sorted and we are looking for a pair sum, we can maintain two inward pointers with an invariant that left only moves right and right only moves left."*
4. **Code Cleanly with Idiomatic Python**:
   * Use clean names (`left`, `right`, `seen`, `curr_sum`), leverage standard libraries (`collections.defaultdict`, `collections.deque`, `heapq`), and avoid unnecessary variables.
5. **Dry Run on Edge Cases Before Saying 'Done'**:
   * Test on: (1) Empty / single element, (2) All duplicates, (3) Target not found, (4) Minimum and maximum possible values.
