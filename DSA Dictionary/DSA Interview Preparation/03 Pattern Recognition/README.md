# 03 — Pattern Recognition

Pattern recognition is the skill of turning an unfamiliar problem into a known family of algorithmic decisions. It is not keyword matching. A phrase such as "longest", "minimum", "subarray", or "dependencies" may suggest a technique, but the constraints and correctness argument determine whether that technique is valid.

> **Goal:** Identify candidate patterns from the input, constraints, data shape, and required output; verify their preconditions; explain the invariant or state transition; and choose an approach whose costs fit the problem.

This guide complements [17 — Interview Patterns](../../Complete%20DSA/17%20Interview%20Patterns/README.md). That chapter contains broader explanations and implementations; this file concentrates on the decision process used during interviews.

---

## 1. The pattern-selection workflow

### Step 1 — Clarify the contract

Write down:

- **Input:** Array, string, linked list, tree, graph, stream, intervals, or records?
- **Output:** One value, position, pair, range, path, count, or every valid solution?
- **Constraints:** Maximum sizes? Sorted, positive, negative, unique, bounded, mutable?
- **Semantics:** Do duplicates matter? Does order matter? Are interval endpoints inclusive? Are graph edges directed or weighted?
- **Operations:** Static data, repeated queries, updates, insertions, deletions, or one pass?

Never silently assume sorted input, positive values, unique elements, or non-empty collections.

### Step 2 — Build a simple baseline

A straightforward solution provides a correctness baseline and exposes the expensive operation:

- repeated membership searches;
- comparing every pair;
- recomputing the same range or subproblem;
- repeatedly selecting a minimum or maximum;
- revisiting graph nodes;
- exploring branches that can be rejected early.

State the baseline complexity before optimizing.

### Step 3 — Identify the structural property

Ask:

1. Is the data ordered?
2. Is the required region contiguous?
3. Are there repeated lookups or counts?
4. Are there repeated range queries?
5. Does the problem describe relationships or dependencies?
6. Must we repeatedly retrieve the smallest, largest, or highest-priority item?
7. Are we generating possibilities or finding one optimum?
8. Do smaller subproblems overlap?
9. Is there a monotonic yes/no condition over a search range?

### Step 4 — Propose candidates, then test assumptions

For each candidate, explain:
- why its data structure matches the input;
- what information it maintains;
- what remains true after each operation;
- why no valid answer is skipped;
- how duplicates and boundaries are handled;
- time and auxiliary-space complexity.

If you cannot explain why a pointer moves, why a branch is pruned, or why a greedy choice is safe, you have not justified the pattern.

### Step 5 — Compare alternatives

Choose the simplest correct approach that fits the constraints. Compare time, memory, implementation risk, and assumptions. A theoretically faster approach is not better if its preconditions do not hold.

---

## 2. A practical decision map

Use this as a shortlist, not an automatic answer key.

| Observed structure or requirement | Candidate pattern | What must be checked |
|---|---|---|
| Repeated membership, counting, grouping, complement lookup | Hash map / set | Keys must be hashable; expected lookup costs rely on hash-table behavior |
| Sorted sequence and pair/partition condition | Two pointers | Pointer movement must be justified by ordering |
| Contiguous subarray or substring | Sliding window / prefix sums | Window movement must be valid; negative values can invalidate common sum-window logic |
| Many static range-sum queries | Prefix sums | Data is not updated between queries |
| Linked-list cycle or middle node | Fast and slow pointers | Termination and node identity |
| Overlapping intervals | Sort and merge / sweep line / heap | Endpoint convention and required output |
| Next greater/smaller or nearest qualifying element | Monotonic stack | Stack order and equality behavior |
| Moving-window maximum/minimum | Monotonic deque | Expired indices and dominated candidates |
| Ordered lookup or monotonic feasibility | Binary search | Ordering or predicate is genuinely monotonic |
| Repeated minimum/maximum or top K | Heap / priority queue | Clarify K, duplicates, ordering, stream versus batch |
| Reachability or connected components | DFS / BFS | Graph representation, direction, disconnected vertices |
| Fewest edges in an unweighted graph | BFS | Edges have equal cost |
| Dependency ordering | Topological sort | Directed graph; a cycle prevents a complete ordering |
| Connectivity as edges are added | Disjoint Set Union (DSU) | Connectivity queries/unions, not arbitrary path queries |
| All combinations or valid arrangements | Backtracking | Output may be exponential; pruning must be safe |
| Repeated overlapping subproblems | Dynamic programming | Define states, recurrence, base cases, evaluation order |
| Locally optimal choices | Greedy | A correctness proof is needed |
| Prefix lookup over many strings | Trie | Prefix operations justify the extra structure |

