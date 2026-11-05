# 17 — Interview Patterns

Data-structure and algorithm interviews are not only about knowing individual algorithms. They test whether you can recognize a problem shape, select a useful approach, explain its correctness, and implement it carefully.

A **problem-solving pattern** is a reusable way to organize a solution. It is not a formula to apply blindly. The same pattern can solve many problems, but you must verify that the problem's constraints and properties actually support it.

> **Learning goal:** Build a repeatable interview workflow, recognize common patterns from problem constraints, explain trade-offs, and validate solutions with edge cases.

---

## 1. A repeatable interview workflow

Use this sequence before writing code:

1. **Clarify the task.** Confirm input, output, constraints, duplicates, ordering, and whether input may be modified.
2. **Work through an example.** Trace a normal case and a boundary case.
3. **Start with a simple solution.** A correct brute-force approach gives you a baseline.
4. **Identify the bottleneck.** Is repeated searching, sorting, nested iteration, or recomputation causing the cost?
5. **Choose a pattern.** Match the bottleneck and problem properties to a suitable technique.
6. **State the invariant.** Explain what remains true as the algorithm runs.
7. **Implement clearly.** Use meaningful names and handle boundaries.
8. **Test deliberately.** Include empty, smallest, largest, duplicate, and adversarial cases where applicable.
9. **Analyze complexity.** State time and auxiliary-space costs, including assumptions.
10. **Discuss alternatives.** Explain when a different structure or pattern may be preferable.

Do not begin by guessing the pattern from a keyword. Derive it from the operations and constraints.

---

## 2. Pattern recognition map

| Problem signal | Pattern to investigate |
|---|---|
| Sorted sequence, pair or boundary search | Two pointers or binary search |
| Contiguous subarray or substring | Sliding window or prefix sums |
| Repeated lookup, counting, grouping | Hash map or set |
| Linked-list cycle or middle node | Fast and slow pointers |
| Overlapping time or numeric ranges | Sort and merge intervals |
| Next greater/smaller element | Monotonic stack |
| First/last valid position in sorted data | Binary search on a boundary |
| Top K, repeated minimum/maximum | Heap / priority queue |
| Shortest path in an unweighted graph | Breadth-first search |
| Explore components or reachability | Depth-first search or BFS |
| Dependencies and ordering constraints | Topological sorting |
| Connected components under edge additions | Disjoint Set Union |
| Generate all valid choices | Backtracking |
| Repeated subproblems with shared states | Dynamic programming |
| Locally optimal choice with a proof | Greedy algorithm |
| Prefix-based string search | Trie |

A signal suggests a candidate; it does not prove the pattern is correct.

---

## 3. Two pointers

Two pointers maintain two positions while scanning a sequence. The pointers may move toward each other, move in the same direction, or represent the beginning and end of a current range.

### When to consider it

- The input is sorted and you need a pair satisfying a condition.
- You need to reverse or compare elements from opposite ends.
- You need to remove or compact elements in place.
- You need to merge or compare two ordered sequences.

### Example: pair sum in a sorted sequence

Given a sorted list, determine whether two different positions contain values whose sum equals a target.

```python
def has_pair_with_sum(values, target):
    left = 0
    right = len(values) - 1

    while left < right:
        current = values[left] + values[right]

        if current == target:
            return True
        if current < target:
            left += 1
        else:
            right -= 1

    return False
```

**Why the pointer movement is valid:** Because the sequence is sorted, if the current sum is too small, keeping the left value and moving the right pointer left cannot increase the sum. The left pointer must move right. If the sum is too large, moving the right pointer left is the direction that can reduce it.

- Time: (O(n))
- Auxiliary space: (O(1))

The sorted-input precondition is essential. This reasoning does not automatically work on an unsorted list.

### Practice variations

- Reverse a sequence in place.
- Check whether a sorted sequence contains a pair with a target difference.
- Remove duplicates from a sorted sequence.
- Merge two sorted sequences.
- Compare characters from both ends while ignoring selected characters.

---

## 4. Sliding window

A sliding window maintains a contiguous region of a sequence. Instead of rebuilding every candidate region from scratch, the algorithm updates the state as the left or right boundary moves.

### Fixed-size window

Example: find the maximum sum of any window of exactly (k) values.

```python
def max_window_sum(values, k):
    if not 1 <= k <= len(values):
        raise ValueError("k must be between 1 and the input length")

    window_sum = sum(values[:k])
    best = window_sum

    for right in range(k, len(values)):
        window_sum += values[right]
        window_sum -= values[right - k]
        best = max(best, window_sum)

    return best
```

