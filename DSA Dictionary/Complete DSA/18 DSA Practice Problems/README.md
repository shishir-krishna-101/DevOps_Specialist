# 18 — DSA Practice Problems

This chapter is a structured practice bank for applying the concepts in **Complete DSA**. The problems are original learning prompts written for this repository. They are not copied from a commercial interview platform.

The goal is to practice the full engineering process:

```text
Understand the contract
        ↓
Work through examples
        ↓
Build a correct baseline
        ↓
Choose a suitable pattern
        ↓
Implement and test
        ↓
Analyze time and space
        ↓
Explain the trade-offs
```

> **How to use this chapter:** Solve each problem before reading its hints. Do not treat a hint as a complete solution. Record your approach, complexity, edge cases, and what you learned.

---

## 1. Practice rules

For every problem, create a short solution note containing:

1. **Problem summary:** Restate the task in your own words.
2. **Input contract:** Types, valid ranges, ordering, duplicates, and empty-input behavior.
3. **Output contract:** Exact return value and how no-solution cases are represented.
4. **Examples:** Trace at least one normal case and one edge case.
5. **Baseline:** Describe a straightforward correct approach.
6. **Optimization:** Identify the bottleneck and the property that permits improvement.
7. **Invariant:** State what remains true while the algorithm runs.
8. **Complexity:** Give time and auxiliary-space complexity with assumptions.
9. **Tests:** Include normal, boundary, duplicate, and adversarial cases.
10. **Reflection:** Explain what would change under different constraints.

Do not claim an optimized solution is correct merely because it passes a few examples. Explain why its decisions are safe.

### Suggested implementation standard

- Use descriptive function and variable names.
- Keep one problem per function.
- Avoid hidden input mutation unless the problem permits it.
- Validate inputs when invalid values are part of the stated contract.
- Keep demonstration code separate from reusable functions.
- Use tests to check behavior, not only printed output.
- State whether returned memory is included in the space analysis.

---

## 2. Level 1 — Foundations

These problems build confidence with traversal, counting, and simple complexity analysis.

### Problem 1.1 — Count matching values

**Task:** Given a sequence of integers and a target value, return how many elements equal the target.

**Example**

- Input: `values = [4, 2, 4, 4, 7]`, `target = 4`
- Output: `3`

**Requirements**
- Do not modify the input.
- Return zero for an empty sequence.
- Support negative values and duplicates.

**Hint:** A single traversal is sufficient.

**Expected reasoning**
- Identify the number of elements examined.
- Explain why every element must be considered in the general case.
- State the auxiliary-space cost.

### Problem 1.2 — Second distinct maximum

**Task:** Return the second-largest distinct value in a sequence. If fewer than two distinct values exist, return `None`.

**Example**

- Input: `[8, 3, 8, 5, 5]`
- Output: `5`

**Requirements**
- Duplicate maximum values do not count as the second distinct value.
- Do not sort the input.
- Handle empty and single-distinct-value sequences.

**Hint:** Track the largest and second-largest distinct values as you traverse.

**Expected reasoning**
- Define how the two tracked values change.
- Consider a new maximum, a value between the two maxima, and a duplicate.
- State whether the algorithm uses constant auxiliary space.

### Problem 1.3 — First non-repeating value

**Task:** Return the first value in input order that occurs exactly once. Return `None` if no such value exists.

**Example**

- Input: `[5, 3, 5, 2, 3, 8]`
- Output: `2`

**Hint:** Separate frequency counting from selecting the first qualifying value.

**Expected reasoning**
- Explain why one pass may not provide enough information to know that a value never appears again.
- Describe the data structure used for counts.
- State expected hash-table assumptions if using a dictionary.

### Problem 1.4 — Reverse words

**Task:** Given a sentence represented as a string, return its words in reverse order, separated by one space.

**Example**

- Input: `"learn data structures"`
- Output: `"structures data learn"`

**Requirements**
- Treat runs of whitespace as separators.
- Return an empty string when there are no words.
- Do not reverse the characters inside each word.

**Hint:** Separate tokenization from word-order reversal.