A signal narrows the search. It does not establish correctness.

---

## 3. Core patterns and how to recognize them

### 3.1 Hash map and set

**Consider it when** you repeatedly ask whether a value appeared, need frequencies or grouping, need a complement lookup, or can replace nested searching by recording information already seen.

**Invariant:** The map or set contains exactly the relevant information from the portion already processed. In a left-to-right complement search, it contains only earlier elements, preventing an element from being used twice.

Example: count event records by service name.

~~~python
def count_by_service(events):
    counts = {}

    for event in events:
        service = event["service"]
        counts[service] = counts.get(service, 0) + 1

    return counts
~~~

- Expected time: O(n) for n events, assuming average constant-time dictionary operations.
- Auxiliary space: O(u), where u is the number of distinct service names.

**DevOps/SRE application:** Count alerts, requests, or deployment events by service, status, or region.

**Do not choose it blindly:** If keys are unhashable, stable sorted order is required, or the task is an ordered range query, another structure may fit better.

### 3.2 Two pointers

**Consider it when** a sequence is sorted and a pair or boundary condition is involved; two ordered sequences must be merged; values must be compacted in place; or a left/right boundary can move without revisiting discarded candidates.

**Invariant:** The unprocessed region still contains every candidate that could satisfy the condition. The reason a pointer advances must follow from ordering or a problem-specific proof.

Common variants include pointers moving toward each other, moving forward at different rates, a scan pointer plus a write pointer, or one pointer per sorted input.

- Often O(n) time and O(1) auxiliary space for a single scan.
- Exact cost depends on the operation and required output storage.

**Do not use it merely because there are two indices.** On unsorted input, the usual pair-sum pointer movement is not justified unless the data is sorted or a different invariant is established.

### 3.3 Sliding window

**Consider it when** the answer concerns a contiguous region and the current region can be updated as boundaries move instead of recalculated from scratch.

- **Fixed-size:** Add the entering element and remove the leaving element.
- **Variable-size:** Expand the right boundary; move the left boundary while a condition permits contraction.

**Invariant:** Stored state describes exactly the current contiguous region.

Typical state includes a running sum, frequency map, distinct-count, or deque of candidate indices.

**Important limitation:** The familiar variable-window method for a minimum subarray sum relies on positive values. With arbitrary negative values, removing the left value does not necessarily reduce the sum. Prove boundary movement before applying the technique.

Many standard window algorithms are O(n) because each boundary advances at most n times. Expensive work inside the loop can change that total.

**DevOps/SRE application:** Analyze contiguous time buckets in an ordered event stream, such as a fixed number of consecutive samples. Clarify whether samples are equally spaced and how missing samples are treated.

### 3.4 Prefix sums

**Consider it when** many queries ask for sums over static contiguous ranges.

For an array a, define P[0] = 0 and P[i+1] = P[i] + a[i]. The half-open range [l,r) has sum:

\[
P[r]-P[l]
\]

- Build time: O(n).
- Query time: O(1).
- Extra storage: O(n).

**Do not assume prefix sums solve dynamic updates.** If values change between queries, a Fenwick tree or segment tree may be more appropriate. See [16 — Advanced Data Structures](../../Complete%20DSA/16%20Advanced%20Data%20Structures/README.md).

**DevOps/SRE application:** Answer many static queries for total requests or errors over selected sample ranges after a batch has been collected.

### 3.5 Fast and slow pointers

**Consider it when** a linked list may contain a cycle, the middle or a position relative to the end is needed, or pointers moving at different speeds create a useful positional relationship.

**Invariant:** State the relationship between pointer positions. For cycle detection, if a cycle exists, the relative distance changes around the cycle until the pointers meet.

- Typical cycle detection: O(n) time and O(1) auxiliary space.
- Use node identity to detect that two references point to the same node; equal values do not imply equal nodes.

The useful property is the speed difference, not simply having two variables.

### 3.6 Sorting and interval patterns

**Consider it when** intervals overlap or must be scheduled, ordering makes neighboring records easier to compare, or a problem asks for minimum resources across time ranges.