The first window is calculated once. Each following window adds one entering value and removes one leaving value.

- Time: (O(n))
- Auxiliary space: (O(1))

### Variable-size window

A variable window expands and contracts according to a condition.

Example: find the minimum length of a contiguous subarray whose sum is at least a positive target.

```python
def min_length_at_least(target, values):
    left = 0
    current_sum = 0
    best = len(values) + 1

    for right, value in enumerate(values):
        current_sum += value

        while current_sum >= target:
            best = min(best, right - left + 1)
            current_sum -= values[left]
            left += 1

    return 0 if best == len(values) + 1 else best
```

**Important precondition:** This shrinking-window reasoning relies on positive values. With arbitrary negative values, removing the left value does not necessarily reduce the sum, so this algorithm is not generally valid.

### Common window state

A window may track:
- a running sum;
- character frequencies;
- the number of distinct values;
- counts of values meeting a condition;
- a maximum or minimum maintained by a monotonic deque.

Always explain why moving a boundary preserves the intended search.

---

## 5. Prefix sums and difference arrays

Prefix sums are useful when many queries ask for the sum of a contiguous range in an array that is not being updated.

Define a prefix array where `prefix[i]` is the sum of the first `i` values. Then the sum of the half-open range `[left, right)` is:

[
prefix[right] - prefix[left]
]

```python
def build_prefix(values):
    prefix = [0]

    for value in values:
        prefix.append(prefix[-1] + value)

    return prefix


def range_sum(prefix, left, right):
    if not 0 <= left <= right < len(prefix):
        raise IndexError("invalid range")
    return prefix[right] - prefix[left]
```

- Build: (O(n))
- Each range-sum query: (O(1))
- Extra storage: (O(n))

For repeated range additions, a **difference array** can represent changes at boundaries and reconstruct values with a prefix pass. Be precise about inclusive versus half-open interval conventions.

Use a Fenwick tree or segment tree instead when the problem requires interleaved updates and queries; see [16 — Advanced Data Structures](../16%20Advanced%20Data%20Structures/README.md).

---

## 6. Hash map and set patterns

Hash maps and sets are common when a problem needs fast membership checks, counting, grouping, or a relationship between values.

### Complement lookup

For each value, check whether the value needed to reach the target has already been seen.

```python
def two_sum_indices(values, target):
    seen = {}

    for index, value in enumerate(values):
        needed = target - value

        if needed in seen:
            return [seen[needed], index]

        seen[value] = index

    return None
```

The map stores values from earlier positions, so the same element is not used twice.

- Expected time: (O(n))
- Auxiliary space: (O(n))

The expected-time statement assumes average constant-time hash-table operations.

### Other common uses

**Frequency counting**

```python
def frequencies(values):
    counts = {}

    for value in values:
        counts[value] = counts.get(value, 0) + 1

    return counts
```

**Duplicate detection**

Use a set to record values already encountered. Stop when a value appears a second time.

**Grouping**

Map a normalized key to a list of records. Examples include grouping words by a signature or grouping events by service identifier.

### Interview caution

Clarify whether keys are hashable, whether equality has special semantics, whether input order must be preserved, and whether the result needs stable ordering.

---

## 7. Fast and slow pointers

Two pointers can move at different speeds. This is particularly useful for linked lists and cycle-related problems.

### Detect a cycle in a linked list

One pointer advances one node at a time; the other advances two nodes at a time. If a cycle exists and both pointers enter it, their positions eventually meet. If the fast pointer reaches the end, the list is acyclic.

```python
def has_cycle(head):
    slow = head
    fast = head

    while fast is not None and fast.next is not None:
        slow = slow.next
        fast = fast.next.next

        if slow is fast:
            return True

    return False
```

- Time: (O(n))
- Auxiliary space: (O(1))

Use object identity for node meeting in this linked-list model. Comparing node values could report a false meeting when different nodes contain equal values.

### Other applications

- Find the middle node of a linked list.
- Find the node where a cycle begins after detecting one.
- Find the (k)-th node from the end using a fixed pointer gap.

---

## 8. Merge intervals

Interval problems often involve overlaps, scheduling, availability, or ranges of values.

A common approach is:
1. Sort intervals by start.
2. Keep the most recently merged interval.
3. If the next interval overlaps it, extend the end.
4. Otherwise, append a new interval.