**Expected reasoning**
- State how whitespace is handled.
- Discuss the memory required to build the output.

---

## 3. Level 2 — Arrays, strings, and hashing

### Problem 2.1 — Pair with target sum

**Task:** Return the indices of two different elements whose values add to a target. Return `None` if no pair exists.

**Example**

- Input: `values = [6, 1, 8, 3]`, `target = 9`
- Output: `[0, 2]` (or another valid pair if multiple answers exist)

**Requirements**
- Each input position can be used at most once.
- Do not assume the input is sorted.
- State whether the returned indices must be in ascending order.

**Hint:** Consider storing values from earlier positions and looking up the complement.

**Expected reasoning**
- Explain why the current item is checked before it is inserted.
- Compare a nested-loop baseline with a hash-map approach.
- State expected-time assumptions.

### Problem 2.2 — Longest substring without repeated characters

**Task:** Return the length of the longest substring containing no repeated characters.

**Example**

- Input: `"abcaabcd"`
- Output: `4` (one longest substring is `"abcd"`)

**Requirements**
- A substring must be contiguous.
- Empty input returns zero.
- Treat characters as Python string elements; do not assume ASCII only.

**Hint:** Use a moving window and remember the most recent position of each character.

**Expected reasoning**
- Explain why the left boundary never needs to move backward.
- Define what the current window guarantees.
- Analyze the number of times each boundary moves.

### Problem 2.3 — Group words by anagram signature

**Task:** Group words so that words containing the same characters with the same multiplicities appear in the same group.

**Example**

- Input: `["listen", "silent", "stone", "tones", "abc"]`
- Output: groups equivalent to `[["listen", "silent"], ["stone", "tones"], ["abc"]]`

**Requirements**
- Group order is not important.
- Preserve each original word in the output.
- State how your key handles case and Unicode characters.

**Hint:** Build a canonical key for each word, or use a character-frequency representation.

**Expected reasoning**
- Compare sorting each word to constructing a frequency key.
- Define the character domain before claiming a fixed-size counting array is valid.

### Problem 2.4 — Product of all other elements

**Task:** For each position, return the product of all values except the value at that position.

**Example**

- Input: `[1, 2, 3, 4]`
- Output: `[24, 12, 8, 6]`

**Requirements**
- Do not use division.
- Aim for linear time.
- Explain how zeros are handled.

**Hint:** Combine a prefix product with a suffix product.

**Expected reasoning**
- State what each pass has accumulated.
- Consider one zero, multiple zeros, negative values, and a one-element input.
- Discuss output space separately from auxiliary space.

### Problem 2.5 — Minimum-length subarray with target sum

**Task:** Given a list of positive integers and a positive target, return the minimum length of a contiguous subarray whose sum is at least the target. Return zero if no such subarray exists.

**Example**

- Input: `target = 11`, `values = [1, 2, 3, 4, 5]`
- Output: `3` (the subarray `[3, 4, 5]`)

**Requirements**
- All values must be strictly positive.
- The target must be positive.
- Empty input returns zero.

**Hint:** Use a variable-size sliding window. The positivity assumption is essential to the usual shrinking rule.

**Expected reasoning**
- Explain why removing the leftmost value reduces the current sum.
- Explain why the window can be shrunk while it remains valid.
- Describe what breaks if negative values are allowed.

---

## 4. Level 3 — Searching, stacks, queues, and intervals

### Problem 3.1 — First and last position

**Task:** Given a sorted sequence and a target, return the first and last indices where the target appears. Return `[-1, -1]` if it is absent.

**Example**

- Input: `values = [1, 2, 2, 2, 4, 7]`, `target = 2`
- Output: `[1, 3]`

**Requirements**
- Use logarithmic-time boundary searches.
- The input is sorted in non-decreasing order.
- Handle empty input.

**Hint:** Find the first position that is not less than the target, then determine the last matching position with another boundary search.

**Expected reasoning**
- Define the invariant for each binary search.
- Explain why a normal exact-match binary search does not guarantee the first occurrence.