Common approaches:
- **Sort and merge:** Produce combined non-overlapping intervals.
- **Sweep line:** Convert starts and ends into ordered events and track active items.
- **Heap:** Track the next finishing or highest-priority active interval.

**Clarify interval semantics.** For closed intervals, [a,b] and [b,c] overlap at b. For half-open intervals, [a,b) and [b,c) do not overlap. The comparison operator changes with the contract.

Sorting-based approaches commonly take O(n log n) time. Extra space depends on output, event records, or a heap.

**DevOps/SRE application:** Merge maintenance windows, deployment windows, or periods of service unavailability. Do not merge intervals if the task asks for maximum concurrent jobs; that may call for a sweep line or heap.

### 3.7 Monotonic stack

**Consider it when** each item needs its next or previous greater/smaller item, or a nearest qualifying item is required.

**Invariant:** The stack holds unresolved candidates in monotonic order. When a new item resolves previous candidates, pop them.

A standard next-greater-element implementation pushes each index once and pops it at most once.

- Time: O(n).
- Auxiliary space: O(n).

**Pitfalls:** Decide whether equal values should remain on the stack. Using < versus <= changes duplicate behavior. Store indices when results must be written to positions or checked against a window boundary.

### 3.8 Monotonic deque

**Consider it when** you need the maximum or minimum in every fixed-size moving window. Candidates expire from the left and less useful candidates can be discarded from the right.

**Invariant:** The deque contains indices in window order and their values are monotonic. The front is the current best candidate; expired indices are removed.

- Time: O(n) for the standard moving-window algorithm.
- Auxiliary space: O(k) for a window of size k.

Each index is inserted and removed at most once. A normal FIFO queue and a monotonic deque maintain different invariants.

### 3.9 Binary search and boundary search

**Consider it when** input is sorted under the comparison being used, a monotonic predicate divides a range into false and true regions, or a candidate numeric answer can be tested for feasibility.

**Invariant:** The search interval still contains the boundary or answer. Define exactly what the bounds mean and ensure each iteration shrinks the interval.

- Search over n ordered candidates: O(log n), assuming O(1) comparisons.
- If a feasibility check costs O(f(n)), binary search over an answer range of size R costs O(f(n) log R), plus setup.

**Do not use it on a non-monotonic condition.** Prove that once a candidate is feasible (or infeasible), all candidates on the appropriate side remain so.

Python's bisect module finds insertion points in sorted sequences; insertion into a Python list still shifts elements and is not O(1).

### 3.10 Heap and top-K

**Consider it when** you repeatedly need the smallest or largest pending item, only the best k candidates need to be retained, several sorted streams must be merged, or tasks are processed by priority or timestamp.

**Invariant:** The heap root is the highest-priority item according to its ordering. Python's heapq provides a min-heap, so the smallest item is at index zero.

For retaining the largest k values, a min-heap of size at most k keeps the smallest retained value at its root. Replace it only when a larger candidate arrives.

- Maintaining a heap of size k: commonly O(n log k) time and O(k) auxiliary space.
- Sorting retained k items adds O(k log k) if sorted output is required.

Clarify whether duplicates are retained, whether only the K-th value is needed, whether input is a stream, and whether output must be sorted.

**DevOps/SRE application:** Keep the top K services by error count, slowest requests, or highest resource usage. Define tie-breaking behavior.

### 3.11 Breadth-first search (BFS)

**Consider it when** the input is a graph or can be modeled as one, you need reachability by layers, the fewest number of edges in an unweighted graph, or level-order traversal.

**Invariant:** The queue processes discovered vertices in nondecreasing distance from the start. Mark a vertex discovered when enqueued to avoid duplicate queue entries.

- Adjacency-list traversal: O(V+E) over the reachable graph.
- Auxiliary space: up to O(V) for visited/distance state and queue.

**Do not use ordinary BFS for arbitrary weighted shortest paths.** Equal edge costs make first discovery give minimum edge-count distance. For weighted graphs, choose an algorithm based on weight assumptions.

### 3.12 Depth-first search (DFS)

**Consider it when** you need reachability, connected components, graph structure, or a traversal naturally described by exploring one path before backtracking.

**Invariant:** Track visited vertices so each reachable vertex is processed according to the traversal rules. Cycle detection may need additional state to distinguish unvisited, active, and completed vertices.

- Adjacency-list traversal: O(V+E).
- Auxiliary space: O(V) in the worst case for visited state and recursion/explicit stack.