```python
def merge_intervals(intervals):
    if not intervals:
        return []

    ordered = sorted(intervals, key=lambda interval: interval[0])
    merged = [list(ordered[0])]

    for start, end in ordered[1:]:
        if start <= merged[-1][1]:
            merged[-1][1] = max(merged[-1][1], end)
        else:
            merged.append([start, end])

    return merged
```

This example treats intervals as closed intervals, so intervals sharing an endpoint overlap. If the problem uses half-open intervals such as `[start, end)`, the overlap condition may need to change.

- Time: (O(n \log n)), dominated by sorting.
- Auxiliary space: (O(n)) for the output, plus sorting-related implementation space.

### Variations

- Insert one interval into a sorted, non-overlapping list.
- Find the minimum number of rooms needed for meetings.
- Determine whether any intervals overlap.
- Compute free time from busy intervals.

For room-allocation and event-sweep problems, a min-heap or sorted start/end events may be more appropriate than directly merging intervals.

---

## 9. Monotonic stack and deque

A **monotonic stack** maintains elements in increasing or decreasing order. It helps find the next greater or smaller element without comparing every pair.

### Example: next greater element to the right

For each position, find the first later value that is greater. If none exists, return `None`.

```python
def next_greater_to_right(values):
    result = [None] * len(values)
    stack = []

    for index, value in enumerate(values):
        while stack and values[stack[-1]] < value:
            previous_index = stack.pop()
            result[previous_index] = value

        stack.append(index)

    return result
```

Each index is pushed once and popped at most once.

- Time: (O(n))
- Auxiliary space: (O(n))

The stack stores indices rather than values so that the result can be written to the correct position.

### Monotonic deque

A deque can maintain candidate indices for a moving-window maximum or minimum. When an index leaves the window, remove it from the front; when a new value arrives, remove dominated candidates from the back.

Use a monotonic deque when the query is about the maximum or minimum of a moving window. Do not confuse it with a normal FIFO queue.

---

## 10. Binary search and boundary search

Binary search applies when the search space has an order or a monotonic condition that lets you discard a portion safely.

### Exact search

```python
def binary_search(values, target):
    left = 0
    right = len(values) - 1

    while left <= right:
        middle = left + (right - left) // 2

        if values[middle] == target:
            return middle
        if values[middle] < target:
            left = middle + 1
        else:
            right = middle - 1

    return -1
```

- Time: (O(\log n))
- Auxiliary space: (O(1))

The input must be sorted according to the same ordering used by the comparisons.

### First true boundary

Many interview problems ask for the first position where a monotonic predicate becomes true. The predicate has a shape like:

```text
False False False True True True
```

Binary search can find the transition without checking every position. Define the search bounds carefully and state what each bound represents.

### Binary search on an answer

Sometimes the answer is a number, not an index. If you can test whether a candidate answer is feasible and feasibility is monotonic, binary search over the answer range may apply.

Examples include minimizing a maximum capacity or finding the smallest rate that completes a task within a limit. The feasibility function must be correct and monotonic; otherwise binary search is not justified.

---

## 11. Heap and top-K patterns

A heap is useful when you repeatedly need access to the smallest or largest priority item.

Typical problems:
- find the (k) largest or smallest values;
- merge sorted sequences;
- schedule the next highest-priority task;
- repeatedly process the next event by time.

Python's `heapq` implements a min-heap.

### Keep the largest K values

A min-heap of size (k) can retain the largest (k) values seen so far. The smallest retained value stays at the root.

```python
import heapq


def k_largest(values, k):
    if not 0 <= k <= len(values):
        raise ValueError("k must be between 0 and the input length")

    heap = []

    for value in values:
        if len(heap) < k:
            heapq.heappush(heap, value)
        elif k and value > heap[0]:
            heapq.heapreplace(heap, value)

    return sorted(heap, reverse=True)
```

For (k > 0):
- Time: (O(n \log k + k \log k)), including sorting the retained values.
- Auxiliary space: (O(k)).

For (k = 0), the function returns an empty list.

### Interview questions to clarify

- Do you need the top K values or only the K-th value?
- Are duplicates retained?
- Is the result required in sorted order?
- Is the input a stream or a complete collection?
- Is (k) much smaller than (n)?

---

## 12. Graph traversal patterns

Graph problems often become simpler after identifying the graph model:
- What are the vertices?
- What do edges represent?
- Are edges directed?
- Are edges weighted?
- Can there be disconnected components?
- Can cycles exist?