### Problem 3.2 — Balanced delimiters

**Task:** Determine whether a string containing only `()`, `[]`, and `{}` is correctly balanced.

**Example**

- Input: `"{[()]}"`
- Output: `True`

- Input: `"([)]"`
- Output: `False`

**Requirements**
- Every opening delimiter must be closed by the matching type.
- Closing order must be properly nested.
- Empty input is balanced.

**Hint:** The most recently opened delimiter must be the next one closed.

**Expected reasoning**
- Explain why a stack models the required order.
- Test an early closing delimiter, mismatched types, and leftover openings.

### Problem 3.3 — Next greater value

**Task:** For each element, return the first greater value to its right, or `None` if no such value exists.

**Example**

- Input: `[2, 1, 5, 3, 4]`
- Output: `[5, 5, None, 4, None]`

**Hint:** Keep unresolved indices in a monotonic stack.

**Expected reasoning**
- State the ordering invariant of the stack.
- Explain why each index is pushed once and removed at most once.

### Problem 3.4 — Merge overlapping intervals

**Task:** Given a collection of closed intervals, merge all intervals that overlap.

**Example**

- Input: `[[1, 4], [2, 5], [8, 9]]`
- Output: `[[1, 5], [8, 9]]`

**Requirements**
- Intervals are closed: sharing an endpoint counts as overlap.
- Input may be unsorted.
- Empty input returns an empty list.

**Hint:** Sort by start and compare each interval with the last merged interval.

**Expected reasoning**
- Explain the overlap condition for closed intervals.
- State the cost of sorting and the output memory.

### Problem 3.5 — Queue using two stacks

**Task:** Implement a FIFO queue using two LIFO stacks. Support enqueue, dequeue, and checking whether the queue is empty.

**Requirements**
- Dequeue on an empty queue must have a clearly documented behavior.
- Each element should be transferred between stacks only when required.
- Explain the amortized cost of a sequence of operations.

**Hint:** One stack receives new elements; the other supplies the oldest element.

**Expected reasoning**
- Explain why reversing order twice restores FIFO behavior.
- Distinguish worst-case cost of one transfer operation from amortized cost across many operations.

---

## 5. Level 4 — Linked lists, trees, and heaps

### Problem 4.1 — Remove the N-th node from the end

**Task:** Given the head of a singly linked list and a positive integer (n), remove the N-th node from the end and return the new head.

**Requirements**
- Define behavior when (n) exceeds the list length.
- Handle removal of the head.
- Prefer a one-pass pointer approach.

**Hint:** Keep two pointers a fixed distance apart. A dummy node can simplify head removal.

**Expected reasoning**
- Explain the pointer-gap invariant.
- Test an empty list, one-node list, head removal, tail removal, and invalid (n).

### Problem 4.2 — Validate a binary search tree

**Task:** Determine whether a binary tree satisfies the strict binary-search-tree ordering rule: every key in a node's left subtree is smaller, and every key in its right subtree is larger.

**Requirements**
- Duplicate keys are invalid under this definition.
- Checking only each node against its immediate children is insufficient.
- State assumptions about comparable values.

**Hint:** Carry an allowed lower and upper bound during traversal.

**Expected reasoning**
- Explain why each node must satisfy constraints inherited from all its ancestors.
- Discuss recursive depth and an iterative alternative.

### Problem 4.3 — Level-order traversal

**Task:** Return the values of a binary tree level by level, from left to right.

**Example**

For a tree with root `1`, children `2` and `3`, and grandchildren `4`, `5`, return `[[1], [2, 3], [4, 5]]`.

**Hint:** Use a queue and process the number of nodes currently in the queue as one level.

**Expected reasoning**
- Explain why FIFO order processes shallower nodes first.
- State the time and space complexity in terms of the number of nodes.

### Problem 4.4 — K-th largest element

**Task:** Return the K-th largest element in a sequence, counting duplicates. For example, the second-largest element in `[7, 7, 3]` is `7`.

**Requirements**
- Define valid values of (k).
- Compare sorting with a bounded heap.
- Do not remove duplicates unless the problem definition changes.