**Pitfall:** Recursive DFS can hit Python's recursion limit on deep inputs. An explicit stack may be safer for very deep traversals.

### 3.13 Topological sorting

**Consider it when** tasks have prerequisite relationships and a valid order must respect every directed dependency.

**Invariant:** In Kahn's algorithm, the queue contains vertices whose remaining in-degree is zero. Removing one releases its outgoing neighbors.

- Time: O(V+E).
- Auxiliary space: O(V).

A topological order exists only for a directed acyclic graph (DAG). If fewer than V vertices are output, a directed cycle prevents a full ordering.

**DevOps/SRE application:** Model build stages, deployment prerequisites, or service startup dependencies as a directed graph. A valid order exists only if the dependency graph is acyclic.

### 3.14 Disjoint Set Union (DSU / Union-Find)

**Consider it when** edges are added and connectivity queries follow, or groups must be combined and queried.

**Invariant:** Each element belongs to one represented set. find identifies its representative; union combines two sets.

With path compression and union by rank/size, operations have near-constant amortized complexity (more precisely, inverse-Ackermann amortized time per operation under the standard analysis).

**Do not choose DSU** when you need shortest paths, route reconstruction, or frequent deletions unsupported by the basic structure.

### 3.15 Backtracking

**Consider it when** every valid combination, arrangement, or path must be generated and a solution is built through a sequence of choices.

**Invariant:** The current state represents exactly the choices made along the active search path. After exploring a choice, restore state before trying another branch.

A pruning rule may stop a branch only when no completion of that branch can be valid.

- Time: often exponential; exact complexity depends on branching, depth, and pruning.
- Auxiliary space: commonly O(d) for search depth d, excluding output.
- Output itself may be exponential.

**Pitfalls:** Forgetting to undo a mutation, appending a mutable working list without copying it, or pruning a branch that could still contain a solution.

### 3.16 Dynamic programming (DP)

**Consider it when** the same subproblems recur and the answer can be assembled from smaller answers using a state and recurrence.

Before coding, define:
1. the meaning of each state;
2. the recurrence;
3. base cases;
4. evaluation order or memoization strategy;
5. dimensions of the state space;
6. whether the requested path or choices must be reconstructed.

**Invariant:** Each stored state has the meaning defined by the recurrence and contains the correct result for that subproblem.

Derive complexity from the number of states multiplied by work per state. Do not label a solution O(n) without counting both.

Python's functools.cache and lru_cache can memoize calls when arguments are hashable and cache memory is acceptable. Memoization does not remove the need to define correct states and base cases.

### 3.17 Greedy

**Consider it when** a locally best choice appears to make progress toward a global optimum and a proof can show the choice is safe.

**Proof obligation:** Establish why choosing the local option cannot make the remaining problem worse than an optimal alternative. Exchange arguments and stays-ahead arguments are common techniques.

**Do not infer correctness from examples.** For arbitrary coin denominations, repeatedly taking the largest coin is not guaranteed to minimize the number of coins.

### 3.18 Trie

**Consider it when** many strings are queried by prefix, prefix existence/enumeration is central, or shared prefixes can be represented once.

**Invariant:** Each path from the root represents a prefix; terminal markers distinguish complete stored words from prefixes of longer words.

- Search/insert: commonly O(L) for string length L, assuming constant-time child lookup.
- Space: depends on stored nodes and child representation.

Do not use a trie for a small number of ordinary exact lookups if a set or dictionary is simpler and more memory efficient.

---

## 4. Distinguish patterns that are easy to confuse

| Similar-looking requirements | Prefer this when | Key distinction |
|---|---|---|
| Two pointers vs hash map | Sorted input enables safe pointer movement; otherwise lookup map may be simpler | Ordering versus remembered values |
| Sliding window vs prefix sums | A contiguous window can be adjusted; many static range queries need independent answers | Moving region versus arbitrary range queries |
| Sliding window vs monotonic deque | Window state is a sum/frequency versus max/min candidate tracking | Aggregate update versus extremum candidates |
| Prefix sums vs Fenwick/segment tree | Static values versus interleaved updates and queries | No updates versus dynamic operations |
| Merge intervals vs heap/sweep line | Need merged ranges versus peak concurrency or active interval tracking | Output shape and event ordering |
| BFS vs Dijkstra | Equal edge costs versus nonnegative weighted edges | Edge-count layers versus accumulated path cost |
| DFS vs backtracking | Explore graph structure versus enumerate choices and undo state | Traversal versus decision-tree search |
| Backtracking vs DP | Enumerate solutions versus reuse subproblem results | Output enumeration versus state reuse |
| Greedy vs DP | Safe local choice has a proof versus alternatives require state comparison | Proven choice versus recurrence |
| Heap vs sorting | Repeated best-item retrieval or bounded top K versus one complete ordered result | Incremental priority versus full ordering |
| Binary search vs linear scan | Ordered/monotonic property safely discards candidates | Proven boundary versus no safe elimination |