### BFS: shortest path in an unweighted graph

Breadth-first search explores vertices in layers. In an unweighted graph, the first discovered distance to a vertex is its minimum number of edges from the start.

```python
from collections import deque


def shortest_unweighted_distances(graph, start):
    distances = {start: 0}
    queue = deque([start])

    while queue:
        node = queue.popleft()

        for neighbor in graph.get(node, []):
            if neighbor not in distances:
                distances[neighbor] = distances[node] + 1
                queue.append(neighbor)

    return distances
```

The graph is represented as an adjacency mapping. The function returns distances only for vertices reachable from `start`.

For an adjacency-list graph, traversal takes (O(V + E)) time over the reachable portion, assuming constant-time dictionary/set membership on average.

### DFS: reachability and components

Depth-first search explores as far as possible along a path before backtracking. It is useful for:
- reachability;
- connected components;
- cycle detection;
- traversal and structural analysis.

### Topological sorting

For a directed acyclic graph (DAG), topological sorting produces an order in which every directed edge goes from an earlier vertex to a later vertex.

If the graph contains a directed cycle, no topological ordering exists. Kahn's algorithm uses in-degrees and a queue; if it cannot output every vertex, the graph is not a DAG.

### Weighted shortest paths

Do not use ordinary BFS to solve a general weighted shortest-path problem. Select an algorithm based on edge-weight assumptions:
- nonnegative weights: Dijkstra is a common choice;
- negative edges: Bellman–Ford may be appropriate;
- all-pairs shortest paths: Floyd–Warshall may fit smaller dense graphs.

See [11 — Graphs](../11%20Graphs/README.md) and [15 — Advanced Graph Algorithms](../15%20Advanced%20Graph%20Algorithms/README.md).

---

## 13. Backtracking

Backtracking explores a decision tree of partial solutions. At each step, it:
1. chooses an option;
2. explores the resulting partial state;
3. undoes the choice;
4. tries another option.

It is appropriate when the task asks for all valid combinations, permutations, arrangements, or paths and the search space can be explored with pruning.

### Example: generate subsets

```python
def subsets(values):
    result = []
    current = []

    def backtrack(index):
        if index == len(values):
            result.append(current.copy())
            return

        # Choice 1: exclude this value.
        backtrack(index + 1)

        # Choice 2: include this value.
        current.append(values[index])
        backtrack(index + 1)
        current.pop()

    backtrack(0)
    return result
```

For (n) input values, there are (2^n) subsets. The output itself has exponential size, so exponential total work is expected when every subset must be returned.

### Pruning

Stop exploring a branch when it cannot produce a valid solution. A pruning rule must be logically safe: it must not discard a branch that could contain a valid answer.

Common mistakes include forgetting to undo state, using the wrong stopping condition, and accidentally sharing mutable result lists.

---

## 14. Dynamic programming

Dynamic programming (DP) is useful when a problem can be represented using states with overlapping subproblems and a recurrence that combines smaller states.

A practical DP design process:

1. Define what a state means in plain language.
2. Write the recurrence.
3. Identify base cases.
4. Choose memoization or tabulation.
5. Determine the order in which states are computed.
6. Check whether stored results can be reduced.
7. Reconstruct a solution if the task asks for choices, not only an optimal value.

### Example: minimum number of coins

Assume the coin denominations are positive integers and unlimited copies of each denomination are available.

```python
def min_coins(coins, amount):
    if amount < 0:
        raise ValueError("amount must be non-negative")
    if any(coin <= 0 for coin in coins):
        raise ValueError("coin values must be positive")

    unreachable = amount + 1
    dp = [unreachable] * (amount + 1)
    dp[0] = 0

    for current in range(1, amount + 1):
        for coin in coins:
            if coin <= current:
                dp[current] = min(
                    dp[current],
                    dp[current - coin] + 1
                )

    return -1 if dp[amount] == unreachable else dp[amount]
```

For (A) as the amount and (C) as the number of denominations:
- Time: (O(A \cdot C))
- Auxiliary space: (O(A))

The recurrence considers every denomination for each amount. This is not a greedy algorithm: choosing the largest available coin first is not guaranteed to produce a minimum count for arbitrary denominations.

---

## 15. Greedy patterns

A greedy algorithm makes a locally preferred choice at each step. It is correct only when those choices can be shown to lead to a globally optimal solution for the problem.

A plausible local rule is not a proof.

### Example: select maximum number of non-overlapping activities