**Hint:** A min-heap of size (k) can retain the largest (k) elements seen.

**Expected reasoning**
- State the heap invariant.
- Compare (O(n \log n)) sorting with a bounded-heap approach.
- Explain whether the result needs to be fully sorted.

### Problem 4.5 — Lowest common ancestor in a BST

**Task:** Given a binary search tree and two keys known to exist in it, return their lowest common ancestor.

**Requirements**
- Use the BST ordering property.
- State whether keys are unique.
- You may assume both keys exist, but explain how the contract would change if that were not guaranteed.

**Hint:** If both keys are smaller than the current node, move left. If both are larger, move right. Otherwise, the current node separates the two search paths.

**Expected reasoning**
- Explain why the first split point is the lowest common ancestor.
- Analyze complexity in terms of tree height, not automatically (O(\log n)) unless the tree is balanced.

---

## 6. Level 5 — Graphs and dependency reasoning

### Problem 5.1 — Count connected components

**Task:** Given an undirected graph, return the number of connected components.

**Requirements**
- The graph may contain isolated vertices.
- It may be disconnected.
- State the graph representation.

**Hint:** Start a traversal from every vertex that has not yet been visited. Each new traversal identifies one component.

**Expected reasoning**
- Explain why one traversal from a single start vertex is not enough for a disconnected graph.
- State (V) and (E) based complexity for an adjacency-list representation.

### Problem 5.2 — Dependency order

**Task:** Given tasks and directed prerequisite edges, return a valid execution order, or report that no valid order exists because the graph contains a cycle.

**Requirements**
- A directed edge `A -> B` means A must be completed before B.
- Include tasks with no dependencies.
- If several valid orders exist, any valid order is acceptable.

**Hint:** Use in-degree counts and a queue, or DFS with cycle-state tracking.

**Expected reasoning**
- Explain the condition that makes a topological order possible.
- Verify every edge points forward in the returned order.

### Problem 5.3 — Shortest unweighted path

**Task:** Given an unweighted graph, a start vertex, and a destination, return a shortest path as a sequence of vertices. Return an empty result if no path exists.

**Requirements**
- State whether the graph is directed or undirected.
- Handle start equal to destination.
- Include disconnected vertices in your tests.

**Hint:** BFS finds minimum edge-count distances. Store a parent when a vertex is first discovered, then reconstruct the path.

**Expected reasoning**
- Explain why first discovery in BFS gives a shortest unweighted path.
- Distinguish path reconstruction storage from the queue.

### Problem 5.4 — Detect a directed cycle

**Task:** Determine whether a directed graph contains a cycle.

**Requirements**
- Include disconnected components.
- Self-loops count as cycles.
- State the graph representation.

**Hint:** Consider DFS states such as unvisited, currently exploring, and fully processed; alternatively, use Kahn's topological-sort process.

**Expected reasoning**
- Explain why an edge to a currently exploring node indicates a cycle in DFS.
- Explain how an incomplete topological ordering indicates a cycle.

### Problem 5.5 — Network connectivity under added edges

**Task:** Start with (n) isolated vertices. Process a sequence of undirected edge additions and answer whether two vertices are connected after each addition.

**Requirements**
- Edges are only added; there are no edge removals.
- Repeated edge additions should not break the result.
- State how invalid vertex identifiers are handled.

**Hint:** Disjoint Set Union supports merging components and checking whether two vertices share a representative.

**Expected reasoning**
- Explain why DSU handles edge additions but does not directly support splitting a component after edge removal.
- State the amortized complexity assumptions for path compression and union by size.

---

## 7. Level 6 — Greedy, backtracking, and dynamic programming

### Problem 6.1 — Maximum compatible activities

**Task:** Given activities with start and finish times, select the largest possible number of mutually non-overlapping activities.

**Requirements**
- State whether an activity ending at time (t) can be followed by one starting at (t).
- Return the selected activities, not only the count.

**Hint:** Investigate sorting by finish time and selecting the next compatible activity.