---

## 5. Constraint-based selection

Constraints help reject unsuitable approaches. They do not provide a universal mapping from input size to one algorithm; constants, data shape, language, memory, and required output all matter.

- If n is large, question nested O(n²) scans and repeated sorting.
- If all pairs are required as output, remember output size itself may be quadratic.
- If only the best k items are needed and k is small relative to n, consider a bounded heap.
- If there are many static range queries, consider prefix-sum preprocessing.
- If updates are interleaved with queries, investigate a dynamic data structure.
- If the graph is sparse or dense, account for representation and V+E traversal cost.
- If all combinations must be returned, expect output-sensitive or exponential work.
- If values may be negative, re-check sum-window and greedy assumptions.
- If a numeric answer can be tested with a monotonic feasibility predicate, binary search on answer may be viable.

Never claim an input size automatically makes an approach acceptable without considering actual constraints and execution environment.

---

## 6. An interview decision worksheet

| Question | Your answer |
|---|---|
| What is the exact input and output contract? | |
| What are the largest input dimensions? | |
| Which assumptions are explicitly guaranteed? | |
| What is the simplest correct baseline? | |
| What operation makes the baseline expensive? | |
| What structure or property could remove that cost? | |
| Which pattern is the leading candidate? | |
| What invariant or recurrence justifies it? | |
| What edge case could break the approach? | |
| What are time and auxiliary-space costs? | |
| What alternative would I use if an assumption changed? | |

A good interview explanation connects every choice to a property of the problem.

---

## 7. Pattern recognition drills

For each drill, identify the likely pattern, state the precondition that makes it valid, and estimate complexity. The prompt deliberately does not name the pattern.

### Drill 1 — Duplicate deployment identifiers

A batch contains deployment IDs. Determine whether any ID appears more than once. Input is not sorted and original order does not matter.

**Reasoning cue:** Membership tracking avoids pairwise comparison.

### Drill 2 — Pair in an ordered metric list

A nondecreasing list of measurements is given. Determine whether two different positions sum to a target.

**Reasoning cue:** Ordering supports safe movement of two boundaries. Ensure the same position is not reused.

### Drill 3 — Peak request total

Given one request count per minute, find the maximum total across any k consecutive minutes.

**Reasoning cue:** Every candidate has the same length. Update the previous total by adding one value and removing one value.

### Drill 4 — Many historical range totals

A static array contains daily incident counts. Answer many queries for total in half-open range [l,r). There are no updates.

**Reasoning cue:** Precompute cumulative information so each query uses two stored boundaries.

### Drill 5 — Earliest available machine

Jobs arrive with priorities. Repeatedly process the job with the smallest priority; new jobs can be inserted during processing.

**Reasoning cue:** Repeated best-item retrieval from a changing collection.

### Drill 6 — Service dependency order

A deployment pipeline has directed prerequisite relationships. Produce an order respecting every prerequisite, or report that no complete order exists.

**Reasoning cue:** Model tasks as vertices and prerequisites as directed edges. A cycle prevents a full order.

### Drill 7 — Minimum number of network hops

An unweighted network graph is given. Find the fewest edges from one node to another.

**Reasoning cue:** Search by layers. Do not extend this reasoning to arbitrary edge costs.

### Drill 8 — Largest contiguous error burst

Find the longest contiguous segment of status codes containing no more than two distinct codes.

**Reasoning cue:** Maintain frequencies in a variable-size window.

### Drill 9 — Next threshold crossing

For every metric sample, find the first later sample whose value is strictly greater. Return no result if none exists.

**Reasoning cue:** Maintain unresolved indices. Be precise about strict versus non-strict comparison.

### Drill 10 — Capacity planning