For activities represented by start and finish times, sorting by finish time and repeatedly selecting the next compatible activity is a standard greedy strategy for maximizing the number of non-overlapping activities under the usual interval assumptions.

```python
def select_activities(activities):
    ordered = sorted(activities, key=lambda item: item[1])
    selected = []
    last_finish = None

    for start, finish in ordered:
        if last_finish is None or start >= last_finish:
            selected.append((start, finish))
            last_finish = finish

    return selected
```

This example treats an activity ending at time (t) as compatible with one starting at (t). If the problem defines endpoint overlap differently, adjust the compatibility test.

- Time: (O(n \log n)), dominated by sorting.
- Auxiliary space: depends on the output and sorting implementation.

For an interview, explain why choosing the earliest finishing compatible activity leaves as much remaining time as possible for later activities. Then relate that exchange argument to an optimal solution.

---

## 16. Choosing between patterns

Several patterns can solve similar-looking problems. Use the requirements to distinguish them.

| Requirement | Candidate | Why |
|---|---|---|
| Pair sum in sorted values | Two pointers | Ordering allows safe pointer movement |
| Pair sum in unsorted values | Hash map | Stores complements already seen |
| Many static range-sum queries | Prefix sums | Precomputation makes each query constant time |
| Range queries mixed with point updates | Fenwick / segment tree | Supports interleaved changes and queries |
| Fixed-length contiguous window | Sliding window | Add entering value, remove leaving value |
| Next greater value | Monotonic stack | Each index is pushed and popped at most once |
| First feasible capacity | Binary search on answer | Feasibility must be monotonic |
| Top K values | Heap | Maintains a bounded set of candidates |
| Minimum edge-count path | BFS | Explores by distance layers |
| Minimum weighted path with nonnegative weights | Dijkstra | Uses tentative distances and a min-priority queue |
| All valid combinations | Backtracking | Enumerates choices with optional pruning |
| Optimal value with repeated subproblems | DP | Reuses computed states |

If two patterns appear plausible, estimate their complexity and compare implementation complexity, memory use, and assumptions.

---

## 17. Complexity discussion in interviews

State the variables before giving the complexity.

Examples:
- (n): number of elements;
- (V): number of graph vertices;
- (E): number of graph edges;
- (k): requested top-K count;
- (L): length of a string;
- (A): numeric target amount;
- (C): number of coin denominations.

Distinguish:
- **Time complexity:** how the amount of work grows.
- **Auxiliary space:** extra working memory, excluding required output unless stated otherwise.
- **Output space:** memory required to return the answer.
- **Expected complexity:** depends on assumptions such as average hash-table behavior.
- **Amortized complexity:** average cost over a sequence of operations, not a guarantee that each individual operation has that cost.

Do not claim (O(1)) merely because a loop or operation looks short. Explain the data structure and assumptions.

---

## 18. Common interview mistakes

1. Coding before confirming the problem's assumptions.
2. Applying a pattern because of a keyword rather than proving its preconditions.
3. Forgetting whether interval endpoints are inclusive or exclusive.
4. Using a sliding window when negative values invalidate the movement rule.
5. Using BFS for arbitrary weighted shortest paths.
6. Using greedy choices without a correctness argument.
7. Forgetting to restore mutable state during backtracking.
8. Failing to distinguish a prefix from a complete word in trie problems.
9. Ignoring duplicates, empty input, or a one-element input.
10. Giving time complexity without defining the input variables.
11. Ignoring output memory when the task returns all combinations.
12. Overengineering before a simple correct baseline exists.

---

## 19. Practice plan

Work through the following sequence. For each problem, write down the pattern, its preconditions, the invariant, complexity, and at least three edge cases before considering the exercise complete.

### Foundation patterns

- Two pointers: pair sum in a sorted list.
- Hash map: find two indices whose values sum to a target.
- Sliding window: maximum sum of a fixed-size window.
- Prefix sums: answer repeated range-sum queries.
- Binary search: find the first occurrence of a target.

### Intermediate patterns

- Fast/slow pointers: detect a linked-list cycle.
- Merge intervals: combine overlapping intervals.
- Monotonic stack: next greater element.
- Heap: return the K largest values.
- BFS: shortest path in an unweighted graph.
- DFS: count connected components.

### Advanced patterns

- Topological sort: determine whether dependencies can be completed.
- Backtracking: generate valid combinations under constraints.
- Dynamic programming: minimum coin count or longest common subsequence.
- Greedy: select the maximum number of compatible activities.
- DSU: process connectivity queries as edges are added.
- Binary search on answer: find the smallest feasible capacity.