**Expected reasoning**
- Give an exchange argument for the greedy choice.
- Explain why sorting by start time alone does not establish the same rule.

### Problem 6.2 — Generate balanced parentheses

**Task:** Given (n) pairs of parentheses, generate every valid string containing exactly (n) opening and (n) closing parentheses.

**Example**

- Input: `n = 2`
- Output: `["(())", "()()"]` (order does not matter)

**Requirements**
- For (n = 0), define whether the result is a list containing the empty string.
- Never generate invalid prefixes and filter them only at the end.
- Explain output-size implications.

**Hint:** Add an opening parenthesis while fewer than (n) have been used. Add a closing parenthesis only when it cannot make the prefix invalid.

**Expected reasoning**
- Define the valid-prefix invariant.
- Explain why pruning invalid prefixes is safe.

### Problem 6.3 — Minimum coin count

**Task:** Given positive coin denominations with unlimited supply and a non-negative amount, return the minimum number of coins needed. Return `-1` if the amount cannot be formed.

**Requirements**
- Do not assume the greedy largest-coin strategy is always optimal.
- Define the result for amount zero.
- Reject non-positive coin denominations.

**Hint:** Let a state represent the minimum number of coins needed to form a particular amount.

**Expected reasoning**
- Define the recurrence and base case.
- State complexity in terms of amount and number of denominations.
- Explain why the greedy strategy fails for some denomination systems.

### Problem 6.4 — Longest common subsequence

**Task:** Given two strings, return the length of their longest common subsequence. A subsequence preserves relative order but does not need to be contiguous.

**Example**

- Input: `"abcde"`, `"ace"`
- Output: `3`

**Hint:** Use a two-dimensional DP state for prefixes of the two strings.

**Expected reasoning**
- Explain the matching-character and non-matching-character transitions.
- State time and space complexity in terms of the two string lengths.
- Consider whether the space can be reduced when only the length is required.

### Problem 6.5 — Can the target be reached?

**Task:** Given a list of non-negative integers and a target, determine whether some subset sums exactly to the target. Each input element can be used at most once.

**Requirements**
- Distinguish this from the unlimited coin-change problem.
- Define behavior for an empty list and target zero.
- Explain how the update order affects a one-dimensional DP implementation.

**Hint:** Think of a state that records which sums are reachable after processing a prefix of the input. If compressing to one dimension, process candidate sums in descending order so the same item is not reused during its own iteration.

**Expected reasoning**
- State the DP invariant after processing each item.
- Discuss time and space in terms of the number of items and target.
- Explain why negative values would require a different state representation.

---

## 8. Level 7 — Advanced data structures and mixed patterns

### Problem 7.1 — Prefix dictionary

**Task:** Implement a data structure that supports inserting words, exact-word search, and checking whether any stored word starts with a given prefix.

**Requirements**
- Distinguish a complete word from a path that is only a prefix.
- Support repeated insertion.
- Define behavior for empty words and empty prefixes.

**Hint:** Use a trie with a terminal marker.

**Expected reasoning**
- State complexity in terms of the length of the queried word or prefix.
- Discuss memory growth with the number of distinct prefix nodes.

### Problem 7.2 — Dynamic range sum

**Task:** Maintain an integer sequence while supporting point additions and range-sum queries.

**Requirements**
- Use half-open query ranges `[left, right)`.
- Support an empty range, whose sum is zero.
- Define valid index bounds.

**Hint:** A Fenwick tree is a compact option for point additions and prefix sums. A segment tree is another option.

**Expected reasoning**
- Explain the internal indexing convention.
- Derive a range sum from two prefix sums.
- Compare with direct list summation when updates or queries are rare.

### Problem 7.3 — Static range minimum

**Task:** Given an immutable array, answer many non-empty range-minimum queries.

**Requirements**
- Use half-open ranges.
- The array does not change after preprocessing.
- Reject empty or out-of-bounds query ranges.

**Hint:** A sparse table supports constant-time minimum queries after preprocessing.

**Expected reasoning**
- Explain why the two selected power-of-two blocks may overlap safely for minimum.
- State preprocessing, query, and storage complexity.
- Explain why the same two-block formula does not directly work for sum.