Find the smallest server capacity that can process all jobs within a fixed time. A helper correctly tests a proposed capacity, and if a capacity is sufficient, every larger capacity is also sufficient.

**Reasoning cue:** Feasibility is monotonic. Include the cost of the helper.

### Drill 11 — All valid configuration combinations

Given independent option groups and constraints, generate every configuration that satisfies the constraints.

**Reasoning cue:** Build a partial configuration through choices, reject impossible partial states, and restore state after exploring a branch.

### Drill 12 — Repeated state evaluation

A recursive cost calculation repeatedly reaches the same state identified by a small tuple of parameters. Only minimum cost is required, not every path.

**Reasoning cue:** Define state and recurrence, then reuse results.

### Drill 13 — Top K noisy services

Given a large batch of services and error counts, return only the k services with the largest counts.

**Reasoning cue:** If k is much smaller than the number of services, retain only k candidates. Define tie behavior.

### Drill 14 — Connectivity after adding links

Initially every host is isolated. Links are added one by one, and after each addition the system asks whether two hosts are connected.

**Reasoning cue:** Merge components and query component membership.

---

## 8. Drill answer key

These are candidate approaches under the stated assumptions, not a claim that no alternative can work.

| Drill | Candidate | Why it fits | Typical cost |
|---|---|---|---|
| 1 | Set | Tracks previously seen IDs | O(n) expected time, O(u) space |
| 2 | Two pointers | Sorted order supports safe movement | O(n) time, O(1) extra space |
| 3 | Fixed sliding window | Add entering and remove leaving sample | O(n) time, O(1) extra space |
| 4 | Prefix sums | Static range query is a prefix difference | O(n) build, O(1) per query, O(n) space |
| 5 | Heap | Repeated minimum-priority retrieval | O(log n) insertion/removal |
| 6 | Topological sort | Directed dependency ordering | O(V+E) time |
| 7 | BFS | Minimum edge-count path in unweighted graph | O(V+E) time |
| 8 | Variable sliding window | Track at most two distinct codes | O(n) expected time with frequency map |
| 9 | Monotonic stack | Resolves next-greater queries as values arrive | O(n) time |
| 10 | Binary search on answer | Feasibility is explicitly monotonic | O(f(n) log R) if check costs f(n) |
| 11 | Backtracking | Enumerates choices and prunes invalid partial states | Output-sensitive; can be exponential |
| 12 | Dynamic programming | Repeated states can be reused | Number of states × work per state |
| 13 | Bounded heap | Retains only best k candidates | Commonly O(n log k), O(k) space |
| 14 | DSU | Component merges and connectivity checks | Near-constant amortized per operation with path compression and union by size/rank |

Challenge the assumptions. If Drill 4 included updates, plain prefix sums would no longer answer all operations efficiently. If Drill 7 had nonnegative but unequal edge weights, ordinary BFS would not generally minimize total path cost.

---

## 9. A seven-day pattern recognition practice plan

| Day | Focus | Practice task |
|---|---|---|
| 1 | Hash maps, sets, two pointers | One membership/counting problem and one sorted-pair problem |
| 2 | Sliding windows, prefix sums | One fixed window, one variable window, one range-query problem |
| 3 | Intervals, stacks, deques | Compare merge intervals, next greater element, and window maximum |
| 4 | Binary search, heaps | Solve a boundary search and a top-K problem |
| 5 | BFS, DFS, topological sort, DSU | Draw the graph model before writing traversal code |
| 6 | Backtracking, DP, greedy | Explain why each approach fits and where it can fail |
| 7 | Mixed review | Complete unseen drills without the answer key, then explain each invariant aloud |

For every solved problem, record the baseline, pattern signal, exact precondition, invariant or recurrence, complexity, a failing example for a tempting invalid approach, and one alternative.

Repeat missed problems after a delay rather than memorizing only final code.

---

## 10. Common pattern-recognition mistakes