---

## 20. Mini project — Interview Pattern Lab

**Goal:** Build a small, testable collection of problem-solving exercises. Each exercise should demonstrate one pattern and explain why it is valid.

### Suggested structure

```text
interview-pattern-lab/
├── README.md
├── two_pointers.py
├── sliding_window.py
├── prefix_sums.py
├── hash_map_patterns.py
├── intervals.py
├── monotonic_stack.py
├── binary_search.py
├── heap_patterns.py
├── graph_traversal.py
├── backtracking.py
├── dynamic_programming.py
└── tests/
    ├── test_arrays.py
    ├── test_search.py
    └── test_graphs.py
```

This is a suggested project layout, not a requirement imposed by Python or any external tool.

### Engineering requirements

1. Give each exercise a clear input and output contract.
2. Document assumptions such as sorted input or positive values.
3. Include a readable baseline when it helps explain the optimization.
4. Add unit tests for normal and boundary cases.
5. Compare optimized results with a simple reference implementation where practical.
6. Document time and auxiliary-space complexity.
7. Keep each implementation focused; avoid building a large framework around small exercises.

**Repository design decision:** The lab is an educational collection. It is not intended to be a competitive-programming judge or an interview question bank copied from a proprietary source.

---

## 21. Interview answer template

Use this short structure when explaining a solution aloud:

1. **Restate:** “We need to …”
2. **Assumptions:** “I will assume …; if that differs, the approach changes because …”
3. **Baseline:** “A direct solution would … and costs …”
4. **Observation:** “The key property is …”
5. **Pattern:** “I will use … because …”
6. **Invariant:** “At each step, … remains true.”
7. **Correctness:** “This is correct because …”
8. **Complexity:** “For (n) …, time is … and auxiliary space is …”
9. **Tests:** “I would check …”
10. **Trade-off:** “If the workload instead required …, I would consider …”

A concise explanation with a valid invariant is stronger than naming several algorithms without connecting them to the problem.

---

## 22. Completion checklist

- [ ] I clarify input constraints before choosing a pattern.
- [ ] I can recognize when sorted input enables two pointers.
- [ ] I can distinguish fixed and variable sliding windows.
- [ ] I know when prefix sums are sufficient and when updates require a tree.
- [ ] I can use hash maps for counting, grouping, and complement lookup.
- [ ] I can explain fast/slow pointer reasoning.
- [ ] I can merge intervals with a clearly defined endpoint convention.
- [ ] I can explain why a monotonic stack is linear overall.
- [ ] I can distinguish exact binary search from boundary search.
- [ ] I can use a heap for top-K problems.
- [ ] I can choose BFS, DFS, or a weighted shortest-path algorithm appropriately.
- [ ] I can explain backtracking state and safe pruning.
- [ ] I can define DP states, recurrences, and base cases.
- [ ] I can justify a greedy choice with a correctness argument.
- [ ] I can state time and auxiliary-space complexity with assumptions.
- [ ] I test edge cases and explain trade-offs clearly.

---

## 23. Sources and quality notes

Official Python documentation for language and standard-library behavior used in examples:

- [Python — Built-in Types](https://docs.python.org/3/library/stdtypes.html) — sequence and mapping behavior.
- [Python — Data Structures Tutorial](https://docs.python.org/3/tutorial/datastructures.html) — lists, tuples, sets, and dictionaries.
- [Python — `collections.deque`](https://docs.python.org/3/library/collections.html#collections.deque) — double-ended queue used in BFS.
- [Python — `heapq`](https://docs.python.org/3/library/heapq.html) — min-heap operations.
- [Python — `bisect`](https://docs.python.org/3/library/bisect.html) — binary search helpers for sorted sequences.

These sources document Python APIs and behavior; they are not presented as proofs of every algorithmic claim. Complexity statements in this chapter describe the stated algorithms and assumptions.

**Verified fact:** The linked Python documentation describes the named built-in types and standard-library APIs.

**Repository design decision:** This chapter organizes recurring techniques into a practical interview workflow and provides illustrative Python implementations.

**Demonstration:** Code examples are teaching implementations. Test them against the problem's actual constraints before reusing them in another context.

---

**Previous:** [16 — Advanced Data Structures](../16%20Advanced%20Data%20Structures/README.md)  
**Back to:** [Complete DSA Roadmap](../README.md)