### Problem 7.4 — LRU cache

**Task:** Implement a fixed-capacity least-recently-used cache with `get` and `put`.

**Requirements**
- A successful get makes the key most recently used.
- Updating an existing key makes it most recently used.
- Inserting beyond capacity evicts the least recently used key.
- Define behavior for capacity zero and missing keys.

**Hint:** Combine a mapping with an ordering mechanism. Python's `collections.OrderedDict` can support an educational implementation.

**Expected reasoning**
- Trace the recency order after every operation.
- Explain the cache miss representation.
- Discuss why concurrency, expiration, and memory-based limits are additional production concerns.

### Problem 7.5 — Merge K sorted sequences

**Task:** Given (k) sorted sequences, return one sorted sequence containing all their values.

**Requirements**
- Empty input and empty individual sequences are valid.
- Preserve duplicates.
- Explain how the approach behaves when one sequence is much longer than the others.

**Hint:** Keep the next unprocessed value from each non-empty sequence in a min-heap.

**Expected reasoning**
- State the heap invariant.
- Express complexity using (N), the total number of values, and (k), the number of input sequences.
- Compare with concatenating and sorting all values.

---

## 9. Mixed interview drills

For each drill, choose the pattern yourself before looking at the suggested topic.

| Drill | Task summary | Topics to consider |
|---|---|---|
| A | Find the longest run of consecutive integers, regardless of input order | Hash set |
| B | Return the maximum value in every fixed-size window | Monotonic deque |
| C | Find the minimum number of unweighted edges from a source to every reachable vertex | BFS |
| D | Determine whether a dependency schedule is possible | Directed cycle detection / topological sort |
| E | Return the K most frequent values | Frequency map + heap or bucket strategy |
| F | Find the smallest capacity that can process all jobs within a time limit | Binary search on a monotonic feasibility condition |
| G | Find the largest square of true values in a Boolean matrix | Dynamic programming |
| H | Find a minimum spanning tree of a connected weighted undirected graph | Kruskal or Prim |
| I | Count the number of ways to reach a target under defined step choices | Dynamic programming |
| J | Find all combinations that satisfy a constraint | Backtracking and pruning |

For every drill, document why the selected approach applies and what assumptions it needs.

---

## 10. Test design checklist

A solution should be tested against more than the example in the prompt.

### General cases

- [ ] Empty input, when permitted.
- [ ] One element.
- [ ] Two elements.
- [ ] All values equal.
- [ ] All values distinct.
- [ ] Negative, zero, and positive values where valid.
- [ ] Duplicate values.
- [ ] No valid answer.
- [ ] Multiple valid answers.
- [ ] Minimum and maximum allowed input sizes.

### Data-structure cases

- [ ] Empty stack, queue, or heap.
- [ ] Repeated insertion of the same key.
- [ ] Missing key lookup.
- [ ] First and last valid index.
- [ ] Single-element and full-range queries.
- [ ] Update followed by a query.
- [ ] Repeated union of already-connected DSU elements.
- [ ] Trie prefix that exists but is not a complete word.

### Graph cases

- [ ] Isolated vertex.
- [ ] Disconnected components.
- [ ] Self-loop.
- [ ] Parallel edges, if the graph contract permits them.
- [ ] Directed cycle.
- [ ] Directed acyclic graph.
- [ ] Start vertex equals destination.
- [ ] Unreachable destination.
- [ ] Negative edge, if the selected algorithm has weight restrictions.

### Property and differential testing

Where possible:
1. Generate small random inputs.
2. Run the optimized implementation.
3. Run a simple reference implementation.
4. Compare their outputs.
5. Preserve any failing input as a regression test.

A reference implementation may be slower; its purpose is to provide a simple correctness comparison on small inputs.

---

## 11. Suggested weekly practice schedule

This is a suggested learning schedule, not a requirement. Adjust the pace to your available time.