1. **Keyword matching:** Choosing a sliding window whenever the prompt says substring, without checking whether window state can be updated.
2. **Ignoring input guarantees:** Using two pointers on unsorted data without a movement proof.
3. **Assuming positivity:** Applying a shrinking sum-window algorithm when negative values are allowed.
4. **Confusing static and dynamic queries:** Using prefix sums when updates are interleaved with queries.
5. **Ignoring edge weights:** Using BFS to minimize arbitrary weighted path cost.
6. **Using greedy by intuition:** Failing to prove the local choice preserves an optimal solution.
7. **Misusing binary search:** Searching a condition that is not monotonic.
8. **Forgetting duplicate semantics:** Choosing < versus <= incorrectly in a monotonic stack or window.
9. **Ignoring output size:** Expecting every combination to be returned in polynomial total time when output itself is exponential.
10. **Skipping complexity assumptions:** Calling hash-table operations guaranteed O(1), or ignoring feasibility-check cost.
11. **Overfitting to one implementation:** Treating a code template as more important than the problem contract.
12. **Not testing the invariant:** Checking only a normal example instead of empty, smallest, duplicate, boundary, and adversarial cases.

---

## 11. Further study in this repository

Use the broader Complete DSA track for full concept lessons and implementations:

- [02 — Arrays and Strings](../../Complete%20DSA/02%20Arrays%20and%20Strings/README.md)
- [03 — Hash Tables and Sets](../../Complete%20DSA/03%20Hash%20Tables%20and%20Sets/README.md)
- [04 — Linked Lists](../../Complete%20DSA/04%20Linked%20Lists/README.md)
- [05 — Stacks and Queues](../../Complete%20DSA/05%20Stacks%20and%20Queues/README.md)
- [06 — Recursion and Backtracking](../../Complete%20DSA/06%20Recursion%20and%20Backtracking/README.md)
- [07 — Sorting](../../Complete%20DSA/07%20Sorting/README.md)
- [08 — Searching](../../Complete%20DSA/08%20Searching/README.md)
- [09 — Trees and Binary Search Trees](../../Complete%20DSA/09%20Trees%20and%20Binary%20Search%20Trees/README.md)
- [10 — Heaps and Priority Queues](../../Complete%20DSA/10%20Heaps%20and%20Priority%20Queues/README.md)
- [11 — Graphs](../../Complete%20DSA/11%20Graphs/README.md)
- [12 — Greedy Algorithms](../../Complete%20DSA/12%20Greedy%20Algorithms/README.md)
- [13 — Divide and Conquer](../../Complete%20DSA/13%20Divide%20and%20Conquer/README.md)
- [14 — Dynamic Programming](../../Complete%20DSA/14%20Dynamic%20Programming/README.md)
- [15 — Advanced Graph Algorithms](../../Complete%20DSA/15%20Advanced%20Graph%20Algorithms/README.md)
- [16 — Advanced Data Structures](../../Complete%20DSA/16%20Advanced%20Data%20Structures/README.md)
- [17 — Interview Patterns](../../Complete%20DSA/17%20Interview%20Patterns/README.md)
- [18 — DSA Practice Problems](../../Complete%20DSA/18%20DSA%20Practice%20Problems/README.md)

Continue the interview preparation sequence:

- [Interview Preparation Home](../README.md)
- [01 — Interview Roadmap](../01%20Interview%20Roadmap/README.md)
- [02 — Python Coding Interview](../02%20Python%20Coding%20Interview/README.md)

---

## 12. Sources and quality notes

Official Python documentation consulted for standard-library behavior used by the pattern examples:

- [Python — Built-in Types](https://docs.python.org/3/library/stdtypes.html) — built-in containers and sequence/mapping behavior.
- [Python — collections](https://docs.python.org/3/library/collections.html) — specialized containers, including Counter, defaultdict, and deque.
- [Python — heapq](https://docs.python.org/3/library/heapq.html) — heap invariants and min-heap operations.
- [Python — bisect](https://docs.python.org/3/library/bisect.html) — insertion-point search in sorted sequences.
- [Python — itertools](https://docs.python.org/3/library/itertools.html) — iterator building blocks and combinatoric iterators.
- [Python — functools](https://docs.python.org/3/library/functools.html) — memoization helpers such as cache and lru_cache.

These references verify the named Python APIs and their documented behavior. They are not presented as proof of every algorithmic correctness or complexity claim. Those claims are explained using the stated algorithm assumptions and invariants.

**Verified facts:** The linked Python documentation describes the standard-library facilities named above.

**Repository design decision:** This guide groups algorithmic techniques by problem structure and teaches a repeatable interview selection workflow.

**Demonstrations:** The drills and DevOps/SRE scenarios are educational examples. Adapt them to the exact input contract and validate implementations with tests before production use.

---

**Previous:** [02 — Python Coding Interview](../02%20Python%20Coding%20Interview/README.md)  
**Back to:** [DSA Interview Preparation](../README.md)