| Session | Focus | Suggested activity |
|---|---|---|
| 1 | Foundations | Solve Problems 1.1–1.4 |
| 2 | Arrays and hashing | Solve Problems 2.1–2.5 |
| 3 | Searching and stacks | Solve Problems 3.1–3.5 |
| 4 | Linked lists and trees | Solve Problems 4.1–4.5 |
| 5 | Graphs | Solve Problems 5.1–5.5 |
| 6 | Greedy, backtracking, DP | Solve Problems 6.1–6.5 |
| 7 | Advanced structures | Solve Problems 7.1–7.5 |
| 8 | Mixed practice | Attempt drills A–J under a time limit |

After each session, revisit one problem without looking at your earlier implementation. The goal is independent reasoning, not merely recognizing familiar code.

---

## 12. Progress tracker

Use this checklist to track your own progress. A problem is complete only when you can explain the solution and its trade-offs.

### Level 1 — Foundations

- [ ] 1.1 Count matching values
- [ ] 1.2 Second distinct maximum
- [ ] 1.3 First non-repeating value
- [ ] 1.4 Reverse words

### Level 2 — Arrays, strings, and hashing

- [ ] 2.1 Pair with target sum
- [ ] 2.2 Longest substring without repeated characters
- [ ] 2.3 Group words by anagram signature
- [ ] 2.4 Product of all other elements
- [ ] 2.5 Minimum-length subarray with target sum

### Level 3 — Searching, stacks, queues, and intervals

- [ ] 3.1 First and last position
- [ ] 3.2 Balanced delimiters
- [ ] 3.3 Next greater value
- [ ] 3.4 Merge overlapping intervals
- [ ] 3.5 Queue using two stacks

### Level 4 — Linked lists, trees, and heaps

- [ ] 4.1 Remove the N-th node from the end
- [ ] 4.2 Validate a binary search tree
- [ ] 4.3 Level-order traversal
- [ ] 4.4 K-th largest element
- [ ] 4.5 Lowest common ancestor in a BST

### Level 5 — Graphs

- [ ] 5.1 Count connected components
- [ ] 5.2 Dependency order
- [ ] 5.3 Shortest unweighted path
- [ ] 5.4 Detect a directed cycle
- [ ] 5.5 Network connectivity under added edges

### Level 6 — Greedy, backtracking, and dynamic programming

- [ ] 6.1 Maximum compatible activities
- [ ] 6.2 Generate balanced parentheses
- [ ] 6.3 Minimum coin count
- [ ] 6.4 Longest common subsequence
- [ ] 6.5 Can the target be reached?

### Level 7 — Advanced data structures

- [ ] 7.1 Prefix dictionary
- [ ] 7.2 Dynamic range sum
- [ ] 7.3 Static range minimum
- [ ] 7.4 LRU cache
- [ ] 7.5 Merge K sorted sequences

### Mixed drills

- [ ] A
- [ ] B
- [ ] C
- [ ] D
- [ ] E
- [ ] F
- [ ] G
- [ ] H
- [ ] I
- [ ] J

---

## 13. Source and quality notes

This practice set is an original repository learning resource. The problem statements are designed to exercise concepts explained in the preceding chapters.

Official Python documentation is relevant when implementing the examples with Python:

- [Python — Built-in Types](https://docs.python.org/3/library/stdtypes.html)
- [Python — Data Structures Tutorial](https://docs.python.org/3/tutorial/datastructures.html)
- [Python — `collections`](https://docs.python.org/3/library/collections.html)
- [Python — `heapq`](https://docs.python.org/3/library/heapq.html)
- [Python — `bisect`](https://docs.python.org/3/library/bisect.html)

These documents describe Python language and standard-library behavior. They do not establish a unique solution for each exercise.

**Repository design decision:** Hints are intentionally brief. Full solutions are not included here so the problem set can be used for independent practice.

**Demonstration:** Examples and requirements define the intended learning contracts. If an exercise is adapted for a different platform or interview, clarify any changed assumptions before solving it.

---

**Previous:** [17 — Interview Patterns](../17%20Interview%20Patterns/README.md)  
**Back to:** [Complete DSA Roadmap](../README.md)
